# 第六章：集合框架

本章将涵盖以下食谱：

+   如何合并两个集合

+   将原始集合拆分为一对集合

+   按指定比较器对列表进行排序

+   降序排序

+   使用 Gson 解析 JSON 响应

+   如何使用 lambda 表达式进行过滤和映射

+   如何对对象列表进行排序并保持 null 对象在末尾

+   如何在 Kotlin 中实现一个懒列表

+   如何在 Kotlin 中填充字符串

+   如何展平数组或映射

+   如何在 Kotlin 中按多个字段对集合进行排序

+   如何在 Kotlin 列表中使用 `limit`

+   如何在 Kotlin 中创建二维数组

+   如何在 Kotlin 中跳过前 N 个条目

# 简介

当我们想要处理集合中的项目时，集合框架非常有用。如果你使用过 Java，你可能熟悉集合框架。集合框架最常见的使用是映射、集合、列表等。Kotlin 也有自己的集合框架，但它比 Java 的集合框架更好，因为在 Kotlin 中，我们可以利用函数式编程方法使我们的代码更加简洁且易于使用。因此，让我们深入了解与 Kotlin 集合框架相关的食谱。

# 如何合并两个集合

在这个食谱中，我们将看到如何将两个或多个集合合并为一个。然而，在我们继续之前，我们需要了解可变类型和不可变类型之间的区别。不可变类型对象是一个不能被改变的对象。例如，如果我们定义一个不可变列表，我们就无法向其中添加其他对象。考虑到这一点，让我们开始这个食谱！

# 准备工作

我将使用 IntelliJ IDEA 进行编码。只要它能够编译和运行 Kotlin 代码，你可以使用你喜欢的任何 IDE。

# 如何做到这一点...

你可以使用 `listOf` 方法在 Kotlin 中创建一个列表。然而，此方法返回的列表是一个不可变列表，因此我们需要创建一个可变列表以便向其中添加对象。让我们查看提到的步骤：

1.  让我们创建两个列表，`listA` 和 `listB`，如下所示：

```kt
var listA= mutableListOf<String>("a","a","b")
var listB= mutableListOf<String>("a","c")
```

如果类型声明是从 `listOf/mutableListOf` 方法内的对象中推断出来的，我们就不需要显式地声明类型声明。因此，前面的代码将被重写为 `mutableListOf("a","a","b")`。

1.  现在，我们将尝试将 `listA` 的内容添加到 `listB` 中。为此，我们将需要 `addAll()` 方法：

```kt
fun main(args: Array<String>) {
    val listA= mutableListOf<String>("a","a","b")
    val listB= mutableListOf<String>("a","c")
    listB.addAll(listA)
    println(listB)
}
```

这是输出结果：

```kt
[a, c, a, a, b]
```

1.  合并两个列表的另一种方法是使用 `union`。这返回组合集合的唯一元素：

```kt
fun main(args: Array<String>) {
    val listA= mutableListOf<String>("a","a","b")
    val listB= mutableListOf<String>("a","c")
    val listC=listB.union(listA)
    println(listC)
}
```

这是输出结果：

```kt
[a, c, b]
```

1.  同样，可变集合也可以合并，唯一的区别是集合中的 `addAll` 将类似于我们使用 `union` 方法将获得的结果；由于它是一个集合，只允许唯一值：

```kt
val setA= mutableSetOf<String>("a","b","c")
val setB= mutableSetOf<String>("a","b","c","d")
setB.addAll(setA)
println(setB)
println(setB.union(setA))
```

这是输出结果：

```kt
[a, b, c, d]
[a, b, c, d]
```

如果你想要合并两个映射，你需要 `putAll()` 方法，因为 `addAll` 和 `union` 对于 `map` 来说是不存在的：

```kt
val mapA= mutableMapOf<String,Int>("a" to 1, "b" to 2)
val mapB= mutableMapOf<String,Int>("a" to 2, "d" to 4)
mapA.putAll(mapB)
println(mapA)
```

这是输出结果：

```kt
{a=2, b=2, d=4}
```

注意，键 `a` 在两个映射中都定义了，但后来出现的那个（在这种情况下，`mapB`）是获胜者。

# 将原始集合拆分为一对集合

