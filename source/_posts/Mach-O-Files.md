---
title: Mach-O Files
date: 2018-04-09 00:19:01
tags:
---

程序经过编译器编译，将源代码编译成 `.o` 文件，被动态链接器链接，打包成可执行文件或者静态库，最后被用户执行，这是代码到可执行文件再到被计算机运行的整个过程。

今天我想根据之前看过的文章还有对动态链接器的了解，记录一下关于动态链接器的一切。

## 0x00 动态链接器

首先，我们上文提到的动态链接器 dynamic linker，你肯定见过它们在不同平台对应的链接文件，只是不知道它的名字而已。在 Microsoft Windows 系统，它叫 DLL (Dynamic link library)，在遵循 [ELF 可执行文件格式](https://en.wikipedia.org/wiki/Executable_and_Linkable_Format) 的 Unix 系统（不包括 Darwin），你会看到一个 `.so` （shared object） 拓展名的文件。在 macOS 和 iOS 系统，则有 `dylib` (dynamic library)。