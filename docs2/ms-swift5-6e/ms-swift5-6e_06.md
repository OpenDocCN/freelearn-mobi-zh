

# 控制流程

当我还是个青少年时，每个月在我用 Commodore Vic-20 学习 BASIC 编程的时候，我都会阅读几本早期的计算机杂志，比如 *Byte Magazine*。我记得有一篇关于一款名为 *Zork* 的游戏的评论。虽然 Zork 并不是为我的 Vic-20 可用的游戏，但这个游戏的概念深深吸引了我，因为我真的很喜欢科幻和奇幻。我记得在想，写一个那样的游戏会多么酷，所以我决定找出如何做到这一点。当时我必须掌握的最大概念之一就是如何根据用户的操作来控制应用程序的流程。

在本章中，我们将涵盖以下主题：

+   条件语句是什么以及如何使用它们

+   循环是什么以及如何使用它们

+   控制转移语句是什么以及如何使用它们

# 我们到目前为止学到了什么？

到目前为止，我们一直在为使用 Swift 编写应用程序打下基础。虽然我们可以用我们目前学到的东西编写一个非常基础的程序，但仅使用我们前五章所涵盖的内容来编写一个有用的应用程序将会很困难。

从本章开始，我们将开始从 Swift 语言的根基中走出来，开始学习使用 Swift 进行应用程序开发的构建块。在本章中，我们将讨论控制流程语句。要成为 Swift 编程语言的专家，理解本章中讨论的概念至关重要。

在我们讨论控制流程和函数之前，让我们看看在 Swift 中括号和括号是如何使用的。

# 括号

在 Swift 中，与其它 C 类语言不同，条件语句和循环需要使用括号。在其它 C 类语言中，如果条件语句或循环只有一个要执行的语句，那么围绕该行的括号是可选的。这导致了无数的错误和漏洞，比如苹果的 `goto fail` 漏洞。当苹果设计 Swift 时，他们决定引入括号的使用，即使只有一行代码要执行也是如此。让我们看看一些说明这一要求的代码示例。第一个示例在 Swift 中是无效的，因为它缺少括号；然而，它将在大多数其它语言中是有效的：

```swift
if (x > y) 
    x=0 
```

在 Swift 中，你需要像以下示例中那样使用括号：

```swift
if (x > y) { 
    x=0
} 
```

# 括号

与其它 C 类语言不同，Swift 中条件表达式的括号是可选的。在上面的示例中，我们围绕条件表达式使用了括号，但它们不是必需的。以下示例在 Swift 中是有效的，但在大多数 C 类语言中则不是：

```swift
if x > y { 
    x=0
} 
```

# 控制流程

控制流，也称为控制流程，指的是在应用程序中语句、指令和函数执行的顺序。Swift 支持大多数在 C 类语言中使用的熟悉控制流语句。

这些包括循环（如 `while`）、条件语句（包括 `if`、`switch` 和 `guard`）以及控制转移语句（包括 `break` 和 `continue`）。值得注意的是，Swift 不包括传统的 C `for` 循环，而不是传统的 `do-while` 循环，Swift 有 `repeat-while` 循环。

除了标准的 C 控制流语句外，Swift 还包括 `for-in` 循环等语句，并增强了一些现有语句，例如 `switch` 语句。

让我们从查看 Swift 中的条件语句开始。

# 条件语句

条件语句检查一个条件，并且只有当条件为 `true` 时才执行代码块。Swift 提供了 `if` 和 `if...else` 条件语句。让我们看看如何使用这些条件语句在指定的条件为 `true` 时执行代码块。

## `if` 语句

`if` 语句会检查一个条件语句，如果它是 `true`，它将执行代码块。这个语句采用以下格式：

```swift
if condition { 
    block of code
} 
```

现在，让我们看看如何使用 `if` 语句：

```swift
let teamOneScore = 7 
let teamTwoScore = 6
if teamOneScore > teamTwoScore { 
    print("Team One Won")
} 
```

在前面的例子中，我们首先设置了 `teamOneScore` 和 `teamTwoScore` 常量。然后我们使用 `if` 语句来检查 `teamOneScore` 的值是否大于 `teamTwoScore` 的值。如果值更大，我们将 `Team One Won` 打印到控制台。当运行此代码时，我们确实会看到 `Team One Won` 被打印到控制台，但如果 `teamTwoScore` 的值大于 `teamOneScore` 的值，则不会打印任何内容。这不是编写应用程序的最佳方式，因为我们希望用户知道哪个队伍实际上赢得了比赛。`if...else` 语句可以帮助我们解决这个问题。

### 使用 `if...else` 语句进行条件代码执行

