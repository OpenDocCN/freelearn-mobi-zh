# 第九章：为并发设计

在本章中，我们将讨论最常见的并发设计模式，这些模式使用协程实现，以及协程如何同步它们的执行。

并发设计模式帮助我们同时管理许多任务。是的，我知道，我们在上一章就是这样做的。那是因为其中一些设计模式已经内置到语言中。

在本章中，我们将简要介绍你需要自己实现的设计模式和并发设计模式，这些模式需要付出很少的努力。

在本章中，我们将介绍以下主题：

+   活动对象

+   延迟值

+   障碍

+   调度器

+   管道

+   扇出

+   扇入

+   缓冲通道

+   无偏选择

+   互斥锁

+   在关闭时选择

+   伴随通道

+   延迟通道

# 活动对象

这种设计模式允许一个方法以安全的方式在另一个线程上执行。猜猜还有什么是在另一个线程上执行的？

你完全正确：`actor()`。

因此，它是一种已经内置到语言中的设计模式。或者，更准确地说，内置到其中一个兼容库中。

我们已经看到了如何向`actor()`发送数据。但我们如何从它那里接收数据？

一种方法是为它提供一个输出通道：

```java
fun activeActor(out: SendChannel<String>) = actor<Int> {
    for (i in this) {
        out.send(i.toString().reversed())
    }
    out.close()
}
```

记得在你完成时关闭输出通道。

# 测试

为了测试**活动对象**模式，我们将启动两个作业。一个将数据发送到我们的 actor：

```java
val channel = Channel<String>()
val actor = activeActor(channel)

val j1 = launch {
    for (i in 42..53) {
        actor.send(i)
    }
    actor.close()
}
```

另一个将等待输出通道上的输出：

```java
val j2 = launch {
    for (i in channel) {
        println(i)
    }
}

j1.join()
j2.join()
```

# 延迟值

我们已经在第八章的*线程和协程*部分遇到了延迟值，*返回结果*部分。`Deferred`是`async()`函数的结果，例如。你可能也知道它们来自 Java 或 Scala 的*Future*，或者来自 JavaScript 的*Promise*。

有趣的是，Deferred 是一种我们在前面章节中遇到的**代理**设计模式。

就像 Kotlin 的`Sequence`与 Java8 的`Stream`非常相似一样，Kotlin 的 Deferred 与 Java Future 也非常相似。你很少需要自己创建 Deferred。通常，你会使用`async()`返回的那个。

在你需要返回一个未来将评估的值的占位符的情况下，你可以这样做：

```java
val deferred = CompletableDeferred<String>()

launch {
    delay(100)
    if (Random().nextBoolean()) {
        deferred.complete("OK")
    }
    else {
        deferred.completeExceptionally(RuntimeException())
    }
}

println(deferred.await())
```

这段代码将有一半的时间打印`OK`，另一半的时间抛出`RuntimeException`。

确保你总是完成你的延迟。通常，将包含延迟的任何代码包装在`try...catch`块中是一个好主意。

如果你对延迟的结果不再感兴趣，也可以取消延迟。只需在它上面调用`cancel()`即可：

```java
deferred.cancel()
```

# 障碍

障碍设计模式为我们提供了在进一步操作之前等待多个并发任务的手段。一个常见的用例是从不同的来源组合对象。

例如，以下是一个类：

```java
data class FavoriteCharacter(val name: String, val catchphrase: String, val repeats: Int)
```

假设我们在获取名称、`catchphrase`和数字。这个`catchphrase`正从三个不同的来源重复。

最基本的方法是使用 `CountDownLatch`，就像我们在一些之前的例子中所做的那样：

```java
val latch = CountDownLatch(3)

var name: String? = null
launch {
    delay(Random().nextInt(100))
    println("Got name")
    name = "Inigo Montoya"
    latch.countDown()
}

var catchphrase = ""
launch {
    delay(Random().nextInt(100))
    println("Got catchphrase")
    catchphrase = "Hello. My name is Inigo Montoya. You killed my father. Prepare to die."
    latch.countDown()
}

var repeats = 0
launch {
    delay(Random().nextInt(100))
    println("Got repeats")
    repeats = 6
    latch.countDown()
}

latch.await()

println("${name} says: ${catchphrase.repeat(repeats)}")
```

