# 第六章：捕捉和分享

我们喜欢与他人分享我们生活的世界，所以我们将使用我们的智能手机拍摄我们关心的所有事物和所有人的图像或视频。在 Android 上，这相当容易。

在本章中，你将学习以下内容：

+   以简单的方式捕捉图像

+   使用 Camera2 API 进行图像捕捉

+   图像分享

+   方向问题

# 介绍

作为开发者，你可以启动一个意图，获取数据，并对其进行任何你想要的操作。

如果你想自己处理图像或视频捕捉，事情会变得有点复杂。那么，为什么有人要这样做呢？这给了我们更多的灵活性，以处理相机的预览、过滤或处理方式。

从 Android Lollipop 开始，我们一直在使用的旧相机 API 已被 Camera2 API 取代，这被证明是一个巨大的改进。不幸的是，一些方向问题仍然存在，主要是由于 Android 硬件和软件的大碎片化。在一些设备上，捕获的图像似乎被旋转了 90 度。为什么会这样？你将在本章的最后一个配方中找到答案。

# 以简单的方式捕捉图像

当然，在 Android 上有许多拍照或录像的方式。捕捉图像的最简单方式是使用意图启动相机应用程序，并在拍摄完成后获取结果。

## 准备工作

对于这个配方，你只需要运行 Android Studio。

## 如何做...

启动相机意图通常是这样的：

1.  在 Android Studio 中，创建一个新项目。

1.  在`activity_main.xml`布局中，添加一个新按钮和一个图像视图。将图像视图命名为`image`。

1.  为该按钮创建一个点击处理程序。

1.  从事件处理程序实现中调用`takePicture`方法。

1.  实现`takePicture`方法。如果设备支持，启动捕捉意图：

```kt
static final int REQUEST_IMAGE_CAPTURE = 1;
private void takePicture() {
  Intent captureIntent = new  
    Intent(MediaStore.ACTION_IMAGE_CAPTURE);
  if (captureIntent.resolveActivity(  
   getPackageManager()) != null) {
    startActivityForResult(captureIntent,   
       REQUEST_IMAGE_CAPTURE);
   }
}
```

1.  重写`onActivityResult`方法。你将从返回的数据中获取缩略图，并在图像视图中显示结果：

```kt
@Override 
  protected void onActivityResult(int requestCode, int resultCode, Intent data) { 
   if (requestCode == REQUEST_IMAGE_CAPTURE &&resultCode == RESULT_OK) {     
        Bundle extras = data.getExtras();
        Bitmap thumbBitmap = (Bitmap)  
         extras.get("data");");
         ((ImageView)findViewById(R.id.image) 
         ).setImageBitmap(thumbBitmap);
    }
}
```

这是捕捉图像的最简单方式，也许你以前已经这样做过了。

## 还有更多...

如果你想在自己的应用程序中预览图像，还有更多工作要做。Camera2 API 可用于预览、捕捉和编码。

在 Camera2 API 中，你会找到诸如`CameraManager`、`CameraDevice`、`CaptureRequest`和`CameraCaptureSession`之类的组件。

以下是最重要的 Camera2 API 类：

| 类 | 目标 |
| --- | --- |
| `CameraManager` | 选择相机，创建相机设备 |
| `CameraDevice` | `创建 CaptureRequest`，`CameraCaptureSession` |
| `CaptureRequest, CameraBuilder` | 链接到表面视图（预览） |
| `CameraCaptureSession` | 捕捉图像并在表面视图上显示 |

我们将在下一个配方“图像捕捉”中调查的示例可能一开始看起来有点令人困惑。这主要是因为设置过程需要许多步骤，大部分将以异步方式执行。但不要担心，我们将逐步调查它。

# 使用 Camera2 API 进行图像捕捉

让我们与我们所爱的人分享我们周围的世界。一切都始于预览和捕捉。这就是这个配方的全部内容。我们还将回到那些旧日的照片是棕褐色调的好日子。

有许多应用程序，比如 Instagram，提供了添加滤镜或效果到你的照片的选项。如果棕褐色是过滤和分享照片的唯一选项，会发生什么？也许我们可以设置一个趋势。#每个人都喜欢棕褐色！

