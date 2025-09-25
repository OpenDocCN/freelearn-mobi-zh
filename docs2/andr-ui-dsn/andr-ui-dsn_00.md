# 前言

你的 UI 是你与用户最直接的沟通方式，但在应用开发中，设计往往被忽视，或者只是“顺便”发生的事情。

为了开发一个出色的应用，你需要认真对待你的 UI 设计。即使你的应用提供了出色的功能，但如果其用户界面笨拙、卡顿、难以导航，或者整体上令人不快，那么没有人会想使用它。

设计 Android 并不是一件容易的事情。创建一个在不同设备上看起来都很棒的应用，这些设备具有不同的硬件、软件和屏幕组合，是一项了不起的成就。但如果你在设计过程中投入一些思考，你可以创建一个在整个 Android 生态系统中提供良好体验的 UI。

本书将帮助你成为一个有设计思维的开发者。在 10 章的内容中，你将学习如何设计、精炼、测试和开发一个有效、吸引人的 UI，人们会*喜爱*使用，并且无论设备的具体情况如何，都能提供最佳的用户体验。

从布局和视图的基础知识到创建响应式、多面板布局，再到通过绘制你的屏幕，使用 Android Studio 的工具来审查你的视图层次结构并查找内存泄漏，本书涵盖了创建完美 Android UI 所需的一切。

# 本书涵盖内容

*第一章，介绍 Android UI*，解释为什么设计和开发一个有效的 UI 对你的应用成功至关重要。

*第二章，一个有效的 UI 包含哪些内容？*，教你如何掌握每个 Android UI 的构建块：布局和视图。学习如何创建和设计常见的 UI 组件，包括文本、图像和按钮。

*第三章，扩展你的 UI – 片段、资源以及收集用户输入*，帮助你使用数组、维度、状态列表和 9-patch 图像将你的 UI 提升到下一个层次。使用片段，你将学习如何创建一个灵活的 UI，它能根据用户的特定屏幕尺寸做出反应。

*第四章，开始使用 Material Design*，教你如何掌握谷歌的新设计原则，并通过一个感觉像 Android 操作系统的无缝扩展的 UI 来取悦你的用户。

*第五章，将你的创意转化为详细草图*，通过介绍路线图、流程图、屏幕列表和屏幕图到你的 Android 应用“待办事项”列表中，帮助你成为一个有设计思维的开发者。

*第六章，将你的草图转换为线框*，展示了如何使用纸原型和线框将前一章中的高级计划转换为详细的屏幕设计。

*第七章，构建原型*，对你的计划进行测试！到本章结束时，你将创建一个完整的数字原型。

*第八章，扩大受众范围 – 支持多设备*，教你如何通过支持广泛的硬件、软件、屏幕尺寸、屏幕密度、方向、Android 平台版本以及甚至不同语言的 APP 来吸引更广泛的受众。

*第九章，优化你的 UI*，展示了如何创建一个流畅且响应迅速的 UI，人们会喜欢使用。如果你的应用运行缓慢、容易崩溃、消耗数据内存，或者耗尽用户的电池，那么没有人会想使用它！

*第十章*，*最佳实践和安全保护你的应用程序*，将指导你完成 UI 的最终润色，包括使用即将发布的 Android N 版本的通知。

# 阅读本书所需的条件

你将需要 Android SDK 和 Android Studio（首选）或 Eclipse。

# 适合阅读本书的读者

如果你是对为应用程序构建令人惊叹的 Android UI 有浓厚兴趣的 Java 开发者，目的是保留客户并为他们创造美好的体验，那么这本书就是为你准备的。假设你具备良好的 XML 知识并对 Android 开发有一定了解。

# 习惯用法

在本书中，你会发现许多不同的文本样式，用于区分不同类型的信息。以下是一些这些样式的示例及其含义的解释。

文本中的代码词汇、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 昵称将如下显示：“我们可以通过使用`include`指令来包含其他上下文。”

代码块将如下设置：

```java
<LinearLayout

   android:orientation="vertical"
   android:layout_width="match_parent"
   android:layout_height="match_parent">
```

当我们希望引起你对代码块中特定部分的注意时，相关的行或项目将以粗体显示：

```java
<LinearLayout 

   android:orientation="vertical"
   android:layout_width="match_parent"
   android:layout_height="match_parent">
```

任何命令行输入或输出都应如下编写：

```java
cd /Users/username/Downloads/android/sdk/platform-tools

```

