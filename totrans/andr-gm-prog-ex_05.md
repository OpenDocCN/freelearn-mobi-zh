# 第五章. 平台游戏 - 升级游戏引擎

欢迎来到这本书的第二个项目。在这里，我们将构建一个真正困难的复古平台游戏。它不是难以构建，而是当你玩它时难以击败。在项目结束时，我们还将讨论如何使游戏玩法稍微不那么严苛，如果你希望的话。

本章将完全聚焦于我们的游戏引擎，本质上将导致 Tappy Defender 代码的升级版本。

首先，我们将讨论我们希望通过这个游戏实现的目标：背景故事、游戏机制和规则。

然后，我们将快速创建一个活动，实例化一个将完成所有工作的视图。

之后，我们将充实`PlatformView`类的基本结构，它将有一些微妙的但重要的区别于我们的`TDView`类。最值得注意的是，`PlatformView`将有一个简单但有效的方式来管理我们游戏所有事件的时间。

然后，我们将开始迭代构建我们的`GameObject`类，游戏世界中的几乎每一个实体都将由此派生。

接下来，我们将讨论视口的概念，玩家通过这个视口来观看游戏世界。我们不再将游戏对象设计为在屏幕分辨率层面操作，而是存在于一个拥有自身*x*和*y*坐标的世界中，我们可以将这些坐标视为虚拟米。在*z*轴上也有一个简单的深度系统。这将由我们新的`Viewport`类来处理。

在此之后，我们将研究如何设计和布局游戏内容。这是通过一个用作关卡设计师的类完成的，可以非编程地使用它来规划跳跃、敌人、奖励和目标，这些构成了一个关卡的布局。

为了管理关卡设计并将它们加载到我们的游戏引擎中，我们将需要另一个类。我们将它称为`LevelManager`。

在本章的最后，我们将查看`PlatformView`类中的增强型`update`和`draw`方法，这样我们就可以实际运行我们的新游戏，并在屏幕上看到首次输出。

有这么多事情要做，我们最好开始吧。

# 游戏

我们将要构建的游戏基于一些 80 年代残酷难度的平台游戏，如 Bounty Bob Strikes Back 和 Impossible Mission 的游戏玩法。这些游戏以难以跳跃和同时需要极其精确的时机控制著称，同时给玩家一个不宽恕的生命/机会数量。这种游戏风格很适合我们，因为我们可以实际上在四个章节内构建一个多级别的可玩游戏。

类的设计将使你能够轻松添加自己的额外功能、游戏对象，或者如果你愿意，也可以稍微降低游戏的难度。

## 背景故事

我们的英雄鲍勃刚从地球中心摧毁一个邪恶科学家的秘密任务中回来，发现他正处于地下深处。更糟的是，尽管他已经击败了邪恶科学家，但似乎来不及拯救这个星球免受他释放的强大守卫和致命的飞行机器人无人机的侵袭。

鲍勃必须从深地下的火焰洞穴出发，穿过重兵把守的城市和山区森林，他希望在那里过上自由的生活，摆脱接管这个星球的可怕新秩序。

在这四个关卡中，他必须避开守卫，摧毁无人机，收集大量金钱，并升级他最初弱小的机枪。

## 游戏机制

游戏将围绕执行精确的跳跃，规划通过关卡的最佳路径以收集战利品并逃脱。鲍勃将能够小心翼翼地站在边缘，脚只有几个像素悬空，以完成看似不可能的跳跃。鲍勃将能够控制跳跃时的距离，这意味着有时他需要确保自己不会跳过头。

鲍勃在尝试通过重兵把守的区域逃脱前，需要收集机枪升级。

鲍勃只有三条生命，但在他的旅程中可能会找到更多。

## 游戏规则

当鲍勃被无人机/守卫捕获、触碰到火焰，或者跌出游戏世界而失去生命时，他将在当前关卡的起点重新出现。无人机可以飞行，并且一旦鲍勃进入视线就会锁定他。鲍勃需要确保他有足够的火力来对付无人机。守卫将在关卡预定区域巡逻，但他们很强大，只能被鲍勃的机枪击退。通常，鲍勃需要执行一个精确计时跳跃以绕过守卫。

环境同样会非常艰难。鲍勃需要完全掌握每个关卡，因为一次错误的跳跃就会让他直接回到起点，落入敌人手中，甚至直接遭遇火葬。

# 升级游戏引擎

所有的守卫、无人机、火焰、收藏品、枪支的讨论，以及暗示的更大游戏世界，表明我们需要管理一个更为复杂的系统。我们的游戏引擎的目标之一就是让这种复杂性易于管理。另一个目标是将关卡设计从编码中分离出来。当我们的游戏完成时，你将能够轻松设计出最邪恶但也最有成就感的关卡，在不同的环境中无需触碰代码就能完成设计。

## 平台活动

首先，我们从`Activity`类开始，这是进入我们游戏的入口点。这里没有太多新内容，让我们快速构建它。创建一个新项目，在**应用名称**字段中输入`C5 平台游戏`。选择**手机和平板**，然后在提示时选择**空白活动**。在**活动名称**字段中，输入`PlatformActivity`。

### 提示

显然，您不必遵循我的确切命名选择，但请记得在代码中进行一些小修改，以反映您自己的命名选择。

您可以从`layout`文件夹中删除`activity_platform.xml`。您还可以删除`PlatformActivity.java`文件中的所有代码。只保留包声明。现在，我们有一个完全空白的画布，准备开始编码。以下是到目前为止我们的项目的全部内容：

```java
package com.gamecodeschool.c5platformgame;
```

让我们开始构建我们的引擎。就像在我们的 Tappy Defender 项目中一样，我们将构建一个类来处理游戏视图方面。或许不足为奇，我们将这个类称为`PlatformView`。因此，我们的`PlatformActivity`类需要实例化一个`PlatformView`对象，并将其设置为应用程序的主要视图，就像在之前的项目中一样。

我们将对引擎进行一些重大升级，但主要是在视图层面进行。在接下来要看的`PlatformActivity`类的代码中，我们与上一个项目所做的类似。首先，在重写的`onCreate`方法中声明`PlatformView`对象，并将其设置为主要的视图；但在这样做之前，我们还需要捕获并传入设备屏幕的分辨率。

我们通过使用`Display`类，链式调用`getWindowManager()`和`getDefaultDisplay()`方法来获取我们游戏将要运行的物理显示硬件的属性。然后，我们创建一个名为`resolution`的`Point`类型的对象，并通过调用`display.getSize(size)`将显示的分辨率存储到我们的`Point`对象中。

