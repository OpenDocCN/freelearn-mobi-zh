# 启用VirtualBox特定硬件接口

在上一章中，我们对Android Gralloc HAL模块进行了深入分析。我们分析了默认的Gralloc模块和Android模拟器的硬件GPU仿真Gralloc HAL。我们还没有时间走完与图形系统相关的启动过程。在本章中，我们将走完图形系统的启动过程，并探索VirtualBox特定的硬件驱动程序。在本章结束时，我们将在VirtualBox上拥有一个相对完整的系统。本章我们将涵盖以下主题：

+   OpenGL ES和图形硬件初始化

+   VirtualBox Guest Additions的集成

# OpenGL ES和图形硬件初始化

在Android系统中，图形系统的初始化是由SurfaceFlinger完成的。除了我们在第10章中讨论的Gralloc HAL之外，*启用图形*是图形系统初始化的另一个重要部分，即加载OpenGL ES库。在我们的VirtualBox实现中，我们使用了Android-x86的大部分HAL模块。图形系统支持包括以下组件：

+   Gralloc HAL

+   Mesa库用于OpenGL ES

+   uvesafb帧缓冲区驱动程序或VirtualBox视频驱动程序

我们在上章中讨论了Gralloc HAL。在本章中，我们将探讨OpenGL ES库和uvesafb帧缓冲区驱动程序的加载。在介绍图形系统初始化时，我们将使用默认的uvesafb帧缓冲区驱动程序。在后面讨论VirtualBox Guest Additions的集成时，我们还将介绍如何从VirtualBox中使用本地图形驱动程序。

# 加载OpenGL ES库

**OpenGL ES**代表**OpenGL嵌入式系统**，它是Khronos的OpenGL的一个子集。EGL是OpenGL ES和底层本地平台之间的接口。EGL的API应该是平台无关的，但EGL API的实现不是。

Android中OpenGL ES的实现包括Java API和本地实现。这两部分可以在以下位置找到：

+   Java API：`$AOSP/frameworks/base/opengl`

+   OpenGL ES本地：`$AOSP/frameworks/native/opengl`

这两部分OpenGL实现依赖于供应商实现来提供OpenGL ES API的完整功能。在系统启动过程中，系统将搜索`/system/lib/egl`或`/vendor/lib/egl`路径以找到供应商的OpenGL库。

OpenGL ES供应商库应遵循以下命名约定。如果供应商库是一个单独的库，它应使用`libGLES_*.so`的名称。在我们的例子中，VirtualBox的OpenGL ES库是`libGLES_mesa.so`，它作为一个单独的库提供。

如果供应商库作为单独的库提供，它们必须类似于以下内容：

+   `/system/lib/egl/libEGL_*.so`

+   `/system/lib/egl/libGLESv1_CM_*.so`

+   `/system/lib/egl/libGLESv2_*.so`

这适用于 Android 模拟器的硬件仿真库。我们可以找到以下 Android 模拟器的库：

+   `/system/lib/egl/libEGL_emulation.so`

+   `/system/lib/egl/libGLESv1_CM_emulation.so`

+   `/system/lib/egl/libGLESv2_emulation.so`

供应商库在 `SurfaceFlinger` 初始化期间加载。在我们深入探讨启动过程之前，让我们先看看调试日志中的消息。

我已经从下面的日志中移除了时间戳，以便我们有一个更好的格式：

[PRE0]

如我们所见，当 `SurfaceFlinger` 的主线程准备运行时，它会在 x86vbox 设备启动期间加载 `/system/lib/egl/libGLES_mesa.so` 库。之后，它加载并初始化 `gralloc.default.so` Gralloc 模块：

[PRE1]

接下来，`SurfaceFlinger` 初始化 EGL 库，正如前面的日志消息所示。我们环境中的 EGL 版本是 1.4：

[PRE2]

