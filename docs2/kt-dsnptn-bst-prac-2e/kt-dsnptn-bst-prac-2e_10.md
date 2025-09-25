# *第八章*：设计并发

**并发设计模式**帮助我们同时管理许多任务并结构化它们的生命周期。通过有效地使用这些模式，我们可以避免资源泄露和死锁等问题。

在本章中，我们将讨论并发设计模式及其在**Kotlin**中的实现方式。为此，我们将使用前几章中的构建块：协程、通道、流以及来自**函数式编程**的概念。

本章我们将涵盖以下内容：

+   延迟值

+   障碍

+   调度器

+   管道

+   扇出

+   扇入

+   竞赛

+   互斥锁

+   辅助通道

完成本章学习后，你将能够高效地处理异步值，协调不同协程的工作，以及分配和汇总工作，同时拥有解决过程中可能出现的任何并发问题的工具。

# 技术要求

除了前几章的技术要求外，你还需要一个启用了**Gradle**的 Kotlin 项目，以便能够添加所需的依赖项。

你可以在以下位置在**GitHub**上找到本章使用的源代码：

[`github.com/PacktPublishing/Kotlin-Design-Patterns-and-Best-Practices/tree/main/Chapter08`](https://github.com/PacktPublishing/Kotlin-Design-Patterns-and-Best-Practices/tree/main/Chapter08)

# 延迟值

**延迟值**设计模式的目的是返回异步计算结果的引用。在**Java**和**Scala**中，**Future**，在**JavaScript**中则是**Promise**，都是延迟值设计模式的实现。

我们已经讨论了`async()`函数返回一个名为`Deferred`的类型，这同样也是该设计模式的一个实现。

有趣的是，`Deferred`值本身是我们在*第三章*“理解结构型模式”中看到的**代理**设计模式以及*第四章*“熟悉行为型模式”中的**状态**设计模式的实现。

我们可以使用`CompletableDeferred`构造函数创建一个新的异步计算结果的容器：

```kt
val deferred = CompletableDeferred<String>()
```

要用结果填充`Deferred`值，我们使用`complete()`函数，如果在过程中发生错误，我们可以使用`completeExceptionally()`函数将异常传递给调用者。为了更好地理解它，让我们编写一个返回异步结果的函数。一半的时间结果将包含`OK`，另一半的时间它将包含一个异常。

```kt
suspend fun valueAsync(): Deferred<String> = coroutineScope {
```

```kt
    val deferred = CompletableDeferred<String>()
```

```kt
    launch {
```

```kt
        delay(100)
```

```kt
        if (Random.nextBoolean()) {
```

```kt
            deferred.complete("OK")
```

```kt
        }
```

```kt
        else {
```

```kt
            deferred.completeExceptionally(
```

```kt
              RuntimeException()
```

```kt
            )
```

```kt
        }
```

```kt
    }
```

```kt
    deferred
```

```kt
}
```

你可以看到我们几乎立即返回`Deferred`值，然后我们开始使用`launch`启动异步计算，并使用`delay()`函数模拟一些计算。

由于过程是异步的，结果不会立即准备好。为了等待结果，我们可以使用我们已经在 *第六章* 中讨论过的 `await()` 函数，*线程和协程*：

```kt
runBlocking {
```

```kt
    val value = valueAsync()
```

```kt
    println(value.await())
```

```kt
}
```

确保您始终通过调用 `complete()` 或 `completeExceptionally()` 函数之一来完成您的 `Deferred` 值非常重要。否则，您的程序可能会无限期地等待结果。如果您不再对 `deferred` 的结果感兴趣，也可以取消它。为此，只需调用其上的 `cancel()` 即可：

```kt
deferred.cancel()
```

您很少需要创建自己的 deferred 值。通常，您会使用 `async()` 函数返回的那个。

接下来，让我们讨论如何一次性等待多个异步结果。

# 屏障

**屏障**设计模式为我们提供了在进一步操作之前等待多个并发任务完成的机制。这种用法的一个常见场景是从不同的来源组合对象。

例如，考虑以下类：

```kt
data class FavoriteCharacter(
```

```kt
    val name: String,
```

```kt
    val catchphrase: String,
```

```kt
    val picture: ByteArray = Random.nextBytes(42)
```

```kt
)
```

假设 `catchphrase` 数据来自一个服务，而 `picture` 数据来自另一个服务。我们希望并发获取这两份数据：

```kt
fun CoroutineScope.getCatchphraseAsync
```

```kt
(
```

```kt
    characterName: String
```

```kt
) = async { … }
```

```kt
fun CoroutineScope.getPicture
```

```kt
(
```

```kt
    characterName: String
```

```kt
) = async { … }
```

实现并发获取的最基本方式如下：

```kt
suspend fun fetchFavoriteCharacter(name: String) = coroutineScope {
```

```kt
    val catchphrase = getCatchphraseAsync(name).await()
```

```kt
    val picture = getPicture(name).await()
```

```kt
    FavoriteCharacter(name, catchphrase, picture)
```

```kt
}
```

但这个解决方案有一个主要问题——我们只有在 `catchphrase` 数据被获取之后才开始获取 `picture` 数据。换句话说，代码是不必要地 *顺序的*。让我们看看如何改进这一点。

## 使用数据类作为屏障

我们可以稍微修改之前的代码，以实现我们想要的并发：

```kt
suspend fun fetchFavoriteCharacter(name: String) = coroutineScope { 
```

```kt
    val catchphrase = getCatchphraseAsync(name) 
```

```kt
    val picture = getPicture(name) 
```

```kt
    FavoriteCharacter(name, catchphrase.await(),       picture.await()) 
```

```kt
}
```

将 `await` 函数移动到数据类构造函数的调用中，使我们能够同时启动所有协程，然后等待它们完成，正如我们想要的。

使用数据类作为屏障的额外好处是能够轻松地进行 *解构*：

```kt
val (name, catchphrase, _) = fetchFavoriteCharacter("Inigo Montoya")
```

```kt
println("$name says: $catchphrase")
```

如果我们从不同的异步任务中接收到的数据类型是异构的，这效果很好。在某些情况下，我们从不同的来源接收相同类型的数据。

例如，让我们询问 `Michael`（我们的金丝雀产品所有者），`Taylor`（我们的咖啡师），以及 `Me` 我们最喜欢的电影角色是谁：

```kt
object Michael {
```

```kt
    suspend fun getFavoriteCharacter() = coroutineScope {
```

```kt
        async {
```

```kt
            FavoriteCharacter("Terminator", 
```

```kt
              "Hasta la vista, baby")
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

```kt
object Taylor {
```

```kt
    suspend fun getFavoriteCharacter() = coroutineScope {
```

```kt
        async {
```

```kt
            FavoriteCharacter("Don Vito Corleone", "I'm 
```

```kt
              going to make him an offer he can't refuse")
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

```kt
object Me {
```

```kt
    suspend fun getFavoriteCharacter() = coroutineScope {
```

```kt
        async {
```

```kt
            // I already prepared the answer!
```

```kt
            FavoriteCharacter("Inigo Montoya",               "Hello, my name is...")
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

在这里，我们有三个非常相似的对象，它们之间的区别仅在于它们返回的异步结果的内容。

在这种情况下，我们可以使用一个列表来收集结果：

```kt
val characters: List<Deferred<FavoriteCharacter>> =    listOf(
```

```kt
        Me.getFavoriteCharacter(),
```

```kt
        Taylor.getFavoriteCharacter(),
```

```kt
        Michael.getFavoriteCharacter(),
```

```kt
    )
```

注意列表的类型。它是一个包含 `FavoriteCharacter` 类型 `Deferred` 元素的集合。在这样的集合上，有一个可用的 `awaitAll()` 函数，它也充当一个屏障：

```kt
println(characters.awaitAll())
```

当处理一组同构的异步结果，并且需要在进一步操作之前等待所有结果完成时，请使用 `awaitAll()`。

屏障设计模式为多个异步任务创建了一个 rendezvous 点。下一个模式将帮助我们抽象这些任务的执行。

# 调度器

**调度器** 设计模式的目标是将 *做什么* 与 *怎么做* 解耦，并在这样做时优化资源的使用。

在 Kotlin 中，**分发器** 是调度器设计模式的一种实现，它将协程（即，*做什么*）与底层的线程池（即，*怎么做*）解耦。

我们已经在 *第六章*，*线程和协程* 中简要介绍了分发器。

为了提醒你，协程构建器如 `launch()` 和 `async()` 可以指定要使用哪个分发器。以下是如何明确指定它的一个示例：

```kt
runBlocking {
```

```kt
    // This will use the Dispatcher from the parent 
```

```kt
    // coroutine
```

```kt
    launch {
```

```kt
        // Prints: main
```

```kt
        println(Thread.currentThread().name) 
```

```kt
    }
```

```kt
    launch(Dispatchers.Default) {
```

```kt
        // Prints DefaultDispatcher-worker-1
```

```kt
        println(Thread.currentThread().name) 
```

```kt
    }
```

```kt
}
```

默认分发器会根据底层线程池中的 CPU 数量创建线程。你还可以使用另一个分发器，即 **IO 分发器**：

```kt
async(Dispatchers.IO) {
```

```kt
    for (i in 1..1000) {
```

```kt
        println(Thread.currentThread().name)
```

```kt
        yield()
```

```kt
    }
```

```kt
}
```

这将输出以下内容：

```kt
> …
```

```kt
> DefaultDispatcher-worker-2
```

```kt
> DefaultDispatcher-worker-1
```

```kt
> DefaultDispatcher-worker-1
```

```kt
> DefaultDispatcher-worker-1
```

```kt
> DefaultDispatcher-worker-3
```

```kt
> DefaultDispatcher-worker-3
```

```kt
> ...
```

IO 分发器用于可能运行时间较长或阻塞的操作，并将为此创建多达 64 个线程。由于我们的示例代码没有做太多，IO 分发器不需要创建很多线程。这就是为什么你在这个例子中只会看到少量工作者。

## 创建自己的调度器

我们不仅限于 Kotlin 提供的分发器。我们还可以定义自己的分发器。

下面是一个创建分发器的例子，该分发器将使用基于 `ForkJoinPool` 的专用线程池，其中包含 `4` 个线程，这对于 *分而治之* 任务是高效的：

```kt
val forkJoinPool = ForkJoinPool(4).asCoroutineDispatcher()
```

```kt
repeat(1000) {
```

```kt
    launch(forkJoinPool) {
```

```kt
        println(Thread.currentThread().name)
```

```kt
    }
```

```kt
}
```

如果你创建了自己的分发器，请确保你使用 `close()` 释放它或者重用它，因为创建新的分发器并持有它会在资源方面造成开销。

# 管道

**管道** 设计模式允许我们通过将工作分解成更小的、并发的部分来扩展异构工作，这些工作由多个具有不同复杂性的步骤组成，跨越多个 CPU。让我们通过以下示例来更好地理解它。

回到 *第四章*，*熟悉行为模式*，我们编写了一个 HTML 页面解析器。假设 HTML 页面本身已经被我们获取了。我们现在想要设计一个可能无限生成页面的过程。

首先，我们希望偶尔抓取新闻页面。为此，我们将有一个生产者：

```kt
fun CoroutineScope.producePages() = produce {
```

```kt
    fun getPages(): List<String> {
```

```kt
        // This should actually fetch something
```

```kt
        return listOf(
```

```kt
            "<html><body><h1>
```

```kt
               Cool stuff</h1></body></html>",
```

```kt
            "<html><body><h1>
```

```kt
               Even more stuff</h1></body></html>"
```

```kt
        )
```

```kt
    }
```

```kt
    val pages = getPages()
```

```kt
    while (this.isActive) {
```

```kt
        for (p in pages) {
```

```kt
            send(p)
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

`isActive` 标志将在协程运行且未被取消的情况下为真。在可能运行时间较长的循环中检查这个属性是一个好习惯，这样它们就可以在迭代之间停止，如果需要的话。

每次我们收到新的标题时，我们会将它们发送到下游。由于技术新闻更新并不频繁，我们可以通过使用 `delay()` 来偶尔检查更新。在实际代码中，延迟可能是几分钟，甚至几小时。

下一步是创建一个由包含 HTML 的原始字符串组成的 **文档对象模型**（**DOM**）。为此，我们将有一个第二个生产者，这个生产者接收一个连接到第一个生产者的通道：

```kt
fun CoroutineScope.produceDom(pages: ReceiveChannel<String>) = produce {
```

```kt
    fun parseDom(page: String): Document {
```

```kt
        // In reality this would use a DOM library to parse 
```

```kt
        // string to DOM
```

```kt
        return Document(page)
```

```kt
    }
```

```kt
    for (p in pages) {
```

```kt
        send(parseDom(p))
```

```kt
    }
```

```kt
}
```

只要通道仍然打开，我们就可以使用`for`循环遍历通道。这是一种非常优雅地从异步源消费数据的方式，无需定义回调函数。

我们将有一个第三个函数，它接收解析后的文档并从每个文档中提取标题：

```kt
fun CoroutineScope.produceTitles(parsedPages: ReceiveChannel<Document>) = produce {
```

```kt
    fun getTitles(dom: Document): List<String> {
```

```kt
        return dom.getElementsByTagName("h1").map {
```

```kt
            it.toString()
```

```kt
        }
```

```kt
    }
```

```kt
    for (page in parsedPages) {
```

```kt
        for (t in getTitles(page)) {
```

```kt
            send(t)
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

我们正在寻找标题，所以我们使用`getElementsByTagName("H1")`。对于找到的每个标题，我们将其转换为字符串表示形式。

现在，我们将继续将我们的协程组合成管道。

## 构建管道

现在我们已经熟悉了管道的组件，让我们看看如何将多个组件组合在一起：

```kt
runBlocking {
```

```kt
    val pagesProducer = producePages()
```

```kt
    val domProducer = produceDom(pagesProducer)
```

```kt
    val titleProducer = produceTitles(domProducer)
```

```kt
    titleProducer.consumeEach {
```

```kt
        println(it)
```

```kt
    }
```

```kt
}
```

生成的管道将如下所示：

```kt
Input=>pagesProducer=>domProducer=>titleProducer=>Output
```

管道是将一个长过程分解成更小步骤的绝佳方式。请注意，每个生成的协程都是一个*纯函数*，因此它也易于测试和推理。

可以通过在第一协程上调用`cancel()`来停止整个管道。

# Fan Out

Fan Out 设计模式的目的是在多个并发处理器之间分配工作，也称为*工作者*。为了更好地理解它，让我们再次查看前面的部分，但考虑以下问题：

*如果我们在管道中不同步骤的工作量非常不同怎么办？*

例如，获取 HTML 内容比解析它花费的时间要多得多。在这种情况下，我们可能希望将这项繁重的工作分配给多个协程。在先前的例子中，只有一个协程从每个通道读取。但多个协程也可以从单个通道消费，从而分担工作。

为了简化我们即将讨论的问题，让我们只有一个协程产生一些结果：

```kt
fun CoroutineScope.generateWork() = produce {
```

```kt
    for (i in 1..10_000) {
```

```kt
        send("page$i")
```

```kt
    }
```

```kt
    close()
```

```kt
}
```

我们将有一个函数来创建一个新的协程，该协程读取这些结果：

```kt
fun CoroutineScope.doWork(
```

```kt
    id: Int,
```

```kt
    channel: ReceiveChannel<String>
```

```kt
) = launch(Dispatchers.Default) {
```

```kt
    for (p in channel) {
```

```kt
        println("Worker $id processed $p")
```

```kt
    }
```

```kt
}
```

这个函数将生成一个在`Default`调度器上执行的协程。每个协程将监听一个通道，并将接收到的每条消息打印到控制台。

现在，让我们启动我们的生产者。记住，所有以下代码片段都需要包裹在`runBlocking`函数中，但为了简单起见，我们省略了这部分：

```kt
val workChannel = generateWork()
```

然后，我们可以创建多个工作者，他们通过从相同的通道读取来相互分配工作：

```kt
val workers = List(10) { id ->
```

```kt
    doWork(id, workChannel)
```

```kt
}
```

现在让我们检查这个程序的输出的一部分：

```kt
> ...
```

```kt
> Worker 4 processed page9994
```

```kt
> Worker 8 processed page9993
```

```kt
> Worker 3 processed page9992
```

```kt
> Worker 6 processed page9987
```

注意，没有两个工作进程收到相同的信息，并且消息的打印顺序与发送顺序不同。Fan Out 设计模式允许我们高效地将工作分配给多个协程、线程和 CPU。

接下来，让我们讨论一个与 Fan Out 经常一起出现的伴随设计模式。

# Fan In

Fan In 设计模式的目的是将多个工作者的结果合并起来。当我们的工作者产生结果而我们又需要收集它们时，这个设计模式非常有用。

这种设计模式是我们在上一节中讨论的扇出设计模式的对立面。不是多个协程从同一个通道中 *读取*，而是多个协程可以将它们的结果 *写入* 到同一个通道。

结合扇出和扇入设计模式是 **MapReduce** 算法的好基础。为了演示这一点，我们将对前一个例子中的工作进程进行轻微的修改，如下所示：

```kt
private fun CoroutineScope.doWorkAsync(
```

```kt
    channel: ReceiveChannel<String>,
```

```kt
    resultChannel: Channel<String>
```

```kt
) = async(Dispatchers.Default) {
```

```kt
    for (p in channel) {
```

```kt
        resultChannel.send(p.repeat(2))
```

```kt
    }
```

```kt
}
```

现在，一旦完成，每个工作进程将其计算结果发送到 `resultChannel`。

注意，这种模式与我们之前看到的演员和生产者构建器不同。每个演员都有自己的通道，而在这个例子中，`resultChannel` 是所有工作进程共享的。

为了收集来自工作进程的结果，我们将使用以下代码：

```kt
runBlocking {
```

```kt
    val workChannel = generateWork()
```

```kt
    val resultChannel = Channel<String>()
```

```kt
    val workers = List(10) {
```

```kt
        doWorkAsync(workChannel, resultChannel)
```

```kt
    }
```

```kt
    resultChannel.consumeEach {
```

```kt
        println(it)
```

```kt
    }
```

```kt
}
```

让我们现在明确一下这段代码的作用：

1.  首先，我们创建 `resultChannel`，所有我们的工作进程都将共享。

1.  然后，我们将它提供给每个工作进程。我们总共有十个工作进程。每个工作进程将其接收到的消息重复两次，并将其发送到 `resultChannel`。

1.  最后，我们在主协程中从通道中消费结果。这样，我们就可以在同一个地方累积来自多个并发工作进程的结果。

这是前面代码的输出样本：

```kt
> ...
```

```kt
> page9995page9995
```

```kt
> page9996page9996
```

```kt
> page9997page9997
```

```kt
> page9999page9999
```

```kt
> page9998page9998
```

```kt
> page10000page10000
```

接下来，让我们讨论另一种设计模式，它将有助于我们在某些情况下提高代码的响应性。

# 竞赛

**竞赛**是一种并发运行多个作业的设计模式，选择第一个返回的结果作为 *赢家*，并将其他作为 *输家*。

我们可以使用 Kotlin 中的 `select()` 函数在通道上实现竞赛。

让我们想象你正在构建一个天气应用程序。为了冗余，你从两个不同的来源获取天气，*Precise Weather* 和 *Weather Today*。我们将它们描述为两个返回其名称和温度的生产者。

如果我们有多个生产者，我们可以订阅它们的通道，并获取第一个可用的结果。

首先，让我们声明两个天气生产者：

```kt
fun CoroutineScope.preciseWeather() = produce {
```

```kt
    delay(Random.nextLong(100))
```

```kt
    send("Precise Weather" to "+25c")
```

```kt
}
```

```kt
fun CoroutineScope.weatherToday() = produce {
```

```kt
    delay(Random.nextLong(100))
```

```kt
    send("Weather Today" to "+24c")
```

```kt
}
```

它们的逻辑几乎相同。两者都等待随机的毫秒数，然后返回温度读数和来源名称。

我们可以使用 `select` 表达式同时监听两个通道：

```kt
runBlocking {
```

```kt
  val winner = select<Pair<String, String>> {
```

```kt
    preciseWeather().onReceive { preciseWeatherResult ->
```

```kt
            preciseWeatherResult
```

```kt
        }
```

```kt
        weatherToday().onReceive { weatherTodayResult ->
```

```kt
            weatherTodayResult
```

```kt
        }
```

```kt
    }
```

```kt
    println(winner)
```

```kt
}
```

使用 `onReceive()` 函数允许我们同时监听多个通道。

运行此代码多次将随机打印 `(Precise Weather, +25c)` 和 `(Weather Today, +24c)`，因为它们到达第一个的机会是均等的。

竞赛是一个非常有用的概念，当你愿意牺牲资源以从你的系统中获得最大的响应性时，我们就使用 Kotlin 的 `select` 表达式实现了这一点。现在，让我们进一步探索 `select` 表达式，以发现它实现的另一个并发设计模式。

## 无偏选择

当使用 `select` 子句时，顺序很重要。因为它固有的偏差，如果两个事件同时发生，它将选择第一个子句。

让我们看看以下示例中的含义。

这次我们只有一个生产者，它通过通道发送我们应该观看的下一部电影：

```kt
fun CoroutineScope.fastProducer(
```

```kt
    movieName: String
```

```kt
) = produce(capacity = 1) {
```

```kt
    send(movieName)
```

```kt
}
```

由于我们在通道上定义了非零容量，值将在这个协程运行时立即可用。

现在，让我们启动两个生产者，并使用`select`表达式来查看哪部电影将被选中：

```kt
runBlocking {
```

```kt
    val firstOption = fastProducer("Quick&Angry 7")
```

```kt
    val secondOption = fastProducer(
```

```kt
      "Revengers: Penultimatum")
```

```kt
    delay(10)
```

```kt
    val movieToWatch = select<String> {
```

```kt
        firstOption.onReceive { it }
```

```kt
        secondOption.onReceive { it }
```

```kt
    }
```

```kt
    println(movieToWatch)
```

```kt
}
```

无论你运行此代码多少次，获胜者总是相同的：`Quick&Angry 7`。这是因为如果两个值同时准备好，`select`子句总是会选择它们声明的第一个通道。

现在，让我们使用`selectUnbiased`而不是`select`子句：

```kt
...
```

```kt
val movieToWatch = selectUnbiased<String> {
```

```kt
    firstOption.onReceive { it }
```

```kt
    secondOption.onReceive { it }
```

```kt
}
```

```kt
...
```

现在运行此代码有时会产生`Quick&Angry 7`，有时会产生`Revengers: Penultimatum`。与常规的`select`子句不同，`selectUnbiased`不关心顺序。如果有多个结果可用，它将随机选择一个。

# 互斥锁

也称为**互斥**，**互斥锁**提供了一种保护共享状态的方法，该状态可以被多个协程同时访问。

让我们从那个古老的令人讨厌的`counter`例子开始，其中多个并发任务尝试更新同一个`counter`：

```kt
var counter = 0
```

```kt
val jobs = List(10) {
```

```kt
    async(Dispatchers.Default) {
```

```kt
        repeat(1000) {
```

```kt
            counter++
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

```kt
jobs.awaitAll()
```

```kt
println(counter)
```

如你可能猜到的，打印出的结果小于 10,000 – *完全尴尬！*

为了解决这个问题，我们可以引入一个锁定机制，它将允许只有一个协程一次与变量交互，使操作*原子化*。

每个协程都会尝试获取`counter`的所有权。如果另一个协程正在更新`counter`，我们的协程将耐心等待，然后再次尝试获取锁。一旦更新完成，它必须释放锁，以便其他协程可以继续：

```kt
var counter = 0
```

```kt
val mutex = Mutex()
```

```kt
val jobs = List(10) {
```

```kt
    launch {
```

```kt
        repeat(1000) {
```

```kt
            mutex.lock()
```

```kt
            counter++
```

```kt
            mutex.unlock()
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

现在，我们的例子总是打印出正确的数字：`10,000`。

Kotlin 中的互斥锁与 Java 中的互斥锁不同。在 Java 中，互斥锁上的`lock()`会阻塞线程，直到锁可以被获取。而 Kotlin 互斥锁会挂起协程，从而提供更好的并发性。Kotlin 中的锁更便宜。

这对于简单情况很好。但如果在临界区（即`lock()`和`unlock()`之间）的代码抛出异常怎么办？

我们不得不将我们的代码包裹在`try...catch`中，这并不方便：

```kt
try { 
```

```kt
    mutex.lock() 
```

```kt
    counter++                      
```

```kt
} 
```

```kt
finally { 
```

```kt
    mutex.unlock()                     
```

```kt
}
```

然而，如果我们省略了`finally`块，我们的锁将永远不会释放，这将阻止所有其他协程继续进行并导致死锁。

正是为了这个目的，Kotlin 还引入了`withLock()`：

```kt
mutex.withLock {
```

```kt
    counter++
```

```kt
}
```

注意，与之前的例子相比，这个语法要简洁得多。

# Sidekick channel

**Sidekick channel**设计模式允许我们将一些工作从主工作线程卸载到*后台工作线程*。

到目前为止，我们只讨论了将`select`用作*接收者*的使用。但我们也可以使用`select`将项目*发送*到另一个通道。让我们看看以下示例。

首先，我们将`batman`声明为一个处理每秒 10 条消息的 actor 协程：

```kt
val batman = actor<String> {
```

```kt
    for (c in channel) {
```

```kt
        println("Batman is beating some sense into $c")
```

```kt
        delay(100)
```

```kt
    }
```

```kt
}
```

接下来，我们将声明 `robin` 作为另一个协程演员，它稍微慢一些，每秒只处理四条消息：

```kt
val robin = actor<String> {
```

```kt
    for (c in channel) {
```

```kt
        println("Robin is beating some sense into $c")
```

```kt
        delay(250)
```

```kt
    }
```

```kt
}
```

因此，我们有两个演员：一个超级英雄和他的助手。由于超级英雄更有经验，他通常需要更少的时间来击败他面对的反派。

但在某些情况下，他仍然手头很忙，所以需要一个助手介入。我们将向这对组合投掷五个反派，并观察他们的表现：

```kt
val epicFight = launch {
```

```kt
    for (villain in listOf("Jocker", "Bane", "Penguin",       "Riddler", "Killer Croc")) {
```

```kt
        val result = select<Pair<String, String>> {
```

```kt
            batman.onSend(villain) {
```

```kt
                "Batman" to villain
```

```kt
            }
```

```kt
            robin.onSend(villain) {
```

```kt
                "Robin" to villain
```

```kt
            }
```

```kt
        }
```

```kt
        delay(90)
```

```kt
        println(result)
```

```kt
    }
```

```kt
}
```

注意，`select` 表达式的参数类型指的是从块中**返回**的内容，而不是被**发送**到通道的内容。这就是我们在这里使用 `Pair<String, String>` 的原因。

这段代码打印以下内容：

```kt
> Batman is beating some sense into Jocker
```

```kt
> (Batman, Jocker)
```

```kt
> Robin is beating some sense into Bane
```

```kt
> (Robin, Bane)
```

```kt
> Batman is beating some sense into Penguin
```

```kt
> (Batman, Penguin)
```

```kt
> Batman is beating some sense into Riddler
```

```kt
> (Batman, Riddler)
```

```kt
> Robin is beating some sense into Killer Croc
```

```kt
> (Robin, Killer Croc)
```

使用助手通道是一种有用的技术，可以提供回退值。考虑在需要消费一致数据流且无法轻松扩展消费者的情况下使用它。

# 摘要

在本章中，我们介绍了与 Kotlin 中**并发**相关的各种设计模式。大多数模式都是基于协程、通道、延迟值或这些构建块的组合。

延迟值用作异步值的占位符。屏障设计模式允许多个异步任务在继续之前进行会合。调度器设计模式将任务代码与其在运行时执行的方式解耦。

管道、扇入和扇出设计模式帮助我们分配工作并收集结果。互斥锁帮助我们控制同时执行的任务数量。竞态设计模式允许我们提高应用程序的响应性。最后，助手通道设计模式在主任务无法快速处理传入事件时，将工作卸载到备用任务上。

所有这些模式都应该帮助你以高效和可扩展的方式管理应用程序的并发。在下一章中，我们将讨论 Kotlin 的惯用用法和最佳实践，以及随着语言出现的某些反模式。

# 问题

1.  当我们说 Kotlin 中的 `select` 表达式是**有偏好的**时，这是什么意思？

1.  你应该在什么情况下使用**互斥锁**而不是**通道**？

1.  哪些并发设计模式可以帮助你有效地实现 MapReduce 或分而治之算法？
