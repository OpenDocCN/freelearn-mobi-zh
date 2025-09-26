# 第八章：使用云函数

在应用程序开发中，并非所有过程都可以在客户端执行。因此，建议在云端执行此类过程。Parse 允许用户开发移动应用程序，绕过服务器端编码和管理。在开发复杂应用程序时，用户希望某些业务逻辑不在客户端执行。因此，对于此类应用程序，Parse 提供了 Parse Cloud，您可以在其中部署自定义逻辑，该逻辑可以通过您的应用程序访问。

云代码必须用 JavaScript 语言编写。区别在于执行位置；云代码将在 Parse Cloud 上执行，而不是在移动设备上执行。一旦云代码更新，它将立即对所有移动环境可用。这些功能可以帮助您轻松地即时更改应用程序的行为。在本章中，我们将学习云代码的实现及其在移动端的用法。

# 云代码

在开始使用云代码之前，您需要设置 Parse 命令行工具。此工具将帮助您管理和部署代码到云上。

## 安装命令行工具

要在 Mac/Linux 环境中安装 Parse 命令行工具，您需要在终端窗口中执行以下命令：

```swift
curl -s https://www.parse.com/downloads/cloud_code/installer.sh | sudo /bin/bash
```

上述代码行将在`/usr/local/bin/parse`目录下安装一个名为`parse`的工具。要卸载，只需删除文件即可，因为它没有安装任何垃圾文件。

## 设置云代码

要设置云代码，创建一个您想要保存云代码的目录。建议将云代码保存在您的项目工作区中。

命令`parse new`为您创建一个新的目录，并提示选择您要为它创建云代码的应用程序：

```swift
$ parse new MyCloudCode
Email: demo@gmail.com
Password:
1:DemoApp
Select an App: 1
$ cd MyCloudCode
```

您需要使用 Parse 账户的电子邮件地址和密码进行登录。如果使用`OAuth`登录，您需要从 Parse 设置中设置新密码以设置云代码。在成功执行前面的命令后，将为您创建以下文件结构：

```swift
-config/
  global.json
-cloud/
  main.js
-public/
  index.html
```

以下是对前面提到的文件的说明：

+   `config`目录中的 JSON 文件不应被编辑，它是供 Parse 使用的

+   `cloud`目录是您存储云代码的地方

+   初始时，`config`文件夹将包含`main.js`，它包含您的云函数

+   `public`目录将包含用于在 Parse 上托管静态数据内容的静态数据

## 创建云函数

要在云上创建函数，您需要将函数添加到您的`cloud`/`main.js`文件中。以下是一个`cloud`函数的示例：

```swift
Parse.Cloud.define("demoCloudCode", function(request, response) {
  response.success("Cloud integration is easy!");
});
```

在您的终端中运行以下命令以在 Parse Cloud 上部署代码：

```swift
$ parse deploy
```

一旦函数部署到云端，你可以在 Parse 应用的**云代码**部分查看云代码。部署的代码应该在那里发布。现在，是时候在移动端执行云代码了。Parse 为你提供了`callFunctionInBackground:withParameters:block:`方法，用于从移动端执行云代码。

您必须使用以下代码来执行云代码：

```swift
[PFCloud callFunctionInBackground:@"demoCloudCode"
                   withParameters:@{}
                            block:^(NSString *result, NSError *error) {
   if (!error) {
     NSLog(@"%@", result);
     // result is @"Cloud integration is easy!"
   }
}];
```

因此，在本节中，我们探讨了如何编写和执行云代码。

## 托管一个网站

在 Parse 上托管一个网站非常简单。你`public`目录中的所有文件都将托管在`your-custom-subdomain.parseapp.com`。以下代码将展示如何在 Parse Cloud 上托管你的网站。打开终端并执行以下代码：

```swift
$ echo "Hello World" > public/index.html
$ parse deploy
```

上一行代码将在您的`index.html`文件中添加`Hello World`，并且使用`deploy`命令，您的代码将被部署到 Parse 云端。

要更新您的子域名，请导航到您应用设置的**Web Hosting**部分。在**ParseApp 名称**字段中提供唯一的名称，您的所有网站文件都将可在`your-custom-subdomain.parseapp.com`上访问。

## 理解复杂函数

云端代码对于减少客户端的计算量非常重要。比如说，如果你的应用程序需要计算与 MBA 课程相关的学生的平均成绩。`Marks`对象将如下所示：

