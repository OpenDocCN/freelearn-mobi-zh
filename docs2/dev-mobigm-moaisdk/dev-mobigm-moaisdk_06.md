# 第六章 资源管理器

如果我们开始开发我们的游戏，我们最终需要创建一个处理所有我们资产的实体。我们将称之为`ResourceManager`。我们将看到如何创建一个允许你将图像、字体和声音添加到你的游戏中的实体。

资源管理器背后的主要思想是缓存我们将多次使用的资产，并有一个集中化和抽象化的方式来创建资产。

# 资源定义

为了能够定义资源，我们需要创建一个负责处理此任务的模块。主要思想是在通过`ResourceManager`调用某个资产之前，它必须在`ResourceDefinitions`中定义。这样，`ResourceManager`将始终能够访问创建资产所需的某些元数据（文件名、大小、音量等）。

为了识别资产类型（声音、图像、瓦片和字体），我们将定义一些常量（请注意，这些常量的数值是任意的；你可以在这里使用任何你想要的）。让我们称它们为`RESOURCE_TYPE_[type]`（如果你想使用其他约定，请随意）。为了使事情更简单，现在就遵循这个约定，因为这是我们将在本书的其余部分使用的约定。你应该在`main.lua`中按如下方式输入它们：

```swift
RESOURCE_TYPE_IMAGE = 0
RESOURCE_TYPE_TILED_IMAGE = 1
RESOURCE_TYPE_FONT = 2
RESOURCE_TYPE_SOUND = 3
```

如果你想要了解这些资源类型常数的实际原因，请查看下一节中我们的`ResourceManager`实体的`load`函数。

我们需要创建一个名为`resource_definitions.lua`的文件，并添加一些简单的处理方法。

向其中添加以下行：

```swift
module ( “ResourceDefinitions”, package.seeall )
```

前面的行表示文件中的所有代码都应该被视为一个`module`函数，通过代码中的`ResourceDefinitions`来访问。这是 Lua 创建模块的许多模式之一。

### 注意

