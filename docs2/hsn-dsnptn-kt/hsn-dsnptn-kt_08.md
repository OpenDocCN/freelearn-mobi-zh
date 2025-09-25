# 第八章：线程和协程

在本章中，我们将讨论我们的应用程序如何高效地每秒处理数千个请求。在前一章中，我们已经对其有所了解——**响应式流**使用多个不同的线程（由`Schedulers` API 公开），我们甚至不得不使用`thread()`函数创建一个线程一次或两次。但在深入细节之前，让我们首先讨论线程能够解决哪些类型的问题。

在你的笔记本电脑中，你有一个具有多个核心的 CPU，可能四个。这意味着它可以同时进行四个不同的计算，这相当令人印象深刻，考虑到 10 年前，单核 CPU 是默认的，甚至双核 CPU 也只针对爱好者。

但即使在那时，你实际上并没有限制于一次只能执行一个任务，对吧？你可以一边听音乐一边浏览互联网，甚至在单核 CPU 上也能做到。你的 CPU 是如何做到这一点的呢？嗯，和你的大脑一样。它处理多个任务。当你一边读书一边听朋友说话时，你的一部分时间并不是真的在读书，另一部分时间并不是真的在听。直到我们的大脑中至少有两个核心。

你运行代码的服务器具有几乎相同的 CPU。这意味着它们可以同时处理四个请求。但如果你每秒有 10,000 个请求怎么办？你不能并行处理它们，因为你没有 10,000 个 CPU 核心。但你可以尝试并发处理它们。

在本章中，我们将涵盖以下主题：

+   线程

+   协程

+   通道

# 线程

最基本的并发模型是由 JVM 线程提供的。线程允许我们并发地运行代码（但不一定是并行），从而更好地利用多个 CPU 核心，例如。它们比进程更轻量级。一个进程可能产生数百个线程。与进程不同，线程之间共享数据很容易。但这也会带来很多问题，我们稍后会看到。

让我们先看看如何在 Java 中创建两个线程：

```kt
new Thread(() -> {
   for (int i = 0; i < 100; i++) {
      System.out.println("T1: " + i);
   }
}).start();

new Thread(() -> {
   for (int i = 0; i < 100; i++) {
      System.out.println("T2: " + i);
   }
}).start();
```

输出将类似于以下内容：

```kt
...
T2: 12
T2: 13
T1: 60
T2: 14
T1: 61
T2: 15
T2: 16
...
```

注意，输出将在不同的执行之间有所不同，并且在任何时候都没有保证它是交错进行的。

Kotlin 中的相同代码如下所示：

```kt
val t1 = thread {
    for (i in 1..100) {
        println("T1: $i")
    }
}

val t2 = thread {
    for (i in 1..100) {
        println("T2: $i")
    }
}
```

在 Kotlin 中，由于有一个帮助我们创建新线程的函数，所以代码更简洁。请注意，与 Java 不同，我们不需要调用`start()`来启动线程。它默认启动。如果我们想稍后启动它，我们可以将`start`参数设置为`false`：

```kt
val t2 = thread(start = false) {
    for (i in 1..100) {
        println("T2: $i")
    }
}
...
// Later
t2.start()
```

Java 中另一个有用的概念是*守护线程*。这些线程不会阻止 JVM 退出，非常适合非关键的后台任务。

在 Java 中，API 不是流畅的，所以我们必须将我们的线程分配给一个变量，将其设置为守护线程，然后启动它：

```kt
Thread t1 = new Thread(() -> {
    for (int i = 0; i < 100; i++) {
        System.out.println("T1: " + i);
    }
});
t1.setDaemon(true);
t1.start();
```

在 Kotlin 中，这要简单得多：

```kt
val t3 = thread(isDaemon = true) {
    for (i in 1..1_000_000) {
        println("T3: $i")
    }
}
```

注意，尽管我们要求这个线程打印到一百万，但它只打印了几百个。这是因为它是一个守护线程。当父线程停止时，它也会停止。

# 线程安全

关于线程安全有许多书籍，这是有充分理由的。由缺乏线程安全引起的并发错误是最难追踪的。它们很难重现，因为你通常需要很多线程在同一资源上竞争，以实际发生竞争。因为这本书是关于 Kotlin 而不是一般的线程安全，所以我们只会触及这个话题的表面。如果您对 JVM 语言的线程安全主题感兴趣，您应该查看 Brian Goetz 所著的《Java Concurrency in Practice》这本书。

