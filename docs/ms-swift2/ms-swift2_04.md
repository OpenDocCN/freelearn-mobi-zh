# 第四章。控制流和函数

当我在我的 Vic-20 上学习 BASIC 编程时，每个月我都会阅读几本早期的计算机杂志，如《Byte 杂志》。我记得我阅读的一个特别评论；它是一篇关于一款名为《Zork》的游戏的评论。虽然《Zork》不是为我的 Vic-20 提供的游戏，但游戏的概念深深吸引了我，因为我真的很喜欢科幻和奇幻。我记得在想，写一个那样的游戏会多么酷，所以我决定找出如何做到这一点。当时我必须掌握的最大概念之一是根据用户的操作控制应用程序的流程。

在本章中，我们将涵盖以下主题：

+   条件语句是什么以及如何使用它们

+   循环是什么以及如何使用它们

+   控制转移语句是什么以及如何使用它们

+   如何在 Swift 中创建和使用函数

# 我们到目前为止学到了什么

到目前为止，我们一直在为使用 Swift 编写应用程序打下基础。虽然我们可以用我们学到的东西编写一个非常基础的程序，但仅使用我们前三章中涵盖的内容来编写一个有用的应用程序将会非常困难。

从本章开始，我们将开始从 Swift 语言的根基中走出来，并开始学习使用 Swift 进行应用开发的构建块。在本章中，我们将介绍控制流和函数。要成为 Swift 编程语言的大师，完全理解和掌握本章以及第五章中讨论的概念是非常重要的。

在我们介绍控制流和函数之前，让我们看看在 Swift 中如何使用花括号和括号。

## 括号

在 Swift 中，与其它 C-like 语言不同，条件语句和循环需要使用花括号。在其它 C-like 语言中，如果条件语句或循环只有一个要执行的语句，那么围绕该行的花括号是可选的。这导致了大量的错误和 bug，例如苹果的`goto fail` bug；因此，当苹果设计 Swift 时，他们决定使用花括号，即使只有一行代码要执行。让我们看看一些说明这一点的代码。以下第一个例子在 Swift 中是无效的，因为它缺少花括号；然而，它将在大多数其它语言中是有效的：

```swift
if (x > y)
  x=0
```

在 Swift 中，你需要使用花括号，如下面的例子所示：

```swift
if (x > y) {
  x=0
}
```

## 括号

与其它 C-like 语言不同，Swift 中条件表达式的括号是可选的。在上面的例子中，我们围绕条件表达式放置了括号，但它们不是必需的。以下例子在 Swift 中是有效的，但在大多数 C-like 语言中则不是：

```swift
if x > y {
  x=0
}
```

# 控制流

控制流，也称为流程控制，指的是在应用程序中语句、指令或函数执行的顺序。Swift 支持所有熟悉的 C 类语言中的控制流语句。这包括循环（包括 `for` 和 `while`）、条件语句（包括 `if` 和 `switch`）和控制语句的转移（包括 `break` 和 `continue`）。除了标准的 C 控制流语句之外，Swift 还添加了额外的语句，例如 `for-in` 循环，并增强了一些现有的语句，例如 `switch` 语句。

让我们从查看 Swift 中的条件语句开始。

## 条件语句

条件语句将检查条件，并且只有当条件为真时才会执行代码块。Swift 提供了 `if` 和 `if-else` 条件语句。让我们看看如何使用这些条件语句在指定的条件为真时执行代码块。

### `if` 语句

`if` 语句将检查条件语句，如果条件为真，它将执行代码块。`if` 语句采用以下格式：

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

在前面的示例中，我们首先设置了 `teamOneScore` 和 `teamTwoScore` 常量。然后我们使用 `if` 语句来检查 `teamOneScore` 的值是否大于 `teamTwoScore` 的值。如果值更大，我们将 `Team One Won` 打印到控制台。如果我们运行此代码，我们确实会看到 `Team One Won` 被打印到控制台，但如果 `teamTwoScore` 的值大于 `teamOneScore` 的值，则不会打印任何内容到控制台。这并不是编写应用程序的最佳方式，因为我们希望用户知道哪个队伍实际上获胜了。`if-else` 语句可以帮助我们解决这个问题。

### 使用 `if-else` 语句进行条件代码执行

`if-else` 语句将检查条件语句，如果条件为真，它将执行一个代码块。如果条件语句不为真，它将执行另一个代码块。`if-else` 语句遵循以下格式：

```swift
if condition {
    block of code if true
} else {
    block of code if not true
}
```

让我们修改前面的示例，使用 `if-else` 语句来告诉用户哪个队伍获胜：

```swift
var teamOneScore = 7
var teamTwoScore = 6
if teamOneScore > teamTwoScore {
    print("Team One Won")
} else {
    print("Team Two Won")
}
```

这个新版本将在 `teamOneScore` 的值大于 `teamTwoScore` 的值时打印出 `Team One Won`；否则，它将打印出消息，`Team Two Won`。你认为如果 `teamOneScore` 的值等于 `teamTwoScore` 的值，代码会做什么？在现实世界中，我们将会有平局，但在前面的代码中，我们将打印出 `Team Two Won`；这对第一队来说是不公平的。在这种情况下，我们可以使用多个 `else if` 语句和一个普通的 `else` 语句，如下面的示例所示：

```swift
var teamOneScore = 7
var teamTwoScore = 6
if teamOneScore > teamTwoScore {
    print("Team One Won")
} else if teamTwoScore > teamOneScore {
    print("Team Two Won")
} else {
    print("We have a tie")
}
```

在前面的代码中，如果 `teamOneScore` 的值大于 `teamTwoScore` 的值，我们将打印 `Team One Won` 到控制台。然后我们还有一个 `if` 语句来检查 `teamTwoScore` 的值是否大于 `teamOneScore` 的值，但这个 `if` 语句跟随一个 `else` 语句，这意味着只有当前面的条件语句为假时，才会检查这个 `if` 语句。最后，如果两个 `if` 语句都为假，那么我们假设值是相等的，并将 `We have a tie` 打印到控制台。

条件语句会检查条件一次，如果条件满足，它将执行代码块。如果我们想连续执行代码块直到满足条件怎么办？为此，我们将使用 Swift 中的循环语句之一。让我们看看 Swift 中的循环语句。

## For 循环

`for` 循环的变体可能是最广泛使用的循环语句。Swift 提供了基于 C 的标准 `for` 循环以及额外的 `for-in` 循环。基于 C 的标准 `for` 循环会执行代码块直到满足条件，通常是通过增加或减少计数器来实现的。`for-in` 语句将为范围、集合或序列中的每个项目执行代码块。当我们需要遍历集合或有一个固定次数要执行代码块时，我们通常使用 `for` 循环的变体之一。

