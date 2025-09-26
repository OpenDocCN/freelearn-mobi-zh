# 第五章. 多点触控和手势支持

在本章中，我们将介绍：

+   设置触摸点输入模式

+   检测多个触摸点

+   拖动多个显示对象

+   跟踪移动

+   设置手势输入模式

+   处理滑动手势

+   平移一个对象

+   旋转一个对象

+   缩放一个对象

# 简介

iPhone 并非第一个使用这项技术的设备，但它的成功启动了触摸屏革命。对这一成功至关重要的是苹果决定包含多点触控支持。无论是单次点击、滑动还是捏合；与 iPhone 屏幕的交互总是直观而自然。

Flash 平台允许应用程序开发者充分利用多点触控，当针对 iOS 时。在本章中，我们将探讨如何处理多个触摸点，然后再介绍如何检测和响应手势。

# 设置触摸点输入模式

iPhone 的成功改变了人们使用移动设备的方式，现在用户期望通过物理触摸屏幕直接与设备交互。虽然鼠标仅限于选择单个点，但 iOS 设备可以检测多个触摸并同时跟踪它们的每个移动。

Flash 提供了对多点触控的全面支持，但为了利用它，您必须首先通知平台您的意图，即接收和使用基于触摸的事件。

让我们看看这是如何完成的。

## 准备工作

已提供一个 FLA 作为本菜谱的起点。

从本书的配套代码包中，将 `chapter5\recipe1\recipe.fla` 打开到 Flash Professional 中。

阶段上有一个名为 `output` 的动态文本字段。我们将编写一些代码以启用多点触控输入并将请求的成功或失败写入文本字段。

## 如何做...

执行以下步骤：

1.  创建一个名为 `Main` 的文档类。

1.  添加以下两个导入语句：

    ```swift
    import flash.display.MovieClip;
    import flash.ui.Multitouch;
    import flash.ui.MultitouchInputMode; 

    ```

1.  现在在构造函数中添加一些代码以启用多点触控支持：

    ```swift
    public function Main() {
     if(Multitouch.supportsTouchEvents)
    {
    Multitouch.inputMode = MultitouchInputMode.TOUCH_POINT;
    output.text = ("inputMode = " + Multitouch.inputMode);
    }
    else
    {
    output.text = "Multi-touch events not supported.";
    } 
    }

    ```

1.  将类文件保存为 `Main.as`。

1.  发布 FLA 并将 IPA 文件部署到您的 iOS 设备上。

当您运行应用程序时，以下文本应输出到屏幕上：

+   **inputMode = touchPoint**

## 它是如何工作的...

Flash 提供了各种多点触控输入模式，这些模式决定了您的应用程序可以接收的事件类型。为了接收触摸事件，您需要将 `Multitouch.inputMode` 属性设置为 `MultitouchInputMode.TOUCH_POINT`。这是您文档类中执行此操作的代码行：

```swift
Multitouch.inputMode = MultitouchInputMode.TOUCH_POINT;

```

除了设置 `Multitouch.inputMode`，您还可以查询它以确定当前选定的输入模式。对于这个菜谱，当前输入模式被写入 `output` 文本字段以确认它已成功设置：

```swift
output.text = ("inputMode = " + Multitouch.inputMode);

```

虽然 Flash 支持所有 iOS 设备的多触控，但并非所有其他操作系统和平台都如此。在编写跨平台代码时，您可能需要通过检查`Multitouch.supportsTouchEvents`属性来确认支持。在我们的代码示例中，这是在设置输入模式之前完成的。

如需了解更多关于多触控输入模式的信息，请在 Adobe 社区帮助中搜索`flash.ui.Multitouch`和`flash.ui.MultitouchInputMode`。

