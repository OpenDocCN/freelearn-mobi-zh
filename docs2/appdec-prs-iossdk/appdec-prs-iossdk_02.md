# 第二章：Parse 对象和查询

Parse 帮助你管理移动应用程序的完整后端结构，并取消开发复杂服务器端代码及其维护的需求。Parse 以对象的形式提供数据库结构。它以对象的形式保存所有你的数据，并在从 Parse 检索时返回相同的对象。

每个应用程序都有一个与 **Client Key** 相关的不同且特定的 **Application ID**，对于同一用户的同一应用程序，该 ID 保持不变。

Parse 基于面向对象的原则。所有对 Parse 的操作都将以对象的形式进行。Parse 以你发送的对象的形式保存你的数据，并帮助你以相同的格式再次获取数据。在本章中，你将了解对象以及可以在 Parse 对象上执行的操作。

在本章中，我们将学习如何与 Parse 对象一起工作，并编写查询以从 Parse 设置和获取数据。

# Parse 对象

Parse 中的所有数据都以 PFObject 的形式保存。当你通过执行查询从 Parse 获取任何数据时，结果将以 PFObject 的形式呈现。PFObject 的详细概念将在以下章节中解释。

## PFObject

存储在 Parse 上的数据以对象的形式存在，并且围绕 PFObject 开发。PFObject 可以定义为 JSON 数据的关键值（字典格式）对。Parse 数据是无模式的，这意味着你不需要提前指定每个 PFObject 上存在的键。Parse 后端将负责以你想要的任意键值对集合的形式简单地存储你的数据。

假设你正在使用你的应用程序跟踪具有用户 ID 的用户名的访问次数。一个单一的 PFObject 可以包含以下代码：

```swift
visitedCount:1122, userName:"Jack Samuel", userId:1232333332
```

Parse 只接受字符串作为 `Key`。值可以是字符串、数字、布尔值，甚至是数组、字典——任何可以 JSON 编码的内容。

PFObject 的类名用于区分不同类型的数据。比如说，你调用用户的 `visitedCounts` 对象。Parse 建议你编写类名 `NameYourClassLikeThis` 和 `nameYourKeysLikeThis`，以便提供代码的可读性。正如你在前面的示例中看到的，我们使用了 `visitedCounts` 来表示访问次数键。

# 对 Parse 对象的操作

你可以对 Parse 对象执行保存、更新和删除操作。以下是对可以在 Parse 对象上执行的操作的详细说明。

## 保存对象

要在 Parse 云上保存你的 `User` 表并添加额外的字段，你需要遵循类似于 `NSMutableDictionary` 方法的编码约定。更新数据后，你必须调用 `saveInBackground` 方法将其保存到 Parse 云。以下是一个示例，说明如何在 Parse 云上保存额外的数据：

```swift
PFObject *userObject = [PFObject currentUser];
[userObject setObject:[NSNumber numberWithInt:1122] forKey:@"visitedCount"];
[userObject setObject:@"Jack Samuel" forKey:@"userName"];
[userObject setObject:@"1232333332" forKey:@"userId"];
[userObject saveInBackground];
```

在执行上述代码后，您的数据已保存到 Parse Cloud。您可以在 Parse 应用程序的**数据浏览器**中检查您的数据。它应该类似于以下代码行：

```swift
objectId: "xWMyZ4YEGZ", visitedCount: 1122, userName: "Jack Samuel", userId: "1232333332",
createdAt:"2011-06-10T18:33:42Z", updatedAt:"2011-06-10T18:33:42Z"
```

这里有两点需要注意：

+   在运行您的代码之前，您不需要配置或设置一个新的名为 User 的类。Parse 将在第一次遇到它时自动创建该类。

