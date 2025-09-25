# 前言

Kotlin 是由 JetBrains 创建的一种编程语言，它在 **Java 虚拟机**（**JVM**）上运行。它旨在解决诸如冗长、空指针异常、并发挑战以及在 Java 中发现的缺乏功能支持等问题。Kotlin 提供了一种更现代、更简洁的编程方法，同时仍然与现有的 Java 代码和库兼容。谷歌将 Kotlin 识别为构建 Android 应用程序的主要语言，这导致了支持开发者的重大努力。本书采用行业导向的方法，为您在任意公司担任 Android 开发者的角色做好准备。它遵循谷歌 Android 团队推荐的最佳实践，并基于实践经验提供见解。

通过实际示例，本书引导您使用 Kotlin 开发 Android 应用程序，传授成为熟练 Android 开发者所必需的实践经验。内容包括使用 Jetpack Compose 构建 App、融入 Material Design 3 以实现个性化体验，以及在 MVVM 架构中构建 App。指南进一步展示了如何通过依赖注入、使用 Room 等 Jetpack 库进行本地数据持久化、实施调试技术等功能来增强您的应用程序架构。它还涵盖了测试、使用 Ktlint 和 Detekt 等工具识别代码问题，并指导您完成 Google Play 商店的发布流程。探索通过 GitHub Actions 自动化连续发布以及使用 Firebase App Distribution 分发测试构建。此外，本书还讨论了应用程序改进策略，包括崩溃报告工具、提高用户参与度的技巧以及确保应用程序安全性的见解。

# 本书面向对象

本书面向有志成为 Android 开发者或正在使用 Java 进行 Android 开发的 Android 开发者，因为他们将学习如何从头开始使用 Kotlin 构建 Android 应用程序，了解 Android 中的架构和其他相关主题，并最终了解如何将他们的应用程序发布到 Google Play 商店。本书以当前最佳实践为出发点，并指导您为 Android 开发者的角色做好准备。

# 本书涵盖内容

*第一章*，*Kotlin Android 开发入门*，介绍了 Kotlin 作为一种编程语言。它涵盖了适用于 Android 开发的特性及其对 Android 开发者的重要性。此外，它还涵盖了如何从 Java 迁移到 Kotlin，以及针对来自 Java 背景的开发者的有用提示。

*第二章*，*创建您的第一个 Android 应用程序*，涵盖了如何创建 Android 应用程序。它使您熟悉 Android Studio，这是您将用于开发 Android 应用程序的 **集成开发环境**（**IDE**）。它还涵盖了一些技巧、快捷方式和有用的 Android Studio 功能，并探讨了在 Android Studio 中创建项目的流程。

*第三章*，*Jetpack Compose 布局基础*，探讨了 Jetpack Compose，一种用于创建应用用户界面的声明式方法。它涵盖了 Jetpack Compose 的基础及其布局。

*第四章*，*使用 Material Design 3 进行设计*，介绍了 Material 3 及其提供的功能。它还涵盖了如何在 Android 应用中使用 Material 3 以及其一些组件。

*第五章*，*构建您的应用架构*，探讨了 Android 项目中可用的不同架构。它深入探讨了 MVVM 架构及其不同层以及如何在其中使用一些 Jetpack 库。此外，它还展示了如何使用高级架构功能，例如依赖注入、Kotlin Gradle DSL 和版本目录来定义依赖项。

*第六章*，*使用 Kotlin 协程进行网络调用*，讨论了如何使用网络库 Retrofit 进行网络调用。它展示了如何使用此库消费 **应用程序编程接口**（**API**）。此外，它还涵盖了如何利用 Kotlin 协程执行异步网络请求。

*第七章*，*在您的应用中导航*，解释了如何使用 Jetpack Compose 导航库导航到不同的 Jetpack Compose 屏幕。它涵盖了使用此库的技巧和最佳实践。此外，它还涵盖了如何在导航到屏幕时传递参数。最后，它还涵盖了如何在大型屏幕和可折叠设备上处理导航。

*第八章*，*本地持久数据和后台工作*，介绍了如何将数据保存到本地数据库 Room，它是 Jetpack 库的一部分。它展示了如何保存项目和从 Room 数据库中读取。此外，它还涵盖了如何使用 WorkManager 执行长时间运行的操作以及一些最佳实践。

*第九章*，*运行时权限*，深入探讨了运行时权限以及如何请求运行时权限。

*第十章*，*调试您的应用*，讨论了调试技巧和窍门，如何使用 LeakCanary 检测泄漏，如何使用 Chucker 检查由应用触发的网络请求/响应，以及如何使用 App Inspection 检查 Room 数据库、网络请求和后台任务。

*第十一章*，*提升代码质量*，探讨了 Kotlin 风格和编写 Kotlin 代码的最佳实践。它还演示了如何使用 Ktlint 和 Detekt 等插件来格式化、检查和早期检测代码异味。

*第十二章*，*测试您的应用*，检查了如何在 MVVM 架构的不同层添加测试。它涵盖了添加测试的重要性以及如何添加单元测试、集成测试和仪器测试。

*第十三章*，*发布您的应用*，深入探讨了如何将新应用发布到 Google Play Store。它逐步介绍了如何创建签名应用包，以及诸如回答有关我们应用内容的问题、创建发布版本、设置用户如何访问应用（无论是通过受控测试轨道还是公开）等主题。此外，它还涵盖了 Google Play Store 的一些政策以及如何始终遵守规定，以避免应用被移除或账户被封禁。

