# 第十章. 使用扩展开发工具包（EDK）扩展 Marmalade

在上一章中，我们提到了 Marmalade 的**扩展开发工具包**（**EDK**）是如何通过使用标准的 Marmalade API 来向 Marmalade 应用程序添加未在其他方面公开的功能的一种可能方式。

在本章中，我们将探讨以下主题：

+   EDK 概述及其必要性

+   如何通过创建 Windows、iOS 和 Android 的 EDK 扩展来扩展 Marmalade 以支持读取陀螺仪信息

# 为什么 EDK 是必要的？

Marmalade SDK 通过提供一套 API，这些 API 位于每个平台特定 API 之上，实现了能够将一个代码库部署到多个平台上的神奇功能。

部署的应用程序可执行文件实际上由两个独立的文件组成。我们的应用程序代码被编译成**S3E 文件**，这是 Marmalade 的 Windows**动态链接库**（**DLL**）等价物。该文件在所有平台上都是相同的。

为了执行我们的 S3E 文件，我们使用一个加载程序。这个程序是我们运行的平台和我们的代码之间的粘合剂。加载程序首先启动，将 S3E 文件加载到内存中，然后将控制权传递给其中的代码。如果我们的代码需要执行平台相关的调用，它实际上会向加载程序中的一个函数发出请求，然后该函数会调用正确的操作系统函数。

加载程序是一个固定的实体，我们无法更改它，因此 Marmalade 为我们提供了 EDK 系统，使我们能够进行平台特定的函数调用。Marmalade SDK 的某些部分实际上就是这样实现的；例如，s3eFacebook API 实际上是一个扩展！

EDK 的唯一问题是它不是一个完全跨平台的解决方案。在撰写本文时，只能为 iOS、Android、Windows 和 Mac OSX 编写扩展。

### 注意

由于本书主要关注使用 Marmalade 的 Windows 版本进行开发，因此我们不会探讨如何构建 Mac 扩展，然而，由于我们不可避免地需要使用 Apple iOS SDK，因此我们需要访问 Mac 计算机来构建 iOS 扩展，因为 iOS SDK 作为 Windows 下载不可用。有关创建 Mac 扩展的详细信息，请参阅 Marmalade 文档，前往**Marmalade (C++)** | **扩展开发工具包 (EDK)** | **按平台划分的 EDK 指南** | **OS X EDK 指南**。

# 创建用于陀螺仪输入的扩展

为了说明创建 Marmalade 扩展的过程，我们将查看如何添加对陀螺仪输入的支持。这是一个有用的添加，因为它让我们能够向我们的游戏添加全新的输入方法，同时也展示了扩展 Marmalade 功能是多么容易。

我们扩展将包含以下函数：

| 函数 | 描述 |
| --- | --- |
| `GyroscopeAvailable` | 这个函数由 EDK 构建过程自动为我们生成。如果当前平台支持陀螺仪扩展，则返回 `S3E_TRUE`；如果不支持，则返回 `S3E_FALSE`。 |
| `GyroscopeSupported` | 并非所有移动设备实际上都包含陀螺仪硬件，因此这个函数提供了确定我们是否可以在游戏中使用陀螺仪的方法。该函数返回一个正常的 C++ `bool` 值，指示是否存在陀螺仪。 |
| `GyroscopeStart` 和 `GyroscopeStop` | 这两个函数用于启动和停止生成陀螺仪输入数据的硬件。 |
| `GyroscopeGetX`、`GyroscopeGetY` 和 `GyroscopeGetZ` | 返回 X、Y 和 Z 轴的当前陀螺仪数据值。这些值以每秒弧度（`float` 值）的形式返回。 |

之前详细说明的 API 提供了提供陀螺仪支持所需的最小功能，并且故意保持简单，以便更清楚地展示构建扩展的过程。

## 声明扩展 API

创建扩展的第一步是指定它将包含的函数，我们将使用 **S4E 文件** 来完成这项工作。这个文件用于定义我们扩展的 API，最好通过一个例子来说明。如果您想跟上来，创建一个名为 `Gyroscope` 的新目录，并在其中创建一个名为 `Gyroscope.s4e` 的文件，其内容如下：

