---
date: 2021-08-31
title: HelloWorld系列一
tags: [计算机基础]
---

对抗无知的武器

- GNU Toolchain: gcc, binutils, binutils-dev
- ELF editor
- ggcov

binutils 常用工具

- ar: 建立static library (Insert, Delete, List, Extract)
- strings: 
- strip
- nm
- size
- readelf
- objdump

从源代码到可执行文件的四大阶段

- Pre-processing 预处理: gcc -E hello.c -o hello.i
- Compilation 编译: gcc -c hello.i -o hello.s
- Assembly 汇编: gcc -S hello.s -o hello.o
- Linking 链接: gcc ld hello.o -o hello

## 


## Reference

[memory-layout-of-c-program](https://www.geeksforgeeks.org/memory-layout-of-c-program/)