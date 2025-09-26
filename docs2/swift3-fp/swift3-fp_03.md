# 第三章 类型与类型转换

本章首先解释类型，非常简短地触及范畴论中的类型概念。然后，它解释值类型和引用类型，并详细比较它们。最后，它讨论了相等性、身份和类型转换。

本章将通过代码示例涵盖以下主题：

+   类型

+   值类型与引用类型

    +   值类型和引用类型常量

    +   混合值类型和引用类型

    +   复制

    +   值类型特性

+   相等性、身份和比较

+   类型检查和转换

你可能听说过函数式编程使用范畴论的概念。这个链接是为什么有些人觉得函数式编程更接近数学的原因。在下一章中，我们将简要介绍范畴论，所以我们现在不会深入探讨这些概念。在此阶段，重要的是要知道，从理论上讲，范畴指的是包含以下内容的集合：

+   一组对象（Swift 中的类型）

+   一组将两个对象联系在一起的形态（Swift 中的函数）

+   范畴的形态组合（Swift 中的函数组合）

我们已经讨论了函数和函数组合，现在我们将探索类型。

有两种不同的方式来对类型进行分类。第一种是 Swift 中命名的类型和复合类型的概念。第二种是基于值与引用的类型分类。

我们在定义时可以为其命名的任何类型都是命名类型。例如，如果我们创建一个名为`OurClass`的类，`OurClass`的任何实例都将具有`OurClass`类型。

函数类型和元组类型是复合类型。复合类型可以包含命名类型和其他复合类型。例如，`(String, (Double, Double))`是一个复合类型，实际上是一个`String`和另一个`(Double, Double)`类型元组的元组。

我们可以在类型注释、标识和别名中使用命名类型和复合类型。

在前面的章节中，我们已经看到我们可以使用 Swift 的推断来推断类型，除非我们想要显式地指定类型。如果我们需要显式地指定类型，我们会注释类型。

此外，我们并没有过多地讨论引用类型与值类型以及类型转换。在本章的后续部分，我们将探讨这些概念。

# 值类型与引用类型

在 Swift 中，有两种类型的类型：值类型和引用类型。

值类型实例保留其数据的一个副本。每种类型都有自己的数据，并且不被其他变量引用。`Structures`、`enums`和`tuples`是值类型；因此，它们不会在其实例之间共享数据。赋值会复制实例的数据到另一个实例，并且没有引用计数。以下示例展示了具有复制的`struct`：

```swift
struct ourStruct {
    var data: Int = 3
}

var valueA = ourStruct()
var valueB = valueA // valueA is copied to valueB
valueA.data = 5 // Changes valueA, not valueB
print("\(valueA.data), \(valueB.data)") // prints "5, 3"

```

如前例所示，改变`valueA.data`不会改变`valueB.data`。

在 Swift 中，数组、字典、字符串和集合都是值类型。

另一方面，引用类型实例共享相同的数据副本。类和闭包是引用类型，所以赋值只添加一个引用，而不复制数据。实际上，引用类型的初始化创建了一个共享实例，该实例将被引用类型的不同实例（如类或闭包）使用。同一类类型的两个变量将引用数据的一个单一实例，因此如果我们修改其中一个变量的数据，它也会影响另一个变量。以下示例展示了具有引用的类：

```swift
class ourClass {
    var data: Int = 3
}
var referenceA = ourClass()
var referenceB = referenceA // referenceA is copied to referenceB
referenceA.data = 5 // changes the instance referred to by
  referenceA and referenceB
print("\(referenceA.data), \(referenceB.data)") // prints "5, 5"

```

如前例所示，更改 `referenceA.data` 也会更改 `referenceB.data`，因为它们引用了相同的共享实例。

