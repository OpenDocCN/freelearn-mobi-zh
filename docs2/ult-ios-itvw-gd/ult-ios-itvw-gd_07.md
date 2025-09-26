# 5

# Swift 编程语言

正如我们在*第四章*中讨论的那样，理解数据结构是任何开发者，无论他们使用的是哪个平台或语言，都至关重要的复杂技能。数据结构是计算机科学编程和算法的基础，掌握它们对于开发者成功至关重要。现在我们已经对数据结构有了坚实的理解，是时候转向 iOS 开发的重要方面：Swift。

Swift 是 iOS 面试中一个非常热门的话题，它不仅是一种 iOS 开发者的编程语言，也是苹果公司新框架和技术的核心基础。

因此，理解 Swift 的主要特性，如结构体、属性包装器、泛型和更多内容，对于在 iOS 开发中取得成功和通过面试至关重要。Swift 与苹果最新技术之间的紧密关系使得对语言的深入理解对任何 iOS 开发者来说都至关重要。

在本章中，我们将学习关于可选类型、访问级别和闭包的内容。我们还将回顾计算属性和懒加载属性、扩展、泛型、错误处理、协议和内存管理问题。

因此，我们将涵盖以下主题：

+   我们如何掌握所有 Swift 特性？

+   基本 Swift 特性

+   高级 Swift 语言特性

确保我们对主要语言特性有良好的掌握是 iOS 面试过程中取得优异成绩的关键。但我们应该如何确保我们在知识和理解上全面覆盖呢？我们将在本章中看到。

# 我们如何掌握所有 Swift 特性？

首先，阅读本章将帮助我们了解 iOS 技术面试中面试官询问的大多数重要 Swift 特性。

但这还不够。

要成为一名真正的专业人士，我们必须开始像专业人士一样行事。

例如，阅读*官方 Swift 文档*是确保我们覆盖了最新的 Swift 增强功能的绝佳开始。我们将通过回顾访问级别、错误处理和扩展来确保我们覆盖了基础知识。但不要将 Swift 仅仅视为一种编程语言。一些特性是通过深入思考和有趣的方法开发的。

理想情况下，我们不应该仅仅通过记忆技术文档来回答 Swift 面试问题——面试官更希望听到我们的思考、最佳实践和建议。

让我们用扩展来解释这个想法。

对于“*你能告诉我关于 Swift 扩展的内容吗*？”这个问题的一个典型回答可能是，“*Swift 扩展允许我们向现有的类* *或结构体* 添加功能。”

虽然这个回答并不算错误，但它仍然非常技术性。试着深入思考：

+   我们为什么需要扩展？

+   扩展如何帮助我们编写更好的代码？

+   哪些用例使得扩展如此强大？

一个更好的回答可能是以下内容：

*Swift 扩展是一个强大的工具，允许开发者向现有的类、结构体、枚举和协议添加新功能。它们通过将相关功能分组在一起来组织代码，使其更容易阅读和维护。它们还增加了代码的可重用性、可读性和可测试性。*

当然，我们必须确保我们完全理解扩展，以便表达这类答案。

我们下一步将是我们刚刚学到的关于扩展的知识，并将其应用于其他主题，例如可选类型、协议、泛型和 Swift 的其他特性。这正是我们将在本章中做的。

我们是否需要了解所有 Swift 特性？答案是肯定的。我们是否需要非常出色地了解所有特性？强烈推荐，但我们可以在对某些特性没有专业知识的情况下通过一些面试。

正因如此，我将 Swift 特性分为两个层次：基础和高级。

让我们从一些基本语言特性开始，比如可选类型、访问级别和闭包。

# 基础 Swift 特性

对 Swift 的基本概念有一个扎实的理解是至关重要的，因为这些领域的知识不足可能会给 iOS 开发者带来重大问题，更不用说求职面试了。

## 回答可选类型问题

变量类型后的 `?`。

这里有一个例子：

```swift
var name: String?
```

在上一行代码中，`name` 可以包含一个值或 nil。

解包可选类型并提取其值的一个简单方法是使用 `if let` 语句：

```swift
var name: String? = "Avi"if let unwrappedValue = name {
    print("The unwrapped value is: \(unwrappedValue)")
} else {
    print("The optional was nil")
}
if let statement safely “extracts” the value from the unwrappedValue variable and provides an else statement in case it is nil.
```

注意，自 Swift 5.7 以来，可以更优雅地解包，同时保持可选名称不变：

```swift
var name: String? = "Avi"if let name {
    print("The unwrapped value is: \(name)")
} else {
    print("The optional was nil")
}
```

`if let` 简写使解包变得更加简单，因为它不需要我们创建与可选类型同名的新变量/常量。

现在，让我们转向一些面试问题。

