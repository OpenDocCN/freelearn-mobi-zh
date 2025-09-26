# 14

# 与闭包一起工作

现在，大多数主要的编程语言都有类似于 Swift 中闭包的功能。其中一些实现非常难以使用（Objective-C 的 blocks），而其他则相对容易（Java 的 lambdas 和 C# 的 delegates）。我发现闭包提供的功能在开发框架时特别有用。我也在通过网络连接与远程服务通信时广泛使用了它们。虽然 Objective-C 中的 blocks 非常有用，但声明 block 所使用的语法绝对糟糕。幸运的是，当 Apple 开发 Swift 语言时，他们使闭包的语法更容易使用和理解。

在本章中，我们将涵盖以下主题：

+   什么是闭包？

+   如何创建闭包

+   如何使用闭包

+   有哪些有用的闭包示例？

+   如何避免闭包内的强引用循环

# 闭包简介

闭包是包含代码块的自包含代码块，可以在我们的应用程序中传递和使用。我们可以将 `Int` 类型视为包含整数的类型，将 `String` 类型视为包含字符串的类型。在这种情况下，闭包可以被视为包含代码块的类型。这意味着我们可以将闭包赋值给变量，将它们作为参数传递给函数，并从函数中返回它们。

闭包可以捕获并存储它们定义上下文中的任何变量或常量的引用。这被称为覆盖变量或常量，并且大部分情况下，Swift 会为我们处理内存管理。唯一的例外是在创建强引用循环时，我们将在第十八章“内存管理”的“使用闭包创建强引用循环”部分中探讨如何解决这个问题。

Swift 中的闭包与 Objective-C 中的 blocks 类似；然而，Swift 中的闭包使用起来更容易，也更易于理解。让我们看看在 Swift 中定义闭包所使用的语法：

```swift
{
(<#parameters#>) -> <#return-type#> in <#statements#>
} 
```

创建闭包所使用的语法看起来与我们用来创建函数的语法非常相似，在 Swift 中，全局和嵌套函数都是闭包。闭包和函数之间格式上的最大区别是 `in` 关键字。`in` 关键字用于代替花括号，将闭包的参数和返回类型的定义与闭包体分开。

闭包有很多用途，我们将在本章后面讨论其中的一些，但首先，我们需要了解闭包的基本知识。让我们先看看一些非常基础的闭包，以便我们更好地理解它们是什么，如何定义它们，以及如何使用它们。

# 简单闭包

我们将从创建一个非常简单的闭包开始，这个闭包不接受任何参数，也不返回任何值。它所做的只是将 `Hello World` 打印到控制台。让我们看看以下代码：

```swift
let clos1 = { () -> Void in
    print("Hello World")
} 
```

在这个例子中，我们创建了一个闭包并将其赋值给`clos1`常量。由于括号之间没有定义任何参数，这个闭包将不接受任何参数。此外，返回类型被定义为`Void`；因此，这个闭包不会返回任何值。闭包的主体包含一行，将`Hello World`打印到控制台。

使用闭包的方法有很多；在这个例子中，我们只想执行它。我们可以按照以下方式执行闭包：

```swift
clos1() 
```

在执行闭包之后，我们会看到`Hello World`被打印到了控制台。在这个阶段，闭包可能看起来并不那么有用，但随着我们继续学习本章内容，我们将看到它们是多么有用和强大。

让我们看看另一个简单的例子。这个闭包将接受一个名为`name`的`String`参数，但不会返回任何值。在闭包的主体内部，我们将打印出通过`name`参数传递给闭包的问候语。这是第二个闭包的代码：

```swift
let clos2 = {
    (name: String) -> Void in 
    print("Hello \(name)")
} 
```

`clos2`闭包和`clos1`闭包之间最大的区别是我们定义了一个单独的`String`参数在括号之间。正如我们所看到的，我们为闭包定义参数的方式就像我们为函数定义参数一样。我们可以以执行`clos1`闭包相同的方式执行这个闭包。下面的代码展示了如何做到这一点：

