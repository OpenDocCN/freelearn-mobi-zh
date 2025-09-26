

# 第十二章：错误处理和可用性

当我开始用 Objective-C 编写应用程序时，最明显的不足之一是缺乏异常处理。大多数现代编程语言，如 Java 和 C#，使用 `try...catch` 块或类似的结构进行异常处理。虽然 Objective-C 确实有 `try...catch` 块，但它并没有在 Cocoa 框架内部使用，而且它从未真正感觉像是语言的一部分。我有丰富的 C 语言经验，因此我能够理解苹果的框架如何接收和响应错误。说实话，我有时甚至更喜欢这种方法，尽管我已经习惯了 Java 和 C# 中的异常处理。当 Swift 首次推出时，我希望能看到苹果将真正的错误处理加入语言中，这样我们就有选择使用它的选项；然而，它并没有在 Swift 的初始版本中。直到 Swift 2 发布，苹果才将错误处理添加到 Swift 中。虽然这种错误处理看起来可能类似于 Java 和 C# 中的异常处理，但它们之间有一些非常显著的不同之处。

我们在本章中将涵盖以下主题：

+   如何表示错误

+   如何在 Swift 中使用 `do-catch` 块

+   如何使用 `defer` 语句

+   如何使用可用性属性

让我们开始吧！

# 原生错误处理

类似于 Java 和 C# 这样的语言通常将错误处理过程称为异常处理。在 Swift 文档中，苹果将这个过程称为错误处理。虽然从外表上看，Java 和 C# 的异常处理可能看起来与 Swift 的错误处理有些相似，但那些熟悉其他语言中异常处理的人会在本章中注意到一些显著的差异。

## 表示错误

在我们真正理解 Swift 中的错误处理工作原理之前，我们必须看看我们如何表示一个错误。在 Swift 中，错误由符合 `Error` 协议的类型值表示。Swift 的枚举非常适合建模错误条件，因为我们通常有有限数量的错误条件需要表示。

让我们看看我们如何使用枚举来表示错误。为此，我们将定义一个虚构的错误 `MyError`，它有三个错误条件：`Minor`、`Bad` 和 `Terrible`：

```swift
enum MyError: Error { 
    case Minor
    case Bad
    case Terrible
} 
```

在这个例子中，我们定义了 `MyError` 枚举符合 `Error` 协议，并且定义了三个错误条件：`Minor`、`Bad` 和 `Terrible`。这就是定义基本错误条件所需要做的全部。

我们还可以使用与我们的错误条件关联的值来添加有关错误条件的更多详细信息。比如说，我们想要向 `Terrible` 错误条件添加一个描述。我们会这样做：

```swift
enum MyError: Error { 
    case Minor
    case Bad
    case Terrible(description:String)
} 
```

熟悉 Java 和 C#中异常处理的人会发现，在 Swift 中表示错误要干净得多，也容易得多，因为我们不需要创建大量的样板代码或一个完整的类。在 Swift 中，它可能只需要定义一个包含我们的错误条件的枚举。另一个优点是，定义多个错误条件并将它们分组在一起非常容易，这样所有相关的错误条件都是同一类型的。

现在，让我们学习如何在 Swift 中建模错误。为了这个例子，我们将看看我们如何为一个棒球队分配球员号码。对于一个棒球队，每个被召回的新球员都会被分配一个唯一的号码。这个号码也必须在一定的范围内，因为只有两个号码可以放在棒球球衣上。

因此，我们将有三个错误条件：号码太大，号码太小，以及号码不唯一。以下示例显示了我们可以如何表示这些错误条件：

```swift
enum PlayerNumberError: Error {
    case NumberTooHigh(description: String) 
    case NumberTooLow(description: String) 
    case NumberAlreadyAssigned
} 
```

使用`PlayerNumberError`类型，我们定义了三种非常具体的错误条件，这些条件确切地告诉我们出了什么问题。由于这些错误条件都与分配球员号码相关，因此它们也被组合在一个类型中。

这种定义错误的方法允许我们定义非常具体的错误，这样当发生错误条件时，我们的代码可以确切地知道出了什么问题。它还允许我们将错误分组，这样所有相关的错误都可以定义在同一个类型中。

现在我们知道了如何表示错误，让我们看看如何抛出错误。

## 抛出错误

