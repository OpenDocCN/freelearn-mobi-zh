# 第十四章. Swift 中的并发与并行

当我开始学习 Objective-C 时，我已经对并发和多任务处理有了很好的理解，这得益于我在 C 和 Java 等其他语言中的背景知识。这个背景使我能够很容易地使用 Objective-C 中的线程创建多线程应用程序。然后，当苹果在 OS X 10.6 和 iOS 4 中发布 **Grand Central Dispatch** (**GCD**) 时，他们为我改变了一切。起初，我感到否认；GCD 根本不可能比我更好地管理我的应用程序的线程。然后我进入了愤怒阶段，GCD 难以使用和理解。接下来是讨价还价阶段，也许我可以使用 GCD 与我的线程代码一起使用，这样我仍然可以控制线程的工作方式。然后是抑郁阶段，也许 GCD 确实比我更好地处理线程。最后，我进入了哇哦阶段；这个 GCD 真的很容易使用，并且工作得非常出色。在使用了 Grand Central Dispatch 和 Operation Queues 与 Objective-C 一起之后，我看不到使用 Swift 中的手动线程的理由。

在本章中，我们将学习以下主题：

+   并发与并行的基础知识

+   如何使用 GCD 创建和管理并发调度队列

+   如何使用 GCD 创建和管理串行调度队列

+   如何使用各种 GCD 函数将任务添加到调度队列中

+   如何使用 `NSOperation` 和 `NSOperationQueues` 为我们的应用程序添加并发

# 并发与并行

并发是多个任务在同一时间段内启动、运行和完成的理念。这并不一定意味着任务是在同时执行的。为了使任务能够同时执行，我们的应用程序需要在多核或多处理器系统上运行。并发使我们能够与多个任务共享处理器或核心；然而，单个核心一次只能执行一个任务。

并行是两个或更多任务同时运行的概念。由于我们的处理器每个核心一次只能执行一个任务，因此同时执行的任务数量限制在我们处理器中的核心数量。因此，如果我们有一个四核处理器，那么我们只能同时运行四个任务。今天的处理器可以非常快速地执行任务，这可能会让人误以为更大的任务是在同时执行。然而，在系统中，较大的任务实际上是在核心上轮流执行子任务。

为了理解并发与并行的区别，让我们看看一个杂技演员如何抛接球。如果你观察一个杂技演员，他们似乎在任何给定时间都在同时接住和抛出多个球；然而，仔细观察会发现，他们实际上每次只接住和抛出一个球。其他球在空中等待被接住和抛出。如果我们想要能够同时接住和抛出多个球，我们需要添加多个杂技演员。

这个例子非常好，因为我们可以把杂技演员看作是处理器的核心。拥有单核处理器的系统（一个杂技演员），无论看起来如何，一次只能执行一个任务（接住并抛出一个球）。如果我们想同时执行多个任务，我们需要使用多核处理器（多个杂技演员）。

在那些处理器都是单核的古老日子里，要有一个能够同时执行任务的系统，唯一的办法是在系统中拥有多个处理器。这也需要专门的软件来利用多个处理器。在当今世界，几乎每个设备都有一个拥有多个核心的处理器，iOS 和 OS X 操作系统都是设计用来利用多个核心来同时运行任务的。

传统上，应用程序添加并发的方式是创建多个线程；然而，这种模型在任意数量的核心上扩展性不好。使用线程的最大问题是我们的应用程序运行在各种各样的系统（和处理器的）上，为了优化我们的代码，我们需要知道在给定时间可以高效使用多少核心/处理器，这有时在开发时是未知的。

为了解决这个问题，许多操作系统，包括 iOS 和 OS X，开始依赖异步函数。这些函数通常用于启动可能需要很长时间才能完成的任务，例如发起 HTTP 请求或写入数据到磁盘。异步函数通常在长时间运行的任务开始后立即返回，在任务完成之前。通常，这个任务在后台运行，并在任务完成时使用回调函数（例如 Swift 中的闭包）。

这些异步函数对于操作系统提供的任务非常有效，但如果我们需要创建自己的异步函数而不想自己管理线程呢？为此，Apple 提供了一些技术。在本章中，我们将介绍其中两种技术。这些是 GCD 和操作队列。

GCD 是一个基于 C 的低级 API，允许将特定任务排队执行，并在任何可用的处理器核心上调度执行。操作队列与 GCD 类似；然而，它们是 Cocoa 对象，并且内部使用 GCD 实现。

让我们从查看最大公约数（GCD）开始。

## 大中枢调度（Grand Central Dispatch）

大中枢调度提供了所谓的调度队列来管理提交的任务。队列管理这些提交的任务，并以**先进先出**（**FIFO**）的顺序执行它们。这确保了任务是以它们提交的顺序开始的。