我们将使用 Camera2 API 来捕捉图像，基于 Google 在 GitHub 上提供的 Camera2 Basic 示例。作为配方步骤的参考，你可以查看以下类图。它将清楚地显示我们正在处理的类以及它们之间的交互方式：

![使用 Camera2 API 进行图像捕捉](img/B04299_06_01.jpg)

我们将调查其中的具体情况，一旦您找出了问题所在，我们将通过使预览和捕获的图像呈现为棕褐色（或者，如果您愿意，可以选择其他效果）来为其添加一些我们自己的东西。

## 准备工作

对于这个示例，我们将使用 Camera2 API。由于我们将使用此 API，您需要使用运行 Android 5.0 或更高版本（推荐）的真实设备，或者您需要创建一个虚拟设备。

## 操作步骤...

让我们看看如何快速上手。Google 已经为我们准备了一个整洁的示例：

1.  在 Android Studio 中，从启动向导中选择**导入 Android 代码示例**，或者在**文件**菜单上选择**导入示例**。

1.  在下一个对话框中，您将看到许多有趣的示例应用程序，展示了各种 Android 功能。选择**Camera2 Basic**示例，然后点击**Next**按钮：![操作步骤...](img/B04299_06_02.jpg)

1.  将项目命名为`EverybodyLovesSepia`，然后点击**Finish**按钮。

### 注意

如果点击按钮后什么都没有发生（由于 Android Studio 的某些版本中存在的错误），请再试一次，但这次保持项目名称不变。

