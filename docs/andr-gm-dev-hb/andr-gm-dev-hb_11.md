# 第十一章。使用 C ++和 OpenGL 进行 Android 游戏开发

我们已经看到了 Android 应用程序和 Android 游戏之间的区别。 Android SDK 非常擅长处理这两者。当然，引起的问题是“在本地语言（如 C 和 C ++）中需要单独的开发工具集是什么要求？”与 Java 相比，C 和 C ++更难管理和编写代码。答案就在问题本身。 Java 架构在 JVM 上运行，与 Android 操作系统相关联。这会产生额外的延迟，导致性能滞后。滞后的规模取决于应用程序的规模。

高度 CPU 密集型的应用程序可能会在 Java 架构中导致大量可见的延迟。本地语言代码可以更快地处理。此外，本地代码可以根据 CPU /平台架构的不同而变化，而在 Android SDK 使用的 Java 架构中是不可能的。

Android 使用 OpenGL 渲染系统。因此，使用 OpenGL 制作的应用程序也可以选择 Android 作为目标平台。本地代码有助于直接使用 OpenGL 构建应用程序。

我们将通过以下主题在本章中详细了解这些方面：

+   介绍 3Android NDK

+   游戏中的 C ++-优缺点

+   本地代码性能

+   OpenGL 简介

+   使用 OpenGL 进行渲染

+   不同的 CPU 架构支持

# Android NDK 简介

Android 实际上是基于 Java 架构的。但是，Android 应用程序的一部分可以使用 C 和 C ++等本地语言开发。这就是 Android NDK 出现的地方。

Android NDK 是一个工具集，用于开发将与硬件交互得更快的应用程序模块。众所周知，C 和 C ++能够直接与本地组件交互，从而减少应用程序和硬件之间的延迟。

## NDK 的工作原理

Android 本地代码段通过**Java 本机接口**（**JNI**）与主应用程序进行交互。 Android NDK 附带一个构建脚本，将本地代码转换为二进制代码并将其包含在主 Android 应用程序中。

该二进制代码基本上是一个本地代码库，可以根据需要在任何 Android 应用程序中使用。 NDK 构建脚本创建`.so`文件并将其添加到应用程序路径中。

Android 构建过程创建 Dalvik 可执行文件（`.dex`）以在 Android OS 的 Dalvik 虚拟机（或 ART）上运行。 Java 可执行文件识别本地库并加载已实现的方法。本地方法使用`native`关键字声明：

```kt
public native void testFunc (int param);
```

该方法始终具有公共访问权限，因为本地库始终被视为外部来源。在这里，开发人员应始终牢记，对于所有包含的本地库的同一声明，永远不应该有多个方法的定义。这将始终创建编译错误。

本地构建过程可以将本地项目构建为两种类型的库：

+   本地共享库

+   本地静态库

### 本地共享库

本地构建脚本从 C ++文件创建`.so`文件，称为本地共享库。但是，它并不总是在真正意义上在应用程序之间共享。 Android 应用程序是一个 Java 应用程序，但是本地应用程序可以通过本地共享库触发。

对于游戏开发，如果游戏是用本地语言编写的，则游戏代码将包含在共享库中。

### 本地静态库

本地静态库基本上是已编译对象的集合，并由`.a`文件表示。这些库包含在其他库中。编译器可以在编译过程中删除未使用的代码。

## 构建依赖

Android SDK 能够使用 Java 构建和打包 Android 应用程序项目为 APK 文件。然而，NDK 不足以构建和打包 APK 文件。除了 NDK 之外，创建 Android 应用程序 APK 的依赖如下：

+   Android SDK

+   C++编译器

+   Python

+   Gradle

+   Cygwin

+   Java

### Android SDK

Android 应用程序基本上是 Java 应用程序。因此，有必要拥有 Android SDK 来创建 Android 应用程序包。

### C++编译器

