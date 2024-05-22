# 第二章. 与 Android 交互

在上一章中，你在 BBB 上安装了 Android 系统，并收集了所有需要的硬件和组件，以便尝试本书中的练习。现在你有了可用的 Android 系统和探索它所需的硬件，是时候深入了解 Android，并找出如何为其准备与自定义硬件的接口了。

大多数人可能不会认为 Android 和 Linux 非常相似，但两者之间的共同点比你想象的要多。在精美的用户界面和各种应用之下，Android 实际上是 Linux 系统。Android 的文件系统布局和服务与典型的 Linux 系统大不相同，因此在用户空间（应用和其他进程执行的地方）肯定存在许多差异。在内核空间（设备驱动程序执行的地方，并为每个运行中的进程分配资源），它们在功能上几乎完全相同。理解 BBB 如何与 Linux 内核驱动程序交互是创建能够执行相同操作的 Android 应用程序的关键。

在本章中，我们将向您介绍 Android 的硬件抽象层（HAL）。我们还将向您介绍 PacktHAL，这是一个特殊的库，您可以在应用中包含它，以便与 BBB 上的硬件进行接口交互。我们假设您已经在系统上安装并运行了 Eclipse **Android 开发工具**（**ADT**）、Android **原生开发工具包**（**NDK**）和 **Android 调试桥**（**ADB**）工具。

在本章中，我们将涵盖以下主题：

+   了解 Android HAL

+   安装 PacktHAL

+   为 PacktHAL 设置 Android NDK

+   多路复用 BBB 引脚

### 提示

**您是否缺少一些工具？**

如果您还没有在系统上安装 Eclipse ADT 或 Android NDK 工具，您可以在以下位置找到安装说明和下载链接：

