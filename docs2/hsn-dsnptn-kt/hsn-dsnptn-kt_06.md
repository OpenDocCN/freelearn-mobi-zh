# 数据流

在本章中，我们将讨论集合的高级函数。对于 Java 开发者来说，它们首次出现在 Java 8 中，随着 Stream API 的引入。但在函数式语言中，它们已经存在很长时间了。

首先，由于我们预计许多读者都熟悉 Java 8，让我们简要介绍一下 Java 中的 Stream API 是什么。

Java8 中的流与一些具有相似名称的 I/O 类（如`InputStream`或`OutputStream`）不可混淆。后者表示数据，而前者是相同类型元素的序列。

如果这些是序列，并且它们都具有相同的类型，那么它们与`Lists`有何不同？嗯，流可以是无限的，而集合不是。

为 Java 流定义了一系列操作。这些操作不仅对任何类型的流都是相同的，而且对于那些来自完全不同语言的操作，它们还有熟悉的名字。例如，JavaScript 中的`map()`函数与 Java 中的`map()`方法做的是相同的事情。

大量使用小型、可重用和可组合函数的想法直接来自我们在上一章讨论的函数式编程。它们允许我们以描述*我们想要做什么*的方式编写代码，而不是*我们想要如何做*。

但在 Java 中，要使用这些函数，我们必须接收一个流或从集合创建一个流。

在 Java 中，为了获取集合的所有这些功能，我们可以做以下操作：

```kt
Arrays.asList("a", "b", "c") // Initialize list
    .stream() // Convert to stream
    .map(...) // Do something functional here   
    .toList() // Convert back to list
```

在 Kotlin 中，你可以做同样的事情：

```kt
listOf("a", "b", "c").stream().map{...}.toList()
```

但所有这些方法以及更多方法都直接在集合上可用：

```kt
listOf("a", "b", "c").map(...) 
```

那就是全部；除非您最初计划操作*无限数据*，否则不需要从流转换回集合。

当然，事情并没有那么简单，但我们在本章末尾的*流是懒的，集合不是*部分中涵盖了差异和陷阱。让我们先了解那些奇怪函数实际上做什么。

在本章中，我们无法涵盖集合上所有可用的函数，但我们将介绍最常用的那些。

示例将相对无聊，主要是数字、字母和人的列表。这样可以让您专注于每个函数的实际工作方式。我们将在下一章回到一些疯狂示例。敬请期待。

# it 表示法

我们在前几章中简要地提到了`it`的概念，但为了本章，我们需要更深入地理解它（有意为之）。

Kotlin 的一切都关于简洁。首先，如果我们的 lambda 没有参数，我们就不需要指定任何内容：

```kt
val noParameters = { 1 } // () -> Int implicitly
```

但如果我们有一个接受另一个函数作为参数（为了简单起见，我们不对其做任何操作）的函数呢？请看以下代码：

```kt
fun oneParameter(block: (Int)->Long){ }
```

我们可以明确指定参数名称和类型，并将它们放在括号中，就像调用其他任何函数一样：

```kt
val oneParameterVeryVeryExplicit = oneParameter( {x: Int -> x.toLong() })
```

但由于 lambda 是最后一个参数（在这种情况下，也是唯一的参数），我们可以省略括号：

```kt
val oneParameterVeryExplicit = oneParameter {x: Int -> x.toLong() }
```

由于编译器可以推断参数的类型，我们也可以省略它：

```kt
val oneParameterExplicit = oneParameter {x -> x.toLong() }
```

由于 `x` 是唯一的参数，我们可以使用它的隐含名称，即 `it`：

```kt
val oneParameterImplicit = oneParameter { it.toLong() }
```

在接下来的大多数示例中，我们将使用最简短的表示法。

# `map()` 函数

集合中最知名的高阶函数之一是 `map()`。

假设你有一个函数，它接收一个字符串列表并返回一个新列表，其大小与原列表相同，每个字符串都与其自身连接：

```kt
val letters = listOf("a", "b", "c", "d")

println(repeatAll(letters)) // [aa, bb, cc, dd]
```

这个任务相当简单：

```kt
fun repeatAll(letters: List<String>): MutableList<String> {
    val repeatedLetters = mutableListOf<String>()

    for (l in letters) {
        repeatedLetters.add(l + l)
    }
    return repeatedLetters
}
```

但对于这样一个简单的任务，我们不得不写很多代码。为了将每个字符串改为大写而不是重复两次，我们需要做些什么改变？我们只想改变这一行：