本机 Android 应用程序是用 C++编写的，因此需要 C++编译器在开发平台上编译代码库。C++编译器是平台相关的，因此可能不是每个平台上都是相同的 C++编译器。

例如，在 Windows 机器上，目前在开发行业中使用 C++11 编译器，而在 Linux 机器上使用 GC++编译器。

这可能会在语法和 API 调用方面为实际开发项目创建不同的代码库。

### Python

Python 是一种独立的开发语言。它可以用于创建 Android 应用程序，并通过将源代码转换为本机语言来支持多个平台。在 Android NDK 开发的情况下，Python 用于将 C++代码转换为本机二进制。

### Gradle

Gradle 被构建脚本和 Android 本机构建工具使用，将本机代码转换为共享库。它还提供了一个虚拟的 Unix 环境来制作应用程序包。

### Cygwin

Android 需要一个 Unix 环境来构建 NDK 应用程序项目。Windows 系统没有 Unix 环境。需要 Cygwin 来提供虚拟的 Unix 环境来支持构建平台。

### Java

最后但并非最不重要的是需要 Java 来创建 Android 应用程序包。然而，对于任何类型的 Android 开发，都需要 Java。

## 本机项目构建配置

为了从本机源代码创建应用程序包，Android 项目需要以下配置。本机项目构建取决于这两个文件中定义的配置：

+   `Android.mk`

+   `Application.mk`

### Android.mk 配置

**位置**

`Android.mk`文件可以在`<应用程序项目路径>/jni/`中找到。

**配置选项**：

`Android.mk`文件包含以下选项来创建应用程序包：

+   `CLEAR_VARS`：这清除本地和用户定义的变量。这个选项由`include $(CLEAR_VARS)`语法调用。

+   `BUILD_SHARED_LIBRARY`：这包括所有本地文件，在`LOCAL_MODULE`和`LOCAL_SRC_FILES`中定义，以共享库的形式。它由`include $(BUILD_SHARED_LIBRARY)`语法调用。

+   `BUILD_STATIC_LIBRARY`：这指定静态库以创建由共享库使用的`.a`文件。它由`include $(BUILD_STATIC_LIBRARY)`语法调用。

+   `PREBUILT_SHARED_LIBRARY`：这表示特定路径上的预构建共享库，用于从本地包含构建依赖的共享库。它由`include $(PREBUILT_SHARED_LIBRARY)`语法调用。

+   `PREBUILT_STATIC_LIBRARY`：这表示特定路径上的预构建静态库，用于从本地包含构建依赖的共享库。它由`include $(PREBUILT_STATIC_LIBRARY)`语法调用。

+   `TARGET_ARCH`：这表示基本处理器架构系列，如 ARM、x86 等。

+   `TARGET_PLATFORM`：这定义目标 Android 平台。所述平台必须通过 Android SDK 管理器安装在开发系统中。它指示 Android API 级别以创建应用程序包。

+   `TARGET_ARCH_ABI`：这表示目标处理器架构的特定 ABI，如 armeabi、armeabi-v7、x86 等。

+   `LOCAL_PATH`：这指向当前文件目录。这个变量不会被`CLEAR_VARS`命令清除。它由`LOCAL_PATH := $ (call my-dir)`语法调用。

+   `LOCAL_MODULE`：这表示所有唯一的本地模块名称。它由`LOCAL_MODULE := "<模块名称>"`语法调用。

+   `LOCAL_MODULE_FILENAME`：这表示包含已定义`LOCAL_MODULE`的库名称。它由`LOCAL_MODULE_FILENAME := "<模块库文件名>"`语法调用。

+   `LOCAL_SRC_FILES`：这表示要编译为共享库的所有本地源代码文件路径。它由`LOCAL_SRC_FILES := <本地源文件路径>`语法调用。

还有其他可选配置可以在此文件中设置，例如`LOCAL_C_INCLUDES`、`LOCAL_CFLAGS`、`LOCAL_CPP_EXTENSION`、`LOCAL_CPP_FEATURES`、`LOCAL_SHARED_LIBRARIES`、`LOCAL_STATIC_LIBRARIES`和`LOCAL_EXPORT_CFLAGS`。

