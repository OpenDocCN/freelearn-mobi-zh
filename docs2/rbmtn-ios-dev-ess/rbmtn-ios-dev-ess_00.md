# 前言

随着 iOS 设备系列的到来，软件开发的方向发生了根本性的变化。如今，人们花费大量时间在智能设备上而不是在 PC 上，这产生了前所未有的收入，这是任何行业都未曾见过的。尽管如此，它仍然可以放入您的口袋中。

到目前为止，iOS 生态系统中的应用程序开发场景一直由 Objective-C 主导。然而，随着革命性的 RubyMotion 工具链的引入，Ruby 开发者不再是创建纯原生 iOS 应用的局外人。他们可以利用 iOS SDK 的每一部分；而且最好的部分是，这可以在不使用 Xcode 的情况下完成。

Ruby 和 RubyMotion 都是一些想要在复杂世界中简化事物的人的智慧结晶。*松本行弘*（也被称为*Matz*）因创建 Ruby 编程语言而闻名，这种语言通常被认为是开发者的最佳朋友。而*Laurent Sansonetti*因创建革命性的工具链 RubyMotion 而受到赞誉。

RubyMotion iOS 开发必备将吸引开发者们的思维，特别是那些寻找可靠工具链进行 iOS 开发的科技官僚。这本书] 是从零开始构建 iOS 应用的逐步指南，直到部署。

# 本书涵盖的内容

第一章, *为 RubyMotion 做准备*，使您熟悉 RubyMotion。在这里，我们将从 RubyMotion 的介绍开始，然后是详细的安装步骤。

第二章, *即时满足 – 您的第一个应用*，解释了如何创建一个简单的 Hello World 应用，以及 RubyMotion 应用的总体结构。

第三章，*演变 – 从 Objective-C 到 RubyMotion*，帮助您理解从 Objective-C 到 RubyMotion 的旅程。本章也是理解 RubyMotion 语法与其 Objective-C 语法的快速指南。

第四章, *掌握 MVC 范式*，专注于使用模型-视图-控制器架构编写更好的代码。我们还将学习如何将应用程序连接到外部 API。

第五章, *用户界面 – 为您的应用添加魅力*，描述了用户界面是 iOS 应用的关键部分。此外，本章还解释了我们可以如何使用各种用户界面元素。

第六章, *设备功能 – 力量的释放*，教您如何使用各种设备功能，例如相机、位置管理器、手势、Core Data 和地址簿。我们将为每个功能创建示例应用，以便更好地理解它们。

第七章，*界面构建器和 WebView – 更多好东西！*，解释了如何使用界面构建器和 `UIWebView` 与 RubyMotion 应用程序一起使用。

第八章，*测试 – 让我们优雅地失败*，通过遵循测试驱动开发的原则，讨论了 RubyMotion 应用程序中的单元测试和功能测试。

第九章，*创建游戏应用程序*，帮助你使用 Cocoa2D 和 RubyMotion 创建流行的街机游戏 Whack-a-Mole。这是与 RubyMotion 一起工作的最激动人心和独特的功能之一，因为它可以创建图形游戏应用程序。

第十章，*为 App Store 准备*，解释了将 RubyMotion 应用程序提交到 Apple App Store 的过程。

第十一章，*扩展 RubyMotion*，描述了如何通过利用现有的开源宝石，如 TeaCup、BubbleWrap 和 Address Book，来增强我们的 RubyMotion 应用程序。

# 你需要这本书的内容

