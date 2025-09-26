# 第八章. 屏幕分辨率和方向变化

在本章中，我们将涵盖：

+   针对设备

+   针对 Retina 显示屏

+   支持多个分辨率

+   设置默认宽高比

+   启用自动旋转

+   监听方向变化

+   响应方向变化

# 简介

加速度传感器的加入使得原始 iPhone 能够检测其物理方向的变化；根据用户的偏好重新排列其屏幕内容以适应纵向或横向观看。然而，提供这一点意味着应用程序必须为两种屏幕宽高比进行设计和编码——一种是纵向，另一种是横向。

随着 iOS 家族的壮大，必须支持的屏幕分辨率数量也在增加。随着 iPad 的推出和 Retina 显示屏设备的引入，现在有三个不同的分辨率。如果你还考虑了两种屏幕宽高比，那么你的应用程序的屏幕布局管理可能会变得令人望而生畏。

本章提供了将帮助你在任何 iOS 设备上正确渲染内容的菜谱，无论其屏幕分辨率和物理方向如何。

# 针对设备

iOS 家族包括三种设备：iPhone、iPod touch 和 iPad。在为 iOS 开发时，你必须声明你具体针对的设备。你的选择将决定你的应用程序如何由这三种设备运行。

这个菜谱将带你完成创建一个针对整个家族的应用程序所需的必要步骤。

## 准备工作

已提供了一个 FLA 作为本菜谱的起点。从书籍的配套代码包中打开`chapter8\recipe1\recipe.fla`。

在舞台上有一个名为`output`的动态文本字段。我们将使用这个文本字段来报告应用程序当前正在运行的设备。

为了避免疑问，这个菜谱将不会利用 iPhone 4/4S 和第四代 iPod touch 的 Retina 显示屏分辨率。如果你想提供 Retina 支持，请参阅本章的*针对 Retina 显示屏*菜谱。

## 如何做到这一点...

我们将指定目标设备，然后编写一些 ActionScript 来报告应用程序当前正在运行的设备。

### 设置目标设备

让我们先设置目标设备：

1.  通过从 Flash Professional 中选择**文件** | **AIR for iOS 设置**来打开 AIR for iOS 设置面板。

1.  确保已选择**常规**选项卡。

1.  从**设备**字段中选择**iPhone 和 iPad**。

1.  如果你正在使用 Flash Professional CS5.5，请将**分辨率**字段设置为**标准**。

1.  点击**确定**以关闭面板。

### 检测当前设备

现在，让我们继续添加这个菜谱所需的 ActionScript：

1.  创建一个文档类；命名为`Main`。

1.  将以下三个导入语句添加到类中：

    ```swift
    import flash.display.MovieClip;
     import flash.display.StageAlign;
    import flash.display.StageScaleMode;
    import flash.system.Capabilities;

    ```

1.  在构造函数中，设置舞台的缩放模式和定位：

    ```swift
    public function Main() {
    stage.scaleMode = StageScaleMode.NO_SCALE;
    stage.align = StageAlign.TOP_LEFT;
    }

    ```

1.  使用 `Capabilities` 类，确定应用正在运行的设备：

    ```swift
    public function Main() {
    stage.scaleMode = StageScaleMode.NO_SCALE;
    stage.align = StageAlign.TOP_LEFT;
     if(Capabilities.screenResolutionX == 320)
    {
    output.text = "iPhone / iPod touch";
    }
    else if(Capabilities.screenResolutionX == 768)
    {
    output.text = "iPad";
    } 
    }

    ```

1.  将文档类保存在与您的 FLA 相同的文件夹中，并将其命名为 `Main.as`。同时保存 FLA。

1.  在您的设备上发布和测试应用。如果您有多台设备，请安装到多台设备上。

当您在 iPhone 或 iPod touch 上运行它时，以下文本将输出到屏幕上：

**iPhone / iPod touch**

在 iPad 上测试的人将看到以下内容：

```swift
iPad

```

## 它是如何工作的...

您可以通过设置 AIR for iOS 设置面板中的 **Device** 字段来指定您希望针对的设备。以下有三个选项可供您选择：

+   **iPhone**

+   **iPad**

+   **iPhone 和 iPad**

iPod touch 没有列出，因为它被视为 iPhone——这两个设备具有相同的宽高比和分辨率。

选择 **iPhone** 将屏幕大小限制在 iPhone 和 iPod touch 的大小。您的应用仍然可以在 iPad 上运行，但它不会尝试利用 iPad 的更高分辨率。相反，iPad 将使用 iPhone 的标准 320x480 分辨率来运行应用。用户可以选择在正常大小下运行应用或使用像素加倍来覆盖几乎整个屏幕。

选择 **iPad** 可以利用 iPad 的整个 768x1024 分辨率，但将您的应用限制在该设备上。

如果您正在编写一个可以通过适应不同屏幕尺寸在所有三种设备上运行的 app，请选择 **iPhone 和 iPad**。这通常被称为通用应用，也是我们在本菜谱中选择的类型。

苹果承认编写针对多个设备和屏幕分辨率的难度。为了简化开发，当针对整个 iOS 系列时，他们提供了以下两个选项：

