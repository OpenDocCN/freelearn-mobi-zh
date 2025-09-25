# 在VirtualBox上创建您的设备

我们已经学习了如何使用x86emu定制和增强现有设备以支持新功能。x86emu设备是在以下Android模拟器之上创建的设备：goldfish和ranchu。从本章到第11章[启用VirtualBox特定硬件接口]，我们将转向一个高级主题：移植Android系统。对于AOSP不支持的平台，我们能做什么？

在本章中，我们将转向一个新的设备，x86vbox。我们将创建这个新的x86vbox设备，以便在VirtualBox上运行它。由于VirtualBox是AOSP直接不支持的虚拟硬件，我们必须自己创建HAL层。自己创建HAL层并不意味着我们必须从头开始创建一切。正如我之前提到的，移植和定制是集成的艺术。我们可以从其他开源项目中集成我们需要的设备驱动程序。在本章中，我们将涵盖以下主题：

+   分析Android-x86项目的HAL并使用Android-x86 HAL为x86vbox设备

+   基于Android-x86 HAL分析创建x86vbox设备

+   分析x86vbox的启动过程

# x86vbox的HAL

在我们创建新的x86vbox设备之前，我们需要解决一个关键问题：创建x86vbox的HAL。这意味着我们需要支持在VirtualBox上出现的硬件设备。正如我们之前所说的，Android-x86项目是一个旨在为任何基于x86的计算设备提供**板级支持包**（**BSP**）的项目。尽管VirtualBox是一个虚拟化的x86硬件环境，我们仍然可以使用Android-x86项目的一部分来支持它。在下面的表中，我们可以看到我们从Android-x86中重用的项目列表。我们需要在构建中包含以下三个项目类别：

+   **Linux内核**：Android-x86提供了一个可以与Android配合使用的内核，用于Intel x86架构。

+   **针对Intel x86架构的HAL**：Android-x86在大多数你可以在PC上找到的设备上包含了HAL支持。

+   **Android系统项目和框架项目**：Android-x86将`system/`和`frameworks/`目录下的某些项目进行了更改，以满足x86架构特定的要求。例如，`system/core`下的`init`和`init.rc`已经被更改以与Android-x86的双阶段启动相兼容。

在下面的表中，我们还可以从另一个维度查看项目：

+   Android-x86更改的AOSP项目。

+   仅支持Android-x86的项目。

+   仅支持x86vbox的项目。

在本章和随后的章节中，我们将创建x86vbox设备，并对以下一些项目进行更改，以便在VirtualBox上运行x86vbox。

在下面的表中，我们还列出了来自AOSP、Android-x86和x86vbox的所有内核和HAL相关项目。由它们创建或更改的项目用**X**标记：

| **项目** | **AOSP** | **Android-x86** | **x86vbox** | **HAL 模块** |
| --- | --- | --- | --- | --- |
| `kernel` | X | X | X |  |
| `device/generic/x86vbox` |  |  | X |  |
| `bionic` | X | X |  |  |
| `bootable/newinstaller` |  | X | X |  |
| `device/generic/common` | X | X | X |  |
| `device/generic/firmware` |  | X |  |  |
| `external/alsa-lib` |  | X |  |  |
| `external/alsa-utils` |  | X |  |  |
| `external/bluetooth/bluez` |  | X |  | `bluetooth.default` `audio.a2dp.default` |
| `external/bluetooth/glib` |  | X |  |  |
| `external/bluetooth/sbc` |  | X |  |  |
| `external/busybox` |  | X |  |  |
| `external/drm_gralloc` | X | X |  | `gralloc.drm` |
| `external/drm_hwcomposer` | X | X |  | `hwcomposer.drm` |
| `external/e2fsprogs` | X | X |  |  |
| `external/ffmpeg` |  | X |  |  |
| `external/libdrm` | X | X |  |  |
| `external/libpciaccess` |  | X |  |  |
| `external/libtruezip` |  | X |  |  |
| `external/llvm` | X | X |  |  |
| `external/mesa` |  | X |  |  |
| `external/s2tc` |  | X |  |  |
| `external/stagefright-plugins` |  | X |  |  |
| `external/v86d` |  | X |  |  |
| `frameworks/av` | X | X |  |  |
| `frameworks/base` | X | X | X |  |
| `frameworks/native` | X | X |  |  |
| `hardware/broadcom/wlan` | X | X |  |  |
| `hardware/gps` |  | X |  | `gps.default` `gps.huawei` |
| `hardware/intel/audio_media` | X | X |  | `audio.primary.hdmi` |
| `hardware/intel/libsensors` |  | X |  | `sensors.hsb` |
| `hardware/libaudio` |  | X |  | `audio.primary.x86` |
| `hardware/libcamera` |  | X |  | `camera.x86` |
| `hardware/libhardware` | X | X |  | `libhardware` |
| `hardware/libhardware_legacy` | X | X |  | `audio_policy.default` |
| `hardware/liblights` |  | X |  | `lights.default` |

