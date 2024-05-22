# 第十九章：动画和插值

在这里，我们将看到如何使用`Animation`类使我们的 UI 不那么静态，更有趣。正如我们所期望的那样，Android API 将允许我们用相对简单的代码做一些相当高级的事情，`Animation`类也不例外。

本章大致可分为以下几个部分：

+   介绍了 Android 中动画的工作原理和实现方式

+   介绍了一个我们尚未探索的 UI 小部件`SeekBar`类

+   一个有效的动画应用程序

首先，让我们探索一下 Android 中的动画是如何工作的。

# Android 中的动画

在 Android 中创建动画的常规方式是通过 XML。我们可以编写 XML 动画，然后通过 Kotlin 代码在指定的 UI 小部件上加载和播放它们。因此，例如，我们可以编写一个动画，在三秒内淡入淡出五次，然后在`ImageView`或任何其他小部件上播放该动画。我们可以将这些 XML 动画看作脚本，因为它们定义了类型、顺序和时间。

让我们探索一些可以分配给我们的动画的不同属性，如何在我们的 Kotlin 代码中使用它们，最后，我们可以制作一个漂亮的动画应用程序来尝试一切。

## 在 XML 中设计酷炫的动画

我们已经了解到 XML 不仅可以用来描述 UI 布局，还可以用来描述动画，但让我们确切地了解一下。我们可以为动画的属性值指定起始和结束外观的小部件。然后，我们的 Kotlin 代码可以通过引用包含动画的 XML 文件的名称来加载 XML，将其转换为可用的 Kotlin 对象，再次，与 UI 布局类似。

许多动画属性成对出现。以下是一些我们可以使用的动画属性对的快速查看。在查看了一些 XML 后，我们将看到如何使用它。

### 淡入淡出

Alpha 是透明度的度量。因此，通过说明起始`fromAlpha`和结束`toAlpha`值，我们可以淡入淡出物品。值`0.0`是不可见的，`1.0`是对象的正常外观。在两者之间稳定移动会产生淡入效果：

```kt
<alpha
   android:fromAlpha = "0.0"
   android:toAlpha = "1.0" />
```

### 移动它，移动它

我们可以使用类似的技术在 UI 中移动对象；`fromXDelta`和`toXDelta`的值可以设置为被动画化对象大小的百分比。

以下代码将使对象从左到右移动，距离等于对象本身的宽度：

```kt
<translate     
android:fromXDelta = "-100%"
android:toXDelta = "0%"/>
```

此外，还有用于上下移动动画的`fromYDelta`和`toYDelta`属性。

### 缩放或拉伸

`fromXScale`和`toXScale`属性将增加或减少对象的比例。例如，以下代码将使运行动画的对象从正常大小变为不可见：

```kt
<scale
android:fromXScale = "1.0"
android:fromYScale = "0.0"/>
```

作为另一个例子，我们可以使用`android:fromYScale = "0.1"`将对象缩小到通常大小的十分之一，或者使用`android:fromYScale = "10.0"`将其放大十倍。

### 控制持续时间

当然，如果这些动画只是立即结束，那将不会特别有趣。因此，为了使我们的动画更有趣，我们可以设置它们的持续时间（以毫秒为单位）。毫秒是一秒的千分之一。我们还可以通过设置`startOffset`属性（也是以毫秒为单位）来使时间更容易，特别是与其他动画相关。

下一个代码将在我们启动动画的 1/3 秒后开始（在代码中），并且需要 2/3 秒才能完成：

```kt
android:duration = "666"
android:startOffset = "333"
```

### 旋转动画

如果要使某物旋转，只需使用`fromDegrees`和`toDegrees`属性。下一个代码，可能可以预测，将使小部件在一个完整的圆圈中旋转，因为当然，一个圆圈有 360 度：

```kt
<rotate android:fromDegrees = "360"
        android:toDegrees = "0"
/>
```

### 重复动画

在一些动画中，重复可能很重要，也许是摇摆或抖动效果，因此我们可以添加一个`repeatCount`属性。此外，我们可以通过设置`repeatMode`属性来指定动画的重复方式。