在 EGL 初始化之后，OpenGL ES 库被初始化，正如我们从前面的日志消息中可以看到的。我们可以看到 Mesa 库支持 OpenGL ES 3.0。渲染引擎是一个使用 Gallium 和 `llvmpipe` 的软件实现。

每个图形硬件供应商通常都有自己的 OpenGL 实现。Mesa 是 OpenGL 的开源实现。Mesa 为 OpenGL 提供了多个后端支持。它可以根据硬件 GPU 支持硬件和软件实现。如果您没有硬件 GPU，Mesa 有三个基于 CPU 的实现：swrast、softpipe 和 llvmpipe。我们在 x86vbox 中使用的是 llvmpipe。Mesa 驱动实现有两种架构。Gallium 是 Mesa 驱动实现的新架构。

# 分析加载过程

在我们对 x86vbox 中 OpenGL ES 实现进行一般介绍（重用自 Android-x86）之后，我们将分析源代码，以获得更深入的理解。由于图形系统和 OpenGL ES 的详细实现非常庞大，我们无法在一个章节中涵盖它们。在我们的分析中，我们将专注于图形系统的加载过程和 OpenGL ES 库。

再次，当我们遍历源代码时，您可能会感到沮丧。帮助您最好的方法是，在阅读本章内容的同时打开您的源代码编辑器。如果您手头没有 AOSP 源代码，您始终可以参考以下网站：

