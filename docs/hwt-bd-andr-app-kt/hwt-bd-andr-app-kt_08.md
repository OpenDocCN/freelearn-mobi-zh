# 第八章：8. 服务、WorkManager 和通知

概述

本章将向您介绍在应用程序的后台管理长时间运行任务的概念。通过本章结束时，您将能够触发后台任务，为用户创建通知，当后台任务完成时启动应用程序。本章将使您对如何管理后台任务并让用户了解这些任务的进度有一个扎实的理解。

# 介绍

在上一章中，我们学习了如何从用户那里请求权限并使用谷歌的地图 API。有了这些知识，我们获得了用户的位置，并允许他们在本地地图上部署特工。在本章中，我们将学习如何跟踪长时间运行的进程，并向用户报告其进度。

我们将构建一个示例应用程序，假设**秘密猫特工**（**SCAs**）在 15 秒内部署。这样，我们就不必等待很长时间才能完成后台任务。当猫成功部署时，我们将通知用户，并让他们启动应用程序，向他们呈现成功部署的消息。

移动世界中，长时间运行的后台任务非常常见。即使应用程序不活跃，后台任务也会运行。长时间运行的后台任务的例子包括文件下载、资源清理作业、播放音乐和跟踪用户位置。在历史上，谷歌为 Android 开发者提供了多种执行此类任务的方式：服务、`JobScheduler`、Firebase 的`JobDispatcher`和`AlarmManager`。随着 Android 世界的碎片化，处理这些任务变得非常混乱。幸运的是，自 2019 年 3 月以来，我们有了更好（更稳定）的选择。随着`WorkManager`的推出，谷歌已经为我们抽象出了根据 API 版本选择后台执行机制的逻辑。我们仍然使用前台服务，这是一种特殊类型的服务，用于在运行中的应用程序中应用用户应该知道的某些任务，比如播放音乐或跟踪用户的位置。

在我们继续之前，先快速回顾一下。我们已经提到了服务，我们将专注于前台服务，但我们还没有完全解释服务是什么。服务是设计为在后台运行的应用程序组件，即使应用程序不运行。除了与通知相关联的前台服务外，服务没有用户界面。重要的是要注意，服务在其托管进程的主线程上运行。这意味着它们的操作可能会阻塞应用程序。我们需要在服务内部启动一个单独的线程来避免这种情况。

让我们开始看一下 Android 中管理后台任务的多种方法的实现。

# 使用 WorkManager 启动后台任务

我们将在这里首先要解决的问题是，我们应该选择`WorkManager`还是前台服务？要回答这个问题，一个很好的经验法则是问：您是否需要用户实时跟踪操作？如果答案是肯定的（例如，如果您有任务，如响应用户位置或在后台播放音乐），那么您应该使用前台服务，并附加通知以向用户实时指示状态。当后台任务可以延迟或不需要用户交互时（例如，下载大文件），请使用`WorkManager`。

注意

从`WorkManager`的 2.3.0-alpha02 版本开始，您可以通过调用`setForegroundAsync(ForegroundInfo)`来启动前台服务。我们对前台服务的控制相当有限。它确实允许您将（预定义的）通知附加到工作中，这就是值得一提的原因。

在我们的例子中，在我们的应用程序中，我们将跟踪 SCA 的部署准备。在特工出发之前，他们需要伸展、梳理毛发、去猫砂盆和穿上衣服。每一个任务都需要一些时间。因为你不能催促一只猫，特工将在自己的时间内完成每一步。我们能做的就是等待（并在任务完成时通知用户）。`WorkManager`对于这样的情况非常合适。

要使用`WorkManager`，我们需要熟悉它的四个主要类：

+   第一个是`WorkManager`本身。`WorkManager`接收工作并根据提供的参数和约束（如互联网连接和设备充电）对其进行排队。

+   第二个是`Worker`。现在，`Worker`是需要执行的工作的包装器。它有一个函数`doWork()`，我们重写它来实现后台工作代码。`doWork()`将在后台线程中执行。

+   第三个类是`WorkRequest`。这个类将`Worker`类与参数和约束绑定在一起。有两种类型的`WorkRequest`：`OneTimeWorkRequest`，它运行一次工作，和`PeriodicWorkRequest`，它可以用来安排工作以固定间隔运行。

+   第四个类是`ListenableWorker.Result`。你可能已经猜到了，但这是保存执行工作结果的类。结果可以是`Success`、`Failure`或`Retry`中的一个。

除了这四个类，我们还有`Data`类，它保存了传递给工作者和从工作者传递出来的数据。

让我们回到我们的例子。我们想定义四个需要按顺序发生的任务：猫需要伸展，然后它需要梳理毛发，然后去猫砂盆，最后，它需要穿上衣服。