任务只是我们应用程序需要执行的一些工作。例如，我们可以创建执行简单计算、读取/写入磁盘数据、发起 HTTP 请求或我们应用程序需要做的任何其他任务的作业。我们通过将代码放在函数或闭包内部并将它们添加到调度队列中来定义这些任务。

GCD 提供了三种类型的队列：

+   **串行队列**：串行队列中的任务（也称为**私有队列**）按提交顺序逐个执行。只有在先前的任务完成后，才会启动每个任务。串行队列通常用于同步访问特定资源，因为我们有保证，串行队列中的两个任务永远不会同时运行。因此，如果访问特定资源的唯一方式是通过串行队列中的任务，那么两个任务将不会同时尝试访问该资源或出现顺序混乱。

+   **并发队列**：并发队列中的任务（也称为**全局调度队列**）是并发执行的；然而，任务的启动顺序仍然是它们被添加到队列中的顺序。在任何给定时刻可以执行的任务的确切数量是可变的，并且取决于系统的当前条件和资源。何时启动任务的决定权在 GCD，而不是我们可以在应用程序内部控制的事情。

+   **主调度队列**：主调度队列是一个全局可用的串行队列，它在应用程序的主线程上执行任务。由于放入主调度队列的任务在主线程上运行，因此它通常在后台处理完成并且用户界面需要更新时从后台队列中调用。

调度队列相对于传统线程提供了许多优势。首要的优势是，使用调度队列时，系统处理线程的创建和管理，而不是应用程序本身。系统可以根据系统的总体可用资源和当前系统条件动态地调整线程的数量。这意味着调度队列可以比我们更有效地管理线程。

调度队列的另一个优点是我们能够控制任务启动的顺序。使用串行队列，我们不仅控制了任务启动的顺序，还确保在先前的任务完成之前不会启动下一个任务。使用传统的线程，这可能会非常繁琐且难以实现，但正如我们在本章后面将看到的，使用调度队列则相当容易。

## 创建和管理调度队列

让我们看看如何创建和使用调度队列。以下三个函数用于创建或检索队列。这些函数如下：

+   `dispatch_queue_create`: 这将创建一个并发或串行类型的调度队列

+   `dispatch_get_global_queue`: 这返回一个具有指定服务质量系统定义的全局并发队列

+   `dispatch_get_main_queue`: 这返回与应用程序主线程关联的串行调度队列

我们还将查看几个将任务提交到队列以执行的功能。这些函数如下：

+   `dispatch_async`: 这提交一个任务以异步执行并立即返回。

+   `dispatch_sync`: 这提交一个任务以同步执行，并在返回之前等待其完成。

+   `dispatch_after`: 这提交一个任务以在指定时间执行。

+   `dispatch_once`: 这提交一个任务，在应用程序运行期间只执行一次。如果应用程序重新启动，它将再次执行该任务。

在我们查看如何使用调度队列之前，我们需要创建一个类来帮助我们演示各种类型队列的工作方式。此类将包含两个基本函数。第一个函数将简单地执行一些基本计算，然后返回一个值。以下是此函数的代码，该函数名为 `doCalc()`：

```swift
func doCalc() {
  var x=100
  var y = x*x
  _ = y/x
}
```

另一个名为 `performCalculation()` 的函数接受两个参数。一个是名为 `iterations` 的整数，另一个是名为 `tag` 的字符串。`performCalculation()` 函数会重复调用 `doCalc()` 函数，直到调用的次数与迭代参数定义的次数相同。我们还使用 `CFAbsoluteTimeGetCurrent()` 函数来计算执行所有迭代所需的时间，然后使用 `tag` 字符串将经过的时间打印到控制台。这将让我们知道函数何时完成以及完成它所需的时间。此函数的代码看起来类似于以下内容：

```swift
func performCalculation(iterations: Int, tag: String) {
  let start = CFAbsoluteTimeGetCurrent()
  for var i=0; i<iterations; i++ {
    self.doCalc()
  }
  let end = CFAbsoluteTimeGetCurrent()
  print("time for \(tag):  \(end-start)")
}
```

这些函数将一起使用以保持我们的队列忙碌，这样我们就可以看到它们是如何工作的。让我们首先通过使用 `dispatch_queue_create()` 函数来创建并发和串行队列来查看 GCD 函数。

### 使用 dispatch_queue_create() 函数创建队列

`dispatch_queue_create()` 函数用于创建并发和串行队列。`dispatch_queue_create()` 函数的语法看起来类似于以下内容：

```swift
func dispatch_queue_t dispatch_queue_create(label: UnsafePointer<Int8>, attr: dispatch_queue_attr_t!) -> dispatch_queue_t!
```

它需要以下参数：

