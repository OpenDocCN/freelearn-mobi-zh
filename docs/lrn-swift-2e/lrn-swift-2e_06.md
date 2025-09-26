# 第六章. 让 Swift 为你工作 – 协议和泛型

如我们在第二章中学习的，*构建块 – 变量、集合和流程控制*，Swift 是一种强类型语言，这意味着每份数据都必须有一个类型。我们不仅可以利用这一点来减少代码的混乱，还可以利用它让编译器为我们捕获错误。我们越早捕获错误，越好。除了最初不编写它们之外，我们最早可以捕获错误的地方是当编译器报告错误时。

Swift 提供的两个大工具，用于实现这一功能，被称为**协议**和**泛型**。它们都利用类型系统来使编译器清楚我们的意图，以便它能为我们捕获更多错误。

在本章中，我们将涵盖以下主题：

+   协议

+   泛型

+   扩展现有的泛型

+   扩展协议

+   使用协议和泛型

# 协议

我们将要查看的第一个工具是**协议**。协议本质上是一种类型可以签署的合同，指定它将向其他组件提供一定的接口。这种关系比子类与其超类之间的关系要松散得多。协议不会为它们实现的类型提供任何实现。相反，一个类型可以以任何它们喜欢的方式实现它们。

让我们看看我们如何定义一个协议，以便更好地理解它们。

## 定义一个协议

假设我们有一些需要与字符串集合交互的代码。我们实际上并不关心它们存储的顺序，我们只需要能够在容器内部添加和枚举元素。一个选项是简单地使用数组，但数组的功能远远超出了我们的需求。如果我们后来决定我们更愿意从文件系统中写入和读取元素呢？此外，如果我们想编写一个在变得非常大时智能地开始使用文件系统的容器呢？我们可以通过定义一个字符串容器协议来使我们的代码足够灵活，以便在未来做到这一点，这个协议是一个松散的合同，定义了我们希望它执行的操作。这个协议可能看起来类似于以下代码：

```swift
protocol StringContainer {
    var count: Int { get }
    mutating func addString(string: String)
    func enumerateStrings(handler: (string: String) -> Void)
}
```

预计，协议是通过使用 `protocol` 关键字定义的，类似于类或结构体。它还允许你指定计算属性和方法。你不能声明存储属性，因为无法直接创建协议的实例。你只能创建实现协议的类型实例。此外，你可能注意到，计算属性或方法都没有提供实现。在协议中，你只提供接口。

由于协议不能自行初始化，因此在我们创建实现它们的类型之前，它们是无用的。让我们看看我们如何创建一个实现我们的 `StringContainer` 协议的类型。

## 实现一个协议

类型“签署”协议的合同的方式与类从另一个类继承的方式相同，只是结构和枚举也可以实现协议：

```swift
struct StringBag: StringContainer {
    // Error: Type 'StringBag' does not conform to protocol 'StringContainer'
}
```

如您所见，一旦一个类型声明实现了特定的协议，如果它没有通过实现协议中定义的所有内容来履行合同，编译器将会报错。为了满足编译器的要求，我们现在必须实现`count`计算属性、修改函数`addString:`和函数`enumerateStrings:`，正如它们在协议中定义的那样。我们将通过在内部使用数组来持有我们的值来完成这项工作：

```swift
struct StringBag: StringContainer {
    var strings = [String]()
    var count: Int {
        return self.strings.count
    }

    mutating func addString(string: String) {
        self.strings.append(string)
    }

    func enumerateStrings(handler: (string: String) -> Void) {
        for string in self.strings {
            handler(string: string)
        }
    }
}
```

`count`属性将始终只返回我们的`strings`数组中的元素数量。`addString:`方法可以简单地添加字符串到我们的数组中。最后，我们的`enumerateString:`方法只需要遍历我们的数组，并对每个元素调用处理程序。

到目前为止，编译器已经满意地认为`StringBag`正在履行其与`StringContainer`协议的合同。

现在，我们可以类似地创建一个实现`StringContainer`协议的类。这一次，我们将使用内部字典而不是数组来实现它：

