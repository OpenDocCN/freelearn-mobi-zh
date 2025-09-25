# 启用 ARM 翻译器和介绍原生桥接

我们在上一章中创建了一个新的 x86emu 设备。这是进一步定制和扩展的基础。正如我们所知，如果应用程序包含原生库，则无法在不同的处理器架构上运行。大多数 Android 应用程序是为 ARM 平台构建的。我们通常在 Intel x86 平台上运行这些带有 ARM 原生库的应用程序时遇到问题。然而，从 Android 5 及以上版本开始，Google 为此情况提供了一个名为 **原生桥接** 的解决方案。在本章中，我们将深入研究原生桥接和 Intel Houdini 的实现，以扩展 x86emu 以支持 ARM 原生应用程序。在本章中，我们将涵盖以下主题：

+   介绍原生桥接

+   将 Houdini 库集成到 x86emu 设备中

+   使用 Houdini 集成构建和测试图像

# 介绍原生桥接

在 Android 架构中，原生桥接作为 Android Runtime（**ART**）的一部分实现。它用于支持在不同的处理器架构上运行原生库，以便具有原生库的应用程序可以在更广泛的设备上运行。名为 Houdini 的 Intel ARM 翻译器是原生桥接的一个用例。在 ART 中，原生桥接的初始化有两个阶段：

1.  在第一阶段，原生桥接作为 ART 初始化过程的一部分加载到系统中。这对于所有应用程序都是常见的。

1.  在第二阶段，当启动具有原生库的应用程序时，它将从 Zygote 中 fork。此时，原生桥接将被初始化并准备好供应用程序使用。这是一个针对单个应用程序的特定过程。例如，如果没有使用原生库，则不会为此应用程序初始化原生桥接。

**Zygote** Android 在其核心有一个名为 Zygote 的进程，它在 init 时启动。此进程是一个“预热”进程，这意味着它是一个已经初始化并且所有核心库都已链接的进程。当你启动一个应用程序时，Zygote 将被 fork 以创建新的进程。真正的加速是通过*不*复制共享库来实现的。只有当新进程尝试修改它时，此内存才会被复制。这意味着所有核心库都可以存在于一个地方，因为它们是只读的。

当应用程序开始从不同的处理器架构加载原生库时，原生桥接将帮助解决此库的加载。例如，当我们加载 ARM 库在 Intel 的 x86 架构上时，原生桥接将使用 Houdini 在 Intel x86 环境中加载和执行此 ARM 库。

![](img/image_05_001.png)

原生桥接在 Android 架构中

Native Bridge作为Android系统库的一部分构建为一个`libnativebridge.so`共享库，如图所示。实现可以在`$AOSP/system/core/libnativebridge`中找到。在Native Bridge实现中，它在`native_bridge.cc`中定义了五个状态，如下所示：

[PRE0]

当Android系统刚刚启动时，Native Bridge处于`kNotSetup`状态。在ART的初始化过程中，它将被加载到系统中，阶段变为`kOpened`。

这两个状态是Native Bridge初始化的第一阶段。当用户启动带有本地库的应用程序时，系统将从Zygote中fork一个新的进程。此时，系统将为Native Bridge做一些预初始化工作，我们将在本章后面看到这一点。此时状态变为`kPreInitialized`。从Zygote fork进程后，Native Bridge作为进程创建的一部分被初始化，其状态变为`kInitialized`。`kClosed`状态通常不使用，除非出现错误并且关闭Native Bridge。这三个状态属于Native Bridge初始化的第二阶段。

在了解了Android系统架构中关于Native Bridge的概述之后，我们将不得不深入了解运行时使用的Native Bridge的每个阶段的细节。

# 将Native Bridge作为ART初始化的一部分设置起来

首先，让我们看看Native Bridge如何在系统中加载。Native Bridge作为ART初始化的一部分被加载。如图所示，它包括从**ART**到**Native Bridge**实现的函数调用。在这个阶段结束时，**Native Bridge**的状态将被设置为`kOpened`。

![图片](img/image_05_002.png)

加载Native Bridge

当系统初始化ART时，会调用`Runtime::Init`函数。在`Runtime::Init`内部，会调用`LoadNativeBridge`函数来加载Native Bridge共享库。我们可以在下面的代码片段中看到这一点：

[PRE1]