你会注意到异步任务完成的顺序正在改变：

```java
Got name
Got catchphrase
Got repeats
```

但最终，我们总是打印出相同的结果：

```java
Inigo Montoya says: Hello. My name is Inigo Montoya. ...
```

但这个解决方案带来了很多问题。我们需要处理可变变量，要么为它们设置默认值，要么使用空值。

此外，只要我们使用闭包，这也会工作。如果我们的函数比几行长呢？

# CountDownLatch

当然，我们可以传递它们闩锁。我们已经见过几次的闩锁允许一个线程等待，直到其他线程完成工作：

```java
private fun getName(latch: CountDownLatch) = launch {
    ...
    latch.countDown()
}
```

但这并不是一个清晰的职责分离。我们真的想要指定这个函数应该如何同步吗？

让我们再试一次：

```java
private fun getName() = async {
    delay(Random().nextInt(100))
    println("Got name")
    "Inigo Montoya"
}

private fun getCatchphrase() = async {
    delay(Random().nextInt(100))
    println("Got catchphrase")
    "Hello. My name is Inigo Montoya. You killed my father. Prepare to die."
}

private fun getRepeats() = async {
    delay(Random().nextInt(100))
    println("Got repeats")
    6
}
```

只是一个提醒，`fun getRepeats() = async { ... }` 中并没有什么魔法。它的更长等效形式是：

```java
private fun getCatchphrase(): Deferred<String> {
    return async {
        ...
    }
}
```

我们可以调用我们的代码来得到与之前相同的结果：

```java
val name = getName()
val catchphrase = getCatchphrase()
val repeats = getRepeats()

println("${name.await()} says: ${catchphrase.await().repeat(repeats.await())}")
```

但我们可以通过使用我们的老朋友，数据类，来进一步改进它。

# 数据类作为屏障

现在我们的数据类是屏障：

```java
val character = FavoriteCharacter(getName().await(), getCatchphrase().await(), getRepeats().await())

// Will happen only when everything is ready
with(character) {
    println("$name says: ${catchphrase.repeat(repeats)}")    
}
```

数据类作为屏障的额外好处是能够轻松地解构它们：

```java
val (name, catchphrase, repeats) = character
println("$name says: ${catchphrase.repeat(repeats)}")
```

如果我们从不同的异步任务中接收到的数据类型差异很大，这会工作得很好。在这个例子中，我们接收到了 `String` 和 `Int`。

在某些情况下，我们从不同的来源接收相同类型的数据。

例如，让我们问问迈克尔（我们的金丝雀产品负责人），杰克（我们的咖啡师），以及我，我们最喜欢的电影角色是谁：

```java
object Michael {
    fun getFavoriteCharacter() = async {
        // Doesn't like to think much
        delay(Random().nextInt(10))
        FavoriteCharacter("Terminator", "Hasta la vista, baby", 1)
    }
}

object Jake {
    fun getFavoriteCharacter() = async {
        // Rather thoughtful barista
        delay(Random().nextInt(100) + 10)
        FavoriteCharacter("Don Vito Corleone", "I'm going to make him an offer he can't refuse", 1)
    }
}

object Me {
    fun getFavoriteCharacter() = async {
        // I already prepared the answer!
        FavoriteCharacter("Inigo Montoya", "Hello, my name is...", 6)
    }
}
```

在那种情况下，我们可以使用列表来收集结果：

```java
val favoriteCharacters = listOf(Me.getFavoriteCharacter().await(),
        Michael.getFavoriteCharacter().await(),
        Jake.getFavoriteCharacter().await())

println(favoriteCharacters)
```

# Scheduler

这是我们在 *启动协程* 部分简要讨论过的另一个概念，在 *第八章* 中，*线程和协程*。

记得我们的 `launch()` 或 `async()` 可以接收 `CommonPool` 吗？

这里有一个例子来提醒你，你可以明确指定它：

```java
// Same as launch {}
launch(CommonPool) {
...
}

// Same as async {}
val result = async(CommonPool) {
...
}
```

