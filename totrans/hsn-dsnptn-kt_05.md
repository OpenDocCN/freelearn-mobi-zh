# 函数式编程

在本章中，我们将讨论函数式编程的基本原则，以及它们如何融入 Kotlin 编程语言。我们不会介绍很多新的语法，正如你很快就会看到的。在前面几章中，如果不涉及诸如 *数据不可变性* 和 *函数作为一等值* 等概念，很难讨论该语言的好处。但就像我们之前做的那样，我们将从不同的角度来审视这些特性：不是如何用它们以更好的方式实现已知的设计模式，而是它们的用途。

在本章中，我们将涵盖以下主题：

+   为什么选择函数式编程？

+   不可变性

+   作为值的函数

+   表达式，而不是语句

+   递归

# 为什么选择函数式编程？

函数式编程几乎与其他编程范式一样历史悠久，比如过程式编程和面向对象编程，如果不是更久。但近 10 年来，它已经取得了巨大的进展。原因在于其他一些事情停滞了：CPU 速度。我们不能再像过去那样大幅提高 CPU 速度，因此我们必须并行化我们的程序。结果证明，函数式编程范式在运行并行任务方面非常出色。

多核处理器的演变本身就是一个非常有趣的话题，但我们只能简要地介绍。工作站自 20 世纪 80 年代以来至少就有多个处理器，以支持并行运行来自不同用户的任务。由于工作站本身就很大，它们不需要担心将所有东西都挤在一个芯片上。但到了 2005 年左右，消费市场出现了多处理器，就需要一个能够并行工作的物理单元。这就是为什么我们的 PC 或笔记本电脑上有一个芯片上有多个核心的原因。

但这并不是一些人坚信函数式编程的唯一原因。这里还有更多：

+   函数式编程倾向于纯函数，而纯函数通常更容易推理和测试

+   以函数方式编写的代码通常比命令式代码更具声明性，处理的是 *what* 而不是 *how*

# 不可变性

函数式编程的一个关键概念是不可变性。这意味着从函数接收输入的那一刻起，到函数返回输出的那一刻止，对象不会改变。你怎么可能改变呢？让我们看看一个简单的例子：

```kt
fun <T> printAndClear(list: MutableList<T>) {
    for (e in list) {
        println(e)
        list.remove(e)
    }
}
printAndClear(mutableListOf("a", "b", "c"))
```

输出将会是首先 `"a"`，然后我们会收到`ConcurrentModificationException`。

如果我们一开始就能保护自己免受此类运行时异常的侵害，那岂不是很好？

# 元组

在函数式编程中，元组是在创建后无法更改的数据片段。Kotlin 中最基本的元组之一是 Pair：

```kt
val pair = "a" to 1
```

一对包含两个属性，第一个和第二个，且是不可变的：

```kt
pair.first = "b" // Doesn't work
pair.second = 2  // Still doesn't
```

我们可以将一对拆分为两个单独的值：

```kt
val (key, value) = pair
println("$key => $value")
```

当遍历映射时，我们会收到另一个元组，`Map.Entry`：

```kt
for (p in mapOf(1 to "Sunday", 2 to "Monday")) {
   println("${p.key} ${p.value}")
}
```

通常，*数据类*通常是元组的良好实现。但是，正如我们将在*值突变*部分中看到的，并非每个数据类都是合适的元组。

# 值突变

在 Maronic 中，我们希望计算一千场比赛的平均分。为此，我们有一个以下的数据类：

```kt
data class AverageScore(var totalScore: Int = 0,
                        var gamesPlayed: Int = 0) {
    val average: Int
        get() = if (gamesPlayed <= 0)
                    0
                else
                    totalScore / gamesPlayed
}
```

我们很聪明：我们通过检查除以零来保护自己免受任何无效输出的影响。

但当我们编写以下代码时会发生什么？

```kt
val counter = AverageScore()

thread(isDaemon = true) {
    while(true) counter.gamesPlayed = 0
}

for (i in 1..1_000) {
    counter.totalScore += Random().nextInt(100)
    counter.gamesPlayed++

    println(counter.average)
}
```

很快，你将不可避免地收到 `ArithmeticException`。我们的计数器不知何故变成了零。

如果你希望你的数据类是不可变的，请确保将所有属性指定为 `val`（值），而不是 `var`（变量）。

# 不可变集合

我想我们的初级开发者已经吸取了教训。相反，他们产生了以下代码，虽然效率不高，但去掉了那些变量：

```kt
data class ScoreCollector(val scores: MutableList<Int> = mutableListOf())

val counter = ScoreCollector()

for (i in 1..1_000) {
    counter.scores += Random().nextInt(100)

    println(counter.scores.sumBy { it } / counter.scores.size)
}
```

但邪恶的线程再次发动攻击：

```kt
thread(isDaemon= true, name="Maleficent") {
    while(true) counter.scores.clear()
}
```

我们再次收到 `ArithmeticException`。

