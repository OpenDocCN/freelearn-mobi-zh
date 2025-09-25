# *第五章*：介绍函数式编程

本章将讨论函数式编程的基本原则以及它们如何融入**Kotlin**编程语言。

正如你将发现的，我们已经在本章中触及了一些概念，因为如果不涉及函数式编程的概念，如**数据不可变性**和**函数作为值**，那么讨论语言的优点将非常困难。但就像我们之前做的那样，我们将从不同的角度来探讨这些特性。

在本章中，我们将涵盖以下主题：

+   函数式方法背后的原因

+   不可变性

+   函数作为值

+   表达式，而非语句

+   递归

完成本章后，你将了解函数式编程的概念如何嵌入到 Kotlin 语言中，以及何时使用它们。

# 技术要求

对于本章，你需要安装以下内容：

+   **IntelliJ IDEA** **社区版** ([`www.jetbrains.com/idea/download/`](https://www.jetbrains.com/idea/download/))

+   **OpenJDK** **11**（或更高版本）([`openjdk.java.net/install/`](https://openjdk.java.net/install/))

你可以在**GitHub**上找到本章的代码文件，地址为[`github.com/PacktPublishing/Kotlin-Design-Patterns-and-Best-Practices/tree/main/Chapter05`](https://github.com/PacktPublishing/Kotlin-Design-Patterns-and-Best-Practices/tree/main/Chapter05)。

# 函数式方法背后的原因

**函数式编程**与其他编程范式一样历史悠久，例如过程式和面向对象编程。但在过去的 15 年里，它已经取得了显著的进展。原因在于其他方面的发展停滞了：**CPU**速度。我们无法像过去那样大幅提高 CPU 速度，因此我们必须**并行化**我们的程序。而且结果证明，函数式编程范式在运行并行任务方面非常出色。

多核处理器的演变本身就是一个迷人的话题，但在这里我们只会简要介绍。工作站自 20 世纪 80 年代以来就已经拥有多个处理器，以支持并行运行来自不同用户的任务。由于这个时代的工作站体积庞大，它们不需要担心将所有内容都挤在一个芯片上。但到了 2005 年左右，消费市场出现了多处理器，这时就需要一个能够并行工作的物理单元。这就是为什么我们的 PC 或笔记本电脑中有一个多核芯片。

但这并不是我们使用函数式编程的唯一原因。这里还有更多：

+   函数式编程倾向于纯函数，而纯函数通常更容易推理和测试。

+   以函数式方式编写的代码通常比命令式代码更具声明性，处理的是**什么**而不是**如何**，这可以是一个优点。

在接下来的章节中，我们将探讨函数式编程的不同方面，从*不可变性*开始。

# 不可变性

函数式编程的一个基本概念是**不可变性**。这意味着从函数接收输入的那一刻起，到函数返回输出的那一刻，对象不会改变。*但是它怎么能改变呢？*好吧，让我们看看一个简单的例子：

```kt
fun <T> printAndClear(list: MutableList<T>) {
```

```kt
    for (e in list) {
```

```kt
        println(e)
```

```kt
        list.remove(e)
```

```kt
    }
```

```kt
}
```

```kt
printAndClear(mutableListOf("a", "b", "c"))
```

这段代码将首先输出`a`，然后我们会收到`ConcurrentModificationException`。

原因在于，`for-each` 循环使用了一个迭代器（我们已经在上一章中讨论过），而在循环内部修改列表会干扰其操作。然而，这引发了一个问题：

*如果我们可以从一开始就保护自己免受这些运行时异常的影响，那岂不是很好？*

让我们看看*不可变集合*如何帮助我们解决这个问题。

## 不可变集合

在*第一章*，“Kotlin 入门”，我们已经提到 Kotlin 中的集合默认是不可变的，这与许多其他语言不同。

前一个问题是由我们没有遵循*单一职责原则*引起的，该原则指出，一个函数应该只做一件事，并且要做好。我们的函数试图同时从数组中删除元素并打印它们。

如果我们将参数从`MutableList`改为`List`，我们就无法调用其上的`remove()`函数，从而解决我们当前的问题。但这又提出了另一个问题：

*如果我们需要一个空列表怎么办？*

在这种情况下，我们的函数应该返回一个新的对象：

```kt
private fun <T> printAndClear(list: MutableList<T>): 
```

```kt
  MutableList<T> {
```

```kt
    for (e in list) {
```

```kt
        println(e)
```

```kt
    }
```

```kt
    return mutableListOf()
```

```kt
}
```

通常，在函数式编程中应避免不返回任何值的函数，因为这通常意味着它们有副作用。

然而，集合*类型*不可变还不够。集合的*内容*也应该不可变。为了更好地理解这一点，让我们看看以下简单的类：

```kt
data class Player(var score: Int)
```

您可以看到，这个类只有一个变量：`score`。

接下来，我们将创建一个`data` `class`的单个实例并将其放入不可变集合中：

```kt
val scores = listOf(Player(0))
```

我们可以在集合中放置这个类的多个实例，但为了说明我们的观点，只需要一个实例。

接下来，让我们介绍*线程*的概念。

## 共享可变状态的问题

如果您对**线程**不熟悉，不要担心，我们将在*第六章*，“线程和协程”中详细讨论它们。现在您需要知道的是，线程允许代码*并发*运行。当使用并发代码和利用多个 CPU 的代码时，函数式编程非常有帮助。您可能会发现，任何不涉及并发的其他示例都可能显得相当复杂或人为。

现在，让我们创建一个包含两个线程的列表：

```kt
val threads = List(2) {
```

```kt
        thread {
```

```kt
            for (i in 1..1000) {
```

```kt
                scores[0].score++
```

```kt
            }
```

```kt
        }
```

```kt
    }
```

如您所见，每个线程使用常规的`for`循环总共增加`score` 1000。

我们通过使用`join()`等待线程完成，然后检查计数器的值：

```kt
for (t in threads) {
```

```kt
    t.join()
```

```kt
}
```

```kt
println(scores[0].score) // Less than 2000 for sure
```

如果你自己运行代码，值将在`2000`以下。

这是一个可变变量的经典*竞态条件*案例。每次运行此代码时，你得到的结果都会不同。如果你之前遇到过并发，这个原因可能对你来说很熟悉。顺便说一句，这与线程没有完成它们的工作无关。你可以在循环后添加一个打印消息来确保这一点：

```kt
thread { 
```

```kt
    for (i in 1..1000) { 
```

```kt
        scores[0].score = scores[0].score + 1 
```

```kt
    }
```

```kt
    println("Done")
```

```kt
}
```

这也不是使用增量（`++`）操作符的错。正如你所见，我们使用了长格式来增加值，但如果你尽可能多次地运行它，你仍然会得到错误的结果。

这种行为的原因是加法操作和赋值操作不是原子的。两个线程可能会覆盖彼此的加法操作，导致数值增加的次数不够。

在这里，我们使用了一个包含恰好一个元素的集合的极端例子。在现实世界中，你将要处理的集合通常包含多个元素。例如，你可能需要跟踪多个玩家的得分，甚至同时维护数千个玩家的排名系统。这会使例子变得更加复杂。

你需要记住的是以下内容：即使一个集合是不可变的，它仍然可能包含可变对象。可变对象不是线程安全的。

接下来，让我们看看元组，它们是不可变对象的一个例子。

## 元组

在函数式编程中，一个`pair`：

```kt
val pair = "a" to 1
```

`pair`包含两个属性，称为`first`和`second`，是不可变的：

```kt
pair.first = "b" // Doesn't work
```

```kt
pair.second = 2  // Still doesn't
```

我们可以使用`解构声明`将`pair`解构为两个单独的值：

```kt
val (key, value) = pair
```

```kt
println("$key => $value")
```

当迭代映射时，我们还会处理另一种类型的元组：`Map.Entry`：

```kt
for (p in mapOf(1 to "Sunday", 2 to "Monday")) {
```

```kt
   println("${p.key} ${p.value}")
```

```kt
}
```

这个元组已经包含了`key`和`value`成员，而不是`first`和`second`。

除了`pair`之外，还有一个包含`third`值的`Triple`元组：

```kt
val firstEdition = Triple("Design Patterns with Kotlin",   310, 2018)
```

通常，`data`类是元组的一个很好的实现，因为它们提供了清晰的命名。如果你看前面的例子，并不立即明显地看出`310`值代表页数。

然而，正如我们在上一节中看到的，并不是每个`data`类都是合适的元组。你需要确保它的所有成员都是值而不是变量。你还需要检查它是否有嵌套的不可变集合或类。

现在，让我们讨论函数式编程中的另一个重要主题：函数作为语言的第一等公民。

# 函数作为值

我们已经在关于设计模式的章节中介绍了 Kotlin 的一些函数式能力。**策略**和**命令**设计模式只是两个依赖于接受函数作为参数、返回函数、将函数作为值存储或将函数放入集合中的能力的例子。在本节中，我们将介绍 Kotlin 函数式编程的一些其他方面，例如*函数纯度*和*柯里化*。

## 了解高阶函数

如我们之前讨论的，在 Kotlin 中，一个函数可以返回另一个函数。让我们看看以下简单的函数，以深入了解这个语法：

```kt
fun generateMultiply(): (Int) -> Int {
```

```kt
    return fun(x: Int): Int {
```

```kt
        return x * 2
```

```kt
    }
```

```kt
}
```

在这里，我们的 `generateMultiply` 函数返回另一个没有名字的函数。没有名字的函数被称为**匿名函数**。

我们也可以使用更短的语法重写前面的代码：

```kt
fun generateMultiply(): (Int) -> Int {
```

```kt
    return { x: Int ->
```

```kt
        x * 2
```

```kt
    }
```

```kt
} 
```

如果一个没有名字的函数使用短语法，它被称为**lambda 函数**。

接下来，让我们看看返回类型的签名：

```kt
(Int) -> Int
```

从这个签名中，我们知道我们返回的函数将接受一个整数作为输入并产生一个整数作为输出。

如果一个函数不接受任何参数，我们使用空圆括号来表示：

```kt
() -> Int
```

如果一个函数不返回任何内容，我们使用 `Unit` 类型来指定：

```kt
(Int) -> Unit
```

Kotlin 中的函数可以被分配给变量或值，稍后调用：

```kt
val multiplyFunction = generateMultiply()
```

```kt
...
```

```kt
println(multiplyFunction(3, 4))
```

分配给变量的函数通常被称为**字面函数**。

我们在*第四章*“熟悉行为模式”中讨论策略设计模式时应用了这一点。

也可以将函数指定为参数：

```kt
fun mathInvoker(x: Int, y: Int, mathFunction: (Int, Int) ->   Int) {
```

```kt
    println(mathFunction(x, y))
```

```kt
}
```

```kt
mathInvoker(5, 6, multiplyFunction)
```

如果函数是最后一个参数，它也可以以临时方式提供，在括号之外：

```kt
mathInvoker(7, 8) { x, y ->
```

```kt
   x * y
```

```kt
}
```

这种语法也被称为**尾随 lambda**或**调用后缀**。我们在*第四章*“熟悉行为模式”中讨论解释器设计模式时看到了这个例子。

现在我们已经了解了函数的基本语法，让我们看看它们是如何被使用的。

## 标准库中的高阶函数

当使用 Kotlin 时，你将每天都会做的一件事是与**集合**一起工作。正如我们在*第一章*“开始使用 Kotlin”中简要提到的，集合支持高阶函数。

例如，在前面的章节中，为了逐个打印集合中的元素，我们使用了无聊的 `for-each` 循环：

```kt
val dwarfs = listOf("Dwalin", "Balin", "Kili", "Fili",   "Dori", "Nori", "Ori", "Oin", "Gloin", "Bifur", "Bofur",   "Bombur", "Thorin")
```

```kt
for (d in dwarfs) {
```

```kt
    println(d)
```

```kt
}
```

很多人看到这个可能都会感到沮丧。但我希望你没有完全停止阅读这本书。当然，在许多编程语言中，还有另一种实现相同目标的方法：一个 `forEach` 函数：

```kt
dwarfs.forEach { d -> 
```

```kt
    println(d)
```

```kt
}
```

这个函数是高阶函数最基本的一个例子。让我们看看它是如何声明的：

```kt
fun <T> Iterable<T>.forEach(action: (T) -> Unit)
```

在这里，`action` 是一个接收集合中元素但不返回任何内容的函数。这个函数提供了一个讨论 Kotlin 另一个方面的机会：`it` 语法。

# `it` 语法

在函数式编程中，保持函数小而简单是非常常见的。函数越简单，理解起来就越容易，也越有可能在其他地方被重用。而重用代码的目标是 Kotlin 的基本原则之一。

注意，在先前的例子中，我们没有指定`d`变量的类型。我们可以使用我们之前使用过的相同冒号符号来完成此操作：

```kt
dwarfs.forEach { d: String ->  
```

```kt
    println(d) 
```

```kt
}
```

然而，通常我们不需要这样做，因为编译器可以从我们使用的泛型类型中推断出这一点。毕竟，`dwarfs`是`List<String>`类型，所以`d`也是`String`类型。

当我们编写像这样简短的 lambda 表达式时，我们不需要省略的不仅仅是参数的类型。如果一个 lambda 只有一个参数，我们可以使用它的隐含名称，在这种情况下，是`it`：

```kt
dwarfs.forEach {
```

```kt
    println(it)
```

```kt
}
```

在我们需要将单个函数调用到单个参数的情况下，我们也可以使用*函数引用*。我们在*第四章*中看到了一个例子，*熟悉行为模式*，在讨论策略设计模式时：

```kt
dwarfs.forEach(::println)
```

在接下来的大多数例子中，我们将使用最简短的表示法。建议在像*嵌套在一个 lambda 中的 lambda*这样的情况下使用较长的语法。在这些情况下，为参数提供适当的名称比简洁更重要。

## 闭包

在面向对象范式中，状态始终存储在对象中。但在函数式编程中，这并不一定是这种情况。让我们以以下函数为例：

```kt
fun counter(): () -> Int {
```

```kt
    var i = 0
```

```kt
    return { i++ }
```

```kt
}
```

之前的例子显然是一个高阶函数，正如您可以通过其`return`类型看到的那样。它返回一个不带参数的函数，该函数产生一个整数。

让我们按照我们已经学到的这种方式将其存储在一个变量中，并多次调用它：

```kt
val next = counter()
```

```kt
println(next())
```

```kt
println(next())
```

```kt
println(next())
```

如您所见，该函数能够保持状态，在这种情况下，计数器的值，即使它不是对象的一部分。

这被称为**闭包**。lambda 可以访问包装它的函数的所有局部变量，并且只要保持对 lambda 的引用，这些局部变量就会持续存在。

闭包的使用是函数式编程工具箱中的另一个工具，它减少了定义大量仅用一些状态包装单个函数的类的需求。

## 纯函数

**纯函数**是一个没有副作用的功能。副作用可以被认为是任何访问或改变外部状态的行为。外部状态可以是非局部变量（其中闭包中的变量仍然被认为是非局部的）或任何类型的 IO（即，读取或写入文件或使用任何类型的网络功能）。

重要提示：

对于不熟悉这个术语的人来说，**IO**代表**输入/输出**，这涵盖了任何超出我们程序范围的外部交互，例如写入文件或从网络读取。

例如，我们在*闭包*部分讨论的 lambda 不被认为是*纯*的，因为它在多次调用时对相同的输入可以返回不同的输出。

**不纯函数**通常很难测试和推理，因为它们返回的结果可能取决于执行顺序或我们无法控制的因素（例如网络问题）。

要记住的一件事是，记录日志或甚至打印到控制台仍然涉及 I/O，并受到相同问题的影响。

让我们看看以下简单的函数：

```kt
fun sayHello() = println("Hello")
```

*那么，在这种情况下，你如何确保打印出 Hello 呢？* 这个任务并不像看起来那么简单，因为我们需要某种方式来捕获标准输出——即我们通常看到打印内容的同一个控制台。

我们将把它与以下函数进行比较：

```kt
fun hello() = "Hello"
```

以下函数没有任何副作用。这使得测试它变得容易得多：

```kt
fun testHello(): Boolean {
```

```kt
    return "Hello" == hello()
```

```kt
}
```

`hello()` 函数可能看起来有点没有意义，但实际上这是纯函数的一个特性。如果我们提前知道结果，它们的调用可以被它们的结果所替代。这通常被称为**引用透明性**。

如我们之前提到的，不是每个用 Kotlin 编写的函数都是纯函数：

```kt
fun <T> removeFirst(list: MutableList<T>): T {
```

```kt
    return list.removeAt(0)
```

```kt
}
```

如果我们在同一个列表上两次调用该函数，它将返回不同的结果：

```kt
val list = mutableListOf(1, 2, 3)
```

```kt
println(removeFirst(list)) // Prints 1
```

```kt
println(removeFirst(list)) // Prints 2
```

将前面的函数与这个函数比较：

```kt
fun <T> withoutFirst(list: List<T>): T {
```

```kt
    return ArrayList(list).removeAt(0)
```

```kt
}
```

现在，我们的函数是完全可预测的，无论我们调用多少次：

```kt
val list = mutableListOf(1, 2, 3)
```

```kt
println(withoutFirst(list)) // It's 1
```

```kt
println(withoutFirst(list)) // Still 1
```

如您所见，在这个例子中，我们使用了一个不可变的接口`List<T>`，它通过防止我们修改输入来帮助我们。当与上一节中讨论的不可变值结合使用时，纯函数通过提供可预测的结果和算法的并行化，使得测试更容易。

利用纯函数的系统更容易推理，因为它不依赖于任何外部因素——所见即所得。

## 柯里化

**柯里化**是将接受多个参数的函数转换为一系列函数的方法，其中每个函数只接受一个参数。这听起来可能有些令人困惑，所以让我们看看一个简单的例子：

```kt
fun subtract(x: Int, y: Int): Int {
```

```kt
    return x - y
```

```kt
}
```

```kt
println(subtract(50, 8))
```

这是一个接受两个参数作为输入并返回它们之间差的函数。然而，一些语言允许我们使用以下语法调用此函数：

```kt
subtract(50)(8)
```

这就是柯里化的样子。柯里化允许我们将具有多个参数的函数（在我们的例子中是两个）转换为一系列函数，其中每个函数只接受一个参数。

让我们看看如何在 Kotlin 中实现这一点。我们已经看到如何从一个函数中返回另一个函数：

```kt
fun subtract(x: Int): (Int) -> Int {
```

```kt
    return fun(y: Int): Int {
```

```kt
        return x - y
```

```kt
    }
```

```kt
}
```

这里是前面代码的简短形式：

```kt
fun subtract(x: Int) = fun(y: Int): Int {
```

```kt
    return x - y
```

```kt
}
```

在前面的例子中，我们使用单表达式语法返回一个匿名函数，而不需要声明`return`类型或使用`return`关键字。

这里是它的更简短形式：

```kt
fun subtract(x: Int) = {y: Int -> x - y}
```

现在，一个匿名函数被转换为一个 lambda，lambda 的`return`类型也被推断出来。

虽然它本身可能不是非常有用，但它仍然是一个有趣的概念，值得掌握。如果你是一位正在寻找新工作的**JavaScript**开发者，确保你完全理解它，因为几乎在每次面试中都会问到它。

你可能会想要使用柯里化的一个真实世界场景是*日志记录*。一个`log`函数通常看起来像这样：

```kt
enum class LogLevel {
```

```kt
    ERROR, WARNING, INFO
```

```kt
}
```

```kt
fun log(level: LogLevel, message: String) =     println("$level: $message")
```

我们可以通过将函数存储在变量中来设置日志级别：

```kt
val errorLog = fun(message: String) {
```

```kt
    log(LogLevel.ERROR, message)
```

```kt
}
```

注意到`errorLog`函数比常规的`log`函数更容易使用，因为它只接受一个参数而不是两个。然而，这引发了一个问题：

*如果我们不想提前创建所有可能的记录器呢？*

在这种情况下，我们可以使用柯里化。这个代码的*柯里化*版本将看起来像这样：

```kt
fun createLogger(level: LogLevel): (String) -> Unit {
```

```kt
    return { message: String ->
```

```kt
        log(level, message)
```

```kt
    }
```

```kt
}
```

现在，取决于谁使用我们的代码，他们可以创建他们想要的记录器：

```kt
val infoLogger = createLogger(LogLevel.INFO)
```

```kt
infoLogger("Log something")
```

实际上，这与我们在*第二章*中讨论的工厂设计模式非常相似，*使用创建型模式*。同样，现代语言的力量减少了我们需要实现相同行为所需的自定义类的数量。

接下来，让我们谈谈另一种强大的技术，它可以让我们免于重复进行相同的计算。

## 记忆化

如果我们的函数总是对相同的输入返回相同的输出，我们就可以轻松地将输入映射到输出，并在过程中缓存结果。这种技术称为**记忆化**。

在开发不同类型的系统或解决问题时，一个常见的任务是找到一种方法来避免多次重复相同的计算。假设我们收到多个整数列表，并且对于每个列表，我们希望打印其总和：

```kt
val input = listOf(
```

```kt
    setOf(1, 2, 3),
```

```kt
    setOf(3, 1, 2),
```

```kt
    setOf(2, 3, 1),
```

```kt
    setOf(4, 5, 6)
```

```kt
)
```

看一下输入，你可以看到前三个集合实际上是相等的——区别只在于元素的顺序，所以计算三次总和将是浪费的。

求和计算可以很容易地描述为一个纯函数：

```kt
fun sum(numbers: Set<Int>): Double {
```

```kt
    return numbers.sumByDouble { it.toDouble() }
```

```kt
}
```

这个函数不依赖于任何外部状态，并且以任何方式都不会改变外部状态。因此，对于相同的输入，可以用它之前返回的值替换对这个函数的调用是安全的。

我们可以在可变映射中存储相同集合的前一次计算结果：

```kt
val resultsCache = mutableMapOf<Set<Int>, Double>()
```

为了避免创建过多的类，我们可以使用一个高阶函数，它会将结果包装在我们之前创建的缓存中：

```kt
fun summarizer(): (Set<Int>) -> Double {
```

```kt
    val resultsCache = mutableMapOf<Set<Int>, Double>()
```

```kt
    return { numbers: Set<Int> ->
```

```kt
        resultsCache.computeIfAbsent(numbers, ::sum)
```

```kt
    }
```

```kt
}
```

这里，我们使用方法引用操作符（`::`）告诉`computeIfAbsent`在输入尚未缓存的情况下使用`sum()`方法。

注意，`sum()`是一个纯函数，而`summarize()`则不是。后者对于相同的输入会有不同的行为。但这正是我们想要的。

在前面的输入上运行以下代码将只调用求和函数两次：

```kt
val summarizer = summarizer()
```

```kt
input.forEach {
```

```kt
    println(summarizer(it))
```

```kt
}
```

不变对象、纯函数和闭包的结合为我们提供了强大的性能优化工具。只需记住：没有什么是免费的。我们用 CPU 时间这种资源交换另一种资源，即内存。而决定每种情况下哪种资源更昂贵的是你。

# 使用表达式而不是语句

**语句**是一段不返回任何内容的代码块。相反，**表达式**返回一个新的值。由于语句不产生结果，它们唯一有用的方式是改变状态，无论是改变一个变量、改变数据结构还是执行某种类型的 I/O。

函数式编程试图尽可能避免改变状态。从理论上讲，我们越依赖表达式，我们的函数就越纯，从而获得函数式纯度的所有好处。

我们已经多次使用过`if`表达式，因此它的一些好处应该是显而易见的：它比其他语言的`if`语句更简洁，因此更不容易出错。

## 模式匹配

`switch`/`case`的强化概念。我们已经看到了`when`表达式是如何被使用的，这在*第一章*，“Kotlin 入门”中进行了探讨，所以让我们简要讨论一下为什么这个概念对于函数式范式很重要。

你可能知道，在 Java 中，`switch`只能接受一些原始类型、字符串或枚举。

考虑以下代码，这通常用来演示语言中如何实现多态性：

```kt
class Cat : Animal {
```

```kt
    fun purr(): String {
```

```kt
        return "Purr-purr";
```

```kt
    }
```

```kt
}
```

```kt
class Dog : Animal {
```

```kt
    fun bark(): String {
```

```kt
        return "Bark-bark";
```

```kt
    }
```

```kt
}
```

```kt
interface Animal
```

如果我们要决定调用哪个函数，我们需要编写类似于以下代码：

```kt
fun getSound(animal: Animal): String {
```

```kt
    var sound: String? = null;
```

```kt
    if (animal is Cat) {
```

```kt
        sound = (animal as Cat).purr();
```

```kt
    }
```

```kt
    else if (animal is Dog) {
```

```kt
        sound = (animal as Dog).bark();
```

```kt
    }
```

```kt
    if (sound == null) {
```

```kt
        throw RuntimeException();
```

```kt
    }
```

```kt
    return sound;
```

```kt
}
```

这段代码试图在运行时确定`getSound`类实现了哪些方法。

这个方法可以通过引入多个返回值来缩短，但在实际项目中，通常认为多个返回值是不良实践。

由于我们没有为类提供`switch`语句，我们需要用`if`语句来代替。

现在，让我们将前面的代码与以下 Kotlin 代码进行比较：

```kt
fun getSound(animal: Animal) = when(animal) {
```

```kt
    is Cat -> animal.purr()
```

```kt
    is Dog -> animal.bark()
```

```kt
    else -> throw RuntimeException("Unknown animal")
```

```kt
}
```

由于`when`是一个表达式，我们完全避免了之前声明的中间变量。此外，使用模式匹配的代码不需要任何类型检查和转换。

现在我们已经学会了如何用更函数式的`when`表达式替换命令式的`if`语句，让我们看看我们如何通过使用*递归*来替换代码中的命令式循环。

# 递归

**递归**是一个函数用新的参数调用自身。许多著名的算法，如**深度优先搜索**，都依赖于递归。

这里是一个使用递归计算给定列表中所有数字之和的非常低效的函数示例：

```kt
fun sumRec(i: Int, sum: Long, numbers: List<Int>): Long {
```

```kt
    return if (i == numbers.size) {
```

```kt
        return sum
```

```kt
    } else {
```

```kt
        sumRec(i+1, numbers[i] + sum, numbers)
```

```kt
    }
```

```kt
}
```

我们经常试图避免递归，因为我们可能会收到栈溢出错误，如果我们的调用栈太深的话。你可以用一个包含一百万个数字的列表来调用这个函数，以演示这一点：

```kt
val numbers = List(1_000_000) {it}
```

```kt
println(sumRec(0,  numbers)) 
```

```kt
// Crashed pretty soon, around 7K
```

然而，Kotlin 支持一种称为**尾递归**的优化。尾递归的一个巨大好处是它避免了可怕的栈溢出异常。如果我们的函数中只有一个递归调用，我们可以使用这个优化。

让我们使用一个新的关键字 `tailrec` 重写我们的递归函数，以避免这个问题：

```kt
tailrec fun sumRec(i: Int, sum: Long, numbers: List<Int>): 
```

```kt
  Long {
```

```kt
    return if (i == numbers.size) {
```

```kt
        return sum
```

```kt
    } else {
```

```kt
        sumRec(i+1, numbers[i] + sum, numbers)
```

```kt
    }
```

```kt
}
```

现在，编译器将优化我们的调用并完全避免异常。

然而，如果你有多个递归调用，这种优化就不起作用了，例如在**归并排序**算法中。

让我们检查以下函数，这是归并排序算法的**排序**部分：

```kt
tailrec fun mergeSort(numbers: List<Int>): List<Int> {
```

```kt
    return when {
```

```kt
        numbers.size <= 1 -> numbers
```

```kt
        numbers.size == 2 -> {
```

```kt
            return if (numbers[0] < numbers[1]) {
```

```kt
                numbers
```

```kt
            } else {
```

```kt
                listOf(numbers[1], numbers[0])
```

```kt
            }
```

```kt
        }
```

```kt
        else -> {
```

```kt
            val left = mergeSort(numbers.slice               (0..numbers.size / 2))
```

```kt
            val right = mergeSort(numbers.slice               (numbers.size / 2 + 1 until numbers.size))
```

```kt
            return merge(left, right)
```

```kt
        }
```

```kt
    }
```

```kt
} 
```

注意，这里有两个递归调用而不是一个。Kotlin 编译器随后将发出以下警告：

```kt
> "A function is marked as tail-recursive but no tail calls are found"
```

# 摘要

你现在应该对函数式编程及其优势以及 Kotlin 如何处理这个主题有更好的理解。我们讨论了**不可变性**和**纯函数**的概念，以及将它们结合起来如何产生更容易测试且更容易维护的代码。

我们讨论了 Kotlin 如何支持**闭包**，这允许一个函数访问其包装函数的变量，并有效地在执行之间存储状态。这使我们可以使用**柯里化**和**记忆化**等技术，这些技术允许我们固定一些函数参数（通过充当默认值）并记住函数返回的值，以避免重新计算。

我们了解到 Kotlin 使用 `tailrec` 关键字来允许编译器优化**尾递归**。我们还探讨了**高阶函数**、**表达式与语句的区别**以及**模式匹配**。所有这些概念都使我们能够编写更容易测试且并发错误风险更低的代码。

在下一章中，我们将将这些知识应用于实践，并了解**响应式编程**是如何建立在函数式编程之上以创建可扩展和健壮的系统的。

# 问题

1.  什么是高阶函数？

1.  Kotlin 中的 `tailrec` 关键字是什么？

1.  什么是纯函数？