```swift
class SomeSuperclass {}
class StringBag2: SomeSuperclass, StringContainer {
    var strings = [String:Void]()
    var count: Int {
        return self.strings.count
    }

    func addString(string: String) {
        self.strings[string] = ()
    }

    func enumerateStrings(handler: (string: String) -> Void) {
        for string in self.strings.keys {
            handler(string: string)
        }
    }
}
```

在这里，我们可以看到，一个类可以同时从超类继承并实现一个协议。超类始终必须排在列表的第一位，但你可以实现尽可能多的协议，每个协议之间用逗号分隔。实际上，结构和枚举也可以实现多个协议。

在这个实现中，我们对字典做了一些稍微奇怪的事情。我们定义它没有值；它只是一个键的集合。这允许我们存储字符串，而不考虑它们的顺序。

现在，当我们创建实例时，我们可以将任何实现我们的协议的类型的任何实例分配给定义为我们的协议的变量，就像我们可以用超类一样：

```swift
var someStringBag: StringContainer = StringBag()
someStringBag.addString("Sarah")
someStringBag = StringBag2()
someStringBag.addString("Sarah")
```

当一个变量使用我们的协议作为其类型被定义时，我们只能通过协议定义的接口与之交互。这是一种很好的抽象实现细节并创建更灵活代码的方法。通过对我们想要使用的类型施加较少的限制，我们可以轻松地更改我们的代码，而不会影响我们使用它的方式。协议提供了与超类相同的利益，但以一种更加灵活和全面的方式，因为它们可以被所有类型实现，并且一个类型可以实现无限数量的协议。超类相对于协议的唯一好处是，超类与其子类共享它们的实现。

## 使用类型别名

可以使用一个名为**类型别名**的功能来使协议更加灵活。它们作为稍后将在实现协议时定义的类型的一个占位符。例如，我们不需要创建一个专门包含字符串的接口，我们可以创建一个可以容纳任何类型值的容器接口，如下所示：

```swift
protocol Container {
    typealias Element

    mutating func addElement(element: Element)
    func enumerateElements(handler: (element: Element) -> Void)
}
```

如你所见，此协议使用关键字 `typealias` 创建了一个名为 `Element` 的类型别名。它实际上并没有指定一个真实类型；它只是为稍后定义的类型提供一个占位符。在我们之前使用字符串的所有地方，我们只需将其称为 `Element`。

现在，我们可以创建另一个使用新的 `Container` 协议和类型别名而不是 `StringContainer` 协议的字符串包。为此，我们不仅需要实现每个方法，还需要为类型别名提供一个定义，如下所示：

```swift
struct StringBag3: Container {
    typealias Element = String

    var elements = [Element:Void]()

    var count: Int {
        return elements.count
    }

    mutating func addElement(element: Element) {
        self.elements[element] = ()
    }

    func enumerateElements(handler: (element: Element) -> Void) {
        for element in self.elements.keys {
            handler(element: element)
        }
    }
}
```

使用此代码，我们指定了 `Element` 类型别名应该为此实现是字符串，使用等号 (`=`)。此代码继续使用类型别名来表示所有属性和方法，但你也可以使用字符串，因为它们现在实际上是同一件事。

使用类型别名实际上使我们能够轻松地创建另一个可以存储整数而不是字符串的结构：

```swift
struct IntBag: Container {
    typealias Element = Int

    var elements = [Element:Void]()

    var count: Int {
        return elements.count
    }

    mutating func addElement(element: Element) {
        self.elements[element] = ()
    }

    func enumerateElements(handler: (element: Element) -> Void) {
        for element in self.elements.keys {
            handler(element: element)
        }
    }
}
```

这两段代码之间的唯一区别在于，在第二种情况下，类型别名被定义为整数而不是字符串。我们可以使用复制和粘贴来创建几乎任何类型的容器，但像往常一样，大量复制和粘贴是更好的解决方案的迹象。此外，你可能注意到，我们新的 `Container` 协议本身并不是特别有用，因为使用我们现有的技术，我们无法将一个变量视为一个 `Container`。如果我们将与实现此协议的实例交互，我们需要知道它将类型别名分配给了什么类型。

Swift 提供了一个名为 **泛型** 的工具来解决这两个问题。

# 泛型

泛型非常类似于类型别名。区别在于泛型的确切类型是由其使用的上下文决定的，而不是由实现类型决定的。这也意味着泛型只有一个实现，必须支持所有可能的类型。让我们先定义一个泛型函数。

