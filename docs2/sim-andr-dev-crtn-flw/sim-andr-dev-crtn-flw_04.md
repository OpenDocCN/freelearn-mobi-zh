# 第三章：*第三章*：处理协程取消和异常

在上一章中，你深入学习了 Kotlin 协程，并了解了如何使用简单的代码在 Android 中进行异步编程。你学习了如何使用协程构建器创建协程。最后，你探索了协程调度器、协程作用域、协程上下文和任务。

当协程的目的已经实现或任务已完成时，可以取消协程。你也可以根据你应用程序中的特定实例来取消它们，例如当你想要用户通过点击按钮手动停止一个任务时。协程并不总是成功的，也可能失败；开发者必须能够处理这些情况，以确保应用程序不会崩溃，并且可以通过显示 toast 或 snackbar 消息来通知用户。

在本章中，我们将首先理解协程取消。你将学习如何取消协程，并处理协程的取消和超时。然后，你将学习如何管理协程中可能发生的失败和异常。

在本章中，我们将涵盖以下主题：

+   取消协程

+   管理协程超时

+   在协程中捕获异常

到本章结束时，你将理解协程取消以及如何使你的协程可取消。你将能够在你自己的协程中添加和处理超时。你还将知道如何在协程中添加代码来捕获异常。

# 技术要求