这会将屏幕的水平像素数和垂直像素数分别存储在`size.x`和`size.y`中。然后我们可以继续通过调用其构造函数并传入`size.x`和`size.y`中存储的值来实例化一个新的`PlatformView`对象。与之前一样，我们还需要传入应用程序的`Context`对象（`this`），正如在之前的项目中，我们会发现它有很多用途。

我们可以通过调用`setContentView()`方法，将`platformView`设置为视图。如前所述，我们重写`Activity`类的生命周期方法`onPause()`和`onResume()`，让它们调用我们即将编写的`PlatformView`类中的相应方法。这两个方法可以启动和停止我们的`Thread`类。

下面是我们刚刚讨论的`PlatformActivity`类的完整代码，没有新的重要方面。将代码输入或复制粘贴到您的项目中。本章的代码可以在 Packt Publishing 网站的书籍页面下载捆绑包中找到。本章的所有代码和资源都可以在`Chapter5`文件夹中找到。这个文件叫做`PlatformActivity.java`。

### 提示

当提示导入所有新类时，请记得导入，或者当因缺少类而出现错误时，将光标悬停在错误上，按*Alt* | *Enter*键盘组合进行导入。

```java
import android.app.Activity;
import android.graphics.Point;
import android.os.Bundle;
import android.view.Display;

public class PlatformActivity extends Activity {

    // Our object to handle the View
    private PlatformView platformView;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        // Get a Display object to access screen details
        Display display = getWindowManager().getDefaultDisplay();

        // Load the resolution into a Point object
        Point resolution = new Point();
        display.getSize(resolution);

        // And finally set the view for our game
        // Also passing in the screen resolution
        platformView = new PlatformView
        (this, resolution.x, resolution.y);

        // Make our platformView the view for the Activity
        setContentView(platformView);

    }

    // If the Activity is paused make sure to pause our thread
    @Override
    protected void onPause() {
        super.onPause();
        platformView.pause();
    }

    // If the Activity is resumed make sure to resume our thread
    @Override
    protected void onResume() {
        super.onResume();
        platformView.resume();
    }
}
```

### 注意

显然，在我们创建`PlatformView`类之前，我们的`PlatformActivity`类代码中将会出现错误。

## 将布局锁定为横屏

正如我们在上一个项目中做的那样，我们将确保游戏只在横屏模式下运行。我们将使我们的`AndroidManifest.xml`文件强制我们的`PlatformActivity`类以全屏运行，并且我们还将将其锁定为横屏布局。让我们进行以下更改：

1.  现在打开`manifests`文件夹，双击`AndroidManifest.xml`文件，在代码编辑器中打开它。

1.  在`AndroidManifest.xml`文件中，找到以下代码行：

    ```java
    android:name=".PlatformActivity"
    ```

1.  在它下面，输入或复制粘贴以下两行代码，使`PlatformActivity`全屏运行，并将其锁定为横屏方向。

    ```java
    android:theme="@android:style/Theme.NoTitleBar.Fullscreen"
    android:screenOrientation="landscape"
    ```

现在，我们可以进入游戏的核心部分，看看我们如何实现我们讨论的所有这些改进。

## PlatformView 类

到完成时，这个类将依赖于许多其他类。我不想逐一介绍每个类，因为这样会很难跟上，而且哪些代码实现了哪个功能也会变得混乱。相反，我们将根据需要逐个查看和编写每个功能，并多次回顾许多类以添加更多功能。这将使代码每一部分的特定目的保持焦点。

说到这里，我们已经非常注意，尽管我们会多次回顾这些类，但我们不会不断地删除代码，而只是在原有代码中增加内容。当我们增加代码时，将在适当的上下文中展示代码，并将新部分在现有代码中突出显示。

至于类的结构，它们被设计为尽可能最小，同时也不会限制你轻松添加功能和扩展代码的潜力。

这不是关于游戏引擎设计的课程，而是更多地学习如何实现和压缩四个章节中的不同功能，而不会使代码变得难以管理。

如果你计划构建非常大的游戏，尤其是在团队中工作时，那么更健壮的设计将是必要的。这种更健壮的设计也将意味着大量的额外类、接口、包等等。

### 提示

如果这类讨论吸引了你，我强烈推荐你阅读 Mario Zechner 所著的《Beginning Android Games》，由 APRESS 出版。Mario 是跨平台游戏库 LibGDX 的创始人/创造者，他的书详细介绍了构建高度可扩展和可重用游戏代码库所需的设计模式。这本书详细的设计细节的唯一缺点是，它需要大约 600 页来构建一个简单的复古贪吃蛇游戏。

首先，让我们创建一个类。在 Android Studio 项目浏览器中右键点击包名，选择**New** | **Java Class**。将新类命名为`PlatformView`。删除类中自动生成的代码，因为我们将很快添加自己的代码。

在整个项目过程中，我们将会继续向这个类添加代码。本章中添加到类中的完整代码可以在下载包中的`Chapter5/PlatformView.java`找到。

我们需要一个能够管理我们关卡的类。让我们称它为`LevelManager`。

我们还需要一个类来保存我们关卡的数据，这样每次我们创建一个新/不同的关卡设计时，都可以扩展它。让我们将父类称为`LevelData`，而 Bob 逃脱的第一个真实关卡称为`LevelCave`。

此外，由于这个游戏将有许多敌人、道具和地形类型，我们需要一个更清洁的系统来管理它们。我们需要一个相当通用的`GameObject`类，所有不同的游戏对象都可以继承它。这样，我们在`update`和`draw`方法中可以更容易地管理它们。

同样，由于必要性，我们将构建一个稍微复杂一些的方法来检测玩家的输入。我们将创建一个`InputController`类，将所有代码从`PlatformView`委托给它。但是，我们将在下一章中完全展开我们的`Player`对象来表示玩家之后，才会了解这个类的细节。

我们可以快速编写基本的`PlatformView`类，其代码与第一个项目非常相似，但有几个值得注意的区别，我们将在后面讨论。

### `PlatformView`的基本结构

这里是必要的导入和我们开始需要的成员变量。随着项目的进行，我们将会增加这些内容。

请注意，我们还声明了三种新的对象类型，`lm`是我们的`LevelManager`类，`vp`是我们的`Viewport`类，以及`ic`，它是我们的`InputController`类。我们将在本章中开始处理其中一些内容。当然，在我们实现它们各自的类之前，这些声明将显示错误。