```swift
{
  "student": "John Melon",
  "marks": 5,
  "course": "MBA"
}
```

在这种情况下，你不应该获取一个长的数据列表并在客户端进行过滤。你可以添加一个用于过滤结果并提供结果数据的 Parse 代码。以下 Parse 云端代码将帮助你进行数据过滤：

```swift
Parse.Cloud.define("averageMarks", function(request, response) {
  var query = new Parse.Query("Marks");
  query.equalTo("course", request.params.course);
  query.find({
    success: function(results) {
      var sum = 0;
      for (var i = 0; i < results.length; ++i) {
        sum += results[i].get("marks");
      }
      response.success(sum / results.length);
    },
    error: function() {
      response.error("course lookup failed");
    }
  });
});
```

以下是需要注意的关于前一个 `Cloud` 函数的要点：

+   定义一个`cloud`函数，并对`Marks`对象进行查询

+   根据在`params query`中提供的`course`名称过滤`Marks`对象`params.query.equalTo("course", request.params.course);`

+   然后获取过滤后的数据，在完成计算后返回平均值

现在要在客户端使用云代码获取数据，你需要使用以下代码：

```swift
[PFCloud callFunctionInBackground:@"averageMarks"
                   withParameters:@{@"course": @"MBA"}
                            block:^(NSNumber *marks, NSError *error) {
  if (!error) {
     // marks is 45
  }
}];
```

之前的代码将调用名为 `averageMarks` 的 `cloud` 函数，并将 `MBA` 课程名称作为参数。在云代码成功执行后，你将获得 `marks` 作为响应，该响应将包含与 MBA 课程相关联的学生的平均分数。

## 使用云存储的代码

在多平台移动应用开发的情况下，云代码可以节省在客户端编写的大量代码。假设您有一个用于应用开发的 iOS、Android 或 Windows 环境，那么在这种情况下，您可以通过在云上部署验证代码来避免在所有环境中进行编码验证。云为您提供`beforeSave`方法。以下代码将展示其用法：

```swift
Parse.Cloud.beforeSave("averageMarks", function(request, response) {
  if (request.object.get("marks") < 1) {
    response.error("you cannot give less than one mark");
  } else if (request.object.get("marks") > 50) {
    response.error("you cannot give more than five marks");
  } else {
    response.success();
  }
});
```

通过这种方法，Parse 允许你在对象在云上保存之前添加自定义逻辑。在这个函数中，你可以在对象上添加验证代码。

同样，Parse 也为你提供了`afterSave`方法，这将帮助你提供自定义逻辑，该逻辑将在对象保存后执行：

```swift
Parse.Cloud.afterSave("averageMarks", function(request) {
  query = new Parse.Query("Marks");
  query.get(request.object.get("course").course, {
    success: function(course) {
      post.increment("marks");
      post.save();
    },
    error: function(error) {
      console.error("Got an error " + error.code + " : " + error.message);
    }
  });
});
```

之前的代码将在对象在云上保存后执行。这种方法允许你添加自定义逻辑，该逻辑将在对象保存后执行。

## 在删除代码中使用云

就像保存一样，你可以在删除任何对象之前和之后添加自定义代码。Parse 为你提供了`beforeDelete`和`afterDelete`方法，这将帮助你在这些事件上添加自定义逻辑。

以下代码将展示`beforeDelete`方法的用法。同样，你也可以使用`afterDelete`方法：

```swift
Parse.Cloud.beforeDelete("Marks", function(request) {
  query = new Parse.Query("Marks");
  query.equalTo("course", request.object.course);
  query.count({
    success: function(count) {
      if (count > 0) {
        response.error("Can't delete.");
      } else {
        response.success();
      }
    },
    error: function(error) {
      response.error("Error " + error.code + " : " + error.message + " when getting photo count.");
    }
  });
});
```

# 摘要

在这一章中，我们探讨了 Parse 云代码及其各种实现，以简化移动端的应用开发。

我们首先探索了命令行工具的安装过程，然后我们看到了在 Parse 上设置云代码。

我们还学习了编写云函数及其在客户端的用法。

然后，我们学习了如何在云上发布我们的网站。

最后，我们探讨了使用云的复杂场景和实现。

在下一章中，我们将学习关于 Parse 的错误处理和安全。