```kt
repeatedLetters.add(l + l) ----> repeatedLetters.add(l.toUpperCase())
```

但我们必须为这个创建另一个函数。

当然，在 Kotlin 中，我们可以将一个函数作为第二个参数传递。由于我们实际上并不关心类型是什么，只要输入和输出相同即可，我们可以使用泛型：

```kt
fun <T> repeatSomething(input: List<T>, action: (T) -> T): MutableList<T> {
    val result = mutableListOf<T>()

    for (i in input) {
        result.add(action(i))
    }
    return result
}
```

现在，我们可以这样调用我们的 *泛型化* 函数：

```kt
println(repeatSomething(letters) {
    it.toUpperCase()
})
```

这几乎就是 `.map()` 所做的：

```kt
println(letters.map {
    it.toUpperCase()
})
```

`map()` 的另一种变体是 `mapTo()`。

除了 lambda，它还接收一个目的地，结果应该被合并到那里。

你可以这样做：

```kt
val letters = listOf("a", "B", "c", "D")
val results = mutableListOf<String>()

results.addAll(letters.map {
    it.toUpperCase()
})

results.addAll(letters.map {
    it.toLowerCase()
})

println(results)
```

但 `mapTo()` 允许你这样做：

```kt
val letters = listOf("a", "B", "c", "D")
val results = mutableListOf<String>()

letters.mapTo(results) {
    it.toUpperCase()
}

letters.mapTo(results) {
    it.toLowerCase()
}

println(results)
```

在第二种情况下，我们使用结果列表作为参数，这允许我们减少代码嵌套。

# 过滤家族

另一个常见任务是过滤集合。你知道该怎么做。你遍历它，只将符合你标准的值放入新的集合中。例如，如果给定 1-10 的数字范围，我们只想返回奇数。当然，我们已经从上一个例子中学到了这个教训，不会简单地创建一个名为 `filterOdd()` 的函数，因为后来我们还需要实现 `filterEven()`、`filterPrime()` 等等。我们将立即接收到一个 lambda 作为第二个参数：

```kt
fun filter(numbers: List<Int>, check: (Int)->Boolean): MutableList<Int> {
    val result = mutableListOf<Int>()

    for (n in numbers) {
        if (check(n)) {
            result.add(n)
        }
    }

    return result
}
```

调用它将只打印奇数。多么奇怪：

```kt
println(filter((1..10).toList()) {
    it % 2 != 0
}) // [1, 3, 5, 7, 9]
```

当然，我们已经有了一个内置的函数，它正好做这件事：

```kt
println((1..10).toList().filter {
    it % 2 != 0
})
```

# 查找家族

假设你有一个无序的对象列表：

```kt
data class Person(val firstName: String, 
                  val lastName: String,
                  val age: Int)
val people = listOf(Person("Jane", "Doe", 19),
            Person("John", "Doe", 24),
            Person("John", "Smith", 23))
```

并且你想找到第一个符合 *某些标准* 的对象。使用扩展函数，你可以编写如下内容：

```kt
fun <T> List<T>.find(check: (T) -> Boolean): T? {
    for (p in this) {
        if (check(p)) {
            return p
        }
    }
    return null
}
```

然后，当你有一个对象列表时，你可以简单地调用 `find()`：

```kt
println(people.find {
    it.firstName == "John"
}) // Person(firstName=John, lastName=Doe)
```

幸运的是，你不需要实现任何内容。这个方法已经在 Kotlin 中为你实现了。

此外，还有一个配套的 `findLast()` 方法，它执行相同的操作，但以集合的最后一个元素开始：

```kt
println(people.findLast {
    it.firstName == "John"
}) // Person(firstName=John, lastName=Smith)
```

# 删除家族

好吧，如果你必须遍历集合中的所有元素，这很酷。但在 Java 的 `for` 循环中，你可以这样做：

```kt
// Skips first two elements
for (int i = 2; i < list.size(); i++) {
   // Do something here
}
```

你打算用你那些古怪的功能如何实现这一点，嗯？

好吧，为此我们有 `drop()`：

```kt
val numbers = (1..5).toList()
println(numbers.drop(2)) // [3, 4, 5]
```

请注意，这不会以任何方式修改原始集合：

```kt
println(numbers) // [1, 2, 3, 4, 5]
```

如果你想要提前停止你的 *循环*，可以使用 `dropLast()`：

```kt
println(numbers.dropLast(2)) // [1, 2, 3]
```

另一个有趣的功能是`dropWhile()`，它接收一个谓词而不是数字。它跳过直到谓词第一次返回 true：

