# 第十章 创建 HUD

**抬头显示（HUD**）是游戏设计师用来向用户提供信息的关键技术。它基本上是一组与世界本身分离的图形或文本，用于告诉玩家他们剩余的生命值或*魔法值*，或他们的得分是多少。

我们将创建一个简单的 HUD，它将提供非常有用的信息，例如角色移动的方向和其在世界坐标中的位置。这将足以说明如何创建一个 HUD。你可能会想在你的游戏中显示更多有用的信息。那么，让我们开始吧！

# 基础知识

创建一个名为`hud.lua`的文件，从`game.lua`中调用它，然后开始我们的模块。

我们的 HUD 实现将基于我们不希望将不在世界内的对象定位在世界坐标上的想法。此外，我们更习惯于在设计 UI 布局时考虑(0, 0)位于左上角，`Y`轴向下增长。

所以这就是我们要做的：

```swift
module ( "HUD", package.seeall )
function HUD:initialize ()
    self.viewport = MOAIViewport.new ()
```

可以这样做：

1.  为了使用不同的坐标系，我们将创建一个新的视口。

    ```swift
        viewport:setSize ( SCREEN_RESOLUTION_X, SCREEN_RESOLUTION_Y )
    ```

1.  这个视口将与屏幕大小相同：

    ```swift
       viewport:setScale  (SCREEN_RESOLUTION_X, -SCREEN_RESOLUTION_Y )
    ```

1.  它不会使用世界坐标。相反，它将使用屏幕坐标，但我们将反转`Y`轴，使其向下增长。

    ```swift
        viewport:setOffset ( -1, 1 )
    ```

1.  现在有一些新内容。`setOffset`方法用于使用投影空间移动视口。投影空间是一个 2 x 2 的矩形，其`Y`轴向上。将`-1`作为`X`值传递给`setOffset`将此投影空间向左移动半个屏幕，将`1`作为`Y`值传递将移动它半个屏幕到顶部，达到我们的目标，即(0, 0)坐标位于左上角。

    ### 小贴士

    你可以看到这里非常实用；你可以将(0, 0)移动到任何你想要的位置，位置将基于此进行计算。

1.  在此之后，我们创建一个层并将其加载到渲染表中，就像我们习惯的那样：

    ```swift
        self.layer = MOAILayer2D.new ()
        self.layer:setViewport ( self.viewport )
        -- Now we need to render the layer.
        local renderTable = MOAIRenderMgr.getRenderTable ()
        table.insert ( renderTable, self.layer )
        MOAIRenderMgr.setRenderTable ( renderTable )
    end
    ```

1.  在`Game:initialize`的底部添加对`HUD:initialize ()`的调用，你应该就可以开始了。

现在，我们准备好开始创建我们的`HUD`元素。

# 左边还是右边，这是个问题

现在我们将在屏幕上显示我们的信息：

1.  首先，我们需要在`game.lua`文件中的`resource_definitions`表中定义一个字体。

    我们为这一章提供了源代码，但你可以使用你喜欢的任何字体：

    ```swift
        hudFont ={
            type = RESOURCE_TYPE_FONT,
            fileName = 'fonts/tuffy.ttf',
        glyphs = 
        "abcdefghijklmnopqrstuvwxyzABCDEFGHI
        JKLMNOPQRSTUVWXYZ0123456789,.?!",
        fontSize = 26,
        dpi = 160
      }
    ```

    这应该很熟悉，因为我们已经在第六章中讨论过，*资源管理器*。

    现在我们知道`hudFont`将引用我们的字体。

1.  让我们回到`hud.lua`，创建一个名为`initializeDebugHud`的方法，并在`HUD:initialize`中调用它：

    ```swift
    function HUD:initializeDebugHud ()
        self.font = MOAIFont.new ()
        self.font = ResourceManager:get ( "hudFont" )
    ```

    我们使用我们刚刚创建的字体资源作为文本框的字体：

    ```swift
        self.leftRightIndicator = self:newDebugTextBox ( 30, {10, 10, 100, 50} )
        self.positionIndicator = self:newDebugTextBox ( 30, {10, 50, 200, 100} )
    end
    ```

