# 7

# 函数

当我最初用 BASIC 学习编程时，我的前几个程序都是用一大块代码编写的。我很快意识到我一直在重复相同的代码。我想一定有更好的方法来做这件事，也就是我学习子程序和函数的时候。函数是编写良好代码时需要理解的关键概念之一。

在本章中，我们将介绍以下主题：

+   什么是函数？

+   如何从函数返回值

+   如何在函数中使用参数

+   什么是可变参数？

+   什么是`inout`参数？

在 Swift 中，函数是一个执行特定任务的独立代码块。函数通常用于将我们的代码逻辑地分解成可重用的命名块。函数的名称用于调用函数。

当我们定义一个函数时，我们还可以选择性地定义一个或多个参数。参数是通过调用它的代码传递给函数的命名值。这些参数通常在函数内部用于执行函数的任务。我们还可以为参数定义默认值，以简化函数的调用方式。

每个 Swift 函数都有一个与其关联的类型。这个类型被称为返回类型，它定义了从函数返回到调用它的代码的数据类型。如果一个函数没有返回值，则返回类型为`Void`。

让我们看看如何在 Swift 中定义函数。

# 使用单参数函数

在 Swift 中定义函数的语法非常灵活。这种灵活性使得我们能够轻松地定义简单的 C 风格函数，或者更复杂的具有局部和外部参数名称的函数，我们将在本章后面看到。让我们看看一些定义函数的例子。以下示例接受一个参数，但不向调用它的代码返回任何值：

```swift
func sayHello(name: String) ->    Void { 
    let retString = "Hello " + name
    print(retString)
} 
```

在前面的例子中，我们定义了一个名为`sayHello()`的函数，它接受一个名为`name`的参数。在函数内部，我们打印了一条问候信息给这个人。一旦函数内部的代码执行完毕，函数就会退出，控制权返回到调用它的代码。我们不仅可以将问候信息打印出来，还可以通过添加返回类型将其返回给调用它的代码，如下所示：

```swift
func sayHello2(name: String) ->String { 
    let retString = "Hello " + name 
    return retString
} 
```

`->`字符串定义了与函数关联的返回类型为字符串。这意味着该函数必须返回一个`String`类型的实例给调用它的代码。在函数内部，我们构建了一个名为`retString`的字符串常量，其中包含问候信息，然后使用`return`语句返回它。

调用 Swift 函数的过程与其他语言（如 C 或 Java）中调用函数或方法的过程类似。以下示例展示了如何调用`sayHello(name:)`函数，该函数将问候信息打印到屏幕上：

```swift
sayHello(name:"Jon") 
```

现在，让我们看看如何调用`sayHello2(name:)`函数，该函数将值返回给调用它的代码：

```swift
var message = sayHello2(name:"Jon")
print(message) 
```

在前面的例子中，我们调用了`sayHello2(name:)`函数，并将返回值赋给了`message`变量。如果一个函数定义了返回类型，就像`sayHello2(name:)`函数那样，它必须向调用它的代码返回该类型的一个值。因此，函数中每个可能的条件路径都必须以返回指定类型的值结束。这并不意味着调用函数的代码必须检索返回的值。以下是一个示例，以下两个语句都是有效的：

```swift
sayHello2(name:"Jon")
var message = sayHello2(name:"Jon") 
```

如果你没有指定变量来存储返回值，该值将被丢弃。当代码编译时，如果你没有将函数返回的值放入变量或常量中，你会收到一个警告。你可以通过使用下划线来避免这个警告，如下面的示例所示：

```swift
_ = sayHello2(name:"Jon") 
```

下划线告诉编译器你已经知道返回值，但不想使用它。在声明函数时使用`@discardableResult`属性也会消除警告。该属性的使用方法如下：

```swift
@discardableResult func sayHello2(name: String) ->String { 
    let retString = "Hello " + name
    return retString
} 
```

在 Swift 5.1 的 SE-0255 中，我们可以省略单表达式函数中的`return`语句。以下代码将展示这种情况：

```swift
func sayHello4(name: String) -> String {
    "Hello " + name
} 
```

这个函数与之前的 hello 函数定义类似，具有`String`返回类型；然而，你会注意到函数中没有返回语句。根据 SE-0255，如果我们有一个像`sayHello4(name:)`函数这样的单表达式函数，表达式的值可以在不需要`return`语句的情况下返回。如果我们像这样调用函数：

```swift
Let message = sayHello4(name:"Kara") 
```

`message`常量将包含字符串`"Hello Kara"`。

让我们看看如何为我们的函数定义多个参数。

# 使用多参数函数

我们的函数不仅限于只有一个参数；我们还可以定义多个参数。要创建一个多参数函数，我们在括号中列出参数，并用逗号分隔参数定义。让我们看看如何在函数中定义多个参数：

