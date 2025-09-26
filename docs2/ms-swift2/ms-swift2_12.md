# 第十二章。使用闭包

现在，大多数主要的编程语言都有类似于闭包提供的功能。其中一些实现非常难以使用（Objective-C blocks），而其他则很容易（Java lambda 和 C# delegates）。我发现闭包提供的功能在开发框架时特别有用。我还在通过网络连接与远程服务通信时广泛使用了它们。虽然 Objective-C 中的 blocks 非常有用（我相当多地使用了它们），但它们用于声明 block 的语法绝对糟糕。幸运的是，当 Apple 开发 Swift 语言时，他们使闭包的语法更容易使用和理解。

在本章中，我们将涵盖以下主题：

+   闭包简介

+   定义闭包

+   使用闭包

+   闭包的一些有用示例

+   如何避免闭包中的强引用循环

# 闭包简介

闭包是自包含的代码块，可以在我们的应用程序中传递和使用。我们可以将 int 类型视为存储整数的类型，而将 string 类型视为存储字符串的类型。在这种情况下，闭包可以被视为一个包含代码块的类型。这意味着我们可以将闭包赋值给变量，将它们作为函数的参数传递，也可以从它们中返回函数。

闭包能够捕获并存储它们定义上下文中任何变量或常量的引用。这被称为覆盖变量或常量，最好的是，在大多数情况下，Swift 会为我们处理内存管理。唯一的例外是我们创建强引用循环，我们将在本章的 *创建闭包中的强引用循环* 部分中探讨如何解决这个问题。

Swift 中的闭包与 Objective-C 中的 blocks 类似；然而，Swift 中的闭包更容易使用和理解。让我们看看定义 Swift 中闭包所使用的语法：

```swift
{
(parameters) -> return-type in
  statements
}
```

如我们所见，创建闭包所使用的语法与我们在 Swift 中创建函数所使用的语法非常相似，实际上，在 Swift 中，全局和嵌套函数都是闭包。闭包和函数之间格式上的最大区别是 `in` 关键字。`in` 关键字用于代替花括号，将闭包的参数和返回类型的定义与闭包体部分隔开。

闭包有许多用途，我们将在本章后面详细讨论它们，但首先我们需要了解闭包的基本知识。让我们先看看一些非常基础的闭包用法，以便我们更好地理解它们是什么，如何定义它们，以及如何使用它们。

# 简单闭包

我们将从创建一个非常简单的闭包开始，这个闭包不接受任何参数，也不返回任何值。它所做的只是将 `Hello World` 打印到控制台。让我们看看以下代码：

```swift
let clos1 = {
  () -> Void in

  print("Hello World")
}
```

在这个例子中，我们创建了一个闭包并将其赋值给常量 `clos1`。由于括号之间没有定义任何参数，这个闭包将不接受任何参数。此外，返回类型被定义为 `Void`；因此，这个闭包不会返回任何值。闭包的主体包含一行代码，将 `Hello World` 打印到控制台。

使用闭包的方法有很多；在这个例子中，我们只想执行它。我们将像这样执行这个闭包：

```swift
clos1()
```

当我们执行闭包时，我们会看到 `Hello World` 被打印到控制台。在这个时候，闭包可能看起来并不那么有用，但随着我们继续阅读本章，我们将看到它们是多么有用和强大。

让我们看看另一个简单的闭包示例。这个闭包将接受一个名为 `name` 的字符串参数，但仍然不会返回任何值。在闭包的主体内部，我们将通过 `name` 参数打印出对传递给闭包的名称的问候语。以下是第二个闭包的代码：

```swift
let clos2 = {
  (name: String) -> Void in

  print("Hello \(name)")
}
```

与这个例子中定义的 `clos2` 和之前的 `clos1` 闭包相比，最大的不同之处在于我们在闭包中定义了一个单独的字符串参数。正如我们所看到的，我们定义闭包参数的方式就像定义函数参数一样。

我们可以用执行 `clos1` 的相同方式执行这个闭包。以下代码显示了如何做到这一点：

```swift
clos2("Jon")
```

当执行这个例子时，它将在控制台打印消息 `Hello Jon`。让我们看看另一种使用 `clos2` 闭包的方法。

我们对闭包的原始定义是，“闭包是自包含的代码块，可以在我们的应用程序代码中传递和使用”。这告诉我们，我们可以将我们的闭包从它们被创建的上下文中传递到代码的其他部分。让我们看看如何将我们的 `clos2` 闭包传递给一个函数。我们将定义一个接受我们的 `clos2` 闭包的函数，如下所示：

```swift
func testClosure(handler:(String)->Void) {
  handler("Dasher")
}
```

