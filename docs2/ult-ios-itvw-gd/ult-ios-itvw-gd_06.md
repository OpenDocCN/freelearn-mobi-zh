# 4

# 数据结构与算法

我们在简历上投入了时间和精力，进行了公司研究并建立了我们的开发者品牌。但我们为什么要这样做？为了开始我们想要工作的公司的面试流程，并且*被雇佣*！

因此，现在我们已经迈出了第一步，我们将转向我们的下一个挑战，通过 iOS 技术面试。

通过 iOS 技术面试的第一个主题将涵盖类、结构体、字典和数组等数据结构。虽然我们可以从 Swift 语言的基本原则开始，但这可能被认为是一个更技术性的主题。相比之下，*数据结构涉及更抽象的概念*。因此，我们将把 Swift 语言留到*第五章*。

本章将涵盖在 iOS 技术面试中经常被问到的基础数据结构。为此，我们将在本章中介绍以下内容：

+   学习数据结构的重要性

+   回答类和结构体的问题

+   回答关于 Swift 数组的提问

+   覆盖 Codable 协议

+   准备与字典和集合相关的面试问题

在我们开始之前，让我们花一点时间来理解数据结构的重要性以及为什么我选择从这一主题开始我们的技术讨论。

# 学习数据结构的重要性

数据结构是我们 iOS 开发的基石。实际上，数据结构是许多编程语言的基石，对该领域的深入理解是成功开发和因此通过 iOS 面试的关键。

什么是数据结构？嗯，类、结构体、数组、字典和集合都是数据结构的例子。

但究竟是什么使得数据结构如此重要？以下是一些为什么数据结构是 iOS 面试不可或缺部分的原因。在我们讨论一些面试问题之前，让我们先列出它们。

## 提高效率

当我们讨论资源时，我们总是面临短缺和限制。尽管 iOS 设备在过去几年中变得更为强大，但 iOS 开发也不例外，需要效率。

每种数据结构在时间和空间复杂度方面都有其独特的优势；因此，在我们的代码中会有不同的用途。有时，性能的差异可能非常显著，以至于它可以使我们的应用程序在即使是功能最强大的 iPhone 上也运行得非常慢。

什么是“时间和空间复杂度？”

时间复杂度是指执行算法或解决问题所需的时间，它是输入大小的函数。它通常以算法执行的基本操作数量来衡量。

相反，空间复杂度是指执行算法或解决问题所需的内存量，它是输入大小的函数。它通常以算法用于存储数据和中间结果的内存量来衡量。

在面试中展示出巨大的知识差距会引发一个大红旗，而对于 iOS 开发者来说，即使对于初级开发者，具备基本知识也是一个最低要求。

尽管如此，让我们尝试改变一下书本氛围——不是所有东西都是红旗，我们甚至可以在面试中得分。模块化就是一个例子。

## 使我们的代码模块化

叫我“超级极客”吧，但我认为数据结构的一个很好的用途是让我们的代码看起来像一件艺术品。模块化可能是艺术代码的最佳例子。

数据结构提供了一种组织和封装我们代码的方法，使其更容易阅读和维护。

跟随我的在线内容的开发者已经知道，我是一个**单一职责原则**的大粉丝，这是 SOLID 原则集的一部分。这个原则指出，每个模块、函数、类，甚至变量都应该只有一个且仅有一个职责。

关于 SOLID 原则

SOLID 是面向对象编程中一组原则的缩写，有助于设计更易于维护、可扩展和可重用的代码。SOLID 原则是由 Robert C. Martin 在 2000 年代初的论文《设计原则和设计模式》中提出的。

五个 SOLID 原则如下：

- **单一职责原则**（**SRP**）：一个类应该只有一个改变的理由

- **开闭原则**（**OCP**）：软件实体应该对扩展开放但对修改封闭

- **里氏替换原则**（**LSP**）：子类型应该是其基类型的可替换的

- **接口隔离原则**（**ISP**）：客户端不应该被迫依赖于它们不使用的接口

- **依赖倒置原则**（**DIP**）：高层模块不应该依赖于低层模块；两者都应依赖于抽象

让我们看看以下代码：

```swift
class Employee {  var name: String
  var salary: Double
  // responsibility 1: store employee data
  init(name: String, salary: Double{
    self.name = name
    self.salary = salary
  }
  // responsibility 2: calculate payroll
  func calculatePayroll() -> Double {
  }
}
```