```swift
func sayHello(name: String, greeting: String) { 
    print("\(greeting) \(name)")
} 
```

在前面的例子中，该函数接受两个参数：`name`和`greeting`。然后我们使用这两个参数在控制台打印问候语。

调用一个多参数函数与调用一个单参数函数略有不同。在调用多参数函数时，我们用逗号分隔参数。我们还需要为所有参数包含参数名称。以下示例显示了如何调用多参数函数：

```swift
sayHello(name:"Jon", greeting:"Bonjour") 
```

如果我们为函数的每个参数定义了默认值，我们就不需要为每个参数提供一个参数。让我们看看如何为我们的参数配置默认值。

# 定义参数的默认值

我们可以通过在函数定义中声明参数时使用等于运算符（`=`）来为任何参数定义默认值。以下示例展示了如何声明一个具有参数默认值的函数：

```swift
func sayHello(name: String, greeting: String = "Bonjour") { 
    print("\(greeting) \(name)")
} 
```

在函数声明中，我们定义了一个没有默认值的参数 `(name:String)` 和一个有默认值的参数 `(greeting: String = "Bonjour")`。当一个参数有默认值声明时，我们可以带或不带设置该参数的值来调用函数。以下示例展示了如何不带设置 `greeting` 参数来调用 `sayHello()` 函数，以及如何设置 `greeting` 参数来调用它：

```swift
sayHello(name:"Jon") 
sayHello(name:"Jon", greeting: "Hello") 
```

在 `sayHello(name:"Jon")` 这一行，函数将打印出消息 `Bonjour Jon`，因为它使用了 `greeting` 参数的默认值。在 `sayHello(name:"Jon", greeting: "Hello")` 这一行，函数将打印出消息 `Hello Jon`，因为我们已经覆盖了 `greeting` 参数的默认值。

我们可以通过使用参数名称来声明多个具有默认值的参数，并且只覆盖我们想要的那些。以下示例展示了我们如何通过在调用时覆盖其中一个默认值来实现这一点：

```swift
func sayHello(name: String = "Test", name2: String = "Kailey", greeting: String = "Bonjour") {
    print("\(greeting) \(name) and \(name2)")
}
sayHello(name:"Jon",greeting: "Hello") 
```

在前面的例子中，我们声明了一个有三个参数的函数，每个参数都有一个默认值。然后我们调用该函数，将 `name2` 参数保留为其默认值，同时覆盖了剩余两个参数的默认值。

前面的例子将打印出消息 `Hello Jon and Kailey`。

现在，让我们看看我们如何从函数中返回多个值。

# 从函数中返回多个值

从 Swift 函数中返回多个值有几种方法。其中最常见的一种是将值放入一个集合类型（数组或字典）中，然后返回该集合。

以下示例展示了如何从 Swift 函数中返回一个集合类型：

```swift
func getNames() -> [String] {
    var retArray = ["Jon", "Kailey", "Kara"]
    return retArray
}
var names = getNames() 
```

在前面的例子中，我们声明了 `getNames()` 函数，它没有参数，返回类型为 `[String]`。`[String]` 的返回类型指定了返回类型为字符串类型的数组。

在前面的例子中，我们的数组只能返回字符串类型。如果我们需要返回包含数字的字符串，我们可以返回一个 `Any` 类型的数组，然后使用类型转换来指定 `object` 类型。然而，这并不是我们应用程序的好设计，因为它容易出错。返回不同类型值的一个更好的方法是用 `tuple` 类型。

当我们从函数中返回一个元组时，建议我们使用命名元组，这样我们可以使用点语法来访问返回的值。以下示例展示了如何从函数中返回一个命名元组并访问返回的命名元组的值：

```swift
func getTeam() -> (team:String, wins:Int, percent:Double) { 
    let retTuple = ("Red Sox", 99, 0.611)
    return retTuple
}
var t = getTeam()
print("\(t.team) had \(t.wins) wins") 
```

在前面的例子中，我们定义了`getTeam()`函数，该函数返回一个包含三个值的命名元组：`String`、`Int`和`Double`。在函数内部，我们创建了将要返回的元组。请注意，我们不需要将将要返回的元组定义为命名元组，因为元组内的值类型与函数定义中的值类型相匹配。现在我们可以像调用其他函数一样调用这个函数，并使用点语法来访问返回的元组的值。在先前的例子中，代码将打印出以下行：

```swift
Red Sox had 99 wins 
```

在前面的章节中，我们从函数中返回了非`nil`值；然而，这并不总是我们需要我们的代码做的事情。如果我们需要从函数中返回一个`nil`值，会发生什么？以下代码将无效并引发`Nil is incompatible with return type String`异常：

```swift
func getName() ->String { 
    return nil
} 
```

