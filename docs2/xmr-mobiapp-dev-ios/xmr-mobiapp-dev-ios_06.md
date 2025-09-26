# 第六章 事件

事件是任何 iOS 应用程序的一个基本方面。实际上，没有它们，你的手机将像一块无用的塑料一样静静地待在那里！每当发生某事时，都会引发一个事件。对于开发者来说，事件就是一切。

在本章中，我们将讨论以下主题：

+   处理事件

+   事件引用

+   控制事件引用

# 处理事件

事件处理使用以下类型之一：

+   代表

+   处理程序

## 代表

你可以将代表事件视为内联事件。它们可以是匿名的，或者可以像以下这样使用事件：

+   当代表是匿名的，代码如下：

    ```swift
    var uiButton = new UIButton();
    uiButton.TouchUpInside += delegate {…};
    ```

+   当代表使用事件时，代码如下：

    ```swift
    var uiButton = new UIButton();
    uiButton.TouchUpInside +=(object sender, EventArgs e) => {…};
    ```

如果按钮没有需要发送者或事件的任何内容，可以使用匿名事件。

Xamarin.iOS 在未使用处理程序时将所有事件委托给使用`EventArgs e`。

### 将事件附加到多个控件

为了方便起见，我将假设一个视图控制器有四个按钮。为每个按钮分配它自己的事件处理程序没有问题，但如果一个事件可以处理多个事件，那么这将是内存的浪费。然而，出于理智的考虑，应该限制这种情况，以便`TouchUpInside`事件一起处理，而不是在接受`TouchUpInside`事件的视图（或视图控制器）上处理所有事件。

一个简单的解决方案是为所有按钮使用单个事件。然而，问题在于如何在事件处理程序本身中识别和操作正确的按钮。

可能最简单的方法是使用数字设置`Tag`属性，如下面的代码所示：

```swift
private void setup()
{
    UIButton btn1 = new UIButton()
    {
        // set up the properties
        Tag = 1,
    };
    UIButton btn2 = new UIButton()  
    {
        // set up the properties
        Tag = 2,
    };
    UIButton btn3 = new UIButton()
    {
        // set up the properties
        Tag = 3, 
    };
    btn1.TouchUpInside += HandleButtonPressedEvent;
    btn2.TouchUpInside += HandleButtonPressedEvent;
    btn3.TouchUpInside += HandleButtonPressedEvent;
}
private void HandleButtonPressedEvent(object sender, EventArgs e)
{
    UIButton theButton = (UIButton)sender;
    switch(theButton.Tag)
    {
        // do what is needed
    }
}
```

## 同步与异步事件处理

事件有两种流程方法：同步和异步。它们之间的区别与理解它们的用途一样重要。为了理解这些区别，你需要想象两个人在散步。

### 同步行走

两个人一起走，来到路上的一盏交通灯前；他们都停下来，当交通灯变绿时，他们一起出发。散步结束时，他们仍然在一起，坐在阳光下喝酒。

### 异步行走

两个人一起出发去酒吧，但当他们来到交通灯前时，其中一个人停下来等待信号，而另一个人过马路。一旦信号允许安全过马路，停下来的人过马路并继续前进。刚刚过马路的人因为乱穿马路被警察拦下。无法保证这对夫妇会同时到达酒吧；这取决于其他使他们减速的因素。

### 在编程环境中

考虑一个简单的信使应用程序，它由两部分组成：获取消息和显示它们。同步事件是用于调用地址视图控制器。在这里，点击按钮可以被视为两个人一起散步并停下来。

异步部分是消息的下载；在这里，慢的部分是从服务器获取消息。如果这是通过同步事件执行的，那么将请求消息，而线程会在从服务器获取返回请求时冻结。当这个过程中发生时，UI 被创建，但没有数据如何创建 UI？简而言之，这是不可能的，应用会崩溃。由于它是异步执行的，UI 的创建直到服务器发送了所有内容才会发生。当发生这种情况时，下一个阶段可以执行。

例如，考虑以下代码（这段代码直接取自 Xamarin ZXing 组件，并通过创意共享许可证发布）：

```swift
buttonCustomScan.TouchUpInside += async (sender, e) =>
{
    //Create an instance of our custom overlay
    customOverlay = new CustomOverlayView();
    //Wireup the buttons from our custom overlay
    customOverlay.ButtonTorch.TouchUpInside += delegate {
        scanner.ToggleTorch();
    };
    customOverlay.ButtonCancel.TouchUpInside += delegate
    {
        scanner.Cancel();
    };

    //Tell our scanner to use our custom overlay
    scanner.UseCustomOverlay = true;
    scanner.CustomOverlay = customOverlay;
    var result = await scanner.Scan();
    HandleScanResult(result);
};
```

代码的重要部分是这一行 `var result = await scanner.Scan();`，直到它返回，`HandleScanResult` 方法将不会被调用。在 `async` 处理器内部的 `TouchUpInside` 事件是同步的，一旦点击，它们要么终止处理器，要么启动另一段代码——它们都立即执行。

由于 Mono 3 对其的支持，Xamarin.iOS（以及 Xamarin.Android）支持异步调用。

# 事件和控件参考

