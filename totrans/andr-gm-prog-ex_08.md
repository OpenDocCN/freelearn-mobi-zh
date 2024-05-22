# 第八章. 平台游戏——整合所有功能

最后，我们将让子弹造成一些伤害。当子弹的能量被一团草地吸收时，这种反弹声非常令人满意。我们将添加大量的新平台类型和非动画场景对象，使我们的关卡更有趣。通过实现多个滚动视差背景，我们将提供一种真正的运动感和沉浸感。

我们还将添加一个动画火焰瓦片，让玩家避开，此外，还会添加一个特殊的`Teleport`类，将各个关卡连接成一个可玩的游戏。然后，我们将使用所有的游戏对象和背景创建四个连接、完全可玩的游戏关卡。

然后，我们将添加一个 HUD 来跟踪拾取物和生命值。最后，我们将讨论一些无法在这四章中容纳的精彩内容。

# 子弹碰撞检测

检测子弹碰撞相当直接。我们遍历由我们的`MachineGun`对象持有的所有现有`Bullet`对象。接下来，我们将每个子弹的点转换成`RectHitBox`对象，并使用`intersects()`方法测试我们的视口中的每个对象。

如果我们受到攻击，我们会检查它击中的对象类型。然后，我们会切换到处理我们关心的每种类型的对象。如果是`Guard`对象，我们将其稍微击退一点；如果是`Drone`对象，我们将其销毁；如果是其他任何对象，我们只需让子弹消失，并播放一种沉闷的/反弹声。

我们只需在我们处理玩家碰撞的`switch`块之后，但在我们调用所有未剪辑对象的`update()`之前，放置我们讨论过的这个逻辑，如下所示：

```java
default:// Probably a regular tile
    if (hit == 1) {// Left or right
        lm.player.setxVelocity(0);
        lm.player.setPressingRight(false);
    }

   if (hit == 2) {// Feet
        lm.player.isFalling = false;
    }
    break;
}
}

//Check bullet collisions
for (int i = 0; i < lm.player.bfg.getNumBullets(); i++) {
 //Make a hitbox out of the the current bullet
 RectHitbox r = new RectHitbox();
 r.setLeft(lm.player.bfg.getBulletX(i));
 r.setTop(lm.player.bfg.getBulletY(i));
 r.setRight(lm.player.bfg.getBulletX(i) + .1f);
 r.setBottom(lm.player.bfg.getBulletY(i) + .1f);

 if (go.getHitbox().intersects(r)) {
 // Collision detected
 // make bullet disappear until it 
 // is respawned as a new bullet
 lm.player.bfg.hideBullet(i);

 //Now respond depending upon the type of object hit
 if (go.getType() != 'g' && go.getType() != 'd') {
 sm.playSound("ricochet");

 } else if (go.getType() == 'g') {
 // Knock the guard back
 go.setWorldLocationX(go.getWorldLocation().x +
 2 * (lm.player.bfg.getDirection(i)));

 sm.playSound("hit_guard");

 } else if (go.getType() == 'd') {
 //destroy the droid
 sm.playSound("explode");
 //permanently clip this drone
 go.setWorldLocation(-100, -100, 0);
 }
 }
}

if (lm.isPlaying()) {
    // Run any un-clipped updates
    go.update(fps, lm.gravity);
        //...
```

尝试一下，尤其是高射速时，这真的很令人满意。

# 添加一些火焰瓦片

这些新的基于`GameObject`的对象将对 Bob 造成即死的效果。它们不会移动，但它们将被动画化。我们将看到，只需设置`GameObject`已有的属性，我们就可以实现这一点。

将这个功能添加到我们的游戏中非常简单，因为我们已经实现了所需的所有功能。我们已经有了定位和添加新瓦片的方法，检测并响应碰撞的方法，精灵图动画等等。让我们一步步进行，然后我们就可以将这些危险且致命的元素添加到我们的世界中。

我们可以将类的所有功能都放入其构造函数中。我们所要做的就是像配置`Grass`对象那样配置这个对象，此外，我们还要为其配置所有动画设置，就像我们对`Player`和`Guard`对象所做的那样。`fire.png`精灵图有三种动画帧，我们希望在一秒钟内播放它们。

![添加一些火焰瓦片](img/B04322_08_01.jpg)

创建一个新类，将其命名为`Fire`，并向其中添加以下代码：

```java
import android.content.Context;

public class Fire extends GameObject{

    Fire(Context context, float worldStartX, 
    float worldStartY, char type, int pixelsPerMetre) {

        final int ANIMATION_FPS = 3;
        final int ANIMATION_FRAME_COUNT = 3;
        final String BITMAP_NAME = "fire";

        final float HEIGHT = 1;
        final float WIDTH = 1;

        setHeight(HEIGHT); // 1 metre tall
        setWidth(WIDTH); // 1 metre wide

        setType(type);
        // Now for the player's other attributes
        // Our game engine will use these
        setMoves(false);
        setActive(true);
        setVisible(true);

        // Choose a Bitmap
        setBitmapName(BITMAP_NAME);
        // Set this object up to be animated
        setAnimFps(ANIMATION_FPS);
        setAnimFrameCount(ANIMATION_FRAME_COUNT);
        setBitmapName(BITMAP_NAME);
        setAnimated(context, pixelsPerMetre, true);

        // Where does the tile start
        // X and y locations from constructor parameters
        setWorldLocation(worldStartX, worldStartY, 0);
        setRectHitbox();
    }

 public void update(long fps, float gravity) {
 }
}
```

现在，当然，我们需要将下载包中`Chapter8/drawable`目录下的`fire.png`精灵图添加到项目的`drawable`文件夹中。

然后，我们按照为所有新的`GameObject`派生类所做的方式，以通常的三种方法将它们添加到我们的`LevelManager`类中。

在`getBitmap`方法中，添加突出显示的代码：

```java
case 'g':
    index = 7;
    break;

case 'f':
 index = 8;
 break;

default:
    index = 0;
    break;
```

在`getBitmapIndex`方法中：

```java
case 'g':
    index = 7;
    break;

case 'f':
 index = 8;
 break;

default:
    index = 0;
    break;
```

在`loadMapData()`方法中：

```java
case 'g':
     // Add a guard to the gameObjects
     gameObjects.add(new Guard(context, j, i, c, pixelsPerMetre));
     break;

 case 'f':
 // Add a fire tile the gameObjects
 gameObjects.add(new Fire
 (context, j, i, c, pixelsPerMetre));

 break;

```

