# 第六章：映射、过滤和归约

在前面的章节中，我们简要介绍了`map`函数作为内置高阶函数的例子。在本章中，我们将进一步探讨这个主题，并通过示例熟悉 Swift 中的`map`、`flatMap`、`filter`和`reduce`函数。我们还将熟悉诸如模态、函子和应用函子等范畴论概念。

本章将通过代码示例涵盖以下主题：

+   函子

+   应用函子

+   模态

+   映射

+   平铺和扁平化

+   过滤

+   归约

+   应用

+   连接

+   连接高阶函数

+   Zip

+   实际例子

在我们的日常开发中，集合被到处使用，为了能够声明式地使用集合，我们需要像`map`、`filter`和`reduce`这样的手段。在探讨这些内置到 Swift 中的函数之前，让我们探索这些概念的理论背景。

# 函子

函子的名称来源于范畴论。在范畴论中，函子包含诸如`map`函数这样的态射，它转换函子。我们可以将函子视为一种函数式设计模式。

了解范畴论很好，但我们不必这样做，简单来说，函子是我们可以在其上映射的结构。换句话说，函子是任何实现了`map`函数的类型。函子的例子包括`Dictionary`、`Array`、`Optional`和`Closure`类型。每当谈到函子时，我们首先想到的是我们可以调用`map`函数来操作和转换它们。

与其名字不同，这个概念非常简单。我们将在接下来的部分中更详细地讨论`map`函数，并探索函子的用法。

# 应用函子

应用函子的名称也来自范畴论，我们可以将**应用函子**视为一种函数式设计模式。

应用函子是一种配备了将值映射到包含该值的函子实例的函数的函子。应用函子为我们提供了在函子上下文中操作值的能力，例如可选值，而无需解包或对它们的内容进行`map`操作。

假设我们有一个可选函子（一个具有`map`函数的可选值）。我们不能直接在可选值上应用`map`函数，因为我们需要先解包它们。应用函子就派上用场了。它们为函子添加了一个新函数，例如`apply`，使得可以在函子上应用`map`。再次强调，与它的名字不同，这个概念很简单；我们将在接下来的部分中讨论`apply`函数。

# 模态

模态的名称也来自范畴论，我们也可以将模态视为一种函数式设计模式。

Monad 是一种 Functor 类型，它是一种类型，除了 `map` 之外，还实现了 `flatMap` 函数。这很简单，对吧？我们有一个具有额外功能的 Functor，那就是 `flatMap` 实现。所以，任何我们可以对其调用 `map` 和 `flatMap` 函数的类型都是 Monads。在接下来的章节中，我们将讨论 `map` 和 `flatMap` 函数。

到目前为止，我们了解到 Functors 是具有 `map` 函数的结构。Applicative Functors 是具有 `apply` 函数的 Functors，而 Monads 是具有 `flatMap` 函数的 Functors。现在，让我们来谈谈这些重要的函数。

# Map

Swift 内置了一个名为 `map` 的高阶函数，它可以与数组等集合类型一起使用。`map` 函数解决了使用函数转换数组元素的问题。以下示例展示了两种不同的方法来转换一组数字：

```swift
let numbers = [10, 30, 91, 50, 100, 39, 74]
var formattedNumbers: [String] = []

for number in numbers {
    let formattedNumber = "\(number)$"
    formattedNumbers.append(formattedNumber)
}

let mappedNumbers = numbers.map { "\($0)$" }

```

解决这个问题的第一种方法是命令式，使用 for-in 循环遍历集合并转换数组中的每个元素。这种迭代技术被称为外部迭代，因为我们指定了如何迭代。它要求我们显式地从开始到结束按顺序访问元素。此外，它还需要在循环执行任务时创建一个被反复修改的变量。

这个过程容易出错，因为我们可能会错误地初始化 `formattedNumbers`。而不是使用外部迭代技术，我们可以使用内部迭代技术。