这个 `CommonPool` 是一个伪装成调度器的调度器设计模式。许多异步任务可能被映射到同一个调度器。

运行以下代码：

```java
val r1 = async(CommonPool) {
    for (i in 1..1000) {
        println(Thread.currentThread().name)
        yield()
    }
}

r1.await()
```

有趣的是，同一个协程被不同的线程选中：

```java
ForkJoinPool.commonPool-worker-2
ForkJoinPool.commonPool-worker-3
...
ForkJoinPool.commonPool-worker-3
ForkJoinPool.commonPool-worker-1
```

你也可以指定上下文为 `Unconfined`：

```java
val r1 = async(Unconfined) {
    ...
}
```

这将在主线程上运行协程。它打印：

```java
main
main
...
```

你也可以从你的父协程继承上下文：

```java
val r1 = async {
    for (i in 1..1000) {
        val parentThread = Thread.currentThread().name
        launch(coroutineContext) {
            println(Thread.currentThread().name == parentThread)
        }
        yield()
    }
}
```

注意，但运行在同一个上下文中并不意味着我们在同一个线程上运行。

你可能会问自己：继承上下文和使用 `Unconfined` 之间有什么区别？我们将在下一节详细讨论这个问题。

# 理解上下文

为了理解不同的上下文，让我们看看下面的代码：

```java
val r1 = async(Unconfined) {
    for (i in 1..1000) {
        println(Thread.currentThread().name)
        delay(1)
    }
}

r1.await()
```

我们不是使用 `yield()`，而是使用 `delay()` 函数，它也会挂起当前的协程。

但与 `yield()` 相比，输出是不同的：

```java
main
kotlinx.coroutines.DefaultExecutor
...
```

在第一次调用 `delay()` 之后，协程已经切换了上下文，从而导致了线程的切换。

因此，不建议对于 CPU 密集型任务或需要在特定线程上运行的任务（如 UI 渲染）使用`Unconfined`。

您也可以为协程创建自己的线程池来运行：

```java
val pool = newFixedThreadPoolContext(2, "My Own Pool")
val r1 = async(pool) {
    for (i in 1..1000) {
        println(Thread.currentThread().name)
        yield()
    }
}

r1.await()
pool.close()
```

它打印：

```java
...
My Own Pool-2
My Own Pool-1
My Own Pool-2
My Own Pool-2
...
```

如果你创建了自己的线程池，请确保你使用`close()`释放它或重用它，因为创建新的线程池并持有它从资源角度来看是昂贵的。

# 管道

在我们的`StoryLand`中，同样的懒散建筑师，也就是我，正在努力解决一个问题。回到第四章，*熟悉行为模式*，我们编写了一个 HTML 页面解析器。但它取决于是否有人已经为我们抓取了要解析的页面。它也不够灵活。

我们希望有一个协程产生无限的新闻流，而其他协程则逐步解析这个流。

要开始使用 DOM，我们需要一个库，例如`kotlinx.dom`。如果你使用**Gradle**，请确保将以下行添加到你的`build.gradle`文件中：

```java
repositories {
    ...
    jcenter()
}

dependencies {
    ...
    compile "org.jetbrains.kotlinx:kotlinx.dom:0.0.10"
}
```

现在，让我们着手处理当前的任务。

首先，我们希望偶尔抓取新闻页面。为此，我们将有一个生产者：

```java
fun producePages() = produce {
    fun getPages(): List<String> {
        // This should actually fetch something
        return listOf("<html><body><H1>Cool stuff</H1></body></html>",
                "<html><body><H1>Event more stuff</H1></body></html>").shuffled()
    }
    while (this.isActive) {
        val pages = getPages()
        for (p in pages) {
            send(p)
        }
        delay(TimeUnit.SECONDS.toMillis(5))
    }
}

```

我们在这里使用`shuffled()`，这样列表元素的顺序就不会总是相同。

只要协程正在运行且未被取消，`isActive`标志将为真。在可能运行很长时间的循环中检查此属性是一种良好的做法，这样它们就可以在迭代之间停止。

每次我们收到新的标题时，我们就将它们发送到下游。

