# 前言

基于 Android 的设备的广泛普及引发了开发针对 Android 的软件应用（或应用）的极大兴趣。幸运的是，一个功能强大且成本低的硬件平台——BeagleBone Black，可以让你快速轻松地在真实硬件上测试你的应用。BeagleBone Black 专注于小型化以及广泛的扩展和接口机会，以非常低的价格提供了大量的处理能力。它还为应用开发者提供了曾经只有硬件黑客专家或昂贵的硬件开发套件拥有者才有的机会：编写能够与自定义硬件电路交互的 Android 应用。

无论你是硬件接口的新手还是经验丰富的专家，*针对 BeagleBone Black 的 Android*为你提供了开始创建与自定义硬件直接通信的 Android 应用所需的工具。从一开始，这本书将帮助你理解 Android 独特的硬件接口方法。你将安装和定制 Android，构建与你的 BeagleBone Black 平台接口的电路，并构建使用该硬件与外部世界通信的本地代码和 Android 应用。通过逐章顺序地工作示例，你将学会如何创建能够同时与多个硬件组件接口的多线程应用。

一旦你探索了本书中的各种示例电路和应用，你将走在成为 Android 硬件接口专家的道路上！

# 本书涵盖的内容

第一章，*Android 和 BeagleBone Black 的介绍*，将指导你完成将 Android 操作系统安装到你的 BeagleBone Black 开发板上的过程。同时，还提供了你在这本书中进行活动时需要用到的一系列硬件组件清单。

第二章，*与 Android 接口*，向你介绍了 BeagleBone Black 的硬件和 Android 硬件抽象层的多个方面。它描述了如何对你的开发环境和安装在 BeagleBone Black 上的 Android 进行一些修改，以便 Android 应用能够访问 BeagleBone Black 的各种硬件功能。

第三章，*使用 GPIO 处理输入和输出*，指导你构建你的第一个硬件接口电路，并解释了一个基本的 Android 应用与它通信的细节。这是你向构建更复杂交互的与 BeagleBone Black 外部世界交互的应用迈出的第一步。

第四章，*使用 I2C 存储和检索数据*，扩展了第三章，*使用 GPIO 处理输入和输出*的基础知识，并解释了应用程序内部如何使用异步后台线程与硬件通信。它指导你构建一个与非易失性存储芯片接口的电路，以及与该芯片交互的应用程序的实现细节。

第五章，*使用 SPI 与高速传感器接口*，探讨了如何创建执行高速接口的应用程序，使用温度和压力传感器与 BeagleBone Black 进行接口。

第六章，*创建一个完整的接口解决方案*，将之前章节关于 GPIO，I2C 和 SPI 接口的知识结合起来，创建一个单一的复杂硬件和软件解决方案，该方案使用这三种接口来响应来自外部世界的硬件事件。

第七章，*未来的方向*，描述了 BeagleBone Black 上更多可用的硬件接口，解释如何创建更永久的 Android 硬件/软件解决方案，并为你提供一些未来探索的项目想法。

# 你需要为这本书准备什么

我们在这本书中提供了假设你使用的是基于 Windows 或 Linux 的计算机的指导。如果你已经是 Android 应用开发者，你可能已经安装了所需的所有软件应用。我们期望你已经安装了 Eclipse ADT 和 Android NDK，尽管我们在第二章，*与 Android 接口*的开始部分提供了这些工具的下载链接，以防你还没有安装。 第一章，*Android 和 BeagleBone Black 介绍*，提供了实现书中使用的示例接口电路所需的各种硬件组件和设备列表。

# 本书的目标读者

如果你是一名想要开始实验 BeagleBone Black 平台硬件功能的 Android 应用开发者，那么这本书非常适合你。具备基本的电子原理知识会有所帮助，我们期望读者具备使用 Eclipse ADT 和 Android SDK 开发 Android 应用的基本知识，但不需要有先前的硬件经验。

