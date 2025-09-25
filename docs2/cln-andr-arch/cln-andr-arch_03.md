# 第二章：*第二章*：深入数据源

在本章中，我们将研究一些用于在 Android 上检索和管理数据的流行库和框架，以及如何在不阻塞应用程序主线程的情况下完成这些操作。我们将首先概述在 Android 应用程序中如何处理多线程以及我们现在有哪些易于处理这项技术的技术。然后，我们将继续实现使用 Retrofit 和 OkHttp 等库从互联网加载数据，之后我们将探讨如何使用 Room 和 DataStore 等库在设备上持久化数据。

本章将涵盖以下主要内容：

+   理解 Kotlin 协程和流

+   使用 OkHttp 和 Retrofit 进行网络操作

+   使用 Room 库进行数据持久化

+   理解和使用 DataStore 库

在本章结束时，你将熟悉如何在 Android 应用程序中加载数据、管理数据和持久化数据。

# 技术要求

本章有以下硬件和软件要求：

+   Android Studio Arctic Fox 2020.3.1 Patch 3

本章的代码文件可以在此处找到：

[`github.com/PacktPublishing/Clean-Android-Architecture/tree/main/Chapter2`](https://github.com/PacktPublishing/Clean-Android-Architecture/tree/main/Chapter2)

查看以下视频以查看代码的实际应用：[`bit.ly/38uecPi`](https://bit.ly/38uecPi)

# 理解 Kotlin 协程和 Flows

在本节中，我们将探讨在 Android 生态系统中线程的工作方式以及应用程序必须做什么以确保长时间运行的操作不会阻止用户使用应用程序。然后，我们将探讨我们有哪些可用的选项来在后台执行操作，重点关注协程。最后，我们将回顾 Kotlin 流，我们可以使用它以响应式和函数式方法处理异步工作。

Android 应用程序通常在用户的设备上以单个进程运行。当操作系统启动应用程序的进程时，它将为执行进程分配内存资源。此进程启动时，将有一个执行线程在其中运行。这个线程被称为“主线程”或“**用户界面**（**UI**）线程”。在 Android 中，这个概念非常重要，因为它处理用户交互的线程。这给开发者带来了一些限制，如下所述：

+   主线程不得被长时间运行或**输入/输出**（**I/O**）操作阻塞。

+   所有 UI 更新都必须在主线程上完成。

理念是，即使应用程序在进行一些工作，用户仍然应该尽可能多地与应用程序交互。每次我们想要从或向互联网、本地存储、内容提供者等加载数据或保存数据时，我们应该使用另一个线程或使用多个线程。设备处理器处理多个线程的方式是为每个线程分配一个核心。当线程数量多于核心数量时，它将在每个线程的每条指令之间跳来跳去。同时执行太多线程最终会创建一个糟糕的**用户体验**（UX），因为处理器现在需要在主线程和其他同时执行的线程之间跳转，因此我们需要注意同时执行多少个线程。

在 Java 中，可以使用`Thread`类创建线程；然而，为每个异步操作创建一个新的线程是一个非常资源密集的操作。Java 还提供了`ThreadPool`或`Executor`的概念。这些通常管理一组固定数量的线程，这些线程将被重用于不同的操作。由于 Android 对在主线程上更新 UI 的限制，引入了`Handler`和`Looper`类，通过这些类，你可以将后台线程上执行的操作的结果提交回主线程。这里提供了一个例子：

```java
class MyClass {
    fun asyncSum(a: Int, b: Int, callback: (Int) -> Unit) {
        val handler = Handler(Looper.getMainLooper())
        Thread(Runnable {
            val result = a + b
            handler.post(Runnable {
                callback(result)
            })
        }).start()
    }
}
```

在前面的代码片段中，两个数字的求和将在一个新的线程上执行，然后使用连接到主`Looper`对象的`Handler`对象将结果发送回去，主`Looper`对象本身将循环主线程。

`Handler`和`Looper`的重复使用催生了`AsyncTask`，它提供了在后台线程上移动必要操作并在主线程上接收结果的可能性。`AsyncTask`与前面的例子工作原理相同，只是它不是为每个新操作创建一个新的线程，而是默认使用相同的线程（尽管这后来变得可配置），这意味着如果有两个`AsyncTask`实例同时执行，一个会在另一个之后等待。相同的求和操作的例子可能看起来像这样：

```java
    fun asyncSum(a: Int, b: Int, callback: (Int) -> Unit) {
        object : AsyncTask<Nothing, Nothing, Int>() {
            override fun doInBackground(vararg params: 
                Nothing?): Int {
                return a+b
            }
            override fun onPostExecute(result: Int) {
                super.onPostExecute(result)
                callback(result)
            }
        }.execute()
    }
```

在前面的例子中，求和操作是在`doInBackground`方法中完成的，这个方法在一个单独的线程上执行，而`onPostExecute`方法将在主线程上执行。

现在让我们想象一下，我们想要将这些求和操作串联起来并多次应用，如下所示：

```java
    fun asyncComplicatedSum(a: Int, b: Int, c: Int) {
        asyncSum(a, b) { tempSum ->
            asyncSum(tempSum, c) { finalSum ->
                Log.d(this.javaClass.name, "Final sum 
                    $finalSum")
            }
        }
    }
```

在前面的例子中，我们尝试将两个数字相加，并将结果加到数字`c`上。正如你所看到的，我们需要使用回调并等待`a`和`b`完成，然后对`a+b`的结果和数字`c`应用相同的函数。

让我们想象一下，当需要处理从多个数据源加载数据、合并它们、处理错误以及如果用户离开当前活动或片段则停止异步执行时，一个应用程序可能看起来像什么。RxJava 库试图通过事件驱动的方法来解决所有这些问题。它引入了可以观察、转换、与其他数据流合并并在不同线程上执行的数据流和流的概念。在 RxJava 中，两个数字的和可能看起来像这样：

```java
fun asyncSum(a: Int, b: Int): Single<Int> {
        return Single.create<Int> {
            it.onSuccess(a + b)
        }.subscribeOn(Schedulers.io())
            .observeOn(AndroidSchedulers.mainThread())
    }
```

在前面的例子中，我们创建了一个`Single`实例，这是一个只发射一个值（对于发射多个值，我们有`Flowable`和`Observable`选项）的流。发射的值是两个数字的和。使用`subscribeOn`是为了在 RxJava 内部管理的 I/O 线程上执行上游（求和），而使用`observeOn`是为了让所有下游（所有后续的命令）在主线程上获取结果。

如果我们想要链式多个求和，那么我们会有如下所示的内容：

```java
fun asyncComplicatedSum(a: Int, b: Int, c: Int) {
        val disposable = asyncSum(a, b)
            .flatMap {
                asyncSum(it, c)
            }
            .subscribe ({
                Log.d(this.javaClass.name, "Final sum $it")
            },{
                Log.d(this.javaClass.name, "Something went 
                    wrong")
            })
    }
```

在前面的例子中，执行了`a`和`b`的和，然后通过`flatMap`操作符，我们将`c`添加到这个结果中。使用`subscribe`方法是为了触发求和并监听结果。这是因为使用的`Single`实例是一个冷观察者；它只有在调用`subscribe`时才会执行。还有热观察者的概念，无论是否有订阅者都会发射。`subscribe`操作符的结果将返回一个`Disposable`实例，它提供了一个`dispose`方法，可以在我们想要停止监听流中的数据时调用。这在我们的活动和片段被销毁，我们不想更新 UI 以避免上下文泄露的情况下非常有用。

## Kotlin 协程

到目前为止，我们已经分析了围绕 Java 和 Android 框架的技术。随着 Kotlin 的采用，出现了其他处理多线程且特定于 Kotlin 的技术。其中之一是协程的概念。协程简化了我们编写异步代码的方式。我们不需要处理回调，协程引入了作用域的概念，我们可以指定代码块将在哪个线程上执行。作用域还可以连接到生命周期感知组件，帮助我们在我们生命周期感知组件终止时取消订阅异步工作的结果。让我们看看以下关于相同求和的协程示例：

```java
   suspend fun asyncSum(a: Int, b: Int): Int {
        return withContext(Dispatchers.IO) {
            a + b
        }
    }
```

在前面的示例中，`withContext` 方法将在 I/O 分发器管理的线程中执行其内部的代码块。与此分发器相关联的线程数量由 Kotlin 框架内部管理，并与设备处理器的核心数相关联。这通常意味着当多个异步操作同时执行时，我们不必担心应用程序的性能。在示例中还有另一个有趣的事情需要注意，那就是 `suspend` 关键字的使用。这是为了提醒调用此方法的调用者，它将在单独的线程上使用协程执行。

现在，让我们看看当我们想要调用此方法时，事情会是什么样子。看一下以下代码片段：

```java
class MyClass : CoroutineScope {
    override val coroutineContext: CoroutineContext
        get() = Dispatchers.Main + job
    private lateinit var job: Job
    fun asyncComplicatedSum(a: Int, b: Int, c: Int) {
        launch {
            try {
                val tempSum = asyncSum(a, b)
                val finalSum = asyncSum(tempSum, c)
                Log.d(this.javaClass.name, "Final sum 
                    $finalSum")
            } catch (e: Exception) {
                Log.d(this.javaClass.name, "Something went 
                    wrong")
            }
        }
    }
    fun create() {
        job = Job()
    }
    fun destroy() {
        job.cancel()
    }
}
```

在 `asyncComplicatedSum` 中，我们使用了 `launch` 方法。此方法与在此类中定义的 `CoroutineContext` 对象相关联。上下文是通过使用 `Main` 分发器结合与该对象的生命周期相关联的 `Job` 对象来定义的。如果在等待求和结果时调用 `destroy` 方法，则求和的执行将停止，我们将停止获取求和的结果。代码将在 I/O 线程上执行每个求和，如果作业仍然存活，则将在主线程上执行日志语句。

在 Android 中，我们已经有几个 `CoroutineScope` 对象已经定义并关联到我们的生命周期感知类。其中一个与我们相关的是为 `ViewModels` 定义的。这可以在 `org.jetbrains.kotlinx:kotlinx-coroutines-android` 库中找到，看起来可能如下所示：

```java
class MyViewModel: ViewModel() {
    init {
        viewModelScope.launch {  }
    }
}
```

`viewModelScope` 是为 `ViewModel` 实例创建的 Kotlin 扩展，如果 `ViewModel` 实例处于活动状态，则将执行。如果在 `ViewModel` 实例上调用 `onCleared`，则它将停止监听 `launch` 块中剩余要执行的代码。

在本节中，我们分析了 Kotlin 协程的工作原理以及我们如何使用它们在 Android 应用程序中处理异步操作。在下一节中，我们将创建一个 Android 应用程序，该程序将使用 Kotlin 协程进行简单的异步操作。

## 练习 02.01 – 使用 Kotlin 协程

创建一个应用程序，该程序将显示两个输入字段、一个文本字段和一个按钮。输入字段仅限于数字，当用户按下按钮时，文本字段将在 5 秒后显示两个数字的和。求和和等待将使用协程实现。

要完成练习，你需要构建以下内容：

+   一个将执行两个数字加法的类

+   一个将调用加法的 `ViewModel` 类

+   使用 Compose 的 UI 将使用以下函数：

    ```java
    @Composable
    fun Calculator(
        a: String,
        onAChanged: (String) -> Unit,
        b: String,
        onBChanged: (String) -> Unit,
        result: String,
        onButtonClick: () -> Unit
    ) {
        Column(modifier = Modifier.padding(16.dp)) {
            OutlinedTextField(
                value = a,
                onValueChange = onAChanged,
                keyboardOptions = KeyboardOptions
                    (keyboardType = KeyboardType.Number),
                label = { Text("a") }
            )
            OutlinedTextField(
                value = b,
                onValueChange = onBChanged,
                keyboardOptions = KeyboardOptions
                    (keyboardType = KeyboardType.Number),
                label = { Text("b") }
            )
            Text(text = result)
            Button(onClick = onButtonClick) {
                Text(text = "Calculate")
            }
        }
    }
    ```

按照以下步骤完成练习：

1.  在 Android Studio 中使用 **Empty Compose Activity** 创建一个新的项目。

1.  在`build.gradle`文件的顶层，定义 Compose 库版本如下：

    ```java
    buildscript {
        ext {
            compose_version = '1.0.5'
        }
        … 
    }
    ```

1.  在`app/build.gradle`文件中，我们需要添加以下依赖项：

    ```java
    dependencies {
        implementation 'androidx.core:core-ktx:1.7.0'
        implementation 'androidx.appcompat:appcompat:1.4.0'
        implementation 'com.google.android.material:material:1.4.0'
        implementation "androidx.compose.ui:ui:$compose_version"
        implementation "androidx.compose.material:material:$compose_version"
        implementation "androidx.compose.ui:ui-tooling-preview:$compose_version"
        implementation 'androidx.lifecycle:lifecycle-runtime-ktx:2.4.0'
        implementation 'androidx.activity:activity-compose:1.4.0'
        implementation 'org.jetbrains.kotlinx:kotlinx-coroutines-android:1.5.0'
        implementation "androidx.lifecycle:lifecycle-viewmodel-ktx:2.4.0"
        implementation "androidx.lifecycle:lifecycle-viewmodel-compose:2.4.0"
        testImplementation 'junit:junit:4.13.2'
        androidTestImplementation 'androidx.test.ext:junit:1.1.3'
        androidTestImplementation 'androidx.test.espresso:espresso-core:3.4.0'
        androidTestImplementation "androidx.compose.ui:ui-test-junit4:$compose_version"
        testImplementation "org.jetbrains.kotlinx:kotlinx-coroutines-test:1.5.0"
        debugImplementation "androidx.compose.ui:ui-tooling:$compose_version"
    }
    ```

1.  首先，创建一个`NumberAdder`类，并定义一个`add`操作和一个延迟，如下所示：

    ```java
    private const val DELAY = 5000
    class NumberAdder(
        private val dispatcher: CoroutineDispatcher = 
            Dispatchers.IO,
        private val delay: Int = DELAY
    ) {
        suspend fun add(a: Int, b: Int): Int {
            return withContext(dispatcher) {
                delay(delay.toLong())
                a + b
            }
        }
    }
    ```

在这个类中，我们将在执行两个数字的求和之前添加一个 5 秒的延迟。这是为了更突出异步操作。`CoroutineDispatcher`和我们要延迟的量将通过构造函数注入。这是因为我们想要对这个类进行单元测试。

1.  接下来，我们需要对这个类进行单元测试。在我们编写测试之前，创建一个测试规则，以便我们可以为协程重用它，如下所示：

    ```java
    class DispatcherTestRule : TestRule {
        @ExperimentalCoroutinesApi
        val testDispatcher = TestCoroutineDispatcher()
        @ExperimentalCoroutinesApi
        override fun apply(base: Statement?, description: 
            Description?): Statement {
            try {
                Dispatchers.setMain(testDispatcher)
                base?.evaluate()
            } catch (e: Exception) {
            } finally {
                Dispatchers.resetMain()
                testDispatcher.cleanupTestCoroutines()
            }
            return base!!
        }
    }
    ```

在这个类中，我们创建了一个`TestCoroutineDispatcher`实例，稍后将其注入到单元测试中，以便测试可以以同步方式执行求和操作。`@ExperimentalCoroutinesApi`表明`TestCoroutineDispatcher`的使用仍然处于实验状态，将来将被移至稳定版本。

1.  现在，以`NumberAdderTest`的形式编写类的单元测试，如下所示：

    ```java
    class NumberAdderTest {
        @get:Rule
        val dispatcherTestRule = DispatcherTestRule()
        @ExperimentalCoroutinesApi
        @Test
        fun testAdd() = runBlockingTest {
            val adder = NumberAdder(dispatcherTestRule.
                testDispatcher, 0)
            assertEquals(5, adder.add(1, 4))
        }
    }
    ```

在这里，我们将我们在`DispatcherTestRule`中创建的`testDispatcher`对象注入到`NumberAdder`中，然后调用`add`函数。整个测试是在一个特殊的`CoroutineScope`块`runBlockingTest`中执行的，这将确保所有启动的协程必须完成。

1.  接下来，继续创建一个`ViewModel`类，如下所示：

    ```java
    class MainViewModel(private val adder: NumberAdder = NumberAdder()) : ViewModel() {
        var resultState by mutableStateOf("0")
            private set
        fun add(a: String, b: String) {
            viewModelScope.launch {
                val result = adder.add(a.toInt(), 
                    b.toInt())
                resultState = result.toString()
            }
        }
    }
    ```

在这里，我们使用一个 Compose 状态来保留加法的结果，以及一个将触发加法操作到`viewModelScope`的方法。

1.  在创建完`ViewModel`类之后，继续创建一个活动类，如下所示：

    ```java
    class MainActivity : ComponentActivity() {
        override fun onCreate(savedInstanceState: Bundle?) {
            super.onCreate(savedInstanceState)
            setContent {
                Exercise201Theme {
                    Surface {
                        Screen()
                    }
                }
            }
        }
    }
    ```

在这里，我们使用内容初始化我们的活动。`Exercise201Theme`应替换为 Android Studio 在创建项目时生成的主题。通常，这应该在一个`Theme`文件中，并且应该是一个带有应用名称后跟`Theme`后缀的`@Composable`函数。如果不可用，可以为了练习的目的使用`MaterialTheme`。

1.  接下来，创建一个`Screen`函数，如下所示：

    ```java
    @Composable
    fun Screen(viewModel: MainViewModel = viewModel()) {
        var a by remember { mutableStateOf("") }
        var b by remember { mutableStateOf("") }
        Calculator(
            a = a,
            onAChanged = {
                a = it
            },
            b = b,
            onBChanged = {
                b = it
            },
            result = viewModel.resultState,
            onButtonClick = {
                viewModel.add(a, b)
            })
    }
    ```

在这个方法中，我们为文本字段定义变量，然后传递从 ViewModel 得到的数字相加的结果，最后调用 ViewModel 执行加法操作。

1.  最后，将练习定义中的`Calculator`函数添加到`MainActivity`文件中。

如果我们运行前面的示例，我们应该看到我们的 UI 元素，在输入数字并点击按钮后，我们将得到结果。需要注意的是，当`add`方法执行时，用户将能够与 UI 交互，并且点击不同的数字将分别在每次按钮按下后 5 秒后得到结果。

使用协程可以提高 Android 应用程序的质量，尤其是在与`ViewModel`类和生命周期感知组件的 Android 扩展结合使用时。协程简化了我们编写的异步操作代码，并且添加`suspend`关键字可以增强处理这些操作时的严谨性。

## Kotlin 流

协程为处理异步操作提供了一个很好的解决方案；然而，它们并没有像 RxJava 那样提供处理多个数据流的好能力。Flows 是协程的扩展，旨在解决这个问题。在处理流时，需要考虑三个实体，如下所述：

+   **生产者**：这个实体负责发出数据。

+   **中间者**：这个实体负责数据的转换或操作。

+   **消费者**：这个实体消费流中的数据。

让我们看看以下示例，如何使用 Kotlin 流来添加两个数字，以及它可能的样子：

```java
fun asyncSum(a: Int, b: Int): Flow<Int> {
        return flow {
            this.emit(a + b)
        }.flowOn(Dispatchers.IO)
    }
```

在这里，我们创建了一个`Flow`对象，它将在流上发出`a + b`的结果。`flowOn`方法将上游的执行移动到 I/O 线程。在这里，我们注意到与 RxJava 在`Flows`工作概念上的相似性，但我们还注意到它因为使用了`Dispatchers`而是一个协程的扩展。现在让我们看看消费者侧的流是如何看的，如下所示：

```java
class MyClass : CoroutineScope {
    override val coroutineContext: CoroutineContext
        get() = Dispatchers.Main + job
    private lateinit var job: Job
    @FlowPreview
    fun asyncComplicatedSum(a: Int, b: Int, c: Int) {
        launch {
            asyncSum(a, b)
                .flatMapConcat {
                    asyncSum(it, c)
                }
                .catch {
                    Log.d(this.javaClass.name, "Something 
                         went wrong")
                }
                .collect {
                    Log.d(this.javaClass.name, "Final sum 
                        $it")
                }
        }
    }
}
```

在这里，我们也注意到它与 RxJava 的相似之处——即当我们尝试通过`catch`方法操纵流以执行对数字`c`的加法操作时，以及当处理由于`catch`方法引起的错误时。然而，`collect`方法更接近协程，并且需要使用`CoroutineScope`或声明调用方法为挂起方法。

Flows 为特定用例提供了一些专门的类：`StateFlow`和`SharedFlow`。`StateFlow`类很有用，因为它会在订阅者订阅时提供存储的最后一个值，就像`LiveData`的工作方式一样。Flows 也可以是冷流和热流，而`SharedFlow`是热流的专门实现。如果内存中有任何消费者，`SharedFlow`将发出项目。当消费者订阅`SharedFlow`时，它也会向消费者发出存储的最后一个值，就像`StateFlow`一样。

在本节中，我们探讨了 Kotlin 流及其在处理异步操作时提供的优势。接下来，我们将通过一个简单的练习来看看如何在 Android 应用程序中使用 Kotlin 流。

## 练习 02.02 – 使用 Kotlin 流

修改应用程序，使其从*练习 02.01*开始，将两个数字的加法返回一个 Flow 而不是挂起函数。

要完成练习，你需要做以下事情：

+   将`NumberAdder`中的`add`函数重写为返回一个 Flow。

+   修改`ViewModel`调用`add`函数的方式。

按照以下步骤完成练习：

1.  将`NumberAdder`中的`add`函数修改为返回一个 Flow，如下所示：

    ```java
    private const val DELAY = 5000
    class NumberAdder(
        private val dispatcher: CoroutineDispatcher = 
            Dispatchers.IO,
        private val delay: Int = DELAY
    ) {
        suspend fun add(a: Int, b: Int): Flow<Int> {
            return flow {
                emit(a + b)
            }.onEach {
                delay(delay.toLong())
            }.flowOn(dispatcher)
        }
    }
    ```

在这里，我们创建一个新的 Flow，其中发出`a`和`b`的和，然后对流中发出的每个项目进行延迟，最后，我们指定我们希望在它上执行求和的`CoroutineDispatcher`实例。

1.  接下来，让我们修改求和的单元测试，如下所示：

    ```java
    class NumberAdderTest {
        @get:Rule
        val dispatcherTestRule = DispatcherTestRule()
        @ExperimentalCoroutinesApi
        @Test
        fun testAdd() = runBlockingTest {
            val adder = NumberAdder
                (dispatcherTestRule.testDispatcher, 0)
            val result = adder.add(1, 4).first()
            assertEquals(5, result)
        }
    }
    ```

由于`add`方法返回一个`Flow`对象，我们现在必须找到流中发出的第一个项目，并将该项目的值与我们的预期结果进行断言。

1.  修改`MainViewModel`类以消费`add`操作，如下所示：

    ```java
    class MainViewModel(private val adder: NumberAdder = NumberAdder()) : ViewModel() {
        var resultState by mutableStateOf("0")
            private set
        fun add(a: String, b: String) {
            viewModelScope.launch {
                adder.add(a.toInt(), b.toInt())
                    .collect {
                        resultState = it.toString()
                    }
            }
        }
    }
    ```

在这里，`add`方法仍然将使用相同的`CoroutineScope`实例来启动`add`方法，该方法现在将使用`collect`方法来获取求和的结果。

如果我们按照练习中的步骤启动应用程序，其行为将与*练习 02.01*相同，我们可以看到 Kotlin 流如何通过引入来自 RxJava 的概念来扩展协程的功能，从而简化我们处理多个数据流的方式。

在本节中，我们看到了异步操作是如何随着时间的推移而演变的，以及我们的应用程序从诸如协程和流等概念中获得了多少好处，这些概念提供了对后台线程的管理，简化了执行异步操作的方式，管理多个数据流，并且可以连接到 Android 组件的生命周期。在下一节中，我们将探讨我们可以用来从网络获取数据的工具，以及它们如何与 Kotlin 协程和流集成。

# 使用 OkHttp 和 Retrofit 进行网络操作

在本节中，我们将探讨如何使用 Retrofit 库执行网络操作以及它提供的优势。

许多安卓应用程序需要互联网来访问存储在各种服务器上的数据。通常，这通过`HttpURLConnection`或 Apache HttpClient 来实现。使用这些组件中的任何一个意味着开发者需要手动处理从**普通的 Java 对象**（**POJOs**）到 JSON 的转换，处理各种网络配置，以及处理向后兼容性问题。

OkHttp 库将通过`OkHttpClient`类来解决这些问题，该类将处理各种网络配置并提供其他功能，如缓存。Retrofit 库可以放在 OkHttp 库之上，旨在确保处理各种数据格式时的类型安全。它非常可配置，并允许插入各种转换库以进行 POJO 到 JSON 的转换或**可扩展标记语言**（**XML**）或其他类型的格式。

为了将 Retrofit 和 OkHttp 添加到项目中，我们将向`build.gradle`文件中添加以下依赖项：

```java
dependencies {
    …
    implementation "com.squareup.okhttp3:okhttp:4.9.0"
    implementation "com.squareup.retrofit2:retrofit:2.9.0"
    …
}
```

接下来，我们需要确定需要使用哪些转换器来处理数据。由于 JSON 是一种常见的格式，我们将使用 JSON 转换器和 Moshi 库来完成此操作，因此我们需要添加这两个库的依赖项，如下所示：

```java
dependencies {
    …
    implementation "com.squareup.okhttp3:okhttp:4.9.0"
    implementation "com.squareup.retrofit2:retrofit:2.9.0"
    implementation "com.squareup.retrofit2:converter-moshi:2.9.0"
    implementation "com.squareup.moshi:moshi:1.13.0"
    …
}
```

在这里，Moshi 库将负责将 POJO 转换为 JSON，转换库将连接到 Retrofit 库，并在 Android 应用程序和服务器之间交换数据时触发此转换。

假设我们需要从服务器以 JSON 格式获取数据。我们可以使用[`jsonplaceholder.typicode.com/`](https://jsonplaceholder.typicode.com/)服务作为示例。如果我们想获取用户列表，我们可以使用[`jsonplaceholder.typicode.com/users`](https://jsonplaceholder.typicode.com/users) **统一资源定位符**（**URL**）。一个用户的 JSON 表示如下：

```java
{
"id": 1,
    "name": "Leanne Graham",
    "username": "Bret",
    "email": "Sincere@april.biz",
    "address": {
      "street": "Kulas Light",
      "suite": "Apt. 556",
      "city": "Gwenborough",
      "zipcode": "92998-3874",
      "geo": {
        "lat": "-37.3159",
        "lng": "81.1496"
      }
    },
    "phone": "1-770-736-8031 x56442",
    "website": "hildegard.org",
    "company": {
      "name": "Romaguera-Crona",
      "catchPhrase": "Multi-layered client-server neural-
          net",
      "bs": "harness real-time e-markets"
    }
```

我们可以在 JSON 表示中看到，用户有一个`id`，一个`username`，一个`email`值等等。在 Kotlin 中，我们可以创建一个表示，并且可以排除应用程序不需要的属性，例如`email`、`address`、`phone`、`website`和`company`，如下所示：

```java
    data class User(
        @Json(name = "id") val id: Long,
        @Json(name = "name") val name: String,
        @Json(name = "username") val username: String
    )
```

在这里，我们使用 Moshi 将 JSON 属性映射到 Kotlin 类型，并且只保留了初始 JSON 中存在的三个字段。现在，让我们看看我们如何初始化我们的网络库。完成此操作的代码如下所示：

```java
  fun createOkHttpClient() =  OkHttpClient
        .Builder()
        .readTimeout(15, TimeUnit.SECONDS)
        .connectTimeout(15, TimeUnit.SECONDS)
        .build()
```

对于 OkHttp，我们使用`Builder`方法创建一个新的`OkHttpClient`实例，并且我们可以为其提供某些配置。现在，我们将使用之前创建的`OkHttpClient`实例来创建一个`Retrofit`实例，如下所示：

```java
fun createRetrofit(
        okHttpClient: OkHttpClient
    ): Retrofit {
        return Retrofit.Builder()
            .baseUrl("https://jsonplaceholder.typicode.com/")
            .client(okHttpClient)
            .build()
    }
```

在这里，我们创建了一个新的`Retrofit`实例，其基本 URL 设置为[`jsonplaceholder.typicode.com/`](https://jsonplaceholder.typicode.com/)。在开发过程中更改基本 URL 非常有用。许多团队将有一个内部使用的开发 URL，用于测试功能和集成，并将有一个生产 URL，其中设置了实际的用户数据。现在，我们需要将 Moshi JSON 序列化连接到`Retrofit`实例，如下所示：

```java
Fun createConverterFactory(): MoshiConverterFactory = MoshiConverterFactory.create()
```

在这里，我们创建`MoshiConverterFactory`，这是一个 Retrofit 转换器，旨在将`Retrofit`连接到 Moshi 执行的 JSON 序列化。现在，我们需要将我们的`Retrofit`初始化更改为以下内容：

```java
fun createRetrofit(
        okHttpClient: OkHttpClient,
        gsonConverterFactory: MoshiConverterFactory
    ): Retrofit {
        return Retrofit.Builder()
.baseUrl("https://jsonplaceholder.typicode.com/")
            .client(okHttpClient)
            .addConverterFactory(gsonConverterFactory)
            .build()
    }
```

在这里，我们将`MoshiConverterFactory`转换器添加到 Retrofit 的`Builder`方法中，以允许这两个组件协同工作。最后，我们可以创建一个 Retrofit 接口，其中包含 HTTP 请求的模板，如下所示：

```java
interface UserService {
        @GET("/users")
        fun getUsers(): Call<List<User>>
        @GET("/users/{userId}")
        fun getUser(@Path("userId") userId: Int): 
            Call<User>
        @POST("/users")
        fun createUser(@Body user: User): Call<User>
        @PUT("/users/{userId}")
        fun updateUser(@Path("userId") userId: Int, @Body 
            user: User): Call<User>   
    }
```

此接口包含在服务器上获取、创建、更新和删除数据的各种方法的示例。请注意，这些方法的返回类型是`Call`对象，它提供了执行 HTTP 请求同步或异步的能力。使 Retrofit 对开发者更具吸引力的事情之一是它可以与其他异步库（如 RxJava 和协程）集成。将前面的示例转换为协程将看起来像这样：

```java
interface UserService {
        @GET("/users")
        suspend fun getUsers(): List<User>
        @GET("/users/{userId}")
        suspend fun getUser(@Path("userId") userId: Int): 
            User
        @POST("/users")
        suspend fun createUser(@Body user: User): User
        @PUT("/users/{userId}")
        suspend fun updateUser(@Path("userId") userId: Int, 
            @Body user: User): User
    }
```

在前面的示例中，我们为每个方法添加了 `suspend` 关键字，并移除了对 `Call` 类的依赖。这允许我们使用协程执行这些方法。要创建此类的实例，我们需要执行以下操作：

```java
fun createUserService(retrofit: Retrofit) = retrofit.create(UserService::class.java)
```

在这里，我们使用之前创建的 `Retrofit` 实例来创建一个新的 `UserService` 实例。

在本节中，我们分析了如何使用 OkHttp 和 Retrofit 从互联网加载数据以及这些库提供的优势，特别是当与 Kotlin 协程和流结合使用时。在下一节中，我们将创建一个 Android 应用程序，该应用程序将使用这些库从 UI 中获取和显示数据。

## 练习 02.03 – 使用 OkHttp 和 Retrofit

创建一个连接到 [`jsonplaceholder.typicode.com/`](https://jsonplaceholder.typicode.com/) 并使用 OkHttp、Retrofit 和 Moshi 显示用户列表的 Android 应用程序。对于每个用户，我们将显示姓名、用户名和电子邮件。

要完成练习，你需要执行以下操作：

+   创建一个将映射用户 JSON 表示的 `User` 数据类。

+   创建一个具有获取用户列表方法 `UserService` 类。

+   创建一个将使用 `UserService` 获取用户列表的 `ViewModel` 类。

+   实现一个将显示用户列表的 `Activity` 类。

将使用以下方法创建一个 UI 列表：

```java
@Composable
fun UserList(users: List<User>) {
    LazyColumn(modifier = Modifier.padding(16.dp)) {
        items(users) {
            Column(modifier = Modifier.padding(16.dp)) {
                Text(text = it.name)
                Text(text = it.username)
                Text(text = it.email)
            }
        }
    }
}
```

按照以下步骤完成练习：

1.  创建一个具有 **Empty Compose Activity** 的 Android 应用程序。

1.  在 `build.gradle` 文件的顶层，定义 Compose 库版本，如下所示：

    ```java
    buildscript {
        ext {
            compose_version = '1.0.5'
        }
        … 
    }
    ```

1.  在 `app/build.gradle` 文件中，添加以下依赖项：

    ```java
    dependencies {
        implementation 'androidx.core:core-ktx:1.7.0'
        implementation 'androidx.appcompat:appcompat:1.4.0'
        implementation 'com.google.android.material:material:1.4.0'
        implementation "androidx.compose.ui:ui:$compose_version"
        implementation "androidx.compose.material:material:$compose_version"
        implementation "androidx.compose.ui:ui-tooling-preview:$compose_version"
        implementation 'androidx.lifecycle:lifecycle-runtime-ktx:2.4.0'
        implementation 'androidx.activity:activity-compose:1.4.0'
        implementation 'org.jetbrains.kotlinx:kotlinx-coroutines-android:1.5.0'
        implementation "androidx.lifecycle:lifecycle-viewmodel-ktx:2.4.0"
        implementation "androidx.lifecycle:lifecycle-viewmodel-compose:2.4.0"
        implementation "com.squareup.okhttp3:okhttp:4.9.0"
        implementation "com.squareup.retrofit2:retrofit:2.9.0"
        implementation "com.squareup.retrofit2:converter-moshi:2.9.0"
        implementation "com.squareup.moshi:moshi:1.13.0"
        implementation "com.squareup.moshi:moshi-kotlin:1.13.0"
        testImplementation 'junit:junit:4.13.2'
        androidTestImplementation 'androidx.test.ext:junit:1.1.3'
        androidTestImplementation 'androidx.test.espresso:espresso-core:3.4.0'
        androidTestImplementation "androidx.compose.ui:ui-test-junit4:$compose_version"
        testImplementation "org.jetbrains.kotlinx:kotlinx-coroutines-test:1.5.0"
        debugImplementation "androidx.compose.ui:ui-tooling:$compose_version"
    }
    ```

1.  现在，将互联网访问权限添加到 `AndroidManifest.xml` 文件中，如下所示：

    ```java
    <uses-permission android:name="android.permission.INTERNET"/>
    ```

1.  现在继续并创建一个将包含用户信息的类，如下所示：

    ```java
    @JsonClass(generateAdapter = true)
    data class User(
        @Json(name = "id") val id: Long,
        @Json(name = "name") val name: String,
        @Json(name = "username") val username: String,
        @Json(name = "email") val email: String
    )
    ```

在这里，我们将保存 `id` 字段，这通常是一个用于区分不同用户和我们需要显示的字段的关联字段。

1.  接下来，创建一个将获取用户数据的 `UserService` 类，如下所示：

    ```java
    interface UserService {
        @GET("/users")
        suspend fun getUsers(): List<User>
    }
    ```

在这里，我们只有一个方法，它将从 `/users` 路径获取用户列表。

1.  现在，我们初始化网络对象。因为我们没有使用任何 `MainApplication` 类，如下所示：

    ```java
    class MyApplication : Application() {
        companion object {
            lateinit var userService: UserService
        }
        override fun onCreate() {
            super.onCreate()
            val okHttpClient = OkHttpClient
                .Builder()
                .readTimeout(15, TimeUnit.SECONDS)
                .connectTimeout(15, TimeUnit.SECONDS)
                .build()
            val moshi = Moshi.Builder().
                add(KotlinJsonAdapterFactory()).build()
            val retrofit = Retrofit.Builder()
                .baseUrl("https://jsonplaceholder.typicode.com/")
                .client(okHttpClient)
                .addConverterFactory(MoshiConverterFactory.create(moshi))
                .build()
            userService = retrofit.create(UserService::class.java)
        }
    }
    ```

在这里，我们正在初始化我们的网络库和 `UserService` 对象。目前，我们持有对这个对象的静态引用，这在一般情况下不是一个好主意。通常，我们会依赖 DI 框架来管理这些网络依赖项。

1.  在 `AndroidManifest.xml` 文件中，添加以下代码：

    ```java
      <application
            …
            android:name=".MyApplication"
            …>
    ```

由于我们正在继承 `Application` 类，我们需要将此类添加到清单中。

1.  接下来，继续创建一个 `MainViewModel` 类，如下所示：

    ```java
    class MainViewModel(private val userService: 
        UserService) : ViewModel() {
        var resultState by mutableStateOf
            <List<User>>(emptyList())
            private set
        init {
            viewModelScope.launch {
                    val users = userService.getUsers()
                    resultState = users
            }
        }
    }
    class MainViewModelFactory : ViewModelProvider.Factory {
        override fun <T : ViewModel> create(modelClass: 
            Class<T>): T =
            MainViewModel(MyApplication.userService) as T
    }
    ```

`MainViewModel` 类将依赖于 `UserService` 类来获取 `Users` 列表并将它们存储在用于 UI 的 Compose 状态中。在这里，我们还在创建一个 `MainViewModelFactory` 类，该类将负责将 `UserService` 类注入到 `MainViewModel` 类中。

1.  现在，我们继续创建一个 `MainActivity` 类，如下所示：

    ```java
    class MainActivity : ComponentActivity() {
        override fun onCreate(savedInstanceState: Bundle?) {
            super.onCreate(savedInstanceState)
            setContent {
                Exercise0203Theme {
                    Surface {
                        Screen()
                    }
                }
            }
        }
    }
    ```

在这里，我们使用内容初始化我们的活动。应将 `Exercise203Theme` 主题替换为 Android Studio 在创建项目时生成的主题。通常，这应该在一个 `Theme` 文件中，并且应该是一个 `@Composable` 函数，该函数以应用程序名称结尾并带有 `Theme` 后缀。如果不可用，可以为了练习的目的使用 `MaterialTheme`。

1.  创建一个 `Screen` 方法，我们将从 `MainViewModel` 类中获取用户列表并绘制一个项目列表，如下所示：

    ```java
    @Composable
    fun Screen(viewModel: MainViewModel = viewModel
        (factory = MainViewModelFactory())) {
        UserList(users = viewModel.resultState)
    }
    ```

1.  最后，将练习定义中的 `UserList` 函数添加到 `MainActivity` 文件中。

如果我们按照练习中的步骤启动应用程序，如果设备有互联网访问，我们应该能够看到用户列表正在加载。

在本节中，我们看到了如何在 Android 应用程序中通常从互联网检索数据。我们探讨了 OkHttp 和 Retrofit 等库，并看到了如何以类型安全的方式直接进行 HTTP 调用，而无需手动将 JSON 文件转换为数据类。我们还观察了这些库的潜力，因为它们与异步技术如 RxJava 和协程的集成。在下一节中，我们将探讨用于持久化数据的库以及如何将它们与网络库以及协程和流集成。

# 使用 Room 库进行数据持久化

在本节中，我们将讨论如何在 Android 应用程序中持久化数据，以及我们如何使用 Room 库来实现这一点。

Android 提供了许多在 Android 设备上持久化数据的方法，大多数涉及文件。其中一些文件采用了专门的数据持久化方法。其中一种方法是以 SQLite 的形式。SQLite 是一种特殊类型的文件，可以使用 **结构化查询语言**（**SQL**）查询在其中存储结构化数据，就像 MySQL 和 Oracle 等其他类型的数据库一样。

在过去，如果开发人员想在 SQLite 中持久化数据，他们需要手动定义表、编写查询，并将包含此数据的对象转换为执行 **创建、读取、更新和删除**（**CRUD**）操作的正确格式。这类工作涉及大量样板代码，容易出错。Room 通过在 SQLite 操作之上提供抽象层来解决这个问题。

为了将 Room 添加到应用程序中，我们将在 `build.gradle` 中添加以下库：

```java
dependencies {
    …
    implementation "androidx.room:room-runtime:2.4.0"
    kapt "androidx.room:room-compiler:2.4.0"
    …
}
```

使用 `kapt` 的原因是 Room 使用会生成与 SQLite 层交互所需代码的注解。为了使用 `kapt` 功能，我们需要将插件添加到 `build.gradle` 文件中，如下所示：

```java
plugins {
    …
    id 'kotlin-kapt'
}
```

这将允许构建系统分析项目中需要代码生成的注解，并根据提供的注解生成必要的类。

我们想要存储的数据被 `@Entity` 注解标记，如下面的代码片段所示：

```java
@Entity(tableName = "user")class UserEntity(
    @PrimaryKey @ColumnInfo(name = "id") val id: Long,
    @ColumnInfo(name = "name") val name: String,
    @ColumnInfo(name = "username") val username: String
)
```

在这里，我们定义了一个名为 `UserEntity` 的 Room 实体，它将代表一个名为 `user` 的表，并且 `@ColumnInfo` 注解用于指定列在数据库中的名称。

一组典型的 CRUD 操作可能看起来像这样：

```java
@Dao
interface UserDao {
    @Query("SELECT * FROM user")
    fun getAll(): List<UserEntity>
    @Query("SELECT * FROM user WHERE id IN (:userIds)")
    fun loadAllByIds(userIds: IntArray): List<UserEntity>
    @Insert
    fun insert(vararg users: User)
    @Update
    fun update(vararg users: User)
    @Delete
    fun delete(user: User)
}
```

正如我们在 Retrofit 中定义了一个用于与服务器通信的服务接口一样，我们也为 Room 定义了一个类似的接口，并使用 `@Dao` 注解，用于 **数据访问对象**（**DAO**）。在这个例子中，我们定义了一系列函数，用于获取存储在表中的所有用户，查找用户，插入新用户，更新用户和删除用户。

与 Retrofit 一样，Room 也提供了与协程的集成，如下面的代码片段所示：

```java
@Dao
interface UserDao {
    @Query("SELECT * FROM user")
    suspend fun getAll(): List<UserEntity>
    @Query("SELECT * FROM user WHERE id IN (:userIds)")
    suspend fun loadAllByIds(userIds: IntArray): 
        List<UserEntity>
    @Insert
    suspend fun insert(vararg users: User)
    @Update
    suspend fun update(vararg users: User)
    @Delete
    suspend fun delete(user: User)
}
```

在前面的示例中，我们添加了 `suspend` 关键字，这使得 Room 库易于集成并在协程中执行。

在协程之上，Room 库还可以与 Kotlin flows 集成。这对于每次特定表发生变化时都会发出事件的查询非常有用。这种集成看起来可能如下所示：

```java
@Dao
interface UserDao {
    @Query("SELECT * FROM user")
    fun getAll(): Flow<List<UserEntity>>
    @Query("SELECT * FROM user WHERE id IN (:userIds)")
    fun loadAllByIds(userIds: IntArray): 
        Flow<List<UserEntity>>
}
```

在前面的示例中，我们将 `@Query` 函数更改为返回一个 `Flow` 对象。如果用户表发生变化，则查询将被重新触发，并发出新的用户列表。

我们现在需要设置数据库，如下所示：

```java
@Database(entities = [UserEntity::class], version = 1)
abstract class AppDatabase : RoomDatabase() {
    abstract fun userDao(): UserDao
}
```

在前面的代码片段中，我们定义了一个新的类，该类从 `RoomDatabase` 类扩展，并使用 `@Database` 注解声明我们的实体和当前版本。这个版本用于跟踪在应用程序新版本发布之间数据库结构变化时的迁移。

为了初始化数据库，我们需要执行以下代码：

```java
val db = Room.databaseBuilder(
            applicationContext,
            AppDatabase::class.java, "name"
        ).build()
```

这将创建我们的 SQLite 数据库，并返回一个 `AppDatabase` 实例，我们可以通过它访问我们定义的 DAO 对象并调用它们的方法来处理数据。

在本节中，我们探讨了如何使用 Room 持久化数据以及如何将其与协程和 flows 集成。在下一节中，我们将创建一个 Android 应用程序，该应用程序将使用 Room 持久化数据，并探讨如何将其与 Retrofit 和 OkHttp 集成。

## 练习 02.04 – 使用 Room 持久化数据

将 Room 集成到 *练习 02.03* 中，以便当用户从 Retrofit 加载时，它们将被存储在数据库中，然后显示在 UI 上。

要完成练习，你需要做以下事情：

1.  创建一个`UserEntity`类，它将成为 Room 实体。

1.  创建一个`UserDao`类，其中将包含插入用户和查询所有用户作为流的函数。

1.  创建一个将代表应用程序数据库的`AppDatabase`类。

1.  修改`MainViewModel`类，从`UserService`类获取用户，然后将其插入到`UserDao`类中。

1.  修改`MainActivity`类，使用`UserEntity`对象列表而不是`User`对象。

按照以下步骤完成练习：

1.  将`kapt`插件添加到`app/build.gradle`文件中，如下所示：

    ```java
    plugins {
        …
        id 'kotlin-kapt'
    }
    ```

1.  将 Room 依赖项添加到`app/build.gradle`中，如下所示：

    ```java
    dependencies {
        … 
        implementation "androidx.room:room-runtime:2.4.0"
        implementation "androidx.room:room-ktx:2.4.0"
        kapt "androidx.room:room-compiler:2.4.0"
        …
    }
    ```

1.  创建一个`UserEntity`类，如下所示：

    ```java
    @Entity(tableName = "user")
    class UserEntity(
        @PrimaryKey @ColumnInfo(name = "id") val id: Long,
        @ColumnInfo(name = "name") val name: String,
        @ColumnInfo(name = "username") val username:
            String,
        @ColumnInfo(name = "email") val email: String
    )
    ```

`UserEntity`类与`User`类具有相同的字段，并且包含 Room 注解，用于表名和每列的名称。

1.  接下来，创建一个`UserDao`类，如下所示：

    ```java
    @Dao
    interface UserDao {
        @Query("SELECT * FROM user")
        fun getUsers(): Flow<List<UserEntity>>
        @Insert(onConflict = OnConflictStrategy.REPLACE)
        fun insertUsers(users: List<UserEntity>)
    }
    ```

在这里，我们使用 flows 返回用户列表，并使用`OnConflictStrategy.REPLACE`选项，如果插入相同用户多次，则将其替换为将要插入的那个。其他选项包括`OnConflictStrategy.ABORT`，如果发生冲突，将丢弃整个事务，或者`OnConflictStrategy.IGNORE`，如果发生冲突，将跳过插入行。

1.  现在，继续创建一个`AppDatabase`类，如下所示：

    ```java
    @Database(entities = [UserEntity::class], version = 1)
    abstract class AppDatabase : RoomDatabase() {
        abstract fun userDao(): UserDao
    } 
    ```

在`AppDatabase`中，我们提供`UserDao`类以供访问，并使用`UserEntity`类作为用户表。

1.  接下来，我们需要初始化`AppDatabase`对象，如下所示：

    ```java
    class MyApplication : Application() {
        companion object {
            …
            lateinit var userDao: UserDao 
            …
        }
        override fun onCreate() {
            super.onCreate()
            …
            val db = Room.databaseBuilder(
                applicationContext,
                AppDatabase::class.java, "my-database"
            ).build()
            userDao = db.userDao()
            …
        }
    } 
    ```

在这里，我们遇到了与 Retrofit 相同的问题，因此我们将采取相同的方法并使用`Application`类。就像 Retrofit 一样，一个依赖注入框架将帮助我们解决这个问题。

1.  现在，让我们将 Room 集成到`MainViewModel`类中，如下所示：

    ```java
    class MainViewModel(
        private val userService: UserService,
        private val userDao: UserDao
    ) : ViewModel() {
        var resultState by mutableStateOf<List<UserEntity>>(emptyList())
            private set
        init {
            viewModelScope.launch {
                flow { emit(userService.getUsers()) }
                    .onEach {
                    val userEntities =
                        it.map { user -> UserEntity
    (user.id, user.name, 
                             user.username, user.email) }
                    userDao.insertUsers(userEntities)
                }.flatMapConcat { userDao.getUsers() }
                    .catch { emitAll(userDao.getUsers()) }
                    .flowOn(Dispatchers.IO)
                    .collect {
                        resultState = it
                    }
            }
        } 
    }
    class MainViewModelFactory : ViewModelProvider.Factory {
        override fun <T : ViewModel> create(modelClass: Class<T>): T =
            MainViewModel(MyApplication.userService, MyApplication.userDao) as T
    } 
    ```

`MainViewModel`类现在有一个新的依赖项`UserDao`类。在`init`块中，我们现在创建一个流，其中发出从 Retrofit 获取的用户列表，然后将其转换为`UserEntity`并插入到数据库中。之后，我们将查询`UserEntities`实例，并以流的形式返回它们，这将作为结果。如果发生错误，我们将返回当前存储的用户。

1.  最后，更新`MainActivity`类中用户的类型，如下所示：

    ```java
    class MainActivity : ComponentActivity() {
    …
    @Composable
    fun UserList(users: List<UserEntity>) {
    …
        }
    } 
    ```

在这里，我们只是将依赖项更改为现在依赖于`UserEntity`类。

如果我们按照练习中的步骤运行应用程序，我们将看到与*练习 02.03*相同的输出。然而，如果我们关闭应用程序，在设备上开启飞行模式，然后重新打开应用程序，我们仍然会看到之前显示的信息。

在本节中，我们分析了如何在设备上持久化结构化数据，并使用了 Room 库来实现这一点。我们还观察了 Room 与其他库（如 Retrofit 和流）之间的交互，以及我们如何使用流以非常直接的方式结合 Room 和 Retrofit 的数据流。在下一节中，我们将探讨如何以键值对的形式持久化简单数据。

# 理解和使用 DataStore 库

在本节中，我们将讨论如何持久化数据的键值对，以及如何使用 DataStore 库来实现这一点。在 Android 中，我们有在键值对中持久化基本类型和字符串的可能性。在过去，这是通过 `SharedPreferences` 类来完成的，它是 Android 框架的一部分。键和值最终会被保存在设备上的一个 XML 文件中。由于这涉及到 I/O 操作，随着时间的推移，它演变为提供异步保存数据的能力，并保持内存缓存以快速访问数据。然而，这存在一些不一致性，尤其是在初始化 `SharedPreferences` 对象时。DataStore 的设计是为了解决这些问题，因为它与协程和流集成。

要将 DataStore 添加到项目中，我们需要以下依赖项：

```java
dependencies {
    …
    implementation "androidx.datastore:datastore-preferences:1.0.0"
    …
} 
```

使用 DataStore 的样子如下：

```java
private val KEY_TEXT = stringPreferencesKey("key_text")
class AppDataStore(private val dataStore: 
    DataStore<Preferences>) {
    val savedText: Flow<String> = dataStore.data
        .map { preferences ->
            preferences[KEY_TEXT].orEmpty()
        }
    suspend fun saveText(text: String) {
        dataStore.edit { preferences ->
            preferences[KEY_TEXT] = text
        }
    }
}
```

`KEY_TEXT` 字段将代表一个用于存储文本的键。`DataStore<Preferences>` 负责获取和将数据写入 `SharedPreferences`。`savedText` 字段将监控首选项的变化，并为每个变化在 `Flow` 对象中发出一个新值。要异步写入数据，我们需要编辑当前的数据存储并设置与键关联的值。

要初始化 DataStore 库，我们需要将以下内容声明为顶级声明：

```java
val Context.dataStore: DataStore<Preferences> by preferencesDataStore(name = "my_preferences")
```

这将使我们能够访问应用程序其余部分的 DataStore 库。

当我们想要初始化 `AppDataStore` 时，我们可以使用以下代码：

```java
val appDataStore = AppDataStore(dataStore) 
```

这允许我们包装 `DataStore` 类，避免将依赖项暴露给应用程序的其他地方。

在本节中，我们探讨了如何以键值对的形式持久化数据，以及如何使用 DataStore 库来实现这一点。在下一节中，我们将创建一个 Android 应用程序，该程序将使用 DataStore 并将其与 Kotlin 流和协程集成。

## 练习 02.05 – 使用 DataStore 持久化数据

修改 *练习 02.04* 并引入 DataStore 库，该库将持久化获取用户时执行的请求数量，并在项目列表上方显示此数量。

要完成练习，你需要做以下事情：

+   创建一个名为 `AppDataStore` 的类，该类将管理与 DataStore 库的交互。

+   修改 `MainViewModel` 类，以便注入并使用 `AppDataStore` 依赖项来检索当前请求数量并增加请求数量。

+   修改`MainActivity`类，添加一个新的`Text`对象，用于显示请求的计数。

按照以下步骤完成练习：

1.  将以下依赖项添加到`app/build.gradle`文件中：

    ```java
    dependencies {
        …
        implementation "androidx.datastore:datastore-
            preferences:1.0.0"
        …
    }
    ```

1.  创建一个`AppDataStore`类，如下所示：

    ```java
    private val KEY_COUNT = intPreferencesKey("key_count")
    class AppDataStore(private val dataStore: 
        DataStore<Preferences>) {
        val savedCount: Flow<Int> = dataStore.data
            .map { preferences ->
                preferences[KEY_COUNT] ?: 0
            }
        suspend fun incrementCount() {
            dataStore.edit { preferences ->
                val currentValue = preferences[KEY_COUNT] 
                    ?: 0
                preferences[KEY_COUNT] = currentValue.
                    inc()
            }
        }
    }}
    ```

在这里，`KEY_COUNT`代表 DataStore 库用于存储请求数量的键。每当`saveCount`字段发生变化时，它将发出一个新的计数值，而`incrementCount`将增加当前保存的数字 1。

1.  现在，设置`AppDataStore`依赖项，就像我们处理 Retrofit 和 Room 依赖项一样。代码如下所示：

    ```java
    val Context.dataStore: DataStore<Preferences> by preferencesDataStore(name = "my_preferences")
    class MyApplication : Application() {
        companion object {
            …
            lateinit var appDataStore: AppDataStore
        }
        override fun onCreate() {
            super.onCreate()
            …
            appDataStore = AppDataStore(dataStore)
        }
    }
    ```

在这里，我们初始化`DataStore`对象，然后将其注入到`AppDataStore`类中。

1.  接下来，按照以下方式修改`MainViewModel`类：

    ```java
    class MainViewModel(
        private val userService: UserService,
        private val userDao: UserDao,
        private val appDataStore: AppDataStore
    ) : ViewModel() {
        var resultState by mutableStateOf(UiState())
            private set
        init {
            viewModelScope.launch {
                flow { emit(userService.getUsers()) }
                    .onEach {
                        val userEntities =
                            it.map { user -> UserEntity
                                (user.id, user.name, user.
                                 username, user.email) }
                        userDao.insertUsers(userEntities)
                        appDataStore.incrementCount()
                    }.flatMapConcat { userDao.getUsers() }
                    .catch { emitAll(userDao.getUsers()) }
                    .flatMapConcat { users ->
    appDataStore.savedCount.map { 
                            count -> UiState(users,  
                                count.toString()) }
                    }
                    .flowOn(Dispatchers.IO)
                    .collect {
                        resultState = it
                    }
            }
        }
    }
    ```

在这里，我们向`AppDataStore`添加一个新的依赖项，然后在将用户从 Retrofit 插入后，从`AppDataStore`调用`incrementCount`方法，然后我们将`AppDataStore`中的`savedCount`插入到现有的流程中，并创建一个新的`UiState`对象，该对象包含用户列表和计数，这些将被收集在`resultState`对象中。

1.  `UiState`类看起来可能如下所示：

    ```java
    data class UiState(
        val userList: List<UserEntity> = listOf(),
        val count: String = ""
    )
    ```

此类将保存来自我们两个持久数据源的信息。

1.  接下来，按照以下方式更改`MainViewModelFactory`：

    ```java
    class MainViewModelFactory : ViewModelProvider.Factory {
        override fun <T : ViewModel> create(modelClass: 
            Class<T>): T =
            MainViewModel(
                MyApplication.userService,
                MyApplication.userDao,
                MyApplication.appDataStore
            ) as T
    }
    ```

在这里，我们将一个新的依赖项注入到`AppDataStore`中，并将其注入到`MainViewModel`类中。

1.  最后，按照以下方式修改`MainActivity`类：

    ```java
    @Composable
    fun UserList(uiState: UiState) {
        LazyColumn(modifier = Modifier.padding(16.dp)) {
            item(uiState.count) {
                Column(modifier = Modifier.padding(16.dp)) {
                    Text(text = uiState.count)
                }
            }
            items(uiState.userList) {
                Column(modifier = Modifier.padding(16.dp)) {
                    Text(text = it.name)
                    Text(text = it.username)
                    Text(text = it.email)
                }
            }
        }
    }
    ```

在这里，我们将`UserEntity`列表替换为`UiState`依赖项，并在项目列表中添加一行新行，以指示请求的计数。

如果我们运行应用程序，我们将在顶部看到发送到服务器的当前请求计数。如果我们终止并重新打开应用程序，我们将看到计数增加，这显示了它如何在使用户停止应用程序或操作系统终止应用程序时存活下来。

在本节中，我们分析了在 Android 设备上通过 DataStore 库持久化数据的另一种常见方法。我们还观察到 DataStore 库与 flows 和其他库（如 Room 和 Retrofit）集成的简便性。

# 摘要

在本章中，我们讨论了如何在 Android 中加载数据和持久化数据，以及我们必须遵循的线程规则。我们首先分析了如何异步加载数据，并专注于协程和流，为此我们进行了简单的练习，以在不同的线程上执行异步操作并更新主线程上的 UI。然后我们研究了如何使用 OkHttp 和 Retrofit 从互联网加载数据，接着探讨了如何使用 Room 和 DataStore 持久化数据，以及我们如何将这些与协程和流集成在一起。我们在练习中强调了这些库的使用，同时也展示了它们如何与协程和流集成。不同数据流的集成被组合在`ViewModel`类中，其中我们加载网络数据并将其插入到本地数据库中。这通常不是一个好的方法，我们将在未来的章节中进一步探讨如何改进这一点。

在下一章中，我们将探讨如何向用户展示数据，以及我们可以使用的库和框架来实现这一点。