在我们开始使用`WorkManager`之前，我们必须首先在我们的应用程序`build.gradle`文件中包含其依赖项：

```kt
implementation "androidx.work:work-runtime:2.4.0"
```

有了`WorkManager`包含在我们的项目中，我们将继续创建我们的工作者。第一个工作者将如下所示：

```kt
class CatStretchingWorker(
    context: Context,
    workerParameters: WorkerParameters
) : Worker(context, workerParameters) {
    override fun doWork(): Result {
        val catAgentId = inputData.getString(INPUT_DATA_CAT_AGENT_ID)
        Thread.sleep(3000L)
        val outputData = Data.Builder()
            .putString(OUTPUT_DATA_CAT_AGENT_ID, catAgentId)
            .build()
        return Result.success(outputData)
    }
    companion object {
        const val INPUT_DATA_CAT_AGENT_ID = "id"
        const val OUTPUT_DATA_CAT_AGENT_ID = "id"
    }
}
```

我们首先通过扩展`Worker`并重写其`doWork()`函数来开始。然后，我们从输入数据中读取 SCA ID。然后，因为我们没有真正的传感器来跟踪猫伸展的进度，我们通过引入一个 3 秒（3,000 毫秒）的`Thread.sleep(Long)`调用来伪造等待。最后，我们用我们在输入中收到的 ID 构造一个输出数据类，并将其与成功的结果一起返回。

一旦我们为所有任务创建了工作者（`CatStretchingWorker`、`CatFurGroomingWorker`、`CatLitterBoxSittingWorker`和`CatSuitUpWorker`），类似于我们创建第一个工作者的方式，我们可以调用`WorkManager`来将它们链接起来。假设我们无法在没有连接到互联网时了解特工的进度。我们的调用将如下所示：

```kt
val catStretchingInputData = Data.Builder()
  .putString(CatStretchingWorker.INPUT_DATA_CAT_AGENT_ID, 
    "catAgentId").build()
val catStretchingRequest = OneTimeWorkRequest
  .Builder(CatStretchingWorker::class.java)
val catStretchingRequest =   OneTimeWorkRequest.Builder(CatStretchingWorker::class.java)
    .setConstraints(networkConstraints)
    .setInputData(catStretchingInputData)
    .build()
...
WorkManager.getInstance(this).beginWith(catStretchingRequest)
    .then(catFurGroomingRequest)
    .then(catLitterBoxSittingRequest)
    .then(catSuitUpRequest)
    .enqueue()
```

在上述代码中，我们首先构造了一个`Constraints`实例，声明我们需要连接到互联网才能执行工作。然后，我们定义了我们的输入数据，将其设置为 SCA ID。接下来，我们通过构造`OneTimeWorkRequest`将约束和输入数据绑定到我们的`Worker`类。其他`WorkRequest`实例的构造已经被省略了，但它们与这里显示的基本相同。现在我们可以将所有请求链接起来并将它们排队到`WorkManager`类上。您可以通过直接将单个`WorkRequest`实例传递给`WorkManager`的`enqueue()`函数来排队一个单独的`WorkRequest`实例，或者您也可以通过将它们全部传递给`WorkManager`的`enqueue()`函数作为列表来并行运行多个`WorkRequest`实例。

当满足约束时，我们的任务将由`WorkManager`执行。

每个`Request`实例都有一个唯一的标识符。`WorkManager`为每个请求公开了一个`LiveData`属性，允许我们通过传递其唯一标识符来跟踪其工作的进度，如下面的代码所示：

```kt
workManager.getWorkInfoByIdLiveData(catStretchingRequest.id)
    .observe(this, Observer { info ->
        if (info.state.isFinished) {
            doSomething()
        }
    })
```

最后，还有 `Result.retry`。返回此结果会告诉 `WorkManager` 类重新排队工作。决定何时再次运行工作的策略由设置在 `WorkRequest` `Builder` 上的 `backoff` 标准定义。默认的 `backoff` 策略是指数的，但我们也可以将其设置为线性的。我们还可以定义初始的 `backoff` 时间。

这将为 `Worker` 实现添加所需的依赖项，然后扩展 `Worker` 类。要实现实际的工作，你将重写 `doWork(): Result`，使其从输入中读取 Cat Agent ID，休眠 `3` 秒（`3000` 毫秒），使用 Cat Agent ID 构造一个输出数据实例，并将其传递到 `Result.success` 值中。

在这个第一个练习中，我们将跟踪 SCA 在准备出发时通过排队的链式 `WorkRequest` 类：

在这一部分，我们将从我们发出部署到现场的命令开始跟踪我们的 SCA，直到它到达目的地。