+   还有一些字段您不需要指定，这些字段作为便利提供：

    +   `objectId`是每个保存对象的唯一标识符。

    +   `createdAt`和`updatedAt`表示每个对象在 Parse Cloud 中被创建和最后修改的时间。这些字段中的每一个都是由 Parse 填充的，因此它们在 PFObject 上不存在，直到保存操作完成。

    ### 注意

    您可以使用 Parse 提供的`saveInBackgroundWithBlock`或`saveInBackgroundWithTarget:selector:`方法在回调操作成功或失败后提供额外的逻辑：

    ```swift
    [userObject saveInBackgroundWithBlock:^(BOOL succeeded, NSError *error) {
            if (succeeded)
                NSLog(@"Success");
            else
                NSLog(@"Error  %@",error);
        }];
    ```

### 小贴士

**下载示例代码**

您可以从您在[`www.PacktPub.com`](http://www.PacktPub.com)的账户下载所有已购买的 Packt 书籍的示例代码文件。如果您在其他地方购买了这本书，您可以访问[`www.PacktPub.com/support`](http://www.PacktPub.com/support)并注册，以便将文件直接通过电子邮件发送给您。

## 获取对象

从 Parse Cloud 获取保存的数据甚至比保存数据更容易。您可以通过以下方式从 Parse Cloud 获取数据。

您可以使用 PFQuery 从其`objectId`获取完整对象。从云中获取数据的方法是异步的。您可以通过使用 Parse 提供的基于块的或基于回调的方法来实现：

```swift
PFQuery *query = [PFQuery queryWithClassName:@"GameScore"]; // 1
[query getObjectInBackgroundWithId:@"xWMyZ4YEGZ" block:^(PFObject  *gameScore, NSError *error) { //2
    // Do something with the returned PFObject in the gameScore variable.
   int score = [[gameScore objectForKey:@"score"] intValue];
   NSString *playerName = [gameScore objectForKey:@"playerName"]; //3
   BOOL cheatMode = [[gameScore objectForKey:@"cheatMode"] boolValue];
         NSLog(@"%@", gameScore);
}];
// The InBackground methods are asynchronous, so the code written after this will be executed
// immediately.  The codes which are dependent on the query result should be moved
// inside the completion block above.
```

让我们逐行分析如下：

+   第 1 行：它创建一个指向参数中给出的类名的查询对象。

+   第 2 行：它在第 1 行创建的查询对象上调用异步方法，以获取作为参数提供的`objectId`的完整对象。由于我们使用的是基于块的方法，我们可以在块内提供代码，该代码将在成功或失败时执行。

+   第 3 行：它从查询响应中获取的`PFObject`读取数据。

Parse 提供了一些所有 Parse 对象的公共值作为属性：

```swift
NSString *objectId = gameScore.objectId;
NSDate *updatedAt = gameScore.updatedAt;
NSDate *createdAt = gameScore.createdAt;
```

要刷新当前 Parse 对象，请输入：

```swift
[myObject refresh];
```

此方法可以在任何 Parse 对象上调用，这在您想要刷新对象数据时很有用。比如说，您想重新验证用户，那么您可以在用户对象上调用`refresh`方法来刷新它。

## 离线保存对象

Parse 为你提供了在用户离线时保存数据的函数。因此，当用户未连接到互联网时，数据将保存在本地对象中，一旦用户连接到互联网，数据将自动保存到 Parse 云上。如果你的应用程序在建立连接之前被强制关闭，Parse 将在下次应用程序打开时再次尝试保存对象。对于此类操作，Parse 为你提供了`saveEventually`方法，这样即使用户未连接到互联网，你也不会丢失任何数据。最终所有调用都是按照请求的顺序执行的。以下代码演示了`saveEventually`调用：

```swift
// Create the object.
PFObject *gameScore = [PFObject objectWithClassName:@"GameScore"];
[gameScore setObject:[NSNumber numberWithInt:1337] forKey:@"score"];
[gameScore setObject:@"Sean Plott" forKey:@"playerName"];
[gameScore setObject:[NSNumber numberWithBool:NO] forKey:@"cheatMode"];
[gameScore saveEventually];
```

## 更新对象

对于在 Parse 上更新对象，我们只需要通过调用任何保存函数将新数据提供给 Parse。

例如，假设你有一个保存在 Parse 上的对象的`objectId`。我们可以使用 Parse 的`PFQuery`获取相应的`PFObject`：

```swift
PFQuery *query = [PFQuery queryWithClassName:@"GameScore"];

// Retrieve the object by id
[query getObjectInBackgroundWithId:@"xWMyZ4YEGZ" block:^(PFObject *gameScore, NSError *error) {

    // Now let's update it with some new data. In this case, only cheatMode and score
    // will get sent to the cloud. playerName hasn't changed.
    [gameScore setObject:[NSNumber numberWithBool:YES] forKey:@"cheatMode"];
    [gameScore setObject:[NSNumber numberWithInt:1338] forKey:@"score"];
    [gameScore saveInBackground];

}];
```

执行前面的代码后，Parse 客户端将自动检测已更改的字段，并将仅更改的字段发送到 Parse 进行更新。在这里，我们使用`saveInBackground`方法异步地将我们的数据保存到 Parse 云上。

## 更新计数器

在某些情况下，你可能需要更新计数器，例如在游戏得分的情况下。然后，在这些情况下，为了增加键的值，你可以使用`incrementKey`方法，而对于减少值，你可以使用 PFObject 上的`decrementKey`方法：

```swift
[gameScore incrementKey:@"score"];
[gameScore saveInBackground];
```

我们甚至可以使用以下行代码按任何特定数字增加或减少计数器的值：

```swift
[gameScore incrementKey:@"score" byAmount:8];
```

上一行代码将`"score"`键增加`8`。

## 以数组格式存储数据

要以数组格式存储数据，Parse 提供了几种保存数据的方法。要将数据添加到键的现有值中，你可以使用以下代码片段。此代码将`"flying"`对象添加到`"skills"`键中：

```swift
[gameScore addObject:@"flying" forKey:@"skills"];
```

以下行代码将提供的对象数组添加到现有的云数组中。（键的值可以包含重复数据）： 

```swift
[gameScore addObjectsFromArray:[NSArray arrayWithObjects:@"flying", @"kungfu", nil] forKey:@"skills"];
```

在上一行代码中，数组的数据将被追加到现有的云数据中，而不检查数组值的唯一性。如果你想要避免在云上保存重复的值，你可以使用`addUniqueObjectsFromArray`方法来保存你的数组。此方法将确保只为键保存唯一的值。以下行代码将在提供的对象数组尚未添加的情况下将其添加到现有数组中：

```swift
[gameScore addUniqueObjectsFromArray:[NSArray arrayWithObjects:@"flying", @"kungfu", nil] forKey:@"skills"];
[gameScore saveInBackground];
```

## 删除对象

以下行代码将帮助你从 Parse 云中删除对象。此方法将异步删除你的对象：

```swift
[myObject deleteInBackground];
```

你可以使用`deleteInBackgroundWithBlock:`方法在对象删除后获得回调。

以下代码将删除背景中的对象，并返回响应中的成功或错误消息：

```swift
[ myObject deleteInBackgroundWithBlock:^(BOOL succeeded, NSError *error) {
        if (error) {
            // handle error
        }
        if (succeeded) {
            //code on success
        }
    }];
```

# 关系和数据类型

就像我们的数据库中的关系一样，我们有多张相互关联的表。Parse 为我们提供了在 Parse 类之间建立关系的便利。Parse 中的关系将在以下部分中详细解释。

## 关系

您可以在两个 Parse 对象之间添加关系。为此，我们可以在另一个`PFObject`中将一个`PFObject`作为值添加。

让我们举一个例子，我们有两个`Course`和`Student`对象，我们可以通过在它们的类之间建立关系来将课程与学生关联起来：

```swift
// Create course data
PFObject *myCourse = [PFObject objectWithClassName:@"Course"];
[myCourse setObject:@"MBA" forKey:@"courseName"];
[myCourse setObject:@"2 years" forKey:@"courseDuration"];

// Create student data
PFObject *studentData = [PFObject objectWithClassName:@"Student"];
[studentData setObject:@"Jack Samuel" forKey:@"name"];
[studentData setObject:[NSNumber numberWithInt:22] forKey:@"age"];

// Add a relation between the student and course
[studentData setObject:myCourse forKey:@"registeredCourse"];

// This will save both myPost and myComment
[studentData saveInBackground];
```

通过使用`objectId`也可以实现对象之间的关系：

```swift
// Add a relation between the Post with objectId "1zEcyElZ80" and the comment
[studentData setObject:[PFObject objectWithoutDataWithClassName:@"Course" objectId:@"1zEcyElZ80"] 
              forKey:@"registeredCourse"];
```

## 数据类型

Parse 支持所有数据类型格式，如`NSString`、`NSData`、`NSNumber`、`PFObject`、`NSDate`、`NSNull`和`NSData`。您甚至可以使用`NSDictionary`和`NSArray`的形式创建嵌套对象，以在单个`PFObject`中存储结构化数据。

让我们创建一些如下所示的随机变量：

```swift
NSNumber *number = [NSNumber numberWithInt:42];
NSString *string = [NSString stringWithFormat:@"the number is %i", number];
NSDate *date = [NSDate date];
NSData *data = [@"foo" dataUsingEncoding:NSUTF8StringEncoding];
NSArray *array = [NSArray arrayWithObjects:string, number, nil];
NSDictionary *dictionary = [NSDictionary dictionaryWithObjectsAndKeys:number, @"number",string, @"string",
nil];
NSNull *null = [NSNull null];

PFObject *bigObject = [PFObject objectWithClassName:@"BigObject"];
[bigObject setObject:number     forKey:@"myNumber"];
[bigObject setObject:string     forKey:@"myString"];
[bigObject setObject:date       forKey:@"myDate"];
[bigObject setObject:data       forKey:@"myData"];
[bigObject setObject:array      forKey:@"myArray"];
[bigObject setObject:dictionary forKey:@"myDictionary"];
[bigObject setObject:null       forKey:@"myNull"];
[bigObject saveInBackground];
```

# 处理查询

要从云端获取保存的数据，我们可以使用查询来获取所需的数据。

让我们从简单的查询开始。我们可以通过执行以下简单的代码行异步地从 Parse 获取数据：

```swift
PFQuery *query = [PFQuery queryWithClassName:@"GameScore"];

// Retrieve the object by id
[query getObjectInBackgroundWithId:@"xWMyZ4YEGZ" block:^(PFObject *gameScore, NSError *error) {

    // Now let's update it with some new data. In this case, only cheatMode and score
    // will get sent to the cloud. playerName hasn't changed.
    [gameScore setObject:[NSNumber numberWithBool:YES] forKey:@"cheatMode"];
    [gameScore setObject:[NSNumber numberWithInt:1338] forKey:@"score"];
    [gameScore saveInBackground];

}];
```

假设您想在从 Parse 获取结果时过滤结果，您可以在查询对象上添加条件，并检索过滤后的`PFObject`的`NSArray`。您可以使用以下方法从 Parse 云端查找数据：

```swift
findObjectsInBackgroundWithBlock: 
```

或者您可以使用：

```swift
findObjectsInBackgroundWithTarget:selector:
```

这两种方法都是异步的，将在主线程上执行回调。有一个名为`[query findObjects]`的方法；此方法将阻塞执行它的线程。

例如，如果您想获取注册 MBA 课程的学生的列表，我们可以使用以下代码：

```swift
PFQuery *query = [PFQuery queryWithClassName:@"Student"];
[query whereKey:@"registeredCourse" equalTo:@"MBA"];
[query findObjectsInBackgroundWithBlock:^(NSArray *objects, NSError *error) {
  if (!error) {
    // The find succeeded.
    NSLog(@"Successfully retrieved list of %d students.", objects.count);
  } else {
    // Log details of the failure
    NSLog(@"Error: %@ %@", error, [error userInfo]);
  }
}];
```

## 使用谓词

要向查询结果添加约束，我们可以使用以下方法：

```swift
NSPredicate *predicate = [NSPredicate predicateWithFormat:@"studentName = 'Jack Samuel'"];
PFQuery *query = [PFQuery queryWithClassName:@"Student" predicate:predicate];
```

Parse 支持以下类型的谓词功能：

+   您可以使用与键和常量一起使用的比较，如=、!=、<、>、<=、>=和 BETWEEN

+   包含谓词，例如 x IN {1, 2, 3}

+   存在性谓词，例如 x IN SELF

+   以 BEGINWITH 开头的表达式

+   使用 AND、OR 和 NOT 组合的复合谓词

+   使用“key IN %@”的子查询，subquery

以下类型的谓词不受 Parse 支持：

+   聚合操作，例如 ANY、SOME、ALL 或 NONE

+   正则表达式，如 LIKE、MATCHES、CONTAINS 或 ENDSWITH

+   比较两个键的谓词

+   具有许多 OR 子句的复杂谓词

## 添加查询约束

有多种方法可以添加查询结果的约束。以下是一些实现约束的方法：

+   为了找到查询中排除的指定学生之外的所有学生，Parse 提供了一个`whereKey:notEqualTo:`方法来从 Parse 云端获取选择性过滤数据。此方法有多种组合方式。以下代码将获取学生姓名不是 John Simon 的对象：

    ```swift
    [query whereKey:@"studentName" notEqualTo:@"John Simon"];
    ```

+   我们可以在同一个查询中添加多个约束，结果对象将匹配所有约束：

    ```swift
    [query whereKey:@"studentName" notEqualTo:@"John Simon"];
    [query whereKey:@"studentAge" greaterThan:[NSNumber numberWithInt:18]];
    ```

+   您可以通过设置查询对象的`limit`来限制查询的结果：

    ```swift
    query.limit = 10;
    ```

+   如果您只想获取结果中的第一个对象，可以使用`getFirstObject`或`getFirstObjectInBackground`。这两种方法都将返回从响应中接收到的对象数组中的第一个对象：

    ```swift
    PFQuery *query = [PFQuery queryWithClassName:@"Student"];
    [query whereKey:@"studentName" equalTo:@"John"];
    [query getFirstObjectInBackgroundWithBlock:^(PFObject *object, NSError *error) {
      if (!object) {
        NSLog(@"The getFirstObject request failed.");
      } else {
        // The find succeeded.
        NSLog(@"Successfully retrieved the object.");
      }
    }];
    ```

+   您可以通过设置`skip`来跳过结果。这也可以用于分页：

    ```swift
    query.skip = 10; // skip the first 10 results
    ```

+   Parse 提供了一个简单的方法来排序数字和字符串。它允许您控制返回结果的顺序：

    ```swift
    // Sorts the results in ascending order by the name field
    [query orderByAscending:@"studentName"];

    // Sorts the results in descending order by the name field
    [query orderByDescending:@"studentName"];
    ```

+   您可以通过在查询中提供比较来过滤结果：

    ```swift
    // Fetch list of students who scored < 50
    [query whereKey:@"score" lessThan:[NSNumber numberWithInt:50]];

    // Fetch list of students who scored <= 50
    [query whereKey:@"score" lessThanOrEqualTo:[NSNumber numberWithInt:50]];
    ```

+   您可以根据不同的值过滤结果。您可以为结果提供应出现在结果中的值的数组：

    ```swift
    // Finds scores from any of Jonathan, Dario, or Shawn
    NSArray *names = [NSArray arrayWithObjects:@"Jonathan Walsh",
      @"Dario Wunsch",@"Shawn Simon",nil];
    [query whereKey:@"studentName" containedIn:names];
           Fetch objects excluding the data provided in the array.
    // Finds scores from anyone who is neither Jonathan, Dario, nor Shawn
    NSArray *names = [NSArray arrayWithObjects:@"Jonathan Walsh",@"Dario Wunsch",@"Shawn Simon",nil];
    [query whereKey:@"studentName" notContainedIn:names];
    ```

# 关系和复杂查询

您可以通过在关系模式上执行操作来触发查询。以下是在关系模型下从 Parse 获取数据的各种方式：

## 基于数组的查询

如果您有数组格式的数据键，您可以找到包含值`x`的对象：

```swift
// Find objects where the array in arrayKey contains 2.
[query whereKey:@"arrayKey" equalTo:[NSNumber numberWithInt:2]];
```

如果您想获取匹配数组多个值的匹配数据，可以使用以下代码：

```swift
// Find objects where the array in arrayKey contains each of the
// elements 2, 3, and 4.
[query whereKey:@"arrayKey" containsAllObjectsInArray:@[@2, @3, @4]];
```

## 基于字符串的查询

您可以使用`where:hasPrefix:`在查询结果中添加约束：

```swift
// Finds student name that start with "Stuart".
PFQuery *query = [PFQuery queryWithClassName:@"Student"];
[query whereKey:@"name" hasPrefix:@"Stuart"];
```

## 关系查询

您可以从 Parse 中获取与特定 PFObject 字段匹配的对象。Parse 提供了一个`whereKey:equalTo:`方法来从查询中获取数据。假设您想获取已报名 MBA 课程的学生名单：

```swift
// Assume PFObject *myPost was previously created.
PFQuery *query = [PFQuery queryWithClassName:@"Student"];
[query whereKey:@"course" equalTo:myCourse];

[query findObjectsInBackgroundWithBlock:^(NSArray *studentList, NSError *error) {
    // list of student enrolled for MBA course
}];
```

如果您想在单个查询中获取多种相关对象，Parse 提供了一个`includeKey:`方法来获取这类关系结果。例如，如果您想获取得分最高的学生名单，并且想同时获取他们的课程，您可以使用以下代码片段来获取所需结果：

```swift
PFQuery *query = [PFQuery queryWithClassName:@"student"];

// Retrieve the top scorer
[query orderByAscending:@"score"];

// Only retrieve the top ten
query.limit = [NSNumber numberWithInt:10];

// Include the course data with each student score
[query includeKey:@"course"];

[query findObjectsInBackgroundWithBlock:^(NSArray *students, NSError *error) {
    // Top ten scorer Students from all courses
}];
```

## 计数对象

在某些情况下，您可能只需要知道结果数组的计数。为此，您不需要使用`findObjects`方法查询完整数据。Parse 提供了一个`countObjects`方法来获取 Parse 上查询数据的计数：

```swift
PFQuery *query = [PFQuery queryWithClassName:@"Student"];
[query whereKey:@"studentName" equalTo:@"Sean"];
[query countObjectsInBackgroundWithBlock:^(int count, NSError *error) {
  if (!error) {
    // The count request succeeded. Log the count
  } else {
    // The request failed
  }
}];
```

## 复合查询

如果您想获取多个查询的结果对象，Parse 提供了一个`orQueryWithSubqueries:`方法来获取这类结果。

假设您需要获取得分最高和最低学生的名单；以下代码片段给出了所需输出：

```swift
// Query to fetch the list of students with highest score
PFQuery *highestScorer = [PFQuery queryWithClassName:@"Student"];
[lotsOfWins whereKey:@"score" greaterThan:[NSNumber numberWithInt:150]];

// Query to fetch the list of students with lowest score 
PFQuery *lowestScorer = [PFQuery queryWithClassName:@"Player"];
[fewWins whereKey:@"score" lessThan:[NSNumber numberWithInt:5]];

PFQuery *query = [PFQuery orQueryWithSubqueries:[NSArray arrayWithObjects:highestScorer,lowestScorer,nil]];
[query findObjectsInBackgroundWithBlock:^(NSArray *results, NSError *error) {
  // results contains students with highest and lowest score.
  }];
```

## 缓存查询

缓存数据是一个过程，我们将从远程 API 调用接收到的数据临时存储在设备上。因此，对于相同的请求，你不需要反复获取数据。缓存通过最小化的 API 调用提高了效率、性能，并减少了服务器的负载。

一旦通过查询从 Parse 请求数据，建议缓存它，以减少对 Parse 云的访问次数。你可以在磁盘上缓存结果。此功能将帮助你以离线模式显示数据或在应用程序刚刚启动且数据尚未获取时。

默认情况下，`PFQuery` 不缓存查询；然而，你可以通过设置属性 `query.cachePolicy` 来启用 `PFQuery` 的缓存。让我们举一个例子，如果网络可达性不可用，你可以添加一个自动回退到缓存数据：

```swift
PFQuery *query = [PFQuery queryWithClassName:@"Student"];
query.cachePolicy = kPFCachePolicyNetworkElseCache;
[query findObjectsInBackgroundWithBlock:^(NSArray *objects, NSError *error) {
  if (!error) {
    // Results were successfully found, looking first on the
    // network and then on disk.
  } else {
    // The network was inaccessible and we have no cached data for
    // this query.
  }
}];
```

Parse 为你提供了以下缓存策略：

| `kPFCachePolicyIgnoreCache` | 这是默认的缓存策略。查询既不加载缓存也不将结果保存到缓存中。 |
| --- | --- |
| `KPFCachePolicyCacheOnly` | 查询只从缓存加载，忽略网络。如果没有缓存结果，将导致 `PFError`。 |
| `KPFCachePolicyNetworkOnly` | 它不加载缓存，但将结果保存到缓存中。 |
| `KPFCachePolicyCacheElseNetwork` | 它尝试从缓存加载，但如果查询失败，则从网络加载结果。如果缓存和网络都未成功，则会有 `PFError`。 |
| `KPFCachePolicyNetworkElseCache` | 这个查询首先尝试从网络加载，如果失败，则从缓存中加载结果。如果网络和缓存都未成功，你将得到一个 `PFError`。 |
| `KPFCachePolicyCacheThenNetwork` | 这个查询首先从缓存加载，然后从网络加载。在这种情况下，回调将被调用两次——首先使用缓存结果，然后使用网络结果。由于它在不同时间返回两个结果，因此此缓存策略不能与 `findObjects` 同步使用。 |

可以使用以下操作在缓存上控制缓存行为：

+   它帮助你检查查询的缓存结果：

    ```swift
    BOOL isInCache = [query hasCachedResult];
    ```

+   它清除查询对象的缓存结果：

    ```swift
    [query clearCachedResult];
    ```

+   它清除 `PFQuery` 的所有缓存：

    ```swift
    [PFQuery clearAllCachedResults];
    ```

+   你可以控制缓存结果存在的时长：

    ```swift
    query.maxCacheAge = 60 * 60 * 24;  // One day, in seconds.
    ```

# 摘要

在本章中，我们探讨了 Parse 对象以及查询 Parse 上可用数据的方式。

我们首先探索了 Parse 对象以及将这些对象保存到云上的方法。

然后，我们学习了查询，这些查询将帮助我们从 Parse 获取保存的数据。

最后，我们看到了实现查询和约束的多种方式，这些方式可以帮助我们减少查询的复杂性。

在下一章中，我们将学习在 Parse 云上保存我们的文件的各种方法，以及 Parse 对象的子类化概念。
