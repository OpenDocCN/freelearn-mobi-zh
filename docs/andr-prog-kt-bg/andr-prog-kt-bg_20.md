# 第二十章。绘图图形

整个章节将讨论 Android 的`Canvas`类以及一些相关类，如`Paint`，`Color`和`Bitmap`。当这些类结合在一起时，在屏幕上绘图时会带来巨大的力量。有时，Android API 提供的默认 UI 并不是我们所需要的。如果我们想要制作一个绘图应用程序，绘制图表，或者制作游戏，我们需要控制 Android 设备提供的每个像素。

在本章中，我们将涵盖以下主题：

+   了解`Canvas`类及一些相关类

+   编写一个基于`Canvas`的演示应用程序

+   查看 Android 坐标系统，以便知道在哪里进行绘制

+   学习绘制和操作位图图形

+   编写一个基于位图图形的演示应用程序

所以，让我们开始绘图吧！

# 了解 Canvas 类

`Canvas`类是`android.graphics`包的一部分。在接下来的两章中，我们将使用`android.graphics`包中的所有以下`import`语句以及来自现在熟悉的`View`包的另一个`import`语句。它们为我们提供了从 Android API 中获取一些强大绘图功能的途径：

```kt
import android.graphics.Bitmap
import android.graphics.Canvas
import android.graphics.Color
import android.graphics.Paint
import android.widget.ImageView

```

首先，让我们讨论前面代码中突出显示的`Bitmap`，`Canvas`和`ImageView`。

## 使用 Bitmap，Canvas 和 ImageView 开始绘制

由于 Android 设计用于运行各种类型的移动应用程序，我们不能立即开始输入我们的绘图代码并期望它能够工作。我们需要做一些准备（也就是更多的编码）来考虑我们的应用程序运行在特定设备上。这种准备有时可能有点反直觉，但我们将一步一步地进行。

### Canvas 和 Bitmap

根据您如何使用`Canvas`类，这个术语可能会有点误导。虽然`Canvas`类确实是您绘制图形的类，就像绘画画布一样，但您仍然需要一个**表面**来转置画布。

在这种情况下（以及我们的前两个演示应用程序中），表面将来自`Bitmap`类。

### 提示

请注意，位图是一种图像类型，Android 有一个`Bitmap`类。`Bitmap`类可用于将位图图像绘制到屏幕上，但正如我们将看到的那样，它还有其他用途。在谈论位图图像和`Bitmap`类时，我会尽量清晰明了，以便区分得更清楚。

我们可以将这个过程看作是：我们得到一个`Canvas`对象和一个`Bitmap`对象，然后将`Bitmap`对象设置为`Canvas`对象的一部分来进行绘制。

如果按照字面意义理解"画布"这个词有点反直觉，但一旦设置好了，我们就可以忘记它，专注于我们想要绘制的图形。

### 提示

`Canvas`类提供了绘制的*能力*。它具有绘制形状、文本、线条和图像文件（如其他位图）的所有功能，甚至支持绘制单个像素。

`Bitmap`类由`Canvas`类使用，是被绘制的表面。您可以将`Bitmap`实例视为位于`Canvas`实例上的图片框。

### Paint

除了`Canvas`和`Bitmap`，我们还将使用`Paint`类。这更容易理解；`Paint`是用于配置特定属性的类，例如我们将在`Canvas`实例中绘制的颜色（在`Canvas`实例内的`Bitmap`实例上）。

然而，在我们开始绘制之前，还有一个谜题需要解决。

### ImageView 和 Activity

`ImageView`类是`Activity`类用于向用户显示输出的类。引入这第三层抽象的原因是，正如我们在整本书中所看到的，`Activity`类需要将一个`View`引用传递给`setContentView`函数，以向用户显示内容。在整本书中，这一直是我们在可视化设计器或 XML 代码中创建的布局。

然而，这一次我们不需要用户界面 - 相反，我们需要绘制线条、像素、图像和形状。

有多个从 `View` 继承的类，可以制作所有不同类型的应用程序，并且它们都与 `Activity` 类兼容，这是所有常规 Android 应用程序（包括绘图应用程序和游戏）的基础。

