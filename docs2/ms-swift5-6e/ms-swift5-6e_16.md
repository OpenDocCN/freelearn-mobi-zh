

# Swift 中的并发和并行

当我刚开始学习 Objective-C 时，我已经对其他语言（如 C 和 Java）中的并发和多任务处理有了很好的理解。这个背景使我能够很容易地使用线程创建多线程应用程序。然后，当苹果在 OS X 10.6 和 iOS 4 中发布 **Grand Central Dispatch** （**GCD**）时，一切发生了改变。起初，我陷入了否认；GCD 根本不可能比我更好地管理我的应用程序的线程。然后，我进入了愤怒阶段；GCD 难以使用和理解。接下来是讨价还价阶段；也许我可以使用 GCD 和我的线程代码，这样我仍然可以控制线程的工作方式。然后，是抑郁阶段；也许 GCD 确实比我更好地处理线程。最后，我进入了哇哦阶段；这个 GCD 真的很容易使用，并且工作得非常出色。

在使用 GCD 和操作队列进行 Objective-C 开发后，我看不到使用 Swift 中的手动线程的理由。

在本章中，我们将学习以下主题：

+   并发和并行的基础知识

+   如何使用 GCD 创建和管理并发调度队列

+   如何使用 GCD 创建和管理串行调度队列

+   如何使用各种 GCD 函数将任务添加到调度队列中

+   如何使用 `Operation` 和 `OperationQueues` 为我们的应用程序添加并发

在 Swift 5.x 的整个过程中，我们在 Swift 语言中的并发方面并没有看到太多的改进。但似乎这将在未来改变，因为并发改进是 Swift 6 的主要目标之一。让我们首先看看并发和并行之间的区别，这是理解的重要之处。

# 并发和并行

**并发** 是指在相同的时间段内，多个任务开始、运行和完成的概念。这并不一定意味着任务正在同时执行。实际上，为了使任务能够同时执行，我们的应用程序需要在多核或多处理器系统上运行。并发使我们能够为多个任务共享处理器或核心；然而，单个核心在任何给定时间只能执行一个任务。

**并行** 是指两个或更多任务同时运行的概念。由于我们的处理器的每个核心一次只能执行一个任务，因此同时执行的任务数量受限于处理器中的核心数量和我们拥有的处理器数量。例如，如果我们有一个四核处理器，那么我们同时运行的任务数量将受到限制为四个。今天的处理器可以如此快速地执行任务，以至于看起来更大的任务似乎是在同时执行。然而，在系统中，较大的任务实际上是在核心上轮流执行子任务。

为了理解并发和并行之间的区别，让我们看看杂技演员是如何抛接球的。如果你观察一个杂技演员，他们似乎在任何时候都能同时接住和抛出多个球；然而，仔细观察会发现，他们实际上每次只接住和抛出一个球。其他球在空中等待被接住和抛出。如果我们想能够同时接住和抛出多个球，我们需要有多个杂技演员。

这个例子非常好，因为我们可以把杂技演员看作是处理器的核心。单核处理器的系统（一个杂技演员），无论看起来如何，一次只能执行一个任务（接住或抛出一个球）。如果我们想一次执行多个任务，我们需要使用多核处理器（多个杂技演员）。

在所有处理器都是单核的时代，要有一个能够同时执行任务的系统，唯一的方法是在系统中拥有多个处理器。这也需要专门的软件来利用多个处理器。在当今世界，几乎每个设备都有一个拥有多个核心的处理器，iOS 和 macOS 都被设计用来利用这些多个核心来同时运行任务。

传统上，应用程序通过创建多个线程来添加并发性；然而，这种模型在任意数量的核心上扩展性不好。使用线程的最大问题是我们的应用程序运行在各种各样的系统（和处理器的）上，为了优化我们的代码，我们需要知道在给定时间可以高效使用多少核心/处理器，这通常在开发时是未知的。

