# 前言

Android Wear 2.0 是针对可穿戴智能设备的一个强大平台。从 Wear 发布之日起，Wear 2.0 使可穿戴设备市场的新设备激活率接近 72%。谷歌正在与多个标志性品牌合作，为智能手表带来最佳用户体验。市场上新设备的穿戴硬件不断改进，展示了穿戴设备所具有的潜力。谷歌通过材料设计、独立应用程序、表盘创新等，引入了一种全新的体验可穿戴技术的方式。

穿戴平台越来越受欢迎，Android 开发者可以提高他们为穿戴设备编程的能力，从而获益。

这本书帮助创建五个穿戴设备应用程序，并配有详尽的解释。我们从探索穿戴设备特定的用户界面组件开始，制作一个可穿戴的笔记应用程序，并构建一个可以在地图上保存快速便签的穿戴地图应用程序。我们还将构建一个带有配套移动应用程序的完整聊天应用程序。我们还将开发一个健康和健身应用程序，用于监测脉搏速率，提醒喝水等，并编写一个数字手表。我们通过探索 Wear 2.0 平台的能力来完成本书。

在构建出色的穿戴应用程序中享受乐趣。

# 本书涵盖的内容

第一章，*让你准备起飞 - 设置你的开发环境*，教你编写第一个穿戴应用程序，探索特定于穿戴应用程序的基本 UI 组件，并讨论 Android Wear 设计原则。

第二章，*让我们帮助你捕捉思维 - WearRecyclerView 及更多*，涵盖了`WearableRecyclerView`和`WearableRecyclerView`适配器，以及`SharedPreferences`、`BoxInsetLayout`和动画`DelayedConfirmation`。

第三章，*让我们帮助你捕捉思维 - 保存数据和定制 UI*，探讨了 Realm 数据库的集成、自定义字体、UI 更新以及项目的最终确定。

第四章，*测量你的健康 - 传感器*，展示了传感器的准确性、电池消耗、Wear 2.0 休眠模式、材料设计等。

第五章，*测量你的健康 - 同步收集的传感器数据*，专注于同步收集的传感器数据，从穿戴设备收集传感器数据，处理接收到的数据以计算卡路里和距离，从移动应用程序向穿戴应用程序发送数据，Realm 数据库集成，`WearableRecyclerView`和`CardView`。

第六章，*无处不在的方法 - WearMap 和 GoogleAPIclient*，解释了开发者 API 控制台；地图 API 密钥；以及 SHA1 指纹，SQlite 集成，Google Maps，Google API 客户端和 Geocoder。

第七章，*无处不在的方法 - UI 控件及更多*，探讨了理解 UI 控件、标记控件、地图缩放控件、Wear 中的 StreetView 以及最佳实践。

第八章，*智能聊天方式 - 消息传递 API 及更多*，讨论了为移动应用程序配置 Firebase，创建用户界面，理解消息传递 API，使用 Google API 客户端以及构建 Wear 模块。

第九章，*智能聊天方式 - 通知及更多*，涵盖了 Firebase 功能、通知、材质设计 Wear 应用程序 Wear 2.0 输入法框架等。

第十章，*仅为你的时间 - WatchFace 和服务*，概述了`CanvasWatchFaceService`和注册一个手表表盘，`CanvasWatchFaceService.Engine`和回调，手表表盘元素及其初始化编写手表表盘，以及处理手势和点击事件。

第十一章，*关于 Wear 2.0 的更多信息*，探讨了独立应用程序，曲线布局和更多 UI 组件的 Complications API，不同的导航和动作，手腕手势，输入法框架，以及将 Wear 应用程序分发到 Play 商店。

# 阅读本书所需的条件

要能够跟随本书，你需要一台安装了最新版 Android Studio 的计算机。你需要互联网来设置 Wear 开发所需的所有 SDK。如果你有一个 Wear 设备来测试应用程序，那将很好；否则，Android Wear 模拟器将完成这项工作。

# 本书的目标读者

本书面向已经对编程和 Android 应用开发有深入了解的 Android 开发者。本书帮助读者从中级开发者进阶为专家级 Android 开发者，通过增加 Wear 开发技能来丰富他们的知识。

# 约定

在本书中，你将发现多种文本样式，用于区分不同类型的信息。以下是一些样式示例及其含义的解释。

界面中特定的命令或工具将如下标识：

选择“保存”按钮。

文本中的代码字、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 处理程序将如下显示："我们可以通过使用`include`指令包含其他上下文。"