因此，有必要将在 `Canvas` 上绘制的 `Bitmap` 类与 `ImageView` 类关联起来，一旦绘制完成。最后一步是通过将其传递给 `setContentView` 来告诉 `Activity` 类，我们的 `ImageView` 代表用户要看到的内容。

### Canvas、Bitmap、Paint 和 ImageView - 简要总结

如果我们需要设置的代码结构理论看起来并不简单，那么当你看到稍后的相对简单的代码时，你会松一口气。

到目前为止，我们已经覆盖了以下内容：

+   每个应用程序都需要一个 `Activity` 类来与用户和底层操作系统交互。因此，如果我们想成功，我们必须遵循所需的层次结构。

+   我们将使用继承自 `View` 类的 `ImageView` 类。`View` 类是 `Activity` 需要显示我们的应用程序给用户的东西。

+   `Canvas` 类提供了绘制线条、像素和其他图形的 *能力*。它具有执行诸如绘制形状、文本、线条和图像文件等操作的所有功能，甚至支持绘制单个像素。

+   `Bitmap` 类将与 `Canvas` 类关联，它是被绘制的表面。

+   `Canvas` 类使用 `Paint` 类来配置细节，比如绘制的颜色。

+   最后，一旦 `Bitmap` 实例被绘制，我们必须将其与 `ImageView` 类关联起来，而 `ImageView` 类又被设置为 `Activity` 实例的视图。

结果将是我们在 `Canvas` 实例中绘制的 `Bitmap` 实例将通过调用 `setContentView` 显示给用户的 `ImageView` 实例。呼～

### 提示

如果这并不是 100%清楚也没关系。不是你看不清楚 - 它只是没有清晰的关系。编写代码并反复使用这些技术将使事情变得更清晰。看看代码，执行本章和下一章的演示应用程序，然后重新阅读本节。

让我们看看如何在代码中建立这种关系 - 不要担心输入代码；我们先来学习它。

# 使用 Canvas 类

让我们看看代码和获取绘图所需的不同阶段，然后我们可以快速转移到使用 `Canvas` 演示应用程序真正绘制一些东西。

## 准备所需类的实例

第一步是将我们需要的类转换为可用的实例。

首先，我们声明我们需要的所有实例。我们不能立即初始化这些实例，但我们可以确保在使用它们之前初始化它们，所以我们在同样的方式中使用 `lateinit`，就像在动画演示应用程序中一样：

```kt
// Here are all the objects(instances)
// of classes that we need to do some drawing
lateinit var myImageView: ImageView
lateinit var myBlankBitmap: Bitmap
lateinit var myCanvas: Canvas
lateinit var myPaint: Paint
```

上一个代码声明了 `ImageView`、`Bitmap`、`Canvas` 和 `Paint` 类型的引用。它们分别被命名为 `myImageView`、`myBlankBitmap`、`myCanvas` 和 `myPaint`。

## 初始化对象

接下来，我们需要在使用它们之前初始化我们的新对象：

```kt
// Initialize all the objects ready for drawing
// We will do this inside the onCreate function
val widthInPixels = 800
val heightInPixels = 600

// Create a new Bitmap
myBlankBitmap = Bitmap.createBitmap(widthInPixels,
         heightInPixels,
         Bitmap.Config.ARGB_8888)

// Initialize the Canvas and associate it
// with the Bitmap to draw on
myCanvas = Canvas(myBlankBitmap)

// Initialize the ImageView and the Paint
myImageView = ImageView(this)
myPaint = Paint()
// Do drawing here
```

请注意上一个代码中的以下注释：

```kt
// Do drawing here
```

这是我们将配置颜色并绘制图形的地方。另外，请注意在代码顶部我们声明并初始化了两个 `Int` 变量，称为 `widthInPixels` 和 `heightInPixels`。当我们编写 `Canvas` 演示应用程序时，我将更详细地介绍其中一些代码行。

现在我们已经准备好绘制；我们所需要做的就是通过 `setContentView` 函数将 `ImageView` 实例分配给 `Activity`。

## 设置 Activity 内容

最后，在我们看到我们的绘图之前，我们告诉 Android 使用我们的名为`myImageView`的`ImageView`实例作为要显示给用户的内容：

