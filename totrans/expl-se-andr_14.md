# 附录 A. 开发环境

为了构建 UDOO 提供的 Android 4.3 源代码，您需要一个带有 Oracle Java 6 的 Ubuntu Linux 系统。虽然可能可以使用此设置的变体，但 Google 的 Android 4.3 标准目标开发平台是 Ubuntu 12.04。因此，我们将使用此设置以确保在探索 Linux、SE Linux、Android、UDOO 和 SE for Android 时有最高的成功概率。

在本附录中，我们将执行以下操作：

+   使用 **虚拟机**（**VM**）下载并安装 Ubuntu 12.04。

+   通过安装 VirtualBox 扩展包和 VirtualBox 客户端附加组件来增强我们的虚拟机的性能。

+   设置一个适合构建 Linux 内核和 UDOO 源代码的开发环境。

+   安装 Oracle Java 6

### 小贴士

如果您已经使用 Ubuntu Linux 12.04，您可以跳转到 *构建环境* 部分。如果您打算在本地安装 Ubuntu（不在虚拟机中），则应跳转到 *Ubuntu Linux 12.04* 部分，并遵循那些说明，忽略 VirtualBox 步骤。

# VirtualBox

有许多虚拟化产品可用于运行客户操作系统，例如 Ubuntu Linux，但在此设置中我们将使用 **VirtualBox**。VirtualBox 是一个广泛使用的开源虚拟化系统，适用于 Mac、Linux、Solaris 和 Windows 宿主（以及其他）。它支持多种客户操作系统。VirtualBox 还允许使用许多现代/常见处理器家族的硬件虚拟化，通过为每个虚拟机提供其自己的私有地址空间来提高性能。