我们将从以下示例开始，该示例创建 100,000 个线程来增加计数器：

```kt
var counter = 0
val latch = CountDownLatch(100_000)
for (i in 1..100_000) {
    thread {
        counter++
        latch.countDown()
    }
}

latch.await()
println("Counter $counter")
```

如果您对并发编程有一些经验，您会立刻明白为什么这段代码打印的数字小于 100,000。原因是`++`操作不是原子的。所以尝试增加我们的计数器的线程越多，数据竞争的机会就越多。

但是，与 Java 不同，Kotlin 中没有`synchronized`关键字。原因是 Kotlin 的设计者认为一种语言不应该针对特定的并发模型进行定制。相反，有一个`synchronized()`函数：

```kt
var counter = 0
val latch = CountDownLatch(100_000)
for (i in 1..100_000) {
    thread{
        synchronized(latch) {
            counter++
            latch.countDown()
        }
    }
}

latch.await()
println("Counter $counter")
```

现在代码打印出预期的`100000`。

如果您真的很怀念 Java 中的同步方法，Kotlin 中有`@Synchronized`注解。也没有`volatile`关键字，而是使用`@Volatile`注解。

# 线程很昂贵

每次我们创建一个新线程时，都需要付出代价。每个线程都需要一个新的内存栈。

如果我们在每个线程内部模拟一些工作，让它进入休眠状态会怎样？

在以下代码片段中，我们将尝试创建 10,000 个线程，每个线程休眠相对较短的时间：

```kt
val counter = AtomicInteger()
try {
    for (i in 0..10_000) {
        thread {
            counter.incrementAndGet()
            Thread.sleep(100)
        }
    }
} catch (oome: OutOfMemoryError) {
    println("Spawned ${counter.get()} threads before crashing")
    System.exit(-42)
}
```

根据您的操作系统，这可能会导致`OutOfMemoryError`或整个系统变得非常缓慢。当然，有方法可以限制同时运行的线程数量，使用 Java 5 中的**executors API**。

我们创建一个指定大小的新的线程池：

```kt
// Try setting this to 1, number of cores, 100, 2000, 3000 and see what happens
val pool = Executors.newFixedThreadPool(100)
```

现在我们想提交一个新任务。我们通过调用`pool.submit()`来完成这个任务：

```kt

val counter = AtomicInteger(0)

val start = System.currentTimeMillis()
for (i in 1..10_000) {
    pool.submit {
        // Do something
        counter.incrementAndGet()

        // Simulate wait on IO
        Thread.sleep(100)

        // Do something again
        counter.incrementAndGet()
    }
}
```

然后我们需要确保池终止，通过以下几行代码：

```kt
pool.awaitTermination(20, TimeUnit.SECONDS)
pool.shutdown()

println("Took me ${System.currentTimeMillis() - start} millis to complete ${counter.get() / 2} tasks")
```

注意，我们花了 20 秒才完成。这是因为新任务不能开始，直到之前的任务*醒来*并完成它们的工作。

这正是多线程系统中发生的情况，它并不足够并发。

在下一节中，我们将讨论协程如何尝试解决这个问题。

# 协程

除了 Java 提供的线程模型之外，Kotlin 还引入了协程模型。协程可以被认为是轻量级的线程，我们很快就会看到它们相对于现有线程模型的优势。

你需要知道的第一件事是，协程不是语言的一部分。它们只是 JetBrains 提供的另一个库。因此，如果我们想使用它们，我们需要在我们的 Gradle 配置文件`build.gradle`中指定：

```kt
dependencies {
    ...
    compile "org.jetbrains.kotlinx:kotlinx-coroutines-core:0.21"
    ...
}
```

截至 Kotlin 1.2，协程仍然被视为实验性的。但这并不意味着它们工作得不好，尽管有些人可能会这样认为。这只意味着 API 的一些部分可能在下一个版本中发生变化。

可能会发生什么变化？例如，在 0.18 版本中，我们将在此章节后面讨论的 Actor 暴露了一个通道成员。在 0.21 版本中，这个成员被设置为私有，并添加了一个方法。因此，你将不再调用`actor.channel.send()`，而是调用`actor.send()`。

如果你现在还不清楚*actor*或*channel*是什么，我们将在接下来的几节中简要介绍这些术语。

因此，在你添加这个依赖项并开始使用它们之后，你可能会在编译或你的 IDE 中收到警告：

```kt
The feature "coroutines" is experimental
```

你可以使用以下 Gradle 配置来隐藏这些警告：

