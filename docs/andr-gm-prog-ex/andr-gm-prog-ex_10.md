# 第十章. 使用 OpenGL ES 2 进行移动和绘制

在本章中，我们将实现所有的图形、游戏玩法和移动。在 30 多页的内容中，我们将完成除了碰撞检测之外的所有内容。我们能完成这么多，是因为我们在上一章打下了基础。

首先，我们将在游戏世界周围绘制一个静态边界，然后是一些闪烁的星星，接着为我们的太空船添加移动以及一些子弹。在那之后，我们将快速添加玩家的控制，我们将在屏幕上飞快地移动。 

我们还将通过实现带有一些新声音效果的`SoundManager`类来制造一些噪音。

完成这些后，我们将添加随机形状的小行星，这些小行星在旋转的同时穿过整个世界。

然后，我们可以添加一个 HUD 来突出屏幕的可触摸区域，并提供剩余玩家生命值和需要摧毁的小行星数量的统计。

# 绘制静态游戏边界

在这个简单的类中，我们定义了四组点，它们将代表四条线。毫不奇怪，`GameObject`类将使用这些点作为线的端点来绘制边界。

在构造函数中，也就是类的全部内容，我们通过调用`setType()`设置类型，将世界位置设置为地图中心，以及将`height`和`width`设置为整个地图的高度和宽度。

然后，我们在一个 float 数组中定义四条线，并调用`setVertices()`来准备一个`FloatBuffer`。

创建一个名为`Border`的新类，并添加以下代码：

```java
public class Border extends GameObject{

  public Border(float mapWidth, float mapHeight){

        setType(Type.BORDER);
        //border center is the exact center of map
        setWorldLocation(mapWidth/2,mapHeight/2);

        float w = mapWidth;
        float h = mapHeight;
        setSize(w, h);

       // The vertices of the border represent four lines
       // that create a border of a size passed into the constructor
       float[] borderVertices = new float[]{
           // A line from point 1 to point 2
            - w/2, -h/2, 0,
            w/2, -h/2, 0,
            // Point 2 to point 3
            w/2, -h/2, 0,
            w/2, h/2, 0,
            // Point 3 to point 4
            w/2, h/2, 0,
            -w/2, h/2, 0,
            // Point 4 to point 1
            -w/2, h/2, 0,
            - w/2, -h/2, 0,
    };

        setVertices(borderVertices);

  }

}
```

然后，我们可以像这样在`GameManager`中声明一个`Border`对象：

```java
// Our game objects
SpaceShip ship;
Border border;

```

在`AsteroidsRenderer`的`createObjects`方法中这样初始化它：

```java
// Create our game objects

// First the ship in the center of the map
gm.ship = new SpaceShip(gm.mapWidth / 2, gm.mapHeight / 2);

// The deadly border
gm.border = new Border(gm.mapWidth, gm.mapHeight);

```

现在，我们可以在`AsteroidsRendrer`类的`draw`方法中添加一行代码来绘制我们的边界：

```java
gm.ship.draw(viewportMatrix);
gm.border.draw(viewportMatrix);

```

你现在可以运行游戏了。如果你想实际看到边界，可以将我们初始化飞船的位置改到靠近边界的地方。记住，在`draw`方法中，我们将视口围绕飞船居中。要看到边界，将`SpaceShip`类中的这一行改为这样：

```java
setWorldLocation(10,10);
```

运行游戏看看效果。

![绘制静态游戏边界](img/B043422_10_01.jpg)

改回这一行：

```java
setWorldLocation(worldLocationX,worldLocationY);
```

现在，我们将在边框内填充星星。

# 闪烁的星星

我们将使边界更加动态，而不仅仅是静态的。在这里，我们将向一个简单的`Star`类中添加一个`update`方法，该方法可以用来随机地打开或关闭星星。

我们将类型设置为`normal`，并在边界的范围内为星星创建一个随机位置，并像往常一样调用`setWorldLocation()`。

星星将被绘制成点，因此我们的顶点数组将只包含模型空间 0,0,0 的一个顶点。然后，我们像往常一样调用`setVertices()`。

创建一个新类，命名为`Star`，并输入我们讨论过的代码：

```java
public class Star extends GameObject{

    // Declare a random object here because
    // we will use it in the update() method
    // and we don't want GC to have to keep clearing it up
    Random r;

    public Star(int mapWidth, int mapHeight){
    setType(Type.STAR);
    r = new Random();
    setWorldLocation(r.nextInt(mapWidth),r.nextInt(mapHeight));

    // Define the star
    // as a single point
    // in exactly the coordinates as its world location
    float[] starVertices = new float[]{

                0,
                0,
                0

    };

    setVertices(starVertices);

    }
```

这是我们的`Star`类的`update`方法。正如我们所见，每一帧都有千分之一的机会让星星改变其状态。为了更多闪烁，请使用较低的种子值，为了减少闪烁，请使用较高的种子值。

```java
public void update(){

  // Randomly twinkle the stars
     int n = r.nextInt(1000);
     if(n == 0){
       // Switch on or off
       if(isActive()){
         setActive(false);
        }else{
          setActive(true);
        }
   }

}
```

然后，我们在`GameManager`中声明一个`Star`数组成员，以及一个额外的`int`变量来控制我们想要绘制的星星数量，如下所示：

```java
// Our game objects
SpaceShip ship;
Border border;
Star[] stars;
int numStars = 200;

```

在`AsteroidsRenderer`的`createObjects`方法中初始化`Star`对象的数组，如下所示：

```java
// The deadly border
gm.border = new Border(gm.mapWidth, gm.mapHeight);

// Some stars
gm.stars = new Star[gm.numStars];
for (int i = 0; i < gm.numStars; i++) {

 // Pass in the map size so the stars no where to spawn
 gm.stars[i] = new Star(gm.mapWidth, gm.mapHeight);
}

```

现在，我们可以在`AsteroidsRenderer`类的`draw`方法中添加以下代码行来绘制我们的星星。注意，我们首先绘制星星，因为它们在背景中。

```java
// Start drawing!

// Some stars
for (int i = 0; i < gm.numStars; i++) {

 // Draw the star if it is active
 if(gm.stars[i].isActive()) {
 gm.stars[i].draw(viewportMatrix);
 }
}

gm.ship.draw(viewportMatrix);
gm.border.draw(viewportMatrix);
```

当然，为了使它们闪烁，我们从`AsteroidsRenderer`类的`update`方法中调用它们的`update`方法，如下所示：

```java
private void update(long fps) {

 // Update (twinkle) the stars
 for (int i = 0; i < gm.numStars; i++) {
 gm.stars[i].update();
 }

}
```

你现在可以运行游戏了：