## 泛型函数

在 第五章 中，*现代范式 – 闭包和函数式编程*，我们创建了一个函数，帮助我们找到通过测试的数组中的第一个数字：

```swift
func firstInNumbers(
    numbers: [Int],
    passingTest: (number: Int) -> Bool
    ) -> Int?
{
    for number in numbers {
        if passingTest(number: number) {
            return number
        }
    }
    return nil
}
```

如果我们只处理整数数组，这将非常棒，但显然能够用其他类型做这件事会更有帮助。事实上，我敢说，对所有类型都这样做会更好。我们通过使我们的函数泛型来实现这一点。泛型函数的声明方式与普通函数类似，但在函数名称的末尾包含一个逗号分隔的占位符列表，放在尖括号 (`<>`) 内，如下所示：

```swift
func firstInArray<ValueType>(
    array: [ValueType],
    passingTest: (value: ValueType) -> Bool
    ) -> ValueType?
{
    for value in array {
        if passingTest(value: value) {
            return value
        }
    }
    return nil
}
```

在这个函数中，我们声明了一个名为`ValueType`的单个占位符。就像类型别名一样，我们可以在实现中继续使用这个类型。这将代表一个在我们要使用函数时确定的单个类型。你可以想象在这个代码中插入`String`或任何其他类型而不是`ValueType`，它仍然可以工作。

我们使用这个函数的方式与任何其他函数类似，如下所示：

```swift
var strings = ["This", "is", "a", "sentence"]
var numbers = [1, 1, 2, 3, 5, 8, 13]
firstInArray(strings, passingTest: {$0 == "a"}) // "a"
firstInArray(numbers, passingTest: {$0 > 10}) // 13
```

在这里，我们使用了`firstInArray:passingTest:`，同时使用了字符串数组和数字数组。编译器根据我们传递给函数的变量来确定要替换占位符的类型。在第一种情况下，`strings`是一个`String`数组。它将这个与`[ValueType]`进行比较，并假设我们想要用`String`替换`ValueType`。在第二种情况下，我们的`Int`数组发生同样的情况。

那么，如果我们闭包中使用的类型与我们传递的数组类型不匹配会发生什么？

```swift
firstInArray(numbers, passingTest: {$0 == "a"}) // Cannot convert
// value of type '[Int]' to expected argument type'[_]'
```

正如你所见，我们得到了一个错误，类型不匹配。

你可能已经注意到，我们实际上之前已经使用过泛型函数。我们在第五章中查看的所有内置函数，如`map`和`filter`都是泛型的；它们可以与任何类型一起使用。

我们甚至已经体验过泛型类型。数组和字典也是泛型的。Swift 团队不需要为容器内可能使用的每个类型编写新的数组和字典实现；他们创建了泛型类型。

## 泛型类型

与泛型函数类似，泛型类型就像一个普通类型一样定义，但它在名称的末尾有一个占位符列表。在本章早些时候，我们为字符串和`integers`创建了我们的容器。让我们创建这些容器的泛型版本，如下所示：

```swift
struct Bag<ElementType> {
    var elements = [ElementType]()

    mutating func addElement(element: ElementType) {
        self.elements.append(element)
    }

    func enumerateElements(
        handler: (element: ElementType) -> ()
        )
    {
        for element in self.elements {
            handler(element: element)
        }
    }
}
```

这个实现看起来与我们的类型别名版本相似，但我们使用的是`ElementType`占位符。

虽然泛型函数的占位符是在函数调用时确定的，但泛型类型的占位符是在初始化新实例时确定的：

```swift
var stringBag = Bag(elements: ["This", "is", "a", "sentence"])
var numberBag = Bag(elements: [1, 1, 2, 3, 5, 8, 13])
```

与一个通用实例的所有未来交互都必须使用相同的类型来为其占位符。这实际上是泛型之美之一，编译器为我们做了工作。如果我们创建了一个类型的实例，却意外地尝试将其用作不同类型，编译器不会允许我们这样做。这种保护在许多其他编程语言中都不存在，包括苹果之前使用的语言：Objective-C。

考虑一个有趣的案例，如果我们尝试用一个空数组初始化一个包：