```kt
kotlin {
    experimental {
        coroutines 'enable'
    }
}
```

现在，让我们开始学习协程。

# 启动协程

我们已经看到了如何在 Kotlin 中启动新线程。现在让我们启动一个新的协程。

我们将创建一个几乎与线程相同的示例。每个协程将增加某个计数器，暂停一段时间来模拟某种 I/O，然后再次增加：

```kt
val latch = CountDownLatch(10_000)
val c = AtomicInteger()

val start = System.currentTimeMillis()
for (i in 1..10_000) {
    launch(CommonPool) {
        c.incrementAndGet()
        delay(100)
        c.incrementAndGet()
        latch.countDown()
    }
}

latch.await(10, TimeUnit.SECONDS)

println("Executed ${c.get() / 2} coroutines in ${System.currentTimeMillis() - start}ms")
```

启动新协程的第一种方式是使用`launch()`函数。再次提醒，这只是一个函数，而不是语言构造。

这个函数接收一个参数：`context: CoroutineContext`。

在底层，协程仍然使用线程池。因此，我们可以指定要使用哪个线程池。`CommonPool`是库中提供的单例。

另一个有趣的地方是关于我们使用的`delay()`函数，它用于模拟一些 I/O 密集型工作，比如从数据库或网络上获取数据。

就像`Thread.sleep()`方法一样，它会使当前协程休眠。但与`Thread.sleep()`不同，其他协程可以在这个协程休眠时继续工作。这是因为`delay()`被标记为带有挂起关键字，我们将在“等待协程”部分讨论它。

如果你运行这段代码，你会看到使用协程的任务大约需要 200 毫秒，而使用线程要么需要 20 秒，要么耗尽内存。而且我们并没有对代码做太多修改。这都要归功于协程在本质上高度并发。它们可以在不阻塞运行它们的线程的情况下被挂起。不阻塞线程是件好事，因为我们可以用更少的操作系统线程（这些线程成本较高）来完成更多的工作。

当然，它们并不是神奇的。让我们为我们的协程创建一个**工厂**，它将能够生成短运行或长运行的协程：

```kt
object CoroutineFactory {
    fun greedyLongCoroutine(index: Int) = async {
        var uuid = UUID.randomUUID()
        for (i in 1..100_000) {
            val newUuid = UUID.randomUUID()

            if (newUuid < uuid) {
                uuid = newUuid
            }
        }

        println("Done greedyLongCoroutine $index")
        latch.countDown()
    }

    fun shortCoroutine(index: Int) = async {
        println("Done shortCoroutine $index!")
        latch.countDown()
    }
}
```

我们实际上并不需要**工厂方法**设计模式，但这是一个很好的提醒。你很快就会明白为什么长时间运行的协程被称为**贪婪**。

如果你忘记了工厂方法（Factory Method）是什么，你应该再次查看第二章，*使用创建型模式*，*工厂方法*部分。简而言之，它是一个返回对象的函数。在我们的情况下，它返回什么对象呢？当然是一个代表协程的工作！我们很快就会解释工作是什么。

# 工作

运行异步任务的结果被称为工作。就像`Thread`对象代表一个实际的操作系统线程一样，`job`对象代表一个实际的协程。工作有一个简单的生命周期。

它可以是以下两种情况之一：

+   新：已创建，但尚未开始。

+   活动：例如，刚刚由`launch()`函数创建。这是默认状态。

+   完成：一切顺利。

+   取消：出了点问题。

与有子工作的工作相关的还有两个状态：

+   完成：在完成之前等待子协程执行完毕

+   取消：在取消之前等待子协程执行完毕

如果你想要了解更多关于父工作和子工作，请跳转到本章的*父工作*部分。

工作还有一些有用的方法，我们将在接下来的章节中讨论。

# 协程饥饿

我们将分别调用`greedyLongCoroutine()`和`shortCoroutine()`方法 10 次，并等待它们完成：

```kt
val latch = CountDownLatch(10 * 2)
fun main(args: Array<String>) {

    for (i in 1..10) {
        CoroutineFactory.greedyLongCoroutine(i)
    }

    for (i in 1..10) {
        CoroutineFactory.shortCoroutine(i)
    }

    latch.await(10, TimeUnit.SECONDS)
}
```

显然，由于协程是异步的，我们首先会看到短协程的前 10 行，然后是长协程的前 10 行：

