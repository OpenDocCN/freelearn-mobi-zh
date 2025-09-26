前言

自从苹果在 2014 年 WWDC 上宣布 Swift 编程语言以来，它已经成为增长最快的编程语言之一。Swift 是现代的、开源的、易于使用的，因此它的实用性可以扩展到苹果生态系统之外，使其有可能在所有平台上以及任何场景中使用。

Swift 5.3 是这款令人兴奋的新编程语言的最新版本，为您提供构建性能卓越、响应迅速的应用程序的工具，同时代码安全且整洁。

本书将引导您了解 Swift 的特性，逐步构建您的知识和工具集，以便您可以使用 Swift 构建下一个伟大的应用或服务。

您将获得使用 Swift 完成实际任务的有用、易于遵循的食谱。每个食谱只使用本书中之前覆盖过的概念，因此您永远不会感到迷茫。

了解是什么让 Swift 成为当今增长最快、最激动人心的编程语言之一。

# 第一章：本书面向对象

如果您正在寻找一本书来帮助您了解 Swift 5 提供的各种特性，以及高效编码和构建应用的技巧和窍门，那么这本书就是为您准备的。了解 Swift 或一般编程概念将有所帮助。

# 本书涵盖内容

第一章*,* *Swift 构建块*，向您介绍 Swift 5 的基本构建块、其语法以及基本 Swift 构造的功能。本章还将向您介绍苹果的 Xcode IDE 和 Swift Playgrounds，这提供了一个理想的方式来创建、执行、调试和理解本书中包含的食谱，从而为您启动开发过程做好准备。在本章中，您将学习如何编写您的第一个 Swift 程序，并理解 Swift 语言的各个基本元素。

第二章*,* *掌握构建块*，教您如何在第一章学习的基础构建块和 Swift 标准库提供的功能的基础上创建更复杂的结构。您将了解如何将变量打包成元组，借助数组对数据进行排序，以及使用字典存储键值对。您还将学习如何使用属性观察器来控制对代码的访问和可见性。然后，您将学习如何使用扩展来扩展代码的功能。

第三章，*使用 Swift 控制流进行数据处理*，将向您展示编程的全部内容都是关于做决策，因此在本章中，我们将探讨如何基于获得的信息做出决策，以及如何改变代码的控制流。您将学习如何使用`if`/`else`语句有条件地执行代码。您还将学习如何使用`switch`语句控制代码的执行流程，并通过理解如何使用`for`和`while`循环来循环此代码。然后您将了解如何使用`try`、`throw`、`do`和`catch`语句处理 Swift 错误。此外，我们还将介绍`defer`语句如何在函数执行完成后改变状态或清理不再需要的值时非常有用。

第四章，*泛型、操作符和嵌套类型*，为您提供了对 Swift 两个高级特性的理解，即泛型和操作符。使用这些特性，您将学习如何构建灵活且定义良好的功能。此外，您还将了解嵌套类型如何允许对您的构造进行逻辑分组、访问和命名空间。

第五章，*超越标准库*，带您探索 Foundation 和 UIKit 等框架提供的标准库之外的功能。学习如何使用这些功能将帮助您充分利用 Swift 语言。

第六章，*使用 Swift 构建 iOS 应用*，我们将开始构建我们自己的 iOS 应用的旅程。结合前几章的食谱中学到的所有内容，本章将教授您如何使用 Swift 构建 iOS 应用。

第七章，*Swift Playgrounds*，您将全面了解使用 Swift Playgrounds，并探索前几章未涉及的高级功能，以创建完全交互式的体验。

第八章*，*服务器端 Swift*，向您介绍 Swift 编程的全新方面：使用 Swift 进行服务器端编程。此外，您还将了解如何通过安装 Swift 工具链在 Linux 上运行 Swift。您将学习如何使用 Web 服务器框架构建 REST API，以及如何通过托管服务托管您的 API。您还将了解如何通过理解如何使用 Vapor（Swift 5 中最受欢迎的框架之一）来轻松完成任务。

第九章，*Swift 中的性能和响应性*，探讨了 Swift 编程的更高级概念，以了解某些 Swift 类型是如何实现及其性能特性的。此外，您还将学习如何使用 Grand Central Dispatch API 执行异步任务。然后，我们将探讨所有苹果平台上可用的多线程环境以及如何增强 Swift 结构的性能配置文件，以构建快速响应的应用程序。

第十章，*SwiftUI 和 Combine 框架*，直接深入苹果的新 UI 框架 SwiftUI。使用 SwiftUI，我们将通过探索用于实现此目的的声明性语法来学习如何创建 iOS 应用程序。我们还将探讨 SwiftUI 和 UIKit 如何协同工作。除了 SwiftUI，我们还将探索新 Combine 框架的基础知识以及它是如何与 SwiftUI 一起工作的。

第十一章，*在 Swift 中使用 CoreML 和 Vision*，将深入探讨苹果提供的框架以及我们如何使用 Swift 代码来利用机器学习技术，因为机器学习正变得越来越热门。在这一章中，我们将介绍 CoreML 和 Vision 框架，使用预训练模型来实时识别对象。

# 要充分利用本书

