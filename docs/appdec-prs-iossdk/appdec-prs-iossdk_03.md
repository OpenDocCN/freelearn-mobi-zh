# 第三章：子类和文件

Parse 为你提供了快速即插即用的方法来开始你的应用程序开发。你可以使用 `PFObject` 在 Parse 上保存和检索数据，并且你可以使用 `objectForKey` 方法访问对象中的任何数据字段。为了减少代码的复杂性，同时提高代码的可读性和可扩展性，你可以为你的 `PFObject` 使用子类。这也帮助你将应用程序架构调整为模型视图控制器架构。在本章中，我们将学习如何子类化 `PFObject` 以及在 Parse 云上保存文件的方法。

# 子类化 PFObject

PFObject 可以被子类化以提供代码的灵活性。基本上，它帮助你以面向对象的方式塑造后端代码，以提高可读性和应用程序架构。它还有助于使你的代码可重用。例如，你可以通过以下传统方式在 Parse 上保存你的数据：

```swift
       // Create/Save course data  
PFObject *myCourse = [PFObject objectWithClassName:@"Course"];
[myCourse setObject:@"MBA" forKey:@"courseName"];
[myCourse setObject:@"2 years" forKey:@"courseDuration"];
[myCourse saveInBackground];
```

子类化之后，你可以将之前的代码转换为以下代码：

```swift
Course *courseObject = [Course object];
courseObject.courseName = @"MBA";
courseObject.courseDuration = @"2 years";
[courseObject saveInBackground];
```

通过子类化 PFObject 创建 `Course` 对象，你可以轻松地与业务对象交互。此外，通过在代码中使用属性，它增加了代码库的可读性，从而简化了你的代码架构。

执行以下步骤以创建 PFObject 的子类：

1.  创建一个 `PFObject` 的子类以遵循 PFSubclassing 协议。

1.  实现类方法 `parseClassName`。此方法将返回作为 `string` 的类名，你将传递给 `initWithClassName:`。

1.  实现文件（`.m` 文件）导入 `PFObject + Subclass`。这个类包括 PFSubclassing 中的所有方法。

1.  在 `ApplicationDelegate` 中的 `userInfo setApplicationId:clientKey:` 调用之前，调用 `[YourClass registerSubclass]` 方法。

1.  以下代码演示了 `PFObject` 的 `Course` 子类：

    ```swift
    // Course.h
    @interface Course : PFObject<PFSubclassing>
    + (NSString *)parseClassName;
    @end

    // Course.m
    // Import this header to let your Course class know that PFObject privately 
    // provides most of the methods for PFSubclassing.
    #import <Parse/PFObject+Subclass.h>

    @implementation  Course
    + (NSString *)parseClassName {
      return @"Course";
    }
    @end

    // AppDelegate.m 
    #import <Parse/Parse.h>
    #import "Course.h"

    - (BOOL)application:(UIApplication *)application
    didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
      [Course registerSubclass];
      [Parse setApplicationId:parseAppId clientKey:parseClientKey];
    }
    ```

# 自定义属性和方法

为了封装你的自定义业务逻辑，你可以在你的 `PFObject` 子类中添加自定义属性和方法。通过使用 PFSubclassing 方法，你可以将代码简化并组织在一个地方，而不是让它散布在不同的类中。PFObject 支持属性的动态合成。你可以像通常声明属性一样声明属性；然而，这次你需要在实现文件中使用 `@dynamic` 代替 `@synthesis`。动态属性告诉编译器设置器和获取器方法不是由类实现的，而是在某个地方作为超类实现的。假设你需要在你 `Course` 类中添加一个属性。以下代码将属性添加到你的 `Course` 类中：

```swift
// Course.h
@interface Course : PFObject<PFSubclassing>
+ (NSString *)parseClassName;
@property (nonatomic, strong) NSString *courseName;
@property (nonatomic, strong) NSString *courseDuration;
@end

// Course.m
@dynamic courseName;
@dynamic courseDuration;
```