### “你能举一个例子说明你会在代码中使用可选类型的情况吗？”

*为什么这个问题很重要？*

这个问题测试了我们对于 Swift 可选类型的实际理解。因为可选类型是一个广泛使用的特性，它涉及到 API 接口设计、函数声明和控制流，面试官需要看到我们是否正确理解了如何使用它。

*答案是什么？*

我们可以在一些日常情况下在我们的代码中使用可选类型。以下是一些例子：

+   函数声明中的可选参数：

    ```swift
    func checkPerson(name: String, age: Int, address: Address?) -> Bool
    ```

+   处理函数可能返回 null 的情况：

    ```swift
    func getParentViewController() -> UIViewController?
    ```

+   使用结构体中的可选类型处理 JSON 响应中的缺失数据：

    ```swift
    struct Person {    var name: String    var age: Int    var address: String?}
    ```

我们应该在理解我们*可能不会收到值*并收到 nil 的任何地方使用可选类型。

### “列出你了解的所有解包可选类型的方法”

*为什么这个问题很重要？*

解包可选类型有几种方法。这并不意味着它们都是彼此的替代品——每种方法都解决不同的用例。了解大多数方法和它们的用例显示了我们在代码中优雅且有效地解包可选类型的能力。

*答案是什么？*

让我们来看看解包可选的一些方法：

+   使用**if let**语法来执行带有未包装值的代码块：

    ```swift
    if let value = optionalValue {    // Do something with the unwrapped value}
    ```

+   使用可选链来避免多个**if** **let**语句：

    ```swift
    if let country = person.address?.country {    print("The person lives in \(country).")} else {    print("The person's address is unknown.")}
    ```

+   使用**guard let**来设置停止条件，如果值是 nil 则退出作用域：

    ```swift
    guard let value = optionalValue else {    return}// Do something with the unwrapped value
    ```

+   使用**!**运算符。如果我们确定可选包含一个值，则强制解包：

    ```swift
    let value = optionalValue!// Do something with the unwrapped value
    ```

+   使用空合并（**??**）来提供默认值：

    ```swift
    let value = optionalValue ?? defaultValue
    ```

没有一种解包值的首选方法。这完全取决于控制流和情况。

### “使用强制解包会在 nil 的情况下崩溃我们的应用。那么我们为什么要使用它？”

*为什么这个问题很重要？*

这是一个在面试中经常被问到的问题。以下是一段代码：

```swift
let value = optionalValue!
```

如果`optionalValue`为 nil，我们会得到一个异常。那么，我们为什么要使用那个方法呢？

这个面试问题实际上并不是关于可选的——它是关于我们管理代码中异常的能力，并在需要时崩溃应用。

*答案是什么？*

最直接的答案可能是，“当我们确定值不是 nil 时。”以下是一个例子：

```swift
var maybeString: String?maybeString = "Hello"
if let unwrappedString = maybeString {
    // If the optional has a value, print it
    print(unwrappedString) // Output: Hello
    print(maybeString!) // force unwrapping
}
```

但这个答案并不完整。为什么我们甚至要接近`maybeString`变量，即使我们只是解包了它？

如前所述，这个问题测试了我们使用可选来管理异常的能力。有些情况下，可选必须包含一个值。否则，程序无法继续。

一个流行的例子是将`IBOutlet`声明为强制解包：

```swift
class ViewController: UIViewController {    // Declare an IBOutlet
    @IBOutlet var label: UILabel!
    override func viewDidLoad() {
        super.viewDidLoad()
        // Force unwrap the label outlet
        label.text = "Hello World!"
    }
}
```

如果`label`为 nil，我们会得到一个异常。一般来说，我们不希望我们的应用崩溃，但在这个情况下，崩溃表明*我们的程序设置有误*——我们可能断开了那个出口，甚至从故事板中移除了它。

另一个很好的例子是在`cellForRow`方法中强制转换`UITableViewCell`。尽管这是一个转换操作，但它与可选相关，因为转换的结果是一个可选值，我们强制它成功。

如果这个转换失败，我们的程序就不再相关，因此我们将使用强制转换：

```swift
override func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {    let cell = tableView.dequeueReusableCell
        (withIdentifier: "customCell", for: indexPath) as!
            CustomTableViewCell
    // configure the cell using the properties and methods
       of the custom class
    return cell
}
```

总结来说，强制解包不是一个常见的技巧，但在某些情况下，它可能很有帮助。

## 解决访问级别问题

起初，访问级别问题看起来像是一个小主题。从技术上讲，它确实是一个小主题。学习和记住不同的访问级别相当直接。

问题始终是，我们是否正确地使用了访问级别？

当一个关键字代表访问级别时，它们会影响代码封装、可见性、项目和组织的可读性。

