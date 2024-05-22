# 第十四章：使您的应用程序面向全球

在经过多个小时的工作和学习许多新知识后构建应用程序，开发人员的最终满足感是看到用户轻松下载并享受使用应用程序的体验，从中获得最大的使用价值。

在本章中，我们将学习通过 Google Play 商店和亚马逊应用商店分发我们的应用程序所涉及的各种步骤。我们还将了解数字签名我们的应用程序以验证其真实性。

在本章中，重点将是学习以下内容：

+   通过 Android Studio 和命令行生成密钥库

+   通过 Google Play 商店发布应用程序

+   通过亚马逊应用商店发布应用程序

# 密钥库生成

Android 的最重要的安全功能之一是允许安装 APK，但只能来自受信任的来源，如 Google Play 商店或亚马逊应用商店。这些分发渠道要求开发人员对应用程序进行身份验证，声明这确实是他或她打算分发的应用程序。

应用程序的所有者，即开发人员，将拥有私钥，并且分发渠道将使用相应的公钥对其进行签名。公钥和私钥的组合意味着数字签名存储在`keyStore`文件中。`keyStore`是一个二进制文件，其中存储了用于对应用程序进行签名的数字密钥。

在将应用程序发布到 Google Play 商店进行分发之前，必须对 APK 进行数字签名。数字签名用作开发人员的身份验证，并确保只能通过受信任的来源进行应用程序更新。

保持密钥库文件的安全并记住密钥密码非常重要。一旦使用密钥库文件对应用程序进行签名并发布，任何对应用程序的进一步更新只能使用相同的密钥进行。

`KeyStore`可以通过几种方式生成：

+   Android Studio

+   命令行

让我们详细讨论生成密钥库所涉及的步骤。

# 通过 Android Studio 生成密钥库

这些是我们需要遵循的通过 Android 生成密钥库的步骤：

1.  一旦打开我们希望为其生成 APK 的项目，点击“构建|生成已签名 APK**。**”这将导致“生成已签名 APK”屏幕显示。 Android Studio 期望用户选择密钥库路径：

![](img/c9ac4376-b811-4ff7-b38a-8e030ce63796.png)

1.  由于我们将生成一个新的密钥库，点击“创建新”按钮。这将显示如下的“新密钥库”窗口：

![](img/444a4f85-f43a-40e8-bf21-79fc39dd9bb1.png)

1.  选择密钥库路径并为`.jks`（Java 密钥库）文件提供名称：

![](img/94d5b9d4-a6fc-4004-ad46-b3ef943e9d9f.png)

1.  确认密钥库路径后，我们需要填写密钥库密码、密钥别名、密钥别名密码、名字和姓氏、组织单位、组织、城市或地点、州或省和国家代码（XX）：

![](img/25ecf02c-068d-417a-8988-b7b01c897be8.png)

1.  一旦填写了所需的详细信息并点击“确定”按钮，我们应该能够继续进行“生成已签名 APK”屏幕。点击下一步：

![](img/f0f56374-a0ec-4471-8c32-e69c610ef506.png)

1.  在下一个屏幕上，我们将有选择 APK 目标文件夹和构建类型的选项。然后，点击“完成”按钮：

![](img/f71ee512-86de-4821-adce-fd03db5b1d6f.png)

1.  完成后，控制台中显示已签名 APK 的生成确认以及定位或分析 APK 的选项：

![](img/20790051-e811-489b-bcd6-38fe45b1f15c.png)

1.  已签名 APK 经过数字签名，可以通过 Google Play 商店和其他发布平台发布，并且可以在目标文件夹中找到：

![](img/5be464c0-d783-4cc2-bb4c-444d79bc07c1.png)

1.  现在我们已经生成了密钥库，以后每当我们更新应用程序时，Android Studio 都会提供我们生成已签名 APK 的屏幕，并期望填写密码：

![](img/2919050f-4ea7-4de4-acf3-2c4c6efe53be.png)

按照新*密钥库* *生成*中描述的相同过程，用户应该能够生成已签名的 APK。

# 通过 Android Studio 自动签署 APK

我们有选项，可以在对应用程序进行更改时自动生成已签名的 APK。这可以通过在 Android Studio 中执行以下操作来实现：

1.  右键单击**App** | **项目结构**。

1.  选择签名标签。在此标签中，我们需要提供应用程序签名配置的详细信息。在这里，我们将其命名为`config`，并存储密钥别名、密码和存储文件的路径：

