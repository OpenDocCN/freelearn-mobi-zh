# 第六章：音频

Cocos2d-x 框架带有一个名为`CocosDenshion`的音频引擎，它从 Cocos2d for iPhone 继承而来。这个引擎封装了播放声音效果和背景音乐的所有复杂性。现在，Cocos2d-x 有一个从零开始构建的新的音频引擎，旨在提供比`CocosDenshion`库更多的灵活性。请注意，没有计划从 Cocos2d-x 框架中消除`CocosDenshion`音频引擎，现在在 Cocos2d-x 中通常会有冗余的组件，以便程序员可以选择更适合他们需求的部分。

本章将涵盖以下主题：

+   播放背景音乐和声音效果

+   修改音频属性

+   离开游戏时处理音频

+   新的音频引擎

# 播放背景音乐和声音效果

为了通过使用`CocosDenshion`音频引擎向我们的游戏中添加背景音乐，第一步是在我们的`HelloWorldScene.cpp`实现文件中添加以下文件包含：

```java
#include "SimpleAudioEngine.h"
```

在这个头文件中，我们将在私有成员部分也添加我们新的`initAudio`方法的声明，该方法将用于启动背景音乐以及预加载每次`player`精灵被炸弹撞击时要播放的音效：

```java
void initAudio();
```

现在，在`HelloWorld.cpp`实现文件中，我们将使用`CocosDenshion`命名空间，这样我们在每次访问音频引擎单例实例时就不必隐式引用这个命名空间：

```java
using namespace CocosDenshion;
```

现在，在同一个实现文件中，我们将编写`initAudio`方法的主体，正如我们之前提到的，它将开始播放循环的背景音乐。我们提供了这一章的源代码，并将预加载每次我们的玩家失败时要播放的音效。`playBackgroundMusic`方法的第二个参数是一个布尔值，它决定了我们是否希望背景音乐永远重复。

```java
void HelloWorld::initAudio()
{
   SimpleAudioEngine::getInstance()->playBackgroundMusic("music.mp3",    true);
   SimpleAudioEngine::getInstance()->preloadEffect("uh.wav");   
}
```

让我们在`Resources`目录中创建一个名为`sounds`的文件夹，这样我们可以把所有的声音文件以有组织的方式放在那里。完成此操作后，我们需要在`AppDelegate.cpp`实现文件中实例化`searchPaths std::vector`之后添加以下行，将`sounds`目录添加到搜索路径中，以便音频引擎可以找到我们的文件：

```java
searchPaths.push_back("sounds");
```

### 注意

我们鼓励您组织您的`Resources`文件夹，为音频和音乐创建一个声音文件夹以及子文件夹，这样我们就不必把所有内容都放在根目录下。

让我们转到每次两个物理体碰撞时调用的`onCollision`方法。如果玩家的精灵物理体涉及到碰撞，那么我们将在添加以下代码行之后停止背景音乐并播放`uh.wav`音效，然后切换到游戏结束场景：

```java
SimpleAudioEngine::getInstance()->stopBackgroundMusic();
SimpleAudioEngine::getInstance()->playEffect("uh.wav");
```

最后，我们将在`HelloWorld.cpp`实现文件中的`init`方法末尾添加对我们`initAudio`方法的调用：

```java
initAudio();
```

# 修改音频属性

您可以通过调用`setBackgroundMusicVolume`方法和`setEffectsVolume`方法轻松修改背景音乐和声音效果的基本音频属性。两者都接收一个`float`类型的参数，其中`0.0`表示静音，`1.0`表示最大音量，如下面的代码清单所示：

```java
SimpleAudioEngine::getInstance()->setBackgroundMusicVolume(0.5f);
SimpleAudioEngine::getInstance()->setEffectsVolume(1.0f);
```

## 处理离开游戏时的音频

当游戏活动不再处于活动状态时，背景音乐和声音效果不会自动停止，应该通过从`AppDelegate`类的`applicationDidEnterBackgound`方法中移除以下注释块来手动停止：

```java
// if you use SimpleAudioEngine, it must be pause
 SimpleAudioEngine::getInstance()->pauseBackgroundMusic();
```

为了让这行新代码工作，我们需要在`HelloWorld.cpp`实现文件中添加相同的行，以便使用`CocosDenshion`命名空间：

```java
using namespace CocosDenshion;
```

当用户切换到另一个应用程序时，您的游戏将停止所有当前的声音。用户一回到我们的游戏，我们就需要恢复音乐。我们可以像以前一样做，但现在，我们将从`AppDelegate`类的`applicationWillEnterForeground`方法中移除以下注释块：

