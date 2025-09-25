# 第六章：*第六章*：处理 Flow 取消和异常

在上一章中，我们专注于 Kotlin Flows，并学习了如何在我们的 Android 项目中使用它们。我们学习了如何使用 Flow builders 创建 Kotlin Flows。然后，我们探讨了 Flow operators 以及如何与 Kotlin Flows 一起使用它们。接着，我们学习了缓冲和组合 Flows。最后，我们探讨了 `SharedFlow` 和 `StateFlow`。

Flows 可以被取消，也可能失败或遇到异常。开发者必须能够正确处理这些问题，以防止应用程序崩溃，并通过对话框或吐司消息通知用户。我们将在本章中讨论如何完成这些任务。

在本章中，我们将首先理解 Flow 取消。我们将学习如何取消 Flows 并处理 Flows 的取消。然后，我们将学习如何管理 Flows 中可能发生的失败和异常。我们还将学习关于重试和处理 Flow 完成的情况。

在本章中，我们将涵盖以下主要主题：

+   取消 Kotlin Flows

+   使用 Flow 重试任务

+   在 Flows 中捕获异常

+   处理流完成

到本章结束时，您将了解如何取消 Flows，并学习如何管理取消以及如何在 Flows 中处理异常。

# 技术要求

您需要下载并安装最新版本的 Android Studio。您可以在[`developer.android.com/studio`](https://developer.android.com/studio)找到最新版本。为了获得最佳学习体验，建议使用以下配置的计算机：

+   英特尔酷睿 i5 或等效或更高配置

+   至少 4 GB RAM

+   至少 4 GB 可用空间

本章的代码示例可以在 GitHub 上找到，地址为 [`github.com/PacktPublishing/Simplifying-Android-Development-with-Coroutines-and-Flows/tree/main/Chapter06`](https://github.com/PacktPublishing/Simplifying-Android-Development-with-Coroutines-and-Flows/tree/main/Chapter06)

# 取消 Kotlin Flows

在本节中，我们将首先探讨 Kotlin Flow 的取消。与协程一样，Flows 也可以手动或自动取消。

在*第三章*，“处理协程取消和异常”中，我们学习了如何取消协程，以及协程取消必须是合作的。由于 Kotlin Flows 是建立在协程之上的，Flow 遵循协程的协作取消。

使用 `flow{}` 构建器创建的 Flows 默认可取消。每次向 Flow 发送新值的 `emit` 调用也会内部调用 `ensureActive`。这会检查协程是否仍然处于活动状态，如果不是，它将抛出 `CancellationException`。

例如，我们可以使用 `flow{}` 构建器创建一个可取消的 Flow，如下所示：

```java
class MovieViewModel : ViewModel() {
```

```java
    ...
```

```java
    fun fetchMovies(): Flow<Movie> = flow {
```

```java
        movieRepository.fetchMovies().forEach {
```

```java
            emit(it)
```

```java
        }
```

```java
    }
```

```java
}
```

在这里的 `fetchMovies` 函数中，我们使用了 `flow` 构建器来创建由 `movieRepository.fetchMovies` 返回的电影 Flows。这个 `Flow<Movie>` 默认将是一个可取消的 Flow。

所有其他 Flow，如使用`asFlow`和`flowOf` Flow 构建器创建的 Flow，默认情况下不可取消。我们必须自己处理取消。有一个`cancellable()`操作符可以用于 Flow，使其可取消。这将添加一个`ensureActive`调用到每个新值的发射。

以下示例展示了我们如何使用`cancellable` Flow 操作符使 Flow 可取消：

```java
class MovieViewModel : ViewModel() {
```

```java
    ...
```

```java
    fun fetchMovies(): Flow<Movie> {
```

```java
        return movieRepository.fetchMovies().cancellable()
```

```java
    }
```

```java
}
```

在这个示例中，我们使用了`movieRepository.fetchMovies()`返回的 Flow 上的可取消操作符，使结果 Flow 可取消。

在本节中，我们学习了如何取消 Kotlin Flows 以及如何确保你的 Flows 可取消。在下一节中，我们将关注如何使用 Kotlin Flows 重试你的任务。

# 使用 Flow 重试任务

在本节中，我们将探讨 Kotlin Flow 的重试。在某些情况下，重试操作对于你的应用程序是必要的。

在执行长时间运行的任务，如网络调用时，有时需要再次尝试调用。这包括登录/注销、发布数据，甚至获取数据的情况。用户可能处于网络连接低下的区域，或者可能存在其他导致调用失败的因素。使用 Kotlin Flows，我们有`retry`和`retryWhen`操作符，可以用来自动重试 Flows。

`retry`操作符允许你设置`retries`作为 Flow 重试的最大次数。你也可以设置一个`predicate`条件，一个当返回`true`时将重试 Flow 的代码块。`predicate`有一个**Throwable**参数，代表发生的异常；你可以使用它来检查是否需要进行重试。

以下示例展示了我们如何使用`retry` Flow 操作符来重试我们的 Flow 中的任务：

```java
class MovieViewModel : ViewModel() {
```

```java
    ...
```

```java
    fun favoriteMovie(id: Int) =
```

```java
        movieRepository.favoriteMovie(id)
```

```java
            .retry(3) { cause -> cause is IOException }
```

```java
}
```

在这里，当遇到`IOException`异常时，`movieRepository.favoriteMovie(id)`的 Flow 将会重试最多三次。

如果你没有为重试传递值，将使用默认值`Long.MAX_VALUE`。当未提供`predicate`时，默认值为`true`，这意味着如果尚未达到`retries`，Flow 将始终重试。

`retryWhen`操作符类似于`retry`操作符。我们需要指定`predicate`，这是条件，只有当`true`时才会执行重试。`predicate`有`true`，将重试 Flow。以下代码展示了使用`retryWhen`在 Flow 中重试任务的示例：

```java
class MovieViewModel : ViewModel() {
```

```java
    ...
```

```java
    fun favoriteMovie(id: Int) =
```

```java
        movieRepository.favoriteMovie(id)
```

```java
            .retryWhen { cause, attempt -> attempt <3 &&
```

```java
                cause is IOException }
```

```java
}
```

在这个示例中，我们使用了`retryWhen`并指定了当`attempt`的值小于三且异常为`IOException`时进行重试。

使用`retryWhen`操作符，我们还可以向 Flow（使用`emit`函数）发出一个值，我们可以使用它来表示重试尝试或一个值。然后我们可以显示这个值或在屏幕上处理它。以下示例展示了我们如何使用`emit`与`retryWhen`操作符：

```java
class MovieViewModel : ViewModel() { 
```

```java
    ... 
```

```java
    fun getTopMovieTitle(): Flow<String> { 
```

```java
        return movieRepository.getTopMovieTitle(id) 
```

```java
            .retryWhen { cause, attempt -> 
```

```java
                emit("Fetching title again...")
```

```java
                attempt <3 && cause is IOException 
```

```java
            }
```

```java
            ...
```

```java
}
```

在这里，当尝试次数少于三次时，如果异常是活动或片段可以处理的`Fetching title again`字符串，那么 Flow 的任务将会重试。

在本节中，你学习了如何使用 Kotlin Flow 重试网络请求等任务。在下一节中，我们将探讨 Kotlin Flow 异常以及如何更新我们的代码来捕获这些异常。

# 流中的异常捕获

当你的代码中的 Flow 被取消或收集值时，可能会遇到`CancellationException`或其他异常。在本节中，我们将学习如何处理这些 Kotlin Flow 异常。

在收集值或使用 Flow 上的任何操作符时，Flows 可能会发生异常。我们可以通过在代码中将 Flow 的收集用`try-catch`块包围来处理 Flows 中的异常。例如，在以下代码中，`try-catch`块被用来添加异常处理：

```java
class MainActivity : AppCompatActivity() {
```

```java
 ...
```

```java
 override fun onCreate(savedInstanceState: Bundle?) {
```

```java
     ...
```

```java
     lifecycleScope.launch {
```

```java
         repeatOnLifecycle(Lifecycle.State.STARTED) {
```

```java
             try {
```

```java
                 viewModel.fetchMovies().collect { movie ->
```

```java
                     processMovie(movie)
```

```java
                 }
```

```java
             } catch (exception: Exception) {
```

```java
                 Log.e("Error", exception.message)
```

```java
             }  
```

```java
         }
```

```java
     }
```

```java
 }
```

```java
}
```

在这里，`viewModel.fetchMovies`返回的 Flow 的收集代码被包裹在一个`try-catch`块中。如果在 Flow 中遇到异常，异常消息将使用`Error`标签和`exception.message`作为消息进行记录。

我们还可以使用`catch` Flow 操作符来处理我们的 Flow 中的异常。使用`catch`操作符，我们可以捕获来自上游 Flow 的异常，或者是在调用`catch`操作符之前的功能和操作符。

在以下示例中，`catch`操作符被用来捕获`viewModel.fetchMovies`返回的 Flow 中的异常：

```java
class MainActivity : AppCompatActivity() {
```

```java
  ...
```

```java
  override fun onCreate(savedInstanceState: Bundle?) {
```

```java
      ...
```

```java
      lifecycleScope.launch {
```

```java
          repeatOnLifecycle(Lifecycle.State.STARTED) {
```

```java
              viewModel.fetchMovies()
```

```java
                  .catch { exception ->
```

```java
                      handleException(exception) }
```

```java
                  .collect { movie -> processMovie(movie) }
```

```java
            }
```

```java
        }
```

```java
    }
```

```java
}
```

在这里，`catch`操作符被用于 Flow 中捕获异常。该异常是一个将要处理异常的`handleException`函数的实例。

我们还可以使用`catch`操作符来发出一个新值来表示错误或用作备用值，例如一个空列表。在以下示例中，当 Flow 返回顶级电影标题时发生异常，将使用默认字符串值`No Movie Fetched`：

```java
class MainActivity : AppCompatActivity() {
```

```java
  ...
```

```java
  override fun onCreate(savedInstanceState: Bundle?) {
```

```java
      ...
```

```java
      lifecycleScope.launch {
```

```java
          repeatOnLifecycle(Lifecycle.State.STARTED) {
```

```java
              viewModel.getTopMovieTitle()
```

```java
                  .catch { emit("No Movie Fetched") }
```

```java
                  .collect { title -> displayTitle(title) }
```

```java
          }
```

```java
      }
```

```java
  }
```

```java
}
```

在此示例中，我们使用`catch`操作符在从`ViewModel`获取顶级电影标题时发生异常时发出`No Movie Fetched`字符串。这将是在`displayTitle()`调用中使用的值。

由于`catch`操作符仅处理上游 Flow 中的异常，因此在`collect{}`调用期间发生的异常不会被捕获。虽然你可以使用`try-catch`块来处理这些异常，但你也可以将收集代码移动到`onEach`操作符中，在其后添加`catch`操作符，并使用`collect()`来开始收集。

以下示例显示了当使用`onEach`操作符进行值收集和`catch`操作符处理异常时，你的代码可能看起来像什么：

```java
class MainActivity : AppCompatActivity() {
```

```java
  ...
```

```java
  override fun onCreate(savedInstanceState: Bundle?) {
```

```java
      ...
```

```java
      lifecycleScope.launch {
```

```java
          repeatOnLifecycle(Lifecycle.State.STARTED) {
```

```java
              viewModel.fetchMovies()
```

```java
                  .onEach { movie -> processMovie(movie) }
```

```java
                  .catch { exception ->
```

```java
                      handleError(exception) }
```

```java
                  .collect()
```

```java
          }
```

```java
      }
```

```java
  }
```

```java
}
```

在这里，使用了不带参数的`collect()`函数，`onEach`操作符将处理 Flow 中的每一部电影。

在本节中，我们学习了如何在 Flows 中捕获异常。在下一节中，我们将重点关注 Kotlin Flow 的完成。

# 处理 Flow 完成

在本节中，我们将探讨如何处理 Flow 的完成。我们可以在我们的 Flows 完成后添加代码来执行额外的任务。

当 Flow 遇到异常时，它将被取消并完成 Flow。当 Flow 的最后一个元素被发出时，Flow 也会完成。

要在 Flow 完成时在你的 Flow 中添加监听器，你可以使用 `onCompletion` 操作符并添加当 Flow 完成时将运行的代码块。`onCompletion` 的一个常见用法是在 Flow 完成时隐藏你的 UI 中的 **ProgressBar**，如下面的代码所示：

```java
class MainActivity : AppCompatActivity() {
```

```java
  ...
```

```java
  override fun onCreate(savedInstanceState: Bundle?) {
```

```java
      ...
```

```java
      lifecycleScope.launch {
```

```java
          repeatOnLifecycle(Lifecycle.State.STARTED) {
```

```java
              viewModel.fetchMovies()
```

```java
                  .onStart { progressBar.isVisible = true }
```

```java
                  .onEach { movie -> processMovie(movie) }
```

```java
                  .onCompletion { progressBar.isVisible =
```

```java
                      false }
```

```java
                  .catch { exception ->
```

```java
                      handleError(exception) }
```

```java
                  .collect()
```

```java
          }
```

```java
      }
```

```java
  }
```

```java
}
```

在这个例子中，我们向 Flow 中添加了 `onCompletion` 操作符来在 Flow 完成时隐藏 `progressBar`。我们还使用了 `onStart` 来显示 `progressBar`。

`onStart` 操作符是 `onCompletion` 的对立面。它将在 Flow 开始发出值之前被调用。在之前的示例中，使用了 `onStart` 以确保在 Flow 开始之前，`progressBar` 将显示在屏幕上。

在你添加到 `onStart` 和 `onCompletion`（如果 Flow 成功完成且没有异常）的代码块中，你也可以发出值，例如初始值和最终值。在以下示例中，使用了 `onStart` 操作符来发出一个初始值，该值将在屏幕上显示：

```java
class MainActivity : AppCompatActivity() {
```

```java
  ...
```

```java
  override fun onCreate(savedInstanceState: Bundle?) {
```

```java
      ...
```

```java
      lifecycleScope.launch {
```

```java
          repeatOnLifecycle(Lifecycle.State.STARTED) {
```

```java
              viewModel.getTopMovieTitle()
```

```java
                  .onStart { emit("Loading...") }
```

```java
                  .catch { emit("No Movie Fetched") }
```

```java
                  .collect { title -> displayTitle(title) }
```

```java
          }
```

```java
      }
```

```java
  }
```

```java
}
```

在这里，`onStart` 用于监听 Flow 开始时的情况。当 Flow 开始时，它将发出一个 `Loading…` 字符串作为 Flow 的初始值。这将随后成为屏幕上显示的第一个条目。

`onCompletion` 代码块还有一个可空的 `catch`，异常本身不会被处理，所以你仍然需要使用 `catch` 或 `try-catch` 来处理这个异常。

以下示例显示了我们可以如何使用这个可空的 `onCompletion` 调用：

```java
class MainActivity : AppCompatActivity() {
```

```java
 ...
```

```java
 override fun onCreate(savedInstanceState: Bundle?) {
```

```java
     ...
```

```java
     lifecycleScope.launch {
```

```java
         repeatOnLifecycle(Lifecycle.State.STARTED) {
```

```java
             viewModel.getTopMovieTitle()
```

```java
                 .onCompletion { cause ->
```

```java
                     progressBar.isVisible = false
```

```java
                     if (cause != null) displayError(cause)
```

```java
                 }
```

```java
                 .catch { emit("No Movie Fetched") }
```

```java
                 .collect { title -> displayTitle(title) }
```

```java
         }
```

```java
     }
```

```java
 }
```

```java
}
```

在这个例子中，我们在 `onCompletion` 块中检查了原因，如果它不为空（这意味着遇到了异常），则将调用 `displayError` 并将原因传递给它。

在本节中，我们学习了如何使用 `onStart` 和 `onCompletion` 来处理 Flows 开始和完成的情况。

让我们尝试将你所学到的知识应用到在 Android 项目中处理 Flows 可能发生的异常的代码中。

# 练习 6.01 – 在 Android 应用中处理 Flow 异常

在这个练习中，你将继续使用你在 *练习 5.01 – 在 Android 应用中使用 Kotlin Flow* 中工作的电影应用。这个应用显示正在电影院上映的电影。你将通过以下步骤更新项目以处理 Flow 取消和异常：

1.  在 Android Studio 中，打开你在 *练习 5.01 – 在 Android 应用中使用 Kotlin Flow* 中工作的电影应用。

1.  前往 `MovieViewModel` 类。在 `fetchMovies` 函数中，删除设置 `_loading` 值为 `true` 的行。你的函数将看起来像以下这样：

    ```java
    fun fetchMovies() {
        viewModelScope.launch (dispatcher) {
            MovieRepository.fetchMoviesFlow()
                .collect {
                    _movies.value = it
                    _loading.value = false
                }
        }
    }
    ```

您已移除设置 `loading` 为 `true`（并在屏幕上显示 `ProgressBar`）的代码。它将在下一步被 `onStart` Flow 操作符所替代。

1.  在 `collect` 调用之前添加一个 `onStart` 操作符，当 Flow 开始时，它将 `_loading` 的值设置为 `true`，如下所示：

    ```java
    fun fetchMovies() {
        viewModelScope.launch (dispatcher) {
            MovieRepository.fetchMoviesFlow()
                .onStart { _loading.value = true }
                .collect {
                    _movies.value = it
                    _loading.value = false
                }
        }
    }
    ```

`onStart` 操作符将在 Flow 开始时将 `_loading` 的值设置为 `true` 并在屏幕上显示 `ProgressBar`。

1.  接下来，从 `collect` 调用内部的代码块中移除设置 `_loading` 为 `false` 的行。您的函数将如下所示：

    ```java
    fun fetchMovies() {
        viewModelScope.launch (dispatcher) {
            MovieRepository.fetchMoviesFlow()
                .onStart { _loading.value = true }
                .collect {
                    _movies.value = it
                }
        }
    }
    ```

您已移除在 Flow 收集时将 `_loading` 的值设置为 `false` 并在屏幕上隐藏 `ProgressBar` 的代码。

1.  在 `collect` 调用之前添加一个 `onCompletion` 操作符，当 Flow 完成时，它将 `_loading` 的值设置为 `false`，如下所示：

    ```java
    fun fetchMovies() {
        viewModelScope.launch (dispatcher) {
            MovieRepository.fetchMoviesFlow()
                .onStart { _loading.value = true }
                .onCompletion { _loading.value = false }
                .collect {
                    _movies.value = it
                }
        }
    }
    ```

`onCompletion` Flow 操作符将 `_loading` 的值设置为 `false`。这将隐藏屏幕上显示的 `ProgressBar`，在获取电影时显示。

1.  在 `collect` 函数之前添加一个 `catch` 操作符，以处理 Flow 遇到异常的情况：

    ```java
    fun fetchMovies() {
        viewModelScope.launch (dispatcher) {
            MovieRepository.fetchMoviesFlow()
                .onStart { _loading.value = true }
                .onCompletion { _loading.value = false }
                .catch {
                    _error.value = "An exception occurred:
                      ${it.message}"
                }
                .collect {
                    _movies.value = it
                }
        }
    }
    ```

这将设置一个包含 `An exception occurred:` 和异常信息的字符串，并将其作为 `_error` LiveData 的值。这个 `_error` LiveData 将在 `MainActivity` 中显示错误信息。

1.  在您的设备或模拟器上关闭 Wi-Fi 和移动数据。然后运行应用程序。这将导致获取电影时出现错误，因为没有互联网连接。应用程序将显示一个 `SnackBar` 消息，如下面的截图所示：

![Figure 6.1 – 电影应用中显示的错误信息

![img/Figure_6.1_B17773_new.jpg]

图 6.1 – 电影应用中显示的错误信息

1.  关闭应用程序，并在您的设备或模拟器上开启 Wi-Fi 和/或移动数据。再次运行应用程序。应用程序应显示 `ProgressBar`，在屏幕上显示电影列表（包括电影标题和海报），并在完成后隐藏 `ProgressBar`，如下面的截图所示：

![Figure 6.2 – 包含电影列表的电影应用

![img/Figure_6.2_B17773.jpg]

图 6.2 – 包含电影列表的电影应用

在这个练习中，您已更新应用程序，使其能够处理 Flow 中的异常而不是崩溃。

# 摘要

本章重点介绍了 Kotlin Flow 的取消操作。您了解到 Flows 遵循协程的协作取消。`flow{}` 构建器和 `StateFlow` 以及 `SharedFlow` 实现默认可取消。您可以使用 `cancellable` 操作符使其他 Flows 可取消。

我们接着学习了使用 Kotlin Flow 重试任务。您可以使用 `retry` 和 `retryWhen` 函数根据尝试次数和 Flow 遇到的异常来重试 Flow。

然后，我们学习了在 Flow 中的数据发射或收集过程中可能发生的异常处理。你可以使用`try-catch`块或`catch` Flow 操作符来处理 Flow 异常。

我们学习了如何处理 Flow 的完成。使用`onStart`和`onCompletion`操作符，你可以在 Flows 开始和结束时监听并运行代码。你还可以使用`onStart`和`onCompletion`代码块来发射值，例如当你想要为 Flow 设置初始值和最终值时。

最后，我们进行了一个练习来更新我们的 Android 项目并处理在 Flow 中可能遇到的异常。我们使用了`catch` Flow 操作符来处理项目中的异常。

在下一章中，我们将深入探讨在 Android 项目中创建和运行 Kotlin Flows 的测试。