我们有一个`Employee`类，它有两个职责——第一个是*存储*员工的个人数据，第二个是*计算*工资。

我将从基础知识开始讲起——这不是你希望面试官看到的代码片段，因为它在一个类中混合了两种职责。将`calculatePayroll()`函数移到名为`Payroll`（例如）的单独类中会更好。

代码分离之所以至关重要，是因为我们理解，控制我们代码中发生的事情是至关重要的。如果一个类或结构有更多的职责，它可能会产生副作用，导致其他问题。

我们总是需要解释（对面试官和我们自己）我们刚刚编写的类或函数的目标，它在所选的设计模式中的作用，以及为方法和变量做同样的事情。

模块化代码不仅用于逻辑分离——它还在我的下一个要点中扮演着重要的角色，那就是可重用性。

## 代码的重用

我需要解释代码重用的重要性吗？

但以防万一，当编写可以在不同地方使用的代码时，代码重用可以防止错误和不一致的行为。

在数据结构中的代码重用指的是逻辑和数据的双重重用。这可以包括使用类和结构体来重用逻辑和数据结构，例如数组和字典，以可重用的方式存储和访问数据。

而这正是代码重用与先前的模块化要点相关联的地方——如果一个数据结构只有一个职责，它就更容易重用，因为没有副作用会阻止我们在其他地方使用这段代码。

数据结构中代码重用的另一个例子是*类继承*。当我们创建一个子类时，我们可以使用所有超类代码。

通常，面试官喜欢看到可重用代码，因为它使我们的代码更有效且更少出错。数据结构还可以以另一种方式减少我们的代码出错，那就是 API 和接口。

## 使用数据结构进行 API 设计

数据结构的另一个优秀用途是用于**API**和接口。我们应该将数据结构视为表示实体或某些其他复杂数据集合的方式。如果你这样想，创建我们代码中不同组件之间的 API 接口会更容易。

让我们通过编写一个`sendPersonToServer()`函数来看看它在代码中的含义：

```swift
func sendPersonToServer(name: String, age: Int, email:     String, phone: String, address: String) {
}
```

`sendPersonToServer()`函数的目标是将一个人的详细信息发送到服务器，但我们很容易看到这里的主要问题。首先，我们需要提供一个很长的参数列表，这非常不方便。但更重要的是，看起来所有参数都可以封装到一个我们可以称之为的数据结构中——`Person`。

让我们看看一旦我们将它提取到一个`Person`结构体中，函数接口看起来会是什么样子：

```swift
struct Person {    let name: String
    let age: Int
    let email: String
    let phone: String
    let address: String
}
func sendPersonToServer(person: Person) {
}
```

这看起来更优雅，不是吗？

看起来`sendPersonToServer()`函数的接口现在更清晰了，也更一致。每次我们向`Person`添加一个新变量时，我们不再需要更新函数头，因为它现在被封装在结构体定义中。

这也是为什么面试官在讨论潜在策略时，通常会欣赏看到涉及函数和 API 接口重用的解决方案。

我们讨论了数据结构的重要性——我们提到了效率、模块化、可重用代码和接口。我的主要目标是为你提供一个知识基础设施，帮助你面试时构建答案。

现在，让我们回顾一些关于数据结构的面试问题，从*类*和*结构体*开始！

# 回答类和结构体问题

在数据结构、类和结构体的范畴下，类和结构体问题可能是我们在项目中使用的最基本形式。原因是类和结构体不仅包含数据，还提供了应用的主要逻辑以及它们所代表的对象。

当苹果宣布 Swift 时，他们不断强调在许多情况下使用结构体而不是类。当宣布基于结构体而非类的**SwiftUI**时，这一趋势变得更加极端。因此，不言而喻，为什么类和结构体的问题在 iOS 面试中扮演着重要的角色。

让我们转到第一个也是最流行的问题——类与结构体的比较。

## “类和结构体之间的区别是什么？”

*这个问题为什么重要？*

类和结构体有许多共同特征——它们都用于定义复杂的数据类型、**方法**和**属性**。

“函数”还是“方法？”

在 iOS 开发中进行工作面试意味着我们需要专业。作为专业的一部分，术语至关重要，因此区分函数和方法很重要。函数是一个执行特定任务的代码块，可以从程序的任何地方调用。相比之下，“方法”是与类或结构体相关联的函数。

然而，类和结构体也有一些重要的区别，这些区别会影响我们解决问题时的选择。

*答案是什么？*

类和结构体之间有三个主要区别：