```kt
// Associate the drawn upon Bitmap with the ImageView
myImageView.setImageBitmap(myBlankBitmap);
// Tell Android to set our drawing
// as the view for this app
// via the ImageView
setContentView(myImageView);
```

正如您在迄今为止的每个应用程序中已经看到的，`setContentView`函数是`Activity`类的一部分，这一次我们将`myImageView`作为参数传递，而不是像我们在整本书中一直做的那样传递 XML 布局。就是这样 - 现在我们要学习的就是如何在`Bitmap`实例上实际绘制。

在进行一些绘图之前，启动一个真正的项目将非常有用。我们将逐步复制并粘贴我们刚刚讨论过的代码到正确的位置，然后实际上在屏幕上看到一些绘制的东西。

所以，让我们开始绘图吧。

# Canvas Demo 应用程序

首先，创建一个新项目来探索使用`Canvas`进行绘图的主题。我们将重复利用我们所学到的知识，这一次我们还将绘制到`Bitmap`实例上。

## 创建一个新项目

创建一个新项目，将其命名为`Canvas Demo`，并确保选择**空活动**模板选项。

在这个应用程序中，我们将进行一个以前未见过的更改。我们将使用`Activity`类的原始版本。因此，`MainActivity`将继承自`Activity`，而不是之前一直使用的`AppCompatActivity`。我们这样做是因为我们不使用来自 XML 文件的布局，因此我们不需要`AppCompatActivity`的向后兼容功能，就像在以前的所有项目中一样。

您应该编辑类声明如下。

```kt
class MainActivity : Activity() {
```

您还需要添加以下导入语句：

```kt
import android.app.Activity
```

### 注意

此应用程序的完整代码可以在`Chapter20/Canvas Demo`文件夹的下载包中找到。

### 编写 Canvas 演示应用程序

接下来，删除`onCreate`函数的所有内容，除了声明/签名、调用 super.onCreate 以及打开和关闭大括号。

现在，我们可以在类声明之后但在`onCreate`函数之前添加以下突出显示的代码。在此步骤之后，代码将如下所示：

```kt
// Here are all the objects(instances)
// of classes that we need to do some drawing
lateinit var myImageView: ImageView
lateinit var myBlankBitmap: Bitmap
lateinit var myCanvas: Canvas
lateinit var myPaint: Paint

override fun onCreate(savedInstanceState: Bundle?) {
   super.onCreate(savedInstanceState)
}
```

在 Android Studio 中，四个新类都被下划线标记为红色。这是因为我们需要添加适当的`import`语句。您可以从本章的第一页复制它们，但更快的方法是依次将鼠标光标放在每个错误上，然后按住*ALT*键并轻按*Enter*键。如果从弹出选项中提示，请选择**导入类**。

完成对`ImageView`、`Bitmap`、`Canvas`和`Paint`的操作后，所有错误都将消失，并且相关的`import`语句将被添加到代码的顶部。

现在我们已经声明了所需类的实例，我们可以对它们进行初始化。将以下代码添加到`onCreate`函数中，添加到“super.onCreate…”之后，如下所示：

```kt
override fun onCreate(savedInstanceState: Bundle?) {
   super.onCreate(savedInstanceState)

   // Initialize all the objects ready for drawing
   // We will do this inside the onCreate function
   val widthInPixels = 800
   val heightInPixels = 600

   // Create a new Bitmap
   myBlankBitmap = Bitmap.createBitmap(widthInPixels,
                heightInPixels,
                Bitmap.Config.ARGB_8888)

   // Initialize the Canvas and associate it
   // with the Bitmap to draw on
   myCanvas = Canvas(myBlankBitmap)

   // Initialize the ImageView and the Paint
   myImageView = ImageView(this)
   myPaint = Paint()
}
```

前面的代码与我们在理论上讨论`Canvas`时看到的代码相同。但是，值得更深入地探索`Bitmap`类的初始化，因为它并不简单。

#### 探索位图初始化

位图在基于图形的应用程序和游戏中更常见，用于表示不同的画笔、玩家角色、背景、游戏对象等对象。在这里，我们只是用它来绘制。在下一个项目中，我们将使用位图来表示我们绘制的主题，而不仅仅是绘制的表面。

需要解释的函数是`createBitmap`函数。从左到右的参数如下：