```java
 // if you use SimpleAudioEngine, it must resume here
SimpleAudioEngine::getInstance()->resumeBackgroundMusic();
```

# 新的音频引擎

在 Cocos2d-x 3.4 的实验阶段，从头开始构建了一个新的音频引擎，以便添加更多功能和灵活性。现在 Cocos2d-x 的新音频引擎可用于 Android、iOS、Mac OS 和 win-32 平台。它能够在 Android 平台上同时播放多达 24 个声音；这个数字可能会根据平台的不同而改变。

如果您运行与 Cocos2d-x 框架捆绑的测试，那么您可以测试两个音频引擎。在运行时，它们可能听起来没有明显差异，但它们在内部是非常不同的。

与`CocosDenshion`引擎不同，这个新引擎中声音效果和背景音乐没有区别。因此，与`CocosDenshion`的两个方法—`setBackgroundMusicVolume`和`setEffectsVolume`相比，框架只有一个`setVolume`方法。在本节后面，我们将向您展示如何调整每个播放音频的音量，无论它是声音效果还是背景音乐。

让我们在`HelloWorldScene.h`头文件中添加一个新的方法声明，名为`initAudioNewEngine`，顾名思义，它将初始化我们游戏的音频功能，但现在它将使用新的音频引擎来完成同样的任务。

我们需要在我们的`HelloWorldScene.h`文件中包含新的引擎头文件，如下所示：

```java
#include "audio/include/AudioEngine.h"
```

让我们在`HelloWorld.cpp`实现文件中包含以下代码行，以便我们可以直接调用`AudioEngine`类，而无需每次使用时都引用其命名空间：

```java
using namespace cocos2d::experimental;
```

现在，让我们按照以下方式在我们的实现文件中编写`initAudioNewEngine`方法的代码：

```java
void HelloWorld::initAudioNewEngine()
{   
   if(AudioEngine::lazyInit())
   {
      auto musicId = AudioEngine::play2d("music.mp3");
      AudioEngine::setVolume(musicId, 0.25f);
      CCLOG("Audio initialized successfully");

   }else
   {
      log("Error while initializing new audio engine");
   }   
}
```

与使用单例实例的`CocosDenshion`不同，新音频引擎的所有方法都是静态声明的。

从前面的代码清单中我们可以看到，在调用`play2d`方法之前，我们调用了`lazyInit`方法。尽管`play2d`内部调用了`lazyInit`方法，但我们希望尽快知道我们的 Android 系统是否能够播放音频并采取行动。请注意，当`play2d`方法返回`AudioEngine::INVALID_AUDIO_ID`值时，您还需要找出音频初始化是否出现了问题。

每次我们通过调用`play2d`方法播放任意声音时，它都会返回一个唯一的递增的基于零的`audioID`索引，我们将保存它，这样每次我们想要对该特定音频实例执行特定操作时，比如更改音量、移动到特定位置、停止、暂停或恢复，我们都可以引用它。

新音频引擎的一个缺点是它仍然支持有限的音频格式。它目前不支持`.wav`文件。因此，为了播放`uh.wav`声音，我们将它转换为 mp3，然后在`onCollision`方法中通过如下调用`play2d`来播放：

```java
AudioEngine::stopAll();
AudioEngine::play2d("uh.mp3");
```

我们在本章提供的代码资源存档中包含了新的`uh.mp3`音频文件。

对于我们的游戏，我们将实施两种方案；传统的`CocosDenshion`引擎，这是最成熟的音频引擎，为我们提供了所需的基本功能，比如播放音效和背景音乐；以及新引擎中的相同音频功能。

## 新音频引擎中包含的新功能

`play2d`方法被重载，以便我们可以指定是否希望声音循环播放、初始音量以及我们希望应用的声音配置文件。`AudioProfile`类是 Cocos2d-x 框架的一部分，它只有三个属性：`name`，不能为空；`maxInstances`，将定义将同时播放多少个声音；以及`minDelay`，它是一个`double`数据类型，将指定声音之间的最小延迟。

新音频引擎具有的另一个功能是，通过调用`setCurrentTime`方法并传递`audioID`方法和以秒为单位的自定义位置（由`float`表示）来从自定义位置播放音频。

在新音频引擎中，您可以指定在给定音频实例播放完成时您希望调用的函数。这可以通过调用`setFinishCallback`方法来实现。