没有指定如何遍历元素或声明和使用任何可变变量，Swift 可以确定如何访问所有元素以执行任务，并隐藏这些细节。这种技术被称为内部迭代。

内部迭代方法之一是 `map` 方法。`map` 方法优雅地简化了我们的代码，使其具有声明性。让我们使用 `map` 函数来检查第二种方法：

```swift
let mappedNumbers = numbers.map { "\($0)$" }

```

如此示例所示，我们可以用一行代码实现相同的结果。使用 `map` 的一大好处是我们可以清楚地声明我们试图应用于元素列表的转换。`map` 函数允许我们声明我们想要实现的目标，而不是它的实现方式。这使得阅读和推理我们的代码变得更加简单。

`map` 函数可以应用于任何包含自身内部值或多个值的容器类型。任何提供 `map` 函数的容器都成为 Functor，正如我们之前所看到的。

我们知道 `map` 函数/方法使用的优点以及它的用法。让我们探索它的动态特性并创建一个 `map` 函数。

在 第五章 中，*泛型和关联类型协议*，我们有以下示例：

```swift
func calculate<T>(a: T,
                  b: T,
              funcA: (T, T) -> T,
              funcB: (T) -> T) -> T {

    return funcA(funcB(a), funcB(b))
}

```

`calculate` 函数可以接受 `a`、`b`、`funcA` 和 `funcB` 作为参数。让我们简化这个函数，只使用两个参数并更改返回类型：

```swift
func calculate<T, U>(a: T,
                 funcA: (T) -> U) -> U {

    return funcA(a)
}

```

现在，`calculate` 函数接受类型为 `T` 的 `a` 和将 `T` 转换为 `U` 的 `funcA`。`calculate` 函数返回 `U`。尽管这个函数不适用于数组，但添加数组转换会很容易：

```swift
func calculate<T, U>(a: [T],
                 funcA: ([T]) -> [U]) -> [U] {

    return funcA(a)
}

```

到目前为止，我们有一个 `calculate` 函数，它接受一个泛型类型 `T` 的数组和一个将 `T` 类型的数组转换为 `U` 类型的数组的函数，并最终返回转换后的 `U` 类型的数组。

只需更改函数和参数的名称，我们就可以使这个函数更加通用。所以让我们更改函数和参数的名称：

```swift
func map<T, U>(a: [T], transform: [T] -> [U]) -> [U] {
    return transform(a)
}

```

到目前为止，我们有一个半成品 `map` 函数，它接受一个 `T` 类型的数组，并将其应用于 `transform` 函数，以返回一个转换后的 `U` 类型的数组。

实际上，这个函数什么也不做，映射发生在 transform 中。让我们使这个函数可用并更易于理解：

```swift
func map<ElementInput, ElementResult>(elements: [ElementInput],
  transform: (ElementInput) -> ElementResult) -> [ElementResult] {
    var result: [ElementResult] = []

    for element in elements {
        result.append(transform(element))
    }

    return result
}

```

现在，我们的 `map` 函数接受一个元素数组（范畴论中的域），遍历数组中的每个元素，将其转换，并将其追加到新数组中（范畴论中的陪域）。

结果将是一个 `ElementResult` 类型的数组，它实际上已经转换了输入数组的元素。让我们测试这个函数：

```swift
let numbers = [10, 30, 91, 50, 100, 39, 74]

let result = map(elements: numbers, transform: { $0 + 2 })

```

结果将是 `[12, 32, 93, 52, 102, 41, 76]`。

这个例子表明，通过高阶函数和泛型，我们能够定义像 `map` 这样的函数，这些函数已经是 Swift 语言的一部分。

现在，让我们检查 Swift 中提供的 `map` 函数：

```swift
public func map<T>(@noescape transform: (Self.Generator.Element) -> T) ->
  [T]
```

这个定义与我们的实现非常相似，只是在一些我们将在这里讨论的不同之处。

首先，这是一个可以在数组等集合上调用的方法，因此我们不需要任何如 `[ElementInput]` 这样的输入类型。