+   宽度（以像素为单位）

+   高度（以像素为单位）

+   位图配置

`Bitmap`实例可以以几种不同的方式进行配置；`ARGB_8888`配置意味着每个像素由四个字节的内存表示。

### 注意

Android 可以使用多种位图格式。这种格式非常适合绘制一系列颜色，并确保我们使用的位图和请求的颜色将按预期绘制。还有更高和更低的配置，但`ARGB_8888`非常适合本书。

现在，我们可以进行实际绘制。

### 在屏幕上绘制

在`myPaint`初始化之后和`onCreate`函数的闭合大括号内添加以下突出显示的代码：

```kt
// Draw on the Bitmap
// Wipe the Bitmap with a blue color
myCanvas.drawColor(Color.argb(255, 0, 0, 255))

// Re-size the text
myPaint.textSize = 100f
// Change the paint to white
myPaint.color = Color.argb(255, 255, 255, 255)
// Draw some text
myCanvas.drawText("Hello World!",100f, 100f, myPaint)

// Change the paint to yellow
myPaint.color = Color.argb(255, 212, 207, 62)
// Draw a circle
myCanvas.drawCircle(400f, 250f, 100f, myPaint)
```

前面的代码使用：

+   `myCanvas.drawColor`用颜色填充屏幕

+   `myPaint.textSize`属性定义了接下来将绘制的文本的大小

+   `myPaint.color`属性决定了未来任何绘图的颜色

+   `myCanvas.drawText`函数实际上将文本绘制到屏幕上。

如果我们分析传递给`drawText`的参数，我们可以看到文本将会显示"Hello World!"，并且将在我们的`Bitmap`实例(`myBitmap`)的左侧 100 像素和顶部 100 像素处绘制。

接下来，我们再次使用`color`属性来更改将用于绘制的颜色。最后，我们使用`drawCircle`函数来绘制一个距左侧 400 像素，顶部 100 像素的圆。圆的半径为 100 像素。

直到现在，我一直没有解释`Color.argb`函数。

#### 解释 Color.argb

`Color`类，不出所料，帮助我们操纵和表示颜色。`argb`函数返回使用**a**lpha（不透明度和透明度）、**r**ed、**g**reen、**b**lue 模型构建的颜色。该模型对于每个元素使用从 0（无颜色）到 255（全颜色）的值。重要的是要注意 - 尽管这似乎是显而易见的 - 混合颜色是不同颜色光的强度，结果与我们混合颜料时发生的情况完全不同。

### 提示

