# 第二十二章：*第二十二章*：粒子系统和处理屏幕触摸

我们已经在上一章中使用线程实现了实时系统。在本章中，我们将创建将存在并在这个实时系统中发展的实体，就好像它们有自己的思想一样；它们将形成用户可以实现的绘图的外观。

我们还将看到用户如何通过学习如何响应与屏幕的交互来实现这些实体。这与在 UI 布局中与小部件交互是不同的。

以下是本章即将涉及的内容：

+   向屏幕添加自定义按钮

+   编写`Particle`类

+   编写`ParticleSystem`类

+   处理屏幕触摸

+   Android Studio Profiler 工具

我们将首先为我们的应用程序添加自定义 UI。

警告

这个应用程序产生明亮的闪烁颜色。这可能会引起光敏性癫痫的人不适或癫痫发作。请谨慎阅读。您可能只想阅读这个项目的理论，而不运行完成的项目。

# 技术要求

您可以在 GitHub 上找到本章中的代码文件，网址为[`github.com/PacktPublishing/Android-Programming-for-Beginners-Third-Edition/tree/main/chapter%2022`](https://github.com/PacktPublishing/Android-Programming-for-Beginners-Third-Edition/tree/main/chapter%2022)。

# 向屏幕添加自定义按钮

我们需要让用户控制何时开始另一次绘制，并清除他们以前的作品。我们需要让用户能够决定何时以及何时将绘图带到生活中。为了实现这一点，我们将在屏幕上添加两个按钮，每个任务一个按钮。

在`LiveDrawingView`类中添加下面突出显示的成员：

```kt
// These will be used to make simple buttons
private RectF mResetButton;
private RectF mTogglePauseButton;
```

我们现在有两个`RectF`实例。这些对象每个都包含四个浮点坐标，每个按钮的每个角落一个坐标。

在`LiveDrawingView`的构造函数中初始化位置：

```kt
// Initialize the two buttons
mResetButton = new RectF(0, 0, 100, 100);
mTogglePauseButton = new RectF(0, 150, 100, 250);
```

添加`RectF`类的`import`：

```kt
import android.graphics.RectF;
```

现在我们已经为按钮添加了实际的坐标。如果您在屏幕上可视化坐标，您会看到它们位于左上角，暂停按钮就在重置/清除按钮的下方。

现在我们可以绘制按钮。在`LiveDrawingView`类的`draw`方法中添加以下两行代码：

```kt
// Draw the buttons
mCanvas.drawRect(mResetButton, mPaint);
mCanvas.drawRect(mTogglePauseButton, mPaint);
```

新代码使用了`drawRect`方法的重写版本，我们只需将两个`RectF`实例与通常的`Paint`实例一起传递进去。我们的按钮现在将被绘制到屏幕上。

我们将在本章后面看到如何与这些略显粗糙的按钮进行交互。

# 实现粒子系统效果

粒子系统是控制粒子的系统。在我们的情况下，`ParticleSystem`是一个我们将编写的类，它将生成`Particle`类的实例（许多实例），从而创建一个简单的爆炸效果。

这是一张由粒子系统控制的一些粒子的图像：

![图 22.1 - 粒子系统效果](img/Figure_22.01_B16773.jpg)

图 22.1 - 粒子系统效果

为了澄清，每个彩色方块都是`Particle`类的一个实例，所有`Particle`实例都由`ParticleSystem`类控制和持有。此外，用户将通过用手指绘制来创建多个（数百个）`ParticleSystem`实例。粒子将显示为点或块，直到用户点击暂停按钮时才会活跃起来。我们将仔细检查代码，以便您能够在代码中设置`Particle`和`ParticleSystem`实例的大小、颜色、速度和数量。

注意

读者可以在屏幕上添加额外的按钮，以允许用户更改这些属性作为应用程序的一个特性。

我们将从编写`Particle`类开始。

## 编写`Particle`类

按照以下代码中所示的`import`语句、成员变量和构造方法添加：

```kt
import android.graphics.PointF;
class Particle {
    PointF mVelocity;
    PointF mPosition;
    Particle(PointF direction)
    {
        mVelocity = new PointF();
        mPosition = new PointF();
        // Determine the direction
        mVelocity.x = direction.x;
        mVelocity.y = direction.y;
    }
}
```

我们有两个成员变量：一个用于速度，一个用于位置。它们都是`PointF`对象。`PointF`包含两个浮点值。位置很简单；它只是一个水平和垂直值。速度值值得更详细解释。`PointF`中的两个值将是速度，一个是水平的，另一个是垂直的。这两个速度的组合将意味着一个方向。

注意

在构造函数中，两个新的`PointF`对象被实例化，并且`mVeleocity`的`x`和`y`值被初始化为由`PointF direction`参数传入的值。注意值是如何从`direction`复制到`mVelocity`的。现在，`PointF mVelocity`不是作为参数传入的`PointF`的引用。每个`Particle`实例将从`direction`复制值（对于每个实例它们将是不同的），但`mVelocity`与`direction`没有持久的连接。

接下来，添加以下三种方法，然后我们可以讨论它们：

```kt
void update(float fps)
{
   // Move the particle
   mPosition.x += mVelocity.x;
   mPosition.y += mVelocity.y;
}
void setPosition(PointF position)
{
   mPosition.x = position.x;
   mPosition.y = position.y;
}
PointF getPosition()
{
   return mPosition;
}
```

也许并不奇怪，有一个`update`方法。`Particle`实例的`update`方法将由`ParticleSystem`类的`update`方法在应用程序的每一帧调用，而`ParticleSystem`类的`update`方法将由`LiveDrawingView`类（在`update`方法中）调用，我们将在本章后面编写。

在`update`方法中，使用`mVelocity`的相应值更新`mPosition`的水平和垂直值。

注意

请注意，我们在更新中没有使用当前帧速率。如果您想确保粒子以完全正确的速度飞行，可以修改这一点。但是所有速度都将是随机的。增加这个额外的计算并没有太多好处（对于每个粒子）。然而，正如我们很快将看到的，`ParticleSystem`类将需要考虑当前每秒帧数，以测量它应该运行多长时间。

接下来，我们编写了`setPosition`方法。请注意，该方法接收`PointF`，用于设置初始位置。`ParticleSystem`类将在触发效果时传递此位置。

最后，我们有`getPosition`方法。我们需要这个方法，以便`ParticleSystem`类可以在正确的位置绘制所有粒子。我们本可以在`Particle`类中添加一个`draw`方法，而不是`getPosition`方法，并让`Particle`类自己绘制。在这个实现中，两种选项都没有特别的好处。

现在我们可以继续`ParticleSysytem`类。

## 编写`ParticleSystem`类。

`ParticleSystem`类比`Particle`类有更多的细节，但仍然相当简单。记住我们需要用这个类实现的目标：保存、生成、更新和绘制一堆（相当大的一堆）`Particle`实例。

添加以下成员和`import`语句：

```kt
import android.graphics.Canvas;
import android.graphics.Color;
import android.graphics.Paint;
import android.graphics.PointF;
import java.util.ArrayList;
import java.util.Random;
class ParticleSystem {
    private float mDuration;
    private ArrayList<Particle> mParticles;
    private Random random = new Random();
    boolean mIsRunning = false;

}
```

我们有四个成员变量：首先，一个名为`mDuration`的`float`变量，它将被初始化为我们希望效果运行的秒数。名为`mParticles`的`ArrayList`实例保存`Particle`实例，并将保存我们实例化的所有`Particle`对象。

称为`random`的`Random`实例被创建为成员，因为我们需要生成如此多的随机值，每次创建一个新对象都会减慢速度。

最后，`mIsRunning`布尔值将跟踪粒子系统当前是否正在显示（更新和绘制）。

现在我们可以编写`init`方法。每当我们想要一个新的`ParticleSystem`时，将调用此方法。请注意，唯一的参数是一个名为`numParticles`的`int`参数。

当我们调用`init`时，我们可以有一些乐趣初始化疯狂数量的粒子。添加`init`方法，然后我们将更仔细地查看代码：

```kt
void init(int numParticles){
   mParticles = new ArrayList<>();
   // Create the particles
   for (int i = 0; i < numParticles; i++){
         float angle = (random.nextInt(360)) ;
         angle = angle * 3.14f / 180.f;
         // Option 1 - Slow particles
         //float speed = (random.nextFloat()/10);
         // Option 2 - Fast particles
         float speed = (random.nextInt(10)+1);
         PointF direction;
         direction = new PointF((float)Math.cos(angle) * 
                     speed, (float)Math.sin(angle) * 
                     speed);
         mParticles.add(new Particle(direction));
   }
}
```

`init`方法只包括一个`for`循环，完成所有工作。`for`循环从零到`numParticles-1`运行。

首先，生成一个介于零和 359 之间的随机数，并存储在名为`angle`的`float`变量中。接下来，进行一些数学运算，将`angle`乘以`3.14/180`。这将角度从度转换为弧度测量，这是`Math`类在稍后将要使用的。

然后我们生成另一个 1 到 10 之间的随机数，并将结果赋给一个名为`speed`的`float`变量。

注意

我已经添加了注释，建议在代码的这部分中使用不同的值。我在`ParticleSystem`类的几个地方都这样做了，当我们到达章节的末尾时，我们将乐趣地改变这些值，看看对绘图应用有什么影响。

现在我们有了一个随机角度和速度，我们可以将它们转换并组合成一个矢量，该矢量可以在`Particle`类的`update`方法中使用，以更新其每帧的位置。

注意

矢量是一个确定方向和速度的值。我们的矢量存储在`direction`对象中，直到传递到`Particle`构造函数中。矢量可以是多维的。我们的矢量是二维的，因此定义了 0 到 359 度之间的航向和 1 到 10 之间的速度。您可以在我的网站上阅读更多关于矢量、航向、正弦和余弦的内容：[`gamecodeschool.com/essentials/calculating-heading-in-2d-games-using-trigonometric-functions-part-1/`](http://gamecodeschool.com/essentials/calculating-heading-in-2d-games-using-trigonometric-functions-part-1/)。

使用`Math.sin`和`Math.cos`创建矢量的单行代码，我决定不完全解释，因为其中的魔法部分在以下公式中发生：

+   角度的余弦 * `speed`

+   角度的正弦 * `speed`

这也在`Math`类提供的余弦和正弦函数的隐藏计算中部分发生。如果您想了解它们的全部细节，请参阅前面的提示框。

最后，创建一个新的`Particle`，然后将其添加到`mParticles ArrayList`实例中。

接下来，我们将编写`update`方法。请注意，`update`方法确实需要当前帧速率作为参数。编写如下所示的`update`方法：

```kt
void update(long fps){
   mDuration -= (1f/fps);
   for(Particle p : mParticles){
         p.update(fps);
   }
   if (mDuration < 0)
   {
         mIsRunning = false;
   }
}
```

`update`方法内部发生的第一件事是减去`mDuration`的经过时间。请记住，`fps`参数是每秒帧数，所以`1/fps`会给出一个作为秒的分数值。

接下来是一个增强的`for`循环，调用`mParticles` `ArrayList`实例中每个`Particle`实例的`update`方法。

最后，代码检查粒子效果是否已经完成，使用`if(mDuration < 0)`，如果是，则将`mIsRunning`设置为`false`。

现在我们可以编写`emitParticles`方法，它将使每个`Particle`实例运行。这不应与`init`混淆，后者创建所有新的粒子并赋予它们速度。`init`方法将在用户开始交互之前调用一次，而`emitParticles`方法将在每次需要启动效果时调用，用户在屏幕上绘制时。

添加`emitParticles`方法：

```kt
void emitParticles(PointF startPosition){
   mIsRunning = true;
   // Option 1 - System lasts for half a minute
   //mDuration = 30f;
   // Option 2 - System lasts for 2 seconds
   mDuration = 3f;
   for(Particle p : mParticles){
         p.setPosition(startPosition);
   }
}
```

首先，请注意将所有粒子的起始位置作为参数传递给`PointF`引用。所有粒子将从完全相同的位置开始，然后根据它们各自的速度每帧扩散。

`mIsRunning`布尔值设置为`true`，`mDuration`设置为`1f`，所以效果将持续一秒，增强的`for`循环调用`setPosition`来移动每个粒子到起始坐标。

我们`ParticleSysytem`类的最后一个方法是`draw`方法，它将展示效果的全部荣耀。该方法接收一个`Canvas`实例和一个`Paint`实例的引用，因此它可以在`LiveDrawingView`类的`draw`方法中锁定的相同画布上绘制。

添加`draw`方法：

```kt
void draw(Canvas canvas, Paint paint){
         for (Particle p : mParticles) {
                // Option 1 - Coloured particles
                //paint.setARGB(255, random.nextInt(256),
                            //random.nextInt(256),
                            //random.nextInt(256));
                // Option 2 - White particles
                paint.setColor(
                Color.argb(255,255,255,255));
                // How big is each particle?
                float sizeX = 0;
                float sizeY = 0;
                // Option 1 - Big particles
                //sizeX = 25;
                //sizeY = 25;
                // Option 2 - Medium particles
                sizeX = 10;
                sizeY = 10;
                // Option 3 - Tiny particles
                //sizeX = 1;
                //sizeY = 1;
                // Draw the particle
                // Option 1 - Square particles
                //canvas.drawRect(p.getPosition().x, 
                            //p.getPosition().y,
                            //p.getPosition().x + sizeX,
                            //p.getPosition().y + sizeY,
                            //paint);
                // Option 2 - Circle particles
                canvas.drawCircle(p.getPosition().x, 
                            p.getPosition().y,
                            sizeX, paint);
         }
}
```

增强的`for`循环遍历`mParticles` `ArrayList`实例中的每个`Particle`实例。依次使用`drawRect`方法和`getPosition`方法绘制每个`Particle`。请注意调用`paint.setARGB`方法。您将看到我们随机生成每个颜色通道。

注意

请注意评论中我建议了不同的代码更改选项，这样在完成编码后我们就可以玩得更开心。

我们现在可以开始让粒子系统工作了。

## 在`LiveDrawingView`类中生成粒子系统

添加一个充满系统的`ArrayList`实例和一些其他成员来跟踪事物。在现有注释所指示的位置添加突出显示的代码：

```kt
// The particle systems will be declared here later
private ArrayList<ParticleSystem> 
          mParticleSystems = new ArrayList<>();
private int mNextSystem = 0;
private final int MAX_SYSTEMS = 1000;
private int mParticlesPerSystem = 100;
```

将`ArrayList`类导入如下：

```kt
import java.util.ArrayList;
```

现在我们可以跟踪多达 1000 个每个系统中有 100 个粒子的粒子系统。随意尝试调整这些数字。

注意

在现代设备上，您可以运行数百万个粒子而不会遇到任何问题，但在模拟器上，处理数十万个粒子就会有些吃力。

通过添加以下突出显示的代码在构造函数中初始化系统：

```kt
// Initialize the particles and their systems
for (int i = 0; i < MAX_SYSTEMS; i++) {
   mParticleSystems.add(new ParticleSystem());
   mParticleSystems.get(i).init(mParticlesPerSystem);
}
```

该代码循环遍历`ArrayList`实例，对每个`ParticleSystem`实例调用构造函数，然后调用`init`方法。

通过在`update`方法中添加以下突出显示的代码，为循环的每一帧更新系统：

```kt
private void update() {
   // Update the particles
   for (int i = 0; i < mParticleSystems.size(); i++) {
          if (mParticleSystems.get(i).mIsRunning) {
                 mParticleSystems.get(i).update(mFPS);
          }
   }
}
```

前面的代码循环遍历每个`ParticleSystem`实例，首先检查它们是否活动，然后调用`update`方法并传入当前的每秒帧数。

通过将此突出显示的代码添加到`draw`方法中，为循环的每一帧绘制系统：

```kt
// Choose a color to paint with
mPaint.setColor(Color.argb(255, 255, 255, 255));
// Choose the font size
mPaint.setTextSize(mFontSize);
// Draw the particle systems
for (int i = 0; i < mNextSystem; i++) {
     mParticleSystems.get(i).draw(mCanvas, mPaint);
}
// Draw the buttons
mCanvas.drawRect(mResetButton, mPaint);
mCanvas.drawRect(mTogglePauseButton, mPaint);
```

前面的代码循环遍历`mParticleSystems`，对每个调用`draw`方法。当然，我们实际上还没有生成任何实例。为此，我们需要学习如何响应屏幕交互。

# 处理触摸

要开始，请将`OnTouchEvent`方法添加到`LiveDrawingView`类中：

```kt
@Override
public boolean onTouchEvent(MotionEvent motionEvent) {

   return true;
}
```

这是一个重写的方法，每当用户与屏幕交互时，Android 都会调用它。查看`OnTouchEvent`方法的唯一参数。

使用以下代码行导入`MotionEvent`类：

```kt
import android.view.MotionEvent;
```

原来`motionEvent`中隐藏了大量数据，这些数据包含了刚刚发生的触摸的详细信息。操作系统将其发送给我们，因为它知道我们可能需要其中的一些数据。

请注意，我说的是*一些*。`MotionEvent`类非常庞大。它包含了数十种方法和变量。

注意

在这个项目中，我们将揭示`MotionEvent`类的一些细节。您可以在这里完整地探索`MotionEvent`类：[`stuff.mit.edu/afs/sipb/project/android/docs/reference/android/view/MotionEvent.html`](https://stuff.mit.edu/afs/sipb/project/android/docs/reference/android/view/MotionEvent.html)。请注意，完成此项目并不需要进行进一步的研究。

目前，我们只需要知道在玩家的手指在屏幕上移动、触摸屏幕或从屏幕上移开时的精确时刻的屏幕坐标。

我们将使用`motionEvent`中包含的一些变量和方法，包括以下内容。

+   `getAction`方法，意料之中地“获取”执行的动作。不幸的是，它以稍微编码的格式提供这些信息，这解释了其他一些变量的必要性。

+   `ACTION_MASK`变量提供一个称为掩码的值，借助一些更多的 Java 技巧，可以用来过滤`getAction`的数据。

+   `ACTION_UP`变量，我们可以使用它来比较并查看执行的动作是否是我们想要响应的动作（从屏幕上移开手指）。

+   `ACTION_DOWN`变量，我们可以使用它来比较并查看执行的动作是否是我们想要响应的动作。

+   `ACTION_MOVE`变量，我们可以用它来比较并查看执行的动作是否是移动/拖动。

+   `getX`方法告诉我们事件发生的水平浮点坐标。

+   `getY`方法告诉我们事件发生的垂直浮点坐标。

举个具体的例子，假设我们需要使用`ACTION_MASK`过滤`getAction`方法返回的数据，并查看结果是否与`ACTION_UP`相同。如果是，那么我们知道用户刚刚从屏幕上移开手指，也许是因为他们刚刚点击了一个按钮。一旦我们确定事件是正确类型的，我们就需要使用`getX`和`getY`方法找出事件发生的位置。

最后一个复杂之处在于，“Java 诡计”我所指的是`&`位运算符，不要与我们一直与`if`关键字一起使用的逻辑`&&`运算符混淆。

`&`位运算符检查两个值中的每个对应部分是否为真。这是在使用`ACTION_MASK`与`getAction`时所需的过滤器。

注意

理智检查。我不愿详细讨论`MotionEvent`和位运算符。完全可以完成整本书甚至一个专业质量的交互式应用程序，而不需要完全理解它们。如果你知道我们在下一节中写的代码行确定了玩家刚刚触发的事件类型，那就足够了。我只是猜想像你这样挑剔的读者可能想要了解其中的细节。总之，如果你理解位运算符，很好，你可以继续。如果你不理解，没关系，你仍然可以继续。如果你对位运算符感兴趣（有很多种），你可以在这里阅读更多关于它们的信息：[`en.wikipedia.org/wiki/Bitwise_operation`](https://en.wikipedia.org/wiki/Bitwise_operation)。

现在我们可以编写`onTouchEvent`方法并查看所有`MotionEvent`的操作。

## 编写`onTouchEvent`方法

通过在`onTouchEvent`方法中添加以下突出显示的代码来处理用户在屏幕上移动手指：

```kt
// User moved a finger while touching screen
   if ((motionEvent.getAction() &
                 MotionEvent.ACTION_MASK)
                 == MotionEvent.ACTION_MOVE) {
          mParticleSystems.get(mNextSystem).emitParticles(
                       new PointF(motionEvent.getX(),
                                     motionEvent.getY()));
          mNextSystem++;
          if (mNextSystem == MAX_SYSTEMS) {
                 mNextSystem = 0;
          }
   }
   return true;
```

添加以下代码行以导入`PointF`类：

```kt
import android.graphics.PointF;
```

`if`条件检查事件类型是否是用户移动手指。如果是，那么`mParticleSystems`中的下一个粒子系统将调用其`emitParticles`方法。之后，`mNextSystem`变量递增，并进行测试以查看是否是最后一个粒子系统。如果是，那么`mNextSystem`将被设置为零，准备在下次需要时重新使用现有的粒子系统。

通过在我们刚讨论过的代码之后并在我们已经编写的`return`语句之前添加以下突出显示的代码来处理用户按下按钮之一：

```kt
// Did the user touch the screen
   if ((motionEvent.getAction() &
                 MotionEvent.ACTION_MASK)
                 == MotionEvent.ACTION_DOWN) {
// User pressed the screen see if it was in a 
          button
          if (mResetButton.contains(motionEvent.getX(),
                       motionEvent.getY())) {
                 // Clear the screen of all particles
                 mNextSystem = 0;
          }
// User pressed the screen see if it was in a 
          button
          if (mTogglePauseButton.contains
          (motionEvent.getX(), motionEvent.getY())) {
                 mPaused = !mPaused;
          }
   }
   return true;
```

`if`语句的条件检查用户是否点击了屏幕。如果是，那么`RectF`类的`contains`方法与`getX`和`getY`方法一起被用来检查这次按压是否在我们自定义按钮的范围内。如果按下重置按钮，所有的粒子都会消失，因为`mNextSystem`被设置为零。如果按下暂停按钮，那么`mPaused`的值将被切换，导致线程中的`update`方法停止/开始被调用。

## 完成 HUD

将以下突出显示的代码添加到`printDebuggingText`方法中：

```kt
// We will add more code here in the next chapter
mCanvas.drawText("Systems: " + mNextSystem,
10, mFontMargin + debugStart + debugSize * 2, 
          mPaint);
mCanvas.drawText("Particles: " + mNextSystem * mParticlesPerSystem,
10, mFontMargin + debugStart + debugSize * 3, 
          mPaint);
```

这段代码将在屏幕上打印一些有趣的统计数据，告诉我们当前绘制了多少粒子和系统。

警告

这个应用程序会产生明亮的闪烁颜色。这可能会引起光敏性癫痫的人感到不适或发作。请谨慎阅读。您可能只想阅读这个项目的理论，而不运行已完成的项目。

# 运行应用程序

现在我们可以看到实时绘图应用程序的运行并尝试一些我们在代码中注释掉的不同选项。

使用小型、圆形、彩色、快速粒子运行应用程序。只需在屏幕上轻点几下：

![图 22.2 – 点击屏幕](img/Figure_22.2_B16773.jpg)

图 22.2 – 点击屏幕

然后恢复绘图：

![图 22.3 – 点击结果](img/Figure_22.3_B16773.jpg)

图 22.3 – 点击结果

使用小型、白色、方形、缓慢、长时间的粒子进行儿童风格的绘图：

![图 22.4 – 儿童风格的绘图](img/Figure_22.4_B16773.jpg)

图 22.4 – 儿童风格的绘图

然后取消绘图暂停，等待 20 秒，直到绘图活跃起来并发生变化：

![图 22.5 – 儿童风格的绘图结果](img/Figure_22.5_B16773.jpg)

图 22.5 – 儿童风格的绘图结果

在我们进行下一个项目之前，Live Drawing 应用程序为我们提供了一个很好的机会，可以探索 Android Studio 的另一个功能。

# Android Studio Profiler 工具

Android Studio Profiler 工具非常复杂和深入。但是，使用它来进行一些真正重要的测量非常简单。我们可以看到我们的应用程序使用了设备资源的多少，因此可以尝试提高应用程序的效率，使其运行更高效，并且使用更少的资源。资源包括 CPU 和内存使用率。

代码优化超出了本书的范围，但是我们开始监视应用程序性能的方式是一个很好的介绍。从主 Android Studio 菜单中选择**View**，然后选择**Tool Windows** | **Profiler**。

您将在 Android Studio 的下部区域看到以下窗口：

![图 22.6 – Android Studio 窗口](img/Figure_22.6_B16773.jpg)

图 22.6 – Android Studio 窗口

要开始使用 Profiler 工具，请运行 Live Drawing 应用程序。Profiler 工具应该开始显示图表和数据，如下图所示。

根据您的 PC 防火墙软件的配置，您可能需要允许 Profiler 工具运行。此外，您可能需要在**Profiler**窗口左上角的**+**图标上单击，然后选择您的 AVD，以便 Profiler 工具连接到：

![图 22.7 – 实时图表数据](img/Figure_22.7_B16773.jpg)

图 22.7 – 实时图表数据

在上图中，我们可以看到 CPU 使用率、内存使用率、网络使用率和能量/电池使用率的实时图表数据。我们将重点关注 CPU 和内存使用率。

将鼠标悬停在**CPU**行，然后悬停在**MEMORY**行上，以查看每个指标的弹出详细信息。下图显示了我 PC 上这两个指标的详细信息，经过了 Photoshop 处理：

![图 22.8 – 每个指标的弹出详细信息](img/Figure_22.8_B16773.jpg)

图 22.8 – 每个指标的弹出详细信息

您可能会看到与我不同的值。前面的图表显示大约四分之一的 CPU 正在使用，大约使用了 121MB 的 RAM。

接下来，让我们稍微修改我们的代码并观察效果。在`LiveDrawingView`类中，编辑`mParticlesPerSystem`成员变量的初始化：

```kt
private int mParticlesPerSystem = 100;
```

将其更改为：

```kt
private int mParticlesPerSystem = 1000;
```

我们现在将每个系统的粒子数量增加了 10 倍。我们这样做是为了在分析器数据中获得一个峰值，因为我们现在将使用应用程序来绘制一些粒子系统。

当您再次运行应用程序时，通过在屏幕上移动手指/指针来绘制大量的粒子系统。请注意，当您在屏幕上绘制一些粒子系统时，CPU 使用率会急剧上升，尽管可能没有您预期的那么多。当粒子移动时，我的 CPU 使用率急剧上升到接近 40%，然后回落到 25%以上。如果您以前从未使用过类似分析器的工具，更令人惊讶的是内存使用几乎没有变化。

我们得到这样的结果的原因是，成千上万个粒子的计算占用了相当大量的 CPU。然而，在屏幕上绘制粒子并不需要增加内存。原因在于应用程序的内存都是在执行开始时分配的。无论粒子当前是否显示给用户都不重要。

这一小节并不打算深入探讨如何优化我们的图形或 CPU 密集型应用程序；它只是想介绍一下，您可能希望将优化添加到您进一步调查的事项列表中。

# 总结

在本章中，我们看到了如何向我们的实时系统添加成千上万个独立的实体。这些实体由`ParticleSystem`类控制，而`ParticleSystem`类又与游戏循环进行交互和控制。由于游戏循环在一个线程中运行，我们看到用户仍然可以无缝地与屏幕进行交互，操作系统通过`onTouchEvent`方法向我们发送这些交互的详细信息。

在下一章中，当我们探讨如何播放音效时，我们的应用程序最终会变得有些喧闹；我们还将学习如何检测不同版本的安卓系统。
