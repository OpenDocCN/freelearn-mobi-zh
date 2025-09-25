# 第四章：强大的数据处理

在本章中，我们将介绍以下菜谱：

+   简单地组合和消费集合

+   过滤数据集

+   自动移除`null`

+   使用自定义比较器排序数据

+   根据数据集元素构建字符串

+   将数据划分为子集

+   使用`map`和`flatMap`转换数据

+   数据集折叠和缩减

+   数据分组

# 简介

本章重点介绍标准库对声明式操作集合的支持。以下菜谱展示了针对数据集转换、缩减和分组的不同类型编程问题的解决方案。我们将学习如何以函数式编程风格处理数据处理操作，以及标准库中内置的强大数据处理扩展。

# 简单地组合和消费集合

Kotlin 标准库提供了一些方便的扩展，使得集合创建和合并变得简单且安全。我们将逐步学习它们。假设我们定义了以下`Message`类：

```java
data class Message(val text: String, 
                   val sender: String, 
                   val timestamp: Instant = Instant.now())
```

在这个菜谱中，我们将创建两个包含`Message`实例的样本集合，并将它们合并成一个`Message`对象的列表。接下来，我们将遍历消息列表，并将它们的文本打印到控制台。

# 准备工作

Kotlin 标准库提供了两个基本接口，用于表示集合数据结构——`Collection`和`MutableCollection`，两者都扩展了`Iterable`接口。第一个接口定义了一个不可变集合，它只支持对其元素的读取访问。第二个接口允许我们添加和删除元素。还有更多专门化的接口扩展了`Collection`和`MutableCollection`基本类型，例如`List`、`MutableList`、`Set`和`MutableSet`。

有许多函数可用于创建不同类型的集合。最常用的函数是`<T> listOf(vararg elements: T)`，它实例化一个`List`实例，以及`<T> mutableListOf(vararg elements: T)`，它返回一个`MutableList`实例，作为函数参数给出的元素。

# 如何做到这一点...

1.  让我们声明两个包含样本数据的列表：

```java
val sentMessages = listOf (
    Message("Hi Agat, any plans for the evening?", "Samuel"),
    Message("Great, I'll take some wine too", "Samuel")
)
val inboxMessages = mutableListOf(
        Message("Let's go out of town and watch the stars tonight!",
         "Agat"),
        Message("Excelent!", "Agat")
)
```

1.  将`sentMessages`和`inboxMessages`合并到一个集合中：

```java
val allMessages: List<Message> = sentMessages + inboxMessages
```

1.  将存储在`allMessages`列表中的`Message`对象的文本打印到控制台：

```java
val allMessages: List<Message> = sentMessages + inboxMessages
allMessages.forEach { (text, _) ->
    println(text)
}
```

# 它是如何工作的...

因此，我们的代码将打印以下文本到控制台：

```java
Hi Agat, any plans for the evening?
Great, I'll take some wine too
Let's go out of town and watch the stars tonight!
Excelent!
```

为了将一个集合的元素添加到另一个集合中，我们使用了 `+` 操作符。标准库通过合并两个 `Collection` 类型实例集合的元素到单个实例的代码来重载此操作符。`sentMessages` 和 `inboxMessages` 变量被声明为 `List` 实例。`plus` 函数返回一个新的 `Collection` 实例，包含 `sentMessages` 和 `inboxMessages` 列表中的元素。最后，我们使用 `forEach()` 函数遍历列表的下一个元素。在传递给 `forEach` 函数的 lambda 块中，我们将当前 `Message` 的 `text` 属性打印到控制台。我们在 `println()` 函数内部直接访问 lambda 的 `Message` 类型参数的文本属性。

# 更多内容...

标准库还为 `Collection` 类型重载了 `-` 操作符。我们可以用它从集合中减去一些元素。例如，我们可以这样使用它：

```java
val receivedMessages = allMessages - sentMessages
receivedMessages.forEach { (text, _) ->
    println(text)
}
```

我们将得到以下输出：

```java
Let's go out of town and watch the stars tonight!
Excelent!
```

我们也可以使用标准的 `for` 循环来实现迭代：

```java
for (msg in allMessages) {
    println(msg.text)
}
```

# 相关内容

+   你可以在第二章 *Expressive Functions and Adjustable Interfaces* 中的 *Destructuring types* 菜单中了解更多关于解构声明的信息，第二章。