```kt
Done greedyLongCoroutine 2
Done greedyLongCoroutine 4
Done greedyLongCoroutine 3
Done greedyLongCoroutine 5
Done shortCoroutine 1! <= You should have finished long ago!
Done shortCoroutine 2!
Done shortCoroutine 3!
Done shortCoroutine 4!
Done shortCoroutine 5!
Done shortCoroutine 6!
Done shortCoroutine 7!
Done shortCoroutine 8!
Done shortCoroutine 9!
Done shortCoroutine 10!
Done greedyLongCoroutine 6
Done greedyLongCoroutine 7
Done greedyLongCoroutine 1
Done greedyLongCoroutine 8
Done greedyLongCoroutine 9
Done greedyLongCoroutine 10
```

哦，这不是你预期的结果。看起来长协程以某种方式阻塞了短协程。

这种行为的原因是协程背后仍然有一个基于线程池的事件循环。由于我的笔记本电脑有四个核心，四个长协程占用了所有资源，直到它们完成 CPU 密集型任务，其他协程无法开始。为了更好地理解这一点，让我们深入了解协程是如何工作的。

# 协程的内部机制

因此，我们提到了几次以下事实：

+   协程就像轻量级线程。它们需要的资源比常规线程少，因此你可以创建更多的它们。

+   协程在幕后使用线程池。

+   与阻塞整个线程不同，协程会挂起。

但这实际上是如何工作的呢？

让我们看看一个抽象的例子。我们该如何构建一个用户资料？

```kt
fun profile(id: String): Profile {
    val bio = fetchBioOverHttp(id) // takes 1s
    val picture = fetchPictureFromDB(id) // takes 100ms
    val friends = fetchFriendsFromDB(id) // takes 500ms
    return Profile(bio, picture)
}
```

总结起来，我们的函数现在完成大约需要 1.6 秒。

但我们已经了解了线程。让我们重构这个函数，使用它们来代替！

```kt
fun profile(id: String): Profile {
    val bio = fetchBioOverHttpThread(id) // still takes 1s
    val picture = fetchPictureFromDBThread(id) // still takes 100ms
    val friends = fetchFriendsFromDBThread(id) // still takes 500ms
    return Profile(bio, picture)
}
```

现在我们的函数平均需要 1 秒钟，这是三个请求中最慢的一个。但由于我们为每个请求创建了一个线程，我们的内存占用是原来的三倍。而且我们可能会很快耗尽内存。

因此，让我们使用线程池来限制内存占用：

```kt
fun profile(id: String): Profile {
    val bio = fetchBioOverHttpThreadPool()
    val picture = fetchPictureFromDBThreadPool()
    val friends = fetchFriendsFromDBThreadPool()
    return Profile(bio, picture)
}
```

但如果我们现在调用这个函数 100 次会发生什么呢？如果我们有一个包含 10 个线程的线程池，前 10 个请求将进入池中，第 11 个请求将卡住，直到第一个请求完成。这意味着我们可以同时服务三个用户，第四个用户将等待直到第一个用户得到他的/她的结果。

与协程相比，这有什么不同呢？协程将你的方法分解成更小的方法。

让我们深入了解其中一个函数，了解它是如何完成的：

```kt
fun fetchBioOverHttp(id: String): Bio {
    doSomething() // 50ms
    val result = httpCall() // 900ms
    return Bio(result) // 50ms
}
```

这是一个执行时间将达 1 秒的函数。

尽管如此，我们可以用`suspend`关键字标记`httpCall()`：

```kt
suspend fun httpCall(): Result { 
    ...
}
```

当 Kotlin 编译器看到这个关键字时，它知道它可以像这样拆分并重写函数：

```kt
fun fetchBioOverHttp(id: String): Bio {
   doSomething() // 50ms
   httpCall() { // It was marked as suspend, so I can rewrite it!
      callback(it)
   } // Thread is released after 50ms
}

// This will be called after 950ms
fun callback(httpResult: Result) {
   return Bio(httpResult) 
}
```

通过这样做重写，我们能够更早地释放执行协程的线程。

对于单个用户来说，这并不重要。他仍然会在 1 秒后得到结果。

但从更大的角度来看，这意味着通过使用相同数量的线程，我们可以服务 20 倍多的用户，这都要归功于 Kotlin 以智能的方式重写我们的代码。

# 解决饥饿问题

让我们在我们的工厂中使用扩展方法添加另一个方法：