### 使用 `for` 循环变体

让我们从查看基于 C 的标准 `for` 循环及其使用方法开始。`for` 语句的格式看起来类似于这个：

```swift
for initialization; condition; update-rule {
   block of code
}
```

如前所述的格式所示，`for` 循环有三个部分：

+   `初始化`：这是初始化所需变量的地方；如果需要，可以包含多个初始化，用逗号分隔

+   `条件`：这是需要检查的条件；当条件为假时，循环将退出

+   `更新规则`：这是每个循环结束时需要更新的内容

理解各个部分被调用的顺序是很重要的。当代码执行遇到一个 `for` 循环时，会调用 `for` 循环的初始化部分来初始化变量。接下来，执行条件部分以验证是否应该执行代码块，如果是的话，它将执行代码块。最后，调用更新规则来在循环回跳并重新开始之前执行任何更新。

以下示例展示了如何使用 `for` 循环遍历一系列数字：

```swift
for var index = 1; index <= 4; index++ {
    print(index)
}
```

在前面的例子中，`index` 变量被初始化为数字 `1`。在每次循环的开始，我们检查 `index` 变量是否等于或小于数字 `4`。如果 `index` 变量等于或小于数字 `4`，则执行代码的内层块，并将 `index` 变量的值打印到控制台。最后，我们在循环回跳并重新开始之前增加 `index` 变量。一旦 `index` 变量大于 `4`，`for` 循环就会退出。如果我们运行前面的例子，数字 `1` 到 `4` 将确实被打印到控制台。

`for` 循环最常见的一个用途是遍历一个集合并对该集合中的每个项目执行一段代码。让我们看看如何遍历一个数组，然后通过一个例子来看看如何遍历一个字典：

```swift
var countries = ["USA","UK", "IN"]
for var index = 0; index < countries.count; index++ {
    print(countries[index])
}
```

在前面的例子中，我们首先使用三个国家的缩写初始化 `countries` 数组。在 `for` 循环中，我们将 `index` 变量初始化为 `0`（数组的第一个索引），并在 `for` 循环的条件语句中检查 `index` 变量是否小于 `countries` 数组中的元素数量。每次循环时，我们从 `countries` 数组中检索并打印由 `index` 变量指定的索引处的值。

新程序员在使用 `for` 循环遍历数组时犯的最大错误之一是使用小于或等于 (`<=`) 操作符而不是小于 (`<`) 操作符。使用小于或等于 (`<=`) 操作符会导致循环迭代次数过多，并在代码运行时生成 `Index out of Bounds` 异常。在前面的例子中，小于或等于 (`<=`) 操作符将生成从 `0` 到 `3`（包括）的计数，因为数组中有三个元素；然而，数组中的元素索引从 `0` 到 `2`（0、1 和 2）。因此，当我们尝试获取索引 `3` 的值时，将抛出 `Index out of Bounds` 异常。建议使用 `for-in` 循环遍历数组而不是标准 `for` 循环。我们将在本章稍后讨论 `for-in` 循环。

让我们看看如何使用基于 C 的标准 `for` 循环遍历字典：

```swift
var dic = ["USA": "United States", "UK": "United Kingdom", "IN":"India"]

var keys  = Array(dic.keys)
for var index = 0; index < keys.count; index++ {
  print(dic[keys[index]])
}
```

在前面的例子中，我们首先创建了一个包含国家名称作为值、缩写作为键的字典对象。然后我们使用字典的 `keys` 属性来获取键的数组。在 `for` 循环中，我们将 `index` 变量初始化为 `0`，验证 `index` 变量是否小于国家数组中的元素数量，并在每次循环结束时增加 `index` 变量。每次循环时，我们将打印国家的名称到控制台。

现在，让我们看看如何使用 `for-in` 语句以及它如何帮助我们防止在使用标准 `for` 语句时出现的常见错误。

### 使用 `for-in` 循环变体

在标准的`for`循环中，我们提供一个索引，然后循环直到满足条件。虽然这种方法非常好，但当我们想要遍历一系列数字时，它可能会导致错误，正如之前提到的，如果我们的条件语句不正确。`for-in`循环旨在防止这些类型的异常。

`for-in`循环遍历一个项集合或数字范围，并为集合或范围中的每个项执行一段代码。`for-in`语句的格式看起来类似于以下：

```swift
for variable in Collection/Range {
  block of code
}
```

如前述代码所示，`for-in`循环有两个部分：

+   `变量`: 这个变量会在每次`for-in`循环执行时改变，并保存集合或范围内的当前项

+   `集合/范围`: 这是遍历的集合或范围

让我们看看如何使用`for-in`循环遍历一系列数字：

```swift
for index in 1...5 {
    print(index)
}
```

在前面的例子中，我们遍历了从`1`到`5`的数字范围，并将每个数字打印到控制台。这个特定的`for-in`语句使用了闭包范围运算符（`…`）为`for-in`循环提供一个遍历的范围。Swift 还提供了一个名为半开范围运算符的第二个范围操作（`..<`）。半开范围运算符遍历数字范围，但不包括最后一个数字。让我们看看如何使用半开范围运算符：

```swift
for index in 1..<5 {
  print(index)
}
```

在闭包范围运算符示例（`…`）中，我们将看到数字`1`到`5`被打印到控制台。在半开范围运算符示例中，最后一个数字（`5`）将被排除；因此，我们将看到数字`1`到`4`被打印到控制台。

现在，让我们看看如何使用`for-in`循环遍历数组：

```swift
var countries = ["USA","UK", "IN"]
for item in countries {
    print(item)
}
```

在前面的例子中，我们遍历了`countries`数组，并将`countries`数组中的每个元素打印到控制台。正如我们所见，使用`for-in`循环遍历数组比使用标准的基于 C 的`for`循环更安全、更简洁，也更简单。使用`for-in`循环可以防止我们犯一些常见的错误，例如在条件语句中使用`<=`（小于等于）运算符而不是`<`（小于）运算符。

让我们看看如何使用`for-in`循环遍历字典：

```swift
var dic = ["USA": "United States", "UK": "United Kingdom", "IN":"India"]

for (abbr, name) in dic {
  print("\(abbr) --  \(name)")
}
```

在前面的例子中，我们使用了`for-in`循环来遍历字典中的每个键值对。在这个例子中，字典中的每个项都作为（键，值）元组返回。我们可以在`for-in`循环体中将（键，值）元组成员分解为命名常量。需要注意的是，由于字典不保证存储项的顺序，遍历的顺序可能与插入的顺序不同。

