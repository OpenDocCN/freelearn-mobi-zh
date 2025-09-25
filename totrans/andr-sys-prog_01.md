# Android系统编程简介

本书是关于Android系统编程的。在本章中，我们将从对系统编程和Android系统编程范围的讨论开始（以给出本书的高级概述）。之后，我们将查看Android系统架构。从架构中，我们可以看到本书将关注的层次。我们还将讨论本书中将使用的虚拟硬件平台和第三方开源项目。总之，本章将涵盖以下主题：

+   Android系统编程简介

+   Android系统架构概述

+   本书所使用的第三方项目简介

+   虚拟硬件平台简介

# 什么是系统编程？

当我们谈论什么是系统编程时，我们可以从维基百科中系统编程的定义开始：

"系统编程（或系统软件编程）是指编写系统软件的活动。与应用编程相比，系统编程的主要区别特征在于，应用编程旨在产生为用户提供服务的软件（例如文字处理器），而系统编程旨在产生为其他软件提供服务的软件和软件平台，这些软件和平台受性能限制，或者两者兼而有之（例如操作系统、计算科学应用、游戏引擎和AAA级视频游戏、工业自动化以及软件即服务应用）。"

从前面的定义中，我们可以看出，当我们谈论系统编程时，我们实际上是在处理计算机系统本身的构建块。我们可以描述系统架构及其在系统内部的外观。例如，我们可以参考关于Windows或Linux的系统编程书籍。由O'Reilly Media, Inc.出版的《Linux系统编程》一书包括关于文件I/O、进程管理、内存管理、中断处理等内容。还有一本名为《Windows系统编程》的书，由Addison-Wesley Professional出版，其内容与Linux版本非常相似。

您可能期望在这本书中看到类似的内容，但您会发现本书中的主题与经典的系统编程书籍相当不同。首先，为Android编写系统编程书籍讨论文件I/O、进程管理或内存管理并没有太多意义，因为《Linux系统编程》一书几乎可以涵盖Android的相同主题（Android使用Linux内核和设备驱动模型）。

当你想探索内核空间系统编程时，你可以阅读O'Reilly的《Linux设备驱动程序》或Prentice Hall的《Essential Linux Device Drivers》等书籍。当你想探索用户空间系统编程时，你可以阅读我之前提到的书籍，O'Reilly的《Linux系统编程》。然后你可能想知道，在这种情况下，我们需要一本《Android系统编程》的书吗？为了回答这个问题，这取决于我们如何看待Android的系统编程。或者换句话说，这取决于我们从哪个角度看待*Android系统编程*。我们可以从不同的视角对同一个世界讲述不同的事情。从这个意义上讲，我们可能需要不止一本书来讨论Android系统编程。

要谈论Android系统编程，我们可以从理论上或实践上进行讨论。在这本书中，我们将通过几个实际项目和动手实例来实践。我们的重点是定制Android系统以及如何将其移植到新的平台。

# 这本书的范围是什么？

如我们所知，Android有两种编程类型：应用编程和系统编程。

通常，很难在系统编程和应用编程之间划清界限，尤其是在基于C语言的操作系统，如Linux和各种Unix系统。在Android框架中，应用层与系统其他部分被很好地分离。你可能知道，Android应用编程使用Java语言，而Android系统编程则使用Java、C/C++或汇编语言。为了简化，我们可以将Android世界中除了应用编程之外的所有内容视为系统编程的范畴。从这个意义上讲，Android框架也属于系统编程的范畴。

从本书读者的角度来看，他们可能想了解他们在项目工作中可能触及的层次。Android框架是Google在大多数情况下唯一会更改的层次。从这个角度来看，我们不会花太多时间讨论框架本身。相反，我们将专注于如何将包括Android框架在内的系统从**Android开源项目**（**AOSP**）的标准平台移植到其他平台。在这本书中，我们将重点关注移植过程中需要更改的层次。

在我们完成移植工作后，将有一个新的Android系统可用。对于这个新系统，我们需要做的一件事是，不时地将新系统的更改传递给最终用户。这可能是一次重大的系统更新或错误修复。这由Android中的**空中**（**OTA**）更新支持。这也是本书的一个主题。

