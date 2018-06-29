---
title: Swift 里的几个容易混淆的概念
date: 2018-05-28 21:56:42
tags:
---

## Package

> 一个 Packge 里面通常会包含源文件目录 Sources 以及一个单独的[清单文件](https://zh.wikipedia.org/wiki/%E6%B8%85%E5%8D%95%E6%96%87%E4%BB%B6)（Manifest file）

如果你安装了 Swift Package Manager 的话，可以用 `swift package init` 来快速初始化一个 Package。

目录如下：

````swift
Souces
    sourceA.swift
    sourceB.swift
Package.swift
````

之后你会注意到目录中有一个 `Package.swift`, 就是我们上面所指的清单文件。清单文件主要用来定义或拓展一些规范。

[里面](https://github.com/apple/swift-package-manager/blob/master/Documentation/PackageDescriptionV4.md#package-manifest-file-format-reference) 包含有很多属性，例如：

- name: The name of the package.
- pkgConfig: Name of the pkg-config (.pc) file to get the additional flags for system modules.
- providers: Defines hints to display for installing system modules.
- products: The products vended by the package.
- dependencies: The external package dependencies.
- targets: The list of targets in the package.
- swiftLanguageVersions: Specifies the set of supported Swift language versions.

## Library

### Linking Library

我们知道从源代码到可执行文件的过程需要通过几个步骤：

> 源码 - 编译预处理 - 编译程序(\*.s) - 优化程序 - 汇编程序(\*.o) - 链接程序 - 可执行文件（*.out）

其中链接部分实质是将很多需要的 *.o 文件链接起来，最后生成 *.out 可执行文件。

1. Static linking 静态链接

![](http://ww1.sinaimg.cn/large/8c7526ebgy1fsautl5jmfj20x60mc0tm.jpg)

首先，静态意味着源代码被编译后，会被**全部**塞进最终的二进制文件中，最后包体积会相对比较大，由于 library 包含的内容多，因此加载起来会比较慢。还有个问题就是，如果尝试链接两个相同的 library 会导致 symbol duplicate。你一定遇到过这种情况，就是有的时候在项目中创建了两个相同的文件，编译时就会报错。

2. Dynamic linking 动态链接

跟静态链接库相对的，是动态链接库。

![](http://ww1.sinaimg.cn/large/8c7526ebgy1fsautl4sykj20x60kf0tu.jpg)

动态链接库只会在运行时加载，因此并不会因为合并到二进制文件中而导致包体积增加。

## Framework

> Framework 其实是一个包含很多东西的 *文件夹*，里面可以包括有：动态链接库，nib 文件，图片资源文件，本地化文件，头文件以及相关文档。

如果你使用过 [Carthage](https://github.com/Carthage/Carthage) ，那你一定见过 ` /Carthage/Build/` 目录下的 xxx.Framework 文件。

Framework 按我个人的理解来说，其实更像是开发者开发好的框架，开发者已经（根据 Library）开发好接口，你去调用就好了。

我们经常使用的 UIKit，Foundation 等等都是 Framework，Framework 其中里面包含有图片素材等其他资源文件。

## Module

> Swift organizes code into modules. Each module specifies a namespace and enforces access controls on which parts of that code can be used outside of the module.

````swift
import Alamofire
import UIKit
````
我们首先要知道命名空间这个概念，在没有 Module 之前 （Objective-C），我们需要手动地用 #import 来 import framework，还要手动在 Xcode 里 链接 framework's binary。编译时，有时候会由于两个 framework 里使用了相同的宏定义而导致冲突，因此你需要在宏前面加上你独特的前缀，如 `NS_MY_MACRONAME`。

而 Swift 里的命名空间是隐式的，所以当你调用 `NSLog` 函数的时候，实际上是 `Foundation.NSLog`。

举个例子，假设两个 modules，moduleA 和 moduleB 都有两个相同的函数，都叫 foo，这时候可以在函数前面添加 module 的命名空间来区分开来。

````swift
import moduleA
import moduleB

moduleA.foo()

moduleB.foo()
````

