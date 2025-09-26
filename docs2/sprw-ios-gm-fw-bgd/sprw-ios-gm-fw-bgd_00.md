# 前言

游戏开发可能是软件开发中最具挑战性和最有回报的挑战之一。如果我们从头开始，将需要非常长的时间才能看到任何结果。

With the introduction of the iPhone in 2007 and subsequent devices in the following years, developing applications for mobile devices took off, and more than 1,000,000 apps can now be downloaded from the App Store.

幸运的是，Sparrow，一个开源的 iOS 游戏框架，为我们提供了一系列预定义的类和方法，这将有助于我们的游戏开发过程。

Instead of showing how to develop a part of a game example-by-example during the course of the book, we will learn each stage of game development. With each chapter, our game will mature from being just an idea to a complete entity, while extending our knowledge of Sparrow.

# 本书涵盖的内容

第一章, *Sparrow 入门*，*s*hows us how to set up Xcode, Sparrow, and our game template that we will use throughout the book. This chapter also sets up our goals and expectations for the kind of game we will develop.

第二章, *显示我们的第一个对象*，explains the concept of display objects, which we need to achieve in order to get anything to show up on the screen, and how to manipulate these objects.

第三章, *管理资源和场景*，introduces us to the concepts of scene and asset management and how to implement them for our purposes.

第四章, *我们游戏的基础*，deals with setting up our game to work on iPhone, iPod Touch, and iPad in the same manner. We will also create the game skeleton in this chapter.

第五章, *美化我们的游戏*，covers moving and animating our entities on the screen. We will also learn how to generate sprite sheets, what to consider when using sprite sheets, and how to integrate them into our game.

第六章, *添加游戏逻辑*，focuses on getting actual gameplay into our game as well as managing our game-relevant data in separate files.

第七章, *用户界面*，shows us how to implement the user interface in our game, for example, displaying text on the screen, structuring our user interface, and updating the user interface to what is currently happening in the game.

第八章, *人工智能和游戏进程*，explains what we need to know in order to implement basic artificial intelligence and how we need to apply this for our enemies in the game.

第九章，*为我们的游戏添加音频*，介绍了如何加载音频以及如何在我们的游戏中集成它们。

第十章，*完善我们的游戏*，涉及为我们的游戏添加最后的 10%。我们将添加主菜单、开场和教程机制，以获得更流畅的游戏体验。

第十一章，*集成第三方服务*，探讨了如何集成第三方服务，如 Apple Game Center，以改善玩家的体验。

# 你需要为本书准备的东西

为了开发 iOS 应用程序，你需要一台 Mac，最好是最新版本的 Mac OS X。尽管 Apple 开发者账户和 iOS 开发者计划不是必需的，但建议使用，因为它允许你在实际的设备上运行示例，例如 iPod Touch、iPhone 和 iPad，并将你的应用程序分发到苹果应用商店。请记住，iOS 开发者计划会带来额外的费用。

在你的系统上不需要安装 Sparrow 和 Xcode；我们将在第一章中介绍安装过程。

# 本书面向的对象

本书旨在为对游戏开发感兴趣的人、已经涉足游戏开发但尚未为移动设备制作过游戏的人，以及希望在将来将游戏发布到苹果应用商店的人提供帮助。

你需要具备扎实的 Objective-C 理解能力才能跟随书中的示例，并且一些游戏开发经验肯定有帮助，尽管这不是必需的。

# 惯例

在本书中，你会发现几个标题频繁出现。

为了清楚地说明如何完成一个程序或任务，我们使用：

# 行动时间 – 标题

1.  动作 1

1.  动作 2

1.  动作 3

指令通常需要一些额外的解释，以便它们有意义，因此它们后面跟着：

## *刚才发生了什么？*

这个标题解释了你刚刚完成的任务或指令的工作原理。

你还会在本书中找到一些其他的学习辅助工具，包括：

## 快速测验 – 标题

这些是简短的多选题，旨在帮助你测试自己的理解。

## 有所作为的英雄 – 标题

这些实践挑战为你提供了实验你所学内容的想法。

你也会发现许多文本样式，这些样式可以区分不同类型的信息。以下是一些这些样式的示例，以及它们含义的解释。

文本中的代码词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和推特用户名如下所示："这个类需要从 `SPSprite` 类继承。"

代码块设置如下：

```swift
// Setting the background
SPSprite *background = [[SPSprite alloc] init];
[self addChild:background];

// Loading the logo image and bind it on the background sprite
SPSprite *logo = [SPImage imageWithContentsOfFile:@"logo.png"];
[background addChild:logo];
```

当我们希望引起你对代码块中特定部分的注意时，相关的行或项目将以粗体显示：

```swift
// Setting the background
SPSprite *background = [[SPSprite alloc] init];
[self addChild:background];

// Loading the logo image and bind it on the background sprite
SPSprite *logo = [SPImage imageWithContentsOfFile:@"logo.png"];
[background addChild:logo];

```

任何命令行输入或输出都应如下编写：

```swift
sudo gem install cocoapods
pod setup
touch Podfile
pod install

```

**新** **术语** 和 **重要** **词汇** 以粗体显示。您在屏幕上看到的单词，例如在菜单或对话框中，在文本中显示如下：“在**选择** **目标** **位置**屏幕上，点击**下一步**以接受默认目标。”

### 注意

警告或重要提示会以这样的框中出现。

### 小贴士

小贴士和技巧会像这样显示。

# 读者反馈

读者的反馈总是受欢迎的。请告诉我们您对这本书的看法——您喜欢什么或可能不喜欢什么。读者反馈对我们开发您真正能从中获得最大价值的标题非常重要。

要发送一般反馈，只需发送一封电子邮件到 `<feedback@packtpub.com>`，并在邮件主题中提及书名。

如果您在某个领域有专业知识，并且对撰写或为书籍做出贡献感兴趣，请参阅我们的作者指南，网址为 [www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

现在您是 Packt 书籍的骄傲拥有者，我们有一些事情可以帮助您从您的购买中获得最大收益。

## 下载示例代码

您可以从您在 [`www.packtpub.com`](http://www.packtpub.com) 的账户下载所有已购买的 Packt 书籍的示例代码文件。如果您在其他地方购买了这本书，您可以访问 [`www.packtpub.com/support`](http://www.packtpub.com/support) 并注册，以便直接将文件通过电子邮件发送给您。

## 错误更正

尽管我们已经尽一切努力确保我们内容的准确性，但错误仍然会发生。如果您在我们的某本书中发现错误——可能是文本或代码中的错误——如果您能向我们报告这一点，我们将不胜感激。通过这样做，您可以节省其他读者的挫败感，并帮助我们改进此书的后续版本。如果您发现任何错误，请通过访问 [`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)，选择您的书籍，点击**错误** **提交** **表单**链接，并输入您的错误详情来报告它们。一旦您的错误得到验证，您的提交将被接受，错误将被上传到我们的网站，或添加到该标题的**错误**部分中现有的错误列表。

## 盗版

互联网上版权材料的盗版是一个跨所有媒体的持续问题。在 Packt，我们非常重视我们版权和许可证的保护。如果您在互联网上发现我们作品的任何非法副本，无论形式如何，请立即提供位置地址或网站名称，以便我们可以追究补救措施。

请通过 `<copyright@packtpub.com>` 联系我们，并提供涉嫌盗版材料的链接。

我们感谢您在保护我们作者和提供有价值内容方面的帮助。

## 询问

如果你在本书的任何方面遇到问题，可以通过 `<questions@packtpub.com>` 联系我们，我们将尽力解决。