我们定义函数的方式就像定义任何其他函数一样；然而，在我们的参数列表中，我们定义了一个名为 `handler` 的参数，并为 `handler` 参数定义的类型是 `(String)->Void`。如果我们仔细观察，我们可以看到 `handler` 参数的 `(String)->Void` 定义与为我们定义的 `clos2` 闭包的参数和返回类型相匹配。这意味着我们可以将 `clos2` 闭包传递给函数。让我们看看如何做到这一点：

```swift
testClosure(clos2)
```

我们像调用任何其他函数一样调用 `testClosure()` 函数，传入的闭包看起来就像任何其他变量。由于 `clos2` 闭包在 `testClosure()` 函数中执行，因此当这段代码执行时，我们将看到消息 `Hello Dasher` 被打印到控制台。

正如我们将在本章稍后看到的那样，将闭包传递给函数的能力使得闭包变得如此令人兴奋和强大。

作为闭包问题的最后一部分，让我们看看如何从闭包中返回一个值。以下示例展示了这一点：

```swift
let clos3 = {
  (name: String) -> String in

  return "Hello \(name)"
}
```

`clos3` 闭包的定义看起来与我们定义 `clos2` 闭包的方式非常相似。区别在于我们将 `Void` 返回类型更改为 `String` 类型。然后，在闭包的主体中，我们不是将消息打印到控制台，而是使用返回语句返回消息。现在我们可以像前两个闭包一样执行 `clos3` 闭包，或者像 `clos2` 闭包一样将其传递给函数。以下示例展示了如何执行 `clos3` 闭包：

```swift
var message = clos3("Buddy")
```

执行这一行代码后，`message` 变量将包含 `Hello Buddy` 字符串。

之前的三个闭包示例展示了闭包的格式以及如何定义一个典型的闭包。熟悉 Objective-C 的人可以注意到 Swift 中闭包的格式要干净得多，也更容易使用。在本章中我们展示的创建闭包的语法相当简短；然而，我们还可以进一步缩短它。在接下来的这一节中，我们将探讨如何做到这一点。

# 闭包的简写语法

在本节中，我们将探讨几种缩短闭包定义的方法。

### 注意

使用闭包的简写语法完全是个人喜好问题。有很多开发者喜欢把他们的代码写得尽可能小和紧凑，并且为此感到非常自豪。然而，有时这可能会让其他开发者难以阅读和理解代码。

我们将要查看的第一个闭包简写语法是最受欢迎的，也是我们在使用数组中的算法时看到的语法，即 第三章，*使用集合和 Cocoa 数据类型*。这种格式主要在我们想要向函数发送一个非常小的闭包（通常是一行）时使用，就像我们使用数组算法那样。在我们查看这种简写语法之前，我们需要编写一个函数，该函数将接受一个闭包作为参数：

```swift
func testFunction(num: Int, handler:()->Void) {
  for var i=0; i < num; i++ {     handler()
  }
}
```

这个函数接受两个参数——第一个参数是一个名为 `num` 的整数，第二个参数是一个名为 `handler` 的闭包，该闭包没有参数且不返回任何值。在函数内部，我们创建一个 `for` 循环，该循环将使用 `num` 整数来定义循环的次数。在 `for` 循环中，我们调用传递给函数的 `handler` 闭包。

我们可以创建一个闭包，并将其像这样传递给 `testFunction()`：

```swift
let clos = {
    () -> Void in
    print("Hello from standard syntax")
}
testFunction(5,handler: clos)
```

这段代码非常容易阅读和理解；然而，它需要五行代码。现在，让我们看看如何通过在函数调用内编写闭包来缩短这段代码：

```swift
testFunction(5,handler: {print("Hello from Shorthand closure")})
```

在这个示例中，我们使用与数组算法相同的语法，在函数调用内联创建闭包。闭包放置在两个花括号（`{}`）之间，这意味着创建我们的闭包的代码是 `{print("Hello from Shorthand closure")}`。当这段代码执行时，它将在屏幕上打印出消息，`Hello from Shorthand closure`，五次。

在 第三章，*使用集合和 Cocoa 数据类型* 中，我们看到了我们可以使用 `$0`、`$1`、`$2` 等参数将参数传递给数组算法。让我们看看如何使用简写语法来使用参数。我们将首先创建一个新的函数，该函数将接受一个带有单个参数的闭包。我们将把这个函数命名为 `testFunction2`。以下示例显示了新的 `testFunction2` 函数的作用：

```swift
func testFunction2(num: Int, handler:(name: String)->Void) {
  for var i=0; i < num; i++ {
    handler(name: "Me")
  }
}
```

在 `testFunction2` 函数中，我们这样定义我们的闭包：`(name: String)->Void`。这个定义意味着闭包接受一个参数并且不返回任何值。现在，让我们看看如何使用相同的简写语法来调用这个函数：

```swift
testFunction2(5,handler: {print("Hello from \($0)")}) 
```

这个闭包定义与上一个定义之间的区别在于 `$0`。`$0` 参数是传递给函数的第一个参数的简写。如果我们执行这段代码，它将打印出消息，`Hello from Me`，五次。

