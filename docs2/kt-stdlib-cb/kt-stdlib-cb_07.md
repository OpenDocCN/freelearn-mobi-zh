# 第七章：让异步编程再次伟大

在本章中，我们将涵盖以下食谱：

+   使用线程在后台执行任务

+   后台线程同步

+   使用协程进行异步、并发任务执行

+   使用协程执行异步、并发任务执行并处理结果

+   应用协程进行异步数据处理

+   简单的协程取消

+   使用 Retrofit 和协程适配器构建 REST API 客户端

+   使用协程包装第三方回调式 API

# 简介

本章将讨论异步编程问题的各个方面。前两个食谱，“使用线程在后台执行任务”和“后台线程同步”，将解释标准库对使用 JVM 线程运行后台任务的支持。

在后续的食谱中，我们将更深入地探讨强大的 Kotlin 协程框架。这些食谱将解释协程在异步和并发任务执行中的通用用法。它们还将展示如何使用协程解决更具体的日常编程问题，例如并发数据处理、异步 REST 调用处理，以及以整洁的方式与第三方回调式 API 一起工作。阅读完这一章后，您将感到方便地将协程框架应用于编写健壮的异步代码，或通过并发运行昂贵的计算来优化您的代码。

Kotlin 协程框架不仅是一个平台特定并发和异步框架的便捷替代品。其力量在于提供统一的、通用的 API，使我们能够编写可以在 JVM、Android、JavaScript 和本地平台上运行的异步代码。

# 使用线程在后台执行任务

在这个食谱中，我们将探讨如何使用 Kotlin 标准库中专门为方便线程运行而设计的函数，以整洁的方式有效地与 JVM `Thread` 类一起工作。我们将模拟两个长时间运行的任务，并在后台线程中并发执行它们。

# 准备工作

在这个食谱中，我们将使用两个模拟长时间运行操作的功能。这是第一个：

```java
private fun `5 sec long task`() = Thread.sleep(5000)
```

这是第二个：

```java
private fun `2 sec long task`() = Thread.sleep(2000)
```

它们两者都只是分别阻塞当前线程五秒和两秒，以模拟长时间运行的任务。我们还将使用预定义的函数返回当前线程名称，用于调试目的：

```java
private fun getCurrentThreadName(): String = Thread.currentThread().name
```

# 如何做到这一点...

1.  让我们先记录当前线程名称到控制台：

```java
println("Running on ${getCurrentThreadName()}")
```

1.  在一个新的 `Thread` 中启动并调用 `5 sec long task()` 函数：

```java
println("Running on ${getCurrentThreadName()}")

thread {
    println("Starting async operation on ${getCurrentThreadName()}")
 `5 sec long task`()
 println("Ending async operation on ${getCurrentThreadName()}")
}
```

1.  在另一个 `Thread` 中启动另一个 `Thread` 并调用 `2 sec long task()`：

```java
println("Running on ${getCurrentThreadName()}")

thread {
    println("Starting async operation on ${getCurrentThreadName()}")
    `5 sec long task`()
    println("Ending async operation on ${getCurrentThreadName()}")
}

thread {
    println("Starting async operation on ${getCurrentThreadName()}")
 `2 sec long task`()
 println("Ending async operation on ${getCurrentThreadName()}")
}
```

# 它是如何工作的...

以下代码将打印以下文本到控制台：

```java
Running on main
Starting async operation on Thread-0
Starting async operation on Thread-1
Ending async operation on Thread-1
Ending async operation on Thread-0
```

如您所见，我们已经成功启动了两个并发运行的后台线程。我们使用来自 `kotlin.concurrent` 包的 `thread()` 工具函数，该函数负责实例化和启动一个新的线程，该线程以 lambda 表达式的形式运行传递给它的代码块。

# 参见

+   查看其余的菜谱，了解如何使用 Kotlin 协程框架用更强大和灵活的框架替换线程机制。一个好的起点可以是 *使用协程执行异步并发任务* 和 *使用协程执行异步并发任务并处理结果* 菜谱。

# 背景线程同步

在这个菜谱中，我们将探讨如何使用 Kotlin 标准库中专门用于以方便方式运行线程的函数，以干净的方式有效地与 JVM `Thread` 类一起工作。我们将模拟两个长时间运行的任务，并在后台线程中同步执行它们。

# 准备工作

在这个菜谱中，我们将使用以下两个函数来模拟长时间运行的操作。`5 秒长任务()` 函数：

```java
private fun `5 sec long task`() = Thread.sleep(5000)
```

以及 `2 秒长任务()` 函数：

```java
private fun `2 sec long task`() = Thread.sleep(2000)
```

它们各自仅负责阻塞当前线程五秒和两秒，以模拟长时间运行的任务。我们还将使用预定义的函数返回当前线程名称，用于调试目的：

```java
private fun getCurrentThreadName(): String = Thread.currentThread().name
```

# 如何操作...

1.  让我们先记录当前线程名称到控制台：

```java
println("Running on ${getCurrentThreadName()}")
```

1.  在其中启动一个新的 `Thread` 并调用 `5 秒长任务()` 函数：

```java
println("Running on ${getCurrentThreadName()}")

thread {
    println("Starting async operation on ${getCurrentThreadName()}")
 `5 sec long task`()
 println("Ending async operation on ${getCurrentThreadName()}")
}
```

1.  等待线程完成：

