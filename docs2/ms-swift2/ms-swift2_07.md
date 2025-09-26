# 第七章：使用可用性和错误处理编写更安全的代码

当我最初开始用 Objective-C 编写 iOS 和 OS X 应用程序时，最明显的 *缺陷* 是在处理 Cocoa 和 Cocoa Touch 框架时缺乏异常处理。大多数现代编程语言，如 Java 和 C#，都使用 `try-catch` 块或类似机制来处理异常。虽然 Objective-C 确实有 `try-catch` 块，但它并没有在 Cocoa 框架内部使用，并且它从未感觉像是语言的一部分。我确实有丰富的 C 语言经验，所以我能够理解 Cocoa 和 Cocoa Touch 框架如何接收和响应错误，并且坦白说，我实际上更喜欢这种方法，尽管我已经习惯了使用 Java 和 C# 进行异常处理。当 Swift 首次推出时，我希望能看到 Apple 将真正的错误处理集成到语言中，这样我们就有选择使用它的选项；然而，它并没有包含在 Swift 的初始版本中。现在随着 Swift 2 的推出，Apple 已经将错误处理添加到了 Swift 中。虽然这种错误处理看起来可能类似于 Java 和 C# 中的异常处理，但有一些非常显著的不同之处。

本章我们将涵盖以下主题：

+   如何在 Swift 中使用 `do-catch` 块

+   如何表示错误

+   如何使用可用性属性

# Swift 2.0 之前的错误处理

错误处理是我们应用程序中响应和恢复错误条件的过程。在 Swift 2.0 之前，错误报告遵循与 Objective-C 相同的模式；然而，随着 Swift 的推出，我们确实有使用可选返回值的额外好处，其中返回 nil 会指示函数中存在错误。

在最简单的错误处理形式中，函数的返回值将指示其是否成功。这个返回值可以是简单的布尔值 true/false，也可以是更复杂的枚举，其值表示如果函数失败，实际上发生了什么。如果我们需要报告关于发生的错误的附加信息，我们可以添加一个 `NSError` 输出参数，其类型为 `NSErrorPointer`，但这并不是最容易的方法，并且这些错误往往被开发者忽略。以下示例说明了在 Swift 2.0 之前通常是如何处理错误的：

```swift
var str = "Hello World"
var error: NSError

var results = str.writeToFile(path, atomically: true, encoding: NSUTF8StringEncoding, error: &error)

if results {
  // successful code here
} else {
    println("Error writing filer:  \(error)")
}
```

虽然以这种方式处理错误效果良好，并且可以根据大多数需求进行修改，但它绝对不是完美的解决方案。这个解决方案有几个问题，其中最大的问题是开发者很容易忽略返回的值以及错误本身。虽然大多数经验丰富的开发者都会非常小心地检查所有错误，但有时，对于新手开发者来说，很难理解应该检查什么以及何时检查，尤其是如果函数不包含 `NSError` 参数的话。

除了使用 `NSError`，我们还可以使用 `NSException` 类来抛出和捕获异常；然而，实际上很少开发者使用这种方法。即使在 Cocoa 和 Cocoa Touch 框架中，这种异常处理方法也很少被使用。

虽然使用 `NSError` 类和返回值来处理错误是可行的，但包括我在内很多人对 Apple 在 Swift 最初发布时没有包含额外的错误处理功能感到失望。然而，现在随着 Swift 2.0 的发布，我们确实有了本地的错误处理功能。

# Swift 2 中的错误处理

类似于 Java 和 C# 这样的语言通常将错误处理过程称为 *异常处理*；在 Swift 文档中，Apple 将这个过程称为 *错误处理*。虽然从外观上看，Java 和 C# 的异常处理可能与 Swift 的错误处理非常相似，但熟悉其他语言异常处理的开发者会在本章中注意到一些显著的不同之处。

## 表示错误

在我们真正理解 Swift 中的错误处理机制之前，我们首先需要了解我们如何表示一个错误。在 Swift 中，错误是通过符合 `ErrorType` 协议的类型值来表示的。Swift 的枚举非常适合于建模错误条件，因为通常我们只需要表示有限数量的错误条件。

