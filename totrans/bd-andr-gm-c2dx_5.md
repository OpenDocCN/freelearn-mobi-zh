# 第五章. 处理文本和字体

在我们的游戏中，添加文本以向玩家显示信息是非常常见的。这可以通过使用 TrueType 字体或位图字体来完成，这将给我们带来更大的灵活性，实际上，这是专业游戏中使用最广泛的字体类型，因为它允许我们为游戏定制外观。本章将涵盖以下主题：

+   创建 TrueType 字体标签

+   添加标签效果

+   创建系统字体

+   创建位图字体标签

# 创建 TrueType 字体标签

使用 TrueType 字体添加文本非常简单。打开我们在第二章*图形*中创建的`PauseScene.cpp`实现文件。在`init`方法中，你会看到我们通过调用静态方法`createWithTTF`创建了一个`Label`类的实例。这个方法接收三个参数，第一个是我们想要绘制的字符串，第二个是表示你想要使用的字体文件的字符串，包括它在`Resources`文件夹中的路径，第三个是字体大小。

### 注意

`Label`类在 Cocos2d-x 3.x 版本中引入。它将 TrueType 字体和位图字体处理合并到一个单一类中。然而，尽管已弃用，为了兼容性，之前的标签处理类仍然在 API 中可用。

现在，让我们将`createWithTTF`方法中的第三个参数值从 24 更改为 96，使字体变得更大：

```java
auto label = Label::createWithTTF("PAUSE", "fonts/Marker Felt.ttf", 96);
```

### 注意

`cocos new`命令生成的模板 Cocos2d-x 项目中包含了 Marker Felt 字体。

## 创建我们的 GameOverScene

现在是创建游戏结束场景的时候了，一旦炸弹与我们的`bunny`精灵相撞，就会显示这个场景。

我们将通过复制`Classes`目录中的`PauseScene.cpp`和`PauseScene.h`文件，并将它们分别重命名为`GameOverScene.cpp`和`GameOverScene.h`来完成这一操作。

### 提示

请记住，每次你向 Cocos2d-x 文件夹添加新的源文件时，都需要将类添加到`jni`文件夹中的`Android.mk`文件中，这样在下次构建时就会编译这个新的源文件。

现在，在`GameOverScene.h`和`GameOverScene.cpp`文件中，对这两个文件执行查找和替换操作，将单词`Pause`替换为单词`GameOver`。

最后，将`GameOverScene.cpp`实现文件中的前几行代码替换为以下内容：

```java
#include "GameOverScene.h"
#include "HelloWorldScene.h"
```

在`GameOverScene.cpp`实现文件中的`exitPause`方法体内，我们将用以下这行代码替换这个方法中的唯一一行：

```java
   Director::getInstance()->replaceScene(TransitionFlipX:: create(1.0, HelloWorld::createScene()));;
```

## 当玩家失败时调用我们的 GameOverScene

我们已经创建了游戏结束场景；现在让我们在炸弹与我们的`player`精灵相撞时立即显示它。为了实现这一点，我们将在`HelloWorld`类中的`onCollision`方法中添加以下代码行。

```java
_director->replaceScene(TransitionFlipX::create(1.0, GameOver::createScene()));
```

现在，通过在`HelloWorldScene.h`头文件的开始处添加以下行，将游戏结束场景头文件包含到我们的`gameplay`类中：

```java
#include "GameOverScene.h"
```

## 自定义`GameOverScene`

现在，我们不希望有黑色背景，所以我们将添加我们在第二章*图形*中在游戏玩法中使用的相同背景：

```java
  auto bg = Sprite::create("background.png");
  bg->setAnchorPoint(Vec2());
  bg->setPosition(0,0);
  this->addChild(bg, -1);
```

现在，我们将更改从`PauseScene`复制的 TrueType 字体标签，它现在将显示为`Game Over`。在下一节中，我们将给这个标签添加一些效果。