### Application.mk 配置

**位置**

`Application.mk`文件可以位于`<应用程序项目路径>/jni/`。

**配置选项**

`Application.mk`文件包含以下选项以创建应用程序包：

+   `APP_PROJECT_PATH`：这是项目根目录的绝对路径。

+   `APP_OPTIM`：这表示可选设置，用于将构建包创建为发布版或调试版。

+   `APP_CFLAGS`：这定义了一组 C 编译器标志，用于构建而不是在`Android.mk`文件中更改。

+   `APP_CPPFLAGS`：这定义了一组 C++编译器标志，用于构建而不是在`Android.mk`文件中更改。

+   `APP_BUILD_SCRIPT`：这是一个可选设置，用于指定除默认的`jni/Android.mk`脚本之外的构建脚本。

+   `APP_ABI`：此选项指定要为 Android 应用程序包进行优化的 ABI 集。以下是每个 ABI 支持的完整列表和关键字：

+   ARMv5：`armeabi`

+   ARMv7：`armeabi-v7a`

+   ARMv8：`arm64-v8a`

+   Intel 32 位：`x86`

+   Intel 64 位：`x86_64`

+   MIPS 32 位：`mips`

+   MIPS 64 位：`mips64`

+   ALL-SET：`all`

+   `APP_PLATFORM`：此选项指定目标 Android 平台。

+   `NDK_TOOLCHAIN_VERSION`：此选项指定 GCC 编译器的版本。默认情况下，64 位和 32 位分别使用版本 4.9 和 4.8 进行编译。

+   `APP_STL`：这是一个可选的配置，用于链接替代 C++实现。

+   `APP_LDFLAGS`：在构建共享库和可执行文件的情况下，此选项用于将链接标志链接到构建系统以链接应用程序。

# 用于游戏的 C++-优点和缺点

C++和 Java 之间有一个永无止境的争论。然而，我们不会讨论争议，而是试图从游戏开发的角度来看待它们。C++在性能上略优于 Java，而 Java 以其简单性而闻名。

可能有许多程序员更喜欢 C++而不是 Java，或者反之亦然。在游戏开发中，编程语言的个人选择并不重要。因此，使用 NDK 或 SDK 必须根据需求来确定。建议您始终使用 Android SDK 来开发应用程序，而不是使用 NDK。

让我们讨论使用本地语言进行游戏编程的优缺点。

## 使用 C++的优势

让我们首先从以下几点来看看使用 C++进行游戏编程的积极方面：

+   通用游戏编程语言

+   跨平台可移植性

+   更快的执行

+   CPU 架构支持

### 通用游戏编程语言

在游戏开发的情况下，C++被广泛用于许多平台，特别是用于主机和 PC 游戏开发。这就是为什么许多游戏引擎选择 C++作为主要编程语言的原因。

有时，学习许多编程语言以在不同架构的不同平台上工作是困难的。C++提供了这个问题的最常见解决方案，因为大多数程序员都熟悉 C++库和 API 的使用。

### 跨平台可移植性

相同的 C++代码被编译为针对特定操作平台的库。因此，同一个项目可以编译为不同的平台。因此，如果游戏是用 C++编写的，那么将游戏移植到各种平台就会变得非常容易。

例如，著名而有效的跨平台游戏引擎 Cocos2d-x 使用 C++作为开发语言。因此，相同的游戏可以轻松移植到许多平台，如 Android、iOS、Mac OS 和 Windows。

### 更快的执行

C++能够很好地与平台硬件交互，并且使用 C++编写游戏有助于提高性能。然而，在 Android 的情况下，如果游戏不是 CPU 密集型，性能提升几乎是不可察觉的。

### CPU 架构支持

C++代码可以针对特定的目标 CPU 架构进行编译，如 x86、ARM、Neon 或 MIPS。这种规范表明在特定处理器上有更好的性能。