你可以使用 `course.courseName` 或 `[course courseName]` 来访问 `courseName` 和 `courseDuration` 属性，而如果你想设置数据到属性，你可以使用 `course.courseName = @"MBA"` 或 `[course setCourseName:@"MBA"]`。

如果你使用 `NSNumber` 作为数据类型，你可以按以下方式实现：

```swift
@property BOOL isGameOver;
@property float amount;
```

在获取数据时，你可以使用`course [object objectForKey:@"isGameOver"]`，这将返回`NSNumber`，你可以通过`boolValue`来访问它。而在处理金额时，你可以使用`floatValue`方法来访问。动态获取器方法会自动将`float`或`bool`值转换为，而动态设置器方法会将所有原始数据类型方法封装在`NSNumber`下。

## 初始化子类

使用`object`类方法创建新对象。这将自动创建一个提供类型的自动释放实例，并处理进一步的子类化。你可以使用这个`objectWithoutDataWithObjectId:`方法来创建当前对象的引用：

```swift
PFObject *parseObject = [PFObject objectWithoutDataWithClassName:@"Course" objectId:courseObject.objectId];
```

上一行代码将创建一个对现有 Parse 对象的引用；在我们的例子中是`Course`。

## 创建查询

你可以使用`query`类方法为子类对象创建查询。Parse 为你提供了`PFQuery`类，它允许你在 Parse 云上获取和保存数据。PFQuery 为你提供了许多设置和获取 Parse 云上数据的方法。假设你需要为你的`Course`类创建一个查询，你需要使用以下方法来创建查询：

```swift
PFQuery *query = [Course query];
[query whereKey:@"courseName" isEqualTo:PFUser.currentUser.courseName];
[query findObjectsInBackgroundWithBlock:^(NSArray *objects, NSError *error) {
  if (!error) {
   Course *firstCourse = [objects objectAtIndex:0];
    // Your code here
  }
}];
```

之前的代码创建了一个查询，使用`courseName`过滤器来获取你的`Course`类数据。你将得到一个当前用户订阅的所有课程的数组。此操作将异步执行，因为我们是在后台线程上执行此操作。PFQuery 为你提供了各种方法来获取和保存 Parse 云上的数据。

# 处理文件

在应用程序中，有时我们需要在数据库或后端存储我们的文件和数据。文件可以是任何类型，可以是我们的图片文件、视频文件、音频文件或数据文件。Parse 为我们提供了即插即用的功能，以便在云上保存数据和文件。

## PFFile

Parse 应用程序可以支持任何类型的文件，如图片、视频文件、音频文件或其他类型的数据文件，但它们的大小应该小于 10 兆字节。PFFile 帮助你将应用程序相关的文件存储在 Parse 云上。

为了在云上保存数据，你需要将你的文件转换为 NSData。然后你可以从该数据创建一个 PFFile 对象，并将 NSData 传递给 PFFile 对象以在云上保存：

```swift
NSData *data = [@"Hello world!" dataUsingEncoding:NSUTF8StringEncoding];
PFFile *exampleFile = [PFFile fileWithName:@"brochure.txt" data:data];
```

PFFile 提供了`fileWithName: data:`方法来在 Parse 云上保存你的文件。你可以在 Parse 上保存任何类型的文件，如图片、文本文件和数据文件。在将文件保存到 Parse 之前，你需要将所有文件转换为 NSData。在之前的代码中，我们在将`Hello world!`字符串保存到 Parse 云之前将其转换为 NSData。文件名应该是 NSString 格式，在我们的例子中，我们使用了`brochure.txt`作为文件名。

在之前的代码行中，我们使用了文件名`demo.txt`。请记住以下关键点：

+   您无需担心云端的文件名冲突，因为 Parse 会自动处理这些事情，为所有文件提供唯一的标识符，因此您可以拥有多个同名文件。