```java
import android.content.Context;
import android.graphics.Canvas;
import android.graphics.Color;
import android.graphics.Paint;
import android.graphics.Rect;
import android.util.Log;
import android.view.SurfaceHolder;
import android.view.SurfaceView;

public class PlatformView extends SurfaceView 
  implements Runnable {

  private boolean debugging = true;
  private volatile boolean running;
  private Thread gameThread = null;

  // For drawing
  private Paint paint;
  // Canvas could initially be local.
  // But later we will use it outside of draw()
  private Canvas canvas;
  private SurfaceHolder ourHolder;

  Context context;
  long startFrameTime;
  long timeThisFrame;
  long fps;

   // Our new engine classes
   private LevelManager lm;
   private Viewport vp;
   InputController ic;
```

在这里，我们有我们的`PlatformView`构造函数。在这个阶段，它没有做任何新的操作，实际上，它的代码比我们的`TDView`构造函数还要少，但它很快就会得到增强。现在，请输入如下代码：

```java
PlatformView(Context context, int screenWidth, 
    int screenHeight) {

    super(context);
    this.context = context;

    // Initialize our drawing objects
    ourHolder = getHolder();
    paint = new Paint();
}
```

这是我们的线程的`run`方法。注意，在调用`update()`之前，我们获取当前时间（毫秒）并将其放入`startFrameTime`长整型变量中。然后在`draw()`完成之后，我们再次调用以获取系统时间，并测量自帧开始以来已经过去了多少毫秒。然后我们执行计算`fps = 1000 / thisFrameTime`，这给了我们上一个帧中游戏运行的帧数。这个值存储在`fps`变量中。随着游戏的进行，我们将到处使用这个值。编写我们刚刚讨论的`run`方法，如下所示：

```java
@Override
public void run() {

  while (running) {
       startFrameTime = System.currentTimeMillis();

       update();
       draw();

      // Calculate the fps this frame
      // We can then use the result to
      // time animations and movement.
      timeThisFrame = System.currentTimeMillis() - startFrameTime;
            if (timeThisFrame >= 1) {
                fps = 1000 / timeThisFrame;
            }
     }
}
```

在本章后面，我们将看到如何管理多种对象类型的额外复杂性，并在必要时更新它们。现在，只需向`PlatformView`类添加一个空的`update`方法，如下所示：

```java
private void update() {
  // Our new update() code will go here
}
```

在这里，我们看到我们熟悉的`draw`方法的部分。在本章后面，我们将看到一些新代码。现在，添加`draw`方法的基本部分，如下所示，这部分将保持不变：

```java
private void draw() {

     if (ourHolder.getSurface().isValid()){
      //First we lock the area of memory we will be drawing to
      canvas = ourHolder.lockCanvas();

      // Rub out the last frame with arbitrary color
      paint.setColor(Color.argb(255, 0, 0, 255));
      canvas.drawColor(Color.argb(255, 0, 0, 255));

      // New drawing code will go here

      // Unlock and draw the scene
      ourHolder.unlockCanvasAndPost(canvas);
  }
}
```

视图第一阶段组合的最后部分是`pause`和`resume`方法，这些方法是由操作系统调用相应的 Activity 生命周期方法时由`PlatformActivity`调用的。它们与上一个项目中的方法没有变化，但为了完整性和便于跟踪，这里再次列出。将这些方法添加到`PlatformView`类中：

```java
// Clean up our thread if the game is interrupted    
public void pause() {
  running = false;
   try {
       gameThread.join();
   } catch (InterruptedException e) {
       Log.e("error", "failed to pause thread");
   }
}

// Make a new thread and start it
// Execution moves to our run method
public void resume() {
   running = true;
   gameThread = new Thread(this);
   gameThread.start();

}

}// End of PlatformView
```

现在，我们已经完成了视图的基本大纲编码并准备就绪。让我们首先看看`GameObject`类。

### `GameObject`类

我们知道我们需要一个父类来保存我们游戏对象的大部分内容，因为我们想要改进上一个项目中代码的灵活性和重复性。从上一个项目我们知道，它需要许多属性和方法。

首先，我们需要一个简单的类来表示所有未来`GameObject`类的世界位置。这个类将在*x*和*y*轴上保存一个详细的位置。请注意，这些与我们的游戏将运行的设备上的像素坐标完全独立。我们可以将*z*坐标视为图层编号。数字越小，越先绘制。因此，创建一个新的 Java 类，将其命名为`Vector2Point5D`，并输入以下代码：

```java
public class Vector2Point5D {

    float x;
    float y;
    int z;
}
```

现在，让我们看看并编码`GameObject`类的基本工作大纲，然后在项目过程中，我们可以回过头来添加额外的功能。创建一个新的 Java 类，将其命名为`GameObject`。让我们看看我们需要开始编写使这个类有用的代码。首先，我们导入所需的类。

```java
import android.content.Context;
import android.graphics.Bitmap;
import android.graphics.BitmapFactory;
```

当我们编写`GameObject`本身时，请注意该类没有提供构造函数，因为这将根据我们实现的特定`GameObject`而有所不同。

你在代码中注意到的第一个变量是`worldLocation`，正如你所预期的，它是`Vector2Point5D`类型的。然后我们有两个 float 成员，将保存`GameObject`类的宽度和高度。接下来是布尔变量`active`和`visible`，它们可能用于标记对象在活动、可见或其它状态时的标签。我们将在本章后面看到这样做的好处。

我们还需要知道任何给定的对象有多少内部动画帧。默认值将是`1`，因此`animFrameCount`相应地初始化。

然后，我们有一个名为`type`的`char`类。这个`type`变量将确切地确定任何特定的`GameObject`可能是什么。它将被广泛使用。目前最后一个成员变量是`bitmapName`。我们将看到，知道代表我们每个单独对象外观的图形的名称将非常有用。添加我们刚刚讨论的成员变量：

```java
public abstract class GameObject {

    private Vector2Point5D worldLocation;
    private float width;
    private float height;

    private boolean active = true;
    private boolean visible = true;
    private int animFrameCount = 1;
    private char type;

    private String bitmapName;

```

现在，我们可以看看`GameObject`功能的第一部分。我们有一个抽象方法`update()`。我们的计划是所有对象都需要更新自身。在四章内容中，这显得有些过于雄心勃勃，我们的一些对象（主要是平台和场景）将只提供一个空的`update()`实现。但是，这并不妨碍你让场景比我们现在有时间处理的更具互动性，或者在我们了解事物如何运作后，让平台更具动态性和冒险性。添加以下抽象`update`方法：

```java
public abstract void update(long fps, float gravity);
```

我们处理管理我们图形的方法。我们有一个获取器来检索`bitmapName`。然后，我们有一个`prepareBitmap()`方法，它使用字符串`bitmapName`从`.png`图像文件制作一个 Android 资源 ID。这个文件必须存在于项目的`drawable`文件夹中。就像我们之前看到的那样创建位图。