```swift
clos2("Jon") 
```

当这个例子执行时，将会打印出`Hello Jon`的消息到控制台。让我们看看另一种我们可以使用`clos2`闭包的方法。在这个例子中需要注意的一点是，闭包中的命名参数不需要使用参数名。

我们对闭包的原始定义是：闭包是自包含的代码块，可以在我们的应用程序中传递和使用。这告诉我们我们可以将我们的闭包从它们被创建的上下文中传递到代码的其他部分。让我们看看如何将我们的`clos2`闭包传递给一个函数。我们将定义一个函数，它接受我们的`clos2`闭包，如下所示：

```swift
func testClosure(handler: (String) -> Void) { 
    handler("Dasher")
} 
```

我们定义函数的方式就像定义任何其他函数一样；然而，在参数列表中，我们定义了一个名为`handler`的参数，并为`handler`参数定义了类型`(String) -> Void`。如果我们仔细观察，我们可以看到`handler`参数的`(String) -> Void`定义与为`clos2`闭包定义的参数和返回类型相匹配。这意味着我们可以将`clos2`闭包传递给函数。让我们看看如何做到这一点：

```swift
testClosure(handler: clos2) 
```

我们像调用任何其他函数一样调用 `testClosure()` 函数，传入的闭包看起来就像任何其他变量一样。由于 `clos2` 闭包在 `testClosure()` 函数中被执行，因此当这段代码执行时，我们将在控制台看到打印出的消息 `Hello Dasher`。正如我们将在本章稍后看到的那样，将闭包传递给函数的能力使得闭包如此令人兴奋和强大。作为闭包拼图的最后一部分，让我们看看如何从闭包中返回一个值。以下示例展示了这一点：

```swift
let clos3 = {
    (name: String) -> String in
    return "Hello \(name)"
} 
```

`clos3` 闭包的定义看起来与我们之前定义的 `clos2` 闭包非常相似。不同之处在于我们将返回类型从 `Void` 改为了 `String` 类型。然后，在闭包体中，我们不是将消息打印到控制台，而是使用返回语句返回消息。现在我们可以像之前两个闭包一样执行 `clos3` 闭包，或者像处理 `clos2` 闭包那样将闭包传递给函数。以下示例展示了如何执行 `clos3` 闭包：

```swift
var message = clos3("Buddy") 
```

执行此行代码后，消息变量将包含 `Hello Buddy` 字符串。前面三个闭包示例展示了闭包的格式和定义典型闭包的方法。熟悉 Objective-C 的人可以注意到，Swift 中闭包的格式要干净得多，也更容易使用。我们在本章中迄今为止展示的闭包创建语法相当简短；然而，我们还可以进一步缩短它。在下一节中，我们将探讨如何做到这一点。

# 闭包的简写语法

在本节中，我们将探讨几种缩短语法的途径。

使用闭包的简写语法完全是个人喜好问题。许多开发者喜欢将他们的代码尽可能小和紧凑，并且为此感到非常自豪。然而，有时这会使代码对其他开发者来说难以阅读和理解。

我们将要查看的第一个闭包简写语法是最受欢迎的之一，这是我们使用 *第五章* 中数组算法时看到的语法。这种格式主要用于当我们想要向函数发送一个非常小的闭包（通常是单行）时，就像我们处理数组算法那样。在我们查看这种简写语法之前，我们需要编写一个接受闭包作为参数的函数：

```swift
func testFunction(num: Int, handler:() -> Void) { 
    for _ in 0..<num {
        handler()
    }
} 
```

此函数接受两个参数；第一个参数是一个名为 `num` 的整数，第二个参数是一个名为 `handler` 的闭包，该闭包没有参数，也不返回任何值。在函数内部，我们创建一个 `for` 循环，该循环将使用 `num` 整数来定义循环的次数。在 `for` 循环中，我们调用传入函数的 `handler` 闭包。现在让我们创建一个闭包并将其传递给 `testFunction()`，如下所示：