如果你的数据类只包含值，那就不够了。如果它的值是一个集合，那么为了使数据类被认为是不可变的，这个集合必须是不可变的。同样的规则也适用于其他数据类中包含的类：

```kt
data class ImmutableScoreCollector(val scores: List<Int>)
```

现在，邪恶的线程甚至不能调用这个集合上的 `clear()`。但我们该如何向其中添加分数呢？

一种选择是将整个列表传递给构造函数：

```kt
val counter = ImmutableScoreCollector(List(1_000) {
    Random().nextInt(100)
})
```

# 函数作为值

我们已经在专门介绍设计模式的章节中涵盖了 Kotlin 的一些功能特性。其中，**策略**和**命令**设计模式只是少数几个大量依赖接受函数作为参数、返回函数、将它们作为值存储或将它们放入集合中的能力。在本节中，我们将介绍 Kotlin 中函数式编程的一些其他方面，例如函数纯度和柯里化。

# 高阶函数

正如我们之前讨论的，在 Kotlin 中，一个函数可以返回另一个函数：

```kt
fun generateMultiply(): (Int, Int) -> Int {
    return { x: Int, y: Int -> x * y}
}
```

函数也可以分配给变量或值，稍后调用：

```kt
val multiplyFunction = generateMultiply()
...
println(multiplyFunction(3, 4))
```

分配给变量的函数通常被称为*字面函数*。也可以将函数指定为参数：

```kt
fun mathInvoker(x: Int, y: Int, mathFunction: (Int, Int) -> Int) {
    println(mathFunction(x, y))
}

mathInvoker(5, 6, multiplyFunction)
```

如果函数是最后一个参数，它也可以在括号外以特定方式提供：

```kt
mathInvoker(7, 8) { x, y ->
   x * y
}
```

通常，没有名称的函数被称为*匿名函数*。如果一个没有名称的函数使用简短语法，它被称为 lambda：

```kt
val squareAnonymous = fun(x: Int) = x * x
val squareLambda = {x: Int -> x * x} 
```

# 纯函数

纯函数是没有副作用的函数。以下函数就是一个例子：

```kt
fun sayHello() {
    println("Hello")
}
```

你如何测试 `"Hello"` 是否确实被打印出来？这个任务并不像看起来那么简单，因为我们需要某种方法来捕获标准输出，即我们通常看到打印内容的同一个控制台。

将它与以下函数进行比较：

```kt
fun hello() = "Hello"
```

以下函数没有任何副作用。这使得它更容易测试：

```kt
fun testHello(): Boolean {
    return "Hello" == hello()
}
```

`hello()`函数看起来有点无意义吗？这实际上是纯函数的一个特性。它们的调用可以被它们的结果所替代（如果我们知道所有结果的话）。这通常被称为*引用透明性*。

并非所有用 Kotlin 编写的函数都是纯函数：

```kt
fun <T> removeFirst(list: MutableList<T>): T {
    return list.removeAt(0)
}
```

如果我们在同一个列表上两次调用该函数，它将返回不同的结果：

```kt
val list = mutableListOf(1, 2, 3)

println(removeFirst(list)) // Prints 1
println(removeFirst(list)) // Prints 2
```

尝试这个：

```kt
fun <T> withoutFirst(list: List<T>): T {
    return ArrayList(list).removeAt(0)
}
```

现在函数是完全可预测的，无论我们调用多少次：

```kt
val list = mutableListOf(1, 2, 3)

println(withoutFirst(list)) // It's 1
println(withoutFirst(list)) // Still 1
```

如您所见，我们这次使用了一个不可变接口 `List<T>`，它通过防止我们甚至有可能修改输入来帮助我们。与上一节中的不可变值一起，纯函数提供了一种非常强大的工具，它通过提供可预测的结果和算法的并行化，使我们更容易进行测试。

# 柯里化

柯里化是将接受多个参数的函数转换为一系列每个函数只接受一个参数的函数的方法。这听起来可能有些令人困惑，所以让我们看看一个简单的例子：

```kt
fun subtract(x: Int, y: Int): Int {
    return x - y
}
println(subtract(50, 8))
```

这是一个返回两个参数的函数。结果非常明显。但也许我们希望用以下语法调用这个函数：

```kt
subtract(50)(8)
```

我们已经看到如何从一个函数中返回另一个函数：

```kt
fun subtract(x: Int): (Int) -> Int {
    return fun(y: Int): Int {
        return x - y
    }
}
```

这里是它的简短形式：

```kt
fun subtract(x: Int) = fun(y: Int): Int {
    return x + y
}
```

这里还有一个更简短的形式：

```kt
fun subtract(x: Int) = {y: Int -> x - y}
```

虽然它本身可能不太有用，但掌握这个概念仍然很有趣。如果你是一位正在寻找新工作的 JavaScript 开发者，确保你真正理解它，因为几乎在每次面试中都会问到它。

# 记忆化

如果我们的函数总是对相同的输入返回相同的输出，我们就可以轻松地在先前的输入和输出之间建立映射，并将其用作缓存。这种技术被称为*记忆化*：

