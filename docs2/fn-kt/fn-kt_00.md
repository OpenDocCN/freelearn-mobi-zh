# 前言

随着 Google 在 2017 年 I/O 大会上的宣布，将 Kotlin 定为 Android 的官方语言，Kotlin 在全球开发者中的受欢迎程度正在上升。

然而，Kotlin 的使用和流行并不局限于 Android 社区。许多其他社区，如桌面、Web 和后端社区，也在接受 Kotlin。许多新的库和框架正在被创建，现有的也在为 Kotlin 提供支持。

随着越来越多的开发者加入 Kotlin 社区，以及其自然的灵活性，更多的编程风格正在被尝试。本书的目的是向广泛的 Kotlin 社区介绍函数式编程风格，引领和指导初学者，并提供基本工具以进一步学习更高级的概念。

# 本书面向的对象

这本书是为 Kotlin 用户（程序员、工程师、库作者和架构师）而写的，他们已经对 Kotlin 有基本的了解，并希望了解函数式编程的基本理念以及如何在实践中使用它（如果你对 Kotlin 完全陌生，我们的附录《Kotlin 快速入门》将为你提供语言快速入门）。

# 本书涵盖的内容

第一章，*Kotlin - 数据类型、对象和类*，介绍了 Kotlin 中的面向对象编程。Kotlin 主要是一种面向对象编程语言，我们将使用这些特性来引入函数式编程风格。

第二章，*开始使用函数式编程*，涵盖了使用 Kotlin 的面向对象编程特性来介绍函数式编程的基本原则。

第三章，*不可变性 - 它很重要*，强调不可变性是函数式编程中最重要概念之一。本章将深入讲解不可变性。

第四章，*函数、函数类型和副作用*，介绍了围绕函数、纯函数以及各种函数类型和副作用的函数式编程基本概念。

第五章，*关于函数的更多内容*，讨论了 Kotlin 在函数式编程方面的特性，例如扩展函数、操作符重载、DSL 和核心递归。

第六章，*Kotlin 中的委托*，介绍了 Kotlin 如何在语言级别支持委托。尽管委托是一种面向对象编程概念，但它们可以帮助使你的代码更加模块化。

第七章，*使用协程进行异步编程*，介绍了 Kotlin 中的异步编程，并比较了协程与其他不同风格的对比。

第八章，*Kotlin 中的集合和数据操作*，介绍了 Kotlin 的增强集合 API 和 Kotlin 集合框架提供的功能接口。

第九章，*函数式编程和响应式编程*，展示了如何将函数式编程与其他编程范式结合以获得最佳效果。本章讨论了您如何将函数式编程与面向对象编程和响应式编程结合使用。

第十章，*函子、应用和单子*，为您介绍了类型化函数式编程及其基本概念。它还讨论了如何在 Kotlin 中实现它。

第十一章，*在 Kotlin 中使用流*，为您介绍了 Kotlin 中的 Streams API。

第十二章，*使用 Arrow 入门*，介绍了如何使用 Arrow 及其扩展进行函数式编程、函数组合、柯里化、部分应用、记忆化和光学。

第十三章，*箭头类型*，帮助您理解箭头数据类型，如 Option、Either、Try 和 State 及其类型类、函子和单子。

附录，*Kotlin 快速入门*，提供了您开始编写 Kotlin 代码所需的一切，例如工具、基本语法结构以及其他资源，以帮助您在 Kotlin 之旅中进步。

# 要充分利用这本书

运行和编写 Kotlin 程序唯一推荐的软件是 IntelliJ IDEA（还有其他方法可以做到这一点，我们将在附录 *Kotlin 快速入门* 中介绍）。

