# 第十章. 使用 OpenGL 进行 3D 游戏编程

在本章中，我们将关注以下食谱：

+   介绍 OpenGL

+   使用 OpenGL 构建一个迷你三维动画游戏

# 简介

在前面的章节中，你深入学习了物理模拟的结构。现在，我们将探索游戏中最有趣的部分，即向你的游戏中添加三维对象。在本章中，我们将开始探索 OpenGL 的基础知识。然后，我们将通过制作一些三维模型逐步深入探索 OpenGL。在本章中，我们将通过使用二维模型项目开始学习，然后我们将增强项目以适应三维模型。

# 介绍 OpenGL

OpenGL 代表开放图形库。这是一个广泛使用的库，用于可视化二维和三维对象。这是一个标准的多功能二维和三维内容创建图形库。它在各种领域中使用，如机械设计、建筑设计、游戏、原型设计、飞行模拟等。OpenGL 用于配置和提交三维图形数据到设备。所有数据都以矩阵的形式准备，并转换为顶点，这些顶点被转换和组装以生成二维光栅化图像。二维图形有两个轴，即 x 轴和 y 轴；然而，在三维图形的情况下，我们有三维轴，即 x 轴、y 轴和 z 轴，其中 z 轴是深度轴。

这个库旨在将普通函数调用编译成图形命令，这些命令将在图形渲染硬件上执行。所有图形硬件都设计用来执行图形命令。OpenGL 以非常快的速度绘制和渲染视图。

## 准备工作

Xcode 提供了一个内置的 OpenGL-ES 项目模板；然而，我们认为对于初学者来说，从该模板开始可能会感到困惑。一种典型的方法是逐步编写代码，以便理解代码的工作方式和每个函数的功能。

在这里，我们将从头开始编写所有代码，这将帮助你理解使用 OpenGL 渲染视图时每行代码的细节。要添加框架，请执行以下步骤：

1.  启动 Xcode。创建一个单视图应用程序，点击**下一步**。将你的项目命名为 OpenGLSample，点击**下一步**，选择一个文件夹来保存它，然后点击**创建**。

1.  现在，我们将向项目中添加所有必需的框架。要添加框架，第一步是添加我们 OpenGL 项目所需的两个框架。它们是`OpenGLES.frameworks`和`GLKit.framework`。

1.  要在 Xcode 6 中添加这些框架，请单击 Groups & Files 树中的 OpenGLSample 项目，并选择**OpenGLSample**目标。展开**Link Binary with the Libraries**部分，单击加号按钮，并选择**OpenGLES.framework**。同样，为**GLKit.framework**也重复此操作，如下所示：![准备就绪](img/00157.jpeg)

## 如何操作

要在我们的应用程序中创建三维纹理，请执行以下步骤：

1.  打开`Main.storyboard`文件并选择视图控制器。您将看到在视图控制器中已经添加了一个视图。

1.  选择视图，打开 Identity 检查器，并将视图的类设置为`GLKView`，如下所示：![如何操作](img/00158.jpeg)

1.  打开`ViewController.h`，导入`GLKit`/`GLKit.h`，并将视图控制器的主类从`UIViewController`更改为`GLKViewController`。`GLKViewController`除了提供所有原生视图控制器功能外，还提供了 OpenGL 渲染循环。代码如下：

    ```swift
    #import"GLKit/GLKit.h"
    @interface ViewController : GLKViewController
    ```

1.  修改`viewController.m`。在`viewDidLoad`中添加以下代码：

    ```swift
    GLKView* view = (GLKView*)self.view;

        view.context = [[EAGLContext alloc] initWithAPI:kEAGLRenderingAPIOpenGLES2];
    ```

1.  添加以下方法：

    ```swift
    (void)glkView:(GLKView *)view drawInRect:(CGRect)rect{
    glClearColor(0.0, 0.5, 0.0, 1.0); glClear(GL_COLOR_BUFFER_BIT);
    }
    ```

    代码的第一行调用`glClearColor`，这实际上告诉 OpenGL 将清除颜色设置为 RGBA 值（0.0，0.5，0.0，1.0）——即完全不透明，深绿色。下一行指示 OpenGL 实际清除颜色缓冲区——即，它用上一行设置的清除颜色填充整个屏幕。

