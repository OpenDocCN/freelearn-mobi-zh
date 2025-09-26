# 前言

*Swift 基础知识*提供了 Swift 语言及其编写 iOS 应用程序所需的工具的概述。从使用 Swift 开源版本的简单命令行命令，到在 OS X 上使用 Xcode Playground 编辑器交互式测试图形内容，Swift 语言和语法通过示例进行介绍。

本书还通过展示如何创建一个简单的 iOS 应用程序，然后介绍如何使用故事板和自定义视图来构建更复杂的网络应用程序，介绍了在 OS X 上使用 Xcode 进行端到端 iOS 应用程序开发的流程。

本书通过提供一个从头开始构建的示例，构建了一个 iOS GitHub 仓库浏览器，以及一个 Apple Watch 应用程序，作为结尾。

# 本书涵盖的内容

第一章，*探索 Swift*，展示了带有 Swift Read-Evaluate-Print-Loop (REPL)的开源 Swift 版本，并通过标准数据类型、函数和循环的示例介绍了 Swift 语言。

第二章，*玩转 Swift*，展示了 Swift Xcode Playgrounds 作为交互式玩转 Swift 代码并查看图形结果的一种方式。它还介绍了 playground 格式，并展示了如何对 playground 进行文档化。

第三章，*创建 iOS Swift 应用程序*，展示了如何使用 Xcode 在 Swift 中创建和测试 iOS 应用程序，并概述了 Swift 类、协议和枚举。

第四章，*使用 Swift 和 iOS 的 Storyboard 应用程序*，介绍了 Storyboard 作为创建多屏幕 iOS 应用程序的手段，并展示了如何将 Interface Builder 中的视图连接到 Swift 的输出和动作。

第五章，*在 Swift 中创建自定义视图*，涵盖了使用自定义表格视图、布局嵌套视图以及绘制自定义图形和分层动画的 Swift 自定义视图。

第六章，*解析网络数据*，展示了 Swift 如何使用 HTTP 和自定义基于流的协议与网络服务进行通信。

第七章，*构建仓库浏览器*，使用本书中描述的技术构建了一个可以显示用户 GitHub 仓库信息的仓库浏览器。

第八章, *添加手表支持*，介绍了 Apple Watch 的功能，并展示了如何为 iOS 应用构建扩展以在手表上直接提供数据。

附录, *Swift 相关网站、博客和知名推特用户参考*，提供了额外的参考和资源，以继续学习 Swift。

# 您需要为本书准备

