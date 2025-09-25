# 第一章：范围、进度和序列

在本章中，我们将介绍以下食谱：

+   探索使用范围表达式遍历字母字符

+   使用自定义步长值的进度遍历范围

+   构建自定义进度来遍历日期

+   使用范围表达式与流程控制语句

+   发现序列的概念

+   将序列应用于解决算法问题

# 简介

本章将重点解释**范围表达式**和**序列**的优势。Kotlin 标准库提供的这些强大的数据结构概念可以帮助您提高代码的质量和可读性，以及其安全性和性能。范围表达式提供了一种声明性的方式，通过`for`循环迭代可比较类型的集合。它们对于实现简洁和安全的控制流语句和条件也非常有用。`Sequence`类作为`Collection`类型的补充，提供了其元素的内置惰性求值。在许多情况下，使用序列可以帮助优化数据处理操作，并在计算复杂度和内存消耗方面使代码更加高效。本章涵盖的食谱将专注于解决现实生活中的编程问题。此外，同时，它们还将解释这些概念在底层是如何工作的。

# 探索使用范围表达式遍历字母字符

Kotlin 标准库提供的范围是一个强大的解决方案，用于以自然和安全的方式实现迭代和条件语句。范围可以被理解为一种抽象数据类型，它表示一组可迭代的元素，并允许以声明性的方式遍历它们。来自`kotlin.ranges`包的`ClosedRange`接口是范围数据结构的基本模型。它包含对范围首尾元素的引用，并提供`contains(value: T): Boolean`和`isEmpty(): Boolean`函数，这些函数负责检查指定元素是否属于范围以及范围是否为空。在本食谱中，我们将学习如何声明一个由字母字符组成的范围，并以降序遍历它。

# 准备工作

Kotlin 标准库提供了允许声明整数、原始类型（如`Int`、`Long`和`Char`）范围的函数。要定义一个新的范围实例，我们可以使用`rangeTo()`函数。例如，我们可以以下这种方式声明一个从`0`到`1000`的整数范围：

```kt
val range: IntRange = 0.rangeTo(1000)
```

`rangeTo()`函数也有其自己的特殊运算符等价物，即`..`，它允许使用更自然的语法声明一个范围：

```kt
val range: IntRange = 0..1000
```

此外，为了以降序声明一系列元素，我们可以使用`downTo()`函数。

# 如何做...

1.  声明字母字符的降序范围：

```kt
'Z' downTo 'A'
```

2. 创建一个`for`循环来遍历范围：

```kt
for (letter in 'Z' downTo 'A') print(letter)
```

# 它是如何工作的...

因此，我们将得到以下代码打印到控制台：

```kt
ZYXWVUTSRQPONMLKJIHGFEDCBA
```

如你所见，还有`downTo()`扩展函数变体用于`Char`类型。我们使用它来创建从`Z`到`A`的字符范围。请注意，多亏了内联表示法，我们可以在调用函数时省略括号——`'Z' downTo 'A'`。

接下来，我们创建了一个`for`循环，它遍历范围并打印出后续的`Char`元素。使用`in`运算符，我们指定了循环中正在迭代的对象——就是这样！正如你所见，Kotlin 中`for`循环的语法整洁且易于使用。

原始类型范围实现，如`IntRange`、`LongRange`和`CharRange`，在底层也包含`Iterator`接口实现。在底层使用`for`循环遍历范围时，它们被使用。实际上，实现`Iterable`接口的范围被称为**进度**。在底层，`IntRange`、`LongRange`和`CharRange`类从`IntProgression`、`LongProgression`和`CharProgression`基类继承，并内部提供`Iterator`接口的实现。

# 更多内容...

还有一个方便的方法可以反转已定义进度的顺序。我们可以使用为`IntProgression`、`LongProgression`和`CharProgression`类型提供的扩展函数`reversed()`来实现这一点。它返回具有元素顺序反转的新进度实例。以下是如何使用`reversed()`函数的示例：

```kt
val daysOfYear: IntRange = 1..365
for(day in daysOfYear.reversed()) {
    println("Remaining days: $day")
}
```

前面的`for`循环将以下文本打印到控制台：

```kt
Remaining days: 365
Remaining days: 364
Remaining days: 363
…
Remaining days: 2
Remaining days: 1
```

Kotlin 标准库还提供了一个名为`until()`的便捷扩展函数，它允许声明不包含最后一个元素的范围。当与包含内部集合且不提供优雅接口访问它们的类一起工作时，这非常有用。一个很好的例子是 Android 的`ViewGroup`类，它是一个用于子`View`类型对象的容器。以下示例展示了如何遍历任何给定`ViewGroup`实例子元素的下一个索引，以修改每个子元素的状态：