在 Android NDK 中为 CPU 架构配置编译器确保在每个平台上获得最佳结果。然而，并不总是需要为避免额外的编译而定义每个平台。

## 使用 C++的缺点

现在，让我们通过以下几点来讨论问题的另一面：

+   高程序复杂性

+   依赖平台的编译器

+   手动内存管理

### 高程序复杂性

C++带来了额外的程序复杂性。在 Java 编程的情况下，JVM 完全负责内存管理，并遵循面向对象的概念。C++在提供这一特性方面存在不足。因此，开发人员需要额外的工作来处理每个编程方面。

与 Java 相比，C++本身的架构更加复杂。如果使用 C++，面临异常和错误的机会会增加。

### 依赖平台的编译器

使用 C++进行跨平台开发很容易。然而，在大多数情况下，配置构建脚本可能会很麻烦。同样的游戏由于错误的配置而无法在移植的平台上运行是非常常见的情况。此外，由于游戏在其他平台上成功运行，因此很难找出问题所在。

大多数情况下，不同的平台使用不同的 C++编译器。因此，需要额外的努力来识别特定于平台的代码，并在必要时为每个平台找到替代方案。 

### 手动内存管理

Java 不需要开发人员实现内存管理，内存由 JVM（在 Android 的情况下是 DVM）高效管理。因此，不会出现内存泄漏或碎片化的情况。JVM 会运行垃圾收集器来自动释放未使用的内存。然而，垃圾收集器的调用会消耗一些性能，频繁的垃圾收集器调用可能会导致严重的性能下降。

开发人员应该使用最佳内存，因为如果代码中有任何活动引用，垃圾收集器无法识别未使用的内存块。

## 结论

C++有其自身的优势。然而，当涉及到为 Android 进行游戏编程时，在技术上并没有太大帮助。因此，如果我们比较选择 C++和在 Android 上使用 Java 编码所需的工作量和风险，Java 应该始终优先考虑。DVM 能够有效地运行 Java 代码，以在 Android 设备上实现合理的性能。此外，Android NDK 库实际上并不是为开发独立的 Android 应用程序而设计的。尽管它具有本地活动支持，作为 DVM 和用 C++编写的本地应用程序之间的中间层，但并没有太大帮助。

如果开发人员选择不跨平台，并将游戏范围限制在 Android 上，那么建议您使用 Android SDK 而不是 Android NDK。这将减少开发的麻烦和复杂性，同时性能损失可以忽略不计。

# 本地代码性能

正如我们已经知道的，本地代码可以以更快的处理速度运行。这可以进一步针对特定的 CPU 架构进行优化。这种性能提升的主要原因是在内存操作中使用指针。但是，这取决于开发人员和编码风格。

让我们看一个简单的例子，我们可以更好地理解本地语言中的性能增益。

考虑这段 Java 代码：

```kt
int[] testArray = new int[1000];
for ( int i = 0; i < 1000; ++ i)
{
  testArray[i] = i;
}
```

在这种情况下，数组中 1000 个字段的地址由 JVM（在 Android Dalvik 系统中为 DVM）处理。因此，解释器每次解析到第*i*个位置并执行赋值操作，这需要很长时间。

现在，让我们使用本地 C/C++语言实现相同的功能并使用指针：

```kt
int testArray[1000];
int *ptrArray = testArray;
for ( int i = 0; i < 1000; ++ i)
{
  *ptrArray = i;
  ptrArray += i * sizeof(int);
}
```

在这个例子中，解释器不需要解析到目标内存位置。`ptrArray`指出了位置的地址。因此，值可以直接赋给内存位置。

特别是对于多维数组，在正确编写的本地代码中可以观察到显著的性能增益，用于相同功能。本地代码的另一个重要用途是二进制数据处理和图像处理，在这些情况下，大量数据一次性处理。

# 使用 OpenGL 进行渲染