为了解决这个问题，许多操作系统，包括 iOS 和 macOS，开始依赖异步函数。这些函数通常用于启动可能需要很长时间才能完成的任务，例如发起 HTTP 请求或写入磁盘数据。异步函数通常在任务完成之前启动一个长时间运行的任务，并返回。通常，这个任务在后台运行，并在任务完成时使用回调函数（例如 Swift 中的闭包）。

这些异步函数对于操作系统提供的任务来说效果很好，但如果我们需要创建自己的异步函数而不想自己管理线程怎么办？为此，Apple 提供了一些技术。在本章中，我们将介绍其中两种：GCD 和操作队列。

GCD 是一个基于 C 的低级 API，它允许将特定任务排队以供执行，并在任何可用的处理器核心上调度执行。操作队列与 GCD 类似；然而，它们是 Foundation 对象，并且内部使用 GCD 实现。

让我们从查看最大公约数（GCD）开始。

# 大中枢调度（Grand Central Dispatch, GCD）

在 Swift 3 之前，使用 GCD 感觉像是编写低级 C 代码。API 有点笨拙，有时难以理解，因为它没有使用 Swift 语言设计中的任何特性。这一切都在 Swift 3 中发生了变化，因为苹果承担了重写 API 的任务，以便它符合 Swift 3 API 指南。

GCD 提供了所谓的调度队列来管理提交的任务。队列管理这些提交的任务，并以**先进先出**（**FIFO**）的顺序执行它们。这确保了任务是以提交的顺序启动的。

任务只是我们应用程序需要执行的一些工作。例如，我们可以创建执行简单计算、读写磁盘数据、发起 HTTP 请求或我们应用程序需要做的任何其他任务的作业。我们通过将代码放在函数或闭包中并将它们添加到调度队列中来定义这些任务。

GCD 提供了三种类型的调度队列：

+   **串行队列**：串行队列中的任务（也称为**私有队列**）按提交的顺序一次执行一个。只有在前一个任务完成后，才会启动每个任务。串行队列通常用于同步访问特定资源，因为我们保证串行队列中的两个任务永远不会同时运行。因此，如果访问特定资源的唯一方式是通过串行队列中的任务，那么两个任务将不会同时或顺序错误地尝试访问资源。

+   **并发队列**：并发队列中的任务（也称为**全局调度队列**）是并发执行的；然而，任务的启动顺序仍然是它们被添加到队列中的顺序。在任何给定时刻可以执行的任务的确切数量是可变的，并且取决于系统的当前条件和资源。何时启动任务的决定权在 GCD，而不是我们可以在应用程序内部控制的事情。

+   **主调度队列**：主调度队列是一个全局可用的串行队列，它在应用程序的主线程上执行任务。由于放入主调度队列的任务在主线程上运行，因此它通常在后台处理完成并且用户界面需要更新时从后台队列中调用。

调度队列相对于传统线程提供了几个优势。首要的优势是，使用调度队列时，系统处理线程的创建和管理，而不是应用程序本身。系统可以根据系统的总体可用资源和当前系统条件动态地调整线程的数量。这意味着调度队列可以比我们更有效地管理线程。

调度队列的另一个优点是我们能够控制任务启动的顺序。对于串行队列，我们不仅控制任务启动的顺序，而且还确保在先前的任务完成之前不会启动另一个任务。使用传统的线程，这可能会非常繁琐且难以实现，但正如我们将在本章后面看到的那样，使用调度队列则相当容易。

## 计算类型

在我们查看如何使用调度队列之前，让我们创建一个类来帮助我们演示各种队列类型的工作原理。这个类将包含两个基本函数，我们将这个类命名为 `DoCalculations`。第一个函数将简单地执行一些基本计算，然后返回一个值。以下是这个函数的代码，该函数名为 `doCalc()`:

```swift
func doCalc() { 
    let x = 100
    let y = x*x
    _ = y/x
} 
```