```kt
val container: ViewGroup = activity.findViewById(R.id.container) as ViewGroup
(0 until container.childCount).forEach {
    val child: View = container.getChildAt(it)
    child.visibility = View.INVISIBLE
}
```

`until()`内联函数有助于使循环条件清晰且易于理解。

# 参见

+   这个配方让我们了解了 Kotlin 标准库对原始类型范围实现的易用性。如果我们想使用`for`循环遍历非原始类型，可能会出现一个问题。然而，我们很容易为任何`Comparable`类型声明一个范围。这将在*构建自定义进度以遍历日期*配方中展示。

+   正如你所注意到的，我们使用`in`运算符来指定循环中正在迭代的对象。然而，还有其他场景中，`in`和`!in`运算符可以与范围一起使用。我们将在*使用范围表达式与流程控制语句*菜谱中深入探讨它们。

# 使用自定义步长值遍历范围

除了对`Iterator`实例这样做之外，对于整型类型（如`Int`、`Long`和`Char`类型）的进度实现也包括`step`属性。`step`值指定了范围后续元素之间的间隔。默认情况下，进度的`step`值等于`1`。在本例中，我们将学习如何使用`step`值等于`2`遍历字母字符的范围。在结果中，我们希望每第二个字母字符被打印到控制台。

# 准备工作

Kotlin 标准库提供了一个方便的方法来创建具有自定义`step`值的进度。我们可以使用名为`step()`的进度整型类型的扩展函数来实现这一点。我们还可以利用中缀表示法，如下声明具有自定义`step`的进度：

```kt
val progression: IntProgression = 0..1000 step 100
```

如果我们在`for`循环中使用`progression`，它将迭代 10 次：

```kt
val progression: IntProgression = 0..1000 step 100
for (i in progression) {
    println(i)
}
```

我们也可以通过以下方式使用`while`循环达到相同的结果：

```kt
var i = 0
while (i <= 1000) {
    println(i)
    i += 100
}
```

# 如何操作...

1.  使用`downTo()`函数声明一个`Char`类型的范围：

```kt
'z' downTo 'a'
```

1.  使用`step()`函数将范围转换为具有自定义`step`值的进度：

```kt
'z' downTo 'a' step 2
```

1.  使用`forEach()`函数遍历进度的元素，并将每个元素打印到控制台：

```kt
('z' downTo 'a' step 2).forEach { character -> print(character) }
```

# 它是如何工作的...

在结果中，我们将得到以下代码打印到控制台：

```kt
zxvtrpnljhfdb
```

在开始时，我们使用`downTo()`函数声明了一个包含所有字母字符的降序范围的变量。然后，我们使用`step()`函数将范围转换为包含每个第二个字符的自定义进度。最后，我们使用`Iterable.forEach()`函数遍历进度的下一个元素，并将每个元素打印到控制台。

`step()`扩展函数适用于`IntProgression`、`LongProgression`和`CharProgression`类型。在内部，它创建一个进度的新实例，复制原始进度的属性，并设置新的`step`值。

# 参见

+   除了迭代之外，范围表达式对于定义流程控制条件也很有用。你可以在*使用范围表达式与流程控制语句*菜谱中了解更多信息。

# 构建用于遍历日期的自定义进度

Kotlin 为原始类型的范围提供了内置支持。在之前的菜谱中，我们使用了`IntRange`和`CharRange`类型，这些类型包含在 Kotlin 标准库中。然而，通过实现`Comparable`接口，我们可以为任何类型实现自定义的递增。在这个菜谱中，我们将学习如何创建`LocalDate`类型的递增，并了解如何轻松地遍历日期。

# 准备工作

为了完成任务，我们首先需要熟悉`ClosedRange`和`Iterator`接口。我们需要使用它们来为`LocalDate`类声明一个自定义的递增：

```kt
public interface ClosedRange<T: Comparable<T>> {
    public val start: T
    public val endInclusive: T
    public operator fun contains(value: T): Boolean {
        return value >= start && value <= endInclusive
    }                                      
    public fun isEmpty(): Boolean = start > endInclusive
}
```

`Iterator`接口提供了关于后续值及其可用性的信息：

```kt
public interface Iterator<out T> {
    public operator fun next(): T
    public operator fun hasNext(): Boolean
}
```

`ClosedRange`接口提供了范围的最低值和最高值。它还提供了`contains(value: T): Boolean`和`isEmpty(): Boolean`函数，分别检查给定值是否属于范围以及范围是否为空。这两个函数在`ClosedRange`接口中提供了默认实现。因此，我们不需要在我们的自定义`ClosedRange`接口实现中重写它们。

