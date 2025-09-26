# 前言

自 2014 年 iOS 8 发布以来，iOS 平台的游戏开发经历了一些重大变化。这些变化中的第一个是 Swift 编程语言的引入，这是一种由苹果公司制作的函数式编程语言，旨在编写简单且功能现代，同时足够快以处理现代应用程序和游戏开发。iOS 8 还引入了 3D 游戏开发框架 SceneKit。SceneKit 允许开发者快速设计 3D 游戏，并以类似于 iOS 7 的 SpriteKit 的方式处理 3D 资产。一年后，在 2015 年夏季，推出了 iOS 9，同时为经验丰富的和全新的 iOS 游戏开发者提供了一套新工具。新的框架 GameplayKit 允许开发者分别处理游戏规则、人工智能、游戏实体和游戏状态，而不必从继承逻辑中分离出来。除了 GameplayKit 之外，苹果还展示了 Fox 游戏示例，展示了 Xcode 现在可以执行多平台游戏引擎（如 Unity 和 Unreal Engine）中看到的许多相同的视觉编辑功能。

我们首先将熟悉 Swift 编程语言及其在游戏开发中的应用范围。目标是深入理解 iOS 游戏开发，学习以代码为中心的更困难的方法来制作游戏。除了查看苹果公司为各种 iOS 游戏框架提供的示例项目外，我们还将看到两个游戏中的代码示例，第一个游戏是一个已发布的 Swift 开发的 2D 滚动游戏，名为 PikiPop，另一个是基于拼图的扫雷克隆游戏，名为 SwiftSweeper。随着我们阅读本书的进展，我们仍将代码作为游戏开发方法的核心，但将探讨 iOS 9 中引入的视觉工具，这些工具除了 GameplayKit 和基于组件的结构外，还可以使我们能够创建仅受想象力限制的游戏。然后，我们将深入探讨针对希望直接接触 GPU 的开发者的低级 API 主题，例如 OpenGL ES 和 Metal。

最后，我们希望您了解 iOS 如何继续成为强大的游戏开发平台，无论您是来自传统代码中心的计算机科学学校，还是您是日益增长的基于视觉/拖放设计范例的一部分。我们的目标是，当您完成这本书后，您将拥有许多截然不同且详细的游戏想法，然后您可以立即使用 Swift 和 iOS 9 平台开始构建。

# 本书涵盖的内容