使用美元符号（`$`）后跟一个数字的简写语法允许我们在不将参数列表放入定义的情况下定义闭包。美元符号后面的数字定义了参数在参数列表中的位置。让我们更详细地检查这个格式，因为我们不仅限于在简写闭包中使用美元符号（`$`）和数字简写格式。这种简写语法还可以用来缩短闭包定义，允许我们省略参数名称。以下示例演示了这一点：

```swift
let clos5: (String, String) ->Void = {
    print("\($0) \($1)")
}
```

在这个示例中，我们的闭包定义了两个字符串参数，但我们没有给它们命名。参数的定义如下：`(String, String)`。我们可以在闭包体内部使用 `$0` 和 `$1` 访问这些参数。此外，请注意，闭包的定义是在冒号（`:`）之后，使用与定义变量类型相同的语法，而不是在花括号内。当我们使用匿名参数时，这就是我们定义闭包的方式。以下定义方式是不正确的：

```swift
let clos5b = {
    (String, String) -> Void in
    print("\($0) \($1)")
}
```

在这个示例中，我们将收到 `匿名闭包参数不能在具有显式参数的闭包中使用` 错误。

我们将像这样使用 `clos5` 闭包：

```swift
clos5("Hello","Kara")
```

由于 `Hello` 是参数列表中的第一个字符串，因此它使用 `$0` 访问，而 `Kara` 是参数列表中的第二个字符串，因此它使用 `$1` 访问。当我们执行这段代码时，我们将看到消息，`Hello Kara`，打印到控制台。

在这个下一个例子中，当闭包不返回任何值时使用。我们不必将返回类型定义为 `Void`，我们可以使用括号，如下例所示：

```swift
let clos6: () -> () = {
    print("Howdy")
}
```

在这个例子中，我们定义闭包为 `() -> ()`。这告诉 Swift 该闭包不接受任何参数，也不返回任何值。我们将这样执行这个闭包：

```swift
clos6()
```

在我们开始展示一些有用的闭包示例之前，我们还有一个简写闭包示例要演示。在这个最后的例子中，我们将演示如何在不需要包含单词`return`的情况下从闭包中返回一个值。

如果整个闭包体只包含一个语句，那么我们可以省略 `return` 关键字，并且该语句的结果将被返回。让我们看看一个这样的例子：

```swift
let clos7 = {
    (first: Int, second: Int) -> Int in
    first + second
}
```

在这个例子中，闭包接受两个 `Int` 类型的参数，并将返回 `Int` 类型的值。闭包体内的唯一语句是将第一个参数加到第二个参数上。然而，如果你注意到，我们在加法语句之前没有包含 `return` 关键字。Swift 会看到这是一个单语句闭包，并将自动返回结果，就像我们在加法语句之前放置了 `return` 关键字一样。我们确实需要确保我们语句的结果类型与闭包的返回类型相匹配。

在前两个部分中展示的所有示例都是为了展示如何定义和使用闭包。单独来看，这些示例并没有真正展示闭包的力量，也没有展示闭包是多么有用。本章的剩余部分旨在展示 Swift 中闭包的力量和实用性。

# 使用 Swift 的数组算法与闭包

在第三章中，我们探讨了几个可以与 Swift 的数组一起使用的内置算法。在第三章中，我们简要地看到了如何使用非常基本的闭包为这些算法中的每一个添加简单的规则。现在，我们对闭包有了更好的理解，让我们看看如何使用更高级的闭包来扩展这些算法。

在本节中，我们将主要使用 map 算法以保持一致性；然而，我们可以使用任何算法中展示的基本思想。我们将首先定义一个数组来使用：

```swift
let guests = ["Jon", "Kim", "Kailey", "Kara"]
```

这个数组包含一个名字列表，这个数组被命名为 `guests`。这个数组将在本节的所有示例中使用，除了最后几个。

现在我们有了 `guests` 数组，让我们添加一个闭包，该闭包将打印一个问候语到 `guests` 数组中的每个名字：

```swift
guests.map({
    (name: String) -> Void in
    print("Hello \(name)")
})
```

由于映射算法将闭包应用于数组中的每个项目，因此此示例将为`guests`数组中的每个名称打印一条问候语。在本章的第一部分之后，我们应该对闭包是如何工作的有一个相当好的理解。使用我们在上一节中看到的简写语法，我们可以将前面的示例简化为以下单行代码：

```swift
guests.map({print("Hello \($0)")})
```

在我看来，这可能是少数几个简写语法可能比标准语法更容易阅读的情况之一。

现在，假设我们不想将问候语打印到控制台，而是想返回一个包含问候语的新数组。为此，我们将在闭包中返回一个字符串类型，如下面的示例所示：

```swift
var messages = guests.map({
    (name:String) -> String in
    return "Welcome \(name)"
})
```