```kt
fun CoroutineFactory.longCoroutine(index: Int) = launch {
    var uuid = UUID.randomUUID()
    for (i in 1..100_000) {
        val newUuid = UUID.randomUUID()

        if (newUuid < uuid) {
            uuid = newUuid
        }

        if (i % 100 == 0) {
            yield()
        }
    }

    println("Done longCoroutine $index")
    latch.countDown()
}
```

我们在第一个循环中调用这个方法：

```kt
...
for (i in 1..10) {
    CoroutineFactory.longCoroutine(i)
}
...
```

现在我们运行它，我们得到了最初预期的输出：

```kt
Done shortCoroutine 0!
Done shortCoroutine 1!
Done shortCoroutine 2!
Done shortCoroutine 3!
Done shortCoroutine 5!
Done shortCoroutine 6!
Done shortCoroutine 7!
Done shortCoroutine 8!
Done shortCoroutine 9!
Done shortCoroutine 4!
Done longCoroutine 4 <= That makes more sense
Done longCoroutine 2
Done longCoroutine 3
Done longCoroutine 9
Done longCoroutine 5
Done longCoroutine 1
Done longCoroutine 10
Done longCoroutine 6
Done longCoroutine 7
Done longCoroutine 8
```

现在我们来了解实际上发生了什么。我们使用了一个新的函数：`yield()`。我们本可以在每次循环迭代中都调用`yield()`，但决定每 100 次迭代才这样做。它会**询问**线程池是否还有其他人想要执行一些工作。如果没有其他人，当前协程的执行将恢复。否则，另一个协程将从它之前停止的地方开始或恢复。

注意，如果没有在函数或协程生成器（如`launch()`）上使用`suspend`关键字，我们无法调用`yield()`。这对于任何标记为`suspend`的函数都适用：它应该从另一个`suspend`函数或从协程中调用。

# 等待协程

到目前为止，为了让我们的异步代码完成，我们使用了`Thread.sleep()`或`CountDownLatch`。但线程和协程有更好的选择。与线程类似，一个任务也有`join()`函数。通过调用它，我们可以等待协程的执行完成。

看看下面的代码：

```kt
val j = launch(CommonPool) {
    for (i in 1..10_000) {
        if (i % 1000 == 0) {
            println(i)
            yield()
        }
    }
}
```

尽管它应该打印 10 行，但实际上并没有打印任何东西。这是因为我们的主线程在给协程一个开始的机会之前就终止了。

通过添加以下行，我们的示例将打印预期的结果：

```kt
runBlocking {
    j.join()
}
```

你问这个`runBlocking`是什么？记住，我们只能从另一个协程中调用`yield()`，因为它是一个**挂起函数**？`join()`也是同样的道理。由于我们的主方法不是协程，我们需要在我们的常规代码（即不是挂起函数）和协程之间有一个**桥梁**。这个函数正是这样做的。

# 取消协程

如果您是 Java 开发者，您可能知道停止线程相当复杂。

例如，`Thread.stop()`方法已被弃用。有`Thread.interrupt()`，但并非所有线程都会检查这个标志，更不用说设置自己的`volatile`标志了，这通常被建议，但非常繁琐。

如果您使用线程池，您将得到`Future`，它有`cancel(boolean mayInterruptIfRunning)`方法。在 Kotlin 中，`launch()`函数返回一个任务。

这个任务可以被取消。尽管如此，前一个示例中的规则仍然适用。如果您的协程从未调用另一个`suspend`方法或产生输出，它将忽略`cancel()`。

为了演示这一点，我们将创建一个偶尔产生输出的*良好*协程：

```kt
val cancellable = launch {
    try {
        for (i in 1..1000) {
            println("Cancellable: $i")
            computeNthFibonacci(i)
            yield()
        }
    }
    catch (e: CancellationException) {
        e.printStackTrace()
    }
}
```

另一个不产生输出的协程：

```kt
val notCancellable = launch {
    for (i in 1..1000) {
        println("Not cancellable $i")
        computeNthFibonacci(i)
    }
}
```

我们将尝试取消两者：

```kt
println("Canceling cancellable")
cancellable.cancel()
println("Canceling not cancellable")
notCancellable.cancel()
```

等待结果：

```kt
runBlocking {
    cancellable.join()
    notCancellable.join()
}
```

几个有趣的观点：

1.  取消*良好*协程不会立即发生。在取消之前，它可能还会打印一两行。

1.  我们可以捕获`CancellationException`，但无论如何，我们的协程将被标记为已取消。

# 返回结果

调用`launch()`就像调用返回`Unit`的函数一样。但我们的大多数函数都返回某种类型的结果。为此，我们有一个`async()`函数。它也会启动一个协程，但它返回的是`Deferred<T>`，其中`T`是您期望稍后获得的数据类型。

