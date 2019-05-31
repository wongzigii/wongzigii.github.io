title: "LLVM Lifetime Qualifiers"
date: 2015-1-10 22:55:06
tags:
---

LLVM 在 3.1 以后引入了四个 `Lifetime Qualifiers`，它们分别是 `__autoreleasing`，`__strong`，`__unsafe_untained` 还有 `__weak`。其中 __weak 只在 ARC 下才存在，而其他三个仍然可以在 MRC 下看到它们。
 
- __strong is the default. An object remains “alive” as long as there is a strong pointer to it.
- __weak specifies a reference that does not keep the referenced object alive. A weak reference is set to nil when there are no strong references to the object.
- __unsafe_unretained specifies a reference that does not keep the referenced object alive and is not set to nil when there are no strong references to the object. If the object it references is deallocated, the pointer is left dangling.
- __autoreleasing is used to denote arguments that are passed by reference (id *) and are autoreleased on return.

## `__strong`

 在 ARC 下，所有的指针都会默认为 `__strong`. 也就是说当指针指向一个对象时，这个对象的 retain count 就会加 1。当超出作用域或者指针被赋值后，编译器会自动为 `__strong` 修饰的对象指针添加 release。
 
## `__weak` and `__unsafe_unretained`
  
 为了解决这个问题，`__unsafe_unretained` 和 `__weak` 诞生了。最常见的是当你声明一个 delegate 的时候，你需要将它的类型声明为 `weak` 或者 `unsafe_unretained` (assign 在这里相当于 `unsafe_unretained`)。在实例变量前面标记为 `__weak` 或者 `__unsafe_unretained` 。这样，delegate 的实例变量仍然指向第一个对象，但是不会被 retain，从而避免了 retain cycle。
 
 除了 delegate 之外，也可以用来避免其他代码的 retain cycle。而 Leaks Instrument 里面已经可以检查是否有 retain cycle 所导致的 memory leak 了。

###`__unsafe_unretained` 和 `__weak` 的区别

对于 `__weak`，当对象销毁后指针会被设为 nil， 这个做法使得指针的使用更安全。而正如它的名字那样，`__unsafe_unretained` 仍然会指向销毁对象的地址，顾名思义这是 unsafe 的。

值得注意的是：

> __weak is only supported for iOS 5.0 and Lion as deployment targets.

## `__autoreleasing`

将对象指向自动释放池，编译器会自动计算 retain count，并且释放内存。

````objc
+ (id)array 
{
  return [[NSMutableArray alloc] init]; 
}
````

我发现这个修饰符一般用在方法里面，因为你需要在消息的作用域里拥有这个对象，但是你又不能在消息执行完之前把它释放掉，因此把它交给 autoreleasepool，在消息执行完后，再由自动释放池释放。

`Variable Qualifiers` 的格式：

````objc
MyClass * __weak myWeakReference;
MyClass * __unsafe_unretained myUnsafeReference;
````

注意：如果你这像下面那样使用 `__weak`，

````objc
NSString * __weak string = [[NSString alloc] initWithFormat:@"First Name: %@", [self firstName]];
NSLog(@"string: %@", string);
````

编译器会警告你：`Assigning retained object to weak variable; object will be released after assignment.` 当你 init 完这个对象后，由于前面是 __weak 修饰符，`string` 会马上被 dealloc，因为这时候没有其他强引用指向这个对象。

还有一个需要注意的地方，就是将对象传递给消息：

````objc
NSError *error;
BOOL OK = [myObject performOperationWithError:&error];
if (!OK) {
    // Report the error.
    // ...
````

实际上，

````objc
NSError * __strong e;
````

并且（上面说过的作为参数会加一个 `__autoreleasing` 修饰符）

````objc
-(BOOL)performOperationWithError:(NSError * __autoreleasing *)error;
````

所以编译器实际上会重写成这样：

````objc
NSError * __strong error;
NSError * __autoreleasing tmp = error;
BOOL OK = [myObject performOperationWithError:&tmp];
error = tmp;
if (!OK) {
    // Report the error.
    // ...
````

因为局部变量同时声明了 `__strong` 和 `__autoreleasing`，编译器实际上会创建一个修饰符为 `__autoreleasing` 的临时变量，在消息执行完后再传回 `__strong` 修饰符的变量。
