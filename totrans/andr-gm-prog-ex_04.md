# 第四章。Tappy Defender – 回家之路

我们即将完成我们的第一款游戏。在本章中，我们将绘制一个 HUD 来显示玩家游戏内的信息，并实现游戏规则，以便玩家可以赢得胜利、失败，并获得最快时间。

之后，我们将制作一个暂停屏幕，以便玩家在赢得或输掉比赛后可以欣赏他们的成就（或不是）。

在本章中，我们还将生成自己的声音效果，并将它们添加到游戏中。接下来，我们将允许玩家保存他们的最快时间，最后，我们将添加一系列小改进，包括根据玩家的 Android 设备屏幕分辨率进行一些难度平衡调整。

# 显示 HUD

我们需要开始使我们的游戏更加完善。游戏有得分，或者在我们的情况下是时间，还有其他规则。为了让玩家跟踪他们的进度，我们需要显示游戏的统计数据。

在这里，我们将快速设置一个 HUD，它将显示玩家在躲避敌人时需要知道的所有信息。我们还将声明并初始化为 HUD 提供数据的所需变量。在下一节*实现规则*中，我们可以开始操纵诸如护盾、时间、最快时间等变量。

我们可以从为`TDView`类添加一些成员变量开始。我们使用浮点值作为`distanceRemaining`变量，因为我们将使用伪公里和公里分数来表示英雄到达她的家园星球前剩余的距离。对于`timeTaken`、`timeStarted`和`fastestTime`变量，我们将使用**长整型**，因为时间以毫秒表示，数值会变得非常大。在`TDView`类声明后添加以下代码：

```java
private float distanceRemaining;
private long timeTaken;
private long timeStarted;
private long fastestTime;
```

目前，我们将这些变量保留为其默认值，并专注于在 HUD 中显示它们。在下一节*实现规则*中，我们将使它们变得有用和有意义。

现在，我们可以继续绘制我们的 HUD，以显示玩家在游戏过程中可能想要知道的所有数据。像往常一样，我们将使用多功能`Paint`类对象`paint`来完成大部分工作。这次，我们使用`drawText`方法向屏幕添加文本，`setTextAlign`方法来对齐文本，以及`setTextSize`来缩放文本的大小。

我们现在可以将这段代码添加到`TDView`类的`draw`方法中。将其作为最后要绘制的内容，就在调用`unlockCanvasAndPost()`之前，如高亮代码所示：

```java
// Draw the hud
paint.setTextAlign(Paint.Align.LEFT);
paint.setColor(Color.argb(255, 255, 255, 255));
paint.setTextSize(25);
canvas.drawText("Fastest:"+ fastestTime + "s", 10, 20, paint);
canvas.drawText("Time:" + timeTaken + "s", screenX / 2, 20, paint);
canvas.drawText("Distance:" + 
 distanceRemaining / 1000 + 
 " KM", screenX / 3, screenY - 20, paint);

canvas.drawText("Shield:" + 
 player.getShieldStrength(), 10, screenY - 20, paint);

canvas.drawText("Speed:" + 
 player.getSpeed() * 60 + 
 " MPS", (screenX /3 ) * 2, screenY - 20, paint);

// Unlock and draw the scene
ourHolder.unlockCanvasAndPost(canvas);
```

输入这段代码后，我们遇到了一些错误，可能还有一些疑问。

首先，我们将处理这些问题。在下一节*实现规则*中，我们将更详细地了解我们对`fastestTime`、`timeTaken`、`distanceRemaining`以及`getSpeed`返回值的操作。简单来说，它们是表示距离和时间的量，旨在让玩家了解自己的表现如何。它们并不是真实的距离模拟，尽管时间是一致的。

我们将处理的第一 个错误是由于调用一个不存在的方法`player.getShieldStrength`引起的。在`PlayerShip`类中添加一个成员变量`shieldStrength`：

```java
private int shieldStrength;
```

在`PlayerShip`构造函数中将其初始化为`2`：

```java
 shieldStrength = 2;
```

在`PlayerShip`类中实现你缺失的 getter 方法：

```java
public int getShieldStrength() {
  return shieldStrength;
}
```

最后的错误是由于未声明的变量`screenX`和`screenY`引起的。现在显然我们需要在这部分代码中获取屏幕分辨率。处理这个问题的最快方式是声明两个名为`screenX`和`screenY`的新类变量。现在就在`TDView`类声明之后声明它们：

```java
private int screenX;
private int screenY;
```

如我们所见，知道屏幕坐标在许多地方都很有用，所以这样做是有意义的。

现在，在`TDView`构造函数中，使用`GameActivity`类传递进来的分辨率初始化`screenX`和`screenY`。在构造函数开始时进行如下操作：

```java
screenX = x;
screenY = y;
```

我们现在可以运行游戏并查看我们的 HUD。我们 HUD 中唯一具有有意义数据的部分是**Shield**和**Speed**标签。速度是 MPS（每秒米数）的伪测量值。当然，这并不反映现实，但相对于呼啸而过的星星、逼近的敌人，以及玩家目标距离的减少，它是有相对性的：