其次，`@noescape` 是 Swift 中用于告知函数用户该参数不会比调用持续更长时间的属性。在逃逸场景中，如果函数调度到不同的线程，参数可能会被捕获，以便在需要时在稍后的时间存在。`@noescape` 属性确保这种情况不会发生。

最后，`transform` 是参数的名称。参数的类型声明为 `(Self.Generator.Element) -> T`。这是一个闭包，它接受 `Self.Generator.Element` 类型的参数并返回 `T` 类型的实例。`Self.Generator.Element` 类型与集合中包含的类型相同。

# FlatMap 和 flatten

数组的 `flatMap` 方法可以用来扁平化数组的某一维度的维度。以下示例展示了一个二维数组，换句话说，嵌套数组。在这个数组上调用 `flatMap` 会减少一个维度并将其扁平化，使得结果数组变为 `[1, 3, 5, 2, 4, 6]`：

```swift
let twoDimensionalArray = [[1, 3, 5], [2, 4, 6]]
let oneDimensionalArray = twoDimensionalArray.flatMap { $0 }

```

在这个例子中，`flatMap` 返回一个包含映射转换结果的 `Array`，我们可以通过在数组上调用 flatten 然后进行 map 来达到相同的结果，如下所示：

```swift
let oneDimensionalArray = twoDimensionalArray.flatten().map { $0 }

```

为了能够将每个元素转换为一个数组，我们需要将 `map` 方法作为闭包提供给 `flatMap` 方法，如下所示：

```swift
let transofrmedOneDimensionalArray = twoDimensionalArray.flatMap { 
      $0.map { $0 + 2 } 
}
```

结果将是 `[3, 5, 7, 4, 6, 8]`。

可以通过以下方式获得相同的结果：

```swift
let oneDimensionalArray = twoDimensionalArray.flatten().map { $0 + 2 }

```

让我们通过一个三维 `Array` 的例子来检查另一个例子：

```swift
let threeDimensionalArray = [[1, [3, 5]], [2, [4, 6]]]
let twoDimensionalArray = threeDimensionalArray.flatMap { $0 }

```

结果数组将是 `[1, [3, 5], 2, [4, 6]]`。

因此，`flatMap` 和 `flatten` 只能扁平化一个维度，要处理更多维度和转换，我们需要相应地多次调用 `flatMap` 和 `map` 方法。

我们还知道 `twoDimensionalArray` 和 `threeDimensionalArray` 是 Monads，因为我们可以在它们上调用 `map` 和 `flatMap`。

# 过滤

`filter` 函数接收一个函数，该函数给定 `Array` 中的一个元素，返回一个 `Bool` 值，指示该元素是否应包含在结果 `Array` 中。在 Swift 中，`filter` 方法声明如下：

```swift
public func filter(@noescape includeElement: 
  (Self.Generator.Element) -> Bool) -> [Self.Generator.Element]
```

其定义与 `map` 方法类似，但有以下区别：

+   `filter` 函数接收一个闭包，该闭包接收自身元素并返回一个 `Bool` 值

+   `filter` 方法的输出将是一个其自身类型的数组

让我们检查以下代码以了解其工作原理：

```swift
let numbers = [10, 30, 91, 50, 100, 39, 74]
let evenNumbers = numbers.filter { $0 % 2 == 0 }

```

结果的 `evenNumbers` 数组将是 `[10, 30, 50, 100, 74]`。

让我们亲自实现 `filter` 函数。实际上，它的实现将与 `map` 的实现类似，只是它不需要指定第二个泛型来指定陪域。相反，它有条件地将原始元素添加到新的 `Array` 中：

```swift
func filter<Element> (elements: [Element], 
                      predicate:(Element -> Bool)) -> [Element] {
    var result = [Element]()
    for element in elements {
        if predicate(element) {
            result.append(element)
        }
    }
    return result
}

```

`filter` 函数遍历 `Array` 中的每个元素，并对其应用谓词。如果谓词函数的结果为 `true`，则 `element` 被添加到我们的新 `Array` 中。我们可以如下测试我们的 `filter` 函数：