+   第一个区别是，*类是引用类型*，而结构体是*值类型*。这意味着当我们将基于类的对象传递给函数或另一个实例时，我们实际上是在操作和修改原始实例，因为我们传递的对象只是一个引用。结构体则不是这样——每次我们将结构体传递给函数时，我们都在处理该结构体的一个副本，而不是原始的那个。

看看下面的代码：

```swift
struct A {    var name: String
}
var a = A(name: "Avi")
let b = a
a.name = "John"
print(b)
      print(a)
```

代码的结果将是`Avi`，然后是`John`。然而，如果我们将`A`声明为类，结果将是`John`和`John`，因为它是对原始变量的引用。

+   另一个区别是我们*可以继承类*，这意味着我们可以从一个类派生出一个类，包括属性和方法。结构体不能从另一个结构体（或类）派生。

+   最后的区别是**可变性**。因为结构体是值类型，如果我们将其标记为**let**，则不能更改其属性。类则不是这样——如果类被标记为**let**，我们仍然可以修改其属性。

以下代码将引发错误：

```swift
struct A {    var name: String
}
let a = A(name: "Avi")
a.name = "John"
```

然而，如果我们将`A`改为类，那么这将有效：

```swift
class A {    var name: String
    init(name: String) {
        self.name = name
    }
}
let a = A(name: "Avi")
a.name = "John"
```

重要提示

在我们继续之前，让我们澄清一下——在诸如数据结构这样的主题中，并不存在“更好”这一说法。总是存在权衡；我们应该在我们的面试中强调这一事实。

现在我们来讨论下一个问题。

## “类和结构体哪个更好？”

*这个问题为什么重要？*

要给面试官留下深刻印象，不仅需要能够区分结构体和类，而且还需要展示对不同情况下何时使用每种数据结构的理解。

正如我们在本章前面讨论的那样，结构体和类有不同的特性，因此它们的使用场景也不同。这是一个比仅仅知道它们之间的区别更实际的问题。

**是什么** **答案**？

它们都是针对不同目的的绝佳数据结构。从结构体开始，看看它是否满足我们的需求是个好主意。如果我们需要额外的功能，我们可以将其更改为类。

如果我们需要做以下事情，我们应该使用类：

+   将其用作**引用**类型

+   从另一个类**继承**（例如，例如从**UIViewController**子类化）

如果我们需要做以下事情，我们应该使用结构体：

+   在**线程**之间传递

+   优化**性能**

重要的一点是，没有上下文就没有“更好”这一说。一切都基于用例和我们的代码需求。

## “为什么结构体（struct）比类（class）更快？”

**为什么** **这个问题重要**？

这个问题本身可能没有意义，但面试官喜欢问它，因为他们想检查候选人是否深入理解结构体和类在设备内存中的存储方式。知道这个问题的答案，甚至在其他情况下解释差异，都可以在我们的面试得分板上加分。

**是什么** **答案**？

结构体比类更快，是因为它们在内存中的存储方式。结构体是值类型；因此，它们存储在**栈**中，栈也存储局部变量和函数参数。

另一方面，类是引用类型，并且它们间接存储在**堆**中。堆用于存储动态分配的对象。

栈比堆更快，因为它的组织更可预测，并且 I/O 操作执行得更快。

重要的是要说明，性能差异并不显著。我们应该首先根据我们的需求选择合适的数据结构。只有当我们遇到性能问题时，才考虑对其进行优化。

类和结构体是 iOS 开发中极其重要的主题。我们每天都会遇到这些数据结构，并且对它们的深入了解对于编写引人入胜和优秀的代码至关重要。

但在编码中，数据结构通常是在其他数据结构之上创建的。数组就是这样。

让我们确保我们对 Swift 中数组的工作方式有一个全面的理解。

# 回答关于 Swift 数组的疑问

与许多其他数据结构不同，数组被认为比较棘手。一方面，数组非常适合存储数据集合，并且在许多广泛使用的情况下非常有价值，例如管理对象和实体的列表。另一方面，我们需要了解它们的优缺点才能有效地使用它们。

面试官喜欢问一些问题，以检查我们对数组内部工作原理的更深入知识。

像这样的问题，“*如何声明一个数组?*”并不常见，因为假设 iOS 开发者知道如何创建数组。

那么，关于数组的有趣面试问题有哪些？

通常，关于数组的问题集中在优缺点、内存管理和数据处理上。

让我们从优点开始。

## “请列出 Swift 数组的优点”

