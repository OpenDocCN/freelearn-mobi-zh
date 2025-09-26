

# 泛型

我第一次接触泛型是在 2004 年，当时它们首次在 Java 编程语言中引入。我仍然记得我拿起我的 *《Java 编程语言，第四版》* 的副本，它涵盖了 Java 5，并阅读了关于 Java 泛型实现的内容。从那时起，我在几个项目中使用了泛型，不仅是在 Java 中，还在其他语言中。如果你熟悉其他语言中的泛型，如 Java，Swift 使用的语法将会非常熟悉。泛型允许我们编写非常灵活和可重用的代码；然而，就像与子脚本一样，我们需要确保我们正确地使用它们，并且不要过度使用它们。

在本章中，我们将涵盖以下主题：

+   什么是泛型？

+   如何创建和使用泛型函数

+   如何创建和使用泛型类型

+   如何使用协议关联类型

# 泛型的介绍

泛型的概念已经存在了一段时间，所以对于来自像 Java 或 C# 这样的语言的开发者来说，这应该不是一个新的概念。Swift 的泛型实现与这些语言非常相似。对于那些来自没有泛型的语言，如 **Objective-C** 的开发者来说，它们可能一开始看起来有些陌生，但一旦开始使用，你就会意识到它们是多么强大。

泛型允许我们编写非常灵活和可重用的代码，从而避免重复。在像 Swift 这样的类型安全语言中，我们经常需要编写适用于多种类型的函数、类和结构体。没有泛型，我们需要为每个我们希望支持的类型编写单独的函数；然而，有了泛型，我们可以编写一个通用的函数来为多种类型提供功能。泛型允许我们告诉一个函数或类型，“*我知道 Swift 是一个类型安全语言，但我还不知道需要哪种类型。我现在给你一个占位符，稍后我会告诉你应该使用哪种类型。*”

在 Swift 中，我们有能力定义泛型函数和泛型类型。让我们首先看看泛型函数。

# 通用函数

让我们先来分析泛型试图解决的问题，然后我们将看到泛型是如何解决这个问题的。假设我们想要创建交换两个变量值的函数，正如本章第一部分所描述的；然而，对于我们的应用程序，我们需要交换两个 `integer` 类型、两个 `Double` 类型以及两个 `String` 类型的值。以下代码展示了这些函数可能的样子：

```swift
func swapInts(a: inout Int,b: inout Int) { 
    let tmp = a
    a = b
    b = tmp
}
func swapDoubles(a: inout Double,b: inout Double) { 
    let tmp = a
    a = b
    b = tmp
}
func swapStrings(a: inout String, b: inout String) { 
    let tmp = a
    a = b
    b = tmp
} 
```

使用这三个函数，我们可以交换两个 `Integer` 类型、两个 `Double` 类型以及两个 `String` 类型的原始值。现在，假设我们在进一步开发应用程序时发现我们还需要交换两个无符号 `Integer` 类型、两个 `Float` 类型以及一些自定义类型的值。我们可能会轻易地得到八个或更多的交换函数。最糟糕的部分是，这些函数之间唯一的区别是参数类型的变化。虽然这个解决方案是可行的，但泛型提供了一个更简单、更优雅的解决方案，消除了所有重复的代码。让我们看看如何将前面三个函数压缩成一个泛型函数：

```swift
func swapGeneric<T>(a: inout T, b: inout T) { 
    let tmp = a
    a = b
    b = tmp
} 
```

让我们看看我们是如何定义 `swapGeneric` 函数的。函数本身看起来与普通函数非常相似，只是多了个大写 `T`。在 `swapGeneric` 函数中使用的大写 `T` 是一个占位符类型，告诉 Swift 我们将在稍后定义该类型。当类型被定义时，它将替换所有占位符。

要定义一个泛型函数，我们在函数名称后面包含两个尖括号 `(<T>)` 中的占位符类型。然后我们可以使用该占位符类型来代替参数定义、返回类型或函数本身中的任何类型定义。需要记住的是，一旦占位符被定义为类型，所有其他占位符都假定该类型。因此，使用该占位符定义的任何变量或常量都必须符合该类型。

大写 `T` 没有什么特别之处；我们可以用任何有效的标识符代替 `T`。我们还可以使用描述性的名称，例如键和值，就像 Swift 语言在字典中使用的那样。以下定义是完全有效的：