```swift
let filteredArray = filter(elements: numbers) { $0 % 2 == 0 }

```

结果数组将是 `[10, 30, 50, 100, 74]`，这与 Swift 提供的 `filter` 方法相同。

# 减少

`reduce` 函数将列表缩减为一个单一值。通常被称为 `fold` 或 `aggregate`，它接受两个参数：一个起始值和一个函数。

函数接受一个累计总和和列表中的一个元素作为参数，并返回一个通过组合列表中的元素创建的值。

与 `map`、`filter` 和 `flatMap` 不同，这些方法会返回相同类型的结果，`reduce` 会改变类型。换句话说，`map`、`filter` 和 `flatMap` 会将 `Array` 转换为改变了的 `Array`。这与 `reduce` 不同，因为它可以将数组转换为元组或单个值。

Swift 为数组提供了 `reduce` 方法，其定义如下：

```swift
func reduce<T>(initial: T, @noescape combine: (T, 
  Self.Generator.Element) -> T) -> T
```

如果我们在 `numbers` 数组上使用 `reduce` 方法，这次调用的结果变为 `394`：

```swift
let total = numbers.reduce(0) { $0 + $1 }

```

我们也可以将 `reduce` 称为 `+` 操作符，因为在 Swift 中它是一个函数：

```swift
let total = numbers.reduce(0, combine: +)

```

与 `map` 和 `filter` 方法一样，开发 `reduce` 函数也很简单：

```swift
func reduce<Element, Value>(elements: [Element],
                            initial: Value,
                            combine: (Value, Element) -> Value) ->     Value {
    var result = initial

    for element in elements {
        result = combine(result, element)
    }

    return result
}

```

我们可以通过以下调用获得相同的结果（`394`）：

```swift
let total = reduce(elements: numbers, initial: 0) { $0 + $1 }

```

`reduce` 方法可以与其他类型一起使用，例如字符串数组。

## 从 `reduce` 的角度来理解 `map` 函数

减少模式非常强大，以至于任何遍历列表的其他函数都可以用它的术语来指定。让我们用 `reduce` 来开发一个 `map` 函数：

```swift
func mapIntermsOfReduce<Element, ElementResult>(elements: [Element],
  transform: Element -> ElementResult) -> [ElementResult] {
    return reduce(elements: elements, initial: [ElementResult]()) {
        $0 + [transform( $1 )]
    }
}

let result = mapIntermsOfReduce(elements: numbers, transform: { $0 + 2 })
```

结果与我们在本章早期开发的 `map` 函数的结果相同。这是一个理解 `reduce` 基础的好例子。

在函数体中，我们提供 `elements`，一个空的初始 `ElementResult` 数组，最后提供一个闭包来组合元素。

## 从 `reduce` 的角度来理解 `filter` 函数

也可以开发一个从 `reduce` 的角度来理解的 `filter` 函数：

```swift
func filterIntermsOfReduce<Element>(elements: [Element],
                                    predicate: Element -> Bool) -> [Element] {
    return reduce(elements: elements, initial: []) {
        predicate($1) ? $0 + [ $1 ] : $0
    }
}

let result = filterIntermsOfReduce(elements: numbers) { $0 % 2 == 0 }
```

再次，结果与之前开发的 `filter` 函数相同。

在函数体中，我们提供 `elements`，一个空的初始 `ElementResult` 数组，最后提供 `predicate` 作为组合器。

## 从 `reduce` 的角度来理解 `flatMap` 函数

为了理解 `reduce` 的强大之处，我们可以用 `reduce` 来实现 `flatMap` 函数：

```swift
func flatMapIntermsOfReduce<Element>(elements: [Element],
  transform: (Element) -> Element?) -> [Element] {
    return reduce(elements: elements, initial: []) {
        guard let transformationResult = transform($1) else {
            return $0
        }
        return $0 + [transformationResult]
    }
}

let anArrayOfNumbers = [1, 3, 5]
let oneDimensionalArray = flatMapIntermsOfReduce(elements:
  anArrayOfNumbers) { $0 + 5 }
```