| `hardware/libsensors` |  | X |  | `sensors.hdaps` `sensors.iio`

`sensors.kbd`

`sensors.s103t`

`sensors.w500` |

| `hardware/ril` | X | X |  |  |
| --- | --- | --- | --- | --- |
| `hardware/x86power` |  | X |  | `power.x86` |
| `system/core` | X | X |  |  |

# x86vbox 的清单

根据 preceding 表格的分析，我们可以为 x86vbox 创建清单文件。从前面的表格中，我们可以看到我们重用了来自 Android-x86 的 39 个项目来形成 VirtualBox 的 HAL。在这 39 个项目中，有 16 个来自 AOSP，并由 Android-x86 进行了修改。为了在 VirtualBox 上运行我们的 x86vbox 设备，我们需要在 `device/generic/x86vbox` 创建设备 x86vbox。我们还需要更改四个项目：`kernel`、`bootable/newinstaller`、`device/generic/common` 和 `frameworks/base`。

在 x86vbox 的清单中，我们将包括前面提到的项目，用于 x86 内核、HAL，并且修改了 `system/` 以及 `frameworks/`：

[PRE0]

我们可以看到，x86vbox的清单包含两部分。第一部分包括x86内核、x86vbox HAL以及所有位于GitHub上的修改过的AOSP项目。第二部分包括原始的AOSP项目。第二部分的所有项目都没有被Android-x86或x86vbox修改过。第一部分的大多数项目只被Android-x86修改过，所以我们也不必对这些项目做任何事情。

在清单的第一部分中，`external/`或`hardware/`目录下的所有项目都与x86 HAL相关。你可能对唯一的AOSP项目**bionic**有疑问。你可能想知道为什么Android-x86会修改它，因为它是Android的C库。你可能知道系统调用是在Linux系统的C库中实现的。原始的bionic缺少两个系统调用`ioperm`和`iopl`，而它们是`external/v86d`项目所需要的，该项目是`vesafb`帧缓冲驱动器的用户空间守护进程。

所有的前述分析帮助我们明确了工作范围。正如我们所见，工作范围并不像我们最初想象的那么大。现在有很多开源项目可供使用。如果我们尽可能多地重用它们，通常可以大幅减少工作量。

GitHub上的所有Android-x86项目都是从Android-x86镜像分叉的，这样我们就可以修改它们。

# 创建新的x86vbox设备

一旦我们有了VirtualBox的HAL，我们现在可以创建一个名为x86vbox的新设备。如果我们回顾一下在[第4章](f69f330a-932c-4a32-bb78-e427c860b65e.xhtml)中创建x86emu设备的过程，*自定义Android模拟器*，我们知道我们需要为新设备准备一个板/设备配置Makefile和一个产品定义Makefile。我们也可以通过继承现有设备来创建一个新设备。如果我们查看前面的x86 HAL表格，我们可以看到一个共同的x86设备项目，`device/common`，可以在Android-x86中找到。我们将通过继承这个共同的x86设备来创建我们的新设备x86vbox。本章中创建的x86vbox是一个32位x86设备。你可以按照相同的说明自己创建一个x86_64设备。

正如我们在[第4章](f69f330a-932c-4a32-bb78-e427c860b65e.xhtml)中所述，*自定义Android模拟器*，我们创建了一个`AndroidProducts.mk` Makefile来包含x86vbox的产品定义Makefile，如下所示：

[PRE1]

# x86vbox的产品定义Makefile

如我们所知，AOSP构建系统会查找`AndroidProducts.mk`以找到特定设备的产品定义Makefile。让我们回顾一下产品定义Makefile `x86vbox.mk`，如下所示：

[PRE2]

如我们所见，产品定义Makefile非常简单。它执行以下操作：

+   它包含了通用的x86产品定义Makefile，`device/generic/common/x86.mk`

