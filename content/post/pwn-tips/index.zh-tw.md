---
title: Pwn 技巧
description: Pwn 題目分析前的實用技巧，包括移除 alarm timeout 與連結帶除錯資訊的 libc。
slug: pwn-tips
date: 2018-06-22 23:36:52+08:00
categories:
    - Note
tags:
    - Pwn
    - GDB
    - glibc
---

## 開始 Pwn 之前

<!--more-->

### 移除 alarm timeout

如果有 `alarm` 做 timeout 且確定題目的洞不在 `alarm` 裡，可以將它 patch 成 `isnan`

1. 把 binary 用 vim 打開
2. 將 `alarm` 的 function name 取代為 `isnan` 儲存
3. 完成

### 連結帶有除錯資訊的 libc

一般使用 gdb debug 的時候，進入 libc 的 function 中不會 symbol 等資訊。這樣比較難 debug 所以會希望可以 link 到一個有 debug info 的 libc

1. 參考 [Build libc with debug info]({{< ref "/post/build-libc-with-debug-info" >}}) 先編出一個帶有 debug info 的 libc
2. 使用 `ldd <binary>` 查看 link 到的 loader
3. 將 loader 路徑 patch 成自己編的 loader 路徑
4. 完成

#### 修改 loader 路徑的小技巧

假設原本 loader 路徑為 `/lib64/ld-linux-x86-64.so.2`，可以先建一個 soft link `/lib64/ld_linux-x86-64.so.2` 指向自己編的 loader，接著只要把 binary loader 路徑中的 `ld-` 換成 `ld_` 即可

#### 補充

更換 loader 後他就會優先使用該目錄下的 libc，所以就可以 link 到有 debug info 的 libc囉
