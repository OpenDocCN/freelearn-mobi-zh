# 第十二章：*第十二章*：使用位置服务改进应用程序

所有 iOS 设备都配备了各种芯片和传感器，可以用来增强用户体验。增强现实应用程序大量使用陀螺仪、加速度计和摄像头等传感器。如果您想拍照或想知道设备是如何移动的，这些传感器非常棒。其他应用程序需要不同的数据，例如用户在特定时间的 GPS 位置。在本章中，您将学习如何使用**Core Location**框架来实现这一点。

**Core Location**是一个框架，允许开发者访问用户的当前位置，但它还允许开发者跟踪用户是否进入或离开了特定区域，甚至可以监控用户随时间的变化位置。Core Location 的正确实现可以是您应用程序中许多优秀功能的基石，但糟糕的实现可能会迅速耗尽用户的电池。

在本章中，您将了解以下与位置相关的主题：

+   请求用户的地理位置

+   订阅位置变化

+   设置地理围栏

到本章结束时，您应该能够就如何在您的应用程序中实现 Core Location 做出明智的决定。

# 技术要求

本章的代码包包括一个名为**LocationServices**的入门项目。您可以在代码包仓库中找到它：

[`github.com/PacktPublishing/Mastering-iOS-14-Programming-4th-Edition/tree/master/Chapter%2012%20-%20Location%20Services`](https://github.com/PacktPublishing/Mastering-iOS-14-Programming-4th-Edition/tree/master/Chapter%2012%20-%20Location%20Services)

# 请求用户的地理位置

如您所想，允许应用程序访问您的确切位置是一件相当重要的事情。如果落入错误的手中，这些数据可能会让有恶意意图的人知道您在任何给定时间的确切位置，并以多种方式滥用这种知识。因此，如果您绝对需要，才请求用户的地理位置信息至关重要。仅仅为了实现一个小功能，或者确保用户在注册服务之前位于某个任意位置，可能并不总是请求用户地理位置信息的充分理由。

现在我们来看看在 iOS 中请求用户允许访问其位置数据的不同方法。

## 请求访问位置数据的许可

当您确实需要访问用户的地理位置信息时，您必须首先请求许可。类似于您需要在`Info.plist`文件中添加需要相机或用户联系人的原因一样，您还必须提供请求位置数据的原因。在位置数据的情况下，您可以在`Info.plist`中添加两个键：

+   隐私-位置 `NSLocationWhenInUseUsageDescription`)

+   隐私-位置 `NSLocationAlwaysAndWhenInUseUsageDescription`)

当你的应用请求使用用户的位置数据时，他们可以选择仅在应用使用时允许你的应用访问他们的位置，或者他们可以选择始终允许你的应用访问他们的位置，即使应用处于后台。你还可以配置你想要请求的访问类型。如果你只需要在用户使用应用时获取用户的位置，请确保正确配置你的权限请求，这样用户就不会在应用处于后台时被要求向你的应用提供位置信息。

在将所需的密钥添加到**LocationServices**应用的`Info.plist`文件后，你需要编写一些代码来请求用户允许使用他们的位置。在这样做之前，让我们快速检查一下示例项目的结构和内容，以便你知道信息可以在哪里找到。

首先，在项目中打开`Main.storyboard`文件。你将找到一个带有两个视图控制器的标签栏控制器。在本章中，你将实现功能以填充这些视图控制器，并添加适当的数据。接下来，查看`AppDelegate`的实现。这里的实现遵循`GeofenceViewController`，这是你将首先工作的，以便在屏幕上显示用户的当前位置。

你会注意到在这个视图控制器中已经实现了大量的代码。稍微检查一下现有代码，你会发现所有代码都调用了`LocationHelper.swift`中的空方法。在本章中，你将主要关注实现与用户位置数据一起工作的 Core Location 代码，因此 UI 工作已经设置好了。当你向`LocationHelper`添加代码时，你会发现`LocationServices`的用户界面将逐步变得生动起来。