+   它定义了产品定义变量，例如`PRODUCT_NAME`、`PRODUCT_BRAND`、`PRODUCT_DEVICE`、`PRODUCT_MODEL`等

+   它指定了如何为x86vbox构建内核。

它看起来甚至比我们在[第4章](f69f330a-932c-4a32-bb78-e427c860b65e.xhtml)中创建的*x86emu的Android模拟器*的定制版本还要简单。继承的`x86.mk` Makefile做了大部分实际工作，我们将在稍后进行更深入的分析。

# x86vbox的板级配置

我们将为x86vbox创建的另一个Makefile是板级配置Makefile `BoardConfig.mk`，如下所示：

[PRE3]

它看起来也很简单。它定义了目标架构的特定变量`TARGET_ARCH`、`TARGET_CPU_ABI`、`TARGET_CPU_ABI_LIST_32_BIT`和`TARGET_CPU_ABI_LIST`。然后它定义了系统映像文件的参数。最后，它包含了通用的板级配置Makefile `device/generic/common/BoardConfig.mk`，我们稍后会查看这个文件。

# 常见的x86设备

在Android-x86项目中，它定义了一个通用的x86设备，以便每个人都可以基于它创建特定的x86设备。继承的设备可以是32位或64位的x86设备。

我们可以先查看`device/generic/common`的内容，如下所示：

![图片](img/image_08_001.png)

我们可以看到有很多文件和目录。我们将首先从`BoardConfig.mk`和`x86.mk` Makefile开始分析。

在`BoardConfig.mk`中，构建系统所需的变量定义如下：

[PRE4]

这是一个长长的列表。它定义了音频、Wi-Fi、GPU和蓝牙相关特性。它也是一个禁用的模拟器相关构建。

现在，让我们看看`x86.mk`：

[PRE5]

在`x86.mk`中，它从AOSP构建系统中包含了两个通用Makefile，`full.mk`和`locales_full.mk`。如果我们回想一下x86emu的设备定义Makefile，它也从构建系统中包含了这两个Makefile。

`x86.mk`还导入了另外两个本地Makefile，`device.mk`和`packages.mk`。在`packages.mk`中，HAL模块包定义如下：

[PRE6]

这并不是包的完整列表。在`device.mk`中，还有更多组件被添加到`PRODUCT_PACKAGES`中，如下所示：

[PRE7]

在`device.mk`中，它定义了x86设备的属性，并随后列出了要复制的文件列表。最后，它包括各种组件的单独Makefile，例如固件、触摸屏校准工具、音频、GPS、传感器和本地桥接器等。你可以自己分别在每个相应的文件夹中找到并调查它们。在本章中，我们只概述了如何创建x86vbox设备。我们将在后面的章节中深入探讨一些硬件接口的细节。

# 获取源代码和构建x86vbox设备

要构建x86vbox设备，我们可以使用以下命令从GitHub和AOSP获取源代码：

[PRE8]

使用`android-7.1.1_r4_ch08_aosp`标签作为本章更改的基线。

在我们获取本章的源代码后，我们可以设置环境和构建系统，如下所示：

[PRE9]

# 启动过程和设备初始化

由于我们使用Android-x86内核和HAL为x86vbox，我们将进一步分析本节中x86vbox的启动过程。通过分析，我们可以了解Android-x86是如何使用一个代码库支持多个设备的。你可以回顾我们在[第6章](b6a6462f-1c3d-46fd-89e9-a543423c576d.xhtml)，*使用自定义ramdisk调试启动过程*中讨论的两个阶段启动过程。现在，我们将在此基础上进行更详细的分析。

Android-x86的内核与我们[第6章](b6a6462f-1c3d-46fd-89e9-a543423c576d.xhtml)，*使用自定义ramdisk调试启动过程*中用于模拟器的内核不同。Android-x86内核并不知道它需要支持哪些硬件接口，因此它尽可能多地构建与它相关的设备驱动程序。另一方面，goldfish内核知道它需要支持哪些硬件。这种差异意味着它们是以两种不同的方式构建的。goldfish内核包含了内核内部支持的所有设备，因此它根本不使用内核模块。然而，对于Android-x86内核来说，这样做是不可能的，因为这会使内核的大小变得过大。Android-x86的内核广泛使用内核模块。

我们在本章中将重点分析设备节点是如何在启动过程中创建的，以及内核模块是如何加载的。由于Android-x86的启动分为两个阶段，设备初始化也被分为两个阶段。