*这个问题为什么重要?*

Swift 数组具有几个优点，使它们非常有用。如果开发者不了解这些优点，数组可能不是正确的数据结构。记住，对于每个优点，也可能存在缺点。

*什么是* *答案?*

数组有几个优点：

+   **通过索引轻松访问元素**：可以通过提供所需元素的索引号快速检索或修改元素

+   **类型安全**：数组只能包含来自预定义类型的元素

+   **内置操作**：数组有许多操作，例如添加、删除、过滤、排序等

记住，像这样的问题往往是更多问题的基石，比如“*数组的用例有哪些?*”或“*列出数组的缺点.*”深入研究材料可以帮助你为任何意外惊喜做好准备。

## “如何从数组中移除重复项？”

*这个问题为什么重要?*

与 `Set` 数据结构不同，Swift 数组可以包含重复项。这个问题测试了我们操纵数据数组的能力，并与面试官讨论解决方案。有几种方法可以解决这个问题，展示其中一些方法可以让面试官感觉到我们能够从不同方向操纵数据。

*什么是* *答案?*

让我们看看可能的解决方案。

### 解决方案 #1

移除重复元素最简单的方法是将数组转换为 `Set`，然后再将其转换回数组。`Set` 是一种不能包含重复元素的数据结构，将数组转换为 `Set` 会移除这些重复元素。让我们看看它是如何实现的：

```swift
let arrayWithDuplicates = [1, 2, 3, 3, 4, 5, 5]let arrayWithNoDuplicates = Array(Set(arrayWithDuplicates))
```

这段简单优雅的代码片段将非常有效！但对于一些面试官来说，这样做可能看起来像“作弊。”

记住，将转换为 `Set` 可能会 *破坏项目的顺序*，所以如果顺序很重要，这可能不是最佳解决方案。

因此，最好明确我们还有其他解决问题的方法。

### 解决方案 #2

另一种解决方案是构建一个新的数组，并且只有当项目不存在时才添加：

```swift
var newArray: [Int] = []for number in array {
    if ! newArray.contains(number) {
        newArray.append(number)
   }
}
```

虽然这个解决方案是可行的，但它被认为是一种 **暴力** 解决方案。

什么是“暴力”解法？

暴力解法是一种依赖于纯粹的计算能力来解决而不是使用更巧妙方法的算法。这通常是一种简单直接的方法，但这也可能非常耗时，并且可能不是解决更大问题的最有效方法。

一种使循环更高效的方法是使用 `Set` 来检查项目是否已存在，而不是使用 `array.contains()`：

```swift
var newArray: [Int] = []var newAddedItems = Set<Int>()
for number in array {
    if ! newAddedItems.contains(number) {
        newArray.append(number)
        newAddedItems.insert(number)
    }
}
```

与数组的 `contains` 方法不同，`Set` 的 `contains` 方法的平均时间复杂度为 `O(1)`，这将提高我们的答案。

### 解决方案 #3

更优雅的方法是使用数组的 **filter** 方法：

```swift
let numbers = [1, 2, 3, 3, 4, 5, 5]let uniqueNumbers = numbers.filter { number in
    numbers.firstIndex(of: number) == numbers.lastIndex
        (of: number)
}
```

通常，我们应该始终优先选择 filter 方法而不是循环元素。filter 方法更易于阅读且更高效，因为 Swift 语言的本地优化（我们不需要在面试中深入了解，尽管我承认这很迷人）。

## “如何使用数组实现队列？”

*为什么这个问题很重要？*

如果我们知道在计算机科学中 **队列** 的含义以及基本的数组操作方法，这个问题就很简单。

这些就是为什么这个问题相对流行的原因。它展示了我们对队列基本概念的理解，以及在使用数组执行此任务时的通常权衡。

*答案是什么？*

要使用数组创建队列，我们应该首先创建一个具有基本方法（如 `isEmpty`、`count`、`enqueue` 和 `dequeue`）的 `Queue` 结构体。

`Queue` 结构体包含一个底层数组，我们可以使用 `append` 和 `removeFirst` 数组方法来执行队列的基本功能。让我们看看一个例子：

```swift
struct Queue<Element> {    private var array: [Element] = []
    var isEmpty: Bool {
        return array.isEmpty
    }
    var count: Int {
        return array.count
    }
    mutating func enqueue(_ element: Element) {
        array.append(element)
    }
    mutating func dequeue() -> Element? {
        return array.isEmpty ? nil : array.removeFirst()
    }
}
```