并非每个小部件默认都附加有事件（尽管你可以编写事件并将其添加到小部件中），例如，`UIImageView` 没有任何附加的事件。

iOS 非常重视其触摸系统，这在分配给触摸的事件数量上得到了体现。除非小部件没有附加事件，否则以下表格适用于所有小部件：

| 事件名称 | 执行动作及时机 |
| --- | --- |
| `TouchCancel` | 当取消控制器的当前触摸时被调用（任何处理 `TouchCancel` 事件的控制器）。 |
| `TouchDown` | 这是小部件上的触摸事件。 |
| `TouchDownRepeat` | 这是一个重复的触摸事件。要触发此事件，`UITouch` 的 `tapCount` 值必须大于 1。 |
| `TouchDragEnter` | 当手指被拖入控制范围时被调用。 |
| `TouchDragExit` | 当手指被拖出控制范围时被调用。 |
| `TouchDragInside` | 当手指被拖入控制范围时被调用。 |
| `TouchDragOutside` | 当手指被拖到控制范围外时被调用。 |
| `TouchUpInside` | 当手指在控制范围内时被调用。 |
| `TouchUpOutside` | 当手指在控制范围外时被调用。 |

## 其他重要的控制事件

下表中列出的事件可以在 iOS 中的许多控件上找到：

| 事件名称 | 执行动作及时机 |
| --- | --- |
| `ValueChanged` | 当控制器的值发生变化时（例如，`UITextField` 的文本被更改），允许访问控制器上的更改值。 |
| `EditingDidBegin` | 当在`UITextField`类中开始编辑会话时触发。 |
| `EditingChanged` | 当`UITextField`类的值发生变化时发出。 |
| `EditingDidEnd` | 当通过离开边界结束`UITextField`类的编辑会话时触发。 |
| `EditingDidEndOnExit` | 当通过触摸结束`UITextField`类的编辑会话时触发。 |
| `AllTouchEvents` | 它拦截所有触摸事件。 |
| `AllEditingEvents` | 它拦截所有编辑事件。 |
| `AllEvents` | 它拦截所有事件（包括系统事件）。 |

`TouchesBegan`、`Moved`、`Ended`和`Cancelled`以及手势识别器事件将在下一章中介绍。

### AVAudioPlayer 和 AVRecordClass

播放和录制音频和视频是 iOS 体验的关键功能。以下表格列出了您需要挂钩以实现流畅播放和录制的事件：

| 事件名称 | 功能及触发时间 |
| --- | --- |
| `BeginInterruption` | 当音频播放器被中断（例如，接听电话）时触发。 |
| `DecodeError` | 当文件无法解码时触发。 |
| `EndInterruption` | 当中断完成时触发。 |
| `FinishedPlaying` | 当文件播放完毕时触发。 |

### AVAudioSession

音频会话是播放音频或视频的物理行为。许多事件可以中断正在播放的视频或音频。以下表格列出了您需要了解的事件：

| 事件名称 | 功能及触发时间 |
| --- | --- |
| `BeginInterruption` | 当音频会话被中断时触发。 |
| `CategoryChanged` | 当音频会话类别发生变化时触发。 |
| `EndInterruption` | 当音频会话的中断完成时触发。 |
| `InputAvailabilityChanged` | 当设备上音频输入的可用性发生变化时触发。 |
| `InputChannelsChanged` | 当输入通道数量发生变化时触发。 |
| `OutputChannelsChanged` | 当输出通道数量发生变化时触发。 |
| `SampleRateChanged` | 当采样率改变时触发。 |

### ABAddressBook

iOS 中的地址簿可以从地址簿功能内部和外部进行修改。

| 事件名称 | 功能及触发时间 |
| --- | --- |
| `ExternalChange` | 当地址簿外部（例如，更改地址簿的用户应用程序）更改地址簿内部的内容时触发。 |

### ABNewPersonViewController

与所有视图一样，视图控制器用于放置 UI。地址簿`NewPersonViewController`也不例外。

| 事件名称 | 功能及触发时间 |
| --- | --- |
| `NewPersonComplete` | 当用户点击**保存**或**取消**按钮时调用。如果点击**保存**，则新联系人将被保存到地址簿数据库中。 |

### ABPeoplePickerNavigationController

`ABPeoplePickerNavigationController`的行为与普通导航控制器相同，但增加了一些功能，如下表所示：

| 事件名称 | 执行的操作及时间 |
| --- | --- |
| `Cancelled` | 当点击**取消**按钮时发送。 |
| `PerformAction` | 当在人员选择器上选择一个对象时触发。 |
| `SelectPerson` | 当选择一个人时触发。 |

### ABPersonViewController

**人员视图控制器**显示所选人员，并附加一个事件。

| 事件名称 | 执行的操作及时间 |
| --- | --- |
| `PerformDefaultAction` | 当用户在“人员视图控制器”中选择一个人的属性值时发送。 |

### ABUnknownPersonViewController

**未知人员视图** **控制器**在人员被接受到`ABAddressBook`类之前显示人员信息。其事件列在以下表中：

| 事件名称 | 执行的操作及时间 |
| --- | --- |
| `PerformDefaultAction` | 当用户在“人员视图控制器”中选择一个人的属性值时发送。 |
| `PersonCreated` | 当用户点击**保存**或**取消**按钮时调用。如果点击保存，则新联系人将保存到地址簿数据库中。 |

