# 前言

Flutter 是由 Google 开发的跨平台应用开发框架。它使用 Dart 编程语言来满足其开发需求。本书将指导您开始跨平台应用开发的旅程，通过帮助您理解 Flutter 的基本概念。

# 本书面向对象

本书旨在为对学习 Flutter 的基本概念以及学习如何构建跨平台应用感兴趣的读者提供帮助。

# 本书涵盖内容

第一章，*介绍 Flutter*，简要介绍了 Flutter 以及本书如何作为指南帮助您学习使用 Flutter 进行跨平台应用开发。然后，我们将探讨 Flutter 的起源及其如何与现有的移动应用开发世界相结合。

第二章，*开始使用 Flutter*，涵盖了使用 Flutter 所需工具的安装，使读者熟悉 IDE，并查看 Flutter 中最好的功能之一——热重载。然后，我们将了解每个应用开发工作流程中必需的两个主要概念——调试和测试。

第三章，*Widgets 无处不在*，遍历了 widget 目录，并解释了如何创建自定义 Widgets。然后，我们将学习如何在这些 Widgets 之间进行路由和导航。

第四章，*探索 Widgets 的多样性*，探讨了 Flutter 中的限制，并介绍了 Flutter 中的动画。然后，我们将学习如何使用 Listview 和滚动 Widgets，并在本章末尾介绍 silver。

第五章，*拓宽我们的 Flutter 视野*，解释了网络在应用中扮演的重要角色，并提供了设置和运行服务器以获取 JSON 代码的示例代码。本节之后，我们将了解为什么可访问性很重要，以及开发者可以带来哪些改进以支持应用的可访问性。下一节是关于应用支持国际化。

第六章，*使用平台为 Flutter 应用提供动力*，解释了如何在 Flutter 代码中包含包，然后介绍如何包含特定平台的通道以支持 Flutter 代码。然后，我们将使用 BatteryManager API 来了解 Android 手机的电池状态。我们将介绍在构建自己的插件之前需要考虑的一些最佳实践，然后介绍如何在 Flutter Pub 网站上发布自己的插件。

第七章，*Firebase - Flutter 的最佳朋友*，探讨了如何使用 Firestore 云数据库帮助我们更快地构建应用。我们还将查看一个使用 Firestore 云数据库捕获 ListView 的示例。最后，我们将讨论一些关于使用远程配置的应用用例。

第八章，*部署 Flutter 应用*，介绍了如何在相应的商店上部署和发布 Android 和 iOS 应用。

# 为了充分利用这本书

在开始阅读之前，一些 Dart 语言的经验将很有帮助，同时还需要在 Android、iOS 或任何移动开发框架上有经验。最后，熟悉面向对象的语言，如 Java 和 C++，以及一些 OOPS 知识将非常有用。

# 下载示例代码文件

您可以从[www.packt.com](http://www.packt.com)的账户下载本书的示例代码文件。如果您在其他地方购买了这本书，您可以访问[www.packt.com/support](http://www.packt.com/support)并注册，以便将文件直接通过电子邮件发送给您。

您可以通过以下步骤下载代码文件：

1.  在[www.packt.com](http://www.packt.com)登录或注册。

1.  选择 SUPPORT 选项卡。

1.  点击代码下载与勘误。

1.  在搜索框中输入书籍名称，并遵循屏幕上的说明。

一旦文件下载完成，请确保使用最新版本的以下软件解压或提取文件夹：

+   Windows 上的 WinRAR/7-Zip

+   Mac 上的 Zipeg/iZip/UnRarX

+   Linux 上的 7-Zip/PeaZip

该书的代码包也托管在 GitHub 上，链接为[`github.com/PacktPublishing/Google-Flutter-Mobile-Development-Quick-Start-Guide`](https://github.com/PacktPublishing/Google-Flutter-Mobile-Development-Quick-Start-Guide)。如果代码有更新，它将在现有的 GitHub 仓库中更新。

我们还有其他来自我们丰富的书籍和视频目录的代码包，可在以下链接找到：**[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)**。查看它们吧！

# 使用的约定

在本书中使用了多种文本约定。

`CodeInText`：表示文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 昵称。以下是一个示例：“将下载的`WebStorm-10*.dmg`磁盘映像文件挂载为系统中的另一个磁盘。”

代码块设置如下：

```java
void main() {
debugPaintSizeEnabled=true;
runApp(MyApp());
}
```

当我们希望您注意代码块中的特定部分时，相关的行或项目将以粗体显示：

```java
 Center(
child: Container(
decoration: BoxDecoration(border: Border.all()),
height: 200.0,
width: 200.0,
),
), 
```

任何命令行输入或输出都按以下方式编写：

```java
$ flutter packages get
```

**粗体**：表示新术语、重要单词或您在屏幕上看到的单词。例如，菜单或对话框中的单词在文本中显示如下。以下是一个示例：“从管理面板中选择系统信息。”

警告或重要提示看起来像这样。

小贴士和技巧看起来像这样。

# 联系我们

我们始终欢迎读者的反馈。

**总体反馈**：如果您对本书的任何方面有任何疑问，请在您的邮件主题中提及书名，并将邮件发送至 `customercare@packtpub.com`。

**勘误**：尽管我们已经尽最大努力确保内容的准确性，但错误仍然可能发生。如果您在这本书中发现了错误，我们将不胜感激，如果您能向我们报告这一点。请访问 [www.packt.com/submit-errata](http://www.packt.com/submit-errata)，选择您的书籍，点击勘误提交表单链接，并输入详细信息。

**盗版**：如果您在互联网上发现我们作品的任何非法副本，如果您能提供位置地址或网站名称，我们将不胜感激。请通过 `copyright@packt.com` 与我们联系，并附上材料的链接。

**如果您有兴趣成为作者**：如果您在某个领域有专业知识，并且您有兴趣撰写或为书籍做出贡献，请访问 [authors.packtpub.com](http://authors.packtpub.com/)。

# 评论

请留下评论。一旦您阅读并使用了本书，为何不在您购买书籍的网站上留下评论呢？潜在读者可以查看并使用您的客观意见来做出购买决定，Packt 公司可以了解您对我们产品的看法，我们的作者也可以看到他们对书籍的反馈。谢谢！

如需更多关于 Packt 的信息，请访问 [packt.com](http://www.packt.com/)。