+   建议为您的数据文件提供扩展名，因为这有助于 Parse 理解您的文件格式并适当处理文件。因此，在存储图片时，您应使用 `.png` 或 `.jpg` 作为扩展名。

现在，您可以将文件保存到云端。所有保存方法都适用于 PFFile。您可以使用适合您需求的任何方法：

```swift
[file saveInBackground];
```

在成功保存您的文件后，您可以像关联其他数据一样将您的文件与 PFObject 关联：

```swift
PFObject *parseObject = [PFObject objectWithClassName:@"Course"]
[parseObject setObject:@"MBA" forKey:@"courseName"];
[parseObject setObject:exampleFile forKey:@"brochure"];
[parseObject saveInBackground];
```

上一行的代码将您的 `brochure` 文件与 `Course` 类关联。您可以使用以下代码从 Parse 获取相同的文件：

```swift
PFFile *fetchObj = [parseObject objectForKey:@"brochure"];
NSData *resumeData = [fetchObj getData];
```

要从 Parse 云端获取数据，您可以使用 `getData:` 方法及其变体。Parse 为您提供了 `getData` 方法的所有变体，就像我们之前看到的将 PFObject 保存到云端一样。

## 保存图片

要将您的图片保存到云端，您必须将您的图片转换为 NSData，然后您可以将这些数据保存到云端。假设您有一个图片，您需要将其保存到云端，那么您应该执行以下步骤：

```swift
UIImage *imageObj = [UIImage imageNamed:@"picture.png"];
NSData *imageData = UIImagePNGRepresentation(imageObj);
PFFile *imageFile = [PFFile fileWithName:@"picture.png" data:imageData];
[imageFile save];

PFObject *userPhoto = [PFObject objectWithClassName:@"UserPhoto"];
[userPhoto setObject:@"Test Image" forKey:@"imageName"];
[userPhoto setObject:imageFile forKey:@"imageFile"];
[userPhoto save];
```

在上一行的代码中，我们首先将一个图片文件转换为 NSData。然后我们创建了一个具有该数据作为 `picture.png` 图像键值的 PFFile 对象。然后我们在 PFFile 对象上调用 `save` 函数来保存文件。因此，现在这个 PFFile 对象可以用作任何其他 Parse 对象的值。接下来，我们在 `UserPhoto` 类上创建了一个对象，并将 PFFile 对象作为我们的 `imageFile` 键的值保存。然后最终我们在 `UserPhoto` Parse 对象上调用了 `save` 方法。

# 跟踪进度

Parse 帮助您通过以下提供的方法使用您的 PFFile 跟踪上传和下载的进度：

1.  使用以下方法，您可以跟踪您在云端保存的数据的进度：

    ```swift
    saveInBackgroundWithBlock:progressBlock
    ```

1.  使用以下方法，您可以在从云端获取数据时跟踪进度：

    ```swift
    getDataInBackgroundWithBlock:progressBlock
    ```

让我们通过一个例子来说明如何使用您的应用程序进行进度跟踪：

```swift
NSData *data = [@"Progress tracking of files is Easy!" dataUsingEncoding:NSUTF8StringEncoding];
PFFile *file = [PFFile fileWithName:@"brochure.txt" data:data];
[file saveInBackgroundWithBlock:^(BOOL succeeded, NSError *error) {
  // Handle success or failure here ...
} progressBlock:^(int percentDone) {
  // Update your progress spinner here. percentDone will be between 0 and 100.
}];
```

# 概述

在本章中，我们探讨了将我们的数据保存到 Parse 的各种方法。数据可以是任何东西，例如图片文件、音频文件、视频文件或简单的二进制数据文件。

我们首先探讨了子类化我们的 PFObject 的步骤。然后我们学习了 `PFFile` 类，它允许我们将数据保存到云端。我们还学习了将数据保存到云端以及将图片保存到云端的方法。

最后，我们看到了跟踪我们文件进度的各种方法。

在下一章中，我们将学习关于 Parse 分析以及跟踪您应用程序的各种方法。