以下代码将重复一个动画 10 次，每次都会反转动画的方向。`repeatMode`属性是相对于动画的当前状态。这意味着，如果你将一个按钮从 0 度旋转到 360 度，例如，动画的第二部分（第一次重复）将以相反的方式旋转，从 360 度回到 0 度。动画的第三部分（第二次重复）将再次反转，并从 0 度旋转到 360 度：

```kt
android:repeatMode = "reverse"
android:repeatCount = "10"
```

### 将动画的属性与集合结合

要组合这些效果的组，我们需要一组属性。这段代码展示了我们如何将我们刚刚看到的所有先前的代码片段组合成一个实际的 XML 动画，它将被编译：

```kt
<?xml version="1.0" encoding="utf-8"?>
<set 
     ...All our animations go here
</set>
```

到目前为止我们还没有看到任何 Kotlin 来使这些动画生动起来。让我们现在来解决这个问题。

## 实例化动画并使用 Kotlin 代码控制它们

下面的代码片段展示了我们如何声明一个`Animation`类型的对象，用一个名为`fade_in.xml`的 XML 文件中包含的动画来初始化它，并在一个`ImageView`小部件上启动动画。我们很快将在一个项目中这样做，并且还会看到我们可以放置 XML 动画的地方：

```kt
// Declare an Animation object
var animFadeOut: Animation? = null

// Initialize it 
animFadeIn = AnimationUtils.loadAnimation(
                this, R.anim.fade_in)

// Start the animation on the ImageView
// with an id property set to imageView
imageView.startAnimation(animFadeIn)
```

我们已经有了相当强大的动画和控制特性，比如时间控制。但是 Android API 还给了我们更多的东西。

## 更多动画特性

我们可以监听动画的状态，就像我们可以监听按钮的点击一样。我们还可以使用**插值器**使我们的动画更加生动和愉悦。让我们先看看监听器。

### 监听器

如果我们实现`AnimationListener`接口，我们确实可以通过覆盖告诉我们发生了什么的三个函数来监听动画的状态。然后我们可以根据这些事件来采取行动。

`OnAnimationEnd`宣布动画结束，`onAnimationRepeat`在每次动画开始重复时调用，而-也许可以预料到-`onAnimationStart`在动画开始动画时调用。如果在动画 XML 中设置了`startOffset`，这可能不是调用`startAnimation`时的同一时间：

```kt
override fun onAnimationEnd(animation: Animation) {   
   // Take some action here

}

override fun onAnimationStart(animation: Animation) {

   // Take some action here

}

override fun onAnimationRepeat(animation: Animation){

   // Take some action here

}
```

我们将在 Animations 演示应用程序中看到`AnimationListener`的工作原理，并且我们还将把另一个小部件`SeekBar`投入使用。

### 动画插值器

如果你能回想起高中时的情景，你可能会记得关于计算加速度的激动人心的课程。如果我们以恒定的速度对某物进行动画处理，乍一看，事情可能看起来还不错。如果我们将动画与另一个使用渐进加速的动画进行比较，那么后者几乎肯定会更令人愉悦地观看。

有可能，如果我们没有被告知两个动画之间唯一的区别是一个使用了加速度，另一个没有，我们可能无法说出*为什么*我们更喜欢它。我们的大脑更容易接受符合我们周围世界规范的事物。因此，添加一点真实世界的物理，比如加速和减速，可以改善我们的动画。

然而，我们最不想做的事情是开始做一堆数学计算，只是为了将一个按钮滑动到屏幕上或者让一些文本在圆圈中旋转。

这就是**插值器**的用武之地。它们是我们可以在我们的 XML 中用一行代码设置的动画修改器。

一些插值器的例子是`accelerate_interpolator`和`cycle_interpolator`：

```kt
android:interpolator="@android:anim/accelerate_interpolator"
android:interpolator="@android:anim/cycle_interpolator"/>
```

接下来我们将投入使用一些插值器，以及一些 XML 动画和相关的 Kotlin 代码。

### 提示

