# 前言

本书旨在为在移动、桌面或网络平台上工作的开发者提供指导，帮助他们学习如何为可穿戴设备（也称为 wear apps）构建应用程序。此外，您可能已经在 Google Play 商店上发布了应用程序，并希望为现有的 Android 应用提供 Android Wear 支持。如果这两个陈述中的任何一个是真的，那么这本书就是为您准备的。

本书的主要目标是向您，读者，提供对构建设计良好且稳健的 Android Wear 应用程序所涉及的哲学、思维过程、开发细节和方法论有一个坚实的理解。我们将讨论可穿戴计算范式的优缺点，并在此过程中，希望为构建满足实际和现实世界用例的可穿戴应用提供一个坚实的基础。

我们将探讨从基础到中级再到高级的广泛概念和功能，复杂程度各异。每章附带的代码示例旨在让您通过实际操作了解使用工具、库、SDK 和其他相关技术来构建 Android Wear 应用所需的知识。

随着您阅读本书的章节，您可以期待实现以下目标：

+   理解可穿戴计算技术

+   使用 Android Studio 为构建 Android Wear 应用设置开发环境

+   开始掌握 Android Wear SDK 和 API

+   理解围绕 Android Wear 应用开发的常用 UI 模式和 UX 原则

+   与不同形态的可穿戴设备（圆形、方形）协同工作

+   利用 Android Wear 设备上的传感器

+   开发 Android Wear 示例应用以尝试您所学的概念

+   在 Android 移动（手持）应用和 Android Wear 应用之间进行通信

+   学习如何将 Android Wear 应用发布到 Google Play 商店

# 本书涵盖内容

第一章, *可穿戴计算简介*, 介绍了可穿戴计算的基本知识以及该技术的演变过程。它还包括了关于移动计算、普适计算和云计算的讨论。

第二章, *设置开发环境*, 重点介绍让读者熟悉设置开发环境的过程，从 IDE 安装到讨论 Android Wear 开发所需的 SDK 和库。

第三章, *开发 Android Wear 应用*, 指导读者逐步学习如何使用 Android Studio 从零开始开发 Android Wear 应用程序，即 Today 应用。

第四章, *开发表盘 UI*，通过使用 Android Wear SDK 中可用的 UI 组件扩展 Today 应用，并使用自定义布局构建自定义 UI 组件。

第五章, *同步数据*，介绍了需要伴侣手持应用的想法，包括将手持设备与 Android Wear 模拟器配对的步骤，从而扩展了你的可穿戴应用开发环境。Today 应用进一步扩展以演示这些概念。

第六章, *上下文通知*，讨论了 Android Wear 中的通知，并通过 On This Day 活动扩展 Today 应用来演示 Android Wear 通知 API。

第七章, *语音交互、传感器和跟踪*，讨论了 Wear API 提供的语音功能。我们定义了一个语音操作来启动我们的应用。我们介绍了设备传感器，并讨论了如何使用它们来跟踪数据。

第八章, *创建自定义 UI*，涵盖了 Android Wear UI 空间中至关重要的设计原则，并考察了几种常见的 Wear UI 模式。我们还增强了 On This Day 活动，以便以用户友好的格式显示。

第九章, *材料设计*，提供了对材料设计的概念理解，并触及了几个特定于可穿戴应用设计和开发的几个关键原则。通过将前几章中的 Todo 应用扩展以包含一个导航抽屉，我们可以在此抽屉中切换待办事项类别、查看项目并执行针对每个类别的特定操作，从而巩固我们的理解。

第十章, *表盘*，在本章中介绍了表盘的概念。在简要概述了可用于帮助我们开发表盘的 Android Wear API 之后，我们开发了一个简单的交互式表盘。

第十一章, *高级功能和概念*，描述了与使应用始终运行相关的 API 功能和设计考虑因素。我们开发了一个活动来演示 Wear API 提供的始终在线功能。我们还简要讨论了通过蓝牙连接调试可穿戴应用。

第十二章, *将应用发布到 Google Play*，讨论了可用于测试 Android Wear 应用的工具以及如何自动化 UI 测试。我们通过逐步说明使应用准备发布的过程来结束本章。

# 你需要为此书准备的东西

您将需要以下工具集来尝试书中的代码并自己练习应用开发：

+   Android Studio v2 或更高版本

+   JDK v7 或更高版本

+   Git 版本控制

+   一个具有良好硬件配置的开发系统，例如用于开发移动应用的高速 CPU 和足够的 RAM

# 本书面向的对象

Java 应用程序开发者——无论是 Web、桌面还是移动，都希望接触 Android Wear 平台并掌握开发 Android Wear 应用所需的知识。