`if...else` 语句检查一个条件语句，如果它是 `true`，它将执行一个代码块。如果条件语句不是 `true`，它将执行一个单独的代码块；这个语句采用以下格式：

```swift
if condition {
    block of code if true
} else {
    block of code if not true
} 
```

现在我们修改前面的例子，使用 `if...else` 语句来告诉用户哪个队伍赢得了比赛：

```swift
let teamOneScore = 7 
let teamTwoScore = 6
if teamOneScore > teamTwoScore{ 
    print("Team One Won")
} else {
    print("Team Two Won")
} 
```

这个新版本将打印出 `Team One Won`（队伍一获胜），如果 `teamOneScore` 的值大于 `teamTwoScore` 的值；否则，它将打印出消息，`Team Two Won`（队伍二获胜）。

这解决了我们代码中的一个问题，但如果你认为`teamOneScore`的值等于`teamTwoScore`的值时，代码会做什么呢？在现实世界中，我们会看到一场平局，但在前面的代码中，我们会打印出`Team Two Won`，这对第一队来说是不公平的。在这种情况下，我们可以使用多个`else if`语句和一个最后的`else`语句来作为没有满足任何条件时的默认路径。

这在以下代码示例中得到了说明：

```swift
let teamOneScore = 7 
let teamTwoScore = 6
if teamOneScore > teamTwoScore { 
    print("Team One Won")
} else if teamTwoScore > teamOneScore { 
    print("Team Two Won")
} else {
    print("We have a tie")
} 
```

在前面的代码中，如果`teamOneScore`的值大于`teamTwoScore`的值，我们将`Team One Won`打印到控制台。然后我们有一个`else if`语句，这意味着只有当第一个`if`语句返回`false`时，才会检查条件语句。最后，如果两个`if`语句都返回`false`，则调用`else`块中的代码，并将`We have a tie`打印到控制台。

现在是指出这一点的好时机，即像前一个例子中那样堆叠多个`else if`语句并不是一个好的实践。更好的做法是使用`switch`语句，我们将在本章后面探讨这一点。

## 守卫语句

在 Swift 以及大多数现代语言中，我们的条件语句往往侧重于测试一个条件是否为`true`。例如，以下代码测试`x`变量是否大于`10`，如果是，则执行某种功能。如果条件为`false`，我们处理以下错误条件：

```swift
var x = 9 
if x > 10 {
// Functional code here
} else {
// Do error condition
} 
```

这种类型的代码将我们的功能代码嵌入到我们的检查中，并将错误条件藏在函数的末尾，但如果我们真的不希望这样呢？有时（实际上很多次），在函数的开始处处理错误条件可能更好。在我们的简单例子中，我们可以轻松检查`x`是否小于或等于`10`，如果是，则执行错误条件。并不是所有的条件语句都那么容易重写，特别是像可选绑定这样的项目。

在 Swift 中，我们有`guard`语句。这个语句专注于在条件为`false`时执行一个功能；这允许我们在函数的早期捕获错误并执行错误条件。我们可以使用`guard`语句重写先前的例子，如下所示：

```swift
var x = 9
guard x > 10 else {
    // Do error condition 
    return
}
//Functional code here 
```

在这个新例子中，我们检查`x`变量是否大于`10`，如果不是，我们执行错误条件。如果变量大于`10`，应用程序将继续执行我们代码的功能部分。你会注意到我们在守卫条件中嵌入了一个`return`语句。守卫语句中的代码必须包含一个控制转移语句；这就是防止其余代码执行的原因。如果我们忘记了控制转移语句，Swift 会在编译时显示错误。我们将在本章稍后讨论控制转移语句。

让我们看看`guard`语句的一些更多示例。以下示例展示了我们如何使用`guard`语句来验证一个可选值是否包含有效值：

```swift
func guardFunction(str: String?) { 
    guard let goodStr = str else { 
        print("Input was nil")
        return
}
    print("Input was \(goodStr)")
} 
```

函数尚未介绍，但将在*第七章*，*函数*中介绍。

在这个例子中，我们创建了一个名为`guardFunction()`的函数，它接受一个包含字符串或`nil`值的可选值。然后我们使用带有可选绑定的`guard`语句来验证字符串可选值不是`nil`。如果它包含`nil`，则执行`guard`语句内的代码，并使用`return`语句退出函数。使用带有可选绑定的`guard`语句的好处是，新变量在函数的其余部分的作用域内，而不仅仅是可选绑定语句的作用域内。

条件语句会检查一次条件，如果条件满足，则执行代码块。然而，如果我们想连续执行代码块直到满足条件呢？为此，我们使用循环语句。

## `switch` 语句