既然你对如何设置 LocationServices 应用有了更好的理解，让我们看看请求用户允许使用其位置所需的步骤。由于此应用最终将在后台跟踪位置变化，因此即使在应用处于后台时，你也应该请求用户允许访问其位置。为此，将以下`viewDidAppear(_:)`代码添加到`GeofenceViewController`中：

```swift
locationHelper.askPermission { [weak self] status in if status == .authorizedAlways {
  self?.showCurrentLocation()
  } else {
    // handle the case where you don't always have access
  }
}
```

这是用户将看到的第一个视图控制器，因此当这个视图出现时立即请求用户的位置是个好主意。如果你不明确你将提示用户位置，通常在显示位置访问对话框之前通知用户你为什么要请求位置权限是个好主意。要实际使权限对话框出现，你需要在`LocationHelper.swift`中添加一些代码。

所有位置服务相关的请求都是通过`CLLocationManager`的一个实例来执行的。位置管理器负责获取用户的 GPS 位置，请求访问用户位置的权限，以及更多。当位置管理器收到有关用户位置、授权状态或其他事件的更新时，它将通知其代理。位置管理器代理应遵守`CLLocationManagerDelegate`协议。请注意，`LocationHelper`已经遵守了`CLLocationManagerDelegate`协议，并且在这个对象上已经创建了一个`CLLocationManager`的实例。剩下要做的就是将辅助器分配为位置管理器的代理。在`LocationHelper`的`init()`方法末尾添加以下行以将其设置为位置管理器代理：

```swift
locationManager.delegate = self
```

接下来，为`askPermission(_:)`方法添加以下实现：

```swift
func askPermission(_ completion: @escaping (CLAuthorizationStatus) -> Void) {
  let authorizationStatus = 
    CLLocationManager.authorizationStatus()
  if authorizationStatus != .notDetermined {
    completion(authorizationStatus)
  } else {
    askPermissionCallback = completion
    locationManager.requestAlwaysAuthorization()
  }
}
```

此实现检查是否存在当前授权状态。如果存在，则使用当前状态调用完成`callback`。如果当前状态尚未确定，则请求位置管理器使用`requestAlwaysAuthorization()`方法请求访问用户位置的授权。这将提示用户进行位置权限的请求。在这个应用中你需要永久访问用户的位置的原因是为了确保你可以在本章后面实现地理围栏。将以下方法添加到`CLLocationManagerDelegate`以检索用户对授权提示的响应：

```swift
func locationManager(_ manager: CLLocationManager, didChangeAuthorization status: CLAuthorizationStatus) {
  askPermissionCallback?(status)
  askPermissionCallback = nil
}
```

之前的代码立即将用户的响应传递给存储的完成回调，该回调是通过`askPermission(_:)`传递的。在调用回调后，它被设置为 nil 以避免意外再次调用它。此时，你已经完成了请求访问用户位置所需的所有工作。接下来，让我们看看如何检索用户当前位置，以便你可以在你的应用中使用它。

## 获取用户的位置

一旦你的应用可以访问位置数据，你可以使用位置管理器来开始观察用户的位置、用户移动的方向等等。目前，你将专注于获取用户的当前位置。`GeofenceViewController`已经包含了一个名为`showCurrentLocation()`的方法，该方法负责请求位置辅助器提供当前位置。如果你仔细检查这个方法，你会发现它还通过调用`getLocationName(for:_:)`并传递获取到的位置到这个方法来请求位置名称。`showCurrentLocation()`方法还使用获取到的位置通过在地图视图上调用`setRegion(_:animated:)`来聚焦地图视图在用户的位置上。

由于视图控制器已经完全准备好处理位置更新，你所需要做的就是添加对`getLatestLocation(_:)`和`getLocationName(for:_:)`的正确实现。首先，为`getLatestLocation(_:)`添加以下实现：