有时候，您可能希望只需将列表拆分为子列表，而不需要进入`for`和`while`循环。Kotlin 为您提供了专门为此目的的函数。在本菜谱中，我们将了解如何根据某些标准拆分列表。

# 准备工作

我将使用 IntelliJ IDEA 来编写和运行 Kotlin 代码；您可以使用任何能够完成相同任务的 IDE。

# 如何实现它...

Kotlin 提供了一个`partition`函数。根据`partition`函数的文档，它执行以下操作：

将原始数组拆分为一对列表，其中第一个列表包含谓词返回 true 的元素，而第二个列表包含谓词返回 false 的元素。

让我们通过这个示例更清楚地理解它：

1.  在此示例中，我们将创建一个数字列表，并希望将此列表拆分为两个子列表：一个包含奇数，另一个包含偶数：

```kt
fun main(args: Array<String>) {
    val listA= listOf(1,2,3,4,5,6)
    val pair=listA.partition {
        it%2==0
    }
    println(pair)
}
```

这是输出：

```kt
([2, 4, 6], [1, 3, 5])
```

1.  如前一个示例所示，我们需要在`partition`块中将条件放入谓词中。返回的对象是一个`Pair`对象，包含两个子列表。

1.  `partition`函数也可以以类似的方式与`set`集合一起使用：

```kt
val setA= setOf(1,2,3,4,5,6)
val pair=setA.partition {
    it%2==0
}
println(pair)

```

这里是输出：

```kt
([2, 4, 6], [1, 3, 5])
```

# 它是如何工作的...

让我们看看 Kotlin 中`partition`函数的实现：

```kt
public inline fun <T> Iterable<T>.partition(predicate: (T) -> Boolean): Pair<List<T>, List<T>> {
    val first = ArrayList<T>()
    val second = ArrayList<T>()
    for (element in this) {
        if (predicate(element)) {
            first.add(element)
        } else {
            second.add(element)
        }
    }
    return Pair(first, second)
}
```

如您所见，`partition`函数只是一个抽象，它可以帮助您避免编写冗长的循环，但内部它仍然以相同的方式执行。

# 还有更多...

`partition`函数与数组一起使用时也以类似的方式工作。以下是它的不同用法。每个用法都类似，只是产生不同类型的列表：

```kt
// Produces two lists
inline fun <T> Array<out T>.partition(
    predicate: (T) -> Boolean
): Pair<List<T>, List<T>>
```

```kt
// Breaks original list of Byte and produces two lists of Byte
inline fun ByteArray.partition(
    predicate: (Byte) -> Boolean
): Pair<List<Byte>, List<Byte>>

```

```kt
// Breaks original list of Short and produces two lists of Short
inline fun ShortArray.partition(
    predicate: (Short) -> Boolean
): Pair<List<Short>, List<Short>>
```

```kt
// Breaks original list of Int and produces two lists of Int
inline fun IntArray.partition(
    predicate: (Int) -> Boolean
): Pair<List<Int>, List<Int>>
```

```kt
// Breaks original list of Long and produces two lists of Long
inline fun LongArray.partition(
    predicate: (Long) -> Boolean
): Pair<List<Long>, List<Long>>
```

```kt
// Breaks original list of Float and produces two lists of Float
inline fun FloatArray.partition(
    predicate: (Float) -> Boolean
): Pair<List<Float>, List<Float>>
```

```kt
// Breaks original list of Double and produces two lists of Double
inline fun DoubleArray.partition(
    predicate: (Double) -> Boolean
): Pair<List<Double>, List<Double>>
```

```kt
// Breaks original list of Boolean and produces two lists of Boolean
inline fun BooleanArray.partition(
    predicate: (Boolean) -> Boolean
): Pair<List<Boolean>, List<Boolean>>
```

```kt
// Breaks original list of Char and produces two lists of Char
inline fun CharArray.partition(
    predicate: (Char) -> Boolean
): Pair<List<Char>, List<Char>>
```

# 根据指定的比较器对列表进行排序

对列表进行排序是列表上执行的最常见操作之一。当我们尝试对自定义对象列表进行排序时，我们需要指定比较器。让我们看看我们如何根据指定比较器对列表进行排序。

# 准备工作

我将使用 IntelliJ IDEA 来编写和运行 Kotlin 代码；您可以使用任何能够完成相同任务的 IDE。

# 如何实现它...