每次播放音频时，它都会被缓存，因此无需再次从文件系统中读取。如果我们想要释放一些资源，可以调用`uncacheAll`方法来移除音频引擎内部用于回放音频的所有缓冲区，或者可以通过调用`uncache`方法并指定要移除的文件系统中的文件路径来从缓存中移除任何特定的音频。

本节的目的是让您了解另一个处于实验阶段的音频引擎，如果`CocosDenshion`没有您想要添加到游戏中的任何音频功能，那么您应该检查另一个音频引擎，看看它是否具备您所需的功能。

### 注意

新的音频引擎可以在 Mac OS、iOS 和 win-32 平台上同时播放多达 32 个声音，但在 Android 上只能同时播放多达 24 个声音。

# 向我们的游戏添加静音按钮

在本章结束之前，我们将在游戏中添加一个静音按钮，这样我们就可以通过一次触摸将音效和背景音乐音量设置为零。

为了实现这一点，我们将在`HelloWorld`类中添加两个方法；一个用于初始化按钮，另一个用于实际静音所有声音。

为了实现这一点，我们将在`HelloWorldScene.h`头文件的私有部分添加以下几行：

```java
int _musicId;
cocos2d::MenuItemImage* _muteItem;
cocos2d::MenuItemImage* _unmuteItem;
void initMuteButton();
void muteCallback(cocos2d::Ref* pSender);
```

现在，我们将以下`initMuteButton`实现代码添加到`HelloWorldScene.cpp`文件中：

```java
void HelloWorld::initMuteButton()
{
   _muteItem = MenuItemImage::create("mute.png", "mute.png", CC_CALLBACK_1(HelloWorld::muteCallback, this));    

   _muteItem->setPosition(Vec2(_visibleSize.width - _muteItem-  >getContentSize().width/2 ,
   _visibleSize.height - _muteItem->getContentSize(). height / 2));
   _unmuteItem = MenuItemImage::create("unmute.png", "unmute.png", CC_CALLBACK_1(HelloWorld::muteCallback, this));    

   _unmuteItem->setPosition(Vec2(_visibleSize.width - _unmuteItem- >getContentSize().width/2 , _visibleSize.height - _unmuteItem->getContentSize().height /2));
   _unmuteItem -> setVisible(false);

   auto menu = Menu::create(_muteItem, _unmuteItem , nullptr);
   menu->setPosition(Vec2::ZERO);
   this->addChild(menu, 1);
}
```

如您所见，我们刚刚创建了一个新的菜单，我们在其中添加了两个按钮，一个用于静音游戏，另一个不可见用于取消静音。我们将这些分别存储在成员变量中，这样我们就可以通过在以下代码清单中声明的`muteCallback`方法访问它们：

```java
void HelloWorld::muteCallback(cocos2d::Ref* pSender)
{   
   if(_muteItem -> isVisible())
   {

      //CocosDenshion
      //SimpleAudioEngine::getInstance()->setBackgroundMusicVolume(0);
      AudioEngine::setVolume(_musicId, 0);
   }else
   {   
      //SimpleAudioEngine::getInstance()->setBackgroundMusicVolume(1);
      AudioEngine::setVolume(_musicId, 1);
   }

   _muteItem->setVisible(!_muteItem->isVisible());
   _unmuteItem->setVisible(!_muteItem->isVisible());
}
```

在这里，我们基本上只是判断`_muteItem`菜单项是否可见。如果可见，则通过使用新的音频引擎`CocosDenshion`将音量设置为零，否则将音量设置为最大值，即一。在任何一种情况下，都要改变静音和取消静音菜单项的实际可见值。

我们可以在以下屏幕截图中看到最终结果：

![向我们的游戏添加静音按钮](img/B04193_06_01.jpg)

# 把所有内容放在一起

在我们添加了将`sounds`文件夹包含在`resources`路径中的行之后，我们的`AppDelegate.cpp`实现文件中的`applicationDidFinishLaunching`方法如下所示：

