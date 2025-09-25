# 前言

Android 引发了我们这个时代最伟大的革命之一。在智能手机、电视、平板电脑、手表、嵌入式板上都有其存在，可以说是无处不在。其开源特性为公司、专家用户和黑客提供了学习、改进和定制系统、创建最流行移动操作系统的定制版本的机会。

这本书是一次从 Android 项目的起源到未来的旅程，沿途经过构建自定义 Android 系统所需的所有阶段，无论是从源代码还是从二进制镜像开始。

## 本书涵盖的内容

第一章，“理解架构*”，解释了 Android 硬件和软件架构、Android 兼容性定义文档、Android 兼容性测试套件和 Android 运行时。

第二章，“获取源代码 – 结构和哲学*”，解释了 Android 开源项目。

第三章，“设置和构建 – 模拟器方式*”，教你如何设置构建环境并构建 Android 模拟器的系统镜像。

第四章，“转向现实世界硬件*”，告诉你如何构建一个真实设备以及如何刷系统镜像。

第五章，“定制内核和引导序列*”，深入内核和引导序列的定制，以便打造完美的系统。

第六章，“制作”您的第一个 ROM*，讨论了自定义恢复镜像、root 权限和 Android Kitchen。

第七章，“定制您的个人 Android 系统*”，讨论了黑客攻击 Android 框架、添加应用程序和优化系统。

第八章，“超越智能手机*”，讨论了下一步是什么，一旦你离开智能手机世界，Android 的可能性是什么。

*更多关于 Android N 编程*：在本章中，你将找到有关 Android N 编程的更多信息，请参阅[`www.packtpub.com/sites/default/files/downloads/MoreaboutAndroidNProgramming.pdf`](https://www.packtpub.com/sites/default/files/downloads/MoreaboutAndroidNProgramming.pdf)。

## 你需要这本书

旅行所需的一切就是一台个人电脑，Ubuntu Linux 或 OS X 都可以，一个互联网连接，以及你的热情！

## 这本书面向的对象

如果你是一名程序员或嵌入式系统黑客，想要定制、构建和部署自己的 Android 版本，那么这本书绝对是为你准备的。

## 术语约定

在这本书中，你会发现许多文本样式，用于区分不同类型的信息。以下是一些这些样式的示例及其含义的解释。

文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 昵称如下所示：“我们可以通过使用`include`指令来包含其他上下文。”

代码块设置如下：

```java
LOCAL_SRC_FILES:=\
        netcat.c \
        atomicio.c
```

任何命令行输入或输出都写作如下：

```java
$ git add art_new_feature
$ git commit -m "Add new awesome feature to ART"

```

**新术语**和**重要词汇**以粗体显示。屏幕上显示的单词，例如在菜单或对话框中，在文本中显示如下：“点击**下一个**按钮将您带到下一屏幕。”

#### 注意

警告或重要注意事项以这样的框显示。

#### 小贴士

小贴士和技巧如下显示。

## 读者反馈

我们始终欢迎读者的反馈。告诉我们您对这本书的看法——您喜欢或不喜欢的地方。读者反馈对我们很重要，因为它帮助我们开发出您真正能从中获得最大收益的标题。

要发送一般反馈，请简单地发送电子邮件至`<feedback@packtpub.com>`，并在邮件主题中提及书名。

如果您在某个主题上具有专业知识，并且您对撰写或为书籍做出贡献感兴趣，请参阅我们的作者指南，网址为[www.packtpub.com/authors](http://www.packtpub.com/authors)。

## 客户支持

现在您已经是 Packt 图书的骄傲拥有者，我们有一些事情可以帮助您从您的购买中获得最大收益。

### 下载示例代码

您可以从您的账户下载此书的示例代码文件。[`www.packtpub.com`](http://www.packtpub.com)。如果您在其他地方购买了这本书，您可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)并注册，以便将文件直接通过电子邮件发送给您。

您可以通过以下步骤下载代码文件：

1.  使用您的电子邮件地址和密码登录或注册我们的网站。

1.  将鼠标指针悬停在顶部的**支持**标签上。

1.  点击**代码下载与勘误**。

1.  在**搜索**框中输入书名。

1.  选择您想要下载代码文件的书。

1.  从下拉菜单中选择您购买此书的来源。

1.  点击**代码下载**。

您还可以通过点击 Packt Publishing 网站上的书页上的**代码文件**按钮下载代码文件。您可以通过在**搜索**框中输入书名来访问此页面。请注意，您需要登录到您的 Packt 账户。

下载文件后，请确保您使用最新版本解压缩或提取文件夹。

+   Windows 上的 WinRAR / 7-Zip

+   Mac 上的 Zipeg / iZip / UnRarX

+   Linux 上的 7-Zip / PeaZip

书籍的代码包也托管在 GitHub 上，网址为[`github.com/PacktPublishing/Learning-Embedded-Android-N-Programming`](https://github.com/PacktPublishing/Learning-Embedded-Android-N-Programming)。我们还有其他来自我们丰富图书和视频目录的代码包可供在[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)找到。查看它们吧！

### 勘误

尽管我们已经尽一切努力确保内容的准确性，但错误仍然会发生。如果您在我们的某本书中发现错误——可能是文本或代码中的错误——如果您能向我们报告这一点，我们将不胜感激。通过这样做，您可以节省其他读者的挫败感，并帮助我们改进本书的后续版本。如果您发现任何勘误，请通过访问[`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)，选择您的书籍，点击**勘误提交表单**链接，并输入您的勘误详情。一旦您的勘误得到验证，您的提交将被接受，勘误将被上传到我们的网站或添加到该标题的勘误部分下的现有勘误列表中。

要查看之前提交的勘误表，请访问[`www.packtpub.com/books/content/support`](https://www.packtpub.com/books/content/support)，并在搜索字段中输入书籍名称。所需信息将出现在**勘误**部分。

### 侵权

在互联网上侵犯版权材料是一个跨所有媒体的持续问题。在 Packt，我们非常重视保护我们的版权和许可证。如果您在互联网上发现任何形式的我们作品的非法副本，请立即提供位置地址或网站名称，以便我们可以寻求补救措施。

请通过`<copyright@packtpub.com>`与我们联系，并提供涉嫌侵权材料的链接。

我们感谢您在保护我们的作者和提供有价值内容的能力方面的帮助。

### 问题

如果您对本书的任何方面有问题，您可以通过`<questions@packtpub.com>`与我们联系，我们将尽力解决问题。