这个`LoadNativeBridge`函数是ART的一部分，它在`native_bridge_art_interface.cc`文件中实现，如图所示。这个函数简单地调用在`android`命名空间中的另一个函数`android::LoadNativeBridge`，而它本身在`art`命名空间中。`android`命名空间中的函数是Native Bridge实现的一部分，如图所示，我们将在本章后面看到更多关于这一点。我们可以在下面的代码片段中看到`LoadNativeBridge`的实现：

[PRE2]

`android`命名空间中的`android::LoadNativeBridge`函数与`art`命名空间中的`art:LoadNativeBridge`函数相比，有一个额外的`native_bridge_art_callbacks`参数。这个参数的类型是`struct NativeBridgeRuntimeCallbacks`的指针，它在`native_bridge.h`中定义。在`struct NativeBridgeRuntimeCallbacks`中，它定义了以下三个回调方法：

[PRE3]

这些作为ART一部分的三个回调函数在`native_bridge_art_interface.cc`文件中实现。这些回调函数为原生方法调用JNI原生函数提供了一种方式。我们将在稍后看到这种回调数据结构是如何传递给实际的Native Bridge实现的。在我们的例子中，实际的实现是Houdini库。

`native_bridge.h`文件定义了另一个回调函数数据结构，`NativeBridgeCallbacks`，它用作其实际实现的Native Bridge接口。在我们的例子中，这个实现是Houdini库。Houdini库需要实现这些回调函数并将指针传递给Native Bridge，以便ART可以使用它们。以下图显示了这两组回调函数之间的关系：

![图片](img/image_05_003.png)

ART、Native Bridge和Houdini

在前面的图中，我们可以看到**ART**调用**Native Bridge**函数来加载和初始化**Native Bridge**模块。**Native Bridge**模块调用由**Houdini**注册的回调函数来处理所有ARM原生二进制翻译。在**Native Bridge**初始化期间，**NativeBridgeRuntimeCallbacks**传递给**Houdini**库，以便**Houdini**库中的方法可以调用JNI原生函数。

现在我们来看看`android::LoadNativeBridge`的实现：

[PRE4]

从前面的代码片段中我们可以看到，`android::LoadNativeBridge`首先检查状态。它应该处于`kNotSetup`状态。否则，它将报告错误并返回。

为了方便起见，在接下来的几段中，我们将把Android命名空间中的函数称为`LoadNativeBridge`，而不是`android::LoadNativeBridge`。将要讨论的文件可以在以下位置找到：

`$AOSP/art/runtime/runtime.c`

`$AOSP/art/runtime/native_bridge_art_interface.c`

`$AOSP/system/core/libnativebridge/native_bridge.cc`

之后，它将检查第一个参数是否为`NULL`以及文件名是否可以使用。如果一切正常，它将通过`dlopen`使用文件名`nb_library_filename`打开库。

那么`nb_library_filename`文件名的内容是什么呢？从`Runtime::Init`函数中我们可以看到，`LoadNativeBridge`的第一个参数使用`Opt::NativeBridge`属性初始化：

[PRE5]

这个属性是从默认属性`ro.dalvik.vm.native.bridge`初始化的，该属性定义在Android系统的`default.prop`文件中。这是在`AndroidRuntime::startVm`函数中完成的，如以下片段所示。此函数定义在`$AOSP/frameworks/base/core/jni/AndroidRuntime.cpp`文件中：

[PRE6]

当启用原生桥接时，`ro.dalvik.vm.native.bridge` 属性通常包含一个共享库文件名。在我们的例子中，对于英特尔设备是 `libhoudini.so`，对于 Android-x86 是 `libnb.so`。如果禁用原生桥接，其值是 0。一旦库加载成功，它将使用 `kNativeBridgeInterfaceSymbol` 符号来获取内存位置，并将位置转换为 `NativeBridgeCallbacks` 的指针。这意味着 Houdini 库提供了一个 `NativeBridgeCallbacks` 的实现。让我们看看 `NativeBridgeCallbacks` 中都有什么：

[PRE7]

从前面的代码片段中，我们可以看到 `NativeBridgeCallbacks` 包含一个变量和七个回调函数：

+   `version`：这是接口的版本号。到目前为止，有两个版本。版本 1 定义了前五个回调函数，版本 2 增加了另外两个新函数，我们很快就会看到。

