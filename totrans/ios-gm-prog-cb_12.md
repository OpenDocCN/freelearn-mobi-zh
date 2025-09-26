# 第12章。实现多人游戏

在本章中，我们将关注以下主题：

+   创建我们的TankRace环境

+   玩家的移动

+   实现游戏玩法

# 简介

在前面的章节中，我们介绍了玩家的握手过程，如连接建立和为玩家分配唯一标识。现在，我们将继续探索更多关于多人游戏的内容，并创建一个名为TankRace的多人游戏。在创建游戏的过程中，我们将实现以下功能：

+   创建视觉游戏设置，例如添加玩家、背景和其他资产

+   实现玩家在触摸屏幕时的移动，并同步设备上的玩家

+   实现游戏玩法，即玩家操作上的胜利和失败逻辑

# 创建我们的TankRace环境

游戏的名字本身暗示它是一个赛车游戏，其中将有两位玩家和名为蓝色坦克和红色坦克的坦克。在iPhone的纵向模式下，坦克将被放置在屏幕的两侧，并将有两个终点线，一个蓝色终点线和一条红色终点线，蓝色坦克（比如说第一位玩家）和红色坦克（比如说第二位玩家）分别要穿越。玩家的移动将基于触摸行为，第一个穿越自己终点线的坦克的玩家将获胜，另一个将失败。我们将创建游戏环境，即TankRace，它将包括添加玩家、游戏背景和终点线。

## 准备工作

在前面的章节中，你已经学习了如何添加精灵和背景，以及更新它们的属性，如位置、旋转等。现在，我们将添加游戏中需要的所有资产，使其可玩。

## 如何操作

添加游戏中所需所有资产的操作步骤如下：

1.  将本章代码包中提供的资源（`BlueTank.png`、`RedTank.png`和`Background.png`）拖放到项目中，添加文件后，项目导航器将看起来像这样：![如何操作](img/00175.jpeg)

1.  现在，在这里，我们将使用GameKit，这是一个创建社交游戏的好框架。这个框架提供了各种功能，如点对点连接、游戏中心和游戏内语音聊天。导入GameKit并声明一些枚举、结构、属性和常量，以便在接下来的代码中使用，如下所示：

    +   导入`GameKit`以使用CGPoint存储数据结构：

        [PRE0]

    +   为玩家添加两个`NetworkPacketCode`以移动，并为游戏结束时添加一个包代码，例如，一个表示游戏失败的包：

        [PRE1]

    +   声明一个名为`TankInfo`的结构，它将被用作发送坦克信息的数据库结构；相同的结构将在远程玩家的接收端进行同步。

        [PRE2]

    +   对于坦克的移动，添加`Speed`和`TurnSpeed`常量

        [PRE3]

    +   将第一和第二位玩家的标签文本分别改为蓝色和红色

        [PRE4]

    +   添加当有人获胜或失败时的文本哈希定义。

        [PRE5]

1.  向 `GameScene` 的私有接口添加一些属性，如要维护和更新的坦克局部数据结构，以便它可以发送到远程端。同时，声明所有在游戏中使用的 `SKSpriteNodes` 和 `SKShapeNodes`。

    [PRE6]

1.  现在，我们将在场景中添加一些节点，因此 `GameScene` 需要正确的大小。点击 `GameScene.sks` 并从 Xcode 右上角工具栏的最后一个图标打开右侧面板。从那里，将 `GameScene` 的大小更改为 `320 x 568`，这是 iPhone 4 英寸的大小。![如何操作](img/00176.jpeg)

1.  在 `adding assets` 方法的 `pragma` 标记下添加一个方法 `addGameBackground`，在其中创建一个 `SKSpriteNode`，并添加之前在项目中添加的图像 `Background.png`。保持背景的 *z* 位置为 `0`，因为它将在游戏中的所有其他节点下方。

    [PRE7]

1.  在 `adding assets` 方法下添加两个不同的方法 `pragma` 标记，用于玩家，即蓝色和红色坦克，分别使用 `BlueTank.png` 和 `RedTank.png` 图像作为 `SpriteNodes`。

    [PRE8]

1.  此外，在 `adding assets` 方法的 `pragma` 标记下创建两个方法，用于完成线，两个玩家都必须达到以获胜。

    [PRE9]

1.  这里我们使用 `SKNodeShape` 类对象，用于在屏幕上使用核心图形路径绘制任何所需的形状。因此，我们在游戏场景的两端添加红色和蓝色线条。