+   编写两个单独的应用——一个用于 iPad，另一个用于 iPhone/iPod touch

+   编写一个适应每个设备屏幕大小的通用应用

当然，您不必支持所有三种设备。例如，编写一个仅针对 iPad 的应用是完全可行的。

通过查询设备的屏幕分辨率，可以确定您的应用正在运行的设备。iPhone 和 iPod touch 的水平分辨率为 320 像素，而 iPad 的宽度为 768 像素。因此，我们只需要查询静态的 `Capabilities.screenResolutionX` 属性来确定设备类型，而无需也检查垂直分辨率 `Capabilities.screenResolutionY`。

您可能会想使用 `Stage.stageWidth`，但请记住，由于舞台尺寸固定，此菜谱中所有设备类型都将返回相同的宽度。

您可以从 Adobe 社区帮助中获取有关 `flash.system.Capabilities, flash.display.StageScaleMode` 和 `flash.display.StageAlign` 的更多信息。

## 更多信息...

在针对多个设备时，还应考虑以下信息。

### 应用程序启动图像

当针对多种设备类型时，请记住为每个支持的分辨率包含一个启动图像。如果您选择支持它们，也可以提供不同方向的备用启动图像。有关更多详细信息，请参阅第三章*包含应用程序启动图像*的菜谱[ch03.html "第三章。编写您的第一个应用程序"]。

### 包含图标

请记住包含每个设备类型所需的适当图标艺术品。有关更多详细信息，请参阅第三章*包含图标*的菜谱[ch03.html "第三章。编写您的第一个应用程序"]。

### 横屏宽高比

`screenResolutionX` 和 `screenResolutionY` 属性总是以设备竖直方向握持时返回。例如，一个默认横屏方向的 iPad 应用仍然会返回水平分辨率为 768 像素，垂直分辨率为 1024 像素的分辨率。

## 参见

+   *AIR for iOS 通用设置，第二章*

+   *针对 Retina 显示*

+   *设置默认宽高比*

# 针对 Retina 显示

iPhone 4 将 Retina 显示引入了世界。它将四倍的像素数压缩到与早期型号相同的屏幕中，提供了在移动设备上所见到的最清晰的图像。

使用 Flash Professional CS5.5 和至少 AIR 2.6 的用户可以使用 Retina 显示的 iPhone 和 iPod touch 提供的完整 640x960 分辨率。

## 准备中

从本书的配套代码示例中，将 `chapter8\recipe2\recipe.fla` 打开到 Flash Professional CS5.5 中。

这个 FLA 已作为起点提供，并在舞台上有一个名为 `output` 的动态文本字段。我们将使用这个文本字段来报告应用是否在 Retina 显示设备上运行。

虽然这个菜谱的示例应用可以在 iPad 上运行，但它不会针对其 768x1024 的屏幕分辨率。相反，它将模拟标准分辨率的 iPhone/iPod touch。

## 如何操作...

Retina 显示支持是在 AIR for iOS 设置面板中指定的。我们还将编写一个基本的文档类来确定应用实际上是否在 Retina 设备上运行。

### 设置 Retina 显示支持

让我们先启用对 Retina 显示屏幕分辨率的支持：

1.  通过选择**文件** | **AIR for iOS 设置**来打开 AIR for iOS 设置面板。

1.  确保已选择**常规**选项卡。

1.  将**分辨率**字段设置为**高**。

1.  此外，对于这个菜谱，确保**设备**字段设置为**iPhone**。

1.  点击**确定**按钮关闭面板。

### 检测 Retina 显示设备

在处理完 AIR for iOS 设置后，我们可以将注意力转向确定应用是否在 Retina 显示设备上运行的所需 ActionScript：

1.  创建一个文档类，并将其命名为 `Main`。

1.  将以下三个导入语句添加到类中：

    ```swift
    import flash.display.MovieClip;
     import flash.display.StageAlign;
    import flash.display.StageScaleMode;
    import flash.system.Capabilities; 

    ```

1.  设置舞台的缩放模式和居中对齐属性：

    ```swift
    public function Main() {
     stage.scaleMode = StageScaleMode.NO_SCALE;
    stage.align = StageAlign.TOP_LEFT; 
    }

    ```

1.  查询设备的屏幕分辨率以确定它是否有视网膜显示屏：

    ```swift
    public function Main() {

    stage.scaleMode = StageScaleMode.NO_SCALE;
    stage.align = StageAlign.TOP_LEFT;
     if(Capabilities.screenResolutionX == 320)
    {
    output.text = "Standard Resolution";
    }
    else if(Capabilities.screenResolutionX == 640)
    {
    output.text = "Retina Resolution";
    } 
    }

    ```

1.  将类文件保存为`Main.as`在 FLA 相同的文件夹中。也要保存 FLA。

1.  发布并部署 IPA 到你的设备。如果你有视网膜和标准显示屏设备可用，那么在两个设备上安装应用。

当你在视网膜显示屏的 iPhone 或 iPod touch 上运行应用时，以下文本将被输出到屏幕：

**视网膜分辨率**

对于标准显示屏设备，你会看到以下内容：

**标准分辨率**

## 它是如何工作的...

虽然已经保持了宽高比，但在为 iPhone 和 iPod touch 开发时，现在有两个屏幕分辨率可以针对：