# 约定

在本书中，您将找到许多文本样式，用于区分不同类型的信息。以下是一些这些样式的示例及其含义的解释。

文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 昵称如下所示："我们可以通过使用`include`指令来包含其他上下文。"

代码块设置如下：

```java
public static void Main(string[] args)
```

```java
{

```

```java
var host = new WebHostBuilder()

```

```java
.UseKestrel()

```

```java
}
```

任何命令行输入或输出都应如下书写：

```java
vi run json
```

新术语和重要词汇以粗体显示。您在屏幕上看到的单词，例如在菜单或对话框中，在文本中如下所示："点击**下一步**按钮将您带到下一个屏幕。"

### 注意

警告或重要注意事项以如下框的形式出现。

### 小贴士

小技巧和技巧如下所示。

# 读者反馈

我们始终欢迎读者的反馈。告诉我们您对这本书的看法——您喜欢或不喜欢什么。读者反馈对我们很重要，因为它帮助我们开发您真正能从中获得最大收益的标题。要发送一般反馈，请简单地发送电子邮件至 feedback@packtpub.com，并在邮件主题中提及书籍的标题。如果您在某个主题领域有专业知识，并且您有兴趣撰写或为书籍做出贡献，请参阅我们的作者指南[www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

现在您已经是 Packt 书籍的骄傲拥有者，我们有一些可以帮助您充分利用购买的东西。

## 下载示例代码

您可以从[`www.packtpub.com`](http://www.packtpub.com)的账户下载此书的示例代码文件。如果您在其他地方购买了此书，您可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)并注册，以便将文件直接通过电子邮件发送给您。

您可以通过以下步骤下载代码文件：

1.  使用您的电子邮件地址和密码登录或注册我们的网站。

1.  将鼠标指针悬停在顶部的“支持”标签上。

1.  点击“代码下载与勘误”。

1.  在搜索框中输入书籍名称。

1.  选择您想要下载代码文件的书籍。

1.  从下拉菜单中选择您购买此书的来源。

1.  点击“代码下载”。

下载文件后，请确保您使用最新版本的以下软件解压缩或提取文件夹：

+   Windows 版的 WinRAR / 7-Zip

+   Zipeg / iZip / UnRarX for Mac

+   7-Zip / PeaZip for Linux

本书的相关代码包也托管在 GitHub 上，网址为[`github.com/PacktPublishing/Mastering-Android-Wear-Application-Development`](https://github.com/PacktPublishing/Mastering-Android-Wear-Application-Development)。我们还有其他来自我们丰富图书和视频目录的代码包，可在[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)找到。查看它们吧！

## 下载本书的颜色图像

我们还为您提供了一个包含本书中使用的截图/图表的颜色图像的 PDF 文件。这些颜色图像将帮助您更好地理解输出的变化。您可以从[`www.packtpub.com/sites/default/files/downloads/MasteringAndroidWearApplicationDevelopment_ColorImages.pdf`](https://www.packtpub.com/sites/default/files/downloads/MasteringAndroidWearApplicationDevelopment_ColorImages.pdf)下载此文件。

## 勘误

尽管我们已经尽最大努力确保内容的准确性，错误仍然可能发生。如果您在我们的某本书中发现错误——可能是文本或代码中的错误——如果您能向我们报告这一点，我们将不胜感激。通过这样做，您可以避免其他读者感到沮丧，并帮助我们改进本书的后续版本。如果您发现任何勘误，请通过访问[`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)，选择您的书籍，点击勘误提交表单链接，并输入您的勘误详情来报告它们。一旦您的勘误得到验证，您的提交将被接受，勘误将被上传到我们的网站或添加到该标题的勘误部分下的现有勘误列表中。

要查看之前提交的勘误，请访问[`www.packtpub.com/books/content/support`](https://www.packtpub.com/books/content/support)，并在搜索字段中输入书籍名称。所需信息将出现在勘误部分下。

## 侵权

互联网上对版权材料的侵权是一个持续存在的问题，跨越所有媒体。在 Packt，我们非常重视我们版权和许可证的保护。如果您在互联网上发现我们作品的任何非法副本，请立即提供位置地址或网站名称，以便我们可以寻求补救措施。

请通过版权@packtpub.com 与我们联系，并提供涉嫌侵权材料的链接。

我们感谢您在保护我们的作者和我们为您提供有价值内容的能力方面的帮助。

## 问答

如果您对本书的任何方面有问题，您可以通过 questions@packtpub.com 与我们联系，我们将尽力解决问题。
