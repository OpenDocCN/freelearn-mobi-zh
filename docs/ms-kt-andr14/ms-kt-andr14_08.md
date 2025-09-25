

# 第六章：使用 Kotlin 协程进行网络调用

我们在手机上使用的多数应用程序都是从服务器上托管的数据。因此，作为开发者，我们必须了解如何向服务器请求数据和发送数据。在本章中，我们将学习如何发送和请求在线托管的数据，并在我们的应用程序中显示它。

在本章中，我们将学习如何使用网络库**Retrofit**执行网络调用。我们将学习如何使用此库消费**应用程序编程接口**（**API**）。更重要的是，我们将学习如何利用**Kotlin 协程**在我们的应用程序中执行异步网络请求。

在本章中，我们将涵盖以下主要主题：

+   设置 Retrofit

+   Kotlin 协程简介

+   使用 Kotlin 协程进行网络调用

# 技术要求

要遵循本章的说明，您需要下载 Android Studio Hedgehog 或更高版本([`developer.android.com/studio`](https://developer.android.com/studio))。

您可以在[`github.com/PacktPublishing/Mastering-Kotlin-for-Android/tree/main/chaptersix`](https://github.com/PacktPublishing/Mastering-Kotlin-for-Android/tree/main/chaptersix)找到本章的代码。

# 设置 Retrofit

Retrofit 是由 Square 开发的 Android、Java 和 Kotlin 的 Type-safe REST 客户端。该库提供了一个强大的框架，用于验证和与 API 交互，以及使用 OkHttp 发送网络请求。在本书中，我们将使用 Retrofit 来执行我们的网络请求。

首先，我们将使用我们新创建的版本目录添加 Retrofit 依赖项。让我们在`libs.versions.toml`文件中定义版本，如下所示：

```java
retrofit = "2.9.0"
retrofitSerializationConverter = "1.0.0"
serializationJson = "1.5.1"
coroutines = "1.7.3"
okhttp3 = "4.11.0"
```

接下来，让我们在版本目录的库部分中定义`libs.versions.toml`文件中的库，如下所示：

```java
retrofit = { module = "com.squareup.retrofit2:retrofit" , version.ref = "retrofit" }
retrofit-serialization = { module = "com.jakewharton.retrofit:retrofit2-kotlinx-serialization-converter", version.ref = "retrofitSerializationConverter" }
coroutines = { module = "org.jetbrains.kotlinx:kotlinx-coroutines-core" , version.ref = "coroutines" }
coroutines-android = { module = "org.jetbrains.kotlinx:kotlinx-coroutines-android" , version.ref = "coroutines" }
serialization-json = { module = "org.jetbrains.kotlinx:kotlinx-serialization-json", version.ref = "serializationJson" }
okhttp3 = { module = "com.squareup.okhttp3:okhttp", version.ref = "okhttp3" }
```

我们正在将以下依赖项添加到我们的项目中：

+   **Retrofit**：如前所述，我们将使用 Retrofit 来执行我们的网络请求。

+   **Retrofit 序列化**：这是一个使用**Kotlinx serialization**将 Kotlin 对象转换为 JSON 并从 JSON 转换回来的转换器。

+   **协程**：我们将使用 Kotlin 协程来异步执行我们的网络请求。我们很快就会了解更多关于协程的内容。

+   **Kotlinx serialization JSON**：这是一个用于 JSON 的 Kotlin 序列化库。我们将使用它来解析我们的 JSON 响应。我们还有其他序列化库，如 Moshi 和 Gson，但我们选择使用 Kotlinx serialization 库的原因如下：

    +   **以 Kotlin 为中心的开发**：Kotlinx serialization 是针对 Kotlin 设计的，提供无缝集成和原生支持，以便进行 Kotlin 序列化。

    +   **声明式语法**：Kotlinx serialization 使用声明式语法，利用 Kotlin 的语言特性来编写简洁且易于阅读的序列化代码。

    +   **编译时安全性**：编译时安全性是一个关键特性，在编译阶段捕获与序列化相关的错误，并减少运行时错误的可能性。

    +   **自定义序列化策略**：我们有灵活性为特定类型或场景定义自定义序列化策略，提供对序列化过程的精细控制。

    +   **与 Kotlin 生态系统的无缝集成**：作为 Kotlin 生态系统的一部分，Kotlinx 序列化与其它 Kotlin 库和框架无缝集成，有助于提供一致的开发体验。

+   **OkHttp**：这是一个用于发送网络请求的 HTTP 客户端。它为使用 Retrofit 提供了一些实用工具。

所有这些依赖项都将一起添加，因此这是我们可以在我们的 `libs.versions.toml` 文件中将它们分组的机会，在 Koin 包下面添加此包：

```java
networking = ["retrofit", "retrofit-serialization", "serialization-json", "coroutines", "coroutines-android"]
```

在这里，我们创建了一个名为 `networking` 的新包，并添加了之前指定的所有依赖项。我们必须同步项目，以便将我们的更改添加到项目中。点击 `libs.versions.toml` 文件。同步后，让我们开始设置插件和依赖项。

首先，在我们的项目级别 `build.gradle.kts` 文件中，我们需要添加 Kotlinx 序列化插件。打开项目级别的 `build.gradle.kts` 文件，并在插件块中添加以下内容：

```java
id("org.jetbrains.kotlin.plugin.serialization") version "1.8.20" apply false
```

我们定义了 Kotlinx 序列化插件并指定了要使用的版本。这将为我们设置 Kotlinx 序列化插件。该插件为可序列化类生成 Kotlin 代码。我们将使用此插件来生成我们的模型。接下来，让我们在我们的应用模块中设置此插件。打开应用级别的 `build.gradle.kts` 文件，并在插件块中添加以下内容：

```java
id("kotlinx-serialization")
```

这确保了我们的模块已设置好以使用 Kotlinx 序列化插件。接下来，我们将添加我们的 `networking` 包到我们的应用模块中。在应用级别的 `build.gradle.kts` 文件中，添加以下内容：

```java
implementation(libs.bundles.networking)
```

这将添加我们在 `networking` 包中指定的所有依赖项。完成所有这些后，我们的项目已设置好以使用 Retrofit。我们将使用 Koin 创建一个 Retrofit 实例，该实例将被注入到需要它的类中。让我们转到 `Module.kt` 文件并添加 `PetsViewModel` 定义：

```java
single {
    Retrofit.Builder()
        .addConverterFactory(
            Json.asConverterFactory(contentType = "application/json".toMediaType())
        )
        .baseUrl("https://cataas.com/api/")
        .build()
}
```

在前面的代码中，我们使用 Retrofit 构建器创建了 Retrofit 实例。我们还添加了一个使用 Kotlinx 序列化来将 Kotlin 对象转换为 JSON 并从 JSON 转换回 Kotlin 对象的转换器工厂。我们还指定了我们的 API 的基本 URL。我们使用 `CatsAPI.kt` 并添加以下方法：

```java
@GET("cats")
suspend fun fetchCats(
    @Query("tag") tag: String,
): Response<List<Cat>>
```

在前面的代码中，我们使用 `@GET` 注解来指定我们将使用 `GET` HTTP 方法进行此请求。在方法内部，我们还指定了一个将被附加到我们的基本 URL 的路径，以形成请求的完整 URL。使用 `GET` 方法意味着我们的方法将只请求数据。我们有以下内置的 HTTP 注解：

+   **POST**：当我们想要向服务器发送数据时使用

+   **PUT**: 这用于当我们想要在服务器上更新数据时

+   **DELETE**: 这用于当我们想要从服务器删除数据时

+   **HEAD**: 此方法请求与**GET**请求相对应的响应，但不包含响应体

+   **PATCH**: 这用于当我们想要在服务器上部分更新数据时

+   **OPTIONS**: 此方法请求目标资源的允许通信选项

回到我们的`fetchCats()`函数，你可以注意到我们使用了`@Query`注解来指定请求的查询参数。我们使用`tag`查询参数来指定我们想要获取的猫的类型。我们还使用了`suspend`关键字来指定这个方法将从协程或另一个`suspend`函数中被调用。我们将在本章的*Kotlin 协程简介*部分稍后了解更多关于协程的内容。我们还使用了`Response`类来封装我们的响应。这个类由 Retrofit 提供，它包含了 HTTP 响应元数据，如响应代码、头信息和原始响应体。我们还指定响应将是一个`Cat`对象的列表。Retrofit 会将响应映射到一个`Cat`对象的列表。为了解决`Cat`数据类的错误，让我们创建它。在数据包内创建一个新的 Kotlin 数据类，命名为`Cat.kt`，并添加以下内容：

```java
@Serializable
data class Cat(
    @SerialName ("createdAt")
    val createdAt: String,
    @SerialName("_id")
    val id: String,
    @SerialName("owner")
    val owner: String,
    @SerialName("tags")
    val tags: List<String>,
    @SerialName("updatedAt")
    val updatedAt: String
)
```

`Cat`数据类具有与 Cat as a Service API 的 JSON 响应相对应的字段。它还注解了`@Serializable`注解。这个注解由 Kotlinx Serialization 提供，它用于标记一个类为可序列化。这个注解对于所有我们想要序列化或反序列化的类都是必需的。我们在数据类的每个变量之前使用了`@SerialName`注解。`@SerialName`是一个用于自定义 Kotlin 属性名与序列化形式（如 JSON 或其他数据交换格式）中相应名称之间映射的注解。这个注解允许你在序列化或反序列化时为属性指定不同的名称，从而在处理命名约定时提供灵活性。

在我们的项目中，我们使用 Koin 进行依赖注入。因此，我们现在需要在我们的 Koin 模块中创建`CatsAPI`类的实例。让我们回到`Module.kt`文件，并在 Retrofit 实例下面添加以下内容：

```java
single { get<Retrofit>().create(CatsAPI::class.java) }
```

在这里，我们获取我们的 Retrofit 实例，并使用它来创建我们的`CatsAPI`类的实例，我们使用它来执行实际的网络请求。有了这个，我们的项目就准备好执行网络请求了。但在那之前，让我们更多地了解 Kotlin 协程，因为我们将修改我们的仓库以使用协程。

# Kotlin 协程简介

JetBrains 为 Kotlin 引入的协程提供了一种以更可读和同步的方式编写异步代码的方法。我们可以使用它们来执行后台任务，它们是执行网络请求和长时间运行任务（如读取和写入数据库）的绝佳方式。它们在主线程之外执行这些任务，并确保我们在执行这些操作时不会阻塞主线程。使用协程的主要好处如下：

+   它们轻量级且易于使用。

+   它们内置了取消支持。

+   它们降低了应用出现内存泄漏的可能性。

+   如前几章所述，Jetpack 库也支持并使用协程。

我们已经将核心和 Android 协程库添加到我们的应用中。在继续在项目中使用协程之前，让我们先了解一些协程基础知识。

## 协程基础

在本节中，我们将探讨在 Kotlin 协程中使用的不同术语和概念：

+   **挂起**：这是一个用于标记函数的关键字。**挂起**函数是可以暂停并在稍后恢复的函数。我们已经在**CatsAPI**类中使用此关键字将**fetchCats()**函数标记为**挂起**函数。**挂起**函数只能从另一个**挂起**函数或从协程中调用。

+   **协程构建器**：这些是用于创建协程的函数。我们有**启动**和**异步**协程构建器。**启动**用于创建不返回结果的协程，而**异步**用于创建返回结果的协程。结果是**延迟**对象，我们可以使用**await()**方法来获取结果。这两个构建器都返回一个**作业**对象，我们可以用它来检查协程是否仍然活跃或已被取消。我们还可以使用作业来等待协程完成。作业在完成或取消时结束。

+   **作业**：作业是一个具有**生命周期**的协程实例，可以被取消。我们可以使用作业来检查协程是否仍然活跃或已被取消。我们还可以使用作业来等待协程完成。作业在完成或取消时结束。如前所述，**启动**和**异步**协程构建器都返回一个**作业**对象，我们使用它来管理协程的生命周期。我们有一个普通的**作业**和一个**监督作业**。当任何子作业失败时，普通的**作业**会被取消。**监督作业**在子作业失败时不会被取消。当有多个协程同时运行时，建议使用**监督作业**。

+   **协程作用域（Coroutine scope）**：这跟踪我们使用**launch**或**async**构建器创建的所有协程。它负责知道协程将存活多久。每个协程构建器都被定义为作用域的扩展函数。没有作用域就无法启动协程。我们有**GlobalScope**，这是一个与任何生命周期无关的作用域。不建议使用此作用域，因为它可能导致内存泄漏。在 Android 中，KTX 库提供了**viewModelScope**，这是一个与**ViewModel**相关联的作用域。我们可以使用此作用域来启动当**ViewModel**被销毁时将取消的协程。我们还有**lifecycleScope**，这是一个与活动或片段生命周期相关联的作用域。我们可以使用此作用域来启动当生命周期被销毁时将取消的协程。我们还可以创建自己的自定义作用域，如果我们想启动当自定义生命周期被销毁时将取消的协程。

+   **协程上下文（Coroutine context）**：这是一个包含许多元素的集合。**CoroutineContext**使用如下元素等定义了我们的协程的行为：

    +   **作业（Job）**：这管理协程的生命周期。

    +   **协程调度器（CoroutineDispatcher）**：这定义了协程将在哪个线程上运行。

    +   **协程名称（CoroutineName）**：这定义了协程的名称。

    +   **协程异常处理器（CoroutineExceptionHandler）**：这用于处理协程中的未捕获异常。

+   `withContext()`函数用于在不同调度器之间切换。`withContext()`是一个**挂起**函数，它切换协程的上下文。

+   **流（Flows）**：挂起函数仅返回单个值。流是一种可以返回多个值的异步数据流。我们可以使用流从**挂起**函数返回多个值。我们还可以使用流来执行异步操作。流是**冷流**。这意味着它们只有在被收集时才开始发出值。我们可以使用**collect()**函数从流中收集值。我们有**StateFlow**和**SharedFlow**，它们是流的一种类型。**StateFlow**是一种流，它将当前值发送给新的收集器，并将新值发送给现有的收集器。**SharedFlow**是一种将新值发送给所有收集器的流。我们将在下一章中学习更多关于流的内容。在 Android 中，我们通常使用这两种类型的流将数据发射到我们的 UI。我们将看到**StateFlow**在**ViewModel**重构时使用协程的用法。

通过对基础知识的理解，在下一节中，我们将重构我们的仓库以使用协程。

# 使用 Kotlin 协程进行网络调用

在本节中，我们将重构我们的仓库以使用协程。我们将使用`StateFlow`从`ViewModel`向视图层发射数据。我们还将使用`Dispatchers.IO`调度器在后台线程上执行我们的网络请求。

让我们首先创建一个`NetworkResult`密封类，它将表示我们的网络请求的不同状态：

```java
sealed class NetworkResult<out T> {
    data class Success<out T>(val data: T) : NetworkResult<T>()
    data class Error(val error: String) : NetworkResult<Nothing>()
}
```

`NetworkResult`类是一个密封类，有两个子类。我们有`Success`数据类，它将用于表示成功的网络请求。它有一个数据属性，将用于存储从网络请求返回的数据。我们还有`Error`类，它将用于表示失败的网络请求。它有一个`error`属性，将用于存储从网络请求返回的错误信息。密封类封装了一个泛型数据类型`T`，这使得我们更容易在所有网络调用中重用该类。`Success`数据类也有一个泛型参数，出于相同的目的。

接下来，让我们按照以下方式修改`PetsRepository`：

```java
interface PetsRepository {
    suspend fun getPets(): NetworkResult<List<Cat>>
}
```

我们已更新界面以使用`NetworkResult`类。我们还将`getPets()`函数标记为`suspend`函数。我们将使用此方法从 API 获取猫。接下来，让我们修改`PetsRepositoryImpl`以添加来自`PetsRepository`的更改：

```java
class PetsRepositoryImpl(
    private  val catsAPI: CatsAPI,
    private val dispatcher: CoroutineDispatcher
): PetsRepository {
    override suspend fun getPets(): NetworkResult<List<Cat>> {
        return withContext(dispatcher) {
            try {
                val response = catsAPI.fetchCats("cute")
                if (response.isSuccessful) {
                    NetworkResult.Success(response.body()!!)
                } else {
                    NetworkResult.Error(response.errorBody().toString())
                }
            } catch (e: Exception) {
                NetworkResult.Error(e.message ?: "Unknown error")
            }
        }
    }
}
```

在这里我们更改了许多事情：

+   首先，我们添加了一个构造函数，它接受我们的**CatsAPI**类的实例，我们将使用它来发出网络请求。它还有一个**dispatcher**参数，它将用于指定我们将用于执行网络请求的调度器。我们将使用**Dispatchers.IO**调度器在后台线程上执行我们的网络请求。

+   我们还将`getPets()`函数的返回类型更改为`NetworkResult<List<Cat>>`。这是因为我们将从这个方法返回一个`NetworkResult`对象。

+   我们使用**withContext()**函数将协程的上下文切换到**Dispatchers.IO**调度器。这确保了网络请求是在后台线程上执行的。

+   我们还将在**try-catch**块中包装我们的网络请求。这是为了确保我们捕获网络请求过程中可能发生的所有错误。

+   在我们的**try**块内部，我们使用我们的**CatsAPI**实例发出网络请求。我们使用**fetchCats()**方法发出请求。我们传递**cute**标签来指定我们想要获取的猫的类型。我们检查响应是否成功。如果是，我们返回一个包含响应体的**NetworkResult.Success**对象。如果不是，我们返回一个包含错误信息的**NetworkResult.Error**对象。

+   最后，我们捕获网络请求过程中可能发生的所有异常，并返回一个包含错误信息的**NetworkResult.Error**对象。

在我们的 Koin 模块中，我们还需要更改我们实例化我们的仓库的方式。让我们转到`Module.kt`并更新`PetsRepository`定义如下：

```java
single<PetsRepository> { PetsRepositoryImpl(get(), get()) }
single { Dispatchers.IO }
```

我们将 `CatsAPI` 实例和 `dispatcher` 注入到我们的仓库中。我们还声明 `dispatcher` 为一个单例实例。现在我们需要修改我们的 `PetsViewModel` 以适应这些更改。首先，我们需要创建一个状态类来保存我们的网络请求状态并将其暴露给我们的视图。在 `view` 包中创建一个新的 Kotlin 数据类，命名为 `PetsUIState.kt`：

```java
data class PetsUIState(
    val isLoading: Boolean = false,
    val pets: List<Cat> = emptyList(),
    val error: String? = null
)
```

`PetsUIState` 类是一个数据类，用于保存我们的网络请求状态。它有三个属性：

+   **isLoading**：这是一个布尔值，用于指示网络请求是否正在加载。

+   **pets**：这是一个将从网络请求返回的猫的列表。

+   **error**：这是一个字符串，将用于保存从网络请求返回的错误信息。

接下来，在 `PetsViewModel` 中，让我们创建一个变量来保存我们的网络请求状态：

```java
val petsUIState = MutableStateFlow(PetsUIState())
```

我们使用 `MutableStateFlow` 类来保存我们的网络请求状态。`MutableStateFlow` 允许我们更新状态的值。我们用空的 `PetsUIState` 对象初始化它。接下来，让我们更新 `getPets()` 方法如下：

```java
private fun getPets() {
    petsUIState.value = PetsUIState(isLoading = true)
    viewModelScope.launch {
        when (val result = petsRepository.getPets()) {
            is NetworkResult.Success -> {
                petsUIState.update {
                    it.copy(isLoading = false, pets = result.data)
                }
            }
            is NetworkResult.Error -> {
                petsUIState.update {
                    it.copy(isLoading = false, error = result.error)
                }
            }
        }
    }
}
```

在这里，我们将分解前面的代码：

+   我们更新 **petsUIState** 变量的值以指示网络请求正在加载。

+   我们使用 **viewModelScope** 来启动一个协程。这确保了当 **ViewModel** 被销毁时，协程将被取消。

+   有一个 **when** 语句，这是 Kotlin 的模式匹配功能，用于检查网络请求的结果。如果结果是 **NetworkResult.Success** 对象，我们将更新 **petsUIState** 的值以指示网络请求成功，并传入猫的列表。如果结果是 **NetworkResult.Error** 对象，我们将更新 **petsUIState** 的值以指示网络请求失败，并传入错误信息。

在 `PetsViewModel` 中，让我们添加一个新的 `init` 块，该块将调用 `getPets()` 函数：

```java
init {
    getPets()
}
```

这将确保在创建 `ViewModel` 时调用 `getPets()` 函数。我们现在需要更新我们的 `PetList` 可组合组件以适应这些更改，我们还将添加更多的 UI 组件，因为我们需要显示加载状态、图片和错误信息。让我们首先添加一个允许我们从 URL 加载图片的库。我们将使用 Coil ([`coil-kt.github.io/coil/`](https://coil-kt.github.io/coil/))，这是一个图像加载库。在版本目录中，让我们添加以下内容：

```java
coil-compose = "io.coil-kt:coil-compose:2.4.0"
```

我们还将向 `compose` 包添加 `coil-compose` 依赖项，以便它可以与其他 compose 库一起提供。更新的 `compose` 包将如下所示：

```java
compose = ["compose.ui", "compose.ui.graphics", "compose.ui.tooling", "compose.material3", "compose.viewmodel", "coil-compose"]
```

现在，让我们在名为 `PetListItem.kt` 的 `view` 包中创建一个新的可组合组件，用于显示每只猫的图片和标签，并添加以下内容：

```java
@OptIn(ExperimentalLayoutApi::class)
@Composable
fun PetListItem(cat: Cat) {
    ElevatedCard(
        modifier = Modifier
            .fillMaxWidth()
            .padding(6.dp)
    ) {
        Column(
            modifier = Modifier
                .fillMaxWidth()
                .padding(bottom = 10.dp)
        ) {
            AsyncImage(
                model = "https://cataas.com/cat/${cat.id}",
                contentDescription = "Cute cat",
                modifier = Modifier
                    .fillMaxWidth()
                    .height(200.dp),
                contentScale = ContentScale.FillWidth
            )
            FlowRow(
                modifier = Modifier
                    .padding(start = 6.dp, end = 6.dp)
            ) {
                repeat(cat.tags.size) {
                    SuggestionChip(
                        modifier = Modifier
                            .padding(start = 3.dp, end = 3.dp),
                        onClick = { },
                        label = {
                            Text(text = cat.tags[it])
                        }
                    )
                }
            }
        }
    }
}
```

此可组合组件接受一个`Cat`对象，并显示猫咪的图片和标签。我们使用 Coil 库中的`AsyncImage`可组合组件从 URL 加载图片。我们还使用`FlowRow`可组合组件显示猫咪的标签。我们使用`SuggestionChip`可组合组件显示每个标签。我们在`ElevatedCard`可组合组件中显示图片和标签。

接下来，让我们更新我们的`PetList`可组合组件以适应这些更改。在`PetList.kt`文件中，更新`PetList`可组合组件如下：

```java
@Composable
fun PetList(modifier: Modifier) {
    val petsViewModel: PetsViewModel = koinViewModel()
    val petsUIState by petsViewModel.petsUIState.collectAsStateWithLifecycle()
    Column(
        modifier = modifier
            .padding(16.dp),
        verticalArrangement = Arrangement.Center,
        horizontalAlignment = Alignment.CenterHorizontally
    ) {
        AnimatedVisibility(
            visible = petsUIState.isLoading
        ) {
            CircularProgressIndicator()
        }
        AnimatedVisibility(
            visible = petsUIState.pets.isNotEmpty()
        ) {
            LazyColumn {
                items(petsUIState.pets) { pet ->
                    PetListItem(cat = pet)
                }
            }
        }
        AnimatedVisibility(
            visible = petsUIState.error != null
        ) {
            Text(text = petsUIState.error ?: "")
        }
    }
}
```

以下是对前面代码的分解：

+   与之前一样，我们使用**koinViewModel()**函数来获取**PetsViewModel**的一个实例。

+   我们使用**collectAsStateWithLifecycle()**函数来收集我们的网络请求状态。此函数由**lifecycle-runtime-compose**库提供。它用于收集流的状态，并在生命周期被销毁时自动取消收集。我们使用**PetsViewModel**的**petsUIState**属性来获取我们的网络请求状态。

+   我们有一个**列**可组合组件，它包含三个**动画可见性**可组合组件。第一个用于在网络请求加载时显示**CircularProgressIndicator**。第二个用于在网络请求成功时显示猫咪列表。最后一个用于在网络请求失败时显示错误信息。

`collectAsStateWithLifecycle()`显示了一个错误，因为我们还没有添加其依赖项。让我们将其添加到版本目录中的库部分，如下所示：

```java
compose-lifecycle = { module = "androidx.lifecycle:lifecycle-runtime-compose", version.ref = "lifecycle" }
```

我们还将将其添加到我们的`compose`包中，以便它可以与其他`compose`库一起提供。更新后的`compose`包如下所示：

```java
compose = ["compose.ui", "compose.ui.graphics", "compose.ui.tooling", "compose.material3", "compose.viewmodel", "coil-compose", "compose-lifecycle"]
```

执行 Gradle 同步，IDE 将提示您为`collectAsStateWithLifecycle()`函数添加导入。

我们已经完成了所有层的更新，以使用新的协程方法。到目前为止做得很好！最后一件事：由于我们的应用现在是从在线托管 API 中获取这些项目，我们需要向我们的应用添加`INTERNET`权限。打开`AndroidManifest.xml`文件，并添加以下内容：

```java
<uses-permission android:name="android.permission.INTERNET" />
```

运行应用并查看一切是否按预期工作。我们可以看到显示标签的可爱猫咪列表。我们还可以看到网络请求加载时的加载指示器以及网络请求失败时的错误信息。我们已经成功重构了我们的应用以使用协程。

![图 6.1 – 可爱的猫咪](img/B19779_06_01.jpg)

图 6.1 – 可爱的猫咪

# 摘要

在本章中，我们学习了如何使用 Retrofit 执行网络调用。更重要的是，我们学习了如何利用 Kotlin 协程在我们的应用中执行异步网络请求，并使用 Kotlin 协程重构了我们的应用以获取一些可爱的猫咪。

在下一章中，我们将探讨另一个 Jetpack 库，**Jetpack Navigation**，以处理我们的应用中的导航。