+   **标准**：320x480

+   **视网膜**：640x960

视网膜显示屏的分辨率出现在第三代 iPhone 和 iPod touch 之后。所有之前的型号都使用标准显示屏分辨率。

如果你选择支持视网膜显示屏，那么你的应用也必须支持标准显示屏。苹果不允许你为每个支持的分辨率提交单独的 IPA。然而，你可以选择只支持标准显示屏，iOS 会将应用像素加倍以适应视网膜显示屏屏幕。

要指定你的应用将支持的分辨率，请在 AIR for iOS 设置面板中设置**分辨率**字段。以下有两个选项可用：

+   **标准**

+   **高**

如果你同时为视网膜和标准显示屏提供支持，请选择**高**。否则选择**标准**。

当针对多个屏幕分辨率时，设置舞台的缩放模式和校准非常重要。

将缩放模式设置为`NO_SCALE`可以防止舞台内容缩放以适应视网膜屏幕的分辨率。相反，你的内容将保持其定义的大小。将舞台校准设置为`TOP_LEFT`将强制所有内容相对于左上角进行定位。

虽然缩放可能听起来很吸引人，但对于使用位图的应用程序来说，实际上并不建议这样做。位图在缩放时会出现像素化。相反，你应在你的应用中编写逻辑来在标准分辨率和视网膜分辨率的位图版本之间切换，并自行处理内容和布局的位置。

此外，你的 FLA 舞台大小应该与标准分辨率显示屏匹配——320x480 像素。当在视网膜显示屏设备上运行，并且将舞台的缩放模式设置为`NO_SCALE`时，舞台的尺寸将被报告为是你在 Flash Professional 中设置的尺寸的两倍。这将允许你利用更高的分辨率。

通过查询属于`Capabilities`类的静态`screenResolutionX`和`screenResolutionY`属性，可以确定你的应用正在使用的屏幕分辨率。对于这个菜谱，我们只是检查了`screenResolutionX`，因为这足以确定是否正在使用标准或视网膜屏幕分辨率。

您可以从 Adobe 社区帮助中获取有关 `flash.system.Capabilities`、`flash.display.StageScaleMode` 和 `flash.display.StageAlign` 的更多信息。

## 还有更多...

在使用 Retina 显示屏时，还有其他一些考虑因素。

### Retina 主屏幕图标

当支持 Retina 显示屏时，您必须提供一个 114x114 像素的主屏幕图标。有关详细信息，请参阅第三章中的*包含图标*配方。

### Retina 应用程序启动图像

您可以在 IPA 中包含一个额外的启动图像，该图像利用了 Retina 显示屏的高分辨率。有关更多详细信息，请参阅第三章中的*包含应用程序启动图像*配方。

### 仅矢量应用程序

如果您的应用程序仅使用矢量内容，则可以简单地让 AIR 为您处理缩放。这将允许您编写一个针对标准 320x480 像素显示屏的应用程序，这将同时利用更高分辨率的 Retina 显示屏。

要启用此功能，请将场景的缩放模式和定位设置为以下内容：

```swift
stage.scaleMode = StageScaleMode.SHOW_ALL;
stage.align = StageAlign.TOP_LEFT;

```

此外，请记住将 AIR for iOS 设置面板中的**分辨率**字段设置为**高**。

不要将此与 iOS 使用的像素加倍混淆。您的应用程序的矢量内容实际上会被缩放，而不是每个像素的大小加倍。本质上，在 Retina 设备上会有明显的图像保真度提升。

当然，您仍然可以使用位图，但与矢量内容不同，当它们放大时，像素化将非常明显。

### iPad 显示屏

iPad 1 和 2 都支持 768x1024 像素的单一屏幕分辨率。当针对 iPad 或非 Retina 显示屏的 iPhone 和 iPod touch 时，将**分辨率**字段设置为**标准**。

## 参见

+   *AIR for iOS 通用设置，第二章*

+   *针对设备*

+   *支持多分辨率*

# 支持多分辨率

iPad 的推出导致开发者必须支持两种不同的物理屏幕尺寸和分辨率。iPhone 和 iPod touch 引入 Retina 显示屏后，这种情况更加严重，因为同一设备类型的屏幕分辨率不再保证相同。

当针对多种设备和型号时，您的应用程序能够适应不同的屏幕分辨率非常重要。

让我们通过一个简单的示例来创建一个支持 iPhone、iPod touch 和 iPad 使用的各种纵向分辨率的通用应用程序。

## 准备工作

书籍附带的代码包中提供了一个包含图形的 FLA 文件，供您从中工作。

将 `chapter8\recipe3\recipe.fla` 打开到 Flash Professional 中。

在 FLA 的库中，你会找到三个 PNG 图像——每个屏幕分辨率一个。每个图像也已被链接以供 ActionScript 使用，允许它在运行时轻松添加到屏幕上。以下表格总结了这一点：

|   | 位图名称 | AS 链接 |
| --- | --- | --- |
| 标准显示 320x480 | rocket-standard.png | RocketStandardBitmapData |
| 视网膜显示屏 640x960 | rocket-retina.png | RocketRetinaBitmapData |
| iPad 显示 768x1024 | rocket-ipad.png | RocketIPadBitmapData |