+   `label`: 这是一个附加到队列上的字符串标签，用于在调试工具（如 Instruments 和崩溃报告）中唯一标识它。建议我们使用反向 DNS 命名约定。此参数是可选的，可以是 nil。

+   `attr`: 这指定了要创建的队列类型。这可以是 `DISPATCH_QUEUE_SERIAL, DISPATCH_QUEUE_CONCURRENT` 或 nil。如果此参数为 nil，则创建一个串行队列。

此函数的返回值是新创建的调度队列。让我们通过创建一个并发队列并查看其工作方式来了解如何使用 `dispatch_queue_create()` 函数。

### 注意

一些编程语言使用反向 DNS 命名约定来命名某些组件。这个约定基于一个反转的已注册域名。例如，如果我们为名为`mycompany.com`的公司工作，该公司有一个名为`widget`的产品，那么反向 DNS 名称将是`com.mycompany.widget`。

#### 使用`dispatch_queue_create()`函数创建并发调度队列

以下行创建了一个带有标签`cqueue.hoffman.jon`的并发调度队列：

```swift
let queue = dispatch_queue_create("cqueue.hoffman.jon", DISPATCH_QUEUE_CONCURRENT)
```

正如我们在本节开始时看到的，我们可以使用几个函数将任务提交到调度队列。当我们与队列一起工作时，我们通常希望使用`dispatch_async()`函数来提交任务，因为我们通常不希望等待响应。`dispatch_async()`函数具有以下签名：

```swift
func dispatch_async(queue: dispatch_queue_t!, block: dispatch_queue_block!)
```

以下示例显示了如何使用我们刚刚创建的并发队列的`dispatch_async()`函数：

```swift
let c = { performCalculation(1000, tag: "async0") }
dispatch_async(queue, c)
```

在前面的代码中，我们创建了一个闭包，它代表我们的任务，简单地调用`DoCalculation`实例的`performCalculation()`函数，请求它运行`doCalc()`函数的 1000 次迭代。最后，我们使用`dispatch_async()`函数将任务提交到并发调度队列。此代码将在并发调度队列中执行任务，该队列与主线程分开。

尽管前面的例子工作得很好，但实际上我们可以稍微缩短代码。下一个例子显示，我们实际上不需要像前面例子中那样创建一个单独的闭包；我们也可以像这样提交任务来执行：

```swift
dispatch_async(queue) {
  calculation.performCalculation(10000000, tag: "async1")
}
```

这种简写版本是我们通常提交到队列的小代码块的方式。如果我们有更大的任务，或者需要多次提交的任务，我们通常希望创建一个闭包，并将闭包提交到队列，就像我们最初展示的那样。

让我们通过向队列中添加几个项目并查看它们的返回顺序和时间来查看并发队列实际上是如何工作的。以下代码将向队列添加三个任务。每个任务都会用不同的迭代次数调用`performCalculation()`函数。记住，`performCalculation()`函数将连续执行计算例程，直到执行次数等于传入的迭代次数。因此，我们传递给`performCalculation()`函数的迭代次数越大，它应该执行的时间就越长。让我们看一下以下代码：

```swift
dispatch_async(queue) {
  calculation.performCalculation(10000000, tag: "async1")
}

dispatch_async(queue) {
  calculation.performCalculation(1000, tag: "async2")
}

dispatch_async(queue) {
  calculation.performCalculation(100000, tag: "async3")
}
```

注意到每个函数都是用不同的值在`tag`参数中调用的。由于`performCalculation()`函数会打印出带有经过时间的`tag`变量，我们可以看到任务完成的顺序以及执行所需的时间。如果我们执行前面的代码，我们应该看到以下结果：

```swift
time for async2:  0.000200986862182617
time for async3:  0.00800204277038574
time for async1:  0.461670994758606
```

### 注意

经过的时间会因运行而异，也会因系统而异。

由于队列以 FIFO（先进先出）顺序工作，标签为`async1`的任务首先执行。然而，正如我们从结果中看到的，它是最后一个完成的任务。由于这是一个并发队列，如果可能（如果系统有可用资源），代码块将并发执行。这就是为什么标签为`async2`和`async3`的任务在标签为`async1`的任务之前完成，尽管`async1`任务的执行开始在其他两个任务之前。

现在，让我们看看串行队列是如何执行任务的。

#### 使用`dispatch_queue_create()`函数创建串行调度队列

串行队列的功能与并发队列略有不同。串行队列一次只会执行一个任务，并在开始下一个任务之前等待当前任务完成。这个队列，就像并发调度队列一样，遵循先入先出的顺序。以下代码行将创建一个标签为`squeue.hoffman.jon`的串行队列：

```swift
let queue2 = dispatch_queue_create("squeue.hoffman.jon", DISPATCH_QUEUE_SERIAL)
```