1.  Android Studio 将为您从 GitHub 获取示例项目。您可以在[`github.com/googlesamples/android-Camera2Basic`](https://github.com/googlesamples/android-Camera2Basic)找到它。

1.  在设备上或虚拟设备上运行应用程序。

### 注意

如果您正在使用 Genymotion 上运行的虚拟设备，请首先通过单击右侧的相机图标，打开相机开关，并选择（网络）相机来启用相机。

在应用程序中，您将看到相机的预览，如下截图所示：

![操作步骤...](img/B04299_06_03.jpg)

许多事情又自动发生了！这个 Camera2 API 示例中有什么？需要什么来捕获图像？实际上，需要相当多的东西。打开`Camera2BasicFragment`类。这就是大部分魔术发生的地方。

### 折叠所有方法

为了创建一个不那么压倒性的视图，折叠所有方法：

1.  您可以通过从**Code**菜单中选择**Folding**选项来做到这一点。在子菜单中，选择**Collapse all**。

1.  您还会在此子菜单中找到其他选项；例如，**展开所有**方法或**展开**（仅展开所选方法）。

### 提示

使用快捷键*Cmd*后跟*+*和*Cmd*后跟*–*（或者*Ctrl*后跟*+*和*Ctrl*后跟*–*对于 Windows）来展开或折叠一个方法。使用快捷键*Cmd* + *Shift*后跟*+*和*Cmd* + *Shift*后跟*–*（*Ctrl* + *Shift*和*+*和*Shift* + *Ctrl*和*–*对于 Windows）来展开或折叠类中的所有方法。

1.  展开`onViewCreated`方法。在这里，我们看到了`mTextureView`的初始化，它是对自定义小部件`AutoFitTextureView`的引用。它将显示相机预览。

1.  接下来，展开`onResume`方法。最初，这是设置`SurfaceTextureListener`类的地方。正如示例中的注释已经建议的那样，这允许我们在尝试打开相机之前等待表面准备就绪。双击`mSurfaceTextureListener`，使用快捷键*Cmd* + *B*（对于 Windows，是*Ctrl* + *B*）跳转到其声明，看看这是怎么回事。

1.  完全展开`mSurfaceTextureListener`的初始化。就像活动一样，纹理视图也有一个生命周期。事件在这里被处理。目前，这里最有趣的是`onSurfaceTextureAvailable`事件。一旦表面可用，将调用`openCamera`方法。双击它并跳转到它。

1.  `openCamera`方法中发生了许多事情。调用了`setUpCameraOutputs`方法。此方法将通过设置私有成员`mCameraId`和图像的（预览）大小来处理要使用的相机（如果有多个）。这对于每种类型的设备可能是不同的。它还会处理宽高比。几乎任何设备都支持 4:3 的宽高比，但许多设备也支持 16:9 或其他宽高比。

### 注意

大多数设备都有一到两个摄像头。有些只有一个后置摄像头，有些只有一个前置摄像头。前置摄像头通常支持较少的图像尺寸和宽高比。

另外，随着 Android Marshmallow（Android 6.0）带来的新权限策略，您的应用程序可能根本不被允许使用任何摄像头。这意味着您始终需要测试您的应用程序是否可以使用摄像头功能。如果不能，您将需要通过显示对话框或 toast 向用户提供一些反馈。

1.  接下来，让我们看一下`openCamera`方法中的以下行。它说要打开`setCameraOutputs`方法为我们选择的相机：

```kt
manager.openCamera(mCameraId, mStateCallback, mBackgroundHandler);
```

1.  它还提供了一个`mStateCallback`参数。如果您双击它并跳转到它，您可以看到它的声明。这里的事情再次是异步发生的。

1.  一旦相机被打开，预览会话将会开始。让我们跳转到`createCameraPreviewSession`方法。

1.  看一下`mCameraDevice.createCaptureSession`。进入该方法的一个参数是捕获会话状态回调。它用于确定会话是否成功配置，以便可以显示预览。

1.  现在，需要做什么来拍照？找到`onClick`方法。您会注意到调用`takePicture`方法。跳转到它。`takePicture`方法又调用`lockFocus`方法。跳转到它。

1.  拍照涉及几个步骤。相机的焦点必须被锁定。接下来，需要创建一个新的捕获请求并调用`capture`方法：

```kt
mCaptureSession.capture(mPreviewRequestBuilder.build(),  
 mCaptureCallback, mBackgroundHandler);
```

1.  进入`capture`方法的一个参数是`mCaptureCallback`。使用*Cmd* + *B*（或 Windows 的*Ctrl* + *B*）跳转到它的声明。

1.  您会注意到两个方法：`onCaptureProgressed`和`onCaptureCompleted`。它们都调用私有方法`process`并将结果或部分结果传递给它。

1.  `process`方法将根据各种可能的状态而有所不同。最后，它将调用`captureStillPicture`方法。使用*Cmd* + *B*（或 Windows 的*Ctrl* + *B*）跳转到它的声明。

1.  `captureStillPicture`方法初始化了一个`CaptureRequest.Builder`类，用于拍照并以正确的属性存储照片，例如方向信息。一旦捕获完成并且文件已保存，相机焦点将被解锁，并通过 toast 通知用户：

```kt
CameraCaptureSession.CaptureCallback CaptureCallback= new CameraCaptureSession.CaptureCallback() {
    @Override
    public void onCaptureCompleted 
     (CameraCaptureSession session, 
         CaptureRequest request, TotalCaptureResult  
          result) {
           showToast("Saved: " + mFile);
          unlockFocus();
       }
};
```

前面的步骤向您展示了基本的 Camera2 示例应用程序的亮点。为了在您的应用程序中拍照，需要做相当多的工作！如果您不需要在应用程序中进行预览，您可能希望考虑使用意图来拍照。但是，拥有自己的预览可以为您提供更多的控制和效果的灵活性。

### 添加深褐色效果

我们将在预览中添加一个深褐色效果，只是因为它看起来很酷（当然，一切在早期都更好），使用以下步骤：

1.  转到`createCameraPreviewSession`方法，并在相机捕获会话状态回调实现的`onConfigured`类内部，在设置`autofocus`参数之前添加这一行：

```kt
mPreviewRequestBuilder.set(
 CaptureRequest.CONTROL_EFFECT_MODE,  
  CaptureRequest.CONTROL_EFFECT_MODE_SEPIA);
```

1.  如果您现在运行您的应用程序，您的预览将是深褐色。但是，如果您按下按钮来捕获图像，它将不会产生这种效果。在`onCaptureStillPicture`方法中，您将不得不做同样的事情。在设置`autofocus`参数的行的上面添加这一行：

```kt
captureBuilder.set(   
 CaptureRequest.CONTROL_EFFECT_MODE,  
  CaptureRequest.CONTROL_EFFECT_MODE_SEPIA);
```

再次运行您的应用程序，捕捉一张图像，并使用 Astro 应用程序（或其他文件浏览器应用程序）找到捕捉的文件。您可以在`Android/data/com.example.android.camera2basic`找到它（显然，如果您接受了建议的包名称，否则路径将包括您提供的包名称）。它是泛黄的！

如果您愿意，您还可以尝试一些其他可用效果的负面实验，这也很有趣，至少有一段时间。

目前就是这样。我们还没有做太多的编程，但我们已经看了一些有趣的代码片段。在下一个教程中，我们将在 Facebook 上分享我们捕捉的图像。

## 还有更多...

欲了解更多信息，请访问 GitHub [`github.com/googlesamples/android-Camera2Basic`](https://github.com/googlesamples/android-Camera2Basic) 和 Google Camera2 API 参考 [`developer.android.com/reference/android/hardware/camera2/package-summary.html`](https://developer.android.com/reference/android/hardware/camera2/package-summary.html)。

您可以在[`github.com/ChristianBecker/Camera2Basic`](https://github.com/ChristianBecker/Camera2Basic)找到一个有趣的 Camera2 API 示例的分支，支持 QR 码扫描。

# 图像分享

图像捕捉如果没有分享图像的能力就不好玩；例如，在 Facebook 上。我们将使用 Facebook SDK 来实现这一点。

挑战！如果您正在构建一个在 Parse 后端上运行的应用程序，就像我们在第二章中所做的那样，*云端后端的应用程序*，那就没有必要了，因为 Facebook SDK 已经在其中了。如果您愿意，您可以将第二章的教程与本教程结合起来，快速创建一个真正酷的应用程序！

## 准备工作

对于这个教程，您需要成功完成上一个教程，并且需要有一个真正的 Android 设备（或虚拟设备，但这将需要一些额外的步骤）。

您还需要一个 Facebook 账户，或者您可以只为测试目的创建一个。

## 操作步骤...

让我们看看如何在 Facebook 上分享我们的泛黄捕捉的图像：

1.  从上一个教程中获取代码。打开`app`文件夹中的`build.gradle`文件。在`dependencies`部分添加一个新的依赖项，并在添加了这行代码后点击**立即同步**链接：

```kt
compile 'com.facebook.android:facebook-android-sdk:4.1.0'

```

1.  要获取 Facebook 应用程序 ID，请浏览[`developers.facebook.com`](https://developers.facebook.com)（是的，这需要一个 Facebook 账户）。从**MyApps**菜单中，选择**添加新应用**，选择**Android**作为您的平台，输入您的应用名称，然后点击**创建新的 Facebook 应用程序 ID**。选择一个类别-例如，**娱乐**-然后点击**创建应用程序 ID**。

1.  您的应用程序将被创建，并显示一个快速入门页面。向下滚动到**告诉我们关于您的 Android 项目**部分。在**包名称**和**默认活动类名称**字段中输入详细信息，然后点击**下一步**按钮。

1.  将显示一个弹出警告。您可以放心地忽略警告，然后点击**使用此包名称**按钮。Facebook 将开始思考，一段时间后**添加您的开发和发布密钥哈希**部分将出现。

1.  要获取开发密钥哈希，打开终端应用程序（在 Windows 中，启动命令提示符）并输入以下内容：

```kt
keytool -exportcert -alias androiddebugkey -keystore ~/.android/debug.keystore | openssl sha1 -binary | openssl base64
```

### 提示

如果提示输入密钥库密码，请输入`android`，这应该就可以了 - 除非您之前已更改了密码。

1.  点击*Enter*，复制显示的值，并粘贴到 Facebook 网页的**开发密钥哈希**中。点击**下一步**按钮继续。

1.  在**下一步**部分，点击**跳转到开发者仪表板**按钮。它会直接带你到你需要的信息，即应用 ID。复制**应用 ID**字段中的值：![操作步骤...](img/B04299_06_04.jpg)

1.  接下来，初始化 Facebook SDK。打开`CameraActivity`类，在`onCreate`方法中，在`super.OnCreate`行后添加以下行。使用*Alt* + *Enter*快捷键导入所需的包`com.facebook.FacebookSdk`：

```kt
FacebookSdk.sdkInitialize(getApplicationContext());
```

1.  现在我们需要告诉应用关于 Facebook 应用 ID 的信息。打开`res/values`文件夹中的`strings.xml`文件。添加一个包含你的 Facebook 应用 ID 的新字符串：

```kt
<string name="facebook_app_id">Your facebook app id</string>
```

1.  打开`AndroidManifest.xml`文件。

1.  在`application`元素中添加一个元数据元素：

```kt
<meta-data android:name="com.facebook.sdk.ApplicationId" android:value="@string/facebook_app_id"/>
```

1.  在`manifest`文件中添加一个`FacebookActivity`声明：

```kt
<activity android:name="com.facebook.FacebookActivity"android:configChanges="keyboard|keyboardHidden|screenLayout|   
   screenSize|orientation"
  android:theme="@android:style/Theme.Translucent.
   NoTitleBar"
  android:label="@string/app_name" />
```

1.  在`Camera2BasicFragment`类中，找到`captureStillPicture`方法。在`onCaptureCompleted`回调实现的末尾添加一个新的调用，就在`unlockFocus`类后面：

```kt
sharePictureOnFacebook();
```

1.  最后，在`manifest`文件中的`application`部分添加一个提供者，这将允许你在 Facebook 上分享图片。下一章将讨论内容提供者。现在只需在`authorities`的`FaceBookContentProvider`末尾添加你的应用 ID，替换示例中的零：

```kt
<provider android:authorities="com.facebook.app. 
  FacebookContentProvider000000000000"android:name="com.facebook.FacebookContentProvider"android:exported="true" />
```

1.  实现`sharePictureOnFacebook`方法。我们将从文件中加载位图。在真实的应用中，我们需要计算`inSampleSize`的所需值，但为了简单起见，我们在这里只使用固定的`inSampleSize`设置为`4`。在大多数设备上，这将足以避免其他情况下可能发生的任何`OutOfMemory`异常。此外，我们将在拍照后显示的`share`对话框中添加照片：

```kt
private void sharePictureOnFacebook(){
    final BitmapFactory.Options options = new  
     BitmapFactory.Options();
    options.inJustDecodeBounds = false;
    options.inSampleSize = 4;
    Bitmap bitmap =  
     BitmapFactory.decodeFile(mFile.getPath(), options); 
    SharePhoto photo = new  
    SharePhoto.Builder().setBitmap(bitmap).build();
    SharePhotoContent content = new  
    SharePhotoContent.Builder().addPhoto(photo).build();
    ShareDialog.show(getActivity(), content);
}
```

1.  为了安全起见，我们希望为每张图片创建一个唯一的文件名。修改`onActivityCreated`方法以实现这一点：

```kt
@Override
public void onActivityCreated(Bundle savedInstanceState) {
    super.onActivityCreated(savedInstanceState);
    mFile = new 
    File(getActivity().getExternalFilesDir(null),  
      "pic"+ new Date().getTime()+".jpg");
}
```

1.  在你的 Facebook 时间轴上，页面会显示如下。这里是用荷兰语显示的：![操作步骤...](img/B04299_06_05.jpg)

1.  运行应用程序，在你自己的 Facebook 时间轴上分享一些棕褐色的图片！

我们的应用已经完全可用，尽管可能需要一些调整。在我的三星设备上，我以竖屏模式拍摄的所有图像都旋转了 90 度。这有点太艺术了。让我们在下一个示例中修复它！

# 方向问题

在一些设备上（如三星设备），以竖屏模式捕获的图像会旋转 90 度；而在其他设备上（如 Nexus 设备），情况似乎很好。例如，如果你使用 Astro 应用查看文件，你可能不会注意到这一点，但如果你在 Facebook 的**share**对话框中预览，你就会注意到。

这是许多 Android 开发者都面临的一个众所周知的挑战。图像可能包含有关旋转角度的元数据，但显然并不是每个应用都尊重这些元数据。最好的解决方案是什么？每次显示图像时都应该旋转图像吗？应该旋转位图本身，这可能非常耗时和占用处理器吗？

## 做好准备

对于这个示例，你需要成功完成之前的示例。最好如果你有多个 Android 设备来测试你的应用。否则，如果你至少有一台三星设备可用，那就太好了，因为这个品牌的大多数（如果不是全部）型号都可以重现方向问题。

## 操作步骤

让我们看看如果出现这个方向问题，你如何解决它：

1.  在 Facebook 的**share**对话框中，预览图像会旋转 90 度（在一些设备上），如下所示：![操作步骤...](img/B04299_06_06.jpg)

1.  这看起来不像我生活的世界。在我的三星 Galaxy Note 3 设备上是这样的，但在我的 Nexus 5 设备上不是。显然，三星将图片存储为从横向角度看的样子，然后向其中添加元数据以指示图像已经旋转（与默认方向相比）。然而，如果你想在 Facebook 上分享它，事情就会出错，因为元数据中的方向信息没有得到尊重。

1.  因此，我们需要检查元数据，并找出其中是否有旋转信息。添加`getRotationFromMetaData`方法：

```kt
private int getRotationFromMetaData(){
   try {
      ExifInterface exif = new 
      ExifInterface(mFile.getAbsolutePath());
      int orientation = exif.getAttributeInt(
       ExifInterface.TAG_ORIENTATION,
        ExifInterface.ORIENTATION_NORMAL);
      switch (orientation) {
		  case ExifInterface.ORIENTATION_ROTATE_270:
                return 270;
          case ExifInterface.ORIENTATION_ROTATE_180:
                return 180;case ExifInterface.ORIENTATION_ROTATE_90:
                return 90;
          default:
                return 0;
      }
   }
   catch (IOException ex){
       return 0;
   }
}
```

1.  如果需要，您必须在显示共享预览之前旋转位图。这就是`rotateCaptureImageIfNeeded`方法的用处。

在这里，我们可以安全地在内存中旋转位图，因为`inSampleSet`值为`4`。如果旋转原始全尺寸位图，很可能会耗尽内存。无论哪种方式，都会耗费时间，并导致捕获图像和显示共享预览对话框之间的延迟：

```kt
private Bitmap rotateCapturedImageIfNeeded(Bitmap bitmap){
    int rotate = getRotationFromMetaData();
    Matrix matrix = new Matrix();
    matrix.postRotate(rotate);
    bitmap = Bitmap.createBitmap(bitmap, 0, 0, bitmap.getWidth(),
     bitmap.getHeight(), matrix, true);
    Bitmap mutableBitmap = bitmap.copy(Bitmap.Config.ARGB_8888,  
     true);
   return mutableBitmap;
}
```

1.  然后，在`sharePictureOnFacebook`方法中，在使用`BitmapFactory`类检索位图后，调用`onRotateCaptureImageIfNeeded`方法，并将位图作为参数传递：

```kt
bitmap = rotateCapturedImageIfNeeded(bitmap);

```

1.  如果再次运行应用程序，您会发现在纵向模式下一切都很好：![如何做...](img/B04299_06_07.jpg)

这些东西很容易实现，并且会提高您的应用程序的质量，尽管有时它们也会让您感到困惑，让您想知道为什么一个解决方案不能在任何设备上都正常工作。现在一切看起来都很好，但在平板电脑或华为、LG 或 HTC 设备上会是什么样子呢？没有什么是不能解决的，但由于您没有一堆 Android 设备（或者也许您有），测试是困难的。

尽可能在尽可能多的设备上测试您的应用程序总是一件好事。考虑使用远程测试服务，例如 TestDroid。您可以在[www.testdroid.com](http://www.testdroid.com)找到他们的网站。在第八章中，将讨论这些和其他主题，但首先我们将在即将到来的章节中看一下可观察对象和内容提供程序。

## 还有更多...

拍摄视频更有趣。还有一个用于视频捕获的 Camera2 API 示例可用。您也可以通过**导入示例**选项来检查示例项目。

## 另请参阅

+   第八章, *提高质量*