想象一下您想要从一个来源获取用户的配置文件，从另一个来源获取他们的历史记录的情况。这可能涉及两个数据库查询，或者是对两个远程服务的网络调用，或者任何组合。

您必须展示配置文件和历史记录，但您不知道哪个先返回。通常，检索配置文件更快。但有时可能会有延迟，因为配置文件经常更新，而历史记录会先返回。

我们运行一个协程，它将返回用户的配置文件字符串，在我们的例子中：

```kt
val userProfile = async {
    delay(Random().nextInt(100))
    "Profile"
}
```

我们将运行另一个来返回历史记录。为了简单起见，我们只返回一个整数列表：

```kt
val userHistory = async {
    delay(Random().nextInt(200))
    listOf(1, 2, 3)
}
```

为了等待结果，我们使用`await()`函数：

```kt
runBlocking {
    println("User profile is ${userProfile.await()} and his history is ${userHistory.await()}")
}

```

# 设置超时

如果，正如某些情况下发生的那样，获取用户配置文件花费的时间太长怎么办？如果我们决定如果配置文件返回超过 0.5 秒，我们就只显示*无配置文件*怎么办？

这可以通过使用`withTimeout()`函数来实现：

```kt
 val coroutine = async {
    withTimeout(500, TimeUnit.MILLISECONDS) {
        try {
            val time = Random().nextInt(1000)

            println("It will take me $time to do")

            delay(time)

            println("Returning profile")
            "Profile"
        }
        catch (e: TimeoutCancellationException) {
            e.printStackTrace()
        }
    }
}
```

我们将超时设置为 500 毫秒，并且我们的协程将在 0 到 1,000 毫秒之间延迟，给它 50%的失败机会。

我们将从协程中等待结果并看看会发生什么：

```kt
val result = try {
    coroutine.await()
}
catch (e: TimeoutCancellationException) {
    "No Profile"
}

println(result)
```

在这里，我们得益于`try`在 Kotlin 中是一个表达式的这一事实。因此，我们可以立即从它返回一个结果。

如果协程在超时之前成功返回，则`result`的值变为*配置文件*。否则，我们收到`TimeoutCancellationException`，并将`result`的值设置为*无配置文件*。

有趣的是，我们的协程总是接收到 `TimeoutCancellationException`，我们可以处理它。如果发生超时，*返回配置文件*将永远不会打印出来。

超时和 try-catch 表达式的组合是一个非常强大的工具，它允许我们创建健壮的交互。

# 父任务

如果我们想同时取消多个协程呢？这就是父任务发挥作用的地方。记住，`launch()` 接收 `CoroutineContext`，通常是 `CommonPool`？它还可以接收其他参数，我们很快就会看到。

我们将从一个工作一段时间的中断函数开始：

```kt
suspend fun produceBeautifulUuid(): String {
    try {
        val uuids = List(1000) {
            yield()
            UUID.randomUUID()
        }

        println("Coroutine done")
        return uuids.sorted().first().toString()
    } catch (t: CancellationException) {
        println("Got cancelled")
    }

    return ""
}
```

我们希望启动 10 个这样的协程，并在 100 毫秒后取消它们。

为了做到这一点，我们将使用一个父任务：

```kt
val parentJob = Job()

List(10) {
    async(CommonPool + parentJob) {
        produceBeautifulUuid()
    }
}

delay(100)
parentJob.cancel()
delay(1000) // Wait some more time
```

如你所见，父任务只是一个任务。我们将其传递给 `async()` 函数。我们可以使用 `+` 号，因为 `CoroutineContext` 覆盖了 `plus()` 函数。你也可以使用命名参数来指定它：

```kt
async(CommonPool, parent= parentJob)
```

一旦我们在父任务上调用 `cancel()`，它的所有子任务也会被取消。

# 通道

到目前为止，我们学习了如何生成协程并控制它们。但如果有两个协程需要相互通信呢？

在 Java 中，线程通过使用 `wait()`/`notify()`/`notifyAll()` 模式或使用 java.util.concurrent 包中的一组丰富的类来通信。例如：`BlockingQueue` 或 `Exchanger`。

在 Kotlin 中，正如你可能已经注意到的，没有 `wait()`/`notify()` 方法。但是有通道，它们与 `BlockingQueue` 非常相似。但是，通道不是阻塞线程，而是挂起协程，这要便宜得多。