+   `initialize`：这个函数初始化 Native Bridge 的一个实例。Native Bridge 的内部实现必须确保多线程安全，并且 Native Bridge 只初始化一次。

+   `loadLibrary`：这个函数加载 Native Bridge 支持的共享库。

+   `getTrampoline`：这个函数获取指定原生方法的 Native Bridge 跳转函数。

+   `isSupported`：这个函数检查 Native Bridge 的实例是否有效，以及它是否为 Native Bridge 支持的 ABI。

在版本 2 中，增加了以下两个函数：

+   `isCompatibleWith`：这个函数检查桥接是否与给定的库版本兼容。桥接可能决定不向前或向后兼容，此时 `libnativebridge` 将停止使用它。

+   `getSignalHandler`：一个回调函数，用于检索指定信号的 Native Bridge 信号处理器。运行时会确保在运行时自己的处理器之后、所有链式处理器之前调用信号处理器。原生桥接不应尝试自行安装处理器，因为这可能导致循环。

现在我们已经完成了 Native Bridge 初始化的第一阶段。从前面的列表中我们可以看到，Native Bridge 在 ART 启动时加载。在这个阶段，初始化不是进程特定的。库名在 `ro.dalvik.vm.native.bridge` 属性中定义。在我们的例子中，ART 通过在 `libnativebridge.so` 中定义的 `LoadNativeBridge` 函数加载 `libhoudini.so` 库。一旦 Native Bridge 加载成功，状态设置为 `kOpened`。

# 预初始化 Native Bridge

在 Native Bridge 初始化的第二阶段，它变得与进程相关。Android 应用可以通过 Native Bridge 在不同于当前设备处理器架构的情况下加载本地库。其他两个状态，`kPreInitialized` 和 `kInitialized`，与 Android 应用程序的创建有关，因为我们知道在 Android 中所有应用程序都是从 Zygote 分叉出来的。让我们首先看看 Native Bridge 的预初始化，如下面的图所示：

![](img/image_05_004.png)

Native Bridge 的预初始化

在创建应用程序的过程中，会调用 `ForkAndSpecializeCommon` 函数。Native Bridge 的预初始化就在这个函数中完成。这个函数定义在 `$AOSP/frameworks/base/core/jni/com_android_internal_os_Zygote.cpp` 文件中：

[PRE8]

在这个 `ForkAndSpecializeCommon` 函数中，它检查当前进程是否不是 SystemServer 进程，以及 Native Bridge 是否准备好使用。之后，它调用 `NeedsNativeBridge` 函数来检查当前进程是否需要使用 Native Bridge：

[PRE9]

`NeedsNativeBridge` 函数将 `instruction_set` 与当前 Android 平台的指令集进行比较。如果这两个指令集不同，那么我们需要使用 Native Bridge；否则，我们不需要。`NeedsNativeBridge` 函数在 `native_bridge.cc` 文件中实现。

如果应用程序需要 Native Bridge，那么 `PreInitializeNativeBridge` 函数将被调用，该函数也实现于 `native_bridge.cc`，并带有两个参数，`app_data_dir_in` 和 `instruction_set`：

[PRE10]

从前面的代码片段中，我们可以看到它会检查状态是否为 `kOpened`。然后 `PreInitializeNativeBridge` 将执行两件事。首先，它使用第一个参数 `app_data_dir_in` 在应用程序的 `data` 文件夹中为 Native Bridge 创建一个代码缓存目录。接下来，它使用第二个参数 `instruction_set` 来查找 `/system/lib/<isa>/cpuinfo` 路径，并将其绑定挂载到 `/proc/cpuinfo`。如果设备上可用 Houdini，你可以在 `system` 文件夹中找到 `/system/lib/arm/cpuinfo` 文件。一旦完成前面的两项任务，Native Bridge 的状态将被设置为 `kPreInitialized`。

# 初始化 Native Bridge

状态变为 `kPreInitialized` 后，新的 Android 应用程序将在 `ForkAndSpecializeCommon` 函数中继续创建。在这个函数的末尾，它通过一个全局变量 `gCallPostForkChildHooks` 调用一个已注册的 `callPostForkChildHooks` 函数。调用栈最终会进入 `ZygoteHooks_nativePostForkChild` 函数，这是 `postForkChild` Java 方法的 JNI 实现。`postForkChild` 函数在每次分叉后由 Zygote 在子进程中调用。以下表格是对调用栈的总结：

