# 前言

无论您在 Android 设计上投入多少时间，或者编程时多么小心，错误是不可避免的，bug 将会出现。本书将帮助您最小化这些错误对您的 Android 项目的影响，并提高您的开发效率。它将向您展示容易避免的问题，以帮助您快速进入测试阶段。

《*Android 应用程序测试指南*》是第一本也是唯一一本提供对最常用技术、框架和工具的实用介绍的书籍，旨在提高您的 Android 应用程序开发。清晰的、分步的指导说明如何为您的应用程序编写测试，并使用各种方法确保质量控制。

作者将应用测试技术应用于实际项目的经验使他能够分享创建专业 Android 应用程序的见解。

本书首先介绍了测试驱动开发（Test Driven Development），这是软件开发过程中的一个敏捷组件，也是一种早期解决 bug 的技术。从应用于示例项目的最基本单元测试到更复杂的性能测试，本书以食谱为基础的方法，详细描述了在 Android 测试领域最广泛使用的技巧。

作者在其职业生涯中在多个开发项目上拥有丰富的经验。所有这些研究和知识都有助于创建一本对任何在 Android 测试领域导航的开发者都非常有用的书籍。

# 本书涵盖内容

第一章，*测试入门*介绍了不同类型的测试及其在软件开发项目中的适用性，特别是 Android。

第二章，*Android 上的测试*涵盖了在 Android 平台上的测试、单元测试和 JUnit、创建 Android 测试项目以及运行测试。

第三章，*Android SDK 的构建块*开始深入挖掘，以识别可用于创建测试的构建块。它涵盖了断言（Assertions）、TouchUtils（用于测试用户界面）、模拟对象（Mock objects）、Instrumentation 和具有 UML 图的 TestCase 类层次结构。

第四章，*测试驱动开发*介绍了测试驱动开发的学科。它从一般复习开始，然后转向与 Android 平台紧密相关的概念和技术。这是一章代码密集的章节。

第五章，*Android 测试环境*提供了运行测试的不同条件。它从创建 Android 虚拟设备（AVD）开始，为测试中的应用提供不同的条件和配置，然后使用可用的选项运行测试。最后，它介绍了使用猴子作为生成用于测试的模拟事件的方法。

第六章，*行为驱动开发*介绍了行为驱动开发以及一些概念，例如使用通用词汇来表达测试以及将业务参与者纳入软件开发项目。

第七章，*测试食谱*提供了在应用之前描述的学科和技术时可能遇到的不同常见情况的实际示例。这些示例以食谱风格呈现，以便您可以根据自己的项目进行修改和使用。这些食谱涵盖了 Android 单元测试、活动、应用程序、数据库和内容提供者、本地和远程服务、用户界面、异常、解析器和内存泄漏。

第八章，*持续集成*介绍了这种旨在通过持续应用集成和测试来提高软件质量和减少集成更改所需时间的敏捷技术。

第九章，*性能测试*介绍了一系列与基准测试和配置文件相关的概念，从传统的日志语句方法到创建 Android 性能测试和使用剖析工具。本章还介绍了 Caliper 来创建微基准测试。

第十章，*替代测试策略*涵盖了从源代码构建 Android、使用 EMMA、Robotium 进行代码覆盖率测试、在主机上进行测试以及使用 Robolectric。

# 您需要这本书的什么

