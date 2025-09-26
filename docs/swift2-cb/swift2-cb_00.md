# 前言

经过一年的使用后，Swift 已经开始成熟并迅速添加新功能。苹果推出了 Swift 2，并带来了一些优秀的新特性和对 Xcode 及底层技术的改进。本书旨在更新希望迁移到 Swift 2 的 Objective-C 开发者，同时也帮助 Swift 开发者通过更好地了解这种编程语言及其第二个版本来获得更坚实的基础。

如果您喜欢创建小型应用程序，这本书非常适合您。它将向您展示如何从头开始创建 Swift 应用程序。所以，拿起您的 Mac，打开您的 Xcode，让我们来制作 Swift 吧！

# 本书涵盖内容

第一章, *使用 Xcode 和 Swift 入门*，向您介绍了一些 Xcode 的特定于 Swift 的功能。这可能对第一章来说有点高级，但这并不困难，也非常重要，尤其是对于那些希望专业开发的人来说。

第二章, *标准库和集合*，展示了如何使用 Swift 的方式操作数组、字典、集合、字符串和其他对象。对于一直在使用 Objective-C 的人来说，这一章非常重要。

第三章, *使用结构体和泛型*，展示了 Swift 的结构体与 Objective-C（甚至 C）的结构体不同，以及泛型是一个允许您创建不局限于单一类型的函数的功能。这两个功能都有它们自己的技巧。

第四章, *使用 Swift 实现设计模式*，解释了如何使用 Swift 实现设计模式，尤其是如果您喜欢面向对象编程的话。

第五章, *在您的应用程序中使用多任务*，展示了如何在您的应用程序中使用不同类型的多任务，这是一个几乎在当今每个应用程序中都存在的功能。

第六章, *使用 Playgrounds*，教您如何使用 Playgrounds，这是一个优秀的 Xcode 功能，允许您在将代码添加到项目之前对其进行测试。

第七章, *使用 Xcode 进行 Swift 调试*，解释了如何使用 Xcode、LLDB 和 Instruments 调试 Swift 代码。在这里，您将学习一些在您的应用程序中查找和解决 bug 的技巧。

第八章，*与 Objective-C 集成*，展示了 Swift 和 Objective-C 如何共存，并为你提供了如何将 Objective-C 应用程序迁移到 Swift 的逐步指南。

第九章，*处理其他语言*，展示了如何使用 C、C++ 和汇编语言与 Swift 一起使用，因为你知道 Swift 并非 iOS 和 OS X 开发的唯一选择。

第十章，*数据访问*，展示了不同的数据存储方式，这些数据可以是本地的或远程的。

第十一章，*扩展、照片和更多*，阐述了在 Swift 开发世界中非常重要的几个主题，从新的框架如 WatchKit 到广泛使用的框架。我们还将涵盖一些高级主题，如方法交换和关联对象。

# 您需要这本书什么

使用 Swift 2 进行开发需要 Xcode 7 或更高版本，而 Xcode 本身需要安装在 Yosemite（OS X 10.10）上。这只能在 Mac 计算机上安装。所以，这基本上是你需要的。

一些食谱只能在物理设备（iPhone、iPad 或 iPod）上测试；因此，只有当您注册了 Apple 开发者计划时才能安装。

# 这本书面向谁

如果你是一位经验丰富的 Objective-C 程序员，正在寻找 Swift 中快速解决许多不同编码任务的解决方案，那么这本书就是为你准备的。你预计将拥有开发经验，尽管不一定使用 Swift。

# 习惯用法

在这本书中，您将找到许多不同风格的文本，以区分不同类型的信息。以下是一些这些样式的示例及其含义的解释。

文本中的代码单词如下所示："创建一个名为 `Chapter 9 Assembly` 的新 Swift 单视图应用程序，并添加一个名为 `AssemblyCode.c` 的新文件。"

代码块如下设置：