当此代码执行时，`messages`数组将包含对`guests`数组中每个名称的问候，而`guests`数组将保持不变。

本节前面的示例展示了如何将闭包内联添加到映射算法中。如果我们只想使用一个闭包与映射算法一起，这是很好的；但如果我们有多个闭包想要使用，或者我们想要多次使用闭包或用不同的数组重用它们，我们应该怎么做呢？为此，我们可以将闭包分配给一个常量或变量，然后根据需要使用其常量或变量名称传递闭包。让我们看看如何做到这一点。我们将首先定义两个闭包。其中一个闭包将为`guests`数组中的每个名称打印一条问候语，另一个闭包将为`guests`数组中的每个名称打印一条告别语：

```swift
let greetGuest = {
  (name:String) -> Void in
    print("Hello guest named \(name)")
}

let sayGoodbye = {
  (name:String) -> Void in
    print("Goodbye \(name)")
}
```

现在我们有了两个闭包，我们可以根据需要使用它们与映射算法一起。以下代码显示了如何将这些闭包与`guests`数组交互使用：

```swift
guests.map(greetGuest)
guests.map(sayGoodbye)
```

无论何时我们使用`greetGuest`闭包与`guests`数组一起，问候消息都会打印到控制台；无论何时我们使用`sayGoodbye`闭包与`guests`数组一起，告别消息都会打印到控制台。如果我们还有一个名为`guests2`的数组，我们可以使用相同的闭包来处理该数组，如下面的示例所示：

```swift
guests.map(greetGuest)
guests2.map(greetGuest)
guests.map(sayGoodbye)
guests2.map(sayGoodbye)
```

在本节中，到目前为止的所有示例要么是将消息打印到控制台，要么是从闭包返回一个新数组。我们的闭包并不局限于这样的基本功能。例如，我们可以在闭包内过滤数组，如下面的示例所示：

```swift
let greetGuest2 = {
  (name:String) -> Void in
    if (name.hasPrefix("K")) {
      print("\(name) is on the guest list")
    } else {
    print("\(name) was not invited")
    }
}
```

在此示例中，我们根据名称是否以字母 K 开头打印不同的消息。

如我们本章前面提到的，闭包具有捕获并存储它们定义的上下文中任何变量或常量的引用的能力。让我们来看一个这样的例子。假设我们有一个函数，它包含了一个特定位置过去七天的最高温度，并且这个函数接受一个闭包作为参数。这个函数将在温度数组上执行闭包。这个函数可以写成这样：

```swift
func temperatures(calculate:(Int)->Void) {
    var tempArray = [72,74,76,68,70,72,66]
    tempArray.map(calculate)

}
```

这个函数接受一个定义为 `(Int)->Void` 的闭包。然后我们使用映射算法为 `tempArray` 数组中的每个项目执行这个闭包。在这种情况下正确使用闭包的关键是理解 `temperatures` 函数并不知道或关心 `calculate` 闭包内部发生的事情。此外，请注意，闭包也无法更新或更改函数上下文中的项目，这意味着闭包不能更改温度函数中的任何其他变量；然而，它可以更新它在创建的上下文中的变量。

让我们看看我们将创建闭包的函数。我们将把这个函数命名为 `testFunction`。让我们看一下以下代码：

```swift
func testFunction() {
    var total = 0
    var count = 0
    let addTemps = {
      (num: Int) -> Void in
      total += num
      count++
    }
    temperatures(addTemps)
    print("Total: \(total)")
    print("Count: \(count)")
    print("Average: \(total/count)")
}
```

在这个函数中，我们首先定义了两个名为 `total` 和 `count` 的变量，这两个变量都是整数类型。然后我们创建了一个名为 `addTemps` 的闭包，它将被用来将 `temperatures` 函数中的所有温度相加。`addTemps` 闭包还将计算数组中有多少个温度。为此，`addTemps` 闭包计算数组中每个项目的总和，并将总和保存在函数开头定义的 `total` 变量中。`addTemps` 闭包还通过为每个项目递增 `count` 变量来跟踪数组中的项目数量。请注意，`total` 和 `count` 变量都不是在闭包内部定义的；然而，我们能够在闭包中使用它们，因为它们是在与闭包相同的上下文中定义的。

然后，我们调用 `temperatures` 函数，并传递 `addTemps` 闭包。最后，我们将总温度、计数和平均温度打印到控制台。当 `testFunction` 执行时，我们会在控制台看到以下输出：

```swift
Total: 498
Count: 7
Average: 71
```

如我们从输出中可以看到的，`addTemps` 闭包能够在创建它的上下文中更新和使用定义的项目，即使闭包在另一个上下文中使用。

现在我们已经探讨了使用闭包与数组映射算法相结合的方法，让我们来看看如何单独使用闭包。我们还将探讨我们可以采取的方法来清理我们的代码，使其更容易阅读和使用。