+   **Eclipse ADT**：[`developer.android.com/sdk`](http://developer.android.com/sdk)

+   **Android NDK**：[`developer.android.com/tools/sdk/ndk`](http://developer.android.com/tools/sdk/ndk)

如何安装 ADB 在本章后面会讨论。本章假设您已经将 Eclipse ADT 安装到 `c:\adt-bundle` 目录（如果您使用的是 Windows 系统，我们不假设 Linux 的情况），并且您已经将 Android NDK 安装到 `c:\android-ndk` 目录（Windows）或主目录下的 `android-ndk`（Linux）。如果您将这些工具安装到了其他位置，那么您需要对本章后面的一些指令进行一些简单的调整。

# 了解 Android HAL

安卓内核包含一些在典型 Linux 内核中找不到的额外功能，如**Binder IPC**和低内存杀手，但除此之外，它仍然是 Linux。这为您在与安卓硬件接口时提供了一个很大的优势，即如果安卓系统使用的内核中已经存在 Linux 驱动，那么您已经拥有该设备的安卓驱动。

安卓应用必须通过生成视频和音频数据、接收按钮和触摸屏输入事件以及从摄像头、加速度计等收集外部世界信息的设备接收传感器事件，与安卓设备的硬件进行交互。利用这些设备的现有 Linux 驱动，使得安卓支持变得更加容易。与传统的 Linux 发行版不同，后者允许应用程序直接访问许多不同的设备文件（直接在`/dev`文件系统中打开文件），安卓极大地限制了进程直接访问硬件的能力。

考虑到有多少不同的安卓应用使用设备的音频功能来播放声音或录制音频数据。在安卓之下，Linux 内核通过**高级 Linux 声音架构**（**ALSA**）音频驱动提供这种音频功能。在大多数情况下，一次只能有一个进程打开和控制 ALSA 驱动资源。如果各个应用负责获取、使用和释放 ALSA 驱动，那么在所有各种应用之间协调音频资源的使用将变得非常混乱。一个行为不当的应用很容易控制音频资源，并阻止所有其他应用使用它们！但是，这些资源的分配和控制如何处理呢？为了解决这个问题，安卓使用了*管理者*。

## 安卓管理者

管理者是系统中的组件，代表所有应用控制硬件设备。每个应用都需要一组资源（如音频、GPS 和网络访问）来完成其工作。管理者负责分配和接口每个资源，并确定应用是否有权限使用该资源。

让管理者处理这些低级细节可以使生活变得轻松许多。安卓可以安装在各种硬件平台上，这些平台在物理尺寸和输入/输出能力上有很大差异，不能期望应用开发者对其应用可能安装的每个平台都有深入了解。

要使用资源，应用必须通过`android.content.Context`类的`getSystemService()`方法创建对适当管理者的引用：

```java
// Create a reference to the system "location" manager
LocationManager locationManager = (LocationManager)
  mContext.getSystemService(LOCATION_SERVICE);
```

然后，通过这个管理者引用来发起信息和控制请求：

```java
// Query the location manager to determine if GPS is enabled
isGPSEnabled = locationManager.
isProviderEnabled(LocationManager.GPS_PROVIDER);
```

应用通过 Java Android API 与管理者交互。虽然管理者响应这些 Java 方法，但它们最终必须使用**Java 本地接口**（**JNI**）调用直接与硬件交互的本机代码。这才是真正控制硬件的地方。Android API 与控制硬件的本机代码调用之间的桥梁被称为**硬件抽象层**（**HAL**）。

HAL 的各个部分通常用 C/C++编写，每个设备的供应商负责实现它们。如果 HAL 的某些部分缺失，服务和应用将无法充分利用硬件平台的所有方面。各种 Android 服务使用 HAL 与硬件通信，应用通过 IPC 与这些服务通信，从而访问硬件。服务代表应用与硬件交互（假设应用具有访问该特定硬件资源的适当 Android 权限）。

## HAL 开发工作流程

通常，创建一个完整的 HAL 需要遵循以下步骤：

1.  识别或开发一个 Linux 内核设备驱动程序以控制硬件。

1.  创建一个内核设备树覆盖层，以实例化和配置驱动程序。

1.  开发一个用户空间库以与内核设备驱动程序接口。

1.  为用户空间库开发 JNI 绑定。

1.  使用 JNI 绑定开发一个 Android 管理者以与硬件接口。

有时，很难决定特定的自定义硬件应该正确地集成到 HAL 的哪个位置，以及哪个管理者应该负责访问硬件。Android 的哪些权限控制对硬件的访问？API 是否需要扩展以提供新型权限？是否需要创建自定义服务？

对于爱好者、学生和其他对硬件接口简单实验感兴趣的开发商来说，为一块自定义硬件实现一个适当 HAL 的每个方面都有点过于复杂。虽然商业 Android 系统必须完成所有这些步骤以开发适当的 HAL，但本书采取了一种更为直接的硬件访问方法。

由于我们的重点是展示如何将 Android 应用与硬件接口，我们通过提供**PacktHAL**这一本地库来跳过步骤 1 至 4。PacktHAL 是一个实现了非常简单的 HAL 的本地库，它将帮助你轻松地开始 BBB 上与硬件接口的艰巨任务，并提供了一组能够与本书示例中使用的硬件接口的函数。严格来说，你的应用将作为每个硬件资源的管理者。

## 使用 PacktHAL 工作

应用程序通过 JNI 与 PacktHAL 的本地调用进行通信。PacktHAL 展示了如何通过三种不同的接口方法：`GPIO`、`SPI` 和 `I2C`，与硬件进行用户空间交互。使用 PacktHAL，你可以直接访问硬件设备。第三章至第六章提供了这种接口如何工作以及如何在你的 Android 应用代码中使用它的示例。每一章将检查该章节应用示例中使用的 PacktHAL 的各个部分。

### 提示

**PacktHAL 实际上是如何与硬件通信的？**

通常，任何允许你在 Linux 下与硬件接口的方法也可以被 HAL 用于接口。读取、写入以及对 `/dev` 文件系统中的文件进行 `ioctl()` 调用是有效的，使用 `mmap()` 提供对内存映射控制寄存器的访问也同样有效。PacktHAL 使用这些技术与你连接到 BBB 的硬件进行接口。

使用 PacktHAL 远没有正确的 HAL 实现安全，因为我们必须改变硬件用户空间接口的权限，使得*任何*应用都能直接访问硬件。这可能会使你的系统容易受到恶意应用的攻击，因此这种做法绝不能在生产设备中使用。用户通常会对商业 Android 手机和平板进行 root（获取超级用户权限），以减少这些设备默认的严格权限。这使得他们可以安装和启用自定义功能，并为他们的设备提供更多的灵活性和定制。

由于你将 BBB 作为 Android 原型设备使用，这种做法是你与硬件交互的最简单方式。这是朝着开发自己的自定义管理器和服务迈出的一步，这些管理器和服务代表应用与你的硬件通信。理想情况下，在商业设备上，只有 Android 管理器才有必要的权限直接与硬件接口。

### 提示

一旦你习惯在应用中使用 PacktHAL，你可以检查 PacktHAL 的源代码，以更好地理解本地代码如何与 Linux 内核接口。最终，你可能会发现自己将 PacktHAL 集成到自己的自定义管理器中。你甚至可能会发现自己为实际的内核开发自定义代码！

# 安装 PacktHAL

PacktHAL 的所有组成部分都位于 `PacktHAL.tgz` 文件中，该文件可在 Packt 的网站([`www.packtpub.com/support`](http://www.packtpub.com/support))下载。这是一个压缩的 tar 文件，包含了修改 BBBAndroid 以使用 PacktHAL 并在应用中包含 PacktHAL 支持所需的所有源代码和配置文件。

## 在 Linux 下准备 PacktHAL

下载`PacktHAL.tgz`文件后，你必须解压并展开它。我们将假设你在下载后已将`PacktHAL.tgz`复制到你的主目录并从那里解压。我们将你的主目录称为`$HOME`。

使用 Linux 的`tar`命令来解压并展开文件：

```java
$ cd $HOME
$ tar –xvf PacktHAL.tgz

```

在你的`$HOME`目录中现在存在一个名为`PacktHAL`的目录。所有 PacktHAL 文件都位于此目录中。

## 在 Windows 下准备 PacktHAL

下载`PacktHAL.tgz`文件后，解压并展开它。我们将假设你在下载后已将`PacktHAL.tgz`复制到`C:`驱动器的根目录，并使用 WinRAR 从那里解压。

### 提示

**我应该在哪里解压 PacktHAL.tgz？**

你可以在桌面或其他任何地方解压和展开`PacktHAL.tgz`文件，但稍后你将需要执行一些命令行命令来复制文件。如果`PacktHAL.tgz`在`C:`驱动器的根目录下解压和展开，这些操作会简单得多，因此我们将假设你从那里执行这些操作。

执行以下步骤以提取`PacktHAL.tgz`文件：

1.  打开文件资源管理器窗口，导航至`C:`驱动器的根目录。

1.  在文件资源管理器中右键点击`PacktHAL.tgz`文件并选择**在此处解压**。

现在存在一个名为`C:\PacktHAL`的目录。所有 PacktHAL 文件都位于此目录中。

## PacktHAL 目录结构

`PacktHAL`目录具有以下结构：

```java
PacktHAL/
  |
  +----cape/
  |      |
  |      +----BB-PACKTPUB-00A0.dts
  |      +----build_cape.sh
  |
  +----jni/
  |      |
  |      +----(Various .c and .h files)
  |      +----(Various .mk files)
  |
  +----prebuilt/
  |      |
  |      +----BB-PACTPUB-00A0.dtbo
  |      +----init.genericam33xx(flatteneddevicetr.rc
  |      +----spi
  |             |
  |             +----spidev.h
  |
  +----README.txt
```

`cape`子目录包含了构建 Device Tree 覆盖所需源代码和构建脚本，以启用 PacktHAL 所需的所有硬件功能。你将在本章后面了解更多关于 Device Tree 覆盖的内容。`jni`子目录包含了实现 PacktHAL 的源代码文件。这些源文件将在后面的章节中添加到你的项目中，以便在应用中构建对 PacktHAL 的支持。`prebuilt`目录包含一些预制的文件，这些文件必须添加到你的 BBBAndroid 映像和 Android NDK 中，以构建和使用 PacktHAL。你将在接下来的几节中将`prebuilt`目录中的文件安装到它们所需的位置。

## 为 PacktHAL 准备 Android

在任何应用中使用 PacktHAL 之前，你必须准备你的 BBBAndroid 安装环境。默认情况下，Android 对硬件设备的权限分配非常严格。要使用 PacktHAL，你必须减少权限限制并为将要接口的硬件配置 Android。这些操作需要将一些预构建的文件复制到你的 Android 系统中，进行一些配置更改，以放宽各种 Android 权限并正确为 PacktHAL 配置硬件。

你将使用 ADB 工具将必要的文件推送到正在运行的 BBB 系统。在推送文件之前，启动 BBB 上的 Android 并使用随 BBB 一起提供的 USB 电缆将 BBB 连接到你的电脑。一旦你达到这个阶段，继续按照说明操作。

## 在 Linux 下推送 PacktHAL 文件

以下步骤是在 Linux 下发布 PacktHAL 文件的方法：

1.  在开始之前，请确保使用 `adb devices` 命令确认 ADB 能够看到你的 BBB。BBB 将报告有一个序列号为 `BBBAndroid`。执行以下命令：

    ```java
    $ adb devices
    List of devices attached
    BBBAndroid      device

    ```

1.  如果你缺少 `adb` 命令，可以通过 `apt-get` 安装 `android-tools-adb` 包：

    ```java
    $ sudo apt-get install android-tools-adb

    ```

    ### 提示

    **为什么 Linux 找不到我的 BBB？**

    如果你的系统上安装了 `adb` 但无法看到 BBB，你可能需要向系统添加一个 `udev` 规则并进行一些额外的故障排除。如果你遇到任何困难，Google 提供了添加此规则和故障排除步骤的指导，可以在 [`developer.android.com/tools/device.html`](http://developer.android.com/tools/device.html) 找到。

    BBBAndroid 报告其 ADB 接口的 USB 设备 ID 为 `18D1:4E23`，这是 Google Nexus S 的设备 ID，所以 BBB 的 USB 供应商 ID 是 18D1（Google 设备的设备 ID）。

1.  当你确认 `adb` 能够看到 BBB 后，切换到 `PacktHAL` 目录，通过 `adb` 进入 Android 的 shell，并将只读的 `rootfs` 文件系统重新挂载为可读写：

    ```java
    $ cd $HOME/PacktHAL/prebuilt
    $ adb shell
    root@beagleboneblack:/ # mount rootfs rootfs / rw
    root@beagleboneblack:/ # exit

    ```

1.  现在，将必要的文件推送到 Android 的 `rootfs` 文件系统：

    ```java
    $ adb push BB-PACKTPUB-00A0.dtbo /system/vendor/firmware
    $ adb push init.genericam33xx\(flatteneddevicetr.rc /
    $ adb chmod 750 /init.genericam33xx\(flatteneddevicetr.rc

    ```

1.  最后，进入 Android 的 `rootfs` 文件系统以同步它，并将其重新挂载为只读：

    ```java
    $ adb shell
    root@beagleboneblack:/ # sync
    root@beagleboneblack:/ # mount rootfs rootfs / ro remount
    root@beagleboneblack:/ # exit

    ```

1.  现在，你已经在 Linux 下为 PacktHAL 准备好了 BBBAndroid 镜像。从你的 BBB 上拔掉电源线和 USB 电缆以关闭它。

1.  然后，启动 BBB 以验证你刚才所做的修改后 Android 是否能正常启动。

## 在 Windows 下推送 PacktHAL 文件

你需要找到你的 `adb.exe` 文件的位置。它是 Android SDK 中平台工具的一部分。在以下说明中，我们假设你将 Eclipse ADT 安装在 `c:\adt-bundle` 目录下，那么 `adb` 的完整路径就是 `c:\adt-bundle\sdk\platform-tools\adb.exe`。

以下步骤是在 Windows 下发布 PacktHAL 文件的方法：

1.  在开始之前，请确保使用 `adb devices` 命令确认 `adb` 能够看到你的 BBB。BBB 将报告有一个序列号为 `BBBAndroid`：

    ```java
    $ adb devices
    List of devices attached
    BBBAndroid      device

    ```

    ### 提示

    **为什么 Windows 找不到我的 BBB？**

    在 Windows 下让`adb`识别 Android 设备可能会非常困难。这是因为每个创建 Android 设备的硬件制造商都为其提供了自己的 Windows ADB 设备驱动程序，Windows 使用该驱动程序与设备通信。BBBAndroid 报告其 ADB 接口的 USB 设备 ID 为`18D1:4E23`，这是 Google Nexus S 的设备 ID。这是 Koushik Dutta 为 Windows 提供的优秀通用 ADB 驱动程序支持的众多 USB 设备之一。如果`adb`找不到您的 BBB，请安装通用 ADB 驱动程序，然后重试。您可以从[`www.koushikdutta.com/post/universal-adb-driver`](http://www.koushikdutta.com/post/universal-adb-driver)下载驱动程序。

1.  验证这一点后，`adb`可以看到 BBB，通过`adb`进入 Android 的 shell，并将只读的`rootfs`文件系统重新挂载为读写：

    ```java
    $ adb shell
    root@beagleboneblack:/ # mount rootfs rootfs / rw
    root@beagleboneblack:/ # exit

    ```

1.  现在，将必要的文件推送到 Android 的`rootfs`文件系统：

    ```java
    $ adb push c:\PacktHAL\prebuilt\BB-PACKTPUB-00A0.dtbo /system/vendor/firmware
    $ adb push c:\PacktHAL\prebuilt\init.genericam33xx(flatteneddevicetr.rc /
    $ adb chmod 750 /init.genericam33xx\flatteneddevicetr.rc

    ```

1.  最后，通过 shell 进入 Android 的`rootfs`文件系统，将其同步并重新挂载为只读：

    ```java
    $ adb shell
    root@beagleboneblack:/ # sync
    root@beagleboneblack:/ # mount rootfs rootfs / ro remount
    root@beagleboneblack:/ # exit

    ```

1.  您现在已经在 Windows 下为 PacktHAL 准备好了 BBBAndroid 映像。请将电源线和 USB 线从 BBB 上拔下以关闭它。然后，给 BBB 供电，以验证您刚才所做的修改后 Android 是否能正常启动。

    ### 提示

    **为什么 init.genericam33xx（flatteneddevicetr.rc 文件命名如此奇怪？**

    Android 设备有一组只读属性，它们向应用程序和管理器描述系统的硬件和软件。其中之一是`ro.hardware`，它描述了内核配置的硬件。Android 中的设备特定`.rc`文件具有`init.{ro.hardware}.*rc`的形式。

    在 Linux 内核源代码中，`arch/arm/mach-omap2/board-generic.c`文件使用`DT_MACHINE_START()`宏来指定 BBB 平台的名称为`Generic AM33XX (Flattened Device Tree)`。这个文本字符串被转换为小写，删除空格，并截断以生成存储在`ro.hardware`属性中的最终字符串。

# 为 PacktHAL 设置 Android NDK

不幸的是，Android 的**原生开发工具包**（**NDK**）缺少一个构建 PacktHAL 所需的内核头文件。这个缺失的头文件描述了用户空间应用程序与通用 SPI 驱动程序（`spidev`，您将在第五章，*使用 SPI 与高速传感器接口*中使用）之间的接口。这个头文件缺失并不是 NDK 的错，因为通常应用程序不需要直接访问`spidev`驱动程序。

由于您是使用应用程序直接与硬件通信，因此需要将这个缺失的头文件复制到您的 NDK 安装中。

### 提示

为了方便起见，我们在 PacktHAL 源代码压缩包中包含了这个头文件的副本。在构建 PacktHAL 之前，您只需要将文件复制到您的 NDK 安装中。

BBBAndroid 是 4.4.4 KitKat 版本，API 级别 19 是此版本支持的最高级别。你将为本书的示例构建 API 级别 19 的所有内容。每个 API 级别在 NDK 中都有不同的头文件集，因此你必须向 API 级别 19 的`include/linux`目录添加缺失的头文件。如果你决定在较低的 API 级别构建应用，可以重复以下步骤，将附加头文件添加到你想使用的任何其他 API 级别中。

## 在 Linux 下向 NDK 添加头文件

如果你打算在 Linux 下使用 Eclipse ADT 构建应用，你需要在你的 Linux 系统上安装 Android NDK。对于这些说明，我们将假设你已经将 NDK 安装到`$HOME`目录下的`android-ndk`文件夹中。由于在本章前面你已经下载、解压并解包了`PacktHAL.tgz`文件到你的`$HOME`目录，我们将假设你创建的`PacktHAL`目录还在那里：

```java
$ cd $HOME/android-ndk/platforms/android-19/arch-arm/usr/include/linux
$ cp -rf $HOME/PacktHAL/prebuilt/spi

```

这将把`spi`头文件目录的内容复制到你的 NDK 头文件中。现在你的 Linux NDK 安装中有了构建 PacktHAL 所需的额外头文件。

## 在 Windows 下向 NDK 添加头文件

如果你打算在 Windows 下使用 Eclipse ADT 构建应用，你需要在你的 Windows 系统上安装 Android NDK。对于这些说明，我们将假设你已经将 NDK 安装到`c:\android-ndk`文件夹中。由于在本章前面你已经下载、解压并解包了`PacktHAL.tgz`文件到你的`c:\`目录，我们将假设你创建的`PacktHAL`目录还在那里：

1.  打开文件资源管理器窗口，导航至`c:\android-ndk\platforms\android-19\arch-arm\usr\include\linux`路径。

1.  打开第二个文件资源管理器窗口，导航至`c:\PacktHAL\prebuilt`路径。右键点击`spi`目录，并在上下文菜单中选择**复制**。

1.  切换到 Android NDK 窗口，在窗口中的文件列表空白处右键点击，然后在上下文菜单中选择**粘贴**。

这将把`spi`头文件目录的内容复制到你的 NDK 头文件中。现在你的 Windows NDK 安装中有了构建 PacktHAL 所需的额外头文件。

# 对 BBB 引脚进行复用

由于在 Android 下访问硬件资源与在 Linux 下遵循相同的流程，因此了解 Linux 内核如何配置设备驱动程序并将它们分配给特定的硬件非常重要。也有必要了解这些内核驱动程序如何为 PacktHAL 提供可以与之交互的用户空间接口。

BBB 的 AM3359 处理器在其数百个引脚上提供了各种各样的信号。这些信号包括许多不同的、专门的接口总线和传感器输入。潜在的信号数量远远超过了可用于将这些信号输出到外界的引脚数量。为了选择哪些信号在引脚上可用，这些引脚被复用，或称为*muxed*，到特定的信号。

处理器的几个引脚被连接到 BBB 的 P8 和 P9 头的连接上。这些特定引脚的复用对 BBB 用户来说非常重要，因为复用决定了哪些处理器信号和功能可以容易被用户用于硬件接口。BBB 的两个头各有 46 个引脚，总共有 92 个引脚可供接口使用。不幸的是，默认情况下有 61 个引脚正在使用，这意味着在不禁用 BBB 的一个或多个标准功能的情况下，只有 31 个引脚可以为你项目所变动。

![BBB 引脚复用](img/00008.jpeg)

BeagleBone Black 的 P8 和 P9 扩展头

头上的某些引脚是永久分配的，例如提供访问电压（1.8、3.3 和 5 VDC 可用）和地线的引脚。然而，其他引脚可以根据项目的需要进行复用。正确复用所有的 P8/P9 引脚以提供你所需要的所有资源有时可能很棘手，特别是如果你刚开始学习 BBB 的硬件接口方面。幸运的是，我们已经为你确定了一个引脚复用配置，这将提供给 PacktHAL 运行本书中所有练习所需的所有硬件资源。

![BBB 引脚复用](img/00009.jpeg)

BeagleBone Black 上默认使用的引脚

## 内核 Device Tree 和 capemgr

BBB 的引脚必须以特定的方式复用以与自定义硬件通信，但实际在哪里以及如何进行呢？"答案是"内核的**Device Tree**。" Device Tree 是内核中的一个层次化数据结构，描述了存在哪些硬件，这些硬件使用了哪些资源，以及应该使用哪些内核驱动程序与每个硬件设备通信。它描述了硬件的不同方面，例如引脚复用设置、时钟速度和传递给内核设备驱动程序的参数。

如果要求用户在每次硬件更改时都安装新内核，这将是一件非常麻烦的事情。对于 BBB 这样的硬件平台，用户可以在电源周期之间更改连接到 BBB 的硬件！能够动态地更改 Device Tree 以即时添加或移除硬件将非常有用。BBB 的 Linux 3.8 内核有一个特殊的子系统，称为**cape 管理器**（**capemgr**），它允许你这样做。

capemgr 动态地添加和移除设备树的片段或*覆盖层*。它提供了三项重要的服务：

+   它识别任何连接到 BBB 的 Cape 硬件

+   它加载适当的设备树覆盖层以启用和配置每个被识别的 Cape

+   它允许从用户空间动态加载任意的设备树覆盖层，以配置任何未被自动发现的硬件。

## 定义 Cape

Cape 是任何连接到 BBB 的 P8/P9 连接器（类似于盾板连接到 Arduino）的硬件扩展，并包含一个**电可擦可编程只读存储器**（**EEPROM**）芯片，向内核的 capebus 报告 Cape 的身份。内核中的 capemgr 然后可以为该特定 Cape 动态启用适当的设备树覆盖层。这就是允许你将各种不同的商业 Cape 板连接到 BBB，并且它们全部自动工作，而无需你更改任何配置文件的原因。

对 Cape 的定义较为宽松的是指任何通过 P8/P9 连接器接口的外部电路。如果没有包含一个 EEPROM 来告诉 capemgr “我是一个名为 XYZ 的 Cape”，capemgr 便不会自动定位并加载适合该 Cape 的正确设备树覆盖层。本书中的所有示例都是这种情况。你仍然可以将连接到 BBB 的硬件视为 Android 正在接口的 Cape，但设备树覆盖层必须从用户空间手动加载。

在本章前面，你使用了 `adb` 将一个名为 `BB-PACKTPUB-00A0.dtbo` 的文件推送到你的 Android 映像中。这个文件是配置 BBB 以适应你将在本书练习中使用的硬件的设备树覆盖层。你同样推送过去的自定义 `init.genericam33xx(flatteneddevicetr.rc` 文件在 Android 启动过程中为你手动加载了这个覆盖层。

在 Linux 文件系统中，自定义覆盖层被放置在 `/lib/firmware` 目录中。但在 Android 下，`rootfs` 中没有 `/lib` 目录，因此覆盖层被放置在 `/system/vendor/firmware` 目录中。这也是在内核编译期间构建的固件（`.fw` 文件）安装的位置。在使用你未来的项目中的自定义设备树覆盖层时，请记得将它们放置到 `/system/vendor/firmware` 目录中，以便 capemgr 能够找到它们。

### 提示

**我在哪里可以了解更多关于复用 BBB 的引脚、设备树以及创建自定义覆盖层的信息？**

学习如何为自定义项目选择最佳的引脚复用（pin muxing）并创建适当的设备树覆盖层超出了本书的范围，但有许多优秀的资源可以介绍你了解这个过程。以下是我们推荐你阅读的一些学习更多知识的优秀资源：

+   BeagleBone Black 系统参考手册：[`www.adafruit.com/datasheets/BBB_SRM.pdf`](http://www.adafruit.com/datasheets/BBB_SRM.pdf)

+   Derek Molloy 的网站：[`derekmolloy.ie/category/embedded-systems/beaglebone/`](http://derekmolloy.ie/category/embedded-systems/beaglebone/)

+   AdaFruit 的 Device Tree Overlay 教程：[`learn.adafruit.com/introduction-to-the-beaglebone-black-device-tree`](https://learn.adafruit.com/introduction-to-the-beaglebone-black-device-tree)

# 总结

在本章中，我们解释了 Android 如何使用 HAL 让 Android 管理器向应用提供硬件访问权限。我们向你介绍了 PacktHAL，它可用于与本书中的所有示例进行接口交互。你配置了 BBBAndroid 镜像以使用 PacktHAL，并且修改了你的 NDK 安装，以便将 PacktHAL 构建到你的应用中。

我们还展示了 BBB 的 P8/P9 头部哪些引脚可以进行复用，Device Tree 是什么以及如何使用它来复用引脚，以及 capemgr 如何加载 Device Tree 覆盖层以动态复用 BBB 的引脚。

在下一章中，你将开始使用 PacktHAL 并构建你的第一个使用 GPIOs 的硬件接口应用。