您可以从 [`www.jetbrains.com/idea/download/`](https://www.jetbrains.com/idea/download/) 下载 IntelliJ IDEA。

您可以在 Windows、Mac 和 Linux 上安装 IntelliJ IDEA：

+   **对于 Windows**：您可以使用从 XP 到 10 的任何 Windows 版本。要在 Windows 上安装它，运行安装程序可执行文件并遵循说明。

+   **对于 Mac**：您可以使用从 10.8 开始的任何 macOS 版本。要在 macOS 上安装它，挂载磁盘映像文件并将 *IntelliJ IDEA.app**60* 复制到您的 *Application* 文件夹。

+   **对于 Linux**：您可以使用任何 GNOME 或 KDE 桌面。要在 Linux 上安装它，使用 *tar -xzf idea-*.tar.gz* 命令解压缩 *tar.gz* 文件，然后从 *bin* 子目录中运行 *idea.sh*。

# 下载示例代码文件

您可以从 [www.packtpub.com](http://www.packtpub.com) 的账户下载这本书的示例代码文件。如果您在其他地方购买了这本书，您可以访问 [www.packtpub.com/support](http://www.packtpub.com/support) 并注册，以便将文件直接通过电子邮件发送给您。

您可以通过以下步骤下载代码文件：

1.  在 [www.packtpub.com](http://www.packtpub.com/support) 上登录或注册。

1.  选择“支持”选项卡。

1.  点击“代码下载与勘误”。

1.  在搜索框中输入书籍名称，并遵循屏幕上的说明。

文件下载后，请确保使用最新版本解压缩或提取文件夹：

+   WinRAR/7-Zip for Windows

+   Zipeg/iZip/UnRarX for Mac

+   7-Zip/PeaZip for Linux

本书代码包也托管在 GitHub 上，地址为 [`github.com/PacktPublishing/Functional-Kotlin`](https://github.com/PacktPublishing/Functional-Kotlin)。如果代码有更新，它将在现有的 GitHub 仓库中更新。

我们还有其他来自我们丰富图书和视频目录的代码包可供在 **[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)** 上获取。去看看吧！

# 下载彩色图像

我们还提供了一份包含本书中使用的截图/图表彩色图像的 PDF 文件。您可以从这里下载： [`www.packtpub.com/sites/default/files/downloads/FunctionalKotlin_ColorImages.pdf`](https://www.packtpub.com/sites/default/files/downloads/FunctionalKotlin_ColorImages.pdf)。

# 使用约定

本书使用了多种文本约定。

`CodeInText`: 表示文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 昵称。以下是一个示例：“我们引入了一个新的`BakeryGood`类，它具有`Cupcake`和`Biscuit`类共享的行为和状态，并且我们让这两个类都扩展了`BakeryGood`。”

代码块设置如下：

```kt
open class BakeryGood(val flavour: String) { 
  fun eat(): String { 
    return "nom, nom, nom... delicious $flavour bakery good" 
  } 
} 

class Cupcake(flavour: String): BakeryGood(flavour) 
class Biscuit(flavour: String): BakeryGood(flavour)
```

当我们希望您注意代码块中的特定部分时，相关的行或项目将以粗体显示：

```kt
fun main(args: Array<String>) { 
 val emptyList1 = listOf<Any>() val emptyList2 = emptyList<Any>() 

    println("emptyList1.size = ${emptyList1.size}") 
    println("emptyList2.size = ${emptyList2.size}") 
} 
```

任何命令行输入或输出都按以下方式编写：

```kt
$ kotlin HelloKt
```

**粗体**: 表示新术语、重要单词或您在屏幕上看到的单词。例如，菜单或对话框中的单词在文本中显示如下。以下是一个示例：“将出现一个对话框，询问您是否想将其打开为文件或项目。点击 Open As Project”

警告或重要注意事项如下所示。

技巧和技巧如下所示。

# 联系我们

我们始终欢迎读者的反馈。

**一般反馈**: 请通过电子邮件 `feedback@packtpub.com` 发送反馈，并在邮件主题中提及书籍标题。如果您对本书的任何方面有疑问，请通过电子邮件 `questions@packtpub.com` 联系我们。

**勘误**: 尽管我们已经尽最大努力确保内容的准确性，但错误仍然可能发生。如果您在这本书中发现了错误，如果您能向我们报告，我们将不胜感激。请访问 [www.packtpub.com/submit-errata](http://www.packtpub.com/submit-errata)，选择您的书籍，点击勘误提交表单链接，并输入详细信息。

**盗版**: 如果您在互联网上以任何形式发现我们作品的非法副本，如果您能提供位置地址或网站名称，我们将不胜感激。请通过电子邮件 `copyright@packtpub.com` 与我们联系，并附上材料的链接。

**如果您有兴趣成为作者**：如果您在某个领域有专业知识，并且您有兴趣撰写或为书籍做出贡献，请访问 [authors.packtpub.com](http://authors.packtpub.com/).

# 评论

请留下评论。一旦您阅读并使用了这本书，为何不在您购买它的网站上留下评论呢？潜在读者可以查看并使用您的客观意见来做出购买决定，我们 Packt 可以了解您对我们产品的看法，我们的作者也可以看到他们对书籍的反馈。谢谢！

关于 Packt 的更多信息，请访问 [packtpub.com](https://www.packtpub.com/)
