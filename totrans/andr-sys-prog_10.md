# 启用图形

在上一章中，我们学习了如何使用PXE和NFS启动x86vbox设备。我们可以将设备引导到嵌入式Linux环境，这是Android-x86引导的第一阶段。在这个阶段，我们可以使用调试控制台来验证系统的状态，以确保我们在启动真正的Android系统之前一切正常。在本章中，我们将讨论在Android系统启动过程中遇到的第一问题。这是关于如何为x86vbox设备启用Android图形系统。本章将涵盖以下主题：

+   Android图形架构概述

+   深入探讨图形HAL

+   分析Android模拟器图形HAL进行比较

图形系统可能是Android系统架构中最复杂的软件堆栈。

正如您将看到的，本章的内容比其他章节都要长。阅读和理解本章的内容可能更困难。我的建议是，您可以在阅读本章的同时打开源代码编辑器并加载相关源代码。这将极大地帮助您理解源代码以及我在本章中想要阐述的点。

# Android图形架构简介

Android中的图形系统与我们讨论过的架构相似，即[第3章](e0f861c2-5832-402f-89d3-cfc75785e759.xhtml)中提到的*发现内核、HAL和虚拟硬件*。在那里，我们以goldfish lights HAL为例，从应用层到HAL和设备驱动层进行了详细分析。这种分析有助于我们垂直理解Android架构。

然而，图形系统可能是Android架构中最复杂的系统。需要另一本书来详细介绍Android图形系统。本书的重点是如何将Android系统移植到新的硬件平台。为了专注于这个目标，我们将在本章中讨论图形HAL，而不是整个图形系统。如果我们能选择正确的图形HAL并正确配置它，图形系统就能正常工作。

根据谷歌关于图形实现文档，Android图形支持需要以下组件：

+   EGL驱动程序

+   OpenGL ES 1.x驱动程序

+   OpenGL ES 2.0驱动程序

+   OpenGL ES 3.x驱动程序（可选）

+   Vulkan（可选）

+   Gralloc HAL实现

+   硬件合成器HAL实现

在前面的列表中，OpenGL ES实现是图形系统中最复杂的组件。我们将讨论它如何在Android模拟器和Android-x86中选取和集成，但我们不会深入讨论如何分析OpenGL ES实现，而是会概述底层的OpenGL ES库。Android系统中必须支持OpenGL ES 1.x和2.0。OpenGL ES 3.x目前是一个可选组件。EGL驱动通常作为OpenGL ES实现的一部分实现，我们将在讨论Android模拟器和Android-x86（x86vbox）图形系统时看到这一点。

Vulkan是Khronos Group新一代GPU API。Vulkan是新的且可选的，仅在Android 7中引入。涵盖Vulkan超出了本书的范围，因此我们不会讨论它。Gralloc HAL是处理图形硬件的HAL，是我们深入分析的重点。在大多数图形系统的移植工作中，Gralloc HAL是启用图形的关键。

硬件合成器是图形HAL的一部分。然而，它不是Android模拟器或Android-x86必须拥有的组件。**硬件合成器**（**HWC**）HAL用于将表面合成到屏幕上。HWC抽象了如叠加层等对象，并帮助卸载一些通常使用OpenGL完成的工作。

![图片10_001](img/image_10_001.png)

Android图形架构

如前所述的Android图形架构图所示，我们也可以将相关组件在Android架构中划分为不同的层，就像我们在前面的章节中所做的那样。这个架构图是图形系统的一个简化视图。**SurfaceFlinger**是提供图形相关系统支持的系统服务，面向应用层。**SurfaceFlinger**将连接到**OpenGL ES**库和**HAL**层组件以执行实际工作。在**HAL**中，我们有**HWC**、**gralloc**以及与内核空间中的驱动程序通信的特定于供应商的GPU库。

# 深入分析图形HAL

在我们了解图形系统架构概述之后，我们将分析Gralloc模块，它是图形HAL。在AOSP源代码中，Gralloc HAL实现的骨架可以在以下文件夹中找到：

`$AOSP/hardware/libhardware/modules/gralloc`

这是一个通用实现，为开发者提供创建他们自己的Gralloc模块的参考。Gralloc将访问帧缓冲区和GPU，为上层提供服务。在本节中，我们将首先分析这个通用实现。在分析完这个通用的Gralloc HAL模块之后，我们将介绍Android模拟器的Gralloc HAL。

# 加载Gralloc模块

当应用程序开发者将图像绘制到屏幕上时，有两种方式可以实现。他们可以使用 Canvas 或 OpenGL。从 Android 4.0 开始，这两种方法默认都使用硬件加速。要使用硬件加速，我们需要使用 Open GL 库，最终 Gralloc 模块将作为图形系统初始化的一部分被加载。正如我们在[第 3 章](e0f861c2-5832-402f-89d3-cfc75785e759.xhtml)“发现内核、HAL 和虚拟硬件”中看到的，每个 HAL 模块都有一个引用 ID，该 ID 可以被 `hw_get_module` 函数用来将其加载到内存中。`hw_get_module` 函数定义在 `$AOSP/hardware/libhardware/hardware.c` 文件中：

[PRE0]

在 `hw_get_module` 中，它实际上调用另一个函数，`hw_get_module_by_class` 来完成工作：

