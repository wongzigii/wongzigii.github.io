title: "MetaClass"
date: 2015-04-17 09:51:43
tags: runtime
---

本文翻译整理自：[From Cocoa with Love](http://www.cocoawithlove.com/2010/01/what-is-meta-class-in-objective-c.html)

在这篇文章中，我们会看到一个在 Objective-C 中很陌生的概念 -- meta-class。Objective-C 里面每个类都有它自己的 meta-class，但是你很少会注意到你使用了它。我们在开头会学习怎样在 runtime 创建一个类，通过创建的 "class pair"，我会解释什么是 meta-class，然后深入研究它对于 Objective-C 中对象和类的意义。

### 在 Runtime 创建一个类

下面的代码会在 runtime 创建一个 `NSError` 的子类，并在其中增加一个方法：

````objc
Class newClass =
    objc_allocateClassPair([NSError class], "RuntimeErrorSubclass", 0);
class_addMethod(newClass, @selector(report), (IMP)ReportFunction, "v@:");
objc_registerClassPair(newClass);
````

这个方法使用一个叫 `ReportFunction` 的函数作为它的实现，它的定义如下：

````objc
void ReportFunction(id self, SEL _cmd)
{
    NSLog(@"This object is %p.", self);
    NSLog(@"Class is %@, and super is %@.", [self class], [self superclass]);
 
    Class currentClass = [self class];
    for (int i = 1; i < 5; i++)
    {
        NSLog(@"Following the isa pointer %d times gives %p", i, currentClass);
        currentClass = object_getClass(currentClass);
    }
 
    NSLog(@"NSObject's class is %p", [NSObject class]);
    NSLog(@"NSObject's meta class is %p", object_getClass([NSObject class]));
}
````

表面上看这些都很简单。在 runtime 创建一个类只需三个步骤：
1. 为 `class pair` 开辟内存，（使用 objc_allocateClassPair）。
2. 添加方法或成员变量到这个类中（通过 class_addMethod 增加一个方法）。
3. 注册这个类，以便它能使用(使用 objc_registerClassPair)。

然而，有一个很关键的问题是：`class pair` 是什么东西？`objc_allocateClassPair` 函数只返回一个值：the class。那另外一半呢？

我相信你已经猜到了，另一半就是 meta-class，在解释这个东西是什么和你为什么需要这个东西之前，还需要先了解一下对象和类的背景知识。

### 怎么样的数据类型才能成为对象？

每个对象都有它自己的类。这是面向对象的基本概念，但是在 Objective-C 中，数据结构都有它自己的类。含有一个指针且该指针可以正确指向类的数据结构，都可以称为对象。

在 Objective-C 中，对象的类是由一个叫 `isa` 的指针决定的，`isa` 指针指向对象所属的类（Class）。

> 译者注：实际上在 Runtime 这篇文章中我们已经一窥 Objective-C 中最基本的定义：

````objc
typedef struct objc_object {
    Class isa;
} *id;
````

所有以指针开始并指向 Class 结构体的数据结构都可以看成是 `objc_object`。

而对象在 Objective-C 中最重要的特点就是你可以发送消息给它们：

````objc
[@"stringValue"
    writeToFile:@"/file.txt" atomically:YES encoding:NSUTF8StringEncoding error:NULL];
````

你向对象发送一个消息，runtime 会顺着对象的 isa 指针来获得对象所属的 Class。这个 Class 包含了很多定义的的 Method，还包含一个指向 superclass 的指针，来寻找继承自父类的方法。当你发送消息的时候，首先，runtime 会在所属 Class 的 Method 中查找方法，如果找不到，就会去 superclass 中查找父类的方法。如果找到了，runtime 会根据这个方法调用实现函数（IMP）。

### 什么是 meta-class？

现在，正如你所了解那样，一个 Class 在 Objective-C 中也算一个对象。这意味着你可以向 Class 发送消息。

````objc
NSStringEncoding defaultStringEncoding = [NSString defaultStringEncoding];
````

在这个例子中，`defaultStringEncoding` 类方法被发送到 `NSString` 类。

由于 Objective-C 里所有 Class 都是一个对象，因此，Class 结构体的第一项必须是一个指向 isa 的指针，从而符合上面 objc_object 结构体的定义，而第二项必须是一个指向 superclass （或者 nil，对于基本的类）的指针。

一个 Class 的定义会因你所使用的 runtime 的版本而异，但有一点不变的是，它们第一项都是一个 isa 指针，然后紧接着是 superclass 指针。

````objc
typedef struct objc_class *Class;
struct objc_class {
    Class isa;
    Class super_class;
    /* followed by runtime specific details... */
};
````

为了调用 Class 里的方法（这里指的是类方法），isa 指针必须指向一个 Class 结构体，这个 Class 结构体必须包含那些我们可以调用的方法列表。

>注：这就像上面提到过的，对象为了调用 Class 里的实例方法，Class 里必须包含这些自定义的方法，只不过这里不是对象，而是 Class，因为 Class 也是一个对象

这就要引入一个关于 meta-class 的概念：meta-class 是一个 Class 对象的类。

简单说就是：

- 当你向对象发送消息，是在这个对象的类的方法列表中寻找。（实例方法）
- 当你向类（Class）发送消息，是在这个 Class 的 meta-class 的方法列表中寻找。（类方法）

meta-class 是必不可少的，因为它储存了这个类的类方法。由于每个类都有独一无二的类方法，所以每个类都有独一无二的 meta-class。

### meta-class 的类是什么?

meta-class，像前面的 Class，也是一个对象。这意味着你也可以向它发送消息。同样，它也必须要有一个类。

所有的 meta-class 使用 NSObject 的 meta-class 作为它的类，因此 isa 指针都指向 NSObject 的 meta-class。那 NSObject 的 meta-class 的 isa 指针指向哪呢？ 它自己。

说的更拗口一点就是，根元类把它自己的基类设置成了super_class。

在这样的继承体系下，所有实例、类以及元类（meta class）都继承自一个基类。

这意味着对于继承于 NSObject 的所有实例、类和元类，他们可以使用 NSObject 的所有实例方法，类和元类可以使用 NSObject 的所有类方法。

![](http://ww4.sinaimg.cn/large/8c7526ebgw1f84s9nt6pzj21fv1ia45g.jpg)

### 实验

为了验证，让我们看看我在文章开始写的 ReportFunction 函数的输出。这个函数的目的是跟随 isa 指针并打印出它的路径。

为了运行 ReportFunction，我们需要创建一个动态实例来创建类调用 report 方法。

````objc
id instanceOfNewClass =
    [[newClass alloc] initWithDomain:@"someDomain" code:0 userInfo:nil];
[instanceOfNewClass performSelector:@selector(report)];
[instanceOfNewClass release];
````

这里没有声明 report 方法，但我使用 performSelector: 调用它，所以编译器不会给出警告。
然后 ReportFunction 函数会沿着 isa 进行检索，来告诉我们 class，meta-class 以及meta-class 的 class 是什么样的情况：

> 如何获取对象的类：在文章的开头部分，ReportFunction 函数通过 object_getClass 方法来跟踪 isa 指针，因为 isa 指针是类的保护成员（你不能直接接收其他对象的 isa 指针）。ReportFunction 不使用类方法，因为在类对象里调用类方法不能返回元类，它会再次返回这个类（因此 [NSString class] 会返回 NSString 类而不是 NSString 的 meta-class）。

输出（省略了 NSLog 前缀）：

````objc
This object is 0x10010c810.
Class is RuntimeErrorSubclass, and super is NSError.
Following the isa pointer 1 times gives 0x10010c600
Following the isa pointer 2 times gives 0x10010c630
Following the isa pointer 3 times gives 0x7fff71038480
Following the isa pointer 4 times gives 0x7fff71038480
NSObject's class is 0x7fff710384a8
NSObject's meta class is 0x7fff71038480
````

观察 isa 经过的地址的值：

- 对象的地址是 0x10010c810。
- 类的地址是 0x10010c600。
- 元类的地址是 0x10010c630。
- 根元类（ NSObject 的元类）的地址是 0x7fff71038480。
- NSObject 元类的类是它本身。

这些地址的值并不重要，重要的是它们说明了文中讨论的从类到 meta-class 到 NSObject 的 meta-class 的整个流程。

### 尾声

元类是 Class 对象的类。每个类都有自己独一无二的 meta-class（每个类都有自己独一无二的方法列表）。这意味着所有的类对象都不同。

元类总是会确保类对象和基类的所有实例和类方法。对于从 `NSObject` 继承下来的类，这意味着所有的 `NSObject` 实例和 `Protocol` 方法在所有的类（和 meta-class ）中都可以使用。

所有的 meta-class 使用基类的 meta-class 作为自己的基类，对于顶层基类的 meta-class 也是一样，只是它指向自己而已。