另一个函数，我们将命名为 `performCalculation()`，接受两个参数。一个是名为 `iterations` 的整数，另一个是名为 `tag` 的字符串。`performCalculation()` 函数重复调用 `doCalc()` 函数，直到它调用的次数与 `iterations` 参数定义的次数相同。我们还使用 `CFAbsoluteTimeGetCurrent()` 函数来计算执行所有迭代所需的时间，然后我们使用 `tag` 字符串将经过的时间打印到控制台。这将让我们知道函数何时完成以及完成它所需的时间。以下是这个函数的代码：

```swift
func performCalculation(_ iterations: Int, tag: String) { 
    let start = CFAbsoluteTimeGetCurrent()
    for _ in 0 ..< iterations { 
        self.doCalc()
    }
    let end = CFAbsoluteTimeGetCurrent() 
    print("time for \(tag):\(end-start)")
} 
```

这些函数将一起使用，以使我们的队列保持忙碌，这样我们就可以看到它们是如何工作的。让我们首先看看我们如何创建一个调度队列。

## 创建队列

我们使用 `DispatchQueue` 初始化器来创建一个新的调度队列。以下代码展示了我们如何创建一个新的调度队列：

```swift
let concurrentQueue = DispatchQueue(label: "cqueue.hoffman.jon",                       attributes: .concurrent)
let serialQueue = DispatchQueue(label: "squeue.hoffman.jon") 
```

第一行将创建一个带有标签 `cqueue.hoffman.jon` 的并发队列，而第二行将创建一个带有标签 `squeue.hoffman.jon` 的串行队列。`DispatchQueue` 初始化器接受以下参数：

+   `label`: 这是一个字符串标签，它附加到队列上，以在调试工具（如仪器和崩溃报告）中唯一标识它。建议我们使用反向 DNS 命名约定。此参数是可选的，可以是 `nil`。

+   `attributes`: 这指定了要创建的队列类型。这可以是 `DispatchQueue.Attributes.serial`、`DispatchQueue.Attributes.concurrent` 或 `nil`。如果这个参数是 `nil`，则创建一个串行队列。您可以使用 `.serial` 或 `.concurrent`，就像我们在示例代码中所展示的那样。

一些编程语言使用反向 DNS 命名约定来命名某些组件。这个约定基于一个注册的域名，并将其反转。例如，如果我们为一家名为 `mycompany.com` 的公司工作，该公司有一个名为 widget 的产品，那么反向 DNS 名称将是 `com.mycompany.widget`。

现在让我们看看我们如何创建和使用并发队列。

## 创建和使用并发队列

并发队列将按照 FIFO 顺序执行任务；然而，任务将并发执行并按任意顺序完成。让我们看看我们如何创建和使用一个并发队列。以下行将创建我们将用于本节的并发队列，并将创建一个`DoCalculations`类型的实例，该实例将用于测试队列：

```swift
let cqueue = DispatchQueue(label: "cqueue.hoffman.jon",              attributes:.concurrent)
let calculation = DoCalculations() 
```

第一行将创建一个新的调度队列，我们将命名为`cqueue`，第二行创建一个`DoCalculations`类型的实例。现在，让我们看看我们如何使用并发队列，通过使用`DoCalculations`类型的`performCalculation()`方法来执行一些计算：

```swift
let c = {calculation.performCalculation(1000, tag: "async1")}
cqueue.async(execute: c) 
```

在前面的代码中，我们创建了一个闭包，它代表我们的任务，并简单地调用`DoCalculation`实例的`performCalculation()`函数，请求它运行`doCalc()`函数的 1,000 次迭代。最后，我们使用队列的`async(execute:)`方法来执行它。此代码将在并发调度队列中执行任务，该队列与主线程分开。

虽然前面的例子工作得很好，但实际上我们可以稍微缩短代码。下一个例子显示，我们实际上不需要像前面例子中那样创建一个单独的闭包。我们也可以像下面这样提交任务来执行：