```java
bool AppDelegate::applicationDidFinishLaunching() {
    auto director = Director::getInstance();
    // OpenGL initialization done by cocos project creation script
    auto glview = director->getOpenGLView();
    if(!glview) {
    glview = GLViewImpl::create("Happy Bunny");
    glview->setFrameSize(480, 800);
    director->setOpenGLView(glview);
   }

   Size screenSize = glview->getFrameSize();
   Size designSize(768, 1280);
   std::vector<std::string> searchPaths;   
 searchPaths.push_back("sounds");

   if (screenSize.height > 800){
      //High Resolution
      searchPaths.push_back("images/high");
      director->setContentScaleFactor(1280.0f / designSize.height);
   }
   else if (screenSize.height > 600){
      //Mid resolution
      searchPaths.push_back("images/mid");
      director->setContentScaleFactor(800.0f / designSize.height);
   }
   else{
      //Low resolution
      searchPaths.push_back("images/low");
      director->setContentScaleFactor(320.0f / designSize.height);
   }
   FileUtils::getInstance()->setSearchPaths(searchPaths);
   glview->setDesignResolutionSize(designSize.width, designSize. height, ResolutionPolicy::EXACT_FIT);
   auto scene = HelloWorld::createScene();
   director->runWithScene(scene);
   return true;
}
```

下面的代码清单显示了我们在本章中进行更改后`HelloWorldScene.h`头文件的样子：

```java
#ifndef __HELLOWORLD_SCENE_H__
#define __HELLOWORLD_SCENE_H__

#include "cocos2d.h"
#include "PauseScene.h"
#include "GameOverScene.h"

class HelloWorld : public cocos2d::Layer
{
public:
    static cocos2d::Scene* createScene();
    virtual bool init();
    CREATE_FUNC(HelloWorld);
private:
   cocos2d::Director *_director;
   cocos2d::Size _visibleSize;   
   cocos2d::Sprite* _sprBomb;
   cocos2d::Sprite* _sprPlayer;   
   cocos2d::MenuItemImage* _muteItem;
   cocos2d::MenuItemImage* _unmuteItem;   
   int _score;
   int _musicId;
   void initPhysics();
   void pauseCallback(cocos2d::Ref* pSender);
   void muteCallback(cocos2d::Ref* pSender);
   bool onCollision(cocos2d::PhysicsContact& contact);
   void setPhysicsBody(cocos2d::Sprite* sprite);
   void initTouch();
   void movePlayerByTouch(cocos2d::Touch* touch, cocos2d::Event* event);
   void movePlayerIfPossible(float newX);
   void movePlayerByAccelerometer(cocos2d::Acceleration* acceleration, cocos2d::Event* event);
   void initAccelerometer();
   void initBackButtonListener();
   void onKeyPressed(cocos2d::EventKeyboard::KeyCode keyCode, cocos2d::Event* event);
   void updateScore(float dt);
   void addBombs(float dt);   
   void initAudio();
   void initAudioNewEngine();
   void initMuteButton();
};

#endif // __HELLOWORLD_SCENE_H__
```

最后，在添加了音频管理代码之后，我们的`HelloWorldScene.cpp`实现文件如下所示：

```java
#include "HelloWorldScene.h"#include "SimpleAudioEngine.h"
#include "audio/include/AudioEngine.h"
#include "../cocos2d/cocos/platform/android/jni/Java_org_cocos2dx_lib_Cocos2dxHelper.h"

USING_NS_CC;
using namespace CocosDenshion;
using namespace cocos2d::experimental;

Scene* HelloWorld::createScene()
{
  //no changes here
}

// physics code …

// event handling code …
```

在以下方法中，我们将通过使用新的音频引擎来初始化音频。注意，我们会将音频实例的背景音乐的 ID 存储在`_musicId`整型成员变量中：

```java
void HelloWorld::initAudioNewEngine()
{   
   if(AudioEngine::lazyInit())
   {      
      _musicId = AudioEngine::play2d("music.mp3");
      AudioEngine::setVolume(_musicId, 1);      
      AudioEngine::setLoop(_musicId,true);      
      CCLOG("Audio initialized successfully");
   }else
   {
      CCLOG("Error while initializing new audio engine");
   }   
}
```

在这里，我们执行了与上一个方法中相同的初始化工作，但现在我们是使用`CocosDenshion`音频引擎来完成：

```java
void HelloWorld::initAudio()
{
   SimpleAudioEngine::getInstance()->playBackgroundMusic("music. mp3",true);   
   SimpleAudioEngine::getInstance()->preloadEffect("uh.wav");   
   SimpleAudioEngine::getInstance()->setBackgroundMusicVolume(1.0f);
}
```

在以下方法中，我们创建了一个简单的菜单，以展示静音和取消静音游戏的选项。这里我们将静音和取消静音的精灵存储在对应的成员变量中，以便我们可以在`muteCallback`方法中稍后访问它们，并操作它们的`visibility`属性：