访问级别还影响我们简单应用组件之间的接口看起来如何。

我们应该了解不同的级别代表什么，以及它们对我们项目结构意味着什么。

### “Swift 中有哪些不同的访问级别，以及它们的用例是什么？”

*为什么这个问题很重要？*

这个问题被认为是一个筛选问题，其目的是确保我们在继续探讨这个主题的更高级问题之前，理解 Swift 中的基本访问级别。

一个筛选问题

筛选问题是一个面试官提出的问题，以确保我们符合该职位的最低资格，并且具备该角色的基本知识。经验丰富的开发者可能会觉得这些问题很奇怪——但我们应该记住，面试官之前并不认识我们。我们应该对这些问题保持谨慎，并确保它们不会成为我们面试的陷阱。筛选问题也被称为“基本”或“核心”问题。

*答案是什么？*

有五种不同的访问级别：

1.  *公开* —— 被标记为公开的实体可以被任何其他模块，包括其他框架访问和子类化。当我们希望允许我们的类或方法被子类化或重写时，使用此级别。

1.  *公开* —— 使用公开，我们允许实体可以从任何其他模块或框架访问，而无需子类化它。有时，由于向后兼容性或安全性，我们不希望其他用户子类化我们的类，使用公开是一个确保这一点的绝佳方式。

1.  *内部* —— 当我们希望我们的实体在同一模块内可访问但不在外部时，应使用内部访问级别。将实体标记为内部不是强制性的——如果我们没有明确定义它，这是默认访问级别。但在库中，将类明确标记为内部是一个最佳实践。

1.  *fileprivate* —— 被标记为 fileprivate 的实体在同一文件内可访问。当我们有一个名为 A 的类，并且我们想要添加另一个仅与类 A 相关的类时，这被使用。如果我们将这两个类都写在同一个文件中，**fileprivate**实体将确保这个约束。

1.  *私有* —— 私有方法和变量仅对*同一类或结构体*（封装声明）可访问。使用私有，我们可以隐藏实体外的代码实现。

### “访问级别如何影响代码组织和可读性？”

*为什么这个问题很重要？*

就像上一个问题一样，作为 iOS 开发者，我们不应该将访问级别视为技术特性。访问级别极大地影响了我们的代码组织和视图。事实上，在某种程度上，访问级别在我们的代码文档中扮演了一定的角色，因为它描述了哪些方法是接口的一部分，哪些方法是实现的一部分。

*答案是什么？*

访问级别通过将其分离为接口和实现来影响代码组织。例如，让我们看看以下`Game`类：

```swift
class Game {    private var gameOver: Bool = false
    public func restart() {
        gameOver = false
        // Other restart logic here
    }
}
```

我们可以看到`Game`类有一个`gameOver`属性，它是被声明为`private`的，还有一个`restart()`方法，它是`public`的。我们理解`gameOver`是被隐藏的，不能直接从类外部修改。唯一改变它的方法是通过使用`restart()`方法，这引出了我的主要观点——可读性。

通过查看`Game`类，我们可以立即看到停止游戏只有一个方法：调用`restart()`。它们可以安全地忽略任何其他私有方法或变量，因为它们只用于实现。如果`gameOver`不是私有的，那么从外部修改它是可能的，而不需要调用`restart()`方法中正在进行的必要步骤。

简而言之——访问级别解释了如何使用类或结构体，并将它们很好地分离为接口和实现。

## 处理关于闭包的问题

闭包取代了 Objective-C 中曾经使用的 Blocks，并且在 Swift 开发中被广泛使用。但我将其归入*基本 Swift 特性*，是因为闭包已经成为许多高级 Swift 特性的基本组成部分。它被用作完成处理程序、高级集合类型函数、SwiftUI 和 Combine。如果不熟悉闭包，可能会影响我们作为 iOS 开发者快速移动和实现高级功能的能力。

### “你是如何使用闭包来处理 iOS 中的回调的？”

*这个问题的意义是什么？*

我选择从这个问题开始，因为回调和异步操作是许多 Swift 应用程序中如何使用闭包的典型例子。与代理不同，闭包可以使异步任务看起来简单，并且始终处于上下文中。

*答案是什么？*

闭包作为参数传递给函数，可以在稍后执行。假设异步操作基于代理或任何其他机制，其响应超出了函数的作用域。在这种情况下，我们可以通过将闭包保存到实例变量中，并在完成任务时调用闭包来处理这个依赖关系。

这里有一个代码示例来解释这一点：

```swift
class SomeClass: SomeDelegate {    var completion: ((Bool) -> Void)?
    Func startAsyncOperation(completion: @escaping ((Bool) -> Void)) {
        self.completion = completion
        // Start async operation
        NetworkManager.shared.performAsyncOperation (delegate: self)
    }
    func operationDidFinish(success: Bool) {
        self.completion?(success)
    }
}
protocol SomeDelegate: AnyObject {
    func operationDidFinish(success: Bool)
}
```