要设计 ARGB 值并进一步探索这个模型，请查看这个方便的网站：[`www.rapidtables.com/web/color/RGB_Color.html`](https://www.rapidtables.com/web/color/RGB_Color.html)。该网站可以帮助您选择 RGB 值；然后您可以尝试 alpha 值。

用于清除绘图表面的值是`255`、`0`、`0`和`255`。这些值表示完全不透明（即纯色），没有红色，没有绿色，完全蓝色。这会产生蓝色。

对`argb`函数的下一个调用是在`setColor`的第一个调用中，我们正在为文本设置所需的颜色。`255`、`255`、`255`和`255`的值表示完全不透明，完全红色，完全绿色和完全蓝色。当您将光与这些值结合时，您将得到白色。

对`argb`的最终调用是在`setColor`的最终调用中，我们正在设置绘制圆的颜色；`255`、`21`、`207`和`62`产生太阳黄色。

在运行代码之前，我们需要执行的最后一步是添加对`setContentView`函数的调用，将我们的`ImageView`实例(`myImageView`)放置为此应用程序的内容视图。以下是我们已经添加的代码之后，但在`onCreate`的闭合大括号之前的最后几行代码：

```kt
// Associate the drawn upon Bitmap with the ImageView
myImageView.setImageBitmap(myBlankBitmap);
// Tell Android to set our drawing
// as the view for this app
// via the ImageView
setContentView(myImageView);
```

最后，我们通过调用`setContentView`告诉`Activity`类使用`myImageView`。

下面的屏幕截图展示了当您运行 Canvas 演示应用程序时的外观。我们可以看到一个 800x800 像素的绘图。在下一章中，我们将使用更高级的技术来利用整个屏幕，并且我们还将学习有关线程，以使图形实时移动：

![解释 Color.argb](img/B12806_C20_01.jpg)

如果您了解 Android 坐标系统的更多信息，将有助于您理解我们在`Canvas`绘图函数中使用的坐标的结果。

# Android 坐标系统

正如您将看到的，绘制位图图形是微不足道的。但是，我们用来绘制图形的坐标系统需要简要解释。

## 绘图和绘制

当我们在屏幕上绘制位图图形时，我们传入要绘制对象的坐标。给定 Android 设备的可用坐标取决于其屏幕的分辨率。

例如，Google Pixel 手机在横向方向上的屏幕分辨率为 1,920 像素（横向）x 1,080 像素（纵向）。

这些坐标的编号系统从左上角的 0,0 开始，向下和向右移动，直到右下角是像素 1919, 1079。1,920/1,919 和 1,080/1,079 之间明显的 1 像素差异是因为编号从 0 开始。

因此，当我们在屏幕上绘制位图图形或其他任何东西（如`Canvas`圆和矩形）时，我们必须指定*x*，*y*坐标。

此外，位图图形（或`Canvas`形状）当然包括许多像素。因此，我们将要指定的*x*，*y*屏幕坐标上绘制给定位图图形的哪个像素？ 

答案是位图图形的左上角像素。看一下下一个图表，它应该使用 Google Pixel 手机作为示例来澄清屏幕坐标。作为解释 Android 坐标绘制系统的图形手段，我将使用一个可爱的太空飞船图形：

![绘图和绘制](img/B12806_C20_02.jpg)

此外，这些坐标是相对于您绘制的内容。因此，在我们刚刚编写的`Canvas`演示和下一个演示中，坐标是相对于`Bitmap`对象（`myBitmap`）的。在下一章中，我们将使用整个屏幕，上一个图表将更准确地表示发生的情况。

让我们做一些更多的绘图 - 这次使用位图图形（再次使用`Bitmap`类）。我们将使用与此应用程序中看到的相同的起始代码。

# 使用 Bitmap 类创建位图图形

在我们深入代码之前，让我们先研究一些理论，并考虑我们将如何将图像绘制到屏幕上。要绘制位图图形，我们将使用`Canvas`类的`drawBitmap`函数。

首先，我们需要在`res/drawable`文件夹中的项目中添加一个位图图形 - 我们将在 Bitmap 演示应用程序中进行这个操作。现在，假设图形文件/位图的名称为`myImage.png`。

接下来，我们将以与我们在上一个演示中用于背景的`Bitmap`对象相同的方式声明`Bitmap`类型的对象。

接下来，我们需要使用我们之前添加到项目的`drawable`文件夹中的首选图像文件来初始化`myBitmap`实例：

```kt
myBitmap = BitmapFactory.decodeResource
                (resources, R.drawable.myImage)
```

`BitmapFactory`类的`decodeResource`函数用于初始化`myBitmap`。它需要两个参数；第一个是`Activity`类提供的`resources`属性。这个函数，正如其名称所示，可以访问项目资源，第二个参数`R.drawable.myImage`指向`drawable`文件夹中的`myImage.png`文件。`Bitmap`（`myBitmap`）实例现在已准备好由`Canvas`类绘制。

现在，您可以使用以下代码通过`Bitmap`实例绘制位图图形：

```kt
// Draw the bitmap at coordinates 100, 100
canvas.drawBitmap(myBitmap, 
                100, 100, myPaint);
```

当在屏幕上绘制时，上一节中太空飞船图形的样子如下（仅供参考，当我们谈论旋转位图时）：

![使用 Bitmap 类创建位图图形](img/B12806_C20_03.jpg)

# 操作位图

然而，通常情况下，我们需要以旋转或其他方式改变的状态绘制位图。使用 Photoshop 或您喜欢的其他图像编辑软件创建更多的位图以面向其他方向是非常容易的。然后，当我们要绘制位图时，我们可以简单地决定以哪种方式绘制适当的预加载位图。

然而，如果我们只使用一个单一的源图像并学习 Android 提供的用于在 Kotlin 代码中操作图像的类，那将会更有趣和有教育意义。然后，你就可以将旋转和反转图形添加到你的应用程序开发工具包中。

## 什么是位图？

位图之所以被称为位图，是因为它确实就是一个“位的地图”。虽然有许多使用不同范围和值来表示颜色和透明度的位图格式，但它们都归结为同一件事。它们是一组值的网格或地图，每个值代表一个像素的颜色。

因此，要旋转、缩放或反转位图，我们必须对位图的每个像素或位进行适当的数学计算。这些计算并不是非常复杂，但也不是特别简单。如果你上完高中的数学课，你可能不会对这些数学感到太困难。

不幸的是，理解数学还不够。我们还需要设计高效的代码，了解位图格式，然后针对每种格式修改我们的代码；这并不是微不足道的。幸运的是（正如我们所期望的那样），Android API 已经为我们做好了一切 - 认识`Matrix`类。

## Matrix 类

这个类被命名为`Matrix`，是因为它使用数学概念和规则来对一系列值进行计算，这些值被称为矩阵 - 矩阵的复数。

### 提示

Android 的`Matrix`类与同名电影系列无关。然而，作者建议所有有抱负的应用程序开发者服用**红色**药丸。

你可能对矩阵很熟悉，但如果你不熟悉也不用担心，因为`Matrix`类将所有复杂性都隐藏起来了。此外，`Matrix`类不仅允许我们对一系列值进行计算，还具有一些预先准备好的计算，使我们能够做一些事情，比如围绕另一个点旋转一个点特定角度，而无需了解三角学。

### 提示

如果你对`Matrix`类背后的数学运作感兴趣，并且想要一个绝对初学者指南来学习旋转游戏对象的数学，那么请查看我网站上的这一系列 Android 教程，其中包括一个可飞行和可旋转的太空飞船。这些教程是用 Java 编写的，但应该很容易理解：

[`gamecodeschool.com/essentials/calculating-heading-in-2d-games-using-trigonometric-functions-part-1/`](http://gamecodeschool.com/essentials/calculating-heading-in-2d-games-using-trigonometric-functions-part-1/)

[`gamecodeschool.com/essentials/rotating-graphics-in-2d-games-using-trigonometric-functions-part-2/`](http://gamecodeschool.com/essentials/rotating-graphics-in-2d-games-using-trigonometric-functions-part-2/)

[`gamecodeschool.com/android/2d-rotation-and-heading-demo/`](http://gamecodeschool.com/android/2d-rotation-and-heading-demo/)

这本书将继续使用 Android 的`Matrix`类，但在下一章中创建粒子系统时，我们将进行稍微更高级的数学运算。

### 将位图反转以面对相反方向

首先，我们需要创建一个`Matrix`类的实例。下面的代码行以熟悉的方式调用默认构造函数来实现这一点：

```kt
val matrix = Matrix()
```

### 提示

请注意，你现在不需要将任何这些代码添加到项目中；它很快就会再次显示，并且会有更多的上下文。我只是觉得在此之前单独看到所有与`Matrix`相关的代码会更容易些。

现在我们可以使用`Matrix`类的许多巧妙功能之一。`preScale`函数接受两个参数；一个用于水平变化，一个用于垂直变化。看一下下面的代码行：

```kt
matrix.preScale(-1, 1)
```

`preScale`函数将循环遍历每个像素位置，并将所有水平坐标乘以`-1`，所有垂直坐标乘以`1`。

这些计算的效果是所有垂直坐标将保持不变，因为如果乘以一，那么数字不会改变。但是，当您乘以负一时，像素的水平位置将被倒转。例如，水平位置 0、1、2、3 和 4 将变为 0、-1、-2、-3 和-4。

在这个阶段，我们已经创建了一个可以在位图上执行必要计算的矩阵。我们实际上还没有对位图做任何事情。要使用`Matrix`实例，我们调用`Bitmap`类的`createBitmap`函数，如下面的代码行：

```kt
myBitmapLeft = Bitmap
    .createBitmap(myBitmapRight,
          0, 0, 50, 25, matrix, true)
```

上面的代码假设`myBitmapLeft`已经与`myBitmapRight`一起初始化。`createBitmap`函数的参数解释如下：

+   `myBitmapRight`是一个已经创建并缩放的`Bitmap`对象，并且已经加载了图像（面向右侧）。这是将用作创建新`Bitmap`实例的源的图像。源`Bitmap`对象将不会被改变。

+   `0, 0`是我们希望将新的`Bitmap`实例映射到的水平和垂直起始位置。

+   `50, 25`参数是设置位图缩放到的大小。

+   下一个参数是我们预先准备的`Matrix`实例`matrix`。

+   最后一个参数`true`指示`createBitmap`函数需要过滤以正确处理`Bitmap`类型的创建。

这就是在绘制到屏幕时`myBitmapLeft`的样子：

![将位图反转以面向相反方向](img/B12806_C20_04.jpg)

我们还可以使用旋转矩阵创建面向上和下的位图。

### 将位图旋转以面向上和下

让我们看看如何旋转`Bitmap`实例，然后我们可以构建演示应用程序。我们已经有了`Matrix`类的一个实例，所以我们只需要调用`preRotate`函数来创建一个能够将每个像素旋转指定角度的矩阵，该角度作为`preRotate`的单个参数。看看下面的代码行：

```kt
// A matrix for rotating
matrix.preRotate(-90)
```

是不是很简单？`matrix`实例现在已经准备好以逆时针（`-`）`90`度旋转我们传递给它的任何一系列数字（位图）。

以下代码行与我们分解的先前对`createBitmap`的调用具有相同的参数，只是新的`Bitmap`实例分配给了`myBitmapUp`，并且`matrix`的效果是执行旋转而不是`preScale`函数：

```kt
mBitmapUp = Bitmap
   .createBitmap(mBitmap,
         0, 0, 25, 50, matrix, true)
```

这就是在绘制时`myBitmapUp`的样子：

![将位图旋转以面向上和下](img/B12806_C20_05.jpg)

您还可以使用相同的技术，但在`preRotate`的参数中使用不同的值，以使位图面向下。让我们继续演示应用程序，看看所有这些东西是如何运作的。

# Bitmap 操作演示应用程序

现在我们已经学习了理论，让我们绘制和旋转一些位图。首先，创建一个新项目并将其命名为`Bitmap manipulation`。选择**空活动**选项，其他设置与整本书中的设置相同。

## 将 Bob 图形添加到项目中

右键单击并选择**复制**，从`Chapter20/Bitmap Manipulation/drawable`文件夹中的下载包中复制`bob.png`图形文件。由`bob.png`表示的 Bob 是一个简单的静态视频游戏角色。

在 Android Studio 中，定位项目资源管理器窗口中的`app/res/drawable`文件夹，并将`bob.png`图像文件粘贴到其中。以下屏幕截图清楚地显示了该文件夹的位置以及带有`bob.png`图像的外观：

![将 Bob 图形添加到项目中](img/B12806_C20_06.jpg)

右键单击`drawable`文件夹，然后选择**粘贴**以将`bob.png`文件添加到项目中。点击两次**确定**以确认将文件导入项目的默认选项。

在这个应用程序中，我们将做与上一个应用程序相同的更改。我们将使用`Activity`类的原始版本。因此，`MainActivity`将继承自`Activity`而不是`AppCompatActivity`，这是以前的情况。我们这样做是因为，再次强调，我们不使用来自 XML 文件的布局，因此我们不需要`AppCompatActivity`的向后兼容功能，就像在以前的所有项目中一样。

您应该编辑类声明如下。

```kt
class MainActivity : Activity() {
```

您还需要添加以下导入语句：

```kt
import android.app.Activity
```

在`MainActivity`类的类声明之后，在`onCreate`函数之前，添加以下必需的属性，准备进行一些绘图：

```kt
// Here are all the objects(instances)
// of classes that we need to do some drawing
lateinit var myImageView: ImageView
lateinit var myBlankBitmap: Bitmap
lateinit var bobBitmap: Bitmap
lateinit var myCanvas: Canvas
lateinit var myPaint: Paint
```

### 提示

在包声明之后添加以下导入：

```kt
import android.graphics.Bitmap
import android.graphics.BitmapFactory
import android.graphics.Canvas
import android.graphics.Color
import android.graphics.Matrix
import android.graphics.Paint
import android.widget.ImageView
```

现在，我们可以在`onCreate`中初始化所有实例，如下所示：

```kt
// Initialize all the objects ready for drawing
val widthInPixels = 2000
val heightInPixels = 1000

// Create a new Bitmap
myBlankBitmap = Bitmap.createBitmap(widthInPixels,
         heightInPixels,
         Bitmap.Config.ARGB_8888)

// Initialize Bob
bobBitmap = BitmapFactory.decodeResource(
          resources, R.drawable.bob)

// Initialize the Canvas and associate it
// with the Bitmap to draw on
myCanvas = Canvas(myBlankBitmap)

// Initialize the ImageView and the Paint
myImageView = ImageView(this)
myPaint = Paint()

// Draw on the Bitmap
// Wipe the Bitmap with a blue color
myCanvas.drawColor(Color.argb(
         255, 0, 0, 255))
```

接下来，我们添加对三个函数的调用，我们很快将编写这些函数，并将我们的新绘图设置为应用程序的视图：

```kt
// Draw some bitmaps
drawRotatedBitmaps()
drawEnlargedBitmap()
drawShrunkenBitmap()

// Associate the drawn upon Bitmap
// with the ImageView
myImageView.setImageBitmap(myBlankBitmap)
// Tell Android to set our drawing
// as the view for this app
// via the ImageView
setContentView(myImageView)
```

现在，添加`drawRotatedBitmap`函数，执行位图操作：

```kt
fun drawRotatedBitmaps() {
   var rotation = 0f
   var horizontalPosition = 350
   var verticalPosition = 25
   val matrix = Matrix()

   var rotatedBitmap: Bitmap

   rotation = 0f
   while (rotation < 360) {
         matrix.reset()
         matrix.preRotate(rotation)
         rotatedBitmap = Bitmap
                      .createBitmap(bobBitmap, 
                      0, 0, bobBitmap.width - 1,
                      bobBitmap.height - 1,
                      matrix, true)

        myCanvas.drawBitmap(
                    rotatedBitmap,
                    horizontalPosition.toFloat(),
                    verticalPosition.toFloat(),
                    myPaint)

        horizontalPosition += 120
        verticalPosition += 70
        rotation += 30f
  }
}
```

先前的代码使用循环迭代 360 度，每次 30 度。值（在循环中的每次通过）用于在`Matrix`实例中旋转 Bob 的图像，然后使用`drawBitmap`函数将其绘制到屏幕上。

添加最后两个函数，如下所示：

```kt
fun drawEnlargedBitmap() {
  bobBitmap = Bitmap
               .createScaledBitmap(bobBitmap,
                           300, 400, false)
  myCanvas.drawBitmap(bobBitmap, 25f, 25f, myPaint)

}

fun drawShrunkenBitmap() {
  bobBitmap = Bitmap
              .createScaledBitmap(bobBitmap,
                          50, 75, false)
  myCanvas.drawBitmap(bobBitmap, 250f, 25f, myPaint)
}
```

`drawEnlargedBitmap`函数使用`createScaledBitmap`函数，将位图图形放大到 300 x 400 像素。然后`drawBitmap`函数将其绘制到屏幕上。

`drawShrunkenBitmap`函数使用完全相同的技术，只是它缩放然后绘制一个 50 x 75 像素的图像。

最后，运行应用程序，看到 Bob 在 30 度间隔下生长、缩小，然后围绕 360 度旋转，如下截图所示：

![将 Bob 图形添加到项目中](img/B12806_C20_07.jpg)

我们绘图库中唯一缺少的是观看所有这些活动发生的能力。我们将在下一步中填补这一知识空白。

# 常见问题

Q 1）我知道如何进行所有这些绘图，但为什么我看不到任何东西移动？

A）要看到物体移动，您需要能够调节绘图的每个部分发生的时间。您需要使用动画技术。这并不是微不足道的，但对于一个有决心的初学者来说也不是难以掌握的。我们将在下一章中学习所需的主题。

# 摘要

在本章中，我们学习了如何绘制自定义形状、文本和位图。现在我们知道如何绘制和操作原始形状、文本和位图，我们可以提升一级。

在下一章中，我们将开始我们的下一个多章节应用程序，这是一个儿童风格的绘图应用程序，只需轻按按钮即可生动起来。
