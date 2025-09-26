# 第六章：用户和角色

如今，大多数应用程序都在处理用户账户，并帮助用户安全地访问他们的数据。Parse 为您提供了一个专门封装了处理用户数据所需所有功能的 `PFUser` 类。所有应用程序通常都包含某种形式的用户。借助 Parse 和 `PFUser` 类，您可以在应用程序中轻松集成用户认证和数据存储的功能。此类帮助您在应用程序中添加用户账户功能。

`PFUser` 类继承自 `PFObject`，因此为您提供了标准 `PFObject` 的所有功能，例如灵活的数据馈送模式，以及基于字典的关键值关系。`PFUser` 可以访问 `PFObject` 的所有方法，除此之外，`PFUser` 还有一些额外的帮助管理账户信息的方法。

在本章中，您将详细了解如何处理用户，以及如何为用户提供角色以限制对数据的访问。

# PFUser

此类允许您在 Parse 上管理用户账户。它有几个属性，使其与 `PFObject` 区分开来，如下所示：

+   `username`: 用户的用户名（必填）

+   `password`: 用户的密码（注册时必填）

+   `email`: 用户的电子邮件 ID（可选）

这些属性将内部调用 `setObject:forKey:` 方法，因此您不需要外部调用此方法。我们将在接下来的主题中详细探讨这些字段。

所有的 `PFUser` 对象都将存储在 `User` 类中。您可以在应用程序的 **数据浏览器** 中访问 `User` 类。此类还允许您为用户添加自定义字段，例如添加电话号码等相关细节。

# 用户登录

在大多数应用程序中，第一步是拥有一个有效的登录过程。Parse 为您提供了一个稳定的即插即用登录设置。您可以使用各种登录过程来验证用户，例如 Facebook、Twitter 或用户的电子邮件地址。这个过程将在以下主题中详细解释。

## 注册

您可以使用以下代码来展示允许用户使用电子邮件 ID 和密码注册的一种方式：

```swift
- (void)signUp {
    PFUser *user = [PFUser user];
    user.username = @"demo name";
    user.password = @"demo pass";
    user.email = @"demo@example.com";

    // other fields can be set just like with PFObject
    [user setObject:@"user display name" forKey:@"displayName"];

    [user signUpInBackgroundWithBlock:^(BOOL succeeded, NSError *error) {
      if (!error) {
          // Successfully signed up
      } else {
          NSString *errorString = [[error userInfo] objectForKey:@"error"];
          // Show the errorString somewhere and let the user try again.
      }
    }];
}
```

`signUpInBackgroundWithBlock:` 方法会异步（推荐）地在您的 Parse 应用程序中创建一个新用户。此方法封装了基本验证，例如电子邮件验证以及用户名和电子邮件 ID 的唯一性。Parse 将所有密码以哈希形式保存在云端。您还可以将用户的电子邮件 ID 作为其用户名保存。

Parse 上有各种 `signUp` 方法的版本。您可以从 API 文档中的方法列表中访问所有这些方法。

## 验证用户

Parse 还允许您验证用户的电子邮件 ID。在验证电子邮件 ID 后，`PFUser`类将添加`emailVerified`键。该字段包含 false 值，并且一旦用户通过其 ID 验证了他们的电子邮件地址，它就会更新为 true。

# 登录

在注册后，用户将登录其账户以访问应用程序。为了允许用户登录应用程序，您可以使用`PFUser`类的`logInWithUsernameInBackground:password:`类方法：

```swift
[PFUser logInWithUsernameInBackground:@"demoName" password:@"demopass"
  block:^(PFUser *user, NSError *error) {
    if (user) {
        // Successful login.
    } else {
        // The login failed. Check error to see why.
    }
}];
```

登录成功后，您可以访问`PFUser`对象，它将包含所有用户相关数据。

## 保存用户数据

对于应用程序的用户来说，每次打开应用程序时都进行登录是繁琐的。为了避免这种情况，您需要使用缓存来保存用户数据。保存用户详情将允许用户每次无需通过登录过程即可使用应用程序。`PFUser`类提供自动缓存策略，缓存对象保存在`currentUser`对象中：

```swift
PFUser *currentUser = [PFUser currentUser];
if (currentUser) {
    // do stuff with the user
} else {
    // show the signup or login screen
}
```

您可以通过在`PFUser`类上调用`logOut`方法来重置当前用户。

```swift
[PFUser logOut];
```

## 创建匿名用户

