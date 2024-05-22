# 第九章. 实现游戏逻辑

对本章最简短的描述只有两个字：状态机。在这里，我们将介绍一种常见的处理游戏代码与用户界面部分交互的方法。我们从 Boids 算法的实现开始，然后继续扩展我们在前几章中实现的用户界面。

# 鸟群

在许多游戏应用中，你通常会看到移动的对象相互碰撞、射击、相互追逐、可以被其他对象触摸或避开，或者产生类似的行为。对象的可见复杂行为通常可以分解为几个简单状态之间的相互操作。例如，在街机游戏中，敌人会随机*四处漫游*，直到它看到由玩家控制的角色。遭遇后，它会切换到*追逐*状态，在接近目标时可能会切换到*射击*或*攻击*状态。如果敌方单位感知到某些劣势，它可能会*逃离*玩家。*追逐*状态反过来不仅仅是将敌人指向玩家，同时也会避免与环境发生碰撞。每个对象在不同的状态下可能会有不同的动画或材质。让我们使用由 Craig Reynolds 发明的已确立的方法来实现追逐和漫游算法，这种方法被称为*群体行为*或*鸟群*（[`en.wikipedia.org/wiki/Boids`](https://en.wikipedia.org/wiki/Boids)）。这种方法用于创建半意识群体或某些生物智能群体的印象。在本章中，我们广泛使用状态模式（[`en.wikipedia.org/wiki/State_pattern`](https://en.wikipedia.org/wiki/State_pattern)）来定义复杂用户交互场景。

我们仅考虑一个二维游戏世界，并将每个对象（或群体成员）近似为一个带有速度的圆形。速度作为一个矢量，既有大小也有方向。每个对象遵循三条简单的规则来计算其期望速度（[`www.red3d.com/cwr/boids`](http://www.red3d.com/cwr/boids)）：

1.  *避障*：为了避免拥挤局部群体成员而偏离。

1.  *凝聚*：朝向局部群体成员的平均朝向。

1.  *对齐*：朝向局部群体成员的平均位置移动。

我们实施的附加规则或行为包括*到达*和*漫游*算法，这些可以作为我们行为机制实现的基本调试工具。

第一条规则，*避障*，是指将速度从障碍物以及其他群体成员的方向偏离，如下图所示：

![鸟群](img/image00228.jpeg)

第二条规则，*凝聚*，是指向局部群体成员的平均朝向，如下图所示：

![鸟群](img/image00229.jpeg)

第三条规则，*Alignment*，尝试调整附近物体的计算平均速度。它以这种方式影响一群同伴，使它们很快移动方向成为共线和同向的，如下图所示：

![鸟类](img/image00230.jpeg)

*ArriveTo*规则将速度方向设置为空间中预定义的目标点或区域。通过允许目标在空间中移动，我们可以创建一些复杂的行为。

# 群集算法的实现

为了实现上述行为，我们将考虑以下类层次结构：

![群集算法的实现](img/image00231.jpeg)

单个鸟类由`clBoid`类的实例表示，该实例保存了一个指向`iBehaviour`实例的指针。`iBehavior`接口包含一个单一的成员函数`GetControl()`，它计算作用在鸟类上的即时力。由于力的大小可能取决于鸟类的状态，我们将原始的非拥有指针传递给`clBoid`到`GetControl()`中：

```java
class iBehaviour: public iIntrusiveCounter
{
public:
  virtual vec2 GetControl( float dt, clBoid* Boid ) = 0;
};
```

让我们考虑一下`clBoid`类本身。它包含`m_Pos`和`m_Vel`字段，分别保存了鸟类的当前位置和速度。这些值是二维向量，但整个结构可以使用三组件向量扩展到 3D 逻辑：

```java
class clBoid: public iActor
{
public:
  vec2 m_Pos;
  vec2 m_Vel;
```

`m_Angle`字段是鸟类的即时方向，它从`m_Vel`值计算而来。`m_MaxVel`字段包含鸟类的最大速度：

```java
  float m_Angle;
  float m_MaxVel;
```

`m_Behaviour`字段保存了一个指向`iBehaviour`实例的指针，该实例计算所需行为的控制力：

```java
  clPtr<iBehaviour> m_Behaviour;
```

由于鸟类在群体中移动，并且依赖于邻近鸟类的位置和速度来调整其速度，我们保留一个非拥有的指向父`clSwarm`对象的指针，以避免智能指针之间的循环引用：

```java
  clSwarm* m_Swarm;
```

类的构造函数初始化默认值，并设置一个空的行为：

```java
public:
  clBoid():
  m_Pos(), m_Vel(),
  m_Angle(0.0f), m_MaxVel(1.0f),
  m_Behaviour(), m_Swarm(nullptr)
  {}
```

唯一的成员函数是`Update()`，它计算作用在物体上的力：

```java
  virtual void Update( float dt ) override
  {
    if ( m_Behaviour )
    {
      vec2 Force = m_Behaviour->GetControl( dt, this );
```

计算完力后，根据牛顿定律，*a = F/m*，以及欧拉积分方法（[`en.wikipedia.org/wiki/Euler_method`](https://en.wikipedia.org/wiki/Euler_method)）修改速度。鸟类的质量设置为常数`1.0`。可以自由引入可变参数，观察它如何改变群体的视觉行为：

```java
      const float Mass = 1.0f;
      vec2 Accel = Force / Mass;
      m_Vel += Accel * dt;
    }
```

为了使视觉效果可信，我们将可能的最大速度限制在`0…m_MaxVel`区间内：

```java
    m_Vel = ClampVec2( m_Vel, m_MaxVel );
```

计算完速度后，更新鸟类的位置：

```java
    m_Pos += m_Vel * dt;
```

最后，鸟类的方向应该被评估为`X`轴与`m_Vel`向量之间的角度。这个值用于使用指向箭头在屏幕上渲染鸟类：

```java
    if ( m_Vel.SqrLength() > 0.0f )
    {
      m_Angle = atan2( m_Vel.y, m_Vel.x );
    }
  }
};
```

最简单的非静态行为是在突发随机脉冲影响下的随机移动。这称为*漫游*行为，在`clWanderBehaviour`类中实现。`GetControl()`方法计算一个在-1..+1 范围内具有两个随机分量的向量：

```java
class clWanderBehaviour: public iBehaviour
{
public:
  virtual vec2 GetControl( float dt, clBoid* Boid ) override
  {
    return vec2( RandomFloat() * 2.0f - 1.0f, RandomFloat() * 2.0f - 1.0f );
  }
};
```

`clGoToBehaviour`类实现了鸟群另一种有用的行为。给定`m_Target`字段中的目标坐标，这种行为驱动可控鸟群到那个点。一旦鸟位于距离`m_Target`的`m_TargetRadius`范围内，移动就会停止：

```java
class clGoToBehaviour: public iBehaviour
{
public:
```

`m_Target`和`m_TargetRadius`字段定义了目标点的位置和半径：

```java
  vec2 m_Target;
  float m_TargetRadius;
```

`m_VelGain`和`m_PosGain`成员保存两个值，定义了当达到目标时鸟应该多快制动，以及鸟群与目标距离成比例加速的快慢：

```java
  float m_VelGain;
  float m_PosGain;
```

构造函数设置默认值和非零增益：

```java
  clGoToBehaviour():
  m_Target(),
  m_TargetRadius(0.1f),
  m_VelGain(0.05f),
  m_PosGain(1.0f)
  {}
```

`GetControl()`例程计算了鸟的位置与目标之间的差值。这个差值乘以`m_PosGain`，并用作控制力：

```java
  virtual vec2 GetControl( float dt, clBoid* Boid ) override
  {
    auto Delta = m_Target - Boid->m_Pos;
```

如果鸟群中的鸟位于`m_TargetRadius`距离内，我们将返回零作为控制力的值：

```java
    if ( Delta.Length() < m_TargetRadius )
    {
      return vec2();
    }
```

### 注意

通过将前面的行替换为`return -m_VelGain * Boid->m_Vel / dt;`这行，可以实现一个视觉上有趣的制动效果。施加制动脉冲，通过减少一些分数来降低速度，从而实现速度的平滑指数衰减。在视觉上，鸟群在目标中心附近平滑地停止。

计算出的脉冲在成员函数的末尾返回：

```java
    return Delta * m_PosGain;
  }
};
```

# 偏题：辅助例程

在这里，我们应该描述在控制计算中使用的几个函数。

在前面的代码中，我们使用了`ClampVec2()`例程，该例程计算向量`V`的长度，将这个长度与`MaxValue`进行比较，并返回相同的向量`V`或其长度为`MaxValue`的约束同轴版本：

```java
inline vec2 ClampVec2(const vec2& V, float MaxValue)
{
  float L = V.Length();
  return (L > MaxValue) ? V.GetNormalized() * MaxValue : V;
}
```

另一组方法包括随机数生成例程。`RandomFloat()`方法使用 C++11 标准库在 0…1 区间内生成均匀分布的浮点值：

```java
std::random_device rd;
std::mt19937 gen( rd() );
std::uniform_real_distribution<> dis( 0.0, 1.0 );
float RandomFloat()
{
  return static_cast<float>( dis( gen ) );
}
```

`RandomVec2Range()`方法使用`RandomFloat()`函数两次，以返回在指定区间内具有随机分量的向量：

```java
vec2 RandomVec2Range( const vec2& Min, const vec2& Max )
{
  return Min + vec2( RandomFloat() * ( Max - Min ).x,
    RandomFloat() * ( Max - Min ).y );
}
```

# 群体行为

到目前为止，我们只定义了`clWanderBehaviour`类。为了实现群集算法，我们需要一次性存储所有鸟的信息。这样的集合在这里称为*群体*。`clSwarm`类保存一个`clBoid`对象的向量，并实现了一些在鸟群控制计算中使用的例程：

```java
class clSwarm: public iIntrusiveCounter
{
public:
  std::vector< clPtr<clBoid> > m_Boids;
  clSwarm() {}
```

为了调试和视觉演示目的，`GenerateRandom()`方法分配了具有随机位置和零速度的`clBoid`对象的数量：

```java
  void GenerateRandom( size_t N )
  {
    m_Boids.reserve( N );
    for ( size_t i = 0; i != N; i++ )
    {
      m_Boids.emplace_back( make_intrusive<clBoid>() );
```

默认情况下，每只鸟都具有一种*漫游*行为：

```java
      m_Boids.back()->m_Behaviour = make_intrusive<clWanderBehaviour>();
      m_Boids.back()->m_Swarm = this;
```

位置是随机的，并且每个坐标也在-1..+1 范围内保持：

```java
      m_Boids.back()->m_Pos = RandomVec2Range( vec2(-1, -1), vec2(1, 1) );
    }
  }
```

`Update()`方法遍历集合并更新每个鸟类：

```java
  void Update( float dt )
  {
    for ( auto& i : m_Boids )
    {
      i->Update( dt );
    }
  }
```

*分离*或*避障*算法使用与其他鸟类的距离总和作为控制力。`clSwarm::CalculateSeparation()`方法遍历鸟类集合并计算所需的和：

```java
  vec2 CalculateSeparation( clBoid* B, float SafeDistance )
  {
    vec2 Control;
    for ( auto& i : m_Boids)
    {
      if ( i.GetInternalPtr() != B )
      {
```

对于每个鸟类，除了作为参数传递的那个，我们计算位置差分：

```java
        auto Delta = i->m_Pos - B->m_Pos;
```

如果距离超过安全阈值，例如，如果鸟类与其他鸟类接近，我们向控制力中添加负的差分：

```java
        if ( Delta.Length() < SafeDistance )
        {
          Control += Delta;
        }
      }
    }
    return Control;
  }
```

*凝聚力*算法中使用了类似的例程来计算邻近鸟类的平均位置：

```java
  vec2 CalculateAverageNeighboursPosition( clBoid* B )
  {
    int N = static_cast<int>( m_Boids.size() );
```

如果不止一个鸟类，我们才累加位置：

```java
    if ( N > 1 )
    {
      vec2 Avg(0, 0);
```

对鸟类列表进行遍历，我们可以得到位置的总和：

```java
      for ( auto& i : m_Boids )
      {
        if ( i.GetInternalPtr() != B )
        {
          Avg += i->m_Pos;
        }
      }
      Avg *= 1.0f / (float)(N - 1);
      return Avg;
    }
```

在单个鸟类的情况下，我们使用它的位置。这样，*凝聚力*算法中的控制力将为零：

```java
    return B->m_Pos;
  }
```

类似的过程也应用于速度：

```java
  vec2 CalculateAverageNeighboursVelocity( clBoid* B )
  {
    int N = (int)m_Boids.size();
    if (N > 1)
    {
      vec2 Avg(0, 0);
      for ( auto& i : m_Boids )
        if ( i.GetInternalPtr() != B )
          Avg += i->m_Vel;
          Avg *= 1.0f / (float)(N - 1);
      return Avg;
    }
    return B->m_Vel;
  }
```

实用方法`SetSingleBehaviour()`将群体中每个鸟类的行为设置为指定值：

```java
  void SetSingleBehaviour( const clPtr<iBehaviour>& B )
  {
    for ( auto& i : m_Boids )
    {
      i->m_Behaviour = B;
    }
  }
};
```

现在我们有了`clSwarm`类，我们终于可以实施群聚行为了。`clFlockingBehaviour`使用邻近鸟类的信息，并使用经典的 Boids 算法计算控制力：

```java
class clFlockingBehaviour : public iBehaviour
{
```

与往常一样，构造函数设置默认参数：

```java
public:
  clFlockingBehaviour():
  m_AlignmentGain(0.1f),
  m_AvoidanceGain(2.0f),
  m_CohesionGain(0.1f),
  m_SafeDistance(0.5f),
  m_MaxValue(1.0f)
  {}
```

`m_SafeDistance`字段定义了一个距离，在该距离上避障算法不作用：

```java
  float m_SafeDistance;
```

下一个字段包含每个群聚算法影响的权重：

```java
  float m_AvoidanceGain;
  float m_AlignmentGain;
  float m_CohesionGain;
  virtual vec2 GetControl(float dt, clBoid* Boid) override
  {
    auto Swarm = Boid->m_Swarm;
```

第一步是*分离*和*避障*：

```java
    vec2 Sep = m_AvoidanceGain * Swarm->CalculateSeparation(Boid, m_SafeDistance);
```

第二步是*对齐*：

```java
    auto AvgPos = Swarm->CalculateAverageNeighboursPosition(Boid);
    vec2 Alignment = m_AlignmentGain * (AvgPos - Boid->m_Pos);
```

第三步是*凝聚力*。转向邻居的平均位置：

```java
    auto AvgVel = Swarm->CalculateAverageNeighboursVelocity(Boid);
    vec2 Cohesion = m_CohesionGain * (AvgVel - Boid->m_Vel);
```

最后，我们将这三个值相加，并保持力的大小在`m_MaxValue`以下：

```java
    return ClampVec2( Sep + Alignment + Cohesion, m_MaxValue );
  }
};
```

我们行为系统的最后润色是一个实现行为混合的类。`clMixedBehaviour`类包含行为和行为相应权重因子的向量，这些权重因子表示在结果行为中使用了多少行为的控制力：

```java
class clMixedBehaviour : public iBehaviour
{
public:
  std::vector< clPtr<iBehaviour> > m_Behaviours;
  std::vector<float> m_Weights;
```

`AddBehaviour()`成员函数向容器中添加一个新的权重因子和行为：

```java
  void AddBehaviour( float Weight, const clPtr<iBehaviour>& B )
  {
    m_Weights.push_back( Weight );
    m_Behaviours.push_back( B );
  }
```

如类名所示，`GetControl()`例程为包含的每个行为计算控制力，并将所有这些控制向量乘以适当的权重相加：

```java
  virtual vec2 GetControl(float dt, clBoid* Boid) override
  {
    vec2 Control;
    for ( size_t i = 0; i < m_Behaviours.size(); i++)
    {
      Control += m_Weights[i] * m_Behaviours[i]->GetControl(dt, Boid);
    }
    return Control;
  }
};
```

如我们所见，`clFlockingBehaviour`类可以分解为*避障*、*凝聚力*和*分离*部分。我们决定不使书籍结构复杂化，并将群聚行为实现为单个类。请随意实验和混合这些子行为。

# 渲染群体模拟。

为了使用开发好的群体模拟系统，我们需要渲染单个鸟类。由于我们已经有了 OpenGL 3D 场景图渲染系统，我们用三角形网格表示每个鸟类，并为它们创建场景节点。让我们这样做：

```java
class clSwarmRenderer
{
private:
```

`m_Root`字段中的单个`clSceneNode`对象作为整个群体的根场景节点：

```java
  clPtr<clSceneNode> m_Root;
```

保留指向 `clSwarm` 对象的指针，以将鸟群位置和角度与场景节点变换同步：

```java
  clPtr<clSwarm> m_Swarm;
```

每个鸟群个体的场景节点存储在 `m_Boids` 向量中：

```java
  std::vector< clPtr<clSceneNode> > m_Boids;
```

类的构造函数为群体中的每个鸟群个体创建了一个场景节点：

```java
public:
  explicit clSwarmRenderer( const clPtr<clSwarm> Swarm )
  : m_Root( make_intrusive<clSceneNode>() )
  , m_Swarm( Swarm )
  {
    m_Boids.reserve( Swarm->m_Boids.size() );
    const float Size = 0.05f;
    for ( const auto& i : Swarm->m_Boids )
    {
      m_Boids.emplace_back( make_intrusive<clSceneNode>() );
```

从视觉上看，鸟群个体是一个三角形，因此我们调用 `clGeomServ::CreateTriangle()` 来创建一个包含单个三角形的顶点数组：

```java
      auto VA = clGeomServ::CreateTriangle( -0.5f * Size, Size, Size, 0.0f );
      auto GeometryNode = make_intrusive<clGeometryNode>( );
      GeometryNode->SetVertexAttribs( VA );
      m_Boids.back()->Add( GeometryNode );
```

一旦几何节点初始化，我们将其添加到 `m_Root`：

```java
      m_Root->Add( m_Boids.back() );
    }
    Update();
  }
```

在每一帧中，我们计算与鸟群根节点关联的每个 `clSceneNode` 的变换。变换包括鸟群位置的平移，然后绕垂直 `Z` 轴旋转：

```java
  void Update()
  {
    for ( size_t i = 0; i != m_Boids.size(); i++ )
    {
      float Angle = m_Swarm->m_Boids[i]->m_Angle;
      mat4 T = mat4::GetTranslateMatrix( vec3( m_Swarm->m_Boids[i]->m_Pos ) );
      mat4 R = mat4::GetRotateMatrixAxis( Angle,vec3( 0, 0, 1 ) );
      m_Boids[i]->SetLocalTransform( R * T );
    }
  }
  clPtr<clSceneNode> GetRootNode() const { return m_Root; }
};
```

所有其他场景管理代码与之前章节的类似：

# 鸟群演示

`1_Boids` 中的演示代码混合使用了 *GoTo* 和 *Flocking* 行为，使一群鸟群追逐用户指定的目标，同时创建类似群体移动的错觉。

在这里，我们不讨论应用程序的整个源代码，只强调最重要的部分。演示的初始化从创建填充有随机位置鸟群的 `clSwarm` 开始：

```java
auto Swarm = make_intrusive<clSwarm>();
Swarm->GenerateRandom( 10 );
```

我们为所有鸟群个体设置相同的控制器。控制器本身是 `g_Behaviour` 对象中的 `clFlockingBehaviour` 和 `clGoToBehavior` 的混合体：

```java
auto MixedControl = make_intrusive<clMixedBehaviour>();
MixedControl->AddBehaviour(0.5f, make_intrusive<clFlockingBehaviour>());
MixedControl->AddBehaviour(0.5f, g_Behaviour);
Swarm->SetSingleBehaviour(MixedControl);
```

`g_Behaviour` 实例保存目标的坐标，最初设置为 `(1.0, 1.0)`：

```java
g_Behaviour->m_TargetRadius = 0.1f;
g_Behaviour->m_Target = vec2( 1.0f );
g_Behaviour->m_PosGain = 0.1f;
```

在渲染循环的每一帧中，都使用局部的 `clSwarmRenderer` 对象：

```java
clSwarmRenderer SwarmRenderer( Swarm );
```

该演示使用触摸输入来改变目标的坐标。当发生触摸时，我们将穿过触摸点的线与鸟群所在的平面相交。这个交点被用作新的目标点：

```java
void OnTouch( int X, int Y, bool Touch )
{
  g_MouseState.FPos = g_Window->GetNormalizedPoint( X, Y );
  g_MouseState.FPressed = Touch;
  if ( !Touch )
  {
```

一旦我们知道触摸已经结束，我们使用透视和视图矩阵将 2D 鼠标坐标反投到世界空间中：

```java
    vec3 Pos = Math::UnProjectPoint( vec3( g_MouseState.FPos ), Math::Perspective( 45.0f, g_Window->GetAspect(), 0.4f, 2000.0f ), g_Camera.GetViewMatrix() );
```

使用摄像机视图矩阵，我们计算旋转和平移，并使用这些值将鼠标位置的射线与 `Z=0` 平面相交：

```java
    mat4 CamRotation;
    vec3 CamPosition;
    DecomposeCameraTransformation( g_Camera.GetViewMatrix(), CamPosition, CamRotation );
    vec3 isect;
    bool R = IntersectRayToPlane( CamPosition, Pos - CamPosition, vec3( 0, 0, 1 ), 0, isect );
```

一旦构建了一个 3D 交点，它就可以用作 *GoTo* 行为的 2D 目标：

```java
    g_Behaviour->m_Target = isect.ToVector2();
  }
}
```

在每次迭代中，我们调用 `Swarm::Update()` 和 `clSwarmRenderer::Update()` 方法来更新单个鸟群个体的位置和速度，并将场景节点变换与新的数据同步。

现在，去运行 `1_Boids` 示例，亲自看看效果。

# 基于页面的用户界面

前几章的大部分内容已经为可移植的 C++ 应用程序奠定了基础。现在是时候向您展示如何将更多部分连接在一起了。第七章中，我们讨论了如何在 C++ 中创建一个简单的自定义用户界面以及如何响应用户输入。在这两种情况下，我们只实现了一个固定的行为，而没有解释如何在不编写意大利面条代码的情况下切换到另一个行为。本章的第一段介绍了 *行为* 的概念，我们现在将其应用于我们的用户界面。

我们将用户界面的单个全屏状态称为 *页面*。因此，应用程序的每个不同屏幕都由 `clGUIPage` 类表示，我们在此之后对其进行注释。

`clUIPage` 的三个主要方法是 `Render()`、`Update()` 和 `OnTouch()`。`Render()` 方法渲染包含所有子视图的完整页面。`Update()` 方法将视图与应用程序状态同步。`OnTouch()` 方法响应用户输入。`clGUIPage` 类是从 `clUIView` 派生的，因此理解这个类应该没有问题。

类包含两个字段。`FFallbackPage` 字段持有指向另一个页面的指针，该页面用作返回页面，例如，在 Android 上按下后退键时：

```java
class clGUIPage: public clUIView
{
public:
```

按下后退键时返回的页面：

```java
  clPtr<clGUIPage> FFallbackPage;
```

这个页面上的 GUI 对象的非拥有指针来自：

```java
  clGUI* FGUI;
public:
  clGUIPage(): FFallbackPage( nullptr ) {}
  virtual ~clGUIPage() {}
  virtual void Update( float DeltaTime ) {}
  virtual bool OnTouch( int x, int y, bool Pressed );
  virtual void Update( double Delta );
  virtual void SetActive();
```

当 GUI 管理器切换页面时，会调用 `OnActivation()` 和 `OnDeactivation()` 方法：

```java
  virtual void OnActivation() {}
  virtual void OnDeactivation() {}
public:
  virtual bool OnKey( int Key, bool KeyState );
};
```

页面列表存储在 `clGUI` 类中。`FActivePage` 字段指示当前可见的页面。用户输入的事件会被重定向到激活页面：

```java
class clGUI: public iObject
{
public:
  clGUI(): FActivePage( NULL ), FPages() {}
  virtual ~clGUI() {}
```

`AddPage()` 方法设置指向父 GUI 对象的指针，并将此页面添加到页面容器中：

```java
  void AddPage( const clPtr<clGUIPage>& P )
  {
    P->FGUI = this;
    FPages.push_back( P );
  }
```

`SetActivePage()` 方法除了实际设置页面为激活状态外，还会调用一些回调函数。如果新页面与当前激活的页面相同，则不会执行任何操作：

```java
  void SetActivePage( const clPtr<clGUIPage>& Page )
  {
    if ( Page == FActivePage ) { return; }
```

如果我们之前有一个激活的页面，我们会通知该页面切换到另一个页面：

```java
    if ( FActivePage )
    {
      FActivePage->OnDeactivation();
    }
```

如果新页面不是空页面，那么它会被通知已被激活：

```java
    if ( Page )
    {
      Page->OnActivation();
    }
    FActivePage = Page;
  }
```

正如我们之前提到的，每个事件都会被重定向到存储在 `FActivePage` 中的激活页面：

```java
  void Update( float DeltaTime )
  {
    if ( FActivePage )
    {
      FActivePage->Update( DeltaTime );
    }
  }
  void Render()
  {
    if ( FActivePage )
    {
      FActivePage->Render();
    }
  }
  void OnKey( vec2 MousePos, int Key, bool KeyState )
  {
    FMousePosition = MousePos;
    if ( FActivePage )
    {
      FActivePage->OnKey( Key, KeyState );
    }
  }
  void OnTouch( const LVector2& Pos, bool TouchState )
  {
    if ( FActivePage )
    {
      FActivePage->OnTouch( Pos, TouchState );
    }
  }
private:
  vec2 FMousePosition;
  clPtr<clGUIPage> FActivePage;
  std::vector< clPtr<clGUIPage> > FPages;
};
```

`OnKey()` 方法的实现只用于 Windows 或 OSX。然而，如果我们把后退键视为 Esc 键的类似物，同样的逻辑也可以应用于 Android：

```java
bool clGUIPage::OnKey( int Key, bool KeyState )
{
  if ( !KeyState && Key == LK_ESCAPE )
  {
```

如果我们有一个非空的备用页面，我们会将其设置为激活状态：

```java
    if ( FFallbackPage )
    {
      FGUI->SetActivePage( FFallbackPage );
      return true;
    }
  }
  return false;
}
```

`SetActive()` 的实现被放在类声明之外，因为它使用了当时未声明的 `clGUI` 类。这是为了从头文件中移除依赖：

```java
void clGUIPage::SetActive()
{
  FGUI->SetActivePage( this );
}
```

现在，我们的小型 GUI 页面机制已经完整，可以用来在实际应用程序中处理用户界面逻辑。

# 总结

在本章中，我们学习了如何实现对象的不同行为以及使用状态机和设计模式来实现群体逻辑。让我们继续最后一章，这样我们就可以将许多之前的示例整合到一个更大的应用程序中。
