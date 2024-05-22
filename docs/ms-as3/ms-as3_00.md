# 前言

欢迎阅读《掌握 Android Studio 3》，这是对最新和最复杂的 Android 开发环境的全面指南。无论您是 IDE 的新手还是想要从其他 IDE（如 Eclipse）迁移，本书都使用实际示例来演示 Android Studio 如何促进开发的每个阶段。

本书从工作区本身的介绍开始，然后探索了 IDE 提供的各种 UI 设计工具，包括强大的可视化布局编辑器、自动约束布局工具和动画材料图标。

一旦掌握了 IDE 的设计工具，本书将继续探讨使用 Android Studio 进行代码开发以及其许多有用和创新的编程工具，例如代码完成、模板定制，以及最重要的是提供的 Android Studio 3 的出色测试和分析工具。

Android Studio 不仅是一个用于基本编码的好工具；它还提供了各种插件和扩展，支持诸如 C++和 Kotlin 等语言的本地语言支持。正是这种本地 SDK 的可扩展性使得掌握 Android Studio 3 成为任何移动开发人员的必备技能，本书详细介绍了其中最有用和最受欢迎的内容，使读者能够掌握当今最令人兴奋的开发工具之一。

# 本书内容

第一章《工作区结构》介绍了整体工作区。它涵盖了主要功能，对于那些全新于 IDE 的人来说将非常有用。

第二章《UI 设计》介绍了 UI 设计和开发的主题，着眼于布局编辑器的自动化和节省时间的功能。

第三章《UI 开发》继续使用 UI 开发工具，探讨了更复杂的布局以及如何使用打包在支持存储库中的代码库轻松实现这些布局。

第四章《设备开发》扩展了之前的工作，探讨了针对物理设备和形态因素的开发，涵盖了屏幕旋转和适用于可穿戴设备的形状感知布局等主题。

第五章《资源和资产》着眼于资源管理，特别是 Android 对材料图标和矢量资产的使用。它演示了 Android Studio 为开发的这一方面提供了很好的节省时间的功能。

第六章《模板和插件》是关于扩展 Android Studio 的两章中的第一章。在这里，我们将看到现成的和免费提供的代码样本，不仅在 IDE 中提供，还通过第三方插件提供。

第七章《语言支持》延续了前一章的主题。在这里，我们将看到如何无缝地包含 C++和 Kotlin 代码。

第八章《测试和分析》探讨了 IDE 提供的强大测试和分析工具，以及如何使用它们来测试和微调我们的工作。

第九章《打包和分发》涵盖了开发周期的最后方面。这涉及仔细研究 Gradle 并涵盖了货币化技术。

# 本书所需内容

Android Studio SDK 都是开源的，可以从[developer.android.com](https://developer.android.com/index.html)下载。

本书中提到了各种第三方插件，以及相关的下载位置。

# 本书适合对象

本书适用于任何经验水平的 Android 开发人员，他们希望迁移到或简单掌握 Android Studio 3。

# 惯例

在本书中，您会发现一些文本样式，用于区分不同类型的信息。以下是一些样式的示例及其含义的解释。文本中的代码词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 句柄显示如下："在前面的例子中，我们使用`app:srcCompat`而不是`android:src`。"

代码块设置如下：

```kt
public class ExampleUnitTest 
    { 
      @Test 
        public void addition_isCorrect() throws Exception { 
               assertEquals(4, 2 + 2); 
   } 
} 
```

当我们希望引起您对代码块的特定部分的注意时，相关行或项目将以粗体显示。

```kt
buildTypes { 
release { 
         . . .  
         } 
    } 
    productFlavors { 
        flavorDimensions "partial", "full" 
```

任何命令行输入或输出都以以下方式编写：

```kt
gradlew clean 
```

**新术语**和**重要单词**以粗体显示。您在屏幕上看到的单词，例如菜单或对话框中的单词，会以这样的方式出现在文本中："除了 MakeBuild 和 Analyze 之外，Build 菜单还有其他有用的条目，例如 Clean Project 项目，它会从构建目录中删除构建工件"

警告或重要说明会以这种方式出现。

技巧和窍门显示如下。

# 读者反馈

我们始终欢迎读者的反馈。让我们知道您对本书的看法-您喜欢或不喜欢的地方。读者的反馈对我们很重要，因为它帮助我们开发您真正能从中获益的标题。要向我们发送一般反馈，只需发送电子邮件至`feedback@packtpub.com`，并在主题中提及书名。如果您在某个专题上有专业知识，并且有兴趣撰写或为书籍做出贡献，请参阅我们的作者指南[www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

现在您是 Packt 书籍的自豪所有者，我们有一些事情可以帮助您充分利用您的购买。 

# 下载本书的彩色图像

我们还为您提供了一个 PDF 文件，其中包含本书中使用的屏幕截图/图表的彩色图像。彩色图像将帮助您更好地理解输出的变化。您可以从[`www.packtpub.com/sites/default/files/downloads/MasteringAndroidStudio3_ColorImages.pdf`](https://www.packtpub.com/sites/default/files/downloads/MasteringAndroidStudio3_ColorImages.pdf)下载此文件。

# 勘误

尽管我们已经尽一切努力确保内容的准确性，但错误是难免的。如果您在我们的书中发现错误-可能是文本或代码中的错误-我们将不胜感激，如果您能向我们报告。通过这样做，您可以帮助其他读者避免挫折，并帮助我们改进本书的后续版本。如果您发现任何勘误，请访问[`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)报告，选择您的书，点击勘误提交表格链接，并输入您的勘误详情。一旦您的勘误经过验证，您的提交将被接受，并且勘误将被上传到我们的网站或添加到该标题的勘误部分的任何现有勘误列表中。要查看以前提交的勘误，请访问[`www.packtpub.com/books/content/support`](https://www.packtpub.com/books/content/support)，并在搜索框中输入书名。所需信息将出现在勘误部分下。

# 盗版

互联网上盗版受版权保护的材料是一个持续存在的问题，涉及所有媒体。在 Packt，我们非常重视保护我们的版权和许可。如果您在互联网上发现我们作品的任何非法副本，请立即向我们提供位置地址或网站名称，以便我们采取补救措施。请通过`copyright@packtpub.com`与我们联系，并附上涉嫌盗版材料的链接。我们感谢您帮助保护我们的作者和我们为您提供有价值内容的能力。

# 问题

如果您对本书的任何方面有问题，可以通过`questions@packtpub.com`与我们联系，我们将尽力解决问题。
