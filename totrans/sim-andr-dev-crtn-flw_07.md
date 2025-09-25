# *第五章*：使用 Kotlin Flows

在前三章中，我们深入探讨了 Kotlin 协程，并学习了如何在 Android 中使用它们进行异步编程。我们学习了协程构建器、作用域、调度器、上下文和作业。然后我们学习了如何处理协程取消、超时和异常。我们还学习了如何在代码中对协程进行测试。

在接下来的三章中，我们将重点关注 Kotlin Flow，这是一个基于 Kotlin 协程构建的新异步流库。Flow 可以在一段时间内发出多个值，而不仅仅是单个值。你可以使用 Flows 来处理数据流，例如实时位置、传感器读数和实时数据库值。

在本章中，我们将探索 Kotlin Flows。我们将从构建 Kotlin Flows 开始。然后，我们将探讨你可以用于转换、组合、缓冲以及更多 Flows 的各种操作符。最后，我们将了解 StateFlows 和 SharedFlows。

本章涵盖了以下主要主题：

+   在 Android 中使用 Flows

+   使用 Flow 构建器创建 Flows

+   使用 Flows 的操作符

+   缓冲和组合 Flows

+   探索 StateFlow 和 SharedFlow

到本章结束时，你将更深入地了解使用 Kotlin Flows。你将能够在你 Android 应用程序的各种情况下使用 Flows。你还将了解流构建器、操作符、组合 Flows、StateFlow 和 SharedFlow。

# 技术要求