![](img/6705b337-6114-47cb-8b5a-ea447d13b43f.png)

添加签名`config`将导致签名详细信息以纯文本形式添加到`build.gradle`文件中：

```kt
android {
     signingConfigs {
         config {
             keyAlias 'packtkey' keyPassword 'vasantham' storeFile file('G:/newkey/dreamindia.jks')
             storePassword 'vasantham' } 

```

将此信息移出构建文件以确保敏感信息不易被他人访问是明智的。在项目的根目录中，我们应该创建一个名为`keystore.properties`的文件。该文件将包含以下信息：

```kt
storePassword = OurStorePassword
KeyPassword = ourKeyPassword
keyAlias = ourKeyAlias
storeFile = ourStoreFileLocation
```

由于我们已经将密钥库详细信息移动到单独的文件中，现在我们应该在`build.gradle`文件中添加以下代码，以确保签名配置可用于自动签署 APK。我们应该在`android{}`块之前加载`keystore.properties`文件。

在此代码中，我们创建了一个名为`keystorePropertiesFile`的变量，并将其初始化为我们创建的`keystore.properties`文件。此外，我们初始化了一个名为`keyStoreProperties`的新`Properties()`对象。`keystorePropertiesFile`的详细信息被加载到`keystoreProperties`对象中：

```kt
def keystorePropertiesFile = rootProject.file("keystore.properties")
def keystoreProperties = new Properties()
keystoreProperties.load(new FileInputStream(keystorePropertiesFile))

android {
.......
}
```

通过添加上述代码，我们可以使用`keystoreProperties['propertyName']`的语法引用存储在`keystoreProperties`中的所有属性***。***我们应该能够修改`build.gradle`文件中的签名配置，如下面的代码所示：

```kt
android {
     signingConfigs {
         config {
             keyAlias keystoreProperties['keyAlias'] keyPassword keystoreProperties['keyPassword'] storeFile file(keystoreProperties['storeFile'])
             storePassword keystoreProperties['storePassword'] } 
}
..........
}
```

构建文件现在是安全的，不包含任何敏感信息。如果我们要使用任何源代码控制或共享代码库，我们需要确保删除并保护`keystore.properties`文件。

# 构建类型和风味

开发人员可以通过`build.gradle`文件维护构建类型中的变体，并且可以通过配置来配置这些变体。该配置使开发人员能够在同一应用程序中维护调试代码和发布版本的代码：

+   **调试：**打开调试选项**并且**也可以使用调试密钥签署应用程序

+   **发布：**关闭调试选项，使用发布密钥签署应用程序，并且还会减少或删除最终 APK 中与调试相关的代码

我们可以在 Android Studio 中定义调试或发布类型的构建：

1.  右键单击 app | 项目结构。

1.  在构建类型标签中，我们需要添加一个新的构建变体类型。我们有两种构建类型，调试和发布，如下截图所示。在创建构建类型时，我们将有选项选择构建变体的签名配置：

![](img/8bfdaa86-84ef-475d-98c3-f8a730347f15.png)

这将在`build.gradle`文件中添加以下代码：

```kt
    buildTypes {
         release {
             minifyEnabled false proguardFiles getDefaultProguardFile('proguard-
             android.txt'), 'proguard-rules.pro' signingConfig signingConfigs.config }
     }
```

在应用程序世界中，为免费应用程序提供基本功能，并为付费应用程序提供高级功能是一种通用规范。Android 提供了将它们定义为**productFlavors**的选项。

免费和付费是开发人员常用的产品风味。通过定义各种产品风味，开发人员将有选择地维护不同的代码，从而为同一应用程序提供不同或额外的功能。免费和付费版本提供的共同功能的代码基础可以相同，而付费产品风味版本可以启用高级功能。

要定义产品口味，右键单击**app** |** Project Structure**，在 Flavors 选项卡中，可以定义产品口味-免费或付费。签名配置也可以自定义以匹配`productFlavors`：

![](img/0a356a2e-9508-438b-bdd1-c5fc96c1f43f.png)

`build.gradle`文件将包含以下信息：

```kt
android {
........
     productFlavors {
         paid {
             signingConfig signingConfigs.paidconfig }
         free {
             signingConfig signingConfigs.freeconfig }
     }
 }
```

# 通过命令行生成密钥库

密钥库也可以通过使用 keytool 命令行生成。keytool 可在 jdk 的`bin`目录中找到：