+   如果你想掌握 lambda 表达式，可以查看第三章 *Working effectively with lambdas and closures* 中的 *Shaping Code with Kotlin Functional Programming Features* 菜单，第三章。

# 过滤数据集

过滤是数据处理领域中最常见的编程挑战之一。在本菜谱中，我们将探索标准库的内置扩展函数，这些函数提供了一种简单的方式来过滤 `Iterable` 数据类型。假设我们有一个以下 `Message` 类声明：

```java
data class Message(val text: String,
                   val sender: String,
                   val receiver: String,
                   val folder: Folder = Folder.INBOX,
                   val timestamp: Instant = Instant.now())

enum class Folder { INBOX, SENT }
```

`getMessages()` 函数返回以下数据：

```java
fun getMessages() = mutableListOf(
        Message("Je t'aime", "Agat", "Sam", Folder.INBOX),
        Message("Hey, Let's go climbing tomorrow", "Stefan", "Sam", Folder.INBOX),
        Message("<3", "Sam", "Agat", Folder.SENT),
        Message("Yeah!", "Sam", "Stefan", Folder.SENT)
)
```

我们将对 `getMessages()` 函数应用一个过滤操作，只返回具有 `Folder.INBOX` 属性和 `sender` 属性等于 `Agat` 的消息。

# 准备工作

为了实现过滤转换，我们将使用标准库提供的 `Iterable<T>.filter(predicate: (T) -> Boolean)` 扩展函数。`filter()` 函数接受一个返回 `true` 或 `false` 值的谓词函数，用于给定泛型 `Iterable` 数据集类型 `T` 的元素。

# 如何操作...

1.  对 `getMessages()` 函数应用过滤：

```java
getMessages().filter { it.folder == Folder.INBOX && it.sender == "Agat" }
```

1.  遍历过滤后的消息并将它们的消息打印到控制台：

```java
getMessages().filter { it.folder == Folder.INBOX && it.sender == "Agat" }
 .forEach { (text) ->
     println(text)
 }
```

# 工作原理...

我们正在将 `filter` 函数应用于 `ge``tMessages()` 函数的结果。我们将一个 lambda 块传递给 `filter` 函数，该函数为列表的每个元素返回一个布尔值。`filter` 函数返回一个包含过滤对象的列表。最后，我们使用 `forEach()` 函数遍历列表的下一个元素。在传递给 `forEach` 函数的 lambda 块中，我们将当前 `Message` 的 `text` 属性打印到控制台。

因此，上一节中的代码将在控制台打印以下输出：

```java
Je t'aime
```

# 还有更多...