传统上，所有Unix编程都是系统级编程。Unix或Linux编程是围绕三个基石构建的，即系统调用、C库和C编译器。Android系统编程也是如此。在系统调用和C库之上，Android为Android应用级别提供了一个额外的抽象层。这就是Android框架和Java虚拟机。

从这个意义上讲，大多数Android应用程序都是使用Android SDK和Java语言构建的。你可能想知道是否可以使用C/C++或甚至使用Java进行系统级编程。是的，所有这些都是可能的。除了Android SDK之外，我们还可以使用Android NDK开发本地应用程序。还有许多使用Java语言开发的Android框架组件。我们甚至可以使用Visual Studio（Xamarin）用C#开发Android应用程序。然而，本书不会涉及这种复杂性。我们将专注于应用框架之下的层次。再次强调，重点是定制和扩展现有系统，或者将整个系统移植到新的硬件平台。

我们将重点关注Android系统的移植和定制，因为这些是大多数从事Android系统级工作的人会做的事情。在谷歌发布Android新版本后，芯片供应商需要将其移植到他们的参考平台上。当OEM/ODM公司获得参考平台时，他们必须将其定制为自己的产品。定制包括构建初始系统本身以及部署到部署系统的更新。本书的第一部分将讨论Android系统的移植。本书的第二部分将讨论如何更新现有系统。

如果我们考虑以下图中右侧的Android架构，我们可以看到大部分移植工作将集中在Android系统架构中的**内核**和**硬件抽象层（HAL**）上。这对其他Android衍生版本也是如此。本书中的知识和概念也适用于Android可穿戴设备和Brillo。以下图的左侧显示了**Brillo**的架构图。**Brillo**是谷歌为物联网设备提供的物联网操作系统。它是Android物联网设备的简化版和较小版本。然而，移植层与Android相同。

![图片](img/6099_01_01.png)

Android与Brillo系统架构比较

左侧的Brillo/Weave架构图是根据OpenIoT Summit上布鲁斯·比尔的演示制作的。感谢布鲁斯·比尔提供的精彩演示和YouTube上的视频，它对Brillo/Weave架构进行了非常全面的介绍。

# Android系统概述

如我们从架构图中可以看到，Android的架构层包括**应用程序框架**、**Android系统服务**、**HAL**和**内核**。Binder IPC用作进程间通信的机制。我们将在本节中介绍它们。由于恢复也是系统编程范围的一部分，我们还将在本节中概述恢复。

你可以在以下谷歌网站上找到有关关键移植层和系统架构内部信息的更多信息：

