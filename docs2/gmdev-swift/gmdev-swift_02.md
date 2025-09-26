# 第二章：精灵、相机、动作！

使用 SpriteKit 绘制非常简单。我们可以自由地专注于构建出色的游戏体验，同时 SpriteKit 执行游戏循环的机械工作。要在屏幕上绘制一个项目，我们创建一个 SpriteKit 节点的新的实例。这些节点很简单；我们为每个要绘制的项目将子节点附加到场景或现有节点上。精灵、粒子发射器和文本标签在 SpriteKit 中都被视为节点。

### 注意

游戏循环是一个常用的游戏设计模式，用于每秒多次更新游戏，并在硬件快或慢的情况下保持相同的游戏速度。

SpriteKit 会自动将新节点连接到游戏循环中。随着你对 SpriteKit 的熟练，你可能希望进一步探索游戏循环以了解“幕后”发生了什么。

本章包括以下主题：

+   准备你的项目

+   绘制你的第一个精灵

+   动画：移动、缩放和旋转

+   与纹理一起工作

+   将艺术作品组织到纹理图集中

+   在精灵上居中相机

# 磨尖我们的铅笔

在我们开始绘制之前，有四个快速事项需要注意：

1.  由于我们将设计我们的游戏以使用横幅屏幕方向，我们将完全禁用纵向视图：

    1.  在 Xcode 中打开你的游戏项目后，在项目导航器中选择整体项目文件夹（最顶部的项目）。

    1.  你将在 Xcode 的主框架中看到你的项目设置。在**部署信息**下，找到**设备方向**部分。

    1.  取消选择**纵向**选项，如图所示：![磨尖我们的铅笔](img/Image_B04532_02_01.jpg)

1.  SpriteKit 模板为在场景中排列精灵生成一个视觉布局文件。我们不需要它；在探索关卡设计时，我们将使用 SpriteKit 视觉编辑器。要删除这个额外的文件：

    1.  在项目导航器中右键单击 `GameScene.sks` 并选择**删除**。

    1.  在对话框窗口中选择**移动到废纸篓**。

