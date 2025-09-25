# 前言

Unity 从其谦逊的起点已经走了很长的路；然而，在过去几年中，它几乎已经成为一个行业通用的工具，几乎所有独立游戏开发者都使用它来开发他们的游戏。它非常易于使用，可以快速开发原型，而且当你有一个成功的想法时，它足够灵活，可以将小型原型扩展成一个完整的游戏。

尽管 Unity 具有专业能力，但它使用起来非常简单，即使是完全的新手也能在几小时内开发出一个基本游戏。而且，只要稍加努力，就可以开发出一个非常精致的游戏，具有出色的光照和动画。

使用当前版本，Unity 使游戏开发对每个人来说都更加容易接近。

在这本书中，我们将涵盖 2D 和 3D 游戏开发的各种主题。我们将看到如何在 2D 中开发类似喷射背包冒险游戏的游戏，以及一个带有完整 3D 动画、光照和相机的 3D 战斗游戏。我们还将看到如何添加按钮、文本和屏幕过渡。最后，一旦我们创建了游戏，我们将看到如何通过添加应用内购买和广告来实现盈利。

# 本书涵盖的内容

第一章，*使用 Unity3D 介绍 Android 游戏开发*，涵盖了 Android 游戏开发的基本概念、Android 游戏简史、Unity3D 中 Android 游戏的基本构建块以及游戏的基本流程。

第二章，*完成活泼企鹅 2D 游戏*，通过完成喷射背包冒险游戏克隆游戏来扩展 2D 游戏开发。本章介绍了各种主题，如粒子系统、相机管理、预制体、动画、触发器、碰撞器和基本的 GUI 系统。

第三章，*动作战斗游戏玩家角色*，涵盖了 3D 动作战斗游戏的基本设置，包括导入模型和纹理、为角色设置绑定、在模型上应用动画，以及使用虚拟屏幕摇杆控制玩家角色。

第四章，*具有 AI 的敌人角色*，涵盖了从导入模型到应用动画再到使用 AI 进行决策的游戏敌人模型创建方面。

第五章，*游戏玩法、UI 和效果*，展示了如何完成游戏玩法循环，添加用户界面。

为游戏添加计分文本，并添加游戏粒子效果。

第六章，*游戏场景和场景流*，涵盖了创建主菜单场景，解释选项场景，并演示如何在游戏中切换场景。

第七章，*游戏统计、社交、IAP 和广告集成*，演示了如何保存游戏进度，添加社交媒体集成，如 Facebook 和 Twitter，以及广告集成。

并通过应用内购买来增加盈利。

第八章，*声音、收尾工作和发布*，让我们为游戏添加收尾工作和声音。我们将了解如何在设备上运行游戏并将游戏发布到 Android Play 商店。

# 阅读本书所需的条件

您需要最新版本的 Unity，您可以从他们的网站下载，并需要一个能够运行 Unity 的计算机。需要具备基本的 C# 知识，因为本书中的代码是用 C# 编写的。尽管游戏可以在 Android 模拟器上运行，但要看到游戏的实际性能，需要一个 Android 设备。

# 本书面向的对象

本书面向希望扩展 Unity3D 知识并创建高端、复杂 Android 游戏的初学者或中级 Unity3D 开发者。您应具备基本的或中级 Unity3D 理解，包括其环境、基本概念如游戏对象和预制体，以及使用 Unity 脚本。

C# 或 JavaScript，以及如何使用 Unity3D 开发基本的 2D/3D 游戏。

本书对那些为 Android 开发过基本/简单游戏的 Unity 开发者非常有用，他们想了解高端复杂游戏（如详细动画、多个关卡、角色能力、敌人弱点、智能 AI、成就、排行榜等）的细节和核心组件。

# 惯例

在这本书中，你会发现许多不同的文本样式，用于区分不同类型的信息。以下是一些这些样式的示例及其含义的解释。

文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 处理方式如下所示：“The”

`bIsDefending` 变量与我们定义的动画控制器参数相同。

代码块以如下方式设置：

```kt
    private Animator anim;
    // Use this for initialization

    void Start () {
        anim = GetComponent<Animator>();
    } // start

```

当我们希望将您的注意力引向代码块中的某个特定部分时，相关的行或项目将以粗体显示：