```swift
cqueue.async {
    calculation.performCalculation(1000, tag: "async1")
} 
```

这种简写版本是我们通常向队列提交小代码块的方式。如果我们有更大的任务或需要多次提交的任务，我们通常希望创建一个闭包，并将闭包提交到队列中，就像我们在第一个例子中展示的那样。

让我们通过向队列中添加几个项目并查看它们的返回顺序和时间来了解并发队列的工作方式。以下代码将向队列中添加三个任务。每个任务将使用不同的迭代计数调用`performCalculation()`函数。

记住，`performCalculation()`函数将连续执行计算例程，直到执行了传入的迭代次数。因此，我们传入函数的迭代计数越大，它应该执行的时间就越长。让我们看看以下代码：

```swift
cqueue.async {
    calculation.performCalculation(10_000_000, tag: "async1")
}
cqueue.async {
    calculation.performCalculation(1000, tag: "async2")
}
cqueue.async {
    calculation.performCalculation(100_000, tag: "async3")
} 
```

注意，每个函数在`tag`参数中调用时都使用不同的值。由于`performCalculation()`函数会打印出带有经过时间的`tag`变量，我们可以看到任务完成的顺序以及它们执行所需的时间。如果我们执行前面的代码，我们应该看到类似以下的结果：

```swift
time for async2: 0.000200986862182617
time for async3: 0.00800204277038574
time for async1: 0.461670994758606 
```

经过的时间会因运行而异，也会因系统而异。

由于队列按照 FIFO（先进先出）的顺序工作，具有`async1`标签的任务首先被执行。然而，从结果中我们可以看到，它是最后一个完成的任务。由于这是一个并发队列，如果可能的话（如果系统有可用资源），代码块将并发执行。这就是为什么带有`async2`和`async3`标签的任务在带有`async1`标签的任务之前完成，尽管`async1`任务的执行在其他两个任务之前开始。

现在，让我们看看一个串行队列是如何执行任务的。

## 创建和使用串行队列

串行队列与并发队列的工作方式略有不同。串行队列一次只执行一个任务，并在开始下一个任务之前等待当前任务完成。这个队列，就像并发调度队列一样，遵循 FIFO 顺序。以下代码行将创建我们将用于本节的串行队列，并将创建一个`DoCalculations`类型的实例：

```swift
let squeue = DispatchQueue(label: "squeue.hoffman.jon")
let calculation = DoCalculations() 
```

第一行将创建一个新的名为`squeue`的串行调度队列，第二行创建一个`DoCalculations`类型的实例。现在，让我们看看我们如何使用我们的串行队列，通过使用`DoCalculations`类型的`performCalculation()`方法来执行一些计算：

```swift
let s = {calculation.performCalculation(1000, tag: "async1")}
squeue.async (execute: s) 
```

在前面的代码中，我们创建了一个闭包，它代表我们的任务，简单地调用`DoCalculation`实例的`performCalculation()`函数，请求它运行`doCalc()`函数的 1,000 次迭代。最后，我们使用队列的`async(execute:)`方法来执行它。此代码将在一个串行调度队列中执行任务，该队列与主线程分开。从这段代码中我们可以看到，我们使用串行队列的方式与使用并发队列的方式完全相同。

我们可以稍微缩短这段代码，就像我们处理并发队列时做的那样。以下示例展示了我们如何使用串行队列来完成这个操作：

```swift
squeue.async {
    calculation.performCalculation(1000, tag: "async2")
} 
```

让我们通过向队列中添加几个项目并查看它们完成的顺序来了解串行队列的工作方式。以下代码将添加三个任务到队列中，这些任务将使用不同的迭代次数调用`performCalculation()`函数：

```swift
squeue.async {
    calculation.performCalculation(100000, tag: "async1")
}
squeue.async {
    calculation.performCalculation(1000, tag: "async2")
}
squeue.async {
    calculation.performCalculation(100000, tag: "async3")
} 
```