```java
void HelloWorld::initMuteButton()
{
   _sprMute = Sprite::create("mute.png");
   _sprUnmute = Sprite::create("unmute.png");
   _muteItem = MenuItemImage::create("mute.png", "mute.png", CC_CALLBACK_1(HelloWorld::muteCallback, this));    
   _muteItem->setPosition(Vec2(_visibleSize.width - _muteItem- >getContentSize().width/2 ,
   _visibleSize.height - _muteItem->getContentSize().height / 2));
   _unmuteItem = MenuItemImage::create("unmute.png", "unmute.png", CC_CALLBACK_1(HelloWorld::muteCallback, this));    
   _unmuteItem->setPosition(Vec2(_visibleSize.width - _unmuteItem->getContentSize().width/2 ,
   _visibleSize.height - _unmuteItem->getContentSize(). height /2));
   _unmuteItem -> setVisible(false);
   auto menu = Menu::create(_muteItem, _unmuteItem , nullptr);
    menu->setPosition(Vec2::ZERO);
    this->addChild(menu, 1);

}
```

以下方法将在每次按下静音或取消静音菜单项时被调用，在这个方法中，我们只需将音量设置为 0，并根据触摸的选项显示静音或取消静音按钮：

```java
void HelloWorld::muteCallback(cocos2d::Ref* pSender)
{   
   if(_muteItem -> isVisible())
   {
      //CocosDenshion
      //SimpleAudioEngine::getInstance()->setBackgroundMusicVolume(0);
      AudioEngine::setVolume(_musicId, 0);   

   }else
   {   
      //SimpleAudioEngine::getInstance()->setBackgroundMusicVolume(1);
      AudioEngine::setVolume(_musicId, 1);
   }

   _muteItem->setVisible(!_muteItem->isVisible());
   _unmuteItem->setVisible(!_muteItem->isVisible());
}
```

我们对`init`方法做的唯一修改是在其最后添加了对`initMuteButton();`方法的调用：

```java
bool HelloWorld::init()
{
    if ( !Layer::init() )
    {
        return false;
    }
   _score = 0;
   _director = Director::getInstance();
   _visibleSize = _director->getVisibleSize();
   auto origin = _director->getVisibleOrigin();
   auto closeItem = MenuItemImage::create("pause.png", "pause_pressed.png", CC_CALLBACK_1(HelloWorld::pauseCallback, this));
   closeItem->setPosition(Vec2(_visibleSize.width - closeItem->getContentSize().width/2, closeItem->getContentSize().height/2));
   auto menu = Menu::create(closeItem, nullptr);
   menu->setPosition(Vec2::ZERO);
   this->addChild(menu, 1);
   _sprBomb = Sprite::create("bomb.png");   
   _sprBomb->setPosition(_visibleSize.width / 2, _visibleSize.height +_sprBomb->getContentSize().height/2);
   this->addChild(_sprBomb,1);
   auto bg = Sprite::create("background.png");
   bg->setAnchorPoint(Vec2());
   bg->setPosition(0,0);
   this->addChild(bg, -1);
   _sprPlayer = Sprite::create("player.png");   
   _sprPlayer->setPosition(_visibleSize.width / 2, _visibleSize.height* 0.23);
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
   #if (CC_TARGET_PLATFORM == CC_PLATFORM_ANDROID)
   setKeepScreenOnJni(true);
   #endif
   initBackButtonListener();
   schedule(CC_SCHEDULE_SELECTOR(HelloWorld::updateScore), 3.0f);
   schedule(CC_SCHEDULE_SELECTOR(HelloWorld::addBombs), 8.0f);
   initAudioNewEngine();
 initMuteButton();
   return true;
}
```

如你所见，尽管我们使用了新的音频引擎来播放声音，但我们展示了使用传统`CocosDenshion`音频引擎所需的所有代码。为了启用`CocosDenshion`实现，你只需在`HelloWorld.cpp`文件的`init`方法的底部调用`initAudio`方法，而不是调用`initAudioNewEngine`方法，最后，你还需要在`onCollision`方法中移除`CocosDenshion`实现代码的注释斜杠，并注释掉新的音频引擎播放代码。

# 总结

在本章中，我们通过使用 Cocos2d-x 框架捆绑的两个音频引擎，以非常简单的方式为我们的游戏添加了背景音乐和音效。

在下一章中，我们将介绍如何将粒子系统添加到我们的游戏中，以模拟每次炸弹击中`player`精灵时的更真实的爆炸效果。