```swift
let clos = { () -> Void in
    print("Hello from standard syntax")
}
testFunction(num: 5, handler: clos) 
```

这段代码非常易于阅读和理解；然而，它需要五行代码。现在让我们看看如何通过在函数调用内编写闭包来缩短它：

```swift
testFunction(num: 5,handler: {print("Hello from Shorthand closure")}) 
```

在这个例子中，我们使用与数组算法相同的语法在函数调用内行内创建了闭包。闭包放在两个花括号（`{}`）之间，这意味着创建闭包的代码是 `{print("Hello from Shorthand closure")}`。当这段代码执行时，它会在屏幕上打印出五次`Hello from Shorthand closure`消息。调用 `testFunction()` 时，为了简洁性和可读性，理想的方式如下：

```swift
testFunction(num: 5) {
    print("Hello from Shorthand closure")
} 
```

将闭包作为最后一个参数允许我们在调用函数时省略标签。这个例子给出了既紧凑又易读的代码。让我们看看如何使用这种简写语法来使用参数。我们将首先创建一个新的函数，该函数将接受一个具有单个参数的闭包。我们将把这个函数命名为 `testFunction2`。以下示例显示了新的 `testFunction2` 函数的作用：

```swift
func testFunction2(num: Int, handler: (_ : String)->Void) { 
    for _ in 0..<num {
        handler("Me")
    }
} 
```

在 `testFunction2` 中，我们这样定义闭包：`(_ : String)->Void`。这个定义意味着闭包接受一个参数并且不返回任何值。现在让我们看看如何使用相同的简写语法来调用这个函数：

```swift
testFunction2(num: 5){ 
    print("Hello from \($0)")
} 
```

这个闭包定义与之前的定义之间的区别是 `$0`。`$0` 参数是传递给函数的第一个参数的简写。如果我们执行这段代码，它会打印出五次`Hello from Me`消息。使用美元符号（`$`）后跟一个数字的行内闭包允许我们在定义中不需要创建参数列表就能定义闭包。美元符号后面的数字定义了参数在参数列表中的位置。让我们更详细地考察一下这个格式，因为我们不仅限于只使用美元符号（`$`）和数字简写格式来定义行内闭包。这种简写语法也可以用来缩短闭包定义，允许我们省略参数名称。以下示例演示了这一点：

```swift
let clos5: (String, String) -> Void = { 
    print("\($0) \($1)")
} 
```

在这个例子中，闭包定义了两个字符串参数；然而，我们没有给它们命名。参数是这样定义的：（`String`，`String`）。我们可以在闭包体中使用 `$0` 和 `$1` 访问这些参数。此外，请注意，闭包定义在冒号（`:`）之后，使用的是我们用来定义变量类型的相同语法，而不是在花括号内。当我们使用匿名参数时，这就是我们定义闭包的方式。以下定义闭包的方式是不正确的：

```swift
let clos5b = { (String, String) in 
    print("\($0) \($1)")
} 
```

在这个例子中，我们会收到一个错误，告诉我们这种格式是无效的。接下来，让我们看看如何使用 `clos5` 闭包：

```swift
clos5("Hello", "Kara") 
```

由于 `Hello` 是参数列表中的第一个字符串，它通过 `$0` 访问，而 `Kara` 作为参数列表中的第二个字符串，通过 `$1` 访问。当我们执行这段代码时，我们将看到 `Hello Kara` 消息打印到控制台。下一个示例用于当闭包不返回任何值时。我们不必将返回类型定义为 `Void`，我们可以使用括号，如下例所示：

```swift
let clos6: () -> () = { 
    print("Howdy")
} 
```

在这个例子中，我们将闭包定义为 `() -> ()`。这告诉 Swift 该闭包不接受任何参数，也不返回任何值。我们将如下执行这个闭包：

