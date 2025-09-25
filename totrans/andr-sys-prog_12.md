# 介绍恢复

在这本书中，我们到目前为止已经完成了两个项目。通过第一个x86emu项目，我们学习了如何扩展现有设备以支持附加功能。之后，我们通过第二个项目x86vbox学习了如何创建新设备。在Android的系统级编程中，还有一个重要的话题，那就是如何修补或更新已发布的系统。

在Android系统中，修补或更新已发布系统的方法是使用一个名为**恢复**的工具。在接下来的三个章节中，我们将学习如何在x86vbox设备上构建恢复。由于x86vbox是为VirtualBox构建的，我们将使用VirtualBox作为本章的虚拟硬件，[第14章](32a9d777-755e-4758-a38a-f4e11d8ca2e9.xhtml)，*创建OTA更新包*。我们还将使用我们构建的恢复准备和测试几个更新包。在本章中，我们将涵盖以下主题：

+   恢复介绍

+   分析恢复源代码

+   为x86vbox构建恢复

# 恢复介绍

在Android中，恢复是一个包含内核和专用ramdisk的最小Linux环境。当这个最小Linux环境启动时，它会运行一个二进制工具，即恢复，以进入所谓的恢复模式。恢复模式的Linux内核和ramdisk通常存储在一个专用的可启动分区中。在恢复模式下，内核和根文件系统都在内存中，因此它可以管理其他分区而无需任何依赖。

在现场更新设备有两种方法。第一种方法是使用引导加载程序中的fastboot协议。设备可以通过引导加载程序进行重写。在这种情况下，您可以在fastboot模式下启动您的设备，并使用Android SDK中的fastboot工具来重写您的设备。第二种方法是通过恢复模式来重写设备。如果您将设备启动到恢复模式，您可以使用存储上的映像文件或通过USB在sideload模式下提供映像来重写设备。

引导加载程序和恢复可以使用的映像文件不同。来自AOSP构建输出的映像文件可以直接由引导加载程序使用。我们可以直接使用`fastboot`工具在引导加载程序中重写`system.img`、`userdata.img`、`boot.img`或`recovery.img`映像文件。我们不能使用这些映像文件进行恢复。我们必须使用AOSP中提供的工具专门构建恢复映像文件。我们将在下一章中介绍这个主题。

与快启协议相比，恢复模式的关键优势在于支持**空中**（**OTA**）更新。如果OTA服务器上有可用的更新，用户将收到通知。用户可以将更新下载到缓存或数据分区。在更新包使用其签名进行验证后，用户可以响应更新通知。之后，设备将重启进入恢复模式。在恢复模式下，将启动恢复二进制文件，并使用存储在`/cache/recovery/command`文件中的命令行参数来查找更新包以更新系统镜像。

# Android设备分区

要在设备上启用恢复，我们需要再次查看设备分区。在Android SDK中，我们有以下可用于模拟器的镜像文件：

[PRE0]

启动模拟器后，我们可以看到以下分区已挂载：

[PRE1]

我们可以看到`system`、`data`和`cache`分区已挂载为virtio块设备。由于virtio是网络和磁盘设备驱动程序的虚拟化标准，性能应该优于物理设备驱动程序。仅使用这些分区，我们无法创建一个可以使用恢复工具的系统。在下图中，这些是我们需要在存储设备上拥有的最小分区，以支持快启和恢复：

![图片](img/image_12_001.png)

Android设备分区

+   **引导**：这是包含内核和ramdisk镜像的分区。

+   **系统**：这是包含Android系统的分区。它通常作为只读挂载，只能在OTA更新期间进行更改。

+   **厂商**：这是包含厂商私有系统文件的分区。它与系统分区类似，作为只读挂载，只能在OTA更新期间进行更改。

+   **用户数据**：此分区包含用户安装的应用程序保存的数据。此分区通常不会在OTA更新过程中被触及。

+   **缓存**：此分区持有临时数据。OTA包安装可以使用它作为工作空间。