# 如何实现...

1.  让我们从为`LocalDate`类型实现`Iterator`接口开始。我们将创建一个自定义的`LocalDateIterator`类，该类将实现`Iterator<LocalDate>`接口：

```kt
class DateIterator(startDate: LocalDate,
                   val endDateInclusive: LocalDate,
                   val stepDays: Long) : Iterator<LocalDate> {
    private var currentDate = startDate
    override fun hasNext() = currentDate <= endDateInclusive
    override fun next(): LocalDate {
        val next = currentDate
        currentDate = currentDate.plusDays(stepDays)
        return next
    }
}
```

1.  现在，我们可以实现`LocalDate`类型的递增。让我们创建一个新的类，称为`DateProgression`，它将实现`Iterable<LocalDate>`和`ClosedRange<LocalDate>`接口：

```kt
class DateProgression(override val start: LocalDate,
                      override val endInclusive: LocalDate,
                      val stepDays: Long = 1) : 
                                                Iterable<LocalDate>, 
                                                ClosedRange<LocalDate> {
    override fun iterator(): Iterator<LocalDate> {
        return DateIterator(start, endInclusive, stepDays)
    } 

    infix fun step(days: Long) = DateProgression(start, endInclusive, days)
}
```

1.  最后，为`LocalDate`类声明一个自定义的`rangeTo`操作符：

```kt
operator fun LocalDate.rangeTo(other: LocalDate) = DateProgression(this, other)
```

# 工作原理...

现在，我们能够为`LocalDate`类型声明范围表达式。让我们看看如何使用我们的实现。在下面的示例中，我们将使用我们自定义的`LocalDate.rangeTo`操作符实现来创建一个日期范围并迭代其元素：

```kt
val startDate = LocalDate.of(2020, 1, 1)
val endDate = LocalDate.of(2020, 12, 31)
for (date in startDate..endDate step 7) {
    println("${date.dayOfWeek} $date ")
}
```

因此，我们将以一周为间隔将日期打印到控制台：

```kt
WEDNESDAY 2020-01-01
WEDNESDAY 2020-01-08
WEDNESDAY 2020-01-15
WEDNESDAY 2020-01-22
WEDNESDAY 2020-01-29
WEDNESDAY 2020-02-05
...
WEDNESDAY 2020-12-16
WEDNESDAY 2020-12-23
WEDNESDAY 2020-12-30
```

`DateIterator`类包含三个属性——`currentDate: LocalDate`、`endDateInclusive: LocalDate`和`stepDays: Long`。一开始，`currentDate`属性使用构造函数中传入的`startDate`值初始化。在`next()`函数内部，我们返回`currentDate`值，并使用给定的`stepDays`属性间隔将其更新到下一个日期值。

`DateProgression` 类结合了 `Iterable<LocalDate>` 和 `ClosedRange<LocalDate>` 接口的特性。它通过返回 `DateIterator` 实例来提供 `Iterable` 接口所需的 `Iterator` 对象。它还重写了 `ClosedRange` 接口的 `start` 和 `endInclusive` 属性。还有一个默认值为 `1` 的 `stepDays` 属性。请注意，每次调用 `step` 函数时，都会创建一个新的 `DateProgression` 类实例。

您可以使用相同的模式为任何实现 `Comparable` 接口的类实现自定义序列。

# 使用范围表达式与流程控制语句

除了迭代之外，当涉及到使用流程控制语句时，Kotlin 范围表达式可能非常有用。在本食谱中，我们将学习如何将范围表达式与 `if` 和 `when` 语句一起使用，以优化代码并使其更安全。在本食谱中，我们将考虑使用 `in` 操作符来定义 `if` 语句条件的示例。

# 准备工作

Kotlin 范围表达式——由 `ClosedRange` 接口表示——实现了一个 `contains(value: T): Boolean` 函数，该函数返回一个信息，表明给定的参数是否属于该范围。这个特性使得将范围与控制流指令一起使用变得方便。`contains()` 函数也有其等价的操作符 `in` 和其否定 `!in`。

# 如何操作...

1.  让我们创建一个变量，并给它分配一个随机整数值：

```kt
val randomInt = Random().nextInt()
```

1.  现在，我们可以使用范围表达式检查 `randomInt` 值是否属于从 `0` 到 `10`（包括 `10`）的整数范围：

```kt
if (randomInt in 0..10) {
    print("$randomInt belongs to <0, 10> range")
} else {
    print("$randomInt doesn't belong to <0, 10> range")
}
```

# 工作原理...

