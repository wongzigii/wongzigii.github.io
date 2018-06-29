title: Pod install vs. Pod update
date: 2016-04-23 15:19:53
tags: CocoaPods
---

## Introduction

对于很多经常使用 CocoaPods 开发者，有一个比较困惑的问题是：什么时候用 `pod install` 以及 `pod update` ?

**TL;DR**: 

- 使用 `pod install` 安装 *新* 的 pod，即使你之前已经通过 `pod install` 安装（或移除过）项目中的 pod
- 当且仅当你需要将 pods 升级到一个更新的版本的时候，使用 `pod update [PODNAME]` 

### pod install

这个命令经常用于为新项目安装 pods，以及通过修改 Podfile 来增加，升级或移除 pod 的操作。

- 每次当你运行 `pod install` 这个命令的时候，CocoaPods 会下载并安装你在 Podfile 中指定的 pods，并将这次安装的 pod 以及 pod 对应的版本写入 `Podfile.lock` 这个文件，这个文件的作用用于跟踪并锁定已安装 pod 的版本。

- 当你运行 `pod install` 时，CocoaPods 仅会对 `Podfile.lock` 文件里没有的 pod 安装。

  - 对于那些已经在 Podfile.lock 里的 pod，CocoaPods 会下载文件中指定的版本号，而不会尝试检查是否有新的版本
  - 对于那些还不在 Podfile.lock 里的 pod，CocoaPods 会尝试搜索你在 Podfile.lock 中指定的 版本 (`pod 'MyPod', '~> 1.2'`)


### pod outdated

当你运行 `pod outdated`，CocoaPods 会将所有比 Podfile.lock 中指定的存在更新的版本的 pods 罗列出来。
这意味着如果你对 *这些 pod* 运行 `pod update PODNAME`，他们会被升级到更新的版本。
（注：仅会升级到仍满足 `Podfile.lock` 中版本的最新版本，举个例子，你在 Podfile.lock 中指定 `pod 'MyPod', '~>1.2'`，那仅会升级到 `1.2` 的最新版本， 不会升级到 `1.3` 或 `1.4` ）

### pod update

当你 *直接* 运行 `pod update`，CocoaPods 会尝试将 `PODNAME` 更新到**最新**版本。如上面所说的，仍会满足 Podfile.lock 的限制。

## Intended usage

- `pod update PODNAME` 只会更新一个特定的 pod，相反，`pod install` 不会尝试去对已经安装过的 pod 更新。

- 当你在 Podfile 中增加了一个新的 pod 的时候，你应该用 `pod install`，而不是 `pod update`。

- 使用 `pod update` 当且仅当你需要更新 pod 的版本的时候。

## Commit your Podfile.lock

*尽可能将 Podfile.lcok 文件提交到项目的目录中，否则，由于 Podfile.lock 锁定的 pod 的版本问题，可能会使项目构建失败。*

## Scenario Example

例子略过，详情请看[这里](https://guides.cocoapods.org/using/pod-install-vs-update.html#scenario-example)

## Using exact versions in the Podfile is not enough

你可能会想：如果我在 Podfile 中特定指定一个具体的版本，像 `pod 'A', '1.0.0'`这样，这样就保证了每个人都会用到相同版本的 pod 了。

即使接着使用 `pod update` 来增加新的 pod，那也不会导致其他的 pod 升级。（因为你已经明确指定一个版本号了）

事实真的是这样吗？不是的。

一个典型的例子：

如果 pod `A` 有一个依赖叫 pod `A2` -- 在 `A.podspec` 声明为 `dependency 'A2', '~> 3.0'`，在这个情况下，你在 Podfile 中指定 A 的版本如：`pod 'A', '1.0.0'`，User1 和 User2 确实使用的是同一个版本的 pod `A`。 但是：

- User1 可能会将 `A2` 升级到 `3.4`，因为它可能是那时候 `A2` 的最新版本。
- 当新加入项目的 User2 运行 `pod install`，它也可能会安装 `A2` 的 `3.5` 版本，因为 `A2` 的维护者可能也发布了新的版本。

这也就是 `Podfile.lock` 以及合理使用 `pod install` vs. `pod update` 的作用，来确保项目组里的每个人在各自的电脑上都使用相同的 pod 版本。


