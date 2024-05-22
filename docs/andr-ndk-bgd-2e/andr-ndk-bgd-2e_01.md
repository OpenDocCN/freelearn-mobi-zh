# 第一章：设置你的开发环境

> *你准备好接受移动开发挑战了吗？你的电脑打开了，鼠标和键盘插上了，屏幕照亮了你的桌子吗？那么我们不要再等一分钟了！*
> 
> *开发 Android 应用程序需要一套特定的工具。你可能已经了解到了用于纯 Java 应用程序的 Android 软件开发工具包（SDK）。然而，要完全访问 Android 设备的强大功能，还需要更多：Android 原生开发工具包（NDK）。*

设置一个合适的 Android 环境并不是那么复杂，但它可能相当棘手。实际上，Android 仍然是一个不断发展的平台，最近的添加内容，如 Android Studio 或 Gradle，在 NDK 开发方面支持得并不好。尽管有这些烦恼，任何人都可以在一个小时内拥有一个可以立即工作的环境。

在第一章中，我们将要：

+   安装必备软件包

+   设置一个 Android 开发环境

+   启动一个 Android 模拟器

+   连接一个用于开发的 Android 设备

# 开始 Android 开发

区分人类与动物的是工具的使用。Android 开发者，你所属的真正物种，也不例外！

要在 Android 上开发应用程序，我们可以使用以下三个平台中的任何一个：

+   微软 Windows（XP 及更高版本）

+   苹果 OS X（版本 10.4.8 或更高版本）

+   Linux（使用 GLibc 2.7 或更高版本的发行版，如最新版本的 Ubuntu）

这些系统在 x86 平台（即使用 Intel 或 AMD 处理器的 PC）上支持 32 位和 64 位版本，Windows XP 除外（仅 32 位）。

这是一个不错的开始，但除非你能像说母语一样读写二进制代码，否则仅有一个原始操作系统是不够的。我们还需要专门用于 Android 开发的软件：

+   一个**JDK**（**Java 开发工具包**）

+   一个 Android SDK（软件开发工具包）

+   一个 Android NDK（原生开发工具包）

+   一个**IDE**（**集成开发环境**），如 Eclipse 或 Visual Studio（或为硬核程序员准备的 vi）。尽管 Android Studio 和 IntelliJ 为原生代码提供了基本支持，但它们还不适合 NDK 开发。

+   一个好的旧命令行终端来操作所有这些工具。我们将使用 Bash。

既然我们知道与 Android 工作需要哪些工具，那么让我们开始安装和设置过程。

### 注意

以下部分专门针对 Windows。如果你是 Mac 或 Linux 用户，可以跳到*设置 OS X*或*设置 Linux*部分。

## 设置 Windows

在安装必要工具之前，我们需要正确设置 Windows 以承载我们的 Android 开发工具。尽管 Windows 并不是 Android 开发的最自然选择，但它仍然提供了一个功能齐全的环境。

以下部分将解释如何在 Windows 7 上设置必备软件包。这个过程同样适用于 Windows XP、Vista 或 8。

# 动手操作——为 Android 开发准备 Windows

要在 Windows 上使用 Android NDK 进行开发，我们需要设置一些先决条件：Cygwin、JDK 和 Ant。

