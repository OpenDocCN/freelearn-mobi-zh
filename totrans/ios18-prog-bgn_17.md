# 14

# 开始使用MVC和表格视图

在上一章中，你修改了“期刊列表”屏幕、“添加新期刊条目”屏幕和“期刊条目详情”屏幕，以匹配第10章中显示的应用程序导游，“设置用户界面”。你已经完成了*JRNL*应用的初始UI，这标志着本书**第二部分**的结束。

本章开始本书的**第三部分**，在这一部分中，你将专注于使你的应用工作的代码。在本章中，你将学习**模型-视图-控制器**（**MVC**）设计模式以及应用的不同部分如何相互交互。然后，你将使用沙盒程序以编程方式实现一个表格视图（这意味着使用代码而不是故事板来实现），以了解表格视图是如何工作的。最后，你将回顾在“期刊列表”屏幕上实现的表格视图，以便你可以看到在故事板中实现它和在编程中实现它的区别。

到本章结束时，你将理解**模型-视图-控制器**（**MVC**）设计模式，你将学会如何以编程方式创建表格视图控制器，并且将知道如何使用表格视图代理和数据源协议。

本章将涵盖以下主题：

+   理解**模型-视图-控制器**（**MVC**）设计模式

+   理解表格视图

+   回顾“期刊列表”屏幕

# 技术要求

本章的完成版Xcode沙盒程序和项目位于本书代码包的`Chapter14`文件夹中，可以在此处下载：

