# 第八章：处理输入设备和传感器

> *Android 的一切都是关于互动。诚然，这意味着通过图形、音频、振动等方式进行反馈。但没有输入就没有互动！当今智能手机的成功源于它们多样化和现代的输入方式：触摸屏、键盘、鼠标、GPS、加速度计、光线检测器、声音记录器等等。正确处理和结合它们是丰富您的应用程序并使其成功的关键。*

尽管 Android 处理许多输入外设，但 Android NDK 在其支持上长期以来非常有限（甚至可以说是最少），直到 R5 版本的发布！我们现在可以通过原生 API 直接访问。可用的设备实例包括：

+   键盘，可以是物理键盘（带有滑出式键盘）或虚拟键盘（在屏幕上显示）

+   方向键（上、下、左、右和动作按钮），通常简称为 D-Pad。

+   轨迹球，包括光学轨迹球

+   触摸屏，使现代智能手机成功

+   鼠标或触摸板（自 NDK R5 起，但仅在 Honeycomb 设备上可用）

我们还可以访问以下硬件传感器：

+   加速度计，测量施加在设备上的线性加速度。

+   陀螺仪，测量角速度。它通常与磁力计结合使用，以准确快速地计算方向。陀螺仪是最近引入的，并且大多数设备上还不可用。

+   磁力计，提供环境磁场，从而得出基本方向。

+   光传感器，例如，自动适应屏幕亮度。

+   近距离传感器，例如，在通话期间检测耳朵的距离。

除了硬件传感器外，Gingerbread 版本还引入了“软件传感器”。这些传感器源自硬件传感器的数据：

+   重力传感器，测量重力的方向和大小

+   线性加速度传感器，测量设备“移动”时排除重力的部分

+   旋转矢量，表示设备在空间中的方向

重力传感器和线性加速度传感器源自加速度计。另一方面，旋转矢量由磁力计和加速度计派生。由于这些传感器通常需要计算一段时间，因此它们在获取最新值时通常会有轻微的延迟。

为了更深入地了解输入设备和传感器，本章将介绍如何：

+   处理屏幕触摸

+   检测键盘、方向键和轨迹球事件

+   将加速度计传感器转变为游戏手柄

# 与触摸事件互动

当今智能手机最具标志性的创新是触摸屏，它已经取代了现已过时的鼠标。正如其名，触摸屏可以检测手指或手写笔在设备表面的触摸。根据屏幕的质量，可以处理多个触摸（在 Android 中也称为光标），从而增加了互动的可能性。

让我们从在 `DroidBlaster` 中处理触摸事件开始本章的内容。为了简化示例，我们只处理单一的“触摸”。目标是使飞船向触摸的方向移动。触摸越远，飞船移动越快。超过预定义的范围 `TOUCH_MAX_RANGE`，飞船的速度将达到其速度极限，如下图所示：

![与触摸事件交互](img/9645OS_08_01.jpg)

### 注意

本书提供的项目名为 `DroidBlaster_Part13`。

# 动手实践——处理触摸事件

让我们在 `DroidBlaster` 中拦截触摸事件：

1.  正如在 第五章《编写一个完全原生的应用程序》中创建 `ActivityHandler` 来处理应用程序事件一样，创建 `jni/InputHandler.hpp` 来处理输入事件。输入 API 在 `android/input.h` 中声明。创建 `onTouchEvent()` 来处理触摸事件。这些事件被封装在 `AInputEvent` 结构中。其他输入外设将在本章后面描述：

    ```java
    #ifndef _PACKT_INPUTHANDLER_HPP_
    #define _PACKT_INPUTHANDLER_HPP_

    #include <android/input.h>

    class InputHandler {
    public:
        virtual ~InputHandler() {};

        virtual bool onTouchEvent(AInputEvent* pEvent) = 0;
    };
    #endif
    ```

1.  修改 `jni/EventLoop.hpp` 头文件，包含并处理 `InputHandler` 实例。

    类似于活动事件，定义一个内部方法 `processInputEvent()`，该方法由静态回调 `callback_input()` 触发：

    ```java
    ...
    #include "ActivityHandler.hpp"
    #include "InputHandler.hpp"

    #include <android_native_app_glue.h>

    class EventLoop {
    public:
    EventLoop(android_app* pApplication,
                ActivityHandler& pActivityHandler,
                InputHandler& pInputHandler);
        ...
    private:
        ...
        void processAppEvent(int32_t pCommand);
        int32_t processInputEvent(AInputEvent* pEvent);

        static void callback_appEvent(android_app* pApplication,
                int32_t pCommand);
        static int32_t callback_input(android_app* pApplication,
     AInputEvent* pEvent);

        ...
        ActivityHandler& mActivityHandler;
        InputHandler& mInputHandler;
    };
    #endif
    ```

1.  我们需要在 `jni/EventLoop.cpp` 源文件中处理输入事件，并通知相关的 `InputHandler`。

    首先，将 Android 输入队列连接到 `callback_input()`。`EventLoop` 本身（即 `this`）通过 `android_app` 结构的 `userData` 成员匿名传递。这样，回调能够将输入处理委托回我们自己的对象，即 `processInputEvent()`。

    ```java
    ...
    EventLoop::EventLoop(android_app* pApplication,
        ActivityHandler& pActivityHandler, InputHandler& pInputHandler):
            mApplication(pApplication),
            mActivityHandler(pActivityHandler),
            mEnabled(false), mQuit(false),
            mInputHandler(pInputHandler) {
        mApplication->userData = this;
        mApplication->onAppCmd = callback_appEvent;
     mApplication->onInputEvent = callback_input;
    }

    ...

    int32_t EventLoop::callback_input(android_app* pApplication,
     AInputEvent* pEvent) {
     EventLoop& eventLoop = *(EventLoop*) pApplication->userData;
     return eventLoop.processInputEvent(pEvent);
    }
    ...
    ```

1.  触摸屏事件属于 `MotionEvent` 类型（与按键事件相对）。通过 Android 原生输入 API（此处为 `AinputEvent_getSource()`），可以根据它们的来源（`AINPUT_SOURCE_TOUCHSCREEN`）来区分它们。

    ### 注意

    请注意 `callback_input()` 以及扩展的 `processInputEvent()` 返回一个整数值（本质上是一个布尔值）。这个值表示输入事件（例如，按下的按钮）已经被应用程序处理，不需要系统进一步处理。例如，当按下返回按钮时返回 `1`，以停止事件处理，防止活动被终止。

    ```java
    ...
    int32_t EventLoop::processInputEvent(AInputEvent* pEvent) {
        if (!mEnabled) return 0;

        int32_t eventType = AInputEvent_getType(pEvent);
        switch (eventType) {
        case AINPUT_EVENT_TYPE_MOTION:
            switch (AInputEvent_getSource(pEvent)) {
            case AINPUT_SOURCE_TOUCHSCREEN:
                return mInputHandler.onTouchEvent(pEvent);
                break;
            }
            break;
        }
        return 0;
    }
    ```

