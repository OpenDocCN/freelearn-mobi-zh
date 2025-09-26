# 第十一章。泛型的工作方式

我第一次接触泛型是在 2004 年，当时它们首次在 Java 编程语言中引入。我仍然记得我拿起我的 *The Java Programming Language* 第四版，它涵盖了 Java 5，并阅读了关于 Java 泛型实现的内容。从那时起，我在许多项目中使用了泛型，不仅是在 Java 中，还在其他语言中也是如此。如果你熟悉其他语言中的泛型，如 Java，Swift 使用的语法将对你来说很熟悉。泛型允许我们编写非常灵活和可重用的代码；然而，就像与下标一样，我们需要确保我们正确地使用它们，并且不要过度使用它们。

在本章中，我们将涵盖以下主题：

+   泛型的介绍

+   创建和使用泛型函数

+   创建和使用泛型类

+   使用协议中的关联类型

# 泛型的介绍

泛型的概念已经存在了一段时间，因此对于来自像 Java 或 C# 这样的语言的开发者来说，这应该不是一个新的概念。Swift 对泛型的实现与这些语言非常相似。对于那些来自像 Objective-C 这样的没有泛型的语言的开发者来说，它们可能一开始会显得有些陌生。

泛型允许我们编写非常灵活和可重用的代码，避免了重复。在像 Swift 这样的类型安全语言中，我们经常需要编写适用于多种类型的函数或类型。例如，我们可能需要编写一个交换两个变量值的函数；然而，我们可能使用这个函数来交换两个字符串类型、两个整型类型和两个双精度浮点类型。没有泛型，我们将需要编写三个单独的函数；但是，有了泛型，我们可以编写一个泛型函数，为多种类型提供交换功能。泛型允许我们告诉一个函数或类型——我知道 Swift 是一个类型安全语言，但我不知道还需要哪种类型。我现在会给你一个占位符，稍后我会告诉你需要强制执行的类型。

在 Swift 中，我们有能力定义泛型函数和泛型类型。让我们首先看看泛型函数。

# 泛型函数

让我们从检查泛型试图解决的问题开始，然后我们将看到泛型是如何解决这个问题的。假设我们想要创建交换两个变量值的函数（如介绍中所述）；然而，对于我们的应用程序，我们需要交换两个整型、两个双精度浮点型和两个字符串。没有泛型，这将需要我们编写三个单独的函数。以下代码显示了这些函数可能看起来像什么：

```swift
func swapInts (inout a: Int, inout b: Int) {
    let tmp = a
    a = b
    b = tmp
}

func swapDoubles(inout a: Double, inout b: Double) {
    let tmp = a
    a = b
    b = tmp
}

func swapStrings(inout a: String, inout b: String) {
    let tmp = a
    a = b
    b = tmp
}
```

使用这三个函数，我们可以交换两个整数的原始值、两个双精度浮点数和两个字符串的值。现在，假设，随着我们进一步开发应用程序，我们发现我们还需要交换两个`UInt32`、两个浮点数或甚至是一些自定义类型的值。我们可能会轻易地得到八个或九个交换函数。最糟糕的部分是，这些函数中每个都包含重复的代码。这些函数之间的唯一区别是变量类型的改变。虽然这个解决方案是可行的，但泛型提供了一个更加优雅和简单的解决方案，它消除了代码的重复。让我们看看我们如何将这些三个先前的函数压缩成一个泛型函数：

```swift
func swap<T>(inout a: T, inout b: T) {
    let tmp = a
    a = b
    b = tmp
}
```

让我们看看我们是如何定义`swap()`函数的。函数本身看起来与普通函数非常相似，只是有一个大写的`T`。在`swap()`函数中使用的大写`T`是一个占位符类型，它告诉 Swift 我们将在稍后定义该类型。当我们定义类型时，我们定义的类型将替换所有占位符。

要定义一个泛型函数，我们在函数名后面包含两个尖括号`<T>`中的占位符类型。然后我们可以使用该占位符类型来代替参数定义、返回类型或函数本身中的任何类型定义。需要牢记的是，一旦占位符被定义为类型，所有其他占位符都会假定该类型。因此，使用该占位符定义的任何变量或常量都必须符合该类型。

