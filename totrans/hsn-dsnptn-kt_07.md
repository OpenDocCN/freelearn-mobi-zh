# 保持响应式

一旦我们熟悉了函数式编程及其构建块，我们就可以开始讨论响应式编程的概念。虽然它并不与函数式编程耦合（你甚至可以在编写面向对象或过程式代码时变得响应式），但在了解函数式编程及其基础之后讨论会更好。

在本章中，我们将涵盖以下主题：

+   响应式原则

+   响应式扩展

# 响应式原则

那么，什么是响应式编程呢？

响应式宣言很好地总结了这一点：[`www.reactivemanifesto.org`](https://www.reactivemanifesto.org)。

引用它，响应式程序是：

+   响应性

+   弹性

+   弹性

+   消息驱动

为了理解这四个主题，让我们想象有 10 个人站在收银员队伍中。他们中的每一个人只能看到前面的人，但看不到前面有多少人在排队，或者收银员在做什么。你脑海中有没有这个画面？那么，我们就从这里开始。

# 响应性

你会站在那个收银员队伍中吗？

这取决于紧急程度和你的时间。如果你很匆忙，你可能会在到达收银台之前空手而归。

这是一种系统对你不响应的情况。当你通过电话联系某家服务提供商的客户服务中心时，你经常会遇到这种情况。你会被要求在电话线上等待，然后你就等待。但是，更常见的情况是，一个友好的自动语音会告诉你，在你前面有多少人在等待，甚至告诉你你需要等待多长时间。

在这两种情况下，结果都是一样的。你在排队或电话线上浪费了时间。但第二个系统对你的需求做出了响应，你可以据此做出决定。

# 弹性

让我们继续讨论弹性。你已经在电话线上等了 10 分钟，然后队伍消失了。或者，你联系到了一位客户服务代表，但他们不小心挂断了你的电话。这种情况有多常见？这就是系统对失败不具弹性的表现。或者，你排队等了半小时去看医生，但他们突然离开办公室去打高尔夫球，让你明天再来。这是一个面对失败时没有响应的系统。

响应式宣言讨论了实现弹性的各种方法：

+   委派

+   复制

+   隔离

+   隔离

委派是指医生走出办公室告诉你，“我今天不能见你，但敲一下另一扇门；他们很快就会见你。”

复制是为了诊所总是有两名医生可用，以防其中一名医生今晚错过了他们最喜欢的球队比赛。这与弹性有关，我们将在下一节讨论。

限制和隔离通常一起讨论。如果你实际上不需要看医生呢？也许你只需要他们的处方。那么，你可以给他们留条消息（我们很快会讨论消息传递，因为它也是反应性的一个重要部分），当他们游戏间隙时，他们会给你发送处方。你从看医生的需求中解脱出来。这也让你从医生的失败或问题中得到了隔离。你不知道的是，在他们打印处方的时候，他们的电脑两次崩溃，他们对此非常焦虑。但因为你不在他们面前，他们把这件事留给了自己。

# 弹性

所以，在前一节中，我们讨论了复制。为了防止失败，我们的诊所总是有两个医生可用。也许第二位医生服务了一些病人，或者他们只是耐心地等待第一位医生离开去踢足球比赛开始工作。

但是，如果突然爆发流感疫情或一群狂怒的松鼠开始在附近的公园攻击市民，那么那个弹性系统会发生什么？两位医生将无法处理所有病人，然后，我们再次面临弹性问题。

但如果我们有一批退休医生坐在家里打麻将呢？当然，我们可以叫他们来帮助包扎所有那些松鼠受害者。在他们都得到适当治疗后，医生们可以回到他们的麻将桌。

这是一个根据工作量而弹性的系统。

弹性建立在可扩展性之上。我们之所以能够治疗所有那些病人，是因为每位医生可以独立工作。但如果所有的绷带都存放在一个盒子里呢？那么就会形成瓶颈，所有医生都站在那里等待下一包绷带。

# 消息驱动

这也被称为*异步消息传递*。所以，我们在*弹性*部分看到，如果你能给医生留条消息，这可能会使系统更具弹性。

如果所有的病人都只留下消息呢？那么每位医生都可以优先处理它们或批量处理这些消息。例如，一起打印所有处方，而不是在不同任务之间切换。

除了松散耦合和隔离之外，还有*位置透明性*。你不知道你的医生是在开车回家的路上给你发送这个处方的（在你留言的时候，他们从窗户溜出去）。但你并不在乎，因为你得到了你想要的东西。

使用消息还允许一个有趣的选项，即*背压*。如果医生收到的消息太多，他们可能会因为压力而崩溃。为了避免这种情况，他们可能会给你发短信说你需要等待更长一点时间才能收到处方。或者，如果他们有秘书，我们甚至可以要求他们这样做。再次强调，我们在这里讨论的是委托，因为所有这些原则都是相关的。

消息也是非阻塞的。在您离开消息后，您不必等待医生的回复。您通常会回家，继续您的常规任务。在等待时执行其他任务的能力是并发的一个基石。

# 反应式扩展

本章的其余部分将致力于 Kotlin 中反应式原则的具体实现。这个领域的首选库是 RxJava。由于 Kotlin 与 Java 库完全兼容，RxKotlin 只是原始 RxJava 的一层薄包装。因此，我们将将其视为同一个库，并突出显示任何差异。

一旦我们开始讨论 RxJava，您就会认出它是建立在我们在 第四章 中讨论的 **观察者** 设计模式之上的。

我们将首先在我们的 Gradle 项目中添加以下依赖项：

```kt
compile "io.reactivex.rxjava2:rxjava:2.1.14"
```

目前，这是 RxJava2 的最新版本，但当你阅读这一章时，可能已经有了更新的版本。请随意使用它。

您可能还记得，该模式由两个对象组成：

+   `发布者`：产生数据

+   `订阅者`：消费数据

在 RxJava 中，发布者被称为 `Observable`。

以下代码将创建我们的第一个发布者：

```kt
val publisher = Observable.fromArray(5, 6, 7)
```

要开始消费这些数字，我们可以向 `subscribe()` 函数提供一个 lambda 表达式：

```kt
publisher.subscribe {
    println(it)
} // Prints 1, 2, 3
```

`Observable` 上还有其他可用的函数，您会立即认出：例如 `map()` 和 `filter()`。这些函数与 Kotlin 中常规数组上的函数相同：

```kt
publisher.filter {
    it > 5
}.map {
    it + it
}.subscribe {
    println(it)
}
```

好的，这很好，但我们已经在上一章讨论了序列中的集合和流。为什么还要再次讨论？

让我们看看以下示例：

```kt
val publisher = Observable.interval(1, TimeUnit.SECONDS)

publisher.subscribe {
    println("P1 $it")
}

publisher.subscribe {
    println("P2 $it")
}

Thread.sleep(TimeUnit.SECONDS.toMillis(5))
```

此代码将在终止前等待五毫秒，并打印以下内容：

```kt
Sleeping <= This was the last line in our code, actually
P2 0     <= P2 came after P1 in code, but it comes before now
P1 0
P2 1
P1 1
P2 2
P1 2
```

这是不预期的。`Sleeping` 是代码中的最后一行，但它首先被打印出来。然后注意，如果您多次运行此示例，有时 `P2` 会先于 `P1` 打印。有时，`P1` 会先于 `P2` 打印，这与代码中的情况相似。这里发生了什么？

这就是异步操作的实际应用。我们需要在这里使用 `Thread.sleep()` 以允许我们的监听器运行一段时间，否则我们的程序将终止。并且当它们被调用时，它们被放置在代码中的位置无关紧要。

在本章中，我们将大量使用 `Thread.sleep()` 和 `CountDownLatch` 来演示异步操作的工作原理。在实际应用中，您永远不应该使用 `Thread.sleep()`。如果您还不熟悉 `CountDownLatch`，不要担心，我们将在第一次遇到它时在 *Flowables* 部分解释它的工作原理。

好吧，这就是观察者设计模式自然的行为。但观察者还有一个取消订阅的选项。我们在这里是如何实现的？

让我们用以下代码替换第二个监听器：

```kt
...
val subscription = publisher.subscribe {
    println("P2 $it")
}

println("Sleeping")
Thread.sleep(TimeUnit.SECONDS.toMillis(2))
subscription.dispose()
...
```

调用 `subscribe()` 返回一个 `Disposable`。当你不再想接收更新时，你可以调用它上的 `dispose()`，这与 *取消订阅* 同义。

你的输出可能看起来像这样：

```kt
Sleeping
P1 0        <= Notice that P1 is the first one now
P2 0
P1 1
Sleeping    <= This is after dispose/unsubscribe
P2 1        <= But it may still take some time, so P2 prints again
P1 2 
P1 3
P1 4        <= No more prints from P2, it unsubscribed
```

如果我们要创建自己的 `Observable`，具有它自己的特定逻辑怎么办？有一个 `create()` 方法可以做到这一点：

```kt
val o = Observable.create<Int> {
    for (i in 1..10_000) {
        it.onNext(i)
    }
    it.onComplete()
}
```

我们创建一个发布数字的 `Observable`。要向所有听众推送新值，我们使用 `onNext()` 方法。我们通过 `onComplete()` 通知听众没有更多数据了。最后，如果发生错误，我们可以调用 `onError()`，并提供异常作为参数。

你会注意到，如果我们尝试实际调用 `onError()`，我们会得到一个异常：

```kt
val o = Observable.create<Int> {
    it.onError(RuntimeException())
}

o.subscribe {
    println("All went good: $it")
} // OnErrorNotImplementedException
```

这是因为我们使用了带有 lambda 监听器的简写形式。

如果我们想要正确处理错误，我们还需要提供 *错误处理器* 作为第二个参数：

```kt
o.subscribe({
    println("All went good: $it")
}, {
    println("There was an error $it")
})
```

还有一个第三个参数，即 `onComplete` 处理器：

```kt
o.subscribe({
    println("All went good: $it")
}, {
    println("There was an error $it")
}, {
    println("Publisher closed the stream")
})
```

在我们的示例中，我们很少使用错误处理器，因为我们的代码非常基础。但在实际应用中，你应该始终提供它们。

# 热观察者

热 `Observable` 是我们在本章中会大量使用的一个术语，与冷 `Observable` 相对。我们之前讨论的所有 `Observable` 都是冷的。这意味着它们知道从时间开始发生的一切，每次有人礼貌地询问时，它们都可以重复整个历史。热 `Observable` 只知道现在发生的事情。例如，想想天气预报和天气历史。天气预报是热的——你将每分钟得到当前的天气。天气历史是冷的——如果你关心它，你可以得到整个天气变化的历史。如果你仍然不理解这个概念，不要过于担心。我们还有一半的章节要覆盖它。

如你可能已经注意到的，到目前为止，所有我们的订阅者总是得到所有数据，无论他们何时订阅：

```kt
publisher.subscribe {
    println("S1 $it")
} // Prints 10K times

publisher.subscribe {
    println("S2 $it")
} // Also prints 10K times
```

但情况并不总是这样。更常见的是，数据源来自外部，而不是每次都由 `publisher` 创建：

```kt
val iterator = (1..10).iterator()

val publisher = Observable.create<Int> {
    while (iterator.hasNext()) {
        val nextNumber = iterator.next()
        it.onNext(nextNumber)
    }
}
```

在这里，我们不是在内部创建列表，而是有一个对其迭代器的引用。

让我们看看以下代码现在是如何表现的：

```kt
publisher.subscribeOn(Schedulers.newThread()).subscribe {
    println("S1: $it")
    Thread.sleep(10)
}

Thread.sleep(50)

publisher.subscribeOn(Schedulers.newThread()).subscribe {
    println("S2: $it")
    Thread.sleep(10)
}

Thread.sleep(50)
```

我们有两个订阅者，就像之前一样。到目前为止，所有订阅者都在我们运行的同一个线程上执行。对于这个例子，我们为每个订阅者分配了一个单独的线程。这将允许我们模拟运行一段时间（在这种情况下是 10 毫秒）的操作。要指定订阅者应该在哪个线程上运行，我们使用 `subscribeOn()`。`Schedulers` 是一个实用类，类似于 Java 5 中的 `Executors`。在这种情况下，它将为每个听众分配一个新的线程。

输出可能看起来像这样：

```kt
S1: 1
S1: 2
S1: 3
S1: 4
S1: 5
S2: 6 <= That's where "Subscriber 2" begins listening
S1: 7
S2: 8
S1: 9
S2: 10
```

注意，如果每个消费者之前都接收到了所有数据，现在第二个订阅者将永远不会接收到数字 1-5。

在第二个订阅者连接后，每次只有其中之一会接收到数据。

如果我们想同时向所有订阅者发布数据怎么办？

# 多播

有一个 `publish()` 方法可以做到这一点：

```kt
val iterator = (1..5).iterator()
val subject = Observable.create<Int> {
    while (iterator.hasNext()) {
        val number = iterator.nextInt()
        println("P: $number")
        it.onNext(number)
        Thread.sleep(10)
    }
}.observeOn(Schedulers.newThread()).publish()
```

我们再次创建了一个稍微热的`Observable`，但这次我们指定它将在单独的线程上运行，使用`observeOn()`。我们还使用了`publish()`方法，它将我们的`Observable`转换为`ConnectableObservable`。

如果我们仅仅订阅这种类型的`Observable`，将不会发生任何事情。我们需要告诉它何时开始运行。我们使用`connect()`方法来做这件事。由于`connect()`方法是阻塞的，我们将在这个例子中从单独的线程执行它：

```kt
thread { // Connect is blocking, so we run in on another thread
    subject.connect() // Tells observer when to start
}
```

现在，我们将让发布者工作几毫秒，然后连接我们的第一个监听器：

```kt
Thread.sleep(10)
println("S1 Subscribes")
subject.subscribeOn(Schedulers.newThread()).subscribe {
    println("S1: $it")
    Thread.sleep(100)
}

```

经过一段时间后，我们连接第二个监听器，并允许它们完成：

```kt
Thread.sleep(20)

println("S2 Subscribes")
subject.subscribeOn(Schedulers.newThread()).subscribe {
    println("S2: $it")
    Thread.sleep(100)
}
Thread.sleep(2000)
```

让我们看看现在的输出，因为它相当有趣：

```kt
P: 1 *<= Publisher starts publishing even before someone subscribes*
*S1 Subscribes*
P: 2
P: 3
S1: 3 *<= Subscriber actually missed some values*
*S2 Subscribes*
P: 4
P: 5
P: 6 *<= Publisher completes here*
S1: 4
S2: 4 
S1: 5
S2: 5 *<= Both subscribers receive same values*
```

当然，拥有这个`connect()`并不总是舒适的。

因此，我们有一个名为`refCount()`的方法，它将我们的`ConnectableObservable`转换回常规的`Observable`。它将保持订阅者的引用计数，并且只有在所有订阅者都这样做之后才会取消订阅：

```kt
// This is a connectable Observable
val connectableSource = Observable.fromIterable((1..3)).publish()

// Should call connect() on it
dataSource.connect()

// This is regular Observable which wraps ConnectableObservable
val regularSource = connectableSource.refCount()

regularSource.connect() // Doesn't compile
```

如果调用`publish().refCount()`太繁琐，还有一个`share()`方法可以做到这一点：

```kt
val regularSource = Observable.fromIterable((1..3)).publish().refCount()

val stillRegular = Observable.fromIterable((1..3)).share()
```

# Subject

理解`Subject`的最简单方式是`Subject = Observable + Observer`。

一方面，它允许其他人`subscribe()`订阅它。另一方面，它可以`subscribe`到其他`Observable`：

```kt
val dataSource = Observable.fromIterable((1..3))

val multicast = PublishSubject.create<Int>()

multicast.subscribe {
    println("S1 $it")
}

multicast.subscribe {
    println("S2 $it")
}

dataSource.subscribe(multicast)

Thread.sleep(1000)
```

以下代码将打印六行，每个订阅者三行：

```kt
S1 1
S2 1
S1 2
S2 2
S1 3
S2 3
```

注意，我们没有在`dataSource`上使用`publish()`，所以它是冷的。冷意味着每次有人订阅这个源时，它都会重新开始发送数据。另一方面，热的`Observable`并没有所有数据，它只会从这一刻开始发送它所拥有的数据。

因此，我们需要首先连接所有监听器，然后才开始监听`dataSource`。

如果我们使用热的`dataSource`，我们可以切换调用：

```kt
val dataSource = Observable.fromIterable((1..3)).publish()

val multicast = PublishSubject.create<Int>()

dataSource.subscribe(multicast)

multicast.subscribe {
    println("S1 $it")
}
println("S1 subscribed")

multicast.subscribe {
    println("S2 $it")
}
println("S2 subscribed")

dataSource.connect()

Thread.sleep(1000)
```

如前所述，我们使用`connect()`来告诉`dataSource`何时开始发射数据。

# ReplaySubject

除了我们在上一节中讨论的`PublishSubject`之外，还有其他主题可用。为了理解`ReplaySubject`是如何工作的，让我们首先看看以下使用`PublishSubject`的例子：

```kt
val list = (8..23).toList() // Some non trivial numbers
val iterator = list.iterator()
val o = Observable.intervalRange(0, list.size.toLong(), 0, 10, TimeUnit.MILLISECONDS).map {
    iterator.next()
}.publish()

val subject = PublishSubject.create<Int>()

o.subscribe(subject)

o.connect() // Start publishing

Thread.sleep(20)

println("S1 subscribes")
    subject.subscribe {
        println("S1 $it")
    }
    println("S1 subscribed")

    Thread.sleep(10)

    println("S2 subscribes")
    subject.subscribe {
        println("S2 $it")
    }
    println("S2 subscribed")

    Thread.sleep(1000)
```

这将打印以下内容：

```kt
S1 11 <= Lost 8, 9, 10
S1 12
S2 12 <= Lost also 11
S1 13
S2 13
...
```

显然，一些事件已经永久丢失。

现在，让我们用`ReplaySubject`替换`PublishSubject`并检查输出：

```kt
val subject = ReplaySubject.create<Int>()
```

以下输出将被打印：

```kt
S1 subscribes
S1 8
S1 9
S1 10 <= S1 catchup 
S1 subscribed
S1 11
S1 12
S2 subscribes
S2 8
S2 9
S2 10
S2 11
S2 12 <= S2 catchup 
S2 subscribed
S1 13 <= Regular multicast from here
S2 13
...
```

使用`ReplaySubject`，不会丢失任何事件。然而，从输出中可以看出，直到某个点，事件不会多播，即使有多个`subscriber`。相反，对于每个`subscriber`，`ReplaySubject`会执行一种追赶，以弥补到目前为止错过的事件。

这种方法的优点是显而易见的。我们将看似热的`Observable`转换成了相当冷的。但也有一些限制。通过使用`ReplaySubject.create`，我们产生了一个无界的主题。如果它尝试记录太多事件，我们可能会简单地耗尽内存。为了避免这种情况，我们可以使用`createWithSize()`方法：

```kt
val subject = ReplaySubject.createWithSize<Int>(2)
```

它创建了以下输出：

```kt
S1 subscribes
S1 9 <= lost 8
S1 10
S1 subscribed
S1 11
S2 subscribes
S1 12
S2 11 <= lost 8, 9, 10
S2 12
S2 subscribed
S1 13
S2 13
...
```

如您所见，现在我们的主题记住的项目更少，因此最早的事件丢失了。

# `BehaviorSubject`

想象一下这种情况：你每分钟都会有一串更新。你想要显示你收到的最新值，然后在有新数据进来时继续更新它。你可以使用大小为 1 的`ReplaySubject`。但还有`BehaviorSubject`正好适用于这种情况：

```kt
val subject = BehaviorSubject.create<Int>()
```

输出结果如下：

```kt
S1 subscribes
S1 10 <= This was the most recent value, 8 and 9 are lost
S1 subscribed
S1 11 <= First update 
S2 subscribes
S2 11 <= This was most recent value, 8, 9 and 10 lost
S2 subscribed
S1 12 <= As usual from here
S2 12 
```

# `AsyncSubject`

这是一个奇怪的`subject`，因为它与其他不同，它不会更新其订阅者。那么它有什么好处呢？

如果你想有一个非常基本的功能，只是更新屏幕上的最新值，并且直到屏幕关闭不再刷新：

```kt
val subject = AsyncSubject.create<Int>()
```

这里是输出结果：

```kt
S1 subscribes
S1 subscribed
S2 subscribes
S2 subscribed
S1 23 <= This is the final value
S2 23
```

但是要小心。由于`AsyncSubject`等待序列完成，如果序列是无限的，它将永远不会调用其订阅者：

```kt
// Infinite sequence of 1
val o = Observable.generate<Int> { 1 }.publish()
...
o.connect() // Hangs here forever
```

# `SerializedSubject`

重要的是不要从不同的线程调用`onNext()`/`onComplete()`/`onError()`方法，因为这会使调用非序列化。

这是一个某种形式的**代理**，围绕任何常规`subject`，它同步调用不安全的方法。你可以使用`toSerialized()`方法将任何`subject`包装在`SerializedSubject`中：

```kt
val subject = ReplaySubject.createWithSize<Int>(2).toSerialized()
```

# `Flowables`

在所有之前的例子中，我们使用`Observable`或`subject`来发射数据，它们也扩展了`Observable`，并且效果相当不错。

但我们的听众并没有做什么。如果他们要做更多实质性的事情会怎样？

让我们看看以下示例。我们将生成大量的唯一字符串：

```kt
val source = Observable.create<String> {
    var startProducing = System.currentTimeMillis()
    for (i in 1..10_000_000) {
        it.onNext(UUID.randomUUID().toString())

        if (i % 100_000 == 0) {
            println("Produced $i events in ${System.currentTimeMillis() - startProducing}ms")
            startProducing = System.currentTimeMillis()
        }
    }
    latch.countDown()
}
```

我们使用`CountDownLatch`以便主线程能够等待我们完成。此外，我们还打印了发射 100,000 个事件所需的时间。这将在以后很有用。

在`subscribe()`方法中，我们将重复这些字符串 1,000 次：

```kt
val counter = AtomicInteger(0)
source.observeOn(Schedulers.newThread())
        .subscribe( {
            it.repeat(500)
            if (counter.incrementAndGet() % 100_000 == 0) {
                println("Consumed ${counter.get()} events")
            }
        }, {
            println(it)
        })
```

`AtomicInteger`用于以线程安全的方式计数线程中处理的事件的数量。

我们显然消费的速度比生产慢：

```kt
Produced 100000 events in 1116ms
Produced 200000 events in 595ms
Produced 300000 events in 734ms
Consumed 100000 events
Produced 400000 events in 815ms
Produced 500000 events in 705ms
Consumed 200000 events
Produced 600000 events in 537ms
Produced 700000 events in 390ms
Produced 800000 events in 529ms
Produced 900000 events in 387ms
Consumed 300000 events
Produced 1000000 events in 531ms
Produced 1100000 events in 537ms
Produced 1200000 events in 11241ms <= What happens here?
Consumed 400000 events
Produced 1300000 events in 19472ms
Produced 1400000 events in 31993ms
Produced 1500000 events in 52650ms
```

但有趣的是，在一段时间之后，生产时间将急剧增加。

这是我们开始耗尽内存的点。现在，让我们用`Flowable`替换我们的`Observable`：

```kt
val source = Flowable.create<String> ({
    var startProducing = System.currentTimeMillis()
    for (i in 1..10_000_000) {
        it.onNext(UUID.randomUUID().toString())

        if (i % 100_000 == 0) {
            println("Produced $i events in ${System.currentTimeMillis() - startProducing}ms")
            startProducing = System.currentTimeMillis()
        }
    }
    it.onComplete()
    latch.countDown()
}, BackpressureStrategy.DROP)
```

如您所见，我们不仅传递了一个 lambda 表达式，还传递了第二个参数，即`BackpressureStrategy`。幕后发生的事情是`Flowable`有一个有界缓冲区。这与我们如何使`ReplaySubject`有界非常相似。第二个参数告诉`Flowable`当缓冲区达到限制时应该发生什么。在这种情况下，我们要求它丢弃这些事件。

现在，我们应该检查我们输出的最后部分：

```kt
...
Produced 9500000 events in 375ms
Produced 9600000 events in 344ms
Produced 9700000 events in 344ms
Consumed 2800000 events
Produced 9800000 events in 351ms
Produced 9900000 events in 333ms
Produced 10000000 events in 340ms
```

首先，请注意，我们没有在任何地方卡住。实际上，我们的生产速度是恒定的。

第二，你应该注意，尽管我们*生产*了 1,000 万次事件，但我们*消费*了只有 280 万次。其他所有事件都被丢弃了。

但我们没有耗尽内存，这是`Flowable`的巨大好处。

如果你确实想让`Flowable`表现得像`Observable`，你可以指定`BackpressureStrategy.BUFFER`，并看到它开始在这些行周围出现卡顿。

作为一般指南，当以下情况发生时使用`Flowable`：

+   你计划发射超过 1,000 个项目（有些人可能会说 10,000）

+   你正在读取文件

+   你正在查询数据库

+   你有一些网络流需要处理

如此使用`Observable`：

+   你计划发射有限数量的数据。

+   你处理用户输入。人类并不像他们想象的那么快，也不产生很多事件。

+   你关心流的性能：`Observable`更简单，因此更快。

当我们使用 lambda 表达式时，我们没有在`Flowable`和`Observable`之间注意到太大的区别。

取而代之，现在我们将使用匿名类，并看看这种方法提供了哪些好处：

```kt
source.observeOn(Schedulers.newThread())
        .subscribe(object : Subscriber<String> {
    lateinit var subscription: Subscription

    override fun onSubscribe(s: Subscription?) {
        s?.let {
            this.subscription = it
        } ?: throw RuntimeException()
    }

    override fun onNext(t: String?) {
        ...
    }

    override fun onError(t: Throwable?) {
        ...
    }

    override fun onComplete() {
        ...
    }
})
```

这显然是更多的代码。现在我们需要实现四个方法。

我们最感兴趣的是`onSubscribe()`方法。在这里，我们接收一个新的对象，称为`Subscription`，并将其存储在一个属性中。

现在，我们将丢弃我们在监听器之前使用的花哨代码，并简单地打印我们接收到的每个新字符串：

```kt
override fun onNext(t: String?) {
    println(t)
}
```

嗯？这很奇怪。我们的监听器没有打印任何东西。

让我们进入我们的`onSubscribe`并稍作修改：

```kt
override fun onSubscribe(s: Subscription) {
    this.subscription = s
    this.subscription.request(100)
}
```

`Subscription`有一个名为`request()`的方法，它接收我们愿意接收的项目数量。

你可以再次运行代码，现在我们的订阅者打印了前 100 个字符串，然后又沉默了。

我们已经讨论了`BackpressureStrategy.DROP`和`BackpressureStrategy.BUFFER`策略。现在让我们专注于`BackpressureStrategy.MISSING`策略。这个名字有点令人困惑；*自定义*可能更好。我们很快就会看到原因：

```kt
val source = Flowable.create<String> ({
    ...
}, BackpressureStrategy.MISSING)
```

然后，我们将回到`onNext()`，它实际上做了一些事情：

```kt
override fun onNext(t: String) {
    t.repeat(500) // Do something

    println(counter.get()) // Print index of this item
    this.subscription.request(1) // Request next

    if (counter.incrementAndGet() % 100_000 == 0) {
        println("Consumed ${counter.get()} events")
    }
}
```

因此，我们又回到了重复字符串。在完成每个字符串后，我们通过`subscription.request(1)`要求我们的`Flowable`提供下一个字符串。

尽管如此，我们很快就会收到`MissingBackpressureException`。

这是因为我们指定了`BackpressureStrategy.MISSING`策略，但没有指定缓冲区的大小。

为了解决这个问题，我们将使用`onBackpressureBuffer()`方法：

```kt
val source = Flowable.create<String> ({
    ...
}, BackpressureStrategy.MISSING).onBackpressureBuffer(10_000)
```

这推迟了问题，但我们仍然因为`MissingBackpressureException`而崩溃。

在这种情况下，我们需要的不是*创建*一个`Flowable`，而是*生成*它：

```kt
val count = AtomicInteger(0)
// This is not entirely correct, but simplifies our code
val startTime = System.currentTimeMillis()
val source = Flowable.generate<String> {
        it.onNext(UUID.randomUUID().toString())

        if (count.incrementAndGet() == 10_000_000) {
            it.onComplete()
            latch.countDown()
        }

        if (count.get() % 100_000 == 0) {
            println("Produced ${count.get()} events in ${System.currentTimeMillis() - startTime}ms")
            startTime = System.currentTimeMillis()
        }
    }
```

注意，与`create()`不同，`generate()`接收一个 lambda，它代表*单个操作*。因此，我们无法在其中使用循环。相反，我们将状态（如果有的话）存储在外部。

输出如下所示：

```kt
Produced 100000 events in 3650ms
Produced 200000 events in 1942ms
Produced 300000 events in 1583ms
Produced 400000 events in 1630ms
...
```

注意现在生产速度有多慢。这是因为我们在向消费者提供下一批之前等待消费者处理事件。

# 保持状态

在闭包中捕获这些值可能看起来有点丑陋。有一个更函数式的替代方案，但它很难掌握。`Generate`可以接收两个函数而不是一个：

```kt
<T, S> Flowable<T> generate(Callable<S> initialState, BiFunction<S, Emitter<T>, S> generator)
```

嗯，这听起来有点复杂。让我们试着理解那里发生了什么。

第一个初始状态是 `() -> State`。在我们的例子中，状态可以表示如下：

```kt
data class State(val count: Int, val startTime: Long)
```

为了简单起见，我们不向我们的函数传递 `CountDownLatch` 的实例。你很快就会明白为什么。

因此，我们的第一个参数是 `() -> State` 函数，它没有参数并返回一个 `State`。现在，第二个参数应该是一个函数，即 `(State, Emitter<T>) -> State`。在我们的例子中，我们发射字符串，所以我们的函数是 `(State, Emitter<String>) -> State`。

由于这不仅仅对我们，也对 Kotlin 编译器来说有点混乱，我们明确指定了这些函数的类型，即 `Callable<State>` 和 `BiFunction<State, Emitter<String>, State>`：

```kt
val source = Flowable.generate<String, State>(
    Callable<State> { State(0, System.currentTimeMillis()) },
    BiFunction<State, Emitter<String>, State> { state, emitter ->
        emitter.onNext(UUID.randomUUID().toString())

        // In other cases you could use destructuring
        val count = state.count + 1
        var startTime = state.startTime
        if (count == 10_000_000) {
            emitter.onComplete()
            latch.countDown()
        }

        if (count % 100_000 == 0) {
            println("Produced ${count} events in ${System.currentTimeMillis() - startTime}ms")
            startTime = System.currentTimeMillis()
        }

        // Return next state
        State(count, startTime)
    }
)
```

如你所见，有时纯函数式代码可能非常复杂。幸运的是，Kotlin 允许我们根据不同情况选择不同的方法。

# FlowableProcessor

就像任何 `Subject` 同时是 `Observer` 和 `Observable` 一样，任何 `FlowableProcessor` 都是一个既是 `Publisher` 也是 `Subscriber` 的 `Flowable`。

为了理解这个陈述，让我们以 `ReplaySubject` 为例，并使用 `ReplayProcessor` 重新编写它：

```kt
val list = (8..23).toList() // Some non trivial numbers
val iterator = list.iterator()
val o = Observable.intervalRange(0, list.size.toLong(), 0, 10, TimeUnit.MILLISECONDS).map {
    iterator.next()
}.toFlowable(BackpressureStrategy.DROP).publish()
```

任何 `Observable` 都可以使用 `toFlowable()` 方法转换为 `Flowable`。与任何 `Flowable` 一样，我们需要指定要使用哪种策略。在我们的例子中，我们使用 `BackpressureStrategy.DROP`。

正如你所见，`Flowable` 支持与 `Observable` 相同的 `publish()` 方法：

```kt
val processor = ReplayProcessor.createWithSize<Int>(2)
```

我们不是创建 `ReplaySubject`，而是创建 `ReplayProcessor`，它也支持大小限制：

```kt
o.subscribe(processor)

o.connect() // Start publishing

Thread.sleep(20)

println("S1 subscribes")
processor.subscribe {
    println("S1 $it")
}
println("S1 subscribed")

Thread.sleep(10)

println("S2 subscribes")
processor.subscribe {
    println("S2 $it")
}
println("S2 subscribed")

Thread.sleep(1000)
```

输出实际上是相同的：

```kt
S1 subscribes
S1 9
S1 10
S1 subscribed
S1 11
S2 subscribes
S2 10
S2 11
S2 subscribed
S1 12
S2 12
```

但在处理大量输入时，我们现在有了背压来保护我们。

# 批处理

有时候，减慢生产者是不可能的。那么，我们是否又回到了原始问题，即丢弃一些事件或耗尽内存？幸运的是，Rx 仍然有一些锦囊妙计。批量处理数据通常更有效率。我们已经在上一章讨论了这种情况。为此，我们可以为我们的 `subscriber` 指定 `buffer()`。

缓冲区有三种类型。第一种是按大小批处理：

```kt
val latch = CountDownLatch(1)
val o = Observable.intervalRange(8L, 15L, 0L, 100L, TimeUnit.MILLISECONDS)

o.buffer(3).subscribe({
    println(it)
}, {}, { latch.countDown()})

latch.await()
```

它输出以下内容：

```kt
[8, 9, 10]
[11, 12, 13]
[14, 15, 16]
[17, 18, 19]
[20, 21, 22]
```

第二种是每时间间隔的批次。想象一下，我们有一个屏幕，上面显示最新的新闻，并且每几秒钟就会更新。但对我们来说，每五秒刷新一次视图就足够了：

```kt
val latch = CountDownLatch(1)
val o = Observable.intervalRange(8L, 15L, 0L, 100L, TimeUnit.MILLISECONDS)

o.buffer(300L, TimeUnit.MILLISECONDS).subscribe ({
    println(it)
}, {}, { latch.countDown() })

latch.await()
```

它输出以下内容：

```kt
[8, 9, 10, 11]
[12, 13, 14]
[15, 16, 17]
[18, 19, 20]
[21, 22]
```

第三种类型允许我们依赖于另一个 `Observable`。我们将批量处理，直到它要求我们刷新数据：

```kt
val latch = CountDownLatch(1)
val o = Observable.intervalRange(8L, 15L, 0L, 100L, TimeUnit.MILLISECONDS)

o.buffer(Observable.interval(200L, TimeUnit.MILLISECONDS)).subscribe ({
    println(it)
}, {}, { latch.countDown() })

latch.await()
```

它输出以下内容：

```kt
[8, 9, 10]
[11, 12]
[13, 14]
[15, 16]
[17, 18]
[19, 20]
[21, 22]
[]
```

# 节流

消费者端的节流类似于生产者端的丢弃。但它不仅可以应用于 `Flowable`，也可以应用于 `Observable`。

你指定时间间隔，然后在该间隔内每次只获取一个元素，要么是第一个，要么是最后一个：

```kt
val o = PublishSubject.intervalRange(8L, 15L, 0L, 100L, TimeUnit.MILLISECONDS).publish()

o.throttleFirst(280L, TimeUnit.MILLISECONDS).subscribe {
    println(it)
}

o.buffer(280L,  TimeUnit.MILLISECONDS).subscribe {
    println(it)
}

o.connect()

Thread.sleep(100 * 15)
```

执行这个示例几次，你就会看到你得到不同的结果。节流对时间非常敏感。

`throttleFirst()` 输出 `[8, 11, 15, 17, 21]`，因为它接收到了以下窗口：

```kt
8
[8, 9, 10]
11
[11, 12, 13]
14
[14, 15, 16]
17
[17, 18, 19]
20
[20, 21]
[22]
```

注意，`[22]`被节流并且从未打印出来。

现在，让我们看看当我们使用`throttleLast()`时会发生什么：

```kt
val o = Observable.intervalRange(8L, 15L, 5L, 100L, TimeUnit.MILLISECONDS)

o.throttleLast(280L, TimeUnit.MILLISECONDS).subscribe {
    println(it)
}

o.buffer(280L,  TimeUnit.MILLISECONDS).subscribe {
    println(it)
}

Thread.sleep(100 * 30)
```

`throttleLast()`输出`[10, 13, 16, 19, 22]`，因为它接收到了以下窗口：

```kt
10
[8, 9, 10]
13
[11, 12, 13]
16
[14, 15, 16]
19
[17, 18, 19]
21
[20, 21]
[22]
```

再次，`[22]`被节流并且从未打印出来。

节流是本章我们将讨论的最后一种弹性工具，但它可能是最有用的之一。

# 摘要

在本章中，我们学习了反应式系统的主要好处。这些系统应该是响应的、弹性的、可伸缩的，并由消息驱动。

我们还讨论了 Java 9 反应式流 API 及其最流行的实现，即 Rx。

现在，你应该更好地理解了冷`Observable`和热`Observable`之间的区别。一个冷`Observable`只有在有人订阅它时才开始工作。另一方面，热`Observable`始终发出事件，即使没有人监听。

我们还讨论了`Flowable`实现的背压概念。它允许生产者和消费者之间有反馈机制。

此外，你应该熟悉使用主题进行多播的概念。它允许我们向多个监听器发送相同的消息。

最后，我们讨论了一些弹性机制，例如缓冲和节流，这些机制允许我们在无法及时处理消息时积累或丢弃消息。

在下一章中，我们将开始讨论线程，如果你有 Java 背景，你应该熟悉这个概念，以及协程，这是 Kotlin 1.1 中引入的轻量级线程。
