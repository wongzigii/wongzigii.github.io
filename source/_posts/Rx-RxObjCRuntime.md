title: "扒一扒 RxSwift 里面的 RxObjCRuntime "
date: 2018-03-27 12:20:15
tags: rxswift
---

## 0x00

在使用 RxSwift 或者 RxCocoa 的过程，总是有疑惑，这个框架是怎么将 sentMessage 的方法传递出来给 Observer 监听的呢？其实不外乎通过 Method Swizzling 或者其他 KVO 之类的方法，在消息转发 hook 一层自己的逻辑。

怀着疑问，我今天想探究一下 RxCocoa 里面的 Runtime 机制，RxSwift 的方法监听 feature 源自这个 2015 年的 issue - [Is there any way to observe method calls? · Issue #181 · ReactiveX/RxSwift · GitHub](https://github.com/ReactiveX/RxSwift/issues/181)。

我们直接到 RxCocoa/Runtime/_RXObjCRuntime 这个文件看代码：

首先第一行代码是一个宏开关：

`#if define !DISABLE_SWIZZLING`

让你可以从全局控制是否开启 method swizzling.

接下来是另一个宏

`#define HIDDEN_ARGUMENT_COUNT 2` 

在 Objective-C 的消息里面，会自带两个隐藏的参数：

1. id 类型的 self (这个很好理解吧，任意对象都可以发消息，id 泛指任意对象) 
2. SEL 类型的 _cmd (其实就是方法名的选择器，你可以通过 NSStringFromSelector(_cmd) 取到方法名)。

所以，其实 Objective-C 的方法就是最少自带两个参数的 C 函数。

## 0x01

代码比较多，我主要选几个重要方法作为入口开始看：

```
// This is the main entry point for observing messages sent to arbitrary objects.
-(IMP __nullable)ensurePrepared:(id __nonnull)target forObserving:(SEL __nonnull)selector error:(NSErrorParam)error
```

ensurePrepared 这个方法从方法名字看是为了真正的 swizzling 做准备工作。

首先它通过 `class_getInstanceMethod` 拿到某个具体的方法，如果这个方法 Method 为空，则抛出一个 SelectorNotImplemented 的异常。
除此之外，还对一些敏感的方法名做了过滤，如：`class`，`forwardingTargetForSelector`，`methodSignatureForSelector` 及 `respondsToSelector` 等方法，如果你针对这些方法做 swizzling，就会抛一个 ObservingPerformanceSensitiveMessages 的异常。

接着往下走，如果这个 selector 是 dealloc 方法，原始的方法将会被 swizzled 掉。

````
// For `dealloc` message, original implementation will be swizzled.
// This is a special case because observing `dealloc` message is performed when `observeWeakly` is used.
//
// Some toll free bridged classes don't handle `object_setClass` well and cause crashes.
//
// To make `deallocating` as robust as possible, original implementation will be replaced.
if (selector == deallocSelector) { 
   ....
}
````

> 关于 toll-free-bridged 我想多说几句：

> 1. 在 Objective-C 中，Foundation 和 CoreFoundation 之间类型是可以互相 toll-free-bridged 的。比如 NSString 和 CFStringRef，NSArray 和 CFArrayRef 等，他们之间其实是可以互相转换的。
2. 在 Swift 中，由于 Swift 支持跨平台可以运行在 Linux 系统上，因此  Swift 上的类型 bridging 要比 Objective-C 更复杂。

> 举个简单的例子，在 iOS/macOS 平台，将 Array 转换成 NSArray 非常简单：
>
````
let myArray: NSArray = ["foo", "bar", "baz"]
````
> 而在 Linux 平台，你需要另外用 CoreFoundation 的 [`_bridgeToObjectiveC` 方法](https://github.com/apple/swift-corelibs-foundation/blob/master/Foundation/Array.swift#L13) 。
>
````
let myArray: NSArray = ["foo", "bar", "baz"]._bridgeToObjectiveC()
````
> 更多关于类型的 bridging 可以查看 `swift-corelibs-foundation/bridging.swift`

除了以上这几种方法之外，就是我们正常调用的方法了。

````
Class __nullable swizzlingImplementorClass = [self prepareTargetClassForObserving:target error:error];
if (swizzlingImplementorClass == nil) {
    return nil;
}
````

首先确保这个类不为空，

````
NSString *methodEncoding = RX_method_encoding(instanceMethod);
````

然后获取这个实例方法的 encode 类型，

这里定义了一个 `RXInterceptWithOptimizedObserver` 的变量，我们再细看，其实它是一个返回 Bool 类型的 Block，接受的参数就是我们上面判断的变量值，结合来看的话，就是能拿到这个方法的 method encoding 的话，就返回 true，反之为 false。

````
typedef BOOL (^RXInterceptWithOptimizedObserver)(RXObjCRuntime * __nonnull self, Class __nonnull class, SEL __nonnull selector, NSErrorParam error);
````

接着通过 `RX_method_has_supported_return_type` 检查这个方法的返回类型：

`````
if (!RX_method_has_supported_return_type(instanceMethod)) {
    RX_THROW_ERROR([NSError errorWithDomain:RXObjCRuntimeErrorDomain
                                       code:RXObjCRuntimeErrorObservingMessagesWithUnsupportedReturnType
                                   userInfo:nil], nil);
}
`````

````
static supported_type_t supported_types[] = {
    { .encoding = @encode(void)},
    { .encoding = @encode(id)},
    { .encoding = @encode(Class)},
    { .encoding = @encode(void (^)(void))},
    { .encoding = @encode(char)},
    { .encoding = @encode(short)},
    { .encoding = @encode(int)},
    { .encoding = @encode(long)},
    { .encoding = @encode(long long)},
    { .encoding = @encode(unsigned char)},
    { .encoding = @encode(unsigned short)},
    { .encoding = @encode(unsigned int)},
    { .encoding = @encode(unsigned long)},
    { .encoding = @encode(unsigned long long)},
    { .encoding = @encode(float)},
    { .encoding = @encode(double)},
    { .encoding = @encode(BOOL)},
    { .encoding = @encode(const char*)},
};
````
如果方法返回的类型不包含在 supported_types 数组里的话就直接抛异常。

最后，

````
IMP interceptorIMPForSelector = [self interceptorImplementationForSelector:selector forClass:swizzlingImplementorClass];
````

`interceptorImplementationForSelector` 这个方法会接受上面经过层层检查的 `selector` 变量 和 `swizzlingImplementorClass` 变量，并返回一个对 selector 进行 swizzling 的 IMP。

## 0x02

好了，ensurePrepared 方法看完了，接下来用 <kbd>CTRL</kbd>+<kbd>F</kbd> 搜索调用这个方法的地方，有且仅有一处 -- `RX_ensure_observing(id __nonnull target, SEL __nonnull selector, NSErrorParam error)`

一个叫 `RX_ensure_observing` 的方法，看来这个才是真正的入口。

这个方法很有意思，执行上面的 `interceptorImplementationForSelector` 方法前会进行加锁操作以确保原子性。

````
@synchronized([target class]) { ... }


// The only other resource that all other swizzling libraries have in common without introducing external
// dependencies is class object.
//
// It is polite to try to synchronize it in hope other unknown entities will also attempt to do so.
// It's like trying to figure out how to communicate with aliens without actually communicating,
// save for the fact that aliens are people, programmers, authors of swizzling libraries.
````

> 有时候，我们不知道是什么东西在对代码做了修改，就像你想找到与外星人沟通的方法一样，你不知道外星人到底是普通的正常人，一个程序员，还是那些写 swizzling library 的作者。

所以应该是担心其他框架也对目标实现 swizzling 造成难以 debug 所以才这样做吧。

接着全文搜索找到只有 `NSObject+Rx` 这个文件里 `registerMessageInterceptor` 调用这个 `RX_ensure_observing` 方法，那就不难看出这个是用来拓展 RxSwift runtime 的特性。

我们熟悉的 `sentMessage`, `methodInvoked` 和 `deallocating` 均调用了 `registerMessageInterceptor`。

## 0x03

总结一下，我们上面顺藤摸瓜地从 _RXObjCRuntime 这个文件通过注释找到一个貌似是入口的方法，其实这个文件里还有很多黑科技。

比如用 

````
BOOL isThisTollFreeFoundationClass = CFGetTypeID((CFTypeRef)target) != defaultTypeID;
````

来判断我上面提过的 toll-free-bridge，

还有它是怎么统计 `number_of_dynamic_subclasses`, `number_of_forwarding_enabled_classes`, `number_of_intercepting_classes`, `RX_number_of_forwarded_methods` 和 `RX_number_of_swizzled_methods` 等等。

总之，你会惊讶地发现 runtime 原来还可以这么玩。

最后，have fun with Rx!