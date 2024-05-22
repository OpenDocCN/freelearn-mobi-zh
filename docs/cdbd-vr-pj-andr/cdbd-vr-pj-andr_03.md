# 第三章：Cardboard Box

还记得小时候开心地玩纸板盒吗？这个项目甚至可能比那更有趣！我们的第一个 Cardboard 项目将是一个简单的场景，有一个盒子（一个几何立方体），一个三角形，还有一点用户交互。我们称之为“CardboardBox”。懂了吗？

具体来说，我们将创建一个新项目，构建一个简单的应用程序，只绘制一个三角形，然后增强该应用程序以绘制阴影的 3D 立方体，并通过在观察时突出显示立方体来说明一些用户交互。

在本章中，您将会：

+   创建一个新的 Cardboard 项目

+   向场景添加三角形对象，包括几何、简单着色器和渲染缓冲区

+   使用 3D 相机、透视和头部旋转

+   使用模型变换

+   制作和绘制立方体对象

+   添加光源和阴影

+   旋转立方体

+   添加地板

+   突出显示用户正在查看的对象

本章中的项目源自 Google Cardboard 团队提供的一个示例应用程序，名为*寻宝游戏*。最初，我们考虑让您简单地下载寻宝游戏，然后我们会在代码中引导您解释其工作原理。相反，我们决定从头开始构建一个类似的项目，并在进行过程中进行解释。这也减轻了谷歌在本书出版后更改或甚至替换该项目的可能性。

