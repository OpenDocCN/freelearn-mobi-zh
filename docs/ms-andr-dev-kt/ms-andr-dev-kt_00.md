# 前言

Android 是移动设备最流行的平台。每年都有越来越多的开发人员参与 Android 开发。Android 框架使得可以为手机、平板电脑、电视等开发应用成为可能！到目前为止，所有的开发都是用 Java 完成的。最近，Google 宣布 Kotlin 作为开发人员可以使用的第二种语言。因此，鉴于 Kotlin 日益增长的受欢迎程度，我们决定介绍使用 Kotlin 作为其主要开发编程语言的 Android。

有了 Kotlin，你可以做任何你用 Java 做的事情，但更加愉快和有趣！我们将向你展示如何在 Android 和 Kotlin 中玩耍，以及如何创造令人惊叹的东西！多亏了 Kotlin，可以肯定 Android 平台会进一步发展。在不久的将来，Kotlin 有可能成为该平台的主要开发语言。坐稳，准备开始一段伟大的旅程吧！

# 本书涵盖的内容

第一章，“开始 Android”，教你如何使用 Kotlin 开始 Android 开发，以及如何设置你的工作环境。

第二章，“构建和运行”，向你展示如何构建和运行你的项目。它将演示如何记录和调试应用程序。

第三章，“屏幕”，从 UI 开始。在这一章中，我们将为我们的应用程序创建第一个屏幕。

第四章，“连接屏幕流”，解释了如何连接屏幕流并定义与 UI 的基本用户交互。

第五章，“外观和感觉”，涵盖了 UI 的主题。我们将向你介绍 Android 主题的基本概念。

第六章，“权限”，解释了为了利用某些系统功能，需要获取适当的系统权限，这将在本章中讨论。

第七章，“使用数据库”，向你展示如何使用 SQLite 作为应用程序的存储。你将创建一个数据库来存储和共享数据。

第八章，“Android 偏好设置”，指出并非所有数据都应存储在数据库中；一些信息可以存储在共享偏好设置中。我们将解释原因和方法。

第九章，“Android 中的并发”，解释了如果你熟悉编程中的并发，那么你会知道在软件中许多事情是同时发生的。Android 也不例外！

第十章，“Android 服务”，介绍了 Android 服务以及如何使用它们。

第十一章，“消息”，说在 Android 中，你的应用程序可以监听各种事件。如何做到这一点将在本章中得到解答。

第十二章，“后端和 API”，连接到远程后端实例以获取数据。

第十三章，“为高性能进行调优”，是一个完美的章节，当你不确定你的应用程序是否足够快时，它会给你答案。

第十四章，“测试”，提到在发布任何东西之前，我们必须对其进行测试。在这里，我们将解释如何为你的应用程序编写测试。

第十五章，“迁移到 Kotlin”，指导你如果计划将现有的 Java 代码库迁移到 Kotlin。

第十六章，“部署你的应用程序”，指导你完成部署过程。我们将发布本书中开发的所有内容。

# 本书所需内容

对于本书，需要运行 Microsoft Windows、Linux 或 macOS 的现代计算机。您需要安装 Java JDK、Git 版本控制系统和 Android Studio。

为了运行所有代码示例和您编写的代码，您需要一部运行 Android 操作系统版本>= 5 的 Android 手机。

# 本书适合对象

本书旨在希望以简单有效的方式构建令人惊叹的 Android 应用程序的开发人员。假定具有 Kotlin 的基本知识，但不熟悉 Android 开发。

# 约定

在本书中，您将找到一些区分不同类型信息的文本样式。以下是这些样式的一些示例及其含义的解释。

文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL 和用户输入显示如下：“我们将为`Application`类的每个生命周期事件和我们创建的屏幕（活动）添加适当的日志消息。”

代码块设置如下：

```kt
    override fun onCreate(savedInstanceState: Bundle?) { 
      super.onCreate(savedInstanceState) 
      setContentView(R.layout.activity_main) 
      Log.v(tag, "[ ON CREATE 1 ]") 
    } 
```

任何命令行输入或输出都是这样写的。输入命令可能会被分成几行以增加可读性，但需要作为一个连续的行输入到提示符中：

