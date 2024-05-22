# 第四章 用户输入

到目前为止，我们已经添加了在屏幕上移动并相互碰撞的图形，但这还不够有趣，因为玩家无法控制我们的主角，除非用户能够与之互动，否则它就不能算是一个游戏。在本章中，我们将向游戏中添加用户交互。本章将涵盖以下主题：

+   理解事件分发机制

+   处理触摸事件

+   处理加速计事件

+   保持屏幕活跃

+   处理 Android 返回键按下事件

# 理解事件分发机制

从 Cocos2d-x 的先前版本（版本 2）开始，事件处理现在有所不同。从 3.0 版本开始，我们现在有一个统一的事件分发机制，称为事件分发器，它处理游戏中可能发生的各种用户输入事件。

我们可以处理多种类型的用户输入事件，例如触摸、键盘按键按下、加速度和鼠标移动。在以下各节中，我们将介绍如何处理与移动游戏相关的用户输入事件，例如触摸和加速计。

有许多类允许我们监听之前提到的事件；一旦我们实例化了这些类中的任何一个，我们需要将其添加到事件分发器中，以便在触发用户事件时，它会调用相应监听器定义的方法。

你可以通过从`Node`类继承的`_eventDispatcher`实例属性访问事件分发器，或者调用位于 Cocos2d-x API `Director`类中的`getEventDispatcher`静态方法。

### 注意

Cocos2d-x 的事件分发机制使用了观察者设计模式，这是用于处理 Android 原生应用程序上用户输入事件的模式。

# 处理触摸事件

在游戏和用户之间创建交互的最常见方式是通过触摸事件。在 Cocos2d-x 中处理触摸事件非常直接。

在本节中，我们将允许用户通过触摸并移动到所需位置来移动我们的玩家精灵。

我们首先要做的是在`HelloWorldScene.h`类头文件中创建`initTouch`、`movePlayerByTouch`和`movePlayerIfPossible`方法，正如我们以下代码清单中看到的那样：

```java
void initTouch();
void movePlayerByTouch(cocos2d::Touch* touch, cocos2d::Event* event);
void movePlayerIfPossible(float newX);
```

现在让我们将初始化代码添加到实现文件`HelloWorldScene.cpp`中的`initTouch`方法中。在这个简单的游戏中，我们将使用单一触摸来四处移动我们的兔子角色，不需要处理多点触控。