最后，我们在碰撞检测的`switch`块中添加处理触碰这个可怕瓦片的后果。

```java
case 'g':
    //hit by guard
    sm.playSound("player_burn");
    ps.loseLife();
    location = new PointF(ps.loadLocation().x,
        ps.loadLocation().y);
    lm.player.setWorldLocationX(location.x);
    lm.player.setWorldLocationY(location.y);
    lm.player.setxVelocity(0);
    break;

case 'f':
 sm.playSound("player_burn");
 ps.loseLife();
 location = new PointF(ps.loadLocation().x,
 ps.loadLocation().y);
 lm.player.setWorldLocationX(location.x);
 lm.player.setWorldLocationY(location.y);
 lm.player.setxVelocity(0);
 break;

default:// Probably a regular tile
    if (hit == 1) {// Left or right
        lm.player.setxVelocity(0);
        lm.player.setPressingRight(false);
    }

    if (hit == 2) {// Feet
        lm.player.isFalling = false;
    }
    break;
```

不如在`LevelCave`中添加一些`f`瓦片，并实验玩家能够跳跃过哪些。这将帮助我们在本章后面设计一些具有挑战性的关卡。

![添加一些火焰瓦片](img/B04322_08_02.jpg)

我们不希望玩家一直走在草地上，所以让我们添加一些多样性。

# 眼前一亮

本章接下来的三个部分将纯粹关注外观。我们将添加一整套不同的瓦片图像和匹配的类，这样我们可以使用更多的艺术许可来使我们的关卡更有趣。这些瓦片之间的区别将纯粹是视觉上的，但使它们具有比这更多的功能性将相当简单。

例如，我们可以轻松检测与雪瓦片的碰撞，并让玩家在短暂停止后继续移动以模拟滑行，或者；混凝土瓦片可以让玩家移动得更快，因此改变我们设计大跳跃的方式等等。重点是，你不必仅仅复制粘贴这里呈现的类。

我们还将添加一些完全为了美观的道具：矿车、巨石、石钟乳石等。这些对象不会有碰撞检测。它们将允许关卡设计师使关卡在视觉上更有趣。

### 提示

要使这些美观元素更具功能性很简单。只需添加一个碰撞箱并在碰撞检测`switch`块中添加一个案例来处理后果。

可能，我们添加的视觉上最重要的改进将是滚动背景。我们将添加一些类，允许关卡设计师向关卡设计中添加多个不同的滚动背景。

### 提示

不妨将下载包中`Chapter8/drawable`文件夹的所有图像添加到项目的`drawable`文件夹中。这样，你将拥有所有图形，包括本节和接下来两节的图形都准备好了。

## 新的平台瓦片

现在，按照显示的文件名添加所有这些类。我移除了代码中的所有注释，因为它们在功能上都与`Grass`类相同。按照显示的名称创建以下每个类，并输入代码：

这是`Brick`类的代码：

```java
public class Brick extends GameObject {

    Brick(float worldStartX, float worldStartY, char type) {
        setTraversable();
        final float HEIGHT = 1;
        final float WIDTH = 1;
        setHeight(HEIGHT); 
        setWidth(WIDTH); 
        setType(type);
        setBitmapName("brick");
        setWorldLocation(worldStartX, worldStartY, 0);
        setRectHitbox();
    }

    public void update(long fps, float gravity) {
    }
}
```

这是`Coal`类的代码：

```java
public class Coal extends GameObject {

    Coal(float worldStartX, float worldStartY, char type) {
        setTraversable();
        final float HEIGHT = 1;
        final float WIDTH = 1;
        setHeight(HEIGHT); 
        setWidth(WIDTH);
        setType(type);
        setBitmapName("coal");
        setWorldLocation(worldStartX, worldStartY, 0);
        setRectHitbox();
    }

    public void update(long fps, float gravity) {
    }
}
```

这是`Concrete`类的代码：

```java
public class Concrete extends GameObject {

    Concrete(float worldStartX, float worldStartY, char type) {
        setTraversable();
        final float HEIGHT = 1;
        final float WIDTH = 1;
        setHeight(HEIGHT);
        setWidth(WIDTH);
        setType(type);
        setBitmapName("concrete");
        setWorldLocation(worldStartX, worldStartY, 0);
        setRectHitbox();
    }

    public void update(long fps, float gravity) {
    }
}
```

以下是`Scorched`类的代码：

```java
public class Scorched extends GameObject {

    Scorched(float worldStartX, float worldStartY, char type) {
        setTraversable();
        final float HEIGHT = 1;
        final float WIDTH = 1;
        setHeight(HEIGHT);
        setWidth(WIDTH);
        setType(type);
        setBitmapName("scorched");
        setWorldLocation(worldStartX, worldStartY, 0);
        setRectHitbox();
    }

    public void update(long fps, float gravity) {
    }
}
```

这是`Snow`类的代码：

```java
public class Snow extends GameObject {

    Snow(float worldStartX, float worldStartY, char type) {
        setTraversable();
        final float HEIGHT = 1;
        final float WIDTH = 1;
        setHeight(HEIGHT);
        setWidth(WIDTH);
        setType(type);
        setBitmapName("snow");
        setWorldLocation(worldStartX, worldStartY, 0);
        setRectHitbox();
    }

    public void update(long fps, float gravity) {
    }
}
```

这是`Stone`类的代码：

```java
public class Stone extends GameObject {

    Stone(float worldStartX, float worldStartY, char type) {
        setTraversable();
        final float HEIGHT = 1;
        final float WIDTH = 1;
        setHeight(HEIGHT);
        setWidth(WIDTH); 
        setType(type);
        setBitmapName("stone");
        setWorldLocation(worldStartX, worldStartY, 0);
        setRectHitbox();
    }

    public void update(long fps, float gravity) {
    }
}
```

现在，像我们习惯的那样，我们需要将它们全部添加到我们的`LevelManager`中，在通常的三个地方。

在`getBitmap()`方法中，我们像平常一样将它们添加进去。请注意，尽管这些值是任意的，但我们将为类型 2、3、4 等使用数字。这样在设计关卡时容易记住，我们所有的实际平台都是数字。实际的索引编号对我们来说不重要，只要它们与`getBitmapIndex`方法中的相同即可。此外，记住我们在`LevelData`类的注释中有一个类型列表，以便在设计关卡时方便参考。

