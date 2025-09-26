# 第七章：抓紧你的椅子；高级类型变更即将到来！

Swift 是一种伟大的语言，并且随着每个版本的发布而变得越来越好。到目前为止，我们已经涵盖了你在日常项目中可能使用的大多数功能。我们将介绍一些你可能不会经常使用的语言改进。本章重点介绍`UnsafePointer`类型、`typealiases`和浮点运算。

# Unmanaged 和 UnsafePointer 的变更

准备好，因为我们即将浏览一些你可能见得不多、名字听起来让人想回避的类型，比如“我宁愿不碰那些”这类对你们中的一些人来说可能会感到不适的类型。就 Swift 中类型的命名惯例而言，对于普通开发者来说似乎既友好又合理。然而，有一组类型甚至没有列在 Swift 编程语言文档的主要部分中。这些是语言的“黑羊”类型。它们的名称如`Unmanaged`、`UnsafeMutableRawPointer`和`UnsafeBufferPointer`等。这些类型使用起来感觉`不安全`。也许，这些名称本身就是一个大大的提示，表明作为开发者的你，在使用这些类型时需要采取一些预防措施。如果你在 Swift 中开发的时间足够长，你最终会遇到这些类型之一。我们不妨先了解一下 Swift 3 中这些类型的变更，这样当你需要使用新特性时，你就能掌握最新的知识。

## 将 Unmanaged 转换为使用 UnsafePointer [SE-0017]

`Unmanaged`是 Swift 中的一种类型，允许你与一个`未管理的`对象引用一起工作，这意味着你负责对象的内存，并保持其存活。`UnsafePointer`是一种表示指针类型数据的原始指针的类型。你完全负责使用此类型管理内存。这两种类型在处理 C API 时都很有用。接受`void *`或`const void *`等类型的 C 函数非常常见，但在 Swift 中可能会出现一些问题。

当 C API 传递`void *`或`const void *`类型（或其他不易转换为 Foundation 类型的类型）到 Swift 时，该类型会被转换为`UnsafePointer`。这是我们第一步，但不是我们最终的目的地，因为我们想要的是一个可以在 Swift 中高效使用的类型。我们最终想要的是一个`Unmanaged`类型，因为这个类型为我们提供的对象提供了一个类型安全的包装，即使它不参与**自动引用计数（ARC**）。使用`Unmanaged`类型，开发者可以手动做出内存决策。在 Swift 2 中，没有直接的转换允许你从`UnsafePointer`转换为`Unmanaged`类型。你必须先转换为一个**桥接类型**，然后再转换为你想要的类型，即**UnsafePointer** | **COpaquePointer** | **Unmanaged**。

你使用以下方法之一在`Unmanaged`类型上完成转换：

```swift
static func fromOpaque(value: COpaquePointer) -> Unmanaged<Instance> 
func toOpaque() -> COpaquePointer 

```

在 Swift 2 中：

```swift
let str0: CFString = "Test string" as CFString 
let bits: Unmanaged<CFString> = Unmanaged.passRetained(str0) 
let oPtr: COpaquePointer = bits.toOpaque() 
let ptr: UnsafePointer<CFString> = UnsafePointer(oPtr) 

let oPtr2 = COpaquePointer(ptr) 
let str1: Unmanaged<CFString> = Unmanaged.fromOpaque(oPtr2) 
str1.takeRetainedValue() 

```

在 Swift 3 中，我们现在可以直接在 `Unmanaged` 和 `UnsafePointer` 之间进行转换。`fromOpaque` 和 `toOpaque` 方法用 `UnsafeRawPointer` 和 `UnsafeMutableRawPointer` 类型替换了 `COpaquePointer`。我们通过消除中间人简化了我们的代码。

```swift
 static func fromOpaque(_ value: UnsafeRawPointer) -> Unmanaged<Instance> 
func toOpaque() -> UnsafeMutableRawPointer 

```

在 Swift 3 中：

```swift
let str0: CFString = "Test string" as CFString 
let bits = Unmanaged.passUnretained(str0) 
let ptr = bits.toOpaque() 

let str1: Unmanaged<CFString> = Unmanaged.fromOpaque(ptr) 
str1.takeRetainedValue() 

```