```java
println("Running on ${getCurrentThreadName()}")

thread {
    println("Starting async operation on ${getCurrentThreadName()}")
    `5 sec long task`()
    println("Ending async operation on ${getCurrentThreadName()}")
}.join()
```

1.  在其中启动另一个 `Thread` 并调用 `2 秒长任务()` 函数：

```java
println("Running on ${getCurrentThreadName()}")

thread {
    println("Starting async operation on ${getCurrentThreadName()}")
    `5 sec long task`()
    println("Ending async operation on ${getCurrentThreadName()}")
}.join()

thread {
    println("Starting async operation on ${getCurrentThreadName()}")
 `2 sec long task`()
 println("Ending async operation on ${getCurrentThreadName()}")
}
```

1.  等待线程完成：

```java
println("Running on ${getCurrentThreadName()}")

thread {
    println("Starting async operation on ${getCurrentThreadName()}")
    `5 sec long task`()
    println("Ending async operation on ${getCurrentThreadName()}")
}.join()

thread {
    println("Starting async operation on ${getCurrentThreadName()}")
    `2 sec long task`()
    println("Ending async operation on ${getCurrentThreadName()}")
}.join()
```

1.  在结束时测试主线程是否空闲：

```java
println("Running on ${getCurrentThreadName()}")

thread {
    println("Starting async operation on ${getCurrentThreadName()}")
    `5 sec long task`()
    println("Ending async operation on ${getCurrentThreadName()}")
}.join()

thread {
    println("Starting async operation on ${getCurrentThreadName()}")
    `2 sec long task`()
    println("Ending async operation on ${getCurrentThreadName()}")
}.join()

println("${getCurrentThreadName()} thread is free now")
```

# 它是如何工作的...

之前的代码将打印以下文本到控制台：

```java
Running on main
Starting async operation on Thread-0
Ending async operation on Thread-0
Starting async operation on Thread-1
```

```java
Ending async operation on Thread-1
main thread is free now
```

我们已经成功启动了两个同步的后台线程。为了顺序运行这两个后台线程，我们使用了 `Thread.join()` 函数，它只是阻塞主线程，直到后台线程完成。为了实例化和启动一个新的后台线程，我们使用了来自 `kotlin.concurrent` 包的 `thread()` 工具函数。我们向它传递一个以 lambda 表达式形式传递给它的代码块，以在线程中运行。

# 参见

+   查看下一组菜谱，了解如何使用 Kotlin 协程框架用更强大和灵活的框架替换线程机制。一个好的起点可以是 *使用协程执行异步并发任务* 和 *使用协程执行异步并发任务并处理结果* 菜谱。

# 使用协程进行异步、并发任务执行

在这个菜谱中，我们将探讨如何使用协程框架来安排异步、并发的任务执行。我们将学习如何同步一系列短的后台任务，以及如何同时运行昂贵、长时间运行的任务。我们将通过模拟寿司卷准备过程来发现如何一起安排阻塞和非阻塞任务。

# 准备工作

开始使用 Kotlin 协程的第一步是将核心框架依赖项添加到项目中：

```java
implementation 'org.jetbrains.kotlinx:kotlinx-coroutines-core:0.23.3' 
```