值类型和引用类型之间的这种基本差异可能对我们的系统架构产生巨大影响。在函数式编程中，建议优先使用值类型而不是引用类型，因为值类型更容易追踪和推理。由于我们总是得到数据的一个唯一副本，并且数据在实例之间不共享，我们可以推断出程序的其他部分不会更改数据。值类型的这一特性使它们在多线程环境中特别有用，因为在不同的线程中可以更改我们的数据而无需通知我们。这可能会创建非常难以调试和修复的错误。

为了能够在 Swift 中使用类实现此功能，我们可以仅使用不可变存储属性开发不可变类，并避免公开任何可以更改状态的 API。然而，Swift 并没有提供任何语言机制来强制执行类不可变性，就像它强制执行 `struct` 和 `enum` 的不可变性一样。任何 API 用户都可以子类化我们提供的类并将其变为可变的，除非我们将其定义为 **final**。这与 `struct`、`enum` 和元组不同，因为我们基本上不能对它们进行子类化。

## 值类型和引用类型常量

常量的行为取决于它们是值类型还是引用类型。我们可以在常量类中更改变量，但不能在结构体中更改。

让我们检查以下示例：

```swift
class User {
    var name: String
    init(name: String) {
        self.name = name
    }
}

let julie = User(name: "Julie")
let steve = User(name: "Steve")

struct Student {
    var user: User
}

let student = Student(user: julie)
student.user = steve // compiler error - cannot assign to
  property: 'student' is a 'let' constant
```

在这个例子中，我们有一个名为 `User` 的类和两个指向该类实例的常量。此外，我们还有一个具有 `User` 类型变量的 `Student` 结构体。

我们使用 `Student` 结构体创建 `student`。如果我们尝试更改 `student` 中的 `user` 变量，编译器会给出错误，指出 `student` 是一个常量，尽管我们定义 `user` 为一个变量。

因此，如果我们将 `struct` 实例化为一个常量，我们无法更改其中的任何变量。换句话说，`let student = Student(user: julie)` 使得整个 `struct` 都是不可变的。

让我们用类来尝试相同的操作。在下面的代码中，我们更改了名为 `steve` 的常量名称。编译器没有给出错误并接受这个赋值。

```swift
steve.name = "Steve Jr." 
steve.name // prints "Steve Jr." 

```

尽管我们将 `steve` 定义为常量，但我们仍然可以更改 `name` 变量，因为它是一个 `class`。

从前面的例子中，我们已经看到我们可以改变一个常量实例的 `class`（引用类型）变量的值，但不能改变一个常量实例的 `struct`（值类型）变量的值。

由于 `steve` 是引用类型的实例，它引用了 `User` 的实例。当我们更改 `name` 时，我们实际上并没有改变 `steve` 是什么，`steve` 是对 `User` 的引用。我们改变的是我们通过将其定义为变量而使其可变的名称。这并不适用于我们的 `student` 常量，因为它是一个值类型。将其定义为常量也使其变量成为常量。

引用类型的这种特性使得它们难以追踪，并且由于我们将它们定义为常量，这并不会使它们免受更改的影响。为了使它们不可变，我们需要将它们的属性定义为常量。

## 混合值类型和引用类型

在现实世界的问题中，我们可能需要混合引用类型和值类型。例如，我们可能需要在 `struct` 中有对 `class` 的引用，就像我们之前的例子一样，或者我们可能需要在 `class` 中有 `struct` 变量。在这种情况下，我们如何推理赋值和复制呢？

让我们检查以下示例：

```swift
class User {
    var name: String
    init(name: String) {
        self.name = name
    }
}
let julie = User(name: "Julie")

struct Student {
    var user: User
}

let student = Student(user:julie)
student.user.name // prints "Julie"
let anotherStudent = student
julie.name = "Julie Jr."
anotherStudent.user.name // prints "Julie Jr."

```

在这个例子中，我们有一个 `User` 类，一个包含用户变量的 `Student` 结构体。我们定义一个常量 `student`，其值为 `julie`，它是 `class` 类型。如果我们打印 `student.user.name`，结果将是 `julie`。

