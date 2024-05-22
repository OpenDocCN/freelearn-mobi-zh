# 第五章：必要的库：Retrofit、Moshi 和 Glide

概述

在本章中，我们将介绍呈现来自远程服务器获取的动态内容所需的步骤。您将了解到检索和处理此动态数据所需的不同库。

在本章结束时，您将能够使用 Retrofit 从网络端点获取数据，使用 Moshi 将 JSON 有效负载解析为 Kotlin 数据对象，并使用 Glide 将图像加载到`ImageViews`中。

# 介绍

在上一章中，我们学习了如何在应用程序中实现导航。在本章中，我们将学习如何在用户在我们的应用程序中导航时向他们呈现动态内容。

向用户呈现的数据可以来自不同的来源。它可以硬编码到应用程序中，但这会带来一些限制。要更改硬编码数据，我们必须发布应用程序的更新。某些数据由于其性质而无法硬编码，例如货币汇率、资产的实时可用性和当前天气等。其他数据可能会过时，例如应用程序的使用条款。

在这种情况下，您通常会从服务器获取相关数据。用于提供此类数据的最常见架构之一是**表现状态转移**（**REST**）架构。REST 架构由一组六个约束定义：客户端-服务器架构、无状态性、可缓存性、分层系统、按需代码（可选）和统一接口。要了解有关 REST 的更多信息，请访问[`medium.com/extend/what-is-rest-a-simple-explanation-for-beginners-part-1-introduction-b4a072f8740f`](https://medium.com/extend/what-is-rest-a-simple-explanation-for-beginners-part-1-introduction-b4a072f8740f)。

当应用于 Web 服务**应用程序编程接口**（**API**）时，我们得到了基于**超文本传输协议**（**HTTP**）的 RESTful API。HTTP 协议是互联网数据通信的基础，也被称为万维网。它是全球各地服务器用来向用户提供 HTML 文档、图像、样式表等的协议。有关此主题的有趣文章，可以在[`developer.mozilla.org/en-US/docs/Web/HTTP/Overview`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Overview)找到。

RESTful API 依赖于标准的 HTTP 方法——`GET`、`POST`、`PUT`、`DELETE`和`PATCH`——来获取和转换数据。这些方法允许我们在远程服务器上获取、存储、删除和更新数据实体。

要执行这些 HTTP 方法，我们可以依赖于内置的 Java `HttpURLConnection`类，或者使用诸如`OkHttp`之类的库，它提供了额外的功能，如 gzip 压缩、重定向、重试以及同步和异步调用。有趣的是，从 Android 4.4 开始，`HttpURLConnection`只是`OkHttp`的一个包装器。如果我们选择`OkHttp`，我们也可以选择**Retrofit**，正如我们将在本章中所做的那样，以从其类型安全中受益，这更适合处理 REST 调用。

最常用的数据表示形式是**JavaScript 对象表示法**（**JSON**）。JSON 是一种基于文本的数据传输格式。顾名思义，它源自 JavaScript。然而，它已经成为了最流行的数据传输标准之一，大多数现代编程语言都有编码或解码数据到 JSON 或从 JSON 的库。一个简单的 JSON 有效负载可能如下所示：

```kt
{"employees":[  
    {"name": "James", "email": "james.notmyemail@gmail.com"},
    {"name": "Lea", "email": "lea.dontemailme@gmail.com"},
    {"name": "Steve", "email": "steve.notreally@gmail.com"}
]}
```

RESTful 服务常用的另一种数据结构是**可扩展标记语言**（**XML**），它以一种人类和机器可读的格式对文档进行编码。XML 比 JSON 更冗长。以 XML 表示的与前述相同的数据结构可能如下所示：

```kt
<employees>
    <employee>
        <name>James</name>
        <email>james.notmyemail@gmail.com</email>
    </employee>
    <employee>
        <name>Lea</name>
        <email>lea.dontemailme@gmail.com</email>
    </employee>
    <employee>
        <name>Steve</name>
        <email>steve.notreally@gmail.com</email>
    </employee>
</employees>
```

在本章中，我们将专注于 JSON。

当获取 JSON 有效负载时，我们实质上是接收一个字符串。要将该字符串转换为数据对象，我们有一些选项，其中最流行的是`org.json`包等库。由于其轻量级特性，我们将专注于 Moshi。

最后，我们将研究如何从网络加载图像。这样做不仅可以让我们提供最新的图像，还可以为用户的设备加载正确的图像。这样做还可以让我们在需要时才加载图像，从而保持 APK 大小较小。

# 从网络端点获取数据

为了完成本节的目的，我们将使用 TheCatAPI（[`thecatapi.com/`](https://thecatapi.com/)）。这个 RESTful API 为我们提供了大量关于猫的数据。

要开始，我们将创建一个新项目。然后我们必须授予我们的应用程序互联网访问权限。这是通过在您的`AndroidManifest.xml`文件中，在`Application`标签之前添加以下代码来完成的：

```kt
    <uses-permission android:name="android.permission.INTERNET" />
```

接下来，我们需要设置我们的应用程序以包含 Retrofit。`OkHttp` HTTP 客户端。 Retrofit 帮助我们生成**统一资源定位符**（**URL**），这些是我们要访问的服务器端点的地址。它还通过与几个解析库的集成，使 JSON 有效负载的解码更容易。使用 Retrofit 发送数据到服务器也更容易，因为它有助于对请求进行编码。您可以在这里阅读更多关于 Retrofit 的信息：[`square.github.io/retrofit/`](https://square.github.io/retrofit/)。

要将 Retrofit 添加到我们的项目中，我们需要将以下代码添加到我们应用程序的`build.gradle`文件的`dependencies`块中：

```kt
implementation 'com.squareup.retrofit2:retrofit:(insert latest version)'
```

注意

您可以在这里找到最新版本：[`github.com/square/retrofit`](https://github.com/square/retrofit)。

在我们的项目中包含了 Retrofit 后，我们可以继续设置它。

首先，要访问 HTTP(S)端点，我们首先要定义与该端点的合同。访问`https://api.thecatapi.com/v1/images/search`端点的合同如下：

```kt
interface TheCatApiService {
    @GET("images/search")
    fun searchImages(
        @Query("limit") limit: Int,
        @Query("size") format: String
    ): Call<String>
}
```

这里有几点需要注意。首先，您会注意到合同被实现为一个接口。这是您为 Retrofit 定义合同的方式。接下来，您会注意到接口的名称暗示着这个接口最终可以涵盖对 TheCatAPI 服务的所有调用。遗憾的是 Square 选择了`Service`作为这些合同的常规后缀，因为在 Android 世界中，服务一词有不同的含义，您将在*第八章*，*服务、广播接收器和通知*中看到。尽管如此，这是惯例。

要定义我们的端点，我们首先要声明使用适当注释进行调用的方法。在我们的情况下，是`@GET`。传递给注释的参数是要访问的端点的路径。您会注意到`https://api.thecatapi.com/v1/`从该路径中删除了。这是因为这是 TheCatAPI 所有端点的常用地址，因此将在构建时传递给我们的 Retrofit 实例。接下来，我们选择一个有意义的函数名字，比如在这种情况下，我们将调用图像搜索端点，所以`searchImages`似乎是合适的。`searchImages`函数的参数定义了我们在进行调用时可以传递给 API 的值。

有多种方式可以将数据传输到 API。`@Query`允许我们定义添加到请求 URL 查询的值（这是 URL 中问号后面的可选部分）。它接受一个键值对（在我们的例子中，我们有`limit`和`size`）和一个数据类型。如果数据类型不是字符串，那么该类型的值将被转换为字符串。传递的任何值都将被 URL 编码。

另一种方法是使用`@Path`。此注释可用于将路径中用大括号括起来的标记替换为提供的值。`@Header`、`@Headers`和`@HeaderMap`注释将允许我们向请求添加或删除 HTTP 标头。`@Body`可用于在`POST`/`PUT`请求的正文中传递内容。

最后，我们有一个返回类型。在这个阶段，为了保持简单，我们将接受响应作为字符串。我们将字符串包装在`Call`接口中。`Call`是 Retrofit 执行网络请求的机制，可以同步（通过`execute()`）或异步（通过`enqueue(Callback)`）执行。当使用 RxJava（ReactiveX 的 Java 实现，或者叫做 Reactive Extensions；您可以在`https://reactivex.io/`上了解更多关于 ReactiveX 的信息）时，我们可以适当地将结果包装在`Observable`类（发出数据的类）或`Single`类（一次发出数据的类）中（有关 RxJava 的更多信息，请参见*第十三章*，*RxJava 和协程*）。

定义了我们的合同，我们可以让 Retrofit 实现我们的服务接口：

```kt
val retrofit = Retrofit.Builder()
    .baseUrl("https://api.thecatapi.com/v1/")
    .build()
val theCatApiService = retrofit.create(TheCatApiService::class.java)
```

如果我们尝试使用此代码运行应用程序，应用程序将崩溃并显示`IllegalArgumentException`。这是因为 Retrofit 需要我们告诉应用程序如何将服务器响应处理为字符串。这个处理是通过 Retrofit 调用的`ConverterFactory`实例来完成的，我们需要向我们的`retrofit`实例添加以下内容：

```kt
val retrofit = Retrofit.Builder()
    .baseUrl("https://api.thecatapi.com/v1/")
    .addConverterFactory(ScalarsConverterFactory.create())
    .build()
```

为了使我们的项目识别`ScalarsConverterFactory`，我们需要通过添加另一个依赖项来更新我们的应用程序的`build.gradle`文件：

```kt
implementation 'com.squareup.retrofit2:converter-scalars:(insert latest version)'
```

现在，我们可以通过调用`val call = theCatApiService.searchImages(1, "full")`来获得一个`Call`实例。通过以这种方式获得的实例，我们可以通过调用`call.enqueue(Callback)`来执行异步请求。

我们的`Callback`实现将有两个方法：`onFailure(Call, Throwable)`和`onResponse(Call, Response)`。请注意，如果调用了`onResponse`，我们不能保证会有一个成功的响应。只要我们成功地从服务器接收到任何响应并且没有发生意外异常，就会调用`onResponse`。因此，为了确认响应是成功的响应，我们应该检查`response.isSuccessful`属性。在网络错误或沿途某处发生意外异常的情况下，将调用`onFailure`函数。

那么我们应该在哪里实现 Retrofit 代码呢？在干净的架构中，数据由存储库提供。存储库又有数据源。这样的数据源可以是网络数据源。这就是我们将实现网络调用的地方。然后，我们的 ViewModel（在**Model-View-ViewModel**（**MVVM**）的情况下，ViewModel 是暴露属性和命令的视图的抽象）将通过用例从存储库请求数据。

对于我们的实现，我们将简化流程，通过在 Activity 中实例化 Retrofit 和服务来完成。这不是一个好的做法。在生产应用中不要这样做。它不具有良好的可扩展性，并且非常难以测试。相反，采用一种将视图与业务逻辑和数据解耦的架构。参见*第十四章*，*架构模式*，了解一些想法。

## 练习 5.01：从 API 读取数据

在接下来的章节中，我们将开发一个为一家全球特工机构的虚构应用程序，该机构拥有一个遍布全球的特工网络，拯救世界免受无数危险。所涉及的秘密机构非常独特：它运营秘密猫特工。在这个练习中，我们将创建一个应用程序，该应用程序将向我们展示来自 TheCatAPI 的一个随机秘密猫特工。在向用户呈现 API 数据之前，您首先必须获取该数据。让我们开始：

1.  首先创建一个新的`Empty Activity`项目（`文件`|`新建`|`新项目`|`空活动`）。然后点击`下一步`。

1.  将应用程序命名为`Cat Agent Profile`。

1.  确保您的包名称为`com.example.catagentprofile`。

1.  将保存位置设置为您要保存项目的位置。

1.  将其他所有内容保留为默认值，然后点击“完成”。

1.  确保你在`Project`窗格中处于`Android`视图下：![图 5.1：项目窗格中的 Android 视图](img/B15216_05_01.jpg)

图 5.1：项目窗格中的 Android 视图

1.  打开你的`AndroidManifest.xml`文件。像这样为你的应用程序添加互联网权限：

```kt
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.catagentprofile">
    <uses-permission android:name="android.permission.INTERNET" />
    <application ...>
        ...
    </application>
</manifest>
```

1.  要向你的应用程序添加 Retrofit 和标量转换器，打开应用程序模块的`build.gradle`（`Gradle Scripts` | `build.gradle (Module: app)`），并在`dependencies`块的任何位置添加以下行：

```kt
dependencies {
    ...
    implementation 'com.squareup.retrofit2:retrofit:2.9.0'
    implementation 'com.squareup.retrofit2:converter-scalars:2.9.0'
    ...
}
```

你的`dependencies`块现在应该看起来像这样：

```kt
dependencies {
    implementation "org.jetbrains.kotlin:kotlin-stdlib
      :$kotlin_version"
    implementation 'androidx.core:core-ktx:1.3.2'
    implementation 'androidx.appcompat:appcompat:1.2.0'
    implementation 'com.google.android.material:material:1.2.1'
    implementation 'androidx.constraintlayout:constraintlayout:2.0.4'
    implementation 'androidx.navigation:navigation-fragment
      -ktx:2.2.2'
    implementation 'androidx.navigation:navigation-ui-ktx:2.2.2'
    implementation 'com.squareup.retrofit2:retrofit:2.9.0'
    implementation 'com.squareup.retrofit2:converter-scalars:2.9.0'
    testImplementation 'junit:junit:4.+'
    androidTestImplementation 'androidx.test.ext:junit:1.1.2'
    androidTestImplementation 'androidx.test.espresso:espresso-core
      :3.3.0'
}
```

在你进行这个练习的时间和实施之间，一些依赖关系可能已经发生了变化。你应该仍然只添加前面代码块中加粗的行。这些将添加 Retrofit 和支持读取服务器响应作为单个字符串的功能。

注意

值得注意的是，Retrofit 现在要求至少 Android API 21 或 Java 8。

1.  在 Android Studio 中点击`Sync Project with Gradle Files`按钮。

1.  以`Text`模式打开你的`activity_main.xml`文件。

1.  为了能够使用标签来呈现最新的服务器响应，你需要为它分配一个 ID：

```kt
<TextView
    android:id="@+id/main_server_response"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="Hello World!"
    app:layout_constraintBottom_toBottomOf="parent"
    app:layout_constraintLeft_toLeftOf="parent"
    app:layout_constraintRight_toRightOf="parent"
    app:layout_constraintTop_toTopOf="parent" />
```

1.  在左侧的`Project`窗格中，右键单击你的应用程序包（`com.example.catagentprofile`），然后选择`New` | `Package`。

1.  将你的包命名为`api`。

1.  现在，在新创建的包（`com.example.catagentprofile.api`）上右键单击，然后选择`New` | `Kotlin File/Class`。

1.  给你的新文件命名为`TheCatApiService`。对于`Kind`，选择`Interface`。

1.  在`interface`块中添加以下内容：

```kt
interface TheCatApiService {
    @GET("images/search")
    fun searchImages(
        @Query("limit") limit: Int,
        @Query("size") format: String
    ) : Call<String>
}
```

这定义了图像搜索的端点。确保导入所有必需的 Retrofit 依赖项。

1.  打开你的`MainActivity`文件。

1.  在`MainActivity`类块的顶部添加以下内容：

```kt
class MainActivity : AppCompatActivity() {
lazy to make sure the instances are only created when needed.
```

1.  将`serverResponseView`添加为一个字段：

```kt
class MainActivity : AppCompatActivity() {
main_server_response ID the first time serverRespnseView is accessed and then keep a reference to it.
```

1.  现在，在`onCreate(Bundle?)`函数之后添加`getCatImageResponse()`函数：

```kt
override fun onCreate(savedInstanceState: Bundle?) {
    ...
}
private fun getCatImageResponse() {
val call = theCatApiService.searchImages(1, "full")
    call.enqueue(object : Callback<String> {
        override fun onFailure(call: Call<String>, t: Throwable) {
            Log.e("MainActivity", "Failed to get search results", t)
        }
        override fun onResponse(
            call: Call<String>,
            response: Response<String>
        ) {
            if (response.isSuccessful) {
                serverResponseView.text = response.body()
            } else {
                Log.e(
                    "MainActivity",
                    "Failed to get search results\n
                      ${response.errorBody()?.string() ?: ""}"
                )
            }
        }
    })
}
```

这个函数将发出搜索请求并处理可能的结果——成功的响应、错误的响应和任何其他抛出的异常。

1.  在`onCreate()`中调用`getCatImageResponse()`。这将在活动创建时触发调用：

```kt
override fun onCreate(savedInstanceState: Bundle?) {
    ...
    getCatImageResponse()
}
```

1.  添加缺失的导入。

1.  通过点击`Run 'app'`按钮或按下*Ctrl* + *R*来运行你的应用程序。在模拟器上，它应该看起来像这样：

![图 5.2：应用程序呈现服务器响应 JSON](img/B15216_05_02.jpg)

图 5.2：应用程序呈现服务器响应 JSON

因为每次运行应用程序都会进行一次新的调用并返回一个随机的响应，所以你的结果可能会有所不同。然而，无论你的结果如何，如果成功的话，它应该是一个 JSON 负载。接下来，我们将学习如何解析该 JSON 负载并从中提取我们想要的数据。

# 解析 JSON 响应

现在我们已经成功从 API 中检索到了 JSON 响应，是时候学习如何使用我们获取到的数据了。为了做到这一点，我们需要解析 JSON 负载。这是因为负载是一个表示数据对象的纯字符串，我们对该对象的特定属性感兴趣。如果你仔细看*图 5.2*，你可能会注意到 JSON 包含品种信息、图像 URL 和一些其他信息。然而，为了让我们的代码使用这些信息，首先我们需要提取它。

如介绍中所述，存在多个库可以解析 JSON 负载。最流行的是 Google 的 GSON（[`github.com/google/gson`](https://github.com/google/gson)）和最近更受欢迎的是 Square 的 Moshi（[`github.com/square/moshi`](https://github.com/square/moshi)）。Moshi 非常轻量级，这就是为什么我们选择在本章中使用它的原因。

JSON 库的作用是什么？基本上，它们帮助我们将数据类转换为 JSON 字符串（序列化）以及反之（反序列化）。这帮助我们与理解 JSON 字符串的服务器进行通信，同时允许我们在代码中使用有意义的数据结构。

要在 Retrofit 中使用 Moshi，我们需要将 Moshi Retrofit 转换器添加到我们的项目中。这是通过将以下行添加到我们应用程序的`build.gradle`文件的`dependencies`块中来完成的：

```kt
implementation 'com.squareup.retrofit2:converter-moshi:2.9.0'
```

由于我们将不再接受字符串作为响应，我们可以继续删除标量 Retrofit 转换器。

接下来，我们需要创建一个数据类，将服务器 JSON 响应映射到该类。一个惯例是给 API 响应数据类的名称添加后缀`Data`——所以我们将我们的数据类称为`ImageResultData`。另一个常见的后缀是`Entity`。

当我们设计服务器响应数据类时，我们需要考虑两个因素：JSON 响应的结构和我们的数据需求。第一个将影响我们的数据类型和字段名称，而第二个将允许我们省略我们当前不需要的字段。JSON 库知道它们应该忽略我们在数据类中未定义的字段中的数据。

JSON 库为我们做的另一件事是，如果它们恰好具有完全相同的名称，它们会自动将 JSON 数据映射到字段。虽然这是一个很好的功能，但也有问题。如果我们完全依赖它，我们的数据类（以及访问它们的代码）将与 API 命名紧密耦合。因为并非所有 API 都设计良好，您可能最终会得到毫无意义的字段名称，例如`fn`或`last`，或者不一致的命名。幸运的是，这个问题有解决办法。Moshi 为我们提供了一个`@field:Json`注解。它可以用来将 JSON 字段名称映射到有意义的字段名称：

```kt
data class UserData(
    @field:Json(name = "fn") val firstName: String,
    @field:Json(name = "last") val lastName: String
)
```

有人认为，出于一致性考虑，即使 API 名称与字段名称相同，也最好包括该注解。当字段名称足够清晰时，我们更喜欢直接转换的简洁性。当我们混淆我们的代码时，这种方法可能会受到挑战。如果我们这样做，我们要么排除我们的数据类，要么确保对所有字段进行注释。

虽然我们并不总是幸运地拥有适当记录的 API，但当我们拥有时，最好在设计我们的模型时咨询文档。我们的模型将是一个数据类，其中我们进行的所有调用的 JSON 数据将被解码。TheCatAPI 图像搜索端点的文档可以在[`docs.thecatapi.com/api-reference/images/images-search`](https://docs.thecatapi.com/api-reference/images/images-search)找到。您经常会发现文档是部分的或不准确的。如果是这种情况，您能做的最好的事情就是联系 API 的所有者，并要求他们更新文档。不幸的是，您可能不得不尝试使用端点。这是有风险的，因为未记录的字段或结构不能保证保持不变，所以在可能的情况下，尽量获取文档更新。

根据从上述链接获取的响应模式，我们可以定义我们的模型如下：

```kt
data class ImageResultData(
    @field:Json(name = "url") val imageUrl: String,
    val breeds: List<CatBreedData>
)
data class CatBreedData(
    val name: String,
    val temperament: String
)
```

请注意，响应结构实际上是结果列表。这意味着我们需要将我们的响应映射到`List<ImageResultData>`，而不仅仅是`ImageResultData`。

现在，我们需要更新`TheCatApiService`。现在我们可以有`Call<List<ImageResultData>>`，而不是`Call<String>`。

接下来，我们需要更新我们的 Retrofit 实例的构造。现在我们将有`MoshiConverterFactory`，而不是`ScalarsConverterFactory`。

最后，我们需要更新我们的回调，因为它不再处理字符串调用，而是处理`List<ImageResultData>`：

```kt
@GET("images/search")
fun searchImages(
    @Query("limit") limit: Int,
    @Query("size") format: String
) : Call<List<ImageResultData>>
```

## 练习 5.02：从 API 响应中提取图像 URL

因此，我们有一个作为字符串的服务器响应。现在，我们想从该字符串中提取图像 URL，并仅在屏幕上显示该 URL：

1.  打开应用程序的`build.gradle`文件，并用 Moshi 转换器替换标量转换器实现：

```kt
    implementation 'com.squareup.retrofit2:retrofit:2.9.0'
    implementation 'com.squareup.retrofit2:converter-moshi:2.9.0'
    testImplementation 'junit:junit:4.12'
```

1.  单击“与 Gradle 文件同步”按钮。

1.  在您的应用程序包（`com.example.catagentprofile`）下，创建一个`model`包。

1.  在`com.example.catagentprofile.model`包中，创建一个名为`CatBreedData`的新的 Kotlin 文件。

1.  使用以下内容填充新创建的文件：

```kt
package com.example.catagentprofile.model
data class CatBreedData(
    val name: String,
    val temperament: String
)
```

1.  接下来，在同一个包下创建`ImageResultData`。

1.  将其内容设置为以下内容：

```kt
package com.example.catagentprofile.model
import com.squareup.moshi.Json
data class ImageResultData(
    @field:Json(name = "url") val imageUrl: String,
val breeds: List<CatBreedData>
)
```

1.  打开`TheCatApiService`文件并更新`searchImages`返回类型：

```kt
    @GET("images/search")
    fun searchImages(
        @Query("limit") limit: Int,
        @Query("size") format: String
    ) : Call<List<ImageResultData>>
```

1.  最后，打开`MainActivity`。

1.  更新 Retrofit 初始化块以使用 Moshi 转换器进行反序列化 JSON：

```kt
    private val retrofit by lazy {
        Retrofit.Builder()
            .baseUrl("https://api.thecatapi.com/v1/")
            .addConverterFactory(MoshiConverterFactory.create())
            .build()
    }
```

1.  更新`getCatImageResponse()`函数以处理`List<ImageResultData>`的请求和响应：

```kt
private fun getCatImageResponse() {
    val call = theCatApiService.searchImages(1, "full")
    call.enqueue(object : Callback<List<ImageResultData>> {
        override fun onFailure(call: Call<List<ImageResultData>>,           t: Throwable) {
            Log.e("MainActivity", "Failed to get search results",             t)
        }
        override fun onResponse(
           call: Call<List<ImageResultData>>,
            response: Response<List<ImageResultData>>
        ) {
            if (response.isSuccessful) {
                val imageResults = response.body()
                val firstImageUrl = imageResults?.firstOrNull()                  ?.imageUrl ?: "No URL"
                serverResponseView.text = "Image URL:                   $firstImageUrl"
            } else {
                Log.e(
                    "MainActivity",
                    "Failed to get search                        results\n${response.errorBody()?.string()                          ?: ""}"
                )
            }
        }
    })
}
```

1.  现在，您需要检查的不仅是成功的响应，还要确保至少有一个`ImageResultData`实例。然后，您可以读取该实例的`imageUrl`属性并将其呈现给用户。

1.  运行您的应用程序。现在它应该看起来像下面这样：![图 5.3：应用程序呈现解析后的图像 URL](img/B15216_05_03.jpg)

图 5.3：应用程序呈现解析后的图像 URL

1.  由于 API 响应的随机性，您的 URL 可能会不同。

您现在已成功从 API 响应中提取了特定属性。接下来，我们将学习如何从 API 提供的 URL 加载图像。

# 从远程 URL 加载图像

我们刚学会了如何从 API 响应中提取特定数据。很多时候，这些数据将包括我们想要呈现给用户的图像的 URL。这需要相当多的工作。首先，您必须从 URL 中获取图像作为二进制流。然后，您需要将该二进制流转换为图像（可以是 GIF、JPEG 或其他几种图像格式之一）。然后，您需要将其转换为位图实例，可能调整大小以使用更少的内存。

您可能还希望在此时对其进行其他转换。然后，您需要将其设置为`ImageView`。听起来是很多工作，不是吗？幸运的是，有一些库可以为我们完成所有这些工作（甚至更多）。最常用的库是 Square 的**Picasso**（[`square.github.io/picasso/`](https://square.github.io/picasso/)）和 Bump Technologies 的**Glide**（[`github.com/bumptech/glide`](https://github.com/bumptech/glide)）。Facebook 的**Fresco**（[`frescolib.org/`](https://frescolib.org/)）相对不太受欢迎。我们将继续使用 Glide，因为它始终是加载图像的两者中更快的一个，无论是来自互联网还是缓存。值得注意的是 Picasso 更轻量级，所以这是一个权衡，两个库都非常有用。

要在项目中包含 Glide，请将其添加到应用程序的`build.gradle`文件的`dependencies`块中：

```kt
dependencies {
    implementation 'com.github.bumptech.glide:glide:4.10.0'
    ...
}
```

实际上，因为我们可能在以后改变主意，这是一个很好的机会来将具体库抽象出来，以拥有更简单的接口。所以，让我们从定义我们的`ImageLoader`接口开始：

```kt
interface ImageLoader {
    fun loadImage(imageUrl: String, imageView: ImageView)
}
```

这是一个天真的实现。在生产实现中，您可能希望添加参数（或多个函数）以支持不同的裁剪策略或具有加载状态。

我们的接口实现将依赖于 Glide，因此看起来会像这样：

```kt
class GlideImageLoader(private val context: Context) : ImageLoader {
    override fun loadImage(imageUrl: String, imageView: ImageView) {
        Glide.with(context)
            .load(imageUrl)
            .centerCrop()
            .into(imageView)
    }
}
```

我们在类名前加上`Glide`以区别于其他潜在的实现。使用`context`构建`GlideImageLoader`允许我们实现清晰的`loadImage(String, ImageView)`接口，而不必担心 Glide 所需的上下文，这实际上是 Glide 对 Android 上下文的智能处理。这意味着我们可以针对`Activity`和`Fragment`范围有单独的实现，而 Glide 会知道何时图像加载请求超出范围。

由于我们尚未在布局中添加`ImageView`，现在让我们这样做：

```kt
<TextView
    ...
    app:layout_constraintBottom_toTopOf="@+id/main_profile_image"
    ... />
<ImageView
    android:id="@+id/main_profile_image"
    android:layout_width="150dp"
    android:layout_height="150dp"
    app:layout_constraintBottom_toBottomOf="parent"
    app:layout_constraintEnd_toEndOf="parent"
app:layout_constraintStart_toStartOf="parent"
    app:layout_constraintTop_toBottomOf="@+id/main_server_response" />
```

这将在我们的`TextView`下方添加一个 ID 为`main_profile_image`的`ImageView`。

现在我们可以在`MainActivity`中创建`GlideImageLoader`的实例：

```kt
private val imageLoader: ImageLoader by lazy { GlideImageLoader(this) }
```

在生产应用中，您将注入依赖项，而不是内联创建。

接下来，我们告诉我们的 Glide 加载器加载图像，并且一旦加载完成，就在提供的`ImageView`内居中裁剪它。这意味着图像将被放大或缩小以完全填充`ImageView`，任何多余的内容都将被裁剪掉。由于我们之前已经获得了图像 URL，所以我们需要做的就是调用：

```kt
val firstImageUrl = imageResults?.firstOrNull()?.imageUrl ?: ""
if (!firstImageUrl.isBlank()) {
    imageLoader.loadImage(firstImageUrl, profileImageView)
} else {
    Log.d("MainActivity", "Missing image URL")
}
```

我们必须确保结果包含一个不为空或由空格组成的字符串（在前面的代码块中使用`isBlank()`）。然后，我们可以安全地将 URL 加载到我们的`ImageView`中。然后就完成了。如果现在运行我们的应用程序，应该会看到类似以下的东西：

![图 5.4：服务器响应图像 URL 与实际图像](img/B15216_05_04.jpg)

图 5.4：服务器响应图像 URL 与实际图像

请记住，API 返回随机结果，因此实际图像可能会有所不同。如果我们幸运的话，甚至可能会得到一个动画 GIF，然后我们就会看到它动画。

## 练习 5.03：从获取的 URL 加载图像

在上一个练习中，我们从 API 响应中提取了图像 URL。现在，我们将使用该 URL 从网络获取图像并在我们的应用程序中显示它：

1.  打开应用程序的`build.gradle`文件并添加 Glide 依赖项：

```kt
dependencies {
    ... 
    implementation 'com.squareup.retrofit2:converter-moshi:2.9.0'
    implementation 'com.github.bumptech.glide:glide:4.11.0'
    testImplementation 'junit:junit:4.12'
    ...
}
```

将项目与 Gradle 文件同步。

1.  在左侧的`Project`面板上，右键单击您的项目包名称（`com.example.catagentprofile`），然后选择`New` | `Kotlin File/Class`。

1.  在`Name`字段中填写`ImageLoader`。对于`Kind`，选择`Interface`。

1.  打开新创建的`ImageLoader.kt`文件，并像这样更新它：

```kt
interface ImageLoader {
    fun loadImage(imageUrl: String, imageView: ImageView)
}
```

这将是应用程序中任何图像加载器的接口。

1.  右键单击项目包名称，然后再次选择`New` | `Kotlin File/Class`。

1.  将新文件命名为`GlideImageLoader`，并选择`Class`作为`Kind`。

1.  更新新创建的文件：

```kt
class GlideImageLoader(private val context: Context) : ImageLoader {
override fun loadImage(imageUrl: String, imageView: ImageView) {
        Glide.with(context)
            .load(imageUrl)
            .centerCrop()
            .into(imageView)
    }
}
```

1.  打开`activity_main.xml`。

像这样更新它：

```kt
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">
    <TextView
        android:id="@+id/main_server_response"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Hello World!"
        app:layout_constraintBottom_toImageView named main_profile_image below your TextView.
```

1.  打开`MainActivity.kt`文件。

1.  在您的类顶部添加一个新添加的`ImageView`字段：

```kt
private val serverResponseView: TextView
    by lazy { findViewById(R.id.main_server_response) } 
private val profileImageView: ImageView
by lazy { findViewById(R.id.main_profile_image) } 
```

1.  在`onCreate(Bundle?)`函数的上方定义`ImageLoader`：

```kt
private val imageLoader: ImageLoader by lazy { GlideImageLoader(this) }
override fun onCreate(savedInstanceState: Bundle?) {
```

1.  像这样更新您的`getCatImageResponse()`函数：

```kt
private fun getCatImageResponse() {
    val call = theCatApiService.searchImages(1, "full")
    call.enqueue(object : Callback<List<ImageResultData>> {
        override fun onFailure(call: Call<List<ImageResultData>>,           t: Throwable) {
            Log.e("MainActivity", "Failed to get search results", t)
        }
        override fun onResponse(
            call: Call<List<ImageResultData>>,
            response: Response<List<ImageResultData>>
        ) {
            if (response.isSuccessful) {
                val imageResults = response.body()
                val firstImageUrl =                   imageResults?.firstOrNull()?.imageUrl ?: ""
                if (firstImageUrl.isNotBlank()) {
imageLoader.loadImage(firstImageUrl, 
                      profileImageView)
                } else {
                    Log.d("MainActivity", "Missing image URL")
                }
                serverResponseView.text = "Image URL: $firstImageUrl"
            } else {
                Log.e(
                    "MainActivity",
                    "Failed to get search results\n
                      ${response.errorBody()?.string() ?: ""}"
                )
            }
        }
    })
}
```

1.  现在，一旦您有一个非空的 URL，它将被加载到`profileImageView`中。

1.  运行应用程序：![图 5.5：练习结果-显示随机图像及其源 URL](img/B15216_05_05.jpg)

图 5.5：练习结果-显示随机图像及其源 URL

以下是额外的步骤。

1.  像这样更新您的布局：

```kt
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">
    <TextView
        android:id="@+id/main_agent_breed_label"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:padding="16dp"
        android:text="Agent breed:"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />
    <TextView
        android:id="@+id/main_agent_breed_value"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:paddingTop="16dp"
        app:layout_constraintStart_toEndOf=
          "@+id/main_agent_breed_label"
        app:layout_constraintTop_toTopOf=
          "@+id/main_agent_breed_label" />
    <ImageView
        android:id="@+id/main_profile_image"
        android:layout_width="150dp"
        android:layout_height="150dp"
        android:layout_margin="16dp"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf=
          "@+id/main_agent_breed_label" />
</androidx.constraintlayout.widget.ConstraintLayout>
```

这将添加一个`Agent breed`标签并整理视图布局。现在，您的布局看起来更像是一个合适的猫代理配置文件应用程序。

1.  在`MainActivity.kt`中，找到以下行：

```kt
private val serverResponseView: TextView
    by lazy { findViewById(R.id.main_server_response) } 
```

用以下内容替换该行以查找新的名称字段：

```kt
private val agentBreedView: TextView
    by lazy { findViewById(R.id.main_agent_breed_value) }
```

1.  更新`getCatImageResponse()`如下：

```kt
private fun getCatImageResponse() {
    val call = theCatApiService.searchImages(1, "full")
    call.enqueue(object : Callback<List<ImageResultData>> {
        override fun onFailure(call: Call<List<ImageResultData>>,           t: Throwable) {
            Log.e("MainActivity", "Failed to get search results", t)
        }
        override fun onResponse(
            call: Call<List<ImageResultData>>,
            response: Response<List<ImageResultData>>
        ) {
            if (response.isSuccessful) {
                val imageResults = response.body()
                val firstImageUrl =                   imageResults?.firstOrNull()?.imageUrl ?: ""
                if (!firstImageUrl.isBlank()) {
                    imageLoader.loadImage(firstImageUrl, 
                      profileImageView)
                } else {
                    Log.d("MainActivity", "Missing image URL")
                }
                agentBreedView.text =
                    imageResults?.firstOrNull()?.breeds?                      .firstOrNull()?.name ?: "Unknown"
            } else {
                Log.e(
                  "MainActivity",
                  "Failed to get search results\n
                    ${response.errorBody()?.string() ?:""}"
                )
            }
        }
    })
}
```

这是为了将 API 返回的第一个品种加载到`agentNameView`中，如果没有则回退到`Unknown`。

1.  在撰写本文时，TheCatAPI 中没有太多带有品种数据的图片。但是，如果您运行应用程序足够多次，最终会看到类似这样的东西：

![图 5.6：显示猫代理图像和品种](img/B15216_05_06.jpg)

图 5.6：显示猫代理图像和品种

在本章中，我们学习了如何从远程 API 获取数据。然后，我们学习了如何处理这些数据并从中提取我们需要的信息。最后，我们学习了如何在给定图像 URL 时在屏幕上呈现图像。

在接下来的活动中，我们将应用我们的知识开发一个应用程序，告诉用户纽约的当前天气，并向用户展示相关的天气图标。

## 活动 5.01：显示当前天气

假设我们想要构建一个应用程序，显示纽约的当前天气。此外，我们还想显示代表当前天气的图标。

这个活动旨在创建一个应用程序，它会轮询一个 API 端点以获取 JSON 格式的当前天气，将这些数据转换为本地模型，并使用该模型呈现当前天气。它还会提取代表当前天气的图标的 URL，并获取该图标以在屏幕上显示。

我们将使用免费的 OpenWeatherMap.org API 来完成这个活动的目的。文档可以在[`www.metaweather.com/api/`](https://www.metaweather.com/api/)找到。要注册 API 令牌，请转到[`home.openweathermap.org/users/sign_up`](https://home.openweathermap.org/users/sign_up)。您可以在[`home.openweathermap.org/api_keys`](https://home.openweathermap.org/api_keys)找到您的密钥，并根据需要生成新的密钥。

步骤如下：

1.  创建一个新的应用程序。

1.  授予应用程序互联网权限，以便能够进行 API 和图像请求。

1.  将 Retrofit、Moshi 转换器和 Glide 添加到应用程序中。

1.  更新应用程序布局，以支持以文本形式（简短和长描述）呈现天气以及天气图标图像。

1.  定义模型。创建包含服务器响应的类。

1.  为 OpenWeatherMap API 添加 Retrofit 服务，[`api.openweathermap.org/data/2.5/weather`](https://api.openweathermap.org/data/2.5/weather)。

1.  使用 Moshi 转换器创建一个 Retrofit 实例。

1.  调用 API 服务。

1.  处理成功的服务器响应。

1.  处理不同的失败场景。

预期输出如下：

![图 5.7：最终的天气应用程序](img/B15216_05_07.jpg)

图 5.7：最终的天气应用程序

注意

此活动的解决方案可以在此处找到：http://packt.live/3sKj1cp

# 总结

在本章中，我们学会了如何使用 Retrofit 从 API 获取数据。然后我们学会了如何使用 Moshi 处理 JSON 响应，以及纯文本响应。我们还看到了如何处理不同的错误场景。

后来我们学会了如何使用 Glide 从 URL 加载图像，以及如何通过`ImageView`呈现给用户。

有很多流行的库可以从 API 中获取数据，以及加载图像。我们只涵盖了一些最流行的库。您可能想尝试一些其他库，找出哪些最适合您的目的。

在下一章中，我们将介绍`RecyclerView`，这是一个强大的 UI 组件，我们可以用它来向用户呈现项目列表。