| **函数** | **类** | **语言** |
| --- | --- | --- |
| `ForkAndSpecializeCommon` |  | C++ |
| `gCallPostForkChildHooks` |  | C++ |
| `callPostForkChildHooks` | Zygote | Java |
| `postForkChild` | ZygoteHooks | Java |
| `ZygoteHooks_nativePostForkChild` | JNI (postForkChild) | C++ |

`ZygoteHooks_nativePostForkChild` 函数在 `$AOSP/ art/runtime/native/dalvik_system_ZygotHooks.cc` 文件中实现。`DidForkFromZygote` 函数在 `$AOSP/art/runtime/runtime.cc` 文件中实现。

以下图表是涉及 Native Bridge 初始化第二阶段函数的总结。请注意，我们现在处于子进程中。我们可以看到，**ART** 中的 **Runtime::DidForkFromZygote** 函数将调用以下 Native Bridge 接口函数：**InitializeNativeBridge** 和 **SetupEnvironment**。Native Bridge 接口函数最终将调用 Houdini 库中注册的回调函数。

![](img/image_05_005.png)

Native Bridge 的初始化

让我们看看 `postForkChild` 的 JNI 实现：

[PRE11]

在这里，它再次检查指令集以决定我们是否需要为应用程序使用 Native Bridge。然后它调用 `Runtime::DidForkFromZygote` 函数以在新进程中初始化 Native Bridge：

[PRE12]

正如我们所见，`Runtime::DidForkFromZygote` 根据操作调用 `InitializeNativeBridge`。现在让我们深入了解 `InitializeNativeBridge` 函数，该函数在 `native_bridge.cc` 中实现：

[PRE13]

在 `InitializeNativeBridge` 函数中，它首先创建代码缓存的文件夹。然后，它调用由我们案例中的 Houdini 库实现的 `initialize` 函数。

在英特尔设备上，共享库是 `libhoudini.so`。如果您在英特尔设备上运行 Android-x86，共享库是 `libnb.so`。我们将在本章后面讨论 `libnb.so`。

之后，它会在 `native_bridge.cc` 中调用另一个 `SetupEnvironment` 函数，为当前应用程序中的 Native Bridge 设置环境。最后，它将状态设置为 `kInitialized`。现在 Native Bridge 已准备好供当前应用程序使用。

# 加载本地库

一旦 Native Bridge 准备好使用，我们可以看看当应用程序在不同的处理器架构中加载本地库时会发生什么。

我们知道，如果我们在一个共享库中实现原生方法，我们需要实现一个 `JNI_OnLoad` 入口点，用于注册原生方法。Java 代码需要调用 `System.load` 或 `System.loadLibrary` 来加载这个共享库。在以下表中，这是从 `System.loadLibrary` 到 `JNI_OnLoad` 的调用栈：

| **函数** | **类** | **语言** |
| --- | --- | --- |
| `System.loadLibrary` | Runtime | Java |
| `doLoad` | Runtime | Java |
| `nativeLoad` | Runtime | JNI |
| `Runtime_nativeLoad` | Runtime | C++ |
| `LoadNativeLibrary` | JavaVMExt | C++ |
| `JNI_OnLoad` |  | C++ |

让我们深入了解`JavaVMExt::LoadNativeLibrary`的细节。此函数定义在`$AOSP/art/runtime/jni_internal.cc`中。以下图是`JavaVMExt::LoadNativeLibrary`与Native Bridge相关部分：

![](img/image_05_006.png)

加载本地库

当Android应用程序加载本地库时，会调用此函数。通常，我们在这里指的是相同处理器架构下的本地库。通过Native Bridge，我们也可以使用此函数加载支持的不同处理器架构的本地库：

[PRE14]

`LoadNativeLibrary`函数首先调用`dlopen`来加载共享库。如果它是在不同处理器架构上的共享库，例如在Intel x86平台上的ARM库，`dlopen`调用应该失败。在这种情况下，它将尝试使用Native Bridge而不是返回错误来重新加载库。

要使用Native Bridge，它首先调用`NativeBridgeIsSupported`函数来检查Native Bridge是否受支持。`NativeBridgeIsSupported`函数调用Houdini回调函数`isSupported`来检查给定的共享库是否可以通过Native Bridge支持：

[PRE15]