不要害怕。没有必要记住那个解决方案！

最好的做法是提高我们对队列的理解，并记住 `append()` 和 `removeFirst()` 数组方法。

最好练习一次或两次，以确保我们第一次就能解决这个问题。

## “如何在 Swift 中通过映射现有数组的元素来创建一个新数组？”

*为什么这个问题很重要？*

除了搜索数组之外，iOS 开发者还需要知道如何从一个数据结构到另一个数据结构的操作和转换数据。这种能力不仅适用于数组，也适用于字典和集合。

转换或映射数据是开发中的常见做法，尤其是在 **combine** 和 **reactive programming** 的世界中。

*答案是什么？*

我们可以使用 `map` 方法将数组映射到新数组。`map` 方法接受一个闭包作为参数，并为每个元素返回一个新值。这样，`map` 方法就产生了一个包含新值的数组。

让我们看看一个基本的映射方法，它接受一个整数数组，并返回一个新数组，其值是原数组的两倍：

```swift
let array = [1, 2, 3, 4, 5]let doubledArray = array.map { element in return element * 2
}
```

循环结束后，数组包含 [2, 4, 6, 8, 10] 的值。

使用映射的另一种更优雅的方法是使用 Swift 的一个特性，称为 **隐式闭包参数语法**。使用该特性，我们可以创建更短、更易读的代码：

```swift
let doubledArray = array.map { $0 * 2 }
```

`$0` 代表闭包中的第一个参数，不需要使用 `return` 关键字，这使得语句看起来更加简洁！

我们可以看到数组在 Swift 和 iOS 开发中是基本的，在数据存储、状态、API 接口以及许多需要处理集合的其他情况下扮演着重要角色。优雅且高效地操作数组是面试官喜欢测试的内容！

现在，我们已经完成了简单数据结构的处理，接下来我们将使用 Codable 来序列化它们。

# 覆盖 Codable 协议

**Codable** 协议是 Swift 中的一个重要特性，它允许我们将序列化数据（如字符串）转换为我们可以处理的数据结构。

关于 Codable 协议的第一件事要知道的是，它结合了两个其他协议——“Encodable”和“Decodable”。这两个协议帮助我们双向转换数据。

数据对象（如结构体）需要遵循 Codable 协议，这样我们才能进行双向转换。

这里有一个简单的代码片段：

```swift
struct Person: Codable {    var name: String
    var age: Int
    var address: String
}
let person = Person(name: "John", age: 30, address: "123 Main St.")
let encoder = JSONEncoder()
let data = try encoder.encode(person)
let decoder = JSONDecoder()
let person = try decoder.decode(Person.self, from: data)
```

如前述代码块所示，`Person` 结构体遵循 Codable 协议，因此我们可以将其转换为数据，并在两个方向上进行转换。

关于 Codable 的第二件事是我们需要知道，所有结构体属性都必须遵循它。

```swift
Person has three properties from the String and Int types. String and Int already conform to Codable, so we don’t need to do anything else. However, if we want to add additional custom properties, we need to make sure they conform to Codable as well.
```

以下示例将 `Child` 属性添加到 `Person` 结构体中：

```swift
struct Person: Codable {    var name: String
    var age: Int
    var address: String
    var children: [Child]
}
struct Child: Codable {
    var name: String
    var age: Int
}
```

`Child` 结构体遵循 Codable 协议的事实使得 `Person` 结构体也遵循 Codable 协议。

让我们回顾一下关于 Codable 协议的一些问题！

## “在使用 Codable 协议时，你是如何处理可选属性的？”

*这个问题为什么重要？*

可选属性在处理 Codable 时是必不可少的，因为它在需要与 API 一起工作时非常有用。有时（或者说，经常），返回的数据可能不包括我们在结构体中定义的所有属性。

熟悉 Codable 在常见 API 用例中的工作方式，可以证明你对它的理解。

*答案是什么？*

要处理可选属性，我们只需将它们声明为可选，使用 `?` 参数即可。

让我们以之前示例中的 `Person` 结构体为例：

```swift
struct Person: Codable {    var name: String
    var age: Int?
    var address: String?
}
```

我们可以看到 `address` 和 `age` 属性都是可选的。如果我们没有在 API 响应中收到这些值，它们将保持为 nil。

另一方面，如果一个值没有被标记为可选，并且没有设置默认值，那么如果相应的键在 JSON 响应中不存在，将会抛出异常。

