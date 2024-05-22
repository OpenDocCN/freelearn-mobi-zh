# 第八章. 编写渲染引擎

在前面的章节中，我们学习了如何组织一个在移动和桌面 OpenGL 之上的薄抽象层。现在，我们可以进入实际的渲染领域，并使用这层来实现一个能够渲染从文件加载的几何图形的 3D 渲染框架，使用材质、光线和阴影。

# 场景图

场景图是一种常用的数据结构，用于构建空间图形场景的分层表示。第六章《OpenGL ES 3.1 与跨平台渲染》中介绍类的主要局限在于，它们缺乏对整个场景的信息。这些类的用户必须对变换、状态更改和依赖关系进行临时记账，使得实现和支持任何相对复杂的场景变得非常具有挑战性。此外，除非可以访问当前帧的整个场景信息，否则许多渲染优化都无法完成。

在我们当前的低级实现中，我们使用`clVertexArray`类描述所有可见实体，并通过`clGLSLShaderProgram`类访问着色器程序进行渲染，这需要手动绑定矩阵和着色器参数。让我们学习如何将这些属性组合到一个更高级的数据结构中。首先，我们将从场景图节点开始。

`clSceneNode`类包含了局部和全局变换以及一个子节点向量。这些字段是受保护的，我们通过设置器和获取器来访问它们：

```java
class clSceneNode: public iIntrusiveCounter
{
protected:
  mat4 m_LocalTransform;
  mat4 m_GlobalTransform;
  std::vector< clPtr<clSceneNode> > m_ChildNodes;
```

当我们需要层次结构时，我们必须区分节点的全局和局部变换。从用户的角度来看，局部变换很容易理解。这定义了一个节点相对于其父节点在分层空间结构中的位置和方向。全局变换用于渲染几何体。*本质上*，它将几何体从模型坐标系转换到世界坐标系。局部变换可以直观地手动修改，而全局变换应随后重新评估。`clSceneNode`的构造函数设置默认的变换值：

```java
public:
  clSceneNode():
  m_LocalTransform( mat4::Identity() ),
  m_GlobalTransform( mat4::Identity() ) {}
```

`clSceneNode`类提供了设置器和获取器，用于访问和修改变换矩阵。实现很简单。然而，它相当繁琐，因此这里只引用了局部变换矩阵的方法。查看源代码`1_SceneGraphRenderer`以获取完整信息：

```java
  void SetLocalTransform( const mat4& Mtx )
  { m_LocalTransform = Mtx; }
  const mat4& GetLocalTransformConst() const
  { return m_LocalTransform; }
  mat4& GetLocalTransform()
  { return m_LocalTransform; };
```

此外，我们需要一种方法将子节点添加到此场景节点。我们当前的实现非常简单：

```java
  virtual void Add( const clPtr<clSceneNode>& Node )
  {
    m_ChildNodes.push_back( Node );
  }
```

然而，这种方法将来可以扩展，以允许某些优化。例如，我们可以一旦添加新节点，就将场景图的某些部分标记为“脏”。这将允许我们保留从场景图构建的帧间渲染队列。

有时，需要直接设置全局变换矩阵。例如，如果你想在 3D 应用程序中使用物理模拟库。完成后，应该重新计算局部变换。`RecalculateLocalFromGlobal()`方法为层次结构中的每个节点计算相对局部变换。对于根节点，局部和全局变换是重合的。对于子节点，每个全局变换矩阵必须与父节点的逆全局变换相乘：

```java
  void RecalculateLocalFromGlobal()
  {
    mat4 ParentInv = m_GlobalTransform.GetInversed();
    for ( auto& i : m_ChildNodes )
    {
```

我们将父节点的全局逆变换与当前节点的全局变换相乘：

```java
    i->SetLocalTransform( ParentInv * i->GetGlobalTransform() );
```

过程在层次结构中逐级重复：

```java
    i->RecalculateLocalFromGlobal();
  }
}
```

