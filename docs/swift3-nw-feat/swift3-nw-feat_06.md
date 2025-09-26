# 第六章。额外，额外令人兴奋的集合和闭包变化！

在本章中，我们专注于 Swift 3 中的集合和闭包的变化。集合对于所有编程语言都很重要，因为它们允许你持有相关项的组。闭包对于 Swift 也同样重要，因为它们赋予你将功能传递到代码中不同位置的能力。有一些很好的新增功能将使使用集合变得更加有趣。我们还将探讨在 Swift 2.2 中创建闭包的一些令人困惑的副作用，以及这些副作用如何在 Swift 3 中得到修复。

# 集合和序列类型变化

让我们从 Swift 3 对集合和序列类型的变化开始讨论。其中一些变化很微妙，而另一些则可能需要对你自定义实现进行相当数量的重构。Swift 提供了三种主要的集合类型来存储你的值：数组、字典和集合。数组允许你按顺序列表存储值。字典为你提供无序的键值存储。最后，集合提供了一组无序的唯一值列表（即不允许重复）。

## 懒惰的 FlatMap 用于可选序列 [SE-0008]

数组、字典和集合在 Swift 中作为泛型类型实现。它们各自实现了新的 Collection 协议，该协议实现了 Sequence 协议。沿着从顶级类型到 Sequence 协议的路径，你将找到在这个继承链中也实现了的各种其他协议。对于我们对 `flatMap` 和懒 `flatMap` 变化的讨论，我想专注于序列。

序列包含一组值，允许用户逐个访问每个值。在 Swift 中，你可能会考虑使用 for-in 循环来遍历你的集合。Sequence 协议提供了许多操作的实施，这些操作可能是你想要在列表上使用顺序访问来执行的操作；所有这些你都可以在采用协议时在你的自定义集合中重写。其中一个操作是 `flatMap` 函数，它返回一个包含从序列的每个元素应用转换操作得到的扁平化或连接的值的数组。让我们考虑一下我们如何使用 `flatMap` 方法。

```swift
let scores = [0, 5, 6, 8, 9] 
         .flatMap{ [$0, $0 * 2] } 
print(scores)  // [0, 0, 5, 10, 6, 12, 8, 16, 9, 18] 

```

在我们上面的示例中，我们取一个分数列表，并使用我们的转换闭包调用 `flatMap`。每个值都被转换成一个包含原始值和双倍值的序列。一旦转换操作完成，`flatMap` 方法将中间序列扁平化为单个序列。

我们还可以使用 `flatMap` 方法与包含可选值的 *序列* 来实现类似的结果。这次我们在扁平化的序列中省略了值，通过在转换中返回 nil。在下一个示例中，我们使用 `flatMap` 方法从我们的集合中移除所有 nil 值。

```swift
let oddSquared = [1, 2, 3, 4, 5, 10].flatMap { n in 
    n % 2 == 1 ? n*n : nil 
} 
print(oddSquared) // [1, 9, 25] 

```

前两个示例是对小集合值的基本转换。在更复杂的情况下，你需要处理的集合可能非常大，且转换操作成本高昂。在这些参数下，你不会想在绝对需要之前执行 `flatMap` 操作或其他任何昂贵的操作。幸运的是，在 Swift 中我们有针对此用例的懒操作。序列包含一个 `lazy` 属性，它返回一个 `LazySequence`，可以对序列方法执行懒操作。使用我们上面的第一个示例，我们可以获得一个懒序列并调用 `flatMap` 来获取懒实现。只有在懒操作场景下，操作才会在代码中稍后使用 `scores` 时完成。为了演示懒操作，我们定义了一个使用 `lazy` 属性和我们的 `flatMap` 方法的集合。

```swift
let scores = [0, 5, 6, 8, 9] 
    .lazy 
    .flatMap{ [$0, $0 * 2] } // lazy assignment has not executed 

for score in scores{  
    print(score) 
} 

```