本书中的练习是为 Swift 2.1 编写和测试的，它包含在 Xcode 7.2 中，并与 Swift 2.2 的开发版本进行了验证。要实验 Swift，您需要一个符合 [`swift.org/download/`](https://swift.org/download/) 所示要求的 Mac OS X 或 Linux 计算机。

要运行第 2-8 章中涉及 Xcode 的练习，您需要一个运行 10.9 或更高版本且安装了 Xcode 7.2 或更高版本的 Mac OS X 计算机。如果发布了新的 Swift 版本，请检查本书的 GitHub 仓库或 Packtpub 的勘误页以获取可能影响本书内容的任何更改的详细信息。

### 注意

Swift 演示场（在第二章 [https://part0023_split_000.html#LTSU2-d7e55eb5242648e89c396442afe4f84b "第二章。与 Swift 玩耍"] 中描述），*与 Swift 玩耍*，仅作为 OS X 上 Xcode 的一部分提供，不是 Swift 开源版本的一部分。

此外，iOS 和 watchOS 开发（*第 3-8 章*）只能在 OS X 上使用 Xcode 进行；在其他平台上无法创建 iOS 或 watchOS 应用程序。iOS 开发所需的大部分库和模块在 Swift 的开源版本中不可用。

Xcode 可以通过 App Store 免费下载安装；在搜索框中搜索 `Xcode`。或者，您可以从 [`developer.apple.com/xcode/downloads/`](https://developer.apple.com/xcode/downloads/) 下载 Xcode，该链接来自 iOS 开发者中心 [`developer.apple.com/devcenter/ios/`](https://developer.apple.com/devcenter/ios/)。

Xcode 安装完成后，可以从 `/Applications/Xcode.app` 或 Finder 中启动。要运行基于命令行的练习，可以从 `/Applications/Utilities/Terminal.app` 启动终端，如果 Xcode 安装成功，可以通过运行 `xcrun swift` 启动 `swift`。

iOS 应用可以在 iOS 模拟器中开发和测试，该模拟器包含在 Xcode 中。编写或测试代码时不需要 iOS 设备。如果您想在您的 iOS 设备上运行代码，则需要 Apple ID 进行登录，但应用程序将仅限于直接连接的设备。同样，手表应用程序也可以在本地模拟器或本地设备上进行测试。

将应用程序发布到 AppStore 需要您加入**苹果开发者计划**。更多信息请参阅[`developer.apple.com/programs/`](https://developer.apple.com/programs/)。

# 本书面向的对象

本书旨在帮助对学习 Swift 编程语言感兴趣的开发者，无论是使用 Linux 上的 Swift 开源版本，还是使用 OS X 上 Xcode 打包的版本。然而，在第一章之后，即“探索 Swift”，剩余章节使用 Xcode 功能或包含仅能在 OS X 上使用 Xcode 的 iOS 示例。这些章节展示了如何使用 Swift 在 OS X 上编写 iOS 应用程序。虽然不假设有 iOS 编程经验，但预期读者具备动态或静态类型编程语言的基本编程经验。读者应熟悉导航和使用 Mac OS X，在需要终端命令的情况下，开发者应有简单 shell 命令的经验，或者可以从提供的示例中快速掌握。

熟悉 Objective-C 的开发者将了解许多提到的框架和库；然而，对 Objective-C 及其框架的现有知识既不是必需的，也不被假设。

源代码存储在 GitHub 仓库中，网址为[`github.com/alblue/com.packtpub.swift.essentials/`](https://github.com/alblue/com.packtpub.swift.essentials/)，您可以使用仓库中的标签在章节内容之间切换。如果您想在不同版本之间导航，了解 Git 知识会有所帮助；或者，您也可以使用 GitHub 的基于网页的界面。强烈建议读者熟悉 Git，因为它是 Xcode 的标准版本控制系统，也是开源项目的既定标准。如果读者不熟悉且想了解更多，请访问作者的博客[`alblue.bandlem.com/Tag/git/`](http://alblue.bandlem.com/Tag/git/)。

# 商标

GitHub 是 GitHub Inc. 的商标，本书中的示例未经 GitHub Inc. 批准、审查或认可。Mac 和 OS X 是 Apple Inc. 的商标，在美国和其他国家注册。iOS 是 Cisco 在美国和其他国家的商标或注册商标，并在此许可下使用。

# 术语

在本书中，您将找到许多不同的文本样式，用于区分不同类型的信息。以下是一些这些样式的示例及其含义的解释。

文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 用户名如下所示：“`"hello".hasPrefix("he")`方法在 OS X 和 iOS 上编译和运行成功。”

代码块设置如下：

```swift
> var shopping = [ "Milk", "Eggs", "Coffee", ]
shopping: [String] = 3 values {
  [0] = "Milk"
  [1] = "Eggs"
  [2] = "Coffee"
}
```

当我们希望将你的注意力引到代码块的一个特定部分时，相关的行或项目将以粗体显示：

```swift
func setupView() {
 contentMode = .Redraw
}
```

**新术语**和**重要词汇**以粗体显示。屏幕上看到的单词，例如在菜单或对话框中，在文本中显示如下：“Xcode 文档可以通过导航到**帮助** | **文档和 API 参考**来搜索。”

### 注意事项

警告或重要注意事项以这样的框形式出现。

### 小贴士

技巧和窍门以这样的形式出现。

# 读者反馈

我们欢迎读者的反馈。告诉我们你对这本书的看法——你喜欢什么或不喜欢什么。读者反馈对我们来说很重要，因为它帮助我们开发出你真正能从中获得最大收益的标题。

要发送给我们一般性的反馈，只需发送电子邮件至 `<feedback@packtpub.com>`，并在邮件主题中提及书籍的标题。

如果你在一个领域有专业知识，并且你对撰写或为书籍做出贡献感兴趣，请参阅我们的作者指南[www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

现在你已经是 Packt 图书的骄傲拥有者，我们有一些事情可以帮助你从购买中获得最大收益。

## 下载示例代码

你可以从你购买的所有 Packt 出版物的账户中下载示例代码文件。[`www.packtpub.com`](http://www.packtpub.com)。如果你在其他地方购买了这本书，你可以访问[`www.packtpub.com/support`](http://www.packtpub.com)，并注册以将文件直接通过电子邮件发送给你。

## 错误清单

尽管我们已经尽一切努力确保内容的准确性，但错误仍然可能发生。如果你在我们的书中发现错误——可能是文本或代码中的错误——如果你能向我们报告这一点，我们将不胜感激。通过这样做，你可以帮助其他读者避免挫败感，并帮助我们改进本书的后续版本。如果你发现任何错误清单，请通过访问[`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)，选择你的书籍，点击**错误提交表单**链接，并输入你的错误清单详情来报告它们。一旦你的错误清单得到验证，你的提交将被接受，错误清单将被上传到我们的网站或添加到该标题的错误部分下的现有错误清单中。

要查看之前提交的错误清单，请访问[`www.packtpub.com/books/content/support`](https://www.packtpub.com/books/content/support)，并在搜索字段中输入书籍的名称。所需信息将出现在**错误清单**部分下。

## 侵权

互联网上版权材料的侵权是一个持续存在的问题，涉及所有媒体。在 Packt，我们非常重视我们版权和许可证的保护。如果你在互联网上遇到任何形式的我们作品的非法副本，请立即提供位置地址或网站名称，以便我们可以寻求补救措施。

请通过 `<copyright@packtpub.com>` 联系我们，并提供涉嫌盗版材料的链接。

我们感谢您的帮助，以保护我们的作者和为您提供有价值的内容的能力。

## 问题和建议

如果您对本书的任何方面有问题，您可以通过 `<questions@packtpub.com>` 联系我们，我们将尽力解决问题。