如果库可以通过Native Bridge支持，`LoadNativeLibrary`将调用另一个Native Bridge函数，`android::NativeBridgeLoadLibrary`来加载库：

[PRE16]

Native Bridge的`NativeBridgeLoadLibrary`函数将调用Houdini回调函数`loadLibrary`来加载库。本地库加载成功后，库中的`JNI_OnLoad`入口点将被找到，系统将调用它来注册本地库注册的本地方法。对于正常的本地库，系统函数`dlsym`用于获取`JNI_OnLoad`方法，但`FindSymbolWithNativeBridge`函数用于从Houdini库中获取`JNI_OnLoad`：

[PRE17]

`FindSymbolWithNativeBridge`调用`NativeBridgeGetTrampoline` Native Bridge函数，而`NativeBridgeGetTrampoline`调用`getTrampoline` Houdini回调函数来完成实际工作：

[PRE18]

从前面的分析中，我们可以看出Houdini使用的ARM翻译库在Android中通过Native Bridge支持ARM二进制翻译。Houdini库与系统之间的接口是两套回调函数。在`NativeBridgeCallbacks`中定义的回调函数被系统用来对ARM本地库进行函数调用，而定义在`NativeBridgeRuntimeCallbacks`中的回调函数可以被Houdini库中的函数用来调用系统中的JNI函数。

# 将Houdini集成到x86emu设备中

本章的目标是在Android模拟器中支持Houdini ARM二进制翻译。在了解Native Bridge的内部原理之后，它是Houdini库的基础，我们可以着手为我们的x86emu设备提供Houdini支持。

由于 Houdini 库是英特尔专有库，因此它不可公开获取。对于那些想要将 Houdini 添加到新设备（如不受英特尔支持的 Android 模拟器）的人，唯一可能的方法是从受支持的设备复制 Houdini 库并将其添加到不受支持的设备。

幸运的是，开源项目 Android-x86 为任何英特尔设备提供了对 Houdini 的基本支持，我们可以将其作为本书的参考。在本章中，我们将基于 Android-x86 项目将 Houdini 添加到 Android 模拟器。

# 修改 x86emu 构建配置

在新设备上支持 Houdini 的基本步骤是：

+   根据我们在 [第 4 章](f69f330a-932c-4a32-bb78-e427c860b65e.xhtml)，*自定义 Android 模拟器* 中 *为什么要自定义 Android 模拟器？* 部分讨论的内容，更改设备配置

+   将适合版本的 Houdini 库复制到 `system` 文件夹

要处理前两个步骤，我们首先从修改 x86emu 设备配置开始。我们将为此使用 [第 4 章](f69f330a-932c-4a32-bb78-e427c860b65e.xhtml)，*自定义 Android 模拟器* 中的源代码作为基准，并在其基础上进行修改。这也是本书大多数章节中我们将采用的方法。我们将根据每个主题最简单的代码库进行独立修改。我的意思是，[第 4 章](f69f330a-932c-4a32-bb78-e427c860b65e.xhtml)，*自定义 Android 模拟器* 中 x86emu 的源代码是这个设备的最简单代码库。

既然我们已经有了一份 x86emu 的 AOSP 源代码的工作副本，我们可以在一个新分支中为本章进行修改：

[PRE19]

我为每个章节创建了一个标签，我们可以将其作为新开发的起点。`android-7.1.1_r4_x86emu_ch04_r1` 标签是 [第 4 章](f69f330a-932c-4a32-bb78-e427c860b65e.xhtml)，*自定义 Android 模拟器* 的基准。从前面的命令中，我们创建了一个新的分支，`android-7.1.1_r4_ch05`，用于本章的开发工作。我没有将开发分支推送到 GitHub，但我将所有标签推送到 GitHub。在我们完成本章的所有更改后，我们将为这一章创建一个新的标签，`android-7.1.1_r4_x86emu_ch05_r1`。

在我们完成所有修改后，我们还需要更新清单文件，以便我们可以有一个清单来下载本章的代码。我不用标签，而是用分支来管理清单。本章清单的分支是 `android-7.1.1_r4_ch05_aosp`。我们可以使用以下命令下载以下内容的源代码：

[PRE20]

如果你已按照我们在 [第 2 章](984e0cef-7bf6-4454-bede-bb34c553be12.xhtml)，*设置开发环境* 中讨论的方式设置了本地镜像，你可以按照以下方式检出源代码：

