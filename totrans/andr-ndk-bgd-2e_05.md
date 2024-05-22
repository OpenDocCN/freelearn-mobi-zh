# 第五章.编写一个完全原生的应用程序

> *在前面的章节中，我们通过 JNI 打破了 Android NDK 的表面。但里面还有更多内容！NDK 包含自己的一套特定功能，其中之一就是**原生活动**。原生活动允许仅基于原生代码创建应用程序，无需编写任何 Java 代码。不再需要 JNI！不再需要引用！不再需要 Java！*
> 
> *除了原生活动之外，NDK 还为原生访问 Android 资源提供了一些 API，例如**显示窗口**、**资产**、**设备配置**…这些 API 有助于摆脱通常需要嵌入原生代码的复杂的 JNI 桥接。尽管还有很多缺失且不太可能可用（Java 仍然是 GUI 和大多数框架的主要平台语言），但多媒体应用程序是应用它们的完美目标…*

本章启动了一个在本书中逐步开发的本地 C++ 项目：**DroidBlaster**。从自上而下的视角来看，这个示例滚动射击游戏将包含 2D 图形，稍后还将包括 3D 图形、声音、输入和传感器管理。在本章中，我们将创建其基础结构和主要游戏组件。

现在让我们通过以下方式进入 Android NDK 的核心：

+   创建一个完全原生的活动

+   处理主活动事件

+   原生访问显示窗口

+   获取时间并计算延迟

# 创建一个原生 Activity

`NativeActivity`类提供了一种简化创建原生应用程序所需工作的方法。它让开发者摆脱了所有用于初始化和与原生代码通信的样板代码，从而专注于核心功能。这种*胶水* Activity 是编写无需一行 Java 代码的应用程序（如游戏）的最简单方式。

### 注意

本书提供的项目成果名为`DroidBlaster_Part1`。

# 动手时间——创建一个基本的原生 Activity

我们现在将了解如何创建一个运行事件循环的最小原生 activity。

1.  创建一个新的混合 Java/C++ 项目，如第二章 *启动一个原生 Android 项目*所示。

    +   将其命名为`DroidBlaster`。

    +   将项目转换为原生项目，如前一章所见。将原生模块命名为`droidblaster`。

    +   删除由 ADT 创建的原生源文件和头文件。

    +   在**项目属性** | **Java 构建路径** | **源**中删除对 Java `src` 目录的引用。然后在磁盘上删除该目录本身。

    +   删除`res/layout`目录中的所有布局。

    +   如果创建了`jni/droidblaster.cpp`，请将其删除。

1.  在`AndroidManifest.xml`中，将应用程序主题设置为`Theme.NoTitleBar.Fullscreen`。

    声明一个指向名为`droidblaster`的原生模块（即我们将编译的原生库）的`NativeActivity`，使用元数据属性`android.app.lib_name`：

    ```java
    <?xml version="1.0" encoding="utf-8"?>
    <manifest 
        package="com.packtpub.droidblaster2d" android:versionCode="1"
        android:versionName="1.0">
        <uses-sdk
            android:minSdkVersion="14"
            android:targetSdkVersion="19"/>

        <application android:icon="@drawable/ic_launcher"
            android:label="@string/app_name"
            android:allowBackup="false"
            android:theme         ="@android:style/Theme.NoTitleBar.Fullscreen">
     <activity android:name="android.app.NativeActivity"
                android:label="@string/app_name"
                android:screenOrientation="portrait">
                <meta-data android:name="android.app.lib_name"
     android:value="droidblaster"/>
                <intent-filter>
                    <action android:name ="android.intent.action.MAIN"/>
                    <category
                        android:name="android.intent.category.LAUNCHER"/>
                </intent-filter>
            </activity>
        </application>
    </manifest>
    ```

1.  创建`jni/Types.hpp`文件。这个头文件将包含通用类型和`cstdint`头文件：

    ```java
    #ifndef _PACKT_TYPES_HPP_
    #define _PACKT_TYPES_HPP_

    #include <cstdint>

    #endif
    ```

1.  让我们编写一个日志类，以便在 Logcat 中得到一些反馈。

    +   创建`jni/Log.hpp`文件，并声明一个新的`Log`类。

    +   定义`packt_Log_debug`宏，以便通过一个简单的编译标志来激活或禁用调试信息：

        ```java
        #ifndef _PACKT_LOG_HPP_
        #define _PACKT_LOG_HPP_

        class Log {
        public:
            static void error(const char* pMessage, ...);
            static void warn(const char* pMessage, ...);
            static void info(const char* pMessage, ...);
            static void debug(const char* pMessage, ...);
        };

        #ifndef NDEBUG
            #define packt_Log_debug(...) Log::debug(__VA_ARGS__)
        #else
            #define packt_Log_debug(...)
        #endif

        #endif
        ```

1.  实现文件`jni/Log.cpp`，并实现`info()`方法。为了将消息写入 Android 日志，NDK 在`android/log.h`头文件中提供了一个专用的日志 API，可以像在 C 中使用`printf()`或`vprintf()`（带有`varArgs`）一样使用：

    ```java
    #include "Log.hpp"

    #include <stdarg.h>
    #include <android/log.h>

    void Log::info(const char* pMessage, ...) {
        va_list varArgs;
        va_start(varArgs, pMessage);
        __android_log_vprint(ANDROID_LOG_INFO, "PACKT", pMessage,
            varArgs);
        __android_log_print(ANDROID_LOG_INFO, "PACKT", "\n");
        va_end(varArgs);
    }
    ...
    ```

    编写其他日志方法，`error()`、`warn()`和`debug()`，它们几乎相同，除了级别宏分别是`ANDROID_LOG_ERROR`、`ANDROID_LOG_WARN`和`ANDROID_LOG_DEBUG`。

1.  `NativeActivity`中的应用事件可以通过事件循环处理。因此，创建`jni/EventLoop.hpp`文件，定义一个具有唯一方法`run()`的类。

    包含`android_native_app_glue.h`头文件，它定义了`android_app`结构体。这代表了一个可以称为**应用上下文**的东西，其中所有信息都与本地活动相关；它的状态、它的窗口、它的事件队列等等：

    ```java
    #ifndef _PACKT_EVENTLOOP_HPP_
    #define _PACKT_EVENTLOOP_HPP_

    #include <android_native_app_glue.h>

    class EventLoop {
    public:
        EventLoop(android_app* pApplication);

        void run();

    private:
        android_app* mApplication;
    };
    #endif
    ```

1.  创建`jni/EventLoop.cpp`文件，并在`run()`方法中实现活动事件循环。包含一些日志事件，以便在 Android 日志中得到一些反馈。

    在整个活动生命周期中，`run()`方法会不断循环处理事件，直到请求终止。当一个活动即将被销毁时，`android_app`结构中的`destroyRequested`值会在内部改变，以指示客户端代码必须退出。

    同时，调用`app_dummy()`以确保将本地代码与`NativeActivity`连接的胶水代码不会被链接器移除。我们将在第九章，*将现有库移植到 Android*中了解更多相关信息。

    ```java
    #include "EventLoop.hpp"
    #include "Log.hpp"

    EventLoop::EventLoop(android_app* pApplication):
            mApplication(pApplication)
    {}

    void EventLoop::run() {
        int32_t result; int32_t events;
        android_poll_source* source;

        // Makes sure native glue is not stripped by the linker.
        app_dummy();

        Log::info("Starting event loop");
        while (true) {
            // Event processing loop.
            while ((result = ALooper_pollAll(-1, NULL, &events,
                    (void**) &source)) >= 0) {
                // An event has to be processed.
                if (source != NULL) {
                    source->process(mApplication, source);
                }
                // Application is getting destroyed.
                if (mApplication->destroyRequested) {
                    Log::info("Exiting event loop");
                    return;
                }
            }
        }
    }
    ```