+   **恢复**：此分区包含用于恢复的Linux内核和ramdisk。它与引导分区类似，只是ramdisk镜像仅用于恢复模式。

+   **杂项**：此分区由恢复用于在不同引导会话之间存储信息。

在本章中，我们将为x86vbox设备构建恢复。正如我们从[第8章](acf2363a-2a0f-40b9-a35f-c8bb0e523737.xhtml)“在VirtualBox上创建自己的设备”到[第11章](3c6453e9-98bb-4979-9c61-f0df071b1255.xhtml)“启用VirtualBox特定硬件接口”所学的，我们只为x86vbox设备使用一个分区来存储所有内容。我们将在本章前面的解释基础上，稍后扩展x86vbox设备以使用多个分区。

# 分析恢复

在我们开始为我们的x86vbox设备构建恢复之前，我们将分析恢复的代码流程以了解其工作原理。从最终用户的角度来看，有两种进入恢复模式的方法。当用户想要执行出厂重置或可用OTA更新时，主系统可以在重置系统之前将恢复命令写入**引导加载程序控制块**（**BCB**）和缓存分区。

进入恢复模式的第二种方法是手动使用键组合。在关闭手机后，同时按下一个键组合以手动进入恢复模式。键组合由设备制造商定义，例如，它可以是音量下键和电源按钮的组合。

在这两种情况下，进入恢复模式与引导加载程序的实现密切相关。Android系统、恢复和引导加载程序通过两个接口相互通信：分区`/cache`和`/misc`。我们可以使用以下图表来描述通信接口：

![图片](img/image_12_002.png)

Android系统、恢复和引导加载程序的接口

在前面的图表中，引导加载程序使用`/misc`分区中的**BCB**与Android系统和恢复进行通信。Android系统和恢复使用`/cache`分区中的信息相互通信。让我们深入了解这两个通信通道的细节。

# BCB

BCB是引导加载程序与主系统和恢复之间的通信接口。

Android系统在恢复源代码中也被称为主系统。在本章中，我们将主系统一词与Android系统视为同义词。

BCB以原始分区格式存储在`/misc`分区中，这意味着这个分区就像一个没有文件系统的二进制文件一样被使用。

恢复使用`recovery.fstab`文件挂载系统中的所有分区。如果我们查看`recovery.fstab`中`/misc`分区的文件系统类型，它是`emmc`，这是恢复中使用的原始文件系统之一：

[PRE2]

恢复支持五种文件系统类型，包括两种原始文件系统和三种正常文件系统。

支持的两个原始文件系统是：

+   `mtd`：这是旧版Android设备中使用的分区。这些设备使用NAND闪存和MTD分区。

+   `emmc`：这是最近Android设备中使用的原始eMMC块设备。

`boot`、`recovery`和`misc`分区可以是`mtd`或`emmc`文件系统类型。

支持的正常文件系统类型包括：

+   `yaffs2`：`yaffs2`文件系统通常用于MTD设备的`system`、`userdata`或`cache`分区。这通常用于较老的Android设备。

+   `ext4`：在最新的Android设备中，使用eMMC块设备。通常在eMMC块设备上使用标准的Linux `ext4`文件系统。与`yaffs2`文件系统类型一样，`system`、`userdata`或`cache`分区可以使用`ext4`格式。

+   `vfat`：这是用于外部存储（如SD卡或USB）的文件系统类型。

让我们回到BCB的话题。在`$AOSP/bootable/recovery/bootloader.h`文件中，BCB被定义为以下数据结构：

[PRE3]

当主系统想要将设备重启到恢复模式时，会使用`command`字段。这可能是用户从设置中选择出厂重置或可用OTA更新时的情况。引导加载程序也可以使用此字段，当引导加载程序完成固件更新时，它可能想要进入恢复模式进行最后的清理。

系统引导加载程序在完成固件更新后更新`status`字段。

`recovery`字段由主系统用于向恢复发送消息，或者恢复可能使用此字段向主系统发送消息。