1.  创建 `jni/InputManager.hpp` 以处理触摸事件并实现我们的新 `InputHandler` 接口。

    按如下方式定义方法：

    +   `start()` 执行必要的初始化。

    +   `onTouchEvent()` 在触发新事件时更新管理器状态。

    +   `getDirectionX()` 和 `getDirectionY()` 指示飞船的方向。

    +   `setRefPoint()` 指的是飞船的位置。实际上，方向被定义为触摸点和飞船位置（即参考点）之间的向量。

    同时，声明必要的成员变量，尤其是`mScaleFactor`，它包含了从屏幕坐标到游戏坐标的正确比例（记住我们使用的是固定大小）。

    ```java
    #ifndef _PACKT_INPUTMANAGER_HPP_
    #define _PACKT_INPUTMANAGER_HPP_

    #include "GraphicsManager.hpp"
    #include "InputHandler.hpp"
    #include "Types.hpp"

    #include <android_native_app_glue.h>

    class InputManager : public InputHandler {
    public:
        InputManager(android_app* pApplication,
                 GraphicsManager& pGraphicsManager);

        float getDirectionX() { return mDirectionX; };
        float getDirectionY() { return mDirectionY; };
        void setRefPoint(Location* pRefPoint) { mRefPoint = pRefPoint; };

        void start();

    protected:
        bool onTouchEvent(AInputEvent* pEvent);

    private:
        android_app* mApplication;
        GraphicsManager& mGraphicsManager;

        // Input values.
        float mScaleFactor;
        float mDirectionX, mDirectionY;
        // Reference point to evaluate touch distance.
        Location* mRefPoint;
    };
    #endif
    ```

1.  创建`jni/InputManager.cpp`，从构造函数开始：

    ```java
    #include "InputManager.hpp"
    #include "Log.hpp"

    #include <android_native_app_glue.h>
    #include <cmath>

    InputManager::InputManager(android_app* pApplication,
            GraphicsManager& pGraphicsManager) :
        mApplication(pApplication), mGraphicsManager(pGraphicsManager),
        mDirectionX(0.0f), mDirectionY(0.0f),
        mRefPoint(NULL) {
    }
    ...
    ```

1.  编写`start()`方法以清除成员变量并计算缩放因子。这个缩放因子是必要的，因为在第六章，*使用 OpenGL ES 渲染图形*中提到，我们需要将输入事件提供的屏幕坐标（这取决于设备）转换为游戏坐标：

    ```java
    ...
    void InputManager::start() {
        Log::info("Starting InputManager.");
        mDirectionX = 0.0f, mDirectionY = 0.0f;
        mScaleFactor = float(mGraphicsManager.getRenderWidth())
                           / float(mGraphicsManager.getScreenWidth());
    }
    ...
    ```

1.  有效的事件处理在`onTouchEvent()`中实现。根据参考点与触摸点之间的距离，计算出水平方向和垂直方向。这个距离通过`TOUCH_MAX_RANGE`限制在一个任意的`65`单位范围内。因此，当参考点到触摸点的距离超出`TOUCH_MAX_RANGE`像素时，飞船将达到最大速度。

    当你移动手指时，通过`AMotionEvent_getX()`和`AMotionEvent_getY()`获取触摸坐标。当不再检测到触摸时，方向向量重置为`0`：

    ```java
    ...
    bool InputManager::onTouchEvent(AInputEvent* pEvent) {
        static const float TOUCH_MAX_RANGE = 65.0f; // In game units.

        if (mRefPoint != NULL) {
            if (AMotionEvent_getAction(pEvent)
                            == AMOTION_EVENT_ACTION_MOVE) {
                float x = AMotionEvent_getX(pEvent, 0) * mScaleFactor;
                float y = (float(mGraphicsManager.getScreenHeight())
                         - AMotionEvent_getY(pEvent, 0)) * mScaleFactor;
                // Needs a conversion to proper coordinates
                // (origin at bottom/left). Only moveY needs it.
                float moveX = x - mRefPoint->x;
                float moveY = y - mRefPoint->y;
                float moveRange = sqrt((moveX * moveX) + (moveY * moveY));

                if (moveRange > TOUCH_MAX_RANGE) {
                    float cropFactor = TOUCH_MAX_RANGE / moveRange;
                    moveX *= cropFactor; moveY *= cropFactor;
                }

                mDirectionX = moveX / TOUCH_MAX_RANGE;
                mDirectionY   = moveY / TOUCH_MAX_RANGE;
            } else {
                mDirectionX = 0.0f; mDirectionY = 0.0f;
            }
        }
        return true;
    }
    ```

1.  创建一个简单的组件`jni/MoveableBody.hpp`，其作用是根据输入事件移动`PhysicsBody`：

    ```java
    #ifndef _PACKT_MOVEABLEBODY_HPP_
    #define _PACKT_MOVEABLEBODY_HPP_

    #include "InputManager.hpp"
    #include "PhysicsManager.hpp"
    #include "Types.hpp"

    class MoveableBody {
    public:
        MoveableBody(android_app* pApplication,
           InputManager& pInputManager, PhysicsManager& pPhysicsManager);

        PhysicsBody* registerMoveableBody(Location& pLocation,
                int32_t pSizeX, int32_t pSizeY);

        void initialize();
        void update();

    private:
        PhysicsManager& mPhysicsManager;
        InputManager& mInputManager;

        PhysicsBody* mBody;
    };
    #endif
    ```

1.  在`jni/MoveableBody.cpp`中实现这个组件。

    在`registerMoveableBody()`中，将`InputManager`和物体绑定：

    ```java
    #include "Log.hpp"
    #include "MoveableBody.hpp"

    MoveableBody::MoveableBody(android_app* pApplication,
          InputManager& pInputManager, PhysicsManager& pPhysicsManager) :
        mInputManager(pInputManager),
        mPhysicsManager(pPhysicsManager),
        mBody(NULL) {
    }

    PhysicsBody* MoveableBody::registerMoveableBody(Location& pLocation,
    int32_t pSizeX, int32_t pSizeY) {
        mBody = mPhysicsManager.loadBody(pLocation, pSizeX, pSizeY);
        mInputManager.setRefPoint(&pLocation);
        return mBody;
    }
    ...
    ```

1.  最初，物体没有速度。

    然后，每次更新时，速度都会反映当前的输入状态。这个速度由第五章，*编写一个完全原生的应用程序*中创建的`PhysicsManager`接收，以更新实体的位置：

    ```java
    ...
    void MoveableBody::initialize() {
        mBody->velocityX = 0.0f;
        mBody->velocityY = 0.0f;
    }

    void MoveableBody::update() {
        static const float MOVE_SPEED = 320.0f;
        mBody->velocityX = mInputManager.getDirectionX() * MOVE_SPEED;
        mBody->velocityY = mInputManager.getDirectionY() * MOVE_SPEED;
    }
    ```

    在`jni/DroidBlaster.hpp`中引用新的`InputManager`和`MoveableComponent`：

    ```java
    ...
    #include "EventLoop.hpp"
    #include "GraphicsManager.hpp"
    #include "InputManager.hpp"
    #include "MoveableBody.hpp"
    #include "PhysicsManager.hpp"
    #include "Resource.hpp"
    ...

    class DroidBlaster : public ActivityHandler {
        ...
    private:
        TimeManager     mTimeManager;
        GraphicsManager mGraphicsManager;
        PhysicsManager  mPhysicsManager;
        SoundManager    mSoundManager;
        InputManager    mInputManager;
        EventLoop mEventLoop;
        ...
        Asteroid mAsteroids;
        Ship mShip;
        StarField mStarField;
        SpriteBatch mSpriteBatch;
        MoveableBody mMoveableBody;
    };
    #endif
    ```

