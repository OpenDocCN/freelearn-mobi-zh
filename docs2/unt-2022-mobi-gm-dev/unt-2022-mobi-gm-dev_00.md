# 前言

作为游戏开发者，您的目标是让您的客户在他们的位置上接触到您，随着每年越来越多的人购买移动设备，移动平台是一个需要考虑的关键平台。幸运的是，Unity 提供了跨平台功能，允许您一次编写游戏，然后通过最小修改将其移植到其他游戏机。然而，为移动设备开发也需要特定的考虑和功能，这正是*Unity 2022 移动游戏开发*所涉及的。

在本书中，我们将引导您通过使用 Unity 创建和部署移动游戏到 iOS 和 Android 的过程。我们将涵盖添加移动设备输入、设计适应各种屏幕尺寸的用户界面以及探索使用 Unity 的**内购**（**IAP**）和广告系统来盈利等基本主题。我们还将讨论使用通知来保留用户以及使用 Twitter 和 Facebook 的 SDK 与全世界分享您游戏的重要性。

此外，我们将深入研究 Unity 的分析系统以优化您游戏的表现并提供关于用户行为的见解。您还将学习在将游戏发布到 Google Play 和 iOS 应用商店之前，以各种方式对您的游戏进行润色。 

最后，我们将介绍 Unity 的 AR Foundation 框架的使用，该框架使您能够创建未来兼容且与多台设备兼容的**增强现实**（**AR**）应用。

在本书结束时，您将深入理解如何使用 Unity 进行移动游戏开发，包括移动设备特有的关键功能。

# 本书面向的对象

如果您是一位对为 iOS 和 Android 开发移动游戏感兴趣的 Unity 游戏开发者，那么这本书是您理想的资源。尽管具备 C#的相关知识会有帮助，但并非必需。无论您是经验丰富的开发者还是刚开始，本书提供的逐步指导将帮助您理解使用 Unity 进行移动游戏开发所需的独特功能和考虑因素。

# 本书涵盖的内容

*第一章*, *构建您的游戏*，通过创建一个将在整本书中修改以融入移动特定功能的基础项目，介绍了 Unity 游戏开发的基础。

*第二章*, *Android 和 iOS 开发的项目设置*，解释了如何配置您的开发环境以便将游戏部署到 Android 和 iOS 移动设备上。

*第三章*, *移动输入/触摸控制*，向您介绍移动输入的基本原理，包括触摸和手势识别、使用加速度计以及通过`Touch`类访问设备信息。

*第四章*, *分辨率无关 UI*，专注于如何构建分辨率无关的 UI 元素，这对于使用不同宽高比和分辨率的全部游戏项目都很有用。

*第五章*，*高级移动 UI*，基于前一章的知识，扩展到包括在 UI 上工作的移动特定方面，如需要屏幕控制，以及调整 UI 以适应有刘海的设备。

*第六章*，*实现应用内购买*，解释了如何将 Unity 的 IAP 系统集成到我们的项目中，包括创建可消耗和非可消耗的 IAP。

*第七章*，*使用 Unity Ads 进行广告宣传*，涵盖了将 Unity 的广告框架集成到我们的项目中，并探讨了创建简单和复杂广告的方法。

*第八章*，*将社交媒体集成到我们的项目中*，展示了如何通过整合分享到 Twitter 的高分等功能将社交媒体集成到您的游戏中，并使用 Facebook SDK 进行登录并显示玩家的姓名和头像。

*第九章*，*通过通知保持玩家参与度*，展示了如何将通知集成到您的游戏中，包括它们的设置、创建基本通知以及自定义它们呈现的方式。

*第十章*，*使用 Unity Analytics*，涵盖了将 Unity 的分析工具集成到您的游戏中，包括跟踪自定义事件和使用远程设置修改游戏玩法，而无需玩家重新下载游戏。

*第十一章*，*远程配置*，将展示如何轻松设置 Unity 的远程配置系统，以及我们如何通过改变玩家移动速度来改变游戏难度，通过简单示例利用它。

*第十二章*，*提升游戏感觉*，介绍了游戏设计中的“游戏感觉”概念，并探讨了如何集成缓动动画、材质、后处理效果和粒子效果来增强玩家体验。

*第十三章*，*构建游戏发布版本*，指导您如何为 iOS 和 Android 设备构建游戏发布版本所需的步骤。

*第十四章*，*将游戏提交到应用商店*，提供了将您的游戏提交到 Google Play 和 iOS 应用商店的技巧和窍门。

*第十五章*，*增强现实*，涵盖了将 AR 添加到您的游戏中的过程，包括 ARCore、ARKit 和 AR Foundation 的设置、安装和配置，检测现实世界中的表面，以及通过生成对象与环境交互。

# 为了充分利用这本书