现在如果我们定义 `anotherStudent` 并通过赋值将其复制到 `student`，更改朱莉的名字也会更改 `anotherStudent` 的名字。

我们预计 `anotherStudent` 将有 `student` 的一个副本，但 `name` 已经更改了。这是因为 `user` 变量是 `User` 类型，它是 `class` 类型，因此是引用类型。

这个例子展示了在值类型中使用引用类型的复杂性。为了避免这些复杂性，建议避免在值类型内部使用引用类型变量。如果我们需要在我们的值类型中使用引用类型，正如我们之前所述，我们应该将它们定义为常量。

## 复制

值类型上的赋值操作将值从一个值类型复制到另一个值类型。在不同的编程语言中，有两种复制类型，浅度复制和深度复制。

浅度复制尽可能少地复制。例如，集合的浅度复制是集合结构的副本，而不是其元素。在浅度复制中，两个集合共享相同的单个元素。

深度复制会复制一切。例如，一个集合的深度复制将导致另一个集合，其中包含原始集合中所有元素的副本。

Swift 进行浅度复制，并且不提供深度复制的机制。让我们通过以下示例来了解浅度复制：

```swift
let julie = User(name: "Julie")
let steve = User(name: "Steve")
let alain = User(name: "Alain")
let users = [alain, julie, steve]

```

在前面的示例中，我们创建了一个名为 `alain` 的新 `User` 对象，并将三个用户添加到了一个名为 `users` 的新数组中。在下面的示例中，我们将 `users` 数组复制到了一个名为 `copyOfUsers` 的新数组中。然后我们按照以下方式更改 `users` 数组中一个用户的名称：

```swift
let copyOfUsers = users
users[0].name = "Jean-Marc"

print(users[0].name) // prints "Jean-Marc"
print(copyOfUsers[0].name) // prints "Jean-Marc"

```

打印 `users` 和 `copyOfUsers` 将显示，在 `users` 数组中将 `Alain` 的 `name` 更改为 `Jean-Marc` 也会将 `copyOfUsers` 中的 `Alain` 的名称更改为 `Jean-Marc`。`users` 和 `copyOfUsers` 都是数组，我们预计赋值表达式会像数组是值类型一样从 `users` 复制值到 `copyOfUsers`，但正如前一个示例所示，在一个数组中更改 `user` 的名称也会更改复制数组中的用户名。这种行为有两个原因。首先，`User` 是 `class` 类型的一种。因此，它是一个引用类型。其次，Swift 进行浅拷贝。

浅拷贝并不提供与示例中看到的不同实例的副本，浅拷贝只是复制了实例相同元素的引用。因此，这个示例再次展示了在 Swift 中使用引用类型作为值类型时的复杂性，因为 Swift 不提供深拷贝来克服这些复杂性。

## 复制引用类型

两个变量可以指向同一个对象，因此更改一个变量也会更改另一个变量。在某些情况下，让许多对象指向相同的数据可能是有用的，但大多数情况下，我们希望修改副本，以便修改一个对象不会影响其他对象。为了实现这一点，我们需要做以下事情：

+   我们的类应该是 `NSObject` 类型

+   我们的类应该遵守 `NSCopying` 协议（这不是强制性的，但可以使我们的 API 用户意图更明确）

+   我们的类应该实现 `copy(with: NSZone)` 方法

+   要复制对象，我们需要在对象上调用 `copy()` 方法

这里是一个完全符合 `NSCopying` 协议的 `Manager` 类的示例：

```swift
class Manager: NSObject, NSCopying {
    var firstName: String
    var lastName: String
    var age: Int

    init(firstName: String, lastName: String, age: Int) {
        self.firstName = firstName
        self.lastName = lastName
        self.age = age
    }

    func copy(with: NSZone? = nil) -> AnyObject {
        let copy = Manager(firstName: firstName, lastName: lastName,
          age: age)
        return copy
    }
}

```