```kt
class Summarizer {
    private val resultsCache = mutableMapOf<List<Int>, Double>()

    fun summarize(numbers: List<Int>): Double {
        return resultsCache.computeIfAbsent(numbers, ::sum)
    }

    private fun sum(numbers: List<Int>): Double {
        return numbers.sumByDouble { it.toDouble() }
    }
}
```

我们使用方法引用操作符 `::` 来告诉 `computeIfAbsent` 在输入尚未缓存的情况下使用 `sum()` 方法。

注意，`sum()` 是一个纯函数，而 `summarize()` 则不是。后者对相同的输入会有不同的行为。但这正是我们想要的：

```kt
val l1 = listOf(1, 2, 3)
val l2 = listOf(1, 2, 3)
val l3 = listOf(1, 2, 3, 4)

val summarizer = Summarizer()

println(summarizer.summarize(l1)) // Computes, new input
println(summarizer.summarize(l1)) // Object is the same, no compute
println(summarizer.summarize(l2)) // Value is the same, no compute
println(summarizer.summarize(l3)) // Computes
```

不变对象、纯函数和平凡的类的组合为我们提供了一种强大的性能优化工具。只需记住，没有什么是免费的。我们只是用 CPU 时间交换另一种资源，即内存。而决定哪种资源对你来说更贵，则取决于你。

# 表达式，而不是语句

语句是一段不返回任何内容的代码块。另一方面，表达式返回一个新的值。由于语句不产生结果，它们唯一有用的方式就是改变状态。而函数式编程试图尽可能避免改变状态。理论上，我们越依赖表达式，我们的函数就越纯，从而获得所有函数式纯度的益处。

我们已经多次使用了`if`表达式，因此它的一些好处应该很清楚：它比`if`语句更简洁，因此更不容易出错。

# 模式匹配

模式匹配的概念对于来自 Java 的人来说就像`switch/case`的强化版。我们已经在第一章，“Kotlin 入门”，中看到了`when`表达式是如何使用的，所以让我们简要讨论一下为什么这个概念对于函数式范式来说很重要。

如你所知，Java 中的`switch`只能接受一些原始类型、字符串或枚举。

考虑以下 Java 代码：

```kt
class Cat implements Animal {
    public String purr() {
        return "Purr-purr";
    }
}

class Dog implements Animal {
    public String bark() {
        return "Bark-bark";
    }
```

```kt
}

interface Animal {}
```

如果我们要决定调用哪个函数，我们需要像这样：

```kt
public String getSound(Animal animal) {
    String sound = null;
    if (animal instanceof Cat) {
        sound = ((Cat)animal).purr();
    }
    else if (animal instanceof Dog) {
        sound = ((Dog)animal).bark();
    }

    if (sound == null) {
        throw new RuntimeException();
    }
    return sound;
}
```

这个方法可以通过引入多个返回来简化，但在实际项目中，通常不推荐使用多个返回。

由于我们没有为类提供`switch`语句，我们需要使用`if`语句代替。

与下面的 Kotlin 代码比较一下：

```kt
fun getSound(animal: Animal) = when(animal) {
    is Cat -> animal.purr()
    is Dog -> animal.bark()
    else -> throw RuntimeException()
}
```

由于`when`是一个表达式，我们完全避免了中间变量的使用。更重要的是，使用模式匹配，我们还可以避免大部分与类型检查和转换相关的代码。

# 递归

递归是一个函数用新的参数调用自己：

```kt
fun sumRec(i: Int, numbers: List<Int>): Long {
    return if (i == numbers.size) {
        0
    } else {
        numbers[i] + sumRec(i + 1, numbers)
    }
}
```

我们通常避免递归，因为如果我们的调用栈太深，我们可能会收到**栈溢出**错误。你可以用一个包含一百万个数字的列表调用这个函数来体验它：

```kt
val numbers = List(1_000_000) {it}
println(sumRec(0,  numbers)) // Crashed pretty soon, around 7K 
```

尾递归的一个巨大好处是它避免了可怕的栈溢出异常。

让我们使用一个新的关键字`tailrec`重写我们的递归函数，以避免这个问题：

```kt
tailrec fun sumRec(i: Int, sum: Long, numbers: List<Int>): Long {
    return if (i == numbers.size) {
        return sum
    } else {
        sumRec(i+1, numbers[i] + sum, numbers)
    }
}
```

现在编译器将优化我们的调用并完全避免异常。

# 摘要

你现在应该对函数式编程及其好处有了更好的理解。我们讨论了不可变性和纯函数的概念。两者的结合通常会产生更易于测试的代码，也更容易维护。

柯里化和记忆化是两个源自函数式编程的有用模式。

Kotlin 有一个`tailrec`关键字，允许编译器优化*尾递归*。我们还探讨了高阶函数、表达式与语句的区别，以及模式匹配。

在下一章中，我们将把所学知识应用于实践，并了解如何通过在函数式编程的基础上构建，来创建可扩展和健壮的系统。