现在，让我们看看另一种类型的循环，即`while`循环。

## `while`循环

`while`循环会执行一个代码块，直到满足某个条件。Swift 提供了两种`while`循环的形式；这些是`while`循环和`repeat-while`循环。在 Swift 2.0 中，Apple 用`repeat-while`循环替换了`do-while`循环。`repeat-while`循环的功能与`do-while`循环完全相同。现在，Apple 使用`do`语句来进行错误处理。

当要执行的迭代次数未知且通常依赖于某些业务逻辑时，我们使用`while`循环。当你想运行零次或多次循环时，使用`while`循环；而当你想运行一次或多次循环时，使用`repeat-while`循环。

### 使用`while`循环

`while`循环首先评估一个条件语句，然后如果条件语句为真，则重复执行一个代码块。`while`语句的格式如下：

```swift
while condition {
  block of code
}
```

让我们看看如何使用`while`循环。在下面的例子中，如果生成的随机数小于`4`，`while`循环将继续循环。在这个例子中，我们使用`arc4random()`函数在`0`和`4`之间生成一个随机数：

```swift
var ran = 0
while ran < 4 {
    ran = Int(arc4random() % 5)
}
```

在前面的例子中，我们首先将`ran`变量初始化为`0`。然后`while`循环检查`ran`变量，如果其值小于`4`，就会生成一个新的随机数，介于`0`和`4`之间。只要生成的随机数小于`4`，`while`循环将继续循环。一旦生成的随机数等于或大于`4`，`while`循环将退出。

在前面的例子中，`while`循环在生成新的随机数之前会检查条件语句。如果我们不想在生成随机数之前检查条件语句呢？我们可以在第一次初始化`ran`变量时生成一个随机数，但这意味着我们需要复制生成随机数的代码，而复制代码永远不是一种理想的做法。在这种情况下，使用`repeat-while`循环会更合适。

### 使用`repeat-while`循环

`while`循环和`repeat-while`循环的区别在于，`while`循环在第一次执行代码块之前会检查条件语句；因此，所有在条件语句中的变量都需要在执行`while`循环之前初始化。`repeat-while`循环会在第一次检查条件语句之前运行循环块；这意味着我们可以在条件代码块中初始化变量。当条件语句依赖于循环块中的代码时，`repeat-while`循环的使用更为合适。`repeat-while`循环的格式如下：

```swift
repeat {
   block of code
} while condition
```

让我们通过创建一个`repeat-while`循环来查看这个具体的例子，在这个循环中，我们在循环块内初始化我们要检查的变量，在条件`while`语句中：

```swift
var ran: Int
repeat {
    ran = Int(arc4random() % 5)
} while ran < 4
```

在前面的例子中，我们将`ran`变量定义为`Int`类型，但我们直到进入循环块并生成随机数之前都没有初始化它。如果我们尝试使用`while`循环（未初始化`ran`变量），我们会收到一个`变量在使用前未初始化`异常。

## `switch`语句

`switch`语句取一个值，然后将其与几个可能的匹配项进行比较，并根据第一次成功的匹配执行相应的代码块。当可能有多个匹配项时，`switch`语句是`if-else`语句的替代方案。`switch`语句的格式如下：

```swift
switch value {
  case match1 :
    block of code
  case match2 :
    block of code
  …… as many cases as needed
  default :
    block of code
}
```

与大多数其他语言中的`switch`语句不同，在 Swift 中，它不会自动跳转到下一个`case`语句；因此，我们不需要使用`break`语句来防止跳转。这是 Swift 内置的另一个安全特性，因为初学者在`switch`语句中最常见的编程错误之一就是忘记在`case`语句的末尾添加`break`语句。让我们看看如何使用`switch`语句：

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

在前面的例子中，`switch`语句将`speed`变量的值与两个`case`语句进行比较，如果`speed`的值与任一`case`匹配，它将打印出速度是多少。如果`switch`语句找不到匹配项，它将打印出`Unknown speed`消息。

每个`switch`语句都必须匹配所有可能的值。这意味着除非我们是在匹配枚举，否则每个`switch`语句都必须有一个`default`情况。让我们看看没有`default`情况的例子：

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

如果我们将前面的代码放入游乐场并尝试编译，我们会收到一个`switch 必须完整，考虑添加默认情况`错误。这是一个编译时错误；因此，我们只有在尝试编译代码时才会收到通知。

在单个`case`中可以包含多个项。要在单个`case`中设置多个项，我们需要用逗号分隔这些项。让我们看看我们如何使用`switch`语句来判断一个字符是元音还是辅音：

```swift
var char : Character = "e"
switch char {
case "a", "e", "i", "o", "u":
    print("letter is a vowel")
case "b", "c", "d", "f", "g", "h", "j", "k", "l", "m",
"n", "p", "q", "r", "s", "t", "v", "w", "x", "y", "z":
    print("letter is a consonant")
default:
    print("unknown letter")
}
```

我们可以在前面的例子中看到，每个`case`都有多个项。这些项用逗号分隔，`switch`语句将尝试将`char`变量与`case`语句中列出的每个项匹配。

我们也可以检查`switch`语句的值，看看它是否包含在某个范围内。为此，我们会在`case`语句中使用范围运算符，如下面的示例所示：

```swift
var grade = 93
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
default:
    print("Unknown Grade")
}
```

在前面的例子中，`switch`语句将`grade`变量与每个`case`语句中的`grade`范围进行比较，并打印出相应的等级。

在 Swift 中，任何 `case` 语句都可以包含一个可选的 guard 条件，它可以提供额外的条件来验证。guard 条件是用 `where` 关键字定义的。假设，在我们的上一个例子中，我们有一些学生在课堂上得到了特殊帮助，我们想要为他们定义一个 `D` 等级，范围在 `55` 到 `69` 之间。以下示例展示了如何做到这一点：

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

在处理 guard 表达式时，需要注意的一点是，Swift 将尝试从第一个 `case` 语句开始匹配值，并按顺序检查每个 `case` 语句。这意味着如果我们把带有 guard 表达式的 `case` 语句放在 Grade F `case` 语句之后，那么带有 guard 表达式的 `case` 语句将永远不会被触及。以下示例说明了这一点：

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
//The following case statement would never be reached because the
//grades would always match one of the previous two
case 55...69 where studentId == 4:
    print("Grade is a D for student 4")
default:
    print("Unknown Grade")
}
```

### 注意

一个好的经验法则是，如果你正在使用 guard 表达式，总是把带有 guard 条件的 `case` 语句放在任何没有 guard 表达式的类似 `case` 语句之前。

`Switch` 语句对于评估枚举也非常有用。由于枚举具有有限数量的值，如果我们为枚举中的所有值提供了 `case` 语句，我们就不需要提供默认情况。以下示例展示了我们如何使用 `switch` 语句来评估枚举：

```swift
 Product {
    case Book(String, Double, Int)
    case Puzzle(String, Double)
}

