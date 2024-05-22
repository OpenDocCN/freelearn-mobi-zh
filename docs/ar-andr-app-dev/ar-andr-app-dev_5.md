# 第五章。与好莱坞相同——物理对象上的虚拟效果

在前一章中，你已经学习了实现基于 GPS 和传感器的 AR 应用程序的基本构建块。如果你尝试了我们提供的不同示例，你可能会注意到将数字对象放置在真实空间中的感觉（*注册*）是可行的，但可能会变得粗糙且不稳定。这主要是由于智能手机或平板电脑中使用的传感器（如 GPS、加速度计等）的准确性问题，以及这些技术的特性（例如，陀螺仪漂移、GPS 对卫星可见性的依赖等）。在本章中，我们将介绍一种更健壮的解决方案，这是支持移动 AR 的第二种主要方法：**基于计算机视觉的 AR**。

基于计算机视觉的 AR 不依赖于任何外部传感器，而是使用摄像头图像的内容来支持跟踪，这是通过不同算法流程的分析。使用基于计算机视觉的 AR，你可以在数字和物理世界之间获得更好的注册效果，尽管在处理上的成本会稍微高一些。

可能你甚至已经不知不觉中见过基于计算机视觉的注册效果。如果你去看一个充满电影特效的大片动作电影，有时你会注意到一些数字内容已经覆盖在物理录制场景上（例如，假的爆炸、假的背景和假的奔跑角色）。与 AR 一样，电影行业也必须处理数字和物理内容之间的注册问题，依靠分析录制的图像来恢复跟踪和相机信息（例如，使用匹配移动技术）。然而，与增强现实相比，它是离线完成的，不是实时完成，通常依赖于重型工作站进行注册和视觉整合。

在本章中，我们将向你介绍不同类型的基于计算机视觉的 AR 跟踪。我们还将为你描述一个广泛使用且高质量的移动 AR 跟踪库——高通公司®的**Vuforia^(TM)**的集成。使用这个库，我们将能够实现我们的第一个基于计算机视觉的 AR 应用程序。

# 介绍基于计算机视觉的跟踪和 Vuforia^(TM)

迄今为止，你一直将手机的摄像头专门用于渲染真实世界的视图，作为你模型的背景。基于计算机视觉的 AR 更进了一步，它处理每一帧图像，寻找摄像头图像中熟悉的*模式*（或图像特征）。

在典型的基于计算机视觉的 AR 应用中，平面对象如*帧标记*或*自然特征追踪目标*被用来在*局部坐标系*中定位摄像头（请参阅第三章，*覆盖世界*，*显示三个最常见的坐标系统的图*）。这与基于传感器的 AR 中使用的全球坐标系（地球）相对立，但允许在此局部坐标框架内更精确和稳定地覆盖虚拟内容。与之前类似，获取追踪信息允许我们更新 3D 图形渲染引擎中虚拟摄像头的相关信息，并自动为我们提供注册。

## 选择物理对象

为了成功实现基于计算机视觉的增强现实（AR），你需要了解哪些物理对象可以用来追踪摄像头。目前主要有两种方法可以实现这一点：帧标记（**Fiducials**）和平面纹理对象作为自然特征追踪目标，如下所示。在下一节中，我们将讨论这两种方法。

![选择物理对象](img/8553_05_01_FINAL.jpg)

### 理解帧标记

在移动增强现实技术的早期，使用计算效率高的算法至关重要。传统上，计算机视觉算法要求较高，因为它们通常依赖于图像分析、复杂的几何算法和数学变换，所有这些操作需要在每一帧内完成（为了保持 30 赫兹的恒定帧率，你只有 33 毫秒的时间）。因此，基于计算机视觉的 AR 的最初方法之一是使用相对简单的对象类型，这些对象可以用计算要求较低的算法检测，例如 Fiducial 标记。这些标记通常只在灰度级别定义，简化了在传统物理世界中的分析和识别（类似于 3D 中的二维码）。