[PRE21]

你需要为你的本地镜像创建一个 `android-7.1.1_r4_ch05` 分支，它引用了 `android-7.1.1_r4_ch05_aosp` 分支。

使用前面的清单创建源代码的工作副本后，我们可以查看 `.repo/manifest.xml` 文件：

[PRE22]

在前面的清单文件中，我们使用 `android-7.1.1_r4_x86emu_ch05_r1` 标签标记所有不在 AOSP 项目中的项目。`device/generic/common` 项目是从 Android-x86 复制的，而 `device/generic/goldfish` 项目是从 AOSP 复制的。除了 `kernel` 和 `device/generic/x86emu`，这些是我们本章需要更改的两个项目。

# 扩展 x86emu 设备

一旦我们对源代码配置的所有更改都完成了，我们现在就可以开始扩展 x86emu 设备以支持 Houdini。我们需要做出哪些更改？由于我在解释之前已经做了所有更改，让我们使用一个工具来比较 [第 4 章](f69f330a-932c-4a32-bb78-e427c860b65e.xhtml) 中 *自定义 Android 模拟器* 和本章代码的源代码之间的差异。

![](img/image_05_007.png)

支持Houdini的更改

如前一个截图所示，我们添加了一个 `system` 文件夹，并修改了四个 Makefile。我们可以忽略 `x86emu_x86_64.mk` Makefile，因为我们不会在本书中讨论 64 位构建。`x86emu_x86_64.mk` 的更改与 `x86emu_x86.mk` 的更改类似，所以我们省去了重复讨论类似内容的麻烦。你自己启用 x86emu 的 64 位构建不会是一个很大的工作量。其他两个文件，`.cproject` 和 `.project`，是由于 Eclipse 集成而生成的，我们也可以忽略它们。让我们逐一查看 `BoardConfig.mk`、`x86emu_x86.mk` 和 `device.mk`。

# 对 BoardConfig.mk 的更改

在板配置文件中，我们需要将 ARM 指令集添加到 CPU ABI 列表中，以便程序可以检测对 ARM 指令集的支持，如下所示：

[PRE23]

你可能已经注意到了 `BUILD_ARM_FOR_X86` 宏。这个宏被 Android-x86 Houdini 支持使用，我们将在稍后讨论它。

# 对 x86emu_x86.mk 的更改

在产品定义的 Makefile 中，`x86emu_x86.mk`，我们将 `persist.sys.nativebridge` 属性设置为 `1`。然后我们将 `$AOSP/device/generic/x86emu/system` 文件夹下的所有文件复制到 `$OUT/system` 映像中。`$AOSP/device/generic/x86emu/system/lib/arm` 文件夹下的所有文件都是 Houdini 库的副本：

[PRE24]

# 对 device.mk 的更改

在设备 Makefile `device.mk` 中，我们只添加了一行来包含另一个 Makefile，`nativebridge.mk`，在 `device/generic/common/nativebridge` 目录下。正如我们在源配置章节中讨论的那样，我们使用 Android-x86 的版本来支持 Houdini 集成。我们将在下一节分析 `nativebridge.mk` Makefile：

[PRE25]

# 使用 Android-x86 实现

由于我们使用 Android-x86 项目的 Houdini 支持，我们可以看到我们只需要对 x86emu Makefile 进行非常小的修改。

现在让我们看看 Android-x86 中的 `nativebridge.mk`：

[PRE26]

`nativebridge.mk` 首先将 `enable_nativebridge` 脚本复制到 `system` 文件夹。之后，它设置 `ro.dalvik.vm.isa.arm` 和 `ro.enable.native.bridge.exec` 属性。这两个属性将被添加到系统镜像中的 `system/build.prop` 文件中。它还设置了默认属性 `ro.dalvik.vm.native.bridge` 为 `libnb.so`。此属性由 ART 用于查找 Houdini 库。Android-x86 使用 `libnb.so` 库而不是所有支持英特尔设备都使用的 `libhoudini.so`。`libnb.so` 库是 `libhoudini.so` 的包装器。由于我们使用 `libnb.so` 作为 ARM 二进制翻译库，我们需要将此包添加到构建中。

# 分析 libnb.so