大写的`T`并没有什么特殊之处，我们可以用任何有效的标识符来代替`T`。以下定义是完全有效的：

```swift
func swap<G>(inout a: G, inout b: G) {
  //Statements
}

func swap<xyz>(inout a: xyz, inout b: xyz) {
  //Statements
}
```

在大多数文档中，泛型占位符通常使用`T`（表示类型）或`E`（表示元素）来定义。对于标准用途，我们将在本书中使用`T`来定义泛型占位符。在代码中使用`T`来定义泛型占位符也是一个好的实践，这样当我们稍后查看代码时，占位符可以很容易地被识别。

如果我们需要使用多个泛型类型，我们可以通过逗号分隔来创建多个占位符。以下示例展示了如何为单个函数定义多个占位符：

```swift
func testGeneric<T,E>(a:T, b:E) {

}
```

在这个例子中，我们定义了两个泛型占位符，`T`和`E`。在这种情况下，我们可以将`T`占位符设置为一种类型，将`E`占位符设置为另一种类型。

让我们看看如何调用一个泛型函数。以下代码将使用`swapGeneric<T>(inout a: T, inout b: T)`函数交换两个整数：

```swift
var a = 5
var b = 10
swap(&a, b: &b)

print("a:  \(a) b:  \(b)")
```

如果我们运行这段代码，控制台将打印出`a: 10 b: 5`这一行。我们可以看到，调用一个泛型函数不需要做任何特殊的事情。函数会从第一个参数推断类型，然后将所有剩余的占位符设置为该类型。现在，如果我们需要交换两个字符串的值，我们将以这种方式调用相同的函数：

```swift
var c = "My String 1"
var d = "My String 2"
swapGeneric(&c, b: &d)
print("c:  \(c) d:  \(d)")
```

我们可以看到，我们调用函数的方式与我们想要交换两个整数时调用的方式完全相同。我们无法做的一件事是将两种不同的类型传递给`swap()`函数，因为我们只定义了一个泛型占位符。如果我们尝试运行以下代码，我们会收到一个错误：

```swift
var a = 5
var c = "My String 1"
swapGeneric(&a, b: &c)
```

我们将收到的错误是`cannot invoke 'swap' with an argument list of type '(inout Int, b: inout String)`，这告诉我们我们试图在函数只需要类型时使用一个字符串值和一个整数值。函数寻找 Int 值的原因是我们传递给函数的第一个参数是一个 Int 值；因此，函数中的所有泛型类型都变成了 Int 类型。

现在，假设我们有一个以下具有多个泛型类型定义的函数：

```swift
func testGeneric<T,E>(a:T, b:E) {
    print("\(a)  \(b)")
}
```

此函数将接受不同类型的参数；然而，由于它们的类型不同，我们将无法交换值，因为类型不同。泛型还有一些其他限制。例如，我们可能会认为以下泛型函数是有效的；然而，如果我们尝试实现它，我们会收到一个错误：

```swift
func genericEqual<T>(a: T, b: T) -> Bool{
    return a == b
}
```

我们收到的错误是`binary operator '==' cannot be applied to two 'T' operands`。由于在编译代码时参数的类型是未知的，Swift 不知道它是否能够在这些类型上使用等号运算符；因此，抛出了错误。我们可能会认为这是一个将使泛型难以使用的限制；然而，我们有一种方法可以告诉 Swift 我们期望由占位符表示的类型将具有某些功能。这是通过类型约束来完成的。

类型约束指定了一个泛型类型必须从特定的类继承或符合特定的协议。这允许我们在泛型函数中使用由父类或协议定义的方法和属性。让我们通过重写`genericEqual()`函数以使用可比较协议来查看如何使用类型约束：

```swift
func testGenericComparable<T: Comparable>(a: T, b: T) -> Bool{
    return a >= b
}
```

要指定类型约束，我们在泛型占位符之后放置类或协议约束，其中泛型占位符和约束由冒号分隔。这个新函数按我们预期的方式工作，并且它将比较两个参数的值，如果它们相等则返回`true`，如果不相等则返回`false`。

我们可以声明多个约束，就像我们声明多个泛型类型一样。以下示例展示了如何声明具有不同约束的两个泛型类型：