### 注意

你可以在 [`github.com/apple/swift-evolution/blob/master/proposals/0017-convert-unmanaged-to-use-unsafepointer.md`](https://github.com/apple/swift-evolution/blob/master/proposals/0017-convert-unmanaged-to-use-unsafepointer.md) 阅读提案。

## 使用可选类型明确 `UnsafePointer` [SE-0055]

与 Objective-C 不同，在 Objective-C 中你可以将指针标记为 `可空` 或 `非空`，Swift 没有确定指针是否为空的方法。因此，当你获得一个 `UnsafePointer<T>` 的引用时，你可能会持有指向空的指针。这是一个问题，因为 `UnsafePointer`（以及类似类型）本质上是在引用 C 指针。如果开发者的代码没有期望或考虑到空值，那么程序可能会崩溃。这在 Swift 中尤其令人担忧，因为使用 `UnsafePointer` 可以执行的所有非平凡操作都依赖于一个有效的、非空的底层指针。

幸运的是，在 Swift 中，对于内置类型、类和结构体，我们不需要处理这个问题，因为我们有可选类型。众所周知，`可选类型` 允许我们处理类型可能包含或不包含值的情况。Swift 3 新增了将 `可选类型` 应用于 `UnsafePointer` 类型的功能。当你确信你的指针不能为空时，你使用常规形式：`UnsafePointer<T>`。当你想要表示一个 `可空` 版本时，你使用可选语法（`UnsafePointer<T>?`）。

### 注意

你可以在 [`github.com/apple/swift-evolution/blob/master/proposals/0055-optional-unsafe-pointers.md`](https://github.com/apple/swift-evolution/blob/master/proposals/0055-optional-unsafe-pointers.md) 阅读提案。

## 添加 `UnsafeRawPointer` [SE-0107]

Swift 添加了 `UnsafePointer` 类型，以便与 C API 进行交互并简化高性能数据结构的构建（例如，低级图形编程或基于数学的建模）。在这方面，`UnsafePointer` 是 Swift 中的一个重要补充。不幸的是，`UnsafePointer` 的实现允许开发者绕过为确保类型安全的内存访问而设置的防护措施。可以使用 `UnsafePointer` 类型来违反编译器逻辑。违反通常会导致编译错误。然而，编译器为 `UnsafePointer` 类型内置了一个异常，允许编译过程继续进行。在许多情况下，运行使用类型化内存访问来引用不同类型内存位置的程序会导致程序崩溃。Swift 3 引入了 `UnsafeRawPointer` 来处理无类型内存。让我们用一个例子来说明类型内存访问的滥用。

在 Swift 2 中：

```swift
let msg: CFString = "just a few characters" as CFString 
let unmgd: Unmanaged<CFString> = Unmanaged.passRetained(msg) 
let ptr: UnsafeMutablePointer<CFString> = UnsafeMutablePointer(unmgd.toOpaque()) 
// reassign pointer address with new value 
ptr[0] = "testing..." as CFString 
// use typed access of Int to access CFSTring memory location 
let u = UnsafePointer<Int>(ptr)[0] 

```

在我们的类型内存访问示例中，我们创建了一个`UnsafeMutablePointer<CFString>`来引用一个类型化的内存块。然后，我们将第一个块中的值更改为一个新的`CFString`，测试我们的指针。到目前为止一切正常。接下来，我们从一个现有的内存位置创建了一个`UnsafePointer<Int>`。请注意，我们将类型绑定到了*Int*，而不是原始指针中使用的`CFString`。如果我们使用这个*Int*指针，我们的程序可能会崩溃。即使编译器认为这段代码可疑，它也允许编译，因为我们正在使用`UnsafePointer`类型，这些类型在编译器的异常列表中对于某些类型的操作是允许的。为了纠正我们示例中的问题，我们需要使用一个不需要知道类型的类型来访问内存。`UnsafeRawPointer`和`UnsafeMutableRawPointer`在 Swift 3 中被引入，就是为了做这件事。

**要点：**

+   *原始*类型允许对内存进行无类型访问；*类型化*版本使用它们的类型访问内存

+   *原始*访问基本上允许 C 类型的`memcopy`操作，而*类型化*访问遵循类型别名规则

+   当类型不明确时（例如，`const void *`或`void *`），C 类型现在被导入为`UnsafeMutableRawPointer`和`UnsafeRawPointer`，而当它们的类型可以确定时（例如，`const T*`），则导入为`UnsafePointer<T>`和`UnsafeMutablePointer<T>`。

### 注意

您可以在[`github.com/apple/swift-evolution/blob/master/proposals/0107-unsaferawpointer.md`](https://github.com/apple/swift-evolution/blob/master/proposals/0107-unsaferawpointer.md)阅读另一份提案。

# 类型别名和协议更改

类型别名是命名类型，用于在 Swift 中替换现有类型。一旦定义，您就可以在代码的任何地方使用这些类型。Swift 3 现在支持基于泛型的类型别名。此外，现在也支持协议和协议扩展的类型别名。谈到协议，Swift 3 对协议的使用进行了更改，这使得事情变得更简单，并为该功能的预期未来更改铺平了道路。让我们更详细地添加新更改，并通过一些示例来探讨。

## 泛型类型别名 [SE-0048]

泛型类型别名是 Swift 3 的新增功能。提醒一下，类型别名是在语言中为现有类型声明一个命名别名的途径。创建您的命名别名后，您可以在代码中使用这些别名，就像使用任何其他类型一样。泛型类型别名允许您添加可以在定义泛型类型时使用的类型参数。让我们考虑几个示例，以展示在 Swift 3 中创建类型别名的新可能性：

```swift
    typealias ScoreBag<T> = [T]
    typealias TriplePointTuple<T> = (T,T,T)
    typealias AddPlotter<X:Hashable,Y> = Dictionary<X, Y>
    typealias UndoItem<T> = [Date:T]

```

### 注意

您可以在[`github.com/apple/swift-evolution/blob/master/proposals/0048-generic-typealias.md`](https://github.com/apple/swift-evolution/blob/master/proposals/0048-generic-typealias.md)阅读另一份提案。

### 注意

你可以在 [`github.com/apple/swift-evolution/blob/master/proposals/0095-any-as-existential.md`](https://github.com/apple/swift-evolution/blob/master/proposals/0095-any-as-existential.md) 阅读这个提议。

在 Swift 的早期版本中，当你定义一个遵循多个协议的类型时，你需要使用 `protocol<...>` 语法。在 Swift 3 中，你现在在每个你类型采用的协议之间使用 `&`。这仅仅是语法糖，但将成为在 Swift 未来版本中定义更多泛型类型的基础。

在 Swift 2 中：

```swift
protocol Driving {} 
protocol Parking {} 
protocol Braking {} 
struct Car: Driving, Parking, Braking {} 
let zoomzoom: protocol<Driving, Braking, Parking> = Car() 

```

在 Swift 3 中：

```swift
let zoomzoom: Driving & Braking & Parking = Car() 

```

为了更好地传达由多个协议构建的复合类型的意图，Swift 团队现在更倾向于使用 `&` 而不是逗号来定义类型上的多个协议。

## 协议和协议扩展中的类型别名 [SE-0092]

Swift 2.2 引入了 **associatedtype** 关键字来处理协议中的关联类型。这个变化消除了使用 *typealias* 关键字时的混淆，因为它现在只负责定义类型。添加 *associatedtype* 关键字的另一个好处是，它允许我们在协议和协议扩展中使用基于关联类型的类型别名。查看标准库中的 *Sequence* 协议，我们可以看到 *Iterator* 被定义为继承自 *IteratorProtocol* 的关联类型。使用 Swift 3，我现在可以添加一个名为 *Element* 的类型别名，它间接引用了 *IteratorProtocol* 上的关联类型，从而使我的语法更简洁。此外，我还可以在我的协议扩展中使用我创建的任何类型别名。 

在 Swift 3 中：

```swift
public protocol Sequence { 

    associatedtype Iterator : IteratorProtocol 

    typealias Element = Iterator.Element 

    public func makeIterator() -> Iterator 

   func map<T>(_ transform: (Element) throws -> T)  
rethrows -> [T] 

} 

```

在先前的 *Sequence* 协议中，我可以在 `map()` 函数中使用我的类型别名 `Element`。在 Swift 的早期版本中，你将不得不使用 `Iterator.Element`。

### 注意

你可以在 [`github.com/apple/swift-evolution/blob/master/proposals/0092-typealiases-in-protocols.md`](https://github.com/apple/swift-evolution/blob/master/proposals/0092-typealiases-in-protocols.md) 阅读这个提议。

# 浮点类型变更

浮点类型用于存储分数数字。标准库中的主要浮点类型是 `Float` 和 `Double`。Swift 团队创建了一个 `FloatingPoint` 协议来存储常见的数学操作，这使得你更容易创建支持所有浮点类型的函数。在本节中，我们将介绍对 `FloatingPoint` 协议和舍入函数的扩展。

## 增强的浮点协议 [SE-0067]

当前的 `FloatingPoint` 协议并没有提供一套完整的特性来真正符合 IEEE 754 [`en.wikipedia.org/wiki/IEEE_floating_point#CITEREFIEEE_7542008`](https://en.wikipedia.org/wiki/IEEE_floating_point#CITEREFIEEE_7542008) 类型。对 `FloatingPoint` 协议的更改旨在扩展大多数预期应包含的操作的范围。Swift 还添加了一个名为 `BinaryFloatingPoint` 的第二个协议（符合 `FloatingPoint`），它对于泛型编程将非常有用。

现在的 `FloatingPoint` 协议包含了 IEEE 754 的基本操作的大部分。`BinaryFloatingPoint` 协议额外符合 `FloatLiteralConvertible`。你可以使用 `FloatingPoint` 协议执行正常的算术和比较操作，以及使用 `BinaryFloatingPoint` 协议执行更适合使用具有固定基数 2 的浮点类型的更复杂操作。

在 `FloatingPoint` 和 `BinaryFloatingPoint` 协议上定义了许多新的操作，我将把它们留给你作为未来的练习。

### 注意

你可以在 [`github.com/apple/swift-evolution/blob/master/proposals/0067-floating-point-protocols.md`](https://github.com/apple/swift-evolution/blob/master/proposals/0067-floating-point-protocols.md) 阅读该提案。

## FloatingPoint 协议上的新舍入函数 [SE-0113]

Swift 标准库没有内置方法来实现舍入函数，如 `floor()` 或 `ceil()`。当你需要这些方法时，你必须导入 `Darwin` 或 `Glibc` 来访问使用 C 实现的版本。在 Swift 3 中，Swift 团队为 `FloatingPoint` 协议添加了新的舍入方法。舍入以及/或将浮点类型转换为整数的操作是协议应该处理的操作。

`FloatingPoint` 协议的更改包括添加了 `FloatingPointRoundingRule` 枚举和两个舍入方法，`round()` 和 `rounded()`：

```swift
public func rounded(_ rule: FloatingPointRoundingRule) -> Self 
public mutating func round(_ rule: FloatingPointRoundingRule) 
public mutating func round() 

public enum FloatingPointRoundingRule { 
   case toNearestOrAwayFromZero 
   case toNearestOrEven 
   case up 
         case down 
   case towardZero 
   case awayFromZero 
} 

```

使用新的本地化实现的舍入方法，你将能够复制使用 `ceil` 和 `floor` 等方法得到的行为：

```swift
(10.5).rounded(.down) // -> 10.0 
(5.2).rounded(.up) // -> 6.0 

```

### 注意

你可以在 [`github.com/apple/swift-evolution/blob/master/proposals/0113-rounding-functions-on-floatingpoint.md`](https://github.com/apple/swift-evolution/blob/master/proposals/0113-rounding-functions-on-floatingpoint.md) 阅读该提案。

# 摘要

我们介绍了在使用 C APIs 时使用的 `Unmanaged`、`UnsafePointer` 和类似类型的更改。你了解了在使用这些类型时 Swift 2 的编译器怪癖以及 Swift 3 中这些改进。接下来，我们讨论了类型别名更改、它们与协议的使用以及协议扩展。最后，我们探讨了 FloatingPoint 协议的更改。在下一章中，我们将介绍 Foundation 框架的新增功能。
