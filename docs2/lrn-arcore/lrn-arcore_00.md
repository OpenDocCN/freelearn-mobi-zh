# 前言

增强现实应用已经从新奇转变为现实，随着 ARKit 和 ARCore 的发布，它们对普通开发者来说变得更加容易接触。现在，几乎任何掌握编程语言的人都可以快速使用各种平台构建 AR 体验。随着 ARCore 的发布，谷歌现在使这一过程更加简单，并为多个开发平台提供支持。本书将指导您使用 JavaScript 和 Web 在移动端（Java/Android）以及移动端（C# / Unity）构建 AR 应用。在这个过程中，您将学习为您的用户构建高质量 AR 体验的基础知识。

# 本书面向的对象

本书面向任何想要使用 ARCore 深入构建增强现实应用的开发者，但没有任何游戏或图形编程背景。尽管本书仅假设读者具备基本的中学水平数学知识，但读者仍应至少掌握以下编程语言之一：JavaScript、Java 或 C#。

# 本书涵盖的内容

第一章，*入门*，涵盖了任何现代 AR 应用都需要解决的基本概念，以便为用户提供良好的体验。我们将学习运动跟踪、环境理解和光估计的基本概念。

第二章，*Android 上的 ARCore*，是使用 Android Studio 进行 Android 开发的入门指南，其中我们向您展示如何安装 Android Studio 并设置您的第一个 ARCore 应用。

第三章，*Unity 上的 ARCore*，讨论了如何使用 Unity 安装和构建 ARCore 应用。本章还向您展示如何使用 Android 开发工具远程调试应用。

第四章，*Web 上的 ARCore*，跳入使用 JavaScript 的 Web 开发，重点介绍如何使用 Node.js 设置您自己的简单 Web 服务器。然后，本章将浏览各种 ARCore 示例模板，并讨论如何扩展这些模板以进行进一步开发。

第五章，*现实世界运动跟踪*，扩展了上一章的学习内容，并将一个 Web 示例扩展到添加现实世界的运动跟踪。这不仅将展示与 3D 概念一起工作的几个基本原理，还将演示 ARCore 如何跟踪用户的运动。

第六章，*理解环境*，回到 Android 平台，处理 ARCore 如何理解用户的环境。我们将了解 ARCore 如何识别环境中的平面或表面并将它们网格化以供用户交互和可视化。在这里，我们将探讨如何修改着色器以测量和着色用户的数据点。

第七章，*光线估计*，解释了光照和阴影在向用户销售 AR 体验中所扮演的角色。我们学习了 ARCore 如何提供光线估计以及它是如何用于照亮用户放置到 AR 世界中的虚拟模型的。

第八章，*识别环境*，介绍了机器学习的基础以及这项技术对 AR 革命成功的重要性。然后我们转向构建一个简单的神经网络，通过监督训练使用称为反向传播的技术进行学习。在了解了神经网络和深度学习的基础之后，我们转向一个更复杂的示例，展示了各种机器学习的形式。

第九章，*为建筑设计融合光线*，介绍了构建一个 AR 设计应用程序，允许用户在客厅或他们需要的任何地方放置虚拟家具。我们还介绍了如何使用触摸在 AR 中放置和移动对象，以及如何识别对象已被选中。然后，我们将从第七章，*光线估计*，扩展我们的光照和阴影，并在虚拟对象上提供实时阴影。

第十章*，混合现实中的融合*，介绍了通过使用低成本的 MR 头戴式设备来引入混合现实。ARCore 非常适合用于这些低成本的耳机，因为它已经内部跟踪用户并监控其环境。我们将概述如何将我们的应用程序从使用 Unity 的 3D `WRLD` API 的传统地图应用程序转变为 AR 地图应用程序，同时我们还将提供一个选项来切换到 MR 和 MR 耳机。

第十一章，*性能提示和故障排除*，涵盖了我们在所有处理的开发平台上测量应用程序性能的技术。然后我们讨论了性能的重要性及其对各种系统可能产生的影响。之后，我们介绍了通用的调试和故障排除技巧，最后用一个表格总结了用户在此书中可能遇到的常见错误。

# 为了充分利用这本书

为了最大限度地利用这本书，以下是需要记住的事项：

+   读者需要精通以下编程语言之一：JavaScript、Java 或 C#

+   高中数学的记忆