var order = Product.Book("Mastering Swift 2", 49.99, 2015)

switch order {
case .Book(let name, let price, let year):
    print("You ordered the book \(name) for \(price)")
case .Puzzle(let name, let price):
    print("You ordered the Puzzle \(name) for \(price)")
}
```

在这个例子中，我们首先定义了一个名为 `Product` 的枚举，它有两个值，每个值都关联着相应的值。然后我们创建了一个 `order` 变量，其类型为产品类型，并使用 `switch` 语句来评估它。请注意，我们在 `switch` 语句的末尾没有放置默认情况。如果我们稍后向产品枚举中添加额外的值，我们需要在 `switch` 语句的末尾放置一个默认情况，或者添加额外的 `case` 语句来处理额外的值。

## 使用 `case` 和 `where` 语句与条件语句

正如我们在上一节中看到的，`switch` 语句中的 `case` 和 `where` 语句可以非常强大。从 Swift 2 开始，我们能够使用这些语句与 `if`、`for` 和 `while` 等其他条件语句一起使用。在我们的条件语句中使用 `case` 和 `where` 语句可以使我们的代码更加紧凑且易于阅读。让我们看看一些示例，从使用 `where` 语句在 `for-in` 循环中过滤结果开始。

### 使用 `where` 语句进行过滤

在本节中，我们将看到如何使用 `where` 语句来过滤 `for-in` 循环的结果。为了举例，我们将使用一个整数数组，并只打印出偶数；然而，在我们查看如何使用 `where` 语句过滤结果之前，让我们看看没有使用 `where` 语句会如何做：

```swift
for number in 1…30 {
    if number % 2 == 0 {
        print(number)
    }
}
```

在这个例子中，我们使用一个 `for-in` 循环来遍历数字 `1` 到 `30`。在 `for-in` 循环内部，我们使用一个 `if` 条件语句来过滤出奇数。在这个简单的例子中，代码读起来相当容易，但让我们看看我们如何使用 `where` 语句来减少代码行数并使其更容易阅读：

```swift
for number in 1...30 where number % 2 == 0 {
    print(number)
}
```

我们仍然有与上一个例子相同的 `for-in` 循环；然而，现在我们将 `where` 语句放在最后，在这个特定的例子中，我们只遍历偶数。使用 `where` 语句缩短了我们的例子两行，并且也使其更容易阅读，因为过滤语句与 `for-in` 循环在同一行，而不是嵌入在循环本身中。

现在，让我们看看如何使用 `for-case` 语句进行过滤。

### 使用 `for-case` 语句进行过滤

在下一个例子中，我们将使用 `for-case` 语句来过滤一个元组数组，并仅打印出符合我们标准的结果。`for-case` 例子与我们之前看到的 `where` 语句非常相似，它旨在消除循环中过滤结果时需要 `if` 语句的需求。在这个例子中，我们将使用 `for-case` 语句来过滤一系列世界系列冠军，并打印出特定球队赢得世界系列赛的年份：

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
    ("Giants", 2014)
]

for case let ("Red Sox", year) in worldSeriesWinners {
    print(year)
}
```

在这个例子中，我们创建了一个名为 `worldSeriesWinners` 的元组数组，其中数组中的每个元组都包含球队的名称和他们赢得世界系列赛的年份。然后我们使用 `for-case` 语句来过滤这个数组，并仅打印出红袜队赢得世界系列赛的年份。过滤是在 `case` 语句中完成的，其中 `("Red Sox", year)` 表示我们想要所有第一个元素为字符串 `"Red Sox"` 且第二个元素的值赋给 `year` 常量的结果。然后 `for` 循环遍历 `case` 语句的结果，并打印出 `year` 常量的值。

`for-case` 语句也使得在可选数组中过滤出 nil 值变得非常容易。让我们看看一个这样的例子：

```swift
let myNumbers: [Int?] = [1, 2, nil, 4, 5, nil, 6]

for case let .Some(num) in myNumbers {
    print(num)
}
```

在这个例子中，我们创建了一个名为 `myNumbers` 的可选数组，它可能包含一个整数值，也可能包含 nil。正如我们将在第十章中看到的那样，*使用可选类型*，可选在内部被定义为枚举，如下面的代码所示：

```swift
enum Optional<T> {
     case None,
     case Some(T)
}
```

如果一个可选被设置为 nil，它将有一个值为 `None`，但如果它不是 nil，那么它将有一个值为 `Some`，并关联实际值的类型。在我们的例子中，当我们过滤 `.Some(num)` 时，我们正在寻找任何具有 `.Some (非 nil 值)` 的可选。作为 `.Some()` 的简写，我们可以使用 `?`（问号）符号，正如我们将在下面的例子中看到的那样。

我们还可以将 `for-case` 与 `where` 语句结合使用以进行额外的过滤，如下例所示：

```swift
let myNumbers: [Int?] = [1, 2, nil, 4, 5, nil, 6]

for case let num? in myNumbers where num > 3 {
    print(num)
}
```

这个例子与上一个例子相同，只是我们添加了 `where` 语句的额外过滤。在上一个例子中，我们遍历了所有非 nil 值，但在本例中，我们遍历了大于 3 的非 nil 值。让我们看看我们如何在不使用 `case` 或 `where` 语句的情况下进行相同的过滤：

```swift
for num in myNumbers {
    if let num = num {
        if num > 3 {
            print(num)
        }
    }
}
```

如我们所见，使用 `for-case` 和 `where` 语句可以大大减少所需的行数。它还使我们的代码更容易阅读，因为所有过滤语句都在同一行上。

让我们再看一个过滤示例。这次，我们将查看 `if-case` 语句。

### 使用 `if-case` 语句

使用 `if-case` 语句与使用 `switch` 语句非常相似。大多数情况下，`switch` 语句是首选，但在某些情况下，`if-case` 语句更好。其中一种情况是我们只寻找一两个可能的匹配项，并且我们不希望处理所有可能的匹配项。让我们看看这个例子：

```swift
enum Identifier {
    case Name(String)
    case Number(Int)
    case NoIdentifier
}

var playerIdentifier = Identifier.Number(42)

if case let .Number(num) = playerIdentifier {
    print("Player's number is \(num)")
}
```