[PRE1]

在前面的函数中，它试图在 `/system/lib/hw` 或 `/vendor/lib/hw` 中使用以下名称查找 Gralloc 模块的共享库：

[PRE2]

如果上述文件中的任何一个存在，它们将调用 `load` 函数来加载共享库。如果它们都不存在，将使用默认的共享库 `gralloc.default.so`。Gralloc 的硬件模块 ID 在 `gralloc.h` 文件中定义如下：

[PRE3]

`load` 函数将调用 `dlopen` 来加载库，并调用 `dlsym` 来获取数据结构 `hw_module_t` 的地址：

[PRE4]

在我们获取到数据结构 `hw_module_t` 的地址后，我们可以调用 Gralloc HAL 中定义的 `open` 方法来初始化帧缓冲区和 GPU。

如我们在[第 3 章](e0f861c2-5832-402f-89d3-cfc75785e759.xhtml)“发现内核、HAL 和虚拟硬件”中讨论的，硬件供应商需要实现以下三个 HAL 数据结构：

[PRE5]

在 HAL 共享库加载后，数据结构 `hw_module_t` 被用来发现 HAL 模块，正如我们在前面的代码片段中看到的。每个 HAL 模块都应该在数据结构 `hw_module_methods_t` 中实现一个 `open` 方法，该方法负责硬件的初始化。我们可以看到，在以下代码片段中，`gralloc_device_open` 函数被定义为 Gralloc 模块的 `open` 方法：

[PRE6]

在数据结构 `hw_module_methods_t` 中，`open` 方法被分配为一个静态函数，`gralloc_device_open`。`HAL_MODULE_INFO_SYM` 符号定义为 `struct private_module_t`。

你可能会注意到，当我们加载 Gralloc 模块时，实际上是将 `HAL_MODULE_INFO_SYM_AS_STR` 符号映射到 `hw_module_t`，而在默认的 Gralloc 模块中，数据结构 `hw_module_t` 是通过另外两个继承的数据结构 `private_module_t` 和 `gralloc_module_t` 实现的。让我们看看 `private_module_t`、`gralloc_module_t` 和 `hw_module_t` 之间的关系。