### AudioConverter

`AudioConverter`类用于转换音频格式。

| 事件名称 | 执行的操作及时间 |
| --- | --- |
| `InputData` | 当通过端口提供数据作为输入时触发。 |

### AudioSession

`AudioSession`类类似于`AVAudioSession`类，但它仅用于音频。

| 事件名称 | 执行的操作及时间 |
| --- | --- |
| `AudioRouteChanged` | 当输出路由更改时触发（例如，扬声器到耳机）。 |
| `Interrupted` | 当音频会话被中断时触发。 |
| `Resumed` | 当音频会话的中断完成时触发。 |

### InputAudioQueue

当音频被输入到设备时使用输入队列。

| 事件名称 | 执行的操作及时间 |
| --- | --- |
| `InputCompleted` | 当输入完成时触发。 |

### OutputAudioQueue

输出队列用于音频输出。

| 事件名称 | 执行的操作及时间 |
| --- | --- |
| `OutputCompleted` | 当音频输出完成时触发。 |

### AUGraph 和 AudioUnit

`AUGraph`和`AudioUnit`共享此事件，对两者都有相同的效果。

| 事件名称 | 执行的操作及时间 |
| --- | --- |
| `RenderCallback` | 在音频图渲染期间使用的回调。 |

### AudioConverter

回调事件用于将对象从事件回调到对象。

| 事件名称 | 执行的操作及时间 |
| --- | --- |
| `EncoderCallback` | 为转换编码器创建的回调。 |

### CAAnimation

动画有两个关键事件；开始和结束。

| 事件名称 | 执行的操作及时间 |
| --- | --- |
| `AnimationStarted` | 当动画开始时调用。 |
| `AnimationEnded` | 当动画结束时被调用。 |

### CBCentralManager

The `CoreBluetoothCentralManager` 是处理添加或移除蓝牙设备的类。处理事件已在以下表中列出：

| 事件名称 | 它做什么以及何时发生 |
| --- | --- |
| `ConnectedPeripheral` | 当连接成功建立时被触发。 |
| `DisconnectedPeripheral` | 当连接成功断开时被触发。 |
| `DiscoveredPeripheral` | 当发现连接时被触发。 |
| `FailedToConnectPeripheral` | 当连接外围设备失败时被触发。 |
| `RetrievedConnectedPeripheral` | 在检索到已连接的外围信息后被触发。 |
| `RetrievedPeripherals` | 在请求所有存储的外围列表之后被触发。 |
| `UpdatedState` | 在连接状态改变后调用。 |

### CBPeripheral

每个蓝牙设备都附有特性和描述符。事件已在以下表中列出：

| 事件名称 | 它做什么以及何时发生 |
| --- | --- |
| `DiscoverCharacteristic` | 发现外围服务的特性。 |
| `DiscoveredDescriptor` | 发现特性的描述符。 |
| `DiscoveredIncludedService` | 它发现外围指定的包含服务。 |
| `DiscoveredService` | 在发现外围指定的服务并完成连接后调用。 |
| `InvalidatedService` | 当外围服务改变时被触发。 |
| `RssiUpdated` | 当外围连接到 `CoreBluetooth` 中心管理器时，其当前 **RSSI** 值被更新时被触发。 |
| `UpdatedCharacteristicValues` | 在更新特性的值之后被触发。 |
| `UpdatedName` | 当外围名称改变时被触发。 |
| `UpdatedNotificationState` | 当外围通知状态改变时被触发。 |
| `UpdatedValue` | 当外围特性描述符的值改变时被触发。 |
| `WroteCharacteristicValue` | 它在写入特性的值之后被触发。 |
| `WroteDescriptorValue` | 在写入描述符的值之后被触发。 |

### CBPeripheralManager

The `CBPeripheralManager` 类管理连接到蓝牙管理器的外围设备。

| 事件名称 | 它做什么以及何时发生 |
| --- | --- |
| `AdvertisingStarted` | 当外围开始广播其存在时被触发。 |
| `CharacteristicSubscribed` | 当远程设备订阅特性的值时被触发。 |
| `CharacteristicUnsubscribed` | 当远程设备取消订阅特性的值时被触发。 |
| `ReadRequestReceived` | 在接收到读取请求后被触发。 |
| `ReadyToUpdateSubscribers` | 当本地外围准备好发送特性更新值时被调用。 |
| `ServiceAdded` | 添加服务后，会触发此事件。 |
| `StateUpdated` | 当外围设备状态更新后，会触发此事件。 |
| `WriteRequestsReceived` | 接收到写请求后，会触发此事件。 |

### CFSocket

`CoreFoundation` 套接字覆盖了到远程套接字的连接（通常是网络连接）。

| 事件名称 | 功能及时间 |
| --- | --- |
| `AcceptEvent` | 当 `CoreFoundation` 设置为接受来自套接字的事件时，会调用此方法。 |
| `ConnectEvent` | 当客户端连接到它所调用的远程套接字时，会触发此事件。 |

### CFStream

类似于 **IOStream**，**CFStream** 处理来自和到套接字的数据。