1.  最后，创建`jni/Main.cpp`文件，定义程序入口点`android_main()`，它在一个新的文件`Main.cpp`中运行事件循环：

    ```java
    #include "EventLoop.hpp"
    #include "Log.hpp"

    void android_main(android_app* pApplication) {
        EventLoop(pApplication).run();
    }
    ```

1.  编辑`jni/Android.mk`文件，定义`droidblaster`模块（即`LOCAL_MODULE`指令）。

    使用`LS_CPP`宏帮助描述编译`LOCAL_SRC_FILES`指令的 C++文件（关于这方面的更多信息，请见第九章，*将现有库移植到 Android*）。

    将`droidblaster`与`native_app_glue`模块（即`LOCAL_STATIC_LIBRARIES`指令）和`android`（**Native App Glue**模块所必需的）以及`log`库（即`LOCAL_LDLIBS`指令）链接起来：

    ```java
    LOCAL_PATH := $(call my-dir)

    include $(CLEAR_VARS)

    LS_CPP=$(subst $(1)/,,$(wildcard $(1)/*.cpp))
    LOCAL_MODULE := droidblaster
    LOCAL_SRC_FILES := $(call LS_CPP,$(LOCAL_PATH))
    LOCAL_LDLIBS := -landroid -llog
    LOCAL_STATIC_LIBRARIES := android_native_app_glue

    include $(BUILD_SHARED_LIBRARY)

    $(call import-module,android/native_app_glue)
    ```

1.  创建`jni/Application.mk`文件，以编译针对多个`ABI`的本地模块。我们将使用最基本的内容，如下代码所示：

    ```java
    APP_ABI := armeabi armeabi-v7a x86
    ```

## *刚才发生了什么？*

构建并运行应用程序。当然，启动此应用程序时你不会看到任何惊人的东西。实际上，你只会看到一个黑屏！但是，如果你仔细查看 Eclipse 中的**LogCat**视图（或使用`adb logcat`命令），你会发现一些有趣的信息，这些信息是响应活动事件由你的原生应用程序发出的。

![刚才发生了什么？](img/9645_05_01.jpg)

我们启动了一个没有一行 Java 代码的 Java Android 项目！在`AndroidManifest`中，我们没有引用`Activity`的子类，而是引用了 Android 框架提供的`android.app.NativeActivity`类。

`NativeActivity`是一个 Java 类，像任何其他 Android 活动一样启动，并由 Dalvik 虚拟机像任何其他 Java 类一样解释。然而，我们从未直接面对它。`NativeActivity`实际上是 Android SDK 提供的一个辅助类，它包含处理应用程序事件（生命周期、输入、传感器等）的所有必要的胶水代码，并透明地将它们广播给原生代码。因此，原生活动并没有消除对 JNI 的需求。它只是将其隐藏在幕后！然而，由`NativeActivity`运行的本地 C/C++模块在其自己的线程中执行，完全本地化（使用 Posix 线程 API）！

`NativeActivity`和原生代码通过`native_app_glue`模块连接在一起。原生应用胶水有以下职责：

+   启动运行我们自己的原生代码的原生线程

+   从`NativeActivity`接收事件

+   将这些事件路由到原生线程事件循环以进行进一步处理

`Native glue`模块的代码位于`${ANDROID_NDK}/sources/android/native_app_glue`，可以随意分析、修改或派生（更多信息请参见第九章，*将现有库移植到 Android*）。与原生 API 相关的头文件，如`looper.h`，可以在`${ANDROID_NDK}/platforms/<目标平台>/<目标架构>/usr/include/android/`中找到。让我们更详细地了解它是如何工作的。

## 关于原生应用胶水的更多内容

我们自己的原生代码入口点在`android_main()`方法内声明，这类似于桌面应用程序中的主方法。当`NativeActivity`被实例化和启动时，它只被调用一次。它会循环处理应用程序事件，直到用户终止`NativeActivity`（例如，当按下设备的返回按钮时）或直到它自行退出（下一部分将详细介绍）。

`android_main()`方法并不是真正的原生应用入口点。真正的入口点是隐藏在`android_native_app_glue`模块中的`ANativeActivity_onCreate()`方法。我们在`android_main()`中实现的事件循环实际上是一个代理事件循环，由胶水模块在其自己的原生线程中启动。这种设计将原生代码与在 Java 端的 UI 线程上运行的`NativeActivity`类解耦。因此，即使你的代码处理事件需要很长时间，`NativeActivity`也不会被阻塞，你的 Android 设备仍然保持响应。

`android_main()`中的代理原生事件循环由两个嵌套的 while 循环组成。在我们的例子中，外层循环是一个无限循环，只有在系统请求活动销毁时（由`destroyRequested`标志指示）才会终止。它执行一个内层循环，处理所有待处理的应用程序事件。

```java
...
int32_t result; int32_t events;
android_poll_source* source;
while (true) {
    while ((result = ALooper_pollAll(-1, NULL, &events,
            (void**) &source)) >= 0) {
        if (source != NULL) {
           source->process(mApplication, source);
        }
        if (mApplication->destroyRequested) {
            return;
        }
    }
}
...
```

内层的`For`循环通过调用`ALooper_pollAll()`来轮询事件。这个方法是`Looper` API 的一部分，可以描述为 Android 提供的一个通用事件循环管理器。当超时设置为`-1`时，如前面的示例中，`ALooper_pollAll()`在等待事件时会保持阻塞。当至少收到一个事件时，`ALooper_pollAll()`返回，代码流程继续。

描述事件的`android_poll_source`结构体被填充，并由客户端代码用于进一步处理。这个结构体如下所示：

```java
struct android_poll_source {
    int32_t id; // Source identifier
    struct android_app* app; // Global android application context
    void (*process)(struct android_app* app,
            struct android_poll_source* source); // Event processor
};
```

`process()`函数指针可以被自定义以手动处理应用程序事件，我们将在下一节中看到这一点。

正如我们在这一部分看到的，事件循环接收一个`android_app`结构体作为参数。这个在`android_native_app_glue.h`中描述的结构体包含一些上下文信息，如下表所示：

| `void* userData` | 指向任何你想要的数据的指针。这对于向活动或输入事件回调提供一些上下文信息至关重要。 |
| --- | --- |
| `void (*pnAppCmd)(…)` 和 `int32_t (*onInputEvent)(…)` | 这些成员变量表示当活动或输入事件发生时由原生应用胶水触发的事件回调。我们将在下一节中了解更多相关信息。 |
| `ANativeActivity* activity` | 描述 Java 原生活动（其类作为 JNI 对象，其数据目录等）并提供获取 JNI 上下文所需的必要信息。 |
| `AConfiguration* config` | 描述当前的硬件和系统状态，例如当前的语言和国家，当前的屏幕方向，密度，大小等。 |
| `void* savedState size_t` 和 `savedStateSize` | 用于在活动（及其原生线程）被销毁并稍后恢复时保存数据缓冲区。 |
| `AInputQueue* inputQueue` | 提供输入事件（由原生胶水内部使用）。我们将在第八章，*处理输入设备和传感器*中了解更多关于输入事件的信息。 |
| `ALooper* looper` | 允许附加和分离本地胶水内部使用的事件队列。监听器轮询并等待通信管道上发送的事件。 |
| `ANativeWindow* window`和`ARect contentRect` | 表示可以绘制图形的“可绘制”区域。`ANativeWindow` API，在`native_window.h`中声明，允许获取窗口宽度、高度和像素格式，并更改这些设置。 |
| `int activityState` | 当前活动状态，即`APP_CMD_START`，`APP_CMD_RESUME`，`APP_CMD_PAUSE`等。 |
| `int destroyRequested` | 等于`1`时，表示应用程序即将被销毁，本地线程必须立即终止。这个标志必须在事件循环中检查。 |

`android_app`结构体还包含了一些仅供内部使用的额外数据，这些数据不应被更改。

知道这些细节并不是编写本地程序的必要条件，但可以帮助你了解幕后发生的情况。现在让我们看看如何处理这些活动事件。

# 处理活动事件