1.  现在，编译并运行项目。您应该会看到以下截图类似的内容：![如何操作](img/00159.jpeg)

1.  现在，是时候创建 OpenGL 上下文了。在 OpenGL 中绘制三角形比绘制正方形更容易，因为三角形总是共面的，也就是说，形状中的所有点都在同一个平面上。因此，要绘制一个正方形，我们首先绘制两个共享边的三角形。

1.  为了通知 OpenGL 顶点的位置，我们使用一个结构体来表示顶点和数组。这将稍后在`GLView`中显示如下：

    ```swift
        typedef struct {
        GLKVector3 position;
        } Vertex;
        const Vertex SquareVertices[] = {
        {-1, -1 , 0},// vertex 0: bottom left 
        {1, -1 , 0}, // vertex 1: bottom right
        {1, 1 , 0}, // vertex 2: top right
        {-1, 1 , 0}, // vertex 4: top left
        };
    ```

1.  现在我们需要定义哪个三角形使用哪个顶点。在 OpenGL 中，我们通过为每个顶点编号，然后每次给出三个数字来描述三角形：

    ```swift
       const GLubyte SquareTriangles[] = {
       0, 1, 2, // BL->BR->TR
       2, 3, 0 // TR->TL->BL
       };
    ```

    在这种情况下，`Glubyte`是第一个三角形使用顶点`0`、`1`和`2`，第二个三角形使用顶点`2`、`3`和`0`的类型。请注意，两个三角形都使用了顶点`0`和`2`。这意味着它们共享一条边，这意味着两个三角形之间不会有任何间隙。

1.  `SquareVertices`和`SquareTriangles`数组都需要存储在缓冲区中，以便 OpenGL 可以使用它们进行渲染，如下所示：

    ```swift
       @interface ViewController () {
       GLuint _vertexBuffer;
       GLuint _indexBuffer;
       GLKBaseEffect* _squareEffect;
       }
       @end
    ```

1.  首先，我们使用 OpenGL 上下文设置`GLKView`，如下所示。因为，如果我们不这样做，我们的所有 OpenGL 命令都不会有任何作用。

    ```swift
       GLKView* view = (GLKView*)self.view;
       view.context = [[EAGLContext alloc] initWithAPI:kEAGLRenderingAPIOpenGLES2];
       [EAGLContext setCurrentContext:view.context];
    ```

1.  接下来，我们从顶点缓冲区开始创建缓冲区：

    ```swift
       glGenBuffers(1, &_vertexBuffer);
       glBindBuffer(GL_ARRAY_BUFFER, _vertexBuffer);
    ```

1.  然后将顶点缓冲区填充顶点信息：

    ```swift
         glBufferData(GL_ARRAY_BUFFER, sizeof(SquareVertices),
         SquareVertices, GL_STATIC_DRAW);
    ```

1.  然后对索引缓冲区做同样的事情，你可能还记得它存储了两个三角形将使用哪些顶点的信息：

    ```swift
       glGenBuffers(1, &_indexBuffer);
       glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, _indexBuffer);
       glBufferData(GL_ELEMENT_ARRAY_BUFFER, sizeof(SquareTriangles), SquareTriangles, GL_STATIC_DRAW);
    ```

    一旦完成，所有信息都已传递给 OpenGL。GLKit 提供了一些效果，这些是包含颜色、位置、方向等信息的对象。在这个活动中，我们将使一个正方形变成红色，并将其显示在屏幕中间。

1.  第一步是创建效果对象，然后向其提供一个投影矩阵。投影矩阵控制屏幕上物体的整体大小。在这种情况下，我们创建了一个使用屏幕宽高比和 60 度视场的投影矩阵：

    ```swift
    _squareEffect = [[GLKBaseEffect alloc] init];
    float aspectRatio = self.view.bounds.size.width/self.view.bounds.size.height;
    float fieldOfViewDegrees = 60.0;

    GLKMatrix4 projectionMatrix = GLKMatrix4MakePerspective(GLKMathDegreesToRadians(fieldOfViewDegrees),aspectRatio, 0.1, 10.0);
    _squareEffect.transform.projectionMatrix = projectionMatrix;
    ```

