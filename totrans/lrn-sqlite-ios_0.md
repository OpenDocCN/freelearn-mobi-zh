# 前言

SQLite仍然是智能手机和平板电脑上移动应用程序广泛使用的数据库。对于有SQL经验的人来说，理解和学习它所能提供的内容以及它可以用作的应用程序将更容易。SQLite于2000年发布，已发展成为移动设备开发的常用数据库。

D. Richard Hipp先生在一家名为通用动力公司的公司期间，在一艘战舰上开发了它。最初用作存储，后来使用B树实现进行开发，这增强了它并使其能够存储行和事务。

本书为您提供了学习SQLite（移动数据库）元素的机会，它与MAC操作系统、Xcode以及Apple应用程序和PhoneGap的开发者IDE的交互；它使HTML5成为可能。它概述了与SQLite一起工作的简便性。

# 本书涵盖的内容

[第1章](ch01.html "第1章。SQL和SQLite简介"), *SQL和SQLite简介*，为您介绍了结构化查询语言（SQL）和移动数据库SQLite的背景。

[第2章](ch02.html "第2章。数据库设计概念"), *数据库设计概念*，讨论了SQLite中的数据库概念。

[第3章](ch03.html "第3章。数据库管理"), *数据库管理*，为您介绍了管理SQLite数据库的方法，并使您了解这个关系型数据库的不同组件。

[第4章](ch04.html "第4章。SQL基础"), *SQL基础*，本章讨论了SQL的基础知识。它将概述SQL的主要可能性以及如何在SQLite上正确使用它。这是理解SQL的使用、其局限性和优势所必需的。

[第5章](ch05.html "第5章。公开C API"), *公开C API*，处理C API及其扩展应用使用的方法，以及如何使用代码生成所需的应用程序。

[第6章](ch06.html "第6章。使用Swift与iOS和SQLite"), *使用Swift与iOS和SQLite*，探讨了如何使用苹果公司的新编程语言Swift与SQLite一起使用。

[第7章](ch07.html "第7章。使用PhoneGap和HTML5进行iOS开发"), *使用PhoneGap和HTML5进行iOS开发*，探讨了如何使用Xcode与PhoneGap一起集成和编译源代码，包括HTML5。

[第8章](ch08.html "第8章。SQLite的更多功能和进步"), *SQLite的更多功能和进步*，讨论了SQLite近年来如何变化，它如何发展到集成到各种现有技术中，以及其简单易用的公式如何确保它在其他人中的流行。

# 您需要这本书的内容

在这本书中，所需的软件如下：

+   Mac操作系统：

    +   OS X 10.9或更高版本

+   软件：

    +   支持Swift的Xcode IDE软件开发环境（版本7.0-7.1.1+）

    +   来自[PhoneGap.com](http://PhoneGap.com)的最新PhoneGap版本

    +   来自[https://nodejs.org/en/](https://nodejs.org/en/)的最新版Node.js

# 本书面向对象

本书旨在帮助那些想要了解最强大、最灵活的移动数据库，用于以正确的方式在Swift或Objective-C中开发应用程序的人。如果你是Objective-C的专家程序员或对这个平台是新手，你将快速学习，掌握真实应用程序的代码，以有效地使用Swift。

# 规范

在本书中，你会发现许多文本样式，用于区分不同类型的信息。以下是一些这些样式的示例及其含义的解释。

文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟URL、用户输入和Twitter昵称显示如下：“这种语言有多种语句，但大多数人会识别`INSERT`、`SELECT`、`UPDATE`和`DELETE`语句。”

代码块设置如下：

[PRE0]

任何命令行输入或输出都如下所示：

[PRE1]

**新术语**和**重要单词**以粗体显示。屏幕上看到的单词，例如在菜单或对话框中，在文本中如下所示：“然后在页面底部，在**链接框架和库**中点击**+**，将出现一个模态窗口。”

### 注意

警告或重要注意事项会出现在这样的框中。

### 提示

小技巧和技巧会像这样显示。

# 读者反馈

我们读者的反馈总是受欢迎的。告诉我们你对这本书的看法——你喜欢什么或不喜欢什么。读者反馈对我们很重要，因为它帮助我们开发出你真正能从中获得最大价值的标题。

要发送一般反馈，只需发送电子邮件至`<[feedback@packtpub.com](mailto:feedback@packtpub.com)>`，并在邮件主题中提及书籍标题。

如果你在一个领域有专业知识，并且你对撰写或为书籍做出贡献感兴趣，请参阅我们的作者指南[www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

现在你已经是Packt图书的骄傲所有者，我们有一些事情可以帮助你从购买中获得最大价值。

## 下载示例代码

你可以从你的账户中下载本书的示例代码文件。[http://www.packtpub.com](http://www.packtpub.com)。如果你在其他地方购买了这本书，你可以访问[http://www.packtpub.com/support](http://www.packtpub.com/support)并注册，以便将文件直接通过电子邮件发送给你。

你可以通过以下步骤下载代码文件：

1.  使用你的电子邮件地址和密码登录或注册我们的网站。

1.  将鼠标指针悬停在顶部的**支持**选项卡上。

1.  点击**代码下载与勘误表**。

1.  在**搜索**框中输入书籍名称。

1.  选择你想要下载代码文件的书籍。

1.  从下拉菜单中选择你购买这本书的地方。

1.  点击**代码下载**。

文件下载完成后，请确保使用最新版本的以下软件解压缩或提取文件夹：

+   WinRAR / 7-Zip for Windows

+   Zipeg / iZip / UnRarX for Mac

+   7-Zip / PeaZip for Linux

## 下载本书的彩色图像

我们还为您提供了一个包含本书中使用的截图/图表彩色图像的PDF文件。这些彩色图像将帮助您更好地理解输出中的变化。您可以从[http://www.packtpub.com/sites/default/files/downloads/Learning_SQLite_for_iOS_ColoredImages.pdf](http://www.packtpub.com/sites/default/files/downloads/Learning_SQLite_for_iOS_ColoredImages.pdf)下载此文件。

## 勘误

尽管我们已经尽一切努力确保内容的准确性，但错误仍然可能发生。如果您在我们的书中发现错误——可能是文本或代码中的错误——如果您能向我们报告这一点，我们将不胜感激。通过这样做，您可以节省其他读者的挫败感，并帮助我们改进本书的后续版本。如果您发现任何勘误，请通过访问[http://www.packtpub.com/submit-errata](http://www.packtpub.com/submit-errata)，选择您的书籍，点击**勘误提交表单**链接，并输入您的勘误详情来报告它们。一旦您的勘误得到验证，您的提交将被接受，勘误将被上传到我们的网站或添加到该标题的勘误部分下的现有勘误列表中。

要查看之前提交的勘误，请访问[https://www.packtpub.com/books/content/support](https://www.packtpub.com/books/content/support)，并在搜索字段中输入书籍名称。所需信息将出现在**勘误**部分下。

## 盗版

在互联网上，版权材料的盗版是一个跨所有媒体的持续问题。在Packt，我们非常重视保护我们的版权和许可证。如果您在互联网上发现任何形式的我们作品的非法副本，请立即提供位置地址或网站名称，以便我们可以寻求补救措施。

请通过发送链接到疑似盗版材料至`<[copyright@packtpub.com](mailto:copyright@packtpub.com)>`与我们联系。

我们感谢您在保护我们的作者和我们为您提供有价值内容的能力方面的帮助。

## 问题

如果您在这本书的任何方面遇到问题，您可以通过发送电子邮件至`<[questions@packtpub.com](mailto:questions@packtpub.com)>`与我们联系，我们将尽力解决问题。
