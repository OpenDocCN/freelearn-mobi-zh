# 第九章。错误处理和安全

在处理云代码时，用户可能在查询中发送无效的参数。为了处理云代码上的此类条件，我们必须向这些查询发送错误代码。错误可以有多种形式，例如超时、ACL 列表、未定义的操作等等。

在本章中，我们将涵盖：

+   为了增强应用程序的稳定性以处理此类错误并相应地响应的错误处理

+   提高数据安全性的措施

+   导出你的 Parse 数据的方法

# 错误处理

在使用 Parse 的时候，你可能会遇到两种类型的错误。第一种是逻辑错误，这可能是由于你使用 SDK 的方式引起的。这种类型的错误通常会引发一个名为 `NSException` 的异常。

假设在使用登录的情况下，理想的登录流程是在调用 `PFUser` 类上的 `signUp` 方法之前提供用户名和密码。所以，根据以下代码，你绕过了用户名和密码，直接调用 `signUp`：

```swift
PFUser *user = [PFUser user];
[user signUp];
```

在这种情况下，操作将抛出 `NSInternalInconsistencyException` 异常，因为 `signUp` 方法被调用时没有用户名和密码属性。

与 Parse 云交互时，也可能会发生错误。错误可能是由于与 Parse 云的连接，也可能是由于任何请求的操作。让我们看看另一个例子：

```swift
- (void)fetchCourse {
    PFQuery *query = [PFQuery queryWithClassName:@"Course"];
    [query getObjectInBackgroundWithId:@"invalidObjectId"
                                target:self
              selector:@selector(callbackForFetch:error:)];
}
```

在前面的代码中，我们正在尝试获取在 Parse 云上不可用的对象。在这种情况下，Parse 将以字典格式抛出错误，其中错误代码可以通过 `error.code` 读取，消息可以通过 `error.userinfo` 读取。

以下代码演示了前一个方法调用的 `callback` 方法：

```swift
- (void)callbackForFetch:(PFObject *)result error:(NSError *)error {
    if (result) {
        NSLog(@"Success!");
    } else {
        if ([error code] == kPFErrorObjectNotFound) {
            NSLog(@"Object not available!");
        } else if (error) {
            NSLog(@"Error: %@", [[error userInfo] objectForKey:@"error"]);
        }
    }
}
```

在前面的代码中，你会注意到我们正在使用 `kPFErrorObjectNotFound` 来检查错误代码。Parse 提供了各种预定义的常量用于错误处理。你可以在 `PFConstants` 头文件中找到所有预定义的错误代码。

查询失败可能有其他几个可能的原因，例如设备无法连接到 Parse 云服务器。在这种情况下，你将收到一个包含错误描述的错误消息。为此，`callback` 方法将类似于以下代码片段：

```swift
- (void)callbackForFetch:(PFObject *)result error:(NSError *)error {
    if (result) {
        NSLog(@"Success!");
    } else {
        if ([error code] == kPFErrorObjectNotFound) {
            NSLog(@"Object not available!");
        // Now also check for connection errors:
        } else if ([error code] == kPFErrorConnectionFailed) {
            NSLog(@"Connection failed with Parse Cloud!");
        } else if (error) {
            NSLog(@"Error: %@", [[error userInfo] objectForKey:@"error"]);
        }
    }
}
```

当你向云代码发送 `NSNumber` 作为响应时，其布尔值用于检查操作是否成功。例如，你调用了一个 `cloud` 函数来获取学生的平均分数。在这种情况下，`callback` 方法应该类似于以下方法：

```swift
- (void)callbackForAverageMarks:(NSNumber *)result error:(NSError *)error {
    if ([result boolValue]) {
        NSLog(@"Success!");
    } else {
        if ([error code] == kPFErrorConnectionFailed) {
            NSLog(@"Connection failed with Cloud!");
        } else if (error) {
            NSLog(@"Error: %@", [[error userInfo] objectForKey:@"error"]);
        }
    }
}
```

在前一个方法中，如果结果布尔值为 `true`，则操作成功，否则操作失败并带有错误。错误处理对所有前台任务都是相同的。

所有通信的请求超时时间为 10 秒，因此同步调用不会长时间挂起。同步和异步任务都使用相同的连接超时。

Parse 为您提供错误代码列表及其解释。您可以在[`www.parse.com/docs/ios/api/Classes/PFConstants.html`](https://www.parse.com/docs/ios/api/Classes/PFConstants.html)上查看此列表。

# 保护您的应用程序数据

在软件行业中，尽可能保护您的应用程序数据被认为是最佳实践。为了保护您的数据，您可以使用 Parse 提供的访问控制列表，根据用户为对象提供安全性。建议在应用程序启动时自动启用匿名用户。还建议仅在所需的对象上提供公共可写性。此类设置将保护您的应用程序数据免受未经授权的访问。以下代码将说明在您的应用程序中使用安全性的用法：

```swift
[PFUser enableAutomaticUser];
PFACL *defaultACL = [PFACL ACL];
// Optionally enable public read access while disabling public write access.
// [defaultACL setPublicReadAccess:YES];
[PFACL setDefaultACL:defaultACL withAccessForCurrentUser:YES];
```

根据您在第六章“用户和角色”中学习到的内容，建议为所有对象提供 ACL。

您可以通过在您的**应用设置**页面上自定义以下设置来进一步增强您的应用程序的安全性：

+   如果您不使用它们，请禁用所有登录机制。例如，如果您的应用程序仅允许用户使用 Facebook 登录，那么请从设置中禁用所有其他登录方法。

+   在您的 Parse 应用程序设置页面上提供您的 Facebook**应用 ID**和 Twitter**消费者密钥**信息，以启用服务器端验证用户登录尝试。

# 导出数据

Parse 还允许您导出您的数据库。要导出数据，请导航到应用程序的**设置**页面并点击**导出数据**按钮。一旦您点击**导出数据**按钮，Parse 将压缩所有数据并将其导出到您的电子邮件 ID，该 ID 用于登录。导出的数据是 JSON 格式。ZIP 文件将包含所有类对象在一个单独的文件中。以下是从 Parse 导出的`Class`对象的示例数据：

```swift
{ "results": [
{
  "name": "buy groceries",
  "createdAt": "2013-07-03T11:04:05.062Z",
  "updatedAt": "2013-07-05T10:15:08.909Z",
  "objectId": "fYQ5ZfsM9m"
},
{
  "name": "team planning",
  "createdAt": "2013-08-15T07:22:51.976Z",
  "updatedAt": "2013-08-15T07:22:51.976Z",
  "objectId": "sBsAfr2Fm5"
}
] }
```

# 摘要

在本章中，我们探讨了 Parse 云代码及其各种实现，以简化移动端的应用程序开发。

我们首先探索了在项目中处理错误类型和方式。

然后，我们学习了如何增强应用程序的安全性。

最后，我们看到了如何从 Parse 导出我们的数据。