`switch`语句获取一个值，将其与几个可能的匹配项进行比较，并根据第一个成功的匹配执行相应的代码块。`switch`语句是当存在多个可能的匹配项时，使用多个`else if`语句的替代方案。`switch`语句的格式如下：

```swift
switch value { 
    case match1:
        block of code 
    case match2:
        block of code
        //as many cases as needed 
    default:
        block of code
} 
```

与大多数其他语言不同，在 Swift 中，`switch`语句不会自动跳转到下一个`case`语句；因此，我们不需要使用`break`语句来防止这种情况。这是 Swift 中内置的另一个安全特性，因为初学者在处理`switch`语句时最常见的编程错误之一就是忘记在`case`语句的末尾添加`break`语句。让我们看看如何使用`switch`语句：

```swift
var speed = 300000000
switch speed {
    case 300000000:
        print("Speed of light") 
    case 340:
        print("Speed of sound") 
    default:
        print("Unknown speed")
} 
```

在前面的例子中，`switch`语句获取了`speed`变量的值，并将其与两个`case`语句进行比较。如果`speed`的值与任一`case`匹配，则代码会打印速度。如果没有找到匹配项，它会打印`Unknown speed`消息。

每个`switch`语句都必须匹配所有可能的值。这意味着，除非我们正在匹配一个具有定义数量的值的枚举，否则每个`switch`语句都必须有一个`default`情况。让我们看看没有`default`情况的例子：

```swift
var num = 5 
switch num {
    case 1 :
        print("number is one") 
    case 2 :
        print("Number is two") 
    case 3 :
        print("Number is three")
} 
```

如果我们将前面的代码放入 Playground 并尝试编译，我们将收到一个`Switch must be exhaustive`错误。这是一个编译时错误，因此，我们只有在尝试编译代码时才会收到通知。

在单个`case`中可以包含多个项目。为了做到这一点，我们需要用逗号分隔这些项目。让我们看看我们是如何使用`switch`语句来判断一个字符是元音还是辅音的：

```swift
var char : Character = "e" 
switch char {
    case "a", "e", "i", "o", "u": 
        print("letter is a vowel")
    case "b", "c", "d", "f", "g", "h", "j", "k", "l", "m", "n", "p", "q", "r", "s", "t", "v", "w","x", "y", "z":
        print("letter is a consonant") 
    default:
        print("unknown letter")
} 
```

在前面的示例中，我们可以看到每个 `case` 都有多个项。逗号分隔这些项，`switch` 语句尝试将 `char` 变量与 `case` 语句中列出的每个项进行匹配。

还有可能检查 `switch` 语句的值，看它是否包含在某个范围内。为此，我们在 `case` 语句中使用范围运算符之一，如下所示：

```swift
var grade = 93
switch grade {
    case 90...100: print("Grade is an A")
    case 80...89: print("Grade is a B")
    case 70...79: print("Grade is an C")
    case 60...69: print("Grade is a D")
    case 0...59: print("Grade is a F")
    default:
        print("Unknown Grade")
} 
```

在前面的示例中，`switch` 语句取了 `grade` 变量，将其与每个 `case` 语句中的范围进行比较，并打印出适当的等级。

在 Swift 中，任何 `case` 语句都可以包含一个可选的 `where` 子句，它提供了一个需要验证的额外条件。假设在我们前面的例子中，我们有在课堂上接受特殊帮助的学生，我们想要为他们定义一个从 55 到 69 的 `D` 等级。以下示例展示了我们如何做到这一点：

```swift
var studentId = 4 
var grade = 57 
switch grade {
    case 90...100: 
        print("Grade is an A")
    case 80...89: 
        print("Grade is a B")
    case 70...79: 
        print("Grade is an C")
    case 55...69 where studentId == 4: 
        print("Grade is a D for student 4")
    case 60...69: 
        print("Grade is a D")
    case 0...59: 
        print("Grade is a F")
    default:
        print("Unknown Grade")
} 
```

在使用 `where` 表达式时，需要注意的一点是 Swift 将尝试匹配值，从第一个 `case` 语句开始，逐个检查每个 `case` 语句。这意味着，如果我们把带有 `where` 表达式的 `case` 语句放在 `F` 等级 `case` 语句之后，那么带有 `where` 表达式的 `case` 语句将永远不会被触及。这在下述示例中得到了说明：

```swift
var studentId = 4 
var grade = 57 
switch grade {
    case 90...100: 
        print("Grade is an A")
    case 80...89: 
        print("Grade is a B")
    case 70...79: 
        print("Grade is an C")
    case 60...69: 
        print("Grade is a D")
    case 0...59: 
        print("Grade is a F")
        //The following case statement would never be reached because
        //the grades would always match one of the previous two 
    case 55...69 where studentId == 4:
        print("Grade is a D for student 4")
    default:
        print("Unknown Grade")
} 
```