```swift
struct Contact {
    char name[60];
    char phone[20];
    struct date {
        int day;
        int month;
        int year;
    } birthday;
};

struct ContactList {
    struct Contact contact;
    struct ContactList * next;
};
```

当我们希望将您的注意力引到代码块的一个特定部分时，相关的行或项目将被设置为粗体：

```swift
    override func viewDidLoad() {
        super.viewDidLoad()
        initializeContactList(&list)
    }
```

任何命令行输入或输出都如下所示：

```swift
6 / 3 = 2
6 / 2 = 3
6 / 1 = 6
Execution interrupted. Enter Swift code to recover and continue.
Enter LLDB commands to investigate (type :help for assistance.)

```

**新术语**和**重要词汇**以粗体显示。屏幕上看到的单词，例如在菜单或对话框中，在文本中如下所示："当 Xcode 请求桥接文件时，点击**是**。"

### 注意

警告或重要注意事项如下所示。

### 小贴士

小贴士和技巧看起来像这样。

# 读者反馈

我们欢迎读者的反馈。告诉我们您对这本书的看法——您喜欢或不喜欢什么。读者反馈对我们来说很重要，因为它帮助我们开发出您真正能从中获得最大收益的标题。

要发送给我们一般反馈，只需发送电子邮件至`<feedback@packtpub.com>`，并在邮件主题中提及书籍的标题。

如果您在某个主题上具有专业知识，并且您对撰写或为书籍做出贡献感兴趣，请参阅我们的作者指南[www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

现在，您已经成为 Packt 图书的骄傲拥有者，我们有多种方式可以帮助您从您的购买中获得最大收益。

## 下载示例代码

您可以从[`www.packtpub.com`](http://www.packtpub.com)的账户下载本书的示例代码文件。如果您在其他地方购买了这本书，您可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)并注册，以便将文件直接通过电子邮件发送给您。

您可以通过以下步骤下载代码文件：

1.  使用您的电子邮件地址和密码登录或注册我们的网站。

1.  将鼠标指针悬停在顶部的**支持**选项卡上。

1.  点击**代码下载 & 勘误**。

1.  在**搜索**框中输入书籍的名称。

1.  选择您想要下载代码文件的书籍。

1.  从下拉菜单中选择您购买这本书的地方。

1.  点击**代码下载**。

文件下载完成后，请确保您使用最新版本的软件解压缩或提取文件夹：

+   适用于 Windows 的 WinRAR / 7-Zip

+   适用于 Mac 的 Zipeg / iZip / UnRarX

+   适用于 Linux 的 7-Zip / PeaZip

## 勘误

尽管我们已经尽最大努力确保内容的准确性，错误仍然可能发生。如果您在我们的某本书中发现错误——可能是文本或代码中的错误——如果您能向我们报告这一点，我们将不胜感激。通过这样做，您可以避免其他读者感到沮丧，并帮助我们改进本书的后续版本。如果您发现任何勘误，请通过访问[`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)，选择您的书籍，点击**勘误提交表单**链接，并输入您的勘误详情来报告它们。一旦您的勘误得到验证，您的提交将被接受，勘误将被上传到我们的网站或添加到该标题的勘误部分下的现有勘误列表中。

要查看之前提交的勘误，请访问[`www.packtpub.com/books/content/support`](https://www.packtpub.com/books/content/support)，并在搜索字段中输入书籍的名称。所需信息将出现在**勘误**部分下。

## 盗版

互联网上版权材料的盗版是一个跨所有媒体的持续问题。在 Packt，我们非常重视我们版权和许可证的保护。如果您在互联网上发现任何形式的我们作品的非法副本，请立即提供位置地址或网站名称，以便我们可以寻求补救措施。

请通过链接将疑似盗版材料发送至 `<copyright@packtpub.com>` 与我们联系。

我们感谢您在保护我们作者和我们提供有价值内容的能力方面的帮助。

## 问题和建议

如果您对本书的任何方面有问题，请通过 `<questions@packtpub.com>` 与我们联系，我们将尽力解决问题。
