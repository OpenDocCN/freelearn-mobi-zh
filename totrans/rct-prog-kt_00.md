# 前言

我们的世界仅仅是状态的集合吗？不。那么，为什么所有的编程范式都将我们的世界视为一系列的状态？我们难道不能在编程中反映那些真实、运动且持续变化状态的物体吗？这些问题自从我第一次学习编程以来就一直吸引着我。

当我开始作为 Android 开发者工作时，这些问题仍然困扰着我，但也结识了一些朋友。为什么应用程序中需要这么多循环？难道没有可以替代迭代器的东西吗？此外，对于 Android 应用程序，我们必须考虑很多因素，因为移动设备的处理器和 RAM 没有像你的 PC 那么强大。如果你没有很好地组织项目，经常会遇到内存不足异常。所以，如果我们能在程序中减少迭代器的使用，用户体验将显著提升，但，我们该如何做到？我们如何替换迭代器，用什么呢？

有一天，我读了一篇关于反应式编程和 ReactiveX 框架的博客文章（很可能是托马斯·尼尔德写的，感谢他），它让我看到了所有问题的答案。于是，我开始学习反应式编程。

我发现反应式编程的学习曲线非常复杂，许多开发者都在中途放弃。在大多数地方，反应式编程被认为是一个高级话题。然而，我继续我的反应式编程学习之旅，作为耐心和一致性的回报，我得到了我的问题的答案。RxJava（以及所有其他的 ReactiveX 库）代表的是类似于我们现实世界的模型，并且，与状态不同，它们通过运动和持续变化的状态来模拟行为。与迭代器模式不同，它依赖于推送机制，即数据/事件到来时就会推送给订阅者/观察者，这使得编程变得更加容易，更接近人类世界。

另一方面，大约两年前（2015 年 12 月），当我读到一篇关于将在 JVM 上运行的新语言的 Jetbrains 博客（是的，我读了很多，也写了很多）时，我的第一个想法是，为什么需要一种新语言？于是，我开始探索 Kotlin，并爱上了它。该语言唯一的目的就是让编程变得更加容易。每当有人谈论 Kotlin 的好处时，他们都会提到轻松处理空指针异常，但还有更多优势；这个列表永远没有尽头，并且还在不断增长。

对于程序员来说，最好的事情莫过于将 Kotlin 和 ReactiveX 框架结合起来；马里奥·阿里亚斯为了开发者社区的利益，出色地完成了这项工作，并在 2013 年 10 月启动了 RxKotlin。

RxKotlin 缺少的只有文档；我个人认为，ReactiveX 库复杂的学习曲线背后的主要原因是缺乏文档，以及，很大程度上，缺乏意识。我见过很多开发者，甚至有 6-8 年以上经验的，都没有听说过反应式编程；我相信这本书将在改变这种状况中扮演更大的角色。这本书也是我使命的一部分（也是 Kotlin Kolkata 用户组的使命），尽可能广泛地传播 Kotlin 的使用和知识。

根据我的知识，这是第一本帮助你学习 Kotlin 反应式编程的书籍，涵盖了 RxKotlin（确切地说，是 RxKotlin 2.0）和 Reactor-Kotlin 框架。这是一本逐步指南，教你如何学习 RxKotlin 和 Reactor-Kotlin，并增加了对 Spring 和 Android 的覆盖。我希望这本书能帮助你发现 Kotlin 和反应式编程的全部好处，并且，借助这本书，你将能够成功地将反应式编程应用到所有的 Kotlin 项目中。

