# 第七章：*第七章*：测试 Kotlin 流

在上一章中，我们专注于理解 Kotlin 流的取消，学习如何使流可取消，并处理取消。我们还学习了使用流重试任务以及处理流中的完成和异常。

为您的代码中的 Kotlin 流添加测试是应用程序开发的重要部分。测试将确保我们添加到项目中的流没有错误或错误，并且它们将按预期工作。它们可以使开发应用程序更容易，并帮助您自信地重构和维护代码。

在本章中，我们将学习如何在 Android 中测试 Kotlin 流。首先，我们将了解如何为测试流设置您的 Android 项目。然后，我们将继续创建和运行 Kotlin 流的测试。

本章涵盖了以下主要内容：

+   为测试流设置 Android 项目

+   测试 Kotlin 流

+   使用 Turbine 测试流

到本章结束时，您将了解 Kotlin 流测试。您将能够为您的 Android 应用程序中的流编写和运行单元和集成测试。

# 技术要求

您需要下载并安装最新版本的 Android Studio。您可以在 [`developer.android.com/studio`](https://developer.android.com/studio) 找到最新版本。为了获得最佳学习体验，建议使用以下规格的计算机：

+   英特尔酷睿 i5 或更高版本

+   至少 4 GB 的 RAM

+   可用空间至少 4 GB

本章的代码示例可以在 GitHub 上找到，网址为 [`github.com/PacktPublishing/Simplifying-Android-Development-with-Coroutines-and-Flows/tree/main/Chapter07`](https://github.com/PacktPublishing/Simplifying-Android-Development-with-Coroutines-and-Flows/tree/main/Chapter07)。

# 为测试流设置 Android 项目

在本节中，我们将首先了解如何为测试 Kotlin 流设置我们的 Android 项目。一旦我们完成了这个，添加我们项目中流的单元和集成测试将变得容易。

要在 Android 中创建单元测试，您的项目必须具有 JUnit 4 测试库，这是 Java 的单元测试框架。在 Android Studio 中创建的新项目应该已经将此添加到 `app/build` 依赖项中。如果您的项目还没有 JUnit，您可以通过在 `app/build.gradle` 依赖项中添加以下内容来添加它：

```kt
dependencies {
```

```kt
    …
```

```kt
    testImplementation 'junit:junit:4.13.2'
```

```kt
}
```

将此添加到您的依赖项中，您可以使用 JUnit 4 测试框架对您的代码进行单元测试。

使用模拟对象进行测试也是一个好主意。Mockito 是一个流行的 Java 模拟库，您可以在 Android 上使用它。您还可以使用 Mockito-Kotlin 来使用 Mockito 与惯用的 Kotlin 代码一起使用。要将 Mockito 和 Mockito-Kotlin 添加到您的 Android 测试中，您可以在 `app/build.gradle` 依赖项中添加以下内容：

```kt
dependencies {
```

```kt
    …
```

```kt
    testImplementation 'org.mockito:mockito-core:4.0.0'
```

```kt
    testImplementation 'org.mockito.kotlin:mockito-
```

```kt
      kotlin:4.0.0'
```

```kt
}
```

这将允许你使用 Mockito 通过 Kotlin 代码创建用于 Android 测试的模拟对象。Mockito-Kotlin 依赖于 `mockito-core` 和 `mockito-kotlin`：

```kt
dependencies {
```

```kt
    …
```

```kt
    testImplementation 'org.mockito.kotlin:mockito-
```

```kt
      kotlin:4.0.0'
```

```kt
}
```

由于 Kotlin Flow 是建立在协程之上的，你可以使用 `kotlinx-coroutines-test` 库来帮助你添加对协程和 Flows 的测试。这个库包含了一些实用类，可以让你更容易地编写测试。要将它添加到你的项目中，你可以在 `app/build.gradle` 的依赖项中添加以下内容：

```kt
dependencies {
```

```kt
    …
```

```kt
    testImplementation 'org.jetbrains.kotlinx:kotlinx-
```

```kt
      coroutines-test:1.6.0'
```

```kt
}
```

添加这个库允许你在项目中使用 `kotlinx-coroutines-test` 库来测试协程和 Flows。

在本节中，我们学习了如何设置我们的 Android 项目以测试 Kotlin 流。在下一节中，我们将学习如何测试 Kotlin 流。

# 测试 Kotlin 流

在本节中，我们将专注于测试使用 Flow 的 Kotlin 流。我们可以为使用 Flow 的类，如 `ViewModel`，创建单元和集成测试。

要测试收集 Flow 的代码，你可以使用可以返回值的模拟对象来进行断言检查。例如，如果你的 `ViewModel` 监听来自存储库的 Flow，你可以创建一个自定义的 `Repository` 类，该类发出一个包含预定义值的 Flow，以便更容易地进行测试。

例如，假设你有一个 `MovieViewModel` 类，如下所示，它有一个返回 Flow 的 `fetchMovies` 函数：

```kt
class MovieViewModel(private val movieRepository:
```

```kt
  MovieRepository) {
```

```kt
    ...
```

```kt
    suspend fun fetchMovies() {
```

```kt
        movieRepository.fetchMovies().collect {
```

```kt
            _movies.value = it
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

在这里，`fetchMovies` 函数从 `movieRepository.fetchMovies()` 收集一个 Flow。你可以通过创建 `MovieRepository` 并返回一组特定的值来为这个 `MovieViewModel` 编写测试，然后检查这些值是否与 `MovieViewModel` 中的 `movies` `LiveData` 设置的值相同。这个实现的示例如下：

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
        val list = listOf(movie1, movie2)
```

```kt
        val expected = MutableLiveData<List<Movie>>()
```

```kt
        expectedMovies.value = list
```

```kt
        val movieRepository: MovieRepository = mock {
```

```kt
            onBlocking { fetchMoviesFlow() } doReturn
```

```kt
              flowOf(movies)
```

```kt
        }
```

```kt
        val dispatcher = StandardTestDispatcher()
```

```kt
        val movieViewModel =
```

```kt
          MovieViewModel(movieRepository, dispatcher)
```

```kt
        runTest {
```

```kt
            movieViewModel.fetchMovies()
```

```kt
            dispatcher.scheduler.advanceUntilIdle()
```

```kt
            assertEquals(expectedMovies.value,
```

```kt
              movieViewModel.movies.value)
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

在这个示例中，`MovieRepository` 的 `fetchMoviesFlow` 返回一个只有一个项目的电影列表。在调用 `movieViewModel.fetchMovies()` 之后，测试检查 `MovieViewModel.movies` `LiveData` 中的值是否被设置为这个列表。

你也可以通过收集它到另一个对象来测试一个 Flow。你可以通过将 Flow 转换为列表（使用 `toList()`）或集合（使用 `toSet()`），获取第一个元素（使用 `first`），获取元素（使用 `take()`）和其他终端操作来实现。然后，你可以检查返回的值与预期值是否一致。

例如，假设你有一个 `MovieViewModel`，它有一个返回 Flow 的函数，如下面的类所示：

```kt
class MovieViewModel(private val movieRepository:
```

```kt
  MovieRepository) {
```

```kt
    ...
```

```kt
    fun fetchFavoriteMovies(): Flow<List<Movie>> {
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

在这里，`fetchFavoriteMovies` 函数返回一个 `List<Movie>` 的 Flow。你可以通过将 `Flow<List<Movie>>` 转换为列表来为这个函数编写测试，如下面的示例所示：

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
    fun fetchFavoriteMovies() {
```

```kt
        ...
```

```kt
        val expectedList = listOf(movie1, movie2)
```

```kt
        val movieRepository: MovieRepository = mock {
```

```kt
            onBlocking { fetchFavoriteMovies() } doReturn
```

```kt
              flowOf(expectedList)
```

```kt
        }
```

```kt
        val movieViewModel =
```

```kt
          MovieViewModel(movieRepository)
```

```kt
        runTest {
```

```kt
            ...
```

```kt
            assertEquals(expectedList,
```

```kt
             movieViewModel.fetchFavoriteMovies().toList())
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

在这个示例中，你将 `movieViewModel.fetchFavoriteMovies()` 返回的电影列表 Flow 转换为电影列表，并与预期的列表进行比较。

要测试 Flow 中的错误处理，你可以模拟你的测试对象以抛出异常。然后你可以检查抛出的异常或处理它的代码。以下示例展示了如何为 Flow 的失败情况编写测试：

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
    fun fetchMoviesFlowWithError() {
```

```kt
        val movieService: MovieService = mock {
```

```kt
            onBlocking { getMovies(anyString()) } doThrow
```

```kt
              IOException(exception)
```

```kt
        }
```

```kt
        val movieRepository = MovieRepository(movieService)
```

```kt
        runTest {
```

```kt
            movieRepository.fetchMoviesFlow().catch {
```

```kt
                assertEquals(exception, it.message)
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

在这个测试类中，每次调用 `MovieService.getMovies()` 时，它都会抛出 `IOException`。然后我们调用 `movieRepository.fetchMoviesFlow()` 并使用 `catch` 操作符来处理异常。然后，我们将异常消息与预期字符串进行比较。

我们也可以通过模拟我们的类以返回一个可以触发重试的特定异常来测试 Flow 重试。对于之后仍然失败的重试，你可以检查异常或异常处理。要测试成功的重试，你可以模拟你的类以抛出异常或返回一个可以与预期值比较的 Flow。

以下示例展示了如何测试一个具有对 `IOException` 重试和任意次数尝试的 Flow：

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
    fun fetchMoviesWithError() { 
```

```kt
        ...
```

```kt
        val movies = listOf(Movie(title = "Movie"))
```

```kt
        val exception = "Exception"
```

```kt
        val hasRetried = false
```

```kt
        val movieRepository: MovieRepository = mock { 
```

```kt
            onBlocking { fetchMoviesFlow() } doAnswer {
```

```kt
                flow {
```

```kt
                    if (hasRetried) emit(movies) else throw
```

```kt
                      IOException (exception)
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

```kt
}
```

在这里，我们使用了一个 `hasRetried` 变量来确定是否返回一个电影流或抛出一个可以触发重试的异常。默认情况下它是 `false` 以允许重试。在代码的后面部分，我们可以将此值更改为 `true` 以返回一个电影流，然后我们可以将其与预期值进行比较。

在本节中，我们学习了如何在 Android 项目中创建和运行 Kotlin Flow 的测试。我们将在下一节中学习如何使用 Turbine 测试热流。

# 使用 Turbine 测试 Flows

在本节中，我们将学习如何使用 Turbine 测试 Flows，这是一个第三方库，我们可以用它来测试项目中的流。

热流，如 `SharedFlow` 和 `StateFlow`，正如你在上一章中学到的，即使在没有监听器的情况下也会发出值。它们也会继续发出值而不会完成。测试它们稍微复杂一些。你无法将这些流转换为列表并与预期值进行比较。

为了测试热流并使测试其他 Flows 更容易，你可以使用 Cash App 中的一个名为 Turbine 的库（[`github.com/cashapp/turbine`](https://github.com/cashapp/turbine)）。Turbine 是一个用于 Kotlin Flow 的小型测试库，你可以在 Android 中使用它。

你可以通过在你的 `app/build.gradle` 依赖项中添加以下内容来在你的 Android 项目中使用 Turbine 测试库：

```kt
dependencies {
```

```kt
    …
```

```kt
    testImplementation 'app.cash.turbine:turbine:0.8.0'
```

```kt
}
```

添加此内容将允许你在项目中使用 Turbine 测试库来测试代码中的 Flow。

Turbine 在 Flow 上有一个 `test` 扩展函数。它有一个挂起验证块，你可以逐个从 Flow 中消费项目并与预期值进行比较。然后在验证块的末尾取消 Flow。

以下代码块展示了如何使用 Turbine 和 `test` 扩展函数测试 Flows 的示例：

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
        val expectedList = listOf(movie1, movie2)
```

```kt
        val movieRepository: MovieRepository = mock {
```

```kt
            onBlocking { fetchMovies() } doReturn
```

```kt
              flowOf(expectedList)
```

```kt
        }
```

```kt
        val movieViewModel =
```

```kt
          MovieViewModel(movieRepository)
```

```kt
        runTest {
```

```kt
            movieViewModel.fetchMovies().test {
```

```kt
               assertEquals(movie1, awaitItem())
```

```kt
               assertEquals(movie2, awaitItem())
```

```kt
               awaitComplete()
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

在这里，测试使用了 `awaitItem()` 函数来获取 Flow 发出的下一个项目，并将其与预期的项目进行比较。然后，它使用了 `awaitComplete()` 函数来断言 Flow 已完成。

要测试 Flow 抛出的异常，你可以使用返回 `Throwable` 的 `awaitError()` 函数。然后你可以将这个 `Throwable` 与你期望抛出的异常进行比较。以下是如何使用此方法测试你的 Flow 的示例：

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
    fun fetchMoviesError() {
```

```kt
        ...
```

```kt
        val exception = "Test Exception"
```

```kt
        val movieRepository: MovieRepository = mock {
```

```kt
            onBlocking { fetchMovies() } doAnswer
```

```kt
                flow {
```

```kt
                    throw RuntimeException(exception)
```

```kt
                }
```

```kt
            }
```

```kt
        }//mock
```

```kt
        val movieViewModel =
```

```kt
          MovieViewModel(movieRepository)
```

```kt
        runTest {
```

```kt
            movieViewModel.fetchMovies().test {
```

```kt
                assertEquals(exception,
```

```kt
                  awaitError().message)
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

在这个例子中，我们使用了 `awaitError()` 函数来接收异常，并将其消息与预期的异常进行比较。

要测试热流，你必须在 `test` lambda 中发出值。你也可以使用 `cancelAndConsumeRemainingEvents()` 函数或 `cancelAndIgnoreRemainingEvents()` 函数来取消 Flow 中任何剩余的事件。

以下是一个使用 `cancelAndIgnoreRemainingEvents()` 函数的示例，在检查 Flow 的第一个项目之后：

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
        val expectedList = listOf(movie1, movie2)
```

```kt
        val movieRepository: MovieRepository = mock {
```

```kt
            onBlocking { fetchMovies() } doReturn
```

```kt
              flowOf(expectedList)
```

```kt
        }
```

```kt
        val movieViewModel =
```

```kt
          MovieViewModel(movieRepository)
```

```kt
        runTest {
```

```kt
            movieViewModel.fetchMovies().test {
```

```kt
               assertEquals(movie1, awaitItem())
```

```kt
               cancelAndIgnoreRemainingEvents()
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

在这里，测试将检查 Flow 的第一个项目，忽略任何剩余的项目，并取消 Flow。

在本节中，你学习了如何使用 Turbine 测试 Flows。让我们通过向 Android 项目中的 Flows 添加一些测试来尝试我们迄今为止所学的内容。

# 练习 7.01 – 在 Android 应用中为 Flows 添加测试

在这个练习中，你将继续你在 *练习 6.01 – 在 Android 应用中处理 Flow 异常* 中工作的电影应用。这个应用显示当前正在电影院上映的电影。你将通过以下步骤在项目中添加对 Kotlin Flows 的测试：

1.  在 Android Studio 中打开你在 *练习 6.01 – 在 Android 应用中处理 Flow 异常* 中工作的电影应用。

1.  前往 `MovieViewModelTest` 类。运行测试类，`fetchMovies()` 测试函数将失败。那是因为我们在上一章中更改了实现以使用 Flow。

1.  移除 `fetchMovies()` 测试函数的内容，并替换为以下内容：

    ```kt
    @Test
    fun fetchMovies() {
        val dispatcher = StandardTestDispatcher()

        val movies = listOf(Movie(title = "Movie"))
        val expectedMovies =
          MutableLiveData<List<Movie>>()
        expectedMovies.postValue(movies)

        val movieRepository: MovieRepository = mock {
            onBlocking { fetchMoviesFlow() } doReturn
              flowOf(movies)
        }

        val movieViewModel =
          MovieViewModel(movieRepository, dispatcher)
    }
    ```

使用此代码，我们将模拟 `MovieRepository` 返回一个包含单个电影的列表流 `movies`。

1.  在 `fetchMovies()` 函数的末尾，添加以下代码来测试 `MovieViewModel` 的 `fetchMovies()` 函数：

    ```kt
    @Test
    fun fetchMovies() {
        ...

        runTest {
            movieViewModel.fetchMovies()
            dispatcher.scheduler.advanceUntilIdle()
            assertEquals(expectedMovies.value,
              movieViewModel.movies.value)
        }
    }
    ```

这将调用 `movieViewModel` 中的 `fetchMovies()` 函数。然后我们将比较返回的 `movieViewModel.movies` 是否与预期的 `movies` 列表（包含单个 Movie 项目）相同。

1.  在 `loading()` 测试函数中，将断言替换为以下内容：

    ```kt
    assertTrue(movieViewModel.loading.value)
    dispatcher.scheduler.advanceUntilIdle()
    assertFalse(movieViewModel.loading.value)
    ```

`loading` 变量不再可以为空，因此这简化了断言语句。

1.  再次运行 `MovieViewModelTest` 类。它应该成功运行，并且所有测试都将通过。

1.  打开 `MovieRepositoryTest` 类。我们将为 `MovieRepository` 的 `fetchMoviesFlow()` 函数添加测试。首先，添加以下函数来测试函数的成功情况：

    ```kt
    @Test
    fun fetchMoviesFlow() {
        val movies = listOf(Movie(id = 3), Movie(id = 4))
        val response = MoviesResponse(1, movies)

        val movieService: MovieService = mock {
            onBlocking { getMovies(anyString()) } doReturn
              response
        }
        val movieRepository =
          MovieRepository(movieService)

        runTest {
            movieRepository.fetchMoviesFlow().collect {
                assertEquals(movies, it)
            }
        }
    }
    ```

这将模拟`MovieRepository`，使其始终返回我们将要稍后与`fetchMoviesFlow()`函数中的电影进行比较的电影列表。

1.  添加以下函数以添加对`fetchMoviesFlow()`函数抛出异常情况的测试：

    ```kt
    @Test
    fun fetchMoviesFlowWithError() {
        val exception = "Test Exception"

        val movieService: MovieService = mock {
            onBlocking { getMovies(anyString()) } doThrow
              RuntimeException(exception)
        }
        val movieRepository =
          MovieRepository(movieService)

        runTest {
            movieRepository.fetchMoviesFlow().catch {
                assertEquals(exception, it.message)
            }
        }
    }
    ```

这个测试将使用一个假的`MovieRepository`，在调用`fetchMoviesFlow`时始终抛出错误。然后，我们将测试抛出的异常是否与我们预期的相同。

1.  运行`MovieRepositoryTest`类。`MovieRepository Test`中的所有测试应该运行并通过，没有错误。

1.  现在，我们将使用 Turbine 测试库来测试`MovieRepository`的`fetchMoviesFlow()`函数生成的 Flow。在`app/build.gradle`依赖中添加以下内容：

    ```kt
    testImplementation 'app.cash.turbine:turbine:0.8.0'
    ```

这将使我们能够使用 Turbine 测试库为 Android 项目中的 Flows 创建单元测试。

1.  添加以下新测试函数以测试`fetchMoviesFlow()`函数的成功情况：

    ```kt
    @Test
    fun fetchMoviesFlowTurbine() {
        val movies = listOf(Movie(id = 3), Movie(id = 4))
        val response = MoviesResponse(1, movies)

        val movieService: MovieService = mock {
            onBlocking { getMovies(anyString()) } doReturn
              response
        }
        val movieRepository =
          MovieRepository(movieService)

        runTest {
            movieRepository.fetchMoviesFlow().test {
                assertEquals(movies, awaitItem())
                awaitComplete()
            }
        }
    }
    ```

通过这种方式，我们将模拟`MovieRepository`以返回一个电影列表。然后，我们将使用`awaitItem()`将其与`movieRepository.fetchMoviesFlow()`返回的列表进行比较。然后`awaitComplete()`函数将检查 Flow 是否已终止。

1.  在`fetchMoviesFlow`抛出异常的情况下，添加以下函数来测试使用 Turbine：

    ```kt
    @Test
    fun fetchMoviesFlowWithErrorTurbine() {
        val exception = "Test Exception"

        val movieService: MovieService = mock {
            onBlocking { getMovies(anyString()) } doThrow
              RuntimeException(exception)
        }
        val movieRepository =
          MovieRepository(movieService)

        runTest {
            movieRepository.fetchMoviesFlow().test {
                assertEquals(exception,
                  awaitError().message)
            }
        }
    }
    ```

这将使用一个`MovieRepository`模拟类，当调用`fetchMoviesFlow()`时将抛出`RuntimeException`。然后，我们将使用`awaitError()`调用测试异常信息是否与获取的相同。

1.  再次运行`MovieRepositoryTest`类。`MovieRepository Test`中的所有测试应该运行并通过，没有错误。

在这个练习中，我们处理了一个使用 Kotlin Flow 的 Android 项目，并为这些 Flows 创建了测试。

# 摘要

本章重点介绍了在 Android 项目中测试 Kotlin Flows。我们首先为添加 Flows 测试设置了项目。协程测试库（**kotlinx-coroutines-test**）可以帮助你创建协程和 Flows 的测试。

我们学习了如何在 Android 应用程序中添加 Flows 的测试。你可以使用返回值流的模拟类，然后将其与返回的值进行比较。你还可以将 Flow 转换为`List`或`Set`，或从 Flow 中获取值；然后你可以将它们与预期值进行比较。

然后，我们学习了如何使用 Turbine 测试库测试热流，这是一个用于测试 Kotlin 流的第三方测试库。Turbine 在 Flow 上有一个`test`扩展，你可以逐个消费和比较值。

最后，我们在一个练习中为现有 Android 项目中的 Kotlin Flows 编写了测试。我们还使用了 Turbine 测试库来简化 Flows 测试的编写。

在整本书中，我们获得了关于 Android 中异步编程的知识和技能。我们学习了如何使用 Kotlin 协程和 Flow 简化 Android 项目中的异步编程。

Android 中的所有内容都在不断进化。还有一些关于协程和 Flow 的更高级主题我们没有涉及。保持对 Android、Kotlin 协程和 Kotlin Flow 最新更新的了解是很好的。您可以在 [`developer.android.com/kotlin/coroutines`](https://developer.android.com/kotlin/coroutines) 上找到关于 Android 协程的最新信息，以及在 [`developer.android.com/kotlin/flow`](https://developer.android.com/kotlin/flow) 上找到关于 Android Kotlin Flow 的最新信息。