现在，让我们看看如何使用闭包而不使用任何代理：

```swift
let someObject = SomeClass()someObject.startAsyncOperation { success in
    if success {
        print("Async operation succeeded")
    } else {
        print("Async operation failed")
    }
}
```

在前面的代码块中，我演示了如何将代理封装在`SomeClass`中，并仅暴露一个在异步操作结束时运行的闭包。这种模式在调用`startAsyncOperation`时为开发者提供了一个更清晰的接口。

### “你能解释一下 Swift 中的闭包捕获语义如何导致保留循环以及如何避免它们吗？”

*这个问题的意义是什么？*

这个经典的面试问题是初级开发者在与闭包一起工作时常见的陷阱。

在 iOS 开发中，主题之间相互关联，即使我们处理的是闭包而不是内存管理。

闭包很强大，但如果我们使用不当，它们可能会产生内存泄漏并影响我们的应用性能。

这个问题测试了我们对于闭包工作原理的理解。它检查我们在创建和调用闭包时，在应用程序内存中会发生什么，以及如何处理作用域。

*答案是什么？*

闭包通过 **强引用** 从周围的作用域捕获变量和常量。这些常量中可能包含持有闭包本身的对象，这可能导致 **保留周期**。

看看以下代码：

```swift
class SomeClass {    let someProperty = "property value"
    var closure: (() -> Void)?
    func setupClosure() {
        closure = {
            print(self.someProperty)
        }
    }
}
let someObject = SomeClass()
someObject.setupClosure()
```

我们可以看到 `SomeClass` 对 `closure` 有一个强引用，`closure` 打印 `someProperty`，这要求 `closure` 对 `SomeClass`（即 `self`）有一个强引用。

避免保留周期的最简单方法是将 `self` 声明为 `weak` 引用，并通过这种方式解开保留周期：

```swift
class SomeClass {    let someProperty = "property value"
    var closure: (() -> Void)?
    func setupClosure() {
        closure = { [weak self] in
            guard let self else { return }
            print(self.someProperty)
        }
    }
}
```

我们也可以使用 `unowned` 而不是 `weak`，但这是一种危险的方法 – 闭包可能仍然存在，而 `self` 被释放，这可能导致异常。然而，在某些情况下，使用 `unowned` 而不是 `weak` 是安全的，并且这可以从我们类之间的关系中推导出来。一个很好的例子是 `Country` 类和 `CapitalCity` 类。一个国家有一个对其首都城市的引用，而首都城市可以有一个对其国家的 `unowned` 引用。我们理解首都城市的生命周期与其国家的生命周期是一致的，因此，它不能在没有其国家的情况下存在。因此，在这种情况下使用 `unowned` 引用将更加实用，如果发生异常，则表明代码实现中存在错误。

这里有一个代码示例，演示了在 `Country` 类和 `CapitalCity` 类之间使用 `unowned`：

```swift
class Country {    let name: String
    var capital: CapitalCity?
    init(name: String) {
        self.name = name
    }
    deinit {
        print("\(name) is no longer a country.")
    }
}
class CapitalCity {
    let name: String
    unowned let country: Country
    init(name: String, country: Country) {
        self.name = name
        self.country = country
    }
    deinit {
        print("\(name) is no longer a capital city.")
    }
}
```

在 `CapitalCity` 和 `Country` 之间有一个 `unowned` 引用确保我们避免了保留周期，同时仍然保持类之间的引用。

现在我们已经了解了基本的 Swift 特性，我们正在转向更高级的 Swift 特性，以确保我们在这方面有所覆盖。

# 高级 Swift 语言特性

通常，面试官喜欢用 Swift 特性开始，检查不同的语言方面，并试图找到我们可能对 Swift 存在的任何红旗。

在本节中，我们将探讨 Swift 的更多高级特性，从计算变量和懒变量开始。

## 解决计算变量和懒变量问题

计算变量和懒变量都是 Swift 变量的高级特性，提供了提高性能和代码可读性的有效方法。

首先，让我们明确计算变量和懒变量是什么：

+   **计算变量** – 一种根据其他属性计算其值的变量，不将其值存储在内存中，每次访问时都会重新计算

在以下 `Rectangle` 类中，`area` 是一个基于 `width` 和 `height` 值的计算变量：

```swift
class Rectangle {    var width: Double
    var height: Double
    var area: Double {
        return width * height
    }
}
```

+   **懒变量** – 当它首次访问时，其初始值只计算一次的变量

以下代码示例解释了什么是懒变量：

```swift
class ExpensiveObject {    // Some expensive initialization
}
class MyClass {
    lazy var expensiveObject = ExpensiveObject()
}
```

