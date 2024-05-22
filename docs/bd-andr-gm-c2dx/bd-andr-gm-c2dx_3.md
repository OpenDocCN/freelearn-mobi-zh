# 第三章：理解游戏物理

在本章中，我们将介绍如何通过使用基于流行的 Chipmunk 框架的 Cocos2d-x 内置引擎，向游戏中添加物理效果。我们将解释以下主题：

+   设置物理世界

+   检测碰撞

+   处理重力

+   处理物理属性

有关 Chipmunk 物理引擎的更多信息，你可以访问[`chipmunk-physics.net`](https://chipmunk-physics.net)。

物理引擎封装了与给场景真实运动相关的所有复杂性，例如给物体添加重力使其被吸引到屏幕底部，或检测实体之间的碰撞等等。

在处理物理时，我们应该记住我们在场景中处理的是一个物理世界，所有参与世界的物理元素都被称为物理实体。这些实体具有质量、位置和旋转等属性。这些属性可以更改以自定义实体。一个物理实体可以通过关节定义附着在另一个实体上。

需要注意的是，从物理学的角度来看，物理实体并不知道物理世界外的精灵和其他对象，但我们将在这章中看到如何将精灵与物理实体连接起来。

视频游戏最常见的特征之一是碰撞检测；我们经常需要知道物体何时与其他物体发生碰撞。这可以通过定义代表每个实体碰撞区域的形状轻松完成，然后指定一个碰撞监听器，我们将在本章后面展示如何操作。

最后，我们将介绍 Box2D 物理引擎，这是一个完全独立的物理引擎，与 Chipmunk 无关。Box2D 是用 C++编写的，而 Chipmunk 是用 C 编写的。

# 设置物理世界

为了在游戏中启用物理，我们需要向我们的`HelloWorldScene.h`头文件中添加以下几行：

```java
cocos2d::Sprite* _sprBomb;
  void initPhysics();
  bool onCollision(cocos2d::PhysicsContact& contact);
  void setPhysicsBody(cocos2d::Sprite* sprite);
```

在这里，我们为`_sprBomb`变量创建了一个实例变量，这样它就可以被所有实例方法访问。在这种情况下，我们希望能够在每次检测到物理实体之间的碰撞时调用的`onCollision`方法中访问炸弹实例，这样我们只需将它的可见属性设置为 false，就可以让炸弹消失。

现在，让我们转到我们的`HelloWorld.cpp`实现文件，并进行一些更改以设置我们的物理世界。

首先，让我们修改我们的`createScene`方法，现在它看起来应如下所示：

```java
Scene* HelloWorld::createScene()
{
  auto scene = Scene::createWithPhysics();
  scene->getPhysicsWorld()->setGravity(Vect(0,0));
  auto layer = HelloWorld::create();
  //enable debug draw
  scene->getPhysicsWorld()->setDebugDrawMask(PhysicsWorld::DEBUGDRAW_ALL);
  scene->addChild(layer);
  return scene;
}
```

### 注意

在 Cocos2d-x 3 分支的早期版本中，你需要指定当前场景层将要使用的物理世界。但在 3.4 版本中，这不再必要，`Layer`类中移除了`setPhysicsWorld`方法。

在这里，我们可以看到我们现在是通过`Scene`类中的`createWithPhysics`静态方法创建场景实例，而不是使用简单的创建方法。

我们接下来要进行的第二步是将重力设置为（0,0），这样物理世界的重力就不会将我们的精灵吸引到屏幕底部。

然后，我们将启用物理引擎的调试绘制功能，这样我们就能看到所有的物理实体。这个选项将有助于我们在开发阶段，我们将使用 COCOS2D_DEBUG 宏，使其仅在调试模式下运行时显示调试绘制，如下所示：

```java
#if COCOS2D_DEBUG
  scene->getPhysicsWorld()->setDebugDrawMask(PhysicsWorld::DEBUGDRAW_ALL);
#endif
```

在以下屏幕截图中，我们可以看到围绕炸弹和玩家精灵的红色圆形。这表示附加到每个玩家精灵的物理实体：

![设置物理世界](img/B04193_03_01.jpg)

现在，让我们实现我们的`setPhysicsBody`方法，它接收一个指向我将添加物理实体的精灵对象的指针作为参数。此方法将创建一个表示物理实体和碰撞区域的圆形。该圆形的半径将是精灵宽度的一半，以尽可能覆盖精灵的面积。

```java
void HelloWorld::setPhysicsBody(cocos2d::Sprite* sprite){
  auto body = PhysicsBody::createCircle(sprite->getContentSize().width/2);
  body->setContactTestBitmask(true);
  body->setDynamic(true);
  sprite -> setPhysicsBody(body);
}
```

### 注意

圆形通常用于检测碰撞，因为它们在每个帧中检测碰撞所需的 CPU 努力较小；然而，在某些情况下，它们的精度可能无法接受。

现在，让我们在`init`方法中为我们的玩家和炸弹精灵添加物理实体。为此，我们将在每个精灵初始化后调用我们的实例方法 setPhysicsBody。

# 碰撞检测

首先，让我们实现我们的`onCollision`实例方法。每次检测到两个物理实体之间的碰撞时，都会调用它。正如在下面的代码中我们可以看到，当炸弹物理实体与我们的玩家碰撞时，它使炸弹变得不可见：

```java
bool HelloWorld::onCollision(PhysicsContact& contact){
  _sprBomb->setVisible(false);
  return false;
}
```

### 注意

在开发过程中，这里是一个放置一些日志的好地方，以了解何时检测到碰撞。在 Cocos2d-x 3.4 中，你可以使用`CCLOG`宏打印日志消息。通过以下方式定义宏`COCOS2D_DEBUG`可以开启它：`#define COCOS2D_DEBUG 1`。

如我们所见，这个方法返回一个布尔值。它表示这两个实体是否可以再次碰撞。在这个特定的情况下，我们将返回 false，表示一旦这两个物理实体碰撞，它们就不应该继续碰撞。如果我们返回 true，那么这两个对象将继续碰撞，这将导致我们的玩家精灵移动，从而给我们的游戏带来不希望出现的视觉效果。

现在，让我们使我们的游戏能够在炸弹与玩家碰撞时检测到。为此，我们将创建一个`EventListenerPhysicsContact`实例，我们将设置它，以便当两个物理体开始碰撞时，它应该调用我们的`onCollision`实例方法。然后，我们将事件监听器添加到事件分发器中。我们将在`initPhysics`实例方法中创建这三个简单步骤。所以，我们的代码将如下所示：

```java
void HelloWorld::initPhysics()
{
  auto contactListener = EventListenerPhysicsContact::create();
  contactListener->onContactBegin = CC_CALLBACK_1(HelloWorld::onCollision,this);
  getEventDispatcher() ->addEventListenerWithSceneGraphPriority(contactListener,this);
}
```

我们的`init`方法的代码将如下所示：

```java
bool HelloWorld::init() {
  if( !Layer::init() ){
    return false;
  }
  _director = Director::getInstance();
  _visibleSize = _director->getVisibleSize();
  auto origin = _director->getVisibleOrigin();
  auto closeItem = MenuItemImage::create("pause.png", "pause_pressed.png", CC_CALLBACK_1(HelloWorld::pauseCallback, this));
  closeItem->setPosition(Vec2(_visibleSize .width - closeItem->getContentSize().width/2, closeItem->getContentSize().height/2));

  auto menu = Menu::create(closeItem, nullptr);
  menu->setPosition(Vec2::ZERO);
  this->addChild(menu, 1);
  _sprBomb = Sprite::create("bomb.png");
  _sprBomb->setPosition(_visibleSize .width/2, _visibleSize .height + _sprBomb->getContentSize().height/2);
  this->addChild(_sprBomb,1);
  auto bg = Sprite::create("background.png");
  bg->setAnchorPoint(Vec2());
  bg->setPosition(0,0);
  this->addChild(bg, -1);
  auto sprPlayer = Sprite::create("player.png");
  sprPlayer->setPosition(_visibleSize .width / 2, _visibleSize .height * 0.23);
  setPhysicsBody(sprPlayer);
  this->addChild(sprPlayer, 0);
  //Animations
  Vector<SpriteFrame*> frames;
  Size playerSize = sprPlayer->getContentSize();
  frames.pushBack(SpriteFrame::create("player.png", Rect(0, 0, playerSize.width, playerSize.height)));
  frames.pushBack(SpriteFrame::create("player2.png", Rect(0, 0, playerSize.width, playerSize.height)));
  auto animation = Animation::createWithSpriteFrames(frames,0.2f);
  auto animate = Animate::create(animation);
  sprPlayer->runAction(RepeatForever::create(animate));
  setPhysicsBody(_sprBomb);
  initPhysics();
  return true;
}
```

# 处理重力

既然我们已经成功使用内置物理引擎来检测碰撞，那么让我们来玩弄一下重力。转到`createScene`方法，并修改我们发送到构造函数的参数。在我们的游戏中，我们使用了`(0,0)`值，因为我们不希望我们的世界有任何在*x*或*y*轴上移动我们物体的重力。

现在，尝试一下，将值改为正数或负数。当我们在*x*轴上使用负值时，它会将物体吸引向左，而在*y*轴上使用负值时，它会将物体吸引向下。

### 注意

改变这些值并理解添加到我们游戏中的物理可能会为你的下一个游戏提供一些想法。

## 处理物理属性

既然我们已经创建了对应于物理世界的场景，我们现在有能力改变物理属性，比如每个物体的速度、线性阻尼、力、冲量和扭矩。

### 应用速度

在上一章中，我们设法使用`MoveTo`动作将炸弹从屏幕顶部移动到底部。现在我们使用了内置的物理引擎，只需为炸弹设置速度就能实现同样的效果。这可以通过简单地调用炸弹精灵物理体的`setVelocity`方法来完成。速度是一个矢量量；因此，提到的方法接收一个`Vect`实例作为参数。*x*的值表示其水平分量；在这个轴上，正值意味着物体将向右移动，负值意味着物体将向左移动。*y*值影响垂直运动。正值将物体向屏幕顶部移动，负值将物体向屏幕底部移动。

我们在`HelloWorld.cpp`实现文件的`init`方法中，在返回语句之前添加了以下行：

```java
  _sprBomb->getPhysicsBody()->setVelocity(Vect(0,-100));
```

记得删除请求炸弹精灵执行`MoveTo`动作的代码行，以便确认炸弹现在是因为其速度参数而移动。

现在让我们转到`onCollision`方法，当炸弹与我们的玩家精灵发生碰撞时，我们将把炸弹的速度设置为零。

```java
  _sprBomb -> getPhysicsBody()->setVelocity(Vect());
```

### 注意

与`Vec2`类相似，空构造函数会将所有向量值初始化为零。

### 线性阻尼

我们可以降低物理体的速度，以产生摩擦效果。实现这一目标的方法之一是调用`linearDamping`方法，并指定身体速度的变化率。该值应该是一个介于`0.0`和`1.0`之间的浮点数。

你可以通过将炸弹物理体的值设置为`0.1f`来测试线性阻尼，并观察炸弹速度如何降低。

```java
  _sprBomb->getPhysicsBody()->setLinearDamping(0.1f);
```

测试线性阻尼后，记得记录或删除这行代码，以防止游戏出现预期之外的行为。

### 应用力

我们可以通过简单地调用想要施加力的物理体的`applyForce`方法，来立即对物体施加力。与前面章节中解释的方法类似，它接收一个向量作为参数，这意味着力有垂直和水平分量。

我们可以通过在`onCollision`方法中给炸弹施加一个力来测试这个方法，使它在与玩家精灵碰撞后立即向右移动。

```java
  _sprBomb->getPhysicsBody()->applyForce(Vect(1000,0));
```

### 应用冲量

在上一节中，我们给物理体添加了一个即时力，现在我们可以通过调用`applyImpulse`方法对其施加一个连续力。

在`onCollision`方法中对物理体施加即时力之后，添加以下代码行：

```java
  _sprBomb->getPhysicsBody()->applyImpulse(Vect(10000,0));
```

现在运行游戏，你将看到炸弹向右移动。

![应用冲量](img/B04193_03_02.jpg)

删除在`onCollision`方法中给炸弹添加力和冲量的代码行。

### 应用扭矩

最后，让我们在炸弹与玩家精灵碰撞后，使炸弹旋转。我们可以通过使用`applyTorque`方法给炸弹的物理体施加一个扭矩力，该方法接收一个浮点数；如果是正数，它将使物理体逆时针旋转。

让我们在`onCollision`方法中的返回语句之前，添加一个任意的正扭矩：

```java
  auto body = _sprBomb -> getPhysicsBody();
body->applyTorque(100000);
```

现在给`applyTorque`方法添加一个负值，你将看到物理体如何顺时针旋转。

# 把所有东西放在一起

经过所有修改后，我们的`onCollision`方法看起来像这样：

```java
bool HelloWorld::onCollision(PhysicsContact& contact){
  auto body = _sprBomb -> getPhysicsBody();
  body->setVelocity(Vect());
  body->applyTorque(100900.5f);
  return false;
}
```

我们现在的`init`方法看起来像这样：

```java
bool HelloWorld::init()
{
  if( !Layer::init() ){
    return false;
  }
  _director = Director::getInstance();
  _visibleSize = _director->getVisibleSize();
  auto origin = _director->getVisibleOrigin();
  auto closeItem = MenuItemImage::create("CloseNormal.png", "CloseSelected.png", CC_CALLBACK_1(HelloWorld::pauseCallback, this));
  closeItem->setPosition(Vec2(_visibleSize .width - closeItem->getContentSize().width/2, closeItem->getContentSize().height/2));

  auto menu = Menu::create(closeItem, nullptr);
  menu->setPosition(Vec2::ZERO);
  this->addChild(menu, 1);
  _sprBomb = Sprite::create("bomb.png");
  _sprBomb->setPosition(_visibleSize .width/2, _visibleSize .height + _sprBomb->getContentSize().height/2);
  this->addChild(_sprBomb,1);
  auto bg = Sprite::create("background.png");
  bg->setAnchorPoint(Vec2());
  bg->setPosition(0,0);
  this->addChild(bg, -1);
  auto sprPlayer = Sprite::create("player.png");
  sprPlayer->setPosition(_visibleSize .width/2, _visibleSize .height * 0.23);
  setPhysicsBody(sprPlayer);

  this->addChild(sprPlayer, 0);
  //Animations
  Vector<SpriteFrame*> frames;
  Size playerSize = sprPlayer->getContentSize();
  frames.pushBack(SpriteFrame::create("player.png", Rect(0, 0, playerSize.width, playerSize.height)));
  frames.pushBack(SpriteFrame::create("player2.png", Rect(0, 0, playerSize.width, playerSize.height)));
  auto animation = Animation::createWithSpriteFrames(frames,0.2f);
  auto animate = Animate::create(animation);
  sprPlayer->runAction(RepeatForever::create(animate));

  setPhysicsBody(_sprBomb);
  initPhysics();
  _sprBomb->getPhysicsBody()->setVelocity(Vect(0,-100));
  return true;
}
```

下面的截图展示了我们对游戏进行修改后的样子：

![把所有东西放在一起](img/B04193_03_03.jpg)

### 提示

**Box2D 物理引擎**

到目前为止，我们使用了框架提供的内置物理引擎，该引擎基于 chipmunk C 物理库；然而，Cocos2d-x 也在其 API 中集成了 Box2D 物理引擎。

为了创建一个 Box2D 世界，我们实例化`b2World`类，然后向其构造函数传递一个表示世界重力的`b2Vec`对象。世界实例有一个用于创建`b2Bodies`的实例方法。Sprite 类有一个名为`setB2Body`的方法，它允许我们将 Box2D 物理体关联到任何给定的精灵。这比框架的第二个分支中的处理要平滑得多；之前需要更多的代码才能将`b2Body`与精灵绑定。

尽管 Box2D 集成使用起来很方便，但我强烈建议使用内置的物理引擎，因为 Box2D 集成已不再积极开发中。

# 总结

我们通过创建物理世界和代表炸弹及玩家精灵的物理体，向游戏中添加了物理效果，并且在很少的步骤内使用了内置物理引擎提供的碰撞检测机制。我们还展示了如何更改重力参数，以便物理体根据重力力量移动。我们可以轻松地更改物体的物理属性，例如速度、摩擦力、力、冲量和扭矩，每个属性只需一行代码。到目前为止，我们的玩家忽略了用户事件。在下一章中，我们将介绍如何向游戏中添加用户交互。