`copyWithZone()` 函数通过使用当前 `Manager` 的信息创建一个新的 `Manager` 对象来实现。为了测试我们的类，我们创建了两个实例，并将一个实例复制到另一个实例上，如下所示：

```swift
let john = Manager(firstName: "John", lastName: "Doe", age: 35)
let jane = john.copy() as! Manager

jane.firstName = "Jane"
jane.lastName = "Doe"
jane.age = 40

print("\(john.firstName) \(john.lastName) is \(john.age)")
print("\(jane.firstName) \(jane.lastName) is \(jane.age)")

```

结果将如下所示：

```swift
"John Doe is 35"
"Jane Doe is 40"

```

## 值类型特性

我们已经研究了值类型和引用类型的观念。我们探讨了值类型与引用类型使用的简单场景。我们理解使用值类型可以使我们的代码更简单，更容易追踪和推理。现在让我们更详细地看看值类型的特性。

### 行为

值类型不具有行为。值类型存储数据并提供使用其数据的方法。值类型只能有一个所有者，并且由于没有引用，它没有析构器。一些值类型的方法可能会使值类型自身发生突变，但控制流严格由实例的单个所有者控制。由于代码仅在直接由单个所有者调用时执行，而不是来自多个来源，因此很容易推理值类型代码的执行流程。

另一方面，引用类型可能会将自己订阅为其他系统的目标。它可能会从其他系统接收通知。这种类型的交互需要引用类型，因为它们可以有多个所有者。在大多数情况下，开发能够自行执行副作用的价值类型是不必要的困难。

### 隔离

典型的值类型对外部系统的行为没有隐式依赖。因此，值类型是隔离的。它只与其所有者交互，与引用类型与多个所有者交互相比，其交互方式更容易理解。

如果我们访问一个可变实例的引用，我们隐式地依赖于所有其他所有者，并且他们可以在不通知我们的情况下随时更改实例。

### 可互换性

由于值类型在分配给新变量时被复制，因此所有这些副本都是完全可互换的。

我们可以安全地存储传递给我们的值，然后稍后将其用作新值。不可能使用除其数据之外的其他任何方式来比较实例。

可互换性还意味着给定值的定义方式并不重要。如果通过 `==` 比较结果为相等，则两个值类型在所有意义上都是相等的。

### 可测试性

对于处理值类型的单元测试，没有必要使用模拟框架。我们可以直接定义与我们的应用程序中的实例不可区分的值。

如果我们使用具有行为的行为类型，我们必须测试我们将要测试的行为类型与系统其余部分之间的交互。这通常意味着大量的模拟或大量的设置代码来建立所需的关系。

相反，值类型是隔离和可互换的，因此我们可以直接定义一个值，调用一个方法，并检查结果。更简单的测试具有更大的覆盖率，从而产生更容易更改和维护的代码。

### 威胁

虽然值类型的结构鼓励可测试性、隔离性和可互换性，但可以定义减少这些优势的值类型。包含在所有者未调用的情况下执行代码的值类型通常难以跟踪和推理，通常应避免。

此外，包含引用类型的值类型并不一定是隔离的。在值类型中使用引用类型通常应避免，因为它们依赖于该引用的所有其他所有者。这类值类型也不容易互换，因为外部引用可能会与系统的其余部分交互并引起一些复杂问题。

## 使用值类型和引用类型

*《Swift 编程语言（Swift 3.0）》* 由 *苹果公司* 出版，其中有一节介绍了如何比较结构体（值类型）和类（引用类型），以及如何选择其中一种类型而舍弃另一种。强烈建议阅读该部分内容，以了解我们为什么选择其中一种类型而舍弃另一种。尽管我们在 *第一章* ，*Swift 中的函数式编程入门* 中简要地提到了这个话题，但我们仍将进一步探讨这个话题，因为在函数式编程中，引用类型和值类型的区别非常重要。

