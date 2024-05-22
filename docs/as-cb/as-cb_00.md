# 前言

Android Studio 是开发 Android 应用程序的最佳 IDE，任何想要开发专业 Android 应用程序的人都可以免费使用。

现在有了 Android Studio，我们有了一个稳定和更快的 IDE，它带来了很多很酷的东西，比如 Gradle、更好的重构方法和更好的布局编辑器。如果您曾经使用过 Eclipse，那么您一定会喜欢这个 IDE。

简而言之，Android Studio 真的带回了移动开发的乐趣，在这本书中，我们将看到如何做到这一点。

# 本书涵盖的内容

第一章，欢迎来到 Android Studio，演示了如何配置 Android Studio 和 Genymotion，这是一个非常快速的模拟器。

第二章，具有基于云的后端的应用程序，解释了如何使用 Parse 在很短的时间内开发使用基于云的后端的应用程序。

第三章，Material Design，解释了材料设计的概念以及如何使用 RecycleViews、CardViews 和过渡来实现它。

第四章，Android Wear，涵盖了 Android Wear API 以及如何开发自己的手表表盘或其他在智能手表上运行的应用程序。

第五章，Size Does Matter，演示了如何使用片段和其他资源来帮助您创建能够在手机、平板电脑、平板电脑甚至电视上运行的应用程序。我们将即时连接到 YouTube API，使示例更有趣。

第六章，Capture and Share，是关于使用新的 Camera2 API 捕获和预览图像的深入教程。它还告诉您如何在 Facebook 上分享捕获的图像。

第七章，内容提供程序和观察者，解释了如何从使用内容提供程序来显示和观察持久数据中受益。

第八章，Improving Quality，详细介绍了应用模式、单元测试和代码分析工具。

第九章，Improving Performance，介绍了如何使用设备监视器来优化应用程序的内存管理，以及如何使用手机上的开发者选项来检测过度绘制和其他性能问题。

第十章，Beta Testing Your Apps，指导您完成一些最后步骤，例如使用构建变体（类型和风味）和在 Google Play 商店上进行测试版分发。除此之外，它还涵盖了 Android Marshmallow（6.0）提供的运行时权限与安装权限的不同之处。

# 您需要为本书准备什么

对于这本书，您需要下载并设置 Android Studio 和最新的 SDK。Android Studio 是免费的，适用于 Windows、OSX 和 Linux。

强烈建议至少拥有一部手机、平板电脑或平板电脑，但在第一章，欢迎来到 Android Studio 中，我们将向您介绍 Genymotion，一个非常快速的模拟器，在大多数情况下可以代替真实设备使用。

最后，对于一些示例，您需要拥有 Google 开发者帐户。如果您还没有，请尽快获取一个。毕竟，您需要一个才能将您的应用程序放入 Play 商店。

# 这本书是为谁准备的

这本书适合任何已经熟悉 Java 语法并可能已经开发了一些 Android 应用程序的人，例如使用 Eclipse IDE。

这本书特别解释了使用 Android Studio 进行 Android 开发的概念。为了演示这些概念，提供了真实世界的示例。而且，通过真实世界的应用程序，我指的是连接到后端并与 Google Play 服务或 Facebook 等进行通信的应用程序。

# 部分

在本书中，您会经常看到几个标题（准备工作、如何做、它是如何工作的、还有更多和另请参阅）。

为了清晰地说明如何完成食谱，我们使用以下这些部分：

## 准备工作

本节告诉您在食谱中可以期待什么，并描述如何设置所需的任何软件或任何预备设置。

## 如何做…

本节包含了遵循食谱所需的步骤。

## 它是如何工作的…

本节通常包括对前一节发生的事情的详细解释。

## 还有更多…

本节包括有关食谱的额外信息，以使读者更加了解食谱。

## 另请参阅

本节提供了有关食谱的其他有用信息的有用链接。

# 约定

所有针对 Android Studio 的屏幕截图、快捷方式和其他元素都基于 OSX 上的 Android Studio。