1.  最后，调整`jni/DroidBlaster.cpp`构造函数，以实例化`InputManager`和`MoveableComponent`。

    在构造时，将`InputManager`添加到`EventLoop`中，后者负责分派输入事件。

    飞船是被移动的实体。因此，需要将它的位置引用传递给`MoveableBody`组件：

    ```java
    ...
    DroidBlaster::DroidBlaster(android_app* pApplication):
        mTimeManager(),
        mGraphicsManager(pApplication),
        mPhysicsManager(mTimeManager, mGraphicsManager),
        mSoundManager(pApplication),
        mInputManager(pApplication, mGraphicsManager),
     mEventLoop(pApplication, *this, mInputManager),
        ...
        mAsteroids(pApplication, mTimeManager, mGraphicsManager,
        mPhysicsManager),
        mShip(pApplication, mGraphicsManager, mSoundManager),
        mStarField(pApplication, mTimeManager, mGraphicsManager,
                STAR_COUNT, mStarTexture),
        mSpriteBatch(mTimeManager, mGraphicsManager),
        mMoveableBody(pApplication, mInputManager, mPhysicsManager) {
        ...
        Sprite* shipGraphics = mSpriteBatch.registerSprite(mShipTexture,
                SHIP_SIZE, SHIP_SIZE);
        shipGraphics->setAnimation(SHIP_FRAME_1, SHIP_FRAME_COUNT,
                SHIP_ANIM_SPEED, true);
        Sound* collisionSound =
                mSoundManager.registerSound(mCollisionSound);
        mMoveableBody.registerMoveableBody(shipGraphics->location,
     SHIP_SIZE, SHIP_SIZE);
        mShip.registerShip(shipGraphics, collisionSound);

        // Creates asteroids.
        ...
    }
    ...
    ```

1.  在相应的函数中初始化和更新`MoveableBody`和`InputManager`：

    ```java
    ...
    status DroidBlaster::onActivate() {
        Log::info("Activating DroidBlaster");
        if (mGraphicsManager.start() != STATUS_OK) return STATUS_KO;
        if (mSoundManager.start() != STATUS_OK) return STATUS_KO;
        mInputManager.start();

        mSoundManager.playBGM(mBGM);

        mAsteroids.initialize();
        mShip.initialize();
        mMoveableBody.initialize();

        mTimeManager.reset();
        return STATUS_OK;
    }

    ...

    status DroidBlaster::onStep() {
        mTimeManager.update();
        mPhysicsManager.update();

        mAsteroids.update();
        mMoveableBody.update();

        return mGraphicsManager.update();
    }
    ...
    ```

## *刚才发生了什么？*

我们创建了一个基于触摸事件的输入系统的简单示例。飞船以与触摸距离相关的速度向触摸点飞行。触摸事件坐标是绝对的。它们的原点在屏幕左上角，与 OpenGL 的左下角相对。如果应用程序允许屏幕旋转，那么屏幕原点对于用户来说仍然在左上角，无论设备是纵向还是横向模式。

![刚才发生了什么？](img/9645OS_08_07.jpg)

为了实现这个新特性，我们将事件循环连接到了`native_app_glue`模块提供的输入事件队列。这个队列在内部表现为一个 UNIX 管道，类似于活动事件队列。触摸屏事件嵌入在`AInputEvent`结构中，该结构还存储其他类型的输入事件。使用`android/input.h`中声明的`AInputEvent`和`AMotionEvent` API 处理输入事件。`AInputEvent` API 通过`AInputEvent_getType()`和`AInputEvent_getSource()`方法来区分输入事件类型。而`AMotionEvent` API 仅提供处理触摸事件的方法。

触摸 API 相当丰富。许多细节可以按照以下表格所示请求（非详尽无遗）：

| 方法 | 描述 |
| --- | --- |

|

```java
AMotionEvent_getAction()
```

| 用于检测手指是否与屏幕接触、离开或在其表面移动。结果是一个整数值，由事件类型（例如，第 1 个字节上的`AMOTION_EVENT_ACTION_DOWN`）和一个指针索引（在第 2 个字节上，以了解事件指的是哪个手指）组成。 |
| --- |

|

```java
AMotionEvent_getX()
AMotionEvent_getY()
```

| 用于获取屏幕上的触摸坐标，以浮点数（像素）表示（可能存在亚像素值）。 |
| --- |

|

```java
AMotionEvent_getDownTime()
AMotionEvent_getEventTime()
```

| 用于获取手指在屏幕上滑动的时间和事件生成的时间（单位为纳秒）。 |
| --- |

|

```java
AMotionEvent_getPressure()
AMotionEvent_getSize()

```

| 用于检测压力强度和区域。值通常在`0.0`到`1.0`之间（但可能会超出）。大小和压力通常密切相关。行为可能会因硬件而异并产生噪声。 |
| --- |

|

```java
AMotionEvent_getHistorySize()
AMotionEvent_getHistoricalX()
AMotionEvent_getHistoricalY()
```

| 为了提高效率，可以将类型为`AMOTION_EVENT_ACTION_MOVE`的触摸事件分组在一起。这些方法提供了对发生在前一个事件和当前事件之间的这些*历史点*的访问。 |
| --- |

查看完整的`android/input.h`方法列表。

如果你深入查看`AMotionEvent` API，你会注意到一些事件有一个第二个参数`pointer_index`，其范围在`0`到活动指针的数量之间。实际上，当今大多数触摸屏都支持多点触控！屏幕上的两个或更多手指（如果硬件支持）在 Android 中由两个或更多指针表示。要操作它们，请查看以下表格：

| 方法 | 描述 |
| --- | --- |

|

```java
AMotionEvent_getPointerCount()
```

| 用于了解有多少手指触摸屏幕。 |
| --- |

|

```java
AMotionEvent_getPointerId()
```

| 从指针索引获取一个指针的唯一标识符。这是跟踪特定指针（即*手指*）随时间变化唯一的方式，因为当手指触摸或离开屏幕时其索引可能会改变。 |
| --- |

### 提示

如果你关注过（现在已成古董的！）Nexus One 的故事，那么你应该知道它曾因硬件缺陷而出名。指针经常混淆，其中两个会交换它们的坐标。因此，一定要准备好处理硬件的特定行为或表现异常的硬件！

# 检测键盘、D-Pad 和轨迹球事件

在所有的输入设备中，最常见的是键盘。这对于 Android 来说也是如此。Android 的键盘可以是物理的：位于设备正面（如传统的黑莓手机），或者是在滑出式屏幕上的。然而，键盘通常是虚拟的，即在屏幕上模拟，这会占用大量的空间。除了键盘本身，每个 Android 设备都必须包括一些物理或模拟的按钮，如**菜单**、**主页**和**任务**。

一种不太常见的输入设备类型是方向键。方向键是一组物理按钮，用于向上、向下、向左或向右移动以及特定的动作/确认按钮。尽管它们经常从最近的手機和平板电脑上消失，但方向键仍然是在文本或 UI 小部件之间移动的最方便方式之一。方向键通常被轨迹球取代。轨迹球的行为类似于鼠标（带有一个球体的鼠标）并倒置。一些轨迹球是模拟的，但其他（例如，光学的）则表现为方向键（即全有或全无）。

![检测键盘、方向键和轨迹球事件](img/9645OS_08_02.jpg)

为了了解它们是如何工作的，让我们在`DroidBlaster`中使用这些外设来移动我们的太空船。现在 Android NDK 允许在本地处理所有这些输入外设。那么，让我们试试看！

