# 第七章。社交媒体集成

这些天，社交媒体集成是所有移动应用程序都包含的常见功能。一些应用程序允许用户将数据更新分享到他们的社交媒体资料，如 Facebook 和 Twitter。

现在，使用电子邮件 ID 和密码的传统登录过程正变得越来越繁琐。因此，为了解决这些问题，许多应用程序使用 Facebook 和 Twitter 作为其基础登录。用户可以使用他们的 Facebook 或 Twitter 账户进行注册。这甚至简化了登录过程的步骤，并为用户提供了一键注册和登录。考虑到这些因素，Parse 提供了即插即用的方法来集成社交媒体，无论是用于分享还是用于登录过程。在本章中，你将详细了解使用 Parse 进行社交媒体集成。

# 集成 Facebook

将 Facebook 集成到你的 Parse 应用程序中很容易。Facebook SDK 可以与 Parse SDK 一起使用。你可以将你的 `PFUser` 与 Facebook 用户身份链接。你也可以使用 Facebook 进行注册过程。Parse 允许你从 Facebook 验证用户并将用户关联到你的 `PFUser` 类。

在本节中，你将学习如何从你的 Parse 应用程序中集成 Facebook。

## 前提条件

要开始将 Facebook 与 Parse 集成，你需要设置以下事项：