正如我们在并发队列示例中所做的那样，我们使用不同的迭代次数和不同的`tag`参数值调用`performCalculation()`函数。由于`performCalculation()`函数会打印出带有经过时间的`tag`字符串，我们可以看到任务的完成顺序和执行所需的时间。如果我们执行此代码，我们应该看到以下结果：

```swift
time for async1: 0.00648999214172363
time for async2: 0.00009602308273315
time for async3: 0.00515800714492798 
```

运行时间会因运行次数和系统而异。

与并发队列不同，我们可以看到任务是以它们提交的顺序完成的，尽管`sync2`和`sync3`任务的完成时间明显更短。这表明串行队列一次只执行一个任务，并且队列在开始下一个任务之前会等待每个任务完成。

在之前的示例中，我们使用了`async`方法来执行代码块。我们也可以使用`sync`方法。

## 异步与同步

在之前的示例中，我们使用了`async`方法来执行代码块。当我们使用`async`方法时，调用将不会阻塞当前线程。这意味着该方法返回，并且代码块是异步执行的。

而不是使用`async`方法，我们可以使用`sync`方法来执行代码块。`sync`方法将阻塞当前线程，这意味着它不会返回直到代码执行完成。通常，我们使用`async`方法，但有一些情况下`sync`方法是有用的。这些用例通常是我们有一个单独的线程，并且我们希望该线程等待某些工作完成。

## 在主队列上执行代码

`DispatchQueue.main.async(execute:)`函数将在应用程序的主队列上执行代码。我们通常在想要从另一个线程或队列更新我们的代码时使用此函数。

当应用程序启动时，会自动为主线程创建主队列。这个主队列是一个串行队列；因此，队列中的项目将按它们提交的顺序一次执行一个。我们通常希望避免使用此队列，除非我们有必要从后台线程更新用户界面。

以下代码示例显示了我们将如何使用此函数：

```swift
let squeue = DispatchQueue(label: "squeue.hoffman.jon") 
squeue.async {
    let resizedImage = image.resize(to: rect) 
    DispatchQueue.main.async {
        picView.image = resizedImage
    }
} 
```

在之前的代码中，我们假设我们已经向`UIImage`类型添加了一个方法，该方法将调整图像大小。在这段代码中，我们创建了一个新的串行队列，并在该队列中调整图像大小。这是一个很好的使用调度队列的例子，因为我们不希望在主队列上调整图像大小，因为这会在调整图像大小时冻结 UI。一旦图像调整大小，我们就需要使用新的图像更新`UIImageView`；然而，所有对 UI 的更新都需要在主线程上发生。因此，我们将使用`DispatchQueue.main.async`函数在主队列上执行更新。

有时会需要延迟执行任务。如果我们使用线程模型，我们需要创建一个新的线程，执行某种`delay`或`sleep`函数，然后执行我们的任务。使用 GCD，我们可以使用`asyncAfter`函数。

## 使用 asyncAfter

`asyncAfter`函数将在给定延迟后异步执行代码块。当我们需要暂停代码执行时，这非常有用。以下代码示例显示了我们将如何使用`asyncAfter`函数：

```swift
let queue2 = DispatchQueue(label: "squeue.hoffman.jon") 
let delayInSeconds = 2.0
let pTime = DispatchTime.now() + Double(delayInSeconds * Double(NSEC_PER_SEC)) / Double(NSEC_PER_SEC) 
queue2.asyncAfter(deadline: pTime) {
    print("Time's Up")
} 
```

在此代码中，我们首先创建一个串行调度队列。然后创建一个`DispatchTime`类型的实例，并基于当前时间计算执行代码块所需的时间。接着，我们使用`asyncAfter`函数在延迟后执行代码块。

现在，我们已经了解了 GCD，让我们来看看操作队列。

# 使用`Operation`和`OperationQueue`类型

`Operation`和`OperationQueue`类型协同工作，为我们提供了在应用程序中添加并发的 GCD 替代方案。操作队列是 Foundation 框架的一部分，它们作为 GCD 的高级抽象，其功能类似于调度队列。