您可以允许用户使用您的应用程序，而无需强迫他们登录您的应用程序。可以创建一个没有用户名和密码的匿名用户，具有所有`PFUser`的功能。您可以通过提供 Facebook、Twitter 或电子邮件 ID 登录，随时将匿名用户转换为普通用户。

以下代码演示了创建匿名用户的方法：

```swift
[PFAnonymousUtils logInWithBlock:^(PFUser *user, NSError *error) {
    if (error) {
      NSLog(@"Anonymous login failed.");
    } else {
      NSLog(@"Anonymous user logged in.");
    }
}];
```

匿名用户的所有数据在注销后将不可访问。然而，如果用户注册为普通用户，所有数据都将持久化，并且可以被用户访问。

Parse 提供了检查用户是否已链接的方法。以下代码将演示检查用户是否为匿名的方法：

```swift
if ([PFAnonymousUtils isLinkedWithUser:[PFUser currentUser]]) {
    [self enableSignUpButton];
} else {
    [self enableLogOutButton];
}
```

# 重置密码

当用户开始登录您的应用程序时，他们经常会忘记用户名和密码。在这种情况下，您需要为用户提供一种重置密码的方式。Parse 为您提供了`requestPasswordResetForEmailInBackground:`方法，以安全地重置用户密码。

以下代码将说明重置密码功能的使用：

```swift
[PFUser requestPasswordResetForEmailInBackground:@"demo@example.com"];
```

重置密码链接将被发送到提供的电子邮件 ID，用户可以从那里输入应用程序的新密码。

## 编写用户查询

以下代码将演示如何在`PFUser`类上编写查询：

```swift
PFQuery *query = [PFUser query];
[query whereKey:@"gender" equalTo:@"female"]; // find all the women
NSArray *girls = [query findObjects];
```

上述代码将为您提供所有输出中的女性用户。

# 提供安全性

在保存用户数据时，确保所有用户数据都应该是安全的。用户数据和相关信息非常重要，它们应该安全地保存。Parse 允许您在云上安全地保存您的数据。

## 更新用户对象

`PFUser` 类已经得到了保护。在 `PFUser` 中保存的数据只能由该用户更新。此外，您不能在 `PFUser` 类上调用任何 `save` 或 `delete` 方法。唯一可以更新 `PFUser` 类上数据的 `logIn` 和 `signUp` 方法。这确保了只有用户可以更新他们在云上的数据：

```swift
The following code will illustrate the security of the user on cloud.
PFUser *user = [PFUser logInWithUsername:@"my_username" password:@"my_password"];
user.username = "my_new_username"; // attempt to change username
[user save]; // This succeeds, since the user was authenticated on the device

// Get the user from a non-authenticated method
PFQuery *query = [PFUser query];
PFUser *userAgain = (PFUser *)[query getObjectWithId:user.objectId];

userAgain.username = "another_username";

// This will throw an exception, since the PFUser is not authenticated
[userAgain save];
```

要获取用户对象的认证副本，请使用 `currentUser` 方法，您可以通过使用 `isAuthenticated` 布尔属性来检查用户的真实性。

## 保护对象

您也可以将 `PFUser` 的安全模型用于其他对象。您可以在 Parse 上保存的对象上指定读/写权限。您可以允许特定的用户组读取对象，以及其他可以修改对象的用户组。为了安全起见，每个对象都提供了一个访问控制列表，由 `PFACL` 类实现。

以下代码将展示如何设置当前用户在 Parse 上的对象隐私：

```swift
PFObject *privateNote = [PFObject objectWithClassName:@"Note"];
[privateNote setObject:@"This note is private!" forKey:@"content"];
privateNote.ACL = [PFACL ACLWithUser:[PFUser currentUser]];
[privateNote saveInBackground];
```

`ACLWithUser` 方法可以帮助您生成仅限制当前用户访问对象的 ACL。在调用 `save` 方法后，ACL 将被更新。受保护的数据将可供登录在设备上的用户访问。

您可以更新您的 Parse 对象的权限，针对不同的用户。以下两个方法将帮助您更新 Parse 对象的权限：

+   要提供读访问权限，请使用 `setReadAccess:forUser:`

+   要提供写访问权限，请使用 `setWriteAccess:forUser:`

以下代码将展示如何设置读/写访问权限：

```swift
PFObject *groupMessage = [PFObject objectWithClassName:@"Message"];
PFACL *groupACL = [PFACL ACL];

// userList is an NSArray with the users we are sending this message to.
for (PFUser *user in userList) {
    [groupACL setReadAccess:YES forUser:user];
    [groupACL setWriteAccess:YES forUser:user];
}

groupMessage.ACL = groupACL;
[groupMessage saveInBackground];
```

