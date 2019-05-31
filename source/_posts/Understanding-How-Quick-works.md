title: Understanding How Quick Works
date: 2016-09-06 23:54:14
tags: XCTest
---

## 前言

[Quick](https://github.com/Quick/Quick) 是一个基于 XCTest 框架的测试框架。

## 目录

![](../img/files.png)

## Concept

- 关于 **XCTestCase**, **XCTestSuite** 以及 **XCTest** 的关系，请参考 [StackOverflow 上面的一个回答](http://stackoverflow.com/a/39288791/4234171)。
- Example
- ExampleGroup
- ExampleMetadata

## World

主要用于自定义测试的 Configuration。
World 是一个单例，它主要功能是用来维护 `QuickSpec` 对应的根 example group 以及 example。

## QuickSpec 的实现

你写测试时所有的 TestCase 都要继承于这个 QuickSepc 类，我们前面说过，Quick 是一个基于 XCTest 的测试框架，并且实现了自己的语法。你可能会有疑问，XCTest 是如何识别 Quick 的语法的？其实这个类就是关键所在，因此只有这样你的测试代码才能够被 XCTest 框架识别出来。 `QuickSepc` 继承自 `XCTestCase`，并依赖 Objective-C 的 runtime 特性来实现。

### 重写 initialize

当每个 spec 初始化时，`[NSObject initialize]` 方法会被调用。QuickSpec 通过重写 `initialize` 类方法的方式调用 `[QuickSpec spec]` 来初始化一个 spec 实例。*其中会创建维护 example group 的堆栈，并使用 Quick.World 单例注册 example。*

````objc
+ (void)initialize {
    [QuickConfiguration initialize];
    World *world = [World sharedWorld];
    [world performWithCurrentExampleGroup:[world rootExampleGroupForSpecClass:self] closure:^{
        QuickSpec *spec = [self new];
        @try {
            [spec spec];
        }
        @catch (NSException *exception) {
            [NSException raise:NSInternalInconsistencyException
                        format:@"An exception occurred when building Quick's example groups.\n"
             @"Some possible reasons this might happen include:\n\n"
             @"- An 'expect(...).to' expectation was evaluated outside of "
             @"an 'it', 'context', or 'describe' block\n"
             @"- 'sharedExamples' was called twice with the same name\n"
             @"- 'itBehavesLike' was called with a name that is not registered as a shared example\n\n"
             @"Here's the original exception: '%@', reason: '%@', userInfo: '%@'",
             exception.name, exception.reason, exception.userInfo];
        }
        [self testInvocations];
    }];
}
````

### 自定义 spec

写测试的时候，你需要重写 spec 方法来自定义一组 example group 或者 example。看代码我们知道，World 单例从根 example group 开始，顺着结构树往下遍历，并对每一个 example 执行 clourse。其中，clourse 包括几个部分：

1. 新建一个 QuickSpec 对象
2. try catch 获取我们的测试用例时可能抛出的异常
3. 向这个对象发送 `testInvocations` 消息

### 重写 testInvocation 方法

````objc
+ (NSArray *)testInvocations {
    NSArray *examples = [[World sharedWorld] examplesForSpecClass:[self class]];
    NSMutableArray *invocations = [NSMutableArray arrayWithCapacity:[examples count]];
    
    NSMutableSet<NSString*> *selectorNames = [NSMutableSet set];
    
    for (Example *example in examples) {
        SEL selector = [self addInstanceMethodForExample:example classSelectorNames:selectorNames];
        NSInvocation *invocation = [self invocationForInstanceMethodWithSelector:selector
                                                                         example:example];
        [invocations addObject:invocation];
    }

    return invocations;
}

- (void)setInvocation:(NSInvocation *)invocation {
    self.example = objc_getAssociatedObject(invocation, QCKExampleKey);
    [super setInvocation:invocation];
}
````

上面的代码做了以下两件事：

1. 通过 World 单例保存的映射获取我们已经定义的 examples，包括 configuration 里面设置的过滤规则
2. 遍历 examples，为每个对应的 example：通过[方法签名](https://developer.apple.com/library/mac/documentation/Cocoa/Reference/Foundation/Classes/NSMethodSignature_Class/) 生成 [NSInvocation](https://developer.apple.com/library/mac/documentation/Cocoa/Reference/Foundation/Classes/NSInvocation_Class/) 实现 XCTestCase 的消息转发（即执行测试方法），然后 hook 进 Invocation 的 setter 方法，这样就能对 current example 保持引用。*从而测试失败时可以获取 current example 的相关信息如名字，行数等等*

> 未完待续