# 第六章. 谷歌播放服务

既然你已经熟悉了在布局中使用组件，那么你应该开始考虑额外的功能。谷歌播放服务提供了使用谷歌地图、谷歌+等功能来吸引用户的功能。你如何轻松将这些功能添加到你的应用程序中？有哪些功能可用？使用谷歌播放服务需要满足哪些安卓版本要求？

本章重点介绍如何使用 Android Studio 创建、集成和使用谷歌播放服务。我们将了解哪些谷歌服务可用。我们还将学习标准的授权 API，以安全地授予和接收访问谷歌播放服务的令牌。我们还将了解这些服务的限制及其使用的好处。

以下是本章将要讨论的主题：

+   现有的谷歌服务

+   从 IDE 中添加谷歌播放服务

+   在你的应用中集成谷歌播放服务

+   理解自动更新

+   在你的应用中使用谷歌服务

# 谷歌播放服务的工作原理

当谷歌在 2012 年的谷歌 I/O 大会上预览谷歌播放服务时，它表示这个平台（[`developers.google.com/events/io/2012/`](https://developers.google.com/events/io/2012/)）...

> ...由在设备上运行的服务组件和一个与你应用一起打包的轻量级客户端库组成。

这意味着谷歌播放服务之所以能够工作，得益于两个主要组件：谷歌播放服务客户端库和谷歌播放服务 APK。

+   **客户端库**：谷歌播放服务客户端库包含了你的应用所使用的每个谷歌服务的接口。当你打包应用时，会包含这个库，它允许你的用户使用他们的凭据授权应用访问这些服务。客户端库会定期由谷歌进行升级，增加新的功能和服务。你可以在应用更新时升级这个库，当然，如果你不打算使用任何新功能，这并非必须。

+   **谷歌播放服务 APK**：谷歌播放服务**安卓软件包**（**APK**）在安卓操作系统中作为后台服务运行。使用客户端库，你的应用程序可以访问这项服务，它负责在运行时执行操作。并不保证所有设备上都安装有此 APK。如果设备未预装此 APK，可以在谷歌播放商店中获取。

这样，谷歌将其服务的运行时与作为开发者的你所做的实现分离开来，因此你无需在谷歌播放服务每次升级时都更新你的应用程序。

尽管谷歌播放服务并未包含在安卓平台本身，但它们得到了大多数基于安卓的设备的支持。任何运行安卓 2.2 或更新版本的安卓设备都可以安装使用谷歌播放服务的应用程序。

# 可用的服务

Google Play 服务被认为是轻松为各种设备上的用户添加更多特性的方式，同时使用由 Google 提供支持的知名功能。使用这些服务，你可以添加新的收入来源，管理应用程序的分布，访问统计信息并了解应用程序用户的习惯，以及通过易于实现的 Google 功能（如地图或 Google 的社交网络 Google+）来改进你的应用程序。以下是对这些服务的说明：

+   **游戏**：使用 Google Play 游戏服务，你可以通过更社交的体验来改进你的游戏。

+   **位置**：整合位置 API，你可以使你的应用程序具有位置感知功能。

+   **Google 地图**：Google 地图 API 允许你在应用程序中使用 Google 提供的地图，并对其进行自定义。

+   **Google+**：通过使用 Android 的 Google+平台，你可以验证你的应用程序的用户。一旦验证成功，你还可以访问他们的公开资料和社交网络图。

+   **应用内购买**：使用 Google Play 应用内购买，你可以从你的应用程序中出售数字内容。你可以使用这项服务来销售一次性购买或对高级服务和特性的时间订阅。

+   **云消息传递**：**Google 云消息传递**（**GCM**）允许你在运行在基于 Android 的设备和你的服务器之间的应用程序中交换数据。

+   **全景图**：它使用户能够看到 360 度的全景图片。

# 向 Android Studio 添加 Google Play 服务

我们需要知道的第一件事是什么需要添加到我们的 Android Studio 中。我们刚刚了解到 APK 在 Google Play 商店中可用，它是服务的实际运行时。作为开发者的我们，在调试应用程序时只需要这个包在测试设备上可用。我们需要添加到 Android Studio 的是 Google Play 服务客户端库。

这个库通过 Android SDK 管理器（软件开发工具包管理器）进行分发，具体将在第七章《工具》中进行详细介绍。要打开它，请导航到**工具** | **Android** | **SDK 管理器**。在**Extras**文件夹下的软件包列表中我们可以找到 Google Play 服务。勾选**Google Play 服务**复选框，然后点击**安装 1 个软件包...**按钮。

![向 Android Studio 添加 Google Play 服务](img/5273OS_06_01.jpg)

执行这些操作将把库项目添加到我们 SDK 安装文件夹的位置，`/sdk/extras/google/google_play_services/`。你可以在 SDK 管理器中悬停在 Google Play 服务行上，查看工具提示来检查确切的路径。

![向 Android Studio 添加 Google Play 服务](img/5273OS_06_02.jpg)

导航到库文件夹以检查其内容。`samples`文件夹包含身份验证服务（`auth/`）、Google Maps v2 服务（`maps/`）、Google+服务（`plus/`）以及全景服务（`panorama/`）的示例项目。包含 Google Play Services 库项目的文件夹是`libproject/`。在这个项目文件夹中放置了`google-play-services.jar`文件，即`libproject/google-play-services_lib/libs/google-play-services.jar`。

只需将此 JAR 文件拖放到`libs/`文件夹中即可将其添加到项目中。完成此操作后，选择 JAR 文件并在其上按鼠标右键。选择**作为库添加**的选项。在**创建库**对话框中，选择项目库级别，选择您的应用程序模块，然后点击**确定**。

您现在可以在项目库的`libs/`文件夹下找到`google-play-services.jar`文件，现在您将能够从代码中引用 Google Play 服务。

最后，您需要将库添加到您的 Gradle 构建文件中。为此，只需编辑`MyApplication/build.gradle`文件，并在`dependencies`部分添加以下行：

```java
compile files('libs/google-play-services.jar')
```

# Google Maps Android API v2

Google Maps Android API 允许您的应用程序用户探索 Google 服务上可用的地图。新的地图版本 2 提供了更多功能，如 3D 地图、室内和卫星地图、基于矢量技术的有效缓存和绘制，以及地图上的动画过渡。

让我们导入示例项目以检查最重要的类。点击**文件** | **导入项目**。在你 SDK 安装文件夹中搜索示例项目，并选择项目根目录，即 `/google_play_services/samples/maps/`。在下一个对话框中，勾选**从现有源创建项目**的选项。在后续的对话框中继续点击**下一步**，最后点击**完成**按钮，在新的窗口中打开示例项目。现在我们在 Android Studio 的新窗口中加载了 Google Play Services 项目和地图示例项目。

打开`BasicMapActivity`类，查看一个使用 Google Maps 的简单示例。你可以在`src/`文件夹中的 maps 项目里找到这个活动。`com.google.android.gms.maps`包包含了 Google Maps Android API 的类。

此活动声明了一个名为`mMap`的私有`GoogleMap`对象。**GoogleMap 类**是 API 的主要类，它是与地图相关的所有方法的入口点。您可以更改地图的主题颜色和图标以匹配您应用程序的风格。您还可以通过向地图添加标记来自定义地图。要添加一个简单的标记，你可以使用`GoogleMap`类的`addMarker`方法。在`BasicMapActivity`类中检查`setUpMap`方法，查看以下代码示例：

```java
mMap.addMarker(new MarkerOptions().position(new LatLng(0, 0)).title("Marker"));
```

`addMarker`方法有一个`MarkerOptions`对象作为参数。使用`position`方法我们指定地图上标记的坐标，使用`title`方法我们可以添加一个自定义字符串在标记上显示。

要将地图添加到布局中，我们可以使用`MapView`类，它扩展了`View`类并显示了一个地图。但在应用程序中放置地图的最简单方法是使用`MapFragment`对象。片段表示可以嵌入活动中的用户界面或行为的一部分。片段是一个可重用的模块。

**MapFragment 类**包装了一个地图视图，以自动处理组件的生命周期需求。它扩展了`Fragment`类，因此可以通过添加以下 XML 代码将其实例添加到布局中：

```java
<fragment
class="com.google.android.gms.maps.MapFragment"
android:layout_width="match_parent"
android:layout_height="match_parent" />
```

要查看前面代码的示例，请打开与`BasicMapActivity`类关联的布局；这是`/res/layout/`文件夹中的`basic_demo.xml`文件。

最后，我们需要从片段中获取`GoogleMap`对象的代码。我们可以使用`findFragmentById`方法找到地图`Fragment`，然后使用`getMap`方法从`Fragment`中获取地图。

```java
mMap = ((MapFragment) getFragmentManager().findFragmentById(R.Id.map).getMap();
```

在`BasicMapActivity`类中，此代码的示例位于`setUpMapIfNeeded`方法中。

最后一个重要的类是`GoogleMapOptions`类，它定义了地图的配置。您还可以通过编辑布局 XML 代码来修改地图的初始状态。以下是一些可用的有趣选项：

+   `mapType`：指定地图的类型。其值可以是`none`、`normal`、`hybrid`、`satellite`和`terrain`。

+   `uiCompass`：定义罗盘控制是否启用或禁用。

+   `uiZoomControls`：定义缩放控制是否启用或禁用。

+   `cameraTargetLat`和`cameraTargetLong`：指定初始相机位置。

# Android 上的 Google+平台

使用 Android 上的 Google+平台可以让开发者用用户在 Google+上使用的相同凭据对用户进行身份验证。您还可以使用公开的个人资料和社会关系图来欢迎用户，显示他们的名字、照片，或者与朋友建立联系。

包`com.google.android.gms.plus`包含了 Android 上的 Google+平台类。导入 Google+示例项目以了解最重要的类。Google+示例项目可以在 Google Play Services 安装文件夹中找到，位于`/google_play_services/samples/plus/`。

+   `PlusClient`和`PlusClient.Builder`：`PlusClient`是 API 的主要类。它是 Google+集成的入口点。`PlusClient.Builder`是一个构建器，用于配置`PlusClient`对象以正确与 Google+ API 通信。

+   `PlusOneButton`：实现 Google+上的+1 按钮以推荐 URL 的类。使用以下代码将其添加到布局中：

    ```java
    <com.google.android.gms.plus.PlusOneButton
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    plus:size="standard" />
    ```

    可用的尺寸有小型、中型、大型或标准。

    关于此功能的示例代码可以在示例项目中找到，在`src/`文件夹中的`PlusOneActivity`类及其关联的布局`res/layout/plus_one_activity.xml`中。

+   `PlusShare`：在 Google+上共享的帖子中包含资源。关于共享资源的示例代码可以在`src/`文件夹中的`ShareActivity`类及其关联的布局`res/layout/share_activity.xml`中找到。

首先，应在活动类的`onCreate`方法中实例化一个`PlusClient`对象，以调用其异步方法`connect`，这将连接客户端到 Google+服务。当应用完成使用`PlusClient`实例时，它应该调用`disconnect`方法，该方法将终止连接，并且也应该始终从活动的`onStop`方法中调用。

# Google Play 应用内购买 v3

应用内购买 v3 允许您从应用中出售虚拟内容。这种虚拟内容可以通过一次性计费支付一次，也可以通过订阅或费用进行计时特许。使用这项服务，您可以允许用户为额外功能付费并访问高级内容。

任何在 Google Play 商店发布的应用都可以实现应用内购买 API，因为它只需要与发布应用相同的账户：一个 Google Play 开发者控制台账户和一个 Google Wallet 商家账户。

使用 Google Play 开发者控制台，您可以定义您的产品，包括类型、识别代码（SKU）、价格、描述等。定义好产品后，您可以从这个应用程序访问这些内容。当用户想要购买这些内容时，以下购买流程将在您的应用内购买应用和 Google Play 应用之间发生：

1.  您的应用调用`isBillingSupported()`来检查 Google Play 是否支持您正在使用的应用内购买版本。

1.  如果支持应用内购买 API 版本，您可以使用`getPurchases()`来获取已购买物品的 SKU 列表。这个列表将返回在一个`Bundle`对象中。

1.  您可能想要通知用户可用的应用内购买选项。为此，您的应用可以发送一个`getSkuDetails()`请求，这将导致一个列表生成，其中包含产品的价格、标题、描述以及更多关于该物品的信息。

# Google 云消息传递

Android 的 GCM 允许通过使用异步消息在您的服务器和应用程序之间进行通信。您无需担心处理这种通信的低级别方面，如排队和消息构造。使用这项服务，您可以轻松地为您的应用实现一个通知系统。

使用 GCM 时，您有两个选项：

+   服务器可以通知您的应用有新的数据可供从服务器获取，然后应用程序获取这些数据。

+   服务器可以直接在消息中发送数据。消息负载可以达到 4 KB。这使得您的应用程序可以一次性访问数据并相应地采取行动。

为了发送或接收消息，您需要获取一个注册 ID。这个注册 ID 标识了设备和应用程序的组合。为了让您的应用程序使用 GCM 服务，您需要将以下行添加到项目的清单文件中：

```java
<uses-permission android:name="com.google.android.c2dm.permission.RECEIVE"/>
```

您需要使用的主要类是`GoogleCloudMessaging`。这个类在`com.google.android.gms.gcm`包中可用。

# 总结

到本章结束时，我们将了解可用的 Google Play 服务。我们学习了如何通过其客户端库和 Android 包来改善我们的应用程序。读者应该已经通过 SDK 管理器在 Android Studio 中成功安装了 Google Play 服务客户端库，并且应该能够使用库功能构建应用程序。我们还学习了一些关于 Google Maps v2、用于 Android 身份验证的 Google+平台、Google Play 应用内购买和 GCM 的技巧。

在下一章中，我们将了解 Android Studio 中提供的某些有用工具。我们将再次详细使用 SDK 管理器来安装不同的包。我们还将学习关于 AVD 管理器，以便能够拥有不同的虚拟设备来测试我们的应用程序。我们将使用 Javadoc 工具为我们的项目生成 Javadoc 文档，并且我们将了解 Android Studio 中可用的版本控制系统。