1.  要在数据结构和 `GameScene` 中设置坦克的初始位置，我们将编写一个方法，即 `resetLocalTanksAndInfoToInitialState`。

    [PRE10]

1.  在这种方法中，局部数据结构 `tankStatsForLocal` 和局部玩家的属性位置、`zRotation` 使用唯一的 `tankStatsForLocal` 被设置为游戏的初始状态。远程玩家的位置和 `zRotation` 在游戏的初始状态中硬编码。所有这些设置都是基于在设备上谁属于蓝色和红色，谁是本地和远程坦克。

1.  要动画化玩家身份文本，添加一个方法 `hideGameInfoLabelWithAnimation`。

    [PRE11]

1.  使用精灵 `SKAction` 创建一系列动画，最初有延迟，然后淡出标签，最后使用回调将其移除。这是在用户连接时完成的。对于标签的淡入淡出，我们使用之前在 [第 3 章](part0025_split_000.html#page "第 3 章。动画和纹理") 中使用的相同动画代码，*动画和纹理*。

1.  现在，编辑 `MCSession` 的 `didChangeState` 代理方法，当状态变为连接时，如以下所示：

    [PRE12]

1.  在接收`MCSession`方法中，在`switch`案例中添加所有`NetworkPacketCode`，并在接收到`KNetworkPacketCodePlayerAllotment`数据包时进行更改。在分配玩家时，根据相应的身份名称设置，并根据`gameUniqueIdForPlayerAllocation`分配本地和远程精灵对象。最后，调用一个私有方法`resetLocalTanksAndInfoToInitialState`，在其中设置本地和远程精灵及其本地数据结构的初始状态。所有这些步骤将在会话中的连接设备上执行，这将确保两个设备保持同步。

1.  现在一旦设置了显示玩家身份的`gameInfoLabel`，然后在`GameScene`的`startGame`方法中插入代码以改变游戏状态，并通过`MCBrowerViewController`的代理方法来隐藏`gameInfoLabel`，就可以通过隐藏来动画化游戏状态。

    [PRE13]

1.  当该方法由代理方法调用时，它会检查玩家是否已经分配，然后更改状态为播放，并动画化隐藏标签，标签上写着**你是蓝色**或**你是红色**。

在添加所有这些游戏资产之后，其环境已经设置好，玩家的身份或可以说名字，已经被分配为蓝色和红色。所有这些构成了本章的入门套件。

## 如何工作

整个部分都是关于使用SpriteKit作为添加游戏资源的方式，以及作为多人游戏的一部分，玩家被分配为蓝色和红色。`GameScene`上节点添加的工作部分已经在[第11章](part0065_split_000.html#page "第11章。开始多人游戏")中解释过，即*开始多人游戏*，并且由于这些添加的结果，游戏在两个设备上看起来如下：

![如何工作](img/00177.jpeg)

在多人游戏部分，当用户按下**完成**按钮时，连接建立，两名玩家都连接上了。此外，当用户按下**完成**按钮时，已经为玩家分配了名称，如以下图像所示，两种设备上均为蓝色和红色。这些标签位于两个不同的设备上，并且将在分配时进行动画：

![如何工作](img/00178.jpeg)

# 玩家的移动

一旦多人游戏的环境准备就绪，就是实际操作的时候了：当用户触摸或拖动屏幕时让玩家移动。由于这是一个多人游戏，移动需要与远程设备同步，这是一个很好的挑战，也是一次神奇的经历。

## 准备工作

在开始本节之前，我们应该了解`SKScene`的触摸方法，如`touchBegan`、`touchMoved`和`touchEnded`，因为所有的移动都将基于触摸进行。此外，我们还应该擅长数学，因为坦克的移动需要使其头部指向触摸方向，并且需要与远程设备同步。在本节中，我们将实现坦克的移动，并通过发送和接收网络数据包来同步其他设备上的移动。

## 如何实现

以下是在通过发送和接收数据包使坦克在触摸时移动并在远程设备上同步的步骤：

1.  在触摸屏幕时，会调用`SKScene`的这些方法，这些方法将用于在本地设备上移动坦克。

    [PRE14]

1.  第一个方法已经实现。让我们在游戏状态为`kGameStatePlaying`时，在触摸开始时添加一些更新的坦克信息，如下所示：

    [PRE15]

1.  在通过`touch`方法接收的事件中，获取了一个`UITouch`对象，并使用它来确定触摸位置相对于`GameScene`的位置。然后，如果触摸的`tapCount`为`0`，即用户正在拖动代码以移动，则执行玩家。在这段代码中，`tankStatsForLocal`根据触摸在屏幕上的位置进行更新。目的地设置为触摸位置，方向通过使用向量数学，结合触摸点和坦克的当前位置来计算。为了保持坦克的方向角度在0到359度之间，一旦计算出方向，就需要进行额外的检查。完成所有这些后，为了更新坦克的实际位置和旋转，将实现一个名为`updateLocalTank`的方法，我们将在稍后讨论它。

1.  在`GameScene`中实现另外两个触摸方法，即`touchMoved`和`touchEnded`。

1.  在`touchesMoved`中，如果游戏状态是`kGameStatePlaying`，则执行与`touchesBegan`方法相同的代码，如前一点所述。

    [PRE16]

1.  在`touchesEnded`中，我们不应该做其他两个触摸方法中所做的；在这里，我们只更新`tankDestination`和`tankDirection`到本地数据结构中，然后调用相同的`updateLocalTank`方法来更新最终的位置和旋转。

    [PRE17]

1.  在所有触摸方法中，都会调用`updateLocalTank`方法来更新最终的位置和旋转。在为本地坦克更新这些属性后，将发送一个网络数据包以与远程坦克玩家同步。

    [PRE18]

    初始时，检查目的地向量与`kTankSpeed`的差异不要超过。然后，通过`tankStatsForLocal`的目的地更新位置，如果差异大于，则编写代码进行转弯；即计算坦克方向和`tankRotation`之间的角度差异。

1.  如果差异大于`kTankTurnSpeed`，则找到最近的180度旋转，并根据该旋转减去或加上`kTankTurnSpeed`以调整旋转。如果差异不大于面向，则围绕线移动到目的地。将旋转设置为方向，并使用当前位置、目的地和`kTankSpeed`计算坦克的位置。

    所有这些计算都应该分配给`tankStatsForLocaldata`结构。完成所有这些后，将本地数据结构的`tankPreviousPosition`设置为本地玩家的当前精灵位置。更新`tankStatsForLocalstructure`中计算的位置和旋转。为了同步由该方法产生的玩家移动，我们需要向其他玩家发送一个包含`NetworkPacketCode`为`KNetworkPacketCodePlayerMove`的数据包，数据部分将在`tankStatsForLocal`结构中，并且这个数据包应该不可靠地发送，因为它发送得非常频繁。

1.  在`MCSessiondidReceiveData`的委托方法中，接收到了类型为`KNetworkPacketCodePlayerMove`的玩家移动数据包。

    [PRE19]

1.  在这里，`TankInfo`是包含用户所有位置和旋转相关数据的结构。`TankInfo`结构通过网络发送以同步两个设备。

数据在`TankInfo`变量`ts`中解析，该变量包含发送的坦克位置和旋转。因此，它是远程坦克的数据，所以使用接收到的属性更新它。结果，我们将在远程设备中看到坦克移动，就像用户在另一台设备中驾驶坦克一样。

## 如何工作

坦克的移动是通过向量数学实现的，使用用户触摸的点、坦克在触摸时的朝向等因素。我们最终在我们的游戏中实现了多人行为，其中坦克根据设备中本地坦克的移动进行远程移动，这可以在以下快照中看到。远程设备的同步完全依赖于网络。

如果网络较弱，用户可能会在设备之间同步时遇到一些延迟。

![如何工作](img/00179.jpeg)

# 实现游戏玩法

现在是时候实现游戏玩法了，在这个游戏中，玩家（坦克）将如何赢得或输掉名为TankRace的游戏将得到确定。游戏玩法是这样的，哪个玩家首先到达他们自己颜色相同的一侧的终点线，就赢得比赛，而其他玩家则输掉。

## 准备工作

在我们开始之前，我们必须知道如何从SpriteKit中的精灵检测碰撞，以及一些关于SpriteKit标签的玩法。在本节中，我们将实现游戏玩法，并在设备之间没有连接时显示一个警告。

## 如何实现

下面是实现游戏玩法和决定谁赢谁输所涉及的步骤：

1.  我们将通过使用名为`CGRectIntersectsRect`的方法在`GameScene`的更新方法中检测碰撞。在此方法中，我们可以传递两个节点的框架，它将检查这两个框架是否相互交叉。通过这个检查，我们只为本地玩家碰撞进行检查，如果发生碰撞，则更新游戏状态为`kGameStateComplete`，并显示到达终点的本地玩家**你赢了**。此外，由于游戏已经结束，为了自动启动游戏，调用一个名为`restartGameAfterSomeTime`的方法，我们将在继续进行时了解它。

1.  之后，本地玩家将显示正确的结果，游戏将重新启动，但由于这是一个多人游戏，碰撞的反应也应该反映在另一设备上。因此，传递一个带有名为`KNetworkPacketCodePlayerLost`的`NetworkPacketCode`的数据包，该数据包将被发送给已输掉游戏的另一玩家。以下是实现此功能的代码：

    [PRE20]

1.  当前面的游戏丢失的数据包发送给另一玩家时，会调用名为`MCSession`的代理方法`didReceiveData`，数据包的头部有一个名为`KNetworkPacketCodePlayerLost`的`NetworkPacketCode`。当接收到此数据包时，游戏状态将变为`kGameStateComplete`，游戏信息标签显示为**你输了**，以通知其他用户游戏失败。此外，我们调用一个名为`restartGameAfterSomeTime`的方法，该方法将游戏重置为其初始状态，玩家可以再次开始游戏。

    [PRE21]

    使用这两段代码后，大约一名玩家获胜，另一名玩家失败，两个设备上的游戏看起来如下所示：

    ![如何操作](img/00180.jpeg)

1.  由于`restartGameAfterSomeTime`在发送和接收数据包时使用，让我们按照以下代码所示编写此方法：

    [PRE22]

1.  在此方法中，将触发一个2.0秒的`NSTimer`，调用一个名为`restartGame`的函数，在此函数中，`gameUniqueIdForPlayerAllocation`将再次生成，游戏状态设置为`kGameStatePlayerToConnect`，标签文本更改为**点击连接**。为了视觉初始状态，我们调用名为`resetLocalTanksAndInfoToInitialState`的方法，在此方法中，设置本地数据结构和坦克的视觉属性。

1.  每当游戏完成`restartGameAfterSomeTime`方法时，在本地和远程设备上都会调用此方法，以设置游戏的初始状态，看起来如下所示：![如何操作](img/00181.jpeg)

1.  有时，由于网络弱，连接可能会丢失，屏幕会卡住，因此不会向用户显示任何消息。因此，我们将添加一个警告，说明存在网络问题，并且要玩游戏，请在两个设备上重新启动您的应用，如下所示的方法：

    [PRE23]

1.  要显示网络已断开连接的警告，当 `MCSession` 的代理方法 `didChangeState` 被调用，并且状态变化为 `MCSessionStateNotConnected` 时，调用 `showNetworkDisconnectAlertView`。它应该在当前游戏状态为 `kGameStatePlaying` 时显示。

    [PRE24]

1.  每当游戏断开连接时，`showNetworkDisconnectAlertView` 被调用，并显示如图所示的警告视图：![如何操作](img/00182.jpeg)

在实现所有这些游戏逻辑之后，或者说游戏机制之后，我们已经完成了制作多人游戏，这是本章的解决方案套件。

## 它是如何工作的

本节全部关于游戏玩法和游戏机制；它分为两部分，一是检测碰撞以决定胜者，二是游戏结束后重置游戏到初始状态。

为了实现这一点，我们使用框架交集方法检测碰撞，并通过发送数据包将本地玩家宣布为胜者，其他玩家为败者。由于游戏在这里结束，为了帮助用户重新开始游戏，在宣布胜者和败者的同时，我们也重置游戏到其初始状态，以便玩家可以再次开始游戏。

## 更多内容

我们使用了多点连接框架，尽管我们也可以使用 GameKit 框架；有关更多信息，请使用以下链接：

[https://developer.apple.com/library/ios/documentation/GameKit/Reference/GameKit_Collection/index.html](https://developer.apple.com/library/ios/documentation/GameKit/Reference/GameKit_Collection/index.html)

## 参见

为了更好地理解和学习多点连接框架，您可以访问以下链接：

[https://developer.apple.com/library/prerelease/ios/documentation/MultipeerConnectivity/Reference/MultipeerConnectivityFramework/index.html](https://developer.apple.com/library/prerelease/ios/documentation/MultipeerConnectivity/Reference/MultipeerConnectivityFramework/index.html)