[第 1 章](part0014_split_000.html#DB7S1-d06b23b4a4554b3182353558917969c2 "第 1 章. Swift 编程语言")，*Swift 编程语言*，提供了对 Swift 编程语言的介绍和指导。

[第二章](part0028_split_000.html#QMFO1-d06b23b4a4554b3182353558917969c2 "第二章. 使用 iOS 9 Storyboards 和 Segues 结构化和规划游戏"), *使用 Storyboards 和 Segues 结构化和规划游戏*，通过利用故事板和转场来规划或预先规划 iOS 游戏，帮助读者了解 iOS 应用的基本流程。

[第三章](part0033_split_000.html#VF2I1-d06b23b4a4554b3182353558917969c2 "第三章. SpriteKit 和 2D 游戏设计"), *SpriteKit 和 2D 游戏设计*，介绍了 iOS 2D 游戏开发框架 SpriteKit 的使用和解释。

[第四章](part0039_split_000.html#1565U1-d06b23b4a4554b3182353558917969c2 "第四章. SceneKit 和 3D 游戏设计"), *SceneKit 和 3D 游戏设计*，帮助读者深入了解 iOS 3D 开发框架 SceneKit 以及 iOS 中引入的可用于 SceneKit 和 SpriteKit 的视觉工具。

[第五章](part0047_split_000.html#1CQAE2-d06b23b4a4554b3182353558917969c2 "第五章. GameplayKit"), *Gameplaykit*，介绍了 GameplayKit 框架，这是一个在 iOS 9 中引入的通用游戏逻辑和 AI 框架。

[第六章](part0050_split_000.html#1FLS41-d06b23b4a4554b3182353558917969c2 "第六章. 在你的游戏中展示 Metal"), *在你的游戏中展示 Metal*，讨论了高级主题，如 GPU 图形管道以及 Apple 的低级 API Metal 的介绍，以从你的游戏中获得最佳性能。

[第七章](part0055_split_000.html#1KEEU2-d06b23b4a4554b3182353558917969c2 "第七章. 发布我们的 iOS 游戏"), *发布我们的 iOS 游戏*，通过利用测试和质量保证服务 TestFlight，解释了进行测试和发布 iOS 游戏所需的步骤。

[第八章](part0059_split_000.html#1O8H61-d06b23b4a4554b3182353558917969c2 "第八章. iOS 游戏开发的未来"), *iOS 游戏开发的未来*，讨论了编程、iOS 以及整体游戏开发在不久的将来可能会如何改变或改进，以及它与最新的 iOS 平台和框架的关系。

# 你需要这本书什么

这本书你需要什么：

+   Xcode 7 或更高版本

+   Mac OS X Yosemite 或更高版本

# 本书面向对象

本书旨在为希望为 iPhone 和 iPad 开发 2D 和 3D 游戏的游戏开发者编写。如果你来自其他平台或游戏引擎，如 Android 或 Unity，是一名希望了解更多 Swift 和 iOS 9 最新特性的当前 iOS 开发者，或者即使你是游戏开发的新手，这本书也是为你准备的。建议具备一定的编程知识，但不是必需的。

# 习惯用法

在这本书中，你将找到许多文本样式，用于区分不同类型的信息。以下是一些这些样式的示例及其含义的解释。

文本中的代码单词、文件名、文件扩展名、路径名和用户输入以如下方式显示：“正如我们所见，`AppDelegate.swift`和`ViewController.swift`文件为我们自动创建，紧接着我们就能找到`Main.Storyboard`文件。”

代码块设置如下：

[PRE0]

当我们希望您注意代码块中的特定部分时，相关的行或项目将以粗体显示：

[PRE1]

新术语和重要词汇将以粗体显示。例如，您在屏幕上看到的单词，如菜单或对话框中的单词，在文本中显示如下：“或者，想象一下玩家失去了所有生命，并收到了一个**游戏结束**的消息。”

### 注意

警告或重要注意事项以如下框的形式出现。

### 小贴士

小贴士和技巧看起来是这样的。

# 读者反馈

我们始终欢迎读者的反馈。请告诉我们您对这本书的看法——您喜欢什么或不喜欢什么。读者反馈对我们来说非常重要，因为它帮助我们开发出您真正能从中获得最大价值的书籍。

要向我们发送一般反馈，只需发送电子邮件至 `<[feedback@packtpub.com](mailto:feedback@packtpub.com)>`，并在邮件主题中提及书籍的标题。

如果您在某个主题上具有专业知识，并且您对撰写或为书籍做出贡献感兴趣，请参阅我们的作者指南[www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

现在您已经是Packt书籍的骄傲拥有者，我们有一些事情可以帮助您从您的购买中获得最大价值。

## 下载示例代码

您可以从您在[http://www.packtpub.com](http://www.packtpub.com)的账户下载示例代码文件，适用于您购买的所有Packt出版社的书籍。如果您在其他地方购买了这本书，您可以访问[http://www.packtpub.com/support](http://www.packtpub.com/support)并注册，以便将文件直接通过电子邮件发送给您。

## 下载本书的彩色图像

我们还为您提供了一个包含本书中使用的截图/图表彩色图像的PDF文件。彩色图像将帮助您更好地理解输出的变化。您可以从[http://www.packtpub.com/sites/default/files/downloads/iOS9_GameDevelopmentEssentials_ColorImages.pdf](http://www.packtpub.com/sites/default/files/downloads/iOS9_GameDevelopmentEssentials_ColorImages.pdf)下载此文件。

## 错误清单

尽管我们已经尽一切努力确保内容的准确性，但错误仍然可能发生。如果您在我们的某本书中发现错误——可能是文本或代码中的错误——如果您能向我们报告这一点，我们将不胜感激。通过这样做，您可以节省其他读者的挫败感，并帮助我们改进本书的后续版本。如果您发现任何勘误，请通过访问 [http://www.packtpub.com/submit-errata](http://www.packtpub.com/submit-errata)，选择您的书籍，点击**勘误提交表单**链接，并输入您的勘误详情来报告它们。一旦您的勘误得到验证，您的提交将被接受，勘误将被上传到我们的网站或添加到该标题的勘误部分下的现有勘误列表中。

要查看之前提交的勘误，请访问 [https://www.packtpub.com/books/content/support](https://www.packtpub.com/books/content/support)，并在搜索字段中输入书籍名称。所需信息将出现在**勘误**部分下。

## 侵权

在互联网上对版权材料的侵权是一个跨所有媒体的持续问题。在Packt，我们非常重视保护我们的版权和许可证。如果您在互联网上发现任何形式的我们作品的非法副本，请立即提供位置地址或网站名称，以便我们可以寻求补救措施。

请通过 `<[copyright@packtpub.com](mailto:copyright@packtpub.com)>` 联系我们，并提供疑似侵权材料的链接。

我们感谢您在保护我们的作者和我们为您提供有价值内容的能力方面的帮助。

## 问题

如果您在这本书的任何方面遇到问题，您可以通过 `<[questions@packtpub.com](mailto:questions@packtpub.com)>` 联系我们，我们将尽力解决问题。