## “你是如何使用 CodingKeys 枚举将 JSON 对象中的键映射到自定义数据类型的属性中的？”

*这个问题为什么重要？*

**CodingKeys** 枚举允许我们自定义编码/解码过程中使用的键。

一个关于 CodingKey 的好答案表明我们完全理解 Codable 协议的工作方式，并在需要时可以处理更复杂的解析。

*答案是什么？*

Codable 将键映射到属性的方式是通过使用它们的名称。如果我们以 `Person` 结构体为例，`name` 属性将被映射到 JSON 结构中的 `name` 键，因为它们具有相同的键名。

然而，我们可以通过定义自定义映射来使用 CodingKey 枚举轻松地自定义它。

我们需要在符合 CodingKey 的结构体下创建`enum`并定义一个新的映射值。

看看以下示例：

```swift
struct Person: Codable {    var name: String
    var age: Int
    var address: String
    enum CodingKeys: String, CodingKey {
        case name = "full_name"
        case age
        case address
    }
}
Person struct has a name property, but that property is now mapped to a full_name key that will appear in the JSON.
```

这里是带有`full_name`键的 JSON：

```swift
{  "full_name": "Avi Tsadok",
  "age": 42,
  "address": "Hamargalit Street
}
```

使用 CodingKeys 可以帮助我们处理我们结构体和需要解析的数据之间的不同命名。然而，当将一种数据类型转换为另一种数据类型时，CodingKeys 并不能提供帮助。在这种情况下，我们有一个解码器。让我们看看一个与这个问题直接相关的问题。

## “如何在 Codable 中将格式化的日期字符串转换为日期对象？”

*为什么这个问题重要？*

当与 API 一起工作时，除了使用`String`或`Double`之外，没有其他方式来表示日期值。

如果我们想要一个包含日期值的结构体，我们需要一种方法将`Double`或`String`值转换为日期对象。

我们需要理解这些类型的用例并不罕见。

有更多示例，我们需要将 JSON 对象中已有的值转换为我们的结构体中的另一种类型。以下是一些这些示例：

+   将 JSON 值转换为枚举。

+   创建嵌套对象

+   处理可选属性中的默认值

最终，解析 API 响应是 iOS 开发者的一项常见且重要的职责，在这个领域的熟练度是必不可少的。

*什么是* *答案？*

我们使用几个特性来将基于字符串的日期转换为日期对象。第一个是 CodingKey，用于映射键和属性，正如我们在上一个问题中学到的。其次，初始化`using init(from decoder: Decoder)`结构体，最后使用`DateFormatter`将字符串转换为日期对象。

让我们看看一个例子。这是`Person`属性列表：

```swift
struct Person: Codable {    var name: String
    var age: Int
    var address: Address
    var birthday: Date
    enum CodingKeys: String, CodingKey {
        case name
        case age
        case address
        case birthday
    }
    enum Address: String, Codable {
        case home
        case work
    }
```

这是`init(from decoder:)`函数：

```swift
init(from decoder: Decoder) throws {    let container = try decoder.container(keyedBy:CodingKeys.self)
    name = try container.decode(String.self,forKey: .name)
    age = try container.decode(Int.self, forKey: .age)
    address = try container.decode(Address.self,forKey: .address)
    // Decode birthday using a custom date formatter
    let dateFormatter = DateFormatter()dateFormatter.dateFormat = 
    "yyyy-MM-dd"
    if let birthdayString = try? container.decode(String.self, forKey: 
    .birthday) {
        birthday = dateFormatter.date(from:birthdayString) ?? Date()
        } else {
        birthday = Date
```

这是一段值得注意的代码！

在面试中，提供关于如何实现本节中提到的先前功能的详细解释是很重要的。

当我们想要根据序列化数据创建结构体时，会调用`init(from:)`。

使用这个方法，我们可以进入接收到的数据的键容器，并将它们映射到结构体属性上。这为我们解析更复杂的数据结构提供了完全的灵活性。

在我们继续之前，有一个关于 Codable 的重要注意事项：从技术上讲，我们“不必”在我们的项目中使用 Codable。还有其他解决方案可以解析和序列化对象和结构体。但在现代 Swift 开发中，Codable 是 API 管理和数据存储的主要参与者；因此，它是面试中必须讨论的主题，我们需要确保我们完全准备好了。

Codable 协议非常适合结构体，但它不是管理复杂数据结构的唯一选项。字典是组织不同类型和结构数据的灵活方法，并且在面试中经常被使用。