OSX 被使用的主要原因是因为它允许我们在同一台机器上为 Android 和 iOS 开发应用程序。除此之外，选择特定操作系统的原因除了个人（或公司）的偏好之外没有其他原因。

虽然屏幕截图是基于 OSX 上的 Android Studio，但如果您的操作系统是 Windows 或 Linux，您也不难弄清楚事情。

在需要时，还会提及 Windows 的快捷键。

在本书中，您会发现一些区分不同类型信息的文本样式。以下是一些这些样式的示例及其含义的解释。

文本中的代码词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 句柄都会以这种方式显示："我们可以通过使用 include 指令包含其他上下文"。

代码块设置如下：

```kt
public void onSectionAttached(int number) {
    switch (number) {
        case 0:
            mTitle = getString(  
             R.string.title_section_daily_notes);
            break;

        case 1:
            mTitle = getString( 
             R.string.title_section_note_list);
             break;
    }
}
```

**新术语**和**重要单词**以粗体显示。例如，屏幕上看到的菜单或对话框中的单词会以这种方式出现在文本中："单击**下一步**按钮将您移至下一个屏幕"。

### 注

警告或重要提示会以这种方式显示在一个框中。

### 提示

技巧和窍门会以这种方式出现。

# 读者反馈

我们始终欢迎读者的反馈。请告诉我们您对本书的看法——您喜欢或不喜欢的地方。读者的反馈对我们开发您真正受益的标题非常重要。

要向我们发送一般反馈，只需发送电子邮件至 `<feedback@packtpub.com>`，并在消息主题中提及书名。

如果您在某个专业领域有专长，并且有兴趣撰写或为书籍做出贡献，请参阅我们的作者指南 [www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

现在您是 Packt 书籍的自豪所有者，我们有许多事情可以帮助您充分利用您的购买。

## 下载示例代码

您可以从 [`www.packtpub.com`](http://www.packtpub.com) 的帐户中下载您购买的所有 Packt 书籍的示例代码文件。如果您在其他地方购买了本书，您可以访问 [`www.packtpub.com/support`](http://www.packtpub.com/support) 并注册，以便文件直接通过电子邮件发送给您。

## 下载本书的彩色图像

我们还为您提供了一个 PDF 文件，其中包含本书中使用的屏幕截图/图表的彩色图像。彩色图像将帮助您更好地理解输出中的变化。您可以从 [`www.packtpub.com/sites/default/files/downloads/B04299_ColoredImages.pdf`](https://www.packtpub.com/sites/default/files/downloads/B04299_ColoredImages.pdf) 下载此文件。

## 勘误

尽管我们已经尽最大努力确保内容的准确性，但错误确实会发生。如果您在我们的书籍中发现错误，无论是文字还是代码方面的错误，我们将不胜感激地希望您向我们报告。通过这样做，您可以帮助其他读者避免挫折，并帮助我们改进本书的后续版本。如果您发现任何勘误，请访问[`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)，选择您的书籍，点击**勘误提交表格**链接，并输入您的勘误详情。一旦您的勘误经过验证，您的提交将被接受，并且勘误将被上传到我们的网站上，或者添加到该书籍的勘误列表中的**勘误**部分。您可以通过从[`www.packtpub.com/support`](http://www.packtpub.com/support)选择您的书籍来查看任何现有的勘误。

## 盗版

互联网上的版权盗版是所有媒体的持续问题。在 Packt，我们非常重视对我们的版权和许可的保护。如果您在互联网上发现我们作品的任何非法副本，请立即向我们提供位置地址或网站名称，以便我们采取补救措施。

请通过`<copyright@packtpub.com>`与我们联系，并附上涉嫌盗版材料的链接。

我们感谢您帮助保护我们的作者，以及我们为您提供有价值内容的能力。

## 问题

如果您在阅读本书的过程中遇到任何问题，可以通过`<questions@packtpub.com>`与我们联系，我们将尽力解决。
