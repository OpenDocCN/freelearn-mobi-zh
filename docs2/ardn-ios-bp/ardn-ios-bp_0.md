# 前言

那是 2011 年的秋天，我正在开发一些 iOS 应用程序。在我的桌子上放着一块几乎未被使用的 Arduino 板。一个想法闪过我的脑海——将 iOS 平台和 Arduino 集成在一起会非常棒。我可以从任何地方控制几乎一切，从我的家到工业机器。

我开始着手这项工作，最终设计了 Arduino Manager。这是一个基于小部件的通用 iOS 应用程序，它从 Arduino 获取数据，并通过仪表、图表和其他方式展示它。Arduino Manager 允许你通过开关、旋钮、滑块等方式控制 Arduino 板（更多相关信息请参阅[`apple.co/1NPfL6i`](http://apple.co/1NPfL6i))。

本书展示了这些年来我开发的一些基本技术，以使 Arduino 和 iOS 设备协同工作。

我将向你展示如何构建五个令人惊叹的项目，你将学习如何让 Arduino 周围的额外电子设备工作。你还将学习如何使用数字和模拟传感器，编程 Arduino 板，并开发可以与 Arduino 传输数据的 iOS 应用程序。项目描述详细，为你提供学习工具，而不仅仅是复制一些草图或 iOS 代码。

我保证你不会感到无聊。

# 本书涵盖的内容

第一章，*Arduino 和 iOS – 平台和集成*，将简要介绍这两个平台，并介绍它们之间的集成方法。此外，你还将学习如何在两个平台上设置开发环境，并为后续章节中的项目做好准备。

第二章，*蓝牙宠物门锁*，开发了一个项目，可以帮助你通过测量外部光线、监控门是否锁定或未锁定以及根据需要手动操作，在夜间自动锁定宠物门。这是通过使用 iOS 设备完成的。

第三章，*Wi-Fi 电源插头*，是关于学习如何制作一个可以通过 Wi-Fi 控制的智能电源插头。这并不是基于传统的继电器技术，iOS 应用程序充满了实用功能。

第四章，*iOS 引导式漫游者*，将教你如何通过语音命令和移动你的 iOS 设备来控制漫游机器人。

第五章，*电视设置恒定音量控制器*，是为那些对广告音量过大感到厌烦的人准备的。你将学习如何制作一个自动红外控制器，以保持电视机的音量恒定。

第六章，*自动车库门开启器*，介绍了一个项目，通过靠近车库门即可自动开启，无需触摸 iOS 设备。该项目利用了 iBeacon 技术。

# 您需要为本书准备的内容

要构建项目的硬件部分，您需要一个 Arduino 板和一些其他电子组件。

要开发 Arduino 固件，您只需要 Arduino IDE，可以从 Arduino 网站免费下载。

要开发 iOS 应用程序，您需要 Xcode 开发环境（可以从 Apple 免费获得）。由于 Xcode 仅适用于 MAC 平台，因此您需要一个基于 Intel 的 MAC 电脑。大多数项目基于蓝牙 4 协议。因此，您需要一个支持此协议的 iOS 设备。

所有内容都在第一章，*Arduino 和 iOS – 平台和集成*中详细解释。

# 本书面向对象

本书是面向对 Arduino 和 iOS 平台有基本了解但想学习如何将它们集成的 Arduino 和 iOS 开发者技术指南。本书包含大量外部参考资料，指向额外的文档和学习材料。因此，即使是经验较少的读者也可以提高其在涵盖主题上的知识。

# 约定

在本书中，您将找到多种文本样式，用于区分不同类型的信息。以下是一些这些样式的示例及其含义的解释。

文本中的代码词汇、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 用户名如下所示：“通常，浏览器会下载到您的`下载`文件夹中。”

代码块设置如下：

```swift
        if (deviceIdentifier!=nil) {

            NSArray *devices = [_centralManager retrievePeripheralsWithIdentifiers:@[[CBUUID UUIDWithString:deviceIdentifier]]];
            _arduinoDevice = devices[0];
            _arduinoDevice.delegate = self;
        }
```

当我们希望您注意代码块中的特定部分时，相关的行或项目将以粗体显示：

```swift
        if (deviceIdentifier!=nil) {

            NSArray *devices = [_centralManager retrievePeripheralsWithIdentifiers:@[[CBUUID UUIDWithString:deviceIdentifier]]];
            _arduinoDevice = devices[0];
            _arduinoDevice.delegate = self;
        }
```

**新术语**和**重要词汇**以粗体显示。屏幕上显示的词汇，例如在菜单或对话框中，在文本中显示如下：“或者，您可以从 Launchpad 运行它，或者通过导航到**Finder** | **应用程序** | **App Store**。”

### 注意

警告或重要提示以如下方框显示。

### 小贴士

小贴士和技巧显示如下。

# 读者反馈

我们始终欢迎读者的反馈。让我们知道您对本书的看法——您喜欢或不喜欢的地方。读者反馈对我们很重要，因为它帮助我们开发出您真正能从中受益的标题。

要向我们发送一般反馈，请简单地发送电子邮件至 `<feedback@packtpub.com>`，并在邮件主题中提及本书的标题。

如果你在一个你擅长的主题上有专业知识，并且你对撰写或为书籍做出贡献感兴趣，请参阅我们的作者指南 [www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

现在你已经是 Packt 书籍的骄傲拥有者，我们有许多事情可以帮助你从购买中获得最大收益。

## 下载示例代码

你可以从 [`www.packtpub.com`](http://www.packtpub.com) 的账户下载示例代码文件，这是所有你购买的 Packt 出版物的示例代码。如果你在其他地方购买了这本书，你可以访问 [`www.packtpub.com/support`](http://www.packtpub.com/support) 并注册，以便将文件直接通过电子邮件发送给你。

## 错误清单

尽管我们已经尽一切努力确保我们内容的准确性，但错误仍然可能发生。如果你在我们的书中发现错误——可能是文本或代码中的错误——如果你能向我们报告这个问题，我们将不胜感激。通过这样做，你可以帮助其他读者避免挫败感，并帮助我们改进本书的后续版本。如果你发现任何错误清单，请通过访问 [`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)，选择你的书籍，点击**错误清单****提交****表单**链接，并输入你的错误详情来报告它们。一旦你的错误清单得到验证，你的提交将被接受，错误清单将被上传到我们的网站或添加到该标题的错误清单部分。

要查看之前提交的错误清单，请访问 [`www.packtpub.com/books/content/support`](https://www.packtpub.com/books/content/support)，并在搜索字段中输入书籍名称。所需信息将出现在**错误清单**部分。

## 盗版

在互联网上对版权材料的盗版是一个跨所有媒体的持续问题。在 Packt，我们非常重视保护我们的版权和许可证。如果你在互联网上发现任何形式的我们作品的非法副本，请立即提供位置地址或网站名称，以便我们可以寻求补救措施。

请通过 `<copyright@packtpub.com>` 联系我们，并提供疑似盗版材料的链接。

我们感谢你在保护我们的作者和我们提供有价值内容的能力方面的帮助。

## 问题

如果你对本书的任何方面有问题，你可以通过 `<questions@packtpub.com>` 联系我们，我们将尽力解决问题。
