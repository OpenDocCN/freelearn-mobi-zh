# 第一章：使用命令行工具

在本章中，我们将介绍主要与 Android 应用程序的创建和打包相关的命令行工具。我们将学习如何在 Microsoft Windows、Apple OS X 和 Ubuntu/Debian Linux 上安装和配置 Android NDK，以及如何在 Android 设备上构建和运行你的第一个本地应用程序。使用命令行工具构建项目对于使用 C++进行跨平台移动开发至关重要。

### 注意

本书基于 Android SDK 修订版 24.3.3 和 Android NDK r10e。源代码已使用 Android API 级别 23（Marshmallow）进行测试。

我们的主要关注点将是命令行为中心和平台无关的开发过程。

### 注意

Android Studio 是一个非常不错的新便携式开发 IDE，最近已更新至 1.4 版本。然而，它对 NDK 的支持仍然非常有限，本书将不对其进行讨论。

# 在 Windows 上使用 Android 命令行工具

要在 Microsoft Windows 环境中开始开发 Android 的原生 C++应用程序，你需要在系统上安装一些基本工具。

使用以下所需前提条件的列表开始为 Android 开发 NDK：

+   Android SDK：你可以在[`developer.android.com/sdk/index.html`](http://developer.android.com/sdk/index.html)找到它。我们使用修订版 24。

+   Android NDK：你可以在[`developer.android.com/tools/sdk/ndk/index.html`](http://developer.android.com/tools/sdk/ndk/index.html)找到它。我们使用版本 r10e。

+   **Java 开发工具包**（**JDK**）：你可以在[`www.oracle.com/technetwork/java/javase/downloads/index.html`](http://www.oracle.com/technetwork/java/javase/downloads/index.html)找到它。我们使用 Oracle JDK 版本 8。

+   Apache Ant：你可以在[`ant.apache.org`](http://ant.apache.org)找到它。这是用于构建 Java 应用程序的工具。

+   Gradle：你可以在[`www.gradle.org`](https://www.gradle.org)找到它。与 Ant 相比，这是一个更现代的 Java 构建自动化工具，能够管理外部依赖。

这些工具的当前版本在 Windows 上运行时无需使用任何中间兼容层；它们不再需要 Cygwin。

尽管这让我们感到痛苦，但 Android SDK 和 NDK 仍应安装到不包含空格的文件夹中。这是 Android SDK 内部构建脚本的限制；未加引号的环境变量内容会根据制表符、空格和新行字符分割成单词。

我们将把 Android SDK 安装到`D:\android-sdk-windows`，Android NDK 安装到`D:\ndk`，其他软件安装到它们的默认位置。

为了编译我们可移植的 C++代码以在 Windows 上运行，我们需要一个像样的工具链。我们推荐使用 Equation 软件包提供的最新版 MinGW，可在[`www.equation.com`](http://www.equation.com)获取。你可以根据需要选择 32 位或 64 位版本。

将所有工具放入各自的文件夹后，你需要设置环境变量以指向这些安装位置。`JAVA_HOME` 变量应指向 Java 开发工具包文件夹：

```java
JAVA_HOME="D:\Program Files\Java\jdk1.8.0_25"

```

`NDK_HOME` 变量应指向 Android NDK 安装目录：

```java
NDK_HOME=D:\NDK

```

`ANDROID_HOME` 应指向 Android SDK 文件夹：

```java
ANDROID_HOME=D:\\android-sdk-windows

```

### 注意

注意最后一行中的双反斜杠。

NDK 和 SDK 将会不定期推出新版本，因此如果需要在文件夹名称中包含版本号，并按项目管理 NDK 文件夹可能会有帮助。

# 在 OS X 上使用 Android 命令行工具

在 OS X 上安装 Android 开发工具非常直接。首先，你需要从 [`developer.android.com/sdk/index.html`](http://developer.android.com/sdk/index.html) 下载所需的官方 SDK 和 NDK 包。由于我们使用的是命令行工具，我们可以使用在 [`dl.google.com/android/android-sdk_r24.0.2-macosx.zip`](http://dl.google.com/android/android-sdk_r24.0.2-macosx.zip) 可用的 SDK 工具包。至于 NDK，OS X Yosemite 可以使用 64 位 Android NDK，可以从 [`developer.android.com/tools/sdk/ndk/index.html`](http://developer.android.com/tools/sdk/ndk/index.html) 下载。

我们将所有这些工具安装到用户的 home 文件夹中；在我们的例子中，它是 `/Users/sk`。

要获取 Apache Ant 和 Gradle，最好的方式是安装包管理器 Homebrew，访问 [`brew.sh`](http://brew.sh) 并使用以下命令安装所需的工具：

```java
$ brew install ant
$ brew install gradle

```

这样你就不会被安装路径和其他低级配置问题所困扰。以下是安装包和设置路径的步骤：

### 注意

由于这本书的理念是通过命令行执行操作，我们确实会采取较为复杂的方式。不过，我们建议你实际上在浏览器中访问下载页面，[`developer.android.com/sdk/index.html`](http://developer.android.com/sdk/index.html)，检查 Android SDK 和 NDK 的更新版本。

1.  从官方网站下载适用于 OS X 的 Android SDK 并将其放入你的 home 目录：

    ```java
    >curl -o android-sdk-macosx.zip http://dl.google.com/android/android-sdk_r24.0.2-macosx.zip

    ```

1.  解压它：

    ```java
    >unzip android-sdk-macosx.zip

    ```

1.  然后，下载 Android NDK。它是一个自解压的二进制文件：

    ```java
    >curl -o android-ndk-r10e.bin http://dl.google.com/android/ndk/android-ndk-r10e-darwin-x86_64.bin

    ```

1.  因此，只需将其设置为可执行并运行：

    ```java
    >chmod +x android-ndk-r10e.bin
    >./android-ndk-r10e.bin

    ```

1.  包已就位。现在，在你的 home 目录中的 `.profile` 文件中添加工具的路径以及所有必要的环境变量：

    ```java
    export PATH=/Users/sk/android-ndk-r10e:/Users/sk/android-ndk-r10e/prebuilt/darwin-x86_64/bin:/Users/sk/android-sdk-macosx/platform-tools:$PATH

    ```

1.  在 Android 脚本和工具中使用这些变量：

    ```java
    export NDK_ROOT="/Users/sk/android-ndk-r10e"
    export ANDROID_SDK_ROOT="/Users/sk/android-sdk-macosx"

    ```

1.  编辑 `local.properties` 文件以按项目设置路径。

# 在 Linux 上使用 Android 命令行工具

在 Linux 上的安装与 OS X 一样简单。

### 注意

实际上，由于所有工具链和 Android 开源项目都基于 Linux 工具，Linux 开发环境确实是所有类型 Android 开发的原生环境。

在这里，我们仅指出一些不同之处。首先，我们不需要安装 Homebrew。只需使用可用的包管理器。在 Ubuntu 上，我们更愿意使用 `apt`。以下是安装包以及设置 Linux 上的路径的步骤：

1.  首先，我们来更新所有的 `apt` 包并安装默认的 Java 开发工具包：

    ```java
    $ sudo apt-get update
    $ sudo apt-get install default-jdk

    ```

1.  安装 Apache Ant 构建自动化工具：

    ```java
    $ sudo apt-get install ant

    ```

1.  安装 Gradle：

    ```java
    $ sudo apt-get install gradle

    ```

1.  从 [`developer.android.com/sdk/index.html`](http://developer.android.com/sdk/index.html) 下载适合你 Linux 版本的官方 Android SDK，并将其解压到你的主目录下的一个文件夹中：

    ```java
    $ wget http://dl.google.com/android/android-sdk_r24.0.2-linux.tgz
    $ tar –xvf android-sdk_r24.0.2-linux.tgz

    ```

1.  下载适合你 Linux 系统（32 位或 64 位）的官方 NDK 包并运行它：

    ```java
    $ wget http://dl.google.com/android/ndk/android-ndk-r10e-linux-x86_64.bin
    $ chmod +x android-ndk-r10e-linux-x86_64.bin
    $ ./android-ndk-r10e-linux-x86_64.bin

    ```

    该可执行文件将把 NDK 包的内容解压到当前目录。

1.  现在，你可以设置环境变量以指向实际的文件夹：

    ```java
    NDK_ROOT=/path/to/ndk
    ANDROID_HOME=/path/to/sdk

    ```

    ### 注意

    将环境变量定义添加到 `/etc/profile` 或 `/etc/environment` 中很有用。这样，这些设置将适用于系统的所有用户。

# 手动创建基于 Ant 的应用程序模板

让我们从最低级别开始，创建一个可使用 Apache Ant 构建的应用程序模板。每个要使用 Apache Ant 构建的应用程序都应包含预定义的目录结构和配置 `.xml` 文件。这通常使用 Android SDK 工具和 IDE 完成。我们将解释如何手动完成，以让你了解幕后的机制。

### 提示

**下载示例代码**

你可以从 [`www.packtpub.com`](http://www.packtpub.com) 的账户下载你购买的所有 Packt Publishing 书籍的示例代码文件。如果你在其他地方购买了这本书，可以访问 [`www.packtpub.com/support`](http://www.packtpub.com/support) 并注册，我们会直接将文件通过电子邮件发送给你。

对于这本书，源代码文件也可以从以下 GitHub 仓库下载或派生：[`github.com/corporateshark/Mastering-Android-NDK`](https://github.com/corporateshark/Mastering-Android-NDK)

我们最小化项目的目录结构如下截图所示（完整的源代码请参见源代码包）：

![手动创建基于 Ant 的应用程序模板](img/image00212.jpeg)

我们需要在此目录结构中创建以下文件：

+   `res/drawable/icon.png`

+   `res/values/strings.xml`

+   `src/com/packtpub/ndkmastering/App1Activity.java`

+   `AndroidManifest.xml`

+   `build.xml`

+   `project.properties`

图标 `icon.png` 应该在那里，目前包含一个安卓应用程序的示例图像：

![手动创建基于 Ant 的应用程序模板](img/image00213.jpeg)

文件`strings.xml`是使用 Android 本地化系统所必需的。在`AndroidManifest.xml`清单文件中，我们使用字符串参数`app_name`而不是实际的应用程序名称。文件`strings.xml`将此参数解析为人类可读的字符串：

```java
<?xml version="1.0" encoding="utf-8"?>
<resources>
  <string name="app_name">AntApp1</string>
</resources>
```

最小可构建应用程序的 Java 源代码在`App1Activity.java`文件中：

```java
package com.packtpub.ndkmastering;
import android.app.Activity;
public class App1Activity extends Activity
{
};
```

其他三个文件`AndroidManifest.xml`、`build.xml`和`project.properties`，包含了 Ant 构建项目所需的描述。

清单文件`AndroidManifest.xml`如下所示：

```java
<?xml version="1.0" encoding="utf-8"?>
<manifest 
package="com.packtpub.ndkmastering"
android:versionCode="1"
android:versionName="1.0.0">
```

我们的应用程序将需要 Android 4.4（API 级别 19），并且已经在 Android 6.0（API 级别 23）上进行了测试：

```java
<uses-sdk android:minSdkVersion="19" android:targetSdkVersion="23" />
```

本书中的大多数示例将需要 OpenGL ES 3。在此提及一下：

```java
<uses-feature android:glEsVersion="0x00030000"/>
<application android:label="@string/app_name"
android:icon="@drawable/icon"
android:installLocation="preferExternal"
android:largeHeap="true"
android:allowBackup="true">
```

这是主活动的名称：

```java
<activity android:name="com.packtpub.ndkmastering.App1Activity"
android:launchMode="singleTask"
```

我们希望应用程序在全屏模式下，且为横屏方向：

```java
android:theme="@android:style/Theme.NoTitleBar.Fullscreen"
android:screenOrientation="landscape"
```

我们的应用程序可以从系统启动器中启动。应用程序的可显示名称存储在`app_name`参数中：

```java
android:configChanges="orientation|keyboardHidden"
android:label="@string/app_name">
<intent-filter>
  <action android:name="android.intent.action.MAIN" />
  <category android:name="android.intent.category.LAUNCHER" />
</intent-filter>
</activity>
</application>
</manifest>
```

### 注意

你可以在[`developer.android.com/guide/topics/manifest/manifest-intro.html`](http://developer.android.com/guide/topics/manifest/manifest-intro.html)阅读官方关于应用程序清单的 Google 文档。

文件`build.xml`要简单得多，主要与 Android 工具生成的类似：

```java
<?xml version="1.0" encoding="UTF-8"?>
<project name="App1" default="help">
  <loadproperties srcFile="project.properties" />
  <fail message="sdk.dir is missing. Make sure to generate local.properties using 'android update project' or to inject it through an env var"
    unless="sdk.dir"/>
  <import file="${sdk.dir}/tools/ant/build.xml" />
</project>
```

与 Android SDK Tools 相比，这里我们没有使用`ant.properties`。这样做只是为了简单起见，仅具有教育目的。

文件`project.properties`同样包含特定平台的声明，情况类似：

```java
target=android-19
sdk.dir=d:/android-sdk-windows

```

现在，我们的第一个应用程序（甚至还没有包含任何本地代码）已经准备好构建了。使用以下命令行构建它：

```java
$ ant debug

```

如果一切操作都正确，你应该会看到类似于以下的输出尾部：

![手动创建基于 Ant 的应用程序模板](img/image00214.jpeg)

要从命令行安装`.apk`文件，请运行`adb install -r bin/App1-debug.apk`以将新构建的`.apk`安装到你的设备上。从启动器（**AntApp1**）启动应用程序，并享受黑色的屏幕。你可以使用**BACK**键退出应用程序。

# 手动创建基于 Gradle 的应用程序模板

相比于 Ant，Gradle 是一个更加多功能的 Java 构建工具，它能轻松地处理外部依赖和仓库。

### 注意

我们建议在继续使用 Gradle 之前，观看 Google 提供的[`www.youtube.com/watch?v=LCJAgPkpmR0`](https://www.youtube.com/watch?v=LCJAgPkpmR0)这个视频，并阅读官方的命令行构建手册[`developer.android.com/tools/building/building-cmdline.html`](http://developer.android.com/tools/building/building-cmdline.html)。

近期的 Android SDK 版本与 Gradle 紧密集成，Android Studio 就是使用它作为其构建系统的。让我们扩展之前的`1_AntApp`应用程序，使其能够用 Gradle 构建。

首先，进入项目的根目录，并创建一个包含以下内容的`build.gradle`文件：

```java
buildscript {
  repositories {
    mavenCentral()
  }
  dependencies {
    classpath 'com.android.tools.build:gradle:1.0.0'
  }
}
apply plugin: 'com.android.application'
android {
  buildToolsVersion "19.1.0"
  compileSdkVersion 19
  sourceSets {
    main {
      manifest.srcFile 'AndroidManifest.xml'
      java.srcDirs = ['src']
      resources.srcDirs = ['src']
      aidl.srcDirs = ['src']
      renderscript.srcDirs = ['src']
      res.srcDirs = ['res']
      assets.srcDirs = ['assets']
    }
  }
  lintOptions {
    abortOnError false
  }
}
```

完成后，运行命令`gradle init`。输出结果应类似于以下内容：

```java
>gradle init
:init
The build file 'build.gradle' already exists. Skipping build initialization.
:init SKIPPED
BUILD SUCCESSFUL
Total time: 5.271 secs

```

当前文件夹中将创建`.gradle`子文件夹。现在，运行以下命令：

```java
>gradle build

```

输出的末尾应如下所示：

```java
:packageRelease
:assembleRelease
:assemble
:compileLint
:lint
Ran lint on variant release: 1 issues found
Ran lint on variant debug: 1 issues found
Wrote HTML report to file:/F:/Book_MasteringNDK/Sources/Chapter1/2_GradleApp/build/outputs/lint-results.html
Wrote XML report to F:\Book_MasteringNDK\Sources\Chapter1\2_GradleApp\build\outputs\lint-results.xml
:check
:build
BUILD SUCCESSFUL
Total time: 9.993 secs

```

生成的`.apk`包可以在`build\outputs\apk`文件夹中找到。尝试在您的设备上安装并运行`2_GradleApp-debug.apk`。

# 嵌入本地代码

让我们继续这本书的主题，为我们的模板应用程序编写一些本地 C++代码。我们将从包含单个函数定义的`jni/Wrappers.cpp`文件开始：

```java
#include <stdlib.h>
#include <jni.h>
#include <android/log.h>
#define LOGI(...) ((void)__android_log_print(ANDROID_LOG_INFO, "NDKApp", __VA_ARGS__))
extern "C"
{
  JNIEXPORT void JNICALL Java_com_packtpub_ndkmastering_AppActivity_onCreateNative( JNIEnv* env, jobject obj )
  {
    LOGI( "Hello Android NDK!" );
  }
}
```

这个函数将通过 JNI 机制从 Java 中调用。如下更新`AppActivity.java`：

```java
package com.packtpub.ndkmastering;
import android.app.Activity;
import android.os.Bundle;
public class AppActivity extends Activity
{
  static
  {
    System.loadLibrary( "NativeLib" );
  }
  @Override protected void onCreate( Bundle icicle )
  {
    super.onCreate( icicle );
    onCreateNative();
  }
  public static native void onCreateNative();
};
```

现在，我们需要将这段代码构建成一个可安装的`.apk`包。为此我们需要几个配置文件。第一个是`jni/Application.mk`，它包含平台和工具链信息：

```java
APP_OPTIM := release
APP_PLATFORM := android-19
APP_STL := gnustl_static
APP_CPPFLAGS += -frtti
APP_CPPFLAGS += -fexceptions
APP_CPPFLAGS += -DANDROID
APP_ABI := armeabi-v7a-hard
APP_MODULES := NativeLib
NDK_TOOLCHAIN_VERSION := clang
```

我们使用最新版本的 Clang 编译器——即在我们编写这些内容时的 3.6 版本，以及`armeabi-v7a-hard`目标，它支持硬件浮点计算和通过硬件浮点寄存器传递函数参数，从而实现更快的代码。

第二个配置文件是`jni/Android.mk`，它指定了我们想要编译的`.cpp`文件以及应使用的编译器选项：

```java
TARGET_PLATFORM := android-19
LOCAL_PATH := $(call my-dir)
include $(CLEAR_VARS)
LOCAL_MODULE := NativeLib
LOCAL_SRC_FILES += Wrappers.cpp
LOCAL_ARM_MODE := arm
COMMON_CFLAGS := -Werror -DANDROID -DDISABLE_IMPORTGL
ifeq ($(TARGET_ARCH),x86)
  LOCAL_CFLAGS := $(COMMON_CFLAGS)
else
  LOCAL_CFLAGS := -mfpu=vfp -mfloat-abi=hard -mhard-float -fno-short-enums -D_NDK_MATH_NO_SOFTFP=1 $(COMMON_CFLAGS)
endif
LOCAL_LDLIBS := -llog -lGLESv2 -Wl,-s
LOCAL_CPPFLAGS += -std=gnu++11
include $(BUILD_SHARED_LIBRARY)
```

在这里，我们链接到 OpenGL ES 2，为非 x86 目标启用硬件浮点数的编译器开关，并列出所需的`.cpp`源文件。

使用以下命令从项目根目录构建本地代码：

```java
>ndk-build

```

输出结果应如下所示：

```java
>ndk-build
[armeabi-v7a-hard] Compile++ arm  : NativeLib <= Wrappers.cpp
[armeabi-v7a-hard] SharedLibrary  : libNativeLib.so
[armeabi-v7a-hard] Install        : libNativeLib.so => libs/armeabi-v7a/libNativeLib.so

```

最后，我们需要告诉 Gradle，我们希望将生成的本地库打包进`.apk`。编辑`build.gradle`文件，在`sourceSets`的`main`部分添加以下行：

```java
jniLibs.srcDirs = ['libs']

```

现在，如果我们运行命令`gradle build`，生成的包`build\outputs\apk\3_NDK-debug.apk`将包含所需的`libNativeLib.so`文件。您可以像往常一样安装并运行它。使用`adb logcat`检查 Android 系统日志中打印的**Hello Android NDK!**这一行。

### 注意

那些不想在这样的小项目中处理 Gradle 的人可以使用古老的 Apache Ant。只需运行命令`ant debug`即可实现。这种方式不需要额外的配置文件将共享的 C++库放入`.apk`。

# 构建并签署发布版的 Android 应用

我们已经学习了如何使用命令行创建带有本地代码的 Android 应用。让我们在命令行工具的话题上画上圆满的句号，学习如何准备并签署应用程序的发布版本。

关于在 Android 上签名过程的详细解释，可以在开发者手册中找到，地址是 [`developer.android.com/tools/publishing/app-signing.html`](http://developer.android.com/tools/publishing/app-signing.html)。让我们使用 Ant 和 Gradle 来完成签名。

首先，我们需要重新构建项目并创建 `.apk` 包的发布版本。让我们用 `3_NDK` 项目来做这件事。我们使用以下命令调用 `ndk-build` 和 Apache Ant：

```java
>ndk-build
>ant release

```

Ant 输出的末尾如下所示：

```java
-release-nosign:
[echo] No key.store and key.alias properties found in build.properties.
[echo] Please sign F:\Book_MasteringNDK\Sources\Chapter1\3_NDK\bin\App1-release-unsigned.apk manually
[echo] and run zipalign from the Android SDK tools.
[propertyfile] Updating property file: F:\Book_MasteringNDK\Sources\Chapter1\3_NDK\bin\build.prop
[propertyfile] Updating property file: F:\Book_MasteringNDK\Sources\Chapter1\3_NDK\bin\build.prop
[propertyfile] Updating property file: F:\Book_MasteringNDK\Sources\Chapter1\3_NDK\bin\build.prop
[propertyfile] Updating property file: F:\Book_MasteringNDK\Sources\Chapter1\3_NDK\bin\build.prop
-release-sign:
-post-build:
release:
BUILD SUCCESSFUL
Total time: 2 seconds

```

让我们用 Gradle 做同样的事情。也许您已经注意到，当我们运行 gradle build 时，`build/outputs/apk` 文件夹中有一个 `3_NDK-release-unsigned.apk` 文件。这正是我们所需要的。这将是我们签名过程的原材料。

现在，我们需要一个有效的发布密钥。我们可以使用 Java 开发工具包中的 `keytool` 创建自签名的发布密钥，使用以下命令：

```java
$ keytool -genkey -v -keystore my-release-key.keystore -alias alias_name -keyalg RSA -keysize 2048 -validity 10000

```

这将要求我们填写创建 `release key` 和 `keystore` 时所需的所有字段。

```java
Enter keystore password:
Re-enter new password:
What is your first and last name?
 [Unknown]:  Sergey Kosarevsky
What is the name of your organizational unit?
 [Unknown]:  SD
What is the name of your organization?
 [Unknown]:  Linderdaum
What is the name of your City or Locality?
 [Unknown]:  St.Petersburg
What is the name of your State or Province?
 [Unknown]:  Kolpino
What is the two-letter country code for this unit?
 [Unknown]:  RU
Is CN=Sergey Kosarevsky, OU=SD, O=Linderdaum, L=St.Petersburg, ST=Kolpino, C=RU correct?
 [no]:  yes
Generating 2048 bit RSA key pair and self-signed certificate (SHA1withRSA) with a validity of 10000 days
for: CN=Sergey Kosarevsky, OU=SD, O=Linderdaum, L=St.Petersburg, ST=Kolpino, C=RU
Enter key password for <alias_name>
 (RETURN if same as keystore password):
[Storing my-release-key.keystore]

```

现在，我们准备进行实际的 `.apk` 包签名。使用 Java 开发工具包中的 `jarsigner` 工具来完成这个操作：

```java
>jarsigner -verbose -sigalg MD5withRSA -digestalg SHA1 -keystore my-release-key.keystore 3_NDK-release-unsigned.apk alias_name

```

这个命令是交互式的，它将要求用户输入 `keystore` 和 `key passwords`。但是，我们可以以下面的方式将这两个密码作为参数提供给这个命令：

```java
>jarsigner -verbose -sigalg MD5withRSA -digestalg SHA1 -keystore my-release-key.keystore -storepass 123456 –keypass 123456 3_NDK-release-unsigned.apk alias_name

```

当然，密码应与您在创建 `release key` 和 `keystore` 时输入的密码相匹配。

在我们能够安全地在 Google Play 上发布 `.apk` 包之前，还有一件重要的事情要做。Android 应用程序可以使用内存映射文件和 `mmap()` 系统调用来访问 `.apk` 中的未压缩内容，但 `mmap()` 可能会对底层数据施加一些对齐限制。我们需要将 `.apk` 中的所有未压缩数据按照 4 字节边界对齐。Android SDK 有 `zipalign` 工具来完成这个操作，如下面的命令所示：

```java
>zipalign -v 4 3_NDK-release-unsigned.apk 3_NDK-release.apk

```

现在，我们的 `.apk` 已准备好在 Google Play 上发布。

# 组织跨平台代码

本书延续了我们之前出版的《*Android NDK 游戏开发手册*, *Packt Publishing*> 的思想：即使用“所见即所得”原则进行跨平台开发的可能。大部分应用程序逻辑可以在熟悉的桌面环境如 Windows 中开发并测试，手头拥有所有必要的工具，必要时可以构建为 Android 使用 NDK。

为了组织和维护跨平台的 C++ 源代码，我们需要将所有内容分为平台特定和平台独立部分。我们的 Android 特定本地代码将存储在项目的 `jni` 子文件夹中，这与我们之前的简约示例完全相同。共享的平台独立 C++ 代码将放入 `src-native` 子文件夹。

# 使用 TeamCity 持续集成服务器与 Android 应用程序

TeamCity 是一个强大的持续集成和部署服务器，可用于自动化你的 Android 应用构建。这可以在 [`www.jetbrains.com/teamcity`](https://www.jetbrains.com/teamcity) 找到。

### 注意

TeamCity 对最多需要 20 个构建配置和 3 个构建代理的小型项目是免费的，对于开源项目则是完全免费的。在 [`www.jetbrains.com/teamcity/buy`](https://www.jetbrains.com/teamcity/buy) 申请开源许可。

服务器安装过程非常直接。Windows、OS X 或 Linux 机器可以作为服务器或构建代理。这里，我们将展示如何在 Windows 上安装 TeamCity。

从 [`www.jetbrains.com/teamcity/download`](https://www.jetbrains.com/teamcity/download) 下载最新版本的安装程序，并使用以下命令运行它：

```java
>TeamCity-9.0.1.exe

```

安装所有组件并将其作为 **Windows 服务** 运行。为了简单起见，我们将在一台机器上同时运行服务器和代理，如下面的屏幕截图所示：

![在 Android 应用程序中使用 TeamCity 持续集成服务器](img/image00215.jpeg)

选择所需的 TeamCity 服务器端口。我们将使用默认的 HTTP 端口 80。在 `SYSTEM` 账户下运行 **TeamCity 服务器** 和 **代理** 服务。

一旦服务器上线，打开你的浏览器并通过地址 `http://localhost` 连接到它。创建一个新项目和构建配置。

### 注意

要使用 TeamCity，你应该将你的项目源代码放入版本控制系统。Git 和 GitHub 将是一个不错的选择。

如果你的项目已经在 GitHub 上，你可以创建一个指向你的 GitHub 仓库 URL 的 Git 版本控制系统根目录，如下所示 `https://github.com/<你的登录名>/<你的项目>.git`。

添加一个新的命令行构建步骤并输入脚本的内容：

```java
ndk-build
ant release

```

你也可以在这里添加使用 `jarsigner` 的签名，并使用 `zipalign` 工具创建最终的 `.apk` 生产文件。

现在，进入 **通用设置** 步骤并将工件路径添加到 `bin/3_NDK-release.apk`。项目已准备好进行持续集成。

# 概括

在本章中，我们学习了如何使用命令行安装和配置 Android 原生开发的基本工具，以及如何不依赖图形 IDE 而手动编写 Android 应用基本配置文件。在后续章节中，我们将练习这些技能并构建一些项目。
