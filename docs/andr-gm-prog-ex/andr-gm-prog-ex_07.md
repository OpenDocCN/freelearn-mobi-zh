# 第七章。平台游戏 - 枪支、生命、金钱和敌人

在本章中，我们将做很多事情。首先，我们将构建一个可变射速的机枪，让它射击子弹。然后，我们将引入拾取物或收藏品。这些给玩家在尝试逃到下一个关卡时提供了搜寻的目标。

然后，就在 Bob 开始认为他的生活是充满草丛和收藏品的幸福生活时，我们将为他构建两个对手，让他智取或消灭。一个追踪无人机和一个巡逻的守卫。我们可以轻松地将所有这些事物添加到我们的关卡设计中。

# 准备，瞄准，开火。

现在，我们可以给我们的英雄一把枪，稍后，我们可以给他敌人射击。我们将创建一个`MachineGun`类来完成所有工作，以及一个`Bullet`类来表示它发射的炮弹。`Player`类将控制`MachineGun`类，而`MachineGun`类将控制和跟踪它发射的所有`Bullet`对象。

创建一个新的 Java 类，将其命名为`Bullet`。子弹并不复杂。我们的子弹需要有一个*x*和*y*的位置，一个水平速度和一个方向，以帮助计算速度。

这意味着以下简单的类、构造函数以及一堆的 getter 和 setter：

```java
public class Bullet  {

    private float x;
    private float y;
    private float xVelocity;
    private int direction;

    Bullet(float x, float y, int speed, int direction){
        this.direction = direction;
        this.x = x;
        this.y = y;
        this.xVelocity = speed * direction;
    }

    public int getDirection(){
        return direction;
    }

    public void update(long fps, float gravity){
        x += xVelocity / fps;
    }

    public void hideBullet(){
        this.x = -100;
        this.xVelocity = 0;
    }

    public float getX(){
        return x;
    }

    public float getY(){
        return y;
    }

}
```

现在，让我们实现`MachineGun`类。

创建一个新的 Java 类，将其命名为`MachineGun`。首先，我们添加一些成员。`maxBullets`变量不是玩家拥有的射击次数，那是无限的，它是`MachineGun`类可以拥有的子弹对象数量。对于非常快速射击的枪来说，10 个就足够了，正如我们将看到的。成员`numBullets`和`nextBullet`帮助类跟踪其 10 个子弹。`rateOfFire`变量控制玩家能够多快地按下射击按钮，`lastShotTime`通过跟踪上次发射子弹的系统时间来帮助执行`rateOfFire`。射速将是武器可升级的方面。

输入我们讨论过的代码如下。

```java
import java.util.concurrent.CopyOnWriteArrayList;

public class MachineGun extends GameObject{
    private int maxBullets = 10;
    private int numBullets;
    private int nextBullet;
    private int rateOfFire = 1;//bullets per second
    private long lastShotTime;

    private CopyOnWriteArrayList<Bullet> bullets;

    int speed = 25;
```

### 注意

对于功能性目的，我们可以将存储我们子弹的`CopyOnWriteArrayList` `bullets`视为一个普通的`ArrayList`对象。我们使用这个更复杂且稍慢的类，因为它线程安全，当玩家点击射击按钮时，子弹可能会同时从 UI 线程以及我们自己的线程中被访问。这篇文章解释了`CopyOnWriteArrayList`，如果你想知道更多，请访问：