现在，我们的`prepareBitmap`方法做了些新的事情。它使用`createScaledBitmap`方法来改变我们刚刚创建的位图的大小。它不仅使用我们之前讨论的`animFrameCount`，还使用方法的参数`pixelsPerMetre`变量。

想法是，每个设备都有一个适合该设备的`pixelsPerMetre`值，这将帮助我们跨不同分辨率的设备创建一个相同的游戏视图。当我们讨论`Viewport`类时，我们将确切地了解我们从哪里获取这个`pixelsPerMetre`值。在`GameObject`类中输入以下方法：

```java
public String getBitmapName() {
        return bitmapName;
}

public Bitmap prepareBitmap(Context context, 
    String bitmapName, 
    int pixelsPerMetre) {

   // Make a resource id from the bitmapName
   int resID = context.getResources().
        getIdentifier(bitmapName,
        "drawable", context.getPackageName());

    // Create the bitmap
    Bitmap bitmap = BitmapFactory.
        decodeResource(context.getResources(),
        resID);

    // Scale the bitmap based on the number of pixels per metre
    // Multiply by the number of frames in the image
    // Default 1 frame
    bitmap = Bitmap.createScaledBitmap(bitmap,
                (int) (width * animFrameCount * pixelsPerMetre),
                (int) (height * pixelsPerMetre),
                false);

    return bitmap;
}
```

我们还希望能够知道每个`GameObject`在世界的哪个位置，当然，也要设置它在世界的哪个位置。以下是一个获取器和设置器，它们正好实现了这个功能。

```java
    public Vector2Point5D getWorldLocation() {
        return worldLocation;
    }

    public void setWorldLocation(float x, float y, int z) {
        this.worldLocation = new Vector2Point5D();
        this.worldLocation.x = x;
        this.worldLocation.y = y;
        this.worldLocation.z = z;
    }
```

我们希望能够获取和设置我们之前已经讨论过的许多成员变量。这些获取器和设置器将实现这一功能。

```java
    public void setBitmapName(String bitmapName){
        this.bitmapName = bitmapName;
    }

    public float getWidth() {
        return width;
    }

    public void setWidth(float width) {
        this.width = width;
    }

    public float getHeight() {
        return height;
    }

    public void setHeight(float height) {
        this.height = height;
    }
```

此外，我们还将希望能够检查和更改我们活动变量和可见变量的状态。

```java
    public boolean isActive() {
        return active;
    }

    public boolean isVisible() {
        return visible;
    }

    public void setVisible(boolean visible) {
        this.visible = visible;
    }
```

设置和获取每个`GameObject`的`type`。

```java
    public char getType() {
        return type;
    }

    public void setType(char type) {
        this.type = type;
    }

}// End of GameObject
```

现在，我们将从`GameObject`创建我们的第一个子类。在 Android Studio 资源管理器中右键点击包名，并创建一个名为`Grass`的类。这将是我们第一个基本的地砖类型，玩家可以在上面走动。

这段简单的代码使用构造函数来初始化高度、宽度、类型以及游戏世界中的位置。请注意，所有这些信息都是作为参数传递给构造函数的。`Grass`类唯一“知道”的，以及与其他简单的`GameObject`子类区别开来的少数几件事之一，就是`bitmapName`的值，在这个情况下是`turf`。

如先前讨论的，我们还提供了一个空的`update`方法的实现：

```java
public class Grass extends GameObject {

    Grass(float worldStartX, float worldStartY, char type) {
        final float HEIGHT = 1;
        final float WIDTH = 1;

        setHeight(HEIGHT); // 1 metre tall
        setWidth(WIDTH); // 1 metre wide

        setType(type);

        // Choose a Bitmap
        setBitmapName("turf");

        // Where does the tile start
        // X and y locations from constructor parameters
        setWorldLocation(worldStartX, worldStartY, 0);
    }

    public void update(long fps, float gravity) {}
}
```

现在，将下载包中`Chapter5/drawable`文件夹里的`turf.png`图形添加到 Android Studio 的`drawable`文件夹中。

最后，我们将对我们的`Player`类进行一个最基础的实现，该类也将扩展`GameObject`。我们不会在这个类中放置任何功能，只需一个*x*和*y*的世界位置。这样，我们接下来要实现的`Viewport`类就知道要聚焦在哪里了。

这是代表我们的英雄 Bob 的`Player`类。在这个阶段，这个类和`Grass`类一样简单直接，几乎与`Grass`类相同。随着我们进展，这将会有实质性的变化和发展。注意，我们将类型设置为`p`。

```java
import android.content.Context;

public class Player extends GameObject {

    Player(Context context, float worldStartX, 
        float worldStartY, int pixelsPerMetre) {

        final float HEIGHT = 2;
        final float WIDTH = 1;

        setHeight(HEIGHT); // 2 metre tall
        setWidth(WIDTH); // 1 metre wide

        setType('p');

        // Choose a Bitmap
        // This is a sprite sheet with multiple frames
        // of animation. So it will look silly until we animate it
        // In chapter 6.

        setBitmapName("player");

        // X and y locations from constructor parameters

        setWorldLocation(worldStartX, worldStartY, 0);

    }

    public void update(long fps, float gravity) {

    }
}
```

将下载包中`drawable`文件夹里的`player.png`图形添加到 Android Studio 的`drawable`文件夹中。这个图形是一个多帧的精灵表，所以在第六章*平台游戏 – Bob, Beeps, 和 Bumps*中进行动画处理之前，它不会很好地显示，但现在它可以作为一个占位符。

正如我们接下来将看到的，玩家看到的游戏世界的视图，将聚焦于 Bob，这应该是在你意料之中的。

### 通过视口看到的视图。

可以将视口视为跟随我们游戏动作的电影摄像机。它定义了要向玩家展示的游戏世界区域。通常，它会以 Bob 为中心。

它还通过确定哪些物体在玩家的视野内外，使我们的绘图方法更加高效。如果在一特定时刻它们并不相关，那么绘制或处理一堆敌人是毫无意义的。

通过实现第一阶段检测，即从需要检查碰撞的对象列表中移除屏幕外的对象，这将显著加快碰撞检测等任务的速度，而且这样做出奇地简单。

此外，我们的`Viewport`类将负责将游戏世界的坐标转换为屏幕上绘制的适当像素坐标。我们还将了解这个类是如何计算`GameObject`类在`prepareBitmap`方法中使用的`pixelsPerMetre`值的。

`Viewport`类确实是一个功能全面的东西。那么，让我们开始编程吧。

首先，我们将声明一大堆有用的变量。我们还有一个 `Vector2Point5D`，它将用于表示当前视口中焦点的世界上的任意点。然后，我们分别为 `pixelsPerMetreX` 和 `pixelsPerMetreY` 分配了整数值。