![闪烁的星星](img/B043422_10_02.jpg)

# 让飞船生动起来

首先，我们需要为我们的`GameObject`类添加更多功能。我们在`GameObject`中这样做，因为子弹和行星与飞船共享许多惊人的相似之处。

我们需要一堆获取器和设置器来获取和设置旋转速率、行驶角度和面向角度。向`GameObject`类添加以下方法：

```java
public void setRotationRate(float rotationRate) {
  this.rotationRate = rotationRate;
}

public float getTravellingAngle() {
  return travellingAngle;
}

public void setTravellingAngle(float travellingAngle) {
  this.travellingAngle = travellingAngle;
}

public float getFacingAngle() {
  return facingAngle;
}

public void setFacingAngle(float facingAngle) {
  this.facingAngle = facingAngle;
}
```

现在，我们添加一个`move`方法，该方法根据当前的每秒帧数调整对象的*x*和*y*坐标以及`facingAngle`。添加`move`方法：

```java
void move(float fps){
  if(xVelocity != 0) {
       worldLocation.x += xVelocity / fps;
    }

     if(yVelocity != 0) {
       worldLocation.y += yVelocity / fps;
    }

     // Rotate
     if(rotationRate != 0) {
       facingAngle = facingAngle + rotationRate / fps;
    }

}
```

为了完善我们对`GameObject`类的添加，为速度、速度和最大速度添加以下获取器和设置器：

```java
public float getxVelocity() {
  return xVelocity;
}

public void setxVelocity(float xVelocity) {
  this.xVelocity = xVelocity;
}

public float getyVelocity() {
  return yVelocity;
}

public void setyVelocity(float yVelocity) {
  this.yVelocity = yVelocity;
}

public float getSpeed() {
  return speed;
}

public void setSpeed(float speed) {
  this.speed = speed;
}

public float getMaxSpeed() {
  return maxSpeed;
}

public void setMaxSpeed(float maxSpeed) {
  this.maxSpeed = maxSpeed;
}
```

我们可以为`SpaceShip`类添加一些内容。向`SpaceShip`类添加以下三个成员，以控制玩家的飞船是否在转向或向前移动：

```java
boolean isThrusting;
private boolean isPressingRight = false;
private boolean isPressingLeft = false;
```

现在，在`SpaceShip`构造函数内部，让我们设置飞船的最大速度。我在现有代码中突出了新的一行代码：

```java
setSize(width, length);

setMaxSpeed(150);

// It will be useful to have a copy of the
```

接下来，在`SpaceShip`类中，我们添加一个`update`方法，首先根据`isThrusting`是真是假来增加或减少速度。

```java
public void update(long fps){

float speed = getSpeed();
if(isThrusting) {
  if (speed < getMaxSpeed()){
       setSpeed(speed + 5);
     }

     }else{
       if(speed > 0) {
         setSpeed(speed - 3);
        }else {
         setSpeed(0);
        }
}
```

然后，我们根据角度、船舶面向的方向以及速度来设置*x*和*y*速度。

### 注意

我们使用速度乘以船舶面向角度的余弦值来设置在*x*轴上的速度。这样做有效是因为余弦函数是一个完美的变量，当船舶分别面向左或右时，它会返回-1 或 1 的值；当船舶正好指向上或下时，该变量返回精确的 0 值。它也在两者之间的角度返回精细的值。正弦函数在*y*轴上以完全相同的方式工作。代码看起来有些复杂，这是因为我们需要将角度转换为弧度，并且必须给我们的`facingAngle`加上 90 度，因为 0 度是指向三点钟方向。这个事实不利于我们按照现在的方式在*x*, *y*平面上使用它，所以我们将其修改为 90 度，船舶就能如预期般移动了。有关这一工作原理的更多信息，请查看以下教程：

