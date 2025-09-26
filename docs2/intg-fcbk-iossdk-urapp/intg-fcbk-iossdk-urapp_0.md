# 前言

Facebook 是全球最受欢迎的社交网站，拥有超过 10 亿的用户基础。用户信任这个平台，并愿意通过第三方应用程序与之互动。利用社交平台功能的应用程序更有可能因为 Facebook 平台提供的不同渠道而走红。

本书将重点介绍 Facebook 平台的概念和功能实现。我们将使用 Facebook iOS SDK 创建一个新的社交应用程序，使用户能够利用通过其 API 提供的多种功能。

# 本书涵盖的内容

第一章，*Facebook 平台简介*，描述了社交网络平台的概念和社交网络上的不同功能。在这里，我们将学习社交网络的架构以及不同的部分如何相互交互。

第二章，*创建新的 iOS 社交项目*，介绍了如何创建一个将使用 Facebook iOS SDK 和 API 的新 iOS 应用程序。我们将学习如何设置一个新的 iOS XCode 项目，以便用户能够与社交平台互动。设置的一部分将涉及创建一个新的 Facebook App ID 和下载 iOS SDK。

第三章，*连接到 Facebook 用户账户*，描述了如何设置我们的应用程序，以便使用用户的 Facebook 凭证进行用户身份验证。我们将学习如何设计一个新的界面，使用户能够提供他们的 Facebook 凭证并在社交平台界面中进行身份验证。UI 实现的一部分将使用一个新的 SDK 组件，`FBLoginView`。我们将处理用户登录和注销流程。

第四章，*显示用户资料*，描述了我们可以从社交网络中检索哪些用户信息。我们将学习如何在用户成功认证后获取用户信息。我们将创建一个新的用户界面，以显示从 Facebook 平台获取的数据。

第五章，*在墙上发布*，描述了如何通过 Facebook iOS SDK 与 Facebook 用户的墙进行交互。我们将学习如何通过 Facebook iOS SDK 在用户的墙上分享由应用程序驱动的活动。我们将使用基于原生和基于 Web 的 API。我们将定制实现，以便选择当前登录用户或他们的任何朋友的墙。

第六章，*Facebook Graph API*，描述了如何使用 Graph API 与社交平台进行交互。Graph API 是查询和从平台获取信息的主要工具。我们将学习如何使用 Graph API 调试应用程序请求，以验证查询和响应，以及如何使用 Facebook iOS SDK 实现 Graph API。

第七章，*分发你的社交应用*，描述了如何让你的应用程序在众多用户中走红。在这里，我们将学习如何使用 Facebook 的推广渠道来与我们的 Facebook 朋友分享我们新的 iOS 社交应用。我们将负责在 Facebook 应用配置文件上设置所有必要的信息，以创建无瑕疵的分享体验。

第八章，*推广你的社交应用*，描述了如何使用 Facebook 工具来推广和跟踪我们的社交应用用户和用法。在这里，我们将学习如何使用 Facebook Insight 工具。本章的部分内容将专注于使用移动和网页广告来扩大我们的应用程序受众。

# 你需要本书的内容

为了开始使用 Facebook iOS SDK 进行开发，你应该拥有一台装有 XCode 的 Mac OS 机器。为了在 Facebook 社交图谱上注册你的应用程序，你需要一个 Facebook 账户。此外，为了将你的应用程序推广到全世界，你需要一个 Apple 的开发者账户。

# 本书面向的对象

本书是为从新手到专家的 iOS 开发者编写的。我肯定会推荐阅读来自 Apple 网站的故事板参考。本书使用 Objective-C、Storyboard 和 Facebook iOS SDK。

# 惯例

在本书中，你将找到多种文本样式，用于区分不同类型的信息。以下是一些这些样式的示例及其含义的解释。

文本中的代码单词如下所示：“将`FBLoginView`对象添加到我们的界面中，将使我们的社交应用能够以透明的方式执行登录和注销。”

代码块设置如下：

```swift
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
{ 
    [FBProfilePictureView class];
 return YES;
}
```

当我们希望引起你对代码块中特定部分的注意时，相关的行或项目将以粗体显示：

```swift
- (BOOL)application:(UIApplication *)application <span class="strong"><strong>didFinishLaunchingWithOptions:(NSDictionary *)launchOptions</strong></span>
{ 
    [FBProfilePictureView class];
 return YES;
}
```

**新术语**和**重要词汇**将以粗体显示。你在屏幕上看到的，例如在菜单或对话框中的单词，将以如下方式显示：“为了创建一个新的 Facebook App ID，你可以在 App 仪表板的左上角点击**创建新应用**”。

### 注意

警告或重要注意事项将以如下所示的框中出现。

### 小贴士

小技巧和窍门如下所示。

# 读者反馈

我们的读者反馈总是受欢迎的。告诉我们你对这本书的看法——你喜欢什么或可能不喜欢什么。读者反馈对我们开发你真正能从中获得最大收益的标题非常重要。

要发送一般反馈，只需发送一封电子邮件到`&lt;<a class="email" href="mailto:feedback@packtpub.com">feedback@packtpub.com</a>&gt;`，并在邮件的主题中提及书名。

如果你在一个领域有专业知识，并且你对撰写或为书籍做出贡献感兴趣，请参阅我们的作者指南[www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

现在你已经是 Packt 图书的骄傲拥有者，我们有许多事情可以帮助你从购买中获得最大收益。

## 下载示例代码

您可以从[`www.packtpub.com`](http://www.packtpub.com)下载您购买的所有 Packt 书籍的示例代码文件。如果您在其他地方购买了这本书，您可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)并注册，以便将文件直接通过电子邮件发送给您。

## 勘误

尽管我们已经尽最大努力确保内容的准确性，但错误仍然可能发生。如果您在我们的某本书中发现错误——可能是文本或代码中的错误——如果您能向我们报告这一点，我们将不胜感激。通过这样做，您可以避免其他读者感到沮丧，并帮助我们改进本书的后续版本。如果您发现任何勘误，请通过访问[`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)，选择您的书籍，点击**勘误提交表单**链接，并输入您的勘误详情来报告它们。一旦您的勘误得到验证，您的提交将被接受，勘误将被上传到我们的网站，或添加到该标题的勘误部分下的现有勘误列表中。您可以通过选择您的标题从[`www.packtpub.com/support`](http://www.packtpub.com/support)查看任何现有勘误。

## 盗版

互联网上对版权材料的盗版是所有媒体中持续存在的问题。在 Packt，我们非常重视保护我们的版权和许可证。如果您在互联网上发现我们作品的任何非法副本，无论形式如何，请立即提供位置地址或网站名称，以便我们可以寻求补救措施。

请通过链接发送至`&lt;<a class="email" href="mailto:copyright@packtpub.com">copyright@packtpub.com</a>&gt;`，告知我们疑似盗版材料的位置。

我们感谢您在保护我们的作者以及为我们提供有价值内容的能力方面提供的帮助。

## 问题

如果您在本书的任何方面遇到问题，可以通过`&lt;<a class="email" href="mailto:questions@packtpub.com">questions@packtpub.com</a>&gt;`联系我们，我们将尽力解决。