首先，您需要实际的 Android 开发经验，因为我们不会涵盖基础知识。假设您已经有一些 Android 应用程序，或者至少您熟悉 Android 开发指南（[`developer.android.com/guide/index.html`](http://developer.android.com/guide/index.html)）中广泛描述的主题。此外，跟随一些示例代码项目（[`developer.android.com/resources/browser.html?tag=sample`](http://developer.android.com/resources/browser.html?tag=sample)）也非常有帮助，可能从 API 演示开始，然后转向其他更复杂的话题。这样，您将最大限度地利用这本书。

为了能够跟随不同章节中的示例，您需要安装一套通用的软件和工具，以及每个章节中特别描述的几个其他组件，包括它们的相应下载位置。

所有示例均基于：

+   Ubuntu 10.04.2 LTS (lucid) 64 位，完全更新

+   Java SE 版本 "1.6.0_24" (构建 1.6.0_24-b07)

+   Android SDK 工具，修订版 11

+   Android SDK 平台工具，修订版 4

+   SDK 平台 Android 2.3.1，API 9，修订版 2

+   Android 兼容性包，修订版 2

+   为 Java 开发者提供的 Eclipse IDE，版本：Helios Service Release 1 (3.6.1)

+   Android 开发工具包，版本：10.0.1.v201103111512-110841

+   Dalvik 调试监控服务，版本：10.0.1.v201103111512-110841

+   Apache Ant 版本 1.8.0，编译于 2010 年 4 月 9 日

+   Git 版本 1.7.0.4

+   Subversion 版本 1.6.6 (r40053)，编译于 2011 年 3 月 23 日，13:08:34

书中展示的 UML 图形是使用 BOUML 版本 4.21 创建的。

截图使用 Shutter 0.86.3 拍摄和编辑。

手稿使用 OpenOffice.org 3.2.1 进行编辑。

# 本书面向对象

如果您是希望测试您的应用程序或优化应用程序开发过程的 Android 开发者，那么这本书适合您。不需要在应用程序测试方面有先前的经验。

# 惯例

在本书中，您将找到许多不同风格的文本，以区分不同类型的信息。以下是一些这些样式的示例及其含义的解释。

文本中的代码单词按以下方式显示：“要调用`am`命令，我们将使用`adb shell`命令”。

代码块设置如下：

```java
@VeryImportantTest
public void testOtherStuff() {
fail("Not implemented yet");
}

```

当我们希望您注意代码块中的特定部分时，相关的行或项目将以粗体显示：

```java
public class MyFirstProjectTests extends TestCase { public MyFirstProjectTests() {
this("MyFirstProjectTests");
}

```

任何命令行输入都按以下方式编写：

```java
$ adb shell am instrument -w -e class com.example.aatg.myfirstproject.test.MyFirstProjectTests com.example.aatg.myfirstproject.test/android.test.InstrumentationTestRunner

```

任何命令行输出都按以下方式编写：

**08-10 00:26:11.820: ERROR/AndroidRuntime(510): FATAL EXCEPTION: main**

**08-10 00:26:11.820: ERROR/AndroidRuntime(510): java.lang.IllegalAccessError: Class ref in pre-verified class resolved to unexpected implementation**

**新术语**和**重要词汇**以粗体显示。您在屏幕上看到的单词，例如在菜单或对话框中，在文本中显示如下：“选择测试项目，然后**运行 | 运行配置**”。

### 注意

警告或重要说明以如下框的形式出现。

### 小贴士

小技巧和技巧以如下形式出现。

# 读者反馈

我们始终欢迎读者的反馈。请告诉我们您对这本书的看法——您喜欢什么或可能不喜欢什么。读者反馈对我们开发您真正能从中获得最大收益的标题非常重要。

如要向我们发送一般反馈，请简单地将电子邮件发送到 `<feedback@packtpub.com>`，并在邮件主题中提及书名。

如果您需要一本书并且希望我们出版，请通过 [www.packtpub.com](http://www.packtpub.com) 上的**建议标题**表单或发送电子邮件到 `<suggest@packtpub.com>》。

如果您在某个主题上具有专业知识，并且您有兴趣撰写或为书籍做出贡献，请参阅我们的作者指南 [www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

现在您是 Packt 书籍的骄傲拥有者，我们有一些事情可以帮助您从您的购买中获得最大收益。

## 下载示例代码

您可以从您在 [`www.PacktPub.com`](http://www.PacktPub.com) 的账户下载您购买的所有 Packt 书籍的示例代码文件。如果您在其他地方购买了这本书，您可以访问 [`www.PacktPub.com/support`](http://www.PacktPub.com/support) 并注册，以便将文件直接通过电子邮件发送给您。

## 错误清单

尽管我们已经尽一切努力确保我们内容的准确性，但错误仍然可能发生。如果您在我们的某本书中发现错误——可能是文本或代码中的错误——如果您能向我们报告这一点，我们将不胜感激。通过这样做，您可以节省其他读者的挫败感，并帮助我们改进本书的后续版本。如果您发现任何错误清单，请通过访问 [`www.packtpub.com/support`](http://www.packtpub.com/support)，选择您的书籍，点击**错误提交表单**链接，并输入您的错误详细信息来报告它们。一旦您的错误得到验证，您的提交将被接受，错误将被上传到我们的网站，或添加到该标题的错误清单部分。任何现有的错误清单都可以通过从 [`www.packtpub.com/support`](http://www.packtpub.com/support) 选择您的标题来查看。

## 盗版

在互联网上侵犯版权材料是一个跨所有媒体持续存在的问题。在 Packt，我们非常重视我们版权和许可证的保护。如果您在互联网上发现任何我们作品的非法副本，无论形式如何，请立即提供位置地址或网站名称，以便我们可以寻求补救措施。

请通过 `<copyright@packtpub.com>` 与我们联系，并提供疑似盗版材料的链接。

我们感谢您在保护我们作者以及为我们带来有价值内容方面的帮助。

## 询问

如果您在本书的任何方面遇到问题，可以通过 `<questions@packtpub.com>` 联系我们，我们将尽力解决。