```java
case 'f':
    index = 8;
    break;

case '2':
 index = 9;
 break;

case '3':
 index = 10;
 break;

case '4':
 index = 11;
 break;

case '5':
 index = 12;
 break;

case '6':
 index = 13;
 break;

case '7':
 index = 14;
 break;

default:
    index = 0;
    break;
```

在`getBitmapIndex()`中，我们做同样的事情：

```java
case 'f':
    index = 8;
    break;

case '2':
 index = 9;
 break;

case '3':
 index = 10;
 break;

case '4':
 index = 11;
 break;

case '5':
 index = 12;
 break;

case '6':
 index = 13;
 break;

case '7':
 index = 14;
 break;

default:
    index = 0;
    break;
```

在`loadMapData()`中，我们只需在我们的新`GameObjects`上调用`new()`，将它们添加到`gameObjects`列表中。

```java
case 'f':
    // Add a fire tile the gameObjects
    gameObjects.add(new Fire(context, j, i, c, pixelsPerMetre));
    break;

case '2':
 // Add a tile to the gameObjects
 gameObjects.add(new Snow(j, i, c));
 break;

case '3':
 // Add a tile to the gameObjects
 gameObjects.add(new Brick(j, i, c));
 break;

case '4':
 // Add a tile to the gameObjects
 gameObjects.add(new Coal(j, i, c));
 break;

case '5':
 // Add a tile to the gameObjects
 gameObjects.add(new Concrete(j, i, c));
 break;

case '6':
 // Add a tile to the gameObjects
 gameObjects.add(new Scorched(j, i, c));
 break;

case '7':
 // Add a tile to the gameObjects
 gameObjects.add(new Stone(j, i, c));
 break;

```

现在，大胆地为`LevelCave`类添加不同的地形：

![新的平台瓦片](img/B04322_08_03.jpg)

现在，我们来添加一些景观对象。

## 新的景观对象

在这里，我们会添加一些除了看起来漂亮之外什么都不做的对象。我们只需不添加碰撞箱，并将它们随机设置为 z 层-1 或 1，让游戏引擎知道这一点。然后玩家可以出现在它们前面或后面。

我们首先会添加所有类，然后像往常一样更新`LevelManager`的三个地方。按照以下方式创建每个新类：

这是`Boulders`类：

```java
public class Boulders extends GameObject {

    Boulders(float worldStartX, float worldStartY, char type) {

        final float HEIGHT = 1;
        final float WIDTH = 3;

        setHeight(HEIGHT); // 1 metre tall
        setWidth(WIDTH); // 1 metre wide

        setType(type);

        // Choose a Bitmap
        setBitmapName("boulder");
        setActive(false);//don't check for collisions etc

        // Randomly set the tree either just in front or just 
        //behind the player -1 or 1
        Random rand = new Random();
        if(rand.nextInt(2)==0) {
            setWorldLocation(worldStartX, worldStartY, -1);
        }else{
            setWorldLocation(worldStartX, worldStartY, 1);//
        }
        //No hitbox!!

    }

    public void update(long fps, float gravity) {
    }
}
```

从现在开始，我删除了所有注释以节省墨水。该类的功能与`Boulders`中的相同，只是属性有些许不同。

这是`Cart`类：

```java
public class Cart extends GameObject {

  Cart(float worldStartX, float worldStartY, char type) {

        final float HEIGHT = 2;
        final float WIDTH = 3;
        setWidth(WIDTH);
        setHeight(HEIGHT);
        setType(type);
        setBitmapName("cart");
        setActive(false);
        Random rand = new Random();
        if(rand.nextInt(2)==0) {
          setWorldLocation(worldStartX, worldStartY, -1);
        }else{
          setWorldLocation(worldStartX, worldStartY, 1);
        }
     }

  public void update(long fps, float gravity) {
     }
}
```

这是`Lampost`类的代码：

```java
public class Lampost extends GameObject {

  Lampost(float worldStartX, float worldStartY, char type) {

        final float HEIGHT = 3;
        final float WIDTH = 1;
        setHeight(HEIGHT);
        setWidth(WIDTH); 
        setType(type);
        setBitmapName("lampost");
        setActive(false);
        Random rand = new Random();
        if(rand.nextInt(2)==0) {
          setWorldLocation(worldStartX, worldStartY, -1);
        }else{
          setWorldLocation(worldStartX, worldStartY, 1);
        }
  }

    public void update(long fps, float gravity) {
   }
}
```

这是`Stalagmite`类：

```java
import java.util.Random;

public class Stalagmite extends GameObject {

  Stalagmite(float worldStartX, float worldStartY, char type) {

        final float HEIGHT = 3;
        final float WIDTH = 2;
        setHeight(HEIGHT);
        setWidth(WIDTH);
        setType(type);
        setBitmapName("stalacmite");
        setActive(false);
        Random rand = new Random();
        if(rand.nextInt(2)==0) {
         setWorldLocation(worldStartX, worldStartY, -1);
        }else{
         setWorldLocation(worldStartX, worldStartY, 1);
        }
    }

    public void update(long fps, float gravity) {
    }
}
```

这是`Stalactite`类：

```java
import java.util.Random;

public class Stalactite extends GameObject {

  Stalactite(float worldStartX, float worldStartY, char type) {

        final float HEIGHT = 3;
        final float WIDTH = 2;
        setHeight(HEIGHT);
        setWidth(WIDTH);
        setType(type);
        setBitmapName("stalactite");
        setActive(false);
        Random rand = new Random();
        if(rand.nextInt(2)==0) {
          setWorldLocation(worldStartX, worldStartY, -1);
        }else{
          setWorldLocation(worldStartX, worldStartY, 1);
        }
  }

     public void update(long fps, float gravity) {
     }
}
```

这是`Tree`类：

```java
import java.util.Random;

public class Tree extends GameObject {

  Tree(float worldStartX, float worldStartY, char type) {

       final float HEIGHT = 4;
       final float WIDTH = 2;
       setWidth(WIDTH);
        setHeight(HEIGHT);
        setType(type);
        setBitmapName("tree1");
        setActive(false);
        Random rand = new Random();
        if(rand.nextInt(2)==0) {
          setWorldLocation(worldStartX, worldStartY, -1);
        }else{
          setWorldLocation(worldStartX, worldStartY, 1);
        }
     }

     public void update(long fps, float gravity) {
     }
}
```

这是`Tree2`类：