| 事件名称 | 功能及时间 |
| --- | --- |
| `CanAcceptBytesEvent` | 当流中有信息可供写入时，会触发此事件。 |
| `ClosedEvent` | 当流上的关闭操作完成时，会触发此事件。 |
| `ErrorEvent` | 当流中发生错误时，会触发此事件。 |
| `HasBytesAvailableEvent` | 当流中有信息可供读取时，会触发此事件。 |
| `OpenCompletedEvent` | 当流上的打开操作完成时，会触发此事件。 |

### CLLocationManager

The `CoreLocation` `LocationManager` 类是 iOS 设备上位置管理器的控制类。

| 事件名称 | 功能及时间 |
| --- | --- |
| `AuthorizationChanged` | 当用户允许或阻止使用 `CoreLocation` 函数时，会调用此方法。 |
| `DeferredUpdatesFinished` | 当延迟更新时间结束时，会触发此事件。 |
| `DidStartMonitoringForRegion` | 通知代理正在监控一个新的区域。 |
| `Failed` | 当 `CLLocationManager` 无法启动时，会触发此事件。 |
| `LocationUpdatesPaused` | 它暂停更新位置。 |
| `LocationUpdatesResumed` | 它重新启动暂停的位置更新。 |
| `LocationsUpdated` | 它更新 GPS 位置。 |
| `MonitoringFailed` | 当 `CLLocationManager` 无法监控时，会触发此事件（通常是由于与卫星没有通信）。 |
| `RegionEntered` | 当进入一个区域时，会调用此方法。 |
| `RegionLeft` | 当离开一个区域时，会调用此方法。 |
| `UpdatedHeading` | 当航向更新时，会调用此方法。 |
| `UpdatedLocation` | 当位置更新时，会调用此方法。 |

### MidiClient

在 iPhone 上可能不太有用（尽管在 iPad 上有用），iOS 具有丰富的 **MIDI** 层。

| 事件名称 | 功能及时间 |
| --- | --- |
| `IOError` | 当 MIDI 客户端遇到输入/输出错误时，会触发此事件。 |
| `ObjectAdded` | 当添加 MIDI 设备时，会调用此方法。 |
| `ObjectRemoved` | 当移除 MIDI 设备时，会调用此方法。 |
| `PropertyChanged` | 当设备的 MIDI 属性更改时，会调用此方法。 |
| `SerialPortOwnerChanged` | 当串行端口所有者更改时，会调用此方法（实际上是在设备切换时调用）。 |
| `SetupChanged` | 当客户端设置更改时，会触发此事件。 |
| `ThruConnectionsChanged` | 当 MIDI 菊花链连接改变时被触发。 |

### MidiEndpoint 和 MidiPort

这两个类都有相同名称的事件，并且在两个类中都执行完全相同的操作！

| 事件名称 | 功能及时间 |
| --- | --- |
| `MessageReceived` | 当 MIDI 子系统从设备接收到消息时被触发。 |

### Monotouch.Dialog

`MonoTouch.Dialog`是一个功能极其强大的类，它极大地简化了在您的应用中创建和使用`UITableViews`的麻烦。

### BadgeElement, BaseBooleanImageElement, GlassButton, LoadMoreElement, MessageElement, 和 StringElement

这些类都包含`Tapped`事件。它在选项被点击时执行。请注意，它发送一个`NSAction`事件而不是典型的对象发送器/`EventArgs e`组合，这意味着你不能为类的多个实例使用相同的处理程序。

| 事件名称 | 功能及时间 |
| --- | --- |
| `Tapped` | 当元素被点击时触发。 |

### BoolElement

这是一个简单的开/关元素。

| 事件名称 | 功能及时间 |
| --- | --- |
| `ValueChanged` | 当布尔值改变时触发。 |

### DateTimeElement

当调用时，会生成一个标准的**日期时间选择器视图**。

| 事件名称 | 功能及时间 |
| --- | --- |
| `DateSelected` | 当日期被选中时被调用。 |

### DialogViewController

`DialogViewController`是放置`MT.D`的视图控制器。

| 事件名称 | 功能及时间 |
| --- | --- |
| `OnSelection` | 当 DVC 内的对象被选中时被调用。 |
| `RefreshRequested` | 当调用`MT.D`刷新时被调用。 |
| `SearchTextChanged` | 当搜索文本字段文本改变时被调用。 |
| `ViewAppearing` | 当`MT.D`视图正在创建时被触发。 |
| `ViewDisappearing` | 当`MT.D`视图正在销毁时被触发。 |

### EntryElement

一个条目元素允许在`MT.D`单元格中持有的`UITextField`进行数据输入。

| 事件名称 | 功能及时间 |
| --- | --- |
| `Changed` | 当`EntryElement`内容被改变时被调用。 |
| `ShouldReturn` | 询问文本字段是否应该处理**返回**按钮。 |

### StyledStringElement

`StyledStringElement`类与`StringElement`类相同，不同之处在于你可以为其添加样式。

| 事件名称 | 功能及时间 |
| --- | --- |
| `AccessoryTapped` | 当元素被点击时触发。 |

### EKCalendarChooser

**EventKit**的`CalendarChooser`类允许用户访问日历。

