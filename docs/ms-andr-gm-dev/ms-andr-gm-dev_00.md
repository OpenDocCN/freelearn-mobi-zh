# 前言

这本书是学习高级 Android 应用开发的实用指南。本书帮助掌握 Android 的核心概念，并在实际项目中快速应用知识。在整本书中，创建了一个应用，并在每一章中不断进化，以便读者可以轻松地跟随并吸收概念。

本书分为十二章。前三章专注于应用的设计，解释了设计的基本概念以及 Android 中使用的编程模式。接下来的几章旨在改进应用，访问服务器端以下载要在应用中显示的信息。一旦应用功能完成，它将使用 Material Design 组件和其他第三方库进行改进。

在完成之前，应用中添加了额外的服务，如位置服务、分析、崩溃报告和盈利化。最后，导出应用，解释不同的构建类型和证书，并将其上传到 Play 商店，准备进行分发。

# 本书涵盖的内容

第一章, *入门*, 介绍了 Android 6 Marshmallow 的基础知识和 Material Design 的重要概念。我们将设置开始开发所需的工具，并且可选地安装一个比 Android 默认模拟器更快的超快模拟器，这将帮助我们在书中测试我们的应用。

第二章, *设计我们的应用*, 介绍了创建应用的第一步——设计导航——以及不同的导航模式。我们将应用带有滑动屏幕的标签页模式，解释并使用 Fragments，这是 Android 应用开发的一个关键组件。

第三章, *从云端创建和访问内容*, 涵盖了在我们的应用中显示互联网信息所需的一切。这些信息可以在外部服务器或 API 上。我们将使用 Parse 创建自己的服务器，并使用 Volley 和 OKHttp 进行高级网络请求来访问它，处理信息并使用 Gson 将其转换为可用的对象。

第四章, *并发和软件设计模式*, 讨论了 Android 中的并发性和处理它的不同机制，如 AsyncTask、服务、加载器等。本章的下半部分讨论了在 Android 中最常见的编程模式。

第五章, *列表和网格*, 讨论了列表和网格，从 ListViews 开始。它解释了这一组件如何在 RecyclerView 中演变，并作为一个示例，展示了如何创建带有不同类型元素的列表。

第六章，*CardView 和材料设计*，专注于从用户界面角度改进应用，并引入材料设计，解释并实现如 CardView、Toolbar 和 CoordinatorLayout 等功能。

第七章，*图像处理和内存管理*，主要讨论了如何在我们应用中显示从互联网上下载的图片，使用不同的机制，如 Volley 或 Picasso。它还涵盖了不同类型的图像，如矢量可绘制图像和 Nine patch。最后，它讨论了内存管理以及预防、检测和定位内存泄漏。

第八章，*数据库和加载器*，主要解释了安卓中数据库的工作原理，内容提供者是什么，以及如何使用 CursorLoaders 让数据库直接与视图通信。

第九章，*推送通知和分析*，讨论了如何使用 Google Cloud Messaging 和 Parse 实现推送通知。章节的下半部分讨论了分析，这对于理解用户如何与我们的应用互动，捕获错误报告以及保持应用无虫至关重要。

第十章，*位置服务*，通过在应用中实现一个示例来介绍 MapView，从开发者控制台初始设置到应用中最终显示位置标记的地图视图。

第十一章，*在安卓上的调试和测试*，主要讨论测试。它涵盖了单元测试、集成测试和用户界面测试。还讨论了使用市场上不同的工具和最佳实践，通过自动化测试开发可维护的应用程序。

第十二章，*货币化、构建过程和发布*，展示了如何实现应用的货币化，并解释了广告货币化的关键概念。它展示了如何导出具有不同构建类型的应用，并最终如何在 Google Play 商店上传和推广此应用。

# 阅读本书所需的条件

你的系统必须具备以下软件才能执行本书中提到的代码：

+   安卓工作室 1.0 或更高版本

+   Java 1.7 或更高版本