由于 `libnb.so` 库是 Android-x86 中 Native Bridge 支持的关键起点，我们现在将深入探讨其细节。构建 `libnb.so` 的 Makefile 可以在 `device/generic/common/nativebridge/Android.mk` 中找到。`libnb.so` 的源代码仅包含一个文件，即 `libnb.cpp`，如下所示：

[PRE27]

在 `libnb.cpp` 中，我们可以看到它加载了 `libhoudini.so` 库，这是英特尔提供的原始 Houdini 库，并且它只做了两个修改。在初始化之前，它检查 `persist.sys.nativebridge` 属性。其余的代码提供了一个 `NativeBridgeCallbacks` 的包装器，包装器函数直接调用 Houdini 库中的函数。

# 使用 binfmt_misc

到目前为止，我们所讨论的是如何在英特尔 x86 架构中加载 ARM 共享库。Houdini 还可以支持在英特尔设备上运行独立的 ARM 应用程序。为此，它使用了一种称为 `binfmt_misc` 的机制。`binfmt_misc` 是 Linux 内核的一种功能，允许识别任意可执行文件格式并将其传递给某些用户空间应用程序，例如模拟器和虚拟机。

根据Linux内核文档，此内核功能允许您通过在shell中简单地键入程序名称来调用几乎每个程序。例如，这包括编译的Java (TM)、Python或Emacs。为了实现这一点，您必须告诉 `binfmt_misc` 应该使用哪个解释器调用哪个二进制文件。`binfmt_misc` 通过匹配您提供的魔字节序列（屏蔽指定的位）与文件开头的某些字节来识别二进制类型。`binfmt_misc` 还可以识别文件扩展名，如 `.com` 或 `.exe`。

要使用此方法，首先我们必须挂载 `binfmt_misc`：

[PRE28]

要注册新的二进制类型，我们必须设置一个看起来如下所示的字符串：

[PRE29]

然后我们需要将其添加到 `/proc/sys/fs/binfmt_misc/register`。

下面是这些字段的意义：

+   `name` 是一个标识符字符串。在 `/proc/sys/fs/binfmt_misc` 目录下将创建一个以该名称命名的新 `/proc` 文件。

+   `type` 是识别类型。它给出 `M` 表示 magic 和 `E` 表示扩展。

+   `offset` 是文件中 magic/mask 的偏移量，按字节计算。如果你省略它（即，你写入 `:name:type::magic...`），则默认为 0。