您也可以使用 `setPublicReadAccess:` 和 `setPublicWriteAccess:` 方法将读写权限授予所有用户。

要为应用程序中的所有对象提供一个通用的访问控制列表，您可以更改默认的 ACL：

```swift
PFACL *defaultACL = [PFACL ACL];
[defaultACL setPublicReadAccess:YES];
[PFACL setDefaultACL:defaultACL withAccessForCurrentUser:YES];
```

在前面的代码中，我们正在更新默认的 ACL，以提供对所有对象的公共访问权限。

# 分配角色

随着您的应用程序的增长，访问控制列表（ACL）变得不足。您需要更加约束，并采取更好的方法来控制用户访问。Parse 为您提供角色来解决此问题。角色允许您创建用户逻辑组。角色是一个包含用户和其他角色的对象。更新任何角色的权限将更新该角色中所有相关用户的权限。

例如，您可以将应用程序用户分为管理员、标准用户和访客角色。更改任何角色的权限将更新属于该组的所有用户的权限。提供角色功能的是 `PFRole` 类。这个类从 `PFObject` 继承，因此在 `PFRole` 中也可以访问所有方法。

## PFRole

`PFRole` 类除了 `PFObject` 中的属性外，还提供了以下附加属性：

+   `name`: 此字段将存储角色的名称，例如管理员或标准用户。

+   `users`：此字段将存储 `PFUser` 对象作为关系。用户将继承其关联角色的所有权限。

+   `roles`：此字段将保存 `PFRole` 对象的实际权限。

## 保护角色对象

`PFRole` 类使用与其他所有 Parse 对象相同的 ACL 安全机制。

以下代码将展示创建新的 `PFRole` 对象的方法：

```swift
// By specifying no write privileges for the ACL, we can ensure the role cannot be altered.
PFACL *roleACL = [PFACL ACL];
[roleACL setPublicReadAccess:YES];
PFRole *role = [PFRole roleWithName:@"Administrator" acl:roleACL];
[role saveInBackground];
```

之前的代码将创建一个新的 `role` 对象，现在它可以与 `User` 对象关联。用户访问控制将根据 `role` 对象进行更新。

以下代码将关联 `User` 与 `role` 对象，并使用角色提供的新 ACL 更新用户 ACL：

```swift
PFRole *role = [PFRole roleWithName:roleName acl:roleACL];
for (PFUser *user in usersToAddToRole) {
  [role.users addObject:user];
}
for (PFRole *childRole in rolesToAddToRole) {
  [role.roles addObject:childRole];
}
[role saveInBackground];
```

在分配 ACLs 到您的角色时要格外小心，因为它们将直接影响用户的可访问性。

## 保护其他对象

现在您已经学会了如何为用户提供角色。在用户对象的 `PFACL` 类中，您可以指定哪些用户和角色应该被授予对哪些对象的读取或写入访问权限。

以下代码将展示在 `PFObject` 中使用角色的用法：

```swift
PFObject *wallPost = [PFObject objectWithClassName:@"WallPost"];
PFACL *postACL = [PFACL ACL];
[postACL setWriteAccess:YES forRole:@"Administrator"];
wallPost.ACL = postACL;
[wallPost saveInBackground];
```

## 实现角色层次结构

您可以将一个角色作为另一个角色的子集提供。这意味着一个角色可以包含另一个角色。您可以在角色之间创建父子关系。这种关系的结果是，授予父角色的任何权限将隐式地授予其所有子角色。

例如，您可以说 `Administrator` 角色包含在 `Moderator` 角色中授予的所有权限。以下代码将解释角色层次结构的实现：

```swift
PFRole *administrators = /* Your "Administrators" role */;
PFRole *moderators = /* Your "Moderators" role */;
[moderators.roles addObject:administrators];
[moderators saveInBackground];
```

之前的代码将在 `Administrator` 和 `Moderator` 角色之间添加一个关系。在这里，`Administrator` 角色是 `Moderator` 的子角色。

# 摘要

在本章中，我们探讨了使用 Parse 的应用程序的用户和角色管理。

我们从探索具有其属性和登录过程的 `PFUser` 开始。

然后，我们学习了用户和其他对象的安全性。

最后，我们探讨了 `PFRole` 以扩展用户权限和安全，同时分组用户。

在下一章中，我们将学习如何在应用中使用 Parse 集成社交媒体（Facebook、Twitter）。