```kt
sudo apt-get install libc6:i386 libncurse
libstdc++6:i386 lib32z1 libbz2-1.0:i386
```

**新术语**和**重要单词**以粗体显示。例如，屏幕上看到的单词，例如菜单或对话框中的单词，会出现在文本中，如下所示：“选择**工具**|**Android**|**AVDManager**或单击工具栏中的 AVDManager 图标。”

警告或重要说明会出现在这样。

提示和技巧会出现在这样。

# 读者反馈

我们的读者的反馈总是受欢迎的。让我们知道您对这本书的看法--您喜欢或不喜欢什么。读者的反馈对我们很重要，因为它可以帮助我们开发出您真正能够充分利用的标题。

要向我们发送一般反馈，只需发送电子邮件至 feedback@packtpub.com，并在消息主题中提及书名。

如果您在某个主题上有专业知识，并且有兴趣撰写或为书籍做出贡献，请参阅我们的作者指南，网址为[www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

现在您是 Packt 书籍的自豪所有者，我们有一些事情可以帮助您充分利用您的购买。

# 下载示例代码

您可以从您在[`www.packtpub.com`](http://www.packtpub.com)的帐户中下载本书的示例代码文件。如果您在其他地方购买了这本书，您可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)并注册，以便文件直接通过电子邮件发送给您。

您可以按照以下步骤下载代码文件：

1.  使用您的电子邮件地址和密码登录或注册到我们的网站。

1.  将鼠标指针悬停在顶部的“支持”选项卡上。

1.  点击“代码下载和勘误”。

1.  在搜索框中输入书名。

1.  选择您要下载代码文件的书籍。

1.  从下拉菜单中选择您购买本书的地方。

1.  点击“代码下载”。

下载文件后，请确保使用最新版本的解压缩或提取文件夹：

+   WinRAR / 7-Zip 适用于 Windows

+   Zipeg / iZip / UnRarX 适用于 Mac

+   7-Zip / PeaZip 适用于 Linux

本书的代码包也托管在 GitHub 上，网址为[`github.com/PacktPublishing/-Mastering-Android-Development-with-Kotlin/branches/all`](https://github.com/PacktPublishing/-Mastering-Android-Development-with-Kotlin/branches/all)。我们还有来自丰富书籍和视频目录的其他代码包，可在[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)上找到。去看看吧！

# 下载本书的彩色图片

我们还为您提供了一个 PDF 文件，其中包含本书中使用的屏幕截图/图表的彩色图像。彩色图像将帮助您更好地理解输出的变化。您可以从[`www.packtpub.com/sites/default/files/downloads/MasteringAndroidDevelopmentwithKotlin_ColorImages.pdf`](https://www.packtpub.com/sites/default/files/downloads/MasteringAndroidDevelopmentwithKotlin_ColorImages.pdf)下载此文件。

# 勘误

尽管我们已经非常小心地确保内容的准确性，但错误是难免的。如果您在我们的书籍中发现错误——可能是文本或代码中的错误——我们将不胜感激地希望您能向我们报告。通过这样做，您可以帮助其他读者避免挫折，并帮助我们改进本书的后续版本。如果您发现任何勘误，请访问[`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)，选择您的书籍，点击“勘误提交表格”链接，并输入您的勘误详情。一旦您的勘误经过验证，您的提交将被接受，并且勘误将被上传到我们的网站或添加到该书籍“勘误”部分的任何现有勘误列表中。

要查看先前提交的勘误，请访问[`www.packtpub.com/books/content/support`](https://www.packtpub.com/books/content/support)，并在搜索框中输入书名。所需信息将出现在“勘误”部分下。

# 盗版

互联网上盗版受版权保护的材料是所有媒体的持续问题。在 Packt，我们非常重视版权和许可的保护。如果您在互联网上发现我们作品的任何非法副本，请立即向我们提供位置地址或网站名称，以便我们采取补救措施。

请通过 copyright@packtpub.com 与我们联系，并提供涉嫌盗版材料的链接。

我们感谢您在保护我们的作者和我们为您提供有价值的内容的能力方面的帮助。

# 问题

如果您对本书的任何方面有问题，您可以通过 questions@packtpub.com 与我们联系，我们将尽力解决问题。
