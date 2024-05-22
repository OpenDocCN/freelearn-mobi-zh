# 第九章。使用 OpenGL ES 2 以 60 FPS 实现小行星游戏

欢迎来到最终项目。在接下来的三章中，我们将使用 OpenGL ES 2 图形 API 构建一个类似小行星的游戏。如果你好奇 OpenGL ES 2 到底是什么，我们将在本章后面讨论其细节。

我们将构建一个非常简单但有趣且具有挑战性的游戏，可以在一次绘制和动画化数百个对象，即使在相当老旧的 Android 设备上也能实现。

使用 OpenGL，我们将把绘图效率提升到一个更高的层次，通过一些不太复杂的数学运算，我们的移动和碰撞检测将比之前的项目大大增强。

在本章结束时，我们将拥有一个基本的可工作的 OpenGL ES 2 引擎，以 60 FPS 或更高的帧率绘制我们简单但暂时静态的飞船到屏幕上。

### 提示

如果你从未见过或玩过 1979 年 11 月发布的 80 年代街机游戏（Asteroids），为什么不去看看它的克隆版或视频呢？

在[`www.freeasteroids.org/`](http://www.freeasteroids.org/)免费玩网络游戏。

在 YouTube 上观看[`www.youtube.com/watch?v=WYSupJ5r2zo`](https://www.youtube.com/watch?v=WYSupJ5r2zo)。

让我们确切地讨论一下我们打算构建的内容。

# 小行星模拟器

我们的游戏设定在一个四个方向滚动的世界中，玩家可以在寻找小行星的同时穿越这个世界。世界将被一个矩形边界所包围，以防止小行星漂移得太远，这个边界也将成为玩家需要避开的另一个障碍。

## 游戏控制

我们将重新使用我们的`InputController`类，并进行一些简单的修改，甚至可以保持相同的按钮布局。然而，正如我们将看到的，我们将以与复古平台游戏非常不同的方式在屏幕上绘制按钮。此外，玩家将旋转飞船向左和向右 360 度，而不是左右走动。跳跃按钮将变成一个推进开关，以开启或关闭前进动力，而射击按钮将保持原样。我们还将保持暂停按钮在同一位置。

## 游戏规则

当小行星撞击边界时，它将反弹回游戏世界。如果玩家撞击边界，将损失一条生命，并且飞船将在屏幕中心重新生成。如果小行星撞击飞船，这将是致命的。

玩家将从三条生命开始，必须清除所有小行星模拟器中的小行星。抬头显示（HUD）将显示剩余小行星和生命的计数。如果玩家清除了所有小行星，下一波将会比上一波更多，并且它们的移动速度会稍快一些。每清除一波，玩家将获得一条额外的生命。

我们将在项目进行中实施这些规则。

# 介绍 OpenGL ES 2

OpenGL ES 2 是针对嵌入式系统的**开放图形库**（**OpenGL**）的第二个主要版本。它是桌面系统 OpenGL 在移动设备上的化身。

## 为什么使用它以及它是如何工作的？

OpenGL 作为一个本地进程运行，而不是像我们其他 Java 代码那样在 Dalvik 虚拟机上运行。这是它超级快速的一个原因。OpenGL ES API 消除了与本地代码交互的所有复杂性，而 OpenGL 本身在其本地代码库中也提供了非常高效和快速的算法。

第一版的 OpenGL 在 1992 年完成。重点是即使在那时，OpenGL 也使用了可以说最高效的代码和算法来绘制图形。现在，20 多年后，它一直在不断改进和完善，同时适配最新的图形硬件，包括移动端和桌面端。所有移动 GPU 制造商都专门设计硬件以兼容最新版本的 OpenGL ES。

因此，试图改进 OpenGL ES 可能是一项愚蠢的尝试。

### 提示

当专门为 Windows 设备开发时，还有另一个可行的图形 API 选项，即 DirectX。

## 第二版的亮点是什么？

第一版的 OpenGL ES 在当时的确令人印象深刻。我记得当我第一次在手机上玩 3D 射击游戏时，几乎从椅子上掉下来！现在这当然很常见。然而，与桌面版的 OpenGL 相比，OpenGL ES 1 有一个重大缺点。

OpenGL ES 1 有一个所谓的固定功能管线。要绘制的几何图形输入到 GPU 中，它就会被绘制，但任何对单个像素的进一步操作都需要在 OpenGL ES 接管游戏帧的绘制之前进行。

现在，通过 OpenGL ES 2，我们可以使用所谓的可编程管线。也就是说，我们可以将图形发送出去进行绘制，同时我们还可以编写在 GPU 上运行的代码，这种代码能够独立操作每一个像素。这是一个非常强大的特性，尽管我们不会深入探讨它。

在 GPU 上运行的这段额外代码称为**着色器**程序。我们可以编写代码在所谓的**顶点着色器**中操作图形的几何（位置）。我们还可以编写代码，单独操作每一个像素的外观，这称为**片断着色器**。

### 注意

实际上，我们甚至可以比像素操作做得更好。片断不一定是像素。这取决于硬件和正在处理的图形的具体性质。它可以是一个以上的像素或子像素：屏幕硬件中组成一个像素的多个光线之一。

对于这种简单游戏而言，OpenGL ES 2 的缺点在于，即使你不会大量使用它们，也必须至少提供一个顶点和片断着色器。然而，正如我们将看到的，这并不困难。虽然我们不会深入探讨着色器，但我们将使用**GL Shader Language** (**GLSL**)编写一些着色器代码，并一窥它们所提供的可能性。

### 提示

如果可编程图形管线和着色器的力量让你兴奋到无法等待，那么我强烈推荐 Jacobo Rodríguez 编写的*GLSL Essentials*。

[GLSL 基础](https://www.packtpub.com/hardware-and-creative/glsl-essentials)

这本书探讨了桌面上的 OpenGL 着色器，对于任何具有基本编程知识并愿意学习不同语言（GLSL）的读者来说，都是高度可访问的，尽管它有一些与 Java 相似的语法。

我们将如何使用 OpenGL ES 2？

## 我们将如何使用 OpenGL ES 2？

在 OpenGL 中，一切都是点、线或三角形。此外，我们可以将颜色和纹理附加到这种基本几何图形上，并将这些元素组合起来制作出现在现代移动游戏中的复杂图形。

我们将使用各种类型的元素（点、线和三角形），这些元素统称为图元。

在这个项目中，我们将不使用纹理。幸运的是，未纹理化的图元的外观适合构建类似小行星的游戏。

除了图元，OpenGL 还使用矩阵。**矩阵**是一种进行算术的方法和结构。这种算术可以从非常简单的高中水平的计算来移动（转换）一个坐标，也可以进行更复杂的计算，将我们的游戏世界坐标转换为 GPU 可以使用的 OpenGL 屏幕坐标。

重点在于，无论是矩阵还是使用它们的方法，完全由 OpenGL API 提供。这意味着我们只需了解哪些方法可以进行哪些图形操作，无需关心背后可能复杂的数学运算（在 GPU 上进行的）。

学习 OpenGL 中的着色器、图元和矩阵的最佳方式是继续使用它们。

# 准备 OpenGL ES 2

首先，我们从`Activity`类开始，这和之前一样，是进入我们游戏的入口点。创建一个新项目，在**应用名称**字段中输入`C9 Asteroids`。选择**手机和平板**，然后在提示时选择**空白活动**。在**活动名称**字段中输入`AsteroidsActivity`。

### 提示

显然，你不必遵循我确切的命名选择，但只需记得在代码中进行小改动，以反映你自己的命名选择。

你可以从`layout`文件夹中删除`activity_asteroids.xml`。你也可以删除`AsteroidsActivity.java`文件中的所有代码。只需保留包声明。

## 将布局锁定为横屏

正如我们在前两个项目中做的那样，我们将确保游戏只在横屏模式下运行。我们将使我们的`AndroidManifest.xml`文件，强制我们的`AsteroidsActivity`类以全屏运行，并将其锁定为横屏方向。让我们进行这些更改：

1.  现在打开`manifests`文件夹，双击`AndroidManifest.xml`文件，在代码编辑器中打开它。

1.  在`AndroidManifest.xml`文件中，找到以下代码行：

    ```java
    android:name=".AsteroidsActivity"
    ```

1.  立即输入或复制粘贴以下两行代码，使`PlatformActivity`全屏运行，并将其锁定为横屏方向：

    ```java
    android:theme="@android:style/Theme.NoTitleBar.Fullscreen"
    android:screenOrientation="landscape"
    ```

现在，我们可以继续使用 OpenGL 实现我们的 Asteroids 模拟器游戏。

## 活动

首先，我们有一个熟悉的`Activity`类。这里唯一的新事物是我们视图类的类型。我们声明了一个名为`asteroidsView`的成员，其类型为`GLSurfaceView`。这个类将为我们提供轻松访问 OpenGL 的途径。我们很快就会看到具体如何实现。注意，我们所做的一切就是通过传递`Activity`上下文和以通常方式获取的屏幕分辨率来初始化`GLSurfaceView`。按照所示实现`AsteroidsActivity`类：

```java
package com.gamecodeschool.c9asteroids;

import android.app.Activity;
import android.graphics.Point;
import android.opengl.GLSurfaceView;
import android.os.Bundle;
import android.view.Display;

public class AsteroidsActivity extends Activity {

    private GLSurfaceView asteroidsView;

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        // Get a Display object to access screen details
        Display display = getWindowManager().getDefaultDisplay();

        // Load the resolution into a Point object
        Point resolution = new Point();
        display.getSize(resolution);

        asteroidsView = new AsteroidsView 
          (this, resolution.x, resolution.y); 

        setContentView(asteroidsView);
    }

    @Override
    protected void onPause() {
        super.onPause();

        asteroidsView.onPause();

    }

    @Override
    protected void onResume() {
        super.onResume();

        asteroidsView.onResume();

    }
}
```

接下来，我们将看到一些 OpenGL 代码。

## 视图

在这里，我们将实现`GLSurfaceView`类。实际上，这里并不是真正的动作发生的地方，但它确实允许我们附加一个 OpenGL 渲染器。这是一个实现了`Renderer`接口的类。除了这个关键的`Renderer`之外，`GLSurfaceView`类还允许我们覆盖`onTouchListener`方法，这将允许我们以与前一个项目中`SurfaceView`相同的方式来检测玩家输入。

### 注意

Android Studio 不会自动导入甚至建议所有必需的 OpenGL 导入。因此，我在代码清单中包含了一些类的所有导入。此外，你会注意到我们有时使用静态导入。这将使代码更具可读性。

在下面的代码中，我们声明并初始化了一个即将实现的`GameManager`类型的新对象。我们通过调用`setEGLContextClientVersion(2)`将 OpenGL 版本设置为 2，并通过调用`setRenderer()`并传入我们的`GameManager`对象来设置我们关键的渲染器对象。创建一个名为`AsteroidsView`的新类，并按以下方式实现它：

```java
import android.content.Context;
import android.opengl.GLSurfaceView;

public class AsteroidsView extends GLSurfaceView{

    GameManager gm;

    public AsteroidsView(Context context, int screenX, int screenY) {
        super(context);

        gm = new GameManager(screenX, screenY);

        // Which version of OpenGl we are using
        setEGLContextClientVersion(2);

        // Attach our renderer to the GLSurfaceView
        setRenderer(new AsteroidsRenderer(gm));

    }

}
```

现在，我们可以看看我们的`GameManager`类涉及到哪些内容。

## 一个管理我们游戏的类

这个类将控制诸如玩家所在的关卡、生命数量以及游戏世界的整体大小之类的事情。随着项目的进行，它会有一些变化，但与之前项目中的`LevelManager`和`PlayerState`类的综合深度相比，它将保持相当简单，尽管它实际上取代了这两个类。

在下面的代码中，我们声明了`int`类型的成员来保存游戏世界的宽度和高度；我们可以根据需要将其设置得更大或更小。我们使用布尔值`playing`来跟踪游戏的状态。

`GameManager`类还需要知道屏幕的宽度和高度（以像素为单位），这个信息是在`AsteroidsView`类中初始化对象时传递给构造函数的。

也请注意`metresToShowX`和`metresToShowY`成员变量。这些可能听起来很熟悉，因为它们来自上一个项目的`Viewport`类。这些变量将被用于完全相同的事情：定义游戏世界的当前可查看区域。然而这次，OpenGL 将负责在绘制之前裁剪哪些对象（使用矩阵）。我们很快就会看到这是在哪里发生的。

### 注意

请注意，尽管 OpenGL 负责裁剪和缩放我们想要显示的游戏世界区域，但它对每帧更新哪些对象没有任何影响。然而，正如我们将要看到的，这正是我们想要的游戏效果，因为即使对象在屏幕外，我们也希望所有对象每帧都能更新自己。因此，这个游戏不需要`Viewport`类。

最后，我们需要一个便捷的方式来暂停和继续游戏，我们通过`switchPlayingStatus`方法提供这个功能。创建一个名为`GameManager`的新类，并按照所示实现它：

```java
public class GameManager {

    int mapWidth = 600;
    int mapHeight = 600;
    private boolean playing = false;

    // Our first game object
    SpaceShip ship;

    int screenWidth;
    int screenHeight;

    // How many metres of our virtual world
    // we will show on screen at any time.
    int metresToShowX = 390;
    int metresToShowY = 220;

    public GameManager(int x, int y){

        screenWidth = x;
        screenHeight = y;

    }

    public void switchPlayingStatus() {
        playing = !playing;

    }

    public boolean isPlaying(){
        return playing;
    }
}
```

现在我们可以第一次了解这些强大的着色器以及我们将如何管理它们。

## 管理简单的着色器

应用程序可以有许多着色器。然后我们可以将不同的着色器附加到不同的游戏对象上，以创建所需的效果。

在这个游戏中，我们只会有一个顶点着色器和一个片段着色器。然而，当你了解到如何将着色器附加到图元上时，你会发现拥有更多着色器是非常简单的。

1.  首先，我们需要在 GPU 中执行的着色器的代码。

1.  然后，我们需要编译那段代码。

1.  最后，我们需要将两个编译后的着色器链接成一个 GL 程序。

当我们实现这个下一个简单的类时，我们将看到如何将这个功能打包到一个方法调用中，这个调用可以由游戏中的对象发出，并将准备运行的 GL 程序返回给游戏对象。在本章后面构建我们的`GameObject`类时，我们将看到如何使用这个 GL 程序。

让我们在一个新类中实现必要的三个步骤。创建一个新类，并将其命名为`GLManager`。添加如下所示的静态导入：

```java
import static android.opengl.GLES20.GL_FRAGMENT_SHADER;
import static android.opengl.GLES20.GL_VERTEX_SHADER;
import static android.opengl.GLES20.glAttachShader;
import static android.opengl.GLES20.glCompileShader;
import static android.opengl.GLES20.glCreateProgram;
import static android.opengl.GLES20.glCreateShader;
import static android.opengl.GLES20.glLinkProgram;
import static android.opengl.GLES20.glShaderSource;
```

接下来，我们将添加一些公共静态最终成员变量，我们可以在本章后面的`GameObject`类中使用它们。虽然我们将在使用它们时确切地看到它们是如何工作的，但这里有一个快速预览解释。

`COPONENTS_PER_VERTEX`是用于表示构成游戏对象的图元中的单个顶点（点）的值数量。如您所见，我们将这个值初始化为三个坐标：*x*，*y*和*z*。

我们还有`FLOAT_SIZE`，它被初始化为`4`。这是 Java 浮点数的字节数。正如我们很快将看到的，OpenGL 喜欢所有传入它的图元以`ByteBuffer`的形式。我们需要确保精确地知道在`ByteBuffer`中的每个信息片段的位置。

接下来，我们声明`STRIDE`并将其初始化为`COMPONENTS_PER_VERTEX * FLOAT_SIZE`。由于 OpenGL 使用浮点类型来处理几乎所有其工作的数据，`STRIDE`现在等于表示单个物体顶点的数据大小（以字节为单位）。请将这些成员添加到类的顶部：

```java
public class GLManager {

     // Some constants to help count the number of bytes between
     // elements of our vertex data arrays
     public static final int COMPONENTS_PER_VERTEX = 3;
     public  static final int FLOAT_SIZE = 4;
     public static final int STRIDE =
       (COMPONENTS_PER_VERTEX)
        * FLOAT_SIZE;

     public static final int ELEMENTS_PER_VERTEX = 3;// x,y,z
```

GLSL 是一种自身的语言，并且它也有自己的类型，这些类型的变量可以被利用。在这里，我们声明并初始化一些字符串，我们可以使用它们在代码中更清晰地引用这些变量。

这些类型的讨论超出了本书的范围，但简单来说，它们将代表一个矩阵（`u_matrix`）、一个位置（`a_position`）和一个颜色（`u_Color`）。我们很快将在我们的着色器代码中看到这些变量实际的 GLSL 类型。

在字符串之后，我们声明了三个`int`类型。这三个公共静态（但不是最终的）成员将用于存储我们着色器中同名类型的位置。这使得我们可以在给 OpenGL 最终的绘图指令之前，在着色器程序中操作这些值。

```java
// Some constants to represent GLSL types in our shaders
public static final String U_MATRIX = "u_Matrix";
public static final String A_POSITION = "a_Position";
public static final String U_COLOR = "u_Color";

// Each of the above constants also has a matching int
// which will represent its location in the open GL glProgram
public static int uMatrixLocation;
public static int aPositionLocation;
public static int uColorLocation;
```

最后，我们来到了打包在字符串中的顶点着色器 GLSL 代码。注意，我们声明了一个名为`u_Matrix`的统一变量，类型为`mat4`，以及一个类型为属性`vec4`的`a_Position`。稍后在我们`GameObject`类中，我们将看到如何获取这些变量的位置，以便我们可以从 Java 代码中为它们传递值。

以`void main()`开始的代码行是实际着色器代码执行的地方。注意`gl_position`被分配了我们刚才声明的两个变量的乘积值。同时`gl_PointSize`被分配了`3.0`的值。这将是我们绘制所有点图元的大小。在之前的代码块之后，输入顶点着色器的代码：

```java
// A very simple vertexShader glProgram
// that we can define with a String

private static String vertexShader =
     "uniform mat4 u_Matrix;" +
     "attribute vec4 a_Position;" +

     "void main()" +
     "{" +
       "gl_Position = u_Matrix * a_Position;" +
       "gl_PointSize = 3.0;"+
  "}";
```

接下来，我们将实现片元着色器。这里发生了一些事情。首先，以`precision mediump float`开头的行告诉 OpenGL 以中等精度（因此也是中等速度）进行绘制。然后我们可以看到我们的变量`u_Color`被声明为统一类型`vec4`。我们很快会在`GameObject`类中看到如何将`color`值传递给这个变量。

当在`void main()`开始执行时，我们只需将`u_Color`分配给`gl_FragColor`。因此，无论分配给`u_Color`什么颜色，所有片段都将具有那个颜色。在片元着色器之后，我们声明了一个名为`program`的`int`，它将作为我们 GL 程序的句柄。

在前面的代码块之后，输入片元着色器的代码：

```java
// A very simple vertexShader glProgram
// that we can define with a String

private static String vertexShader =
    "uniform mat4 u_Matrix;" +
    "attribute vec4 a_Position;" +

    "void main()" +
    "{" +
        "gl_Position = u_Matrix * a_Position;" +
        "gl_PointSize = 3.0;"+
    "}";
```

这是一个获取器方法，它返回 GL 程序的句柄：

```java
public static int getGLProgram(){
  return program;
}
```

下一个方法可能看起来复杂，但它所做的只是将一个编译并链接的程序返回给调用者。它通过使用`compileVertexShader()`和`compileFragmentShader()`作为参数调用 OpenGL 的`linkProgram`方法来实现。接下来，我们看到这两个新方法，它们只需要使用代表着色器类型的 OpenGL 常量以及包含匹配着色器 GLSL 代码的适当字符串来调用我们的方法`compileShader()`。

将我们刚刚讨论的三个方法输入到`GLManager`类中：

```java
public static int buildProgram(){
    // Compile and link our shaders into a GL glProgram object
    return linkProgram(compileVertexShader(),compileFragmentShader());

}

private static int compileVertexShader() {
    return compileShader(GL_VERTEX_SHADER, vertexShader);
}

private static int compileFragmentShader() {
    return compileShader(GL_FRAGMENT_SHADER, fragmentShader);
}
```

现在我们来了解当调用方法`compileShader()`时会发生什么。首先，我们会根据`type`参数创建一个着色器的句柄。然后，我们将该句柄和代码传递给`glShaderSource()`。最后，我们使用`glCompileShader()`编译着色器，并将句柄返回给调用方法：

```java
private static int compileShader(int type, String shaderCode) {

    // Create a shader object and store its ID
    final int shader = glCreateShader(type);

    // Pass in the code then compile the shader
    glShaderSource(shader, shaderCode);
    glCompileShader(shader);

    return shader;
}
```

现在我们可以看到这个过程最后一步。我们使用`glCreateProgram()`创建一个空程序。然后依次使用`glAttachShader()`附加每个编译后的着色器，并最终使用`glLinkProgram()`将它们链接成一个我们可以实际使用的程序：

```java
private static int linkProgram(int vertexShader, int fragmentShader) {

  // A handle to the GL glProgram -
  // the compiled and linked shaders
     program = glCreateProgram();

     // Attach the vertex shader to the glProgram.
     glAttachShader(program, vertexShader);

     // Attach the fragment shader to the glProgram.
     glAttachShader(program, fragmentShader);

     // Link the two shaders together into a glProgram.
     glLinkProgram(program);

     return program;
}
}// End GLManager
```

注意我们创建了一个程序，并且可以通过其句柄和`getProgram`方法访问它。我们还可以访问我们创建的所有那些公共静态成员，因此我们将能够从 Java 代码中调整着色器程序中的变量。

## 游戏的主循环——渲染器

现在我们将看到代码的核心部分将如何进行。创建一个名为`AsteroidsRenderer`的新类。这是我们附加到`GLSurfaceView`的渲染器类。按照如下方式添加导入语句，注意其中有一些是静态的：

```java
import android.graphics.PointF;
import android.opengl.GLSurfaceView.Renderer;
import android.util.Log;
import javax.microedition.khronos.egl.EGLConfig;
import javax.microedition.khronos.opengles.GL10;
import static android.opengl.GLES20.GL_COLOR_BUFFER_BIT;
import static android.opengl.GLES20.glClear;
import static android.opengl.GLES20.glClearColor;
import static android.opengl.GLES20.glViewport;
import static android.opengl.Matrix.orthoM;
```

现在我们将构建这个类。首先要注意的是，我们之前提到过这个类实现了`Renderer`，因此我们需要重写三个方法。它们是`onSurfaceCreated()`、`onSurfaceChanged()`和`onDrawFrame()`。此外，我们将在该类中最初添加一个构造函数来设置一切，一个`createObjects`方法，我们最终将在其中初始化所有游戏对象，一个`update`方法，我们将在其中每帧更新所有对象，以及一个`draw`方法，我们将在其中每帧绘制所有对象。

我们将在实现每个方法时探讨和解释它，我们还将看到我们的方法如何融入到 OpenGL 渲染器系统中，该系统决定了这个类的流程。

首先，有一些成员变量值得我们仔细查看。

布尔值`debugging`将被用来切换控制台输出的开启和关闭。`frameCounter`、`averageFPS`和`fps`变量不仅将被用来检查我们达到的帧率，而且还将被用来传递给我们的游戏对象，这些对象将根据每帧的流逝时间更新自己。

我们第一个真正有趣的变量是浮点数数组`viewportMatrix`。顾名思义，它将保存一个 OpenGL 可以用来计算到我们游戏世界的视口的矩阵。

我们有一个`GameManager`来保存对`GameManager`对象的引用，这是`AsteroidsView`在传递到这个类的构造函数中。最后，我们有两个`PointF`对象。

我们将在构造函数中初始化`PointF`对象，并将它们用于一些不同的事情，以避免在主游戏循环中取消引用任何对象。当垃圾收集器开始清理丢弃的对象时，即使是 OpenGL 也会减慢速度。避免召唤垃圾收集器将是整个游戏的目标。

在`AsteroidsRenderer`类的顶部输入成员变量：

```java
public class AsteroidsRenderer implements Renderer {

// Are we debugging at the moment

boolean debugging = true;

// For monitoring and controlling the frames per second

long frameCounter = 0;
long averageFPS = 0;
private long fps;

// For converting each game world coordinate
// into a GL space coordinate (-1,-1 to 1,1)
// for drawing on the screen

private final float[] viewportMatrix = new float[16];

// A class to help manage our game objects
// current state.

private GameManager gm;

// For capturing various PointF details without
// creating new objects in the speed critical areas

PointF handyPointF;
PointF handyPointF2;
```

这是我们的构造函数，我们从参数初始化`GameManager`引用，并创建两个方便的`PointF`对象以备使用：

```java
public AsteroidsRenderer(GameManager gameManager) {

     gm = gameManager;

     handyPointF = new PointF();
     handyPointF2 = new PointF();

}
```

这是第一个重写的方法。每次创建带有附加渲染器的`GLSurfaceView`类时都会调用它。我们调用`glClearColor()`来设置每次 OpenGL 清除屏幕时使用的颜色。然后我们使用`GLManager.buildProgram()`方法构建我们的着色器程序，并调用我们将很快编写的`createObjects`方法。

```java
@Override
public void onSurfaceCreated(GL10 glUnused, EGLConfig config) {

   // The color that will be used to clear the
   // screen each frame in onDrawFrame()
   glClearColor(0.0f, 0.0f, 0.0f, 0.0f);

   // Get GLManager to compile and link the shaders into an object
   GLManager.buildProgram();

   createObjects();

}
```

下一个重写的方法在`onSurfaceCreated()`之后调用一次，并且在屏幕方向改变时调用。在这里，我们调用`glViewport()`方法，告诉 OpenGL 将 OpenGL 坐标系统映射到哪个像素坐标上。

OpenGL 坐标系统与我们之前在两个项目中习惯处理的像素坐标有很大不同。屏幕的中心是 0,0，左下角是-1，右上角是 1。

![游戏主循环 - 渲染器](img/B04322_09_01.jpg)

前面的情况由于大多数屏幕不是正方形而进一步复杂化，但-1 到 1 的范围必须同时表示*x*和*y*轴。幸运的是，我们的`glViewport()`已经为我们处理了这个问题。

在这个方法中最后我们看到的是调用`orthoM`方法，将我们的`viewportMatrix`作为第一个参数。OpenGL 现在将准备使用`viewportMatrix`。`orthoM()`方法创建一个矩阵，将坐标转换为正交视图。如果我们的坐标是三维的，它将使所有物体看起来距离相同。由于我们正在制作一个二维游戏，这也适合我们。

输入`onSurfaceChanged`方法的代码：

```java
@Override
    public void onSurfaceChanged(GL10 glUnused, int width, int height) {

        // Make full screen
        glViewport(0, 0, width, height);

        /*
            Initialize our viewport matrix by passing in the starting
            range of the game world that will be mapped, by OpenGL to
            the screen. We will dynamically amend this as the player
            moves around.

            The arguments to setup the viewport matrix:
            our array,
            starting index in array,
            min x, max x,
            min y, max y,
            min z, max z)
        */

            orthoM(viewportMatrix, 0, 0, 
        gm.metresToShowX, 0, 
        gm.metresToShowY, 0f, 1f);
}
```

这是我们的`createObjects`方法，如您所见，我们创建了一个`SpaceShip`类型的对象，并将地图的高度和宽度传递给构造函数。我们将在本章后面构建`SpaceShip`类及其父类`GameObject`。输入`createObjects`方法：

```java
    private void createObjects() {
        // Create our game objects

        // First the ship in the center of the map
        gm.ship = new SpaceShip(gm.mapWidth / 2, gm.mapHeight / 2);
    }
```

这是重写的`onDrawFrame`方法。系统会连续调用它。我们可以通过设置渲染模式来控制何时调用它，当我们把`AsteroidsRenderer`附加到视图上，但默认的 OpenGL 控制连续调用正是我们所需要的。

我们将`startFrameTime`设置为当前的系统时间。然后，如果`isPlaying()`返回`true`，我们调用即将实现的`update`方法。然后，我们调用`draw()`，这将告诉我们的所有对象绘制自己。

然后，我们更新`timeThisFrame`和`fps`，可以选择在每 100 帧输出一次平均每秒帧数，如果我们正在调试的话。

现在我们知道 OpenGL 每秒最多可以调用`onDrawFrame()`数百次。我们将条件性地每次调用我们的`update`方法，以及调用我们的`draw`方法。除了实际的`draw`和`update`方法本身，我们已经有效地实现了游戏循环。

向类中添加`onDrawFrame`方法：

```java
@Override
public void onDrawFrame(GL10 glUnused) {

        long startFrameTime = System.currentTimeMillis();

        if (gm.isPlaying()) {
            update(fps);
        }

        draw();

        // Calculate the fps this frame
        // We can then use the result to
        // time animations and more.
        long timeThisFrame = System.currentTimeMillis() - startFrameTime;
        if (timeThisFrame >= 1) {
            fps = 1000 / timeThisFrame;
        }

        // Output the average frames per second to the console
        if (debugging) {
            frameCounter++;
            averageFPS = averageFPS + fps;
            if (frameCounter > 100) {
                averageFPS = averageFPS / frameCounter;
                frameCounter = 0;
                Log.e("averageFPS:", "" + averageFPS);
            }
        }
    }
```

这是我们的`update`方法，现在先留一个空体：

```java
    private void update(long fps) {

    }
```

现在，我们来看看`draw`方法，它从`onDrawFrame`方法中每帧调用一次。在这里，我们将飞船的当前位置加载到我们的便捷`PointF`对象之一中。显然，由于我们还没有实现`SpaceShip`类，这个方法调用将产生错误。

在`draw()`中我们接下来要做的事情相当有趣。我们根据游戏世界中当前的位置以及分配给`metresToShowX`和`metresToShowY`的值修改我们的`viewportMatrix`。简单来说，我们将视口中心定位在飞船所在的位置，并向四个方向扩展我们希望显示的一半距离。记住，这会在每一帧发生，所以我们的视口将始终跟随玩家飞船。

接下来，我们调用`glClear()`，用`onSurfaceCreated()`中设置的颜色清除屏幕。在`draw()`中我们做的最后一件事是在我们的`SpaceShip`对象上调用`draw`方法。这意味着这与我们之前的两款游戏有一个非常基本的设计变化。

我们已经提到过这一点，但在这里我们可以看到它的实际应用：每个对象将绘制自己。同时，注意我们传递了新配置的`viewportMatrix`。

输入`draw`方法的代码：

```java
private void draw() {

    // Where is the ship?
    handyPointF = gm.ship.getWorldLocation();

    // Modify the viewport matrix orthographic projection
    // based on the ship location
    orthoM(viewportMatrix, 0,
        handyPointF.x - gm.metresToShowX / 2,
        handyPointF.x + gm.metresToShowX / 2,
        handyPointF.y - gm.metresToShowY / 2,
        handyPointF.y + gm.metresToShowY / 2,
        0f, 1f);

    // Clear the screen
    glClear(GL_COLOR_BUFFER_BIT);

    // Start drawing!

    // Draw the ship
    gm.ship.draw(viewportMatrix);
}
}
```

现在，我们可以构建我们的`GameObject`超类，紧接着是它的第一个子类`SpaceShip`。我们将看到这些对象如何设法使用 OpenGL 来绘制自己。

# 构建一个对 OpenGL 友好的`GameObject`超类

让我们直接进入代码。正如我们将看到的，这个`GameObject`将与之前项目中的`GameObject`类有很多共同之处。最显著的区别将在于，这个最新的`GameObject`当然会使用 GL 程序的句柄、子类中的原始（顶点）数据以及`viewportMatrix`中的视口矩阵来绘制自己。

创建一个新类，将其命名为`GameObject`，并输入这些导入语句，再次注意其中一些是静态的：

```java
import android.graphics.PointF;
import java.nio.ByteBuffer;
import java.nio.ByteOrder;
import java.nio.FloatBuffer;
import static android.opengl.GLES20.GL_FLOAT;
import static android.opengl.GLES20.GL_LINES;
import static android.opengl.GLES20.GL_POINTS;
import static android.opengl.GLES20.GL_TRIANGLES;
import static android.opengl.GLES20.glDrawArrays;
import static android.opengl.GLES20.glEnableVertexAttribArray;
import static android.opengl.GLES20.glGetAttribLocation;
import static android.opengl.GLES20.glGetUniformLocation;
import static android.opengl.GLES20.glUniform4f;
import static android.opengl.GLES20.glUniformMatrix4fv;
import static android.opengl.GLES20.glUseProgram;
import static android.opengl.Matrix.multiplyMM;
import static android.opengl.Matrix.setIdentityM;
import static android.opengl.Matrix.setRotateM;
import static android.opengl.Matrix.translateM;
import static android.opengl.GLES20.glVertexAttribPointer;
import static com.gamecodeschool.c9asteroids.GLManager.*;
```

有很多成员变量，其中许多是自解释的并已加上注释以刷新我们的记忆，但也有一些全新的变量。

例如，我们有一个`enum`来表示我们将要创建的每种类型的`GameObject`。这样做的原因是，我们将某些对象绘制为点，一些绘制为线，一个绘制为三角形。我们使用 OpenGL 的方式在不同类型的图元之间是一致的；因此，我们将代码捆绑到这个父类中。然而，最终绘制图元的调用取决于图元的类型。我们可以使用`type`变量在`switch`语句中执行正确的`draw`方法。

我们还有一个名为`int numElements`和`numVertices`的变量，用于记录构成任何给定`GameObject`的点数。我们将在后面的子类中设置这些值。

我们还有一个浮点数数组`modelVertices`，它将存储构成一个模型的全部顶点。

在`GameObject`类中输入第一组成员变量，并查看注释以刷新您的记忆或明确各种成员最终将用于什么：

```java
public class GameObject {

    boolean isActive;

    public enum Type {SHIP, ASTEROID, BORDER, BULLET, STAR}

    private Type type;

    private static int glProgram =-1;

    // How many vertices does it take to make
    // this particular game object?
    private int numElements;
    private int numVertices;

    // To hold the coordinates of the vertices that
    // define our GameObject model
    private float[] modelVertices;

    // Which way is the object moving and how fast?
    private float xVelocity = 0f;
    private float yVelocity = 0f;
    private float speed = 0;
    private float maxSpeed = 200;

    // Where is the object centre in the game world?
    private PointF worldLocation = new PointF();
```

接下来，我们将添加另一组成员变量。首先，最值得注意的是，我们有一个名为`vertices`的`FloatBuffer`。我们知道，OpenGL 在本地代码中执行，而`FloatBuffers`是它喜欢消费数据的方式。我们将看到如何将所有顶点打包到这个`FloatBuffer`中。

我们还将使用`GLManager`类中的所有公共静态成员来帮助我们正确处理。

在 OpenGL 方面，可能第二个最有趣的新成员是我们还有另外三个浮点数数组，名为`modelMatrix`、`viewportModelMatrix`和`rotateViewportModelMatrix`。这些将帮助 OpenGL 精确地按照要求绘制`GameObject`类。我们将在到达本类的`draw`方法时详细检查它们是如何初始化和使用的。

我们还有一些成员变量，用于保存不同的角度和旋转速率。我们如何使用和更新这些变量，以便通知 OpenGL 我们对象的方向，我们很快就会看到：

```java
    // This will hold our vertex data that is
    // passed into the openGL glProgram
    // OPenGL likes FloatBuffer
    private FloatBuffer vertices;

    // For translating each point from the model (ship, asteroid etc)
    // to its game world coordinates
    private final float[] modelMatrix = new float[16];

    // Some more matrices for Open GL transformations
    float[] viewportModelMatrix = new float[16];
    float[] rotateViewportModelMatrix = new float[16];

    // Where is the GameObject facing?
    private float facingAngle = 90f;

    // How fast is it rotating?
    private float rotationRate = 0f;

    // Which direction is it heading?
    private float travellingAngle = 0f;

    // How long and wide is the GameObject?
    private float length;
    private float width;
```

现在我们实现构造函数。首先，我们检查是否之前已经编译过着色器，因为我们只需要做一次。如果我们没有，这就是`if(glProgarm == -1)`块内发生的事情。

我们调用`setGLProgram()`，然后传入`glProgram`参数调用`glUseProgram()`。我们需要做的就这么多，`GLManager`会处理其余的工作，我们的 OpenGL 程序就可以使用了。

然而，在我们继续之前，通过调用相应的方法（`glGetUniformLocation()`和`glGetAttrtibuteLocation`）来保存关键着色器变量的位置，这些位置在我们的 GL 程序中的位置。我们将在本类的`draw`方法中看到如何使用这些位置来操作着色器内的值。

最后，我们将`isActive`设置为`true`。将此方法输入到`GameObject`类中：

```java
public GameObject(){
    // Only compile shaders once
    if (glProgram == -1){
        setGLProgram();

        // tell OpenGl to use the glProgram
        glUseProgram(glProgram);

        // Now we have a glProgram we need the locations
        // of our three GLSL variables.
        // We will use these when we call draw on the object.
        uMatrixLocation = glGetUniformLocation(glProgram, U_MATRIX);
        aPositionLocation = glGetAttribLocation(glProgram, A_POSITION);
        uColorLocation = glGetUniformLocation(glProgram, U_COLOR);
    }

    // Set the object as active
    isActive = true;

}
```

现在我们有一些 getters 和 setters，包括从`AsteroidsRenderer`中的`draw`方法调用的`getWorldLocation()`，以及`setGLProgram()`。这使用`GLManager`类的静态方法`getGLProgram()`来获取我们 GL 程序的句柄。

在`GameObject`类中输入所有这些方法：

```java
public boolean isActive() {
  return isActive;
}

public void setActive(boolean isActive) {
  this.isActive = isActive;
}

public void setGLProgram(){
  glProgram = GLManager.getGLProgram();
}

public Type getType() {
  return type;
}

public void setType(Type t) {
  this.type = t;
}

public void setSize(float w, float l){
  width = w;
  length = l;

}

public PointF getWorldLocation() {
  return worldLocation;
}

public void setWorldLocation(float x, float y) {
  this.worldLocation.x = x;
  this.worldLocation.y = y;
}
```

下一个方法`setVertices()`是在准备由 OpenGL 绘制对象时的关键步骤。在我们的每个子类中，我们将构建一个浮点类型的数组来表示构成游戏对象形状的顶点。每个游戏对象在形状上显然都是不同的，但`setVertices`方法无需关注这些差异，它只需要数据。

正如我们在下一个代码块中看到的，该方法接收一个浮点数组作为参数。然后它将等于数组长度的元素数量存储在`numElements`中。请注意，元素的数量与它们表示的顶点数量不同。需要三个元素（*x*，*y*和*z*）来表示一个顶点。因此，我们可以通过将`numElements`除以`ELEMENTS_PER_VERTEX`来将正确的值存储在`numVertices`中。

现在，我们可以通过调用`allocateDirect()`并传入我们新初始化的变量以及`FLOAT_SIZE`来实际初始化我们的`ByteBuffer`。`ByteOrder.nativeOrder`方法只是检测特定系统的字节序，而`asFloatBuffer()`告诉`ByteBuffer`将要存储的数据类型。现在，我们可以通过调用`vertices.put(modelVertices)`将顶点数组存储到我们的顶点`ByteBuffer`中。这些数据现在可以传递给 OpenGL 了。

### 提示

如果你想了解更多关于字节序的信息，请查看这篇维基百科文章：

[关于字节序的维基百科文章](http://en.wikipedia.org/wiki/Endianness)

在`GameObject`类中输入`setVertices`方法：

```java
public void setVertices(float[] objectVertices){

    modelVertices = new float[objectVertices.length];
    modelVertices = objectVertices;

    // Store how many vertices and elements there is for future use
    numElements = modelVertices.length;

    numVertices = numElements/ELEMENTS_PER_VERTEX;

    // Initialize the vertices ByteBuffer object based on the
    // number of vertices in the ship design and the number of
    // bytes there are in the float type
    vertices = ByteBuffer.allocateDirect(
            numElements
            * FLOAT_SIZE)
            .order(ByteOrder.nativeOrder()).asFloatBuffer();

    // Add the ship into the ByteBuffer object
    vertices.put(modelVertices);

}
```

现在我们来看看我们是如何实际绘制我们的`ByteBuffer`的内容的。乍一看，以下代码可能看起来很复杂，但当我们讨论`ByteBuffer`中的数据性质以及 OpenGL 绘制这些数据的步骤时，我们会发现这实际上相当直接。

由于我们还没有编写第一个`GameObject`子类的代码，有一个关键点需要指出。表示游戏对象形状的顶点是基于其自身的中心为零的。

OpenGL 坐标系统的中心是**0,0**，但为了明确起见，这与我们无关，这被称为模型空间。下一张图片展示了我们即将创建的飞船，在模型空间中的表示：

![构建一个对 OpenGL 友好的 GameObject 超类](img/B04322_09_02.jpg)

这些数据包含在我们的`ByteBuffer`中。这些数据不考虑方向（飞船或小行星是否旋转），不考虑在游戏世界中的位置，并且再次提醒，它与 OpenGL 坐标系统完全无关。

因此，在我们绘制`ByteBuffer`之前，我们需要转换这些数据，或者更准确地说，我们需要准备一个合适的矩阵，并将其与数据一起传递给 OpenGL，以便 OpenGL 知道如何使用或转换数据。

我将`draw`方法分成了六个部分来讲解我们是如何做到这一点的。请注意，我们的`viewPort`矩阵在我们的`AsteroidsRenderer`类的`draw`方法中准备，该方法以飞船的位置为中心，基于我们想要显示的游戏世界比例，并作为参数传入。

首先，我们调用`glUseProgram()`并传入我们程序的句柄。然后我们将`ByteBuffer`的内部指针设置为起始位置，使用`vertices.position(0)`。

`glVertexAttributePointer`方法使用我们的`aPositionLocation`变量以及我们的`GLManager`静态常量，当然还有`vertices` `ByteBuffer`，将我们的顶点与顶点着色器中的`aPosition`变量相关联。最后，对于这段代码，我们告诉 OpenGL 启用属性数组：

```java
    public void draw(float[] viewportMatrix){

        // tell OpenGl to use the glProgram
        glUseProgram(glProgram);

        // Set vertices to the first byte
        vertices.position(0);

        glVertexAttribPointer(
              aPositionLocation,
              COMPONENTS_PER_VERTEX,
              GL_FLOAT,
              false,
              STRIDE,
              vertices);

        glEnableVertexAttribArray(aPositionLocation);
```

现在，我们将矩阵投入使用。通过调用`setIndentityM()`，我们从`modelMatrix`数组中创建一个单位矩阵。

### 注意

正如我们将要看到的，我们将使用和组合相当多的矩阵。单位矩阵作为一个起点或容器，我们可以在其上构建一个矩阵，该矩阵结合了我们需要发生的所有变换。一种非常简单但并不完全准确的方式来考虑单位矩阵是，它就像数字 1。当你乘以一个单位矩阵时，它不会对和的其它部分造成任何改变。然而，这个答案对于继续方程的下一部分是正确的。如果这让你感到烦恼，并且你想了解更多，请查看关于矩阵和单位矩阵的以下快速教程。

矩阵：

[矩阵介绍](https://www.khanacademy.org/math/precalculus/precalc-matrices/Basic_matrix_operations/v/introduction-to-the-matrix)

单位矩阵：

[单位矩阵教程](https://www.khanacademy.org/math/precalculus/precalc-matrices/zero-identity-matrix-tutorial/v/identity-matrix)

然后，我们将新的`modelMatrix`传递给`translateM`方法。在数学术语中，translate 意为移动。仔细观察传递给`translateM()`的参数。我们正在传递物体的*x*和*y*世界坐标。这就是 OpenGL 知道物体位置的方式：

```java
    // Translate model coordinates into world coordinates
    // Make an identity matrix to base our future calculations on
    // Or we will get very strange results
    setIdentityM(modelMatrix, 0);
    // Make a translation matrix

    /*
        Parameters:
        m   matrix
        mOffset index into m where the matrix starts
        x   translation factor x
        y   translation factor y
        z   translation factor z
    */
    translateM(modelMatrix, 0, worldLocation.x, worldLocation.y, 0);
```

我们知道 OpenGL 有一个矩阵可以将我们的对象转换到它的世界位置。它还有一个`ByteBuffer`类，其中包含模型空间坐标，但它如何将转换后的模型空间坐标转换为使用 OpenGL 坐标系统绘制的视口呢？

它使用了视口矩阵，该矩阵每帧都会被修改并传入这个方法。我们需要做的就是使用`multiplyMM()`将`viewportMatrix`和最近转换的`modelMatrix`相乘。这个方法创建了组合或乘积矩阵，并将结果存储在`viewportModelMatrix`中：

```java
   // Combine the model with the viewport
   // into a new matrix
   multiplyMM(viewportModelMatrix, 0, 
      viewportMatrix, 0, modelMatrix, 0);
```

我们几乎完成了矩阵的创建。OpenGL 需要对`ByteBuffer`中的顶点进行的唯一其他可能的扭曲就是根据`facingAngle`参数旋转它们。

接下来，我们创建一个适合当前物体面向角度的旋转矩阵，并将结果重新存储在`modelMatrix`中。

然后，我们将新旋转的`modelMatrix`与我们的`viewportModelMatrix`组合或相乘，并将结果存储在`rotateViewportModelMatrix`中。这是我们最终要传入 OpenGL 系统的矩阵：

```java
   /*
        Now rotate the model - just the ship model

        Parameters
        rm  returns the result
        rmOffset    index into rm where the result matrix starts
        a   angle to rotate in degrees
        x   X axis component
        y   Y axis component
        z   Z axis component
    */
    setRotateM(modelMatrix, 0, facingAngle, 0, 0, 1.0f);

    // And multiply the rotation matrix into the model-viewport 
    // matrix
    multiplyMM(rotateViewportModelMatrix, 0, 
      viewportModelMatrix, 0, modelMatrix, 0);
```

现在，我们使用`glUniformMatrix4fv()`方法传入矩阵，并在参数中使用`uMatrixLocation`变量（这是顶点着色器中与矩阵相关的变量的位置）和我们最终的矩阵。

我们还通过调用`glUniform4f()`并使用`uColorLocation`和一个 RGBT（红、绿、蓝、透明度）值来选择颜色。所有值都设置为 1.0，因此片段着色器将绘制白色。

```java
   // Give the matrix to OpenGL

    glUniformMatrix4fv(uMatrixLocation, 1, false,                                        
    rotateViewportModelMatrix, 0);

    // Assign a color to the fragment shader
    glUniform4f(uColorLocation, 1.0f, 1.0f, 1.0f, 1.0f);
```

最后，我们根据对象类型进行切换，并绘制点、线或三角形图元：

```java
   // Draw the point, lines or triangle
    switch (type){
        case SHIP:
        glDrawArrays(GL_TRIANGLES, 0, numVertices);
        break;

        case ASTEROID:
        glDrawArrays(GL_LINES, 0, numVertices);
        break;

        case BORDER:
        glDrawArrays(GL_LINES, 0, numVertices);
        break;

       case STAR:
        glDrawArrays(GL_POINTS, 0, numVertices);
        break;

        case BULLET:
        glDrawArrays(GL_POINTS, 0, numVertices);
        break;
    }

} // End draw()

}// End class
```

现在我们已经掌握了`GameObject`类的基础知识，我们可以创建一个类来表示我们的飞船并在屏幕上绘制它。

# 飞船

这个类非常简单，尽管它会随着项目的发展而演变。构造函数接收游戏世界中起点的位置。我们使用`GameObject`类的方法设置飞船的类型和世界位置，并设置宽度和高度。

我们声明并初始化一些变量，以简化模型空间坐标的初始化，然后继续初始化一个浮点数组，其中包含三个顶点，这些顶点表示我们的飞船的三角形。请注意，这些值是基于*x = 0*和*y = 0*中心的。

接下来，我们只需调用`setVertices()`，`GameObject`就会准备好`ByteBuffer`供 OpenGL 使用：

```java
public class SpaceShip extends GameObject{

  public SpaceShip(float worldLocationX, float worldLocationY){
       super();

        // Make sure we know this object is a ship
        // So the draw() method knows what type
        // of primitive to construct from the vertices

        setType(Type.SHIP);

        setWorldLocation(worldLocationX,worldLocationY);

        float width = 15;
        float length = 20;

        setSize(width, length);

        // It will be useful to have a copy of the
        // length and width/2 so we don't have to keep dividing by 2
        float halfW = width / 2;
        float halfL = length / 2;

        // Define the space ship shape
        // as a triangle from point to point
        // in anti clockwise order
        float [] shipVertices = new float[]{

               - halfW, - halfL, 0,
               halfW, - halfL, 0,
               0, 0 + halfL, 0

      };

       setVertices(shipVertices);

     }

}
```

最后，我们可以看到我们努力的成果。

# 以 60 + FPS 的速度绘制

通过三个简单的步骤，我们将能够看到我们的飞船：

+   在`GameManager`成员变量中添加一个`SpaceShip`对象：

    ```java
    private boolean playing = false;

     // Our first game object
     SpaceShip ship;

         int screenWidth;
    ```

+   在`createObjects`方法中添加对新的`SpaceShip()`的调用：

    ```java
    private void createObjects() {

      // Create our game objects
     // First the ship in the center of the map
     gm.ship = new SpaceShip(gm.mapWidth / 2, gm.mapHeight / 2);
    }
    ```

+   在`AsteroidsRenderer`的`draw`方法中，添加每一帧绘制飞船的调用：

    ```java
    // Start drawing!
    // Draw the ship
    gm.ship.draw(viewportMatrix);

    ```

运行游戏并查看输出：

![以 60 + FPS 的速度绘制](img/B04322_09_03.jpg)

视觉效果并不令人印象深刻，但在调试模式下，在老旧的三星 Galaxy S2 手机上输出到控制台时，它的运行帧数在 67 到 212 之间。

![以 60 + FPS 的速度绘制](img/B04322_09_04.jpg)

在整个项目中，我们的目标是在保持每秒 60 帧以上的情况下添加数百个对象。

### 提示

一位书籍评论者在 Nexus 5 上报告了每秒超过 1000 帧的速率！因此，如果你打算将此应用发布到 Google Play 商店，考虑采用最大帧率锁定策略以节省电池寿命将是非常值得的。

# 总结

设置绘图系统可能有些繁琐。然而，现在它已经完成，我们可以更轻松地快速生成新对象。我们只需定义类型和顶点，然后就可以轻松地绘制它们。

正是因为有了这些基础工作，下一章内容将更加视觉上令人满意。接下来，我们将创建闪烁的星星、游戏世界的边界、旋转移动的小行星、呼啸而过的子弹、以及一个 HUD（平视显示器），同时为太空飞船添加完整的控制和动作。