+   支持 ARCore 的 Android 设备；以下链接可以查看设备列表：[`developers.google.com/ar/discover/`](https://developers.google.com/ar/discover/)

+   一台可以运行 Android Studio 和 Unity 的桌面电脑；不需要明确要求专门的 3D 显卡

# 下载示例代码文件

您可以从[www.packtpub.com](http://www.packtpub.com)的账户下载本书的示例代码文件。如果您在其他地方购买了本书，您可以访问[www.packtpub.com/support](http://www.packtpub.com/support)并注册，以便将文件直接通过电子邮件发送给您。

您可以按照以下步骤下载代码文件：

1.  在[www.packtpub.com](http://www.packtpub.com/support)登录或注册。

1.  选择 SUPPORT 标签。

1.  点击代码下载与勘误。

1.  在搜索框中输入书籍名称，并遵循屏幕上的说明。

文件下载后，请确保使用最新版本的软件解压或提取文件夹：

+   适用于 Windows 的 WinRAR/7-Zip

+   适用于 Mac 的 Zipeg/iZip/UnRarX

+   适用于 Linux 的 7-Zip/PeaZip

本书代码包也托管在 GitHub 上，地址为[`github.com/PacktPublishing/Learn-ARCore-Fundamentals-of-Google-ARCore`](https://github.com/PacktPublishing/Learn-ARCore-Fundamentals-of-Google-ARCore)。如果代码有更新，它将在现有的 GitHub 仓库中更新。

我们还有来自我们丰富的书籍和视频目录中的其他代码包，可在**[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)**找到。查看它们吧！

# 下载彩色图像

我们还提供包含本书中使用的截图/图表彩色图像的 PDF 文件。您可以从这里下载：[`www.packtpub.com/sites/default/files/downloads/LearnARCoreFundamentalsofGoogleARCore_ColorImages.pdf`](https://www.packtpub.com/sites/default/files/downloads/LearnARCoreFundamentalsofGoogleARCore_ColorImages.pdf)。

# 使用的约定

本书使用了多种文本约定。

`CodeInText`：表示文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 昵称。以下是一个示例：“向下滚动到`draw`方法，并在指定的行下方添加以下代码。”

代码块设置如下：

```java
void main() {
   float t = length(a_Position)/u_FurthestPoint;
   v_Color = vec4(t, 1.0-t,t,1.0);
   gl_Position = u_ModelViewProjection * vec4(a_Position.xyz, 1.0);
   gl_PointSize = u_PointSize;
}
```

当我们希望您注意代码块中的特定部分时，相关的行或项目将以粗体显示：

```java
uniform mat4 u_ModelViewProjection;
uniform vec4 u_Color;
uniform float u_PointSize;
uniform float u_FurthestPoint;
```

任何命令行输入或输出都按以下方式编写：

```java
cd c:\Android
npm install http-server -g
```

**粗体**：表示新术语、重要单词或您在屏幕上看到的单词。例如，菜单或对话框中的单词在文本中显示如下。以下是一个示例：“从管理面板中选择系统信息。”

警告或重要注意事项如下所示。

小贴士和技巧如下所示。

# 联系我们

我们欢迎读者的反馈。

**一般反馈**：请将电子邮件发送至`feedback@packtpub.com`，并在邮件主题中提及书名。如果您对本书的任何方面有疑问，请通过电子邮件联系我们`questions@packtpub.com`。

**勘误**：尽管我们已经尽一切努力确保内容的准确性，但错误仍然可能发生。如果您在这本书中发现了错误，我们将不胜感激，如果您能向我们报告这一点。请访问 [www.packtpub.com/submit-errata](http://www.packtpub.com/submit-errata)，选择您的书籍，点击勘误提交表单链接，并输入详细信息。

**盗版**：如果您在互联网上以任何形式发现我们作品的非法副本，如果您能提供位置地址或网站名称，我们将不胜感激。请通过发送链接至 `copyright@packtpub.com` 与我们联系。

**如果您有兴趣成为作者**：如果您在某个领域有专业知识，并且您有兴趣撰写或为书籍做出贡献，请访问 [authors.packtpub.com](http://authors.packtpub.com/)。

# 评论

请留下评论。一旦您阅读并使用了这本书，为何不在您购买它的网站上留下评论呢？潜在读者可以查看并使用您的客观意见来做出购买决定，我们 Packt 可以了解您对我们产品的看法，我们的作者也可以看到他们对书籍的反馈。谢谢！

如需更多关于 Packt 的信息，请访问 [packtpub.com](https://www.packtpub.com/)。