```swift
func getLatestLocation(_ completion: @escaping (CLLocation) -> Void) {
  if let location = trackedLocations.last {
    completion(location)
  } else if CLLocationManager.locationServicesEnabled() {
    latestLocationObtainedCallback = completion
    locationManager.startUpdatingLocation()
  }
}
```

上述方法首先检查是否已经获取了位置。如果已经获取，则返回最新获取的位置。如果没有现有位置，代码将检查位置服务是否已启用。检查你即将使用的位置服务是否实际可用始终是一个好习惯。如果位置服务可用，则将`completion`回调存储在辅助器中，并通过调用`startUpdatingLocation()`告诉位置管理器开始监控用户的位置。

调用`startUpdateLocation()`将使位置观察者持续监控用户的 GPS 位置，并通过调用`locationManager(_:didUpdateLocations:)`将其相关更新发送到其代理。此方法将始终接收到管理器获取的一个或多个新位置，其中最新位置将是获取位置列表中的最后一个项。将以下实现添加到`LocationHelper`的`CLLocationManagerDelegate`扩展中：

```swift
func locationManager(_ manager: CLLocationManager, didUpdateLocations locations: [CLLocation]) {
  latestLocationObtainedCallback?(locations.last!)
  latestLocationObtainedCallback = nil
  locationManager.stopUpdatingLocation()
  trackedLocations += locations
}
```

`locationManager(_:didUpdateLocations:)`的实现相当简单：将最新位置传递给回调，并移除回调以防止后续的位置更新意外触发回调。此外，通过调用`stopUpdatingLocation()`告诉位置管理器停止监控用户的位置。最后，将获取到的位置存储以供以后使用。

重要提示

总是好的做法，如果你不会很快需要更新，就停止位置管理器监控位置更新。监控位置更新对电池寿命有相当大的影响，因此你不应该花费比所需更多的时间来跟踪用户的位置。

现在你已经可以检索用户的位置，下一步是通过在位置辅助器中实现`getLocationName(for:_:_)`来检索位置名称。将以下实现添加到位置辅助器中：

```swift
func getLocationName(for location: CLLocation, _ completion: @escaping (String) -> Void) {
  let geocoder = CLGeocoder()
  geocoder.reverseGeocodeLocation(location) { placemarks,
    error in
  guard error == nil else {
    completion("An error ocurred: 
    \(error?.localizedDescription ?? "Unknown error")")
    return
  }
  completion(placemarks?.first?.name ?? "Unkown location")
  }
}
```

上述代码使用`CLGeocoder`查找与用户当前位置相对应的地点标记。请注意，此功能需要互联网连接，因此名称查找只有在用户有互联网连接时才会工作。常规的 GPS 相关功能不需要互联网访问，因此即使用户没有活跃的互联网连接，你的应用也可以监控和跟踪用户的位置。

现在尝试运行您的应用——您应该能够在地图上看到用户的当前位置，并且位置名称、纬度和经度也应显示在屏幕上。现在您已经知道如何获取用户的位置，让我们看看您如何有效地订阅应用以跟踪用户位置的变化，以便跟踪他们的位置。

# 订阅位置变化

订阅用户位置变化的一种方法已经在本章的前一节中介绍过了。当您在位置管理器上调用`startUpdatingLocation()`时，它将自动订阅用户的位置。如果您需要非常详细的位置报告，这种跟踪用户位置的方法是极好的，但通常您不需要这种详细程度。更重要的是，长时间使用这种位置跟踪会耗尽用户的电池。

幸运的是，有更好的方法来监控位置变化。一种方法是调用`startMonitoringVisits()`来订阅用户访问的位置。如果您不感兴趣于用户的详细移动，但只想知道用户是否在特定区域停留了较长时间，则使用此方法。这种跟踪用户位置的方式非常适合您需要低功耗方式来跟踪非常粗略的位置变化。即使您的应用在后台运行，这种跟踪方式也能很好地工作，因为如果发生访问事件，您的应用将自动被唤醒或启动。