`stage`字段用于指示更新的阶段。在某些情况下，安装更新包可能需要多次重启。恢复用户界面可以使用此字段来显示安装的当前阶段：

![](img/image_12_003.png)

在前面的图中，显示与检查键组合和BCB相关的引导加载程序逻辑。只要引导加载程序根据AOSP恢复定义处理BCB，实现可以是供应商特定的。通常，引导加载程序首先检查键组合，以决定用户是否想要进入恢复模式。如果没有按下键组合，它将检查BCB以决定引导路径。

# 缓存分区

缓存分区中有三个文件，可以作为主系统和恢复工具之间的通信通道。这三个文件是：

+   `/cache/recovery/command`：这是一个从恢复点输入参数的文件。此文件中每行有一个命令。可能在此文件中提供的参数是：

    +   `-send_intent=anystring`：主系统可能使用此命令在恢复退出后向自身发送消息

    +   `-update_package=path`：此命令指定安装OTA包文件的路径

    +   `-wipe_data`：此命令指示恢复擦除用户数据（和缓存），然后重启

    +   `-wipe_cache`：此命令指示恢复擦除缓存（但不擦除用户数据），然后重启

    +   `-set_encrypted_filesystem=on|off`：启用/禁用加密文件系统

    +   `-just_exit`：不执行任何操作；退出并重启

+   `/cache/recovery/log`：恢复的运行时日志文件位于`/tmp/recovery.log`。在恢复退出之前，它将备份旧日志文件并将当前日志文件移动到`/cache/recovery/log`。

+   `/cache/recovery/intent`：在恢复退出之前，它将检查是否有任何需要通过此文件发送到主系统的意图。意图可以是主系统通过`/cache/recovery/command`文件中的`-send_intent`命令发送给恢复的消息。

# 恢复的主流程

在我们了解了关于恢复以及与恢复相关的所有背景知识后，让我们看看恢复的主要工作流程。我们将使用以下流程图来探索恢复的工作流程：

1.  当开始恢复时，它首先将日志文件设置为 `/tmp/recovery.log`。

1.  之后，它检查 `--adbd` 选项。如果指定了此选项，它将运行用于侧载的 `adb` 守护程序。您可以参考 `$AOSP/bootable/recovery/adb_install.cpp` 中的源代码，了解如何以 `adb` 守护程序启动恢复。

1.  它通过调用 `get_args` 函数从缓存分区和 BCB 中检索和处理参数。

1.  根据 `get_args` 从中检索到的命令，它可能会调用 `install_package` 函数来安装更新，或者调用 `wipe_data` 或 `wipe_cache` 函数来擦除用户数据或缓存分区。

1.  如果没有更新包或擦除数据的命令，它将调用 `prompt_and_wait` 函数进入恢复用户界面。根据用户输入，它可能会调用 `apply_from_adb` 或 `apply_from_sdcard` 从 USB 或 SD 卡更新包。它可能会调用 `wipe_data` 或 `wipe_cache` 函数来擦除用户数据或缓存分区等。

1.  在所有任务完成或用户选择退出恢复后，它将调用清理函数 `finish_recovery` 来进行最后的清理。之后，它将重新启动或关闭系统：

![](img/image_12_004.png)

恢复工作流程

根据前面的流程分析，我们可以查看 `$AOSP/bootable/recovery/recovery.cpp` 中的 `main` 函数的代码片段如下：

[PRE4]

在我们了解了恢复工作流程的概述后，我们将看看恢复如何在 `get_args` 函数中从 BCB 或缓存文件中检索参数。之后，我们将从用户的角度查看两个重要的工作流程：工厂重置和 OTA 更新。

# 从 BCB 和缓存文件中检索参数

如我们在恢复的主要函数中看到的那样，它调用了 `get_args` 函数从主系统或引导加载程序中检索参数。以下是为 `get_args` 的流程图。它与恢复的 `main` 函数位于同一 `$AOSP/bootable/recovery/recovery.cpp` 文件中。