```java
import java.util.Random;

public class Tree2 extends GameObject {

  Tree2(float worldStartX, float worldStartY, char type) {

        final float HEIGHT = 4;
        final float WIDTH = 2;
        setWidth(WIDTH);
        setHeight(HEIGHT);
        setType(type);
        setBitmapName("tree2");
        setActive(false);
        Random rand = new Random();
        if(rand.nextInt(2)==0) {
          setWorldLocation(worldStartX, worldStartY, -1);
        }else{
          setWorldLocation(worldStartX, worldStartY, 1);
        }
  }

     public void update(long fps, float gravity) {
     }
}
```

这就是所有新的景观对象类。现在，我们可以在`LevelManager`类中用七种新类型更新`getBitmap`方法。

```java
case '7':
    index = 14;
    break;

case 'w':
 index = 15;
 break;

case 'x':
 index = 16;
 break;

case 'l':
 index = 17;
 break;

case 'r':
 index = 18;
 break;

case 's':
 index = 19;
 break;

case 'm':
 index = 20;
 break;

case 'z':
 index = 21;
 break;

default:
    index = 0;
    break;
```

以同样的方式更新`getBitmapIndex`方法：

```java
case '7':
    index = 14;
    break;

case 'w':
 index = 15;
 break;

case 'x':
 index = 16;
 break;

case 'l':
 index = 17;
 break;

case 'r':
 index = 18;
 break;

case 's':
 index = 19;
 break;

case 'm':
 index = 20;
 break;

case 'z':
 index = 21;
 break;

default:
    index = 0;
    break;
```

最后，确保我们的新景观物品被添加到`gameObjects`数组列表中：

```java
case '7':
    // Add a tile to the gameObjects
    gameObjects.add(new Stone(j, i, c));
    break;

case 'w':
 // Add a tree to the gameObjects
 gameObjects.add(new Tree(j, i, c));
 break;

case 'x':
 // Add a tree2 to the gameObjects
 gameObjects.add(new Tree2(j, i, c));
 break;

case 'l':
 // Add a tree to the gameObjects
 gameObjects.add(new Lampost(j, i, c));
 break;

case 'r':
 // Add a stalactite to the gameObjects
 gameObjects.add(new Stalactite(j, i, c));
 break;

case 's':
 // Add a stalagmite to the gameObjects
 gameObjects.add(new Stalagmite(j, i, c));
 break;

case 'm':
 // Add a cart to the gameObjects
 gameObjects.add(new Cart(j, i, c));
 break;

case 'z':
 // Add a boulders to the gameObjects
 gameObjects.add(new Boulders(j, i, c));
 break;

```

现在，我们可以设计带有景观的关卡。注意当对象在层零与层一上绘制时外观上的细微差别，以及玩家角色如何穿过它们前面或后面：

![新的景观对象](img/B04322_08_04.jpg)

### 提示

当然，如果你想要碰撞到路灯、被石笋刺穿，或者跳到矿车顶上，只需给它们一个碰撞箱即可。

我们还有一种美化游戏世界的方法。

## 滚动的视差背景

视差背景是滚动的背景，我们根据它们距离的远近来减慢滚动速度。因此，如果玩家脚边有草地，我们会快速滚动它。然而，如果远处有山脉，我们会慢慢滚动它。这种效果可以给玩家带来运动的感觉。

为了实现这一功能，我们首先将添加一个数据结构来表示背景的参数。我们将这个类称为`BackgroundData`，然后实现一个`Background`类，它具有控制滚动所需的功能，然后我们将会看到如何在我们的关卡设计中定位和定义背景。最后，我们将编写一个`drawBackground`方法，我们将会从常规的`draw`方法中调用它。

确保你已经将从下载包的`Chapter8/drawable`文件夹中的所有图像添加到你的项目的`drawable`文件夹中。

首先，让我们构建一个简单的类来保存定义我们背景的数据结构。正如在下一个代码块中我们可以看到的，我们有很多参数和成员变量。我们需要知道哪个位图将代表背景，在*z*轴上哪个层面绘制它（前面为 1，后面为-1），在*y*轴上它在全球的哪个位置开始和结束，背景滚动的速度有多快，以及背景的高度是多少。

`isParallax`布尔值旨在提供一种让背景静止的选项，但我们不会实现这个功能。当你看到背景类的代码时，你会发现如果你想要，添加这个功能是非常简单的。

创建一个新类，将其命名为`BackgroundData`，然后用以下代码实现它：

```java
public class BackgroundData {
  String bitmapName;
     boolean isParallax;
     //layer 0 is the map
     int layer;
     float startY;
     float endY;
     float speed;
     int height;
     int width;

     BackgroundData(String bitmap, boolean isParallax, 
     int layer, float startY, float endY, 
     float speed, int height){

      this.bitmapName = bitmap;
      this.isParallax = isParallax;
      this.layer = layer;
      this.startY = startY;
      this.endY = endY;
      this.speed = speed;
      this.height = height;
  }
}
```

现在，我们在`LevelData`类中添加了一个我们新类型的`ArrayList`：

```java
ArrayList<String> tiles;
ArrayList<BackgroundData> backgroundDataList;

// This class will evolve along with the project
```

接下来，让我们创建`Background`类本身。创建一个新类，并将其命名为`Background`。首先，我们设置一组变量来保存背景图像以及它的反转副本。我们将通过将图像背靠背交替使用常规图像和反转图像，使背景看起来*无限*。我们将在代码中进一步了解如何实现这一点。

我们还有用于存储图像的宽度和高度的像素变量。`reversedFirst`布尔值将决定当前在屏幕左侧（首先）绘制哪个图像副本，并将在玩家移动和图像滚动时改变。`xClip`变量将保存我们将在屏幕左侧边缘开始绘制的图像的*x*轴的确切像素。

`y`、`endY`、`z`和`speed`成员变量用于保存作为参数传递的相关值：

```java
import android.content.Context;
import android.graphics.Bitmap;
import android.graphics.BitmapFactory;
import android.graphics.Matrix;

public class Background {

     Bitmap bitmap;
     Bitmap bitmapReversed;

     int width;
     int height;

     boolean reversedFirst;
     int xClip;// controls where we clip the bitmaps each frame
     float y;
     float endY;
     int z;

     float speed;
     boolean isParallax;//Not currently used
```