在第一部分中，运行了一个本地事件循环，它刷新事件而不真正处理它们。在这个第二部分中，我们将发现更多关于活动生命周期中发生的事件，以及如何处理它们，并花费剩余时间步进我们的应用程序。

### 注意

本书提供的项目结果名为`DroidBlaster_Part2`。

# 行动时间——步进事件循环

让我们扩展上一个示例，在处理事件时步进我们的应用程序。

1.  打开`jni/Types.hpp`文件，定义一个新的类型 status 以表示返回码：

    ```java
    #ifndef _PACKT_TYPES_HPP_
    #define _PACKT_TYPES_HPP_

    #include <cstdlib>

    typedef int32_t status;

    const status STATUS_OK   = 0;
    const status STATUS_KO   = -1;
    const status STATUS_EXIT = -2;

    #endif
    ```

1.  创建`jni/ActivityHandler.hpp`头文件，并定义一个“接口”以观察本地活动事件。每个可能的事件都有其自己的处理方法：`onStart()`，`onResume()`，`onPause()`，`onStop()`，`onDestroy()`等。然而，我们通常只对活动生命周期中的三个特定时刻感兴趣：

    +   `onActivate()`，在活动恢复且其窗口可用并获得焦点时调用。

    +   `onDeactivate()`，在活动暂停或显示窗口失去焦点或被销毁时调用。

    +   `onStep()`，在没有事件需要处理且可以进行计算时调用。

        ```java
        #ifndef _PACKT_ACTIVITYHANDLER_HPP_
        #define _PACKT_ACTIVITYHANDLER_HPP_

        #include "Types.hpp"

        class ActivityHandler {
        public:
            virtual ~ActivityHandler() {};

            virtual status onActivate() = 0;
            virtual void onDeactivate() = 0;
            virtual status onStep() = 0;

            virtual void onStart() {};
            virtual void onResume() {};
            virtual void onPause() {};
            virtual void onStop() {};
            virtual void onDestroy() {};

            virtual void onSaveInstanceState(void** pData, size_t* pSize) {};
            virtual void onConfigurationChanged() {};
            virtual void onLowMemory() {};

            virtual void onCreateWindow() {};
            virtual void onDestroyWindow() {};
            virtual void onGainFocus() {};
            virtual void onLostFocus() {};
        };
        #endif
        ```

1.  使用以下方法增强`jni/EventLoop.hpp`：

    +   `activate()`和`deactivate()`，在活动可用性发生变化时执行。

    +   `callback_appEvent()`，它是静态的，将事件路由到`processActivityEvent()`

    还定义一些成员变量如下：

    +   `mActivityHandler`观察活动事件。这个实例作为构造函数参数提供，需要包含`ActivityHandler.hpp`。

    +   `mEnabled`保存应用程序在活动/暂停状态时的状态。

    +   `mQuit`表示事件循环需要退出。

        ```java
        #ifndef _PACKT_EVENTLOOP_HPP_
        #define _PACKT_EVENTLOOP_HPP_

        #include "ActivityHandler.hpp"
        #include <android_native_app_glue.h>

        class EventLoop {
        public:
            EventLoop(android_app* pApplication,
                    ActivityHandler& pActivityHandler);

            void run();

        private:
         void activate();
         void deactivate();

         void processAppEvent(int32_t pCommand);

         static void callback_appEvent(android_app* pApplication,
         int32_t pCommand);

        private:
            android_app* mApplication;
            bool mEnabled;
         bool mQuit;

         ActivityHandler& mActivityHandler;
        };
        #endif
        ```

1.  编辑`jni/EventLoop.cpp`。构造函数初始化列表本身实现起来非常简单。然后，为`android_app`应用程序上下文填充额外的信息：

    +   `userData`指向您想要的任何数据。这是从之前声明的`callback_appEvent()`中唯一可以访问的信息。在我们的例子中，这是`EventLoop`实例（即`this`）。

    +   `onAppCmd`指向每次发生事件时触发的内部回调。在我们的例子中，这是分配给静态方法`callback_appEvent()`的角色。

        ```java
        #include "EventLoop.hpp"
        #include "Log.hpp"

        EventLoop::EventLoop(android_app* pApplication,
                ActivityHandler& pActivityHandler):
         mApplication(pApplication),
         mEnabled(false), mQuit(false),
         mActivityHandler(pActivityHandler) {
         mApplication->userData = this;
         mApplication->onAppCmd = callback_appEvent;
        }
        ...
        ```

    +   更新`run()`主事件循环。当没有更多活动事件需要处理时，`ALooper_pollAll()`不再阻塞，必须让程序流程继续执行周期性处理。在这里，处理是由`mActivityHandler.onStep()`中的监听器执行的。这种行为只有在应用程序被启用时才需要。

    +   同时，允许使用`AnativeActivity_finish()`方法以编程方式终止活动。

        ```java
        ...
        void EventLoop::run() {
            int32_t result; int32_t events;
            android_poll_source* source;

            // Makes sure native glue is not stripped by the linker.
            app_dummy();

            Log::info("Starting event loop");
            while (true) {
                // Event processing loop.
                while ((result = ALooper_pollAll(mEnabled ? 0 : -1,         NULL,
         &events, (void**) &source)) >= 0) {
                    // An event has to be processed.
                    if (source != NULL) {
                        Log::info("Processing an event");
                        source->process(mApplication, source);
                    }
                    // Application is getting destroyed.
                    if (mApplication->destroyRequested) {
                        Log::info("Exiting event loop");
                        return;
                    }
                }

                // Steps the application.
         if ((mEnabled) && (!mQuit)) {
         if (mActivityHandler.onStep() != STATUS_OK) {
         mQuit = true;
         ANativeActivity_finish(mApplication->activity);
                    }
                }
            }
        }
        ...
        ```

## *刚才发生了什么？*

我们改变了事件循环，以在处理完所有事件后更新应用程序，而不是无用地阻塞。这种行为在`ALooper_pollAll()`的第一个参数，即超时中指定：

+   当超时为`-1`时，如先前定义的，调用将阻塞直到接收到事件。

+   当超时为`0`时，调用是非阻塞的，因此，如果队列中没有任何剩余，程序流程将继续（内部循环结束），这使得可以执行周期性处理。

+   当超时大于`0`时，我们有一个阻塞调用，该调用将保持直到接收到事件或持续时间结束。

在这里，我们希望在活动状态（即执行计算，`mEnabled`为`true`）时执行活动步骤；在这种情况下，超时为`0`。当活动处于非活动状态（`mEnabled`为`false`）时，仍然会处理事件（例如，恢复活动），但无需进行计算。为了避免无谓地消耗电池和处理时间，线程必须被阻塞；在这种情况下，超时为`-1`。

当所有待处理的事件都处理完毕后，将执行监听器的步骤。例如，如果游戏结束，它可以请求应用程序终止。为了从程序上退出应用程序，NDK API 提供了`AnativeActivity_finish()`方法来请求活动终止。终止不会立即发生，而是在处理完最后几个事件（暂停、停止等）后发生。

# 行动时间——处理活动事件。

我们还没有完成。让我们继续我们的示例，以处理活动事件并将它们记录到**LogCat**视图：

1.  继续编辑`jni/EventLoop.cpp`。实现`activate()`和`deactivate()`。在通知监听器之前检查两个活动状态（以避免过早触发）。我们认为只有当显示窗口可用时，活动才被视为激活：

    ```java
    ...
    void EventLoop::activate() {
        // Enables activity only if a window is available.
        if ((!mEnabled) && (mApplication->window != NULL)) {
            mQuit = false; mEnabled = true;
            if (mActivityHandler.onActivate() != STATUS_OK) {
                goto ERROR;
            }
        }
        return;

    ERROR:
        mQuit = true;
        deactivate();
        ANativeActivity_finish(mApplication->activity);
    }

    void EventLoop::deactivate() {
        if (mEnabled) {
            mActivityHandler.onDeactivate();
            mEnabled = false;
        }
    }
    ...
    ```

    +   将活动事件从静态回调`callback_appEvent()`路由到成员方法`processAppEvent()`。

    +   为此，通过`userData`指针获取`EventLoop`实例（静态方法无法使用此指针）。然后，有效的事件处理委托给`processAppEvent()`，这让我们回到了面向对象的世界。同时，原生胶水给出的命令（即活动事件）也被传递。

        ```java
        ...
        void EventLoop::callback_appEvent(android_app* pApplication,
            int32_t pCommand) {
            EventLoop& eventLoop = *(EventLoop*) pApplication->userData;
            eventLoop.processAppEvent(pCommand);
        }
        ...
        ```

