---
title: What's New in Swift 3
date: 2016-09-14 16:50:54
tags: swift3
---

[原文地址](https://realm.io/news/swiftcasts-daniel-steinberg-whats-new-in-swift-3-part-2/)

## [SE-0002](https://github.com/apple/swift-evolution/blob/master/proposals/0002-remove-currying.md): Removing [currying](https://en.wikipedia.org/wiki/Currying) function declaration syntax
{% codeblock lang:swift%} //Before:
func curried(x: Int)(y: String) -> Float {
    return Float(x) + Float(y)!
}
{% endcodeblock %}

在 Swift 3 之前定义柯里化函数需要使用两个括号，好消息是现在可以这样写了：

````swift
//After

func curried(x: Int) -> (String) -> Float {
    return {(y: String) -> Float in
        return Float(x) + Float(y)!
    }
}
````

函数变得更加直观：传入一个 `Int` 类型的 x，并返回一个接受 `String` 类型并返回 `Float` 类型的函数。

## [SE-0042](https://github.com/apple/swift-evolution/blob/master/proposals/0042-flatten-method-types.md): Flattening the function type of unapplied method references

假设我们有一个叫 Type 的 `struct` 类型，其中包含 instanceMethod 方法。

````swift
struct Type {
    var x: Int
    func instanceMethod(y: Int) -> Int {
        return x + y
    }
}
````

对 instanceMethod 进行赋值时，会显示 f 的类型为 `(Type) -> (Int) -> Int`

````swift
let f = Type.instanceMethod  //(Type) -> (Int) -> Int
````
对应的调用方式：

````swift
f(Type(x: 1))(2) // ==> 3
````
这个提案将 f 的类型 `(Type) -> (Int) -> Int` flatten 成了 `(Type, y: Int) -> Int`，这样 f 的调用语法更像一个标准的 `func`：

````swift
let f = Type.instanceMethod  // f: (Type, y: Int) -> Int
f(Type(x: 1), y: 2)          // ==> 3
````

## [SE-0049](https://github.com/apple/swift-evolution/blob/master/proposals/0049-noescape-autoclosure-type-attrs.md): Move @noescape and @autoclosure to be type attributes

假设现在有一个叫 `f2` 的 `func`，接受一个带有 `@autoclosure` 关键字的参数：

````swift
func f2(@autoclosure a : () -> ()) {}
````
可以这样调用 `f2`：

````swift
f2(print("hello”))
````
或者将这个 `f2` 赋值给 `x`（注：函数是 Swift 中的一等公民，这里 `x` 的类型是 `(@autoclosure () -> ()) -> ()`）：

````swift
let x = f2
x(print("hello"))
````
但如果你强制将 `x` 定义为其他类型时，编译器会报错：

````swift
let y : Int = x 
// error: cannot convert value of type '(@autoclosure () -> ()) -> ()' to specified type 'Int'
````
但是奇怪的是，如果你显式地定义一个变量的类型，编译器会报错：

````swift
let x2 : (@autoclosure () -> ()) -> () = f2
// error: attribute can only be applied to declarations, not types
````
对比上面 `x` 和 `x2`，编译器的行为自相矛盾了。

因此，这个提案将原本用于定义参数的 `@noescape` 和 `@autoclosure` 关键字改为定义类型。

````swift
// Before

func f(@noescape fn : () -> ()) {}  // declaration attribute

// After

func f(fn : @noescape () -> ()) {}  // type attribute
````

## [SE-0035](https://github.com/apple/swift-evolution/blob/master/proposals/0035-limit-inout-capture.md): Limiting inout capture to @noescape contexts

在 Swift 中，`@noescape` 关键字能够保证闭包在被调用的函数返回前就已经执行。

> A closure is said to escape a function when the closure is passed as an argument to the function, but is called after the function returns. When you declare a function that takes a closure as one of its parameters, you can write @escaping before the parameter’s type to indicate that the closure is allowed to escape. [^noescape]

这份提案将 `inout` 关键字捕获的内容限制在 `@noescape` 中。
注：参数标记为 `inout`，意味着它的值是要被修改并且最后被函数返回的。

因此，在 Swift 3 中，如果你在 `@escape` 闭包中试图改变一个 `inout` 的参数，编译器会报错：

````swift
func escape(f: () -> ()) {}
func noEscape(@noescape f: () -> ()) {}

func example(inout x: Int) {
  escape { _ = x } // error: closure cannot implicitly capture an inout parameter unless @noescape
  noEscape { _ = x } // OK, closure is @noescape
  escape {[x] in _ = x } // OK, immutable capture
}

struct Foo {
  mutating func example() {
    escape { _ = self } // error: closure cannot implicitly capture a mutating self parameter
    noEscape { _ = self } // OK
  }
}
````

## [SE-0103](https://github.com/apple/swift-evolution/blob/master/proposals/0103-make-noescape-default.md): Make non-escaping closures the default

现在闭包默认添加关键字 `@noescape`：

> To provide some more details, this approach has the following advantages:

> - Most functional algorithms written in pure Swift will benefit because they are naturally noescape. The core team feels that this will reduce the boilerplate involved with writing these algorithms.

> - The compiler has enough logic in it to provide a great QoI experience when a developer doesn’t think about escaping, and tries to escape a closure - it can provide a fixit that suggests adding @escaping.

> - Recent changes (to disallow escaping closures to close over an inout parameter) are pushing the language to prefer noescape closures. noescape closures have also always been the preferred default, since they eliminate a class of retain cycle issues.

> - "@autoclosure(escaping)" can be simplified and standardized to "@autoclosure @escaping”

`@noescape` 关键字将会在 Swift 3 中被移除，同时，编译器在发现作为参数的闭包在可能逃逸出函数体时会报错，提示你应该为参数添加 `@escaping` 关键字。

## [SE-0066](https://github.com/apple/swift-evolution/blob/master/proposals/0066-standardize-function-type-syntax.md): Standardize function type argument syntax to require parentheses

如果你再看回 #1，你就会发现柯理化函数是这样定义的： `func curried(x: Int) -> (String) -> Float`。

而这正是这个提案所修改的地方。

````swift
// Before
func curried(x: Int) -> String -> Float {

// After
func curried(x: Int) -> (String) -> Float {
````

作为参数的函数类型都要加上圆括号以表示这是一个函数参数。

类似地：

````
(Int) -> Int        // 传入一个 Int 
(Int, Int) -> Int   // 传入两个 Int
((Int, Int)) -> Int // 传入一个 tuple (包含两个 Int)
````

## [SE-0047](https://github.com/apple/swift-evolution/blob/master/proposals/0047-nonvoid-warn.md): Defaulting non-Void functions on unused results

考虑下面的代码：

````swift
func f() -> T {}
f()  //warning: Result of call to 'f()' is unused
````
当调用 `f` 时，编译器提示 `f()` 的返回值没有使用。

你可以将 `f` 赋值给下划线 `_` 来消除 warning：

````swift
func f() -> T {}
_ = f()
````
或者在这个函数前面添加 `@discardableResult`。

````swift
@discardableResult func f() -> T {}
````

## [SE-0102](https://github.com/apple/swift-evolution/blob/master/proposals/0102-noreturn-bottom-type.md) Remove @noreturn attribute and introduce an empty Never type

`@noreturn` 关键字被移除，并加入全新的 `Never` 枚举类型，定义如下：

> The return type of functions that do not return normally; a type with no
> values.
> 
> Use `Never` as the return type when declaring a closure, function, or
> method that unconditionally throws an error, traps, or otherwise does
> not terminate.
> 
>      func crashAndBurn() -> Never {
>          fatalError("Something very, very bad happened")
>      }

因此，抛出异常可以这些写：

````swift
func noReturn() -> Never {
    fatalError()
}

func pickPositiveNumber(below limit: Int) -> Int {
    guard limit >= 1 else {
        noReturn()
    }
    return rand(limit)
}
````

## [SE-0054](https://github.com/apple/swift-evolution/blob/master/proposals/0054-abolish-iuo.md): Abolish implicitly unwrapped optional type


````swift
let x: Int! = 5
let y = x        // y: Int?
let z = x + 0    // z: Int
````
在 Swift 3，y 的类型会变成 `Int?`，一个 optional Int，因为我们没有强制将它 unwrap。但 z 的类型为 `Int`，因为我们在做加法前已经强制把它 unwrap 成 `Int` 了。

更详细的例子：

````swift
func f() -> Int! { return 3 } // f: () -> Int?, has IUO attribute
let x1 = f() // succeeds; x1: Int? = 3
let x2: Int? = f() // succeeds; x2: Int? = .some(3)
let x3: Int! = f() // succeeds; x3: Int? = .some(3), has IUO attribute
let x4: Int = f() // succeeds; x4: Int = 3
let a1 = [f()] // succeeds; a: [Int?] = [.some(3)]
let a2: [Int!] = [f()] // illegal, nested IUO type
let a3: [Int] = [f()] // succeeds; a: [Int] = [3]

func g() -> Int! { return nil } // f: () -> Int?, has IUO attribute
let y1 = g() // succeeds; y1: Int? = .none
let y2: Int? = g() // succeeds; y2: Int? = .none
let y3: Int! = g() // succeeds; y3: Int? = .none, has IUO attribute
let y4: Int = g() // traps
let b1 = [g()] // succeeds; b: [Int?] = [.none]
let b2: [Int!] = [g()] // illegal, nested IUO type
let b3: [Int] = [g()] // traps

func p<T>(x: T) { print(x) }
p(f()) // prints "Optional(3)"; p is instantiated with T = Int?

if let x5 = f() {
  // executes, with x5: Int = 3
}
if let y5 = g() {
  // does not execute
}
````

接下来这几个 proposal 都和 Objective-C 有关系：

## [SE-0111](https://github.com/apple/swift-evolution/blob/master/proposals/0111-remove-arg-label-type-significance.md): Remove type system significance of function argument labels

[SE-0111](https://github.com/apple/swift-evolution/blob/master/proposals/0111-remove-arg-label-type-significance.md) 是关于定义函数参数时下划线的讨论。

## [SE-0021](https://github.com/apple/swift-evolution/blob/master/proposals/0021-generalized-naming.md): Naming functions with argument labels

[SE-0021](https://github.com/apple/swift-evolution/blob/master/proposals/0021-generalized-naming.md) 是关于函数命名风格的讨论。

## [SE-0022](https://github.com/apple/swift-evolution/blob/master/proposals/0022-objc-selectors.md): Referencing the Objective-C selector of a method

Swift 3 中你可以通过使用 `#selector` 来操作 selector，就像 Objective-C 中的 `@selector` 和 `SEL` 一样。

````swift
extension UIView {
    func insertSubview(view: UIView, at index: Int)
    func insertSubview(view; UIView, aboveSubview siblingSubview: UIView)
    func insertSubview(view; UIView, belowSubview siblingSubview: UIView)
}

let f2 = someView.insertSubview(view: aboveSubview:)
let sel = #selector(UIView.insertSubview(view:aboveSubview:))
````

## [SE-0064](https://github.com/apple/swift-evolution/blob/master/proposals/0064-property-selectors.md): Referencing the Objective-C selector of property getters and setters

现在可以通过 `#selector` 为一个类的属性指定 setter 和 getter 方法。

- 你需要使用 dynamic 修饰符，因为需要调用和 Objective-C 里相同的运行时特性。
- 你需要将属性定义为 `var` 变量，因为需要通过 `#selector` 进行 overrides。

````swift
class Person: NSObject {
    dynamic var firstName: String
    dynamic let lastName: String
    dynamic var fullName: String {
        return "\(firstName) \(lastName)"
    }

    init(firstName: String, lastName: String) {
        self.firstName = firstName
        self.lastName = lastName
    }
}

// Pass
let firstNameGetter = #selector(getter: Person.firstName)
let firstNameSetter = #selector(setter: Person.firstName)

// Error
let lastNameSetter = #selector(setter: Person.lastName)
// Argument of #selector(setter:) must refer to a variable property
````

## [SE-0062](https://github.com/apple/swift-evolution/blob/master/proposals/0062-objc-keypaths.md): Referencing Objective-C key-paths

跟 #13 类似，现在可以用 `#keyPath` 来指定 key path 来获取属性。

````swift
extension UIView {
    func insertSubview(view: UIView, at index: Int)
}

let path = #keyPath(UIView.superview.backgroundColor)
let color = someView.value(forKeyPath: path)
````

## [SE-0004](https://github.com/apple/swift-evolution/blob/master/proposals/0004-remove-pre-post-inc-decrement.md): Remove the ++ and – operators

关于将 `++`，`--` 替换为 `+= 1`，`-= 1` 的讨论。

## [SE-0007](https://github.com/apple/swift-evolution/blob/master/proposals/0007-remove-c-style-for-loops.md): Remove C-style for-loops with conditions and incrementors

C 风格的 for 循环从此消失：

````swift
for var i = 0 ; i < 10 ; i++ {
    print (i)
}
````

Swift 风格的循环如约而至。

````swift
for i in 0 ..< 10 {
    print (i)
}
````

## [SE-0065](https://github.com/apple/swift-evolution/blob/master/proposals/0065-collections-move-indices.md): A new model for collections and indices

以往，我们可以通过先获取 `collections` 类型的 index，并通过 index 的 `successor()` 来获得下一个元素。

Swift 3 中，`collections` 类型将负责遍历器的遍历。

````swift
// Before
let xs = [10, 20, 30, 40]
let i = xs.startIndex
let next = i.successor()

// After
let xs = [10, 20, 30, 40]
let i = xs.startIndex
let next = xs.index(after: i)
````

更多有关 `Collection` 类型的提案：

- [Adding a public base property to slices (SE-0093)](https://github.com/apple/swift-evolution/blob/master/proposals/0093-slice-base.md)
- [Constraining AnySequence.init (SE-0014)](https://github.com/apple/swift-evolution/blob/master/proposals/0014-constrained-AnySequence.md)
- [Add prefix(while:) and drop(while:) to the stdlib (SE-0045)](https://github.com/apple/swift-evolution/blob/master/proposals/0045-scan-takewhile-dropwhile.md)
- [Add sequence(first:next:) and sequence(state:next:) to the stdlib (SE-0094)](https://github.com/apple/swift-evolution/blob/master/proposals/0094-sequence-function.md)

## [SE-0032](https://github.com/apple/swift-evolution/blob/master/proposals/0032-sequencetype-find.md): Add first(where:) method to Sequence ()

我们经常会在 `Collection` 或 `Sequence` 中查找第一个满足某种特定条件的元素，Swift 3 中为 `Sequence` 类型添加了一个 `first(where:)` 的内置方法。

从此，你再也不用写一些类似 `seq.filter(predicate).first` 或 `seq.lazy.filter(predicate).first` 的代码。

## [SE-0008](https://github.com/apple/swift-evolution/blob/master/proposals/0008-lazy-flatmap-for-optionals.md): Add a lazy flatMap for sequences of optionals

为 `Sequence` 添加 [lazy](https://airspeedvelocity.net/2014/07/10/swifts-lazy-collections-and-sequences/) 版本的 flatMap 方法。

## [SE-0052](https://github.com/apple/swift-evolution/blob/master/proposals/0052-iterator-post-nil-guarantee.md): Change IteratorType post-nil guarantee

当迭代器类型开始返回 nil 时，后面的结果都会返回 nil。

## [SE-0025](https://github.com/apple/swift-evolution/blob/master/proposals/0025-scoped-access-level.md): Scooped access level

Swift 3 加入新的访问级别：`fileprivate`，只能在当前文件内可以访问对应的 API。

> - **Open** access and public access enable entities to be used within any source file from their defining module, and also in a source file from another module that imports the defining module. You typically use open or public access when specifying the public interface to a framework. The difference between open and public access is described below.
> 
> - **Internal** access enables entities to be used within any source file from their defining module, but not in any source file outside of that module. You typically use internal access when defining an app’s or a framework’s internal structure.
> 
> - **File-private** access restricts the use of an entity to its own defining source file. Use file-private access to hide the implementation details of a specific piece of functionality when those details are used within an entire file.
> 
> - **Private** access restricts the use of an entity to the enclosing declaration. Use private access to hide the implementation details of a specific piece of functionality when those details are used only within a single declaration.

`Open` > `Internal(default)` > `fileprivate` > `private`

## [SE-0070](https://github.com/apple/swift-evolution/blob/master/proposals/0070-optional-requirements.md): Make optional requirements Objective-C only

在 Objective-C 中，协议方法有两种：分别是 require 和 optional。在 Swift 3 中，你也可以为一个协议方法添加 `optional` 关键字：前提是 protocol 和 func 前都有 `@objc` 标志。

````swift
@objc protocol NSTableViewDelegate {
    @objc optional func tableView(_: NSTableView,
                            viewFor: NSTableColumn,
                            row: Int) -> NSView?
}
````

## [SE-0011](https://github.com/apple/swift-evolution/blob/master/proposals/0011-replace-typealias-associated.md): Replace typealias keyword with associatedtype for associated type declarations 
## [SE-0092](https://github.com/apple/swift-evolution/blob/master/proposals/0092-typealiases-in-protocols.md): Typealiases in protocols and protocol extensions

`typealias` 在以前的版本中有两个用处，一个是用在 protocol 中，还有一个是用在 extension 中。

````swift
// Before
protocol Prot {
    typealias Container : SequenceType
}
extension Prot {
    typealias Element = Container.Generator.Element
}
````
第二个 `typealias` 将 `Container.Generator.Element` 的名字用 `Element` 代替。 而 protocol 里面所用的 `typealias`，其实并不是一个真正意义上的 *typealias* ，它只是一个我们稍后在 protocol 里面用到的东西，并知道它会遵从 SequenceType。这个 `Container` 应该是一个 `associatedtype`，而不是一个严格意义上的 *typealias*。

Swift 3 中加入了 `associatedtype` 关键字更明确地区分开这两种用法。

````swift
// After
protocol Prot {
    associatedtype Container : SequenceType
}
extension Prot {
    typealias Element = Container.Generator.Element
}
protocol Sequence {
    associatedtype Iterator : IteratorProtocol
    typealias Element = Iterator.Element
}
````

## [SE-0048](https://github.com/apple/swift-evolution/blob/master/proposals/0048-generic-typealias.md): Generic type aliases

你可以将范型用在 `typealias` 上，使类型定义更具表达性。

````swift
typealias StringDictionary<T> = Dictionary<String, T>
typealias DictionaryOfStrings<T : Hashable> = Dictionary<T, String>
typealias IntFunction<T> = (T) -> Int
typealias Vec3<T> = (T, T, T)
typealias BackwardTriple <T1, T2, T3> = (T3, T2, T1)
````

## [SE-0069](https://github.com/apple/swift-evolution/blob/master/proposals/0069-swift-mutability-for-foundation.md): Mutability and foundation value types

关于 Mutability 和值类型的讨论。

## [SE-0086](https://github.com/apple/swift-evolution/blob/master/proposals/0086-drop-foundation-ns.md): Drop NS prefix in swift foundation

[带有历史包袱](http://stackoverflow.com/questions/473758/what-does-the-ns-prefix-mean)的 `NS` 前缀终于在 Swift 3 移除了，开发者的大事，大快所有人心的大好事。

更多相关提案：

- [Fully eliminate implicit bridging conversions from Swift (SE-0072)](https://github.com/apple/swift-evolution/blob/master/proposals/0072-eliminate-implicit-bridging-conversions.md)
- [Remove the Boolean protocol (SE-0109)](https://github.com/apple/swift-evolution/blob/master/proposals/0109-remove-boolean.md)


## [SE-0112](https://github.com/apple/swift-evolution/blob/master/proposals/0112-nserror-bridging.md): Improved NSError bridging

还记得 Swift 2 加入的 Error Handling (`try` 和 `catch`，`throws` 和 `rethrows`，`throw` 和 `do`) 吗？

这个提案就是关于桥接 `ErrorProtocol` 和 `NSError` 的讨论，Swift 3 改善了 `NSError` 的错误处理，并加入新的 `LocalizedError` 和 `RecoverableError` 类型。

````swift
protocol LocalizedError : Error {
    var errorDescription: String? { get }
    var failureReason: String? { get }
    var recoverySuggestion: String? { get }
    var helpAnchor: String? { get }
}
protocol RecoverableError : Error {
    var recoveryOptions: [String] { get }
    func attemptRecovery(optionIndex recoveryOptionIndex: Int,
                          resultHandler handler: (recovered: Bool) -> Void)            
    func attemptRecovery(optionIndex recoveryOptionIndex: Int) -> Bool
}
````

## 相关链接：

- [SE-0002](https://github.com/apple/swift-evolution/blob/master/proposals/0002-remove-currying.md) - Removing currying func declaration syntax
- [SE-0004](https://github.com/apple/swift-evolution/blob/master/proposals/0004-remove-pre-post-inc-decrement.md) - Remove the ++ and -- operators
- [SE-0007](https://github.com/apple/swift-evolution/blob/master/proposals/0007-remove-c-style-for-loops.md) - Remove C-style for-loops with conditions and incrementers
- [SE-0008](https://github.com/apple/swift-evolution/blob/master/proposals/0008-lazy-flatmap-for-optionals.md) - Add a Lazy flatMap for Sequences of Optionals
- [SE-0011](https://github.com/apple/swift-evolution/blob/master/proposals/0011-replace-typealias-associated.md) - Replace typealias keyword with associatedtype for associated type declarations
- [SE-0021](https://github.com/apple/swift-evolution/blob/master/proposals/0021-generalized-naming.md) - Naming Functions with Argument Labels
- [SE-0022](https://github.com/apple/swift-evolution/blob/master/proposals/0022-objc-selectors.md) - Referencing the Objective-C selector of a method
- [SE-0025](https://github.com/apple/swift-evolution/blob/master/proposals/0025-scoped-access-level.md) - Scoped Access Level
- [SE-0032](https://github.com/apple/swift-evolution/blob/master/proposals/0032-sequencetype-find.md) - Add first(where:) method to Sequence
- [SE-0035](https://github.com/apple/swift-evolution/blob/master/proposals/0035-limit-inout-capture.md) - Limiting inout capture to @noescape contexts
- [SE-0042](https://github.com/apple/swift-evolution/blob/master/proposals/0042-flatten-method-types.md) - Flattening the function type of unapplied method references
- [SE-0047](https://github.com/apple/swift-evolution/blob/master/proposals/0047-nonvoid-warn.md) - Defaulting non-Void functions so they warn on unused results
- [SE-0048](https://github.com/apple/swift-evolution/blob/master/proposals/0048-generic-typealias.md) - Generic Type Aliases
- [SE-0049](https://github.com/apple/swift-evolution/blob/master/proposals/0049-noescape-autoclosure-type-attrs.md) - Move @noescape and @autoclosure to be type attributes
- [SE-0052](https://github.com/apple/swift-evolution/blob/master/proposals/0052-iterator-post-nil-guarantee.md) - Change IteratorType post-nil guarantee
- [SE-0054](https://github.com/apple/swift-evolution/blob/master/proposals/0054-abolish-iuo.md) - Abolish ImplicitlyUnwrappedOptional type
- [SE-0062](https://github.com/apple/swift-evolution/blob/master/proposals/0062-objc-keypaths.md) - Referencing Objective-C key-paths
- [SE-0064](https://github.com/apple/swift-evolution/blob/master/proposals/0064-property-selectors.md) - Referencing the Objective-C selector of property getters and setters
- [SE-0065](https://github.com/apple/swift-evolution/blob/master/proposals/0065-collections-move-indices.md) - A New Model for Collections and Indices
- [SE-0066](https://github.com/apple/swift-evolution/blob/master/proposals/0066-standardize-function-type-syntax.md) - Standardize function type argument syntax to require parentheses
- [SE-0069](https://github.com/apple/swift-evolution/blob/master/proposals/0069-swift-mutability-for-foundation.md) - Mutability and Foundation Value Types
- [SE-0070](https://github.com/apple/swift-evolution/blob/master/proposals/0070-optional-requirements.md) - Make Optional Requirements Objective-C-only
- [SE-0086](https://github.com/apple/swift-evolution/blob/master/proposals/0086-drop-foundation-ns.md) - Drop NS Prefix in Swift Foundation
- [SE-0092](https://github.com/apple/swift-evolution/blob/master/proposals/0092-typealiases-in-protocols.md) - Typealiases in protocols and protocol extensions
- [SE-0102](https://github.com/apple/swift-evolution/blob/master/proposals/0102-noreturn-bottom-type.md) - Remove @noreturn attribute and introduce an empty Never type
- [SE-0103](https://github.com/apple/swift-evolution/blob/master/proposals/0103-make-noescape-default.md) - Make non-escaping closures the default
- [SE-0111](https://github.com/apple/swift-evolution/blob/master/proposals/0111-remove-arg-label-type-significance.md) - Remove type system significance of function argument labels
- [SE-0112](https://github.com/apple/swift-evolution/blob/master/proposals/0112-nserror-bridging.md) - Improved NSError Bridging