代码块按照以下方式设置：

```java
compile 'com.google.android.support:wearable:2.0.0' compile 'com.google.android.gms:play-services-wearable:10.0.1' provided 'com.google.android.wearable:wearable:2.0.0'

```

当我们希望您注意代码块中的特定部分时，相关的行或项目会以粗体显示：

```java
<?xml version="1.0" encoding="utf-8"?> <android.support.wearable.view.BoxInsetLayout xmlns:android="http://schemas.android.com/apk/res/android"  xmlns:app="http://schemas.android.com/apk/res-auto"  xmlns:tools="http://schemas.android.com/tools"  android:id="@+id/container"  android:layout_width="match_parent"  android:layout_height="match_parent"  tools:context="com.ashok.packt.wear_note_1.MainActivity"  tools:deviceIds="wear"> </android.support.wearable.view.BoxInsetLayout>

```

任何命令行输入或输出都如下所示：

```java
adb connect 192.168.1.100

```

新术语和重要词汇会以粗体显示。您在屏幕上看到的词，例如菜单或对话框中的，会像这样出现在文本中："让默认选择的模板为始终开启的穿戴应用代码存根 Always On Wear Activity。"

警告或重要提示会像这样出现。

提示和技巧会像这样出现。

# 读者反馈

我们一直欢迎读者的反馈。请告诉我们您对这本书的看法——您喜欢或不喜欢的地方。读者的反馈对我们很重要，因为它能帮助我们开发出您真正能从中获益的图书。要给我们发送一般性反馈，只需发送电子邮件至`feedback@packtpub.com`，并在邮件的主题中提及书籍的标题。如果您在某个主题上有专业知识，并且有兴趣撰写或参与书籍编写，请查看我们的作者指南：[www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

既然您已经拥有了 Packt 的一本书，我们有很多方法可以帮助您充分利用您的购买。

# 下载示例代码

您可以从您的账户[`www.packtpub.com`](http://www.packtpub.com)下载本书的示例代码文件。如果您在其他地方购买了这本书，可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)注册，我们会直接将文件通过电子邮件发送给您。您可以按照以下步骤下载代码文件：

1.  使用您的电子邮件地址和密码登录或注册我们的网站。

1.  将鼠标悬停在顶部的“支持”标签上。

1.  点击“代码下载与勘误”。

1.  在搜索框中输入书籍的名称。

1.  选择您要下载代码文件的书。

1.  从下拉菜单中选择您购买本书的地方。

1.  点击“代码下载”。

下载文件后，请确保您使用最新版本的以下软件解压或提取文件夹：

+   对于 Windows 用户，可以使用 WinRAR / 7-Zip。

+   对于 Mac 用户，可以使用 Zipeg / iZip / UnRarX。

+   对于 Linux 用户，可以使用 7-Zip / PeaZip。

本书的代码包也托管在 GitHub 上：[`github.com/PacktPublishing/Android-Wear-Projects`](https://github.com/PacktPublishing/Android-Wear-Projects)。我们还有其他丰富的书籍和视频代码包，可以在[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)查看。请查看！

# 勘误

尽管我们已经尽力确保内容的准确性，但错误仍然会发生。如果您在我们的书中发现了一个错误——可能是文本或代码中的错误——如果您能报告给我们，我们将不胜感激。这样做，您可以避免其他读者感到沮丧，并帮助我们改进本书后续版本。如果您发现任何勘误信息，请通过访问[`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)，选择您的书籍，点击勘误提交表单链接，并输入您的勘误详情。一旦您的勘误信息被验证，您的提交将被接受，勘误信息将被上传到我们的网站或添加到该标题勘误部分下现有的勘误列表中。要查看之前提交的勘误信息，请前往[`www.packtpub.com/books/content/support`](https://www.packtpub.com/books/content/support)，并在搜索字段中输入书名。所需信息将在勘误部分下显示。

# 盗版

互联网上版权材料的盗版问题在所有媒体中持续存在。在 Packt，我们非常重视保护我们的版权和许可。如果您在任何形式的互联网上发现我们作品非法副本，请立即提供位置地址或网站名称，以便我们可以寻求补救措施。请通过 `copyright@packtpub.com` 联系我们，并提供疑似盗版材料的链接。我们感谢您帮助保护我们的作者和我们提供有价值内容的能力。

# 问题

如果您对本书的任何方面有问题，可以通过 `questions@packtpub.com` 联系我们，我们将尽力解决问题。