```swift
func swapGeneric<G>(a: inout G, b: inout G) {
    //Statements
}
func swapGeneric<xyz>(a: inout xyz, b: inout xyz) {
    //Statements
} 
```

在大多数文档中，通用占位符是用 `T`（表示类型）或 `E`（表示元素）定义的。在本章的目的上，我们将使用大写 `T` 来定义通用占位符。在代码中定义通用占位符时使用大写 `T` 也是一个好的实践，这样在稍后查看代码时可以轻松识别占位符。

如果你不喜欢使用大写 `T` 或大写 `E` 来定义泛型，尽量保持一致性。我建议你在整个代码中避免使用不同的标识符来定义泛型。

如果我们需要使用多个泛型类型，我们可以通过逗号分隔来创建多个占位符。以下示例展示了如何为单个函数定义多个占位符：

```swift
func testGeneric<T,E>(a: T, b: E) {
    //Statements
} 
```

在这个例子中，我们定义了两个通用的占位符，`T` 和 `E`。在这种情况下，我们可以将 `T` 占位符设置为一种类型，而将 `E` 占位符设置为不同的类型。

让我们看看如何调用一个泛型函数。以下代码将使用 `swapGeneric<T>(inout a: T, inout b: T)` 函数交换两个整数：

```swift
var a = 5
var b = 10
swapGeneric(a: &a, b: &b)
print("a:\(a) b:\(b)") 
```

如果我们运行此代码，输出将是`a: 10 b: 5`。我们可以看到，我们不需要做任何特别的事情就可以调用泛型函数。函数从第一个参数推断类型，然后设置所有剩余的占位符为此类型。现在，如果我们需要交换两个字符串的值，我们将以如下方式调用相同的函数：

```swift
var c = "My String 1"
var d = "My String 2"
swapGeneric(a: &c, b: &d)
print("c:\(c) d:\(d)") 
```

我们可以看到，函数的调用方式与我们想要交换两个整数时调用的方式完全相同。我们无法做到的一件事是将两种不同的类型传递给交换函数，因为我们只定义了一个泛型占位符。如果我们尝试运行以下代码，我们将收到一个错误：

```swift
var a = 5
var c = "My String 1"
swapGeneric(a: &a, b: &c) 
```

我们将收到的错误是，无法将类型`String`的值转换为期望的参数类型`Int`，这告诉我们我们试图在期望`Int`值时使用`String`值。函数寻找`Int`值的原因是因为我们传递给函数的第一个参数是一个`Int`值，因此，函数中的所有泛型类型都变为`Int`类型。

现在，假设我们有一个以下这样的函数，它定义了多个泛型类型：

```swift
func testGeneric<T,E>(a: T, b: E) { 
    print("\(a)\(b)")
} 
```

此函数将接受不同类型的参数；然而，由于它们是不同类型的，我们将无法交换它们的值，因为它们是不同的。泛型还有一些其他限制。例如，我们可能会认为以下泛型函数是有效的；然而，如果我们尝试实现它，我们将收到一个错误：

```swift
func genericEqual<T>(a: T, b: T) -> Bool{ 
    return a == b
} 
```

我们收到一个错误，因为二元运算符`==`不能应用于两个`T`操作数。由于在编译代码时参数类型是未知的，Swift 不知道它是否可以在这些类型上使用等号运算符，因此抛出错误。我们可能会认为这是一个将使泛型难以使用的限制。然而，我们有一种方法可以告诉 Swift，我们期望由占位符表示的类型具有某种功能。这是通过类型约束实现的。

类型约束指定泛型类型必须继承自特定类或符合特定协议。这允许我们在泛型函数中使用由父类或协议定义的方法或属性。让我们看看如何通过重写`genericEqual`函数以使用`Comparable`协议来使用类型约束：

```swift
func testGenericComparable<T: Comparable>(a: T, b: T) -> Bool{ 
   a == b
} 
```

要指定类型约束，我们在泛型占位符之后放置类或协议约束，其中泛型占位符和约束由冒号分隔。这个新函数按我们预期的方式工作，它将比较两个参数的值，如果它们相等则返回`true`，如果不相等则返回`false`。

我们可以声明多个约束，就像我们声明多个泛型类型一样。以下示例展示了如何声明具有不同约束的两个泛型类型：