在构造函数中，我们从作为参数传递的图形文件名创建一个 Android 资源 ID。然后，通过调用`BitmapFactory.decodeResource()`创建实际的位图。我们将`reversedFirst`设置为`false`，因此将从屏幕左侧开始使用图像的正常（非反转）副本。我们初始化成员变量，并通过调用`Bitmap.createScaledBitmap()`并传入位图、屏幕宽度和背景在游戏世界中的高度乘以`pixelsPerMetre`来缩放我们刚刚创建的位图，使位图恰好适合当前设备屏幕的尺寸。

### 提示

请注意，我们必须为背景设计选择适当的高度，否则它们会出现拉伸。

构造函数中最后要做的就是在调用`createScaledBitmap`方法时创建一个`Matrix`对象，并连同位图一起传递，这样我们现在在`bitmapReversed Bitmap`对象中存储了一个背景图像的反转副本。

```java
  Background(Context context, int yPixelsPerMetre, 
    int screenWidth, BackgroundData data){

      int resID =   context.getResources().getIdentifier
      (data.bitmapName, "drawable", 
      context.getPackageName());

          bitmap = BitmapFactory.decodeResource
          (context.getResources(), resID);

          // Which version of background (reversed or regular) is // currently drawn first (on left)
          reversedFirst = false;

          //Initialize animation variables.
          xClip = 0;  //always start at zero
          y = data.startY;
          endY = data.endY;
          z = data.layer;
          isParallax = data.isParallax;
          speed = data.speed; //Scrolling background speed

          //Scale background to fit the screen.
          bitmap = Bitmap.createScaledBitmap(bitmap, screenWidth,
                data.height * yPixelsPerMetre
                , true); 

          width = bitmap.getWidth();
          height = bitmap.getHeight();

          // Create a mirror image of the background
          Matrix matrix = new Matrix();  
          matrix.setScale(-1, 1); //Horizontal mirror effect.
          bitmapReversed = Bitmap.createBitmap(
          bitmap, 0, 0, width, height, matrix, true);

    }
}
```

现在，我们在关卡设计中添加两个背景。我们填写已经讨论过的所需参数。请注意，第 1 层的“草地”背景滚动速度比-1 层的“天际线”背景快得多。这将产生所需的视差效果。在`LevelCave`构造函数的末尾添加以下代码：

```java
backgroundDataList = new ArrayList<BackgroundData>();
// note that speeds less than 2 cause problems
this.backgroundDataList.add(
  new BackgroundData("skyline", true, -1, 3, 18, 10, 15 ));

this.backgroundDataList.add(
  new BackgroundData("grass", true, 1, 20, 24, 24, 4 ));
```

### 注意

大多数洞穴确实没有草地和天际线，这只是一个演示，让代码工作起来。我们将在本章稍后重新设计`LevelCave`，并设计一些更合适的关卡。

现在，我们通过声明一个`Arraylist`对象作为`LevelManager`类的成员，用我们的`LevelManager`类加载它们。

```java
LevelData levelData;
ArrayList<GameObject> gameObjects;
ArrayList<Background> backgrounds;

```

然后，在`LevelManager`中添加一个新方法来加载背景数据：

```java
private void loadBackgrounds(Context context, 
  int pixelsPerMetre, int screenWidth) {

  backgrounds = new ArrayList<Background>();
     //load the background data into the Background objects and
     // place them in our GameObject arraylist
     for (BackgroundData bgData : levelData.backgroundDataList) {
            backgrounds.add(new Background(context,       
            pixelsPerMetre, screenWidth, bgData));
     }
}
```

我们在`LevelManager`构造函数中调用这个新方法：

```java
// Load all the GameObjects and Bitmaps
loadMapData(context, pixelsPerMetre, px, py);
loadBackgrounds(context, pixelsPerMetre, screenWidth);

```

并且，不是最后一次，我们将升级我们的`Viewport`类，让`PlatformView`方法能够获取它们需要的信息，以绘制视差背景。

```java
public int getPixelsPerMetreY(){
  return  pixelsPerMetreY;
}

public int getyCentre(){
  return screenCentreY;
}

public float getViewportWorldCentreY(){
  return currentViewportWorldCentre.y;
}
```

然后，我们将在`PlatformView`类中添加一个实际执行绘图的方法。接下来，我们会在`onDraw()`中的恰当位置调用这个方法。请注意，我们正在使用刚刚添加到`Viewport`类中的新方法。

首先，我们定义四个`Rect`对象，用来保存`bitmap`和`reversedBitmap`的起始和结束点。

按照所示实现`drawBackground`方法的第一部分：

```java
private void drawBackground(int start, int stop) {

     Rect fromRect1 = new Rect();
     Rect toRect1 = new Rect();
     Rect fromRect2 = new Rect();
     Rect toRect2 = new Rect();
```

现在，我们只需遍历所有背景，使用`start`和`stop`参数来确定哪些背景具有我们当前感兴趣的*z*层。

```java
     for (Background bg : lm.backgrounds) {
     if (bg.z < start && bg.z > stop) {

```

接下来，我们将背景的世界坐标发送到`Viewport`类进行裁剪。如果没有裁剪（并且应该绘制），我们将使用之前添加到我们的`Viewport`类中的新方法，获取*y*轴上的起始像素坐标和结束像素坐标。请注意，我们将结果转换为`int`变量，以便绘制到屏幕上。

```java
          // Is this layer in the viewport?
            // Clip anything off-screen
            if (!vp.clipObjects(-1, bg.y, 1000, bg.height)) {
                float floatstartY = ((vp.getyCentre() -                     
                    ((vp.getViewportWorldCentreY() - bg.y) * 
                    vp.getPixelsPerMetreY())));

                int startY = (int) floatstartY;

                float floatendY = ((vp.getyCentre() -           
                    ((vp.getViewportWorldCentreY() - bg.endY) *                                 
                    vp.getPixelsPerMetreY())));

                int endY = (int) floatendY;
```

下面的代码块是真正行动发生的地方。我们用两个`Bitmap`对象的起始和结束坐标初始化四个`Rect`对象。请注意，计算出的点（或像素）由`xClip`确定，最初为零。因此，首先我们会看到`background`（如果它没有被剪辑）拉伸到屏幕的宽度。很快，我们会看到根据 Bob 的速度修改`xClip`，并展示每个位图的不同区域：

```java
        // Define what portion of bitmaps to capture 
        // and what coordinates to draw them at
        fromRect1 = new Rect(0, 0, bg.width - bg.xClip,     
          bg.height);

        toRect1 = new Rect(bg.xClip, startY, bg.width, endY);
             fromRect2 = new Rect(bg.width - bg.xClip, 0, bg.width, bg.height);

        toRect2 = new Rect(0, startY, bg.xClip, endY);
        }// End if (!vp.clipObjects...
```

