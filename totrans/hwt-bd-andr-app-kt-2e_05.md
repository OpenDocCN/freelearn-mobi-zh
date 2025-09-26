# 5

# 必要的库：Retrofit、Moshi和Glide

在本章中，我们将介绍向应用用户提供从远程服务器获取的动态内容的步骤。您将了解用于检索和处理这些动态数据的不同库。

到本章结束时，您将能够使用Retrofit从网络端点获取数据，使用Moshi将JSON有效载荷解析为Kotlin数据对象，并使用Glide将图片加载到`ImageView`中。

在上一章中，我们学习了如何在应用中实现导航。在本章中，我们将学习如何在用户在应用中导航时向用户展示动态内容。

本章我们将涵盖以下主题：

+   介绍REST、API、JSON和XML

+   从网络端点获取数据

+   解析JSON响应

+   从远程URL加载图片

# 技术要求

本章所有练习和活动的完整代码可在GitHub上找到，链接为[https://packt.link/Uqtjm](https://packt.link/Uqtjm)。

# 介绍REST、API、JSON和XML

呈现给用户的数据可以来自不同的来源。它可能被硬编码到应用中，但这会带来限制。要更改硬编码的数据，我们必须发布应用的更新。某些数据，如货币汇率、资产实时可用性和当前天气，由于其本质不能被硬编码。其他数据可能会过时，如应用的使用条款。

在这种情况下，您通常会从服务器获取相关数据。为这种数据提供服务最常见的一种架构是**表示状态转移**（**REST**）架构。REST架构由一组六个约束定义：客户端-服务器架构、无状态、可缓存性、分层系统、按需代码（可选）和统一接口。

注意

想了解更多关于REST的信息，请访问[https://packt.link/YsSRV](https://packt.link/YsSRV)。

当应用于**应用程序编程接口**（**API**）的Web服务时，我们得到基于**超文本传输协议**（**HTTP**）的RESTful API。HTTP协议是万维网数据通信的基础，它托管在互联网上，并通过互联网访问。它是全球服务器用于以HTML文档、图片、样式表等形式向用户提供服务所使用的协议。

注意

关于这个主题的一篇有趣的文章可以在[https://developer.mozilla.org/en-US/docs/Web/HTTP/Overview](https://developer.mozilla.org/en-US/docs/Web/HTTP/Overview)找到。

RESTful API依赖于标准的HTTP方法——`GET`、`POST`、`PUT`、`DELETE`和`PATCH`——来获取和转换数据。这些方法允许我们从远程服务器获取、存储、删除和更新数据实体。

我们可以依赖内置的 Java `HttpURLConnection` 类来执行这些 HTTP 方法。或者，我们可以使用像 `OkHttp` 这样的库，它提供了额外的功能，如 gzip（压缩和解压缩）、重定向、重试以及同步和异步调用。有趣的是，从 Android 4.4 开始，`HttpURLConnection` 只是 `OkHttp` 的包装器。如果我们选择 `OkHttp`，我们不妨选择 **Retrofit**（正如本章所做的那样），这是当前行业标准。然后我们可以从其类型安全中受益，这对于处理 REST 调用来说更适合。

最常见的情况下，数据是通过 **JavaScript 对象表示法**（**JSON**）表示的。JSON 是一种基于文本的数据传输格式。正如其名所示，它源自 JavaScript。然而，它已经成为了数据传输中最流行的标准之一，并且其最现代的编程语言都有库来编码或解码数据到或从 JSON。

一个简单的 JSON 有效负载可能看起来像这样：

[PRE0]

另一个由 RESTful 服务常用的数据结构是 **可扩展标记语言**（**XML**），它以人类和机器可读的格式编码文档。XML 比JSON更冗长。在 XML 中与之前相同的数据结构看起来可能像这样：

[PRE1]

在本章中，我们将专注于 JSON。

当获取 JSON 有效负载时，我们实际上接收的是一个字符串。要将该字符串转换为数据对象，我们有几种选择，最受欢迎的包括 `org.json` 包等库。由于其轻量级特性，我们将关注 Moshi。

最后，我们将探讨从网络加载图片。这样做将允许我们提供最新的图片，并为用户的设备加载正确的图片。它还将允许我们仅在需要时加载图片，从而保持我们的 APK 大小更小。

# 从网络端点获取数据

为了本节的目的，我们将使用 The Cat API ([https://thecatapi.com/](https://thecatapi.com/))。这个 RESTful API 为我们提供了关于，嗯……猫的大量数据。

要开始，我们需要创建一个新的项目。然后我们必须授予我们的应用程序互联网访问权限。这通过在 `AndroidManifest.xml` 文件中 `Application` 标签之前添加以下代码来完成：

[PRE2]

接下来，我们需要设置我们的应用程序以包含 Retrofit。Retrofit 是由 Square 提供的一个类型安全的库，它建立在 `OkHttp` HTTP 客户端之上。Retrofit 通过提供与几个解析库的集成来帮助我们生成 **统一资源定位符**（**URL**），这是我们想要访问的服务器端点的地址。它还通过简化 JSON 有效负载的解码使发送数据到服务器变得更容易。

注意

你可以在这里了解更多关于 Retrofit 的信息 [https://square.github.io/retrofit/](https://square.github.io/retrofit/)。

要将Retrofit添加到我们的项目中，我们需要将以下代码添加到我们应用`build.gradle`文件的`dependencies`块中：

[PRE3]

注意

你可以在这里找到最新版本[https://github.com/square/retrofit](https://github.com/square/retrofit)。

在我们的项目中包含Retrofit之后，我们可以继续设置它。

首先，为了访问一个HTTP(S)端点，我们首先定义与该端点的合约。访问`https://api.thecatapi.com/v1/images/search`端点的合约看起来是这样的：

[PRE4]

这里有一些需要注意的事项。首先，你会注意到合约是以接口的形式实现的。这就是定义Retrofit合约的方式。接下来，你会注意到接口的名称暗示了这个接口最终可以覆盖对`TheCatAPIService`的所有调用。

很不幸，Square将这些合约的传统后缀选为`Service`，因为在Android世界中，术语`service`有不同的含义，正如你在[*第8章*](B19411_08.xhtml#_idTextAnchor471)中将要看到的，*服务、WorkManager和通知*。尽管如此，这已经成为了一种惯例。

为了定义我们的端点，我们首先使用适当的注解声明将要使用的调用方法——在我们的例子中，是`@GET`。传递给注解的参数是要访问的端点路径。你会注意到`https://api.thecatapi.com/v1/`已经被从该路径中移除。

这是因为这是`TheCatAPI`所有端点的通用地址，因此它将在构造时传递给我们的Retrofit实例。接下来，我们为我们的函数选择一个有意义的名称——在这种情况下，我们将调用图像搜索端点，所以`searchImages`似乎很合适。`searchImages`函数的参数定义了我们在调用API时可以传递的值。

我们有几种不同的方式可以将数据传输到API。`@Query`允许我们定义添加到请求URL查询中的值（这是URL中问号之后的可选部分）。它需要一个键值对（在我们的例子中，我们有`limit`和`size`）和一个数据类型。如果数据类型不是字符串，该类型的值将被转换为字符串。任何传递的值都将为我们进行URL编码。

另一种方式是使用`@Path`。这个注解可以用来用提供的值替换我们路径中用花括号包裹的标记。`@Header`、`@Headers`和`@HeaderMap`注解将允许我们在请求中添加或删除HTTP头。`@Body`可以用来在`POST`和`PUT`请求的体中传递内容。

最后，我们有一个返回类型。为了在这个阶段保持简单，我们将接受响应作为字符串。我们将我们的字符串包装在 `Call` 接口内。`Call` 是 Retrofit 执行网络请求的机制，可以是同步的（通过 `execute()`）或异步的（通过 `enqueue(Callback)`）。当使用协程时，我们可以使函数成为一个 `suspend` 函数，并省略 `Call` 包装器（有关协程的更多信息，请参见 [*第 14 章*](B19411_14.xhtml#_idTextAnchor751)，*协程和 Flow*）。

在我们的合约定义之后，我们可以让 Retrofit 实现我们的服务接口：

[PRE5]

如果我们尝试运行带有此代码的应用程序，我们的应用程序将因 `IllegalArgumentException` 而崩溃。这是因为 Retrofit 需要我们告诉应用程序如何处理服务器响应字符串。这个过程是通过 Retrofit 调用 `ConverterFactory` 实例到我们的 `retrofit` 实例来完成的，我们需要添加以下内容：

[PRE6]

为了让我们的项目识别 `ScalarsConverterFactory`，我们需要通过在应用程序的 `build.gradle` 文件中添加另一个依赖项来更新它：

[PRE7]

现在，我们可以通过调用 `val call = theCatApiService.searchImages(1, "full")` 来获取一个 `Call` 实例。通过这种方式获得的实例，我们可以通过调用 `call.enqueue(Callback)` 来执行异步请求。

我们的 `Callback` 实现将有两个方法：`onFailure(Call, Throwable)` 和 `onResponse(Call, Response)`。请注意，如果调用 `onResponse`，我们并不保证会有一个成功的响应。`onResponse` 在我们成功从服务器接收到 *任何* 响应且没有发生意外异常时被调用。

因此，为了确认响应是成功的，我们应该检查 `response.isSuccessful` 属性。在发生网络错误或过程中某个地方出现意外异常的情况下，将调用 `onFailure` 函数。

那么，我们应该在哪里实现 Retrofit 代码呢？在干净的架构中，数据由仓库提供。仓库反过来又拥有数据源。其中一个数据源可以是一个网络数据源。这就是我们将实现我们的网络调用的地方。然后，我们的 ViewModels 将通过用例从仓库请求数据。

在 **模型-视图-视图模型**（**MVVM**）的情况下，视图模型是视图的抽象，它公开属性和命令。

对于我们的实现，我们将通过在 Activity 中实例化 Retrofit 和服务来简化这个过程。这不是一个好的实践。*不要在生产应用中这样做*。它扩展性不好，并且很难测试。相反，采用一种将你的视图与你的业务逻辑和数据解耦的架构。参见 [*第 15 章*](B19411_15.xhtml#_idTextAnchor789)，*架构模式*，以获取一些想法。

## 练习 5.01 – 从 API 读取数据

在接下来的章节中，我们将开发一个为一家虚构的秘密机构开发的应用程序，该机构在全球范围内拥有特工网络，拯救世界免受无数危险。这个秘密机构相当独特：它运营着秘密的猫特工。

在这个练习中，我们将创建一个应用，它从 The Cat API 中随机展示一个秘密猫特工。在你能够向用户展示 API 数据之前，你首先必须获取这些数据。让我们从这里开始：

1.  创建一个新的 **空活动** 项目（**文件** | **新建** | **新建项目** | **空活动**）。点击 **下一步**。

1.  将你的应用程序命名为 `Cat Agent Profile`。

1.  确保你的包名是 `com.example.catagentprofile`。

1.  将 **保存位置** 设置为你想要保存项目的地方。

1.  将其他所有内容保留为默认值并点击 **完成**。

1.  确保你处于 **Android** 视图在你的 **项目** 窗格中：

![图 5.1 – 项目窗格中的 Android 视图](img/B19411_05_01.jpg)

图 5.1 – 项目窗格中的 Android 视图

1.  打开你的 `AndroidManifest.xml` 文件。向你的应用添加如下互联网权限：

    [PRE8]

1.  要将 Retrofit 和标量转换器添加到你的应用中，打开 `build.gradle` 应用模块（`Gradle 脚本` | `build.gradle (Module: app)`），并在 `dependencies` 块内添加以下行：

    [PRE9]

你的 `dependencies` 块现在可能看起来像这样：

[PRE10]

在撰写本文和执行此练习之间，一些依赖项可能已更改。你仍然只需添加前面代码块中加粗的行。这些行将添加 Retrofit 和读取服务器响应作为单个字符串的支持。

备注

值得注意的是，Retrofit 现在至少需要 Android API 21 或 Java 8。

1.  点击 Android Studio 中的 **同步项目与 Gradle 文件** 按钮。

1.  以 **文本** 模式打开你的 `activity_main.xml` 文件。

1.  为了能够使用你的标签来展示最新的服务器响应，你需要给它分配一个 ID：

    [PRE11]

1.  在 `com.example.catagentprofile` 中，然后选择 **新建** | **包**。

1.  将你的包命名为 `api`。

1.  现在，在 **项目** 窗格中右键单击新创建的包（`com.example.catagentprofile.api`），然后选择 **新建** | **Kotlin 文件/类**。

1.  将你的新文件命名为 `TheCatApiService`。对于 **类型**，选择 **接口**。

1.  将以下内容添加到 `interface` 块中：

    [PRE12]

这定义了图像搜索端点。请确保导入所有必需的 Retrofit 依赖项。

1.  打开你的 `MainActivity` 文件。

1.  在 `MainActivity` 类块顶部添加以下内容：

    [PRE13]

这将实例化 Retrofit 和 API 服务。我们使用 `lazy` 确保实例仅在需要时创建。

1.  添加 `serverResponseView` 作为字段：

    [PRE14]

这将在第一次访问 `serverResponseView` 时查找具有 `main_server_response` ID 的视图，并保留对该视图的引用。

1.  现在，在 `onCreate(Bundle?)` 函数之后添加 `getCatImageResponse()` 函数：

    [PRE15]

此函数将触发搜索请求并处理可能的结果——成功响应、错误响应以及任何其他抛出的异常。

1.  在 `onCreate()` 中调用 `getCatImageResponse()`。这将触发调用，一旦活动创建完成：

    [PRE16]

1.  添加缺失的导入。

1.  通过点击**运行‘app’**按钮或按*Ctrl* + *R*来运行你的应用。在模拟器上，它应该看起来像这样：

![图5.2 – 应用展示服务器响应的JSON](img/B19411_05_02.jpg)

图5.2 – 应用展示服务器响应的JSON

因为每次运行你的应用时，都会发起一个新的调用，并返回一个随机响应，所以你的结果可能会不同。然而，无论结果如何，如果成功，它应该是一个JSON负载。接下来，我们将学习如何解析这个JSON负载并从中提取我们想要的数据。

# 解析JSON响应

现在我们已经成功从API中检索到了JSON响应，是时候学习如何使用我们获得的数据了。为此，我们需要解析JSON负载。这是因为负载是一个表示数据对象的普通字符串，而我们感兴趣的是该对象的具体属性。如果你仔细看*图5**.2*，你可能会注意到JSON包含品种信息、一个图片URL和一些其他信息。然而，为了我们的代码能够使用这些信息，首先，我们必须提取它们。

如介绍中所述，存在多个库可以帮助我们解析JSON负载。最受欢迎的是Google的GSON（见[https://github.com/google/gson](https://github.com/google/gson)）和更近期的Square的Moshi（见[https://github.com/square/moshi](https://github.com/square/moshi)）。Moshi非常轻量级，这就是为什么我们选择在本章中使用它。

JSON库的作用是什么？基本上，它们帮助我们将数据类转换为JSON字符串（序列化）以及相反的过程（反序列化）。这有助于我们与理解JSON字符串的服务器进行通信，同时允许我们在代码中使用有意义的结构化数据。

要在Retrofit中使用Moshi，我们需要将Moshi Retrofit转换器添加到我们的项目中。这是通过将以下行添加到我们应用的`build.gradle`文件中的`dependencies`块来完成的：

[PRE17]

由于我们不再将响应作为字符串处理，我们可以继续移除Retrofit的标量转换器。接下来，我们需要创建一个数据类来映射服务器JSON响应。一个约定是将API响应数据类的名称后缀为`Data`——所以我们将我们的数据类命名为`ImageResultData`。另一个常见的后缀是`Entity`。

当我们设计服务器响应的数据类时，需要考虑两个因素：JSON响应的结构和我们的数据需求。第一个将影响我们的数据类型和字段名称，而第二个将允许我们省略当前不需要的字段。JSON库会忽略我们在数据类中未定义的字段中的数据。

JSON库为我们做的另一件事是自动将JSON数据映射到字段，如果它们恰好有相同的名称。虽然这是一个很好的特性，但它也带来了风险。如果我们完全依赖它，我们的数据类（以及访问它们的代码）将与API命名紧密耦合。

由于并非所有 API 都设计得很好，你可能会遇到没有意义的字段名称，如 `fn` 或 `last`，或者不一致的命名。幸运的是，有一个解决方案可以解决这个问题。Moshi 提供了一个 `@Json` 注解。它可以用来将 JSON 字段名称映射到一个有意义的字段名称：

[PRE18]

使用 `field:` 前缀来确保生成的 Java 字段被正确注解。

有些人认为，即使 API 名称与字段名称相同，为了保持一致性，最好还是包含该注解。我们更喜欢当字段名称足够清晰时直接转换的简洁性。当我们的代码被混淆时，这种方法可能会受到挑战。如果我们这样做，我们必须排除我们的数据类或者确保所有字段都被注解。

虽然我们并不总是有幸拥有文档齐全的 API，但当我们确实有文档时，在设计我们的模型时最好查阅文档。我们的模型将是一个数据类，我们将在其中解码我们发出的所有调用的 JSON 数据。The Cat API 的图像搜索端点的文档可以在 [https://protect-eu.mimecast.com/s/d7uqCWwlKf56Q17s6kKb5?domain=developers.thecatapi.com/](https://protect-eu.mimecast.com/s/d7uqCWwlKf56Q17s6kKb5?domain=developers.thecatapi.com/) 找到。

你经常会发现文档是不准确或错误的。如果这种情况发生，最好的办法是联系 API 的所有者，并要求他们更新文档。不幸的是，你可能不得不求助于对端点的实验。这是有风险的，因为未记录的字段或结构不保证保持不变，所以当可能时，尽量让文档得到更新。

根据从先前的链接中获得的响应模式，我们可以定义我们的模型如下：

[PRE19]

注意，响应结构是结果列表。这意味着我们需要将响应映射到 `List<ImageResultData>`，而不仅仅是 `ImageResultData`。现在，我们需要更新 `TheCatApiService`。我们可以不再使用 `Call<String>`，而是现在可以使用 `Call<List<ImageResultData>>`。

接下来，我们需要更新我们的 Retrofit 实例的构建。我们将不再使用 `ScalarsConverterFactory`，而是现在将使用 `MoshiConverterFactory`。最后，我们需要更新我们的回调，因为它不再应该处理字符串调用，而是应该处理 `List<ImageResultData>`：

[PRE20]

## 练习 5.02 – 从 API 响应中提取图像 URL

因此，我们有一个作为字符串的服务器响应。现在，我们想要从该字符串中提取图像 URL，并在屏幕上仅显示该 URL：

1.  打开应用的 `build.gradle` 文件，并将标量转换器的实现替换为 Moshi 转换器的一个：

    [PRE21]

1.  点击**同步项目与 Gradle 文件**按钮。

1.  在您的应用包（`com.example.catagentprofile`）下创建一个 `model` 包。

1.  在 `com.example.catagentprofile.model` 包中，创建一个名为 `CatBreedData` 的新 Kotlin 文件。

1.  在新创建的文件中填充以下内容：

    [PRE22]

1.  接下来，在相同的包下创建 `ImageResultData`。

1.  设置其内容如下：

    [PRE23]

1.  打开`TheCatApiService`文件并更新`searchImages`的返回类型：

    [PRE24]

1.  最后，打开`MainActivity`。

1.  更新Retrofit初始化块以使用Moshi转换器反序列化JSON：

    [PRE25]

1.  更新`getCatImageResponse()`函数以处理`List<ImageResultData>`请求和响应：

    [PRE26]

1.  现在，您需要检查是否有成功的响应，并且至少有一个`ImageResultData`实例。然后，您可以读取该实例的`imageUrl`属性并将其展示给用户。

1.  运行您的应用程序。现在它应该看起来像以下这样：

![图5.3 – 应用展示解析后的图片URL](img/B19411_05_03.jpg)

图5.3 – 应用展示解析后的图片URL

1.  再次强调，由于API响应的随机性，您的URL可能不同。

您现在已成功从API响应中提取了特定的属性。接下来，我们将学习如何从API提供的URL加载图片。

# 从远程URL加载图片

我们刚刚学习了如何从API响应中提取数据。这些数据通常包括我们想要展示给用户的图片的URL。要实现这一点，涉及的工作量相当大。首先，您必须从URL获取图片作为二进制流。然后，您需要将那个二进制流转换成图片（它可能是GIF、JPEG或其他几种图像格式之一）。

然后，您需要将其转换为位图实例，可能需要调整大小以节省内存。您还可能想在此处应用其他转换。然后，您需要将其设置到`ImageView`。

这听起来像是一项繁重的工作，不是吗？幸运的是，对我们来说，有几个库可以为我们完成所有这些工作（甚至更多）。最常用的库是Square的**Picasso**（见[https://square.github.io/picasso/](https://square.github.io/picasso/))和Bump Technologies的**Glide**。Facebook的**Fresco**（见[https://frescolib.org/](https://frescolib.org/))相对不太受欢迎。最近获得关注的一个库是**Coil**（见[https://coil-kt.github.io/coil/](https://coil-kt.github.io/coil/))。

我们将继续使用Glide，因为它始终是两种流行的图像加载选项中最快的，无论是从互联网还是从缓存中加载。然而，值得注意的是，Picasso更轻量级，因此这是一个权衡，这两个库都非常有用。

要将Glide添加到您的项目中，将其添加到应用程序的`build.gradle`文件中的`dependencies`块：

[PRE27]

事实上，因为我们可能在以后改变主意，这是一个很好的机会将具体的库抽象出来，以拥有我们自己的更简单的接口。所以，让我们首先定义我们的`ImageLoader`接口：

[PRE28]

这是一个简单的实现。在生产实现中，您可能希望添加参数（或多个函数）以支持不同的裁剪策略或加载状态等选项。

我们对接口的实现将依赖于 Glide，所以看起来可能像这样：

[PRE29]

我们在类名前加上 `Glide` 以区分其他可能的实现。使用 `context` 构建出 `GlideImageLoader` 允许我们实现干净的 `loadImage(String, ImageView)` 接口，而无需担心上下文，这是 Glide 加载图像所必需的。

事实上，Glide 对 Android 上下文非常智能。这意味着我们可以为 `Activity` 和 `Fragment` 范围分别实现不同的实现，并且 Glide 会知道当图像加载请求超出相关范围时。

由于我们还没有将 `ImageView` 添加到布局中，让我们现在就做：

[PRE30]

这将在我们的 `TextView` 下方添加一个 ID 为 `main_profile_image` 的 `ImageView`。

我们现在可以在 `MainActivity` 中创建 `GlideImageLoader` 的实例：

[PRE31]

在生产应用中，你会注入依赖项，而不是直接创建。

接下来，我们告诉我们的 Glide 加载器加载图像，并在加载完成后将其中心裁剪到提供的 `ImageView` 中。这意味着图像将被缩放或缩小以完全填充 `ImageView`，任何多余的内容将被裁剪掉。由于我们之前已经获取了一个图像 URL，我们只需要进行调用：

[PRE32]

我们必须确保结果包含一个非空或由空格组成的字符串（前一个代码块中的 `isBlank()`）。然后，我们可以安全地将 URL 加载到 `ImageView` 中。完成。如果我们现在运行我们的应用，我们应该会看到以下截图：

![图 5.4 – 服务器响应图像 URL 与实际图像](img/B19411_05_04.jpg)

图 5.4 – 服务器响应图像 URL 与实际图像

记住 API 返回的是随机结果，所以实际图像可能不同。如果我们很幸运，甚至可能会得到一个动画 GIF，我们将会看到它动画播放。

## 练习 5.03 – 从获取的 URL 加载图像

在之前的练习中，我们从 API 响应中提取了图像 URL。现在，我们将使用该 URL 从网络获取图像并在我们的应用中显示：

1.  打开应用的 `build.gradle` 文件并添加 Glide 依赖：

    [PRE33]

将项目与 Gradle 文件同步。

1.  在左侧 `com.example.catagentprofile`) 上右键单击并选择 **新建** | **Kotlin 文件/类**。

1.  在 **名称** 字段中填写 `ImageLoader`。对于 **类型**，选择 **接口**。

1.  打开新创建的 `ImageLoader.kt` 文件并更新如下：

    [PRE34]

这将是应用中任何图像加载器的接口。

1.  再次右键单击项目包名并选择 **新建** | **Kotlin 文件/类**。

1.  将新文件命名为 `GlideImageLoader` 并选择 **类** 作为 **类型**。

1.  更新新创建的文件：

    [PRE35]

1.  打开 `activity_main.xml` 并更新如下：

    [PRE36]

这将在 `TextView` 下方添加一个名为 `main_profile_image` 的 `ImageView`。

1.  打开 `MainActivity.kt` 文件。

1.  在类的顶部添加一个字段用于你新添加的 `ImageView`：

    [PRE37]

1.  在 `onCreate(Bundle?)` 函数上方定义 `ImageLoader`：

    [PRE38]

1.  更新您的 `getCatImageResponse()` 函数如下：

    [PRE39]

1.  现在，一旦您有一个非空 URL，它将被加载到 `profileImageView`。

1.  运行应用程序：

![图 5.5 – 展示随机图像及其源 URL 的练习结果](img/B19411_05_05.jpg)

图 5.5 – 展示随机图像及其源 URL 的练习结果

以下是一些附加步骤。

1.  更新您的布局如下：

    [PRE40]

这将添加一个 `Agent breed` 标签并整理视图布局。现在，您的布局看起来更像是一个正规的猫代理配置文件应用程序。

1.  在 `MainActivity.kt` 中定位以下行：

    [PRE41]

将前面的行替换为以下内容以查找新的名称字段：

[PRE42]

1.  更新 `getCatImageResponse()` 如下：

    [PRE43]

这样做是为了将 API 返回的第一个品种加载到 `agentNameView` 中，如果失败则显示 `Unknown`。

1.  在撰写本文时，The Cat API 中没有多少图片包含品种数据。然而，如果您运行应用程序足够多次，您最终会看到如下内容：

![图 5.6 – 展示猫代理图像和品种](img/B19411_05_06.jpg)

图 5.6 – 展示猫代理图像和品种

在本章中，我们学习了如何从远程 API 获取数据。然后我们学习了如何处理这些数据并从中提取所需的信息。最后，我们学习了如何在给定图像 URL 的情况下在屏幕上显示图像。

在以下活动中，我们将应用我们的知识来开发一个应用程序，该应用程序会告诉用户纽约当前的天气，并向用户提供相关的天气图标。

## 活动第 5.01 部分 – 显示当前天气

假设我们想要构建一个显示纽约当前天气的应用程序。此外，我们还想显示代表当前天气的图标。

本活动旨在创建一个应用程序，该应用程序轮询 API 端点以获取当前天气的 JSON 格式，将数据转换为本地模型，并使用该模型来显示当前天气。它还提取表示当前天气的图标 URL 并将其图标加载到屏幕上显示。

在本活动中，我们将使用免费的 OpenWeatherMap API。文档可以在 [https://openweathermap.org/api](https://openweathermap.org/api) 找到。要注册 API 令牌，请访问 [https://home.openweathermap.org/users/sign_up](https://home.openweathermap.org/users/sign_up)。您可以在 [https://home.openweathermap.org/api_keys](https://home.openweathermap.org/api_keys) 找到您的密钥并按需生成新的密钥。

本活动的步骤如下：

1.  创建一个新的应用程序。

1.  授予应用程序互联网权限，以便能够进行 API 和图像请求。

1.  将 Retrofit、Moshi 转换器和 Glide 添加到应用程序中。

1.  更新应用程序布局以支持以文本形式（简短和详细描述）以及天气图标图像的形式展示天气。

1.  定义模型。创建包含服务器响应的类。

1.  添加 Retrofit 服务以使用 OpenWeatherMap API，[https://api.openweathermap.org/data/2.5/weather](https://api.openweathermap.org/data/2.5/weather)。

1.  使用 Moshi 转换器创建 Retrofit 实例。

1.  调用 API 服务。

1.  处理成功的服务器响应。

1.  处理不同的失败场景。

预期的输出如下所示：

![图 5.7 – 最终的天气应用](img/B19411_05_07.jpg)

图 5.7 – 最终的天气应用

注意

该活动的解决方案可以在 [https://packt.link/By7eE](https://packt.link/By7eE) 找到。

# 摘要

在本章中，我们学习了如何使用 Retrofit 从 API 获取数据。然后我们学习了如何使用 Moshi 处理 JSON 响应以及纯文本响应。我们还看到了如何处理不同的错误场景。

我们后来学习了如何使用 Glide 从 URL 加载图片，以及如何通过 `ImageView` 将它们展示给用户。

有很多流行的库用于从 API 获取数据以及加载图片。我们只介绍了一些最受欢迎的库。你可能想尝试其他一些库，以找出最适合你需求的库。

在下一章中，我们将介绍 `RecyclerView`，这是一个强大的 UI 组件，我们可以用它向用户展示项目列表。