![](img/image_12_005.png)

get_args 流程图

从以下代码片段中，我们可以看到它调用了 `get_bootloader_message` 函数来获取 BCB 数据结构 `boot`：

[PRE5]

如果没有参数，`argc` 的值将小于或等于 1。它将尝试从 BCB 中获取参数，如下面的代码片段所示。在 BCB 的 `recovery` 字段中，命令将以 `recovery\n` 开始。`recovery\n` 之后的内容与缓存命令文件格式相同，即 `/cache/recovery/command`：

[PRE6]

如果可以从 BCB 中检索到参数，它将跳过缓存命令文件。否则，它将尝试按照以下方式从缓存命令文件中读取参数：

[PRE7]

在处理完BCB和缓存命令文件后，它将BCB块写入`/misc`分区，以便在更新或擦除过程中出现任何错误时，重启后相同的进程将继续：

[PRE8]

从前面的代码分析中，我们可以看到缓存命令文件只是一个普通的文本文件。它可以通过使用标准的C函数来访问。要访问BCB数据结构的`/misc`分区，使用`get_bootloader_message`函数读取BCB，并使用`set_bootloader_message`函数写入BCB。BCB数据结构`bootloader_message`在`bootloader.h`文件中定义，相关函数在`bootloader.cpp`文件中实现。

`/misc`分区是一个原始分区，它被`bootloader.cpp`中的代码作为一个普通文件而不是文件系统卷来使用。

我们可以快速查看`get_bootloader_message`函数及其支持函数`get_bootloader_message_block`，如下所示：

[PRE9]

在`get_bootloader_message`函数中，它将根据分区类型调用另一个函数，`/misc`。正如我们所见，支持的原始文件系统类型是`mtd`和`emmc`。我们可以查看`emmc`版本的`get_bootloader_message_block`，如下所示：

[PRE10]

正如我们所见，在`get_bootloader_message_block`函数中，它使用C函数`fopen`、`fread`和`fclose`将`/misc`分区作为普通文件访问。

现在我们已经完成了BCB和缓存文件处理的分析。在接下来的两节中，我们将探讨恢复过程中最重要的两个工作流程：

+   工厂数据重置

+   OTA更新

# 工厂数据重置

恢复的主要功能之一是支持工厂数据重置。通常，用户可以从设备上的设置中选择工厂数据重置，如下面的截图所示：

![图片](img/image_12_006.png)

工厂数据重置

整个过程可以分为以下步骤：

1.  用户从设置中选择工厂数据重置。

1.  主系统将`--wipe_data`写入`/cache/recovery/command`。

1.  主系统重新启动设备进入恢复模式。我们已经在上一节讨论BCB时对此进行了分析。

1.  恢复从BCB或`/cache/recovery/command`中的`get_args()`检索参数。在读取参数后，恢复将使用`boot-recovery`和`--wipe_data`写入BCB。

1.  恢复擦除`/data`和`/cache`分区。在此之后，任何后续的重启将继续此步骤，直到擦除完成或用户从恢复用户界面采取其他操作退出恢复。

1.  在擦除`/data`和`/cache`分区后，恢复调用`finish_recovery`函数来擦除BCB。

1.  恢复将设备重新启动到主系统。

我们已经分析了前面的大多数步骤，除了`finish_recovery`。让我们看看`finish_recovery`函数：

[PRE11]

在`finish_recovery`函数中，它将意图写入`/cache/recovery/intent`。然后，它处理本地文件并创建日志文件备份。最后，通过调用`set_bootloader_message`擦除BCB，并删除`/cache/recovery/command`以恢复正常的启动过程。

# OTA更新

OTA更新是恢复的另一个主要功能。在手动进入恢复模式后，可以使用恢复用户界面更新OTA包。在接收到更新通知后也可以自动更新。在这两种情况下，更新包的路径可能不同，但安装过程是相同的。在本节中，我们将查看设备接收到OTA更新通知后的流程。然后，我们将探讨安装过程的细节：