```swift
clos6() 
```

作为个人偏好，我并不特别喜欢这种简写语法。我认为当使用 `Void` 关键字而不是括号时，代码更容易阅读。

在我们开始展示一些非常有用的闭包示例之前，我们还有一个简写闭包示例要展示。在这个最后的例子中，我们将演示我们如何从闭包中返回一个值，而无需包含 `return` 关键字。如果整个闭包体只包含一个语句，我们可以省略 `return` 关键字，该语句的结果将被返回。让我们看看一个这样的例子：

```swift
let clos7 = {(first: Int, second: Int) -> Int in first + second } 
```

在这个例子中，闭包接受两个 `Int` 类型的参数，并将返回一个 `Int` 类型的实例。闭包体内的唯一语句是将第一个参数添加到第二个参数。然而，如果你注意到，我们在附加语句之前没有包含 `return` 关键字。Swift 会看到这是一个单语句闭包，并将自动返回结果，就像我们在附加语句之前放置了 `return` 关键字一样。我们确实需要确保我们语句的结果类型与闭包的返回类型相匹配。

前两个部分中展示的所有示例都是为了展示如何定义和使用闭包。单独来看，这些示例并没有真正展示闭包的力量，也没有展示闭包是多么有用。本章的剩余部分将展示 Swift 中闭包的力量和实用性。

# 使用 Swift 数组与闭包

在 *第五章*，*使用 Swift 集合* 中，我们查看了一些可以与 Swift 数组一起使用的内置算法。在那一章中，我们简要地看了如何使用非常基本的闭包为这些算法中的每一个添加简单的规则。现在我们更好地理解了闭包，让我们看看我们如何可以使用更高级的闭包来扩展这些算法。

## 使用 Swift 的数组算法与闭包

在本节中，我们将主要使用 `map` 算法以保持一致性；然而，我们可以使用任何算法中展示的基本思想。我们将首先定义一个数组来使用：

```swift
let guests = ["Jon", "Kailey", "Kara"] 
```

这个数组包含一个名字列表，这个数组被命名为`guests`。这个数组将用于本节的大部分示例。现在我们有了`guests`数组，让我们添加一个闭包，该闭包将打印数组中每个名字的问候语：

```swift
guests.map { name in 
    print("Hello \(name)")
} 
```

由于`map`算法将闭包应用于数组的每个项目，此示例将为数组中的每个名字打印一个问候语。在本章的第一部分之后，我们应该对闭包的工作原理有一个相当好的理解。使用我们在上一节中看到的简写语法，我们可以将前面的示例简化为以下单行代码：

```swift
guests.map {
    print("Hello \($0)")} 
```

在我看来，这可能是少数几次，简写语法可能比标准语法更容易阅读的情况之一。现在，假设我们不是要将问候语打印到控制台，而是想要返回一个包含问候语的新数组。为此，我们应从闭包中返回一个`String`类型，如下面的示例所示：

```swift
var messages = guests.map { 
    (name:String) -> String in
    return "Welcome \(name)"
} 
```

当这段代码执行时，`messages`数组将包含对`guests`数组中每个名字的问候，而数组本身将保持不变。我们可以按如下方式访问问候语：

```swift
for message in messages { 
    print("\(message)")
} 
```

本节前面的示例展示了如何在`map`算法中内联添加闭包。如果我们只想使用一个闭包与`map`算法，这是很好的，但如果我们想使用多个闭包，或者想多次使用闭包或在不同数组中重用它们呢？为此，我们可以将闭包分配给一个常量或变量，然后根据需要使用闭包的常量或变量名。让我们看看如何做到这一点。我们将首先定义两个闭包。

其中一个闭包将打印数组中每个元素的问候语，另一个闭包将打印数组中每个元素的告别信息：

```swift
let greetGuest = { (name:String) -> Void in 
    print("Hello guest named \(name)")
}
let sayGoodbye = { (name:String) -> Void in 
    print("Goodbye \(name)")
} 
```