Android 使用 OpenGL 进行渲染。Android SDK 库包括专门针对 Android 进行优化的 OpenGL 库。Android 从 API 级别 4 开始支持 OpenGL，然后随着级别的增加而增加其支持。目前，OpenGL 的最大支持版本是从 API 级别 21 开始的 OpenGL ES 3.1。

## OpenGL 版本

不同的 OpenGL 版本具有不同的功能集。版本 1.0 和 2.0 在编码风格、API 便利性、功能和特性支持方面有很多不同之处。让我们讨论一下对 Android 开发具有重要意义的以下 OpenGL ES 版本：

+   OpenGL ES 1.x

+   OpenGL ES 2.0

+   OpenGL ES 3.0

+   OpenGL ES 3.1

### OpenGL 1.x

从 Android API 级别 4 开始支持 OpenGL 版本 1.x，使用共享的 OpenGL ES 1.x 库`libGLESv1.so`。头文件`gl.h`和`glext.h`包含了 OpenGL 功能所需的所有必要 API。

### OpenGL 2.0

在当前行业中，开发人员更倾向于在游戏中使用 OpenGL ES 2.0，因为几乎每台设备都支持这个 OpenGL 版本，并且它提供了对游戏有用的顶点和片段着色器。OpenGL ES 2.0 可以通过在项目中包含`libGLESv2.so`共享库来用于 Android 本地开发项目，如下所示：

```kt
LOCAL_LDLIBS := -lGLESv2
```

头文件是`gl2.h`和`gl2ext.h`。OpenGL ES 2.0 从 Android API 级别 5 开始支持。

### OpenGL 3.0

从 Android API 级别 21 开始，支持 OpenGL ES 3.0。开发人员可以包含`libGLESv3.so`来使用 OpenGL 3.1，如下所示：

```kt
LOCAL_LDLIBS := -lGLESv3
```

头文件是`gl3.h`和`gl3ext.h`。

### OpenGL 3.1

从 Android API 级别 21 开始，支持 OpenGL ES 3.1。开发人员可以包含`libGLESv3.so`来使用 OpenGL 3.1，如下所示：

```kt
LOCAL_LDLIBS := -lGLESv3
```

头文件是`gl31.h`和`gl3ext.h`。

许多 Android 设备不支持 OpenGL ES 3.0 和 OpenGL ES 3.1。如果开发人员打算使用它们，则在使用版本之前应该进行 OpenGL 版本检查。此外，必须使用适当的 OpenGL ES 版本在特定设备上运行游戏。最新的 Android N 支持 OpenGL ES 3.2。

### 检测和设置 OpenGL 版本

这段 Android Java 代码可以用来为 Android 游戏实现适当的 OpenGL ES 支持：

```kt
private GLSurfaceView glSurfaceView;
void setOpenGLVersion()
{
  final boolean supportOpenGLEs3 = configurationInfo.reqGlEsVersion >= 0x30000;

  if (supportOpenGLEs3) 
  {
    glSurfaceView = new GLSurfaceView(this);
    glSurfaceView.setEGLContextClientVersion(3);
    glSurfaceView.setRenderer(new RendererWrapper());
    setContentView(glSurfaceView);
  }
  else
  {
  final boolean supportOpenGLEs2 = configurationInfo.reqGlEsVersion >= 0x20000;

  if (supportsOpenGLEs2) 
    {
      glSurfaceView = new GLSurfaceView(this);
      glSurfaceView.setEGLContextClientVersion(2);
      glSurfaceView.setRenderer(new RendererWrapper());
      setContentView(glSurfaceView);
    }
    else
    {
      glSurfaceView = new GLSurfaceView(this);
      glSurfaceView.setEGLContextClientVersion(1);
      glSurfaceView.setRenderer(new RendererWrapper());
      setContentView(glSurfaceView);
    }
  }
}
```

## 纹理压缩和 OpenGL