还强烈建议您花时间研究苹果的 iOS 人机界面指南，在那里您将找到通过多触控驱动用户体验的最佳实践。文档可以在 iOS 开发中心[`developer.apple.com/devcenter/ios`](http://developer.apple.com/devcenter/ios)找到。

## 更多内容...

让我们看看一些额外的细节。

### 可用的触控事件

设置触控的输入模式允许您监听由类型为`InteractiveObject`或继承自`InteractiveObject`的对象（如`Sprite, MovieClip`和`Stage`）分发的特定于触控的事件。在多触控屏幕上，每个手指可以接触的点称为触点。

以下可用的触控事件：

| 触控事件 | 描述 |
| --- | --- |
| `TOUCH_BEGIN` | 触点已被按下。 |
| `TOUCH_END` | 触点已被释放。 |
| `TOUCH_MOVE` | 触点正在移动。这发生在手指在屏幕上拖动时。 |
| `TOUCH_TAP` | 快速的手指轻触。 |
| `TOUCH_OVER` | 触点已移至交互对象上。 |
| `TOUCH_OUT` | 触点已从交互对象移开。 |
| `TOUCH_ROLL_OVER` | 触点已移至交互对象上。与`TOUCH_OVER`不同，此事件不会对对象的所有子对象触发。 |
| `TOUCH_ROLL_OUT` | 触点已从交互对象移开。与`TOUCH_OUT`不同，此事件不会对对象的所有子对象触发。 |

在接下来的几个示例中，我们将看到一些这些事件的实际应用。

### 确定支持的触点数量

您可以通过检查静态属性`Multitouch.maxTouchPoints`来确定您的 iOS 设备支持的触点数量。

虽然您会发现 iOS 设备支持 5 个触点，但此属性在跨平台项目中更有实用价值，因为在不同的操作系统和输入设备选择下，触点数量可能会有很大的变化。当针对多个平台时，请考虑这一点来优化您的内容。

### 触点击中目标

对于您应用程序中可以点击的元素，请确保使用至少与指尖大小相当的击中区域。在标准分辨率屏幕上，这相当于大约 44x44 像素，在 Retina 显示屏上为 88x88 像素。

### 鼠标事件

默认输入模式是 `MultitouchInputMode.NONE`，这表示所有与触摸设备交互的用户操作都被解释为鼠标事件。然而，与 `MultitouchInputMode.TOUCH_POINT` 不同，任何时刻只能处理一个触摸点。

注意，当输入模式设置为 `MultitouchInputMode.TOUCH_POINT` 时，你可以继续监听和响应由 `flash.events.MouseEvents` 提供的鼠标事件。

### 在 ADL 中测试

很可能在 ADL 中测试此菜谱代码会导致以下消息输出到屏幕：

+   **不支持多点触控事件。**

并非所有台式计算机或操作系统都支持多点触控。当在代码中使用基于触摸的事件时，建议直接在 iOS 设备上测试。

## 参见

+   *创建基本文档类，第三章*

+   *处理用户交互，第四章*

+   *检测多个触摸点*

+   *设置手势输入模式*

# 检测多个触摸点

多点触控术语指的是同时检测和跟踪触摸屏上两个或更多不同接触点的功能。触摸事件类似于 Flash 提供的鼠标事件，除了你可以同时监听和响应多个触摸事件。

让我们回顾一下第三章 中的 Bubbles 应用程序，并向其中添加多点触控交互。我们将添加功能，使用户可以通过在每个气泡上方放置手指来捕获多个气泡。

## 准备工作

已为你提供了一个 Bubbles FLA 版本供你工作。

从 Flash Professional 打开书籍附带代码包中的 `chapter5\recipe2\recipe.fla`。

## 如何实现...

我们将对 FLA 的文档类和 `Bubble.as` 进行修改。

### 更新 Bubble 类

让我们从向 `Bubble.as` 添加一些代码开始，以防止在触摸时移动任何气泡实例：

1.  打开 `Bubble.as` 类。

1.  添加以下成员变量：

    ```swift
    private var _held:Boolean = false;

    ```

1.  现在为 `_held` 变量编写获取器和设置器方法：

    ```swift
    public function set held(h:Boolean):void {
    _held = h;
    }
    public function get held():Boolean {
    return _held;
    }

    ```

1.  移动到 `update()` 方法，并在开始处编写几行代码以防止在触摸时更新气泡的逻辑：

    ```swift
    public function update():void {
     if(_held)
    {
    return;
    } 

    ```

1.  最后，在构造函数中，禁用气泡内任何子显示对象的用户输入——我们只对接收气泡容器的事件感兴趣：

    ```swift
    public function Bubble() {
    mouseChildren = false; 
    }

    ```

1.  保存 `Bubble.as`。

### 响应多个触摸事件

现在在 FLA 的文档类中，让我们监听多点触控事件并捕获任何被触摸的气泡：

1.  打开 `Main.as` 文档类。

1.  包含以下三个导入语句：

    ```swift
    import flash.ui.Multitouch;
    import flash.ui.MultitouchInputMode;
    import flash.events.TouchEvent;

    ```

1.  在构造函数中设置输入模式，并监听从舞台派发的 `TouchEvent.TOUCH_BEGIN` 和 `TouchEvent.TOUCH_END`：

    ```swift
    public function Main() {
     Multitouch.inputMode = MultitouchInputMode.TOUCH_POINT;
    stage.addEventListener(TouchEvent.TOUCH_BEGIN, touchBegin);
    stage.addEventListener(TouchEvent.TOUCH_END, touchEnd); 
    application = NativeApplication.nativeApplication;

    ```

1.  为 `TOUCH_BEGIN` 事件编写处理程序：

    ```swift
    private function touchBegin(e:TouchEvent):void {
    var b:Bubble = e.target as Bubble;
    if(b != null)
    {
    b.held = true;
    }
    }

    ```

1.  并为 `TOUCH_END` 事件添加处理程序：

    ```swift
    private function touchEnd(e:TouchEvent):void {
    var b:Bubble = e.target as Bubble;
    if(b != null)
    {
    b.held = false;
    }
    }

    ```

1.  保存 `Main.as`。

1.  发布 FLA 文件并将 IPA 文件部署到您的设备上。

运行应用，并将手指放在多个气泡上以阻止它们向上漂浮。将手指从气泡上抬起将再次释放它。

## 它是如何工作的...

此菜谱利用了多点触控，允许同时检测多个对象的触摸点。

在`Main`类的构造函数中，通过以下代码行启用了触摸功能：

```swift
Multitouch.inputMode = MultitouchInputMode.TOUCH_POINT;

```

任何`InteractiveObject`或其子类，如`Sprite, MovieClip`和`Stage`，都可以分发触摸事件。

已经将以下两个事件的监听器添加到了舞台：

+   `TouchEvent.TOUCH_BEGIN`

+   `TouchEvent.TOUCH_END`

当用户将手指按在任何舞台的子交互对象上时，会触发`TOUCH_BEGIN`事件。即使屏幕的其他区域已经被触摸，这也是这种情况。当用户从交互对象上抬起手指时，会分发`TOUCH_END`事件。

在此菜谱中，舞台的子对象包括气泡实例和单个背景电影剪辑。事件处理程序只是检查事件是否是从`Bubble`实例分发的，如果是，则将气泡的`held`属性设置为。对于`TOUCH_BEGIN`事件，将气泡的`held`属性设置为`true`，从而将其冻结。当接收到`TOUCH_END`事件时，将气泡的`held`属性设置为`false`，允许它再次向上移动。

您可以从 Adobe 社区帮助中获取有关`flash.events.TouchEvent`类的更多信息。

### 处理滚动退出

有时用户的指尖可能从气泡上滑开，而不是干净地抬起。当在此菜谱的示例代码中发生这种情况时，被持有的气泡不会收到`TOUCH_END`事件，因此不会再次开始向上漂浮。

您可以通过监听每个气泡实例分发的`TouchEvent.TOUCH_ROLL_OUT`来纠正这一点。只需在`Main`类的构造函数中添加以下行：

```swift
for(var i:uint = 0; i < bubbles.length; i++)
{
bubbles[i].speed = speeds[i];
bubbles[i].addEventListener(TouchEvent.TOUCH_ROLL_OUT, touchEnd); 
}

```

现在，当用户将手指从气泡上滑开时，`touchEnd()`处理程序将被调用，气泡将再次开始移动。

### 注意

您可以将此菜谱中的代码带入下一个菜谱。如果您为`TouchEvent.TOUCH_ROLL_OUT`添加了事件监听器，则在继续之前请将其再次移除，因为它不是必需的，如果保留它会产生奇怪的行为。

## 还有更多...

您的处理程序收到的每个事件对象都包含与该触摸事件相关的多个属性。在检测多个触摸点时，您可能会发现以下属性很有用。

### 主要触摸点

当在屏幕的多个位置触摸时，首先触摸的位置被认为是主要触摸点。当您的处理程序接收到`TouchEvent`对象时，您可以通过查询其`isPrimaryTouchPoint`属性来确定这一点。

### 触摸点 ID

每个新的触摸点都会分配一个唯一的 ID，并用于所有与该接触点相关的事件。您可以通过检查事件的 `touchPointID` 属性来确定与事件相关联的触摸点。

### 本地触摸坐标

您可以使用 `localX` 和 `localY` 属性来确定触摸事件相对于交互对象的坐标。请注意，由于技术的性质和指尖覆盖的表面积，这两个属性在触摸屏设备上不会提供像素级的精确度。

### 全局触摸坐标

您还可以获取事件发生的位置，在全局舞台坐标中。只需检查事件的 `stageX` 和 `stageY` 属性的值。

## 参见

+   *将类链接到电影剪辑符号，第三章*

+   *使用更新循环，第三章*

+   *设置触摸点输入模式*

+   *拖动多个显示对象*

# 拖动多个显示对象

在触摸屏上，用手指拖动显示对象的行为非常直观。Adobe AIR 提供了 API 调用，允许这种类型的交互而不需要太多努力。此外，iOS 的多点触控功能可以被利用，以允许同时拖动多个对象。

我们将继续在 *检测多个触摸点* 配方中留下的地方，并添加拖动气泡在屏幕周围的能力。

## 准备工作

如果您还没有这样做，请在继续之前完成 *检测多个触摸点* 配方。

您可以继续使用在该配方中编写的代码。或者，从本书的配套代码包中，将 `chapter5\recipe3\recipe.fla` 打开到 Flash Professional 中，并从那里开始工作。

## 如何做...

打开 FLA 的文档类并执行以下步骤：

1.  声明以下成员变量：

    ```swift
    private var touching:Array;

    ```

1.  此数组将用于将触摸点映射到 `Bubble` 实例。

1.  在构造函数中，将 `touching` 成员变量设置为空数组：

    ```swift
    public function Main() {
    touching = [];
    Multitouch.inputMode = MultitouchInputMode.TOUCH_POINT;
    stage.addEventListener(TouchEvent.TOUCH_BEGIN, touchBegin);
    stage.addEventListener(TouchEvent.TOUCH_END, touchEnd);

    ```

1.  移动到 `touchBegin()` 事件处理程序并做出以下更改：

    ```swift
    private function touchBegin(e:TouchEvent):void {
    var b:Bubble = e.target as Bubble;
    if(b != null && !b.held)
    {
    b.held = true;
    b.startTouchDrag(e.touchPointID);
    touching[e.touchPointID] = b;
    }
    }

    ```

1.  现在也修改 `touchEnd()` 处理程序：

    ```swift
    private function touchEnd(e:TouchEvent):void {
    var b:Bubble = touching[e.touchPointID];
    if(b != null)
    {
    b.held = false;
    b.stopTouchDrag(e.touchPointID);
    delete touching[e.touchPointID];
    touching[e.touchPointID] = null;
    }
    }

    ```

1.  保存 `Main.as`。

1.  发布 FLA 并将 IPA 部署到您的设备上。

当您运行应用程序时，您现在将能够停止并拖动气泡。您甚至可以同时拖动多个气泡。试试看。

## 它是如何工作的...

该配方中的大部分工作都是由 `startTouchDrag()` 和 `stopTouchDrag()` 执行的。这两个方法都是由 `flash.display.Sprite` 定义的，并且也适用于继承它的类，包括我们的 `Bubble` 类。

`startTouchDrag()` 方法允许用户拖动气泡穿过屏幕。它是通过将触摸点 ID 与气泡关联，并持续更新气泡的位置以反映触摸点的位置来实现的。

另一方面，`stopTouchDrag()` 方法通过指定的触摸点 ID 停止气泡的拖动。

让我们看看调用 `startTouchDrag()` 的 `touchBegin()` 事件处理器。有两行代码特别引人注目：

```swift
private function touchBegin(e:TouchEvent):void {
var b:Bubble = e.target as Bubble; 
if(b != null && !b.held)
{
b.held = true;
b.startTouchDrag(e.touchPointID); 
touching[e.touchPointID] = b;
}
}

```

首先，通过查询事件对象的 `target` 属性来获取被触摸的交互对象，并尝试将其转换为 `Bubble` 对象。如果对象是气泡，则使用事件对象的触摸点 ID 作为参数调用其 `startTouchDrag()` 方法。

屏幕上的每个接触点都被分配了一个唯一的 ID，这些 ID 可以用于 `startTouchDrag()` 和 `stopTouchDrag()` 等方法。您可以通过检查事件的 `touchPointID` 属性来确定与事件相关联的触摸点。我们利用这些唯一的 ID 在代码中将气泡与触摸点关联起来。您可以在 `touchBegin()` 事件处理器的末尾看到这种映射：

```swift
private function touchBegin(e:TouchEvent):void {
var b:Bubble = e.target as Bubble;
if(b != null && !b.held)
{
b.held = true;
b.startTouchDrag(e.touchPointID);
touching[e.touchPointID] = b; 
}
}

```

处理器使用 `touching` 成员变量来存储用户正在拖动的气泡的引用。气泡实例被放置在 `touching` 数组中，事件对象的触摸点 ID 被用作索引位置。

现在，让我们看看 `touchEnd()` 事件处理器，我们从 `touching` 数组中获取用户刚刚释放的气泡：

```swift
private function touchEnd(e:TouchEvent):void {
var b:Bubble = touching[e.touchPointID]; 
if(b != null)
{
b.held = false;
b.stopTouchDrag(e.touchPointID); 
delete touching[e.touchPointID];
touching[e.touchPointID] = null; 
}
}

```

在上述方法中，通过获取事件对象的 `touchPointID` 并使用它从 `touching` 数组中检索相关的气泡。如果检索到了气泡实例，则使用事件对象的触摸点 ID 作为参数调用其 `stopTouchDrag()` 方法。这将停止用户用手指拖动气泡。

最后，在方法末尾，从 `touching` 数组中移除气泡引用，因为它不再需要：

```swift
private function touchEnd(e:TouchEvent):void {
var b:Bubble = touching[e.touchPointID];
if(b != null)
{
b.held = false;
b.stopTouchDrag(e.touchPointID);
delete touching[e.touchPointID];
touching[e.touchPointID] = null;
}
}

```

那就是全部内容。通过使用 `Sprite` 类的 `startTouchDrag()` 和 `stopTouchDrag()` 方法以及跟踪触摸点 ID，我们能够管理多个交互对象的拖动。

您可以从 Adobe 社区帮助中获取有关 `flash.display.Sprite` 类的更多信息。

## 还有更多...

以下是一些关于 `startTouchDrag()` 方法的更多信息。

### startTouchDrag() 参数

`startTouchDrag()` 方法期望有三个参数，其中只有第一个是必需的：

+   `touchPointID:` 将用于拖动精灵的触摸点的 ID。

+   `lockCenter:` 是否将精灵的中心锁定到触摸点（`true`），还是锁定到与精灵接触的点（`false`）。默认为 `false`。

+   `bounds:` 定义正在拖动的精灵的约束区域的 `Rectangle`。默认为 `null`，表示没有约束区域。

此菜谱的示例代码只使用了第一个参数。这意味着气泡的中心没有被锁定到用户的指尖，用户可以自由地将气泡拖动到屏幕上的任何位置。

要强制将气泡的中心锁定到用户的指尖，只需将 `true` 作为第二个参数传递：

```swift
b.startTouchDrag(e.touchPointID, true);

```

以下示例还将气泡限制在屏幕左上角的 200x200 像素区域内：

```swift
b.startTouchDrag(
e.touchPointID, true, new Rectangle(0, 0, 200, 200));

```

尝试将每个示例应用到你的代码中。

## 参见

+   *设置触摸点输入模式*

+   *跟踪移动*

# 跟踪移动

我们还没有详细研究的一个重要基于触摸的事件是 `TouchEvent.TOUCH_MOVE`。当用户在屏幕上移动手指时，此事件被触发，可以通过查询来确定接触点的坐标。每次用户的手指改变位置时，都会派发一个新的 `TOUCH_MOVE` 事件。

虽然 `startTouchDrag()` 和 `stopTouchDrag()` 建议用于拖动对象，但你也可以通过响应 `TOUCH_MOVE` 事件来更新对象的位置。

让我们对 *拖动多个显示对象* 配方中的代码进行一些修改以实现这一点。

## 准备工作

在继续之前，如果你还没有这样做，请完成 *拖动多个显示对象* 配方。

你可以继续使用那个配方中的代码。或者，从本书的配套代码包中，打开 `chapter5\recipe4\recipe.fla` 到 Flash Professional 中，并从那里开始工作。

## 如何操作...

在 FLA 的文档类中执行以下步骤：

1.  移动到构造函数并添加对 `TouchEvent.TOUCH_MOVE:` 的监听器：

    ```swift
    public function Main() {
    touching = [];
    Multitouch.inputMode = MultitouchInputMode.TOUCH_POINT;
    stage.addEventListener(TouchEvent.TOUCH_BEGIN, touchBegin);
    stage.addEventListener(TouchEvent.TOUCH_END, touchEnd);
    stage.addEventListener(TouchEvent.TOUCH_MOVE, touchMove); 

    ```

1.  通过从 `if` 语句中删除高亮显示的代码行，从 `touchBegin()` 处理器中删除对 `startTouchDrag()` 的调用：

    ```swift
    private function touchBegin(e:TouchEvent):void {
    var b:Bubble = e.target as Bubble;
    if(b != null && !b.held)
    {
    b.held = true;
    b.startTouchDrag(e.touchPointID, true); 
    touching[e.touchPointID] = b;
    }
    }

    ```

1.  还要从 `touchEnd()` 处理器中删除对 `stopTouchDrag()` 的调用，通过从 `if` 语句中删除高亮显示的代码行：

    ```swift
    private function touchEnd(e:TouchEvent):void {
    var b:Bubble = touching[e.touchPointID];
    if(b != null)
    {
    b.held = false;
    b.stopTouchDrag(e.touchPointID); 
    delete touching[e.touchPointID];
    touching[e.touchPointID] = null;
    }
    }

    ```

1.  现在编写 `touchMove()` 事件处理器：

    ```swift
    private function touchMove(e:TouchEvent):void {
    var b:Bubble = touching[e.touchPointID] as Bubble;
    if(b != null)
    {
    b.x = e.stageX;
    b.y = e.stageY;
    }
    }

    ```

1.  保存 `Main.as`。

1.  发布 FLA 并将 IPA 部署到您的设备上。

此示例应与上一个配方以相同的方式表现。

## 它是如何工作的...

虽然之前通过 `startTouchDrag()` 和 `stopTouchDrag()` 来处理拖动，但在这个配方中，我们通过响应 `TouchEvent.TOUCH_MOVE` 来手动更新气泡的位置。

大部分工作是在 `touchMove()` 事件处理器中完成的，如下所示：

```swift
private function touchMove(e:TouchEvent):void {
var b:Bubble = touching[e.touchPointID] as Bubble;
if(b != null)
{
b.x = e.stageX;
b.y = e.stageY;
}
}

```

使用事件的触摸点 ID，从 `touching` 数组中检索与其关联的气泡（如果有）。然后更新气泡的 `x` 和 `y` 属性，以反映用户手指在屏幕上的位置，该位置是从事件的 `stageX` 和 `stageY` 属性中检索到的。

每当用户的手指在设备屏幕上改变位置时，`TOUCH_MOVE` 事件就会被派发，并且与该接触点关联的任何气泡都会重新定位以匹配手指的位置。

尽管这个配方主要集中在使用 `TOUCH_MOVE` 来执行拖动，但此事件可以应用于许多其他任务。例如，它可以用于跟踪绘画应用程序中的手指移动或为游戏角色绘制路径。

如果你正在你的应用中实现拖动，建议你使用 `startTouchDrag()` 和 `stopTouchDrag()`，这比监听 `TOUCH_MOVE` 事件有更好的性能。然而，既然你已经熟悉了 `TOUCH_MOVE` 事件，你应该能够将其用于各种其他用途。

## 参考也

+   *设置触摸点输入模式*

+   *拖动多个显示对象*

# 设置手势输入模式

除了处理简单的多触点事件外，AIR for iOS 还提供了对缩放、旋转和滑动等手势的支持。手势是由一系列简单的多触点事件组成的单个事件。虽然你可以捕获多触点事件并将它们转换为手势，但 Flash 平台也提供了对最常见手势的支持，从而减轻了你的工作量。

为了接收手势事件，你必须选择适当的输入模式。

让我们看看这是如何完成的。

## 准备工作

从 Flash Professional 中，打开书籍附带代码包中的 `chapter5\recipe5\recipe.fla`。

在舞台上坐着的是一个名为 `output` 的动态文本字段。我们将编写一些代码以启用手势输入，并将请求的成功或失败写入文本字段。

## 如何操作...

执行以下步骤：

1.  创建一个文档类，并将其命名为 `Main`。

1.  添加以下两个导入语句：

    ```swift
    import flash.display.MovieClip;
     import flash.ui.Multitouch;
    import flash.ui.MultitouchInputMode; 

    ```

1.  现在在构造函数中，添加一些代码以启用手势支持：

    ```swift
    public function Main() {
     if(Multitouch.supportsGestureEvents)
    {
    Multitouch.inputMode = MultitouchInputMode.GESTURE;
    output.text = ("inputMode = " + Multitouch.inputMode);
    }
    else
    {
    output.text = "Gesture events not supported.";
    } 
    }

    ```

1.  将类文件保存为 `Main.as`。

1.  发布 FLA 并将 IPA 文件部署到你的 iOS 设备上。

当你运行应用时，以下文本应显示在屏幕上：

+   **inputMode = gesture**

## 它是如何工作的...

Flash 提供了各种多触点输入模式，这些模式决定了你的应用可以接收的事件类型。为了接收手势事件，你需要将 `Multitouch.inputMode` 属性设置为 `MultitouchInputMode.GESTURE`。以下是从你的文档类中执行此操作的代码行：

```swift
Multitouch.inputMode = MultitouchInputMode.GESTURE;

```

除了设置 `Multitouch.inputMode`，你还可以查询它以确定当前选定的输入模式。在这个菜谱的代码示例中，当前输入模式被写入 `output` 文本字段以确认它已成功设置：

```swift
output.text = ("inputMode = " + Multitouch.inputMode);

```

虽然 Flash 在所有 iOS 设备上提供手势支持，但在编写跨平台代码时，你可能想通过检查 `Multitouch.supportsGestureEvents` 属性来确认支持。在我们的代码示例中，这是在设置输入模式之前完成的。

应注意，当输入模式设置为处理手势时，你将无法从 `TouchEvent` 类接收基本的触摸事件。如果你需要同时接收触摸和手势事件，那么你需要选择 `MultitouchInputMode.TOUCH_POINT` 输入模式，并在将它们合成手势事件之前捕获多个触摸事件。

关于手势输入模式的更多信息，请在 Adobe 社区帮助中搜索`flash.ui.Multitouch`和`flash.ui.MultitouchInputMode`。

苹果非常重视手势的正确使用，以确保应用程序之间的一致性。苹果的 iOS 人机界面指南包含了一系列标准手势以及用户通常使用它们执行的操作。当在您自己的应用程序中支持手势时，尽量不偏离预期的行为。

## 还有更多...

以下是一些关于手势的额外细节。

### 可用的手势事件和类型

设置手势的输入模式允许您监听由类型为`InteractiveObject`或继承自`InteractiveObject`的对象（如`Sprite, MovieClip`, 和`Stage`）分发的特定手势事件。

与触摸点输入一样，手势也可以利用多个触摸点。

以下手势事件可用：

+   `GestureEvent.GESTURE_TWO_FINGER_TAP:` 当使用两个手指轻击屏幕时触发。

+   `TransformGestureEvent.GESTURE_PAN:` 尝试移动通常太大而无法适应屏幕的内容。当两个手指在屏幕内容上移动时，平移手势被触发。

+   `TransformGestureEvent.GESTURE_ROTATE:` 当两个触摸点相互旋转时触发。此手势通常用于旋转屏幕上的内容。

+   `TransformGestureEvent.GESTURE_SWIPE:` 用手指快速在屏幕上划过。滑动手势通常用于滚动列表或快速在信息页之间切换。

+   `TransformGestureEvent.GESTURE_ZOOM:` 两个触摸点要么相互靠近，要么相互远离。此手势通常用于缩放屏幕上的内容。

我们将在本章剩余部分看到一些这些事件的实际应用。

### 确定支持的手势

您可以使用静态属性`Multitouch.supportedGestures`获取您的 iOS 设备支持的手势列表。将返回一个字符串向量数组，其中每个字符串代表一个手势事件。

### 鼠标事件

默认输入模式是`MultitouchInputMode.NONE`，它指定所有与触摸设备交互的用户操作都被解释为鼠标事件。然而，与`MultitouchInputMode.GESTURE`和`MultitouchInputMode.TOUCH_POINT`不同，任何时刻只能处理一个触摸点。

注意，当输入模式设置为`MultitouchInputMode.GESTURE`时，您可以继续监听并响应由`flash.events.MouseEvents`提供的鼠标事件。

### 在 ADL 中进行测试

很可能在 ADL 中测试此代码配方会导致屏幕上输出以下消息：

+   **不支持的手势事件。**

并非所有台式电脑或操作系统都支持基于手势的事件。当在代码中使用手势时，建议您直接在 iOS 设备上进行测试。

## 参见

+   *处理用户交互*，第四章

+   *设置触摸点输入模式*

+   *处理滑动手势*

+   *平移对象*

# 处理滑动手势

触摸屏的引入使得在信息页面之间移动的过程变得更加自然。iPhone 使得简单的触摸手势，如滑动，变得流行。例如，许多图片查看器应用程序允许用户快速滑动手指以查看序列中的下一张图片。

`TransformGestureEvent.GESTURE_SWIPE` 事件可用，允许检测并处理垂直和水平滑动手势。

让我们看看这是如何完成的。

## 准备工作

从 Flash Professional 中，打开书籍附带代码包中的 `chapter5\recipe6\recipe.fla`。

在舞台上有一个名为 `content` 的容器电影剪辑，其宽度是舞台的两倍以上。容器内部有两个不同的气泡，任何时刻只能有一个气泡能够显示在屏幕上。我们将编写代码，让用户可以在两个气泡之间水平滑动。

## 如何完成...

执行以下步骤以监听并响应水平滑动手势：

1.  创建一个文档类，并将其命名为 `Main`。

1.  包含以下导入语句：

    ```swift
    import flash.display.MovieClip;
     import flash.events.TransformGestureEvent;
    import flash.ui.Multitouch;
    import flash.ui.MultitouchInputMode;
    import fl.transitions.Tween;
    import fl.motion.easing.Sine; 

    ```

1.  在类的构造函数中添加以下代码：

    ```swift
    public function Main() {
     Multitouch.inputMode = MultitouchInputMode.GESTURE;
    stage.addEventListener(TransformGestureEvent.GESTURE_SWIPE, swipe);
    } 

    ```

1.  现在添加滑动手势的事件处理程序：

    ```swift
    private function swipe(e:TransformGestureEvent):void {
    var t:Tween;
    if(e.offsetX == -1) 
    {
    t = new Tween(content, "x", Sine.easeOut, content.x, -465, 0.25, true);
    }
    else if(e.offsetX == 1) 
    {
    t = new Tween(content, "x", Sine.easeOut, content.x, -35, 0.25, true);
    }
    }

    ```

1.  将类文件保存为 `Main.as`。

1.  发布 FLA 并将 IPA 部署到您的设备上。

快速从右向左滑动手指以移动到第二个气泡，从左向右滑动以返回到第一个。

## 它是如何工作的...

手势事件可以由任何 `InteractiveObject` 或其子类（如 `Sprite, MovieClip` 和 `Stage`）触发。

在这个菜谱中，我们利用手势输入模式来监听并响应 `TransformGestureEvent.GESTURE_SWIPE`。

通过以下代码行启用了基于手势的输入支持：

```swift
Multitouch.inputMode = MultitouchInputMode.GESTURE;

```

然后添加了对 `GESTURE_SWIPE` 事件的监听器：

```swift
stage.addEventListener(TransformGestureEvent.GESTURE_SWIPE, swipe);

```

注意，监听器实际上被添加到了舞台，允许用户在屏幕上的任何位置滑动，而不仅仅是 `content` 电影剪辑上。

在 `swipe()` 处理程序中，我们响应用户的滑动手势并将 `content` 电影剪辑滑动到其最左侧或最右侧。

为了决定将电影剪辑滑动到哪个方向，我们检查 `TransformGestureEvent` 对象的 `offsetX` 属性。值为 `1` 表示用户向右滑动，而值为 `-1` 表示用户向左滑动。您可以看到以下代码被突出显示：

```swift
private function swipe(e:TransformGestureEvent):void {
var t:Tween;
if(e.offsetX == -1)
{
t = new Tween(content, "x",
Sine.easeOut, content.x, -465, 0.25, true);
}
else if(e.offsetX == 1)
{
t = new Tween(content, "x",
Sine.easeOut, content.x, -35, 0.25, true);
}
}

```

最后，实际的内容滚动操作被执行。使用 `fl.transitions.Tween` 类将 `content` 电影剪辑滑动到正确位置。为了在滑动过渡期间实现逐渐加速和减速，使用了 `fl.motion.easing.Sine` 类。有关这两个类的更多信息，请参考 Adobe Community Help。

## 更多内容...

以下是与滑动手势相关的 `TransformGestureEvent` 的另一个属性。

### 垂直滑动

虽然本食谱的示例代码中没有涉及，但您同样可以通过查询事件的`offsetY`属性来轻松检测垂直滑动。向上滑动将返回`-1`的值，而向下滑动将返回`1`。

虽然水平和垂直滑动动作分别由不同的属性表示，但 AIR for iOS 不支持对角线滑动手势。用户可以水平或垂直滑动。

## 参见

+   *设置手势输入模式*

+   *平移对象*

# 平移对象

由于 iOS 设备（如 iPhone 和 iPod touch）屏幕的大小限制，用户可能需要平移或滚动以显示隐藏的内容。Flash 平台提供了`TransformGestureEvent.GESTURE_PAN`事件，您可以通过监听该事件来检测任何交互对象的平移手势。手势是通过用户将两个手指放在一个对象上并在屏幕上滑动它们来触发的。

本食谱将向您展示如何利用平移手势在您的项目中。

## 准备工作

从 Flash Professional 中，打开书籍附带代码包中的`chapter5\recipe7\recipe.fla`。

舞台上有一个名为`bubble`的电影剪辑，它太大而无法完全显示在屏幕上。我们将编写一些 ActionScript 代码，让用户可以平移电影剪辑，以便看到隐藏的部分。

## 如何操作...

执行以下步骤：

1.  创建一个名为`Main`的文档类。

1.  添加以下三个导入语句：

    ```swift
    import flash.display.MovieClip;
     import flash.events.TransformGestureEvent;
    import flash.ui.Multitouch;
    import flash.ui.MultitouchInputMode; 

    ```

1.  在构造函数中，设置输入模式并监听`TransformGestureEvent.GESTURE_PAN:`。

    ```swift
    public function Main() {
     Multitouch.inputMode = MultitouchInputMode.GESTURE;
    bubble.addEventListener(TransformGestureEvent.GESTURE_PAN, pan); 
    }

    ```

1.  编写`GESTURE_PAN`事件的处理程序：

    ```swift
    private function pan(e:TransformGestureEvent):void {
    bubble.x += e.offsetX;
    bubble.y += e.offsetY; 
    }

    ```

1.  将类文件保存为`Main.as`。

1.  发布 FLA 并将 IPA 部署到您的设备上。

用两个手指触摸屏幕上的气泡，然后滑动它们以平移内容。

## 它是如何工作的...

为了响应平移手势，添加了一个监听器`TransformGestureEvent.GESTURE_PAN`：

```swift
bubble.addEventListener(TransformGestureEvent.GESTURE_PAN,
pan);

```

当用户在屏幕上滑动两个手指时，`GESTURE_PAN`事件会重复发送。实际的内容平移发生在`pan()`事件处理程序中。在这里，我们使用传递给方法的`TransformGestureEvent`对象的`offsetX`和`offsetY`属性：

```swift
private function pan(e:TransformGestureEvent):void {
bubble.x += e.offsetX;
bubble.y += e.offsetY;
}

```

两个属性共同提供了用户自上次`GESTURE_PAN`事件以来手指移动的水平和垂直距离。我们只需使用这些偏移量来重新定位`bubble`电影剪辑。

更多信息，请在 Adobe 社区帮助中搜索`flash.events.TransformGestureEvent`。

## 更多...

以下是一些关于平移的最终事项，包括`TransformGestureEvent`类的一个可能对你有用的属性。

### 手势阶段

一些手势事件被分为三个不同的阶段，称为开始、更新和结束。如果您的应用程序需要响应手势事件的各个阶段，则可以查询事件的`phase`属性。每个阶段都由`flash.events.GesturePhase`类提供的常量表示。

例如，当用户在屏幕上平移内容时，`GESTURE_PAN`事件将经历这些阶段中的每一个：

+   `GesturePhase.BEGIN:` 用户用两只手指触摸屏幕并开始在其上移动

+   `GesturePhase.UPDATE:` 用户正在屏幕上移动两只手指

+   `GesturePhase.END:` 用户从屏幕上抬起一个或两个手指

让我们看看一个简单的例子，其中此菜谱中的`bubble`电影剪辑在手势阶段的开始时扩大尺寸，然后在手势阶段的结束时缩小回原始尺寸。在更新阶段，用户将能够像以前一样平移内容：

```swift
private function pan(e:TransformGestureEvent):void {
if(e.phase == GesturePhase.BEGIN)
{
bubble.scaleX += 0.2;
bubble.scaleY += 0.2;
}
else if(e.phase == GesturePhase.END)
{
bubble.scaleX -= 0.2;
bubble.scaleY -= 0.2;
}
else if(e.phase == GesturePhase.UPDATE)
{
bubble.x += e.offsetX;
bubble.y += e.offsetY;
}
}

```

修改您的代码，并添加对`flash.events.GesturePhase`的导入语句。将 IPA 部署到您的设备上，尝试平移屏幕内容，以感受手势的各个阶段发生时的感觉。

### 注意

您可以将此菜谱中的代码带入下一个菜谱。如果您已经做了上述更改，那么在继续之前请删除它们，因为它们不是必需的。

抓取和双指轻触手势都没有多个阶段。当使用这两种手势之一时，`phase`属性将始终返回`GesturePhase.ALL`。

### 单指平移

`GESTURE_PAN`事件仅检测使用两只手指发起和控制的平移。在 iOS 上预期的单指平移可以通过使用`flash.display.Sprite`提供的`startTouchDrag()`和`stopTouchDrag()`方法来实现。有关详细信息，请参阅本章前面的*拖动多个显示对象*菜谱。

## 参考内容

+   *设置手势输入模式*

+   *拖动多个显示对象*

+   *旋转对象*

# 旋转对象

旋转是 iOS 应用中另一种流行的手势。它通过在您的设备屏幕上放置两只手指，并围绕另一个接触点旋转一个接触点，或者围绕彼此旋转两个接触点来执行。为了允许内容的旋转，您可以监听并响应`TransformGestureEvent.GESTURE_ROTATE`事件。

我们将从*平移对象*菜谱继续，并添加旋转屏幕内容的功能。

## 准备工作

如果您还没有这样做，请在继续之前完成*平移对象*菜谱。

您可以继续使用在该菜谱中编写的代码。或者，从书籍的配套代码包中打开`chapter5\recipe8\recipe.fla`，并从那里开始工作。您也可以在同一个文件夹中找到 FLA 的文档类。

## 如何操作...

在 FLA 的文档类中执行以下步骤：

1.  在构造函数中，监听`TransformGestureEvent.GESTURE_ROTATE:`

    ```swift
    public function Main() {
    Multitouch.inputMode = MultitouchInputMode.GESTURE;
    bubble.addEventListener(TransformGestureEvent.GESTURE_PAN, pan);
    bubble.addEventListener(TransformGestureEvent.GESTURE_ROTATE, rotate); 
    }

    ```

1.  添加一个响应事件的处理器：

    ```swift
    private function rotate(e:TransformGestureEvent):void {
    bubble.rotation += e.rotation;
    }

    ```

1.  保存您的更改。

1.  发布 FLA 并在你的设备上测试它。

你现在应该能够旋转屏幕上的内容，同时也能够平移它。

### 注意

如果你正在较旧的设备上进行测试，可能会发现应用程序性能不佳，帧率远低于目标。如果你使用 Flash Professional CS5 进行开发，这一点尤其正确。不用担心；有方法可以显著提高性能，我们将在第六章*图形和硬件加速*中介绍。

## 它是如何工作的...

仅需要几个简单的步骤即可添加旋转支持。

首先，向`bubble`电影剪辑中添加了对`TransformGestureEvent.GESTURE_ROTATE`事件的监听器：

```swift
bubble.addEventListener(TransformGestureEvent.GESTURE_ROTATE,
rotate);

```

最后，实际的事件处理程序查询了事件的`rotation`属性，以确定如何旋转电影剪辑：

```swift
private function rotate(e:TransformGestureEvent):void {
bubble.rotation += e.rotation; 
}

```

`rotation`属性指定了自上次手势事件以来的旋转变化，以度为单位。然后，将此值添加到气泡的当前旋转角度，以正确更新它。

## 相关内容

+   *设置手势输入模式*

+   *缩放对象*

+   *使用缓存为位图矩阵，第六章*

# 缩放对象

本章我们将探讨的最后一个手势是“缩放”，它允许用户缩放内容。通常是通过用两个手指捏合屏幕来执行。将手指靠近会缩小视图，而将手指分开则会放大视图。Flash 平台提供了`TransformGestureEvent.GESTURE_ZOOM`事件，你可以监听并响应它。

## 准备工作

如果你还没有这样做，请在执行此操作之前完成*旋转对象*配方。

你可以继续使用在那个配方中编写的代码。或者，从书中附带代码包中的 FLA 和文档类开始工作，位于`chapter5\recipe9\`。

## 如何实现...

对 FLA 的文档类进行以下更改：

1.  在构造函数中，监听`GESTURE_ZOOM`事件：

    ```swift
    public function Main() {
    Multitouch.inputMode = MultitouchInputMode.GESTURE;
    bubble.addEventListener(TransformGestureEvent.GESTURE_PAN, pan);
    bubble.addEventListener(TransformGestureEvent.GESTURE_ROTATE, rotate);
    bubble.addEventListener(TransformGestureEvent.GESTURE_ZOOM, zoom); 
    }

    ```

1.  现在，为缩放手势添加事件处理程序：

    ```swift
    private function zoom(e:TransformGestureEvent):void {
    bubble.scaleX *= e.scaleX;
    bubble.scaleY *= e.scaleY;
    }

    ```

1.  保存类文件。

1.  发布 FLA 并将 IPA 部署到你的设备上。

你现在应该能够使用 Flash 平台提供的手势来平移、旋转和缩放气泡。

### 注意

如果你正在较旧的设备上进行测试，可能会发现应用程序性能不佳，帧率远低于目标。如果你使用 Flash Professional CS5 进行开发，这一点尤其正确。不用担心；有方法可以显著提高性能，我们将在第六章*图形和硬件加速*中介绍。

## 它是如何工作的...

与本章中介绍的其他手势一样，实现捏合缩放并不困难。

为`TransformGestureEvent.GESTURE_ZOOM`添加了一个事件监听器，随后是事件处理器，该处理器负责实际放大`bubble`电影剪辑。

以下是用以注册监听器的代码行：

```swift
bubble.addEventListener(TransformGestureEvent.GESTURE_ZOOM, zoom);

```

您可以在`zoom()`事件处理器中看到气泡的`scaleX`和`scaleY`属性被设置如下：

```swift
private function zoom(e:TransformGestureEvent):void {
bubble.scaleX *= e.scaleX;
bubble.scaleY *= e.scaleY;
}

```

`TransformGestureEvent`对象的`scaleX`和`scaleY`属性指定了自上次手势事件以来的水平和垂直缩放变化。要将这些值应用到显示对象上，只需将显示对象的`scaleX`和`scaleY`属性分别与事件对象的`scaleX`和`scaleY`属性相乘。您可以在之前的代码中看到这一操作。

这可能不是立即显而易见的，但气泡的库符号其注册点设置在其中心。在缩放内容时，确保您容器剪辑内的艺术品居中，否则当用户执行缩放手势时可能会遇到意外的行为。

## 参见

+   *设置手势输入模式*

+   *使用缓存作为位图矩阵，第六章*