```swift
var emptyBag = Bag(elements: []) // Cannot invoke initilaizer for
// type 'Bag<_>' with an argument list of type '(elements: [_])'
```

正如你所见，我们得到了一个错误，编译器无法确定要分配给我们的泛型占位符的类型。我们可以通过给我们要分配的泛型一个显式类型来解决这个问题：

```swift
var emptyBag: Bag<String> = Bag(elements: [])
```

这真是太好了，因为编译器不仅可以根据我们传递给它们的变量来确定泛型占位符类型，还可以根据我们使用结果的方式来确定类型。

我们已经看到了如何以强大的方式使用泛型。我们解决了在类型别名部分讨论的第一个问题，即针对不同类型复制粘贴大量实现。然而，我们还没有弄清楚如何解决第二个问题：我们如何编写一个泛型函数来处理`Container`协议中的任何类型？答案是我们可以使用**类型约束**。

## 类型约束

在我们直接跳入解决问题之前，让我们先看看类型约束的简单形式。

### 协议约束

假设我们想编写一个函数，该函数可以使用相等性检查来确定实例在数组中的索引。我们的第一次尝试可能看起来像以下代码：

```swift
func indexOfValue<T>(value: T, inArray array: [T]) -> Int? {
    var index = 0
    for testValue in array {
        if testValue == value { // Error: Cannot invoke '=='
            return index
        }
        index++
    }
    return nil
}
```

在这次尝试中，我们遇到了无法调用相等运算符（`==`）的错误。这是因为我们的实现必须适用于可能分配给我们的占位符的任何可能的类型。并不是 Swift 中的每个类型都可以进行相等性测试。为了解决这个问题，我们可以使用类型约束来告诉编译器，我们只想允许我们的函数以支持相等运算的类型调用。我们通过要求占位符实现一个协议来添加类型约束。在这种情况下，Swift 提供了一个名为`Equatable`的协议，我们可以使用：

```swift
func indexOfValue<T: Equatable>(
    value: T,
    inArray array: [T]
    ) -> Int?
{
    var index = 0
    for testValue in array {
        if testValue == value {
            return index
        }
        index++
    }
    return nil
}
```

类型约束看起来类似于在占位符名称后面使用冒号（`:`）来使用协议实现类型。现在，编译器已经满意，每个可能的类型都可以使用相等运算符进行比较。如果我们尝试用非可比较的类型调用这个函数，编译器会生成一个错误：

```swift
class MyType {}
var typeList = [MyType]()
indexOfValue(MyType(), inArray: typeList)
// Cannot convert value of type '[MyType]' to expected
// argument type '[_]'
```

这又是编译器能帮助我们避免错误的一个例子。

我们也可以向我们的泛型类型添加类型约束。例如，如果我们尝试在没有约束的情况下使用我们的字典实现创建一个包，我们会得到一个错误：

```swift
struct Bag2<ElementType> {
    var elements: [ElementType:Void]
    // Type 'ElementType' does not conform to protocol 'Hashable'
}
```

这是因为字典的键有一个约束，它必须是`Hashable`。字典定义为`struct Dictionary<Key : Hashable, Value>`。`Hashable`基本上意味着该类型可以用整数表示。实际上，如果我们把`Hashable`写在 Xcode 中，然后按住*Command*键点击它，就会带我们到`Hashable`的定义，其中包含注释解释两个相等对象的哈希值必须相同。这对`Dictionary`的实现方式很重要。因此，如果我们想能够在字典中将我们的元素作为键存储，我们必须也添加`Hashable`约束：

```swift
struct Bag2<ElementType: Hashable> {
    var elements: [ElementType:Void]

    mutating func addElement(element: ElementType) {
        self.elements[element] = ()
    }

    func enumerateElements(
        handler: (element: ElementType) -> ()
        )
    {
        for element in self.elements.keys {
            handler(element: element)
        }
    }
}
```

现在编译器很高兴，我们可以开始使用我们的`Bag2`结构体与任何是`Hashable`类型的类型了。我们接近解决我们的`Container`问题，但我们需要对`Container`类型别名的类型进行约束，而不是对`Container`本身进行约束。为了做到这一点，我们可以使用一个`where`子句。

