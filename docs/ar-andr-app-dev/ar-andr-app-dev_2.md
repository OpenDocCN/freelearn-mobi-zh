# 第二章：观察世界

在本章中，我们将学习如何开发任何移动 AR 应用程序的第一个元素：*真实世界的视图*。为了理解真实世界视图的概念，我们将看看你安装在手机上的摄像头应用程序。打开你预装在安卓设备上的任何照片拍摄应用程序（摄像头应用），或者你可能从谷歌应用商店下载的应用（如 Camera Zoom FX、Vignette 等）。你在应用程序取景器上看到的是摄像头捕获的实时视频流，并显示在你的屏幕上。

在运行应用程序时移动设备，仿佛你通过设备看到了真实世界。实际上，摄像头就像是设备的眼睛，感知你周围的环境。这个过程也用于移动 AR 开发，以创建真实世界的视图。这是我们在前一章中介绍的透视视频的概念。

显示真实世界需要两个主要步骤：

+   从摄像头捕获图像（摄像头访问）

+   使用图形库在屏幕上显示这个图像（在 JME 中显示摄像头）

这个过程通常在一个无限循环中重复，创建了物理世界视图的*实时*特性。在本章中，我们将讨论如何使用两个不同的图形库实现这两种技术：一个低级别的（Android 库）和一个高级的（JME 3D 场景图库）。虽然 Android 库能让你快速显示摄像头图像，但它并非设计为与你想在视频流上增强的 3D 图形结合使用。因此，你也将使用 JME 库实现摄像头显示。我们还将介绍处理各种 Android 智能手机及其内置摄像头的挑战和提示。

# 理解摄像头

手机制造商总是在竞相为你的智能手机配备最先进的摄像头传感器，加入更多功能，如更高的分辨率、更好的对比度、更快的视频捕捉、新的自动对焦模式等等。结果是，不同智能手机型号或品牌之间的手机摄像头功能（特性）可能存在显著差异。幸运的是，谷歌 Android API 为底层摄像头硬件提供了一个通用封装，统一了开发者的访问方式：即 Android 摄像头 API。在开发过程中，高效地访问摄像头需要对 API 提供的摄像头功能（参数和函数）有清晰的理解。忽视这一点会导致应用程序运行缓慢或图像像素化，影响你的应用程序用户体验。

## 摄像头特性

现今智能手机上的相机与数码傻瓜相机有许多共同特性。它们通常支持两种操作模式：静态图像模式（即瞬间捕捉的单个图像），或者视频模式（即连续实时捕捉图像）。

视频和图像模式在功能上有所不同：例如，图像捕捉总是具有比视频更高的分辨率（更多像素）。虽然现代智能手机在静态图像模式下轻松实现 800 万像素，但视频模式限制在 1080p（约 200 万像素）左右。在增强现实（AR）中，我们通常使用较低分辨率如 VGA（640 x 480）的视频模式以提高效率。与标准数码相机不同，我们不将任何内容存储在外部存储卡上；我们只是在屏幕上显示图像。这个模式在 Android API 中有一个特殊的名字：预览模式。

预览模式的一些常见设置（参数）包括：

+   **分辨率**：这是捕捉到的图像的大小，可以在你的屏幕上显示。在 Android 相机 API 中也被称为尺寸。分辨率以像素为单位定义图像的宽（x）和高（y）。它们之间的比率称为**宽高比**，这给出了图像与电视分辨率（如 1:1、4:3 或 16:9）相似程度的感知。

+   **帧率**：它定义了图像可以捕捉的速度。这也被称为**每秒帧数**（**FPS**）。

+   **白平衡**：它决定了图像上的白色会是什么样子，主要取决于你的环境光线（例如，户外情况下是日光，家里是白炽灯，工作场所是荧光灯等）。

+   **焦点**：它定义了图像中哪部分会显得清晰，哪部分不容易被辨识（失焦）。与其他相机一样，智能手机相机也支持自动对焦模式。

+   **像素格式**：捕捉到的图像被转换成特定的图像格式，每个像素的颜色（和亮度）以特定格式存储。像素格式不仅定义了颜色通道的类型（如 RGB 与 YCbCr），还定义了每个分量的存储大小（例如 5 位、8 位或 16 位）。一些流行的像素格式包括 RGB888、RGB565 或 YCbCr422。在下面的图中，你可以看到从左到右常见的相机参数：图像分辨率、捕捉图像流的帧率、相机焦点、存储图像的像素格式以及白平衡：![相机特性](img/8553OS_02_01.jpg)

