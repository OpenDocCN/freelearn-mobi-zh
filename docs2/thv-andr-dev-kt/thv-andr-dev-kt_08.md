# 6

# 为 Packtagram 添加视频和编辑功能

在已经掌握了捕捉令人惊叹的照片和用 CameraX 应用迷人滤镜的技艺之后，现在是时候将我们的 Packtagram 应用提升到新的高度了。现在，我们将开始一项激动人心的全新冒险：深入视频的世界。

视频不仅仅是移动的图片；它们是强大的叙事工具，为我们的应用注入生命。它们创造动态交互，保持用户的参与度，并为他们提供一个表达创造力的画布。在本章中，我们将引导你通过将视频功能集成到你的应用中的过程，就像为我们在构建的 Instagram-like 体验中添加一个新维度一样。

我们将首先探讨如何使用 CameraX 库（它是你已熟练掌握的用于照片捕捉技能的扩展）捕捉高质量的视频。然后，我们将深入**快速前进运动图像专家组**（**FFmpeg**）的世界，这是一个用于视频处理的强大库，为你的视频添加创意层次——从传达信息的简单字幕到改变视觉情绪的复杂滤镜。

你将学习如何不仅捕捉和编辑视频，而且高效地将它们上传到 Firebase 存储，确保你的应用能够无缝处理大文件并提供流畅的用户体验。

到本章结束时，你将为你的应用添加一个显著的功能，使其不仅仅是一个照片分享平台，而是一个全面的多媒体体验。

为了实现这一点，在本章中，我们将涵盖以下主题：

+   为我们的应用添加视频功能

+   了解 FFmpeg

+   使用 FFmpeg 为视频添加字幕

+   使用 FFmpeg 为视频添加滤镜

+   上传视频

# 技术要求

如前一章所述，你需要安装 Android Studio（或你偏好的其他编辑器）。