### `where`子句用于协议

在定义了每个占位符类型之后，你可以指定任意数量的`where`子句。它们允许你表示更复杂的关系。如果我们想编写一个函数来检查我们的容器是否包含特定的值，我们可以要求元素类型是可比较的：

```swift
func container<C: Container where C.Element: Equatable>(
    container: C,
    hasElement element: C.Element
    ) -> Bool
{
    var hasElement = false
    container.enumerateElements { testElement in
        if element == testElement {
            hasElement = true
        }
    }
    return hasElement
}
```

在这里，我们指定了一个必须实现`Container`协议的占位符`C`；它还必须有一个是`Equatable`类型的`Element`类型。

有时我们可能还想要在多个占位符之间强制建立关系。为了做到这一点，我们可以在`where`子句中使用等式测试。

### 等式`where`子句

如果我们想编写一个函数，可以将一个容器合并到另一个容器中，同时仍然允许确切的类型变化，我们可以编写一个函数，该函数要求容器持有相同的值：

```swift
func merged<C1: Container, C2: Container where C1.Element == C2.Element>(
    lhs: C1,
    rhs: C2
    ) -> C1
{
    var merged = lhs
    rhs.enumerateElements { element in
        merged.addElement(element)
    }
    return merged
}
```

在这里，我们指定了两个不同的占位符：`C1`和`C2`。它们都必须实现`Container`协议，并且它们还必须包含相同的`Element`类型。这允许我们将第二个容器的元素添加到我们最终返回的第一个容器的副本中。

现在我们知道了如何创建自己的泛型函数和类型，让我们看看我们如何可以扩展现有的泛型。

# 扩展泛型

我们可能想要扩展的两个主要泛型是数组和字典。这两个是最突出的容器，由 Swift 提供，几乎在每一个应用中都被使用。一旦你理解扩展本身不需要是泛型的，扩展泛型类型就很简单了。

## 向所有泛型形式添加方法

知道数组被声明为`struct Array<Element>`，你扩展数组的第一个直觉可能看起来像这样：

```swift
extension Array<Element> { // Use of undeclared type 'Element'
    // ...
}
```

然而，正如你所看到的，你会得到一个错误。相反，你可以简单地省略占位符指定，仍然可以在你的实现中使用`Element`占位符。你的另一个直觉可能是将`Element`声明为你的单个方法的占位符：

```swift
extension Array {
    func someMethod<Element>(element: Element) {
        // ...
    }
}
```

这更危险，因为编译器不会检测到错误。这是错误的，因为你实际上是在方法内部声明一个新的占位符`Element`。这个新的`Element`与在`Array`中定义的`Element`没有任何关系。例如，如果你尝试将一个参数与方法的元素与数组的元素进行比较，你可能会得到一个令人困惑的错误：

```swift
extension Array {
    mutating func addElement<Element>(element: Element) {
        self.append(element)
        // Cannot invoke 'append' with argument list
        // of type '(Element)'
    }
}
```

这是因为在 `Array` 中定义的 `Element` 不能保证与在 `addElement:` 中定义的新 `Element` 完全相同。你可以在泛型类型的方法中声明额外的占位符，但最好给它们唯一的名称，以免隐藏类型的占位符版本。

现在我们已经理解了这一点，让我们添加一个扩展到数组中，允许我们测试它是否包含通过测试的元素：

```swift
extension Array {
    func hasElementThatPasses(
        test: (element: Element) -> Bool
        ) -> Bool
    {
        for element in self {
            if test(element: element) {
                return true
            }
        }
        return false
    }
}
```

如您所见，我们在扩展中继续使用占位符 `Element`。这允许我们为数组中的每个元素调用传入的测试闭包。现在，如果我们想能够添加一个方法来检查元素是否存在，使用等号运算符，会发生什么问题？我们将遇到的问题是数组没有对 `Element` 应用类型约束，要求它必须是 `Equatable`。为了做到这一点，我们可以在我们的扩展中添加一个额外的约束。

## 仅向泛型类型的一定实例添加方法

扩展的约束以 `where` 子句的形式编写，如下所示：

```swift
extension Array where Element: Equatable {
    func containsElement(element: Element) -> Bool {
        for testElement in self {
            if testElement == element {
                return true
            }
        }
        return false
    }
}
```