让我们看看我们如何使用枚举来表示一个错误。为此，我们将定义一个虚构的错误名为 `MyError`，包含三个错误条件：`Minor`、`Bad` 和 `Terrible`：

```swift
enum MyError: ErrorType {
    case Minor
    case Bad
    case Terrible
}
```

在这个例子中，我们定义 `MyError` 枚举符合 `ErrorType` 协议。然后我们定义三个错误条件：`Minor`、`Bad` 和 `Terrible`。我们还可以使用关联值来表示错误条件。假设我们想要给其中一个错误条件添加一个描述；我们可以这样做：

```swift
enum MyError: ErrorType {
    case Minor
    case Bad
    case Terrible (description: String)
}
```

对于熟悉 Java 和 C# 中的异常处理的开发者来说，他们可能会发现 Swift 中的错误表示要干净得多，也更容易。我们拥有的另一个优点是定义多个错误条件并将它们分组在一起非常容易，因此所有相关的错误条件都属于同一类型。

现在，让我们看看我们如何在 Swift 中建模错误。为了这个例子，让我们看看我们如何为一个棒球队分配球员号码。在棒球队中，每个被召回的新球员都会被分配一个唯一的号码，这个号码也必须在一定的号码范围内。在这种情况下，我们会遇到三个错误条件：号码太大、号码太小或号码不唯一。以下示例展示了我们如何表示这些错误条件：

```swift
enum PlayerNumberError: ErrorType {
    case NumberTooHigh(description: String)
    case NumberTooLow(description: String)
    case NumberAlreadyAssigned
}
```

使用 `PlayerNumberError` 类型，我们定义了三个非常具体的错误条件，这些条件可以确切地告诉我们出了什么问题。这些错误条件也分组在同一个类型中，因为它们都与分配玩家号码相关。

这种定义错误的方法允许我们定义非常具体的错误，让我们的代码在发生错误条件时确切地知道出了什么问题，正如我们在示例中看到的那样，它还允许我们将错误分组，因此所有相关的错误都可以在同一个类型中定义。

现在我们知道了如何表示错误，让我们看看我们如何抛出错误。

## 抛出错误

当函数中发生错误时，调用函数的代码必须知道这一点；这被称为**抛出错误**。当函数抛出错误时，它假设调用函数的代码，或者链中的某些代码，将捕获并适当地从错误中恢复。

要从函数中抛出错误，我们使用`throws`关键字。这个关键字让调用它的代码知道函数可能会抛出错误。与其它语言的异常处理不同，我们不需要列出可能抛出的具体错误类型。

### 注意

由于我们不在函数定义中列出可能抛出的具体错误类型，因此在文档和注释中列出这些错误类型是一种良好的实践，这样其他使用我们函数的开发者就会知道应该捕获哪些错误类型。

让我们看看我们如何抛出错误，但首先，让我们向之前定义的`PlayerNumberError`类型中添加一个第四个错误。如果尝试通过球员的号码检索球员但未分配该号码，则抛出此错误条件。新的`PlayerNumberError`类型现在看起来类似于以下内容：

```swift
enum PlayerNumberError: ErrorType {
    case NumberTooHigh(description: String)
    case NumberTooLow(description: String)
    case NumberAlreadyAssigned
    case NumberDoesNotExist
}
```

为了演示如何抛出错误，我们将首先创建一个`BaseballTeam`结构体，它将包含一个给定队伍的球员列表。这些球员将存储在一个名为`players`的字典对象中，并使用球员的号码作为键。用于表示单个球员的`BaseballPlayer`类型，将是一个元组类型的`typealias`，其定义如下：

```swift
typealias BaseballPlayer = (firstName: String, lastName: String, number: Int)
```

在这个`BaseballTeam`结构体中，我们将有两个方法。第一个方法将被命名为`addPlayer()`。这个方法将有一个`BaseballPlayer`类型的参数，并尝试将球员添加到队伍中。这个方法可能会抛出三种错误条件之一：`NumberTooHigh`、`NumberTooLow`或`NumberAlreadyExists`。以下是这个方法的实现方式：