1.  访问[`cygwin.com/install.html`](http://cygwin.com/install.html)并下载适合你环境的 Cygwin 安装程序。下载完成后，执行它。

1.  在安装窗口中，点击**下一步**然后选择**从互联网安装**。![准备在 Windows 上开发 Android 的动作时间](img/9645_01_22.jpg)

    跟随安装向导屏幕操作。考虑选择一个在你国家下载 Cygwin 软件包的下载站点。

    然后，当提议时，包括**Devel**、**Make**、**Shells**和**bash**软件包：

    ![准备在 Windows 上开发 Android 的动作时间](img/9645_01_23.jpg)

    跟随安装向导直到完成。根据你的互联网连接，这可能需要一些时间。

1.  从 Oracle 官网[`www.oracle.com/technetwork/java/javase/downloads/index.html`](http://www.oracle.com/technetwork/java/javase/downloads/index.html)下载 Oracle JDK 7（或者 JDK 8，尽管在本书编写时它还未正式支持）。启动并按照安装向导直到完成。

1.  从 Ant 的官网[`ant.apache.org/bindownload.cgi`](http://ant.apache.org/bindownload.cgi)下载 Ant，并将其二进制包解压到你选择的目录中（例如，`C:\Ant`）。

1.  安装后，在环境变量中定义 JDK、Cygwin 和 Ant 的位置。为此，打开 Windows **控制面板** 并进入 **系统** 面板（或者在 Windows 开始菜单中右键点击 **计算机** 项，选择 **属性**）。

    然后，进入**高级系统设置**。将出现**系统属性**窗口。最后，选择**高级**标签，点击**环境变量**按钮。

1.  在环境变量窗口中，系统变量列表内添加：

    +   设置`CYGWIN_HOME`变量，其值为`Cygwin`安装目录（例如，`C:\Cygwin`）

    +   设置`JAVA_HOME`变量，其值为 JDK 安装目录

    +   设置`ANT_HOME`变量，其值为 Ant 安装目录（例如，`C:\Ant`）

    在你的`PATH`环境变量开头添加`%CYGWIN_HOME%\bin;%JAVA_HOME%\bin;%ANT_HOME%\bin;`，每个路径之间用分号隔开。

    ![准备在 Windows 上开发 Android 的动作时间](img/9645_01_49.jpg)

1.  最后，启动 Cygwin 终端。第一次启动时将创建你的配置文件。检查`make`版本以确保 Cygwin 正常工作：

    ```java
    make –version

    ```

    你将看到以下输出：

    ![准备在 Windows 上开发 Android 的动作时间](img/9645_01_27.jpg)

1.  通过运行 Java 并检查其版本，确保 JDK 已正确安装。仔细检查以确保版本号与刚安装的 JDK 相符：

    ```java
    java –version

    ```

    你将在屏幕上看到以下输出：

    ![准备在 Windows 上开发 Android 的动作时间](img/9645_01_28.jpg)

1.  从经典的 Windows 终端，检查 Ant 版本以确保其正常工作：

    ```java
    ant -version

    ```

    你将在终端上看到以下内容：

    ![准备在 Windows 上进行 Android 开发的行动时间](img/9645_01_48.jpg)

## *刚才发生了什么？*

Windows 现在已设置好所有必要的软件包，以容纳 Android 开发工具：

+   Cygwin 是一个开源软件集合，它允许 Windows 平台模拟类似 Unix 的环境。它的目标是原生地将基于 POSIX 标准（如 Unix、Linux 等）的软件集成到 Windows 中。它可以被视为起源于 Unix/Linux（但在 Windows 上原生重新编译）的应用程序与 Windows 操作系统本身之间的中间层。Cygwin 包括`Make`，这是 Android NDK 编译系统构建原生代码所需的。

    ### 提示

    即使 Android NDK R7 引入了原生的 Windows 二进制文件，不需要 Cygwin 运行时，但为了调试目的，仍然建议安装后者。

+   JDK 7，它包含了在 Android 上构建 Java 应用程序以及运行 Eclipse IDE 和 Ant 所需的运行时和工具。在安装 JDK 时，你可能遇到的唯一真正麻烦是一些来自之前安装的干扰，比如现有的**Java 运行时环境**（**JRE**）。通过`JAVA_HOME`和`PATH`环境变量可以强制使用正确的 JDK。

    ### 提示

    定义`JAVA_HOME`环境变量不是必须的。然而，`JAVA_HOME`是 Java 应用程序（包括 Ant）中的一个流行约定。它首先在`JAVA_HOME`（如果已定义）中查找`java`命令，然后才在`PATH`中查找。如果你稍后在其他位置安装了最新的 JDK，不要忘记更新`JAVA_HOME`。

+   Ant 是一个基于 Java 的构建自动化工具。虽然这不是一个必需品，但它允许从命令行构建 Android 应用程序，如我们将在第二章，*开始一个原生 Android 项目*中看到的。它也是设置持续集成链的一个好解决方案。

下一步是设置 Android 开发工具包。

## 在 Windows 上安装 Android 开发工具

Android 需要特定的开发工具包来开发应用程序：Android SDK 和 NDK。幸运的是，谷歌考虑到了开发者社区，并免费提供所有必要的工具。

在以下部分，我们将安装这些工具，开始在 Windows 7 上开发原生的 Android 应用程序。

# 行动时间——在 Windows 上安装 Android SDK 和 NDK

Android Studio 软件包已经包含了 Android SDK。让我们来安装它。

1.  打开你的网络浏览器，从[`developer.android.com/sdk/index.html`](http://developer.android.com/sdk/index.html)下载 Android Studio 软件包。

    运行下载的程序，并按照安装向导操作。当被请求时，安装所有 Android 组件。

    ![行动时间——在 Windows 上安装 Android SDK 和 NDK](img/9645_01_29.jpg)

    然后，选择 Android Studio 和 Android SDK 的安装目录（例如，`C:\Android\android-studio`和`C:\Android\sdk`）。

1.  启动 Android Studio 以确保其正常工作。如果 Android Studio 提出从之前的安装导入设置，选择你偏好的选项并点击**OK**。![行动时间——在 Windows 上安装 Android SDK 和 NDK](img/9645_01_26.jpg)

    此时应该会出现 Android Studio 的欢迎屏幕。关闭它。

    ![行动时间——在 Windows 上安装 Android SDK 和 NDK](img/9645_01_25.jpg)

1.  访问[`developer.android.com/tools/sdk/ndk/index.html`](http://developer.android.com/tools/sdk/ndk/index.html)，下载适合你环境的 Android NDK（不是 SDK！），将压缩文件解压到你选择的目录中（例如，`C:\Android\ndk`）。

1.  为了从命令行轻松访问 Android 工具，让我们将 Android SDK 和 NDK 声明为环境变量。从现在开始，我们将这些目录称为`$ANDROID_SDK`和`$ANDROID_NDK`。

    打开**环境变量**系统窗口，就像我们之前做的那样。在系统变量列表中添加以下内容：

    +   `ANDROID_SDK`变量应包含 SDK 安装目录（例如，C:\Android\sdk）。

    +   `ANDROID_NDK`变量应包含 NDK 安装目录（例如，C:\Android\ndk）。

    在你的`PATH`环境变量开头添加`%ANDROID_SDK%\tools;%ANDROID_SDK%\platform-tools;%ANDROID_NDK%;`，用分号隔开。

    ![行动时间——在 Windows 上安装 Android SDK 和 NDK](img/9645_01_24.jpg)

1.  当启动 Cygwin 时，所有 Windows 环境变量都应该自动被导入。打开一个 Cygwin 终端，使用`adb`列出连接到电脑的 Android 设备（即使当前没有连接的设备也要这样做），以检查 SDK 是否正常工作。不应该出现错误：

    ```java
    adb devices

    ```

    ![行动时间——在 Windows 上安装 Android SDK 和 NDK](img/9645_01_46.jpg)

1.  检查`ndk-build`版本，以确保 NDK 正常工作。如果一切正常，应该会出现`Make`版本：

    ```java
    ndk-build -version

    ```

    ![行动时间——在 Windows 上安装 Android SDK 和 NDK](img/9645_01_47.jpg)

1.  打开位于 ADB 捆绑目录根目录的**Android SDK Manager**。![行动时间——在 Windows 上安装 Android SDK 和 NDK](img/9645_01_45.jpg)

    在打开的窗口中，点击**New**选择所有包，然后点击**Install packages...**按钮。在弹出的窗口中接受许可协议，点击**Install**按钮开始安装 Android 开发包。

    经过几分钟的等待，所有包都已下载完毕，出现一条确认信息，表明 Android SDK 管理器已更新。

    确认并关闭管理器。

## *刚才发生了什么？*

Android Studio 现在已安装在系统上。虽然它现在是官方的 Android IDE，但由于它对 NDK 的支持不足，我们在本书中不会大量使用它。然而，完全可以用 Android Studio 进行 Java 开发，以及使用命令行或 Eclipse 进行 C/C++ 开发。

Android SDK 通过 Android Studio 包进行了设置。另一种解决方案是手动部署 Google 提供的 SDK 独立包。另一方面，Android NDK 是从其归档文件手动部署的。通过几个环境变量，SDK 和 NDK 都可以通过命令行使用。

为了获得一个完全功能性的环境，所有 Android 包都已通过 Android SDK 管理器下载，该管理器旨在管理通过 SDK 可用的所有平台、源、示例和仿真功能。当新的 SDK API 和组件发布时，这个工具极大地简化了环境的更新。无需重新安装或覆盖任何内容！

然而，Android SDK 管理器不管理 NDK，这就是为什么我们要单独下载它，以及为什么将来你需要手动更新它。

### 提示

安装所有 Android 包并不是严格必要的。真正需要的是您的应用程序所针对的 SDK 平台（可能还有 Google APIs）版本。不过，安装所有包可能会在导入其他项目或示例时避免麻烦。

您的 Android 开发环境安装尚未完成。我们还需要一个东西，才能与 NDK 舒适地开发。

### 注意

这是一段专门介绍 Windows 设置的章节的结束。下一章节将专注于 OS X。

## 设置 OS X

Apple 电脑以其简单易用而闻名。我必须说，在 Android 开发方面，这个谚语是相当正确的。实际上，作为基于 Unix 的系统，OS X 很适合运行 NDK 工具链。

下一节将解释如何在 Mac OS X Yosemite 上设置前提条件包。

# 行动时间 - 准备 OS X 进行 Android 开发

要在 OS X 上使用 Android NDK 进行开发，我们需要设置一些前提条件：JDK、开发者工具和 Ant。

1.  OS X 10.6 Snow Leopard 及以下版本预装了 JDK。在这些系统上，Apple 的 JDK 是版本 6。由于这个版本已被弃用，建议安装更新的 JDK 7（或 JDK 8，尽管在本书编写时它没有得到官方支持）。

    另一方面，OS X 10.7 Lion 及以上版本没有默认安装 JDK。因此，安装 JDK 7 是强制性的。

    为此，从 Oracle 网站下载 Oracle JDK 7，网址为 [`www.oracle.com/technetwork/java/javase/downloads/index.html`](http://www.oracle.com/technetwork/java/javase/downloads/index.html)。启动 `DMG` 并按照安装向导直到结束。

    ![行动时间 - 准备 OS X 进行 Android 开发](img/9645_01_25.jpg)

    检查 Java 版本以确保 JDK 已正确安装。

    ```java
    java -version

    ```

    ![动手操作——为 Android 开发准备 OS X](img/9645_01_76.jpg)

    ### 提示

    要知道是否安装了 JDK 6，请检查通过转到 Mac 上的**应用程序** | **实用工具**找到的**Java 偏好设置.app**。如果你有 JDK 7，检查**系统偏好设置**下是否有**Java**图标。

1.  所有开发者工具都包含在 XCode 安装包中（在本书编写时为版本 5）。XCode 在 AppStore 上免费提供。从 OS X 10.9 开始，开发者工具包可以通过终端提示符使用以下命令单独安装：

    ```java
    xcode-select --install

    ```

    ![动手操作——为 Android 开发准备 OS X](img/9645_01_65.jpg)

    然后，从弹出的窗口中选择**安装**。

1.  要使用 Android NDK 构建本地代码，无论是否安装了 XCode 或单独的开发者工具包，我们都需要`Make`。打开终端提示符并检查`Make`版本以确保它能正常工作：

    ```java
    make –version

    ```

    ![动手操作——为 Android 开发准备 OS X](img/9645_01_62.jpg)

1.  在 OS X 10.9 及以后的版本中，需要手动安装 Ant。从 Ant 的官方网站[`ant.apache.org/bindownload.cgi`](http://ant.apache.org/bindownload.cgi)下载 Ant，并将其二进制包解压到您选择的目录中（例如，`/Developer/Ant`）。

    然后，创建或编辑文件`~/.profile`，并通过添加以下内容使 Ant 在系统路径上可用：

    ```java
    export ANT_HOME="/Developer/Ant"
    export PATH=${ANT_HOME}/bin:${PATH}
    ```

    从当前会话注销并重新登录（或重启计算机），并通过命令行检查 Ant 版本以确认 Ant 是否正确安装：

    ```java
    ant –version

    ```

    ![动手操作——为 Android 开发准备 OS X](img/9645_01_60.jpg)

## *刚才发生了什么？*

我们的 OS X 系统现在已设置好必要的软件包以支持 Android 开发工具：

+   JDK 7，它包含了在 Android 上构建 Java 应用程序以及运行 Eclipse IDE 和 Ant 所需的运行时和工具。

+   开发者工具包，它包含了各种命令行工具。它包括 Make，这是 Android NDK 编译系统构建本地代码所需的。

+   Ant，这是一个基于 Java 的构建自动化工具。尽管这不是必须的，但它允许我们从命令行构建 Android 应用程序，如我们将在第二章，*开始一个本地 Android 项目*中看到的。它也是设置持续集成链的一个好解决方案。

下一步是设置 Android 开发工具包。

## 在 OS X 上安装 Android 开发工具包

Android 开发应用程序需要特定的开发工具包：Android SDK 和 NDK。幸运的是，Google 考虑到了开发者社区，并免费提供所有必要的工具。

在接下来的部分，我们将安装这些工具包，开始在 Mac OS X Yosemite 上开发本地 Android 应用程序。

# 动手操作——在 OS X 上安装 Android SDK 和 NDK

Android Studio 软件包已经包含了 Android SDK。我们来安装它。

1.  打开您的网络浏览器，从[`developer.android.com/sdk/index.html`](http://developer.android.com/sdk/index.html)下载 Android Studio 软件包。

1.  运行下载的`DMG`文件。在出现的窗口中，将**Android Studio**图标拖到**应用程序**中，等待 Android Studio 完全复制到系统上。![行动时间 – 在 OS X 上安装 Android SDK 和 NDK](img/9645_01_66.jpg)

1.  从 Launchpad 运行 Android Studio。

    如果出现错误**无法找到有效的 JVM**（因为 Android Studio 在启动时找不到合适的 JRE），您可以通过命令行以下方式运行 Android Studio（使用适当的 JDK 路径）：

    ```java
    export STUDIO_JDK=/Library/Java/JavaVirtualMachines/jdk1.7.0_71.jdk
    open /Applications/Android\ Studio.apps

    ```

    ### 提示

    为了解决 Android Studio 启动问题，您也可以安装苹果提供的旧版 JDK 6。注意！这个版本已经过时，因此不推荐使用。

    如果 Android Studio 提示您从之前的安装导入设置，选择您偏好的选项并点击**确定**。

    ![行动时间 – 在 OS X 上安装 Android SDK 和 NDK](img/9645_01_63.jpg)

    在出现的下一个**设置向导**屏幕中，选择**标准**安装类型并继续安装。

    ![行动时间 – 在 OS X 上安装 Android SDK 和 NDK](img/9645_01_75.jpg)

    完成安装直到出现 Android Studio 欢迎屏幕。然后关闭 Android Studio。

    ![行动时间 – 在 OS X 上安装 Android SDK 和 NDK](img/9645_01_77.jpg)

1.  访问[`developer.android.com/tools/sdk/ndk/index.html`](http://developer.android.com/tools/sdk/ndk/index.html)并下载适合您环境的 Android NDK（不是 SDK！）归档文件。将其解压到您选择的目录中（例如，`~/Library/Android/ndk`）。

1.  为了从命令行轻松访问 Android 实用工具，我们将 Android SDK 和 NDK 声明为环境变量。从现在开始，我们将这些目录称为`$ANDROID_SDK`和`$ANDROID_NDK`。假设您使用默认的`Bash`命令行外壳，在您的家目录中创建或编辑`.profile`（这是一个隐藏文件！）并在最后添加以下指令（根据您的安装调整路径）：

    ```java
    export ANDROID_SDK="~/Library/Android/sdk"
    export ANDROID_NDK="~/Library/Android/ndk"
    export PATH="${ANDROID_SDK}/tools:${ANDROID_SDK}/platform-tools:${ANDROID_NDK}:${PATH}"
    ```

1.  从当前会话注销并重新登录（或者重启电脑）。使用`adb`列出连接到电脑的 Android 设备（即使当前没有连接的设备），以检查 Android SDK 是否正常工作。不应该出现错误：

    ```java
    adb devices

    ```

    ![行动时间 – 在 OS X 上安装 Android SDK 和 NDK](img/9645_01_68.jpg)

1.  检查`ndk-build`版本以确保 NDK 正常工作。如果一切正常，应该会显示`Make`版本：

    ```java
    ndk-build -version

    ```

    ![行动时间 – 在 OS X 上安装 Android SDK 和 NDK](img/9645_01_69.jpg)

1.  打开终端，使用以下命令启动 Android SDK 管理器：

    ```java
    android

    ```

    ![行动时间 – 在 OS X 上安装 Android SDK 和 NDK](img/9645_01_70.jpg)

    在打开的窗口中，点击**新建**以选择所有软件包，然后点击**安装软件包...**按钮。在弹出的窗口中接受许可协议，并通过点击**安装**按钮开始安装所有 Android 软件包。

    几分钟后，所有软件包下载完毕，出现一条确认信息，表明 Android SDK 管理器已更新。

    验证并关闭管理器。

## *刚才发生了什么？*

Android Studio 现在已安装在系统上。尽管它现在是官方的 Android IDE，但由于它对 NDK 的支持不足，我们在书中不会过多地使用它。然而，完全可以用 Android Studio 进行 Java 开发，以及使用命令行或 Eclipse 进行 C/C++开发。

Android SDK 已经通过 Android Studio 软件包进行了设置。另一种解决方案是手动部署由 Google 提供的 SDK 独立软件包。另一方面，Android NDK 则是从其归档文件中手动部署的。通过几个环境变量，SDK 和 NDK 都可以通过命令行使用。

### 提示

在处理环境变量时，OS X 会有些棘手。它们可以在`.profile`中轻松声明，供从终端启动的应用程序使用，正如我们刚才所做的。也可以使用`environment.plist`文件为那些不是从 Spotlight 启动的 GUI 应用程序声明。

为了获得一个完全可用的环境，所有 Android 软件包都通过 Android SDK 管理器下载，该管理器旨在管理通过 SDK 提供的所有平台、源、示例和仿真功能。当新的 SDK API 和组件发布时，这个工具可以大大简化你的环境更新工作。无需重新安装或覆盖任何内容！

然而，Android SDK 管理器并不管理 NDK，这就是为什么我们要单独下载 NDK，以及将来你需要手动更新它的原因。

### 提示

安装所有 Android 软件包并不是绝对必要的。真正需要的是你的应用程序所针对的 SDK 平台（可能还有 Google APIs）。不过，安装所有软件包可以避免在导入其他项目或示例时遇到麻烦。

你的 Android 开发环境安装尚未完成。我们还需要一个东西，以便更舒适地使用 NDK 进行开发。

### 注意

这是一段专门针对 OS X 设置的章节的结束。下一节将专门介绍 Linux。

## 设置 Linux

Linux 非常适合进行 Android 开发，因为 Android 工具链是基于 Linux 的。实际上，作为基于 Unix 的系统，Linux 非常适合运行 NDK 工具链。但是要注意，安装软件包的命令可能会根据你的 Linux 发行版而有所不同。

下一节将解释如何在 Ubuntu 14.10 Utopic Unicorn 上设置必备软件包。

# 动手时间——为 Android 开发准备 Ubuntu

要在 Linux 上使用 Android NDK 进行开发，我们需要设置一些先决条件：Glibc、Make、OpenJDK 和 Ant。

1.  从命令提示符中检查是否安装了 Glibc（GNU C 标准库）2.7 或更高版本，通常 Linux 系统默认会安装：

    ```java
    ldd -–version

    ```

    ![行动时间 - 准备 Ubuntu 以进行 Android 开发](img/9645_01_31.jpg)

1.  `Make` 也需要用来构建本地代码。从 build-essential 软件包中安装它（需要管理员权限）：

    ```java
    sudo apt-get install build-essential

    ```

    运行以下命令以确保正确安装了 `Make`，如果安装正确，将显示其版本：

    ```java
    make –version

    ```

    ![行动时间 - 准备 Ubuntu 以进行 Android 开发](img/9645_01_32.jpg)

1.  在 64 位 Linux 系统上，安装 32 位库兼容性软件包，因为 Android SDK 只有编译为 32 位的二进制文件。在 Ubuntu 13.04 及更早版本上，只需安装 `ia32-libs` 软件包即可：

    ```java
    sudo apt-get install ia32-libs

    ```

    在 Ubuntu 13.10 64 位及以后的版本中，这个软件包已经被移除。因此，手动安装所需的软件包：

    ```java
    sudo apt-get install lib32ncurses5 lib32stdc++6 zlib1g:i386 libc6-i386

    ```

1.  安装 Java OpenJDK 7（或者 JDK 8，尽管在本书编写时它没有得到官方支持）。Oracle JDK 也可以：

    ```java
    sudo apt-get install openjdk-7-jdk

    ```

    通过运行 Java 并检查其版本，确保 JDK 正确安装：

    ```java
    java –version

    ```

    ![行动时间 - 准备 Ubuntu 以进行 Android 开发](img/9645_01_33.jpg)

1.  使用以下命令安装 Ant（需要管理员权限）：

    ```java
    sudo apt-get install ant

    ```

    检查 Ant 是否正常工作：

    ```java
    ant -version

    ```

    ![行动时间 - 准备 Ubuntu 以进行 Android 开发](img/9645_01_34.jpg)

## *刚才发生了什么？*

我们的 Linux 系统现在已准备好必要的软件包以支持 Android 开发工具：

+   build-essential 软件包是 Linux 系统上用于编译和打包的最小工具集。它包括 Make，这是 Android NDK 编译系统构建本地代码所必需的。**GCC**（**GNU C 编译器**）也包括在内，但不是必需的，因为 Android NDK 已经包含了自己的版本。

+   64 位系统上的 32 位兼容库，因为 Android SDK 仍然使用 32 位二进制文件。

+   JDK 7，其中包含在 Android 上构建 Java 应用程序以及在 Eclipse IDE 和 Ant 中运行所需的运行时和工具。

+   Ant 是一个基于 Java 的构建自动化工具。尽管这不是一个硬性要求，但它允许我们从命令行构建 Android 应用程序，正如我们将在第二章《*开始一个本地 Android 项目*》中看到的那样。它也是设置持续集成链的一个好解决方案。

下一步是设置 Android 开发工具包。

## 在 Linux 上安装 Android 开发工具包

Android 开发应用程序需要特定的开发工具包：Android SDK 和 NDK。幸运的是，Google 已经考虑到了开发者社区，并免费提供所有必要的工具。

在以下部分，我们将安装这些工具包，以便在 Ubuntu 14.10 Utopic Unicorn 上开始开发本地 Android 应用程序。

# 行动时间 - 在 Ubuntu 上安装 Android SDK 和 NDK

Android Studio 包已经包含了 Android SDK。让我们来安装它。

1.  打开你的网页浏览器，从 [`developer.android.com/sdk/index.html`](http://developer.android.com/sdk/index.html) 下载 Android Studio 包。将下载的归档文件解压到你选择的目录中（例如，`~/Android/Android-studio`）。

1.  运行 Android Studio 脚本 `bin/studio.sh`。如果 Android Studio 提出从之前的安装导入设置，选择你偏好的选项并点击 **确定**。![行动时间——在 Ubuntu 上安装 Android SDK 和 NDK](img/9645_01_04.jpg)

    在出现的下一个 **设置** **向导** 屏幕上，选择 **标准** 安装类型并继续安装。

    ![行动时间——在 Ubuntu 上安装 Android SDK 和 NDK](img/9645_01_01.jpg)

    完成安装直到出现 Android Studio 欢迎屏幕。然后关闭 Android Studio。

    ![行动时间——在 Ubuntu 上安装 Android SDK 和 NDK](img/9645_01_02.jpg)

1.  访问 [`developer.android.com/tools/sdk/ndk/index.html`](http://developer.android.com/tools/sdk/ndk/index.html) 并下载适合你环境的 Android NDK（不是 SDK！）归档文件。将其解压到你选择的目录中（例如，`~/Android/Ndk`）。

1.  为了从命令行轻松访问 Android 实用工具，让我们将 Android SDK 和 NDK 声明为环境变量。从现在开始，我们将这些目录称为 `$ANDROID_SDK` 和 `$ANDROID_NDK`。编辑你主目录中的 `.profile` 文件（注意这是一个隐藏文件！），并在文件末尾添加以下变量（根据你的安装目录调整它们的路径）：

    ```java
    export ANDROID_SDK="~/Android/Sdk"
    export ANDROID_NDK="~/Android/Ndk"
    export PATH="${ANDROID_SDK}/tools:${ANDROID_SDK}/platform-tools:${ANDROID_NDK}:${PATH}"
    ```

1.  从当前会话中注销并重新登录（或者重启你的电脑）。使用 `adb` 列出连接到电脑的 Android 设备（即使当前没有连接也要列出），以检查 Android SDK 是否正常工作。不应该出现错误：

    ```java
    adb devices

    ```

    ![行动时间——在 Ubuntu 上安装 Android SDK 和 NDK](img/9645_01_35.jpg)

1.  检查 `ndk-build` 的版本以确保 NDK 正在运行。如果一切正常，应该会出现 `Make` 的版本：

    ```java
    ndk-build -version

    ```

    ![行动时间——在 Ubuntu 上安装 Android SDK 和 NDK](img/9645_01_32.jpg)

1.  打开终端，使用以下命令启动 Android SDK 管理器：

    ```java
    android

    ```

    ![行动时间——在 Ubuntu 上安装 Android SDK 和 NDK](img/9645_01_03.jpg)

    在打开的窗口中，点击 **新建** 以选择所有包，然后点击 **安装包...** 按钮。在出现的弹出窗口中接受许可协议，并通过点击 **安装** 按钮开始所有 Android 包的安装。

    经过一些漫长的等待，所有包都已下载完毕，出现一条确认信息表明 Android SDK 管理器已更新。

    确认并关闭管理器。

## *刚才发生了什么？*

现在系统上已经安装了 Android Studio。尽管它现在是官方的安卓 IDE，但由于它对 NDK 的支持不足，我们在本书中不会大量使用它。然而，完全可以用 Android Studio 进行 Java 开发，用命令行或 Eclipse 进行 C/C++开发。

安卓 SDK 已经通过 Android Studio 软件包进行了设置。另一种解决方案是手动部署谷歌提供的 SDK 独立安装包。另一方面，安卓 NDK 则是从其归档文件中手动部署的。通过几个环境变量，SDK 和 NDK 都可以在命令行中使用。

为了获得一个完全功能的环境，所有安卓软件包都是通过安卓 SDK 管理器下载的，该管理器旨在管理通过 SDK 提供的所有平台、源代码、示例和仿真功能。当新的 SDK API 和组件发布时，这个工具可以极大地简化环境的更新。无需重新安装或覆盖任何内容！

然而，安卓 SDK 管理器并不管理 NDK，这就是为什么我们要单独下载 NDK，以及为什么将来需要手动更新它的原因。

### 提示

安装所有的安卓软件包并非严格必要。真正需要的是您的应用程序所针对的 SDK 平台（可能还有 Google APIs）。不过，安装所有软件包可能会在导入其他项目或示例时避免麻烦。

安卓开发环境的安装还没有结束。我们还需要一个东西，才能更舒适地使用 NDK 进行开发。

### 注意

这是一段专门针对 Linux 设置的章节的结束。下一节适用于所有操作系统。

## 安装 Eclipse IDE

由于 Android Studio 的限制，Eclipse 仍然是最适合在安卓上开发本地代码的 IDE 之一。然而，使用 IDE 并非必须；命令行爱好者或`vi`狂热者可以跳过这一部分！

在下一节中，我们将了解如何设置 Eclipse。

# 行动时间 – 在您的操作系统上安装带有 ADT 的 Eclipse

自从最新的安卓 SDK 发布以来，Eclipse 及其插件（ADT 和 CDT）需要手动安装。为此，执行以下步骤：

1.  访问[`www.eclipse.org/downloads/`](http://www.eclipse.org/downloads/)并下载适用于 Java 开发者的 Eclipse。将下载的压缩文件解压到您选择的目录中（例如，在 Windows 上的`C:\Android\eclipse`，Linux 上的`~/Android/Eclipse`，Mac OS X 上的`~/Library/Android/eclipse`）。

    然后，运行 Eclipse。如果 Eclipse 在启动时询问工作空间（其中包含 Eclipse 设置和项目），请定义您选择的位置或保留默认设置，然后点击**确定**。

    当 Eclipse 加载完毕后，关闭欢迎页面。应该会出现以下窗口：

    ![行动时间 – 在您的操作系统上安装带有 ADT 的 Eclipse](img/9645_01_71.jpg)

1.  转到 **帮助** | **安装新软件…**。在 **工作空间:** 字段中输入 `https://dl-ssl.google.com/android/eclipse` 并验证。几秒钟后，会出现一个 **开发者工具** 插件。选择它并点击 **下一步** 按钮。

    ### 提示

    如果在访问更新站点时此步骤失败，请检查您的互联网连接。您可能是断开连接或通过代理连接。在后一种情况下，您可以从 ADT 网页上单独下载 ADT 插件存档并手动安装，或者配置 Eclipse 通过代理连接。

    ![操作时间 – 在您的操作系统上安装带有 ADT 的 Eclipse](img/9645_01_73.jpg)

    按照向导提示操作，并在询问时接受条件。在向导的最后一页，点击**完成**以安装 ADT。可能会出现警告，提示插件内容未签名。忽略它并点击**确定**。完成后，按照请求重启 Eclipse。

1.  返回到 **帮助** | **安装新软件…**。打开 **工作空间** 下拉框，并选择包含 Eclipse 版本名称的项（这里为 Luna）。然后，勾选 **只显示适用于目标环境的软件** 选项。在插件树中找到 **编程语言** 并展开它。最后，勾选所有 C/C++ 插件并点击 **下一步**。![操作时间 – 在您的操作系统上安装带有 ADT 的 Eclipse](img/9645_01_72.jpg)

    按照向导提示操作，并在询问时接受条件。在向导的最后一页，点击**完成**。等待安装完成并重启 Eclipse。

1.  转到 **Windows** | **首选项...**（在 Mac OS X 上为 **Eclipse** | **首选项...**），然后在左侧树中选择 **Android**。如果一切正常，SDK 位置应该已填写 Android SDK 路径。![操作时间 – 在您的操作系统上安装带有 ADT 的 Eclipse](img/9645_01_78.jpg)

    然后，在同一个窗口中，转到 **Android** | **NDK**。**NDK 位置**字段应为空。填写 Android NDK 路径并验证。如果路径错误，Eclipse 会提示目录无效。

    ![操作时间 – 在您的操作系统上安装带有 ADT 的 Eclipse](img/9645_01_74.jpg)

## *刚才发生了什么？*

现在 Eclipse 已经配置好相应的 SDK 和 NDK 并运行起来了。由于 Google 不再提供 ADT 包，因此需要手动在 Eclipse 中安装 Android 开发插件 ADT 和 C/C++ Eclipse 插件 CDT。

请注意，Eclipse 已经被 Google 弃用，并由 Android Studio 替换。遗憾的是，目前 Android Studio 对 C/C++ 和 NDK 的支持相当有限。构建本地代码的唯一方式是通过 Gradle，这个新的 Android 构建系统，其 NDK 功能仍然不稳定。如果舒适的 IDE 对您至关重要，您仍然可以使用 Android Studio 进行 Java 开发，使用 Eclipse 进行 C/C++ 开发。

如果您在 Windows 上工作，可能您是 Visual Studio 的熟练用户。在这种情况下，我建议您注意一些项目，如下所示，将 Android NDK 开发带到了 Visual Studio：

+   Android++是一个免费的 Visual Studio 扩展，可以在[`android-plus-plus.com/`](http://android-plus-plus.com/)找到。尽管在本书编写时仍处于测试阶段，但 Android++看起来相当有前景。

+   NVidia Nsight 可以在 Nvidia 开发者网站[`developer.nvidia.com/nvidia-nsight-tegra`](https://developer.nvidia.com/nvidia-nsight-tegra)（如果你有 Tegra 设备）用开发者账户下载。它将 NDK、一个稍微定制版的 Visual Studio 和一个不错的调试器打包在一起。

+   可以在[`github.com/gavinpugh/vs-android`](https://github.com/gavinpugh/vs-android)找到的 VS-Android 是一个有趣的开放源代码项目，它将 NDK 工具带到了 Visual Studio 中。

我们的开发环境现在几乎准备好了。尽管如此，还缺少最后一块：运行和测试我们应用程序的环境。

## 设置 Android 模拟器

Android SDK 提供了一个模拟器，以帮助希望加快部署-运行-测试周期的开发者，或者希望测试例如不同类型的分辨率和操作系统版本的开发者。让我们看看如何设置它。

# 行动时间 – 创建 Android 虚拟设备

Android SDK 提供了我们轻松创建新的模拟器**Android Virtual Device** (**AVD**)所需的一切：

1.  从终端运行以下命令打开**Android SDK Manager**：

    ```java
    android

    ```

1.  转到**工具** | **管理 AVD...**。或者，在 Eclipse 的主工具栏中点击专用的**Android Virtual Device Manager**按钮。

    然后，点击**新建**按钮创建一个新的 Android 模拟器实例。用以下信息填写表单并点击**确定**：

    ![行动时间 – 创建 Android 虚拟设备](img/9645_01_40.jpg)

1.  新创建的虚拟设备现在显示在**Android Virtual Device Manager**列表中。选择它并点击**启动...**。

    ### 注意

    如果你在 Linux 上遇到与`libGL`相关的错误，请打开命令提示符并运行以下命令以安装 Mesa 图形库：`sudo apt-get install libgl1-mesa-dev`。

1.  **启动选项**窗口出现。根据需要调整显示大小，然后点击**启动**。模拟器启动，一段时间后，你的虚拟设备将加载完毕：![行动时间 – 创建 Android 虚拟设备](img/9645_01_41.jpg)

1.  默认情况下，模拟器的 SD 卡是只读的。虽然这是可选的，但你可以通过从提示符发出以下命令来将其设置为写入模式：

    ```java
    adb shell
    su
    mount -o rw,remount rootfs /
    chmod 777 /mnt/sdcard
    exit

    ```

## *刚才发生了什么？*

安卓模拟器可以通过安卓虚拟设备管理器轻松管理。我们现在能够在代表性的环境中测试我们即将开发的应用程序。更妙的是，我们现在可以在多种条件和分辨率下进行测试，而无需昂贵的设备。然而，如果模拟器是有用的开发工具，请记住模拟并不总是完全具有代表性，并且缺少一些功能，尤其是硬件传感器，这些传感器只能部分模拟。

安卓虚拟设备管理器并非我们管理模拟器的唯一场所。我们还可以使用安卓 SDK 提供的命令行工具 emulator。例如，要从终端直接启动先前创建的 Nexus4 模拟器，请输入以下内容：

```java
emulator -avd Nexus4

```

在创建`Nexus4` AVD 时，敏锐的读者可能已经注意到我们将 CPU/ABI 设置为 Intel Atom（x86），而大多数安卓设备运行在 ARM 处理器上。实际上，由于 Windows、OS X 和 Linux 都运行在 x86 上，只有 x86 安卓模拟器镜像可以受益于硬件和 GPU 加速。另一方面，ARM ABI 在没有加速的情况下可能会运行得相当慢，但它可能更符合你的应用程序可能运行的设备。

### 提示

若要使用 X86 AVD 获得完全硬件加速，你需要在 Windows 或 Mac OS X 系统上安装英特尔**硬件加速执行管理器**（**HAXM**）。在 Linux 上，你可以安装 KVM。这些程序只有在你的 CPU 支持虚拟化技术时才能工作（如今大多数情况下都是如此）。

敏锐的读者可能还会惊讶于我们没有选择最新的安卓平台。原因仅仅是并非所有安卓平台都提供 x86 镜像。

### 注意

快照选项允许在关闭模拟器之前保存其状态。遗憾的是，这个选项与 GPU 加速不兼容。你必须选择其中之一。

最后需要注意的是，在创建 AVD 以在有限的硬件条件下测试应用程序时，自定义其他选项（如 GPS、摄像头等的设置）也是可能的。屏幕方向可以通过快捷键*Ctrl* + *F11*和*Ctrl* + *F12*进行切换。有关如何使用和配置模拟器的更多信息，请访问安卓网站：[`developer.android.com/tools/devices/emulator.html`](http://developer.android.com/tools/devices/emulator.html)。

## 使用安卓设备进行开发

尽管模拟器可以提供帮助，但它们显然无法与真实设备相比。因此，请拿起你的安卓设备，打开它，让我们尝试将其连接到我们的开发平台。以下步骤可能会因你的制造商和手机语言而有所不同。因此，请参阅你的设备文档以获取具体说明。

# 行动时间——设置安卓设备

设备配置取决于你的目标操作系统。为此：

1.  如果适用，请在你的操作系统上配置设备驱动：

    +   如果你使用的是 Windows，开发设备的安装是特定于制造商的。更多信息可以在[`developer.android.com/tools/extras/oem-usb.html`](http://developer.android.com/tools/extras/oem-usb.html)找到，那里有设备制造商的完整列表。如果你的 Android 设备附带有驱动 CD，你可以使用它。请注意，Android SDK 也包含一些 Windows 驱动程序，位于`$ANDROID_SDK\extras\google\usb_driver`目录下。针对 Google 开发手机，Nexus One 和 Nexus S 的具体说明可以在[`developer.android.com/sdk/win-usb.html`](http://developer.android.com/sdk/win-usb.html)找到。

    +   如果你使用的是 OS X，只需将你的开发设备连接到你的 Mac 应该就足以让它工作了！你的设备应该会立即被识别，无需安装任何东西。Mac 的易用性并非传说。

    +   如果你是一个 Linux 用户，将你的开发设备连接到你的发行版（至少在 Ubuntu 上）应该就足以让它工作了！

1.  如果你的移动设备运行的是 Android 4.2 或更高版本，从应用程序列表屏幕，进入**设置** | **关于手机**，并在列表末尾多次点击**构建编号**。经过一番努力后，**开发者选项**将神奇地出现在你的应用程序列表屏幕中。

    在 Android 4.1 设备及其早期版本上，**开发者选项**应该默认可见。

1.  仍然在你的设备上，从应用程序列表屏幕，进入**设置** | **开发者选项**，并启用**调试**和**保持唤醒**。

1.  使用数据连接线将你的设备连接到计算机。注意！有些线缆是仅供充电的，不能用于开发！根据你的设备制造商，它可能显示为 USB 磁盘。

    在 Android 4.2.2 设备及其后续版本上，手机屏幕上会出现一个**允许 USB 调试？**的对话框。选择**始终允许从此计算机**以永久允许调试，然后点击**确定**。

1.  打开命令提示符并执行以下操作：

    ```java
    adb devices

    ```

    ![行动时间——设置 Android 设备](img/9645_01_50.jpg)

    在 Linux 上，如果出现**?????????**而不是你的设备名称（这很可能会发生），那么`adb`没有适当的访问权限。一个可能的解决方案是以 root 权限重启`adb`（风险自负！）：

    ```java
    sudo $ANDROID_SDK/platform-tools/adb kill-server
    sudo $ANDROID_SDK/platform-tools/adb devices

    ```

    另一个找到你的 Vendor ID 和 Product ID 的解决方案可能是必要的。Vendor ID 是每个制造商的固定值，可以在 Android 开发者网站[`developer.android.com/tools/device.html`](http://developer.android.com/tools/device.html)上找到（例如，HTC 是`0bb4`）。设备的产品 ID 可以通过`lsusb`命令的结果找到，我们在其中查找 Vendor ID（例如，这里的 0c87 是 HTC Desire 的产品 ID）：

    ```java
    lsusb | grep 0bb4

    ```

    ![行动时间——设置 Android 设备](img/9645_01_51.jpg)

    然后，使用 root 权限创建一个文件`/etc/udev/rules.d/51-android.rules`，并填入你的 Vendor ID 和 Product ID，然后将文件权限改为 644：

    ```java
    sudo sh -c 'echo SUBSYSTEM==\"usb\", SYSFS{idVendor}==\"<Your Vendor ID>\", ATTRS{idProduct}=\"<Your Product ID>\", GROUP=\"plugdev\", MODE=\"0666\" > /etc/udev/rules.d/52-android.rules'
    sudo chmod 644 /etc/udev/rules.d/52-android.rules

    ```

    最后，重启`udev`服务和`adb`：

    ```java
    sudo service udev restart
    adb kill-server
    adb devices

    ```

1.  启动 Eclipse 并打开**DDMS**透视图（**窗口** | **打开透视图** | **其他...**）。如果正常工作，你的手机应该列在**设备**视图中。

    ### 提示

    Eclipse 是由许多视图组成的，例如包资源管理器视图、调试视图等。通常，它们大多数已经可见，但有时并非如此。在这种情况下，通过主菜单导航到**窗口** | **显示视图** | **其他…**来打开它们。Eclipse 中的视图被组织在**透视图**中，这些透视图存储工作区布局。可以通过转到**窗口** | **打开透视图** | **其他…**来打开它们。请注意，某些上下文菜单可能只在某些透视图可用。

## *刚才发生了什么？*

我们的 Android 设备已切换到开发模式，并通过 Android 调试桥守护进程连接到工作站。第一次从 Eclipse 或命令行调用 ADB 时，它会自动启动。

我们还启用了**保持唤醒**选项，以防止在手机充电或开发时自动关闭屏幕！而且，比任何事情都重要的是，我们发现 HTC 代表的是高技术计算机！玩笑归玩笑，在 Linux 上的连接过程可能会很棘手，尽管现在应该不会遇到太多麻烦。

仍然遇到不情愿的 Android 设备的问题？这可能意味着以下任何一种情况：

+   ADB 出现故障。在这种情况下，重启 ADB 守护进程或以管理员权限执行它。

+   你的开发设备工作不正常。在这种情况下，尝试重启你的设备或禁用并重新启用开发模式。如果仍然不起作用，那么购买另一个设备或使用模拟器。

+   你的主机系统没有正确设置。在这种情况下，仔细检查你的设备制造商的说明，确保必要的驱动程序已正确安装。检查硬件属性，看它是否被识别，并打开 USB 存储模式（如果适用），看它是否被正确检测。请参考你的设备文档。

    ### 提示

    当激活仅充电模式时，SD 卡中的文件和目录对手机上安装的 Android 应用可见，但对电脑不可见。相反，当激活磁盘驱动器模式时，这些文件和目录只对电脑可见。当你的应用无法在 SD 卡上访问其资源文件时，请检查你的连接模式。

## 关于 ADB 的更多信息

ADB 是一个多功能的工具，用作开发环境和设备之间的中介。它包括以下部分：

+   在模拟器和设备上运行的后台进程，用于接收来自工作站的任务或请求。

+   工作站上与连接设备和模拟器通信的后台服务器。列出设备时，会涉及到 ADB 服务器。调试时，会涉及到 ADB 服务器。与设备进行任何通信时，都会涉及到 ADB 服务器！

+   在你的工作站上运行的客户端，通过 ADB 服务器与设备通信。我们与之交互列出设备的 ADB 客户端。

ADB 提供了许多有用的选项，其中一些在以下表格中：

| 命令 | 描述 |
| --- | --- |
| `adb help` | 获取详尽的帮助，包括所有可用的选项和标志 |
| `adb bugreport` | 打印整个设备的状态 |
| `adb devices` | 列出当前连接的所有 Android 设备，包括模拟器 |
| `adb install [-r] <apk path>` | 安装应用程序包。添加`-r`以重新安装已部署的应用程序并保留其数据 |
| `adb kill-server` | 终止 ADB 守护进程 |
| `adb pull <device path> <local path>` | 将文件传输到你的电脑 |
| `adb push <local path> <device path>` | 将文件传输到你的设备或模拟器 |
| `adb reboot` | 以编程方式重启 Android 设备 |
| `adb shell` | 在 Android 设备上启动 shell 会话（更多内容请见第二章，*开始一个本地 Android 项目*) |
| `adb start-server` | 启动 ADB 守护进程 |
| `adb wait-for-device` | 等待直到设备或模拟器连接到你的电脑（例如，在脚本中） |

当同时连接多个设备时，ADB 还提供了可选的标志来定位特定设备：

| `-s <device id>` | 通过设备的名称（可以在 adb devices 中找到）来定位一个特定的设备 |
| --- | --- |
| `-d` | 如果只连接了一个物理设备，则定位当前物理设备（或者会引发错误信息） |
| `-e` | 如果只连接了一个模拟器，则定位当前运行的模拟器（或者会引发错误信息） |

例如，当设备连接时同时转储模拟器状态，执行以下命令：

```java
adb -e bugreport

```

这只是 ADB 功能的概述。更多信息可以在 Android 开发者网站找到，网址是[`developer.android.com/tools/help/adb.html`](http://developer.android.com/tools/help/adb.html)。

# 总结

设置我们的 Android 开发平台可能有些繁琐，但希望这是一劳永逸的！

总之，我们在系统上安装了所有必备的软件包。其中一些是特定于目标操作系统的，例如 Windows 上的 Cygwin，OS X 上的 Developer Tools，或者 Linux 上的 build-essential 软件包。然后，我们安装了包含 Android Studio IDE 和 Android SDK 的 Android Studio 捆绑包。Android NDK 需要单独下载和设置。

即使我们在这本书中不会经常使用它，Android Studio 仍然是纯 Java 开发的最佳选择之一。它由谷歌维护，当 Gradle NDK 的集成更加成熟时，它可能成为一个不错的选择。

同时，最简单的解决方案是使用 Eclipse 进行 NDK 开发。我们安装了带有 ADT 和 CDT 插件的 Eclipse。这些插件能够很好地整合在一起，它们允许将 Android Java 和本地 C/C++ 代码的强大功能结合到一个单一的 IDE 中。

最后，我们启动了一个 Android 模拟器，并通过 Android 调试桥接器将一个 Android 设备连接到我们的开发平台。

### 提示

由于 Android NDK 是“开放的”，任何人都可以构建自己的版本。Crystax NDK 是由 Dmitry Moskalchuk 创建的特殊 NDK 包。它带来了 NDK 不支持的高级功能（最新的工具链、开箱即用的 Boost…最初支持异常的是 CrystaxNDK）。高级用户可以在 Crystax 网站上找到它，网址为[`www.crystax.net/en/android/ndk`](https://www.crystax.net/en/android/ndk)。

现在我们手中有了塑造我们移动想法所需的工具。在下一章中，我们将驯服它们来创建、编译并部署我们的第一个 Android 项目！