```kt
val readings = listOf(-7, -2, -1, -1, 0, 1, 3, 4)

println(readings.dropWhile {
    it <= 0
}) // [1, 3, 4]
```

并且还有相应的`dropLastWhile()`。

# 排序家族

别担心，我们不需要实现自己的排序算法。这不是计算机科学 101。

在拥有前一个`find()`示例中的人员列表的情况下，我们希望按年龄对他们进行排序：

```kt
val people = listOf(Person("Jane", "Doe", 19),
        Person("John", "Doe", 24),
        Person("John", "Smith", 23))
```

这可以通过`sortedBy()`轻松实现：

```kt
println(people.sortedBy { it.age })
```

前一个代码打印以下输出：

```kt
[Person(firstName=Jane, lastName=Doe, age=19), Person(firstName=John, lastName=Smith, age=23), Person(firstName=John, lastName=Doe, age=24)]
```

还有`sortedByDescending()`，它将反转结果的顺序：

```kt
println(people.sortedByDescending { it.lastName })
```

前一个代码打印以下输出：

```kt
[Person(firstName=John, lastName=Smith, age=23), Person(firstName=John, lastName=Doe, age=24), Person(firstName=Jane, lastName=Doe, age=19)]
```

如果你想要根据多个参数进行比较，请使用`sortedWith`和`compareBy`的组合：

```kt
println(people.sortedWith(compareBy({it.lastName}, {it.age})))
```

# ForEach

这是我们将要看到的第一个 *终止符*。终止符函数返回的不是新集合，因此你不能将此调用的结果链式调用到其他调用中。

在`forEach()`的情况下，它返回`Unit`。所以它就像普通的旧`for`循环：

```kt
val numbers = (0..5)

numbers.map { it * it}          // Can continue
       .filter { it < 20 }      // Can continue
       .sortedDescending()      // Still can
       .forEach { println(it) } // Cannot continue
```

请注意，`forEach()`与传统的`for`循环相比，有一些轻微的性能影响。

还有`forEachIndexed()`，它提供了集合中的索引和实际值：

```kt
numbers.map { it * it }
        .forEachIndexed { index, value ->
    print("$index:$value, ")
}

```

前一个代码的输出将如下所示：

```kt
0:1, 1:4, 2:9, 3:16, 4:25, 
```

自从 Kotlin 1.1 以来，还有一个`onEach()`函数，它更有用一些，因为它会再次返回集合：

```kt
numbers.map { it * it}         
       .filter { it < 20 }     
       .sortedDescending()     
       .onEach { println(it) } // Can continue now
       .filter { it > 5 }
```

# 连接家族

在先前的例子中，我们使用了打印到控制台的外部效应，这在函数式编程中并不理想。更重要的是，我们还在输出末尾有一个难看的逗号，如下所示：

```kt
0:1, 1:4, 2:9, 3:16, 4:25,
```

肯定有更好的方法。

你有多少次不得不实际编写代码来简单地将一些值列表连接成一个字符串？好吧，Kotlin 有一个函数可以做到这一点：

```kt
    val numbers = (1..5)

    println(numbers.joinToString { "$it"})
```

前一个代码将给出以下输出：

```kt
1, 2, 3, 4, 5
```

简直太美了，不是吗？

如果你想要用其他字符分隔，或者不想有空格，有一种方法可以配置它：

```kt
println(numbers.joinToString(separator = "#") { "$it"})
```

前一个代码的输出将如下所示：

```kt
1#2#3#4#5
```

# Fold/Reduce

与`forEach()`类似，`fold()`和`reduce()`都是终止函数。但它们不是以无用的`Unit`终止，而是以相同类型的单个值终止。

`reduce`最常用的例子当然是累加。在先前的例子中的人员列表中，我们可以做以下操作：

```kt
println(people.reduce {p1, p2 ->
        Person("Combined", "Age", p1.age + p2.age)
    })
```

前一个代码的输出将如下所示：

```kt
Person(firstName=Combined, lastName=Age, age=64)
```

嗯，把很多人合并成一个没有太多意义，除非你是某些恐怖电影的粉丝。

但使用`reduce`，我们还可以计算列表中最老或最年轻的人：

```kt
println(people.reduce {p1, p2 ->
    if (p1.age > p2.age) { p1 } else { p2 }
})
```

我们将要讨论的第二个函数`fold()`与`reduce()`非常相似，但它还接受另一个参数，即初始值。当你已经计算了一些东西，现在想使用这个中间结果时，它很有用：