# 独立闭包和良好的风格指南

闭包让我们能够真正地将代码中的数据部分与用户界面和业务逻辑部分分离。这使我们能够创建仅关注数据检索的可重用类。这对于开发旨在从外部服务（如网络服务、数据库或文件）检索数据的类和框架尤其有用。本节将展示如何开发一个类，当数据准备好返回时，将执行一次闭包。

让我们从创建一个包含代码数据部分的类开始。在这个例子中，这个类将被命名为 `Guests`，它将包含一个嘉宾名字的数组。让我们看一下以下代码：

```swift
class Guests {
    var guestNames = ["Jon","Kim","Kailey","Kara","Buddy","Lily","Dash"]

    typealias UseArrayClosure = [String] -> Void   
    func getGuest(handler:UseArrayClosure) {
        handler(guestNames)
    }

}
```

`Guests` 类的第一行定义了一个名为 `guestNames` 的数组。`guestNames` 数组包含七个名字。在我们定义 `guestNames` 数组之后，我们接着创建一个类型别名。类型别名定义了一个现有类型的命名别名。就像函数一样，闭包有由参数类型和返回类型组成的类型，这些类型可以被别命名。这允许我们定义一次闭包，然后在代码的任何地方使用这个别名。使用类型别名可以减少我们需要输入的代码量，并防止错误。因此，建议我们使用它们，而不是在代码中多次重写闭包定义。它还允许我们在一个位置更改定义，然后整个代码将更新。

在这个例子中，我们的类型别名被命名为 `UseArrayClosure`，它被定义为一个接受一个字符串数组作为唯一参数且不返回任何值的闭包。现在我们可以在整个代码中使用这个类型别名作为闭包定义的简写。

最后，我们定义了一个 `getGuest()` 方法，它接受一个名为 `handler` 的闭包作为其唯一参数。在 `getGuests()` 方法内部，我们唯一要做的事情就是执行这个处理器。通常，在这个方法中，我们将有从外部数据源检索数据的逻辑；然而，在这个例子中，我们有一个硬编码的包含嘉宾名单的数组。因此，我们只需要使用 `guestsNames` 数组作为唯一参数来执行这个闭包。

现在，假设我们想在 `UITableView` 视图中显示这个名字数组。`UITableView` 是一个 iOS 视图，用于显示信息列表。在视图控制器中，我们需要创建一个数组来存储在 `UITableView` 中显示的数据，以及一个变量来链接到显示中的 `UITableView`。这两个都将是我们视图控制器类中的类变量，并且它们的定义如下：

```swift
@IBOutlet var tableView:UITableView?
var tableData: [String]?
```

现在，让我们创建一个名为 `getData()` 的函数，它将用于检索嘉宾列表并更新表格视图：

```swift
func getData() {
  let dataClosure: Guests.UseArrayClosure = {
    self.tableData = $0
    if let tView = self.tableView {
      tView.reloadData()
    }
  }

  let guests = Guests()
  guests.getGuest(dataClosure)
}
```

我们从定义一个名为`dataClosure`的闭包开始`getData()`函数。这个闭包使用了我们在`Guests`类中定义的`UseArrayClosure`类型别名来定义闭包。在闭包定义中，我们将定义在视图控制器本身内部（而不是在闭包中）的`tableData`数组设置为传递给闭包的字符串数组。然后我们验证`tableView`变量是否包含`UITableView`类的实例，如果是，我们重新加载数据。最后，我们创建一个`Guests`类的实例，并通过传递`dataClosure`闭包来调用`getGuest()`方法。

请记住，定义名称列表的`guestNames`数组是在`Guest`类中定义的，而`tableView`、`UITableView`和`tableData`数组是在视图控制器类中定义的。

当将`dataClosure`闭包传递给`getGuests()`方法时，它将从`Guests`类中加载名称数组到`tableData`数组中。然后`tableData`数组在视图控制器类中使用，作为`UITableView`数组的数据元素。在这个例子中需要注意的关键点是，我们能够将一个上下文（`Guests`类）中的数据加载到与闭包定义相同的上下文（视图控制器）中的变量，并且还有能力在定义在闭包相同上下文中的类的实例上调用方法（`tableView`和`UITableView`）。

我们很容易在`Guest`类中创建一个返回`guestNames`数组的方法。对于像`Guest`类中那样的硬编码数组，这个方法会工作得很好。然而，如果我们从需要一些时间来加载的 Web 服务加载数据，这将不会很好地工作，因为我们的 UI 会在等待数据加载时冻结。通过使用本例中所示的方法，我们可以异步调用 Web 服务，然后当数据返回时，闭包将被执行，UI 会自动更新而不会冻结我们的 UI。

### 注意

这本书主要是为了教授 Swift 语言，而不是专门针对 iOS 开发；因此，我们在这个例子中不涵盖 Cocoa Touch 框架中的 UI 元素是如何工作的。如果您想看到完整的 iOS 示例，请下载本书的代码示例。