现在我们有了两个闭包，我们可以根据需要使用它们与`map`算法。下面的代码展示了如何将这些闭包与`guests`数组交互使用：

```swift
guests.map(greetGuest)
guests.map(sayGoodbye) 
```

当我们使用`greetGuest`闭包与`guests`数组一起使用时，问候信息将被打印到控制台；当我们使用`sayGoodbye`闭包与`guests`数组一起使用时，告别信息将被打印到控制台。如果我们还有一个名为`guests2`的数组，我们可以使用相同的闭包来处理该数组，如下面的示例所示：

```swift
guests.map(greetGuest)
guests2.map(greetGuest)
guests.map(sayGoodbye)
guests2.map(sayGoodbye) 
```

到目前为止，本节中的所有示例要么是将消息打印到控制台，要么是从闭包中返回一个新数组。我们的闭包并不局限于这样的基本功能。例如，我们可以在闭包内过滤数组，如下面的示例所示：

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

在这个例子中，我们根据名字是否以字母`K`开头打印不同的消息。

如本章前面所述，闭包具有捕获并存储它们定义上下文中任何变量或常量的引用的能力。让我们来看一个这样的例子。假设我们有一个函数，它包含了一个特定位置过去七天的最高温度，并且这个函数接受一个闭包作为参数。这个函数将在温度数组的数组上执行闭包。这个函数可以写成如下形式：

```swift
func temperatures(calculate:(Int)->Void) { 
    var tempArray = [72,74,76,68,70,72,66] 
    tempArray.map(calculate)
} 
```

这个函数接受一个闭包，定义为`(Int)-> Void`。然后我们使用`map`算法来执行`tempArray`数组中的每个元素的闭包。在这种情况下正确使用闭包的关键是理解`temperatures`函数不知道，也不关心`calculate`闭包内部的任何操作。此外，请注意，闭包也无法更新或更改函数上下文中的项目，这意味着闭包不能更改温度函数中的任何其他变量；然而，它可以更新它在创建上下文中定义的变量。

让我们看看我们将创建闭包的函数。我们将把这个函数命名为`testFunction`：

```swift
func testFunction() { 
    var total = 0
    var count = 0
    let addTemps = {
        (num: Int) -> Void in 
        total += num 
        count += 1
    }
    temperatures(calculate: addTemps) 
    print("Total: \(total)")
    print("Count: \(count)") 
    print("Average: \(total/count)")
} 
```

在这个函数中，我们首先定义了两个变量，分别命名为`total`和`count`，这两个变量都是`Int`类型。然后我们创建了一个名为`addTemps`的闭包，它将被用来将`temperatures`函数中的所有温度加在一起。`addTemps`闭包还将计算数组中的温度数量。为此，`addTemps`闭包计算数组中每个项目的总和，并将总和保存在函数开头定义的`total`变量中。`addTemps`闭包还通过为每个项目递增`count`变量来跟踪数组中的项目数量。请注意，`total`和`count`变量都没有在闭包内部定义；然而，我们能够在闭包中使用它们，因为它们是在与闭包相同的上下文中定义的。

然后我们调用`temperatures`函数，并传递`addTemps`闭包。最后，我们将`Total`、`Count`和`Average`温度打印到控制台。当`testFunction`执行时，我们将在控制台看到以下输出：

```swift
Total: 498
Count: 7
Average: 71 
```

如输出所示，`addTemps`闭包能够更新和使用在它创建的上下文中定义的项目，即使闭包在另一个上下文中使用。

现在我们已经看了如何使用数组`map`算法与闭包一起使用，让我们看看如何单独使用闭包。我们还将看看我们可以如何清理代码，使其更容易阅读和使用。

## 数组中的非连续元素

在*第五章*，*使用 Swift 集合*中，我们展示了如何使用带有闭包的`subrange`方法从数组中检索非连续的子元素。现在我们知道了更多关于闭包的知识，让我们再次看看这个例子。