VirtualBox 文档为各种平台提供了优秀的安装说明，我们建议您参考这些说明以适应您的宿主平台。您可以在[`www.virtualbox.org/manual/ch02.html`](http://www.virtualbox.org/manual/ch02.html)找到有关在宿主操作系统上安装和运行 VirtualBox 的信息。

# Ubuntu Linux 12.04（精确的穿山甲）

要安装 Ubuntu Linux 12.04，您首先需要下载适当的发行版镜像。这些可以在[`releases.ubuntu.com/12.04/`](http://releases.ubuntu.com/12.04/)找到。虽然有许多可接受的镜像，但我们将安装发行版的 64 位桌面版本——[`releases.ubuntu.com/12.04/ubuntu-12.04.5-desktop-amd64.iso`](http://releases.ubuntu.com/12.04/ubuntu-12.04.5-desktop-amd64.iso)。在此示例中，我们使用的宿主机器是运行 OS X 10.9.2 的 64 位 Macbook Pro，因此我们也针对 64 位客户机。如果您有 32 位机器，我们涵盖的基本机制将是相同的；只有一些细节会有所不同，因此我们将留待您去发现和解决。

在您的宿主上启动 VirtualBox，等待 **虚拟机管理器** 窗口出现，然后执行以下步骤：

1.  点击 **新建**。

1.  对于 **名称** 和 **操作系统** 设置，请进行以下选择：

    +   **名称**: **SE for Android 书籍**

    +   **类型**: **Linux**

    +   **版本**：**Ubuntu (64 位)**

1.  将**内存大小**设置为至少`16` GB 的值。低于此值的配置将导致构建失败。

1.  要设置硬盘，选择**现在创建虚拟硬盘**。将此值设置为至少`80` GB。

1.  选择**硬盘文件类型**，**VDI (VirtualBox 磁盘镜像)**。

1.  确保物理硬盘上的存储设置为**动态分配**。

1.  当提示输入**文件位置和大小**时，将新的虚拟硬盘命名为**SE for Android Book**，并设置其大小为 80 GB。

确保在左侧面板中选择**SE for Android Book**虚拟机。点击绿色的启动箭头以执行虚拟机的初始启动。将出现一个对话框，要求您选择虚拟光驱文件。点击小文件夹图标，找到您之前下载的`ubuntu-12.04.5-desktop-amd64.iso` CD 镜像文件。然后点击**启动**。

当屏幕变黑并在虚拟机窗口的底部中央显示键盘图像时，按任意键开始 Ubuntu 安装。您这样做后，将出现语言选择屏幕。选择最适合您的语言，但在这个例子中，我们将选择**英语**。然后选择**安装 Ubuntu**。

有时，您可能会在虚拟机窗口中看到一些看起来不寻常的错误信息——例如**SMBus 基本地址未初始化**。此消息显示是因为 VirtualBox 不支持 Ubuntu 12.04 默认加载的特定内核模块。然而，这不会造成任何困难，只是一个小小的外观烦恼。几分钟后，一个漂亮的 GUI 安装屏幕将出现，等待您再次选择语言。我们将再次选择**英语**。

在接下来的**准备安装 Ubuntu**屏幕上，显示了三项清单项目。您应该已经满足了第一项，因为您的虚拟驱动器比 Ubuntu 的最小要求大得多。为了满足其他要求，请确保您的宿主系统已连接电源并有稳定的网络连接。尽管这对我们的目的来说完全是多余的，但我们几乎总是在继续之前标记**安装时下载更新**和**安装此第三方软件**复选框。

在**安装类型**屏幕上，我们将选择简单路径并选择**擦除磁盘并安装 Ubuntu**。请记住，这将仅擦除您虚拟机的虚拟硬盘，而不会影响您的宿主系统。在**擦除磁盘并安装 Ubuntu**屏幕上，您的虚拟硬盘应该已经选中，因此您只需点击**现在安装**。

从现在起，在 Ubuntu 安装过程中，将同时发生两个单独的任务：在后台线程中，安装程序将为基本系统的安装准备虚拟驱动器；其次，您将配置您新系统的一些基本方面。但在继续之前，您必须通过点击世界地图上的适当位置来识别您的时区。然后识别您的键盘布局并继续。

设置您的第一个用户帐户。在这种情况下，它将是我们在本书中工作的帐户，因此我们将输入以下信息：

+   **您的姓名**：**Book User**

+   **您的计算机名称**：**SE-for-Android**

+   **选择用户名**：**bookuser**

+   **密码字段**：（您喜欢的任何内容）

我们还将选择**自动登录**。虽然出于安全原因我们通常不会这样做，但为了方便起见，我们将在本地虚拟机中这样做；但您可以选择您喜欢的任何方式来保护此帐户。

一旦 Ubuntu 安装完成，将出现一个对话框，提示您重新启动计算机。点击**现在重新启动**按钮，几分钟后，终端提示将通知您移除所有安装介质并按*Enter*。要移除虚拟安装 CD，请使用 VirtualBox 菜单栏转到**设备** | **CD/DVD 设备** | **从虚拟驱动器中移除磁盘**。然后按*Enter*重新启动虚拟机，但通过关闭虚拟机窗口来中断启动过程。它将询问您是否要关闭机器的电源。只需点击**确定**。

# VirtualBox 扩展包和客户机增强功能

为了从您的客户机 Ubuntu VM 中获得最佳性能并访问与 UDOO 一起工作所需的虚拟 USB 设备，您需要安装 VirtualBox 扩展包和客户机增强功能。

## VirtualBox 扩展包

从 VirtualBox 网站下载扩展包，网址为[`www.virtualbox.org/wiki/Downloads`](http://www.virtualbox.org/wiki/Downloads)。那里将有一个针对**所有支持的平台**的下载链接。一旦下载了此文件，您就需要安装它。对于每种类型的宿主系统，这个过程都不同，但它非常简单。对于 Linux 和 Mac OS X 宿主系统，只需双击下载的扩展包文件即可。对于 Windows 系统，您需要运行您下载的安装程序。

## VirtualBox 客户机增强功能

一旦您完成了扩展包的安装，请通过在 VirtualBox 左侧面板中选择虚拟机并点击工具栏中的**启动**来从 VirtualBox 启动 Ubuntu Linux 12.04 VM。一旦您的 Ubuntu 桌面变得活跃，您会注意到它无法适应您的虚拟机窗口。调整虚拟机窗口的大小以使其更大，虚拟机屏幕将保持相同的大小。这以及其他性能问题将通过安装 VirtualBox 客户机增强功能来解决。您也可能在虚拟桌面上看到一个窗口打开，指示有新的 Ubuntu 版本可用。请不要升级；只需关闭该窗口。

使用 VirtualBox 菜单栏，转到 **设备** | **插入虚拟光盘镜像…**。不久之后，将出现一个对话框，询问您是否要在新插入的媒体上运行软件。点击 **运行** 按钮。然后您需要通过输入您的用户密码（您在设置时输入的）来验证您的用户身份。一旦用户验证成功，脚本将自动构建和更新几个内核模块。一旦脚本完成，通过点击屏幕右上角的齿轮图标，选择 **关机…**，然后在随后出现的对话框中点击 **重启** 来重启虚拟机。

当虚拟机重启时，您应该注意到的第一件事是虚拟机屏幕现在适合虚拟机窗口。此外，如果您调整虚拟机窗口的大小，虚拟机屏幕也会相应调整。这是确定您已成功安装 VirtualBox 客户端扩展的最简单方法。

# 使用共享文件夹节省时间

在为 UDOO 开发图像时，为了提高您的整体性能，您还可以在主机系统和您的 Ubuntu Linux 客户端系统之间设置共享文件夹。这样，一旦您为 UDOO 构建了新的 SD 卡镜像，您就可以通过共享文件夹直接将镜像提供给主机。然后，主机可以执行长时间运行的命令来刷写 SD 卡，而不会通过虚拟化层减慢对主机卡读者的访问速度，从而不会增加过程的时间。在我们编写这本书所使用的系统中，每次刷写镜像可以节省大约 10 分钟。

要设置共享文件夹，您必须先打开 VirtualBox 管理器，并关闭您的 Ubuntu 虚拟机。点击 **设置** 工具栏图标。然后选择打开的 **设置** 对话框中的 **共享文件夹** 选项卡。点击右侧的 **添加共享文件夹** 图标。输入您想要共享的主机上的 **文件夹路径**。在我们的例子中，我们创建了一个名为 `vbox_share` 的新文件夹来与我们的虚拟机客户机共享。VirtualBox 将生成 **文件夹名称**，但在点击 **确定** 之前，请确保您选择了 **自动挂载**。从现在起，当您启动 Ubuntu 虚拟机时，共享文件夹将在您的客户机虚拟机中作为 `/media/sf_<folder_name>` 可访问。然而，如果您尝试从客户机列出该目录中的文件，您可能会被拒绝。为了使我们的 `bookuser` 对此文件夹（即读写访问）具有完全访问权限，我们需要将该 UID 添加到 `vboxsf` 组中：

```kt
$ sudo usermod -a -G vboxsf bookuser

```

再次从您的客户机注销并登录，或者重启客户机虚拟机以完成此过程。

# 构建环境

为了准备我们的系统构建 Linux 内核、Android 和 Android 应用程序，我们需要安装和设置一些关键的软件组件。点击屏幕左侧启动栏顶部的 Ubuntu 桌面图标。在出现的搜索栏中输入 `term` 并按 *Enter* 键。将打开一个终端窗口。然后执行以下命令：

```kt
$ sudo apt-get update
$ sudo apt-get install apt-file git-core gnupg flex bison gperf build-essential zip curl zlib1g-dev libc6-dev lib32ncurses5-dev ia32-libs x11proto-core-dev libx11-dev ia32-libs dialog liblzo2-dev libxml2-utils minicom

```

当被询问是否想要继续时，请输入`y`并按*Enter*键。

# Oracle Java 6

从 Oracle Java 存档网站下载最新的 Java 6 SE 开发工具包（版本 6u45），网址为[`www.oracle.com/technetwork/java/javase/archive-139210.html`](http://www.oracle.com/technetwork/java/javase/archive-139210.html)。您需要`jdk-6u45-linux-x64.bin`版本以满足 Google 的目标开发环境。下载完成后，执行以下命令来安装 Java 6 JDK：

```kt
$ chmod a+x jdk-6u45-linux-x64.bin
$ sudo mkdir -p /usr/lib/jvm
$ sudo mv jdk-6u45-linux-x64.bin /usr/lib/jvm/
$ cd /usr/lib/jvm/
$ sudo ./jdk-6u45-linux-x64.bin
$ sudo update-alternatives --install "/usr/bin/java" "java" "/usr/lib/jvm/jdk1.6.0_45/bin/java" 1
$ sudo update-alternatives --install "/usr/bin/jar" "jar" "/usr/lib/jvm/jdk1.6.0_45/bin/jar" 1
$ sudo update-alternatives --install "/usr/bin/javac" "javac" "/usr/lib/jvm/jdk1.6.0_45/bin/javac" 1
$ sudo update-alternatives --install "/usr/bin/javaws" "javaws" "/usr/lib/jvm/jdk1.6.0_45/bin/javaws" 1
$ sudo update-alternatives --install "/usr/bin/jar" "jar" "/usr/lib/jvm/jdk1.6.0_35/bin/jar" 1
$ sudo update-alternatives --install "/usr/bin/javadoc" "javadoc" "/usr/lib/jvm/jdk1.6.0_45/bin/javadoc" 1
$ sudo update-alternatives --install "/usr/bin/jarsigner" "jarsigner" "/usr/lib/jvm/jdk1.6.0_45/bin/jarsigner" 1
$ sudo update-alternatives --install "/usr/bin/javah" "javah" "/usr/lib/jvm/jdk1.6.0_45/bin/javah" 1
$ sudo rm jdk-6u45-linux-x64.bin

```

# 摘要

在本附录中，我们讨论了 Google 为 Android 设置的目标开发环境，并展示了如何创建一个兼容的环境，这可能在虚拟机中完成。您应该自由地修改系统中的其他元素，但安装本附录中的元素将为您提供执行第四章中概述的所有步骤以及更多步骤所需的最小可行环境。