如果您的应用因与位置相关的事件而重新启动，那么`UIApplication.LaunchOptionsKey.location`将存在于应用的启动选项字典中。当它存在时，您应该创建一个位置管理器的实例并将其分配一个代理以接收相关的位置更新。

如果访问监控对您的用途来说不够精确，但您不需要持续的位置跟踪，您可以使用`LocationServices`示例应用来查看它们是如何工作的。

如果您查看`SignificantChangesViewController`，您会注意到视图控制器已经完全设置好以开始监控显著的位置变化。在位置辅助器中定义的`monitorSignificantChanges(_:)`方法，每次发生显著的位置变化时都会调用回调。每次检索到新的位置数据时，表格视图都会重新加载以显示最新的可用数据。由于显著的位置变化可以通过应用启动选项中的特殊键唤醒应用，因此让我们更新`AppDelegate`以处理这种情况。在`return`语句之前添加以下代码`application(_:didFinishLaunchingWithOptions:)`：

```swift
if launchOptions?[UIApplication.LaunchOptionsKey.location] != nil
  { locationHelper.monitorSignificantChanges { _ in
    // continue monitoring
  }
}
```

由于 `AppDelegate` 已经有对位置辅助的引用，它只需要重新启用显著位置变化的监控。这种对 `AppDelegate` 的小改动非常强大，因为它允许你的应用在应用未运行时也能对用户位置的变化做出响应。接下来，让我们在位置辅助中实现适当的代码。

为 `LocationHelper` 中的 `monitorSignificantLocationChanges(_:)` 方法添加以下实现：

```swift
func monitorSignificantChanges(_ locationHandler: @escaping (CLLocation) -> Void) {
  guard CLLocationManager. 
    significantLocationChangeMonitoringAvailable() else { return }
  significantChangeReceivedCallback = locationHandler
  locationManager.startMonitoringSignificantLocationChanges()
  isTrackingSignificantLocationChanges = true
}
```

此方法与之前见过的位置辅助方法非常相似。当检测到显著的位置变化时，位置管理器会调用其代理的 `locationManager(_:didUpdateLocations:)` 方法。由于此方法已经实现，你应该按照以下方式更新实现：

```swift
func locationManager(_ manager: CLLocationManager, didUpdateLocations locations: [CLLocation]) {
  latestLocationObtainedCallback?(locations.last!)
  latestLocationObtainedCallback = nil
  if isTrackingSignificantLocationChanges == false {
    locationManager.stopUpdatingLocation()
  }
  significantChangeReceivedCallback?(locations.last!)
  trackedLocations += locations
}
```

注意，当显著位置变化跟踪不活跃时，位置管理器才被告知停止更新用户的位置。当你调用 `stopUpdatingLocation()` 时，位置管理器将停止向此代理方法提供任何位置更新。另外，请注意，在调用后 `significantChangeReceivedCallback` 不会被移除。这是因为调用 `monitorSignificantChanges(_:)` 的调用者对连续的位置更新感兴趣，所以每次调用此方法时，应该始终调用启动了显著位置变化跟踪的 `SignificantChangesViewController` 视图控制器。

最后，为了确保你的应用在后台时也能接收到显著的位置变化，你需要将 `allowsBackgroundLocationUpdates` 属性设置为 `true`。将以下代码行添加到位置辅助的 `init()` 方法中：

```swift
locationManager.allowsBackgroundLocationUpdates = true
```

除了订阅显著的位置变化或访问之外，你还可以使用地理围栏对用户进入或离开某个特定区域做出响应。

# 设置地理围栏

有时，你的应用并不真的需要知道用户的位置详情。有时，你只对跟踪用户是否离开了或离开了某个特定区域感兴趣，以便在你的应用中显示某些内容或解锁某种特殊功能。Core Location 对监控地理围栏提供了很好的支持。`CLRegion` 是一个子类。Core Location 提供了两种不同的区域类型，你可以使用：

+   `CLCircularRegion`

+   `CLBeaconRegion`