注意，我们使用`DISPATCH_QUEUE_SERIAL`属性创建串行队列。如果你还记得，当我们创建并发队列时，我们使用的是`DISPATCH_QUEUE_CONCURRENT`属性。我们也可以将此属性设置为`nil`，这将默认创建一个串行队列。然而，建议始终将属性设置为`DISPATCH_QUEUE_SERIAL`或`DISPATCH_QUEUE_CONCURRENT`，以便更容易识别我们正在创建哪种类型的队列。

正如我们通过并发调度队列所看到的，我们通常想使用`dispatch_async()`函数来提交任务，因为当我们向队列提交任务时，我们通常不希望等待响应。然而，如果我们确实想等待响应，我们会使用`dispatch_synch()`函数。

```swift
var calculation = DoCalculations()
let c = { calculation.performCalculation(1000, tag: "sync0") }
dispatch_async(queue2, c)
```

就像并发队列一样，我们不需要创建闭包来向队列提交任务。我们也可以这样提交任务：

```swift
dispatch_async(queue2) {
    calculation.performCalculation(100000, tag: "sync1")
}
```

让我们通过向队列添加几个项目并查看它们的完成顺序和时间来了解串行队列的工作原理。以下代码将添加三个任务，这些任务将使用不同的迭代次数调用`performCalculation()`函数：

```swift
dispatch_async(queue2) {
    calculation.performCalculation(100000, tag: "sync1")
}

dispatch_async(queue2) {
    calculation.performCalculation(1000, tag: "sync2")
}

dispatch_async(queue2) {
    calculation.performCalculation(100000, tag: "sync3")
}
```

就像并发队列示例一样，我们使用不同的迭代次数和不同的`tag`参数值调用`performCalculation()`函数。由于`performCalculation()`函数会打印出带有经过时间的`tag`字符串，我们可以看到任务的完成顺序和执行时间。如果我们执行此代码，我们应该看到以下结果：

```swift
time for sync1:  0.00648999214172363
time for sync2:  0.00009602308273315
time for sync3:  0.00515800714492798
```

### 注意

经过时间会因运行而异，也会因系统而异。

与并发队列不同，我们可以看到，尽管`sync2`和`sync3`任务完成所需时间明显更短，但完成的任务顺序与提交的顺序相同。这表明串行队列一次只执行一个任务，并且队列在开始下一个任务之前会等待每个任务完成。

现在我们已经看到了如何使用`dispatch_queue_create()`函数创建并发和串行队列，让我们看看如何使用`dispatch_get_global_queue()`函数获取四个系统定义的全局并发队列之一。

### 使用`dispatch_get_global_queue()`函数请求并发队列

系统为每个应用程序提供四个不同优先级级别的并发全局调度队列。不同的优先级级别是区分这些队列的因素。四个优先级如下：

+   `DISPATCH_QUEUE_PRIORITY_HIGH`：此队列中的项目以最高优先级运行，并且会在默认和低优先级队列的项目之前进行调度

+   `DISPATCH_QUEUE_PRIORITY_DEFAULT`：此队列中的项目以默认优先级运行，并且会在低优先级队列的项目之前，但在高优先级队列的项目之后进行调度

+   `DISPATCH_QUEUE_PRIORITY_LOW`：此队列中的项目以低优先级运行，并且只有在高优先级和默认队列的项目之后才会进行调度

+   `DISPATCH_QUEUE_PRIORITY_BACKGROUND`：此队列中的项目以后台优先级运行，优先级最低

由于这些是全局队列，我们实际上不需要创建它们；相反，我们请求具有所需优先级的队列的引用。要请求全局队列，我们使用`dispatch_get_global_queue()`函数。此函数具有以下语法：

```swift
func dispatch_get_global_queue(identifier: Int, flags: UInt) -> dispatch_queue_t!
```

在这里，以下参数被定义：

+   `identifier`：这是我们请求的队列的优先级

+   `flags`：此参数保留供将来扩展使用，目前应设置为零

我们使用`dispatch_get_global_queue()`函数请求一个队列，如下面的示例所示：

```swift
let queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0)
```

在此示例中，我们请求具有默认优先级的全局队列。然后我们可以像使用我们使用`dispatch_queue_create()`函数创建的并发队列一样使用此队列。使用`dispatch_get_global_queue()`函数返回的队列与使用`dispatch_create_queue()`函数创建的队列之间的区别在于，使用`dispatch_create_queue()`函数时，我们实际上是在创建一个新的队列。使用`dispatch_get_global_queue()`函数返回的队列是在我们的应用程序首次启动时创建的全局队列；因此，我们是在请求一个队列而不是创建一个新的队列。

当我们使用`dispatch_get_global_queue()`函数时，我们避免了创建队列的开销；因此，除非你有特定的理由来创建队列，否则我建议使用`dispatch_get_global_queue()`函数。

