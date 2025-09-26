# 第10章。与Game Center集成

苹果提供了一个名为**Game Center**的在线社交游戏网络。您的玩家可以分享高分、追踪成就、挑战朋友，并通过Game Center开始多人游戏的匹配。在本章中，我们将使用苹果的iTunes Connect网站来注册我们的应用，然后我们可以将其与Game Center集成，以在我们的游戏中添加排行榜和成就。

### 小贴士

您需要一个有效的Apple开发者账户（每年99美元），才能将您的应用注册到苹果，使用带有Game Center的iTunes Connect网站，并将您的游戏发布到App Store。

本章包括以下主题：

+   使用iTunes Connect注册应用

+   在我们的应用中验证玩家的Game Center账户

+   从`MenuScene`类打开Game Center

+   添加排行榜

+   创建和授予成就

# 使用iTunes Connect注册应用

由于苹果将在他们的集中式服务器上存储我们的高分和成就，我们需要通知苹果我们需要为我们的应用提供Game Center。第一步是在iTunes Connect网站上为我们的应用创建一个记录。按照以下步骤创建iTunes Connect记录：

1.  在网页浏览器中，导航到[http://itunesconnect.apple.com](http://itunesconnect.apple.com)。

1.  使用您的Apple开发者账户信息登录。

1.  当您到达**iTunes Connect**仪表板时，点击**My Apps**图标。

1.  在左上角，点击**+**符号并选择**New iOS App**，如图所示：![使用iTunes Connect注册应用](img/Image_B04532_10_01.jpg)

1.  在随后的对话框中，找到底部链接，上面写着**在开发者门户上注册新的bundle ID**。点击此链接为您应用创建一个bundle ID。

1.  您将到达一个标题为**Registering an App ID**的页面。这个页面一开始可能看起来令人不知所措，但您只需要填写两个字段。首先，在**App Description**部分输入您应用的名称。

1.  滚动到**App ID Suffix**部分。请确保选择**Explicit App ID**，然后从您的Xcode项目设置中输入**Bundle ID**，如图所示：![使用iTunes Connect注册应用](img/Image_B04532_10_02.jpg)

1.  滚动到**App Services**部分，并确保Game Center选项已经被勾选。

1.  在页面底部，点击**Continue**。然后在随后的确认页面上点击**Submit**。

1.  您现在可以关闭此标签页，返回到iTunes Connect，从新iOS应用屏幕上您离开的地方继续。

    ### 小贴士

    您刚刚创建的bundle ID可能需要一段时间才能在iTunes Connect中显示。如果发生这种情况，请休息一下，几分钟后再次尝试。

1.  输入您的应用的**名称**、**主要语言**、**版本**和**SKU**（对公众不可见）。然后选择您刚刚创建的**Bundle ID**，如图所示：![使用iTunes Connect注册应用](img/Image_B04532_10_03.jpg)

1.  在右下角点击**创建**。现在您应该在 iTunes Connect 中看到您应用的概览，它看起来可能像以下截图：![在 iTunes Connect 中注册应用](img/Image_B04532_10_05.jpg)

恭喜，我们不仅更接近配置 Game Center，我们还迈出了将我们的应用提交到应用商店的第一步！

## 配置 Game Center

现在我们有了 iTunes Connect 应用记录，我们可以告诉苹果我们如何在游戏中使用 Game Center。按照以下步骤配置 Game Center：

1.  在您的应用页面上，点击顶部导航中的 Game Center 链接。

1.  选择**为单款游戏启用**，如图所示：![配置 Game Center](img/Image_B04532_10_06.jpg)

1.  您将看到一个屏幕，允许您为您的游戏创建新的排行榜和成就。太好了！我们将在本章的后面使用这个页面。

我们已通知苹果我们希望在游戏中使用 Game Center。接下来，我们需要创建一个用于测试目的的沙盒用户账户。

## 创建测试用户

在应用开发期间，Game Center 使用独立的测试服务器，因此在我们测试时无法使用我们的真实 Apple ID 登录到 Game Center。相反，我们将在 iTunes Connect 中创建一个沙盒账户。

> *iOS 开发者库的网站指出：“始终为测试 Game Center 中的游戏创建新的测试账户。切勿使用现有的 Apple ID。”*

按照以下步骤创建用于测试的 Game Center 沙盒账户：

1.  在**iTunes Connect**中，使用左上角的下拉菜单选择**用户和角色**，如图所示：![创建测试用户](img/Image_B04532_10_07.jpg)

1.  一旦您进入**用户和角色**页面，请点击屏幕顶部的导航栏中的**沙盒测试者**。

1.  如**沙盒测试者**页面上的指示，点击**+**图标添加新用户。

1.  按照您的喜好填写测试用户的详细信息。以下是我如何填写我的测试用户信息的：![创建测试用户](img/Image_B04532_10_08.jpg)

1.  点击**保存**按钮创建新用户。

### 小贴士

确保将您的实时 Apple ID 和沙盒账户分开。如果您使用沙盒账户登录到实时 Game Center 应用，该账户将失效。

太好了！这就是我们开始将 Game Center 集成到游戏中的所有所需步骤。下一步是将 Game Center 与我们的游戏代码集成。我们将从玩家打开我们的应用时验证他们的 Game Center 账户开始。

# 验证玩家的 Game Center 账户

当我们的应用启动时，我们将检查玩家是否已经登录到他们的 Game Center 账户。如果没有，我们将给他们一个登录的机会。稍后，当我们想要提交高分或成就时，我们可以使用应用启动时收集到的验证信息，而不是打断他们的游戏会话来收集他们的 Game Center 信息。

按照以下步骤在应用启动时验证玩家的 Game Center 账户：

1.  我们将在`GameViewController`类中工作，所以请在Xcode中打开`GameViewController.swift`文件。

1.  在文件顶部添加一个新的`import`语句，以便我们可以使用`GameKit`框架：

    [PRE0]

1.  在`GameViewController`类中，添加一个名为`authenticateLocalPlayer`的新函数，代码如下：

    [PRE1]

1.  在`GameViewController`类的`viewWillLayoutSubviews`函数底部，添加对新创建的`authenticateLocalPlayer`函数的调用：

    [PRE2]

运行你的项目。你应该会看到游戏中心动画进入，请求你的凭证，如下所示：

![验证玩家的游戏中心账户](img/Image_B04532_10_13.jpg)

太好了！记得使用你的沙盒账户。第一次登录时，游戏中心将询问一些额外的问题来设置你的账户。一旦完成游戏中心的表格，你应该返回到主菜单，一个小横幅从屏幕顶部动画进入和退出，告诉你你已经登录。横幅看起来像这样：

![验证玩家的游戏中心账户](img/Image_B04532_10_14.jpg)

如果你看到这个欢迎回来横幅，你就成功实现了游戏中心认证代码。接下来，我们将在菜单中添加一个排行榜按钮，以便玩家可以在我们的应用中查看他们的进度。

# 在我们的游戏中打开游戏中心

如果用户已验证，我们将在`MenuScene`类中添加一个按钮，以便他们可以在我们的游戏中打开排行榜并查看成就。或者，玩家始终可以使用iOS中的游戏中心应用查看他们的进度。

按照以下步骤在菜单场景中创建排行榜按钮：

1.  在Xcode中打开`MenuScene.swift`文件。

1.  在文件顶部添加一个新的`import`语句，以便我们可以使用`GameKit`框架：

    [PRE3]

1.  更新声明`MenuScene`类的行，以便我们的类采用`GKGameCenterControllerDelegate`协议。这允许游戏中心屏幕在玩家关闭游戏中心时通知我们的场景：

    [PRE4]

1.  我们需要一个函数来创建排行榜按钮并将其添加到场景中。当游戏中心验证玩家后，我们将调用此函数。在`MenuScene`类中添加一个名为`createLeaderboardButton`的新函数，如下所示：

    [PRE5]

1.  如果玩家已经通过游戏中心进行了验证，我们将从`didMoveToView`函数中调用我们的`createLeaderboardButton`函数。这为在游戏后返回主菜单的玩家创建按钮。将以下代码添加到`didMoveToView`函数的底部：

    [PRE6]

1.  接下来，我们将创建一个实际打开游戏中心的函数。添加一个名为`showLeaderboard`的新函数，如下所示：

    [PRE7]

1.  我们需要添加另一个函数以遵守`GKGameCenterControllerDelegate`协议。这个函数名为`gameCenterViewDidFinish`，当玩家在游戏中心点击**完成**按钮时，游戏中心将调用它。将此函数添加到`MenuScene`类中，如下所示：

    [PRE8]

1.  为了完成`MenuScene`代码，我们需要在`touchesBegan`函数中检查我们的排行榜按钮的点击，以调用`showLeaderboard`。按照以下方式更新`touchesBegan`函数的`if`块（粗体为新代码）：

    [PRE9]

1.  接下来，打开`GameViewController.swift`并定位到`authenticateLocalPlayer`函数。

1.  更新玩家成功认证后调用的代码块，以在`MenuScene`类中调用我们的新`createLeaderboardButton`函数。这为新认证的人创建排行榜按钮，当他们开始应用程序时。以下是代码示例（粗体为新代码）：

    [PRE10]

干得好。运行项目，您应该在游戏中心认证后看到菜单中出现排行榜按钮，如图所示：

![在我们的游戏中打开游戏中心](img/Image_B04532_10_15.jpg)

太棒了——如果您点击**排行榜**文本，游戏中心将在游戏中打开。现在您的玩家可以直接从您的游戏中查看排行榜和成就。接下来，我们将在iTunes Connect中创建一个排行榜和一个成就来填充游戏中心。

# 检查点10-A

要下载到目前为止的项目，请访问此URL：

[http://www.thinkingswiftly.com/game-development-with-swift/chapter-10](http://www.thinkingswiftly.com/game-development-with-swift/chapter-10)

# 添加高分排行榜

我们将在玩家完成每场比赛后将其分数提交到Game Center服务器。第一步是在iTunes Connect上注册一个新的排行榜。

## 在iTunes Connect中创建新的排行榜

首先，我们将在iTunes Connect中创建我们的排行榜。然后我们可以从我们的代码中连接到这个排行榜并发送新的分数。按照以下步骤在iTunes Connect中创建排行榜记录：

1.  重新登录到iTunes Connect，并导航到您的应用程序的游戏中心页面。

1.  定位并点击显示**添加排行榜**的按钮。

1.  下一页会询问您想创建哪种类型的排行榜。选择**单个排行榜**。

1.  填写您排行榜的信息。您可以在此处参考我的示例：![在iTunes Connect中创建新的排行榜](img/Image_B04532_10_09.jpg)

    让我们看看每个字段：

    +   **参考名称**是iTunes Connect中排行榜列表的内部使用名称

    +   **排行榜ID**是我们将在代码中引用的唯一标识符

    +   **分数格式类型**描述了您将要传递的数据类型（通常用于高分的是整数数据）

    +   正常排行榜使用**分数提交类型**为**最佳分数**，**排序顺序**为**从高到低**

    +   **分数范围**是一种反作弊措施，您可以使用它来阻止明显虚假的分数出现在排行榜上

1.  接下来，点击**添加语言**按钮。您将在此屏幕上选择排行榜的名称和分数格式。这些字段大部分是自我解释的，但您可以在此处参考我的示例：![在iTunes Connect中创建新的排行榜](img/Image_B04532_10_10.jpg)

1.  点击**保存**两次（一次用于语言对话框，一次用于排行榜屏幕）。

你应该回到了游戏中心页面，你的新排行榜在排行榜部分列出。接下来，我们将从我们的游戏代码中推送新的分数到排行榜。

## 从代码更新排行榜

从代码向排行榜发送新分数很简单。按照以下步骤，每次游戏结束时发送收集到的金币数量到排行榜：

1.  在Xcode中打开`GameScene.swift`。

1.  在顶部添加一个`import`语句，这样我们就可以在这个文件中使用`GameKit`框架：

    [PRE11]

1.  在`GameScene`类中添加一个名为`updateLeaderboard`的新函数，如下所示：

    [PRE12]

1.  在`GameScene`类的`GameOver`函数中，调用新的`updateLeaderboard`函数：

    [PRE13]

运行项目并玩一个游戏，将测试金币分数发送到排行榜。然后，点击返回菜单场景并点击**排行榜**按钮，在游戏中打开Game Center。你应该会看到你的第一个分数出现在排行榜上！它看起来可能像这样：

![从代码更新排行榜](img/Image_B04532_10_16.jpg)

干得好——你已经实现了你的第一个Game Center排行榜。接下来，我们将遵循一系列类似的步骤来创建一个收集500个金币的成就。

# 添加成就

成就为你的游戏增添了第二层乐趣，并创造了重玩价值。为了展示一个Game Center成就，我们将添加一个收集500个金币而不死奖励。

## 在iTunes Connect中创建新的成就

就像排行榜一样，我们首先需要为我们的成就创建一个iTunes Connect记录。按照以下步骤创建记录：

1.  登录iTunes Connect并导航到你的应用的Game Center页面。

1.  在排行榜列表下方，找到并点击**添加成就**按钮。

1.  填写你成就的信息。以下是我的值：![在iTunes Connect中创建新的成就](img/Image_B04532_10_11.jpg)

    让我们看看每个字段：

    +   **参考名称**是iTunes Connect将用于内部引用成就的名称

    +   **成就ID**是我们将在代码中引用此成就的唯一标识符

    +   你可以为每个成就分配一个**点数**，这样玩家在收集新的成就时可以获得更多的成就点数

    +   **隐藏**和**可多次获得**是显而易见的，但你可以在右侧使用问号按钮获取来自苹果的更多信息

1.  点击**添加语言**按钮。我们将命名成就并给出描述，就像排行榜过程一样。此外，成就还需要一个图片。图片尺寸可以是512x512或1024x1024。你可以在我们的**Assets**包下载中找到我使用的图片，在`Extras`文件夹中，`gold-medal.png`。以下是我的值：![在iTunes Connect中创建新的成就](img/Image_B04532_10_12.jpg)

1.  点击 **保存** 两次（一次用于语言对话框，一次用于成就屏幕）。

太棒了，你应该已经回到了你应用的首页，在 **成就** 部分列出了你的新成就。接下来，我们将把这个成就集成到游戏中。

## 从代码更新成就

就像发送排行榜更新一样，我们可以从 `GameScene` 发送成就更新到 Game Center。按照以下步骤集成我们的 500 枚金币成就：

1.  在 Xcode 中打开 `GameScene.swift` 文件。

1.  如果你跳过了排行榜部分，你需要在文件的顶部添加一个新的 `import` 语句，这样我们就可以使用 `GameKit`。如果你已经实现了排行榜，你可以跳过这一步：

    [PRE14]

1.  在 `GameScene` 类中添加一个名为 `checkForAchievements` 的新函数，如下所示：

    [PRE15]

1.  在 `gameOver` 函数的底部，调用新的 `checkForAchievements` 函数：

    [PRE16]

运行项目，如果你敢的话，完成 500 枚金币飞越。当你的游戏结束时，你应该会看到一个横幅宣布你的新成就征服，如下所示：

![从代码更新成就](img/Image_B04532_10_17.jpg)

干得好！你已经将 Game Center 排行榜和成就集成到了你的游戏中。

# 检查点 10-B

要下载到这一点的我的项目，请访问此 URL：

[http://www.thinkingswiftly.com/game-development-with-swift/chapter-10](http://www.thinkingswiftly.com/game-development-with-swift/chapter-10)

# 摘要

与 Game Center 集成是为你玩家提供的优秀功能。在本章中，我们学习了如何为我们的应用创建一个 iTunes Connect 记录，在我们的代码中验证 Game Center 用户，在 iTunes Connect 上创建新的排行榜和成就，然后在我们游戏中集成这些排行榜和成就。我们已经取得了很大的进步！

我们正式完成了游戏本身的工作。在下一章中，我们将为应用发布做准备，上传代码供苹果审核，并回顾我们在创建伟大游戏过程中所学到的知识。一切都在顺利进行，我们准备迈出最后一步，发布我们的游戏。恭喜！
