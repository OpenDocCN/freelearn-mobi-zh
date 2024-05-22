# 第二章：本地库

在本章中，你将学习如何使用 Android NDK 构建流行的 C/C++库，并将它们链接到你的应用程序中。这些库是实现图像、视频、声音、物理模拟和完全在 C++中网络功能的丰富功能应用程序的构建块。我们将提供最小示例来演示每个库的功能。音频和网络库将在后续章节中详细讨论。我们将向你展示如何编译库，当然也会提供一些简短的示例和如何开始使用它们的提示。

在不同处理器和操作系统间移植库的典型陷阱包括内存访问（结构对齐和填充）、字节序（大小端）、调用约定和浮点问题。前面章节中描述的所有库都很好地解决了这些问题，即使其中一些库没有正式支持 Android NDK，修复这些问题也只是几个编译器开关的问题。

为了构建上述库，我们需要为 Windows、Linux 和 OS X 创建 makefile，并为 NDK 创建一对`Android.mk/Application.mk`文件。库的源文件被编译成目标文件。一系列目标文件组合成一个档案，这也称为静态库。之后，这个静态库可以作为链接器的输入。我们从桌面版本开始，首先为 Windows。

为了构建特定于 Windows 的库版本，我们需要一个 C++编译器。我们将使用来自 MinGW 的 GCC 工具链，该工具链在第一章，*使用命令行工具*中描述。对于每个库，我们有一系列源代码文件，我们需要得到静态库，一个带有`.a`扩展名的文件。

# 处理预编译的静态库

将我们需要的库在不同平台构建的源代码放入`src`目录中。Makefile 脚本应该如下开始：

```java
CFLAGS = -O2 -I src
```

这行定义了一个变量`CFLAGS`，其中包含编译器命令行参数的列表。在我们的例子中，我们指示编译器在`src`目录中查找头文件。如果库源代码跨越多个目录，我们需要为每个目录添加`–I`开关。`-O2`开关告诉编译器启用 2 级优化。接下来，我们为每个源文件添加以下行：

```java
<SourceFileName>.o:
gcc $(CFLAGS) –c <SourceFile>.cpp –o <SourceFile>.o
```

字符串`<SourceFileName>`应该被替换为实际的`.cpp`源文件名，并且这些行应该针对每个源文件编写。

现在，我们添加目标文件列表：

```java
ObjectFiles = <SourceFile1>.o <SourceFile2>.o
```

最后，我们将编写我们库的目标：

```java
<LibraryName>:
ar –rvs <LibraryName>.a $(ObjectList)
```

Makefile 脚本中除了空行和目标名称以外的每一行都应该以制表符开头。要构建库，请调用以下命令：

```java
>make <LibraryName>.a

```

当在我们的程序中使用库时，我们将`LibraryName.a`文件作为参数传递给`gcc`。

Makefile 由类似于编程语言中子例程的目标组成，通常每个目标都会生成一个目标文件。例如，我们已经看到，库的每个源文件都编译成相应的目标文件。

目标名称可能包括文件名模式以避免复制粘贴，但在最简单的情况下，我们只需列出所有源文件，并复制这些行，将`SourceFileName`字符串替换为适当的文件名。`gcc`命令后的`–c`开关是编译源文件的选项，而`–o`指定输出目标文件的名字。`$(CFLAGS)`符号表示将`CFLAGS`变量的值代入命令行。

Windows 的 GCC 工具链包括`ar`工具，它是归档器的缩写。我们库的 Makefile 调用此工具来创建库的静态版本。这将在 Makefile 脚本的最后几行完成。

当带有目标文件列表的一行变得过长时，可以使用反斜杠符号将其分成多行，如下所示：

```java
ObjectFileList = FileName1.o \
                 ... \
                 FileNameN.o
```

反斜杠后面不应该有空白，因为这是`make`工具的限制。`make`工具是可移植的，因此同样的规则精确适用于我们使用的所有桌面操作系统：Windows、Linux 和 OS X。

现在，我们能够使用 Makefiles 和命令行构建大多数库。让我们为 Android 构建它们。首先，创建一个名为`jni`的文件夹，并创建一个`jni/Application.mk`文件，其中包含适当的编译器开关并相应地设置库的名称。例如，Theora 库的一个应该如下所示：

```java
APP_OPTIM := release
APP_PLATFORM := android-19
APP_STL := gnustl_static
APP_CPPFLAGS += -frtti
APP_CPPFLAGS += -fexceptions
APP_CPPFLAGS += -DANDROID
APP_ABI := armeabi-v7a-hard
APP_MODULES := Theora
NDK_TOOLCHAIN_VERSION := clang
```

### 注意

在这里，我们将使用`armeabi-v7a-hard`作为支持最广泛的现代 ABI 之一。Android NDK 支持许多其他架构和 CPU。请参考 NDK 程序员指南以获取完整且最新的列表。

它将使用安装的 NDK 中可用的最新版本的 Clang 编译器。`jni/Android.mk`文件与我们之前章节为`3_NDK`示例应用程序编写的文件类似，但有一些例外。在文件顶部，必须定义一些必要的变量。让我们看看 OpenAL-Soft 库的`Android.mk`文件可能的样子：

```java
TARGET_PLATFORM := android-19
LOCAL_PATH := $(call my-dir)
include $(CLEAR_VARS)
LOCAL_ARM_MODE := arm
LOCAL_MODULE := OpenAL
LOCAL_C_INCLUDES += src
LOCAL_SRC_FILES += <ListOfSourceFiles>
```

定义一些常见的编译器选项：将所有警告视为错误（`-Werror`），定义`ANDROID`预处理符号：

```java
COMMON_CFLAGS := -Werror -DANDROID
```

编译标志根据选定的 CPU 架构定义：

```java
ifeq ($(TARGET_ARCH),x86)
  LOCAL_CFLAGS := $(COMMON_CFLAGS)
else
  LOCAL_CFLAGS := -mfpu=vfp -mfloat-abi=hard -mhard-float -fno-short-enums -D_NDK_MATH_NO_SOFTFP=1 $(COMMON_CFLAGS)
endif
```

在我们所有的示例中，我们将使用硬件浮点 ABI `armeabi-v7a-hard`，因此让我们相应地构建库。