[http://source.android.com/devices/index.html](http://source.android.com/devices/index.html)

# 内核

如我们所知，Android使用Linux内核。Linux是由Linus Torvalds在1991年开发的。当前的Linux内核由Linux Kernel Organization Inc.维护。最新的主线内核发布可以在[https://www.kernel.org](https://www.kernel.org)找到。

Android使用略微定制的Linux内核。以下是Linux内核更改的简要列表：

+   **ashmem（Android共享内存）**：一个基于文件的共享内存系统，用于用户空间

+   **Binder**：一个**进程间通信**（IPC）和**远程过程调用**（RPC）系统

+   **logger**：一个针对写入进行优化的高速内核日志机制

+   **paranoid networking**：一种限制网络I/O到某些进程的机制

+   **pmem（物理内存）**：一个将大块物理内存映射到用户空间的驱动程序

+   **Viking Killer**：一个替换的OOM杀手，在低内存条件下实现了Android的“杀死最近最少使用进程”逻辑

+   **wakelocks**：Android独特的电源管理解决方案，其中设备的默认状态是睡眠，需要显式操作（通过wakelock）来防止设备进入睡眠状态

大多数更改都作为设备驱动程序实现，对核心内核代码的更改很少或没有。唯一显著的跨子系统更改是wakelocks。

今天，许多Android补丁被主线Linux内核接受。主线内核甚至可以直接启动Android。关于如何在运行主线内核的Nexus 7上启动的Linaro博客，你可以找到说明在[https://wiki.linaro.org/LMG/Kernel/FormFactorEnablement](https://wiki.linaro.org/LMG/Kernel/FormFactorEnablement)。

如果某个硬件设备的Linux设备驱动程序可用，它通常也可以在Android上工作。设备驱动程序的开发与典型Linux设备驱动程序的开发相同。如果你想了解与Android相关的主线内核合并，你可以查看内核发布说明[https://kernelnewbies.org/LinuxVersions](https://kernelnewbies.org/LinuxVersions)。

Android内核源代码通常由SoC供应商提供，例如高通或MTK。Google设备的内核源代码可以在[https://android.googlesource.com/kernel/](https://android.googlesource.com/kernel/)找到。

Google设备使用来自不同供应商的SoC，因此您可以在不同供应商这里找到内核源代码。例如，QualComm SoC的内核源代码位于`kernel/msm`，MediaTek的内核源代码位于`kernel/mediatek`。一般的Android内核源代码在`kernel/common`文件夹中，看起来与Vanilla内核非常相似。

AOSP的默认构建版本适用于谷歌的各种设备，例如Nexus或Pixel。最近它也开始包含一些来自硅供应商的参考板，例如`hikey-linaro`等。如果您需要针对您的参考平台的特定供应商的Android内核，您应该从您的平台供应商那里获取内核源代码。

同时，还有开源社区在维护Android内核。例如，ARM架构的内核可以在Linaro的许多参考板上找到。对于Intel x86架构，您可以在Android-x86项目中找到各种内核版本。正如您可以从以下Linaro Linux内核状态网站上看到的那样，`linaro-android`树是树外AOSP补丁的前向移植。它提供了Google下一个AOSP `kernel/common.git`树“可能”看起来是什么样的预览。

Linaro的Android内核树可以在[https://android.git.linaro.org/gitweb/kernel/linaro-android.git](https://android.git.linaro.org/gitweb/kernel/linaro-android.git)找到。

该内核树的状态可以在[https://wiki.linaro.org/LMG/Kernel/Upstreaming](https://wiki.linaro.org/LMG/Kernel/Upstreaming)查看。

# HAL

HAL为硬件供应商定义了一个标准接口，允许Android对底层驱动实现保持无知。HAL允许您在不影响或修改高级系统的情况下实现功能。HAL实现被打包成模块（`.so`）文件，并在适当的时候由Android系统加载。这是将Android系统移植到新平台的一个重点。我们将在[第3章](e0f861c2-5832-402f-89d3-cfc75785e759.xhtml)“发现内核、HAL和虚拟硬件”中了解更多关于HAL的内容。在这本书中，我将详细分析各种硬件接口的HAL层。

# Android系统服务

应用框架API公开的功能与系统服务通信，以访问底层硬件。应用开发者可能主要交互的两个服务组是**系统**（如窗口管理器和通知管理器）和**媒体**（涉及播放和录制媒体的服务）。这些服务是作为Android框架的一部分提供应用程序接口的。

除了这些服务之外，还有支持这些系统服务的本地服务，例如SurfaceFlinger、netd、logcatd、rild等等。许多服务与您可能在Linux发行版中找到的Linux守护进程非常相似。在复杂的硬件模块中，例如图形，系统服务和本地服务都需要访问HAL，以便向应用层提供框架API。我们将在[第6章](7ff9a111-96cb-44ed-b92d-435e73821306.xhtml)，*在Android模拟器上启用Wi-Fi*到[第9章](7ff9a111-96cb-44ed-b92d-435e73821306.xhtml)，*使用PXE/NFS启动x86vbox*时讨论系统服务。

# Binder IPC

Binder IPC机制允许应用框架跨越进程边界并调用Android系统服务代码。这使得高级框架API能够与Android系统服务交互。Android应用程序通常在其自己的进程空间中运行。它没有直接访问系统资源或底层硬件的能力。它必须通过Binder IPC与系统服务通信来访问系统资源。由于应用程序和系统服务运行在不同的进程中，Binder IPC为此提供了机制。

Binder IPC代理是应用框架访问不同进程空间中系统服务的渠道。这并不意味着它是应用框架和系统服务之间的一个层。Binder IPC是任何想要与其他进程通信的进程都可以使用的进程间通信机制。例如，系统服务可以使用Binder IPC相互通信。

# 应用框架

应用框架为应用程序提供API。它通常被应用程序开发者使用。当应用程序调用一个接口后，应用框架通过Binder IPC机制与系统服务进行通信。Android应用框架不仅仅是一组供应用开发者使用的库。它提供了比这更多的功能。

Android应用框架带给开发社区的突破性技术是应用层和系统层之间一个非常清晰的分离。正如我们所知，Android应用开发使用Java语言，Android应用程序在类似于Java虚拟机的环境中运行。在这种设置中，应用层与系统层被非常清晰地分离。

Android应用框架还与Google的**集成开发环境**（**IDE**）紧密集成，并提供了一种独特的编程模型。使用这种编程模型和相关工具，Android开发者可以以极高的效率和生产力进行应用开发。所有这些都是Android在移动设备世界中取得巨大成功的关键原因。

我已经在之前的Android系统架构图中对所有的层进行了整体介绍。正如我之前提到的关于Android系统编程的范围，我们可以将Android系统中的所有编程视为系统编程的范围，而不是应用编程。带着这个概念，我们实际上在之前的架构图中遗漏了一部分，那就是恢复。

# 恢复

在这一章中，我们还想简要地看看恢复，因为在这本书的第二部分中，我们有三章是关于它的。

恢复是一个可以用来升级或重新安装Android系统的工具。它是AOSP源代码的一部分。恢复的源代码可以在`$AOSP/bootable/recovery`找到。

与Android的其他部分相比，恢复的独特之处在于它本身是一个自包含的系统。我们可以通过以下图示来查看恢复，并将其与我们之前讨论过的Android和Brillo架构进行比较：

![图片](img/image_01_002.png)

恢复是一个与Android系统分开的系统，它与它所支持的Android系统共享相同的内核。我们可以将其视为一个迷你操作系统或一个可以在许多嵌入式设备中找到的嵌入式应用程序。它是在与Android相同的Linux内核上运行的专用应用程序，它执行单个任务，即更新当前的Android系统。

当系统启动到恢复模式时，它从闪存中的专用分区启动。这个分区包括包含Linux内核和特殊ramdisk图像的恢复镜像。如果我们查看Nexus 5分区，我们会看到以下列表：

[PRE0]

列表中包括29个分区，恢复分区是其中之一。恢复ramdisk与正常的ramdisk具有类似的目录结构。在恢复ramdisk的init脚本中，init启动恢复程序，它是恢复模式的主要进程。恢复本身与Android系统中的其他本地守护进程相同。恢复的编程是Android系统编程范围的一部分。恢复的编程语言和调试方法也与原生Android应用程序相同。我们将在本书的第二部分更深入地讨论这个问题。

# 从AOSP派生出的第三方开源项目

如我们所知，AOSP 源代码是我们可以从系统级编程开始工作的主要源代码。各种芯片供应商通常与谷歌合作，以启用他们的参考平台。这是一项巨大的努力，他们不会将所有内容都公布于众，除非是他们的客户。这给开源世界带来了限制。由于 AOSP 源代码主要是为谷歌设备，如模拟器、Nexus 或 Pixel 系列，因此使用 Nexus 设备作为硬件参考平台的开发者没有问题。其他设备呢？制造商可能会发布他们设备的内核源代码，但仅此而已。在开源世界中，几个第三方组织为这种情况提供了解决方案。在接下来的几节中，我们将简要介绍我们在本书中使用的一些解决方案。

# LineageOS (CyanogenMod)

LineageOS 是一个社区，为许多流行的 Android 设备提供售后固件分发。它是高度受欢迎的 CyanogenMod 的继任者。如果您无法从 AOSP 源代码为您的设备构建 ROM，您可以考虑 LineageOS 源代码。由于 LineageOS 支持许多设备，许多主要的第三方 ROM 映像都是基于其前身 CyanogenMod 构建的。从中国的著名 MIUI 到最新的 OnePlus 设备，它们都使用 CyanogenMod 源代码作为基础开始。LineageOS/CyanogenMod 对开源世界的重大贡献是将 Linux 内核和 HAL 适配到各种 Android 设备。

LineageOS 的源代码维护在 GitHub 上，您可以在 [https://github.com/LineageOS](https://github.com/LineageOS) 找到它。

要为您的设备构建 LineageOS 源代码，整体构建过程与 AOSP 构建类似。关键区别是 LineageOS 支持的设备数量众多。对于每个设备，都有一个网页提供有关如何为该设备构建的信息。我们以 Nexus 5 为例。您可以通过以下页面获取详细信息：

[https://wiki.lineageos.org/devices/hammerhead](https://wiki.lineageos.org/devices/hammerhead)

在信息页面上，您可以找到有关如何下载 ROM 映像、如何安装映像以及如何构建映像的信息。有一个针对设备的构建指南，我们可以在 [https://wiki.lineageos.org/devices/hammerhead/build](https://wiki.lineageos.org/devices/hammerhead/build) 找到 Nexus 5 的构建指南。

要为 Nexus 5 构建 LineageOS，两个关键元素是 **内核** 和 **设备**。内核包括 Linux 内核和 Nexus 5 特定的设备驱动程序，而设备包括设备特定 HAL 代码的大部分。内核和设备文件夹的命名规范是 `android_kernel/device_{制造商}_{代号}`。

Nexus 5 的代号为 **hammerhead**，制造商为 **lge**，即 LG。

我们可以找到以下两个用于内核和设备的 Git 仓库：

[https://github.com/LineageOS/android_kernel_lge_hammerhead](https://github.com/LineageOS/android_kernel_lge_hammerhead)

[https://github.com/LineageOS/android_device_lge_hammerhead](https://github.com/LineageOS/android_device_lge_hammerhead)

除了内核和设备之外，其他重要信息是 LineageOS 版本。你可以在同一设备信息页面上找到它。对于 Nexus 5，可用的版本有 11、12、12.1、13 和 14.1。你可能想知道如何将 LineageOS 版本与 AOSP 版本匹配。

有关 CyanogenMod 和 LineageOS 的信息可以在以下两个维基百科页面上找到：

[https://en.wikipedia.org/wiki/CyanogenMod#Version_history](https://en.wikipedia.org/wiki/CyanogenMod#Version_history)

[https://en.wikipedia.org/wiki/LineageOS#Version_history](https://en.wikipedia.org/wiki/LineageOS#Version_history)

Nexus 5 支持的 LineageOS/CyanogenMod 和 AOSP 版本是 CM11 (Android 4.4)、CM 12 (Android 5.0)、CM 12.1 (Android 5.1)、CM 13 (Android 6.0) 和 CM 14.1 (Android 7.1.1)。

在阅读这本书的过程中，你将无法访问与 CyanogenMod 相关的链接，因为最近 CyanogenMod 背后的基础设施已经关闭。你可以阅读以下帖子以获取更多信息：

[https://plus.google.com/+CyanogenMod/posts/RYBfQ9rTjEH](https://plus.google.com/+CyanogenMod/posts/RYBfQ9rTjEH)

尽管如此，前述配置中的想法是，区分不同设备的关键代码是内核和设备。有可能在设备之间共享其余的代码。这是本书中项目的一个目标。我们试图将不同硬件平台的更改限制在内核和设备中，同时保持其余的 AOSP 源代码不变。这并非 100% 可能，但我们尽可能尝试这样做。好处是我们可以将我们的代码与 AOSP 代码分开，并且更新到新 AOSP 版本要容易得多。

# Android-x86

虽然 LineageOS/CyanogenMod 为大量 Android 设备提供了出色的支持，但这些设备中的许多都是来自各种硅供应商的 ARM 架构设备，例如高通、三星、MTK 等。同样，也有针对基于 Intel 的 Android 设备的开源社区。这是另一个著名的开源项目，Android-x86。尽管市场上基于 Intel x86 的 Android 设备数量无法与 ARM 架构设备相比，但还有另一个市场广泛使用 Intel x86 Android 构建，那就是 Android 模拟器市场。对于商业 Android 模拟器产品，你可以找到 AMI DuOS、Genymotion、Andy 等等。

与LineageOS/CyanogenMod相比，Android-x86项目在支持各种基于Intel x86的设备时采用了非常不同的方法。其目标是为任何Intel x86设备提供**板级支持包**（**BSP**）。这类似于您在PC上安装Microsoft Windows或Linux的方式。您只有一份发布版本，可以在任何Intel PC上安装。没有为每个不同的PC或笔记本电脑专门构建的Windows或Linux版本。

为了在Android上实现这一目标，Android-x86对Android启动过程进行了显著定制。在Android-x86中，启动过程分为两个阶段。第一阶段是使用特殊的ramdisk--`initrd.img`启动一个最小的Linux环境。当系统可以启动到这个Linux环境时，它将通过`chroot`或`switch_root`命令启动第二阶段。在这个阶段，它将启动实际的Android系统。

这是一种非常聪明的利用现有技术解决新挑战的方法。本质上，我们试图分两步解决问题。在启动过程的第一个阶段，由于Windows和Linux都可以在没有专用构建的情况下在Intel x86 PC上启动，您应该能够不太费力地在Intel设备上启动Linux。这正是Android-x86启动的第一个阶段所做的事情。当最小的Linux系统可以正常运行时，这意味着最小的一组硬件设备已经初始化，您可以使用这个最小的Linux环境来调试或启动系统的其余部分。在第二阶段，可以启动一个适用于Intel x86的通用Android镜像，并且只需要有限的硬件初始化。这种方法也可以用于硬件设备的调试。本书将展示我们如何在Android模拟器上做同样的事情。

Android-x86项目的官方网站是[http://www.android-x86.org/](http://www.android-x86.org/)。您可以在那里找到关于Android-x86项目的相关信息。要构建Android-x86，获取源代码稍微有些棘手。原始源代码托管在`http://git.android-x86.org`，由**台湾Linux用户组**（**TLUG**）的志愿者维护。它有效了几年。然而，从2015年4月起，它就不再工作了。

你可以从Google讨论组[https://groups.google.com/forum/#!forum/android-x86](https://groups.google.com/forum/#!forum/android-x86)中找到最新的状态。讨论组中有一个关于`git.android-x86.org`问题的官方公告，由维护者Chih-Wei Huang发布。后来，托管被短暂地移至SourceForge。然而，自2016年7月以来，又报告了从SourceForge检索源代码的问题。目前，源代码托管在OSDN上，你可以在Android-x86讨论组中搜索Chih-Wei Huang于2016年9月8日发布的公告。由于大多数开源项目都是由志愿者维护的，它们可能会不时地上线或下线。始终保留你正在工作的项目自己的镜像是个好主意。我们将在本书中讨论这个问题，以便你可以完全控制自己的工作。

我们知道许多开源项目之间相互关联，Android-x86和LineageOS/CyanogenMod也是如此。从2016年1月开始，Jaap Jan Meijer对CyanogenMod进行了初始移植到Android-x86，这使得CyanogenMod可以在大多数英特尔设备上使用。如果你对这个话题感兴趣，你可以在Android-x86讨论组中搜索`CM移植计划`。

# CWM/CMR/TWRP

作为系统级编程的一部分，我们在上一节中介绍了恢复功能。AOSP的原始恢复功能仅支持非常有限的功能，因此有许多第三方恢复项目。

**ClockworkMod恢复**（**CWM**）是著名的开源恢复项目之一，由Koushik Dutta编写。尽管现在许多人仍然使用ClockworkMod恢复，但该项目已经停止开发一段时间了。

另一个恢复项目是**CyanogenMod恢复**（**CMR**）。CMR由CyanogenMod团队维护，它与ClockworkMod恢复非常相似。

**TWRP**或**TeamWin恢复项目**是另一个非常广泛使用的自定义恢复。它是完全触摸驱动的，并拥有最完整的特性集之一。TWRP是OmniROM的默认恢复，其源代码托管在GitHub上，作为OmniROM的一部分，网址为[https://github.com/omnirom/android_bootable_recovery/](https://github.com/omnirom/android_bootable_recovery/)。

# 集成策略

在前面的章节中，我们讨论了Android架构、AOSP以及Android的第三方开源项目。软件行业已经存在了几十年。有如此多的现有源代码可以重用，从头开始创建的需求非常罕见。为新平台进行移植和定制基本上是集成艺术。

在这本书中，我们将使用AOSP源代码作为基础，并尝试在其上构建一切。然而，我们可能不能仅依赖于AOSP源代码。实际上，我们想展示如何支持AOSP不支持的平台。我们将如何做到这一点？我们是从头开始创建吗？答案是：不是。我们将演示如何将所有现有项目整合在一起以创建一个新平台。这就是我们讨论第三方开源项目的原因。

在我们的案例中，VirtualBox不受AOSP支持，我们将使用AOSP和Android-x86来启用它。我们需要使用来自AOSP和Android-x86的项目来为VirtualBox构建一个系统。然而，我们的目标是创建一个对AOSP源代码树改动最小的VirtualBox新构建系统。这也是许多基于AOSP的其他项目的目标。

基于前面的理解，在我们的集成过程中，我们有四个类别的项目：

+   **未经修改的原始AOSP项目**：在这些类型的项目中，我们将使用未经修改的AOSP项目。

+   **第三方项目**：在这个类别中，项目是由第三方项目添加的，并且不是AOSP的一部分，因此也没有涉及任何更改。

+   **由AOSP和第三方项目共同修改的项目**：这很复杂。我们需要审查第三方更改并决定我们是否希望将它们包含在我们的系统中。

+   **由多个开源项目和AOSP修改的项目**：这是我们应尽量避免集成或更改的最复杂情况。

很容易理解，我们应该尽可能多地重用类别1和2中的项目。挑战和主要工作将在类别3中，而我们应该尽可能避免类别4。

# 虚拟硬件参考平台

新的Android发布通常包含两个参考平台。开发者可以先在Android模拟器上测试新的Android发布。这在预览阶段可能非常有用。在官方发布后，Google硬件平台，如Nexus或Pixel，通常成为开发者的设备。模拟器和Nexus/Pixel构建是AOSP中最早可用的构建。

在这本书中，我们将使用Android模拟器作为我们主题的虚拟硬件参考平台。由于Android模拟器的构建已经在AOSP中可用，你可能会想知道我们能用它做什么。实际上，我们可以通过添加新功能来定制现有的平台。这就是OEM/ODM公司通常使用来自硅供应商的参考平台所做的事情。使用Android模拟器，我们将演示如何创建一个新设备，以便我们可以对其进行定制。如果你了解任何商业模拟器产品，例如Genymotion和AMI DuOS，那么你可能知道这些产品为模拟器添加了哪些功能。我们将以非常相似的方式扩展Android模拟器。

在我们探讨了关于新设备定制的主题之后，我们将探讨更多关于移植的高级主题。移植的主要工作是内核和HAL的更改。为了讨论关于移植和调试的高级主题，我们还将使用VirtualBox作为另一个虚拟硬件参考平台。尽管VirtualBox被许多商业模拟器产品（如Genymotion、AMI DuOS、Leapdroid等）使用，但它并未直接得到AOSP的支持。大多数PC上的Android模拟器都是基于VirtualBox的，它们是为游戏玩家运行Android游戏而设计的。在这本书中，我们将学习如何使用各种开源资源创建类似的构建。

# x86架构的Android模拟器简介

Android模拟器在Android 4、5、6和7版本中也发生了巨大变化。在Android 5之前，Android模拟器是基于名为goldfish的虚拟硬件参考板构建的。

金鱼虚拟硬件平台的硬件规范可以在AOSP源代码树中的`$AOSP/platform/external/qemu/docs/GOLDFISH-VIRTUAL-HARDWARE.TXT`找到。在这本书中，我们将把AOSP根目录称为`$AOSP`。

金鱼虚拟硬件平台是基于QEMU 1.x构建的，用于在x86环境中模拟ARM设备。x86主机环境可以是Windows、Linux或macOS X计算机。由于目标设备架构是通过QEMU模拟的，性能较差。模拟器运行非常缓慢，对于应用程序开发者来说难以使用。然而，QEMU在x86架构上得到了积极开发，并且与各种虚拟化技术（如VT-x、AMD-V等）广泛使用。

自Android 4.x以来，英特尔在Linux上使用KVM和Windows及macOS X上的Intel HAXM开发了一个基于x86的Android模拟器。随着虚拟化技术被引入模拟器，基于英特尔x86的模拟器比模拟ARM或MIPS架构的模拟器要快得多。为了方便Android应用程序开发者，谷歌官方将基于英特尔x86的Android模拟器集成到Android SDK中。基于英特尔x86的Android模拟器已成为开发者测试其Android应用程序的首选选择。

# ranchu简介

随着Android 5（Lollipop）的引入，64位硬件架构对ARM和英特尔平台都可用。然而，当时Android的64位硬件设备仍在开发中。开发者唯一的选择是从硅供应商那里获取硬件参考平台。

为了帮助开发者测试其应用程序在64位架构上的运行，Linaro的工程师在QEMU上实现了一个虚拟硬件平台，以测试ARMv8-A 64位架构。他们给这个虚拟硬件平台取了一个代号，**ranchu**。您可以参考Linaro的博客，由Alex Bennée撰写，[https://www.linaro.org/blog/core-dump/running-64bit-android-l-qemu/](https://www.linaro.org/blog/core-dump/running-64bit-android-l-qemu/)。

这个改变后来被谷歌采纳，并作为下一代Android模拟器的硬件参考平台。如果你安装了Android SDK镜像，你可以从Android 5开始看到两个内核镜像。内核镜像`kernel-qemu`是与goldfish虚拟硬件平台一起使用的镜像，而镜像`kernel-ranchu`是与ranchu虚拟硬件平台一起使用的镜像。

为了应对这一变化，英特尔和MIPS都对其架构进行了工作，以支持ranchu中的64位硬件模拟。你可以参考在[https://groups.google.com/forum/#!topic/android-emulator-dev/dltBnUW_HzU](https://groups.google.com/forum/#!topic/android-emulator-dev/dltBnUW_HzU)的群组讨论。

ranchu硬件平台基于更新的QEMU版本，其架构的改变减少了对于谷歌修改和goldfish特定设备的依赖。例如，它使用virtio-block设备来模拟NAND和SD卡。这有可能提供更好的性能，同时也使得利用最新QEMU代码库提供的功能成为可能。ranchu内核是基于`android-goldfish-3.10`分支的新版本构建的，而最新的goldfish内核在`android-goldfish-3.4`分支。你可以通过在Android虚拟设备上使用来自Android SDK的不同内核来注意到这种差异。

# 基于VirtualBox的Android模拟器

随着虚拟化技术的不断演变，市场上也出现了许多基于商业的Android模拟器产品。你可能听说过其中的一些，例如Genymotion、AMIDuOS、Andy、BlueStacks等等。其中许多都是使用Oracle的VirtualBox构建的，例如Genymotion、AMIDuOS和Andy。使用VirtualBox而不是其他解决方案（如VMware）的原因是VirtualBox是一个开源解决方案。

为了实现最佳的性能和用户体验，商业模拟器产品中的主机和目标都需要进行定制。除了Android模拟器，我们还将使用VirtualBox作为虚拟硬件平台来演示如何将Android移植到新平台。在这本书中我们需要另一个虚拟硬件平台的原因是Android模拟器已经在AOSP中得到支持。我们将使用Android模拟器作为一个平台来教授如何扩展和定制现有平台。而VirtualBox在AOSP中不受支持，但它可以用作目标平台来教授如何将Android移植到新平台。尽管Android已经被Genymotion、AMI和其他人移植到VirtualBox，但它们都不是开源产品。

# 摘要

在本章中，我们讨论了Android系统编程是什么以及本书中涉及的系统级编程范围。之后，我们对Android系统架构进行了概述，并讨论了本书中我们将关注的层次。我们还讨论了本书中使用的虚拟硬件平台。在本书中，我们使用了来自各种第三方项目的代码，因此在本章中也简要概述了每个项目。在下一章中，我们将开始学习Android系统编程的开发环境设置。这包括开发工具和源代码仓库的设置。