```kt
println(people.drop(1) // Skipping first one
       .fold(people.first()) // Using first one as initial value
             {p1, p2 ->
    Person("Combined", "Age", p1.age + p2.age)
})
```

# 平铺家族

假设你有一系列其他列表。你可能从不同的数据库查询中得到了它，或者可能来自不同的配置文件：

```kt
val listOfLists = listOf(listOf(1, 2),
        listOf(3, 4, 5), listOf(6, 7, 8))

// [[1, 2], [3, 4, 5], [6, 7, 8]]
```

而您希望将它们转换成一个如下的单列表：

```kt
[1, 2, 3, 4, 5, 6, 7, 8]
```

合并这些列表的一种方法是通过编写一些命令式代码：

```kt
val results = mutableListOf<Int>()

for (l in listOfLists) {
    results.addAll(l)
}
```

但调用 `flatten()` 会为您做同样的事情：

```kt
listOfLists.flatten()
```

您也可以使用 `flatMap()` 控制这些结果的处理：

```kt
println(listOfLists.flatMap {
    it.asReversed()
})
```

注意，在这种情况下，它指的是其中一个子列表。

您也可以选择使用 `flatMap()` 进行类型转换：

```kt
println(listOfLists.flatMap {
    it.map { it.toDouble() }
//  ^        ^
// (1)      (2)
})
```

上述代码打印出以下输出：

```kt
[1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0]
```

我们将所有整数转换为双精度浮点数，然后合并到一个列表中。

注意，第一个 `it` 指的是列表中的一个，而第二个 `it` 指的是当前列表中的一个单独的项目。

就 `flatten()` 而言，它只会扁平化一层。为了证明这一点，我们将使用 `Set` 作为第一层嵌套，`List` 作为第二层嵌套，再次使用 `Set` 作为第三层嵌套：

```kt
val setOfListsOfSets = setOf(
//                     ^
//                    (1)
        listOf(setOf(1, 2), setOf(3, 4, 5), setOf(6, 7, 8)), 
//      ^      ^
//     (2)    (3)
        listOf(setOf(9, 10), setOf(11, 12))
//      ^      ^
//     (2)    (3)
)
// Prints [[[1, 2], [3, 4, 5], [6, 7, 8]], [[9, 10], [11, 12]]]
```

如果我们只调用一次 `flatten`，我们只会收到第一层扁平化：

```kt
println(setOfListsOfSets.flatten())
```

上述代码打印出以下输出：

```kt
[[1, 2], [3, 4, 5], [6, 7, 8], [9, 10], [11, 12]]
```

要完全扁平化列表，我们需要调用 `flatten()` 两次：

```kt
println(setOfListsOfSets.flatten().flatten())
```

上述代码打印出以下输出：

```kt
[1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12]
```

Kotlin 阻止我们三次调用 `flatten()`，因为它识别出我们有多少嵌套：

```kt
//Won't compile
println(setOfListsOfSets.flatten().flatten().flatten())
```

# Slice

假设我们有一个元素列表，如下所示：

```kt
val numbers = (1..10).toList()
// Prints [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
```

我们可以使用 `slice()` 只取列表的一部分：

```kt
println(numbers.slice((0..3)))
// Prints [1, 2, 3, 4], last index is included
```

我们正在使用 Kotlin 范围，这是一个很好的语法。

在 Java 中，有一个 `subList()` 方法，它与它类似，但不包含：

```kt
println(numbers.subList(0, 3))
// Prints [1, 2, 3], last index is excluded
```

# Chunked

在生产代码中，这种分块逻辑非常常见。

您从某处读取了一个庞大的标识符列表，您需要检查您的数据库或某些远程服务是否包含它们。但是，对单个请求中可以传递的标识符数量有限制。例如，数据库通常对查询的参数数量和总查询长度有限制：

```kt
fun dbCall(ids: List<Int>) {
    if (ids.size > 1000) {
        throw RuntimeException("Can't process more than 1000 ids")
    }
```

```kt
    // Does something here
}
```

我们不能简单地将整个列表传递给我们的函数：

```kt
// That will fail at runtime
dbCall(hugeList)

```

因此，我们编写了大量命令式代码：

```kt
val pageSize = 1000
val pages = hugeList.size / pageSize

for (i in 0..pages) {
    val from = i * pageSize
    val p = (i+1) * pageSize
    val to = minOf(p, hugeList.size)
    dbCall(hugeList.slice(from until to))
}
```

幸运的是，自从 Kotlin 1.2 以来，有 `chunked()` 函数可以做到这一点：