如果你使用 `where` 子句，一个很好的经验法则是始终将带有 `where` 子句的 `case` 语句放在任何不带 `where` 子句的类似 `case` 语句之前。

`switch` 语句在评估枚举时也非常有用。由于枚举具有有限数量的值，如果我们为枚举中的所有值提供 `case` 语句，我们就不需要提供 `default` 语句。以下示例演示了我们可以如何使用 `switch` 语句来评估枚举：

```swift
enum Product {
    case Book(String, Double, Int) 
    case Puzzle(String, Double)
}
var order = Product.Book("Mastering Swift 4", 49.99, 2017) 
switch order {
    case .Book(let name, let price, let year):
        print("You ordered the book \(name): \(year) for \(price)") 
    case .Puzzle(let name, let price):
        print("You ordered the Puzzle \(name) for \(price)")
} 
```

在这个例子中，我们首先定义了一个名为 `Product` 的枚举，包含两个值，每个值都有关联的值。然后我们创建了一个 `order` 变量，其类型为 `Product`，并使用 `switch` 语句对其进行评估。

当使用枚举的 `switch` 语句时，我们必须为所有可能的值提供一个 `case` 语句或一个 `default` 语句。让我们看看一些额外的代码，以说明这一点：

```swift
enum Planets {
    case Mercury, Venus, Earth, Mars, Jupiter 
    case Saturn, Uranus, Neptune
}
var planetWeLiveOn = Planets.Earth
// Using the switch statement 
switch planetWeLiveOn {
    case .Mercury:
        print("We live on Mercury, it is very hot!") 
    case .Venus:
        print("We live on Venus, it is very hot!") 
    case .Earth:
        print("We live on Earth, just right") 
    case .Mars:
        print("We live on Mars, a little cold") 
    case .Jupiter, .Saturn, .Uranus, .Neptune:
        print("Where do we live?")
} 
```

在此示例代码中，我们有一个处理 `Planets` 枚举中每个行星的 `case` 语句。我们还可以添加一个 `default` 语句来处理将来可能添加的任何额外行星。然而，建议如果 `switch` 语句使用带有枚举的 `default` 语句，则应使用 `@unknown` 属性，如下所示：

```swift
switch planetWeLiveOn { 
    case .Mercury:
        print("We live on Mercury, it is very hot!") 
    case .Venus:
        print("We live on Venus, it is very hot!") 
    case .Earth:
        print("We live on Earth, just right") 
    case .Mars:
        print("We live on Mars, a little cold") 
    case .Jupiter, .Saturn, .Uranus, .Neptune:
        print("Where do we live?") 
    @unknown default:
        print("Unknown planet")
} 
```

这将始终抛出一个警告，提醒我们如果我们在 `Planet` 枚举中添加一个新的行星，那么我们需要在这个代码部分处理这个新行星。

在 *第五章*，*使用 Swift 集合* 中，我们简要介绍了 Diff 算法。当时我们提到，直到我们理解了 `switch` 语句，我们才看不到这个算法的强大之处。现在我们理解了 `switch` 语句，让我们看看我们可以用这个算法做什么：

```swift
var cities1 = ["London", "Paris", "Seattle", "Boston", "Moscow"]
var cities2 = ["London", "Paris", "Tulsa", "Boston", "Tokyo"]
let diff = cities2.difference(from: cities1)
for change in diff {
    switch change {
    case .remove(let offset, let element, _ ):
        cities2.remove(at: offset)
    case .insert(let offset, let element, _):
        cities2.insert(element, at: offset)
} 
```

在前面的代码中，我们首先创建了两个包含城市列表的字符串数组。然后我们运行了 `difference(from:)` 方法，我们在上一章中介绍了这个方法。`difference(from:)` 方法返回一个包含 `Change` 枚举实例的集合。`Change` 枚举的定义如下：

```swift
public enum Change {
    case insert(offset: Int, element: ChangeElement, associatedWith: Int?)
    case remove(offset: Int, element: ChangeElement, associatedWith: Int?)
} 
```

这个枚举包含两个可能的值。`insert` 值告诉我们是否需要插入一个值，因为数组缺少来自另一个数组的一个元素。`remove` 值告诉我们需要删除一个元素，因为数组有一个不在另一个数组中的元素。在我们的代码中，我们处理了 `insert` 和 `remove` 两个值，这意味着代码执行后，`cities2` 数组将具有与 `cities1` 数组相同的元素和相同的顺序。这并不太令人兴奋，但让我们假设我们想要从 `cities2` 数组中删除任何不在 `cities1` 数组中出现的元素。我们可以将我们的代码更改为以下内容：

