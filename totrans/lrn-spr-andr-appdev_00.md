# 前言

本书旨在开发应用程序的服务器端和客户端。我们为服务器端和客户端都使用了 Kotlin 语言。在本书中，Spring 将作为服务器端应用，Android 作为客户端应用。我们的主要重点是那些能够帮助开发者使用最新架构开发安全应用的区域。本书描述了 Kotlin 和 Spring 的基础知识，如果你对这些平台不熟悉，这将很有益。我们还设计了关于在项目中实现安全和数据库的章节。本书深入探讨了 Retrofit 在处理 HTTP 请求中的应用，以及 SQLite Room 在 Android 设备中存储数据。你还将找到一种开发健壮、响应式项目的方法。然后，你将学习如何使用 JUnit 和 Espresso 测试项目，以开发一个更少错误和更稳定的工程。

# 本书面向对象

本书是为那些希望使用 Spring 和 Android 开发项目的 Kotlin 新手开发者设计的。Spring for Android 提供了一个功能性的 REST 客户端，支持从 JSON 中序列化对象。开发者依赖于其他语言平台，如 PHP 和 Python 进行 REST API 开发，但 Spring 与 Java/Kotlin 结合，并提供了丰富的内容，帮助开发者以最大安全性使用 REST API。应用程序代码中存在一些依赖项，Spring 会移除这些依赖项。如今，Java 正在被 Kotlin 取代，Kotlin 更轻量级，需要更少的代码行来完成工作。

# 本书涵盖内容

第一章，*关于环境*，为服务器端和客户端创建了一个环境。我们还将探讨项目所需的各种工具类型。我们将了解使用 Spring 和 Android 平台可以创建什么。

第二章，*Kotlin 概述*，涵盖了 Kotlin 的基础知识，并探讨了如何设置环境以及 Kotlin 可用的工具或 IDE，包括基本语法和类型。我们将看到流程结构，包括 `if-else` 语句、`for` 循环和 `while` 循环。我们还将探讨 Kotlin 的面向对象编程，包括类、接口、对象等。函数也将被涵盖，包括参数、构造函数和语法。我们还将解释空安全、反射和注解，这些都是 Kotlin 的核心特性。

第三章，*Spring 框架概述*，涵盖了 Spring 框架的基础知识，读者将学习如何配置 Spring 和 bean。本章将解释依赖注入，以及 Spring 的架构。读者将了解 Spring MVC 和 Spring Boot，这些对于快速开发应用程序非常有帮助。还将解释 Spring 数据模块。我们还将介绍 Spring Security，它为应用程序提供身份验证和其他安全功能。

第四章，*Android 的 Spring 模块*，涵盖了与 Android 项目相关的 RestTemplate 和 Retrofit 模块。提供了 HTTP 客户端的解释。还将涵盖对象到 JSON 的序列化。我们将学习如何启动和设置环境。RestTemplate 和 Retrofit 模块的 HTTP 请求方法，如`POST`、`GET`、`UPDATE`和`DELETE`，以及其他 Spring 模块的常见功能，以及 Maven 依赖管理也将被介绍。

第五章，*使用 Spring Security 保护应用程序*，涵盖了 Spring Security 的要求。我们将学习如何在 Web 服务器中注册和配置安全和身份验证。我们还将了解 Spring Security 的架构以及如何将其用于客户端。我们将看到为 Android 应用程序保护 API 的方法以及安全流程。我们将学习如何与 REST API 相关联使用 Spring Security。还将讨论基本身份验证、OAuth2、隐式流和授权代码流的使用。我们还将学习如何与 Android 项目连接并使用基本身份验证。

第六章，*访问数据库*，涵盖了现有的 Spring 数据模块。我们还将介绍 JDBC、JPA、H2、MySQL for Spring 以及 Android 的 SQLite Room。我们还将学习如何使用 JPA 在 Spring 中创建 REST API 以及如何在 Android 中获取 API 和处理内容。

第七章，*并发*，涵盖了协程，包括并发、并行和线程池等内容。我们还将学习关于顺序操作和回调地狱的知识。

第八章，*响应式编程*，涵盖了响应式编程相关主题，包括 Spring Reactor 和阻塞。在本章中，读者还将学习 RxJava 和 RxAndroid。

第九章 Creating an Application，*创建应用程序*，从安装 Android 环境开始。然后，我们将在 Web 服务器上配置 Spring 并设计项目。然后，我们将创建 UI、布局和 RESTful Web 服务，并从 API 中检索 JSON。我们还将学习使用 Spring Boot 和 Spring Security 为应用程序提供支持。然后，我们将学习如何使用基本认证来保护数据并允许用户访问。我们将使用安全的 REST API 为 Android 应用程序提供支持，以及如何在 Android 中处理内容。此应用程序将基于 Kotlin，我们将利用 Kotlin 的功能，包括空安全、反射和注解等功能。