```swift
func testFunction<T: MyClass, E: MyProtocol>(a: T, b: E) {
}
```

在此函数中，由`T`占位符定义的类型必须继承自`MyClass`类，而由`E`占位符定义的类型必须实现`MyProtocol`协议。现在我们已经了解了泛型函数，让我们来看看泛型类型。

# 泛型类型

我们在查看 Swift 数组和字典时已经对泛型类型的工作方式有了总的介绍。泛型类型是一个类、结构体或枚举，它可以与任何类型一起工作，就像 Swift 数组和字典一样工作。回想起来，Swift 数组和字典被编写成可以包含任何类型。问题是我们在数组或字典中不能混合使用不同类型。当我们创建泛型类型的实例时，我们定义了实例将与之一起工作的类型。在定义了该类型之后，我们无法更改该实例的类型。

为了演示如何创建泛型类型，让我们创建一个简单的 `List` 类。这个类将使用 Swift 数组作为列表的后端存储，并允许我们向列表中添加项目或从列表中检索值。

让我们先看看如何定义我们的泛型列表类型：

```swift
class List<T> {
}
```

上述代码定义了泛型列表类型。我们可以看到，我们使用 `<T>` 标签来定义一个泛型占位符，就像我们在定义泛型函数时做的那样。这个 `T` 占位符可以然后在类型的任何地方使用，而不是具体的类型定义。

要创建此类型的实例，我们需要定义列表将包含的项目类型。以下示例展示了如何为各种类型创建泛型列表类型的实例：

```swift
var stringList = List<String>()
var intList = List<Int>()
var customList = List<MyObject>()
```

上述示例创建了 `List` 类的三个实例。`stringList` 实例可以用于 String 类型，`intList` 实例可以用于 Int 类型，而 `customList` 实例可以用于 `MyObject` 类型的实例。

我们不仅限于使用泛型与类一起。我们还可以将结构体和枚举定义为泛型。以下示例展示了如何定义泛型结构体和泛型枚举：

```swift
struct GenericStruct<T> {

}

enum GenericEnum<T> {

}
```

我们 `List` 类的下一步是添加后端存储数组。存储在这个数组中的项目需要与我们初始化类时定义的类型相同；因此，当我们定义数组的类型时，我们将使用 `T` 占位符。以下代码显示了带有名为 `items` 的数组的 `List` 类。`items` 数组将使用 `T` 占位符定义，因此它将包含我们为类定义的相同类型：

```swift
class List<T> {
    var items = [T]()
}
```

此代码定义了我们的泛型列表类型，并使用 `T` 作为类型占位符。然后我们可以在类的任何地方使用 `T` 占位符来定义项目的类型。那个项目将是我们在创建 `List` 类的实例时定义的相同类型。因此，如果我们创建一个类似这样的列表类型实例 `var stringList = List<String>()`，项目数组将是一个字符串实例的数组。如果我们创建一个类似这样的列表类型实例 `var intList = List<Int>()`，项目数组将是一个 Int 实例的数组。

现在，我们需要添加一个`addItems()`方法，该方法将用于向列表中添加一个项目。我们将在方法声明中使用`T`占位符来定义项目参数将与我们在初始化类时声明的类型相同。因此，如果我们创建一个使用字符串类型的列表类型实例，我们就必须使用字符串类型作为`addItems()`方法的参数。然而，如果我们创建一个使用整型类型的列表类型实例，我们就必须使用整型类型作为`addItems()`方法的参数。

下面是`addItems()`函数的代码：

```swift
func addItem(item: T) {
    items.append(item)
}
```

要创建一个独立的泛型函数，我们在函数名称后添加`<T>`声明来声明它是一个泛型函数；然而，当我们在一个泛型类型中使用泛型方法时，我们不需要`<T>`声明。相反，我们只需要使用我们在类声明中定义的类型。如果我们想引入另一个泛型类型，我们可以在方法声明中定义它。

现在，让我们添加一个`getItemAtIndex()`方法，该方法将从后端数组中返回指定索引的项目：

```swift
func getItemAtIndex(index: Int) -> T? {
    if items.count > index {
        return items[index]
    } else {
        return nil
    }
}
```