1.  在`processAppEvent()`中处理转发的事件。`pCommand`参数包含一个枚举值（`APP_CMD_*`），描述发生的事件（`APP_CMD_START, APP_CMD_GAINED_FOCUS`等）。

    根据事件，激活或停用事件循环并通知监听器：

    当活动获得焦点时会发生激活。这个事件总是在活动恢复并创建窗口后发生的最后一个事件。获得焦点意味着活动可以接收输入事件。

    当窗口失去焦点或应用暂停时（两者都可能首先发生）会发生停用。为了安全起见，在窗口被销毁时也会执行停用，尽管这应该总是在失去焦点之后发生。失去焦点意味着应用不再接收输入事件。

    ```java
    ...
    void EventLoop::processAppEvent(int32_t pCommand) {
        switch (pCommand) {
        case APP_CMD_CONFIG_CHANGED:
            mActivityHandler.onConfigurationChanged();
            break;
        case APP_CMD_INIT_WINDOW:
            mActivityHandler.onCreateWindow();
            break;
        case APP_CMD_DESTROY:
            mActivityHandler.onDestroy();
            break;
        case APP_CMD_GAINED_FOCUS:
            activate();
            mActivityHandler.onGainFocus();
            break;
        case APP_CMD_LOST_FOCUS:
            mActivityHandler.onLostFocus();
            deactivate();
            break;
        case APP_CMD_LOW_MEMORY:
            mActivityHandler.onLowMemory();
            break;
        case APP_CMD_PAUSE:
            mActivityHandler.onPause();
            deactivate();
            break;
        case APP_CMD_RESUME:
            mActivityHandler.onResume();
            break;
        case APP_CMD_SAVE_STATE:
            mActivityHandler.onSaveInstanceState(
               &mApplication->savedState, &mApplication->savedStateSize);
              break;
        case APP_CMD_START:
            mActivityHandler.onStart();
            break;
        case APP_CMD_STOP:
            mActivityHandler.onStop();
            break;
        case APP_CMD_TERM_WINDOW:
            mActivityHandler.onDestroyWindow();
            deactivate();
            break;
        default:
            break;
        }
    }
    ```

    ### 提示

    一些事件，如`APP_CMD_WINDOW_RESIZED`，虽然可用，但从未触发。除非你准备深入胶水，否则不要监听它们。

1.  创建`jni/DroidBlaster.hpp`，实现`ActivityHandler`接口及其所有方法（这里为了简洁起见，省略了一些）。这个类将按如下方式运行游戏逻辑：

    ```java
    #ifndef _PACKT_DROIDBLASTER_HPP_
    #define _PACKT_DROIDBLASTER_HPP_

    #include "ActivityHandler.hpp"
    #include "EventLoop.hpp"
    #include "Types.hpp"

    class DroidBlaster : public ActivityHandler {
    public:
        DroidBlaster(android_app* pApplication);
        void run();

    protected:
        status onActivate();
        void onDeactivate();
        status onStep();

        void onStart();
        ...

    private:
        EventLoop mEventLoop;
    };
    #endif
    ```

1.  使用所有必需的处理程序实现`jni/DroidBlaster.cpp`。为了使这个活动生命周期的介绍保持简单，我们只需记录下面代码中省略的所有处理程序发生的每个事件。使用`onStart()`作为所有处理程序的模型。

    步骤限制为简单的线程休眠（以避免淹没 Android 日志），这需要包含`unistd.h`。

    注意，现在事件循环直接由`DroidBlaster`类运行：

    ```java
    #include "DroidBlaster.hpp"
    #include "Log.hpp"

    #include <unistd.h>

    DroidBlaster::DroidBlaster(android_app* pApplication):
        mEventLoop(pApplication, *this) {
        Log::info("Creating DroidBlaster");
    }

    void DroidBlaster::run() {
        mEventLoop.run();
    }

    status DroidBlaster::onActivate() {
        Log::info("Activating DroidBlaster");
        return STATUS_OK;
    }

    void DroidBlaster::onDeactivate() {
        Log::info("Deactivating DroidBlaster");
    }

    status DroidBlaster::onStep() {
        Log::info("Starting step");
        usleep(300000);
        Log::info("Stepping done");
        return STATUS_OK;
    }

    void DroidBlaster::onStart() {
        Log::info("onStart");
    }
    ...
    ```

1.  最后，在`android_main()`入口点初始化并运行`DroidBlaster`游戏：

    ```java
    #include "DroidBlaster.hpp"
    #include "EventLoop.hpp"
    #include "Log.hpp"

    void android_main(android_app* pApplication) {
        DroidBlaster(pApplication).run();
    }
    ```

## *刚才发生了什么？*

如果你喜欢黑色屏幕，那么你已经得到了！同样，这次，所有的事情都在 Eclipse 的**LogCat**视图中发生。所有对应用事件反应而发出的消息都在这里显示，如下面的截图所示：

![刚才发生了什么？](img/9645_05_02.jpg)

我们创建了一个最小化的框架，它使用事件驱动的方法在本地线程中处理应用事件。事件（被称为命令）被重定向到一个监听器对象，该对象执行其自己的特定计算。

原生活动事件大多对应于经典的 Java 活动事件。事件是任何应用都需要处理的临界点，而且相当棘手。它们通常成对出现，如`start/stop`、`resume/pause`、`create/destroy`、`create window/destroy window`或`gain/lose focus`。尽管它们大多数时间按预定顺序发生，但某些特定情况可能导致不同的行为，例如：

+   使用后退按钮离开应用会销毁活动和原生线程。

+   使用主页按钮离开应用会停止活动并释放窗口。原生线程保持暂停状态。

+   长按设备的主页按钮然后返回，应该只导致失去和获得焦点。原生线程保持暂停状态。

+   关闭手机屏幕并重新打开应该在活动恢复后立即终止并重新初始化窗口。原生线程保持暂停状态。

+   在更改屏幕方向（此处不适用）时，整个活动可能不会失去焦点，尽管重新创建的活动将重新获得焦点。

