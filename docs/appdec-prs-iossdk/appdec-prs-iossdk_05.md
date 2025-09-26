# 第五章。推送通知

推送通知允许您在应用处于后台或根本未运行时发送消息。推送通知增加了用户与应用的互动，并通过消息让所有用户了解应用，从而产生收入。此外，通过将您的数据和推送服务相互连接，您可以以选择性的方式发送推送通知，例如*祝贺您与 Packt Publishing 合作满一年*。这是接触所有用户的最简单方法。Parse 为您提供推送通知服务。本章将帮助您为您的应用配置推送通知。

# 设置推送通知

通知帮助您在应用中添加实时消息。为此，您需要创建 SSL 证书，然后将创建的证书与您的**App ID**和**配置文件**关联。

## 创建 SSL 证书

您需要在 Apple 开发者门户上创建您的**App ID**及其关联的 SSL 证书。Parse 服务器将使用此证书向由该**App ID**标识的应用发送推送通知。

## 生成证书请求

执行以下步骤以生成证书并将其保存到您的磁盘：

1.  在您的 Mac 上打开**钥匙串访问**应用程序。

1.  导航到**钥匙串访问** | **证书助手** | **从证书颁发机构请求证书…**。

1.  在**证书信息**窗口中，输入您的电子邮件地址和姓名。

1.  选择**保存到磁盘**以将**.certSigningRequest**文件下载到您的桌面。

## 设置您的 App ID

所有安装在开发者设备上的 iOS 应用都需要 App ID。您可以使用现有的 App ID，但请确保它没有通配符。您可以通过以下步骤创建新的 App ID：