[`gamecodeschool.com/essentials/calculating-heading-in-2d-games-using-trigonometric-functions-part-1/`](http://gamecodeschool.com/essentials/calculating-heading-in-2d-games-using-trigonometric-functions-part-1/)

```java
setxVelocity((float) 
  (speed* Math.cos(Math.toRadians(getFacingAngle() + 90))));

setyVelocity((float) 
  (speed* Math.sin(Math.toRadians(getFacingAngle() + 90))));

```

现在，我们根据玩家是向左还是向右转动来设置旋转速度。最后，我们调用`move()`以使所有更新生效。

```java
if(isPressingLeft){
  setRotationRate(360);
}

else if(isPressingRight){
  setRotationRate(-360);
     }else{
       setRotationRate(0);
    }

     move(fps);
}
```

现在，我们需要添加一个`pullTrigger`方法，目前我们只需返回`true`。我们还提供了三种方法供未来的`InputController`调用，触发`update`方法以进行各种更改。

```java
public boolean pullTrigger() {
  //Try and fire a shot
  // We could control rate of fire from here
  // But lets just return true for unrestricted rapid fire
  // You could remove this method and any code which calls it

   return true;
}

public void setPressingRight(boolean pressingRight) {
  isPressingRight = pressingRight;
}

public void setPressingLeft(boolean pressingLeft) {
  isPressingLeft = pressingLeft;
}

public void toggleThrust() {
  isThrusting = ! isThrusting;
}
```

我们已经在每一帧中绘制了飞船，但我们需要在`AsteroidsRenderer`类的`update`方法中添加一行代码。添加这行代码以调用`SpaceShip`类的`update`方法：

```java
// Update (twinkle) the stars
for (int i = 0; i < gm.numStars; i++) {
  gm.stars[i].update();
}

// Run the ship,s update() method
gm.ship.update(fps);

```

显然，在我们添加玩家控制之前，我们实际上无法移动。让我们快速向游戏中添加一些子弹。然后，我们将添加声音和控制，这样我们就可以看到和听到我们添加的酷炫新功能。

# 快速连发子弹

自 20 世纪 70 年代的 Pong 游戏以来，我就沉迷于游戏，记得当一位朋友在家中拥有一台太空侵略者游戏机大约一周时，我是多么兴奋。尽管真正让小行星比太空侵略者好的地方在于，你可以多快地进行射击。秉承这一传统，我们将制作一个令人满意的快速子弹流。

创建一个名为`Bullet`的新类，它有一个顶点，并将被绘制成一个点。注意，我们还声明并初始化了一个`inFlight`布尔值。

```java
public class Bullet extends GameObject {

  private boolean inFlight = false;

  public Bullet(float shipX, float shipY) {
       super();

       setType(Type.BULLET);

       setWorldLocation(shipX, shipY);

       // Define the bullet
       // as a single point
       // in exactly the coordinates as its world location
       float[] bulletVertices = new float[]{

                0,
                0,
                0

       };

    setVertices(bulletVertices);

}
```

接下来，我们有一个`shoot`方法，它将子弹的`facingAngle`设置为飞船的`facingAngle`。这将导致子弹在按下开火按钮时沿飞船面向的方向移动。我们还设置`inFlight`为真，并查看在`update`方法中是如何使用它的。最后，我们将速度设置为`300`。

我们还添加了一个`resetBullet`方法，它将子弹设置在飞船内部并取消其速度和速度。这让我们对如何实现我们的子弹有了线索。子弹将在飞船内部不可见，直到它们被发射。

```java
public void shoot(float shipFacingAngle){

     setFacingAngle(shipFacingAngle);
     inFlight = true;
     setSpeed (300);
}

public void resetBullet(PointF shipLocation){

     // Stop moving if bullet out of bounds
     inFlight = false;
     setxVelocity(0);
     setyVelocity(0);
     setSpeed(0);
     setWorldLocation(shipLocation.x, shipLocation.y);

}

public boolean isInFlight(){
  return  inFlight;
}
```

现在，我们根据子弹的`facingAngle`和速度移动子弹，但只有当`inFlight`为真时。否则，我们将子弹保留在飞船内部。然后，我们调用`move()`。

```java
public void update(long fps, PointF shipLocation){
        // Set the velocity if bullet in flight
        if(inFlight){
            setxVelocity((float)(getSpeed()* 
               Math.cos(Math.toRadians(getFacingAngle() + 90))));
            setyVelocity((float)(getSpeed()* 
               Math.sin(Math.toRadians(getFacingAngle() + 90))));
        }else{
            // Have it sit inside the ship
            setWorldLocation(shipLocation.x, shipLocation.y);
        }

        move(fps);
    }
}
```

现在，我们有一个`Bullet`类，可以在`GameManager`类中声明一个数组，用来保存这一类型的多个对象。

```java
int numStars = 200;
Bullet [] bullets;
int numBullets = 20;

```

在`createObjects()`中初始化它们，就在`AsteroidsRenderer`中上一节星星之后。注意我们是如何将它们在游戏世界中的位置初始化为飞船的中心。

```java
// Some bullets
gm.bullets = new Bullet[gm.numBullets];
for (int i = 0; i < gm.numBullets; i++) {
  gm.bullets[i] = new Bullet(
     gm.ship.getWorldLocation().x,
     gm.ship.getWorldLocation().y);
}
```

在`update`方法中更新它们，就在我们的闪烁星星之后。

```java
// Update all the bullets
for (int i = 0; i < gm.numBullets; i++) {

    // If not in flight they will need the ships location
    gm.bullets[i].update(fps, gm.ship.getWorldLocation());

}
```

在`draw`方法中绘制它们，再次在星星之后。

```java
for (int i = 0; i < gm.numBullets; i++) {
  gm.bullets[i].draw(viewportMatrix);
}
```

子弹现在已准备好发射！

我们将添加一个`SoundManager`和`InputController`类，然后我们可以看到我们的飞船及其快速开火枪支的行动。

# 重用现有类

让我们快速将`SoundManager`和`InputController`类添加到这个项目中，因为它们只需要稍作调整就能满足我们这里的需求。

在`AsteroidsView`和`AsteroidsRenderer`类中都添加一个`SoundManager`和一个`InputController`对象的成员。

```java
private InputController ic;
private SoundManager sm;
```

在`AsteroidsView`类的`onCreate`方法中初始化新对象，并像这样调用`loadSound`方法：

```java
public AsteroidsView(Context context, int screenX, int screenY) {
  super(context);

 sm = new SoundManager();
 sm.loadSound(context);
 ic = new InputController(screenX, screenY);
     gm = new GameManager(screenX, screenY);
```

同样在`AsteroidsView`中，向`AsteroidsRenderer`构造函数的调用中添加两个额外的参数，以传递对`SoundManager`和`InputController`对象的引用。

```java
setEGLContextClientVersion(2);
setRenderer(new AsteroidsRenderer(gm,sm,ic));

```

现在，在`AsteroidsRenderer`构造函数中添加两个额外的参数，并像这样初始化两个新成员：

```java
public AsteroidsRenderer(GameManager gameManager,
 SoundManager soundManager, InputController inputController) {

        gm = gameManager;
 sm = soundManager;
 ic = inputController;

       handyPointF = new PointF();
       handyPointF2 = new PointF();

}
```

在我们添加这两个类之前，你的 IDE 中会有错误。我们现在就来做这件事。

## 添加`SoundManager`类

`SoundManager`类的工作方式与上一个项目完全一样，所以这里没有什么新内容需要解释。

将下载包`Chapter10/assets`文件夹中的所有声音文件添加到项目的 assets 文件夹中。与最后两个项目一样，你可能需要在项目的`.../app/src/main`文件夹中创建 assets 文件夹。

### 提示

与往常一样，你可以使用提供的声音效果，或者创建自己的效果。

现在，向项目中添加一个名为`SoundManager`的新类。请注意，该类的功能与上一个项目完全相同，但代码不同仅仅是因为声音文件和相关变量的名称不同。将以下代码添加到`SoundManager`类中：

```java
public class SoundManager {
    private SoundPool soundPool;
    private int shoot = -1;
    private int thrust = -1;
    private int explode = -1;
    private int shipexplode = -1;
    private int ricochet = -1;
    private int blip = -1;
    private int nextlevel = -1;
    private int gameover = -1;

    public void loadSound(Context context){
        soundPool = new SoundPool(10, AudioManager.STREAM_MUSIC,0);
        try{
            //Create objects of the 2 required classes
            AssetManager assetManager = context.getAssets();
            AssetFileDescriptor descriptor;

            //create our fx
            descriptor = assetManager.openFd("shoot.ogg");
            shoot = soundPool.load(descriptor, 0);

            descriptor = assetManager.openFd("thrust.ogg");
            thrust = soundPool.load(descriptor, 0);

            descriptor = assetManager.openFd("explode.ogg");
            explode = soundPool.load(descriptor, 0);

            descriptor = assetManager.openFd("shipexplode.ogg");
            shipexplode = soundPool.load(descriptor, 0);

            descriptor = assetManager.openFd("ricochet.ogg");
            ricochet = soundPool.load(descriptor, 0);

            descriptor = assetManager.openFd("blip.ogg");
            blip = soundPool.load(descriptor, 0);

            descriptor = assetManager.openFd("nextlevel.ogg");
            nextlevel = soundPool.load(descriptor, 0);

            descriptor = assetManager.openFd("gameover.ogg");
            gameover = soundPool.load(descriptor, 0);

        }catch(IOException e){
            //Print an error message to the console
            Log.e("error", "failed to load sound files");
        }
    }

    public void playSound(String sound){
        switch (sound){
            case "shoot":
                soundPool.play(shoot, 1, 1, 0, 0, 1);
                break;

            case "thrust":
                soundPool.play(thrust, 1, 1, 0, 0, 1);
                break;

            case "explode":
                soundPool.play(explode, 1, 1, 0, 0, 1);
                break;

            case "shipexplode":
                soundPool.play(shipexplode, 1, 1, 0, 0, 1);
                break;

            case "ricochet":
                soundPool.play(ricochet, 1, 1, 0, 0, 1);
                break;

            case "blip":
                soundPool.play(blip, 1, 1, 0, 0, 1);
                break;

            case "nextlevel":
                soundPool.play(nextlevel, 1, 1, 0, 0, 1);
                break;

            case "gameover":
                soundPool.play(gameover, 1, 1, 0, 0, 1);
                break;

        }

    }
}
```

我们现在可以从任何有对新类引用的地方调用`playSound()`。

## 添加`InputController`类

这与上一个项目中的处理方式相同，只是我们调用适当的`PlayerShip`方法，而不是 Bob 的。此外，当游戏暂停时，我们不会移动视口，因此无需在游戏暂停时以不同的方式处理屏幕触摸；这使得这个`InputController`更简单，更短。

在`AsteroidsView`类中添加`onTouchEvent`方法，以将处理触摸的责任传递给`InputController`：

```java
@Override
    public boolean onTouchEvent(MotionEvent motionEvent) {
        ic.handleInput(motionEvent, gm, sm);
        return true;
    }
```

添加一个名为`InputController`的新类，并添加以下代码，这些代码很直观，除了我们处理玩家发射子弹的方式。

我们声明一个成员`int currentBullet`，用于跟踪我们将要发射的下一个子弹，来自我们即将声明的数组。然后，当按下开火按钮时，我们可以计算子弹数量，并在数组中的最后一个子弹发射后回到第一个子弹。

创建一个名为`InputController`的新类，并输入以下代码：

```java
public class InputController {

    private int currentBullet;

    Rect left;
    Rect right;
    Rect thrust;
    Rect shoot;
    Rect pause;

    InputController(int screenWidth, int screenHeight) {

        //Configure the player buttons
        int buttonWidth = screenWidth / 8;
        int buttonHeight = screenHeight / 7;
        int buttonPadding = screenWidth / 80;

        left = new Rect(buttonPadding,
            screenHeight - buttonHeight - buttonPadding,
            buttonWidth,
            screenHeight - buttonPadding);

        right = new Rect(buttonWidth + buttonPadding,
            screenHeight - buttonHeight - buttonPadding,
            buttonWidth + buttonPadding + buttonWidth,
            screenHeight - buttonPadding);

        thrust = new Rect(screenWidth - buttonWidth - 
            buttonPadding,
            screenHeight - buttonHeight - buttonPadding - 
            buttonHeight - buttonPadding,
            screenWidth - buttonPadding,
            screenHeight - buttonPadding - buttonHeight - 
            buttonPadding);

        shoot = new Rect(screenWidth - buttonWidth - 
            buttonPadding,
            screenHeight - buttonHeight - buttonPadding,
            screenWidth - buttonPadding,
            screenHeight - buttonPadding);

        pause = new Rect(screenWidth - buttonPadding - 
            buttonWidth,
            buttonPadding,
            screenWidth - buttonPadding,
            buttonPadding + buttonHeight);
```

让我们将所有按钮捆绑在一个列表中，并通过一个公共方法使它们可用。

```java
    }    
    public ArrayList getButtons(){

        //create an array of buttons for the draw method
        ArrayList<Rect> currentButtonList = new ArrayList<>();
        currentButtonList.add(left);
        currentButtonList.add(right);
        currentButtonList.add(thrust);
        currentButtonList.add(shoot);
        currentButtonList.add(pause);
        return  currentButtonList;
    }
```

接下来，我们像以前一样处理输入，只是调用我们的`Ship`类的方法。

```java
public void handleInput(MotionEvent motionEvent,GameManager l,                                      
  SoundManager sound){

        int pointerCount = motionEvent.getPointerCount();

        for (int i = 0; i < pointerCount; i++) {
        int x = (int) motionEvent.getX(i);
        int y = (int) motionEvent.getY(i);

          switch (motionEvent.getAction() & 
             MotionEvent.ACTION_MASK) {

            case MotionEvent.ACTION_DOWN:
                    if (right.contains(x, y)) {
                    l.ship.setPressingRight(true);
                    l.ship.setPressingLeft(false);
                 } else if (left.contains(x, y)) {
                    l.ship.setPressingLeft(true);
                    l.ship.setPressingRight(false);
                    } else if (thrust.contains(x, y)) {
                    l.ship.toggleThrust();
                    } else if (shoot.contains(x, y)) {
                        if (l.ship.pullTrigger()) {
                        l.bullets[currentBullet].shoot
                                (l.ship.getFacingAngle());

                            currentBullet++;
                       // If we are on the last bullet restart
                       // from the first one again
                       if(currentBullet == l.numBullets){
                            currentBullet = 0;
                        }

                           sound.playSound("shoot");
                    }

                    } else if (pause.contains(x, y)) {
                    l.switchPlayingStatus();
                    }
                    break;

            case MotionEvent.ACTION_UP:
            if (right.contains(x, y)) {
                    l.ship.setPressingRight(false);
                } else if (left.contains(x, y)) {
                    l.ship.setPressingLeft(false);
                }

                break;

            case MotionEvent.ACTION_POINTER_DOWN:
            if (right.contains(x, y)) {
                    l.ship.setPressingRight(true);
                    l.ship.setPressingLeft(false);
                } else if (left.contains(x, y)) {
                    l.ship.setPressingLeft(true);
                 l.ship.setPressingRight(false);
                } else if (thrust.contains(x, y)) {
                    l.ship.toggleThrust();
                } else if (shoot.contains(x, y)) {
                    if (l.ship.pullTrigger()) {
                    l.bullets[currentBullet].shoot
                            (l.ship.getFacingAngle());

                        currentBullet++;
                    // If we are on the last bullet restart
                    // from the first one again
                    if(currentBullet == l.numBullets){
                        currentBullet = 0;
                    }
                    sound.playSound("shoot");
                    }
                } else if (pause.contains(x, y)) {
                    l.switchPlayingStatus();
                }
                break;

            case MotionEvent.ACTION_POINTER_UP:
            if (right.contains(x, y)) {
                    l.ship.setPressingRight(false);
                } else if (left.contains(x, y)) {
                    l.ship.setPressingLeft(false);
                }

                break;
            }
         }

    }
}
```

现在，我们可以四处飞行并发射几轮太空子弹！当然，在绘制本章后面的 HUD 之前，您将不得不估计屏幕位置。别忘了玩家需要首先点击暂停按钮（右上角）。

### 注意

请注意，目前我们不使用`resetBullet`方法，一旦您发射了二十颗子弹，您将无法再射击。我们可以快速检查子弹是否位于边界外，然后调用`resetBullet`，但我们将与所有的碰撞检测一起，在下一章中完全处理这个问题。

当然，没有行星的话，我们不能有一个行星游戏。

# 绘制和移动行星

最后，我们将添加酷炫的旋转行星。首先，我们将看看与其他游戏对象构造函数相当相似的构造函数，不同之处在于我们随机设置世界位置。但是，需要特别小心，不要在游戏开始的太空船中心位置生成它们。

创建一个名为`Asteroid`的新类，并添加这个构造函数。注意我们没有定义任何顶点。我们将这个任务委托给即将看到的`generatePoints`方法。

```java
public class Asteroid extends GameObject{

    PointF[] points;

    public Asteroid(int levelNumber, int mapWidth, int mapHeight){
        super();

        // set a random rotation rate in degrees per second
        Random r = new Random();
        setRotationRate(r.nextInt(50 * levelNumber) + 10);

        // travel at any random angle
        setTravellingAngle(r.nextInt(360));

        // Spawn asteroids between 50 and 550 on x and y
        // And avoid the extreme edges of map
        int x = r.nextInt(mapWidth - 100)+50;
        int y = r.nextInt(mapHeight - 100)+50;

        // Avoid the center where the player spawns
        if(x > 250 && x < 350){ x = x + 100;}
        if(y > 250 && y < 350){ y = y + 100;}

        // Set the location
        setWorldLocation(x,y);

        // Make them a random speed with the maximum
        // being appropriate to the level number
        setSpeed(r.nextInt(25 * levelNumber)+1);

        setMaxSpeed(140);

        // Cap the speed
        if (getSpeed() > getMaxSpeed()){
            setSpeed(getMaxSpeed());
        }

        // Make sure we know this object is a ship
        setType(Type.ASTEROID);

        // Define a random asteroid shape
        // Then call the parent setVertices()
        generatePoints();

    }
```

我们的更新方法仅根据速度和移动角度计算速度，就像我们对`SpaceShip`类所做的那样。然后以常规方式调用`move()`。

```java
public void update(float fps){

  setxVelocity ((float) (getSpeed() * Math.cos(Math.toRadians  (getTravellingAngle() + 90))));

  setyVelocity ((float) (getSpeed() * Math.sin(Math.toRadians(getTravellingAngle() + 90))));

     move(fps);

}
```

在这里我们看到`generatePoints`方法，它将创建一个随机形状的行星。简单来说，每个行星都有六个顶点。每个顶点都有一个随机生成的位置，但限制相当严格，这样我们就不会得到任何重叠的线条。

```java
// Create a random asteroid shape
public void generatePoints(){
  points = new PointF[7];

   Random r = new Random();
   int i;

     // First a point roughly centre below 0
     points[0] = new PointF();
     i = (r.nextInt(10))+1;
     if(i % 2 == 0){i = -i;}
     points[0].x = i;
     i = -(r.nextInt(20)+5);
     points[0].y = i;

     // Now a point still below centre but to the right and up a bit
     points[1] = new PointF();
     i = r.nextInt(14)+11;
     points[1].x = i;
     i = -(r.nextInt(12)+1);
     points[1].y =  i;

     // Above 0 to the right
     points[2] = new PointF();
     i = r.nextInt(14)+11;
     points[1].x = i;
     i = r.nextInt(12)+1;
     points[2].y = i;

     // A point roughly centre above 0
     points[3] = new PointF();
     i = (r.nextInt(10))+1;
     if(i % 2 == 0){i = -i;}
     points[3].x = i;
     i = r.nextInt(20)+5;
     points[3].y =  i;

     // left above 0
     points[4] = new PointF();
     i = -(r.nextInt(14)+11);
     points[4].x = i;
     i = r.nextInt(12)+1;
     points[4].y = i ;

     // left below 0
     points[5] = new PointF();
     i = -(r.nextInt(14)+11);
     points[5].x =  i;
     i = -(r.nextInt(12)+1);

     points[5].y = i;
```

现在，我们有六个点用来构建表示顶点的浮点数数组。最后，我们调用`setVertices()`来创建我们的`ByteBuffer`。请注意，行星将被绘制成一系列的线条，这就是数组中的最后一个顶点与第一个顶点相同的原因。

```java
  // Now use these points to draw our asteroid
  float[] asteroidVertices = new float[]{
     // First point to second point
     points[0].x, points[0].y, 0,
     points[1].x, points[1].y, 0,

     // 2nd to 3rd
     points[1].x, points[1].y, 0,
     points[2].x, points[2].y, 0,

     // 3 to 4
     points[2].x, points[2].y, 0,
     points[3].x, points[3].y, 0,

     // 4 to 5
     points[3].x, points[3].y, 0,
     points[4].x, points[4].y, 0,

     // 5 to 6
     points[4].x, points[4].y, 0,
     points[5].x, points[5].y, 0,

     // 6 back to 1
     points[5].x, points[5].y, 0,
     points[0].x, points[0].y, 0,
};

setVertices(asteroidVertices);

}// End method

}// End class
```

如您所料，我们在`GameManager`中添加了一个数组来保存所有的行星。同时，我们还将声明一些变量，用来记录玩家当前的关卡以及初始（基础）的行星数量。随后，当我们初始化所有行星时，我们将看到如何确定需要摧毁的行星数量以完成一个关卡。

```java
Asteroid [] asteroids;
int numAsteroids;
int numAsteroidsRemaining;
int baseNumAsteroids = 10;
int levelNumber = 1;
```

在`GameManager`构造函数中初始化数组：

```java
// For all our asteroids
asteroids = new Asteroid[500];
```

在`createObjects`方法中使用我们之前声明的变量来初始化对象本身，根据当前关卡确定行星的数量。

```java
// Determine the number of asteroids
gm.numAsteroids = gm.baseNumAsteroids * gm.levelNumber;
// Set how many asteroids need to be destroyed by player
gm.numAsteroidsRemaining = gm.numAsteroids;
// Spawn the asteroids

for (int i = 0; i < gm.numAsteroids * gm.levelNumber; i++) {
     // Create a new asteroid
     // Pass in level number so they can be made
     // appropriately dangerous.
     gm.asteroids[i] = new Asteroid
      (gm.levelNumber, gm.mapWidth, gm.mapHeight);

}
```

在`update`方法中更新它们。

```java
// Update all the asteroids
for (int i = 0; i < gm.numAsteroids; i++) {
  if (gm.asteroids[i].isActive()) {
    gm.asteroids[i].update(fps);
  }
}
```

最后，我们在`draw`方法中绘制所有的行星。

```java
// The bullets
for (int i = 0; i < gm.numBullets; i++) {
  gm.bullets[i].draw(viewportMatrix);
}

for (int i = 0; i < gm.numAsteroids; i++) {
 if (gm.asteroids[i].isActive()) {
 gm.asteroids[i].draw(viewportMatrix);
 }

}

```

现在，运行游戏并查看那些流畅的 60+ FPS 旋转行星。

![绘制和移动行星](img/B043422_10_03.jpg)

现在，我们需要通过添加按钮图像以及一些其他覆盖信息，包括 HUD，来使控制飞船变得容易。

# 分数和 HUD（头上显示装置）

HUD 对象永远不会被旋转。另外，它们是在`InputController`类中根据屏幕坐标定义的，而不是游戏世界或甚至是 OpenGL 坐标。因此，我们的`GameObject`类不是一个合适的父类。

为了简单起见，这三个 HUD 类将各自拥有自己的`draw`方法。我们将看到如何使用新的视口矩阵以一致的大小和屏幕位置绘制它们。

创建了我们所有的 HUD 类之后，我们将添加所有的对象声明、初始化和绘制代码。

## 添加控制按钮

我们将为第一个 HUD 对象创建一个类，这是一个简单的按钮。

### 注意

我明确地展示了所有的导入语句，因为它们不会自动导入。请注意，接下来的两个类也需要这些。代码像往常一样包含在下载包中，如果你希望直接复制粘贴。

创建一个新类，将其命名为`GameButton`，然后添加以下导入语句。请确保根据你使用的章节代码或你给项目命的名声明正确的包名。

```java
import android.graphics.PointF;
import java.nio.ByteBuffer;
import java.nio.ByteOrder;
import java.nio.FloatBuffer;
import static android.opengl.GLES20.GL_FLOAT;
import static android.opengl.GLES20.GL_LINES;
import static android.opengl.GLES20.glDrawArrays;
import static android.opengl.GLES20.glEnableVertexAttribArray;
import static android.opengl.GLES20.glGetAttribLocation;
import static android.opengl.GLES20.glGetUniformLocation;
import static android.opengl.GLES20.glUniform4f;
import static android.opengl.GLES20.glUniformMatrix4fv;
import static android.opengl.GLES20.glUseProgram;
import static android.opengl.Matrix.orthoM;
import static android.opengl.GLES20.glVertexAttribPointer;
import static com.gamecodeschool.c10asteroids.GLManager.A_POSITION;
import static com.gamecodeschool.c10asteroids.GLManager.COMPONENTS_PER_VERTEX;
import static com.gamecodeschool.c10asteroids.GLManager.FLOAT_SIZE;
import static com.gamecodeschool.c10asteroids.GLManager.STRIDE;
import static com.gamecodeschool.c10asteroids.GLManager.U_COLOR;
import static com.gamecodeschool.c10asteroids.GLManager.U_MATRIX;
```

首先，我们声明一些成员；`viewportMatrix`，我们将把来自`InputController`类的基于屏幕坐标的视口变换的新矩阵放入其中——一个整型`glprogram`值，一个`int numVertices`值，以及一个`FloatBuffer`类。

```java
public class GameButton {

    // For button coordinate
    // into a GL space coordinate (-1,-1 to 1,1)
    // for drawing on the screen
    private final float[] viewportMatrix = new float[16];

    // A handle to the GL glProgram -
    // the compiled and linked shaders
    private static int glProgram;

    // How many vertices does it take to make
    // our button
    private int numVertices;

    // This will hold our vertex data that is
    // passed into openGL glProgram
    private FloatBuffer vertices;
```

在构造函数中我们首先通过调用`orthoM()`并传入屏幕的高度和宽度作为`0,0`来创建我们的视口矩阵。这使得 OpenGL 将一个与设备分辨率相同的坐标范围映射到 OpenGL 坐标范围之上。

然后，我们获取传入按钮的坐标并将其缩小以使其变小。然后，我们初始化一个顶点数组作为四条线来表示一个按钮。显然，我们将需要创建一个新的按钮对象来代表`InputController`类中的每个按钮。

```java
public GameButton(int top, int left, 
    int bottom, int right, GameManager gm){

    //The HUD needs its own viewport
    // notice we set the screen height in pixels as the
    // starting y coordinates because
    // OpenGL is upside down world :-)
    orthoM(viewportMatrix, 0, 0, 
        gm.screenWidth, gm.screenHeight, 0, 0, 1f);

        // Shrink the button visuals to make
        // them less obtrusive while leaving
        // the screen area they represent the same.
        int width = (right - left) / 2;
        int height = (top - bottom) / 2;
        left = left + width / 2;
        right = right - width / 2;
        top = top - height / 2;
        bottom = bottom + height / 2;

        PointF p1 = new PointF();
        p1.x = left;
        p1.y = top;

        PointF p2 = new PointF();
        p2.x = right;
        p2.y = top;

        PointF p3 = new PointF();
        p3.x = right;
        p3.y = bottom;

        PointF p4 = new PointF();
        p4.x = left;
        p4.y = bottom;

        // Add the four points to an array of vertices
        // This time, because we don't need to animate the border
        // we can just declare the world space coordinates, the
        // same as above.
        float[] modelVertices = new float[]{
                // A line from point 1 to point 2
                p1.x, p1.y, 0,
                p2.x, p2.y, 0,
                // Point 2 to point 3
                p2.x, p2.y, 0,
                p3.x, p3.y, 0,
                // Point 3 to point 4
                p3.x, p3.y, 0,
                p4.x, p4.y, 0,
                // Point 4 to point 1
                p4.x, p4.y, 0,
                p1.x, p1.y, 0
        };
```

现在，我们从`GameObject`复制了一些代码来准备`ByteBuffer`，但我们仍然使用我们的静态`GLManager.getGLProgram()`来获取 GL 程序的句柄。

```java
       // Store how many vertices and 
       // elements there is for future use
       final int ELEMENTS_PER_VERTEX = 3;// x,y,z
       int numElements = modelVertices.length;
       numVertices = numElements/ELEMENTS_PER_VERTEX;

       // Initialize the vertices ByteBuffer object based on the
       // number of vertices in the button and the number of
       // bytes there are in the float type
       vertices = ByteBuffer.allocateDirect(
                numElements
                * FLOAT_SIZE)
                .order(ByteOrder.nativeOrder()).asFloatBuffer();

       // Add the button into the ByteBuffer object
       vertices.put(modelVertices);

       glProgram = GLManager.getGLProgram();

}
```

最后，我们实现了`draw`方法，这是来自`GameObject`的`draw`方法的简化版本。注意我们不需要处理模型、转换和旋转矩阵，并且我们传递了一个不同的颜色给片段着色器。

```java
public void draw(){

    // And tell OpenGl to use the glProgram
    glUseProgram(glProgram);

    // Now we have a glProgram we need the locations
    // of our three GLSL variables
    int uMatrixLocation = glGetUniformLocation(glProgram, U_MATRIX);

    int aPositionLocation = 
        glGetAttribLocation(glProgram, A_POSITION);

    int uColorLocation = glGetUniformLocation(glProgram, U_COLOR);

    vertices.position(0);

    glVertexAttribPointer(
        aPositionLocation,
        COMPONENTS_PER_VERTEX,
        GL_FLOAT,
        false,
        STRIDE,
        vertices);

    glEnableVertexAttribArray(aPositionLocation);

    // give the new matrix to OpenGL
    glUniformMatrix4fv(uMatrixLocation, 1, false, viewportMatrix, 0);

    // Assign a different color to the fragment shader
    glUniform4f(uColorLocation, 0.0f, 0.0f, 1.0f, 1.0f);

    // Draw the lines
    // start at the first element of the
    // vertices array and read in all vertices
    glDrawArrays(GL_LINES, 0, numVertices);

}
}// End class
```

## 计数字符

这个类与`GameButton`相同，不同之处在于计数字符将是一个单一的垂直直线；因此，我们只需要两个顶点。

但是请注意，我们在构造函数中有一个名为`nthIcon`的参数。调用代码需要负责让`TallyIcon`知道已经创建的`TallyIcon`对象的总数量加一。然后，当前的`TallyIcon`对象可以使用内边距变量来适当定位自己。

创建一个名为 `TallyIcon` 的新类，并输入以下代码。像之前一样，根据需要包含静态导入。以下是所有声明和构造函数的代码：

```java
public class TallyIcon {

    // For button coordinate
    // into a GL space coordinate (-1,-1 to 1,1)
    // for drawing on the screen
    private final float[] viewportMatrix = new float[16];

    // A handle to the GL glProgram -
    // the compiled and linked shaders
    private static int glProgram;

    // How many vertices does it take to make
    // our button
    private int numVertices;

    // This will hold our vertex data that is
    // passed into openGL glProgram
    //private final FloatBuffer vertices;
    private FloatBuffer vertices;

    public TallyIcon(GameManager gm, int nthIcon){

        // The HUD needs its own viewport
        // notice we set the screen height in pixels as the
        // starting y coordinates because
        // OpenGL is upside down world :-)
        orthoM(viewportMatrix, 0, 0,
          gm.screenWidth, gm.screenHeight, 0, 0f, 1f);

        float padding = gm.screenWidth / 160;
        float iconHeight = gm.screenHeight / 15;
        float iconWidth = 1; // square icons
        float startX = 10 + (padding + iconWidth)* nthIcon;
        float startY = iconHeight * 2 + padding;

        PointF p1 = new PointF();
        p1.x = startX;
        p1.y = startY;

        PointF p2 = new PointF();
        p2.x = startX;
        p2.y = startY - iconHeight;

        // Add the four points to an array of vertices
        // This time, because we don't need to animate the border
        // we can just declare the world space coordinates, the
        // same as above.
        float[] modelVertices = new float[]{
                // A line from point 1 to point 2
                p1.x, p1.y, 0,
                p2.x, p2.y, 0,

        };

        // Store how many vertices and 
        //elements there is for future use
        final int ELEMENTS_PER_VERTEX = 3;// x,y,z
        int numElements = modelVertices.length;
        numVertices = numElements/ELEMENTS_PER_VERTEX;

        // Initialize the vertices ByteBuffer object based on the
        // number of vertices in the button and the number of
        // bytes there are in the float type
        vertices = ByteBuffer.allocateDirect(
                numElements
                * FLOAT_SIZE)
                .order(ByteOrder.nativeOrder()).asFloatBuffer();

        // Add the button into the ByteBuffer object
        vertices.put(modelVertices);

        glProgram = GLManager.getGLProgram();
    }
```

这就是 draw 方法，现在看起来可能相当熟悉了。

```java
    public void draw(){

        // And tell OpenGl to use the glProgram
        glUseProgram(glProgram);

        // Now we have a glProgram we need the locations
        // of our three GLSL variables
        int uMatrixLocation = 
        glGetUniformLocation(glProgram, U_MATRIX);

        int aPositionLocation = 
        glGetAttribLocation(glProgram, A_POSITION);

        int uColorLocation = 
        glGetUniformLocation(glProgram, U_COLOR);

        vertices.position(0);

        glVertexAttribPointer(
                aPositionLocation,
                COMPONENTS_PER_VERTEX,
                GL_FLOAT,
                false,
                STRIDE,
                vertices);

        glEnableVertexAttribArray(aPositionLocation);

        // Just give the passed in matrix to OpenGL
        glUniformMatrix4fv(uMatrixLocation, 1, 
          false, viewportMatrix, 0);

        // Assign a color to the fragment shader
        glUniform4f(uColorLocation, 1.0f, 1.0f, 0.0f, 1.0f);

        // Draw the lines
        // start at the first element of the vertices array and read in all vertices
        glDrawArrays(GL_LINES, 0, numVertices);
    }
```

现在是最后的 HUD 元素。

## 生命图标

我们最后的图标将是一种迷你飞船，用来指示玩家还剩下多少生命。

我们将使用线条构建一个三角形形状，以创建一个漂亮的空心效果。请注意，`LifeIcon` 构造函数还使用 `nthIcon` 元素来控制填充和屏幕上的位置。

创建一个名为 `LifeIcon` 的新类，并输入以下代码，记住所有不会自动导入的导入语句。以下是声明和构造函数：

```java
public class LifeIcon {

     // Remember the static import for GLManager

     // For button coordinate
     // into a GL space coordinate (-1,-1 to 1,1)
     // for drawing on the screen
     private final float[] viewportMatrix = new float[16];

     // A handle to the GL glProgram -
     // the compiled and linked shaders
     private static int glProgram;

     // Each of the above constants also has a matching int
     // which will represent its location in the open GL glProgram
     // In GameButton they are declared as local variables

     // How many vertices does it take to make
     // our button
     private int numVertices;

     // This will hold our vertex data that is
     // passed into openGL glProgram
     //private final FloatBuffer vertices;
     private FloatBuffer vertices;

     public LifeIcon(GameManager gm, int nthIcon){

     // The HUD needs its own viewport
     // notice we set the screen height in pixels as the
     // starting y coordinates because
     // OpenGL is upside down world :-)
     orthoM(viewportMatrix, 0, 0,
       gm.screenWidth, gm.screenHeight, 0, 0f, 1f);

     float padding = gm.screenWidth / 160;
     float iconHeight = gm.screenHeight / 15;
     float iconWidth = gm.screenWidth / 30;
     float startX = 10 + (padding + iconWidth)* nthIcon;
     float startY = iconHeight;

     PointF p1 = new PointF();
     p1.x = startX;
     p1.y = startY;

     PointF p2 = new PointF();
     p2.x = startX + iconWidth;
     p2.y = startY;

     PointF p3 = new PointF();
     p3.x = startX + iconWidth/2;
     p3.y = startY - iconHeight;

     // Add the four points to an array of vertices
     // This time, because we don't need to animate the border
     // we can just declare the world space coordinates, the
     // same as above.
     float[] modelVertices = new float[]{
               // A line from point 1 to point 2
               p1.x, p1.y, 0,
               p2.x, p2.y, 0,
               // Point 2 to point 3
               p2.x, p2.y, 0,
               p3.x, p3.y, 0,
               // Point 3 to point 1
               p3.x, p3.y, 0,
               p1.x, p1.y, 0,

  };

     // Store how many vertices and elements there is for future 
     // use
     final int ELEMENTS_PER_VERTEX = 3;// x,y,z
     int numElements = modelVertices.length;
     numVertices = numElements/ELEMENTS_PER_VERTEX;

     // Initialize the vertices ByteBuffer object based on the
     // number of vertices in the button and the number of
     // bytes there are in the float type
     vertices = ByteBuffer.allocateDirect(
              numElements
              * FLOAT_SIZE)
              .order(ByteOrder.nativeOrder()).asFloatBuffer();

     // Add the button into the ByteBuffer object
     vertices.put(modelVertices);

       glProgram = GLManager.getGLProgram();
     }
```

这是 `LifeIcon` 类的 `draw` 方法：

```java
    public void draw(){

            // And tell OpenGl to use the glProgram
            glUseProgram(glProgram);

            // Now we have a glProgram we need the locations
            // of our three GLSL variables
            int uMatrixLocation = glGetUniformLocation 
              (glProgram, U_MATRIX);
            int aPositionLocation = glGetAttribLocation 
              (glProgram, A_POSITION);
            int uColorLocation = glGetUniformLocation 
               (glProgram, U_COLOR);

            vertices.position(0);

            glVertexAttribPointer(
                    aPositionLocation,
                    COMPONENTS_PER_VERTEX,
                    GL_FLOAT,
                    false,
                    STRIDE,
                    vertices);

            glEnableVertexAttribArray(aPositionLocation);

            // Just give the passed in matrix to OpenGL
            glUniformMatrix4fv(uMatrixLocation, 1, 
              false, viewportMatrix, 0);
            // Assign a color to the fragment shader
            glUniform4f(uColorLocation, 1.0f, 
              1.0f, 0.0f, 1.0f);
            // Draw the lines
            // start at the first element of 
            // the vertices array and read in all vertices
            glDrawArrays(GL_LINES, 0, numVertices);
        }

}
```

我们已经有了三个 HUD 类，并且可以将它们绘制到屏幕上。

### 声明、初始化并绘制 HUD 对象

我们将像所有 `GameObject` 类一样声明、初始化并绘制我们的 HUD 对象。但是请注意，如预期的那样，我们不向 `draw` 方法传递视口矩阵，因为 HUD 类提供了自己的视口矩阵。

向 `GameManager` 添加这些成员：

```java
TallyIcon[] tallyIcons;
int numLives = 3;
LifeIcon[] lifeIcons;
```

与我们对 `asteroids` 数组的操作一样，在 `GameManager` 构造函数中初始化 `tallyIcons` 和 `lifeIcons`：

```java
lifeIcons = new LifeIcon[50];
tallyIcons = new TallyIcon[500];
```

向 `AsteroidsRenderer` 类添加一个新的成员数组：

```java
// This will hold our game buttons
private final GameButton[] gameButtons = new GameButton[5];
```

添加这段代码以创建我们所有新 HUD 类的对象。将其添加到 `createObjects` 方法中的闭合大括号之前：

```java
// Now for the HUD objects
// First the life icons
for(int i = 0; i < gm.numLives; i++) {
    // Notice we send in which icon this represents
    // from left to right so padding and positioning is correct.
    gm.lifeIcons[i] = new LifeIcon(gm, i);
}

// Now the tally icons (1 at the start)
for(int i = 0; i < gm.numAsteroidsRemaining; i++) {
    // Notice we send in which icon this represents
    // from left to right so padding and positioning is correct.
    gm.tallyIcons[i] = new TallyIcon(gm, i);
}

// Now the buttons
ArrayList<Rect> buttonsToDraw = ic.getButtons();
int i = 0;
for (Rect rect : buttonsToDraw) {
    gameButtons[i] = new GameButton(rect.top, rect.left, 
        rect.bottom, rect.right, gm);

    i++;

}
```

现在，我们可以根据剩余的生命次数和升级前剩余的 asteroids 数量来绘制我们的 HUD。将此代码添加到 `draw` 方法的末尾：

```java
// the buttons
for (int i = 0; i < gameButtons.length; i++) {
  gameButtons[i].draw();
}

// Draw the life icons
for(int i = 0; i < gm.numLives; i++) {
     // Notice we send in which icon this represents
     // from left to right so padding and positioning is correct.
     gm.lifeIcons[i].draw();
}

// Draw the level icons
for(int i = 0; i < gm.numAsteroidsRemaining; i++) {
  // Notice we send in which icon this represents
  // from left to right so padding and positioning is correct.
  gm.tallyIcons[i].draw();
}
```

现在你可以飞来飞去，欣赏你的新 HUD 了。

![声明、初始化并绘制 HUD 对象](img/B043422_10_04.jpg)

显然，如果我们想要充分利用生命和 asteroid 计数指示器，那么我们首先需要能够射击 asteroid，并在飞船被击中时检测到它们。

# 总结

在本章中我们取得了很大的成就，实际上可以很容易地快速添加更多的游戏对象。也许，可以像原始街机经典游戏中那样偶尔添加一个 UFO。

在下一章中，我们将利用在前一个项目中学习到的内容来设置碰撞检测，并完成游戏。然而，一个拥有精确、清晰、平滑移动线条的游戏，理应比我们至今所使用的更精确的碰撞检测。

因此，我们将专注于实现精确高效的碰撞检测，以使我们的 Asteroids 仿真模拟器得以完善。