下图展示了一个典型的检测这类标记的算法流程，接下来将对其进行简要说明：

![理解帧标记](img/8553_05_02.jpg)

获取的摄像机图像转换为灰度图像后，将应用**阈值**，即灰度级别被转换为纯黑白图像。下一步是**矩形检测**，在简化后的图像中搜索边缘，然后通过检测封闭轮廓的过程，可能是平行四边形形状。进一步的步骤是为了确保检测到的轮廓确实是一个平行四边形（即它恰好有四个点以及几条平行线）。一旦确认形状，就会分析标记的内容。在**模式检查**步骤中提取标记边框内的（二进制）模式以*识别*标记。这对于能够在不同的标记上叠加不同的虚拟内容非常重要。对于帧标记，使用一个简单的位编码，支持 512 种不同的组合（因此也支持 512 个不同的标记）。

在最后一步中，通过**姿态估计**步骤计算姿态（即摄像机在标记局部坐标系统中的平移和旋转，反之亦然）。

### 注意

姿态计算，在其最简单的形式中是*homography*（两个平面上点之间的映射），可以与内在参数一起使用来恢复摄像机的平移和旋转。

在实际应用中，这不是一次性的计算，而是一个迭代过程，初始姿态会经过多次细化以获得更准确的结果。为了可靠地估计摄像机姿态，至少需要让系统知道标记的一边（宽度或高度）的长度；这通常在加载标记描述时的配置步骤中完成。否则，系统可能无法可靠地判断一个小的标记是近还是大的标记是远（由于透视投影的影响）。

### 理解自然特征跟踪目标

尽管帧标记可以有效地用于许多应用中跟踪摄像机姿态，但你可能希望用不那么显眼的物体进行跟踪。通过使用更高级（但也计算成本更高）的算法，你可以实现这一点。自然特征跟踪的一般思想是使用目标上的多个（理论上只需三个，实际上则需要数十个或数百个）局部点来计算摄像机姿态。挑战在于这些点必须是可靠的，能够健壮地检测并跟踪。这是通过先进的计算机视觉算法来检测和描述**兴趣点**（或特征点）的局部邻域来实现的。兴趣点具有清晰的细节（如角落），例如，使用梯度方向，这适用于由黄色十字标记的特征点。圆形或直线没有清晰的细节，不适合作为兴趣点：

![理解自然特征跟踪目标](img/8553_05_03.jpg)

在纹理丰富的图像上可以找到许多特征点（比如本章中使用的街道图像）：

![理解自然特征跟踪目标](img/8553_05_04.jpg)

请注意，在颜色均匀区域或边缘柔和的图像上（如蓝天或一些计算机图形渲染的图片），特征点无法被很好地识别。

# Vuforia^(TM)架构

Vuforia^(TM)是由高通公司®分发的增强现实库。该库在非商业或商业项目中免费使用。库支持帧标记和自然特征目标跟踪以及多目标，这是多个目标的组合。库还具备基本的渲染功能（视频背景和 OpenGL® 3D 渲染）、线性代数（矩阵/向量变换）以及交互能力（虚拟按钮）。实际上，该库在 iOS 和 Android 平台上都可以使用，并且在配备高通®芯片组的移动设备上性能有所提升。以下图展示了库架构的概览：

![VuforiaTM 架构](img/8553_05_05.jpg)