### 注意

实际上，在这个实现中，`pixelsPerMetrX` 和 `pixelsPerMetreY` 之间没有区别。但是，`Viewport` 类可以升级，以考虑基于屏幕尺寸而不是仅分辨率的不同设备宽高比。在这个实现中我们没有这样做。

接下来，我们简单地在两个轴上都有屏幕的分辨率：`screenXResolution` 和 `screenYResolution`。然后我们有 `screenCentreX` 和 `screenCentreY`，它们基本上是前两个变量除以二以找到中间位置。

在我们声明的变量列表中，我们有 `metresToShowX` 和 `metresToShowY`，它们将是我们将压缩到视口中的米数。改变这些值将显示屏幕上更多或更少的游戏世界。

在这一点上，我们将声明的最后一个成员是 `int numClipped`。我们将使用它输出调试文本，以查看 `Viewport` 类在提高绘图、更新和多阶段碰撞检测的效率方面有何影响。

创建一个名为 `Viewport` 的新类，并声明我们刚刚讨论过的变量：

```java
import android.graphics.Rect;

public class Viewport {
    private Vector2Point5D currentViewportWorldCentre;
    private Rect convertedRect;
    private int pixelsPerMetreX;
    private int pixelsPerMetreY;
    private int screenXResolution;
    private int screenYResolution;
    private int screenCentreX;
    private int screenCentreY;
    private int metresToShowX;
    private int metresToShowY;
    private int numClipped;
```

现在，让我们看看构造函数。构造函数只需要知道屏幕的分辨率。这是通过参数 *x* 和 *y* 获取的，当然，我们分别将其分配给 `screenXResolution` 和 `screenYResolution`。

然后，如前所述，我们将这两个变量除以二，并将结果分别分配给 `screenCentreX` 和 `screenCentreY`。

`pixelsPerMetreX` 和 `pixelsPerMetreY` 是通过分别除以 32 和 18 来计算的，因此一个分辨率为 840 x 400 像素的设备将会有每米*x/y*的像素数为 32/22。现在，我们有变量表示当前设备上表示游戏世界一米的屏幕像素数量。在代码中我们会多次看到，这将非常有用。

我们实际上会绘制一个比这稍宽的区域，以确保屏幕边缘不会有难看的缝隙/线条，并将 34 分配给 `metresToShowX`，20 分配给 `metresToShowY`。现在，我们有变量表示我们每一帧将绘制多少游戏世界。

### 提示

一旦有了屏幕输出，你可以通过调整这些值来为玩家创建放大或缩小的体验。

在构造函数即将结束时，我们创建了一个名为 `convertedRect` 的新 `Rect` 对象，我们很快就会看到它的实际应用。我们在 `currentViewportWorldCentre` 上调用 `new()` 方法，所以它很快就能投入使用。

```java
 Viewport(int x, int y){

        screenXResolution = x;
        screenYResolution = y;

        screenCentreX = screenXResolution / 2;
        screenCentreY = screenYResolution / 2;

        pixelsPerMetreX = screenXResolution / 32;
        pixelsPerMetreY = screenYResolution / 18;

        metresToShowX = 34;
        metresToShowY = 20;

        convertedRect = new Rect();
        currentViewportWorldCentre = new Vector2Point5D();

}
```

### 注意

如果这个项目中的某些截图看起来与您得到的结果略有不同，那是因为一些图片是使用不同的视口设置来突出游戏世界的不同方面。

我们为`Viewport`类编写的第一个方法是`setWorldCentre()`。它接收一个*x*和*y*参数，并立即分配给`currentWorldCentre`。我们需要这个方法，因为玩家当然会在世界中移动，我们需要让`Viewport`类知道 Bob 的位置。同样，正如我们将在第八章，*组合在一起*中看到的，我们也会有不想让 Bob 成为关注焦点的情况。

```java
void setWorldCentre(float x, float y){
  currentViewportWorldCentre.x  = x;
  currentViewportWorldCentre.y  = y;
}
```

现在，一些简单的获取器和设置器将在我们进行时非常有用。

```java
public int getScreenWidth(){
  return  screenXResolution;
}

public int getScreenHeight(){
  return  screenYResolution;
}

public int getPixelsPerMetreX(){
  return  pixelsPerMetreX;
}
```

我们通过`worldToScreen()`方法实现了`Viewport`类的主要功能之一。顾名思义，这个方法是用来将当前可见视口中的所有对象的位置从世界坐标转换为可以实际绘制在屏幕上的像素坐标。它返回我们之前准备好的`rectToDraw`对象作为结果。

`worldToScreen()`方法就是这样工作的。它接收一个对象的*x*和*y*世界位置以及该对象的宽度和高度。利用这些值，分别从当前屏幕的世界视口中心（*x*或*y*）减去对象的世界坐标乘以每米的像素数。然后，对于对象的左和上坐标，从像素屏幕中心值中减去结果，对于下和右坐标，则加上。

这些值随后被包装进`convertedRect`的左、上、右和下值中，并返回给`PlatformView`的`draw`方法。将`worldToScreen`方法添加到`Viewport`类中：

```java

public Rect worldToScreen(
  float objectX, 
  float objectY, 
  float objectWidth, 
  float objectHeight){

   int left = (int) (screenCentreX -               
    ((currentViewportWorldCentre.x - objectX) 
    * pixelsPerMetreX));

    int top =  (int) (screenCentreY -         
    ((currentViewportWorldCentre.y - objectY) 
    * pixelsPerMetreY));

   int right = (int) (left + 
    (objectWidth * 
    pixelsPerMetreX));

  int bottom = (int) (top + 
    (objectHeight * 
    pixelsPerMetreY));

  convertedRect.set(left, top, right, bottom);

  return convertedRect;
}
```

现在，我们实现了`Viewport`类的第二个主要功能，即移除当前对我们没有兴趣的对象。我们称这个过程为剪辑，我们将要调用的方法是`clipObjects()`。

再次，我们接收作为参数的物体的`x`、`y`、`width`和`height`。测试首先假设我们想要剪辑当前对象，并将`true`分配给`clipped`。

然后，四个嵌套的`if`语句测试对象的每一个点是否都在视口相关侧边的范围内。如果是，我们将`clipped`设置为`false`。我们设计的某些级别将包含超过一千个对象，但我们将会看到，在任何给定帧中，我们很少需要处理（更新、碰撞检测和绘制）超过四分之一的对象。输入`clipObjects`方法的代码：