```swift
    mutating func addPlayer(player: BaseballPlayer) throws {

        guard player.number < maxNumber else {
            throw PlayerNumberError.NumberTooHigh(description: "Max number is \(maxNumber)")
        }

        guard player.number > minNumber else {
            throw PlayerNumberError.NumberTooLow(description: "Min number is \(minNumber)")
        }

        guard players[player.number] == nil else {
            throw PlayerNumberError.NumberAlreadyAssigned
        }
        players[player.number] = player
    }
```

在方法定义中，我们看到添加了`throws`关键字。`throws`关键字让调用此方法的任何代码都知道它可能会抛出错误，并且必须处理这些错误。然后我们使用了三个`guard`语句。这些`guard`语句用于验证数字不是太大、不是太小，并且在`players`字典中是唯一的。如果任何条件不满足，我们将使用`throw`关键字抛出适当的错误。

如果我们通过了所有三个检查，球员将被添加到`players`字典中。

我们将要添加到 `BaseballTeam` 结构体的第二个方法是 `getPlayerByNumber()` 方法。这个方法将尝试检索被分配了给定编号的棒球运动员。如果没有球员被分配了这个编号，这个方法将抛出 `PlayerNumberError.NumberDoesNotExist` 错误。`getPlayerByNumber()` 方法将看起来像这样：

```swift
func getPlayerByNumber(number: Int) throws -> BaseballPlayer {
    if let player = players[number] {
        return player
    } else {
        throw PlayerNumberError.NumberDoesNotExist
    }
}
```

在这个方法定义中，我们看到它可以抛出错误，因为我们使用了定义中的 `throws` 关键字。`throws` 关键字必须放在方法定义中的 `return` 类型之前。

在方法中，我们尝试检索传递给方法编号的棒球运动员。如果我们能够检索到球员，我们就返回它；否则，我们抛出 `PlayerNumberError.NumberDoesNotExist` 错误。注意，如果我们从一个具有 `return` 类型的方法中抛出错误，我们不需要返回一个值。

现在我们来看看我们如何用 Swift 捕获一个错误。

## 捕获错误

当从函数中抛出错误时，我们需要在调用函数的代码中捕获它；这是通过使用 `do-catch` 块来完成的。`do-catch` 块采用以下语法：

```swift
do {
    try [Some function that throws]
    [Any additional code]
} catch [pattern] {
    [Code if function threw error]
}
```

如果抛出错误，它将传播出去，直到被 `catch` 子句处理。`catch` 子句由 `catch` 关键字组成，后跟一个用于匹配错误的模式。如果错误与模式匹配，则执行 `catch` 块内的代码。

让我们来看看我们如何通过调用 `BaseballTeam` 结构体的 `getPlayerByNumber()` 和 `addPlayer()` 方法来使用 `do-catch` 块。首先让我们看看 `getPlayerByNumber()` 方法，因为它只抛出一个错误条件：

```swift
do {
    let player = try myTeam.getPlayerByNumber(34)
    print("Player is \(player.firstName) \(player.lastName)")
} catch PlayerNumberError.NumberDoesNotExist {
    print("No player has that number")
}
```

在这个例子中，`do-catch` 块调用了 `BaseballTeam` 结构体的 `getPlayerByNumber()` 方法。如果队中没有球员被分配了这个号码，该方法将抛出 `PlayerNumberError.NumberDoesNotExist` 错误条件；因此，我们在 `catch` 语句中尝试匹配这个错误。

任何时候在 `do-catch` 块中抛出错误，块内的其余代码将被跳过，并且执行与错误匹配的 `catch` 块内的代码。因此，在我们的例子中，如果 `getPlayerByNumber()` 方法抛出 `PlayerNumberError.NumberDoesNotExist` 错误，那么 `print()` 函数永远不会被执行。