`clSceneNode`声明中还有一个更有趣的东西。这就是纯虚拟方法`AcceptTraverser()`。为了渲染场景图，使用了名为*访问者设计模式*的技术（[`en.wikipedia.org/?title=Visitor_pattern`](https://en.wikipedia.org/?title=Visitor_pattern)）：

```java
virtual void AcceptTraverser(iSceneTraverser* Traverser) = 0;
```

`iSceneTraverser`接口用于定义场景图上的不同操作。由于不同类型的场景图节点的数量是有限且恒定的，我们可以通过实现`iSceneTraverser`接口简单地添加新操作：

```java
class iSceneTraverser: public iIntrusiveCounter
{
public:
  virtual void Traverse( clPtr<clSceneNode> Node );
  virtual void Reset() = 0;
```

接口被声明为`clSceneNode`所有后代的友元，以允许直接访问这些类的字段：

```java
  friend class clSceneNode;
  friend class clMaterialNode;
  friend class clGeometryNode;
protected:
  virtual void PreAcceptSceneNode( clSceneNode* Node ) {};
  virtual void PostAcceptSceneNode( clSceneNode* Node ) {};
  virtual void PreAcceptMaterialNode( clMaterialNode* Node ) {};
  virtual void PostAcceptMaterialNode( clMaterialNode* Node ){};
  virtual void PreAcceptGeometryNode( clGeometryNode* Node ) {};
  virtual void PostAcceptGeometryNode( clGeometryNode* Node ){};
};
```

`Traverse()`的实现被所有遍历器共享。它重置遍历器并调用虚拟方法`clSceneNode::AcceptTraverser()`：

```java
void iSceneTraverser::Traverse( clPtr<clSceneNode> Node )
{
  if ( !Node ) return;
  Reset();
  Node->AcceptTraverser( this );
}
```

在`iSceneTraverser`的声明中，你可以看到两种额外的场景节点类型。`clSceneNode`对象的树可以保持空间变换的层次结构。显然，这还不足以渲染任何东西。为此，我们至少需要几何数据和一个材质。

为了这个目的，我们再声明两个类：`clMaterialNode`和`clGeometryNode`。

本章的第一个示例中，材质将只包含环境光和漫反射颜色（[`en.wikipedia.org/wiki/Phong_shading`](https://en.wikipedia.org/wiki/Phong_shading)）：

```java
struct sMaterial
{
public:
  sMaterial()
  : m_Ambient( 0.2f )
  , m_Diffuse( 0.8f )
  , m_MaterialClass()
  {}
  vec4 m_Ambient;
  vec4 m_Diffuse;
```

`m_MaterialClass`字段包含一个材质标识符，可以用来区分特殊材质，例如，为粒子渲染启用 alpha 透明度：

```java
  std::string m_MaterialClass;
};
```

现在，可以声明一个材质场景节点。它是一个简单的数据容器：

```java
class clMaterialNode: public clSceneNode
{
public:
  clMaterialNode() {};
  sMaterial& GetMaterial() { return m_Material; }
  const sMaterial& GetMaterial() const { return m_Material; }
  void SetMaterial( const sMaterial& Mtl ) { m_Material = Mtl; }
  virtual void AcceptTraverser(iSceneTraverser* Traverser) override;
private:
  sMaterial m_Material;
};
```

让我们看看`AcceptTraverser()`方法的实现。它非常简单且高效：

```java
void clMaterialNode::AcceptTraverser( iSceneTraverser* Traverser )
{
  Traverser->PreAcceptSceneNode( this );
  Traverser->PreAcceptMaterialNode( this );
  AcceptChildren( Traverser );
  Traverser->PostAcceptMaterialNode( this );
  Traverser->PostAcceptSceneNode( this );
}
```

几何节点更为复杂。这是因为`clVertexAttribs`中的 API 独立几何数据表示应该被输入到`clGLVertexArray`的实例中：

```java
class clGeometryNode: public clSceneNode
{
public:
  clGeometryNode() {};
  clPtr<clVertexAttribs> GetVertexAttribs() const
  { return m_VertexAttribs; }
  void SetVertexAttribs( const clPtr<clVertexAttribs>& VA )
  { m_VertexAttribs = VA; }
```

这里，我们以懒惰的方式将几何数据输入到 OpenGL 中：

```java
  clPtr<clGLVertexArray> GetVA() const
  {
    if ( !m_VA )
    {
      m_VA = make_intrusive<clGLVertexArray>();
      m_VA->SetVertexAttribs( m_VertexAttribs );
    }
    return m_VA;
  }
  virtual void AcceptTraverser(iSceneTraverser* Traverser) override;
private:
  clPtr<clVertexAttribs> m_VertexAttribs;
  mutable clPtr<clGLVertexArray> m_VA;
};
```

`AcceptTraverser()`的实现与`clMaterialNode`内部的实现非常相似。只需查看捆绑的源代码即可。

如你所见，这一大堆场景节点类只不过是一个简单的数据容器。实际操作在遍历器类中发生。`iSceneTraverser`的第一个实现是`clTransformUpdateTraverser`类，它更新每个场景节点的全局变换——即相对于图根的变换：

```java
class clTransformUpdateTraverser: public iSceneTraverser
{
private:
  std::vector<mat4> m_ModelView;
```

私有字段`m_ModelView`包含一个作为`std::vector`实现的矩阵栈。这个栈的顶部元素是节点的当前全局变换。`Reset()`方法清除栈，并在栈上推入单位矩阵，这后来用作根场景节点的全局变换：

```java
public:
  clTransformUpdateTraverser(): m_ModelView() {}
  virtual void Reset() override
  {
    m_ModelView.clear();
    m_ModelView.push_back( mat4::Identity() );
  }
```

`PreAcceptSceneNode()` 方法将当前全局变换的新值推送到`m_ModelView`栈上，然后将其作为传入节点的全局变换使用：

```java
protected:
  virtual void PreAcceptSceneNode( clSceneNode* Node ) override
  {
    m_ModelView.push_back( Node->GetLocalTransform() * m_ModelView.back() );
    Node->SetGlobalTransform( m_ModelView.back() );
  }
```

`PostAcceptSceneNode()` 方法从栈中弹出最顶层的、现在未使用的矩阵：

```java
  virtual void PostAcceptSceneNode( clSceneNode* Node ) override
  {
    m_ModelView.pop_back();
  }
};
```

这个遍历器在每一帧开始时使用，在渲染任何几何体之前：

```java
clTransformUpdateTraverser g_TransformUpdateTraverser;
clPtr<clSceneNode> g_Scene;
g_TransformUpdateTraverser.Traverse( g_Scene );
```

我们现在几乎准备好进行实际渲染了。为此，我们需要将场景图线性化为渲染操作的向量。让我们看看`ROP.h`文件。每个渲染操作都是独立的几何体、材质和一系列变换矩阵。所需的矩阵存储在`sMatrices`结构中：

```java
struct sMatrices
{
```

投影、视图和模型矩阵从外部状态明确设置。

```java
  mat4 m_ProjectionMatrix;
  mat4 m_ViewMatrix;
  mat4 m_ModelMatrix;
```

使用`UpdateMatrices()`方法更新光照和着色所需的其它矩阵：

```java
  mat4 m_ModelViewMatrix;
  mat4 m_ModelViewMatrixInverse;
  mat4 m_ModelViewProjectionMatrix;
  mat4 m_NormalMatrix;
  void UpdateMatrices()
  {
    m_ModelViewMatrix = m_ModelMatrix * m_ViewMatrix;
    m_ModelViewMatrixInverse = m_ModelViewMatrix.GetInversed();
    m_ModelViewProjectionMatrix = m_ModelViewMatrix * m_ProjectionMatrix;
    m_NormalMatrix = mat4( m_ModelViewMatrixInverse.ToMatrix3().GetTransposed() );
  }
};
```

这个结构可以根据需要轻松地扩展额外的矩阵。此外，将这个结构的值打包到统一缓冲区对象中非常方便。

现在，我们的渲染操作可以如下所示：

```java
class clRenderOp
{
public:
  clRenderOp( const clPtr<clGeometryNode>& G, const clPtr<clMaterialNode>& M):
    m_Geometry(G), m_Material(M) {}
  void Render( sMatrices& Matrices ) const;
  clPtr<clGeometryNode> m_Geometry;
  clPtr<clMaterialNode> m_Material;
};
```

`clRenderOp::Render()`的最小化实现在`ROP.cpp`中可以找到。那里定义了两个着色器。首先是一个通用的顶点着色器，用于将法线变换到世界空间：

```java
static const char g_vShaderStr[] = R"(
  uniform mat4 in_ModelViewProjectionMatrix;
  uniform mat4 in_NormalMatrix;
  uniform mat4 in_ModelMatrix;
  in vec4 in_Vertex;
  in vec2 in_TexCoord;
  in vec3 in_Normal;
  out vec2 v_Coords;
  out vec3 v_Normal;
  out vec3 v_WorldNormal;
  void main()
  {
    v_Coords = in_TexCoord.xy;
    v_Normal = mat3(in_NormalMatrix) * in_Normal;
    v_WorldNormal = ( in_ModelMatrix * vec4( in_Normal, 0.0 ) ).xyz;
    gl_Position = in_ModelViewProjectionMatrix * in_Vertex;
  }
)";
```

然后，一个片段着色器使用单个方向光源进行简单的逐像素光照，该光源指向与相机相同的方向：

```java
static const char g_fShaderStr[] = R"(
  in vec2 v_Coords;
  in vec3 v_Normal;
  in vec3 v_WorldNormal;
  out vec4 out_FragColor;
  uniform vec4 u_AmbientColor;
  uniform vec4 u_DiffuseColor;
  void main()
  {
    vec4 Ka = u_AmbientColor;
    vec4 Kd = u_DiffuseColor;
```

相机是静态定位的，光照在世界空间中进行：

```java
    vec3 L = normalize( vec3( 0.0, 0.0, 1.0 ) );
    float d = clamp( dot( L, normalize(v_WorldNormal) ), 0.0, 1.0 );
    vec4 Color = Ka + Kd * d;
    out_FragColor = Color;
  }
)";
```

静态全局变量保存了使用前述代码中提到的着色器链接的着色器程序：

```java
clPtr<clGLSLShaderProgram> g_ShaderProgram;
```

实际的渲染代码更新所有矩阵，设置着色器程序的参数并渲染几何体：

```java
void clRenderOp::Render( sMatrices& Matrices ) const
{
  if ( !g_ShaderProgram )
  {
    g_ShaderProgram = make_intrusive<clGLSLShaderProgram>( g_vShaderStr, g_fShaderStr );
  }
  Matrices.m_ModelMatrix = this->m_Geometry->GetGlobalTransformConst();
  Matrices.UpdateMatrices();
```

一旦渲染操作和统一变量的数量增加，以下代码段将成为瓶颈。它可以通过预缓存统一位置来进行优化：

```java
  g_ShaderProgram->Bind();
  g_ShaderProgram->SetUniformNameVec4Array( "u_AmbientColor", 1, this->m_Material->GetMaterial().m_Ambient );
  g_ShaderProgram->SetUniformNameVec4Array( "u_DiffuseColor", 1, this->m_Material->GetMaterial().m_Diffuse );
  g_ShaderProgram->SetUniformNameMat4Array( in_ProjectionMatrix", 1, Matrices.m_ProjectionMatrix );
  g_ShaderProgram->SetUniformNameMat4Array( "in_ViewMatrix", 1, Matrices.m_ViewMatrix );
  g_ShaderProgram->SetUniformNameMat4Array( "in_ModelMatrix", 1, Matrices.m_ModelMatrix );
  g_ShaderProgram->SetUniformNameMat4Array( "in_NormalMatrix", 1, Matrices.m_NormalMatrix );
  g_ShaderProgram->SetUniformNameMat4Array( "in_ModelViewMatrix", 1, Matrices.m_ModelViewMatrix );
  g_ShaderProgram->SetUniformNameMat4Array( "in_ModelViewProjectionMatrix", 1, Matrices.m_ModelViewProjectionMatrix );
  this->m_Geometry->GetVA()->Draw( false );
}
```

让我们将场景图转换为渲染操作的向量，这样我们就可以看到实际渲染的图像。这是由`clROPsTraverser`类完成的：

```java
class clROPsTraverser: public iSceneTraverser
{
private:
  std::vector<clRenderOp> m_RenderQueue;
  std::vector<clMaterialNode*> m_Materials;
public:
  clROPsTraverser()
  : m_RenderQueue()
  , m_Materials() {}
```

在构建新的渲染操作队列之前清除所有内容：

```java
  virtual void Reset() override
  {
    m_RenderQueue.clear();
    m_Materials.clear();
  }
```

返回对最近构造的队列的引用：

```java
  virtual const std::vector<clRenderOp>& GetRenderQueue() const
  {
    return m_RenderQueue;
  }
```

现在，我们实现了`iSceneTraverser`接口。这里的大多数方法将是空的：

```java
protected:
  virtual void PreAcceptSceneNode( clSceneNode* Node ) override
  {
  }
  virtual void PostAcceptSceneNode( clSceneNode* Node ) override
  {
  }
```

当下一个几何节点进入时，使用材质堆栈中最顶层的材质并创建一个新的渲染操作：

```java
  virtual void PreAcceptGeometryNode( clGeometryNode* Node ) override
  {
    if ( !m_Materials.size() ) return;
    m_RenderQueue.emplace_back( Node, m_Materials.front() );
  }
  virtual void PostAcceptGeometryNode( clGeometryNode* Node ) override
  {
  }
```

每当有`clMaterialNode`进入时，都会更新材质堆栈：

```java
  virtual void PreAcceptMaterialNode( clMaterialNode* Node ) override
  {
    m_Materials.push_back( Node );
  }
  virtual void PostAcceptMaterialNode( clMaterialNode* Node ) override
  {
    m_Materials.pop_back();
  }
};
```

最后，这个框架现在可以用来渲染实际的 3D 图形。示例场景在`1_SceneGraphRenderer/main.cpp`中创建。首先，创建我们场景的根节点：

```java
g_Scene = make_intrusive<clSceneNode>();
```

创建一个红色材质并将其绑定到材质场景节点：

```java
auto MaterialNode = make_intrusive<clMaterialNode>();
sMaterial Material;
Material.m_Ambient = vec4( 0.1f, 0.0f, 0.0f, 1.0f );
Material.m_Diffuse = vec4( 0.9f, 0.0f, 0.0f, 1.0f );
MaterialNode->SetMaterial( Material );
```

让我们创建一个以原点为中心的立方体：

```java
auto VA = clGeomServ::CreateAxisAlignedBox( vec3(-1), vec3(+1) );
g_Box= make_intrusive<clGeometryNode>();
g_Box->SetVertexAttribs( VA );
MaterialNode->Add( g_Box );
```

并将其添加到场景中：

```java
g_Scene->Add( MaterialNode );
```

渲染直接且非常通用：

```java
void OnDrawFrame()
{
  static float Angle = 0.0f;
```

绕其对角线旋转立方体：

```java
  g_Box->SetLocalTransform( mat4::GetRotateMatrixAxis( Angle, vec3( 1, 1, 1 ) ) );
  Angle += 0.01f;
```

更新节点的全局变换并构建一个渲染队列：

```java
  g_TransformUpdateTraverser.Traverse( g_Scene );
  g_ROPsTraverser.Traverse( g_Scene );
  const auto& RenderQueue = g_ROPsTraverser.GetRenderQueue();
```

设置矩阵。摄像机目前只提供了一个虚拟实现，它返回一个单位视图矩阵：

```java
  sMatrices Matrices;
  Matrices.m_ProjectionMatrix = Math::Perspective( 45.0f, g_Window->GetAspect(), 0.4f, 2000.0f );
  Matrices.m_ViewMatrix = g_Camera.GetViewMatrix();
```

在渲染帧之前清除屏幕：

```java
  LGL3->glClear( GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT );
  LGL3->glEnable( GL_DEPTH_TEST );
```

遍历渲染队列并渲染所有内容：

```java
  for ( const auto& ROP : RenderQueue )
  {
    ROP.Render( Matrices );
  }
}
```

最终的图像将如下面的截图所示：

![场景图](img/image00225.jpeg)

让我们扩展场景图渲染示例，加入灯光和阴影，并确保在 Android 上一切正常工作。

# 灯光和着色

为了渲染灯光和阴影，我们需要扩展前文展示的方法。接下来我们要讨论的代码示例是`2_ShadowMaps`。阴影映射将使用投影阴影映射（[`en.wikipedia.org/wiki/Shadow_mapping`](https://en.wikipedia.org/wiki/Shadow_mapping)）。这样，场景从光线的视角渲染到一个离屏深度缓冲区。接下来，像往常一样渲染场景，并将每个片段投影到光的阴影映射上，并将与光相关的深度值与构建的阴影映射中的值进行比较。如果深度值大于阴影映射中相应的深度值，则片段处于阴影中。要进行离屏渲染，我们需要回顾第六章中引入的*OpenGL ES 3.1 与跨平台渲染*，并为其添加一个帧缓冲区抽象。

`clGLFrameBuffer`类在`GLFrameBuffer.h`中声明：

```java
class clGLFrameBuffer: public iIntrusiveCounter
{
public:
  clGLFrameBuffer():
  FFrameBuffer(0),
  FColorBuffer(),
  FDepthBuffer(),
  FColorBuffersParams(),
  FHasDepthBuffer( false )
  {}
  virtual ~clGLFrameBuffer();
```

`InitRenderTargetV()`方法接受一个包含宽度、高度和每个通道位数整数值的向量：

```java
  virtual void InitRenderTargetV( const ivec4& WidthHeightBitsPerChannel, const bool HasDepthBuffer );
```

此方法提供了对私有数据成员的访问，这些成员包括宽度、高度和每个通道的位数，它们被传递到`InitRenderTargetV()`中：

```java
  virtual ivec4 GetParameters() const
  {
    return FColorBuffersParams;
  }
```

帧缓冲区最重要的能力是能够将其内容作为纹理提供——一个颜色纹理和一个深度纹理：

```java
  virtual clPtr<clGLTexture> GetColorTexture() const
  {
    return FColorBuffer;
  }
  virtual clPtr<clGLTexture> GetDepthTexture() const
  {
    return FDepthBuffer;
  }
```

`Bind()`方法将此帧缓冲区设置为当前的 OpenGL 帧缓冲区：

```java
  virtual void Bind( int TargetIndex ) const;
  virtual void UnBind() const;
```

受保护的`CheckFrameBuffer()`方法用于根据 OpenGL 规范检查帧缓冲区的完整性：

```java
protected:
  void CheckFrameBuffer() const;
```

类的私有部分包含一个 OpenGL 缓冲区标识符，分别用于颜色和深度纹理的两个`clGLTexture`对象，以及包含帧缓冲区参数的两个字段：

```java
private:
  GLuint FFrameBuffer;
  clPtr<clGLTexture> FColorBuffer;
  clPtr<clGLTexture> FDepthBuffer;
  ivec4 FColorBuffersParams;
  bool FHasDepthBuffer;
};
```

对于 Android 和其他平台正确构建帧缓冲区需要一些工作和仔细选择参数。让我们看看`InitRenderTargetV()`成员函数的实现：

```java
void clGLFrameBuffer::InitRenderTargetV( const ivec4& WidthHeightBitsPerChannel, const bool HasDepthBuffer )
{
```

首先，我们在私有数据成员中存储帧缓冲区的参数。这些值稍后用于视口宽高比计算：

```java
  FColorBuffersParams = WidthHeightBitsPerChannel;
  FHasDepthBuffer = HasDepthBuffer;
```

接下来，我们将调用 OpenGL 函数创建帧缓冲区对象：

```java
  LGL3->glGenFramebuffers( 1, &FFrameBuffer );
```

创建帧缓冲区对象后，我们可以将其绑定为当前帧缓冲区以设置其属性：

```java
  Bind( 0 );
```

创建并附加一个颜色纹理到帧缓冲区：

```java
  FColorBuffer = make_intrusive<clGLTexture>();
  int Width = FColorBuffersParams[0];
  int Height = FColorBuffersParams[1];
```

这里 Android 和桌面实现之间的唯一区别在于缓冲数据格式的选择。OpenGL 4 Core Profile 要求显式指定内部格式和深度格式的位数，而 OpenGL ES 3 则分别希望通用的`GL_RGBA`和`GL_DEPTH_COMPONENT`。我们以平台特定的方式声明两个常量：

```java
  #if defined( OS_ANDROID )
    const Lenum InternalFormat = GL_RGBA;
    auto DepthFormat = GL_DEPTH_COMPONENT;
  #else
    const Lenum InternalFormat = GL_RGBA8;
    auto DepthFormat = GL_DEPTH_COMPONENT24;
  #endif
```

我们将调用`clGLTexture`的`SetFormat()`方法来设置颜色纹理的格式：

```java
  FColorBuffer->SetFormat( GL_TEXTURE_2D, InternalFormat, GL_RGBA, Width, Height );
```

`AttachToCurrentFB()`方法将创建的颜色纹理附加到当前绑定的帧缓冲区。`GL_COLOR_ATTACHMENT0`的值指定了一个 OpenGL 附着点：

```java
  FColorBuffer->AttachToCurrentFB( GL_COLOR_ATTACHMENT0 );
```

阴影图包含深度缓冲区的值，因此我们按需以下列方式创建深度纹理：

```java
  if ( HasDepthBuffer )
  {
    FDepthBuffer = make_intrusive<clGLTexture>();
```

深度缓冲区应与颜色缓冲区具有相同的尺寸：

```java
    int Width = FColorBuffersParams[0];
    int Height = FColorBuffersParams[1];
```

深度缓冲区的设置与颜色缓冲区类似：

```java
    FDepthBuffer->SetFormat( GL_TEXTURE_2D, DepthFormat, GL_DEPTH_COMPONENT, Width, Height );
    FDepthBuffer->AttachToCurrentFB( GL_DEPTH_ATTACHMENT );
  }
```

为了确保正确操作，我们将检查错误代码并解绑缓冲区：

```java
  CheckFrameBuffer();
  UnBind();
}
```

`CheckFrameBuffer()`成员函数使用 OpenGL 调用来检查当前帧缓冲区的状态：

```java
void clGLFrameBuffer::CheckFrameBuffer() const
{
  Bind( 0 );
  GLenum FBStatus = LGL3->glCheckFramebufferStatus( GL_FRAMEBUFFER );
```

将错误代码转换为字符串并将其打印到系统日志中：

```java
  switch ( FBStatus )
  {
    case GL_FRAMEBUFFER_COMPLETE: // Everything's OK
      break;
    case GL_FRAMEBUFFER_INCOMPLETE_ATTACHMENT:
      LOGI( "GL_FRAMEBUFFER_INCOMPLETE_ATTACHMENT" );
      break;
    case GL_FRAMEBUFFER_INCOMPLETE_MISSING_ATTACHMENT:
      LOGI("GL_FRAMEBUFFER_INCOMPLETE_MISSING_ATTACHMENT" );
      break;
```

OpenGL ES 缺少 OpenGL 的一些功能。这里，我们省略它们以使代码可移植：

```java
    #if !defined(OS_ANDROID)
      case GL_FRAMEBUFFER_INCOMPLETE_DRAW_BUFFER:
        LOGI( "GL_FRAMEBUFFER_INCOMPLETE_DRAW_BUFFER" );
        break;
      case GL_FRAMEBUFFER_INCOMPLETE_READ_BUFFER:
        LOGI( "GL_FRAMEBUFFER_INCOMPLETE_READ_BUFFER" );
        break;
    #endif
      case GL_FRAMEBUFFER_UNSUPPORTED:
        LOGI( "GL_FRAMEBUFFER_UNSUPPORTED" );
        break;
      default:
        LOGI( "Unknown framebuffer error: %x", FBStatus );
  }
```

默认情况下，不打印任何内容：

```java
  UnBind();
}
```

让我们继续前进，以便我们可以使用这个类。

# 光照和光照节点

将光源表示为 3D 场景的一部分非常方便。当我们提到“3D 场景”时，我们指的是场景图。为了将光源附加到场景图，我们需要一个特殊的节点。以下是持有指向具有所有光照属性的`iLight`的`clLightNode`类：

```java
class clLightNode: public clSceneNode
{
public:
  clLightNode() {}
  clPtr<iLight> GetLight() const
  {
    return m_Light;
  }
  void SetLight( const clPtr<iLight>& L )
  {
    m_Light = L;
  }
  virtual void AcceptTraverser( iSceneTraverser* Traverser )
    override;
  clPtr<iLight> m_Light;
};
```

`AcceptTraverser()`方法与`clGeometryNode`和`clMaterialNode`中的方法类似。但这次，我们将调用`iSceneTraverser`的`PreAcceptLightNode()`和`PostAcceptLightNode()`方法：

```java
void clLightNode::AcceptTraverser( iSceneTraverser* Traverser )
{
  Traverser->PreAcceptSceneNode( this );
  Traverser->PreAcceptLightNode( this );
  AcceptChildren( Traverser );
  Traverser->PostAcceptLightNode( this );
  Traverser->PostAcceptSceneNode( this );
}
```

这种新的场景节点类型迫使我们扩展`iSceneTraverser`的接口：

```java
protected:
friend class clLightNode;
virtual void PreAcceptLightNode( clLightNode* Node ) {}
virtual void PostAcceptLightNode( clLightNode* Node ) {}
```

现在遍历器可以以特殊方式处理光照节点。我们将利用这一能力在每帧基础上维护场景中的活动光照列表。

`iLight`类封装了光参数。它计算了光源所需的投影和视图矩阵，将它们传递给着色器程序并持有一个阴影贴图。我们应该注意到，为可能未使用的光源保持一个初始化的阴影贴图肯定不是最优的。在我们这个最小化的示例中，最少可以做到将阴影贴图的创建推迟到真正需要的时候。在我们的示例中，我们只处理聚光灯。然而，这种方法可以很容易地扩展到方向光和点光源：

```java
class iLight: public iIntrusiveCounter
{
public:
  iLight():
  m_Ambient(0.2f),
  m_Diffuse(0.8f),
  m_Position(0),
  m_Direction(0.0f, 0.0f, 1.0f),
```

如果你想要实现多种类型的光源，建议将这个字段推送到表示聚光灯的类中。由于我们的示例只有单一类型的光源，在这里放置这个值是合理的折中：

```java
  m_SpotOuterAngle(45.0f)
  {}
```

`UpdateROPUniforms()`方法更新了阴影贴图渲染所需的着色器程序中的所有 uniform。在完成`iLight`之后，将详细描述`clMaterialSystem`类：

```java
  void UpdateROPUniforms( const std::vector<clRenderOp>& ROPs, const clPtr<clMaterialSystem>& MatSys, const clPtr<clLightNode>& LightNode ) const;
```

为了从光的角度渲染场景，我们需要计算两个矩阵。第一个是标准的*look-at*矩阵，定义了光的视图矩阵；第二个是透视投影矩阵，定义了光的截锥体：

```java
  mat4 GetViewForShadowMap() const
  {
    return Math::LookAt( m_Position, m_Position + m_Direction, vec3( 0, 0, 1 ) );
  }
  mat4 GetProjectionForShadowMap() const
  {
    float NearCP = 0.1f;
    float FarCP = 1000.0f;
    return Math::Perspective( 2.0f * this->m_SpotOuterAngle, 1.0f, NearCP, FarCP );
  }
```

`GetShadowMap()`函数返回一个延迟初始化的阴影贴图缓冲区，该缓冲区附加到此光源：

```java
  clPtr<clGLFrameBuffer> iLight::GetShadowMap() const
  {
    if ( !m_ShadowMap )
    {
      m_ShadowMap = make_intrusive<clGLFrameBuffer>();
      m_ShadowMap->InitRenderTargetV(
        { ivec4(1024, 1024, 8, 0) },
        true
      );
    }
    return m_ShadowMap;
  }
```

光源的性质包括在简单光照模型中使用的漫反射和环境颜色，用于视图矩阵计算的位置和方向，以及聚光灯锥角：

```java
public:
  vec4 m_Ambient;
  vec4 m_Diffuse;
  vec3 m_Position;
  vec3 m_Direction;
  float m_SpotOuterAngle;
```

最后，我们声明一个持有阴影贴图的帧缓冲区：

```java
  mutable clPtr<clGLFrameBuffer> m_ShadowMap;
};
```

让我们看看着色器程序的 uniform 是如何更新的。这发生在`UpdateROPUniforms()`中，该函数在每个渲染操作之前对每个阴影贴图进行渲染时被调用：

```java
void iLight::UpdateROPUniforms( const std::vector<clRenderOp>& ROPs, const clPtr<clMaterialSystem>& MatSys, const clPtr<clLightNode>& LightNode ) const
{
  mat4 LightMV = this->GetViewForShadowMap();
  mat4 LightProj = GetProjectionForShadowMap();
  mat4 Mtx = LightNode->GetGlobalTransformConst();
  vec3 Pos = ( Mtx * vec4( this->m_Position, 1.0f ) ).XYZ();
  vec3 Dir = ( Mtx * vec4( this->m_Direction, 0.0f ) ).XYZ();
  auto AmbientSP = MatSys->GetShaderProgramForPass( ePass_Ambient );
  AmbientSP->Bind();
  AmbientSP->SetUniformNameVec3Array( "u_LightPos", 1, Pos );
  AmbientSP->SetUniformNameVec3Array( "u_LightDir", 1, Dir );
  auto LightSP = MatSys->GetShaderProgramForPass( ePass_Light );
  LightSP->Bind();
  LightSP->SetUniformNameVec3Array( "u_LightPos", 1, Pos );
  LightSP->SetUniformNameVec3Array( "u_LightDir", 1, Dir );
  LightSP->SetUniformNameVec4Array( "u_LightDiffuse", 1, this->m_Diffuse );
  mat4 ScaleBias = GetProjScaleBiasMat();
  mat4 ShadowMatrix = ( Mtx * LightMV * LightProj ) * ScaleBias;
  LightSP->SetUniformNameMat4Array( "in_ShadowMatrix", 1, ShadowMatrix );
  this->GetShadowMap()->GetDepthTexture()->Bind( 0 );
}
```

`GetProjScaleBiasMat()`辅助例程返回一个缩放矩阵，该矩阵将[-1..1]标准化设备坐标映射到[0..1]范围：

```java
mat4 GetProjScaleBiasMat()
{
  return mat4( 0.5f, 0.0f, 0.0f, 0.0f, 0.0f, 0.5f, 0.0f, 0.0f, 0.0f, 0.0f, 0.5f, 0.0f, 0.5f, 0.5f, 0.5f, 1.0 );
}
```

在这段代码中提到的`clMaterialSystem`类需要一些额外的解释。

# 材质系统

在我们之前的示例`1_SceneGraphRenderer`中，我们使用单个着色器程序来渲染场景中的所有对象。现在，我们的渲染器将变为多通道。我们需要创建阴影贴图，然后渲染带阴影的对象并计算光照。这是通过在三个不同的渲染通道中使用三个不同的着色器程序完成的。为了区分通道，我们定义了`ePass`枚举如下：

```java
enum ePass
{
  ePass_Ambient,
  ePass_Light,
  ePass_Shadow
};
```

为了基于通道和材质属性处理不同的着色器程序，我们实现了`clMaterialSystem`类：

```java
class clMaterialSystem: public iIntrusiveCounter
{
public:
  clMaterialSystem();
```

`GetShaderProgramForPass()`方法返回指定通道在`std::map`中存储的着色器程序：

```java
  clPtr<clGLSLShaderProgram> GetShaderProgramForPass(ePass P)
  {
    return m_ShaderPrograms[ P ];
  }
private:
  std::map<ePass, clPtr<clGLSLShaderProgram>> m_ShaderPrograms;
};
```

这个类的构造函数创建了渲染所需的每个着色器程序，并将其插入到映射中：

```java
clMaterialSystem::clMaterialSystem()
{
  m_ShaderPrograms[ ePass_Ambient ] = make_intrusive<clGLSLShaderProgram>( g_vShaderStr, g_fShaderAmbientStr );
  m_ShaderPrograms[ ePass_Light   ] = make_intrusive<clGLSLShaderProgram>( g_vShaderStr, g_fShaderLightStr );
  m_ShaderPrograms[ ePass_Shadow  ] = make_intrusive<clGLSLShaderProgram>( g_vShaderShadowStr, g_fShaderShadowStr );
}
```

### 注意

在此示例中，映射可以用一个简单的 C 风格数组替换。但是，稍后我们将使用不同材质类型和不同的着色器程序，所以使用映射会更合适。

与之前的示例一样，每个着色器的源代码存储在一个静态字符串变量中。这次，代码有点复杂。顶点着色器源代码在环境传递和每个光线传递之间共享：

```java
static const char g_vShaderStr[] = R"(
  uniform mat4 in_ModelViewProjectionMatrix;
  uniform mat4 in_NormalMatrix;
  uniform mat4 in_ModelMatrix;
  uniform mat4 in_ShadowMatrix;
  in vec4 in_Vertex;
  in vec2 in_TexCoord;
  in vec3 in_Normal;
  out vec2 v_Coords;
  out vec3 v_Normal;
  out vec3 v_WorldNormal;
  out vec4 v_ProjectedVertex;
  out vec4 v_ShadowMapCoord;
```

同一个函数在 C++代码中被用来将值从[-1..1]范围转换到[0..1]范围：

```java
  mat4 GetProjScaleBiasMat()
  {
    return mat4( 
      0.5, 0.0, 0.0, 0.0, 
      0.0, 0.5, 0.0, 0.0, 
      0.0, 0.0, 0.5, 0.0, 
      0.5, 0.5, 0.5, 1.0 );
  }
```

值传递给后续的片段着色器：

```java
  void main()
  {
    v_Coords = in_TexCoord.xy;
    v_Normal = mat3(in_NormalMatrix) * in_Normal;
    v_WorldNormal = ( in_ModelMatrix * vec4( in_Normal, 0.0 ) ).xyz;
    v_ProjectedVertex = GetProjScaleBiasMat() * in_ModelViewProjectionMatrix * in_Vertex;
    v_ShadowMapCoord = in_ShadowMatrix * in_ModelMatrix * in_Vertex;
    gl_Position = in_ModelViewProjectionMatrix * in_Vertex;
  }
)";
```

这是环境传递的片段着色器。只需将环境色输出到帧缓冲区，我们就完成了：

```java
static const char g_fShaderAmbientStr[] = R"(
  in vec2 v_Coords;
  in vec3 v_Normal;
  in vec3 v_WorldNormal;
  out vec4 out_FragColor;
  uniform vec4 u_AmbientColor;
  uniform vec4 u_DiffuseColor;
  void main()
  {
    out_FragColor = u_AmbientColor;
  }
)";
```

每个光线传递的片段着色器根据光线的参数和阴影映射计算实际的光照和着色。这就是为什么它比我们之前的着色器要长得多：

```java
static const char g_fShaderLightStr[] = R"(
  in vec2 v_Coords;
  in vec3 v_Normal;
  in vec3 v_WorldNormal;
  in vec4 v_ProjectedVertex;
  in vec4 v_ShadowMapCoord;
  out vec4 out_FragColor;
  uniform vec4 u_AmbientColor;
  uniform vec4 u_DiffuseColor;
  uniform vec3 u_LightPos;
  uniform vec3 u_LightDir;
  uniform vec4 u_LightDiffuse;
  uniform sampler2D Texture0;
```

阴影是使用称为*百分比接近过滤*的技术计算得出的。如果我们使用简单的阴影映射方法，得到的阴影将有很多锯齿。**百分比接近过滤**（**PCF**）的理念是在当前像素周围的阴影映射中进行采样，并将其深度与所有样本进行比较。通过平均比较的结果（而不是采样的结果），我们可以得到光与影之间更平滑的边缘。我们的示例使用了带有 26 个抽头的 5 X 5 PCF 滤波器：

```java
  float PCF5x5( const vec2 ShadowCoord, float Depth )
  {
    float Size = 1.0 / float( textureSize( Texture0, 0 ).x );
    float Shadow =( Depth >= texture( Texture0, ShadowCoord ).r ) ? 1.0 : 0.0;
    for ( int v=-2; v<=2; v++ ) for ( int u=-2; u<=2; u++ )
    {
       Shadow += ( Depth >= texture( Texture0, ShadowCoord + Size * vec2(u, v) ).r ) ? 1.0 : 0.0;
    }
    return Shadow / 26.0;
  }
```

这是评估给定片段是否在阴影中的函数：

```java
  float ComputeSpotLightShadow()
  {
```

进行透视除法，将阴影映射投影到物体上：

```java
    vec4 ShadowCoords4 = v_ShadowMapCoord / v_ShadowMapCoord.w;
    if ( ShadowCoords4.w > 0.0 )
    {
      vec2 ShadowCoord = vec2( ShadowCoords4 );
      float DepthBias = -0.0002;
      float ShadowSample = 1.0 - PCF5x5( ShadowCoord, ShadowCoords4.z + DepthBias );
```

`DepthBias`系数用于防止阴影痘痘。以下是同一场景的两次渲染，一次是零`DepthBias`（左），一次是`-0.0002`（右）：

![材质系统](img/image00226.jpeg)

通常，这需要手动调整，并应该是光线参数的一部分。查看以下链接，了解更多关于如何改善阴影的想法：

[`msdn.microsoft.com/en-us/library/windows/desktop/ee416324(v=vs.85).aspx`](https://msdn.microsoft.com/en-us/library/windows/desktop/ee416324(v=vs.85).aspx)。

现在，乘以系数并返回结果值：

```java
      float ShadowCoef = 0.3;
      return ShadowSample * ShadowCoef;
    }
    return 1.0;
  }
```

现在，我们可以根据实际的光线方向及其阴影映射计算一个简单的光照模型：

```java
  void main()
  {
    vec4 Kd = u_DiffuseColor * u_LightDiffuse;
    vec3 L = normalize( u_LightDir );
    vec3 N = normalize( v_WorldNormal );
    float d = clamp( dot( -L, N ), 0.0, 1.0 );
    vec4 Color = Kd * d * ComputeSpotLightShadow();
    Color.w = 1.0;
    out_FragColor = Color;
  }
)";
```

要构建上一个着色器中使用的阴影映射，我们需要一个额外的渲染传递。对于每个光线，使用以下顶点和片段着色器：

```java
static const char g_vShaderShadowStr[] = R"(
  uniform mat4 in_ModelViewProjectionMatrix;
  in vec4 in_Vertex;
  void main()
  {
    gl_Position = in_ModelViewProjectionMatrix * in_Vertex;
  }
)";
static const char g_fShaderShadowStr[] = R"(
  out vec4 out_FragColor;
  void main()
  {
    out_FragColor = vec4( 1, 1, 1, 1 );
  }
)";
```

现在我们可以渲染一个更美观的图像，包含所有阴影和更精确的光照。让我们看看`2_ShadowMaps/main.cpp`文件。

# 演示应用程序和渲染技术

新代码最重要的部分在`OnDrawFrame()`方法中。它使用`clForwardRenderingTechnique`类来渲染场景。让我们看看`Technique.cpp`文件。

辅助函数`RenderROPs()`用于渲染渲染操作的向量：

```java
void RenderROPs( sMatrices& Matrices, const std::vector<clRenderOp>& RenderQueue, ePass Pass )
{
  for ( const auto& ROP : RenderQueue )
  {
    ROP.Render( Matrices, g_MatSys, Pass );
  }
}
```

现在，所有传递都可以用这个函数来描述。看看`clForwardRenderingTechnique::Render()`函数。首先，让我们构建两个渲染队列，一个用于不透明物体，一个用于透明物体。透明物体是指其材质类别为字符串`Particle`的物体。我们将在下一章中使用透明物体：

```java
m_TransformUpdateTraverser.Traverse( Root );
m_ROPsTraverser.Traverse( Root );
const auto& RenderQueue = m_ROPsTraverser.GetRenderQueue();
auto RenderQueue_Opaque = Select( RenderQueue, []( const clRenderOp& ROP )
    {
      return ROP.m_Material->GetMaterial().m_MaterialClass != "Particle";
    } );
    auto RenderQueue_Transparent = Select( RenderQueue, []( const clRenderOp& ROP )
    {
      return ROP.m_Material->GetMaterial().m_MaterialClass == "Particle";
    } 
  );
```

为着色器准备矩阵并清除 OpenGL 缓冲区：

```java
  sMatrices Matrices;
  Matrices.m_ProjectionMatrix = Proj;
  Matrices.m_ViewMatrix = View;
  Matrices.UpdateMatrices();
  LGL3->glClearColor( 0.0f, 0.0f, 0.0f, 0.0f );
  LGL3->glClear( GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT );
  LGL3->glEnable( GL_DEPTH_TEST );
```

现在，使用它们的环境色渲染所有对象。这就是全部内容，环境传递不需要任何光照。作为副产品，我们将得到一个充满值的 Z 缓冲区，因此在后续传递中我们可以禁用深度写入：

```java
  LGL3->glDepthFunc( GL_LEQUAL );
  LGL3->glDisablei( GL_BLEND, 0 );
  RenderROPs( Matrices, RenderQueue_Opaque, ePass_Ambient, MatSys );
```

对于后续的每个光照传递过程，我们需要从场景中获取一个光照向量。从遍历器中获取它，并更新所有的阴影贴图：

```java
  auto Lights = g_ROPsTraverser.GetLights();
  UpdateShadowMaps( Lights, RenderQueue );
```

`UpdateShadowMaps()`函数遍历光照节点向量，并将阴影投射器渲染到相应的阴影贴图中：

```java
  void UpdateShadowMaps( const std::vector<clLightNode*>& Lights, const std::vector<clRenderOp>& ROPs )
  {
    LGL3->glDisable( GL_BLEND );
    for ( size_t i = 0; i != Lights.size(); i++ )
    {
      sMatrices ShadowMatrices;
      clPtr<iLight> L = Lights[ i ]->GetLight();
      clPtr<clGLFrameBuffer> ShadowBuffer = L->GetShadowMap();
```

绑定并清除阴影贴图帧缓冲区：

```java
      ShadowBuffer->Bind( 0 );
      LGL3->glClearColor( 0, 0, 0, 1 );
      LGL3->glClear( GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT );
```

光源知道其投影和视图矩阵。这段代码相当通用，可以扩展用于包括具有多个视锥体的光源在内的光类型：

```java
      LMatrix4 Proj = L->GetProjectionForShadowMap();
      LMatrix4 MV = L->GetViewForShadowMap();
```

在着色器程序中更新 uniform 变量：

```java
      ShadowMatrices.m_ViewMatrix = MV;
      ShadowMatrices.m_ProjectionMatrix = Proj;
      ShadowMatrices.UpdateMatrices();
      L->UpdateROPUniforms( ROPs, g_MatSys, Lights[i] );
```

渲染到阴影贴图并解绑帧缓冲区：

```java
      RenderROPs( ShadowMatrices, ROPs, ePass_Shadow );
      ShadowBuffer->UnBind();
    }
  }
```

所有的阴影贴图现在都准备好在渲染代码中使用。让我们继续`OnDrawFrame()`函数。每个光照传递会累积所有光源的光照，如下所示：

```java
  LGL3->glDepthFunc( GL_EQUAL );
  LGL3->glBlendFunc( GL_ONE, GL_ONE );
  LGL3->glEnablei( GL_BLEND, 0 );
  for ( const auto& L : Lights )
  {
    L->GetLight()->UpdateROPUniforms( RenderQueue, MatSys, L );
    RenderROPs( Matrices, RenderQueue, ePass_Light, MatSys );
  }
```

最后但同样重要的是，为透明对象渲染环境光照：

```java
  LGL3->glBlendFunc(GL_SRC_ALPHA, GL_ONE);
  LGL3->glDepthFunc(GL_LESS);
  LGL3->glEnablei(GL_BLEND, 0);
  LGL3->glDepthMask( GL_FALSE );
  RenderROPs( Matrices, RenderQueue_Transparent, ePass_Ambient, MatSys );
```

不要忘记重置 OpenGL 状态。扩展渲染器的一个好主意是将深度测试、深度掩码、混合模式等状态封装到管道状态对象中，并且只在状态改变时更新管道状态。如果您想将示例扩展到完整的渲染代码，这个改进是必须的：

```java
  LGL3->glDepthMask( GL_TRUE );
```

我们已经涵盖了所有低级渲染代码。让我们提高一个层次，看看如何构建一个场景。

## 场景构建

我们的测试场景在`main()`中构建，过程如下所示。首先实例化全局对象：

```java
  g_MatSys = make_intrusive<clMaterialSystem>();
  g_Scene = make_intrusive<clSceneNode>();
  g_Canvas = make_intrusive<clGLCanvas>( g_Window );
```

之后，设置材质和材质节点：

```java
  auto CubeMaterialNode = make_intrusive<clMaterialNode>();
  {
    sMaterial Material;
    Material.m_Ambient = vec4( 0.2f, 0.0f, 0.0f, 1.0f );
    Material.m_Diffuse = vec4( 0.8f, 0.0f, 0.0f, 1.0f );
    CubeMaterialNode->SetMaterial( Material );
  }
  auto PlaneMaterialNode = make_intrusive<clMaterialNode>();
  {
    sMaterial Material;
    Material.m_Ambient = vec4( 0.0f, 0.2f, 0.0f, 1.0f );
    Material.m_Diffuse = vec4( 0.0f, 0.8f, 0.0f, 1.0f );
    PlaneMaterialNode->SetMaterial( Material );
  }
  auto DeimosMaterialNode = make_intrusive<clMaterialNode>();
  {
    sMaterial Material;
    Material.m_Ambient = vec4( 0.0f, 0.0f, 0.2f, 1.0f );
    Material.m_Diffuse = vec4( 0.0f, 0.0f, 0.8f, 1.0f );
    DeimosMaterialNode->SetMaterial( Material );
  }
```

现在，我们可以使用一推箱子和一个从`.obj`文件加载的 Deimos（[`en.wikipedia.org/wiki/Deimos_(moon)`](https://en.wikipedia.org/wiki/Deimos_(moon)）的 3D 模型来创建场景几何：

```java
  {
    auto VA = clGeomServ::CreateAxisAlignedBox( vec3(-0.5), vec3(+0.5) );
    g_Box= make_intrusive<clGeometryNode>();
    g_Box->SetVertexAttribs( VA );
    CubeMaterialNode->Add( g_Box );
```

这个函数可以在`Loader_OBJ.cpp`文件中找到，并解析 Wavefront OBJ 文件格式（[`en.wikipedia.org/wiki/Wavefront_.obj_file`](https://en.wikipedia.org/wiki/Wavefront_.obj_file)）：

```java
    auto DeimosNode = LoadOBJSceneNode( g_FS->CreateReader( "deimos.obj" ) );
    DeimosNode->SetLocalTransform( mat4::GetScaleMatrix(vec3(0.01f, 0.01f, 0.01f)) * mat4::GetTranslateMatrix(vec3(1.1f, 1.1f, 0.0f)) );
    DeimosMaterialNode->Add( DeimosNode );
  }
  {
    auto VA = clGeomServ::CreateAxisAlignedBox( vec3(-2.0f, -2.0f, -1.0f), vec3(2.0f, 2.0f, -0.95f) );
    auto Geometry = make_intrusive<clGeometryNode>();
    Geometry->SetVertexAttribs( VA );
    PlaneMaterialNode->Add( Geometry );
  }
```

最后但同样重要的是，我们将向场景中添加两个光源，这将产生两个不同的阴影：

```java
  {
    auto Light = make_intrusive<iLight>( );
    Light->m_Diffuse = vec4( 0.5f, 0.5f, 0.5f, 1.0f );
    Light->m_Position = vec3( 5, 5, 5 );
    Light->m_Direction = vec3( -1, -1, -1 ).GetNormalized();
    Light->m_SpotOuterAngle = 21;
    g_LightNode = make_intrusive<clLightNode>( );
    g_LightNode->SetLight( Light );
    g_Scene->Add( g_LightNode );
  }
  {
    auto Light = make_intrusive<iLight>();
    Light->m_Diffuse = vec4( 0.5f, 0.5f, 0.5f, 1.0f );
    Light->m_Position = vec3( 5, -5, 5 );
    Light->m_Direction = vec3( -1, 1, -1 ).GetNormalized();
    Light->m_SpotOuterAngle = 20;
    auto LightNode = make_intrusive<clLightNode>();
    LightNode->SetLight( Light );
    g_Scene->Add( LightNode );
  }
```

将所有内容组合在一起，并进入应用程序主循环：

```java
  g_Scene->Add( CubeMaterialNode );
  g_Scene->Add( PlaneMaterialNode );
  g_Scene->Add( DeimosMaterialNode );
  while( g_Window && g_Window->HandleInput() )
  {
    OnDrawFrame();
    g_Window->Swap();
  }
```

结果应用程序渲染了以下图像，包含旋转的立方体和两个光源的阴影：

![场景构建](img/image00227.jpeg)

演示应用程序也可以在 Android 上运行。去试试吧！

## 用户与 3D 场景的交互

我们希望您尝试运行了`2_ShadowMaps`示例。您可能已经注意到，3D 场景可以通过触摸屏上的手势或在桌面计算机上使用鼠标进行旋转。

这是使用`clVirtualTrackball`类完成的，它通过根据提供的触摸点计算视图矩阵来模拟虚拟轨迹球：

```java
class clVirtualTrackball
{
public:
  clVirtualTrackball():
  FCurrentPoint( 0.0f ),
  FPrevPoint( 0.0f ),
  FStarted( false )
  {
    FRotation.IdentityMatrix();
    FRotationDelta.IdentityMatrix();
  };
```

获取与新的触摸点相对应的视图矩阵：

```java
  virtual LMatrix4 DragTo( LVector2 ScreenPoint, float Speed, bool KeyPressed )
  {
    if ( KeyPressed && !FStarted )
    {
      StartDragging( ScreenPoint );
      FStarted = KeyPressed;
      return mat4::Identity();
    }
    FStarted = KeyPressed;
```

如果我们没有触摸屏幕，返回一个单位矩阵：

```java
    if ( !KeyPressed ) return mat4::Identity();
```

将触摸点投影到虚拟轨迹球上，并找到当前投影点与上一个投影点之间的距离：

```java
    FCurrentPoint = ProjectOnSphere( ScreenPoint );
    LVector3 Direction = FCurrentPoint - FPrevPoint;
    LMatrix4 RotMatrix;
    RotMatrix.IdentityMatrix();
    float Shift = Direction.Length();
```

如果距离不为零，计算并返回一个旋转矩阵：

```java
    if ( Shift > Math::EPSILON )
    {
      LVector3 Axis = FPrevPoint.Cross( FCurrentPoint );
      RotMatrix.RotateMatrixAxis( Shift * Speed, Axis );
    }
    FRotationDelta = RotMatrix;
    return RotMatrix;
  }
  LMatrix4& GetRotationDelta()
  {
    return FRotationDelta;
  };
```

获取当前矩阵：

```java
  virtual LMatrix4 GetRotationMatrix() const
  {
    return FRotation * FRotationDelta;
  }
  static clVirtualTrackball* Create()
  {
    return new clVirtualTrackball();
  }
```

当用户首次触摸屏幕时，重置轨迹球的状态：

```java
private:
  virtual void StartDragging( LVector2 ScreenPoint )
  {
    FRotation = FRotation * FRotationDelta;
    FCurrentPoint = ProjectOnSphere( ScreenPoint );
    FPrevPoint = FCurrentPoint;
    FRotationDelta.IdentityMatrix();
  }
```

投影数学计算如下：

```java
  LVector3 ProjectOnSphere( LVector2 ScreenPoint )
  {
    LVector3 Proj;
```

将标准化点坐标转换为`-1.0...1.0`范围：

```java
    Proj.x = 2.0f * ScreenPoint.x - 1.0f;
    Proj.y = -( 2.0f * ScreenPoint.y - 1.0f );
    Proj.z = 0.0f;
    float Length = Proj.Length();
    Length = ( Length < 1.0f ) ? Length : 1.0f;
    Proj.z = sqrtf( 1.001f - Length * Length );
    Proj.Normalize();
    return Proj;
  }
  LVector3 FCurrentPoint;
  LVector3 FPrevPoint;
  LMatrix4 FRotation;
  LMatrix4 FRotationDelta;
  bool FStarted;
};
```

该类在`UpdateTrackball()`函数中使用，该函数是从`OnDrawFrame()`中调用的：

```java
void UpdateTrackball( float Speed )
{
  g_Trackball.DragTo( g_MouseState.FPos, Speed, g_MouseState.FPressed );
}
void OnDrawFrame()
{
  UpdateTrackball( 10.0f );
  mat4 TrackballMtx = g_Trackball.GetRotationMatrix();
  Matrices.m_ViewMatrix = TrackballMtx * g_Camera.GetViewMatrix();
}
```

这个类允许你在触摸屏上旋转 3D 场景，并可用于在设备上调试场景。

# 总结

在本章中，我们学习了如何在我们平台独立的 OpenGL 封装之上构建更高级的场景图抽象。我们可以创建带有材质和光源的场景对象，并使用光照和阴影渲染场景。在下一章中，我们将暂时离开渲染——好吧，不是完全离开——并学习如何用 C++实现游戏逻辑。