您可以在 Android 开发者网站上了解有关插值器和 Android `Animation`类的更多信息：[`developer.android.com/guide/topics/resources/animation-resource.html`](http://developer.android.com/guide/topics/resources/animation-resource.html)。

# 动画演示应用程序-介绍 SeekBar

这就足够的理论了，尤其是对于应该如此明显的东西。让我们构建一个动画演示应用程序，探索我们刚刚讨论过的一切，以及更多内容。

这个应用程序涉及许多不同文件中少量的代码。因此，我已经尽量清楚地说明了哪些代码在哪个文件中，这样您就可以跟踪发生了什么。这也将使我们为这个应用程序编写的 Kotlin 更容易理解。

该应用程序将演示旋转、淡入淡出、平移、动画事件、插值和使用`SeekBar`小部件控制持续时间的功能。解释`SeekBar`的最佳方法是构建它，然后观察它的运行情况。

## 布局动画演示

使用**空活动**模板创建一个名为`Animation Demo`的新项目，将所有其他设置保持为通常的设置。如果您希望通过复制和粘贴布局、代码或动画 XML 来加快速度，可以在`Chapter19`文件夹中找到所有内容。

使用完成布局的参考截图来帮助您完成接下来的步骤：

![布局动画演示](img/B12806_19_10.jpg)

以下是您可以为此应用程序布局 UI 的方法：

1.  在编辑器窗口的设计视图中打开`activity_main.xml`。

1.  删除默认的**Hello world!** `TextView`。

1.  在布局的顶部中心部分添加一个**ImageView**。使用之前的参考截图来指导您。在弹出的**资源**窗口中选择**项目** | **ic_launcher**，使用`@mipmap/ic_launcher`来在`ImageView`中显示 Android 机器人。

1.  将`ImageView`的`id`属性设置为`imageView`。

1.  在`ImageView`的正下方，添加一个`TextView`。将`id`设置为`textStatus`。我通过拖动其边缘（而不是约束手柄）使我的`TextView`变大，并将其`textSize`属性更改为`40sp`。到目前为止，布局应该看起来像下一个截图：![布局动画演示](img/B12806_19_01.jpg)

1.  现在我们将在布局中添加大量的**Button**小部件。确切的定位并不重要，但稍后在教程中为它们添加的确切`id`属性值是重要的。按照下一个截图的指示，在布局中放置 12 个按钮。修改`text`属性，使您的按钮与下一个截图中的按钮具有相同的文本。如果截图不够清晰，`text`属性将在下一步中具体详细说明：![布局动画演示](img/B12806_19_02.jpg)

### 提示

为了加快布局按钮的过程，首先大致布局它们，然后从下一步中添加`text`属性，最后微调按钮位置以获得整洁的布局。

1.  按照截图中的方式添加`text`值；从左到右，从上到下，这里是所有的值：`淡入`、`淡出`、`淡入淡出`、`放大`、`缩小`、`左右`、`右左`、`上下`、`弹跳`、`闪烁`、`向左旋转`和`向右旋转`。

1.  从左侧的调色板中的**小部件**类别中添加一个`SeekBar`小部件，将`id`属性设置为`seekBarSpeed`，将`max`属性设置为`5000`。这意味着`SeekBar`小部件将在用户从左向右拖动时保持一个值在 0 到 5000 之间。我们将看到如何读取和使用这些数据。

1.  我们想要让`SeekBar`小部件变得更宽。为了实现这一点，您可以使用与任何小部件相同的技术；只需拖动小部件的边缘。然而，由于`SeekBar`小部件相当小，很难增加其大小而不小心选择约束手柄。为了克服这个问题，通过按住*Ctrl*键并向前滚动鼠标滚轮来放大设计视图。然后，您可以抓住`SeekBar`小部件的边缘，而不触摸约束手柄。我在下一个截图中展示了这一点：![布局动画演示](img/B12806_19_07.jpg)

1.  现在，在`SeekBar`小部件的右侧添加一个`TextView`小部件，并将其`id`属性设置为`textSeekerSpeed`。这一步，结合前两步，应该看起来像这张截图：![布局动画演示](img/B12806_19_03.jpg)

1.  微调位置，使其看起来像这些步骤开始时的参考截图，然后单击**推断约束**按钮以锁定位置。当然，如果你想练习，你也可以手动完成。这是一个包含所有约束的截图：![布局动画演示](img/B12806_19_08.jpg)

1.  接下来，根据你已经设置的文本属性，为按钮添加以下`id`属性。如果在输入这些值时询问是否要**更新用法…**，请选择**是**：

| **现有文本属性** | **要设置的 id 属性的值** |
| --- | --- |
| 淡入 | `btnFadeIn` |
| 淡出 | `btnFadeOut` |
| 淡入淡出 | `btnFadeInOut` |
| 放大 | `btnZoomIn` |
| 缩小 | `btnZoomOut` |
| 左右 | `btnLeftRight` |
| 右左 | `btnRightLeft` |
| 上下 | `btnTopBottom` |
| 弹跳 | `btnBounce` |
| 闪烁 | `btnFlash` |
| 旋转左侧 | `btnRotateLeft` |
| 旋转右侧 | `btnRotateRight` |

当我们在几节时间内编写`MainActivity`类时，我们将看到如何使用这个新来的 UI(`SeekBar`)。

## 编写 XML 动画

右键单击**res**文件夹，然后选择**新建 | Android 资源目录**。在`目录名称：`字段中输入`anim`，然后左键单击**确定**。

现在右键单击新的**anim**目录，然后选择**新建 | 动画资源文件**。在**文件名：**字段中，键入`fade_in`，然后左键单击**确定**。删除内容并添加以下代码来创建动画：

```kt
<?xml version="1.0" encoding="utf-8"?>
<set 
   android:fillAfter="true" >

   <alpha
   android:fromAlpha = "0.0"
   android:interpolator = 
              "@android:anim/accelerate_interpolator"

   android:toAlpha="1.0" />
</set>
```

右键单击**anim**目录，然后选择**新建 | 动画资源文件**。在`文件名：`字段中，键入`fade_out`，然后左键单击**确定**。删除内容并添加以下代码来创建动画：

```kt
<?xml version="1.0" encoding="utf-8"?>
<set 
   android:fillAfter = "true" >

   <alpha  
         android:fromAlpha = "1.0"
         android:interpolator = 
              "@android:anim/accelerate_interpolator"

   android:toAlpha = "0.0" />
</set>
```

右键单击**anim**目录，然后选择**新建 | 动画资源文件**。在`文件名：`字段中，键入`fade_in_out`，然后左键单击**确定**。删除内容并添加以下代码来创建动画：

```kt
<?xml version="1.0" encoding="utf-8"?>
<set 
    android:fillAfter = "true" >

    <alpha
          android:fromAlpha="0.0"
          android:interpolator = 
          "@android:anim/accelerate_interpolator"

          android:toAlpha = "1.0" />

    <alpha
       android:fromAlpha = "1.0"
          android:interpolator = 
         "@android:anim/accelerate_interpolator"

         android:toAlpha = "0.0" />
</set>
```

右键单击**anim**目录，然后选择**新建 | 动画资源文件**。在`文件名：`字段中，键入`zoom_in`，然后左键单击**确定**。删除内容并添加以下代码来创建动画：

```kt
<set 
    android:fillAfter = "true" >

    <scale
        android:fromXScale = "1"
        android:fromYScale = "1"
        android:pivotX = "50%"
        android:pivotY = "50%"
        android:toXScale = "6"
        android:toYScale = "6" >
    </scale>
</set>
```

右键单击**anim**目录，然后选择**新建 | 动画资源文件**。在`文件名：`字段中，键入`zoom_out`，然后左键单击**确定**。删除内容并添加以下代码来创建动画：

```kt
<?xml version="1.0" encoding="utf-8"?>
<set >
    <scale
        android:fromXScale = "6"
        android:fromYScale = "6"
        android:pivotX = "50%"
        android:pivotY = "50%"
        android:toXScale = "1"
        android:toYScale = "1" >
    </scale>
</set>
```

右键单击**anim**目录，然后选择**新建 | 动画资源文件**。在`文件名：`字段中，键入`left_right`，然后左键单击**确定**。删除内容并添加以下代码来创建动画：

```kt
<?xml version="1.0" encoding="utf-8"?>
<set >
    <translate     

        android:fromXDelta = "-500%"
        android:toXDelta = "0%"/>
</set>
```

再次右键单击**anim**目录，然后选择**新建 | 动画资源文件**。在**文件名：**字段中，键入`right_left`，然后左键单击**确定**。删除整个内容并添加以下代码来创建动画：

```kt
<?xml version="1.0" encoding="utf-8"?>
<set >
    <translate 
        android:fillAfter = "false"
        android:fromXDelta = "500%"
        android:toXDelta = "0%"/>
</set>
```

与以前一样，右键单击**anim**目录，然后选择**新建 | 动画资源文件**。在**文件名：**字段中，键入`top_bot`，然后左键单击**确定**。删除整个内容并添加以下代码来创建动画：

```kt
<?xml version="1.0" encoding="utf-8"?>
<set >
    <translate 
        android:fillAfter = "false"
        android:fromYDelta = "-100%"
        android:toYDelta = "0%"/>
</set>
```

你猜对了；右键单击**anim**目录，然后选择**新建 | 动画资源文件**。在**文件名：**字段中，键入`flash`，然后左键单击**确定**。删除内容并添加以下代码来创建动画：

```kt
<?xml version="1.0" encoding="utf-8"?>
<set >
    <alpha android:fromAlpha = "0.0"
        android:toAlpha = "1.0"
        android:interpolator = 
           "@android:anim/accelerate_interpolator"

        android:repeatMode = "reverse"
        android:repeatCount = "10"/>
</set>
```

还有一些要做 - 右键单击**anim**目录，然后选择**新建 | 动画资源文件**。在**文件名：**字段中，键入`bounce`，然后左键单击**确定**。删除内容并添加以下代码来创建动画：

```kt
<?xml version="1.0" encoding="utf-8"?>
<set 
    android:fillAfter = "true"
    android:interpolator = 
         "@android:anim/bounce_interpolator">

    <scale
        android:fromXScale = "1.0"
        android:fromYScale = "0.0"
        android:toXScale = "1.0"
        android:toYScale = "1.0" />

</set>
```

右键单击**anim**目录，然后选择**New | Animation resource file**。在**File name:**字段中，键入`rotate_left`，然后左键单击**OK**。删除内容并添加此代码以创建动画。在这里，我们看到了一些新东西，`pivotX="50%"`和`pivotY="50%"`。这使得旋转动画在将要被动画化的小部件上是中心的。我们可以将其视为设置动画的*中心*点：

```kt
<?xml version="1.0" encoding="utf-8"?>
<set >
    <rotate android:fromDegrees = "360"
        android:toDegrees = "0"
        android:pivotX = "50%"
        android:pivotY = "50%"
        android:interpolator = 
           "@android:anim/cycle_interpolator"/>
</set>
```

右键单击**anim**目录，然后选择**New | Animation resource file**。在**File name:**字段中，键入`rotate_right`，然后左键单击**OK**。删除内容并添加此代码以创建动画：

```kt
<?xml version="1.0" encoding="utf-8"?>
<set >
    <rotate android:fromDegrees = "0"
        android:toDegrees = "360"
        android:pivotX = "50%"
        android:pivotY = "50%"
        android:interpolator =
             "@android:anim/cycle_interpolator"/>

</set>
```

呼！现在我们可以编写 Kotlin 代码将我们的动画添加到我们的 UI 中。

## 在 Kotlin 中连接动画演示应用程序

打开`MainActivity.kt`文件。现在，在类声明之后，我们可以声明以下动画属性：

```kt
var seekSpeedProgress: Int = 0

private lateinit var animFadeIn: Animation
private lateinit var animFadeOut: Animation
private lateinit var animFadeInOut: Animation

private lateinit var animZoomIn: Animation
private lateinit var animZoomOut: Animation

private lateinit var animLeftRight: Animation
private lateinit var animRightLeft: Animation
private lateinit var animTopBottom: Animation

private lateinit var animBounce: Animation
private lateinit var animFlash: Animation 

private lateinit var animRotateLeft: Animation
private lateinit var animRotateRight: Animation
```

### 提示

此时，您需要添加以下`import`语句：

```kt
import android.view.animation.Animation;
```

在上述代码中，我们在声明`Animation`实例时使用了`lateinit`关键字。这意味着 Kotlin 将在使用每个实例之前检查它是否已初始化。这避免了我们在每次在这些实例中使用函数时使用`!!`（空检查）。有关`!!`运算符的复习，请参阅第十二章*将我们的 Kotlin 连接到 UI 和空值*。

我们还添加了一个`Int`属性`seekSpeedProgress`，它将用于跟踪`SeekBar`的当前值/位置。

现在，在`setContentView`调用之后，让我们从`onCreate`中调用一个新函数：

```kt
override fun onCreate(savedInstanceState: Bundle?) {
   super.onCreate(savedInstanceState)
   setContentView(R.layout.activity_main)

 loadAnimations()
}
```

在这一点上，新的代码行在实现新函数之前将出现错误。

现在我们将实现`loadAnimations`函数。虽然这个函数中的代码相当庞大，但也非常直接。我们所做的就是使用`AnimationUtils`类的`loadAnimation`函数，用我们的 XML 动画初始化每个`Animation`引用之一。您还会注意到，对于`animFadeIn` `Animation`，我们还在其上调用`setAnimationListener`。我们将很快编写监听事件的函数。

添加`loadAnimations`函数：

```kt
private fun loadAnimations() {

   animFadeIn = AnimationUtils.loadAnimation(
                this, R.anim.fade_in)
   animFadeIn.setAnimationListener(this)
   animFadeOut = AnimationUtils.loadAnimation(
                this, R.anim.fade_out)
   animFadeInOut = AnimationUtils.loadAnimation(
                this, R.anim.fade_in_out)

   animZoomIn = AnimationUtils.loadAnimation(
                this, R.anim.zoom_in)
   animZoomOut = AnimationUtils.loadAnimation(
                this, R.anim.zoom_out)

   animLeftRight = AnimationUtils.loadAnimation(
                 this, R.anim.left_right)
   animRightLeft = AnimationUtils.loadAnimation(
                 this, R.anim.right_left)
   animTopBottom = AnimationUtils.loadAnimation(
                 this, R.anim.top_bot)

   animBounce = AnimationUtils.loadAnimation(
                 this, R.anim.bounce)
   animFlash = AnimationUtils.loadAnimation(
                 this, R.anim.flash)

   animRotateLeft = AnimationUtils.loadAnimation(
                 this, R.anim.rotate_left)
   animRotateRight = AnimationUtils.loadAnimation(
                 this, R.anim.rotate_right)
}
```

### 提示

此时，您需要导入一个新的类：

```kt
import android.view.animation.AnimationUtils
```

现在，我们将为每个按钮添加一个点击监听器。在`onCreate`函数的右大括号之前立即添加以下代码：

```kt
btnFadeIn.setOnClickListener(this)
btnFadeOut.setOnClickListener(this)
btnFadeInOut.setOnClickListener(this)
btnZoomIn.setOnClickListener(this)
btnZoomOut.setOnClickListener(this)
btnLeftRight.setOnClickListener(this)
btnRightLeft.setOnClickListener(this)
btnTopBottom.setOnClickListener(this)
btnBounce.setOnClickListener(this)
btnFlash.setOnClickListener(this)
btnRotateLeft.setOnClickListener(this)
btnRotateRight.setOnClickListener(this)
```

### 注意

我们刚刚添加的代码在所有代码行中都创建了错误。我们现在可以忽略它们，因为我们很快就会修复它们并讨论发生了什么。

现在，我们可以使用 lambda 来处理`SeekBar`的交互。我们将重写三个函数，因为在实现`OnSeekBarChangeListener`时接口要求这样做：

+   一个检测`SeekBar`小部件位置变化的函数，称为`onProgressChanged`

+   一个检测用户开始改变位置的函数，称为`onStartTrackingTouch`

+   一个检测用户完成使用`SeekBar`小部件的函数，称为`onStopTrackingTouch`

为了实现我们的目标，我们只需要向`onProgressChanged`函数添加代码，但我们仍然必须重写它们全部。

在`onProgressChanged`函数中，我们所做的就是将`SeekBar`对象的当前值分配给`seekSpeedProgress`成员变量，以便可以从其他地方访问。然后，我们使用这个值以及`SeekBar`对象的最大可能值，通过使用`seekBarSpeed.max`，并向`textSeekerSpeed` `TextView`输出一条消息。

在`onCreate`函数的右大括号之前添加我们刚刚讨论过的代码：

```kt
seekBarSpeed.setOnSeekBarChangeListener(
         object : SeekBar.OnSeekBarChangeListener {

   override fun onProgressChanged(
                seekBar: SeekBar, value: Int, 
                fromUser: Boolean) {

         seekSpeedProgress = value
         textSeekerSpeed.text =
               "$seekSpeedProgress of $seekBarSpeed.max"
  }

  override fun onStartTrackingTouch(seekBar: SeekBar) {}

  override fun onStopTrackingTouch(seekBar: SeekBar) {}
})
```

现在，我们需要修改`MainActivity`类声明以实现两个接口。在这个应用程序中，我们将监听点击和动画事件，所以我们将使用的两个接口是`View.OnClickListener`和`Animation.AnimationListener`。您会注意到，要实现多个接口，我们只需用逗号分隔接口。

通过添加我们刚讨论过的突出显示的代码来修改`MainActivity`类声明：

```kt
class MainActivity : AppCompatActivity(),
        View.OnClickListener,
 Animation.AnimationListener {
```

在这个阶段，我们可以添加并实现这些接口所需的函数。首先是`AnimationListener`函数，`onAnimationEnd`，`onAnimationRepeat`和`onAnimationStart`。我们只需要在这些函数中的两个中添加一点代码。在`onAnimationEnd`中，我们将`textStatus`的`text`属性设置为`STOPPED`，在`onAnimationStart`中，我们将其设置为`RUNNING`。这将演示我们的动画监听器确实在监听和工作：

```kt
override fun onAnimationEnd(animation: Animation) {
   textStatus.text = "STOPPED"
}

override fun onAnimationRepeat(animation: Animation) {
}

override fun onAnimationStart(animation: Animation) {
   textStatus.text = "RUNNING"
}
```

`onClick`函数非常长，但并不复杂。`when`块的每个选项处理 UI 中的每个按钮，根据`SeekBar`小部件的当前位置设置动画的持续时间，设置动画以便监听事件，然后启动动画。

### 提示

您需要使用您喜欢的技术来导入`View`类：

```kt
import android.view.View;

```

添加我们刚讨论过的`onClick`函数，然后我们就完成了这个迷你应用程序：

```kt
override fun onClick(v: View) {
when (v.id) {
  R.id.btnFadeIn -> {
        animFadeIn.duration = seekSpeedProgress.toLong()
        animFadeIn.setAnimationListener(this)
        imageView.startAnimation(animFadeIn)
  }

  R.id.btnFadeOut -> {
        animFadeOut.duration = seekSpeedProgress.toLong()
        animFadeOut.setAnimationListener(this)
        imageView.startAnimation(animFadeOut)
  }

  R.id.btnFadeInOut -> {

        animFadeInOut.duration = seekSpeedProgress.toLong()
        animFadeInOut.setAnimationListener(this)
        imageView.startAnimation(animFadeInOut)
  }

  R.id.btnZoomIn -> {
        animZoomIn.duration = seekSpeedProgress.toLong()
        animZoomIn.setAnimationListener(this)
        imageView.startAnimation(animZoomIn)
  }

  R.id.btnZoomOut -> {
        animZoomOut.duration = seekSpeedProgress.toLong()
        animZoomOut.setAnimationListener(this)
        imageView.startAnimation(animZoomOut)
  }

  R.id.btnLeftRight -> {
        animLeftRight.duration = seekSpeedProgress.toLong()
        animLeftRight.setAnimationListener(this)
        imageView.startAnimation(animLeftRight)
  }

  R.id.btnRightLeft -> {
        animRightLeft.duration = seekSpeedProgress.toLong()
        animRightLeft.setAnimationListener(this)
        imageView.startAnimation(animRightLeft)
  }

  R.id.btnTopBottom -> {
        animTopBottom.duration = seekSpeedProgress.toLong()
        animTopBottom.setAnimationListener(this)
        imageView.startAnimation(animTopBottom)
  }

  R.id.btnBounce -> {
        /*
        Divide seekSpeedProgress by 10 because with
        the seekbar having a max value of 5000 it
        will make the animations range between
        almost instant and half a second
        5000 / 10 = 500 milliseconds
        */
        animBounce.duration = 
              (seekSpeedProgress / 10).toLong()
        animBounce.setAnimationListener(this)
        imageView.startAnimation(animBounce)
  }

  R.id.btnFlash -> {
        animFlash.duration = (seekSpeedProgress / 10).toLong()
        animFlash.setAnimationListener(this)
        imageView.startAnimation(animFlash)
  }

  R.id.btnRotateLeft -> {
        animRotateLeft.duration = seekSpeedProgress.toLong()
        animRotateLeft.setAnimationListener(this)
        imageView.startAnimation(animRotateLeft)
  }

  R.id.btnRotateRight -> {
        animRotateRight.duration = seekSpeedProgress.toLong()
        animRotateRight.setAnimationListener(this)
        imageView.startAnimation(animRotateRight)
  }
}

}
```

现在运行应用程序，并将`SeekBar`小部件移动到大致中心，以便动画运行一段合理的时间：

![在 Kotlin 中连接动画演示应用程序](img/B12806_19_04.jpg)

点击**放大**按钮：

![在 Kotlin 中连接动画演示应用程序](img/B12806_19_05.jpg)

注意 Android 机器人上的文本在适当的时间从**RUNNING**更改为**STOPPED**。现在，点击其中一个**ROTATE**按钮：

![在 Kotlin 中连接动画演示应用程序](img/B12806_19_06.jpg)

大多数其他动画在截图中无法展现出自己的价值，所以一定要自己尝试它们。

# 经常问的问题

Q.1) 我们现在知道如何为小部件添加动画，但是我自己创建的形状或图像呢？