# 功能更改

闭包还赋予我们动态更改类功能的能力。我们在第十一章中看到，*使用泛型*，泛型赋予我们编写适用于多种类型的函数的能力。通过闭包，我们能够编写功能和闭包作为参数传递给它时可以改变的功能的函数和类。在本节中，我们将展示如何编写一个可以通过闭包改变功能的函数。

让我们先定义一个类，用于演示如何替换功能。我们将这个类命名为 `TestClass`：

```swift
class TestClass {
  typealias getNumClosure = ((Int, Int) -> Int)

  var numOne = 5
  var numTwo = 8

  var results = 0
  func getNum(handler: getNumClosure) -> Int {
    results = handler(numOne,numTwo)
    return results

  }
}
```

我们从这个类开始，定义一个名为 `getNumClosure` 的闭包类型别名。任何定义为 `getNumClosure` 闭包的闭包都将接受两个整数并返回一个整数。在这个闭包中，我们假设它会对我们传入的整数做些处理以得到返回的值，但实际上并不需要。说实话，这个类并不关心闭包做了什么，只要它符合 `getNumClosure` 类型。接下来，我们定义三个名为 `numOne`、`NumTwo` 和 `results` 的整数。

最后，我们定义一个名为 `getNum()` 的方法。该方法接受一个确认 `getNumClosure` 类型的闭包作为其唯一参数。在 `getNum()` 方法中，我们通过传递 `numOne` 和 `numTwo` 类变量以及整数来执行闭包，返回的整数被放入 `results` 类变量中。

现在，让我们看看几个符合 `getNumClosure` 类型的闭包，我们可以使用这些闭包与 `getNum()` 方法一起使用：

```swift
var max: TestClass.getNumClosure = {
  if $0 > $1 {
    return $0
  } else {
    return $1
  }
}

var min: TestClass.getNumClosure = {
  if $0 < $1 {
    return $0
  } else {
    return $1
  }
}

var multiply:  TestClass.getNumClosure = {
  return $0 * $1
}

var second: TestClass.getNumClosure = {
  return $1
}

var answer: TestClass.getNumClosure = {
  var tmp = $0 + $1
  return 42
}
```

在此代码中，我们定义了五个符合 `getNumClosure` 类型的闭包：

+   `max`: 这返回传入的两个整数的最大值

+   `min`: 这返回传入的两个整数的最小值

+   `multiply`: 这将传入的两个值相乘并返回乘积

+   `second`: 这返回传入的第二个参数

+   `answer`: 这返回生命、宇宙和万物的答案

在 `answer` 闭包中，有一行看起来好像没有作用：`var tmp = $0 + $1`。我们故意这样做，因为以下代码是不合法的：

```swift
var answer: TestClass.getNumClosure = {
    return 42
}
```

这个类给我们提供了 `error: tuple types '(Int, Int)' and '()' have a different number of elements (2 vs. 0)` 错误。正如错误所示，Swift 认为除非我们在闭包体中使用 `$0` 和 `$1`，否则我们的闭包不接受任何参数。在名为 `second` 的闭包中，Swift 假设有两个参数，因为 `$1` 指定了第二个参数。

现在，我们可以将这些闭包逐个传递给我们的 `TestClass` 的 `getNum` 方法，以改变函数的功能以适应我们的需求。以下代码说明了这一点：

```swift
var myClass = TestClass()

myClass.getNum(max)
myClass.getNum(min)
myClass.getNum(multiply)
myClass.getNum(second)
myClass.getNum(answer)
```

当运行此代码时，我们将为每个闭包收到以下结果：

+   `max`: 结果 = 8

+   `min`: 结果 = 5

+   `multiply`: 结果 = 40

+   `second`: 结果 = 8

+   `answer`: 结果 = 42

本章将要展示的最后一个示例是在框架中经常使用的一个，尤其是在那些设计为异步运行的功能中。

# 根据结果选择闭包

在最后的例子中，我们将向一个方法传递两个闭包，然后根据某些逻辑，执行其中一个或两个闭包。通常情况下，如果方法成功执行，则调用其中一个闭包；如果方法失败，则调用另一个闭包。

让我们从创建一个类开始，这个类将包含一个方法，该方法将接受两个闭包，并根据定义的逻辑执行其中一个闭包。我们将这个类命名为`TestClass`。以下是`TestClass`类的代码：

```swift
class TestClass {
  typealias ResultsClosure = ((String) -> Void)

  func isGreater(numOne: Int, numTwo:Int, successHandler: ResultsClosure, failureHandler: ResultsClosure) {
    if numOne > numTwo {
      successHandler("\(numOne) is greater than \(numTwo)")
    }
    else {
      failureHandler("\(numOne) is not greater than \(numTwo)")
    }

  }
}
```