# Android启动前的设备初始化

启动过程将从嵌入式Linux环境作为第一阶段开始。大多数设备将在这一阶段进行初始化。好事是，Android-x86可以通过定义的环境变量进入一个带有调试控制台的控制台环境。在这个控制台中，我们可以检查系统状态，以确定我们是否拥有我们想要创建的正确配置。默认的init脚本包含两个调试检查点。第一个检查点是在根设备挂载之后。第二个检查点是在所有驱动程序加载之后。当然，你可以通过更改init脚本来设置尽可能多的检查点。

以下是我们进入第一个检查点之前想要查看的init脚本的一部分：

[PRE10]

在早期启动阶段，init脚本使用内核挂载`/proc`和`/sys`文件系统。之后，它设置`busybox`的符号链接，以便我们可以使用所有`busybox`命令。然后，它将`/sbin/mdev`设置为热插拔的处理程序。`mdev`命令是`udev`的最小实现。`mdev`可以在内核检测到新设备时动态管理`/dev`下的设备节点。`mdev`是`busybox`的一部分，因此我们需要首先创建所有`busybox`符号链接。它还需要`/proc`和`/sys`文件系统。设置热插拔后，脚本运行`mdev -s`命令以找到内核当前检测到的所有设备。此时，`/dev`下的所有设备节点都已创建。

udev和**mdev** **udev**是Linux内核的设备管理器。作为devfsd和hotplug的后继者，udev主要管理`/dev`目录下的设备节点。同时，udev还处理在硬件设备添加到系统或从系统中移除时产生的所有用户空间事件，包括某些**设备**所需的固件加载。

**mdev**是`busybox`中udev的最小实现。它用于嵌入式系统以替换udev。mdev在udev中缺少一些功能，例如设备驱动程序加载的完整实现等。我们可以看到，Android-x86在第一次启动阶段使用mdev。

让我们看看这个阶段的内核模块和设备节点：

![](img/image_08_002.png)

内核模块和设备节点

如前一个截图所示，所有设备节点都创建在`/dev`下。然而，此时只加载了一个内核模块。我们现在处于第一个检查点。

让我们继续前进，看看在达到下一个检查点之前的脚本中会发生什么。要退出第一个检查点，我们需要运行`exit`命令以继续执行脚本。

[PRE11]

在退出第一个检查点后，它将继续执行以下脚本：

[PRE12]

我们可以看到，在进入下一个检查点之前，init脚本执行以下任务：

1.  加载内核模块。

1.  挂载数据分区。

1.  挂载SD卡。

1.  设置触摸屏校准工具。

1.  设置屏幕DPI。

1.  执行任何其他启动后的检测。

您可以自行学习任务2到6的脚本，因为它们非常直接且易于理解。我们在这里想更详细地看看第一个任务：

[PRE13]

`load_modules`脚本函数在前面代码片段所示的脚本文件`0-auto-detect`中实现。它调用另一个函数`auto-detect`来完成实际工作。这个函数并不容易理解。现在让我们解释一下它做了什么。这个函数的目的是动态创建一个名为`dev2mod`的shell命令。这个`dev2mod`函数的作用是接受模块别名作为参数，并根据模块别名加载相应的驱动模块。在创建`dev2mod`命令后，`auto_detect`将使用在`/sys/bus`文件夹下由内核找到的设备调用此函数。

Android-x86内核的所有内核模块都可以在`/lib/modules/4.x.x-android-x86/modules.alias`文件中找到。此文件被处理，以便在每行的末尾添加`modprobe`命令，以便可以使用模块别名作为参数加载内核模块。临时脚本文件位于`/tmp/dev2mod`，如下所示代码片段：

[PRE14]

在将`/sys`文件系统中的设备传递给`dev2mod`函数之前，我们可以查看在我的系统上输出看起来如下：

[PRE15]

如前所述输出所示，它包括了内核找到的所有模块别名。模块别名的先前输出将通过管道传递给shell脚本函数`dev2mod`。`dev2mod`函数将加载内核找到的所有相应模块。

在执行`load_modules`之后，我们进入第二个检查点，现在我们可以查看系统状态：

![图片](img/image_08_003.png)

内核模块加载

从前述截图我们可以看到，现在系统中加载了许多内核模块。从内核模块名称中，我们可以看到音频、鼠标和键盘驱动程序已被加载。这就是Android-x86 init脚本在`initrd.img`中自动加载设备驱动程序的方式。在init脚本末尾，它将根据环境变量`DEBUG`的设置调用`chroot`或`switch_root`。在任一情况下，根文件系统将更改为Android的`ramdisk.img`，并启动Android init进程，如下所示：