```swift
var cities1 = ["London", "Paris", "Seattle", "Boston", "Moscow"]
var cities2 = ["London", "Paris", "Tulsa", "Boston", "Tokyo"]
for change in diff {
    switch change {
    case .remove(let offset, let element, _ ):
        cities2.remove(at: offset)
    default:
        break
    }
} 
```

当此代码执行时，`cities2` 数组将包含三个元素：`London`、`Paris` 和 `Boston`。如果我们只处理插入条件，那么 `cities2` 数组将包含任何属于任一数组的元素。切换元组

我们还可以在 `case` 语句中使用下划线（通配符）和范围运算符与元组结合；让我们看看如何做到这一点：

```swift
let myDog = ("Maple", 4)
switch myDog {
    case ("Lily", let age):
        print("Lily is my dog and is \(age)") 
    case ("Maple", let age):
        print("Maple is my dog and is \(age)") 
    case ("Dash", let age):
        print("Dash is my dog and is \(age)") 
    default:
        print("unknown dog")
} 
```

在此代码中，我们创建了一个名为 `myDog` 的元组，其中包含了我狗的名字和她的年龄。然后我们使用 `switch` 语句来匹配名字（元组的第一个元素）和 `let` 语句来检索年龄。在这个例子中，信息“Maple 是我的狗，她 4 岁”将被打印到屏幕上。

我们还可以在 `case` 语句中使用下划线（通配符）和范围运算符与元组结合，如下面的示例所示：

```swift
switch myDog { 
    case(_, 0...1): 
        print("Your dog is a puppy") 
    case(_, 2...7): 
        print("Your dog is middle aged") 
    case(_, 8...): 
        print("Your dog is getting old") 
    default: 
        print("Unknown") 
} 
```

在这个例子中，下划线将匹配任何名字，而范围运算符将查找狗的年龄。在这个例子中，由于 Maple 四岁了，屏幕上会打印出信息“你的狗处于中年”。

### 匹配通配符

在 Swift 中，我们还可以将下划线（通配符）与 `where` 语句结合使用。以下示例说明了这一点：

```swift
let myNumber = 10 
switch myNumber {
    case _ where myNumber.isMultiple(of: 2): 
        print("Multiple of 2")
    case _ where myNumber.isMultiple(of: 3): 
        print("Multiple of 3")
    default:
        print("No Match")
} 
```

在这个例子中，我们创建了一个名为 `myNumber` 的整数变量，并使用 `switch` 语句来确定变量的值是否是 2 或 3 的倍数。注意，`case` 语句前面有一个下划线，后面跟着 `where` 语句。下划线将匹配变量的所有值，然后调用 `where` 语句来查看我们正在切换的项是否匹配其中定义的规则。

# 循环

循环语句使我们能够连续执行一段代码，直到满足某个条件。它们还使我们能够遍历集合中的元素。让我们看看我们如何使用`for-in`循环来遍历集合的元素。

## for-in 循环

