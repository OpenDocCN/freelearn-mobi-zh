# 前言

在过去几年中，移动应用程序的受欢迎程度大幅增加，用户对它的兴趣仍在增长。移动操作系统不仅适用于智能手机，也适用于平板电脑，因此增加了这些应用程序的可能市场份额。

Android 具有让开发者感到愉悦的特性，如开源和具有高度社区驱动的开发。Android 在各个方面都与 iOS（苹果移动系统）竞争，与 XCode 相比，iOS 呈现为一个更加集中的开发环境。新的 Android Studio IDE 终于为 Android 开发者提供了这种集中化，并使这个工具对于优秀的 Android 开发者来说是不可或缺的。

这本关于 Android Studio 的书展示了用户如何使用这个新 IDE 开发并构建 Android 应用程序。它不仅是一本入门书籍，也是高级开发者构建应用程序更快、更高效指南。本书将采用教程方法，从基本功能到构建发布的步骤，包括实际示例。

# 本书涵盖的内容

第一章，*安装和配置 Android Studio*，描述了 Android Studio 的安装和基本配置。

第二章，*开始项目*，展示了如何创建新项目以及我们可以选择的活动类型。

第三章，*项目导航*，探讨了 Android Studio 中项目的基本结构。

第四章，*使用代码编辑器*，展示了代码编辑器的基本功能，以便充分利用它。

第五章，*创建用户界面*，专注于用户界面的创建，使用图形视图和基于文本的视图。

第六章，*工具*，介绍了某些额外的工具，例如 Android SDK 工具、Javadoc 和版本控制集成。

第七章，*Google Play Services*，介绍了当前现有的 Google Play Services 以及如何在 Android Studio 项目中集成它们。

第八章，*调试*，详细介绍了如何在 Android Studio 中调试应用程序以及调试时提供的信息。

第九章，*准备发布*，描述了如何为应用程序准备发布。

附录，*获取帮助*，介绍了如何使用 Android Studio 获取帮助，并提供了一个在线网站列表，以了解更多关于本书涵盖的主题。

# 你需要这本书的内容

对于这本书，您需要一个装有 Windows、Mac OS 或 Linux 系统的计算机。您还需要在系统中安装 Java。

# 这本书面向的对象

这本书不仅是一本入门书，也是为之前未使用 Android Studio 构建 Android 应用的资深开发者提供的指南。这本书非常适合想要学习 Android Studio 关键功能的开发者，也适合想要创建他们第一个应用程序的开发者。假设您熟悉面向对象编程范式和 Java 编程语言。还建议您了解 Android 移动系统的主要特点。

# 习惯用法

在这本书中，您将找到许多不同的文本样式，用于区分不同类型的信息。以下是一些这些样式的示例及其含义的解释。

文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 用户名如下所示：“`AppData`目录通常是一个隐藏目录。”

代码块设置如下：

```java
package ${PACKAGE_NAME};

import android.app.Activity;
import android.os.Bundle;

#parse("File Header.java")
public class ${NAME} extends Activity {
    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
    }
}
```

当我们希望您注意代码块中的特定部分时，相关的行或项目将以粗体显示：

```java
if (savedInstanceState != null) {

}
```

**新术语**和**重要词汇**以粗体显示。您在屏幕上看到的单词，例如在菜单或对话框中，在文本中显示如下：“您也可以通过点击屏幕左侧的**项目**按钮来打开它。”

### 注意

警告或重要注意事项以如下框的形式出现。

### 小贴士

小技巧和窍门如下所示。

# 读者反馈

我们始终欢迎读者的反馈。请告诉我们您对这本书的看法——您喜欢什么或不喜欢什么。读者反馈对我们很重要，因为它帮助我们开发出您真正能从中受益的书籍。

要向我们发送一般反馈，请简单地发送电子邮件至 `<feedback@packtpub.com>`，并在邮件主题中提及书籍的标题。

如果您在某个主题上具有专业知识，并且您对撰写或为书籍做出贡献感兴趣，请参阅我们的作者指南[www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

现在您已经是 Packt 图书的骄傲拥有者，我们有一些可以帮助您充分利用购买的东西。

## 下载示例代码

您可以从您的账户中下载这本书的示例代码文件。[`www.packtpub.com`](http://www.packtpub.com)。如果您在其他地方购买了这本书，您可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)并注册，以便将文件直接通过电子邮件发送给您。

您可以通过以下步骤下载代码文件：

1.  使用您的电子邮件地址和密码登录或注册我们的网站。

1.  将鼠标指针悬停在顶部的**支持**标签上。

1.  点击**代码下载与勘误表**。

1.  在**搜索**框中输入书籍的名称。

1.  选择您想要下载代码文件的书籍。

1.  从下拉菜单中选择您购买此书籍的来源。

1.  点击**代码下载**。

您还可以通过点击 Packt Publishing 网站上书籍网页上的**代码文件**按钮来下载代码文件。您可以通过在**搜索**框中输入书籍名称来访问此页面。请注意，您需要登录您的 Packt 账户。

下载文件后，请确保使用最新版本的以下软件解压或提取文件夹：

+   WinRAR / 7-Zip（适用于 Windows）

+   Zipeg / iZip / UnRarX（适用于 Mac）

+   7-Zip / PeaZip（适用于 Linux）

该书的代码包也托管在 GitHub 上，网址为[`github.com/PacktPublishing/Android_Studio_2_Essentials_Second_Edition_Code`](https://github.com/PacktPublishing/Android_Studio_2_Essentials_Second_Edition_Code)。我们还有其他来自丰富图书和视频目录的代码包，可在[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)找到。查看它们吧！

## 下载本书的彩色图像

我们还为您提供了一个包含本书中使用的截图/图表彩色图像的 PDF 文件。彩色图像将帮助您更好地理解输出的变化。您可以从[`www.packtpub.com/sites/default/files/downloads/AndroidStudio2EssentialsSecondEdition_ColorImages.pdf`](https://www.packtpub.com/sites/default/files/downloads/AndroidStudio2EssentialsSecondEdition_ColorImages.pdf)下载此文件。

## 勘误

尽管我们已经尽一切努力确保内容的准确性，但错误仍然可能发生。如果您在我们的某本书中找到错误——可能是文本或代码中的错误——如果您能向我们报告这一点，我们将不胜感激。通过这样做，您可以节省其他读者的挫败感，并帮助我们改进后续版本。如果您发现任何勘误，请通过访问[`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)，选择您的书籍，点击**勘误提交表单**链接，并输入您的勘误详情来报告它们。一旦您的勘误得到验证，您的提交将被接受，勘误将被上传到我们的网站或添加到该标题的勘误部分下的现有勘误列表中。

要查看之前提交的勘误表，请访问[`www.packtpub.com/books/content/support`](https://www.packtpub.com/books/content/support)并在搜索字段中输入书籍名称。所需信息将出现在**勘误**部分。

## 盗版

互联网上对版权材料的盗版是一个持续存在的问题，涉及所有媒体。在 Packt，我们非常重视我们版权和许可证的保护。如果您在互联网上发现我们作品的任何非法副本，请立即提供位置地址或网站名称，以便我们可以追究补救措施。

请通过 `<copyright@packtpub.com>` 联系我们，并提供涉嫌盗版材料的链接。

我们感谢您在保护我们的作者以及为我们提供有价值内容的能力方面的帮助。

## 问题

如果您在这本书的任何方面遇到问题，您可以通过 `<questions@packtpub.com>` 联系我们，我们将尽力解决问题。