在面向对象编程中，我们将现实世界中的对象建模为类和接口。例如，为了模拟一家提供不同类型披萨的意大利餐厅，我们可能有一个披萨对象及其子类，如玛格丽塔、那不勒斯或罗马披萨。每种披萨都会有不同的配料。不同的餐厅可能制作得略有不同，而且每当我们在不同的书籍或网站上阅读它们的食谱时，我们可能会有不同的理解。这种抽象级别使我们能够引用特定的披萨，而不必关心其他人真正想象的那种披萨。当我们谈论那种披萨时，我们并不是转移它，我们只是引用它。

另一方面，在我们的意大利餐厅中，我们需要向顾客提供账单。每当他们要求账单时，我们将提供有关数量和价格的真实信息。任何人对于数量、美元价格以及实际上价值的感知都是相同的。我们的顾客可以计算发票总额。如果我们的顾客修改账单，它不会修改我们用来提供账单的原始数据。无论他们在账单上写什么，或者洒上酒，账单的价值和总额都不会改变。前面的例子展示了引用类型与值类型在现实世界中的简单应用。在 Swift 编程语言以及网络、移动或桌面应用程序编程中，值类型和引用类型都有它们自己的用途。

值类型使我们能够使架构更清晰、更简单、更易于测试。值类型通常对外部状态有较少或没有依赖，因此在推理它们时考虑的因素更少。

此外，值类型因其可互换性而本质上更具可重用性。

随着我们使用更多的值类型和不可变实体，我们的系统将随着时间的推移变得更加易于测试和维护。

相比之下，引用类型是系统中的行为实体。它们有身份。它们可以表现。它们的行为通常是复杂且难以推理的，但其中的一些细节通常可以用简单的值和涉及这些值的隔离函数来表示。

引用类型维护由值定义的状态，但这些值可以独立于引用类型来考虑。

引用类型会执行副作用，例如 I/O、文件和数据库操作，以及网络操作。

引用类型可以与其他引用类型交互，但它们通常发送值，而不是引用，除非它们真正计划与外部系统创建持久连接。

在不需要创建共享可变状态的情况下，尽可能多地使用值类型（`enums`、`tuples` 或 `structs`）。有些情况下我们必须使用类。例如，当我们使用 **Cocoa** 时，许多 API 期望 `NSObject` 的子类，因此在这些情况下我们必须使用类。每次我们需要使用类时，我们避免使用变量；我们将我们的属性定义为常量，并避免暴露任何可以改变状态的 API。

# 相等性与同一性

如果两个实例具有相同的值，则它们是相等的。相等性用于确定两个值类型的相等性。例如，如果两个 `String` 具有相同的文本值，则它们是相等的。`==` 运算符用于检查相等性。以下示例展示了两个 `Int` 数字（`Int` 是值类型）的相等性检查：

```swift
let firstNumber = 1
let secondNumber = 1

if firstNumber == secondNumber {
    print("Two numbers are equal") // prints "Two numbers are equal\n"
}

```

另一方面，如果两个实例引用了相同的内存实例，则它们是相同的。同一性用于确定两个引用类型是否相同。`===` 运算符用于检查同一性。以下示例展示了我们之前定义的 `User` 类的两个实例的同一性检查：

```swift
let julie = User(name: "Julie")
let steve = User(name: "Steve")

if julie === steve {
    print("Identical")
} else {
    print("Not identical")
}

```

标识检查运算符仅适用于引用类型。

# Equatable 和 Comparable

我们能够比较两种值类型，例如 `String`、`Int` 和 `Double`，但我们不能比较我们自己开发的两种值类型。为了使我们的自定义值类型可比较，我们需要实现 Equatable 和 Comparable 协议。让我们首先检查一个不遵守协议的相等性检查的例子：

```swift
struct Point {
    let x: Double
    let y: Double
}

let firstPoint = Point(x: 3.0, y: 5.5)
let secondPoint = Point(x: 7.0, y: 9.5)

let isEqual = (firstPoint == secondPoint)

```