| 事件名称 | 功能及时间 |
| --- | --- |
| `Cancelled` | 当用户取消选择时被调用。 |
| `Finished` | 当用户选择**完成**时被调用。 |
| `SelectionChanged` | 当选定的日期改变时被调用。 |

### EKEventEditViewController 和 EKEventViewController

这两个 EventKit 视图控制器都有相同名称的事件，对两个控制器都有相同的效果。

| 事件名称 | 功能及时间 |
| --- | --- |
| `Completed` | 当控制完成其操作时发出。 |

### EAAccessory

外部配件类处理不属于手机或蓝牙的任何配件。

| 事件名称 | 功能及时间 |
| --- | --- |
| `Disconnected` | 当外部配件断开连接时发出。 |

## NS 类

**NS** 代表 **NextStep**。它们是一系列类，在史蒂夫·乔布斯从荒野归来后，苹果收购 NeXT 时成为了一部分传承。虽然与它们关联的事件并不多，但它们至关重要。许多 NS 类是 Xamarin.iOS 与 Objective-C 底层之间的绑定所必需的。

### NSCache

NSCache 是一个内部缓存系统，用于许多不同的任务。

| 事件名称 | 功能及时间 |
| --- | --- |
| `WillEvictObject` | 当对象即将从缓存中移除时调用。 |

### NSKeyedArchiver

`KeyedArchiver` 类使用密钥编码数据。

| 事件名称 | 功能及时间 |
| --- | --- |
| `EncodedObject` | 当对象被编码时调用。 |
| `Finished` | 编码完成后触发。 |
| `Finishing` | 当编码即将完成时触发。 |
| `ReplacingObject` | 通知代理，给定对象将被另一个对象替换。 |

### NSKeyedUnarchiver

`KeyedUnarchiver` 类执行 `KeyedArchiver` 类的逆操作。

| 事件名称 | 功能及时间 |
| --- | --- |
| `Finished` | 当解码完成时触发。 |
| `Finishing` | 当解码即将完成时触发。 |
| `ReplacingObject` | 通知代理，给定对象将被另一个对象替换。 |

### NSNetService

这是用于网络服务的类。

| 事件名称 | 功能及时间 |
| --- | --- |
| `AddressResolved` | 当地址已解析时发出。 |
| `PublishFeature` | 通知服务功能已成功发布。 |
| `Published` | 通知服务已成功发布。 |
| `ResolveFailed` | 当地址无法解析时发出。 |
| `Stopped` | 通知请求发布或解析已停止。 |
| `UpdatedTxtRecordData` | 通知服务的一个 `TXT` 记录已被更新。 |
| `WillPublish` | 通知网络已准备好发布服务。 |
| `WillResolve` | 通知网络已准备好解析服务 |

### NSNetServiceBrowser

`NetServiceBrowser` 类用于连接外部世界。

| 事件名称 | 功能及时间 |
| --- | --- |
| `DomainRemoved` | 当域名消失或不再可用时通知。 |
| `FoundDomain` | 当发送者找到域名时触发。 |
| `FoundService` | 当发送者找到服务时触发。 |
| `NotSearched` | 当搜索未成功时触发。 |
| `SearchStarted` | 当搜索开始时，会触发此事件。 |
| `SearchStopped` | 当搜索停止时，会触发此事件。 |
| `SearchRemoved` | 当搜索从浏览器中移除时，会触发此事件。 |

### NSStream

这与标准的 .NET Stream 类似。

| 事件名称 | 它的作用和触发时间 |
| --- | --- |
| `OnEvent` | 当给定流上发生给定事件时，会调用此事件。 |

### GLKView

**GL KitView** 是用于 **OpenGL** 图形的视图。

| 事件名称 | 它的作用和触发时间 |
| --- | --- |
| `DrawInRect` | 在给定的矩形内绘制视图的内容。 |

### GK 类

游戏是任何用户体验的重要组成部分。如果你只想让手机成为一部手机，那没问题，但如果你有玩愤怒刺猬或足球经理游戏的能力，为什么不利用它们呢？

### GKAchievementViewController, GKFriendRequestComposeViewController, 和 GKLeaderboardViewController

这三个类具有相同名称的事件。

| 事件名称 | 它的作用和触发时间 |
| --- | --- |
| `DidFinish` | 当视图被关闭时，会调用此事件。 |

### GKGameCenterViewController

此视图控制器是主要游戏视图控制器。

| 事件名称 | 它的作用和触发时间 |
| --- | --- |
| `Finished` | 当玩家停止与视图控制器交互时，会调用此事件。 |

### GKMatch

`GKMatch` 类处理玩家的连接。

| 事件名称 | 它的作用和触发时间 |
| --- | --- |
| `DataReceived` | 当从玩家接收到数据时，会触发此事件。 |
| `Failed` | 当匹配无法连接到其他玩家时，会调用此事件。 |
| `StateChanged` | 当玩家连接或断开与匹配的连接时，会触发此事件。 |

### GKMatchmakerViewController

`MatchMaker` 视图控制器处理设备之间的匹配。