如前所述，使用 `CLCircularRegion` 类型来设置地理围栏。使用 `CLBeaconRegion` 类型与物理 BLE iBeacon 配合使用，基本上在非常小的半径内提供地理围栏，例如，仅几米。在本节中，你将学习如何设置一个围绕用户首次检测到的位置的 `CLCircularRegion` 类型。使用这两种类型的区域设置地理围栏或区域监控非常相似，所以监控圆形区域的所有原则也适用于信标区域。

如果你查看 `GeofenceViewController`，你会注意到它有一个标记为 `@IBAction` 的按钮，这个按钮已经做了很多工作，但缺少一个关键元素——它没有通知位置管理器应该监控的区域。请将以下代码添加到 `GeofenceViewController` 中的 `setGeofence()` 方法末尾：

```swift
let region = CLCircularRegion(center: location.coordinate, radius: 30, identifier: "current-location-geofence") locationHelper.setGeofence(at: region, exitHandler, enterHandler)
```

上述代码使用从用户那里获取的位置信息，并使用它创建一个半径为 30 米的圆形区域。传递给区域的标识符应该是一个唯一定义该区域的标识符。如果你重复使用一个标识符，Core Location 将停止监控具有该标识符的旧区域，并监控新的区域。对于 LocationServices 应用来说这很完美，但如果你想让你的应用监控多个区域，你必须确保每个区域都有自己的唯一标识符。

接下来，将以下 `setGeofence(at:_:_:)` 的实现添加到 `LocationHelper` 中：

```swift
func setGeofence(at region: CLRegion, _ exitHandler: @escaping () -> Void, _ enterHandler: @escaping () -> Void) {
  guard CLLocationManager.isMonitoringAvailable(for: 
  CLCircularRegion.self) else { return }
  geofenceExitCallback = exitHandler
  geofenceEnterCallback = enterHandler
  locationManager.startMonitoring(for: region)
}
```

上述方法与其他位置助手方法非常相似。让我们直接进入实现位置管理器将调用的 `CLocationManagerDelegate` 方法：

```swift
func locationManager(_ manager: CLLocationManager, didEnterRegion region: CLRegion) {
  geofenceEnterCallback?()
}
func locationManager(_ manager: CLLocationManager, didExitRegion region: CLRegion) {
  geofenceExitCallback?()
}
```

上述两个方法是 `CLocationManagerDelegate` 协议的一部分，当用户进入或离开某个区域时被调用。由于助手没有额外的工作要做，相应的回调立即被调用，以便 `GeofenceViewController` 可以相应地更新其界面。

尝试打开应用并点击 **设置地理围栏** 按钮。现在应该会在地图上出现一个橙色圆圈，以可视化你设置的地理围栏。如果你退出或进入该区域，状态标签应该相应更新，以显示你是否刚刚进入或离开了地理围栏。请注意，iOS 可能需要最多五分钟的时间来正确注册、监控和报告关于你的地理围栏的更新。请注意，为了使区域监控工作最佳，用户应该有一个活跃的互联网连接。

# 摘要

在本章中，你学习了获取和响应用户位置的一些技术。你实现了一个 `LocationHelper` 类，它为视图控制器提供了一个简单的接口来使用包含在助手中的位置管理器。你了解了在请求用户访问其位置数据方面的最佳实践，并且你了解到请求用户的位置是一个相当敏感的隐私问题，在没有充分理由的情况下不应该提出。

你了解到有不同方式，每种方式都有不同级别的细节，你可以用来追踪用户的位置。你看到了你可以订阅连续的变化，这对电池寿命有不良影响。你还学习了关于订阅访问和显著位置变化的内容。除了学习追踪用户的位置外，你还学习了通过实现地理围栏来监控用户是否进入或离开了某个区域。当你在自己的应用中实现 Core Location 时，始终确保将用户的隐私放在心上。如果你真的不需要位置数据，那么不要请求访问它。如果你需要，确保非常小心地处理用户的位置数据。

在下一章中，你将学习关于 Combine 框架以及如何使用它来增强你的应用。