我们从这个类开始创建一个类型别名，该别名定义了我们将在成功和失败闭包中使用的闭包。我们将这个类型别名命名为`ResultsClosure`。这个例子还将说明为什么使用类型别名而不是重新输入闭包定义可以节省我们大量的输入，并防止我们出错。在这个例子中，如果我们没有使用类型别名，我们就需要四次重新输入闭包定义，如果我们需要更改闭包定义，我们就需要在四个地方进行更改。使用类型别名，我们只需要输入一次闭包定义，然后在剩余的代码中使用别名。

我们接着创建一个名为`isGreater`的方法，该方法接受两个整数作为前两个参数，然后接受两个闭包作为接下来的两个参数。第一个闭包命名为`successHandler`，第二个闭包命名为`failureHandler`。在`isGreater`方法内部，我们检查第一个整数参数是否大于第二个参数。如果第一个整数大于第二个，则执行`successHandler`闭包；否则，执行`failureHandler`闭包。

现在，让我们创建两个闭包。这两个闭包的代码如下：

```swift
var success: TestClass. ResultsClosure = {
    print("Success: \($0)")
}

var failure: TestClass. ResultsClosure = {
    print("Failure: \($0)")
}
```

注意，这两个闭包都被定义为`TestClass.ResultsClosure`类型。在每个闭包中，我们只是简单地打印一条消息到控制台，让我们知道哪个闭包被执行。通常情况下，我们会在闭包中放置一些功能。

然后，我们将像这样使用两个闭包调用该方法：

```swift
var test = TestClass()
test.isGreater(8, numTwo: 6, successHandler:success, failureHandler:failure)
```

注意，在方法调用中，我们发送了成功闭包和失败闭包。在这个例子中，我们会看到消息，“成功：8 大于 6”。如果我们反转数字，我们会看到消息，“失败：6 不大于 8”。这种用法在调用异步方法时非常好，例如从网络服务加载数据。如果网络服务调用成功，则调用成功闭包；否则，调用失败闭包。

使用这种闭包的一个大优点是，在等待网络服务调用完成时，UI 不会冻结。这也涉及到并发部分，我们将在本书的第十四章，*Swift 中的并发与并行*中稍后进行介绍。作为一个例子，如果我们尝试以这种方式从网络服务检索数据：

```swift
var data = myWebClass.myWebServiceCall(someParameter)
```

当我们等待响应返回时，我们的 UI 会冻结，或者我们必须在单独的线程中发起调用，这样 UI 就不会挂起。使用闭包，我们将闭包传递给网络框架，并依赖于框架在完成时执行适当的闭包。这确实依赖于框架正确实现并发来异步调用，但一个不错的框架应该为我们处理这一点。

# 使用闭包创建强引用循环

在本章的早期，我们说，“最好的是，大多数情况下，Swift 会为我们处理内存管理”。这句话中的“大多数情况下”部分意味着，如果一切按照标准方式编写，Swift 会为我们处理闭包的内存管理。然而，就像类一样，有时内存管理会让我们失望。到目前为止，本章中我们看到的所有示例的内存管理都将正常工作。有可能创建一个强引用循环，这将阻止 Swift 的内存管理正常工作。让我们看看如果我们使用闭包创建强引用循环会发生什么。

如果我们将一个闭包分配给类实例的一个属性，并在该闭包中捕获该类的实例，可能会发生强引用循环。这种捕获发生是因为我们使用`self`访问该特定实例的属性，例如`self.someProperty`，或者将`self`赋值给一个变量或常量，如`let c = self`。通过捕获实例的属性，我们实际上捕获了实例本身，从而创建了一个强引用循环，内存管理器将不知道何时释放实例。结果，内存将无法正确释放。

让我们从创建一个具有闭包和字符串类型实例作为其两个属性的类开始。我们还将在这个类中创建闭包类型的别名，并定义一个`deinit()`方法，该方法将打印一条消息到控制台。`deinit()`方法在类被释放和内存释放时被调用。当`deinit()`方法的消息打印到控制台时，我们将知道类何时被释放。这个类将被命名为`TestClassOne`。让我们看一下以下代码：

```swift
class TestClassOne {
  typealias nameClosure = (() -> String)

  var name = "Jon"

  lazy var myClosure: nameClosure =  {
    return self.name
  }

  deinit {
    print("TestClassOne deinitialized")
  }
}
```

现在，让我们创建第二个类，该类将包含一个接受`nameClosure`类型闭包的方法，该闭包是在`TestClassOne`类中定义的。这个类也将有一个`deinit()`方法，这样我们也可以看到它何时被释放。我们将这个类命名为`TestClassTwo`。让我们看一下以下代码：

```swift
class TestClassTwo {

  func closureExample(handler: TestClassOne.nameClosure) {
    print(handler())
  }

  deinit {
    print("TestClassTwo deinitialized")
  }
}
```

