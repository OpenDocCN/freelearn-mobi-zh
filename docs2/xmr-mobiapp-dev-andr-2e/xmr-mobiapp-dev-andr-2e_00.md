# 前言

Xamarin 是基于 ECMA 标准的 .NET 框架的开源版本 Mono 的上层构建。Xamarin 为您提供了一套工具，包括其自己的 C# 编译器和公共语言运行时 (CLR)。Mono 框架源项目由位于旧金山的 Xamarin 公司（之前由 Novell 和最初由 Ximian 维护）维护。Mono 项目的首要目的是使 .NET 平台与 Linux 等其他非 Windows 平台兼容。

在 2011 年 4 月 Attachmate 收购 Novell 之后，Mono 平台的未来陷入了黑暗。几个月后，前 Novell 员工 Miguel de Icaza 创立了一家名为 Xamarin 的公司，并宣布将继续使用 Mono 平台进行商业软件开发。从那时起，Xamarin 赞助了 Mono 开源平台的发展，并为 iOS 和 Android 平台提供了商业 .NET 堆栈。iOS 的 .NET 被称为 MonoTouch 或 Xamarin.iOS，而 Android 的 .NET 被称为 Mono for Android 或 Xamarin.Android。

Xamarin 框架使开发者能够编写针对不同平台（包括 iOS、Android 和 Windows Phone）的跨平台移动应用程序。使用 Xamarin，您可以使用 C# 编程语言开发纯原生的 Android 或 iOS 应用程序，并在不同平台之间共享应用程序逻辑。这导致开发周期更快，开发者可以利用现有的 C# 和 .NET 编程技能，这有助于降低开发移动应用程序的学习曲线。

本书按照逻辑顺序组织，旨在帮助 C# 和 .NET 开发者从头开始构建 Xamarin.Android 应用程序。它解释了广泛使用的基调和高级 Android 概念，包括用户界面、数据存储、消费网络服务、地理位置、地图、摄像头以及构建和分发过程。

本书提供了对基本和高级 Xamarin.Android 概念的最全面解释；您可以通过实际的生活示例精确构建，以开发一个完整的工作应用。在本书的整个过程中，您将构建一个单一的应用程序，即 `POIApp`。通过这个应用程序，我们将涵盖所有 Xamarin.Android 的基础知识，以帮助您开始自己的应用程序开发。

# 本书涵盖的内容

第一章，*Android 应用的解剖结构*，概述了 Android 平台以及 Android 应用由什么组成。

第二章，*Xamarin.Android 架构*，概述了 Xamarin 平台并描述了 Mono 和 Android 运行时如何协同工作，以便开发者可以使用 C# 构建 Android 应用。

第三章，*创建兴趣点应用*，指导您如何设置开发环境，创建新的 Xamarin.Android 应用，并在 Android 模拟器中运行该应用。

第四章，*添加列表视图*，描述了 Android 的 AdapterView 架构，并指导你如何使用`ListView`和创建自定义适配器。本章还涵盖了如何从网络服务异步下载数据并在自定义`ListView`上显示响应。

第五章，*添加详细信息视图*，指导你如何创建详细信息视图以显示`POIApp`的详细信息，从列表视图中添加导航，并添加执行保存和删除网络服务操作的命令。

第六章，*使你的应用方向感知*，指导你如何检测设备方向并处理配置更改时的应用程序行为。

第七章，*为多种屏幕尺寸设计*，介绍了 Android 片段以及用于管理资源和支持多种屏幕尺寸（包括 Android 平板电脑）的不同技术。

第八章，*创建数据存储机制*，讨论了 Xamarin.Android 中可用的多种数据存储选项，并使用 SQLite 数据库引擎存储从网络服务获取的兴趣点列表，以便在设备离线时使列表可访问。

第九章，*使 POIApp 具有位置感知功能*，讨论了开发者为使他们的应用具有位置感知功能所拥有的各种选项，并介绍了如何添加逻辑以确定设备的地理位置、位置的地址以及在地图应用中显示位置。

第十章，*添加相机应用集成*，讨论了与设备相机集成的各种选项，以捕获`POIApp`的图片，并使用 HTTP 多部分表单上传将捕获的图片上传到网络服务。

第十一章，*将应用发布到应用商店*，讨论了分发 Android 应用的多种选项，并介绍了如何准备 Xamarin.Android 应用以进行分发。

# 你需要这本书的内容

本书中的所有示例都可以使用 Xamarin.Android 的 30 天试用版完成。这些示例是在 Mac OS X（Yosemite）、Xamarin Studio 5.9.3 和 Xamarin.Android 5.1.3（试用版）上开发的。只要它们是有效的 Xamarin 配置，任何后续版本都应该可以正常工作。你可以查看 Xamarin 网站以获取详细信息。

Xamarin.Android 也可以用于其他配置，包括 Windows 操作系统。在 Windows 操作系统上，你可以选择使用 Xamarin Studio 或 Visual Studio Xamarin 插件作为你选择的 IDE。使用与本书示例开发时不同的配置可能会导致书中描述的屏幕或步骤略有变化。