### 使用`dispatch_get_main_queue()`函数请求主队列

`dispatch_get_main_queue()`函数返回我们的应用程序的主队列。当应用程序启动时，为主线程自动创建主队列。这个主队列是一个串行队列；因此，队列中的项目将按它们提交的顺序逐个执行。我们通常想要避免使用这个队列，除非我们需要从后台线程更新用户界面。

`dispatch_get_main_queue()`函数的语法如下：

```swift
func dispatch_get_main_queue() -> dispatch_queue_t!
```

以下代码示例展示了如何请求主队列：

```swift
let mainQueue = dispatch_get_main_queue();
```

我们将像提交任何其他串行队列一样将任务提交到主队列。只需记住，提交到这个队列的任何内容都将运行在主线程上，这是所有用户界面更新运行的线程；因此，如果我们提交了一个长时间运行的任务，用户界面将冻结，直到该任务完成。

在前面的章节中，我们看到了`dispatch_async()`函数是如何将任务提交到并发和串行队列中的。现在，让我们看看两个额外的函数，我们可以使用它们将任务提交到我们的队列中。我们将首先查看的函数是`dispatch_after()`函数。

### 使用`dispatch_after()`函数

有时候我们需要在延迟后执行任务。如果我们使用线程模型，我们需要创建一个新的线程，执行某种延迟或睡眠函数，然后执行我们的任务。使用 GCD，我们可以使用`dispatch_after()`函数。`dispatch_after()`函数的语法如下：

```swift
func dispatch_after(when: dispatch_time_t, queue: dispatch_queue_t, block: dispatch_block_t)
```

在这里，`dispatch_after()`函数接受以下参数：

+   `when`：这是我们希望队列执行我们的任务的时间

+   `queue`：这是我们想要在队列中执行我们的任务。

+   `block`：这是要执行的任务

与`dispatch_async()`和`dispatch_synch()`函数一样，我们不需要将任务作为参数包含。我们可以在两个大括号之间包含要执行的任务，就像我们之前在`dispatch_async()`和`dispatch_synch()`函数中做的那样。

如我们从`dispatch_after()`函数中看到的，我们使用`dispatch_time_t`类型来定义执行任务的时间。我们使用`dispatch_time()`函数来创建`dispatch_time_t`类型。`dispatch_time()`函数的语法如下：

```swift
func dispatch_time(when: dispatch_time_t, delta:Int64) -> dispatch_time_t
```

在这里，`dispatch_time()`函数接受以下参数：

+   `when`：这个值用作执行任务的时间基础。我们通常传递`DISPATCH_TIME_NOW`值来创建基于当前时间的时。

+   `delta`：这是要添加到`when`参数中的纳秒数，以获取我们的时间。

我们将像这样使用`dispatch_time()`和`dispatch_after()`函数：

```swift
var delayInSeconds = 2.0
let eTime = dispatch_time(DISPATCH_TIME_NOW, Int64(delayInSeconds * Double(NSEC_PER_SEC)))
dispatch_after(eTime, queue2) {
  print("Times Up")
}
```

上述代码将在两秒后执行任务。在`dispatch_time()`函数中，我们创建了一个`dispatch_time_t`类型，表示未来两秒。`NSEC_PER_SEC`常量用于从秒计算纳秒。在两秒的延迟后，我们将消息`Times Up`打印到控制台。

使用`dispatch_after()`函数时要注意的一件事是。让我们看一下以下代码：

```swift
let queue2 = dispatch_queue_create("squeue.hoffman.jon", DISPATCH_QUEUE_SERIAL)

var delayInSeconds = 2.0
let pTime = dispatch_time(DISPATCH_TIME_NOW,Int64(delayInSeconds * Double(NSEC_PER_SEC)))
dispatch_after(pTime, queue2) {
  print("Times Up")
}

dispatch_sync(queue2) {
  calculation.performCalculation(100000, tag: "sync1")
}
```

在此代码中，我们首先创建了一个串行队列，然后向队列中添加了两个任务。第一个任务使用了`dispatch_after()`函数，第二个任务使用了`dispatch_sync()`函数。我们最初的设想可能是，当我们在这个串行队列中执行这段代码时，第一个任务会在两秒后执行，然后才是第二个任务；然而，这并不正确。第一个任务被提交到队列中并立即执行。它也立即返回，这使得队列在等待第一个任务正确执行的时间的同时执行下一个任务。因此，尽管我们在串行队列中运行任务，第二个任务却在第一个任务之前完成。以下是在运行前面的代码时的输出示例：

```swift
time for sync1:  0.00407701730728149
Times Up
```

我们将要查看的最后一个 GCD 函数是`dispatch_once()`。

### 使用`dispatch_once()`函数