在这个例子中，编译器会抱怨 **二进制运算符 '==' 不能应用于两个 'Point' 操作数**。让我们通过遵守 `Equatable` 协议来解决这个问题：

```swift
struct Point: Equatable {
    let x: Double
    let y: Double
}

func ==(lhs: Point, rhs:Point) -> Bool {
    return (lhs.x == rhs.x) && (lhs.y == lhs.y)
}

let firstPoint = Point(x: 3.0, y: 5.5)
let secondPoint = Point(x: 7.0, y: 9.5)

let isEqual = (firstPoint == secondPoint)

```

`isEqual` 的值将会是 `false`，因为它们并不相等。为了能够比较两个点，我们需要遵守 `Comparable` 协议。我们的例子如下：

```swift
struct Point: Equatable, Comparable {
    let x: Double
    let y: Double
}

func ==(lhs: Point, rhs:Point) -> Bool {
    return (lhs.x == rhs.x) && (lhs.y == lhs.y)
}

func <(lhs: Point, rhs: Point) -> Bool {
    return (lhs.x < rhs.x) && (lhs.y < rhs.y)
}

let firstPoint = Point(x: 3.0, y: 5.5)
let secondPoint = Point(x: 7.0, y: 9.5)

let isEqual = (firstPoint == secondPoint)
let isLess = (firstPoint < secondPoint)

```

比较的结果将是 `true`。

# 类型检查和类型转换

Swift 提供类型检查和类型转换。我们可以使用 `is` 关键字检查变量的类型。它最常用于 `if` 语句中，如下面的代码所示：

```swift
let aConstant = "String"

if aConstant is String {
    print("aConstant is a String")
} else {
    print("aConstant is not a String")
}

```

由于 `String` 是值类型，编译器可以推断类型，因此 Swift 编译器会发出警告，因为它已经知道 `aConstant` 是 `String` 类型。另一个例子如下，我们检查 `anyString` 是否是 `String` 类型：

```swift
let anyString: Any = "string"

if anyString is String {
    print("anyString is a String")
} else {
    print("anyString is not a String")
}

```

使用 `is` 操作符有助于检查类实例的类型，特别是具有子类的实例。我们可以使用 `is` 操作符来确定一个对象是否是特定类的实例。

同样，我们可以使用 `as` 操作符将对象强制转换为编译器推断的类型之外的类型。`as` 操作符有两种形式：普通的 `as` 操作符和 `as?`。前者在不需要询问的情况下将对象转换为所需的类型。如果对象无法转换为该类型，则会抛出运行时错误。`as?` 操作符询问对象是否可以转换为给定的类型。如果对象可以转换，则返回 *some* 值；否则，返回 `nil`。`as?` 操作符通常用作 `if` 语句的一部分。

显然，在可能的情况下最好使用 `as?`。我们应该只在知道它不会导致运行时错误的情况下使用 `as`。

# 摘要

在本章中，我们探讨了类型的一般概念，并详细探讨了引用类型和值类型。我们涵盖了值类型和引用类型常量、值类型和引用类型的混合以及复制等内容。然后我们学习了值类型的特征、值类型和引用类型之间的关键区别以及我们应该如何决定使用哪一个。我们继续探讨了相等性、身份、类型检查和类型转换等主题。尽管我们探讨了值类型的话题，但我们没有在本章中探讨一个相关的话题——不可变性。第九章，*不可变性的重要性*将涵盖不可变性的重要性。此外，为了深入了解这些概念，建议观看以下视频：WWDC 2015 - Session 414，WWDC 2016 - Session 418，和 WWDC 2016 - Session 419。

在下一章中，我们将探讨枚举和模式匹配主题。我们将熟悉关联值和原始值。我们将介绍代数数据类型，最后，我们将涵盖模式和模式匹配。