`expensiveObject`变量只有在第一次访问它时才会初始化。我们可以看到变量声明前缀了`lazy`关键字，使其变为懒加载。

许多 iOS 开发者很少使用计算属性和懒变量，大多数情况下，原因是对其缺乏理解以及过早的优化。

现在，让我们深入探讨我们的第一个问题。

### “你会在什么情况下使用计算属性而不是存储属性，反之亦然？”

*这个问题为什么重要？*

这是一个深思熟虑的问题，它有助于测试我们如何将理论应用于实践的理解。在性能和准确性方面，计算属性和存储属性都有其优缺点，因此这个问题超出了仅仅技术考虑的范围。

*答案是什么？*

当需要每次访问属性时都计算值时，会使用计算属性。计算属性通常使用其他属性来计算其值。一些例子包括日期格式化、矩形的面积，或基于其他属性（如名和姓）的全名值。

另一方面，存储属性是从类的外部根据用户输入或其他事件存储和更改的——例如，用户名、配置值等。

计算属性具有更动态的特性。它们不断被计算，因此更准确。缺点是它们在许多情况下效率较低，尤其是在值倾向于变化时。

计算属性和存储属性之间存在紧张关系。存储属性在性能方面非常出色，但我们需要维护它们的数据准确性。计算属性则相反——它们总是准确的，但需要不断计算。

### “你如何使用懒变量来提高加载大量数据的应用的性能？”

*这个问题为什么重要？*

懒变量对性能和内存消耗非常重要。这个问题测试了我们使用懒变量优化应用和 UI 加载的能力。

*答案是什么？*

懒变量可以通过延迟数据的初始化直到实际需要时来提高我们应用的性能。加载一个对象始终被视为一项繁重的任务，因为运行时环境需要初始化对象及其属性。因此，需要初始化和加载大量数据的变量可能会影响被加载对象的加载时间（以及内存消耗）。如果有可能将数据加载推迟到以后，那么可以提高对象的加载时间。

这是一个懒加载代码的例子：

```swift
class MyData {    lazy var largeData: [String] = {
        // load large data from a file or remote API
        return loadLargeData()
    }()
    private func loadLargeData() -> [String] {
        // perform the expensive operation to load the
           large data
        // here we just return an array of string but it
           could be some large data
        return ["large","data","loaded"]
    }
}
let data = MyData()
// the largeData is not loaded until this point
print(data.largeData)
```

从前面的代码中我们可以看到，`largeData`变量可能需要一些时间来初始化，所以我们将其声明为`lazy`。当我们分配`data`时，`largeData`还没有被分配，直到我们使用`print`命令调用它。

## 解决扩展问题

我们讨论的一些特性与代码组织相关。例如，访问级别不仅仅是技术限制；它们也是组织我们的代码和声明接口部分以及封装部分的一部分。

在那个领域，另一个重要的特性是**扩展**。

Swift 中的扩展有几个重要的角色：

+   扩展允许我们向现有的类、结构体和枚举添加新功能，而无需修改它们的源代码

+   扩展可以帮助我们**分组相关功能**，提高我们的代码可读性和组织性

+   扩展用于向类型添加**协议符合性**，使它们的接口与其他符合同一协议的类型对齐

我们可以看到，Swift 语言中有多少扩展是必不可少的，因为它们在我们的日常 iOS 开发中被广泛使用。

尽管扩展功能强大，但它们的使用和理解却毫不费力。这就是为什么我们必须对这个主题做好充分准备，因为任何错误都可能在我们面试官面前拉响红灯。

现在，让我们转到我们的第一个问题。

### “你能否使用扩展向结构体或类添加新属性？”

*这个问题为什么重要？*

这个问题看起来像是一个简单的肯定/否定问题，但现实情况是它隐藏了面试官希望听到的两个更深的理解层次。

首先，他们想听到实际层面——在扩展中什么可行，什么不可行（即完整答案）。

但其次，这是一个额外的要求，他们想听到为什么扩展会以这种方式工作。这将展示我们如何深入理解 Swift 的内存使用。

别担心，我们会在答案中涵盖这两个层次。

*答案是什么？*

简短的答案是“不”，我们不能在扩展中添加存储属性。但值得一提的是，我们可以添加计算属性。原因是我们可以向类型添加新功能，但不能修改其内存布局，这可以暗示我们使用扩展可以向类型添加什么/不能添加什么。

对于这个问题，有几个解决方案——包装原始类型或使用全局变量来存储属性值，但基本思路是一样的。

现在是答案的“额外”部分：一个类型的内存布局是在编译时确定的，并嵌入到二进制文件中。这意味着我们不能使用扩展即时添加新的存储属性，因为它们将改变之前设置的内存布局。将这个事实添加到答案中将在面试中给我们额外的分数！