1.  登录到 Apple 开发者门户([`developer.apple.com/membercenter/index.action`](https://developer.apple.com/membercenter/index.action))并导航到**证书、标识符和配置文件**([`developer.apple.com/account/overview.action`](https://developer.apple.com/account/overview.action))。

1.  点击门户左侧面板中的**标识符**。

1.  点击右上角的**+**按钮以创建新的 App ID。

1.  输入新 App ID 的名称，并在**应用服务**部分下选择**推送通知**复选框。

1.  在**App ID 后缀**部分，选择**显式 App ID**。输入您的 iOS 应用的**Bundle ID**。此字符串应与 iOS 应用`info.plist`文件中的 Bundle 标识符相匹配。

1.  点击**继续**然后点击**提交**以完成注册。

1.  从**iOS App ID**列表中选择您新创建的 App ID，然后点击**设置**。

1.  导航到**推送通知**部分。您可以下载**开发**和**生产**SSL 证书。

1.  点击 **创建证书...**，然后点击 **继续**。

1.  现在，你可以上传上一节中生成的证书，然后点击 **生成**。

1.  上传你的证书后，点击 **完成**，然后从 **iOS App ID 设置** 屏幕下载生成的 SSL 证书。

1.  通过双击 SSL 证书在你的密钥链中安装下载的证书。

1.  在 **密钥链访问** 中，在 **我的证书** 部分选择你安装的证书，它应该以 **Apple Development IOS Push Services:** 作为前缀。

1.  右键单击选定的证书并将其保存为 `.p12` 文件格式。在提示时不要输入任何密码。

# 创建你的配置文件

为了验证你的设备以用于应用程序，你需要创建一个配置文件。每次更新你的 App ID 时，你都需要重新生成你的配置文件。按照以下步骤创建配置文件：

1.  登录到苹果开发者门户并导航到 **证书、标识符和配置文件** 部分。

1.  从 **iOS 应用** 部分选择 **配置文件**。

1.  点击该部分右上角的 **+** 按钮，以添加一个新的配置文件。

1.  选择你的配置文件类型为 **iOS 应用开发** 并点击 **继续**。

1.  选择上一节中创建的应用 ID 并点击 **继续**。

1.  选择你的证书并点击 **继续**。

1.  从设备列表中选择你想要安装和测试应用程序的设备，并点击 **继续**。

1.  为你的配置文件提供一个名称并点击 **生成**。

1.  下载生成的配置文件并双击以安装它。

这应该会在 **设备** 面板中启动你的 Xcode 的 **组织者**。你的新配置文件应该出现在 **库** 的 **配置文件** 部分中。确保配置文件的 **状态** 是 **有效** 的。

## 配置你的 Parse 应用

要使用 Parse 的推送通知服务，你必须在 Parse 应用中启用此功能，通过上传上一节中生成的推送 SSL 证书：

1.  导航到你的 Parse 应用在 Parse **仪表板**上，并选择 **设置** 选项卡。

1.  在设置面板的左侧，你将看到 **推送通知**。在 **Apple 推送证书** 标题下，点击 **选择你的证书** 并定位到上一节从密钥链导出的 `.p12` 证书。

## 配置 iOS 应用以使用推送通知

你需要更新以下应用程序设置以配置推送通知：

1.  你的应用程序包标识符应该与你创建 App ID 时在苹果开发者门户上提供的完全相同。

1.  更新你的 **代码签名标识符** 字段以匹配上一节中创建的配置文件。

## 安装

在所有前面的部分中，我们已经配置了开始使用推送通知所需的所有内容。在本节中，我们将添加一些代码以在我们的应用程序中接收推送通知。

您需要注册您的设备以获取推送通知，在应用的代理方法 `[application:didFinishLaunchingWithOptions:]` 中添加以下代码。

```swift
- (BOOL)application:(UIApplication *)application
 didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
    ...
    // Register for push notifications
    [application registerForRemoteNotificationTypes:
                                 UIRemoteNotificationTypeBadge |
                                 UIRemoteNotificationTypeAlert |             
                                 UIRemoteNotificationTypeSound];
    ...
}
```

在成功注册后，您的应用代理中的回调方法 `[application:didRegisterForRemoteNotificationsWithDeviceToken:]` 将被执行。我们需要配置此方法以通知 Parse 关于新设备的信息：

```swift
- (void)application:(UIApplication *)application
didRegisterForRemoteNotificationsWithDeviceToken:(NSData *)newDeviceToken {
    // Store the deviceToken in the current installation and save it to Parse.
    PFInstallation *currentInstallation = [PFInstallation currentInstallation];
    [currentInstallation setDeviceTokenFromData:newDeviceToken];
    [currentInstallation saveInBackground];
}
```

您可以像更新 `PFObject` 一样更新 `PFInstallation`。您可以添加各种特殊字段，这些字段将帮助您在 Parse 上管理您的设备。以下是一些字段及其用途：

+   `badges`: 您可以在 `PFInstallation` 中更改此值以更新应用图标的徽章。建议在服务器上保存徽章更改，以便将来进行徽章增量通知。

+   `channels`: 这将存储订阅当前设备的频道数组。

+   `timeZone`: 此属性用于设备的当前位置。当我们更新服务器上的 `Installation` 对象时，它会自动更新。这是一个只读属性。

+   `deviceType`: 此属性适用于任何设备类型，可以是 iOS、Android、winrt、winphone 或 dotnet。这是一个只读属性。

+   `installationId`: 这是 Parse 用于设备的唯一 ID 属性。这是一个只读属性。

+   `deviceToken`: 这是用于 iOS 设备的 Apple 生成的令牌。这是一个只读属性。

+   `channelUris`: 这是 Microsoft 为 Windows 设备生成的推送 URI。这是一个只读属性。

+   `appName`: 这是属于此安装的客户端应用程序的显示名称。这是一个只读属性。

+   `appVersion`: 这是属于当前安装的客户端应用程序的版本字符串。这是一个只读属性。

+   `parseVersion`: 这是使用此安装的 Parse SDK 的版本。这是一个只读属性。

+   `appIdentifier`: 这是此安装客户端应用程序的唯一标识符。在 iOS 中，这是 `Bundle Identifier`。这是一个只读属性。

当您的应用程序在前台模式下接收到通知时，您可以使用以下代码来处理通知：

```swift
- (void)application:(UIApplication *)application
didReceiveRemoteNotification:(NSDictionary *)userInfo {
    [PFPush handlePush:userInfo];
}
```

在接收到通知后，我们可以将通知传递给 Parse 以处理。它将自动创建一个模态警报，并显示推送消息内容。

您现在可以在您的设备上执行应用程序以确保一切设置正确。如果您是第一次在您的设备上运行此应用，您将收到一个模态框请求用户发送推送通知的权限。

# 发送推送通知

您可以使用以下方式向设备发送推送通知。

## Parse 网站

执行以下步骤通过 Parse 网站发送通知：

1.  导航到您的 Parse 应用程序，位于 Parse.com 上。

1.  点击**推送通知**标签。

1.  您可以点击框的右上角的**发送推送**。

1.  在消息框中编写一条消息，并将其广播给**所有人**。您还有其他选项可供选择设备类型。

1.  Parse 允许您安排通知。您可以配置此计划并点击**发送通知**来发送通知。

## Parse 应用程序

通常，您可以通过 Web 控制台、使用 REST API 或 Cloud Code 发送推送通知。然而，Parse 还允许您从您的移动应用程序发送推送通知。请记住，您已将**客户端推送**设置为**是**。启用的**客户端推送**设置允许您从一个设备向另一个设备发送通知。`PFPush`类中提供了许多方法，您可以通过这些方法通过您的移动设备发送推送通知。您可以在 API 文档中访问所有这些方法。以下是从 iOS 设备生成的推送通知的示例代码：

```swift
// Create our Installation query
PFQuery *pushQuery = [PFInstallation query];
[pushQuery whereKey:@"deviceType" equalTo:@"ios"];

// Send push notifications to query
[PFPush sendPushMessageToQueryInBackground:pushQuery	
                               withMessage:@"Hello World!"];
```

## Cloud Code

Cloud Code 还允许您使用`Parse.Cloud.afterSave`方法发送推送通知，该方法将在对象在 Parse 上成功保存后执行。Cloud 可以用 JavaScript 编写。您应该熟悉 JavaScript 才能编写 Cloud Code。让我们以发送每条评论后发送推送通知的示例为例：

```swift
Parse.Cloud.afterSave("Comment", function(request) {
  // Our "Comment" class has a "text" key with the body of the comment itself
  var commentText = request.object.get('text');

  var pushQuery = new Parse.Query(Parse.Installation);
  pushQuery.equalTo('deviceType', 'ios');

  Parse.Push.send({
    where: pushQuery, // Set our Installation query
    data: {
      alert: "New comment: " + commentText
    }
  }, {
    success: function() {
      // Push was successful
    },
    error: function(error) {
      throw "Got an error " + error.code + " : " + error.message;
    }
  });
});
```

# 发送通知

使用 Parse 向设备发送推送通知有两种方式。如下所示。

## 通道

这是开始发送通知的最简单方法。它基于发布/订阅模型。例如，如果您想向用户发送关于他/她创建的每条评论的通知，那么您必须创建一个通道并将用户订阅到该通道，并且对于每条评论您都需要为该通道发送一个推送。这将使用户知道当有人评论他/她的帖子时，即使应用程序未运行或处于后台。因此，为了发送推送通知，您需要允许用户创建一个通道。一个设备可以订阅一个或多个通道以接收可以发送给此类订阅者的通知。订阅的通道存储在`Installation`对象的`channels`字段中。

### 订阅通道

您可以通过字符串识别一个通道，这可以是由字母数字字符、下划线和破折号组成的组合。每个安装可以在任何时候订阅任意数量的通道。您可以通过在`Installation`对象中使用以下代码来添加一个通道：

```swift
// When users Comment, we subscribe them to that channel.
PFInstallation *currentInstallation = [PFInstallation currentInstallation];
[currentInstallation addUniqueObject:@"Comments" forKey:@"channels"];
[currentInstallation saveInBackground];
```

在订阅评论后，您的`Installation`对象应类似于以下内容：

```swift
objectId:yvoZDtAxUR channels:["Comments"]
```

要从通道中取消用户订阅，您可以添加以下代码：

```swift
// When users indicate they are no longer Giants fans, we unsubscribe them.
PFInstallation *currentInstallation = [PFInstallation currentInstallation];
[currentInstallation removeObject:@"Comments" forKey:@"channels"];
[currentInstallation saveInBackground];
```

为了缓存目的，获取通道列表作为数组，您可以使用以下代码：

```swift
NSArray *subscribedChannels = [PFInstallation currentInstallation].channels;
```

### 向通道发送推送通知

要向所有订阅了`Comments`渠道的用户发送推送通知，以下代码将向所有订阅用户发送推送：

```swift
// Send a notification to all devices subscribed to the "Giants" channel.
PFPush *push = [[PFPush alloc] init];
[push setChannel:@"Comments"];
[push setMessage:@"Thats the new comment!"];
[push sendPushInBackground];
```

如果您想同时向多个渠道发送推送，可以使用以下代码：

```swift
NSArray *channels = [NSArray arrayWithObjects:@"Comments", @"Mets", nil];
PFPush *push = [[PFPush alloc] init];

// Be sure to use the plural 'setChannels'.
[push setChannels:channels];
[push setMessage:@"Multiple channel comments."];
[push sendPushInBackground];
```

# 使用高级定位

在高级推送通知中，Parse 允许您嵌入查询，并将查询结果作为推送通知发送给用户。这将帮助您向自定义动态用户群体发送推送。

您可以将数据保存到`Installation`对象中，就像任何`PFObject`一样。假设在您的应用程序中，您允许用户点赞、评论和发布。以下代码片段将帮助您实现这一点：

```swift
// Store app language and version
PFInstallation *installation = [PFInstallation currentInstallation];
[installation setObject:[NSNumber numberWithBool:YES] forKey:@"Like"];
[installation setObject:[NSNumber numberWithBool:YES] forKey:@"Comment"];
[installation setObject:[NSNumber numberWithBool:YES] forKey:@"Posts"];
[installation saveInBackground];
```

## 向查询发送推送

您可以通过对目标设备应用查询来过滤`Installation`对象。这可以通过以下代码实现：

```swift
// Create our Installation query
PFQuery *pushQuery = [PFInstallation query];
[pushQuery whereKey:@"Comments" equalTo:[NSNumber numberWithBool:YES]];

// Send push notifications to query
PFPush *push = [[PFPush alloc] init];
[push setQuery:pushQuery]; // Set our Installation query
[push setMessage:@"This is the test comment."];
[push sendPushInBackground];
```

在发送通知之前，我们可以将查询设置为`PFPush`。您还可以将数据存储在关系中。这些关系也可以用于查询：

```swift
// Find users near a given location
PFQuery *userQuery = [PFUser query];
[userQuery whereKey:@"location"
        nearGeoPoint:partyLocation
         withinMiles:[NSNumber numberWithInt:1]]

// Find devices associated with these users
PFQuery *pushQuery = [PFInstallation query];
[pushQuery whereKey:@"user" matchesQuery:userQuery];

// Send push notifications to query
PFPush *push = [[PFPush alloc] init];
[push setQuery:pushQuery]; // Set our Installation query
[push setMessage:@"Party tickets free to all nearby users!"];
[push sendPushInBackground];
```

## 自定义通知

您可以自定义推送通知数据，例如播放的声音、徽章编号、要发送的自定义数据和通知的过期时间。

您还可以通过将所有设置包装在`NSDictionary`方法中来自定义通知。以下是一个示例，通过通知增加徽章计数并添加自定义声音和消息：

```swift
NSDictionary *data = [NSDictionary dictionaryWithObjectsAndKeys:
    @"New comment!", @"alert",
    @"Increment", @"badge",
    @"demo.caf", @"sound",
    nil];
PFPush *push = [[PFPush alloc] init];
[push setChannels:[NSArray arrayWithObjects:@"Mets", nil]];
[push setData:data];
[push sendPushInBackground];
```

要设置通知的过期日期，可以使用以下示例代码：

```swift
// Create date object for tomorrow
NSDateComponents *comps = [[NSDateComponents alloc] init];
[comps setYear:2013];
[comps setMonth:8];
[comps setDay:27];
NSCalendar *gregorian =[[NSCalendar alloc] initWithCalendarIdentifier:NSGregorianCalendar];
NSDate *date = [gregorian dateFromComponents:comps];

// Send push notifications with expiration date
PFPush *push = [[PFPush alloc] init];
[push expireAtDate:date];
[push setQuery:everyoneQuery];
[push setMessage:@"Movie tickets on sale until August 27th"];
[push sendPushInBackground];
```

## 基于平台的定位

您可以让应用程序在多个平台上运行。如果您想针对特定平台设备或操作系统发送推送，则可以使用以下代码。此代码将为 iOS、Android 和 Windows 应用程序发送单独的通知：

```swift
PFQuery *query = [PFInstallation query];
[query whereKey:@"channels" equalTo:@"suitcaseOwners"];

// Notification for Android users
[query whereKey:@"deviceType" equalTo:@"android"];
PFPush *androidPush = [[PFPush alloc] init];
[androidPush setMessage:@"Your suitcase has been filled with tiny robots!"];
[androidPush setQuery:query];
[androidPush sendPushInBackground];

// Notification for iOS users
[query whereKey:@"deviceType" equalTo:@"ios"];
PFPush *iOSPush = [[PFPush alloc] init];
[iOSPush setMessage:@"Your suitcase has been filled with tiny apples!"];
[iOSPush setChannel:@"suitcaseOwners"];
[iOSPush setQuery:query];
[iOSPush sendPushInBackground];

// Notification for Windows 8 users
[query whereKey:@"deviceType" equalTo:@"winrt"];
PFPush *winPush = [[PFPush alloc] init];
[winPush setMessage:@"Your suitcase has been filled with tiny glass!"];
[winPush setQuery:query];
[winPush sendPushInBackground];

// Notification for Windows 8 users
[query whereKey:@"deviceType" equalTo:@"winphone"];
PFPush *winPush = [[PFPush alloc] init];
[wpPush setMessage:@"Your suitcase is very hip; very metro."];
[wpPush setQuery:query];
[wpPush sendPushInBackground];
```

# 接收推送通知

当您收到自定义通知时，您可以从通知中检索相关数据。这些数据可以帮助根据通知修改应用程序的行为。例如，如果通知是因为对帖子的评论而发送的，您可以直接通过阅读推送通知数据来打开该评论的帖子。

苹果对包大小有限制，因此建议尽可能保持数据大小最小：

```swift
NSDictionary *data = @{
  @"alert": @"James commented on your photo!",
  @"p": @"vmRZXZ1Dvo" // Photo's object id
};
PFPush *push = [[PFPush alloc] init];
[push setQuery:photoOwnerQuery];
[push setData:data];
[push sendPushInBackground];
```

# 响应有效载荷

如果您的应用是通过通知启动的，您可以通过`launchOptions`字典从`[application:didFinishLaunchingWithOptions:]`方法访问数据：

```swift
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
  . . .
  // Extract the notification data
  NSDictionary *notificationPayload = launchOptions[UIApplicationLaunchOptionsRemoteNotificationKey];

  // Create a pointer to the Post object
  NSString *postId = [notificationPayload objectForKey:@"p"];
  PFObject *targetPost = [PFObject objectWithoutDataWithClassName:@"Post"objectId:postId];

  // Fetch photo object
  [targetPhoto fetchIfNeededInBackgroundWithBlock:^(PFObject *object, NSError *error) {
    // Show post view controller
    if (!error && [PFUser currentUser]) {
      PostVC *viewController = [[PostVC alloc] initWithPost:object];
      [self.navController pushViewController:viewController animated:YES];
    }
  }];
}
```

如果您的应用程序已经在前台模式下运行，则可以从`[application:didRegisterForRemoteNotificationsWithDeviceToken:]`方法中的`userInfo`字典中获取数据：

```swift
- (void)application:(UIApplication *)application didReceiveRemoteNotification:(NSDictionary *)userInfo {
  // Create empty post object
  NSString *postId = [userInfo objectForKey:@"p"];
  PFObject *targetPost = [PFObject objectWithoutDataWithClassName:@"Post"objectId:postId];

  // Fetch photo object
  [targetPhoto fetchIfNeededInBackgroundWithBlock:^(PFObject *object, NSError *error) {
    // Show photo view controller
    if (!error && [PFUser currentUser]) {
      PostVC *viewController = [[PostVC alloc] initWithPost:object];
      [self.navController pushViewController:viewController animated:YES];
    }
  }];
}
```

# 摘要

在本章中，我们探讨了在我们的应用程序中设置和安装推送通知。

我们首先在 Apple 开发者门户上设置了推送通知，并在代码库中进行了必要的安装。

然后，我们学习了发送推送通知及其调度。

最后，我们了解了从通知中接收有效载荷的方法。

在下一章中，我们将学习关于 Parse 用户及其角色的内容。
