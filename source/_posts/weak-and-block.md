title: "__weak and __block"
date: 2014-12-26 23:42:49
tags:
---
## __block

> __block variables live in storage that is shared between the lexical scope of the variable and all blocks and block copies declared or created within the variable’s lexical scope. Thus, the storage will survive the destruction of the stack frame if any copies of the blocks declared within the frame survive beyond the end of the frame (for example, by being enqueued somewhere for later execution). Multiple blocks in a given lexical scope can simultaneously use a shared variable.

`__block` 是一个存储的修饰词，它定义了在 Block 里的变量应该直接被捕获，而不是 copy。换句话说，`__block` 让你在 Block 中修改变量：

````objc
__block NSString *aString = @"Hey!";
void(^aBlock)() = ^{ aString = @"Hello!" }; // without __block you couldn't modify aString
NSLog(@"%@", aString); // Hey!
aBlock();
NSLog(@"%@", aString); // Hello!
````

在 ARC 中，因为这个变量在 Block 中需要被引用，所以会被自动地 retain。在上面这个例子中，当在 Block 中捕获时 `aString` 会被发送一个 retain 的消息。

值得注意的是，如果是在 MRC 中，这会出问题。因为你在引用之前没有 retain 。

## __weak

> __weak specifies a reference that does not keep the referenced object alive. A weak reference is set to nil when there are no strong references to the object.

而如果你将它标记为 `__weak` 的话，这个变量就不会被 retain 。想象一下，如果 block 的生命周期比 block 里面所引用的变量长，会发生什么情况？

[clang doc](http://clang.llvm.org/docs/BlockLanguageSpec.html#objective-c-extensions) ：
> In the Objective-C and Objective-C++ languages, we allow the `__weak` specifier for `__block` variables of object type. If garbage collection is not enabled, this qualifier causes these variables to be kept without retain messages being sent. This knowingly leads to **dangling pointers** if the Block (or a copy) outlives the lifetime of this object.

> In garbage collected environments, the `__weak` variable is set to nil when the object it references is collected, as long as the `__block` variable resides in the heap (either by default or via Block_copy()). The initial Apple implementation does in fact start `__block` variables on the stack and migrate them to the heap only as a result of a Block_copy() operation.

> It is a runtime error to attempt to assign a reference to a stack-based Block into any storage marked `__weak`, including `__weak __block` variables.

如果 Block 比这个对象还要晚才销毁的话，这对象的指针就会变成悬挂指针（dangling pointers）。

因此，`__block` 可以用来避免 ARC 下的 `retain cycle` 这个说法是错误的。

To sum it up,

MRC：

````objc
__block typeof(self) blockSelf = self; //this would retain self in ARC!
[self methodThatTakesABlock:^ {
    [blockSelf doSomething];
}];
````

ARC:

````objc
__weak typeof(self) weakSelf = self;
[self methodThatTakesABlock:^ {
    [weakSelf doSomething];
}];
````