| 事件名称 | 它的作用和触发时间 |
| --- | --- |
| `DidFailWithError` | 当视图控制器遇到无法恢复的错误时，会触发此事件。 |
| `DidFindMatch` | 当找到对等方匹配时，会触发此事件。 |
| `DidFindPlayers` | 当找到托管匹配时，会调用此事件。 |
| `ReceivedAcceptFromHostedPlayer` | 当玩家接受托管匹配的邀请时，会触发此事件。 |
| `WasCancelled` | 当用户取消匹配请求时，会触发此事件。 |

### GKSession

`GKSession` 类可以是设备本地或外部的。

| 事件名称 | 它的作用和触发时间 |
| --- | --- |
| `ConnectionFailed` | 尝试连接到另一个对等方失败时，会触发此事件。 |
| `ConnectionRequest` | 尝试连接到另一个对等方时，会触发此事件。 |
| `Failed` | 当会话中发生严重错误时，会发送此事件。 |
| `PeerChanged` | 当对等方状态改变时，会收到此事件。 |
| `ReceiveData` | 当从对等方接收到数据时，会触发此事件。 |

### MKMapView

**MapKit** 的 `MapView` 类处理地图的创建和显示，以及将指针添加到视图上（如用户或兴趣点）。

| 事件名称 | 它的作用和触发时间 |
| --- | --- |
| `CalloutAccessoryControlTapped` | 当用户点击注释视图的附件之一（如按钮）时触发。 |
| `ChangedDragState` | 当注释视图的状态之一发生变化时触发。 |
| `DidAddAnnotationViews` | 当一个或多个注释视图被添加到地图上时调用。 |
| `DidAddOverlayViews` | 当一个或多个覆盖视图被添加到地图上时调用。 |
| `DidChangeUserTrackingModel` | 当用户跟踪模式更改时触发。 |
| `DidDeselectAnnotationView` | 当取消选择注释视图时调用。 |
| `DidFailToLocateUser` | 当尝试查找用户位置失败时触发。 |
| `DidSelectAnnotationView` | 当选择其中一个注释时调用。 |
| `DidStopLocatingUser` | 当定位用户服务停止时调用。 |
| `DidUpdateUserLocation` | 当用户位置更新时触发。 |
| `LoadingMapFailed` | 当地图加载失败时调用（通常由没有 GPS 连接引起）。 |
| `MapLoaded` | 当地图加载完成时触发。 |
| `RegionChanged` | 当区域发生变化时触发。 |
| `RegionWillChange` | 当区域正在变化时触发。 |
| `WillStartLoadingMap` | 当地图即将开始加载时触发。 |
| `WillStartLocatingUser` | 当用户即将被定位时触发。 |

### MPMediaPickerController

这些事件附加到 `MediaPickerController` 视图控制器。

| 事件名称 | 执行的操作及何时发生 |
| --- | --- |
| `DidCancel` | 当用户点击**取消**按钮时调用。 |
| `ItemsPicked` | 当用户选择了一组媒体项目时调用。 |

### MFMailComposeViewController 和 MFMessageComposeViewController

`MFMailComposeViewController` 和 `MFMessageComposeViewController` 都有此事件。它用于编写电子邮件或消息。

| 事件名称 | 执行的操作及何时发生 |
| --- | --- |
| `Finished` | 当组合完成时发出。 |

### PKAddPassesViewController

**passkit** 视图控制器用于存储密码。

| 事件名称 | 执行的操作及何时发生 |
| --- | --- |
| `Finished` | 在添加通行证视图控制器完成后触发。 |

### QLPreviewController

**QuickLook 预览** 控制器允许快速查看文件。

| 事件名称 | 执行的操作及何时发生 |
| --- | --- |
| `DidDismiss` | 在预览控制器关闭后调用。 |
| `WillDismiss` | 在预览控制器关闭之前调用。 |

## SK 类

**StoreKit** 类处理应用商店和从应用商店的在线购买。

### SKProductsRequest

此类处理从应用商店请求产品。

| 事件名称 | 执行的操作及何时发生 |
| --- | --- |
| `ReceivedResponse` | 当应用商店对产品请求做出响应时调用。 |
| `RequestFailed` | 当请求应用商店失败时触发。 |
| `RequestFinished` | 当产品请求关闭时触发。 |

### SKRequest

此类用于处理对商店的请求。

| 事件名称 | 它做什么以及何时做 |
| --- | --- |
| `RequestFailed` | 当请求应用商店失败时触发。 |
| `RequestFinished` | 当产品请求关闭时触发。 |

### SKStoreProductViewController

`StoreProductViewController`是用于应用商店内容的主要视图控制器。

| 事件名称 | 它做什么以及何时做 |
| --- | --- |
| `Finished` | 当产品请求关闭时触发。 |

## UIClasses

这些类专门处理用户界面事件，没有它们几乎什么也做不了。

### UIAccelerometer

加速计检测设备的移动。

| 事件名称 | 它做什么以及何时做 |
| --- | --- |
| `Acceleration` | 当检测到移动时触发。 |

### UIActionSheet 和 UIAlertView

这两个类在两者中都共享这些命名事件，并且具有相同的效果。