### 注意

armeabi-v7a-hard 和 armeabi-v7a 之间的主要区别在于，硬件浮点 ABI 在 FPU 寄存器内部传递浮点函数参数。在浮点密集型应用程序中，这可以显著提高代码的性能，其中浮点值在不同的函数之间传递。

由于我们正在构建静态库，我们需要在`Android.mk`文件末尾添加以下行：

```java
include $(BUILD_STATIC_LIBRARY)
```

现在构建静态库只需调用一次`ndk-build`脚本。在对动态链接和 Windows 平台做一点简短的说明之后，我们继续编译实际的库。

# 在 Windows 平台上的动态链接

本章考虑的库可以作为 Windows 的动态链接库进行构建。我们不提供这样做的方法，因为每个项目已经包含了所有必要的说明，而且 Windows 开发不是本书的重点。唯一的例外是 libcurl 和 OpenSSL 库。我们建议您从官方库网站下载预构建的 DLL 文件。

在 FreeImage、FreeType 和 Theora 的示例代码中，我们使用函数指针，这些指针使用 WinAPI 的`GetProcAddress()`和`LoadLibrary()`函数进行初始化。在 Android 上使用相同的函数指针，但在这种情况下，它们指向静态库中的相应函数。

例如，函数`FreeImage_OpenMemory()`声明如下：

```java
typedef FIMEMORY* ( DLL_CALLCONV* PFNFreeImage_OpenMemory )
  ( void*, unsigned int );
PFNFreeImage_OpenMemory  FI_OpenMemory = nullptr;
```

在 Windows 上，我们使用`GetProcAddress()`调用来初始化指针：

```java
FI_OpenMemory = (PFNFreeImage_OpenMemory)
  GetProcAddress (hFreeImageDLL,"FreeImage_OpenMemory");
```

在 Android、OSX 和 Linux 上，这是一个重定向：

```java
FI_OpenMemory = &FreeImage_OpenMemory;
```

示例代码仅引用了`FI_OpenMemory()`，因此对于 Android 和 Windows 来说是一样的。

# Curl