`getItemAtIndex()`方法接受一个参数，即我们要检索的项目索引。然后我们使用`T`占位符来指定我们的返回类型是一个可能为`T`类型或`nil`的可选类型。如果后端存储数组在指定的索引处包含一个项目，我们将返回该项目；否则，我们不返回任何值。

现在，让我们看看我们的整个泛型列表类：

```swift
class List<T> {
    var items = [T]()

    func addItem(item: T) {
        items.append(item)
    }

    func getItemAtIndex(index: Int) -> T? {
        if items.count > index {
            return items[index]
        } else {
            return nil
        }
    }
}
```

如我们所见，我们最初在类声明中定义了泛型`T`占位符类型。然后我们在类中使用这个占位符类型。在我们的`List`类中，我们在三个地方使用这个占位符。我们将其用作项目数组的类型，用作`addItem()`方法的参数类型，以及用作`getItemAtIndex()`方法中可选返回类型的关联值。

现在，让我们看看如何使用`List`类。当我们使用泛型类型时，我们将在类内部使用尖括号定义要使用的类型，例如`<type>`。以下代码显示了如何使用`List`类来存储字符串类型：

```swift
var list = List<String>()
list.addItem("Hello")
list.addItem("World")
print(list.getItemAtIndex(1))
```

在此代码中，我们首先创建一个名为`list`的列表类型实例，并将其设置为存储`String`类型。然后我们使用`addItem()`方法两次将两个项目存储在列表实例中。最后，我们使用`getItemAtIndex()`方法检索索引号为`1`的项目，它将在控制台显示`Optional(World)`。

我们还可以使用多个占位符类型定义我们的泛型类型，类似于我们在泛型方法中使用多个占位符的方式。要使用多个占位符类型，我们需要用逗号将它们分开。以下示例显示了如何定义多个占位符类型：

```swift
class MyClass<T,E>{

}
```

然后，我们创建一个使用`String`和`Int`类型的`MyClass`类型实例，例如：

```swift
var mc = MyClass<String, Int>()
```

我们还可以使用泛型类型的类型约束。再次强调，为泛型类型使用类型约束与为泛型函数使用类型约束完全相同。以下代码展示了如何使用类型约束来确保泛型类型符合可比较协议：

```swift
class MyClass<T: Comparable>{}
```

到目前为止，在本章中，我们已经看到了如何使用占位符类型与函数和类型一起使用。有时，在协议中声明一个或多个占位符类型可能很有用。这些类型被称为**关联类型**。

# 关联类型

关联类型声明了一个占位符名称，可以在协议中使用该名称代替类型。实际要使用的类型直到协议被采用时才指定。在创建泛型函数和类型时，我们使用了非常相似的语法。然而，为协议定义关联类型却非常不同。我们使用`typealias`关键字来指定关联类型。

当我们定义协议时，让我们看看如何使用关联类型。在这个例子中，我们将定义一个`QueueProtocol`协议，该协议将定义需要由实现它的队列实现的能力：

```swift
protocol QueueProtocol {
    typealias QueueType
    mutating func addItem(item: QueueType)
    mutating func getItem() -> QueueType?
    func count() -> Int
}
```

在这个协议中，我们定义了一个名为`QueueType`的关联类型。然后我们在协议中两次使用这个关联类型——一次作为`addItem()`方法的参数类型，一次当我们定义`getItem()`方法的返回类型为一个可能返回关联类型`QueueType`或`nil`的可选类型。

任何实现`QueueProtocol`协议的类型都必须能够指定用于`QueueType`占位符的类型，并且必须确保在协议中使用`QueueType`占位符的地方只使用该类型的项。

让我们看看如何在非泛型类`IntQueue`中实现`QueueProtocol`。这个类将使用`Int`类型实现`QueueProtocol`协议：

```swift
class IntQueue: QueueProtocol {
  var items = [Int]()

  func addItem(item: Int) {
    items.append(item)
  }

  func getItem() -> Int? {
    if items.count > 0 {
      return items.removeAtIndex(0)
    }
    else {
      return nil
    }
  }

  func count() -> Int {
    return items.count
  }
}
```