+   在 Facebook 开发者门户（[`developers.facebook.com/`](https://developers.facebook.com/））上注册并创建一个 Facebook 应用程序（[https://developers.facebook.com/docs/ios/getting-started/](https://developers.facebook.com/docs/ios/getting-started/）），如果你还没有创建的话。

+   Facebook 提供了创建应用程序的逐步指南（[https://developers.facebook.com/apps](https://developers.facebook.com/apps)）并将其与 Facebook SDK 集成。

## 设置应用程序

要开始使用 Facebook，你需要在应用程序中配置以下内容：

1.  从左侧面板中选择你的项目文件，点击 **目标** 并导航到 **信息** 选项卡。

1.  在 **自定义 iOS 目标属性** 部分中，添加一个新键，将 **Facebook App Id** 作为字符串值，并提供你的 Facebook 应用 ID 作为值。

1.  现在添加另一个带有 **URL** **类型** 的键名。展开键名，最后你会看到 **URL 标识符**。将 **URL 标识符** 替换为 **URL 方案** 并填写 `fbYour_App_Id`（例如，Fb1234567890）。

在你的应用程序中更新 Facebook 键后，执行以下步骤以设置代码：

1.  在你的 `application:didFinishLaunchingWithOptions:` 方法中，添加以下代码行以初始化 Facebook，就在你初始化 Parse SDK 的代码下方：

    ```swift
    [PFFacebookUtils initializeFacebook];
    ```

1.  在你的 `AppDelegate` 文件中添加以下方法以处理 Facebook SDK 的单点登录功能：

    ```swift
    - (BOOL)application:(UIApplication *)application openURL:(NSURL *)url
      sourceApplication:(NSString *)sourceApplication annotation:(id)annotation {
        return [PFFacebookUtils handleOpenURL:url];
    }

    - (BOOL)application:(UIApplication *)application handleOpenURL:(NSURL *)url {
        return [PFFacebookUtils handleOpenURL:url];
    }
    ```

## 在 Facebook 上注册

使用 Parse SDK，您可以允许用户使用 Facebook 注册。Parse 还将为您创建和保存用户对象。以下方法将允许您使用 Facebook 登录：

```swift
[PFFacebookUtils logInWithPermissions:permissions block:^(PFUser *user, NSError *error) {
    if (!user) {
        NSLog(@"Uh oh. The user cancelled the Facebook login.");
    } else if (user.isNew) {
        NSLog(@"User signed up and logged in through Facebook!");
    } else {
        NSLog(@"User logged in through Facebook!");
    }
}];
```

在之前的代码中，`permissions`是一个字符串数组，指定了您的应用程序想要从 Facebook 访问哪些数据。

以下代码将展示您如何使用 Facebook 登录或注册：

```swift
- (IBAction)loginButtonTouchHandler:(id)sender  {
    // The permissions requested from the user
    NSArray *permissionsArray = @[ @"user_about_me", @"user_relationships", @"user_birthday", @"user_location"];

    // Login PFUser using Facebook
    [PFFacebookUtils logInWithPermissions:permissionsArray block:^(PFUser *user, NSError *error) {
        [_activityIndicator stopAnimating]; // Hide loading indicator

        if (!user) {
            if (!error) {
                NSLog(@"Uh oh. The user cancelled the Facebook login.");
            } else {
                NSLog(@"Uh oh. An error occurred: %@", error);
            }
        } else if (user.isNew) {
            NSLog(@"User with facebook signed up and logged in!");
            [self.navigationController pushViewController:[[UserDetailsViewController alloc] initWithStyle:UITableViewStyleGrouped] animated:YES];
        } else {
            NSLog(@"User with facebook logged in!");
            [self.navigationController pushViewController:[[UserDetailsViewController alloc] initWithStyle:UITableViewStyleGrouped] animated:YES];
        }
    }];
}
```

在按钮动作中，您可以添加之前的代码以使用 Facebook 登录或注册。在之前的代码中，我们使用了`permissionsArray`。此数组将包含表示从 Facebook 需要获取的用户数据的字符串。

执行前面的函数后，将发生以下步骤：

1.  应用程序将被重定向到 Facebook 或提示获取访问 Facebook 数据的权限。

1.  当用户通过 Facebook 进行身份验证时，您的应用程序将使用我们在上一节中配置的`handleOpenUrl`方法接收回调。

1.  如果用户是新的，Parse SDK 将获取数据并将其保存到`PFUser`类中，并将数据保存在 Parse 上。

1.  您的代码块将使用用户调用。

由于 Parse 已被 Facebook，Inc.收购，Parse 提供了与 Facebook 的非常深入的集成，这也预示着未来 Parse 将实现更加深入和广泛的集成。

## 在 Facebook 上发布

要在用户墙上发布一些更新，您需要从用户那里获取发布权限。以下代码将获取用户的发布权限：

```swift
[PFFacebookUtils reauthorizeUser:[PFUser currentUser]
                 withPermissions:@["publish_stream"]
                        audience:PF_FBSessionDefaultAudienceFriends
                           block:^(BOOL succeeded, NSError *error) {
    if (succeeded) {
        //Your app now has publishing permissions for the user
    }
}];
```

## 请求用户数据

用户登录后，您可以使用 Graph API 开始从 Facebook 获取用户数据。以下代码将展示如何从 Facebook 获取用户数据：

```swift
- (void)fetchUserData {
    // ...
    // Create request for user's Facebook data
    FBRequest *request = [FBRequest requestForMe];

    // Send request to Facebook
    [request startWithCompletionHandler:^(FBRequestConnection *connection, id result, NSError *error) {
        if (!error) {
            // result is a dictionary with the user's Facebook data
            NSDictionary *userData = (NSDictionary *)result;

            NSString *facebookID = userData[@"id"];
            NSString *name = userData[@"name"];
            NSString *location = userData[@"location"][@"name"];
            NSString *gender = userData[@"gender"];
            NSString *birthday = userData[@"birthday"];
            NSString *relationship = userData[@"relationship_status"];

            NSURL *pictureURL = [NSURL URLWithString:[NSString stringWithFormat:@"https://graph.facebook.com/%@/picture?type=large&return_ssl_resources=1", facebookID]];

            // Now add the data to the UI elements
            // ...
        }
    }];
}
```

`result`字典将包含所有请求的数据。您可以通过键查询来获取数据。

## 链接用户

如果是匿名登录或使用电子邮件 ID 和密码注册，可以将现有的`PFUser`与 Facebook 账户关联。以下代码将展示如何将现有用户链接到 Facebook：

```swift
if (![PFFacebookUtils isLinkedWithUser:user]) {
    [PFFacebookUtils linkUser:user permissions:nil block:^(BOOL succeeded, NSError *error) {
        if (succeeded) {
            NSLog(@"User logged in with Facebook!");
        }
    }];
}
```

登录成功后，`PFUser`类将更新为包含 Facebook 信息。

以下代码将取消将 Facebook 账户从当前用户中链接：

```swift
[PFFacebookUtils unlinkUserInBackground:user block:^(BOOL succeeded, NSError *error) {
    if (succeeded) {
        NSLog(@"The user is no longer associated with their Facebook account.");
    }
}];
```

## 登出

要从应用程序中登出当前用户，您只需在`PFUser`类上调用`logOut`方法。以下代码将从应用程序中登出用户：

```swift
- (void)logoutButtonAction:(id)sender  {
    [PFUser logOut]; // Log out
    // Return to login page
}
```

## 保持用户登录

每次启动应用程序时用户都需要登录，这显得有些繁琐。这是因为 Facebook 会话信息在启动之间不会被保存。Parse 通过在`PFUser`类的`currentUser`对象中内部保存会话信息，使我们能够轻松地实现这一点。

在启动视图控制器中，您可以检查当前会话。如果存在，您可以绕过登录。以下代码将展示如何绕过登录：

```swift
- (void)viewDidLoad {
    ...
    if ([PFUser currentUser] && // Check if a user is cached
        [PFFacebookUtils isLinkedWithUser:[PFUser currentUser]]) // Check if user is linked to Facebook
    {
        // Push the next view controller without animation
        [self.navigationController pushViewController:
                                       [[UserDetailsViewController alloc]
                                       initWithStyle:UITableViewStyleGrouped]
                                   animated:NO];
    }
}
```

# 集成 Twitter

就像 Facebook 集成一样，Parse 提供了一个简单的方法来集成 Twitter 认证。只需几行代码，您就能使用 Twitter 登录并保存数据到 Parse Cloud。

## 设置 Twitter

您需要执行以下步骤来使用 Parse 设置 Twitter 认证：

1.  如果您还没有在 Twitter 上创建应用程序，您需要在 Twitter 开发者门户上创建您的应用程序（[`dev.twitter.com/apps`](https://dev.twitter.com/apps)）。

1.  在 Parse 应用程序的设置页面上添加您的 Twitter 应用程序的**消费者密钥**。

1.  为 Twitter 应用程序输入一个有效的**回调 URL**。这个值对于您控制这个回调 URL 非常重要，这样信息就不会在传输过程中被劫持。因此，以这种方式，这个值被 Twitter 用于认证目的。

1.  在您的`application:didFinishLaunchingWithOptions:`方法中添加以下代码，就在初始化 Parse 的代码下方：

    ```swift
    [PFTwitterUtils initializeWithConsumerKey:@"YOUR CONSUMER KEY"
                               consumerSecret:@"YOUR CONSUMER SECRET"];
    ```

在您的应用程序中设置 Twitter 的所有步骤如下。现在，我们可以允许用户使用 Twitter 进行登录。

## 注册和登录

Parse 提供了`PFTwitterUtils`类，允许您的用户使用 Twitter 进行登录。以下代码将允许用户使用 Twitter 进行注册：

```swift
[PFTwitterUtils logInWithBlock:^(PFUser *user, NSError *error) {
    if (!user) {
        NSLog(@"Uh oh. The user cancelled the Twitter login.");
        return;
    } else if (user.isNew) {
        NSLog(@"User signed up and logged in with Twitter!");
    } else {
        NSLog(@"User logged in with Twitter!");
    }     
}];
```

在执行前面的代码后，应该发生以下情况：

+   用户将通过弹窗或登录对话框被提示允许访问 Twitter 认证。

+   在授权应用程序后，应用程序将收到一个回调。

+   Parse 将在登录成功后接收数据，并将其保存到云上的`PFUser`类。

## 链接用户

如果是匿名登录或使用电子邮件 ID 和密码注册，您可以将现有的 PFUser 与 Twitter 账户关联起来。以下代码将展示如何将现有用户链接到 Twitter：

```swift
if (![PFTwitterUtils isLinkedWithUser:user]) {
    [PFTwitterUtils linkUser:user block:^(BOOL succeeded, NSError *error) {
        if ([PFTwitterUtils isLinkedWithUser:user]) {
            NSLog(@"User logged in with Twitter!");
        }
    }];
}
```

登录成功后，`PFUser`类将更新为包含 Twitter 信息。

以下代码将解除 Twitter 账户与当前用户的链接：

```swift
[PFTwitterUtils unlinkUserInBackground:user block:^(BOOL succeeded, NSError *error) {
    if (!error && succeeded) {
        NSLog(@"The user is no longer associated with their Twitter account.");
    }
}];
```

# 概述

在这一章中，我们探索了 Facebook 和 Twitter SDK 与 Parse SDK 的集成。

我们首先通过在 Facebook 开发者门户上设置应用程序来探索 Facebook 集成，然后看到了登录设置。

然后，我们学习了如何在 Facebook 上发布帖子，以及如何链接 Facebook 用户。

接下来，我们学习了如何在应用程序中集成。

最后，我们在应用程序中探索了 Twitter 集成和登录。

在下一章中，我们将学习云函数及其在我们应用程序中的使用。