### 注意

本书提供的结果项目名为`DroidBlaster_Part14`。

# 动手实践——本地处理键盘、方向键和轨迹球事件

让我们扩展我们新的输入系统，加入更多的事件类型：

1.  打开`jni/InputHandler.hpp`并添加键盘和轨迹球事件处理程序：

    ```java
    #ifndef _PACKT_INPUTHANDLER_HPP_
    #define _PACKT_INPUTHANDLER_HPP_

    #include <android/input.h>

    class InputHandler {
    public:
        virtual ~InputHandler() {};

        virtual bool onTouchEvent(AInputEvent* pEvent) = 0;
        virtual bool onKeyboardEvent(AInputEvent* pEvent) = 0;
     virtual bool onTrackballEvent(AInputEvent* pEvent) = 0;
    };
    #endif
    ```

1.  更新现有文件`jni/EventLoop.cpp`中的`processInputEvent()`方法，将键盘和轨迹球事件重定向到`InputHandler`。

    轨迹球和触摸事件被归纳为动作事件，可以根据它们的来源进行区分。相对的，按键事件根据它们的类型进行区分。实际上，存在两个专用的 API，一个用于`MotionEvents`（轨迹球和触摸事件相同），另一个用于`KeyEvents`（键盘、方向键等相同）。

    ```java
    ...
    int32_t EventLoop::processInputEvent(AInputEvent* pEvent) {
        if (!mEnabled) return 0;

        int32_t eventType = AInputEvent_getType(pEvent);
        switch (eventType) {
        case AINPUT_EVENT_TYPE_MOTION:
            switch (AInputEvent_getSource(pEvent)) {
            case AINPUT_SOURCE_TOUCHSCREEN:
                return mInputHandler.onTouchEvent(pEvent);
                break;

            case AINPUT_SOURCE_TRACKBALL:
     return mInputHandler.onTrackballEvent(pEvent);
     break;
            }
            break;

        case AINPUT_EVENT_TYPE_KEY:
     return mInputHandler.onKeyboardEvent(pEvent);
     break;
        }
    return 0;
    }
    ...
    ```

1.  修改`jni/InputManager.hpp`文件，重写这些新方法：

    ```java
    ...
    class InputManager : public InputHandler {
        ...
    protected:
        bool onTouchEvent(AInputEvent* pEvent);
        bool onKeyboardEvent(AInputEvent* pEvent);
     bool onTrackballEvent(AInputEvent* pEvent);

        ...
    };
    #endif
    ```

1.  在`jni/InputManager.cpp`中，使用`onKeyboardEvent()`处理键盘事件：

    +   使用`AKeyEvent_getAction()`获取事件类型（即按下或未按下）。

    +   使用`AKeyEvent_getKeyCode()`获取按钮标识。

    在以下代码中，当按下左、右、上或下按钮时，`InputManager`会计算方向并将其保存到`mDirectionX`和`mDirectionY`中。按钮按下时开始移动，松开时停止。

    当按键被消耗时返回`true`，否则返回`false`。实际上，如果用户按下了例如返回键（`AKEYCODE_BACK`）或音量键（`AKEYCODE_VOLUME_UP`，`AKEYCODE_VOLUME_DOWN`），我们就让系统为我们做出适当的反应：

    ```java
    ...
    bool InputManager::onKeyboardEvent(AInputEvent* pEvent) {
        static const float ORTHOGONAL_MOVE = 1.0f;

        if (AKeyEvent_getAction(pEvent) == AKEY_EVENT_ACTION_DOWN) {
            switch (AKeyEvent_getKeyCode(pEvent)) {
            case AKEYCODE_DPAD_LEFT:
                mDirectionX = -ORTHOGONAL_MOVE;
                return true;
            case AKEYCODE_DPAD_RIGHT:
                mDirectionX = ORTHOGONAL_MOVE;
                return true;
            case AKEYCODE_DPAD_DOWN:
                mDirectionY = -ORTHOGONAL_MOVE;
                return true;
            case AKEYCODE_DPAD_UP:
                mDirectionY = ORTHOGONAL_MOVE;
                return true;
            }
        } else {
            switch (AKeyEvent_getKeyCode(pEvent)) {
            case AKEYCODE_DPAD_LEFT:
            case AKEYCODE_DPAD_RIGHT:
                mDirectionX = 0.0f;
                return true;
            case AKEYCODE_DPAD_DOWN:
            case AKEYCODE_DPAD_UP:
                mDirectionY = 0.0f;
                return true;
            }
        }
        return false;
    }
    ...
    ```

1.  在新方法 `onTrackballEvent()` 中处理轨迹球事件。使用 `AMotionEvent_getX()` 和 `AMotionEvent_getY()` 获取轨迹球的大小。由于一些轨迹球不提供渐变的幅度，因此使用普通常量来量化移动，并通过任意的触发阈值忽略可能出现的噪声：

    ```java
    ...
    bool InputManager::onTrackballEvent(AInputEvent* pEvent) {
        static const float ORTHOGONAL_MOVE = 1.0f;
        static const float DIAGONAL_MOVE   = 0.707f;
        static const float THRESHOLD       = (1/100.0f);

         if (AMotionEvent_getAction(pEvent) == AMOTION_EVENT_ACTION_MOVE) {
            float directionX = AMotionEvent_getX(pEvent, 0);
            float directionY = AMotionEvent_getY(pEvent, 0);
            float horizontal, vertical;

            if (directionX < -THRESHOLD) {
                if (directionY < -THRESHOLD) {
                    horizontal = -DIAGONAL_MOVE;
                    vertical   = DIAGONAL_MOVE;
                } else if (directionY > THRESHOLD) {
                    horizontal = -DIAGONAL_MOVE;
                    vertical   = -DIAGONAL_MOVE;
                } else {
                    horizontal = -ORTHOGONAL_MOVE;
                    vertical   = 0.0f;
                }
            } else if (directionX > THRESHOLD) {
                if (directionY < -THRESHOLD) {
                    horizontal = DIAGONAL_MOVE;
                    vertical   = DIAGONAL_MOVE;
                } else if (directionY > THRESHOLD) {
                    horizontal = DIAGONAL_MOVE;
                    vertical   = -DIAGONAL_MOVE;
                } else {
                    horizontal = ORTHOGONAL_MOVE;
                    vertical   = 0.0f;
                }
            } else if (directionY < -THRESHOLD) {
                horizontal = 0.0f;
                vertical   = ORTHOGONAL_MOVE;
            } else if (directionY > THRESHOLD) {
                horizontal = 0.0f;
                vertical   = -ORTHOGONAL_MOVE;
            }
    ...
    ```

1.  以这种方式使用轨迹球时，飞船会一直移动，直到出现“反方向移动”（例如，在向左移动时请求向右移动）或按下动作按钮（最后的 `else` 部分）：

    ```java
            ...
            // Ends movement if there is a counter movement.
            if ((horizontal < 0.0f) && (mDirectionX > 0.0f)) {
                mDirectionX = 0.0f;
            } else if ((horizontal > 0.0f) && (mDirectionX < 0.0f)) {
                mDirectionX = 0.0f;
            } else {
                mDirectionX = horizontal;
            }

            if ((vertical < 0.0f) && (mDirectionY > 0.0f)) {
                mDirectionY = 0.0f;
            } else if ((vertical > 0.0f) && (mDirectionY < 0.0f)) {
                mDirectionY = 0.0f;
            } else {
                mDirectionY = vertical;
            }
        } else {
            mDirectionX = 0.0f; mDirectionY = 0.0f;
        }
        return true;
    }
    ```

