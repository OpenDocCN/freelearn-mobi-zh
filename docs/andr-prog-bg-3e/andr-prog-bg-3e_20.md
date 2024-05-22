# 第二十章：*第二十章*：绘制图形

整个章节将涉及 Android 的`Canvas`类和一些相关类，包括`Paint`、`Color`和`Bitmap`。这些类结合起来在绘制屏幕时具有强大的功能。有时，Android API 提供的默认 UI 并不是我们需要的。如果我们想要制作一个绘图应用、绘制图表，或者创建一个游戏，我们需要控制 Android 设备提供的每个像素。

在这一章中，我们将涵盖以下内容：

+   理解`Canvas`和相关类

+   编写基于`Canvas`的演示应用

+   查看 Android 坐标系统，以便知道在哪里进行绘图

+   学习绘制和操作位图

+   编写基于位图的演示应用

让我们开始绘图！

# 技术要求

您可以在 GitHub 上找到本章的代码文件，网址为[`github.com/PacktPublishing/Android-Programming-for-Beginners-Third-Edition/tree/main/chapter%2020`](https://github.com/PacktPublishing/Android-Programming-for-Beginners-Third-Edition/tree/main/chapter%2020)。

# 理解 Canvas 类

`Canvas`类是`android.graphics`包的一部分。在接下来的两章中，我们将使用`android.graphics`包中的所有以下`import`语句，以及来自现在熟悉的`View`包的一个额外的`import`语句。它们为我们提供了从 Android API 中获取一些强大绘图方法的权限：

```kt
import android.graphics.Bitmap;
import android.graphics.Canvas;
import android.graphics.Color;
import android.graphics.Paint;
import android.widget.ImageView;
```

首先，让我们讨论`Bitmap`、`Canvas`和`ImageView`，正如前面的代码所强调的那样。

## 使用 Bitmap、Canvas 和 ImageView 开始绘图

由于 Android 设计用于运行各种类型的移动应用程序，我们不能立即开始输入绘图代码并期望它能够工作。我们需要做一些准备（编码）来考虑我们的应用正在运行的特定设备。有些准备可能有点反直觉，但我们将一步一步地进行。

### Canvas 和 Bitmap

根据您如何使用`Canvas`类，这个术语可能会有点误导。虽然`Canvas`类*是*您绘制图形的类，就像绘画画布一样，但您仍然需要一个表面来转置画布。

在这种情况下（以及我们的前两个演示应用中），表面将来自`Bitmap`类。我们可以这样想：我们得到一个`Canvas`对象和一个`Bitmap`对象，然后将`Bitmap`对象设置为`Canvas`对象的一部分来进行绘制。

如果您按照字面意义上的画布这个词，这可能有点反直觉，但一旦设置好了，我们就可以忘记它，专注于我们想要绘制的图形。

注意

`Canvas`类提供了绘制的*能力*。它具有绘制形状、文本、线条和图像文件（包括其他位图），甚至绘制单个像素的所有方法。

`Bitmap`类被`Canvas`类使用，并且是被绘制的表面。你可以把`Bitmap`实例想象成在`Canvas`实例上的图片框内。

### Paint

除了`Canvas`和`Bitmap`类之外，我们还将使用`Paint`类。这更容易理解。`Paint`是用来配置特定属性的类，比如我们将在`Bitmap`（在`Canvas`内）上绘制的颜色。

在我们开始绘图之前，还有一个谜题需要解决。

### ImageView 和 Activity

`ImageView`类是`Activity`类用于向用户显示输出的类。有第三层抽象的原因是，正如我们在整本书中所看到的，`Activity`类需要将一个`View`传递给`setContentView`方法，以向用户显示内容。到目前为止，这一直是我们在可视化设计器或 XML 代码中创建的布局。

这一次，我们不需要常规的 UI；我们需要绘制线条、像素和形状。

有多种类型的类扩展了`View`类，使得可以制作许多不同类型的应用程序，并且它们都与`Activity`类兼容，这是所有常规 Android 应用程序（包括绘图应用程序和游戏）的基础。

因此，一旦绘制完成，就需要将`Bitmap`类（通过与`Canvas`的关联）与`ImageView`关联起来。最后一步是通过将其传递给`setContentView`方法，告诉`Activity`我们的`ImageView`类代表用户要看到的内容。

### Canvas、Bitmap、Paint 和 ImageView 的简要总结

如果我们需要设置的代码结构的理论看起来并不简单，当你看到相对简单的代码时，你会松一口气。

迄今为止我们所知道的内容的一个简要总结：

+   每个应用程序都需要一个`Activity`类来与用户和底层操作系统交互。因此，如果我们想成功，我们必须遵循所需的层次结构。

+   我们将使用`ImageView`类，它是`View`类的一种类型。`View`类是`Activity`需要显示我们的应用程序给用户的内容。

+   `Canvas`类还提供了绘制线条、像素和其他图形的*能力*。它具有绘制形状、文本、线条和图像文件，甚至绘制单个像素的所有方法。

+   `Bitmap`类将与`Canvas`类关联，它是实际绘制的表面。

+   `Canvas`类使用`Paint`类来配置颜色等细节。

最后，一旦位图被绘制，我们必须将其与`ImageView`实例关联起来，然后通过`setContentView`方法将其设置为`Activity`的视图。

结果将是我们在`Canvas`实例中绘制的`Bitmap`实例，通过调用`setContentView`方法显示给用户的`ImageView`实例。哦！

注意

如果这并不是 100%清楚也没关系。并不是你没有清晰地看到事情 - 它只是没有一个清晰的关系。反复编写代码并使用这些技术将使事情变得更清晰。看看代码，做一下本章和下一章的演示应用程序，然后重新阅读本节。

让我们看看如何在代码中建立这种关系。不要担心输入代码，先学习它。

# 使用 Canvas 类

让我们看看代码和获取绘图所需的不同阶段，然后我们可以快速转移到使用`Canvas`演示应用程序真正绘制一些东西。

## 准备所需类的实例

第一步是声明我们需要的类的实例：

```kt
// Here are all the objects(instances)
// of classes that we need to do some drawing
ImageView myImageView;
Bitmap myBlankBitmap;
Canvas myCanvas;
Paint myPaint;
```

上面的代码声明了`ImageView`、`Bitmap`、`Canvas`和`Paint`类型的引用。它们分别被命名为`myImageView`、`myBlankBitmap`、`myCanvas`和`myPaint`。

## 初始化对象

接下来，我们需要在使用之前初始化我们的新对象：

```kt
// Initialize all the objects ready for drawing
// We will do this inside the onCreate method
int widthInPixels = 800;
int heightInPixels = 800;
myBlankBitmap = Bitmap.createBitmap(widthInPixels,
         heightInPixels,
         Bitmap.Config.ARGB_8888);
myCanvas = new Canvas(myBlankBitmap);
myImageView = new ImageView(this);
myPaint = new Paint();
// Do drawing here
```

注意上面代码中的这条注释：

```kt
// Do drawing here
```

这是我们配置颜色和绘制内容的地方。还要注意代码顶部我们声明和初始化了两个名为`widthInPixels`和`heightInPixels`的`int`变量。当我们编写`Canvas`演示应用程序时，我将更详细地介绍其中的一些代码行。

我们现在准备好绘制了。我们所需要做的就是将`ImageView`实例分配给`Activity`。

## 设置 Activity 内容

最后，在我们看到我们的绘图之前，我们告诉 Android 使用我们称为`myImageView`的`ImageView`实例作为要显示给用户的内容：

```kt
// Associate the drawn upon Bitmap with the ImageView
myImageView.setImageBitmap(myBlankBitmap);
// Tell Android to set our drawing
// as the view for this app
// via the ImageView
setContentView(myImageView);
```

正如我们迄今为止在每个应用程序中看到的那样，`setContentView`方法是`Activity`类的一部分，我们传入`myImageView`作为参数，而不是像我们在整本书中一直做的那样传入 XML 布局。就是这样。我们现在所需要学习的就是如何在`Bitmap`实例上实际绘制。

在我们进行一些绘图之前，我认为开始一个真正的项目，逐步复制和粘贴我们刚刚讨论过的代码到正确的位置，然后实际看到一些东西被绘制到屏幕上会很有用。

让我们进行一些绘制。 

# 画布演示应用程序

我们将创建一个新项目，只是为了探索使用`Canvas`进行绘制的主题。我们将重用我们刚刚学到的知识，这次还将绘制到`Bitmap`实例上。

## 创建一个新项目

创建一个新项目，命名为`Canvas Demo`。选择**空活动**模板。

此外，我们将使用`Activity`类的原始版本，因此`MainActivity`类将扩展`Activity`，而不是之前使用的`AppCompatActivity`。这仅仅是因为我们不再需要`AppCompatActivity`类提供的额外功能。

注意

此应用程序的完整代码可以在*第二十章*`/Canvas Demo`文件夹中的下载包中找到。

### 编写 Canvas 演示应用程序

要开始，请编辑`MainActivity.java`中的代码，包括添加`import`指令和更改`MainActivity`类继承的`Activity`类的版本。还要注意下一个代码中`setContentView`方法的调用也已被删除。我们很快会替换它：

```kt
import android.app.Activity;
import android.graphics.Bitmap;
import android.graphics.Canvas;
import android.graphics.Color;
import android.graphics.Paint;
import android.os.Bundle;
import android.widget.ImageView;
public class MainActivity extends Activity {
    // Here are all the objects(instances)
    // of classes that we need to do some drawing
    ImageView myImageView;
    Bitmap myBlankBitmap;
    Canvas myCanvas;
    Paint myPaint;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
    }
}
```

现在我们已经声明了所需类的实例，我们可以初始化它们。在`onCreate`方法中调用`super.onCreate…`之后，添加以下代码，如下一个代码所示：

```kt
@Override
protected void onCreate(Bundle savedInstanceState) {
   super.onCreate(savedInstanceState);
   // Initialize all the objects ready for drawing
   // We will do this inside the onCreate method
   int widthInPixels = 800;
   int heightInPixels = 600;
   // Create a new Bitmap
   myBlankBitmap = Bitmap.createBitmap(widthInPixels,
                heightInPixels,
                Bitmap.Config.ARGB_8888);
   // Initialize the Canvas and associate it
   // with the Bitmap to draw on
   myCanvas = new Canvas(myBlankBitmap);
   // Initialize the ImageView and the Paint
   myImageView = new ImageView(this);
   myPaint = new Paint();
}
```

这段代码与我们之前讨论`Canvas`时看到的一样。值得探索`Bitmap`类的初始化，因为它并不简单。

#### 探索位图初始化

位图，在基于图形的应用程序和游戏中更典型地用于表示不同的画笔、玩家、背景、游戏对象等。在这里，我们只是用它来绘制。在下一个项目中，我们将使用位图来表示我们绘制的主题，而不仅仅是绘制的表面。

需要解释的方法是`createBitmap`方法。从左到右的参数如下：

+   宽度（以像素为单位）

+   高度（以像素为单位）

+   位图配置

位图可以以几种不同的方式配置。`ARGB_8888`配置意味着每个像素由 4 个字节的内存表示。

Android 可以使用几种位图格式。这个对于一系列的颜色来说是完美的，并且将确保我们使用的位图和我们请求的颜色将按预期绘制。有更高和更低的配置，但`ARGB_8888`对于本章来说是完美的。

现在我们可以进行实际的绘制了。

### 在屏幕上绘制

在`myPaint`初始化之后，在`onCreate`方法的闭合大括号内添加下面突出显示的代码：

```kt
   myPaint = new Paint();
   // Draw on the Bitmap
   // Wipe the Bitmap with a blue color
   myCanvas.drawColor(Color.argb(255, 0, 0, 255));
   // Re-size the text
   myPaint.setTextSize(100);
   // Change the paint to white
   myPaint.setColor(Color.argb(255, 255, 255, 255));
   // Draw some text
   myCanvas.drawText("Hello World!",100, 100, myPaint);
   // Change the paint to yellow
   myPaint.setColor(Color.argb(255, 212, 207, 62));
   // Draw a circle
   myCanvas.drawCircle(400,250, 100, myPaint);
}
```

前面的代码使用`myCanvas.drawColor`方法来填充屏幕颜色。

`myPaint.setTextSize`方法定义了接下来将绘制的文本的大小。`myPaint.setColor`方法确定了未来绘制的颜色。`myCanvas.drawText`方法实际将文本绘制到屏幕上。

分析传递给`drawText`方法的参数，我们可以看到文本将会显示“Hello World!”，并且将在我们的位图（`myBitmap`）的左侧 100 像素和顶部 100 像素处绘制。

接下来，我们再次使用`setColor`方法来改变将用于绘制的颜色。最后，我们使用`drawCircle`方法来绘制一个距左侧 400 像素，顶部 100 像素的圆。圆的半径为 100 像素。

我保留了解释`Color.argb`方法直到现在。

#### 解释 Color.argb

`Color`类，不出所料，帮助我们操纵和表示颜色。先前使用的`argb`方法返回使用**alpha（不透明度/透明度），红色，绿色，蓝色**（**argb**）模型构建的颜色。该模型使用从 0（无颜色）到 255（全颜色）的值。重要的是要注意，尽管在反思时可能似乎显而易见，但混合的颜色是光的强度，与我们混合颜色时发生的情况完全不同，例如混合油漆。

注意

要设计 argb 值并进一步探索这个模型，请查看这个方便的网站：[`www.rapidtables.com/web/color/RGB_Color.html`](https://www.rapidtables.com/web/color/RGB_Color.html)。该网站可以帮助您选择 RGB 值；然后您可以尝试 alpha 值。

用于清除绘图表面的值是`255, 0, 0, 255`。这些值意味着完全不透明（纯色），没有红色，没有绿色，完全蓝色。这制作了一个蓝色。

对`argb`方法的下一个调用是在第一个调用`setColor`时，我们正在设置文本所需的颜色。值`255, 255, 255, 255`表示完全不透明，完全红色，完全绿色和完全蓝色。当您将光与这些值混合时，您会得到白色。

对`argb`方法的最终调用是在最终调用`setColor`方法时，我们正在设置绘制圆的颜色。`255, 21, 207, 62`制作了太阳黄色。

在我们运行代码之前的最后一步是添加对`setContentView`方法的调用，该方法将我们的`ImageView`（`myImageView`）放置为要设置为此应用程序内容的`View`实例。以下是要在`onCreate`方法的结束大括号之前添加的最终代码行：

```kt
// Associate the drawn upon Bitmap with the ImageView
myImageView.setImageBitmap(myBlankBitmap);
// Tell Android to set our drawing
// as the view for this app
// via the ImageView
setContentView(myImageView);
```

最后，我们通过调用`setContentView`方法告诉`Activity`类使用`myImageView`。

当您运行时，这就是`Canvas`演示的样子。我们可以看到一个 800 x 800 像素的图纸。在下一章中，我们将使用更高级的技术来利用整个屏幕，并且我们还将学习有关线程以使图形实时移动：

图 20.1 - 画布演示

](img/Figure_20.1_B16773.jpg)

图 20.1 - 画布演示

如果我们更好地了解 Android 坐标系统，将有助于理解我们在`Canvas`类绘图方法中使用的坐标的结果。

# Android 坐标系统

正如我们所看到的，绘制位图是微不足道的。但是，我们用来绘制图形的坐标系统需要简要解释。

## 绘图和绘制

当我们将`Bitmap`对象绘制到屏幕上时，我们传入要绘制对象的坐标。给定 Android 设备的可用坐标取决于其屏幕的分辨率。

例如，Google Pixel 手机在横向视图中的屏幕分辨率为 1,920 像素（横向）x 1,080 像素（纵向）。

这些坐标的编号系统从左上角的`0, 0`开始，然后向下和向右移动，直到右下角，即像素`1919, 1079`。`1920`和`1919`之间以及`1080`和`1079`之间明显的 1 像素差异是因为编号从 0 开始。

因此，当我们将位图或其他任何东西绘制到屏幕上（例如`Canvas`圆和矩形），我们必须指定一个`x，y`坐标。

此外，位图（或`Canvas`形状）当然包括许多像素。那么，给定位图的哪个像素会在我们指定的`x，y`屏幕坐标处绘制？

答案是`Bitmap`对象的左上角像素。看看下一个图，它应该使用 Google Pixel 手机作为示例来澄清屏幕坐标。作为解释 Android 坐标绘图系统的图形手段，我将使用一个可爱的太空飞船图形：

![图 20.2 - 屏幕坐标](img/Figure_20.2_B16773.jpg)

图 20.2 - 屏幕坐标

此外，坐标是相对于您所绘制的内容的。因此，在我们刚刚编写的`Canvas`演示应用程序以及下一个演示应用程序中，坐标是相对于位图（`myBitmap`）的。在下一章中，我们将使用整个屏幕，前面的图将准确地表示发生的情况。

让我们再做一些绘图，这次使用来自图形文件的位图。我们将使用与此应用程序中看到的相同的起始代码。

# 创建位图

在我们深入代码之前，让我们先做一点理论，考虑一下我们将如何在屏幕上呈现图像。要绘制位图，我们将使用`Canvas`类的`drawBitmap`方法。

首先，我们需要在`res/drawable`文件夹中的项目中添加一个位图；我们将在接下来的`Bitmap`演示应用程序中真正做到这一点。现在，假设图形文件/位图的名称为`myImage.png`。

接下来，我们声明一个`Bitmap`类型的对象，就像我们在上一个演示中用于背景的`Bitmap`对象一样：

```kt
Bitmap mBitmap;
```

接下来，我们需要使用我们之前添加到项目的`drawable`文件夹中的首选图像来初始化`mBitmap`对象：

```kt
mBitmap = BitmapFactory.decodeResource
                (getResources(), R.drawable.myImage);
```

`BitmapFactory`方法的静态`decodeResource`方法用于初始化`mBitmap`。它接受两个参数。第一个是对`getResources`的调用，这是由`Activity`类提供的。这个方法，顾名思义，可以访问项目资源，第二个参数`R.drawable.myImage`指向`drawable`文件夹中的`myImage.png`文件。位图（`mBitmap`）现在已经准备好由`Canvas`类绘制。

然后，您可以使用以下代码绘制位图：

```kt
// Draw the bitmap at coordinates 100, 100
mCanvas.drawBitmap(mBitmap, 
                100, 100, mPaint);
```

下面是前一节中的飞船图形在屏幕上绘制时的样子，作为我们讨论旋转位图时的参考：

![图 20.3 - 飞船图形](img/Figure_20.3_B16773.jpg)

图 20.3 - 飞船图形

# 操作位图

然而，通常情况下，我们需要以旋转或其他方式改变的状态绘制位图。很容易使用 Photoshop 或您喜欢的任何图像编辑软件从原始位图创建更多位图以面向其他方向。然后，当我们来绘制我们的位图时，我们可以简单地决定面向哪个方向，并绘制适当的预加载位图。

然而，我认为如果我们只使用一个单一的源图像，并学习 Android 提供的用于在我们的 Java 代码中操作图像的类，将会更有趣和有教育意义。然后，您将能够将旋转和反转的图形添加到您的应用程序开发工具包中。

## 位图到底是什么？

位图之所以被称为位图，是因为它确实就是这样：一张*位的地图*。虽然有许多使用不同范围和值来表示颜色和透明度的位图格式，但它们都归结为同一件事。它们是一组/地图值，每个值代表一个像素的颜色。

因此，要旋转、缩放或反转位图，我们必须对位图的每个像素/位进行适当的数学计算。这些计算并不是非常复杂，但也不是特别简单。如果您完成了高中的数学课程，您可能不会太费力地理解这些数学知识。

不幸的是，仅仅理解数学是不够的。我们还需要设计高效的代码，以及了解位图格式，然后为每种格式修改我们的代码。这并不是微不足道的。幸运的是，Android API 已经为我们做好了一切。见识一下`Matrix`类。

## Matrix 类

这个类被命名为`Matrix`，因为它使用数学概念和规则来对一系列值（称为矩阵的复数）进行计算。

注意

Android 的`Matrix`类与同名电影系列无关。然而，作者建议所有有抱负的应用程序开发者服用红色药丸。

你可能对矩阵很熟悉，但如果你不熟悉也不用担心，因为`Matrix`类隐藏了所有的复杂性。此外，`Matrix`类不仅允许我们对一系列值进行计算，还具有一些预先准备的计算，使我们能够做一些事情，比如围绕另一个点旋转特定角度的点。所有这些都不需要了解三角学。

注意

如果你对数学运算的工作方式感到好奇，并且想要一个绝对初学者指南来了解旋转游戏对象的数学知识，那么请看看我网站上以可飞行和可旋转的飞船为结尾的一系列 Android 教程：

[`gamecodeschool.com/essentials/calculating-heading-in-2d-games-using-trigonometric-functions-part-1/`](http://gamecodeschool.com/essentials/calculating-heading-in-2d-games-using-trigonometric-functions-part-1/)

[`gamecodeschool.com/essentials/rotating-graphics-in-2d-games-using-trigonometric-functions-part-2/`](http://gamecodeschool.com/essentials/rotating-graphics-in-2d-games-using-trigonometric-functions-part-2/)

[`gamecodeschool.com/android/2d-rotation-and-heading-demo/`](http://gamecodeschool.com/android/2d-rotation-and-heading-demo/)

本书将继续使用 Android 的`Matrix`类，但在下一章创建粒子系统时，我们将进行稍微更高级的数学运算。

### 将位图反转以面对相反方向

首先，我们需要创建`Matrix`类的一个实例。下一行代码以熟悉的方式调用`new`来调用默认构造函数：

```kt
Matrix matrix = new Matrix();
```

注意

请注意，您现在不需要将任何此代码添加到项目中；稍后将再次显示所有与`Matrix`相关的代码，其中将提供更多的上下文。我只是认为在此之前单独看到所有与`Matrix`相关的代码会更容易一些。

现在我们可以使用`Matrix`类的许多巧妙方法之一。`preScale`方法需要两个参数：一个用于水平变化，一个用于垂直变化。看看下面的代码：

```kt
matrix.preScale(-1, 1);
```

`preScale`方法将循环遍历每个像素位置，并将所有水平坐标乘以`-1`，所有垂直坐标乘以`1`。

这些计算的效果是所有垂直坐标将保持不变，因为如果乘以 1，那么数字不会改变。然而，当你乘以-1 时，像素的水平位置将被倒置。看下面的例子：

水平位置 0、1、2、3 和 4 将变为 0、-1、-2、-3 和-4。

在这个阶段，我们已经创建了一个可以对位图执行必要计算的矩阵。我们实际上还没有对位图做任何事情。要使用这个矩阵，我们调用`Bitmap`类的`createBitmap`方法，如下面所示的代码行：

```kt
mBitmapLeft = Bitmap
.createBitmap(mBitmap,
          0, 0, 25, 25, matrix, true);
```

先前的代码假设`mBitmapLeft`已经初始化，以及`mBitmap`。`createBitmap`方法的参数解释如下：

+   `mBitmapHeadRight`是一个`Bitmap`对象，已经被创建和缩放，并且已经加载了一个飞船（面向右侧）的图像。这个图像将被用作创建新位图的源。源位图实际上不会被改变。

+   `0, 0`是我们希望将新位图映射到的水平和垂直起始位置。

+   `25, 25`参数是设置位图缩放大小的值。

+   下一个参数是我们预先准备的`Matrix`实例`matrix`。

+   最后一个参数`true`指示`createBitmap`方法需要过滤以正确处理位图的创建。

当绘制到屏幕上时，`mBitmapLeft`将如下所示：

![图 20.4 - mBitmapLeft](img/Figure_20.4_B16773.jpg)

图 20.4 - mBitmapLeft

我们还可以使用旋转矩阵创建面向上或面向下的位图。

### 旋转位图以面向上或向下

让我们看看如何旋转位图，然后我们可以构建演示应用程序。我们已经有了`Matrix`类的一个实例，所以我们所要做的就是调用`preRotate`方法，创建一个能够按照`preRotate`的单个参数指定的角度旋转每个像素的矩阵。看看这行代码：

```kt
// A matrix for rotating
matrix.preRotate(-90);

```

这有多简单？`matrix`实例现在已准备好逆时针（`-`）旋转我们传递给它的任何一系列数字，旋转`90`度。

下一行代码与我们之前解析的对`createBitmap`的调用具有完全相同的参数，只是新的`Bitmap`实例分配给了`mBitmapUp`，`matrix`的效果是执行旋转而不是`preScale`：

```kt
mBitmapUp = Bitmap
.createBitmap(mBitmap,
         0, 0, ss, ss, matrix, true);
```

这是绘制`mBitmapUp`时的样子：

![图 20.5 - mBitmapUp](img/Figure_20.5_B16773.jpg)

图 20.5 - mBitmapUp

您还可以使用相同的技术，但在`preRotate`的参数中使用不同的值来将位图向下旋转。让我们继续进行演示应用程序，看看所有这些内容是如何运作的。

# 位图操作演示应用程序

既然我们已经学习了理论，让我们来绘制和旋转一些位图。使用`操作位图`创建一个新项目。

## 将图形添加到项目中

右键单击并从*第二十章*`/操作位图/可绘制`文件夹中选择`bob.png`图形文件。

在 Android Studio 中，定位项目资源管理器窗口中的`app/res/drawable`文件夹。下一张截图清楚地显示了这个文件夹的位置以及其中带有`bob.png`图像的样子：

![图 20.6 - app/res/drawable 文件夹中的 bob.png](img/Figure_20.6_B16773.jpg)

图 20.6 - app/res/drawable 文件夹中的 bob.png

右键单击项目中的`bob.png`文件。单击两次**确定**，以确认将文件导入项目的默认选项。

编辑`MainActivity`类的代码，包括所有必需的`import`指令、`Activity`类的基本版本和一些成员变量，以便我们可以开始。此阶段`MainActivity`类的状态如下所示：

```kt
import android.app.Activity;
import android.graphics.Bitmap;
import android.graphics.Canvas;
import android.graphics.Paint;
import android.os.Bundle;
import android.widget.ImageView;
import android.graphics.BitmapFactory;
import android.graphics.Color;
import android.graphics.Matrix;
public class MainActivity extends Activity {
    // Here are all the objects(instances)
    // of classes that we need to do some drawing
    ImageView myImageView;
    Bitmap myBlankBitmap;
    Bitmap bobBitmap;
    Canvas myCanvas;
    Paint myPaint;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

    }
}
```

现在我们可以在`onCreate`中初始化所有成员：

```kt
// Initialize all the objects ready for drawing
int widthInPixels = 2000;
int heightInPixels = 1000;
// Create a new Bitmap
myBlankBitmap = Bitmap.createBitmap(widthInPixels,
         heightInPixels,
         Bitmap.Config.ARGB_8888);
// Initialize Bob
bobBitmap = BitmapFactory.decodeResource
         (getResources(), R.drawable.bob);
// Initialize the Canvas and associate it
// with the Bitmap to draw on
myCanvas = new Canvas(myBlankBitmap);
// Initialize the ImageView and the Paint
myImageView = new ImageView(this);
myPaint = new Paint();
// Draw on the Bitmap
// Wipe the Bitmap with a blue color
myCanvas.drawColor(Color.argb(255, 0, 0, 255));
```

接下来，我们添加对三种方法的调用，我们很快将编写这些方法，并将我们的新绘图设置为应用程序的视图：

```kt
// Draw some bitmaps
drawRotatedBitmaps();
drawEnlargedBitmap();
drawShrunkenBitmap();
// Associate the drawn upon Bitmap with the ImageView
myImageView.setImageBitmap(myBlankBitmap);
// Tell Android to set our drawing
// as the view for this app
// via the ImageView
setContentView(myImageView);
```

现在添加执行位图操作的`drawRotatedBitmap`方法：

```kt
void drawRotatedBitmaps(){
   float rotation = 0f;
   int horizontalPosition =350;
   int verticalPosition = 25;
   Matrix matrix = new Matrix();
   Bitmap rotatedBitmap = Bitmap.createBitmap(100,
                200,
                Bitmap.Config.ARGB_8888);
   for(rotation = 0; rotation < 360; rotation += 30){
         matrix.reset();
         matrix.preRotate(rotation);
         rotatedBitmap = Bitmap
                      .createBitmap(bobBitmap,
                                  0, 0, bobBitmap
                                  .getWidth()-1, 
                                  bobBitmap.getHeight()-1, 
                                  matrix, true);
         myCanvas.drawBitmap(rotatedBitmap, 
                      horizontalPosition, 
                      verticalPosition,  
                      myPaint);

         horizontalPosition += 120;
         verticalPosition += 70;
   }
}
```

先前的代码使用`for`循环来循环遍历 360 度，每次 30 度。循环中的每个值都用于`Matrix`实例中旋转 Bob 的图像，然后使用`drawBitmap`方法将其绘制到屏幕上。

按照下面显示的方式添加最后两种方法：

```kt
void drawEnlargedBitmap(){
   bobBitmap = Bitmap
                .createScaledBitmap(bobBitmap,
                            300, 400, false);
   myCanvas.drawBitmap(bobBitmap, 25,25, myPaint);
}
void drawShrunkenBitmap(){
   bobBitmap = Bitmap
                .createScaledBitmap(bobBitmap,
                            50, 75, false);
   myCanvas.drawBitmap(bobBitmap, 250,25, myPaint);
}
```

`drawEnlargedBitmap`方法使用`createScaledBitmap`方法，大小为 300x400 像素。然后，`drawBitmap`方法将其绘制到屏幕上。

`drawShrunkenBitmap`使用完全相同的技术，只是缩放然后绘制一个 50x75 像素的图像。

运行应用程序，看看 Bob 是如何长大、缩小，然后在 30 度间隔下 360 度旋转的，如下一张截图所示：

![图 20.7 - 应用程序运行方式

第 20.7 图 - B16773.jpg

图 20.7 - 应用程序运行方式

我们绘图库中唯一缺少的是观看所有这些活动发生的能力。我们将在下一章中填补这一知识空白。

# 常见问题

1.  我知道如何进行所有这些绘图，但我看不到任何移动。为什么？

要看到物体移动，您需要能够调节绘图的每个部分发生的时间。您需要使用动画技术。这并不是微不足道的，但对于一个有决心的初学者来说也不是难以掌握的。我们将在下一章中学习所需的主题。 

# 总结

在本章中，我们看到了如何绘制自定义形状、文本和位图。现在我们知道如何绘制和操作基本形状、文本和位图，我们可以把事情提升到一个新水平。

在下一章中，我们将开始我们的下一个重要应用程序，这是一个儿童绘画应用程序，实际上在按下按钮时会活跃起来。
