# 第11章。开始多人游戏

在本章中，我们将关注以下食谱：

+   多人游戏的结构

+   多人游戏的设置

+   为玩家分配角色

# 简介

到目前为止，在本书中，我们已经做了很多与游戏相关酷炫的事情，例如SpriteKit、视差滚动背景、使用自主移动代理的物理模拟、使用OpenGL进行三维游戏编程等等。所有这些都是为了制作单人游戏，意味着一次只能有一个人玩。但现在，我们将向前迈进，制作一个多人游戏，这个游戏可以同时吸引多个人。多人游戏本身对用户来说就更有吸引力、更有趣，因为实时竞争加入了进来，使得游戏体验对用户来说更加愉快。所以，现在是时候了解与多人游戏相关的内容了。在[第12章](part0069_split_000.html#page "第12章。实现多人游戏")《实现多人游戏》中，我们将创建一个多人游戏。为了游览多人游戏开发，整体议程将分为以下部分：

1.  创建一个示例多人游戏以了解多人游戏的结构和各种状态。

1.  使用SpriteKit和苹果的Multipeer Connectivity框架设置相同的多人游戏。之后，使用同一框架的`MCBrowserViewController`进行玩家之间的握手或连接建立。

1.  通过发送和接收网络数据包为玩家分配角色。

# 多人游戏的结构

在单人游戏中，只有一个玩家，所以谈论游戏作为一个维护所有游戏行为的对象，而如果我们理解多人游戏的结构，我们会看到它完全不同。在多人游戏中，有多个玩家在玩同一款游戏，所以从技术上讲，对于每个设备，都有一个玩家正在积极驱动该设备上的游戏。这被称为本地玩家，而所有其他玩家都被视为该设备的远程玩家。理想情况下，本地玩家的活动应该更新在远程玩家的设备上，这是多人开发中最主要的挑战。本地玩家的更新被称为在其他设备上同步游戏，这是由游戏对象完成的，该游戏对象位于游戏中。游戏对象（即运行在设备上的游戏实例）的责任是使所有设备上的游戏看起来与实时游戏一样。

因此，在接下来的这一节中，我们将创建一个全新的多人游戏，名为TankRace，使用SpriteKit，其中将实例化游戏会话。我们将结合多人游戏状态及其解释和必要性。所有会话和多人相关的过程都将使用iOS 7中引入的Multipeer Connectivity框架完成，该框架是iOS 6中GameKit的一部分。

## 准备工作

要使用SpriteKit开发坦克大战多人游戏，首先创建一个新的项目。打开Xcode，转到**文件** | **新建** | **项目** | **iOS** | **应用程序** | **SpriteKit游戏**。在弹出的窗口中，将**产品名称**输入为`TankRace`，转到**设备** | **iPhone**，然后点击**下一步**，如下截图所示：

![准备就绪](img/00165.jpeg)

点击**下一步**，并将项目保存在你的硬盘上。

一旦项目保存，你应该能够看到项目设置。在项目设置页面，只需从**设备方向**部分勾选**纵向**，并取消勾选所有其他选项，因为我们只支持这款游戏的纵向模式。同时将部署目标设置为7.0，以便支持一系列设备。

变化如下所示：

![准备就绪](img/00166.jpeg)

让我们更仔细地看看SpriteKit提供的是什么结构：

![准备就绪](img/00167.jpeg)

在`GameViewController`的`viewDidLoad`方法中，编写了一段代码，将其视图转换为`SKView`，并在`SKView`上呈现一个场景，即`GameScene`，如下所示。项目本身实现了`unarchiveFromFile`方法来获取`GameScene.sks`文件，我们可以在创建的项目中看到它。为了不显示FPS和节点，注释掉以下代码中的两行：

[PRE0]

## 如何操作...

在开始多人游戏代码之前，我们应该让游戏为多人游戏做好准备。首先，进入`GameScene`类，并在通常设置场景的覆盖方法`didMoveToView`中删除示例`SKLabelNode`添加代码。其次，从`touchesBegan:withEvent`方法中删除`for`循环，该方法负责添加`SKSpriteNode`及其动作。

我们的项目现在可以开始多人游戏了。多人游戏可以通过多种方式开发。它们可以通过蓝牙、Wi-Fi、互联网或GameCenter进行游戏。所有这些技术都允许我们互联设备并在设备之间共享数据。这使我们能够实时显示玩家的移动。你可能已经看到了多人游戏中的响应速度。它们非常流畅。在本节中，我们将探讨更多关于多人游戏及其在iOS中的实现。在这里，我们将为本地玩家实例化一个会话（即MCSession），该会话将进一步连接到本食谱中的另一个玩家。此外，为了指导用户触摸，我们将添加一个信息标签，显示“点击连接”，并进一步实现MCSession的代理，随后将解释多人游戏状态。以下是实现此任务的步骤：

1.  打开`GameScene.m`文件，创建一个具有`InfoLabel`属性和所有相关会话内容的接口。同时让`GameScene`遵循`MCSessionDelegate`，接口看起来如下：

    [PRE1]

    在这里，`gameSession`是用于玩多人游戏的会话，`gamePeerID`是本地玩家的唯一ID，在将来将作为连接到此设备的远程玩家的唯一ID。这就是为什么它被称为peerID。`ServiceType`是特别分配给游戏的唯一ID；在这里，服务类型将是TankRace，而广告商是一个处理所有传入邀请给用户的类，并处理所有用户响应的类。声明了一个`gameInfoLabel`属性，它将被创建来指导用户与其他玩家连接。

1.  添加一个名为`addGameInfoLabelWithText`的方法，它可以用来显示任何带有`pragma`标记的GameInfo。

    [PRE2]

1.  为不同的GameInfo文本声明哈希定义。

    [PRE3]

1.  从`GameScene`的`didMoveToView`方法中调用`addGameInfoLabelWithText`。使用文本哈希定义`kConnectingDevicesText`和`pragma`标记，如下所示。

    [PRE4]

1.  在`GameScene`的私有接口中声明一个`enum`，`GameState`，以及与之对应的属性。同时，将游戏初始状态设置为`kGameStatePlayerToConnect`，因为要开始多人游戏，玩家首先需要连接才能玩游戏。将这些行添加到哈希定义之上：

    [PRE5]

1.  在`GameScene`的私有接口中添加此`gameState`属性：

    [PRE6]

1.  在`GameScene`的`didMoveToView`中将`gameState`赋值为`kGameStatePlayerToConnect`：

    [PRE7]

1.  创建一个名为`instantiateMCSession`的方法，并添加如下代码中的`pragma`标记：

    [PRE8]

    [PRE9]

1.  实现所有具有以下`pragma`标记的`MCSessionDelegate`：

    [PRE10]

    这些都是在`GameScene`类中实现的`MCSession`的代理方法，其中前两种使用得最多。第一个用于确定游戏状态的变化，例如，是否已连接、正在连接或未连接。后者用于接收数据，因此可以在上述实现中的操作队列块中处理这些数据。

1.  现在根据`GameScene`的`gameState`，在`touchBegan:withEvent`中添加`instantiateMCSession`，并使用`pragma`标记。

    [PRE11]

    在`touchesBegan`方法中，如果状态是`kGameStatePlayerToConnect`，则表示用户已触摸以开始游戏，即技术上需要完成玩家的连接，而在游戏的其他状态下，将根据触摸相应地处理。

    经过所有这些步骤，已经完成了游戏初始会话的设置，并理解了多人游戏架构。

## 它是如何工作的...

在前面的设置中，我们使用了 Multipeer Connectivity 框架，通过 `MCSession` 实例来设置一个多人游戏的结构，这个实例将存在于每个用于玩游戏设备上。我们还实现了所有其代理方法，这些方法会通知 `GameScene` 游戏状态的变化，也将在接收来自某些网络数据包的传入部分时使用。现在，在这一节中，我们放置了一个标签 `Tap to connect`，点击屏幕时将实例化一个会话。现在构建项目。首先你会看到以下启动屏幕，然后是带有标签 **Tap to connect** 的初始 `GameScene`：

![工作原理...](img/00168.jpeg)

# 多人游戏的设置

在这个菜谱中，我们将编写设置我们的多人游戏的代码。所有配置和会话管理器都将包含在本节中。我们将深入研究创建和维护会话的各种概念。

## 准备工作

在开始这个菜谱之前，我们应该了解 Multipeer Connectivity 框架中的 MCSession、MCPeerId、广告商和服务类型术语。在这个菜谱中，我们将建立玩家之间的连接，从而他们可以在未来进行通信，让玩家玩游戏，我们将在下一章中这样做。

## 如何操作

现在，点击屏幕后，已经实例化了一个具有服务类型的 MCSession；我们可以使用这个会话和服务类型来展示 `MCBrowserViewController` 并在玩家（即设备）之间建立连接。`MCBrowserViewController` 是完全配备和设计用于连接 Multipeer Connectivity 框架中提供的会话的多个玩家。以下是涉及的步骤：

1.  首先，创建一个 `GameScene` 的协议 `GameSceneDelegate` 和其在 `GameScene` 中的代理对象，该对象将被设置为 `GameViewController` 以便在用户触摸屏幕时使用其代理方法。`GameViewController` 可以被通知展示 `MCBrowserViewController`。声明协议代码和 `GameSceneDelegate` 对象，如下所示：

    [PRE12]

1.  当用户触摸的屏幕上的 `gameState` 为 `kGameStatePlayerToConnect` 时，我们调用 `instantiateMCSession` 方法，该方法还通知 `gameSceneDelegate` 通过传递创建的 `gameSession` 和 `serviceType` 属性来展示 `MCBrowserViewController`：

    [PRE13]

1.  代理方法必须由 `GameViewController` 调用，在同一个控制器上，`MCBrowserViewController` 也必须展示，它也将有自己的代理方法。现在，是时候声明 `GameViewController` 的私有接口，并遵循 `MCBrowserViewControllerDelegate` 和 `GameSceneDelegate`，如下代码片段所示：

    [PRE14]

1.  在 `GameViewController` 的 `viewDidLoad` 方法中，将本地场景对象替换为 `self.gameScene`，并将 `GameScene` 对象的 `gameSceneDelegate` 属性设置为 `GameViewController`，如下所示：

    [PRE15]

1.  实现如下 `GameSceneDelegate` 的代理方法：

    [PRE16]

    在这个方法中，`MCBrowserViewController` 在 `GameViewController` 上呈现，并设置了其代理，并将对等体限制为 `2`。

1.  向 `GameScene` 添加两个公共方法，用于调用 `MCBrowserViewController` 的取消和完成操作。

    +   在 `GameScene.h` 文件中，声明公共方法，如下所示：

        [PRE17]

    +   在 `GameScene.m` 文件中，定义公共方法，例如：

        [PRE18]

1.  现在我们将在 `GameScene` 文件中添加两个公共方法。这些方法将分别在 `MCBrowserViewControllerDelegate` 的取消和完成操作中调用：

    [PRE19]

    在这两个代理方法中，首先关闭 `MCBrowserViewController`，并通知 `GameScene` 适当更改。

现在当两位设备玩家点击屏幕时，`MCBrowserViewController` 打开，玩家尝试使用此控制器提供的默认行为相互连接，完成后向玩家显示相应的文本。因此，这个整个实现完成了本章的入门套件。

## 工作原理

现在我们将了解如何使用 `MCBrowserViewController` 建立连接，以下步骤（在下图中，左侧是模拟器设备，右侧是iPhone 5s）：

1.  两位玩家点击屏幕，打开 `MCBrowserViewController`，搜索附近的对等体，取消和完成按钮放置在导航栏上。在这里，完成按钮是禁用的，因为最初没有人连接到设备。![工作原理](img/00169.jpeg)

1.  一旦检测到对等体，它会在列表中显示设备的名称。![工作原理](img/00170.jpeg)

1.  之后，两位玩家都按下他们想要连接的设备名称，搜索对等体的操作停止。因此，根据这个设备选择，发送一个连接到它的请求。![工作原理](img/00171.jpeg)

1.  根据另一用户的回复，更新表格行右侧的对等体状态；它可以是**连接中**，**已连接**。当设备连接时，状态变为**已连接**，并且**完成**按钮被启用。![工作原理](img/00172.jpeg)

1.  当玩家选择**完成**或**取消**时，我们显示相应的文本，点击**完成**按钮显示**设备已连接**，点击**取消**按钮显示**点击连接**。现在，设备在逻辑上是连接的，并且共享相同的会话。这个会话将由用户在多人游戏中进一步使用来玩游戏。![工作原理](img/00173.jpeg)

在这个过程中，我们还将看到一些网络延迟，所以如果设备没有连接，尝试通过取消控制器并再次点击屏幕来刷新控制器以重新连接。

# 分配玩家角色

在这个配方中，我们将通过为玩家分配角色，将我们的游戏模板提升到下一个步骤。这意味着我们将从逻辑上划分用户并为他们分配角色。这将为玩家提供个体身份。

## 准备工作

在开始分配或我们也可以称之为玩家身份的分配之前（即第一位玩家和第二位玩家），我们应该熟悉多对等连接框架。我们还必须具备网络数据包发送和接收的基本知识。在本节中，一旦使用前面配方中描述的`MCBrowserViewController`连接，我们将为玩家分配第一位和第二位玩家的身份。

## 如何操作

为了完成玩家的分配，以下是需要遵循的步骤：

1.  为了设置这个目的，添加一些枚举、哈希定义常量和属性，如下所示：

    +   声明一个名为`NetworkPacketCode`的`enum`，目前我们只添加了`KNetworkPacketCodePlayerAllotment`数据包代码，未来可以添加更多数据包代码，用于从游戏中发送和接收数据包。

        [PRE20]

    +   添加在玩家角色正在决定时显示给玩家的文本。

        [PRE21]

    +   在`GameScene.m`中添加最大数据包大小常量以及一些属性，如`gamePacketNumber`、`gameUniqueIdForPlayerAllocation`，以便在发送数据包时使用。

        [PRE22]

1.  现在为了从一个设备向另一个设备发送数据，我们有一个封装的数据容器，称为数据包。现在这个数据包通过网络发送，其他玩家的设备将相应地更新视图和位置。为此，创建一个方法来发送带有头部`NetworkPacketCode`和数据的数据包，指定要发送数据包的`peerId`以及数据包是否应该使用可靠服务发送。

    [PRE23]

    在这里，`networkPacket`通过一个头部和数据创建。声明了一个变量`pIntData`，它是包含`NetworkPacketCode`和`gamePacketNumber`的头部，以便为数据包分配一个唯一的数字，以序列化网络数据包，用于同步或正确更新游戏。一旦创建数据包，就调用`MCSession`的`sendData`方法，传递要发送的数据包，`peerID`数据包需要发送到的对等方，模式可以是`MCSessionSendDataUnreliable`或`MCSessionSendDataReliable`，以及`error`来检查在发送数据包时是否发生错误。

    此方法将在游戏中各个地方重复使用，以向相同游戏的对等方发送数据包。

1.  生成一个随机数并将其存储在上述声明的变量`gameUniqueIdForPlayerAllocation`中，这将有助于决定哪位是第一位和第二位玩家。在`GameScene`的`didMoveToView`方法中添加此行。

    [PRE24]

1.  将以下代码添加到`MCSession`的接收数据代理方法中，根据其`NetworkPacketCode`处理接收到的数据包，如下所示：

    [PRE25]

    在接收数据时，应在`mainQueue`操作块中处理。在这个块中，我们将从`pIntData`指针变量中移除头部并获取数据包中发送的`NetworkPacketCode`。在这个代码中，我们将检查发送的包的类型。然后我们将根据其类型解析数据包。在这里，一个名为`KNetworkPacketCodePlayerAllotment`的玩家分配包类型被传递，因此检索到的数据是`gameUniqueId`。如前所述，在`didMoveToView`中，我们为两个设备分配了一个随机数给变量`gameUniqueIdForPlayerAllocation`。因此，对于两个设备，生成了不同的数字，并且在从两个设备发送分配数据包时，这作为数据（将在下一点讨论要发送的分配数据包）传递。最后，为了决定哪个是第一和第二玩家，将比较本地`gameUniqueIdForPlayerAllocation`的值和包中发送的值，在这个比较中，一个将被分配为第一玩家，另一个将被分配为第二玩家，通过更改`gameInfoLabel`的适当文本来通知用户，如代理方法中所示。

1.  请从`GameScene`的公共方法`startGame`中删除以下写有内容的行，因为现在`gameInfoLabel`将根据接收到的数据包设置。

    [PRE26]

1.  所有这些早期过程都是在用户点击完成按钮时开始的。这个按钮是玩家已经连接的指示，并且将调用`MCSession`的代理方法`didChangeState`，并带有名为`MCSessionStateConnected`的`MCSessionState`，并且由于连接状态的方法中已经内置了检查协议，所以在`if`语句中添加以下代码：

    [PRE27]

    在这个方法中，设置所有来自该方法的属性，因为它是远程玩家信息，并将游戏状态本地设置为`kGameStatePlayerAllotment`。然后，我们将分配玩家的数据包发送到`peerID`，对于该ID已经建立了连接，使用`NetworkPacketCode`和数据部分，这将如之前讨论的那样在远程端接收。 

最后，我们完成了为多人游戏连接两个玩家并为他们分配唯一身份以进一步识别以构建游戏的工作。这个方法作为本章的解决方案集。

## 如何工作

玩家的整个分配取决于数据包中发送的动作和数据，以及接收端根据发送端设定的规范如何解析。为了完成玩家身份的分配，我们使用了一个随机数变量，该变量是本地生成的并在分配数据包中传递。在接收端，编写了分配逻辑，检查本地设置的随机数和远程传递的随机数。基于这个比较，确定了第一和第二玩家。

在两个设备上显示了一些文本，告知玩家他们的身份，如下所示：

![如何工作](img/00174.jpeg)

## 还有更多

在上一节中，我们使用了多对等连接框架。我们还可以使用GameKit框架。有关更多信息，请参阅[https://developer.apple.com/library/ios/documentation/GameKit/Reference/GameKit_Collection/index.html](https://developer.apple.com/library/ios/documentation/GameKit/Reference/GameKit_Collection/index.html)。

## 参见

为了更好地理解和学习多对等连接框架，请访问[https://developer.apple.com/library/prerelease/ios/documentation/MultipeerConnectivity/Reference/MultipeerConnectivityFramework/index.html](https://developer.apple.com/library/prerelease/ios/documentation/MultipeerConnectivity/Reference/MultipeerConnectivityFramework/index.html)。
