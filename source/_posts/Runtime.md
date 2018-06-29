title: "Runtime"
date: 2014-12-11 09:48:15
tags:
---
## What is Runtime

一门动态语言，有一些决定工作从编译连接推迟到运行时才被执行。只有编译器是不够的，还需要一个运行时系统 (runtime system) 来执行编译后的代码。这就是 Runtime 存在的意义。

## Why not function calling but message

当你向 receiver 发送一个消息 `[receiver message]`， 编译器会将其编译为 `objc_msgSend(receiver, selector)`, 如果消息带有参数，如 `[receiver message:arg1]`， 则会编译为 `objc_msgSend(receiver, selector, arg1)`。如果 `receiver` 找到对应的 `selector`，那么相当于执行了 `receiver` 的`message` 方法，否则，消息或是被转发，或是临时添加对应的实现内容，或者 crash。

看 `message.h` 是怎么定义的：

````objc
/** 
 * Sends a message with a simple return value to an instance of a class.
 * 
 * @param self A pointer to the instance of the class that is to receive the message.
 * @param op The selector of the method that handles the message.
 * @param ... 
 *   A variable argument list containing the arguments to the method.
 * 
 * @return The return value of the method.
 * 
 * @note When it encounters a method call, the compiler generates a call to one of the
 *  functions \c objc_msgSend, \c objc_msgSend_stret, \c objc_msgSendSuper, or \c objc_msgSendSuper_stret.
 *  Messages sent to an object’s superclass (using the \c super keyword) are sent using \c objc_msgSendSuper; 
 *  other messages are sent using \c objc_msgSend. Methods that have data structures as return values
 *  are sent using \c objc_msgSendSuper_stret and \c objc_msgSend_stret.
 */
OBJC_EXPORT id objc_msgSend(id self, SEL op, ...)
    __OSX_AVAILABLE_STARTING(__MAC_10_0, __IPHONE_2_0);
````

其中第一个参数类型为 `id`， 第二个参数类型为 `SEL`。

## id

````objc
/// A pointer to an instance of a class.
typedef struct objc_object *id;
````

可以知道，`id` 是一个指向类实例的指针

### objc_object

````objc
/// An opaque type that represents an Objective-C class.
typedef struct objc_class *Class;

/// Represents an instance of a class.
struct objc_object {
    Class isa  OBJC_ISA_AVAILABILITY;
};
````

而 `objc_object` 又是一个结构体，包含一个指向 Class 类型的 指针 `isa`

### objc_class

````objc
struct objc_class {
    Class isa  OBJC_ISA_AVAILABILITY;

#if !__OBJC2__
    Class super_class                                        OBJC2_UNAVAILABLE;
    const char *name                                         OBJC2_UNAVAILABLE;
    long version                                             OBJC2_UNAVAILABLE;
    long info                                                OBJC2_UNAVAILABLE;
    long instance_size                                       OBJC2_UNAVAILABLE;
    struct objc_ivar_list *ivars                             OBJC2_UNAVAILABLE;
    struct objc_method_list **methodLists                    OBJC2_UNAVAILABLE;
    struct objc_cache *cache                                 OBJC2_UNAVAILABLE;
    struct objc_protocol_list *protocols                     OBJC2_UNAVAILABLE;
#endif

} OBJC2_UNAVAILABLE;
/* Use `Class` instead of `struct objc_class *` */

#endif
````

其中 `isa` 指向所属的 Class， `super_class` 指向父类。

而在开源的 *objc-runtime-new.h* 中我们发现在 objc_class 结构体的实现中，objc_class 继承自 objc_object，而且结构体第一项为

````objc
Class superclass
````

我们称之为 `Meta Class`，又由于 `isa` 指向所属的 Class，换句话说，`isa` 指向 `Meta Class`。

## SEL

`objc.h` 中定义了这种数据类型：

````objc
/// An opaque type that represents a method selector.
typedef struct objc_selector *SEL;
````

在 `runtime.h` 中， `/* Working with Selectors */`下面有这样四个方法：