当函数中发生错误时，调用该函数的代码必须知道这一点；这被称为**抛出错误**。当函数抛出错误时，它假设调用该函数的代码，或者链中的某些代码，将捕获并适当地从错误中恢复。

要从函数中抛出错误，我们使用`throws`关键字。这个关键字让调用它的代码知道函数可能会抛出错误。与其他语言的异常处理不同，我们不需要列出可能抛出的具体错误类型。

由于我们不在函数定义中列出可能从函数中抛出的具体错误类型，因此在函数的文档和注释中列出它们是一种良好的实践。这允许使用该函数的其他开发者知道应该捕获哪些错误类型。

很快，我们将看看如何抛出错误。但首先，让我们向之前定义的`PlayerNumberError`类型添加一个第四个错误。这展示了如何轻松地向我们的错误类型添加错误条件。这个错误条件是在我们尝试通过号码检索球员，但没有球员被分配该号码时抛出的。

新的`PlayerNumberError`类型现在看起来将类似于以下内容：

```swift
enum PlayerNumberError: Error {
    case NumberTooHigh(description: String) 
    case NumberTooLow(description: String) 
    case NumberAlreadyAssigned
    case NumberDoesNotExist
} 
```

为了演示如何抛出错误，让我们创建一个包含给定球队球员列表的 `BaseballTeam` 结构。这些球员将被存储在一个名为 `players` 的字典对象中。我们将使用球员的号码作为键，因为我们知道每个球员都必须有一个唯一的号码。用于表示单个球员的 `BaseballPlayer` 类型将是一个元组类型的 `typealias`，定义如下：

```swift
typealias BaseballPlayer = (firstName: String, lastName: String, number: Int) 
```

在这个 `BaseballTeam` 结构中，我们将有两个方法。第一个方法将被命名为 `addPlayer()`。这个方法将接受一个 `BaseballPlayer` 类型的参数，并尝试将球员添加到球队中。这个方法也可以抛出三种错误条件之一：`NumberTooHigh`、`NumberTooLow` 或 `NumberAlreadyExists`。以下是这个方法的写法：

```swift
mutating func addPlayer(player: BaseballPlayer) throws { 
    guard player.number < maxNumber else {
        throw PlayerNumberError.NumberTooHigh(description: "Max         number is \(maxNumber)")
    }
    guard player.number > minNumber else {
        throw PlayerNumberError.NumberTooLow(description: "Min number         is \(minNumber)")
    }
    guard players[player.number] == nil else {
        throw PlayerNumberError.NumberAlreadyAssigned
    }
    players[player.number] = player
} 
```

我们可以看到，`throws` 关键字被添加到了方法的定义中。`throws` 关键字让任何调用此方法的代码知道它可能会抛出错误，并且必须处理这个错误。然后我们使用三个 `guard` 语句来验证数字不是太大，不是太小，并且在 `players` 字典中是唯一的。如果这些条件中的任何一个不满足，我们将使用 `throw` 关键字抛出适当的错误。如果我们通过了所有三个检查，那么球员就会被添加到 `players` 字典中。

我们将要添加到 `BaseballTeam` 结构中的第二个方法是 `getPlayerByNumber()` 方法。这个方法将尝试检索被分配了给定数字的棒球运动员。如果没有球员被分配了该数字，这个方法将抛出 `NumberDoesNotExist` 错误。`getPlayerByNumber()` 方法将看起来像这样：

```swift
func getPlayerByNumber(number: Int) throws -> BaseballPlayer { 
    if let player = players[number] {
        return player
    } else {
        throw PlayerNumberError.NumberDoesNotExist
    }
} 
```

我们也把这个 `throws` 关键字添加到了这个方法定义中；然而，这个方法还有一个返回类型。当我们使用 `throws` 关键字与返回类型一起时，它必须放在方法定义中的返回类型之前。

在方法内部，我们尝试检索传递给方法的具有该数字的棒球运动员。如果我们能检索到球员，我们就返回它；否则，我们抛出 `NumberDoesNotExist` 错误。注意，如果我们从具有返回类型的方法中抛出错误，则不需要返回值。

现在，让我们学习如何使用 Swift 捕获错误。

# 错误处理