从客户端视角来看（如前图左边的应用程序框），架构为开发者提供了一个状态对象，其中包含有关已识别目标以及相机内容的信息。这里我们不会详细介绍，因为他们的网站上有一系列示例，以及完整的文档和一个活跃的论坛，请访问[`developer.vuforia.com/`](http://developer.vuforia.com/)。你需要知道的是，该库使用**Android NDK**进行集成，因为它是用 C++开发的。

这主要是因为使用 C++进行图像分析或计算机视觉的高性能计算收益，而不是用 Java（并发技术也采用相同的方法）。这对于我们来说是一个缺点（因为我们只使用 JME 和 Java），但对于你在应用程序中获得性能来说是一个收益。

要使用这个库，你通常需要遵循以下三个步骤：

+   训练并创建你的目标或标记

+   在你的应用程序中集成库

+   部署你的应用程序

在下一节中，我们将介绍如何创建和训练你的目标。

# 配置 Vuforia^(TM)以识别对象

要使用带有自然特征跟踪目标的 Vuforia^(TM)工具包，首先你需要创建它们。在库的最新版本（2.0）中，你可以在应用程序运行时（在线）自动创建你的目标，或者在部署应用程序之前（离线）预先定义它们。我们将向你展示如何离线创建。首先访问 Vuforia^(TM)开发者网站[`developer.vuforia.com`](https://developer.vuforia.com)。

你需要做的第一件事是登录到网站，以访问创建你目标的工具。点击右上角，如果你之前没有做过，请注册。登录后，你可以点击**目标管理器**，这是创建目标的培训计划。目标管理器以数据库的形式组织（可以对应你的项目），对于数据库，你可以创建一个目标列表，如下截图所示：

![配置 VuforiaTM 以识别物体](img/8553_05_07.jpg)

让我们创建第一个数据库。点击**创建数据库**，并输入`VuforiaJME`。你的数据库应该会出现在**设备数据库**列表中。选择它进入下一页：

![配置 VuforiaTM 以识别物体](img/8553_05_08.jpg)

点击**添加新目标**以创建第一个目标。会出现一个对话框，包含不同文本字段以填写，如下截图所示：

![配置 VuforiaTM 以识别物体](img/8553_05_09.jpg)

首先你需要为你的目标选择一个名字；在我们的例子中，我们将其称为`VuforiaJMETarget`。Vuforia^(TM)允许你创建以下不同类型的目标：

+   **单张图片**：你只创建一个平面表面，并且只使用一张图片。目标通常用于在页面、杂志的一部分等上打印。

+   **立方体**：你定义多个表面（带有多张图片），将用于追踪一个 3D 立方体。这可以用于游戏、包装等。

+   **长方体**：这是立方体类型的变化，具有非正方形面的平行六面体。

选择**单张图片**目标类型。目标尺寸为你的标记定义了一个相对比例。单位没有定义，因为它对应于你的虚拟对象的大小。一个很好的建议是考虑所有尺寸都是以厘米或毫米为单位，这通常是你的物理标记的大小（例如，打印在 A4 或信纸上）。在我们的例子中，我们以厘米为单位输入尺寸。最后，你需要选择一个将用于目标的图像。例如，你可以选择`stones.jpg`图像，这是 Vuforia^(TM)示例发行版中提供的（在 Vuforia^(TM)网站上的*ImageTargets*示例的媒体目录中）。为验证你的配置，点击**添加**，然后等待图像处理。处理完成后，你应该会看到一个如下所示的屏幕：

![配置 VuforiaTM 以识别物体](img/8553_05_10.jpg)

星级会告诉你追踪目标的品质如何。这个例子有五颗星，意味着它将非常好用。你可以在 Vuforia^(TM)网站上获取更多信息，了解如何为追踪目标创建一个好的图像：[`developer.vuforia.com/resources/dev-guide/natural-features-and-rating`](https://developer.vuforia.com/resources/dev-guide/natural-features-and-rating)。

现在的最后一步是导出已创建的目标。因此，选择目标（勾选**VuforiaJMETarget**旁边的框），然后点击**下载选择的目标**。在出现的对话框中，选择**SDK**作为导出，**VuforiaJME**作为我们的数据库名称，然后保存。

![配置 VuforiaTM 以识别对象](img/8853_05_11.jpg)

解压你压缩的文件。你会看到两个文件：一个`.dat`文件和一个`.xml`文件。这两个文件都用于在运行时操作 Vuforia^(TM)追踪。`.dat`文件指定了你的图像中的特征点，而`.xml`文件是一个配置文件。有时你可能想要更改标记的大小或进行一些基本编辑，而不必重新启动或进行训练；你可以直接在 XML 文件上进行修改。现在我们已经准备好实现我们的第一个 Vuforia^(TM)项目的目标了！

# 将它们组合在一起——Vuforia^(TM)与 JME

在本节中，我们将向你展示如何将 Vuforia^(TM)与 JME 集成。我们将使用自然特征追踪目标来实现这一目的。因此，在 Eclipse 中打开**VuforiaJME**项目以开始。正如你已经可以观察到的，与我们的前一个项目相比，有两个主要变化：

+   相机预览类已移除

+   项目根目录中有一个名为`jni`的新目录。

第一次更改是由于 Vuforia^(TM)管理相机的方式。Vuforia^(TM)使用自己的相机句柄和集成在库中的相机预览。因此，我们需要通过 Vuforia^(TM)库查询视频图像，以便在我们的场景图中显示（使用与第二章相同的原理，*观看世界*）。

`jni`文件夹包含 C++源代码，这是 Vuforia^(TM)所需的。为了将 Vuforia^(TM)与 JME 集成，我们需要互操作 Vuforia^(TM)的低级别部分（C++）和高级别部分（Java）。这意味着我们将需要编译 C++和 Java 代码并在它们之间传输数据。如果你做到了，你将需要在继续之前下载并安装 Android NDK（如第一章所述，*增强现实概念和工具*）。

## C++集成

C++层基于 Vuforia^(TM)网站上提供的**ImageTargets**示例的修改版本。`jni`文件夹包含以下文件：

+   `MathUtils.cpp`和`MathUtils.h`：用于数学计算的实用功能函数

+   `VuforiaNative.cpp`：这是与我们的 Java 层交互的主要 C++类

+   `Android.mk`和`Application.mk`：这些包含编译配置文件

打开`Android.mk`文件，检查到你的 Vuforia^(TM)安装路径在`QCAR_DIR`目录中是否正确。使用相对路径使其跨平台（在 MacOS 上使用 android ndk r9 或更高版本，绝对路径将与当前目录拼接，导致不正确的目录路径）。

现在打开`VuforiNative.cpp`文件。文件中定义了很多函数，但只有三个与我们有关系：

+   `Java_com_ar4android_VuforiaJMEActivity_loadTrackerData(JNIEnv *, jobject)`: 这是用于加载我们特定目标（在上一节中创建）的函数

+   `virtual void QCAR_onUpdate(QCAR::State& state)`: 这是查询相机图像并将其传递给 Java 层的函数

+   `Java_com_ar4android_VuforiaJME_updateTracking(JNIEnv *env, jobject obj)`: 这个函数用于查询目标的位置并将其传递给 Java 层

第一步将是在我们的应用程序中使用特定目标以及第一个函数。因此，将`VuforiaJME.dat`和`VuforiaJME.xml`文件复制并粘贴到你的资产目录中（应该已经有两个目标配置）。Vuforia^(TM)根据 XML 配置文件配置将使用的目标。`loadTrackerData`首先访问`TrackerManager`和`imageTracker`（用于非自然特征的追踪器）：

```java
JNIEXPORT int JNICALL
Java_com_ar4android_VuforiaJMEActivity_loadTrackerData(JNIEnv *, jobject)
{
    LOG("Java_com_ar4android_VuforiaJMEActivity_ImageTargets_loadTrackerData");

    // Get the image tracker:
    QCAR::TrackerManager& trackerManager = QCAR::TrackerManager::getInstance();
    QCAR::ImageTracker* imageTracker = static_cast<QCAR::ImageTracker*>(trackerManager.getTracker(QCAR::Tracker::IMAGE_TRACKER));
    if (imageTracker == NULL)
    {
        LOG("Failed to load tracking data set because the ImageTracker has not been initialized.");
        return 0;
    }
```

下一步是创建一个特定的目标，比如实例化一个数据集。在这个例子中，创建了一个名为`dataSetStonesAndChips`的数据集：

```java
    // Create the data sets:
    dataSetStonesAndChips = imageTracker->createDataSet();
    if (dataSetStonesAndChips == 0)
    {
        LOG("Failed to create a new tracking data.");
        return 0;
    }
```

在创建的实例中加载目标的配置后，这里是我们设置 VuforiaJME 目标的地方：

```java
    // Load the data sets:
    if (!dataSetStonesAndChips->load("VuforiaJME.xml", QCAR::DataSet::STORAGE_APPRESOURCE))
    {
        LOG("Failed to load data set.");
        return 0;
    }
```

我们可以通过调用`activateDataSet`函数来激活数据集。如果你不激活数据集，目标将在追踪器中加载和初始化，但在激活之前不会被追踪：

```java
    // Activate the data set:
    if (!imageTracker->activateDataSet(dataSetStonesAndChips))
    {
        LOG("Failed to activate data set.");
        return 0;
    }

    LOG("Successfully loaded and activated data set.");
    return 1;
}
```

一旦我们初始化了目标，就需要使用 Vuforia^(TM)获取现实世界的真实视图。这个概念与我们之前看到的相同：在 JME 类中使用视频背景相机并使用图像更新它。然而，在这里，图像不是来自 Java 的`Camera.PreviewCallback`，而是来自 Vuforia^(TM)。在 Vuforia^(TM)中获取视频图像的最佳位置是在`QCAR_onUpdate`函数中。这个函数在追踪器更新后立即被调用。可以通过查询 Vuforia^(TM)的状态对象的帧来获取图像，使用`getFrame()`。一个帧可能包含多个图像，因为相机图像有不同的格式（例如，YUV、RGB888、GREYSCALE、RGB565 等）。在之前的例子中，我们在 JME 类中使用了 RGB565 格式。这里我们也将这样做。所以我们的类将从这里开始。

```java
class ImageTargets_UpdateCallback : public QCAR::UpdateCallback
{   
    virtual void QCAR_onUpdate(QCAR::State& state)
    {
       //inspired from:
       //https://developer.vuforia.com/forum/faq/android-how-can-i-access-camera-image

 QCAR::Image *imageRGB565 = NULL;
        QCAR::Frame frame = state.getFrame();

        for (int i = 0; i < frame.getNumImages(); ++i) {
              const QCAR::Image *image = frame.getImage(i);
              if (image->getFormat() == QCAR::RGB565) {
                  imageRGB565 = (QCAR::Image*)image;

                  break;
              }
        }
```

该函数解析帧中的图像列表并获取`RGB565`图像。一旦我们得到这个图像，我们需要将其传递给**Java 层**。为此，你可以使用 JNI 函数：

```java
        if (imageRGB565) {
            JNIEnv* env = 0;

            if ((javaVM != 0) && (activityObj != 0) && (javaVM->GetEnv((void**)&env, JNI_VERSION_1_4) == JNI_OK)) {

                const short* pixels = (const short*) imageRGB565->getPixels();
                int width = imageRGB565->getWidth();
                int height = imageRGB565->getHeight();
                int numPixels = width * height;

                jbyteArray pixelArray = env->NewByteArray(numPixels * 2);
                env->SetByteArrayRegion(pixelArray, 0, numPixels * 2, (const jbyte*) pixels);
                jclass javaClass = env->GetObjectClass(activityObj);
                jmethodID method = env-> GetMethodID(javaClass, "setRGB565CameraImage", "([BII)V");
                env->CallVoidMethod(activityObj, method, pixelArray, width, height);

                env->DeleteLocalRef(pixelArray);

            }
        }

};
```

在这个例子中，我们获取关于图像大小以及图像原始数据的指针。我们使用名为`setRGB565CameraImage`的 JNI 函数，该函数在我们的`Java Activity`类中定义。我们从 C++中调用这个函数，并传入图像内容（`pixelArray`）作为图像的`width`和`height`。因此，每次追踪器更新时，我们都会获取新的摄像头图像，并通过调用`setRGB565CameraImage`函数将其发送到 Java 层。JNI 机制非常有用，你可以使用它来传递任何数据，从复杂的计算过程回到你的 Java 类（例如，物理，数值模拟等）。

下一步是从追踪中获取目标的位置。我们将在`updateTracking`函数中执行此操作。像之前一样，我们从 Vuforia^(TM)获取 State 对象的实例。State 对象包含`TrackableResults`，这是视频图像中识别的目标列表（在这里被识别为目标及其位置）：

```java
JNIEXPORT void JNICALL
Java_com_ar4android_VuforiaJME_updateTracking(JNIEnv *env, jobject obj)
{
    //LOG("Java_com_ar4android_VuforiaJMEActivity_GLRenderer_renderFrame");

    //Get the state from QCAR and mark the beginning of a rendering section
    QCAR::State state = QCAR::Renderer::getInstance().begin();

    // Did we find any trackables this frame?
    for(int tIdx = 0; tIdx < state.getNumTrackableResults(); tIdx++)
    {
        // Get the trackable:
        const QCAR::TrackableResult* result = state.getTrackableResult(tIdx);
```

在我们的例子中，只有一个目标被激活，所以如果我们得到一个结果，它显然将是我们的标记。然后我们可以直接查询它的位置信息。如果你有多个激活的标记，你将需要通过调用`result->getTrackable()`从结果中获取信息，以确定哪个是哪个。

通过调用`result->getPose()`来查询`trackable`的位置，这将返回一个定义线性变换的矩阵。这个变换可以给出标记相对于摄像头位置的位置。Vuforia^(TM)使用的是计算机视觉坐标系（x 向左，y 向下，z 远离你），这与 JME 不同，因此我们稍后需要进行一些转换。现在，我们首先要做的是反转变换，以得到相对于标记的摄像头位置；这将使标记成为我们虚拟内容的参考坐标系。所以你将进行以下一些基本的数学运算：

```java
        QCAR::Matrix44F modelViewMatrix = QCAR::Tool::convertPose2GLMatrix(result->getPose());

        QCAR::Matrix44F inverseMV = MathUtil::Matrix44FInverse(modelViewMatrix);
        QCAR::Matrix44F invTranspMV = MathUtil::Matrix44FTranspose(inverseMV);

        float cam_x = invTranspMV.data[12];
        float cam_y = invTranspMV.data[13];
        float cam_z = invTranspMV.data[14];

        float cam_right_x = invTranspMV.data[0];
        float cam_right_y = invTranspMV.data[1];
        float cam_right_z = invTranspMV.data[2];
        float cam_up_x = invTranspMV.data[4];
        float cam_up_y = invTranspMV.data[5];
        float cam_up_z = invTranspMV.data[6];
        float cam_dir_x = invTranspMV.data[8];
        float cam_dir_y = invTranspMV.data[9];
        float cam_dir_z = invTranspMV.data[10];
```

现在我们有了摄像头的位置（`cam_x,y,z`）以及摄像头的方向（`cam_right_/cam_up_/cam_dir_x,y,z`）。

最后一步是将这些信息传递到 Java 层。我们将再次使用 JNI 进行此操作。我们还需要的是关于我们摄像头内部参数的信息。这与第三章中讨论的内容相似，*叠加世界*，但现在这里使用 Vuforia^(TM)完成。为此，你可以从`CameraDevice`访问`CameraCalibration`对象：

```java
float nearPlane = 1.0f;
float farPlane = 1000.0f;
const QCAR::CameraCalibration& cameraCalibration = QCAR::CameraDevice::getInstance().getCameraCalibration();
QCAR::Matrix44F projectionMatrix = QCAR::Tool::getProjectionGL(cameraCalibration, nearPlane, farPlane);
```

我们可以轻松地将投影变换转换为更易于阅读的摄像头配置格式，比如其视场（`fovDegrees`），我们也必须调整它以适应摄像头传感器和屏幕的宽高比差异：

```java
        QCAR::Vec2F size = cameraCalibration.getSize();
        QCAR::Vec2F focalLength = cameraCalibration.getFocalLength();
        float fovRadians = 2 * atan(0.5f * size.data[1] / focalLength.data[1]);
        float fovDegrees = fovRadians * 180.0f / M_PI;
        float aspectRatio=(size.data[0]/size.data[1]);

        float viewportDistort=1.0;
        if (viewportWidth != screenWidth)     {
        	viewportDistort = viewportWidth / (float) screenWidth;
            fovDegrees=fovDegrees*viewportDistort;
            aspectRatio=aspectRatio/viewportDistort;
        }
        if (viewportHeight != screenHeight)  {
        	viewportDistort = viewportHeight / (float) screenHeight;
            fovDegrees=fovDegrees/viewportDistort;
            aspectRatio=aspectRatio*viewportDistort;
        }
```

然后，我们调用三个 JNI 函数，将视场（`setCameraPerspectiveNative`）、摄像头位置（`setCameraPoseNative`）和摄像头方向（`setCameraOrientationNative`）传输到我们的 Java 层。这三个函数在 `VuforiaJME` 类中有定义，这使得我们可以快速修改我们的虚拟摄像头：

```java
jclass activityClass = env->GetObjectClass(obj);
        jmethodID setCameraPerspectiveMethod = env->GetMethodID(activityClass,"setCameraPerspectiveNative", "(FF)V");
        env->CallVoidMethod(obj,setCameraPerspectiveMethod,fovDegrees,aspectRatio);
        jmethodID setCameraViewportMethod = env->GetMethodID(activityClass,"setCameraViewportNative", "(FFFF)V");
        env->CallVoidMethod(obj,setCameraViewportMethod,viewportWidth,viewportHeight,cameraCalibration.getSize().data[0],cameraCalibration.getSize().data[1]);
       // jclass activityClass = env->GetObjectClass(obj);
        jmethodID setCameraPoseMethod = env->GetMethodID(activityClass,"setCameraPoseNative", "(FFF)V");
        env->CallVoidMethod(obj,setCameraPoseMethod,cam_x,cam_y,cam_z);

        //jclass activityClass = env->GetObjectClass(obj);
        jmethodID setCameraOrientationMethod = env->GetMethodID(activityClass,"setCameraOrientationNative", "(FFFFFFFFF)V");
        env->CallVoidMethod(obj,setCameraOrientationMethod,cam_right_x,cam_right_y,cam_right_z,
        cam_up_x,cam_up_y,cam_up_z,cam_dir_x,cam_dir_y,cam_dir_z);

    }

    QCAR::Renderer::getInstance().end();
}
```

最后一步将是编译程序。所以，运行一个命令行窗口，前往包含文件的 `jni` 目录。从那里你需要调用 `ndk-build` 函数。该函数在你的 `android-ndk-r9d` 目录中定义，所以确保它可以从你的路径中访问。如果一切顺利，你应该会看到以下内容：

```java
Install        : libQCAR.so => libs/armeabi-v7a/libQCAR.so
Compile++ arm  : VuforiaNative <= VuforiaNative.cpp
SharedLibrary  : libVuforiaNative.so
Install        : libVuforiaNative.so => libs/armeabi-v7a/libVuforiaNative.so
```

是时候回到 Java 了！

## Java 集成

Java 层定义了之前使用与我们的 *Superimpose* 示例相似的类调用的函数。第一个函数是 `setRGB565CameraImage`，它处理视频图像，如之前的例子所示。

其他 JNI 函数将修改我们前台摄像头的特性。具体来说，我们会调整 JME 摄像头的左侧轴，以匹配 Vuforia^(TM) 使用的坐标系（如图中*选择物理对象*一节所示）。

```java
  public void setCameraPerspectiveNative(float fovY,float aspectRatio) {
            fgCam.setFrustumPerspective(fovY,aspectRatio, 1, 1000);
  }  
  public void setCameraPoseNative(float cam_x,float cam_y,float cam_z){
           fgCam.setLocation(new Vector3f(cam_x,cam_y,cam_z));
  }

  public void setCameraOrientationNative(float cam_right_x,float cam_right_y,float cam_right_z,
  float cam_up_x,float cam_up_y,float cam_up_z,float cam_dir_x,float cam_dir_y,float cam_dir_z) {
       //left,up,direction
       fgCam.setAxes(new Vector3f(-cam_right_x,-cam_right_y,-cam_right_z), 
         new Vector3f(-cam_up_x,-cam_up_y,-cam_up_z), 
         new Vector3f(cam_dir_x,cam_dir_y,cam_dir_z));
  } 
```

最后，我们必须调整显示摄像头图像的背景摄像头的视口，以防止 3D 对象漂浮在物理目标之上：

```java
public void setCameraViewportNative(float viewport_w,float viewport_h,float size_x,float size_y) {		
      float newWidth = 1.f;
      float newHeight = 1.f;

      if (viewport_h != settings.getHeight())
      {
        newWidth=viewport_w/viewport_h;
        newHeight=1.0f;
        videoBGCam.resize((int)viewport_w,(int)viewport_h,true);
        videoBGCam.setParallelProjection(true);
      }
      float viewportPosition_x =  (((int)(settings.getWidth()  - viewport_w)) / (int) 2);//+0
      float viewportPosition_y =  (((int)(settings.getHeight() - viewport_h)) / (int) 2);//+0
      float viewportSize_x = viewport_w;//2560
      float viewportSize_y = viewport_h;//1920

      //transform in normalized coordinate
      viewportPosition_x =  (float)viewportPosition_x/(float)viewport_w;
      viewportPosition_y =  (float)viewportPosition_y/(float)viewport_h;
      viewportSize_x = viewportSize_x/viewport_w;
      viewportSize_y = viewportSize_y/viewport_h;

    //adjust for viewport start (modify video quad)
        mVideoBGGeom.setLocalTranslation(-0.5f*newWidth+viewportPosition_x,-0.5f*newHeight+viewportPosition_y,0.f);
    //adust for viewport size (modify video quad)
    mVideoBGGeom.setLocalScale(newWidth, newHeight, 1.f);
  }
```

就这么多。我们再次想要强调的是背后的概念：

+   你的跟踪器中使用的摄像头模型与你的虚拟摄像头（在这个例子中，Vuforia^(TM) 的 `CameraCalibration` 与我们的 JME 虚拟摄像头）相匹配。这将保证我们正确的注册。

+   你在摄像头坐标系中跟踪一个目标（在这个例子中，是来自 Vuforia^(TM) 的自然特征目标）。这种跟踪取代了我们之前看到的 GPS，并使用了一个局部坐标系。

+   这个目标的位置被用来修改你的虚拟摄像头的姿态（在这个例子中，通过 JNI 将检测到的位置从 C++ 传输到 Java，并更新我们的 JME 虚拟摄像头）。由于我们对每一帧都重复这个过程，因此在物理（目标）和虚拟（我们的 JME 场景）之间有一个完整的 6DOF 注册。

你的结果应该与下面这幅图类似：

![Java 集成](img/8553_05_12.jpg)

## 总结

在本章中，我们介绍了基于计算机视觉的 AR。我们使用 Vuforia^(TM) 库开发了一个应用程序，并展示了如何将其与 JME 集成。你现在可以创建基于自然特征跟踪的 AR 应用程序了。在这个演示中，你可以围绕标记移动你的设备，并从各个方向看到虚拟内容。在下一章中，我们将学习如何进行更多交互。比如能够选择模型并与之互动怎么样？