我们定义我们希望执行的作业（操作），然后将任务添加到操作队列中。操作队列将随后处理任务的调度和执行。操作队列是`OperationQueue`类的实例，而操作是`Operation`类的实例。

一个操作代表一个单独的工作单元或任务。`Operation`类型是一个抽象类，它提供了一个线程安全的结构来模拟状态、优先级和依赖关系。这个类必须被子类化以执行任何有用的操作；我们将在本章的“子类化`Operation`类”部分中查看如何子类化这个类。

苹果为我们提供了一个`Operation`类型的具体实现，我们可以直接使用它，在不需要构建自定义子类的情况下。这个子类是`BlockOperation`。

同时可以存在多个操作队列，实际上，总是至少有一个操作队列在运行。这个操作队列被称为**主队列**。主队列在应用程序启动时自动为主线程创建，并且所有 UI 操作都在这里执行。

在操作队列中需要注意的一点是，由于它们是 Foundation 对象，因此会添加额外的开销。对于大多数应用程序来说，这微小的额外开销不应成为问题，甚至可能不会被注意到；然而，对于一些项目，例如需要尽可能多资源的游戏，这个额外的开销可能确实是一个问题。

我们可以使用多种方式将`Operation`和`OperationQueue`类用于向我们的应用程序添加并发。在本章中，我们将探讨这三种方式中的一种。我们将首先查看`Operation`抽象类的`BlockOperation`实现的使用。

## 使用`BlockOperation`

在本节中，我们将使用与*Grand Central Dispatch (GCD)*部分中相同的`DoCalculation`类，以保持我们的队列忙碌于工作，这样我们就可以看到`OperationQueue`类是如何工作的。

`BlockOperation`类是`Operation`类型的具体实现，它可以管理一个或多个代码块的执行。这个类可以用来同时执行多个任务，而无需为每个任务创建单独的操作。

让我们看看如何使用`BlockOperation`类来为我们的应用程序添加并发性。以下代码展示了如何使用单个`BlockOperation`实例将三个任务添加到操作队列中：

```swift
let calculation = DoCalculations()
let blockOperation1: BlockOperation = BlockOperation.init(
    block: {
        calculation.performCalculation(10000000, tag: "Operation 1")
    }
)
blockOperation1.addExecutionBlock({ 
        calculation.performCalculation(10000, tag: "Operation 2")
    }
)
blockOperation1.addExecutionBlock({ 
        calculation.performCalculation(1000000, tag: "Operation 3")
    }
)
let operationQueue = OperationQueue() 
operationQueue.addOperation(blockOperation1) 
```

在此代码中，我们首先创建`DoCalculation`类的一个实例和`OperationQueue`类的一个实例。接下来，我们使用`init`构造函数创建`BlockOperation`类的一个实例。此构造函数接受一个参数，它是一个代码块，代表我们想在队列中执行的任务之一。然后，我们使用`addExecutionBlock()`方法添加两个额外的任务。

分发队列和操作之间的一个区别是，在分发队列中，如果资源可用，任务将随着它们被添加到队列而执行。在操作中，各个任务不会执行，直到操作本身被提交到操作队列。这允许我们在执行之前将所有操作初始化到一个单独的操作中。

一旦我们将所有任务添加到`BlockOperation`实例中，我们接着将操作添加到我们在代码开头创建的`OperationQueue`实例中。此时，操作内的各个任务开始执行。

这个例子展示了如何使用`BlockOperation`来排队多个任务，然后将任务传递到操作队列。任务按照 FIFO（先进先出）的顺序执行；因此，首先添加的任务将是第一个执行的任务。然而，如果我们有可用的资源，任务可以并发执行。

此代码的输出应类似于以下内容：

```swift
time for Operation 2: 0.00546294450759888
time for Operation 3: 0.0800899863243103
time for Operation 1: 0.484337985515594 
```