当一个函数抛出错误时，我们需要在调用它的代码中捕获它；这是通过使用 `do-catch` 块来完成的。我们在 `do-catch` 块中使用 `try` 关键字来标识代码中可能抛出错误的地方。带有 `try` 语句的 `do-catch` 块具有以下语法：

```swift
do {
    try [Some function that throws]
    [Code if no error was thrown]
} catch [pattern] {
    [Code if function threw error]
} 
```

如果抛出了错误，它将传播出去，直到被 `catch` 子句处理。`catch` 子句由 `catch` 关键字组成，后面跟着一个用于匹配错误的模式。如果错误与模式匹配，则执行 `catch` 块内的代码。

让我们通过调用 `BaseballTeam` 结构的 `getPlayerByNumber()` 和 `addPlayer()` 方法来查看如何使用 `do-catch` 块。首先让我们看看 `getPlayerByNumber()` 方法，因为它只抛出一个错误条件：

```swift
do {
    let player = try myTeam.getPlayerByNumber(number: 34) 
    print("Player is \(player.firstName) \(player.lastName)")
} catch PlayerNumberError.NumberDoesNotExist { 
    print("No player has that number")
} 
```

在这个示例中，`do-catch` 块调用了 `BaseballTeam` 结构的 `getPlayerByNumber()` 方法。如果没有球员被分配这个号码，该方法将抛出 `NumberDoesNotExist` 错误条件；因此，我们尝试在 `catch` 语句中匹配这个错误。

任何在 `do-catch` 块内抛出错误时，块内的剩余代码将被跳过，并且将执行匹配该错误的 `catch` 块内的代码。因此，在我们的示例中，如果 `getPlayerByNumber()` 方法抛出 `NumberDoesNotExist` 错误条件，第一个 `print` 语句永远不会被执行。

我们不需要在 `catch` 语句后面包含一个模式。如果 `catch` 语句后面没有包含模式，或者我们放入一个下划线，`catch` 语句将匹配所有错误条件。例如，以下两个 `catch` 语句中的任何一个都将捕获所有错误：

```swift
do {
    // our statements
} catch {
    // our error conditions
}
do {
    // our statements
} catch _ {
    // our error conditions
} 
```

如果我们想要捕获错误，我们可以使用 `let` 关键字，如下面的示例所示：

```swift
do {
    // our statements
} catch let error { 
    print("Error:\(error)")
} 
```

现在，让我们看看如何使用 `catch` 语句，类似于 `switch` 语句，来捕获不同的错误条件。为此，我们将调用 `BaseballTeam` 结构的 `addPlayer()` 方法：

```swift
do {
    try myTeam.addPlayer(player:("David", "Ortiz", 34))
} catch PlayerNumberError.NumberTooHigh(let description) { 
    print("Error: \(description)")
} catch PlayerNumberError.NumberTooLow(let description) { 
    print("Error: \(description)")
} catch PlayerNumberError.NumberAlreadyAssigned { 
    print("Error: Number already assigned")
} 
```

在这个示例中，我们有三个 `catch` 语句。每个 `catch` 语句都有一个不同的模式来匹配；因此，它们将分别匹配不同的错误条件。如您所回忆的，`NumberTooHigh` 和 `NumberToLow` 错误条件有相关值。要检索相关值，我们使用括号内的 `let` 语句，如前例所示。

总是好的做法是将你的最后一个 `catch` 语句留空，这样它就会捕获之前 `catch` 语句中所有模式未匹配的错误。因此，前面的示例应该重写如下：

```swift
do {
    try myTeam.addPlayer(player:("David", "Ortiz", 34))
} catch PlayerNumberError.NumberTooHigh(let description) { 
    print("Error: \(description)")
} catch PlayerNumberError.NumberTooLow(let description) {
    print("Error: \(description)")
} catch PlayerNumberError.NumberAlreadyAssigned { 
    print("Error: Number already assigned")
} catch {
    print("Error: Unknown Error")
} 
```

我们也可以让错误传播出去而不是立即捕获它们。为此，我们只需在函数定义中添加 `throws` 关键字。例如，在以下示例中，我们不是捕获错误，而是让它传播到调用函数的代码中，如下所示：

```swift
func myFunc() throws {
    try myTeam.addPlayer(player:("David", "Ortiz", 34))
} 
```