[http://xref.opersys.com/](http://xref.opersys.com/)

您只需搜索本章中讨论的函数名，即可定位源代码。

从前面的调试日志中，我们将从看到第一个与图形系统和 `SurfaceFlinger` 相关的调试信息的位置开始，如下所示：

[PRE3]

第一条信息是由 `SurfaceFlinger` 的构造函数打印的，第二条信息是从 `SurfaceFlinger` 的 `init` 方法打印出来的。

`SurfaceFlinger` 的源代码可以在以下位置找到：`$AOSP/frameworks/native/services/surfaceflinger/SurfaceFlinger.cpp`。

我们将从**SurfaceFlinger:init**开始分析，根据以下图中所示的流程：

![图片](img/image_11_001.png)

OpenGL ES库的加载

在`SurfaceFlinger:init`中，如以下代码片段所示，它首先调用EGL函数`eglGetDisplay`。之后，它尝试创建一个硬件合成器实例。使用显示实例`mEGLDisplay`和硬件合成器`mHwc`，它使用底层的OpenGL ES实现创建了一个渲染引擎：

[PRE4]

让我们先分析EGL函数`eglGetDisplay`。`eglGetDisplay`函数在`frameworks/native/opengl/libs/EGL/eglApi.cpp`文件中实现，如下面的代码片段所示：

[PRE5]

在`eglGetDisplay`函数中，它首先检查要初始化的显示索引。在当前的Android代码中，`EGL_DEFAULT_DISPLAY`参数为0，`NUM_DISPLAYS`的定义为1。这意味着当前Android实现只能支持一个显示。这里是什么意思呢？例如，如果你有一台笔记本电脑，你可以将其连接到投影仪。在这种情况下，你可以同时拥有两个显示。现在一些新电脑甚至可以同时连接三个显示。在检查显示数量后，它调用`egl_init_drivers`函数来加载OpenGL ES库：

[PRE6]

`egl_init_drivers`函数获取一个互斥锁并调用另一个函数`egl_init_drivers_locked`来加载OpenGL ES库。在`egl_init_drivers_locked`函数中，它获取一个`Loader`类的实例，该类使用**单例模式**定义。之后，它初始化全局变量`gEGLImpl`，该变量定义为`egl_connection_t`数据结构：

[PRE7]

在`egl_connection_t`数据结构中，它定义了以下字段：

+   `dso`: 这是一个指向`driver_t`数据结构的指针，该数据结构在`Loader`类内部定义。这个`driver_t`数据结构存储了`Loader`类加载后的OpenGL ES库的句柄。

+   `hooks`: 这是一个指向`gl_hooks_t`数据结构指针的数组。`gl_hooks_t`数据结构用于定义OpenGL ES API的所有函数指针。在OpenGL ES库加载后，库中的OpenGL ES函数将被初始化并分配给`hooks`字段。在`enum { GLESv1_INDEX , GLESv2_INDEX }`中定义了两个OpenGL ES版本。`hooks[GLESv1_INDEX]`用于存储OpenGL ES 1 API，并指向全局变量`gHooks[GLESv1_INDEX]`。对于`GLESv2_INDEX`也是同样的情况。OpenGL ES API的列表可以在以下文件中找到：`$AOSP/frameworks/native/opengl/libs/entries.in`

+   `major`和`minor`: 这两个用于存储EGL版本。

+   `egl`: 这被定义为`egl_t`，用于存储EGL API。EGL API的列表可以在以下文件中找到：`$AOSP/frameworks/native/opengl/libs/EGL/egl_entries.in`

+   `libEgl`、`libGles1`和`libGles2`：这些都是OpenGL ES包装库的句柄。我们稍后将看到这些库的初始化。

在`cnx`数据结构初始化后，它调用`loader.open`函数来加载库。让我们看看`loader.open`函数：

[PRE8]

在`Loader::open`中，它首先尝试加载单个OpenGL ES库。如果失败，它将逐个尝试加载分离的库。如果库加载成功，它将把句柄存储到`driver_t`数据结构中。我们之前在讨论`egl_connection_t`数据结构中的`dso`字段时解释了`driver_t`。实际的加载过程是在`load_driver`函数中完成的，我们很快就会看到它。在OpenGL ES库加载后，它还尝试使用`load_wrapper`函数加载包装库。`load_wrapper`函数只是调用`dlopen`系统调用并返回句柄，所以我们不需要调查它。

# 加载驱动程序

让我们分析`load_driver`函数，这是找到和加载OpenGL ES用户空间驱动程序的函数：

[PRE9]

在`load_driver`函数中，它定义了一个`MatchFile`内部类。它使用`MatchFile::find`方法来查找库的路径。`load_driver`函数有三个参数：`kind`、`cnx`和`mask`。根据库的类型，参数`kind`可以是`GLES`、`EGL`、`GLESv1_CM`或`GLESv2`。一旦它获取到库的绝对路径，它就调用`dlopen`系统函数来打开共享库。`mask`参数是`kind`参数的位图。使用`mask`参数，它可以根据库的类型初始化`cnx`参数。正如我们之前提到的，`cnx`参数，它是一个`egl_connection_t`实例，有一个`egl`字段来存储所有EGL函数指针。它还有一个字段，`hooks[GLESv1_INDEX]/hooks[GLESv2_INDEX]`，用于存储所有OpenGL ES函数。

如果库类型是EGL，它首先通过调用`dlsym`系统函数来获取`eglGetProcAddress`函数的地址。之后，它将遍历在`egl_names`全局变量中定义的所有函数名称，以找出地址并将它们存储在`cnx->egl`中。在这个过程中，它首先尝试使用`dlsym`系统函数获取地址。如果`dlsym`调用失败，它将再次尝试使用`eglGetProcAddress`函数。

如果库类型是`GLESv1_CM`或`GLESv2`，它将调用另一个函数`init_api`来初始化所有OpenGL ES函数指针。在`init_api`函数中，它将遍历在`gl_names`全局变量中定义的所有函数名称，以找出地址并将它们存储在`cnx->hooks[egl_connection_t::GLESv?_INDEX]->gl`中。

现在我们已经完成了OpenGL ES用户空间驱动程序的初始化，我们可以使用`egl_connection_t`数据结构来访问所有OpenGL ES供应商API。

# 创建渲染引擎

在加载OpenGL ES供应商库之后，`SurfaceFlinger:init`将创建渲染引擎：

[PRE10]

在`RenderEngine::create`内部，它将调用`RenderEngine::chooseEglConfig`，这将打印出EGL的调试信息：

[PRE11]

在`RenderEngine::create`的末尾，它将打印出以下OpenGL ES初始化信息：

[PRE12]

# uvesafb帧缓冲区驱动程序

帧缓冲区驱动程序是我们需要支持的x86vbox图形系统的第三个组件。由于您可能在不同的英特尔设备上运行VirtualBox，它们可能使用不同的图形硬件，例如Nvidia、AMD或英特尔。为了在虚拟化环境中获得最佳性能，您可能想探索各种GPU虚拟化技术，如GPU直通、GPU共享、GPU软件仿真等。为了有一个简单的解决方案，我们使用了Android-x86的默认解决方案，即uvesafb帧缓冲区驱动程序。

# 什么是uvesafb？

uvesafb是一个与VESA 2.0兼容的图形卡一起工作的用户空间VESA帧缓冲区驱动程序。VESA BIOS扩展通过BIOS接口提供了VESA标准的主要功能。在Linux上，uvesafb需要一个名为`v86d`的用户空间守护进程作为需要执行x86 BIOS代码的内核驱动程序的后端。由于BIOS代码只能在受控环境中执行，因此`v86d`执行的代码可以在完全软件模拟的环境或由CPU支持的虚拟化环境中运行。Android-x86项目已经将`v86d`移植到Android。它可以在`$AOSP/external/v86d`找到。由于`v86d`项目需要额外的系统调用，如`ioperm`和`iopl`，Android-x86项目将bionic库更改为支持这些系统调用。

您可以参考以下内核文档以了解更多关于uvesafb的信息：

[https://www.kernel.org/doc/Documentation/fb/uvesafb.txt](https://www.kernel.org/doc/Documentation/fb/uvesafb.txt)

# 测试uvesafb帧缓冲区驱动程序

在我们尝试了解uvesafb在我们环境中是如何加载之前，我们可以使用两个帧缓冲区测试工具`fbset`和`fbtest`来测试它。

正如我们所知，我们可以使用Android-x86的二级引导从[第9章](c8d10155-cc8e-4b8c-a5e0-f359520c894a.xhtml)，“使用PXE/NFS引导x86vbox”进行两阶段引导进入调试控制台。我们可以在调试控制台中用`fbset`和`fbtest`测试uvesafb。

`fbset`是一个显示或更改帧缓冲区设备设置的系统工具。您可以通过查看Linux命令的帮助页面来了解如何使用`fbset`。在我们的环境中，我们在第一阶段引导中使用`busybox`，在Android环境中使用`toybox`或`toolbox`。由于`fbset`由`busybox`支持，因此我们可以通过`busybox`在第一阶段或第二阶段引导中使用它。

`fbtest`是一个帧缓冲区测试程序，可以在[https://git.kernel.org/pub/scm/linux/kernel/git/geert/fbtest.git](https://git.kernel.org/pub/scm/linux/kernel/git/geert/fbtest.git)找到。

我从内核Git仓库克隆了它，并将其移植到Android环境中。Android的源代码可以通过以下链接在GitHub上找到：

[https://github.com/shugaoye/fbtest](https://github.com/shugaoye/fbtest)

要构建`fbtest`，我们可以从GitHub获取它，并在AOSP构建环境中构建：

[PRE13]

在我们设置好AOSP构建环境后，我们可以使用以下命令检出并构建`fbtest`源代码：

[PRE14]

注意，我已经修改了Makefile，并且它依赖于AOSP环境变量`$OUT`，如下所示：

[PRE15]

如果我们查看前面的`fbtest/Rules.make` Makefile，我们使用`$OUT`环境变量来找到正确的AOSP构建环境。之后，我们可以使用预构建的工具链和`bionic`库来构建`fbtest`。

在我们构建`fbtest`后，我们可以将其复制到`$OUT/system/bin`文件夹，这样我们就可以在测试环境中稍后使用它。正如我们从[第9章](c8d10155-cc8e-4b8c-a5e0-f359520c894a.xhtml)，“使用PXE/NFS引导x86vbox”中记得的那样，我们可以使用PXE/NFS在第一阶段引导时引导到调试控制台。在这种情况下，我们可以更改并测试`fbtest`而不需要重新引导x86vbox，因为我们可以从x86vbox通过NFS访问构建结果。

让我们在第一阶段引导时引导x86vbox到调试控制台，并执行测试。正如我们回忆的那样，我们在第一阶段引导的调试控制台中有一个最小嵌入式Linux环境。在这个环境中我们有内置的`busybox`。在我们测试帧缓冲设备之前，我们必须手动加载`uvesafb`模块，如下面的截图所示：

![图片](img/image_11_002.png)

我们使用以下命令加载`uvesafb`模块：

[PRE16]

从调试输出中，我们可以看到底层图形硬件是`Oracle VM VirtualBox VBE Adapter`。

在加载`uvesafb`模块后，我们可以找到`/dev/fb0`设备。我们可以使用`fbset`来更改帧缓冲设备设置。例如，我们可以根据需要切换到不同的支持分辨率。让我们运行`fbset`命令看看会发生什么。如果我们不带任何参数运行`fbset`，我们可以看到以下输出：

[PRE17]

如果不带任何参数运行`fbset`，它只会打印出帧缓冲设备的当前设置。正如我们从输出中可以看到的，如果我们不带任何参数加载`uvesafb`，默认分辨率为640 x 480，16位颜色。

我们可以用分辨率的名称尝试更改分辨率，如下所示：

[PRE18]

我们得到了一个错误信息，告诉我们分辨率在`/etc/fb.modes`文件中未定义。我们需要创建此文件来更改分辨率。我们可以在`/etc/fb.modes`中添加以下分辨率，如下所示：

[PRE19]

现在我们可以测试分辨率更改。如果我们运行以下命令，我们可以切换到具有真彩色的更高分辨率：

[PRE20]

在我们加载帧缓冲驱动程序并测试配置更改后，我们可以通过在屏幕上绘制一些东西来测试帧缓冲。使用我们在本节中构建的 `fbtest` 命令，我们可以运行一系列帧缓冲测试用例。首先，让我们找出 `fbtest` 可以运行多少个测试用例：

[PRE21]

如果我们在 `fbtest` 中使用 `-l` 选项，它将打印出可用的测试用例列表。我们可以看到我们有 12 个测试用例：

[PRE22]

例如，我们可以运行测试用例 002，我们将看到以下屏幕。您可以自由地测试任何前面的测试用例。

![初始化 uvesafb 在 x86vbox](img/image_11_003.png)

# 在 x86vbox 中初始化 uvesafb

在 x86vbox 中初始化 `uvesafb` 是在启动脚本 `init.sh` 中完成的。如果我们回顾第 8 章[创建您的虚拟机设备](acf2363a-2a0f-40b9-a35f-c8bb0e523737.xhtml)中关于 HAL 初始化的讨论，我们可以看到 `init.sh` 中的以下代码。我们在第 8 章[创建您的虚拟机设备](acf2363a-2a0f-40b9-a35f-c8bb0e523737.xhtml)中简要讨论了图形 HAL 的初始化，现在我们可以深入了解细节：

[PRE23]

在我们当前的设置中，让我们看看 `/proc/fb` 的内容。我们可以从调试控制台或 adb 控制台来检查这一点。在帧缓冲设备初始化之前，`/proc/fb` 的内容是空的。在我们的情况下，它是空的，因为没有帧缓冲设备可用，直到执行 `init.sh` 脚本。如果输出为空，`init.sh` 脚本将调用 `init_uvesafb` 函数来初始化 `uvesafb`。在帧缓冲设备初始化后，我们可以看到 `/proc/fb` 的内容如下：

[PRE24]

如果在调用 `init.sh` 之前有帧缓冲设备可用，`init_hal_gralloc` 将为 DRM 驱动程序设置 `ro.hardware.gralloc` 系统属性。对于 `init_hal_gralloc` 无法处理的设备，它将不执行任何操作。

在 `init_uvesafb` 中，加载 `uvesafb` 的实际命令可以扩展到以下之一：

[PRE25]

`uvesafb` 的选项有：

+   `mtrr:n`: 为帧缓冲设置内存类型范围寄存器，其中 `n` 可以是：

    +   `0`: 禁用（相当于 `nomtrr` 选项）

    +   `3`: 写合并（默认）

内存类型范围寄存器是英特尔处理器中一组处理器补充功能控制寄存器。写合并允许将总线写传输组合成更大的传输，然后再在总线上爆发。这可以帮助提高图形性能。

+   `redraw`: 通过重新绘制受影响的屏幕部分来滚动。

+   `mode_option`: 设置分辨率到一个支持的值。

在加载 `uvesafb` 之后，我们可以使用以下命令找到所有支持的分辨率：

[PRE26]

# 集成 VirtualBox Guest Additions

到目前为止，我们可以启动 x86vbox 到 Android。我们可以进一步做的是将 VirtualBox Guest Additions 集成到 x86vbox 中。

VirtualBox 是一个虚拟化环境。我们可以在 VirtualBox 中安装一个虚拟操作系统，就像它真的存在一样。然而，以这种方式工作有一些限制。在主机环境中运行虚拟操作系统时，你可能期望的不仅仅是硬件虚拟化。例如，当你在不同主机和虚拟系统之间移动鼠标光标时，你可能发现鼠标光标表现不佳。你可能希望轻松地在主机和虚拟机之间共享数据，例如共享剪贴板、共享文件夹等。为了满足这些需求，主机和虚拟机需要相互了解并有一种相互交流的方式。在 VirtualBox 架构中，有一个称为 **主机-虚拟机通信管理器**（**HGCM**）的组件。在主机端，VirtualBox 实现了一个名为 HGCM 服务的功能，可以响应虚拟机的请求。在虚拟机端，有几个来自 VirtualBox 的内核驱动程序，可以用来与主机通信。

VirtualBox 为主机和虚拟机集成提供的附加功能通常包含在一个名为 **VirtualBox 扩展包**（**VirtualBox Extension Pack**）的包中。在 VirtualBox 扩展包中，它包括主机端和虚拟机端所需的所有文件。VirtualBox 扩展包可以从 [https://www.virtualbox.org/wiki/Downloads](https://www.virtualbox.org/wiki/Downloads) 下载。

对于虚拟机端，有一个包含设备驱动程序二进制工具和源代码的单独分发包，称为 VirtualBox 虚拟机增强功能。有针对 Windows、Linux 和 OS X 的单独 VirtualBox 虚拟机增强功能。没有针对 Android 的虚拟机增强功能。然而，由于 Android 使用 Linux 内核，我们可以使用 Linux 的源代码构建 Android 的内核驱动程序。安装 VirtualBox 扩展包后，我们可以找到一个名为 `VBoxGuestAdditions.iso` 的镜像文件，如下所示：

[PRE27]

我们可以提取这个镜像文件，并在 VirtualBox 虚拟机增强功能中找到以下文件：

[PRE28]

有两个压缩文件：`VBoxGuestAdditions-amd64.tar.bz2` 和 `VBoxGuestAdditions-x86.tar.bz2`。如以下截图所示，这是为 Intel x86 Linux 虚拟机提供的虚拟机增强功能的文件夹和文件列表：

![图片](img/image_11_004.png)

VirtualBox 虚拟机增强功能

在虚拟机增强功能中有三个驱动程序的源代码可用：`vboxguest`、`vboxsf` 和 `vboxvideo`：

+   `vboxguest`: 这个模块为虚拟操作系统提供与主机通信的基本服务。

+   `vboxsf`: 这个模块是一个内核驱动程序，用于在主机和虚拟机之间共享文件的能力。

+   `vboxvideo`: 这个模块是针对虚拟机的视频驱动程序。通过这个驱动程序，我们可以通过主机使用图形硬件加速。

我们将构建并将这三个驱动程序集成到 x86vbox 中。

# 构建 VirtualBox 虚拟机增强功能

Guest Additions 中的驱动程序的唯一依赖项是内核源代码。为 Android 构建驱动程序非常简单。要构建 Guest Additions，你可以从你的 VirtualBox 安装中获取源代码，或者你可以从我的 GitHub 上获取一个版本，如下所示：

[PRE29]

在我们成功构建驱动程序后，我们可以按照以下方式找到驱动模块：

[PRE30]

我们可以在我们的 `x86vbox` 设备文件夹下的 `vbox` 文件夹中存储内核模块，因此我们可以在构建过程中将它们复制到文件系统中：

[PRE31]

在我们获得 Guest Additions 的可加载模块后，我们可以将它们添加到我们的 x86vbox 设备 Makefile `x86vbox.mk` 中，如下所示：

[PRE32]

这三个模块将被复制到系统镜像中的 `/system/vendor/vbox` 文件夹。

# 集成 vboxsf

使用可加载模块 `vboxsf.ko`，我们能够在 Android 系统运行时在主机和虚拟机之间交换文件。要创建主机和虚拟机之间的共享文件夹，我们需要首先加载 `vboxsf.ko` 模块。

要使用 `vboxsf.ko`，我们需要一个名为 `mount.vboxsf` 的工具，它可以用来将主机文件系统上的共享文件夹挂载到 Android 文件系统。这个 `mount.vboxsf` 工具是 VirtualBox Guest Additions 提供的实用工具之一。我们将其放在我们的 x86vbox 设备文件夹中，如下所示：

[PRE33]

它包括一个 C 文件和一个头文件。我们创建了以下 Android Makefile 来构建它：

[PRE34]

为了将其包含在系统镜像中，我们还需要将其添加到 `x86vbox.mk` Makefile 中，如下所示：

[PRE35]

为了在系统启动时加载 `vboxsf.ko`，我们需要将 `vboxsf.ko` 的加载添加到 `initrd.img` 中的启动脚本。如果我们回想一下 [第 6 章](b6a6462f-1c3d-46fd-89e9-a543423c576d.xhtml)，*使用自定义 Ramdisk 调试启动过程*，我们讨论了 `initrd.img` 中的 init 脚本。shell 脚本函数 `load_modules` 被调用以在第一阶段启动时加载大部分设备驱动程序。我们可以更改此脚本以加载 VirtualBox 设备驱动程序，如下所示：

[PRE36]

我们定义了一个 `VBOX_GUEST_ADDITIONS` 内核参数，它可以用来启用加载 VirtualBox 特定设备驱动程序。如果定义了这个内核参数，我们将加载两个可加载模块，`vboxguest.ko` 和 `vboxsf.ko`。另一个内核参数 `SDCARD` 也被定义，这样我们就可以将共享文件夹挂载为外部 SD 卡存储。`SDCARD` 内核参数被 shell 脚本函数 `mount_sdcard` 使用。

要在内核命令行上定义这两个内核参数，我们需要更改 `$HOME/.VirtualBox/TFTP/pxelinux.cfg/default` 下的 PXE 启动脚本，如下所示：

[PRE37]

注意两个变量 `SDCARD` 和 `VBOX_GUEST_ADDITIONS`。它们是我们添加的两个新内核参数，用于支持加载 VirtualBox 设备驱动程序。为了挂载共享文件夹，我们在脚本中添加以下命令：

[PRE38]

`mount.vboxsf`的第一个参数是我们定义在VirtualBox设置中的共享文件夹，如下面的截图所示：

![图片](img/image_11_005.png)

通过所有与共享文件夹相关的更改，我们可以有一个可以用来在主机和客户机之间轻松共享数据的方法。

# 集成vboxvideo

在VirtualBox的Guest Additions中，还有一个可以在Android上使用的设备驱动程序，即`vboxvideo.ko`。这是一个视频硬件的设备驱动程序。与本章中刚刚讨论的uvesafb相比，它提供了一个功能更强大的视频驱动程序。

uvesafb是基于VESA 2.0标准的标准帧缓冲驱动程序，它不支持在VirtualBox上的硬件加速。`vboxvideo.ko`是一个支持硬件加速的基于DRM/DRI的视频驱动程序。

**直接渲染基础设施**（**DRI**）是Linux平台上X Window系统的新架构，允许*X*客户端直接与图形硬件通信。**直接渲染管理器**（**DRM**）是DRI架构的内核部分。

Android-x86项目是第一个将Mesa/DRM引入Android平台的开源项目。这是一个针对支持的图形硬件的开源OpenGL ES实现。通过以下组件，我们应该能够支持Android上的OpenGL ES的硬件加速：

+   `external_libdrm`

+   `external_mesa`

+   `external_drm_gralloc`

我们在VirtualBox上已经有了`vboxvideo`的DRM驱动程序，但相关的实现仍需要添加到`external_mesa`和`external_drm_gralloc`中，以支持使用主机GPU的OpenGL ES。

在`external_mesa`和`external_drm_gralloc`中没有针对VirtualBox的特定实现，我们只能使用Mesa的相同基于软件的实现来支持OpenGL ES和默认的Gralloc模块`gralloc.default.so`。这就是为什么大多数基于VirtualBox的模拟器解决方案，如Genymotion、Andy或AMI DuOS，仍然在使用硬件GPU模拟，这与我们在第10章“启用图形”部分中讨论的*硬件GLES模拟概述*中的类似。

要加载`vboxvideo.ko`，我们需要在`load_modules`中添加以下这三行：

[PRE39]

`ttm`和`drm_kms_helper`内核模块是`vboxvideo.ko`所需的两个内核模块。我们还使用`VBOX_VIDEO_DRIVER`内核参数来配置`vboxvideo.ko`的加载。使用这个内核参数，我们可以在uvesafb帧缓冲和VirtualBox帧缓冲之间切换。系统启动后，我们可以看到以下日志消息。我们可以看到`vboxvideo`已成功加载：

[PRE40]

从日志消息中，我们可以看到`vboxvideo`创建了一个`vboxdrmfb`帧缓冲设备。我们可以使用`fbset`来检查帧缓冲设置，就像我们之前做的那样。我们可以看到，对于`vboxdrmfb`，硬件加速被设置为true：

[PRE41]

我们也可以检查 `/proc/fb` 的输出。由于输出是 `0 vboxdrmfb`，`init.sh` 中的 `init_hal_gralloc` 脚本函数不会加载 `uvesafb`：

[PRE42]

使用这种设置，我们可以使用 `vboxvideo` 驱动程序而不是 `uvesafb` 来启动 x86vbox。正如我提到的，在我们能够充分利用 `vboxvideo` 的所有潜在功能之前，还有很多工作要做。

# 使用 VirtualBox Guest Additions 构建和测试镜像

要构建和测试本章中的镜像，我们可以使用 `repo` 工具检索本章中的源代码，如下所示：

我们可以使用以下命令从 GitHub 和 AOSP 获取源代码：

[PRE43]

在我们获取本章的源代码后，我们可以设置环境和构建系统，如下所示：

[PRE44]

要构建 `initrd.img`，我们可以运行以下命令：

[PRE45]

# 摘要

在本章中，我们学习了图形系统的启动过程。这包括 OpenGL ES 库、Gralloc 模块和设备驱动程序。我们在上一章中讨论了 Gralloc 模块。在本章中，我们分析了另外两个组件，即 OpenGL ES 库和帧缓冲区驱动程序。在掌握所有这些知识的基础上，我们将来自 VirtualBox Guest Additions 的驱动程序集成到了 x86vbox 设备中。在下一章中，我们将开始另一个项目，以探索 Android 系统中恢复功能的工作原理。