```kt
hugeList.chunked(pageSize) {
    dbCall(it)
}
```

# Zip/Unzip

与存档无关，`zip()` 允许我们根据索引从两个列表中创建对。这可能听起来有些令人困惑，所以让我们看看一个例子。

我们有两个函数，一个用于获取所有活跃的员工，另一个用于员工在我们初创公司工作的天数：

```kt
val employeeIds = listOf(5, 8, 13, 21, 34, 55, 89)
val daysInCompany = listOf(176, 145, 117, 92, 70, 51, 35, 22, 12, 5)
```

在两者之间调用 `zip()` 将产生以下结果：

```kt
println(employeeIds.zip(daysInCompany))
```

上述代码打印出以下输出：

```kt
[(5, 176), (8, 145), (13, 117), (21, 92), (34, 70), (55, 51), (89, 35)]
```

注意，由于我们在第二个函数中有一个错误，返回了已经离开我们初创公司的员工的日期，因此两个列表的长度一开始就不相等。调用 `zip()` 总是会产生最短的对列表：

```kt
println(daysInCompany.zip(employeeIds))
```

上述代码打印出以下输出：

```kt
[(176, 5), (145, 8), (117, 13), (92, 21), (70, 34), (51, 55), (35, 89)]
```

注意，这不是一个映射，而是一对列表。

有这样一个列表，我们还可以将其解包：

```kt
val employeesToDays = employeeIds.zip(daysInCompany)

val (employees, days) = employeesToDays.unzip()
println(employees)
println(days)
```

上述代码打印出以下内容：

```kt
[5, 8, 13, 21, 34, 55, 89]
[176, 145, 117, 92, 70, 51, 35]
```

# 流是懒加载的，集合不是

虽然要对大型集合上的那些函数小心谨慎。大多数函数都会为了不可变性而复制集合。

以 `as` 开头的函数不会这样做：

```kt
// Returns a view, no copy here
(1..10).toList().asReversed()

// Same here
(1..10).toList().asSequence()
```

要了解差异，请检查以下代码：

```kt
val numbers = (1..1_000_000).toList()
println(measureTimeMillis {
    numbers.stream().map {
        it * it
    }
}) // ~2ms

println(measureTimeMillis {
    numbers.map {
        it * it
    }
}) // ~19ms
```

你会注意到使用 `stream()` 的代码实际上从未执行。流是惰性的，它们等待一个终止函数调用。另一方面，集合上的函数是依次执行的。

如果我们添加终止调用，我们会看到完全不同的数字：

```kt
println(measureTimeMillis {
    numbers.stream().map {
        it * it
    }.toList()
}) // ~70ms
```

将流转换回列表是一个昂贵的操作。在决定使用哪种方法时，请考虑这些要点。

# 序列

由于流仅在 Java 8 中引入，但 Kotlin 与 Java 6 兼容，因此需要提供另一种解决方案以处理无限集合的可能性。这个解决方案被命名为 *sequenced*，因此当 Java 流可用时，它不会与之冲突。

你可以从 `1` 开始生成一个无限序列：

```kt
val seq = generateSequence(1) { it + 1 }
```

要只取前 `100` 个，我们使用 `take()` 函数：

```kt
    seq.take(100).forEach {
        println(it)
    }
```

通过返回 `null` 可以创建有限数量的序列：

```kt
val finiteSequence = generateSequence(1) {
    if (it < 1000) it + 1 else null
}

finiteSequence.forEach {
        println(it)
} // Prints numbers up to 1000
```

通过调用 `asSequence()` 可以从范围或集合创建有限数量的序列：

```kt
(1..1000).asSequence()
```

# 摘要

本章致力于练习函数式编程原则，并学习 Kotlin 中函数式编程的构建块。

现在，你应该知道如何使用 `map()`/`mapTo()` 转换你的数据，如何 `filter()` 集合，以及根据标准 `find()` 元素。

你还应该熟悉如何使用 `drop()` 跳过元素，如何 `sort()` 集合，以及如何使用 `forEach()` 和 `onEach()` 迭代它们。

使用 `join()` 将集合转换为字符串，使用 `fold()` 和 `reduce()` 对集合求和，使用 `flatten()` 和 `flatTo()` 减少集合嵌套。

`slice()` 是获取集合一部分的方法，而 `chunked()` 用于将集合分成相等的部分。

最后，`zip()` 和 `unzip()` 将两个集合组合成一对，或者将一对拆分成两部分。

在下一章中，我们将讨论熟悉这些方法如何帮助我们真正地实现响应式编程。