+   `magic` 是 `binfmt_misc` 匹配的字节序列。magic 字符串可能包含十六进制编码的字符，如 `\x0a` 或 `\xA4`。在 shell 环境中，你应该写 `\\x0a` 以防止 shell 吃掉你的 `\`。如果你选择了匹配的文件名扩展名，这是要识别的扩展名（不带 `.`，不允许 `\x0a` 特殊字符）。扩展名匹配是区分大小写的。

+   `mask` 是一个（可选，默认为所有 `0xff`）掩码。你可以通过提供一个类似于 magic 的字符串来屏蔽匹配的一些位。

+   `interpreter` 是应该用二进制文件作为第一个参数调用的程序（指定完整路径）。

+   `flags` 是一个可选字段，它控制解释器调用的几个方面。它是一个大写字母的字符串，每个字母控制一个特定的方面。

在 `nativebridge.mk` 中，它将 `enable_nativebridge` 脚本复制到 `system` 文件夹。此文件用于在 Android-x86 中启用 Houdini。在 Android-x86 中，Houdini 默认未启用。这可以通过在 Android-x86 设置应用中的选项随时打开。当然，这不在 AOSP 源代码中得到支持。当你打开 Android-x86 中的 Houdini 时，它会调用 `enable_nativebridge` 脚本。此脚本执行两件事：

1.  它从第三方项目仓库下载 Houdini 到本地仓库，并将其安装到 `/system/lib/arm` 系统目录中。它还设置 `persist.sys.nativebridge` 属性为 `1`。

1.  在此脚本的第二部分，它会在 `/proc` 目录中创建 `binfmt_misc` 文件。

我们不会直接使用 `enable_nativebridge` 脚本，但希望在系统启动时运行 `enable_nativebridge` 的第二部分。通过第二部分，Houdini 在 Android 模拟器中默认启用。这可以通过将 `enable_nativebridge` 的第二部分添加到 `device/generic/goldfish/init.goldfish.sh` 来完成。以下是我们添加到 `init.goldfish.sh` 结尾的代码片段。这是在系统启动时用于设置 Android 模拟器环境的脚本：

[PRE30]

在我们重新构建镜像并启动模拟器后，我们可以使用以下命令验证更改：

[PRE31]

我们可以看到我们注册了两个 `binfmt_misc` 类型：`arm_dyn` 和 `arm_exe`。`/proc` 文件 `arm_dyn` 用于加载共享库，而 `arm_exe` 用于加载 ARM 可执行文件：

[PRE32]

如果我们查看 `arm_exe` 的内容，从前面的输出中我们可以看到，`/system/lib/arm/houdini` 解释器用于解释 ARM 二进制文件。

# 构建和测试

我们已经对代码进行了所有更改，以启用 Houdini。我们可以使用以下命令构建系统镜像：

[PRE33]

在我们构建系统镜像之后，我们可以对其进行测试。当然，我们可以使用任何可以在 ARM 架构上运行的 Android 应用程序来测试镜像。然而，为了获取有关测试目标的详细信息，我们将使用两个单元测试应用程序来验证本章中的工作。第一个是一个独立的 ARM 应用程序，可以从命令行运行。第二个是一个仅针对 ARM 的 JNI 共享库的 Android 应用程序。本章中的 Android 模拟器镜像和测试二进制文件可以从 [https://sourceforge.net/projects/android-system-programming/files/android-7/ch05/ch05.zip/download](https://sourceforge.net/projects/android-system-programming/files/android-7/ch05/ch05.zip/download) 下载。

这两个测试应用程序的源代码托管在 GitHub 上。您可以从 [https://github.com/shugaoye/asp-sample/tree/master/ch05](https://github.com/shugaoye/asp-sample/tree/master/ch05) 获取源代码。

要构建测试应用程序，您需要同时拥有 Android SDK 和 NDK，这样您就可以构建 Android 应用程序和本地应用程序。

# 测试命令行应用程序

在您克隆了测试应用程序的先前 Git 仓库之后，您可以构建和测试它们。让我们首先测试命令行应用程序。这是一个非常简单的“hello world”应用程序，它将一行消息打印到标准输出，如下所示：

[PRE34]

您可以在模拟器中按照以下方式构建和测试它：

[PRE35]

在构建完成后，您可以使用 `file` 命令检查文件格式。您可以看到输出是一个32位的ARM ELF文件。您可以使用 `adb` 将此二进制文件推送到模拟器并运行它。您将看到它可以将输出消息正确地打印到标准输出。

# 测试 Android JNI 应用程序

接下来，让我们测试带有 ARM JNI 库的 Android 应用程序。

JNI 库位于 `ch05/test2/jni`。支持的处理器架构在 `Application.mk` 中定义如下：

[PRE36]

我们可以看到我们为 `armeabi` 和 `armeabi-v7a` 构建了 JNI 库。让我们首先使用 NDK 构建JNI库：

[PRE37]

在我们构建 JNI 库之后，我们可以将 Android 源代码导入到 Eclipse 或 Android Studio 中来构建应用程序本身。我们不会解释导入和构建 Android 应用程序的详细步骤。您可以通过阅读关于如何开发 Android 应用程序和如何开发 JNI 库的书籍来了解更多信息。我们在这里想要调查的是测试结果。在我们获得 APK 文件后，我们可以在模拟器中安装并运行它。同时，我们可以使用 logcat 捕获调试日志。以下是我环境的日志：

[PRE38]

从前面的日志消息中我们可以看到 Houdini 已成功初始化，并且 `libHelloJNI.so` JNI 库已被 Native Bridge 加载。

# 摘要

在本章中，我们首先介绍了 Android 架构中的 Native Bridge，以便我们了解其工作原理。基于我们对 Native Bridge 的理解，我们扩展了 x86emu 设备以支持 Houdini。我们修改了 x86emu 设备的 Makefiles，并且还利用了开源项目 Android-x86 来节省集成工作。在我们将 Houdini 集成到 x86emu 之后，我们测试了两种 Houdini 的使用场景：

+   一个独立的命令行应用程序

+   一个使用 JNI 构建的本地共享库的 Android 应用程序

在下一章中，我们将探讨更多关于 x86emu 启动过程的内容，并且我们将学习如何使用定制的 ramdisk 图像来调试启动过程。
