# *第 21 章*：理解 Core Data

您的应用几乎完成了！每个屏幕都像您在[*第 9 章*](B17469_09_Final_VK_ePub.xhtml#_idTextAnchor133)*，设置用户界面*中看到的演示应用一样工作。然而，您还需要完成最后一件事。在[*第 19 章*](B17469_19_Final_VK_ePub.xhtml#_idTextAnchor319)*，开始使用自定义 UIControls*中，您实现了**评论表单**屏幕，允许您为特定餐厅输入评论。在前一章中，您实现了**照片滤镜**屏幕，允许您从相机或相册获取照片并为其添加滤镜。但目前还没有保存评论或照片的方法，当应用关闭时它们都会丢失。

在本章中，您将使用 **Core Data** 在您的应用中保存评论和照片。首先，您将了解 Core Data 及其不同组件。接下来，您将为评论和照片创建数据模型，并为您的应用创建相应的模型对象。然后，您将为您的应用设置 Core Data 组件。

您将了解用于保存特定餐厅的评论和照片的机制，使用餐厅标识符。之后，您将更新 `ReviewFormViewController` 和 `PhotoFilterViewController` 类以保存特定餐厅的评论和照片，并修改 `RestaurantDetailViewController` 类以加载和显示特定餐厅的评论。您还将计算并显示该餐厅的整体评分。

最后，您将独立修改 `RestaurantDetailViewController` 类以加载和显示特定餐厅的照片。

到本章结束时，您将了解 Core Data 的工作原理。您还将能够设置 Core Data 组件，并使用数据管理类在您的应用和 Core Data 组件之间建立接口。您还将学会使用 Core Data 保存和加载评论和照片，您将在自己的应用中实现这些功能。

本节将涵盖以下主题：

+   介绍 Core Data

+   为您的应用实现 Core Data 组件

+   理解保存和加载的工作原理

+   更新 `ReviewFormViewController` 类以保存评论

+   更新 `PhotoFilterViewController` 类以保存照片

+   在 **餐厅详情** 屏幕中显示保存的评论和照片

+   计算餐厅的整体评分

# 技术要求

您将继续在上一章中修改的 `LetsEat` 项目上工作。

本章的完成 Xcode 项目位于本书代码包的 `Chapter21` 文件夹中，可以在此处下载：

[https://github.com/PacktPublishing/iOS-15-Programming-for-Beginners-Sixth-Edition](https://github.com/PacktPublishing/iOS-15-Programming-for-Beginners-Sixth-Edition)

观看以下视频，了解代码的实际应用：

[https://bit.ly/3o81yKK](https://bit.ly/3o81yKK)

让我们先了解核心数据组件及其工作原理。

# 介绍核心数据

Core Data是苹果将应用数据保存到设备的一种机制。它提供持久性、撤销/重做、后台任务、视图同步、版本控制和迁移。你可以使用Xcode的数据模型编辑器定义数据类型和关系，Core Data将自动生成数据类型的类定义。然后，Core Data可以根据类定义创建和管理对象实例。

重要提示

你可以通过这个链接了解更多关于核心数据的信息：[https://developer.apple.com/documentation/coredata](https://developer.apple.com/documentation/coredata)。

Core Data提供了一套称为Core Data堆栈的类，用于管理和持久化对象实例，如下所示：

+   `NSManagedObjectModel`

    描述应用类型，包括它们的属性和关系。

+   `NSManagedObject`

    一个用于根据`NSManagedObjectModel`中的数据实现应用类型实例的类。

+   `NSManagedObjectContext`

    跟踪应用类型实例的变化。

+   `NSPersistentStoreCoordinator`

    从存储中保存和检索应用类型实例。

+   `NSPersistentContainer`

    同时设置模型、上下文和存储协调器。

    重要提示

    你可以通过这个链接了解更多关于Core Data堆栈的信息：[https://developer.apple.com/documentation/coredata/core_data_stack](https://developer.apple.com/documentation/coredata/core_data_stack)。

在下一节中，你将实现应用所需的核心数据组件以保存评论或照片。

# 实现应用的核心数据组件

在实现应用的核心数据组件之前，让我们思考一下保存评论或照片需要做什么。

想象一下你正在使用Microsoft Word保存评论或照片。你首先创建一个新的Word文档模板，其中包含评论或照片的相关字段。然后，你根据模板创建新的Word文档并填写数据。你进行必要的更改，比如更改评论的文本，或者更改应用于照片的效果。在这个阶段，你还没有保存文件。当你对你的文档满意时，你将其保存到电脑的硬盘上。下次你想查看你的评论或照片时，你会在硬盘上搜索相关文档，双击它以在Word中打开，以便再次查看。

现在你已经了解了需要做什么，让我们回顾实现它所需的步骤。首先，你需要为评论或照片创建一个数据模型。你通过在Xcode的数据模型编辑器中创建**实体**来完成此操作，这些实体类似于Microsoft Word模板。实体可以具有**属性**，这些属性类似于Microsoft Word模板中的字段。

Xcode 可以从这个数据模型创建一个 `NSManagedObjectModel` 类。然后，Core Data 将使用这个 `NSManagedObjectModel` 类来创建 `NSManagedObject` 实例，类似于 Microsoft Word 模板被用来创建 Microsoft Word 文件。

这些 `NSManagedObject` 实例被放置在一个 `NSManagedObjectContext` 实例中，你的应用程序可以访问它们，类似于在 Microsoft Word 中打开 Microsoft Word 文件。然后，当你调用 `NSManagedObject` 实例时，你可以像在 Microsoft Word 中编辑 Microsoft Word 文档一样修改它们。

当你完成审查或照片后，`NSManagedObjectContext` 实例中的 `NSManagedObject` 实例将被保存到你的 iOS 设备上的一个文件中，称为 **持久化存储**。这类似于当你完成 Word 文档时将 Word 文档保存到硬盘上。

`NSPersistentStoreCoordinator` 实例管理持久化存储和 `NSManagedObjectContext` 实例之间的信息流。

你将使用 `NSPersistentContainer` 类来为你的应用程序创建 `NSManagedObjectModel`、`NSManagedObjectContext` 和 `NSPersistentStoreCoordinator` 的实例。

重要提示

你可以在此链接中了解更多关于如何设置 Core Data 栈的信息：[https://developer.apple.com/documentation/coredata/setting_up_a_core_data_stack](https://developer.apple.com/documentation/coredata/setting_up_a_core_data_stack)。

在下一节中，你将使用 Xcode 的数据模型编辑器创建表示审查或照片的实体和属性。

## 创建数据模型

目前，当你点击 **保存** 在 **审查表单** 屏幕时，你输入的字段中的数据只是打印到调试区域，而在 **照片滤镜** 屏幕上点击 **保存** 不会做任何事情。第一步是为存储来自 **审查表单** 屏幕的数据和来自 **照片滤镜** 屏幕的照片的对象创建类定义。你将使用 Xcode 的数据模型编辑器创建用于审查和照片的实体，Xcode 将自动生成类定义。让我们首先创建审查的实体。按照以下步骤操作：

1.  在项目导航器中右键点击 `Misc` 文件夹并选择 `Core Data`。

1.  右键点击 `Core Data` 组并选择 **新建文件**。

1.  在过滤器字段中输入 `data`，并选择 **数据模型**。点击 **下一步**：![图 21.1：选择数据模型模板

    ](img/Figure_21.01_B17469.jpg)

    图 21.1：选择数据模型模板

1.  将文件命名为 `LetsEatModel` 并点击 **创建**。数据模型编辑器将在编辑区域显示：![图 21.2：显示数据模型编辑器的编辑区域

    ](img/Figure_21.02_B17469.jpg)

    图 21.2：显示数据模型编辑器的编辑区域

1.  点击 **添加实体** 按钮：![图 21.3：添加实体按钮

    ](img/Figure_21.03_B17469.jpg)

    图 21.3：添加实体按钮

1.  **实体**出现在 **ENTITIES** 部分。此实体的 **属性**出现在实体的右侧：![图 21.4：选择实体的数据模型编辑器

    ![图片](img/Figure_21.04_B17469.jpg)

    图 21.4：选择实体的数据模型编辑器

1.  双击 `Review`：![图 21.5：将实体重命名为 Review 的数据模型编辑器

    ![图片](img/Figure_21.05_B17469.jpg)

    图 21.5：将实体重命名为 Review 的数据模型编辑器

1.  点击 `name` 和 `String`：![图 21.6：显示 Review 实体的 name 属性的数据模型编辑器

    ![图片](img/Figure_21.06_B17469.jpg)

    图 21.6：显示 Review 实体的 name 属性的数据模型编辑器

1.  你还需要为 `restaurantID` 中的每个字段创建一个属性，以便将评论与餐厅关联起来。添加以下属性和类型：![图 21.7：要添加到 Review 实体的属性

    ![图片](img/Figure_21.07_B17469.jpg)

    图 21.7：要添加到 Review 实体的属性

    当创建 `Review` 实例时，`date` 将自动设置。

1.  检查完成时，你的 `Review` 实体的属性看起来应该像这样：![图 21.8：Review 实体的属性

    ![图片](img/Figure_21.08_B17469.jpg)

    图 21.8：Review 实体的属性

1.  添加一个额外的属性，`uuid`，类型为 `UUID`。这被用作每个 `Review` 实例的键值。点击数据模型检查器，在 `Review` 实例下必须有一个键值。![图 21.9：显示 uuid 属性可选未勾选的数据模型检查器

    ![图片](img/Figure_21.09_B17469.jpg)

    图 21.9：显示 uuid 属性可选未勾选的数据模型检查器

1.  添加第二个实体，称为 `RestaurantPhoto`，具有以下属性。Core Data 无法存储 `UIImage` 对象，因此当您需要将 `photo` 属性设置为 `UIImage` 对象以在您的应用程序中显示时，`photo` 属性的类型被设置为 `UIImage` 对象。当创建 `RestaurantPhoto` 实例时，`date` 将自动设置。您将使用 `restaurantID` 将照片与餐厅关联起来，并使用 `uuid` 作为键值：![图 21.10：RestaurantPhoto 实体的属性

    ![图片](img/Figure_21.10_B17469.jpg)

    图 21.10：RestaurantPhoto 实体的属性

1.  对于 `uuid`，不要忘记在数据模型检查器中取消选择 **Optional**：

![图 21.11：显示 uuid 属性可选未勾选的数据模型检查器

![图片](img/Figure_21.11_B17469.jpg)

图 21.11：显示 uuid 属性可选未勾选的数据模型检查器

您已经完成了为您的应用程序创建所需的实体。构建您的应用程序。Xcode 将自动创建 `Review` 和 `RestaurantPhoto` 实体的类文件，但它们在项目导航器中不可见。为了更容易地使用它们，您将为每个实体创建一个模型对象，从下一节的 `ReviewItem` 开始。

## 创建 ReviewItem

您已使用 Xcode 的数据模型编辑器创建了两个实体来存储评论和照片数据。然后，Xcode 将从数据模型自动生成两个 `NSManagedObject` 类定义，`Review` 和 `RestaurantPhoto`，但您在项目导航器中看不到它们。

您将创建两个模型对象，`ReviewItem` 和 `RestaurantPhotoItem`，它们将与 `Review` 和 `RestaurantPhoto` 实例协同工作。现在让我们创建 `ReviewItem`。按照以下步骤操作：

1.  在项目导航器中右键点击 `ReviewForm` 文件夹，并选择 `Model`。

1.  在 `ReviewForm` 文件夹内的 `Model` 文件夹中右键点击，并选择 **New File**。

1.  **iOS** 应已选中。选择 **Swift File** 然后点击 **Next**。

1.  将此文件命名为 `ReviewItem`。点击后，`ReviewItem` 文件将出现在项目导航器中。

1.  按照以下方式修改文件：

    [PRE0]

如您所见，`ReviewItem` 结构的属性与 `Review` 实体的属性相同。初始化器创建一个 `ReviewItem` 实例，并将 `Review` 的属性映射到 `ReviewItem` 实例的属性上。

在下一节中，您将创建第二个模型对象 `RestaurantPhotoItem`，它将是 `RestaurantPhoto` 实体的模型对象。

## 创建 `RestaurantPhotoItem`

创建 `RestaurantPhotoItem` 的过程与创建 `ReviewItem` 类似。按照以下步骤操作：

1.  在 `PhotoFilter` 文件夹内的 `Model` 文件夹中右键点击，并选择 **New File**。

1.  **iOS** 应已选中。选择 **Swift File** 然后点击 **Next**。

1.  将此文件命名为 `RestaurantPhotoItem`。点击 **Create**。

1.  按照以下方式修改文件：

    [PRE1]

现在，您已经声明并定义了 `ReviewItem` 和 `RestaurantPhotoItem`，让我们在下一节创建一个 Core Data 管理器，该管理器将为您的应用设置 Core Data 组件。

## 创建 Core Data 管理器

到目前为止，Xcode 已经从数据模型自动生成了 `Review` 和 `RestaurantData` 类定义，并且您已经声明并定义了相应的模型对象，`ReviewItem` 和 `RestaurantPhotoItem`。现在您将创建一个 `CoreDataManager` 类，该类将为您的应用设置 Core Data 组件。按照以下步骤操作：

1.  在 `Misc` 文件夹内的 `Core Data` 文件夹中右键点击，并选择 **New File**。

1.  **iOS** 应已选中。选择 **Swift File** 然后点击 **Next**。

1.  将此文件命名为 `CoreDataManager`。点击后，`CoreDataManager` 文件将出现在项目导航器中。

1.  在 `import` 语句后添加以下代码：

    [PRE2]

    这使您能够访问 Core Data 库。

1.  添加以下代码以声明和定义 `CoreDataManager` 结构：

    [PRE3]

    这将创建并初始化 `NSManagedObjectModel`、`NSPersistentStoreCoordinator` 和 `NSManagedObjectContext` 的实例。

1.  要创建一个将在整个应用中可用的 `CoreDataManager` 结构实例，请在项目导航器中点击 `AppDelegate` 文件，并在闭合花括号后添加以下代码：

    [PRE4]

接下来，您将添加创建 `Review` 和 `RestaurantPhoto` 实例的方法，使用 `ReviewItem` 和 `RestaurantPhotoItem` 实例填充它们，并将它们保存到持久存储中。按照以下步骤操作：

1.  在项目导航器中点击 `CoreDataManager` 文件。在初始化器之后添加以下代码以实现 `addReview(_:)` 方法：

    [PRE5]

    此方法接受一个 `ReviewItem` 实例作为参数，并从 `NSManagedObjectContext` 实例中获取一个空的 `Review` 实例。`ReviewItem` 实例的属性被分配给 `Review` 实例的属性，并调用 `save()` 方法将 `NSManagedObjectContext` 实例的内容保存到持久存储中。请注意，您将看到错误，因为您尚未实现 `save()` 方法。现在忽略此错误。

1.  在 `addReview(_:)` 方法之后添加以下代码以实现 `addPhoto(_:)` 方法：

    [PRE6]

    此方法与 `addReview(_:)` 类似。它接受一个 `RestaurantPhotoItem` 实例作为参数，并从 `NSManagedObjectContext` 实例中获取一个空的 `RestaurantPhoto` 实例。`RestaurantPhotoItem` 实例的属性被分配给 `RestaurantPhoto` 实例的属性，并调用 `save()` 方法将 `NSManagedObjectContext` 实例的内容保存到持久存储中。请注意，您将看到错误，因为您尚未实现 `save()` 方法。同样，现在忽略此错误。

1.  通过在最后的括号之前添加以下代码来实现 `save()` 方法：

    [PRE7]

    此 `do-catch` 块将 `NSManagedObjectContext` 实例的内容保存到持久存储中。如果保存不成功，将在调试区域打印错误消息。

当您想要从持久存储中检索评论和照片时，您将使用 `restaurantID` 作为标识符来获取特定餐厅的评论和照片。现在让我们实现所需的这些方法。在 `addPhoto(_:)` 方法之后添加以下代码以实现 `fetchReviews(by:)` 和 `fetchPhotos(by:)` 方法：

[PRE8]

让我们分解一下，从 `fetchReviews(by:)` 开始：

[PRE9]

这获取了对 `NSManagedObjectContext` 实例的引用。

[PRE10]

这从持久存储中创建一个 `Review` 实例。

[PRE11]

这将创建一个具有指定 `restaurantID` 的 `Review` 实例。

[PRE12]

这将创建一个数组，`reviewItems`，您将使用它来存储获取请求的结果。

[PRE13]

这按日期对获取请求的结果进行排序，最近的条目排在前面。

[PRE14]

这设置了获取请求的谓词。

[PRE15]

此 `do-catch` 块执行获取请求并将结果放置在 `items` 数组中。如果失败，您的应用程序将崩溃，并在调试区域打印错误消息。

`fetchPhotos(by:)` 与 `fetchReview(by:)` 的工作方式相同，但返回一个 `RestaurantPhotoItems` 实例的数组。

您已经创建了一个 `CoreDataManager` 类，用于向持久存储中添加数据并检索数据。构建并运行您的应用以测试错误。它应该与之前一样工作。

您已经在您的应用中实现了 Core Data 的所有组件。接下来，您将配置 `RestaurantDetailViewController` 以使用 Core Data 在 **餐厅详情** 屏幕中显示评论和照片。您将从学习 **餐厅详情** 屏幕如何显示特定餐厅的评论和照片开始。

提示

由于这是一个较长的章节，您可能希望在这里休息一下。

# 理解保存和加载的工作原理

让我们回顾一下您到目前为止所做的工作。您已经使用数据模型编辑器创建了 `Review` 和 `RestaurantPhoto` 实体，并为它们创建了相应的模型对象，分别命名为 `ReviewItem` 和 `RestaurantPhotoItem`。您创建了 `CoreDataManager` 类来从持久存储中添加和获取 `Review` 和 `RestaurantPhoto` 实例。`CoreDataManager` 类使用餐厅标识符将评论和餐厅照片与特定餐厅关联起来，但它从哪里来？

打开您项目中的 `Misc` 文件夹，然后打开 `JSON` 文件夹。如果您点击其中的任何 JSON 文件，您会看到每个餐厅都有一个唯一的数字标识符。例如，The Tap Trailhouse 餐厅的标识符是 `145237`，如图下所示：

![Figure 21.12: Editor area showing contents for Boston.json

](img/Figure_21.12_B17469.jpg)

图 21.12：编辑区域显示 Boston.json 的内容

当您将餐厅照片和评论保存到持久存储中时，您将连同此标识符一起保存它们。然后，当在 `RestaurantDetailViewController` 中显示特定餐厅时，将使用一个 `ReviewDataManager` 实例来检索该餐厅的评论和餐厅照片，并在集合视图中显示它们，如图中所示：

![Figure 21.13: iOS Simulator showing Restaurant Detail screen with reviews and restaurant photos

](img/Figure_21.13_B17469.jpg)

图 21.13：iOS 模拟器显示带有评论和餐厅照片的餐厅详情屏幕

如果没有评论或照片，您将使用 `NoDataView` 来通知用户没有评论或照片，如图中所示：

![Figure 21.14: iOS Simulator showing Restaurant Detail screen without reviews or restaurant photos

](img/Figure_21.14_B17469.jpg)

图 21.14：iOS 模拟器显示没有评论或餐厅照片的餐厅详情屏幕

`RestaurantItem` 有一个属性，`restaurantID`，用于存储餐厅标识符。当 `RestaurantDataManager` 加载 JSON 文件并创建 `RestaurantItem` 实例数组时，每个餐厅的标识符从 JSON 文件中获取并存储在 `restaurantID` 属性中。

在下一节中，您将更新 `ReviewFormViewController` 以将带有餐厅标识符的评论保存到持久存储中。

# 更新 ReviewFormViewController 类以保存评论

当点击 **保存** 按钮时，`onSaveTapped(_:)` 方法将评论保存到持久存储中。按照以下步骤操作：

1.  在项目导航器中点击 `ReviewFormViewController` 文件。在输出声明之前向 `ReviewFormViewController` 类添加以下属性以存储餐厅标识符：

    [PRE16]

1.  创建一个 `private` 扩展，将 `onSaveTapped(_:)` 方法移动到其中，并按以下方式修改：

    [PRE17]

注意，目前没有机制将餐厅标识符传递给 `ReviewFormViewController`。在下一节中，你将看到如何从 `RestaurantDetailViewController` 获取餐厅标识符并将其传递给 `ReviewFormViewController`。

## 将 RestaurantID 传递给 ReviewFormViewController 实例

`ReviewFormViewController` 如何获取该标识符？你必须从 `RestaurantDetailViewController` 将标识符值传递给 `ReviewFormViewController`，以便它可以保存带有该餐厅标识符的评论。正如你在 [*第 17 章*](B17469_17_Final_VK_ePub.xhtml#_idTextAnchor248)*，入门 JSONFiles* 中所做的那样，你将使用转场标识符来确定哪个转场正在发生，然后实现方法在两个视图控制器之间传递标识符值。按照以下步骤操作：

1.  打开 `RestaurantDetail` 故事板文件，并选择用于转到 **ReviewForm** 场景的转场：![图 21.15：显示 Restaurant Detail 和 Review Form 屏幕之间转场的编辑区域

    ](img/Figure_21.15_B17469.jpg)

    图 21.15：显示 Restaurant Detail 和 Review Form 屏幕之间转场的编辑区域

1.  在属性检查器中设置 `showReview` 并按 *Enter* 键：![图 21.16：属性检查器，标识符设置为 showReview

    ](img/Figure_21.16_B17469.jpg)

    图 21.16：属性检查器，标识符设置为 showReview

1.  在项目导航器中点击 `RestaurantDetailViewController` 文件。在 `viewDidLoad()` 之后添加以下代码以实现 `prepare(for:sender:)` 方法：

    [PRE18]

    `prepare(for:sender:)` 方法检查是否具有 `showReview` 转场标识符。如果有，则在从 `showReview(segue:)` 转换之前执行 `showReview(segue:)` 方法。由于 `showReview(segue:)` 方法尚未实现，你将在下一节中添加它。

1.  在 `private` 扩展中添加 `showReview(segue:)` 方法，在 `createRating()` 方法之前：

    [PRE19]

    这将 `ReviewFormViewController` 的 `restaurantID` 属性设置为所选餐厅的标识符。

1.  在项目导航器中点击 `ReviewFormViewController` 文件。在 `viewDidLoad()` 方法内部添加以下代码以将餐厅标识符打印到调试区域：

    [PRE20]

构建并运行你的项目，设置位置，并点击 **All**。点击一个餐厅，然后点击 **添加评论** 按钮。在 **评论表单** 屏幕中输入评论并点击 **保存**：

![图 21.17：iOS 模拟器显示 Review Form 屏幕的保存按钮

![图片](img/Figure_21.17_B17469.jpg)

图 21.17：iOS 模拟器显示审查表单屏幕的 Save 按钮

餐厅标识符将出现在调试区域：

![图 21.18：调试区域显示餐厅标识符

![图片](img/Figure_21.18_B17469.jpg)

图 21.18：调试区域显示餐厅标识符

您已成功将餐厅标识符从 `RestaurantDetailViewController` 传递到 `ReviewFormViewController`。现在，让我们为照片做同样的事情。在下一节中，您将更新 `PhotoFilterViewController` 以在点击 **Save** 按钮时将带有餐厅标识符的照片保存到持久存储中。

# 更新 PhotoFilterViewController 类以保存照片

使 `PhotoFilterViewController` 类能够将照片保存到持久存储的代码与您在 `ReviewFormViewController` 类中为保存评论所实现的代码类似。现在，您将更新 `PhotoFilterViewController` 类以在点击 **Save** 按钮时保存照片。按照以下步骤操作：

1.  在项目导航器中点击 `PhotoFilterViewController` 文件。在 `initialize()` 方法之后的 `private` 扩展内添加以下方法：

    [PRE21]

    请记住，`mainImageView` 是 `saveSelectedPhoto()` 方法中大型图像视图的输出。首先检查 `mainImageView` 的 `image` 属性是否已设置。如果是，则将图像分配给 `mainImage`。接下来，创建一个 `RestaurantPhotoItem` 实例并将其分配给 `restPhotoItem`，并将当前日期分配给 `restPhotoItem` 实例的 `date` 属性。使用 `mainImage` 实例的 `preparingThumbnail(of:)` 方法创建图像的较小版本，并将其分配给 `restPhotoItem` 实例的 `photo` 属性。之后，将 `restPhotoItem` 实例的 `restaurantID` 属性设置为所选餐厅的标识符。最后，调用 `CoreDataManager.shared.addPhoto(_:)` 方法将照片保存到持久存储，并关闭 **Photo Filter** 屏幕。

1.  您需要在 `onPhotoTapped(_:)` 方法之后的 `private` 扩展中触发此方法：

    [PRE22]

    此方法将在稍后连接到 **Save** 按钮。

1.  将 `onSaveTapped(_:)` 方法分配给 `PhotoFilter` 场景文件，并点击 **Photo Filter View Controller** 图标在 **Photo Filter View Controller Scene** 中。打开连接检查器。从 **onSaveTapped** 动作拖动到 **Save** 按钮：

![图 21.19：连接检查器显示 onSaveTapped: 被分配到 Save 按钮

![图片](img/Figure_21.19_B17469.jpg)

图 21.19：连接检查器显示 onSaveTapped: 被分配到 Save 按钮

在您能够保存之前，您需要将餐厅标识符传递给 `PhotoFilterViewController`。正如您之前所做的那样，您将使用 segue 标识符来确定哪个 segue 正在发生，然后实现方法在两个视图控制器之间传递标识符值。按照以下步骤操作：

1.  在项目导航器中点击`RestaurantDetail`故事板文件，并选择用于转到**相片滤镜**屏幕的切换：![图21.20：编辑区域显示餐厅详情和相片滤镜屏幕之间的切换已选中]

    ![img/Figure_21.20_B17469.jpg]

    图21.20：编辑区域显示餐厅详情和相片滤镜屏幕之间的切换已选中

1.  在属性检查器中设置`showPhotoFilter`并按*回车键*：![图21.21：属性检查器，标识符设置为显示相片滤镜]

    ![img/Figure_21.21_B17469.jpg]

    图21.21：属性检查器，标识符设置为显示相片滤镜

1.  在项目导航器中点击`RestaurantDetailViewController`文件。更新`prepare(for:sender:)`方法，如下所示：

    [PRE23]

1.  在你的`private`扩展内部`showReview()`方法之后添加`showPhotoFilter(segue:)`方法：

    [PRE24]

    这将`PhotoFilterViewController`的`restaurantID`属性设置为所选餐厅的标识符。

构建并运行你的项目，设置位置，并点击**所有**。点击一个餐厅，然后点击**添加照片**按钮。选择一张照片，应用滤镜，然后点击**保存**：

![图21.22：iOS模拟器显示带有已选保存按钮的相片滤镜屏幕]

![img/Figure_21.22_B17469.jpg]

图21.22：iOS模拟器显示带有已选保存按钮的相片滤镜屏幕

照片将被保存到持久存储中，并且你将返回到**餐厅详情**屏幕。

在这一点上，你可以保存评论和照片。太棒了！在下一节中，你将添加代码从持久存储中加载评论和照片以在**餐厅详情**屏幕上显示。

# 在餐厅详情屏幕上显示已保存的评论和照片

在`RestaurantDetail.storyboard`中，你会看到集合视图已经设置为在静态表格单元格中显示照片和评论。你所需要做的就是实现用于显示评论的相应视图控制器。你将从用于显示评论的视图和集合视图单元格开始。按照以下步骤操作：

1.  在项目导航器中右键单击`LetsEat`文件夹，并选择`Reviews`。

1.  右键单击文件夹并选择**新建文件**。

1.  **iOS**应该已经选中。选择**Cocoa Touch 类**，然后点击**下一步**。

1.  按照以下方式配置文件：

    `ReviewCell`

    `UICollectionViewCell`

    `Swift`

    点击**下一步**。

1.  点击项目导航器中出现的`ReviewCell`文件。在花括号之间输入以下代码：

    [PRE25]

`ReviewCell`现在具有集合视图单元格中所有输出属性。让我们接下来创建`ReviewsViewController`。按照以下步骤操作：

1.  右键单击`Reviews`文件夹，并选择**新建文件**。

1.  **iOS**应该已经选中。选择**Cocoa Touch 类**，然后点击**下一步**。

1.  按照以下方式配置文件：

    `ReviewsViewController`

    `UIViewController`

    `Swift`

    点击**下一步**。

1.  点击项目导航器中出现的`ReviewsViewController`文件。按照以下方式修改此文件：

    [PRE26]

1.  添加一个包含以下代码的`private`扩展，如图所示：

    [PRE27]

    `private`扩展包含`initialize()`和`setupCollectionView()`方法的实现。`initialize()`只是调用`setupCollectionView()`。`setupCollectionView()`用于配置集合视图的流和间距，与您之前编写的代码类似。

1.  在`setupCollectionView()`之后添加以下方法以实现`checkReviews()`：

    [PRE28]

    此方法将检索指定餐厅标识符的所有餐厅评论。让我们分解一下：

    [PRE29]

    此语句将`RestaurantDetailViewController`分配给一个临时常量`viewController`。

    [PRE30]

    此语句将显示在`restaurantID`中的餐厅的餐厅标识符分配给此语句。

    [PRE31]

    此语句从持久存储中获取与给定`restaurantID`匹配的评论数组，并将其分配给`reviewItems`。

    [PRE32]

    如果有关于这家餐厅的评论，则将集合视图的背景视图设置为`nil`；否则，创建一个`NoDataView`实例，将`title`和`desc`属性分别设置为`"Reviews"`和`"There are currently no reviews"`，并将其分配给集合视图的背景视图。

    [PRE33]

    这段代码告诉集合视图在屏幕上重新绘制自己。

1.  通过添加以下扩展来实现集合视图的数据源方法：

    [PRE34]

    这与您之前所做的工作类似。集合视图中要显示的单元格数量与`reviewItems`数组中的项目数量相同。您使用相应的`ReviewItem`实例的属性设置每个单元格的内容。

1.  通过添加以下扩展来实现集合视图的流布局代理方法：

    [PRE35]

    此方法返回要显示的集合视图单元格的大小。如果`reviewItems`数组中只有一个项目，则单元格的宽度设置为集合视图的宽度—14点；否则，它设置为集合视图的宽度—21点。高度设置为`200`点。

`ReviewsViewController`现在已完成。现在，您将完成`RestaurantDetail`故事板文件的实现。按照以下步骤操作：

1.  在项目导航器中点击`RestaurantDetail`故事板文件。选择`ReviewsViewController`，然后按*Return*：![图21.23：类设置为ReviewsViewController的Identity inspector

    ](img/Figure_21.23_B17469.jpg)

    图21.23：类设置为ReviewsViewController的Identity inspector

    场景名称将更改为**Reviews View Controller Scene**。

1.  选择`ReviewCell`，然后按*Return*：![图21.24：类设置为ReviewCell的Identity inspector

    ](img/Figure_21.24_B17469.jpg)

    图21.24：类设置为ReviewCell的Identity inspector

1.  在文档大纲中选择视图。点击Identity inspector按钮，设置`RatingsView`，然后按*Return*：![图21.25：类设置为RatingsView的Identity inspector

    ](img/Figure_21.25_B17469.jpg)

    图21.25：类设置为RatingsView的Identity inspector

1.  如果尚未设置，选择`reviewCell`：![图 21.26：属性检查器，将标识符设置为reviewCell

    ![图片](img/Figure_21.26_B17469.jpg)

    图 21.26：属性检查器，将标识符设置为reviewCell

1.  点击连接检查器。如果尚未设置，从`dateLabel`出口拖动到显示的**标签**：![图 21.27：连接检查器显示dateLabel出口

    ![图片](img/Figure_21.27_B17469.jpg)

    图 21.27：连接检查器显示dateLabel出口

1.  如果尚未设置，从`nameLabel`出口拖动到显示的**标签**：![图 21.28：连接检查器显示nameLabel出口

    ![图片](img/Figure_21.28_B17469.jpg)

    图 21.28：连接检查器显示nameLabel出口

1.  如果尚未设置，从`reviewLabel`出口拖动到显示的**标签**：![图 21.29：连接检查器显示reviewLabel出口

    ![图片](img/Figure_21.29_B17469.jpg)

    图 21.29：连接检查器显示reviewLabel出口

1.  如果尚未设置，从`titleLabel`出口拖动到显示的**标签**：![图 21.30：连接检查器显示titleLabel出口

    ![图片](img/Figure_21.30_B17469.jpg)

    图 21.30：连接检查器显示titleLabel出口

1.  如果尚未设置，从`ratingsView`出口拖动到显示的评分视图：![图 21.31：连接检查器显示ratingsView出口

    ![图片](img/Figure_21.31_B17469.jpg)

    图 21.31：连接检查器显示ratingsView出口

1.  选择如所示到**集合视图**的`collectionView`出口，如果尚未设置：![图 21.32：连接检查器显示collectionView出口

    ![图片](img/Figure_21.32_B17469.jpg)

    图 21.32：连接检查器显示collectionView出口

1.  选择到**评论视图控制器**图标的`delegate`和`dataSource`出口：

![图 21.33：连接检查器显示dataSource和delegate出口

![图片](img/Figure_21.33_B17469.jpg)

图 21.33：连接检查器显示dataSource和delegate出口

构建并运行您的应用。您应该看到之前添加的评论：

![图 21.34：iOS 模拟器显示包含评论的餐厅详情屏幕

![图片](img/Figure_21.34_B17469.jpg)

图 21.34：iOS 模拟器显示包含评论的餐厅详情屏幕

用于显示评论的集合视图和集合视图单元格的视图控制器实现现在已完成，您的应用现在可以显示之前使用**评论表单**屏幕输入的评论。如果您有多个评论，您可以左右滑动以查看每个评论。由于每个评论都有一个评分，您可以使用它们来计算并添加一个餐厅的整体评分。让我们修改应用以执行此操作。

# 计算餐厅的整体评分

要执行此操作，请使用`CoreDataManager`。按照以下步骤操作：

1.  在项目导航器中（在`Misc`文件夹中的`Core Data`文件夹内）点击`CoreDataManager`文件。在`addReview(_:)`方法之前添加以下方法：

    [PRE36]

    在这个方法中，从持久存储中检索特定餐厅的所有评论，并分配给`reviews`。`reduce()`方法接受一个闭包，用于将所有评论评分相加。最后，计算平均评分值并返回。

    重要信息

    你可以在以下链接中了解更多关于`reduce()`方法的信息：[https://developer.apple.com/documentation/swift/array/2298686-reduce](https://developer.apple.com/documentation/swift/array/2298686-reduce)。

1.  在项目导航器中（位于`RestaurantDetail`文件夹内）点击`RestaurantDetailViewController`文件。更新`createRating()`方法，如下所示：

    [PRE37]

1.  当用户在`viewDidLoad()`方法中添加新的评论时，也需要更新总体评分：

    [PRE38]

    这将在关闭**评论表单**屏幕并重新出现**餐厅详情**屏幕时重新计算评分。

构建并运行你的项目，你现在应该能看到有评论的餐厅的总体评分，以及相应的星级评分，如下所示：

![Figure 21.35: iOS Simulator showing Restaurant Detail screen with overall ratings](img/Figure_21.35_B17469.jpg)

![Figure 21.35: iOS Simulator showing Restaurant Detail screen with overall ratings](img/Figure_21.35_B17469.jpg)

图21.35：iOS模拟器显示包含总体评分的餐厅详情屏幕

还有一件事要做，那就是添加图片评论。你的挑战是添加图片评论，并在评论集合视图下方显示它们。这样做的方式与你之前添加评论的方式非常相似。本章涵盖了你需要知道的所有内容，如果你遇到困难，请随时使用本章的完整项目文件，这些文件可以在本书的代码包的`Chapter21`文件夹中找到，可从[https://github.com/PacktPublishing/iOS-15-Programming-for-Beginners-Sixth-Edition](https://github.com/PacktPublishing/iOS-15-Programming-for-Beginners-Sixth-Edition)下载。你还可以观看本章的CiA视频，位于[https://bit.ly/3o81yKK](https://bit.ly/3o81yKK)。

# 摘要

在本章中，你学习了Core Data及其不同组件。你为你的应用创建了名为`Review`和`RestaurantPhoto`的数据模型，并为你的应用创建了相应的模型对象，名为`ReviewItem`和`RestaurantPhotoItem`。之后，你实现了`CoreDataManager`来为你的应用设置Core Data组件。

你更新了`ReviewFormViewController`和`PhotoFilterViewController`，将评论和图片与餐厅标识符一起保存到持久存储中。你修改了`RestaurantDetailViewController`，根据餐厅标识符加载特定餐厅的评论，并在集合视图中显示它们。你还计算并显示了该餐厅的总体评分。

最后，你自己修改了`RestaurantDetailViewController`，根据餐厅标识符加载特定餐厅的图片，并在集合视图中显示它们。

你现在对Core Data的工作原理有了基本的了解。你也能够设置Core Data组件，并使用数据管理类在你的应用和Core Data组件之间启用接口。你还知道如何使用Core Data保存和加载评论和照片，你现在将能够在自己的应用中实现这些功能。

你已经完成了漫长旅程的终点，现在你已经完成了构建应用主要功能的工作。所有屏幕都正常工作，评论和照片都得到了持久化。做得太棒了！

这本书的第三部分到此结束。在下一部分，你将了解到苹果在iOS 15中引入的酷炫新功能，以及如何将它们添加到你的应用中，从下一章开始，我们将介绍如何让你的应用为苹果Mac做准备。