这里我们添加了一个约束，保证我们的元素是可比较的。这意味着我们只能在这个方法上调用具有可比较元素的数组：

```swift
[1,2,3,4,5].containsElement(4) // true
class MyType {}
var typeList = [MyType]()
typeList.containsElement(MyType()) // Type 'MyType' does not
// conform to protocol 'Equtable'
```

再次，Swift 正在保护我们免受意外尝试在它不起作用的数组上调用此方法的风险。

这些是我们必须与泛型一起使用的构建块。然而，我们实际上还有一个尚未讨论的协议特性，它与泛型结合得非常好。

## 扩展协议

我们首先在第三章 *一次一件——类型、作用域和项目* 中讨论了如何扩展现有类型。在 Swift 2 中，苹果公司增加了扩展协议的能力。这有一些令人着迷的后果，但在我们深入探讨这些之前，让我们先看看如何向 `Comparable` 协议添加一个方法的示例：

```swift
extension Comparable {
    func isBetween(a: Self, b: Self) -> Bool {
        return a < self && self < b
    }
}
```

这将为所有实现 `Comparable` 的类型添加一个方法。这意味着它现在将可用于任何可比较的内置类型以及我们自己的任何可比较类型：

```swift
6.isBetween(4, b: 7) // true
"A".isBetween("B", b: "Z") // false
```

这是一个非常强大的工具。实际上，这就是 Swift 团队实现我们在第五章 *现代范式——闭包和函数式编程* 中看到的许多功能方法的方式。他们不必在数组、字典或其他应可映射的序列上实现 map 方法；相反，他们直接在 `SequenceType` 上实现了它。

这表明类似地，协议扩展可以用于继承，并且它也可以应用于类和结构体，类型也可以从多个不同的协议中继承这种功能，因为没有限制类型可以实现的协议数量。然而，两者之间有两个主要区别。

首先，类型不能从协议继承存储属性，因为扩展不能定义它们。协议可以定义只读属性，但每个实例都必须重新声明它们作为属性：

```swift
protocol Building {
    var squareFootage: Int {get}
}

struct House: Building {
    let squareFootage: Int
}

struct Factory: Building {
    let squareFootage: Int
}
```

其次，方法重写与协议扩展的工作方式不同。在协议中，Swift 不会根据实例的实际类型智能地确定调用哪个版本的方法。在类继承中，Swift 会调用与实例最直接关联的方法版本。记住，当我们调用第三章中我们的`House`子类实例的`clean`方法时，*一点一滴 – 类型、作用域和项目*，它会调用重写的`clean`版本，如下所示：

```swift
class Building {
    // ...

    func clean() {
        print(
            "Scrub \(self.squareFootage) square feet of floors"
        )
    }
}

class House: Building {
    // ...

    override func clean() {
        print("Make \(self.numberOfBedrooms) beds")
        print("Clean \(self.numberOfBathrooms) bathrooms")
    }
}

let building: Building = House(
    squareFootage: 800,
    numberOfBedrooms: 2,
    numberOfBathrooms: 1
)
building.clean()
// Make 2 beds
// Clean 1 bathroom
```

在这里，即使建筑变量被定义为`Building`类型，实际上它是一栋房子；因此 Swift 会调用房子的`clean`版本。与协议扩展的对比在于，它会调用变量声明为的确切类型的版本的方法：

```swift
protocol Building {
    var squareFootage: Int {get}
}

extension Building {
    func clean() {
        print(
            "Scrub \(self.squareFootage) square feet of floors"
        )
    }
}

struct House: Building {
    let squareFootage: Int
    let numberOfBedrooms: Int
    let numberOfBathrooms: Double

    func clean() {
        print("Make \(self.numberOfBedrooms) beds")
        print("Clean \(self.numberOfBathrooms) bathrooms")
    }
}

let house = House(
    squareFootage: 1000,
    numberOfBedrooms: 2,
    numberOfBathrooms: 1.5
)
house.clean()
// Make 2 beds
// Clean 1.5 bathrooms

(house as Building).clean()
// Scrub 1000 square feet of floors
```