1.  设备接收到OTA更新通知后，主系统将OTA包下载到`/cache/update.zip`。

1.  主系统将`--update_package=/cache/update.zip`命令写入`/cache/recovery/command`。

1.  主系统将设备重新启动到恢复模式。

1.  恢复过程在`get_args()`中从BCB或`/cache/recovery/command`检索参数。读取参数后，恢复过程将使用`boot-recovery`和`update_package=...`写入BCB。

1.  恢复过程调用`install_package`来安装更新。在此步骤中，任何后续的重启将继续此步骤，直到安装完成。

1.  如果安装失败，将调用`prompt_and_wait`函数来显示错误并等待用户操作。如果安装成功完成，它将进入下一步。

1.  恢复过程调用`finish_recovery`函数来擦除BCB并删除`/cache/recovery/command`文件。

1.  恢复过程将设备重新启动到主系统。

一旦更新包下载完成，安装将由`install_package`函数完成：

[PRE12]

在`install_package`函数中，它首先设置安装日志文件。日志文件路径是`/tmp/last_install`。然后，它调用`setup_install_mounts`来挂载相关分区。实际的安装是在`really_install_package`函数中完成的，如下面的代码片段所示：

[PRE13]

在`really_install_package`函数中，它初始化用户界面并在屏幕上显示包位置。然后，它为更新包创建内存映射，这是`zip`函数所需的。之后，它使用其签名验证更新包。最后，它调用另一个函数`try_update_binary,`来完成安装。

`try_update_binary`函数执行三个任务：

1.  从更新包中提取`update_binary`。

1.  准备环境以执行`update_binary`。

1.  监控安装进度。

让我们详细了解这三个任务：

[PRE14]

它尝试从更新包中提取`update_binary`。`update_binary`在更新包中的路径在`META-INF/com/google/android/update-binary`中预定义。

如果`update_binary`可以成功提取，它将被复制到`/tmp/update_binary`：

[PRE15]

如前述代码片段所示，在提取`update_binary`之后，它将准备环境以执行`update_binary`。更新包的安装实际上是通过脚本由`update_binary`完成的。以下参数被传递给`update_binary`以执行：

+   `update_binary`的路径

+   恢复版本

+   父进程和子进程之间的管道用于通信

+   更新包的路径

在环境准备就绪后，它将派生一个子进程来运行`update_binary`。父进程将通过管道与子进程通信来监控安装进度：

[PRE16]

如前述代码片段所示，父进程将从子进程接收命令以显示进度，打印信息到屏幕，或在安装后设置清理配置。

# 为x86vbox构建恢复

在分析恢复源代码的工作流程和关键元素后，我们现在可以开始为我们的x86vbox设备构建它。

支持恢复构建的更改包括对x86vbox设备的更改以及对`recovery`和`newinstaller`的更改。

# 构建配置

在我们查看本章的更改之前，让我们先看看配置文件。像往常一样，我们为每个章节都有一个清单文件。我们根据[第11章](3c6453e9-98bb-4979-9c61-f0df071b1255.xhtml)的源代码，*启用VirtualBox特定的硬件接口*进行本章的更改。以下是我们将要更改的项目：

[PRE17]

我们可以看到我们需要更改四个项目：`recovery`、`newinstaller`、`common`和`x86vbox`。我们使用`android-7.1.1_r4_x86vbox_ch12_r1`标签作为本章源代码的基线。

我们可以使用以下命令从GitHub和AOSP获取源代码：

[PRE18]

在我们获取本章的源代码后，我们可以设置环境并按如下方式构建系统：

[PRE19]

要构建`initrd.img`，您可以运行以下命令：

[PRE20]

# x86vbox的更改

对于x86vbox设备，我们首先需要更改Makefiles设备。由于我们从通用的Android-x86设备继承了x86vbox，所以我们只有以下Makefiles：

[PRE21]

