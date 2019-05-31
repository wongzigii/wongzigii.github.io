---
title: <译> Understanding Buffer Overflows Attack
date: 2018-02-22 21:01:35
tags:
---

[原文地址](https://itandsecuritystuffs.wordpress.com/2014/03/18/understanding-buffer-overflows-attacks-part-1/)

## 内存的分布结构图

![](https://itandsecuritystuffs.files.wordpress.com/2014/03/image_thumb.png?w=415&h=480&zoom=2)

上图是 x86 架构的处理器的内存结构图，其中我们应用程序的代码，蓝色部分，被写在内存地址的最低位，而红色部分，栈地址则位于最高位，紫色部分，堆地址，位于内存中间位置。

## 寄存器

每个不同型号的处理器架构是不一样的。我们用的 x86 处理器和摩托罗拉，苹果手机的处理器架构也不一样。即使同样是 x86 处理器，不同处理器之间也区分 16bits，32bits 和 64bits 的寄存器。

在 x86-32bits 架构中，有以下 8 个通用的寄存器用来指向内存中的其他位置：

- EAX
- EBX
- ECX
- EDX
- ESI
- EDI
- ESP: Extended Stack Pointer - 栈指针，总是指向栈顶
- EBP: Extended Base Stack Pointer - 也是栈指针，但是是指向栈的基地址

我们集中讨论 ESP EBP 这几个比较重要的寄存器。（其实还有 EIP -- Extended Instruction Pointer, 它是一个只读的寄存器，指向 CPU 下次要执行的地址。）

## 栈

栈是一个先进后出的数据结构。

````c
void fun(void) {
    printf("Hello world")
}

void main() {
    fun()
    printf("End")
}
````

考虑上面的代码，系统会帮我们将 EBP 入栈 (push EBP)，当函数调用结束后再将 EBP 出栈 (pop EBP)。其中会用到三种寄存器:

- EIP
- ESP
- EBP

调用 fun 函数前：

1. EIP 寄存器保存 CPU 下次要执行的指令的地址，就是执行完 fun 函数后要执行 printf 语句的地址
2. fun 函数调用前，EBP 寄存器保存着栈的栈底指针，这是由 ESP 传递给 EBP 的，此时 ESP 既是栈顶指针，也是栈底指针 （mov EBP, ESP）
3. 调用过程中，ESP 寄存器始终指向栈顶

调用 fun 函数后：

1. EBP 寄存器保存着栈底指针，而这个地址是调用开始前 （mov EBP, ESP） 由 ESP 传递给 EBP 的，调用完之后，应该要把 EBP 地址回传给 ESP，因此 ESP 又一次指向栈顶的地址。
2. 将之前入栈的 EBP 地址出栈 （pop EBP）


````assembly
push EAX
push byte[EBP+20]
push 3     // 保存上下文环境
call calc  // 正式调用 bar（）
````

````assembly
calc: 
push EBP     // 将 EBP 地址入栈
mov EBP, ESP // 将 ESP 地址赋给 EBP
sub ESP, local size // 为 foo 分配内存地址
...
...
mov ESP, EBP // 将 EBP 地址赋给 ESP，free 本地的变量
pop EBP      // 将 EBP 地址出栈
ret param size // free 参数的内存并返回
````

![](https://itandsecuritystuffs.files.wordpress.com/2014/03/image_thumb1.png?w=555&h=480&zoom=2)

## 缓冲区溢出

说了那么多废话，究竟什么是缓存区溢出。

我们回顾一下函数调用的整个过程，

1. 创建函数调用的堆栈，EBP 作为栈基址
2. 函数的参数通过 EBP+8，EBP+12 等等传入内存
3. 函数被调用，函数结果被保存到 RET 指针指向的地址，即 EBP + 4

假设我们在第二步的时候传了一个 12 个 `A` 的字符串参数进去这个函数，
现在我们的内存如下图，

![](https://itandsecuritystuffs.files.wordpress.com/2014/03/image_thumb2.png?w=617&h=480&zoom=2)

现在，PARAM1 指向我们保存参数的地址，而我们传进去的 12 字节的参数，由于每块内存最多只能保存 4 字节，因此实际上这 12 字节在从低往高的内存地址中被拷贝进去。

那如果我们传进去一个非常非常大的，大到超出内存长度的参数呢？

![](https://itandsecuritystuffs.files.wordpress.com/2014/03/image_thumb3.png?w=617&h=480&zoom=2)

这时候 EIP 寄存器原来的内容就会被覆盖掉，而函数调用后需要用到的 EIP 寄存器所保存的地址，用来执行下一步语句。系统无法知道下一步该怎么做，就会抛出异常。

> 题外话：栈溢出 Stack Overflow 其实是缓冲区溢出 Buffer Overflow 的特殊情形，原因是缓冲区里面包含了堆地址，栈地址和其他内存地址。假设你递归调用一个使用栈地址的函数且没有返回的话，最后会导致栈溢出，而我们上面缓冲区溢出的例子，AAAA 字符串不仅把栈的地址用光了，还把堆和其他内存地址都用光了。

## 利用缓冲区溢出

如果 EIP 被没用的脏数据覆盖掉，程序会崩溃并退出。如果 EIP 被 *别有用心* 的内容覆盖掉，那函数调用完之后会继续执行 *别有用心* 的代码块，通过这样，你就能利用缓冲区溢出来干一点有趣的事情。

但是，要想利用缓冲区溢出，还有几点需要注意：

1. 你不知道 EIP 的确切地址在哪里，从而导致你无法改写 EIP 地址的内容
2. 你改写的地址需要是 ESP 指向*过*的地址，而 ESP 的地址总是在不停地改变，因此当缓冲区溢出的时候，你需要知道 ESP 的地址才行
3. 每个内存地址都有一些没用的十六进制值，像回车字符 0x0a, 换行字符 0x0d 等等，如果 ESP 地址保存着这些没用的地址，那么程序会直接崩溃掉了

如何解决这几个问题，我们逐个问题看：

1. 你需要找到一个可以复现缓冲区溢出的特殊字符串如 （ABCD），当你将整串完整的字符串（PPPPABCDMMMM）传进去后，EIP 的内容会被这个特殊字符串覆盖，这时候你可以通过 ABCD 在PPPPABCDMMMM 字符串的偏移量，找到实际能被利用的 EIP 地址。
2. 有两种办法，第一种是借助一些反编译工具 (Ollydbg, IDAPro, Immunity,等等)，在执行的时候定位 ESP 的值，另一种办法是用程序分析软件 Immunity Debugger 或者 pydbg 来分析程序抛出的异常。
3. 我们可以往里面传一个包含从 0x00 到 0xFF 的测试字符串，并用我们第二个办法，借助一些程序分析工具，判断导致崩溃的字符串是哪一个，并一直重复下去，直到整个测试字符串能通过测试，并不再引起崩溃。

## 最后

我们可以通过一些脚本或者应用 [fuzzers](https://github.com/shellphish/fuzzer) 为我们开发的应用*输入*很多没有用的字符串，以测试是否会崩溃。这里的输入泛指很多东西：包括但不限于，文本输入框的内容，配置文件，上传文件的接口或者应用的进程等等。