```java

public boolean clipObjects(float objectX, 
  float objectY, 
  float objectWidth, 
  float objectHeight) {

  boolean clipped = true;

   if (objectX - objectWidth < 
    currentViewportWorldCentre.x + (metresToShowX / 2)) {

    if (objectX + objectWidth > 
      currentViewportWorldCentre.x - (metresToShowX / 2)) {

      if (objectY - objectHeight <           
        currentViewportWorldCentre.y + 
        (metresToShowY / 2)) {

        if (objectY + objectHeight >       
          currentViewportWorldCentre.y - 
          (metresToShowY / 2)){

                 clipped = false;
        }     
      }

    }

  }

   // For debugging
   if(clipped){
       numClipped++;
   }

   return clipped;
}
```

现在，我们提供了对`numClipped`变量的访问权限，以便它可以每帧被读取并重置为零。

```java
public int getNumClipped(){
  return numClipped;    
}

public void resetNumClipped(){
  numClipped = 0;
}

}// End of Viewport
```

让我们声明并初始化我们的`Viewport`对象。在`PlatformView`构造函数中初始化我们的`Paint`对象之后，添加以下代码。新代码在这里高亮显示：

```java
  // Initialize our drawing objects
  ourHolder = getHolder();
  paint = new Paint();

 // Initialize the viewport
 vp = new Viewport(screenWidth, screenHeight);

}// End of constructor
```

现在，我们可以描述并定位游戏世界中的对象，并专注于我们感兴趣的世界精确部分。让我们看看我们实际上是如何将对象放入那个世界的，这样我们就可以像以前一样更新和绘制它们。我们还将探讨关卡的概念。

### 创建关卡

在这里，我们将了解如何构建我们的`LevelManager`、`LevelData`和我们第一个真正的关卡`LevelCave`。

`LevelManager`类最终将需要我们`InputController`类的一个副本。因此，为了尽量遵循我们不需要删除任何代码的意图，我们将在`LevelManager`构造函数中包含一个`InputController`的参数。

让我们快速为我们的`InputController`类创建一个空白模板。按照通常的方式创建一个新类，并将其命名为`InputController`。添加以下代码：

```java
public class InputController {
    InputController(int screenWidth, int screenHeight) {
    }
}
```

现在，让我们看看我们最初非常简单的`LevelData`类。创建一个新类，将其命名为`LevelData`，并添加此代码。在这个阶段，它仅包含一个用于`Strings`的`ArrayList`对象。

```java
import java.util.ArrayList;

public class LevelData {
    ArrayList<String> tiles;

    // This class will evolve along with the project

    // Tile types
    // . = no tile
    // 1 = Grass

}
```

接下来，我们可以开始创建最终将成为我们第一个可玩关卡的代码。创建一个新类，将其命名为`LevelCave`，并添加此代码：

```java
import java.util.ArrayList;

public class LevelCave extends LevelData{
    LevelCave() {
    tiles = new ArrayList<String>();
    this.tiles.add("p.............................................");
    this.tiles.add("..............................................");
    this.tiles.add(".....................111111...................");
    this.tiles.add("..............................................");
    this.tiles.add("............111111............................");
    this.tiles.add("..............................................");
    this.tiles.add(".........1111111..............................");
    this.tiles.add("..............................................");
    this.tiles.add("..............................................");
    this.tiles.add("..............................................");
    this.tiles.add("..............................11111111........");
    this.tiles.add("..............................................");
    }
}
```

### 提示

在`LevelCave`文件中，`p`代表玩家的位置是任意的。只要它在里面，`Player`对象就会被初始化。玩家角色的实际生成位置由对`loadLevel`方法的调用决定，我们很快就会看到。我通常将代表玩家的`p`作为地图第一行第一个元素，这样就不太可能被遗忘。

现在，让我们谈谈这个关卡设计将如何工作。我们将在`LevelCave`类中的代码的`tiles.add("..."`部分输入字母数字字符。我们将根据要放入关卡的`GameObject`输入不同的字母数字字符。目前，我们只有一个`p`代表`Player`对象，一个`1`代表`Grass`对象，以及一个句点（`.`）代表一个游戏世界一平方米的空地。

### 提示

这意味着上一代码块中使用`1`字符定位`Grass`对象的方式可以完全按照你的喜好来安排。确实如此，每当我们查看`LevelCave`类的代码时，请随意即兴发挥和实验。

随着项目的进行，我们将添加超过二十个不同的`GameObject`子类。有些将像`Grass`一样是静止的，其他的将是具有思考能力的侵略性敌人。所有这些都可放置在我们的关卡设计中。

现在，我们可以实现一个类来管理我们的关卡。创建一个新的 Java 类，将其命名为`LevelManager`。随着我们逐步进行，输入`LevelManager`类的代码，一次讨论一个代码块。

首先，是一些导入指令。

```java
import android.content.Context;
import android.graphics.Bitmap;
import android.graphics.Rect;
import java.util.ArrayList;
```

现在，构造函数是我们有一个`String`类型的`level`来保存关卡名称，`mapWidth`和`mapHeight`以游戏世界米为单位存储当前关卡的宽度和高度，一个`Player`对象，因为我们知道我们总会有一个，以及一个名为`playerIndex`的`int`类型。

不久，我们将拥有许多`GameObject`类的`ArrayList`对象，始终拥有`Player`对象的索引将非常方便。

接下来，我们有布尔值`playing`，因为我们需要知道游戏是在进行中还是暂停，以及一个名为`gravity`的浮点数。

### 提示

在这个项目的背景下，重力不会发挥其全部潜力，但可以轻松地操纵它，使不同级别的重力不同。这就是为什么它在`LevelManager`类中的原因。

最后，我们声明一个`LevelData`类型的对象，一个用于保存所有`GameObject`对象的`ArrayList`对象，一个用于保存玩家控制按钮表示的`ArrayList`对象，以及一个常规数组用于保存我们大部分需要的`Bitmap`对象。

```java
public class LevelManager {

    private String level;
    int mapWidth;
    int mapHeight;

    Player player;
    int playerIndex;

    private boolean playing;
    float gravity;

    LevelData levelData;
    ArrayList<GameObject> gameObjects;

    ArrayList<Rect> currentButtons;
    Bitmap[] bitmapsArray;
```

然后，在构造函数中，我们检查签名并看到它接收一个`Context`对象，`pixelsPerMetre`（在`Viewport`类构造时确定），再次直接来自`Viewport`类的`screenWidth`，我们`InputController`类的一个副本，以及要加载的关卡名称。`int`参数`px`和`py`是玩家的起始坐标。

我们将关卡参数赋值给我们的成员级别，然后切换以确定哪个类将是我们的当前关卡。当然，目前我们只有一个`LevelCave`。

