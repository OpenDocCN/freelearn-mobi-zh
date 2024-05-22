# 第三章：Tappy Defender – 飞翔之旅

我们现在准备快速添加许多新对象和一些功能。在本章结束时，我们将非常接近一个可玩的游戏。我们将检测玩家触摸屏幕，这样他就可以控制飞船。我们将在`SpaceShip`类中添加虚拟推进器，以使飞船上下移动并增加速度。

我们将检测安卓设备的分辨率，并利用它来执行诸如防止玩家从屏幕边缘飞出，以及检测我们的敌人何时需要重生等操作。

我们将创建一个新的`EnemyShip`类，它将代表自杀式的敌人。我们还将看到如何轻松生成并控制它们，而无需更改我们代码中控制部分的任何逻辑。

我们将通过添加一个`SpaceDust`类并生成数十个它们来添加滚动效果，使玩家看起来像是在太空中飞速穿梭。

最后，我们将了解并实现碰撞检测，以便我们知道玩家何时被敌人击中，同时也会看看一个图形技巧，以帮助我们在调试碰撞检测代码时。

# 控制飞船

我们让玩家的飞船在屏幕上毫无目的地漂浮，从左边缘和顶部边缘各 50 像素开始，缓缓向右漂移。现在，我们可以让玩家控制飞船。

记住，控制设计是一个单指点击并长按加速，释放后停止加速并减速。

## 检测触摸

我们扩展的用于视图的`SurfaceView`类非常适合处理屏幕触摸。

我们需要做的就是在我们`TDView`类中重写`onTouchEvent`方法。让我们先看看完整的代码，然后我们可以更仔细地检查以确保我们理解正在发生的事情。在`TDView`类中输入此方法，并以通常的方式导入必要的类。我已经突出了我们稍后将自定义的代码部分：

```java
// SurfaceView allows us to handle the onTouchEvent
@Override
public boolean onTouchEvent(MotionEvent motionEvent) {

    // There are many different events in MotionEvent
    // We care about just 2 - for now.
    switch (motionEvent.getAction() & MotionEvent.ACTION_MASK) {

        // Has the player lifted their finger up?
        case MotionEvent.ACTION_UP:
 // Do something here
            break;

        // Has the player touched the screen?
        case MotionEvent.ACTION_DOWN:
 // Do something here
           break;
    }
   return true;
}
```

这是到目前为止`onTouchEvent`方法的工作方式。玩家触摸屏幕；这可以是任何一种接触。它可能是滑动，捏合，多个手指等。一条详细的信息被发送到`onTouchEvent`方法。

事件详细信息包含在`MotionEvent`类参数中，正如我们在代码中所看到的。`MotionEvent`类包含大量数据。它知道有多少个手指放在屏幕上，每个手指的坐标，以及是否还进行了任何手势。 

由于我们实现了一个简单的点击并长按加速，释放停止加速的控制方案；我们可以通过使用`motionEvent.getAction() & MotionEvent.ACTION_MASK`条件简单地切换，只需处理许多可能不同情况中的两种。

`MotionEvent.ACTION_UP:`的情况，顾名思义，会告诉我们在玩家将手指从屏幕上移开时。然后，不出所料，`MotionEvent.ACTION_DOWN:`的情况会告诉我们在玩家将手指放在屏幕上时。

### 注意