我们已经使用范围表达式与 `in` 操作符一起定义了 `if` 语句的条件。条件语句易于阅读且简洁。相比之下，等效的经典实现可能看起来像这样：

```kt
val randomInt = Random(20).nextInt()
if (randomInt >= 0 && randomInt <= 10) {
    print("$randomInt belongs to <0, 10> range")
} else {
    print("$randomInt doesn't belong to <0, 10> range")
}
```

毫无疑问，使用范围和 `in` 操作符的声明式方法比经典的命令式条件语句更简洁、更容易阅读。

# 更多内容...

范围表达式还可以增强 `when` 表达式的使用。在下面的示例中，我们将实现一个简单的函数，该函数将负责将学生的考试成绩映射到相应的等级。假设我们有以下用于学生等级的枚举类模型：

```kt
enum class Grade { A, B, C, D }
```

我们可以定义一个函数，将考试分数值（在 `0` 到 `100` % 范围内）映射到适当的等级（`A`、`B`、`C` 或 `D`），使用 `when` 表达式，如下所示：

```kt
fun computeGrade(score: Int): Grade =
        when (score) {
            in 90..100 -> Grade.A
            in 75 until 90 -> Grade.B
            in 60 until 75 -> Grade.C
            in 0 until 60 -> Grade.D
            else -> throw IllegalStateException("Wrong score value!")
        }
```

使用范围与 `in` 操作符一起使用，使得 `computeGrade()` 函数的实现比使用传统的比较操作符（如 `<`、`>`、`<=` 和 `>=`）的经典等效实现更简洁、更自然。

# 相关内容

+   如果你想了解更多关于 lambda 表达式、内联表示法和运算符重载的信息，请继续阅读第二章，*表达性函数和可调整接口*。

# 发现序列的概念

在高级功能方面，`Sequence`和`Collection`数据结构几乎相同。它们都允许遍历它们的元素。Kotlin 标准库中还有许多强大的扩展函数，为每个数据结构提供声明式数据处理的操作。然而，`Sequence`数据结构在底层的行为不同——它延迟对其元素的任何操作，直到它们最终被消费。它在遍历它们的同时实例化后续元素。`Sequence`的这些特性，称为**延迟评估**，使这种数据结构与 Java 概念`Stream`非常相似。为了更好地理解这一切，我们将实现一个简单的数据处理场景来分析`Sequence`的效率和行为，并将我们的发现与基于`Collection`的实现进行对比。

# 准备工作

让我们考虑以下示例：

```kt
val collection = listOf("a", "b", "c", "d", "e", "f", "g", "h")
val transformedCollection = collection.map {
    println("Applying map function for $it")
    it
}
println(transformedCollection.take(2))
```

在第一行，我们创建了一个字符串列表并将其分配给`collection`变量。接下来，我们正在将`map()`函数应用于列表。映射操作允许我们转换集合中的每个元素，并返回一个新值而不是原始值。在我们的例子中，我们只是用它来观察是否调用了`map()`，通过向控制台打印消息。最后，我们想要使用`take()`函数过滤我们的集合，只包含前两个元素，并将列表的内容打印到控制台。

最后，前面的代码将打印以下输出：

```kt
Applying map function for a
Applying map function for b
Applying map function for c
Applying map function for d
Applying map function for e
Applying map function for f
Applying map function for g
Applying map function for h
[a, b]
```

如您所见，`map()`函数已正确应用于集合的每个元素，而`take()`函数已正确过滤了列表中的元素。然而，如果我们处理的是更大的数据集，这并不是一个最优的实现。我们最好等到我们知道数据集的哪些特定元素是我们真正需要的，然后再对这些元素应用这些操作。结果证明，我们可以很容易地使用`Sequence`数据结构来优化我们的场景。让我们在下一节中探讨如何实现它。

# 如何实现...

1.  为给定元素声明一个`Sequence`实例：

```kt
val sequence = sequenceOf("a", "b", "c", "d", "e", "f", "g", "h")
```

1.  将映射操作应用于序列的元素：

```kt
val sequence = sequenceOf("a", "b", "c", "d", "e", "f", "g", "h")
val transformedSequence = sequence.map {
    println("Applying map function for $it")
 it
} 
```

1.  将序列的前两个元素打印到控制台：

```kt
val sequence = sequenceOf("a", "b", "c", "d", "e", "f", "g", "h")

val transformedSequence = sequence.map {
    println("Applying map function for $it")
    it
}
println(transformedSequence.take(2).toList())
```

# 它是如何工作的...

基于序列（`Sequence`）的实现将给出以下输出：

```kt
Applying map function for a
Applying map function for b
[a, b]
```