由于科技新闻更新并不频繁。我们可以偶尔使用`delay()`检查更新。在实际代码中，延迟可能是几分钟，甚至几小时。

下一步是创建包含 HTML 的原始字符串的**文档对象模型**（**DOM**）。为此，我们将有一个第二个生产者，这个生产者接收一个连接到第一个生产者的通道：

```java
fun produceDom(pages: ReceiveChannel<String>) = produce {

    fun parseDom(page: String): Document {
         return kotlinx.dom.parseXml(*page.toSource()*)
    }

    for (p in pages) {
        send(parseDom(p))
    }
}
```

我们可以使用`for`循环遍历通道，直到有更多数据到来。这是从通道中消费数据的一种非常优雅的方式。

在这个生产者中，我们最终使用了我们之前导入的 DOM 解析器。我们还引入了一个方便的`String`扩展函数：

```java
private fun String.toSource(): InputSource {
    return InputSource(StringReader(this))
}
```

这是因为`parseXml()`期望`InputSource`作为其输入。基本上，这是一个**适配器**设计模式的应用：

```java
fun produceTitles(parsedPages: ReceiveChannel<Document>) = produce {
    fun getTitles(dom: Document): List<String> {
        val h1 = dom.getElementsByTagName("H1")
        return h1.asElementList().map {
            it.textContent
        }
    }

    for (page in parsedPages) {
        for (t in getTitles(page)) {
            send(t)
        }
    }
}
```

我们正在寻找标题，因此使用`getElementsByTagName("H1")`。对于找到的每个标题，可能有多个，我们使用`textContent`获取其文本。

最后，我们将每个页面的每个标题发送到下一个。

# 建立管道

现在，为了建立我们的管道：

```java
val pagesProducer = producePages()

val domProducer = produceDom(pagesProducer)

val titleProducer = produceTitles(domProducer)

runBlocking {
    titleProducer.consumeEach {
        println(it)
    }
}
```

我们有以下内容：

```java
pagesProducer |> domProducer |> titleProducer |> output
```

管道是将长过程分解成更小步骤的绝佳方式。请注意，每个生产协程都是一个纯函数，因此它也很容易测试和推理。

整个管道可以通过在第一个协程上调用`cancel()`来停止。

我们可以通过使用扩展函数来实现更友好的 API：

```java
private fun ReceiveChannel<Document>.titles(): ReceiveChannel<String> {
    val channel = this
    fun getTitles(dom: Document): List<String> {
        val h1 = dom.getElementsByTagName("H1")
        return h1.asElementList().map {
            it.textContent
        }
    }

    return produce {
        for (page in channel) {
            for (t in getTitles(page)) {
                send(t)
            }
        }
    }
}

private fun ReceiveChannel<String>.dom(): ReceiveChannel<Document> {
    val channel = this
    return produce() {
        for (p in channel) {
            send(kotlinx.dom.parseXml(p.toSource()))
        }
    }
}
```

然后，我们可以这样调用我们的代码：

```java
runBlocking {
    producePages().dom().titles().consumeEach {
        println(it)
    }
}
```

Kotlin 在创建表达性和流畅的 API 方面真的很出色。

# 扇出设计模式

如果我们管道中不同步骤的工作量差异很大怎么办？

例如，获取 HTML 比解析它花费的时间要多得多。或者如果我们根本没有任何管道，只是有很多任务我们希望在协程之间分配。

这就是扇出设计模式发挥作用的地方。协程的数量可以从同一个通道读取，分配工作。

我们可以有一个协程产生一些结果：

```java
private fun producePages() = produce {
    for (i in 1..10_000) {
        for (c in 'a'..'z') {
            send(i to "page$c")
        }
    }
}
```

并有一个函数可以创建一个读取这些结果的协程：

```java

private fun consumePages(channel: ReceiveChannel<Pair<Int, String>>) = async {
    for (p in channel) {
        println(p)
    }
}
```

这使我们能够生成任意数量的消费者：

```java
val producer = producePages()

val consumers = List(10) {
    consumePages(producer)
}

runBlocking {
    consumers.forEach {
        it.await()
    }
}
```

扇出设计模式允许我们高效地将工作分配给多个协程、线程和 CPU。