libcurl 库[`curl.haxx.se/libcurl`](http://curl.haxx.se/libcurl)是一个免费且易于使用的客户端 URL 传输库。它是处理众多网络协议的本机应用程序的实际标准。Linux 和 OS X 用户可以在他们的系统上享受这个库，并且可以使用`-lcurl`开关与之链接。在 Windows 主机上为 Android 编译 libcurl 需要执行一些额外的步骤，我们在这里解释这些步骤。

libcurl 库的构建过程基于`autoconf`；在实际构建库之前，我们需要生成`curl_config.h`文件。从包含未打包的 libcurl 发行包的文件夹中运行配置脚本。交叉编译命令行标志应设置为：

```java
--host=arm-linux CC=arm-eabi-gcc

```

`CPPFLAGS`变量的`-I`参数应指向 NDK 文件夹中的`/system/core/include`子文件夹，在我们的例子中：

```java
CPPFLAGS="-I D:/NDK/system/core/include"

```

libcurl 库可以通过多种方式进行定制。我们使用以下这组参数（除了 HTTP 和 HTTPS 之外禁用所有协议）：

```java
>configure CC=arm-eabi-gcc --host=arm-linux --disable-tftp --disable-sspi --disable-ipv6 --disable-ldaps --disable-ldap --disable-telnet --disable-pop3 --disable-ftp --without-ssl --disable-imap --disable-smtp --disable-pop3 --disable-rtsp --disable-ares --without-ca-bundle --disable-warnings --disable-manual --without-nss --enable-shared --without-zlib --without-random --enable-threaded-resolver --with-ssl

```

`--with-ssl`参数允许使用 OpenSSL 库来提供安全的 HTTPS 传输。这个库将在本章进一步讨论。然而，为了处理 SSL 加密连接，我们需要告诉 libcurl 我们的系统证书位于何处。这可以在`curl_config.h`文件开头通过定义`CURL_CA_BUNDLE`来完成：

```java
#define CURL_CA_BUNDLE "/etc/ssl/certs/ca-certificates.crt"

```

配置脚本将生成一个有效的`curl_config.h`头文件。你可以在书的源代码包中找到它。编译 Android 静态库需要一个通常的`Android.mk`和`Application.mk`文件集，这也包含在`1_Curl`示例中。在下一章，我们将学习如何使用 libcurl 库通过 HTTPS 从互联网下载实际内容。然而，以下是一个简化使用示例来检索 HTTP 页面：

```java
CURL* Curl = curl_easy_init();
curl_easy_setopt( Curl, CURLOPT_URL, "http://www.google.com" );
curl_easy_setopt( Curl, CURLOPT_FOLLOWLOCATION, 1 );
curl_easy_setopt( Curl, CURLOPT_FAILONERROR, true );
curl_easy_setopt( Curl, CURLOPT_WRITEFUNCTION, &MemoryCallback );
curl_easy_setopt( Curl, CURLOPT_WRITEDATA, 0 );
curl_easy_perform( Curl );
curl_easy_cleanup( Curl );
```

在这里`MemoryCallback()`是一个处理接收到的数据的函数。它可以小到像下面的代码片段：

```java
size_t MemoryCallback( void* P, size_t Size, size_t Num, void* )
{
  if ( !P ) return 0;
  printf( "%s\n", P );
}
```

检索到的数据将在你的桌面应用程序上显示在屏幕上。同样的代码在 Android 中会像哑巴一样工作，不会产生任何可见的副作用，因为`printf()`函数在那里只是一个占位符。

# OpenSSL

OpenSSL 是一个开源库，实现了安全套接字层（SSL v2/v3）和传输层安全（TLS）协议，以及一个功能强大的通用加密库。可以在[`www.openssl.org`](https://www.openssl.org)找到它。

在这里，我们将构建 OpenSSL 版本 1.0.1j，其中包含对 Heartbleed 漏洞的修复([`heartbleed.com`](http://heartbleed.com))。

Heartbleed 漏洞是流行的 OpenSSL 加密软件库中一个严重的安全漏洞。这个弱点使得在正常情况下受 SSL/TLS 加密保护的信息可以被窃取，而这种加密被用于确保互联网的安全。

如果你尝试将应用程序静态链接到一个旧版本的 OpenSSL，并在 Google Play 上发布，你可能会看到以下安全警报：

![OpenSSL](img/image00216.jpeg)

到这本书出版时，即使是 OpenSSL 的 1.0.0j 版本也可能已经过时了。因此，下载最新的源代码并相应地更新 NDK Makefile 对你来说将是一个很好的练习。以下是如何进行的一个简要概述。

OpenSSL 被编译为两个相互协作的静态库：`libssl`和`libcrypto`。查看源代码包，并查看文件夹`2_OpenSSL/lib/crypto/jni`和`2_OpenSSL/ssl/jni`。这两个库都应该被链接到使用带有 SSL 功能的 libcurl 版本的应用程序中。

通常，为此准备的`Android.mk`文件可以像下面的列表这样开始：

```java
include $(CLEAR_VARS)
LOCAL_MODULE := libCurl
LOCAL_SRC_FILES := ../../../Libs.Android/libcurl.$(TARGET_ARCH_ABI).a
include $(PREBUILT_STATIC_LIBRARY)
include $(CLEAR_VARS)
LOCAL_MODULE := libCrypto
LOCAL_SRC_FILES := ../../../Libs.Android/libCrypto.$(TARGET_ARCH_ABI).a
include $(PREBUILT_STATIC_LIBRARY)
include $(CLEAR_VARS)
LOCAL_MODULE := libSSL
LOCAL_SRC_FILES := ../../../Libs.Android/libSSL.$(TARGET_ARCH_ABI).a
include $(PREBUILT_STATIC_LIBRARY)
```

在这个文件的最后，只需链接所有的库：

```java
LOCAL_STATIC_LIBRARIES += libCurl
LOCAL_STATIC_LIBRARIES += libSSL
LOCAL_STATIC_LIBRARIES += libCrypto
```

到此为止，你现在可以处理 SSL 连接了。

# FreeImage

FreeImage 是一个流行的位图操作库，Unity 游戏引擎是该库的用户之一（[`freeimage.sourceforge.net/users.html`](http://freeimage.sourceforge.net/users.html)）。该库是 `libpng`、`libjpeg`、`libtiff` 等之上的全功能封装，提供了快速图像加载例程，无需回退到 Java 代码。

FreeImage 包含一套完整的 Makefiles，适用于不同的平台。按照 *处理预编译静态库* 部分的说明，编译 Android 的库非常直接。`Application.mk` 文件与 Curl 的同名文件在一行上有所不同：

```java
APP_MODULES := FreeImage
```

在 `Android.mk` 文件中，我们将更改 C 编译标志：

```java
GLOBAL_CFLAGS   := -O3 -DHAVE_CONFIG_H=1 -DFREEIMAGE_LIB -DDISABLE_PERF_MEASUREMENT
```

在以下示例中，我们将实现两个简单的例程，以在各种文件格式中从内存块加载和保存图像。

我们从 `FreeImage_LoadFromMemory()` 例程开始，它接收 `Data` 数组和其 `Size` 作为输入参数，并将这个数组解码成一个包含位图像素的 `std::vector<char>`。尺寸信息，宽度和高度，存储在 `W` 和 `H` 参数中。颜色深度信息被放入 `BitsPerPixel` 参数中。可选的 `DoFlipV` 参数指示代码垂直翻转加载的图像，这在与不同图形 API 的图像存储约定（从上到下或从下到上）打交道时可能需要：

```java
bool FreeImage_LoadFromStream( void* Data,unsigned int Size,
  std::vector<ubyte>& OutData,int& W,
  int& H,int& BitsPerPixel,bool DoFlipV )
{
```

我们创建内部内存块，它可以被 FreeImage 例程读取。

```java
  FIMEMORY* Mem = FI_OpenMemory(( unsigned char* )Data,
    static_cast<unsigned int>( Size ) 
  );
```

在读取位图之前，我们将以以下方式检测其格式（例如，`.jpg`、`.bmp`、`.png` 等）：

```java
  FREE_IMAGE_FORMAT FIF = FI_GetFileTypeFromMemory( Mem, 0 );
```

然后，解码后的位图被读取到临时的 `FIBITMAP` 结构中：

```java
  FIBITMAP* Bitmap = FI_LoadFromMemory( FIF, Mem, 0 );
  FI_CloseMemory( Mem );
  FIBITMAP* ConvBitmap;
```

如果总位数超过 32 位，例如，每个颜色通道占用超过 8 位，我们很可能处理的是浮点数图像，这将需要一些额外的处理：

```java
  bool FloatFormat = FI_GetBPP( Bitmap ) > 32;
  if ( FloatFormat )
  {
```

本书并未广泛使用浮点数图像，但了解 FreeImage 支持 OpenEXR 格式下的高动态范围图像是有用的。

### 注意

OpenEXR 格式以支持每个通道 16 位的浮点数值而著称，并可用于游戏中存储不同 HDR 效果的纹理。

```java
    ConvBitmap = FI_ConvertToRGBF( Bitmap );
  }
  else
  {
```

使用透明度信息来转换图像。如果图像不是透明的，则忽略 alpha 通道：

```java
    ConvBitmap = FI_IsTransparent( Bitmap ) ? FI_ConvertTo32Bits( Bitmap ) : FI_ConvertTo24Bits( Bitmap );
  }
  FI_Unload( Bitmap );
  Bitmap = ConvBitmap;
```

如有必要，我们以下列方式对图像进行垂直翻转：

```java
  if ( DoFlipV ) FI_FlipVertical( Bitmap );
```

提取图像尺寸和颜色信息：

```java
  W = FI_GetWidth( Bitmap );
  H = FI_GetHeight( Bitmap );
  BitsPP = FI_GetBPP( Bitmap );
```

一旦我们知道尺寸，我们可以调整输出缓冲区的大小，如下所示：

```java
  OutData.resize( W * H * ( BitsPerPixel / 8 ) );
```

最后，我们可以将原始未对齐的位图数据提取到我们的 `OutData` 向量中。每行紧密排列的数据大小为 `W*BitsPP/8` 字节：

```java
  FI_ConvertToRawBits( &OutData[0],Bitmap, W * BitsPP / 8, BitsPP, 0, 1, 2, false );
```

临时位图对象被删除，函数优雅地返回：

```java
  FI_Unload( Bitmap );
  return true;
}
```

位图保存例程可以以类似的方式实现。首先，我们在 FreeImage 库中分配 FIBITMAP 结构来表示我们的图像：

```java
bool FreeImage_SaveToMemory( const std::string& Ext,ubyte* RawBGRImage,int Width,int Height,int BitsPP,std::vector<ubyte>& OutData )
{
  FIBITMAP* Bitmap = FI_Allocate(Width, Height, BitsPP, 0, 0, 0);
```

原始位图数据被复制到 FIBITMAP 结构中：

```java
  memcpy( FI_GetBits( Bitmap ), RawBGRImage, Width * Height * BitsPP / 8 );
```

FreeImage 使用倒置的垂直扫描线顺序，因此在保存之前我们应该垂直翻转图像：

```java
  FI_FlipVertical( Bitmap );
```

然后，我们将使用用户指定的文件扩展名来检测输出图像的格式：

```java
  int OutSubFormat;
  FREE_IMAGE_FORMAT OutFormat;
  FileExtToFreeImageFormats( Ext, OutSubFormat, OutFormat );
```

为了保存图像，我们将分配一个动态内存块：

```java
  FIMEMORY* Mem = FI_OpenMemory( nullptr, 0);
```

`FI_SaveToMemory()`调用根据选定的格式将我们的原始位图编码成压缩表示形式：

```java
  if ( !FI_SaveToMemory( OutFormat,Bitmap, Mem, OutSubFormat ) )
  {
    return false;
  }
```

编码后，我们将直接访问 FreeImage 内存块：

```java
  ubyte* Data = NULL;
  uint32_t Size = 0;
  FI_AcquireMemory( Mem, &Data, &Size );
```

然后，我们将字节复制到我们的`OutData`向量中：

```java
  OutData.resize( Size );
  memcpy( &OutData[0], Data, Size );
```

需要进行一些清理。我们删除内存块和 FIBITMAP 结构：

```java
  FI_CloseMemory( Mem );
  FI_Unload( Bitmap );
  return true;
}
```

辅助的`FileExtToFreeImageFormats()`函数将文件扩展名转换为内部的 FreeImage 格式说明符，并提供多个选项。代码很直观。我们将提供的文件扩展名与多个预定义值进行比较，并填充`FIF_FORMAT`和`SAVE_OPTIONS`结构：

```java
static void FileExtToFreeImageFormats( std::string Ext,int& OutSubFormat, FREE_IMAGE_FORMAT& OutFormat )
{
  OutSubFormat = TIFF_LZW;
  OutFormat = FIF_TIFF; std::for_each( Ext.begin(), Ext.end(),[]( char& in )
  { 
    in = ::toupper( in );
  } 
  );
  if ( Ext == ".PNG" )
  {
    OutFormat = FIF_PNG;
    OutSubFormat = PNG_DEFAULT;
  }
  else if ( Ext == ".BMP" )
  {
    OutFormat = FIF_BMP;
    OutSubFormat = BMP_DEFAULT;
  }
  else if ( Ext == ".JPG" )
  {
    OutFormat = FIF_JPEG;
    OutSubFormat = JPEG_QUALITYSUPERB | JPEG_BASELINE |JPEG_PROGRESSIVE | JPEG_OPTIMIZE;
  }
  else if ( Ext == ".EXR" )
  {
    OutFormat = FIF_EXR;
    OutSubFormat = EXR_FLOAT;
  }
}
```

这可以根据您的需要进行扩展和自定义。

# 加载和保存图像

为了使前面的代码可用，我们添加了两个更多例程，它们从磁盘文件中保存和加载图像。第一个，`FreeImage_LoadBitmapFromFile()`，加载位图：

```java
bool FreeImage_LoadBitmapFromFile( const std::string& FileName, std::vector<ubyte>& OutData, int& W, int& H, int& BitsPP )
{
  std::ifstream InFile( FileName.c_str(),
  std::ios::in | std::ifstream::binary );
  std::vector<char> Data(
    ( std::istreambuf_iterator<char>( InFile ) ), std::istreambuf_iterator<char>() );
  return FreeImage_LoadFromStream(
    ( ubyte* )&Data[0], ( int )data.size(),
    OutData, W, H, BitsPP, true );
}
```

我们使用一个简单的函数来提取文件扩展名，它作为文件类型标签：

```java
std::string ExtractExtension( const std::string& FileName )
{
  size_t pos = FileName.find_last_of( '.' );
  return ( pos == std::string::npos ) ?
    FileName : FileName.substr( pos );
}
```

`FreeImage_SaveBitmapToFile()`函数使用标准的`std::ofstream`流保存文件：

```java
bool FreeImage_SaveBitmapToFile( const std::string& FileName, ubyte* ImageData, int W, int H, int BitsPP )
{
  std::string Ext = ExtractExtension( FileName );
  std::vector<ubyte> OutData;
  if ( !FreeImage_SaveToMemory( Ext, ImageData, W, H, BitsPP, OutData ) )
  {
    return false;
  }
  std::ofstream OutFile( FileName.c_str(),
  std::ios::out | std::ofstream::binary );
  std::copy( OutData.begin(), OutData.end(), std::ostreambuf_iterator<char>( OutFile ) );
  return true;
}
```

这段代码足以涵盖图像加载库的所有基本使用情况。

# FreeType

FreeType 库是一个事实上的标准，用于使用 TrueType 字体渲染高质量文本。由于在几乎任何图形程序中输出文本都是不可避免的，我们给出一个如何使用从等宽 TrueType 文件生成的固定大小字体来渲染文本字符串的例子。

我们将固定大小字体存储在`16x16`网格中。此演示应用程序的源字体名为`Receptional Receipt`，从[`1001freefonts.com`](http://1001freefonts.com)下载。以下图像显示了结果`16x16`网格的四行：

![FreeType](img/image00217.jpeg)

单个字符占用一个矩形区域，我们将这个区域称为*槽*。字符矩形的坐标是使用字符的 ASCII 码计算的。网格中的每个槽占用`SlotW x SlotH`像素，字符本身居中，大小为`CharW x CharH`像素。为了演示，我们简单假设`SlotW`是`CharW`大小的两倍：

![FreeType](img/image00218.jpeg)

我们限制自己使用最简单的可能使用场景：8 位 ASCII 字符，固定大小的字符字形。为了渲染字符串，我们将遍历其字符并调用尚未编写的`RenderChar()`函数：

```java
void RenderStr( const std::string& Str, int x, int y )
{
  for ( auto c: Str )
  {
    RenderChar( c, x, y );
    x += CharW;
  }
}
```

字符渲染例程是一个简单的双循环，将字形像素复制到输出图像中：

```java
void RenderChar( char c, int x, int y )
{
  int u = ( c % 16 ) * SlotW;
  int v = ( c / 16 ) * SlotH;
  for ( int y1 = 0 ; y1 < CharH ; y1++ )
    for ( int x1 = 0 ; x1 <= CharW ; x1++ )
      PutPixel( g_OutBitmap, W, H,
        x + x1, y + y1,
        GetPixel( Font, FontW, FontH,
          x1 + u + CharW, y1 + v)
      );
}
```

`PutPixel()`和`GetPixel()`例程分别设置和获取位图中的像素。每个像素都是 24 位 RGB 格式：

```java
int GetPixel( const std::vector<unsigned char>& Bitmap, int W, int H, int x, int y )
{
  if ( y >= H || x >= W || y < 0 || x < 0 ) { return 0; }
```

在这里，假设扫描线的宽度等于图像宽度，RGB 三元组的颜色分量数量为 3：

```java
  int Ofs = ( y * W + x ) * 3;
```

使用位运算移位来构建结果的 RGB 值：

```java
  return (Bitmap[Ofs+0] << 16) +
    (Bitmap[Ofs+1] <<  8) +
    (Bitmap[Ofs+2]);
}
```

```java
void PutPixel( std::vector<unsigned char>& Bitmap,int W, int H, int x, int y, int Color )
{
  if ( y < 0 || x < 0 || y > H - 1 || x > W - 1 ) { return; }
  int Ofs = ( y * W + x ) * 3;
```

位运算移位和掩码完成了提取工作：

```java
  buffer[Ofs + 0] = ( Color ) & 0xFF;
  buffer[Ofs + 1] = ( Color >> 8 ) & 0xFF;
  buffer[Ofs + 2] = ( Color >> 16 ) & 0xFF;
}
```

另外还有一个辅助函数`Greyscale()`，它使用位运算移位为给定的强度计算 RGB 灰度颜色：

```java
inline int Greyscale( unsigned char c )
{
  return ( (255-c) << 16 ) + ( (255-c) << 8 ) + (255-c);
}
```

对于前面的代码，我们并不需要 FreeType。我们真正只需要该库来生成字体。我们将加载字体数据文件，为其前 256 个字符渲染字形，然后使用生成的字体位图来渲染文本字符串。代码的第一部分生成字体。我们将使用几个变量来存储字体的尺寸：

```java
/// Horizontal size of the character
const int CharW = 32;
const int CharH = 64;
/// Horizontal size of the character slot
const int SlotW = CharW * 2;
const int SlotH = CharH;
const int FontW = 16 * SlotW;
const int FontH = 16 * SlotH;
std::vector<unsigned char> g_FontBitmap;
```

我们将字体存储在一个标准向量中，可以传递给`TestFontRendering()`例程：

```java
void TestFontRendering( const std::vector<char>& Data )
{
  LoadFreeImage();
  LoadFreeType();
  FT_Library Library;
  FT_Init_FreeTypePTR( &Library );
  FT_Face Face;
  FT_New_Memory_FacePTR( Library,
    (const FT_Byte*)Data.data(),
    (int)Data.size(), 0, &face );
```

将字符大小固定在 100 dpi：

```java
  FT_Set_Char_SizePTR( Face, CharW * 64, 0, 100, 0 );
  g_FontBitmap.resize( FontW * FontH * 3 );
  std::fill( std::begin(g_FontBitmap), std::end(g_FontBitmap), 0xFF );
```

我们将在循环中逐个渲染 256 个 ASCII 字符：

```java
  for ( int n = 0; n < 256; n++ )
  {
```

将字形图像加载到槽中：

```java
    if ( FT_Load_CharPTR( Face, n , FT_LOAD_RENDER ) )
      continue;
    FT_GlyphSlot Slot = Face->glyph;
    FT_Bitmap Bitmap = Slot->bitmap;
```

计算每个字符的矩形左上角的坐标：

```java
    int x = (n % 16) * SlotW + CharW + Slot->bitmap_left;
    int y = (n / 16) * SlotH - Slot->bitmap_top + 3*CharH/4;
```

字符的笔形被复制到`g_FontBitmap`位图中：

```java
    for ( int i = 0 ; i < ( int )Bitmap.width; i++ )
    for ( int j = 0 ; j < ( int )Bitmap.rows; j++ )
    PutPixel( g_FontBitmap, FontW, FontH,i + x, j + y,
      Greyscale( Bitmap.buffer[j * Bitmap.width + i])
    );
  }
```

我们将生成的`Font`位图保存到文件中：

```java
  FreeImage_SaveBitmapToFile( "test_font.png",
    g_FontBitmap.data(), FontW, FontH, 24 );
```

在字体位图生成结束时，我们将清除与 FreeType 库相关的所有内容：

```java
  FT_Done_FacePTR    ( Face );
  FT_Done_FreeTypePTR( Library );
```

为了使用我们的等宽字体，我们将声明字符串，计算其在屏幕像素中的宽度，并分配输出位图：

```java
  std::string Str = "Test string";
  W = Str.length() * CharW;
  H = CharH;
  g_OutBitmap.resize( W * H * 3 );
  std::fill( std::begin(g_OutBitmap), std::end(g_OutBitmap), 0xFF );
```

`TestFontRendering()`例程的末尾只是调用了`RenderStr()`：

```java
  RenderStr( Str, 0, 0 );
```

然后将生成的图像保存到文件中：

```java
  FreeImage_SaveBitmapToFile( "test_str.png",
    g_OutBitmap.data(), W, H, 24 );
}
```

结果应该看起来像以下图像：

![FreeType](img/image00219.jpeg)

通常在位图字体渲染方面，你不想自己编写位图生成的代码。建议您使用第三方工具来完成这项工作。这样一款免费工具是 AngelCode，可以在[`www.angelcode.com/products/bmfont`](http://www.angelcode.com/products/bmfont)找到。它可以以最优的方式将字形打包到位图中，并生成处理生成的位图所需的数据。

# Theora（注：此处 Theora 为一种视频压缩格式的名称，不翻译）

Theora 是来自 Xiph.Org 基金会的一个免费且开源的视频压缩格式。与我们的所有多媒体技术一样，它可以用来在线和光盘上分发电影和视频，而无需像许多其他视频格式那样支付许可和版税费用，或受到任何其他供应商的锁定。它可以在[`www.theora.org`](http://www.theora.org)获取。

为了避免混淆，我们将介绍一些术语。我们所说的**比特流**是指一些字节的序列。逻辑比特流是对视频或音频数据的某种表示。**编解码器**，或编码器-解码器，是一组将逻辑比特流编码和解码成一组名为打包比特流的紧凑表示的函数。由于通常的多媒体数据包含多个逻辑比特流，紧凑表示必须被分割成小块，这些小块被称为包。每个**包**都有一个特定的尺寸、时间戳和与之相关的校验和，以保证包的完整性。比特流和包的方案在以下图像中显示：

![Theora](img/image00220.jpeg)

逻辑包和打包比特流的包相互混合，形成一个线性序列，保持每个独立比特流的包的顺序。这称为复用。Ogg 库读取`.ogg`文件并将其分割成打包比特流。每个比特流都可以使用 Theora、Vorbis 或其他解码器进行解码。

### 注意

在我们之前的书籍中，*Android NDK Game Development Cookbook*，*Packt Publishing* ([`www.packtpub.com/game-development/android-ndk-game-development-cookbook`](https://www.packtpub.com/game-development/android-ndk-game-development-cookbook))，我们通过示例教大家如何解码 Ogg Vorbis 音频流。

在本章中，我们只解决了从文件中提取媒体信息的最简单问题。即使这个简单的操作的代码可能看起来又长又复杂。然而，它可以用不到十个步骤来描述：

1.  初始化 OGG 流读取器。

1.  开始一个包构建循环：从源文件中读取一堆字节。

1.  检查是否有足够的数据来生成另一个逻辑包。

1.  如果形成了新的包，检查它是否是`BoS`（流开始）包。

1.  尝试使用`BoS`包初始化 Theora 或 Vorbis 解码器。

1.  如果我们没有足够的音频和视频流来解码，请转到步骤 2。

1.  如果我们没有足够的流信息，继续读取次要流包。

1.  初始化 Theora 解码器并提取视频帧信息。

### 注意

Ogg 流还有一个复杂性级别，因为包被分组形成逻辑页。在前面的伪代码中，我们指的是实际上是页面的包。尽管如此，方案保持不变：读取字节，直到有足够的数据让解码器生成另一个视频帧，或者在我们的情况下，读取视频信息。

我们使用标准的 C++ I/O 流并实现了三个简单的函数：`Stream_Read()`、`Stream_Seek()`和`Stream_Size()`。在后面的第四章，*组织虚拟文件系统*中，我们将使用自己的 I/O 抽象层重新实现这些方法。让我们打开文件流：

```java
std::ifstream Input( "test.ogv", std::ios::binary );
```

这是一个从输入流中读取指定字节数的函数：

```java
int Stream_Read( char* OutBuffer, int Size )
{
  Input.read( OutBuffer, Size );
  return Input.gcount();
}
```

使用以下代码寻找指定位置：

```java
int Stream_Seek( int Offset )
{
  Input.seekg( Offset );
  return (int)Input.tellg();
}
```

要确定文件大小，请使用以下代码：

```java
int Stream_Size()
{
  Input.seekg (0, input.end);
  int Length = Input.tellg();
  Input.seekg( 0, Input.beg );
  return Length;
}
```

首先，应该声明一些变量来存储解码过程的状态、同步对象、当前页面以及音频和视频流：

```java
ogg_sync_state   OggSyncState;
ogg_page         OggPage;
ogg_stream_state VorbisStreamState;
ogg_stream_state TheoraStreamState;
```

Theora 解码器状态：

```java
th_info          TheoraInfo;
th_comment       TheoraComment;
th_setup_info*   TheoraSetup;
th_dec_ctx*      TheoraDecoder;
```

Vorbis 解码器状态：

```java
vorbis_info      VorbisInfo;
vorbis_dsp_state VorbisDSPState;
vorbis_comment   VorbisComment;
vorbis_block     VorbisBlock;
```

函数`Theora_Load()`读取文件头并从中提取视频帧信息：

```java
bool Theora_Load()
{
  Stream_Seek( 0 );
```

当前的 Ogg 包将被读取到`TempOggPacket`结构中：

```java
  ogg_packet TempOggPacket;
```

需要对一些简单但必要的状态变量进行初始化：

```java
  memset( &VorbisStreamState, 0, sizeof( ogg_stream_state ) );
  memset( &TheoraStreamState, 0, sizeof( ogg_stream_state ) );
  memset( &OggSyncState,   0, sizeof( ogg_sync_state ) );
  memset( &OggPage,        0, sizeof( ogg_page ) );
  memset( &TheoraInfo,     0, sizeof( th_info ) );
  memset( &TheoraComment,  0, sizeof( th_comment ) );
  memset( &VorbisInfo,     0, sizeof( vorbis_info ) );
  memset( &VorbisDSPState, 0, sizeof( vorbis_dsp_state ) );
  memset( &VorbisBlock,    0, sizeof( vorbis_block ) );
  memset( &VorbisComment,  0, sizeof( vorbis_comment ) );
  OGG_sync_init   ( &OggSyncState );
  TH_comment_init ( &TheoraComment );
  TH_info_init    ( &TheoraInfo );
  VORBIS_info_init( &VorbisInfo );
  VORBIS_comment_init( &VorbisComment );
```

我们开始读取文件，并使用`Done`标志在文件结束或我们有足够的数据获取信息时终止：

```java
  bool Done = false;
  while ( !Done )
  {
    char* Buffer = OGG_sync_buffer( &OggSyncState, 4096 );
    int BytesRead = ( int )Stream_Read( Buffer, 4096 );
    OGG_sync_wrote( &OggSyncState, BytesRead );
    if ( BytesRead == 0 )
    {
      break;
    }
    while (OGG_sync_pageout( &OggSyncState, &OggPage ) > 0)
    {
```

当我们最终遇到一个完整的包时，我们将检查它是否是`BOS`标记，并将数据输出到其中一个解码器：

```java
      ogg_stream_state OggStateTest;
      if ( !OGG_page_bos( &OggPage ) )
      {
        if ( NumTheoraStreams > 0 )
        {
          OGG_stream_pagein( &TheoraStreamState, &OggPage );
        }
        if ( NumVorbisStreams > 0 )
          {
            OGG_stream_pagein( VorbisStreamState, &OggPage );
          }
          Done = true;
          break;
        }
        OGG_stream_init( &OggStateTest,
        OGG_page_serialno( &OggPage ) );
        OGG_stream_pagein( &OggStateTest, &OggPage );
        OGG_stream_packetout( &OggStateTest, &TempOggPacket );
```

我们将使用两个变量`NumTheoraStreams`和`NumVorbisStreams`分别计算视频和音频流的数量。在以下几行中，我们将 Ogg 包提供给两个解码器，并查看解码器是否对此有异议：

```java
        if ( NumTheoraStreams == 0 )
        {
          int Ret = TH_decode_headerin( &TheoraInfo, &TheoraComment, &TheoraSetup, &TempOggPacket );
          if ( Ret > 0 )
          {
```

下面是 Theora 头信息：

```java
            memcpy( &TheoraStreamState, &OggStateTest, sizeof( OggStateTest ) );
            NumTheoraStreams = 1;
            continue;
          }
        }
        if ( NumVorbisStreams == 0 )
        {
          int Ret = VORBIS_synthesis_headerin( &VorbisInfo, &VorbisComment, &TempOggPacket );
          if ( Ret >= 0 )
          {
```

这是 Vorbis 头：

```java
            memcpy( &VorbisStreamState, &OggStateTest, sizeof( OggStateTest ) );
            NumVorbisStreams = 1;
            continue;
          }
        }
```

因为我们只需要 Theora 流信息，所以忽略其他编解码器并丢弃头信息：

```java
        OGG_stream_clear( &OggStateTest );
      }
    }
```

之前的代码基本上只是计算了流的数量，现在我们应该已经完成了。如果流的数量仍然不足，我们将继续读取并检查次级流头：

```java
    while((( NumTheoraStreams > 0 ) && ( NumTheoraStreams < 3 )) || (( NumVorbisStreams > 0 ) && ( NumVorbisStreams < 3 )))
    {
      int Success = 0;
```

我们将读取所有可用的包，并检查它是否是一个新的 Theora 流的开始：

```java
      while (( NumTheoraStreams > 0 ) &&
        ( NumTheoraStreams < 3 ) &&
        ( Success = OGG_stream_packetout( &TheoraStreamState, &TempOggPacket ) ) )
      {
        if ( Success < 0 ) return false;
        if ( !TH_decode_headerin( &TheoraInfo, &TheoraComment, &TheoraSetup, &TempOggPacket ) ) return false;
        ++NumTheoraStreams;
      }
```

同样的方法，我们将寻找下一个 Vorbis 流的开始：

```java
      while ( NumVorbisStreams < 3 && ( Success = OGG_stream_packetout( &VorbisStreamState, &TempOggPacket ) ) )
      {
        if ( Success < 0 ) return false;
        if ( VORBIS_synthesis_headerin( &VorbisInfo, &VorbisComment, &TempOggPacket ) )
        return false;
        ++NumVorbisStreams;
      }
```

`while (!Done)`循环的最后一步是检查具有实际帧数据的包，或者如果下一个包不可用，从流中读取更多字节：

```java
      if ( OGG_sync_pageout( &OggSyncState, &OggPage ) > 0 )
      {
        if ( NumTheoraStreams > 0 )
        {
          OGG_stream_pagein( &TheoraStreamState, &OggPage );
        }
        if ( NumVorbisStreams > 0 )
        {
          OGG_stream_pagein( &VorbisStreamState, &OggPage );
        }
      }
      else
      {
        char* Buffer = OGG_sync_buffer( &OggSyncState, 4096 );
        int BytesRead = (int)Stream_Read( Buffer, 4096 );
        OGG_sync_wrote( &OggSyncState, BytesRead );
        if ( BytesRead == 0 ) return false;
      }
    }
```

到目前为止，我们已经找到了所有的流头，并准备好初始化 Theora 解码器。初始化后，我们获取帧宽和帧高：

```java
    TheoraDecoder = TH_decode_alloc( &TheoraInfo, TheoraSetup );
    Width  = TheoraInfo.frame_width;
    Height = TheoraInfo.frame_height;
    return true;
  }
```

最后，我们清除编解码器的内部结构以避免内存泄漏：

```java
  void Theora_Cleanup()
  {
    if ( TheoraDecoder )
    {
      TH_decode_free( TheoraDecoder );
      TH_setup_free( TheoraSetup );
      VORBIS_dsp_clear( &VorbisDSPState );
      VORBIS_block_clear( &VorbisBlock );
      OGG_stream_clear( &TheoraStreamState );
      TH_comment_clear( &TheoraComment );
      TH_info_clear( &TheoraInfo );
      OGG_stream_clear( &VorbisStreamState );
      VORBIS_comment_clear( &VorbisComment );
      VORBIS_info_clear( &VorbisInfo );
      OGG_sync_clear( &OggSyncState );
    }
  }
```

到此为止，我们已经读取了视频参数。在接下来的章节中，一旦我们有了基本的图形和音频渲染能力，我们将回到音频和视频的解码和播放。

代码更为复杂，但与我们的示例非常相似，它被广泛用于`LibTheoraPlayer`库源代码中，该代码可在[`libtheoraplayer.cateia.com`](http://libtheoraplayer.cateia.com)获取。

在本章的示例中，我们将使用大写的函数名称来区分动态库使用和静态链接。如果您想静态链接`ogg`、`vorbis`和`theora`库，可以通过将每个`OGG`函数前缀重命名为`ogg`来实现。就是这样，只需将大写字母替换为小写字母。

对于示例 Theora 视频内容，我们将参考官方网站，[`www.theora.org/content`](http://www.theora.org/content)，您可以在那里下载`.ogv`文件。

# OpenAL

OpenAL 是一个跨平台的音频 API。它旨在高效地渲染多通道三维定位音频，并在许多桌面平台的众多游戏引擎和应用程序中广泛使用。许多移动平台提供了不同的音频 API，例如，OpenSL ES 是一个强有力的竞争者。但是，当可移植性受到威胁时，我们应该选择一个能够在所有所需平台上运行的 API。OpenAL 在 Windows、Linux、OS X、Android、iOS、BlackBerry 10 以及许多其他平台上都有实现。在所有这些操作系统中，除了 Windows 和 Android，OpenAL 都是一等公民，所有库在系统中都可用。在 Windows 上，有一个来自 Creative 的实现。在 Android 上，我们需要自己构建库。我们将使用 Martins Mozeiko 的移植版本[`pielot.org/2010/12/14/openal-on-android/`](http://pielot.org/2010/12/14/openal-on-android/)。这个库可以通过对`Android.mk`和`Application.mk`文件进行少量调整来编译为 Android 版本。以下是`Android.mk`文件：

```java
TARGET_PLATFORM := android-19
LOCAL_PATH := $(call my-dir)
include $(CLEAR_VARS)
LOCAL_ARM_MODE := arm
LOCAL_MODULE := OpenAL
LOCAL_C_INCLUDES := $(LOCAL_PATH) $(LOCAL_PATH)/../include $(LOCAL_PATH)/../OpenAL32/Include
LOCAL_SRC_FILES  := ../OpenAL32/alAuxEffectSlot.c \
                    ../OpenAL32/alBuffer.c \
                    ../OpenAL32/alDatabuffer.c \
                    ../OpenAL32/alEffect.c \
                    ../OpenAL32/alError.c \
                    ../OpenAL32/alExtension.c \
                    ../OpenAL32/alFilter.c \
                    ../OpenAL32/alListener.c \
                    ../OpenAL32/alSource.c \
                    ../OpenAL32/alState.c \
                    ../OpenAL32/alThunk.c \
                    ../Alc/ALc.c \
                    ../Alc/alcConfig.c \
                    ../Alc/alcEcho.c \
                    ../Alc/alcModulator.c \
                    ../Alc/alcReverb.c \
                    ../Alc/alcRing.c \
                    ../Alc/alcThread.c \
                    ../Alc/ALu.c \
                    ../Alc/android.c \
                    ../Alc/bs2b.c \
                    ../Alc/null.c
```

`-D`定义是正确编译所需的：

```java
GLOBAL_CFLAGS := -O3 -DAL_BUILD_LIBRARY -DAL_ALEXT_PROTOTYPES -DHAVE_ANDROID=1
```

此`if`块是一种在您想要为 Android 构建 x86 版本的库时，区分 ARM 和 x86 编译器开关的方法：

```java
ifeq ($(TARGET_ARCH),x86)
  LOCAL_CFLAGS := $(GLOBAL_CFLAGS)
else
  LOCAL_CFLAGS := -mfpu=vfp -mfloat-abi=hard -mhard-float -fno-short-enums -D_NDK_MATH_NO_SOFTFP=1 $(GLOBAL_CFLAGS)
endif
include $(BUILD_STATIC_LIBRARY)
```

`Application.mk`文件是标准的，如下所示：

```java
APP_OPTIM := release
APP_PLATFORM := android-19
APP_STL := gnustl_static
APP_CPPFLAGS += -frtti
APP_CPPFLAGS += -fexceptions
APP_CPPFLAGS += -DANDROID
APP_MODULES := OpenAL
APP_ABI := armeabi-v7a-hard x86
NDK_TOOLCHAIN_VERSION := clang
```

为了方便您，我们在`6_OpenAL`示例中提供了所有的源代码和配置文件。此外，本书中使用的所有库都已为 Android 预编译，您可以在本书源代码包中的`Libs.Android`文件夹中找到它们。

# 将库链接到您的应用程序

在我们继续讨论更多主题之前，本章还有一件事需要讨论。实际上，我们学习了如何构建库，但还没学习如何将您的 Android 应用程序与它们链接。为此，我们需要修改您的应用程序的`Android.mk`文件。让我们看看`3_FreeImage_Example`示例及其`Application.mk`。它以声明预构建的静态库指向二进制文件的声明开始：

```java
include $(CLEAR_VARS)
LOCAL_MODULE := libFreeImage
LOCAL_SRC_FILES :=../../../Libs.Android/libFreeImage.$(TARGET_ARCH_ABI).a
include $(PREBUILT_STATIC_LIBRARY)
```

在这里，我们在路径中使用`$(TARGET_ARCH_ABI)`变量，以透明地处理`armeabi-v7a-hard`和`x86`版本的库。您可以轻松地添加更多架构。

一旦声明了库，让我们将应用程序与其链接。看看`Application.mk`的底部：

```java
LOCAL_STATIC_LIBRARIES += FreeImage
include $(BUILD_SHARED_LIBRARY)
```

`LOCAL_STATIC_LIBRARIES`变量包含了所有必要的库。为了方便起见，您可以省略前缀`lib`。

# 概括

在本章中，我们学习了如何在 Android 上处理预编译的静态库，同样的方法也适用于 OS X 和 Linux，以及如何在 Windows 上进行动态链接，同时不破坏代码的多平台功能。我们学习了如何构建 `libcurl` 和 `OpenSSL`，这样你就可以从 C++ 代码中访问 SSL 连接。FreeImage 和 FreeType 的几个示例展示了如何加载和保存光栅字体图像。使用 libtheora 的示例相当全面；然而，结果却很谦虚，我们只是从视频文件中读取元信息。OpenAL 将作为我们音频子系统的基础框架。