在这个例子中，我们创建了一个名为 `Identifier` 的枚举，其中包含三个可能的值：`Name`、`Number` 和 `NoIdentifier`。我们创建了一个名为 `playerIdentifier` 的 `Identifier` 枚举实例，其值为 `Number`，关联值为 `42`。然后我们使用 `if-case` 语句检查 `playerIdentifier` 是否具有 `Number` 值，如果是，则向控制台打印一条消息。

就像 `for-case` 语句一样，我们能够使用 `where` 语句进行额外的过滤。以下示例使用了我们在上一个例子中使用的相同的 `Identifier` 枚举：

```swift
var playerIdentifier = Identifier.Number(42)

if case let .Number(num) = playerIdentifier where num == 2 {
    print("Player is either Xander Bogarts or Derek Jeter")
}
```

在这个例子中，我们仍然使用 `if-case` 语句来检查 `playerIdentifier` 是否具有 `Number` 值，但我们添加了 `where` 语句来检查关联值是否等于 `42`，如果是，我们将玩家标识为 `Xander Bogarts 或 Derek Jeter`。

正如我们在示例中看到的那样，使用 `case` 和 `where` 语句与我们的条件语句结合可以减少进行某些类型过滤所需的行数。它还可以使我们的代码更容易阅读。现在让我们看看控制转移语句。

## 控制转移语句

控制转移语句用于将控制权转移到代码的另一个部分。Swift 提供了五种控制转移语句；这些是 `continue`、`break`、`fallthrough`、`guard`、`throws` 和 `return`。我们将在本章后面的 *函数* 部分中查看 `return` 语句，并在 第七章 中讨论 `throws` 语句，*使用可用性和错误处理编写更安全的代码*。

### `continue` 语句

`continue` 语句告诉循环停止执行代码块并转到循环的下一个迭代。以下示例展示了如何使用 `continue` 语句仅打印出范围内的奇数：

```swift
for i in 1...10 {
    if i % 2 == 0 {
        continue
    }
    print("\(i) is odd")
}
```

在前面的示例中，我们遍历 `1` 到 `10` 的范围。对于 `for-in` 循环的每次迭代，我们使用取余 (`%`) 运算符来判断数字是奇数还是偶数。如果数字是偶数，`continue` 语句告诉循环立即转到下一个迭代。如果数字是奇数，我们打印出该数字是奇数，然后继续前进。前面代码的输出如下：

```swift
1 is odd
3 is odd
5 is odd
7 is odd
9 is odd
```

现在，让我们看看 `break` 语句。

### `break` 语句

`break` 语句立即结束控制流中的代码块执行。以下示例展示了当我们遇到第一个偶数时如何从 `for` 循环中退出：

```swift
for i in 1...10 {
    if i % 2 == 0 {
        break
    }
    print("\(i) is odd")
}
```

在前面的示例中，我们遍历 `1` 到 `10` 的范围。对于 `for` 循环的每次迭代，我们使用取余 (`%`) 运算符来判断数字是奇数还是偶数。如果数字是偶数，我们使用 `break` 语句立即退出循环。如果数字是奇数，我们打印出该数字是奇数，然后进入循环的下一个迭代。前面的代码有以下输出：

```swift
1 is odd
```

### 跌落语句

在 Swift 中，`switch` 语句不像其他语言那样会跌落；然而，我们可以使用 `fallthrough` 语句来强制它们跌落。`fallthrough` 语句可能非常危险，因为一旦找到匹配项，下一个情况默认为真，并且执行该代码块。以下示例说明了这一点：

```swift
var name = "Jon"
var sport = "Baseball"
switch sport {
case "Baseball":
    print("\(name) plays Baseball")
    fallthrough
case "Basketball":
    print("\(name) plays Basketball")
    fallthrough
default:
    print("Unknown sport")
}
```

在前面的示例中，由于第一个情况 `Baseball` 与代码匹配，并且剩余的代码块也执行，输出看起来类似于以下内容：

```swift
Jon plays Baseball
Jon plays Basketball
Unknown sport
```

### `guard` 语句

在 Swift 和大多数现代语言中，我们的条件语句往往侧重于测试一个条件是否为真。例如，以下代码检查变量 `x` 是否大于 `10`，如果是，则执行某些函数；否则，处理错误条件：

```swift
var x = 9
if x > 10 {
  // Functional code here
} else {
   // Do error condition
}
```

这种类型的代码导致我们的功能代码嵌入到我们的检查中，并且错误条件被放在函数的末尾，但如果我们不希望这样呢？有时，在函数的开始处理错误条件可能更好。我知道，在我们的简单示例中，我们可以轻松检查 `x` 是否小于或等于 `10`，如果是，则执行错误条件，但并非所有条件语句都那么容易重写，尤其是像可选绑定这样的项目。

在 Swift 2 中，Apple 引入了新的 `guard` 语句。`guard` 语句侧重于在条件为假时执行一个函数；这允许我们在函数的早期捕获错误并执行错误条件。我们可以使用 `guard` 语句重写之前的示例，如下所示：

```swift
var x = 9
guard x > 10 else {
  // Do error condition
  return
}
// Functional code here
```

在这个新示例中，我们检查变量 `x` 是否大于 `10`，如果不是，我们执行错误条件。如果变量 `x` 大于 `10`，我们的代码将继续执行。我们注意到在错误条件代码中嵌入了一个 `return` 语句。`guard` 语句中的代码必须包含一个控制转移语句；这正是防止其他代码执行的原因。如果我们忘记了控制转移语句，Swift 将在编译时显示错误。

让我们看看 `guard` 语句的一些更多示例。以下示例展示了我们如何使用 `guard` 语句来验证一个可选值是否包含有效值：

```swift
func guardFunction(str: String?) {
    guard let goodStr = str else {
        print("Input was nil")
        return
    }
    print("Input was \(goodStr)")
}
```

在这个示例中，我们创建了一个名为 `guardFunction()` 的函数，它接受一个包含字符串或 `nil` 值的可选值。然后我们使用 `guard` 语句和可选绑定来验证字符串可选值不包含 `nil`。如果它包含 `nil`，则执行 `guard` 语句中的代码，并使用 `return` 语句退出函数。使用 `guard` 语句和可选绑定时真正令人愉快的事情是，新变量在整个函数的范围内有效，而不仅仅是可选绑定语句的范围内。

现在我们已经看到了 Swift 中控制流语句的工作方式，让我们来介绍 Swift 中的函数和类。

# 函数

在 Swift 中，函数是一个执行特定任务的独立代码块。函数通常用于将我们的代码逻辑上分解成可重用的命名块。我们使用函数的名称来调用函数。

当我们定义一个函数时，我们还可以选择性地定义一个或多个参数（也称为参数）。参数是通过调用它的代码传递到函数中的命名值。这些参数通常在函数内部用于执行函数的任务。我们还可以为参数定义默认值，以简化函数的调用方式。