1.  现在，我们需要提供一个模型视图矩阵。模型视图矩阵控制对象相对于摄像机的位置：

    ```swift
      GLKMatrix4 modelViewMatrix = GLKMatrix4MakeTranslation(0.0f,   0.0f, -6.0f);
      _squareEffect.transform.modelviewMatrix = modelViewMatrix;
    //Set the constant color red for the effects.
      _squareEffect.useConstantColor = YES;
      _squareEffect.constantColor = GLKVector4Make(1.0, 0.0, 0.0, 1.0);
    ```

1.  在`viewDidLoad`方法之后创建`drawInRect`方法：

    ```swift
      - (void)glkView:(GLKView *)view drawInRect:(CGRect)rect {
       }
    ```

1.  实际的渲染工作是在`glkView:drawInRect`方法中完成的。在这个过程中发生的第一件事是清除视图，通过用黑色填充屏幕：

    ```swift
       glClearColor(0.0, 0.0, 0.0, 1.0);
       glClear(GL_COLOR_BUFFER_BIT);
    ```

1.  现在调用`prepareToDraw`。它以这种方式配置 OpenGL，使得我们绘制的任何内容都将使用效果的设置：

    ```swift
       [_squareEffect prepareToDraw];
    ```

1.  我们首先告诉 OpenGL 我们将要处理位置，然后告诉 OpenGL 在顶点数据中找到位置信息的位置：

    ```swift
       glEnableVertexAttribArray(GLKVertexAttribPosition);
       glVertexAttribPointer(GLKVertexAttribPosition, 3, GL_FLOAT, GL_FALSE, 0, 0);
    ```

1.  最后，我们需要知道我们要求 OpenGL 绘制多少个顶点。这可以通过取整个索引数组的大小，并将其除以该数组中一个元素的大小来得出：

    ```swift
      int numberOfTriangles = sizeof(SquareTriangles)/sizeof(SquareTriangles[0]);
      glDrawElements(GL_TRIANGLES, numberOfTriangles, GL_UNSIGNED_BYTE, 0);
    ```

因此，我们的最终实现类可能看起来像以下代码：

```swift
#import "ViewController.h"
typedef struct {
GLKVector3 position;
} Vertex;
const Vertex SquareVertices[] = {
{-1, -1 , 0}, {1, -1 , 0}, {1, 1 , 0}, {-1, 1 , 0}
};
const GLubyte SquareTriangles[] = {
0, 1, 2, 
2, 3, 0 
};
@interface ViewController () {
GLuint _vertexBuffer;
GLuint _indexBuffer;
GLKBaseEffect* _squareEffect;
}
@end
@implementation ViewController
- (void)viewDidLoad
{
[super viewDidLoad];
GLKView* view = (GLKView*)self.view;
view.context = [[EAGLContext alloc] initWithAPI:kEAGLRenderingAPIOpenGLES2];
[EAGLContext setCurrentContext:view.context];

glGenBuffers(1, &_vertexBuffer);
glBindBuffer(GL_ARRAY_BUFFER, _vertexBuffer);
glBufferData(GL_ARRAY_BUFFER, sizeof(SquareVertices), SquareVertices,
GL_STATIC_DRAW);
glGenBuffers(1, &_indexBuffer);
glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, _indexBuffer);
glBufferData(GL_ELEMENT_ARRAY_BUFFER, sizeof(SquareTriangles),
SquareTriangles, GL_STATIC_DRAW);
_squareEffect = [[GLKBaseEffect alloc] init];
float aspectRatio = self.view.bounds.size.width/self.view.bounds.size.height;
float fieldOfViewDegrees = 60.0;
GLKMatrix4 projectionMatrix = GLKMatrix4MakePerspective(GLKMathDegreesToRadians(fieldOfViewDegrees),aspectRatio, 0.1, 10.0);
_squareEffect.transform.projectionMatrix = projectionMatrix;

GLKMatrix4 modelViewMatrix = GLKMatrix4MakeTranslation(0.0f, 0.0f, -6.0f);
_squareEffect.transform.modelviewMatrix = modelViewMatrix;

_squareEffect.useConstantColor = YES;
_squareEffect.constantColor = GLKVector4Make(1.0, 0.0, 0.0, 1.0);
}
- (void)glkView:(GLKView *)view drawInRect:(CGRect)rect {
glClearColor(0.0, 0.0, 0.0, 1.0);
glClear(GL_COLOR_BUFFER_BIT);

[_squareEffect prepareToDraw];
glEnableVertexAttribArray(GLKVertexAttribPosition);
glVertexAttribPointer(GLKVertexAttribPosition, 3, GL_FLOAT, GL_FALSE, 0, 0);
int numberOfVertices = sizeof(SquareTriangles)/sizeof(SquareTriangles[0]);
glDrawElements(GL_TRIANGLES, numberOfVertices, GL_UNSIGNED_BYTE, 0);
}
@end
```

