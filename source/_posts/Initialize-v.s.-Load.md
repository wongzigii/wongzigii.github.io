title: "Initialize v.s. Load"
date: 2015-02-01 17:29:08
tags:
---
[NSObject Class Reference](http://developer.apple.com/library/ios/#documentation/Cocoa/Reference/Foundation/Classes/NSObject_Class/Reference/Reference.html)

在一个类被使用之前，我们可以利用这两个方法对类做一些预处理的工作。例如将这个类自动把自己的类名保存到一个数组中。
## + (void) initialize
> The runtime sends initialize to each class in a program exactly one time just before the class, or any class that inherits from it, is sent its first message from within the program. (Thus the method may never be invoked if the class is not used.) The runtime sends the initialize message to classes in a thread-safe manner. Superclasses receive this message before their subclasses.

## + (void) load
> The load message is sent to classes and categories that are both dynamically loaded and statically linked, but only if the newly loaded class or category implements a method that can respond.

> The order of initialization is as follows:
> 
> All initializers in any framework you link to.
> 
> All +load methods in your image.
> 
> All C++ static initializers and C/C++ _attribute_ (constructor) functions in your image.> 
> All initializers in frameworks that link to you.
In addition:

> A class’s +load method is called after all of its superclasses’ +load methods.
> 
> A category +load method is called after the class’s own +load method.
> 
> In a custom implementation of load you can therefore safely message other unrelated classes from the same image, but any load methods implemented by those classes may not have run yet.