## 从 `reduce` 的角度来理解 `flatten` 函数

最后，让我们用 `reduce` 来实现 `flatten` 函数：

```swift
func flattenIntermsOfReduce<Element>(elements: [[Element]]) -> [Element] {
    return elements.reduce([]) { $0 + $1 }
}

```

此函数接受一个二维数组并将其转换为单维数组。让我们测试这个函数：

```swift
let flattened = flattenIntermsOfReduce(elements: [[1, 3, 5], [2, 4, 6]])

```

结果将是 `[1, 3, 5, 2, 4, 6]`。

# Apply

Apply 是一个将函数应用于一系列参数的函数。

不幸的是，Swift 并没有在 `Arrays` 上提供任何 `apply` 方法。为了能够实现 Applicative Functors，我们需要开发 `apply` 函数。以下代码展示了 `apply` 函数的一个简单版本，它只有一个参数：

```swift
func apply<T, V>(fn: [T] -> V, args: [T]) -> V {
    return fn(args)
}

```

`apply` 函数接受一个函数和一个任意类型的数组，并将该函数应用于数组的第一个元素。让我们测试这个函数如下：

```swift
let numbers = [1, 3, 5]

func incrementValues(a: [Int]) -> [Int] {
    return a.map { $0 + 1 }
}

let applied = apply(fn: incrementValues, args: numbers)
```

# Join

`join` 函数接受一个对象数组，并用提供的分隔符将它们连接起来。以下示例展示了 `join` 的一个简单版本：

```swift
func join<Element: Equatable>(elements: [Element],
                              separator: String) -> String {
    return elements.reduce("") {
        initial, element in
        let aSeparator = (element == elements.last) ? "" : separator
        return "\(initial)\(element)\(aSeparator)"
    }
}

```

此函数接受一个带有分隔符的数组，将 `Array` 中的元素连接起来，并提供一个单一的 `String`。我们可以如下测试它：

```swift
let items = ["First", "Second", "Third"]
let commaSeparatedItems = join(elements: items, separator: ", ")

```

结果将是 `"First, Second, Third"`。

# 链式调用高阶函数

到目前为止，我们学习了不同的函数，并为每个函数提供了一些示例。让我们看看我们是否可以将它们结合起来解决我们在日常应用开发中可能遇到的问题。

假设我们需要从后端系统接收一个对象，如下所示：

```swift
struct User {
    let name: String
    let age: Int
}

let users = [
    User(name: "Fehiman", age: 60),
    User(name: "Negar", age: 30),
    User(name: "Milo", age: 1),
    User(name: "Tamina", age: 6),
    User(name: "Neco", age: 30)
]

```

然后我们需要计算 `users` 数组中年龄的总和。我们可以使用 `map` 和 `reduce` 函数的组合来计算 `totalAge`，如下所示：

```swift
let totalAge = users.map { $0.age }.reduce(0) { $0 + $1 }

```

我们能够通过链式调用 `map` 和 `reduce` 方法来实现这一点。

# Zip

`zip` 函数是由 Swift 标准库提供的，它创建了一个由两个底层序列组成的序列对，其中第 *i^(th)* 对的元素是每个底层序列的第 *i^(th)* 个元素。

例如，在以下示例中，`zip` 接受两个 `Arrays` 并创建这两个数组的配对：

```swift
let alphabeticNumbers = ["Three", "Five", "Nine", "Ten"]
let zipped = zip(alphabeticNumbers, numbers).map { $0 }

```

`zipped` 的值将是 `[("Three", 3), ("Five", 5), ("Nine", 9), ("Ten", 10)]`。

# 实际例子

让我们探索一些高阶函数的实际例子。

## 数组的和

我们可以使用 `reduce` 来计算数字列表的总和，如下所示：