为了处理单一触摸，我们将创建`EventListenerTouchOneByOne`类的新实例，然后我们将指定当触摸事件开始、移动和结束时游戏应该做什么。在以下代码清单中，实例化`EventListenerTouchOneByOne`类之后，我们将指定当触发`onTouchBegan`、`onTouchMoved`和`onTouchEnded`事件时应调用的方法。出于当前游戏的目的，我们只使用`onTouchMoved`事件。为此，我们将创建一个回调到我们的方法`movePlayerByTouch`，对于另外两个方法，我们将通过 lambda 函数创建空的结构。你可以通过链接[`en.cppreference.com/w/cpp/language/lambda`](http://en.cppreference.com/w/cpp/language/lambda)了解更多关于 C++ lambda 函数的信息。

```java
void HelloWorld::initTouch() {
  auto listener = EventListenerTouchOneByOne::create();
  listener->onTouchBegan = [](Touch* touch, Event* event){return true;
  }
  listener->onTouchMoved = CC_CALLBACK_2(HelloWorld::movePlayerByTouch,this);
  listener->onTouchEnded = ={};
  _eventDispatcher->addEventListenerWithSceneGraphPriority(listener, this);
}
```

### 注意

按照约定，所有的 C++成员变量都使用下划线前缀命名。

既然我们已经将所有触摸监听器初始化代码封装到一个方法中，让我们在方法的末尾添加以下行来调用它，我们称之为`init`方法：

```java
  initTouch();
```

我们现在将创建`movePlayerIfPossible`方法。这个方法只会在水平轴上的新请求位置没有超出屏幕限制时移动玩家精灵，正如我们可以在插图中看到的那样。这个方法将用于通过触摸输入事件移动我们的玩家精灵，并且在下一节中我们也将使用它通过加速度计移动我们的玩家精灵。

```java
void HelloWorld::movePlayerIfPossible(float newX){
  float sprHalfWidth = _sprPlayer->getBoundingBox().size.width/2;
  if(newX >= sprHalfWidth && newX < visibleSize.width - sprHalfWidth){
    _sprPlayer->setPositionX(newX);
  }
}
```

![处理触摸事件](img/B04193_04_01.jpg)

### 注意

在这个方法中，我们采用了“告诉，不要询问”的设计原则，通过在验证玩家是否超出屏幕的方法中进行验证。这使我们避免了在触摸和加速度事件处理方法中重复验证玩家精灵是否超出屏幕的逻辑。

最后，我们现在将创建`movePlayerByTouch`方法，该方法将在触发触摸事件后立即由事件调度器调用。在这个方法中，我们将评估屏幕上的位置，以及用户触摸屏幕的地方是否与精灵的边界矩形相交：

```java
void HelloWorld::movePlayerByTouch(Touch* touch, Event* event){
  auto touchLocation = touch->getLocation();
  if(_sprPlayer->getBoundingBox().containsPoint(touchLocation)){
    movePlayerIfPossible(touchLocation.x);
  }
}
```

## 处理多点触控事件

在前面的部分中，我们启用了这个游戏所需的触摸事件，这是一个单一触摸；然而，Cocos2d-x 也处理多点触控功能，我们将在本节中介绍。

尽管我们的游戏不需要多点触控功能，但我们将创建一个测试代码，以便我们可以同时移动我们的玩家精灵和炸弹。为了做到这一点，我们将在`HelloWorldScene.h`头文件的末尾添加`initMultiTouch`和`moveByMultitouch`方法，如下所示：

```java
void initMultiTouch();
void moveByMultiTouch(const std::vector<cocos2d::Touch*>& touches, cocos2d::Event* event);
```

现在，让我们将实现添加到`HelloWorldScene.cpp`实现文件中。我们将从`initMultiTouch`初始化方法开始：

```java
void HelloWorld::initMultiTouch() {
  auto listener = EventListenerTouchAllAtOnce::create();
  listener->onTouchesBegan = [](const std::vector<Touch*>& touches, Event* event){};
  listener->onTouchesMoved = CC_CALLBACK_2(HelloWorld::moveByMultiTouch,this);
  listener->onTouchesEnded = [](const std::vector<Touch*>& touches, Event* event){};
  _eventDispatcher->addEventListenerWithSceneGraphPriority(listener, this);
}
```

在这里，我们可以找到与之前单点触控初始化方法的相似之处，但有许多不同之处，最显著的是我们现在实例化的是`EventListenerTouchAllAtOnce`类，而不是像之前那样实例化`EventListenerTouchOneByOne`类。尽管它的事件属性与单点触控版本命名相似，但您可能注意到它们现在是用复数形式书写的，因此现在指的是 touches 而不是 touch，例如`onTouchesBegan`。现在，它也将期待一组不同的参数，因为我们将处理多点触控，事件方法将接收一个`std::vector`参数，其中包含同时发生的触控集合。

如之前的代码所示，每当玩家移动触控时，我们将调用我们的`moveByMultiTouch`方法，因此我们现在展示此方法的实现代码：

```java
void HelloWorld::moveByMultiTouch(const std::vector<Touch*>& touches, Event* event){
  for(Touch* touch: touches){
    Vec2 touchLocation = touch->getLocation();
    if(_sprPlayer->getBoundingBox().containsPoint(touchLocation)){
      movePlayerIfPossible(touchLocation.x);
    }else if(_sprBomb->getBoundingBox().containsPoint(touchLocation)){
      _sprBomb->setPosition(touchLocation);
    }
  }
}
```

如您在前面的代码中所见，我们现在正在处理多点触控，在`moveByMultiTouch`方法中，我们正在遍历所有的触控，并对每一个触控进行验证，看它是否触摸到我们的炸弹或兔子玩家精灵，如果是，那么它将把被触摸的精灵移动到触摸位置。

最后，让我们在`init`方法的末尾调用`initMultiTouch`初始化方法，如下所示：

```java
initMultiTouch();
```

如前所述，本节的目的是向您展示处理多点触控事件是多么简单；然而，由于我们的游戏中不会使用它，一旦您完成多点触控功能的测试，就可以从我们的`init`方法中删除对`initMultiTouch`方法的调用。

# 处理加速度计事件

游戏与玩家之间互动的另一种常见方式是加速度计，它允许我们通过移动手机来移动我们的角色，以达到游戏的目标，从而获得数小时的乐趣。

为了将加速度计支持添加到我们的游戏中，我们首先将在`HelloWorldScene.h`头文件中添加以下方法声明：

```java
void movePlayerByAccelerometer(cocos2d::Acceleration* acceleration, cocos2d::Event* event);
void initAccelerometer();
```

现在，让我们为对应的加速度计初始化创建`HelloWorld.cpp`实现文件中的代码。我们首先要做的是通过调用`Device`类上的静态方法`setAccelerometerEnabled`来启用设备上的加速度计传感器，然后我们将创建一个事件监听器来监听加速度计的事件，最后，我们将它添加到事件分发器中，如下面的代码所示：

```java
void HelloWorld::initAccelerometer(){
  Device::setAccelerometerEnabled(true);
  auto listener = EventListenerAcceleration::create(CC_CALLBACK_2(HelloWorld::movePlayerByAccelerometer, this));
  _eventDispatcher->addEventListenerWithSceneGraphPriority(listener, this);
}
```

### 注意

向分发器添加事件监听器的最常见方式是通过`addEventListenerWithSceneGraphPriority`方法，该方法将把作为第二个参数传递的节点的*z*顺序作为其优先级。当我们有多个同时被触发的监听器，而想要指定哪个代码应该首先运行时，这非常有用。

在此阶段，我们已经初始化了加速度计，并且在上一节中创建了`movePlayerIfPossible`方法，该方法将移动玩家精灵，并确保其不会超出屏幕限制。现在我们将要为`movePlayerByAccelerometer`方法编写实现代码，该方法将在加速度计事件触发后立即被调用。由于我们获得的加速度值非常低，因此我们将其乘以十，以便我们的玩家精灵移动得更快。

```java
void HelloWorld::movePlayerByAccelerometer(cocos2d::Acceleration* acceleration, cocos2d::Event* event){
  int accelerationMult = 10;
  movePlayerIfPossible(_sprPlayer->getPositionX() + (acceleration->x * accelerationMult));
}
```

最后，让我们在`HelloWorldScene.cpp`实现文件的`init`方法末尾调用我们的加速度计初始化代码，如下所示：

```java
initAccelerometer();
```

# 保持屏幕常亮

在上一节中，我们向游戏中添加了加速度计交互，这意味着我们的玩家是通过移动手机而不是触摸屏幕来控制主角的，这将导致许多 Android 设备在一段时间不活动（不触摸屏幕）后关闭屏幕。当然，没有人希望我们的 Android 设备的屏幕突然变黑；为了防止这种情况发生，我们将调用`setKeepScreenOnJni`方法，这是在框架的前一个版本 3.3 中引入的。在此版本之前，这种恼人的情况被认为是框架的缺陷，现在终于得到了修复。

首先，我们需要在`HelloWorldScene.cpp`头文件中包含助手，如下所示：

```java
#include "../cocos2d/cocos/platform/android/jni/Java_org_cocos2dx_lib_Cocos2dxHelper.h"
```

然后，我们将在`HelloWorldScene.cpp`实现文件的`init`方法末尾添加以下行：

```java
setKeepScreenOnJni(true);
```

# 处理 Android 后退键按下事件

我在很多使用 Cocos2d-x 开发的游戏中看到的一个常见错误是，当按下后退按钮时游戏不做任何反应。Android 用户习惯于在想要返回上一个活动时按下后退按钮。如果应用程序在按下后退按钮时没有任何反应，那么它会让用户感到困惑，因为这不符合预期行为，经验不足的用户甚至可能很难退出游戏。

我们可以通过向事件分发器中添加`EventListenerKeyboard`方法，轻松地在用户按下后退按钮时触发自定义代码。

首先，我们将在`HelloWorldScene.h`头文件中添加`initBackButtonListener`和`onKeyPressed`方法的声明，如下代码清单所示：

```java
void initBackButtonListener();
void onKeyPressed(cocos2d::EventKeyboard::KeyCode keyCode, cocos2d::Event* event);
```

现在，让我们在`HelloWorldScene.cpp`实现文件中添加`initBackButtonListener`的实现代码。我们首先实例化`EventListenerKeyboard`类，然后需要指定在`onKeyPressed`事件和`onKeyReleased`事件发生时要调用的方法，否则我们将遇到运行时错误。我们将创建一个空的方法实现，并通过 C++11 的 lambda 表达式将其分配给`onKeyPressed`属性，然后我们将为监听器的`onKeyReleased`属性添加一个回调到我们的`onKeyPressed`方法。然后，像之前所做的那样，我们将这个监听器添加到事件分发机制中：

```java
void HelloWorld::initBackButtonListener(){
  auto listener = EventListenerKeyboard::create();
  listener->onKeyPressed = ={};
  listener->onKeyReleased = CC_CALLBACK_2(HelloWorld::onKeyPressed, this);
  _eventDispatcher->addEventListenerWithSceneGraphPriority(listener, this);
}
```

我们现在将实现`onKeyPressed`方法的代码。这将告诉`Director`，如果按下的键是后退按钮键，则结束游戏：

```java
void HelloWorld::onKeyPressed(EventKeyboard::KeyCode keyCode, Event* event){
  if(keyCode == EventKeyboard::KeyCode::KEY_BACK){
    Director::getInstance()->end();
  }
}
```

最后，我们将在`init`方法的末尾调用`initBackButtonListener`方法，如下所示：

```java
initBackButtonListener();
```

### 注意

请注意，你应该在每个想要捕获后退按钮按下事件的场景中，将`EventListenerKeyboard`监听器添加到事件分发器中。

# 将所有内容整合到一起

在本章中添加了所有代码之后，现在我们的`HelloWorldScene.h`头文件将如下所示：

```java
#ifndef __HELLOWORLD_SCENE_H__
#define __HELLOWORLD_SCENE_H__
#include "cocos2d.h"
class HelloWorld : public cocos2d::Layer
{
public:
  static cocos2d::Scene* createScene();
  virtual bool init();
  void pauseCallback(cocos2d::Ref* pSender);
  CREATE_FUNC(HelloWorld);
private:
  cocos2d::Director *_director;
  cocos2d::Size visibleSize;
  cocos2d::Sprite* _sprBomb;
  cocos2d::Sprite* _sprPlayer;
  void initPhysics();
  bool onCollision(cocos2d::PhysicsContact& contact);
  void setPhysicsBody(cocos2d::Sprite* sprite);
  void initTouch();
  void movePlayerByTouch(cocos2d::Touch* touch, cocos2d::Event* event);
  void movePlayerIfPossible(float newX);
  void movePlayerByAccelerometer(cocos2d::Acceleration* acceleration, cocos2d::Event* event);
  void initAccelerometer();
  void initBackButtonListener();
  void onKeyPressed(cocos2d::EventKeyboard::KeyCode keyCode, cocos2d::Event* event);
};

#endif // __HELLOWORLD_SCENE_H__
```

最终的`HelloWorldScene.cpp`实现文件将如下所示：

```java
#include "HelloWorldScene.h"
#include "PauseScene.h"
#include "../cocos2d/cocos/platform/android/jni/Java_org_cocos2dx_lib_Cocos2dxHelper.h"

USING_NS_CC;
Scene* HelloWorld::createScene()
{
  auto scene = Scene::createWithPhysics();
  scene->getPhysicsWorld()->setGravity(Vect(0,0));
  auto layer = HelloWorld::create();
  //enable debug draw
  //scene->getPhysicsWorld()->setDebugDrawMask(PhysicsWorld::DEBUGDRAW_ALL);
  scene->addChild(layer);
  return scene;
}
```

接下来，我们将尝试移动玩家，使其不会移出屏幕外：

```java
void HelloWorld::movePlayerIfPossible(float newX){
  float sprHalfWidth = _sprPlayer->getBoundingBox().size.width/2;
  if(newX >= sprHalfWidth && newX < visibleSize.width - sprHalfWidth){
    _sprPlayer->setPositionX(newX);
  }
}
void HelloWorld::movePlayerByTouch(Touch* touch, Event* event)
{
  Vec2 touchLocation = touch->getLocation();
  if(_sprPlayer->getBoundingBox().containsPoint(touchLocation)){
    movePlayerIfPossible(touchLocation.x);
  }
}
```

如你所见，在以下两个方法`initTouch`和`initAccelerometer`中，我们为每个初始化任务创建了函数。这将使我们能够简化代码，使其更容易阅读：

```java
void HelloWorld::initTouch()
{
  auto listener = EventListenerTouchOneByOne::create();
  listener->onTouchBegan = [](Touch* touch, Event* event){return true;};
  listener->onTouchMoved = CC_CALLBACK_2(HelloWorld::movePlayerByTouch,this);
  listener->onTouchEnded = ={};
  _eventDispatcher->addEventListenerWithSceneGraphPriority(listener, this);
}
void HelloWorld::initAccelerometer()
{
  Device::setAccelerometerEnabled(true);
  auto listener = EventListenerAcceleration::create(CC_CALLBACK_2(HelloWorld::movePlayerByAccelerometer, this));
  _eventDispatcher->addEventListenerWithSceneGraphPriority(listener, this);
}

void HelloWorld::movePlayerByAccelerometer(cocos2d::Acceleration* acceleration, cocos2d::Event* event)
{
  movePlayerIfPossible(_sprPlayer->getPositionX() + (acceleration->x * 10));
}
```

现在，我们将初始化物理引擎。为此，我们将在`init()`方法中调用`initPhysics()`方法：

```java
bool HelloWorld::init()
{
  if( !Layer::init() ){
    return false;
  }
  _director = Director::getInstance();
  visibleSize = _director->getVisibleSize();
  auto origin = _director->getVisibleOrigin();
  auto closeItem = MenuItemImage::create("pause.png", "pause_pressed.png", CC_CALLBACK_1(HelloWorld::pauseCallback, this));
  closeItem->setPosition(Vec2(visibleSize.width - closeItem->getContentSize().width/2, closeItem->getContentSize().height/2));

  auto menu = Menu::create(closeItem, nullptr);
  menu->setPosition(Vec2::ZERO);
  this->addChild(menu, 1);
  _sprBomb = Sprite::create("bomb.png");
  _sprBomb->setPosition(visibleSize.width/2, visibleSize.height + _sprBomb->getContentSize().height/2);
  this->addChild(_sprBomb,1);
  auto bg = Sprite::create("background.png");
  bg->setAnchorPoint(Vec2());
  bg->setPosition(0,0);
  this->addChild(bg, -1);
  _sprPlayer = Sprite::create("player.png");
  _sprPlayer->setPosition(visibleSize.width/2, visibleSize.height * 0.23);
  setPhysicsBody(_sprPlayer);

  this->addChild(_sprPlayer, 0);
  //Animations
  Vector<SpriteFrame*> frames;
  Size playerSize = _sprPlayer->getContentSize();
  frames.pushBack(SpriteFrame::create("player.png", Rect(0, 0, playerSize.width, playerSize.height)));
  frames.pushBack(SpriteFrame::create("player2.png", Rect(0, 0, playerSize.width, playerSize.height)));
  auto animation = Animation::createWithSpriteFrames(frames,0.2f);
  auto animate = Animate::create(animation);
  _sprPlayer->runAction(RepeatForever::create(animate));

  setPhysicsBody(_sprBomb);
  initPhysics();
  _sprBomb->getPhysicsBody()->setVelocity(Vect(0,-100));
  initTouch();
  initAccelerometer();
  setKeepScreenOnJni(true);
  initBackButtonListener();
  return true;
}

void HelloWorld::pauseCallback(cocos2d::Ref* pSender){
  _director->pushScene(TransitionFlipX::create(1.0, Pause::createScene()));
}
void HelloWorld::initBackButtonListener(){
  auto listener = EventListenerKeyboard::create();
  listener->onKeyPressed = ={};
  listener->onKeyReleased = CC_CALLBACK_2(HelloWorld::onKeyPressed, this);
  _eventDispatcher->addEventListenerWithSceneGraphPriority(listener, this);
}

void HelloWorld::onKeyPressed(EventKeyboard::KeyCode keyCode, Event* event){
  if(keyCode == EventKeyboard::KeyCode::KEY_BACK){
    Director::getInstance()->end();
  }
}
```

如下图所示，我们最终让兔子在屏幕上移动并离开了它的初始中心位置。

![将所有内容整合到一起](img/B04193_04_02.jpg)

### 注意

请注意，在之前的代码列表中，我们省略了与本章不相关的代码部分。如果你想了解到目前为止完整的代码列表是什么样子，你可以查看随书附带的资源材料。

# 总结

在本章中，我们允许用户通过两种不同的输入机制来控制游戏，即触摸屏幕和移动手机以使用加速度传感器；同时，我们还实现了按下后退按钮时游戏暂停的功能。

在下一章，我们将介绍向游戏中添加文本的不同方法。