### “你能否使用扩展向协议添加方法？如果是，怎么做？”

*这个问题为什么重要？*

这是一个棘手的问题。协议不是类型。扩展协议就像为符合协议的类型添加新功能。困惑吗？这就是为什么这个问题棘手……让我们看看答案来澄清一下。

*答案是什么？*

是的，可以扩展一个协议。扩展协议为所有符合该协议的类型添加了新功能，允许我们为协议方法添加默认实现。

让我们看看扩展协议的代码示例：

```swift
protocol MyProtocol {    // existing protocol requirements
}
extension MyProtocol {
    func newMethod() {
        // implementation
    }
}
```

我们可以看到，`MyProtocol` 扩展添加了一个新方法：`newMethod()`。这个新方法可以在所有符合 `MyProtocol` 的类型中使用。让我们继续代码示例来解释这一点：

```swift
struct MyStruct: MyProtocol {    // existing struct properties and methods
}
let myStruct = MyStruct()
myStruct.newMethod()
```

现在应该更清晰了，因为 `myStruct` 可以调用 `newMethod()`，即使它没有在原始协议声明中定义。

## 解决泛型问题

**泛型**是 Swift 特性，允许 iOS 开发者编写可重用的代码，这些代码可以与任何类型的数据一起使用。

对于 iOS 开发者来说，泛型尤为重要，因为它们可以用来编写可重用且类型安全的代码。这意味着开发者可以在应用程序的多个地方使用相同的代码，而无需担心类型转换或其他类型相关的问题。此外，泛型可以帮助防止运行时错误，并通过允许编译器在编译时优化代码来提高性能。总的来说，泛型是强大的工具，可以帮助 iOS 开发者编写更健壮和高效的代码。

现在，让我们看看一个可以与任何类型一起工作的数组反转方法的例子：

```swift
func reverseArray<T>(arr: [T]) -> [T] {    var reversedArr: [T] = []
    for i in stride(from: arr.count - 1, through: 0, by: -1) {
        reversedArr.append(arr[i])
    }
    return reversedArr
}
let numbers = [1, 2, 3, 4, 5]
let reversedNumbers = reverseArray(arr: numbers)
// reversedNumbers is [5, 4, 3, 2, 1]
let words = ["apple", "banana", "cherry"]
let reversedWords = reverseArray(arr: words)
// reversedWords is ["cherry", "banana", "apple"]
```

理解这段代码最重要的地方是这一行：

```swift
func reverseArray<T>(arr: [T]) -> [T]
```

`reverseArray()` 方法接收一个特定类型的数组，并返回一个数组。这可能就是泛型的核心概念——不仅仅是创建可重用的代码，而且还要保持类型安全并避免类型转换问题。

### “你能给出一个使用泛型解决的问题的例子吗？”

*为什么这个问题很重要？*

与之前的问题一样，这个问题通过将一个理论话题与如何在实际生活中使用它的例子相结合来挑战我们。

与其他 Swift 特性相比，如果不通过实际问题和解决方案来了解，泛型的优势就难以理解。

*答案是什么？*

缓存类是使用泛型解决问题的一个绝佳例子。如果我们想要缓存数据，我们需要为每种想要缓存的数据类型创建一个单独的类，或者在某个抽象类中创建不同的方法。

在这种情况下，泛型让我们可以为不同类型重用相同的代码：

```swift
class Cache<T> {    private var cache = [String: T]()
    func set(value: T, for key: String) {
        cache[key] = value
    }
    func get(for key: String) -> T? {
        return cache[key]
    }
}
```

这就是如何使用 `Cache` 类与 `Int`：

```swift
let cache = Cache<[Int]>()cache.set(value: [1, 2, 3, 4, 5], for: "numbers")
let cachedNumbers = cache.get(for: "numbers")
// cachedNumbers is [1, 2, 3, 4, 5]
```

缓存是一个很好的例子，因为它不需要我们转换返回的类型。我们可以初始化一个新的缓存实例，每次都使用不同类型的数据。

### “如何在泛型协议中使用关联类型？”

*为什么这个问题很重要？*

**关联类型**是 iOS 开发者很少使用的一个特性，但我仍然想为它们专门提出一个问题。原因是它可以给你一个更好的泛型在 Swift 中的使用和示例的概览。许多 iOS 开发者很难找到泛型的实际用例，所以从一个不同的角度提供一个例子可能会帮助你准备面试。

**答案是什么**？

关联类型实际上是**协议的泛型**。它们与类和结构体使用泛型的方式相同。

要使用关联类型，我们需要在协议中使用关键字 `associatedtype` 来定义它：

```swift
protocol DataSource {    associatedtype Data
    func fetchData() -> Data
}
```