[如何处理并发修改异常](http://examples.javacodegeeks.com/java-basics/exceptions/java-util-concurrentmodificationexception-how-to-handle-concurrent-modification-exception/)

我们有一个构造函数，它只是初始化子弹，`lastShotTime`和`nextBullet`：

```java
MachineGun(){
   bullets = new CopyOnWriteArrayList<Bullet>();
   lastShotTime = -1;
   nextBullet = -1;
}
```

在这里，我们通过调用每个子弹的`bullet.update`方法，更新枪支控制的所有`Bullet`对象。

```java
public void update(long fps, float gravity){
        //update all the bullets
        for(Bullet bullet: bullets){
            bullet.update(fps, gravity);
        }
    }
```

接下来，我们有一些 getter，它们将让我们了解有关我们的枪及其子弹的信息，以便进行像碰撞检测和绘制子弹等操作。

```java
public int getRateOfFire(){
  return rateOfFire;
}

public void setFireRate(int rate){
  rateOfFire = rate;
}

public int getNumBullets(){
  //tell the view how many bullets there are
  return numBullets;
}

public float getBulletX(int bulletIndex){
  if(bullets != null && bulletIndex < numBullets) {
       return bullets.get(bulletIndex).getX();
    }

  return -1f;
}

public float getBulletY(int bulletIndex){
  if(bullets != null) {
       return bullets.get(bulletIndex).getY();
     }
     return -1f;
}
```

我们还有一个快速帮助方法，当我们想要停止绘制子弹时使用。我们在`shoot`方法中将其隐藏，直到准备好重新分配。

```java
public void hideBullet(int index){
  bullets.get(index).hideBullet();
}
```

一个返回旅行方向的 getter：

```java
public int getDirection(int index){
  return bullets.get(index).getDirection();
}
```

现在，我们添加一个更全面的方法，该方法实际射出一颗子弹。该方法将上一次射击的时间与当前的`rateOfFire`进行比较。然后继续增加`nextBullet`并在允许的情况下创建一个新的`Bullet`对象。子弹以 Bob 面向的同一方向飞速射出。请注意，如果成功发射了子弹，该方法将返回`true`。这样，`InputController`类可以播放与玩家按钮按下相对应的声音效果。

```java
public boolean shoot(float ownerX, float ownerY, 
    int ownerFacing, float ownerHeight){

    boolean shotFired = false;
    if(System.currentTimeMillis() - lastShotTime  >                          
      1000/rateOfFire){

        //spawn another bullet;
        nextBullet ++;

        if(numBullets >= maxBullets){
            numBullets = maxBullets;
        }

        if(nextBullet == maxBullets){
            nextBullet = 0;
        }

        lastShotTime = System.currentTimeMillis();
        bullets.add(nextBullet, 
                new Bullet(ownerX, 
                (ownerY+ ownerHeight/3), speed, ownerFacing));

        shotFired = true;
        numBullets++;
    }
    return shotFired;
}
```

最后，我们有一个方法，当玩家找到机枪升级包时调用。我们将在本章后面看到更多相关内容。在这里，我们只是增加了`rateOfFire`，这使得玩家可以更猛烈地敲击开火按钮，并且仍然能够得到效果。

```java
public void upgradeRateOfFire(){
  rateOfFire += 2;
}
}// End of MachineGun class
```

现在，我们将修改`Player`类以携带一把`MachineGun`。给`Player`一个类型为`MachineGun`的成员变量。

```java
public MachineGun bfg;
```

接下来，在`Player`构造函数中，添加一行代码来初始化我们的新`MachineGun`对象：

```java
bfg = new MachineGun();
```

在`Player`类的`update`方法中，在我们为玩家调用`move()`之前，添加对`MachineGun`类的`update`方法的调用。如下所示突出：

```java
bfg.update(fps, gravity);

// Let's go!
this.move(fps);
```

向`Player`类添加一个方法，这样我们的`InputController`就可以访问虚拟触发器。正如我们所见，如果成功射击，该方法将返回`true`，这样`InputController`类就知道是否播放射击声音。

```java
public boolean pullTrigger() {
        //Try and fire a shot
        return bfg.shoot(this.getWorldLocation().x,  
           this.getWorldLocation().y, 
           getFacing(), getHeight());
}
```

现在，我们可以在`InputController`类中做一些小的添加，让玩家能够开火。要添加的代码在现有代码中突出显示：

```java
} else if (jump.contains(x, y)) {
  l.player.startJump(sound);

} else if (shoot.contains(x, y)) {
 if (l.player.pullTrigger()) {
 sound.playSound("shoot");
 }

} else if (pause.contains(x, y)) {
  l.switchPlayingStatus();

}
```

不要忘记我们新的控制系统的工作方式，我们还需要在`InputController`类的`MotionEvent.ACTION_POINTER_DOWN`情况下的更下方添加同样的额外代码。像往常一样，这里是有很多上下文背景的突出代码：

```java
} else if (jump.contains(x, y)) {
  l.player.startJump(sound);

} else if (shoot.contains(x, y)) {
 if (l.player.pullTrigger()) {
 sound.playSound("shoot");
}

} else if (pause.contains(x, y)) {
  l.switchPlayingStatus();
}
```

现在我们有了一把枪，它已装填好，我们知道如何扣动扳机。我们只需要绘制子弹。

在`draw`方法中添加新代码，在我们绘制调试文本之前，如下所示：

```java
//draw the bullets
paint.setColor(Color.argb(255, 255, 255, 255));
for (int i = 0; i < lm.player.bfg.getNumBullets(); i++) {
   // Pass in the x and y coords as usual
   // then .25 and .05 for the bullet width and height
   toScreen2d.set(vp.worldToScreen
            (lm.player.bfg.getBulletX(i),
            lm.player.bfg.getBulletY(i),
            .25f,
            .05f));

        canvas.drawRect(toScreen2d, paint);
}

// Text for debugging
if (debugging) {
// etc
```

我们现在将发射一些子弹。请注意，开火速率令人不满意且缓慢。我们将添加一些收集品，玩家可以获得这些收集品以增加他的枪的开火速率。

## 收集品

收集品是玩家可以收集的游戏对象。它们包括像升级包、额外生命、金钱等。我们现在将实现其中每一个收集品。由于我们的游戏引擎是这样设置的，这将出奇地简单。

我们首先要创建一个类来保存当前玩家的状态。我们想要监控收集到的金钱、机枪的火力以及剩余的生命。我们将其称为`PlayerState`。创建一个新的 Java 类，并将其命名为`PlayerState`。

除了我们刚才讨论的那些变量之外，我们还希望`PlayerState`类记住一个*x*和*y*位置，以便在玩家失去生命时进行重生。输入这些成员变量和简单的构造函数：

```java
import android.graphics.PointF;

public class PlayerState {

    private int numCredits;
    private int mgFireRate;
    private int lives;
    private float restartX;
    private float restartY;

    PlayerState() {
        lives = 3;
        mgFireRate = 1;
        numCredits = 0;
    }
```

现在，我们需要一个方法，我们可以调用它来初始化重生位置。我们稍后会用到这个方法。此外，我们还需要一个方法来重新加载位置。这是`PlayerState`类的接下来两个方法：

```java
public void saveLocation(PointF location) {
   // The location saves each time the player uses a teleport
     restartX = location.x;
     restartY = location.y;
}

public PointF loadLocation() {
   // Used every time the player loses a life
   return new PointF(restartX, restartY);
}
```

我们只需要一堆 getter 和 setter，以便访问这个类的成员：

```java
public int getLives(){
  return lives;
}

public int getFireRate(){
  return mgFireRate;
}

public void increaseFireRate(){
  mgFireRate += 2;
}

public void gotCredit(){
  numCredits ++;
}

public int getCredits(){
  return numCredits;
}

public void loseLife(){
  lives--;
}

public void addLife(){
  lives++;
}

public void resetLives(){
  lives = 3;
}
public void resetCredits(){
  lives = 0;
}

}// End PlayerState class
```

接下来，在`PlatformView`类中声明一个`PlayerState`类型的成员对象：

```java
// Our new engine classes
private LevelManager lm;
private Viewport vp;
InputController ic;
SoundManager sm;
private PlayerState ps;

```

在`PlatformView`构造函数中初始化它：

```java
vp = new Viewport(screenWidth, screenHeight);
sm = new SoundManager();
sm.loadSound(context);
ps = new PlayerState();

loadLevel("LevelCave", 10, 2);
```

现在，在`loadLevel`方法中，创建一个`RectF`对象，保存玩家的起始位置，并将其传递给`PlayerState`对象`ps`以便妥善保存。每次玩家死亡时，都可以使用这个位置进行重生。

```java
ic = new InputController(vp.getScreenWidth(), vp.getScreenHeight());

PointF location = new PointF(px, py);
ps.saveLocation(location);

//set the players location as the world centre of the viewport
```

现在，我们将创建三个类，分别对应我们的三种收集物。这些类非常简单。它们扩展了`GameObject`，设置了位图，具有碰撞箱和在世界中的位置。还要注意，它们在构造函数中都接收一个类型，并使用`setType()`存储这个值。我们很快就会看到如何使用它们的类型来处理玩家“收集它们”时会发生的事情。创建三个新的 Java 类：`Coin`，`ExtraLife`和`MachineGunUpgrade`。注意，收集物比平台稍小一些，这可能正如我们所预期的。依次输入它们的代码。

以下是`Coin`的代码：

```java
public class Coin extends GameObject{

    Coin(float worldStartX, float worldStartY, char type) {

        final float HEIGHT = .5f;
        final float WIDTH = .5f;

        setHeight(HEIGHT); 
        setWidth(WIDTH); 

        setType(type);

        // Choose a Bitmap
        setBitmapName("coin");

        // Where does the tile start
        // X and y locations from constructor parameters
        setWorldLocation(worldStartX, worldStartY, 0);
        setRectHitbox();
    }

    public void update(long fps, float gravity){}
}
```

现在，对于`ExtraLife`：

```java
public class ExtraLife extends GameObject{

    ExtraLife(float worldStartX, float worldStartY, char type) {

        final float HEIGHT = .8f;
        final float WIDTH = .65f;

        setHeight(HEIGHT); 
        setWidth(WIDTH); 

        setType(type);

        // Choose a Bitmap

        setBitmapName("life");

        // Where does the tile start
        // X and y locations from constructor parameters
        setWorldLocation(worldStartX, worldStartY, 0);
        setRectHitbox();
    }

    public void update(long fps, float gravity){}
}
```

最后，`MachineGunUpgrade`类：

```java
public class MachineGunUpgrade extends GameObject{
    MachineGunUpgrade(float worldStartX, 
        float worldStartY, 
        char type) {

        final float HEIGHT = .5f;
        final float WIDTH = .5f;

        setHeight(HEIGHT); 
        setWidth(WIDTH); 

        setType(type);

        // Choose a Bitmap

        setBitmapName("clip");

        // Where does the tile start
        // X and y locations from constructor parameters
        setWorldLocation(worldStartX, worldStartY, 0);
        setRectHitbox();
    }

    public void update(long fps, float gravity){}
}
```

现在，更新`LevelManager`类，使其能够处理我们的关卡设计中这三个新对象，并将它们添加到`GameObjects`的`ArrayList`中。为此，我们需要在三个地方更新`LevelManager`类：`getBitmap()`，`getBitmapIndex()`和`loadMapData()`。以下是这些小更新的内容，新代码在现有代码中突出显示。

对`getBitmap()`进行以下添加：

```java
case 'p':
  index = 2;
  break;

case 'c':
 index = 3;
 break;

case 'u':
 index = 4;
 break;

case 'e':
 index = 5;
 break;

default:
  index = 0;
  break;
```

进行相同的添加，但这次是在`getBitmapIndex()`中：

```java
case 'p':
  index = 2;
  break;

case 'c':
 index = 3;
 break;

case 'u':
 index = 4;
 break;

case 'e':
 index = 5;
 break;

default:
  index = 0;
  break;
```

在`LevelManager`中进行最后的修改，对`loadMapData()`进行以下添加：

```java
case 'p':// a player
    // Add a player to the gameObjects
    gameObjects.add(new Player(context, px, py, pixelsPerMetre));
    // We want the index of the player
    playerIndex = currentIndex;
    // We want a reference to the player object
    player = (Player) gameObjects.get(playerIndex);
    break;

case 'c':
 // Add a coin to the gameObjects
 gameObjects.add(new Coin(j, i, c));
 break;

case 'u':
 // Add a machine gun upgrade to the gameObjects
 gameObjects.add(new MachineGunUpgrade(j, i, c));
 break;

case 'e':
 // Add an extra life to the gameObjects
 gameObjects.add(new ExtraLife(j, i, c));
 break;
}

```

现在，我们可以将三个适当命名的图形添加到 drawable 文件夹中，并开始将它们添加到我们的`LevelCave`设计中。继续从下载捆绑包中的`Chapter7/drawables`文件夹复制`clip.png`，`coin.png`和`life.png`到你的 Android Studio 项目的`drawable`文件夹中。

添加一系列注释，标识所有游戏对象类型。我们将在项目过程中添加这些注释，以及它们在关卡设计中的字母数字代码。将以下注释添加到`LevelData`类中：

```java
// Tile types
// . = no tile
// 1 = Grass
// 2 = Snow
// 3 = Brick
// 4 = Coal
// 5 = Concrete
// 6 = Scorched
// 7 = Stone

//Active objects
// g = guard
// d = drone
// t = teleport
// c = coin
// u = upgrade
// f = fire
// e  = extra life

//Inactive objects
// w = tree
// x = tree2 (snowy)
// l = lampost
// r = stalactite
// s = stalacmite
// m = mine cart
// z = boulders

```

在我们增强`LevelCave`类以使用我们的新对象之前，我们想要检测玩家收集它们或与它们碰撞的时刻，并采取适当的行动。我们首先会在`Player`类中添加一个快速辅助方法。这样做的原因是，当玩家与另一个对象碰撞时，`Player`类中`checkCollisions`方法的默认动作是停止角色移动。我们不希望拾取物发生这种情况，因为这会让玩家感到烦恼。因此，我们将在`Player`类中快速添加一个`restorePreviousVelocity`方法，在我们不希望发生默认动作时调用它。将此方法添加到`Player`类中：

```java
public void restorePreviousVelocity() {
  if (!isJumping && !isFalling) {
       if (getFacing() == LEFT) {
           isPressingLeft = true;
           setxVelocity(-MAX_X_VELOCITY);
         } else {
           isPressingRight = true;
                     setxVelocity(MAX_X_VELOCITY);
       }
    }
}
```

现在，我们可以依次处理每个拾取物的碰撞。在`PlatformView`类的`update`方法中处理碰撞的 switch 块内，添加以下情况来处理我们的三个拾取物：

```java
switch (go.getType()) {
 case 'c':
 sm.playSound("coin_pickup");
 go.setActive(false);
 go.setVisible(false);
 ps.gotCredit();

 // Now restore state that was 
 // removed by collision detection
 if (hit != 2) {// Any hit except feet
 lm.player.restorePreviousVelocity();
 }
 break;

case 'u':
 sm.playSound("gun_upgrade");
 go.setActive(false);
 go.setVisible(false);
 lm.player.bfg.upgradeRateOfFire();
 ps.increaseFireRate();
 if (hit != 2) {// Any hit except feet
 lm.player.restorePreviousVelocity();
 }
 break;

case 'e':
 //extralife
 go.setActive(false);
 go.setVisible(false);
 sm.playSound("extra_life");
 ps.addLife();

 if (hit != 2) {
 lm.player.restorePreviousVelocity();
 }
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
}
```

最后，将新对象添加到我们的`LevelCave`类中。

### 提示

下面的代码片段，我建议是用于演示我们新对象的简单新布局，但你的布局可以尽可能大或者复杂。我们将在下一章设计并链接一些关卡时做一些更复杂的事情。

将以下代码输入到`LevelCave`中，或者用你自己的设计进行扩展：

```java
public class LevelCave extends LevelData{
  LevelCave() {
    tiles = new ArrayList<String>();
 this.tiles.add("p.............................................");
 this.tiles.add("..............................................");
 this.tiles.add("..............................................");
 this.tiles.add("..............................................");
 this.tiles.add("....................c.........................");
 this.tiles.add("....................1........u................");
 this.tiles.add(".................c..........u1................");
 this.tiles.add(".................1.........u1.................");
 this.tiles.add("..............c...........u1..................");
 this.tiles.add("..............1..........u1...................");
 this.tiles.add("......................e..1....e.....e.........");
 this.tiles.add("....11111111111111111111111111111111111111....");
}

```

这就是简单布局的样子：

![拾取物](img/B04322_07_01.jpg)

尝试收集拾取物，你会听到愉悦的声音效果。此外，每次我们收集一个拾取物，`PlayerState`类就会存储一个更新。这将在我们下一章构建一个 HUD 时非常有用。最有趣的是；如果你收集了机枪升级，然后尝试射击，你会发现使用起来更加令人满意。

我们最好让这些子弹发挥作用。不过，在我们这样做之前，让我们给玩家提供一些炮灰，形式是几个敌人。

### 无人机

无人机是一个简单但邪恶的敌人。它将在视口中检测到玩家并直接向玩家飞去。如果无人机接触到玩家，那么玩家将立即死亡。

让我们构建一个`Drone`类。创建一个新的 Java 类，将其命名为`Drone`。我们需要成员变量来记录我们设置最后一个航点的时刻。这将限制无人机获取 Bob 坐标导航更新的频率。这将阻止无人机过于精确地打击目标。它需要一个航点/目标坐标，还需要知道通过`MAX_X_VELOCITY`和`MAX_Y_VELOCITY`的速度限制。

```java
import android.graphics.PointF;

public class Drone extends GameObject {

    long lastWaypointSetTime;
    PointF currentWaypoint;

    final float MAX_X_VELOCITY = 3;
    final float MAX_Y_VELOCITY = 3;
```

现在，在`Drone`构造函数中，初始化常规的`GameObject`成员，特别是`Drone`类的成员，如`currentWaypoint`。不要忘记，如果我们打算射击无人机，它将需要一个碰撞箱，我们在调用`setWorldLocation()`之后调用`setRectHitBox()`。

```java
Drone(float worldStartX, float worldStartY, char type) {
    final float HEIGHT = 1;
    final float WIDTH = 1;
    setHeight(HEIGHT); // 1 metre tall
    setWidth(WIDTH); // 1 metres wide

    setType(type);

    setBitmapName("drone");
    setMoves(true);
    setActive(true);
    setVisible(true);

    currentWaypoint = new PointF();

    // Where does the drone start
    // X and y locations from constructor parameters
    setWorldLocation(worldStartX, worldStartY, 0);
    setRectHitbox();
    setFacing(RIGHT);
}
```

这是`update`方法的实现，它将比较无人机的坐标与其`currentWaypoint`变量，并据此改变其速度。然后，我们通过调用`move()`然后是`setRectHitbox()`来结束`update()`。

```java
public void update(long fps, float gravity) {
  if (currentWaypoint.x > getWorldLocation().x) {
       setxVelocity(MAX_X_VELOCITY);
   } else if (currentWaypoint.x < getWorldLocation().x) {
       setxVelocity(-MAX_X_VELOCITY);
   } else {
       setxVelocity(0);
   }

    if (currentWaypoint.y >= getWorldLocation().y) {
       setyVelocity(MAX_Y_VELOCITY);
     } else if (currentWaypoint.y < getWorldLocation().y) {
       setyVelocity(-MAX_Y_VELOCITY);
     } else {
       setyVelocity(0);
  }

  move(fps);

  // update the drone hitbox
   setRectHitbox();

}
```

在`Drone`类的最后一个方法中，通过传入 Bob 的坐标作为参数来更新`currentWaypoint`变量。注意，我们会检查是否已经过了足够的时间来进行更新，以确保我们的无人机不会过于精确。

```java
public void setWaypoint(Vector2Point5D playerLocation) {
  if (System.currentTimeMillis() > lastWaypointSetTime + 2000) {//Has 2 seconds passed
        lastWaypointSetTime = System.currentTimeMillis();
        currentWaypoint.x = playerLocation.x;
        currentWaypoint.y = playerLocation.y;
     }
}
}// End Drone class
```

将`drone.png`图形文件从`Chapter7/drawable`文件夹添加到项目的`drawable`文件夹中。

接下来，我们需要在`LevelManager`类中添加无人机，就像我们对每个拾取物品所做的那样，在三个常规位置添加。现在，在`getBitmap()`、`getBitmapIndex()`和`loadMapData()`方法中添加代码。这是按顺序需要添加的三个小部分代码。

在`getBitmap`方法中添加高亮显示的代码：

```java
case 'e':
  index = 5;
  break;

case 'd':
 index = 6;
 break;

default:
  index = 0;
  break;
```

在`getBitmapIndex`方法中添加高亮显示的代码：

```java
case 'e':
  index = 5;
  break;

case 'd':
 index = 6;
 break;

default:
  index = 0;
  break;
```

在`loadMapData`方法中添加高亮显示的代码：

```java
case 'e':
   // Add an extra life to the gameObjects
   gameObjects.add(new ExtraLife(j, i, c));
   break;

case 'd':
 // Add a drone to the gameObjects
 gameObjects.add(new Drone(j, i, c));
 break;

```

一个迫切的问题是：无人机如何知道要去哪里？在每一帧中，如果视口内有无人机，我们可以发送玩家的坐标。在`PlatformView`类的`update`方法中执行以下代码块所示的操作。

与往常一样，新代码以高亮形式展示，并嵌入到现有代码的上下文中。如果你记得`Drone`类中的`setWaypoint()`代码，它只接受每 2 秒更新一次。这防止了无人机过于精确。

```java
if (lm.isPlaying()) {
   // Run any un-clipped updates
   go.update(fps, lm.gravity);

 if (go.getType() == 'd') {
 // Let any near by drones know where the player is
 Drone d = (Drone) go;
 d.setWaypoint(lm.player.getWorldLocation());
 }
}
```

现在，这些邪恶的无人机可以策略性地放置在关卡周围，它们会锁定玩家。要使无人机完全运作，我们需要做的最后一件事是检测它们实际上是否与玩家发生了碰撞。这非常简单。只需在`PlatformView`类的`update`方法中的碰撞检测`switch`块中为无人机添加一个案例：

```java
case 'e':
  //extralife
   go.setActive(false);
   go.setVisible(false);
   sm.playSound("extra_life");
   ps.addLife();
   if (hit != 2) {// Any hit except feet
       lm.player.restorePreviousVelocity();
   }
   break;

case 'd':
 PointF location;
 //hit by drone
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
```

继续在`LevelCave`中添加大量无人机，并观察它们向玩家飞去。注意，如果无人机捕捉到玩家，玩家会死亡并重新生成。

![无人机](img/B04322_07_02.jpg)

现在，尽管世界上已经有足够多的敌方无人机使它变得危险，但让我们再添加一种类型的敌人。

### 守卫

守卫敌人将是一个脚本练习。我们将让`LevelManager`类自动生成一个简单的脚本，为我们的守卫生成一个巡逻路线。

路线将尽可能简单；它只包括两个守卫会不断巡逻的点。预编程两个预定的航点会更快捷、更简单。然而，如果自动生成，我们可以根据需要（在一定的参数范围内）在任何设计的关卡上放置守卫，行为将由系统处理。

我们的守卫将会有动画效果，因此我们将在构造函数中使用一个精灵表单并配置动画细节，就像我们对`Player`类所做的那样。

创建一个新类，并将其命名为`Guard`。首先，处理成员变量。我们的`Guard`类不仅需要两个航点，还需要一个变量来指示当前的航点是哪一个。像其他移动对象一样，它需要速度。以下是开始编写你的类的类声明和成员变量：

```java
import android.content.Context;

public class Guard extends GameObject {

    // Guards just move on x axis between 2 waypoints

    private float waypointX1;// always on left
    private float waypointX2;// always on right
    private int currentWaypoint;
    final float MAX_X_VELOCITY = 3;
```

我们需要通过构造函数设置我们的守卫。首先，设置我们的动画变量、位图和大小。然后像往常一样，设置守卫在关卡中的位置、它的碰撞箱以及它面向的方向。然而，在构造函数的最后一行，我们将`currentWaypoint`设置为`1`；这是新的。我们将在该类的`update`方法中看到这是如何影响守卫的行为的。

```java
Guard(Context context, float worldStartX, 
  float worldStartY, char type, 
  int pixelsPerMetre) {

        final int ANIMATION_FPS = 8;
        final int ANIMATION_FRAME_COUNT = 5;
        final String BITMAP_NAME = "guard";
        final float HEIGHT = 2f;
        final float WIDTH = 1;

        setHeight(HEIGHT); // 2 metre tall
        setWidth(WIDTH); // 1 metres wide

        setType(type);

        setBitmapName("guard");
        // Now for the player's other attributes
        // Our game engine will use these
        setMoves(true);
        setActive(true);
        setVisible(true);

        // Set this object up to be animated
        setAnimFps(ANIMATION_FPS);
        setAnimFrameCount(ANIMATION_FRAME_COUNT);
        setBitmapName(BITMAP_NAME);
        setAnimated(context, pixelsPerMetre, true);

        // Where does the tile start
        // X and y locations from constructor parameters
        setWorldLocation(worldStartX, worldStartY, 0);
        setxVelocity(-MAX_X_VELOCITY);
        currentWaypoint = 1;
}
```

接着，添加一个方法，供我们的`LevelManager`类使用，以告知`Guard`类其两个航点是什么：

```java
public void setWaypoints(float x1, float x2){
  waypointX1 = x1;
  waypointX2 = x2;
}
```

现在，我们将编写`Guard`类的“大脑”部分，也就是它的`update`方法。你基本上可以将这个方法分为两个主要部分。首先，`if(currentWaypoint == 1)`，其次，`if(currentWaypoint == 2)`。在这两个`if`块内部，只需检查守卫是否已经到达或通过了适当的航点。如果是，则切换航点，反转速度，并让守卫面向另一个方向。

最后，调用`move()`然后`setRectHitbox()`，以更新碰撞箱到守卫的新位置。添加`update`方法的代码，然后我们将看到如何让它工作。

```java
public void update(long fps, float gravity) {
  if(currentWaypoint == 1) {// Heading left
       if (getWorldLocation().x <= waypointX1) {
          // Arrived at waypoint 1
           currentWaypoint = 2;
           setxVelocity(MAX_X_VELOCITY);
           setFacing(RIGHT);
      }
  }

  if(currentWaypoint == 2){
    if (getWorldLocation().x >= waypointX2) {
         // Arrived at waypoint 2
          currentWaypoint = 1;
          setxVelocity(-MAX_X_VELOCITY);
          setFacing(LEFT);
      }
  }

  move(fps);
   // update the guards hitbox
   setRectHitbox();
}
}// End Guard class
```

记得从下载包的`Chapter7/drawables`文件夹中添加`guard.png`到项目的`drawable`文件夹中。

现在，我们可以在`LevelManager`类中进行通常的三处添加，以加载可能在我们的关卡设计中找到的任何守卫。

在`getBitmap()`中，添加高亮显示的代码：

```java
case 'd':
  index = 6;
  break;

case 'g':
 index = 7;
 break;

default:
  index = 0;
  break;
```

在`getBitmapIndex()`中，添加高亮显示的代码：

```java
case 'd':
  index = 6;
  break;

case 'g':
 index = 7;
 break;

default:
  index = 0;
  break;
```

在`loadMapData()`中，添加高亮显示的代码：

```java
case 'd':
     // Add a drone to the gameObjects
     gameObjects.add(new Drone(j, i, c));
     break;
case 'g':
 // Add a guard to the gameObjects
 gameObjects.add(new Guard(context, j, i, c, pixelsPerMetre));
 break;

```

我们很快将为`LevelManager`添加一个全新的功能。那就是一个将创建脚本（设置两个巡逻航点）的方法。为了让这个新方法工作，它需要知道瓦片是否适合行走。我们将为`GameObject`添加一个新属性、一个获取器和设置器，以便轻松发现这一点。

在`GameObject`类的类声明后直接添加这个新成员：

```java
private boolean traversable = false;
```

向`GameObject`类添加这两个方法，以获取和设置这个变量：

```java
public void setTraversable(){
  traversable = true;
}

public boolean isTraversable(){
  return traversable;
}
```

现在，在`Grass`类的构造函数中，添加对`setTraversable()`的调用。如果我们希望守卫能够在上面巡逻，我们必须记得为所有未来设计的`GameObject`派生类做这一点。在`Grass`中，在构造函数顶部添加这一行：

```java
setTraversable();
```

接下来，我们将查看为`LevelManager`类新增加的`setWaypoints`方法。它需要检查关卡设计，并为关卡中存在的任何`Guard`对象计算两个航点。

我们将把这个方法分成几个部分，以便我们可以看到每个阶段的操作。

首先，我们需要遍历所有的`gameObjects`类，寻找`Guard`对象。

```java
public void setWaypoints() {
  // Loop through all game objects looking for Guards
    for (GameObject guard : this.gameObjects) {
       if (guard.getType() == 'g') {
```

如果我们到达代码的这一部分，这意味着我们已经找到了一个需要设置两个航点的守卫。首先，我们需要找到守卫“站立”的瓷砖。然后，我们计算每侧最后一个可通行的瓷砖的坐标，但最大范围是每个方向五个瓷砖。这两个点将作为两个航点。以下是添加到`setWaypoints`方法中的代码。它包含大量注释，以清晰说明情况而不中断流程。

```java
// Set waypoints for this guard
// find the tile beneath the guard
// this relies on the designer putting 
// the guard in sensible location

int startTileIndex = -1;
int startGuardIndex = 0;
float waypointX1 = -1;
float waypointX2 = -1;

for (GameObject tile : this.gameObjects) {
    startTileIndex++;
    if (tile.getWorldLocation().y == 
            guard.getWorldLocation().y + 2) {

        // Tile is two spaces below current guard
        // Now see if has same x coordinate
        if (tile.getWorldLocation().x == 
            guard.getWorldLocation().x) {

            // Found the tile the guard is "standing" on
            // Now go left as far as possible 
            // before non travers-able tile is found
            // Either on guards row or tile row
            // upto a maximum of 5 tiles. 
            //  5 is an arbitrary value you can
            // change it to suit

            for (int i = 0; i < 5; i++) {// left for loop
                if (!gameObjects.get(startTileIndex -
                    i).isTraversable()) {

                    //set the left waypoint
                    waypointX1 = gameObjects.get(startTileIndex - 
                        (i + 1)).getWorldLocation().x;

                     break;// Leave left for loop
                     } else {
                    // Set to max 5 tiles as 
                    // no non traversible tile found
                    waypointX1 = gameObjects.get(startTileIndex -
                        5).getWorldLocation().x;
               }
                }// end get left waypoint

                for (int i = 0; i < 5; i++) {// right for loop
                    if (!gameObjects.get(startTileIndex +
                        i).isTraversable()) {

                        //set the right waypoint
                        waypointX2 = gameObjects.get(startTileIndex +
                            (i - 1)).getWorldLocation().x;

                    break;// Leave right for loop
                    } else {
                    //set to max 5 tiles away
                    waypointX2 = gameObjects.get(startTileIndex +
                       5).getWorldLocation().x;
                }

                }// end get right waypoint

        Guard g = (Guard) guard;
        g.setWaypoints(waypointX1, waypointX2);
    }
}
}
}
}
}// End setWaypoints()
```

现在，我们可以在`LevelManager`构造函数的最后调用我们新的`setWaypoints`方法。我们需要在`GameObject`类的`ArrayList`填充完毕后调用此方法，否则其中将没有守卫。像这样突出显示添加对`setWaypoints()`的调用：

```java
// Load all the GameObjects and Bitmaps
loadMapData(context, pixelsPerMetre, px, py);
// Set waypoints for our guards
setWaypoints();

```

接下来，将这段代码添加到`PlatformView`类的`update`方法中的碰撞检测开关块中，以便我们可以与守卫相撞。

```java
case 'd':
    PointF location;
    //hit by drone
    sm.playSound("player_burn");
    ps.loseLife();
    location = new PointF(ps.loadLocation().x, 
        ps.loadLocation().y);

    lm.player.setWorldLocationX(location.x);
    lm.player.setWorldLocationY(location.y);
    lm.player.setxVelocity(0);
    break;

case 'g':
 // Hit by guard
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
```

最后，向`LevelCave`类中添加一些`g`字母。确保将它们放置在平台上方一个空格的位置，因为它们的高度是 2 米，如下面的伪代码所示：

```java
................g............................
...........................d.................
111111111111111111111111111111111111111111111
```

![守卫](img/B04322_07_03.jpg)

# 总结

我们实现了枪支、拾取物、无人机和守卫。这意味着我们现在有很多危险，但拥有一把无法造成伤害的机枪。我们将在下一章首先解决这个问题，为我们的子弹实现碰撞检测。然而，我们的目标不仅仅是让子弹击中敌人。