**新术语**和**重要词汇**将以粗体显示。屏幕上显示的词汇，例如在菜单或对话框中，将以如下方式显示：“点击**下一步**按钮将您带到下一屏幕。”

### 注意

警告或重要注意事项将以如下框中显示。

### 小贴士

小技巧和技巧将以如下方式显示。

对于本书，我们概述了 Mac OS X 平台的快捷键，如果您使用的是 Windows 版本，您可以在 WebStorm 帮助页面[`www.jetbrains.com/webstorm/help/keyboard-shortcuts-by-category.html`](https://www.jetbrains.com/webstorm/help/keyboard-shortcuts-by-category.html)上找到相关快捷键。

# 读者反馈

我们读者的反馈总是受欢迎的。让我们知道您对这本书的看法——您喜欢或不喜欢什么。读者反馈对我们很重要，因为它帮助我们开发出您真正能从中获得最大收益的标题。

要发送一般反馈，请直接发送电子邮件至 feedback@packtpub.com，并在邮件主题中提及本书的标题。

如果您在某个主题上具有专业知识，并且您有兴趣撰写或为本书做出贡献，请参阅我们的作者指南[www.packtpub.com/authors](https://www.packtpub.com/authors)。

# 客户支持

现在您已经是 Packt 图书的骄傲拥有者，我们有一些事情可以帮助您从您的购买中获得最大收益。

## 下载示例代码

您可以从[`www.packtpub.com`](http://www.packtpub.com)的账户下载本书的示例代码文件。如果您在其他地方购买了这本书，您可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)并注册，以便将文件直接通过电子邮件发送给您。

您可以通过以下步骤下载代码文件：

1.  使用您的电子邮件地址和密码登录或注册我们的网站。

1.  将鼠标指针悬停在顶部的**支持**选项卡上。

1.  点击**代码下载与勘误**。

1.  在**搜索**框中输入书的名称。

1.  选择您想要下载代码文件的书籍。

1.  从下拉菜单中选择您购买本书的地方。

1.  点击**代码下载**。

文件下载完成后，请确保您使用最新版本的软件解压缩或提取文件夹：

+   适用于 Windows 的 WinRAR / 7-Zip

+   适用于 Mac 的 Zipeg / iZip / UnRarX

+   适用于 Linux 的 7-Zip / PeaZip

本书代码包也托管在 GitHub 上，网址为[`github.com/PacktPublishing/Android-UI-Design`](https://github.com/PacktPublishing/Android-UI-Design)。我们还有其他来自我们丰富图书和视频目录的代码包可供在[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)找到。查看它们吧！

## 下载本书的颜色图像

我们还为您提供了一个包含本书中使用的截图/图表颜色图像的 PDF 文件。颜色图像将帮助您更好地理解输出的变化。您可以从[`www.packtpub.com/sites/default/files/downloads/AndroidUIDesign_ColoredImages.pdf`](http://www.packtpub.com/sites/default/files/downloads/AndroidUIDesign_ColoredImages.pdf)下载此文件。

## 勘误

尽管我们已经尽一切努力确保我们内容的准确性，但错误仍然会发生。如果您在我们的某本书中发现错误——可能是文本或代码中的错误——如果您能向我们报告这一点，我们将不胜感激。通过这样做，您可以节省其他读者的挫败感，并帮助我们改进本书的后续版本。如果您发现任何勘误，请通过访问 [`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)，选择您的书籍，点击**勘误提交表单**链接，并输入您的勘误详情来报告它们。一旦您的勘误得到验证，您的提交将被接受，勘误将被上传到我们的网站或添加到该标题的勘误部分下的现有勘误列表中。

要查看之前提交的勘误表，请访问 [`www.packtpub.com/books/content/support`](https://www.packtpub.com/books/content/support)，并在搜索字段中输入书籍名称。所需信息将在**勘误**部分显示。

## 盗版

互联网上对版权材料的盗版是一个跨所有媒体的持续问题。在 Packt，我们非常重视我们版权和许可证的保护。如果您在互联网上发现任何形式的我们作品的非法副本，请立即提供位置地址或网站名称，以便我们可以寻求补救措施。

请通过 copyright@packtpub.com 联系我们，并提供涉嫌盗版材料的链接。

我们感谢您在保护我们的作者和我们为您提供有价值内容的能力方面的帮助。

## 询问

如果您对本书的任何方面有问题，您可以通过 questions@packtpub.com 联系我们，我们将尽力解决问题。