![](img/22b2c77a-f1e9-4695-aba5-e564962563cc.png)

启动命令提示符并运行以下命令：

```kt
keytool -genkey -v -keystore dreamindiacmd.jks -keyalg RSA -keysize 2048 -validity 10000 -alias packtcmdkey
```

![](img/f0f113f4-e4de-4e86-9980-416174331acc.png)

上述命令需要一个密钥库路径，用于密钥签名的安全算法类型，密钥大小，有效期和密钥别名。执行上述命令后，我们需要提供密码和一些其他额外的细节，如下面的屏幕截图所示：

![](img/caa30500-3927-480a-a911-2c3a334bd3ea.png)

成功执行命令后，我们可以在 keytool 的相同位置找到生成的`keystore`文件：

![](img/983beac9-97b7-469e-9d31-697c2feee2ca.png)

# 在 Google Play 商店发布应用

现在我们有签名发布版本的 APK 可用，是时候通过 Google Play 商店进行全球分发了。

注册 Google Play 商店开发者帐户需要一次性费用 25 美元。登录[`play.google.com/apps/publish/`](https://play.google.com/apps/publish/)并完成注册过程。

Google Play 商店提供了一个名为 Google Play 控制台的优秀控制台，该控制台包含了管理 Android 应用程序发布生命周期所需的所有功能。我们将看一下使我们能够发布应用的重要功能。

发布应用的第一步是在 Google Play 控制台中创建应用程序。控制台提供了创建应用程序的选项，从而启动了发布流程：

![](img/d9df6089-18c7-475c-9afa-0ca69b0589f1.png)

一旦我们点击“创建应用程序”，就会提示我们输入默认语言和应用程序的标题。点击创建按钮将为我们创建应用程序：

![](img/8a255bae-7921-4d7c-b96d-5212483e4c62.png)

开发者控制台提供了许多选项供开发者在菜单中填写。但是，有四个重要且必填的部分需要填写，以确保应用程序可以发布。

这四个部分分别是应用发布、商店列表、内容评级和定价与分发：

![](img/04b29151-2e78-4a46-a1a4-7050d5c96498.png)

现在，让我们专注于这些必填部分需要填写的细节。

# 应用发布部分

应用发布部分使开发者能够管理 APK 发布的整个生命周期。开发者可以在将 APK 移至公共分发之前，将其应用于内部测试、alpha 和 beta 发布。发布的各个阶段帮助开发者收集有关应用的反馈，通过限制应用，使其仅对特定用户可用：

![](img/4a8a61a8-455c-47e5-ad6e-e8e5dc016d4c.png)

我们需要上传为构建类型发布生成的签名 APK，以便进行生产。可以浏览 APK 文件并将其上传到 Play 商店：

![](img/8cae8811-de6a-4876-a4df-137356ab8e34.png)

一旦 APK 上传完成，可以在发布部分找到相同的版本代码和删除 APK 的选项。上传签名的 APK 完成了应用发布部分所需的详细信息：

![](img/096fdd4c-2235-4c42-bf7c-5b8aa11d4e8c.png)

# 商店列表部分

商店列表部分是接下来要关注的部分。这是一个重要的部分，因为用户将在这里看到应用的各种截图、简短和详细描述。开发人员可以选择保存草稿，并随时返回继续填写详细信息：

![](img/359584c3-af8d-4675-99f4-da36313b2a10.png)

在 Google Play 商店中，商店列表部分要求以下内容：

+   应用的两个截图

+   高分辨率图标 - 512 * 512

+   特色图形 - 1,024 W x 500 H：

![](img/d35945b8-42d4-483e-b49a-d7aa64bc8316.png)

可以使用免费的图像编辑器（如**gimp**）创建图形资产。遵循图形规范的指南非常重要且强制性。

开发人员需要提供应用程序的类型和类别以及联系方式和隐私政策（如果有的话）。一旦提供了所有详细信息，商店列表部分将完成。

# 内容评级部分

开发人员应该对应用中提供的内容进行自我声明。内容评级部分有一个问卷，要求开发人员提供具体答案。回答问卷是一项简单的任务：

![](img/d7babce2-d3f2-4444-9191-da12983dff6f.png)

开发人员提供有关应用内容的正确信息非常重要，因为提供错误信息可能会影响商店列表。

# 定价和分发部分

最后一个强制性部分，定价和分发，要求开发人员提供与其应用定价相关的信息 - 免费或付费，应用分发的国家列表，应用是否主要面向儿童，应用是否包含广告，内容指南，以及开发人员承诺遵守美国出口法的确认：

![](img/4ead3aba-fa15-4fba-9cdb-b87d0004667b.png)

一旦开发人员在定价和分发部分提供了所有必要的详细信息，将出现“准备发布”的消息。还要注意，所有四个强制性部分都标记为绿色，表示已完成：

![](img/2351c62b-d7cd-441b-b1e7-63d2dad21fed.png)

一旦应用提交发布，将在几小时内进行审核并提供下载。如果有任何疑问，开发人员需要解决并重新提交应用以发布。

# 在亚马逊应用商店发布应用

亚马逊应用商店为开发人员提供了一个免费的市场来分发他们的 Android 应用。开发人员可以登录并在以下网址创建他们的免费账户：[`developer.amazon.com/apps-and-games/app-submission/android.`](https://developer.amazon.com/apps-and-games/app-submission/android)

一旦我们登录应用商店，我们需要点击亚马逊应用商店中的“添加 Android 应用”按钮：

![](img/c12f4793-9c16-4dec-ae88-9d976ab7483a.png)

亚马逊应用商店要求填写以下部分：常规信息、可用性和定价、描述、图像和多媒体、内容评级和二进制文件。

让我们详细看看这些部分。

# 常规信息

在常规信息部分，开发人员需要提供有关应用标题、包名称、应用 ID、发布 ID、应用类别以及开发人员的联系方式的信息：

![](img/c446bd6e-a3cb-4e76-9c7d-a80e60a2fe75.png)

# 可用性和定价部分

在这一部分，开发人员需要提供以下信息：

+   应用的定价 - 免费或付费

+   国家列表

+   应用发布的日期和时间：

![](img/3cadd242-28fb-4f5f-bdaf-dfd0394e1707.png)

# 描述部分

在描述部分，开发人员需要填写有关标题、简短描述和长描述的详细信息：

![](img/11555c56-54bf-4cad-84b2-ceaac768536e.png)

该部分还使开发人员能够提供产品特色项目和识别应用的特定关键字。用户还可以选择添加本地化描述：

![](img/3f73a9a7-764c-4fb2-ad00-6dc976fe6ec6.png)

# 图像和多媒体部分

在图像和多媒体部分，开发人员需要输入与应用相关的图形资产。用户需要提供： 

+   **图标**：512 * 512 PNG 和 114 * 114 PNG

+   **屏幕截图**：3 到 10 个 PNG 或 JPG

还有一个选项可以提供与平板电脑和手机等形态因素相关的图形：

![](img/f5f4f1c9-ae66-4fdd-a738-d1527f517065.png)

# 内容评级部分

在内容评级部分，开发人员需要回答一系列与应用中显示的内容性质相关的问题。这些问题属于主题：

![](img/0cd6474e-a7d1-4452-a00b-8751e678ba4f.png)

开发人员需要回答关于使用基于位置的服务、应用中的广告、提供隐私政策（如果有的话）以及披露应用是否面向 13 岁以下儿童的问题：

![](img/ecc4aa67-40d5-4c04-90bd-f3fe1b02f53e.png)

# 二进制文件部分

在此部分，开发人员应上传从 Android Studio 或命令行生成的已签名 APK：

![](img/39094e6c-ad97-4a9e-8066-7ebae7a6288c.png)

开发人员还可以决定设备支持、语言支持、出口合规性和使用亚马逊地图重定向的选项：

![](img/16c9800e-def0-4976-8a77-640b27a3725e.png)

非亚马逊 Android 设备的设备支持默认情况下是未启用的。开发人员需要通过单击“编辑设备支持”并进行所需更改来显式启用此功能。

填写完所有必需信息后，现在是时候在亚马逊应用商店中实际发布应用了。开发人员将有一个选项来审查他们输入的所有信息：

![](img/c9258231-4b5a-419d-98a8-164c1bebf487.png)

# 摘要

商店列表、关键字、描述等在应用识别和最终应用及开发人员的成功方面起着重要作用。

在本章中，我们讨论了使用 Android Studio 生成密钥库文件、自动签名 APK、从命令行生成密钥库文件以及通过 Google Play 商店和亚马逊应用商店发布应用所涉及的各种步骤。

在下一章中，我们将学习如何使用我们可以使用的最有趣和重要的 API 之一——Google Faces API。Google Faces API 使开发人员能够提供诸如面部检测、照片中人物的识别等酷功能。