你需要下载并安装 Android Studio 的最新版本。你可以在 [`developer.android.com/studio`](https://developer.android.com/studio) 找到最新版本。为了获得最佳学习体验，建议使用以下配置的计算机：

+   英特尔酷睿 i5 或更高性能的处理器

+   至少 4 GB RAM

+   4 GB 可用空间

本章的代码示例可以在 GitHub 上找到，地址为 [`github.com/PacktPublishing/Simplifying-Android-Development-with-Coroutines-and-Flows/tree/main/Chapter05`](https://github.com/PacktPublishing/Simplifying-Android-Development-with-Coroutines-and-Flows/tree/main/Chapter05)。

# 在 Android 中使用 Flows

在本节中，我们将首先使用 Android 中的 Flows 进行异步编程。Flows 对于涉及实时数据更新的应用程序部分来说非常理想。

数据流由 `Flow<String>` 表示，这是一个发出字符串值的 Flow。

Android Jetpack 库，如 Room、Paging、DataStore、WorkManager 和 Jetpack Compose，都内置了对 Flow 的支持。

Room 数据库库从 2.2 版本开始支持 Flows。这允许你通过使用 Flows 来通知数据库值的变化。

如果你的 Android 应用程序使用 **数据访问对象**（**DAO**）来显示电影列表，你的项目可以有一个如下所示的 DAO：

```kt
@Dao
```

```kt
interface MovieDao {
```

```kt
    @Query("SELECT * FROM movies")
```

```kt
    fun getMovies(): List<Movie>
```

```kt
    ...
```

```kt
}
```

通过从 `MovieDao` 调用 `getMovies` 函数，你可以从数据库中获取电影列表。

上述代码在调用`getMovies`后只会获取一次电影列表。你可能希望你的应用程序在数据库中的电影被添加、删除或更新时自动更新电影列表。你可以通过使用 Room-KTX 并将你的`MovieDao`更改为使用 Flow 来`getMovies`来实现这一点：

```kt
@Dao
```

```kt
interface MovieDao {
```

```kt
    @Query("SELECT * FROM movies")
```

```kt
    fun getMovies(): Flow<List<Movie>>
```

```kt
    ...
```

```kt
}
```

使用此代码，每当`movies`表发生变化时，`getMovies`将发出一个包含数据库中电影列表的新列表。然后你的应用程序可以使用它来自动更新列表或`RecyclerView`中显示的电影。

如果你使用`LiveData`并想将`LiveData`转换为`Flow`或`Flow`转换为`LiveData`，你可以使用 LiveData KTX。

要将`LiveData`转换为`Flow`，你可以使用`LiveData.asFlow()`扩展函数。使用`Flow.asLiveData()`扩展函数将`Flow`转换为`LiveData`。你可以通过在你的`app/build.gradle`依赖项中包含以下内容将 LiveData KTX 添加到你的项目中：

```kt
dependencies {
```

```kt
    ...
```

```kt
    implementation ‘androidx.lifecycle:lifecycle-livedata-
```

```kt
      ktx:2.2.0'
```

```kt
}
```

这将 LiveData KTX 添加到你的项目中，允许你使用`asFlow()`和`asLiveData()`扩展函数将`LiveData`转换为`Flow`和`Flow`转换为`LiveData`。

第三方 Android 库现在也支持 Flows；一些函数可以返回 Flow 对象。如果你在项目中使用 RxJava 3，你可以使用`Flow`到`Flowable`或`Observable`，反之亦然。

当你调用`collect`函数时，Flow 才会开始发出值。`collect`函数是一个挂起函数，所以你应该从协程或另一个挂起函数中调用它。

在下面的例子中，`collect()`函数是从使用`lifecycleScope`中的`launch`协程构建器创建的协程中调用的：

```kt
class MainActivity : AppCompatActivity() {
```

```kt
    ...
```

```kt
    override fun onCreate(savedInstanceState: Bundle?) {
```

```kt
        ...
```

```kt
        lifecycleScope.launch {
```

```kt
            viewModel.fetchMovies().collect { movie ->
```

```kt
                Log.d("movies", "${movie.title}")
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
}
```

```kt
class MovieViewModel : ViewModel() {
```

```kt
    ...
```

```kt
    fun fetchMovies(): Flow<Movie> {
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

在这个例子中，`collect{}`函数被调用在`Flow<Movie>`上，并通过调用`viewModel.fetchMovies()`返回。这将导致 Flow 开始发出值，然后你可以处理每个值。

流的收集发生在父协程的`CoroutineContext`中。在上面的例子中，协程上下文来自`viewModelScope`。

要更改 Flow 运行的`CoroutineContext`，你可以使用`flowOn()`函数。如果你想将前一个例子中的 Flow 的`Dispatcher`更改为`Dispatchers.IO`，你可以使用`flowOn(Dispatchers.IO)`，如下面的例子所示：

```kt
class MainActivity : AppCompatActivity() {
```

```kt
    ...
```

```kt
    override fun onCreate(savedInstanceState: Bundle?) {
```

```kt
        ...
```

```kt
        lifecycleScope.launch {
```

```kt
            viewModel.fetchMovies()
```

```kt
                .flowOn(Dispatchers.IO)
```

```kt
                .collect { movie ->
```

```kt
                    Log.d("movies", "${movie.title}")
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
}
```

在这里，在收集 Flow 之前，通过调用`flowOn`并使用`Dispatchers.IO`，将 Flow 运行的调度器更改为`Dispatchers.IO`。

当你调用`flowOn`时，它只会改变调用之前的函数或操作符，而不会改变调用之后的。在下面的例子中，在调用`flowOn`之后调用了`map`操作符来更改分发器，因此其上下文不会改变：

```kt
class MainActivity : AppCompatActivity() {
```

```kt
    ...
```

```kt
    override fun onCreate(savedInstanceState: Bundle?) {
```

```kt
        ...
```

```kt
        lifecycleScope.launch {
```

```kt
            viewModel.fetchMovies()
```

```kt
                .flowOn(Dispatchers.IO)
```

```kt
                .map { ... }
```

```kt
                .collect { movie ->
```

```kt
                    Log.d("movies", "${movie.title}")
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
}
```

在这个例子中，`flowOn`只会改变调用之前的上下文，所以`map`调用不会改变。它仍然会使用原始上下文（即来自`lifecycleScope`的上下文）。

在 Android 中，你可以在 Fragment 或 Activity 类中收集 Flow 以在 UI 中显示数据。如果 UI 进入后台，你的 Flow 将继续收集数据。你的应用必须停止收集 Flow 并更新屏幕，以防止内存泄漏和避免资源浪费。

为了在 Android UI 层面上安全地收集 Flows，你需要自己处理生命周期变化。你也可以使用 `Lifecycle.repeatOnLifecycle` 和 `Flow.flowWithLifecycle`，这些在 `app/build.gradle` 依赖中可用：

```kt
dependencies {
```

```kt
    ...    
```

```kt
    implementation ‘androidx.lifecycle:lifecycle-runtime-
```

```kt
      ktx:2.4.1
```

```kt
}
```

这添加了 `Lifecycle.repeatOnLifecycle` 和 `Flow.flowWithLifecycle`。

`Lifecycle.repeatOnLifecycle(state, block)` 将父协程挂起，直到生命周期被销毁，并在生命周期至少处于你设置的 `state` 时执行挂起的 `block` 代码。当生命周期离开该状态时，`repeatOnLifecycle` 将停止 Flow 并在生命周期回到该状态时重新启动。

如果你使用了 `repeatOnLifecycle`，它将在生命周期启动时开始收集 Flow。它将在生命周期停止、调用生命周期的 `onStop()` 时停止。

当你使用 `repeatOnLifecycle` 时，它将在每次生命周期恢复时开始收集 Flow，并在生命周期暂停或调用 `onPause()` 时停止。

建议在活动的 `onCreate` 方法或片段的 `onViewCreated` 方法中调用 `Lifecycle.repeatOnLifecycle`。

以下展示了如何在你的 Android 项目中使用 `Lifecycle.repeatOnLifecycle`：

```kt
class MainActivity : AppCompatActivity() {
```

```kt
    ...
```

```kt
    override fun onCreate(savedInstanceState: Bundle?) {
```

```kt
        ...
```

```kt
        lifecycleScope.launch {
```

```kt
            repeatOnLifecycle(Lifecycle.State.STARTED) {
```

```kt
                viewModel.fetchMovies()
```

```kt
                    .collect { movie ->
```

```kt
                        Log.d("movies", "${movie.title}")
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
    }
```

```kt
}
```

这里，我们使用了 `repeatOnLifecycle` 与 **Lifecycle.State.STARTED** 来在生命周期启动时开始收集电影流，并在生命周期停止时停止。

你可以使用 `Lifecycle.repeatOnLifecycle` 收集多个 Flow。为此，你必须在不同协程中并行收集它们：

```kt
class MainActivity : AppCompatActivity() {
```

```kt
    ...
```

```kt
  override fun onCreate(savedInstanceState: Bundle?) {
```

```kt
      ...
```

```kt
      lifecycleScope.launch {
```

```kt
          repeatOnLifecycle(Lifecycle.State.STARTED) {
```

```kt
              launch {
```

```kt
                 viewModel.fetchMovies().collect { movie ->
```

```kt
                      Log.d("movies", "${movie.title}")
```

```kt
                    }
```

```kt
                }
```

```kt
              launch {
```

```kt
                  viewModel.fetchTVShows.collect { show ->
```

```kt
                      Log.d("tv shows", "${show.title}")
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
        }
```

```kt
    }
```

```kt
}
```

这里有两个 Flows：一个用于收集电影，另一个用于收集电视剧。Flow 的收集是从不同的 `launch` 协程构建器开始的。

如果你只有一个 Flow 要收集，你也可以使用 `Flow.flowWithLifecycle`。当生命周期至少处于 `Lifecycle.repeatOnLifecycle` 内部时，它会从上游 Flow（调用之前的 Flow 和操作符）发出值。你可以像以下代码所示使用 `Flow.flowWithLifecycle`：

```kt
class MainActivity : AppCompatActivity() {
```

```kt
    ...
```

```kt
    override fun onCreate(savedInstanceState: Bundle?) {
```

```kt
        ...
```

```kt
        lifecycleScope.launch {
```

```kt
            viewModel.fetchMovies()          
```

```kt
                .flowWithLifecycle(lifecycle,
```

```kt
                  Lifecycle.State.STARTED)
```

```kt
                .collect { movie ->
```

```kt
                    Log.d("movies", "${movie.title}")
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
}
```

在这个示例中，你使用了 `flowWithLifecycle` 与 **Lifecycle.State.STARTED** 来在生命周期启动时开始收集电影流，并在生命周期停止时停止。

在本节中，你学习了如何在你的 Android 应用中使用 Kotlin Flows。你可以在 Room 等 Android Jetpack 库以及第三方库中使用 Flow。为了在 UI 层面上安全地收集 Flows 并防止内存泄漏以及避免资源浪费，你可以使用 `Lifecycle.repeatOnLifecycle` 和 `Flow.flowWithLifecycle`。

在下一节中，我们将探讨你可以用于为你的应用程序创建 Flows 的不同 Flow 构建器。

# 使用 Flow 构建器创建 Flows

在本节中，我们将首先探讨创建 Flows。要创建一个 Flow，你可以使用 Flow 构建器。

Kotlin Flow API 有你可以用来创建 Flows 的流构建器。以下是你可以使用 Kotlin Flow 构建器：

+   `flow {}`

+   `flowOf()`

+   `asFlow()`

`flow` 构建器函数从一个可挂起 lambda 块创建一个新的 Flow。在块内部，你可以使用 `emit` 函数发送值。例如，这个 `MovieViewModel` 的 `fetchMovieTitles` 函数返回 `Flow<String>`：

```kt
class MovieViewModel : ViewModel() {
```

```kt
    ...
```

```kt
    fun fetchMovieTitles(): Flow<String> = flow {
```

```kt
        val movies = fetchMoviesFromNetwork()
```

```kt
        movies.forEach { movie -> 
```

```kt
            emit(movie.title)
```

```kt
        }
```

```kt
    }
```

```kt
    private fun fetchMoviesFromNetwork(): List<Movie> {
```

```kt
         …
```

```kt
    }
```

```kt
}
```

在这个示例中，`fetchMovieTitles` 创建了一个包含电影标题的 Flow。它遍历了从 `fetchMoviesFromNetwork` 获取的电影列表，并为每部电影使用 `emit` 函数发出电影的标题。

使用 `flowOf` 函数，你可以创建一个生成指定值或 `vararg` 值的 Flow。在下面的示例中，`flowOf` 函数被用来创建一个包含前三部电影标题的 Flow：

```kt
class MovieViewModel : ViewModel() {
```

```kt
    ...
```

```kt
    fun fetchTop3Titles(): Flow<List<Movie>> {
```

```kt
        val movies = fetchMoviesFromNetwork().sortedBy {
```

```kt
            it.popularity }
```

```kt
        return flowOf(movies[0].title, 
```

```kt
            movies[1].title, 
```

```kt
            movies[2].title)
```

```kt
    }
```

```kt
    private fun fetchMoviesFromNetwork(): List<Movie> {
```

```kt
        …
```

```kt
    }
```

```kt
}
```

在这里，`fetchTop3Titles` 使用 `flowOf` 创建了一个包含前三部电影标题的 Flow。

`asFlow()` 扩展函数允许你将类型转换为 Flow。你可以在序列、数组、范围、集合和函数类型上使用它。例如，这个 `MovieViewModel` 有 `fetchMovieIds` 返回 `Flow<Int>`，包含电影 ID：

```kt
class MovieViewModel : ViewModel() {
```

```kt
    ...
```

```kt
    private fun fetchMovieIds(): Flow<Int> {
```

```kt
        val movies: List<Movie> = fetchMoviesFromNetwork()
```

```kt
        return movies.map { it.id }.asFlow()
```

```kt
    }
```

```kt
    private fun fetchMoviesFromNetwork(): List<Movie> {
```

```kt
        …
```

```kt
    }
```

```kt
}
```

在这个示例中，我们在电影列表上使用了一个 `map` 函数来创建一个电影 ID 的列表。然后，使用 `asFlow()` 扩展函数将该电影 ID 列表转换为 `Flow<String>`。

在本节中，我们学习了如何使用 Flow 构建器创建 Flows。在下一节中，我们将检查你可以用于转换、组合和更多操作 Flows 的各种 Kotlin Flow 操作符。

# 使用操作符与 Flows 一起使用

在本节中，我们将重点关注各种 Flow 操作符。Kotlin Flow 有内置的操作符，你可以与 Flows 一起使用。我们可以使用终端操作符收集 Flows，并使用中间操作符转换 Flows。

## 使用终端操作符收集 Flows

在本节中，我们将探讨你可以用于在 Flows 上启动收集的终端操作符。我们在前面的示例中使用的 `collect` 函数是最常用的终端操作符。然而，还有其他内置的终端 Flow 操作符。

以下是你可以使用以启动 Flow 收集的内置终端 Flow 操作符：

+   `toList`：收集 Flow 并将其转换为列表

+   `toSet`：收集 Flow 并将其转换为集合

+   `toCollection`：收集 Flow 并将其转换为集合

+   `count`：返回 Flow 中的元素数量

+   `first`：返回 Flow 的第一个元素，如果 Flow 为空则抛出 **NoSuchElementException**

+   `firstOrNull`：返回 Flow 的第一个元素，如果 Flow 为空则返回 null

+   `last`: 返回流的最后一个元素，如果流为空则抛出**NoSuchElementException**

+   `lastOrNull`: 返回流的最后一个元素，如果流为空则返回 null

+   `single`: 返回流中发出的单个元素，如果流为空或包含多个值则抛出异常

+   `singleOrNull`: 返回流中发出的单个元素，如果流为空或包含多个值则返回 null

+   `reduce`: 对每个发出的项目应用一个函数，从第一个元素开始，并返回累积的结果

+   `fold`: 对每个发出的项目应用一个函数，从初始值开始，并返回累积的结果

这些终端流操作符与标准 Kotlin 库中具有相同名称的 Kotlin 集合函数类似。

在以下示例中，使用`firstOrNull`终端操作符代替`collect`操作符来从`ViewModel`收集流：

```kt
class MainActivity : AppCompatActivity() {
```

```kt
    ...
```

```kt
    override fun onCreate(savedInstanceState: Bundle?) {
```

```kt
        ...
```

```kt
        lifecycleScope.launch {
```

```kt
            repeatOnLifecycle(Lifecycle.State.STARTED) {
```

```kt
                val topMovie =
```

```kt
                  viewModel.fetchMovies().firstOrNull()
```

```kt
                displayMovie(topMovie)
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
}
```

在这里，`firstOrNull`被用于流以获取第一个项目（如果流为空则为 null），这代表顶级电影。然后它将在屏幕上显示。

在本节中，你了解了可以使用哪些流的终端操作符来开始从流中收集数据。在下一节中，我们将学习如何使用中间操作符转换流。

## 使用中间操作符转换流

在本节中，我们将关注你可以用来转换流的中间流操作符。使用中间操作符，你可以根据原始流返回一个新的流。

中间操作符允许你修改一个流并返回一个新的流。你可以链接各种操作符，并且它们将按顺序应用。

你可以通过对它们应用操作符来转换流，就像你可以对 Kotlin 集合做的那样。以下中间操作符与具有相同名称的 Kotlin 集合函数类似：

+   `filter`: 返回一个流，仅选择满足你传递的条件的流中的值

+   `filterNot`: 返回一个流，仅选择不满足你传递的条件的流中的值

+   `filterNotNull`: 返回一个流，仅包含来自原始流且不为 null 的值

+   `filterIsInstance`: 返回一个流，仅包含流中指定类型的实例

+   `map`: 返回一个流，包含使用你指定的操作转换的流中的值

+   `mapNotNull`: 类似于`map`（使用指定的操作转换流），但仅包括非 null 的值

+   `withIndex`: 返回一个流，将每个值转换为包含值的索引和值本身的**IndexedValue**

+   `onEach`: 返回一个流，在发出每个值之前执行指定的操作

+   `runningReduce`: 返回一个包含从第一个元素开始按顺序运行操作所得到的累积值的流

+   `runningFold`: 返回一个包含从指定操作按顺序运行并从设置的初始值开始的累积值的 Flow

+   `scan`: 类似于 `runningFold` 操作符

还有一个 `transform` 操作符，您可以使用它来应用自定义或复杂的操作。使用 `transform` 操作符，您可以通过调用带有要发送值的 `emit` 函数将值发射到新的 Flow 中。

例如，这个 `MovieViewModel` 有一个 `fetchTopMovieTitles` 函数，它使用 `transform` 返回包含顶级电影的 Flow：

```kt
class MovieViewModel : ViewModel() {
```

```kt
    ...
```

```kt
    fun fetchTopMovies(): Flow<Movie> {
```

```kt
        return fetchMoviesFlow()
```

```kt
            .transform {
```

```kt
                if (it.popularity > 0.5f) emit(it)
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

在本例中，在电影的 Flow 中使用了 `transform` 操作符来返回一个新的 Flow。`transform` 操作符用于只发射流行度高于 `0.5` 的电影列表，这意味着超过 50% 的流行度。

您还可以使用大小限制操作符与 Flow 一起使用。以下是一些这些操作符：

+   `drop(x)`: 返回一个忽略前 *x* 个元素的 Flow

+   `dropWhile`: 返回一个忽略满足指定条件的第一个元素的 Flow

+   `take(x)`: 返回包含 Flow 的前 *x* 个元素的 Flow

+   `takeWhile`: 返回一个包含满足指定条件的第一个元素的 Flow

这些大小限制操作符的功能与 Kotlin 收集函数同名操作符类似。

在本节中，我们学习了中间流操作符。中间操作符将 Flow 转换为新的 Flow。在下一节中，我们将学习如何缓冲和组合 Kotlin 流。

# 缓冲和组合流

在本节中，我们将学习关于 Kotlin 流的缓冲和组合。您可以使用 Flow 操作符缓冲和组合 Flow。缓冲允许具有长时间运行任务的 Flow 独立运行并避免竞争条件。组合允许在处理或显示在屏幕上之前将不同的 Flow 源连接起来。

## 缓冲 Kotlin 流

在本节中，我们将学习关于 Kotlin 流的缓冲。缓冲允许您并行运行数据发射。

使用 Flow 运行时，数据发射和收集是顺序进行的。当发射新值时，它将被收集。只有当之前的数据被收集后，才能发射新值。如果从 Flow 发射或收集数据需要较长时间，整个过程将花费更长的时间。

使用缓冲，可以使 Flow 的数据发射和收集并行运行。您可以使用以下三个操作符来缓冲 Flow：

+   `buffer`

+   `conflate`

+   `collectLatest`

`buffer()` 允许 Flow 在数据仍在收集时发射值。发射和收集数据在单独的协程中运行，因此它是并行的。以下是如何使用 `buffer` 与 Flow 的示例：

```kt
class MainActivity : AppCompatActivity() {
```

```kt
    ...
```

```kt
    override fun onCreate(savedInstanceState: Bundle?) {
```

```kt
        ...
```

```kt
        lifecycleScope.launch {
```

```kt
            repeatOnLifecycle(Lifecycle.State.STARTED) {
```

```kt
                viewModel.fetchMovies()
```

```kt
                    .buffer()
```

```kt
                    .collect { movie ->
```

```kt
                        processMovie(movie)
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
    }
```

```kt
}
```

在这里，在调用 `collect` 之前添加了 `buffer` 操作符。如果收集中的 `processMovie(movie)` 函数执行时间较长，Flow 将在收集和处理之前发射并缓冲这些值。

`conflate()` 与 `buffer()` 操作符类似，但 `conflate` 中，收集器将只处理在处理完上一个值之后的最新发出的值。它将忽略之前发出的其他值。以下是在 Flows 中使用 `conflate` 的示例：

```kt
class MainActivity : AppCompatActivity() {
```

```kt
    ...
```

```kt
    override fun onCreate(savedInstanceState: Bundle?) {
```

```kt
        ...
```

```kt
        lifecycleScope.launch {
```

```kt
            repeatOnLifecycle(Lifecycle.State.STARTED) {
```

```kt
                viewModel.getTopMovie()
```

```kt
                    .conflate()
```

```kt
                    .collect { movie ->
```

```kt
                        processMovie(movie)
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
    }
```

```kt
}
```

在这个例子中，添加 `conflate` 操作符将允许我们只处理 Flows 的最新值，并使用该值调用 `processMovie`。

`collectLatest(action)` 是一个终端操作符，它将以与 `collect` 相同的方式收集 Flows，但每当发出新值时，它将重新启动操作并使用这个新值。以下是在 Flows 中使用 `collectLatest` 的示例：

```kt
class MainActivity : AppCompatActivity() {
```

```kt
    ...
```

```kt
    override fun onCreate(savedInstanceState: Bundle?) {
```

```kt
        ...
```

```kt
        lifecycleScope.launch {
```

```kt
            repeatOnLifecycle(Lifecycle.State.STARTED) {
```

```kt
                viewModel.getTopMovie()
```

```kt
                    .collectLatest { movie ->
```

```kt
                        displayMovie(movie)
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
    }
```

```kt
}
```

在这里，`collectLatest` 被用来代替 `collect` 终端操作符来收集来自 `viewModel.getTopMovie()` 的 Flows。每当这个 Flows 发出新值时，它将重新启动并使用新值调用 `displayMovie`。

在本节中，你学习了如何使用 `buffer`、`conflate` 和 `collectLatest` 缓冲 Kotlin Flows。在下一节中，你将学习如何将多个 Flows 合并成一个单一的 Flows。

## 合并 Flows

在本节中，我们将学习如何合并 Flows。Kotlin Flow API 提供了可用于合并多个 Flows 的操作符。

如果你有多条 Flows 并且想要将它们合并成一个，你可以使用以下 Flows 操作符：

+   `zip`

+   `merge`

+   `combine`

`merge` 是一个顶级函数，它将来自多个相同类型的 Flows 的元素合并成一个。你可以传递任意数量的 Flows 来进行合并。当你有两个或更多数据源需要先合并后再收集时，这非常有用。

在以下示例中，有两个 Flows 来自 `viewModel.fetchMoviesFromDb` 和 `viewModel.fetchMoviesFromNetwork`，它们使用 `merge` 进行合并：

```kt
class MainActivity : AppCompatActivity() {
```

```kt
    ...
```

```kt
    override fun onCreate(savedInstanceState: Bundle?) {
```

```kt
        ...
```

```kt
        lifecycleScope.launch {
```

```kt
            repeatOnLifecycle(Lifecycle.State.STARTED) {
```

```kt
                merge(viewModel.fetchMoviesFromDb(),
```

```kt
                  viewModel.fetchMoviesFromNetwork())
```

```kt
                    .collect { movie ->
```

```kt
                        processMovie(movie)
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
    }
```

```kt
}
```

在这个例子中，在收集之前，使用了 `merge` 来合并来自 `viewModel.fetchMoviesFromDb` 和 `viewModel.fetchMoviesFromNetwork` 的 Flows。

`zip` 操作符使用你指定的函数将第一个 Flows 的数据与第二个 Flows 的数据配对成一个新的值。如果一个 Flows 的值比另一个少，`zip` 将在处理完这个 Flows 的所有值后结束。

以下展示了如何使用 `zip` 操作符合并两个 Flows，`userFlow` 和 `taskFlow`：

```kt
class MainActivity : AppCompatActivity() {
```

```kt
    ...
```

```kt
    override fun onCreate(savedInstanceState: Bundle?) {
```

```kt
        ...
```

```kt
        lifecycleScope.launch {
```

```kt
            repeatOnLifecycle(Lifecycle.State.STARTED) {
```

```kt
                val userFlow = viewModel.getUsers()
```

```kt
                val taskFlow = viewModel.getTasks()
```

```kt
                userFlow.zip(taskFlow) { user, task ->
```

```kt
                    AssignedTask(user, task)
```

```kt
                }.collect { assignedTask ->
```

```kt
                    displayAssignedTask(assignedTask)
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
    }
```

```kt
}
```

在这个例子中，你使用了 `zip` 来将 `userFlow` 的每个值与 `taskFlow` 配对，并使用 `user` 和 `task` 的值返回一个 `AssignedTask` 的 Flows。这个新的 Flows 将被收集，然后使用 `displayAssignedTask` 函数显示。

`combine` 将第一个 Flows 的数据与第二个 Flows 的数据配对，就像 `zip` 一样，但使用每个 Flows 发出的最新值。只要 Flows 发出一个值，它就会继续运行。还有一个用于多个 Flows 的顶级 `combine` 函数。

以下示例展示了如何使用 `combine` 操作符在你的应用程序中将两个 Flows 连接起来：

```kt
class MainActivity : AppCompatActivity() {
```

```kt
    ...
```

```kt
    override fun onCreate(savedInstanceState: Bundle?) {
```

```kt
        ...
```

```kt
        lifecycleScope.launch {
```

```kt
            repeatOnLifecycle(Lifecycle.State.STARTED) {
```

```kt
                val yourMesssage =
```

```kt
                  viewModel.getLastMessageSent()
```

```kt
                val friendMessage =
```

```kt
                  viewModel.getLastMessageReceived()
```

```kt
                userFlow.combine(taskFlow) { yourMesssage,
```

```kt
                  friendMessage ->
```

```kt
                    Conversation(yourMessage,
```

```kt
                      friendMessage)
```

```kt
                }.collect { conversation ->
```

```kt
                    displayConversation(conversation)
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
    }
```

```kt
}
```

在这里，你有两个流，`yourMessage` 和 `friendMessage`。`combine` 函数将 `yourMessage` 和 `friendMessage` 的最新值配对，以创建一个 `Conversation` 对象。每当任一流发出新值时，`combine` 将配对最新值并将其添加到结果流中以进行收集。

在本节中，我们探讨了如何组合流。在下一节中，我们将重点关注 `StateFlow` 和 `SharedFlow` 以及如何在您的 Android 应用程序中使用它们。

# 探索 StateFlow 和 SharedFlow

在本节中，我们将深入了解 `StateFlow` 和 `SharedFlow`。`SharedFlow` 和 `StateFlow` 是热流，与默认为冷流的普通 Kotlin 流不同。

流是一个冷数据流。流仅在值被收集时发出值。使用 `SharedFlow` 和 `StateFlow` 热流，您可以在调用它们时立即运行和发出值，甚至在它们没有监听器时也是如此。`SharedFlow` 和 `StateFlow` 是流，因此您也可以在它们上使用操作符。

`SharedFlow` 允许您向多个监听器发出值。`SharedFlow` 可以用于一次性事件。`SharedFlow` 将执行的任务将只运行一次，并由监听器共享。

您可以使用 `MutableSharedFlow` 然后使用 `emit` 函数将值发送到所有收集器。

在以下示例中，`SharedFlow` 用于 `MovieViewModel` 中获取的电影列表：

```kt
class MovieViewModel : ViewModel() {
```

```kt
    private val _message = MutableSharedFlow<String>()
```

```kt
    val movies: SharedFlow<String> =
```

```kt
      _message.asSharedFlow()
```

```kt
    ...
```

```kt
    fun onError(): Flow<List<Movie>> {
```

```kt
        ...
```

```kt
        _message.emit("An error was encountered")
```

```kt
    }
```

```kt
}
```

在此示例中，我们使用了 `SharedFlow` 来处理消息。我们使用 `emit` 函数将错误消息发送到流的监听器。

`StateFlow` 是 `SharedFlow`，但它只向其监听器发出最新值。`StateFlow` 使用一个值（初始状态）初始化并保持此状态。您可以使用 `StateFlow` 的可变版本 `MutableStateFlow` 来更改 `StateFlow` 的值。更新值会将新值发送到流。

在 Android 中，`StateFlow` 可以作为 `LiveData` 的替代方案。您可以为 `ViewModel` 使用 `StateFlow`，然后您的活动或片段可以收集值。例如，在以下 `ViewModel` 中，使用了 `StateFlow` 来表示电影列表：

```kt
class MovieViewModel : ViewModel() {
```

```kt
    private val _movies =
```

```kt
      MutableStateFlow(emptyList<Movie>())
```

```kt
    val movies: StateFlow<List<Movie>> = _movies
```

```kt
    ...
```

```kt
    fun fetchMovies(): Flow<List<Movie>> {
```

```kt
        ...
```

```kt
        _movies.value = movieRepository.fetchMovies()
```

```kt
    }
```

```kt
}
```

在前面的代码中，从仓库获取的电影列表将被设置为 `_movies` 的 `MutableStateFlow`，这也会改变 `movies` 的 `StateFlow`。然后您可以在活动或片段中收集 `movies` 的 `StateFlow`，如下所示：

```kt
class MainActivity : AppCompatActivity() {
```

```kt
    ...
```

```kt
    override fun onCreate(savedInstanceState: Bundle?) {
```

```kt
        ...
```

```kt
        lifecycleScope.launch {
```

```kt
            repeatOnLifecycle(Lifecycle.State.STARTED) {
```

```kt
                viewModel.movies.collect { movies ->
```

```kt
                    displayMovies(movies)
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
    }
```

```kt
}
```

在这里，`viewModel.movies` 的 `StateFlow` 将被收集，然后使用 `displayMovies` 函数将电影列表显示在屏幕上。

在本节中，我们学习了 `StateFlow` 和 `SharedFlow` 以及我们如何在 Android 项目中使用它们。

让我们尝试到目前为止所学的内容，通过将 Kotlin Flow 添加到 Android 项目中。

# 练习 5.01 – 在 Android 应用中使用 Kotlin Flow

对于这个练习，你将继续在*Exercise 4.01 – 在 Android 应用程序中添加协程测试*中工作的电影应用程序。这个应用程序显示当前正在电影院上映的电影。你将通过以下步骤将 Kotlin Flow 添加到项目中：

1.  打开你在*Exercise 4.01 – 在 Android 应用程序中添加协程测试*中工作的电影应用程序。在 Android Studio 中。

1.  前往`MovieRepository`类，添加一个新的`fetchMoviesFlow()`函数，使用`flow`构建器返回一个 Flow，并发出来自`MovieService`的电影列表，如下面的代码片段所示：

    ```kt
    fun fetchMoviesFlow(): Flow<List<Movie>> {
        return flow {
            emit(movieService.getMovies(apiKey).results)
        }.flowOn(Dispatchers.IO)
    }
    ```

这与`fetchMovies()`函数相同，但这个函数使用 Kotlin Flow，并将返回`Flow<List<Movie>>`给将收集它的函数或类。Flow 将从`movieService.getMovies`发出电影列表，并在`Dispatchers.IO`调度器上流动。

1.  打开`MovieViewModel`类，将获取自`movieRepository`的`movies` `LiveData`的初始化替换为以下行：

    ```kt
    private val _movies =
      MutableStateFlow(emptyList<Movie>())
    val movies: StateFlow<List<Movie>> = _movies
    ```

这将允许你使用`_movies` `MutableStateFlow`的值作为`movies` `StateFlow`的值，你将在从`movieRepository`的 Flow 中获取电影列表后稍后更改它。

1.  对`error` `LiveData`做同样的处理，并用以下行替换其初始化，从`movieRepository`获取的值：

    ```kt
    private val _error = MutableStateFlow("")
    val error: StateFlow<String> = _error
    ```

这将使用`_error` `MutableStateFlow`的值来设置`error` `StateFlow`。你将在稍后能够更改这个`StateFlow`的值，以处理 Flow 遇到异常的情况。

1.  用以下行替换`loading`和`_loading`变量：

    ```kt
    private val _loading = MutableStateFlow(true)
    val loading: StateFlow<String> = _loading
    ```

这将使用`_loading` `MutableStateFlow`的值来设置`loading` `StateFlow`。你将在稍后更新这个值以指示电影加载正在进行。

1.  删除`fetchMovies()`函数及其内容。你将在下一步替换它。

1.  添加一个新的`fetchMovies()`函数，它将收集来自`movieRepository.fetchMoviesFlow`的 Flow，如下面的代码块所示：

    ```kt
    fun fetchMovies() {
        _loading.value = true
        viewModelScope.launch (dispatcher) {
            MovieRepository.fetchMoviesFlow()
                .collect {
                    _movies.value = it
                    _loading.value = false
                }
        }
    }
    ```

这将收集来自`movieRepository.fetchMoviesFlow`的电影列表，并将其设置为`_movies` `MutableStateFlow`和`movies` `StateFlow`。然后，这个电影列表将在`MainActivity`中显示。

1.  打开`app/build.gradle`文件。在依赖项中添加以下行：

    ```kt
    implementation 'androidx.lifecycle:lifecycle-runtime-ktx:2.4.1'
    ```

这将允许我们在`MainActivity`稍后使用`lifecycleScope`收集 flows。

1.  打开`MainActivity`，删除观察`movies`、`error`和`loading` `LiveData`的代码行。用以下代码替换它们：

    ```kt
    lifecycleScope.launch {
       repeatOnLifecycle(Lifecycle.State.STARTED) {
           launch {
               movieViewModel.movies.collect { movies ->
                   movieAdapter.addMovies(movies)
               }
           }
           launch {
               movieViewModel.error.collect { error ->
                   if (error.isNotEmpty())
                     Snackbar.make(recyclerView, error, 
                     Snackbar.LENGTH_LONG).show()
               }
           }
           launch {
               movieViewModel.loading.collect { loading ->
                   progressBar.isVisible = loading
               }
           }
       }
    }
    ```

这将收集`movies`并将它们添加到列表中，收集`error`并在`error`不为空时显示`SnackBar`消息，并收集`loading`并根据其值更新`progressBar`。

1.  运行应用程序。应用程序应该仍然显示电影列表（带有海报和标题），如下面的截图所示：

![图 5.1 – 包含电影列表的电影应用](img/Figure_5.1_B17773.jpg)

图 5.1 – 包含电影列表的电影应用

在这个练习中，我们通过创建一个返回电影列表作为 Flow 的 `MovieRepository` 函数，将 Kotlin Flow 添加到 Android 应用中。然后，这个 Flow 被收集到 `MovieViewModel` 中。

# 摘要

本章重点介绍了在 Android 中使用 Kotlin Flows 进行异步编程。Flows 是建立在 Kotlin 协程之上的。一个 Flow 可以按顺序发出多个值，而不仅仅是单个值。

我们从学习如何在您的 Android 应用中使用 Kotlin Flows 开始。Jetpack 库如 Room 和一些第三方库支持 Flow。为了在 UI 层安全地收集 Flows、防止内存泄漏和避免资源浪费，您可以使用 `Lifecycle.repeatOnLifecycle` 和 `Flow.flowWithLifecycle`。

我们随后转向使用 Flow 构建器创建 Flows。`flowOf` 函数创建一个发出您提供的值或 `vararg` 值的 Flow。您可以使用 `asFlow()` 扩展函数将集合和函数类型转换为 Flow。`flow` 构建器函数从挂起 lambda 块中创建一个新的 Flow，在其中您可以使用 `emit()` 发送值。

然后，我们探讨了 Flow 操作符，并学习了如何与 Kotlin Flows 一起使用它们。使用终端操作符，您可以开始 Flow 的收集。中间操作符允许您将一个 Flow 转换为另一个 Flow。

我们随后学习了缓冲和组合 Flows。使用 `buffer`、`conflate` 和 `collectLatest` 操作符，您可以缓冲 Flows。您可以使用 `merge`、`zip` 和 `combine` Flow 操作符组合 Flows。

我们随后探讨了 `SharedFlow` 和 `StateFlow`。这些可以在您的 Android 项目中使用。使用 `SharedFlow`，您可以向多个监听器发出值。`StateFlow` 是只向其监听器发出最新值的 `SharedFlow`。

最后，我们进行了一个练习，将 Kotlin Flows 添加到 Android 应用程序中。我们在 `MovieRepository` 中使用了一个 Flow，然后它在 `MovieViewModel` 中被收集。

在下一章中，我们将关注如何在您的应用程序中处理 Kotlin Flows 的取消和异常。
