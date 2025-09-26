# 前言

建立在上一代产品巨大成功的基础上，iOS 5 包含了超过 200 项新的用户功能，以及一个包含超过 1,500 个新 API 的更新版 SDK。iOS 5 看起来将加强 iPhone 在智能手机市场的统治地位。

iOS 5 基础知识将帮助你学习如何构建简单而强大的 iOS 5 应用程序，包括 iCloud 存储、Twitter、Core Image 和 Newsstand 集成。

你将从了解 iOS 5 的新功能开始。你将查看 iCloud 存储 API、自动引用计数、Twitter 和 AirPlay 集成、如何使用 Cocoa 框架中的各种 Core Image 过滤器，以及 iOS 5 SDK 的新特性。在此之后，你将直接开始使用 Xcode 和 Interface Builder，利用新的故事板布局创建应用程序。然后，我们将学习如何使用 Xcode 工具使你的应用程序运行得更加流畅。

在这本书中，我尽力使代码简单易懂。我在每个步骤都提供了详细的步骤说明，并附有大量的截图，以便更容易跟随。你将很快掌握 iOS 5 编程的各个方面，以及创建一些令人惊叹的应用程序所需的技术和技能。如果你有任何疑问，或者只是想打个招呼，请随时通过`<geniesoftstudios@gmail.com>`联系我。任何关于改进这本书的建议都将受到高度重视。

# 本书涵盖内容

第一章，*iOS5 新特性*，向开发者介绍了 Xcode 开发者工具集、iOS 5 的新特性，以及 Newsstand 和 `MessageUI` 框架的介绍。

第二章，*使用 iCloud 和存储 API*，介绍了使用 iCloud 的好处，以及如何将 iCloud 功能集成到你的应用程序中，以存储和检索文件及其数据，以及通过存储 API 使用其数据。本章还将为你提供一些关于如何处理多个 iOS 设备上同一文件更新时文件版本冲突的见解。

第三章, *使用 OpenGL ES 进行调试*，专注于顶点着色器和片段着色器之间的区别以及它们之间的关系。我们将熟悉 OpenGL ES 2.0 可编程管道，并探讨 OpenGL ES 的新调试功能，这些功能使我们能够在 Xcode IDE 中追踪特定于 OpenGL ES 的问题。我们将了解更多关于 OpenGL ES 帧捕获工具及其停止程序执行的能力，这将使开发者能够捕获 iOS 设备上正在渲染的当前帧内容，以便通过仔细查看程序状态信息来追踪和纠正程序问题，通过滚动调试导航器堆栈跟踪，并能够查看应用程序当前正在使用的所有纹理和着色器。

第四章, *使用 Storyboard*，帮助你理解 Storyboard 是什么以及我们如何应用视图之间的各种过渡。我们将探讨如何创建和配置场景和 Storyboard 文件，以程序化地展示这些内容。我们还将了解如何将 Twitter 功能集成到我们的应用程序中，以便发布照片和标准消息。

第五章, *使用 AirPlay 和 Core Image*，专注于学习 AirPlay 和 Core Image 框架，以及我们如何使用和将这些框架集成到我们的应用程序中。本章还解释了不同的图像过滤效果，如何调整图像的亮度，以及如何制作水波纹效果。它还涵盖了如何将 AirPlay 功能集成到你的应用程序中，以便你的应用程序可以在外部设备上显示，例如 Apple TV。

第六章, *Xcode 工具 - 改进*，专注于学习对 Xcode 开发工具所做的改进。我们将探讨**自动引用计数**（**ARC**），这是 LLVM 编译器最新添加的功能，以及它如何通过最小化应用程序的问题来提高应用程序的性能。它还涵盖了 Interface Builder、iOS 位置模拟器和 OpenGL ES 调试工具集的改进。

第七章, *使用 Instruments 使您的应用程序运行流畅*，重点介绍我们如何在应用程序中有效地使用 Instruments 来追踪可能导致应用程序在用户的 iOS 设备上崩溃的内存泄漏和瓶颈。我们将探讨 Instruments 应用程序中包含的每种不同类型的内置工具，学习如何使用系统跟踪工具来监控系统调用，并追踪应用程序中的性能问题。

# 您需要这本书的内容