*第十四章*，*持续集成与持续部署*，专注于如何使用 GitHub Actions 自动化一些手动任务，例如将新构建部署到 Google Play Store。它还涵盖了如何在 CI/CD 管道上运行测试，并使用 GitHub Actions 将构建推送到 Google Play Store。

*第十五章*，*改进您的应用*，介绍了通过添加分析、Firebase Crashlytics 以及使用云消息来提高应用用户参与度的技术。它涵盖了如何从 Firebase 控制台向应用发送通知。此外，它还提供了确保用户数据不受损害的应用安全提示和技巧。

# 要充分利用本书

| **本书涵盖的软件/硬件** | **操作系统要求** |
| --- | --- |
| Android Studio Hedgehog 或更高版本 | Windows、macOS 或 Linux |

# 下载示例代码文件

您可以从 GitHub 下载本书的示例代码文件：[`github.com/PacktPublishing/Mastering-Kotlin-for-Android`](https://github.com/PacktPublishing/Mastering-Kotlin-for-Android)。如果代码有更新，它将在 GitHub 仓库中更新。

我们还有其他丰富的代码包，这些代码包来自我们的书籍和视频目录，可在[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)找到。查看它们吧！

# 使用的约定

本书使用了多种文本约定。

`文本中的代码`：表示文本中的代码词汇、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 昵称。以下是一个示例：“我们创建了一个名为`PetsRepositoryImpl`的类，该类实现了`PetsRepository`接口。”

代码块设置如下：

```java
class PetsViewModel: ViewModel() {
    private val petsRepository: PetsRepository = PetsRepositoryImpl()
    fun getPets() = petsRepository.getPets()
}
```

当我们希望您注意代码块中的特定部分时，相关的行或项目将以粗体显示：

```java
[default]
exten => s,1,Dial(Zap/1|30)
exten => s,2,Voicemail(u100)
exten => s,102,Voicemail(b100)
exten => i,1,Voicemail(s0)
```

任何命令行输入或输出都按以下方式编写：

```java
FATAL EXCEPTION: main
Process: com.packt.chapterten, PID: 7168
java.lang.RuntimeException: Unable to start activity ComponentInfo{com.packt.chapterten/com.packt.chapterten.MainActivity}: java.lang.RuntimeException: This is a crash
at android.app.ActivityThread.performLaunchActivity(ActivityThread.java:3645)
at android.app.ActivityThread.handleLaunchActivity(ActivityThread.java:3782)
```

**粗体**：表示新术语、重要词汇或屏幕上看到的词汇。例如，菜单或对话框中的单词以**粗体**显示。以下是一个示例：“点击**新建项目**按钮，这将带我们进入**模板**屏幕。”

提示或重要注意事项

看起来像这样。

# 联系我们

欢迎读者反馈。

`customercare@packtpub.com`并在邮件主题中提及本书标题。

**勘误表**：尽管我们已经尽最大努力确保内容的准确性，但错误仍然可能发生。如果你在这本书中发现了错误，我们非常感谢你向我们报告。请访问 [www.packtpub.com/support/errata](http://www.packtpub.com/support/errata) 并填写表格。

`copyright@packt.com` 并附上材料的链接。

**如果你有兴趣成为作者**：如果你在某个领域有专业知识，并且你感兴趣的是撰写或为书籍做出贡献，请访问 [authors.packtpub.com](http://authors.packtpub.com)。

# 分享你的想法

一旦你阅读了 *Mastering Kotlin for Android 14*，我们很乐意听听你的想法！请[点击此处直接进入此书的亚马逊评论页面](https://packt.link/r/1837631719)并分享你的反馈。

你的评论对我们和科技社区都很重要，并将帮助我们确保我们提供高质量的内容。

# 下载此书的免费 PDF 复印本

感谢你购买这本书！

你喜欢在路上阅读，但无法携带你的印刷书籍到处走吗？

你的电子书购买是否与你的选择设备不兼容？

别担心，现在每购买一本 Packt 书，你都可以免费获得该书的 DRM-free PDF 版本。

在任何地方、任何设备上阅读。直接从你最喜欢的技术书籍中搜索、复制和粘贴代码到你的应用中。

好处不止于此，你还可以获得独家折扣、时事通讯和每日收件箱中的精彩免费内容。

按照以下简单步骤获取好处：

1.  扫描下面的二维码或访问以下链接

![下载此书的免费 PDF 复印本](img/B19779_QR_Free_PDF.jpg)

[`packt.link/free-ebook/9781837631711`](https://packt.link/free-ebook/9781837631711)

1.  提交你的购买证明

1.  就这些！我们将直接将你的免费 PDF 和其他好处发送到你的电子邮件。

# 第一部分：构建你的应用

在这部分，你将开始一段探索 Kotlin 的旅程，了解使其成为 Android 开发最佳选择的特性。我们将引导你通过从 Java 迁移的过程，为从 Java 背景过渡的开发者提供有价值的见解。逐步深入，你将构建你的首个 Android 应用，包括设置你的开发环境，以及熟悉 Android Studio。重点扩展到掌握 Jetpack Compose，揭示制作直观用户界面的艺术。最后，我们将向你介绍如何将 Material Design 3 集成到你的应用中，阐明它为开发领域带来的丰富特性和组件。

本节包含以下章节：

+   *第一章*，*开始 Kotlin Android 开发*

+   *第二章*，*创建你的首个 Android 应用*

+   *第三章*，*Jetpack Compose 布局基础*

+   *第四章*, *使用 Material Design 3 进行设计*