如果我们不想让任务并发运行呢？如果我们想像串行分发队列一样按顺序运行它们怎么办？我们可以在操作队列中设置一个属性，该属性定义了队列中可以并发运行的任务数量。该属性名为`maxConcurrentOperationCount`，使用方式如下：

```swift
operationQueue.maxConcurrentOperationCount = 1 
```

然而，如果我们将此行添加到我们之前的示例中，它将不会按预期工作。为了了解为什么，我们需要理解这个属性实际上定义了什么。如果我们查看苹果的`OperationQueue`类参考，这个属性的定义是*可以同时执行的最大排队操作数量*。

这告诉我们，这个属性定义了可以同时执行的操作数量（这是关键词）。我们添加所有任务的`BlockOperation`实例代表一个单一的操作；因此，直到第一个操作完成，队列中不会执行其他添加的`BlockOperation`，但操作内的各个任务可以并发执行。如果我们想按顺序运行任务，我们需要为每个任务创建一个单独的`BlockOperation`实例。

如果我们有几个任务想要并发执行，使用`BlockOperation`类的实例是好的，但它们将不会开始执行，直到我们将操作添加到操作队列中。让我们看看如何使用`addOperationWithBlock()`方法以更简单的方式将任务添加到操作队列。

## 使用操作队列的`addOperation()`方法

`OperationQueue`类有一个名为`addOperation()`的方法，这使得将代码块添加到队列变得容易。此方法自动将代码块包装在操作对象中，然后将该操作传递到队列。让我们看看如何使用此方法将任务添加到队列：

```swift
let operationQueue = OperationQueue() 
let calculation = DoCalculations()
operationQueue.addOperation() {
    calculation.performCalculation(10000000, tag: "Operation1")
}
operationQueue.addOperation() { 
    calculation.performCalculation(10000, tag: "Operation2")
}
operationQueue.addOperation() { 
    calculation.performCalculation(1000000, tag: "Operation3")
} 
```

在本章前面提到的`BlockOperation`例子中，我们将想要执行的任务添加到了`BlockOperation`实例中。在这个例子中，我们将任务直接添加到操作队列中，每个任务代表一个完整的操作。一旦我们创建了操作队列的实例，我们就使用`addOperation()`方法将任务添加到队列。

此外，在`BlockOperation`例子中，各个任务在所有任务都添加完毕并且该操作被添加到队列之前不会执行。这个例子与 GCD 例子相似，其中任务在添加到操作队列后立即开始执行。

如果我们运行前面的代码，输出应该类似于以下内容：

```swift
time for Operation2: 0.0115870237350464
time for Operation3: 0.0790849924087524
time for Operation1: 0.520610988140106 
```

你会注意到操作是并发执行的。通过这个例子，我们可以通过使用前面提到的`maxConcurrentOperationCount`属性来按顺序执行任务。让我们通过以下方式初始化`OperationQueue`实例来尝试这一点：

```swift
var operationQueue = OperationQueue() 
operationQueue.maxConcurrentOperationCount = 1 
```

现在，如果我们运行这个例子，输出应该类似于以下内容：

```swift
time for Operation1: 0.418763995170593 
time for Operation2: 0.000427007675170898 
time for Operation3: 0.0441589951515198 
```

在这个例子中，我们可以看到每个任务都在开始之前等待前一个任务完成。

使用`addOperation()`方法将任务添加到操作队列通常比使用`BlockOperation`方法更容易；然而，任务将在它们被添加到队列后立即开始执行。这通常是期望的行为，尽管在某些情况下我们可能不希望任务在所有操作都添加到队列后才开始执行，就像我们在`BlockOperation`例子中看到的那样。

现在，让我们看看我们如何可以继承`Operation`类来创建一个可以直接添加到操作队列的操作。

## 继承`Operation`类