现在，让我们通过创建每个类的实例，然后尝试通过将它们设置为`nil`来手动释放实例，来看看这段代码的实际效果：

```swift
var testClassOne: TestClassOne? = TestClassOne()
var testClassTwo: TestClassTwo? = TestClassTwo()

testClassTwo?.closureExample(testClassOne!.myClosure)

testClassOne = nil
print("testClassOne is gone")

testClassTwo = nil
print("testClassTwo is gone")
```

在这段代码中，我们创建了两个可选变量，它们可能包含我们的两个测试类的实例或`nil`。我们需要将这些变量创建为可选的，因为我们在代码的后面将它们设置为`nil`，这样我们就可以看到实例是否被正确释放。

然后我们调用`TestClassTwo`实例的`closureExample()`方法，并将`TestClassOne`实例的`myClosure`属性传递给它。我们现在尝试通过将它们设置为`nil`来释放`TestClassOne`和`TestClassTwo`实例。记住，当一个类的实例被释放时，如果存在，它会尝试调用该类的`deinit()`方法。在我们的例子中，两个类都有一个`deinit()`方法，它会向控制台打印一条消息，这样我们就可以知道实例是否真正被释放了。

如果我们运行这个项目，我们将看到以下消息打印到控制台：

```swift
testClassOne is gone
TestClassTwo deinitialized
testClassTwo is gone
```

如我们所见，我们确实尝试释放`TestClassOne`实例，但类的`deinit()`方法从未被调用，这表明它实际上并没有被释放；然而，`TestClassTwo`实例被正确释放，因为那个类的`deinit()`方法被调用了。

要查看在没有强引用循环的情况下它是如何工作的，将`myClosure`闭包更改为返回在闭包内部定义的字符串类型，如下所示：

```swift
lazy var myClosure: nameClosure = {
  return "Just Me"
}
```

现在，如果我们运行项目，我们应该看到以下输出：

```swift
TestClassOne deinitialized
testClassOne is gone
TestClassTwo deinitialized
testClassTwo is gone
```

这表明`TestClassOne`和`TestClassTwo`实例的`deinit()`方法都得到了正确调用，表明它们都被正确释放了。

在第一个例子中，我们在闭包中捕获了`TestClassOne`类的一个实例，因为我们使用`self.name`访问了`TestClassOne`类的一个属性。这从闭包到`TestClassOne`类的一个实例创建了一个强引用，阻止了内存管理释放该实例。

Swift 确实提供了一个非常简单且优雅的方式来解决闭包中的强引用循环。我们只需通过创建捕获列表来告诉 Swift 不要创建强引用。捕获列表定义了在闭包中捕获引用类型时要使用的规则。我们可以声明每个引用为弱引用或`unowned`引用，而不是强引用。

当在引用的生命周期中存在引用可能变为`nil`的可能性时，使用`weak`关键字；因此，类型必须是可选的。当没有引用变为`nil`的可能性时，使用`unowned`关键字。

我们通过将`weak`或`unowned`关键字与一个类实例的引用配对来定义捕获列表。这些配对在方括号`[ ]`内书写。因此，如果我们更新`myClosure`闭包并定义一个指向`self`的`unowned`引用，我们应该消除强引用循环。以下代码显示了新的`myClosure`闭包将类似的样子：

```swift
lazy var myClosure: nameClosure =  {
  [unowned self] in
  return self.name
}
```

注意新的一行——`[unowned self] in`。这一行表示我们不希望创建对 `self` 实例的强引用。如果我们现在运行项目，我们应该看到以下输出：

```swift
TestClassOne deinitialized
testClassOne is gone
TestClassTwo deinitialized
testClassTwo is gone
```

这表明 `TestClassOne` 和 `TestClassTwo` 实例都得到了适当的释放。

# 摘要

在本章中，我们看到了我们可以像定义整型或字符串类型一样定义闭包。我们可以将闭包分配给变量，将它们作为函数的参数传递，也可以从函数中返回它们。

闭包捕获了从定义闭包的上下文中引用的任何常量或变量的存储引用。我们必须小心使用这个功能，以确保我们不会创建一个强引用循环，这会导致我们的应用程序中出现内存泄漏。

Swift 闭包与 Objective-C 中的块非常相似，但它们的语法更加简洁和优雅。这使得它们更容易使用和理解。

对闭包有良好的理解对于掌握 Swift 编程语言至关重要，这将使开发易于维护的 OS X 和 iOS 应用程序变得更加容易。这对于创建可以用于创建 OS X 和 iOS 应用程序的一等框架也是必不可少的。

本章中我们看到的三个用例绝不是闭包的唯一三个*有用*用途。我可以向你保证，你越在 Swift 中使用闭包，你将发现它们的应用越多。闭包无疑是 Swift 语言中最强大和最有用的特性之一，苹果在语言中实现它们做得非常出色。