![显示 HUD](img/B04322_04_01.jpg)

# 实现规则

现在，我们应该暂停并思考后期项目中我们需要做什么，因为这会影响我们实现规则时的操作。当玩家的飞船被摧毁或玩家达到目标时，游戏将结束。这意味着游戏需要重新开始。我们不想每次都退回到主屏幕，所以我们需要一种方法从`TDView`类内部重新开始游戏。

为了实现这一点，我们将在`TDView`类中实现一个`startGame`方法。构造函数将能够调用它，我们的游戏循环在必要时也能调用它。

还需要将构造函数当前执行的一些任务传递给新的`startGame`方法，以便它能正确地完成其工作。此外，我们还将使用`startGame`初始化游戏规则和 HUD 所需的一些变量。

为了完成我们讨论的内容，`startGame()`需要应用程序`Context`对象的副本。所以，就像我们对`startX`和`startY`所做的那样，我们现在将`context`作为`TDView`的成员。在`TDView`类声明之后进行声明：

```java
private Context context;
```

在构造函数中，在调用`super()`之后立即进行初始化，如下所示：

```java
super(context);
this.context  = context;

```

我们现在可以实现新的`startGame`方法。大部分代码只是从构造函数中移动过来的。注意一些微妙但重要的区别，比如使用类的版本`screenX`和`screenY`来代替构造函数参数*x*和*y*。同时，我们初始化`distanceRemaining`、`timeTaken`和`timeStarted`。

```java
private void startGame(){
    //Initialize game objects
        player = new PlayerShip(context, screenX, screenY);
        enemy1 = new EnemyShip(context, screenX, screenY);
        enemy2 = new EnemyShip(context, screenX, screenY);
        enemy3 = new EnemyShip(context, screenX, screenY);

        int numSpecs = 40;
        for (int i = 0; i < numSpecs; i++) {
            // Where will the dust spawn?
            SpaceDust spec = new SpaceDust(screenX, screenY);
            dustList.add(spec);
        }

        // Reset time and distance
        distanceRemaining = 10000;// 10 km
        timeTaken = 0;

        // Get start time
        timeStarted = System.currentTimeMillis();
}
```

### 注意

你是否在疑惑`timeStarted`初始化的部分是怎么回事？我们使用了`System`类的方法`currentTimeMillis`来初始化`startTime`，现在`startTime`保存的是自 1970 年 1 月 1 日以来的毫秒数。我们将在接下来的*结束游戏*部分看到如何使用这个值。`System`类有很多用途，这里我们用它来获取自 1970 年 1 月 1 日以来的毫秒数。这是计算机中测量时间的常见系统，称为 Unix 时间。1970 年 1 月 1 日第一个毫秒之前的那一刻被称为 Unix 纪元。

现在，注释掉或删除`TDView`构造函数中不再需要的代码，但要在相应位置添加对`startGame()`的调用：

```java
// Initialize our player ship
//player = new PlayerShip(context, x, y);
//enemy1 = new EnemyShip(context, x, y);
//enemy2 = new EnemyShip(context, x, y);
//enemy3 = new EnemyShip(context, x, y);

//int numSpecs = 40;

//for (int i = 0; i < numSpecs; i++) {
      // Where will the dust spawn?
      //SpaceDust spec = new SpaceDust(x, y);
      //dustList.add(spec);
//}

startGame();

```

接下来，我们想要创建一个方法来减少`PlayerShip`的护盾强度。这样，当我们检测到碰撞时，可以每次减少一点。在`PlayerShip`类中添加这个简单的方法：

```java
public void reduceShieldStrength(){
  shieldStrength --;
}
```

现在，我们可以跳到`TDView`类的`update`方法，并添加代码进一步实现我们的游戏规则。我们将在进行所有碰撞检测之前添加一个布尔变量`hitDetected`。在每个检测到击中的`if`块内部，我们可以将`hitDetected`设置为`true`。

然后，在所有碰撞检测代码之后，我们可以检查是否检测到击中，并相应地减少玩家的护盾强度。以下是`TDView`类的`update`方法顶部部分，新的代码行已高亮显示：

```java
// Collision detection on new positions
// Before move because we are testing last frames
// position which has just been drawn
boolean hitDetected = false;
if(Rect.intersects(player.getHitbox(), enemy1.getHitbox())){
 hitDetected = true;
    enemy1.setX(-100);
}

if(Rect.intersects(player.getHitbox(), enemy2.getHitbox())){
 hitDetected = true;
    enemy2.setX(-100);
}

if(Rect.intersects(player.getHitbox(), enemy3.getHitbox())){
 hitDetected = true;
    enemy3.setX(-100);
}

if(hitDetected) {
 player.reduceShieldStrength();
 if (player.getShieldStrength() < 0) {
 //game over so do something
 }
}

```

注意在调用`player.reduceShieldStrength`之后的嵌套 if 语句。这会检测玩家是否已经失去了所有护盾并失败。我们很快就会处理这里会发生的情况。

