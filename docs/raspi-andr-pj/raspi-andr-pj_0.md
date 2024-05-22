# 前言

制造者社区中最受欢迎的小工具*Raspberry Pi*和最受欢迎的智能手机操作系统*Android*在本书中结合起来，产生了令人兴奋、有用且易于跟随的项目。所涵盖的项目在您日常与 Pi 的互动中非常有用，并且可以作为更令人惊奇的项目的构建模块。

# 本书涵盖内容

第一章，*从任何地方远程连接到 Pi 的远程桌面*，教会您如何进行初始设置，以便从世界上任何地方的 Android 设备远程连接到 Pi 桌面。

第二章，*使用 Pi 进行服务器管理*，在前一章的基础上管理 Pi 和我们在其上安装的不同服务器。我们甚至会介绍一个有趣、有用的项目，利用这些服务器。

第三章，*从 Pi 直播监控摄像头*，向您展示了如何将 Pi 变成网络摄像头，然后介绍了如何将其用于监控模式的技术，通过 Android 设备和互联网可访问。

第四章，*将 Pi 变成媒体中心*，向您展示如何将 Pi 变成可从 Android 设备控制的媒体中心。

第五章，*使用 Pi 的未接来电*，介绍了通过蓝牙从 Android 访问 Pi 上的传感器和组件所需的技术，并展示了 Pi 如何通知您手机上接收到的来电。

第六章，*车载 Pi*，帮助您将 Pi 连接到您的汽车，并从 Android 手机上进行跟踪。

# 您需要为本书准备什么

本书中使用的所有软件都可以在互联网上免费获得。您需要 Raspberry Pi 2 和一个 Android 设备。在一些章节中，我们甚至会使用 USB 无线网卡、DHT11 或 DHT22 温度传感器、跳线、LED 灯、USB 蓝牙适配器、Pi 摄像头、USB GPS 接收器和 OBD 蓝牙接口，所有这些都可以在在线商店购买。

# 本书的读者

*Raspberry Pi Android Projects*的目标是想要通过 Android 手机控制 Pi 创建引人入胜和有用的项目的读者。不需要先前对 Pi 或 Android 的知识。在每章的末尾，您将成功地创建一个可以日常使用的项目，并将具备能够帮助您将来开发更令人兴奋的项目的技能。本书涵盖的项目将包含一些较小的编程步骤，即使对于最没有经验的读者，这些步骤也将被详细描述。

# 约定

在本书中，您将找到一些区分不同信息类型的文本样式。以下是一些样式的示例及其含义的解释。

文本中的代码词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 用户名显示如下："下一步是安装一个名为`x11vnc`的组件。"

一段代码设置如下：

```kt
network={
    ssid="THE ID OF THE NETWORK YOU WANT TO CONNECT"
    psk="PASSWORD OF YOUR WIFI"
}
```

任何命令行输入或输出都以以下方式书写：

```kt
sudo apt-get install apache2
sudo apt-get install php5 libapache2-mod-php5
sudo apt-get install libapache2-mod-auth-mysql php5-mysql

```

**新术语**和**重要词汇**以粗体显示。您在屏幕上看到的词语，例如菜单或对话框中的词语，会以这样的方式出现在文本中："通过按下**连接**按钮来建立连接，现在您应该能够在您的 Android 设备上看到 Pi 桌面了。"

### 注意

警告或重要提示会以这样的方式出现在一个框中。

### 提示

提示和技巧会以这样的方式出现。

# 读者反馈

我们始终欢迎读者的反馈。让我们知道您对本书的看法——您喜欢或不喜欢什么。读者的反馈对我们很重要，因为它帮助我们开发您真正能充分利用的标题。

要向我们发送一般反馈，只需发送电子邮件至 `<feedback@packtpub.com>`，并在消息主题中提及书名。

如果您在某个主题上有专业知识，并且有兴趣撰写或为一本书做出贡献，请参阅我们的作者指南[www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

既然您是 Packt 书籍的自豪所有者，我们有一些事情可以帮助您充分利用您的购买。

## 下载示例代码

您可以从[`www.packtpub.com`](http://www.packtpub.com)的帐户中下载您购买的所有 Packt Publishing 书籍的示例代码文件。如果您在其他地方购买了本书，您可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)并注册，以便文件直接通过电子邮件发送给您。您还可以从[`github.com/kurtng/Raspberry-Pi-Android-Projects`](https://github.com/kurtng/Raspberry-Pi-Android-Projects)下载本书的代码包。

## 下载本书的彩色图片

我们还为您提供了一个 PDF 文件，其中包含本书中使用的屏幕截图/图表的彩色图片。彩色图片将帮助您更好地理解输出中的变化。您可以从[`www.packtpub.com/sites/default/files/downloads/Raspberry_Pi_Android_Projects_ColoredImages.pdf`](https://www.packtpub.com/sites/default/files/downloads/Raspberry_Pi_Android_Projects_ColoredImages.pdf)下载此文件。

## 勘误

尽管我们已经尽一切努力确保内容的准确性，但错误确实会发生。如果您在我们的书中发现错误——可能是文本或代码中的错误——我们将不胜感激地请您向我们报告。通过这样做，您可以帮助其他读者避免挫折，并帮助我们改进本书的后续版本。如果您发现任何勘误，请访问[`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)，选择您的书，点击**勘误提交表**链接，并输入您的勘误详情。一旦您的勘误被验证，您的提交将被接受，并且勘误将被上传到我们的网站或添加到该标题的勘误部分下的任何现有勘误列表中。

要查看先前提交的勘误表，请访问[`www.packtpub.com/books/content/support`](https://www.packtpub.com/books/content/support)，并在搜索框中输入书名。所需信息将显示在**勘误表**部分下。

## 盗版

互联网上侵犯版权材料的盗版是所有媒体的持续问题。在 Packt，我们非常重视保护我们的版权和许可。如果您在互联网上发现我们作品的任何非法副本，请立即向我们提供位置地址或网站名称，以便我们采取补救措施。

请通过 `<copyright@packtpub.com>` 与我们联系，并附上涉嫌盗版材料的链接。

感谢您帮助我们保护作者和为您提供有价值的内容的能力。

## 问题

如果您对本书的任何方面有问题，您可以通过 `<questions@packtpub.com>` 与我们联系，我们将尽力解决问题。