```swift
func testFunction<T: MyClass, E: MyProtocol>(a: T, b: E) {
    //Statements
} 
```

在这个函数中，由 `T` 占位符定义的类型必须继承自 `MyClass` 类，由 `E` 占位符定义的类型必须符合 `MyProtocol` 协议。现在我们已经了解了泛型函数，让我们看看泛型类型。

# 泛型类型

当我们查看 Swift 数组和字典时，我们已经对泛型类型的工作原理有一个一般性的介绍。泛型类型是一个类、结构体或枚举，它可以与任何类型一起工作，就像 Swift 数组和字典一样工作。正如我们回忆的那样，Swift 数组和字典被编写成可以包含任何类型。问题是，我们无法在数组或字典中混合和匹配不同的类型。当我们创建泛型类型的实例时，我们定义了实例将与之一起工作的类型。在定义了那个类型之后，我们无法为那个实例更改类型。

为了演示如何创建泛型类型，让我们创建一个简单的 `List` 类。这个类将使用 Swift 数组作为列表的后端存储，并允许我们向列表中添加项目或从列表中检索值。

让我们先看看如何定义我们的泛型 `List` 类型：

```swift
class List<T> {
} 
```

之前的代码定义了通用的 `List` 类型。我们可以看到，我们使用 `<T>` 标签来定义一个通用的占位符，就像我们在定义通用函数时做的那样。这个 `T` 占位符然后可以在类型的任何地方使用，而不是使用具体的类型定义。

要创建此类型的实例，我们需要定义列表将包含的项目类型。以下示例展示了如何为各种类型创建泛型 `List` 类型的实例：

```swift
var stringList = List<String>() 
var intList = List<Int>()
var customList = List<MyObject>() 
```

之前的示例创建了 `List` 类的三个实例。`stringList` 实例可以与 `String` 类型的实例一起使用，`intList` 实例可以与 `integer` 类型的实例一起使用，而 `customList` 实例可以与 `MyObject` 类型的实例一起使用。

我们不仅限于在类中使用泛型。我们还可以将结构和枚举定义为泛型。以下示例展示了如何定义一个泛型结构和泛型枚举：

```swift
struct GenericStruct<T> {
}
   enum GenericEnum<T> {
} 
```

现在让我们将后端存储数组添加到我们的 `List` 类中。存储在这个数组中的项目需要与我们初始化类时定义的类型相同；因此，我们将使用 `T` 占位符来定义数组的定义。以下代码显示了具有名为 `items` 的数组的 `List` 类。`items` 数组将使用 `T` 占位符定义，因此它将包含我们为类定义的相同类型：

```swift
class List<T> {
   var items = [T]()
} 
```

此代码定义了我们的泛型 `List` 类型，并使用 `T` 作为类型占位符。然后我们可以在类的任何地方使用这个 `T` 占位符来定义项目的类型。那个项目将是我们在创建 `List` 类的实例时定义的相同类型。因此，如果我们创建一个 `List` 类型的实例，例如 `var stringList = List<String>()`，则 `items` 数组将是一个字符串实例的数组。如果我们创建一个 `List` 类型的实例，例如 `var intList = List<Int>()`，则 `items` 数组将是一个整数实例的数组。

现在，我们需要创建 `add()` 方法，该方法将用于向列表中添加项目。我们将在方法声明中使用 `T` 占位符来定义 `item` 参数将与我们在初始化类时声明的相同类型。因此，如果我们创建一个 `List` 类型的实例来使用 `String` 类型，我们就必须使用 `String` 类型的实例作为 `add()` 方法的参数。然而，如果我们创建一个 `List` 类型的实例来使用 `Int` 类型，我们就必须使用 `Int` 类型的实例作为 `add()` 方法的参数。

下面是 `add()` 函数的代码：

```swift
func add(item: T) { 
    items.append(item)
} 
```

要创建一个独立的泛型函数，我们在函数名后添加 `<T>` 声明来声明它是一个泛型函数；然而，当我们在一个泛型类型中使用泛型方法时，我们不需要 `<T>` 声明。相反，我们只需要使用我们在类声明中定义的类型。如果我们想引入另一个泛型类型，我们可以在方法声明中定义它。

现在，让我们添加 `getItemAtIndex()` 方法，该方法将返回后端数组中指定索引处的项目：