# 扇入设计模式

如果我们的协程总是可以自己做出决定，那会很好。但它们如果需要将计算结果返回给另一个协程怎么办？

与**扇出**相反的是**扇入**设计模式。不是多个协程从同一个通道读取，而是多个协程可以将他们的结果写入同一个通道。

想象一下，你正在从两个显赫的科技资源中阅读新闻：`techBunch`和`theFerge`。

每个资源以自己的速度产生值，并将它们通过通道发送：

```java
private fun techBunch(collector: Channel<String>) = launch {
    repeat(10) {
        delay(Random().nextInt(1000))
        collector.send("Tech Bunch")
    }
}

private fun theFerge(collector: Channel<String>) = launch {
    repeat(10) {
        delay(Random().nextInt(1000))
        collector.send("The Ferge")
    }
}
```

通过提供相同的通道，我们可以合并他们的结果：

```java
val collector = Channel<String>()

techBunch(collector)
theFerge(collector)

runBlocking {
    collector.consumeEachIndexed {
        println("${it.index} Got news from ${it.value}")
    }
}

```

结合扇出和扇入设计模式是**Map**/**Reduce**算法的良好基础。

为了证明这一点，我们将生成 10,000,000 个随机数，并通过多次分割这个任务来计算它们中的最大数。

首先，生成 10,000,000 个随机整数的列表：

```java
val numbers = List(10_000_000) {
    Random().nextInt()
}
```

# 管理工人

现在，我们将有两种类型的工人：

+   分割工人将接收到数字列表，确定列表中的最大数，并将其发送到输出通道：

```java
fun divide(input: ReceiveChannel<List<Int>>, 
           output: SendChannel<Int>) = async {
    var max = 0
    for (list in input) {
        for (i in list) {
            if (i > max) {
                max = i
                output.send(max)
            }
        }
    }
}
```

+   收集器将监听这个通道，并且每次有新的子最大数到达时，将决定它是否是史上最大的：

```java
fun collector() = actor<Int> {
    var max = 0
    for (i in this) {
        max = Math.max(max, i)
    }
    println(max)
}
```

现在，我们只需要建立那些通道：

```java
val input = Channel<List<Int>>()
val output = collector()
val dividers = List(10) {
    divide(input, output)
}

launch {
    for (c in numbers.chunked(1000)) {
        input.send(c)
    }
    input.close()
}

dividers.forEach {
    it.await()
}

output.close()
```

注意，在这种情况下，我们不会获得性能上的好处，而简单的`numbers.max()`会产生更好的结果。但随着需要收集的数据量增加，这种模式变得更加有用。

# 缓冲通道

到目前为止，我们使用的所有通道的容量都是正好一个元素。

这意味着如果你向这个通道写入，但没有人在读取，发送者将被挂起：

```java
val channel = Channel<Int>()

val j = launch {
    for (i in 1..10) {
        channel.send(i)
        println("Sent $i")
```

```java
    }
}

j.join()
```

这段代码没有打印任何东西，因为协程正在等待有人从通道读取。

为了避免这种情况，我们可以创建一个缓冲通道：

```java
val channel = Channel<Int>(5)
```

现在，只有在通道容量达到时才会发生挂起。

它会打印：

```java
Sent 1
Sent 2
Sent 3
Sent 4
Sent 5
```

由于`produce()`和`actor()`也由通道支持，我们也可以将其设置为缓冲：

```java
val actor = actor<Int>(capacity = 5) {
    ...
}

val producer = produce<Int>(capacity = 10) {
    ...        
}

```

# 无偏选择

与通道一起工作的最有用的一种方式是我们之前在第八章“*生产者*”部分中看到的`select {}`子句，*线程和协程*。

但选择是固有的有偏见的。如果两个事件同时发生，它将选择第一个子句。

在下面的例子中，我们将有一个生产者，它以很短的延迟发送五个值：

```java
fun producer(name: String, repeats: Int) = produce {
    repeat(repeats) {
        delay(1)
        send(name)
    }
}
```

我们将创建三个这样的生产者并查看结果：

```java
val repeats = 10_000
val p1 = producer("A", repeats)
val p2 = producer("B", repeats)
val p3 = producer("C", repeats)

val results = ConcurrentHashMap<String, Int>()
repeat(repeats) {
    val result = select<String> {
        p1.onReceive { it }
        p2.onReceive { it }
        p3.onReceive { it }
    }

    results.compute(result) { k, v ->
        if (v == null) {
            1
        }
        else {
            v + 1
        }
    }
}

println(results)
```

我们运行了五次这个代码。以下是一些结果：

```java
{A=8235, B=1620, C=145}
{A=7850, B=2062, C=88}
{A=7878, B=2002, C=120}
{A=8260, B=1648, C=92}
{A=7927, B=2011, C=62}
```

如你所见，`A`几乎总是赢，而`C`总是第三。你设置的`repeats`越多，偏差就越大。

现在我们使用`selectUnbiased`代替：

```java
...
val result = selectUnbiased<String> {
    p1.onReceive { it }
    p2.onReceive { it }
    p3.onReceive { it }
}
...
```

前五次执行的结果可能看起来像这样：

```java
{A=3336, B=3327, C=3337}
{A=3330, B=3332, C=3338}
{A=3334, B=3333, C=3333}
{A=3334, B=3336, C=3330}
{A=3332, B=3335, C=3333}
```

现在数字分布得更均匀了，而且所有子句都有相同的机会被选中。

# 互斥锁

也称为互斥排他，互斥锁提供了一种保护共享状态的手段。

让我们从那个陈旧、令人讨厌的反例开始：

```java
var counter = 0

val jobs = List(10) {
    launch {
        repeat(1000) {
            counter++
            yield()
        }
    }
}

runBlocking {
    jobs.forEach {
        it.join()
    }
    println(counter)
}
```

如你所猜，这会打印出除了`10*100`的结果之外的所有内容。真是尴尬至极。

为了解决这个问题，我们引入了一个互斥锁：

```java
var counter = 0
val mutex = Mutex()

val jobs = List(10) {
    launch {
        repeat(1000) {
            mutex.lock()
            counter++
            mutex.unlock()
            yield()
        }
    }
}
```

现在我们的例子总是打印出正确的数字。

这对简单情况很有用。但如果关键部分（即`lock()`和`unlock()`之间）的代码抛出异常怎么办？

然后，我们必须将所有内容包裹在`try...catch`中，这并不方便：

```java
repeat(1000) {
    try {
        mutex.lock()
        counter++                     
   }
 finally {
        mutex.unlock()                    
    }

    yield()
}
```

正是为了这个目的，Kotlin 还引入了`withLock()`：

```java
...
repeat(1000) {
    mutex.withLock {
        counter++
    }
    yield()
}
...
```

# 选择在关闭时

使用`select()`从通道中读取直到它关闭是件好事。

你可以在这里看到那个问题的例子：

```java
val p1 = produce {
    repeat(10) {
        send("A")
    }
}

val p2 = produce {
    repeat(5) {
        send("B")
    }
}

runBlocking { 
    repeat(15) {
        val result = selectUnbiased<String> {
            p1.onReceive {
                it
            }
            p2.onReceive {
                it
            }
        }

        println(result)
    }
}
```

虽然数字相加，但我们运行此代码时可能会经常收到`ClosedReceiveChannelException`。这是因为第二个生产者有更少的物品，一旦它完成，它将关闭其通道。

为了避免这种情况，我们可以使用`onReceiveOrNull`，它将同时返回一个可空版本。一旦通道关闭，我们在`select`中就会收到`null`。

我们可以按任何我们想要的方式处理这个空值，例如，通过使用`elvis`运算符：

```java
repeat(15) {
    val result = selectUnbiased<String> {
        p1.onReceiveOrNull {
            // Can throw my own exception
            it ?: throw RuntimeException()
        }
        p2.onReceiveOrNull {
            // Or supply default value
            it ?: "p2 closed"
        }
    }

    println(result)
}
```

利用这些知识，我们可以通过跳过空结果来排空两个通道：

```java
var count = 0
while (count < 15) {
    val result = selectUnbiased<String?> {
        p1.onReceiveOrNull {
            it
        }
        p2.onReceiveOrNull {
            it
        }
    }

    if (result != null) {
        println(result)
        count++
    }
}
```

# 辅助通道

到目前为止，我们只讨论了`select`作为接收者的用法。但我们也可以使用`select`将项目发送到另一个通道。

让我们看看以下例子：

```java
val batman = actor<String> {
    for (c in this) {
        println("Batman is beating some sense into $c")
        delay(100)
    }
}

val robin = actor<String> {
    for (c in this) {
        println("Robin is beating some sense into $c")
        delay(250)
    }
}
```

我们有一个超级英雄及其助手作为两个演员。由于超级英雄更有经验，他们通常花费更少的时间来击败他们面对的恶棍。

但在某些情况下，它们仍然手头很忙，所以需要一个助手来帮忙。

我们将向这对组合投掷五个恶棍，并观察它们的反应，同时加入一些延迟：

```java
val j = launch {
    for (c in listOf("Jocker", "Bane", "Penguin", "Riddler", "Killer Croc")) {
        val result = select<Pair<String, String>> {
            batman.onSend(c) {
                Pair("Batman", c)
            }
            robin.onSend(c) {
                Pair("Robin", c)
            }
        }
        delay(90)
        println(result)
    }
}
```

它打印出：

```java
Batman is beating some sense into Jocker
(Batman, Jocker)
Robin is beating some sense into Bane
(Robin, Bane)
Batman is beating some sense into Penguin
(Batman, Penguin)
Batman is beating some sense into Riddler
(Batman, Riddler)
Robin is beating some sense into Killer Croc
(Robin, Killer Croc)
```

注意，这个选择的类型参数指的是从块中返回的内容，而不是发送到通道的内容。

这就是为什么我们在这里使用`Pair<String, String>`的原因。

# 延迟通道

你与协程工作得越多，就越习惯于等待结果。在某个时刻，你将开始在通道中发送延迟值。

我们将首先创建 10 个异步任务。第一个将延迟很长时间，其余的我们将延迟很短的时间：

```java
val elements = 10
val deferredChannel = Channel<Deferred<Int>>(elements)

launch(CommonPool) {
    repeat(elements) { i ->
        println("$i sent")
        deferredChannel.send(async {
            delay(if (i == 0) 1000 else 10)
            i
        })
    }
}

```

我们将把这些结果放入一个缓冲通道中。

现在，我们可以从这个通道读取，并使用第二个`select`块，等待结果：

```java
val time = measureTimeMillis {
    repeat(elements) {
        val result = select<Int> {
            deferredChannel.onReceive {
                select {
                    it.onAwait { it }
                }
            }
        }
        println(result)
    }
}

println("Took ${time}ms")
```

注意，结果时间是最慢的任务的时间：

```java
Took 1010ms
```

你还可以使用`onAwait()`作为另一个通道的停止信号。

因此，我们将创建一个将在 600 毫秒内完成的异步任务：

```java
val stop = async {
    delay(600)
    true
}

```

并且，就像上一个例子一样，我们将通过缓冲通道发送 10 个延迟值：

```java
val channel = Channel<Deferred<Int>>(10)

repeat(10) {i ->
    channel.send(async {
        delay(i * 100)
        i
    })
}
```

然后，我们将等待新的值或通知通道应该关闭：

```java
runBlocking {
    for (i in 1..10) {
        select<Unit> {
            stop.onAwait {
                channel.close()
            }
            channel.onReceive {
                println(it.await())
            }
        }
    }
}
```

如预期的那样，这仅打印了十个值中的六个，在 600 毫秒后停止。

# 摘要

在本章中，我们介绍了与 Kotlin 中并发相关的各种设计模式。其中大多数都是基于协程、通道、延迟值或它们的组合。

**管道**、**扇入**和**扇出**有助于分配工作和收集结果。**延迟值**用作稍后解决某事的占位符。**调度器**帮助我们管理资源，主要是支持协程的线程。**互斥锁**和**屏障**有助于控制并发。

现在，你应该理解了`select`块以及它如何有效地与通道和延迟值结合使用。

在下一章中，我们将讨论 Kotlin 的惯用用法、最佳实践以及随着语言出现的某些反模式。