纹理压缩对由 OpenGL 处理的渲染过程有重要影响。它可以增加或减少不同类型的纹理压缩的性能。让我们快速看一下一些重要的纹理压缩格式：

+   ATC

+   PVRTC

+   DXTC

### ATC

ATI 纹理压缩通常称为 ATITC。该压缩支持 RGB 和带或不带 alpha 通道。这是 Android 最常见和广泛使用的压缩技术。

### PVRTC

Power VR 纹理压缩使用 2 位和 4 位像素压缩，带有或不带有 alpha 通道。这被全球许多游戏开发人员使用。

### DXTC

DXTC 也被称为 S3 纹理压缩，也用于 OpenGL。这使用 4 位或 8 位 ARGB 通道。

## OpenGL 清单配置

Android 需要应用程序中使用的 OpenGL 版本定义，以及其他所需选项。

以下是 OpenGL ES 的版本声明语法：

```kt
<uses-feature android:glEsVersion=<Target version goes here> android:required="true" />
```

以下是目标版本选项：

+   `0x00010000`代表版本 1.0

+   `0x00010001`代表版本 1.1

+   `0x00020000`代表版本 2.0

+   `0x00030000`代表版本 3.0

+   `0x00030001`代表版本 3.1

+   `0x00030002`代表版本 3.2

以下是纹理压缩声明的可选设置：

```kt
<supports-gl-texture android:name=<Compression support type goes here> />
```

这些是压缩类型选项：

+   `GL_OES_compressed_ETC1_RGB8_texture`

+   `GL_OES_compressed_paletted_texture`

+   `GL_EXT_texture_compression_s3tc`

+   `GL_IMG_texture_compression_pvrtc`

+   `GL_EXT_texture_compression_dxt1`

+   `GL_EXT_texture_compression_dxt2`

+   `GL_EXT_texture_compression_dxt3`

+   `GL_EXT_texture_compression_dxt4`

+   `GL_EXT_texture_compression_dxt5`

+   `GL_AMD_compressed_3DC_texture`

+   `GL_EXT_texture_compression_latc`

+   `GL_AMD_compressed_ATC_texture`

+   `GL_ATI_texture_compression_atitc`

然而，并非所有设备都支持所有纹理压缩。开发人员应根据硬件和 Android 版本要求始终选择目标纹理压缩。

### 注意

如果目标设备不支持声明的纹理格式，则 Google 会自动进行设备过滤。

## 选择目标 OpenGL ES 版本

正如您已经了解的，不是所有设备都支持所有 OpenGL 版本。因此，在开发游戏之前选择正确的 OpenGL 版本非常重要。在选择 OpenGL 版本时，应评估以下几个因素：

+   性能

+   纹理支持

+   设备支持

+   渲染特性

+   编程舒适性

### 性能

注意到 OpenGL 版本 3.x 比 OpenGL 版本 2.x 更快，后者比 OpenGL 1.x 更快。因此，在游戏中始终最好使用最新可能的版本。

### 纹理支持

纹理压缩支持因 OpenGL 版本而异。旧版本支持旧的纹理压缩因素。此外，Android 版本支持并非所有 OpenGL 版本都通用。因此，最好使用最新可能的版本来支持纹理。

### 设备支持

这个限制使开发人员脚踏实地。并非所有设备都支持最新版本的 OpenGL。因此，为了针对更广泛的设备范围，用户应将 OpenGL 版本更改为 2.0，因为大多数设备支持这个版本。

### 渲染特性

随着 OpenGL 版本的增加，功能列表成为选择 OpenGL 版本的重要因素。开发人员必须了解开发应用程序所需的支持，并相应地选择版本。

### 编程舒适性

OpenGL 的各个版本之间存在巨大的编码风格和 API 更改。开发人员应该选择版本，如果可以在公司轻松开发。

# 不同的 CPU 架构支持

开发人员有机会为单独的处理器架构优化 Android 应用程序。从高层次的角度来看，这是一个很好的功能。然而，这个功能是以巨大的成本为代价的。让我们来看看这个功能的细节。