第十章 Testing an Application，*测试应用程序*，涉及 Spring 测试。这包括单元测试、集成测试和 UI 测试，以及它们的应用。我们将了解项目的测试结构，以及 JUnit 和 Espresso 等测试工具。还将讨论 JUnit 和 JPA 的测试用例。我们将学习如何编写 Android 应用程序的 UI 测试用例。我们还将学习如何通过 Android Studio 执行这些测试。我们还将学习如何使用 Kotlin 中的 Espresso 测试 UI，以及它在 Android 应用程序中的相关应用。我们还将探讨应用程序中的并发和响应式编程。

# 为了充分利用本书

对 Spring 和 Kotlin 的基本了解将有所帮助，但不是必需的。运行本书的代码示例需要 MySQL Workbench 数据库、Eclipse 或 IntelliJ IDEA 用于 Spring、Android Studio 用于 Android，以及 Postman 或 Insomnia REST 客户端。

# 下载示例代码文件

您可以从[www.packt.com](http://www.packt.com)的账户下载本书的示例代码文件。如果您在其他地方购买了本书，您可以访问[www.packt.com/support](http://www.packt.com/support)并注册，以便将文件直接通过电子邮件发送给您。

您可以通过以下步骤下载代码文件：

1.  在[www.packt.com](http://www.packt.com)登录或注册。

1.  选择支持选项卡。

1.  点击代码下载与勘误表。

1.  在搜索框中输入书名，并遵循屏幕上的说明。

一旦文件下载完成，请确保您使用最新版本的以下软件解压缩或提取文件夹：

+   WinRAR/7-Zip（适用于 Windows）

+   Zipeg/iZip/UnRarX（适用于 Mac）

+   7-Zip/PeaZip（适用于 Linux）

该书的代码包也托管在 GitHub 上，网址为[`github.com/PacktPublishing/Learn-Spring-for-Android-Application-Development/`](https://github.com/PacktPublishing/Learn-Spring-for-Android-Application-Development/)。如果代码有更新，它将在现有的 GitHub 仓库中更新。

我们还有其他丰富的图书和视频的代码包，可在[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)找到。查看它们！

# 下载彩色图片

我们还提供了一份包含本书中使用的截图/图表彩色图像的 PDF 文件。您可以从这里下载：[`www.packtpub.com/sites/default/files/downloads/9781789349252_ColorImages.pdf`](https://www.packtpub.com/sites/default/files/downloads/9781789349252_ColorImages.pdf)。

# 使用约定

本书中使用了多种文本约定。

`CodeInText`：表示文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 昵称。以下是一个示例：“`switch { ... }`控制流元素被`when { ... }`替换。”

代码块设置如下：

```kt
fun test() {
    Bar.NAME
    Bar.printName()
}
```

当我们希望您注意代码块中的特定部分时，相关的行或项目将以粗体显示：

```kt
<!-- A bean example with singleton scope -->
<bean id = "..." class = "..." scope = "singleton"/>
<!-- You can remove the scope for the singleton -->
<bean id = "..." class = "..."/>
```

任何命令行输入或输出都应如下所示：

```kt
$ mkdir css
$ cd css
```

**粗体**：表示新术语、重要单词或屏幕上看到的单词。例如，菜单或对话框中的单词在文本中显示如下。以下是一个示例：“从管理面板中选择系统信息。”

警告或重要提示如下所示。

小贴士和技巧如下所示。

# 联系我们

我们始终欢迎读者的反馈。

**一般反馈**：如果您对本书的任何方面有疑问，请在邮件主题中提及书名，并给我们发送邮件至`customercare@packtpub.com`。

**勘误**：尽管我们已经尽最大努力确保内容的准确性，但错误仍然可能发生。如果您在这本书中发现了错误，我们将非常感激您能向我们报告。请访问[www.packt.com/submit-errata](http://www.packt.com/submit-errata)，选择您的书籍，点击勘误提交表单链接，并输入详细信息。

**盗版**：如果您在互联网上发现我们作品的任何非法副本，我们将非常感激您能提供位置地址或网站名称。请通过`copyright@packt.com`与我们联系，并提供材料的链接。

**如果您有兴趣成为作者**：如果您在某个领域有专业知识，并且您有兴趣撰写或为书籍做出贡献，请访问[authors.packtpub.com](http://authors.packtpub.com/)。

# 评论

请留下评论。一旦您阅读并使用了这本书，为何不在购买它的网站上留下评论呢？潜在读者可以查看并使用您的客观意见来做出购买决定，Packt 团队可以了解您对我们产品的看法，我们的作者也可以看到他们对书籍的反馈。谢谢！

想了解更多关于 Packt 的信息，请访问[packt.com](http://www.packt.com/)。