要跟随本书中的示例，您需要一个运行 macOS Catalina（特别是版本 10.15.4）或更高版本的计算机。您还需要一个 Apple ID，以便从 Mac App Store 下载并安装 Xcode 12。关于服务器端 Swift 的章节（第八章*,* *服务器端 Swift*)还需要一个运行 Ubuntu 20.04 LTS 的系统。

本书中的代码已在 Swift 5.3 上进行了测试，但应与任何更新的 Swift 版本兼容。

| **本书涵盖的软件/硬件** | **操作系统要求** |
| --- | --- |
| macOS | 10.15.4+ (Catalina) |
| Xcode | 12+ |
| Ubuntu（用于第八章*,* *服务器端 Swift*) | 20.04 LTS |

**如果您使用的是本书的数字版，我们建议您亲自输入代码或通过 GitHub 仓库（下一节中提供链接）访问代码。这样做将帮助您避免与代码复制粘贴相关的任何潜在错误。**

## 下载示例代码文件

您可以从 GitHub 下载本书的示例代码文件，网址为[`github.com/PacktPublishing/Swift-Cookbook-Second-Edition`](https://github.com/PacktPublishing/Swift-Cookbook-Second-Edition)。如果代码有更新，它将在现有的 GitHub 仓库中更新。

我们还有其他来自我们丰富图书和视频目录的代码包，可在**[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)**找到。查看它们吧！

## 代码实战

本书的相关代码在行动视频可以在[`bit.ly/3shdTeQ`](http://bit.ly/3shdTeQ)查看。

## 下载彩色图像

我们还提供了一个包含本书中使用的截图/图表的彩色图像的 PDF 文件。您可以从这里下载：[`static.packt-cdn.com/downloads/9781839211195_ColorImages.pdf`](https://static.packt-cdn.com/downloads/9781839211195_ColorImages.pdf)。

## 使用的约定

本书使用了多种文本约定。

`CodeInText`：表示文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 昵称。以下是一个示例：“我们可以在任何使用[Pug]或`Array<Pug>`的地方替换`Grumble`。”

代码块设置如下：

```swift
let fraction = rating / total 
let ratingOutOf5 = fraction * 5 
let roundedRating = round(ratingOutOf5) // Rounds to the nearest 
  // integer.
```

当我们希望将你的注意力引到代码块的一个特定部分时，相关的行或项目将以粗体显示：

```swift
class ProgrammeFetcher { 

    typealias FetchResultHandler = (String?, Error?) -> Void 

    func fetchCurrentProgrammeName(forChannel channel: Channel, 
                                   resultHandler: FetchResultHandler) { 
        // Get next programme 
        let programmeName = "Sherlock" 
        resultHandler(programmeName, nil) 
    } 
```

任何命令行输入或输出都按照以下方式编写：

```swift
$ mkdir css
$ cd css
```

**粗体**：表示新术语、重要单词或你在屏幕上看到的单词。例如，菜单或对话框中的单词在文本中看起来像这样。以下是一个示例：“从管理面板中选择系统信息。”

警告或重要提示看起来像这样。

小技巧和窍门看起来像这样。

# 章节

在这本书中，你会找到一些经常出现的标题（如*准备就绪*、*如何操作...*、*工作原理...*、*还有更多...*和*另请参阅*）。

为了清楚地说明如何完成食谱，请按照以下方式使用这些部分：

## 准备就绪

本节告诉你可以期待在食谱中看到什么，并描述如何设置任何软件或任何为食谱所需的初步设置。

## 如何操作…

本节包含遵循食谱所需的步骤。

## 如何操作…

本节通常包含对上一节发生情况的详细解释。

## 还有更多…

本节包含有关食谱的附加信息，以便你更了解食谱。

## 另请参阅

本节提供了对其他有用信息的链接，这些信息对食谱很有帮助。

# 联系我们

我们欢迎读者的反馈。

**一般反馈**：如果你对本书的任何方面有疑问，请在邮件主题中提及书名，并通过`customercare@packtpub.com`将邮件发送给我们。

**勘误**：尽管我们已经尽一切努力确保内容的准确性，但错误仍然可能发生。如果你在本书中发现错误，我们将不胜感激，如果你能向我们报告这一点。请访问[www.packtpub.com/support/errata](https://www.packtpub.com/support/errata)，选择你的书，点击勘误提交表单链接，并输入详细信息。

**盗版**：如果你在互联网上以任何形式遇到我们作品的非法副本，如果你能提供位置地址或网站名称，我们将不胜感激。请通过`copyright@packt.com`与我们联系，并提供材料的链接。

**如果您有兴趣成为作者**：如果您在某个领域有专业知识，并且您有兴趣撰写或为书籍做出贡献，请访问 [authors.packtpub.com](http://authors.packtpub.com/).

## 评论

请留下评论。一旦您阅读并使用了这本书，为何不在您购买它的网站上留下评论呢？潜在读者可以查看并使用您的客观意见来做出购买决定，我们 Packt 可以了解您对我们产品的看法，我们的作者也可以看到他们对书籍的反馈。谢谢！

关于 Packt 的更多信息，请访问 [packt.com](http://www.packt.com/).