# 准备与字典和集合相关的面试问题

字典和集合都是高度有效的数据结构，能够实现快速存储、检索和数据操作。特别是，字典能够以键值格式存储数据，并且它们也是编码和解码复杂数据结构的基础。

作为一名 iOS 开发者，在我们的项目中有一些可以使用字典的场景：

+   **快速查找**：由于字典使用键值来存储数据，因此它们非常适合快速保存和检索数据。这使得字典非常适合保存用户账户、设置列表、缓存数据等。

+   **计数和频率跟踪**：当字典键代表项目的类型，而值是该类型的实例数量时，我们可以用它来跟踪单词、项目等的频率。

+   **编码和解码复杂数据结构**：字典非常灵活，我们可以存储几乎任何数据结构。此外，字典以 JSON 的形式存在，因此它们适合编码和解码 API 请求和响应。

+   **配置数据**：字典的键值性质使它们非常适合存储配置数据。

尝试将这些用例作为面试中关于字典问题的基础。了解字典的主要优势和用法可以轻松帮助我们通过与字典相关的问题。

从存储和快速检索数据的角度来看，集合与字典相似，但集合不使用字典那样的键值结构。相反，它们是唯一值的列表，可以通过`O(1)`的时间复杂度访问，这使得它们在许多用例中非常高效的数据结构。

实际上，将集合与数组进行比较更为合适。如果我们不需要我们的项目列表排序或包含重复项，那么在大多数情况下，集合是比数组更好的选择。

就像字典一样，让我们看看一些集合的使用场景：

+   **从数组中删除重复项**：你还记得使用集合从数组中删除重复项的例子吗？这是集合的经典应用。

+   **快速成员测试**：集合有助于确定某个特定值是否已经被使用。集合中的**contains**函数比数组更高效，在许多情况下都很有帮助。

+   **实体之间的关系**：集合是创建两个实体（如**用户**和**产品**）之间关系的理想数据结构，因为它们不需要排序或重复。这就是为什么集合是核心数据中“多对一”关系的默认选择。

现在，我们来看看一些与集合和字典相关的面试题。

## “你能使用字典来存储配置数据吗？”

*为什么这个问题很重要？*

如前所述，字典是存储数据键值对的宝贵工具，配置数据是这种数据结构的一种常见用例。这个问题涉及到创建一个“配置管理器”类，该类使用字典作为其主要数据结构，并设计一个接口来存储和检索字典中的值。这个任务结合了多种编码技能，包括创建类和实现字典数据结构。

这里有一个很好的技巧！

通常来说，作为一个 iOS 开发者，这是我们日常工作和面试中遇到的一种模式。

让我们再次回顾一下这个模式：

- 决定适合任务的数据结构

- 在类中封装数据结构

- 设计并创建一个简单的接口来处理该数据结构

遵循这些指南是许多答案的关键！

*答案是什么？*

这是一个很好的代码示例，用于在字典中存储配置数据：

```swift
class Configuration {    static let shared = Configuration()
    private var values: [String: Any] = [:]
    func setValue(_ value: Any, forKey key: String) {
        values[key] = value
    }
    func value(forKey key: String) -> Any? {
        return values[key]
    }
}
// Setting configuration values
Configuration.shared.setValue("Dark", forKey: "theme")
Configuration.shared.setValue(true, forKey: "enable_notifications")
```

通过查看代码，我们可以看到我提到的三个原则 – 数据结构（字典）、包装类（`Configuration`）和接口（`setValue()` 和 `value()`）。这将是一个完美的答案。

## “你是如何使用 filter 方法根据条件从字典中选择键值对子集的？”

*为什么这个问题很重要？*

这个问题测试了我们操作字典数据的能力。为了做到这一点，我们需要完成两个任务 – *遍历* 字典值并创建一个 *新字典*，根据某些条件存储一些值。

这个操作在许多不同类型的程序中都很有用，因为它允许你只关注与特定任务相关的键值对。例如，你可能用它来选择只包含在测试中获得“A”级的学生键值对，或者只包含在文本中频繁使用的单词键值对。

*答案是什么？*

解决这个问题有几个方法 – `for` 循环、过滤和遍历字典键。

### 第一种解决方案 – for 循环

for 循环解决方案遍历字典的键和值，并执行简单的 `if-then` 检查以填充一个新的过滤字典，如下所示：