| 事件名称 | 它做什么以及何时做 |
| --- | --- |
| `Cancelled` | 当点击**取消**按钮时触发。 |
| `Clicked` | 当控件被选中时触发。 |
| `Dismissed` | 当`ActionSheet`或`AlertView`关闭时触发。 |
| `Presented` | 当`ActionSheet`或`AlertView`显示时触发。 |
| `WillDismiss` | 当`ActionSheet`或`AlertView`即将关闭时触发。 |
| `WillPresent` | 当`ActionSheet`或`AlertView`即将显示时触发。 |

### UIButtonBarItem

**按钮栏项**必须连接到`ButtonBar`才能工作。

| 事件名称 | 它做什么以及何时做 |
| --- | --- |
| `Clicked` | 当项目被点击时调用。 |

### UIImagePickerController

此类用于从相册中选择图片。

| 事件名称 | 它做什么以及何时做 |
| --- | --- |
| `Cancelled` | 当选择器被取消时触发。 |
| `DidShowViewController` | 当视图控制器已显示时触发。 |
| `FinishedPickingImage` | 当用户完成选择图片时触发。 |
| `FinishedPickingMedia` | 当用户完成媒体选择时触发。 |
| `WillShowViewController` | 在视图控制器显示之前触发。 |

### UIPageViewController

此类用作“虚拟”页面计数器的一种形式。

| 事件名称 | 它做什么以及何时做 |
| --- | --- |
| `DidFinishAnimating` | 页面滚动过渡完成后触发。 |
| `WillTransition` | 在页面滚动过渡开始之前触发。 |

### UIPopoverController

此类仅在 iPad 上可用。

| 事件名称 | 它做什么以及何时做 |
| --- | --- |
| `DidDismiss` | 当控制器被关闭时触发。 |

### UIPrintInteractionController

用于从设备打印。

| 事件名称 | 它做什么以及何时做 |
| --- | --- |
| `DidDismiss` | 当控制器被关闭时触发。 |
| `DidFinishJob` | 当打印机完成作业时触发。 |
| `DidPresentPrinterOptions` | 当打印机选项被显示时触发。 |
| `WillDismissPrinterOptions` | 当打印机选项被关闭时触发。 |
| `WillPresentPrinterOptions` | 当打印机选项即将显示时触发。 |
| `WillStartJob` | 当打印作业即将开始时触发。 |

### UIScrollView

滚动视图允许在页面上的内容更多，如果没有它，这些内容将无法适应页面。

| 事件名称 | 它的作用和触发时间 |
| --- | --- |
| `DecelerationEnded` | 当滚动减速停止时调用。 |
| `DecelerationStarted` | 当滚动减速开始时调用。 |
| `DidZoom` | 缩放发生后调用一次。 |
| `DraggingEnded` | 当拖动停止（手指从手机上移开）时调用。 |
| `DraggingStarted` | 当开始拖动（手指在手机上拖动）时调用。 |
| `ScrollAnimationEnded` | 当滚动完成后调用。 |
| `Scrolled` | 当滚动完成后调用。 |
| `ScrolledToTop` | 当完成滚动到视图顶部时触发。 |
| `WillEndDragging` | 当用户即将结束拖动时触发。 |
| `ZoomingEnded` | 当缩放完成后触发。 |
| `ZoomingStarted` | 当开始缩放时触发。 |

### UISearchBar

这是一个在应用内或设备上查找信息的搜索方法。

| 事件名称 | 它的作用和触发时间 |
| --- | --- |
| `BookmarkButtonClicked` | 当点击书签按钮时调用。 |
| `CancelButtonClicked` | 当点击**取消**按钮时调用。 |
| `ListButtonClicked` | 当点击**列表**按钮时调用。 |
| `OnEditingStarted` | 当`UITextField`视图边界被进入时调用。 |
| `OnEditingStopped` | 当`UITextField`视图边界被离开时调用。 |
| `SearchButtonClicked` | 当点击**搜索**按钮时调用。 |
| `SelectedScopeButtonIndexChanged` | 当范围按钮选择改变时调用。 |
| `TextChanged` | 当`UITextField`文本被更改时调用。 |

### UISplitViewController

分割视图允许将视图分割成部分（例如，当在按钮栏上按下菜单按钮时，左侧会出现菜单）。

| 事件名称 | 它的作用和触发时间 |
| --- | --- |
| `WillHideViewController` | 在隐藏视图控制器之前触发。 |
| `WillPresentViewController` | 在呈现视图控制器之前触发。 |
| `WillShowViewController` | 在显示视图控制器之前触发。 |

### UITabBar

可以使用标签导航的简单方法与`TabBarController`控制器一起使用。

| 事件名称 | 它的作用和触发时间 |
| --- | --- |
| `DidBeginCustomizingItems` | 在显示自定义模态视图后触发。 |
| `DidEndCustomizingItems` | 在自定义模态视图被关闭后触发。 |
| `ItemSelected` | 当标签栏项目被选择时被调用。 |
| `WillBeginCustomizingItems` | 在自定义模态视图被关闭前被触发。 |
| `WillEndCustomizingItems` | 在自定义模态视图被关闭前被触发。 |

### UITabBarController

This is a convenient method of controlling the NIBs called when a tab bar item is clicked.