如果我们确定不会抛出错误，我们可以使用强制尝试表达式来调用函数，该表达式写作 `try!`。强制尝试表达式禁用了错误传播，并将函数调用包装在一个运行时断言中，因此不会从这个调用中抛出错误。如果抛出错误，您将得到运行时错误，所以使用这个表达式时要非常小心。

强烈建议你在生产代码中避免使用强制尝试表达式，因为它可能导致运行时错误并导致你的应用程序崩溃。

当我在 Java 和 C# 等语言中处理异常时，我看到很多空的 `catch` 块。这就是我们需要捕获异常的地方，因为可能会抛出异常；然而，我们不想对它做任何事情。在 Swift 中，代码看起来可能像这样：

```swift
do {
    let player = try myTeam.getPlayerByNumber(number: 34) 
    print("Player is \(player.firstName) \(player.lastName)")
} catch {} 
```

这样的代码是我不喜欢异常处理的地方之一。嗯，Swift 开发者对此有一个答案：`try?`。这尝试执行可能会抛出错误的操作，并将其转换为可选值；因此，操作的结果将是如果抛出错误则为 `nil`，如果没有抛出错误则为操作的结果。

由于 `try?` 的结果以可选形式返回，我们通常会与可选绑定一起使用它。我们可以将前面的示例重写如下：

```swift
if let player = try? myTeam.getPlayerByNumber(number: 34) { 
    print("Player is \(player.firstName) \(player.lastName)")
} 
```

如我们所见，这使得我们的代码更加整洁且易于阅读。

如果我们需要执行清理操作，无论是否发生错误，我们都可以使用 `defer` 语句。我们使用 `defer` 语句在代码执行离开当前作用域之前执行一段代码。以下示例显示了我们可以如何使用 `defer` 语句：

```swift
func deferFunction(){ 
    print("Function started") 
    var str: String?
    defer {
        print("In defer block")
        if let s = str {
            print("str is \(s)")
        }
    }
    str = "Jon"
    print("Function finished")
} 
```

如果我们调用这个函数，控制台首先打印的将是 `Function started`。代码的执行将跳过 `defer` 块，然后控制台将打印 `Function finished`。最后，在离开函数的作用域之前，将执行 `defer` 块的代码，我们会看到 `In defer block` 的消息。以下是这个函数的输出：

```swift
Function started
Function finished
In defer block
str is Jon 
```

`defer` 块将在执行离开当前作用域之前始终被调用，即使抛出了错误。当我们需要确保执行所有必要的清理操作，即使抛出了错误时，`defer` 语句非常有用。例如，如果我们成功打开一个文件进行写入，我们总是想确保关闭该文件，即使我们在写入操作中遇到错误。

在这种情况下，我们可以将文件关闭功能放在 `defer` 块中，以确保在离开当前作用域之前文件总是被关闭。

# 多模式捕获子句

在前面的章节中，我们有一些看起来像这样的代码：

```swift
do {
    try myTeam.addPlayer(player:("David", "Ortiz", 34))
} catch PlayerNumberError.NumberTooHigh(let description) { 
    print("Error: \(description)")
} catch PlayerNumberError.NumberTooLow(let description) {
    print("Error: \(description)")
} catch PlayerNumberError.NumberAlreadyAssigned { 
    print("Error: Number already assigned")
} catch {
    print("Error: Unknown Error")
} 
```

你会注意到，对于 `PlayerNmberError.NumberTooHigh` 和 `PlayerNumberError.NumberTooLow` 错误的 `catch` 子句包含重复的代码。在开发过程中，总是好的找到一种方法来消除这样的重复代码。然而，在 Swift 5.3 之前，我们没有选择。Swift 通过 Swift 5.3 中的 SE-0276 引入了多模式 `catch` 子句，以帮助减少这种重复代码。让我们通过重写前面的代码来使用多模式 `catch` 子句来查看这一点：

```swift
do {
    try myTeam.addPlayer(player:("David", "Ortiz", 34))
} catch PlayerNumberError.NumberTooHigh(let description), PlayerNumberError.NumberTooLow(let description) { 
    print("Error: \(description)")
} catch PlayerNumberError.NumberAlreadyAssigned { 
    print("Error: Number already assigned")
} catch {
    print("Error: Unknown Error")
} 
```

注意，在第一个 `catch` 子句中，我们现在正在捕获 `PlayerNmberError.NumberTooHigh` 和 `PlayerNumberError.NumberTooLow` 两个错误，并且错误之间用逗号分隔。