该项目的源代码可以在 Packt Publishing 网站和 GitHub 上找到，网址为[`github.com/cardbookvr/cardboardbox`](https://github.com/cardbookvr/cardboardbox)（每个主题作为单独的提交）。

Android SDK 版本对于您的成品应用程序很重要，但您的桌面环境也可以以多种方式设置。我们之前提到，我们使用 Android Studio 2.1 构建了本书中的项目。我们还使用了 Java SDK 版本 8（1.8）。对于您来说，安装这个版本很重要（您可以并排安装许多版本），以便导入项目。与任何开发环境一样，对 Java 或 Android Studio 所做的任何更改可能会在将来“破坏”导入过程，但实际的源代码应该可以编译和运行多年。

# 创建一个新项目

如果您想了解有关这些步骤的更多详细信息和解释，请参考第二章中的*创建新的 Cardboard 项目*部分，*骨架 Cardboard 项目*，并跟随那里进行：

1.  打开 Android Studio，创建一个新项目。让我们将其命名为`CardboardBox`，并针对**Android 4.4 KitKat (API 19)**使用**空活动**。

1.  将 Cardboard SDK 的`common.aar`和`core.aar`库文件作为新模块添加到项目中，使用**文件** | **新建** | **新建模块...**。

1.  将库模块设置为项目应用程序的依赖项，使用**文件** | **项目结构**。

1.  根据第二章中的说明编辑`AndroidManifest.xml`文件，*骨架 Cardboard 项目*，要小心保留此项目的`package`名称。

1.  根据第二章中的说明编辑`build.gradle`文件，*骨架 Cardboard 项目*，以便编译 SDK 22。

1.  根据第二章中的说明编辑`activity_main.xml`布局文件，*骨架 Cardboard 项目*。

1.  编辑`MainActivity` Java 类，使其`extends` `CardboardActivity`并`implement` `CardboardView.StereoRenderer`。修改类声明行如下：

```kt
public class MainActivity extends CardboardActivity implements CardboardView.StereoRenderer {
```

1.  添加接口的存根方法覆盖（使用智能实现方法或按下*Ctrl* + *I*）。

1.  在`MainActivity`类的顶部，添加以下注释作为我们将在此项目中创建的变量的占位符：

```kt
CardboardView.StereoRenderer {
   private static final String TAG = "MainActivity";

   // Scene variables
   // Model variables
   // Viewing variables
   // Rendering variables
```

1.  最后，通过以下方式编辑`onCreate()`，添加`CardboadView`实例：

```kt
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        CardboardView cardboardView = (CardboardView) findViewById(R.id.cardboard_view);
        cardboardView.setRenderer(this);
        setCardboardView(cardboardView);  
    }
```

# 你好，三角形！

让我们在场景中添加一个三角形。是的，我知道三角形甚至不是一个盒子。然而，我们将从非常简单的提示开始。三角形是所有 3D 图形的基本构件，也是 OpenGL 可以渲染的最简单的形状（即以三角形模式）。

## 引入几何图形

在继续之前，让我们谈谈几何图形。

虚拟现实在很大程度上是关于创建 3D 场景。复杂的模型被组织为具有顶点、面和网格的三维数据，形成可以按层次组装成更复杂模型的对象。目前，我们采用了一个非常简单的方法——一个由三个顶点组成的三角形，存储为一个简单的 Java 数组。

三角形由三个顶点组成（这就是为什么它被称为**三角形**！）。我们将把我们的三角形定义为顶部（0.0, 0.6），左下角（-0.5, -0.3），右下角（0.5, -0.3）。第一个顶点是三角形的最顶点，具有*X=0.0*，因此它位于中心，*Y=0.6*向上。

顶点的顺序或三角形的绕组非常重要，因为它指示了三角形的正面方向。OpenGL 驱动程序希望它以逆时针方向绕组，如下图所示：

![引入几何图形](img/B05144_03_01.jpg)

如果顶点是顺时针定义的，着色器将假定三角形朝向相反方向，远离摄像头，因此不可见且不会被渲染。这是一种优化称为**剔除**，它允许渲染管线轻松丢弃在物体背面的几何图形。也就是说，如果对摄像头不可见，甚至不要尝试绘制它。话虽如此，您可以设置各种剔除模式，选择只渲染正面、背面或两者。

请参阅[`learnopengl.com/#!Advanced-OpenGL/Face-culling`](http://learnopengl.com/#!Advanced-OpenGL/Face-culling)上的创作共用来源。

### 提示

*OpenGL 编程指南*，作者 Dave Shreiner，Graham Sellers，John M. Kessenich，Bill Licea-Kane，“按照惯例，在屏幕上顶点逆时针顺序出现的多边形被称为正面朝向”。这是由全局状态模式确定的，默认值为`GL_CCW`（[`www.opengl.org/wiki/Face_Culling`](https://www.opengl.org/wiki/Face_Culling)）。

三维点或顶点是用*x*、*y*和*z*坐标值定义的。例如，在 3D 空间中，三角形由三个顶点组成，每个顶点都有*x*、*y*和*z*值。

我们的三角形位于与屏幕平行的平面上。当我们在场景中添加 3D 视图（本章后面会介绍）时，我们需要一个*z*坐标将其放置在 3D 空间中。为了预期，我们将三角形设置在*Z=-1*平面上。OpenGL 中的默认摄像头位于原点（0,0,0），并朝着负*z*轴方向。换句话说，场景中的物体朝着正*z*轴向摄像头看。我们将三角形放置在离摄像头一单位远的地方，这样我们就可以在*Z=-1.0*处看到它。

## 三角形变量

将以下代码片段添加到`MainActivity`类的顶部：

```kt
    // Model variables
    private static final int COORDS_PER_VERTEX = 3;
    private static float triCoords[] = {
        // in counter-clockwise order
        0.0f,  0.6f, -1.0f, // top
       -0.5f, -0.3f, -1.0f, // bottom left
        0.5f, -0.3f, -1.0f  // bottom right
    };

    private final int triVertexCount = triCoords.length / COORDS_PER_VERTEX;
    // yellow-ish color
    private float triColor[] = { 0.8f, 0.6f, 0.2f, 0.0f }; 
    private FloatBuffer triVerticesBuffer;
```

我们的三角形坐标被分配给`triCoords`数组。所有顶点都在 3D 空间中，每个顶点（`COORDS_PER_VERTEX`）有三个坐标（*x*、*y*和*z*）。预先计算的`triVertexCount`变量是三角形`triCoords`数组的长度，除以`COORDS_PER_VERTEX`。我们还为我们的三角形定义了一个任意的`triColor`值，由 R、G、B 和 A 值（红色、绿色、蓝色和 alpha（透明度））组成。`triVerticesBuffer`变量将在绘制代码中使用。

对于刚接触 Java 编程的人来说，你可能也会对变量类型感到困惑。整数声明为 `int`，浮点数声明为 `float`。这里的所有变量都被声明为 `private`，这意味着它们只能在这个类定义内部可见和使用。被声明为 `static` 的变量将在类的多个实例之间共享数据。被声明为 `final` 的变量是不可变的，一旦初始化就不会改变。

## onSurfaceCreated

这个活动代码的目的是在 Android 设备显示器上绘制东西。我们通过 OpenGL 图形库来实现这一点，它会绘制到一个表面上，一个内存缓冲区，你可以通过渲染管线绘制图形。

活动创建后（`onCreate`），会创建一个表面并调用 `onSurfaceCreated`。它有几个责任，包括初始化场景和编译着色器。它还通过为顶点缓冲区分配内存、绑定纹理和初始化渲染管线句柄来准备渲染。

这是一个方法，我们将把它分成几个私有方法，接下来我们将编写这些方法：

```kt
    @Override
    public void onSurfaceCreated(EGLConfig eglConfig) {
        initializeScene();
        compileShaders();
        prepareRenderingTriangle();
    }
```

在这一点上，场景中没有什么需要初始化的：

```kt
private void initializeScene() {
}
```

让我们继续讨论着色器和渲染。

## 介绍 OpenGL ES 2.0

现在是介绍 *图形管线* 的好时机。当 Cardboard 应用在屏幕上绘制 3D 图形时，它会将渲染交给一个单独的图形处理器（GPU）。Android 和我们的 Cardboard 应用使用 OpenGL ES 2.0 标准图形库。

OpenGL 是应用程序与图形驱动程序交互的规范。你可以说它是一长串在图形硬件中执行操作的函数调用。硬件供应商编写他们的驱动程序以符合最新的规范，而一些中间件，比如 Google，在这种情况下创建了一个库，它连接到驱动程序函数，以提供你可以从任何语言中调用的方法签名（通常是 Java、C++ 或 C#）。

OpenGL ES 是 OpenGL 的移动版，也称为嵌入式系统。它遵循与 OpenGL 相同的设计模式，但其版本历史非常不同。不同版本的 OpenGL ES 甚至同一版本的不同实现都需要不同的方法来绘制 3D 图形。因此，你的代码在 OpenGL ES 1.0、2.0 和 3.0 之间可能会有很大的不同。值得庆幸的是，大部分重大变化发生在版本 1 和 2 之间，Cardboard SDK 设定为使用 2.0。`CardboardView` 接口也与普通的 `GLSurfaceView` 稍有不同。

在屏幕上绘制图形，OpenGL 需要两个基本的东西：

+   定义如何绘制形状的图形程序，或 *着色器*（有时可以互换使用）

+   定义正在绘制的数据，或 *缓冲区*

还有一些参数，用于指定变换矩阵、颜色、向量等。你可能熟悉游戏循环的概念，这是一种设置游戏环境并启动一个循环的基本模式，该循环运行一些游戏逻辑，渲染屏幕，并在半规律的时间间隔内重复，直到游戏暂停或程序退出。`CardboardView` 为我们设置了游戏循环，基本上，我们只需要实现接口方法。

关于着色器的更多信息：至少我们需要一个顶点着色器和一个片段着色器。顶点着色器负责将对象的顶点从世界空间（它们在世界中的位置）转换到屏幕空间（它们应该在屏幕上绘制的位置）。

片段着色器在形状占据的每个像素上调用（由光栅函数确定），并返回绘制的颜色。每个着色器都是一个单一的函数，伴随着一些可以用作输入的属性。

OpenGL 将一组函数（即顶点和片段）编译成一个程序。有时，整个程序被称为着色器，但这是一种俚语，假设需要多个函数或*着色器*才能完全绘制一个对象。程序及其所有参数的值有时会被称为*材质*，因为它完全描述了它绘制的表面的材质。

着色器很酷。但是，在程序设置数据缓冲区并进行大量绘制调用之前，它们不会做任何事情。

绘制调用由**顶点缓冲对象**（**VBO**）、将用于绘制的着色器、指定应用于对象的变换的参数数量、用于绘制的纹理和任何其他着色器参数组成。

VBO 是指用于描述对象形状的任何和所有数据。一个非常基本的对象（例如三角形）只需要一个顶点数组。顶点按顺序读取，每三个空间位置定义一个三角形。稍微更高级的形状使用顶点数组和索引数组，定义了以什么顺序绘制哪些顶点。使用索引缓冲区，可以重复使用多个顶点。

虽然 OpenGL 可以绘制多种形状类型（点、线、三角形和四边形），但我们假设所有形状都是三角形。这既是性能优化，也是方便之处。如果我们想要一个四边形，我们可以绘制两个三角形。如果我们想要一条线，我们可以绘制一个非常长而细的四边形。如果我们想要一个点，我们可以绘制一个微小的三角形。这样，不仅可以将 OpenGL 保留在三角形模式下，还可以以完全相同的方式处理所有 VBO。理想情况下，您希望您的渲染代码完全不受其渲染对象的影响。

总结：

+   OpenGL 图形库的目的是让我们访问 GPU 硬件，然后根据场景中的几何图形在屏幕上绘制像素。这是通过渲染管线实现的，其中数据经过一系列着色器的转换和传递。

+   着色器是一个小程序，它接受某些输入并生成相应的输出，具体取决于管线的阶段。

+   作为一个程序，着色器是用一种特殊的类似于 C 的语言编写的。源代码经过编译后可以在 Android 设备的 GPU 上高效运行。

例如，*顶点着色器*处理单个顶点的处理，输出每个顶点的变换版本。另一个步骤是对几何图形进行光栅化，之后*片段着色器*接收光栅片段并输出彩色像素。

### 注意

我们将在后面讨论 OpenGL 渲染管线，并且您可以在[`www.opengl.org/wiki/Rendering_Pipeline_Overview`](https://www.opengl.org/wiki/Rendering_Pipeline_Overview)上阅读相关内容。

您还可以在[`developer.android.com/guide/topics/graphics/opengl.html`](http://developer.android.com/guide/topics/graphics/opengl.html)上查看 Android OpenGL ES API 指南。

暂时不要太担心这个问题，让我们跟着走就好。

注意：GPU 驱动程序实际上是根据每个驱动程序来实现整个 OpenGL 库的。这意味着 NVIDIA（或在这种情况下，可能是 Qualcomm 或 ARM）的某个人编写了编译您的着色器和读取您的缓冲区的代码。OpenGL 是关于这个 API 应该如何工作的规范。在我们的情况下，这是 Android 的 GL 类的一部分。

## 简单着色器

现在，我们将在`MainActivity`类的末尾添加以下函数。

```kt
   /**
     * Utility method for compiling a OpenGL shader.
     *
     * @param type - Vertex or fragment shader type.
     * @param resId - int containing the resource ID of the shader code file.
     * @return - Returns an id for the shader.
     */
    private int loadShader(int type, int resId){
        String code = readRawTextFile(resId);
        int shader = GLES20.glCreateShader(type);

        // add the source code to the shader and compile it
        GLES20.glShaderSource(shader, code);
        GLES20.glCompileShader(shader);

        return shader;
    }

    /**
     * Converts a raw text file into a string.
     *
     * @param resId The resource ID of the raw text file about to be turned into a shader.
     * @return The content of the text file, or null in case of error.
     */
    private String readRawTextFile(int resId) {
        InputStream inputStream = getResources().openRawResource(resId);
        try {
            BufferedReader reader = new BufferedReader(new InputStreamReader(inputStream));
            StringBuilder sb = new StringBuilder();
            String line;
            while ((line = reader.readLine()) != null) {
                sb.append(line).append("\n");
            }
            reader.close();
            return sb.toString();
        } catch (IOException e) {
            e.printStackTrace();
        }
        return null;
    }
```

我们将调用`loadShader`来加载一个着色器程序（通过`readRawTextFile`）并对其进行编译。这段代码在其他项目中也会很有用。

现在，我们将在`res/raw/simple_vertex.shader`和`res/raw/simple_fragment.shader`文件中编写一些简单的着色器。

在 Android Studio 左侧的**项目文件**层次结构视图中，定位`app/res/`资源文件夹，右键单击它，转到**新建** | **Android 资源目录**。在**新资源目录**对话框中，从**资源类型：**中选择**Raw**，然后单击**确定**。

右键单击新的`raw`文件夹，转到**新建** | **文件**，并将其命名为`simple_vertex.shader`。添加以下代码：

```kt
attribute vec4 a_Position;
void main() {
    gl_Position = a_Position;
}
```

同样，对于片段着色器，右键单击`raw`文件夹，转到**新建** | **文件**，并将其命名为`simple_fragment.shader`。添加以下代码：

```kt
precision mediump float;
uniform vec4 u_Color;
void main() {
    gl_FragColor = u_Color;
}
```

基本上，这些是恒等函数。顶点着色器通过给定的顶点，片段着色器通过给定的颜色。

注意我们声明的参数的名称：`simple_vertex`中的属性名为`a_Position`，`simple_fragment`中的统一变量名为`u_Color`。我们将从`MainActivity onSurfaceCreated`方法中设置这些属性。属性是每个顶点的属性，当我们为它们分配缓冲区时，它们必须都是相等长度的数组。您将遇到的其他属性是顶点法线、纹理坐标和顶点颜色。统一变量将用于指定适用于整个材质的信息，例如在这种情况下，应用于整个表面的固体颜色。

另外，注意`gl_FragColor`和`gl_Position`变量是 OpenGL 正在寻找你设置的内置变量名称。把它们想象成着色器函数的返回值。还有其他内置的输出变量，我们稍后会看到。

## compileShaders 方法

现在我们准备实现`onSurfaceCreated`调用的`compileShaders`方法。

在`MainActivity`的顶部添加以下变量：

```kt
    // Rendering variables
    private int simpleVertexShader;
    private int simpleFragmentShader;
```

实现`compileShaders`，如下：

```kt
    private void compileShaders() {
        simpleVertexShader = loadShader(GLES20.GL_VERTEX_SHADER, R.raw.simple_vertex);
        simpleFragmentShader = loadShader(GLES20.GL_FRAGMENT_SHADER, R.raw.simple_fragment);
    }
```

## prepareRenderingTriangle 方法

`onSurfaceCreated`方法通过为顶点缓冲区分配内存，创建 OpenGL 程序和初始化渲染管道句柄来准备渲染。现在我们将为我们的三角形形状执行此操作。

在`MainActivity`的顶部添加以下变量：

```kt
    // Rendering variables
    private int triProgram;
    private int triPositionParam;
    private int triColorParam;
```

以下是函数的框架：

```kt
    private void prepareRenderingTriangle() {
        // Allocate buffers
        // Create GL program
        // Get shader params
    }
```

我们需要准备一些内存缓冲区，当每帧被渲染时，它们将被传递给 OpenGL。这是我们的三角形和简单着色器的第一次尝试；现在我们只需要一个顶点缓冲区：

```kt
        // Allocate buffers
        // initialize vertex byte buffer for shape coordinates (4 bytes per float)
        ByteBuffer bb = ByteBuffer.allocateDirect(triCoords.length * 4);
        // use the device hardware's native byte order
        bb.order(ByteOrder.nativeOrder());

        // create a floating point buffer from the ByteBuffer
        triVerticesBuffer = bb.asFloatBuffer();
        // add the coordinates to the FloatBuffer
        triVerticesBuffer.put(triCoords);
        // set the buffer to read the first coordinate
        triVerticesBuffer.position(0);
```

这五行代码导致了`triVerticesBuffer`值的设置，如下所示：

+   分配一个足够大的`ByteBuffer`，以容纳我们的三角形坐标值。

+   二进制数据被排列以匹配硬件的本机字节顺序

+   为浮点格式化缓冲区，并将其分配给我们的`FloatBuffer`顶点缓冲区

+   三角形数据被放入其中，然后我们将缓冲区光标位置重置到开头

接下来，我们构建 OpenGL ES 程序可执行文件。使用`glCreateProgram`创建一个空的 OpenGL ES 程序，并将其 ID 分配为`triProgram`。这个 ID 也将在其他方法中使用。我们将任何着色器附加到程序中，然后使用`glLinkProgram`构建可执行文件：

```kt
        // Create GL program
        // create empty OpenGL ES Program
        triProgram = GLES20.glCreateProgram();
        // add the vertex shader to program
        GLES20.glAttachShader(triProgram, simpleVertexShader);
        // add the fragment shader to program
        GLES20.glAttachShader(triProgram, simpleFragmentShader);
        // build OpenGL ES program executable
        GLES20.glLinkProgram(triProgram);
        // set program as current
        GLES20.glUseProgram(triProgram);
```

最后，我们获得了渲染管道的句柄。调用`glGetAttribLocation`的`a_Position`检索顶点缓冲区参数的位置，`glEnableVertexAttribArray`允许访问它，并调用`glGetUniformLocation`的`u_Color`检索颜色组件的位置。一旦我们到达`onDrawEye`，我们会很高兴我们这样做了：

```kt
        // Get shader params
        // get handle to vertex shader's a_Position member
        triPositionParam = GLES20.glGetAttribLocation(triProgram, "a_Position");
        // enable a handle to the triangle vertices
        GLES20.glEnableVertexAttribArray(triPositionParam);
        // get handle to fragment shader's u_Color member
        triColorParam = GLES20.glGetUniformLocation(triProgram, "u_Color");
```

因此，我们在这个函数中隔离了准备绘制三角形模型所需的代码。首先，它为顶点设置了缓冲区。然后，它创建了一个 GL 程序，附加了它将使用的着色器。然后，我们获得了在着色器中使用的参数的句柄，用于绘制。

## onDrawEye

*准备，设置和开始！* 如果您认为我们迄今为止所写的内容是“准备就绪”部分，那么现在我们要做“开始”部分！ 也就是说，应用程序启动并创建活动，调用`onCreate`。 创建表面并调用`onSurfaceCreated`来设置缓冲区和着色器。 现在，随着应用程序的运行，每帧都会更新显示。 开始吧！

`CardboardView.StereoRenderer`接口委托这些方法。 我们可以处理`onNewFrame`（稍后会处理）。 现在，我们只需实现`onDrawEye`方法，该方法将从眼睛的角度绘制内容。 此方法将被调用两次，每只眼睛一次。

现在，`onDrawEye`所需要做的就是渲染我们可爱的三角形。 尽管如此，我们将其拆分为一个单独的函数（稍后会有意义）：

```kt
    @Override
    public void onDrawEye(Eye eye) {
        drawTriangle();
    }

    private void drawTriangle() {
        // Add program to OpenGL ES environment
        GLES20.glUseProgram(triProgram);

        // Prepare the coordinate data
        GLES20.glVertexAttribPointer(triPositionParam, COORDS_PER_VERTEX,
                GLES20.GL_FLOAT, false, 0, triVerticesBuffer);

        // Set color for drawing
        GLES20.glUniform4fv(triColorParam, 1, triColor, 0);

        // Draw the model
        GLES20.glDrawArrays(GLES20.GL_TRIANGLES, 0, triVertexCount);
    }
```

我们需要通过调用`glUseProgram`来指定我们使用的着色器程序。 调用`glVertexAttribPointer`将我们的顶点缓冲区设置到管道中。 我们还使用`glUniform4fv`来设置颜色（`4fv`指的是我们的统一变量是一个具有四个浮点数的向量）。 然后，我们使用`glDrawArrays`来实际绘制。

## 构建和运行

就是这样。 *耶哈！* 这并不那么糟糕，是吧？ 实际上，如果您熟悉 Android 开发和 OpenGL，您可能已经轻松完成了这一步。

让我们构建并运行它。 转到**运行** | **运行'app'**，或者只需使用工具栏上的绿色三角形**运行**图标。

Gradle 将执行其构建操作。 选择 Android Studio 窗口底部的**Gradle 控制台**选项卡以查看 Gradle 构建消息。 然后，假设一切顺利，APK 文件将安装在您连接的手机上（连接并打开了，对吧？）。 选择底部的**运行**选项卡以查看上传和启动消息。

这就是它显示的内容：

![构建和运行](img/B05144_03_02.jpg)

实际上，它看起来有点像万圣节南瓜雕刻！ *阴森*。 但在 VR 中，您只会看到一个单独的三角形。

请注意，虽然三角形的顶点坐标定义了直线边缘，但`CardboardView`以桶形畸变呈现它，以补偿头盔中的透镜光学。 此外，左图像与右图像不同，每只眼睛一个。 当您将手机插入 Google Cardboard 头盔时，左右立体视图将显示为一个三角形，浮在空间中，边缘笔直。

太棒了！ 我们刚刚从头开始为 Android 构建了一个简单的 Cardboard 应用程序。 与任何 Android 应用程序一样，需要定义许多不同的部分才能使基本功能正常运行，包括`AndroidManifest.xml`，`activity_main.xml`和`MainActivity.java`文件。

希望一切都按计划进行。 像一个好的程序员一样，您可能在进行增量更改后构建和运行应用程序，以解决语法错误和未处理的异常。 稍后，我们将调用 GLError 函数来检查来自 OpenGL 的错误信息。 一如既往，要密切关注 logcat 中的错误（尝试过滤正在运行的应用程序）和变量名称。 您的着色器可能存在语法错误，导致编译失败，或者在尝试访问句柄时，属性/统一名称可能存在拼写错误。 这些问题不会导致任何编译时错误（着色器在运行时编译），并且您的应用程序将运行，但可能不会渲染任何内容。

# 3D 相机，透视和头部旋转

尽管这很棒（*哈哈*），但我们的应用有点无聊，不太像 Cardboard。 具体来说，它是立体的（双视图）并具有透镜畸变，但它还不是 3D 透视视图，也不会随着您的头部移动。 我们现在要修复这个问题。

## 欢迎来到矩阵

在谈论为虚拟现实开发时，我们不能不谈论用于 3D 计算机图形的矩阵数学。

什么是矩阵？答案就在那里，Neo，它正在寻找你，如果你愿意，它会找到你。没错，是时候了解矩阵了。一切都将不同。你的视角即将改变。

我们正在构建一个三维场景。空间中的每个位置由 X、Y 和 Z 坐标描述。场景中的物体可以由 X、Y 和 Z 顶点构成。通过移动、缩放和/或旋转其顶点，可以对物体进行变换。这种变换可以用一个包含 16 个浮点值的矩阵来数学表示（每行四个浮点数）。

矩阵可以通过相乘来组合。例如，如果你有一个表示对象缩放（比例）的矩阵和另一个用于重新定位（平移）的矩阵，那么你可以通过将两者相乘来创建第三个矩阵，表示缩放和重新定位。但是，你不能只使用原始的`*`运算符。另外，需要注意的是，与简单的标量乘法不同，矩阵乘法不是可交换的。换句话说，我们知道*a * b = b * a*。然而，对于矩阵 A 和 B，*AB ≠ BA*！Matrix Android 类库提供了执行矩阵运算的函数。以下是一个例子：

```kt
// allocate the matrix arrays
float scale[] = new float[16];
float translate[] = new float[16];
float scaleAndTranslate[] = new float[16];

// initialize to Identity
Matrix.setIdentityM(scale, 0);
Matrix.setIdentityM(translate, 0);

// scale by 2, move by 5 in Z
Matrix.scaleM(scale, 0, 2.0, 2.0, 2.0);
Matrix.translateM(translate, 0, 0, 0.0, 0.0, 5.0);

// combine them with a matrix multiply
Matrix.multipyMM(scaleAndTranslate, 0, translate, 0, scale, 0);
```

需要注意的是，由于矩阵乘法的工作方式，将向量乘以结果矩阵将产生与首先将其乘以缩放矩阵（右侧）相同的效果，然后将其乘以平移矩阵（左侧）。这与你可能期望的相反。

### 注意

Matrix API 的文档可以在[`developer.android.com/reference/android/opengl/Matrix.html`](http://developer.android.com/reference/android/opengl/Matrix.html)找到。

这些矩阵的东西将被大量使用。值得在这里提到的一点是精度损失。如果你反复缩放和平移组合矩阵，可能会出现与实际值的“漂移”，因为浮点计算由于四舍五入而丢失信息。这不仅是计算机图形的问题，也是银行和比特币挖掘的问题！（还记得电影《办公空间》吗？）

这种矩阵数学的一个基本用途是立即将场景转换为用户视角的屏幕图像（投影）。

在 Cardboard 虚拟现实应用中，为了从特定视角渲染场景，我们考虑一个朝向特定方向的摄像机。摄像机像任何其他物体一样具有 X、Y 和 Z 位置，并旋转到其视角方向。在虚拟现实中，当你转动头部时，Cardboard SDK 读取手机中的运动传感器，确定当前的头部姿势（视角和角度），并给你的应用程序相应的变换矩阵。

事实上，在虚拟现实中，对于每一帧，我们渲染两个稍微不同的透视视图：每只眼睛一个，偏移了实际的眼睛间距（瞳距）。

此外，在虚拟现实中，我们希望使用透视投影（而不是等距投影）来渲染场景，以便靠近你的物体比远处的物体更大。这也可以用 4x4 矩阵来表示。

我们可以将这些变换组合起来，将它们相乘以获得`modelViewProjection`矩阵：

```kt
modelViewProjection = modelTransform X camera  X  eyeView  X  perspectiveProjection
```

完整的`modelViewProjection`（MVP）变换矩阵是任何模型变换（例如，在场景中缩放或定位模型）与摄像机视角和透视投影的组合。

当 OpenGL 开始绘制一个对象时，顶点着色器可以使用`modelViewProjection`矩阵来渲染几何图形。整个场景从用户的视角绘制，朝向他的头部指向，每只眼睛都有透视投影，通过你的 Cardboard 观看器呈现立体效果。虚拟现实 MVP FTW！

## MVP 顶点着色器

我们之前编写的超级简单的顶点着色器并不会变换每个顶点；它只是将它传递到管道的下一步。现在，我们希望它能够具有 3D 感知能力，并使用我们的`modelViewProjection`（MVP）变换矩阵。创建一个着色器来处理它。

在层次结构视图中，右键单击`app/res/raw`文件夹，转到**新建** | **文件**，输入名称`mvp_vertex.shader`，然后单击**确定**。编写以下代码：

```kt
uniform mat4 u_MVP;
attribute vec4 a_Position;
void main() {
   gl_Position = u_MVP * a_Position;
}
```

这个着色器几乎和`simple_vertex`一样，但是通过`u_MVP`矩阵来变换每个顶点。（请注意，虽然在 Java 中用`*`来乘矩阵和向量是不起作用的，但在着色器代码中是可以的！）

将`compleShaders`函数中的着色器资源替换为使用`R.raw.mvp_vertex`：

```kt
simpleVertexShader = loadShader(GLES20.GL_VERTEX_SHADER, R.raw.mvp_vertex)
```

## 设置透视视图矩阵

为了将摄像机和视图添加到我们的场景中，我们定义了一些变量。在`MainActivity.java`文件中，在`MainActivity`类的开头添加以下代码：

```kt
// Viewing variables
private static final float Z_NEAR = 0.1f;
private static final float Z_FAR = 100.0f;
private static final float CAMERA_Z = 0.01f;

private float[] camera;
private float[] view;
private float[] modelViewProjection;

// Rendering variables
private int triMVPMatrixParam;
```

`Z_NEAR`和`Z_FAR`常量定义了后面用于计算摄像机眼睛的透视投影的深度平面。`CAMERA_Z`将是摄像机的位置（例如，在 X=0.0，Y=0.0 和 Z=0.01 处）。

`triMVPMatrixParam`变量将用于在我们改进的着色器中设置模型变换矩阵。

`camera`、`view`和`modelViewProjection`矩阵将是 4x4 矩阵（16 个浮点数的数组），用于透视计算。

在`onCreate`中，我们初始化了`camera`、`view`和`modelViewProjection`矩阵：

```kt
    protected void onCreate(Bundle savedInstanceState) {
        //...

        camera = new float[16];
        view = new float[16];
        modelViewProjection = new float[16];
    }
```

在`prepareRenderingTriangle`中，我们初始化了`triMVPMatrixParam`变量：

```kt
// get handle to shape's transformation matrix
triMVPMatrixParam = GLES20.glGetUniformLocation(triProgram, "u_MVP");
```

### 提示

OpenGL 中的默认摄像机位于原点（0,0,0），并朝向负*Z*轴。换句话说，场景中的物体朝着摄像机的正*Z*轴。为了将它们放在摄像机前面，给它们一个带有一些负 Z 值的位置。

在 3D 图形世界中有一个长期存在的（且毫无意义的）关于哪个轴是上的争论。我们可以在某种程度上都同意*X*轴是左右移动的，但*Y*轴是上下移动的，还是*Z*轴是呢？许多软件选择*Z*作为上下方向，并将*Y*定义为指向屏幕内外。另一方面，Cardboard SDK、Unity、Maya 和许多其他软件选择了相反的方式。如果你把坐标平面想象成在图纸上绘制，那么这取决于你把纸放在哪里。如果你把图形想象成从上面往下看，或者在白板上绘制，那么*Y*就是垂直轴。如果图形放在你面前的桌子上，那么*缺失的* *Z*轴就是垂直的，指向上下。无论如何，Cardboard SDK，因此本书中的项目，将 Z 视为*前后*轴。

## 透视渲染

现在，设置好了，我们现在可以处理每一帧重新绘制屏幕的工作。

首先，设置摄像机位置。它可以像在`onCreate`中那样定义一次。但是，在 VR 应用程序中，场景中的摄像机位置通常会发生变化，因此我们需要在每一帧中重置它。

在新的一帧开始时，首先要做的是重置摄像机矩阵，使其指向一个通用的正面方向。定义`onNewFrame`方法如下：

```kt
    @Override
    public void onNewFrame(HeadTransform headTransform) {
        // Build the camera matrix and apply it to the ModelView.
        Matrix.setLookAtM(camera, 0, 0.0f, 0.0f, CAMERA_Z, 0.0f, 0.0f, 0.0f, 0.0f, 1.0f, 0.0f);
    }
```

### 提示

注意，当你写`Matrix`时，Android Studio 会想要自动导入包。确保你选择的导入是`android.opengl.Matrix`，而不是其他矩阵库，比如`android.graphic.Matrix`。

现在，当需要从每只眼睛的视角绘制场景时，我们会计算透视视图矩阵。修改`onDrawEye`如下：

```kt
    public void onDrawEye(Eye eye) {
        GLES20.glEnable(GLES20.GL_DEPTH_TEST);
        GLES20.glClear(GLES20.GL_COLOR_BUFFER_BIT | GLES20.GL_DEPTH_BUFFER_BIT);

        // Apply the eye transformation to the camera
        Matrix.multiplyMM(view, 0, eye.getEyeView(), 0, camera, 0);

        // Get the perspective transformation
        float[] perspective = eye.getPerspective(Z_NEAR, Z_FAR);

        // Apply perspective transformation to the view, and draw
        Matrix.multiplyMM(modelViewProjection, 0, perspective, 0, view, 0);

        drawTriangle();
    }
```

我们添加的前两行重置了 OpenGL 深度缓冲区。当渲染 3D 场景时，除了每个像素的颜色外，OpenGL 还会跟踪占据该像素的对象与眼睛的距离。如果为另一个对象渲染相同的像素，深度缓冲区将知道它是否应该可见（更近）或忽略（更远）。 （或者，也许颜色以某种方式组合在一起，例如透明度）。我们在渲染每只眼睛的任何几何图形之前清除缓冲区。实际上，也清除了屏幕上实际看到的颜色缓冲区。否则，在这种情况下，您最终会用纯色填满整个屏幕。

现在，让我们继续进行查看转换。`onDrawEye`接收当前的`Eye`对象，该对象描述了眼睛的立体渲染细节。特别是，`eye.getEyeView()`方法返回一个包括头部跟踪旋转、位置移动和瞳距移动的变换矩阵。换句话说，眼睛在场景中的位置以及它所看的方向。尽管 Cardboard 不提供位置跟踪，但眼睛的位置会发生变化，以模拟虚拟头部。您的眼睛不会围绕中心轴旋转，而是您的头部围绕颈部旋转，这是眼睛的一定距离。因此，当 Cardboard SDK 检测到方向变化时，两个虚拟摄像头会在场景中移动，就好像它们是实际头部中的实际眼睛一样。

我们需要一个代表该眼睛位置的摄像机透视视图的变换。如前所述，这是如下计算的：

```kt
modelViewProjection = modelTransform  X  camera  X  eyeView  X  perspectiveProjection
```

我们将`camera`乘以眼睛视图变换（`getEyeView`），然后将结果乘以透视投影变换（`getPerspective`）。目前，我们不对三角形模型本身进行变换，而是将`modelTransform`矩阵排除在外。

结果（`modelViewProjection`）被传递给 OpenGL，供渲染管线中的着色器使用（通过`glUniformMatrix4fv`）。然后，我们绘制我们的东西（通过之前写的`glDrawArrays`）。

现在，我们需要将视图矩阵传递给着色器程序。在`drawTriangle`方法中，添加如下内容：

```kt
    private void drawTriangle() {
        // Add program to OpenGL ES environment
        GLES20.glUseProgram(triProgram);

        // Pass the MVP transformation to the shader
        GLES20.glUniformMatrix4fv(triMVPMatrixParam, 1, false, modelViewProjection, 0);

        // . . .
```

## 构建和运行

让我们构建并运行它。转到**运行** | **运行'app'**，或者直接使用工具栏上的绿色三角形**运行**图标。现在，移动手机将改变与您的视图方向同步的显示。将手机插入 Google Cardboard 查看器中，就像 VR 一样（*有点像*）。

请注意，如果您的手机在应用程序启动时平放在桌子上，则我们场景中的摄像头将面向三角形的正下方而不是向前。更糟糕的是，当您拿起手机时，中性方向可能不会正对着您的前方。因此，每次在本书中运行应用程序时，先拿起手机，这样您就可以在 VR 中向前看，或者将手机支撑在位置上（我个人使用的是 Gekkopod，可在[`gekkopod.com/`](http://gekkopod.com/)上购买）。

另外，请确保您的手机在**设置**对话框中未设置为**锁定竖屏**。

# 重新定位三角形

我们的矩阵技术确实让我们走得更远了。

我想把三角形移到一边。我们将通过设置另一个变换矩阵来实现这一点，然后在绘制时将其用于模型。

添加两个名为`triTransform`和`triView`的新矩阵：

```kt
    // Model variables
    private float[] triTransform;

    // Viewing variables
    private float[] triView;
```

在`onCreate`中初始化它们：

```kt
        triTransform = new float[16];
        triView = new float[16];
```

让我们在`initializeScene`方法中设置定位三角形的模型矩阵（由`onSurfaceCreated`调用）。我们将其在 X 轴上偏移 5 个单位，并在 Z 轴上向后偏移 5 个单位。在`initializeScene`中添加以下代码：

```kt
       // Position the triangle
        Matrix.setIdentityM(triTransform, 0);
        Matrix.translateM(triTransform, 0, 5, 0, -5);
```

最后，我们使用模型矩阵在`onDrawEye`中构建`modelViewProjection`矩阵。修改`onDrawEye`如下：

```kt
    public void onDrawEye(Eye eye) {
        ...
        // Apply perspective transformation to the view, and draw
        Matrix.multiplyMM(triView, 0, view, 0, triTransform, 0);
        Matrix.multiplyMM(modelViewProjection, 0, perspective, 0, triView, 0);
        drawTriangle();
    }
```

构建并运行它。现在，您将看到三角形离得更远，偏向一侧。

### 注意

再次总结一下：`modelViewProjection`矩阵是三角形位置变换（`triTransform`）、摄像机位置和方向（`camera`）、基于手机运动传感器的`CardboardView`当前眼睛的视点（`eye.getEyeView`）以及`perspective`投影的组合。这个 MVP 矩阵被传递给顶点着色器，以确定在屏幕上绘制三角形时的实际位置。

# 你好，立方体！

在 3D 空间中漂浮的平面三角形可能很惊人，但与我们接下来要做的事情相比，简直不值一提：一个 3D 立方体！

## 立方体模型数据

为了保持示例简单，三角形只有三个顶点，声明在`MainActivity`类中。现在，我们将引入更复杂的几何形状。我们将把它放在一个名为`Cube`的类中。

好吧，它只是由八个不同的顶点组成的立方体，形成了六个面，对吧？

好吧，GPU 更喜欢渲染三角形而不是四边形，因此将每个面细分为两个三角形；总共有 12 个三角形。要单独定义每个三角形，总共需要 36 个顶点，带有适当的绕组方向，定义我们的模型，如`CUBE_COORDS`中所示。为什么不只定义八个顶点并重用它们？我们稍后会告诉你如何做。

### 注意

请记住，我们始终需要小心顶点的绕组顺序（逆时针），以便每个三角形的可见面朝外。

在 Android Studio 中，在左侧的 Android 项目层次结构窗格中，找到您的 Java 代码文件夹（例如`com.cardbookvr.cardboardbox`）。右键单击它，然后转到**新建** | **Java 类**。然后，设置**名称：Cube**，然后单击**确定**。然后，编辑文件，如下所示（请记住，本书项目的代码可以从出版商网站和书籍的公共 GitHub 存储库中下载）：

```kt
package com.cardbookvr.cardboardbox;

public class Cube {

    public static final float[] CUBE_COORDS = new float[] {
        // Front face
        -1.0f, 1.0f, 1.0f,
        -1.0f, -1.0f, 1.0f,
        1.0f, 1.0f, 1.0f,
        -1.0f, -1.0f, 1.0f,
        1.0f, -1.0f, 1.0f,
        1.0f, 1.0f, 1.0f,

        // Right face
        1.0f, 1.0f, 1.0f,
        1.0f, -1.0f, 1.0f,
        1.0f, 1.0f, -1.0f,
        1.0f, -1.0f, 1.0f,
        1.0f, -1.0f, -1.0f,
        1.0f, 1.0f, -1.0f,

        // Back face
        1.0f, 1.0f, -1.0f,
        1.0f, -1.0f, -1.0f,
        -1.0f, 1.0f, -1.0f,
        1.0f, -1.0f, -1.0f,
        -1.0f, -1.0f, -1.0f,
        -1.0f, 1.0f, -1.0f,

        // Left face
        -1.0f, 1.0f, -1.0f,
        -1.0f, -1.0f, -1.0f,
        -1.0f, 1.0f, 1.0f,
        -1.0f, -1.0f, -1.0f,
        -1.0f, -1.0f, 1.0f,
        -1.0f, 1.0f, 1.0f,

        // Top face
        -1.0f, 1.0f, -1.0f,
        -1.0f, 1.0f, 1.0f,
        1.0f, 1.0f, -1.0f,
        -1.0f, 1.0f, 1.0f,
        1.0f, 1.0f, 1.0f,
        1.0f, 1.0f, -1.0f,

        // Bottom face
        1.0f, -1.0f, -1.0f,
        1.0f, -1.0f, 1.0f,
        -1.0f, -1.0f, -1.0f,
        1.0f, -1.0f, 1.0f,
        -1.0f, -1.0f, 1.0f,
        -1.0f, -1.0f, -1.0f,
    };
}
```

## 立方体代码

返回`MainActivity`文件，我们将只是复制/粘贴/编辑三角形代码，并将其重用于立方体。显然，这并不理想，一旦我们看到一个好的模式，我们可以将其中一些抽象出来成为可重用的方法。此外，我们将使用与三角形相同的着色器，然后在下一节中，我们将用更好的光照模型替换它们。也就是说，我们将实现光照或 2D 艺术家可能称之为**着色**的东西，这是我们到目前为止还没有做的。

与三角形一样，我们声明了一堆我们将需要的变量。顶点数显然应该来自新的`Cube.CUBE_COORDS`数组：

```kt
    // Model variables
    private static float cubeCoords[] = Cube.CUBE_COORDS;
    private final int cubeVertexCount = cubeCoords.length / COORDS_PER_VERTEX;
    private float cubeColor[] = { 0.8f, 0.6f, 0.2f, 0.0f }; // yellow-ish
    private float[] cubeTransform;
    private float cubeDistance = 5f;

    // Viewing variables
    private float[] cubeView;

    // Rendering variables
    private FloatBuffer cubeVerticesBuffer;
    private int cubeProgram;
    private int cubePositionParam;
    private int cubeColorParam;
    private int cubeMVPMatrixParam;
```

将以下代码添加到`onCreate`中：

```kt
        cubeTransform = new float[16];
        cubeView = new float[16];
```

将以下代码添加到`onSurfaceCreated`中：

```kt
        prepareRenderingCube();
```

编写`prepareRenderingCube`方法，如下所示：

```kt
private void prepareRenderingCube() {
        // Allocate buffers
        ByteBuffer bb = ByteBuffer.allocateDirect(cubeCoords.length * 4);
        bb.order(ByteOrder.nativeOrder());
        cubeVerticesBuffer = bb.asFloatBuffer();
        cubeVerticesBuffer.put(cubeCoords);
        cubeVerticesBuffer.position(0);

        // Create GL program
        cubeProgram = GLES20.glCreateProgram();
        GLES20.glAttachShader(cubeProgram, simpleVertexShader);
        GLES20.glAttachShader(cubeProgram, simpleFragmentShader);
        GLES20.glLinkProgram(cubeProgram);
        GLES20.glUseProgram(cubeProgram);

        // Get shader params
        cubePositionParam = GLES20.glGetAttribLocation(cubeProgram, "a_Position");
        cubeColorParam = GLES20.glGetUniformLocation(cubeProgram, "u_Color");
        cubeMVPMatrixParam = GLES20.glGetUniformLocation(cubeProgram, "u_MVP");

        // Enable arrays
        GLES20.glEnableVertexAttribArray(cubePositionParam);
    }
```

我们将把立方体定位在 5 个单位之外，并在对角轴（1,1,0）上旋转 30 度。没有旋转，我们只会看到正面的正方形。将以下代码添加到`initializeScene`中：

```kt
        // Rotate and position the cube
        Matrix.setIdentityM(cubeTransform, 0);
        Matrix.translateM(cubeTransform, 0, 0, 0, -cubeDistance);
        Matrix.rotateM(cubeTransform, 0, 30, 1, 1, 0);
```

将以下代码添加到`onDrawEye`中以计算 MVP 矩阵，包括`cubeTransform`矩阵，然后绘制立方体：

```kt
        Matrix.multiplyMM(cubeView, 0, view, 0, cubeTransform, 0);
        Matrix.multiplyMM(modelViewProjection, 0, perspective, 0, cubeView, 0);
        drawCube();
```

编写`drawCube`方法，它与`drawTri`方法非常相似，如下所示：

```kt
    private void drawCube() {
        GLES20.glUseProgram(cubeProgram);
        GLES20.glUniformMatrix4fv(cubeMVPMatrixParam, 1, false, modelViewProjection, 0);
        GLES20.glVertexAttribPointer(cubePositionParam, COORDS_PER_VERTEX,
                GLES20.GL_FLOAT, false, 0, cubeVerticesBuffer);
        GLES20.glUniform4fv(cubeColorParam, 1, cubeColor, 0);
        GLES20.glDrawArrays(GLES20.GL_TRIANGLES, 0, cubeVertexCount);
    }
```

构建并运行它。现在您将看到立方体的 3D 视图，如下截图所示。它需要着色。

![Cube code](img/B05144_03_03.jpg)

# 光照和着色

我们需要在场景中引入光源并提供一个将使用它的着色器。为此，立方体需要额外的数据，定义每个顶点的法向量和颜色。

### 注意

顶点颜色并不总是需要用于着色，但在我们的情况下，渐变非常微妙，不同颜色的面将帮助您区分立方体的边缘。我们还将在顶点着色器中进行着色计算，这是一种更快的方法（顶点比光栅像素少），但对于球体等光滑对象效果不佳。要进行顶点光照，您需要在管道中使用顶点颜色，因此对这些颜色做点什么也是有意义的。在这种情况下，我们选择立方体的每个面使用不同的颜色。在本书的后面，您将看到像素级光照的示例以及它带来的差异。

现在我们将构建应用程序来处理我们的光照立方体。我们将通过执行以下步骤来完成：

+   编写并编译用于光照的新着色器

+   生成和定义立方体顶点法线矢量和颜色

+   为渲染分配和设置数据缓冲区

+   定义和设置用于渲染的光源

+   生成和设置用于渲染的变换矩阵

## 添加着色器

让我们编写一个增强的顶点着色器，可以使用模型的光源和顶点法线。

在项目层次结构中的`app/res/raw`文件夹上右键单击，转到**新建** | **文件**，并命名为`light_vertex.shader`。添加以下代码：

```kt
uniform mat4 u_MVP;
uniform mat4 u_MVMatrix;
uniform vec3 u_LightPos;

attribute vec4 a_Position;
attribute vec4 a_Color;
attribute vec3 a_Normal;

const float ONE = 1.0;
const float COEFF = 0.00001;

varying vec4 v_Color;

void main() {
   vec3 modelViewVertex = vec3(u_MVMatrix * a_Position);
   vec3 modelViewNormal = vec3(u_MVMatrix * vec4(a_Normal, 0.0));

   float distance = length(u_LightPos - modelViewVertex);
   vec3 lightVector = normalize(u_LightPos - modelViewVertex);
   float diffuse = max(dot(modelViewNormal, lightVector), 0.5);

   diffuse = diffuse * (ONE / (ONE + (COEFF * distance * distance)));
   v_Color = a_Color * diffuse;
   gl_Position = u_MVP * a_Position;
}
```

不要详细介绍编写光照着色器的细节，您可以看到顶点颜色是根据与光线和表面之间的角度以及光源与顶点之间的距离相关的公式计算的。请注意，我们还引入了`ModelView`矩阵以及 MVP 矩阵。这意味着您需要访问流程的两个步骤，并且在完成后不能覆盖/丢弃 MV 矩阵。

请注意，我们使用了一个小优化。数字文字（例如，`1.0`）使用统一空间，在某些硬件上可能会导致问题，因此我们改为声明常量（参考[`stackoverflow.com/questions/13963765/declaring-constants-instead-of-literals-in-vertex-shader-standard-practice-or`](http://stackoverflow.com/questions/13963765/declaring-constants-instead-of-literals-in-vertex-shader-standard-practice-or)）。

与早期简单着色器相比，此着色器中要设置的变量更多，用于光照计算。我们将把这些发送到绘制方法中。

我们还需要一个略有不同的片段着色器。在项目层次结构中的`raw`文件夹上右键单击，转到**新建** | **文件**，并命名为`passthrough_fragment.shader`。添加以下代码：

```kt
precision mediump float;
varying vec4 v_Color;

void main() {
    gl_FragColor = v_Color;
}
```

片段着色器与简单着色器的唯一区别在于，我们用 varying `vec4 v_Color`替换了 uniform `vec4 u_Color`，因为颜色现在是从管道中的顶点着色器传递的。现在顶点着色器获得了一个颜色数组缓冲区。这是我们需要在设置/绘制代码中解决的新问题。

然后，在`MainActivity`中添加这些变量：

```kt
    // Rendering variables
    private int lightVertexShader;
    private int passthroughFragmentShader;
```

在`compileShaders`方法中编译着色器：

```kt
        lightVertexShader = loadShader(GLES20.GL_VERTEX_SHADER,
                R.raw.light_vertex);
        passthroughFragmentShader = loadShader(GLES20.GL_FRAGMENT_SHADER,
                R.raw.passthrough_fragment);
```

## 立方体法线和颜色

立方体的每个面朝向不同的方向，与面垂直。矢量是 XYZ 坐标。将其归一化为长度为 1 的矢量可用于指示此方向，并称为**法向量**。

我们传递给 OpenGL 的几何图形是由顶点定义的，而不是面。因此，我们需要为面的每个顶点提供一个法向量，如下图所示。严格来说，并非给定面上的所有顶点都必须面向同一方向。这在一种称为**平滑着色**的技术中使用，其中光照计算给出了曲面的错觉，而不是平面的错觉。我们将对每个面使用相同的法线（**硬边缘**），这也节省了我们在指定法线数据时的时间。我们的数组只需要指定六个矢量，可以扩展为 36 个法向量的缓冲区。颜色值也是如此。

![立方体法线和颜色](img/B05144_03_04.jpg)

每个顶点也有一个颜色。假设立方体的每个面都是一个实色，我们可以将该面的每个顶点分配相同的颜色。在`Cube.java`文件中，添加以下代码：

```kt
    public static final float[] CUBE_COLORS_FACES = new float[] {
        // Front, green
        0f, 0.53f, 0.27f, 1.0f,
        // Right, blue
        0.0f, 0.34f, 0.90f, 1.0f,
        // Back, also green
        0f, 0.53f, 0.27f, 1.0f,
        // Left, also blue
        0.0f, 0.34f, 0.90f, 1.0f,
        // Top, red
        0.84f,  0.18f,  0.13f, 1.0f,
        // Bottom, also red
        0.84f,  0.18f,  0.13f, 1.0f,
    };

    public static final float[] CUBE_NORMALS_FACES = new float[] {
        // Front face
        0.0f, 0.0f, 1.0f,
        // Right face
        1.0f, 0.0f, 0.0f,
        // Back face
        0.0f, 0.0f, -1.0f,
        // Left face
        -1.0f, 0.0f, 0.0f,
        // Top face
        0.0f, 1.0f, 0.0f,
        // Bottom face
        0.0f, -1.0f, 0.0f,
    };
```

对于立方体的每个面，我们定义了一个实色（`CUBE_COLORS_FACES`）和一个法向量（`CUBE_NORMALS_FACES`）。

现在，编写一个可重复使用的方法`cubeFacesToArray`，以生成`MainActivity`中实际需要的浮点数组。将以下代码添加到您的`Cube`类中：

```kt
    /**
     * Utility method for generating float arrays for cube faces
     *
     * @param model - float[] array of values per face.
     * @param coords_per_vertex - int number of coordinates per vertex.
     * @return - Returns float array of coordinates for triangulated cube faces.
     *               6 faces X 6 points X coords_per_vertex
     */
    public static float[] cubeFacesToArray(float[] model, int coords_per_vertex) {
        float coords[] = new float[6 * 6 * coords_per_vertex];
        int index = 0;
        for (int iFace=0; iFace < 6; iFace++) {
            for (int iVertex=0; iVertex < 6; iVertex++) {
                for (int iCoord=0; iCoord < coords_per_vertex; iCoord++) {
                    coords[index] = model[iFace*coords_per_vertex + iCoord];
                    index++;
                }
            }
        }
        return coords;
    }
```

将这些数据添加到`MainActivity`中的其他变量中，如下所示：

```kt
    // Model variables
    private static float cubeCoords[] = Cube.CUBE_COORDS;
    private static float cubeColors[] = Cube.cubeFacesToArray(Cube.CUBE_COLORS_FACES, 4);
    private static float cubeNormals[] = Cube.cubeFacesToArray(Cube.CUBE_NORMALS_FACES, 3);
```

您还可以删除`private float cubeColor[]`的声明，因为现在不再需要它。

有了法向量和颜色，着色器可以计算对象占据的每个像素的值。

## 准备顶点缓冲区

渲染管道要求我们为顶点、法向量和颜色设置内存缓冲区。我们已经有了顶点缓冲区，现在需要添加其他缓冲区。

添加变量，如下所示：

```kt
    // Rendering variables
    private FloatBuffer cubeVerticesBuffer;
    private FloatBuffer cubeColorsBuffer;
    private FloatBuffer cubeNormalsBuffer;
```

准备缓冲区，并将以下代码添加到`prepareRenderingCube`方法（从`onSurfaceCreated`调用）。 （这是完整的`prepareRenderingCube`方法的前半部分）：

```kt
    private void prepareRenderingCube() {
        // Allocate buffers
        ByteBuffer bb = ByteBuffer.allocateDirect(cubeCoords.length * 4);
        bb.order(ByteOrder.nativeOrder());
        cubeVerticesBuffer = bb.asFloatBuffer();
        cubeVerticesBuffer.put(cubeCoords);
        cubeVerticesBuffer.position(0);

        ByteBuffer bbColors = ByteBuffer.allocateDirect(cubeColors.length * 4);
 bbColors.order(ByteOrder.nativeOrder());
 cubeColorsBuffer = bbColors.asFloatBuffer();
 cubeColorsBuffer.put(cubeColors);
 cubeColorsBuffer.position(0);

 ByteBuffer bbNormals = ByteBuffer.allocateDirect(cubeNormals.length * 4);
 bbNormals.order(ByteOrder.nativeOrder());
 cubeNormalsBuffer = bbNormals.asFloatBuffer();
 cubeNormalsBuffer.put(cubeNormalParam);
 cubeNormalsBuffer.position(0);

        // Create GL program
```

## 准备着色器

已经定义了`lighting_vertex`着色器，我们需要添加参数句柄来使用它。在`MainActivity`类的顶部，添加四个变量到光照着色器参数：

```kt
    // Rendering variables
    private int cubeNormalParam;
    private int cubeModelViewParam;
    private int cubeLightPosParam;
```

在`prepareRenderingCube`方法中（由`onSurfaceCreated`调用），附加`lightVertexShader`和`passthroughFragmentShader`着色器，而不是简单的着色器，获取着色器参数，并启用数组，使其现在读取如下。（这是`prepareRenderingCube`的后半部分，从前一节继续）：

```kt
        // Create GL program
        cubeProgram = GLES20.glCreateProgram();
        GLES20.glAttachShader(cubeProgram, lightVertexShader);
        GLES20.glAttachShader(cubeProgram, passthroughFragmentShader);
        GLES20.glLinkProgram(cubeProgram);
        GLES20.glUseProgram(cubeProgram);

        // Get shader params
        cubeModelViewParam = GLES20.glGetUniformLocation(cubeProgram, "u_MVMatrix");
        cubeMVPMatrixParam = GLES20.glGetUniformLocation(cubeProgram, "u_MVP");
        cubeLightPosParam = GLES20.glGetUniformLocation(cubeProgram, "u_LightPos");

        cubePositionParam = GLES20.glGetAttribLocation(cubeProgram, "a_Position");
        cubeNormalParam = GLES20.glGetAttribLocation(cubeProgram, "a_Normal");
 cubeColorParam = GLES20.glGetAttribLocation(cubeProgram, "a_Color");

        // Enable arrays
        GLES20.glEnableVertexAttribArray(cubePositionParam);
        GLES20.glEnableVertexAttribArray(cubeNormalParam);
 GLES20.glEnableVertexAttribArray(cubeColorParam);

```

如果您参考我们之前编写的着色器代码，您会注意到这些对`glGetUniformLocation`和`glGetAttribLocation`的调用对应于那些脚本中声明的`uniform`和`attribute`参数，包括`cubeColorParam`从`u_Color`到现在的`a_Color`的更改。OpenGL 不需要这种重命名，但它有助于我们区分顶点属性和统一性。

引用数组缓冲区的着色器属性必须启用。

## 添加光源

接下来，我们将在场景中添加一个光源，并在绘制时告诉着色器它的位置。光源将被放置在用户的正上方。

在`MainActivity`的顶部，添加光源位置的变量：

```kt
    // Scene variables
    // light positioned just above the user
    private static final float[] LIGHT_POS_IN_WORLD_SPACE = new float[] { 0.0f, 2.0f, 0.0f, 1.0f };
    private final float[] lightPosInEyeSpace = new float[4];
```

通过添加以下代码到`onDrawEye`来计算光的位置：

```kt
        // Apply the eye transformation to the camera
        Matrix.multiplyMM(view, 0, eye.getEyeView(), 0, camera, 0);

        // Calculate position of the light
        Matrix.multiplyMV(lightPosInEyeSpace, 0, view, 0, LIGHT_POS_IN_WORLD_SPACE, 0);
```

请注意，我们使用`view`矩阵（眼睛`view *` `camera`）使用`Matrix.multiplyMV`函数将光的位置转换为当前视图空间。

现在，我们只需告诉着色器光源的位置和它所需的视图矩阵。修改`drawCube`方法（由`onDrawEye`调用），如下所示：

```kt
    private void drawCube() {
        GLES20.glUseProgram(cubeProgram);

        // Set the light position in the shader
 GLES20.glUniform3fv(cubeLightPosParam, 1, lightPosInEyeSpace, 0);

        // Set the ModelView in the shader, used to calculate lighting
 GLES20.glUniformMatrix4fv(cubeModelViewParam, 1, false, cubeView, 0);

        GLES20.glUniformMatrix4fv(cubeMVPMatrixParam, 1, false, modelViewProjection, 0);

        GLES20.glVertexAttribPointer(cubePositionParam, COORDS_PER_VERTEX,
                GLES20.GL_FLOAT, false, 0, cubeVerticesBuffer);
        GLES20.glVertexAttribPointer(cubeNormalParam, 3, GLES20.GL_FLOAT, false, 0,
 cubeNormalsBuffer);
 GLES20.glVertexAttribPointer(cubeColorParam, 4, GLES20.GL_FLOAT, false, 0,
 cubeColorsBuffer);

        GLES20.glDrawArrays(GLES20.GL_TRIANGLES, 0, cubeVertexCount);
    }
```

## 构建和运行应用程序

我们现在准备好了。构建并运行应用程序时，您将看到类似以下截图的屏幕：

！[构建和运行应用程序]（img/B05144_03_05.jpg）

# 旋转立方体

下一步很快。让我们让立方体旋转。这是通过在每帧中稍微旋转`cubeTransform`矩阵来实现的。我们可以为此定义一个`TIME_DELTA`值。添加静态变量，如下所示：

```kt
    // Viewing variables
    private static final float TIME_DELTA = 0.3f;
```

然后，修改每帧的`cubeTransform`，并将以下代码添加到`onNewFrame`方法：

```kt
Matrix.rotateM(cubeTransform, 0, TIME_DELTA, 0.5f, 0.5f, 1.0f);
```

`Matrix.rotateM`函数根据角度和轴向量对变换矩阵应用旋转。在这种情况下，我们围绕轴向量（0.5,0.5,1）旋转`TIME_DELTA`的角度。严格来说，您应该提供一个归一化的轴，但重要的是向量的方向而不是大小。

构建并运行它。现在立方体正在旋转。*令人惊叹！*

# 你好，地板！

在虚拟现实中，有一种脚踏实地的感觉可能很重要。感觉像站着（或坐着）要比像一个无身体的眼球漂浮在空间中更舒服得多。因此，让我们在场景中添加一个地板。

现在这应该更加熟悉了。我们将有一个类似于立方体的着色器、模型和渲染管道。所以，我们将不做太多解释，就这样做吧。

## 着色器

地板将使用我们的`light_shader`进行一些小修改和一个新的片段着色器。

通过添加`v_Grid`变量来修改`light_vertex.shader`，如下所示：

```kt
uniform mat4 u_Model;
uniform mat4 u_MVP;
uniform mat4 u_MVMatrix;
uniform vec3 u_LightPos;

attribute vec4 a_Position;
attribute vec4 a_Color;
attribute vec3 a_Normal;

varying vec4 v_Color;
varying vec3 v_Grid;

const float ONE = 1.0;
const float COEFF = 0.00001;

void main() {
 v_Grid = vec3(u_Model * a_Position);

    vec3 modelViewVertex = vec3(u_MVMatrix * a_Position);
    vec3 modelViewNormal = vec3(u_MVMatrix * vec4(a_Normal, 0.0));

    float distance = length(u_LightPos - modelViewVertex);
    vec3 lightVector = normalize(u_LightPos - modelViewVertex);
    float diffuse = max(dot(modelViewNormal, lightVector), 0.5);

    diffuse = diffuse * (ONE / (ONE + (COEFF * distance * distance)));
    v_Color = a_Color * diffuse;
    gl_Position = u_MVP * a_Position;
}
```

在`app/res/raw`中创建一个名为`grid_fragment.shader`的新着色器，如下所示：

```kt
precision mediump float;
varying vec4 v_Color;
varying vec3 v_Grid;

void main() {
    float depth = gl_FragCoord.z / gl_FragCoord.w; // Calculate world-space distance.

    if ((mod(abs(v_Grid.x), 10.0) < 0.1) || (mod(abs(v_Grid.z), 10.0) < 0.1)) {
        gl_FragColor = max(0.0, (90.0-depth) / 90.0) * vec4(1.0, 1.0, 1.0, 1.0)
                + min(1.0, depth / 90.0) * v_Color;
    } else {
        gl_FragColor = v_Color;
    }
}
```

这可能看起来很复杂，但我们所做的只是在一个纯色着色器上绘制一些网格线。`if`语句将检测我们是否在 10 的倍数的 0.1 单位内。如果是，我们将绘制一个颜色，介于白色（1,1,1,1）和`v_Color`之间，根据该像素的深度或其与相机的距离。`gl_FragCoord`是一个内置值，它给出了我们在窗口空间中渲染的像素的位置，以及深度缓冲区（`z`）中的值，该值将在范围[0,1]内。第四个参数`w`本质上是相机绘制距离的倒数，当与深度值结合时，给出了像素的世界空间深度。`v_Grid`变量实际上已经让我们根据顶点着色器中引入的本地顶点位置和模型矩阵，访问了当前像素的世界空间位置。

在`MainActivity`中，添加一个新的片段着色器变量：

```kt
    // Rendering variables
    private int gridFragmentShader;
```

在`compileShaders`方法中编译着色器，如下所示：

```kt
        gridFragmentShader = loadShader(GLES20.GL_FRAGMENT_SHADER,
                R.raw.grid_fragment);
```

## 地板模型数据

在项目中创建一个名为`Floor`的新的 Java 文件。添加地板平面坐标、法线和颜色：

```kt
    public static final float[] FLOOR_COORDS = new float[] {
        200f, 0, -200f,
        -200f, 0, -200f,
        -200f, 0, 200f,
        200f, 0, -200f,
        -200f, 0, 200f,
        200f, 0, 200f,
    };

    public static final float[] FLOOR_NORMALS = new float[] {
        0.0f, 1.0f, 0.0f,
        0.0f, 1.0f, 0.0f,
        0.0f, 1.0f, 0.0f,
        0.0f, 1.0f, 0.0f,
        0.0f, 1.0f, 0.0f,
        0.0f, 1.0f, 0.0f,
    };

    public static final float[] FLOOR_COLORS = new float[] {
            0.0f, 0.34f, 0.90f, 1.0f,
            0.0f, 0.34f, 0.90f, 1.0f,
            0.0f, 0.34f, 0.90f, 1.0f,
            0.0f, 0.34f, 0.90f, 1.0f,
            0.0f, 0.34f, 0.90f, 1.0f,
            0.0f, 0.34f, 0.90f, 1.0f,
    };
```

## 变量

将我们需要的所有变量添加到`MainActivity`中：

```kt
    // Model variables
    private static float floorCoords[] = Floor.FLOOR_COORDS;
    private static float floorColors[] = Floor.FLOOR_COLORS;
    private static float floorNormals[] = Floor.FLOOR_NORMALS;
    private final int floorVertexCount = floorCoords.length / COORDS_PER_VERTEX;
    private float[] floorTransform;
    private float floorDepth = 20f;

    // Viewing variables
    private float[] floorView;

    // Rendering variables
    private int gridFragmentShader;

    private FloatBuffer floorVerticesBuffer;
    private FloatBuffer floorColorsBuffer;
    private FloatBuffer floorNormalsBuffer;
    private int floorProgram;
    private int floorPositionParam;
    private int floorColorParam;
    private int floorMVPMatrixParam;
    private int floorNormalParam;
    private int floorModelParam;
    private int floorModelViewParam;
    private int floorLightPosParam;
```

## onCreate

在`onCreate`中分配矩阵：

```kt
        floorTransform = new float[16];
        floorView = new float[16];
```

## onSurfaceCreated

在`onSufraceCreated`中添加对`prepareRenderingFloor`的调用，我们将其编写如下：

```kt
        prepareRenderingFloor();
```

## initializeScene

在`initializeScene`方法中设置`floorTransform`矩阵：

```kt
        // Position the floor
        Matrix.setIdentityM(floorTransform, 0);
        Matrix.translateM(floorTransform, 0, 0, -floorDepth, 0);
```

## prepareRenderingFloor

这是完整的`prepareRenderingFloor`方法：

```kt
    private void prepareRenderingFloor() {
        // Allocate buffers
        ByteBuffer bb = ByteBuffer.allocateDirect(floorCoords.length * 4);
        bb.order(ByteOrder.nativeOrder());
        floorVerticesBuffer = bb.asFloatBuffer();
        floorVerticesBuffer.put(floorCoords);
        floorVerticesBuffer.position(0);

        ByteBuffer bbColors = ByteBuffer.allocateDirect(floorColors.length * 4);
        bbColors.order(ByteOrder.nativeOrder());
        floorColorsBuffer = bbColors.asFloatBuffer();
        floorColorsBuffer.put(floorColors);
        floorColorsBuffer.position(0);

        ByteBuffer bbNormals = ByteBuffer.allocateDirect(floorNormals.length * 4);
        bbNormals.order(ByteOrder.nativeOrder());
        floorNormalsBuffer = bbNormals.asFloatBuffer();
        floorNormalsBuffer.put(floorNormals);
        floorNormalsBuffer.position(0);

        // Create GL program
        floorProgram = GLES20.glCreateProgram();
        GLES20.glAttachShader(floorProgram, lightVertexShader);
        GLES20.glAttachShader(floorProgram, gridFragmentShader);
        GLES20.glLinkProgram(floorProgram);
        GLES20.glUseProgram(floorProgram);

        // Get shader params
        floorPositionParam = GLES20.glGetAttribLocation(floorProgram, "a_Position");
        floorNormalParam = GLES20.glGetAttribLocation(floorProgram, "a_Normal");
        floorColorParam = GLES20.glGetAttribLocation(floorProgram, "a_Color");

        floorModelParam = GLES20.glGetUniformLocation(floorProgram, "u_Model");
        floorModelViewParam = GLES20.glGetUniformLocation(floorProgram, "u_MVMatrix");
        floorMVPMatrixParam = GLES20.glGetUniformLocation(floorProgram, "u_MVP");
        floorLightPosParam = GLES20.glGetUniformLocation(floorProgram, "u_LightPos");

        // Enable arrays
        GLES20.glEnableVertexAttribArray(floorPositionParam);
        GLES20.glEnableVertexAttribArray(floorNormalParam);
        GLES20.glEnableVertexAttribArray(floorColorParam);
    }
```

## onDrawEye

在`onDrawEye`中计算 MVP 并绘制地板：

```kt
        Matrix.multiplyMM(floorView, 0, view, 0, floorTransform, 0);
        Matrix.multiplyMM(modelViewProjection, 0, perspective, 0, floorView, 0);
        drawFloor();
```

## 绘制地板

定义一个`drawFloor`方法，如下所示：

```kt
    private void drawFloor() {
        GLES20.glUseProgram(floorProgram);
        GLES20.glUniform3fv(floorLightPosParam, 1, lightPosInEyeSpace, 0);
        GLES20.glUniformMatrix4fv(floorModelParam, 1, false, floorTransform, 0);
        GLES20.glUniformMatrix4fv(floorModelViewParam, 1, false, floorView, 0);
        GLES20.glUniformMatrix4fv(floorMVPMatrixParam, 1, false, modelViewProjection, 0);
        GLES20.glVertexAttribPointer(floorPositionParam, COORDS_PER_VERTEX,
                GLES20.GL_FLOAT, false, 0, floorVerticesBuffer);
        GLES20.glVertexAttribPointer(floorNormalParam, 3, GLES20.GL_FLOAT, false, 0,
                floorNormalsBuffer);
        GLES20.glVertexAttribPointer(floorColorParam, 4, GLES20.GL_FLOAT, false, 0,
                floorColorsBuffer);
        GLES20.glDrawArrays(GLES20.GL_TRIANGLES, 0, floorVertexCount);
    }
```

构建并运行它。现在它看起来像以下的截图：

![drawFloor](img/B05144_03_06.jpg)

*哇！*

# 嘿，看这个！

在项目的最后部分，我们添加了一个功能，当您看着一个物体（立方体）时，它会用不同的颜色进行高亮显示。

这是通过`CardboardView`接口方法`onNewFrame`来实现的，该方法传递当前头部变换信息。

## isLookingAtObject 方法

让我们从最有趣的部分开始。我们将从 Google 的寻宝演示中借用`isLookingAtObject`方法。它通过计算对象在眼睛空间中的位置来检查用户是否正在看着一个对象，并在用户看着对象时返回 true。在`MainActivity`中添加以下代码：

```kt
/**
     * Check if user is looking at object by calculating where the object is in eye-space.
     *
     * @return true if the user is looking at the object.
     */
    private boolean isLookingAtObject(float[] modelView, float[] modelTransform) {
        float[] initVec = { 0, 0, 0, 1.0f };
        float[] objPositionVec = new float[4];

        // Convert object space to camera space. Use the headView from onNewFrame.
        Matrix.multiplyMM(modelView, 0, headView, 0, modelTransform, 0);
        Matrix.multiplyMV(objPositionVec, 0, modelView, 0, initVec, 0);

        float pitch = (float) Math.atan2(objPositionVec[1], -objPositionVec[2]);
        float yaw = (float) Math.atan2(objPositionVec[0], -objPositionVec[2]);

        return Math.abs(pitch) < PITCH_LIMIT && Math.abs(yaw) < YAW_LIMIT;
    }
```

该方法接受两个参数：我们要测试的对象的`modelView`和`modelTransform`变换矩阵。它还引用了`headView`类变量，我们将在`onNewFrame`中设置。

一个更精确的方法是从相机向场景中的方向发射一条射线，并确定它是否与场景中的任何几何体相交。这将非常有效，但也非常消耗计算资源。

相反，这个函数采用了更简单的方法，甚至不使用对象的几何形状。它使用对象的视图变换来确定对象距离屏幕中心有多远，并测试该向量的角度是否在一个狭窄的范围内（`PITCH_LIMIT`和`YAW_LIMIT`）。*是的，我知道，人们获得博士学位来想出这些东西！*

让我们按照以下方式定义我们需要的变量：

```kt
    // Viewing variables
    private static final float YAW_LIMIT = 0.12f;
    private static final float PITCH_LIMIT = 0.12f;

    private float[] headView;
```

在`onCreate`中分配`headView`：

```kt
        headView = new float[16];
```

在每一帧新的`headView`值。在`onNewFrame`中添加以下代码：

```kt
        headTransform.getHeadView(headView, 0);
```

然后，修改`drawCube`以检查用户是否正在看着立方体，并决定使用哪种颜色：

```kt
        if (isLookingAtObject(cubeView, cubeTransform)) {
            GLES20.glVertexAttribPointer(cubeColorParam, 4, GLES20.GL_FLOAT, false, 0,
                    cubeFoundColorsBuffer);
        } else {
            GLES20.glVertexAttribPointer(cubeColorParam, 4, GLES20.GL_FLOAT, false, 0,
                    cubeColorsBuffer);
        }
```

*就是这样！*除了一个（微小的）细节：我们需要第二组顶点颜色用于突出显示模式。我们将通过使用相同的黄色绘制所有面来突出显示立方体。为了实现这一点，需要进行一些更改。

在`Cube`中，添加以下 RGBA 值：

```kt
    public static final float[] CUBE_FOUND_COLORS_FACES = new float[] {
        // Same yellow for front, right, back, left, top, bottom faces
        1.0f,  0.65f, 0.0f, 1.0f,
        1.0f,  0.65f, 0.0f, 1.0f,
        1.0f,  0.65f, 0.0f, 1.0f,
        1.0f,  0.65f, 0.0f, 1.0f,
        1.0f,  0.65f, 0.0f, 1.0f,
        1.0f,  0.65f, 0.0f, 1.0f,
    };
```

在`MainActivity`中，添加这些变量：

```kt
    // Model variables
    private static float cubeFoundColors[] = Cube.cubeFacesToArray(Cube.CUBE_FOUND_COLORS_FACES, 4);

    // Rendering variables
    private FloatBuffer cubeFoundColorsBuffer;
```

将以下代码添加到`prepareRenderingCube`方法中：

```kt
        ByteBuffer bbFoundColors = ByteBuffer.allocateDirect(cubeFoundColors.length * 4);
        bbFoundColors.order(ByteOrder.nativeOrder());
        cubeFoundColorsBuffer = bbFoundColors.asFloatBuffer();
        cubeFoundColorsBuffer.put(cubeFoundColors);
        cubeFoundColorsBuffer.position(0);
```

构建并运行它。当你直接看着立方体时，它会被突出显示。

### 提示

如果立方体不那么接近，可能会更有趣和具有挑战性。尝试将`cubeDistance`设置为*12f*。

就像寻宝演示一样，尝试每次看着它时设置一个新的随机立方体位置。现在，你有了一个游戏！

# 摘要

在本章中，我们从头开始构建了一个 Cardboard Android 应用，从一个新项目开始，逐渐添加 Java 代码。在我们的第一个构建中，我们有一个三角形的立体视图，你可以在 Google Cardboard 头盔中看到。

然后我们添加了模型变换、3D 摄像机视图、透视和头部旋转变换，并讨论了一些关于矩阵数学的内容。我们建立了一个立方体的 3D 模型，然后创建了着色器程序，使用光源来渲染带有阴影的立方体。我们还为立方体添加了动画，并添加了一个地板网格。最后，我们添加了一个功能，当用户看着立方体时，它会被突出显示。

在这个过程中，我们享受了关于 3D 几何、OpenGL、着色器、用于渲染管线的几何法线和数据缓冲区的良好讨论。我们还开始思考如何将代码中的常见模式抽象为可重用的方法。

在下一章中，我们将采用不同的方法来使用 Android 布局视图进行立体渲染，构建一个有用的“虚拟大厅”，可以用作 3D 菜单系统或通往其他世界的门户。