在`IntQueue`类中，我们首先定义我们的后端存储机制为一个`Int`类型的数组。然后我们实现协议中定义的每个方法，用 Int 类型替换协议中定义的`QueueType`占位符。在`addItem()`方法中，参数类型被定义为`Int`类型，而在`getItem()`方法中，返回类型被定义为可能返回`Int`类型或无值的可选类型。

我们使用`IntQueue`类的方式就像使用任何其他类一样。以下代码展示了这一点：

```swift
var intQ = IntQueue()
intQ.addItem(2)
intQ.addItem(4)
print(intQ.getItem())
intQ.addItem(6)
```

我们首先创建一个名为`intQ`的`IntQueue`类实例。然后我们调用`addItem()`方法两次，向`intQ`实例添加两个整型的值。然后我们通过调用`getItem()`方法检索`intQ`实例中的第一个项。这一行将打印数字`Optional(2)`到控制台。代码的最后一行添加了另一个整型到`intQ`实例。

在前面的示例中，我们以非泛型的方式实现了`QueueProtocol`协议。这意味着我们用实际类型替换了占位符类型（`QueueType`被替换为`Int`类型）。我们还可以使用泛型类型实现`QueueProtocol`协议。让我们看看如何以泛型类型`GenericQueue`实现`QueueProtocol`协议：

```swift
class GenericQueue<T>: QueueProtocol {
    var items = [T]()

    func addItem(item: T) {
        items.append(item)
    }

    func getItem() -> T? {
        if items.count > 0 {
            return items.removeAtIndex(0)
        } else {
            return nil
        }
    }

    func count() -> Int {
        return items.count
    }
}
```

如我们所见，`GenericQueue`的实现与`IntQueue`的实现非常相似，除了我们定义了要使用的泛型占位符`T`。然后我们可以像使用任何泛型类一样使用`GenericQueue`类。让我们看看如何使用`GenericQueue`类：

```swift
var intQ2 = GenericQueue<Int>()
intQ2.addItem(2)
intQ2.addItem(4)
print(intQ2.getItem())
intQ2.addItem(6)
```

我们首先创建一个`GenericQueue`类的实例，该实例将使用`Int`类型。这个实例被命名为`intQ2`。接下来，我们调用`addItem()`方法两次，向`intQ2`实例添加两个`Int`类型。然后我们使用`getItem()`方法检索添加的第一个`Int`类型，并将其值打印到控制台。这一行将在控制台打印数字`2`。

在使用泛型时，我们应该注意避免在应该使用协议的情况下使用泛型。在我看来，这是在其他语言中泛型最常见误用的一个例子。让我们看看一个例子，以便我们知道要避免什么。

假设我们定义了一个名为`WidgetProtocol`的协议，如下所示：

```swift
protocol WidgetProtocol {
    //Code
}
```

现在，假设我们想要创建一个自定义类型（或函数），该类型将使用`WidgetProtocol`协议的各种实现。我见过一些开发者使用类型约束与泛型一起创建类似这样的自定义类型：

```swift
class MyClass<T: WidgetProtocol> {
    var myProp: T?
    func myFunc(myVar: T) {
        //Code
    }
}
```

虽然这是泛型的一个完全有效的使用，但我们建议避免这种实现方式。如果我们使用`WidgetProtocol`而不使用泛型，代码会更加清晰且易于阅读。例如，我们可以这样编写`MyClass`类型的非泛型版本：

```swift
class MyClass {
    var myProp: WidgetProtocol?
    func myFunc(myVar: WidgetProtocol) {

    }
}
```

`MyClass`类型的第二个非泛型版本更容易阅读和理解；因此，这应该是实现类的首选方式。然而，没有任何阻止我们使用`MyClass`类型的任何实现。

# 摘要

泛型类型可以非常有用，它们也是 Swift 标准集合类型（数组和字典）的基础；然而，正如本章引言中提到的，我们必须小心正确地使用它们。

我们在本章中看到了几个示例，展示了泛型如何让我们的生活变得更简单。本章开头展示的`swapGeneric()`函数是一个泛型函数的良好应用，因为它允许我们只实现一次交换代码，就能交换任何类型我们选择的两值。

通用列表类型也是一个很好的例子，说明了如何创建自定义集合类型，这些类型可以用来存储任何类型的元素。在本章中，我们实现通用列表类型的方式与 Swift 实现泛型数组和大字典的方式相似。