现在，我们确定当前首先绘制的是哪种背景（正常或反向），然后先绘制该背景，接着绘制另一种。

```java
          //draw backgrounds
            if (!bg.reversedFirst) {

                canvas.drawBitmap(bg.bitmap,
                    fromRect1, toRect1, paint);
                canvas.drawBitmap(bg.bitmapReversed, 
                    fromRect2, toRect2, paint);

            } else {
                canvas.drawBitmap(bg.bitmap, 
                    fromRect2, toRect2, paint);

                canvas.drawBitmap(bg.bitmapReversed, 
                    fromRect1, toRect1, paint);
            }
```

我们可以根据 Bob 的速度和方向滚动，`lv.player.getxVelocity()`，如果`xClip`已达到当前第一个背景的末端，`if (bg.xClip >= bg.width)`，只需将`xClip`设为零，并改变我们首先展示的位图。

```java
          // Calculate the next value for the background's
            // clipping position by modifying xClip
            // and switching which background is drawn first,
            // if necessary.
            bg.xClip -= lm.player.getxVelocity() / (20 / bg.speed);
            if (bg.xClip >= bg.width) {
                bg.xClip = 0;
                bg.reversedFirst = !bg.reversedFirst;
            } 
            else if (bg.xClip <= 0) {
                bg.xClip = bg.width;
                bg.reversedFirst = !bg.reversedFirst;

            }
        }
    }
}
```

然后，在*z*层小于零的背景的游戏对象之前，我们添加对`drawBackground()`的调用。

```java
// Rub out the last frame with arbitrary color
paint.setColor(Color.argb(255, 0, 0, 255));
canvas.drawColor(Color.argb(255, 0, 0, 255));

// Draw parallax backgrounds from -1 to -3
drawBackground(0, -3);

// Draw all the GameObjects
Rect toScreen2d = new Rect();
```

在绘制子弹之后，但在那些*z*顺序大于零的背景的调试文本之前。

```java
// Draw parallax backgrounds from layer 1 to 3
drawBackground(4, 0);

// Text for debugging
```

现在，我们可以真正开始发挥创意设计关卡。

![滚动视差背景](img/B04322_08_05.jpg)

很快，我们将制作一些真正可玩的关卡，使用我们在过去四章中实现的所有功能。在我们这样做之前，让我们在`Viewport`类中找点乐趣。

对于玩家来说，在关卡中扫描并规划路线将非常有用。同样，在设计关卡时，放大并围绕关卡查看某个特定部分的外观，而无需让玩家角色到达该部分以便在屏幕上看到它，也会很有帮助。所以，让我们将暂停屏幕变成一个可移动视口。

## 带有可移动视口的暂停菜单

这样做既好又快。我们只需向`Viewport`类添加一堆新方法来改变焦点中心。然后，我们将在`InputController`中调用它们。

如果你记得我们在第六章实现`InputController`类时，*平台游戏 – Bob, Beeps 和 Bumps*，我们将所有控制逻辑封装在一个`if(playing)`测试中。我们还在`else`子句中实现了暂停按钮。我们将要做的就是将左、右、跳跃和射击按钮分别用作左、右、上和下来移动视口。

首先，向`Viewport`类添加以下方法：

```java
public void moveViewportRight(int maxWidth){
  if(currentViewportWorldCentre.x < maxWidth -       
    (metresToShowX/2)+3) {

     currentViewportWorldCentre.x += 1;
  }
}

public void moveViewportLeft(){
  if(currentViewportWorldCentre.x > (metresToShowX/2)-3){
    currentViewportWorldCentre.x -= 1;
     }
}

public void moveViewportUp(){
  if(currentViewportWorldCentre.y > (metresToShowY /2)-3) {
        currentViewportWorldCentre.y -= 1;
   }
}

public void moveViewportDown(int maxHeight){
  if(currentViewportWorldCentre.y < 
    maxHeight - (metresToShowY / 2)+3) {

    currentViewportWorldCentre.y += 1;
  }
}
```

现在，将以下调用添加到我们在`InputController`类中刚刚讨论的`if`条件的`else`子句中。

```java
//Move the viewport around to explore the map
switch (motionEvent.getAction() & MotionEvent.ACTION_MASK) {
  case MotionEvent.ACTION_DOWN:
 if (right.contains(x, y)) {
 vp.moveViewportRight(l.mapWidth);
 } else if (left.contains(x, y)) {
 vp.moveViewportLeft();
 } else if (jump.contains(x, y)) {
 vp.moveViewportUp();
 } else if (shoot.contains(x, y)) {
 vp.moveViewportDown(l.mapHeight);
 } else if (pause.contains(x, y)) {
 l.switchPlayingStatus();
 }
      break;
}
```

在暂停屏幕上，玩家可以四处查看并规划他们在更复杂关卡中的路线。他们可能需要这么做。

## 关卡和游戏规则

我们已经实现了许多功能，但我们仍然没有一个方法将这些功能整合成一个可玩的游戏。我们需要能够在关卡之间移动，并且在移动时保持玩家状态。

### 在关卡之间移动

因为我们将要设计四个关卡，我们希望玩家能够在它们之间移动。首先，让我们在`LevelManager`构造函数的开始部分的`switch`语句中添加代码，包括我们即将构建的所有四个关卡：

```java
switch (level) {
  case "LevelCave":
     levelData = new LevelCave();
     break;

// We can add extra levels here
case "LevelCity": 
 levelData = new LevelCity(); 
 break; 

case "LevelForest": 
 levelData = new LevelForest(); 
 break;

case "LevelMountain": 
 levelData = new LevelMountain(); 
 break;
}
```

如我们所知，我们通过从`PlatformView`构造函数中调用`loadLevel()`来开始游戏。参数包括关卡名称和玩家生成的坐标。如果你正在设计自己的关卡，那么你需要决定从哪个关卡和坐标开始。如果你将跟随我提供的关卡，请在`PlatformView`的构造函数中将`loadLevel()`的调用设置如下：

```java
loadLevel("LevelCave", 1, 16);
```

在`if(lm.isPlaying())`块中，在`update`方法中，我们每一帧设置视口以玩家为中心；添加以下代码以检测（并残忍地消灭）玩家如果他掉出地图，以及当他的生命值耗尽时，使游戏重新开始，拥有三条生命，零金钱，没有升级：