## 要定义一个将休眠 `3` 秒的 `Worker` 实例，更新新类如下：

练习 8.01：使用 WorkManager 类执行后台工作

1.  首先创建一个新的 `Empty Activity` 项目（`File -> New -> New Project -> Empty Activity`）。点击 `Next`。

1.  让我们在接下来的练习中实践到目前为止所学到的知识。

1.  确保你在 `Project` 窗格中处于 Android 视图。

1.  确保你的包名是 `com.example.catagenttracker`。

1.  将其他所有内容保持默认值，然后点击 `Finish`。

1.  将以下内容添加到 `onCreate(Bundle?)` 函数中：

1.  打开你的应用程序的 `build.gradle` 文件。在 `dependencies` 块中，添加 `WorkManager` 依赖项：

```kt
dependencies {
    implementation "org.jetbrains.kotlin:kotlin-stdlib:$kotlin_version"
    ...
    WorkManager and its dependencies in your code.
```

1.  工作的状态可以是 `BLOCKED`（存在一系列请求，它不是下一个请求）、`ENQUEUED`（存在一系列请求，这项工作是下一个请求）、`RUNNING`（`doWork()` 中的工作正在执行）和 `SUCCEEDED`。工作也可以被取消，导致 `CANCELLED` 状态，或者失败，导致 `FAILED` 状态。

1.  用户可见的后台操作 - 使用前台服务

1.  在 `com.example.catagenttracker.worker` 下创建一个名为 `CatStretchingWorker` 的新类（右键单击 `worker`，然后选择 `New` | `New Kotlin File/Class`）。在 `Kind` 下，选择 `Class`。

```kt
package com.example.catagenttracker.worker
import android.content.Context
import androidx.work.Data
import androidx.work.Worker
import androidx.work.WorkerParameters
class CatStretchingWorker(
    context: Context,
    workerParameters: WorkerParameters
) : Worker(context, workerParameters) {
    override fun doWork(): Result {
        val catAgentId = inputData.getString(INPUT_DATA_CAT_AGENT_ID)
        Thread.sleep(3000L)
        val outputData = Data.Builder()
            .putString(OUTPUT_DATA_CAT_AGENT_ID, catAgentId)
            .build()
        return Result.success(outputData)
    }
    companion object {
        const val INPUT_DATA_CAT_AGENT_ID = "inId"
        const val OUTPUT_DATA_CAT_AGENT_ID = "outId"
    }
}
```

将你的应用程序命名为 `Cat Agent Tracker`。

1.  运行你的应用程序：

1.  打开 `MainActivity`。在类的末尾之前，添加以下内容：

```kt
private fun getCatAgentIdInputData(catAgentIdKey: String,   catAgentIdValue: String) =
    Data.Builder().putString(catAgentIdKey, catAgentIdValue)
        .build()
```

这个辅助函数为你构造了一个带有 Cat Agent ID 的输入 `Data` 实例。

1.  将以下内容按行翻译成中文：

```kt
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    setContentView(R.layout.activity_main)
    WorkManager class to wait for an internet connection before executing work. Then, you define your Cat Agent ID. Finally, you define four requests, passing in your Worker classes, the network constraints, and the Cat Agent ID in the form of input data.
```

1.  将保存位置设置为你想要保存项目的位置。

```kt
private val workManager = WorkManager.getInstance(this)
```

1.  在你的应用程序包下创建一个新的包（右键单击 `com.example.catagenttracker`，然后选择 `New` | `Package`）。将新包命名为 `com.example.catagenttracker.worker`。

```kt
val catSuitUpRequest =   OneTimeWorkRequest.Builder(CatSuitUpWorker::class.java)
    .setConstraints(networkConstraints)
    .setInputData(
        getCatAgentIdInputData(CatSuitUpWorker           .INPUT_DATA_CAT_AGENT_ID, catAgentId)
    ).build()
WorkRequests are now enqueued to be executed in sequence when their constraints are met and the WorkManager class is ready to execute them.
```

1.  定义一个显示带有提供的消息的提示的函数。它应该看起来像这样：

```kt
private fun showResult(message: String) {
    Toast.makeText(this, message, LENGTH_SHORT).show()
}
```

1.  为了跟踪排队的 `WorkRequest` 实例的进度，在 `enqueue` 调用之后添加以下内容：

```kt
workManager.beginWith(catStretchingRequest)
    .then(catFurGroomingRequest)
    .then(catLitterBoxSittingRequest)
    .then(catSuitUpRequest)
    .enqueue()
WorkInfo observable provided by the WorkManager class for each WorkRequest. When each request is finished, a toast is shown with a relevant message.
```

1.  在类的顶部，定义你的 `WorkManager`：