```kt
    if (pAnim.GetBool("tIsPunching")){
        if (anim.GetBool("bEnemyIsDefending") == false) {
            Debug.Log("enemy got hit");
            anim.SetTrigger("tEnemyGotHit");
            anim.SetBool("bEnemyIsDefending", true);
 health -= pScript.damage;
        }
    }

```

任何命令行输入或输出将以如下方式书写：

```kt
C:\Program Files\Unity\Editor\Unity.exe

```

新术语和重要词汇将以粗体显示。你在屏幕上看到的单词，例如在菜单或对话框中，将以如下方式显示：“Unity 允许开发者通过检查器面板管理每个游戏对象的这些组件。”

警告或重要注意事项将以如下方式显示。

小贴士和技巧将以如下方式显示。

# 读者反馈

我们欢迎读者的反馈。请告诉我们您对本书的看法——您喜欢或不喜欢的地方。读者反馈对我们非常重要，因为它帮助我们开发出您真正能从中受益的标题。

要发送一般性反馈，请简单地发送电子邮件至 `feedback@packtpub.com`，并在邮件主题中提及本书的标题。

如果您在某个主题上具有专业知识，并且您有兴趣撰写或为本书做出贡献，请参阅我们的作者指南[www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

现在，您是 Packt 图书的骄傲拥有者，我们有一些事情可以帮助您从您的购买中获得最大收益。

# 下载示例代码

您可以从[`www.packtpub.com`](http://www.packtpub.com)的账户下载本书的示例代码文件。如果您在其他地方购买了此书，您可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)并注册，以便将文件直接通过电子邮件发送给您。

您可以通过以下步骤下载代码文件：

1.  使用您的电子邮件地址和密码登录或注册我们的网站。

1.  将鼠标指针悬停在顶部的 SUPPORT 标签上。

1.  点击代码下载与勘误。

1.  在搜索框中输入书籍名称。

1.  选择您想要下载代码文件的书籍。

1.  从下拉菜单中选择您购买此书的来源。

1.  点击代码下载。

文件下载后，请确保使用最新版本的以下软件解压缩或提取文件夹：

+   WinRAR / 7-Zip for Windows

+   Zipeg / iZip / UnRarX for Mac

+   7-Zip / PeaZip for Linux

本书代码包也托管在 GitHub 上，网址为[`github.com/PacktPublishing/Mastering-Android-Game-Development-with-Unity`](https://github.com/PacktPublishing/Mastering-Android-Game-Development-with-Unity)。我们还有其他来自我们丰富图书和视频目录的代码包可供选择，网址为[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)。请查看它们！

# 下载本书的彩色图像

我们还为您提供了一个包含本书中使用的截图/图表彩色图像的 PDF 文件。彩色图像将帮助您更好地理解输出的变化。您可以从[`www.packtpub.com/sites/default/files/downloads/MasteringAndroidGameDevelopmentwithUnity_ColorImages.pdf`](http://www.packtpub.com/sites/default/files/downloads/MasteringAndroidGameDevelopmentwithUnity_ColorImages.pdf)下载此文件。

# 勘误

尽管我们已经尽一切努力确保内容的准确性，但错误仍然可能发生。如果您在我们的书中发现错误——可能是文本或代码中的错误——如果您能向我们报告，我们将不胜感激。通过这样做，您可以节省其他读者的挫败感，并帮助我们改进本书的后续版本。如果您发现任何勘误，请通过访问[`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)，选择您的书籍，点击勘误提交表单链接，并输入您的勘误详情来报告它们。一旦您的勘误得到验证，您的提交将被接受，勘误将被上传到我们的网站或添加到该标题的勘误表中的现有勘误列表。

要查看之前提交的勘误表，请访问[`www.packtpub.com/books/content/support`](https://www.packtpub.com/books/content/support)并在搜索字段中输入书籍名称。所需信息将出现在勘误表部分。

# 侵权

互联网上对版权材料的侵权是一个持续存在的问题，涉及所有媒体。在 Packt，我们非常重视保护我们的版权和许可证。如果您在互联网上发现任何形式的非法复制我们的作品，请立即提供位置地址或网站名称，以便我们可以寻求补救措施。

请通过`copyright@packtpub.com`与我们联系，并提供疑似侵权材料的链接。

我们感谢您在保护我们作者和我们提供有价值内容的能力方面所提供的帮助。

# 询问

如果您对本书的任何方面有问题，您可以联系我们的`questions@packtpub.com`，我们将尽力解决问题。