要使用 RubyMotion 编程，首先你需要一台 Macintosh 计算机。由于 RubyMotion 是专有软件，你必须从 [`sites.fastspring.com/hipbyte/product/rubymotion`](http://sites.fastspring.com/hipbyte/product/rubymotion) 购买其许可证。

# 本书面向的对象

本书是为那些精通 Ruby 编程语言并希望开发原生 iOS 应用的开发者所写。我们不期望你事先了解 RubyMotion。通过 RubyMotion iOS 开发基础知识，我们将从入门到专业级别探索这个令人惊叹的工具链的功能。

在某些时候，对 Objective-C 和 iOS SDK 的先验知识可能会有所帮助，但别担心，我们在本书的结尾处已经涵盖了每一个细节，让你成为一个精通者。

# 习惯用法

在本书中，你会找到多种文本样式，用于区分不同类型的信息。以下是一些这些样式的示例及其含义的解释。

文本中的代码词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 处理方式如下所示："在 `app` 文件夹内创建一个 `contact_us_controller.rb` 文件。"

代码块应如下设置：

```swift
    @submit_button.addTarget(self,
            action:''send_message'', forControlEvents:UIControlEventTouchUpInside)
```

当我们希望引起你对代码块中特定部分的注意时，相关的行或项目将以粗体显示：

```swift
def setupNavigationBar
  back= UIBarButtonItem.alloc.initWithTitle(''Back'', style:UIBarButtonItemStylePlain,target:nil ,action:nil)
  self.navigationItem.backBarButtonItem = back;
 contact_us_button = UIBarButtonItem.alloc.initWithTitle(""Contact Us"", style:UIBarButtonItemStylePlain ,target:self, action:""contact_us"")
 self.navigationItem.rightBarButtonItem = contact_us_button
end
def contact_us
 contact_us_controller = ContactUsController.alloc.initWithNibName(""ViewController"", bundle:nil)
 presentModalViewController(contact_us_controller, animated:true)
end
```

任何命令行输入或输出都应如下编写：

```swift
$rake

```

**新术语**和**重要词汇**将以粗体显示。屏幕上、菜单或对话框中看到的单词，例如，在文本中如下显示："打开 Xcode 并点击**创建一个新的 Xcode 项目**。"

### 注意

警告或重要注意事项将以如下框格形式出现。

### 提示

小技巧和窍门如下所示。

# 读者反馈

我们始终欢迎读者的反馈。告诉我们您对这本书的看法——您喜欢什么或可能不喜欢什么。读者反馈对我们开发您真正从中获得最大收益的标题非常重要。

要发送一般反馈，只需发送电子邮件到`<feedback@packtpub.com>`，并在邮件主题中提及书名即可。

如果您在某个主题上具有专业知识，并且您对撰写或为书籍做出贡献感兴趣，请参阅我们的作者指南[www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

现在您是 Packt 书籍的骄傲拥有者，我们有多个方面的事情可以帮助您从您的购买中获得最大收益。

## 下载示例代码

您可以从[`www.packtpub.com`](http://www.packtpub.com)的账户下载您购买的所有 Packt 书籍的示例代码文件。如果您在其他地方购买了这本书，您可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)并注册，以便将文件直接通过电子邮件发送给您。

## 勘误

尽管我们已经尽最大努力确保内容的准确性，但错误仍然可能发生。如果您在我们的某本书中发现错误——可能是文本或代码中的错误——如果您能向我们报告这一点，我们将不胜感激。通过这样做，您可以避免其他读者感到沮丧，并帮助我们改进本书的后续版本。如果您发现任何勘误，请通过访问[`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)来报告它们，选择您的书籍，点击**勘误****提交****表单**链接，并输入您的勘误详情。一旦您的勘误得到验证，您的提交将被接受，勘误将被上传到我们的网站，或添加到该标题的勘误部分下的现有勘误列表中。您可以通过选择您的标题从[`www.packtpub.com/support`](http://www.packtpub.com/support)查看任何现有勘误。

## 盗版

互联网上版权材料的盗版是一个跨所有媒体的持续问题。在 Packt，我们非常重视我们版权和许可证的保护。如果您在网上遇到我们作品的任何非法副本，无论形式如何，请立即提供位置地址或网站名称，以便我们可以寻求补救措施。

请通过发送电子邮件到`<copyright@packtpub.com>`并附上疑似盗版材料的链接与我们联系。

我们感谢您在保护我们的作者以及为我们提供有价值内容的能力方面的帮助。

## 询问

如果您在本书的任何方面遇到问题，可以通过`<questions@packtpub.com>`与我们联系，我们将尽力解决。