与相机工作流程相关的重要设置还有：

+   **播放控制**：定义了你可以开始、暂停、停止或获取相机图像内容的时间。

+   **缓冲控制**：捕捉到的图像被复制到内存中，以便你的应用程序可以访问。存储这个图像有不同的方法，例如，使用缓冲系统。

正确配置这些设置是 AR 应用的基本要求。尽管流行的相机应用仅使用预览模式来捕捉视频或图像，但预览模式是 AR 中现实世界视图的基础。你需要记住的一些关于配置这些相机参数的事情包括：

+   分辨率越高，帧率越低，这意味着如果你的应用中图像内容移动不快，它看起来可能会更美观，但运行会更缓慢。相比之下，你可以让应用运行得更快，但图像看起来会变得“块状”（像素化效果）。

+   如果白平衡设置不当，叠加在视频图像上的数字模型的外观将不匹配，AR 体验将大打折扣。

+   如果焦点不断变化（自动对焦），你可能无法分析图像的内容，应用的其他部分（如追踪）可能无法正确工作。

+   移动设备上的相机使用压缩图像格式，并且通常不具备高端桌面网络摄像头相同的性能。当你将视频图像（通常是 RGB565 格式与使用 RGB8888 格式的 3D 渲染内容结合）结合在一起时，你可能会注意到它们之间的颜色差异。

+   如果你在图像上进行大量处理，那可能会造成应用延迟。此外，如果你的应用同时运行多个进程，将图像捕获过程与其他进程同步是非常重要的。

我们建议你：

+   获取并测试各种 Android 设备和它们的相机，以了解相机的功能和性能。

+   在分辨率和帧率之间找到平衡点。桌面 AR 上使用的标准分辨率/帧率组合是 640 x 480，30 fps。将此作为你移动 AR 应用的基础，并在此基础上优化，以获得适用于新型设备的更高质量的 AR 应用。

+   如果你的 AR 应用仅计划在特定环境中运行，例如白天户外应用，请优化白平衡。

+   控制焦点一直是 Android 智能手机的限制因素之一（始终开启自动对焦或无法配置）。如果开发的是桌面或室内 AR 应用（近焦）相对于户外 AR 应用（远焦），请优先选择固定焦点，并优化焦点范围。

+   实验不同的像素格式，以与你的渲染内容达到最佳匹配。

+   如果目标设备支持，尝试使用先进的缓冲系统。

还有一些通过 API 无法获取的（或在部分手持设备上才能获取的）相机主要特性，在开发 AR 应用时也应当考虑。这些特性包括视场、曝光时间和光圈。

在这里我们只讨论其中之一：视野。视野对应于相机从现实世界中看到的内容，比如你的眼睛可以从左到右、从上到下看到多少（人类的双眼视觉大约是 120 度）。视野以度数测量，不同相机之间差异很大（15 度至 60 度，无变形）。

你的视野越大，捕获的现实世界视野就越广，体验也越好。视野取决于相机的硬件特性（传感器尺寸和焦距）。可以使用额外的工具来估算这个视野；我们稍后会进行探讨。

## 相机与屏幕特性对比

在你的移动平台上，相机和屏幕特性通常不完全相同。例如，相机图像可能比屏幕分辨率大。屏幕的宽高比也可能与其中一个相机不同。在增强现实（AR）中，这是一个技术挑战，因为你想找到最好的方法将相机图像适配到屏幕上，以创建 AR 显示的感觉。你希望尽可能多地将在相机图像上的信息显示在屏幕上。在电影行业，他们有类似的问题，因为记录的格式可能与播放媒体不同（例如，在 4:3 的移动设备上播放宽银幕电影，在 1080p 的电视屏幕上播放 4K 电影分辨率等）。为了解决这个问题，你可以使用两种全屏方法：拉伸和裁剪，如下图所示：

![相机与屏幕特性对比](img/8553OS_02_02.jpg)