`AndroidProducts.mk`是Android构建系统的入口，它包括我们的`x86vbox.mk` Makefile。在`x86vbox.mk`中，我们添加以下与恢复相关的文件：

[PRE22]

这些更改包括两部分。第一部分与针对VirtualBox的特定环境设置相关，因为我们是在VirtualBox的虚拟硬件上运行恢复。x86vbox特定的初始化脚本`init.recovery.x86vbox.rc`将在系统启动时由init进程执行。

第二部分与存储设备的分区有关。正如我们在前面的章节中讨论的，我们不能像在[第8章](acf2363a-2a0f-40b9-a35f-c8bb0e523737.xhtml)，*在VirtualBox上创建自己的设备*，到[第11章](3c6453e9-98bb-4979-9c61-f0df071b1255.xhtml)，*启用VirtualBox特定的硬件接口*中那样，使用单个分区进行恢复。分区表定义在 `recovery.fstab` 文件中。让我们首先看看启动脚本，`init.recovery.x86vbox.rc`：

[PRE23]

作为Android的init脚本，恢复也有一个针对特定设备的init脚本，`init.recovery.${ro.hardware}.rc`。在我们的例子中，它是 `init.recovery.x86vbox.rc`。在 `init.recovery.x86vbox.rc` 中，它调用Android-x86 HAL初始化脚本，`/system/etc/init.sh`。在[第8章](acf2363a-2a0f-40b9-a35f-c8bb0e523737.xhtml)，*在VirtualBox上创建自己的设备*中的Android启动部分的HAL初始化过程中，我们详细解释了 `/system/etc/init.sh` 脚本。

我们在 `init.recovery.x86vbox.rc` 中添加了两个服务，`network_start` 和 `console`。有了这两个服务，我们能够启用VirtualBox特定的网络接口，并且在启动后也可以拥有一个控制台。有了这个调试控制台，我们可以在本书的后续部分更容易地调试恢复过程。

`x86vbox.mk` 中的另一个重要部分是我们为恢复添加了一个 `recovery.fstab` 分区表，如下所示：

[PRE24]

如我们所见，我们现在有六个分区。我们实际上并没有一个支持fastboot协议和恢复BCB的引导加载程序，所以我们实际上并不使用 `/boot` 和 `/recovery` 分区。然而，我们确实有一个来自Android-x86的两阶段引导过程，并且我们可以有一个无需引导加载程序支持的解决方案。我们将在本章稍后查看对 `newinstaller` 的更改时看到这一点。

`recovery.fstab` 分区表由恢复使用，我们需要更改Android主系统的相关分区表，即 `device/generic/common/fstab.x86` 中的文件。

我们需要在 `device/generic/common/fstab.x86` 中添加两个条目，如下所示：

[PRE25]

这个 `fstab.x86` 文件将在构建过程中被复制到系统镜像中，作为 `fstab.x86vbox`。init进程将处理它以挂载分区。你可能想知道为什么分区表中没有 `/system` 和 `/data`。我们使用两阶段引导，它们在Android启动之前的第一阶段引导中挂载。`/system` 和 `/data` 的来源可以通过内核参数进行配置，正如我们在前面章节中解释两阶段引导过程时讨论的那样。

请注意，恢复和主系统应该挂载相同的块设备分区。例如，如果恢复和主系统为 `/cache` 挂载不同的分区，它们将无法通过 `/cache/recovery/command` 中的命令文件相互通信。

这就是`x86vbox.mk`更改的全部内容，现在让我们看看另一个Makefile，`BoardConfig.mk`。为了启用恢复的构建，我们需要在`BoardConfig.mk`中添加以下两个宏：

[PRE26]

这两个宏的默认值都设置为true，这意味着内核和恢复在默认配置中都没有内置。

我们添加了另一个与恢复源代码更改相关的宏，我们将在稍后查看源代码更改：

[PRE27]