我们非常接近完成游戏规则了。我们只需要根据玩家的速度减少`distanceRemaining`。这样我们才知道玩家何时成功。我们还需要更新`timeTaken`变量，以便每次调用我们的绘图方法时更新 HUD。这可能看起来不重要，但如果我们稍微考虑一下未来，我们可以预见到游戏结束的时候，无论是玩家失败还是玩家获胜。让我们谈谈游戏的结束。

# 结束游戏

如果游戏没有结束，那么游戏正在进行中，如果玩家刚刚死亡或获胜，那么游戏已经结束。我们需要知道游戏何时结束，何时在进行中。让我们在`TDView`类声明之后添加一个新的成员变量`gameEnded`并声明它。

```java
private boolean gameEnded;
```

现在，我们可以在`startGame`方法中初始化`gameEnded`。将这行代码作为该方法中的最后一行输入。

```java
gameEnded = false;
```

现在，我们可以完成游戏规则逻辑的最后几行，但需要用测试来包裹它们，以查看游戏是否已经结束。在 `TDView` 类的 `update` 方法最后添加以下代码，有条件地更新我们的游戏规则逻辑：

```java
if(!gameEnded) {
            //subtract distance to home planet based on current speed
            distanceRemaining -= player.getSpeed();

            //How long has the player been flying
            timeTaken = System.currentTimeMillis() - timeStarted;
}
```

我们的 HUD 现在将具有准确的数据，让玩家了解他们到底做得如何。我们还可以检测玩家是否回到家并获胜，因为 `distanceRemaining` 将通过零。此外，当剩余距离小于零时，我们可以测试 `timeTaken` 是否小于 `fastestTime`，如果是，则更新 `fastestTime`。我们还可以将 `gameEnded` 设置为 `true`。在 `TDView` 类的 `update` 方法的最后一块代码后直接添加以下代码：

```java
//Completed the game!
if(distanceRemaining < 0){
  //check for new fastest time
  if(timeTaken < fastestTime) {
    fastestTime = timeTaken;
  }

  // avoid ugly negative numbers
  // in the HUD
  distanceRemaining = 0;

  // Now end the game
  gameEnded = true;
}
```

当玩家获胜时我们结束了游戏；现在，添加这行代码，当玩家失去所有护盾时结束游戏。在 `TDView` 类的 `update` 方法中更新此代码。新的一行代码已高亮：

```java
if(hitDetected) {
  player.reduceShieldStrength();
  if (player.getShieldStrength() < 0) {
 gameEnded = true;
 }
}
```

现在，我们只需要在 `gameEnded` 设置为 `true` 时实际执行一些操作。

一种方法是，根据 `gameEnded` 布尔值是真是假来交替绘制 HUD。在 `draw` 方法中找到 HUD 绘制代码，再次展示在这里以便于参考：

```java
// Draw the HUD
paint.setTextAlign(Paint.Align.LEFT);
paint.setColor(Color.argb(255, 255, 255, 255));
paint.setTextSize(25);
canvas.drawText("Fastest:"+ fastestTime + "s", 10, 20, paint);
canvas.drawText("Time:" + timeTaken + "s", screenX / 2, 20, paint);

canvas.drawText("Distance:" + 
  distanceRemaining / 1000 + 
  " KM", screenX / 3, screenY - 20, paint);

canvas.drawText("Shield:" + 
  player.getShieldStrength(), 10, screenY - 20, paint);

canvas.drawText("Speed:" + 
  player.getSpeed() * 60 +
  " MPS", (screenX /3 ) * 2, screenY - 20, paint);
```

我们希望将那段代码包裹在一个 `if`-`else` 块中。如果游戏没有结束，就绘制正常的 HUD，否则绘制一个替代的。像这样包裹 HUD 绘制代码：

```java
if(!gameEnded){
  // Draw the hud
  paint.setTextAlign(Paint.Align.LEFT);
  paint.setColor(Color.argb(255, 255, 255, 255));
  paint.setTextSize(25);
  canvas.drawText("Fastest:"+ fastestTime + "s", 10, 20, paint);

  canvas.drawText("Time:" + 
    timeTaken + 
    "s", screenX / 2, 20,   paint);

  canvas.drawText("Distance:" + 
    distanceRemaining / 1000 + 
    " KM", screenX / 3, screenY - 20, paint);

  canvas.drawText("Shield:" + 
    player.getShieldStrength(), 10, screenY - 20, paint);

  canvas.drawText("Speed:" + 
    player.getSpeed() * 60 +
    " MPS", (screenX /3 ) * 2, screenY - 20, paint);

}else{
 //this happens when the game is ended
}

```

现在，让我们处理 `else` 块，当游戏结束时将执行这部分。我们将绘制一个大的**游戏结束**，并显示 HUD 中的结束游戏统计信息。线程继续运行，但 HUD 停止更新。在 `else` 块中输入以下代码：