```swift
var wordFrequencies: [String: Int] = ["apple": 4,     "banana": 3, "cherry": 2, "date": 1]
var highFrequencyWords: [String: Int] = [:]
for (word, frequency) in wordFrequencies {
    if frequency >= 3 {
        highFrequencyWords[word] = frequency
    }
}
print(highFrequencyWords)  // prints ["apple": 4, "banana": 3]
```

记住使用 `for` 循环遍历字典的语法：

```swift
for (key, value) in dictionary {  // code to be executed for each key-value pair
}
```

这在其他问题中也会很有用。

### 第二种解决方案 – 使用 filter() 方法

`filter` 方法解决方案被认为比 `for` 循环更 *优雅*，因为它更易读且优化。此外，它将两个操作（遍历和过滤）合并为一个方法。

让我们看看如何使用 filter 方法执行相同的任务：

```swift
let wordFrequencies: [String: Int] = ["apple": 4,    "banana": 3, "cherry": 2, "date": 1]
let highFrequencyWords = wordFrequencies.filter { $0.value >= 3 }
print(highFrequencyWords)  // prints ["apple": 4, "banana": 3]
```

看看这里我们使用了多少更少的代码。这就是专业人士的做法！

### 第三种解决方案 – 遍历键

filter 中的条件不一定基于字典的值；它也可能存在于键中。在这种情况下，我们需要遍历键并应用过滤条件。

以下代码过滤了相同的字典，但这次它创建了一个以 `a` 开头的果名字典子集：

```swift
var wordFrequencies: [String: Int] = ["apple": 4,    "banana": 3, "cherry": 2, "date": 1]
var highFrequencyWords: [String: Int] = [:]
for (word, frequency) in wordFrequencies {
    if frequency >= 3 {
        highFrequencyWords[word] = frequency
    }
}
print(aWords)  // prints ["apple": 4]
```

这个解决方案使我们能够为更多潜在的使用案例和问题执行更强大的字典过滤。

## “常见集合操作（如插入元素或检查成员资格）的时间复杂度是多少？”

*为什么这个问题很重要？*

操作的 **时间复杂度** 是数据结构问题中的热门话题，尤其是在我们讨论集合时。

如果你对这个术语不熟悉，是时候赶上进度了！作为 iOS 开发者，我们做出的一个决定是为正确的任务选择合适的数据结构。

使用集合而不是数组的一个关键优势是其操作效率，例如插入元素和检查成员资格。因此，当处理缓存、避免重复项等情况时，我们将选择集合。

*答案是什么？*

常见集合操作的时间复杂度如下：

+   将一个项目插入到集合中具有平均时间复杂度 O(1) 和最坏情况时间复杂度 O(n)

+   对于检查成员资格也是如此——平均时间复杂度为 O(1) 和最坏情况时间复杂度为 O(n)

集合在常见操作中可以有平均时间复杂度 O(n)，因为在某些情况下集合可能会聚集，时间复杂度取决于集合的大小。

无论如何，集合在插入和检查方面都比数组更高效。

## “在 Swift 的集合集合中是否可以存储任何类型的数据？”

*为什么这个问题很重要？*

这个问题评估了我们对于 Swift 集合数据结构和数据类型哈希过程的理解。

与原始数据类型（如 `Int` 或 `Double`）一起工作很简单。但其他类型，尤其是我们定义的类型，如结构体或类，则不是这样。

因此，理解如何使用 **Hashable** 协议与集合数据结构对于 iOS 开发者至关重要。

*答案是什么？*

只要符合 `Hashable` 协议，就可以在集合集合中存储任何类型的数据。`Hashable` 协议允许自定义类型生成一个唯一值，该值需要存储在集合和字典数据结构中。

让我们看看如何将名为 `Person` 的结构体存储在集合中的示例：

```swift
struct Person: Hashable {    var age: Int
    func hash(into hasher: inout Hasher) {
        hasher.combine(age)
    }
}
let newSet: Set<Person> = [Person(age: 21),
    Person(age: 35), Person(age: 49)]
```

在 Swift 中处理集合和字典时，理解 `Hashable` 协议至关重要，因为它决定了值和键在这些集合中的存储方式。

# 摘要

在本章中，我们讨论了数据结构的重要性以及为什么它们在面试中如此重要。我们涵盖了结构体和类、数组、Codable 协议、集合和字典。

这是一个关键章节，我们现在对数据结构了解得更多了！

然而，那只是一个热身，因为在下一章中，我们将讨论 iOS 面试中的另一个关键主题——Swift 语言本身。