```swift
func getItemAtIndex(index: Int) -> T? { 
    if items.count>index {
        return items[index]
    } else {
        return nil
    }
} 
```

`getItemAtIndex()` 方法接受一个参数，即我们要检索的项目索引。然后我们使用 `T` 占位符来指定我们的返回类型是一个可能为 `T` 类型或可能为 nil 的可选类型。如果后端存储数组在指定的索引处包含项目，我们将返回该项目；否则，我们返回 `nil`。

现在，让我们看看我们的整个通用 `List` 类：

```swift
class List<T> {
    var items = [T]() 
    func add(item: T) {
        items.append(item)
    }
    func getItemAtIndex(index: Int) -> T? { 
        guard items.count < index else {
            return items[index]
         }
         return nil
    }
} 
```

如我们所见，我们最初在类声明中定义了泛型 `T` 占位符类型。然后我们在类中使用这个占位符类型。在我们的 `List` 类中，我们在三个地方使用了它。我们将其用作 `items` 数组的类型，用作 `add()` 方法的参数类型，以及用作 `getItemAtIndex()` 方法中的可选返回类型。

现在，让我们看看如何使用 `List` 类。当我们使用泛型类型时，我们将在类内部使用尖括号定义要使用的类型，例如 `<type>`。以下代码显示了如何使用 `List` 类来存储 `String` 类型的实例：

```swift
var list = List<String>()
list.add(item: "Hello")
list.add(item: "World")
print(list.getItemAtIndex(index: 1)) 
```

在前面的代码中，我们首先创建了一个名为`list`的`List`类型实例，并指定它将存储`String`类型的实例。然后我们使用`add()`方法两次将两个项目存储在`list`实例中。最后，我们使用`getItemAtIndex()`方法检索索引号为`1`的项目，它将在控制台显示`Optional(World)`。

我们还可以使用多个占位符类型定义我们的泛型类型，类似于我们在泛型方法中使用多个占位符的方式。要使用多个占位符类型，我们用逗号将它们分开。以下示例展示了如何定义多个占位符类型：

```swift
class MyClass<T,E>{
//Code
} 
```

然后，我们创建了一个使用`String`和`Int`类型实例的`MyClass`类型实例，如下所示：

```swift
var mc = MyClass<String, Int>() 
```

我们还可以使用泛型类型的类型约束。再次强调，为泛型类型使用类型约束与为泛型函数使用类型约束完全相同。以下代码展示了如何使用类型约束来确保泛型类型符合`Comparable`协议：

```swift
class MyClass<T: Comparable>{} 
```

到目前为止，在本章中，我们已经看到了如何使用占位符类型与函数和类型一起使用。现在让我们看看我们如何可以条件性地向泛型类型添加扩展。

## 使用泛型条件性地添加扩展

如果类型符合协议，我们可以条件性地向泛型类型添加扩展。例如，如果我们只想在`T`类型符合数字协议的情况下向我们的泛型`List`类型添加`sum()`方法，我们可以这样做：

```swift
extension List where T: Numeric { 
    func sum () -> T {
       items.reduce (0, +)
    }
} 
```

此扩展将`sum()`方法添加到任何`list`实例中，其中`T`类型符合数字协议。这意味着在先前的例子中创建的用于存储`String`实例的`list`实例将不会接收此方法。

在以下代码中，我们创建了一个包含整数的`List`类型实例，该实例将接收`sum()`方法，并可以像下面这样使用：

```swift
var list2 = List<Int>()
list2.add(item: 2)
list2.add(item: 4)
list2.add(item: 6)
print(list2.sum()) 
```

此代码的输出将是`12`。我们也能够在泛型类型或扩展内条件性地添加函数。

## 条件性地添加函数

条件性地添加扩展，正如我们在上一节中看到的，效果很好；然而，如果我们希望为不同的条件添加不同的功能，我们就必须为每个条件创建单独的扩展。从 Swift 5.3 开始，随着 SE-0267 的引入，我们能够条件性地向泛型类型或扩展添加函数。让我们通过重写上一节中的扩展来查看这一点：

```swift
extension List { 
    func sum () -> T where T: Numeric {
       items.reduce (0, +)
    }
} 
```

通过此代码，我们将`where T: Numeric`子句从扩展声明中移出，放入函数声明中。这将条件性地添加函数，如果类型符合`Numeric`协议。现在我们可以根据不同的条件向扩展添加额外的函数，如下面的代码所示：