```swift
include:
#include <s3eTypes.h>

functions:
bool GyroscopeSupported() false
void GyroscopeStart() void
void GyroscopeStop() void
float GyroscopeGetX() 0.0f
float GyroscopeGetY() 0.0f
float GyroscopeGetZ() 0.0f
```

示例从 `include:` 行开始，随后跟随着任意数量的 C 预处理器命令、包含文件、结构定义和类定义，这些将成为扩展的主要头文件的一部分。在我们的例子中，我们只是包含了 `s3eTypes.h` 文件；但如果我们需要在扩展和调用代码之间传递大量数据，我们可能还想在这里添加结构或类、枚举和定义。

接下来是文件的 `functions:` 部分，这基本上是包含我们扩展将包含并可以在使用扩展的 Marmalade 项目中调用的函数的列表。

### 注意

我们不必在函数列表中明确列出 `GyroscopeAvailable` 函数。EDK 构建过程会自动为我们生成这个函数，通过取 S4E 文件名并在其末尾添加 "Available" 来实现。

如您所见，函数的列表几乎就像它们是正常的 C 函数原型一样。每个函数都单独列出，首先声明返回类型，然后是其名称和参数列表（在这个例子中，所有这些都恰好是空的！）。

此外，S4E 文件中函数列表中的每个函数还指定了它将返回的默认值，并且可以跟随着一系列可选指令，这些指令控制函数的行为、如何将其添加到扩展中以及如何调用它。我们的例子没有使用这些指令，但以下表格显示了可以指定的内容：

| 指令 | 描述 |
| --- | --- |
| `run_on_osthread` | 指定扩展函数应在应用程序的主要 OS 线程上执行。如果该函数执行任何类型的用户界面交互，这尤其重要，因为许多平台只允许在主线程上调用 UI 调用。 |
| `no_lock` | 在调用此函数时禁用线程安全锁定。默认情况下，所有扩展函数只能在任何特定时间由单个线程调用，并且会自动生成锁定代码以确保这一点。 |
| `fast` | 启用快速栈切换。这是一个优化选项，意味着在通过使用与加载模块相同的栈来调用扩展函数时，我们应用程序和加载器之间需要传递的数据更少。通常，加载模块和我们的应用程序代码有独立的栈。 |
| `no_assert` | 如果在尚未为该平台构建扩展的情况下调用扩展函数，将阻止抛出断言。函数的默认值将被返回。 |
| `order` | 默认情况下，S4E 文件中列出的每个函数都将按列表顺序添加到扩展中，并且此顺序在内部用于定位要调用的正确函数指针。随着我们的扩展随时间发展，我们可能想要添加或弃用函数，但仍希望将相关函数在 S4E 文件中保持在一起。通过在函数声明后添加`order=x`，我们表示此函数将占据函数顺序中的位置`x`，其中`x=1`表示紧接在未指定顺序值的最后一个函数之后。如果这听起来很复杂，请不要担心；对于我们的项目，我们可能永远不会需要使用此功能，因为它只有在我们将扩展提供给其他人使用时才真正成为问题！ |

在 S4E 文件中还可以指定一些全局指令，并且这些指令应列在文件的最开始处，在`include:`行之前。同样，我们的示例没有使用这些指令，但为了您的信息，它们列在以下表中：

| 指令 | 描述 |
| --- | --- |
| `no_init_term` | 指定扩展不需要自动生成初始化或终止函数。由于这些函数通常用于设置扩展与我们的项目代码之间的接口，因此您不太可能使用此指令。 |
| `errors` | 允许访问一些宏，这些宏通过自动生成函数（如许多 S3E API 中存在的`GetError`）使错误通信更容易实现。这些 API 构成了低级 Marmalade API。 |
| `globals` | 声明扩展将需要为其内部使用分配一个全局结构块，并使一些宏可用，以便支持在此结构中获取和设置值。 |
| `callbacks` | 表示此扩展希望使用回调，并将自动定义回调 ID 以支持此功能，使用与其他内置 S3E API 相同的方法。 |

## 为 Windows 制作扩展