如果你有任何疑问、反馈或评论，可以通过我的网站 [`www.rivuchk.com`](http://www.rivuchk.com) 联系我，或者发送电子邮件到 rivu@rivuchk.com。请确保在电子邮件的主题中提及“Book Query - Reactive Programming in Kotlin”。

# 本书涵盖内容

第一章，*反应式编程简短介绍*，帮助你理解反应式编程的背景、思维模式和原则。

第二章，*使用 Kotlin 和 RxKotlin 进行函数式编程*，这一章将带你了解函数式编程范式的基本概念及其在 Kotlin 上的可能实现，以便你能够轻松理解函数式反应式编程。

第三章，*观察者、观察者和主题*，使你能够掌握 RxKotlin 的基础——观察者、观察者和主题是 RxKotlin 的核心。

第四章，*背压和 Flowables 简介*，介绍了 Flowables，这使你能够使用背压——RxKotlin 中一种防止生产者超过消费者速度的技术。

第五章，*异步数据算子和转换*，介绍了 RxKotlin 中的算子。

第六章，*关于算子和错误处理的更多内容*，帮助你更牢固地掌握算子，并介绍如何组合生产者以及如何使用算子过滤它们。这一章还将帮助你更有效地在 RxKotlin 中处理错误。

第七章，*使用调度器在 RxKotlin 中实现并发和并行处理*，使你能够利用调度器的优势来实现并发编程。

第八章，*测试 RxKotlin 应用程序*，带你了解应用程序开发中最关键的部分——测试，在 RxKotlin 中测试略有不同，因为反应式编程定义的是行为而不是状态。本章从测试的基础开始，使你能够从头开始学习测试。

第九章，*资源管理和扩展 RxKotlin*，帮助你学习如何在 Kotlin 中管理资源——资源可以是数据库实例、文件、HTTP 访问或任何需要关闭的东西。你还将学习如何在第九章中创建自己的自定义操作符。

第十章，*Spring for Kotlin 开发者入门 Web 编程*，让你开始使用 Spring 和 Hibernate，以便你在用 Kotlin 编写 API 时能够利用其优势。

第十一章，*使用 Spring JPA 和 Hibernate 的 REST API*，介绍了 Reactor 框架和 reactor-kotlin 扩展，这样你就可以在 Kotlin 中使用反应式编程。

第十二章，*反应式 Kotlin 和 Android*，本书的最后一章，让你开始使用 Kotlin 在 Android 中进行反应式编程。

# 你需要这本书的内容

我们将在本书的程序中使用 Java 8 和 Kotlin 1.1.50，因此需要 Oracle 的 JDK 1.8 以及 Kotlin 1.1.50（如果你使用 IntelliJ IDEA，则可以跳过下载）。你需要一个环境来编写和编译你的 Kotlin 代码（我强烈推荐 IntelliJ IDEA，但你也可以使用任何你喜欢的工具），并且最好有一个构建自动化系统，如 Gradle 或 Maven。本书后面的内容中，我们将使用 Android Studio（2.3.3 或 3.0）。本书中你需要的一切都应该免费使用，并且不需要商业或个人许可（我们使用的是 IntelliJ IDEA 社区版）。

# 这本书面向的对象

这本书是为希望构建容错、可扩展和分布式系统的 Kotlin 开发者编写的。需要具备 Kotlin 的基本知识；然而，假设没有反应式编程的先验知识。

# 术语约定

在这本书中，你会发现许多文本样式，用于区分不同类型的信息。以下是一些这些样式的示例及其含义的解释。

文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 昵称如下所示：“让我们首先看看`ReactiveCalculator`类的`init`块”

代码块设置如下：

```kt
   async(CommonPool) { 
        Observable.range(1, 10) 
          .subscribeOn(Schedulers.trampoline())//(1) 
          .subscribe { 
              runBlocking { delay(200) } 
              println("Observable1 Item Received $it") 
          } 

```

当我们希望将你的注意力引向代码块的一个特定部分时，相关的行或项目将以粗体显示：

```kt
abstract class BaseActivity : AppCompatActivity() { 
       final override fun onCreate(savedInstanceState: Bundle?) { 
         super.onCreate(savedInstanceState) 
         onCreateBaseActivity(savedInstanceState) 
        } 
 abstract fun onCreateBaseActivity(savedInstanceState: Bundle?) 
       } 
```

任何命令行输入或输出都应如下所示。输入命令可能被分成多行以提高可读性，但需要在提示符中作为一条连续的行输入：

```kt
$ git clone https://github.com/ReactiveX/RxKotlin.git
$ cd RxKotlin/
$ ./gradlew build
```

**新术语**和**重要词汇**以粗体显示。屏幕上看到的单词，例如在菜单或对话框中，在文本中显示如下：“转到 Android Studio | 设置 | 插件。”

警告或重要注意事项看起来像这样。

小贴士和技巧看起来像这样。

# 读者反馈

我们始终欢迎读者的反馈。告诉我们您对这本书的看法——您喜欢或不喜欢什么。读者反馈对我们很重要，因为它帮助我们开发出您真正能从中获得最大收益的标题。

要向我们发送一般反馈，只需发送电子邮件至 feedback@packtpub.com，并在邮件主题中提及书的标题。

如果您在某个主题上具有专业知识，并且您对撰写或为书籍做出贡献感兴趣，请参阅我们的作者指南[www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

现在您已经是 Packt 图书的骄傲拥有者，我们有一些东西可以帮助您从您的购买中获得最大收益。

# 下载示例代码

您可以从[`www.packtpub.com`](http://www.packtpub.com)的账户下载本书的示例代码文件。如果您在其他地方购买了这本书，您可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)并注册，以便将文件直接通过电子邮件发送给您。

您可以通过以下步骤下载代码文件：

1.  使用您的电子邮件地址和密码登录或注册我们的网站。

1.  将鼠标指针悬停在顶部的 SUPPORT 标签上。

1.  点击“代码下载 & 错误更正”。

1.  在搜索框中输入书的名称。

1.  选择您想要下载代码文件的书籍。

1.  从下拉菜单中选择您购买此书的来源。

1.  点击“代码下载”。

文件下载完成后，请确保您使用最新版本的以下软件解压缩或提取文件夹：

+   适用于 Windows 的 WinRAR / 7-Zip

+   适用于 Mac 的 Zipeg / iZip / UnRarX

+   适用于 Linux 的 7-Zip / PeaZip

本书代码包也托管在 GitHub 上[`github.com/PacktPublishing/Reactive-Programming-in-Kotlin`](https://github.com/PacktPublishing/Reactive-Programming-in-Kotlin)。我们还有其他来自我们丰富图书和视频目录的代码包，可在[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)找到。查看它们吧！

# 下载本书的彩色图像

我们还为您提供了一个包含本书中使用的截图/图表彩色图像的 PDF 文件。这些彩色图像将帮助您更好地理解输出的变化。您可以从[`www.packtpub.com/sites/default/files/downloads/ReactiveProgramminginKotlin_ColorImages.pdf`](https://www.packtpub.com/sites/default/files/downloads/ReactiveProgramminginKotlin_ColorImages.pdf)下载此文件。

# 勘误

尽管我们已经尽一切努力确保我们内容的准确性，但错误仍然可能发生。如果您在我们的书中发现错误——可能是文本或代码中的错误——如果您能向我们报告这一点，我们将不胜感激。通过这样做，您可以避免其他读者的挫败感，并帮助我们改进本书的后续版本。如果您发现任何勘误，请通过访问[`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)，选择您的书籍，点击“勘误提交表单”链接，并输入您的勘误详情来报告它们。一旦您的勘误得到验证，您的提交将被接受，勘误将被上传到我们的网站或添加到该标题的勘误部分下的现有勘误列表中。

要查看之前提交的勘误，请访问[`www.packtpub.com/books/content/support`](https://www.packtpub.com/books/content/support)，并在搜索字段中输入书籍名称。所需信息将出现在勘误部分下。

# 侵权

互联网上版权材料的侵权是一个跨所有媒体的持续问题。在 Packt，我们非常重视我们版权和许可证的保护。如果您在互联网上发现任何形式的非法复制我们的作品，请立即提供位置地址或网站名称，以便我们可以寻求补救措施。

请通过 copyright@packtpub.com 与我们联系，并提供涉嫌侵权材料的链接。

我们感谢您在保护我们作者和我们为您提供有价值内容的能力方面的帮助。

# 问答

如果您对本书的任何方面有问题，您可以通过 questions@packtpub.com 联系我们，我们将尽力解决问题。