为了更好地理解通道，让我们创建一个简单的两人游戏，他们将会互相投掷随机数。如果你的数字更大，你就赢了。否则，你将输掉这一轮：

```kt
fun player(name: String,
           input: Channel<Int>,
           output: Channel<Int>) = launch {
    for (m in input) {
        val d = Random().nextInt(100)
        println("$name got $m, ${if (d > m) "won" else "lost" }")

        delay(d)
        output.send(d)
    }
}
```

每个玩家有两个通道。一个用于接收数据，另一个用于发送数据。

我们可以使用常规的 for 循环遍历通道，它将挂起，直到接收到下一个值。

当我们想要将结果发送给另一玩家时，我们只需使用 `send()` 方法。

现在让我们玩一秒钟这个游戏：

```kt
fun main(vararg args: String) {
    val p1p2 = Channel<Int>()
    val p2p1 = Channel<Int>()

    val player1 = player("Player 1", p2p1, p1p2)
    val player2 = player("Player 2", p1p2, p2p1)

    runBlocking {
        p2p1.send(0)
        delay(1000)
    }
}
```

我们的结果可能看起来像这样：

```kt
...
Player 1 got 62, won
Player 2 got 65, lost
Player 1 got 29, lost
Player 2 got 9, won
Player 1 got 46, won
Player 2 got 82, lost
Player 1 got 81, lost
...
```

如你所见，通道是不同协程之间通信的一种方便且类型安全的途径。但我们不得不手动定义通道，并按正确的顺序传递它们。在接下来的两个部分中，我们将看到如何进一步简化这一点。

# 生产者

在 第七章，*保持响应性*，该章节专门讨论了响应式编程，我们讨论了产生值流的 `Observable` 和 `subject`。同样，Kotlin 也为我们提供了 `produce()` 函数。

此函数创建的协程由 `ReceiveChannel<T>` 支持，其中 `T` 是协程产生的类型：

```kt
val publisher: ReceiveChannel<Int> = produce {
        for (i in 2018 downTo 1970) { // Years back to Unix
            send(i)
            delay(20)
        }
}
```

在 Rx 中，有我们在 第七章 中介绍的 `onNext()` 方法，*保持响应性*。

生产者有一个 `send()` 函数，它与它非常相似。

与提供了 `subscribe()` 方法的 Rx `Observable` 类似，这个通道有 `consumeEach()` 函数：

```kt
publisher.consumeEach {
    println("Got $it")
}
```

它将打印以下内容：

```kt
Got 35
Got 34
Got 33
Got 32
Got 31
Got 30
Got 29
```

通道提供的另一个伟大能力是 `select()`。

如果我们有多个生产者，我们可以 `subscribe` 到它们的通道，并获取第一个可用的结果：

```kt
val firstProducer = produce<String> {
    delay(Random().nextInt(100))
    send("First")
}

val secondProducer = produce<String> {
    delay(Random().nextInt(100))
    send("Second")
}

val winner = select<String> {
    firstProducer.onReceive {
        it.toLowerCase()
    }
    secondProducer.onReceive {
        it.toUpperCase()
    }
}

println(winner)
```

这将随机打印 "First" 或 "Second"。

注意，`select()` 只发生一次。一个常见的错误是将 `select()` 应用在两个生成数据流的协程上，而没有将其包含在循环中：

```kt
// Producer 1
val firstProducer = produce {
    for (c in 'a'..'z') {
        delay(Random().nextInt(100))
        send(c.toString())
    }

}

// Producer 2
val secondProducer = produce {
    for (c in 'A'..'Z') {
        delay(Random().nextInt(100))
        send(c.toString())
    }
}

// Receiver
println(select<String> {
    firstProducer.onReceive {
        it
    }
    secondProducer.onReceive {
        it
    }
})
```

与打印字母表不同，这将只打印 "a" 或 "A"，然后退出。请确保你的 `select()` 被包含在一个循环中。

这将打印它接收到的第一个 10 个字符：

```kt
// Receiver
for (i in 1..10) {
    println(select<String> {
        firstProducer.onReceive {
            it
        }
        secondProducer.onReceive {
            it
        }
    })
}
```

另一个选项是使用 `close()` 函数来发送信号：

```kt
// Producer 2
val secondProducer = produce {
    for (c in 'A'..'Z') {
        delay(Random().nextInt(100))
        send(c.toString())
    }
    close()
}
```

在接收器内部使用 `onReceiveOrNull()`：

