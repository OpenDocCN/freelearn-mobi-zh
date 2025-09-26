# 第十章。动画

动画是静态（静止）图像运动的感觉。为了做到这一点，通常每秒大约需要移动 1/25。移动某物的步骤越多，运动越平滑，越容易欺骗大脑。我们已经看到了 第六章，*事件*，如何使用 `UIAlertView` 实现动画。现在我们需要看看我们如何使用 `CoreAnimation` 和 `CoreGraphics` 命名空间正常地做到这一点。这不会是一个详尽无遗的研究，但它会给你一个基础知识。

在本章中，我们将涵盖以下内容：

+   处理位图（缩放和旋转）

+   使用后释放内存

# 处理位图

位图图像可以在应用程序内部或外部创建。外部位图被渲染到 `UIImageView` 中，如下面的代码所示：

```swift
uiImageView = UIImage.FromFile("path/tofile.ext");
```

我们可以获取图像，所以让我们对它做些什么。

## 缩放图像

可以通过设置缩放因子来实现缩放。

```swift
uiImageView = UIImage.FromFile("path/tofile.ext").Scale(new SizeF(float w, float h), float scaleFactor);
```

如果没有指定 `scaleFactor`，则默认为 `1.0f`。

因此，你可以创建如下所示的动画：

```swift
float w = 10f, h = 10f;
for (int i = 0; i < 100; ++i) {
  uiImageView = UIImage.FromFile("path/tofile.ext").Scale(new SizeF(w, h));
  w += (float)i + 5f;
  h += (float)i + 5f;
}
```

这给人一种图像变大的印象。这并不很好，但给你一个想法。要做得更好（比如旋转），我们需要开始查看 `CoreGraphics` 和 `CoreAnimation`。

## 旋转图像 – 第一部分

我在这里假设有一个足够小的 `UIImageView` 小部件被设置在屏幕中间（比如说 122 x 122）。和之前一样，图像被加载进来了，但这次需要使用 `CoreGraphics` 图像。

```swift
uiImageView.Image = UIImage.FromFile("graphics/image.jpg").Scale(new SizeF(122f, 122f));
uiImageView.Transform = CGAffineTransform.MakeRotation((float)Math.PI / 15f);
```

图像被加载并旋转。旋转是静态的（换句话说，瞬间）。

对于动画，需要使用 `CoreAnimation`。然而，在进行旋转动画之前，让我们从更简单的事情开始——将某个东西在屏幕上移动并返回。为了做到这一点，让我们看看一些代码。

```swift
Private PointF startPoint;

public override void ViewDidLoad() {
  base.ViewDidLoad();

  uiImageView.Image = UIImage.FromFile("graphics/image.jpg").Scale(new SizeF(122f, 122f));
  startPoint = uiImageView.Center;

  UIView.BeginAnimations("moveImage");
  UIView.SetAnimationDuration(2);
  UIView.SetAnimationCurve(UIViewAnimationCurve.EaseInOut);
  UIView.SetAnimationRepeatCount(2);
  UIView.SetAnimationRepeatAutoreverses(true);
  UIView.SetAnimationDelegate(this);
  UIView.SetAnimationDidStopSelector(new Selector("moveImageStopped:"));
  uiImageView.Center = new PointF(UIScreen.MainScreen.Bounds.Right – uiImageView.Frame.Width /2, uiImageView.Center.Y);
  UIView.CommitAnimations();
}

[Export("moveImageStopped")]
private void moveImageStopped() {
  uiImageView.Center = startPoint;
}
```

关于此代码有两个重要点需要注意。

+   代码在 `UIView` 而不是 `UIImageView` 上操作

+   Xamarin.iOS 和底层 Objective-C 之间的绑定在动画和一般绘图（绑定是选择器）中变得非常明显。

### 基础绑定

在前面的例子中，代码正在为底层 Objective-C 创建一个接口层。与正常代码相比，编译器以稍微不同的方式处理这一点。添加这种 `Selector` 代码也可以用于其他方式（例如，访问私有 API 代码——尽管应该避免这样做，因为这会使应用程序无法被接受到应用商店）。应该注意的是，与 Objective-C 层的绑定有时可能会导致提交到苹果商店的问题。

## 代码分析

之前代码的分析可以总结如下：

+   startPoint 是图像开始时的位置。

+   为了告诉应用程序将要有一个动画，需要调用 `BeginAnimations`。

+   `Duration` 是动画的长度，`RepeatCount` 是动画被调用的次数。