当我们调用类型为`House`的`house`变量的`clean`方法时，它会调用房子的版本；然而，当我们将该变量转换为`Building`类型并调用它时，它会调用建筑的版本。

所有这些都表明，在结构体、协议或类继承之间进行选择可能会很困难。我们将在下一章关于内存管理的章节中探讨这个考虑的最后一部分，这样我们就能在前进时做出完全明智的决定。

现在我们已经了解了泛型和协议为我们提供的功能，让我们抓住这个机会来探索在 Swift 中使用协议和泛型的一些更高级的方法。

# 将协议和泛型投入使用

Swift 的一个酷特性是生成器和序列。它们提供了一种轻松迭代值列表的方法。最终，它们归结为两个不同的协议：`GeneratorType`和`SequenceType`。如果你在你的自定义类型中实现了`SequenceType`协议，它允许你使用 for-in 循环遍历你的类型的实例。在本节中，我们将探讨我们如何做到这一点。

## 生成器

这中最关键的部分是`GeneratorType`协议。本质上，生成器是一个你可以反复询问序列中下一个对象的对象，直到没有对象为止。大多数时候你可以简单地使用数组来做这件事，但这并不总是最好的解决方案。例如，你甚至可以创建一个无限生成器。

有一个著名的斐波那契数列的无穷级数，其中序列中的每个数都是前两个数的和。这尤其著名，因为它在自然界中无处不在，从巢中的蜜蜂数量到矩形最令人愉悦的宽高比。让我们创建一个无限生成器，它将生成这个序列。

我们首先创建一个实现`GeneratorType`协议的结构体。该协议由两部分组成。首先，它有一个类型别名，用于表示序列中的元素类型，其次，它有一个名为`next`的 mutating 方法，该方法返回序列中的下一个对象。

实现看起来像这样：

```swift
struct FibonacciGenerator: GeneratorType {
    typealias Element = Int

    var values = (0, 1)

    mutating func next() -> Element? {
        self.values = (
            self.values.1,
            self.values.0 + self.values.1
        )
        return self.values.0
    }
}
```

我们定义了一个名为`values`的属性，它是一个表示序列中前两个值的元组。每次调用`next`时，我们都会更新`values`并返回元组的第一个元素。这意味着序列将没有尽头。

我们可以通过实例化它并在 while 循环中反复调用 next 来单独使用这个生成器：

```swift
var generator = FibonacciGenerator()
while let next = generator.next() {
    if next > 10 {
        break
    }
    print(next)
}
// 1, 1, 2, 3, 5, 8
```

我们需要设置某种条件，以便循环不会无限进行。在这种情况下，一旦数字超过 10，我们就跳出循环。然而，这段代码相当丑陋，所以 Swift 还定义了一个名为`SequenceType`的协议来清理它。

## 序列

`SequenceType`是另一个协议，它被定义为具有一个`GeneratorType`的类型别名和一个名为`generate`的方法，该方法返回该类型的新生成器。我们可以为我们的`FibonacciGenerator`声明一个简单的序列，如下所示：

```swift
struct FibonacciSequence: SequenceType {
    typealias Generator = FibonacciGenerator

    func generate() -> Generator {
        return FibonacciGenerator()
    }
}
```

每个 for-in 循环都操作在`SequenceType`协议上，因此现在我们可以在`FibonacciSequence`上使用 for-in 循环：

```swift
for next in FibonacciSequence() {
    if next > 10 {
        break
    }
    print(next)
}
```

这非常酷；我们可以轻松地以非常可读的方式迭代斐波那契数列。理解前面的代码比理解每次都要计算数列下一个值的复杂 while 循环要容易得多。想象一下我们可以设计的其他类型的序列，比如素数、随机姓名生成器等等。

然而，定义两种不同的类型来创建一个单一的序列并不总是理想的。为了解决这个问题，我们可以使用泛型。Swift 提供了一个名为`AnyGenerator`的泛型类型，以及一个名为`anyGenerator:`的伴随函数。这个函数接受一个闭包，并返回一个使用闭包作为其 next 方法的生成器。这意味着我们不必显式地创建生成器；相反，我们可以在序列中直接使用`anyGenerator:`。