`dispatch_once()`函数将只在应用程序的生命周期内执行一次任务，并且只执行一次。这意味着任务将被执行并标记为已执行，然后除非应用程序重新启动，否则该任务将不再执行。虽然`dispatch_once()`函数可以，并且已经被用来实现单例模式，但还有其他更简单的方法可以做到这一点。请参阅第十七章，*在 Swift 中采用设计模式*，了解如何实现单例设计模式的示例。

`dispatch_once()`函数非常适合执行在应用程序最初启动时需要运行的任务。这些初始化任务可以包括初始化我们的数据存储或变量和对象。以下代码显示了`dispatch_once()`函数的语法：

```swift
func dispatch_once(predicate: UnsafeMutablePointer<dispatch_once_t>,block: dispatch_block_t!)
```

让我们看看如何使用`dispatch_once()`函数：

```swift
var token: dispatch_once_t = 0
func example() {
    dispatch_once(&token) {
        print("Printed only on the first call")
    }
    print("Printed for each call")
}
```

在这个例子中，打印消息“仅在第一次调用时打印”的行只会执行一次，无论函数被调用多少次。然而，打印“每次调用都打印”消息的行会在每次函数调用时执行。让我们通过四次调用这个函数来观察这个行为，如下所示：

```swift
for i in 0..<4 {
   example()
}
```

如果我们执行这个示例，我们应该看到以下输出：

```swift
Printed only on the first call
Printed for each call
Printed for each call
Printed for each call
Printed for each call
```

注意，在这个例子中，我们只看到了一次“仅在第一次调用时打印”的消息，而“每次调用都打印”的消息在四次调用函数时都出现了。

现在我们已经了解了 GCD，让我们来看看操作队列。

## 使用 NSOperation 和 NSOperationQueue 类型

`NSOperation` 和 `NSOperationQueues` 类型协同工作，为我们提供了在应用程序中添加并发的替代方案。操作队列是 Cocoa 对象，其功能类似于调度队列，并且内部，操作队列使用 GCD 实现。我们定义要执行的任务（`NSOperations`），然后将任务添加到操作队列（`NSOperationQueue`）。然后，操作队列将处理任务的调度和执行。操作队列是 `NSOperationQueue` 类的实例，操作是 `NSOperation` 类的实例。

操作代表一个单独的工作单元或任务。`NSOperation` 类型是一个抽象类，它提供了一个线程安全的结构来模拟状态、优先级和依赖关系。为了执行任何有用的操作，必须对该类进行子类化。

苹果确实提供了两种 `NSOperation` 类型的具体实现，我们可以直接使用，对于不需要构建自定义子类的情况。这些子类是 `NSBlockOperation` 和 `NSInvocationOperation`。

同时可以存在多个操作队列，实际上，总是至少有一个操作队列在运行。这个操作队列被称为 **主队列**。当应用程序启动时，主队列会自动为主线程创建，并且所有 UI 操作都在这里执行。

我们有几种方法可以使用 `NSOperation` 和 `NSOperationQueues` 类来为我们的应用程序添加并发。在本章中，我们将探讨三种不同的方法。我们将首先查看的是使用 `NSOperation` 抽象类的 `NSBlockOperation` 实现方式。

### 使用 NSOperation 的 NSBlockOperation 实现方式

在本节中，我们将使用与 *Grand Central Dispatch* 部分中相同的 `DoCalculation` 类，以保持我们的队列忙碌于工作，这样我们就可以看到 `NSOperationQueues` 类的工作原理。

`NSBlockOperation` 类是 `NSOperation` 类型的具体实现，可以管理一个或多个块的执行。这个类可以用来同时执行多个任务，而无需为每个任务创建单独的操作。

让我们看看如何使用 `NSBlockOperation` 类来为我们的应用程序添加并发。以下代码展示了如何使用单个 `NSBlockOperation` 实例将三个任务添加到操作队列中：

```swift
let calculation = DoCalculations()
let operationQueue = NSOperationQueue()

let blockOperation1: NSBlockOperation = NSBlockOperation.init(block: {
  calculation.performCalculation(10000000, tag: "Operation 1")
})

blockOperation1.addExecutionBlock(
  {
    calculation.performCalculation(10000, tag: "Operation 2")
  }
)

blockOperation1.addExecutionBlock(
  {
    calculation.performCalculation(1000000, tag: "Operation 3")
  }
)

operationQueue.addOperation(blockOperation1)
```

在此代码中，我们首先创建 `DoCalculation` 类的一个实例和 `NSOperationQueue` 类的一个实例。接下来，我们使用 `init` 构造函数创建 `NSBlockOperation` 类的一个实例。这个构造函数接受一个参数，它是一个代码块，代表我们想在队列中执行的任务之一。然后，我们使用 `addExecutionBlock()` 方法向 `NSBlockOperation` 实例添加两个额外的任务。