Kotlin 标准库为其他类型提供了相应的 `filter()` 扩展函数，例如 `Array`、`Sequence` 和 `Map`。还有许多过滤函数的变体，这些变体可以用于特定场景。您可以在 Kotlin 标准库 `kotlin.collections` 包的官方文档中找到它们，网址为 [`kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections`](http://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/index.html)。

# 参见

+   如果你想掌握 lambda 表达式，你可以查看第三章 *有效地使用 lambda 和闭包* 菜谱，来自 *使用 Kotlin 函数式编程特性塑造代码*。

# 自动移除空值

当与服务器或外部库的糟糕设计 API 一起工作时，我们经常需要处理接收空返回值的情况。幸运的是，有一些标准库特性允许我们有效地处理空值。在本菜谱中，我们将实现一个数据预处理操作，该操作将自动从数据集中移除所有空值。假设我们正在与一个提供最新新闻源的外部 API 一起工作。不幸的是，它不是空安全的，可能会返回随机的空值。例如，让我们假设我们有一个 `getNews(): List<News>` 函数，它返回以下数据：

```java
fun getNews() = listOf(
 News("Kotlin 1.2.40 is out!", "https://blog.jetbrains.com/kotlin/"),
 News("Google launches Android KTX Kotlin extensions for developers",
 "https://android-developers.googleblog.com/"),
 null,
 null,
 News("How to Pick a Career", "waitbutwhy.com")
)
```

`News` 类定义如下：

```java
data class News(val title: String, val url: String)
```

# 如何做到...

将 `filterNotNull` 函数应用于 `getNews()` 函数：

```java
getNews()
        .filterNotNull()
        .forEachIndexed { index, news ->
            println("$index. $news")
        }
```

# 它是如何工作的...

因此，我们将得到以下输出打印到控制台：

```java
0\. News(title=Kotlin 1.2.40 is out!, url=https://blog.jetbrains.com/kotlin/)
1\. News(title=Google launches Android KTX Kotlin extensions for developers, url=https://android-developers.googleblog.com/)
2\. News(title=How to Pick a Career, url=waitbutwhy.com)
```

相比之下，没有 `filterNotNull()` 函数的代码如下：

```java
getNews().forEachIndexed { index, news ->
    println("$index. ${news.toString()}")
}
```

这将在控制台打印以下输出：

```java
0\. News(title=Kotlin 1.2.40 is out!, url=https://blog.jetbrains.com/kotlin/)
1\. News(title=Google launches Android KTX Kotlin extensions for developers, url=https://android-developers.googleblog.com/)
2\. null
3\. null
4\. News(title=How to Pick a Career, url=waitbutwhy.com)
```

`Iterable.filterNotNull()` 扩展函数从原始数据集中移除所有空值。在底层，它将非空值复制到新创建的 `List` 实例中。这就是为什么在处理大型数据集时，使用序列而不是集合以提供延迟求值更有效的原因。

# 参见

+   在 *过滤数据集* 菜谱中，我们探讨了如何使用标准库提供的基本 `filter()` 函数

+   如果你想掌握 lambda 表达式，你可以查看第三章的*有效使用 lambda 和闭包*菜谱，*使用 Kotlin 函数式编程特性塑造代码*。

# 使用自定义比较器进行数据排序

在这个菜谱中，我们将探索按集合元素属性对集合元素进行排序的支持。

# 开始

假设我们正在处理以下声明的两个`Message`类型集合：

```java
data class Message(val text: String,
                   val sender: String,
                   val receiver: String,
                   val time: Instant = Instant.now())
```

这些由`allMessages`变量提供：

```java
val sentMessages = listOf(
        Message("I'm programming in Kotlin, of course", 
                "Samuel", 
                "Agat", 
                Instant.parse("2018-12-18T10:13:35Z")),
        Message("Sure!", 
                "Samuel", 
                "Agat", 
                Instant.parse("2018-12-18T10:15:35Z"))
)
val inboxMessages = mutableListOf(
        Message("Hey Sam, any plans for the evening?", 
                "Samuel", 
                "Agat", 
                Instant.parse("2018-12-18T10:12:35Z")),
        Message("That's cool, can I join you?", 
                "Samuel", 
                "Agat",
                Instant.parse("2018-12-18T10:14:35Z"))
)
val allMessages = sentMessages + inboxMessages
```

如果我们打印出`allMessages`列表中连续消息的文本，我们将在控制台得到以下文本输出：

```java
I'm learning Kotlin, of course
Sure!
Hey Sam, any plans for the evening?
That's cool, can I join you?
```

这看起来不太对。消息应该按时间顺序显示。这意味着它们应该按`time`属性排序。

# 如何做...

1.  将`sortedBy`函数应用于`allMessages`集合：

```java
allMessages.sortedBy { it.time }
```

1.  将排序后的元素打印到控制台：

```java
allMessages.sortedBy { it.time }
        .forEach {
            println(it.text)
 }
```

# 它是如何工作的...

如果我们运行前面的代码，我们得到以下输出：

```java
I'm programming in Kotlin, of course
Sure!
```

```java
Hey Sam, any plans for the evening?
That's cool, can I join you?
```

现在，所有消息都已正确排序，对话也变得有意义。

# 还有更多...

如果我们的集合由实现 Comparable 接口的对象组成，我们就可以通过对其应用`sorted()`函数来简单地对其进行排序。Kotlin 标准库还提供了`sortedBy()`函数的更多专用版本，例如`sortedByDescending()`和`sortedWith()`。第一个是一个基础排序函数，但它返回按相反顺序排序的数据集。`sortedWith()`函数允许我们使用自定义比较器对列表进行排序。例如，为了按`sender`属性首先排序，然后按`time`属性排序`Message`类型元素的集合，我们可以编写以下代码：

```java
allMessages.sortedWith(compareBy({it.sender}, {it.time}))
```

# 基于数据集元素构建字符串

有时候，我们会遇到根据集合元素生成文本的问题。这就是`Iterable.joinToString()`扩展函数能帮到的地方。例如，我们可以考虑实现一个电子邮件转发功能。当用户点击转发按钮时，原始消息的正文文本被连接起来，前缀看起来像这样：

```java
<br/>
<p>---------- Forwarded message ----------</p>
<p>
From: johny.b@gmail.com <br/>
Date: 14/04/2000 <br/>
Subject: Any plans for the evening?<br/>
To: natasha@gmail.com, barbra@gmail.com<br/>
</p>
```

在这个菜谱中，我们将实现一个函数，该函数将生成收件人的字符串，例如：

```java
To: natasha@gmail.com, barbra@gmail.com</br>
```

对于给定的`Address`类型对象列表，它定义如下：

```java
data class Address(val emailAddress: String, val displayName: String)
```

# 如何做...

1.  声明`generateRecipientsString()`函数头：

```java
fun generateRecipientsString(recipients: List<Address?>): String
```

1.  首先从`recipient`参数中移除所有`null`项：

```java
fun generateRecipientsString(recipients: List<Address?>): String =
 recipients.filterNotNull()
```

1.  将`Address`类型的集合元素转换为与`Address.emailAddress`属性对应的`String`类型元素：

```java
fun generateRecipientsString(recipients: List<Address?>): String =
        recipients.filterNotNull()
 .map { it.emailAddress }
```

1.  为了将集合元素合并成字符串，应用`joinToString()`函数：

```java
fun generateRecipientsString(recipients: List<Address?>): String =
        recipients.filterNotNull()
                .map { it.emailAddress }
 .joinToString(", ", "To: ", "<br/>")
```

# 它是如何工作的...

`generateRecipientsString()`函数使用标准库`kotlin.collections`包中的`Iterable.joinToString()`扩展函数来生成输出字符串。`joinToString()`函数接受三个参数——分隔符字符，用于连接下一个子字符串，前缀和后缀字符串。它在一个字符串值的集合上调用。我们还应用了预处理操作，负责从`Address`对象的列表中删除`null`值并将`Address`类型映射到与`Address.emailAddress`属性对应的`String`。

# 还有更多...

我们还可以使用`joinToString()`函数的另一个版本来简化`generateRecipientsString()`函数实现中的逻辑：

```java
fun generateRecipientsString(recipients: List<Address?>): String =
        recipients.filterNotNull()
            .joinToString(", ", "To: ", "<br/>") { it.emailAddress }
```

如您所见，它采用额外的形式为内联 lambda 块的形式的参数，该参数充当应用于每个`recipients`集合元素的转换函数。

# 参见

+   要深入了解数据集映射操作，您可以阅读*使用 map 和 flatMap 进行数据转换*菜谱

# 将数据划分为子集

常见的数据处理任务是将数据集合划分为子集。在这个菜谱中，我们将探索标准库函数，这些函数允许我们将集合缓冲到更小的块中。假设我们有一个包含大量`Message`类型对象的列表，我们希望将其转换成固定大小的子列表集合。例如，转换将原始集合的*n*个元素：

```java
[mssg_1, mssg_2, mssg_3, mssg_4, mssg_5, mssg_6, mssg_7, ..., mssg_n]
```

然后将其分割成包含四个元素子集的集合：

```java
[[mssg_1, mssg_2, mssg_3, mssg_4], ..., [mssg_n-3, mssg_n-2, mssg_n-1, mssg_n]]
```

# 准备工作

让我们先声明一个`Message`类，我们将在下面的菜谱中使用它：

```java
data class Message(val text: String,
                   val time: Instant = Instant.now())
```

让我们声明一个存储样本数据的`messages`变量：

```java
val messages = listOf(
        Message("Any plans for the evening?"),
        Message("Learning Kotlin, of course"),
        Message("I'm going to watch the new Star Wars movie"),
        Message("Would u like to join?"),
        Message("Meh, I don't know"),
        Message("See you later!"),
        Message("I like ketchup"),
        Message("Did you send CFP for Kotlin Conf?"),
        Message("Sure!")
)
```

# 如何做...

1.  将`windowed()`函数应用于`messages`列表：

```java
val pagedMessages = messages.windowed(4, partialWindows = true, step = 4)  
```

1.  将一个`transform: (List<T>) -> R`转换函数作为额外的内联参数添加到`windowed`函数中：

```java
val pagedMessages = messages.windowed(4, partialWindows = true, step = 4) { 
    it.map { it.text }
}
```

# 它是如何工作的...

`windowed`函数将原始消息列表分割成指定大小的子列表。结果，我们得到`List<List<Message>>`类型分配给`pagedMessages`处理。我们可以使用以下代码打印下一个消息子集：

```java
pagedMessages.forEach { println(it) }
```

结果，我们得到以下输出打印到控制台：

```java
[Any plans for the evening?, Learning Kotlin, of course, I'm going to watch the new Star Wars movie, Would u like to join?]
[Meh, I don't know, See you later!, I like the ketchup, Did you send CFP for Kotlin Conf?]
[Sure!]
```

`windowed`函数接受四个参数——窗口大小、一个标志表示是否应创建部分窗口、一个步长值以及一个可选的转换函数，该函数负责转换生成的每个窗口。在我们的场景中，我们使用窗口大小等于`4`。这就是为什么我们需要将步长值也指定为`4`，因为我们希望将连续的`Message`实例存储在下一个窗口中。我们还设置了`partialWindows`参数为`true`，否则包含单个消息的最后一个窗口将被省略。`windowed`函数的最后一个参数允许我们将每个窗口映射到另一个类型。我们将`windowed()`函数返回的每个子列表映射到`List<String>`类型。还有一个`windowed`函数的另一个版本，没有最后一个映射参数，因此它可以被视为可选的。

# 还有更多...

还提供了一个`windowed()`函数的便捷包装器，称为`chunked()`。它不需要步长参数，并自动将其设置为窗口大小值。它非常适合这个问题的解决方案，然而，`windowed()`函数被解释为更基础的。

# 参见

+   还有一些其他函数可供使用，用于解决不同的列表和集合划分场景，例如`subList()`函数（[`kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/-list/sub-list.html`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/-list/sub-list.html)）和`partition()`函数（[`kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/partition.html`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/partition.html)）。您可以通过提供的链接在官方文档中了解更多关于它们的信息。

# 使用 map 和 flatMap 进行数据转换

对声明式数据映射操作的支持是函数式数据处理领域的基本且最强大的功能之一。通常，当处理数据时，我们需要将特定类型的集合转换成另一种类型。从集合的每个元素生成对象并将所有这些新对象合并到目标集合中也是一个常见的场景。这些就是`map()`和`flatMap`扩展函数能够帮助的场景。

在这个菜谱中，我们将使用这两个函数来实现映射数据转换。让我们想象我们正在处理负责管理大学系讲座的系统的一部分。我们得到了以下类型：

```java
class Course(val name: String, val lecturer: Lecturer, val isPaid: Boolean = false)
class Student(val name: String, val courses: List<Course>)
class Lecturer(val name: String)
```

我们还有一个`getStudents(): List<Student>`函数，它从数据库中返回所有学生的列表。我们想要实现`getLecturesOfCoursesWithSubscribedStudents()`函数，该函数将`getStudents()`的结果转换成计算一个列表，其中包含至少有一名学生订阅的课程讲师。

# 如何做到这一点...

1.  声明一个函数头：

```java
fun getLecturesOfCoursesWithSubscribedStudents()
```

1.  对学生列表应用`flatMap`操作：

```java
fun getLecturesOfCoursesWithSubscribedStudents() =
        getStudents()
                .flatMap { student ->
                    student.courses
                }
```

1.  限制集合元素的值为唯一值：

```java
fun getLecturesOfCoursesWithSubscribedStudents() =
        getStudents()
                .flatMap { student ->
                    student.courses
                }
                .distinct()
```

1.  将`Course`类型元素的集合映射到它们对应的`Lecturer`类型属性：

```java
fun getLecturesOfCoursesWithSubscribedStudents() =
 getStudents()
 .flatMap { student ->
     student.courses
 } 
 .distinct()
 .map { course -> course.lecturer } 
 .distinct()
```

# 它是如何工作的...

以下`flatMap`操作将`getLecturesOfCoursesWithSubscribedStudents()`函数将`Student`类型对象的集合转换为`Course`类型对象的集合，通过合并`Student.courses: Collection<Course>`属性中的元素：

```java
getStudents()
        .flatMap { student: Student ->
            student.courses
        }
```

结果，前面的代码返回了`Collection<Course>`类型。`flatMap`操作返回的集合包含所有学生（从`getStudents()`函数获取）订阅的所有课程的集合。

接下来，为了去除重复的课程，我们使用`distinct()`函数来追加操作链。然后，我们使用`map()`函数。它负责将`Course`类型的每个元素转换为从`Course.lecturer`属性对应的`Lecturer`类型。最后，我们再次应用`distinct()`函数，以返回没有重复的讲师列表。

# 还有更多...

`map()`和`flatMap()`扩展函数也适用于`Map`数据结构类型。当需要将映射转换为从映射的键值对转换的对象列表时，它们非常有用。

# 数据集的折叠和归约

虽然`map()`操作符接受给定大小的列表并返回另一个大小相同且类型修改后的列表，但应用于数据集的`fold()`和`reduce()`操作返回一个单一元素，由数据集的连续元素组成。这听起来可能像使用简单的命令式循环和局部累加变量（它持有当前状态并在每次迭代中更新）的简单场景。我们可以考虑一个简单的任务，即求整数值的总和。假设我们想要计算从`0`到`10`的连续整数的总和。我们可以使用一个简单的`for`循环来实现它：

```java
var sum = 0
(1..10).forEach {
    sum += it
}
```

然而，有一种替代的函数式方法可以执行这样的计算，使用`fold()`函数：

```java
val sum = (1..3).toList().fold(0) { acc, i -> acc + i }

```

当我们实现一系列函数式数据处理操作时，第二种方法更可取。与 for 循环相比，`fold`函数不强制显式消费集合元素，并且可以很容易地与其他函数式操作符一起使用。

在这个菜谱中，我们将使用`fold`函数来实现处理音频专辑曲目时负责的函数。假设我们给出了以下数据类型：

```java
data class Track(val title: String, val durationInSeconds: Int)
data class Album(val name: String, val tracks: List<Track>)
```

以及示例`Album`类实例：

```java
val album = Album("Sunny side up", listOf(
        Track("10/10", 176),
        Track("Coming Up Easy", 292),
        Track("Growing Up Beside You", 191),
        Track("Candy", 303),
        Track("Tricks of the Trade", 151)
))
```

我们想要实现一个针对`Album`类型的扩展函数，该函数将为作为参数给出的`Track`返回一个相对起始时间。例如，`Growing Up Beside You`这首曲子的起始时间应该是 468 秒。

# 如何实现...

1.  声明一个针对`Album`类的扩展函数：

```java
fun Album.getStartTime(track: Track): Int
```

1.  计算给定`Track`参数的起始时间：

```java
fun Album.getStartTime(track: Track): Int {
    val index = tracks.indexOf(track)
 return this.tracks
            .take(index)
 .map { (name, duration) -> duration }
            .fold(0) { acc, i -> acc + i }
}
```

1.  为`track`参数添加一个安全检查：

```java
fun Album.getStartTime(track: Track): Int {
 if (track !in tracks) throw IllegalArgumentException("Bad 
     track")

    val index = tracks.indexOf(track)
    return tracks
        .take(index)
        .map { (name, duration) -> duration }
        .fold(0) { acc, i -> acc + i }
}
```

# 它是如何工作的...

在一开始，我们的函数对传入的`track`参数进行安全检查，以验证它是否属于当前的`Album`实例。如果给定的曲目在`Album.tracks`集合中未找到，则抛出`IllegalArgumentException`异常。接下来，我们创建一个子列表，只包含`tracks`属性元素中的从`0`索引到作为函数参数传递的`track`索引的元素。这个子列表是通过使用`take()`操作符创建的。然后，我们将每个`Track`类型的元素映射到对应的`Int`类型，即曲目的持续时间。最后，我们应用`fold`函数，以累加连续`Track`元素的`durationInSeconds`属性值。`fold`函数接受一个`initial`参数，该参数负责初始化内部`accumulator`变量，该变量持有折叠结果的当前状态。

在我们的情况下，我们将 0 作为`initial`值传递，这对应于专辑的起始时间。在传递给`fold`函数的第二个参数中，我们定义了如何使用每个连续的`durationInSeconds`值更新`accumulator`。

让我们测试一下`Album.getStartTime()`函数的实际效果：

```java
println(album.getStartTime(Track("Growing Up Beside You", 191)))
println(album.getStartTime(Track("Coming Up Easy", 292)))
```

上述代码返回以下输出：

```java
468
176
```

# 还有更多...

标准库提供了一个名为`reduce()`的类似函数，它执行与`fold`相同的操作。这两个函数之间的区别在于`fold`需要一个显式的初始值，而`reduce`则使用列表中的第一个元素作为初始值。

# 数据分组

Kotlin 标准库为数据集*按组分组*操作提供了内置支持。在本菜谱中，我们将探讨如何使用它。

假设我们正在处理以下类型：

```java
class Course(val name: String, val lecturer: Lecturer, val isPaid: Boolean = false)
class Student(val name: String, val courses: List<Course>)
class Lecturer(val name: String)
```

我们还有一个`getStudents(): List<Student>`函数，它返回数据库中所有学生的列表。

给定`getStudents(): List<Student>`函数，我们将实现`getCoursesWithSubscribedStudents(): Map<Course, List<Student>>`函数，该函数负责提取所有学生订阅的课程映射以及每个课程订阅的学生列表。

# 如何实现...

1.  声明一个函数头：

```java
fun getCoursesWithSubscribedStudents(): Map<Course, List<Student>> 
```

1.  将每个学生映射到课程-学生配对的列表：

```java
fun getCoursesWithSubscribedStudents(): Map<Course,
 List<Student>> =
    getStudents()
 .flatMap { student ->
                student.courses.map { course -> course to student }
            }
```

1.  按照课程对课程-学生配对进行分组：

```java
fun getCoursesWithSubscribedStudents(): Map<Course,
 List<Student>> =
    getStudents()
            .flatMap { student ->
                student.courses.map { course -> course to student }
            }
 .groupBy { (course, student) -> course }
```

1.  对`Pair<Course, List<Student>>`类型应用映射转换：

```java
fun getCoursesWithSubscribedStudents(): Map<Course,
 List<Student>> =
    getStudents()
            .flatMap { student ->
                student.courses.map { course -> course to student }
            }
            .groupBy { (course, _) -> course }
 .map { (course, courseStudentPairs) -> 
                course to courseStudentPairs.map { (_, student) -> 
                 student } 
            }
```

1.  在最后应用一个`toMap()`函数：

```java
fun getCoursesWithSubscribedStudents(): Map<Course,
 List<Student>> =
    getStudents()
            .flatMap { student ->
                student.courses.map { course -> course to student }
            }
            .groupBy { (course, _) -> course }
            .map { (course, courseStudentPairs) ->
                course to courseStudentPairs.map { (_, student) ->
                 student }
            }
 .toMap()
```

# 它是如何工作的...

我们首先使用 `flatMap()` 函数将学生列表转换为 `Pair<Course, Student>` 类型的列表。接下来，我们应用 `groupBy()` 函数将这些配对按不同的 `Course` 实例进行分组。分组操作的结果是以下类型的数据——`Map.Entry<Course, List<Pair<Course, Student>>>`。我们需要将 `Map.Entry.value` 属性的类型转换为 `List<Student>` 类型。我们通过以下映射转换函数实现它：

```java
map { (course, courseStudentPairs) ->
    course to courseStudentPairs.map { (_, student) -> student }
}
```

因此，每个 `Course` 实例都与订阅它的学生列表相关联（`Pair<Course, List<Student>>`）。请注意，这里使用了中缀 `to` 函数来实例化 Pair 类型。最后，我们调用 `toMap()` 函数，它从课程-学生配对列表生成最终的 `Map<Course, List<Students>>` 实例。

# 还有更多...

我们还可以通过使用 `mapValues` 函数将我们的映射构建操作修改为更简洁的形式：

```java
fun getCoursesWithSubscribedStudents(): Map<Course, List<Student>> =
        getStudents()
                .flatMap { student ->
                    student.courses.map { course -> course to student }
                }
                .groupBy { (course, _) -> course }
                .mapValues { (course, courseStudentPairs) ->
                    courseStudentPairs.map { it -> it.second }
                }
```

# 参见

+   本食谱中的代码在映射操作中使用了解构类型声明。如果您想了解更多关于这个话题的信息，可以查看第二章中的 *Destructuring types* 食谱，*Expressive Functions and Adjustable Interfaces*。