本书假设您使用的是基于 Intel 的 Macintosh 运行 Snow Leopard（Mac OS X 10.6.2 或更高版本）。您可以使用 Leopard，但我强烈建议升级到 Snow Leopard 或 Lion，因为这两个操作系统才有许多仅在 Xcode 中可用的新功能。

我们将使用 Xcode 4.2.1，这是用于创建 iOS 开发应用程序的集成开发环境。您可以通过以下链接下载 Xcode 的最新版本：[`developer.apple.com/xcode/`](http://developer.apple.com/xcode/).

# 这本书面向的对象

如果您想了解 iOS 5 的最新功能，并学习如何将 Twitter、iCloud 和 Core Image 框架效果功能集成到您的应用程序中，那么这本书适合您。您应该具备良好的 Objective-C 编程经验，并使用过 Xcode 4。iOS 编程经验不是必需的。

# 术语

在这本书中，您将找到许多不同风格的文本，以区分不同类型的信息。以下是一些这些风格的示例及其含义的解释。

文本中的代码单词显示如下：“从 `/Developer/Applications` 文件夹启动 Xcode。”

代码块设置如下：

```swift
#import <UIKit/UIKit.h>
#import <MessageUI/MessageUI.h>
@interface MyEmailAppViewController: UIViewController<MFMailComposeViewControllerDelegate> {}
@end

```

当我们希望您注意代码块中的特定部分时，相关的行或项目将以粗体显示：

```swift
<plist version="1.0">
<dict>
<key>application-identifier</key>
<string>AXEUZ3F6VR.com.geniesoftstudios</string>
<key>com.apple.developer.ubiquity-container-identifiers</key>
<array>
<string>TEAMID.com.yourcompany.iCloudExample</string>
</array>
<key>com.apple.developer.ubiquity-kvstore-identifier</key>
<string>TEAMID.com.yourcompany.iCloudExample</string>
<key>get-task-allow</key>
<true/>
</dict>
</plist>

```

**新术语**和**重要词汇**以粗体显示。您在屏幕上看到的单词，例如在菜单或对话框中，在文本中显示如下：“当启动 Xcode 时，您应该看到**欢迎使用 Xcode**屏幕。”

### 注意

警告或重要注意事项以如下框显示。

### 小贴士

小贴士和技巧看起来像这样。

我们始终欢迎读者的反馈。请告诉我们您对这本书的看法，您喜欢或可能不喜欢的地方。读者的反馈对我们开发您真正能从中获得最大收益的标题非常重要。

要向我们发送一般反馈，只需发送一封电子邮件到`<feedback@packtpub.com>`，并在邮件的主题中提及书名。

如果您在某个主题上具有专业知识，并且您有兴趣撰写或为书籍做出贡献，请参阅我们的作者指南[www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

现在您是 Packt 书籍的骄傲拥有者，我们有一些事情可以帮助您从您的购买中获得最大收益。

## 下载示例代码

您可以从您在[`www.packtpub.com`](http://www.packtpub.com)的账户下载您购买的所有 Packt 书籍的示例代码文件。如果您在其他地方购买了这本书，您可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)，注册以直接将文件通过电子邮件发送给您。

## 错误

尽管我们已经尽一切努力确保我们内容的准确性，但错误仍然可能发生。如果您在我们的书中发现错误——可能是文本或代码中的错误——如果您能向我们报告，我们将不胜感激。这样做可以节省其他读者的挫败感，并帮助我们改进本书的后续版本。如果您发现任何错误，请通过访问[`www.packtpub.com/support`](http://www.packtpub.com/support)，选择您的书籍，点击**错误提交表单**链接，并输入您的错误详情来报告它们。一旦您的错误得到验证，您的提交将被接受，错误将被上传到我们的网站，或添加到该标题的错误部分现有的错误列表中。

## 盗版

互联网上对版权材料的盗版是一个跨所有媒体的持续问题。在 Packt，我们非常重视我们版权和许可证的保护。如果您在互联网上发现我们作品的任何非法副本，无论形式如何，请立即提供位置地址或网站名称，以便我们可以寻求补救措施。

请通过`<copyright@packtpub.com>`联系我们，并提供疑似盗版材料的链接。

我们感谢您在保护我们的作者以及为我们提供有价值内容的能力方面的帮助。

## 询问

如果您在本书的任何方面遇到问题，可以通过`<questions@packtpub.com>`联系我们，我们将尽力解决。