当在标准显示的 iPhone 或 iPod touch 上运行时，屏幕上会显示 **rocket-standard.png**。对于视网膜显示屏，应用程序将使用 **rocket-retina.png**。最后，如果检测到 iPad 的屏幕分辨率，将显示 **rocket-ipad.png**。

如果你使用 Flash Professional CS5 进行这个菜谱，那么你的应用程序将无法利用 Retina 显示设备提供的更高分辨率。Retina 显示支持仅从 AIR 2.6 开始可用。

## 如何操作...

我们将把这个菜谱分成两部分。首先，从 AIR for iOS 设置面板中，我们将声明支持所有设备分辨率。其次，我们将编写显示每个支持分辨率的正确位图的代码。

### 设置支持的设备和分辨率

让我们先针对所有可用的设备分辨率进行目标定位：

1.  通过选择 **文件** | **AIR for iOS 设置** 打开 AIR for iOS 设置面板。

1.  确保选择了 **常规** 选项卡。

1.  将 **设备** 字段设置为 **iPhone 和 iPad**。

1.  如果你使用的是 Flash Professional CS5.5，那么也将 **分辨率** 字段设置为 **高**。

    如果你希望单个应用程序支持 iOS 家族中的所有设备并利用 Retina 显示设备提供的更高分辨率，则需要这些设置。

1.  点击 **确定** 按钮关闭面板。

### 显示正确的位图

现在，我们将编写一些 ActionScript 代码以适应预期的每个屏幕分辨率：

1.  创建一个文档类，并将其命名为 `Main`。

1.  为所需的类添加导入语句：

    ```swift
    import flash.display.MovieClip;
     import flash.display.Bitmap;
    import flash.display.StageAlign;
    import flash.display.StageScaleMode;
    import flash.system.Capabilities; 

    ```

1.  在构造函数中，设置舞台的缩放模式和定位属性：

    ```swift
    public function Main() {
     stage.scaleMode = StageScaleMode.NO_SCALE;
    stage.align = StageAlign.TOP_LEFT;
    }

    ```

1.  在构造函数中，调用一个方法来为当前设备的屏幕分辨率创建正确的位图：

    ```swift
    public function Main() {
    stage.scaleMode = StageScaleMode.NO_SCALE;
    stage.align = StageAlign.TOP_LEFT;
    var b:Bitmap = createRocketBitmap();
    addChild(b);
    }

    ```

1.  现在，编写一个实例化和返回位图的函数：

    ```swift
    private function createRocketBitmap():Bitmap {
    var b:Bitmap;
    if(Capabilities.screenResolutionX == 320)
    {
    b = new Bitmap(new RocketStandardBitmapData());
    }
    else if(Capabilities.screenResolutionX == 640)
    {
    b = new Bitmap(new RocketRetinaBitmapData());
    }
    else if(Capabilities.screenResolutionX == 768)
    {
    b = new Bitmap(new RocketIPadBitmapData());
    }
    return b;
    }

    ```

1.  将类文件保存为 `Main.as`。同时，回到你的 FLA 并保存它。

1.  发布 FLA 并在一系列 iOS 设备上测试你的应用程序。

## 它是如何工作的...

这个菜谱的示例确定正在使用的屏幕分辨率，并显示为该分辨率准备的位图图像。

支持以下纵向分辨率：

+   iPad

    +   768x1024

+   iPhone 和 iPod touch

    +   320x480: 标准

    +   640x960: 视网膜

从 AIR for iOS 设置面板中，你必须明确声明你打算创建一个支持所有三种设备和它们屏幕分辨率的通用应用程序。这是通过将 **设备** 字段设置为 **iPhone 和 iPad** 以及将 **分辨率** 字段设置为 **高** 来完成的。

当针对多个屏幕分辨率时，设置舞台的缩放模式和校准非常重要。

将舞台的缩放模式设置为`NO_SCALE`确保你的内容在多个屏幕分辨率下保持其定义的大小。这让你完全控制内容的尺寸和布局。它还确保正在使用的位图在运行时不会被缩放，这将防止它们变得像素化。将舞台校准设置为`TOP_LEFT`强制所有内容相对于左上角定位，这使得定位更容易处理。

创建正确位图的实际代码只是简单地检查静态的`Capabilities.screenResolutionX`属性，以确定应用程序当前需要为哪三个预期的屏幕分辨率提供服务。一旦做出决定，正确的位图从库中实例化并添加到显示列表，其默认位置为(0,0)。

虽然舞台和时间轴仍然是内容布局的选项，但你可能会发现自己更依赖于 ActionScript 来处理更复杂的项目。

你可以从 Adobe 社区帮助中获取更多信息。搜索以下类：`flash.system.Capabilities, flash.display.StageScaleMode`, 和 `flash.display.StageAlign`。

## 还有更多...

我们的例子有些简单。不幸的是，将实际应用适配到多个屏幕分辨率可能会很困难。

### 默认舞台大小

