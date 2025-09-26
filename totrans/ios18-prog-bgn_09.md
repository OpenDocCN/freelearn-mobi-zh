# 8

# 协议、扩展和错误处理

在上一章中，你学习了如何使用类或结构体来表示复杂对象，以及如何使用枚举将相关值分组在一起。

在本章中，你将了解**协议**、**扩展**和**错误处理**。协议定义了一个方法、属性和其他要求的蓝图，这些可以由类、结构体或枚举采用。扩展使你能够为现有的类、结构体或枚举提供新功能。错误处理涵盖了如何响应和从程序中的错误中恢复。

到本章结束时，你将能够编写自己的协议以满足你应用程序的需求，使用扩展为现有类型添加新功能，并在你的应用程序中处理错误条件而不会崩溃。

本章将涵盖以下主题：

+   探索协议

+   探索扩展

+   探索错误处理

# 技术要求

本章的Xcode游乐场位于本书代码包的`Chapter08`文件夹中，可以在此处下载：

[https://github.com/PacktPublishing/iOS-18-Programming-for-Beginners-Ninth-Edition](https://github.com/PacktPublishing/iOS-18-Programming-for-Beginners-Ninth-Edition)

查看以下视频，看看代码的实际应用：

[https://youtu.be/fV6VNlDyyG0](https://youtu.be/fV6VNlDyyG0)

如果你希望从头开始，创建一个新的游乐场，并将其命名为`ProtocolsExtensionsAndErrorHandling`。你可以一边输入一边运行本章中的所有代码。让我们从协议开始，这是一种指定类、结构体或枚举应该具有的属性和方法的方式。

# 探索协议

协议就像蓝图，决定了对象必须拥有的属性或方法。在你声明了一个协议之后，类、结构体和枚举可以采用它，并为所需的属性和方法提供自己的实现。

这就是协议声明的样子：

[PRE0]

就像类和结构体一样，协议名称以大写字母开头。属性使用`var`关键字声明。如果你想有一个可读可写的属性，你使用`{get set}`，如果你想有一个只读属性，你使用`{get}`。请注意，你只需指定属性和方法名称；实现是在采用类、结构体或枚举内部完成的。

更多关于协议的信息，请访问[https://docs.swift.org/swift-book/documentation/the-swift-programming-language/protocols](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/protocols)。

为了帮助你理解协议，想象一个快餐店使用的应用程序。管理层已经决定显示正在提供的餐点的卡路里含量。该应用程序目前有以下类、结构体和枚举，它们都没有实现卡路里含量：

+   一个`Burger`类

+   一个`Fries`结构体

+   一个`Sauce`枚举

将以下代码添加到您的游乐场中，以声明`Burger`类、`Fries`结构和`Sauce`枚举：

[PRE1]

这些代表应用中现有的类、结构和枚举。不用担心空定义，因为它们对于本课程不是必需的。如您所见，它们目前都没有卡路里计数。让我们学习如何创建一个指定实现卡路里计数所需属性和方法协议。您将在下一节中声明此协议。

## 创建协议声明

让我们创建一个协议，该协议指定一个必需的属性`calories`和一个方法`description()`。在类、结构和枚举声明之前，将以下内容输入到您的游乐场中：

[PRE2]

此协议命名为`CalorieCountable`。它指定了采用它的任何对象必须有一个属性`calories`，该属性包含卡路里计数，以及一个返回字符串的方法`description()`。`{ get }`表示您只需能够读取存储在`calories`中的值，而无需写入它。请注意，`description()`方法的定义没有指定，因为这将在类、结构或枚举中完成。您要采用协议只需在类名后键入一个冒号，然后是协议名，并实现所需的属性和方法。

要使`Burger`类符合此协议，请按以下方式修改您的代码：

[PRE3]

如您所见，`calories`属性和`description()`方法已添加到`Burger`类中。尽管协议指定了一个变量，但您在这里可以使用一个常量，因为协议只要求您获取`calories`的值，而不需要设置它。

让我们让`Fries`结构也采用此协议。按以下方式修改您的`Fries`结构代码：

[PRE4]

添加到`Fries`结构的代码与添加到`Burger`类的代码类似，并且它现在也符合`CalorieCountable`协议。

您也可以以相同的方式修改`Sauce`枚举，但让我们使用扩展来实现。扩展可以扩展现有类、结构或枚举的功能。您将在下一节中使用扩展将`CalorieCountable`协议添加到`Sauce`枚举中。

# 探索扩展

扩展允许您在不修改原始对象定义的情况下向对象提供额外的功能。您可以在Apple提供的对象上使用它们（您无法访问对象定义的地方）或当您希望将代码分离以提高可读性和易于维护时。以下是一个扩展的示例：

[PRE5]

在这里，扩展被用来向现有类提供额外的属性和方法。

更多关于扩展的信息，请访问[https://docs.swift.org/swift-book/documentation/the-swift-programming-language/extensions](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/extensions)。

让我们看看如何使用扩展。您将首先通过在下一节中使用扩展使 `Sauce` 枚举符合 `CalorieCountable` 协议。

## 通过扩展采用协议

目前，`Sauce` 枚举不符合 `CalorieCountable` 协议。您将使用扩展来添加使其符合所需的属性和方法。在 `Sauce` 枚举声明之后输入以下代码：

[PRE6]

如您所见，对 `Sauce` 枚举的原定义没有进行任何更改。如果您想扩展现有的 Swift 标准类型的功能，如 `String` 和 `Int`，这也很有用。

枚举实例不能像结构体和类那样在属性中存储值，因此使用 `switch` 语句根据枚举的值返回卡路里数。`description()` 方法与 `Burger` 类和 `Fries` 结构体中的相同。

所有三个对象都有一个 `calories` 属性和一个 `description()` 方法。太棒了！

让我们看看如何在下一节中将它们放入数组中，并执行操作以获取餐点的总卡路里数。

## 创建不同类型对象的数组

通常，数组的元素必须是同一类型。然而，由于 `Burger` 类、`Fries` 结构体和 `Sauce` 枚举都符合 `CalorieCountable` 协议，您可以创建一个包含符合此协议的元素的数组。按照以下步骤操作：

1.  要将 `Burger` 类的实例、`Fries` 结构体和 `Sauce` 枚举的实例添加到数组中，在文件中的所有其他代码之后输入以下代码：

    [PRE7]

1.  要获取总卡路里数，在创建 `foodArray` 常量之后的行中添加以下代码：

    [PRE8]

`reduce` 方法用于从 `foodArray` 数组的元素中生成一个单一值。此方法的第一参数是初始值，设置为 `0`。第二个参数是一个闭包，它将初始值与一个元素的 `calories` 属性中存储的值组合。这将对 `foodArray` 数组中的每个元素重复进行，并将结果分配给 `totalCalories`。总数量，**1315**，将在调试区域显示。

您已经学习了如何创建一个协议，并使类、结构体或枚举符合它，无论是通过类定义还是通过扩展。让我们接下来看看错误处理，并了解如何在程序中响应或恢复错误。

# 探索错误处理

当您编写应用程序时，请记住可能会发生错误条件，错误处理是您的应用程序如何响应和从这些条件中恢复的方式。

首先，你创建一个符合Swift的`Error`协议的类型，这使得该类型可用于错误处理。枚举通常被使用，因为你可以为不同类型的错误指定关联值。当发生意外情况时，你可以通过抛出错误来停止程序执行。你使用`throw`语句来完成此操作，并提供一个符合`Error`协议的类型的实例，并带有适当的值。这允许你看到出了什么问题。

当然，如果你能在不停止程序的情况下响应错误，那就更好了。为此，你可以使用`do-catch`块，它看起来像这样：

[PRE9]

在这里，你尝试使用`try`关键字在`do`块中执行代码。如果抛出错误，`catch`块中的语句将被执行。你可以有多个`catch`块来处理不同类型的错误。

更多关于错误处理的信息，请访问[https://docs.swift.org/swift-book/documentation/the-swift-programming-language/errorhandling](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/errorhandling)。

例如，假设你有一个需要访问网页的应用程序。然而，如果该网页所在的服务器宕机，你需要编写代码来处理错误，例如尝试使用备用网页服务器或通知用户服务器已宕机。

让我们创建一个符合`Error`协议的枚举，当发生错误时使用`throw`语句停止程序执行，并使用`do-catch`块来处理错误。按照以下步骤操作：

1.  将以下代码输入到你的playground中：

    [PRE10]

这声明了一个符合`Error`协议的枚举`WebsiteError`，它涵盖了三种可能的错误条件：没有互联网连接、网站宕机或URL无法解析。

1.  在`WebsiteError`定义之后输入以下代码以声明一个函数，该函数在`WebpageError`声明之后检查网站是否可用：

    [PRE11]

如果`siteUp`为`true`，则返回`"Site is up"`。如果`siteUp`为`false`，程序将停止执行并抛出错误。

1.  在`checkWebsite(siteUp:)`函数定义之后输入以下代码并运行你的程序：

    [PRE12]

由于`siteStatus`为`true`，**Site is up**将出现在结果区域。

1.  将`siteStatus`的值更改为`false`并运行你的程序。你的程序会崩溃，并在调试区域显示以下错误消息：

    [PRE13]

1.  当然，如果你能在不使程序崩溃的情况下处理错误，那就更好了。你可以通过使用`do-catch`块来实现这一点。按照以下所示修改你的代码并运行它：

    [PRE14]

`do`块尝试执行`checkWebsite(siteUp:)`函数，并在成功时打印状态。如果有错误发生，而不是崩溃，`catch`块中的语句将被执行，错误消息`siteDown`将出现在调试区域。

你可以通过实现多个`catch`块来让你的程序处理不同的错误条件。有关详细信息，请参阅此链接：[https://docs.swift.org/swift-book/documentation/the-swift-programming-language/errorhandling](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/errorhandling)。

你已经学会了如何在应用程序中处理错误而不会导致其崩溃。太棒了！

# 摘要

在本章中，你学习了如何编写协议以及如何使类、结构和枚举符合这些协议。你还学习了如何通过使用扩展来扩展类的能力。最后，你学习了如何使用`do-catch`块来处理错误。

这些内容现在可能看起来相当抽象且难以理解，但在本书的**第3部分**中，你将看到如何使用协议在程序的各个部分实现常见功能，而不是一遍又一遍地编写相同的程序。你将看到扩展在组织代码方面的有用性，这使得代码易于维护。最后，你将看到良好的错误处理如何使定位你在编写应用程序时犯的错误变得容易。

在下一章中，你将了解**Swift并发**，这是在Swift中处理异步操作的新方法。

# 加入我们的Discord频道！

与其他用户、专家以及作者本人一起阅读这本书。提出问题，为其他读者提供解决方案，通过“问我任何问题”的环节与作者聊天，还有更多。扫描二维码或访问链接加入社区。

[https://packt.link/ios-Swift](https://packt.link/ios-Swift)

[![](img/QR_Code2370024260177612484.png)](https://packt.link/ios-Swift)