[https://github.com/PacktPublishing/iOS-18-Programming-for-Beginners-Ninth-Edition](https://github.com/PacktPublishing/iOS-18-Programming-for-Beginners-Ninth-Edition%0D)

查看以下视频以查看代码的实际效果：

[https://youtu.be/tvnmgByShF4](https://youtu.be/tvnmgByShF4%0D)

创建一个新的沙盒程序，并将其命名为`TableViewBasics`。你可以使用这个沙盒程序在阅读本章时输入和运行所有代码。在你这样做之前，让我们看看**模型-视图-控制器**（**MVC**）设计模式，这是一种在编写iOS应用时常用的方法。

# 理解**模型-视图-控制器**（**MVC**）设计模式

**模型-视图-控制器**（**MVC**）设计模式是构建iOS应用时常用的一个常见方法。MVC将应用分为三个不同的部分：

+   **模型**：这处理数据存储和表示以及数据处理任务。

+   **视图**：这包括屏幕上用户可以与之交互的所有内容。

+   **控制器**：这管理模型和视图之间信息流的流程。

MVC的一个显著特点是视图和模型不会相互交互；相反，所有通信都由控制器管理。

例如，想象您在一家餐厅。您查看菜单并选择您想要的东西。然后，服务员过来，接收您的订单，并将其发送给厨师。厨师准备您的订单，当它完成时，服务员取走订单并将其带给您。在这个场景中，菜单是视图，服务员是控制器，厨师是模型。此外，请注意，您与厨房之间的所有交互都只通过服务员进行；您与厨师之间没有交互。

要了解更多关于 MVC 的信息，请访问 [https://developer.apple.com/library/archive/documentation/General/Conceptual/DevPedia-CocoaCore/MVC.html](https://developer.apple.com/library/archive/documentation/General/Conceptual/DevPedia-CocoaCore/MVC.html)。

要了解 MVC 在 iOS 应用程序中的工作原理，让我们先更多了解一下视图控制器。您将看到实现一个视图控制器所需的内容，这个控制器需要管理用于 Journal List 屏幕的表格视图。

## 探索视图控制器

到目前为止，您已经实现了 `JournalListViewController`，这是一个管理 Journal List 屏幕上表格视图的视图控制器。然而，您还没有学习到您添加到其中的代码是如何工作的，所以现在让我们来看看。

您可能希望重新阅读 *第 11 章*，*构建您的用户界面*，其中您实现了 `JournalListViewController` 类。

当 iOS 应用程序启动时，将加载要显示的第一个屏幕的视图控制器。视图控制器有一个 `view` 属性，并自动加载分配给其 `view` 属性的视图实例。该视图可能有子视图，这些子视图也会被加载。

如果其中一个子视图是表格视图，它将具有 `dataSource` 和 `delegate` 属性。`dataSource` 属性被分配给一个为表格视图提供数据的对象。`delegate` 属性被分配给一个处理与表格视图用户交互的对象。通常，表格视图的视图控制器将被分配给表格视图的 `dataSource` 和 `delegate` 属性。

表格视图将向其视图控制器发送的方法调用在 `UITableViewDataSource` 和 `UITableViewDelegate` 协议中声明。请记住，协议只提供方法声明；这些方法调用的实现是在视图控制器中。然后，视图控制器将从模型对象中获取数据，并将其提供给表格视图。视图控制器还处理用户输入，并根据需要修改模型对象。

在下一节中，我们将更详细地了解表格视图和表格视图协议。

# 理解表格视图

*JRNL* 应用在 Journal List 屏幕上使用表格视图。表格视图通过单列排列的行来展示表格视图单元格。

要了解更多关于表格视图的信息，请访问 [https://developer.apple.com/documentation/uikit/uitableview](https://developer.apple.com/documentation/uikit/uitableview)。

表视图显示的数据通常由视图控制器提供。为表视图提供数据的视图控制器必须遵守`UITableViewDataSource`协议。该协议声明了一系列方法，告诉表视图显示多少个单元格以及每个单元格中显示什么内容。

要了解更多关于`UITableViewDataSource`协议的信息，请访问[https://developer.apple.com/documentation/uikit/uitableviewdatasource](https://developer.apple.com/documentation/uikit/uitableviewdatasource)。

为了启用用户交互，表视图的视图控制器还必须遵守`UITableViewDelegate`协议，该协议声明了一系列在用户与表视图交互时触发的方法。

要了解更多关于`UITableViewDelegate`协议的信息，请访问[https://developer.apple.com/documentation/uikit/uitableviewdelegate](https://developer.apple.com/documentation/uikit/uitableviewdelegate)。

要了解表视图是如何工作的，你将在`TableViewBasics`游乐场中实现一个控制表视图的视图控制器子类。由于游乐场中没有故事板，你不能像前几章那样使用库添加UI元素。相反，你将一切通过编程实现。

你将首先创建`TableViewExampleController`类，这是一个管理表视图的视图控制器的实现。之后，你将创建一个`TableViewExampleController`实例，并在游乐场的实时视图中显示一个表视图。按照以下步骤操作：

1.  打开你创建的`TableViewBasics`游乐场，删除`var`语句，并添加`import PlaygroundSupport`语句。现在你的游乐场应该包含以下内容：

    [PRE0]

第一个`import`语句导入了创建iOS应用的API。第二个语句使游乐场能够显示实时视图，你将使用它来显示表视图。

1.  在`import`语句之后添加以下代码以声明`TableViewExampleController`类：

    [PRE1]

这个类是`UIViewController`的子类，`UIViewController`是苹果提供的一个用于管理屏幕上视图的类。

1.  在`TableViewExampleController`类中，在大括号内添加以下代码以声明一个表视图属性和一个数组属性：

    [PRE2]

`tableView`属性是一个可选属性，它将被分配一个`UITableView`实例。`journalEntries`数组是用于向表视图提供数据的模型对象。

你刚刚声明并定义了`TableViewExampleController`类的初始实现。太棒了！在下一节中，你将学习如何设置表视图显示的单元格数量，如何设置每个单元格的内容，以及如何从表视图中删除一行。

## 遵守`UITableViewDataSource`协议

表视图使用单列排列的行来呈现表视图单元格。然而，在它能够这样做之前，它需要知道要显示多少个单元格以及每个单元格中要放置什么内容。为了向表视图提供这些信息，你将使`TableViewExampleController`类遵守`UITableViewDataSource`协议。

此协议有两个必需的方法：

+   `tableview(_:numberOfRowsInSection:)`由表视图调用，以确定要显示多少个表视图单元格。

+   `tableView(_:cellForRowAt:)`由表视图调用，以确定在每个表视图单元格中显示什么。

`UITableViewDataSource`协议还有一个可选方法，`tableView(_:commit:forRowAt:)`，当用户在行上向左滑动时，表视图会调用此方法。你将使用此方法来处理用户想要删除行时发生的情况。

让我们添加一些代码来使`TableViewExampleController`类遵守`UITableViewDataSource`协议。按照以下步骤操作：

1.  要使`TableViewExampleController`类采用`UITableViewDataSource`协议，在超类名称后输入一个逗号，然后输入`UITableViewDataSource`。你的代码应该看起来像这样：

    [PRE3]

1.  由于你没有实现两个必需的方法，将出现错误。点击错误图标：

![图片](img/B31371_14_01.png)

图14.1：显示错误图标的编辑区域

错误信息表明`TableViewExampleController`类没有遵守`UITableViewDataSource`协议。

1.  点击**修复**按钮以添加使类遵守协议所需的存根：

![图片](img/B31371_14_02.png)

图14.2：错误解释和修复按钮

1.  确认你的代码看起来像这样：

    [PRE4]

1.  在类定义中，惯例规定属性应在任何方法声明之前在顶部声明。重新排列代码，使属性声明在顶部，如下所示：

    [PRE5]

1.  要使表视图为`journalEntries`数组中的每个元素显示一行，请在`tableView(_:numberOfRowsInSection:)`方法定义内的**代码**占位符中单击，并输入`journalEntries.count`。完成的方法应如下所示：

    [PRE6]

`journalEntries.count`返回`journalEntries`数组中的元素数量。由于其中有三个元素，这将使表视图显示三行。

1.  要使表视图在每个单元格中显示日记条目详情，请在`tableView(_:cellForRowAt:)`方法定义内的**代码**占位符中单击，并输入以下内容：

    [PRE7]

让我们分解一下：

[PRE8]

此语句创建一个新的表格视图单元格或重用现有的表格视图单元格，并将其分配给`cell`。想象一下，你需要在表格视图中显示1,000个项目。你不需要包含1,000个表格视图单元格的1,000行——你只需要足够多的来填满屏幕。滚动出屏幕顶部的表格视图单元格可以重用来显示屏幕底部的项目，反之亦然。由于表格视图可以显示多种类型的单元格，你将单元格重用标识符设置为`"cell"`以识别这种特定的表格视图单元格类型。此标识符稍后将注册到表格视图中。

[PRE9]

`indexPath`值用于定位表格视图中的行。第一行有一个包含分区`0`和行`0`的`indexPath`。`indexPath.row`对第一行返回`0`，因此此语句将`journalEntries`数组中的第一个元素分配给`journalEntry`。

[PRE10]

默认情况下，`UITableViewCell`实例可以存储一个图像、一个文本字符串和一个次要文本字符串。你通过使用表格视图单元格的内容配置属性来设置这些属性。此语句检索表格视图单元格样式的默认内容配置并将其分配给一个变量`content`。

[PRE11]

这些语句使用`journalEntry`的详细信息更新`content`，`journalEntry`是一个包含三个元素的数组。第一个元素用于指定分配给`image`属性的图像。第二个元素分配给`text`属性。第三个元素分配给`secondaryText`属性。最后一行将`content`分配给表格视图单元格的`contentConfiguration`属性。

[PRE12]

此语句返回表格视图单元格，然后将其显示在屏幕上。

`tableView(_:cellForRowAt:)`方法为表格视图中的每一行执行。

1.  要处理表格视图单元格删除，在`tableView(_:cellForRowAt:)`的实现之后输入以下代码：

    [PRE13]

这将移除与用户左滑的表格视图单元格对应的`journalEntries`元素，并重新加载表格视图。

`TableViewExampleController`类现在遵循`UITableViewDataSource`协议。在下一节中，你将使其遵循`UITableViewDelegate`协议。

## 遵循`UITableViewDelegate`协议

用户可以点击表格视图单元格来选择它。为了处理用户交互，你将使`TableViewExampleController`类遵循`UITableViewDelegate`协议。你将实现此协议的一个可选方法，即`tableView(_:didSelectRowAt:)`，当用户点击行时，表格视图会调用此方法。按照以下步骤操作：

1.  要使`TableViewExampleController`类采用`UITableViewDelegate`协议，在类声明中`UITableViewDataSource`之后输入一个逗号，然后输入`UITableViewDelegate`。你的代码应该看起来像这样：

    [PRE14]

1.  在`tableView(_:commit:forRowAt:)`的实现之后输入以下代码：

    [PRE15]

此方法将获取被点击行的`journalEntries`数组元素并将其打印到调试区域。

1.  确认您的`TableViewExampleController`类看起来如下：

    [PRE16]

`TableViewExampleController`类现在符合`UITableViewDelegate`协议。

您已完成了`TableViewExampleController`类的实现。在下一节中，您将学习如何创建此类的实例。

## 创建`TableViewExampleController`实例

现在您已经声明并定义了`TableViewExampleController`类，您将编写一个方法来创建其实例。按照以下步骤操作：

1.  在`journalEntries`变量声明之后输入以下代码以声明一个新的方法：

    [PRE17]

这声明了一个新的方法`createTableView()`，您将使用它来创建表格视图的实例并将其分配给`tableView`属性。

1.  在大括号开头之后输入以下代码：

    [PRE18]

这将创建一个新的表格视图实例并将其分配给`tableView`。

1.  在下一行，输入以下代码以将表格视图的`dataSource`和`delegate`属性设置为`TableViewExampleController`实例：

    [PRE19]

表格视图的`dataSource`和`delegate`属性指定包含`UITableViewDataSource`和`UITableViewDelegate`方法实现的对象。

1.  在下一行，输入以下代码以设置表格视图的背景颜色：

    [PRE20]

1.  在下一行，输入以下代码以设置表格视图单元格的标识符为`"cell"`：

    [PRE21]

此标识符将在`tableView(_:cellForRowAt:)`方法中使用，以识别要使用的表格视图单元格的类型。

1.  在下一行，输入以下代码以将表格视图添加为`TableViewExampleController`实例视图的子视图：

    [PRE22]

1.  确认完成的方法如下：

    [PRE23]

现在您必须确定何时调用此方法。视图控制器有一个`view`属性。分配给`view`属性的视图将在视图控制器加载时自动加载。在视图成功加载后，视图控制器的`viewDidLoad()`方法将被调用。您将在`TableViewControllerExample`类中重写`viewDidLoad()`方法以调用`createTableView()`。在`createTableView()`方法之前输入以下代码：

[PRE24]

这设置了实时视图的大小，创建了一个表格视图实例，将其分配给`tableView`，并将其添加为`TableViewExampleController`实例视图的子视图。然后表格视图调用数据源方法以确定要显示多少表格视图单元格以及每个单元格中显示的内容。`tableView(_:numberOfRowsInSection:)`返回`journalEntries`中的元素数量，因此将显示三个表格视图单元格。`tableView(_:cellForRowAt:)`创建单元格，创建一个新的单元格配置，设置单元格配置的属性，并将更新的配置分配给单元格。

确认你的完成后的游乐场看起来像这样：

[PRE25]

现在是时候看到它的实际效果了。按照以下步骤操作：

1.  在游乐场中所有其他代码之后输入以下内容：

    [PRE26]

此命令创建`TableViewExampleController`的实例并在游乐场的实时视图中显示其视图。`createTableView()`方法将创建一个表格视图并将其添加为`TableViewExampleController`实例视图的子视图，它将显示在屏幕上。

1.  要在屏幕上看到表格视图的表示，游乐场的实时视图必须启用。点击调整编辑选项按钮并确认**实时视图**被选中：

![](img/B31371_14_03.png)

图14.3：调整编辑选项菜单，选择“实时视图”

1.  运行你的代码并验证表格视图是否在实时视图中显示三个表格视图单元格：

![](img/B31371_14_04.png)

图14.4：游乐场实时视图显示包含三个表格视图单元格的表格视图

1.  点击一行。该行的期刊条目详情将在调试区域打印出来：

![](img/B31371_14_05.png)

图14.5：调试区域显示期刊条目详情

1.  在一行上向左滑动。将出现一个**删除**按钮：

![](img/B31371_14_06.png)

图14.6：显示删除按钮的表格视图行

1.  点击**删除**按钮从表格视图中移除行：

![](img/B31371_14_07.png)

图14.7：移除一行后的表格视图

你刚刚创建了一个表格视图的视图控制器，创建了一个实例，并在游乐场的实时视图中显示了一个表格视图。做得好！

在下一节中，你将回顾如何在*第11章*，*构建用户界面*中实现的期刊列表屏幕上使用视图控制器。使用本节学到的内容作为参考，你应该能更好地理解它是如何工作的。

# 回顾期刊列表屏幕

记住*第11章*，*构建用户界面*中的`JournalListViewController`类？这是一个管理表格视图的视图控制器示例。请注意，这个类的代码与你的游乐场中的代码非常相似。差异如下：

+   你通过编程在`TableViewExampleController`中创建并分配了表格视图到`tableView`，而不是使用辅助工具。

+   你通过`UITableView(frame:)`以编程方式设置表格视图的尺寸，而不是使用大小检查器和约束。

+   你通过编程将数据源和代理出口连接到视图控制器，而不是使用连接检查器。

+   你通过编程设置重用标识符和UI元素颜色，而不是使用属性检查器。

+   你通过编程将表格视图作为`TableViewExampleController`视图的子视图添加，而不是从库中拖入一个**表格视图**对象。

你可能希望再次打开 *JRNL* 项目并回顾 *第 11 章*，*构建你的用户界面*，以比较使用故事板实现的表格视图实现和你在本章中按编程方式实现的表格视图实现。

# 摘要

在本章中，你详细学习了 MVC 设计模式和表格视图控制器。然后你回顾了在“期刊列表”屏幕上使用的表格视图，并学习了表格视图控制器是如何工作的。

你现在应该已经理解了 MVC 设计模式，如何创建一个表格视图控制器，以及如何使用表格视图数据源和代理协议。这将使你能够为你的应用程序实现表格视图控制器。

到目前为止，你已经为“期刊列表”屏幕设置了视图和视图控制器，但它只显示一列单元格。在下一章中，你将实现“期刊列表”屏幕的模型对象，以便它可以显示期刊条目列表。为此，你将创建用于存储数据并提供给`JournalListViewController`实例的结构，以便它可以在“期刊列表”屏幕上的表格视图中显示。

# 加入我们的 Discord 社区！

与其他用户、专家和作者本人一起阅读这本书。提出问题，为其他读者提供解决方案，通过 Ask Me Anything 会话与作者聊天，等等。扫描二维码或访问链接加入社区。

[https://packt.link/ios-Swift](https://packt.link/ios-Swift%0D)

[![](img/QR_Code2370024260177612484.png)](https://packt.link/ios-Swift%0D)