+   `RepeatAnimationCurve` 定义了动画如何进行（在这种情况下，重复动画曲线，曲线不一定是圆上的弧线，它也可以是直线）。

+   `EaseInOut` 从慢速开始动画，然后加速并减速。

+   `EaseIn` 从慢速开始动画。

+   `EaseOut` 在结束时减速。

+   `Linear` 提供了均匀的速度。

+   绑定在动画结束后将图像重置到中心。

+   `CommitAnimations` 设置动画开始。Xamarin.iOS 提供了一个很好的动画使用块的示例，这将进一步支持这个主题。

# 使用后释放内存

通常，一旦一个类超出作用域，垃圾收集器（GC）就会释放该类内部进程使用的内存。然而，由于 Xamarin.iOS 作为底层 Objective-C 的绑定层工作，有时释放内存变得很重要；这通常发生在处理动画和图形时。

可能最简单的方法是在创建新的 View Controller 时进行清理。

```swift
public override void DidReceiveMemoryWarning() {
  // Releases the view if it doesn't have a superview.
  base.DidReceiveMemoryWarning();
  // Release any cached data, images, etc that aren't in use.
}
```

例如，要释放 `uiImageView`，请按照以下步骤操作：

```swift
uiImageView.Release();
```

如果代码没有引发内存警告，`ViewDidDisappear()` 也可以用来以相同的方式释放内存。

另一种简单的释放内存的方法是在代码超出作用域后让 GC 做其工作。考虑以下（简化的）代码：

```swift
private async void doSomething() {
  UIImageView image = new UIImageView(new RectangleF(0, 0, 100,100));
  string filename = await GetFileName();
  image.Image = UIImage.FromFile(filename);
  // do a lot of bits and pieces
  if (condition)
    return;
  else
    callNewMethod();
}
```

代码执行并加载 `UIImageView`，按照返回的字符串指定的图像。如果 `condition` 成立（即，它是 `true`），则方法返回。如果 `condition` 不成立，方法将跳转到 `callNewMethod`。这两个问题都不大，除了垃圾收集器（GC）不会在类本身超出作用域之前被调用。因此，`UIImageView` 控件占用的任何内存仍然被使用，尽管它在一个类中只被使用了三行。由于图像太多和操作太多，内存很快就会消失。

如果你考虑一个普通的动画，可能会有 300 张带有背景的图片，所以内存很快就会被耗尽。

一个简单的解决方案是只创建和使用你需要的东西，并在代码超出作用域后调用 GC。以下代码行展示了如何做到这一点：

```swift
private async void doSomething() {
  using (UIImageView image = new UIImageView()) {
    image.Frame = new RectangleF(0, 0, 100, 100);
    string filename = await GetFileName();
    image.Image = UIImage.FromFile(filename);
  };
  // do a lot of bits and pieces
  if (condition)
    return;
  else
    callNewMethod();
}
```

虽然看起来很相似，但正在创建的图像被使用，一旦完成，使用的内存就会被释放，而不是必须等待类超出作用域。

## 旋转图像 – 第二部分

要使图像旋转，必须使用 `CoreGraphics` 图像，然后将其转换为位图。以下展示了如何进行旋转。将 `RotateCTM` 和 `TranslateCTM` 从正数变为负数（反之亦然）应该会得到不同的结果。

```swift
public static UIImage rotateImage(UIImage uiImage) {
  UIImage result;
  using (CGImage cgImage = uiImage.CGImage) {
    CGImageAlphaInfo alpha = cgImage.AlphaInfo;
    CGColorSpace colour = CGColorSpace.CreateDeviceRGB();
    if (alpha == CGImageAlphaInfo.None)
      alpha = CGImageAlphaInfo.NoneSkipLast;
    int width = cgImage.Width, height = cgImage.Height;
    CGBitmapContext bitmap = new CGBitmapContext(IntPtr.Zero, height,
    width, cgImage.BitsPerComponent, cgImage.BytesPerRow,colour, alpha);
    bitmap.RotateCTM((float)Math.PI / 2); // rotate right.
    bitmap.TranslateCTM(0, -height);
    bitmap.DrawImage(new Rectangle(0, 0, width, height), cgImage);
    result = UIImage.FromImage(bitmap.ToImage());
    bitmap = null; // free memory
  }
  return result;
}
```

# 摘要

在 iOS 上，动画和图形处理是一个广泛的主题。虽然本章对这一主题进行了简要概述，但我建议您阅读由 Michael Bluestein 所著、Pearson Education, Inc.出版的*《学习 MonoTouch》*。他的书对这一主题的覆盖比这里所允许的更为详细。