接下来，我们将探讨如何使用新的可用性属性与 Swift 一起使用。

# 可用性属性

为最新的 **操作系统**（**OS**）版本开发我们的应用程序使我们能够访问我们正在开发平台的所有最新功能。然而，有时我们还想针对旧平台。Swift 允许我们使用可用性属性安全地包装代码，以便仅在操作系统正确版本可用时运行。这最初是在 Swift 2 中引入的。

可用性属性仅在我们在 Apple 平台上使用 Swift 时才可用。

可用性代码块本质上允许我们，如果我们正在运行指定的操作系统版本或更高版本，运行此代码或运行其他代码。我们可以使用可用性属性有两种方式。第一种方式允许我们执行一个特定的代码块，该代码块可以与 `if` 或 `guard` 语句一起使用。第二种方式允许我们将方法或类型标记为仅在特定平台上可用。

可用性属性接受最多六个以逗号分隔的参数，这允许我们定义执行我们的代码所需的操作系统的最低版本或应用扩展。这些参数如下：

+   `iOS`：这是我们代码兼容的最低 iOS 版本。

+   `OSX`：这是我们代码兼容的最低 OS X 版本。

+   `watchOS`：这是我们代码兼容的最低 watchOS 版本。

+   `tvOS`：这是我们代码兼容的最低 tvOS 版本。

+   `iOSApplicationExtension`：这是我们代码兼容的最低 iOS 应用扩展。

+   `OSXApplicationExtension`：这是我们代码兼容的最低 OS X 应用扩展。

在参数之后，我们指定所需的最低版本。我们只需要包含与我们的代码兼容的参数。例如，如果我们正在编写 iOS 应用程序，我们只需要在可用性属性中包含 iOS 参数。我们以一个 `*`（星号）结束参数列表，因为它代表未来的版本。让我们看看如何仅在我们满足最低要求时执行特定的代码块：

```swift
if #available(iOS 9.0, OSX 10.10, watchOS 2, *) {
    //Available for iOS 9, OSX 10.10, watchOS 2 or above
    print("Minimum requirements met")
} else {
    //Block on anything below the above minimum requirements
    print("Minimum requirements not met")
} 
```

在这个例子中，`if #available(iOS 9.0, OSX 10.10, watchOS 2, *)` 这行代码防止在运行在不符合指定最低操作系统版本的系统上时执行代码块。在这个例子中，我们还使用了 `else` 语句，如果操作系统不符合最低要求，将执行另一段代码。

我们还可以限制对函数或类型的访问。在前面的代码中，`available` 属性前面加了 `#`（井号，也称为 **octothorpe** 和 **hash**）字符。为了限制对函数或类型的访问，我们需要在 `available` 属性前加上一个 `@`（在）字符。以下示例展示了我们如何限制对类型和函数的访问：

```swift
@available(iOS 9.0, *)
    func testAvailability() {
        // Function only available for iOS 9 or above
}
@available(iOS 9.0, *)
    struct TestStruct {
        // Type only available for iOS 9 or above
} 
```

在前面的示例中，我们指定了只有在代码在具有 iOS 9 或更高版本的设备上运行时，`testAvailability()` 函数和 `testStruct()` 类型才能被访问。为了使用 `@available` 属性来阻止对函数或类型的访问，我们必须将调用该函数或类型的代码用 `#available` 属性包裹起来。

以下示例展示了我们如何调用 `testAvailability()` 函数：

```swift
if #available(iOS 9.0, *) { 
    testAvailability()
} else {
    // Fallback on earlier versions
} 
```

在这个示例中，`testAvailability()` 函数只有在应用程序运行在具有 iOS 9 或更高版本的设备上时才会被调用。

# 摘要

在本章中，我们探讨了 Swift 的错误处理功能。虽然我们不需要在我们的自定义类型中使用这些功能，但它们确实为我们提供了一种统一的方式来处理和响应错误。苹果公司也开始在其框架中使用这种错误处理形式。建议我们在代码中使用错误处理。

我们还探讨了 `availability` 属性，它允许我们开发能够利用目标操作系统的最新功能的应用程序，同时仍然允许我们的应用程序在旧版本上运行。在下一章中，我们将探讨如何编写自定义下标。