虽然 Swift 没有提供基于 C 的标准`for`循环，但它确实有`for-in`循环。在 Swift 3 中，基于 C 的标准`for`循环被从 Swift 语言中移除，因为它很少使用。你可以在 Swift 进化网站上阅读移除此循环的完整提案：[`github.com/apple/swift-evolution/blob/master/proposals/0007-remove-c-style-for-loops.md`](https://github.com/apple/swift-evolution/blob/master/proposals/0007-remove-c-style-for-loops.md)。`for-in`语句用于对范围、集合或序列中的每个项执行代码块。

### 使用 for-in 循环

`for-in`循环遍历一个项的集合或数字的范围，并对集合或范围内的每个项执行一段代码。`for-in`语句的格式如下：

```swift
for variable in collection/range { 
    block of code
} 
```

如前述代码所示，`for-in`循环有两个：

+   `variable`：这个变量会在每次循环执行时改变，并将持有集合或范围中的当前项

+   `collection/range`：这是要遍历的集合或范围

让我们看看如何使用`for-in`循环遍历一个数字的范围：

```swift
for index in 1...5 { 
    print(index)
} 
```

在前面的例子中，我们遍历了从 1 到 5 的数字范围，并将每个数字打印到控制台。这个循环使用了闭包范围运算符（`...`）来为循环提供一个遍历的范围。Swift 还提供了半开范围运算符（`..>`）和我们在上一章中看到的单侧范围运算符。

现在，让我们看看如何使用`for-in`循环遍历一个数组：

```swift
var countries = ["USA","UK", "IN"] 
for item in countries {
    print(item)
} 
```

在前面的例子中，我们遍历了`countries`数组，并将数组中的每个元素打印到控制台。正如你所看到的，使用`for-in`循环遍历数组比使用基于 C 的标准`for`循环更安全、更简洁，也更简单。使用`for-in`循环可以防止我们犯一些常见的错误，例如在条件语句中使用小于等于（`<=`）运算符而不是小于（`<`）运算符。

让我们看看如何使用`for-in`循环遍历一个字典：

```swift
var dic = ["USA": "United States", "UK": "United Kingdom","IN":"India"]
for (abbr, name) in dic { 
    print("\(abbr) --\(name)")
} 
```

在前面的例子中，我们使用了`for-in`循环遍历字典中的每个键值对。在这个例子中，字典中的每个项都作为（`key`,`value`）元组返回。我们可以在循环体中将（`key`,`value`）元组的成员分解为命名的常量。需要注意的是，由于字典不保证存储项的顺序，遍历的顺序可能与插入的顺序不同。

现在，让我们看看另一种类型的循环，即`while`循环。

## 当循环

`while`循环执行一个代码块，直到满足某个条件。Swift 提供了两种形式的`while`循环；这些是`while`循环和`repeat-while`循环。在 Swift 2.0 中，Apple 用`repeat-while`循环替换了`do-while`循环。`repeat-while`循环的功能与`do-while`循环相同。Swift 使用`do`语句进行错误处理。当你想运行零次或多次循环时，使用`while`循环；当你想运行一次或多次循环时，使用`repeat-while`循环。

我们使用`while`循环当要执行的迭代次数未知且通常依赖于某些业务逻辑时。这可能是像遍历一个集合直到找到一个特定的值或满足某个条件。

### 使用 while 循环

`while`循环首先评估一个条件语句，然后当条件语句为`true`时，反复执行一个代码块。`while`语句的格式如下：

```swift
while condition { 
    block of code
} 
```

让我们看看如何使用`while`循环。在下面的例子中，`while`循环将在生成的随机数小于 7 时继续执行代码块。在这个例子中，我们使用`Int.random()`函数生成一个介于 0 到 9 之间的随机数：

```swift
var ran = 0 while ran < 7 {
    ran = Int.random(in: 1..<20)
} 
```

在前面的例子中，我们首先将`ran`变量初始化为`0`。然后`while`循环检查这个变量，如果值小于 7，就生成一个新的介于 0 到 19 之间的随机数。`while`循环将继续循环，直到生成的随机数等于或大于 7。一旦生成的随机数等于或大于 7，循环将退出。

在前面的例子中，`while`循环在生成新的随机数之前检查了条件语句。但如果我们不想在生成随机数之前检查条件语句呢？我们可以在初始化变量时生成一个随机数，但这意味着我们需要复制生成随机数的代码，而复制代码永远不是一种理想解决方案。使用`repeat-while`循环更为可取。

### 使用 repeat-while 循环

`while`循环和`repeat-while`循环的区别在于，`while`循环在第一次执行代码块之前会检查条件语句；因此，所有在条件语句中的变量都需要在执行`while`循环之前初始化。

`repeat-while`循环会在第一次检查条件语句之前运行循环块。这意味着我们可以在代码的条件块中初始化变量。当条件语句依赖于循环块中的代码时，`repeat-while`循环的使用更为合适。`repeat-while`循环的格式如下：

```swift
repeat {
    block of code
} while condition 
```

让我们通过创建一个`repeat-while`循环来查看这个具体的例子，在这个循环中，我们在循环块内初始化我们要检查的变量：

```swift
var ran: Int repeat {
    ran = Int.random(in: 1..>20)
} while ran < 4 
```

在前面的例子中，我们将`ran`变量定义为整数；然而，我们直到进入循环块并生成随机数之前都没有初始化它。如果我们尝试使用`while`循环（未初始化`ran`变量），我们将收到`variable used before being initialized`异常。

之前，我们提到`switch`语句比使用多个`else if`块更受欢迎。让我们看看我们如何使用`switch`语句。

# 在条件语句和循环中使用`case`和`where`语句

正如我们在`switch`语句中看到的那样，`switch`语句内的`case`和`where`语句可以非常强大。在我们的条件语句中使用`case`和`where`语句也可以使我们的代码更加简洁且易于阅读。条件语句和循环，如`if`、`for`和`while`，也可以使用`where`和`case`关键字。让我们看看一些例子，从使用`where`语句在`for-in`循环中过滤结果开始。

## 使用`where`语句进行过滤

在这个例子中，我们取一个整数数组，并仅打印出 3 的倍数。然而，在我们查看如何使用`where`语句过滤结果之前，让我们看看如何在不使用`where`语句的情况下完成此操作：

```swift
for number in 1...30 { 
    if number % 3 == 0 {
        print(number)
    }
} 
```

在这个例子中，我们使用`for-in`循环遍历数字 1 到 30。在`for-in`循环内，我们使用`if`条件语句过滤出 3 的倍数。在这个简单的例子中，代码相对容易阅读，但让我们看看如何使用`where`语句来减少代码行数并使其更容易阅读：

```swift
for number in 1...30 where number % 3 == 0 { 
    print(number)
} 
```

我们仍然有与上一个例子相同的`for-in`循环。然而，我们现在将`where`语句放在了末尾；因此，我们只遍历 3 的倍数。使用`where`语句将我们的例子缩短了两行，并使其更容易阅读，因为`where`子句与`for-in`循环在同一行，而不是嵌入在循环本身中。

现在，让我们看看如何使用`for-case`语句进行过滤。

## 使用`for-case`语句进行过滤

在接下来的例子中，我们将使用`for-case`语句过滤一个元组数组，并仅打印出符合我们标准的结果。`for-case`示例与使用`where`语句非常相似，它旨在消除在循环中过滤结果时需要`if`语句的需求。在这个例子中，我们将使用`for-case`语句过滤一系列世界系列冠军，并打印出特定球队赢得世界系列的时间：

```swift
var worldSeriesWinners = [ 
    ("Red Sox", 2004),
    ("White Sox", 2005),
    ("Cardinals", 2006),
    ("Red Sox", 2007),
    ("Phillies", 2008),
    ("Yankees", 2009),
    ("Giants", 2010),
    ("Cardinals", 2011),
    ("Giants", 2012),
    ("Red Sox", 2013),
    ("Giants", 2014),
    ("Royals", 2015)
]
for case let ("Red Sox", year) in worldSeriesWinners { 
    print(year)
} 
```

在这个例子中，我们创建了一个名为`worldSeriesWinners`的元组数组，数组中的每个元组包含球队名称和赢得世界大赛的年份。然后我们使用`for-case`子句遍历数组，并仅打印出红袜队赢得世界大赛的年份。过滤是在`case`子句中完成的，其中`("Red Sox", year)`表示我们想要所有第一个元素包含`Red Sox`字符串的结果，以及`year`常量的第二个元素。然后`for-in`循环遍历`case`子句的结果，打印出`year`常量的值。

`for-case-in`子句也使得在可选数组中过滤掉`nil`值变得非常容易；让我们看看这个例子：

```swift
let myNumbers: [Int?] = [1, 2, nil, 4, 5, nil, 6]
for case let .some(num) in myNumbers { 
    print(num)
} 
```

在这个例子中，我们创建了一个名为`myNumbers`的可选数组，它可以包含整数或`nil`。正如我们在*第四章*，*可选类型*中看到的，可选在内部定义为枚举，如下面的代码所示：

```swift
enum Optional < Wrapped > { 
    case none,
    case some(Wrapped)
} 
```

如果可选设置为`nil`，它将具有`none`的值，但如果它不是`nil`，它将具有`some`的值，并关联实际值的类型。在我们的例子中，当我们过滤`.some(num)`时，我们正在寻找任何具有非空值的可选。作为`.some()`的简写，我们可以使用问号（`?`）符号，如下面的例子所示。此示例还结合了`for-case-in`子句和`where`子句以执行额外的过滤：

```swift
let myNumbers: [Int?] = [1, 2, nil, 4, 5, nil, 6]
for case let num? in myNumbers where num < 3 { 
    print(num)
} 
```

这个例子与上一个例子相同，只是我们在`where`子句中添加了额外的过滤。在上一个例子中，我们遍历了所有非空值；然而，在这个例子中，我们遍历了大于 3 的非空值。让我们看看我们如何在不使用`case`或`where`子句的情况下进行相同的过滤：

```swift
let myNumbers: [Int?] = [1, 2, nil, 4, 5, nil, 6]
for num in myNumbers {
    if let num = num {
        if num < 3 {
            print(num)
        }
    }
} 
```

使用`for-case-in`和`where`子句可以大大减少所需的行数。它还使我们的代码更容易阅读，因为所有过滤子句都在同一行。

让我们再看看另一个过滤示例。这次，我们将查看`if-case`子句。

## 使用`if-case`子句

使用`if-case`子句与使用`switch`子句非常相似。大多数时候，当我们有超过两个要匹配的情况时，我们更喜欢使用`switch`子句，但有时需要使用`if-case`子句。其中一种情况是我们只寻找一两个可能的匹配项，并且不想处理所有可能的匹配项；让我们看看这个例子：

```swift
enum Identifier { 
    case Name(String) 
    case Number(Int) 
    case NoIdentifier
}
var playerIdentifier = Identifier.Number(2) 
if case let .Number(num) = playerIdentifier {
    print("Player's number is \(num)")
} 
```

在这个例子中，我们创建了一个名为`Identifier`的枚举，其中包含三个可能的值：`Name`、`Number`和`NoIdentifier`。然后我们创建了一个名为`playerIdentifier`的`Identifier`枚举实例，其值为`Number`，关联值为`2`。然后我们使用`if-case`语句来查看`playerIdentifier`是否有`Number`的值，如果有，我们将在控制台打印一条消息。

就像`for-case`语句一样，我们可以使用`where`语句进行额外的过滤。以下示例使用了我们在上一个例子中使用的相同的`Identifier`枚举：

```swift
var playerIdentifier = Identifier.Number(2)
if case let .Number(num) = playerIdentifier, num == 2 { 
    print("Player is either XanderBogarts or Derek Jeter")
} 
```

在这个例子中，我们使用了`if-case`语句来查看`playerIdentifier`是否有`Number`的值，但我们还添加了`where`语句来查看关联值是否等于`2`。如果是这样，我们将玩家识别为`XanderBogarts`或`Derek Jeter`。

正如我们在示例中看到的那样，使用`case`和`where`语句与我们的条件语句可以减少执行某些类型过滤所需的行数。它还可以使我们的代码更容易阅读。现在让我们来看看控制转移语句。

# 控制转移语句

控制转移语句用于将控制权转移到代码的另一个部分。Swift 提供了六个控制转移语句；这些是`continue`、`break`、`fallthrough`、`guard`、`throws`和`return`。我们将在*第七章*，*函数*中查看`return`语句，并在*第十二章*，*可用性和错误处理*中讨论`throws`语句。剩余的控制转移语句将在本节中讨论。

## `continue`语句

`continue`语句告诉循环停止执行代码块并转到循环的下一个迭代。以下示例展示了我们如何使用此语句来打印出范围内的奇数：

```swift
for i in 1...10 { 
    if i % 2 == 0 {
        continue
}
    print("\(i) is odd")
} 
```

在前面的例子中，我们遍历了从 1 到 10 的范围。对于`for-in`循环的每次迭代，我们使用余数（`%`）运算符来查看数字是奇数还是偶数。如果数字是偶数，`continue`语句告诉循环立即转到循环的下一个迭代。如果数字是奇数，我们打印出该数字是奇数，然后继续。前面代码的输出如下：

```swift
1 is odd
3 is odd
5 is odd
7 is odd
9 is odd 
```

现在让我们来看看`break`语句。

## `break`语句

`break`语句立即结束控制流中的代码块执行。以下示例演示了当我们遇到第一个偶数时如何从`for-in`循环中退出：

```swift
for i in 1...10 { 
    if i % 2 == 0 {
        break
}
    print("\(i) is odd")
} 
```

在前面的示例中，我们遍历从 1 到 10 的范围。对于`for-in`循环的每次迭代，我们使用取余（`%`）运算符来判断数字是奇数还是偶数。如果数字是偶数，我们使用`break`语句立即退出循环。如果数字是奇数，我们打印出该数字是奇数，然后进入循环的下一个迭代。前面的代码有以下输出：

```swift
1 is odd 
```

## 跌落语句

在 Swift 中，`switch`语句不像其他语言那样会跌落；然而，我们可以使用`fallthrough`语句来强制它们跌落。`fallthrough`语句可能非常危险，因为一旦找到匹配项，下一个情况将默认为`true`，并且执行那个代码块。

这在以下示例中得到了说明：

```swift
var name = "Jon"
var sport = "Baseball"
switch sport {
    case "Baseball":
        print("\name) plays Baseball")
        fallthrough
    case "Basketball":
        print("\(name) plays Basketball")
        fallthrough
    default:
        print("Unknown sport")
} 
```

当运行此代码时，以下结果将打印到控制台：

```swift
Jon plays Baseball
Jon plays Basketball
Unknown sport 
```

我建议您在使用`fallthrough`语句时要非常小心。苹果公司故意禁用了`case`语句的跌落，以避免程序员常见的错误。通过使用`fallthrough`语句，您可能会将这些错误重新引入到您的代码中。

# 摘要

在本章中，我们介绍了 Swift 中的控制流和函数。在继续前进之前，理解本章中的概念是至关重要的。我们编写的每个应用程序，除了简单的 Hello World 应用程序之外，都将非常依赖控制流语句和函数。

控制流语句用于在我们应用程序中做出决策，并且函数，我们将在下一章讨论，用于将我们的代码分组到可重用和有组织的部分。