- OBJC_EXPORT const char *sel_getName(SEL sel)
    ____OSX_AVAILABLE_STARTING(__MAC_10_0, __IPHONE_2_0);
    
返回一个 char 类型，即 Selector 的名字

- OBJC_EXPORT SEL sel_getUid(const char *str)
    ____OSX_AVAILABLE_STARTING(__MAC_10_0, __IPHONE_2_0);

通过传入一个 UID，返回一个特定的 SEL
    
- OBJC_EXPORT SEL sel_registerName(const char *str)
    ____OSX_AVAILABLE_STARTING(__MAC_10_0, __IPHONE_2_0);

通过一个 char 类型注册一个 SEL

- OBJC_EXPORT BOOL sel_isEqual(SEL lhs, SEL rhs) 
     ____OSX_AVAILABLE_STARTING(__MAC_10_5, __IPHONE_2_0);
     
传入两个 SEL 判断他们是否相等

可以看到，他们都是与 Selector 相关的 Runtime 函数。

## Message Forwarding

`forwardInvocation:` 方法通常用来将消息转发给另一个对象。
通过实现 `forwardInvocation:` 方法赋予消息一个默认的响应者，并且可以避免一些错误的产生。

To see the scope and intent of forwarding, imagine the following scenarios: Suppose, first, that you’re designing an object that can respond to a message called negotiate, and you want its response to include the response of another kind of object. You could accomplish this easily by passing a negotiate message to the other object somewhere in the body of the negotiate method you implement.

Take this a step further, and suppose that you want your object’s response to a negotiate message to be exactly the response implemented in another class. One way to accomplish this would be to make your class inherit the method from the other class. However, it might not be possible to arrange things this way. There may be good reasons why your class and the class that implements negotiate are in different branches of the inheritance hierarchy.

Even if your class can’t inherit the negotiate method, you can still “borrow” it by implementing a version of the method that simply passes the message on to an instance of the other class:

````objc
- (id)negotiate
{
    if ( [someOtherObject respondsTo:@selector(negotiate)] )
        return [someOtherObject negotiate];
    return self;
}
````

换句话说，如果这个对象不能响应 `negotiate` 这个消息，你可以把它**扔**给其他类然后让它决定是否响应。

When an object can’t respond to a message because it doesn’t have a method matching the selector in the message, the runtime system informs the object by sending it a forwardInvocation: message. Every object inherits a forwardInvocation: method from the NSObject class. However, NSObject’s version of the method simply invokes doesNotRecognizeSelector:. By overriding NSObject’s version and implementing your own, you can take advantage of the opportunity that the forwardInvocation: message provides to forward messages to other objects.

当对象由于没有与选择器相匹配的方法时，runtime system 会向其发送一个 `forwardInvocation:` 消息。每一个对象由于都继承自 NSObject 类从而继承了 NSObject 的 `forwardInvocation:` 方法。但是，NSObject 只是简单地调用了 `doesNotRecognizeSelector:` 这个方法。你还需要重载这个方法然后实现自己的 `forwardInvocation:` 方法从而把消息转发给其他对象。

这个 `forwardInvocation:` 方法需要做两样的事情：
- 决定消息应该被转发到哪
- 把原始的参数一并转发出去

````objc
- (void)forwardInvocation:(NSInvocation *)anInvocation
{
    if ([someOtherObject respondsToSelector:
            [anInvocation selector]])
        [anInvocation invokeWithTarget:someOtherObject];
    else
        [super forwardInvocation:anInvocation];
}
````

消息返回的值会被传递到最原始的发送者，包括 `ids`， `structures` 还有 `double-precision floating-point numbers`。 

这个 `forwardInvocation` 方法可以被看成是 unrecognized messages 的转发中心，或者是一个转移枢纽，将所有消息都传递到同一个地方。它可以将消息传递到另一个接受者，也可以把那些没有响应者的**消化**掉，来避免 no response。这这机制让互相关联的对象在设计上提供了无限的可能。

## Forwarding and Multiple Inheritance