在以下示例中，我们将尝试根据某些属性对对象进行排序。这将给我们一个关于如何根据指定比较器进行排序的思路：

1.  让我们创建一个具有年龄属性的`Person`类。我们将根据年龄对人员对象列表进行排序：

```kt
fun main(args: Array<String>) {
    val p1=Person(91)
    val p2=Person(10)
    val p3=Person(78)
    val listOfPerson= listOf(p1,p2,p3)
    var sortedListOfPerson=listOfPerson.sortedBy {
        it.age
    }
}
class Person(var age:Int)
```

1.  要根据指定的比较器对列表进行排序，我们需要使用`sortedBy`函数：

```kt
fun main(args: Array<String>) {
    val p1=Person(91)
    val p2=Person(10)
    val p3=Person(78)
    val listOfPerson= listOf(p1,p2,p3)
    var sortedListOfPerson=listOfPerson.sortedBy {
        it.age
    }
}
class Person(var age:Int)

```

1.  Kotlin 还提供了一个`sortedWith`方法，您可以在其中指定自己的比较器实现：

```kt
fun main(args: Array<String>)
{
  val p1=Person(91)
  val p2=Person(10)
  val p3=Person(78)
  val listOfPerson= listOf(p1,p2,p3)
  var sortedListOfPerson=listOfPerson
  .sortedWith<Person>(object:Comparator<Person>{
      override fun compare(p0: Person, p1: Person):Int {
        if(p0.age>p1.age){
              return 1
          }
          if(p0.age==p1.age){
              return 0
          }
          return -1
      }
  })
}
class Person(var age:Int)
```

# 它是如何工作的...

`sortedBy`函数是 Kotlin 提供的语法糖。内部，它调用接受比较器的`sortedWith`方法。

现在，让我们看看`sortBy`函数的实现：

```kt
public inline fun <T, R : Comparable<R>> Iterable<T>.sortedBy(crossinline selector: (T) -> R?): List<T> {
    return sortedWith(compareBy(selector))
}
```

`sortBy`函数在其内部调用`sortedWith`方法，如下所示：*

```kt
public fun <T> Iterable<T>.sortedWith(comparator: Comparator<in T>): List<T> {
    if (this is Collection) {
       if (size <= 1) return this.toList()
       @Suppress("UNCHECKED_CAST")
       return (toTypedArray<Any?>() as Array<T>).apply { sortWith(comparator) }.asList()
    }
    return toMutableList().apply { sortWith(comparator) }
}
```

# 降序排序

在上一个菜谱中，我们看到了如何使用指定的比较器对列表进行排序。我们提供比较器，然后它按相应的方式排序。有趣的是，Kotlin 还提供了一个方法来按降序对列表中的项目进行排序。在这个菜谱中，我们将看到如何按降序对原始对象以及自定义对象进行排序。所以，让我们开始吧！

# 准备工作

我将使用 IntelliJ IDEA 来编写和运行 Kotlin 代码；您可以使用任何可以完成相同任务的 IDE。

# 如何做…

现在，我们将通过一些示例来了解如何使用降序排序：

1.  首先，我们将尝试对一个简单的整数列表进行排序：

```kt
val listOfInt= listOf(1,2,3,4,5)
var sortedList=listOfInt.sortedDescending()
sortedList.forEach {
    print("${it} ")
}
```

这是输出：

```kt
5 4 3 2 1
```

1.  现在，让我们使用前面菜谱中的`Person`列表。为了按降序排序，我们将这样做：

```kt
val p1=Person(91)
val p2=Person(10)
val p3=Person(78)
val listOfPerson= listOf<Person>(p1,p2,p3)
val sortedListOfPerson=listOfPerson.sortedByDescending {
    it.age
}
sortedListOfPerson.forEach {
    print("${it.age} ")
}
```

这里是输出：

```kt
91 78 10
```

# 工作原理…

`sortedByDescending`的工作方式有点像`sortedBy`。内部，两者都使用`sortedWith`函数：

```kt
public inline fun <T, R : Comparable<R>> Iterable<T>.sortedByDescending(crossinline selector: (T) -> R?): List<T> {
    return sortedWith(compareByDescending(selector))
}
```

以下是对`compareByDescending`的实现：

```kt
@kotlin.internal.InlineOnly
public inline fun <T> compareByDescending(crossinline selector: (T) -> Comparable<*>?): Comparator<T> =
        Comparator { a, b -> compareValuesBy(b, a, selector) }
```