如果你不熟悉 Lua 的`module`函数，你可以在模块教程中阅读有关它的内容，教程地址为[`lua-users.org/wiki/ModulesTutorial`](http://lua-users.org/wiki/ModulesTutorial)。

接下来，我们将创建一个包含这些定义的表：

```swift
local definitions = {}
```

这将在内部使用，并且无法通过模块 API 访问，因此我们使用关键字`local`来创建它。

现在，我们需要为定义创建`setter`、`getter`和`unload`方法。

`setter`方法（称为`set`）将`definition`参数（一个表）存储在`definitions`表中，使用`name`参数（一个字符串）作为键，如下所示：

```swift
function ResourceDefinitions:set(name, definition)
    definitions[name] = definition
end
```

`getter`方法（称为`get`，当然！）使用`name`参数作为`definitions`表的键，检索之前存储的定义（通过使用`ResourceDefinitions:set ()`），如下所示：

```swift
function ResourceDefinitions:get(name)
    return definitions[name]
end
```

我们正在创建的`final`方法是`remove`。我们用它来清除定义所使用的内存空间。为了实现这一点，我们将`nil`赋值给`definitions`表中由`name`参数索引的条目，如下所示：

```swift
function ResourceDefinitions:remove (name)
  definitions[name] = nil
end
```

这样做，我们消除了对对象的引用，允许垃圾收集器释放内存。这在这里可能看起来没有用，但它是一个很好的例子，说明了您应该如何管理对象以便让垃圾收集器从内存中移除。而且除此之外，我们不知道信息来自资源定义；它可能很大，我们只是不知道。

这就是资源定义所需的所有内容。我们正在利用 Lua 提供的动态性。看看创建一个从每个定义的内容中抽象出来的定义存储库有多容易。我们将为每种资产类型定义不同的字段，而且我们不需要事先定义它们，就像我们可能在 C++ 中需要做的那样。

# 资源管理器

我们现在将创建我们的资源管理器。这个模块将负责创建和存储我们的牌组和一般资源。我们将使用一个单一的命令来检索资源，它们将来自缓存或根据定义创建。

我们需要创建一个名为 `resource_manager.lua` 的文件，并将其添加以下行：

```swift
module ( “ResourceManager”, package.seeall )
```

这与资源定义中的情况相同；我们正在创建一个模块，它将通过 `ResourceManager` 来访问。

```swift
ASSETS_PATH = ‘assets/’
```

我们现在创建一个名为 `ASSETS_PATH` 的常量。这是我们将存储我们的资源的地方。您可以为不同类型的资源设置多个路径，但为了保持简单，在这个例子中我们将它们全部保存在一个单独的目录中。使用这个常量将允许我们只使用文件名，而不是在创建实际资源定义时必须写出整个路径，这样可以节省我们一些麻烦！

```swift
local cache = {}
```

再次，我们正在创建一个作为局部变量的 `cache` 表。这将是我们存储初始化资源的变量。

现在我们应该注意实现重要的功能。为了使代码更易读，我将在以下页面中定义的方法中使用方法。因此，我建议您在尝试运行我们现在编写的代码之前阅读整个部分。完整的源代码可以从本书的网站上下载，其中包含内联注释。在书中，为了简洁起见，我们移除了注释。

## 获取器

我们将首先实现我们的 `getter` 方法，因为它足够简单：

```swift
function ResourceManager:get ( name )
    if (not self:loaded ( name )) then
        self:load ( name )
    end

    return cache[name]
end
```

这个方法接收一个 `name` 参数，它是我们正在处理的资源的标识符。在第一行，我们调用 `loaded`（我们很快将定义的方法）来查看由 `name` 标识的资源是否已经被加载。如果是的话，我们只需要返回缓存的值，但如果没有，我们需要加载它，这就是我们在 `if` 语句中做的。我们使用内部的 `load` 方法（我们稍后也将定义）来处理加载。我们将使这个 `load` 方法将加载的对象存储在 `cache` 表中。所以加载后，我们唯一要做的就是返回 `cache` 表中按 `name` 索引的对象。

我们在这里使用的辅助函数之一是 `loaded`。让我们来实现它，因为它真的很容易做：

```swift
function ResourceManager:loaded ( name )
  return cache[name] ~= nil
end
```

我们在这里所做的是检查由 `name` 参数索引的 `cache` 表是否不等于 `nil`。如果 `cache` 在该键下有一个对象，这将返回 `true`，这正是我们想要找到的，以决定由 `name` 参数表示的对象是否已经被加载。

## 加载器

`load` 及其辅助函数是这个模块最重要的方法。它们将比我们迄今为止所做的方法稍微复杂一些，因为它们实现了魔法。请特别注意这一部分。它并不特别困难，但可能会让人困惑。像之前的方法一样，这个方法只接收代表我们要加载的资源的 `name` 参数，如下所示：

```swift
function ResourceManager:load ( name )
```

首先，我们检索与 `name` 关联的资源定义。我们调用之前定义的 `ResourceDefinitions` 中的 `get` 方法，如下所示：

```swift
    local resourceDefinition = ResourceDefinitions:get( name )
```

如果资源定义不存在（因为我们忘记在之前定义它），我们将打印一个错误信息，如下所示：

```swift
    if not resourceDefinition then
        print(“ERROR: Missing resource definition for “ .. name )
```

如果成功检索到资源定义，我们创建一个变量来保存资源，并且（请注意）根据资产类型调用正确的 `load` 辅助函数。

```swift
    else
        local resource
```

记得我们在 `ResourceDefinitions` 模块中创建的 `RESOURCE_TYPE_[type]` 常量吗？这就是它们存在的原因。多亏了 `RESOURCE_TYPE_[type]` 常量的创建，我们现在知道如何正确地加载资源。当我们定义一个资源时，我们必须包含一个 `type` 键，并使用其中一个资源类型。我们很快就会坚持这一点。我们现在所做的是调用正确的 `load` 方法，用于图像、平铺图像、字体和声音，使用存储在 `resourceDefinition.type` 中的值，如下所示：

```swift
    if (resourceDefinition.type == RESOURCE_TYPE_IMAGE) then
        resource = self:loadImage ( resourceDefinition )
    elseif (resourceDefinition.type == RESOURCE_TYPE_TILED_IMAGE) then
        resource = self:loadTiledImage ( resourceDefinition )
    elseif (resourceDefinition.type == RESOURCE_TYPE_FONT) then
        resource = self:loadFont ( resourceDefinition )
    elseif (resourceDefinition.type == RESOURCE_TYPE_SOUND) then
        resource = self:loadSound ( resourceDefinition )
    end
```

在加载当前资源后，我们将其存储在 `cache` 表中，使用 `name` 参数指定的条目，如下所示：

```swift
    -- store the resource under the name on cache
    cache[name] = resource
    end
end
```

现在，让我们看看所有不同的加载方法。实际的函数之前的预期定义是为了在阅读时提供参考。

### 图像

加载图像是我们已经做过的事情，所以这看起来会有些熟悉。

在这本书中，我们将有两种定义图像的方法。让我们来看看它们：

```swift
{
    type = RESOURCE_TYPE_IMAGE
    fileName = “tile_back.png”,
    width = 62, 
    height = 62,
}
```

如你所猜，`type` 键是在 `load` 函数中使用的。在这种情况下，我们需要将其设置为 `RESOURCE_TYPE_IMAGE` 类型。

在这里，我们定义了一个具有特定 `width` 和 `height` 值的图像，并且它位于 `assets/title_back.png`。记住，我们将使用 `ASSET_PATH` 以避免多次写入 `assets/`。这就是为什么我们在定义中不写它的原因。

另一个有用的定义是：

```swift
{
    type = RESOURCE_TYPE_IMAGE
    fileName = “tile_back.png”,
    coords = { -10, -10, 10, 10 }
}
```

当你想要在较大图像中特定矩形时，这很有用。你可以使用 `cords` 属性来定义这个矩形。例如，通过指定 `coords = { -10, -10, 10, 10 }`，我们得到一个边长为 20 像素的正方形，其中心位于图像中。

现在，让我们看看实际的 `loadImage` 方法，看看这一切是如何结合在一起的：

```swift
function ResourceManager:loadImage ( definition )
    local image
```

首先，我们使用与之前相同的技术定义一个空变量，该变量将保存我们的图像：

```swift
    local filePath = ASSETS_PATH .. definition.fileName
```

我们通过将定义中 `fileName` 的值附加到 `ASSETS_PATH` 常量的值来创建实际的全路径。`if` 检查 `coords` 属性是否已定义：

```swift
    if definition.coords then
    image = self:loadGfxQuad2D ( filePath, definition.coords )
```

然后，我们使用另一个辅助函数，称为 `loadGfxQuad2D`。这个函数将负责创建实际图像。我们使用另一个辅助函数的原因是，用于创建图像的代码在两种定义风格中都是相同的，但定义中的数据需要以不同的方式处理。在这种情况下，我们只需传递矩形的坐标。

```swift
  else 
    local halfWidth = definition.width / 2
    local halfHeight = definition.height / 2
    image = self:loadGfxQuad2D(filePath, {-halfWidth, -halfHeight, halfWidth, halfHeight} ) 
```

如果没有 `coords` 属性，我们会假设图像是使用 `width` 和 `height` 定义的。所以我们做的是定义一个覆盖整个宽度和高度的矩形。我们通过计算 `halfWidth` 和 `halfHeight` 并将这些值传递给 `loadGfxQuad2D` 方法来完成此操作。记住 Moai SDK 中关于纹理坐标的讨论；这就是为什么我们需要将维度除以 2 并将它们作为负数和正数参数传递给矩形的原因。这允许它在 (0, 0) 上居中。在加载图像后，我们返回它，以便它可以通过 `load` 方法存储在缓存中：

```swift
    end
    return image
end
```

现在我们需要编写的最后一个方法是 `loadGfxQuad2D`。这个方法基本上与我们在上一章中用于显示图像的方法相同：

```swift
function ResourceManager:loadGfxQuad2D ( filePath, coords )
    local image = MOAIGfxQuad2D.new ()

    image:setTexture ( filePath )
    image:setRect ( unpack(cords) )

    return image
end
```

### 小贴士

Lua 的 `unpack` 方法是一个很好的工具，它允许你将一个表作为单独的参数传递。你可以用它将一个表拆分成多个变量：

```swift
x, y = unpack ( position_table )
```

我们在这里所做的是实例化 `MOAIGfxQuad2D` 类，设置我们在上一个函数中定义的纹理，并使用我们构建的坐标来设置图像将从原始纹理中使用的矩形。然后我们返回它，以便 `loadImage` 可以使用它。

好吧！这就是关于图像的所有内容。一开始可能看起来很复杂，但实际上并不复杂。

其余的资产将比这更简单，所以如果你理解了这个，其余的就会变得容易。

### 瓦片图像

另一种有用的资产类型是瓦片图像。在游戏中，我们通常使用瓦片图像来加载单个纹理，并使用其片段来完成特定目的（例如，可以使用这种技术实现动画）。这将在我们的游戏 *Concentration* 中很有用，用于定义哪些瓦片应该放置在矩阵中。

瓦片图像定义如下：

```swift
{
    type = RESOURCE_TYPE_TILED_IMAGE, 
    fileName = ‘tiles.png’, 
    tileMapSize = {3, 2}
}
```

这基本上与正常图像相同，但我们需要添加 `tileMapSize` 并移除 `width` 和 `height`。这个表将被解包为 `MOAITileDeck2D:setRect` 的参数（请参阅 [`getmoai.com/docs/class_m_o_a_i_tile_deck2_d.html#a504da8814f74038af9d453ebbc7fd5ae`](http://getmoai.com/docs/class_m_o_a_i_tile_deck2_d.html#a504da8814f74038af9d453ebbc7fd5ae)）。在这种情况下，它表示瓦片图有 3 列和 2 行，总共 6 个瓦片。

`loadTiledImage` 的代码如下：

```swift
function ResourceManager:loadTiledImage ( definition )

    local tiledImage = MOAITileDeck2D.new ()

    local filePath = ASSETS_PATH .. definition.fileName

    tiledImage:setTexture ( filePath )
```

这基本上与我们对图像所做的是相同的。我们将 `fileName` 添加到 `ASSETS_PATH` 以获取最终的纹理 `filePath`。

```swift
    tiledImage:setSize ( unpack (definition.tileMapSize) )
```

唯一真正的区别是我们将 `tileMapSize` 属性解包以传递给 `setSize`。您可以使用所有可以传递给 `setSize` 的可能参数；这些在 Moai SDK API 文档中有解释。

```swift
    -- return the final image
    return tiledImage

end
```

这与正常图像的处理方式基本相同。我们将 `tiledImage` 返回以供后续使用。

### 字体

字体更容易，其定义如下：

```swift
{
    type = RESOURCE_TYPE_FONT,
    fileName = ‘myfont.ttf’,
    glyphs = “0123456789”,
    fontSize = 26,
    dpi = 160
}
```

除了类型和文件名之外，我们这里的具体属性还包括 `glyphs`（这是我们将会使用的字符；在这个例子中，我们只使用了从 0 到 9 的数字）、`fontSize` 和字体的大小 `dpi`。

`loadFont` 方法的代码如下：

```swift
function ResourceManager:loadFont ( definition )
    local font = MOAIFont.new ()
    local filePath = ASSETS_PATH .. definition.fileName
    font:loadFromTTF ( filePath, definition.glyphs, 
    definition.fontSize, definition.dpi )
    return font
end
```

整个方法基本上与其他加载器相同；区别在于我们使用 `MOAIFont` 作为类，并使用 `loadFromTTF` 以及必要的参数来初始化它。

### 声音

声音的定义如下：

```swift
{
  type = RESOURCE_TYPE_SOUND, 
  fileName = ‘sugarfree.mp3’, 
  loop = true,
  volume = 0.6
}
```

声音的具体属性包括 `loop`（是否应该持续循环播放）和 `volume`（初始音量）。

Moai SDK 包含对两种不同声音引擎的支持，**Untz**，一个专门为 Moai SDK 创建的开源引擎，以及 **FMOD**，一个行业标准的声音库。值得一提的是，FMOD 需要付费，而 Untz 是免费的。

我们将在本书中向您展示如何使用 Untz，但您可以将 `loadSound` 编码为使用 FMOD（请参阅 Moai SDK API 文档中的 `MOAIFmodExSound` 和 `MOAIFmodExChannel`）。 

```swift
function ResourceManager:loadSound ( definition )
    local sound = MOAIUntzSound.new ()

    local filePath = ASSETS_PATH .. definition.fileName
    sound:load ( filePath )

    sound:setVolume ( definition.volume )
    sound:setLooping ( definition.loop )

    return sound
end
```

这基本上与之前的加载器逻辑相同，但我们使用 `setVolume` 和 `setLooping` 来使用我们在资源定义中选择的属性。我们尚未初始化 Untz；这将在稍后完成，但这段代码已经在这里解释，以完善 `ResourceManager`。

# 练习

将这些组合起来应该是你现在可以实现的。所以，取我们之前创建的 `main.lua` 文件的源代码，并对其进行修改以使用我们的 `ResourceManager` 和 `ResourceDefinitions` 实体。您需要在 `main.lua` 中包含 `resource_definition.lua` 和 `resource_manager.lua`，然后充分利用它们。如果您从网站上下载本章的源代码，`main.lua` 已经被修改为与 `ResourceManager` 一起工作。

另一个练习是编写一种方法，从`ResourceManager`中卸载资源。这一功能也在本章的代码中实现，您可以从网站上下载。

### 注意

`unload`方法非常重要，因为实际上需要它来释放为资源保留的内存。

另一个有趣的话题是增强我们的`ResourceManager`实体，即使用序列化字体，以便在运行时更快地加载字体。

# 摘要

在本章中，我们创建了一个简单的框架来处理资源。我们创建了定义图像、声音和字体的方法，并编写了`ResourceDefinitions`类来存储这些数据。我们还编写了`ResourceManager`类，该类负责加载不同的资源并对其进行缓存。我们还介绍了为这些不同资源创建加载函数的方法，并解释了如何定义它们。现在，我们已准备好开始开发我们游戏的游戏玩法；这有多么酷啊！