我们在*第五章*，*使用 Swift 集合*中使用的代码，从整数数组中检索了偶数。让我们再次看看这段代码：

```swift
var numbers = [1,2,3,4,5,6,7,8,9,10]
let evenNum = numbers.subranges(where: { $0.isMultiple(of: 2) })
//numbers[evenNum] contains 2,4,6,8,10 
```

在这个例子中，我们使用了`Int`类型的`isMultiple(of:)`来检索所有偶数元素。由于`subranges(where:)`方法接受一个闭包，我们也可以使用其他逻辑。例如，如果我们想检索所有等于或小于 6 的元素，我们可以使用以下代码行：

```swift
let newNumbers = numbers.subrange(where: { $0 <= 6 }) 
```

现在我们熟悉了闭包，我们可以看到`subrange(where:)`方法的一些可能性。

## 未初始化的数组

Swift 5.2 与 SE-0245 一起引入了一个新的数组初始化器，它不会预先使用默认值填充值。这个初始化器使我们能够提供一个闭包来填充我们想要的任何值。让我们通过创建一个将包含 20 次掷骰子值的数组来看看如何做：

```swift
let capacity = 20
let diceRolls = Array<Int>(unsafeUninitializedCapacity: capacity) { buffer, initializedCount in
    for x in 0..<capacity {
        buffer[x] = Int.random(in: 1...6)
    }
    initializedCount = capacity
} 
```

我们首先设置一个包含数组容量的常量。我们这样做是因为我们需要在初始化器中的几个地方提供这个值，通过使用常量，如果我们以后需要更改这个容量，那么只需要在一个地方更改。

代码的其余部分初始化了数组。闭包提供了一个不安全的可变缓冲区指针，我们在前面的代码中将其命名为`buffer`，也可以用它来为数组写入值。我们使用`for`循环用 1 到 6 的随机整数填充数组。

当你使用这个初始化器时，有一些通用规则：

+   你不需要使用你请求的全部容量；然而，你不能使用更多。在我们的例子中，我们请求 20 个元素的容量，这意味着我们可以使用少于 20 个，但不能使用超过 20 个。

+   如果你没有初始化一个元素，那么它可能被填充了随机数据（一个非常糟糕的想法）。在我们的例子中，我们请求 20 个元素的容量，如果我们只填充了前 10 个元素，那么后面的 10 个元素将包含随机垃圾。

+   如果`initializedCount`没有设置，则默认为`0`，所有数据都将丢失。

+   这是一个非常方便的初始化器，但也很容易出错。我们也可以使用`map`数组算法重写`diceRolls`初始化器，如下面的代码所示：

    ```swift
    let diceRolls = (0…20).map { _ in Int.random(in: 1…6) } 
    ```

    然而，这会效率更低，因为像第一个例子中那样初始化数组在内部优化了更好的性能。如果你不担心最佳性能，那么映射算法更容易阅读和理解。

现在让我们看看我们如何可以使用闭包在运行时改变功能。

# 改变功能

闭包还赋予我们动态改变类型功能的能力。在 *第十一章*，*泛型* 中，我们看到了泛型赋予我们编写适用于多种类型的函数的能力。通过闭包，我们能够编写功能和类型可以根据传入的闭包而变化的函数。在本节中，我们将向您展示如何编写一个可以通过闭包改变功能的函数。

让我们从定义一个类型开始，这个类型将用来演示如何替换功能。我们将把这个类型命名为 `TestType`：

```swift
struct TestType {
    typealias GetNumClosure = ((Int, Int) -> Int)
    var numOne = 5 
    var numTwo = 8
    var results = 0;
    mutating func getNum(handler: GetNumClosure) -> Int { 
        results = handler(numOne,numTwo)
        print("Results: \(results)")
        return results
    }
} 
```