```java
if (lm.isPlaying()) {
    // Reset the players location as 
    // the world centre of the viewport
    //if game is playing
    vp.setWorldCentre(lm.gameObjects.get(lm.playerIndex)
        .getWorldLocation().x,
        lm.gameObjects.get(lm.playerIndex)
        .getWorldLocation().y);

 //Has player fallen out of the map?
 if (lm.player.getWorldLocation().x < 0 ||
 lm.player.getWorldLocation().x > lm.mapWidth ||
 lm.player.getWorldLocation().y > lm.mapHeight) {

 sm.playSound("player_burn");
 ps.loseLife();
 PointF location = new PointF(ps.loadLocation().x,
 ps.loadLocation().y);

 lm.player.setWorldLocationX(location.x);
 lm.player.setWorldLocationY(location.y);
 lm.player.setxVelocity(0);
 }

 // Check if game is over
 if (ps.getLives() == 0) {
 ps = new PlayerState();
 loadLevel("LevelCave", 1, 16);
 }
}
```

现在，我们可以创建一个特殊的`GameObject`类，当玩家接触这个类时，会将玩家传送到一个预定的关卡和位置。然后我们可以策略性地将这些对象添加到我们的关卡设计中，它们将作为我们关卡之间的链接。创建一个名为`Teleport`的新类。如果你还没有这样做，请将`Chapter8/drawable`文件夹中的`door.png`文件添加到项目的`drawable`文件夹中。

这就是我们的`Teleport`对象在游戏中的样子：

![在关卡之间移动](img/B04322_08_06.jpg)

让我们创建一个简单的类来保存每个`Teleport`对象所需的数据。创建一个名为`Location`的新类，如下所示：

```java
public class Location {
     String level;
     float x;
     float y;

     Location(String level, float x, float y){
        this.level = level;
        this.x = x;
        this.y = y;
     }
}
```

实际的`Teleport`类看起来像任何其他的`GameObject`类，但请注意它还包含一个`Location`成员变量。我们将看到关卡设计将如何保存`Teleport`的目的地，`LevelManager`类将初始化它，然后当玩家与它碰撞时，我们可以加载新的位置，将玩家送往他的目的地。

```java
public class Teleport extends GameObject {

    Location target;

    Teleport(float worldStartX, float worldStartY, 
        char type, Location target) {

        final float HEIGHT = 2;
        final float WIDTH = 2;
        setHeight(HEIGHT); // 2 metres tall
        setWidth(WIDTH); // 1 metre wide
        setType(type);
        setBitmapName("door");

        this.target = new Location(target.level, 
            target.x, target.y);

        // Where does the tile start
        // X and y locations from constructor parameters
        setWorldLocation(worldStartX, worldStartY, 0);

        setRectHitbox();
    }

    public Location getTarget(){
        return target;
    }

    public void update(long fps, float gravity){
    }
}
```

为了让我们的`Teleport`类以让关卡设计师决定它将确切执行什么的方式工作，我们需要像这样向我们的`LevelData`类中添加内容：

```java
ArrayList<String> tiles;
ArrayList<BackgroundData> backgroundDataList;
ArrayList<Location> locations;

// This class will evolve along with the project
```

然后，我们需要在想要设置传送门/门的关卡设计中的相应位置添加一个`t`，并在关卡类的构造函数中添加如下代码行。

请注意，你可以在地图中设置任意数量的`Teleport`对象，只要它们在代码中定义的顺序与设计中出现的顺序相匹配。当我们稍后查看实际的关卡设计时，我们会确切地看到这是如何工作的，但代码将如下所示：

```java
// Declare the values for the teleports in order of appearance
locations = new ArrayList<Location>();
this.locations.add(new Location("LevelCity", 118f, 18f));
```

与往常一样，我们需要更新`LevelManager`类以加载和定位我们的传送点。以下是`getBitmap()`的新代码：

```java
case 'z':
  index = 21;
  break;

case 't':
 index = 22;
 break;

default:
  index = 0;
  break;
```

`getBitmapIndex()`的新代码：

```java
case 'z':
  index = 21;
     break;

case 't':
 index = 22;
 break;

default:
  index = 0;
  break;
```

在加载阶段，我们还需要跟踪我们的`Teleport`对象，以防有多个。所以，在`loadMapData`方法中添加一个新的局部变量，如下所示：

```java
//Keep track of where we load our game objects
int currentIndex = -1;
int teleportIndex = -1;
// how wide and high is the map? Viewport needs to know
```

对于`LevelManager`类，我们最终需要初始化所有从关卡设计中获取的传送数据，将其存储在对象中，并添加到我们的`gameObject ArrayList`中。

```java
case 'z':
    // Add a boulders to the gameObjects
    gameObjects.add(new Boulders(j, i, c));
    break;

 case 't':
 // Add a teleport to the gameObjects
 teleportIndex++;
 gameObjects.add(new Teleport(j, i, c,
 levelData.locations.get(teleportIndex)));

 break;

```

我们已经非常接近能够到处传送了。我们需要检测与传送点的碰撞，然后在玩家所需的位置加载新关卡。这段代码将放在`PlatformView`类中的碰撞检测开关块里，如下所示：

```java
case 'f':
    sm.playSound("player_burn");
    ps.loseLife();
    location = new PointF(ps.loadLocation().x,
      ps.loadLocation().y); 
    lm.player.setWorldLocationX(location.x);
    lm.player.setWorldLocationY(location.y);
    lm.player.setxVelocity(0);
    break;

case 't':
 Teleport teleport = (Teleport) go;
 Location t = teleport.getTarget();
 loadLevel(t.level, t.x, t.y);
 sm.playSound("teleport");
 break;

default:// Probably a regular tile
    if (hit == 1) {// Left or right
        lm.player.setxVelocity(0);
        lm.player.setPressingRight(false);
    }
    if (hit == 2) {// Feet
        lm.player.isFalling = false;
    }
    break;
```

当加载新关卡时，`Player`、`MachineGun`和`Bullet`对象都将从头开始创建。因此，我们需要在`loadLevel`方法中添加一行代码，将当前的机枪射速从`PlayerState`类重新加载到`MachineGun`类中。添加高亮显示的代码：

```java
ps.saveLocation(location);

// Reload the players current fire rate from the player state
lm.player.bfg.setFireRate(ps.getFireRate());

```