我们将首先创建用于 Windows 的扩展。显然，Windows PC 很可能没有陀螺仪硬件（尽管我想不是不可能的！），但以 Windows 版本开始是最容易的，因为它不需要我们安装任何额外的软件或 SDK 来构建它。

### 创建 Windows 扩展

由于我们实际上不会在 Windows 上支持陀螺仪输入，我们的 API 只需要在 `GyroscopeSupported` 函数中返回 `false`，并且访问当前陀螺仪值的函数应该始终返回一个 `0` 值。显然，开始和停止函数需要做的是绝对没有任何操作！

我们已经创建了 S4E 文件，现在我们将开始使用它。打开 Windows 资源管理器，导航到 `Gyroscope` 目录，然后右键单击 `Gyroscope.s4e` 文件。选择菜单选项 **构建 Windows 扩展**，这将运行一个 Python 脚本，生成多个新文件和目录。

在主 `Gyroscope` 目录中创建了三个新文件：

+   `Gyroscope_build.mkf` 是扩展的 MKF 文件，允许我们指定构建它所需的附加通用或平台相关源文件。

+   `Gyroscope.mkf` 是任何使用我们扩展的 Marmalade 项目需要包含为子项目以访问扩展函数的 MKF 文件

+   `Gyroscope_windows.mkb` 是创建一个 Visual Studio 项目的 MKB 文件，我们可以使用它来编译扩展代码

还创建了四个子目录。我们可以安全地忽略 `stamp` 目录，它包含一个由 EDK 构建脚本内部使用的文件，用于跟踪扩展 API 的更改。我们还可以忽略 `interface` 目录中的文件，这些文件是自动生成的，不应被修改。

`h` 目录包含一个名为 `Gyroscope.h` 的单个文件，我们不应修改它，因为我们所做的任何更改都将被扩展创建脚本覆盖。然而，这个文件非常有用，因为它是我们将包含在我们的项目源文件中以便访问扩展中函数的文件。

最后是 `source` 目录，它反过来又包含三个更多的子目录。`generic` 子目录包含定义扩展默认行为的源文件，如果未提供平台特定的源文件。`h` 目录还包含用于构建扩展代码的所有平台的文件。虽然我们可以修改这些文件，但我们不太可能需要这样做。

这使得我们拥有一个名为 `Gyroscope_platform.cpp` 的单个文件的 `windows` 子目录。此文件包含从 S4E 文件的功能列表中提供的数据生成的我们每个扩展函数的存根。

然而，请注意，所有模拟函数都以`_platform`后缀结尾。EDK 系统实际上生成了一组具有 S4E 文件中指定名称的通用函数，如果存在，则调用以`_platform`结尾的等效函数。这是必要的，以便使用扩展的代码仍然可以在没有或无法创建扩展的平台上进行编译和执行。

### 实现 Windows 扩展

通常，我们需要修改`Gyroscope_platform.cpp`文件来实现扩展；但出于我们的目的，实际上并不需要任何更改，因为生成的模拟提供了在 Windows 上所需的函数。

显然，在这种情况下，一个 Windows 扩展有点多余，但请记住，我们始终可以创建一个更复杂的扩展，它以某种方式模拟陀螺仪行为，也许使用游戏手柄或其他输入设备。

### 构建 Windows 扩展

要构建扩展，我们只需双击`Gyroscope\Gyroscope_windows.mkb`文件来创建一个 Visual Studio 项目。一旦 Visual Studio 启动，从 Visual Studio IDE 顶部的下拉菜单中选择**（x86）发布**构建类型，转到菜单选项**构建** | **构建解决方案**（或直接按*F7*键），扩展的 Windows 版本将被创建。很简单！

## 创建 Android 扩展

现在，我们将注意力转向 Android。然而，在我们开始之前，我们需要安装一些软件，因为构建过程需要能够访问 Java 开发工具和 Android SDK。

### 安装 Android 开发所需的软件

首先，您需要安装 Java JDK，它可在以下地址下载：