A) 一个`ImageView`小部件可以容纳任何您喜欢的图像。只需将图像添加到`drawable`文件夹，然后在`ImageView`小部件上设置适当的`src`属性。然后您可以对`ImageView`小部件中显示的任何图像进行动画处理。

Q.2) 但是如果我想要比这更灵活的功能，更像是一个绘画应用程序甚至是一个游戏呢？

A) 要实现这种功能，我们需要学习另一个称为**线程**的通用计算概念，以及一些更多的 Android 类（如`Paint`，`Canvas`和`SurfaceView`）。我们将学习如何从单个像素到形状绘制任何东西，然后将它们移动到屏幕上，从下一章开始，第二十章 *绘制图形*。

# 总结

现在我们有另一个增强应用程序的技巧。在本章中，我们看到 Android 中的动画非常简单。我们在 XML 中设计了一个动画，并将文件添加到`anim`文件夹中。接下来，我们在 Kotlin 代码中使用`Animation`对象获取了 XML 中动画的引用。

然后，我们在 UI 中使用小部件的引用，使用`setAnimation`为其设置动画，并传入`Animation`对象。通过在小部件的引用上调用`startAnimation`来启动动画。

我们还看到我们可以控制动画的时间并监听动画事件。

在下一章中，我们将学习在 Android 中绘制图形。这将是关于图形的几章中的开始，我们将构建一个儿童风格的绘画应用程序。