如果你在这部分的分析中感到有些困惑，我建议你在阅读这部分内容的同时查看源代码。如果你没有 AOSP 源代码，有一个非常好的 AOSP 代码交叉引用网站 [http://xref.opersys.com/](http://xref.opersys.com/)。

您可以访问此网站并搜索我们正在讨论的数据结构。

数据结构`private_module_t`在以下文件中定义：

`$AOSP/hardware/libhardware/modules/gralloc/gralloc_priv.h`

[PRE7]

如我们所见，第一个基类字段，或者说在C++术语中的成员变量，是数据结构`gralloc_module_t`。第二个成员变量`framebuffer`是`private_handle_t`数据类型的指针。它是一个指向framebuffer的句柄，我们将在后面探讨它。

成员变量`flags`用于指示系统是否支持双缓冲。如果支持，则`PAGE_FLIP`位设置为1；否则，设置为0。

`numBuffers`成员变量表示framebuffer中的缓冲区数量。它与可见分辨率和虚拟分辨率相关。例如，如果显示器的可见分辨率为800 x 600，则虚拟分辨率可以是1600 x 600。在这种情况下，framebuffer可以为显示器提供两个缓冲区，并且系统可以支持显示器的双缓冲。

`bufferMask`成员变量用于标记framebuffer设备中缓冲区的使用情况。如果我们假设framebuffer中有两个缓冲区，则`bufferMask`变量在二进制中可以有四个值：00、01、10和11。值00表示两个缓冲区都为空。值01表示第一个缓冲区正在使用，第二个缓冲区为空。值10表示第一个缓冲区为空，第二个缓冲区正在使用。值11表示两个缓冲区都在使用。

`lock`成员变量用于保护对`private_module_t`的访问。

`currentBuffer`成员变量用于跟踪用于渲染的当前缓冲区。

`info`和`finfo`成员变量是数据类型`fb_var_screeninfo`和`fb_fix_screeninfo`。它们用于存储显示设备的属性。`fb_var_screeninfo`中的属性是可编程的，而`fb_fix_screeninfo`中的属性是只读的。

`xdpi`和`ydpi`成员变量用于以水平和垂直方向描述像素密度。

`fps`成员变量是每秒显示的帧数。

`gralloc_module_t`数据结构在以下文件中定义：

`$AOSP/hardware/libhardware/include/hardware/gralloc.h`

[PRE8]

如我们所预期，`gralloc_module_t`中的第一个字段是来自前面代码片段的`hw_module_t`。这三个数据结构之间的关系类似于以下UML类图中的面向对象表示法：

![图片](img/image_10_002.png)

Gralloc数据结构之间的关系

这是C语言中模拟继承关系的方式。这样，我们可以将`private_module_t`的数据类型转换为`gralloc_module_t`或`hw_module_t`。

在`gralloc_module_t`中定义了一组成员函数。在本章中，我们将查看其中的四个。

`registerBuffer` 和 `unregisterBuffer` 成员函数用于注册或注销一个缓冲区。要注册一个缓冲区，我们需要将缓冲区映射到应用程序的进程空间。

`lock` 和 `unlock` 成员函数用于锁定或解锁一个缓冲区。缓冲区使用 `buffer_handle_t` 作为函数的参数进行描述。我们可以使用 `l`、`t`、`w` 和 `h` 参数来提供缓冲区的位置和大小。缓冲区锁定后，我们可以在 `vaddr` 输出参数中获取缓冲区的地址。使用完毕后，我们应该解锁缓冲区。

# 初始化 GPU

我们已经讨论了 Gralloc 模块的 HAL 数据结构 `hw_module_t` 和 `hw_module_methods_t`。最后一个，`hw_device_t`，在 Gralloc HAL 模块的 `open` 方法中初始化。现在我们可以查看 Gralloc 模块的 `open` 方法如下：

[PRE9]

如此可见，`gralloc_device_open` 函数可以根据输入参数 `name` 初始化两种设备，`GRALLOC_HARDWARE_GPU0` 和 `GRALLOC_HARDWARE_FB0`。

让我们先看看 GPU0 设备的初始化。`open` 方法的输出参数是 `hw_device_t` 数据结构的地址。调用应用程序获取 `hw_device_t` 实例后，可以使用硬件设备来完成它们的工作。在 Gralloc HAL 的 `open` 方法中，它首先为 `gralloc_context_t` 数据结构分配内存。之后，它填充其 `device` 成员变量并将输出参数赋值给 `dev->device.common` 成员变量的地址。正如我们所期望的，输出是 `hw_device_t` 实例的地址。让我们看看 `gralloc_context_t`、`alloc_device_t` 和 `hw_device_t` 之间的关系：

![](img/image_10_003.png)

如前图所示，`gralloc_context_t` 的第一个字段或成员变量是 `device`，其数据类型为 `alloc_device_t`：

[PRE10]

以下是对 `alloc_device_t` 数据结构的定义。它在 `gralloc.h` 文件中定义：

[PRE11]

我们可以看到 `alloc_device_t` 的第一个字段的数据类型是 `hw_device_t`。这是我们讨论 `private_module_t`、`gralloc_module_t` 和 `hw_module_t` 之间的关系时提到的 C 语言中模拟继承关系的技巧。

Gralloc 设备的 `alloc` 和 `free` 方法在 `gralloc.cpp` 文件中的 `gralloc_alloc` 和 `gralloc_free` 函数中实现。

# 初始化帧缓冲

如果我们使用 `GRALLOC_HARDWARE_FB0` 作为 `name` 值调用 Gralloc 模块的 `open` 方法，它将初始化帧缓冲设备。调用 `fb_device_open` 函数来打开帧缓冲设备：

[PRE12]

`fb_device_open` 函数在 `framebuffer.cpp` 文件中实现如下：

[PRE13]

在 `fb_device_open` 函数中，它为 `fb_context_t` 数据结构分配内存。之后，它填充数据结构中的字段。正如我们在 GPU0 初始化中讨论的那样，我们期望输出为 `hw_device_t` 数据结构的一个实例，以便调用者可以通过 `hw_device_t` HAL 数据结构使用帧缓冲区设备。这三个数据结构 `fb_context_t`、`framebuffer_device_t` 和 `hw_device_t` 之间有类似的继承关系，如下面的图所示：

![](img/image_10_004.png)

`fb_context_t`、`framebuffer_device_t` 和 `hw_device_t` 之间的关系

`fb_context_t` 数据结构将 `framebuffer_device_t` 作为第一个字段，如下所示：

[PRE14]

相应地，`framebuffer_device_t` 数据结构将 `hw_device_t` 作为第一个字段，因此 `fb_context_t` 可以用作 `framebuffer_device_t` 或 `hw_device_t`：

[PRE15]

对于 `framebuffer_device_t` 中剩余的字段，它们是：

+   `flags`: 用于描述帧缓冲区的某些属性。

+   `width` 和 `height`：帧缓冲区的像素尺寸。

+   `stride`：帧缓冲区的像素步长或每行的像素数。

+   `format`：帧缓冲区像素格式。可以是 `HAL_PIXEL_FORMAT_RGBX_8888`、`HAL_PIXEL_FORMAT_565` 等。

+   `xdpi` 和 `ydpi`：帧缓冲区显示面板的每英寸像素分辨率。

+   `fps`：显示面板的每秒帧数。

+   `minSwapInterval`：此帧缓冲区支持的最低交换间隔。

+   `maxSwapInterval`：此帧缓冲区支持的最高交换间隔。

+   `numFramebuffers`：支持的帧缓冲区数量。

在填充 `framebuffer_device_t` 的所有字段之前，`fb_device_open` 函数调用 `mapFrameBuffer` 函数以获取帧缓冲区信息。除了获取帧缓冲区信息外，此 `mapFrameBuffer` 函数还将帧缓冲区映射到当前进程空间，以便当前进程可以使用它。在 Android 中，Gralloc 模块由 SurfaceFlinger 拥有和管理。

让我们看看 `mapFrameBuffer` 函数：

[PRE16]

正如我们所见，`mapFrameBuffer` 首先获取一个互斥锁，然后调用另一个函数 `mapFrameBufferLocked` 来完成剩余的工作：

[PRE17]

在 `mapFrameBufferLocked` 函数中，它检查是否存在 `/dev/graphics/fb0` 或 `/dev/fb0` 设备节点。如果设备节点存在，它将尝试打开它并将文件描述符存储在 `fd` 变量中：

[PRE18]

接下来，它将使用 `ioctl` 命令获取帧缓冲区信息。有两个帧缓冲区数据结构，`fb_fix_screeninfo` 和 `fb_var_screeninfo`，可以用来与帧缓冲区通信。`fb_fix_screeninfo` 数据结构存储固定的帧缓冲区信息，而 `fb_var_screeninfo` 数据结构存储可编程的帧缓冲区信息：

[PRE19]

在获取到帧缓冲区信息后，它试图设置帧缓冲设备的虚拟分辨率。`xres`和`yres`字段用于存储帧缓冲设备的可见分辨率，而`xres_virtual`和`yres_virtual`字段用于存储帧缓冲设备的虚拟分辨率。

为了设置虚拟分辨率，它试图将虚拟垂直分辨率增加到`info.yres * NUM_BUFFERS`值。`NUM_BUFFERS`是用于帧缓冲设备中可以使用的缓冲区数量的宏。在我们的情况下，`NUM_BUFFERS`的值是`2`，因此我们可以使用双缓冲技术来显示。它使用`ioctl`命令`FBIOPUT_VSCREENINFO`来设置虚拟分辨率。如果它成功设置了虚拟分辨率，它将在`flags`中设置`PAGE_FLIP`位；否则，它将清除`PAGE_FLIP`位：

[PRE20]

在设置虚拟分辨率后，它将计算刷新率。要了解刷新率的计算，可以参考Linux内核源代码中的文档`Documentation/fb/framebuffer.txt`：

[PRE21]

接下来，它将计算水平和垂直的像素密度。它还将刷新率转换为每秒帧数，并将其存储到`fps`中。在它有了所有信息后，它将它们存储到数据结构`private_module_t`的字段中。

最后，它将帧缓冲区映射到进程地址空间：

[PRE22]

按虚拟分辨率计算的帧缓冲区大小是`finfo.line_length * info.yres_virtual`。`finfo.line_length`的值等于每行的字节数，而`info.yres_virtual`的值是每帧的行数。为了进行内存映射，我们必须使用`roundUpToPageSize`函数将大小四舍五入到页面边界。

在帧缓冲设备中可以使用的实际缓冲区数量是`info.yres_virtual`除以`info.yres`，并存储在`numBuffers`字段中。`bufferMask`字段被设置为0，这意味着所有缓冲区都是空的，可以用来使用。

它调用`mmap`系统调用来将帧缓冲区映射到当前进程地址空间。当前进程地址空间中帧缓冲区的起始地址是`vaddr`，由`mmap`系统调用返回。它被存储到`framebuffer->base`字段中，这样Gralloc模块就可以使用它来为应用程序分配缓冲区。

到目前为止，我们已经完成了对`mapFrameBuffer`函数的分析。这个函数负责在Gralloc HAL模块中初始化帧缓冲设备的大部分工作。

# 图形缓冲区的分配和释放

到目前为止，在本章中，我们已经讨论了加载Gralloc模块和Gralloc模块提供的`open`方法。现在让我们回顾上层加载、初始化和使用Gralloc模块时的要点：

+   例如，Gralloc 模块主要被 `SurfaceFlinger` 使用。`SurfaceFlinger` 使用 Gralloc；当它创建 `FramebufferNativeWindow` 的实例时，在 `FramebufferNativeWindow` 构造函数中，它将调用 `hw_get_module` 来获取 `hw_module_t` 的实例。

+   在 `hw_module_t` 数据结构中，它有一个名为 `methods` 的字段，其数据类型为 `hw_module_methods_t`。在 `hw_module_methods_t` 中，它有一个返回 `hw_device_t` 数据结构的 `open` 方法。

+   通过 `hw_device_t`，`SurfaceFlinger` 可以使用 `hw_device_t` 内部的 `alloc` 和 `free` 方法来分配或释放图形缓冲区。

让我们在本节中看看 Gralloc 模块如何分配和释放图形缓冲区。我们首先查看 `gralloc_alloc` 的源代码：

[PRE23]

如前述代码片段所示，`alloc` 方法是在 `gralloc_alloc` 函数中实现的。`gralloc_alloc` 有以下参数：

+   `dev`: 它具有从 `hw_device_t` 继承的 `alloc_device` 数据类型。

+   `w` : 它是图形缓冲区的宽度。

+   `h`: 它是图形缓冲区的高度。

+   `format` : 它定义了像素的颜色格式。例如，格式可以是 `HAL_PIXEL_FORMAT_RGBA_8888`、`HAL_PIXEL_FORMAT_RGB_888`、`HAL_PIXEL_FORMAT_RGB_565` 等。

+   `usage` : 它定义了图形缓冲区的用途。例如，如果设置了 `GRALLOC_USAGE_HW_FB` 位，缓冲区将从帧缓冲区分配。

+   `pHandle` : 它具有 `buffer_handle_t` 数据类型。我们将讨论这个数据结构的详细信息。它用于存储分配的缓冲区。

+   `pStride` : 每行的像素数。

在 `gralloc_alloc` 中，它检查像素的格式以决定像素的大小。它可以是以32位、24位、16位等。像素的大小存储在 `bpp` 变量中。`bpr` 变量是每行的字节数，它是通过 `w` 乘以 `bpp` 计算得出的。`bpr` 变量需要对齐到四个字节的边界以进行内存分配。缓冲区的大小可以通过 `h` 乘以 `bpr` 来计算。

在计算缓冲区大小后，它将根据 `GRALLOC_USAGE_HW_FB` 位调用 `gralloc_alloc_framebuffer` 或 `gralloc_alloc_buffer` 函数。

由 `gralloc_alloc` 分配的图形缓冲区存储在 `buffer_handle_t` 数据类型中。`buffer_handle_t` 被定义为 `native_handle` 的指针。`native_handle` 被用作 `private_handle_t` 的父类。`private_handle_t` 是实际用于管理图形缓冲区的数据类型，它是一个硬件相关的数据结构。

![](img/image_10_005.png)

private_handle_t 和 native_handle 之间的关系

上述图表显示了 `private_handle_t` 和 `native_handle` 之间的关系。以下是对 `native_handle` 的定义：

[PRE24]

`version` 字段被设置为 `native_handle` 的大小。`numFds` 和 `numInts` 字段描述了 `data` 数组中的文件描述符和整数的数量。`data` 数组用于存储特定于硬件的信息，我们可以在以下 `private_handle_t` 的定义中看到：

[PRE25]

`fd` 成员变量是一个文件描述符，用于描述帧缓冲区或共享内存区域。`magic` 成员变量存储为在 `sMagic` 静态变量中定义的魔术数字。`flags` 成员变量用于描述图形缓冲区的类型。例如，如果它等于 `PRIV_FLAGS_FRAMEBUFFER`，则此缓冲区是从帧缓冲区分配的。`size` 成员变量是图形缓冲区的大小。`offset` 成员变量是从内存起始地址的偏移量。`base` 成员变量是为缓冲区分配的地址。`pid` 成员变量是图形缓冲区创建者的进程 ID。

构造函数填充 `native_handle` 的成员变量。`validate` 成员函数用于验证图形缓冲区是否是 `private_handle_t` 的实例。

正如我们之前提到的，我们正在分析的 Gralloc 模块是 AOSP 中的默认实现，构建为 `galloc.default.so`。在这个实现中，GPU 不被使用，缓冲区将分配在帧缓冲区或共享内存中。尽管这不是性能的理想情况，但它具有最少的硬件依赖性，这对于理解更复杂的 Gralloc 模块实现是一个很好的参考。

# 从帧缓冲区分配

从 `gralloc_alloc` 函数中我们可以看到，当 `usage` 位设置为 `GRALLOC_USAGE_HW_FB` 时，调用 `gralloc_alloc_framebuffer` 函数。`gralloc_alloc_framebuffer` 函数将从帧缓冲区设备分配缓冲区：

[PRE26]

`gralloc_alloc_framebuffer` 首先获取一个互斥锁，并调用另一个函数，`gralloc_alloc_framebuffer_locked`。在锁定版本中，它调用之前分析的 `mapFrameBufferLocked` 函数，以获取帧缓冲区信息并将其映射到当前进程地址空间。

它将检查帧缓冲区设备是否支持双缓冲。如果它支持双缓冲，它将创建一个新的 `private_handle_t` 实例，并填充此实例中的信息，然后返回给调用者。如果缓冲区是从帧缓冲区设备分配的，它将标记 `private_handle_t` 的 `flags` 成员变量为 `PRIV_FLAGS_FRAMEBUFFER`。它还将设置帧缓冲区的 `usage` 状态在 `bufferMask` 中，这是 `private_module_t` 的成员变量。

如果它不支持双缓冲，它将调用 `gralloc_alloc_buffer` 从系统内存分配缓冲区并返回给调用者。

# 从系统内存分配

当 `usage` 位未设置为 `GRALLOC_USAGE_HW_FB` 或系统不支持双缓冲时，我们必须使用 `gralloc_alloc_buffer` 从系统内存分配缓冲区。让我们看看 `gralloc_alloc_buffer` 的实现：

[PRE27]

在 `gralloc_alloc_buffer` 中，它首先将缓冲区大小向上舍入到页面大小。然后使用 `ashmem_create_region` 创建一个匿名共享内存区域。它创建一个新的 `private_handle_t` 实例来表示这个共享内存区域。

这个共享内存区域被描述为一个文件描述符。要使用它，我们需要将其映射到当前进程的地址空间。这是通过 `mapBuffer` 函数完成的：

[PRE28]

`mapBuffer` 调用另一个函数 `gralloc_map` 来进行内存映射：

[PRE29]

在 `grallo_map` 中，如果 `private_handle_t` 中的文件描述符是帧缓冲设备，我们不需要再次进行映射，因为帧缓冲已经在 `fb_device_open` 中初始化并映射到 `SurfaceFlinger` 地址空间，正如我们之前分析的。

如果是一个共享内存区域，需要使用 `mmap` 系统函数将其映射到当前进程的地址空间。

# 释放图形缓冲区

正如我们之前提到的，Gralloc 模块可以用来分配和释放图形缓冲区。现在我们已经学会了如何从帧缓冲设备或系统内存分配缓冲区，让我们看看如何释放图形缓冲区。

要释放图形缓冲区，使用 `gralloc_free` 函数：

[PRE30]

要释放一个图形缓冲区，使用 `buffer_handle_t` 描述缓冲区。`gralloc_free` 将首先使用 `private_handle_t::validate` 静态函数验证缓冲区。

`handle` 参数可以转换为 `private_handle_t` 的指针，正如我们从之前关于 `private_handle_t` 和 `native_handle` 的讨论中回忆的那样。如果 `hnd` 的 `flags` 字段是 `PRIV_FLAGS_FRAMEBUFFER`，这意味着缓冲区是从帧缓冲设备分配的。它将更新 `bufferMask` 以从帧缓冲中释放它。

如果缓冲区是从系统内存分配的，它将调用 `terminateBuffer` 函数来释放内存：

[PRE31]

`terminateBuffer` 函数调用另一个函数 `gralloc_unmap` 来释放内存：

[PRE32]

在 `gralloc_unmap` 中，它再次检查这个缓冲区不是来自帧缓冲，并调用 `munmap` 系统函数来释放它。

# 渲染帧缓冲

正如我们在本章前面讨论的那样，Gralloc 模块可以支持两种类型的设备：Gralloc 设备和帧缓冲设备。在 Gralloc 设备的 `open` 方法中，它创建一个名为 `GRALLOC_HARDWARE_GPU0` 的设备，并支持两种方法，`alloc` 和 `free`，正如我们可以在下面的代码片段中看到的那样。我们已经在本章前面详细讨论了这两种方法：

[PRE33]

在帧缓冲设备 `open` 方法中，它创建一个名为 `GRALLOC_HARDWARE_FB0` 的设备，并支持四种方法 `close`、`setSwapInterval`、`post` 和 `setUpdateRect`：

[PRE34]

您可以参考 AOSP 源代码或以下 URL 获取有关这些方法实现的信息：

[http://xref.opersys.com/android-7.0.0_r1/xref/hardware/libhardware/modules/gralloc/framebuffer.cpp](http://xref.opersys.com/android-7.0.0_r1/xref/hardware/libhardware/modules/gralloc/framebuffer.cpp)

让我们看看 `post` 方法，它在 `fb_post` 中实现：

[PRE35]

在应用程序准备完图形缓冲区后，它需要将缓冲区发布到显示，以便用户可以在屏幕上看到它。这个 `fb_post` 函数用于将图形缓冲区显示到屏幕上。它接受两个参数，`dev` 和 `buffer`。`dev` 参数是 `framebuffer_device_t` 数据结构实例的指针，这之前已经讨论过（参考 `fb_context_t` 和 `framebuffer_device_t` 之间关系的图）。根据之前的讨论，`dev` 可以转换为 `ctx`，它是 `fb_context_t` 指针。

在我们获得设备实例后，我们可以按照以下方式从其中获取 Gralloc 模块的实例：

[PRE36]

另一个参数是 `buffer`，它具有 `buffer_handle_t` 数据类型。它包括要发布的缓冲区。正如我们之前讨论的，它可以转换为 `private_handle_t` 的指针，并存储在 `hnd` 变量中。这个缓冲区可以是系统内存中的图形缓冲区，也可以是帧缓冲区的一部分。根据 `hnd->flags` 成员变量的值，我们可以确定它是哪种类型的缓冲区。

如果它是帧缓冲区的一部分，我们需要将其激活为显示的缓冲区。这可以通过使用帧缓冲区的 `ioctl` 函数来完成。要调用 `ioctl` 函数，我们需要一个 `fb_var_screeninfo` 数据结构，这可以在 `m->info` 中找到。为了在双缓冲中交换缓冲区，我们只需设置垂直偏移并按照以下方式激活它：

[PRE37]

如果它是在系统内存中分配的缓冲区，我们需要将其复制到帧缓冲区。在这种情况下，它首先尝试锁定图形缓冲区和帧缓冲区，然后使用 `memcpy` 复制图形缓冲区：

[PRE38]

# Android 模拟器的图形 HAL

在我们分析了默认 Gralloc 模块实现之后，我们想简要地看看另一个 Gralloc 模块实现，以便我们可以比较在不同图形硬件上应该如何实现 Gralloc 模块。

在本节中我们将要分析的 Gralloc 模块是 Android 模拟器使用的 Gralloc 模块。默认的 Gralloc 模块 `gralloc.default.so` 只使用帧缓冲区设备，并且不使用 GPU。如果使用默认的 Gralloc 模块，OpenGL 支持必须在软件层中实现。目前的情况是 VirtualBox，因为 VirtualBox 主机端没有 Mesa/DRM 兼容的 OpenGL 实现。这并不意味着 VirtualBox 不支持 OpenGL。它确实支持 OpenGL 和 3D 硬件加速，但实现并不符合开源 Mesa/DRM 架构。

如果您对关于 VirtualBox 上 OpenGL 支持的此主题感兴趣，您可以在 Android-x86 讨论组中阅读以下线程：

[https://groups.google.com/forum/?hl=en#!starred/android-x86/gZYx6oWx4LI](https://groups.google.com/forum/?hl=en#!starred/android-x86/gZYx6oWx4LI)

# 硬件 GLES 模拟概述

Android 模拟器上的 3D 图形支持以不同的方式实现，如下所示：

+   `host`：这是默认模式。这也称为硬件 GLES 模拟。它使用特定的翻译库将客户机 EGL/GLES 命令转换为宿主 GL 命令。这需要在宿主机器上安装有效的 OpenGL 驱动程序。

+   `swiftshader`：这是一个用于在 CPU 上进行高性能图形渲染的软件库。它利用现代 CPU 上的 SIMD 来执行图形渲染。

+   `mesa`：这已被弃用。这是一个使用 Mesa3D 库的软件库。它的速度比 swiftshader 模式慢，并且比 `host` 模式慢得多。

+   `guest`：这是在客户机侧的纯软件实现。

在模拟器中选择图形模式，您可以通过命令行使用 `-gpu` 选项指定，或者在 `config.ini` 配置文件中定义，如下所示：

[PRE39]

我们将在此处查看 `host` 模式下的 Gralloc 模块实现。在硬件 GLES 模拟中，实现了几个宿主“翻译”库：EGL、GLES 1.1 和 Khronos 定义的 GLES 2.0 ABIs（应用程序二进制接口）。这些库将相应的函数调用转换为调用适当的宿主 OpenGL API。

在模拟的客机系统中实现了用于 EGL、GLES 1.1 和 GLES 2.0 ABIs 的相同系统库集合。它们收集 EGL/GLES 函数调用的序列，并将它们转换为发送到模拟器程序的定制线协议流，该流通过称为“QEMU 管道”的高速通信通道发送。管道使用定制的内核驱动程序实现，它可以提供主机和客机系统之间非常快速的通信通道。我在[第 3 章](e0f861c2-5832-402f-89d3-cfc75785e759.xhtml)“发现内核、HAL 和虚拟硬件”中简要介绍了 QEMU 管道，您可以参考它以获取更多信息。

![](img/image_10_006.png)

硬件 GLES 模拟

您可以在模拟器源代码的 `$AOSP/external/qemu/distrib/android-emugl/DESIGN` 中找到前面的图表。

在本章中，不会使用 manifest 文件下载模拟器源代码。您可以参考以下 URL：

[https://android.googlesource.com/platform/external/qemu/+/master/distrib/android-emugl/DESIGN](https://android.googlesource.com/platform/external/qemu/+/master/distrib/android-emugl/DESIGN)

或者，您可以使用以下命令获取整个仓库：

[PRE40]

前面的图示显示了GLES仿真的主机（模拟器）端和客户机端的组件。我们可以将主机端实现视为GPU，**QEMU PIPE**是GPU和CPU之间的连接。需要访问GPU进行3D图形加速的两个东西是Gralloc模块和供应商库。这里的供应商库是指Android模拟器的硬件GLES仿真库。Gralloc模块是我们本节想要探索的模块。

GLES硬件仿真Gralloc模块与我们本章中讨论的默认Gralloc模块非常相似。它需要实现以下三个HAL数据结构：

[PRE41]

对于第一个数据结构，`hw_module_t`，两个Gralloc模块都有自己的实现，称为`private_module_t`，它继承自`hw_module_t`，但定义不同，如下面的代码片段所示。

默认Gralloc模块中的`private_module_t`如下：

[PRE42]

GLES仿真Gralloc模块中的`private_module_t`如下：

[PRE43]

对于`hw_device_t`数据结构实现，我们可以从以下表格中获取详细信息。我们可以使用`hw_module_methods_t`数据结构中的`open`方法创建两种设备，`GPU0`和`FB0`。在这两种实现中，都使用了从`hw_device_t`继承的数据结构：

| **Gralloc模块中的** **hw_device_t** | **GPU0** | **FB0** |
| --- | --- | --- |
| Android 模拟器 | `gralloc_device_t` | `fb_device_t` |
| 默认Gralloc | `gralloc_context_t` | `fb_context_t` |

我们在*初始化GPU*部分分析了`gralloc_context_t`和`fb_context_t`。我们可以在以下GLES仿真实现中查看`gralloc_device_t`和`fb_device_t`的定义：

[PRE44]

# 初始化GLES仿真的GPU0和FB0

正如我们所知，设备初始化是在`hw_module_methods_t`数据结构中定义的`open`方法中完成的。让我们看看GLES仿真的`open`方法的实现。它是在`gralloc_device_open`函数中实现的，如下面的代码片段所示：

[PRE45]

前面的代码片段是`GPU0`初始化的一部分。在它为`GPU0`或`FB0`创建设备之前，它将调用一个`fallback_init`函数来检查系统设置中的硬件仿真。在`fallback_init`中，它将检查一个`ro.kernel.qemu.gles`系统属性。如果这个属性设置为0，则GPU仿真将被禁用。将使用默认的Gralloc模块。在这种情况下，默认Gralloc模块中定义的`open`方法，即`sFallback`，将被调用。

对于`GPU0`初始化，它将检查设备名称是否等于`GRALLOC_HARDWARE_GPU0`。如果是`GPU0`，它将首先获取主机连接。主机连接是我们之前讨论的主机和客户机系统之间的QEMU pipe链接。

之后，它初始化`GPU0`设备，就像我们为默认Gralloc模块讨论的初始化过程一样。

接下来，让我们看一下以下 `FB0` 的初始化：

[PRE46]

在 `FB0` 初始化中，它尝试使用 `DEFINE_AND_VALIDATE_HOST_CONNECTION` 宏获取主机连接和一个 `rcEnc` 指针，这是一个 `renderControl_encoder_context_t` 数据结构的实例。有了 `rcEnc`，它可以从主机连接中获取帧缓冲区属性（`width`、`height`、`xdpi`、`ydpi`、`fps`、`min_si` 和 `max_si`）。之后，它创建了一个 `fb_device_t` 数据结构的实例，并在该实例中填写了帧缓冲区属性。

# GPU0 设备实现

正如我们在默认的 Gralloc 模块中所做的那样，我们将分析 `GPU0` 设备中的 `alloc` 和 `free` 方法。`alloc` 方法实现在 `gralloc_alloc` 函数中。`gralloc_alloc` 函数比默认 Gralloc 模块中的函数要长得多，但它基本上做了三件事：

+   检查 `usage` 参数并决定像素格式以确定像素的大小。

+   根据由 `usage` 参数提供的信息，`w`、`h`、`format` 和 `usage` 创建一个共享内存区域并在主机端（GPU）分配缓冲区。

+   在 Gralloc 设备数据结构 `grdev` 中存储共享内存区域和主机端（GPU）缓冲区信息。

现在让我们看一下 `gralloc_alloc` 的代码：

[PRE47]

在 `gralloc_alloc` 的前面代码中，它首先创建了一个 `gralloc_device_t` 数据结构的实例。之后，它检查 `usage` 和 `format` 参数以确定像素的大小以及相应的 GLES 颜色格式和像素类型，并将它们存储在 `bpp`、`glFormat` 和 `glType` 变量中。有了必要的信息，它可以计算出需要为图形缓冲区分配的共享内存的大小，并将其存储在 `ashmem_size` 变量中：

[PRE48]

至于共享内存大小 `ashmem_size`，它使用 `ashmem_create_region` 函数分配一个共享内存区域，并获取共享内存区域作为一个 `fd` 文件描述符。为了存储共享内存区域和 GPU 缓冲区（我们将讨论的主机端缓冲区），它创建了一个 `cb_handle_t` 数据结构的实例。如果我们回想一下，我们在默认的 Gralloc 模块中使用了 `private_handle_t` 数据结构来表示分配的图形缓冲区。在这里，`cb_handle_t` 是 `private_handle_t` 的等价物：

[PRE49]

因为 `cb_handle_t` 是一个大型数据结构，所以在前面的代码片段中我们没有展示 `cb_handle_t` 的所有成员函数。从成员变量中我们可以看出，它们与 `private_handle_t` 类似。您可以参考 `private_handle_t` 的部分来了解大多数成员变量的解释。请注意最后一个成员变量 `hostHandle`，它用于存储在 GPU 上分配的缓冲区（在 GLES 模拟中的主机端）。如果您对主机端 GLES 模拟感兴趣，可以参考 QEMU 源代码。

让我们看一下 `gralloc_alloc` 的最后一部分代码：

[PRE50]

在缓冲区在GPU上分配并且从系统内存中获取共享内存区域后，它们被存储在`grdev`变量中，并添加到`gralloc_device_t`中的双链表节点。

对于`gralloc_device_t`的`free`方法，它比`alloc`简单得多。为了节省空间，这里不会列出源代码。`free`方法是在`gralloc_free`函数中实现的。它所做的是：

1.  验证`buffer_handle_t`指向有效的`cb_handle_t`数据结构。

1.  在主机端（GPU）释放缓冲区，调用`rcCloseColorBuffer`函数。

1.  在共享内存区域取消映射缓冲区并释放共享内存。

1.  从链表中移除节点。

1.  释放`cb_handle_t`数据结构使用的内存。

# FB0设备实现

对于`FB0`设备的实现，我们将像分析默认的Gralloc模块那样查看`post`方法。这是在`fb_post`函数中实现的，我们可以如下查看其实现：

[PRE51]

它所做的是非常简单的；它增加缓冲区的post计数，并调用`rcFBpost`函数来更新GPU中的缓冲区。

我们现在已经完成了对Android模拟器图形HAL的分析。希望对通用图形HAL和Android模拟器图形HAL的分析能帮助你理解你系统中的图形HAL。

# 摘要

在本章中，我们探讨了并回顾了两个Gralloc HAL模块的实现，即默认的Gralloc模块和Android模拟器使用的模块。默认的Gralloc HAL仅使用帧缓冲设备，而OpenGLES支持使用软件实现。Android模拟器使用的是主机端的硬件仿真。其实现与基于GPU的Gralloc模块类似。

由于图形系统非常复杂，我们将在下一章查看VirtualBox特定实现时，继续更深入地探讨这个主题。我们将解释Gralloc HAL和OpenGL ES库的加载过程。我们将为Android构建一个VirtualBox扩展包，以便我们可以利用VirtualBox提供的功能。
