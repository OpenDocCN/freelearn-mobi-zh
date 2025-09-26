# 第十三章。部署到其他平台

继续讨论多平台开发，我们应该花些时间分析 Moai SDK 支持的其他平台。

Moai SDK 为不同平台提供了示例宿主。建议您阅读并理解平台宿主的代码，这样您就可以修改它们。如果您想添加自己的广告库或与下一代社交网络的集成，您需要修改宿主。

让我们来看看 Moai SDK 发布中包含的宿主。

# Windows

为了为 Windows 构建，您需要使用 Visual Studio 项目中的一个。您可以在`moai-sdk/hosts/folder`（`vs2010`和`vs2008`）中找到这两个项目。

打开适用于您 Visual Studio 版本的正确解决方案。在里面，您将找到所有需要的**GlutHost**代码，它应该在 Windows 上运行顺畅。Moai SDK 已经将此宿主的编译版本包含在`moai-sdk/bin/win32/`中。

# Mac OS X

Mac OS X 的宿主可以使用`moai-sdk/hosts/xcode/osx`中的 Xcode 项目进行编译。

该宿主的代码库基本上与 Windows 版本相同；它使用 GlutHost 程序，并作了一些调整以使其在 Mac OS X 上运行。

# Android

Android 宿主是通过位于`moai-sdk/hosts/android/run-host.sh`（或`.bat`）的 shell 脚本生成的。请确保已安装 Apache Ant 以及所有 Android 相关软件。有关在 Android 设备上构建游戏的更多信息，请参阅维基页面([`getmoai.com/wiki/index.php?title=Building_Moai_Games_For_Android_Devices`](http://getmoai.com/wiki/index.php?title=Building_Moai_Games_For_Android_Devices))。

### 注意

Android 的编译可能需要单独的一章。除了在 Moai SDK 维基页面中提到的链接外，您还需要了解一些 Android 开发知识，因此建议在尝试构建宿主之前先阅读 Android 的文档([`developer.android.com/develop/index.html`](http://developer.android.com/develop/index.html))。

# Google Chrome (本地客户端)

本地客户端([`developers.google.com/native-client/`](https://developers.google.com/native-client/))是一种新的方式，可以将代码引入到计算机中，并且可以从您的 Chrome 浏览器中下载。Moai SDK 附带了一个为它构建的宿主，可以在`moai-sdk/hosts/chrome`中找到。

您需要更改`manifest.json`文件，并将所有代码放在同一个文件夹中。

有关如何进行此操作的详细说明位于 Moai 的维基链接([`getmoai.com/wiki/index.php?title=Building_Moai_Games_for_Google_Chrome`](http://getmoai.com/wiki/index.php?title=Building_Moai_Games_for_Google_Chrome))。

# Linux

在撰写本书时，Linux 不是 Moai SDK 发布中的目标平台，但 Moai 可以从源代码构建上运行。

目前主要有两种方法来做这件事：

+   使用 Moai SDK 的 GitHub 仓库中的 Linux 分支，如[`franciscotufro.com/2013/01/compiling-moai-on-linux/`](http://franciscotufro.com/2013/01/compiling-moai-on-linux/)中所述

+   如[`github.com/spacepluk/moai-dev`](https://github.com/spacepluk/moai-dev)中所述使用 spacepluk 的分支

目前第二种方法更受欢迎，因为它支持 Untz，但官方的 Linux 支持正在路上，所以请务必检查最新的 Moai SDK 发布版本，以确认 Linux 尚未官方支持。

### 注意

**在这里插入下一个平台**

我们之前提到过，Moai SDK 是以一种易于移植到支持 C++和 OpenGL 的任何平台的方式进行开发的。如果你能够让它在新平台上运行，你可以分享你的经验并向[`github.com/moai/moai-dev`](https://github.com/moai/moai-dev)发送 pull request。我们将非常感激！

# 摘要

在本章中，我们总结了目前由 Moai SDK 支持的不同的平台。我们指出了获取有关为这些平台构建主机信息的地方。

你怎么说？你准备好创造下一个大游戏了吗？我打赌你一定准备好了！继续前进，创造史上最佳游戏，别忘了告诉社区关于它的事情！

加入[`getmoai.com/forums/`](http://getmoai.com/forums/)的论坛，成为这个令人惊叹的游戏开发社区的一部分。我们期待着你的加入。

感谢你阅读这本书并支持开源技术。希望不久后能再次见到你！