我们从这个类型开始，定义了一个名为 `GetNumClosure` 的闭包 `typealias`。任何定义为 `GetNumClosure` 闭包的闭包都将接受两个整数并返回一个整数。在这个闭包内部，我们假设它会对我们传入的整数做些处理以得到返回的值，但实际上它并不需要与整数有任何关系。说实话，这个类并不关心闭包做了什么，只要它符合 `GetNumClosure` 类型即可。接下来，我们定义了三个整数，分别命名为 `numOne`、`numTwo` 和 `results`。

我们还定义了一个名为 `getNum()` 的方法。这个方法接受一个符合 `GetNumClosure` 类型的闭包作为其唯一参数。在 `getNum()` 方法内部，我们通过传入 `numOne` 和 `numTwo` 变量以及返回的整数来执行闭包，并将返回的整数放入 `results` 类变量中。现在让我们看看几个符合 `GetNumClosure` 类型的闭包，我们可以使用这些闭包与 `getNum()` 方法一起使用：

```swift
var max: TestType.GetNumClosure = { 
    if $0 > $1 {
        return $0
    } else { 
        return $1
    }
}
var min: TestType.GetNumClosure = { 
    if $0 < $1 {
        return $0
    } else {
        return $1
    }
}
var multiply: TestType.GetNumClosure = { 
    return $0 * $1
}
var second: TestType.GetNumClosure = { 
    return $1
}
var answer: TestType.GetNumClosure = { 
    var _ = $0 + $1
    return 42
} 
```

在此代码中，我们定义了五个符合 `GetNumClosure` 类型的闭包：

+   `max`：返回传入的两个整数中的最大值

+   `in min`：返回传入的两个整数中的最小值

+   `in multiply`：将传入的两个值相乘并返回乘积

+   `second`：返回传入的第二个参数

+   `answer`：返回生命、宇宙和万物的答案

在 `answer` 闭包中，有一行看起来似乎没有作用：

```swift
var _ = $0 + $1 
```

我们故意这样做，因为以下代码是不合法的：

```swift
var answer: TestType.GetNumClosure = {
    return 42
} 
```

这种类型会给我们一个错误：闭包参数列表的上下文类型期望两个参数，这些参数不能被隐式忽略。正如错误所示，Swift 不允许我们在闭包体中忽略期望的参数。在第二个闭包中，Swift 假设有两个参数，因为 `$1` 指定了第二个参数。现在我们可以将这些闭包中的每一个传递给 `getNum()` 方法，以改变函数的功能以适应我们的需求。以下代码说明了这一点：

```swift
var myType = TestType()
myType.getNum(handler: max)
myType.getNum(handler: min)
myType.getNum(handler: multiply)
myType.getNum(handler: second)
myType.getNum(handler: answer) 
```

当运行此代码时，我们将为每个闭包收到以下结果：

```swift
For Max: 
Results: 8
For Min: 
Results: 5
For Multiply: 
Results: 40
For Second: 
Results: 8
For Answer: 
Results: 42 
```

我们将要展示的最后一个例子是在框架中经常使用的一个，尤其是在那些设计为异步运行的功能性框架中。

# 根据结果选择闭包

在最后的例子中，我们将向一个方法传递两个闭包，然后根据某些逻辑，一个或两个闭包将被执行。通常，如果方法成功执行，则调用一个闭包；如果方法失败，则调用另一个闭包。

让我们首先创建一个类型，该类型将包含一个方法，该方法将接受两个闭包，然后根据定义的逻辑执行其中一个闭包。我们将这个名字命名为`TestType`。以下是`TestType`类型的代码：

```swift
class TestType {
    typealias ResultsClosure = ((String) -> Void)

    func isGreater(numOne: Int, numTwo: Int, successHandler: ResultsClosure,failureHandler: ResultsClosure) {
        if numOne > numTwo {
            successHandler("\(numOne) is greater than \(numTwo)")
        }
        else {
            failureHandler("\(numOne) is not greater than \(numTwo)")
        }

    }
} 
```