注意，只需反转变量的顺序即可产生降序。

# 使用 Gson 解析 JSON 响应

在这个菜谱中，我们将学习如何解析 JSON。JSON 是 API 响应中最广泛使用的数据类型。我们将使用 Google 的开源库 Gson。它速度快，即使响应很大也能很好地扩展。

# 准备工作

我将使用 Android Studio 来完成这个任务，JSONObject 由 Android SDK 提供。我们将使用 Gson 进行 JSON 解析。您可以通过将以下行添加到您的`build.gradle`文件中来将其添加到您的项目中：

```kt
compile 'com.google.code.gson:gson:2.8.0'
```

# 如何做…

现在，让我们按照以下步骤使用 Gson 解析 JSON 数据。例如，我们将在这里使用一个原始字符串来保持事情简单：

1.  首先，我们将使用以下方式创建一个模拟的 JSON 数据，使用原始字符串：

```kt
val jsonStr="""
    {
     "name": "Aanand Shekhar",
     "age": 21,
     "isAwesome": true
    }
""".trimIndent()
```

1.  接下来，我们将创建一个数据类来保存这些数据。以下是我们的数据类看起来像这样：

```kt
data class Information(val name:String,val age:Int, val isAwesome:Boolean)
```

1.  最后，我们将使用`Gson`来解析 JSON 字符串：

```kt
val information:Information= Gson().fromJson<Information>(jsonStr,Information::class.java)
```

现在，您可以使用它就像一个 Kotlin 对象一样。

# 更多内容…