现在，我们可以真正开始设计关卡了。

## 关卡设计

你可以从`Chapter8/java`文件夹中复制并粘贴四个类到你的项目中开始游戏，或者你可以从头开始设计自己的关卡。这些关卡相当大，复杂且难以通关。由于篇幅限制，无法在书籍或电子书中以有意义的方式呈现关卡设计，因此你需要打开`LevelCave`、`LevelCity`、`LevelForest`和`LevelMountain`设计文件，以查看四个关卡的详细信息。

然而，以下内容将简要讨论四个设计中的关卡、图片和一些截图，但不会包含实际的代码。

### 注意

请注意，以下截图展示了本章最后将要介绍的新 HUD。

### 洞穴

洞穴关卡是整个游戏的开始。它不仅包含一些令人稍微感到沮丧的跳跃，还有大量的火焰，一旦跌落可能致命。

![洞穴](img/B04322_08_07.jpg)

由于玩家开始时只有一把微弱的机枪，因此关卡中只有少数无人机。但有两个别扭的守卫需要翻越。

![洞穴](img/B04322_08_08.jpg)

### 城市

城市中拥有巨大的奖励，尤其是在左下角收集硬币和左上角升级机枪。

![城市](img/B04322_08_09.jpg)

然而，如果玩家想要收集所有散落的硬币而不选择放弃它们，底层有一个跳跃非常别扭的守卫。必须从左侧几乎垂直上升，这很可能会让玩家感到沮丧。如果玩家选择不去升级机枪，他可能会在与下一层门口外的双守卫战斗中遇到困难。

![城市](img/B04322_08_10.jpg)

### 森林

森林可能是所有关卡中最困难的一个，有一段残酷的长距离跳跃，非常容易跳过或未跳够。

![森林](img/B04322_08_11.jpg)

当 Bob 的像素悬挂在平台边缘时，超过一打无人机正等着猛扑向他。

![森林](img/B04322_08_12.jpg)

### 山脉

清新的山间空气意味着 Bob 几乎要成功了。四周没有守卫或无人机的踪影。

![山脉](img/B04322_08_13.jpg)

然而，看看那条蜿蜒的跳跃路径，如果 Bob 放错了一个像素的位置，大部分路径都会让他直接掉回底部。

![山脉](img/B04322_08_14.jpg)

### 提示

如果你想要在不完成前面的艰难关卡的情况下尝试每个关卡，当然，你可以直接从你选择的关卡和位置开始。为此，只需将`PlatformView`构造函数中的`loadLevel()`调用更改为以下之一：

```java
loadLevel("LevelMountain", 118, 17);
loadLevel("LevelForest", 1, 17);
loadLevel("LevelCity", 118, 18);
loadLevel("LevelCave", 1, 16);
```

### HUD

画龙点睛之笔是添加一个 HUD。`PlatformView`的`draw`方法中的这段代码使用了现有游戏对象中的一些图像。

在最后一次调用`drawBackground()`之后，并在绘制调试文本之前添加代码：

```java
// Draw the HUD
// This code needs bitmaps: extra life, upgrade and coin
// Therefore there must be at least one of each in the level

int topSpace = vp.getPixelsPerMetreY() / 4;
int iconSize = vp.getPixelsPerMetreX();
int padding = vp.getPixelsPerMetreX() / 5;
int centring = vp.getPixelsPerMetreY() / 6;
paint.setTextSize(vp.getPixelsPerMetreY()/2);
paint.setTextAlign(Paint.Align.CENTER);

paint.setColor(Color.argb(100, 0, 0, 0));
canvas.drawRect(0,0,iconSize * 7.0f, topSpace*2 + iconSize,paint);
paint.setColor(Color.argb(255, 255, 255, 0));

canvas.drawBitmap(lm.getBitmap('e'), 0, topSpace, paint);
canvas.drawText("" + ps.getLives(), (iconSize * 1) + padding, 
  (iconSize) - centring, paint);

canvas.drawBitmap(lm.getBitmap('c'), (iconSize * 2.5f) + padding, 
  topSpace, paint);

canvas.drawText("" + ps.getCredits(), (iconSize * 3.5f) + padding * 2, (iconSize) - centring, paint);

canvas.drawBitmap(lm.getBitmap('u'), (iconSize * 5.0f) + padding, 
  topSpace, paint);

canvas.drawText("" + ps.getFireRate(), (iconSize * 6.0f) + padding * 2, (iconSize) - centring, paint);
```

我想我们完成了！

# 总结

我们完成了这个平台游戏，因为篇幅有限。为什么不尝试实施以下一些或全部改进和功能呢？

修改`Player`类中的代码，使 Bob 逐渐加速和减速，而不是一直以全速运行。只需在玩家按住左右方向的每个帧增加速度，在他们不按的每个帧减少速度。

完成这些后，将前面的代码添加到`update`方法中的碰撞检测`switch`块中，以使玩家在雪地上打滑，在混凝土上加速，并为每种瓦片类型提供不同的行走/着陆声效。

在 Bob 身上画一把枪，并调整`Bullet`对象生成的高度，使其看起来是从他的机枪枪管中射出的。

让一些对象可以被推动。在`GameObject`中添加一个`isPushable`成员，并让碰撞检测简单地使对象后退一点。也许，Bob 可以把矿车推入火中，以跳过特别宽的火坑。请注意，推动那些掉到另一个层次的对象将比推动保持在相同*y*坐标的对象复杂得多。

可破坏的瓦片听起来很有趣。给它们一个力量变量，当被子弹击中时递减，当达到零时从`gameObjects`中移除。

移动平台是优秀平台游戏的重要组成部分。只需向瓦片对象添加航点，并在`update`方法中添加移动代码。挑战将是如何分配航点。你可以让它们都向左或向右，或者向上或向下移动固定的空间数量，或者像我们编写`Guard`对象那样，使用某种`setTileWaypoint`方法。

通过保存玩家收集到的硬币总数，记住哪些关卡已被解锁，并在菜单屏幕上提供重新玩任何已解锁关卡的选项，使游戏更具持久性。

使用传送点作为路标，让游戏变得更容易。调整视口缩放以适应不同屏幕尺寸。当前的缩放对于一些小手机来说可能有点太低了。

加入计时跑以获得高分、排行榜和成就，并增加更多关卡。

在下一章中，我们将看到一个更小的项目，但仍然很有趣，因为我们将使用 OpenGL ES 进行超快速、流畅的绘制。