```swift
extension List { 
    func sum () -> T where T: Numeric {
        items.reduce (0, +)
    }
    func sorted() -> [T] where T: Comparable {
        items.sorted()
    }
} 
```

在之前的代码中，我们添加了一个名为 `sorted()` 的额外函数，该函数仅适用于符合 `Comparable` 协议的类型实例。这使得我们能够在同一扩展或泛型类型中放置具有不同条件的函数，而不是创建多个扩展。我肯定会推荐像本节中展示的那样条件性地添加函数，而不是像前节中展示的那样条件性地添加扩展。

现在让我们看看条件遵从性。

## 条件遵从性

条件遵从性允许泛型类型仅在类型满足某些条件时才遵从协议。例如，如果我们想让我们的 `List` 类型仅在存储在列表中的类型也符合 `Equatable` 协议时才符合 `Equatable` 协议，我们可以使用以下代码：

```swift
extension List: Equatable where T: Equatable { 
    static func ==(l1:List, l2:List) -> Bool {
        if l1.items.count != l2.items.count { 
            return false
        }
        for (e1, e2) in zip(l1.items, l2.items) { 
            if e1 != e2 {
                return false
            }
        }
        return true
    }
} 
```

此代码将为任何存储在列表中的类型也符合 `Equatable` 协议的 `List` 类型实例添加对 `Equatable` 协议的遵从性。

这里展示了一个我们之前没有讨论过的新函数：`zip()` 函数。这个函数将同时遍历两个序列，在我们的例子中是数组，并创建可以比较的配对（`e1` 和 `e2`）。

比较函数将首先检查每个数组是否包含相同数量的元素，如果不是，则返回 `false`。然后它将遍历每个数组，同时比较数组的元素；如果任何一对不匹配，则返回 `false`。如果之前的测试通过，则返回 `true`，这表示 `list` 实例是相等的，因为列表中的元素相同。

现在让我们看看如何向非泛型类型添加泛型下标。

# 泛型下标

在 Swift 4 之前，如果我们想在下标中使用泛型，我们必须在类或结构体级别定义下标。这迫使我们当感觉应该使用下标时定义泛型方法。从 Swift 4 开始，我们可以创建泛型下标，其中下标的返回类型或其参数可以是泛型的。让我们看看如何创建一个泛型下标。在这个第一个例子中，我们将创建一个接受一个泛型参数的下标：

```swift
subscript<T: Hashable>(item: T) -> Int { 
    return item.hashValue
} 
```

当我们创建一个泛型下标时，我们在 `subscript` 关键字之后定义占位符类型。在之前的例子中，我们定义了 `T` 占位符类型，并使用类型约束来确保类型符合 `Hashable` 协议。这将允许我们传递任何符合 `Hashable` 协议的类型的实例。

正如我们在本节开头提到的，我们还可以使用泛型作为下标的返回类型。我们定义泛型占位符的方式与定义泛型参数的方式完全相同。以下示例说明了这一点：

```swift
subscript<T>(key: String) -> T? { 
    return dictionary[key] as? T
} 
```

在这个例子中，我们在`subscript`关键字之后定义了`T`占位符类型，就像在之前的例子中做的那样。然后我们将此类型用作`subscript`的返回类型。

# 关联类型

关联类型声明了一个可以在协议中使用代替类型的占位符名称。实际要使用的类型直到协议被采用时才指定。在创建泛型函数和类型时，我们使用了非常相似的语法，正如我们在本章中看到的。然而，为协议定义关联类型是非常不同的。我们使用`associatedtype`关键字来指定关联类型。

让我们看看在定义协议时如何使用关联类型。在这个例子中，我们将定义`QueueProtocol`协议，该协议定义了实现它的队列需要实现的能力：

```swift
protocol QueueProtocol { 
    associatedtype QueueType
    mutating func add(item: QueueType) 
    mutating func getItem() -> QueueType? 
    func count() -> Int
} 
```

在这个协议中，我们定义了一个关联类型，命名为`QueueType`。然后我们在协议中两次使用此关联类型：一次作为`add()`方法的参数类型，一次在我们定义`getItem()`方法的返回类型时，将其作为可能返回`QueueType`关联类型或`nil`的可选类型。

