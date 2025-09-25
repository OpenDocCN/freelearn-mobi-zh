# 前言

Kotlin 协程和流允许开发者使用简单、现代且可测试的代码在 Android 中进行异步编程。

这本书侧重于通过实践学习协程和流。您将从异步编程的基础开始，包括对协程和流的概述，同时将它们集成到您的 Android 项目中。您将了解如何管理取消和异常，然后探索如何测试您的协程和流。

在本书结束时，您将能够使用 Kotlin 协程和流来简化 Android 中的异步编程。

# 本书面向对象

这本书是为想要使用协程和流构建高质量应用并提升 Android 开发技能的 Android 开发者而写的。对 Android 开发和 Kotlin 有基本知识的初学者也会发现这本书很有用。

# 本书涵盖内容

*第一章*, *Android 异步编程简介*，探讨了 Android 中的异步编程，并展示了目前所采用的各种方式。在结尾部分，将介绍新的推荐方式——协程和流。

*第二章*, *理解 Kotlin 协程*，介绍了 Kotlin 协程并展示了它们如何在 Android 中用于异步编程。它将演示如何创建协程，并讨论协程构建器、作用域、调度器、上下文和作业。

*第三章*, *处理协程取消和异常*，讨论了协程取消以及如何正确管理协程取消、超时和异常。

*第四章*, *测试 Kotlin 协程*，描述了如何在 Android 中测试 Kotlin 协程。

*第五章*, *使用 Kotlin 流*，涵盖了 Kotlin 流的基本知识以及它们如何在 Android 中用于异步编程。它继续介绍使用流构建器创建流。还将讨论流操作符、缓冲和组合流，以及 StateFlow 和 SharedFlow。

*第六章*, *处理流程取消和异常*，探讨了如何在您的流程中管理取消、完成和异常。

*第七章*, *测试 Kotlin 流*，提供了如何在您的 Android 项目中测试流的详细信息。

# 为了最大限度地利用本书

您需要具备基本的 Android 开发技能和 Kotlin 使用知识。

您需要拥有最新版本的 Android Studio。您可以在 [`developer.android.com/studio`](https://developer.android.com/studio) 下载最新版本。为了获得最佳体验，以下规格是推荐的：

+   英特尔酷睿 i5 或更高性能的处理器

+   至少 4 GB RAM

+   至少 4 GB 可用空间

![前言表格](img/Preface_Table.jpg)

**如果你使用的是本书的数字版，我们建议你亲自输入代码或从本书的 GitHub 仓库（下一节中提供链接）获取代码。这样做将有助于你避免与代码复制和粘贴相关的任何潜在错误。**

# 下载示例代码文件

你可以从 GitHub 下载本书的示例代码文件[`github.com/PacktPublishing/Simplifying-Android-Development-with-Coroutines-and-Flows/`](https://github.com/PacktPublishing/Simplifying-Android-Development-with-Coroutines-and-Flows/)。如果代码有更新，它将在 GitHub 仓库中更新。

我们还有其他来自我们丰富图书和视频目录的代码包，可在[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)找到。查看它们吧！

# 使用的约定

本书使用了多种文本约定。

`文本中的代码`: 表示文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 用户名。以下是一个示例：“`runOnUIThread`将在主 UI 线程上执行`displayText(text)`函数。”

代码块设置如下：

```kt
lifecycleScope.launch(Dispatchers.IO) {
```

```kt
     val fetchedText = fetchText()
```

```kt
     withContext(Dispatchers.Main) {
```

```kt
           displayText(fetchedText)
```

```kt
    }
```

```kt
}
```

当我们希望引起你对代码块中特定部分的注意时，相关的行或项目将以粗体显示：

```kt
private fun fetchTextWithThread() {
```

```kt
     Thread {
```

```kt
          // get text from network
```

```kt
          val text = getTextFromNetwork()
```

```kt
           runOnUiThread {
```

```kt
 // Display on UI
```

```kt
                displayText(text)
```

```kt
 }
```

```kt
      }.start()
```

```kt
}
```

任何命令行输入或输出都按以下方式编写：

```kt
java.lang.IllegalStateException: Module with the Main dispatcher had failed to initialize. For tests Dispatchers.setMain from kotlinx-coroutines-test module can be used
```

**粗体**: 表示新术语、重要单词或屏幕上看到的单词。例如，菜单或对话框中的单词以**粗体**显示。以下是一个示例：“在 Android Studio 中，**编辑器**窗口使用行号旁边的横幅图标标识你的代码中的挂起函数调用。”

小贴士或重要提示

看起来是这样的。

# 联系我们

我们始终欢迎读者的反馈。

**一般反馈**: 如果你对此书任何方面有疑问，请通过 mailto:customercare@packtpub.com 给我们发邮件，并在邮件主题中提及书名。

**勘误**: 尽管我们已经尽一切努力确保内容的准确性，但错误仍然可能发生。如果你在这本书中发现了错误，我们将不胜感激，如果你能向我们报告这一点。请访问[www.packtpub.com/support/errata](http://www.packtpub.com/support/errata)并填写表格。

**盗版**: 如果你在网上以任何形式发现我们作品的非法副本，我们将不胜感激，如果你能提供位置地址或网站名称，我们将不胜感激。请通过 mailto:copyright@packt.com 与我们联系，并提供材料的链接。

**如果你有兴趣成为作者**：如果你在某个领域有专业知识，并且你感兴趣的是撰写或为书籍做出贡献，请访问[authors.packtpub.com](http://authors.packtpub.com)

# 分享你的想法

一旦您阅读了《使用协程和流简化 Android 开发》，我们非常期待听到您的想法！请[点击此处直接访问亚马逊评论页面](https://packt.link/r/1801816247)并分享您的反馈。

您的评论对我们和科技社区都非常重要，它将帮助我们确保我们提供高质量的内容。