理解活动生命周期对于开发 Android 应用至关重要。请查看官方 Android 文档中的[`developer.android.com/reference/android/app/Activity.html`](http://developer.android.com/reference/android/app/Activity.html)，了解详细描述。

### 提示

Native App Glue 使您有机会在活动被`APP_CMD_SAVE_STATE`触发销毁之前保存活动状态。状态必须在`android_app`结构中的`savedState`（指向要保存的内存缓冲区的指针）和`savedStateSize`（要保存的内存缓冲区的大小）中保存。该缓冲区必须由我们使用`malloc()`（自动释放）分配，并且不得包含指针，只包含“原始”数据。

# 原生地访问窗口表面

应用事件是必须要理解的，但不是特别令人兴奋。Android NDK 的一个有趣特性是能够原生地访问显示窗口。有了这种特权访问，应用可以在屏幕上绘制任何想要的图形。

我们现在将利用这一特性在我们的应用中获得图形反馈：屏幕上的一个红色方块。这个方块将代表用户在游戏中控制的太空船。

### 注意

本书提供的成果项目名为`DroidBlaster_Part3`。

# 动手操作时间 – 显示原始图形

让我们通过添加一些图形和游戏组件，使`DroidBlaster`更具互动性。

1.  编辑`jni/Types.hpp`文件，并创建一个新的`Location`结构体来保存实体位置。同时，定义一个宏以按照以下方式生成指定范围内的随机值：

    ```java
    #ifndef _PACKT_TYPES_HPP_
    #define _PACKT_TYPES_HPP_
    ...
    struct Location {
     Location(): x(0.0f), y(0.0f) {};

        float x; float y;
    };

    #define RAND(pMax) (float(pMax) * float(rand()) / float(RAND_MAX))
    #endif
    ```

1.  创建一个新文件`jni/GraphicsManager.hpp`。定义一个`GraphicsElement`结构体，其中包含要显示的图形元素的位置和尺寸：

    ```java
    #ifndef _PACKT_GRAPHICSMANAGER_HPP_
    #define _PACKT_GRAPHICSMANAGER_HPP_

    #include "Types.hpp"

    #include <android_native_app_glue.h>

    struct GraphicsElement {
        GraphicsElement(int32_t pWidth, int32_t pHeight):
            location(),
            width(pWidth), height(pHeight) {
        }

        Location location;
        int32_t width;  int32_t height;
    };
    ...
    ```

    接着，在同一个文件中，按照以下方式定义一个`GraphicsManager`类：

    +   `getRenderWidth()`和`getRenderHeight()`用于返回显示尺寸

    +   `registerElement()`是一个`GraphicsElement`工厂方法，它告诉管理器要绘制哪个元素。

    +   `start()`和`update()`分别初始化管理器并渲染每一帧的屏幕

    需要几个成员变量：

    +   `mApplication`存储了访问显示窗口所需的应用程序上下文

    +   `mRenderWidth`和`mRenderHeight`用于显示尺寸

    +   `mElements`和`mElementCount`用于绘制所有元素的表格

        ```java
        ...
        class GraphicsManager {
        public:
            GraphicsManager(android_app* pApplication);
            ~GraphicsManager();

            int32_t getRenderWidth() { return mRenderWidth; }
            int32_t getRenderHeight() { return mRenderHeight; }

            GraphicsElement* registerElement(int32_t pHeight, int32_t pWidth);

            status start();
            status update();

        private:
            android_app* mApplication;

            int32_t mRenderWidth; int32_t mRenderHeight;
            GraphicsElement* mElements[1024]; int32_t mElementCount;
        };
        #endif
        ```

1.  实现`jni/GraphicsManager.cpp`，从构造函数、析构函数和注册方法开始。它们管理要更新的`GraphicsElement`列表：

    ```java
    #include "GraphicsManager.hpp"
    #include "Log.hpp"

    GraphicsManager::GraphicsManager(android_app* pApplication) :
        mApplication(pApplication),
        mRenderWidth(0), mRenderHeight(0),
        mElements(), mElementCount(0) {
        Log::info("Creating GraphicsManager.");
    }

    GraphicsManager::~GraphicsManager() {
        Log::info("Destroying GraphicsManager.");
        for (int32_t i = 0; i < mElementCount; ++i) {
            delete mElements[i];
        }
    }

    GraphicsElement* GraphicsManager::registerElement(int32_t pHeight,
            int32_t pWidth) {
        mElements[mElementCount] = new GraphicsElement(pHeight, pWidth);
        return mElements[mElementCount++];
    }
    ...
    ```

1.  实现了`start()`方法来初始化管理器。

    首先，使用`ANativeWindow_setBuffersGeometry()`API 方法强制窗口深度格式为 32 位。传递的参数中的两个零是所需的窗口宽度和高度。除非用正值初始化，否则它们将被忽略。在这种情况下，请求的由宽度和高度定义的窗口区域会被缩放到匹配屏幕尺寸。

    然后，在`ANativeWindow_Buffer`结构中检索所有必要的窗口尺寸。为了填充这个结构，必须首先使用`ANativeWindow_lock()`锁定窗口，完成后再使用`AnativeWindow_unlockAndPost()`解锁。

    ```java
    ...
    status GraphicsManager::start() {
        Log::info("Starting GraphicsManager.");

        // Forces 32 bits format.
        ANativeWindow_Buffer windowBuffer;
        if (ANativeWindow_setBuffersGeometry(mApplication->window, 0, 0,
            WINDOW_FORMAT_RGBX_8888) < 0) {
            Log::error("Error while setting buffer geometry.");
            return STATUS_KO;
        }

        // Needs to lock the window buffer to get its properties.
        if (ANativeWindow_lock(mApplication->window,
                &windowBuffer, NULL) >= 0) {
            mRenderWidth = windowBuffer.width;
            mRenderHeight = windowBuffer.height;
            ANativeWindow_unlockAndPost(mApplication->window);
        } else {
            Log::error("Error while locking window.");
            return STATUS_KO;
        }
        return STATUS_OK;
    }
    ...
    ```

1.  编写`update()`方法，每次应用程序步进时渲染原始图形。

    在任何绘制操作之前，必须使用`AnativeWindow_lock()`锁定窗口表面。同样，`AnativeWindow_Buffer`结构被填充了窗口的宽度和高度信息，但更重要的是`stride`和`bits`指针。

    `stride`给出了窗口中两条连续像素线之间的距离（以“像素”为单位）。

    `bits`指针直接访问窗口表面，与上一章中看到的 Bitmap API 非常相似。

    有了这两部分信息，就可以执行基于像素的操作。

    例如，使用`0`清除窗口内存区域以获得黑色背景。可以使用`memset()`的暴力方法实现这一目的。

    ```java
    ...
    status GraphicsManager::update() {
        // Locks the window buffer and draws on it.
        ANativeWindow_Buffer windowBuffer;
        if (ANativeWindow_lock(mApplication->window,
                &windowBuffer, NULL) < 0) {
            Log::error("Error while starting GraphicsManager");
            return STATUS_KO;
        }

        // Clears the window.
        memset(windowBuffer.bits, 0, windowBuffer.stride *
                windowBuffer.height * sizeof(uint32_t*));
    ...
    ```

    +   清除后，绘制所有通过`GraphicsManager`注册的元素。屏幕上每个元素都表示为一个红色正方形。

    +   首先，计算要绘制的元素的坐标（左上角和右下角）。

    +   然后，将它们的坐标剪辑以避免在窗口内存区域外绘制。这个操作相当重要，因为超出窗口限制可能会导致段错误：

        ```java
        ...
            // Renders graphic elements.
            int32_t maxX = windowBuffer.width - 1;
            int32_t maxY = windowBuffer.height - 1;
            for (int32_t i = 0; i < mElementCount; ++i) {
                GraphicsElement* element = mElements[i];

                // Computes coordinates.
                int32_t leftX = element->location.x - element->width / 2;
                int32_t rightX = element->location.x + element->width / 2;
                int32_t leftY = windowBuffer.height - element->location.y
                                    - element->height / 2;
                int32_t rightY = windowBuffer.height - element->location.y
                                    + element->height / 2;

                // Clips coordinates.
                if (rightX < 0 || leftX > maxX
                 || rightY < 0 || leftY > maxY) continue;

                if (leftX < 0) leftX = 0;
                else if (rightX > maxX) rightX = maxX;
                if (leftY < 0) leftY = 0;
                else if (rightY > maxY) rightY = maxY;
        ...
        ```

1.  之后，在屏幕上绘制元素的每个像素。`line`变量指向第一条像素线的开始位置，该元素在此位置绘制。这个指针是通过`stride`（两条像素线之间的距离）和元素的顶部`Y`坐标计算得出的。

    然后，我们可以遍历窗口像素来绘制一个代表元素的红色方块。从元素的左`X`坐标遍历到右`X`坐标，当达到每行像素的末尾时（即在`Y`轴上）切换到下一行。

    ```java
    ...
            // Draws a rectangle.
            uint32_t* line = (uint32_t*) (windowBuffer.bits)
                            + (windowBuffer.stride * leftY);
            for (int iY = leftY; iY <= rightY; iY++) {
                for (int iX = leftX; iX <= rightX; iX++) {
                    line[iX] = 0X000000FF; // Red color
                }
                line = line + windowBuffer.stride;
            }
        }
    ...
    ```

    使用`ANativeWindow_unlockAndPost()`结束绘图操作，并挂起对`pendANativeWindow_lock()`的调用。这些必须始终成对调用：

    ```java
    ...
        // Finshed drawing.
        ANativeWindow_unlockAndPost(mApplication->window);
        return STATUS_OK;
    }
    ```

1.  创建一个新组件`jni/Ship.hpp`，代表我们的太空船。

    目前我们只处理初始化，使用`initialize()`函数。

    使用工厂方法`registerShip()`创建`Ship`。

    需要初始化`GraphicsManager`和飞船`GraphicsElement`以正确初始化飞船。

    ```java
    #ifndef _PACKT_SHIP_HPP_
    #define _PACKT_SHIP_HPP_

    #include "GraphicsManager.hpp"

    class Ship {
    public:
        Ship(android_app* pApplication,
             GraphicsManager& pGraphicsManager);

        void registerShip(GraphicsElement* pGraphics);

        void initialize();

    private:
        GraphicsManager& mGraphicsManager;

        GraphicsElement* mGraphics;
    };
    #endif
    ```

1.  实现`jni/Ship.cpp`。重要的是`initialize()`函数，它将飞船定位在屏幕的左下角，如下代码所示：

    ```java
    #include "Log.hpp"
    #include "Ship.hpp"
    #include "Types.hpp"

    static const float INITAL_X = 0.5f;
    static const float INITAL_Y = 0.25f;

    Ship::Ship(android_app* pApplication,
            GraphicsManager& pGraphicsManager) :
      mGraphicsManager(pGraphicsManager),
      mGraphics(NULL) {
    }

    void Ship::registerShip(GraphicsElement* pGraphics) {
        mGraphics = pGraphics;
    }

    void Ship::initialize() {
        mGraphics->location.x = INITAL_X
                * mGraphicsManager.getRenderWidth();
        mGraphics->location.y = INITAL_Y
                * mGraphicsManager.getRenderHeight();
    }
    ```

1.  将新创建的管理器和组件添加到`jni/DroidBlaster.hpp`：

    ```java
    ...
    #include "ActivityHandler.hpp"
    #include "EventLoop.hpp"
    #include "GraphicsManager.hpp"
    #include "Ship.hpp"
    #include "Types.hpp"

    class DroidBlaster : public ActivityHandler {
        ...
    private:
        ...

        GraphicsManager mGraphicsManager;
        EventLoop mEventLoop;

        Ship mShip;
    };
    #endif
    ```

1.  最后，更新`jni/DroidBlaster.cpp`构造函数：

    ```java
    ...
    static const int32_t SHIP_SIZE = 64;

    DroidBlaster::DroidBlaster(android_app* pApplication):
     mGraphicsManager(pApplication),
     mEventLoop(pApplication, *this),

     mShip(pApplication, mGraphicsManager) {
        Log::info("Creating DroidBlaster");

        GraphicsElement* shipGraphics = mGraphicsManager.registerElement(
     SHIP_SIZE, SHIP_SIZE);
     mShip.registerShip(shipGraphics);
    }
    ...
    ```

1.  在`onActivate()`中初始化`GraphicsManager`和`Ship`组件：

    ```java
    ...
    status DroidBlaster::onActivate() {
        Log::info("Activating DroidBlaster");

        if (mGraphicsManager.start() != STATUS_OK) return     STATUS_KO;

     mShip.initialize();

        return STATUS_OK;
    }
    ...
    ```

1.  最后，在`onStep()`中更新管理器：

    ```java
    ...
    status DroidBlaster::onStep() {
        return mGraphicsManager.update();
    }
    ```

## *刚才发生了什么？*

编译并运行`DroidBlaster`。结果应该是在屏幕的第一季度显示一个简单的红色方块，代表我们的太空船，如下所示：

![刚才发生了什么？](img/9645_05_04.jpg)

通过`ANativeWindow` API 提供图形反馈，它为显示窗口提供了本地访问。它允许像位图一样操作其表面。同样，访问窗口表面需要在处理前后进行锁定和解锁。

`AnativeWindow` API 在`android/native_window.h`和`android/native_window_jni.h`中定义。它提供以下功能：

`ANativeWindow_setBuffersGeometry()`初始化窗口缓冲区的像素格式（或深度格式）和大小。可能的像素格式有：

+   `WINDOW_FORMAT_RGBA_8888`每个像素 32 位颜色，红、绿、蓝和 Alpha（透明度）通道各 8 位。

+   `WINDOW_FORMAT_RGBX_8888`与上一个相同，只是忽略了 Alpha 通道。

+   `WINDOW_FORMAT_RGB_565`每个像素 16 位颜色（红和蓝 5 位，绿通道 6 位）。

如果提供的尺寸为`0`，则使用窗口大小。如果非零，则当在屏幕上显示时，窗口缓冲区会被缩放以匹配窗口尺寸：

```java
int32_t ANativeWindow_setBuffersGeometry(ANativeWindow* window, int32_t width, int32_t height, int32_t format);
```

+   在执行任何绘图操作之前必须调用`ANativeWindow_lock()`：

    ```java
    int32_t ANativeWindow_lock(ANativeWindow* window, ANativeWindow_Buffer* outBuffer,
            ARect* inOutDirtyBounds);
    ```

+   `ANativeWindow_unlockAndPost()`在绘图操作完成后释放窗口，并将其发送到显示。它必须与`ANativeWindow_lock()`成对调用：

    ```java
    int32_t ANativeWindow_unlockAndPost(ANativeWindow* window);
    ```

+   `ANativeWindow_acquire()`以 Java 方式获取指定窗口的引用，以防止潜在的删除。如果你对表面生命周期没有精细控制，这可能是有必要的：

    ```java
    void ANativeWindow_acquire(ANativeWindow* window);
    ```

+   `ANativeWindow_fromSurface()` 方法将窗口与给定的 Java `android.view.Surface` 关联。此方法会自动获取给定表面的引用。它必须通过 `ANativeWindow_release()` 释放，以避免内存泄漏：

    ```java
    ANativeWindow* ANativeWindow_fromSurface(JNIEnv* env, jobject surface);
    ```

+   `ANativeWindow_release()` 方法释放已获取的引用，以便释放窗口资源：

    ```java
    void ANativeWindow_release(ANativeWindow* window);
    ```

+   以下方法返回窗口表面的宽度、高度（以像素为单位）和格式。如果发生错误，返回值将为负。请注意，这些方法使用起来比较棘手，因为它们的行为有些不一致。在 Android 4 之前，最好锁定一次表面以获取可靠的信息（这已经由 `ANativeWindow_lock()` 提供了）：

    ```java
    int32_t ANativeWindow_getWidth(ANativeWindow* window);
    int32_t ANativeWindow_getHeight(ANativeWindow* window);
    int32_t ANativeWindow_getFormat(ANativeWindow* window);
    ```

现在我们知道如何绘制。但是，我们如何动画绘制的内容呢？为此需要一个关键因素：*时间*。

# 原生地测量时间

那些讨论图形的人也必须讨论定时。实际上，Android 设备具有不同的功能，动画应该适应它们的速度。为了帮助我们完成这项任务，Android 通过其出色的 Posix API 支持，提供了访问时间原语的方法。

为了实验这些功能，我们将使用定时器根据时间在屏幕上移动小行星。

### 注意

结果项目随本书提供，名为 `DroidBlaster_Part4`。

# 动手操作——使用定时器动画图形

让我们动画化游戏。

1.  创建 `jni/TimeManager.hpp` 文件，并在 `time.h` 管理器中定义以下方法：

    +   `reset()` 方法用于初始化管理器。

    +   `update()` 方法用于测量游戏步进时长。

    +   `elapsed()` 和 `elapsedTotal()` 方法用于获取游戏步进时长和游戏总时长。它们将允许应用程序行为适应设备速度。

    +   `now()` 是一个实用方法，用于重新计算当前时间。

    定义以下成员变量：

    +   `mFirstTime` 和 `mLastTime` 用于保存时间检查点，以便计算 `elapsed()` 和 `elapsedTotal()`

    +   `mElapsed` 和 `mElapsedTotal` 用于保存计算出来的时间测量值

        ```java
        #ifndef _PACKT_TIMEMANAGER_HPP_
        #define _PACKT_TIMEMANAGER_HPP_

        #include "Types.hpp"

        #include <ctime>

        class TimeManager {
        public:
            TimeManager();

            void reset();
            void update();

            double now();
            float elapsed() { return mElapsed; };
            float elapsedTotal() { return mElapsedTotal; };

        private:
            double mFirstTime;
            double mLastTime;
            float mElapsed;
            float mElapsedTotal;
        };
        #endif
        ```

1.  实现 `jni/TimeManager.cpp`。当重置 `TimeManager` 时，它会保存通过 `now()` 方法计算出的当前时间。

    ```java
    #include "Log.hpp"
    #include "TimeManager.hpp"

    #include <cstdlib>
    #include <time.h>

    TimeManager::TimeManager():
        mFirstTime(0.0f),
        mLastTime(0.0f),
        mElapsed(0.0f),
        mElapsedTotal(0.0f) {
        srand(time(NULL));
    }

    void TimeManager::reset() {
        Log::info("Resetting TimeManager.");
        mElapsed = 0.0f;
        mFirstTime = now();
        mLastTime = mFirstTime;
    }
    ...
    ```

1.  实现 `update()` 方法，该方法检查：

    +   自上一帧以来的经过时间在 `mElapsed` 中

    +   自第一帧以来的经过时间在 `mElapsedTotal` 中

        ### 注意

        注意，在处理当前时间时使用双精度类型很重要，以避免丢失精度。然后，可以将产生的延迟转换回浮点型，用于经过时间，因为两帧之间的时间差相当低。

        ```java
        ...
        void TimeManager::update() {
        	double currentTime = now();
        	mElapsed = (currentTime - mLastTime);
        	mElapsedTotal = (currentTime - mFirstTime);
        	mLastTime = currentTime;
        }
        ...
        ```

1.  在 `now()` 方法中计算当前时间。使用 Posix 原语 `clock_gettime()` 来获取当前时间。单调时钟至关重要，以确保时间始终向前推进，不受系统更改的影响（例如，如果用户环游世界）：

    ```java
    ...
    double TimeManager::now() {
        timespec timeVal;
        clock_gettime(CLOCK_MONOTONIC, &timeVal);
        return timeVal.tv_sec + (timeVal.tv_nsec * 1.0e-9);
    }
    ```

1.  创建一个新文件 `jni/PhysicsManager.hpp`。定义一个 `PhysicsBody` 结构体，用于保存小行星的位置、尺寸和速度：

    ```java
    #ifndef PACKT_PHYSICSMANAGER_HPP
    #define PACKT_PHYSICSMANAGER_HPP

    #include "GraphicsManager.hpp"
    #include "TimeManager.hpp"
    #include "Types.hpp"

    struct PhysicsBody {
        PhysicsBody(Location* pLocation, int32_t pWidth, int32_t pHeight):
            location(pLocation),
            width(pWidth), height(pHeight),
            velocityX(0.0f), velocityY(0.0f) {
        }

        Location* location;
        int32_t width; int32_t height;
        float velocityX; float velocityY;
    };
    ...
    ```

1.  定义一个基本的`PhysicsManager`。我们需要对`TimeManager`的引用，以将运动体的移动适应到时间。

    定义一个`update()`方法，在每个游戏步骤中移动小行星。`PhysicsManager`在`mPhysicsBodies`和`mPhysicsBodyCount`中存储要更新的小行星：

    ```java
    ...
    class PhysicsManager {
    public:
        PhysicsManager(TimeManager& pTimeManager,
                GraphicsManager& pGraphicsManager);
        ~PhysicsManager();

        PhysicsBody* loadBody(Location& pLocation, int32_t pWidth,
                int32_t pHeight);
        void update();

    private:
        TimeManager& mTimeManager;
        GraphicsManager& mGraphicsManager;

        PhysicsBody* mPhysicsBodies[1024]; int32_t mPhysicsBodyCount;
    };
    #endif
    ```

1.  实现`jni/PhysicsManager.cpp`，从构造函数、析构函数和注册方法开始：

    ```java
    #include "PhysicsManager.hpp"
    #include "Log.hpp"

    PhysicsManager::PhysicsManager(TimeManager& pTimeManager,
            GraphicsManager& pGraphicsManager) :
      mTimeManager(pTimeManager), mGraphicsManager(pGraphicsManager),
      mPhysicsBodies(), mPhysicsBodyCount(0) {
        Log::info("Creating PhysicsManager.");
    }

    PhysicsManager::~PhysicsManager() {
        Log::info("Destroying PhysicsManager.");
        for (int32_t i = 0; i < mPhysicsBodyCount; ++i) {
            delete mPhysicsBodies[i];
        }
    }

    PhysicsBody* PhysicsManager::loadBody(Location& pLocation,
            int32_t pSizeX, int32_t pSizeY) {
        PhysicsBody* body = new PhysicsBody(&pLocation, pSizeX, pSizeY);
        mPhysicsBodies[mPhysicsBodyCount++] = body;
        return body;
    }
    ...
    ```

1.  在`update()`中根据它们的速度移动小行星。计算根据两个游戏步骤之间的时间量进行：

    ```java
    ...
    void PhysicsManager::update() {
        float timeStep = mTimeManager.elapsed();
        for (int32_t i = 0; i < mPhysicsBodyCount; ++i) {
            PhysicsBody* body = mPhysicsBodies[i];
            body->location->x += (timeStep * body->velocityX);
            body->location->y += (timeStep * body->velocityY);
        }
    }
    ```

1.  使用以下方法创建`jni/Asteroid.hpp`组件：

    +   `initialize()`在游戏开始时设置具有随机属性的小行星

    +   `update()`用于检测越出游戏边界的小行星。

    +   `spawn()`被`initialize()`和`update()`两者使用，以设置一个单独的小行星

    我们还需要以下成员：

    +   `mBodies`和`mBodyCount`用于存储要管理的小行星列表

    +   几个整数成员用于存储游戏边界

        ```java
        #ifndef _PACKT_ASTEROID_HPP_
        #define _PACKT_ASTEROID_HPP_

        #include "GraphicsManager.hpp"
        #include "PhysicsManager.hpp"
        #include "TimeManager.hpp"
        #include "Types.hpp"

        class Asteroid {
        public:
            Asteroid(android_app* pApplication,
                TimeManager& pTimeManager, GraphicsManager& pGraphicsManager,
                PhysicsManager& pPhysicsManager);

            void registerAsteroid(Location& pLocation, int32_t pSizeX,
                    int32_t pSizeY);

            void initialize();
            void update();

        private:
            void spawn(PhysicsBody* pBody);

            TimeManager& mTimeManager;
            GraphicsManager& mGraphicsManager;
            PhysicsManager& mPhysicsManager;

            PhysicsBody* mBodies[1024]; int32_t mBodyCount;
            float mMinBound;
            float mUpperBound; float mLowerBound;
            float mLeftBound; float mRightBound;
        };
        #endif
        ```

1.  编写`jni/Asteroid.cpp`的实现。从一些常量以及构造函数和注册方法开始，如下所示：

    ```java
    #include "Asteroid.hpp"
    #include "Log.hpp"

    static const float BOUNDS_MARGIN = 128;
    static const float MIN_VELOCITY = 150.0f, VELOCITY_RANGE = 600.0f;

    Asteroid::Asteroid(android_app* pApplication,
            TimeManager& pTimeManager, GraphicsManager& pGraphicsManager,
            PhysicsManager& pPhysicsManager) :
        mTimeManager(pTimeManager),
        mGraphicsManager(pGraphicsManager),
        mPhysicsManager(pPhysicsManager),
        mBodies(), mBodyCount(0),
        mMinBound(0.0f),
        mUpperBound(0.0f), mLowerBound(0.0f),
        mLeftBound(0.0f), mRightBound(0.0f) {
    }

    void Asteroid::registerAsteroid(Location& pLocation,
            int32_t pSizeX, int32_t pSizeY) {
        mBodies[mBodyCount++] = mPhysicsManager.loadBody(pLocation,
                pSizeX, pSizeY);
    }
    ...
    ```

1.  在`initialize()`中设置边界。小行星在屏幕顶部以上生成（在`mMinBound`中，最大边界`mUpperBound`是屏幕高度的兩倍）。它们从屏幕顶部移动到底部。其他边界对应于边缘带有边距的屏幕（代表小行星大小的两倍）。

    然后，使用`spawn()`初始化所有小行星：

    ```java
    ...
    void Asteroid::initialize() {
        mMinBound = mGraphicsManager.getRenderHeight();
        mUpperBound = mMinBound * 2;
        mLowerBound = -BOUNDS_MARGIN;
        mLeftBound = -BOUNDS_MARGIN;
        mRightBound = (mGraphicsManager.getRenderWidth() + BOUNDS_MARGIN);

        for (int32_t i = 0; i < mBodyCount; ++i) {
            spawn(mBodies[i]);
        }
    }
    ...
    ```

1.  在每个游戏步骤中，检查越界的小行星并重新初始化它们：

    ```java
    ...
    void Asteroid::update() {
        for (int32_t i = 0; i < mBodyCount; ++i) {
            PhysicsBody* body = mBodies[i];
            if ((body->location->x < mLeftBound)
             || (body->location->x > mRightBound)
             || (body->location->y < mLowerBound)
             || (body->location->y > mUpperBound)) {
                spawn(body);
            }
        }
    }
    ...
    ```

1.  最后，在`spawn()`中根据生成的随机速度和位置初始化每个小行星：

    ```java
    ...
    void Asteroid::spawn(PhysicsBody* pBody) {
        float velocity = -(RAND(VELOCITY_RANGE) + MIN_VELOCITY);
        float posX = RAND(mGraphicsManager.getRenderWidth());
        float posY = RAND(mGraphicsManager.getRenderHeight())
                      + mGraphicsManager.getRenderHeight();

        pBody->velocityX = 0.0f;
        pBody->velocityY = velocity;
        pBody->location->x = posX;
        pBody->location->y = posY;
    }
    ```

1.  将新创建的管理器和组件添加到`jni/DroidBlaster.hpp`中：

    ```java
    #ifndef _PACKT_DROIDBLASTER_HPP_
    #define _PACKT_DROIDBLASTER_HPP_

    #include "ActivityHandler.hpp"
    #include "Asteroid.hpp"
    #include "EventLoop.hpp"
    #include "GraphicsManager.hpp"
    #include "PhysicsManager.hpp"
    #include "Ship.hpp"
    #include "TimeManager.hpp"
    #include "Types.hpp"

    class DroidBlaster : public ActivityHandler {
        ...
    private:
        TimeManager     mTimeManager;
        GraphicsManager mGraphicsManager;
        PhysicsManager  mPhysicsManager;
        EventLoop mEventLoop;

        Asteroid mAsteroids;
        Ship mShip;
    };
    #endif
    ```

1.  在`jni/DroidBlaster.cpp`构造函数中，使用`GraphicsManager`和`PhysicsManager`注册小行星：

    ```java
    ...
    static const int32_t SHIP_SIZE = 64;
    static const int32_t ASTEROID_COUNT = 16;
    static const int32_t ASTEROID_SIZE = 64;

    DroidBlaster::DroidBlaster(android_app* pApplication):
        mTimeManager(),
        mGraphicsManager(pApplication),
        mPhysicsManager(mTimeManager, mGraphicsManager),
        mEventLoop(pApplication, *this),

        mAsteroids(pApplication, mTimeManager, mGraphicsManager,
     mPhysicsManager),
        mShip(pApplication, mGraphicsManager) {
        Log::info("Creating DroidBlaster");

        GraphicsElement* shipGraphics = mGraphicsManager.registerElement(
                SHIP_SIZE, SHIP_SIZE);
        mShip.registerShip(shipGraphics);

        for (int32_t i = 0; i < ASTEROID_COUNT; ++i) {
     GraphicsElement* asteroidGraphics =
     mGraphicsManager.registerElement(ASTEROID_SIZE,
     ASTEROID_SIZE);
     mAsteroids.registerAsteroid(
     asteroidGraphics->location, ASTEROID_SIZE,
     ASTEROID_SIZE);
        }
    }
    ...
    ```

1.  在`onActivate()`中适当地初始化新添加的类：

    ```java
    ...
    status DroidBlaster::onActivate() {
        Log::info("Activating DroidBlaster");

        if (mGraphicsManager.start() != STATUS_OK) return STATUS_KO;

        mAsteroids.initialize();
        mShip.initialize();

        mTimeManager.reset();
        return STATUS_OK;
    }
    ...
    Finally, update managers and components for each game step:
    ...
    status DroidBlaster::onStep() {
        mTimeManager.update();
        mPhysicsManager.update();

        mAsteroids.update();

        return mGraphicsManager.update();
    }
    ...
    ```

## *刚才发生了什么？*

编译并运行应用程序。这次它应该会有些动画效果！代表小行星的红色方块以恒定的节奏穿过屏幕。`TimeManger`有助于设置这个节奏。

![刚才发生了什么？](img/9645_05_05.jpg)

定时器对于以正确速度显示动画和移动至关重要。它们可以通过 POSIX 方法`clock_gettime()`实现，该方法以高精度获取时间，理论上可以达到纳秒级。

在本教程中，我们使用了`CLOCK_MONOTONIC`标志来设置定时器。单调时钟提供了一个从过去任意时间点开始的经过的时钟时间。它不受系统日期变更的影响，因此不会像其他选项那样回到过去。`CLOCK_MONOTONIC`的缺点是它是系统特定的，并且不保证支持。幸运的是，Android 支持它，但当将 Android 代码移植到其他平台时，应该注意。另一个特定于 Android 需要注意的点是，当系统挂起时，单调时钟会停止。

另一个选择，不那么精确，且受系统时间变化（这可能是可取的或不可取的）的影响，是`gettimeofday()`，它同样在`ctime`中提供。用法相似，但精度是微秒而不是纳秒。以下可能是一个可以替换`TimeManager`中当前`now()`实现的用法示例：

```java
double TimeManager::now() {
    timeval lTimeVal;
    gettimeofday(&lTimeVal, NULL);
    return (lTimeVal.tv_sec * 1000.0) + (lTimeVal.tv_usec / 1000.0);
}
```

想了解更多信息，请查看[`man7.org/linux/man-pages/man2/clock_gettime.2.html`](http://man7.org/linux/man-pages/man2/clock_gettime.2.html)的 Man 页面。

# 概括

Android NDK 使我们能够编写完全本地化的应用程序，而无需一行 Java 代码。`NativeActivity`提供了一个框架，以实现处理应用程序事件的事件循环。结合 Posix 时间管理 API，NDK 提供了构建复杂多媒体应用程序或游戏所需的基础。

总结一下，我们创建了`NativeActivity`来轮询活动事件，以便相应地启动或停止本地代码。我们原生地访问显示窗口，就像位图一样，以显示原始图形。最后，我们获取了时间，使应用程序能够使用单调时钟适应设备速度。

这里启动的基本框架将作为我们将在本书中开发的 2D/3D 游戏的基础。然而，尽管现在的扁平化设计很流行，但我们需要的不仅仅是红色的方块！

在下一章中，我们将了解如何使用 OpenGL ES 2 为 Android 渲染高级图形。
