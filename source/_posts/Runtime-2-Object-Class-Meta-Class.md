title: "Runtime(2) - Object & Class & Meta Class"
date: 2014-12-14 09:45:14
tags:
---

Consider the output of following program：

````objc
@interface Sark : NSObject
@end

@implementation Sark
@end

int main(int argc, const char * argv[]) {
    @autoreleasepool {
        BOOL res1 = [(id)[NSObject class] isKindOfClass:[NSObject class]];
        BOOL res2 = [(id)[NSObject class] isMemberOfClass:[NSObject class]];
        
        BOOL res3 = [(id)[Sark class] isKindOfClass:[Sark class]];
        BOOL res4 = [(id)[Sark class] isMemberOfClass:[Sark class]];
        
        NSLog(@"%d %d %d %d", res1, res2, res3, res4);
    }
    return 0;
}
````

**Output:** 

````objc
2015-04-14 09:54:52.651 Runtime[14902:354725] 1 0 0 0
````

`isKindOf:` 和 `isMemberOf:` 是两个 NSObject 类的 runtime system 函数，通过 runtime 的源码 *Object.mm* 我们可以一窥它们的实现方法：

````objc
+ (id)class
{
    return self;
}

- (BOOL)isKindOf:aClass
{
	Class cls;
	for (cls = isa; cls; cls = cls->superclass) 
		if (cls == (Class)aClass)
			return YES;
	return NO;
}

- (BOOL)isMemberOf:aClass
{
	return isa == (Class)aClass;
}
````

上一篇 runtime 介绍了 `isa` 是指向 `Meta Class` 的指针。因此：

1. 对于第一个输出，首先，cls 为 NSObject 的 Meta Class，（因为 isa 指向 对象的 Meta Class），这时候 cls 不等于 NSObject class，结束了吗？ 没有。注意判断是在一个循环里面，cls = cls -> superclass，cls 的 superclass 是什么？ NSObject 的 Meta Class 的 superclass 指向 NSObject 本身。这时候 return YES。

2. 对于第二个输出， isa 指向 Meta Class， aClass 指向 NSObject class，return NO。

3. 对于第三个输出， cls 指向 Sark 的 Meta Class，然后指向 NSObject 的 Meta Class，然后指向 NSObject class，最后指向 nil。return NO。

4. 最后一个， isa 指向 Sark 的 Meta Class，return NO。