我们不需要在 `catch` 语句后包含一个模式。如果 `catch` 语句后没有包含模式或者我们放置一个下划线，`catch` 语句将匹配所有错误条件。例如，以下两个 `catch` 语句中的任何一个都将捕获所有错误：

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
    print("Error:  \(error)")
}
```

现在我们来看看我们如何使用 `catch` 语句，类似于 `switch` 语句，来捕获不同的错误条件。为此，我们将调用我们的 `BaseballTeam` 结构体的 `addPlayer()` 方法：

```swift
do {
    try myTeam.addPlayer(("David", "Ortiz", 34))
} catch PlayerNumberError.NumberTooHigh(let description) {
    print("Error: \(description)")
} catch PlayerNumberError.NumberTooLow(let description) {
    print("Error: \(description)")
} catch PlayerNumberError.NumberAlreadyAssigned {
    print("Error: Number already assigned")
}
```

在这个例子中，我们有三个`catch`语句。每个`catch`语句都有一个不同的模式来匹配；因此，它们将分别匹配不同的错误条件。如果我们回想一下，`PlayerNumberError.NumberToHigh`和`PlayerNumberError.NumberToLow`错误条件都有关联的值。要检索关联的值，我们可以在括号内使用`let`语句，就像示例中那样。

总是好的做法是将你的最后一个`catch`语句留空，这样它就能捕获之前`catch`语句中未匹配到的任何错误。因此，之前的例子应该这样重写：

```swift
do {
    try myTeam.addPlayer(("David", "Ortiz", 34))
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

我们也可以让错误传播出去而不是立即捕获它们。要做到这一点，我们只需要在函数定义中添加`throws`关键字。例如，在以下示例中，我们不是捕获错误，而是让它传播到调用函数的代码中，而不是在函数内部处理错误：

```swift
func myFunc() throws {
    try myTeam.addPlayer(("David", "Ortiz", 34))
}
```

如果我们确定不会抛出错误，我们可以使用强制-尝试表达式来调用函数，该表达式写作`try`!。强制-尝试表达式禁用了错误传播，并将函数调用包裹在一个运行时断言中，断言该调用不会抛出错误。如果抛出错误，我们将得到运行时错误，所以使用这个表达式时要非常小心。

当我在使用 Java 和 C#等语言处理异常时，我看到很多空的`catch`块。这就是我们需要捕获异常的地方，因为可能会抛出异常；然而，我们并不想对它做任何事情。在 Swift 中，代码看起来可能像这样：

```swift
do {
    let player = try myTeam.getPlayerByNumber(34)
    print("Player is \(player.firstName) \(player.lastName)")
} catch {}
```

看到这样的代码是我对异常处理不喜欢的事情之一。嗯，Swift 开发者对此有一个答案：`try?`关键字。`try?`关键字尝试执行可能会抛出错误的操作。如果操作成功，结果以可选的形式返回；然而，如果操作失败并抛出错误，操作返回 nil，并且错误被丢弃。

由于`try?`关键字的结果以可选的形式返回，我们通常希望使用可选绑定来使用这个关键字。我们可以将之前的例子重写如下：

```swift
if let player = try? myTeam.getPlayerByNumber(34) {
    print("Player is \(player.firstName) \(player.lastName)")
}
```

如我们所见，`try?`关键字使我们的代码更加简洁和易于阅读。

如果我们需要执行一些清理操作，无论是否有错误，我们都可以使用`defer`语句。我们使用`defer`语句在代码执行离开当前作用域之前执行一段代码。以下示例显示了如何使用`defer`语句：

```swift
func deferFunction()  {
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

如果我们调用此函数，控制台打印的第一行是—`函数开始`。代码的执行将跳过 `defer` 块，接下来控制台会打印 `Function finished`。最后，在离开函数的作用域之前，会执行 `defer` 块中的代码，我们会看到消息，`在 defer 块中`。以下是从该函数输出的内容：

```swift
Function started
Function finished
In defer block
str is Jon
```

`defer` 块将在执行离开当前作用域之前始终被调用，即使抛出错误。当我们需要在离开函数之前执行一些清理函数时，`defer` 块非常有用。

当我们想要确保执行所有必要的清理操作，即使抛出错误时，`defer` 语句非常有用。例如，如果我们成功打开一个文件进行写入，我们总是想确保关闭该文件，即使写入操作期间出现错误。然后我们可以将文件关闭功能放在 `defer` 块中，以确保文件在离开当前作用域之前始终关闭。

现在让我们看看如何使用 Swift 中的新可用性属性。

# 可用性属性

使用最新的 SDK 可以让我们访问为我们正在开发的平台提供的所有最新功能；然而，有时我们还想针对旧平台。Swift 允许我们使用可用性属性来安全地包装代码，以确保只有在正确的操作系统版本可用时才运行。可用性属性首次在 Swift 2 中引入。

可用性块基本上允许我们说，“如果我们正在运行指定的操作系统版本或更高版本，则运行此代码。否则，运行其他代码。”我们可以使用 `availability` 属性的两种方式。第一种方式允许我们执行特定的代码块，并且可以与 `if` 或 `guard` 语句一起使用。第二种方式允许我们将方法或类型标记为仅在特定平台上可用。

`availability` 属性接受最多五个以逗号分隔的参数，允许我们定义执行我们的代码所需的操作系统或应用程序扩展的最小版本。这些参数是：

+   `iOS`: 这是与我们的代码兼容的最小 iOS 版本

+   `OSX`: 这是与我们的代码兼容的最小 OS X 版本

+   `watchOS`: 这是与我们的代码兼容的最小 watchOS 版本

+   `iOSApplicationExtension`: 这是与我们的代码兼容的最小 iOS 应用程序扩展

+   `OSXApplicationExtension`: 这是与我们的代码兼容的最小 OS X 应用程序扩展

在参数之后，我们指定所需的最低版本。我们只需要包含与我们的代码兼容的参数。例如，如果我们正在编写 iOS 应用程序，我们只需要在`available`属性中包含`iOS`参数。我们用`*`（星号）结束参数列表。让我们看看我们如何仅在我们满足最低要求时执行特定的代码块：

```swift
if #available(iOS 9.0, OSX 10.10, watchOS 2, *) {
     // Available for iOS 9, OSX 10.10, watchOS 2 or above
    print("Minimum requirements met")
} else {
    //  Block on anything below the above minimum requirements
    print("Minimum requirements not met")
}
```

在这个示例中，`if #available(iOS 9.0, OSX 10.10, watchOS 2, *)`这一行代码阻止了当应用程序在不符合指定最低操作系统版本的系统上运行时执行代码块。在这个示例中，我们还使用了`else`语句，如果操作系统未满足最低要求，将执行一个单独的代码块。