`lazy` 操作在我们上面的测试中按预期工作。然而，当我们使用包含可选值的第二个示例中的 `lazy` 形式的 `flatMap` 时，Swift 2 中的 `flatMap` 会立即执行。使用 `oddSquared` 的懒版本应该会延迟 `flatMap` 操作的执行，直到我们使用该变量。然而，`flatMap` 方法立即执行，就像没有懒形式一样。

```swift
let oddSquared = [1, 2, 3, 4, 5, 10] 
    .lazy               // lazy assignment but has not executed 
    .flatMap { n in 
    n % 2 == 1 ? n*n : nil 
} 

for odd in oddSquared{ 
    print(odd) 
} 

```

本质上，这曾是 Swift 中的一项特性，在 Swift 3 中被修改，以使其行为类似于其他懒加载实现。

### 注意

你可以在以下链接中阅读提案 [`github.com/apple/swift-evolution/blob/master/proposals/0008-lazy-flatmap-for-optionals.md`](https://github.com/apple/swift-evolution/blob/master/proposals/0008-lazy-flatmap-for-optionals.md)

## 将 first(where:) 方法添加到 Sequence [SE-0032]

使用集合的一个常见任务是找到符合某个条件的第一元素。例如，你可以要求在包含 100 分的测试分数的学生数组中找到第一个学生。你可以通过使用谓词来返回匹配条件的过滤序列，然后只需在序列中返回第一个学生。然而，直接调用一个可以返回项目的方法会更简单，而不需要两步操作。这个功能在 Swift 2 中缺失，但经过社区投票，已被添加到这次发布中。在 Swift 3 中，现在在 Sequence 协议上有一个方法来实现 `first(where:).`

```swift
["Jack", "Roger", "Rachel", "Joey"].first { (name) -> Bool in
name.contains("Ro")
} // =>returns Roger
```

这个 `first(where:)` 扩展是语言的一个很好的补充，因为它确保了一个简单且常见的任务在 Swift 中实际上很容易执行。

### 注意

你可以在以下链接中阅读提案 [`github.com/apple/swift-evolution/blob/master/proposals/0032-sequencetype-find.md`](https://github.com/apple/swift-evolution/blob/master/proposals/0032-sequencetype-find.md)

## 添加序列（first: next:）和序列（state: next:）[SE-0094]

Swift 3 引入了两个新的全局函数，用于操作序列：`sequence(first:next:)`

和 `(state:next:)`。

让我们看看下面的完整定义：

```swift
public func sequence<T>(first: T, next: @escaping (T) -> T?) -> UnfoldSequence<T, (T?, Bool)> 

public func sequence<T, State>(state: State, next: @escaping (inout State) -> T?) -> UnfoldSequence<T, State> 

public struct UnfoldSequence<Element, State> : Sequence, IteratorProtocol 

```

这两个函数被添加作为对 Swift 3 中移除的 C 风格 for 循环的替代，并作为对 Swift 2 中已经存在的全局 reduce 函数的补充。这些添加有趣的地方在于，每个函数都有生成和处理无限大小序列的能力。让我们检查第一个序列函数，以更好地理解它是如何工作的：

```swift
/// - Parameter first: The first element to be returned from the sequence. 
/// - Parameter next: A closure that accepts the previous sequence element and 
///   returns the next element. 
/// - Returns: A sequence that starts with `first` and continues with every 
///   value returned by passing the previous element to `next`. 
/// 
func sequence<T>(first: T, next: @escaping (T) -> T?) -> UnfoldSequence<T, (T?, Bool)> 

```

第一个序列方法返回一个由重复调用 *next* 参数创建的序列，该参数包含一个将按需执行的闭包。返回值是一个 `UnfoldSequence`，它包含传递给序列方法的 `first` 参数以及将 *next* 闭包应用于前一个值的结果。如果 `next` 最终返回 `nil`，则序列是有限的；如果 `next` 永远不返回 *nil*，则序列是无限的。在下面的示例中，我们使用 `sequence(first: next:)` 的尾随闭包形式创建和分配我们的序列。

```swift
let mysequence = sequence(first: 1.1) { $0 < 2 ? $0 + 0.1 : nil } 
for x in mysequence{ 
    print (x) 
} // 1.1 1.2 1.3 1.4 1.5 1.6 1.7 1.8 1.9 2.0 

```

我们的有限序列将从 1.1 开始，并重复调用 `next`，直到我们的下一个结果大于 2，此时 `next` 将返回 `nil`。我们可以很容易地将这个转换为无限序列，通过移除我们之前值不能大于 2 的条件。第二个序列函数维护一个可变状态，该状态传递给所有 `next` 的懒调用以创建和返回一个序列。让我们考虑一个使用第二个方法的示例：

```swift
/// - Parameter state: The initial state that will be passed to the closure. 
/// - Parameter next: A closure that accepts an `inout` state and returns the 
///   next element of the sequence. 
/// - Returns: A sequence that yields each successive value from `next`. 
/// 
public func sequence<T, State>(state: State, next: (inout State) -> T?) -> UnfoldSequence<T, State> 

```

这个版本的序列函数使用一个传入的闭包，允许你在每次调用 `next` 时更新可变状态。正如我们第一个序列函数的情况一样，一个有限序列在 `next` 返回 `nil` 时结束。你可以通过在调用 `next` 时永远不返回 `nil` 来将有限序列转换为无限序列。

让我们创建一个示例，说明这个版本的序列方法可能如何使用。遍历具有嵌套视图或任何嵌套类型列表的视图层次结构是一个使用第二个版本序列函数的完美任务。让我们创建一个具有两个属性的 Item 类。一个名称属性和一个可选的父属性，以跟踪项的所有者。最终所有者将没有父项，这意味着父属性将是 `nil`。

让我们定义一个 Item 类来在我们的示例中使用，以展示这些新概念的使用。

```swift
class Item{ 
    var parent: Item? 
    var name: String = "" 
} 

```

接下来，我们创建一个父项和两个嵌套的子项。`child1` 的父项将是父项本身，而 `child2` 的父项将是 `child1`。

```swift
let parent = Item() 
parent.name = "parent" 

let child1 = Item() 
child1.name = "child1" 
child1.parent = parent 

let child2 = Item() 
child2.name = "child2" 
child2.parent = child1 

```

现在是时候创建我们的序列了。序列需要我们提供两个参数：一个 `state` 参数和一个 `next` 闭包。我将状态设为一个 `Item`，初始值为 `child2`。这样做的原因是我希望从树的最低层叶子节点开始遍历到最终父节点。我们的示例只有三个层级，但在更复杂的示例中可能会有很多层级。至于 `*next*` 参数，我使用了一个期望可变 `Item` 作为其状态的闭包表达式。我的闭包还将返回一个可选的 `Item`。在我们的闭包体中，我使用当前的 `Item`（可变状态参数）来访问 `Item` 的父节点。我更新状态并返回父节点。

```swift
let itemSeq = sequence(state: child2, next: { 
    (next: inout Item)->Item? in 
    let parent = next.parent 
    next = parent != nil ? parent! : next 
    return parent 
}) 

for item in itemSeq{ 
    print("name: \(item.name)") 
} 

```

这里有一些需要注意的问题，我想指出，以便你更好地理解如何为这个序列方法定义自己的下一个闭包。

+   状态参数实际上可以是任何你想要的内容。它是为了帮助你确定序列的下一个元素，并为你提供有关你在序列中位置的相关信息。改进我们上述示例的一个想法是跟踪我们有多少层嵌套。我们可以将状态设为一个元组，其中包含一个表示嵌套层级的整数计数器以及当前项。

+   下一个闭包需要扩展以显示签名。由于 Swift 在闭包方面的表达性和简洁性，你可能会倾向于将 `*next*` 闭包转换为更短的形式并省略签名。除非你的 `*next*` 闭包非常简单，并且你确信编译器能够推断出你的类型，否则不要这样做。使用短闭包格式会使你的代码更难维护，而且当其他人继承它时，你也不会因为风格而得到额外的分数。

+   不要忘记在闭包体中更新你的状态参数。这真的是你了解自己在序列中位置的最佳机会。忘记更新状态可能会在你尝试遍历序列时导致你得到意外的结果。

+   提前明确决定你是在创建有限序列还是无限序列。这一决定在你从下一个闭包返回时是显而易见的。当你期望无限序列时，它并不是坏事，然而，如果你使用 `for…in` 循环遍历这个序列，你可能会得到比你预期的更多，前提是你假设这个循环会结束。

# 集合和索引的新模型 [SE-0065]

Swift 3 引入了一个新的集合模型，将索引遍历的责任从索引转移到集合本身。为了使这一变化适用于集合，Swift 团队引入了四个方面的变化：

+   集合的索引属性可以是实现 *Comparable* 协议的任何类型

+   Swift 移除了区间和范围之间的任何区别；只留下范围

+   私有索引遍历方法现在是公开的

+   Range 的变化使得闭包范围可以无错误地工作

### 注意

您可以在以下链接中阅读提案：[Swift 3 迁移现有类型时的问题](https://github.com/apple/swift-evolution/blob/master/proposals/0065-collections-move-indices.md)

## 介绍 Collection 协议

在 Swift 3 中，Foundation 集合类型如 Arrays、Dictionaries 和 Sets 是实现了新创建的 Collection 协议的泛型类型。这一变化是为了支持集合的遍历。如果您想创建自己的自定义集合，您需要了解 Collection 协议以及它在 Collection 协议层次结构中的位置。我们将介绍新集合模型的重要方面，以帮助您过渡并准备好创建自己的自定义集合类型。

Collection 协议建立在 Sequence 协议之上，提供在集合中使用时访问特定元素的方法。例如，您可以使用集合的`index(_:offsetBy:)`方法返回一个距离参考索引指定距离的索引。

```swift
let numbers = [10, 20, 30, 40, 50, 60] 
let twoAheadIndex = numbers.index(numbers.startIndex, offsetBy: 2) 
print(numbers[twoAheadIndex]) //=> 30   

```

在我们上面的例子中，我们创建了`twoAheadIndex`常量来保存我们的数字集合中距离起始索引两个位置的索引。我们只需使用这个索引通过下标符号从我们的集合中检索值。

## 遵守 Collection 协议

如果您想创建自己的自定义集合，您需要通过声明`startIndex`和`endIndex`属性、一个支持访问您元素的索引和`index(after:)`方法来采用 Collection 协议，以方便遍历您的集合索引。

### 注意

当我们将现有类型迁移到 Swift 3 时，迁移器在转换自定义集合时存在一些已知问题。您很可能会通过检查导入的类型是否符合 Collection 协议来轻松解决编译器问题。

此外，您还需要遵守 Sequence 和 IndexableBase 协议，因为 Collection 协议采用了这两个协议。

```swift
public protocol Collection : Indexable, Sequence { ... } 

```

一个简单的自定义集合可能看起来像以下示例。请注意，我已经将我的`Index`类型定义为`Int`。在 Swift 3 中，您可以将索引定义为任何实现了 Comparable 协议的类型：

```swift
struct MyCollection<T>: Collection{ 
    typealias Index = Int 
    var startIndex: Index 
    var endIndex: Index 

    var _collection: [T] 

    subscript(position: Index) -> T{ 
        return _collection[position] 
    } 

    func index(after i: Index) -> Index { 
        return i + 1 
    } 

    init(){ 
        startIndex = 0 
        endIndex = 0 
        _collection = [] 
    } 

    mutating func add(item: T){ 
        _collection.append(item) 
    } 
} 

var myCollection: MyCollection<String> = MyCollection() 
myCollection.add(item: "Harry") 
myCollection.add(item: "William") 
myCollection[0] 

```

Collection 协议为其大多数方法、Sequence 协议的方法和 IndexableBase 协议的方法提供了默认实现。这意味着您只需要提供一些自己的东西。然而，您可以为您的集合实现尽可能多的其他方法。

## 新的 Range 和相关索引类型

Swift 2 的 `Range<T>`、`ClosedInterval<T>`和`OpenInterval<T>`在 Swift 3 中将被弃用。这些类型将被四种新类型所取代。其中两种新的范围类型支持具有实现`Comparable`协议边界的通用范围：`Range<T>`和`ClosedRange<T>`。其他两种范围类型符合`RandomAccessCollection`。这些类型支持具有实现`Strideable`协议边界的范围。

最后，由于范围现在表示为两个索引的配对，它们不再可迭代。为了保持旧代码的兼容性，Swift 团队引入了一个关联的`Indices`类型，它是可迭代的。此外，创建了三个泛型类型，为每种集合遍历类别提供默认的`Indices`类型。这些泛型是`DefaultIndices<C>`、`DefaultBidirectionalIndices<C>`和`DefaultRandomAccessIndices<C>`；每个都存储其底层集合以进行遍历。

# 快速总结

我在 Swift 3 的集合类型仅用几页纸就涵盖了大量内容。以下是关于集合和索引的要点，请记住。

+   收集类型（内置和自定义）实现了收集协议。

+   遍历集合已移动到 Collection - 索引不再具有该功能。

+   您可以通过采用收集协议来创建自己的集合。您需要实现：

    +   `startIndex`和`endIndex`属性，

    +   下标方法以支持访问您的元素

    +   并且提供了`index(after:)`方法来方便遍历集合的索引。

# Swift 3 的闭包变化

Swift 中的闭包是一段代码块，可以用作函数调用的参数或分配给变量，以便在稍后执行其功能。闭包是 Swift 的核心特性，对于新接触 Swift 的开发者来说很熟悉，因为它们可能会让他们想起其他编程语言中的 lambda 函数。对于 Swift 3，这里将突出两个显著的变化。第一个变化是关于 inout 捕获。第二个变化是将非逃逸闭包作为默认值。

## 限制@noescape 闭包的 inout 捕获 [SE-0035]

在 Swift 2 中，在逃逸闭包中捕获`inout`参数对开发者来说难以理解。一些闭包被分配给变量，然后作为参数传递给函数。如果包含闭包参数的函数从其调用中返回，并且传递的闭包稍后被使用，那么您有一个逃逸闭包。另一方面，如果闭包仅用于传递给它的函数中，并且以后不再使用，那么您有一个非逃逸闭包。这里的区别很重要，因为`inout`参数的修改性质。

当我们将`inout`参数传递给闭包时，可能会因为`inout`参数的存储方式而无法得到预期的结果。`inout`参数被捕获为阴影副本，并且只有在值发生变化时才会写回原始值。这大多数时候都工作得很好。然而，当闭包在稍后时间被调用（即，当它逃逸时），我们不会得到预期的结果。我们的阴影副本无法写回原始值。让我们看一个例子。

```swift
var seed = 10 
let simpleAdderClosure = { (inout seed: Int)->Int in 
    seed += 1 
    return seed * 10 
} 

var result = simpleAdderClosure(&seed)  //=> seed = 11; result = 110 
print(seed) // => 11 

```

在上面的例子中，我们得到了预期的结果。我们创建了一个闭包来增加传入的`inout`参数，然后返回乘以 10 的新参数。当我们调用闭包后检查`seed`的值，我们看到值已经增加到`11`。

在我们的第二个例子中，我们修改我们的闭包，使其返回一个函数而不是仅仅一个`Int`值。我们将我们的逻辑移动到我们定义的返回值闭包中。

```swift
let modifiedClosure = { (inout seed: Int)-> (Int)->Int in 
    return { (Int)-> Int in 
        seed += 1 
        return seed * 10 
    } 
} 

print(seed)  //=> 11 
var resultFn = modifiedClosure(&seed) 
var result = resultFn(1) 
print(seed) // => 11 

```

这次当我们用`seed`值执行`modifiedClosure`时，我们得到一个函数作为结果。执行这个中间函数后，我们检查`seed`的值，发现值没有改变；尽管我们仍在增加`seed`值。

当使用`inout`参数时，这两个微小的语法差异会产生不同的结果。如果没有了解阴影副本的工作原理，就很难理解结果之间的差异。最终，这又是一个你因为允许这个特性保留在语言中而得到更多伤害而不是好处的例子。

### 注意

你可以在以下链接中阅读提案 [`github.com/apple/swift-evolution/blob/master/proposals/0035-limit-inout-capture.md`](https://github.com/apple/swift-evolution/blob/master/proposals/0035-limit-inout-capture.md)

## 解决方案

在 Swift 3 中，编译器现在限制`inout`参数在闭包中的使用为非逃逸（`@noescape`）。如果编译器检测到你的闭包包含`inout`参数时逃逸，你会收到一个错误。

## 将非逃逸闭包设置为默认值 [SE-0103]

在 Swift 的早期版本中，函数参数的类型为闭包时的默认行为是允许逃逸。这很有道理，因为大多数导入到 Swift 中的 Objective-C 块（Swift 中的闭包）都是逃逸的。Objective-C 中的代理模式，作为块实现，由逃逸的代理块组成。那么 Swift 团队为什么要将默认值改为非逃逸呢？让我们通过 Swift 2.2 和 Swift 3 的例子来更好地理解这个变化为什么合理。

在 Swift 2.2 中：

```swift
var callbacks:[String : ()->String] = [:] 
func myEscapingFunction(name:String, callback:()->String){ 
    callbacks[name] = callback 
} 
myEscapingFunction("cb1", callback: {"just another cb"}) 
for cb in callbacks{ 
    print("name: \(cb.0) value: \(cb.1())")  
} 

```

在 Swift 3 中：

```swift
var callbacks:[String : ()->String] = [:] 
func myEscapingFunction(name:String, callback: @escaping ()->String){ 
    callbacks[name] = callback 
} 
myEscapingFunction(name:"cb1", callback: {"just another cb"}) 
for cb in callbacks{ 
    print("name: \(cb.0) value: \(cb.1())")  
} 

```

Swift 团队认为，您可以使用非逃逸闭包编写更好的函数式算法。另一个支持因素是，当使用闭包的`inout`参数时，要求使用非逃逸闭包*[SE-0035]*。综合考虑，这个变化可能对您的代码影响很小。当编译器检测到您正在尝试创建一个逃逸闭包时，您将收到一个警告，提示您可能正在创建一个逃逸闭包。您可以通过添加`@escaping`或通过伴随错误的`fixit`轻松纠正错误。

### 注意

您可以在以下链接中阅读提案[`github.com/apple/swift-evolution/blob/master/proposals/0103-make-noescape-default.md`](https://github.com/apple/swift-evolution/blob/master/proposals/0103-make-noescape-default.md)

# 摘要

在本章中，我们介绍了集合和闭包的变化。我们学习了新的集合协议，它是新集合模型的基础，以及如何在我们的自定义集合中采用该协议。新的集合模型通过将集合遍历从索引移动到集合本身，实现了重大变化。为了支持 Objective-C 的交互性并提供一种机制，使用集合本身来遍历集合项，新的集合模型变化是必要的。至于闭包，我们还探讨了语言转向非逃逸闭包作为默认值的动机。我们还学习了如何在 Swift 3 中正确使用`inout`参数与闭包。在下一章中，我们将介绍更多协议和协议扩展中的类型变化和类型别名。