```java
   auto label = Label::createWithTTF("Game Over", "fonts/Marker Felt.  ttf", 96);
```

## 添加标签效果

现在，我们将添加仅适用于 TrueType 字体的效果。

让我们为我们的字体启用轮廓。`Label`类的`enableOutline`方法接收两个参数，一个`Color4B`实例和一个整数，表示轮廓大小——数字越大，轮廓越粗：

```java
  label->enableOutline(Color4B(255, 0, 0, 100),6);
```

现在，让我们给字体添加一些发光效果：

```java
  label->enableGlow(Color4B(255, 0, 0, 255));
```

最后，让我们给标签添加阴影效果，目前所有三种标签类型都支持这一效果。

```java
  label->enableShadow();
```

你会从以下屏幕截图中注意到，效果相互重叠，所以请决定哪个效果看起来更好：

![添加标签效果](img/B04193_05_01.jpg)

`Color4B`构造方法接收四个参数。前三个是**红、绿、蓝**（**RGB**）分量，第四个是`alpha`分量。这将允许我们添加一些透明度效果，其值可以从 0 到 255。标签实例不支持自定义效果，例如给文本中的每个单词着不同的颜色，为单个文本使用不同的字体，或者在标签中嵌入图像。

### 提示

如果你有兴趣在你的游戏中添加这些字体效果，你可以使用 Luma Stubma 创建的`CCRichLabelTTF`类。这可以在[`github.com/stubma/cocos2dx-better`](https://github.com/stubma/cocos2dx-better)找到。

# 创建系统字体

你可以创建使用宿主操作系统的字体的标签；因此，不需要提供字体文件。建议只将这种标签用于测试目的，因为它会降低框架的灵活性，因为选定的字体可能不在用户的 Android 操作系统版本上可用。

为了测试，在我们当前文本下方，我们将在`GameOverScene.cpp`实现文件的`init`方法中添加以下标签：

```java
auto label2 = Label::createWithSystemFont("Your score is", "Arial", 48);
label2->setPosition(origin.x + visibleSize.width/2,origin.y + visibleSize.height /2.5);
this->addChild(label2, 1);
```

这段代码产生了以下结果：

![创建系统字体](img/B04193_05_02.jpg)

# 创建位图字体标签

到目前为止，我们已经看到了如何通过使用 TrueType 和系统字体轻松创建标签，现在我们将执行一些额外步骤，以使我们的标签具有更专业的风格。如前所述，位图字体是专业游戏中最常用的标签类型。

如其名称所示，位图字体是由代表每个字符的图像生成的，这将允许我们绘制任何我们想要的字体，但它将具有位图的所有缺点，例如标签可能被像素化的风险，处理不同尺寸时的灵活性不足，以及处理这类字体所需的磁盘和 RAM 额外空间。

有多种应用程序可用于创建位图字体。最常见的是**Glyph Designer**，你可以在[`71squared.com`](https://71squared.com)获取它。这个应用程序最初是为 Mac OS 发布的，但在 2015 年初，也为 Windows 发布了**Glyph Designer X**。你还可以使用免费的在线应用程序**Littera**来创建自己的位图字体。它可以在[`kvazars.com/littera`](http://kvazars.com/littera)找到。为了本书的需要，我们在章节中包含了位图字体的代码。我们将使用这个位图字体代码在游戏结束场景中显示玩家的总分。

## 向我们的游戏中添加更多炸弹

考虑到现在我们有一个游戏结束场景，让我们通过添加更多炸弹使这个游戏变得稍微困难一些。我们将使用 Cocos2d-x 调度器机制，它将允许我们在每个给定的时间段内调用一个方法。我们将`addBombs`方法添加到`HelloWorldScene`类中，并在前述类的`init`方法内调度它，使其每八秒被调用一次：

```java
schedule(CC_SCHEDULE_SELECTOR(HelloWorld::addBombs), 8.0f);
```

我们将向场景中添加三个随机位置的炸弹，每次调用`addBombs`方法时都会发生这种情况：

```java
void HelloWorld::addBombs(float dt)
{
   Sprite* bomb = nullptr;
   for(int i = 0 ; i < 3 ; i++)
   {
         bomb = Sprite::create("bomb.png");
         bomb->setPosition(CCRANDOM_0_1() * visibleSize.width,   visibleSize.height + bomb->getContentSize().height/2);
         this->addChild(bomb,1);
         setPhysicsBody(bomb);
         bomb->getPhysicsBody()->setVelocity(Vect(0, ( (CCRANDOM_0_1() + 0.2f) * -250) ));
   }
}
```

这段代码产生了以下结果：

![向我们的游戏中添加更多炸弹](img/B04193_05_03.jpg)

### 注意

使用`CC_SCHEDULE_SELECTOR`宏，我们创建了一个自定义选择器，在这种情况下称为**自定义时间间隔**。所选函数应该接收一个`float`参数，代表自上次调用和当前调用之间经过的时间，以便你可以独立于硬件处理速度计算统一的游戏节奏。如果你没有将第二个`float`参数传递给调度函数，那么它将在每个帧中执行所选函数。

在场景中，我们还将向调度器添加另一个方法，该方法每三秒调用一次，并将为玩家的分数增加 10 分。因此，玩家能够避免被炸弹击中的时间越长，他的得分就越高。

现在我们有超过两个物理体，这意味着我们必须修改我们的`onCollision`方法，使其只有在`player`精灵参与碰撞时才切换到`gameOverScene`。为此，我们将在方法开始处添加以下代码行：

```java
auto playerShape = _sprPlayer->getPhysicsBody()->getFirstShape();
if(playerShape != contact.getShapeA() && playerShape != contact.getShapeB())
   {
      return false;
   }
```

如果该方法没有返回，这意味着玩家精灵确实参与了碰撞。因此，我们将使用 Cocos2d-x 内置的存储机制来写入存储在成员变量`_score`中的玩家分数：

```java
UserDefault::getInstance()->setIntegerForKey("score",_score);
```

### 注意

`UserDefault`类使我们能够访问 Cocos2d-x 的数据存储机制。它可以存储`bool`、`int`、`float`、`double`和`string`类型的值。通过使用此类存储的数据可以通过调用`flush`方法来持久化，该方法将数据存储在 XML 文件中。

我们可以像创建 TrueType 字体和系统字体那样创建我们的位图字体。我们将在`GameOverScene.cpp`实现文件的`init`方法中添加以下代码行：

```java
char scoreText[32];
int score = UserDefault::getInstance()->getIntegerForKey("score",0);
sprintf(scoreText, "%d", score);
auto label3 = Label::createWithBMFont("font.fnt", scoreText);
label3->setPosition(origin.x + visibleSize.width/2,origin.y + visibleSize.height /3.5);
this->addChild(label3, 1);
```

上述代码将产生以下结果：

![向我们的游戏中添加更多炸弹](img/B04193_05_04.jpg)

# 把所有内容放在一起

在我们所有的修改之后，这就是我们的`HelloWorldScene.h`头文件的样子：

```java
#ifndef __HELLOWORLD_SCENE_H__
#define __HELLOWORLD_SCENE_H__

#include "cocos2d.h"

#include "PauseScene.h"
#include "GameOverScene.h"

```

在本章中，我们对这个头文件唯一做的更改是包含了`GameOverScene.h`：

```java
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
   int _score;
   void initPhysics();
   bool onCollision(cocos2d::PhysicsContact& contact);
   void setPhysicsBody(cocos2d::Sprite* sprite);
   void initTouch();
   void movePlayerByTouch(cocos2d::Touch* touch, cocos2d::Event*  event);
   void movePlayerIfPossible(float newX);
   void movePlayerByAccelerometer(cocos2d::Acceleration*  acceleration, cocos2d::Event* event);
   void initAccelerometer();
   void initBackButtonListener();
   void onKeyPressed(cocos2d::EventKeyboard::KeyCode keyCode,    cocos2d::Event* event);
   void updateScore(float dt);
   void addBombs(float dt);   
};

#endif // __HELLOWORLD_SCENE_H__
```

现在，我们的`HelloWorldScene.cpp`实现文件看起来像这样：

```java
#include "HelloWorldScene.h"
#include "../cocos2d/cocos/platform/android/jni/Java_org_cocos2dx_lib_Cocos2dxHelper.h"

USING_NS_CC;

Scene* HelloWorld::createScene()
{
   auto scene = Scene::createWithPhysics();   
   scene->getPhysicsWorld()->setGravity(Vect(0,0));
   auto layer = HelloWorld::create();
   //enable debug draw
   //scene->getPhysicsWorld()->setDebugDrawMask(PhysicsWorld::DEBUGDR  AW_ALL);
   scene->addChild(layer);
   return scene;
}
```

我们现在将添加事件和物理的代码：

```java
void HelloWorld::updateScore(float dt)
{
   _score += 10;
}

void HelloWorld::addBombs(float dt)
{
   Sprite* bomb = nullptr;
   for(int i = 0 ; i < 3 ; i++)
   {
      bomb = Sprite::create("bomb.png");   
      bomb->setPosition(CCRANDOM_0_1() * visibleSize.width,  visibleSize.height + bomb->getContentSize().height/2);
      this->addChild(bomb,1);
      setPhysicsBody(bomb);
      bomb->getPhysicsBody()->setVelocity(Vect(0,  ( (CCRANDOM_0_1() + 0.2f) * -250) ));
   }
}

}

bool HelloWorld::init()
{
    if ( !Layer::init() )
    {
        return false;
    }
   _score = 0;
   _director = Director::getInstance();
   visibleSize = _director->getVisibleSize();
   auto origin = _director->getVisibleOrigin();
   auto closeItem = MenuItemImage::create("pause.png", "pause_pressed.png", CC_CALLBACK_1(HelloWorld::pauseCallback, this));

   closeItem->setPosition(Vec2(visibleSize.width - closeItem-  >getContentSize().width/2, closeItem->getContentSize().height/2));

   auto menu = Menu::create(closeItem, nullptr);
   menu->setPosition(Vec2::ZERO);
   this->addChild(menu, 1);
   _sprBomb = Sprite::create("bomb.png");
   _sprBomb->setPosition(visibleSize.width / 2,  visibleSize.height + _sprBomb->getContentSize().height/2);
   this->addChild(_sprBomb,1);
   auto bg = Sprite::create("background.png");
   bg->setAnchorPoint(Vec2());
   bg->setPosition(0,0);
   this->addChild(bg, -1);
   _sprPlayer = Sprite::create("player.png");   
   _sprPlayer->setPosition(visibleSize.width / 2, visibleSize.height * 0.23);
   setPhysicsBody(_sprPlayer);
   this->addChild(_sprPlayer, 0);
   //Animations
   Vector<SpriteFrame*> frames;
   Size playerSize = _sprPlayer->getContentSize();
   frames.pushBack(SpriteFrame::create("player.png",  Rect(0, 0, playerSize.width, playerSize.height)));
   frames.pushBack(SpriteFrame::create("player2.png",  Rect(0, 0, playerSize.width, playerSize.height)));
   auto animation =  Animation::createWithSpriteFrames(frames,0.2f);
   auto animate = Animate::create(animation);
   _sprPlayer->runAction(RepeatForever::create(animate));   

   setPhysicsBody(_sprBomb);   
   initPhysics();   
   _sprBomb->getPhysicsBody()->setVelocity(Vect(0,-100));   
   initTouch();
   initAccelerometer();   
   setKeepScreenOnJni(true);
   initBackButtonListener();
   schedule(CC_SCHEDULE_SELECTOR (HelloWorld::updateScore), 3.0f);
   schedule(CC_SCHEDULE_SELECTOR (HelloWorld::addBombs), 8.0f);
   return true;
}

void HelloWorld::pauseCallback(cocos2d::Ref* pSender){
   _director->pushScene(TransitionFlipX::create(1.0, Pause::createScene()));
}
```

我们的`GameOverScene.h`头文件现在看起来像这样：

```java
#ifndef __GameOver_SCENE_H__
#define __GameOver_SCENE_H__

#include "cocos2d.h"
#include "HelloWorldScene.h"

class GameOver : public cocos2d::Layer
{
public:
    static cocos2d::Scene* createScene();
    virtual bool init();    
    void exitPause(cocos2d::Ref* pSender);
    CREATE_FUNC(GameOver);
private:
   cocos2d::Sprite* sprLogo;
   cocos2d::Director *director;
   cocos2d::Size visibleSize;   
};

#endif // __Pause_SCENE_H__
```

最后，我们的`GameOverScene.cpp`实现文件将看起来像这样：

```java
#include "GameOverScene.h"

USING_NS_CC;

Scene* GameOver::createScene()
{
    auto scene = Scene::create();
    auto layer = GameOver::create();
    scene->addChild(layer);
    return scene;
}

bool GameOver::init()
{
    if ( !Layer::init() )
    {
        return false;
    }
   director = Director::getInstance();  
   visibleSize = director->getVisibleSize();
   Vec2 origin = director->getVisibleOrigin();
   auto pauseItem = MenuItemImage::create("play.png", "play_pressed.png", CC_CALLBACK_1(GameOver::exitPause, this));
   pauseItem->setPosition(Vec2(origin.x + visibleSize.width -   pauseItem->getContentSize().width / 2, origin.y + pauseItem-  >getContentSize().height / 2));
   auto menu = Menu::create(pauseItem, NULL);
   menu->setPosition(Vec2::ZERO);
   this->addChild(menu, 1);
   auto bg = Sprite::create("background.png");
   bg->setAnchorPoint(Vec2());
   bg->setPosition(0,0);
   this->addChild(bg, -1);
```

在以下代码行中，我们创建了在本章中介绍的三种字体类型：

```java
   auto label = Label::createWithTTF("Game Over", "fonts/Marker  Felt.ttf", 96);
   label->enableOutline(Color4B(255, 0, 0, 100),6);
   label->enableGlow(Color4B(255, 0, 0, 255));
   label->enableShadow();
   label->setPosition(origin.x + visibleSize.width/2,  origin.y + visibleSize.height /2);
   this->addChild(label, 1);
   auto label2 = Label::createWithSystemFont("Your score is",  "Arial", 48);
   label2->setPosition(origin.x + visibleSize.width/2,origin.y  + visibleSize.height/2.5);
   this->addChild(label2, 1);
   char scoreText[32];
   int score = UserDefault::getInstance()- >getIntegerForKey("score",0);
   sprintf(scoreText, "%d", score);
   auto label3 = Label::createWithBMFont("font.fnt", scoreText);
   label3->setPosition(origin.x + visibleSize.width/2,origin.y  + visibleSize.height /3.5);
   this->addChild(label3, 1);
   return true;
}

void GameOver::exitPause(cocos2d::Ref* pSender){
   Director::getInstance()- >replaceScene(TransitionFlipX::create(1.0, HelloWorld::createScene()));
}
```

# 总结

在本章中，我们了解了如何使用 TrueType 字体、系统字体和位图字体向游戏中添加文本，以及如何为这些文本添加效果。标签创建非常简单；您只需要调用其创建的静态方法，并将其添加到场景中后，就可以像在屏幕上定位精灵一样在屏幕上定位它们。

在下一章中，我们将介绍在版本 3 中从头开始编写的新音频引擎，以替代自其前身`cocos2d` for iPhone 以来与引擎捆绑的传统`CocosDenshion`音频引擎。