1.  我们需要调整场景大小以适应新的横幅视图。按照以下步骤调整场景大小：

    1.  从项目导航器打开 `GameViewController.swift` 并定位到 `GameViewController` 类中的 `viewDidLoad` 函数。`viewDidLoad` 函数将在游戏意识到它处于横幅视图之前触发，因此我们需要使用在启动过程中较晚触发的函数。完全删除 `viewDidLoad`，移除其所有代码。

    1.  将 `viewDidLoad` 替换为名为 `viewWillLayoutSubviews` 的新函数。现在不必担心理解每一行；我们只是在配置项目。为 `viewWillLayoutSubviews` 使用以下代码：

        ```swift
        override func viewWillLayoutSubviews() {
            super.viewWillLayoutSubviews()
            // Create our scene:
            let scene = GameScene()
            // Configure the view:
            let skView = self.view as! SKView
            skView.showsFPS = true
            skView.showsNodeCount = true
            skView.ignoresSiblingOrder = true
            scene.scaleMode = .AspectFill
            // size our scene to fit the view exactly:
            scene.size = view.bounds.size
            // Show the new scene:
            skView.presentScene(scene)
        }
        ```

    1.  最后，在 `GameViewController.swift` 中找到 `supportedInterfaceOrientations` 函数并将其缩减到以下代码：

        ```swift
        override func supportedInterfaceOrientations() -> Int {
            return Int(
            UIInterfaceOrientationMask.Landscape.rawValue);
        }
        ```

        ### 小贴士

        **下载示例代码**

        你可以从你购买的所有 Packt 出版物书籍的账户中下载示例代码文件。[`www.packtpub.com`](http://www.packtpub.com)。如果你在其他地方购买了这本书，你可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)并注册，以便将文件直接通过电子邮件发送给你。

        此外，每个章节还提供了检查点链接，你可以使用这些链接下载到该点的示例项目。

1.  我们应该再次确认我们已经准备好继续前进。尝试使用工具栏上的播放按钮或*command* + *r*键盘快捷键在模拟器中运行我们的清洁项目。加载后，模拟器应该切换到横幅视图，背景为空白灰色（并在右下角显示节点和 FPS 计数器）。如果项目无法运行，或者你仍然看到“**Hello World**”，你需要从第一章的结尾，*使用 Swift 设计游戏*，重新追踪你的步骤，以完成你的项目准备。

# 检查点 2- A

如果你想要下载到这一点的我的项目，你可以从以下网址下载：[`www.thinkingswiftly.com/game-development-with-swift/chapter-2`](http://www.thinkingswiftly.com/game-development-with-swift/chapter-2)

# 绘制你的第一个精灵

是时候编写一些游戏代码了——太棒了！打开你的 `GameScene.swift` 文件，找到 `didMoveToView` 函数。回想一下，这个函数每次游戏切换到这个场景时都会触发。我们将使用这个函数来熟悉 `SKSpriteNode` 类。你将在游戏中广泛使用 `SKSpriteNode`，无论何时你想添加一个新的 2D 图形实体。

### 注意

“精灵”一词指的是在屏幕上独立于背景移动的 2D 图形或动画。随着时间的推移，这个术语已经发展到指代 2D 游戏中屏幕上的任何游戏对象。我们将在本章中创建并绘制你的第一个精灵：一只快乐的小蜜蜂。

## 构建 SKSpriteNode 类

让我们先在屏幕上画一个蓝色方块。`SKSpriteNode` 类可以绘制纹理图形和实色块。在花费时间在艺术品上之前，用色块原型化你的新游戏想法通常很有帮助。要绘制蓝色方块，向游戏中添加一个 `SKSpriteNode` 实例：

```swift
override func didMoveToView(view: SKView) {
    // Instantiate a constant, mySprite, instance of SKSpriteNode
    // The SKSpriteNode constructor can set color and size
    // Note: UIColor is a UIKit class with built-in color presets
    // Note: CGSize is a type we use to set node sizes
    let mySprite = SKSpriteNode(color: UIColor.blueColor(), size: 
        CGSize(width: 50, height: 50))

    // Assign our sprite a position in points, relative to its 
    // parent node (in this case, the scene)
    mySprite.position = CGPoint(x: 300, y: 300)

    // Finally, we need to add our sprite node into the node tree.
    // Call the SKScene's addChild function to add the node
    // Note: In Swift, 'self' is an automatic property
    // on any type instance, exactly equal to the instance itself
    // So in this instance, it refers to the GameScene instance
    self.addChild(mySprite)
}
```

好吧，运行项目。你应该在模拟器中看到一个类似的小蓝色方块出现：

![构建 SKSpriteNode 类](img/Image_B04532_02_02.jpg)

### 小贴士

Swift 允许你将变量定义为常量，它只能被赋予一次值。为了最佳性能，尽可能使用 `let` 来声明常量。当你需要在代码中稍后更改值时，使用 `var` 声明变量。

## 将动画添加到你的工具包中

在我们深入精灵理论之前，我们应该用我们的蓝色正方形玩得开心一些。SpriteKit 使用动作对象在屏幕上移动精灵。考虑以下示例：如果我们的目标是移动正方形穿过屏幕，我们必须首先创建一个新的动作对象来描述动画。然后，我们指示我们的精灵节点执行该动作。我将在本章中用许多示例来说明这个概念。现在，在`didMoveToView`函数中，在`self.addChild(mySprite)`行下方添加以下代码：

```swift
// Create a new constant for our action instance
// Use the moveTo action to provide a goal position for a node
// SpriteKit will tween to the new position over the course of the
// duration, in this case 5 seconds
let demoAction = SKAction.moveTo(CGPoint(x: 100, y: 100), 
    duration: 5)
// Tell our square node to execute the action!
mySprite.runAction(demoAction)
```

运行项目。你会看到我们的蓝色正方形滑过屏幕，向（100,100）位置移动。这个动作是可重用的；场景中的任何节点都可以执行这个动作来移动到（100,100）位置。正如你所见，当我们需要对节点属性进行动画处理时，SpriteKit 为我们做了很多繁重的工作。

### 小贴士

中间画，或称补间，使用引擎在起始帧和结束帧之间进行平滑动画。我们的`moveTo`动画是一个补间；我们提供起始帧（精灵的原始位置）和结束帧（新的目标位置）。SpriteKit 生成我们值之间的平滑过渡。

让我们尝试一些其他动作。`SKAction.moveTo`函数只是众多选项之一。尝试将`demoAction`行替换为以下代码：

```swift
let demoAction = SKAction.scaleTo(4, duration: 5)
```

运行项目。你会看到我们的蓝色正方形增长到原来的四倍大小。

### 多个动画的序列化

我们可以使用动作组和序列同时执行动作或依次执行。例如，我们可以轻松地将我们的精灵放大并旋转。删除到目前为止的所有动作代码，并用以下代码替换：

```swift
// Scale up to 4x initial scale
let demoAction1 = SKAction.scaleTo(4, duration: 5)
// Rotate 5 radians
let demoAction2 = SKAction.rotateByAngle(5, duration: 5)
// Group the actions
let actionGroup = SKAction.group([demoAction1, demoAction2])
// Execute the group!
mySprite.runAction(actionGroup)
```

当你运行项目时，你会看到一个旋转并变大的正方形。太棒了！如果你想按顺序运行这些动作（而不是同时运行），将`SKAction.group`更改为`SKAction.sequence`：

```swift
// Group the actions into a sequence
let actionSequence = SKAction.sequence([demoAction1, demoAction2])

// Execute the sequence!
mySprite.runAction(actionSequence)
```

运行代码，观察你的正方形首先变大然后旋转。很好。你不仅限于两个动作；我们可以将所需数量的动作组合或序列化。

我们到目前为止只使用了几个动作；在继续之前，你可以自由探索`SKAction`类并尝试不同的动作组合。

## 回顾你的第一个精灵

恭喜你，你已经学会了如何使用 SpriteKit 动作绘制非纹理精灵并对其进行动画处理。接下来，我们将探索一些重要的定位概念，然后为我们的精灵添加游戏艺术。在你继续之前，请确保你的`didMoveToView`函数与我的匹配，并且你的序列化动画正在正确触发。以下是到目前为止的我的代码：

```swift
override func didMoveToView(view: SKView) {
    // Instantiate a constant, mySprite, instance of SKSpriteNode
    let mySprite = SKSpriteNode(color: UIColor.blueColor(), size: 
        CGSize(width: 50, height: 50))

    // Assign our sprite a position
    mySprite.position = CGPoint(x: 300, y: 300)

    // Add our sprite node into the node tree
    self.addChild(mySprite)

    // Scale up to 4x initial scale
    let demoAction1 = SKAction.scaleTo(CGFloat(4), duration: 2)
    // Rotate 5 radians
    let demoAction2 = SKAction.rotateByAngle(5, duration: 2)

    // Group the actions into a sequence
    let actionSequence = SKAction.sequence([demoAction1, 
        demoAction2])

    // Execute the sequence!
    mySprite.runAction(actionSequence)
}
```

# 定位的故事

SpriteKit 使用点阵来定位节点。在这个网格中，场景的左下角是（0,0），X 轴向右为正方向，Y 轴向上为正方向。

类似地，在单个精灵级别上，（0,0）指的是精灵的左下角，而（1,1）指的是右上角。

## 与锚点对齐

每个精灵都有一个`anchorPoint`属性，或称为原点。`anchorPoint`属性允许您选择精灵的哪个部分与精灵的整体位置对齐。

### 注意

默认锚点为（0.5，0.5），因此新的`SKSpriteNode`在其位置上完美居中。

为了说明这一点，让我们检查一下我们在屏幕上刚刚绘制的蓝色方块精灵。我们的精灵宽度为 50 像素，高度为 50 像素，其位置是（300，300）。由于我们没有修改`anchorPoint`属性，其锚点为（0.5，0.5）。这意味着精灵将在场景网格的（300，300）位置上完美居中。我们的精灵的左侧边缘始于 275，右侧边缘终止于 325。同样，底部始于 275，顶部终止于 325。以下图表说明了我们的方块在网格上的位置：

![使用锚点对齐](img/Image_B04532_02_03.jpg)

为什么我们默认喜欢居中的精灵？您可能会认为通过将`anchorPoint`属性设置为（0，0）来根据元素的左下角定位元素会更简单。然而，当我们在缩放或旋转精灵时，居中行为对我们更有益：

+   当我们使用`anchorPoint`属性为（0，0）缩放精灵时，它只会沿着 y 轴向上扩展并沿 x 轴向外出扩展。旋转动作会使精灵围绕其左下角进行大圆旋转。

+   默认的`anchorPoint`属性为（0.5，0.5）的居中精灵在缩放时会在所有方向上等比例扩展，并且在旋转时会在原地旋转，这通常是期望的效果。

有时候您可能想要更改锚点。例如，如果您在绘制火箭船，您可能希望船围绕其圆锥形的前端旋转，而不是围绕其中心。

# 添加纹理和游戏艺术

您可能想为您的蓝色方块拍一张截图，以备将来欣赏。我非常喜欢回忆我完成的游戏的老截图，当时它们只是简单的彩色方块在屏幕上滑动。现在是我们超越这个阶段，并将一些有趣的艺术作品附加到我们的精灵上的时候了。

## 下载免费资源

我为这本书中使用的所有艺术资源提供了一个可下载的包。我建议您使用这些资源，这样您将为我们的演示游戏准备齐全。或者，如果您愿意，当然可以自由地为您的游戏创建自己的艺术作品。

这些资源来自 Kenney Game Studio 的一个杰出的公共领域资源包。我提供的是我们将用于游戏的资源包的小子集。请从以下 URL 下载游戏艺术资源：

[`www.thinkingswiftly.com/game-development-with-swift/assets`](http://www.thinkingswiftly.com/game-development-with-swift/assets)

### 更出色的艺术作品

如果你喜欢这些艺术作品，你可以在[`kenney.itch.io/kenney-donation`](http://kenney.itch.io/kenney-donation)通过小额捐赠下载超过 16,000 个同风格的游戏资源。我与 Kenney 没有关联；我只是觉得他向独立游戏开发者发布了如此多的公共领域艺术作品令人钦佩。

作为 CC0 资源，你可以复制、修改和分发这些艺术作品，甚至用于商业目的，而无需请求许可。你可以在这里阅读完整的许可证：

[`creativecommons.org/publicdomain/zero/1.0/`](https://creativecommons.org/publicdomain/zero/1.0/)

## 绘制你的第一个纹理精灵

让我们使用你刚刚下载的一些图形。我们将从创建一个蜜蜂精灵开始。我们将把蜜蜂纹理添加到我们的项目中，将图像加载到`SKSpriteNode`类中，然后调整节点大小以在视网膜屏幕上获得最佳清晰度。

### 将蜜蜂图像添加到你的项目中

在我们能够在游戏中使用它们之前，我们需要将图像文件添加到我们的 Xcode 项目中。一旦添加了图像，我们就可以在代码中通过名称引用它们；SpriteKit 足够智能，能够找到并实现图形。按照以下步骤将蜜蜂图像添加到项目中：

1.  在项目导航器中右键单击你的项目，然后点击**将文件添加到“Pierre Penguin Escapes the Antarctic”**（或你的游戏名称）。参考此截图以找到正确的菜单项：![将蜜蜂图像添加到你的项目中](img/Image_B04532_02_04.jpg)

1.  浏览你下载的资产包，并在`Enemies`文件夹中找到`bee.png`图像。

1.  选择**如果需要则复制项目**，然后点击**添加**。

你现在应该在项目导航器中看到`bee.png`。

### 使用 SKSpriteNode 加载图像

使用`SKSpriteNode`将图像绘制到屏幕上相当简单。首先，清除我们在`GameScene.swift`中的`didMoveToView`函数内编写的所有用于蓝色方块的代码。将`didMoveToView`替换为以下代码：

```swift
override func didMoveToView(view: SKView) {
    // set the scene's background to a nice sky blue
    // Note: UIColor uses a scale from 0 to 1 for its colors
    self.backgroundColor = UIColor(red: 0.4, green: 0.6, blue: 
        0.95, alpha: 1.0);

    // create our bee sprite node
    let bee = SKSpriteNode(imageNamed: "bee.png")
    // size our bee node
    bee.size = CGSize(width: 100, height: 100)
    // position our bee node
    bee.position = CGPoint(x: 250, y: 250)
    // attach our bee to the scene's node tree
    self.addChild(bee)
}
```

运行项目并见证我们辉煌的蜜蜂——干得好！

![使用 SKSpriteNode 加载图像](img/Image_B04532_02_05.jpg)

### 为视网膜设计

你可能会注意到我们的蜜蜂图像相当模糊。为了利用视网膜屏幕，资源需要是其节点大小属性的两倍像素维度（对于大多数视网膜屏幕），或者 iPhone 6 Plus 的节点大小的三倍。暂时忽略高度；我们的蜜蜂节点宽度为 100 点，但 PNG 文件只有 56 像素宽。PNG 文件需要宽度为 300 像素才能在 iPhone 6 Plus 上看起来清晰，或者在 2x 视网膜设备上看起来清晰需要宽度为 200 像素。

SpriteKit 会自动调整纹理大小以适应其节点，因此一种方法是在最高的视网膜分辨率（节点大小的三倍）创建一个巨大的纹理，然后让 SpriteKit 将其调整到较低密度屏幕。然而，这会带来相当大的性能损失，并且旧设备甚至可能因为巨大的纹理而耗尽内存并崩溃。

#### 理想资产方法

这些双倍和三倍大小的视网膜资源可能会让新的 iOS 开发者感到困惑。为了解决这个问题，Xcode 通常允许你为每个纹理提供三个图像文件。例如，我们的蜜蜂节点目前宽度为 100 点，高度为 100 点。在一个完美的世界里，你会向 Xcode 提供以下图像：

+   `Bee.png` (100 像素 x 100 像素)

+   `Bee@2x.png` (200 像素 x 200 像素)

+   `Bee@3x.png` (300 像素 x 300 像素)

然而，目前有一个问题阻止 3x 纹理与**纹理图集**正确工作。纹理图集将纹理组合在一起并显著提高渲染性能（我们将在下一节中实现我们的第一个纹理图集）。我希望 Apple 能在 Swift 2 中升级纹理图集以支持 3x 纹理。目前，我们需要在 iPhone 6 Plus 的纹理图集和 3x 资源之间做出选择。

#### 我目前的解决方案

在我看来，纹理图集及其性能优势是 SpriteKit 的关键特性。我将继续使用纹理图集，为 iPhone 6 Plus 提供 2x 图像（它仍然看起来相当清晰）。这意味着在这本书中我们不会使用任何 3x 资源。

进一步简化问题，Swift 只运行在 iOS7 及以上版本。唯一运行 iOS7 的非视网膜设备是老化的 iPad 2 和第一代 iPad mini。如果你的最终游戏需要这些旧设备，你应该为你的游戏创建标准图像和 2x 图像。否则，你可以安全地忽略 Swift 的非视网膜资源。

### 注意

这意味着在这本书中我们只会使用双倍大小的图像。在可下载的资源包中的图像放弃了 2x 后缀，因为我们只使用这个大小。一旦 Apple 更新纹理图集以使用 3x 资源，我建议你切换到*理想资源方法*部分中概述的方法来为你的游戏使用。

#### 在 SpriteKit 中使用视网膜显示

我们的蜜蜂图像说明了这一切是如何工作的：

+   由于我们设置了显式的节点大小，SpriteKit 会自动调整蜜蜂纹理的大小以适应我们 100 点宽、100 点高的节点。这种自动调整大小以适应的功能非常方便，但请注意，我们实际上略微扭曲了图像的宽高比。

+   如果我们不设置显式的大小，SpriteKit 会将节点（以点为单位）的大小调整为与纹理的维度（以像素为单位）相匹配。删除设置我们蜜蜂节点大小的行，并重新运行项目。SpriteKit 会自动保持宽高比，但较小的蜜蜂仍然模糊。这是因为我们的新节点是 56 点 x 48 点，与我们的 PNG 文件的 56 像素 x 48 像素像素维度相匹配……然而，我们的 PNG 文件需要是 112 像素 x 96 像素，才能在 2x 视网膜屏幕上以这个节点大小显示清晰图像。

+   我们无论如何都需要一个更小的蜜蜂，所以我们将调整节点的大小而不是生成更大的艺术品。将你的蜜蜂节点的`size`属性设置为纹理像素分辨率的二分之一：

    ```swift
    // size our bee in points:
    bee.size = CGSize(width: 28, height: 24)
    ```

运行项目，你会看到一个更小、更清晰的蜜蜂，就像这个截图所示：

![在 SpriteKit 中使用视网膜显示](img/Image_B04532_02_08.jpg)

太棒了！这里的重要概念是要将你的艺术文件设计成节点点大小的两倍像素分辨率，以便利用 2x 视网膜屏幕，或者将点大小增加到三倍以充分利用 iPhone 6 Plus。现在我们将看看如何组织和动画多个精灵帧。

# 组织你的资源

如果我们像处理蜜蜂一样添加所有纹理，我们的项目导航器很快就会被图像文件淹没。幸运的是，Xcode 提供了几个解决方案。

## 探索 Images.xcassets

我们可以将图像存储在`.xcassets`文件中，并轻松地从我们的代码中引用它们。这是一个存储背景图像的好地方：

1.  从项目导航器中打开`Images.xcassets`。

1.  目前我们不需要在这里添加任何图像，但将来，你可以直接将图像文件拖到图像列表中，或者右键单击，然后**导入**。

1.  注意，SpriteKit 演示中的飞船图像存储在这里。我们不再需要它，所以我们可以右键单击它，然后选择**删除所选项目**来删除它。

## 将艺术作品收集到纹理图集中

我们将使用纹理图集来组织大部分游戏中的艺术资源。纹理图集通过收集相关的艺术作品来组织资源。它们还通过将每个图集中的所有图像优化为单个纹理来提高性能。SpriteKit 只需要一个绘制调用就能从同一纹理图集中渲染多个图像。此外，它们非常容易使用！按照以下步骤构建你的蜜蜂纹理图集：

1.  我们需要移除旧的蜜蜂纹理。在项目导航器中右键单击`bee.png`，然后选择**删除**，然后**移动到废纸篓**。

1.  使用 Finder，浏览到你下载的资源包，并定位到`Enemies`文件夹。

1.  在`Enemies`内部创建一个新的文件夹，并将其命名为`bee.atlas`。

1.  在`Enemies`中找到`bee.png`和`bee_fly.png`图像，并将它们复制到你的新`bee.atlas`文件夹中。现在你应该有一个名为`bee.atlas`的文件夹，其中包含两个蜜蜂 PNG 文件。创建新的纹理图集你所需要做的就是将相关的图像放置到一个带有`.atlas`后缀的新文件夹中。

1.  将图集添加到你的项目中。在 Xcode 中，在项目导航器中右键单击项目文件夹，然后点击**添加文件…**，就像我们之前为单个蜜蜂纹理所做的那样。

1.  找到`bee.atlas`文件夹，并选择文件夹本身。

1.  选择**如果需要则复制项目**，然后点击**添加**。

纹理图集将出现在项目导航器中。做得好；我们将蜜蜂资源组织到一个集合中，Xcode 将自动创建之前提到的性能优化。

### 更新我们的蜜蜂节点以使用纹理图集

我们实际上现在可以运行我们的项目，看到之前相同的蜜蜂。我们旧的蜜蜂纹理是`bee.png`，而一个新的`bee.png`存在于纹理图集中。尽管我们删除了独立的`bee.png`，但 SpriteKit 足够智能，能够在纹理图集中找到新的`bee.png`。

我们应该确保我们的纹理图集正在正常工作，并且我们已经成功删除了旧的单独的`bee.png`。在`GameScene.swift`中，将我们的`SKSpriteNode`实例化行更改为使用纹理图集中的新`bee_fly.png`图形：

```swift
// create our bee sprite
// notice the new image name: bee_fly.png
let bee = SKSpriteNode(imageNamed: "bee_fly.png")
```

再次运行项目。你应该看到不同的蜜蜂图像，它的翅膀比之前更低。这是蜜蜂动画的第二帧。接下来，我们将学习如何在两个帧之间进行动画，以创建一个动画精灵。

### 遍历纹理图集帧

我们需要学习一种额外的纹理图集技术：我们可以快速翻阅多个精灵帧，让我们的蜜蜂通过动作变得生动起来。我们现在有两个蜜蜂在飞行中的帧；如果我们在这两个帧之间切换，它应该看起来像是悬停在原地。

我们的小结点将运行一个新的`SKAction`在两个帧之间进行动画。更新你的`didMoveToView`函数以匹配我的（我移除了一些旧的注释以节省空间）：

```swift
override func didMoveToView(view: SKView) {
    self.backgroundColor = UIColor(red: 0.4, green: 0.6, blue: 
        0.95, alpha: 1.0)

    // create our bee sprite
    // Note: Remove all prior arguments from this line:
    let bee = SKSpriteNode()
    bee.position = CGPoint(x: 250, y: 250)
    bee.size = CGSize(width: 28, height: 24)
    self.addChild(bee)

    // Find our new bee texture atlas
    let beeAtlas = SKTextureAtlas(named:"bee.atlas")
    // Grab the two bee frames from the texture atlas in an array
    // Note: Check out the syntax explicitly declaring beeFrames
    // as an array of SKTextures. This is not strictly necessary,
    // but it makes the intent of the code more readable, so I 
    // chose to include the explicit type declaration here:
    let beeFrames:[SKTexture] = [
        beeAtlas.textureNamed("bee.png"), 
        beeAtlas.textureNamed("bee_fly.png")]
    // Create a new SKAction to animate between the frames once
    let flyAction = SKAction.animateWithTextures(beeFrames, 
        timePerFrame: 0.14)
    // Create an SKAction to run the flyAction repeatedly
    let beeAction = SKAction.repeatActionForever(flyAction)
    // Instruct our bee to run the final repeat action:
    bee.runAction(beeAction)
}
```

运行项目。你会看到我们的蜜蜂翅膀一上一下地拍打——酷！你已经学会了使用纹理图集进行精灵动画的基础。我们将在本书的后面使用相同的技巧创建越来越复杂的动画。现在，给自己鼓掌。结果可能看起来很简单，但你已经解锁了通往你的第一个 SpriteKit 游戏的主要构建块！

# 将所有这些整合在一起

首先，我们学习了如何使用动作来移动、缩放和旋转我们的精灵。然后，我们探索了通过多个帧进行动画，让我们的精灵栩栩如生。现在，让我们将这些技术结合起来，让我们的蜜蜂在屏幕上飞来飞去，每次转弯时翻转纹理。

在`didMoveToView`函数的底部添加此代码，在`bee.runAction(beeAction)`行之下：

```swift
// Set up new actions to move our bee back and forth:
let pathLeft = SKAction.moveByX(-200, y: -10, duration: 2)
let pathRight = SKAction.moveByX(200, y: 10, duration: 2)
// These two scaleXTo actions flip the texture back and forth
// We will use these to turn the bee to face left and right
let flipTextureNegative = SKAction.scaleXTo(-1, duration: 0)
let flipTexturePositive = SKAction.scaleXTo(1, duration: 0)
// Combine actions into a cohesive flight sequence for our bee
let flightOfTheBee = SKAction.sequence([pathLeft, 
    flipTextureNegative, pathRight, flipTexturePositive])
// Last, create a looping action that will repeat forever
let neverEndingFlight = 
    SKAction.repeatActionForever(flightOfTheBee)

// Tell our bee to run the flight path, and away it goes!
bee.runAction(neverEndingFlight)
```

运行项目。你会看到蜜蜂在飞来飞去，拍打翅膀。你正式学会了 SpriteKit 中的动画基础！我们将在此基础上构建，为我们的玩家创建一个丰富的动画游戏世界。

# 将相机中心对准精灵

游戏通常需要相机跟随玩家精灵在空间中的移动。我们确实希望我们的企鹅角色皮埃尔有这种行为，我们很快就会将其添加到游戏中。由于 SpriteKit 没有内置相机功能，我们将创建自己的结构来模拟我们想要的效果。

我们实现这一目标的一种方法是将皮埃尔保持在同一位置，并将其他每个对象移动过他。这是有效的，但在语义上可能有些混乱，并且在定位游戏对象时可能会引起错误。

## 创建一个新世界

我更喜欢创建一个世界节点，并将所有我们的游戏节点附加到它上（而不是直接附加到场景）。我们可以通过世界将皮埃尔向前移动，并简单地重新定位世界节点，以便皮埃尔始终位于我们设备视口的中心。所有我们的敌人、道具和结构都将作为世界节点的子节点，并且在我们滚动世界时看起来像是在屏幕上移动。

### 小贴士

每个精灵节点的位置始终相对于其直接父节点。当您更改节点的位置时，所有子节点都会随之移动。这对于模拟我们的相机来说是一个非常方便的行为。

此图展示了该技术的简化版本，并使用了一些虚构的数字：

![创建新世界](img/Image_B04532_02_06.jpg)

您可以在以下代码块中找到我们相机功能的代码。阅读注释以获取详细说明。这只是一个快速回顾更改：

+   我们的`didMoveToView`函数变得越来越拥挤。我将我们的飞行蜜蜂代码拆分到一个名为`addTheFlyingBee`的新函数中。稍后，我们将游戏对象，如蜜蜂，封装到它们自己的类中。

+   我在`GameScene`类中创建了两个新的常量：世界节点和蜜蜂节点。

+   我更新了`didMoveToView`函数。它将世界节点添加到场景的节点树中，并调用新的`addTheFlyingBee`函数。

+   在新的蜜蜂函数内部，我移除了蜜蜂常量，因为`GameScene`现在将其声明为其自己的属性。

+   在新的蜜蜂函数内部，我们不是通过`self.addChild(bee)`将蜜蜂节点添加到场景中，而是想通过`world.addChild(bee)`将其添加到世界中。

+   我们正在实现一个新的函数：`didSimulatePhysics`。SpriteKit 在执行物理计算和调整位置后，每帧都会调用此函数。这是一个更新我们世界位置的好地方。更改世界位置的数学计算位于这个新函数中。

请更新您的整个`GameScene.swift`文件以匹配我的：

```swift
import SpriteKit

class GameScene: SKScene {
    // Create the world as a generic SKNode
    let world = SKNode()
    // Create our bee node as a property of GameScene so we can 
    // access it throughout the class
    // (Make sure to remove the old bee declaration inside the 
    // didMoveToView function.)
    let bee = SKSpriteNode()

    override func didMoveToView(view: SKView) {
        self.backgroundColor = UIColor(red: 0.4, green: 0.6, blue: 
            0.95, alpha: 1.0)

        // Add the world node as a child of the scene
        self.addChild(world)
        // Call the new bee function
        self.addTheFlyingBee()
    }

    // I moved all of our bee animation code into a new function:
    func addTheFlyingBee() {
        // Position our bee
        bee.position = CGPoint(x: 250, y: 250)
        bee.size = CGSize(width: 28, height: 24)
        // Notice we now attach our bee node to the world node:
        world.addChild(bee)

        /*
            all of the same bee animation code remains here,
            I am excluding it in this text for brevity
        */
    }

    // A new function
    override func didSimulatePhysics() {
        // To find the correct position, subtract half of the   
        // scene size from the bee's position, adjusted for any  
        // world scaling.
        // Multiply by -1 and you have the adjustment to keep our 
        // sprite centered:
        let worldXPos = -(bee.position.x * world.xScale - 
            (self.size.width / 2))
        let worldYPos = -(bee.position.y * world.yScale - 
            (self.size.height / 2))
        // Move the world so that the bee is centered in the scene
        world.position = CGPoint(x: worldXPos, y: worldYPos)
    }

}
```

运行游戏。您应该看到我们的蜜蜂直接固定在屏幕中心，每两秒翻转一次。

![创建新世界](img/Image_B04532_02_07.jpg)

蜜蜂实际上正在改变位置，就像之前一样，但世界正在补偿以保持蜜蜂在屏幕中心。当我们第三章中添加更多游戏对象时，*加入物理*，蜜蜂看起来就像整个世界在屏幕上滚动时在飞行。

# 检查点 2-B

在本章中，我们对项目进行了许多更改。如果您想下载到这一点的项目，请在此处操作：

[Swift 游戏开发](http://www.thinkingswiftly.com/game-development-with-swift/chapter-2)

# 摘要

您已经获得了 SpriteKit 中精灵、节点和动作的基础知识，并且已经朝着用 Swift 制作您的第一个游戏迈出了巨大的步伐。

您已为项目配置了横幅方向，绘制了您的第一个精灵，然后让它移动、旋转和缩放。您为精灵添加了蜜蜂纹理，创建了一个图像图集，并通过飞行帧进行动画。最后，您构建了一个世界节点，以使游戏玩法始终围绕玩家进行。做得好！

在下一章中，我们将使用 SpriteKit 的物理引擎为我们的世界分配重量和重力，生成更多飞行角色，并创建地面和天空。