```kt
// Receiver
while(true) {
    val result = select<String?> {
        firstProducer.onReceiveOrNull {
            it
        }
        secondProducer.onReceiveOrNull {
            it
        }
    }

    if (result == null) {
        break
    }
    else {
```

```kt
        println(result)
    }
}
```

此选项将打印字符，直到第一个生产者决定关闭通道。

# 演员

本章最后介绍的是演员。与 `producer()` 类似，`actor()` 是一个与通道绑定的协程。但与通道从协程 *出去* 不同，这里有一个通道进入协程。如果你觉得这太学术了，请继续阅读，以获得另一种解释。

那么，演员究竟是什么呢？让我们看看迈克尔和我之间的互动，一个想象中的产品经理，你可能还记得从 第四章 "熟悉行为模式"，他碰巧是一只金丝雀。迈克尔有一系列需要在冲刺/周/月结束前完成的任务。他只是把它们扔给我，希望我能施展魔法，将一些模糊的规格翻译成可工作的代码。他并不等待我的回复。他只是期望这最终会发生——而且越快越好。对迈克尔来说，我是一个演员。不是因为我去过戏剧学校，而是因为我根据他的要求采取行动。

如果你使用过 Scala 或其他具有演员的编程语言，你可能熟悉与我们描述的略有不同的演员模型。在一些实现中，演员既有入站通道也有出站通道（通常称为邮箱）。但在 Kotlin 中，一个演员只有一个入站邮箱。

要创建一个新的演员，我们使用 `actor()` 函数：

```kt
data class Task (val description: String)
val me = actor<Task> {
    while (!isClosedForReceive) {
        println(receive().description.repeat(10))
    }
}
```

注意，与 `select()` 的工作方式相同，除非我们将演员的 `receive()` 包含在某种循环中，否则它只会执行一次。如果你尝试将其发送到已关闭的通道，你会得到 `ClosedSendChannelException`。

你使用 `send()` 与演员进行通信：

```kt
// Imagine this is Michael the PM
fun michael(actor: SendChannel<Task>) {

    runBlocking {
        // He has some range of tasks
        for (i in 'a'..'z') {
            // That he's sending to me
            actor.send(Task(i.toString()))
        }
        // And when he's done with the list, he let's me know
        actor.close()
        // That doesn't mean I'm done working on it, though
    }
}

// And he's calling me
michael(me)
```

对于演员来说，另一种模式是使用 `receiveOrNull()` 函数：

```kt
val meAgain = actor<Task> {
    var next = receiveOrNull()

    while (next != null) {
        println(next.description.toUpperCase())
        next = receiveOrNull()
    }
}

// Michael still can call me in the same manner
michael(meAgain)

```

如您所见，我们不是检查演员的通道是否已关闭，而是在通道上接收到了 null。如果演员从许多 *管理者* 那里接收任务，这种方法可能更可取。

第三种选项，通常是首选，是遍历通道：

```kt
val meWithRange = actor<Task> {
    for (t in channel) {
        println(t.description)
    }

    println("Done everything")
```

```kt
}

michael(meWithRange)
```

如您所见，这是三种实现中最干净的一种。

演员对于需要维护某种状态的后台任务非常有用。例如，你可以创建一个生成报告的演员。它将接收要生成哪种类型的报告，并确保同一时间只生成一个报告：

```kt
data class ReportRequest(val name: String,
                                 val from: LocalDate,
                                 val to: LocalDate)
val reportsActor = actor<ReportRequest>(capacity=100) {
    for (req in this) {
        generateReport(req)
    }
}
```

限制演员可以接收的消息容量通常是一个好主意。

然后，我们可以发送给这个演员要生成哪种类型的报告：

```kt
reportsActor.send(ReportRequest("Monthly Report",
        LocalDate.of(2018, 1, 1),
        LocalDate.of(2018, 1, 31)))
```

# 摘要

在本章中，我们介绍了如何在 Kotlin 中创建线程和协程，以及协程的好处。

与 Java 相比，Kotlin 创建线程的语法简化了。但它们仍然有内存和性能开销。协程能够解决这些问题；在需要并发执行某些代码时，请使用协程。

如果你想在两个协程之间进行通信，请使用通道。

Kotlin 还提供了`actor()`函数来创建演员，该函数也会启动一个带有附加输入流的协程来处理事件。如果你需要创建一个值流，可以使用`produce()`函数。

在下一章中，我们将讨论如何使用这些并发原语来创建适合我们需求的可扩展和健壮的系统。