[`www.oracle.com/technetwork/java/javase/downloads/index.html`](http://www.oracle.com/technetwork/java/javase/downloads/index.html)

### 注意

下载 JDK 时，请确保下载的是版本 6，而不是更新的版本 7。Android SDK 与版本 7 可能无法正确工作。

下载安装包后，执行它并按照说明将 Java 开发工具安装到您的 PC 上。

接下来，您需要下载 Android SDK 和 NDK。Android SDK 是通常用于开发 Android 应用程序的 Java 库，而 NDK 是一组额外的库，它允许 Java Android 代码与编译的 C++代码接口。

Android SDK 可在以下网址获取：

[`developer.android.com/sdk/index.html`](http://developer.android.com/sdk/index.html)

它以 Windows 安装程序文件的形式提供；所以只需执行它，接受所有默认安装选项，等待安装完成。

### 注意

一旦安装了 Android SDK，将环境变量 `ANDROID_ROOT` 设置为安装目录是有用的。这可以让 Marmalade 部署工具知道 Android 平台工具的位置，以便它可以自动安装和运行连接到你的 PC 的 Android 设备上的生成的包文件。

接下来，你可以访问以下 URL 下载 Android NDK：

[`developer.android.com/tools/sdk/ndk/index.html`](http://developer.android.com/tools/sdk/ndk/index.html)

### 注意

根据你使用的 Marmalade 版本，你可能需要不同版本的 NDK。如果你使用的是 Marmalade 6.1 或更高版本，正如本书所预期的那样，你需要 NDK 版本 8。对于 Marmalade 的早期版本，你需要 NDK 版本 7。

NDK 以 ZIP 存档的形式提供，因此你需要使用合适的归档程序（例如，WinZip）将其解压缩。NDK 应该包含在一个名为 `android-ndk-xxx` 的目录中，其中 `xxx` 指的是 NDK 的版本号。你可以将其复制到 `C:` 的根目录，或者你可以设置环境变量 `NDK_ROOT` 以指向安装路径。

### 创建 Android 扩展

现在我们已经安装了必要的开发工具，我们可以通过再次使用 Windows 资源管理器来定位 `Gyroscope.s4e` 文件来创建 Android 扩展文件。右键单击文件并选择 **构建 Android 扩展** 菜单选项。

`Gyroscope_android.mkb` 和 `Gyroscope_android_java.mkb` 文件将在主 `Gyroscope` 目录中创建。这些文件将在稍后用于构建扩展代码。

`source` 目录现在将包含一个名为 `android` 的新目录，其中包含两个文件 `Gyroscope.java` 和 `Gyroscope_platform.cpp`。前者是我们可以添加使用 Android SDK 代码来实现我们的扩展 API 的 Java 代码的地方。后者是我们 Marmalade 项目将调用的 C++ 代码，它反过来调用 Java 实现代码。

有可能通过使用 **Java Native Interface**（**JNI**）来访问和调用编译后的 Java 代码，在 `Gyroscope_platform.cpp` 文件中实现整个扩展；但这增加了一个额外的复杂性层，通常在 Java 中实现扩展要简单得多！

### 实现 Android 扩展

要为 Android 实现陀螺仪代码，我们需要编辑 `source\android\Gyroscope.java` 文件。首先，我们需要引用我们将要使用的 Java 类；因此，将文件顶部的导入声明列表更改为如下所示：

```swift
import com.ideaworks3d.marmalade.LoaderAPI;
import com.ideaworks3d.marmalade.LoaderActivity;

import android.content.Context;
import android.hardware.Sensor;
import android.hardware.SensorEvent;
import android.hardware.SensorEventListener;
import android.hardware.SensorManager;
```

前两个导入允许我们访问一些辅助函数，这些函数提供了对诸如应用程序的主要 `Activity` 类（所有 Android 应用程序都需要从这个基类派生）等事物的访问。我们需要这个来访问一些系统资源。

剩余的导入是为了使用 Android SDK 的某些部分，我们将使用它们来访问陀螺仪数据。

EDK 系统已生成一个名为`Gyroscope`的 Java 类，其中包含我们需要实现的所有方法的占位符。不过，我们需要稍微修改类定义，因为我们需要实现一些将接收陀螺仪更新的方法。按如下方式更改类定义：

```swift
class Gyroscope implements SensorEventListener
```

`SensorEventListener`是一个 Java 接口，我们的类必须实现它以接收传感器事件（在我们的案例中，是陀螺仪数据）。

我们还将添加一些成员变量来缓存陀螺仪值，以及一个标志，我们将用它来处理某些 Android 设备返回每秒度数而不是每秒弧度值的事实。将以下代码添加到类定义的底部：

```swift
// Cached gyroscope values
private float x;
private float y;
private float z;

// Are the results in degrees/s or radians/s
private boolean mUsesDegrees;
```

在开始实现 EDK 本身之前，我们将添加几个私有辅助函数，以便我们可以访问 Android 的`SensorManager`和陀螺仪`Sensor`实例，这将允许我们检索当前的陀螺仪数据。在类定义的开始处添加以下两个方法：

```swift
// Helper function for accessing the Android SensorManager
private SensorManager GetSensorManager()
{
   Context lContext = (Context) LoaderActivity.m_Activity;
   SensorManager lSensorManager = (SensorManager)
      lContext.getSystemService(Context.SENSOR_SERVICE);
   return lSensorManager;
}

// Helper function for accessing the Android Gyroscope Sensor
private Sensor GetGyroscopeSensor()
{
   SensorManager lSensorManager = GetSensorManager();
   if (lSensorManager == null)
      return null;

   Sensor lGyroscope =
      lSensorManager.getDefaultSensor(Sensor.TYPE_GYROSCOPE);
   return lGyroscope;
}
```

`GetSensorManager`方法通过使用 Marmalade 应用程序的主`Context`类来访问全局`SensorManager`实例。我们使用 Marmalade 的`LoaderActivity`类来完成此操作，该类包含一个成员变量，它是主 Android SDK `Activity`类实例的引用。然后，由于`Activity`类继承自`Context`，这个引用可以被转换为`Context`实例的引用。

一旦我们有了`Context`引用，我们就用它来获取负责控制输入设备（包括陀螺仪）的 Android `SensorManager`类的引用。如果没有可用的引用，将返回一个`null`引用。

`GetGyroscopeSensor`方法允许我们通过请求`SensorManager`类的默认陀螺仪处理程序来检查是否存在陀螺仪。如果没有找到合适的处理程序（即返回值为`null`），则表示此设备上没有陀螺仪硬件。

现在我们可以通过查看`GyroscopeSupported`方法来开始实现 API。这个函数只有在设备具有陀螺仪硬件时才需要返回`true`。我们可以这样做：

```swift
public boolean GyroscopeSupported()
{
   Sensor lSensor = GetGyroscopeSensor();
   return lSensor != null;
}
```

现在是时候实现一个函数，使我们能够开始接收陀螺仪数据。找到`GyroscopeStart`方法，并将其更改为以下代码片段：

```swift
public void GyroscopeStart()
{
  x = 0.0f;
  y = 0.0f;
  z = 0.0f;
  mUsesDegrees = false;

  Sensor lGyroscope = GetGyroscopeSensor();
  if (lGyroscope != null)
  {
    mUsesDegrees = lGyroscope.getMaximumRange() > 100;
    GetSensorManager().registerListener(this, lGyroscope,
                  SensorManager.SENSOR_DELAY_FASTEST);
  }
}
```

在这个方法中，我们首先确保缓存的陀螺仪值为零，并假设设备将返回每秒弧度值。然后，我们使用我们的私有`GetGyroscopeSensor`方法获取陀螺仪的`Sensor`类实例。

为了确定这个设备返回的是角度还是弧度值，我们查看陀螺仪传感器的最大范围值。如果最大范围大于 100，我们将`mUsesDegrees`成员变量设置为`true`，因为没有更可靠的方法来确定这一点。

然后我们将我们的类实例设置为陀螺仪数据的监听器。定期地，`onSensorChanged` 方法（我们尚未实现）将使用新的陀螺仪值被调用。

接下来我们将实现 `GyroscopeStop` 函数，它应该看起来像这样：

```swift
public void GyroscopeStop()
{
  SensorManager lSensorManager = GetSensorManager();
  if (lSensorManager != null)
  {
    lSensorManager.unregisterListener(this);
  }

  x = 0.0f;
  y = 0.0f;
  z = 0.0f;
}
```

再次获取 `SensorManager` 类引用，并告诉它我们不再希望接收陀螺仪数据。我们还清除缓存的陀螺仪值，以防我们的代码在陀螺仪硬件不活跃时尝试访问它们。

我们需要实现的下一个三个方法是返回缓存的陀螺仪值的方法。这些方法很容易实现，应该看起来像这样：

```swift
public float GyroscopeGetX()
{
  return x;
}

public float GyroscopeGetY()
{
  return y;
}

public float GyroscopeGetZ()
{
  return z;
}
```

我们现在几乎完成了。剩下要做的就是实现 `SensorEventListener` 接口中我们从中派生 `Gyroscope` 类的监听器方法。在 `GyroscopeGetZ` 方法之后添加以下代码。

```swift
public void onAccuracyChanged(Sensor aSensor, int aAccuracy) 
{
}

public void onSensorChanged(SensorEvent aEvent) 
{
  if (aEvent.accuracy != SensorManager.SENSOR_STATUS_UNRELIABLE)
  {
    x = aEvent.values[0];
    y = aEvent.values[1];
    z = aEvent.values[2];

    if (mUsesDegrees)
    {
      x = (x * 3.14159267f) / 180.0f;
      y = (y * 3.14159267f) / 180.0f;
      z = (z * 3.14159267f) / 180.0f;
    }
  }
}
```

`onAccuracyChanged` 方法留空，因为它必须实现以满足接口。然而，`onSensorChanged` 方法很重要，因为它将接收新的陀螺仪输入值。我们首先检查传入的 `SensorEvent` 是否包含可靠的数据（设备本身将确定什么构成可靠数据）；然后，我们只需提取新的陀螺仪值并将它们存储在我们的成员变量中。

如果我们确定设备返回的是每秒度数，我们进行快速转换到弧度，以确保我们的扩展始终返回一致值。

### 构建 Android 扩展

我们现在 Android 扩展的代码已经准备好构建，这甚至比 Windows 版本还要简单。我们只需要打开 Windows 资源管理器，导航到 `Gyroscope` 目录，然后双击第一个 `Gyroscope_android_java.mkb` 文件，然后是 `Gyroscope_android.mkb` 文件。第一个 MKB 文件将构建 Java 代码，而第二个将构建将被我们的项目代码调用的 C++ 代码，该代码反过来会调用 Java 代码。

## 创建 iOS 扩展

构建 iOS 的 EDK 扩展稍微复杂一些，因为它要求我们能够访问 Apple iOS SDK，因此需要一个 Apple Mac。

### 安装 iOS 开发所需的软件

首先，你需要下载与 Apple 的 XCode 开发环境捆绑在一起的 iOS SDK。访问以下网页，其中将包含一个链接到 Mac App Store，可以下载最新的 XCode 版本：

[`developer.apple.com/xcode/index.php`](https://developer.apple.com/xcode/index.php)

一旦下载了 XCode 并安装了它，你接下来需要下载 Marmalade SDK 的 Mac OS X 版本。访问以下 URL 的 Marmalade 网站，登录并下载 Marmalade 的 Mac 版本。

[`www.madewithmarmalade.com/downloads`](https://www.madewithmarmalade.com/downloads)

将 Marmalade SDK 安装到默认位置。如果您只有一个 Marmalade 许可证，您将需要使用 Marmalade 网站，从您的 PC 释放许可证，以便您可以在 Mac 上使用它。有关如何操作的更多信息，请参阅本书的 第一章，*Marmalade 入门*。

### 创建 iOS 扩展

毫不奇怪，我们以类似的方式创建 iOS 扩展所需的文件，就像 Windows 和 Android 扩展一样。只需右键单击 `Gyroscope.s4e` 文件，然后选择菜单选项 **构建 iPhone 扩展**。

为 iOS 扩展将创建两个新的文件。这些是 `Gyroscope_iphone.mkb`，这是我们用来构建扩展代码的 MKB 文件，以及 `source\iphone\Gyroscope_platform.mm`，其中包含我们 API 函数的自动生成的存根。

### 实现 iOS 扩展

要实现 Gyroscope 扩展的 iOS 版本，我们需要编辑 `Gyroscope_platform.mm` 文件。此文件是一个 Objective-C 源文件，它还允许我们在同一文件中使用 C 和 C++ 代码。函数存根都是标准的 C 风格函数，但我们可以仍然在它们中使用 Objective-C 类和功能。

在 iOS 上，我们使用名为 `CMMotionManager` 的 Objective-C 类来访问陀螺仪数据，因此我们首先需要通过更改包含文件列表来让我们的代码了解这个类：

```swift
#include <CoreMotion/CoreMotion.h>
#include "Gyroscope_internal.h"
```

我们还将声明一个指向 `CMMotionManager` 实例的全局指针，我们将在代码的其余部分使用它。在包含文件之后添加以下行：

```swift
CMMotionManager* gpMotionManager = nil;
```

在我们可以访问陀螺仪之前，我们需要为这个类分配一个实例。幸运的是，EDK 构建脚本已经生成了一个名为 `GyroscopeInit_platform` 的函数，当我们在项目中使用扩展时，它会自动为我们调用，因此这是一个分配新的 `CMMotionManager` 实例的好地方，如下面的代码所示：

```swift
s3eResult GyroscopeInit_platform()
{
  gpMotionManager = [[CMMotionManager alloc] init];

  return S3E_RESULT_SUCCESS;
}
```

我们还需要在应用程序终止时释放该实例，并且 EDK 构建脚本再次通过 `GyroscopeTerminate_platform` 函数来帮助我们。我们需要修改此函数，以便在陀螺仪仍然活跃时停止它，然后释放 `CMMotionManager` 实例。以下是完成后的函数：

```swift
void GyroscopeTerminate_platform()
{
  GyroscopeStop_platform();
  [gpMotionManager release];
}
```

实现的其余部分实际上非常简单，因为 `CMMotionManager` 类的工作方式与我们所选择的扩展 API 非常相似。我们将从检查陀螺仪硬件是否可用开始。`GyroscopeSupported_platform` 函数看起来如下：

```swift
bool GyroscopeSupported_platform()
{
    return gpMotionManager.gyroAvailable;
}
```

启动和停止陀螺仪硬件实际上也不过是调用 `CMMotionManager` 类的一个方法。为了安全起见，我们用进一步的检查来包装这些调用，以确保陀螺仪可用且尚未启动或停止。

```swift
void GyroscopeStart_platform()
{
  if (gpMotionManager.gyroAvailable && !gpMotionManager.gyroActive)
  {
    [gpMotionManager startGyroUpdates];
  }
}

void GyroscopeStop_platform()
{
  if (gpMotionManager.gyroAvailable && gpMotionManager.gyroActive)
  {
    [gpMotionManager stopGyroUpdates];
  }
}
```

剩下的唯一事情就是获取当前的陀螺仪输入值。`CMMotionManager`类包含一个名为`gyroData`的`CMGyroData`类属性，该属性反过来包含一个名为`rotationRate`的`CMRotationRate`属性，它保存当前的陀螺仪数据。

以下代码展示了获取 x 轴陀螺仪数据的实现方法。从这一点可以看出，如何获取 y 轴和 z 轴的值应该是相当明显的！

```swift
float GyroscopeGetX_platform()
{
  CMGyroData* lpGyroData = [gpMotionManager gyroData];
  if (lpGyroData)
  {
    CMRotationRate lpRotRate = [lpGyroData rotationRate];
    return lpRotRate.x;
  }
  else
  {
    return 0.0f;
  }
}
```

在我们构建扩展之前，还有最后一件事要做，那就是告诉 EDK 构建工具我们需要包含 iOS SDK 框架`CoreMotion`，因为这个框架包含了`CMMotionManager`类的代码。

要将框架添加到我们的扩展中，我们必须编辑`Gyroscope.mkf`文件。在文件的底部查找 iOS 的`deployments`部分（由于历史原因，Marmalade 将其称为“iphone 平台”）并添加以下行到其中：

```swift
iphone-link-opts="-framework CoreMotion"
```

### 构建 iOS 扩展

到目前为止，创建 iOS 扩展的所有先前步骤都可以在 Windows 或 Mac 上同样很好地完成，但这个最后一步绝对需要我们使用 Mac。

### 注意

我们需要确保 Mac 可以访问整个`Gyroscope`目录。你如何实现这一点取决于你，但一种好方法是共享`Gyroscope`目录到你的开发 Windows PC 上，然后在 Mac 上访问这个共享。这样代码在 Mac 上构建，但所有编译文件都已经在你 Windows 开发机器的正确位置。

要构建扩展，首先需要打开 Mac 终端窗口。将`Gyroscope`目录设置为终端窗口中的当前目录，然后输入以下命令行：

```swift
mkb Gyroscope_iphone.mkb –arm
```

这将构建扩展，我们在 Mac 上的工作就完成了。简单，但只执行一个命令就有点烦人，不是吗？

# 使用陀螺仪扩展

我们现在已经看到了如何为 Windows、Android 和 iOS 创建和构建扩展模块，但我们如何在 Marmalade 项目中使用它们呢？

实际上，这出奇地简单。我们只需要在项目 MKB 文件的`subprojects`部分引用我们的扩展（最简单的方法是从主项目目录提供到`Gyroscope`目录的相对路径），就像我们处理任何正常的代码模块一样，然后包含自动生成的`Gyroscope.h`头文件，这样我们就可以调用扩展函数。

需要注意的唯一一点是，由于可能没有为我们要针对的每个平台创建扩展，我们必须确保在调用其任何函数之前扩展可用。这可以通过使用 EDK 构建脚本为我们自动生成的`GyroscopeAvailable`函数轻松完成。如果这个函数返回`S3E_TRUE`，则扩展可用。如果它返回`S3E_FALSE`，对扩展函数的任何调用将触发断言，但否则不会做任何事情。

即使在 Android 上，构建或部署我们的应用程序也不需要任何特殊步骤，因为在 Java 中编写的任何代码都需要以 JAR 文件的形式提供。部署工具将自动将所需的扩展文件添加到安装包中，而无需我们做任何事情。

# 示例代码

以下部分详细介绍了伴随本章的代码示例。

## 陀螺仪项目

本项目包含本章开发过程中开发的陀螺仪扩展的完整源代码。还包含了扩展的编译版本，这样您就可以构建本章的其他示例项目，而无需首先构建扩展本身。

## GyroTest 项目

GyroTest 项目是一个简单的示例，它使用了陀螺仪扩展。它演示了如何将陀螺仪扩展包含到项目中，如何检查扩展是否可用，以及如果可用，如何调用扩展函数。

无论陀螺仪支持是否可用，样本都将显示在屏幕上。如果可用，原始陀螺仪值也将显示在屏幕上。

## 滑雪项目

我们对滑雪项目的最终更新使其能够利用本章开发的陀螺仪扩展作为另一种可能的控制方法。

与游戏中其他输入方法一样，创建了一个名为 `GyroscopeManager` 的类，它封装了陀螺仪扩展。这样，所有对扩展函数的使用都集中在一个源文件中，如果将来需要以任何方式更改扩展的 API，这将使得更新变得更加容易。

无论设备多么静止，即使放在稳定的表面上，陀螺仪值总会有一定程度的抖动。`GyroscopeManager` 类通过维护用于控制游戏中滑雪者的陀螺仪输入的过滤版本来处理这个问题。

在主游戏循环的每次更新中，通过将每个轴的当前过滤值与新原始值之间的差异的百分比加到当前过滤值上，计算每个陀螺仪轴的新过滤值。这导致大部分抖动效果被忽略，同时不会丢失玩家有意输入的较大陀螺仪输入。

# 概述

在本章中，我们探讨了如何使用扩展开发工具包来扩展 Marmalade 的功能。如您所见，创建一个可以利用 EDK 当前支持的每个平台上的 API 的扩展相对容易。

陀螺仪扩展是支持尚未在主 Marmalade SDK 中公开的硬件特性的好例子，但如果您想使用可能直接为特定平台使用其本地 SDK 创建的任何第三方库，扩展也会非常有用。

随着本书的结束，你现在应该已经很好地掌握了 Marmalade SDK 的强大功能，并且希望你能抓住开发游戏并在多个极受欢迎的平台上线的机会。祝您编码愉快，创作出下一个大型游戏现象的最佳祝福！