每个 Swift 函数都有一个与其关联的类型。这个类型被称为返回类型，它定义了从函数返回到调用它的代码的数据类型。如果一个函数没有返回值，则返回类型为 `Void`。

让我们看看如何在 Swift 中定义函数。

## 使用单个参数函数

定义 Swift 中函数的语法非常灵活。这种灵活性使得我们能够轻松地定义简单的 C 风格函数或更复杂的 Objective-C 风格函数，包括局部和外部参数名称。让我们看看如何定义函数的一些示例。以下示例接受一个参数，并且不向调用它的代码返回任何值（返回类型—`void`）：

```swift
func sayHello(name: String) -> Void {
let retString = "Hello " + name
   print( retString)
}
```

在前面的例子中，我们定义了一个名为 `sayHello` 的函数，它接受一个名为 `name` 的变量。在函数内部，我们打印出一个 `Hello` 问候语给这个人的名字。一旦函数内部的代码执行完毕，函数就会退出，控制权返回给调用它的代码。如果我们不想打印问候语，而是想将问候语返回给调用它的代码，我们可以添加一个返回类型，如下所示：

```swift
func sayHello2(name: String) ->String {
    let retString = "Hello " + name
    return retString
}
```

`->` 字符串定义了与函数相关联的返回类型是一个字符串。这意味着该函数必须将一个 `string` 变量返回给调用它的代码。在函数内部，我们构建一个包含问候信息的 `string` 常量，然后使用 `return` 关键字返回这个 `string` 常量。

调用 Swift 函数与在其他语言（如 C 或 Java）中调用函数或方法非常相似。以下示例展示了如何在函数内部调用将问候消息打印到屏幕上的 `sayHello()` 函数：

```swift
sayHello("Jon")
```

现在，让我们看看如何调用返回值的 `sayHello2()` 函数，该函数将值返回给调用它的代码：

```swift
var message = sayHello2("Jon")
print(message)
```

在前面的例子中，我们调用了 `sayHello2()` 函数，并将返回的值存储在 `message` 变量中。如果一个函数定义了返回类型，例如 `sayHello2()` 函数那样，它必须返回一个与该类型相匹配的值给调用它的代码。因此，函数内的每一个可能的条件路径都必须以返回指定类型的值结束。这并不意味着调用函数的代码必须检索返回的值。以下是一个例子，其中两行都是有效的：

```swift
sayHello2("Jon")
var message = sayHello2("Jon")
```

如果你没有指定一个变量来存储返回值，该值将被丢弃。

## 使用多参数函数

我们在函数中不仅限于只有一个参数，我们还可以定义多个参数。要创建一个多参数函数，我们在括号中列出参数，并用逗号分隔参数定义。让我们看看如何在函数中定义多个参数：

```swift
func sayHello(name: String, greeting: String) {
    print("\(greeting) \(name)")
}
```

在前面的例子中，该函数接受两个参数：`name` 和 `greeting`。然后我们使用这两个参数在控制台上打印一个 `greeting`。

调用一个多参数函数与调用一个单参数函数略有不同。在调用多参数函数时，我们用逗号分隔参数，并且除了第一个参数之外，我们还需要包括所有其他参数的名称。以下示例展示了如何调用一个多参数函数：

```swift
sayHello("Jon", greeting:"Bonjour")
```

如果我们为参数定义了默认值，我们不需要为函数的每个参数提供一个参数。让我们看看如何为我们的参数配置默认值。

## 定义参数的默认值

我们可以在函数定义中通过使用等于运算符（`=`）为参数定义默认值，当我们声明变量时。以下示例展示了如何声明一个具有默认参数值的函数：

```swift
func sayHello(name: String, greeting: String = "Bonjour") {
    print("\(greeting) \(name)")
}
```

在函数声明中，我们定义了一个没有默认值（`name: String`）的参数和一个具有默认值（`greeting: String = "Bonjour"`）的参数。当一个参数有默认值声明时，我们可以选择在调用函数时为该参数设置值或不设置值。以下示例展示了如何调用 `sayHello()` 函数而不设置 `greeting` 参数，以及如何设置 `greeting` 参数来调用它：

```swift
sayHello("Jon")
sayHello("Jon", greeting: "Hello")
```

在 `sayHello("Jon")` 行中，`sayHello()` 函数将输出消息 `Bonjour Jon`，因为它使用了 `greeting` 参数的默认值。在 `sayHello("Jon", greeting: "Hello")` 行中，`sayHello()` 函数将输出消息 `Hello Jon`，因为我们覆盖了 `greeting` 参数的默认值。

我们可以通过使用参数名称来声明多个具有默认值的参数，并且只覆盖我们想要的那些。以下示例展示了我们如何通过在调用时覆盖其中一个默认值来实现这一点：

```swift
func sayHello4(name: String, name2: String = "Kim", greeting: String = "Bonjour") {
    println("\(greeting) \(name) and \(name2)")
}

sayHello("Jon", greeting: "Hello")
```

在上述示例中，我们声明了一个没有默认值（`name: String`）的参数和两个具有默认值（`name2: String = "Kim", greeting: String = "Bonjour"`）的参数。然后我们调用函数，让 `name2` 参数保持其默认值，但覆盖了 `greeting` 参数的默认值。

上述示例将输出消息，`Hello Jon and Kim`。

## 从函数中返回多个值

有几种方法可以从 Swift 函数中返回多个值。最常见的方法是将值放入集合类型（数组或字典）中，然后返回该集合。以下示例展示了如何从 Swift 函数中返回一个集合类型：

```swift
func getNames() -> [String] {
    var retArray = ["Jon", "Kim", "Kailey", "Kara"]
    return retArray
}

var names = getNames()
```

在上述示例中，我们声明了没有参数且返回类型为 `[String]` 的 `getNames()` 函数。`[String]` 返回类型指定返回类型为字符串类型的数组。

返回集合类型的一个缺点是集合中的值必须是同一类型，或者我们必须将我们的集合类型声明为 `AnyObject` 类型。在上述示例中，我们的数组只能返回字符串类型。如果我们需要与字符串一起返回数字，我们可以返回一个 `AnyObjects` 类型的数组，然后使用类型转换来指定对象类型。然而，这并不是我们应用程序的一个很好的设计，因为它很容易出错。返回不同类型值的一个更好的方式是使用元组类型。

当我们从函数返回一个元组时，建议我们使用命名元组，这样我们可以使用点语法来访问返回的值。以下示例展示了如何从函数中返回一个命名元组并访问返回的命名元组的值：