## *刚才发生了什么？*

我们扩展了输入系统以处理键盘、D-Pad 和轨迹球事件。D-Pad 可以被视为键盘的扩展并以相同的方式处理。实际上，D-Pad 和键盘事件使用相同的结构 (`AInputEvent`) 并通过相同的 API（以 `AKeyEvent` 为前缀）处理。

下表列出了主要的键事件方法：

| 方法 | 描述 |
| --- | --- |
| `AKeyEvent_getAction()` | 指示按钮是按下 (`AKEY_EVENT_ACTION_DOWN`) 还是释放 (`AKEY_EVENT_ACTION_UP`)。请注意，可以批量发出多个键动作 (`AKEY_EVENT_ACTION_MULTIPLE`)。 |
| `AKeyEvent_getKeyCode()` | 获取实际被按下的按钮（在 `android/keycodes.h` 中定义），例如，左按钮为 `AKEYCODE_DPAD_LEFT`。 |
| `AKeyEvent_getFlags()` | 键事件可以与一个或多个标志相关联，这些标志提供了关于事件的各种信息，例如 `AKEY_EVENT_LONG_PRESS`、`AKEY_EVENT_FLAG_SOFT_KEYBOARD` 表示事件源自模拟键盘。 |
| `AKeyEvent_getScanCode()` | 类似于键码，但这是一个原始键 ID，依赖于设备且不同设备之间可能不同。 |
| `AKeyEvent_getMetaState()` | 元状态是标志，表示是否同时按下了某些修饰键，如 Alt 或 Shift（例如 `AMETA_SHIFT_ON`、`AMETA_NONE` 等）。 |
| `AKeyEvent_getRepeatCount()` | 指示按钮事件发生了多少次，通常是在你按下按钮不放时。 |
| `AKeyEvent_getDownTime()` | 了解按钮被按下时间。 |

尽管一些轨迹球（尤其是光学的）表现得像 D-Pad，但轨迹球并不使用相同的 API。实际上，轨迹球是通过 `AMotionEvent` API（如触摸事件）来处理的。当然，一些为触摸事件提供的信息在轨迹球上并不总是可用。需要关注的最重要功能如下：

| `AMotionEvent_getAction()` | 了解事件是否表示移动动作（与按下动作相对）。 |
| --- | --- |
| `AMotionEvent_getX()``AMotionEvent_getY()` | 获取轨迹球移动。 |
| `AKeyEvent_getDownTime()` | 了解轨迹球是否被按下（如 D-Pad 动作按钮）。目前，大多数轨迹球使用全或无的压力来指示按下事件。 |

在处理轨迹球时，要记住的一个棘手点是，没有事件生成来指示轨迹球没有移动。此外，轨迹球事件是作为“爆发”生成的，这使得检测运动何时结束变得更加困难。处理这个问题的唯一方法（除了使用手动计时器并定期检查是否有足够长的时间没有事件发生）。

### 提示

不要期望所有手机上的外围设备行为完全相同。轨迹球就是一个很好的例子；它们可以像模拟垫一样指示方向，也可以像 D-Pad 一样指示直线路径（例如，光学轨迹球）。目前还没有办法从可用的 API 中区分设备特性。唯一的解决方案是在运行时校准设备或配置设备，或者保存一种设备数据库。

# 探测设备传感器

处理输入设备对于任何应用来说都很重要，但对于最智能的应用来说，探测传感器才是关键！在 Android 游戏应用中最常见的传感器就是加速度计。

如其名称所示，加速度计测量施加在设备上的线性加速度。当将设备向上、下、左或右移动时，加速度计会被激发，并在 3D 空间中指示一个加速度矢量。该矢量是相对于屏幕默认方向的。坐标系相对于设备的自然方向：

+   X 轴指向右侧

+   Y 轴指向上方

+   Z 轴从后向前指

如果设备旋转，轴会反转（例如，如果设备顺时针旋转 90 度，Y 轴将指向左侧）。

加速度计一个非常有趣的特点是它们经历恒定加速度：地球上的重力，大约是 9.8m/s²。例如，当设备平放在桌子上时，加速度矢量在 Z 轴上指示-9.8。当设备直立时，它在 Y 轴上指示相同的值。因此，假设设备位置固定，可以从重力加速度矢量推导出设备在空间中的两个轴的方向。要获取 3D 空间中设备的完全方向，还需要一个磁力计。

### 提示

请记住，加速度计处理的是线性加速度。它们可以检测设备不旋转时的平移和设备固定时的部分方向。然而，如果没有磁力计和/或陀螺仪，这两种运动是无法结合的。

因此，我们可以使用从加速度计推导出的设备方向来计算一个方向。现在让我们看看如何在 `DroidBlaster` 中应用这个过程。

### 注意

最终项目与本一起提供，名为 `DroidBlaster_Part15`。

# 行动时间——处理加速度计事件

让我们在 `DroidBlaster` 中处理加速度计事件：

1.  打开 `jni/InputHandler.hpp` 文件，并添加一个新的方法 `onAccelerometerEvent()`。包含官方传感器头文件 `android/sensor.h`：

    ```java
    #ifndef _PACKT_INPUTHANDLER_HPP_
    #define _PACKT_INPUTHANDLER_HPP_

    #include <android/input.h>
    #include <android/sensor.h>

    class InputHandler {
    public:
        virtual ~InputHandler() {};

        virtual bool onTouchEvent(AInputEvent* pEvent) = 0;
        virtual bool onKeyboardEvent(AInputEvent* pEvent) = 0;
        virtual bool onTrackballEvent(AInputEvent* pEvent) = 0;
        virtual bool onAccelerometerEvent(ASensorEvent* pEvent) = 0;
    };
    #endif
    ```

1.  在 `jni/EventLoop.hpp` 中创建新方法：

    +   `activateAccelerometer()`和`deactivateAccelerometer()`在活动开始和停止时启用/禁用加速度传感器。

    +   `processSensorEvent()`检索并分派传感器事件。

    +   回调`callback_input()`静态方法绑定到 Looper。

    同时，定义以下成员：

    +   `mSensorManager`，类型为`ASensorManager`，是与传感器交互的主要“对象”。

    +   `mSensorEventQueue`是`ASensorEventQueue`，这是 Sensor API 定义的结构，用于检索发生的事件。

    +   `mSensorPollSource`是 Native Glue 中定义的`android_poll_source`。这个结构描述了如何将原生线程 Looper 绑定到传感器回调。

    +   `mAccelerometer`，声明为`ASensor`结构，表示所使用的传感器：

        ```java
        #ifndef _PACKT_EVENTLOOP_HPP_
        #define _PACKT_EVENTLOOP_HPP_

        #include "ActivityHandler.hpp"
        #include "InputHandler.hpp"

        #include <android_native_app_glue.h>

        class EventLoop {
            ...
        private:
            void activate();
            void deactivate();
            void activateAccelerometer();
         void deactivateAccelerometer();

            void processAppEvent(int32_t pCommand);
            int32_t processInputEvent(AInputEvent* pEvent);
            void processSensorEvent();

            static void callback_appEvent(android_app* pApplication,
                    int32_t pCommand);
            static int32_t callback_input(android_app* pApplication,
                    AInputEvent* pEvent);
            static void callback_sensor(android_app* pApplication,
         android_poll_source* pSource);

            ...
            InputHandler& mInputHandler;

            ASensorManager* mSensorManager;
         ASensorEventQueue* mSensorEventQueue;
         android_poll_source mSensorPollSource;
         const ASensor* mAccelerometer;
        };
        #endif
        ```