![](http://ww1.sinaimg.cn/large/8c7526ebgw1f1w5q2dxuyg20ag06pdfq.gif)
 
首先，`Warrior` 和 `Diplomat` 没有继承关系，如果你向 `Warrior` 发送一个 `negotiate` 消息， `Warrior` 不能响应这个方法，并把它转发给 `Diplomat` 类，这看起来就像 `Warrior` 能响应 `negotiate` 消息一样。（尽管实际上是 Diplomat 响应的）

这个例子就像 `Warrior` 类是继承自 `Diplomat` 的一样。

消息转发给我们提供了像多重继承一样的特性。但是，它们之间仍然有相当大的区别：多重继承将不同的东西组合到一个臃肿的对象中，而消息转发，则将不同的东西细分到各自所负责的对象中，这使得对象之间的关系由于消息转发机制而透明清晰。

## Surrogate Objects

Forwarding not only mimics multiple inheritance, it also makes it possible to develop lightweight objects that represent or “cover” more substantial objects. The surrogate stands in for the other object and funnels messages to it.

The proxy discussed in “Remote Messaging” in The Objective-C Programming Language is such a surrogate. A proxy takes care of the administrative details of forwarding messages to a remote receiver, making sure argument values are copied and retrieved across the connection, and so on. But it doesn’t attempt to do much else; it doesn’t duplicate the functionality of the remote object but simply gives the remote object a local address, a place where it can receive messages in another application.

Other kinds of surrogate objects are also possible. Suppose, for example, that you have an object that manipulates a lot of data—perhaps it creates a complicated image or reads the contents of a file on disk. Setting this object up could be time-consuming, so you prefer to do it lazily—when it’s really needed or when system resources are temporarily idle. At the same time, you need at least a placeholder for this object in order for the other objects in the application to function properly.

In this circumstance, you could initially create, not the full-fledged object, but a lightweight surrogate for it. This object could do some things on its own, such as answer questions about the data, but mostly it would just hold a place for the larger object and, when the time came, forward messages to it. When the surrogate’s forwardInvocation: method first receives a message destined for the other object, it would ensure that the object existed and would create it if it didn’t. All messages for the larger object go through the surrogate, so, as far as the rest of the program is concerned, the surrogate and the larger object would be the same.

## Forwarding and Inheritance

尽管消息转发很像继承，但是 `NSObject` 还是不会将它们混淆的。看这个例子：

````objc
if ( [aWarrior respondsToSelector:@selector(negotiate)] )
	...
///return YES  (if Warrior inherit from Diplomat)
///return NO   (if Warrior just forward message to Diplomat as above)
````

如果你确实想瞒天过海的话，你需要重新实现 `respondsToSelector:` 和 `isKindOfClass:` 方法像下面那样：

````objc
- (BOOL)respondsToSelector:(SEL)aSelector
{
    if ( [super respondsToSelector:aSelector] )
        return YES;
    else {
        /* Here, test whether the aSelector message can     *
         * be forwarded to another object and whether that  *
         * object can respond to it. Return YES if it can.  */
    }
    return NO;
}
````

除此之外，`instancesRespondToSelector:` 方法也应该实现转发算法。如果遵从协议，那么 `conformsToProtocol:` 方法也应该做同样的事情。同样的，如果一个对象转发它接受的任何远程消息，它得给出一个 `methodSignatureForSelector:` 来返回准确的方法描述，这个方法会最终响应被转发的消息。比如一个对象能给它的替代者对象转发消息，它需要像下面这样实现 `methodSignatureForSelector:`：

````objc
- (NSMethodSignature*)methodSignatureForSelector:(SEL)selector
{
    NSMethodSignature* signature = [super methodSignatureForSelector:selector];
    if (!signature) {
       signature = [surrogate methodSignatureForSelector:selector];
    }
    return signature;
}
````

考虑将这些转发消息的代码单独封装起来，当你需要使用它们的时候，直接调用它们就可以了。

> Note:  This is an advanced technique, suitable only for situations where no other solution is possible. It is not intended as a replacement for inheritance. If you must make use of this technique, make sure you fully understand the behavior of the class doing the forwarding and the class you’re forwarding to.

最后，慎用黑魔法。