![图 8.1：按顺序显示的提示现在你应该看到一个简单的 `Hello World!` 屏幕。但是，如果你等待几秒钟，你将开始看到提示信息，告诉你 SCA 准备部署到现场的进度。你会注意到这些提示信息按照你排队请求的顺序执行它们的延迟。图 8.1：按顺序显示的提示在你刚刚添加的代码下方，仍然在 `onCreate` 函数内，添加一个链式的 `enqueue` 请求：# 重复步骤 9 和 10，创建三个更多相同的工作程序，分别命名为 `CatFurGroomingWorker`、`CatLitterBoxSittingWorker` 和 `CatSuitUpWorker`。我们的 SCA 已经准备好去指定的目的地了。为了跟踪 SCA，我们将使用前台服务定期轮询 SCA 的位置，并使用新位置更新附加到该服务的粘性通知（用户无法解除的通知）。为了简单起见，我们将伪造位置。根据您在*第七章*中学到的内容，*Android 权限和 Google 地图*，您可以稍后用使用地图的真实实现替换这个实现。前台服务是执行后台操作的另一种方式。名称可能有点违反直觉。它的目的是区分这些服务与基本的 Android（后台）服务。前者与通知绑定，而后者在后台运行，没有用户界面表示。前台服务和后台服务之间的另一个重要区别是，当系统内存不足时，后者可能会被终止，而前者不会。从 Android 9（Pie，或 API 级别 28）开始，我们必须请求`FOREGROUND_SERVICE`权限来使用前台服务。由于这是一个普通权限，它将自动授予我们的应用程序。在我们启动前台服务之前，我们必须先创建一个。前台服务是 Android 抽象`Service`类的子类。如果我们不打算绑定到服务，而在我们的示例中确实不打算这样做，我们可以简单地重写`onBind(Intent)`，使其返回`null`。顺便说一句，绑定是感兴趣的客户端与服务通信的一种方式。在本书中，我们不会专注于这种方法，因为您将在下面发现其他更简单的方法。前台服务必须与通知绑定。在 Android 8（Oreo 或 API 级别 26）及更高版本中，如果前台服务在服务的`onCreate()`函数中没有与通知绑定。一个快速的实现看起来会像这样：```ktprivate fun onCreate() {    val channelId = if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {        val newChannelId = "ChannelId"        val channelName = "My Background Service"        val channel =            NotificationChannel(newChannelId, channelName,               NotificationManager.IMPORTANCE_DEFAULT)        val service = getSystemService(Context.NOTIFICATION_SERVICE) as           NotificationManager        service.createNotificationChannel(channel)        newChannelId    } else {        ""    }    val pendingIntent = Intent(this, MainActivity::class.java).let {       notificationIntent ->        PendingIntent.getActivity(this, 0, notificationIntent, 0)    }    val notification = NotificationCompat.Builder(this, channelId)        .setContentTitle("Content title")        .setContentText("Content text")        .setSmallIcon(R.drawable.notification_icon)        .setContentIntent(pendingIntent)        .setTicker("Ticker message")        .build()    startForeground(NOTIFICATION_ID, notificationBuilder.build())}```让我们来分解一下。我们首先要定义频道 ID。这仅适用于 Android Oreo 或更高版本，在早期版本的 Android 中将被忽略。在 Android Oreo 中，Google 引入了频道的概念。频道用于分组通知，并允许用户过滤掉不需要的通知：```kt    val channelId = if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {        val newChannelId = "ChannelId"        val channelName = "My Background Service"        val channel =            NotificationChannel(newChannelId, channelName,               NotificationManager.IMPORTANCE_DEFAULT)        val service = getSystemService(Context.NOTIFICATION_SERVICE) as           NotificationManager        service.createNotificationChannel(channel)        newChannelId    } else {        ""    }```接下来，我们定义`pendingIntent`。这将是用户点击通知时启动的意图。在这个例子中，主活动将被启动：```kt    val pendingIntent = Intent(this, MainActivity::class.java).let {       notificationIntent ->        PendingIntent.getActivity(this, 0, notificationIntent, 0)    }```有了频道 ID 和`pendingIntent`，我们就可以构建我们的通知。我们使用`NotificationCompat`，它可以减少对支持旧 API 级别的样板代码。我们将服务作为上下文和频道 ID 传递进去。我们定义标题、文本、小图标、意图和滚动消息，并构建通知：```kt    val notification = NotificationCompat.Builder(this, channelId)        .setContentTitle("Content title")        .setContentText("Content text")        .setSmallIcon(R.drawable.notification_icon)        .setContentIntent(pendingIntent)        .setTicker("Ticker message")        .build()```要启动一个前台服务，并将通知附加到其中，我们调用`startForeground(Int, Notification)`函数，传入一个通知 ID（任何唯一的整数值来标识此服务，不能为 0）和一个通知，其优先级必须设置为`PRIORITY_LOW`或更高。在我们的情况下，我们没有指定优先级，这将使其设置为`PRIORITY_DEFAULT`：```kt    startForeground(NOTIFICATION_ID, notificationBuilder.build())```如果启动，我们的服务现在将显示一个粘性通知。点击通知将启动我们的主活动。但是，我们的服务不会执行任何有用的操作。要为其添加一些功能，我们需要重写`onStartCommand(Intent?, Int, Int)`。当服务通过意图启动时，此函数将被调用，这也给了我们机会读取通过该意图传递的任何额外数据。它还为我们提供了标志（可能设置为`START_FLAG_REDELIVERY`或`START_FLAG_RETRY`）和一个唯一的请求 ID。我们将在本章后面读取额外的数据。在简单的实现中，您不需要担心标志或请求 ID。重要的是要注意，`onStartCommand(Intent?, Int, Int)`在 UI 线程上调用，因此不要在这里执行任何长时间运行的操作，否则您的应用程序将冻结，给用户带来不良体验。相反，我们可以使用新的`HandlerThread`（一个带有 looper 的线程，用于为线程运行消息循环的类）创建一个新的处理程序，并将我们的工作发布到其中。这意味着我们将有一个无限循环运行，等待我们通过`Handler`发布工作。当我们收到启动命令时，我们可以将要执行的工作发布到其中。然后该工作将在该线程上执行。当我们的长时间运行的工作完成时，有一些事情可能会发生。首先，我们可能希望通知感兴趣的人（例如，如果主要活动正在运行，则通知主要活动）我们已经完成。然后，我们可能希望停止在前台运行。最后，如果我们不希望再次需要服务，我们可以停止它。应用程序有几种与服务通信的方式——绑定、使用广播接收器、使用总线架构或使用结果接收器等。在我们的示例中，我们将使用 Google 的`LiveData`。在我们继续之前，值得一提的是广播接收器。广播接收器允许我们的应用程序使用类似*发布-订阅设计模式*的模式发送和接收消息。系统广播事件，例如设备启动或充电已开始。我们的服务也可以广播状态更新。例如，它们可以在完成时广播长时间的计算结果。如果我们的应用程序注册接收某个消息，系统将在广播该消息时通知它。这曾经是与服务通信的常见方式，但`LocalBroadcastManager`类现在已被弃用，因为它是一个鼓励反模式的应用程序范围事件总线。话虽如此，广播接收器仍然对系统范围的事件很有用。我们首先定义一个类，覆盖`BroadcastReceiver`抽象类：```ktclass ToastBroadcastReceiver : BroadcastReceiver() {    override fun onReceive(context: Context, intent: Intent) {        StringBuilder().apply {            append("Action: ${intent.action}\n")            append("URI: ${intent.toUri(Intent.URI_INTENT_SCHEME)}\n")            toString().let { eventText ->                Toast.makeText(context, eventText,                   Toast.LENGTH_LONG).show()            }        }    }}```当`ToastBroadcastReceiver`接收到事件时，它将显示一个显示事件操作和 URI 的 toast。我们可以通过`Manifest.xml`文件注册我们的接收器：```kt<receiver android:name=".ToastBroadcastReceiver" android:exported="true">    <intent-filter>        <action android:name=          "android.intent.action.ACTION_POWER_CONNECTED" />    </intent-filter></receiver>```指定`android:exported="true"`告诉系统此接收器可以接收来自应用程序外部的消息。操作定义了我们感兴趣的消息。我们可以指定多个操作。在此示例中，我们监听设备开始充电的情况。请记住，将此值设置为"true"允许其他应用程序，包括恶意应用程序，激活此接收器。我们也可以在代码中注册消息：```ktval filter = IntentFilter(ConnectivityManager.CONNECTIVITY_ACTION).apply {    addAction(Intent.ACTION_POWER_CONNECTED)}registerReceiver(ToastBroadcastReceiver(), filter)```将此代码添加到活动或自定义应用程序类中将注册一个新的接收器实例。只要上下文（活动或应用程序）有效，此接收器将保持存在。因此，相应地，如果活动或应用程序被销毁，我们的接收器将被释放以进行垃圾回收。现在回到我们的实现。要在我们的应用程序中使用`LiveData`，我们必须在`app/build.gradle`文件中添加一个依赖项：```ktDependencies {    ...    implementation "androidx.lifecycle:lifecycle-livedata-ktx:2.2.0"    ...}```然后我们可以在服务的伴生对象中定义一个`LiveData`实例，如下所示：```ktcompanion object {    private val mutableWorkCompletion = MutableLiveData<String>()    val workCompletion: LiveData<String> = mutableWorkCompletion}```请注意，我们将`MutableLiveData`实例隐藏在`LiveData`接口后面。这样消费者只能读取数据。现在我们可以使用`mutableWorkCompletion`实例通过为其分配一个值来报告完成。但是，我们必须记住，只能在主线程上为`LiveData`实例分配值。这意味着一旦我们的工作完成，我们必须切换回主线程。我们可以很容易地实现这一点——我们只需要一个具有主`Looper`的新处理程序（通过调用`Looper.getMainLooper()`获得），我们可以将我们的更新发布到其中。现在我们的服务已经准备好做一些工作，我们最终可以启动它。在我们这样做之前，我们必须确保将服务添加到我们的`AndroidManifest.xml`文件中的`<application></application>`块中，如下面的代码所示：```kt<application ...>    <service android:name=".ForegroundService" /></application>```要启动我们刚刚添加到清单中的服务，我们创建`Intent`，传入所需的任何额外数据，如下面的代码所示：```ktval serviceIntent = Intent(this, ForegroundService::class.java).apply {    putExtra("ExtraData", "Extra value")}```然后，我们调用`ContextCompat.startForegroundService(Context, Intent)`来触发`Intent`并启动服务。## 练习 8.02：使用前台服务跟踪您的 SCA 的工作在第一个练习中，您使用`WorkManager`类跟踪了 SCA 在准备出发时的情况。在这个练习中，您将通过显示一个粘性通知来跟踪 SCA 在部署到现场并朝着指定目标移动的情况，倒计时到达目的地的时间。这个通知将由一个前台服务驱动，它将呈现并持续更新它。随时点击通知将启动您的主活动，如果它尚未运行，它将始终将其置于前台：1.  通过更新应用的`build.gradle`文件，首先向您的项目添加`LiveData`依赖项：```kt    implementation "androidx.work:work-runtime:2.4.0"    implementation "androidx.lifecycle:lifecycle-livedata-ktx:2.2.0"    ```1.  然后，创建一个名为`RouteTrackingService`的新类，扩展抽象的`Service`类：```kt    class RouteTrackingService : Service() {        override fun onBind(intent: Intent): IBinder? = null    }    ```在这个练习中，您不会依赖绑定，因此在`onBind(Intent)`实现中简单地返回`null`是安全的。1.  在新创建的服务中，定义一些稍后需要的常量，以及用于观察进度的`LiveData`实例：```kt    companion object {        const val NOTIFICATION_ID = 0xCA7        const val EXTRA_SECRET_CAT_AGENT_ID = "scaId"        private val mutableTrackingCompletion = MutableLiveData<String>()        val trackingCompletion: LiveData<String> = mutableTrackingCompletion    }    ````NOTIFICATION_ID`必须是此服务拥有的通知的唯一标识符，不能是`0`。现在，`EXTRA_SECRET_CAT_AGENT_ID`是您用于向服务传递数据的常量。`mutableTrackingCompletion`是私有的，用于允许您通过`LiveData`在服务内部发布完成更新，而不会在服务外部暴露可变性。然后使用`trackingCompletion`以不可变的方式公开`LiveData`实例以供观察。1.  在您的`RouteTrackingService`类中添加一个函数，以提供给您的粘性通知`PendingIntent`：```kt    private fun getPendingIntent() =        PendingIntent.getActivity(this, 0, Intent(this,       MainActivity::class.java), 0)    ```这将在用户点击`Notification`时启动`MainActivity`。您调用`PendingIntent.getActivity()`，传递上下文、无请求代码（`0`）、将启动`MainActivity`的`Intent`，以及没有标志（`0`）。您会得到一个`PendingIntent`，它将启动该活动。1.  添加另一个函数来为运行 Android Oreo 或更新版本的设备创建`NotificationChannel`：```kt    @RequiresApi(Build.VERSION_CODES.O)    private fun createNotificationChannel(): String {        val channelId = "routeTracking"        val channelName = "Route Tracking"        val channel =            NotificationChannel(channelId, channelName,           NotificationManager.IMPORTANCE_DEFAULT)        val service = getSystemService(Context.NOTIFICATION_SERVICE) as       NotificationManager        service.createNotificationChannel(channel)        return channelId    }    ```首先定义频道 ID。这需要对包进行唯一标识。接下来，定义一个对用户可见的频道名称。这可以（并且应该）进行本地化。出于简单起见，我们跳过了这部分。然后创建一个`NotificationChannel`实例，将重要性设置为`IMPORTANCE_DEFAULT`。重要性决定了发布到此频道的通知有多么具有破坏性。最后，使用`Notification Service`使用`NotificationChannel`实例中提供的数据创建一个频道。该函数返回频道 ID，以便用于构造`Notification`。1.  创建一个函数来提供`Notification.Builder`：```kt    private fun getNotificationBuilder(pendingIntent: PendingIntent, channelId: String) =        NotificationCompat.Builder(this, channelId)            .setContentTitle("Agent approaching destination")            .setContentText("Agent dispatched")            .setSmallIcon(R.drawable.ic_launcher_foreground)            .setContentIntent(pendingIntent)            .setTicker("Agent dispatched, tracking movement")    ```此函数使用您之前创建的函数生成的`pendingIntent`和`channelId`实例，并构造一个`NotificationCompat.Builder`类。该构建器允许您定义标题（第一行）、文本（第二行）、要使用的小图标（根据设备而异的大小）、用户点击`Notification`时触发的意图以及一个提示（用于辅助功能；在 Android Lollipop 之前，这在通知被呈现之前显示）。您也可以设置其他属性。探索`NotificationCompat.Builder`类。在实际项目中，请记住使用来自 strings.xml 的字符串资源而不是硬编码的字符串。1.  实现以下代码，引入一个函数来启动前台服务：```kt    private fun startForegroundService(): NotificationCompat.Builder {        val pendingIntent = getPendingIntent()        val channelId =       if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {            createNotificationChannel()        } else {            ""        }        val notificationBuilder = getNotificationBuilder(pendingIntent,       channelId)        startForeground(NOTIFICATION_ID, notificationBuilder.build())        return notificationBuilder    }    ```您首先使用您之前引入的函数获取`PendingIntent`。然后，根据设备的 API 级别，您创建一个通知通道并获取其 ID，或者设置一个空 ID。您将`pendingIntent`和`channelId`传递给构造`NotificationCompat.Builder`的函数，并将服务作为前台服务启动，为其提供`NOTIFICATION_ID`和使用构建器构建的通知。该函数返回`NotificationCompat.Builder`，以便稍后用于更新通知。1.  在您的服务中定义两个字段——一个用于保存可重用的`NotificationCompat.Builder`类，另一个用于保存对`Handler`的引用，稍后您将在后台中使用它来发布工作：```kt    private lateinit var notificationBuilder: NotificationCompat.Builder    private lateinit var serviceHandler: Handler    ```1.  接下来，重写`onCreate()`以将服务作为前台服务启动，保留对`Notification.Builder`的引用，并创建`serviceHandler`：```kt    override fun onCreate() {        super.onCreate()        notificationBuilder = startForegroundService()        val handlerThread = HandlerThread("RouteTracking").apply {        start()         }        serviceHandler = Handler(handlerThread.looper)    }    ```请注意，要创建`Handler`实例，必须首先定义并启动`HandlerThread`。1.  定义一个跟踪已部署的 SCA 接近其指定目的地的调用：```kt    private fun trackToDestination(notificationBuilder:   NotificationCompat.Builder) {        for (i in 10 downTo 0) {            Thread.sleep(1000L)            notificationBuilder           .setContentText("$i seconds to destination")            startForeground(NOTIFICATION_ID,           notificationBuilder.build())        }    }    ```这将从`10`倒数到`1`，在更新之间每隔 1 秒休眠，然后使用剩余时间更新通知。1.  添加一个函数，在主线程上通知观察者完成：```kt    private fun notifyCompletion(agentId: String) {        Handler(Looper.getMainLooper()).post {            mutableTrackingCompletion.value = agentId        }    }    ```通过在主`Looper`上使用处理程序发布，您确保更新发生在主（UI）应用程序线程上。当将值设置为代理 ID 时，您正在通知所有观察者该代理 ID 已到达目的地。1.  像这样重写`onStartCommand(Intent?, Int, Int)`：```kt    override fun onStartCommand(intent: Intent?, flags: Int,   startId: Int): Int {        val returnValue = super.onStartCommand(intent, flags, startId)        val agentId =       intent?.getStringExtra(EXTRA_SECRET_CAT_AGENT_ID)            ?: throw IllegalStateException("Agent ID must be provided")        serviceHandler.post {            trackToDestination(notificationBuilder)            notifyCompletion(agentId)            stopForeground(true)            stopSelf()        }        return returnValue    }    ```您首先将调用委托给`super`，它在内部调用`onStart()`并返回一个向后兼容的状态，您可以返回。您存储此返回值。接下来，您从通过意图传递的额外参数中获取 SCA ID。如果没有提供代理 ID，则此服务将无法工作，因此如果没有提供代理 ID，您将抛出异常。接下来，您切换到在`onCreate`中定义的后台线程，以阻塞方式跟踪代理到其目的地。跟踪完成后，您通知观察者任务已完成，停止前台服务（通过传递`true`来删除通知），并停止服务本身，因为您不希望很快再次需要它。然后，您返回之前存储的`super`的返回值。1.  更新您的`AndroidManifest.xml`以请求`FOREGROUND_SERVICE`权限并引入服务：```kt    <manifest ...>        FOREGROUND_SERVICE permission. Unless we do so, the system will block our app from using foreground services. Next, we declare the service. Setting android:enabled="true" tells the system it can instantiate the service. The default is "true", so this is optional. Defining the service with android:exported="true" tells the system that other applications could start the service. In our case, we don't need this extra functionality, but we have added it just so that you are aware of this capability.    ```1.  回到您的`MainActivity`。引入一个函数来启动`RouteTrackingService`：```kt    private fun launchTrackingService() {        RouteTrackingService.trackingCompletion.observe(this, Observer {       agentId ->            showResult("Agent $agentId arrived!")        })        val serviceIntent = Intent(this,       RouteTrackingService::class.java).apply {            putExtra(EXTRA_SECRET_CAT_AGENT_ID, "007")        }        ContextCompat.startForegroundService(this, serviceIntent)    }    ```该函数首先观察`LiveData`以获取完成更新，完成时显示结果。然后，它为启动服务定义`Intent`，为该`Intent`的额外参数设置 SCA ID。然后，使用`ContextCompat`启动前台服务，该服务隐藏了与兼容性相关的逻辑。1.  最后，更新`onCreate()`以在准备好并准备好启动时立即开始跟踪 SCA：```kt    workManager.getWorkInfoByIdLiveData(catSuitUpRequest.id)        .observe(this, Observer { info ->            if (info.state.isFinished) {                showResult("Agent done suiting up. Ready to go!")                launchTrackingService()            }        })    ```1.  启动应用程序：![图 8.2：倒计时通知](img/B15216_08_02.jpg)

图 8.2：倒计时通知

在通知您 SCA 准备步骤之后，您应该在状态栏中看到一个通知。该通知然后应该从 10 倒数到 0，消失，并被一个 toast 替换，通知您代理已到达目的地。看到最后的 toast 告诉您，您成功将 SCA ID 传递给服务，并在后台任务完成时将其取回。

通过本章获得的所有知识，让我们完成以下活动。

## 活动 8.01：提醒喝水

平均每天人体失去约 2500 毫升的水（参见[`en.wikipedia.org/wiki/Fluid_balance#Output`](https://en.wikipedia.org/wiki/Fluid_balance#Output)）。为了保持健康，我们需要摄入与失去的水量相同的水。然而，由于现代生活的繁忙性质，很多人经常忘记定期补水。假设您想开发一个应用程序，跟踪您的水分流失（统计数据），并给您不断更新的液体平衡。从平衡状态开始，该应用程序将逐渐减少用户跟踪的水位。用户可以告诉应用程序他们何时喝了一杯水，它将相应地更新水位。水位的持续更新将利用您运行后台任务的知识，并且您还将利用与服务通信的知识来响应用户交互更新平衡。

以下步骤将帮助您完成此活动：

1.  创建一个空活动项目，并将您的应用命名为`My Water Tracker`。

1.  在您的`AndriodManifest.xml`文件中添加前台服务权限。

1.  创建一个新的服务。

1.  在您的服务中定义一个变量来跟踪水位。

1.  为通知 ID 和额外意图数据键定义常量。

1.  设置从服务创建通知。

1.  添加函数来启动前台服务和更新水位。

1.  将水位设置为每 5 秒减少一次。

1.  处理来自服务外部的流体添加。

1.  确保服务在销毁时清理回调和消息。

1.  在`Manifest.xml`文件中注册服务。

1.  在`MainActivity`中创建活动时启动服务。

1.  在主活动布局中添加一个按钮。

1.  当用户点击按钮时，通知服务需要增加水位。

注意

此活动的解决方案可在以下网址找到：http://packt.live/3sKj1cp

# 摘要

在本章中，我们学习了如何使用`WorkManager`和前台服务执行长时间运行的后台任务。我们讨论了如何向用户传达进度，以及如何在任务执行完成后让用户重新进入应用程序。本章涵盖的所有主题都非常广泛，您可以进一步探索与服务通信、构建通知以及使用`WorkManager`类。希望对于大多数常见情况，您现在已经拥有所需的工具。常见用例包括后台下载、清理缓存资产、在应用程序不在前台运行时播放音乐，以及结合我们从*第七章* *Android 权限和谷歌地图*中获得的知识，随时间跟踪用户的位置。

在下一章中，我们将通过编写单元测试和集成测试来使我们的应用程序更加健壮和可维护。当您编写的代码在后台运行并且当出现问题时不会立即显现时，这将特别有帮助。
