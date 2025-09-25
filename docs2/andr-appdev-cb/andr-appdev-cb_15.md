# 第十五章. 后端即服务选项

在本章中，我们将涵盖以下主题：

+   App42

+   Backendless

+   Buddy

+   Firebase

+   Kinvey

# 简介

随着您的应用程序和用户基础的扩大，您可能会希望将您的应用程序连接到跨设备甚至跨用户，例如高分排行榜。您有两个选择：

+   创建和维护自己的服务器

+   使用**后端即服务**（**BaaS**）提供商

作为移动开发者，创建和维护一个网络服务器是一个耗时的工作，可能会使您偏离开发工作。

### 备注

如果您对 BaaS 提供商不熟悉，以下是一些背景信息：

维基百科 – 移动后端即服务：

[`zh.wikipedia.org/wiki/移动后端即服务`](https://zh.wikipedia.org/wiki/移动后端即服务)

我们将探讨几个针对 Android 开发者特定功能的 BaaS 提供商。仅包括提供原生 Android 支持和免费订阅的提供商。（仅提供免费试用或仅付费计划的提供商不包括在内。）随着您的应用程序超出免费层，所有这些提供商都提供不同月费的高级服务。

以下表格提供了每个提供商提供的每月免费服务的快速比较：

| 提供商 | 每月用户数 | API 调用 | 推送通知 | 文件存储 |
| --- | --- | --- | --- | --- |
| Firebase | 无限制 | 100 SC | N/A | 1 GB |
| Buddy | * | 20/sec | 5 Million | 10 GB |
| App42 | * | 1 Million / month | 1 Million | 1 GB |
| Kinvey | 1000 | * | * | 30 GB |
| Backendless | 100 | 50/sec | 1 Million | 20 GB |

> * = 未在其网站上发布
> 
> N/A = 功能不可用
> 
> SC = 同时连接数

### 备注

**免责声明**：上述表格和随后的食谱信息来源于它们的公共网站，并受其自行决定而变更。正如您所知，移动行业一直在不断变化；预计价格和服务将发生变化。请仅将此信息作为起点使用。

最后，这并不是 BaaS 提供商的详尽列表。希望本章能为您介绍 BaaS 能做什么以及如何为您的应用程序使用 BaaS 提供良好的入门。接下来的食谱将查看每个提供商，并指导您将它们的库添加到项目中。这将为您提供服务的直接比较。您将看到，一些服务比其他服务更容易使用，这可能是决定性因素。

# App42

App42 是 ShepHertz 的 BaaS API 产品，ShepHertz 是一家提供多种服务的云服务提供商，包括游戏平台、平台即服务和营销分析。它们拥有非常丰富的功能集，包括许多特别适用于游戏的服务。

App42 Android SDK 支持以下功能：

+   用户服务

+   存储服务

+   定制代码服务

+   推送通知服务

+   事件服务

+   礼品管理服务

+   定时服务

+   社交服务

+   A/B 测试服务

+   Buddy 服务

+   头像服务

+   成就服务

+   排行榜服务

+   奖励服务

+   上传服务

+   相册服务

+   地理服务

+   会话服务

+   评论服务

+   购物车服务

+   目录服务

+   消息服务

+   推荐服务

+   邮件服务

+   日志服务

### 注意

要注册 App42/ShepHertz，请访问以下链接：

[`apphq.shephertz.com/register`](https://apphq.shephertz.com/register)

这是 App4 注册界面的截图：

![App42](img/B05057_15_1.jpg)

## 准备工作

在 Android Studio 中创建一个新的项目，并将其命名为`App42`。使用默认的**手机和平板**选项，并在提示**活动类型**时选择**空活动**。

从以下链接下载并解压 App42 SDK：[`example.com`](https://example.com)

[`github.com/shephertz/App42_ANDROID_SDK/archive/master.zip`](https://github.com/shephertz/App42_ANDROID_SDK/archive/master.zip)

在创建您的 App42 账户（见前述链接）后，登录到 AppHQ 管理控制台，并注册您的应用程序。您将需要 ApiKey 和 SecretKey。

## 如何操作...

要将 App42 支持添加到您的项目中，首先打开 AndroidManifest，并按照以下步骤操作：

1.  添加以下权限：

    ```java
    <uses-permission android:name="android.permission.INTERNET"/>
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
    ```

1.  在您的文件浏览器中打开以下文件夹：`<项目文件夹>\App42\app\libs`（如果`libs`文件夹不存在，请创建它），并将`App42_ANDROID-CAMPAIGN_x.x.jar`文件复制到`app\libs`文件夹中。

1.  打开应用程序模块的 Gradle 构建文件：`build.gradle (Module: app)`，并在`dependencies`部分添加以下内容：

    ```java
    compile files('libs/App42_ANDROID-CAMPAIGN_x.x.jar')
    ```

1.  打开`ActivityMain.java`并添加以下导入：

    ```java
    import com.shephertz.app42.paas.sdk.android.App42API;
    ```

1.  将以下代码添加到`onCreate()`回调中：

    ```java
    App42API.initialize(this, "YOUR_API_KEY", "YOUR_SECRET_KEY");
    ```

1.  您现在可以在设备或模拟器上运行应用程序了。

## 它是如何工作的...

很遗憾，App42 不支持 Gradle 构建格式，因此您需要手动下载 JAR 文件并将其复制到`\libs`文件夹中。

在第 3 步中，将`App42_ANDROID-CAMPAIGN_x.x.jar`中的`x.x`替换为您下载的文件中的当前版本号。

在第 5 步中，将`YOUR_API_KEY`和`YOUR_SECRET_KEY`替换为您在 App42 注册应用程序时收到的凭据。

## 更多内容...

以下是一个使用 App42 API 注册用户的示例：

```java
UserService userService = App42API.buildUserService();
userService.createUser("userName", "password", "email", new App42CallBack() {
    public void onSuccess(Object response) {
        User user = (User)response;
        Log.i("UserService","userName is " + user.getUserName());
        Log.i("UserService", "emailId is " + user.getEmail());
    }
    public void onException(Exception ex) {
        System.out.println("Exception Message"+ex.getMessage());
    }
});
```

## 参见

+   如需更多信息，请参阅 App42 网页[`api.shephertz.com/`](http://api.shephertz.com/)

# Backendless

除了**MBaaS**（**移动后端作为服务**，正如他们所称呼的），Backendless 还提供其他一些服务，如托管、API 服务和市场。他们的 MBaaS 功能包括：

+   用户管理

+   数据持久化

+   地理定位

+   媒体流

+   发布/订阅消息

+   推送通知

+   自定义业务逻辑

+   分析

+   移动代码生成

### 注意

要注册 Backendless，请遵循此链接：

[`develop.backendless.com/#registration`](https://develop.backendless.com/#registration)

这是 Backendless 注册窗口的截图：

![Backendless](img/B05057_15_2.jpg)

## 准备工作

在 Android Studio 中创建一个新的项目，命名为 `Backendless`。使用默认的 **手机和平板** 选项，并在提示 **活动类型** 时选择 **空活动**。

您需要一个 **Backendless** 账户（见前述链接），并且必须通过他们的 **Backendless** 控制台注册您的应用程序。一旦您获得了 App ID 和 Secret Key，开始以下步骤。

## 如何操作...

要将 `Backendless` 添加到您的项目，打开 AndroidManifest.xml 并按照以下步骤操作：

1.  添加以下权限：

    ```java
    <uses-permission android:name="android.permission.INTERNET"/>
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
    ```

1.  打开应用模块的 Gradle 构建文件：`build.gradle (Module: app)` 并在 `dependencies` 部分添加以下内容：

    ```java
    compile 'com.backendless:android:3.0.3'
    ```

1.  打开 `ActivityMain.java` 并添加以下导入：

    ```java
    import com.backendless.Backendless;
    ```

1.  将以下代码添加到 `onCreate()` 回调中：

    ```java
    String appVersion = "v1";
    Backendless.initApp(this, YOUR_APP_ID, YOUR_SECRET_KEY, appVersion);
    ```

1.  您现在可以开始在设备或模拟器上运行应用程序了。

## 工作原理...

在第 4 步中将 `YOUR_APP_ID` 和 `YOUR_SECRET_KEY` 替换为您从 **Backendless** 控制台收到的凭证。

如果您更喜欢直接下载 SDK 而不是使用 Maven 依赖项，它在这里可用：[`backendless.com/sdk/java/3.0.0/backendless-sdk-android.zip`](https://backendless.com/sdk/java/3.0.0/backendless-sdk-android.zip)。

## 更多内容...

以下是一个使用 `BackendlessUser` 对象注册用户的示例：

```java
BackendlessUser user = new BackendlessUser();
user.setEmail("<user@email>");
user.setPassword("<password>");
Backendless.UserService.register(user, new BackendlessCallback<BackendlessUser>() {
    @Override
    public void handleResponse(BackendlessUser backendlessUser) {
        Log.d("Registration", backendlessUser.getEmail() + " successfully registered");
    }
} );
```

## 参见

+   更多信息，请参阅 Backendless 网页 [`backendless.com/`](https://backendless.com/)

# Buddy

与列表中的其他 BaaS 提供商相比，Buddy 有所不同，因为它们主要专注于连接设备和传感器。为了帮助维护隐私法规，Buddy 允许您选择将数据托管在美国或欧盟。

Buddy 支持常见的场景，如：

+   记录指标事件

+   发送推送通知

+   接收和安全存储遥测数据

+   存储和管理二进制文件

+   深度移动分析，了解客户如何使用应用程序

+   将设备或应用程序数据与您的公司 BI 系统集成

+   在您选择的地理位置的沙盒中，存储私有数据。

如果您想审查或贡献 Buddy SDK，源代码可以通过以下 Git 命令获取：

```java
git clone https://github.com/BuddyPlatform/Buddy-Android-SDK.git
```

### 注意

要注册 Buddy，请点击此链接：

[`www.buddyplatform.com/Signup`](https://www.buddyplatform.com/Signup)

这是 Buddy 注册的截图：

![Buddy](img/B05057_15_3.jpg)

## 准备工作

在 Android Studio 中创建一个新的项目，命名为 `Buddy`。使用默认的 **手机和平板** 选项，并在提示 **活动类型** 时选择 **空活动**。

您需要一个 Buddy 账户（见前述链接），并且必须通过他们的仪表板注册您的应用程序。一旦您获得了 App ID 和 App Key，开始以下步骤。

## 如何操作...

要将 Buddy 添加到您的项目，打开 AndroidManifest.xml 并按照以下步骤操作：

1.  添加以下权限：

    ```java
    <uses-permission android:name="android.permission.INTERNET"/>
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
    ```

1.  打开应用模块的 Gradle 构建文件：`build.gradle (Module: app)` 并在 `dependencies` 部分添加以下内容：

    ```java
    compile 'com.buddy:androidsdk:+'
    ```

1.  打开 `ActivityMain.java` 并添加以下导入：

    ```java
    import com.buddy.sdk.Buddy;
    ```

1.  在 `onCreate()` 回调中添加以下代码：

    ```java
    Buddy.init(myContext, "appId", "appKey");
    ```

1.  您已准备好在设备或模拟器上运行应用程序。

## 工作原理...

在第 4 步中，将 `appId` 和 `appKey` 替换为从 Buddy 控制台收到的凭证。

与大多数其他 BaaS 提供商类似，我们只需在我们的 Gradle 构建中添加对 Maven 仓库的引用。然后，我们添加导入并开始调用 Buddy API。

## 更多内容...

以下是一个使用 Buddy 注册用户的示例：

```java
Buddy.createUser("someUser", "somePassword", null, null, null, null, null, null, new BuddyCallback<User>(User.class) {
    @Override
    public void completed(BuddyResult<User> result) {
        if (result.getIsSuccess()) {
            Log.w(APP_LOG, "User created: " + result.getResult().userName);
        }
    }
});
```

## 参见

+   更多信息，请参考 Buddy 网页：[`buddy.com/`](https://buddy.com/)

# Firebase

Firebase 是一个主要专注于数据库功能的 BaaS 提供商。虽然它们不像其他大多数 BaaS 提供商那样功能全面，但它们在数据库方面做得很好。它们是本列表中唯一提供自动同步数据库功能的提供商。

Firebase 服务包括：

+   Firebase 实时数据库

+   Firebase 身份验证

+   Firebase 主机

+   用户身份验证—电子邮件和密码、Facebook、Twitter、GitHub 和 Google

由于它们最近被 Google 收购，您可以期待与 Google Cloud 解决方案的进一步集成，正如您可以从此链接中看到：

[`cloud.google.com/solutions/mobile/firebase-app-engine-android-studio`](https://cloud.google.com/solutions/mobile/firebase-app-engine-android-studio)

### 注意

要使用 Firebase 注册，请访问此链接：

[`www.firebase.com/login/`](https://www.firebase.com/login/)

这是 Firebase 注册窗口的截图：

![Firebase](img/B05057_15_4.jpg)

## 准备就绪

在 Android Studio 中创建一个新的项目，并将其命名为 `Firebase`。使用默认的 **Phone & Tablet** 选项，并在提示 **Activity Type** 时选择 **Empty Activity**。

您将需要当您在 Firebase 上注册应用程序时提供的 Firebase URL。

## 如何操作...

要将 Firebase 添加到您的项目中，首先打开 AndroidManifest.xml 文件，并按照以下步骤操作：

1.  添加以下权限：

    ```java
    <uses-permission android:name="android.permission.INTERNET"/>
    ```

1.  打开应用程序模块的 Gradle 构建文件：`build.gradle (Module: app)` 并在 `dependencies` 部分添加以下内容：

    ```java
    compile 'com.firebase:firebase-client-android:2.5.0+'
    ```

1.  打开 `ActivityMain.java` 并添加以下导入：

    ```java
    import com.firebase.client.Firebase;
    ```

1.  在 `onCreate()` 回调中添加以下代码：

    ```java
    Firebase.setAndroidContext(this);
    Firebase firebase = new Firebase("https://<YOUR-FIREBASE-APP>.firebaseio.com/");
    ```

1.  您已准备好在设备或模拟器上运行应用程序。

## 工作原理...

将 Firebase 支持添加到您的应用程序相当直接。将 `<YOUR-FIREBASE-APP>` 占位符替换为 Firebase 在您注册应用程序时提供的链接。

## 更多内容...

以下是一个使用 Firebase 注册用户的示例：

```java
firebase.createUser("bobtony@firebase.com", "correcthorsebatterystaple", new Firebase.ValueResultHandler<Map<String, Object>>() {
    @Override
    public void onSuccess(Map<String, Object> result) {
        Log.i("Firebase", "Successfully created user account with uid: " + result.get("uid"));
    }
    @Override
    public void onError(FirebaseError firebaseError) {
        // there was an error
    }
});
```

## 参见

+   更多信息，请参考 Firebase 网页 [`www.firebase.com/`](https://www.firebase.com/)

# Kinvey

Kinvey 是最早开始提供移动后端服务的提供商之一。它们的功能包括：

+   用户管理

+   数据存储

+   文件存储

+   推送通知

+   社交网络集成

+   位置服务

+   生命周期管理

+   版本控制

    ### 注意

    在 [`console.kinvey.com/sign-up`](https://console.kinvey.com/sign-up) 上注册 Kinvey。

这是 Kinvey 注册窗口的截图：

![Kinvey](img/B05057_15_5.jpg)

## 准备就绪

在 Android Studio 中创建一个新的项目，并将其命名为 `Kinvey`。使用默认的 **手机和平板** 选项，并在提示 **活动类型** 时选择 **空活动**。

从以下链接下载并解压 Kinvey SDK：[download.kinvey.com/Android/kinvey-android-2.10.5.zip](http://download.kinvey.com/Android/kinvey-android-2.10.5.zip)

您需要一个 Kinvey 账户（见前面的链接），并且必须通过他们的开发者控制台注册您的应用程序。一旦您获得了 App Key 和 App Secret，开始以下步骤。

## 如何做到这一点...

要将 Kinvey 添加到您的项目中，请按照以下步骤操作：

1.  将以下权限添加到 AndroidManifest.xml 文件中：

    ```java
    <uses-permission android:name="android.permission.INTERNET"/>
    ```

1.  在您的文件浏览器中打开以下文件夹：`<项目文件夹>\Kinvey\app\libs`（如果不存在 `libs` 文件夹，请创建它）并将 SDK 的 `lib` 和 `libJar` 文件夹中的所有文件复制到 `app\libs` 文件夹中。

1.  打开应用模块的 Gradle 构建文件：`build.gradle (Module: app)` 并添加以下 `repositories` 和 `dependencies`（保留任何现有的条目）：

    ```java
    repositories {
        flatDir {
            dirs 'libs'
        }
    }

    dependencies {
        compile fileTree(dir: 'libs', include: ['*.jar'])
        compile(name:'kinvey-android-*', ext:'aar')
    }
    ```

1.  打开 `MainActivity.java` 并添加以下导入：

    ```java
    import com.kinvey.android.Client;
    ```

1.  将以下内容添加到类声明中：

    ```java
    final Client mKinveyClient = new mKinveyClient("your_app_key", "your_app_secret", this.getApplicationContext()).build();
    ```

1.  您现在可以开始在设备或模拟器上运行应用程序了。

## 它是如何工作的...

由于 Kinvey 不提供简单的 Gradle 依赖项，因此它不是最容易设置的 BaaS 之一。相反，您需要像第 2 步中那样直接将它们的库添加到项目库中。

这些步骤将设置好 Kinvey 客户端，使其准备好开始添加您应用程序的附加功能。只需确保将 Kinvey 客户端构建器中的占位符替换为您的应用程序凭据。

## 还有更多...

为了验证您的设置是否正确工作，请在 `onCreate()` 方法或按钮点击时调用以下代码：

```java
mKinveyClient.ping(new KinveyPingCallback() {
    public void onFailure(Throwable t) {
        Log.d("KinveyPingCallback", "Kinvey Ping Failed", t);
    }

    public void onSuccess(Boolean b) {
        Log.d("KinveyPingCallback", "Kinvey Ping Success");
    }
});
```

## 参见

+   更多信息，请参阅 Kinvey 网页 [`www.kinvey.com/`](http://www.kinvey.com/)