这是调度队列和操作之间的一种区别。在调度队列中，如果资源可用，任务会按照它们被添加到队列的顺序执行。在操作中，各个任务不会执行，直到操作本身被提交到操作队列。

一旦我们将所有任务添加到 `NSBlockOperation` 实例中，我们接着将操作添加到我们在代码开头创建的 `NSOperationQueue` 实例中。此时，操作内的各个任务开始执行。

这个例子展示了如何使用 `NSBlockOperation` 将多个任务排队，然后将任务传递给操作队列。任务按照先进先出的顺序执行；因此，第一个添加到 `NSBlockOperation` 实例的任务将是第一个执行的任务。然而，由于如果我们有可用资源，任务可以并发执行，所以这段代码的输出应该看起来像这样：

```swift
time for Operation 2:  0.00546294450759888
time for Operation 3:  0.0800899863243103
time for Operation 1:  0.484337985515594
```

如果我们不希望我们的任务并发运行呢？如果我们希望它们像串行调度队列一样按顺序运行呢？我们可以在我们的操作队列中设置一个属性，该属性定义了队列中可以并发运行的任务数量。这个属性叫做 `maxConcurrentOperationCount`，其使用方式如下：

```swift
operationQueue.maxConcurrentOperationCount = 1
```

然而，如果我们把这一行添加到之前的例子中，它将不会按预期工作。为了了解为什么，我们需要理解这个属性实际上定义了什么。如果我们查看苹果的 `NSOperationQueue` 类参考，属性的说明是：“可以同时执行的最大队列操作数。”

这告诉我们，`maxConcurrentOperationCount` 属性定义了可以同时执行的操作数量（这是关键词）。我们添加所有任务到的 `NSBlockOperation` 实例代表了一个单一的操作；因此，不会执行添加到队列中的其他 `NSBlockOperation`，直到第一个完成，但操作内的各个任务可以并发执行。如果我们想按顺序执行任务，我们需要为每个任务创建一个单独的 `NSBlockOperations` 实例。

如果我们有一系列想要并发执行的任务，使用 `NSBlockOperation` 类的实例是好的，但它们不会开始执行，直到我们将操作添加到操作队列中。让我们看看使用队列的 `addOperationWithBlock()` 方法将任务添加到操作队列的更简单方法。

### 使用操作队列的 `addOperationWithBlock()` 方法

`NSOperationQueue` 类有一个名为 `addOperationWithBlock()` 的方法，这使得向队列中添加代码块变得简单。此方法会自动将代码块包装在操作对象中，然后将该操作传递给队列本身。让我们看看如何使用此方法将任务添加到队列中：

```swift
let operationQueue = NSOperationQueue()
let calculation = DoCalculations()

operationQueue.addOperationWithBlock() {
  calculation.performCalculation(10000000, tag: "Operation1")
}

operationQueue.addOperationWithBlock() {
  calculation.performCalculation(10000, tag: "Operation2")
}

operationQueue.addOperationWithBlock() {
  calculation.performCalculation(1000000, tag: "Operation3")
}
```

在本章前面的`NSBlockOperation`示例中，我们将希望执行的任务添加到了一个`NSBlockOperation`实例中。在这个示例中，我们直接将任务添加到操作队列中，每个任务代表一个完整的操作。一旦我们创建了操作队列的实例，我们就使用`addOperationWithBlock()`方法将任务添加到队列中。

此外，在`NSBlockOperation`示例中，各个任务在所有任务都添加到`NSBlockOperation`对象并且该操作被添加到队列之前不会执行。这个`addOperationWithBlock()`示例与 GCD 示例类似，其中任务在添加到操作队列后立即开始执行。

如果我们运行前面的代码，输出应该类似于以下内容：

```swift
time for Operation2:  0.0115870237350464
time for Operation3:  0.0790849924087524
time for Operation1:  0.520610988140106
```

你会注意到操作是并发执行的。使用这个示例，我们可以通过使用我们之前提到的`maxConcurrentOperationCount`属性来按顺序执行任务。让我们通过以下方式初始化`NSOperationQueue`实例来尝试这一点：

```swift
var operationQueue = NSOperationQueue()
operationQueue.maxConcurrentOperationCount = 1
```

现在，如果我们运行这个示例，输出应该类似于以下内容：

```swift
time for Operation1:  0.418763995170593
time for Operation2:  0.000427007675170898
time for Operation3:  0.0441589951515198
```

在这个示例中，我们可以看到每个任务在开始之前都等待前一个任务完成。

使用`addOperationWithBlock()`方法添加任务，通常比使用`NSBlockOperation`方法更容易；然而，任务将在它们被添加到队列后立即开始执行，这通常是期望的行为。