任何实现了`QueueProtocol`协议的类型都必须能够指定用于`QueueType`占位符的类型，并且必须确保在协议中使用`QueueType`占位符的地方只使用该类型的项。

让我们看看如何在非泛型类`IntQueue`中实现`QueueProtocol`协议。这个类将使用`Integer`类型来实现`QueueProtocol`协议：

```swift
class IntQueue: QueueProtocol { 
    var items = [Int]()
    func add(item: Int) { 
        items.append(item)
    }
    func getItem() -> Int? {
        return items.count > 0 ? items.remove(at: 0) : nil
    }
    func count() -> Int { 
        return items.count
    }
} 
```

在`IntQueue`类中，我们首先定义我们的后端存储机制为一个整数类型的数组。然后我们实现`QueueProtocol`协议中定义的每个方法，将协议中定义的`QueueType`占位符替换为`Int`类型。在`add()`方法中，`parameter`类型被定义为`Int`类型的实例，而在`getItem()`方法中，`return`类型被定义为可能返回`Int`类型实例或`nil`的可选类型。

我们使用`IntQueue`类的方式就像使用任何其他类一样。以下代码展示了这一点：

```swift
var intQ = IntQueue()
intQ.add(item: 2)
intQ.add(item: 4)
print(intQ.getItem()!)
intQ.add(item: 6) 
```

我们首先创建了一个名为`intQ`的`IntQueue`类实例。然后我们调用`add()`方法两次，将两个整数值添加到`intQ`实例中。然后我们通过调用`getItem()`方法检索`intQ`实例中的第一个项目。这一行将数字`2`打印到控制台。代码的最后一行将另一个整数类型的实例添加到`intQ`实例中。

在前面的例子中，我们以非泛型的方式实现了`QueueProtocol`协议。这意味着我们将占位符类型替换为实际类型。`QueueType`被替换为`Int`类型。我们还可以使用泛型类型实现`QueueProtocol`。让我们看看我们将如何做：

```swift
class GenericQueue<T>: QueueProtocol { 
    var items = [T]()
    func add(item: T) {
        items.append(item)
    }
    func getItem() -> T? {
        return items.count > 0 ? items.remove(at:0) : nil
    }
    func count() -> Int { 
        return items.count
    }
} 
```

如我们所见，`GenericQueue` 的实现与 `IntQueue` 的实现非常相似，除了我们定义了要使用的类型为泛型占位符 `T`。然后我们可以像使用任何泛型类一样使用 `GenericQueue` 类。让我们看看如何使用 `GenericQueue` 类：

```swift
var intQ2 = GenericQueue<Int>()
intQ2.add(item: 2)
intQ2.add(item: 4)
print(intQ2.getItem()!)
intQ2.add(item: 6) 
```

我们首先创建一个 `GenericQueue` 类的实例，该实例将使用 `Int` 类型，并将其命名为 `intQ2`。然后，我们调用 `add()` 方法两次，将两个整型实例添加到 `intQ2` 实例中。接着，我们使用 `getItem()` 方法检索队列中添加的第一个项目，并将值打印到控制台。这一行将在控制台打印数字 `2`。

我们还可以使用关联类型与类型约束。当采用协议时，为关联类型定义的类型必须从类继承或遵守类型约束定义的协议。以下行定义了一个具有类型约束的关联类型：

```swift
associatedtype QueueType: Hashable 
```

在这个例子中，我们指定当实现协议时，为关联类型定义的类型必须遵守 `Hashable` 协议。

# 摘要

泛型类型非常有用，它们也是 Swift 标准集合类型（数组和字典）的基础；然而，正如本章引言中提到的，我们必须小心正确地使用它们。

在本章中，我们看到了几个示例，展示了泛型如何使我们的生活变得更简单。本章开头展示的 `swapGeneric()` 函数是一个很好的泛型函数的例子，因为它允许我们交换任何类型的选择的两个值，而只需实现一次交换代码。

通用 `List` 类型也是一个很好的例子，说明了如何创建自定义集合类型，这些类型可以用来存储任何类型的对象。在本章中，我们实现通用 `List` 类型的方法与 Swift 使用泛型实现数组和字典的方法类似。

在下一章中，我们将探讨 Swift 中的错误处理以及如何使某些功能仅在用户使用的设备运行特定版本的操作系统时才可用。