运行项目，你将在黑色屏幕上看到红色的正方形框，如下面的截图所示：

![如何操作](img/00160.jpeg)

# 使用 OpenGL 构建一个迷你 3D 动画游戏

在这个菜谱中，我们将加载一个纹理并将其应用到正方形上。稍后，我们将制作一个立方体，最后我们将学习如何通过在三维空间中旋转我们的立方体来实现三维动画。

## 如何操作

现在，我们将从之前留下的地方开始，并加载所有纹理。要加载纹理，请遵循以下步骤：

1.  首先，在我们的顶点结构中，我们需要包含纹理坐标信息：

    ```swift
    typedef struct {
    GLKVector3 position; // the location of each vertex in space
    GLKVector2 textureCoordinates; // the texture coordinates for each vertex
    } Vertex;
    const Vertex SquareVertices[] = {
    {{-1, -1 , 0}, {0,0}}, // bottom left
    {{1, -1 , 0}, {1,0}}, // bottom right
    {{1, 1 , 0}, {1,1}}, // top right
    {{-1, 1 , 0}, {0,1}}, // top left
    };
    ```

1.  接下来，在我们的项目中添加一个图像（任何图像），将其重命名为`Texture.png`，然后在`viewDidLoad`中添加以下代码：

    ```swift
    NSString* imagePath = [[NSBundle mainBundle]
    pathForResource:@"Texture" ofType:@"png"];
    NSError* error = nil;
    GLKTextureInfo* texture = [GLKTextureLoader textureWithContentsOfFile:imagePath options:nil error:&error];
    if (error != nil) {
    NSLog(@"Problem loading texture: %@", error);
    }
    _squareEffect.texture2d0.name = texture.name;
    ```

1.  要修改之前正方形的颜色，请删除以下行：

    ```swift
    _squareEffect.useConstantColor = YES;
    _squareEffect.constantColor = GLKVector4Make(1.0, 0.0, 0.0, 1.0);
    ```

1.  最后，在`glkView:drawInRect`中渲染时，我们指示 OpenGL 在顶点信息中找到纹理坐标的位置：

    ```swift
    glEnableVertexAttribArray(GLKVertexAttribTexCoord0);
    glVertexAttribPointer(GLKVertexAttribTexCoord0, 2, GL_FLOAT, GL_FALSE,
    sizeof(Vertex), (void*)offsetof(Vertex, textureCoordinates));
    ```

    当正方形被渲染时，你会在其上看到你的图像，如下面的截图所示：

    ![如何操作](img/00161.jpeg)