当其缩放模式设置为`NO_SCALE`时，舞台的默认大小对通用应用来说并不重要。例如，这个菜谱的 FLA 使用的是 320x480 的舞台大小，这显然远远低于 iPad 的 768x1024 分辨率。然而，这并不会限制应用程序的显示列表到舞台的默认尺寸——应用程序仍然能够针对其运行设备的全屏分辨率。

在进行跨多个分辨率的布局管理时，尽量不要依赖于默认的舞台大小。当确定你的应用程序当前运行在哪种设备上时，也是如此。相反，使用`Capabilities.screenResolutionX`和`Capabilities.screenResolutionY`。

### 模型-视图-控制器架构

尽管这超出了本书的范围，但你应该考虑使用模型-视图-控制器（MVC）设计模式来管理每个支持的屏幕分辨率的布局。

MVC 由三个元素组成：一个包含应用程序数据和状态管理逻辑的单个模型；至少一个视图，用于展示应用程序的屏幕状态；以及至少一个控制器，用于处理用户交互。

通常，一个视图会与一个控制器配对，你将为每个你希望支持的屏幕分辨率编写一个视图-控制器对。控制器在其视图和模型之间充当中间人。

有许多 ActionScript MVC 框架可用，例如 PureMVC 和 Robotlegs。你可以访问 PureMVC 的主页[`puremvc.org`](http://puremvc.org)，而 Robotlegs 可以从[www.robotlegs.org](http://www.robotlegs.org)下载。

### 小贴士

苹果鼓励在为 iOS 编写应用时使用 MVC 模式。

### 动态尺寸和布局

当处理多个屏幕分辨率和像素密度时，你可能需要考虑开发你的应用以动态调整其内容的大小和布局。虽然这可能是一个复杂的方法，但它将为你的应用提供未来保障，使其能够适应未来可能出现的其他屏幕。然而，对于有限的屏幕数量，你可能发现 MVC 更可行。

更多详情，请查看 Adobe AIR 开发者中心的“编写多屏幕 AIR 应用”文章：[www.adobe.com/devnet/air/flex/articles/writing_multiscreen_air_apps.html](http://www.adobe.com/devnet/air/flex/articles/writing_multiscreen_air_apps.html)。

### 单独支持 iPad

iPad 屏幕的宽高比与 iPhone 和 iPod touch 不同。这可能会使得开发一个适用于所有三个设备的单一应用变得尴尬，因为应用需要在运行时针对三种不同的分辨率调整其 UI。此外，对于位图密集型应用，存储三套图形将消耗大量内存。

为了减轻在一个应用中支持三个设备的复杂性，你可以单独编写你的 iPad 版本。App Store 将接受一个针对 iPad 的应用版本以及一个同时支持 iPhone 和 iPod touch 的应用版本。只需从 AIR for iOS 设置面板中指定你的目标设备。

## 相关内容

+   *AIR for iOS 通用设置，第二章*

+   *使用 ActionScript 访问位图，第六章*

+   *调整位图大小，第四章*

# 设置默认宽高比

像 iPhone 这样的设备可以在真实空间中自由旋转。这很方便，因为它提供了两种可以工作的宽高比——纵向和横向。

让我们看看如何选择默认宽高比。

## 准备工作

使用 Flash Professional，从书籍配套代码包中打开`chapter8\recipe4\recipe.fla`。

该库包含一个为 480x320 像素横幅分辨率设计的消防车位图。

## 如何做到...

我们将首先将 FLA 锁定为横幅宽高比：

1.  使用**选择工具(V**)，在舞台上的任何位置点击。从**属性**面板中，将舞台的宽高比从**纵向**更改为**横向**。要做到这一点，只需将其宽度设置为**480**，高度设置为**320**。

1.  虽然舞台大小暗示了这一点，但 AIR for iOS 设置也需要更改以反映我们选择的宽高比。从下拉菜单中选择**文件** | **AIR for iOS 设置**。

1.  确保已选中**常规**选项卡。

1.  从**宽高比**下拉框中选择**横向**。

1.  点击**确定**以关闭面板。

1.  最后，将库中的位图图像拖动到舞台上，并将其放置在屏幕的左上角。

1.  保存您的 FLA 文件并发布它。

1.  在您的设备上安装应用程序并启动它。

为了正确查看消防车图像，您将不得不将设备侧放，使设备的物理方向与舞台的方向相匹配。

## 它是如何工作的...

您的应用程序可以从两种宽高比之一开始：纵向或横向。这是通过在 AIR for iOS 设置面板中的**常规**选项卡内从**宽高比**下拉框中进行选择来完成的。

此外，您还需要将舞台设置为与所选宽高比相匹配。这将允许您在设计应用程序时正确布局任何内容。

### 注意

当您创建一个新的 AIR for iOS 文档时，默认设置为纵向宽高比。

## 更多内容...

当从 Flash Professional CS5.5 设置默认宽高比时，还有一个额外的选项可用。

### 自动宽高比

从 AIR 2.6 开始，您的应用程序可以在启动时被指示选择与设备当前方向匹配的宽高比。为了适应这一点，从**宽高比**下拉框中提供了一个名为**自动**的第三个选项。

此功能只能与自动方向一起启用，这在下一道菜谱中有所介绍。

## 参见

+   *AIR for iOS 常规设置，第二章*

+   *启用自动方向*

# 启用自动方向

用户可以在任何时刻在真实空间中更改设备的方向，并且可能会期望屏幕上的内容能够反映这一点。AIR for iOS 可以检测这些变化并自动旋转舞台以匹配设备的新物理方向。

本菜谱将向您展示如何启用自动方向。

## 准备工作

如果您已经完成了*设置默认宽高比*菜谱，那么您可以从其 FLA 继续工作。或者，从本书的配套源代码中，在 Flash Professional 中打开`chapter8\recipe5\recipe.fla`，然后从那里开始工作。

在任何情况下，您都会在舞台上看到一个消防车的位图图像。舞台本身的大小是为了适应横向宽高比，FLA 的 AIR for iOS 设置反映了这一点。

## 如何操作...

通过执行以下步骤启用自动方向：

1.  通过选择**文件** | **AIR for iOS 设置**来打开 AIR for iOS 设置面板。

1.  确保已选中**常规**选项卡。

1.  将**宽高比**下拉框设置为**纵向**。

1.  选中**自动方向**复选框。

1.  点击**确定**按钮以关闭面板。

1.  保存您的 FLA 文件并发布它。

1.  将应用程序部署到您的设备并启动它。

当设备的物理方向改变时，舞台将自动旋转以匹配这一变化。

## 它是如何工作的...

当自动旋转启用时，每次旋转时舞台都会尝试匹配设备的当前方向。iOS 支持四种方向：

+   默认：iOS 设备的默认纵向方向

+   向左旋转：顺时针旋转 90 度

+   向右旋转：逆时针旋转 90 度

+   颠倒：旋转 180 度

自动旋转很方便，因为它可以防止你不得不手动旋转显示列表中的对象。相反，这项任务将代表你完成。

在自动旋转期间，舞台的缩放模式和当前的对其设置将被遵守。

此外，如果你决定支持纵向和横向的宽高比，那么苹果建议你的应用以纵向模式启动。

## 更多...

很可能你希望对舞台上的内容在自动旋转时的处理方式有一定的控制。

### 检查方向支持

虽然 Flash 为所有 iOS 设备提供了对设备方向变化的支持，但在编写跨平台代码时，你可能想通过检查舞台的`supportsOrientationChange`属性返回`true`来确认支持。

### 通过编程设置自动旋转

除了使用 AIR for iOS 设置面板外，你还可以通过设置`Stage.autoOrients`属性来程序化地激活和停用自动旋转。该属性的初始值是从 AIR for iOS 设置面板中的**自动旋转**字段派生的。

以下 ActionScript 激活了自动旋转：

```swift
stage.autoOrients = true;

```

### 舞台缩放模式

你可以指定当设备的物理方向改变时舞台的缩放方式。这是通过将`Stage.scaleMode`属性设置为`flash.display.StageScaleMode`类提供的四个常量之一来实现的。

默认模式是`StageScaleMode.SHOW_ALL`，确保整个舞台在屏幕内可见，同时保持其宽高比。如果你不希望在方向变化时调整舞台内容的尺寸，则将缩放模式设置为`NO_SCALE`。这指定了舞台内容的大小将保持不变。

关于缩放模式的更多信息可以在 Adobe 社区帮助中找到。

### 舞台对齐

当设备的物理方向改变时，舞台内容对齐的方式也可以指定。只需将`Stage.align`属性设置为`flash.display.StageAlign`定义的常量之一即可。

舞台的内容默认将居中显示在屏幕上，然而，有八个常量可以用来将舞台的内容对齐到屏幕的边缘。例如，以下 ActionScript 将对舞台进行左上角对齐：

```swift
stage.align = StageAlign.TOP_LEFT;

```

关于舞台对齐的更多信息可以在 Adobe 社区帮助中找到。

## 相关内容

+   *AIR for iOS 常规设置，第二章*

+   *监听方向变化*

# 监听方向变化

iOS 设备中存在的加速度计可以检测物理方向的变化。AIR 提供了 API 来检测这些变化并确定设备在真实空间中的实际方向。

让我们看看这是如何实现的。

## 准备工作

首先，从书籍附带代码包中打开`chapter8\recipe6\recipe.fla`。

在舞台上可以找到一个名为`output`的动态文本字段。我们将使用此文本字段来报告设备经历的方向变化。

## 如何实现...

执行以下步骤以监听方向变化：

1.  从 Flash Professional 的下拉菜单中选择**文件** | **AIR for iOS 设置**。

1.  从**常规**选项卡中，勾选**自动旋转**复选框；然后点击面板的**确定**按钮。

1.  创建一个文档类，并将其命名为`Main`。

1.  添加以下导入语句：

    ```swift
    import flash.display.MovieClip;
     import flash.display.StageAlign;
    import flash.display.StageScaleMode;
    import flash.events.StageOrientationEvent; 

    ```

1.  在构造函数中，设置舞台，并监听正在分发的`ORIENTATION_CHANGING`和`ORIENTATION_CHANGE`事件：

    ```swift
    public function Main() {
     stage.align = StageAlign.TOP_LEFT;
    stage.scaleMode = StageScaleMode.NO_SCALE;
    stage.addEventListener(
    StageOrientationEvent.ORIENTATION_CHANGING,
    orientationChanging);
    stage.addEventListener(
    StageOrientationEvent.ORIENTATION_CHANGE,
    orientationChanged);
    output.text = "Rotate the device."; 
    }

    ```

1.  为`ORIENTATION_CHANGING`事件编写一个处理器：

    ```swift
    private function orientationChanging(e:StageOrientationEvent):void {
    output.text = "Orientation Changing\n";
    output.appendText(
    "Current orientation: " + e.beforeOrientation + "\n");
    output.appendText(
    physical orientation changeslistening"Target orientation: " + e.afterOrientation + "\n\n");
    }

    ```

1.  最后，添加对`ORIENTATION_CHANGE`事件的处理器：

    ```swift
    private function orientationChanged(e:StageOrientationEvent):void {
    output.appendText("Orientation Changed\n");
    output.appendText(
    "Previous orientation: " + e.beforeOrientation + "\n");
    output.appendText(
    "Current orientation: " + e.afterOrientation + "\n\n");
    }

    ```

1.  当提示时，保存类并将其命名为`Main.as`。同时，保存您的 FLA 文件。

1.  发布应用程序并在您的设备上测试它。

通过旋转设备进行实验。一旦检测到方向变化，它们将在屏幕上显示。

## 它是如何工作的...

可以通过监听舞台上的以下两个事件来检测方向变化：

+   `StageOrientationEvent.ORIENTATION_CHANGING`

+   `StageOrientationEvent.ORIENTATION_CHANGE`

当设备的加速度计检测到正在发生方向变化时，会分发生`ORIENTATION_CHANGING`事件，并在自动旋转的默认行为发生之前。`ORIENTATION_CHANGE`事件在所有默认行为发生后分发生。

只有当从 AIR for iOS 设置面板或通过 ActionScript 启用自动旋转时，您的应用程序才会接收到`ORIENTATION_CHANGING`和`ORIENTATION_CHANGE`事件。

您可以通过查询事件对象的`beforeOrientation`和`afterOrientation`属性来检索有关方向变化的信息。这两个属性是只读的，并将设置为`flash.display.StageOrientation`中的以下常量之一：

+   `DEFAULT:` iOS 设备的默认纵向方向。

+   `ROTATED_LEFT:` 顺时针旋转 90 度。

+   `ROTATED_RIGHT:` 逆时针旋转 90 度。

+   `UPSIDE_DOWN:` 旋转了 180 度。

当处理`ORIENTATION_CHANGING`事件时，`beforeOrientation`将保持设备的当前方向，而`afterOrientation`将指定设备旋转到的目标方向。

对于`ORIENTATION_CHANGE`事件，使用`beforeOrientation`来获取设备的前一个方向，并使用`afterOrientation`来确定当前方向。

您可以在我们的文档类的`orientationChanging()`和`orientationChanged()`事件处理器中看到这两个属性的使用。它们的值被获取并写入`output`文本字段。

## 更多内容...

也可以在不使用事件监听器的情况下确定设备的方向。

### 确定设备方向

您还可以通过访问`Stage`类的`deviceOrientation`属性来确定设备的物理方向。此属性为只读，并将返回`StageOrientation`类中定义的其中一个常量。

除了我们已讨论的四个常量之外，`deviceOrientation`还可以返回`StageOrientation.UNKNOWN`。`deviceOrientation`属性不包含设备的起始方向，因此，在第一次发生物理方向变化之前，它被设置为`UNKNOWN`。

## 参见

+   *AIR for iOS 常规设置，第二章*

+   *响应方向变化*

+   *启用自动方向调整*

# 响应方向变化

您可能希望更新应用程序的屏幕内容以反映设备物理方向的变化。虽然自动方向调整与舞台的默认对齐和缩放模式将代表您尝试这样做，但您可能希望自行处理布局。

本食谱将向您展示如何做到这一点。

## 准备工作

已提供一个 FLA 作为起点。从本书的配套代码包中，在 Flash Professional 中打开`chapter8\recipe7\recipe.fla`。

您将在舞台上找到两个电影剪辑。第一个电影剪辑是一个向上指的箭头的图像，它已被创建以适应屏幕的尺寸，并使用纵向方向。第二个显示了相同的箭头，但这次图像的尺寸适合屏幕的横向方向。

两个电影剪辑都位于舞台的左上角，分别具有实例名称`portrait`和`landscape`。

我们将编写一些代码，以确保无论用户将设备旋转到四个可能的物理方向中的哪一个，都能看到向上指的箭头。我们将通过显示适合当前设备方向的适当电影剪辑来实现这一点。

舞台尺寸设置为初始的纵向宽高比，这已在 AIR for iOS 设置中反映出来。

## 如何做到这一点...

在转向 ActionScript 之前，我们将快速更改 AIR for iOS 设置。

1.  从 Flash Professional 的下拉菜单中选择**文件** | **AIR for iOS 设置**。

1.  在**常规**选项卡中，勾选**自动方向调整**复选框；然后单击面板的**确定**按钮。

1.  创建一个文档类并命名为`Main`。

1.  将以下导入语句添加到类中：

    ```swift
    import flash.display.MovieClip;
     import flash.display.StageAlign;
    import flash.display.StageScaleMode;
    import flash.display.StageOrientation;
    import flash.events.StageOrientationEvent; 

    ```

1.  在构造函数中设置舞台并监听`ORIENTATION_CHANGING`事件：

    ```swift
    public function Main() {
     stage.align = StageAlign.TOP_LEFT;
    stage.scaleMode = StageScaleMode.NO_SCALE;
    stage.addEventListener(
    StageOrientationEvent.ORIENTATION_CHANGING,
    orientationChanging);
    landscape.visible = false; 
    }

    ```

1.  最后，添加一个处理`ORIENTATION_CHANGING`事件的处理器：

    ```swift
    private function orientationChanging(e:StageOrientationEvent):void {
    if(
    e.afterOrientation == StageOrientation.DEFAULT || e.afterOrientation == StageOrientation.UPSIDE_DOWN
    )
    {
    portrait.visible = true;
    landscape.visible = false;
    }
    else if(
    e.afterOrientation == StageOrientation.ROTATED_LEFT || e.afterOrientation == StageOrientation.ROTATED_RIGHT
    )
    {
    portrait.visible = false;
    landscape.visible = true;
    }
    }

    ```

1.  将类文件保存为`Main.as`。也移动到您的 FLA 并保存它。

1.  发布 FLA 并将 IPA 部署到您的设备上。

1.  通过旋转您的设备来测试应用程序。屏幕将更新以反映任何方向变化。

## 它是如何工作的...

我们不是让舞台当前屏幕内容在设备方向改变时自动调整大小，而是选择显示不同的视图。

实现这一点的第一步是设置舞台的缩放模式和定位。

将缩放模式设置为`NO_SCALE`可以防止您的内容缩放以适应舞台大小的任何变化。这确保了在方向改变期间，`portrait`和`landscape`电影剪辑保持其定义的大小。

将对齐方式设置为`TOP_LEFT`会强制所有内容相对于屏幕的左上角进行定位。本质上，我们位于舞台(0,0)位置的两个电影剪辑将始终从屏幕的左上角开始，无论设备处于何种方向。

下一步是在方向改变时使相应的电影剪辑可见。这是通过处理`ORIENTATION_CHANGING`事件并根据`afterOrientation`属性的值选择电影剪辑来实现的。

如果设备正在旋转到两种竖屏方向之一（`DEFAULT`或`UPSIDE_DOWN`），则`portrait`电影剪辑将变为可见，而`landscape`将隐藏。对于两种横屏方向（`ROTATED_LEFT`或`ROTATED_RIGHT`），将显示`landscape`电影剪辑，而`portrait`将不可见。

您可能想知道为什么我们只有两个电影剪辑用于四种可能的屏幕方向。`portrait`和`landscape`在`UPSIDE_DOWN`和`ROTATED_LEFT`方向上不会颠倒吗？好吧，它们确实会短暂颠倒，但请记住，自动方向调整的一个好处是它会旋转您的屏幕内容，确保它以正确的方向显示。如果没有自动方向调整，您将不得不手动旋转显示列表上的所有项目。

## 还有更多...

这里有一些更多细节，可以帮助您完善对屏幕方向的知识。

### 舞台颜色

您选择的舞台颜色很重要，因为在自动方向调整期间，舞台尺寸之外的区域将是可见的。选择一种能够衬托您内容的舞台颜色。例如，这个菜谱使用了黑色，因为它与两个电影剪辑的背景颜色相匹配。

将舞台颜色改为白色，并将应用重新部署到您的设备上。在方向改变期间，您现在将能够看到实际上有多少区域是可见的。

### 舞台方向

尽管它们通常匹配，但设备在真实空间中的实际方向与屏幕上查看的内容是独立的。例如，用户可能手持设备处于横屏方向，而应用的内容仍在以竖屏方式渲染。

虽然`Stage.deviceOrientation`属性可用于确定设备的物理方向，但您可以使用`Stage.orientation`属性来检索舞台相对于设备当前的方向。

还可以独立于设备更改舞台的方向。这是通过调用`Stage.setOrientation()`并传递`flash.display.StageOrientation`类中定义的其中一个常量来实现的。您可以通过监听并响应`ORIENTATION_CHANGING`和`ORIENTATION_CHANGE`事件来对此方向更改做出响应。

### 防止自动旋转

自动旋转发生在`ORIENTATION_CHANGING`事件之后，但在`ORIENTATION_CHANGE`事件分发之前。因此，可以通过在`ORIENTATION_CHANGING`事件的对象上调用`preventDefault()`来防止特定方向（或所有方向）的自动旋转。

例如，以下对`orientationChanging()`处理器的修改将防止设备旋转到两种横屏方向之一时舞台被定位：

```swift
private function orientationChanging(e:StageOrientationEvent):void {
 if(
e.afterOrientation == StageOrientation.ROTATED_LEFT ||
e.afterOrientation == StageOrientation.ROTATED_RIGHT
)
{
e.preventDefault();
} 
}

```

您还可以使用`preventDefault()`来阻止自动旋转，以便完全自行管理舞台内容的旋转和定位。

## 参见

+   *AIR for iOS 通用设置，第二章*

+   *监听方向更改*

+   *启用自动旋转*