```swift
func getTeam() -> (team:String, wins:Int, percent:Double) {
    let retTuple = ("Red Sox", 99, 0.611)
 return retTuple
}

var t = getTeam()
print("\(t.team) had \(t.wins) wins")
```

在前面的示例中，我们定义了`getTeam()`函数，该函数返回一个包含三个值（`String`、`Int`和`Double`）的命名元组。在函数内部，我们创建了将要返回的元组。注意，只要元组中的值类型与函数定义中的值类型匹配，我们就不需要将将要返回的元组定义为命名元组。然后我们可以像调用其他函数一样调用该函数，并使用点语法来访问返回的元组的值。在前面示例中，代码会打印出以下行：

```swift
Red Sox had 99 wins
```

## 返回可选值

在前面的章节中，我们从函数中返回了非`nil`值；然而，这并不总是我们所需要的。如果我们需要从函数中返回一个`nil`值，会发生什么？以下代码会抛出`expression does not conform to type 'NilLiteralConvertible'`异常：

```swift
func getName() ->String {
    return nil
}
```

这段代码抛出异常的原因是我们将返回类型定义为`string`值；然而，我们尝试返回`nil`。如果有必要返回`nil`，我们需要将返回类型定义为可选类型，以便调用它的代码知道该值可能是`nil`。要定义返回类型为可选类型，我们使用问号(`?`)的方式，就像我们定义一个变量为可选类型时一样。以下示例展示了如何定义可选返回类型：

```swift
func getName() ->String? {
    return nil
}
```

前面的代码不会抛出异常。

我们也可以将元组设置为可选类型，或者将元组中的任何值设置为可选类型。以下示例展示了我们如何将元组作为可选类型返回：

```swift
func getTeam2(id: Int) -> (team:String, wins:Int, percent:Double)? {
    if id == 1 {
        return ("Red Sox", 99, 0.611)
    }
    return nil
}
```

在下面的示例中，我们可以返回一个在函数定义中定义的元组，或者返回一个`nil`；这两种选项都是有效的。如果我们需要元组中的某个值是`nil`，我们需要在元组中添加一个可选类型。以下示例展示了如何在元组中返回一个`nil`：

```swift
func getTeam() -> (team:String, wins:Int, percent:Double?) {
    let retTuple: (String, Int, Double?) = ("Red Sox", 99, nil)
    return retTuple
}
```

在前面的示例中，我们可以将`percent`值设置为`Double`值或`nil`。

## 添加外部参数名称

在本节前面的示例中，参数的定义方式类似于我们在 C 代码中定义参数的方式，即我们定义参数名称和值类型。当我们调用函数时，我们也可以像调用 C 代码中的函数一样调用它，其中我们使用函数名称并在括号内指定传递给函数的值。在 Swift 中，我们不仅限于这种语法；我们还可以使用外部参数名称。

当我们调用一个函数以指示每个参数的目的时，会使用外部参数名称。如果我们想在函数中使用外部参数名称，我们需要为每个参数定义一个外部参数名称，除了其局部参数名称之外。外部参数名称在函数定义中添加在局部参数名称之前。外部和局部参数名称之间用空格分隔。

让我们看看如何使用外部参数名称。但在这样做之前，让我们回顾一下我们之前是如何定义函数的。在接下来的两个示例中，我们将定义一个没有外部参数名称的函数，然后我们将使用外部参数名称重新定义该函数：

```swift
func winPercentage(team: String, wins: Int, loses: Int) -> Double {
    return Double(wins) / Double(wins + loses)
}
```

在前面的示例中，我们定义了接受三个参数的`winPercentage()`函数。这些参数是`team`、`wins`和`loses`。`team`参数是`String`类型，而`wins`和`loses`参数是`Int`类型。以下代码行展示了如何调用`winPercentage()`函数：

```swift
var per = winPercentage("Red Sox", wins: 99, loses: 63)
```

现在，让我们使用外部参数名称定义相同的函数：

```swift
func winPercentage(BaseballTeam team: String, withWins wins: Int, andLoses losses: Int) -> Double {
  return Double(wins) / Double(wins + losses)
}
```

在前面的示例中，我们使用外部参数名称重新定义了`winPercentage`函数。在这个重新定义中，我们有相同的三个参数：`team`、`wins`和`losses`。区别在于我们定义参数的方式。当使用外部参数时，我们使用外部参数名称和局部参数名称（两者之间用空格分隔）来定义每个参数。在前面的示例中，第一个参数有一个外部参数名称`BaseballTeam`，内部参数名称`team`，以及类型`String`。

当我们使用外部参数名称调用一个函数时，我们需要在函数调用中包含外部参数名称。以下代码展示了如何调用前面示例中的函数：

```swift
var per = winPercentage(BaseballTeam:"Red Sox", withWins:99, andLoses:63)
```

虽然使用外部参数名称需要更多的输入，但它确实使代码更容易阅读。在前面的示例中，很容易看出函数正在寻找一支棒球队的名称，第二个参数是获胜次数，最后一个参数是失败次数。

## 使用可变参数

可变参数是指可以接受零个或多个指定类型的值的参数。在函数定义内部，我们通过在参数类型名称后附加三个点（`...`）来定义可变参数。可变参数的值以指定类型的数组形式提供给函数。以下示例展示了我们如何在一个函数中使用可变参数：

```swift
func sayHello(greeting: String, names: String...) {
  for name in names {
    print("\(greeting) \(name)")
  }
}
```

在前面的示例中，`sayHello()`函数接受两个参数。第一个参数是字符串类型，即要使用的问候语。第二个参数是字符串类型的可变参数，即要向其发送问候语的名字。在函数内部，可变参数是一个包含指定类型的数组；因此，在我们的示例中，`names`参数是一个包含`String`值的数组。在这个例子中，我们使用一个`for-in`循环来访问`names`参数中的值。

以下行代码展示了如何使用可变参数调用`sayHello()`函数：

```swift
sayHello("Hello", names: "Jon", "Kim")
```

上一行代码将打印两个问候语：`Hello Jon`和`Hello Kim`。

## 参数作为变量

默认情况下，参数是常量，这意味着它们在函数内部不能被更改。让我们看看以下示例：

```swift
func sayHello(greeting: String, name: String, count: Int) {
  while count > 0 {
    print("\(greeting) \(name)")
    count--
  }
}
```

如果我们尝试运行此示例，我们将得到一个异常，因为我们尝试使用递减运算符（`--`）更改`count`参数的值。如果我们需要在函数内部更改参数的值，我们需要通过在函数定义中使用`var`关键字来指定该参数是一个变量。