```java
// Show pause screen
paint.setTextSize(80);
paint.setTextAlign(Paint.Align.CENTER);
canvas.drawText("Game Over", screenX/2, 100, paint);
paint.setTextSize(25);
canvas.drawText("Fastest:"+ 
  fastestTime + "s", screenX/2, 160, paint);

canvas.drawText("Time:" + timeTaken + 
  "s", screenX / 2, 200, paint);

canvas.drawText("Distance remaining:" + 
  distanceRemaining/1000 + " KM",screenX/2, 240, paint);

paint.setTextSize(80);
canvas.drawText("Tap to replay!", screenX/2, 350, paint);
```

注意我们使用 `setTextSize()` 切换文本大小，并使用 `setTextAlign()` 将所有文本对准屏幕中心。这就是运行游戏时的样子。我们只需要在游戏结束后找到一种重新开始游戏的方法：

![结束游戏](img/B04322_04_02.jpg)

## 重新开始游戏

为了让玩家在游戏结束后可以重新开始，我们只需要监听触摸事件并调用 `startGame()`。让我们编辑我们的 `onTouchListener()` 代码以实现这一点。我们感兴趣的是修改 `MotionEvent.ACTION_DOWN:` 的情况。我们只需在这里简单地添加条件，如果游戏结束时屏幕被触摸，就重新开始。要添加到 `MotionEvent.ACTION_DOWN:` 情况中的新代码已高亮：

```java
// Has the player touched the screen?
case MotionEvent.ACTION_DOWN:
    player.setBoosting();
 // If we are currently on the pause screen, start a new game
 if(gameEnded){
 startGame();
 }
   break;
```

尝试一下。现在你可以在暂停菜单中通过点击屏幕重新开始游戏。是我太敏感还是这里有点安静？

# 添加声音效果

在 Android 中添加声音效果真的很简单。首先，让我们看看我们可以在哪里获取声音效果。如果你只想继续编程，可以使用我在 `Chapter4/assets` 文件夹中的声音效果。

## 生成效果音

我们需要四个声音效果用于我们的 Tappy Defender 游戏：

+   当我们的玩家撞到外星人时的声音，我们将其称为 `bump.ogg`。

+   当玩家被摧毁时的声音，我们将其称为 `destroyed.ogg`。

+   游戏开始时一个有趣的声音，我们称之为`start.ogg`。

+   最后，一个胜利的欢呼声效，我们称之为`win.ogg`。