# 编写约定

在这本书中，你会发现多种文本样式，用于区分不同类型的信息。以下是一些样式示例及其含义的解释。

文本中的代码字、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 处理程序会如下所示："这样可以避免在`init.{ro.hardware}.rc`文件中包含一个特殊的模块和一个加载命令的覆盖层。"

代码块设置如下：

```java
extern int openFRAM(const unsigned int bus, const unsigned int address);
extern int readFRAM(const unsigned int offset, const unsigned int 
    bufferSize, const char *buffer);
extern int writeFRAM(const unsigned int offset, const unsigned int 
    const char *buffer);
extern void closeFRAM(void);
```

当我们希望引起你对代码块中某个特定部分的注意时，相关的行或项目会以粗体显示：

```java
public void onClickSaveButton(View view) {
   hwTask = new HardwareTask();
 hwTask.saveToFRAM(this); 
}

public void onClickLoadButton(View view) {
   hwTask = new HardwareTask();
 hwTask.loadFromFRAM(this);
}
```

命令行输入或输出内容如下所示：

```java
root@beagleboneblack:/ # i2cdetect -y -r 2

```

**新术语**和**重要词汇**以粗体显示。你在屏幕上看到的词，例如菜单或对话框中的，会在文本中以这种形式出现："如果用户再次点击**Sample**按钮，将实例化另一个`HardwareTask`实例。"

### 注意

警告或重要提示会以如下框中的形式出现。

### 提示

提示和技巧会以这种形式出现。

# 读者反馈

我们始终欢迎读者的反馈。告诉我们你对这本书的看法——你喜欢或不喜欢什么。读者的反馈对我们很重要，因为它帮助我们开发出你真正能从中获得最大收益的标题。

要向我们发送一般反馈，只需通过电子邮件`<feedback@packtpub.com>`，并在邮件的主题中提及书籍的标题。

如果你有一个有经验的主题，并且你对于写作或为书籍做贡献感兴趣，请查看我们在[www.packtpub.com/authors](http://www.packtpub.com/authors)的作者指南。

# 客户支持

既然你现在拥有了 Packt 的一本书，我们有一些事情可以帮助你最大限度地利用你的购买。

## 下载示例代码

你可以从你在[`www.packtpub.com`](http://www.packtpub.com)的账户下载你所购买的所有 Packt Publishing 图书的示例代码。如果你在其他地方购买了这本书，可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)注册，我们会将文件通过电子邮件直接发送给你。

## 勘误

尽管我们已经尽力确保内容的准确性，但错误仍然可能发生。如果你在我们的书中发现了一个错误——可能是文本或代码中的错误——如果你能向我们报告，我们将不胜感激。这样做，你可以避免其他读者感到沮丧，并帮助我们在后续版本中改进这本书。如果你发现任何勘误信息，请通过访问[`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)，选择你的书籍，点击**Errata Submission Form**链接，并输入你的勘误详情来报告。一旦你的勘误信息被验证，你的提交将被接受，并且勘误信息将被上传到我们的网站或添加到该标题下的现有勘误列表中。

要查看先前提交的勘误信息，请访问[`www.packtpub.com/books/content/support`](https://www.packtpub.com/books/content/support)，并在搜索字段中输入书名。所需信息将显示在**勘误**部分下。

## 盗版

在互联网上，盗版受版权保护的材料是所有媒体都面临的持续问题。在 Packt，我们非常重视保护我们的版权和许可。如果您在互联网上以任何形式遇到我们作品的非法副本，请立即提供位置地址或网站名称，以便我们可以寻求补救措施。

如果您发现疑似盗版材料，请通过`<copyright@packtpub.com>`联系我们，并提供该材料的链接。

我们感谢您帮助保护我们的作者以及我们为您提供有价值内容的能力。

## 问题

如果您对这本书的任何方面有问题，可以通过`<questions@packtpub.com>`联系我们，我们将尽力解决问题。
