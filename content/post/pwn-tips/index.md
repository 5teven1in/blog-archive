---
title: Pwn tips
description: Practical preparation tips for Pwn challenges, including disabling alarm timeouts and linking a libc build with debugging information.
slug: pwn-tips
date: 2018-06-22 23:36:52+08:00
categories:
    - Note
tags:
    - Pwn
    - GDB
    - glibc
---

## Before Pwn

<!--more-->

### alarm

If a binary uses `alarm` as a timeout and you are sure the vulnerability is not inside `alarm`, you can patch its function name to `isnan`.

1. Open the binary in Vim.
2. Replace the function name `alarm` with `isnan`, then save the file.
3. Done.

### Link libc with debugging information

When debugging with GDB, stepping into a libc function normally provides no symbols or similar debugging information. This makes debugging harder, so it is useful to link the binary against a libc build that includes debugging information.

1. Follow [Build libc with debug info]({{< ref "/post/build-libc-with-debug-info" >}}) to build a libc with debugging information.
2. Run `ldd <binary>` to identify the linked loader.
3. Patch the loader path so it points to the loader you built.
4. Done.

#### A shortcut for patching the loader path

Suppose the original loader path is `/lib64/ld-linux-x86-64.so.2`. First create a symbolic link at `/lib64/ld_linux-x86-64.so.2` that points to your custom loader. You then only need to replace `ld-` with `ld_` in the binary's loader path.

#### Note

After replacing the loader, it will prefer the libc in the same directory, allowing the binary to link against the libc build with debugging information.
