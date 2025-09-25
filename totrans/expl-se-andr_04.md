# 第四章。在 UDOO 上的安装

为了继续我们的探索，我们需要一个可操作的系统来工作。在本章中，我们将：

+   从源代码构建 UDOO 的 Android 4.3

+   使用我们的引导映像刷写 SD 卡

+   在捕获日志的同时运行 UDOO

+   建立 `adb` 连接到 UDOO

+   使用 SELinux 支持重新构建内核

+   验证我们的 SELinux UDOO 图像按预期工作

我们将使用公开可用的 UDOO Android 4.3 Jelly Bean 源代码开始，该代码可以从[`www.udoo.org/downloads/`](http://www.udoo.org/downloads/)下载。假设您已经拥有 UDOO 并已验证其功能。建议您遵循 UDOO 网站上的说明，以 Android 4.3 预构建映像作为初始测试（更多信息，请参阅[`www.udoo.org/getting-started/`](http://www.udoo.org/getting-started/))。

您还需要一个适合与 Android 和 UDOO 一起工作的适当开发系统，但本章不涉及这些细节。附录提供了一个设置标准 Ubuntu Linux 12.04 系统的详细说明，以确保您有最高概率成功复制本书中的工作。

# 获取源代码

让我们从下载上一节中给出的下载链接中的 Android 4.3 Jellybean 源代码开始这项练习，并使用以下命令将其提取到工作区：

```kt
$ mkdir ~/udoo && cd ~/udoo
$ tar -xavf ~/Downloads/UDOO_Android_4.3_Source_v2.0.tar.gz

```

完成此步骤后，您应查阅以下网址中的 UDOO 文档和 Android 源代码构建说明：

+   [`www.elinux.org/UDOO_compile_android_4-2-2_from_sources`](http://www.elinux.org/UDOO_compile_android_4-2-2_from_sources)

+   [`source.android.com/source/initializing.html`](http://source.android.com/source/initializing.html)

上一 URL 提供的说明讨论了如何使用 Open JDK 7 构建 Android。然而，这些说明适用于当前 Android（L 预览）的版本，并不完全相关。对于 Android 4.3，您必须使用 Oracle Java 6 构建，该版本由 Oracle 归档并在[`www.oracle.com/technetwork/java/javasebusiness/downloads/java-archive-downloads-javase6-419409.html`](http://www.oracle.com/technetwork/java/javasebusiness/downloads/java-archive-downloads-javase6-419409.html)找到。

假设您拥有附录中详细说明的系统副本，即*开发环境*。该附录除了其他内容外，还将指导您设置 Oracle Java 6 作为您的唯一 Java 实例。然而，对于那些更喜欢从现有系统工作，尤其是拥有多个 Java SDK 的系统，请注意，在阅读本书的其余部分时，您需要确保您的系统使用 Oracle Java 6 工具。

通过切换到您的 UDOO 源代码树的根目录并执行以下命令来完成环境设置：

```kt
$ . setup udoo-eng

```

一旦配置了环境，我们需要构建`bootloader`：

```kt
$ cd bootable/bootloader/uboot-imx
$ ./compile.sh -c

```

将出现一个图形菜单。确保设置如下：

+   **DDR 大小**：选择 1 Giga，总线大小 64，和活动 CS \ 1（256Mx4）

+   **板型**：选择 UDOO

+   **CPU 类型**：根据你拥有的系统选择四核或双核选项。我们恰好使用的是四核系统。

+   **操作系统类型**：选择**Android**

+   **环境设备**：必须选择**SD/MMC**

+   **额外选项**：**CLEAN**应该被选中

+   **编译器选项**：可以在此选择工具链的路径；只需采用默认值

以下截图显示了前一个命令显示的图形菜单：

![检索源代码](img/0594OS_04_01.jpg)

当你退出时，请确保保存。然后开始编译：

```kt
$ ./compile.sh 
Board type selected: UDOO
CPU Type: QUAD/DUAL
OS type: Android
...
/home/bookuser/udoo/prebuilts/gcc/linux-x86/arm/arm-eabi-4.6/bin/arm-eabi-objcopy -O srec u-boot u-boot.srec
/home/bookuser/udoo/prebuilts/gcc/linux-x86/arm/arm-eabi-4.6/bin/arm-eabi-objcopy --gap-fill=0xff -O binary u-boot u-boot.bin

```

为了安全起见，通过使用`ls u-boot.bin`来验证`bootloader`镜像现在是否存在。现在，使用以下命令构建 Android：

```kt
$ croot
$ make –j4 2>&1 | tee logz

```

第一个命令是源自 Android 设置脚本中的某个命令，它将我们带回到项目树的根目录。第二个命令`make`构建系统。在大多数情况下，你应该将`j`选项设置为 CPU 核心数的两倍。因为你们中很多人可能拥有双核机器，我们将使用`–j4`。例如，本书的一位作者使用 8 个 CPU 核心，并使用标志`-j16`。文件重定向和`tee`命令将构建输出捕获到文件中。这对于帮助和调试任何构建问题非常重要。这个构建过程，根据你的系统可能需要很长时间。在之前提到的 8 核系统上，16GB RAM，这个过程花费了略超过 35 分钟。在其他系统上，我们经历了超过 3 小时的构建时间。

在这种情况下，捕获日志非常有用。构建以错误终止，通过在日志中搜索`error`，我们找到了以下内容：

```kt
$ grep error logz 
...
external/mtd-utils/mkfs.ubifs/mkfs.ubifs.h:48:23: fatal error: uuid/uuid.h: No such file or directory
external/mtd-utils/mkfs.ubifs/mkfs.ubifs.h:48:23: fatal error: uuid/uuid.h: No such file or directory
external/mtd-utils/mkfs.ubifs/mkfs.ubifs.h:48:23: fatal error: uuid/uuid.h: No such file or directory
...

```

通过评估这些错误，我们发现我们缺少`uuid`和`lzo1x`的头文件。我们还可以打开 Android 的 makefile，`external/mtd-utils/mkfs.ubifs/Android.mk`，并从`LOCAL_LDLIBS:= -lz -llzo2 -lm -luuid -m64`这一行确定可能涉及的库。搜索揭示了我们缺少的特定 Ubuntu 软件包；我们将安装它们并重新构建。搜索字符串末尾的`$`字符确保我们只得到以`uuid/uuid.h`结尾的结果；如果没有它，我们可能会匹配以`.html`或`.hpp`结尾的文件：

$ sudo apt-file search -x "uuid/uuid.h$"

```kt
uuid-dev: /usr/include/uuid/uuid.h
$ sudo apt-get install uuid-dev
$ make –j4 2>&1 | tee logz

```

成功构建应产生一些类似以下内容的最终输出：

```kt
...
Running: mkuserimg.sh out/target/product/udoo/system out/target/product/udoo/obj/PACKAGING/systemimage_intermediates/system.img ext4 system 293601280 out/target/product/udoo/root/file_contexts
Install system fs image: out/target/product/udoo/system.img
out/target/product/udoo/system.img+out/target/product/udoo/obj/PACKAGING/recovery_patch_intermediates/recovery_from_boot.p maxsize=299747712 blocksize=4224 total=294120167 reserve=3028608

```

# 在 SD 卡上闪烁图像

当构建了`bootloader`、Android 用户空间和 Linux 内核后，是时候插入 SD 卡并刷写镜像了。将 SD 卡插入您的宿主机，并确保它已卸载。在 Ubuntu 中，可移动媒体会自动挂载，因此您需要找到您的闪存驱动器对应的`/dev/sd*`设备，并使用`umount`卸载它。在本文的剩余部分，我们将使用`/dev/sdd`作为闪存驱动器，但重要的是要使用您系统中的正确设备。如果您之前已经使用这张 SD 卡安装过 UDOO，该卡将包含多个分区，因此您可能会看到`/dev/sdd<num>`被多次挂载：

```kt
$ mount | grep sdd
/dev/sdd7 on /media/vender type ext4 (rw,nosuid,nodev,uhelper=udisks)
/dev/sdd4 on /media/data type ext4 (rw,nosuid,nodev,uhelper=udisks)
/dev/sdd5 on /media/57f8f4bc-abf4-655f-bf67-946fc0f9f25b type ext4 (rw,nosuid,nodev,uhelper=udisks)
/dev/sdd6 on /media/cache type ext4 (rw,nosuid,nodev,uhelper=udisks)
$ sudo bash -c "umount /dev/sdd4 && umount /dev/sdd5 && umount /dev/sdd6 && umount /dev/sdd7"

```

一旦 SD 卡正确卸载，我们就可以刷写我们的镜像：

```kt
$ sudo -E ./make_sd.sh /dev/sdd

```

### 小贴士

您必须在`sudo`中使用`-E`参数以保留从 Android 构建中导出的所有变量。您必须在与构建 Android 相同的终端会话中。否则，您将看到错误`未找到 OUT 导出变量！未提前调用设置...`。

一旦完成（这将需要一段时间），重要的是使用命令`sudo sync`将块设备缓存刷新回磁盘。然后，您可以取出 SD 卡，将其插入 UDOO，并启动！

# UDOO 串行和 Android 调试桥

现在，UDOO 正在启动到 Android，我们想确保我们可以通过串行端口以及**Android 调试桥**（**adb**）来访问它。您需要适用于您系统的 UDOO 串行驱动程序。有关 Mac、Linux 和 Windows 的详细信息，请参阅

[`www.udoo.org/ProjectsAndTutorials/connecting-via-serial-cable/`](http://www.udoo.org/ProjectsAndTutorials/connecting-via-serial-cable/)。

串行端口是系统将出现的第一种通信方式，它由`bootloader`初始化。它是调试您以后可能遇到的任何内核或系统问题的关键链接。它还必须配置 USB 端口以允许通过 CN3（UDOO 上的 USB OTG 端口）进行`adb`连接。为了配置端口，我们需要配置并使用 minicom 将 shell 连接到设备。首先，将 CN6（靠近电源按钮的 micro USB 端口）的 micro USB 线插入主机。接下来，让我们通过查看`dmesg`来查找 USB TTY 的连接消息，以找到串行连接。

```kt
$ sudo dmesg | tail -n 5
[ 9019.090058] usb 4-1: Manufacturer: Silicon Labs
[ 9019.090061] usb 4-1: SerialNumber: 0078AEDB
[ 9019.096089] cp210x 4-1:1.0: cp210x converter detected
[ 9019.208023] usb 4-1: reset full-speed USB device number 4 using uhci_hcd
[ 9019.359172] usb 4-1: cp210x converter now attached to ttyUSB0

```

我们的 TTY 终端位于最后一行。让我们通过它使用`minicom`进行连接：

```kt
$ sudo minicom -sw

```

选择**串行端口设置**，输入`a`，将**串行设备**更改为`/dev/ttyUSB0`，并输入`f`以关闭硬件流控制：

![UDOO 串行和 Android 调试桥](img/0594OS_04_02.jpg)

要退出，按*Enter*键，选择**保存设置和 DFL**，然后选择**从 Minicom 退出**，并按*Enter*键。现在运行`minicom`以连接到您的 UDOO，并观察其启动：

```kt
$ sudo minicom -w

```

如果设备已启动并运行，您将看到一个友好的 root shell：

![UDOO 串行和 Android 调试桥](img/0594OS_04_03.jpg)

如果正在启动，您将看到日志。只需等待 root shell 提示符：

![UDOO 串行和 Android 调试桥接](img/0594OS_04_04.jpg)

现在我们需要翻转一些 GPIO 引脚，将 CN3 微型 USB 切换到调试模式：

```kt
root@udoo:/ # echo 0 > /sys/class/gpio/gpio203/value 
root@udoo:/ # echo 0 > /sys/class/gpio/gpio128/value 

```

然后，通过移除并更换 J16 跳线来重置使用该总线的 SAM3X8E 处理器。现在，从主机连接到 CN3 的微型 USB 线。你现在应该能看到一个 USB 设备以及`adb`：

```kt
$ lsusb
Bus 001 Device 009: ID 18d1:4e42 Google Inc.
$ adb devices
List of devices attached 
0123456789ABCDEF  offline

```

当 UDOO Android 侧出现提示时，你需要选择**允许 USB 调试**。当你这样做时，设备应该从离线状态变为在线状态；这样你就可以使用`adb`。

现在通过`adb`测试连接并抓取截图：

```kt
$ adb shell
root@udoo:/ # 
$ adb shell screencap -p | perl -pe 's/\x0D\x0A/\x0A/g' > screen.png

```

这是截图：

![UDOO 串行和 Android 调试桥接](img/0594OS_04_05.jpg)

到目前为止，我们有一个工作的开发系统。我们通过串行控制台有早期引导日志和救援 shell。我们还有一个`adb`桥接，我们可以使用标准的 Android 调试工具！现在没有剩下的事情要做，但需要用 SELinux 来确保这个系统的安全性！

# 翻转开关

现在我们正在 UDOO 上启用 SELinux，我们需要验证它没有被打开。要这样做，我们需要检查`/proc`文件系统中的已知`filesystem`类型。SELinux 有自己的伪文件系统，所以如果它被启用，我们应该在列表中看到它：

```kt
$ adb shell cat /proc/filesystems
nodev  sysfs
nodev  rootfs
nodev  bdev
nodev  proc
nodev  cgroup
nodev  cpuset
nodev  tmpfs
nodev  debugfs
nodev  sockfs
nodev  pipefs
nodev  anon_inodefs
nodev  rpc_pipefs
nodev  devpts
 ext3
 ext2
 ext4
 cramfs
nodev  ramfs
 vfat
 msdos
nodev  nfs
nodev  jffs2
nodev  fuse
 fuseblk
nodev  fusectl
nodev  mtd_inodefs
nodev  ubifs

```

这里没有 SELinux 的证据，所以让我们找到内核配置并将其打开。从`~/udoo/kernel_imx`目录执行此命令，最终你将看到一个图形编辑屏幕：

```kt
$ make menuconfig

```

首先，你需要启用**审计支持**，因为这是 SELinux 的依赖项。在**通用设置** | **审计支持**下，启用**审计支持**和**启用系统调用审计**。使用上箭头和下箭头键突出显示一个条目，并按空格键启用它。当一个项目被启用时，你将看到它旁边有一个星号（****）：

![翻转开关](img/0594OS_04_06.jpg)

通过选择**退出**回到主菜单...这不太直观。进入**文件系统**菜单，并为每个三个文件系统**Ext2**、**Ext3**和**Ext4**，确保**扩展属性**和**安全标签**被启用。然后，通过选择**退出**回到主菜单：

![翻转开关](img/0594OS_04_07.jpg)

从那个屏幕退出到主菜单，然后转到**安全选项**。一旦进入**安全选项**子菜单，启用**启用不同的安全模型**和**套接字和网络安全钩子**选项：

![翻转开关](img/0594OS_04_08.jpg)

一旦这些被启用，将出现更多选项。启用**NSA SELinux 支持**并确保以下截图中的其他选择和值被复制：

![翻转开关](img/0594OS_04_09.jpg)

最后，将**默认安全模块**设置为 SELinux：

![翻转开关](img/0594OS_04_10.jpg)

一旦选择**默认安全模块**，将出现一个新窗口，你可以从中选择**SELinux**。通过选择**退出**退出配置菜单，直到被要求保存你的新配置：

![翻转开关](img/0594OS_04_11.jpg)

保存新配置并将这些更改写入原始内核配置文件。否则，它将在后续构建中被覆盖。为此，我们需要发现默认构建中使用了哪个配置文件，这是我们之前在用`make menuconfig`创建自己的配置之前构建的：

```kt
$ grep defconfig logz make -C kernel_imx imx6_udoo_android_defconfig ARCH=arm CROSS_COMPILE=`pwd`/prebuilts/gcc/linux-x86/arm/arm-eabi-4.6/bin/arm-eabi-

```

你可以看到使用了`imx6_udoo_android_defconfig`作为默认配置。复制你的自定义配置并重新构建：

```kt
$ cp .config arch/arm/configs/imx6_udoo_android_defconfig
$ croot
$ make –j4 bootimage 2>&1 | tee logz

```

快速检查日志文件始终是一个好主意，以验证 SELinux 实际上已构建到内核中：

```kt
$ grep -i selinux logz 
HOSTCC scripts/selinux/mdp/mdp
HOSTCC scripts/selinux/genheaders/genheaders
GEN security/selinux/flask.h security/selinux/av_permissions.h
CC security/selinux/avc.o
...

```

现在，使用支持 SELinux 的构建内核，将 SD 卡插入主机并运行以下命令：

```kt
$ sudo -E ./make_sd.sh /dev/sdd
$ sudo sync

```

### 小贴士

不要忘记像之前一样卸载 SD 卡上的任何自动挂载分区。

将 SD 卡插入 UDOO，并启动它。你应该会在串行控制台看到我们之前看到的日志：

![翻转开关](img/0594OS_04_12.jpg)

最终，串行连接应该带我们到一个 root shell。

# 它已经启动了

我们如何知道我们在内核中成功启用了 SELinux？在本章的早期，你运行了命令`adb shell cat /proc/filesystems`。我们将做同样的事情，寻找一个新的文件系统，称为`selinuxfs`。如果它存在，这表明我们已成功启用了 SELinux。在串行终端中运行以下命令：

```kt
# cat /proc/filesystems | grep selinux 
nodev selinuxfs

```

我们可以看到`selinuxfs`已经存在！另一个常见的做法是检查`dmesg`以查找任何 SELinux 输出。为此，通过串行终端执行以下命令：

```kt
# dmesg | grep -i selinux
<6>SELinux: Initializing.
<7>SELinux: Starting in permissive mode
<7>SELinux: Registering netfilter hooks
<3>SELinux: policydb version 26 does not match my version range 15-23
<4>SELinux: Could not load policy: Invalid argument

```

# 摘要

这是一个非常激动人心的章节。你学习了如何在内核配置中启用 SELinux，启动“安全”的系统，以及如何验证其存在。我们还学习了如何为 UDOO 刷写和构建镜像，以及如何通过串行和`adb`连接连接到它。在下一章中，我们将关注如何使 UDOO 具有 SE for Android 功能。