```swift
struct FibonacciSequence2: SequenceType {
    typealias Generator = AnyGenerator<Int>

    func generate() -> Generator {
        var values = (0, 1)
        return anyGenerator({
            values = (values.1, values.0 + values.1)
            return values.0
        })
    }
}
```

在这个版本的`FibonacciSequence`中，每次调用 generate 时，我们都会创建一个新的生成器，它接受一个闭包，执行与原始`FibonacciGenerator`相同的事情。我们在闭包外部声明`values`变量，以便我们可以使用它来在闭包调用之间存储状态。如果你的生成器很简单，不需要复杂的状态，使用`AnyGenerator`泛型是一个很好的选择。

现在，让我们使用这个`FibonacciSequence`来解决计算机擅长的数学问题。

## 50 以下斐波那契数的乘积

如果我们想知道 50 以下斐波那契数中每个数的乘积是什么，我们可以尝试使用计算器并费力地输入所有数字，但使用 Swift 来做会更高效。

让我们从创建一个通用的 `SequenceType` 开始，它将接受另一个序列类型并将其限制为达到最大数字时停止序列。我们需要确保最大值的类型与序列中的类型匹配，并且元素类型是可比较的。为此，我们可以在元素类型上使用 where 子句：

```swift
struct SequenceLimiter<
    S: SequenceType where S.Generator.Element: Comparable
    >: SequenceType
{
    typealias Generator = AnyGenerator<S.Generator.Element>
    let sequence: S
    let max: S.Generator.Element

    init(_ sequence: S, max: S.Generator.Element) {
        self.sequence = sequence
        self.max = max
    }

    func generate() -> Generator {
        var g = self.sequence.generate()
        return anyGenerator({
            if let next = g.next() {
                if next <= self.max {
                    return next
                }
            }
            return nil
        })
    }
}
```

注意，当我们提到元素类型时，我们必须通过生成器类型进行。

当我们的 `SequenceLimiter` 结构被创建时，它会存储原始序列。这是为了让它每次在父序列上调用 `generate` 方法时都能使用 `generate` 方法的返回结果。每次调用 `generate` 都需要重新开始序列。然后它创建一个 `AnyGenerator`，其中包含一个闭包，该闭包在原始序列的本地初始化生成器上调用 `next`。如果原始生成器返回的值大于或等于最大值，我们返回 `nil`，表示序列已结束。

我们甚至可以向 `SequenceType` 添加一个扩展，其中包含一个为我们创建限制器的函数：

```swift
extension SequenceType where Generator.Element: Comparable {
    func limit(max: Generator.Element) -> SequenceLimiter<Self> {
        return SequenceLimiter(self, max: max)
    }
}
```

我们使用 `Self` 作为占位符，表示被调用方法的具体实例类型。

现在，我们可以轻松地将斐波那契序列限制在 50 以下：

```swift
FibonacciSequence().limit(50)
```

我们需要解决的最后一个问题是如何找到序列的乘积。我们可以通过另一个扩展来实现这一点。在这种情况下，我们只将支持包含 `Int` 的序列，这样我们就可以确保元素可以相乘：

```swift
extension SequenceType where Generator.Element == Int {
    var product: Generator.Element {
        return self.reduce(1, combine: *)
    }
}
```

这种方法利用了 reduce 函数，从值一开始，乘以序列中的每个值。现在我们可以轻松地进行最终的计算：

```swift
FibonacciSequence().limit(50).product // 2,227,680
```

几乎瞬间，我们的程序就会返回结果 2,227,680。现在我们真正理解了为什么我们把这些设备称为计算机。

# 摘要

协议和泛型确实很复杂，但我们已经看到它们可以有效地让编译器保护我们免受自己错误的影响。在本章中，我们介绍了协议如何像类型签订的合同。我们还看到，可以通过类型别名使协议更加灵活。泛型允许我们充分利用协议和类型别名，还可以创建强大且灵活的类型，这些类型可以适应它们被使用的上下文。最后，我们探讨了如何以序列和生成器形式使用协议和泛型，以非常干净和易于理解的方式解决一个复杂的数学问题，这可以作为解决其他类型问题的灵感。

到目前为止，我们已经涵盖了 Swift 语言的所有核心功能。我们现在可以更深入地了解程序运行时数据的实际存储方式以及我们如何最好地管理程序使用的资源。