前两个例子展示了如何将小块代码添加到我们的操作队列中。在这些例子中，我们调用`DoCalculation`类中的`performCalculations`方法来执行我们的任务。这些例子说明了两种非常好的方法来为已经编写的功能添加并发性，但如果我们希望在设计时为`DoCalculation`类本身设计并发性怎么办？为此，我们可以继承`Operation`类。

`Operation`抽象类提供了大量的基础设施。这使我们能够非常容易地创建一个子类而不需要做很多工作。我们需要提供一个初始化方法和一个`main`方法。当队列开始执行操作时，将调用`main`方法。

让我们看看如何将`DoCalculation`类作为`Operation`类的子类来实现；我们将这个新类称为`MyOperation`：

```swift
class MyOperation: Operation { 
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
        for _ in 0 ..< iterations {
            self.doCalc()
    }
        let end = CFAbsoluteTimeGetCurrent() 
        print("time for \(tag):\(end-start)")
    }
    func doCalc() {
        let x=100
        let y = x*x
        _ = y/x
    }
} 
```

我们首先定义`MyOperation`类是`Operation`类的子类。在类的实现中，我们定义了两个类常量，它们代表了`performCalculations()`方法使用的迭代次数和标签。记住，当操作队列开始执行操作时，它将不带参数调用`main()`方法；因此，我们需要传递给它的任何参数都必须通过初始化器传递。

在这个例子中，我们的初始化器接受两个参数，用于设置`iterations`和`tag`类常量。然后，操作队列将要调用的`main()`方法，简单地调用`performCalculation()`方法。

现在我们可以非常容易地将我们的`MyOperation`类实例添加到操作队列中，如下所示：

```swift
let operationQueue = NSOperationQueue() 
    operationQueue.addOperation( MyOperation(iterations: 10000000, tag:"Operation 1")
)
operationQueue.addOperation( 
    MyOperation(iterations: 10000, tag:"Operation 2")
)
operationQueue.addOperation(
    MyOperation(iterations: 1000000, tag:"Operation 3")
) 
```

如果我们运行此代码，我们将看到以下结果：

```swift
time for Operation 2: 0.00187397003173828
time for Operation 3: 0.104826986789703
time for Operation 1: 0.866684019565582 
```

如我们之前所见，我们也可以通过将操作队列的`maxConcurrentOperationCount`属性设置为`1`来串行执行任务。

如果我们知道在编写代码之前需要并发执行某些功能，我建议像这个示例中那样对`Operation`类进行子类化，而不是使用之前的示例。这给我们提供了最干净的实现；然而，使用本节前面描述的`BlockOperation`类或`addOperation()`方法并没有什么不妥。

在考虑将并发添加到我们的应用程序之前，我们应该确保我们理解为什么要添加它，并问自己这是否必要。虽然并发可以通过将工作从主应用程序线程卸载到后台线程来使我们的应用程序更响应，但它也增加了代码的额外开销和复杂性。我甚至看到过许多在各种语言中运行得更好的应用程序，在移除一些并发代码之后。这是因为并发没有经过深思熟虑或计划。

# 摘要

在本章的开头，我们讨论了并发运行任务与并行运行任务的区别。我们还讨论了限制在特定设备上可以并行运行多少任务的硬件限制。对这些概念有良好的理解对于理解何时以及如何将并发添加到我们的项目中非常重要。

我们学习了 GCD 和操作队列，这两种实现并发的方式。虽然 GCD 并不局限于系统级应用，但在我们将其应用于我们的应用程序之前，我们应该考虑操作队列是否更容易且更适合我们的需求。一般来说，我们应该使用满足我们需求的最高级别的抽象。这通常会将我们引向使用操作队列；然而，实际上并没有什么阻止我们使用 GCD，而且它可能更适合我们的需求。

我们应该始终考虑是否需要在我们的应用程序中添加并发。在讨论应用程序的预期行为时思考并讨论并发是一个好主意。

在下一章中，我们将探讨一些高级主题以及在我们创建自己的自定义类型时需要考虑的事项。