[PRE16]

Android init进程将为这些内核无法自动检测到的设备执行硬件初始化。init进程还将初始化Android-x86的HAL。

# Android启动时的HAL初始化

让我们更详细地探讨一下内核无法自动检测到的设备的硬件初始化，以及Android-x86 HAL的初始化，本节将进行介绍。尚未初始化的一个外围设备是Android图形用户界面的帧缓冲区。我们将以此为例，解释Android的`ramdisk.img`中init进程是如何初始化硬件的。

如果我们回顾第 6 章 [调试启动过程使用自定义的 ramdisk](b6a6462f-1c3d-46fd-89e9-a543423c576d.xhtml) 中对 init 进程的分析，init 进程将执行 `init.rc` 脚本，这是适用于所有 Android 设备的通用脚本。在 `init.rc` 脚本中，它将导入特定设备的脚本 `init.${ro.hardware}.rc`。在我们的案例中，这个脚本是在目标设备上的 `init.x86vbox.rc`。`ro.hardware` 属性根据内核命令行参数 `androidboot.hardware` 设置，我们将其设置为 `x86vbox`。`init.x86vbox.rc` 的源代码可以在 `device/generic/common/init.x86.rc` 中找到。它通过 `device.mk` 中的以下行复制到目标输出。请注意，脚本名称在复制后已更改：

[PRE17]

从前述代码片段中我们还可以看到，shell 脚本 `init.sh` 也被复制到了系统镜像的 `/system/etc/init.sh` 路径下。这是在 `init.x86vbox.rc` 中用于加载设备驱动程序和初始化 HAL 的脚本。

在 `init.x86vbox.rc` 文件中，一个动作触发器被定义为如下：

[PRE18]

在预定义的触发器 `post-fs` 中，`init.sh` 脚本将作为初始化过程的一部分执行。以下为 `init.sh` 的代码片段：

[PRE19]

如前述代码片段所示，`init.sh` 脚本首先处理内核命令行。之后，它遇到一个多选择语句。根据传递给它的第一个参数执行一个函数。这个参数用于让 `do_init` 函数初始化特定的 HAL 模块。在第一个参数的情况下，它是 `init` 或没有参数，将执行 `do_init` 函数。在这种情况下，所有 HAL 模块都将被初始化，这是我们目前想要调查的情况。我们可以如下查看 `do_init` 函数做了什么：

[PRE20]

`do_init` 函数将逐个调用各个 HAL 模块初始化函数。我们在这里不会查看所有这些函数。我们将看看在 `init_hal_gralloc` 函数中如何初始化帧缓冲区设备。这是我们将在第 10 章 *启用图形* 中更深入调查的内容，因为图形支持在移植过程中是最重要的任务之一：

[PRE21]

在 `init_hal_gralloc` 函数中，它将根据 `/proc/fb` 的内容执行相应的任务。从 `/proc/fb`，它可以检测设备上图形硬件的类型。如果无法检测到图形硬件的类型，它将使用通用的 VESA 帧缓冲区（uvesafb），在我们的案例中用于 VirtualBox。它将调用另一个 shell 函数 `init_uvesafb` 来加载 VESA 帧缓冲区驱动程序。uvesafb 驱动程序将启动一个用户空间守护进程 `v86d` 来执行 x86 BIOS 代码。代码在受控环境中执行，并通过 netlink 接口将结果传回内核。这就是在我们的环境中初始化图形驱动程序的方式。

# 摘要

在本章中，我们分析了Android-x86 HAL并将其集成到x86vbox中，以便我们能够在接下来的几章中启动x86vbox。我们还分析了Android-x86的启动过程。在启动过程的第一个阶段，我们使用了调试控制台来分析内核模块加载过程。在我们实际上在VirtualBox上启动x86vbox之前，一个尚未解决的问题是我们应该使用哪个引导加载程序。与模拟器不同，它不需要引导加载程序，因为模拟器使用内置的迷你引导加载程序来加载内核和ramdisk。VirtualBox非常类似于真实硬件。如果没有合适的引导加载程序，我们将无法启动操作系统。

在下一章中，我们将讨论这个问题，并解释我们如何使用VirtualBox支持的PXE引导来解决它。