此代码抛出异常，因为我们已将返回类型定义为字符串值，但我们尝试返回一个`nil`值。如果有理由返回`nil`，我们需要将返回类型定义为可选类型，以便调用它的代码知道该值可能是`nil`。要定义返回类型为可选类型，我们使用与定义变量为可选类型时相同的方式使用问号（`?`）。以下示例显示了如何定义可选返回类型：

```swift
func getName() ->String? { 
    return nil
} 
```

上述代码不会引发异常。

我们还可以将元组设置为可选类型，或者将元组内的任何值设置为可选类型。以下示例显示了如何将元组作为可选类型返回：

```swift
func getTeam2(id: Int) -> (team:String, wins:Int, percent:Double)? { 
    if id == 1 {
        return ("Red Sox", 99, 0.611)
    }
    return nil
} 
```

在以下示例中，我们可以返回一个元组，就像它在函数定义中定义的那样，或者`nil`；两种选择都是有效的。如果我们需要一个元组内的单个值是`nil`，我们需要在元组内添加一个可选类型。以下示例显示了如何在元组内返回`nil`值：

```swift
func getTeam() -> (team:String, wins:Int, percent:Double?) { 
    let retTuple: (String, Int, Double?) = ("Red Sox", 99, nil) 
    return retTuple
} 
```

在先前的例子中，我们将`percent`值设置为`Double`或`nil`。

现在，让我们看看我们如何为我们的函数添加外部参数名称。

# 添加外部参数名称

在本节的前几个例子中，我们以与在 C 代码中定义参数相同的方式定义了参数的名称和值类型。在 Swift 中，我们不受此语法的限制，因为我们还可以使用外部参数名称。

外部参数名称用于在调用函数时指示每个参数的目的。每个参数的外部参数名称需要与本地参数名称一起定义。外部参数名称添加在函数定义中的本地参数名称之前。外部和本地参数名称之间用空格分隔。

让我们看看如何使用外部参数名称。但在我们这样做之前，让我们回顾一下我们之前是如何定义函数的。在以下两个示例中，我们将定义一个没有外部参数名称的函数，然后使用外部参数名称重新定义它：

```swift
func winPercentage(team: String, wins: Int, loses: Int) -> Double{ 
    return Double(wins) / Double(wins + loses)
} 
```

在前面的例子中，`winPercentage()` 函数有三个参数。这些参数是 `team`、`wins` 和 `loses`。`team` 参数应该是 `String` 类型，而 `wins` 和 `loses` 参数应该是 `Int` 类型。以下行代码展示了如何调用 `winPercentage()` 函数：

```swift
var per = winPercentage(team: "Red Sox", wins: 99, loses: 63) 
```

现在，让我们使用外部参数名称定义相同的函数：

```swift
func winPercentage(baseballTeam team: String, withWins wins: Int, andLoses losses: Int) -> Double {
    return Double(wins) / Double(wins + losses)
} 
```

在前面的例子中，我们使用外部参数名称重新定义了 `winPercentage()` 函数。在这个重新定义中，我们有相同的三个参数：`team`、`wins` 和 `losses`。区别在于我们如何定义参数。当使用外部参数时，我们使用外部参数名称和局部参数名称（用空格分隔）来定义每个参数。

在前面的例子中，第一个参数的外部参数名称为 `baseballTeam`，内部参数名称为 `team`。

当我们使用外部参数名称调用函数时，需要在函数调用中包含外部参数名称。以下代码展示了如何调用此函数：

```swift
var per = winPercentage(baseballTeam:"Red Sox", withWins:99, andLoses:63) 
```

虽然使用外部参数名称需要更多的输入，但它确实使你的代码更容易阅读。在前面的例子中，很容易看出该函数正在寻找一个棒球队的名称，第二个参数是胜利次数，最后一个参数是失败次数。

# 使用可变参数

可变参数是指可以接受零个或多个指定类型的值的参数。在函数定义中，我们通过在参数类型名称后附加三个点（`...`）来定义可变参数。可变参数的值以指定类型的数组形式提供给函数。以下示例展示了我们如何使用函数的可变参数：

```swift
func sayHello(greeting: String, names: String...) { 
    for name in names {
        print("\(greeting) \(name)")
    }
} 
```

在前面的例子中，`sayHello()` 函数接受两个参数。第一个参数是 `String` 类型，用于指定要使用的问候语。第二个参数是 `String` 类型的可变参数，用于指定要发送问候语的人名。在函数内部，可变参数是一个包含指定类型的数组；因此，在我们的例子中，`names` 参数是一个 `String` 值的数组。在这个例子中，我们使用了一个 `for-in` 循环来访问 `names` 参数中的值。

以下行代码展示了如何使用可变参数调用 `sayHello()` 函数：

```swift
sayHello(greeting:"Hello", names: "Jon", "Kara") 
```