以下示例展示了如何将`count`参数声明为一个变量（而不是一个常量），这样我们就可以在函数内部更改其值：

```swift
func sayHello(greeting: String, name: String, var count: Int) {
  while (count > 0) {
    println("\(greeting) \(name)")
    count--
  }
}
```

你可以看到在前面的示例中，我们在`count`参数名称之前添加了`var`关键字。这指定了参数是一个变量而不是常量；因此，我们可以在函数中更改`count`参数的值。

## 使用`inout`参数

如我们刚才所描述的，变量参数只能更改函数内部的参数值；因此，任何更改在函数结束后都会丢失。如果我们希望参数的更改在函数结束后仍然持续，我们需要将参数定义为`inout`参数。对`inout`参数所做的任何更改都会传递回函数调用中使用的变量。

使用`inout`参数时有两个需要注意的事项：这些参数不能有默认值，也不能是可变参数。

让我们看看如何使用`inout`参数来交换两个变量的值：

```swift
func swap(inout first: String, inout second: String) {
  let tmp = first
  first = second
  second = tmp
}
```

此函数将接受两个参数，并交换函数调用中使用的变量的值。当我们调用函数时，我们在变量名称前放置一个井号（`&`），表示函数可以修改其值。以下示例展示了如何调用反转函数：

```swift
var one = "One"
var two = "Two"
swap(&one,&two)
print("one: \(one) two: \(two)")
```

在前面的示例中，我们将变量`one`设置为值`One`，变量`two`设置为值`Two`。然后我们使用`one`和`two`变量调用反转函数。一旦交换函数返回，名为`one`的变量将包含值`Two`，而名为`two`的变量将包含值`One`。

## 函数嵌套

我们迄今为止所展示的所有函数都是全局函数的例子。全局函数是在它们所在的类或文件中的全局作用域内定义的。Swift 还允许我们在一个函数内部嵌套另一个函数。嵌套函数只能在封装函数内部调用；然而，封装函数可以返回一个嵌套函数，这允许它在封装函数的作用域之外使用。我们将在本书的第十二章中介绍如何返回一个函数，*使用闭包*。

让我们看看如何通过创建一个简单的排序函数来嵌套函数，该函数将接受一个整数数组并对其进行排序：

```swift
func sort(inout numbers: [Int]) {
  //This is the nested function
  func reverse(inout first: Int, inout second: Int) {
    let tmp = first
    first = second
    second = tmp
  }
  //Nested function ends.

    var count = numbers.count

    while count > 0 {
      for var i = 1; i < count; i++ {
        if numbers[i] < numbers[i-1] {
          reverse(&numbers[i], second: &numbers[i-1])
        }
      }
    count--
    }
}
```

在前面的代码中，我们首先创建了一个名为`sort`的全局函数，它接受一个`inout`参数，即`Ints`数组。在`sort`函数内部，我们首先定义了一个名为`reverse`的嵌套函数。函数需要在调用之前在代码中定义，因此将所有嵌套函数放在全局函数的开始部分是一个好习惯，这样我们就可以在调用它们之前知道它们已经被定义了。`reverse`函数简单地交换传入的两个值。

在`sort`函数的主体中，我们实现了简单排序的逻辑。在这个逻辑中，我们比较数组中的两个数字，如果需要交换这两个数字，我们就调用嵌套的`reverse`函数来交换这两个数字。这个例子展示了我们如何有效地使用嵌套函数来组织我们的代码，使其易于维护和阅读。让我们看看如何调用全局的排序函数：

```swift
var nums: [Int] = [6,2,5,3,1]

sort(&nums)

for value in nums {
    print("--\(value)")
}
```

上述代码创建了一个包含五个整数的数组，然后将该数组传递给`sort`函数。当`sort`函数返回`nums`数组时，它包含了一个排序后的数组。

### 注意

正确使用嵌套函数时，它们可以非常有用。然而，过度使用它们是非常容易的。在创建嵌套函数之前，你可能想问自己为什么想要使用嵌套函数，以及通过使用嵌套函数你解决了什么问题。

# 将所有这些放在一起

为了巩固我们在本章学到的内容，让我们再看一个例子。对于这个例子，我们将创建一个函数来测试一个字符串值是否包含有效的 IPv4 地址。IPv4 地址是分配给使用互联网协议进行通信的计算机的地址。IP 地址由四个数值组成，范围从 0-255，由点（句号）分隔。一个有效的 IP 地址示例是 10.0.1.250：

```swift
func isValidIP(ipAddr: String?) -> Bool {

    guard let ipAddr = ipAddr else {
        return false
    }

let octets = ipAddr.characters.split { $0 == "."}.map{String($0)}

    guard octets.count == 4 else {
        return false
    }

    func validOctet(octet: String) -> Bool {
        guard let num = Int(String(octet))
            where num >= 0 && num < 256 else {
                return false
        }
        return true
    }

    for octet in octets {
        guard validOctet(octet) else {
            return false
        }
    }

    return true

}
```

由于`isValidIp()`函数的参数是可选类型，我们首先需要验证`ipAddr`参数是否不为 nil。为此，我们使用了一个带有可选绑定的`guard`语句，如果可选绑定失败，则返回一个布尔值`false`，因为 nil 不是一个有效的 IP 地址。

如果 `ipAddr` 参数包含一个非空值，我们将字符串分割成字符串数组，以点为分隔符。由于 IP 地址应该包含由点分隔的四个数字，我们使用 `guard` 语句来检查数组是否包含四个元素。如果不包含，我们返回 `false`，因为我们知道 `ipAddr` 参数没有包含有效的 IP 地址。

接下来，我们创建了一个名为 `validOctet()` 的嵌套函数，它有一个名为 `octet` 的 String 参数。这个嵌套函数将验证 `octet` 参数是否包含介于 0 和 255 之间的数值，如果是，它将返回一个 Boolean `true` 值，否则，它将返回一个 `false` Boolean 值。

最后，我们遍历通过在点处分割原始 `ipAddr` 参数创建的数组中的值，并将这些值传递给 `validOctet()` 嵌套函数。如果所有四个值都通过 `validOctet()` 函数的验证，我们就有一个有效的 IP 地址，并返回一个 Boolean `true` 值；然而，如果任何一个值未能通过 `validOctet()` 函数的验证，我们返回一个 Boolean `false` 值。

# 摘要

在本章中，我们介绍了 Swift 中的控制流和函数。在继续学习之前，理解本章中的概念是至关重要的。我们编写的每一个应用程序，除了简单的 Hello World 应用程序之外，都将非常依赖控制流语句和函数。

控制流语句用于在应用程序中做出决策，而函数将用于将我们的代码分组到可重用和组织化的部分。
