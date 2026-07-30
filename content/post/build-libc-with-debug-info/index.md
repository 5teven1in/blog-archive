---
title: Build libc with debug info
description: Build glibc 2.23 with complete debugging information on Ubuntu 16.04.
slug: build-libc-with-debug-info
date: 2018-06-22 23:36:52+08:00
categories:
    - Note
tags:
    - glibc
    - GDB
    - Pwn
---

This article uses 64-bit Ubuntu 16.04.

<!--more-->

## Download the source code

Download the glibc source code from the official [GNU C Library (glibc)](https://www.gnu.org/software/libc/) website. This example uses glibc 2.23.

Download and extract glibc 2.23:

```shell
wget https://ftp.gnu.org/gnu/libc/glibc-2.23.tar.gz
tar zxvf glibc-2.23.tar.gz
```

## Configure and build

Enter the glibc 2.23 directory and create a separate build directory:

```shell
cd glibc-2.23
mkdir build
```

Enter the build directory and configure the build options:

- CFLAGS
    - Enable full debugging information and minimize optimization.
    - See the example below.
- prefix
    - The destination for the compiled files.
    - This example installs them in a `glibc` directory under the current user's home directory.

```shell
cd build
CFLAGS='-g3 -ggdb3 -gdwarf-4 -Og -Wno-error' ../configure --prefix=/home/`whoami`/glibc
```

Build and install:

```shell
make -j4
make install -j4
```

## Notes

To build a 32-bit version of glibc:

- CC
    - gcc -m32
- CFLAGS
    - host=i686-linux-gnu
    - build=i686-linux-gnu

```shell
CC='gcc -m32' CFLAGS='-g3 -ggdb3 -gdwarf-4 -Og -Wno-error --host=i686-linux-gnu --bulid=i686-linux-gnu' ../configure --prefix=/home/`whoami`/glibc
```