上一行代码将问候语打印到每个名字上，如下所示：

```swift
Hello Jon
Hello Kara 
```

现在，让我们看看 `inout` 参数是什么。

# 输入输出参数

如果我们想更改参数的值，并且希望这些更改在函数结束时仍然保持，我们需要将参数定义为`inout`参数。对`inout`参数所做的任何更改都会传递回函数调用中使用的变量。

使用`inout`参数时有两个需要注意的事项：这些参数不能有默认值，并且它们不能是可变参数。

让我们看看如何使用`inout`参数交换两个变量的值：

```swift
func reverse(first: inout String, second: inout String) { 
    let tmp = first
    first = second
    second = tmp
} 
```

此函数将接受两个参数，并交换函数调用中使用的变量的值。当我们调用函数时，我们在变量名前放置一个反引号（`&`），表示函数可以修改其值。以下示例显示了如何调用`reverse`函数：

```swift
var one = "One"
var two = "Two"
reverse(first: &one, second: &two)
print("one: \(one) two: \(two)") 
```

在前面的例子中，我们将变量`one`设置为`One`的值，将变量`two`设置为`Two`的值。然后我们使用`one`和`two`变量调用`reverse()`函数。一旦`reverse()`函数返回，名为`one`的变量将包含`Two`的值，而名为`two`的变量将包含`One`的值。

关于`inout`参数的两个注意事项：可变参数不能是`inout`参数，并且`inout`参数不能有默认值。

# 省略参数标签

本章中所有函数在将参数传递给函数时都使用了标签。如果我们不想使用标签，我们可以通过使用下划线来省略它们。以下示例说明了这一点：

```swift
func sayHello(_ name: String, greeting: String) { 
    print("\(greeting) \(name)")
} 
```

注意参数列表中`name`标签前的下划线。这表示在调用此函数时不应使用`name`标签。现在，我们能够不使用`name`标签来调用此函数：

```swift
sayHello("Jon", greeting: "Hi") 
```

此调用将输出`Hi Jon`。

现在，让我们将我们所学的内容综合起来，看看一个更复杂的例子。

# 将所有这些放在一起

为了巩固我们在本章学到的内容，让我们再看一个例子。对于这个例子，我们将创建一个函数来测试一个字符串值是否包含有效的 IPv4 地址。IPv4 地址是分配给使用**互联网协议**（**IP**）进行通信的计算机的地址。IP 地址由四个范围从`0-255`的数值组成，由点（句号）分隔。以下是一个有效 IP 地址的代码示例；即`10.0.1.250`：

```swift
func isValidIP(ipAddr: String?) ->Bool { 
    guard let ipAddr = ipAddr else {
        return false
    }
    let octets = ipAddr.split { $0 == "."}.map{String($0)} 
    guard octets.count == 4 else {
        return false
    }
    for octet in octets {
        guard validOctet(octet: octet) else { 
            return false
        }
    }
    return true
} 
```

在`isValidIp()`函数中，唯一的参数是可选类型，我们首先做的事情是验证`ipAddR`参数不是`nil`。为此，我们使用带有可选绑定的`guard`语句。如果可选绑定失败，我们返回一个布尔值`false`，因为`nil`不是一个有效的 IP 地址。

如果`ipAddr`参数包含非空值，我们将使用点作为分隔符将字符串拆分成一个字符串数组。由于 IP 地址应该包含由点分隔的四个数字，我们再次使用`guard`语句来检查数组是否包含四个元素。如果不包含，我们返回`false`，因为我们知道`ipAddr`参数没有包含有效的 IP 地址。

我们随后使用`String`类型的`split()`函数将字符串拆分成四个子字符串，其中每个子字符串包含地址的一个`octet`。这些子字符串存储在`octets`数组中。

然后，我们遍历通过在点处拆分原始的`ipAddr`参数并传递给`validOctet()`函数创建的数组中的值。如果所有四个值都通过`validOctet()`函数的验证，我们就有一个有效的 IP 地址，并返回一个布尔值`true`；然而，如果任何一个值未能通过`validOctet()`函数的验证，我们返回一个布尔值`false`。现在，让我们看看`validOctet()`函数的代码：

```swift
func validOctet(octet: String) ->Bool {
    guard let num = Int(octet),num >= 0 && num <256 else { 
        return false
    }
    return true
} 
```

`validOctet()`函数有一个名为`octet`的`String`参数。这个函数将验证`octet`参数是否包含介于`0`和`255`之间的数值；如果是，函数将返回一个布尔值`true`。否则，它将返回一个布尔值`false`。

# 摘要

在本章中，我们介绍了函数是什么以及如何使用它们。你将在你编写的每一个严肃的应用程序中使用函数。在下一章中，我们将探讨类和结构。类和结构可以包含函数，但这些函数被称为方法。