我们首先通过创建一个`typealias`来定义我们将要用于成功和失败闭包的闭包。我们将这个名字命名为`typealiasResultsClosure`。这个例子也说明了为什么你应该使用`typealias`而不是重新输入闭包定义。这可以节省我们大量的输入，并防止我们出错。在这个例子中，如果我们不使用`typealias`，我们就需要重新输入闭包定义四次，如果我们需要更改闭包定义，我们就需要在四个地方进行更改。使用类型别名，我们只需要输入一次闭包定义，然后在整个剩余的代码中使用别名。

然后，我们创建一个名为`isGreater`的方法，它接受两个整数作为前两个参数，以及两个闭包作为接下来的两个参数。第一个闭包命名为`successHandler`，第二个闭包命名为`failureHandler`。在这个方法内部，我们检查第一个整数参数是否大于第二个。如果第一个整数大于第二个，则执行`successHandler`闭包；否则，执行`failureHandler`闭包。现在，让我们在`TestType`结构外部创建两个闭包。这两个闭包的代码如下：

```swift
var success: TestType.ResultsClosure = { 
    print("Success: \($0)")
}
var failure: TestType.ResultsClosure = {
    print("Failure: \($0)")
} 
```

注意，两个闭包都被定义为`TestClass.ResultsClosure`类型。在每个闭包中，我们只是简单地打印一条消息到控制台，让我们知道哪个闭包被执行。通常，我们会在闭包中放置一些功能。然后我们将调用方法，如下所示：

```swift
var test = TestType()
test.isGreater(numOne: 8, numTwo: 6, successHandler: success, failureHandler: failure) 
```

注意，在方法调用中，我们发送了成功闭包和失败闭包。在这个例子中，我们将看到`Success: 8 is greater than 6`的消息。如果我们反转数字，我们会看到`Failure: 6 is not greater than 8`的消息。这种用例在调用异步方法时非常好，例如从网络服务加载数据。如果网络服务调用成功，则调用成功闭包；否则，调用失败闭包。

使用这种闭包的一个大优点是，在我们等待异步调用完成时，UI 不会冻结。这也涉及到并发部分，我们将在第十六章中介绍，即 Swift 中的并发与并行。作为一个例子，想象我们尝试以下方式从网络服务中检索数据：

```swift
var data = myWebClass.myWebServiceCall(someParameter) 
```

当我们等待响应时，我们的 UI 会冻结，或者我们必须在单独的线程中进行调用，这样 UI 就不会挂起。有了闭包，我们将闭包传递给网络框架，并依赖框架在完成时执行适当的闭包。这依赖于框架正确实现并发，以异步方式进行调用，但一个不错的框架应该为我们处理这一点。

# 摘要

在本章中，我们了解到我们可以定义闭包，就像定义整数或字符串类型一样。我们可以将闭包赋值给变量，将它们作为函数的参数传递，并从函数中返回它们。闭包会捕获定义闭包时上下文中的任何常量或变量的强引用。我们必须小心使用这个功能，以确保我们不创建强引用循环，这会导致我们的应用程序中出现内存泄漏。

Swift 中的闭包与 Objective-C 中的 blocks 非常相似，但它们的语法更加简洁和优雅。这使得它们更容易使用和理解。对闭包有良好的理解对于掌握 Swift 编程语言至关重要，这将使开发易于维护的优秀应用程序变得更加容易。它们对于创建既易于使用又易于维护的一等框架也是必不可少的。

在本章中我们探讨的使用案例绝非闭包的唯一有用用例。我可以向你保证，你在 Swift 中使用闭包越多，你会发现它们的应用越广泛。闭包无疑是 Swift 语言中最强大和最有用的特性之一，而苹果通过实现它们做得非常出色。

在下一章中，我们将探讨如何使用 Swift 提供的高级位运算符，以及如何创建我们自己的自定义运算符。