在本书中，我们将使用 Unity 3D 游戏引擎进行开发，您可以从[`unity.com/download`](https://unity.com/download)下载。项目是使用 Unity 2022.1.0b16 创建的，但如果您使用的是该引擎的后续版本，则可能只需要进行最小更改。如果有一个新版本发布，并且您想下载本书中使用的确切版本，您可以在 Unity 的下载存档[`unity3d.com/get-unity/download/archive`](https://unity3d.com/get-unity/download/archive)中找到。您还可以在 Unity 编辑器的系统需求部分找到 Unity 的系统要求[`docs.unity3d.com/2022.1/Documentation/Manual/system-requirements.html`](https://docs.unity3d.com/2022.1/Documentation/Manual/system-requirements.html)。要部署您的项目，您需要一个 Android 或 iOS 设备。

为了简化，我们假设您在为 Android 开发时使用的是 Windows 操作系统，在为 iOS 开发时使用的是 Macintosh 计算机。

| **本书涵盖的软件/硬件** | **操作系统要求** |
| --- | --- |
| Unity 2022.1.0b16 | Windows, macOS, 或 Linux |
| Unity Hub 3.3.1 | Windows, macOS, 或 Linux |

**如果您使用的是本书的数字版，我们建议您亲自输入代码或从本书的 GitHub 仓库（下一节中提供链接）获取代码。这样做将帮助您避免与代码的复制和粘贴相关的任何潜在错误。**

# 下载示例代码文件

您可以从 GitHub 下载本书的示例代码文件：[`github.com/PacktPublishing/Unity-2022-Mobile-Game-Development-3rd-Edition`](https://github.com/PacktPublishing/Unity-2022-Mobile-Game-Development-3rd-Edition)。如果代码有更新，它将在 GitHub 仓库中更新。

我们还从我们丰富的图书和视频目录中提供了其他代码包，可在[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)找到。查看它们吧！

# 下载彩色图像

我们还提供了一份包含本书中使用的截图和图表彩色图像的 PDF 文件。您可以从这里下载：[`packt.link/6M4wR`](https://packt.link/6M4wR)。

# 使用的约定

本书使用了多种文本约定。

`文本中的代码`：表示文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 用户名。以下是一个示例：“这为我们提供了所需的代码 – 特别是，`GameNotificationManager`类 – 以添加到我们的脚本中。”

代码块设置如下：

```java
public void ShowNotification(string title, string body,
                                DateTime deliveryTime)
{
    IGameNotification notification =
    notificationsManager.CreateNotification();
    if (notification != null)
    {
        notification.Title = title;
        notification.Body = body;
        notification.DeliveryTime = deliveryTime;
        notification.SmallIcon = "icon_0";
        notification.LargeIcon = "icon_1";
        notificationsManager.ScheduleNotification(notification);
    }
}
```

当我们希望您注意代码块中的特定部分时，相关的行或项目将以粗体显示：

```java
ShowNotification("Endless Runner", notifText, notifTime);
        // Example of cancelling a notification
var id = ShowNotification("Test", "Should Not Happen", 
            notifTime);
        if(id.HasValue)
        {
            notificationsManager.CancelNotification(id.Value);
        }
        /* Cannot be added again until the user quits game */
        addedReminder = true;
    }
}
```

任何命令行输入或输出都按照以下方式编写：

```java
$ mkdir css
$ cd css
```

**粗体**: 表示新术语、重要单词或屏幕上看到的单词。例如，菜单或对话框中的单词会以**粗体**显示。以下是一个示例：“通过**编辑** | **项目设置**进入**项目设置**菜单。”

小贴士或重要注意事项

看起来像这样。

# 联系我们

我们始终欢迎读者的反馈。

**一般反馈**: 如果您对本书的任何方面有疑问，请通过电子邮件发送至 customercare@packtpub.com，并在邮件主题中提及书名。

**勘误**: 尽管我们已经尽一切努力确保内容的准确性，但错误仍然可能发生。如果您在此书中发现错误，我们将不胜感激，如果您能向我们报告，我们将不胜感激。请访问 [www.packtpub.com/support/errata](http://www.packtpub.com/support/errata) 并填写表格。

`copyright@packt.com` 并附上材料链接。

**如果您有兴趣成为作者**：如果您在某个主题上具有专业知识，并且您有兴趣撰写或为书籍做出贡献，请访问 [authors.packtpub.com](http://authors.packtpub.com)。

# 分享您的想法

一旦您阅读了《Unity 2022 移动游戏开发》，我们很乐意听听您的想法！请点击此处直接进入此书的亚马逊评论页面并分享您的反馈。

您的评论对我们和科技社区非常重要，并将帮助我们确保我们提供高质量的内容。

# 下载此书的免费 PDF 副本

感谢您购买此书！

您喜欢在路上阅读，但无法携带您的印刷书籍到处走？

您选择的设备是否不兼容电子书购买？

别担心，现在，随着每本 Packt 书籍，您都可以免费获得该书的 DRM 免费 PDF 版本。

在任何地方、任何设备上阅读。直接从您最喜欢的技术书籍中搜索、复制和粘贴代码到您的应用程序中。

优惠远不止于此，您还可以获得独家折扣、时事通讯和每日免费内容的每日电子邮件。

按照以下简单步骤获取好处：

1.  扫描二维码或访问以下链接

![](img/qr-code-https___packt.link_r_180461372X.jpg)

[`packt.link/free-ebook/9781804613726`](https://packt.link/free-ebook/9781804613726)

1.  提交您的购买证明

1.  就这样！我们将直接将您的免费 PDF 和其他好处发送到您的电子邮件。

# 第一部分：游戏玩法/开发设置

在本书的这一部分，我们将探讨 Unity 游戏开发的基础元素，特别是专注于创建移动游戏的准备工作。本部分中的章节将为您提供设置开发环境所需的知识和技能，并指导您构建游戏项目并将其部署到移动设备上。

到本部分结束时，你将拥有关于 Unity 游戏开发的知识基础，并准备好进入书中后续部分更高级的主题。

本部分包含以下章节：

+   *第一章*, *构建你的游戏*

+   *第二章*, *Android 和 iOS 开发的项目设置*