1.  更新`jni/EventLoop.cpp`中的构造函数初始化列表：

    ```java
    #include "EventLoop.hpp"
    #include "Log.hpp"

    EventLoop::EventLoop(android_app* pApplication,
        ActivityHandler& pActivityHandler, InputHandler& pInputHandler):
            mApplication(pApplication),
            mActivityHandler(pActivityHandler),
            mEnabled(false), mQuit(false),
            mInputHandler(pInputHandler),
            mSensorPollSource(), mSensorManager(NULL),
            mSensorEventQueue(NULL), mAccelerometer(NULL) {
        mApplication->userData = this;
        mApplication->onAppCmd = callback_appEvent;
        mApplication->onInputEvent = callback_input;
    }
    ...
    ```

1.  创建传感器事件队列，通过它通知所有`sensor`事件。

    将其绑定到`callback_sensor()`。请注意，这里我们使用 Native App Glue 提供的`LOOPER_ID_USER`常量来附加用户定义的队列。

    接着，调用`activateAccelerometer()`来初始化加速度传感器：

    ```java
    ...
    void EventLoop::activate() {
        if ((!mEnabled) && (mApplication->window != NULL)) {
            mSensorPollSource.id = LOOPER_ID_USER;
     mSensorPollSource.app = mApplication;
     mSensorPollSource.process = callback_sensor;
     mSensorManager = ASensorManager_getInstance();
     if (mSensorManager != NULL) {
     mSensorEventQueue = ASensorManager_createEventQueue(
     mSensorManager, mApplication->looper,
     LOOPER_ID_USER, NULL, &mSensorPollSource);
     if (mSensorEventQueue == NULL) goto ERROR;
     }
     activateAccelerometer();

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
    ...
    ```

1.  当活动被禁用或结束时，禁用正在运行的加速度传感器，以免不必要的消耗电池。

    然后，销毁`sensor`事件队列：

    ```java
    ...
    void EventLoop::deactivate() {
        if (mEnabled) {
            deactivateAccelerometer();
     if (mSensorEventQueue != NULL) {
     ASensorManager_destroyEventQueue(mSensorManager,
     mSensorEventQueue);
     mSensorEventQueue = NULL;
     }
     mSensorManager = NULL;

            mActivityHandler.onDeactivate();
            mEnabled = false;
        }
    }
    ...
    ```

1.  当事件循环被轮询时，`callback_sensor()`会被触发。它将事件分派给`EventLoop`实例上的`processSensorEvent()`。我们只关心`ASENSOR_TYPE_ACCELEROMETER`事件：

    ```java
    ...
    void EventLoop::callback_sensor(android_app* pApplication,
        android_poll_source* pSource) {
        EventLoop& eventLoop = *(EventLoop*) pApplication->userData;
        eventLoop.processSensorEvent();
    }

    void EventLoop::processSensorEvent() {
        ASensorEvent event;
        if (!mEnabled) return;

        while (ASensorEventQueue_getEvents(mSensorEventQueue,
                &event, 1) > 0) {
            switch (event.type) {
            case ASENSOR_TYPE_ACCELEROMETER:
                mInputHandler.onAccelerometerEvent(&event);
                break;
            }
        }
    }
    ...
    ```

1.  在`activateAccelerometer()`中通过三个主要步骤激活传感器：

    +   使用`AsensorManager_getDefaultSensor()`获取特定类型的传感器。

    +   然后，使用`ASensorEventQueue_enableSensor()`启用它，以便传感器事件队列填充相关事件。

    +   使用`ASensorEventQueue_setEventRate()`设置所需的事件率。对于游戏，我们通常希望接近实时测量。可以使用`ASensor_getMinDelay()`查询最小延迟（将其设置得较低可能会导致失败）。

    显然，我们应该只在传感器事件队列准备好时执行此设置：

    ```java
    ...
    void EventLoop::activateAccelerometer() {
        mAccelerometer = ASensorManager_getDefaultSensor(
                mSensorManager, ASENSOR_TYPE_ACCELEROMETER);
        if (mAccelerometer != NULL) {
            if (ASensorEventQueue_enableSensor(
                    mSensorEventQueue, mAccelerometer) < 0) {
                Log::error("Could not enable accelerometer");
                return;
            }

            int32_t minDelay = ASensor_getMinDelay(mAccelerometer);
            if (ASensorEventQueue_setEventRate(mSensorEventQueue,
                    mAccelerometer, minDelay) < 0) {
                Log::error("Could not set accelerometer rate");
            }
        } else {
            Log::error("No accelerometer found");
        }
    }
    ...
    ```

1.  传感器停用更容易，只需调用`AsensorEventQueue_disableSensor()`方法即可：

    ```java
    ...
    void EventLoop::deactivateAccelerometer() {
        if (mAccelerometer != NULL) {
            if (ASensorEventQueue_disableSensor(mSensorEventQueue,
                    mAccelerometer) < 0) {
                Log::error("Error while deactivating sensor.");
            }
            mAccelerometer = NULL;
        }
    }
    ```

## *刚才发生了什么？*

我们创建了一个事件队列来监听传感器事件。事件被封装在`ASensorEvent`结构中，该结构在`android/sensor.h`中定义。这个结构提供了以下内容：

+   传感器事件来源，即哪个传感器产生了这个事件。

+   传感器事件发生的时间。

+   传感器输出值。这个值存储在一个联合结构中，即你可以使用其中一个内部结构（这里我们关心的是`acceleration`向量）。

    ```java
    typedef struct ASensorEvent {
        int32_t version;
        int32_t sensor;
        int32_t type;
        int32_t reserved0;
        int64_t timestamp;
        union {
            float           data[16];
            ASensorVector   vector;
            ASensorVector   acceleration;
            ASensorVector   magnetic;
            float           temperature;
            float           distance;
            float           light;
            float           pressure;
        };
        int32_t reserved1[4];
    } ASensorEvent;
    ```

对于任何 Android 传感器，都使用相同的`ASensorEvent`结构。在加速度传感器的情况下，我们获取一个带有三个坐标`x`、`y`、`z`的向量，每个轴一个：

```java
typedef struct ASensorVector {
    union {
        float v[3];
        struct {
            float x;
            float y;
            float z;
        };
        struct {
            float azimuth;
            float pitch;
            float roll;
        };
    };
    int8_t status;
    uint8_t reserved[3];
} ASensorVector;
```

在我们的示例中，加速度计设置为尽可能低的事件率，这可能在不同的设备之间有所变化。需要注意的是，传感器事件率对电池节省有直接影响！因此，使用对应用程序来说足够的事件率。`ASensor` API 提供了一些方法来查询可用的传感器及其功能，如`ASensor_getName()`、`ASensor_getVendor()`、`ASensor_getMinDelay()`等。

现在我们能够获取传感器事件，让我们用它们来计算飞船的方向。

# 行动时间——将 Android 设备转变为游戏手柄

让我们找到设备方向并正确确定方向。