现在，让我们看看我们如何继承`NSOperation`类来创建一个可以直接添加到操作队列中的操作。

### 继承`NSOperation`类

前两个示例展示了如何将小块代码添加到我们的操作队列中。在这些示例中，我们调用了`DoCalculation`类中的`performCalculations`方法来执行我们的任务。这些示例说明了两种非常好的方法来为已经编写的功能添加并发性，但如果我们希望在设计时为`DoCalculation`类设计并发性，那会怎样呢？为此，我们可以继承`NSOperation`类。

`NSOperation`抽象类提供了大量的基础设施。这使得我们能够非常容易地创建一个子类而不需要做很多工作。我们至少应该提供一个`initialization`方法和一个`main`方法。`main`方法将在队列开始执行操作时被调用：

让我们看看如何将`DoCalculation`类实现为`NSOperation`类的子类；我们将这个新类称为`MyOperation`：

```swift
class MyOperation: NSOperation {
  let iterations: Int
  let tag: String

  init(iterations: Int, tag: String) {
    self.iterations = iterations
    self.tag = tag
  }

  override func main() {
    performCalculation()
  }

  func performCalculation() {
    let start = CFAbsoluteTimeGetCurrent()
    for var i=0; i<iterations; i++ {
      self.doCalc()
    }
    let end = CFAbsoluteTimeGetCurrent()
    print("time for \(tag):  \(end-start)")
  }

  func doCalc() {
    let x=100
    let y = x*x
    _ = y/x
  }
}
```

我们首先定义 `MyOperation` 类是 `NSOperation` 类的子类。在类的实现中，我们定义了两个类常量，它们代表 `performCalculations()` 方法使用的迭代次数和标签。请记住，当操作队列开始执行操作时，它将不带参数调用 `main()` 方法；因此，我们需要传递的任何参数都必须通过初始化器传递。

在这个例子中，我们的初始化器接受两个参数，用于设置 `iterations` 和 `tag` 类常量。然后 `main()` 方法，操作队列将要调用来开始执行操作，简单地调用 `performCalculation()` 方法。

我们现在可以非常容易地将我们的 `MyOperation` 类的实例添加到操作队列中，如下所示：

```swift
var operationQueue = NSOperationQueue()
operationQueue.addOperation(MyOperation(iterations: 10000000, tag: "Operation 1"))
operationQueue.addOperation(MyOperation(iterations: 10000, tag: "Operation 2"))
operationQueue.addOperation(MyOperation(iterations: 1000000, tag: "Operation 3"))
```

如果我们运行此代码，我们将看到以下结果：

```swift
time for Operation 2:  0.00187397003173828
time for Operation 3:  0.104826986789703
time for Operation 1:  0.866684019565582
```

如我们之前所见，我们也可以通过添加以下行来顺序执行任务，该行设置操作队列的 `maxConcurrentOperationCount` 属性：

```swift
operationQueue.maxConcurrentOperationCount = 1
```

如果我们知道在编写代码之前需要并发执行某些功能，我建议像这个例子一样对 `NSOperation` 类进行子类化，而不是使用之前的例子。这给我们提供了最干净的实现；然而，使用 `NSBlockOperation` 类或本节之前描述的 `addOperationWithBlock()` 方法并没有什么问题。

# 摘要

在我们考虑将并发性添加到我们的应用程序之前，我们应该确保我们理解为什么我们要添加它，并问自己这是否是必要的。虽然并发性可以通过将工作从主应用程序线程卸载到后台线程来使我们的应用程序更响应，但它也增加了我们代码的额外复杂性和应用程序的开销。我甚至看到过许多在各种语言中运行得更好的应用程序，在移除一些并发代码之后。这是因为并发性没有经过深思熟虑或计划。考虑到这一点，在我们讨论应用程序预期行为时，思考和讨论并发性总是一个好主意。

在本章的开始，我们讨论了与顺序运行任务相比并行运行任务。我们还讨论了限制特定设备上可以并行运行多少任务的硬件限制。对这些概念有良好的理解对于理解如何在何时将并发性添加到我们的项目中非常重要。

虽然 GCD 并不仅限于系统级应用，但在我们将其应用于我们的应用程序之前，我们应该考虑操作队列是否会更简单、更适合我们的需求。一般来说，我们应该使用满足我们需求的最高级别的抽象。这通常会将我们引向使用操作队列；然而，实际上并没有什么阻止我们使用 GCD，它可能更适合我们的需求。

在操作队列中需要注意的一点是，由于它们是 Cocoa 对象，因此确实会增加额外的开销。对于大多数应用程序来说，这微小的额外开销不应成为问题，甚至可能不会被察觉；然而，对于一些项目，例如需要获取所有可用资源的游戏，这额外的开销可能确实成为一个问题。