然后，我们初始化我们的`gameObject ArrayList`和`bitmapsArray`。然后我们调用`loadMapData()`，这是我们很快会编写的一个方法。在此之后，我们将`playing`设置为`true`，最后我们有一个获取器方法来找出`playing`的状态。在`LevelManager`类中输入我们刚刚讨论的代码：

```java
public LevelManager(Context context, 
    int pixelsPerMetre, int screenWidth, 
    InputController ic, 
    String level, 
    float px, float py) {

    this.level = level;

    switch (level) {
        case "LevelCave":
        levelData = new LevelCave();
        break;

        // We can add extra levels here

    }

    // To hold all our GameObjects
    gameObjects = new ArrayList<>();

    // To hold 1 of every Bitmap
    bitmapsArray = new Bitmap[25];

    // Load all the GameObjects and Bitmaps
    loadMapData(context, pixelsPerMetre, px, py);

    // Ready to play
    playing = true;
}

public boolean isPlaying() {
    return playing;
}
```

现在，我们有一个非常简单的方法，可以基于我们当前处理的`GameObject`类型获取任何`Bitmap`对象。这样，每个`GameObject`不必持有自己的`Bitmap`对象。例如，我们可以设计一个包含数百个`Grass`对象的关卡。这很容易就会用尽即使是现代平板电脑的内存。

我们的`getBitmap`方法接收一个`int`类型的索引值，并返回一个`Bitmap`对象。我们将在下一个方法中看到如何访问`index`的适当值：

```java
    // Each index Corresponds to a bitmap
    public Bitmap getBitmap(char blockType) {

        int index;
        switch (blockType) {
            case '.':
                index = 0;
                break;

            case '1':
                index = 1;
                break;

            case 'p':
                index = 2;
                break;

            default:
                index = 0;
                break;
        }// End switch

        return bitmapsArray[index];

 }// End getBitmap
```

下一个方法将使我们能够获得调用`getBitmap`方法的`index`。只要`char`案例与我们创建的各种`GameObject`子类持有的`type`值相对应，并且此方法返回的索引与`bitmapsArray`中适当`Bitmap`的索引相匹配，我们就只需要每个`Bitmap`对象的一个副本。

```java
// This method allows each GameObject which 'knows'
// its type to get the correct index to its Bitmap
// in the Bitmap array.
public int getBitmapIndex(char blockType) {

    int index;
        switch (blockType) {
            case '.':
                index = 0;
                break;

            case '1':
                index = 1;
                break;

            case 'p':
                index = 2;
                break;

            default:
                index = 0;
                break;

        }// End switch

        return index;
    }// End getBitmapIndex()
```

现在，我们使用`LevelManager`类进行实际的工作，并从我们的设计中加载关卡。该方法需要`pixelsPerMetre`和`Player`对象的坐标才能执行其工作。由于这是一个大方法，解释和代码已经被分成几个部分。

在这一部分，我们简单声明一个名为`index`的`int`类型，并将其设置为`-1`。当我们遍历我们的关卡设计时，它将帮助我们跟踪当前的位置。

然后，我们使用`ArrayList`的大小和`ArrayList`的第一个元素的长度分别计算地图的高度和宽度。

```java
// For now we just load all the grass tiles
// and the player. Soon we will have many GameObjects
private void loadMapData(Context context, 
  int pixelsPerMetre, 
  float px, float py) {

   char c;

   //Keep track of where we load our game objects
   int currentIndex = -1;

   // how wide and high is the map? Viewport needs to know
   mapHeight = levelData.tiles.size();
   mapWidth = levelData.tiles.get(0).length();
```

我们从`ArrayList`对象的第一个字符串的第一个元素开始进入嵌套的`for`循环。我们在移动到第二个字符串之前，从左到右遍历第一个字符串。

我们检查当前位置是否除了空格（.）之外还有其他对象，如果有，我们就进入一个开关块，在指定位置创建适当的对象。

如果我们遇到一个`1`，那么我们向`ArrayList`中添加一个新的`Grass`对象；如果遇到一个`p`，我们就在传递到`LevelManager`类构造函数的位置初始化`Player`对象。当一个新`Player`对象被创建时，我们还会初始化我们的`playerIndex`和`player`对象，以备将来使用。

```java
for (int i = 0; i < levelData.tiles.size(); i++) {
            for (int j = 0; j < 
                    levelData.tiles.get(i).length(); j++) {

                c = levelData.tiles.get(i).charAt(j);

                    // Don't want to load the empty spaces
                    if (c != '.'){ 
                      currentIndex++;
                      switch (c) {

                        case '1':
                            // Add grass to the gameObjects
                            gameObjects.add(new Grass(j, i, c));
                            break;

                        case 'p':
                            // Add a player to the gameObjects
                            gameObjects.add(new Player
                                (context, px, py, 
                                 pixelsPerMetre));

                            // We want the index of the player
                            playerIndex = currentIndex;
                            // We want a reference to the player
                            player = (Player)           
                            gameObjects.get(playerIndex);

                            break;

            }// End switch
```

如果一个新的对象被添加到`gameObjects ArrayList`中，我们需要检查相应的位图是否已经被添加到`bitmapsArray`中。如果没有，我们使用当前考虑的`GameObject`类的`prepareBitmap`方法添加一个。以下是执行此检查并在必要时准备位图的代码：

```java
// If the bitmap isn't prepared yet
if (bitmapsArray[getBitmapIndex(c)] == null) {

    // Prepare it now and put it in the bitmapsArrayList
    bitmapsArray[getBitmapIndex(c)] =
        gameObjects.get(currentIndex).
        prepareBitmap(context,                                                
        gameObjects.get(currentIndex).                                                        
        getBitmapName(),                                     
        pixelsPerMetre);

}// End if

}// End if (c != '.'){ 

}// End for j

}// End for i

}// End loadMapData()

}// End LevelManager
```

回到`PlatformView`类中，为了使用我们的所有关卡对象，我们在`PlatformView`构造函数中初始化`Viewport`类之后立即调用`loadLevel()`。新代码已经突出显示，并提供现有代码作为上下文：

```java
  // Initialize the viewport
  vp = new Viewport(screenWidth, screenHeight);

 // Load the first level
 loadLevel("LevelCave", 15, 2);

}
```

当然，现在我们需要在`PlatformView`类中实现`loadLevel`方法。

`loadLevel`方法需要知道要加载哪个关卡，这样`LevelManager`构造函数中的`switch`语句才能执行其工作，它还需要坐标来生成我们的英雄 Bob。

我们通过从`vp`获取的视口数据以及我们刚刚讨论的关卡/玩家数据调用其构造函数来初始化我们的`LevelManager`对象。