1.  编写一个新文件`jni/Configuration.hpp`，帮助我们获取设备信息，特别是设备旋转（定义为`screen_rot`）。

    声明`findRotation()`，以借助 JNI 发现设备方向：

    ```java
    #ifndef _PACKT_CONFIGURATION_HPP_
    #define _PACKT_CONFIGURATION_HPP_

    #include "Types.hpp"

    #include <android_native_app_glue.h>
    #include <jni.h>

    typedef int32_t screen_rot;

    const screen_rot ROTATION_0   = 0;
    const screen_rot ROTATION_90  = 1;
    const screen_rot ROTATION_180 = 2;
    const screen_rot ROTATION_270 = 3;

    class Configuration {
    public:
        Configuration(android_app* pApplication);

        screen_rot getRotation() { return mRotation; };

    private:
        void findRotation(JNIEnv* pEnv);

        android_app* mApplication;
        screen_rot mRotation;
    };
    #endif
    ```

1.  在`jni/Configuration.cpp`中获取配置详情。

    首先，在构造函数中，使用`AConfiguration` API 转储配置属性，例如当前语言、国家、屏幕大小、屏幕方向。这些信息可能很有趣，但不足以正确分析加速度计事件：

    ```java
    #include "Configuration.hpp"
    #include "Log.hpp"

    #include <stdlib.h>

    Configuration::Configuration(android_app* pApplication) :
        mApplication(pApplication),
        mRotation(0) {
        AConfiguration* configuration = AConfiguration_new();
        if (configuration == NULL) return;

        int32_t result;
        char i18NBuffer[] = "__";
        static const char* orientation[] = {
            "Unknown", "Portrait", "Landscape", "Square"
        };
        static const char* screenSize[] = {
            "Unknown", "Small", "Normal", "Large", "X-Large"
        };
        static const char* screenLong[] = {
            "Unknown", "No", "Yes"
        };

        // Dumps current configuration.
        AConfiguration_fromAssetManager(configuration,
            mApplication->activity->assetManager);
        result = AConfiguration_getSdkVersion(configuration);
        Log::info("SDK Version : %d", result);
        AConfiguration_getLanguage(configuration, i18NBuffer);
        Log::info("Language    : %s", i18NBuffer);
        AConfiguration_getCountry(configuration, i18NBuffer);
        Log::info("Country     : %s", i18NBuffer);
        result = AConfiguration_getOrientation(configuration);
        Log::info("Orientation : %s (%d)", orientation[result], result);
        result = AConfiguration_getDensity(configuration);
        Log::info("Density     : %d dpi", result);
        result = AConfiguration_getScreenSize(configuration);
        Log::info("Screen Size : %s (%d)", screenSize[result], result);
        result = AConfiguration_getScreenLong(configuration);
        Log::info("Long Screen : %s (%d)", screenLong[result], result);
        AConfiguration_delete(configuration);
    ...
    ```

    然后，将当前本地线程附加到 Android VM。

    ### 提示

    如果你仔细阅读了第四章，*从本地代码调用 Java*，你就知道这一步是获取`JNIEnv`对象（特定于线程）的必要步骤。`JavaVM`本身可以从`android_app`结构中获取。

1.  之后，调用`findRotation()`以获取当前设备旋转。

    最后，我们可以将线程从 Dalvik 分离，因为我们不再使用 JNI。请记住，在结束应用程序之前，始终应该分离已附加的线程：

    ```java
    ...
        JavaVM* javaVM = mApplication->activity->vm;
        JavaVMAttachArgs javaVMAttachArgs;
        javaVMAttachArgs.version = JNI_VERSION_1_6;
        javaVMAttachArgs.name = "NativeThread";
        javaVMAttachArgs.group = NULL;
        JNIEnv* env;
        if (javaVM->AttachCurrentThread(&env,
                        &javaVMAttachArgs) != JNI_OK) {
            Log::error("JNI error while attaching the VM");
            return;
        }
        // Finds screen rotation and get-rid of JNI.
        findRotation(env);
        mApplication->activity->vm->DetachCurrentThread();
    }
    ...
    ```

1.  实现`findRotation()`，其基本通过 JNI 执行以下 Java 代码：

    ```java
    WindowManager mgr = (InputMethodManager)
    myActivity.getSystemService(Context.WINDOW_SERVICE);
    int rotation = mgr.getDefaultDisplay().getRotation();
    ```

    显然，这在 JNI 中编写会稍微复杂一些。

    +   首先，获取 JNI 类，然后是方法，最后是字段

    +   然后，执行 JNI 调用

    +   最后，释放分配的 JNI 引用

    以下代码故意简化，以避免额外的检查（即，每个方法调用的`FindClass()`和`GetMethodID()`返回值和异常检查）：

    ```java
    ...
    void Configuration::findRotation(JNIEnv* pEnv) {
        jobject WINDOW_SERVICE, windowManager, display;
        jclass ClassActivity, ClassContext;
        jclass ClassWindowManager, ClassDisplay;
        jmethodID MethodGetSystemService;
        jmethodID MethodGetDefaultDisplay;
        jmethodID MethodGetRotation;
        jfieldID FieldWINDOW_SERVICE;

        jobject activity = mApplication->activity->clazz;

        // Classes.
        ClassActivity = pEnv->GetObjectClass(activity);
        ClassContext = pEnv->FindClass("android/content/Context");
        ClassWindowManager = pEnv->FindClass(
            "android/view/WindowManager");
        ClassDisplay = pEnv->FindClass("android/view/Display");

        // Methods.
        MethodGetSystemService = pEnv->GetMethodID(ClassActivity,
            "getSystemService",
            "(Ljava/lang/String;)Ljava/lang/Object;");
        MethodGetDefaultDisplay = pEnv->GetMethodID(
            ClassWindowManager, "getDefaultDisplay",
            "()Landroid/view/Display;");
        MethodGetRotation = pEnv->GetMethodID(ClassDisplay,
            "getRotation", "()I");

        // Fields.
        FieldWINDOW_SERVICE = pEnv->GetStaticFieldID(
          ClassContext, "WINDOW_SERVICE", "Ljava/lang/String;");

        // Retrieves Context.WINDOW_SERVICE.
        WINDOW_SERVICE = pEnv->GetStaticObjectField(ClassContext,
            FieldWINDOW_SERVICE);
        // Runs getSystemService(WINDOW_SERVICE).
        windowManager = pEnv->CallObjectMethod(activity,
            MethodGetSystemService, WINDOW_SERVICE);
        // Runs getDefaultDisplay().getRotation().
        display = pEnv->CallObjectMethod(windowManager,
            MethodGetDefaultDisplay);
        mRotation = pEnv->CallIntMethod(display, MethodGetRotation);

        pEnv->DeleteLocalRef(ClassActivity);
        pEnv->DeleteLocalRef(ClassContext);
        pEnv->DeleteLocalRef(ClassWindowManager);
        pEnv->DeleteLocalRef(ClassDisplay);
    }
    ```