1.  现在我们将其制作成一个立方体。立方体由八个顶点组成，因此我们需要为每个顶点提供信息，包括其位置和纹理坐标。从现在开始，将`SquareVertices`重命名为`CubeVertices`以遵循更好的命名约定。

    ```swift
    const Vertex CubeVertices[] = {
    {{-1, -1, 1}, {0,0}}, // bottom left front
    {{1, -1, 1}, {1,0}}, // bottom right front
    {{1, 1, 1}, {1,1}}, // top right front
    {{-1, 1, 1}, {0,1}}, // top left front
    {{-1, -1, -1}, {1,0}}, // bottom left back
    {{1, -1, -1}, {0,0}}, // bottom right back
    {{1, 1, -1}, {0,1}}, // top right back
    {{-1, 1, -1}, {1,1}}, // top left back
    };

    const GLubyte CubeTriangles[] = {
    0, 1, 2, // front face 1
    2, 3, 0, // front face 2
    4, 5, 6, // back face 1
    6, 7, 4, // back face 2
    7, 4, 0, // left face 1
    0, 3, 7, // left face 2
    2, 1, 5, // right face 1
    5, 6, 2, // right face 2
    7, 3, 6, // top face 1
    6, 2, 3, // top face 2
    4, 0, 5, // bottom face 1
    5, 1, 0, // bottom face 2
    };
    ```

1.  下一个步骤完全是出于美观考虑：立方体将会旋转，以展示它实际上是一个三维物体。

    选择以下行：

    ```swift
    GLKMatrix4 modelViewMatrix = GLKMatrix4MakeTranslation(0.0f, 0.0f, -6.0f);
    _squareEffect.transform.modelviewMatrix = modelViewMatrix;
    ```

    替换为以下内容：

    ```swift
    GLKMatrix4 modelViewMatrix = GLKMatrix4MakeTranslation(0.0f, 0.0f, -6.0f);
    modelViewMatrix = GLKMatrix4RotateX(modelViewMatrix,GLKMathDegreesToRadians(45));
    modelViewMatrix = GLKMatrix4RotateY(modelViewMatrix,GLKMathDegreesToRadians(45));
    _squareEffect.transform.modelviewMatrix = modelViewMatrix;
    ```

    然而，为了绘制我们的立方体，需要添加并启用深度缓冲区。深度缓冲区是提供三维和更逼真外观的对象所必需的。

1.  在调用 EAGLContext 的 `setCurrentContext` 方法后立即添加以下代码：

    ```swift
    view.drawableDepthFormat = GLKViewDrawableDepthFormat24;
    glEnable(GL_DEPTH_TEST);
    ```

1.  最后，将 `glClear(GL_COLOR_BUFFER_BIT);` 行替换为 `glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT);` 行。

1.  编译并运行项目以查看三维立方体，如图所示：![如何操作](img/00162.jpeg)

1.  现在我们将把它提升到下一个层次，通过添加代码来旋转一个立方体。现在我们将动画化视图中的移动，例如旋转。在接口部分将以下实例变量添加到 `ViewController` 类中：

    ```swift
       float rotation;
    ```

1.  接下来，将以下方法添加到类中：

    ```swift
    - (void) update {
    NSTimeInterval timeInterval = self.timeSinceLastUpdate;
    float rotationSpeed = 15 * timeInterval;
    rotation += rotationSpeed;
    GLKMatrix4 modelViewMatrix = GLKMatrix4MakeTranslation(0.0f, 0.0f, -6.0f);
    modelViewMatrix = GLKMatrix4RotateX(modelViewMatrix,
    GLKMathDegreesToRadians(45));
    modelViewMatrix = GLKMatrix4RotateY(modelViewMatrix,
    GLKMathDegreesToRadians(rotation));
    _squareEffect.transform.modelviewMatrix = modelViewMatrix;
    }
    ```

1.  现在，更新函数应该看起来类似于以下截图：![如何操作](img/00163.jpeg)

1.  现在编译并运行应用程序。您会发现您的立方体以我们指定的角度旋转，如图所示：![如何操作](img/00164.jpeg)

## 参考以下内容

到目前为止，我们已经学习了使用 OpenGL 创建二维和三维模型的各种方法。OpenGL 是三维游戏编程的骨架，因此是一个非常广泛的主题。我们只是对 OpenGL 进行了简要的浏览，要了解更多信息，您可以参考 [`developer.apple.com/opengl/`](https://developer.apple.com/opengl/)。