本书提供的示例使用了 Java JAX-RS 开发的 REST 网络服务。你可以在你的系统上部署网络服务代码以执行端到端测试，或者你可以使用代码包中提供的 Apiary 模拟数据 URL。要部署网络服务代码，你需要 MySQL 和 Apache Tomcat™应用程序服务器。

# 这本书面向的对象

这本书是为那些希望使用现有技能集开发 Android 应用的 C#和.NET 开发者编写的。本书包括使用 Xamarin 平台构建 Android 应用的逐步方法，无论你是经验丰富的移动开发者还是第一次尝试，都将非常有价值。

假设您在软件开发方面有一些经验，并且熟悉基本的面向对象开发概念和实践。理解 C#语法是必需的，并且对 C#有良好的实际知识是一个明显的优势，尽管这并不是严格必要的。

# 习惯用法

在本书中，你会发现许多文本样式，用于区分不同类型的信息。以下是一些这些样式的示例及其含义的解释。

文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 昵称显示如下：“这些常量被放置在一个名为`R.java`的 Java 源文件中。”

代码块设置如下：

```java
public override bool OnCreateOptionsMenu(IMenu menu)
{
    MenuInflater.Inflate(Resource.Menu.POIListViewMenu, menu);
    return base.OnCreateOptionsMenu(menu);
}
```

当我们希望将您的注意力引向代码块中的特定部分时，相关的行或项目将以粗体显示：

```java
public override Dialog OnCreateDialog (Bundle savedInstanceState)
  {
<span class="strong"><strong>    POIDetailFragment targetFragment = (POIDetailFragment) TargetFragment;</strong></span>
    string poiName = Arguments.GetString(&#x201C;name&#x201C;);
```

**新术语**和**重要词汇**以粗体显示。屏幕上显示的单词，例如在菜单或对话框中，在文本中显示如下：“点击**Step Over**两次以查看执行进度。”

### 注意

警告或重要注意事项以如下框的形式出现。

### 小贴士

技巧和窍门如下所示。

# 读者反馈

我们始终欢迎读者的反馈。告诉我们您对这本书的看法——您喜欢或不喜欢什么。读者反馈对我们来说很重要，因为它帮助我们开发出您真正能从中获得最大收益的标题。

要发送一般反馈，只需发送电子邮件至`&lt;<a class="email" href="mailto:feedback@packtpub.com">feedback@packtpub.com</a>&gt;`，并在邮件主题中提及本书的标题。

如果你在某个主题领域有专业知识，并且对撰写或为本书做出贡献感兴趣，请参阅我们的作者指南[www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

现在你已经是 Packt 图书的骄傲拥有者，我们有一些东西可以帮助你从购买中获得最大收益。

## 下载示例代码

您可以从您的账户[`www.packtpub.com`](http://www.packtpub.com)下载示例代码文件，适用于您购买的所有 Packt Publishing 图书。如果您在其他地方购买了这本书，您可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)并注册，以便将文件直接通过电子邮件发送给您。

## 下载本书的颜色图像

我们还为您提供了一个包含本书中使用的截图/图表的颜色图像的 PDF 文件。这些彩色图像将帮助您更好地理解输出的变化。您可以从[`www.packtpub.com/sites/default/files/downloads/B04210_Graphics.pdf`](https://www.packtpub.com/sites/default/files/downloads/B04210_Graphics.pdf)下载此文件。

## 勘误

尽管我们已经尽一切努力确保内容的准确性，但错误仍然可能发生。如果您在我们的某本书中发现错误——可能是文本或代码中的错误——如果您能向我们报告这一点，我们将不胜感激。通过这样做，您可以节省其他读者的挫败感，并帮助我们改进本书的后续版本。如果您发现任何勘误，请通过访问[`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)，选择您的书籍，点击**勘误提交表单**链接，并输入您的勘误详情来报告它们。一旦您的勘误得到验证，您的提交将被接受，勘误将被上传到我们的网站或添加到该标题的勘误部分下的现有勘误列表中。

要查看之前提交的勘误，请访问[`www.packtpub.com/books/content/support`](https://www.packtpub.com/books/content/support)，并在搜索字段中输入书籍名称。所需信息将出现在**勘误**部分下。

## 盗版

互联网上对版权材料的盗版是一个跨所有媒体的持续问题。在 Packt，我们非常重视我们版权和许可证的保护。如果您在互联网上发现任何形式的我们作品的非法副本，请立即提供位置地址或网站名称，以便我们可以寻求补救措施。

请通过`&lt;<a class="email" href="mailto:copyright@packtpub.com">copyright@packtpub.com</a>&gt;`与我们联系，并提供涉嫌盗版材料的链接。

我们感谢您在保护我们作者和我们为您提供有价值内容的能力方面的帮助。

## 问答

如果您对本书的任何方面有问题，您可以通过`&lt;<a class="email" href="mailto:questions@packtpub.com">questions@packtpub.com</a>&gt;`联系我们，我们将尽力解决问题。