1.  在`jni/InputManager.hpp`中管理新的加速度计传感器。

    加速度计轴在`toScreenCoord()`中进行转换。

    这种转换意味着我们要跟踪设备旋转：

    ```java
    ...
    #include "Configuration.hpp"
    #include "GraphicsManager.hpp"
    #include "InputHandler.hpp"
    ...
    class InputManager : public InputHandler {
        ...
    protected:
        bool onTouchEvent(AInputEvent* pEvent);
        bool onKeyboardEvent(AInputEvent* pEvent);
        bool onTrackballEvent(AInputEvent* pEvent);
        bool onAccelerometerEvent(ASensorEvent* pEvent);
     void toScreenCoord(screen_rot pRotation,
     ASensorVector* pCanonical, ASensorVector* pScreen);

    private:
        ...
        float mScaleFactor;
        float mDirectionX, mDirectionY;
         // Reference point to evaluate touch distance.
         Location* mRefPoint;
        screen_rot mRotation;
    };
    #endif
    ```

1.  在`jni/InputManager.hpp`中，利用新的`Configuration`类读取当前屏幕旋转设置。由于`DroidBlaster`强制使用竖屏模式，我们可以一次性存储旋转：

    ```java
    ...
     InputManager::InputManager(android_app* pApplication,
             GraphicsManager& pGraphicsManager) :
            mApplication(pApplication), mGraphicsManager(pGraphicsManager),
            mDirectionX(0.0f), mDirectionY(0.0f),
            mRefPoint(NULL) {
        Configuration configuration(pApplication);
     mRotation = configuration.getRotation();
    }
    ...
    ```

1.  让我们从加速度计传感器值计算一个方向。

    首先，将加速度计值从标准坐标转换为屏幕坐标，以处理竖屏和横屏设备。

    接下来，从捕获到的加速度计值中计算一个方向。在以下代码中，`X`和`Z`轴分别表示滚转和俯仰。检查这两个轴，看设备是否处于中性位置（即`CENTER_X`和`CENTER_Z`）或者是在倾斜（`MIN_X`、`MIN_Z`、`MAX_X`和`MAX_Z`）。请注意，我们的需求需要将 Z 值取反：

    ```java
    ...
    bool InputManager::onAccelerometerEvent(ASensorEvent* pEvent) {
        static const float GRAVITY =  ASENSOR_STANDARD_GRAVITY / 2.0f;
        static const float MIN_X = -1.0f; static const float MAX_X = 1.0f;
        static const float MIN_Z =  0.0f; static const float MAX_Z = 2.0f;
        static const float CENTER_X = (MAX_X + MIN_X) / 2.0f;
        static const float CENTER_Z = (MAX_Z + MIN_Z) / 2.0f;

        // Converts from canonical to screen coordinates.
        ASensorVector vector;
        toScreenCoord(mRotation, &pEvent->vector, &vector);

        // Roll tilt.
        float rawHorizontal = pEvent->vector.x / GRAVITY;
        if (rawHorizontal > MAX_X) {
            rawHorizontal = MAX_X;
        } else if (rawHorizontal < MIN_X) {
            rawHorizontal = MIN_X;
        }
        mDirectionX = CENTER_X - rawHorizontal;

        // Pitch tilt. Final value needs to be inverted.
        float rawVertical = pEvent->vector.z / GRAVITY;
        if (rawVertical > MAX_Z) {
            rawVertical = MAX_Z;
        } else if (rawVertical < MIN_Z) {
            rawVertical = MIN_Z;
        }
        mDirectionY = rawVertical - CENTER_Z;
        return true;
    }
    ...
    ```

1.  在`toScreenCoord()`辅助函数中，根据屏幕旋转交换或反转加速度计轴，使得在使用`DroidBlaster`在纵向模式时，无论你使用哪种设备，`X`和`Z`轴都指向同一方向：

    ```java
    ...
    void InputManager::toScreenCoord(screen_rot pRotation,
        ASensorVector* pCanonical, ASensorVector* pScreen) {
        struct AxisSwap {
            int8_t negX; int8_t negY;
            int8_t xSrc; int8_t ySrc;
        };
        static const AxisSwap axisSwaps[] = {
             {  1, -1, 0, 1},  // ROTATION_0
             { -1, -1, 1, 0},  // ROTATION_90
             { -1,  1, 0, 1},  // ROTATION_180
             {  1,  1, 1, 0}}; // ROTATION_270
        const AxisSwap& swap = axisSwaps[pRotation];

        pScreen->v[0] = swap.negX * pCanonical->v[swap.xSrc];
        pScreen->v[1] = swap.negY * pCanonical->v[swap.ySrc];
        pScreen->v[2] = pCanonical->v[2];
    }
    ```

## *刚才发生了什么？*

加速度计现在是一个游戏手柄！Android 设备可以是自然的纵向（主要是智能手机和小型平板电脑）或横向（主要是平板电脑）。这对接收加速度计事件的应用程序有影响。这些类型的设备及其旋转方式不同，轴线的对齐方式也不相同。

实际上，屏幕可以以四种不同的方式定向：`0`、`90`、`180`和`270`度。0 度是设备的自然方向。加速度计 X 轴始终指向右侧，Y 轴指向上方，Z 轴指向前方。在手机上，Y 轴在纵向模式下指向上方，而在大多数平板电脑上，Y 轴在横向模式下指向上方。当设备以 90 度方向定位时，轴的方向显然会改变（X 轴指向上方，等等）。这种情况也可能发生在以纵向模式使用的平板电脑上（其中 0 度对应于横向模式）。

![发生了什么？](img/9645OS_08_03.jpg)

遗憾的是，使用原生 API 无法获取设备相对于屏幕自然方向的旋转。因此，我们需要依赖 JNI 来获取准确的设备旋转信息。然后，我们可以在`onAccelerometerEvent()`中像这样轻松地推导出方向向量。

## 关于传感器的更多内容

每个 Android 传感器都有一个唯一的标识符，在`android/sensor.h`中定义。这些标识符在所有 Android 设备上都是相同的：

+   `ASENSOR_TYPE_ACCELEROMETER`

+   `ASENSOR_TYPE_MAGNETIC_FIELD`

+   `ASENSOR_TYPE_GYRISCOPE`

+   `ASENSOR_TYPE_LIGHT`

+   `ASENSOR_TYPE_PROXIMITY`

可能存在其他额外的传感器并且可用，即使它们在`android/sensor.h`头文件中没有命名。在 Gingerbread 上，我们有与以下情况相同的情况：

+   重力传感器（标识符`9`）

+   线性加速度传感器（标识符`10`）

+   旋转矢量（标识符`11`）。

旋转矢量传感器是现在已弃用的方向矢量的继承者，在`增强现实`应用中至关重要。它提供了设备在 3D 空间中的方向。结合 GPS，它可以通过设备的视角定位任何物体。旋转传感器提供了一个数据矢量，通过`android.hardware.SensorManager`类（请参阅其源代码），可以将其转换为 OpenGL 视图矩阵。这样，你可以直接将设备方向物化为屏幕内容，将真实与虚拟生活联系起来。

# 总结

在本章中，我们介绍了多种从原生代码与 Android 交互的方法。更准确地说，我们学习了如何将输入队列附加到`Native App Glue`事件循环中。然后，我们处理了触摸事件以及来自键盘、D-Pad 的动作事件和轨迹球的运动事件。最后，我们将加速度计转换成了游戏手柄。

由于 Android 的碎片化，预计输入设备的行为会有所不同，需要准备好调整代码。在应用结构、图形、声音、输入和传感器方面，我们已经深入了解了 Android NDK 的能力。然而，重新发明轮子并不是解决方案！

在下一章中，我们将通过将现有的 C/C++库移植到 Android，释放 NDK 的真正力量。