如您所见，用`Sequence`类型替换`Collection`数据结构使我们能够获得所需的优化。

在本食谱中考虑的场景被实现得完全相同——首先使用`List`，然后使用`Sequence`类型。然而，我们可以注意到`Sequence`数据结构与`Collection`的行为差异。`map()`函数仅应用于序列的前两个元素，尽管在映射转换声明之后调用了`take()`函数。还值得注意的是，在使用`Collection`的示例中，当调用`map()`函数时，映射是立即执行的。在`Sequence`的情况下，映射是在将元素打印到控制台时进行的，更确切地说，是在将`Sequence`转换为以下代码行中的`List`类型时进行的：

```kt
println(transformedSequence.take(2).toList())
```

# 更多...

将`Collection`转换为`Sequence`有一种方便的方法。我们可以使用 Kotlin 标准库为`Iterable`类型提供的`asSequence()`扩展函数来完成此操作。要将`Sequence`实例转换为`Collection`实例，您可以使用`toList()`函数。

# 参见

+   多亏了`Sequence`的延迟求值特性，我们避免了不必要的计算，同时提高了代码的性能。延迟求值允许实现具有无限元素数量的序列，并且在实现算法时也表现出有效性。

+   你可以在*将序列应用于解决算法问题*食谱中探索基于`Sequence`的斐波那契算法的实现。它更详细地介绍了另一个用于定义序列的有用函数，称为`generateSequence()`。

# 将序列应用于解决算法问题

在本食谱中，我们将熟悉`generateSequence()`函数，该函数提供了一种轻松定义各种类型序列的方法。我们将使用它来实现生成斐波那契数的算法。

# 准备工作

`generateSequence()`函数的基本变体声明如下：

```kt
fun <T : Any> generateSequence(nextFunction: () -> T?): Sequence<T>
```

它接受一个名为`nextFunction`的参数，该参数是一个返回序列下一个元素的函数。在底层，它通过`Sequence`类内部实现中的`Iterator.next()`函数被调用，并允许在消耗序列值的同时实例化下一个要返回的对象。

在以下示例中，我们将实现一个有限序列，该序列从`10`到`0`发出整数：

```kt
var counter = 10
val sequence: Sequence<Int> = generateSequence {
    counter--.takeIf { value: Int -> value >= 0 }
}
print(sequence.toList())
```

应用到当前`counter`值的`takeIf()`函数检查其值是否大于或等于`0`。如果条件得到满足，它返回`counter`值；否则，它返回`null`。每当`generateSequence()`函数返回`null`时，序列停止。在`takeIf`函数返回值后，`counter`值将进行后递减。前面的代码将导致以下数字被打印到控制台：

```kt
[10, 9, 8, 7, 6, 5, 4, 3, 2, 1, 0]
```

斐波那契数列的后续值是通过将它们的两个前一个值相加生成的。此外，前两个值等于 `0` 和 `1`。为了实现这样一个序列，我们将使用一个带有额外 `seed` 参数的 `generateSequence()` 函数的扩展版本，声明如下：

```kt
fun <T : Any> generateSequence(seed: T?, nextFunction: (T) -> T?): Sequence<T>
```

# 如何做到这一点...

1.  声明一个名为 `fibonacci()` 的函数，并使用 `generateSequence()` 函数定义序列的下一个元素的公式：

```kt
fun fibonacci(): Sequence<Int> {
    return generateSequence(Pair(0, 1)) { Pair(it.second, it.first + it.second) }
            .map { it.first }
}
```

1.  使用 `fibonacci()` 函数将下一个斐波那契数打印到控制台：

```kt
println(fibonacci().take(20).toList())
```

# 它是如何工作的...

因此，我们将得到下一个 20 个斐波那契数打印到控制台：

```kt
[0, 1, 1, 2, 3, 5, 8, 13, 21, 34, 55, 89, 144, 233, 377, 610, 987, 1597, 2584, 4181]
```

在 `generateSequence()` 中的额外 `seed` 参数提供了一个起始值。在计算第二个值时，将 `nextFunction()` 函数应用于 `seed`。随后，它使用前一个值生成每个后续元素。然而，在斐波那契数列的情况下，我们有两个初始值，我们需要一对前一个值来计算下一个值。因此，我们将它们包装在 `Pair` 类型实例中。基本上，我们正在定义一个 `Pair<Int, Int>` 类型元素的序列，并在每次 `nextFunction()` 调用中返回一个新的对，它包含相应更新的值。最后，我们只需要使用 `map()` 函数将每个 `Pair` 元素替换为其 `first` 属性的值。因此，我们得到一个无限序列，返回后续的斐波那契数。