你需要下载并安装 Android Studio 的最新版本。你可以在 [`developer.android.com/studio`](https://developer.android.com/studio) 找到最新版本。为了获得最佳的学习体验，建议使用以下配置的计算机：Intel Core i5 或更高版本，至少 4 GB RAM，以及 4 GB 可用空间。

本章的代码示例可以在 GitHub 上找到，地址为 [`github.com/PacktPublishing/Simplifying-Android-Development-with-Coroutines-and-Flows/tree/main/Chapter03`](https://github.com/PacktPublishing/Simplifying-Android-Development-with-Coroutines-and-Flows/tree/main/Chapter03)。

# 取消协程

在本节中，我们将首先探讨协程取消。开发者可以在他们的项目中手动或通过程序取消协程。你必须确保你的应用程序可以处理这些取消。

如果你的应用程序正在进行一个长时间运行的操作，耗时超过了预期，你认为它可能会导致崩溃，你可能想要停止该任务。你也可以结束那些不再必要的任务，以释放内存和资源，例如当用户离开启动任务的 activity 或关闭应用程序时。如果你在应用程序中提供了该功能，用户也可以手动停止某些操作。协程使开发者更容易取消这些任务。

如果你正在使用`ViewModel`或`lifecycleScope`从 Jetpack Lifecycle Kotlin 扩展库中，你可以轻松地创建协程而无需手动处理取消。当`ViewModel`被清除时，`viewModelScope`会自动取消，而`lifecycleScope`在生命周期被销毁时会自动取消。如果你创建了自定义的协程作用域，你必须自己添加取消操作。

在上一章中，你了解到使用如`launch`之类的协程构建器会返回一个`cancel()`函数来取消协程。以下是一个示例：

```kt
class MovieViewModel: ViewModel() {
```

```kt
    init {
```

```kt
        viewModelScope.launch {
```

```kt
            val job = launch {
```

```kt
                fetchMovies()
```

```kt
            }
```

```kt
            ...
```

```kt
            job.cancel()
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

`job.cancel()`函数将取消启动以调用`fetchMovies()`函数的协程。

在取消作业后，你可能想在继续下一个任务之前等待取消完成，以避免竞态条件。你可以通过在调用`call`函数后调用`join`函数来实现这一点：

```kt
class MovieViewModel: ViewModel() {
```

```kt
    init {
```

```kt
        viewModelScope.launch() {
```

```kt
            val job = launch {
```

```kt
                fetchMovies()
```

```kt
            }
```

```kt
            ...
```

```kt
            job.cancel()
```

```kt
            job.join()
```

```kt
            hideProgressBar()
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

在这里添加`job.join()`会使代码在执行下一个任务`hideProgressBar()`之前等待作业被取消。

你还可以使用`Job.cancelAndJoin()`扩展函数，它与调用`cancel`然后调用`join`函数相同：

```kt
class MovieViewModel: ViewModel() {
```

```kt
    init {
```

```kt
        viewModelScope.launch() {
```

```kt
            val job = launch {
```

```kt
                fetchMovies()
```

```kt
            }
```

```kt
            ...
```

```kt
            job.cancelAndJoin()
```

```kt
            hideProgressBar()
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

`cancelAndJoin`函数将调用`cancel`和`join`函数简化为单行代码。

协程作业可以有子协程作业。当你取消一个作业时，其子作业（如果有）也将被取消，递归地。

如果你的协程作用域有多个协程，并且你需要取消所有这些协程，你可以使用协程作用域中的`cancel`函数而不是逐个取消作业。这将取消作用域中的所有协程。以下是一个使用协程作用域的`cancel`函数来取消协程的示例：

```kt
class MovieViewModel: ViewModel() {
```

```kt
    private val scope = CoroutineScope(Dispatchers.Main +
```

```kt
      Job())
```

```kt
    init {
```

```kt
        scope.launch {
```

```kt
            val job1 = launch {
```

```kt
                fetchMovies()
```

```kt
            }
```

```kt
            val job2 = launch {
```

```kt
                displayLoadingText()
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
    override fun onCleared() {
```

```kt
        scope.cancel()
```

```kt
    }
```

```kt
}
```

在这个例子中，当调用`scope.cancel()`时，它将取消在协程`scope`作用域中创建的`job1`和`job2`协程。

使用协程作用域中的`cancel`函数可以更容易地取消使用指定作用域启动的多个作业。然而，在你对该作用域调用`cancel`函数之后，协程作用域将无法再启动新的协程。如果你想取消作用域的协程，但之后还想从作用域中创建协程，你可以使用`scope.coroutineContext.cancelChildren()`代替：

```kt
class MovieViewModel: ViewModel() { 
```

```kt
    private val scope = CoroutineScope(Dispatchers.Main +
```

```kt
      Job()) 
```

```kt
    init { 
```

```kt
        scope.launch() { 
```

```kt
            val job1 = launch { 
```

```kt
                fetchMovies() 
```

```kt
            } 
```

```kt
            val job2 = launch { 
```

```kt
                displayLoadingText()
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
    fun cancelAll() { 
```

```kt
        scope.coroutineContext.cancelChildren()
```

```kt
    }
```

```kt
    ...
```

```kt
}
```

调用`cancelAll`函数将取消作用域协程上下文中的所有子作业。你之后仍然可以使用该作用域来创建协程。

取消协程将抛出`CancellationException`，这是一个特殊异常，表示协程已被取消。这个异常不会使应用程序崩溃。你将在本章的后面部分学习更多关于协程和异常的内容。

你还可以将`CancellationException`的子类传递给`cancel`函数以指定不同的原因：

```kt
class MovieViewModel: ViewModel() { 
```

```kt
private lateinit var movieJob: Job
```

```kt
    init { 
```

```kt
        movieJob = scope.launch() { 
```

```kt
            fetchMovies() 
```

```kt
        }
```

```kt
    }
```

```kt
    fun stopFetching() { 
```

```kt
        movieJob.cancel(CancellationException("Cancelled by
```

```kt
          user"))
```

```kt
    }
```

```kt
    ...
```

```kt
}
```

当用户调用 `stopFetching` 函数时，此操作会使用包含消息 `Cancelled by user` 的 `CancellationException` 取消 `movieJob` 作业。

当你取消一个协程时，协程作业的状态将变为 `Cancelling`。它不会自动进入 `Cancelled` 状态并取消协程。除非你的协程有可以停止其运行的代码，否则协程可以在取消后继续运行。作业及其生命周期中的这些状态总结在下图中：

![Figure 3.1 – Coroutine job life cycle]

![Figure 3.1_B17773.jpg]

![Figure 3.1 – Coroutine job life cycle]

你的协程代码需要协作才能可取消。协程应尽可能快地处理取消操作。它必须检查协程的取消操作，如果协程已被取消，则抛出 `CancellationException`。

使你的协程可取消的一种方法是通过使用 `isActive` 来检查协程作业是否处于活动状态（仍在运行或完成）或不是。一旦协程作业的状态变为 `Cancelling`、`Cancelled` 或 `Completed`，`isActive` 的值将变为假。你可以通过以下方法使用 `isActive` 使你的协程可取消：

+   当 `isActive` 为真时执行任务。

+   只有当 `isActive` 为真时才执行任务。

+   如果 `isActive` 为假，则返回或抛出异常。

你还可以使用的另一个函数是 `Job.ensureActive()`。它将检查协程作业是否处于活动状态，如果不是，则抛出 `CancellationException`。

下面是一个示例，说明如何使用 `isActive` 使你的协程可取消：

```kt
class SensorActivity : AppCompatActivity() {
```

```kt
    private val scope = CoroutineScope(Dispatchers.IO)
```

```kt
    private lateinit var job: Job
```

```kt
   …
```

```kt
    private fun processSensorData() {
```

```kt
        job = scope.launch {
```

```kt
            if (isActive) {
```

```kt
                val data = fetchSensorData()
```

```kt
                saveData(data)
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
    fun stopProcessingData() {
```

```kt
        job.cancel()
```

```kt
    }
```

```kt
    ...
```

```kt
}
```

`processSensorData` 函数中的协程将检查作业是否处于活动状态，并且只有在 `isActive` 的值为真时才会继续执行任务。

使你的协程代码可取消的另一种方法是使用 `kotlinx.coroutines` 包中的挂起函数，例如 `yield` 或 `delay`。`yield` 函数将当前协程调度器的线程（或线程池）让给其他协程运行。

`yield` 和 `delay` 函数已经检查了取消操作，并停止了执行或抛出了 `CancellationException`。因此，当你在协程中使用它们时，不再需要手动检查取消操作。以下是一个使用前面代码片段的示例，该片段已更新为使用挂起函数 `delay` 以使协程可取消：

```kt
class SensorActivity : AppCompatActivity() {
```

```kt
    private val scope = CoroutineScope(Dispatchers.IO)
```

```kt
    private lateinit var job: Job
```

```kt
    override fun onCreate(savedInstanceState: Bundle?) {
```

```kt
        ...
```

```kt
        processSensorData()
```

```kt
    }
```

```kt
    private fun processSensorData() {
```

```kt
        job = scope.launch {
```

```kt
            delay (1_000L)
```

```kt
            val data = fetchSensorData()
```

```kt
            saveData(data)
```

```kt
        }
```

```kt
    }
```

```kt
    fun stopProcessingData() {
```

```kt
        job.cancel()
```

```kt
    }
```

```kt
    ...
```

```kt
}
```

`delay` 挂起函数将检查协程作业是否已取消，如果是，则抛出 `CancellationException`，使你的协程可取消。

在下一节中，我们将学习如何在 Android 项目中实现协程取消。

## 练习 3.01 – 在 Android 应用中取消协程

在这个练习中，你将工作于一个使用协程从 100 慢慢倒数到 0 并在 `TextView` 上显示值的程序。然后你将添加一个按钮来取消协程，在计数达到 0 之前停止倒计时：

1.  在 Android Studio 中创建一个新的项目。不要更改`MainActivity`活动的建议名称。

1.  打开`app/build.gradle`文件并添加`kotlinx-coroutines-android`的依赖项：

    ```kt
    implementation ‘org.jetbrains.kotlinx:kotlinx-
      coroutines-android:1.6.0’
    ```

这将把`kotlinx-coroutines-core`和`kotlinx-coroutines-android`库添加到你的项目中，允许你在代码中使用协程。

1.  打开`activity_main.xml`布局文件并为`TextView`添加一个`id`属性：

    ```kt
    <TextView
        android:id="@+id/textView"
        style="@style/TextAppearance.AppCompat.Large"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintRight_toRightOf="parent"
        app:layout_constraintTop_toTopOf="parent"
        tools:text="0" />
    ```

`id`属性将允许你稍后更改此`TextView`的内容。

1.  打开`MainActivity`文件。向`MainActivity`类添加以下属性：

    ```kt
    private val scope = CoroutineScope(Dispatchers.Main)
    private?var job: Job? = null
    private lateinit var textView: TextView
    private var count = 100
    ```

1.  第一行指定协程的作用域，`CoroutineScope`，使用`Dispatchers.Main`作为调度器。第二行创建一个协程作业的`job`属性。`textView`属性将用于显示倒计时文本，`count`初始化倒计时为 100。在`MainActivity`文件的`onCreate`函数中，初始化`TextView`的值：

    ```kt
    textView = findViewById(R.id.textView)
    ```

你将使用这个`textView`来更新值减少的值。

1.  创建一个倒计时函数，用于计数：

    ```kt
    private fun countdown() {
        count--
        textView.text = count.toString()
    }
    ```

这将`count`的值减 1 并在文本视图中显示它。

1.  在`onCreate`函数中，在`textView`初始化下方，添加以下代码以启动协程来计数并显示在文本视图中：

    ```kt
    job = scope.launch {
        while (count > 0) {
            delay(100)
            countdown()
        }
    } 
    ```

这将每`0.1`秒调用倒计时函数，它将倒计时并显示在文本视图中。

1.  运行应用程序。你会看到它缓慢地倒计时并显示从 100 到 0 的值，类似于以下内容：

![图 3.2 – 应用从 100 倒计时到 0

![img/Figure_3.02_B17773_new.jpg]

图 3.2 – 应用从 100 倒计时到 0

1.  打开`strings.xml`文件并为按钮添加一个字符串：

    ```kt
    <string name="stop">Stop</string>
    ```

你将使用这个作为停止倒计时的按钮文本。

1.  再次打开`activity_main.xml`文件，在`TextView`下方添加一个按钮：

    ```kt
    <Button
            android:id="@+id/button"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_marginTop="16dp"
            android:text="@string/stop"
            app:layout_constraintEnd_toEndOf="parent"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintTop_toBottomOf="@id/textView" />
    ```

这将在`TextView`下方添加一个`Button`。该按钮将用于稍后停止倒计时。

1.  打开`MainActivity`并在作业初始化后，为按钮创建一个变量：

    ```kt
    val button = findViewById<Button>(R.id.button)
    ```

当这个按钮被点击时，将允许用户停止倒计时。

1.  在下面，给按钮添加一个取消作业的点击监听器：

    ```kt
    button.setOnClickListener {
        job?.cancel()
    }
    ```

当你点击按钮时，它将取消协程。

1.  再次运行应用程序。点击**STOP**按钮并注意倒计时停止，如图所示：

![图 3.3 – 点击 STOP 按钮取消协程

![img/Figure_3.03_B17773_new.jpg]

图 3.3 – 点击 STOP 按钮取消协程

点击`job.cancel()`调用。这之所以有效，是因为协程正在使用挂起的`delay`函数，该函数会检查协程是否处于活动状态。

在这个练习中，你已经通过点击按钮添加了代码来取消 Android 应用中正在运行的协程。

可能存在一些情况，即使你取消了作业，你仍然想继续工作。为了确保即使在协程被取消的情况下任务也会完成，你可以在任务上使用`withContext(NonCancellable)`。

在本节中，你学习了如何取消协程以及如何确保你的协程代码是可取消的。你将在下一节中学习如何处理协程超时。

# 管理协程超时

在本节中，你将学习关于超时以及如何使用超时取消你的协程。设置一个固定的时间，在此之后停止运行时间超过预期的异步代码，可以帮助你节省资源，并立即通知用户任何问题。

当你的应用程序正在执行后台任务时，你可能想停止它，因为它花费了太长时间。你可以手动跟踪时间并取消任务。或者你可以使用`withTimeout`挂起函数。使用`withTimeout`函数，你可以设置以毫秒或`Duration`为单位的超时时间。一旦这个超时时间被超过，它将抛出`TimeOutCancellationException`，这是`CancellationException`的一个子类。以下是如何使用`withTimeout`的一个示例：

```kt
class MovieViewModel: ViewModel() {
```

```kt
    init {
```

```kt
        viewModelScope.launch {
```

```kt
            val job = launch {
```

```kt
                withTimeout(5_000L) {
```

```kt
                    fetchMovies()
```

```kt
                }
```

```kt
            }
```

```kt
            ...
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

为协程设置了 5,000 毫秒（5 秒）的超时。如果`fetchMovies`任务花费的时间超过这个时间，协程将超时并抛出`TimeoutCancellationException`。

你还可以使用另一个函数`withTimeoutOrNull`。它与`withTimeout`函数类似，但如果超时被超过，它将返回 null。以下是如何使用`withTimeoutOrNull`的一个示例：

```kt
class MovieViewModel: ViewModel() {
```

```kt
    init {
```

```kt
        viewModelScope.launch() {
```

```kt
            val job = async {
```

```kt
                fetchMovies()
```

```kt
            }
```

```kt
            val movies = withTimeoutOrNull(5_000L) {
```

```kt
                job.await()
```

```kt
            }
```

```kt
            ...
```

```kt
        }
```

```kt
    }
```

```kt
    ...
```

```kt
}
```

如果`fetchMovies`在 5 秒后超时，协程将返回 null，如果没有超时，它将返回获取到的电影列表。

如你在前一节中学到的，协程必须是可取消的，这样它才会在超时后被取消。在下一节中，你将学习如何处理协程的取消异常。

在本节中，你了解了协程超时以及如何设置一个时间，在此之后自动取消协程。

# 在协程中捕获异常

在本节中，你将学习关于协程异常以及如何在你的应用程序中处理它们。由于你的协程可能会失败，因此学习如何捕获异常以避免崩溃并通知用户是非常重要的。

要处理协程中的异常，你可以简单地使用`try-catch`。例如，如果你使用`launch`协程构建器启动了一个协程，你可以执行以下操作来处理异常：

```kt
class MovieViewModel: ViewModel() {
```

```kt
    init {
```

```kt
        viewModelScope.launch() {
```

```kt
            try {
```

```kt
                fetchMovies()
```

```kt
            } catch (exception: Exception) {
```

```kt
                Log.e("MovieViewModel",
```

```kt
                  exception.message.toString())
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
    ...
```

```kt
}
```

如果`fetchMovies`有异常，`ViewModel`将异常信息写入日志。

如果你使用`async`协程构建器构建了协程，当你在`Deferred`对象上调用`await`函数时，将抛出异常。处理异常的代码可能如下所示：

```kt
class MovieViewModel: ViewModel() {
```

```kt
    init {
```

```kt
        viewModelScope.launch() {
```

```kt
            val job = async {
```

```kt
                fetchMovies()
```

```kt
            }
```

```kt
            var movies = emptyList<Movie>()
```

```kt
            try {
```

```kt
                movies = job.await()
```

```kt
            } catch (exception: Exception) {
```

```kt
                Log.e("MovieViewModel",
```

```kt
                  exception.message.toString())
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
    ...
```

```kt
}
```

如果在`fetchMovies`调用运行时遇到异常，电影列表将是一个空的电影列表，并且`ViewModel`将异常消息写入日志。

当协程遇到异常时，它将取消作业并将异常传递给其父级。这个父级协程以及其子协程都将被取消。如果你使用如下`SupervisorJob`，子协程中的异常不会影响父级及其兄弟协程：

+   使用`supervisorScope{}`构建器创建协程作用域

+   为你的协程作用域使用`SupervisorJob`：`CoroutineScope(SupervisorJob())`

如果你的协程异常是`CancellationException`的子类，例如`TimeoutCancellationException`或你传递给`cancel`函数的自定义异常，异常将不会传递给父级。

在处理协程异常时，你也可以使用`CoroutineExceptionHandler`在单个位置处理这些异常。`CoroutineExceptionHandler`是你可以添加到你的协程中以处理未捕获异常的协程上下文元素。以下代码行展示了如何使用它：

```kt
class MovieViewModel: ViewModel() {
```

```kt
    private val exceptionHandler =
```

```kt
      CoroutineExceptionHandler { _, exception ->
```

```kt
        Log.e("MovieViewModel",
```

```kt
          exception.message.toString())
```

```kt
    }
```

```kt
    private val scope = CoroutineScope(exceptionHandler)
```

```kt
    ...
```

```kt
}
```

从作用域启动的协程异常将由`exceptionHandler`处理，如果它没有在任何可能出错的地方被处理，它将把异常消息写入日志。

让我们尝试添加代码来处理你的协程中的异常。

## 练习 3.02 – 在你的协程中捕获异常

在这个练习中，你将继续工作于在`TextView`上显示从 100 开始并逐渐减少到 0 的应用程序。你将添加代码来处理协程中的异常：

1.  打开你在上一个练习中构建的倒计时应用程序。

1.  前往`MainActivity`文件，并在倒计时函数的末尾添加以下代码以模拟异常：

    ```kt
    if ((0..9).random() == 0) throw Exception("An error
      occurred")
    ```

这将生成一个从 0 到 9 的随机数，如果是 0，它将抛出异常。这将模拟协程遇到异常。

1.  运行应用程序。它将开始倒计时，并在某个时刻抛出异常并崩溃应用。

1.  在你的协程中的代码周围使用`try-catch`块来捕获应用中的异常：

    ```kt
    job = scope.launch {
        try {
            while (count > 0) {
                delay(100)
                countdown()
            }
        } catch (exception: Exception) {
            //TODO
        }
    }
    ```

这将捕获倒计时函数中的异常。应用将不再崩溃，但你需要通知用户关于异常的信息。

1.  在`catch`块中，将`//TODO`替换为`Snackbar`以显示异常消息：

    ```kt
    Snackbar.make(textView, exception.message.toString(),
      Snackbar.LENGTH_LONG).show()
    ```

这将显示一个包含文本`An error occurred`的 snackbar 消息，这是异常的消息。

1.  再次运行应用程序。它将开始倒计时，但不会崩溃，而是会显示一个 snackbar 消息，如图下所示：

![图 3.4 – 当协程遇到异常时显示的 Snackbar]

![img/Figure_3.04_B17773_new.jpg]

图 3.4 – 当协程遇到异常时显示的 Snackbar

在这个练习中，你更新了你的应用程序，使其能够处理协程中的异常而不是崩溃。

在本节中，你学习了关于协程异常以及如何在你的 Android 应用中捕获它们。

# 摘要

在本章中，你学习了关于协程取消的内容。你可以通过使用协程作业中的`cancel`或`cancelAndJoin`函数，或者协程作用域中的`cancel`函数来取消协程。

你了解到协程取消需要是协作式的。你还学习了如何通过使用`isActive`检查或使用`kotlinx.coroutines`包中的挂起函数来更改你的代码，使你的协程可取消。

然后，你学习了关于协程超时的内容。你可以使用`withTimeout`或`withTimeoutOrNull`来设置超时（以毫秒或`Duration`为单位）。

你还学习了协程异常以及如何捕获它们。可以使用`try-catch`块来处理异常。你还可以在协程作用域中使用`CoroutineExceptionHandler`来在单个位置捕获和处理异常。

最后，你进行了一个练习，向协程添加取消功能，以及另一个练习来更新你的代码以处理协程异常。

在下一章中，你将深入探讨如何在你的 Android 项目中创建和运行协程的测试。