1.  然后我们调用一个辅助方法，我们将在稍后创建它。它将接收字体大小和文本将放置的矩形。矩形由盒子的左上角和右下角的坐标组成：

    1.  首先，我们创建 `MOAITextBox`。这是将用于显示文本的类。它继承自 `MOAIProp`，因此你可以移动它、将其插入到层中，以及执行所有其他可以与 `MOAIProp` 做的事情：

        ```swift
        function HUD:newDebugTextBox ( size, rectangle )
            local textBox = MOAITextBox.new ()
        ```

    1.  我们设置了之前加载的字体：

        ```swift
            textBox:setFont ( self.font )
        ```

    1.  然后我们使用 `size` 参数设置大小：

        ```swift
            textBox:setTextSize ( size )
        ```

        ### 提示

        你可以在 `MOAITextBox` 中使用多种文本样式；请参阅文档中的 `setStyle` 和 `MOAITextStyle`。

    1.  我们使用 `unpack` 将 `rectangle` 表格拆分为参数：

        ```swift
            textBox:setRect ( unpack(rectangle) )
        ```

    1.  矩形定义了一个屏幕上的框，文本被限制在这个框内。文本不会渲染在 `setRect` 定义的矩形之外。

    1.  我们将文本框插入到 `HUD` 的层中。

        ```swift
            layer:insertProp ( textBox )
        ```

    1.  最后返回文本框，以便稍后可以引用。

        ```swift
            return textBox
        end
        ```

现在我们已经将 `HUD` 层填充了文本框，但为了完成我们的目标，我们还需要一件事情。

# 更新信息

我们需要在那些文本框中写入一些内容。在这种情况下，我们将定期在 `HUD` 中调用一个 `update` 方法，以便它可以刷新数据并在屏幕上打印：

```swift
function HUD:update ()
    local x, y = Character.prop:getScl ()
```

1.  我们获取用于设置角色方向的缩放比例，并将其存储在局部变量 `x` 中：

    ```swift
        if x > 0 then
            self.leftRightIndicator:setString ( "Left" )
        else
            self.leftRightIndicator:setString ( "Right" )
        end
    ```

1.  如果你记得我们之前做了什么，这应该很清楚。为了向右转，我们在 `x` 轴上将角色的 `prop` 缩放为 `-1`，要向左转则将其缩放为 `1`。所以这就是我们在这里所做的事情。如果缩放的 `x` 值为正，那么我们面向左边。否则，如果它是 `-1`，那么我们面向右边。然后，根据情况，我们使用 `MOAITextBox:setString` 来更新带有正确方向的字符串。

    ```swift
        x, y = Character.physics.body:getPosition ()
        self.positionIndicator:setString ("(" .. 
        math.floor(x) .. ", " .. 
        math.floor(y) .. ")")
    end
    ```

1.  最后我们更新文本框；我们从物理体中获取位置。使用 `..` 操作符连接字符串，生成格式为 `(x, y)` 的正确字符串。然后我们使用 `math.floor` 向下取整位置，因为它是一个小数。

现在我们需要做的唯一一件事是在 `Game:start` 中的 `while` 循环中添加对 `HUD:update ()` 的调用，我们应该能在屏幕上看到两个调试字符串。

### 提示

如果你不确定框的大小和布局，可以使用 `MOAIDebugLines.setStyle(MOAIDebugLines.TEXT_BOX)` 来调试它们。这将围绕每个文本框显示一条线。

# 摘要

在本章中，我们学习了如何使用 Moai SDK 实现一个 `HUD` 的基础知识。我们深入研究了 `MOAIViewport` 的配置，以便创建一个使用与之前不同的坐标系统的特殊视口。我们还首次使用 `MOAITextBox` 显示文本。

在下一章中，我们将深入探讨游戏的重要方面之一：声音和音乐。