`RECOVERY_GRAPHICS_FORCE_SINGLE_BUFFER`宏是从**Team Win Recovery Project**（**TWRP**）的最新代码中借用的。随着x86vbox Makefiles的更改，我们实际上可以构建TWRP。这是一个许多第三方ROM（如LineageOS/CyanogenMod、Omnirom等）常用的第三方恢复工具。

# 恢复更改

AOSP恢复代码在VirtualBox上可以很好地工作。只有一个与显示相关的问题。为了修复显示问题，我们需要更改恢复源代码中的两个文件。

我们使用前面提到的`RECOVERY_GRAPHICS_FORCE_SINGLE_BUFFER`宏来配置帧缓冲区更改。我们首先需要将其添加到恢复Makefile `minui/Android.mk`中，如下所示：

[PRE28]

由于双缓冲在VirtualBox上目前无法很好地工作，我们必须将其禁用如下：

[PRE29]

通过对TWRP的类似更改，TWRP也可以为x86vbox构建。构建TWRP的分支包含在GitHub的源代码中，您可以自己尝试。

# 新安装器更改

如我们在BCB部分所讨论的，引导加载程序根据BCB中存储的参数来决定引导路径。BCB中存储的恢复命令与`/cache`分区中的`/cache/recovery/command`相同。实际上，我们可以将相同的逻辑移动到`initrd.img`的第一阶段引导中。在这种情况下，我们可以借助第一阶段引导实现相同的结果。工厂数据重置和OTA更新的逻辑将变为以下步骤：

1.  用户可以选择工厂数据重置或可用的OTA更新。

1.  主系统将命令`--wipe_data`或`--update_package=/cache/update.zip`写入`/cache/recovery/command`。

1.  主系统重新启动设备。

1.  在第一阶段引导中，init脚本将检查`/cache`分区中是否存在`/cache/recovery/command`文件。

1.  如果`/cache/recovery/command`存在，它将加载`ramdisk-recovery.img`，否则，它将加载`ramdisk.img`。

1.  其余步骤将与正常引导过程或恢复引导过程相同。

为了实现前面的逻辑，我们在`$AOSP/bootable/newinstaller/initrd/init`文件中添加了一个shell函数`find_ramdisk`，如下所示：

[PRE30]

在这个函数中，我们将缓存分区挂载到`/hd`，并检查`/hd/recovery/command`是否存在。如果存在，我们将`RAMDISK`变量设置为`ramdisk-recovery.img`；否则，我们将其设置为`ramdisk.img`。init脚本将在稍后提取`RAMDISK`变量中包含的ramdisk，如下所示：

[PRE31]

还有一个名为`RECOVERY`的变量，它在`find_ramdisk`中定义，可以从内核命令行传递给init脚本。使用这个变量，我们可以强制引导到恢复模式。

# 测试恢复

在我们构建了恢复和AOSP镜像之后，我们可以在VirtualBox中测试它们。正如我们在[第9章](c8d10155-cc8e-4b8c-a5e0-f359520c894a.xhtml)中学到的，*使用PXE/NFS启动x86vbox*，我们可以使用PXE引导系统，并使用NFS访问AOSP镜像。为了测试恢复，我们可以在`$HOME/.VirtualBox/TFTP/pxelinux.cfg/default`文件中添加一个选项，使用`kernel`和`ramdisk/recovery.img`引导。尽管我们现在可以引导系统到恢复模式，但我们无法使用本章中的恢复来更新系统。我们将在接下来的两章中了解更多信息。

# 摘要

我们已经完成了对x86vbox设备的恢复分析和实现。在本章的第一部分，我们分析了恢复源代码中的工作流程和关键元素。在本章的第二部分，我们将第一部分中获得的知识应用于x86vbox设备的恢复实现。我们修改了x86vbox设备本身以添加恢复支持。我们还修改了恢复源代码以修复显示问题。最后，我们修改了newinstaller，以便我们可以在主系统和恢复系统中都有完整的引导流程。

在下一章中，我们将讨论如何创建恢复包，并解释恢复包中包含的内容。