我们还可以限制对函数或类型的访问。在前面的代码中，`available`属性以`#`（井号）字符为前缀。要限制对函数或类型的访问，我们用`@`（在号）字符作为`available`属性的前缀。以下示例展示了我们如何限制对类型和函数的访问：

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

在上一个示例中，我们指定了只有在代码在具有 iOS 9 或更高版本的设备上运行时，才能访问`testAvailability()`函数和`testStruct()`类型。为了使用`@available`属性来阻止对函数或类型的访问，我们必须使用`#available`属性将调用该函数或类型的代码包裹起来。以下示例展示了我们如何调用`testAvailability()`函数：

```swift
if #available(iOS 9.0, *) {
    testAvailability()
} else {
     // Fallback on earlier versions
}
```

在这个示例中，只有当应用程序在具有 iOS 9 或更高版本的设备上运行时，才会调用`testAvailability()`函数。

# 摘要

在本章中，我们探讨了 Swift 2 中添加的新错误处理功能和`availability`属性。这两个功能可以帮助我们编写更安全的代码，并使我们的应用程序更加稳定。

Swift 2 的错误处理功能显著改变了 Swift 程序员处理错误的方式。虽然我们不需要在我们的自定义类型中使用这个新功能，但它确实为我们提供了一种统一的方式来处理和响应错误。苹果公司也开始在 Cocoa 和 Cocoa Touch 框架中使用这种错误处理。

新的`availability`属性允许我们开发能够利用目标操作系统的最新功能的应用程序，同时仍然允许我们的应用程序在旧版本上运行。

在下一章中，我们将探讨如何创建和解析 XML 和 JSON 文档。