我们接着创建一个新的`InputController`类，同样从`vp`中传递一些数据。在第六章，*Bob, Beeps, 和 Bumps*中构建我们的`InputController`类时，我们会确切地看到如何使用这些数据。最后，我们调用`vp.setWorldCentre()`，并将玩家的位置坐标传递给它，这样屏幕就居中了 Bob。

```java
public void loadLevel(String level, float px, float py) {

    lm = null;

    // Create a new LevelManager
    // Pass in a Context, screen details, level name 
    // and player location
    lm = new LevelManager(context, 
        vp.getPixelsPerMetreX(), 
        vp.getScreenWidth(), 
        ic, level, px, py);

    ic = new InputController(vp.getScreenWidth(),       
        vp.getScreenHeight());

    // Set the players location as the world centre     
    vp.setWorldCentre(lm.gameObjects.get(lm.playerIndex)
        .getWorldLocation().x,
        lm.gameObjects.get(lm.playerIndex)
        .getWorldLocation().y);
    }
```

我们可以在我们的`update`方法中添加一些代码，这将首先利用我们新的`Viewport`类的主要功能。

### 增强的更新方法

最后，我们可以使用我们的`ArrayList`游戏对象和`Viewport`功能来完善我们的增强型`update`方法。在下面的代码中，我们仅使用增强的`for`循环遍历每个`GameObject`。我们检查它是否`isActive()`，然后通过`if`语句将对象的位置和尺寸传递给`clipObjects()`。如果`clipObjects()`返回`false`，则对象没有被剪辑，并通过调用`go.setVisible(true)`将对象标记为可见。否则，通过调用`go.setVisible(false)`将其标记为不可见。这是此刻更新任何对象的唯一方面。我们将在本章末尾运行游戏时看到，它已经很有用了。在`update`方法中输入新代码：

```java
for (GameObject go : lm.gameObjects) {
        if (go.isActive()) {
            // Clip anything off-screen
            if (!vp.clipObjects(go.getWorldLocation().x,                                
                go.getWorldLocation().y, 
                go.getWidth(), 
                go.getHeight())) {

                // Set visible flag to true
                go.setVisible(true);

            } else {
                // Set visible flag to false
                go.setVisible(false);
                // Now draw() can ignore them

            }
        }

    }
}
```

### 增强的绘制方法

现在，我们可以更精确地确定我们需要绘制哪些对象。首先，我们声明并初始化一个新的名为`toScreen2d`的`Rect`对象。

然后，我们从最低层开始，针对每一层遍历一次`gameObjects ArrayList`。在这个阶段，这并不是严格必要的，因为我们的所有对象默认都当前在零层。在项目结束前，我们将添加位于-1 层和 1 层的对象，如果我们能够避免，则不想重写代码。

接下来，我们检查对象是否可见并且是否在当前层。如果是，我们将当前对象的位置和尺寸传递给`worldToScreen`方法，该方法将结果返回给我们之前准备的`toScreen2d Rect`对象。然后，我们使用`bitmapArray`调用`drawBitmap()`以提供适当的位图，并传入`toScreen2d`的坐标。更新突出显示的`draw`方法：

```java
private void draw() {

    if (ourHolder.getSurface().isValid()) {
        //First we lock the area of memory we will be drawing to
        canvas = ourHolder.lockCanvas();

        // Rub out the last frame with arbitrary color
        paint.setColor(Color.argb(255, 0, 0, 255));
        canvas.drawColor(Color.argb(255, 0, 0, 255));
 // Draw all the GameObjects
 Rect toScreen2d = new Rect();

 // Draw a layer at a time
 for (int layer = -1; layer <= 1; layer++){
 for (GameObject go : lm.gameObjects) {
 //Only draw if visible and this layer
 if (go.isVisible() && go.getWorldLocation().z 
 == layer) { 

 toScreen2d.set(vp.worldToScreen
 (go.getWorldLocation().x,
 go.getWorldLocation().y,
 go.getWidth(),
 go.getHeight()));

 // Draw the appropriate bitmap
 canvas.drawBitmap(
 lm.bitmapsArray
 [lm.getBitmapIndex(go.getType())],
 toScreen2d.left,
 toScreen2d.top, paint);
 }
 }
}

```

现在，仍然在`draw`方法中，我们将调试信息打印到屏幕上，包括我们的`gameObjects ArrayList`的大小与这一帧中被剪辑的对象数量比较。

然后，我们通过常规调用`unlockCanvasAndPost()`来完成`draw`方法。注意，在`if(debugging)`块的末尾，我们调用`vp.resetNumClipped`将`numClipped`变量重置为零，为下一帧做准备。在`draw`方法中的上一代码块之后直接添加此代码：

```java
// Text for debugging
if (debugging) {
 paint.setTextSize(16);
 paint.setTextAlign(Paint.Align.LEFT);
 paint.setColor(Color.argb(255, 255, 255, 255));
 canvas.drawText("fps:" + fps, 10, 60, paint);

 canvas.drawText("num objects:" + 
 lm.gameObjects.size(), 10, 80, paint);

 canvas.drawText("num clipped:" + 
 vp.getNumClipped(), 10, 100, paint);

 canvas.drawText("playerX:" + 
 lm.gameObjects.get(lm.playerIndex).
 getWorldLocation().x,
 10, 120, paint);

 canvas.drawText("playerY:" + 
 lm.gameObjects.get(lm.playerIndex).
 getWorldLocation().y, 
 10, 140, paint);

 //for reset the number of clipped objects each frame
 vp.resetNumClipped();

}// End if(debugging)

// Unlock and draw the scene
ourHolder.unlockCanvasAndPost(canvas);

}// End (ourHolder.getSurface().isValid())
}// End draw()
```

在这个项目中，我们第一次实际运行游戏并看到了一些结果：

![增强的绘制方法](img/B04322_05_03.jpg)

注意图像中我们`LevelCave`设计中草地的精确布局。您还可以看到我们压缩的 Bob 精灵表和有 28 个对象，但其中 10 个已被剪辑。随着我们的关卡变得越来越大，剪辑与未剪辑的比例将大幅增加，绝大多数对象将被剪辑。

# 总结

在本章中，我们已经介绍了许多内容，现在拥有了一个完善的游戏引擎。

由于我们已经完成了大部分设置工作，从现在开始，我们添加的大部分代码也将有可见（或可听）的结果，并将更加令人满意，因为我们将能够定期运行我们的游戏以查看改进。

在下一章中，我们将添加声音效果和输入检测，从而让 Bob 栩栩如生。然后，我们将会看到他的世界可能多么危险，并将迅速添加碰撞检测，使他能够站在平台上。