通过`MotionEvent`类我们可以了解到的内容非常丰富。何不在这里看看它的全部潜力：[`developer.android.com/reference/android/view/MotionEvent.html`](http://developer.android.com/reference/android/view/MotionEvent.html)。在接下来的项目中，我们也会在第五章《平台游戏——升级游戏引擎》中进一步探索这个类。

## 为飞船添加助推器

现在，我们需要考虑如何使用这些事件来控制飞船。首先，飞船需要知道它是否正在加速。这需要一个布尔成员变量。在`PlayerShip`类的类声明后立即添加以下代码：

```java
private boolean boosting;
```

然后，我们需要在创建`PlayerShip`对象时初始化它。在`PlayerShip`构造函数中添加以下内容：

```java
boosting = false;
```

现在，我们需要让`onTouchEvent`方法在`boosting`的真和假之间切换，以控制飞船的加速和停止加速。在`PlayerShip`类中添加以下方法：

```java
public void setBoosting() {
  boosting = true;
}

public void stopBoosting() {
  boosting = false;
}
```

现在，我们可以从`onTouchEvent`方法中调用这些公共方法，以控制飞船是否正在加速的状态。在`onTouchEvent`方法中添加以下新代码：

```java
// Has the player lifted there finger up?
case MotionEvent.ACTION_UP:
 player.stopBoosting();
  break;

// Has the player touched the screen?
case MotionEvent.ACTION_DOWN:
 player.setBoosting();
  break;
```

现在，我们的视图与模型进行了交流；我们需要做的是根据加速变量的状态让它执行不同的操作。逻辑上，这部分代码应该放在`PlayerShip`类的`update`方法中。

我们将根据飞船当前是否正在加速来改变飞船的`speed`变量。这看起来很简单，但仅仅基于飞船是否加速来增加速度会有一些小问题：

+   一个问题是`update`方法每秒被调用 60 次。因此，不需要太多加速，飞船就会以荒谬的速度飞行。我们需要限制飞船的速度。

+   另一个问题在于，当飞船加速时，它将向屏幕上方移动，而没有任何东西能阻止它直接飞出屏幕顶部，永远消失不见。我们需要将飞船的*x*和*y*坐标限制在屏幕内。

+   当飞船不加速且速度逐渐降为零时，是什么让飞船再次降下来？我们需要一个简单的重力物理模拟。

要解决这三个问题，我们可以在`PlayerShip`类中添加代码。但在我们这样做之前，先简单谈谈游戏平衡。我们很快就会看到的代码使用了不同的整数值，例如，我们将`GRAVITY`初始化为`-12`，将`MAX_SPEED`初始化为`20`。这些数字在现实中没有任何依据！

这些数值只是为了使游戏玩法保持平衡。随意调整这些任意数值，让游戏变得更容易或更难，甚至不可能完成。在第四章《Tappy Defender——回家》的最后，我们将更详细地探讨游戏迭代，并再次审视难度和平衡。

考虑到我们之前提出的三个问题，请在`PlayerShip`类声明后的类声明后立即添加以下成员变量：

```java
private final int GRAVITY = -12;

// Stop ship leaving the screen
private int maxY;
private int minY;

//Limit the bounds of the ship's speed
private final int MIN_SPEED = 1;
private final int MAX_SPEED = 20;
```

现在，我们已经开始了解决我们三个问题的过程，我们可以在`PlayerShip`类的`update`方法中添加代码。我们将删除之前章节中放入的那行代码。那只是为了快速查看我们的飞船的行动。输入我们`PlayerShip`类的`update`方法的新代码。之后我们将更详细地查看：

```java
public void update() {

  // Are we boosting?
  if (boosting) {
    // Speed up
    speed += 2;
  } else {
    // Slow down
    speed -= 5;
  }

  // Constrain top speed
  if (speed > MAX_SPEED) {
    speed = MAX_SPEED;
}

  // Never stop completely
  if (speed < MIN_SPEED) {
    speed = MIN_SPEED;
}

  // move the ship up or down
  y -= speed + GRAVITY;

  // But don't let ship stray off screen
  if (y < minY) {
    y = minY;
  }

  if (y > maxY) {
    y = maxY;
  }

}
```

从之前代码块的顶部开始，我们根据飞船是否在加速，每一帧游戏都在增加或减少速度变量，这些数值看似是任意的。

然后，我们将飞船的速度限制在最大 20 和最小 1 之间，这是我们之前添加的变量所指定的。通过`y -= speed + GRAVITY`这行代码，我们根据速度和重力将屏幕上的图形向上或向下移动。`GRAVITY`和`MAX_SPEED`的看似任意的值很好地让玩家能够笨拙且不稳定地在太空中弹跳。

最后，我们确保飞船图形永远不会超出屏幕，也就是确保飞船图形不会超过`maxY`和`minY`。你可能已经注意到，到目前为止，我们还没有初始化`maxY`和`minY`。此外，由于许多 Android 设备的屏幕分辨率截然不同，我们到底要将它们初始化为多少？

我们需要做的是在运行时发现 Android 设备的分辨率，并使用这些信息来初始化`MaxY`和`minY`。

## 检测屏幕分辨率

我们知道我们需要玩家屏幕的最大*y*坐标。稍后，在项目中添加背景和敌方飞船时，我们会意识到我们也需要最大的*x*坐标。考虑到这一点，让我们看看如何获取这些信息，并将其提供给`PlayerShip`类。

在应用启动时检测屏幕分辨率最为方便，这发生在我们的视图和模型被实例化之前。这意味着我们的`GameActivity`类是进行这一操作的好地方。现在我们将在`GameActivity`类的`onCreate`方法中添加代码。在创建我们的`TDView`对象的`new...`调用之前，将以下新代码添加到`onCreate`类中：

```java
// Get a Display object to access screen details
Display display = getWindowManager().getDefaultDisplay();
// Load the resolution into a Point object
Point size = new Point();
display.getSize(size);
```

之前的代码使用`getWindowManager().getDefaultDisplay();`声明并初始化了`Display`类型的对象。然后我们创建了一个`Point`类型的新对象。`Point`对象可以保存两个坐标，然后我们将其作为参数传递给新`Display`对象的`getSize`方法。

现在，我们已经将我们游戏运行的 Android 设备的分辨率整洁地存储在`size`中。现在，将这个信息传递给需要它的代码部分。首先，我们将改变我们传递给初始化我们的`TDView`对象的`new`调用的参数。按照如下所示更改`new`的调用，将屏幕分辨率传递给`TDView`构造函数：

```java
// Create an instance of our Tappy Defender View
// Also passing in this.
// Also passing in the screen resolution to the constructor
gameView = new TDView(this, size.x, size.y);

```

然后，当然，我们需要更新`TDView`构造函数本身。在`TDView.java`文件中，修改`TDView`构造函数的签名，使得声明现在看起来像这样：

```java
TDView(Context context, int x, int y) {
```

现在，在构造函数中，改变我们初始化`PlayerShip`对象的玩家方式：

```java
player = new PlayerShip(context, x, y);
```

当然，我们现在必须修改`PlayerShip`类本身中的构造函数声明，如下所示：

```java
public PlayerShip(Context context, int screenX, int screenY) {
```

此外，我们现在可以在`PlayerShip`构造函数内初始化我们的`maxY`和`minY`变量。在我们看到代码之前，我们需要确切地考虑这将如何工作。

保存我们太空飞船图形的位图的坐标是在`TDView`类的`draw`方法中传递给`drawBitmap()`的*x = 0*和*y = 0*坐标处绘制的。这意味着在开始绘制飞船的坐标右侧和下方有一些像素。查看下一张图片以可视化这一点：

![检测屏幕分辨率](img/B043422_03_01.jpg)

因此，我们必须考虑到这一点，设置我们的`minY`和`maxY`值。如图所示，位图的顶部像素确实是在船只的*y*位置精确绘制的。这样我们可以确定`minY`应该是零。

然而，船的底部是在*y + 位图的高度*处绘制的。

我们现在可以在构造函数中添加两行代码来初始化这些变量：

```java
maxY = screenY - bitmap.getHeight();
minY = 0;
```

您现在可以运行游戏并测试您的助推器了！

# 构建敌人

既然我们已经实现了点击控制，现在是时候添加一些玩家可以通过助推来躲避的敌人了。

这将比我们添加玩家太空飞船时要简单得多，因为我们所需的大部分内容已经就位。我们只需编写一个类来表示我们的敌人，实例化我们需要的多个敌人对象，调用它们的`update`方法，然后绘制它们。

我们将看到，我们敌人的`update`方法与`PlayerShip`的将大不相同。它需要处理像简单的 AI 飞向玩家等事情。它还需要处理当它离开屏幕时的重生。

## 设计敌人

首先，创建一个新的 Java 类，将其命名为`EnemyShip`。在类内部添加这些成员变量，这样你的新类将如下所示：

```java
public class EnemyShip{
    private Bitmap bitmap;
    private int x, y;
    private int speed = 1;

    // Detect enemies leaving the screen
    private int maxX;
    private int minX;

    // Spawn enemies within screen bounds
    private int maxY;
    private int minY;
}
```

现在，添加一些 getter 和 setter 方法，以便`draw`方法可以访问它需要绘制的内容以及需要绘制的地方。这里没有新的或异常的内容：

```java
//Getters and Setters
public Bitmap getBitmap(){
  return bitmap;
}

public int getX() {
  return x;
}

public int getY() {
  return y;
}
```

## 生成敌人

让我们完整地实现`EnemyShip`构造函数。现在输入代码，然后我们将更仔细地查看：

```java
// Constructor
public EnemyShip(Context context, int screenX, int screenY){
    bitmap = BitmapFactory.decodeResource 
    (context.getResources(), R.drawable.enemy);

  maxX = screenX;
  maxY = screenY;
  minX = 0;
  minY = 0;

  Random generator = new Random();
  speed = generator.nextInt(6)+10;

  x = screenX;
  y = generator.nextInt(maxY) - bitmap.getHeight();
}
```

构造函数的签名与`PlayerShip`类完全相同。一个用于操作`Bitmap`对象的`Context`类以及保存屏幕分辨率的`screenX`和`screenY`。

就像我们对`PlayerShip`类所做的那样，我们将图像加载到`Bitmap`中。当然，我们再次需要将名为`enemy.png`的图像文件添加到项目的`drawable`文件夹中。下载包的`Chapter3/drawable`文件夹中有一个整洁的敌人图形，或者你可以设计自己的图形。对于这个游戏来说，大约 32 x 32 到 256 x 256 之间的任何尺寸都足够了。同样，你的图形也不需要是正方形。我们会看到，我们的游戏引擎在处理不同屏幕尺寸的外观时并不完美，我们将在下一个项目中解决这个问题：

![生成敌人](img/B043422_03_02.jpg)

接下来，我们初始化`maxX`、`maxY`、`minX`和`minY`。尽管敌人只进行水平移动，我们需要`maxY`和`minY`坐标以确保我们以一个合理的高度生成它们。`maxX`坐标将使我们能够将它们水平地生成在屏幕之外。

我们创建了一个类型为`Random`的新对象，并生成了一个在 10 到 15 之间的随机数。这是我们的敌人能够移动的最大和最小速度。这些值相对随意，我们在第四章进行游戏测试时可能会调整它们，*Tappy Defender – Going Home*。

### 注意

如果你好奇`generator.nextInt(6)+10;`是如何生成 10 到 15 之间的数字的，这是因为`6`参数导致`nextInt()`返回一个 0 到 5 之间的数字。

然后，我们将敌人飞船的*x*坐标设置为屏幕，这样它就会在屏幕最左侧生成。实际上，这是在屏幕外生成的。但这没问题，因为它会逐渐进入玩家的视野，而不是一次性出现。

我们现在基于`maxY`生成另一个随机数——敌人飞船位图的高度`(bitmap.getHeight())`——为我们的敌人飞船生成一个随机但合理的*y*坐标。

现在我们需要做的是通过编写它们的更新方法给敌人赋予生命。

## 让敌人“思考”

现在，我们可以处理`EnemyShip`类的`update`方法。目前，我们只需要处理两件事。首先，让敌人向玩家端的屏幕飞行。我们需要考虑敌人的速度和玩家的速度以准确模拟这一点。我们需要这样做的原因是，当玩家加速时，他期望自己的速度会增加，物体更快地向他冲来。然而，太空船的图形是水平静止的。

我们可以同时根据敌人的静态速度和随机生成的速度以及玩家动态设定的速度（通过加速）增加敌人移动的速度，这将给玩家一种加速的感觉，尽管飞船图形从未向前移动。

另一个问题就是敌人的飞船最终会从屏幕左侧飞出。我们需要检测这种情况，并在右侧以新的随机*y*坐标和速度重生它。这与我们在构造函数中所做的一样。

在我们真正开始编写代码之前，先考虑一个问题。如果敌人要留意并利用玩家的速度，它需要能够获取这个速度。注意在下一个代码块中，`EnemyShip`类的`update`方法声明有一个参数用来接收玩家的速度。

当我们向`TDView`类的`update`方法中添加代码时，我们将会看到它是如何传递的。现在，为`EnemyShip`类的`update`方法输入以下代码，以实现我们刚才讨论的内容：

```java
public void update(int playerSpeed){

  // Move to the left
  x -= playerSpeed;
  x -= speed;

  //respawn when off screen
  if(x < minX-bitmap.getWidth()){
    Random generator = new Random();
    speed = generator.nextInt(10)+10;
    x = maxX;
    y = generator.nextInt(maxY) - bitmap.getHeight();
  }
}
```

如你所见，我们首先将敌人的*x*坐标减去玩家的速度，然后减去敌人的速度。当玩家加速时，敌人会以更快的速度向玩家飞行。然而，如果玩家没有加速，那么敌人将以之前随机生成的速度攻击。

```java
// Move to the left
x -= playerSpeed;
x -= speed;
```

之后，我们简单地检测敌人的位图右侧是否已经从屏幕左侧消失。这是通过检测`EnemyShip`类的*x*坐标是否在屏幕外位图的宽度处完成的。

```java
if(x < minX-bitmap.getWidth()){
```

然后，我们重生同一个对象，让它再次向玩家发起攻击。这对玩家来说就像是完全新的敌人。

我们还必须做的最后三件事是声明并初始化一个来自`EnemyShip`的新对象。实际上，让我们创建三个。

在这里，在我们`TDView.java`文件中声明玩家飞船的地方，像这样声明三个敌舰：

```java
// Game objects
private PlayerShip player;
public EnemyShip enemy1;

```

```java
public EnemyShip enemy2;
public EnemyShip enemy3;

```

现在，在我们`TDView`类的构造函数中，初始化我们的三个新敌人：

```java
// Initialize our player ship
player = new PlayerShip(context, x, y);
enemy1 = new EnemyShip(context, x, y);
enemy2 = new EnemyShip(context, x, y);
enemy3 = new EnemyShip(context, x, y);

```

在我们`TDView`类的`update`方法中，我们依次调用了每个新对象的`update`方法。在这里，我们也可以看到如何将玩家的速度传递给每个敌人，以便它们在自己的`update`方法中使用它来相应地调整速度。

```java
// Update the player
player.update();
// Update the enemies
enemy1.update(player.getSpeed());
enemy2.update(player.getSpeed());
enemy3.update(player.getSpeed());
```

最后，在`TDView`类的`draw`方法中，我们在屏幕上绘制我们新的敌人。

```java
// Draw the player
canvas.drawBitmap
    (player.getBitmap(), player.getX(), player.getY(), paint);

canvas.drawBitmap
 (enemy1.getBitmap(), 
 enemy1.getX(), 
 enemy1.getY(), paint);

canvas.drawBitmap
 (enemy2.getBitmap(), 
 enemy2.getX(), 
 enemy2.getY(), paint);

canvas.drawBitmap
 (enemy3.getBitmap(), 
 enemy3.getX(), 
 enemy3.getY(), paint);

```

你现在可以运行游戏并尝试一下这个功能。

第一个也是最明显的问题是玩家和敌人会直接穿过对方。我们将在本章的*碰撞检测——相互碰撞的部分*解决这个问题。但现在，我们可以通过绘制星形/星际尘埃场作为背景来增强玩家的沉浸感。

# 飞行的刺激——滚动背景

实现我们的星际尘埃将会非常快和简单。我们要做的是创建一个具有与其他游戏对象非常相似属性的`SpaceDust`类。在随机位置生成它们，以随机速度向玩家移动，并在屏幕最右侧重生它们，再次赋予它们随机的速度和*y*坐标。

然后在我们的`TDView`类中，我们可以声明一个这些对象的整个数组，每一帧更新并绘制它们。

创建一个新类，并将其命名为`SpaceDust`。现在输入此代码：

```java
public class SpaceDust {

    private int x, y;
    private int speed;

    // Detect dust leaving the screen
    private int maxX;
    private int maxY;
    private int minX;
    private int minY;

    // Constructor
    public SpaceDust(int screenX, int screenY){

        maxX = screenX;
        maxY = screenY;
        minX = 0;
        minY = 0;

        // Set a speed between  0 and 9
        Random generator = new Random();
        speed = generator.nextInt(10);

        //  Set the starting coordinates
        x = generator.nextInt(maxX);
        y = generator.nextInt(maxY);
    }

    public void update(int playerSpeed){
        // Speed up when the player does
        x -= playerSpeed;
        x -= speed;

        //respawn space dust
        if(x < 0){
            x = maxX;
            Random generator = new Random();
            y = generator.nextInt(maxY);
            speed = generator.nextInt(15);
        }
    }

    // Getters and Setters
    public int getX() {

        return x;
    }

    public int getY() {

        return y;
    }
}
```

这是`SpaceDust`类中发生的事情。在上一代码块的顶部，我们声明了通常的速度和最大/最小变量。它们将使我们能够检测到`SpaceDust`对象离开屏幕左侧并需要在右侧重新生成时，并为重新生成对象的高度提供合理的边界。

然后在`SpaceDust`构造函数中，我们用随机值初始化`speed`、`x`和`y`变量，但要在我们刚刚初始化的最大和最小变量设定的范围内。

然后我们实现了`SpaceDust`类的`update`方法，它根据对象和玩家的速度将对象向左移动，然后检查对象是否已经飞出屏幕左侧边缘并在必要时使用随机但适当的值重新生成它。

在底部，我们提供了两个 getter 方法，以便我们的`draw`方法知道在哪里绘制每一粒尘埃。

现在，我们可以创建一个`ArrayList`对象来保存所有的`SpaceDust`对象。在`TDView`类顶部声明其他游戏对象的地方下面声明它：

```java
// Make some random space dust
public ArrayList<SpaceDust> dustList = new
  ArrayList<SpaceDust>();
```

在`TDView`构造函数中，我们可以使用`for`循环初始化一堆`SpaceDust`对象，然后将它们放入`ArrayList`对象中：

```java
int numSpecs = 40;

for (int i = 0; i < numSpecs; i++) {
  // Where will the dust spawn?
  SpaceDust spec = new SpaceDust(x, y);
  dustList.add(spec);
}
```

我们总共创建了四十粒尘埃。每次通过循环，我们创建一粒新的尘埃，`SpaceDust`构造函数为其分配一个随机位置和一个随机速度。然后，我们使用`dustList.add(spec);`将`SpaceDust`对象放入我们的`ArrayList`对象中。

接下来，我们跳转到`TDView`类的`update`方法，并使用增强的`for`循环来调用每个`SpaceDust`对象的`update()`：

```java
for (SpaceDust sd : dustList) {
  sd.update(player.getSpeed());
}
```

请记住，我们传入了玩家速度，以便尘埃相对于玩家的速度增加和减少其速度。

现在要绘制所有的空间尘埃，我们遍历`ArrayList`对象一次绘制一粒尘埃。当然，我们将代码添加到我们的`TDView`类的`draw`方法中，但我们必须确保首先绘制空间尘埃，使其出现在其他游戏对象后面。此外，我们在使用我们的`Canvas`对象的`drawPoint`方法为每个`SpaceDust`对象绘制单个像素之前，添加了一行额外的代码以切换像素颜色为白色。

在`TDView`类的`draw`方法中，添加此代码来绘制我们的尘埃：

```java
// White specs of dust
paint.setColor(Color.argb(255, 255, 255, 255));

//Draw the dust from our arrayList
for (SpaceDust sd : dustList) {
      canvas.drawPoint(sd.getX(), sd.getY(), paint);

    // Draw the player
    // ...
}
```

这里的唯一新事物是`canvas.drawpoint...`这行代码。除了向屏幕绘制位图，`Canvas`类还允许我们绘制诸如点、线这样的基本图形，以及文本和形状等。在第四章，*Tappy Defender – Going Home*中绘制游戏 HUD 时，我们将使用这些功能。

何不运行这个应用程序，看看我们已经实现了多少整洁的功能？在这张截图中，我临时将`SpaceDust`对象的数量增加到`200`，仅供娱乐。你还可以看到我们已经绘制了敌人，它们在随机的*y*坐标以随机速度攻击：

![飞行的刺激——滚动背景](img/B043422_03_03.jpg)

# 碰撞检测那些事

碰撞检测是一个相当广泛的主题。在本书的三个项目中，我们将使用各种不同的方法来检测物体何时发生碰撞。

所以，这里快速了解一下我们进行碰撞检测的选择，以及不同方法在哪些情况下可能适用。

本质上，我们只需要知道游戏中某些物体何时接触到其他物体。然后，我们可以通过爆炸、减少护盾、播放声音等方式对此事件做出反应，或者采取任何适当的措施。我们需要广泛了解不同的选择，这样我们才能在任何特定游戏中做出正确的决定。

## 碰撞检测选项

首先，这里有一些不同的数学计算方法我们可以利用，以及它们可能在什么情况下有用。

### 矩形相交

这种碰撞检测方法非常直观。我们围绕想要检测碰撞的物体画一个假想的矩形，我们可以称之为命中框或边界矩形。然后，检测它们是否相交。如果相交，那么就发生了碰撞：

![矩形相交](img/B043422_03_04.jpg)

命中框相交的地方，我们称之为碰撞。从前面的图片可以看出，这种方法远非完美。然而，在某些情况下，它已经足够了。要实现这个方法，我们只需要使用两个物体的*x*和*y*坐标来检测它们是否相交。

不要使用下面的代码。它仅用于演示目的。

```java
if(ship.getHitbox().right > enemy.getHitbox().left  
    && ship.getHitbox().left < enemy.getHitbox().right ){
    // Ship is intersecting enemy on x axis
    //But they could be at different heights

    if(ship.getHitbox().top < enemy.getHitbox().bottom  
        && ship.getHitbox().bottom > enemy.getHitbox().top ){
        // Ship is intersecting enemy on y axis as well
        // Crash
    }
}
```

上述代码假设我们有一个`getHitbox`方法，它返回给定物体的左右*x*坐标以及上下*y*坐标。在上述代码中，我们首先检查*x*轴是否重叠。如果没有，那么就没有继续的必要了。如果它们在*x*轴上重叠，那么检查*y*轴。如果它们在*y*轴上也没有重叠，那么可能是敌人从上方或下方飞过。如果它们在*y*轴上也重叠，那么我们就有了碰撞。

请注意，我们可以先检查*x*轴或*y*轴，只要两个轴都检查了即可。

### 半径重叠

这个方法同样用于检测两个命中框是否相互相交，但正如标题所示，它使用圆形而非矩形。这有明显的优缺点。主要是这种方法对于更接近圆形的形状效果很好，对于细长形状则效果不佳。

![半径重叠](img/B043422_03_05.jpg)

从前面的图片中，我们可以很容易地看出半径重叠方法对于这些特定物体是如何不精确的，也不难想象对于一个圆形物体比如球来说，它将是完美的。

这里是我们如何实施这种方法。

### 注意

下面的代码仅用于演示目的。

```java
// Get the distance of the two objects from 
// the edges of the circles on the x axis
distanceX = (ship.getHitBox.centerX + ship.getHitBox.radius) - 
  (enemy.getHitBox.centerX + enemy.getHitBox.radius;

// Get the distance of the two objects from 
// the edges of the circles on the y axis
distanceY = (ship.getHitBox.centerY + ship.getHitBox.radius) -  
  (enemy.getHitBox.centerY + enemy.getHitBox.radius;

// Calculate the distance between the center of each circle
double distance = Math.sqrt
    (distanceX * distanceX + distanceY * distanceY);

// Finally see if the two circles overlap
if (distance < ship.getHitBox.radius + enemy.getHitBox.radius) {
    // bump
}
```

代码再次做出了一些假设。比如我们有一个 `getHitBox` 方法，它可以返回半径以及中心的 *x* 和 *y* 坐标。此外，因为静态的 `Math.sqrt` 方法接收并返回一个 `double` 类型的变量，我们将需要在 `SpaceShip` 和 `EnemyShip` 类中开始使用不同的类型。

### 注意

如果我们初始化距离的方式：`Math.sqrt(distanceX * distanceX + distanceY * distanceY);` 让人有些迷惑，它实际上只是使用了勾股定理来获取一个直角三角形的斜边长度，这个长度等于两个圆心之间直线距离的长度。在我们解决方案的最后一步，我们测试 `distance < ship.getHitBox.radius + enemy.getHitBox.radius`，这样我们可以确定一定发生了碰撞。这是因为如果两个圆的中心点比它们的半径之和还要近，那么它们一定发生了重叠。

### 交叉数算法

这种方法在数学上更为复杂。然而，正如我们将在第三个也是最后一个项目中看到的，它非常适合检测一个点是否与凸多边形相交：

![交叉数算法](img/B043422_03_06.jpg)

这非常适合制作一个《小行星》克隆游戏，我们将在最终项目中进一步探索这种方法，并看到它的实际应用。

## 优化

正如我们所见，不同的碰撞检测方法至少可以根据你在哪种情况下使用哪种方法而有至少两个问题。这些问题是缺乏精确度和对 CPU 周期的消耗。

### 多个碰撞箱

第一个问题，精确度不足，可以通过每个对象具有多个碰撞箱来解决。

我们只需向游戏对象添加所需数量的碰撞箱，以最有效的方式*包装*它，然后依次对每个执行相同的矩形相交代码。

### 邻居检查

这种方法允许我们只检查那些彼此在近似相同区域内的对象。这可以通过检查我们的游戏中的给定两个对象在哪个邻域内，并且只有在有可能发生碰撞的情况下，才执行更耗 CPU 的碰撞检测来实现。

假设我们有 10 个对象，每个对象都需要与其他对象进行检查，那么我们需要执行 10 的平方（100）次碰撞检查。如果我们首先进行邻居检查，我们可以显著减少这个数字。在图表中非常假设的情况下，如果我们首先检查对象是否共享同一个区域，那么对于 10 个对象，我们最多只需要执行 11 次碰撞检查，而不是 100 次。

![邻居检查](img/B043422_03_07.jpg)

在代码中实现这一点可以很简单，即为每个游戏对象提供一个区域成员变量，然后遍历对象列表，仅检查它们是否在同一个区域。

### 注意

在我们的三个游戏项目中，我们将使用所有这些选项和优化。

## 适用于 Tappy Defender 的最佳选项

既然我们已经了解了碰撞检测的选项，我们可以决定在我们当前游戏中采取的最佳行动。我们所有的飞船都近似于矩形（或正方形），它们上面很少有或没有突出部分，而且我们只有一个真正关心与其他物体发生碰撞的对象。

这往往建议我们可以为玩家和敌人使用单一的矩形碰撞箱，并执行纯角对齐的全局碰撞检测。如果你对我们选择简单的方法感到失望，那么你将会很高兴听到在接下来的两个项目中，我们将要研究所有更高级的技术。

为了让生活更加便捷，Android API 有一个方便的`Rect`类，它不仅可以表示我们的碰撞箱，而且还有一个整洁的`intersects`方法，基本上与矩形相交碰撞检测相同。让我们考虑如何为我们的游戏添加碰撞检测。

首先，我们的所有敌人和玩家飞船都需要一个碰撞箱。添加这段代码来声明一个名为`hitbox`的新`Rect`成员。在`PlayerShip`和`EnemyShip`类中都这样做：

```java
// A hit box for collision detection
private Rect hitBox;
```

### 提示

**重要！**

确保为`EnemyShip`类和`PlayerShip`类都完成上一步和接下来的三个代码块。我每次都会提醒你，但觉得还是提前提一下比较好。

现在，我们需要向`PlayerShip`类和`EnemyShip`类添加一个获取器方法。将此代码添加到两个类中：

```java
public Rect getHitbox(){
  return hitBox;
}
```

接下来，我们需要确保在两个构造函数中都初始化我们的碰撞箱。确保在构造函数的最后输入代码：

```java
// Initialize the hit box
hitBox = new Rect(x, y, bitmap.getWidth(), bitmap.getHeight());
```

现在，我们需要确保碰撞箱与我们的敌人和玩家的坐标保持最新。做这件事最好的地方是敌舰/玩家飞船的`update`方法。下一代码块将使用飞船的当前坐标更新碰撞箱。确保将此代码块添加到`update()`方法的最后，以便在`update`方法进行调整后，用坐标更新碰撞箱。同样，也要将其添加到`PlayerShip`和`EnemyShip`中：

```java
// Refresh hit box location
hitBox.left = x;
hitBox.top = y;
hitBox.right = x + bitmap.getWidth();
hitBox.bottom = y + bitmap.getHeight();
```

我们的碰撞箱具有代表位图外框的坐标。这种情况几乎完美，除了边缘周围的透明部分。

现在，我们可以从`TDView`类的`update`方法中使用我们的碰撞箱来检测碰撞。但首先，我们需要决定碰撞发生时我们打算做什么。

我们需要参考我们游戏的规则。我们在第二章，*Tappy Defender – First Step*的开头讨论过它们。我们知道玩家有三个护盾，但一个敌方飞船在一次撞击后就会爆炸。将护盾等内容留到章节的后面部分是有道理的，但我们需要某种方式来查看我们的碰撞检测的实际效果并确保它正常工作。

在这个阶段，最简单的确认碰撞的方法可能是让敌方飞船消失并像正常情况一样重生，就像它是一艘全新的敌方飞船一样。我们已经为此建立了一个机制。我们知道，当敌方飞船从屏幕左侧移出时，它会在右侧重生，就像是一艘新的敌方飞船。我们需要做的就是立即将敌方飞船传送到屏幕左侧外的位置，`EnemyShip`类会完成其余的工作。

我们需要能够改变`EnemyShip`对象的*x*坐标。让我们为`EnemyShip`类添加一个 setter 方法，这样我们就可以操纵所有敌方太空飞船的*x*坐标。如下所示：

```java
// This is used by the TDView update() method to
// Make an enemy out of bounds and force a re-spawn
public void setX(int x) {
  this.x = x;
}
```

现在，我们可以进行碰撞检测并在检测到碰撞时做出响应。下面这段代码使用了静态方法`Rect.intersects()`，通过比较玩家飞船的碰撞箱与每个敌方碰撞箱，来检测是否发生碰撞。如果检测到碰撞，适当的敌方飞船将被移出屏幕，准备在下一帧由其自己的`update`方法重生。将这段代码放在`TDView`类的`update`方法的最顶部：

```java
// Collision detection on new positions
// Before move because we are testing last frames
// position which has just been drawn

// If you are using images in excess of 100 pixels
// wide then increase the -100 value accordingly
if(Rect.intersects
  (player.getHitbox(), enemy1.getHitbox())){
    enemy1.setX(-100);
}

if(Rect.intersects
  (player.getHitbox(), enemy2.getHitbox())){
    enemy2.setX(-100);
}

if(Rect.intersects
  (player.getHitbox(), enemy3.getHitbox())){
    enemy3.setX(-100);
}
```

这样就完成了，我们的碰撞现在可以工作了。能够真正看到发生的情况可能会更好。为了调试的目的，让我们在所有太空飞船周围画一个矩形，这样我们就可以看到碰撞箱了。我们将使用`Paint`类的`drawRect`方法，并将我们的碰撞箱的属性作为参数传递，以定义要绘制的区域。如您所料，这段代码应该放在`draw`方法中。请注意，它应该在绘制我们飞船的代码之前，这样矩形就在它们后面绘制了，但在我们清除屏幕的代码之后，如高亮代码所示：

```java
// Rub out the last frame
canvas.drawColor(Color.argb(255, 0, 0, 0));

// For debugging
// Switch to white pixels
paint.setColor(Color.argb(255, 255, 255, 255));

// Draw Hit boxes
canvas.drawRect(player.getHitbox().left, 
 player.getHitbox().top, 
 player.getHitbox().right, 
 player.getHitbox().bottom, 
 paint);

canvas.drawRect(enemy1.getHitbox().left, 
 enemy1.getHitbox().top, 
 enemy1.getHitbox().right, 
 enemy1.getHitbox().bottom, 
 paint);

canvas.drawRect(enemy2.getHitbox().left, 
 enemy2.getHitbox().top, 
 enemy2.getHitbox().right, 
 enemy2.getHitbox().bottom, 
 paint);

canvas.drawRect(enemy3.getHitbox().left, 
 enemy3.getHitbox().top, 
 enemy3.getHitbox().right, 
 enemy3.getHitbox().bottom, 
 paint);

```

我们现在可以运行 Tappy Defender，开启调试模式的碰撞箱，查看游戏运行的实际效果：

![Tappy Defender 的最佳选项](img/B043422_03_08.jpg)

当我们用完调试代码后，可以注释掉这段代码，如果以后需要，再取消注释。

# 总结

我们现在已经拥有了完成游戏所需的所有游戏对象。它们都在我们设计模式的模型部分内部进行思考和自我表示。此外，我们的玩家终于可以控制他的太空飞船了，我们也能检测到他是否发生碰撞。

在下一章中，我们将为我们的游戏添加最后的润色，包括添加一个 HUD（抬头显示），实现游戏规则，增加一些额外的功能，并通过测试游戏来使一切保持平衡。
