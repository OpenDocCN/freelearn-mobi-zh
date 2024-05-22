# 第一章. 安装和配置 Android Studio

你希望熟悉新的官方 Google IDE Android Studio。你想知道这个环境中可用的功能。你希望制作自己的 Android 应用程序，并希望这些应用程序能够在 Google Play 商店供其他用户使用。你能轻松做到这一点吗？你如何实现这个目标？

本章将指导你如何准备新的 Android Studio 安装，以及如何在新的环境中迈出第一步。我们将从准备安装系统和下载必要的文件开始。我们将看到第一次运行 Android Studio 时出现的欢迎屏幕，并正确配置 Android **SDK**（**软件开发工具包**），以便你准备好创建你的第一个应用程序。

这是我们将在本章中讨论的主题：

+   安装 Android Studio

+   第一次运行 Android Studio 时的欢迎屏幕

+   安装 Android SDK 的配置

# 安装前的准备

开始使用 Android Studio 的一个前提条件是在你的系统中安装 Java。系统还必须能够找到 Java 的安装路径。这可以通过设置一个名为`JAVA_HOME`的环境变量来实现，该变量必须指向你系统中**JDK**（**Java 开发工具包**）的文件夹。检查这个环境变量，以避免在安装 Android Studio 时出现未来的问题。

# 下载 Android Studio

Android Studio 的安装包可以从 Android 开发工具网页下载，地址为：[`developer.android.com/sdk/installing/studio.html`](http://developer.android.com/sdk/installing/studio.html)。

这个包是 Windows 系统的 EXE 文件：

[Windows 系统的安装包下载地址](http://dl.google.com/android/studio/android-studio-bundle-130.737825-windows.exe)。

Mac OS X 系统的 DMG 文件：

[Mac OS X 系统的安装包下载地址](http://dl.google.com/android/studio/android-studio-bundle-130.737825-mac.dmg)。

Linux 系统的 TGZ 文件：

[Linux 系统的安装包下载地址](http://dl.google.com/android/studio/android-studio-bundle-130.737825-linux.tgz)。

## 安装 Android Studio

在 Windows 系统中，请运行 EXE 文件。默认安装目录为`\`Users\\<your_user_name>\Appdata\Local\Android\android-studio`。`Appdata`目录通常是隐藏的目录。

在 Mac OS X 中，打开 DMG 文件，并将 Android Studio 拖放到你的应用程序文件夹中。默认安装目录为`/Applications/Android/ Studio.app`。

在 Linux 系统中，解压 TGZ 文件，并执行位于`android-studio/bin/`目录下的`studio.sh`脚本。

如果你在安装过程或后续步骤中遇到任何问题，可以通过查看第十章，*获取帮助*，了解有关问题和已知问题的帮助。

## 首次运行 Android Studio

执行 Android Studio 并等待其完全加载（可能需要几分钟）。首次运行 Android Studio 时，会提示一个欢迎屏幕。如下截图所示，欢迎屏幕包括一个打开最近项目的部分和一个 **快速入门** 部分。我们可以创建新项目，导入项目，打开项目，甚至执行更高级的操作，例如从版本控制系统检出或打开配置选项。

![首次运行 Android Studio](img/5273OS_01_01.jpg)

让我们看看 **快速入门** 部分提供的各种选项：

+   **新建项目...**：创建一个新的 Android 项目

+   **导入项目**：通过从你的系统中导入现有源代码创建一个新项目

+   **打开项目**：打开一个现有项目

+   **从版本控制检出**：通过从版本控制系统中导入现有源代码创建一个新项目

+   **配置**：打开配置菜单

    +   **设置**：打开 Android Studio 设置

    +   **插件**：打开 Android Studio 的插件管理器

    +   **导入设置**：从文件（`.jar`）导入设置

    +   **导出设置**：将设置导出到文件（`.jar`）

    +   **项目默认设置**：打开项目默认设置菜单

    +   **设置**：打开模板项目设置。这些设置也可以从 Android Studio 设置中访问（**配置** | **设置**）

    +   **项目结构**：打开项目和平台设置

    +   **运行配置**：打开运行和调试设置

+   **文档和操作指南**：打开帮助菜单

    +   **阅读帮助**：打开在线版的 Android Studio 帮助

    +   **每日小贴士**：打开一个显示每日小贴士的对话框

    +   **默认键位参考**：打开包含默认键位的在线 PDF

    +   **JetBrains TV**：打开包含视频教程的 JetBrains 网站

    +   **插件开发**：打开包含插件开发者信息的 JetBrains 网站

# 配置 Android SDK

必须正确配置的核心功能是 Android SDK。尽管 Android Studio 会自动安装最新的可用 Android SDK，因此理论上你已经拥有创建第一个应用程序所需的一切，但检查它并了解如何更改它仍然很重要。

在 Android Studio 的欢迎界面中，导航至 **配置** | **项目默认设置** | **项目结构**。在 **平台设置** 中，点击 **SDKs**。将显示已安装的 SDK 列表，你应当在列表中至少有一个 Android SDK。在 **项目设置** 中，点击 **项目** 以打开项目默认模板的一般设置。你应当已选择一个 **项目 SDK**，如下一个截图中所示。这个选定的 SDK 将默认用于我们的 Android 项目，但即便如此，我们也可以稍后针对需要特殊设置的具体项目进行更改。

![配置 Android SDK](img/5273OS_01_02.jpg)

如果你没有在 Android Studio 中配置任何 Android SDK，那么我们需要手动添加它。

要完成这项任务，在**平台设置** | **SDKs**中，点击绿色的加号按钮来添加 Android SDK 到列表中，然后选择 SDK 的主目录。通过导航到你的 Android Studio 安装目录来检查你的系统中是否已有 SDK。你应该能找到一个名为`sdk`的文件夹，其中包含了 Android SDK 及其工具。Android Studio 的安装目录可能在隐藏文件夹中，所以请点击以下截图高亮的按钮来**显示隐藏文件和目录**：

![配置 Android SDK](img/5273OS_01_03.jpg)

如果你想使用不同于 Android Studio 中包含的另一个 Android SDK，请选择它。例如，如果你之前使用的是为 Eclipse 准备的**ADT**（**Android 开发工具**）插件，那么你的系统中已经安装了 Android SDK。你也可以同时添加这两个 SDK。

添加完 SDK 后，它将出现在列表中，你可以在项目设置中选择默认值。

### 提示

**下载示例代码**

你可以从你在[`www.packtpub.com`](http://www.packtpub.com)的账户下载你所购买的所有 Packt 图书的示例代码文件。如果你在别处购买了这本书，可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)注册，我们会直接将文件通过电子邮件发送给你。

# 概述

我们已经成功为 Android Studio 准备系统并安装了我们的 Android Studio 实例。我们首次运行了 Studio，现在我们知道欢迎屏幕上有哪些选项。我们还学会了如何配置我们的 Android SDK 以及如果你想使用不同版本如何手动安装。完成这些任务后，你的系统将运行并配置好 Android Studio，以便创建你的第一个项目。

在下一章中，我们将了解项目概念以及它如何包含应用程序所需的一切，从类到库。我们将创建我们的第一个项目，并讨论向导中可用的不同类型的活动。