你可以在本书的 GitHub 仓库中找到本章我们将使用的完整代码：[`github.com/PacktPublishing/Thriving-in-Android-Development-using-Kotlin/tree/main/Chapter-6`](https://github.com/PacktPublishing/Thriving-in-Android-Development-using-Kotlin/tree/main/Chapter-6)。

# 为我们的应用添加视频功能

在本节中，我们将扩展我们的 Android 应用功能，使其包括通过 CameraX 的视频捕捉能力。这个强大的库不仅简化了捕捉照片的过程，还提供了一种高效记录视频的方法。我们将首先调整现有的 CameraX 设置，该设置是为捕捉照片设计的，使其也能处理视频录制。目标是提供无缝集成，保持 CameraX 的简单性和健壮性。

首先，我们需要设置视频录制的预览。在上一章中，我们创建了一个`CameraPreview`可组合组件。我们将在这里重用相同的组件：

```kt
@Composable
fun CameraPreview(cameraController:
LifecycleCameraController, modifier: Modifier = Modifier) {
    AndroidView(
        factory = { context ->
            PreviewView(context).apply {
                implementationMode =
                  PreviewView.ImplementationMode.COMPATIBLE
            }
        },
        modifier = modifier,
        update = { previewView ->
            previewView.controller = cameraController
        }
    )
}
```

现在，我们需要创建一个新的按钮组件，用于从预览中记录图像和声音（而不是仅仅捕获图像）：

```kt
@Composable
fun CaptureVideoButton(
    cameraController: LifecycleCameraController,
    onRecordingFinished: (String) -> Unit,
) {
    val context = LocalContext.current
    val recording = remember {
        mutableStateOf<Recording?>(null) }
    IconButton(
        onClick = {
            cameraController.setEnabledUseCases(
                LifecycleCameraController.VIDEO_CAPTURE)
            if (recording.value == null) {
                recording.value =
                    startRecording(cameraController,
                        context, onRecordingFinished)
            } else {
                stopRecording(recording.value)
                recording.value = null
            }
        },
        modifier = Modifier
            .size(60.dp)
            .padding(8.dp),
    ) {
        Icon(
            painter = if (recording.value == null)
                painterResource(id =
                    R.drawable.ic_videocam) else
                        painterResource(id =
                            R.drawable.ic_stop),
            contentDescription = "Capture video",
            tint = MaterialTheme.colorScheme.onPrimary
        )
    }
}
```

在这里，我们正在创建一个新的名为`CaptureVideoButton`的组件。它与`CaptureButton`组件类似，但有一些修改。例如，现在我们需要创建一个可变录制。CameraX 中的`Recording`类负责管理一个活动的视频录制会话。它封装了启动、暂停、恢复和停止录制所需的状态和操作。在我们的代码中，`recording`变量将用于管理当前的录制会话。

一旦用户点击按钮，我们将配置视频捕获用例`cameraController.setEnabledUseCases(LifecycleCameraController.VIDEO_CAPTURE)`，以便`cameraController`可以开始并管理视频录制过程，确保摄像头正确设置以捕获高质量视频，并启用录制会话顺利进行的必要配置和资源。然后，如果没有已经开始录制，我们将启动一个新的录制。如果已经启动，我们将停止它。

在开始录制之前，按钮的图标将显示一个摄像头，如果录制已经开始，则显示一个停止按钮，以提示用户点击它来停止录制。

要完成此录制功能，我们需要实现`startRecording`函数：

```kt
@SuppressLint("MissingPermission")
private fun startRecording(
    cameraController: LifecycleCameraController,
    context: Context,
    onRecordingFinished: (String) -> Unit
): Recording {
    val videoFile = File(context.filesDir,
        "video_${System.currentTimeMillis()}.mp4")
    val outputOptions =
        FileOutputOptions.Builder(videoFile).build()
    val audioConfig = AudioConfig.create(true)
    val executor = Executors.newSingleThreadExecutor()
    return cameraController.startRecording(
        outputOptions,
        audioConfig,
        executor
    ) { recordEvent ->
        when (recordEvent) {
            is VideoRecordEvent.Finalize -> {
                if (recordEvent.hasError()) {
                    Log.e("CaptureVideoButton",
                        "Video recording error:
                            ${recordEvent.error}")
                } else {
                    onRecordingFinished(
                        videoFile.absolutePath)
                }
            }
        }
    }
}
```

此功能使用`@SuppressLint("MissingPermission")`注解标记，表示假设必要的运行时权限，如访问摄像头和麦克风，已经获得授权。我们将以与照片捕获相同的方式处理这些权限，因此在此处使用注解是安全的，因为权限已经获得授权。

函数首先定义视频录制的位置和文件名。它使用`File`类创建一个指向`video_${System.currentTimeMillis()}.mp4`文件的引用，该文件存储在外部存储的应用特定目录中。这种文件存储方法的优势在于它不需要额外的权限，并确保存储的数据对应用程序是私有的。

接下来，代码使用之前定义的文件设置`FileOutputOptions`。这一步至关重要，因为它配置了记录的视频数据将如何写入文件系统。`FileOutputOptions`类是 CameraX 库的一部分，提供了一个直观的 API 来高效地设置这些参数 - 例如，它允许我们使用`ContentResolver`指定视频位置（你可以在[`developer.android.com/reference/androidx/camera/video/FileOutputOptions`](https://developer.android.com/reference/androidx/camera/video/FileOutputOptions)找到有关`FileOutputOptions`的更多信息）。接下来，创建音频配置，在这种情况下，使用`AudioConfig.create(true)`允许音频。

然后，使用`Executors.newSingleThreadExecutor()`创建了一个执行器，这有助于在后台线程中执行任务，从而保持 UI 线程不被阻塞并保持响应。定义了这些参数（`fileOutputOptions`、`AudioConfig`和`Executor`）之后，我们可以执行`cameraController.startRecording`函数，这将启动录制。

此外，使用`Consumer<VideoRecordEvent>`接口定义了一个事件监听器。此监听器使用`when`语句来处理不同类型的`VideoRecordEvent`，例如`VideoRecordEvent.Finalize`，这表示录制的完成。事件监听器还检查录制过程中的错误，确保健壮的错误处理。

然后，返回一个`Recording`对象，代表正在进行的录制会话。这个录制对象对于下一步至关重要。

现在，让我们实现`stopRecording`函数：

```kt
fun stopRecording(recording: Recording?) {
    recording?.stop()
}
```

在这个简洁明了的函数中，我们只有一行代码，但它做了一些基本的事情。该函数接受一个单一参数`recording`，这是我们来自 CameraX 库的`Recording`类的实例。

此函数的核心操作是在`recording`对象上调用`stop()`方法。当此方法被调用时，它告诉`recording`实例终止当前的视频录制会话。这涉及到停止捕获视频帧并最终化正在录制的视频文件。然后，视频文件被保存到录制完成后指定的位置。

现在，我们将新按钮包含到我们之前为捕获功能构建的`CaptureModeContent`中：

```kt
@Composable
private fun CaptureModeContent(
    cameraController: LifecycleCameraController,
    onImageCaptured: (Bitmap) -> Any,
    onVideoCaptured: (String) -> Any
) {
    Box(modifier = Modifier.fillMaxSize()) {
        CameraPermissionRequester {
            Box(
                contentAlignment = Alignment.BottomCenter,
                modifier = Modifier.fillMaxSize()
            ) {
                CameraPreview(...)
                Row {
                    CaptureButton(...)
                    CaptureVideoButton(
                       cameraController =
                           cameraController,
                       onRecordingFinished = { videoPath ->
                           onVideoCaptured(videoPath)
                       }
                    )
                }
            }
        }
    }
}
```

在这里，我们添加了一个`Row`可组合组件来显示两个按钮水平并排。我们还添加了一个新的 Lambda（`onVideoCaptured`），我们将使用它来传递录制完成后视频文件的路径。

经过这些更改，我们应该能够看到新实现的按钮：

![图 6.1：当视频未录制时，视频捕获按钮已集成到 StoryContent 屏幕中](img/B19443_06_01.jpg)

图 6.1：当视频未录制时，视频捕获按钮已集成到 StoryContent 屏幕中

当我们点击视频捕获按钮时，我们应该看到其图标变为停止符号：

![图 6.2：正在进行的视频录制](img/B19443_06_02.jpg)

图 6.2：正在进行的视频录制

并且，有了这个，我们就准备好使用 CameraX 录制视频了！现在，是我们学习如何修改或编辑录制视频的时候了。考虑到这些方面，让我向您介绍 FFmpeg 库。

# 了解 FFmpeg

**FFmpeg**是一个开源的多媒体框架，已成为音频和视频处理领域的基石。以其多功能性和强大功能而闻名，FFmpeg 提供了一套全面的库和工具，用于处理视频、音频和其他多媒体文件和流。在核心上，FFmpeg 是一个命令行工具，使用户能够将媒体文件从一种格式转换为另一种格式，操纵视频和音频记录，并执行一系列其他多媒体处理任务。

注意

您可以在这里找到官方 FFmpeg 文档：[`ffmpeg.org/`](https://ffmpeg.org/)。

在以下子节中，我们将学习 FFmpeg 的组成部分、其关键特性以及如何将其强大的库集成到我们的 Android 应用程序中。

## FFmpeg 的组件

FFmpeg 项目由几个组件组成，每个组件在多媒体处理中扮演着特定的角色：

+   **libavcodec**：一个包含音频/视频编解码器的库

+   **libavformat**：这个库处理容器格式，管理多媒体流的复用和解复用方面

+   **libavutil**：一个提供各种辅助函数和数据结构的实用库

+   **libavfilter**：用于应用各种音频和视频过滤器

+   **libswscale**：专门处理图像缩放和颜色格式转换

这些组件共同为处理各种多媒体处理任务提供了一个坚实的基础。

## FFmpeg 的关键特性

FFmpeg 因其广泛的功能而脱颖而出。其关键特性包括以下内容：

+   **格式支持**：FFmpeg 支持大量的音频和视频格式，包括编码和解码，使其在多媒体处理方面具有极高的灵活性

+   **转换**：它能够以高效率在多种格式之间转换媒体文件，这一特性在各种应用程序和服务中得到广泛应用

+   **流媒体**：FFmpeg 在流媒体能力方面表现出色，允许实时捕获、编码和流式传输音频和视频

+   **过滤**：凭借其强大的过滤能力，用户可以对他们的媒体应用各种转换、叠加和效果

## 将 mobile-ffmpeg 集成到我们的项目中

在 Android 开发的背景下，FFmpeg 可以作为视频编辑功能（如应用过滤器、转码或甚至添加字幕）的强大工具。然而，在使用 C++代码的同时将 FFmpeg 集成到 Android 应用程序中，需要使用`mobile-ffmpeg`。

`mobile-ffmpeg`是专门为移动平台（如 Android 和 iOS）设计的 FFmpeg 的专用端口。它提供了预构建的二进制文件、针对移动特定的 API 和针对移动硬件限制的优化。这使得将 FFmpeg 的强大功能集成到移动应用程序中变得更容易，允许开发者以更少的复杂性利用高级多媒体处理功能。

要将 `mobile-ffmpeg` 库集成到我们的项目中，我们首先需要打开我们的 `libs.versions.toml` 文件。在那里，我们将添加版本和库组以及名称：

```kt
[versions]
...
mobileffmpeg = "4.4"
[libraries]
...
mobileffmpeg = { group = "com.arthenica", name = "mobile-ffmpeg-full", version.ref = "mobileffmpeg" }
```

在这里，我们刚刚将最新的 `mobile-ffmpeg` 版本和库引用添加到我们的版本目录中。

和往常一样，为了在任何模块中使用它，我们将在 `build.gradle.kts` 文件中添加依赖项：

```kt
dependencies {
    ....
    implementation(libs.mobileffmpeg)
}
```

一旦它被添加到我们的依赖项中，我们就需要同步我们的 Gradle 文件，以便它可以在我们的代码中使用。但首先，让我们了解 FFmpeg 的工作原理以及如何使用它。

## 理解 FFmpeg 命令行语法

正如我们所看到的，FFmpeg 是一个强大的多媒体框架，能够解码、编码、转码、复用（例如，将音频和视频合并到单个文件中）、解复用（在不同的文件中分离音频和视频）、流式传输、过滤和播放几乎任何类型的媒体文件。理解其命令行语法对于有效的视频处理至关重要，尤其是在 Android 环境中。

请记住，我们不会在终端中执行这些命令，但 `mobile-ffmpeg` 库使用相同的语法，允许我们通过一个名为 `FFmpeg.execute()` 的函数来执行它们，正如我们现在将要看到的。

在其核心，一个 FFmpeg 命令遵循一个基本结构：

```kt
FFmpeg.execute("[global_options] {[input_file_options] [flags] input_url} ... {[output_file_options] output_url} ...")
```

让我们更详细地看看这个语法的组成部分：

+   **global_options**: 这些是可以应用于整个命令的设置，例如配置日志级别或覆盖默认配置。

+   **input_file_options**: 这些是专门影响输入文件选项，例如格式、编解码器或帧率。

+   **input_url**: 输入文件的路径。

+   **output_file_options**: 这些与输入文件选项类似，但影响输出文件，例如格式、编解码器或比特率。

+   **output_url**: 输出文件的路径。

+   **options**/**flags**: 这些以破折号（**-**）开头，并修改 FFmpeg 处理文件的方式。最常用的选项和标志如下：

    +   **-i**: 指定输入文件

    +   **-c**: 表示编解码器；使用 **-c:v** 用于视频，**-c:a** 用于音频

    +   **-b**: 设置比特率；**-b:v** 用于视频，**-b:a** 用于音频

    +   **-s**: 定义帧大小（分辨率）

    +   **-r**: 设置帧率

    +   **-f**: 表示格式

让我们看看如何使用这个语法来完成一些基本操作。

### 基本转换

将视频文件从一种格式转换为另一种格式是视频编辑中的基本任务。例如，将 MP4 文件转换为 AVI 文件可以这样做：

```kt
FFmpeg.execute("-i input.mp4 output.avi")
```

此命令告诉 FFmpeg 将 `input.mp4` 转换为 `output.avi`，使用编解码器和质量的默认设置（这里使用默认值，因为我们没有指定任何设置）。

### 指定编解码器

**编解码器**是用于编码（压缩）或解码（解压缩）视频和音频流的算法。在 FFmpeg 中，您可以指定文件的视频和音频组件的不同编解码器：

+   **视频编解码器**：视频编解码器处理文件中的视觉数据。选择正确的视频编解码器会影响视频的质量、大小以及与不同播放器和设备的兼容性。

+   **音频编解码器**：音频编解码器处理声音组件。它决定了音频质量、文件大小以及与音频播放系统的兼容性。

在 FFmpeg 中指定编解码器，使用`-c`标志后跟一个冒号，然后是`v`表示视频或`a`表示音频，接着指定编解码器的名称：

```kt
ffmpeg -i input.file -c:v [video_codec] -c:a [audio_codec] output.file
```

例如，要指定 H.264 和 AAC 编解码器，可以运行以下命令：

```kt
ffmpeg -i input.mp4 -c:v libx264 -c:a aac output.mp4
```

让我们了解这个命令的值代表什么：

+   **-i**: 这表示下一个参数将是输入文件。

+   **input.mp4**: 这是输入文件的路径。

+   **-c:v libx264**: 这个值将视频编解码器设置为**libx264**，这是一个流行的 H.264 视频编码编解码器。它以其效率和与大多数视频平台的兼容性而闻名。

+   **-c:a aac**: 这个值将音频编解码器设置为**aac**（代表高级音频编码），在较低的比特率下提供良好的音频质量，使其非常适合网络视频。

+   **output.mp4**: 表示输出文件的路径。

注意，高质量编解码器通常会导致更大的文件大小——质量和文件大小的平衡在特定用例中可能是关键。

此外，重要的是要知道，某些编解码器需要商业使用许可（例如，H.264），而其他则是开源和免费的（例如，VP9 和 Opus）。

### 调整视频质量

在视频处理中，管理输出视频质量是最关键的因素之一。质量通常直接受比特率的影响。**比特率**以**每秒比特数**（**bps**）衡量，表示播放 1 秒视频或音频所编码的数据量。较高的比特率通常意味着更好的质量，但也意味着更大的文件大小。

比特率有两种类型：

+   **恒定比特率**（**CBR**）：在整个文件中以一致的比特率进行编码，导致可预测的文件大小，但可能质量会有所变化

+   **变量比特率**（**VBR**）：这会根据视频每一部分的复杂性调整比特率，更有效地平衡质量和文件大小

要在 FFmpeg 中调整比特率，我们可以使用`-b:v`标志来指定视频比特率，使用`-b:a`来指定音频比特率：

```kt
ffmpeg -i input.file -b:v [video_bitrate] -b:a [audio_bitrate] output.file
```

例如，要设置标准定义视频并具有适中的质量，我们可以运行以下命令：

```kt
ffmpeg -i input.mp4 -b:v 1500k -b:a 128k output.mp4
```

让我们看看这个命令的值代表什么：

+   **-i**: 这表示下一个参数将是输入文件

+   **input.mp4**: 这是输入文件的路径

+   **-b:v 1500k**: 将视频比特率设置为 1,500 kbps，这对于标准定义内容是合适的

+   **-b:a 128k**: 将音频比特率设置为 128 kbps，提供不错的音频质量，而不会导致文件大小过大

+   **output.mp4**: 表示输出文件的路径

值得注意的是，较低的比特率可能会导致明显的压缩伪影，尤其是在快速移动或复杂的场景中。另一方面，较高的比特率提供更好的质量，但代价是文件大小更大，这可能会成为在线流媒体或有限存储的问题。

### 调整视频大小

在视频编辑中，调整或缩放视频是一个常见任务，无论是为了适应不同的屏幕尺寸、减小文件大小还是符合特定的分辨率要求。FFmpeg 提供了强大的工具轻松调整视频大小，但了解这些更改的影响对于保持质量至关重要。

但什么是视频分辨率和宽高比？

+   **分辨率**：视频的分辨率是像素的维度，表示为宽度 x 高度。标准分辨率包括 480p（标清）、720p（高清）、1080p（全高清）和 4K（超高清）。

+   **宽高比**：这是视频宽度和高度的比率。常见的宽高比包括 16:9（宽屏）和 4:3（传统）。

在 FFmpeg 中调整视频大小，使用`-s`（大小）标志。它设置分辨率：

```kt
ffmpeg -i input.file -s [width]x[height] output.file
```

例如，要将分辨率调整为 1080p，命令如下：

```kt
ffmpeg -i input.mp4 -s 1920x1080 output.mp4
```

让我们看看这个命令的值代表什么：

+   **-i**：表示下一个参数将是输入文件

+   **input.mp4**：输入文件的路径

+   **-s 1920x1080**：将视频调整到全高清（1080p），适合高质量演示和大型显示屏

+   **output.mp4**：表示输出文件的路径

在调整视频大小时有一些需要考虑的事项：

+   根据视频将如何以及在哪里被观看来选择分辨率。例如，你应该为电视广播选择高分辨率，而对于网络或移动使用则选择较低的分辨率。

+   较高的分辨率会导致文件更大，这可能会对存储和流媒体传输造成影响。

+   总是考虑源视频的质量。放大低质量素材可能不会产生理想的结果。

现在我们已经熟悉了 FFmpeg 的基本功能，我们将学习高级功能。

## FFmpeg 的高级语法和选项

FFmpeg 的真正力量在于其高级选项，允许对音频和视频文件进行复杂的处理和操作。本节将深入探讨这些高级功能，并提供如何利用它们完成复杂任务的见解。

### 使用过滤器进行增强视频和音频处理

FFmpeg 配备了丰富的视频和音频过滤器。这些过滤器可以应用于裁剪、旋转、添加水印以及调整亮度或对比度等任务。

要应用过滤器，可以使用`-vf`（视频过滤器）或`-af`（音频过滤器）选项。以下是过滤器语法的工作模式：

```kt
ffmpeg -i input.file -vf "[filter1],[filter2]" output.file
```

例如，想象一个场景，你需要裁剪视频并调整其颜色属性。你可以通过运行以下命令来完成：

```kt
ffmpeg -i input.mp4 -vf "crop=640:480:0:0, hue=h=60:s=1" -c:a copy output.mp4
```

让我们更仔细地看看这个命令的值：

+   **-i**：表示下一个参数将是输入文件。

+   **input.mp4**: 这是输入文件的路径。

+   **-vf**: 这代表视频滤镜，允许你将一个或多个滤镜应用到视频流中。

+   **crop=640:480:0:0**: 这是裁剪滤镜。它将视频裁剪到 640 像素宽和 480 像素高。末尾的**0:0**值指定裁剪区域的左上角的 x 和 y 坐标。在这种情况下，它设置为原始视频的左上角。因此，此滤镜实际上从左上角裁剪视频至 640x480 矩形。

+   **hue=h=60:s=1**: 该代码包含两部分：

    +   **h=60**调整视频的色调。色调是允许我们在 360 度色轮上移动颜色的颜色成分。60 度的值将颜色移动 60 度。例如，蓝色可能变成绿色，红色可能变成黄色，依此类推。

    +   **s=1**设置饱和度级别。饱和度为**1**意味着颜色在强度上保持不变。减小此值将使颜色去饱和，导致图像更偏向灰度。

+   **-c:a**: 将视频调整至全高清（1080p）。

+   **output.mp4**: 指示输出文件的路径。

总结来说，这个 FFmpeg 命令读取`input.mp4`，从左上角裁剪视频至 640x480 分辨率，将视频颜色的色调在色轮上调整 60 度，保持原始饱和度，不重新编码复制音频，并将所有这些更改保存在`output.mp4`中。

### 使用覆盖视频滤镜

FFmpeg 中的覆盖滤镜是一个多功能特性，允许用户将一个视频或图像叠加到另一个视频上。这对于添加标志、水印、字幕、画中画效果或任何其他视觉元素到视频中特别有用。

覆盖滤镜可以通过 FFmpeg 中的`-filter_complex`选项应用，该选项用于涉及多个输入流（如合并两个视频或向视频中添加图像）的更复杂过滤。

覆盖滤镜的基本语法如下：

```kt
ffmpeg -i main_video.mp4 -i overlay.mp4 -filter_complex "overlay=x:y" output.mp4
```

在这里，`main_video.mp4`是我们的主要视频，而`overlay.mp4`是我们想要叠加的视频或图像。覆盖滤镜中的`x`和`y`值指定覆盖图像/视频在主视频上的位置。

例如，假设我们想在视频的右下角添加公司标志。首先，我们必须准备文件。在这种情况下，我们有以下文件：

+   主视频文件将是**video.mp4**

+   标志图像将是**logo.png**（最好带有透明背景）

然后，我们将确定标志的位置。标志的位置将取决于主视频的分辨率。例如，如果视频是 1920x1080（全高清），并且你希望标志距离底部和右边框 10 像素，坐标将是（x=1900, y=1060）。

考虑到这一点，我们必须执行以下命令：

```kt
ffmpeg -i video.mp4 -i logo.png -filter_complex "overlay=1900:1060" -codec:a copy output.mp4
```

在此命令中，我们有以下内容：

+   **-i video.mp4**: 指定主视频文件。

+   **-i logo.png**: 指定覆盖层文件（标志）。

+   **-filter_complex "overlay=1900:1060"**: 应用覆盖层过滤器。标志位于（1900,1060），接近右下角。

+   **-codec:a copy**: 复制主视频中的音频，不进行重新编码。

+   **output.mp4**: 在视频上叠加标志的输出文件。

我们能用覆盖层过滤器做到这些吗？不，还有更多！例如，我们可以动态地移动这个覆盖层。

### 使用 FFmpeg 的覆盖层过滤器进行动态定位

FFmpeg 中的覆盖层过滤器不仅允许在主视频上静态放置图像或视频，还提供了动态定位功能。这个高级功能使得覆盖层可以在屏幕上移动或随时间改变其外观，为您的视频添加动态元素。

首先，让我们探索如何在屏幕上创建移动覆盖层的效果。这种技术特别适用于为标志、文本或其他图形元素添加动态效果。

在我们深入命令之前，了解 FFmpeg 如何处理运动表达式非常重要。这些表达式允许覆盖层的位置逐帧改变，从而产生运动错觉。

移动覆盖层的命令如下：

```kt
ffmpeg -i main_video.mp4 -i logo.png -filter_complex "overlay=x='t*100':y=50" output.mp4
```

在这个命令中，我们有以下内容：

+   **x='t*100'**: 覆盖层的水平位置（**x**）从 0 开始，每秒增加 100 像素。**t**变量代表当前时间（秒）。

+   **y=50**: 垂直位置（**y**）固定在帧顶部的**50**像素处。

我们可以调整这些值，在我们的视频覆盖层中引入不同的效果。例如，如果我们创建一个完整的视频编辑器，我们可以允许用户在视频上移动一个元素，并在视频播放期间改变其位置。然后，我们可以使用 FFmpeg 将这些不同的位置映射到我们希望它们移动到的秒数。然而，我们不会这样做，因为这需要另一本书的篇幅！

如果您对此感兴趣，以下是`overlay`参数的文档：[`ffmpeg.org/ffmpeg-filters.html#overlay-1`](https://ffmpeg.org/ffmpeg-filters.html#overlay-1)。

我们还可以使用淡入和淡出效果，这些效果可以应用于我们的覆盖层。让我们看看它是如何工作的。

### 介绍淡入/淡出命令

为了实现淡入/淡出效果，我们将覆盖层过滤器与淡入/淡出过滤器结合使用。让我们分解这个命令，了解其结构：

```kt
ffmpeg -i main_video.mp4 -i logo.png -filter_complex "[1:v]fade=t=in:st=0:d=1,fade=t=out:st=3:d=1[logo];[0:v][logo]overlay=10:10" output.mp4
```

让我们了解这个命令是如何配置的：

+   **[1:v]fade=t=in:st=0:d=1**: 将淡入效果应用于覆盖层，从**0**秒开始，持续**1**秒

+   **fade=t=out:st=3:d=1[logo]**: 随后，从**3**秒开始，淡出效果持续**1**秒

+   **overlay=10:10**: 覆盖层放置在主视频的坐标（10,10）处。

但我们还可以使用 FFmpeg 做更多的事情，而不仅仅是使用公开的过滤器。让我们看看我们如何使用已经集成到我们项目中的`mobile-ffmpeg`库来改进我们正在记录的视频。

## 使用 mobile-ffmpeg 执行 FFmpeg 命令

集成`mobile-ffmpeg`后，在 Android 中执行 FFmpeg 命令变得流程化。

库的`FFmpeg.execute()`方法是运行 FFmpeg 命令的入口。例如，一个如`-i input.mp4 -c:v libx264 output.mp4`的命令，它将输入视频转换为使用 H.264 编解码器，可以在 Android 环境中无缝执行。此函数反映了 FFmpeg 的命令行语法，为熟悉 FFmpeg 命令行界面的人提供了便利。

这就是它的工作方式：

```kt
val command = "-i input.mp4 -c:v libx264 output.mp4"
val returnCode = FFmpeg.execute(command)
```

在前面的代码块中，我们正在构建一个包含`command`指令的字符串，并将其存储在`command`变量中。然后，我们使用`FFmpeg.execute()`方法来执行该命令。请注意，此执行将在当前线程中发生，这在性能方面可能是不理想的。

在 Android 中，管理性能和用户体验至关重要，尤其是在处理像视频处理这样的资源密集型任务时。`mobile-ffmpeg`通过提供命令的异步执行来适应这一点。利用`FFmpeg.executeAsync()`确保长时间操作不会阻塞主线程，从而保持应用程序的响应性。当处理复杂的转换或过滤器，例如缩放视频时，这种方法变得非常有用。

这就是我们可以使用`executeAsync`函数的方式：

```kt
FFmpeg.executeAsync(command) { executionId, returnCode ->
    when (returnCode) {
        Config.RETURN_CODE_SUCCESS -> {
            // Processing was successful
        }
        Config.RETURN_CODE_CANCEL -> {
            // Command execution was cancelled
        }
        else -> {
            // Command execution failed
        }
    }
}
```

在此示例中，`executeAsync()`方法以字符串格式调用`FFmpeg`命令。这是我们打算让 FFmpeg 执行的命令，例如转换视频文件、应用过滤器或任何其他由 FFmpeg 支持的媒体处理任务。此命令的执行在一个单独的线程中发生，防止任何阻塞应用程序的主 UI 线程。

当命令执行完成后，会触发一个 Lambda 函数。此函数的结构是为了接收两个参数：`executionId`和`returnCode`。`executionId`参数是`FFmpeg`命令此特定执行实例的唯一标识符，可以用于跟踪或管理此特定操作，特别是如果我们的应用程序同时处理多个 FFmpeg 进程。

`returnCode`参数至关重要，因为它指示了执行`FFmpeg`命令的结果。不同的返回代码及其含义如下：

+   **Config.RETURN_CODE_SUCCESS**：这个代码表示 FFmpeg 命令成功执行，没有出现任何错误。在相应的 `when` 语句块中，您可能想要实现处理媒体处理任务成功完成的功能。这可能包括更新用户界面、处理或显示输出文件，或触发后续的应用程序逻辑。

+   **Config.RETURN_CODE_CANCEL**：这个返回代码表示 FFmpeg 命令的执行被取消。这可能发生在执行程序被终止或某些外部条件预先停止命令的情况下。处理这个返回代码的代码块可能包括通知用户取消、清理资源或为操作的重试做准备。

+   **else**：这个块捕获所有其他情况，通常表明在执行 FFmpeg 命令期间发生了错误。在这里，错误处理策略包括记录错误以进行诊断、通知用户失败，或在某些条件下尝试重试操作。

为了进一步细化集成，`mobile-ffmpeg` 允许我们处理进度和日志输出。这对于调试和提升用户体验至关重要。以下是它的工作方式：

```kt
FFmpeg.executeAsync(command, ExecuteCallback { executionId,
returnCode ->
    // Handle execution result
}, LogCallback { logMessage ->
    // Handle log message
}, StatisticsCallback { statistics ->
    // Handle progress updates
})
```

在这里，`LogCallback` 补充了我们之前描述的执行回调。FFmpeg 以其详尽的日志记录而闻名，提供了关于正在进行的操作的大量信息。这个回调中的 `logMessage` 参数让您可以访问这些日志，使您可以根据应用程序的需求处理它们。无论是为了调试目的显示这些日志、分析它们以进行详细的错误报告，还是简单地将它们定向到文件以进行记录，这个回调在理解和管理 FFmpeg 操作的复杂性方面发挥着关键作用。

最后但同样重要的是，`StatisticsCallback` 为实时监控 FFmpeg 进程打开了大门。这个回调通过 `statistics` 参数提供实时数据，例如当前正在处理的帧、已过时间、比特率等。利用这些数据可以显著提升用户体验，使您能够实现动态功能，如进度条、预计完成时间指示器，甚至详细报告当前操作的状态。

现在我们已经知道了如何在 Android 中执行 FFmpeg 命令，让我们开始构建一些东西。我们将从给视频添加字幕开始。

# 使用 FFmpeg 给视频添加字幕

在本节中，我们将创建所有需要使用 FFmpeg 添加字幕到视频所需的组件。我们将从这个新功能开始，创建一个用例，其中定义了将字幕添加到视频的业务逻辑。我们将称之为 `AddCaptionToVideoUseCase`，其职责是将字幕添加到视频中，并在添加后返回新的视频文件。

这就是我们构建 `AddCaptionToVideoUseCase` 的方法：

```kt
class AddCaptionToVideoUseCase() {
    suspend fun addCaption(videoFile: File, captionText:
    String): Result<File> = withContext(Dispatchers.IO) {
        val outputFile = File(
            videoFile.parent,
                "${videoFile.nameWithoutExtension}
                    _captioned.mp4")
        val fontFilePath =
            "/system/fonts/Roboto-Regular.ttf"
        val ffmpegCommand = arrayOf(
            "-i", videoFile.absolutePath,
            "-vf", "drawtext=fontfile=$fontFilePath:
                text='$captionText':
                    fontcolor=white:
                        fontsize=24:x=(w-text_w)/2:
                            y=(h-text_h)-10",
            "-c:a", "aac",
            "-b:a", "192k",
            outputFile.absolutePath
        )
        try {
            val executionId =
            FFmpeg.executeAsync(ffmpegCommand)
            { _, returnCode ->
                if (returnCode !=
                Config.RETURN_CODE_SUCCESS) {
                    Result.failure<AddCaptionToVideoError>(
                        AddCaptionToVideoError)
                }
            }
            // Optionally handle the executionId, e.g., for
               cancellation
            Result.success(outputFile)
        } catch (e: Exception) {
            Result.failure(e)
        }
    }
}
object AddCaptionToVideoError: Throwable("There was an
error adding the caption to the video") {
    private fun readResolve(): Any = AddCaptionToVideoError
}
```

在前面的代码中，我们首先创建了一个挂起函数 `addCaption`，它专门设计用来通过协程促进异步执行。由于添加字幕涉及视频处理等密集型任务，我们应该避免在主线程中执行这类逻辑，以防止应用程序出现任何延迟或无响应。该函数接受两个参数：一个表示视频文件的 `File` 对象和一个包含要添加的字幕文本的 `String`。

在 `addCaption` 函数内部，执行上下文切换到 I/O 分派器。这样做是为了优化 I/O 操作，确保文件处理工作负载得到适当的处理，而不会对主线程造成压力。函数接着创建一个 `outputFile` 对象。该对象代表添加字幕后生成的新的视频文件。

函数的下一部分涉及构建 FFmpeg 的命令字符串。这个命令被精心设计，以利用 FFmpeg 的 `drawtext` 过滤器，使得提供的字幕文本可以叠加到视频上。让我们分析一下之前代码块中使用的命令：

```kt
val command = "-i ${videoFile.absolutePath} -vf drawtext=text='$captionText':fontcolor=white:fontsize=24:x=(w-text_w)/2:y=(h-text_h)/2 -codec:a copy ${outputFile.absolutePath}"
```

让我们分解这个命令：

+   **-i ${videoFile.absolutePath}**: 命令的这一部分指定了 FFmpeg 处理的输入文件。**-i** 标志用于 FFmpeg 中的输入文件，而 **${videoFile.absolutePath}** 会动态插入正在处理的视频文件的绝对路径。

+   **-vf drawtext=text='$captionText':...**: **-vf**（视频过滤器）标志用于应用过滤器到视频上。在这里，**drawtext** 过滤器用于向视频中添加文本。

+   **text='$captionText'**: 这指定了要绘制的文本。在这里，**$captionText** 是一个变量，用于存储字幕文本，它将被动态地插入到命令中。

+   **fontcolor=white**: 设置文本的字体颜色为白色。

+   **fontsize=24**: 定义用于文本的字体大小。

+   **x=(w-text_w)/2**: 这设置了文本的水平位置。在这里，**w** 代表视频的宽度，而 **text_w** 是文本的宽度。通过将 **x** 设置为 **(w-text_w)/2**，文本在水平方向上居中。

+   **y=(h-text_h)/2**: 类似地，这设置了文本的垂直位置。在这里，**h** 是视频的高度，而 **text_h** 是文本的高度。这个公式在视频内垂直居中文本。

+   **-codec:a acc**: 命令的这一部分指示 FFmpeg 使用 **acc** 作为音频流的编解码器。

+   **-b:a=192k**：命令的这一部分将比特率设置为 192k。

+   **${outputFile.absolutePath}**：命令的最后部分指定了输出文件的路径，处理并添加了字幕的视频将保存在这里。

使用`FFmpeg.executeAsync()`异步执行这个 FFmpeg 命令。这个方法对于以非阻塞方式运行命令至关重要，并伴随着一个 Lambda 函数来处理执行结果。Lambda 函数评估 FFmpeg 执行中的`returnCode`。在非成功执行（由除`RETURN_CODE_SUCCESS`之外任何返回代码指示）的情况下，函数构建`Result.failure`，并包装一个自定义的`AddCaptionToVideoError`对象。这个自定义错误对象被定义为单例，提供了一个特定的错误消息，指示字幕处理过程中存在问题。

反之，成功执行命令会导致`Result.success`，并传递`outputFile`。这种处理成功和失败场景的分岔确保了健壮的错误管理和关于字幕处理过程结果的清晰反馈。

现在，我们可以在`StoryEditorViewModel`中使用`AddCaptionToVideoUseCase`：

```kt
class StoryEditorViewModel(
    private val saveCaptureUseCase: SaveCaptureUseCase,
    private val addCaptionToVideoUseCase:
    AddCaptionToVideoUseCase
): ViewModel() {
  // Other variables we defined for the photo feature
    var videoFile: File? = null
  // Other code we already added for the photo feature
    fun addCaptionToVideo(captionText: String) {
        videoFile?.let { file ->
            viewModelScope.launch {
                val result =
                    addCaptionToVideoUseCase.addCaption(
                        file, captionText)
                // Handle the result of the captioning
                   process
            }
        }
    }
}
```

我们首先通过构造函数将`AddCaptionToVideoUseCase`注入到`StoryEditorViewModel`中。然后，我们在`ViewModel`中声明一个`videoFile`变量，它持有我们正在处理的视频——它是可空的，因为可能有时我们没有视频来显示或编辑。在`videoFile`中，我们应该存储已经记录的视图。

接下来，这个`ViewModel`的核心函数是`addCaptionToVideo`。这个函数以字幕文本作为输入，并使用我们拥有的视频文件。首先，它会检查`videoFile`是否不是`null`。如果我们有视频，它就会继续进行；如果没有，则什么都不发生。

在`addCaptionToVideo`函数内部，通过在`viewModelScope`中启动协程，我们确保我们的字幕添加过程不会冻结 UI。这对于保持流畅的用户体验至关重要。

然后，我们用视频文件和字幕文本调用我们的用例的`addCaption`方法。无论这个操作返回什么——成功或失败——都会存储在结果中。

`// 处理字幕处理过程的结果`注释是放置我们根据结果更新 UI 的代码的地方。这可能意味着显示带有字幕的视频，显示错误消息，或者对我们应用有意义的任何其他操作。为了简单起见，我们暂时不会在这里添加它，但当我们创建类似 Netflix 的应用时，我们将在本书的最后三章中了解更多关于视频播放的内容。

但我们仍然可以在我们的视频中测试效果。我们只需使用 Android Studio 中的设备资源管理器查看内部应用文件。在那里，我们会看到两个文件——一个是原始视频，另一个是带有`_captioned`后缀的修改版：

![图 6.3：Android Studio 中的设备资源管理器，包含视频文件](img/B19443_06_03.jpg)

图 6.3：Android Studio 中的设备资源管理器，包含视频文件

如果我们下载带有字幕的视频文件并播放，我们应该能看到视频已经添加了字幕：

![图 6.4：带有“这是字幕文本”文字的视频](img/B19443_06_04.jpg)

图 6.4：带有“这是字幕文本”文字的视频

现在我们知道了如何将字幕应用到我们的视频上，让我们看看如何应用过滤器。

# 使用 FFmpeg 为视频添加过滤器

在本节中，我们将学习如何为我们的视频添加过滤器。一个视觉上影响显著的流行过滤器是“渐晕”效果——这种效果通常会使画面的边缘变暗，将观众的注意力引向图像或视频的中心，并可以为视频增添戏剧性或电影般的质感。FFmpeg 具有应用这种艺术过滤器到视频的能力，所以让我们试试吧！

正如我们对字幕所做的那样，我们将首先创建用例：`AddVignetteEffectUseCase`。`AddVignetteEffectUseCase`的主要作用是使用`mobile-ffmpeg`执行将渐晕效果应用到给定视频文件的业务逻辑。我们将使用特定的`FFmpeg`命令，如下所示：

```kt
class AddVignetteEffectUseCase() {
    suspend fun addVignetteEffect(videoFile: File):
    Result<File> = withContext(Dispatchers.IO) {
        val outputFile = File(videoFile.parent,
            "${videoFile.nameWithoutExtension}
                _vignetted.mp4")
        val command = "-i ${videoFile.absolutePath} -vf
            vignette=angle=PI/4 ${outputFile.absolutePath}"
        try {
            val executionId = FFmpeg.executeAsync(command)
            { _, returnCode ->
                if (returnCode !=
                Config.RETURN_CODE_SUCCESS) {
                    Result.failure<AddVignetteEffectError>(
                        AddVignetteEffectError)
                }
            }
            // Optionally handle the executionId, e.g., for
               cancellation
            Result.success(outputFile)
        } catch (e: Exception) {
            Result.failure(e)
        }
    }
}
object AddVignetteEffectError : Throwable("There was an
error adding the vignette effect to the video") {
    private fun readResolve(): Any = AddVignetteEffectError
}
```

让我们逐行分析`AddVignetteEffectUseCase`中的代码。在这里，`addVignetteEffect`是一个挂起函数，意味着它被设计为与 Kotlin 的协程异步运行。在这个函数中，我们获取需要渐晕效果的视频文件，并首先定义处理后的视频的保存位置。我们保留原始视频不变，并为输出创建一个新文件。输出文件的名称保留了原始名称，但添加了`_vignetted`以便于追踪。

接下来，我们构建 FFmpeg 命令。这个命令告诉 FFmpeg 应用渐晕效果。让我们详细看看这个命令（已在之前的代码块中存在）是如何工作的：

```kt
val command = "-i ${videoFile.absolutePath} -vf vignette=angle=PI/4 ${outputFile.absolutePath}"
```

这个命令由以下部分组成：

+   **-i ${videoFile.absolutePath}**：这个命令部分指定了 FFmpeg 要处理的输入文件。**-i**标志用于 FFmpeg 中的输入文件，而**${videoFile.absolutePath}**动态插入要处理的视频文件的绝对路径。简单来说，它告诉 FFmpeg，“这是我想要你处理的视频。”

+   **-vf vignette=angle=PI/4**：这个部分是应用渐晕效果的地方。

+   **-vf**代表视频过滤器，是 FFmpeg 中的一个强大功能，允许你将各种转换或效果应用到你的视频上。

+   **vignette=angle=PI/4**：这是渐晕效果的特定过滤器和设置。FFmpeg 中的渐晕过滤器用于应用渐晕效果，通常会使视频的边缘变暗，以聚焦于中心。**angle=PI/4**部分是渐晕过滤器的参数，用于控制效果的角度。这个特定的设置**PI/4**是为了提供一个视觉上令人愉悦的渐晕效果。这有点像是一种创意选择，平衡了微妙和影响。

+   **${outputFile.absolutePath}**：命令的最后部分指定了处理后的视频的保存位置。它接受你想要保存新视频（应用了图章效果）的路径。通过将此路径放在命令中，你是在告诉 FFmpeg，“一旦你添加了效果，就将新视频保存到这里。”

当运行此命令时，我们使用`FFmpeg.executeAsync`。这个方法很棒，因为它在运行我们的命令时不会阻塞应用程序。该方法还有一个检查是否一切按计划进行的方式。如果命令运行成功，我们返回新图章视频的路径。但如果出现问题，我们会捕获它并返回一个错误。在这里，`AddVignetteEffectError`是我们抛出的自定义错误消息，如果 FFmpeg 命令没有正确执行。这是一个简单的方法，在添加图章效果时，我们可以确切地知道出了什么问题。有了这个，`AddVignetteUseCase`就准备好了。

现在，我们可以将这个用例集成到`StoryEditorViewModel`中：

```kt
class StoryEditorViewModel(
    private val saveCaptureUseCase: SaveCaptureUseCase,
    private val addCaptionToVideoUseCase:
        AddCaptionToVideoUseCase,
    private val addVignetteEffectUseCase:
        AddVignetteEffectUseCase
): ViewModel() {
...
    var videoFile: File? = null
...
    fun addVignetteFilterToVideo() {
        videoFile?.let { file ->
            viewModelScope.launch {
                val result =
                    addVignetteEffectUseCase
                        .addVignetteEffect(file)
                // Handle the result of the filter process
            }
        }
    }
}
```

在这里，`StoryEditorViewModel`被设计为接收`AddVignetteEffectUseCase`作为依赖项。

在这个 ViewModel 中，我们维护一个`videoFile`属性，它包含一个引用，指向将要应用图章效果的视频文件。这个属性的可以为空性允许存在视频文件可能无法立即可用的情况。

执行此功能的函数是`addVignetteEffectToVideo`。当调用此函数时，它会检查`videoFile`是否不为空，确保有一个有效的文件可以处理。如果可用视频文件，函数将继续在`viewModelScope`中启动一个协程。

在协程内部，使用视频文件作为参数调用`addVignetteEffectUseCase.addVignetteEffect`方法。这是图章效果应用到视频上的地方。这个操作的结果被捕获在一个名为`result`的变量中。这个结果可能表明效果成功应用或由于某些错误而失败。

函数内的注释部分`// 处理图章效果处理的结果`是我们通常处理操作结果的地方。根据图章效果是否成功应用，这部分可能包括更新 UI 以显示处理后的视频或处理过程中可能发生的任何错误。

正如我们在讨论添加字幕时提到的，我们还没有实现视频播放，但我们仍然可以在我们的视频中测试效果。就像在*图 6.3*中一样，我们可以看到两个文件，但这次其中一个文件有一个`_vignetted`后缀来表示它已经被修改：

![图 6.5：Android Studio 中的设备资源管理器](img/B19443_06_05.jpg)

图 6.5：Android Studio 中的设备资源管理器

我们可以下载并重新生成这两个视频来检查和测试过滤器：

![图 6.6：应用图章过滤器效果前（左）和后（右）的视频](img/B19443_06_06.jpg)

图 6.6：应用了（右）和未应用（左）渐晕滤镜效果的视频

现在我们已经知道如何集成 FFmpeg 并使用其命令编辑用户的视频，是时候上传这些视频，以便它们可以在他们的联系人之间共享。

# 上传视频

现在视频已经准备好了，是时候将其上传到任何服务，以便用户可以与他们的联系人共享。我们将使用 Firebase 存储来完成这项工作（有关如何设置 Firebase 存储的说明，请参阅*第三章*）。

我们将首先创建一个数据源，负责将视频上传到 Firebase 存储。我们将称之为`VideoStorageDataSource`：

```kt
class VideoStorageDataSource {
    fun uploadVideo(videoFile: File, onSuccess: (String) ->
    Unit, onError: (Exception) -> Unit) {
        val storageReference =
            FirebaseStorage.getInstance().reference
        val videoRef = storageReference.child(
            "videos/${videoFile.name}")
        val uploadTask =
            videoRef.putFile(Uri.fromFile(videoFile))
        uploadTask.addOnSuccessListener {
        videoRef.downloadUrl.addOnSuccessListener { uri ->
            onSuccess(uri.toString())
        }
        }.addOnFailureListener { exception ->
            onError(exception)
        }
    }
}
```

在`uploadVideo`函数内部，我们开始指示我们将执行 I/O 派发器的逻辑。

然后，函数的核心部分是我们使用 Firebase 存储。首先，我们使用`FirebaseStorage.getInstance().reference`获取存储的引用，之后我们设置一个引用，用于在 Firebase 中存储我们的视频，使用`storageReference.child("videos/${videoFile.name}")`。

接下来，我们开始上传本身。使用`putFile`方法上传视频文件。这就是`await()`发挥作用的地方。这个`await()`是一个挂起函数，它耐心地等待上传完成，而不会阻塞线程。它是 Kotlin 协程魔法的一部分，是异步操作的一个变革者。

一旦上传完成，我们需要获取我们视频的 URL。因此，我们调用`downloadUrl.await()`。就像上传一样，`await()`会挂起操作，直到 Firebase 给我们提供视频的 URL。

我们还处理了错误处理。上传和 URL 检索过程被包裹在一个`try-catch`块中。如果在上传或获取 URL 的过程中出现任何问题，我们会捕获异常并将其包裹在`Result.failure(e)`中。另一方面，如果一切顺利，我们返回`Result.success(downloadUrl.toString())`，传递新上传视频的 URL。

接下来，我们将实现一个负责管理并将数据源连接到领域层的仓库。我们将将其接口命名为`VideoRepository`，实现为`VideoRepositoryImpl`：

```kt
interface VideoRepository {
    suspend fun uploadVideo(videoFile: File):
        Result<String>
}
class VideoRepositoryImpl(private val
videoStorageDataSource: VideoStorageDataSource) :
VideoRepository {
    override suspend fun uploadVideo(videoFile: File):
    Result<String> {
        return try {
            var uploadResult: Result<String> =
                Result.failure(RuntimeException("Upload
                    failed"))
            firebaseStorageDataSource.uploadVideo(
            videoFile, { url ->
                uploadResult = Result.success(url)
            }, { exception ->
                uploadResult = Result.failure(exception)
            })
            uploadResult
        } catch (e: Exception) {
            Result.failure(e)
        }
    }
}
```

首先，我们有我们的`VideoRepository`接口。这是一个简单的 Kotlin 接口，包含一个关键功能：`uploadVideo`。

接下来，我们有`VideoRepositoryImpl`类，它实现了`VideoRepository`接口。这个类是动作发生的地方。它使用`VideoStorageDataSource`的实例进行初始化。

然后，`uploadVideo`函数遵循`try-catch`模式以实现健壮的错误处理。最初，它设置一个默认的`uploadResult`为失败。这是一种谨慎的方法，假设事情可能会出错，并且我们只有在上传成功时才会更新它。

然后，我们在`videoStorageDataSource`上调用`uploadVideo`方法，传递视频文件以及两个 Lambda 函数来处理成功和失败的情况。如果上传成功，成功 Lambda 将`uploadResult`更新为已上传视频的 URL。如果出现失败，失败 Lambda 将`uploadResult`更新为遇到的异常。

最后，我们返回`uploadResult`。如果一切顺利，我们将看到已上传视频的 URL。如果不顺利，我们将看到在过程中发生的错误。`try-catch`块确保如果在整个过程中出现任何意外的异常，我们将捕获它并将其作为失败返回。

现在，是我们实现`UploadVideoUseCase`的时候了：

```kt
class UploadVideoUseCase(private val videoRepository:
VideoRepository) {
    suspend fun uploadVideo(videoFile: File):
    Result<String> {
        return videoRepository.uploadVideo(videoFile)
    }
}
```

在这里，我们注入了`VideoRepository`。在`uploadVideo`函数中，我们调用`videoRepository`并传递`videoFile`作为参数。

最后，我们将`UploadVideoUseCase`包含在`StoryEditorViewModel`中，并从那里使用它：

```kt
class StoryEditorViewModel(
private val saveCaptureUseCase: SaveCaptureUseCase,
private val addCaptionToVideoUseCase:
    AddCaptionToVideoUseCase,
private val addVignetteEffectUseCase:
    AddVignetteEffectUseCase,
private val uploadVideoUseCase: UploadVideoUseCase
) : ViewModel() {
...
    fun uploadVideo(videoFile: File) {
        viewModelScope.launch {
            val result =
                uploadVideoUseCase.uploadVideo(videoFile)
            // Handle the result of the upload process
        }
    }
}
```

在`StoryEditorViewModel`中，我们添加了一个名为`uploadVideo`的函数，它接受视频文件并使用`uploadVideoUseCase`来上传它。操作在协程中执行，以确保它不会阻塞 UI 线程。

`// 处理上传过程的结果`注释处是我们根据上传结果实现逻辑的地方。如果上传成功，我们可能会更新 UI 以显示视频已上传或显示视频 URL。在失败的情况下，我们会处理错误，可能通过向用户显示错误消息。

通过这次更改，我们准备好从我们的 ViewModel 上传视频。通过这样做，我们完成了这一章，以及 Packtagram 的工作！

# 概述

总结这一章，你显著提升了 Packtagram 应用的视频功能。

从 CameraX 开始，我们将它的用途从拍照扩展到捕捉高质量视频，但这只是个开始。随后，我们深入研究了 FFmpeg，这是一个功能极其强大的视频编辑工具。在这里，你学习了如何为视频增添创意，无论是通过讲述故事的字幕还是通过改变整体外观和感觉的滤镜。

但如果一部视频不能分享，那它又有什么伟大之处呢？我们也通过集成 Firebase Storage 解决了这个问题，以实现无缝的视频上传。这意味着你的应用现在能够平滑地处理大文件，确保用户享受无故障的体验。

通过这一章，我们完成了 Packtagram 的工作。现在，是时候学习将在最后三章中实现的项目了：一个视频播放应用，这样你就可以观看你最喜欢的电视剧和电影了！