```swift
let listOfNumbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
let sumOfNumbers = reduce(elements: listOfNumbers, initial: 0, combine: +)

print(sumOfNumbers)

```

预期结果是 `55`。

## 数组的乘积

我们可以使用 `reduce` 来计算数组值的乘积，如下所示：

```swift
let productOfNumbers = reduce(elements: listOfNumbers, initial: 1,
  combine: *)

print(productOfNumbers)

```

预期结果是 `3628800`。

## 从数组中移除 `nil` 值

我们可以使用 `flatMap` 从可选数组中获取值并移除 `nil` 值：

```swift
let optionalArray: [String?] = ["First", "Second", nil, "Fourth"]
let nonOptionalArray = optionalArray.flatMap { $0 }

print(nonOptionalArray)

```

预期结果是 `["First", "Second", "Fourth"]`。

## 数组中的重复项移除

我们可以使用 `reduce` 来从数组中移除重复元素，如下所示：

```swift
let arrayWithDuplicates = [1, 1, 2, 3, 3, 4, 4, 5, 6, 7]

arrayWithDuplicates.reduce([]) { (a: [Int], b: Int) -> [Int] in
    if a.contains(b) {
        return a
    } else {
        return a + [b]
    }
}

```

预期结果是 `[1, 2, 3, 4, 5, 6, 7]`。

## 分区数组

我们可以使用 `reduce` 来根据特定标准对数组进行分区。例如，在以下示例中，我们将 `numbersToPartition` 数组分为两个部分，将所有偶数保留在左侧部分：

```swift
typealias Accumlator = (lPartition: [Int], rPartition: [Int])

func partition(list: [Int], criteria: (Int) -> Bool) -> Accumlator {
    return list.reduce((lPartition: [Int](), rPartition: [Int]())) {
        (accumlator: Accumlator, pivot: Int) -> Accumlator in
        if criteria(pivot) {
            return (lPartition: accumlator.lPartition + [pivot],
              rPartition: accumlator.rPartition)
        } else {
            return (rPartition: accumlator.rPartition + [pivot],
              lPartition: accumlator.lPartition)
        }
    }
}

let numbersToPartition = [3, 4, 5, 6, 7, 8, 9]
partition(list: numbersToPartition) { $0 % 2 == 0 }

```

我们可以将此函数泛化如下：

```swift
func genericPartition<T>(list: [T],
                         criteria: (T) -> Bool) -> (lPartition: 
[T], 
                         rPartition: [T]) {
    return list.reduce((lPartition: [T](), rPartition: [T]())) {
        (accumlator: (lPartition: [T], rPartition: [T]), pivot: T) -> (
          lPartition: [T], rPartition: [T]) in
        if criteria(pivot) {
            return (lPartition: accumlator.lPartition + [pivot],
              rPartition: accumlator.rPartition)
        } else {
            return (rPartition: accumlator.rPartition + [pivot],
              lPartition: accumlator.lPartition)
        }
    }
}

let doublesToPartition = [3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0]
print(genericPartition(list: doublesToPartition) { $0.truncatingRemainder(dividingBy: 2.0) == 0 })
```

# 摘要

在本章中，我们从范畴论概念如函子（Functor）、应用函子（Applicative Functor）和单子（Monad）开始，探讨了高阶函数如 `map`、`filter`、`flatMap`、`flatten` 和 `reduce`。然后，我们检查了 Swift 提供的高阶函数版本，并实现了自己的简单版本。此外，我们还根据 `reduce` 函数开发了 `map`、`filter`、`flatMap` 和 `flatten` 函数。

然后，我们继续使用 `apply`、`join` 和 `zip` 函数，并介绍了高阶函数的链式调用。

最后，我们探讨了高阶函数的一些实际例子，例如从数组中移除 `nil` 值、移除重复项和分区数组。

这些函数将成为我们日常开发工具箱中的优秀工具，用于解决各种不同类型的问题。

在下一章中，我们将熟悉可选类型，并讨论处理它们的非函数式和函数式方法。
