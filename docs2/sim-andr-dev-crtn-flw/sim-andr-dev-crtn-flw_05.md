# *第四章*: 测试 Kotlin 协程

在上一章中，你学习了关于协程取消和如何使你的协程可取消的内容。然后，你学习了关于以毫秒或 `try-catch` 和 `CoroutineExceptionHandler` 形式的协程超时。

创建测试是应用程序开发的重要部分。你编写的代码越多，出现错误和缺陷的可能性就越高。有了测试，你可以确保你的应用程序按你编程的方式工作。你可以快速发现问题并立即修复它们。测试可以使开发更容易，节省时间和资源。它们还可以帮助你自信地重构和维护代码。

在本章中，你将学习如何在 Android 中测试 Kotlin 协程。首先，我们将更新 Android 项目以进行测试。然后，我们将继续学习创建 Kotlin 协程测试的步骤。

在本章中，我们将涵盖以下主题：

+   为测试协程设置 Android 项目

+   单元测试挂起函数

+   测试协程

在本章结束时，你将了解协程测试。你将能够为你的 Android 应用中的协程编写和运行单元测试和集成测试。

# 技术要求

你需要下载并安装 Android Studio 的最新版本。你可以在 [`developer.android.com/studio`](https://developer.android.com/studio) 找到最新版本。为了获得最佳学习体验，建议使用以下规格的计算机：

+   英特尔酷睿 i5 或更高版本

+   最小 4 GB RAM

+   可用空间 4 GB

本章的代码示例可以在 GitHub 上找到，链接为 [`github.com/PacktPublishing/Simplifying-Android-Development-with-Coroutines-and-Flows/tree/main/Chapter04`](https://github.com/PacktPublishing/Simplifying-Android-Development-with-Coroutines-and-Flows/tree/main/Chapter04)。

# 为测试协程设置 Android 项目

在本节中，我们将首先探讨如何更新你的 Android 应用以使其准备好添加和运行测试。一旦你的项目设置得当，添加单元和集成测试将变得容易。

在 Android 上创建单元测试时，你必须有 `app/build.gradle` 依赖项，当你在 Android Studio 中创建新的 Android 项目时。

如果你的 Android 项目还没有 JUnit 4，你可以通过在 `app/build.gradle` 依赖中包含以下内容来添加它：

```kt
dependencies {
```

```kt
    ...
```

```kt
    testImplementation ‘junit:junit:4.13.2’
```

```kt
}
```

这允许你使用 JUnit 4 框架进行单元测试。

为了创建测试的模拟对象，你也可以使用模拟库。Mockito 是最受欢迎的 Java 模拟库，你可以在 Android 上使用它。要将 Mockito 添加到你的测试中，请将以下内容添加到 `app/build.gradle` 文件中的依赖项：

```kt
dependencies {
```

```kt
    ...
```

```kt
    testImplementation ‘org.mockito:mockito-core:4.0.0’
```

```kt
}
```

添加此依赖项允许你在项目中使用 Mockito 创建用于单元测试的模拟对象。

如果您更喜欢使用 Mockito 与惯用的 Kotlin 代码，您可以使用 Mockito-Kotlin。Mockito-Kotlin 是一个包含辅助函数的 Mockito 库，可以使您的代码更接近 Kotlin。

要在您的 Android 单元测试中使用 Mockito-Kotlin，您可以在您的 `app/build.gradle` 文件依赖项中添加以下依赖项：

```kt
dependencies {
```

```kt
    ...
```

```kt
    testImplementation ‘org.mockito.kotlin:mockito-
```

```kt
      kotlin:4.0.0’
```

```kt
}
```

这将使您能够使用 Mockito 使用惯用的 Kotlin 代码创建测试的模拟对象。

如果您在项目中同时使用 Mockito (`mockito-core`) 和 Mockito-Kotlin，您只需添加 Mockito-Kotlin 的依赖项。它已经包含了 `mockito-core` 的依赖项，它将自动导入。

要测试 `LiveData` 等 Jetpack 组件，请添加 `androidx.arch.core:core-testing` 依赖项：

```kt
dependencies {
```

```kt
    ... 
```

```kt
    testImplementation ‘androidx.arch.core:core-
```

```kt
      testing:2.1.0’
```

```kt
}
```

此依赖项包含对测试 Jetpack 架构组件的支持。它包括 `InstantTaskExecutorRule` 等 JUnit 规则，您可以使用这些规则来测试您的代码中的 `LiveData` 对象。

测试协程比常规测试要复杂一些。这是因为协程是异步的，任务可以并行运行，并且任务可能需要一段时间才能完成。您的测试必须快速且一致。

为了帮助您测试协程，您可以使用来自 `kotlinx-coroutines-test` 包的协程测试库。它包含一些实用类，使测试协程更容易、更高效。要在您的 Android 项目中使用它，您必须在您的 `app/build.gradle` 文件依赖项中添加以下内容：

```kt
dependencies {
```

```kt
    ...
```

```kt
    testImplementation ‘org.jetbrains.kotlinx:kotlinx-
```

```kt
      coroutines-test:1.6.0’
```

```kt
}
```

这将导入 `kotlinx-coroutines-test` 依赖项到您的 Android 项目中。然后您将能够使用 Kotlin 协程测试库中的实用类为您的协程创建单元测试。

如果您想在将在模拟器或物理设备上运行的 Android 仪器测试中使用 `kotlinx-coroutines-test`，您应该在您的 `app/build.gradle` 文件中添加以下依赖项：

```kt
dependencies {
```

```kt
    ...
```

```kt
    androidTestImplementation
```

```kt
      ‘org.jetbrains.kotlinx:kotlinx-coroutines-test:1.6.0’
```

```kt
}
```

将此添加到您的依赖项中，将允许您在您的仪器测试中使用 `kotlinx-coroutines-test`。

截至 1.6.0 版本，协程测试库仍然被标记为实验性。您可能需要使用 `@ExperimentalCoroutinesApi` 注解来注释测试类，如下面的示例所示：

```kt
@ExperimentalCoroutinesApi
```

```kt
class MovieRepositoryUnitTest {
```

```kt
    ...
```

```kt
}
```

在本节中，您学习了如何设置您的 Android 项目以添加测试。您将在下一节中学习如何为挂起函数创建单元测试。

# 单元测试挂起函数

在本节中，我们将重点介绍您如何对挂起函数进行单元测试。您可以创建对 `ViewModel` 等类进行单元测试，这些类启动协程或具有挂起函数。

为挂起函数创建单元测试比编写挂起函数本身更困难，因为挂起函数只能从协程或另一个协程中调用。您可以做的事情是使用 `runBlocking` 协程构建器，并从那里调用挂起函数。例如，假设您有一个类似于以下 `MovieRepository` 类：

```kt
class MovieRepository (private val movieService:
```

```kt
  MovieService) {
```

```kt
    ...
```

```kt
    private val movieLiveData =
```

```kt
      MutableLiveData<List<Movie>>()
```

```kt
    fun fetchMovies() {
```

```kt
        ...
```

```kt
        val movies = movieService.getMovies()
```

```kt
        movieLiveData.postValue(movies.results)
```

```kt
    }
```

```kt
}
```

这个`MovieRepository`有一个名为`fetchMovies`的挂起函数。这个函数通过调用`movieService`中的`getMovies`挂起函数来获取电影列表。

要为`fetchMovies`函数创建测试，你可以使用`runBlocking`来调用挂起函数，如下所示：

```kt
class MovieRepositoryTest {
```

```kt
    ...
```

```kt
    @Test
```

```kt
    fun fetchMovies() {
```

```kt
        ...
```

```kt
        runBlocking {
```

```kt
            ...
```

```kt
            val movieLiveData =
```

```kt
              movieRepository.fetchMovies()
```

```kt
            assertEquals(movieLiveData.value, movies)
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

使用`runBlocking`协程构建器允许你调用挂起函数并进行断言检查。

`runBlocking`协程构建器对于测试很有用。然而，有时它可能会因为代码中的延迟而变慢。你的单元测试理想上应该尽可能快地运行。协程测试库可以通过其`runTest`协程构建器帮助你。它与`runBlocking`协程构建器相同，只是它立即且无延迟地运行挂起函数。

在上一个示例中将`runBlocking`替换为`runTest`会使你的测试看起来像以下这样：

```kt
@ExperimentalCoroutinesApi
```

```kt
class MovieRepositoryTest {
```

```kt
    ...
```

```kt
    @Test
```

```kt
    fun fetchMovies() {
```

```kt
        ...
```

```kt
        runTest {
```

```kt
            ...
```

```kt
            val movieLiveData =
```

```kt
              movieRepository.fetchMovies()
```

```kt
            assertEquals(movieLiveData.value, movies)
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

`runTest`函数允许你调用`movieRepository.fetchMovies()`挂起函数，然后检查操作的结果。

在本节中，你学习了如何在 Android 项目中编写挂起函数的单元测试。在下一节中，你将学习如何测试协程。

# 测试协程

在本节中，我们将关注如何测试你的协程。你可以为启动协程的类，如`ViewModel`，编写测试。

对于使用`Dispatchers.Main`启动的协程，你的单元测试将失败，并显示以下错误消息：

```kt
java.lang.IllegalStateException: Module with the Main dispatcher had failed to initialize. For tests Dispatchers.setMain from kotlinx-coroutines-test module can be used
```

这个异常发生是因为`Dispatchers.Main`使用`Looper.getMainLooper()`，这是应用程序的主线程。在 Android 中，对于本地单元测试，这个主循环器是不可用的。为了使你的测试工作，你必须使用`Dispatchers.setMain`扩展函数来更改`Main`分发器。例如，你可以在你的测试类中创建一个在测试之前运行的函数：

```kt
@Before
```

```kt
fun setUp() {
```

```kt
    Dispatchers.setMain(UnconfinedTestDispatcher())
```

```kt
}
```

`setUp`函数将在测试之前运行。它将主分发器更改为另一个分发器以供你的测试使用。

`Dispatchers.setMain`将更改所有后续的`Dispatchers.Main`使用。在测试之后，你必须通过调用`Dispatchers.resetMain()`来将`Main`分发器改回。你可以做如下操作：

```kt
@After
```

```kt
fun tearDown() {
```

```kt
    Dispatchers.resetMain()
```

```kt
}
```

测试运行后，将调用`tearDown`函数，该函数将重置`Main`分发器。

如果你有很多协程要测试，在每个测试类中复制和粘贴这个样板代码并不是一个好的选择。你可以创建一个自定义 JUnit 规则，这样你就可以在测试类中重复使用它。这个 JUnit 规则必须位于你的测试源集的根文件夹中，如图*图 4.01*所示：

![图 4.1 – 根测试文件夹中的自定义 TestCoroutineRule]

![图 4.1 – 根测试文件夹中的自定义 TestCoroutineRule]

图 4.1 – 根测试文件夹中的自定义 TestCoroutineRule

你可以编写的自定义 JUnit 规则示例，用于自动设置`Dispatchers.setMain`和`Dispatchers.resetMain`，是这个`TestCoroutineRule`：

```kt
@ExperimentalCoroutinesApi
```

```kt
class TestCoroutineRule(val dispatcher: TestDispatcher =
```

```kt
  UnconfinedTestDispatcher()):
```

```kt
   TestWatcher() {
```

```kt
   override fun starting(description: Description?) {
```

```kt
       super.starting(description)
```

```kt
       Dispatchers.setMain(dispatcher)
```

```kt
   }
```

```kt
   override fun finished(description: Description?) {
```

```kt
       super.finished(description)
```

```kt
       Dispatchers.resetMain()
```

```kt
   }
```

```kt
}
```

这个自定义 JUnit 规则将允许你的测试在测试之前自动调用`Dispatchers.setMain`，并在测试之后调用`Dispatchers.resetMain`。

你可以通过添加`@get:Rule`注解在你的测试类中使用这个`TestCoroutineRule`：

```kt
@ExperimentalCoroutinesApi
```

```kt
class MovieRepositoryTest {
```

```kt
    @get:Rule
```

```kt
    var coroutineRule = TestCoroutineRule()
```

```kt
    ...
```

```kt
}
```

使用这段代码，你不需要在测试类中每次都添加`Dispatchers.setMain`和`Dispatchers.resetMain`函数调用。

在测试你的协程时，你必须用`TestDispatcher`替换你的协程调度器。为了能够替换调度器，你的代码应该有一种方法来更改将用于协程的调度器。例如，这个`MovieViewModel`类有一个用于设置调度器的属性：

```kt
class MovieViewModel(private val dispatcher:
```

```kt
  CoroutineDispatcher = Dispatchers.IO): ViewModel() {
```

```kt
    ...
```

```kt
    fun fetchMovies() {
```

```kt
        viewModelScope.launch(dispatcher) {
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

`MovieViewModel`使用其构造函数中指定的调度器或默认值（`Dispatchers.IO`）来启动协程。

在你的测试中，你可以为测试目的设置不同的`Dispatcher`。对于前面的`ViewModel`，你的测试可以像以下示例那样使用不同的调度器初始化`ViewModel`：

```kt
@ExperimentalCoroutinesApi
```

```kt
class MovieViewModelTest {
```

```kt
    ...
```

```kt
    @Test
```

```kt
    fun fetchMovies() {
```

```kt
        ...
```

```kt
        runTest {
```

```kt
            ...
```

```kt
            val viewModel =
```

```kt
              MovieViewModel(UnconfinedTestDispatcher())
```

```kt
            viewModel.fetchMovies()
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

在`MovieViewModelTest`的`fetchMovies`测试中，`viewModel`使用`UnconfinedTestDispatcher`作为协程调度器以进行测试初始化。

在前面的例子中，你使用了`UnconfinedTestDispatcher`作为测试的`TestDispatcher`。在`kotlinx-coroutines-test`库中，有两个`TestDispatcher`的实现可用：

+   `StandardTestDispatcher`: 不自动运行协程，让你完全控制执行顺序

+   `UnconfinedTestDispatcher`: 自动运行协程；不提供控制协程启动顺序的能力

`StandardTestDispatcher`和`UnconfinedTestDispatcher`都有构造函数属性：`scheduler`用于`TestCoroutineScheduler`，`name`用于识别调度器。如果你没有指定调度器，`TestDispatcher`将默认创建一个`TestCoroutineScheduler`。

`StandardTestDispatcher`的`TestCoroutineScheduler`控制协程的执行。`TestCoroutineScheduler`有三个你可以调用的函数来控制任务的执行：

+   `runCurrent()`: 运行直到当前虚拟时间已安排的任务

+   `advanceUntilIdle()`: 运行所有挂起的任务

+   `advanceTimeBy(milliseconds)`: 运行挂起的任务，直到当前虚拟时间增加指定的毫秒数

`TestCoroutineScheduler`还有一个`currentTime`属性，它指定了当前虚拟时间（以毫秒为单位）。当你调用如`advanceTimeBy`这样的函数时，它将更新调度器的`currentTime`属性。

`runTest`协程构建器创建一个具有`TestScope`协程范围的协程。这个`TestScope`包含一个`TestCoroutineScheduler`（`testScheduler`），你可以使用它来控制任务的执行。

这个 `testScheduler` 还有一个名为 `currentTime` 的扩展属性和 `runCurrent`、`advanceUntilIdle` 以及 `advanceTimeBy` 的扩展函数，这简化了从 `TestScope` 的 `testScheduler` 调用这些函数。

使用 `runTest` 和 `TestDispatcher` 可以测试协程中存在时间延迟的情况，您想在执行下一行代码之前测试一行代码。例如，如果您的 `ViewModel` 有一个在网络操作之前设置为 `true` 的 `loading` 布尔变量，然后操作完成后重置为 `false`，您的 `loading` 变量的测试可能看起来像这样：

```kt
@Test
```

```kt
fun loading() {
```

```kt
    val dispatcher = StandardTestDispatcher()
```

```kt
    runTest() {
```

```kt
        val viewModel = MovieViewModel(dispatcher)
```

```kt
        viewModel.fetchMovies()
```

```kt
        dispatcher.scheduler.advanceUntilIdle()
```

```kt
        assertEquals(false, viewModel.loading.value)
```

```kt
    }
```

```kt
}
```

这个测试使用 `StandardTestDispatcher`，这样您就可以控制任务的执行。在调用 `fetchMovies` 后，您在调度器的 `scheduler` 上调用 `advanceUntilIdle` 来运行任务，任务完成后将 `loading` 值设置为 `false`。

在本节中，您学习了如何为您的协程添加测试。让我们通过向 Android 项目中现有的协程添加一些测试来测试我们到目前为止所学的内容。

# 练习 4.01 – 在 Android 应用中添加协程测试

对于这个练习，您将继续在 *练习 2.01*，*在 Android 应用中使用协程* 中工作的电影应用。您将通过以下步骤为项目中的协程添加单元测试：

1.  打开您在 *练习 2.01*，*在 Android 应用中使用协程* 中工作的电影应用，在 Android Studio 中。

1.  前往 `app/build.gradle` 文件，添加以下依赖项，这些依赖项将用于单元测试：

    ```kt
    testImplementation ‘org.mockito.kotlin:mockito-
      kotlin:4.0.0’
    testImplementation ‘androidx.arch.core:core-
      testing:2.1.0’
    testImplementation ‘org.jetbrains.kotlinx:kotlinx-
      coroutines-test:1.6.0’
    ```

第一行将添加 Mockito-Core 和 Mockito-Kotlin，第二行将添加架构测试库，最后一行将添加 Kotlin 协程测试库。您将使用这些库来为 Android 项目添加的单元测试。

1.  在 `app/src/test/resources` 中创建一个 `mockito-extensions` 目录。在该目录中，创建一个名为 `org.mockito.plugins.MockMaker` 的新文件，如图 4.2 所示：

![图 4.2 – 需要添加到 app/src/test/mockito-extensions 目录的文件]

![img/Figure_4.2_B17773.jpg]

图 4.2 – 需要添加到 app/src/test/mockito-extensions 目录的文件

1.  在 `app/src/test/mockito-extensions/org.mockito.plugins.MockMaker` 文件中，添加以下内容：

    ```kt
    mock-maker-inline
    ```

这将允许您使用 Mockito 为代码中的最终类创建模拟对象。如果没有这个，您的测试将失败，并显示以下错误消息：

```kt
Mockito cannot mock/spy because : final class
```

1.  您首先将为 `MovieRepository` 类添加一个单元测试。在 `app/src/test` 中创建一个名为 `MovieRepositoryTest` 的测试类，并为此类添加 `@OptIn(ExperimentalCoroutinesApi::class)` 注解：

    ```kt
    @OptIn(ExperimentalCoroutinesApi::class)
    class MovieRepositoryTest {
        ...
    }
    ```

这将是 `MovieRepository` 的测试类。添加了 `ExperimentalCoroutinesApi` 的 `OptIn` 注解，因为 `kotlinx-coroutines-test` 库中的一些类仍然标记为实验性。

1.  在`MovieRepositoryTest`类内部，添加一个`InstantTaskExecutorRule`的 JUnit 测试规则：

    ```kt
    @get:Rule
    val rule = InstantTaskExecutorRule()
    ```

`InstantTaskExecutorRule`允许测试同步执行任务。这对于`MovieRepository`中的`LiveData`对象是必需的。

1.  创建一个名为`fetchMovies`的测试函数，以测试`MovieRepository`中的`fetchMovies`挂起函数成功检索电影列表：

    ```kt
    @Test
    fun fetchMovies() {
        ...
    }
    ```

这将是`MovieRepository.fetchMovies`的第一个测试：一个显示电影列表的成功场景。

1.  在`MovieRepositoryTest`类的`fetchMovies`函数中，添加以下代码以模拟`MovieRepository`和`MovieService`：

    ```kt
    @Test
    fun fetchMovies() {
        val movies = listOf(Movie(id = 3), Movie(id = 4))
        val response = MoviesResponse(1, movies)

        val movieService: MovieService = mock {
            onBlocking { getMovies(anyString()) } doReturn
              response
        }
        val movieRepository =
          MovieRepository(movieService)
    }
    ```

这将模拟`MovieService`，使其在调用其`getMovies`函数时始终返回我们提供的`movies`列表。

1.  在`MovieRepositoryTest`的`fetchMovies`函数末尾添加以下代码，以测试从`MovieRepository`类调用`fetchMovies`返回我们期望返回的电影列表：

    ```kt
    @Test
    fun fetchMovies() {
        ...

        runTest {
            movieRepository.fetchMovies()
            val movieLiveData = movieRepository.movies
            assertEquals(movies, movieLiveData.value)
        }
    }
    ```

这将调用`MovieRepository`类中的`fetchMovies`函数，该函数将调用`MovieService`中的`getMovies`函数。我们正在检查它是否确实返回了我们之前在模拟的`MovieService`中设置的影片列表。

1.  运行`MovieRepositoryTest`类。`MovieRepositoryTest`应该通过，并且不应该有错误。

1.  在`MovieRepositoryTest`类中创建另一个名为`fetchMoviesWithError`的测试函数，以测试`MovieRepository`中的`fetchMovies`挂起函数在检索电影列表时失败：

    ```kt
    @Test
    fun fetchMoviesWithError() {
        ...
    }
    ```

这将测试`MovieRepository`在检索电影列表时失败的情况。

1.  在`MovieRepositoryTest`类的`fetchMoviesWithError`函数中，添加以下代码：

    ```kt
    @Test
    fun fetchMoviesWithError() {
        val exception = “Test Exception” 
        val movieService: MovieService = mock {
            onBlocking { getMovies(anyString()) } doThrow
              RuntimeException(exception)
        }
        val movieRepository =
          MovieRepository(movieService)
    }
    ```

这将模拟`MovieService`，使其在调用其`getMovies`函数时始终抛出一个包含消息`Test Exception`的异常。

1.  在`MovieRepositoryTest`的`fetchMoviesWithError`函数末尾添加以下代码，以测试从`MovieRepository`类调用`fetchMovies`返回我们期望返回的电影列表：

    ```kt
    @Test
    fun fetchMovies() {
        ...

        runTest {
            movieRepository.fetchMovies()

            val movieLiveData = movieRepository.movies
            assertNull(movieLiveData.value)

            val errorLiveData = movieRepository.error
            assertNotNull(errorLiveData.value)
            assertTrue(errorLiveData.value.toString()
              .contains(exception))
            }
    }
    ```

这将调用`MovieRepository`类中的`fetchMovies`函数，该函数将调用始终在调用时抛出异常的`MovieService`中的`getMovies`函数。

在第一个断言中，我们检查`movieLiveData`是否为 null，因为没有检索到电影。第二个断言检查`errorLiveData`是否不为 null，因为有异常。最后一个断言检查`errorLiveData`是否包含我们在上一步设置的`Test Exception`消息。

1.  运行`MovieRepositoryTest`测试。`fetchMovies`和`fetchMoviesWithError`测试都应该没有错误，并且都应该通过。

1.  然后，我们将为`MovieViewModel`创建一个测试。首先，我们需要更新`MovieViewModel`，以便我们可以更改协程运行的调度器。打开`MovieViewModel`类，并通过添加一个用于设置协程调度器的`dispatcher`属性来更新其构造函数：

    ```kt
    class MovieViewModel(private val movieRepository:
      MovieRepository, private val dispatcher:
      CoroutineDispatcher = Dispatchers.IO) : ViewModel() 
      {
        ...
    }
    ```

这将允许您使用另一个`dispatcher`更改`MovieViewModel`的`dispatcher`，您将在测试中这样做。

1.  在`fetchMovies`函数中，将`launch`协程构建器更改为使用构造函数中的`dispatcher`而不是硬编码的`dispatcher`：

    ```kt
    viewModelScope.launch(dispatcher) {
        ...
    }
    ```

此更新将代码修改为使用从构造函数设置的`dispatcher`或默认的`Dispatchers.IO`。现在您可以创建一个针对`MovieViewModel`类的单元测试。

1.  在`app/src/test`目录中，为`MovieViewModel`创建一个名为`MovieViewModelTest`的测试类，并将`@OptIn(ExperimentalCoroutinesApi::class)`注解添加到类中：

    ```kt
    @OptIn(ExperimentalCoroutinesApi::class)
    class MovieViewModelTest {
        ...
    }
    ```

这将是`MovieViewModel`的测试类。添加了`ExperimentalCoroutinesApi`注解，因为`kotlinx-coroutines-test`库中的一些类仍然是实验性的。

1.  在`MovieViewModelTest`类内部，添加一个针对`InstantTaskExecutorRule`的 JUnit 测试规则：

    ```kt
    @get:Rule
    val rule = InstantTaskExecutorRule()
    ```

单元测试中的`InstantTaskExecutorRule`会同步执行任务。这是为了`MovieViewModel`中的`LiveData`对象。

1.  创建一个名为`fetchMovies`的测试函数，以测试`MovieViewModel`中的`fetchMovies`挂起函数：

    ```kt
    @Test
    fun fetchMovies() {
        val expectedMovies =
          MutableLiveData<List<Movie>>()
        expectedMovies.postValue(listOf(Movie
          (title = “Movie”)))

        val movieRepository: MovieRepository = mock {
            onBlocking { movies } doReturn expectedMovies
        }
     }
    ```

这将模拟`MovieRepository`，使其`movies`属性始终返回`expectedMovies`作为其值。

1.  在`MovieViewModelTest`的`fetchMovies`测试结束时，添加以下内容以测试`MovieViewModel`的`movies`将等于`expectedMovies`：

    ```kt
    @Test
    fun fetchMovies() {
        ...

        val movieViewModel = 
          MovieViewModel(movieRepository)
        assertEquals(expectedMovies.value,
          movieViewModel.movies.value)
    }
    ```

这将使用模拟的`MovieRepository`创建一个`MovieViewModel`。我们正在检查`MovieViewModel`的`movies`值是否等于我们设置到模拟的`MovieRepository`中的`expectedMovies`值。

1.  运行`MovieViewModelTest`或所有测试（`MovieRepositoryTest`和`MovieViewModelTest`）。所有测试都应该通过。

1.  在`MovieViewModelTest`中创建另一个名为`loading`的测试函数，以测试`MovieViewModel`中的`loading` `LiveData`：

    ```kt
    @Test
    fun loading() {
        ...
    }
    ```

这将测试`MovieViewModel`的`loading` `LiveData`属性。在获取电影时，加载属性为`true`并显示`ProgressBar`。在成功获取电影或遇到错误后，它变为`false`并隐藏`ProgressBar`。

1.  在`loading`测试函数中，添加以下内容以模拟`MovieRepository`并初始化一个将用于`MovieViewModel`的`dispatcher`：

    ```kt
    @Test
    fun loading() {
        val movieRepository: MovieRepository = mock()
        val dispatcher = StandardTestDispatcher()

        ...
    }
    ```

这将模拟`MovieRepository`并创建一个用于`MovieViewModel`测试的`StandardTestDispatcher`类型的`dispatcher`。这将允许您控制任务的执行，稍后用于检查`MovieViewModel`的`loading`属性值。

1.  在`loading`测试函数的末尾，添加以下内容以测试加载`MovieViewModel`的`loading`属性：

    ```kt
    @Test
    fun loading() {
        ...

        runTest {
            val movieViewModel =
              MovieViewModel(movieRepository, dispatcher)

            movieViewModel.fetchMovies()
            assertTrue( movieViewModel.loading.value ==
              true)
            dispatcher.scheduler.advanceUntilIdle()
            assertFalse(movieViewModel.loading.value ==
              true)
        }
    }
    ```

这将创建一个带有模拟的`MovieRepository`和`dispatcher`的`MovieViewModel`。然后，从`MovieViewModel`调用`fetchMovies`以获取电影列表。

第一个断言检查 `loading` 值是否为 `true`。然后我们使用调度器的 `scheduler` 中的 `advanceUntilIdle` 来执行所有任务。这应该将 `loading` 值更改为 `false`。最后一行检查这确实发生了。

1.  运行 `MovieRepositoryTest` 和 `MovieViewModelTest`。所有测试都应该通过。

在这个练习中，你在一个使用协程的 Android 项目上工作，并为这些协程添加了单元测试。

# 摘要

本章重点介绍了在 Android 应用程序中测试协程。你从学习如何设置 Android 项目以准备添加协程测试开始。协程测试库（`kotlinx-coroutines-test`）帮助你为协程创建测试。

你学习了如何为你的挂起函数添加单元测试。你可以使用 `runBlocking` 和 `runTest` 来测试调用挂起函数的代码。`runTest` 立即运行代码，没有延迟。

然后，你学习了如何测试协程。你可以通过 `TestDispatcher`（`StandardTestDispatcher` 或 `UnconfinedTestDispatcher`）更改测试中的调度器。`TestCoroutineScheduler` 允许你控制协程任务的执行。

最后，你完成了一个练习，在该练习中，你为现有的 Android 项目中的协程添加了单元测试。

在下一章中，你将探索 Kotlin 流（Kotlin Flows）并学习如何使用它们在 Android 中进行异步编程。

# 进一步阅读

本书假设你已经具备测试 Android 应用程序的知识。如果你想了解更多关于 Android 测试的内容，你可以阅读《如何用 Kotlin 构建 Android 应用程序》（Packt Publishing，2021，ISBN 9781838984113）一书中第九章，“使用 JUnit、Mockito 和 Espresso 进行单元测试和集成测试”。你还可以查看 Android 测试文档，网址为 [`developer.android.com/training/testing`](https://developer.android.com/training/testing)。

截至撰写本文时，协程测试库仍然被标记为实验性。在库变得稳定之前，类中可能会有一些破坏代码的更改。你可以在 GitHub 上查看库的最新版本，网址为 [`github.com/Kotlin/kotlinx.coroutines/tree/master/kotlinx-coroutines-test`](https://github.com/Kotlin/kotlinx.coroutines/tree/master/kotlinx-coroutines-test)，以获取关于协程测试库的最新信息。