+   安卓 4.0 或更高版本

# 本书的目标读者

如果你是一名有 Gradle 经验的 Java 或项目开发者，并希望成为专家，那么这本书就是为你而写的。对 Gradle 的基本了解是必不可少的。

# 约定

在这本书中，你会发现多种文本样式，用于区分不同类型的信息。以下是一些样式示例及其含义的解释。

文本中的代码字、数据库表名、文件夹名、文件名、文件扩展名、路径名和虚拟 URL 将如下显示："我们可以通过使用`include`指令包含其他上下文。"

代码块设置如下：

```java
<uses-permission android:name="android.permission.INTERNET" /> <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" /> <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" /> 
```

当我们需要引导您关注代码块中的特定部分时，相关的行或项目会以粗体显示：

```java
<uses-permission android:name="android.permission.INTERNET" /> <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" /> <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" /> 
```

**新术语**和**重要词汇**以粗体显示。您在屏幕上看到的词，例如菜单或对话框中的，会像这样在文本中显示："点击**下一步**按钮，将进入下一个屏幕。"

### 注意

警告或重要提示会以这样的框显示。

### 提示

技巧和诀窍会像这样显示。

# 读者反馈

我们始终欢迎读者的反馈。请告诉我们您对这本书的看法——您喜欢或不喜欢的地方。读者的反馈对我们很重要，因为它能帮助我们开发出您真正能从中获得最大收益的图书。

要向我们发送一般反馈，只需将电子邮件发送至`<feedback@packtpub.com>`，并在邮件的主题中提及书籍的标题。

如果您在某个主题上有专业知识，并且有兴趣撰写或参与编写书籍，请查看我们的作者指南：[www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

既然您已经拥有了 Packt 的一本书，我们有许多方法可以帮助您从购买中获得最大收益。

## 下载示例代码

您可以从您的账户[`www.packtpub.com`](http://www.packtpub.com)下载所有您购买的 Packt Publishing 书籍的示例代码文件。如果您在别处购买了这本书，您可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)并注册，以便直接将文件通过电子邮件发送给您。

## 下载本书的彩色图片

我们还为您提供了一个 PDF 文件，其中包含本书中使用的屏幕截图/图表的彩色图片。彩色图片将帮助您更好地理解输出的变化。您可以从[`www.packtpub.com/sites/default/files/downloads/4221OS_ColorImages.pdf`](http://www.packtpub.com/sites/default/files/downloads/4221OS_ColorImages.pdf)下载此文件。

## 勘误

尽管我们已经竭尽全力确保内容的准确性，但错误仍然在所难免。如果您在我们的书中发现了一个错误——可能是文本或代码中的错误——如果您能向我们报告，我们将不胜感激。这样做，您可以避免其他读者的困扰，并帮助我们改进本书后续版本。如果您发现任何勘误信息，请通过访问[`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)，选择您的书籍，点击**Errata Submission Form**链接，并输入您的勘误详情。一旦您的勘误信息被核实，您的提交将被接受，并且勘误信息将被上传到我们的网站或添加到该标题下现有勘误列表中。

要查看之前提交的勘误信息，请访问[`www.packtpub.com/books/content/support`](https://www.packtpub.com/books/content/support)，并在搜索字段中输入书名。所需信息将显示在**勘误**部分下。

## 盗版

网络上对版权材料的盗版行为是所有媒体持续面临的问题。在 Packt，我们非常重视对我们版权和许可的保护。如果您在互联网上以任何形式遇到我们作品的非法副本，请立即提供其位置地址或网站名称，以便我们可以寻求补救措施。

如果您发现疑似盗版材料，请通过`<copyright@packtpub.com>`与我们联系，并提供链接。

我们感谢您帮助保护我们的作者以及我们为您带来有价值内容的能力。

## 问题

如果您对这本书的任何方面有问题，可以通过`<questions@packtpub.com>`联系我们，我们将尽力解决问题。