`DataSource` 协议包含一个 `Data` 关联类型，但它没有指定它将使用哪种类型。我们在协议实现中这样做。

例如，这是一个使用 `Int` 作为 `Data` 的 `DataSource` 的实现：

```swift
struct LocalDataSource: DataSource {    typealias Data = [Int]
    func fetchData() -> [Int] {
    }
}
```

当然，其他结构体或类可以通过在 `Data` 类型别名中定义它来实现该协议，这使得这个协议更加灵活和可重用。

## 解决错误处理问题

错误处理是每种语言和平台中一个基本的话题。它使我们能够对意外的事件或条件做出响应（当我们这样考虑时，它们就变得“可预期”）。

当我们讨论工作面试时，错误处理和 Swift 是很有趣的——首先，当我们从 Objective-C 转移到 Swift 时，这个领域**有了巨大的改进**。然而，在不同的 Swift 版本之间，它也发生了巨大的变化。考虑到你阅读这本书的时候，错误处理可能已经发生了更多的变化，所以看看它是值得的。

此外，**Combine 和 SwiftUI 的日益流行**使得错误处理变得更加普遍。我们可以有信心地说，错误处理是 Combine 数据流的一个基本部分，如果我们对这个领域感到不安全，现在是赶上来的时候了！

### “你如何在 Swift 中使用 try? 和 try! 操作符进行错误处理？”

**为什么这个问题很重要**？

`try?` 和 `try!` 是处理错误更简洁的操作符。

解释这两个操作符之间的区别并在我们的代码流中实现它们是很重要的。

**答案是什么**？

而不是使用 `do-catch` 流，我们可以使用 `try?` 操作符来绕过它，类似于我们解包可选值的方式：

```swift
let result = try? someThrowingFunction()if result != nil {
    // Use the result
} else {
    // Handle the error
}
```

在这个代码示例中，我们使用 `try?` 操作符包裹对 `someThrowingFunction()` 的调用。结果是可选值——如果函数抛出异常，返回的值将是 nil。

然而，`try!` 与强制解包非常相似。如果函数抛出异常，我们的程序将被终止：

```swift
let result = try! someThrowingFunction()
```

注意，我们应该谨慎使用 `try!`，并在函数抛出异常时没有继续程序的意义的情况下使用它。

### “你能解释并给出一个例子说明你如何在 Swift 中编写一个会抛出错误的函数吗？”

**为什么这个问题很重要**？

许多开发者都知道如何执行基本的 `do-catch` 块，主要是因为在很多情况下这是必需的。

向前迈出的自然一步是我们自己执行抛出操作。知道如何编写抛出异常的函数表明了对 Swift 的错误处理机制的深刻理解。

*答案是什么？*

要实现一个抛出异常的函数，我们需要做三件事情：

+   第一件事是在其声明中添加 **throws** 关键字：

    ```swift
    func readFile(at path: String) throws -> String { … }
    ```

+   第二件事是在出现问题时有一个可以 *抛回的错误*：

    ```swift
    enum FileError: Error {    case fileNotFound}
    ```

+   第三件事将是实现并 *在出现异常时抛出错误*（完整的代码如下）：

    ```swift
    enum FileError: Error {    case fileNotFound}func readFile(at path: String) throws -> String {    guard let data = FileManager.default.contents        (atPath: path) else {        throw FileError.fileNotFound    }    return String(data: data, encoding: .utf8) ?? ""}
    ```

记住抛出函数的这三个基本组成部分将帮助我们有效地掌握这个函数。

如果你仍然对错误处理感到不安全，尝试回到你的一个项目中，并在你认为相关的位置添加错误处理。没有什么比实际经验更能处理你不太关心的主题了。

## 解决协议问题

计算机科学中最重要的原则之一是 **关注点的分离**。为了实现这一点，我们想要做的事情之一是减少代码不同部分之间的耦合——解耦对象和类。

协议在这个任务中扮演着重要的角色，使我们的代码更加灵活和可重用。在现代 iOS 开发中，协议是开发的基础部分，我们可以在几乎每个 API 和 SDK 中找到它们被大量使用。

### “你能解释一下在 iOS 开发中使用协议导向编程的用途吗？”

*为什么这个问题很重要？*

尽管这是一个开放性问题，但在面试中很常见。也许因为它是一个开放性问题，面试官喜欢问它，这样他们就可以了解候选人的思考方式。

协议就像烹饪中的调料——技术上，它们很容易使用。问题始于使用多少和何时。

我们应该对此问题有所准备，这也是传播我们关于协议在代码编写中作用的观点的机会。

*答案是什么？*

关于 **协议导向编程**（**Protocol-Oriented Programming**，**POP**）的第一件事是要理解它是一种编程范式。这意味着 POP 是一组用于组织和结构化我们代码的指南和规则。