以下代码在 Gradle 构建脚本中声明了`kotlinx-coroutines-core`依赖项，该依赖项用于示例项目（[`github.com/PacktPublishing/Kotlin-Standard-Library-Cookbook`](https://github.com/PacktPublishing/Kotlin-Standard-Library-Cookbook)）。

在当前菜谱中，我们将假设我们的寿司烹饪模拟需要执行以下四个步骤：

1.  煮米饭

1.  准备鱼

1.  切蔬菜

1.  卷寿司

这些步骤将由以下函数来模拟：

```java
private fun `cook rice`() {
    println("Starting to cook rice on ${getCurrentThreadName()}")
    Thread.sleep(10000)
    println("Rice cooked")
}

private fun `prepare fish`() {
    println("Starting to prepare fish on ${getCurrentThreadName()}")
    Thread.sleep(2000)
    println("Fish prepared")
}

private fun `cut vegetable`() {
    println("Starting to cut vegetables on ${getCurrentThreadName()}")
    Thread.sleep(2000)
    println("Vegetables ready")
}

private fun `roll the sushi`() {
    println("Starting to roll the sushi on ${getCurrentThreadName()}")
    Thread.sleep(2000)
    println("Sushi rolled")
}
```

我们还将使用以下函数来记录当前线程名称到控制台：

```java
private fun `print current thread name`() {
    println("Running on ${getCurrentThreadName()}")
    println()
}

private fun getCurrentThreadName(): String = Thread.currentThread().name
```

为了练习的目的，我们将假设寿司卷准备过程必须满足以下要求：

+   最长的*煮饭*步骤必须在后台以非阻塞方式执行

+   *鱼准备*和*切菜*步骤必须在煮饭时依次执行

+   *卷寿司*步骤只能在完成前三个步骤后进行

# 如何操作...

1.  让我们先记录当前线程名称到控制台：

```java
`print current thread name`()
```

1.  在后台线程池上启动一个新的协程：

```java
`print current thread name`()
var sushiCookingJob: Job
sushiCookingJob = launch(newSingleThreadContext("SushiThread")) {
    `print current thread name`()
}
```

1.  在嵌套协程中异步执行`cook rice()`函数：

```java
`print current thread name`()
var sushiCookingJob: Job
sushiCookingJob = launch(newSingleThreadContext("SushiThread")) {
    `print current thread name`()
    val riceCookingJob = launch {
        `cook rice`()
 }
}
```

1.  在后台运行`cook rice()`函数的同时，按顺序运行`prepare fish()`和`cut vegetable()`函数：

```java
`print current thread name`()
var sushiCookingJob: Job
sushiCookingJob = launch(newSingleThreadContext("SushiThread")) {
    `print current thread name`()
    val riceCookingJob = launch {
        `cook rice`()
    }
    println("Current thread is not blocked while rice is being
     cooked")
    `prepare fish`()
 `cut vegetable`()
}
```

1.  等待米饭烹饪的协程完成：

```java
`print current thread name`()
var sushiCookingJob: Job
sushiCookingJob = launch(newSingleThreadContext("SushiThread")) {
    `print current thread name`()
    val riceCookingJob = launch {
        `cook rice`()
    }
    println("Current thread is not blocked while rice is being
     cooked")
    `prepare fish`()
    `cut vegetable`()
 riceCookingJob.join()
}
```

1.  调用最终的`roll the sushi()`函数并等待主协程完成：

```java
`print current thread name`()
var sushiCookingJob: Job
sushiCookingJob = launch(newSingleThreadContext("SushiThread")) {
    `print current thread name`()
    val riceCookingJob = launch {
        `cook rice`()
    }
    println("Current thread is not blocked while rice is being
     cooked")
    `prepare fish`()
    `cut vegetable`()
    riceCookingJob.join()
 `roll the sushi`()
}
runBlocking {
   sushiCookingJob.join()
}
```

1.  测量函数执行的总时间并将其记录到控制台：

```java
`print current thread name`()
var sushiCookingJob: Job
val time = measureTimeMillis {
    sushiCookingJob = launch(newSingleThreadContext("SushiThread")) {
        `print current thread name`()
        val riceCookingJob = launch {
            `cook rice`()
        }
        println("Current thread is not blocked while rice is being
         cooked")
        `prepare fish`()
        `cut vegetable`()
        riceCookingJob.join()
        `roll the sushi`()
    }
    runBlocking {
        sushiCookingJob.join()
    }
}
println("Total time: $time ms")
```

# 它是如何工作的...

以下代码将在控制台打印以下文本：

```java
Running on main
Running on SushiThread
Current thread is not blocked while rice is being cooked
Starting to cook rice on ForkJoinPool.commonPool-worker-1
Starting to prepare fish on SushiThread
Fish prepared
Starting to cut vegetables on SushiThread
Vegetables ready
Rice cooked
Starting to roll the sushi on SushiThread
Sushi rolled
Total time: 12089 ms
```

在开始时，我们使用`launch()`函数调用在后台线程上启动一个新的协程。我们还在`var sushiCookingJob: Job`变量下创建了`launch()`函数返回的`Job`实例的句柄。

`launch()`函数在默认的`CoroutineContext`实例上启动一个新的协程实例。然而，我们可以将我们想要的`CoroutineContext`作为附加参数传递给`launch()`函数。在针对 JVM 平台时，默认情况下，`launch()`函数在一个后台线程池上启动协程，这对应于`CommonPool`上下文常量。我们也可以通过传递`newSingleThreadContext()`函数的上下文结果来在单个线程上运行协程。如果你正在使用 UI 框架，如 Android、Swing 或 JavaFx，你还可以在`UI`上下文中运行协程。`UI`上下文与负责用户界面更新的主线程相关。有不同模块提供针对特定框架的`UI`上下文实现。你可以在以下官方指南中了解更多关于特定框架的协程 UI 编程：[`github.com/Kotlin/kotlinx.coroutines/blob/master/ui/coroutines-guide-ui.md`](https://github.com/Kotlin/kotlinx.coroutines/blob/master/ui/coroutines-guide-ui.md)。

在主协程内部，我们启动一个新的协程，并在其中调用`cook rice()`函数。我们将对应于处理`cook rice()`函数的协程的`Job`实例的句柄存储在`val riceCookingJob: Job`变量下。此时，米饭烹饪任务开始在多个线程的线程池上并发运行。

接下来，我们调用两个函数——`prepare fish()`和`cut vegetable()`。正如你在控制台输出中看到的，这些函数是顺序执行的。蔬菜切割任务在鱼准备完成后立即开始。如果我们想并发运行它们，我们需要在每个函数内部启动一个新的协程。

最后，我们通过在`riceCookingJob`变量上调用一个`join()`函数来等待米饭烹饪任务的完成。在这里，`join()`函数挂起主`sushiCookingJob`协程，直到`riceCookingJob`完成。在主协程解除阻塞后，紧接着调用的最后一个函数是`roll the sushi()`。

为了等待主协程完成，我们需要在主线程上启动它之后，在`sushiCookingJob`实例上调用一个`join()`函数。然而，我们无法在协程作用域之外调用`join()`函数。我们需要在用`runBlocking()`函数启动的新*阻塞*协程内部调用它。

协程框架旨在允许我们以非阻塞方式执行任务。尽管我们能够在协程的作用域内编写非阻塞代码，但我们仍需要提供一个桥梁，将启动主协程的应用程序中的原始线程连接起来。我们可以使用`runBlocking()`函数将非阻塞的协程作用域与外部的阻塞世界连接起来。

`runBlocking()` 函数启动一个新的协程并阻塞当前线程，直到其完成。它旨在将常规阻塞代码与以挂起风格编写的库桥接起来。例如，它可以在 `main()` 函数和测试中使用。

协程可以被视为轻量级的线程替代品。在资源消耗方面，协程是轻量级的。例如，我们可以轻松地同时启动一百万个协程，其中每个协程在两秒钟后都将当前线程名称记录到控制台：

```java
runBlocking {
    (0..1000000).map {
        launch {
            delay(1000)
            println("Running on ${Thread.currentThread().name}")
        }
    }.map { it.join() }
}
```

之前的代码在标准计算机上大约需要 10 秒钟才能完成。相比之下，如果我们尝试使用线程运行此代码，我们会得到 `OutOfMemoryError: unable to create new native thread` 异常。

# 参见

+   您可以通过阅读 *使用协程进行异步、并发任务执行以及结果处理* 菜谱来跟进。它展示了如何异步安排返回结果的函数。

# 使用协程进行异步、并发任务执行以及结果处理

在这个菜谱中，我们将探索如何使用协程框架来并发运行异步操作，并学习如何正确处理它们返回的结果。我们将安排两个任务，并使用两个协程在后台运行它们。第一个任务将负责显示进度条动画。第二个任务将模拟长时间运行的计算。最后，我们将打印第二个任务返回的结果到控制台。

# 准备工作

开始使用 Kotlin 协程的第一步是将核心框架依赖项添加到项目中：

```java
implementation 'org.jetbrains.kotlinx:kotlinx-coroutines-core:0.23.3' 
```

之前的代码在 Gradle 构建脚本中声明了 `kotlinx-coroutines-core` 依赖项，该依赖项用于示例项目 ([`github.com/PacktPublishing/Kotlin-Standard-Library-Cookbook`](https://github.com/PacktPublishing/Kotlin-Standard-Library-Cookbook))。

在当前菜谱中，我们将使用以下两个函数：

```java
private suspend fun `calculate the answer to life the universe and everything`(): Int {
    delay(5000)
    return 42
}

private suspend fun `show progress animation`() {
    val progressBarLength = 30
    var currentPosition = 0
    while (true) {
        print("\r")
        val progressbar = (0 until progressBarLength)
                .map { if (it == currentPosition) " " else "░" }
                .joinToString("")
        print(progressbar)

        delay(50)

        if (currentPosition == progressBarLength) {
            currentPosition = 0
        }
        currentPosition++
    }
}
```

第一个函数是模拟一个昂贵的计算，它将线程延迟五秒钟并在最后返回结果。第二个函数负责显示无限进度条动画。我们将同时启动这两个操作并等待第一个操作返回的结果。在得到结果后，我们将将其打印到控制台。

我们还将使用以下函数将当前线程名称记录到控制台：

```java
private fun `print current thread name`() {
    println("Running on ${getCurrentThreadName()}")
    println()
}

private fun getCurrentThreadName(): String = Thread.currentThread().name
```

# 如何做到这一点...

1.  首先记录当前线程名称到控制台：

```java
`print current thread name`()
```

1.  启动一个协程，负责从后台显示进度条动画：

```java
`print current thread name`()

launch {
    println("Starting progressbar animation on ${getCurrentThreadName()}")
 `show progress animation`()
}
```

1.  启动一个协程，负责在后台运行 `calculate the answer to life the universe and everything()` 函数：

```java
`print current thread name`()

launch {
    println("Starting progressbar animation on ${getCurrentThreadName()}")
    `show progress animation`()
}

val future = async {
    println("Starting computations on ${getCurrentThreadName()}")
 `calculate the answer to life the universe and everything`()
}

println("${getCurrentThreadName()} thread is not blocked while tasks are in progress")
```

1.  等待 `future` 协程返回的结果并将其打印到控制台：

```java
`print current thread name`()

launch {
    println("Starting progressbar animation on ${getCurrentThreadName()}")
    `show progress animation`()
}

val future = async {
    println("Starting computations on ${getCurrentThreadName()}")
    `calculate the answer to life the universe and everything`()
}

println("${getCurrentThreadName()} thread is not blocked while tasks are in progress")

runBlocking {
    println("\nThe answer to life the universe and everything: ${future.await()}")
 `print current thread name`()
}
```

# 它是如何工作的...

我们将编写代码以显示进度条动画五秒钟，然后一旦模拟计算完成，将打印`calculate the answer to life the universe and everything()`函数的结果：

```java
Running on main
Starting progressbar animation on ForkJoinPool.commonPool-worker-1
Starting calculation of the answer to life the universe and everything on ForkJoinPool.commonPool-worker-2
main thread is not blocked while background tasks are still in progress
░░░ ░░░░░░░░░░░░░░░░░░░░░░░░░░
The answer to life the universe and everything: 42
Running on main
```

我们使用`async()`函数在后台任务中启动`calculate the answer to life the universe and everything()`函数的执行。它只是启动一个新的协程并返回一个`Deferred<T>`类的实例。泛型`T`类型对应于`async()`返回的对象类型。`Deferred<T>`类型的实例只是指向协程提供的未来结果的指针。它是异步编程结构的表示，称为*future*或*promise*。我们可以通过在`Deferred`对象上调用`await()`函数来评估`Deferred`对象的值。然而，我们无法在协程作用域之外调用`await()`函数。我们需要在用`runBlocking()`函数启动的新*阻塞*协程内部调用它。

协程框架旨在允许我们以非阻塞方式执行任务。尽管我们能够在协程的作用域内编写非阻塞代码，但我们仍需要提供一个连接到启动主协程的应用程序中原始线程的桥梁。我们可以使用`runBlocking()`函数将非阻塞的协程作用域与阻塞的外部世界连接起来。`runBlocking()`函数启动一个新的协程并阻塞当前线程直到其完成。它旨在将常规阻塞代码与以挂起风格编写的库桥接起来。例如，它可以在`main()`函数和测试中使用。

就进度条动画而言，我们正在使用`launch()`函数在后台安排它。`launch()`负责启动一个新的协程，然而，它并不关心交付最终结果。

# 更多...

你可能已经注意到我们的预定义函数在`fun`关键字之前标记了`suspend`修饰符，例如，``**suspend** fun `show progress animation`()``。背后的原因是，我们需要明确声明函数将在协程作用域内运行，以便能够在函数体内部使用协程特定的功能。在我们的例子中，我们使用了`delay()`函数，它只能在协程作用域内调用。它负责暂停协程一段时间而不阻塞当前线程。

# 参见

+   你可以在*Applying coroutines for asynchronous data processing*菜谱中调查`delay()`函数的另一种用法。你还可以在*Easy coroutines cancelation*菜谱中探索挂起函数的不同用例。

+   如果您想了解更多关于使用协程进行并发、异步任务调度的信息，您可以查看 *使用协程进行异步、并发任务执行及结果处理* 菜谱。它解释了如何调度在公共协程中运行的顺序和并发任务。

# 应用协程进行异步数据处理

在这个菜谱中，我们将为 `Iterable` 类型实现一个泛型扩展，它将提供 `Iterable<T>.map()` 函数的替代方案。我们的 `Iterable<T>.mapConcurrent()` 函数实现将通过与协程并发运行来优化数据映射操作。接下来，我们将通过将其应用于模拟对样本 `Iterable` 对象每个元素应用耗时操作来测试我们的并发映射函数实现。

# 如何实现...

1.  为泛型 `Iterable<T>` 类实现一个扩展函数，用于处理其元素的并发映射操作：

```java
suspend fun <T, R> Iterable<T>.mapConcurrent(transform: suspend (T) -> R) =
    this.map {
        async { transform(it) }
    }.map {
        it.await()
    }
```

1.  模拟对样本 `Iterable` 范围元素应用耗时映射操作：

```java
runBlocking {
 (0..10).mapConcurrent {
        delay(1000)
        it * it
    }
}
```

1.  将映射后的元素打印到控制台：

```java
runBlocking {
        (0..10).mapConcurrent {
            delay(1000)
            it * it
        }.map { println(it) }
}
```

1.  测量并发映射操作执行的总时间并将其记录到控制台：

```java
runBlocking {
 val totalTime = measureTimeMillis {        (0..10).mapConcurrent {
            delay(1000)
            it * it
        }.map { println(it) }
    }
 println("Total time: $totalTime ms")
}
```

# 它是如何工作的...

让我们先分析一下，将我们在开头实现的 `mapConcurrent()` 函数应用于整数范围 `(0..10)` 的元素时产生的影响。在传递给 `mapConcurrent` 函数的 lambda 块中，我们正在模拟一个长时间运行的处理操作，通过使用 `delay(1000)` 函数挂起协程一秒钟，并返回原始整数值的平方。

我们的代码将打印以下结果到控制台：

```java
0
1
4
9
16
25
36
49
64
81
100
Total time: 1040 ms
```

我们实现的 `Iterable.mapConcurrent()` 扩展函数接受一个函数参数 `transform: suspend (T) -> R`，它表示将要应用于 `Iterable` 对象原始元素的运算。在底层，为了实现并发数据转换，我们为每个原始元素使用 `async()` 函数启动一个新的协程，并将 `transform` 函数应用于它们。此时，原始的 `Iterable<T>` 实例已经被转换成了 `Iterable<Deferred<T>>` 类型。接下来，通过在它们上调用 `await()` 函数，将 `async()` 调用返回的连续 `Deferred` 类型实例同步，并转换为通用的 `R` 类型。最后，我们得到了一个返回所需 `R` 类型的 `Iterable`。

如您所见，在我们的示例输出中，使用 `Iterable.mapConcurrent()` 函数对 10 个整数进行转换在大约一秒钟内完成，在标准计算机上。您可以尝试使用标准的 `Iterable.map()` 运行相同的转换，它将需要大约 10 秒钟。

为了在传递给 `mapConcurrent()` 函数的 `transform` lambda 块内部模拟延迟，我们使用带有指定时间值的 `delay()` 函数。`delay()` 函数会暂停协程一段时间，但不会阻塞线程。`transform` 块将为后台线程池中的每个元素执行。每当一个协程被暂停时，另一个协程就会在原地开始运行以替代第一个协程。如果我们用阻塞的 `Thread.sleep(1000)` 函数替换非阻塞的 `delay(1000)` 调用，我们的示例将大约在四秒内完成。与默认不并发运行的 `Iterable.map()` 函数相比，这仍然是一个巨大的进步。

协程框架旨在允许我们以非阻塞的方式执行任务。尽管我们能够在协程的作用域内编写非阻塞代码，但我们仍需要为启动主协程的应用程序中的原始线程提供一个桥梁。我们能够使用 `runBlocking()` 函数将非阻塞的协程作用域与外部的阻塞世界连接起来。`runBlocking()` 函数启动一个新的协程并阻塞当前线程，直到其完成。它旨在将常规阻塞代码与以挂起风格编写的库桥接起来。例如，它可以在 `main()` 函数和测试中使用。

# 参见

+   如果你想要了解更多关于扩展函数机制的基础知识，你可以查看第二章中的*扩展类的功能*配方，*表达性函数和可调整接口*。

# 简单的协程取消

在这个配方中，我们将探讨如何实现一个允许我们取消其执行的协程。我们将使用协程在后台控制台创建一个无限进度条动画。接下来，在给定延迟后，我们将取消协程并测试动画的行为。

# 准备工作

开始使用 Kotlin 协程的第一步是将核心框架依赖项添加到项目中：

```java
implementation 'org.jetbrains.kotlinx:kotlinx-coroutines-core:0.23.3' 
```

上述代码在 Gradle 构建脚本中声明了 `kotlinx-coroutines-core` 依赖项，该依赖项用于示例项目 ([`github.com/PacktPublishing/Kotlin-Standard-Library-Cookbook`](https://github.com/PacktPublishing/Kotlin-Standard-Library-Cookbook))。

# 如何操作...

1.  实现一个负责在控制台显示无限进度条动画的挂起函数：

```java
private suspend fun `show progress animation`() {
    val progressBarLength = 30
    var currentPosition = 0
    while (true) {
        print("\r")
        val progressbar = (0 until progressBarLength)
                .map { if (it == currentPosition) " " else "░" }
                .joinToString("")
        print(progressbar)

        delay(50)

        if (currentPosition == progressBarLength) {
            currentPosition = 0
        }
        currentPosition++
    }
}
```

1.  在一个新的协程中启动 `show progress animation()` 函数：

```java
runBlocking {
    val job = launch { `show progress animation`() }
}
```

1.  暂停父线程五秒钟：

```java
runBlocking {
    val job = launch { `show progress animation`() }
    delay(5000)
}
```

1.  取消进度条动画任务：

```java
runBlocking {
    val job = launch { `show progress animation`() }
    delay(5000)
    job.cancel()
    println("Cancelled")
}
```

1.  等待作业完成并将完成事件记录到控制台：

```java
runBlocking {
    val job = launch {`show progress animation`()}
    delay(5000)
    job.cancel()
 job.join()
 println("\nJob cancelled and completed")
}
```

# 工作原理...

最后，我们的代码将显示一个持续五秒的进度条动画，然后停止。我们通过在由`launch()`函数创建的新协程实例内部调用它，来安排`show progress animation()`函数在后台运行。我们将`launch()`函数返回的`Job`实例的句柄存储在`job`变量下。

接下来，我们使用`delay(5000)`调用将外部的`runBlocking()`协程作用域挂起五秒。一旦`delay()`函数恢复协程执行，我们就调用负责显示进度条动画的协程`Job`上的`cancel()`函数。

在底层，在`show progress animation()`函数内部，我们运行一个无限`while`循环，每 50 毫秒更新一次最后一条控制台行，以显示新的进度条动画状态。然而，正如你可以通过运行示例来验证的那样，动画在负责运行它的`Job`被取消后立即停止，尽管在取消后，我们调用了`join()`函数来等待其完成。

你还可以使用一个名为`cancelAndJoin()`的`Job`扩展函数，它将`cancel()`和`join()`调用组合在一起。然而，如果你不想等待实际的协程停止事件，一个简单的`cancel()`调用就足够了。

# 参见

+   如果你想要探索协程框架的基础知识，请查看*使用协程执行异步并发任务*和*使用协程执行异步并发任务并处理结果*配方

# 使用 Retrofit 和协程适配器构建 REST API 客户端

在这个配方中，我们将探索如何使用协程通过 REST API 与远程端点交互。我们将使用 Retrofit 库实现 REST 客户端，允许我们异步地通过 HTTP 与 GitHub API 通信。最后，我们将将其用于实际操作，以获取给定搜索查询的 GitHub 存储库搜索结果。

# 准备工作

开始使用 Kotlin 协程的第一步是添加一个核心框架依赖项：

```java
implementation 'org.jetbrains.kotlinx:kotlinx-coroutines-core:0.23.3' 
```

为了使用 Retrofit 库与协程适配器插件，我们还需要将以下依赖项添加到我们的项目中：

```java
implementation 'com.squareup.retrofit2:retrofit:2.4.0'
implementation 'com.squareup.retrofit2:converter-gson:2.4.0'
implementation 'com.jakewharton.retrofit:retrofit2-kotlin-coroutines-experimental-adapter:1.0.0'
```

之前的代码在 Gradle 构建脚本中声明了所需的依赖项，该脚本用于示例项目([`github.com/PacktPublishing/Kotlin-Standard-Library-Cookbook`](https://github.com/PacktPublishing/Kotlin-Standard-Library-Cookbook))。`retrofit`模块提供了核心 Retrofit 库实现。`converter-gson`添加了一个 Gson 插件，它允许自动将 JSON 响应转换为 Kotlin 模型数据类。`retrofit2-kotlin-coroutines-experimental-adapter`模块提供了一个用于异步 REST 调用的适配器，允许我们使用 Kotlin 协程的`Deferred`类型包装响应。

在这个菜谱中，我们将使用公开可用的 GitHub REST API。我们将与一个端点进行通信，该端点负责返回包含给定搜索查询的 GitHub 仓库的搜索结果。您可以在以下位置找到详细的端点文档：[`developer.github.com/v3/search/#search-repositories`](https://developer.github.com/v3/search/#search-repositories)。

`/search/repositories`端点允许我们使用`GET`方法访问远程资源，并通过键`q`传递所需的搜索短语。例如，匹配搜索短语`"live.parrot"`的仓库的`GET`请求 URL 将如下所示：`https://api.github.com/search/repositories?q=parrot.live`。端点返回的结果使用 JSON 格式进行格式化。您可以通过在浏览器中打开示例 URL 或使用`curl`命令行工具来查看原始响应的外观：`curl https://api.github.com/search/repositories?q=parrot.live`。

# 如何做到这一点...

1.  声明数据类来模拟服务器响应：

```java
data class Response(@SerializedName("items")
                                      val list: Collection<Repository>)
data class Repository(val id: Long?,
                      val name: String?,
                      val description: String?,
                      @SerializedName("full_name") val fullName:
                       String?,
                      @SerializedName("html_url") val url: String?,
                      @SerializedName("stargazers_count") val stars:
                       Long?)
```

1.  声明一个模拟 GitHub 端点使用的接口：

```java
interface GithubApi {
    @GET("/search/repositories")
    fun searchRepositories(@Query("q") searchQuery: String):
    Deferred<Response>

}
```

1.  使用`Retrofit`类实例化`GithubApi`接口：

```java
val api: GithubApi = Retrofit.Builder()
        .baseUrl("https://api.github.com/")
        .addConverterFactory(GsonConverterFactory.create())
        .addCallAdapterFactory(CoroutineCallAdapterFactory())
        .build()
        .create(GithubApi::class.java)
```

1.  使用`GithubApi`实例调用端点，并传递搜索短语`"kotlin"`：

```java
val api: GithubApi = Retrofit.Builder()
        .baseUrl("https://api.github.com/")
        .addConverterFactory(GsonConverterFactory.create())
        .addCallAdapterFactory(CoroutineCallAdapterFactory())
        .build()
        .create(GithubApi::class.java)

api.searchRepositories("Kotlin")
```

1.  等待响应并获取到`Repository`类对象的列表引用：

```java
val api: GithubApi = Retrofit.Builder()
        .baseUrl("https://api.github.com/")
        .addConverterFactory(GsonConverterFactory.create())
        .addCallAdapterFactory(CoroutineCallAdapterFactory())
        .build()
        .create(GithubApi::class.java)

val downloadedRepos = api.searchRepositories("Kotlin").await().list
```

1.  按照仓库的星级数量降序排序仓库列表，并将它们打印到控制台：

```java
val api: GithubApi = Retrofit.Builder()
        .baseUrl("https://api.github.com/")
        .addConverterFactory(GsonConverterFactory.create())
        .addCallAdapterFactory(CoroutineCallAdapterFactory())
        .build()
        .create(GithubApi::class.java)

val downloadedRepos = api.searchRepositories("Kotlin").await().list
downloadedRepos
 .sortedByDescending { it.stars }
 .forEach {
 it.apply {
 println("$fullName ![](img/a541e4e1-5cd5-430c-8778-8f64c957e4a2.png)$stars\n$description\n$url\n")
 }
 }
```

# 它是如何工作的...

因此，我们的代码将向服务器发送请求，获取并处理响应，并将以下结果打印到控制台：

```java
JetBrains/kotlin ![](img/f20aeb9f-ae96-4ab4-81ae-59a08f467e67.png)23051
The Kotlin Programming Language
https://github.com/JetBrains/kotlin

perwendel/spark ![](img/f20aeb9f-ae96-4ab4-81ae-59a08f467e67.png)7531
A simple expressive web framework for java. News: Spark now has a kotlin DSL https://github.com/perwendel/spark-kotlin
https://github.com/perwendel/spark

KotlinBy/awesome-kotlin ![](img/f20aeb9f-ae96-4ab4-81ae-59a08f467e67.png)5098
A curated list of awesome Kotlin related stuff Inspired by awesome-java. 
https://github.com/KotlinBy/awesome-kotlin

ReactiveX/RxKotlin ![](img/f20aeb9f-ae96-4ab4-81ae-59a08f467e67.png)4413
RxJava bindings for Kotlin
https://github.com/ReactiveX/RxKotlin

JetBrains/kotlin-native ![](img/f20aeb9f-ae96-4ab4-81ae-59a08f467e67.png)4334
Kotlin/Native infrastructure
https://github.com/JetBrains/kotlin-native
 ...
```

我们首先实现了表示服务器 JSON 响应中返回的数据的模型类。你可能已经注意到，一些属性被标记了`@SerializedName()`注解。这个注解的目的是告诉 Gson 库，指定的属性应该从与`@SerializedName()`传递的值匹配的 JSON 字段反序列化。接下来，我们声明了一个接口，`GithubApi`，它表示我们想要用来与端点通信的方法。我们声明了一个名为`searchRepositories`的单个方法，它接受一个`String`参数，该参数对应于存储库搜索端点所需的搜索查询值。我们还用`@GET`注解标记了`searchRepositories`方法，它指定了要使用的 REST 方法类型和端点的路径。`searchRepositories`方法应该返回一个`Deferred<Response>`类型的实例，表示异步调用的未来结果。`GithubApi`接口的实现是由 Retrofit 库内部生成的。为了获取`GithubApi`实例，我们需要实例化`Retrofit`类型，并使用端点的 URL 地址和负责 JSON 反序列化和对服务器执行异步调用的机制进行配置。最后，我们调用`Retrofit.create(GithubApi::class.java)`来获取`GithubApi`实例。就是这样！

为了执行对服务器的实际调用，我们需要调用`GithubApi.searchRepositories()`函数：

```java
api.searchRepositories("Kotlin")
```

接下来，为了从响应中获取`Repository`对象列表，我们需要等待对服务器的异步调用完成以及响应解析：

```java
val downloadedRepos = api.searchRepositories("Kotlin").await().list

```

最后，我们对从响应中获取的存储库列表进行后处理。我们按星级数量降序排序，并使用以下代码将其打印到控制台：

```java
val downloadedRepos = api.searchRepositories("Kotlin").await().list
downloadedRepos
 .sortedByDescending { it.stars }
        .forEach {
            it.apply {
                println("$fullName ![](img/f20aeb9f-ae96-4ab4-81ae-59a08f467e67.png)$stars\n$description\n$url\n")
 }
        } 
```

# 参见

+   如果你想要探索协程框架的基础知识，可以查看*使用协程执行异步并发任务执行*和*使用协程执行异步并发任务执行并处理结果*配方。你可以通过探索其主页[`square.github.io/retrofit/`](http://square.github.io/retrofit/)来了解更多关于 Retrofit 库的信息，其中包含有用的示例。

# 使用协程包装第三方回调式 API

通常，第三方库提供回调式异步 API。然而，回调函数被认为是一种反模式，尤其是在我们处理多个嵌套回调时。在这个配方中，我们将学习如何处理提供回调式方法的库，通过轻松地将它们转换为可以使用协程运行的挂起函数。

# 准备中

开始使用 Kotlin 协程的第一步是将核心框架依赖项添加到项目中：

```java
implementation 'org.jetbrains.kotlinx:kotlinx-coroutines-core:0.23.3' 
```

上述代码在 Gradle 构建脚本中声明了`kotlinx-coroutines-core`依赖，该依赖用于示例项目([`github.com/PacktPublishing/Kotlin-Standard-Library-Cookbook`](https://github.com/PacktPublishing/Kotlin-Standard-Library-Cookbook))。

就食谱任务而言，让我们假设我们有一个名为`Result`的类，其定义如下：

```java
data class Result(val displayName: String)
```

这里是`getResultsAsync()`函数，它模拟第三方回调风格的 API：

```java
fun getResultsAsync(callback: (List<Result>) -> Unit) =
    thread {
        val results = mutableListOf<Result>()

        // Simulate some extensive bacground task
        Thread.sleep(1000)

        results.add(Result("a"))
        results.add(Result("b"))
        results.add(Result("c"))

        callback(results)
    }
```

`getResultsAsync()`函数只是启动一个后台线程，延迟一秒钟，然后调用传递给它作为参数的回调函数，将`Result`类对象的列表传递给它。

# 如何操作...

1.  使用挂起函数包装`getResultsAsync()`函数，直接返回结果：

```java
suspend fun getResults(): List<Result> =
    suspendCoroutine { continuation: Continuation<List<Result>> ->
        getResultsAsync { continuation.resume(it) }
    }

```

1.  在协程中启动一个协程并调用其中的`getResults()`挂起函数：

```java
val asyncResults = async {
    getResults()
}
```

1.  等待结果并打印到控制台：

```java
val asyncResults = async {
    getResults()
}

println("getResults() is running in bacground. Main thread is not blocked.")
asyncResults.await().map { println(it.displayName) }
println("getResults() completed")
```

# 它是如何工作的...

最后，我们的代码将打印以下输出到控制台：

```java
getResults() is running in bacground. Main thread is not blocked.
a
b
c
getResults() completed
Total time elapsed: 1029 ms
```

我们成功地将回调风格的`getResultsAsync(callback: (List<Result>) -> Unit)`函数转换成了直接返回结果的干净形式的挂起函数–`suspend fun getResults(): List<Result>`。为了去掉原始的`callback`参数，我们使用了标准库提供的`suspendCoroutine()`函数。`suspendCoroutine()`函数接受`block: (Continuation<T>) -> Unit`函数类型作为参数。`Continuation`接口被设计用来允许我们恢复由挂起函数暂停的协程。

当在协程内部调用`suspendCoroutine`函数时，它会捕获其执行状态到一个`Continuation`实例中，并将这个延续传递给指定的块作为参数。为了恢复协程的执行，块可以调用`continuation.resume()`或`continuation.resumeWithException()`。

我们在传递给`suspendCoroutine()`函数的 lambda 中调用原始的`getResultsAcync()`函数，并在传递给`getResultsAsync()`函数作为参数的`callback` lambda 阻塞中调用`continuation.resume(it)`函数：

```java
suspend fun getResults(): List<Result> =
    suspendCoroutine { continuation: Continuation<List<Result>> ->
        getResultsAsync { continuation.resume(it) }
    }
```

作为结果，调用`getResults()`的协程将挂起，直到`getResultsAsync()`函数内部执行`callback` lambda。

# 参见

+   如果你想探索协程框架的基础，请查看*使用协程执行异步、并发任务*和*使用协程执行异步、并发任务并处理结果*的食谱