| 事件名称 | 功能和时机 |
| --- | --- |
| `FinishedCustomizingViewController` | 当标签栏自定义表单被关闭时被触发。 |
| `OnCustomizingViewController` | 当自定义开始时被触发。 |
| `OnEndCustomizingViewController` | 当自定义结束时被触发。 |
| `ViewControllerSelected` | 当用户在标签栏上选择一个项目时被调用。 |

### UITextField

The `UITextField` class is a simple, editable textbox.

| 事件名称 | 功能和时机 |
| --- | --- |
| `Ended` | 当**TextField**的编辑结束时被触发。 |
| `Started` | 当**TextField**的内容被编辑时立即被触发。 |

### UITextView

The `UITextView` class displays text. It also inherits from the `ScrollView` view to enable more text than within the frame.

| 事件名称 | 功能和时机 |
| --- | --- |
| `Changed` | 当文本发生变化时被触发。 |
| `DecelerationEnded` | 当滚动减速停止时被调用。 |
| `DecelerationStarted` | 当滚动减速开始时被调用。 |
| `DidZoom` | 当缩放发生时被调用一次。 |
| `DraggingEnded` | 当拖动停止（手指从手机上移开）时被调用。 |
| `DraggingStarted` | 当拖动开始（手指在手机上拖动）时被调用。 |
| `Ended` | 在滚动完成后被调用。 |
| `ScrollAnimationEnded` | 当滚动完成时被调用。 |
| `Scrolled` | 当文本视图被滚动时被触发。 |
| `ScrolledToTop` | 当视图滚动到顶部完成时被触发。 |
| `SelectionChanged` | 当文本视图中的文本选择发生变化时被触发。 |
| `Started` | 当滚动开始时被触发。 |
| `WillEndDragging` | 当用户即将结束拖动时被触发。 |
| `ZoomingEnded` | 当缩放完成时被触发。 |
| `ZoomingStarted` | 当缩放开始时被触发。 |

### UIView

The `UIView` is a generic view that can be added to any View Controller.

| 事件名称 | 功能和时机 |
| --- | --- |
| `AnimationWillEnd` | 当动画即将结束时被触发。 |
| `AnimationWillStart` | 当动画即将开始时被触发。 |

### UIWebView

用于显示 HTML 的视图（无论是来自网站还是由应用生成）。

| 事件名称 | 功能和时机 |
| --- | --- |
| `LoadError` | 当 URL 加载失败时被触发。 |
| `LoadFinished` | 当 URL 加载完成时被触发。 |
| `LoadStarted` | 当 URL 加载开始时被触发。 |

## 广告类

通常，应用程序内的广告是开发者生成收入的一种便捷方法（被称为点击通过）。

### AdBannerView

`AdBannerView` 视图是包含广告的视图。

| 事件名称 | 执行的操作和时机 |
| --- | --- |
| `ActionFinished` | 当横幅视图完成覆盖 UI 的动作执行时被触发。 |
| `AdLoaded` | 当广告加载时被触发。 |
| `FailedToReceiveAd` | 当广告检索失败时被触发（通常是网络连接错误）。 |
| `WillLoad` | 当广告即将加载时被触发。 |

### AdInterstitialAd

这些是全屏广告。它们的行为与`AdBannerView`相同。

| 事件名称 | 执行的操作和时机 |
| --- | --- |
| `ActionFinished` | 当横幅视图完成覆盖 UI 的动作执行时被触发。 |
| `AdLoaded` | 当广告加载时被触发。 |
| `AdUnloaded` | 在全屏广告处理其内容后调用。 |
| `FailedToReceiveAd` | 当广告检索失败时被触发（通常是网络连接错误）。 |
| `WillLoad` | 当广告即将加载时被触发。 |

### OpenTK

**OpenTK** 是一个在移动和桌面应用程序开发者中广泛使用的开放图形层。

### IGameWindow

这是一个接口类，而不是一个类，它处理的是游戏窗口本身，而不是游戏。

| 事件名称 | 执行的操作和时机 |
| --- | --- |
| `Load` | 在窗口首次显示之前被触发。 |
| `RenderedFrame` | 当渲染时间到达时被触发。 |
| `Unload` | 当窗口被销毁时被触发。 |
| `UpdateFrame` | 当需要更新帧时被调用。 |

### IPhoneOSGameView

这是**OpenTK**类持有的`GameView`。

| 事件名称 | 执行的操作和时机 |
| --- | --- |
| `Closed` | 当游戏视图被关闭但未处置时调用。 |
| `Disposed` | 当游戏视图被处置时被调用。 |
| `Load` | 在运行循环开始之前被触发。 |
| `RenderedFrame` | 作为运行循环处理的一部分，当应该渲染帧时被触发。 |
| `Resize` | 当视图大小改变时被调用。 |
| `TitleChanged` | 当视图标题改变时被调用。 |
| `Unload` | 当运行循环结束时被触发。 |
| `UpdateFrame` | 当帧作为运行循环的一部分更新时被触发。 |
| `VisibleChanged` | 当视图的可见性发生变化时被调用。 |
| `WindowStateChanged` | 当窗口状态改变时被触发。 |

## 摘要

可以说，事件是使你的 iPhone 成为现在的设备的原因。你几乎不太可能需要这里列出的大多数事件，但如果你像我一样讨厌搜索，那么这一章将来真的会对你有所帮助。