POP 的主要思想是对象通过协议相互通信。这使得我们的代码更加灵活和可重用，因为不同类型可以实施不同的行为，并且通过遵循相同的接口仍然可以与其他对象一起工作。

POP 与 **面向对象编程**（**Object-Oriented Programming**，**OOP**）一起工作，并不取代它。

### “你如何在你的 iOS 应用中使用协议？”

*为什么这个问题很重要？*

这个问题将理论话题（协议）转移到实际考虑和权衡的世界中。面试官不希望听到二分法答案，而更希望听到一个更深刻的解决方案，涉及我们对 Swift 开发的看法。

*答案是什么？*

我先从底线开始：我们不应该总是使用协议。我们只应该在它有效且不会使我们的项目比现在更复杂时使用它们。我们应该谨慎行事，并根据以下因素：

+   *接口可重用性* – 如果我们想在不同的类型之间重用特定的接口。

+   *抽象* – 协议通过定义一组由不同对象使用的方法和属性，为我们代码提供了另一层抽象。

+   *依赖注入* – 我们可以使用协议将依赖项注入到类中，从而使其更具灵活性和可测试性。

总结来说，当我们需要更多灵活性和减少代码耦合时，协议是一个很好的解决方案。

另一方面，协议可以为我们的代码增加一层复杂性，在类之间添加一个虚拟层。这在编程中是一个预期的权衡 – *复杂性* 与 *耦合性*。

## 解决内存管理问题

对于 iOS 开发者来说，内存管理自 iOS 开发初期以来一直是一个关键问题。

我必须说，随着时间的推移，事情变得更好了——苹果增加了**自动引用计数**（**ARC**），调试工具变得更好，硬件也发生了巨大变化。

话虽如此，在讨论资源管理时，效率仍然是至关重要的。

准备好在你下一次 iOS 技术面试中回答有关该主题的一些问题。

### “iOS 中强引用和弱引用的区别是什么？”

*这个问题为什么重要？*

**强**和**弱**引用是我们在 Swift 中内存所有权概念的核心组成部分。

所有权是 ARC 的关键，这是 iOS 中内存管理机制的基础，如果我们不了解该机制的工作原理，我们就会走上创建**内存泄漏**和**保留循环**的道路。

*答案是什么？*

答案相当简单：强引用（除非另有定义，否则默认为强引用）是一种指示一个对象被一个或多个元素在内存中持有的方式。相比之下，弱引用允许对象在不再需要时被释放。

强引用会将引用计数增加一，而弱引用则完全不改变引用计数。

弱引用的一个很好的例子是**一个** **委托模式**。

让我们看看一个例子：

```swift
class ViewController: UIViewController {    var delegate: ViewControllerDelegate?
}
protocol ViewControllerDelegate: class {
    func didTapButton()
}
class AnotherViewController: UIViewController,
    ViewControllerDelegate {
    weak var viewController: ViewController?
    override func viewDidLoad() {
        super.viewDidLoad()
        viewController = ViewController()
        viewController!.delegate = self
    }
    func didTapButton() {
        // Perform some action
    }
}
```

在这个代码示例中，我们可以看到`ViewController`对其`delegate`对象有强引用，而`delegate`对象对其`ViewController`有弱引用。这种安排的原因是为了避免保留循环，从而增加我们的应用内存使用。

### “如何在 iOS 中处理低内存警告？”

*为什么这个问题重要？*

这个问题旨在评估我们对资源管理的理解。有些情况下，收到低内存警告是可以接受的，但问题是当它发生时我们应该如何处理。控制我们应用中的资源使我们能够适当地管理和应对这些情况。

*答案是什么？*

当我们收到低内存警告时，我们需要做的一件事是释放任何不必要的资源并减少应用的内存占用。

这里有一些减少我们应用内存占用的例子：

+   释放缓存数据

+   使用 autorelease pools

+   使用弱引用

+   释放未使用的资源，例如屏幕外的视图

+   使用 NSCache

那就是深入挖掘我们过去项目的内存，并尝试回忆起在收到低内存警告时可以释放的资源。

# 摘要

在这一章中，我们讨论了 Swift 开发中的许多主题，包括基础和高级内容。我们涵盖了可选类型、访问级别、闭包、计算属性和懒加载属性、扩展、泛型、错误处理、协议和内存管理。

这些有很多主题！但另一方面，我们是有经验的 Swift 开发者，所有这些都需要熟悉。

如我之前所说，我们是优秀的 iOS 开发者，并且对工作非常了解。我们只需要组织我们的知识，以便为我们的面试做好准备。

我们接下来的章节将讨论不仅仅是 Swift 的内容——我们将讨论代码管理。