您可以使用一些 Android Studio 插件自动创建数据类。最广泛使用的插件之一是**RoboPOJOGenerator** ([`github.com/robohorse/RoboPOJOGenerator`](https://github.com/robohorse/RoboPOJOGenerator) )。

# 如何使用 lambda 表达式进行过滤和映射

在这个菜谱中，我们将学习如何使用 Kotlin 中的`map`函数转换列表，以及如何根据我们喜欢的任何标准过滤列表。我们将使用 lambda 函数，它为函数式编程提供了一种很好的方式。所以，让我们开始吧。

# 准备工作

我将使用 IntelliJ IDEA 来编写和运行 Kotlin 代码；您可以使用任何可以完成相同任务的 IDE。

# 如何做…

首先，让我们看看如何在列表上使用**filter**函数。filter 函数返回一个包含所有匹配给定谓词的元素的列表。我们将创建一个数字列表，并根据偶数或奇数来过滤列表。

`filter`方法对于不可变集合很好，因为它不会修改原始集合，而是返回一个新的集合。在`filter`方法中，我们需要实现谓词。谓词，就像条件一样，基于被过滤的列表。

例如，我们知道偶数项将遵循`it%2==0`。因此，相应的过滤方法将如下所示：

```kt
val listOfNumbers=listOf(1,2,3,4,5,6,7,8,9)
var evenList=listOfNumbers.filter {
    it%2==0
}
println(evenList)

//Output: [2, 4, 6, 8]
```

过滤函数的另一个变体是`filterNot`，正如其名称所暗示的，它返回一个包含所有不匹配给定谓词的元素的列表。

另一个酷炫的 lambda 函数是`map`。它转换列表并返回一个新的列表：

```kt
val listOfNumbers=listOf(1,2,3,4,5,6,7,8,9)
var transformedList=listOfNumbers.map {
    it*2
}
println(transformedList)

//Output: [2, 4, 6, 8, 10, 12, 14, 16, 18]
```

`map`函数的一个变体是`mapIndexed`。它在其构造中提供了索引和项：

```kt
val listOfNumbers=listOf(1,2,3,4,5)
val map=listOfNumbers.mapIndexed { index, it
    -> it*index}
println(map)

//Output: [0, 2, 6, 12, 20]
```

# 如何对列表中的对象进行排序并保持 null 对象在末尾

我们已经看到如何使用比较器根据指定参数对列表进行排序。然而，到目前为止，我们一直在处理具有非 null 值的列表。在这个菜谱中，我们将看到如何对具有 null 属性（我们根据该属性进行排序）的对象列表进行排序。所以让我们开始吧。

# 准备工作

我将使用 IntelliJ IDEA 来编写和运行 Kotlin 代码；你可以自由使用任何可以完成相同任务的 IDE。

# 如何做到这一点…

现在，让我们遵循以下步骤来排序列表，同时保持 null 对象在末尾：

1.  让我们创建一个具有年龄属性（可以是 null）的`Person`类：

```kt
class Person(var age:Int?)
```

1.  现在，让我们创建一个`Person`对象列表：

```kt
val listOfPersons=listOf(Person(10), Person(20), Person(2), Person(null))
```

1.  最后，我们希望按升序对它们进行排序，同时保持 null 项在末尾：

```kt
val sortedList=listOfPersons.sortedWith(compareBy(nullsLast<Int>(),{it.age}))
sortedList.forEach {
    print(" ${it.age} ")
}

```

输出如下：

```kt
2 10 20 null 
```

# 它是如何工作的…

我们使用了`sortedWith`方法。根据文档，`sortedWith`是这样做的：

返回一个序列，该序列根据指定的比较器生成此序列的元素。

除了那个之外，我们还使用了`kotlin.comparisons`包，它为我们提供了在前面解决方案中使用的主要两个函数：

+   `public inline fun <T: Comparable<T>> nullsLast()`: 这个方法提供了一个比较器，用于考虑可空可比较值，其中 null 值大于任何其他值。这就是我们能够在末尾获取 null 项的原因，因为它们被认为比任何其他值都大。

+   `compareBy(comparator: Comparator<in K>, crossinline selector: (T) -> K)`: 这个函数接受一个比较器（例如`nullsLast()`）和一个为比较器提供值的函数，然后将它们组合成一个新的比较器。

# Kotlin 中实现懒列表的方法

如果一个元素或表达式的值在定义时没有被评估，而是在首次访问时才被评估，那么它被称为**延迟评估**。有许多情况它都很有用。例如，你可能有一个列表 A，你想要从它创建一个过滤后的列表，让我们称它为列表 B。如果你做如下操作，过滤操作将在 B 的声明期间执行：

```kt
val A= listOf(1,2,3,4)
var B=A.filter {
    it%2==0
}
```

这迫使程序在定义 B 时立即初始化它。虽然这对小列表来说可能不是什么大问题，但它可能导致大对象的延迟。此外，我们可以在第一次需要时再创建对象。在这个菜谱中，我们将学习如何实现一个惰性列表。

# 准备工作

我将使用 IntelliJ IDEA 来编写和运行 Kotlin 代码；您可以使用任何可以完成相同任务的 IDE。

# 如何实现它…

要创建一个惰性列表，我们需要将列表转换为序列。序列表示惰性评估的集合。让我们用一个例子来理解它：

1.  在给定的例子中，让我们首先根据元素是奇数还是偶数来过滤列表：

```kt
fun main(args: Array<String>) {
    val A= listOf(1,2,3,4)
    var B=A.filter {
        println("checking ${it}")
        it%2==0
    }
}
```

这是输出：

```kt
checking 1
checking 2
checking 3
checking 4
```

在前面的例子中，`filter` 函数仅在对象被定义时被评估。

1.  现在，让我们将列表转换为序列。将列表转换为序列只需一步之遥；您可以使用 `.asSequence()` 方法或通过 `Sequence{ createIterator() }` 将任何列表转换为序列：

```kt
fun main(args: Array<String>) {
    val A= listOf(1,2,3,4).asSequence()
    var B=A.filter {
        println("checking ${it}")
        it%2==0
    }
}
```

1.  如果您运行前面的代码，您在控制台中将看不到任何输出，因为对象尚未创建。它将在列表 B 首次访问时创建：

```kt
fun main(args: Array<String>) {
    val A= listOf(1,2,3,4).asSequence()
    var B=A.filter {
        println("checking ${it}")
        it%2==0
    }
    B.forEach {
        println("printing ${it}")
    }
}

//Output:checking 1
 checking 2
 printing 2
 checking 3
 checking 4
 printing 4
```

当访问项目时，`filter` 函数被评估。这被称为**惰性求值**。

# 它是如何工作的…

Kotlin 中的序列可能是无界的，并且当列表的长度事先未知时（类似于 Java 8 中的 Streams）会使用它。由于它可以无限大，因此需要惰性求值来处理这种类型的结构。考虑以下示例：

```kt
val seq= generateSequence(1){it*2}
seq.take(10).forEach {
    print(" ${it} ")
}
```

这里，`generateSequence` 生成一个无限数字的序列，但当我们调用 `take(10)` 时，只有 10 个项目被评估和打印。

# 如何在 Kotlin 中填充字符串

有时，为了保持字符串的长度，我们会用一些字符填充字符串。在许多通信协议中，保持有效载荷的标准长度至关重要。Kotlin 使得用任何字符和长度填充字符串变得非常容易。让我们看看如何使用它。

# 准备工作

我将使用 IntelliJ IDEA 来编写和运行 Kotlin 代码；您可以使用任何可以完成相同任务的 IDE。

# 如何实现它…

在这个菜谱中，我们将使用 Kotlin 的 `kotlin.stdlib` 库。具体来说，我们将使用 `padStart` 和 `padEnd` 函数。现在，让我们按照给定的步骤来了解如何使用这些函数：

1.  让我们看看 `padStart` 函数的一个例子：

```kt
fun main(args: Array<String>) {
    val string="abcdef"
    val pad=string.padStart(10,'-')
    println(pad)
}
```

这是输出：

```kt
 ----abcdef
```

1.  接下来，我们看看 `padEnd` 的一个例子：

```kt
val string="abcdef"
val pad=string.padEnd(10,'-')
println(pad)
```

这里是输出：

```kt
 abcdef----
```

# 它是如何工作的…

填充函数需要使用函数提供的字符将字符串扩展到一定长度。因此，如果填充后的字符串长度小于原始字符串，它将只返回相同的字符串。

另一个需要注意的关键点是，默认情况下，填充字符是空格字符。这是 `padStart` 函数的实现：

```kt
public fun String.padStart(length: Int, padChar: Char = ' '): String
        = (this as CharSequence).padStart(length, padChar).toString()
```

如您所见，`padChar` 的默认值是空格字符，并且它是在一个字符串对象上调用的。

# 如何展开数组或映射

在本章的前几个食谱中，我们学习了如何创建多维数组。在本食谱中，我们将看到如何将它们转换为 1D 列表，或 *flatten* 它们。

# 准备工作

我将使用 IntelliJ IDEA 编写和运行 Kotlin 代码；您可以使用任何可以完成相同任务的 IDE。

# 如何实现...

我们将使用 `kotlin.stdlib` 库的 `.flatten` 方法。它接受一个数组或集合，并返回一个包含给定集合/数组中所有元素的单一列表。

例如，使用数组数组：

`[[1,2,3],[1,2,3],[1,2,3]] -> [1,2,3,1,2,3,1,2,3]`

```kt
fun main(args: Array<String>) {
    val a= arrayOf(arrayOf(1,2,3),arrayOf(1,2,3),arrayOf(1,2,3))
    a.flatten().forEach { print(" ${it} ") }
}

//Output:  1 2 3 1 2 3 1 2 3 
```

例如，使用列表列表：

`[[1,2,3],[1,2,3],[1,2,3]] -> [1,2,3,1,2,3,1,2,3]`

```kt
fun main(args: Array<String>) {
    val a= listOf(listOf(1,2,3),listOf(1,2,3),listOf(1,2,3))
    a.flatten().forEach { print(" ${it} ") }
}
```

# 它是如何工作的...

让我们看看 `flatten()` 函数的实现：

```kt
public fun <T> Iterable<Iterable<T>>.flatten(): List<T> {
    val result = ArrayList<T>()
    for (element in this) {
        result.addAll(element)
    }
    return result
}
```

如您所见，这只是在新的列表中添加来自可迭代对象（数组或列表）的项目，并返回该列表。

# 如何在 Kotlin 中按多个字段排序集合

在本食谱中，我们将学习如何在 Kotlin 中按多个字段对集合进行排序。当我们想要在两个对象在特定属性上具有相等值时给予一个对象比另一个对象优先权时，这通常很有用。例如，我们可能有一个 `Student` 对象的列表，并希望按年龄升序排列它们，但如果两个学生的年龄相同，我们将根据他们的 GPA 排序。在本食谱中，我们将看到如何处理此类用例。那么，让我们开始吧！

# 准备工作

我将使用 IntelliJ IDEA 编写和运行 Kotlin 代码；您可以使用任何可以完成相同任务的 IDE。

# 如何实现...

现在，让我们按照以下步骤根据对象的多个字段进行排序：

1.  首先，让我们创建 `Student` 类：

```kt
class Student(val age:Int, val GPA: Double)
```

1.  然后，创建一个 `Student` 对象的列表：

```kt
val studentA=Student(11,2.0)
val studentB=Student(11,2.1)
val studentC=Student(11,1.3)
val studentD=Student(12,1.3)
val studentsList=listOf<Student>(studentA,studentB,studentC,studentD)
```

1.  要按多个字段排序，我们只需这样做：

```kt
val sortedList=studentsList.sortedWith(compareBy({it.age},{it.GPA}))
```

1.  如果我们现在打印它，我们将得到以下输出：

```kt
sortedList.forEach {
    println("age: ${it.age}, GPA: ${it.GPA} ")
}

//Output: age: 11, GPA: 1.3 
 age: 11, GPA: 2.0 
 age: 11, GPA: 2.1 
 age: 12, GPA: 1.3
```

# 它是如何工作的...

我们使用了 `sortedWith` 函数，它接受一个比较器。比较器由 `compareBy` 函数提供。`compareBy` 有一个可以接受多个函数的重载：

```kt
public fun <T> compareBy(vararg selectors: (T) -> Comparable<*>?):Comparator<T>
```

如您在前面代码中所见，`vararg` 允许我们在其构造函数中接受多个函数，并返回一个比较器，该比较器将数据传递给 `sortedWith` 函数。

注意，使用多个字段排序的工作方式是按字段 1 排序，然后按字段 2 排序，然后按字段 3 排序，依此类推。

# 如何在 Kotlin 列表中使用 limit

在本食谱中，我们将学习如何从列表中获取特定项。我们将为此目的使用 `kotlin.stdlib` 库。

# 准备工作

我将使用 IntelliJ IDEA 编写和运行 Kotlin 代码；您可以使用任何可以完成相同任务的 IDE。

# 如何实现...

我们将使用 `take` 函数及其变体来限制列表中的项。

`take(n)`: 返回前 n 个元素的列表：

```kt
fun main(args: Array<String>) {
    val list= listOf(1,2,3,4,5)
    val limitedList=list.take(3)
    println(limitedList)
}

//Output: [1,2,3]
```

`takeLast(n)`: 返回包含最后 [n] 个元素的列表：

```kt
fun main(args: Array<String>) {
    val list= listOf(1,2,3,4,5)
    val limitedList=list.takeLast(3)
    println(limitedList)
}

//Output: [3,4,5]
```

`takeWhile{ predicate }`: 返回包含满足给定 [谓词] 的第一个元素的列表：

```kt
val list= listOf(1,2,3,4,5)
val limitedList=list.takeWhile { it<3 }
println(limitedList)

//Output: [1,2]
```

`takeLastWhile{谓词}`：与 `takeWhile` 类似，但它从列表的末尾评估。

`takeIf { 谓词 }`：如果它满足给定的 [谓词]，则返回 `this` 值，如果不满足，则返回 `null`：

```kt
fun main(args: Array<String>) {
    val list= listOf(1,2,3,4,5)
    var limitedList=list.takeIf { it .contains(1) }
    println(limitedList)
}

//Output: [1,2,3,4,5]
```

注意，`takeIf` lambda 中的 `it` 代表列表本身，而不仅仅是列表的一个元素。

# 如何在 Kotlin 中创建二维数组

在某些情况下，如棋盘游戏、图像等，二维数组对于数据表示非常有用。在 Java 中，我们可以通过以下方式表示二维数组：

```kt
int[][] data = new int[size][size];
```

由于 Kotlin 带来了新的语法，让我们看看如何在 Kotlin 中处理二维数组。

# 准备工作

我将使用 IntelliJ IDEA 编写和运行 Kotlin 代码；你可以自由使用任何可以完成相同任务的 IDE。

# 如何操作…

现在让我们按照给定的步骤在 Kotlin 中创建一个二维数组：

1.  我们可以使用以下语法在 Kotlin 中创建一个简单的二维数组：

```kt
val array = Array(n, {IntArray(n)})
```

在这里，`n` 代表数组的维度。在这里，我们使用了 Kotlin 的 `Array` 类，它代表一个数组（特别是当针对 JVM 平台时，是 Java 数组）。我们通过传递大小和初始化器来初始化 `Array` 对象：

```kt
public inline constructor(size: Int, init: (Int) -> T)
```

1.  我们的维度是 `n`，作为初始化器，我们传递一个一维数组，然后它提供了一个二维数组的结构。如果你想要使用特定的值初始化二维数组，你需要将其传递给初始化器。考虑以下示例：

```kt
Array<IntArray>(10,{IntArray(10,{-1})})
```

1.  上述二维数组将被初始化为所有 `-1`。

1.  我们还可以使用 `arrayOf` 构造函数通过传递两个一维数组来创建一个二维数组：

```kt
val even: IntArray = intArrayOf(2, 4, 6)
val odd: IntArray = intArrayOf(1, 3, 5)

val lala: Array<IntArray> = arrayOf(even, odd)
lala.forEach {
    it.forEach {
        print(" ${it} ")
    }
    println()
}

//Output: 2 4 6 
 1 3 5 
```

# 还有更多…

你也可以通过扩展 Kotlin 的代码来创建自己的函数。例如，创建一个方法，如下所示：

```kt
inline fun <reified inside> array2d(sizeOuter: Int, sizeInner: Int, noinline innerInit: (Int)->inside): Array<Array<inside>>
       = Array(sizeOuter) { Array<inside>(sizeInner, innerInit) }
```

这可以通过以下操作轻松创建二维数组：

```kt
array2d(10,10,{0})
```

你也可以以类似的方式创建一个列表的列表。以下是一个列表的列表的示例：

```kt
fun main(args: Array<String>) {
    val a= listOf(listOf(1,2,3), listOf(4,5,6), listOf(7,8,9))
    a.forEach {
        print(" ${it} ")
    }
}

```

这是它的输出：

```kt
[1, 2, 3] [4, 5, 6] [7, 8, 9] 
```

# 如何在 Kotlin 中跳过前 "n" 个条目

在本食谱中，我们将学习如何在集合中删除条目。首先，我们将看到如何删除前 *n* 个项目，然后我们将看到如何删除最后 *n* 个项目，最后，我们将看到如何在删除集合中的元素时使用谓词。

# 准备工作

我将使用 IntelliJ IDEA 编写和运行 Kotlin 代码；你可以自由使用任何可以完成相同任务的 IDE。

# 如何操作…

在以下步骤中，我们将学习如何跳过 Kotlin 列表中的前 *n* 个条目：

1.  首先，让我们看看如何删除集合中的前 *n* 个项目。我们将使用列表，但它也可以与数组一起使用。此外，我们将使用 `kotlin.stdlib`，它包含本食谱中所需的函数。这里要使用的函数是 `drop`：

```kt
fun main(args: Array<String>) {
    val list= listOf<Int>(1,2,3,4,5,6,7,8,9)
    var droppedList=list.drop(2)
    droppedList.forEach {
        print(" ${it} ")
    }
}

//Output: 3 4 5 6 7 8 9 
```

1.  要跳过集合中的最后 *n* 个项目，你需要使用 `dropLast` 函数：

```kt
fun main(args: Array<String>) {
    val list= listOf<Int>(1,2,3,4,5,6,7,8,9)
    var droppedList=list.dropLast(2)
    droppedList.forEach {
        print(" ${it} ")
    }
}

//Output:  1 2 3 4 5 6 7 
```

1.  这个 lambda 函数在谓词返回 true 时删除项目：

```kt
val list= listOf<Int>(1,2,3,4,5,6,7,8,9,1,2,3)
val droppedList=list.dropWhile { it<3 }
droppedList.forEach {
    print(" ${it} ")
}

//Output:  3 4 5 6 7 8 9 1 2 3 
```

1.  此函数在满足条件的情况下删除末尾的项目。

```kt
fun main(args: Array<String>) {
    val list= listOf<Int&gt;(1,2,3,4,5,6,7,8,9,3,1,2)
    val droppedList=list.dropLastWhile { it<3 }
    droppedList.forEach {
        print(" ${it} ")
    }
}

//Output: 1 2 3 4 5 6 7 8 9 3 
```

# 它是如何工作的…

`drop` 函数通过跳过前 n 个元素返回一个新的列表。内部实现上，它只是使用了普通的 `for` 循环，并对输入是否为数组或列表进行了一些检查。
