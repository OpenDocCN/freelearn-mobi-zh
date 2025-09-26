# 3

# 条件语句和可选

在上一章中，你学习了数据类型、常量、变量和操作。到这一点，你可以编写简单的程序来处理字母和数字。然而，程序并不总是按顺序执行。很多时候，你需要根据条件执行不同的指令。Swift 允许你通过使用 **条件语句** 来做到这一点，你将学习如何使用它们。

你可能还注意到，在上一章中，每个变量或常量都被立即赋予了一个值。如果你需要一个可能最初没有值的变量，你将需要一个方法来创建一个可能或可能没有值的变量。Swift 允许你通过使用 **可选** 来做到这一点，你也将在这章中了解它们。

到本章结束时，你应该能够编写根据不同条件执行不同操作的程序，并处理可能或可能没有值的变量。

以下内容将涵盖：

+   介绍条件语句

+   介绍可选和可选绑定

请花些时间了解可选。对于新手程序员来说，它们可能有些令人畏惧，但正如你将看到的，它们是 iOS 开发的重要组成部分。

# 技术要求

本章的 Xcode 演示场位于本书代码包的 `Chapter03` 文件夹中，可以在此处下载：

[https://github.com/PacktPublishing/iOS-18-Programming-for-Beginners-Ninth-Edition](https://github.com/PacktPublishing/iOS-18-Programming-for-Beginners-Ninth-Edition%0D)

查看以下视频以查看代码的实际运行情况：

[https://youtu.be/f90pTabsOgc](https://youtu.be/f90pTabsOgc%0D)

创建一个新的演示场，并将其命名为 `ConditionalsAndOptionals`。你可以一边阅读一边输入并运行本章中的所有代码。你将从学习条件语句开始。

# 介绍条件语句

有时，你将想要根据特定的条件执行不同的代码块，例如在以下场景中：

+   在酒店中选择不同的房间类型。大房间的价格会更高。

+   在在线商店之间切换不同的支付方式。不同的支付方式会有不同的程序。

+   在快餐店决定要订购什么。每种食品的准备程序都不同。

要做到这一点，你会使用条件语句。在 Swift 中，这是通过使用 `if` 语句（用于单个条件）和 `switch` 语句（用于多个条件）来实现的。

更多有关条件语句的信息，请访问 [https://docs.swift.org/swift-book/documentation/the-swift-programming-language/controlflow](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/controlflow)。

让我们看看在下一节中如何使用 `if` 语句根据条件值执行不同的任务。

## 使用if语句

`if`语句在条件为`true`时执行一段代码，如果条件为`false`，则可选地执行另一段代码。它看起来像这样：

[PRE0]

让我们实现一个`if`语句来观察其效果。想象一下，你正在为一家餐厅编写应用程序。该应用程序将允许你检查餐厅是否营业，搜索餐厅，并检查顾客是否超过饮酒年龄限制。按照以下步骤操作：

1.  要检查餐厅是否营业，将以下代码添加到你的playground中。运行它以创建一个常量，并在常量的值为`true`时执行一个语句：

    [PRE1]

首先，你创建了一个常量`isRestaurantOpen`，并将其赋值为`true`。接下来，你有一个`if`语句，它会检查`isRestaurantOpen`中存储的值。由于值是`true`，`print()`语句被执行，并在Debug区域打印出**Restaurant is open**。

1.  尝试将`isRestaurantOpen`的值改为`false`并再次运行你的代码。由于当前条件是`false`，Debug区域将不会打印任何内容。

1.  你也可以在值为`false`时执行语句。假设顾客搜索的餐厅不在应用程序的数据库中，因此应用程序应显示一条消息表示餐厅未找到。输入以下代码以创建一个常量，并在常量的值为`false`时执行一个语句：

    [PRE2]

常量`isRestaurantFound`被设置为`false`。接下来，检查`if`语句。`isRestaurantFound == false`条件返回`true`，并在Debug区域打印出**Restaurant was not found**。

你也可以使用`!isRestaurantFound`代替`isRestaurantFound == false`来检查条件。

1.  尝试将`isRestaurantFound`的值改为`true`。由于当前条件是`false`，Debug区域将不会打印任何内容。

1.  要在条件为`true`时执行一组语句，在条件为`false`时执行另一组语句，请使用`else`关键字。输入以下代码，该代码检查酒吧中的顾客是否超过饮酒年龄限制：

    [PRE3]

在这里，`drinkingAgeLimit`被赋值为`21`，`customerAge`被赋值为`23`。在`if`语句中，检查`customerAge < drinkingAgeLimit`。由于23 < 21返回`false`，所以执行`else`语句，并在Debug区域打印出**Over age limit**。如果你将`customerAge`的值改为`19`，`customerAge < drinkingAgeLimit`将返回`true`，因此将在Debug区域打印出**Under age limit**。

到目前为止，你只处理了单个条件。如果有多个条件怎么办？这就是`switch`语句的用武之地，你将在下一节中学习它们。

## 使用`switch`语句

要理解`switch`语句，我们先从实现一个带有多个条件的`if`语句开始。想象一下你正在编写交通灯的代码。交通灯有三种可能的颜色——红色、黄色或绿色——并且你希望根据灯的颜色执行不同的操作。为此，你可以嵌套多个`if`语句。按照以下步骤操作：

1.  将以下代码添加到你的游乐场中，以使用多个`if`语句实现交通灯，并运行它：

    [PRE4]

第一个`if`条件`trafficLightColor == "Red"`返回`false`，因此执行`else`语句。第二个`if`条件`trafficLightColor == "Yellow"`返回`true`，所以在调试区域打印出**Caution**，并且不再评估更多的`if`条件。尝试更改`trafficLightColor`的值以查看不同的结果。

这里使用的代码是有效的，但读起来有点困难。在这种情况下，使用`switch`语句会更加简洁且易于理解。`switch`语句看起来是这样的：

[PRE5]

值会被检查并与某个`case`匹配，然后执行该`case`的代码。如果没有任何一个`case`匹配，则执行`default`中的代码。

1.  这是如何将前面显示的`if`语句写成`switch`语句的方法。输入以下代码并运行：

    [PRE6]

与之前的版本相比，这里的代码更容易阅读和理解。`trafficLightColor`中的值是`"Yellow"`，所以`case "Yellow":`被匹配，并在调试区域打印出**Caution**。尝试更改`trafficLightColor`的值以查看不同的结果。

关于`switch`语句有两点需要注意：

+   Swift中的`switch`语句默认不会从每个`case`的底部跌落到下一个`case`。在前面显示的示例中，一旦`case "Yellow":`被匹配，`case "Red"`、`case "Green"`和`default:`将不会执行。

+   `switch`语句必须覆盖所有可能的`case`。在前面显示的示例中，任何除了`"Red"`、`"Yellow"`或`"Green"`之外的`trafficLightColor`值都将匹配到`default:`，并在调试区域打印出**Invalid color**。

这部分关于`if`和`switch`语句的内容到此结束。在下一部分，你将学习可选类型，它允许你创建没有初始值的变量，以及**可选绑定**，它允许在可选类型有值时执行指令。

# 介绍可选类型和可选绑定

到目前为止，每次你声明一个变量或常量时，你都会立即给它赋值。但如果你想在声明变量后稍后再赋值呢？在这种情况下，你会使用可选类型。

更多关于可选类型的信息，请访问[https://docs.swift.org/swift-book/documentation/the-swift-programming-language/thebasics](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/thebasics)。

让我们学习如何创建和使用可选类型，并看看它们如何在程序中使用。想象你正在编写一个程序，其中用户需要输入他们配偶的名字。当然，如果用户没有结婚，这个值就不存在。在这种情况下，你可以使用可选来表示配偶的名字。

可选可能有两种可能的状态。它可以包含一个值，或者不包含值。如果可选包含一个值，你可以访问它内部的值。访问可选值的这个过程被称为 **解包** 可选。让我们通过以下步骤看看这是如何工作的：

1.  将以下代码添加到你的游乐场中，以创建一个变量并打印其内容：

    [PRE7]

1.  由于 Swift 是类型安全的，会出现一个错误，**（变量‘spouseName’在使用之前未初始化）**。

1.  为了解决这个问题，你可以将空字符串赋值给 `spouseName`。按照以下方式修改你的代码：

    [PRE8]

这使得错误消失，但空字符串仍然是一个值，而 `spouseName` 不应该有值。

1.  由于 `spouseName` 应该最初没有值，让我们将其设置为可选。要做到这一点，在类型注释后输入一个问号，并删除空字符串赋值：

    [PRE9]

你会看到一个警告，因为 `spouseName` 现在是一个可选的字符串变量，而不是一个常规的字符串变量，而 `print()` 语句期望的是一个常规字符串变量。

![](img/B31371_03_01.png)

图 3.1：警告通知

即使有警告，现在先忽略它并运行你的代码。`spouseName` 的值在结果区域显示为 **nil**，并且在调试区域打印出 **nil**。`nil` 是一个特殊的关键字，表示可选变量 `spouseName` 没有值。

1.  警告出现是因为 `print` 语句将 `spouseName` 视为 `Any` 类型而不是 `String?` 类型。点击黄色三角形以显示可能的修复，并选择第一个修复：

![](img/B31371_03_02.png)

图 3.2：突出显示第一个修复的扩展警告通知

语句将变为 `print(spouseName ?? default value)`。注意 `??` 操作符的使用。这意味着如果 `spouseName` 不包含值，将使用你提供的默认值在 `print` 语句中。

1.  将默认值占位符替换为 `"No value in spouseName"`，如图所示。警告将会消失。再次运行你的程序，**No value in spouseName** 将会在调试区域显示：

![](img/B31371_03_03.png)

图 3.3：显示默认值的调试区域

1.  让我们给 `spouseName` 赋予一个值。按照以下方式修改代码：

    [PRE10]

当你的程序运行时，**Nia** 将会在调试区域显示。

1.  添加一行代码以将 `spouseName` 与另一个字符串连接，如图所示：

    [PRE11]

你会得到一个错误，并且调试区域会显示错误信息和错误发生的位置。这是因为你不能使用 `+` 操作符将常规字符串变量与可选类型连接。你需要首先解包可选类型。

1.  点击红色圆圈以显示可能的修复方案，你会看到以下内容：

![](img/B31371_03_04.png)

图3.4：带有第二个修复方案高亮的扩展错误通知

第二个修复建议使用**强制解包**来解决这个问题。强制解包无论可选是否包含值都会解包。如果`spouseName`有值，它将正常工作，但如果`spouseName`是`nil`，你的代码将崩溃。

1.  点击第二个修复方案，你会在代码的最后一行`spouseName`后面看到一个感叹号，这表示可选被强制解包了：

    [PRE12]

1.  运行你的程序，你会在结果区域看到`Hello, Nia`被分配给`greeting`，如图所示。这意味着`spouseName`已经成功被强制解包。

1.  要查看强制解包包含`nil`的变量的效果，将`spouseName`设置为`nil`：

    [PRE13]

你的代码崩溃了，你可以在调试区域中看到导致崩溃的原因：

![](img/B31371_03_05.png)

图3.5：崩溃的程序及其在调试区域中的详细信息

由于`spouseName`现在是`nil`，程序在尝试强制解包`spouseName`时崩溃了。

处理这个问题的一个更好的方法是使用可选绑定。在可选绑定中，你尝试将可选中的值分配给一个临时变量（你可以随意命名它）。如果分配成功，将执行一个代码块。

1.  要查看可选绑定的效果，修改你的代码如下：

    [PRE14]

**Hello, Nia**将出现在调试区域。以下是它是如何工作的。如果`spouseName`有值，它将被解包并分配给一个临时常量`spouseTempVar`，然后`if`语句将返回`true`。花括号之间的语句将被执行，常量`greeting`将被分配值`Hello, Nia`。然后，**Hello, Nia**将在调试区域打印出来。注意，临时变量`spouseTempVar`不是一个可选变量。如果`spouseName`没有值，无法将值分配给`spouseTempVar`，`if`语句将返回`false`。在这种情况下，花括号中的语句根本不会执行。

1.  你也可以以前一个步骤更简单的方式编写代码，如下所示：

    [PRE15]

在这里，临时常量使用与可选值相同的名称创建，并将用于花括号之间的语句中。

1.  要查看当可选包含`nil`时可选绑定的效果，再次将`nil`分配给`spouseName`：

    [PRE16]

你会注意到调试区域中没有出现任何内容，即使`spouseName`是`nil`，你的程序也不再崩溃。

这就结束了关于可选和可选绑定的部分，你现在可以创建和使用可选变量了。太棒了！

# 摘要

你做得很好！你学习了如何使用`if`和`switch`语句，这意味着你现在能够编写基于不同条件执行不同操作的程序。

你还学习了可选类型和可选绑定。这意味着你现在可以表示可能具有也可能不具有值的变量，并且只有当变量的值存在时才执行指令。

在下一章中，你将学习如何使用一系列值而不是单个值，以及如何使用循环重复程序语句。

# 加入我们的 Discord 社群！

与其他用户、专家以及作者本人一起阅读这本书。提出问题，为其他读者提供解决方案，通过 Ask Me Anything 会话与作者聊天，等等。扫描二维码或访问链接加入社区。

[https://packt.link/ios-Swift](https://packt.link/ios-Swift)

[![](img/QR_Code2370024260177612484.png)](https://packt.link/ios-Swift)
