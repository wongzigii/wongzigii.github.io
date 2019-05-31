---
title: <译> Optimizing Swift Build Times
date: 2017-10-24 17:40:22
tags:
---

[原文地址](https://github.com/fastred/Optimizing-Swift-Build-Times)

## 目录

- 函数和表达式的类型检查
- 关注那些编译很慢的文件
- 只构建你需要的架构
- dSYM 的生成
- 编译器优化等级
- 第三方依赖
- 模块化
- 关于 XIBs 文件
- Xcode Schemes
- 使用新的 Xcode 编译系统
- 启用 Swift 的 Concurrent Swift Build Tasks
- 在 Xcode 查看你的编译时间

## 函数和表达式的类型检查

Swift 的构建时间如此之长主要是因为需要做复杂的类型检查。Xcode 默认不会告诉你哪些代码需要很长时间来编译。你可以通过在 Build Settings 中的 `Other Swift Flags` 添加以下参数来查看：

- `-Xfrontend -warn-long-function-bodies=100` （100 指 100 ms，你也可以根据需要调整这个值）
- `-Xfrontend -warn-long-expression-type-checking=100`

![](https://github.com/fastred/Optimizing-Swift-Build-Times/raw/master/assets/times@2x.png)

现在，重新构建你的项目，然后你就会看见像这样的错误警告：

![](https://github.com/fastred/Optimizing-Swift-Build-Times/raw/master/assets/xcode-warning@2x.png)

下一步就是如何定位那些编译很慢的代码。[John Sundell](https://www.swiftbysundell.com/posts/improving-swift-compile-times) 和 [Robert Gummesson](https://medium.com/@RobertGummesson/regarding-swift-build-time-optimizations-fc92cdd91e31) 的文章可以帮上你的忙。

更多资源：

- [Guarding Against Long Compiles](http://khanlou.com/2016/12/guarding-against-long-compiles/)
- [Measuring Swift compile times in Xcode 9 · Jesse Squires](https://www.jessesquires.com/blog/measuring-compile-times-xcode9/)
- [Improving Swift compile times — Swift by Sundell](https://www.swiftbysundell.com/posts/improving-swift-compile-times)
- [Swift build time optimizations — Part 2](https://medium.com/swift-programming/swift-build-time-optimizations-part-2-37b0a7514cbe)

## 关注那些编译很慢的文件
 
上一章介绍了如何在函数及表达式层面改善编译时间，但有时候你也想知道某一个单独文件的总编译时间是多少。

现在，打开命令行，通过以下命令来收集项目文件的编译时间：

````bash
xcodebuild -destination 'platform=iOS Simulator,name=iPhone 8' \
  -sdk iphonesimulator -project YourProject.xcodeproj \
  -scheme YourScheme -configuration Debug \
  clean build \
  OTHER_SWIFT_FLAGS="-driver-time-compilation \
    -Xfrontend -debug-time-function-bodies \
    -Xfrontend -debug-time-compilation" | \
tee profile.log
````
(如果你使用的是 workspace，请自行把 `-project YourProject.xcodeproj` 替换成 `-workspace YourProject.xcworkspace`)

然后我们就可以把有用的编译时间信息提取出来：

````bash
2.7288 (  0.3%)  {compile: Account.o <= Account.swift }
2.7221 (  0.3%)  {compile: MessageTag.o <= MessageTag.swift }
2.7089 (  0.3%)  {compile: EdgeShadowLayer.o <= EdgeShadowLayer.swift }
2.4605 (  0.3%)  {compile: SlideInPresentationAnimator.o <= SlideInPresentationAnimator.swift }
````

更多资源：

- [Diving into Swift compiler performance](https://koke.me/2017/03/24/diving-into-swift-compiler-performance/)

## 只构建你需要的架构

Xcode 默认会帮你设置好这个编译器参数，但是你应该检查一下以确保万无一失。你的项目在 Debug 模式下应该只构建那些需要的架构。

![](https://github.com/fastred/Optimizing-Swift-Build-Times/raw/master/assets/active-arch@2x.png)

更多资源：

- [What is Build Active Architecture Only](http://samwize.com/2015/01/14/what-is-build-active-architecture-only/)

## dSYM 的生成

在新的项目中，dSYM 文件默认不会在 Debug 模式下自动生成。但是，我们有的时候需要在真机调试中生成这个文件来分析程序的崩溃和闪退原因。

因此，推荐设置如下：

![](https://github.com/fastred/Optimizing-Swift-Build-Times/raw/master/assets/dsym@2x.png)

更多资源：

- [Speeding up Development Build Times With Conditional dSYM Generation](http://holko.pl/2016/10/18/dsym-debug/)

## 编译器优化等级

另一个常见的技巧是：

- Debug 模式下，将 `Optimization Level` 设置成 `Fast, Whole Module Optimization`
- 只在 Debug 模式下，在 `Other Swift Flags` 中添加 `-Onone`

![](https://github.com/fastred/Optimizing-Swift-Build-Times/raw/master/assets/wmo_9@2x.png)

这样能使编译器：

> It runs one compiler job with all source files in a module instead of one job per source file

> Less parallelism but also less duplicated work

> It's a bug that it's faster; we need to do less duplicated work. Improving this is a goal going forward

更多资源：

- [Developear - Speeding Up Compile Times of Swift Projects](http://developear.com/blog/2016/12/30/Speed-Swift-Compilation.html)
- [Slava Pestov on Twitter: “@iamkevb It runs one compiler job with all source files in a module instead of one job per source file”](https://twitter.com/slava_pestov/status/911747257103302656)

### 针对 CocoaPods 的编译器优化

如果你的项目使用 CocoaPods，你也该考虑把 Pod 项目的编译器优化打开。你可以通过在 `Podfile`  文件中添加一个 `post_install` 的 hook 来实现：

````ruby
post_install do |installer|
  installer.pods_project.targets.each do |target|
    target.build_configurations.each do |config|
      if config.name == 'Debug'
        config.build_settings['OTHER_SWIFT_FLAGS'] = ['$(inherited)', '-Onone']
        config.build_settings['SWIFT_OPTIMIZATION_LEVEL'] = '-Owholemodule'
      end
    end
  end
end
````

然后运行 `$ pod install`。别忘了对比一下添加这个 hook 前后的编译速度，来确认这个是否有效。

## 第三方依赖

现在有两种方式可以对项目安装第三方依赖：

1.源代码方式，当你 clean build 的时候，第三方依赖的源代码也会被重新编译一次。（例如：CocoaPods, git submodules, copy-pasted code, internal libraries in subprojects that the app target depends on）
2.预构建好的 framework 或者 library （例如：Carthage, static library distributed by a vendor that doesn’t want to provide the source code）

CocoaPods 作为 iOS 开发中最著名的依赖管理器，会在你每次 clean build 的时候将第三方依赖的源代码重新编译一次，从而使项目的编译时间更长。通常情况下，你不会经常地 clean build 项目。但实际情况是：你经常地不得不需要 clean build。（如需要切换分支，或者由于 Xcode 的 bugs 或其他奇怪的原因）

尽管 Carthage 比 CocoaPods 更加难用，但是它确实能加快项目的编译速度。因为 Carthage 的操作大多数都是修改声明依赖的文件 Podfile，（如添加或删除一个依赖，或者将依赖更新到一个新版本）。这些操作可能需要 5-15 分钟来完成，但是你进行这些操作的频率远远比 CocoaPods 重新把源代码编译一遍要少得多。

更多资源：

- time spent waiting for Xcode to finish builds

## 模块化

Swift 的 Incremental compilation 并不是很完美。有时也许仅仅是修改了一个字符串，就会导致整个项目重新编译。

为了避免这个问题，你可以考虑将 app 拆分为几个模块。这里有 2 种方案：动态库和静态库（Xcode 9 Beta4 版本开始支持 Swift 静态库）。

假设你的 app 依赖一个叫做 `DatabaseKit` 的内部 framework。这种方法能够保证在你对 app 项目做了一些修改时，`DatabaseKit` 不会因为 `Incremental compilation` 而重新编译。

更多资源：

- [Technical Note TN2435 – Embedding Frameworks In An App](https://developer.apple.com/library/content/technotes/tn2435/_index.html)
- [uFeatures](https://github.com/microfeatures/guidelines)

## XIBs

XIBs/storyboards 和 纯代码之间的抉择一直是一个热门的讨论话题，我们在这里不再过多地讨论。但是，有趣的是，如果你修改了 Interface Builder 里面的内容，只有那个修改的文件会被重新编译（成 NIB 格式），另外，如果你在 UIView 的某个子类里修改一个公开的方法，Swift 编译器可能会把继承自这个类的所有类都重新编译。

更多资源：

- [(…) in a large project incremental build is much faster if only a .xib was changed (vs. only a line of Swift UI code)](https://twitter.com/MichalCiuba/status/925326831074643968)

## Xcode Schemes

假设我们有一个项目，里面包含有 3 个 targets：

- `App`
- `AppTests`
- `AppUITests` 

尽管我们可以正常工作，但我们还是可以优化一下 scheme 的配置。下面的配置是默认的，里面包含 3 个 scheme:

### App

用 `⌘+B` 构建这个应用只会跑单元测试，而不会跑 UI 测试。这对于 `short iterations` 很有用，（译者注：`short iterations` 应该是指快速迭代的版本，即只改变 UI，而不改变基本功能逻辑），如那些 UI 代码，只有需要的代码被编译了。

![](https://github.com/fastred/Optimizing-Swift-Build-Times/raw/master/assets/app-scheme@2x.png)

### App - 单元测试（App - Unit Test Flow）

Builds 命令不止会编译 app，还包括单元测试。Runs 命令只会跑单元测试。因为你在 build 的时候马上就能找到单元测试中的错误，而不是在运行时，因此，对于修改有单元测试的代码时，这会提高效率。

![](https://github.com/fastred/Optimizing-Swift-Build-Times/raw/master/assets/app-unit-test-flow@2x.png)

### App - 全部测试

Builds 命令会把所有单元测试 target 编译一遍。Runs 命令会跑单元测试和 UI 测试。当涉及到那些和 UI 关系紧密包含 UI 测试的代码时作用较大。

![](https://github.com/fastred/Optimizing-Swift-Build-Times/raw/master/assets/app-all-tests-flow@2x.png)

更多资源：

- [All About Schemes](http://pilky.me/17/)

## 使用新的 Xcode 编译系统

在最新的 Xcode 9 中，[有一个全新的构建系统](https://developer.apple.com/library/content/releasenotes/DeveloperTools/RN-Xcode/Chapters/Introduction.html#//apple_ref/doc/uid/TP40001051-CH1-SW878)。这是一个预览版，并且默认没有开启，它会明显地减少项目的编译时间。如果要启用他，要从 Xcode 的菜单进入 `Workspace` 或 `Project Settings` 页面，就可以切换到新的编译系统。

![](https://user-gold-cdn.xitu.io/2017/12/25/1608bbde531e79a8?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

更多资源：

- [Faster Swift Builds with the New Xcode Build System](https://github.com/quellish/XcodeNewBuildSystem)

## 启用 Swift 的 Concurrent Swift Build Tasks

Xcode 9.2 版本加入了一个对 Swift 项目的 `concurrent build tasks` 实验性支持。对于一些项目，可以大大减少编译器编译时间。另外值得注意的是，如果你启用这个特性，Xcode 会增加内存的开销。

如果要启用这个特性，退出 Xcode，然后在 Terminal 窗口输入：

````bash
$ defaults write com.apple.dt.Xcode BuildSystemScheduleInherentlyParallelCommandsExclusively -bool YES
````

现在，测试一下项目的编译时间是否减少了。如果你要禁用这个特性，只需要把上面的 `YES` 改成 `NO` 再执行一次即可。

## 在 Xcode 显示编译时间

最后，如果你想知道项目的编译时间是否在减少，你应该通过以下命令来打开：

````bash
$ defaults write com.apple.dt.Xcode ShowBuildOperationDuration -bool YES
````

重新编译一次项目，现在你会看到整个项目的编译总时间：

![](https://github.com/fastred/Optimizing-Swift-Build-Times/raw/master/assets/time@2x.png)

建议比较一下相同条件下的编译时间：

1. 退出 Xcode
2. 清理 Derived Data (`$ rm -rf ~/Library/Developer/Xcode/DerivedData`).
3. 用 Xcode 打开你的项目
4. 立刻 build 你的项目，或者等 indexing 工作完成后再 build

另外，你也可以通过命令行统计编译时间： 

````bash
$ time xcodebuild other params
````

更多资源：

- [How to enable build timing in Xcode? - Stack Overflow](https://stackoverflow.com/a/2801156/1990236)