## 可用的 CPU 架构

以下是 NDK 构建当前支持的架构：

+   ARM

+   x86

+   Neon

+   MIPS

### ARM

**ARM**代表**Acorn RISC Machine**。这是一个基于**RISC**（**Reduced Instruction Set Computing**）的处理器，主要针对嵌入式或移动计算。正如其基础所说，它对于 Android 等操作系统非常高效。

目前，Android 平台上使用最多的处理器来自 ARM 家族。它可以进一步细分如下：

+   ARMv5TE

+   ARMv7

+   ARMv8

### x86

英特尔推出了处理器的**x86**架构。最初，这些处理器主要用于台式机/笔记本电脑。然而，它们经过优化后也可以用于移动设备，以 Celeron 或 Atom 处理器的形式。

Android NDK 构建可以设置两种类型的 x86 架构：

+   i686

+   x86-64

### Neon

**Neon**架构是基于 ARM 技术的，以进一步优化移动计算。Android 构建也可以针对这种特定架构进行优化。所有 Cortex 处理器基本上都是基于 Neon 的处理器。

### MIPS

**MIPS**代表**无互锁流水线级**微处理器。在这个类别中有 32 位和 64 位处理器的变体。正如名称所示，这种架构用于嵌入式设备中的微处理器进行小规模计算。后来，它以 64 位架构引入到 Android 中。然而，这种类型的处理器在今天的 Android 系统中很少使用。

## 集成多种架构支持的优缺点

Android 移动设备在内存和处理能力方面有不同的配置。包括单独的架构支持可能会增加与更大的构建大小相对应的性能。

本地构建工具为每个目标处理器构建一个单独的共享库，并将其包含在构建包中。

以下是提供单独处理器架构支持的一些优缺点。

现在让我们先看看优点：

+   **更快的操作**：为单独的处理器提供单独的架构会导致游戏指令的处理速度更快。如果处理器架构受到 Android 应用的支持，那么处理器就不需要执行任何转换，并且可以以更快的速度运行指令。

+   **处理器的最佳使用**：操作系统总是寻找集成处理器的特定架构。相同的架构可以最佳地利用处理器。

+   最低功耗：最佳处理直接意味着最佳和最低的处理功耗。

+   **最佳内存使用**：如果 Android 应用支持相同的处理器架构，处理器就不需要使用额外的运行时内存来执行指令。

现在让我们看看缺点：

+   **更大的构建大小**：为单独的架构使用单独的共享库会显著增加构建包的大小。整个本地指令代码都会在具有不同处理器优化的单独共享库中重新编写。

+   **减少目标设备数量**：如果 APK 的大小很大，将其适应低存储设备会带来更多问题。因此，设备支持变得更少。

# 总结

我们在本章中简要介绍了 Android NDK，并解决了一些关于本地开发的疑惑。有许多开发人员认为用本地语言开发游戏可以获得巨大的处理能力。然而，这并不总是正确的。处理和性能取决于开发风格和标准。在大多数常见情况下，本地开发和 SDK 开发之间的差异是可以忽略不计的。

OpenGL 在任何情况下都可以与 Android 一起工作。后端渲染基于 OpenGL，无论是 NDK 还是 SDK。我们已经讨论了 OpenGL 的所有技术方面。在这里，您了解了哪个版本的 OpenGL 适用于 Android 以及我们应该使用哪个版本。显然，OpenGL ES 2.0 是一个不错的选择，因为大多数 Android 设备都支持它。另一方面，OpenGL ES 1.0 已经过时，而 OpenGL ES 3.0 尚未得到大多数 Android 设备的支持。

到目前为止，我们已经涵盖了 Android 游戏开发的几乎所有方面。然而，完成游戏的实现并不意味着开发周期的完成。开发人员需要在游戏达到发布准备状态后对其进行打磨，以提高其整体质量。我们将在下一章讨论游戏的打磨，以表示游戏开发过程的完成。