拉伸会根据屏幕特性调整相机图像，这可能会导致图像原始格式（主要是宽高比）的变形。裁剪会选择图像的一个子区域进行显示，你会丢失一些信息（基本上是将图像放大，直到整个屏幕被填满）。另一种方法是改变图像的缩放比例，使得屏幕和图像的一个尺寸（宽度或高度）相同。这里的缺点是，你将无法全屏显示相机图像（图像边缘将出现黑边）。这些技术都不完美，因此你需要实验哪种方法对你的应用和目标设备更方便。

# 在 Android 中访问相机

首先，我们将创建一个简单的相机活动，以了解在 Android 中访问相机的原理。尽管有方便的 Android 应用程序可以通过 Android 意图快速抓拍照片或录制视频，但我们将会亲自动手，使用 Android 相机 API 为我们的第一个应用程序获得定制化的相机访问。

我们将一步一步指导你创建你的第一个显示实时相机预览的应用程序。这将包括：

+   创建一个 Eclipse 项目

+   在 Android Manifest 文件中请求相关权限

+   创建 SurfaceView 以捕获相机的预览帧

+   创建一个显示相机预览帧的活动

+   设置相机参数

### 提示

**下载示例代码**

你可以从你在 [`www.packtpub.com`](http://www.packtpub.com) 的账户下载你所购买的 Packt 书籍的示例代码文件。如果你在别处购买了这本书，可以访问 [`www.packtpub.com/support`](http://www.packtpub.com/support) 注册，直接通过电子邮件发送文件给你。你还可以在 [`github.com/arandroidbook/ar4android`](https://github.com/arandroidbook/ar4android) 找到代码文件。

## 创建一个 Eclipse 项目

我们的第一步是在 Eclipse 中创建 Android 项目的设置过程。我们将第一个项目命名为 `CameraAccessAndroid`。请注意，本小节的描述将与其他将在本书中展示的示例类似。

启动你的 Eclipse 项目并转到 **文件** | **新建** | **Android 应用程序项目**。在接下来的配置对话框中，请按照以下截图填写适当的字段：

![创建一个 Eclipse 项目](img/8553_02_03_FINAL.jpg)

然后，点击另外两个对话框（**配置项目** 选择到你的项目文件路径，**启动器图标**），接受默认值。接着，在 **创建活动** 对话框中，选中 **创建活动** 复选框和 **空白活动** 选项。在接下来的 **新建空白活动** 对话框中，在 **活动名称** 文本框中填写，例如 `CameraAccessAndroidActivity` 并将 **布局名称** 文本框保留为默认值。最后，点击 **完成** 按钮，你的项目应该会被创建并在项目浏览器中可见。

## Android Manifest 中的权限

对于我们将要创建的每个 AR 应用程序，我们将使用相机。使用 Android API，你需要在应用程序的 Android Manifest 声明中明确允许相机访问。在你的 `CameraAccessAndroid` 项目的顶级文件夹中，以文本视图打开 `AndroidManifest.xml` 文件。然后添加以下权限：

```java
<uses-permission android:name="android.permission.CAMERA" />
```

除了此权限，应用程序还需要至少声明使用相机功能：

```java
<uses-feature android:name="android.hardware.camera" />
```

由于我们希望以全屏模式运行 AR 应用程序（以获得更好的沉浸感），请在活动标签中添加以下选项：

```java
android:theme="@android:style/Theme.NoTitleBar.Fullscreen"
```

## 创建一个显示相机的活动

在最基本的形式中，我们的 `Activity` 类负责设置 `Camera` 实例。作为一个类成员，你需要声明一个 `Camera` 类的实例：

```java
public class CameraAccessAndroidActivity extends Activity {
private Camera mCamera;

}
```

下一步是打开相机。为此，我们定义一个 `getCameraInstance()` 方法：

```java
public static Camera getCameraInstance() {
  Camera c = null;
  try {
    c = Camera.open(0);
  } catch (Exception e) { ...  }
  return c;
}
```

`open()` 调用被 `try{}catch{}` 块包围非常重要，因为相机可能当前正被其他进程使用而不可用。此方法在 `Activity` 类的 `onResume()` 方法中调用：

```java
public void onResume() {
  super.onResume();  
  stopPreview = false;
  mCamera = getCameraInstance();
  ...
}
```

当你暂停或退出程序时，正确释放摄像头同样重要。否则，如果你打开另一个（或相同的）程序，摄像头将被占用。我们为此定义了一个`releaseCamera()`方法：

```java
private void releaseCamera() {
  if (mCamera != null) {

    mCamera.release();
    mCamera = null;
  }
}
```

然后，你可以在`Activity`类的`onPause()`方法中调用这个方法。

### 注意

在某些设备上，打开摄像头可能会很慢。在这种情况下，你可以使用`AsyncTask`类来缓解这个问题。

## 设置摄像头参数

现在，你已经有了启动和停止摄像头的的基本工作流程。Android 摄像头 API 还允许你查询和设置本章开始时讨论的各种摄像头参数。特别是，你应该注意不要使用分辨率非常高的图像，因为它们会消耗大量处理能力。对于典型的移动 AR 应用，你不需要高于 640 x 480（VGA）的视频分辨率。

由于摄像头模块可能存在很大差异，不建议硬编码视频分辨率。相反，查询摄像头传感器可用的分辨率，并根据你的应用程序只使用最合适的分辨率，这是一个好的做法（如果支持的话）。

假设你已经在变量`mDesiredCameraPreviewWidth`中预定义了想要的视频宽度。你可以通过以下方法检查该宽度分辨率（以及相关的视频高度）是否被摄像头支持：

```java
private void initializeCameraParameters() {
  Camera.Parameters parameters = mCamera.getParameters();
  List<Camera.Size> sizes = parameters.getSupportedPreviewSizes();
  int currentWidth = 0;
  int currentHeight = 0;
  boolean foundDesiredWidth = false;
  for(Camera.Size s: sizes) {
    if (s.width == mDesiredCameraPreviewWidth)  {             
      currentWidth = s.width;
      currentHeight = s.height;
      foundDesiredWidth = true;
      break;
    }
  }
  if(foundDesiredWidth) 
    parameters.setPreviewSize( currentWidth, currentHeight );

  mCamera.setParameters(parameters);
}     
```

`mCamera.getParameters()`方法用于查询当前的摄像头参数。`mCamera.getParameters()`和`getSupportedPreviewSizes()`方法返回可用的预览尺寸的子集，而`parameters.setPreviewSize`方法用于设置新的预览尺寸。最后，你需要调用`mCamera.setParameters(parameters)`方法，以便实施请求的更改。这个`initializeCameraParameters()`方法也可以在`Activity`类的`onResume()`方法中调用。

## 创建 SurfaceView

对于你的增强现实应用，你希望将后置摄像头的实时图像流显示在屏幕上。在标准应用中，获取视频和显示视频是两个独立的程序。使用 Android API，你还需要一个单独的 SurfaceView 来显示摄像头流。`SurfaceView`类是一个专用的绘图区域，你可以将其嵌入到你的应用程序中。

所以，对于我们的示例，我们需要从 Android 的`SurfaceView`类派生一个新类（我们称之为`CameraPreview`），并实现`SurfaceHolder.Callback`接口。这个接口用于响应与表面相关的任何事件，例如表面的创建、更改和销毁。通过`Camera`类访问移动摄像头。在构造函数中，传递了之前定义的 Android `Camera`实例：

```java
public class CameraPreview extends SurfaceView implements SurfaceHolder.Callback {
  private static final String TAG = "CameraPreview";
  private SurfaceHolder mHolder;
  private Camera mCamera;
  public CameraPreview(Context context, Camera camera) {
    super(context);
    mCamera = camera;
    mHolder = getHolder();
    mHolder.addCallback(this);
    mHolder.setType(SurfaceHolder.SURFACE_TYPE_PUSH_BUFFERS);
  }
```

在`surfaceChanged`方法中，你需要处理传递一个初始化的`SurfaceHolder`实例（这是持有显示表面的实例）并开始相机的预览流，你稍后想在你的应用程序中显示（和处理）。停止相机预览流同样重要：

```java
public void surfaceChanged(SurfaceHolder holder, int format, int w, int h) {
  if (mHolder.getSurface() == null){
    return;
  }
  try {
    mCamera.stopPreview();
  } catch (Exception e){ ...}
  try {       
    mCamera.setPreviewDisplay(mHolder);
    mCamera.startPreview();
  } catch (Exception e){ ... }
}
```

继承的方法`surfaceCreated()`和`surfaceDestroyed()`保持为空。

有了我们定义的`CameraPreview`类，我们可以在`Activity`类中声明它：

```java
private CameraPreview mPreview;
```

然后，在`onResume()`方法中实例化它：

```java
mPreview = new CameraPreview(this, mCamera);
setContentView(mPreview);
```

要测试你的应用程序，你可以对你的其他项目做同样的操作：请通过 USB 线将你的测试设备连接到电脑上。在 Eclipse 中，右键点击你的项目文件夹`CameraAccessAndroid`，在弹出菜单中选择**运行方式** | **1 Android 应用程序**。现在你应该能在应用程序上传并启动后，在手机屏幕上看到实时的相机视图。

# JME 中的实时相机视图

在前面的示例中，你已经了解了如何使用低级图形库（标准的 Android 库）访问 Android 相机。由于我们想要执行增强现实，我们将需要另一种技术将虚拟内容覆盖在视频视图上。有不同的方法可以实现这一点，最好的方法无疑是使用一个公共视图，它将很好地整合虚拟和视频内容。一个强大的技术是使用基于场景图模型的托管 3D 图形库。场景图基本上是一个数据结构，它可以帮助你比在纯 OpenGL®中更容易地构建复杂的 3D 场景，通过逻辑地组织基本构建块，如几何或空间变换。由于你在第一章中安装了 JME，我们将使用这个特定的库，它提供了我们进行 AR 开发所需的所有特性。在本小节中，我们将探讨如何使用 JME 显示视频。与前面的示例不同，相机视图将被整合到 3D 场景图中。为了实现这一点，你需要执行以下步骤：

1.  创建一个支持 JME 的项目。

1.  创建一个设置 JME 的活动。

1.  创建 JME 应用程序，它实际渲染我们的 3D 场景。

要创建支持 JME 的项目，你可以按照第一章中*安装 JMonkeyEngine*一节的说明进行操作，*增强现实概念和工具*。我们将创建一个名为`CameraAccessJME`的新项目。

## 创建 JME 活动

作为一名 Android 开发者，您知道 Android 活动是创建应用程序的主要入口点。然而，JME 是一个平台无关的游戏引擎，可以在支持 Java 的许多平台上运行。JME 的创建者希望尽可能简单地将现有的（和新的）JME 应用程序集成到 Android 中。因此，他们明确区分了实际渲染场景的 JME 应用程序（也可以在其他平台上使用）和 JME 活动中的 Android 特定部分，以设置环境允许 JME 应用程序运行。他们实现这一目标的方法是使用一个特定的类`AndroidHarness`，它减轻了开发者正确配置 Android 活动的负担。例如，它将屏幕上的触摸事件映射到 JME 应用程序中的鼠标事件。这种方法中的一个挑战是将 Android 特定的事件转发到 JME 应用程序中，这些事件在其他平台上并不常见。别担心，我们将向您展示如何为相机图像实现这一点。

您要做的第一件事是创建一个派生自`AndroidHarness`类的 Android 活动，我们将其称为`CameraAccessJMEActivity`方法。它与`CameraAccessAndroidActivity`类相似，包含`Camera`和`CameraPreview`类的实例。不同之处在于，它还将包含您实际的 JME 应用程序实例（将在本章下一节中讨论），负责渲染场景。您尚未提供类的实际实例，只提供了完整的类路径名。在`AndroidHarness`超类中通过反射技术在运行时构造类的实例：

```java
public CameraAccessJMEActivity() {    
  appClass = "com.ar4android.CameraAccessJME";
}
```

在运行时，您可以通过将通用的 JME 应用程序类转换为特定的类来访问实际实例，`AndroidHarness`将其存储在`app`变量中，例如，通过`(com.ar4android.CameraAccessJME)` app。

如本章开头所讨论的，相机可以以各种像素格式提供图像。大多数渲染引擎（JME 也不例外）无法处理各种像素格式，但期望某些格式，如 RGB565。RGB565 格式以 5 位存储红色和蓝色分量，以 6 位存储绿色分量，从而在 16 位每像素的情况下显示 65536 种颜色。您可以在`initializeCameraParameters`方法中通过添加以下代码来检查相机是否支持此格式：

```java
List<Integer> pixelFormats = parameters.getSupportedPreviewFormats();
  for (Integer format : pixelFormats) {
    if (format == ImageFormat.RGB_565) {    
      pixelFormatConversionNeeded = false;
      parameters.setPreviewFormat(format);
      break;
  }
}
```

在这段代码中，我们查询所有可用的像素格式（遍历`parameters.getSupportedPreviewFormats()`）并在支持的情况下设置 RGB565 模型的像素格式（记住，我们是通过对`pixelFormatConversionNeeded`标志进行设置来完成这一步的）。

如前所述，与上一个示例不同，我们不会直接渲染`SurfaceView`类。相反，我们将在每一帧中从摄像头复制预览图像。为此，我们定义了`preparePreviewCallbackBuffer()`方法，你可以在创建摄像头并设置其参数后，在`onResume()`方法中调用它。它分配缓冲区以复制摄像头图像并将其转发给 JME：

```java
public void preparePreviewCallbackBuffer() {    

  mPreviewWidth = mCamera.getParameters().getPreviewSize().width;
  mPreviewHeight = mCamera.getParameters().getPreviewSize().height;
  int bufferSizeRGB565 = mPreviewWidth * mPreviewHeight * 2 + 4096;
  mPreviewBufferRGB565 = null;
  mPreviewBufferRGB565 = new byte[bufferSizeRGB565];
  mPreviewByteBufferRGB565 = ByteBuffer.allocateDirect(mPreviewBufferRGB565.length);
  cameraJMEImageRGB565 = new Image(Image.Format.RGB565, mPreviewWidth, mPreviewHeight, mPreviewByteBufferRGB565);
}
```

如果你的摄像头不支持 RGB565 格式，它可能会以 YCbCr 格式（亮度、蓝色差、红色差）输出帧，这就需要你将其转换为 RGB565 格式。为此，我们将使用一种在 AR 和图像处理中非常常见的色彩空间转换方法。我们在示例项目中提供了一个此方法的实现（`yCbCrToRGB565(…)`）。使用这个方法的基本步骤是创建不同的图像缓冲区，你将在其中复制源图像、中间图像和最终转换后的图像。

因此，在进行转换时，通过在`preparePreviewCallbackBuffer()`方法中调用摄像头实例的`getParameters()`方法来查询`mPreviewWidth`、`mPreviewHeight`和`bitsPerPixel`变量，并确定持有图像数据的字节数组的大小。你将一个 JME 图像（`cameraJMEImageRGB565`）传递给 JME 应用程序，这个图像是由 Java 的`ByteBuffer`类构造的，它只是简单地包装了 RGB565 字节数组。

准备好图像缓冲区后，我们现在需要访问实际图像的内容。在 Android 中，你通过实现`Camera.PreviewCallback`接口来完成这个操作。在这个对象的`onPreviewFrame(byte[] data, Camera c)`方法中，你可以获取到实际存储为字节数组的摄像头图像：

```java
private final Camera.PreviewCallback mCameraCallback = new Camera.PreviewCallback() {
    public void onPreviewFrame(byte[] data, Camera c) {

      mPreviewByteBufferRGB565.clear();
      if(pixelFormatConversionNeeded) {
        yCbCrToRGB565(data, mPreviewWidth, mPreviewHeight, mPreviewBufferRGB565);
        mPreviewByteBufferRGB565.put(mPreviewBufferRGB565);
      }

      cameraJMEImageRGB565.setData(mPreviewByteBufferRGB565);
      if ((com.ar4android.CameraAccessJME) app != null) {
        ((com.ar4android.CameraAccessJME) app).setTexture(cameraJMEImageRGB565);
      }

    }
  }
```

`CameraAccessJME`类的`setTexture`方法仅仅是将传入的数据复制到一个本地图像对象中。

最后，你需要在`CameraPreview`类的`onSurfaceChanged()`方法中注册你的`Camera.PreviewCallback`接口实现：

```java
mCamera.setPreviewCallback(mCameraPreviewCallback);
```

### 注意

一种更快的方法来获取摄像头图像，避免在每一帧都创建新缓冲区，是在之前分配一个缓冲区，并使用`mCamera.addCallbackBuffer()`和`mCamera.setPreviewCallbackWithBuffer()`方法。请注意，这种方法可能与某些设备不兼容。

## 创建 JME 应用程序

如前一部分所述，JME 应用程序是实际进行场景渲染的地方。它不应该关注前面描述的 Android 系统的细节。JME 为你提供了一种方便的方法，使用许多默认设置初始化你的应用程序。你需要做的就是从`SimpleApplication`类继承，在`simpleInitApp()`中初始化你的自定义变量，并在`simpleUpdate()`方法中在渲染新帧之前更新它们。为了我们的渲染相机背景的目的，我们将在`initVideoBackground`方法中创建一个自定义`ViewPort`（显示窗口内的视图）和一个虚拟`Camera`（用于渲染观察到的场景）。在 JME 这样的场景图中显示视频的常见方法是将视频图像作为纹理，放置在一个四边形网格上：

```java
public void initVideoBackground(int screenWidth, int screenHeight){
  Quad videoBGQuad = new Quad(1, 1, true);
  mVideoBGGeom = new Geometry("quad", videoBGQuad);
  float newWidth = 1.f * screenWidth / screenHeight;
  mVideoBGGeom.setLocalTranslation(-0.5f * newWidth, -0.5f, 0.f);mVideoBGGeom.setLocalScale(1.f * newWidth, 1.f, 1);
  mvideoBGMat = new Material(assetManager, "Common/MatDefs/Misc/Unshaded.j3md");
  mVideoBGGeom.setMaterial(mvideoBGMat);
  mCameraTexture = new Texture2D();

  Camera videoBGCam = cam.clone();
  videoBGCam.setParallelProjection(true);
  ViewPort videoBGVP = renderManager.createMainView("VideoBGView",
    videoBGCam);
  videoBGVP.attachScene(mVideoBGGeom);
  mSceneInitialized = true;
}
```

让我们更详细地看看这个为设置视频背景渲染场景图的基本方法。首先，你创建一个四边形形状，并将其分配给一个 JME `Geometry`对象。为了确保屏幕与相机之间的正确映射，你会根据设备屏幕的尺寸缩放和重新定位几何图形。你为四边形分配一个材质，并为它创建一个纹理。由于我们进行的是 3D 渲染，我们需要定义一个观察这个四边形的相机。由于我们希望相机仅在没有失真的情况下，很好地放置在相机前方的四边形，因此我们创建了一个自定义视口和一个正交相机（这种正交相机没有透视缩短效果）。最后，我们将四边形几何添加到这个视口中。

现在，我们的相机观察的是全屏渲染的带纹理四边形。剩下的就是每次相机有新的视频帧可用时，更新四边形的纹理。我们将在`simpleUpdate()`方法中这样做，该方法由 JME 渲染引擎定期调用：

```java
public void simpleUpdate(float tpf) {
if(mNewCameraFrameAvailable) {
  mCameraTexture.setImage(mCameraImage);
  mvideoBGMat.setTexture("ColorMap", mCameraTexture)
}

} 
```

你可能注意到了对`mNewCameraFrameAvailable`变量的条件测试的使用。由于场景图以其内容的不同刷新率（在现代智能手机上高达 60 fps）进行渲染，而移动相机通常能提供的刷新率（通常是 20-30 fps）不同，我们使用`mNewCameraFrameAvailable`标志来更新纹理，仅当有新图像可用时。

这就是全部内容。实施这些步骤后，你可以编译并上传你的应用程序，并应得到与下图类似的结果：

![创建 JME 应用程序](img/8553OS_02_04.jpg)

# 总结

在本章中，您了解了 Android 相机访问的世界以及如何在 JME 3D 渲染引擎中显示相机图像。您学习了各种相机参数以及为了获得有效的相机访问所做出的妥协（例如，在图像大小和每秒帧数之间）。我们还介绍了在 Android 活动中显示相机视图的最简单方法，但也解释了为什么需要超越这个简单示例，将相机视图和 3D 图形在单一应用程序中集成。最后，我们帮助您实现了渲染相机背景的 JME 应用程序。您在本章中获得的知识是为相机视图叠加第一个 3D 对象的有益基础——这是我们将在下一章讨论的主题。
