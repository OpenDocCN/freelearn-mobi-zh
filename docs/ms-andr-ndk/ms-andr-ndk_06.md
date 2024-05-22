# 第六章：OpenGL ES 3.1 与跨平台渲染

在本章中，我们将学习如何实现在 OpenGL 4 和 OpenGL ES 3 之上的抽象层，以便让我们的图形应用程序能在 Android 和桌面计算机上运行。首先，我们从一些基本的向量与线性代数类开始。

# 线性代数与变换

在`Core/VecMath.h`文件中，有一系列针对向量和矩阵的特定类和辅助工具。我们主要使用的类是`LVector2`、`LVector3`、`LVector4`、`LMatrix3`、`LMatrix4`和`LQuaternion`，这些类定义了基本的代数运算。它们有一些快捷方式，以便编写任何数学密集型代码：

```java
using vec2 = LVector2;
using vec3 = LVector3;
using vec4 = LVector4;
using mat3 = LMatrix3;
using mat4 = LMatrix4;
using quat = LQuaternion;
```

这个小型数学库基本上是从 Linderdaum Engine ([`www.linderdaum.com`](http://www.linderdaum.com))压缩的一些代数代码。

此外，在`Math`命名空间中有一组有用的函数，用于处理不同的投影变换计算。在后续章节中，它们将被大量使用。

# 使用 SDL2 进行图形初始化

在我们之前的书籍《*Android NDK Game Development Cookbook*, *Packt Publishing*》中，我们详细学习了如何初始化 Android 上的 OpenGL ES 2 和桌面上的 OpenGL 3 核心配置。现在，我们将使用 SDL2 库来完成这项工作，该库可在[`www.libsdl.org`](https://www.libsdl.org)获取。让我们看看`1_GLES3`示例。这个示例的 Java 代码（除了 SDL2 内部实现之外）简短且简单：

```java
package com.packtpub.ndkmastering;
import android.app.Activity;
import android.os.Bundle;
public class AppActivity extends org.libsdl.app.SDLActivity
{
  static
  {
    System.loadLibrary( "NativeLib" );
  }
  public static AppActivity m_Activity;
  @Override protected void onCreate( Bundle icicle )
  {
    super.onCreate( icicle );
    m_Activity = this;
  }
};
```

其他所有操作都在 C++代码中完成。有一个`main()`函数，它通过 SDL2 使用宏重新定义，使我们的应用程序看起来像是桌面应用程序：

```java
int main(int argc, char* argv[])
{
  clSDL SDLLibrary;
```

首先，使用`clSDLWindow`类创建一个窗口和一个 OpenGL 渲染上下文：

```java
  g_Window = clSDLWindow::CreateSDLWindow( "GLES3", 1024, 768 );
```

然后，我们可以获取 OpenGL 函数的指针。这种抽象比静态链接到 OpenGL 库更优越，因为它使我们的代码更具可移植性。例如，在 Windows 上，如果不使用第三方库，你不能静态链接到核心 OpenGL 函数：

```java
  LGL3 = std::unique_ptr<sLGLAPI>( new sLGLAPI() );
  LGL::GetAPI( LGL3.get() );
```

这是我们在第四章，*组织虚拟文件系统*中处理虚拟文件系统时已经使用的回调。在这个例子中我们不需要任何路径，所以让我们使用一个空字符串：

```java
  OnStart( "" );
```

事件循环是显式完成的，并包含对`OnDrawFrame()`函数的调用：

```java
  while( g_Window && g_Window->HandleInput() )
  {
    OnDrawFrame();
    g_Window->Swap();
  }
  g_Window = nullptr;
  return 0;
}
```

这些包装类（`clSDL`和`clSDLWindow`）分别声明在`SDLLibrary.h`和`SDLWindow.h`文件中。`clSDL`类是基于 SDL 的 RAII 包装器，在构造函数和析构函数中进行库的初始化和反初始化：

```java
clSDL()
{
  SDL_Init( SDL_INIT_VIDEO );
}
virtual ~clSDL()
{
  SDL_Quit();
}
```

`clSDLWindow`类表示一个带有 OpenGL 上下文和系统消息泵的窗口抽象：

```java
class clSDLWindow: public iIntrusiveCounr
{
private:
  SDL_Window* m_Window;
  SDL_GLContext m_Context;
  float m_Width;
  float m_Height;
  std::atomic<bool> m_Pendingit;
public:
  clSDLWindow( const std::string& Title, int Width, int Height );
  virtual ~clSDLWindow();
  void RequestExit()
  {
    m_PendingExit = true;
  }
  void Swap();
```

这个成员函数执行消息循环的一次迭代：

```java
  bool HandleInput()
  {
    SDL_Event Event;
    while ( SDL_PollEvent(&Event) && !m_PendingExit )
    {
      if ( (Event.type == SDL_QUIT) || !this->HandleEvent( Event ) )
      m_PendingExit = true;
    }
    return !m_PendingExit;
  }
```

将整数坐标转换为浮点数标准化坐标 0..1，以便更容易使用不同分辨率的屏幕：

```java
  vec2 GetNormalizedPoint( int x, int y ) const
  {
    return vec2(
      static_cast<float>(x) / m_Width,
      static_cast<float>(y) / m_Height
    );
  }
```

以下方法对于构建当前窗口的投影矩阵很有用：

```java
  float GetAspect() const
  {
    return m_Width / m_Height;
  }
```

一个公共静态辅助方法，用于创建`clSDLWindow`的实例：

```java
public:
  static clPtr<clSDLWindow> CreateSDLWindow(
    const std::string& Title, int Width, int Height )
  {
    return make_intrusive<clSDLWindow>( Title, Width, Height );
  }
```

`HandleEvent()`成员函数负责将 SDL2 事件分派给我们的回调函数：

```java
private:
  bool HandleEvent( const SDL_Event& Event );};
```

`HandleEvent()`的实现如下：

```java
  bool clSDLWindow::HandleEvent( const SDL_Event& Event )
  {
    switch ( Event.type )
    {
      case SDL_WINDOWEVENT:
        if ( Event.window.event == SDL_WINDOWEVENT_SIZE_CHANGED)
        {
          m_Width  = static_cast<float>( Event.window.data1 );
          m_Height = static_cast<float>( Event.window.data2 );
        }
        return true;
        case SDL_KEYDOWN:
        case SDL_KEYUP:         OnKey( Event.key.keysym.sym, Event.type == SDL_KEYDOWN );         break;
        case SDL_MOUSEBUTTONDOWN:
        case SDL_MOUSEBUTTONUP:         break;
        case SDL_MOUSEMOTION          break;
        case SDL_MOUSEWHEEL          break;
    }
    return true;
  }
```

并非所有的案例标签都已实现，也并非所有的 SDL2 事件都被使用。我们将在后续章节根据需要使用这个路由。

在我们的示例中，我们使用一些有用的 OpenGL 包装器渲染一个旋转的盒子，可以隐藏移动版和桌面版 OpenGL 之间的差异。以下是`OnStart()`的代码，它将 OpenGL 的版本打印到系统日志中，并初始化顶点缓冲对象和着色器程序：

```java
  clPtr<clVertexAttribs> g_Box;
  clPtr<clGLVertexArray> g_BoxVA;
  clPtr<clGLSLShaderProgram> g_ShaderProgram;
  void OnStart( const std::string& RootPath )
  {
    LOGI( "Hello Android NDK!" );
    const char* GLVersion  = (const char*)LGL3->glGetString( GL_VERSION  );
    const char* GLVendor   = (const char*)LGL3->glGetString( GL_VENDOR   );
    const char* GLRenderer = (const char*)LGL3->glGetString( GL_RENDERER );
    LOGI( "GLVersion : %s\n", GLVersion );
    LOGI( "GLVendor  : %s\n", GLVendor  );
    LOGI( "GLRenderer: %s\n", GLRenderer );
```

首先，我们创建一个与 API 无关的盒子网格表示：

```java
    g_Box = clGeomServ::CreateAxisAlignedBox( LVector3(-1), LVector3(+1) );
```

然后，我们将其输入到 OpenGL 中，使用顶点缓冲对象创建一个顶点数组：

```java
    g_BoxVA = make_intrusive<clGLVertexArray>();
    g_BoxVA->SetVertexAttribs( g_Box );
```

着色器程序由包含顶点和片段着色器源代码的两个字符串变量构建而成：

```java
    g_ShaderProgram = make_intrusive<clGLSLShaderProgram>( g_vShaderStr, g_fShaderStr );
    LGL3->glClearColor( 0.1f, 0.0f, 0.0f, 1.0f );
    LGL3->glEnable( GL_DEPTH_TEST );
  }
```

下面是使用 GLSL 3.3 核心配置编写的着色器。使用模型-视图-投影矩阵变换顶点：

```java
  static const char g_vShaderStr[] = R"(
    uniform mat4 in_ModelViewProjectionMatrix;
    in vec4 in_Vertex;
    in vec2 in_TexCoord;
    out vec2 Coords;
    void main()
    {
      Coords = in_TexCoord.xy;
      gl_Position = in_ModelViewProjectionMatrix * in_Vertex;
    }
  )";
```

使用纹理坐标作为 RG 颜色分量来绘制盒子：

```java
  static const char g_fShaderStr[] = R"(
    in vec2 Coords;
    out vec4 out_FragColor;
    void main()
    {
      out_FragColor = vec4( Coords, 1.0, 1.0 );
    }
  )";
```

你可能已经注意到着色器的源代码不包含`#version`和`precision`行。这是因为`clGLSLShaderProgram`类对源代码进行了一些操作，以抽象不同版本 GLSL 之间的差异。我们将在后续段落熟悉这个类。在此之前，让我们看看`OnDrawFrame()`：

```java
  void OnDrawFrame()
  {
    static float Angle = 0;
    Angle += 0.02f;
    LGL3->glClear( GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT );
    mat4 Proj = Math::Perspective(
      45.0f, g_Window->GetAspect(), 0.4f, 2000.0f );
```

绕`(1, 1, 1)`轴旋转立方体：

```java
    LMatrix4 MV = LMatrix4::GetRotateMatrixAxis( Angle,
      vec3( 1, 1, 1 ) ) *
      mat4::GetTranslateMatrix( vec3( 0, 0, -5 ) );
    g_ShaderProgram->Bind();
    g_ShaderProgram->SetUniformNameMat4Array(
      "in_ModelViewProjectionMatrix", 1, MV * Proj );
    g_BoxVA->Draw( false );
  }
```

# OpenGL API 绑定

如你所见，前面提到的代码中所有的 OpenGL 调用都是通过`LGL3`前缀完成的。这是一个在`LGLAPI.h`中声明的名为`sLGLAPI`的结构，包含指向实际 OpenGL API 函数的指针：

```java
  struct sLGLAPI
  {
    sLGLAPI()
    {
      memset( this, 0, sizeof( *this ) );
    };
    PFNGLACTIVETEXTUREPROC        glActiveTexture;
    PFNGLATTACHSHADERPROC         glAttachShader;
    PFNGLBINDATTRIBLOCATIONPROC   glBindAttribLocation;
    PFNGLBINDBUFFERPROC           glBindBuffer;
    PFNGLBINDBUFFERBASEPROC       glBindBufferBase;
    PFNGLBINDFRAGDATALOCATIONPROC glBindFragDataLocation;
    ...
  }
```

`sLGLAPI`结构的字段在`LGL::GetAPI()`函数中设置。这个函数有两个不同的实现，一个是 Windows 的`LGL_Windows.h`，另一个是`LGL_Android.h`中为其他所有平台。区别在于 Windows 上的动态链接，如下代码所示：

```java
  void LGL::GetAPI( sLGLAPI* API )
  {
    API->glBlendFunc = ( PFNGLBLENDFUNCPROC )GetGLProc( API, "glBlendFunc" );
    API->glBufferData = ( PFNGLBUFFERDATAPROC )GetGLProc( API, "glBufferData" );
    API->glBufferSubData = ( PFNGLBUFFERSUBDATAPROC )GetGLProc( API, "glBufferSubData");
    ...
  }
```

所有其他平台都使用静态链接系统提供的 OpenGL 库：

```java
  void LGL::GetAPI( sLGLAPI* API )
  {
    API->glActiveTexture = &glActiveTexture;
    API->glAttachShader = &glAttachShader;
    API->glBindAttribLocation = &glBindAttribLocation;
    API->glBindBuffer = &glBindBuffer;
    ...
  }
```

当然，如果你使用特定供应商的 OpenGL 扩展，可以使用动态链接在任何平台上通过`glGetProcAddresss()`访问它们，这时`sLGLAPI`结构就显得非常方便：

这是我们在 OpenGL 之上的抽象层次最低的部分。有人可能会说这个所谓的层次什么也不做。这是不正确的。看看在 Android 上是如何获取`glClearDepth()`的指针的。出于某种原因，不是直接调用函数，而是一个存根：

```java
  API->glClearDepth = &Emulate_glClearDepth;
```

存根定义如下：

```java
  LGL_CALL void Emulate_glClearDepth( double Depth )
  {
    glClearDepthf( static_cast<float>( Depth ) );
  }
```

原因是 OpenGL ES 中没有`glClearDepth()`函数，它接受一个`float`参数，但 OpenGL 3 有这个函数。这种方式可以将移动设备和桌面 OpenGL 之间的 API 差异隐藏在薄薄的抽象层后面。使用这种技术，你可以透明地替换一个 OpenGL 枚举为另一个。可以透明地实现跟踪机制，将 OpenGL 函数参数的值打印到日志中。在将现有应用程序移植到没有图形调试器可用的平台时，这种技术至关重要（是的，我们在说你，黑莓）。我们将这个作为你的练习。

现在让我们深入了解高级抽象是如何实现的。

# 跨平台的 OpenGL 抽象概念

几何对象可以通过它们的表面来表示。在本章中，我们只讨论多边形图形，因此最重要的数据结构是*三角网格*。

就像数字音频一样，我们方便的 API 无关的数据结构在可以渲染之前需要转换成图形 API 的本地格式。让我们从 3D 空间中三角化几何的表示开始。

一个单一三角形可以通过三个顶点来指定。每个顶点至少存储其在 3D 空间中的位置，如下所示：

![跨平台的 OpenGL 抽象概念](img/image00223.jpeg)

在实现可移植渲染器的第一步，我们需要将几何存储分离出来，最简单的情况下，这只是带有属性和顶点的集合以及通过这些顶点构造图形原语的迭代顺序，与任何 API 特定的函数和数据类型分离。这种数据结构在`clVertexAttribs`类中实现：

```java
class clVertexAttribs: public iIntrusiveCounter
{
public:
  clVertexAttribs();
  explicit clVertexAttribs( size_t Vertices );
  void SetActiveVertexCount( size_t Count )
  {FActiveVertexCount = Count; }
  size_t GetActiveVertexCount() const
  { return FActiveVertexCount; }
```

这个方法返回一个包含指向实际顶点属性、位置、纹理坐标、法线和颜色的容器，可以输入到 OpenGL 顶点缓冲对象中：

```java
  const std::vector<const void*>& EnumerateVertexStreams() const;
  {
    FStreams[ L_VS_VERTEX   ] = &FVertices[0];
    FStreams[ L_VS_TEXCOORD ] = &FTexCoords[0];
    FStreams[ L_VS_NORMAL   ] = &FNormals[0];
    FStreams[ L_VS_COLORS   ] = &FColors[0];
    return FStreams;
  }
```

我们声明了一组辅助方法来生成几何数据：

```java
  void Restart( size_t ReserveVertices );
  void EmitVertexV( const vec3& Vec );
  void SetTexCoordV( const vec2& V );
  void SetNormalV( const vec3& Vec );
  void SetColorV( const vec4& Vec );
```

我们声明一组公共字段来存储我们的数据。顶点 3D 位置*x*，*y*，*z*声明如下：

```java
public:
  std::vector<vec3> FVertices;
```

纹理坐标`u`和`v`。这是我们顶点格式的局限性，因为有时纹理坐标可能包含超过两个通道。然而，对于我们的应用程序来说，这种限制是合适且可行的：

```java
  std::vector<vec2> FTexCoords;
```

顶点法线通常在对象空间中：

```java
  std::vector<vec3> FNormals;
```

顶点的 RGBA 颜色。如果你编写了正确的着色器，这个容器可以用于任何你想要的定制数据：

```java
  std::vector<vec4> FColors;
};
```

实现很简单；但是，我们建议在进一步操作之前查看`Geometry.cpp`和`Geometry.h`文件。

为了将有用的数据填充到`clVertexAttribs`的实例中，`clGeomServ`类中声明了一组静态方法：

```java
classlGeomServ
{
public:
  static clPtr<clVertexAttribs> CreateTriangle2D( float vX, float vY, float dX, float dY, float Z );
  static clPtr<clVertexAttribs> CreateRect2D( float X1, float Y1, float X2, float Y2, float Z,
    bool FlipTexCoordsVertical, int Subdivide );
  static void AddAxisAlignedBox( const clPtr<clVertexAttribs>& VA, const LVector3& Min, const LVector3& Max );
  static clPtr<clVertexAttribs> CreateAxisAlignedBox( const LVector3& Min, const LVector3& Max );
  static void AddPlane( const clPtr<clVertexAttribs>& VA, float SizeX, float SizeY, int SegmentsX, int SegmentsY, float Z );
  static clPtr<clVertexAttribs> CreatePlane( float SizeX, float SizeY, int SegmentsX, int SegmentsY, float Z );
};
```

所有`Create*()`方法创建一个新的几何图元并返回包含它的`clVertexAttribs`实例。以`Add`开头的方法将图元添加到现有的`clVertexAttribs`类实例中，假设它有足够的容量来存储新的图元。实现非常简单，可以在`Geometry.cpp`中找到。更复杂的几何生成例程将在后续章节中添加。

# 将几何数据提供给 OpenGL

要渲染`clVertexAttribs`的内容，我们需要将其数据转换为一组特定于 API 的缓冲区和 API 函数调用。这是通过在`clGLVertexArray`类中创建**顶点数组对象**（**VOA**）和**顶点缓冲区对象**（**VBO**）OpenGL 对象，并从`clVertexAttribs`获取内容来完成的：

```java
class clGLVertexArray: public iInusiveCounter
{
public:
  clGLVertexArray();
  virtual ~clGLVertexArray();
```

`Draw()`方法执行实际渲染，它是我们抽象层中可能进行渲染的最低级别：

```java
  void Draw( bool Wireframe ) const;
  void SetVertexAttribs(
    const clPtr<clVertexAttrs>& Attribs );
private:
  void Bind() const;
private:
  Luint FVBOID;
  Luint FVAOID;
```

这些指针实际上是顶点数据在顶点缓冲区内的偏移量：

```java
  std::vector<const void*> FAttribVBOOffset;
```

这些指针指向`clVertexAttribs`中的实际数据：

```java
  std::vector<const void*> FEnumeratedStreams;
  clPtr<clVertexAttribs> FAttribs;
};
```

这个类的实现包括一些簿记工作以及调用 OpenGL 函数。构造函数和析构函数分别初始化和销毁 VOA 和 VBO 的句柄：

```java
clGLVertexArray::clGLVertexArray()
: FVBOID( 0 ),
  FVAOID( 0 ),
  FAttribVBOOffset( L_VS_TOTAL_ATTRIBS ),
  FEnumeratedStreams( L_VS_TOTAL_ATTRIBS ),
  FAttribs( nullptr )
{
```

在 Windows 上，我们使用 OpenGL 4，其中使用顶点数组对象是强制性的：

```java
  #if dined( _WIN32 )
    LGL3->glGenVertexArrays( 1, &FVAOID );
  #endif
}
```

销毁操作以特定于平台的方式进行：

```java
clGLVertexArray::~clGLVertexArray()
{
  LGL3->glDeleteBuffers( 1, &FVBOID );
  #if defined( _WIN32 )
    LGL3->glDeleteVertexArrays( 1, &FVAOID );
  #endif
}
```

私有方法`Bind()`将此顶点数组对象设置为 OpenGL 渲染管线的源顶点流：

```java
void clGLVertexArray::Bind() const
{
  LGL3->glBindBuffer( GL_ARRAY_BUFFER, FVBOID );
  LGL3->glVertexAttribPointer( L_VS_VERTEX, L_VS_VEC_COMPONENTS[ 0 ], GL_FLOAT, GL_FALSE, 0, FAttribVBOOffset[ 0 ] );
  LGL3->glEnableVertexAttribArray( L_VS_VERTEX );
```

绑定并启用顶点位置后，我们启用每个额外的非空属性：

```java
  for ( int i = 1; i < L_VS_TOTAL_ATTRIBS; i++ )
  {
    LGL3->glVertexAttribPointer( i,
      L_VS_VEC_COMPONENTS[ i ],
      GL_FLOAT, GL_FALSE, 0, FAttribVBOOffset[ i ] );
    FAttribVBOOffset[ i ] ?
      LGL3->glEnableVertexAttribArray( i ) :
      LGL3->glDisableVertexAttribArray( i );
  }
}
```

`Draw()`方法绑定 VOA 并调用`glDrawArrays()`来渲染几何图形：

```java
void clGLVertexArray::Draw( bool Wireframe ) const
{
  #if defined( _WIN32 )
    LGL3->glBindVertexArray( FVAOID );
  #else
    Bind();
  #endif
```

第一个参数是图元的类型。如果`Wireframe`参数为`true`，我们告诉 OpenGL 将数据视为一系列线，每个连续的点对一条线。如果参数为`false`，则每个连续的点三元组被用作三角形的三个顶点：

```java
  LGL3->glDrawArrays(
    Wireframe ? GL_LINE_LOOP : GL_TRIANGLES, 0,
    static_cast<GLsizei>( FAttribs->GetActiveVertexCount() ) );
}
```

`SetVertexAttribs()`成员函数将几何数据附加到`GLVertexArray`并重新创建所有必需的 OpenGL 对象：

```java
void clGLVertexArray::SetVertexAttribs(  const clPtr<clVertexAttribs>& Attribs )
{
  FAttribs = Attribs;
```

分配指针后，我们获取一个指向各个顶点属性流的指针数组：

```java
  FEnumeratedStreams = FAttribs->EnumerateVertexStreams();
  LGL3->glDeleteBuffers( 1, &FVBOID );
  size_t VertexCount = FAttribs->FVertices.size();
  size_t DataSize = 0;
```

检查每个流是否包含任何数据，并相应地更新顶点缓冲区的大小：

```java
  for ( int i = 0; i != L_VS_TOTAL_ATTRIBS; i++ )
  {
    FAttribVBOOffset[ i ] = ( void* )DataSize;
    DataSize += FEnumeratedStreams[i] ?
      sizeof( float ) * L_VS_VEC_COMPONENTS[ i ] * VertexCount : 0;
  }
```

之后，我们创建一个新的顶点缓冲区对象，该对象将包含几何数据：

```java
  LGL3->glGenBuffers( 1, &FVBOID );
  LGL3->glBindBuffer( GL_ARRAY_BUFFER, FVBOID );
```

这里最重要的事情是将数据从`clVertexAttribs`对象复制到 GPU 内存中。这是通过使用`nullptr`作为缓冲区指针调用`glBufferData()`来分配存储来完成的：

```java
  LGL3->glBufferData( GL_ARRAY_BUFFER, DataSize, nullptr, GL_STREAM_DRAW );
```

你可以在[`www.khronos.org/opengles/sdk/docs/man3/html/glBufferData.xhtml`](https://www.khronos.org/opengles/sdk/docs/man3/html/glBufferData.xhtml)找到更多关于`glBufferData()`的信息。

这里是对每个非空属性数组的后续`glBufferSubData()`调用，这些属性数组包括顶点位置、纹理坐标、法线和颜色：

```java
  for ( int i = 0; i != L_VS_TOTAL_ATTRIBS; i++ )
  {
    if ( FEnumeratedStreams[i] )
    {
      LGL3->glBufferSubData( GL_ARRAY_BUFFER, ( GLintptr )FAttribVBOOffset[ i ], FAttribs->GetActiveVertexCount() * sizeof( float ) * L_VS_VEC_COMPONENTS[ i ], FEnumeratedStreams[ i ] );
    }
  }
```

绑定对于 VAO 和非 VAO 版本来说有些特定：

```java
  #if defined( _WIN32 )
    LGL3->glBindVertexArray( FVAOID );
    Bind();
    LGL3->glBindVertexArray( 0 );
  #endif
}
```

VAO 版本可以在 OpenGL ES 3 上使用。然而，未经修改的代码也可以在 OpenGL ES 2 上运行。

# 着色器程序

桌面和移动 OpenGL 版本都将着色器程序作为其渲染管道的一部分。仅提供几何图形是不够的。然而，为了创建可移植的渲染子系统，我们应该处理 GLSL 3.00 ES 和 GLSL 3.30 Core 之间的几个重要区别。

让我们从`uniform`值的声明开始：

```java
struct sUniform
{
public:
  explicit sUniform( const std::string& e)
  : FName( e )
  , FLocation( -1 )
  {};
  sUniform( int Location, const std::string& e)  : FName( e )
  , FLocation( Location )
  {};
  std::string FName;
  Lint FLocation;
};
```

这个类存储了在链接着色器程序中统一变量的名称和位置。着色器程序类的结构如下所示：

```java
class clGLSLShaderProgram: public iIntrusiveCounr
{
public:
```

构造函数以顶点和片段着色器的源代码作为参数：

```java
  clGLSLShaderProgram( const std::string& VShader, const std::string& FShader );
  virtual ~clGLSLShaderProgram();
```

`Bind()`方法在使用前绑定着色器程序：

```java
  void Bind();
```

一组处理统一变量的方法：

```java
  Lint CreateUniform( const std::string& Name );
  void SetUniformNameFloat( const std::string& Name, const float Float );
  void SetUniformNameFloatArray( const std::string& Name, int Count, const float& Float );
  void SetUniformNameVec3Array( const std::string& Name, int Count, const LVector3& Vector );
  void SetUniformNameVec4Array( const std::string& Name, int Count, const LVector4& Vector );
  void SetUniformNameMat4Array( const std::string& Name, int Count, const LMatrix4& Matr );
private:
```

使用附加的着色器链接程序：

```java
  bool RelinkShaderProgram();
```

我们需要绑定属性和片段数据的默认位置。这将在以下方法中完成：

```java
  void BindDefaultLocations( Luint ProgramID )
  {
    LGL3->glBindAttribLocation( ProgramID, L_VS_VERTEX, "in_Vertex" );
    LGL3->glBindAttribLocation( ProgramID, L_VS_TEXCOORD, "in_TexCoord" );
    LGL3->glBindAttribLocation( ProgramID, L_VS_NORMAL, "in_Normal" );
    LGL3->glBindAttribLocation( ProgramID, L_VS_COLORS, "in_Color" );
    LGL3->glBindFragDataLocation( ProgramID, 0, "out_FragColor" );
  }
```

它将着色器变量`in_Vertex`、`in_Normal`、`in_TexCoord`和`in_Color`绑定到适当的顶点流。你可以在你的 GLSL 代码中声明并使用这些`in`变量。`out_FragColor`输出变量与片段着色器的单一输出相关联。

编译并将着色器附加到此着色器程序：

```java
  Luint AttachShaderID( Luint Target, const std::string& ShaderCode, Luint OldShaderID );
```

检查编译和链接过程中发生的任何错误并记录：

```java
  bool CheckStatus( Luint ObjectID, Lenum Target, const std::string& Message ) const;
```

此方法从链接的着色器程序检索所有统一变量，并将它们作为`sUniform`结构存储在`FUniforms`容器中：

```java
  void RebindAllUniforms();
private:
  std::string FVertexShader;
  std::string FFragmentShader;
  Luint FVertexShaderID;
  Luint FFragmentShaderID;
```

此着色程序中所有活动的统一变量集合存储如下：

```java
  std::vector<sUniform> FUniforms;
```

OpenGL 着色器程序和着色器标识符存储在以下字段中：

```java
  Luint FProgramID;
  std::vector<Luint> FShaderID;
};
clGLSLShaderProgram::clGLSLShaderProgram(
  const std::string& VShader, const std::string& FShader )
: FVertexShader( VShader )
, FFragmentShader( FShader )
, FUniforms()
, FProgramID( 0 )
, FVertexShaderID( 0 )
, FFragmentShaderID( 0 )
{
  RelinkShaderProgram();
}
```

我们可以如下销毁所有创建的 OpenGL 对象：

```java
clGLSLShaderProgram::~clGLSLShaderProgram()
{
  LGL3->glDeleteProgram( FProgramID );
  LGL3->glDeleteShader( FVertexShaderID );
  LGL3->glDeleteShader( FFragmentShaderID );
}
```

让我们看看如何创建着色器对象并将其附加到着色器程序：

```java
Luint clGLSLShaderProgram::AttachShaderID( Luint Target,
  const std::string& ShaderCode, Luint OldShaderID )
{
```

由于我们使用 OpenGL ES 3 和 OpenGL 4，着色器的版本应相应指定：

```java
  #if defined( USE_OPENGL_4 )
    std::string ShaderStr = "#version 330 core\n";
  #else
    std::string ShaderStr = "#version 300 es\n";
    ShaderStr += "precision highp float;\n";
    ShaderStr += "#define USE_OPENGL_ES_3\n";
  #endif
    ShaderStr += ShaderCode;
```

生成的着色器提交给 OpenGL API 函数：

```java
    Luint Shader = LGL3->glCreateShader( Target );
    const char* Code = ShaderStr.c_str();
    LGL3->glShaderSource( Shader, 1, &Code, nullptr );
    LGL3->glCompileShader( Shader );
```

检查编译状态，并记录编译代码时检测到的任何错误。如果新着色器编译失败，这段代码将回退到之前编译的着色器。你可以使用前几章中的文件系统类来实现动态着色器程序的重新加载：

```java
    if ( !CheckStatus( Shader, GL_COMPILE_STATUS, "Shader wasn''t compiled:" ) )
    {
      LGL3->glDeleteShader( Shader );
      return OldShaderID;
    }
    if ( OldShaderID )
    {
      LGL3->glDeleteShader( OldShaderID );
    }
    return Shader;
  }
```

错误检查和记录实现起来并不复杂，是必须的：

```java
  bool clGLSLShaderProgram::CheckStatus( Luint ObjectID, Lenum Target, const std::string& Message ) const
  {
    Lint   SuccessFlag = 0;
    Lsizei Length      = 0;
    Lsizei MaxLength   = 0;
    if ( LGL3->glIsProgram( ObjectID ) )
    {
      LGL3->glGetProgramiv( ObjectID, Target, &SuccessFlag );
      LGL3->glGetProgramiv( ObjectID, GL_INFO_LOG_LENGTH, &MaxLength );
```

着色器程序错误消息的缓冲区在栈上动态分配：

```java
      char* Log = ( char* )alloca( MaxLength );
      LGL3->glGetProgramInfoLog( ObjectID, MaxLength, &Length, Log );
      if ( *Log ) { LOGI( "Program info:\n%s\n", Log ); }
    }
    else if ( LGL3->glIsShader( ObjectID ) )
    {
      LGL3->glGetShaderiv( ObjectID, Target, &SuccessFlag );
      LGL3->glGetShaderiv( ObjectID, GL_INFO_LOG_LENGTH, &MaxLength );
```

以类似的方式处理着色器对象：

```java
      char* Log = ( char* )alloca( MaxLength );
      LGL3->glGetShaderInfoLog( ObjectID, MaxLength, &Length, Log );
      if ( *Log ) { LOGI( "Shader info:\n%s\n", Log ); }
    }
    return SuccessFlag != 0;
  }
```

当顶点和片段着色器对象都成功编译后，将重新链接着色器程序：

```java
  bool clGLSLShaderProgram::RelinkShaderProgram()
  {
    Luint ProgramID = LGL3->glCreateProgram();
    FVertexShaderID = AttachSaderID( GL_VERTEX_SHADER, FVertexShader, FVertexShaderID );
    if ( FVertexShaderID )
    { LGL3->glAttachShader( ProgramID, FVertexShaderID ); }
    FFragmentShaderID = AttachShaderID( GL_FRAGMENT_SHADER, FFragmentShader, FFragmentShaderID );
    if ( FFragmentShaderID )
    { LGL3->glAttachShader( ProgramID, FFragmentShaderID ); }
```

绑定所有默认顶点属性的定位：

```java
    BindDefaultLocations( ProgramID );
    LGL3->glLinkProgram( ProgramID );
    if ( !CheckStatus( ProgramID, GL_LINK_STATUS, "Program wasn''t linked" ) )
    {
      LOGI( "INTERNAL ERROR: Error while shader relinking" );
      return false;
    }
```

在这一点上，我们知道着色器程序已成功链接，我们可以将其作为渲染管道的一部分使用。用以下代码替换旧程序：

```java
    LGL3->glDeleteProgram( FProgramID );
    FProgramID = ProgramID;
```

从链接的程序中检索活动统一变量的列表并存储它们：

```java
    RebindAllUniforms();
```

将纹理采样器绑定到它们的默认位置。你可以在任何时候添加更多的纹理单元：

```java
    LGL3->glUniform1i( LGL3->glGetUniformLocation(FProgramID, "Texture0"), 0);
    LGL3->glUniform1i( LGL3->glGetUniformLocation(FProgramID, "Texture1"), 1);
    LGL3->glUniform1i( LGL3->glGetUniformLocation(FProgramID, "Texture2"), 2);
    LGL3->glUniform1i( LGL3->glGetUniformLocation(FProgramID, "Texture3"), 3);
    return true;
  }
```

在`RebindAllUniforms()`方法中完成统一变量的排队：

```java
  void clGLSLShaderProgram::RebindAllUniforms()
  {
    Bind();
    FUniforms.clear();
    Lint ActiveUniforms;
    char Buff[256];
    LGL3->glGetProgramiv( FProgramID,
      GL_ACTIVE_UNIFORMS, &ActiveUniforms );
    for ( int i = 0; i != ActiveUniforms; ++i )
    {
      Lsizei Length;
      Lint Size;
      Lenum Type;
      LGL3->glGetActiveUniform( FProgramID, i,
        sizeof( Buff ), &Length, &Size, &Type, Buff );
      std::string Name( Buff, Length );
```

`sUniform`对象被构造并推入容器以供将来访问。作为改进，可以排序向量或用`std::map`替换以允许更快地访问：

```java
      sUniform Uniform( Name );
      Uniform.FLocation = LGL3->glGetUniformLocation( FProgramID, Name.c_str() );
      FUniforms.push_back( Uniform );
    }
  }
```

`SetUniform*()`方法组在 GLSL 着色器程序中设置一个命名统一变量的值。这些方法通过调用`CreateUniform()`获取统一变量的句柄，然后使用`glUniform*()`系列 OpenGL 函数之一来设置新值。字符串名称可以用于着色器的快速原型设计。如果你想要追求性能，可以事先使用`CreateUniform()`成员函数获取统一变量的位置，并使用该值与对应的`SetUniform*()`调用：

```java
  void clGLSLShaderProgram::SetUniformNameFloat( const std::string& Name, const float Float )
  {
    Lint Loc = CreateUniform( Name );
    LGL3->glUniform1f( Loc, Float );
  }
  void clGLSLShaderProgram::SetUniformNamoatArray( const std::string& Name, int Count, const float& Float )
  {
    Lint Loc = CreateUniform( Name );
    LGL3->glUniform1fv( Loc, Count, &Float );
  }
```

向量被转换为指针。注意以下技巧，`ToFloatPtr()`方法返回一个指向向量`x`分量的指针。当这个向量被包装到一个向量数组中时，我们也拥有指向数组开头的指针。因此，`Count`参数非常有意义，我们可以将向量数组传递给这个方法：

```java
  void void clGLSLShaderProgram::SetUniformNameec3Array(     const std::string& Name, int Count, const LVector3& Vector )
  {
    Lint Loc = CreateUniform( Name );
    LGL3->glUniform3fv( Loc, Count, Vector.ToFloatPtr() );
  }
  void clGLSLShaderProgram::SetUniformNameVec4Array( const std::string& Name, int Count, const LVector4& Vector )
  {
    Lint Loc = CreateUniform( Name );
    LGL3->glUniform4fv( Loc, Count, Vector.ToFloatPtr() );
  }
```

矩阵的方法与前面的方法不同，只是参数类型不同：

```java
  void clGLSLShaderProgram::SetUniformNameMat4Array( const std::string& Name, int Count, const LMatrix4& Matrix )
  {
    Lint Loc = CreateUniform( Name );
    LGL3->glUniformMatrix4fv( Loc, Count, false,
    Matrix.ToFloatPtr() );
  }
```

在`SetUniform*()`中使用的`CreateUniform()`方法在`FUniforms`容器中进行搜索，并返回统一变量的 OpenGL 标识符：

```java
  Lint clGLSLShaderProgram::CreateUniform( const std::string& Name )
  {
    for ( size_t i = 0; i != FUniforms.size(); ++i )
    if ( FUniforms[i].FName == Name )
    return FUniforms[i].FLocation;
    return -1;
  }
```

这个方法用于任何名称都是安全的，因为对于在着色器程序中找不到的统一变量返回的`-1`值被 OpenGL 接受并忽略。

`Bind()`方法将着色器程序绑定到当前的 OpenGL 渲染上下文：

```java
  void clGLSLShaderProgram::Bind()
  {
    LGL3->glUseProgram( FProgramID );
  }
```

在更复杂的应用程序中，缓存当前绑定的着色器程序的值是有意义的，并且只有当值发生变化时才调用底层 API。

# 纹理

我们需要包装的最后一个组件是纹理。纹理由`clGLTexture`类的实例表示：

```java
  class clGLTexture: public iIntrusivounter
  {
  public:
    clGLTexture();
    virtual ~clGLTexture();
```

将纹理绑定到一个指定的 OpenGL 纹理单元：

```java
    void Bind( int TextureUnit ) const;
```

从 API 无关的位图中加载纹理像素：

```java
    void LoadFromBitmap( const clPtr<clBitmap>& Bitmap );
```

设置纹理坐标的钳制模式：

```java
    void SetClamping( Lenum Clamping );

```

处理纹理的数据格式和尺寸：

```java
  private:
    void SetFormat( Lenum Target, Lenum InternalFormat, Lenum Format, int Width, int Height );
    Luint FTexID;
    Lenum FInternalFormat;
    Lenum FFormat;
  };
```

该实现相当紧凑。下面就是代码：

```java
  clGLTexturelGLTexture()
  : FTexID( 0 )
  , FIntelFormat( 0 )
  , FFormat( 0 )
  {
  }
  clGLTexture::~clGLTexture()
  {
    if ( FTexID ) { LGL3->glDeleteTextures( 1, &FTexID ); }
  }
  void clGLTexture::Bind( int TextureUnit ) const
  {
    LGL3->glActiveTexture( GL_TEXTURE0 + TextureUnit );
    LGL3->glBindTexture( GL_TEXTURE_2D, FTexID );
  }
```

我们可以设置纹理的格式而不上传任何像素。如果你想要将纹理附加到帧缓冲对象，这非常有用。我们将在第八章，*编写渲染引擎*中使用这个功能来实现渲染到纹理的功能：

```java
  void clGLTexture::SetFormat( Lenum Target, Lenum InternalFormat, Lenum Format, int Width, int Height )
  {
    if ( FTexID )
    {
      LGL3->glDeleteTextures( 1, &FTexID );
    }
    LGL3->glGenTextures( 1, &FTexID );
    LGL3->glBindTexture( GL_TEXTURE_2D, FTexID );
    LGL3->glTexParameterf( GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR );
    LGL3->glTexParameterf( GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR );
    LGL3->glTexParameteri( GL_TEXTURE_2D, GL_TEXTURE_WRAP_S, GL_CLAMP_TO_EDGE );
    LGL3->glTexParameteri( GL_TEXTURE_2D, GL_TEXTURE_WRAP_T, GL_CLAMP_TO_EDGE );
    LGL3->glTexImage2D( GL_TEXTURE_2D, 0, InternalFormat, Width, Height, 0, Format, GL_UNSIGNED_BYTE, nullptr );
    LGL3->glBindTexture( GL_TEXTURE_2D, 0 );
  }
  void clGLTexture::SetClamping( Lenum Clamping )
  {
    Bind( 0 );
```

按如下方式更新`S`和`T`的钳制模式：

```java
    LGL3->glTexParameteri( GL_TEXTURE_2D,
      GL_TEXTURE_WRAP_S, Clamping );
    LGL3->glTexParameteri( GL_TEXTURE_2D,
      GL_TEXTURE_WRAP_T, Clamping );
  }
  void clGLTexture::LoadFromBitmap( const clPtr<clBitmap>& Bitmap )
  {
    if ( !Bitmap ) { return; }
    if ( !FTexID )
    {
      LGL3->glGenTextures( 1, &FTexID );
    }
```

根据位图参数选择合适的 OpenGL 纹理格式：

```java
    ChooseInternalFormat( Bitmap->FBitmapParams, &FFormat, &FInternalFormat );
    Bind( 0 );
```

将默认过滤模式设置为`GL_LINEAR`以避免构建 mipmap 链：

```java
    LGL3->glTexParameteri( GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR );
    LGL3->glTexParameteri( GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR );
    int Width = Bitmap->GetWidth();
    int Height = Bitmap->GetHeight();
```

某些 OpenGL ES 实现不允许零尺寸的纹理（是的，我们说的是你，Vivante）：

```java
    if ( !Width || !Height ) { return; }
```

将原始位图数据加载到 OpenGL 中：

```java
    LGL3->glTexImage2D( GL_TEXTURE_2D, 0, FInternalFormat, Width, Height, 0, FFormat, GL_UNSIGNED_BYTE, Bitmap->FBitmapData );
  }
```

到目前为止，我们已经拥有足够的工具来使用 OpenGL 构建可移植的移动应用程序。本章的示例应用程序`1_GLES`在 Windows 和 Android 上渲染了一个彩色的旋转立方体：

![纹理](img/image00224.jpeg)

Windows 版本可以通过执行`>make all -j16 -B`来编译。可以通过调用以下命令来构建 Android 的`.apk`包： 

```java
>ndk-build -j16 -B
>ant debug

```

# 总结

我们学习了如何将原始的 OpenGL 调用封装在一个轻薄的抽象层中，以隐藏 OpenGL ES 3 和 OpenGL 4 之间的许多差异。现在，让我们进入下一章，学习如何使用 OpenGL 和本章展示的类来实现基本的图形用户界面渲染。