这是一个非常简短的指南，介绍如何使用 BFXR 制作这些音效。从[www.bfxr.net](http://www.bfxr.net)获取 BFXR 的免费副本。

按照网站上的简单说明进行设置。尝试其中一些功能，制作我们酷炫的音效。

### 注意

这是一个非常精简的教程。你可以用 BFXR 做很多事情。想要了解更多，请访问前一个 URL 的网站上的小贴士。

1.  运行`bfxr.exe`。![生成音效](img/B04322_04_03.jpg)

1.  尝试所有预设类型，这些预设会生成你正在处理的类型的随机声音。当你找到一个接近你想要的声音时，进行下一步操作：![生成音效](img/B04322_04_05.jpg)

1.  使用滑块微调你新声音的音调、时长和其他方面：![生成音效](img/B04322_04_04.jpg)

1.  通过点击**导出 Wav**按钮保存你的声音。尽管这个按钮的名字是这样，但如我们所见，我们也可以保存除`.wav`以外的格式。![生成音效](img/B04322_04_06.jpg)

1.  Android 喜欢使用 OGG 格式的声音，因此当要求你命名文件时，在文件名末尾使用`.ogg`扩展名。记住我们需要创建`bump.ogg`，`destroyed.ogg`，`start.ogg`和`win.ogg`。

1.  重复步骤 2 至 5，创建我们讨论过的四种音效。

1.  在 Android Studio 中右键点击`app`文件夹。在弹出菜单中，导航到**新建** | **Android 资源目录**。

1.  在**目录名称**字段中，输入`assets`。点击**确定**创建`assets`文件夹。

1.  使用你的操作系统的文件管理器，在项目的主文件夹中添加一个名为`assets`的文件夹，然后将四个声音文件添加到项目中的新`assets`文件夹中。

## `SoundPool`类

为了播放我们的声音，我们将使用`SoundPool`类。我们使用`SoundPool`构造函数的弃用版本，因为新版本需要 API 21 或更高版本，而且很可能有很多读者在使用更早版本的 Android。我们可以动态获取 Android 版本，并为 API 级别 21 之前和之后提供不同版本的代码，但旧的构造函数满足了我们的需求。

## 编码音效

声明一个`SoundPool`对象和一些整数来代表各个声音。在`TDView`类声明后立即添加此代码：

```java
private SoundPool soundPool;
    int start = -1;
    int bump = -1;
    int destroyed = -1;
    int win = -1;
```

接下来，我们可以初始化我们的`SoundPool`对象和我们的整型声音 ID。我们将代码包裹在必需的`try`-`catch`块中。

注意，调用`load()`开始了一个将我们的`.ogg`文件转换为原始声音数据的过程。如果在进行`playSound()`调用时此过程尚未完成，声音将不会播放。`load()`的调用顺序是按照它们被使用的方式来最小化这种可能性。在`TDView`类的构造函数中输入如下代码。新代码已高亮显示：

```java
TDView(Context context, int x, int y) {
  super(context);
  this.context  = context;

 // This SoundPool is deprecated but don't worry
 soundPool = new SoundPool(10, AudioManager.STREAM_MUSIC,0);
 try{
 //Create objects of the 2 required classes
 AssetManager assetManager = context.getAssets();
 AssetFileDescriptor descriptor;

 //create our three fx in memory ready for use
 descriptor = assetManager.openFd("start.ogg");
 start = soundPool.load(descriptor, 0);

 descriptor = assetManager.openFd("win.ogg");
 win = soundPool.load(descriptor, 0);

 descriptor = assetManager.openFd("bump.ogg");
 bump = soundPool.load(descriptor, 0);

 descriptor = assetManager.openFd("destroyed.ogg");
 destroyed = soundPool.load(descriptor, 0);

 }catch(IOException e){
 //Print an error message to the console
 Log.e("error", "failed to load sound files");
 }

```

在我们代码中表示游戏内适当事件的点处，使用适当的引用添加对`playSound()`的调用。我们有四种声音，所以将会有四次对`playSound()`的调用。

第一个在`startGame()`方法的最后面：

```java
soundPool.play(start, 1, 1, 0, 0, 1);
```

接下来的两行在`if(hitDetected)`块中被高亮显示：

```java
if(hitDetected) {
 soundPool.play(bump, 1, 1, 0, 0, 1);
  player.reduceShieldStrength();
  if (player.getShieldStrength() < 0) {
 soundPool.play(destroyed, 1, 1, 0, 0, 1);
      paused = true;
  }
}
```

最后一个在`if(distanceRemaining < 0)`块中被高亮显示：

```java
//Completed the game!
if(distanceRemaining < 0){
 soundPool.play(win, 1, 1, 0, 0, 1);
     //check for new fastest time
     if(timeTaken < fastestTime) {
         fastestTime = timeTaken;
     }

     // avoid ugly negative numbers
     // in the HUD
     distanceRemaining = 0;

     // Now end the game
     gameEnded = true;
}
```

现在是运行 Tappy Defender 并听听动作中的声音的时候了。

我们将看到当玩家在游戏中达到高分时如何将其保存到文件中，并在 Tappy Defender 启动时重新加载它。

# 添加持久性

您可能已经注意到当前的最快时间是零，因此永远无法被打破。另一个问题是，每次玩家退出游戏时，最高分都会丢失。现在，我们将从文件中加载一个默认的高分。当达到新的高分时，将其保存到文件中。无论玩家退出游戏还是关闭手机，他们的高分都会保留。

首先，我们需要两个新的对象。在`TDView`类声明之后，将它们声明为`TDView`类的成员。第一个是`SharedPreferences`对象，第二个是`Editor`对象，它实际上为我们写入文件：

```java
private SharedPreferences prefs;
private SharedPreferences.Editor editor;
```

我们首先使用`prefs`，因为我们只是想尝试加载一个存在的高分。我们还会初始化`editor`，以便在我们保存高分时可以使用。我们在`TDView`构造函数中这样做：

```java
// Get a reference to a file called HiScores. 
// If id doesn't exist one is created
prefs = context.getSharedPreferences("HiScores", 
  context.MODE_PRIVATE);

// Initialize the editor ready
editor = prefs.edit();

// Load fastest time from a entry in the file
//  labeled "fastestTime"
// if not available highscore = 1000000
fastestTime = prefs.getLong("fastestTime", 1000000);
```

让我们在适当的时候使用我们的`Editor`对象将任何新的最快时间写入到`HiScores`文件中。首先将显示的额外高亮行添加到我们的文件缓冲区中，然后提交更改以添加我们提议的修改：

```java
//Completed the game!
if(distanceRemaining < 0){
 soundPool.play(win, 1, 1, 0, 0, 1);
     //check for new fastest time
     if(timeTaken < fastestTime) {
         // Save high score
         editor.putLong("fastestTime", timeTaken);
         editor.commit();
         fastestTime = timeTaken;
     }

     // avoid ugly negative numbers
     // in the HUD
     distanceRemaining = 0;

     // Now end the game
     gameEnded = true;
}
```

我们需要做的最后一件事是让主屏幕加载最快的游戏时间并将其展示给玩家。我们将以与在`TDView`构造函数中完全相同的方式加载最快的时间。我们还会通过其 ID `textHighScore`获取对`TextView`的引用，这是我们在第二章*Tappy Defender – First Step*开始时分配的。然后我们使用`setText`方法将其展示给玩家。

打开`MainActivity.java`，在`onCreate`方法中添加我们刚才讨论过的那些高亮代码：

```java
// This is the entry point to our game
@Override
protected void onCreate(Bundle savedInstanceState) {
  super.onCreate(savedInstanceState);

  //Here we set our UI layout as the view
  setContentView(R.layout.activity_main);

 // Prepare to load fastest time
 SharedPreferences prefs;
 SharedPreferences.Editor editor;
 prefs = getSharedPreferences("HiScores", MODE_PRIVATE);

  // Get a reference to the button in our layout
  final Button buttonPlay =
    (Button)findViewById(R.id.buttonPlay);

 // Get a reference to the TextView in our layout
 final TextView textFastestTime = 
 (TextView)findViewById(R.id.textHighScore);

  // Listen for clicks
  buttonPlay.setOnClickListener(this);

 // Load fastest time
 // if not available our high score = 1000000
 long fastestTime = prefs.getLong("fastestTime", 1000000);

 // Put the high score in our TextView
 textFastestTime.setText("Fastest Time:" + fastestTime);

}
```

现在，我们已经有了一个可以运行的游戏。然而，它还没有真正完成。为了制作一个真正可玩且有趣的游戏，我们必须改进、优化、测试并迭代。

# 迭代

我们如何使游戏变得更好玩？让我们看看一些可能性，然后去实施它们。

## 多个不同的敌人图形

让我们通过为游戏添加更多图形使敌人更有趣。首先，我们需要将额外的图形添加到项目中。将下载包中`Chapter4/drawables`文件夹中的`enemy2.png`和`enemy3.png`复制并粘贴到 Android Studio 中的`drawables`文件夹中。

![多种不同的敌人图像](img/B04322_04_07.jpg)

enemy2 和 enemy3

现在，我们只需要修改`EnemyShip`构造函数。这段代码生成一个 0 到 2 之间的随机数，然后根据需要切换加载不同的敌人位图。我们完成的构造函数现在看起来像这样：

```java
// Constructor
public EnemyShip(Context context, int screenX, int screenY){
 Random generator = new Random();
 int whichBitmap = generator.nextInt(3);
 switch (whichBitmap){
 case 0:
 bitmap = BitmapFactory.decodeResource
 (context.getResources(), R.drawable.enemy3);
 break;

 case 1:
 bitmap = BitmapFactory.decodeResource
 (context.getResources(), R.drawable.enemy2);
 break;

 case 2:
 bitmap = BitmapFactory.decodeResource
 (context.getResources(), R.drawable.enemy);
 break;
 }

    maxX = screenX;
    maxY = screenY;
    minX = 0;
    minY = 0;

    speed = generator.nextInt(6)+10;
    x = screenX;
    y = generator.nextInt(maxY) - bitmap.getHeight();

    // Initialize the hit box
    hitBox = new Rect(x, y, bitmap.getWidth(),  bitmap.getHeight());

}
```

请注意，我们只需要将`Random generator = new Random();`这行代码移到构造函数的顶部，这样我们就可以用它来选择位图以及在构造函数的后面像往常一样生成一个随机的高度。

## 这是一个平衡的练习

游戏中最大的可玩性问题可能是，在中/高分辨率屏幕上玩游戏与在低分辨率屏幕上相比，难度差异的问题。例如，我的一个测试设备是三星 Galaxy S2，现在它已经有些年头了，当横屏握持时，屏幕分辨率为 800 x 480 像素。相比之下，我在横屏模式下使用 1920 x 1080 像素的三星 Galaxy S4 测试了游戏。这比 S2 的分辨率高出一倍多。

在 S4 上，玩家似乎可以轻松地在几乎微不足道的敌人之间滑行，而在 S2 上，玩家面临的是几乎无法穿透的外星钢铁之墙。

这个问题的真正解决方案是以伪现实世界坐标绘制游戏对象，然后将这些坐标以相同的比例映射回设备，无论分辨率如何。这样，无论在 S2 还是 S4 上，游戏看起来和玩起来的效果都是一样的。在下一个项目中，我们将构建一个更高级的游戏引擎来实现这一点。

当然，我们仍然会考虑实际物理屏幕尺寸，使玩家的体验多样化，但这种情形更容易被游戏玩家接受。

作为一种快速而简便的解决方案，我们将改变战舰的大小和敌人的数量。因此，在低分辨率下，我们将有三个敌人，但会缩小它们的大小。在高分辨率下，我们将逐渐增加敌人的数量。

在`EnemyShip`类中，在将敌人图像加载到我们的`Bitmap`对象的`switch`块之后，添加高亮显示的行，以调用我们将要编写的新方法`scaleBitmap()`：

```java
switch (whichBitmap){
    case 0:
          bitmap = BitmapFactory.decodeResource(context.getResources(),           
          R.drawable.enemy3);
          break;

    case 1:
          bitmap = BitmapFactory.decodeResource(context.getResources(),           
          R.drawable.enemy2);
          break;

   case 2:
          bitmap = BitmapFactory.decodeResource(context.getResources(),           
          R.drawable.enemy);
          break;
}

scaleBitmap(screenX);

```

现在，我们将编写新的`scaleBitmap`方法。这个简单的辅助方法接受一个参数，正如我们所见，是屏幕的水平分辨率。然后我们使用分辨率和静态的`createScaledBitmap`方法，根据屏幕分辨率按 2 或 3 的比例缩小我们的`Bitmap`对象。将新的`scaleBitmap`方法添加到`EnemyShip`类中：

```java
public void scaleBitmap(int x){

  if(x < 1000) {
       bitmap = Bitmap.createScaledBitmap(bitmap,
       bitmap.getWidth() / 3,
       bitmap.getHeight() / 3,
       false);
  }else if(x < 1200){
       bitmap = Bitmap.createScaledBitmap(bitmap,
       bitmap.getWidth() / 2,
       bitmap.getHeight() / 2,
       false);
   }
}
```

在低分辨率屏幕上，敌人的大小会被缩小。现在，让我们为高分辨率增加敌人的数量。

为此，我们将在`TDView`类中添加代码，为高分辨率屏幕添加额外的敌人。

### 注意

警告！这段代码很糟糕，但它有效，它告诉我们可以在下一个项目中在哪里进行改进。在规划游戏时，总是在良好设计与简单性之间进行权衡。从一开始就保持事物有序，我们可以在最后稍微进行一些黑客行为。是的，我们可以重新设计我们生成和存储游戏对象的方式，如果 Tappy Defender 是一个持续的项目，那么这将是有价值的。

在前三个之后，按照所示添加两个更多的敌人飞船对象：

```java
// Game objects
private PlayerShip player;
public EnemyShip enemy1;
public EnemyShip enemy2;
public EnemyShip enemy3;
public EnemyShip enemy4;
public EnemyShip enemy5;

```

现在，在`startGame`方法中添加代码，有条件地初始化这两个新对象：

```java
enemy1 = new EnemyShip(context, screenX, screenY);
enemy2 = new EnemyShip(context, screenX, screenY);
enemy3 = new EnemyShip(context, screenX, screenY);

if(screenX > 1000){
 enemy4 = new EnemyShip(context, screenX, screenY);
}

if(screenX > 1200){
 enemy5 = new EnemyShip(context, screenX, screenY);
}

```

在`update`方法中添加代码，更新我们的第四和第五个敌人并检查碰撞：

```java
// Collision detection on new positions
// Before move because we are testing last frames
// position which has just been drawn
boolean hitDetected = false;
if(Rect.intersects(player.getHitbox(), enemy1.getHitbox())){
  hitDetected = true;
  enemy1.setX(-100);
}

if(Rect.intersects(player.getHitbox(), enemy2.getHitbox())){
  hitDetected = true;
  enemy2.setX(-100);        
}

if(Rect.intersects(player.getHitbox(), enemy3.getHitbox())){
  hitDetected = true;
  enemy3.setX(-100);       
}

if(screenX > 1000){
 if(Rect.intersects(player.getHitbox(), enemy4.getHitbox())){
 hitDetected = true;
 enemy4.setX(-100); 
 }
}

if(screenX > 1200){
 if(Rect.intersects(player.getHitbox(), enemy5.getHitbox())){
 hitDetected = true;
 enemy5.setX(-100);
 }
}

if(hitDetected) {
soundPool.play(bump, 1, 1, 0, 0, 1);
            player.reduceShieldStrength();
            if (player.getShieldStrength() < 0) {
                soundPool.play(destroyed, 1, 1, 0, 0, 1);
                gameEnded = true;
            }
}

// Update the player
player.update();
// Update the enemies
enemy1.update(player.getSpeed());
enemy2.update(player.getSpeed());
enemy3.update(player.getSpeed());

if(screenX > 1000) {
 enemy4.update(player.getSpeed());
}
if(screenX > 1200) {
 enemy5.update(player.getSpeed());
}

```

最后，在`draw`方法中，在适当的时候绘制我们的额外敌人：

```java
// Draw the player
canvas.drawBitmap(player.getBitmap(), player.getX(), player.getY(), paint);
canvas.drawBitmap(enemy1.getBitmap(),
  enemy1.getX(), enemy1.getY(), paint);
canvas.drawBitmap(enemy2.getBitmap(),
  enemy2.getX(), enemy2.getY(), paint);
canvas.drawBitmap(enemy3.getBitmap(),
  enemy3.getX(), enemy3.getY(), paint);

if(screenX > 1000) {
 canvas.drawBitmap(enemy4.getBitmap(),
 enemy4.getX(), enemy4.getY(), paint);
}
if(screenX > 1200) {
 canvas.drawBitmap(enemy5.getBitmap(),
 enemy5.getX(), enemy5.getY(), paint);
}

```

当然，我们现在意识到我们可能还想缩放玩家。这使得或许我们需要一个`Ship`类，从中我们可以派生出`PlayerShip`和`EnemyShip`。

加入我们为更高分辨率屏幕添加额外敌人的笨拙方式，一个更加多态的解决方案可能更有价值。我们将在下一个项目中看到如何彻底改进这一点以及我们游戏引擎的几乎所有其他方面。

## 格式化时间

查看玩家 HUD 中时间是如何格式化的：

![格式化时间](img/B04322_04_08.jpg)

呕！让我们编写一个简单的辅助方法，让这个看起来更美观。我们将在`TDView`类中添加一个名为`formatTime()`的新方法。该方法使用游戏中经过的毫秒数（`timeTaken`）并将它们重新组织成秒和秒的小数部分。它适当地用零填充小数部分，并将结果作为`String`返回，以便在`TDView`类的`draw`方法中绘制。该方法之所以采用参数而不是直接使用成员变量`timeTaken`，是为了我们可以在一分钟内重用这段代码。

```java
private String formatTime(long time){
    long seconds = (time) / 1000;
    long thousandths = (time) - (seconds * 1000);
    String strThousandths = "" + thousandths;
    if (thousandths < 100){strThousandths = "0" + thousandths;}
    if (thousandths < 10){strThousandths = "0" + strThousandths;}
    String stringTime = "" + seconds + "." + strThousandths;
    return stringTime;
}
```

我们修改了绘制玩家 HUD 中时间的行。为了提供上下文，在下一段代码中，我注释掉了原始行的全部内容，并提供了新的行，其中包含我们对`formatTime()`的调用，并已高亮显示：

```java
//canvas.drawText("Time:" + timeTaken + "s", screenX / 2, 20, paint);
canvas.drawText("Time:" + 
 formatTime(timeTaken) + 
 "s", screenX / 2, 20, paint);

```

此外，通过一个小的改动，我们也可以在 HUD 中的**最快时间：**标签上使用这种格式。同样，旧行已被注释掉，新行已高亮显示。在`TDView`类的`draw`方法中查找并修改代码：

```java
//canvas.drawText("Fastest:" + fastestTime + "s", 10, 20, paint);
canvas.drawText("Fastest:" + 
 formatTime(fastestTime) + 
 "s", 10, 20, paint);

```

我们还应该更新暂停屏幕上的时间格式。要更改的行已被注释掉，需要添加的新行已高亮显示：

```java
// Show pause screen
paint.setTextSize(80);
paint.setTextAlign(Paint.Align.CENTER);
canvas.drawText("Game Over", screenX/2, 100, paint);
paint.setTextSize(25);

// canvas.drawText("Fastest:"
  + fastestTime + "s", screenX/2, 160, paint);
canvas.drawText("Fastest:"+ 
 formatTime(fastestTime) + "s", screenX/2, 160, paint);

// canvas.drawText("Time:" + 
  timeTaken + "s", screenX / 2, 200, paint);
canvas.drawText("Time:" 
 + formatTime(timeTaken) + "s", screenX / 2, 200, paint);

canvas.drawText("Distance remaining:" +
  distanceRemaining/1000 + " KM",screenX/2, 240, paint);
paint.setTextSize(80);
canvas.drawText("Tap to replay!", screenX/2, 350, paint);
```

**最快时间：**现在在游戏内 HUD 和暂停屏幕 HUD 上都与**时间：**的格式相同。看看我们现在整洁的时间格式：

![格式化时间](img/B04322_04_09.jpg)

## 处理返回按钮

我们将快速添加一小段代码，以处理玩家在 Android 设备上按下返回键时会发生什么。将这个新方法添加到`GameActivity`和`MainActivity`类中。我们只需检查是否按下了返回键，如果是，就调用`finish()`让操作系统知道我们已经完成了这个活动。

```java
// If the player hits the back button, quit the app
public boolean onKeyDown(int keyCode, KeyEvent event) {
  if (keyCode == KeyEvent.KEYCODE_BACK) {
       finish();
       return true;
  }
  return false;
}
```

# 完成的游戏

最后，如果你是为了理论学习而不是实践而跟进的话，这里有一个在高分辨率屏幕上完成的`GameActivity`，其中包含了几百个额外的星星和盾牌：

![完成的游戏](img/B04322_04_10.jpg)

# 总结

我们已经实现了一个基本游戏引擎的各个组成部分。我们还可以做得更多。当然，一个现代移动游戏会比我们的游戏有更多内容。当有更多的游戏对象时，我们将如何处理碰撞？我们是否可以稍微收紧一下我们的类层次结构，因为我们的`PlayerShip`和`EnemyShip`类之间有很多相似之处？我们如何在不对代码结构造成混乱的情况下添加复杂的内部角色动画，如果我们想要智能敌人，能够实际思考的敌人，该怎么办？

我们需要逼真的背景、侧目标、能量升级和拾取物品。我们希望游戏世界具有真实世界的坐标，无论屏幕分辨率如何，都能准确映射回来。

我们需要一个更智能的游戏循环，无论在哪种 CPU 上处理，都能以相同的速度运行游戏。最重要的是，我们真正需要的，比这些更重要的，是一把大大的机枪。让我们构建一个经典平台游戏。
