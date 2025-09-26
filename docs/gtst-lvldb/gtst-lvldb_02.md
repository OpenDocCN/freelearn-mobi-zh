# 第二章。安装 LevelDB 并为 iOS 构建

下载并解压 LevelDB 的基本步骤与 第一章 中所述的相同，即 *下载 LevelDB 并在 OS X 上构建*，但我们将重新构建库，并且需要根据 iOS 进行一些构建步骤的调整。为 iOS 构建被称为 **交叉编译**，因为生成的代码是为与编译器运行的处理器架构不同的架构。

# 为 iOS 构建静态 LevelDB 库

首先，我们将重新构建 `libleveldb.a` 文件。这次，我们将不带 snappy 进行构建，并为多个架构构建：模拟器的 32 位 x86 和 iOS 设备的 armv6 和 armv7。最后，我们将将其重命名为 iOS。

首先，在 snappy 目录的终端中移除 snappy 压缩：

```swift
$ make uninstall
 ( cd '/usr/local/share/doc/snappy' && rm -f ChangeLog COPYING INSTALL NEWS README format_description.txt framing_format.txt )
 ( cd '/usr/local/include' && rm -f snappy.h snappy-sinksource.h snappy-stubs-public.h snappy-c.h )
 /bin/sh ./libtool   --mode=uninstall rm -f '/usr/local/lib/libsnappy.la'
libtool: uninstall: rm -f /usr/local/lib/libsnappy.la /usr/local/lib/libsnappy.1.dylib /usr/local/lib/libsnappy.dylib /usr/local/lib/libsnappy.a

```

现在，将目录切换回您的 LevelDB 源目录，并清理，然后尝试为 iOS 构建。如果您使用的是 LevelDB 1.10，这将因 makefile 中的错误而失败：

```swift
$ make clean
…
$ CXXFLAGS="-std=c++11 -stdlib=libc++" make PLATFORM=IOS
…
make: /Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/Developer/usr/bin/c++: No such file or directory
make: *** [db/builder.o] Error 1

```

### 注意

LevelDB 1.10 中出现的错误已在 LevelDB 1.14.0 中修复

LevelDB v1.10 版本中包含的 makefile 存在一个错误（记录为问题 177 [`code.google.com/p/leveldb/issues/detail?id=177`](https://code.google.com/p/leveldb/issues/detail?id=177)），因为命令行编译器在 Xcode 4.3 之后移动了。苹果将这些编译器移动了多次，因此不同的开源项目版本会遇到这个问题。您可以从 Packt Publishing 网站获取相关的 `MakefileASD` 或按照以下描述编辑 makefile 的副本：

1.  在文本编辑器中打开 makefile，并向下滚动到以下部分：

    ```swift
    ifeq ($(PLATFORM), IOS)
    # For iOS, create universal object files
    ```

1.  您需要更改以下开始的两个行：

    ```swift
    $(DEVICEROOT)/usr/bin/$(C
    ```

1.  并且只需移除 `$(DEVICEROOT)/usr/bin/`，使它们从 `$(CC)` 和 `$(CXX)` 开始。

1.  假设您将此文件保存为 `MakefileASD`，您可以重复 make：

    ```swift
    $ CXXFLAGS="-std=c++11 -stdlib=libc++" make –f MakefileASD PLATFORM=IOS
    $ mv libleveldb.a /usr/local/lib/libleveldb_IOS.a
    ```

这将生成并复制静态库 `libleveldb_IOS.a`，使用标准的 C++11 libc++ 库，而不是默认库，因此它与默认的 Xcode 项目相匹配。如果您查看日志，您将看到使用了 `lipo` 命令来组合不同处理器类型的二进制文件。您也可以使用 `lipo` 检查库：

```swift
$ lipo -info /usr/local/lib/libleveldb_IOS.a
Architectures in the fat file: libleveldb_IOS.a are: armv6 armv7 i386

```

Lipo 可以检查或操作单个目标文件或编译库。

## 创建最小的 iOS 测试平台

首先，我们将创建一个尽可能简单的 iOS 程序，该程序可以在设备上运行并显示反馈。它将仅显示一个警报，表示我们已创建数据库。

如果您像前一章中推荐的那样使用工作区来组织项目，请在 Xcode 中打开该工作区。

现在，导航到**文件** | **新建** | **项目…**，这将显示模板选择器。在左侧面板中选择 iOS 应用程序，然后点击提供的图标中的**空应用程序**，然后点击**下一步**。确保勾选**使用自动引用计数**复选框，并取消勾选**包含单元测试**。确保指定**产品名称**和**公司标识符**。当你输入这些条目时，你会看到捆绑标识符是从它们生成的，例如，`Packt.LevelDB-iOS-Sample02`。

在项目导航器中点击以选择**AppDelegate**（例如，`GSwLDBAppDelegate.m`）源文件。这是我们为这次测试将要定制的唯一程序源文件。

对于这样一个简单的测试，我们只需要定制方法 `application: didFinishLaunchingWithOptions`，它最初包含：

```swift
- (BOOL)application:(UIApplication *)application
    didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
{
  self.window = [[UIWindow alloc] 
    initWithFrame:[[UIScreen mainScreen] bounds]];
  self.window.backgroundColor = [UIColor whiteColor];
  [self.window makeKeyAndVisible];
  return YES;
}
```

在 `return` 语句之前，添加以下行：

```swift
UIAlertView* helloWorldAlert = [[UIAlertView alloc]
    initWithTitle:@"Getting Started with LevelDB"
    message:@"Hello, LevelDB World!"
    delegate:nil
    cancelButtonTitle:@"OK" 
    otherButtonTitles:nil];
  [helloWorldAlert show];
  return YES;
```

从**运行**按钮附近的弹出菜单中选择一个**模拟器**方案，然后点击**运行**以编译并看到应用程序运行并显示小警报。

## 将 LevelDB 添加到 iOS 测试平台

我们基本上重复了为 OS X 项目所做的步骤：

+   将查找 LevelDB 标头的包含路径 `/usr/local/include` 添加到**用户头文件搜索路径**

+   将库 `/usr/local/lib/libleveldb_IOS.a` 添加到**链接二进制与库**面板

+   将 `/usr/local/lib` 添加到**库搜索路径**

要将一些 C++ 代码包含到我们的 Objective-C 文件中，我们需要告诉编译器将其视为 Objective-C++，这可以通过将文件扩展名从 `.m` 更改为 `.mm` 来实现。在项目导航器中点击文件名，然后按 *Enter* 键以能够编辑名称。

现在，你可以像在 `Sample01` 中那样添加 `#include` 语句：

```swift
#include <cassert>
#include "leveldb/db.h"
```

添加与我们在 OS X 中所做的相同的数据库语句，这样整个方法看起来就像以下代码：

```swift
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
{
    self.window = [[UIWindow alloc] initWithFrame:[
     [UIScreen mainScreen] bounds]];
    // Override point for customization after application launch.
    self.window.backgroundColor = [UIColor whiteColor];
    [self.window makeKeyAndVisible];
    leveldb::DB* db;
    leveldb::Options options;
    options.create_if_missing = true;
    leveldb::Status status = leveldb::DB::Open(
    options, "/tmp/testdbios", &db);
    assert(status.ok());

    UIAlertView* helloWorldAlert = [[UIAlertView alloc]
      initWithTitle:@"Getting Started with LevelDB"
      message:@"Hello, LevelDB World!"
      delegate:nil
      cancelButtonTitle:@"OK" otherButtonTitles:nil];
    [helloWorldAlert show];
    delete db;    
    return YES;
}
```

你可以再次点击**运行**并看到它在模拟器中运行。运行后，去查看目录 `/tmp/testdbios`，你将看到那里创建的文件。然而，这之所以可行，是因为模拟器是以对 OS X 文件系统完全访问的方式运行的。你无法以这种方式在 iOS 设备上的任意目录中创建文件。

连接一个已注册的 iOS 设备，并将**方案**更改为指向该设备，以便可以在设备上运行应用程序。现在运行应用程序。你会看到应用程序出现，但没有预期的警报。Xcode 的**输出**窗口显示一个错误消息，例如：

```swift
Assertion failed: (status.ok()), function -[GSwLDBAppDelegate application:didFinishLaunchingWithOptions:], file /Users/andydent/dev/learning/leveldb_OSX/LevelDB_IOS_Sample02/LevelDB_IOS_Sample02/GSwLDBAppDelegate.mm, line 27.

```

这是因为应用程序无法访问路径 `/tmp`。iOS 文件系统为每个应用程序都配备了**沙盒**，其中包含一些对操作系统具有不同意义的目录。其中一些是缓存，当空间不足时可以清除，而无需通知你的应用程序。还有一些是同步到 iCloud 的。

现在，我们将数据放入标准缓存区域 `NSCachesDirectory`（当前映射到 `Library/Caches`），这是用于临时数据，可能会被清除。

在你的 `GSwLDBAppDelegate.mm` 源文件中尽早添加以下辅助方法：

```swift
- (NSString*)pathForCachedDir:(NSString*)dirname {
  staticNSString* cachePath = nil;
  if (cachePath == nil) {  // save path first time per-runNSArray* paths =
      NSSearchPathForDirectoriesInDomains(NSCachesDirectory,NSUserDomainMask, YES);
    cachePath = [paths objectAtIndex:0];
    BOOL isDir = NO;
    NSError *error;if (! [[NSFileManagerdefaultManager] 
      fileExistsAtPath:cachePath isDirectory:&isDir] && 
      isDir == NO) { // create cache – first run on this device[[NSFileManagerdefaultManager] 
          createDirectoryAtPath:cachePath
          withIntermediateDirectories:NO
          attributes:nilerror:&error];}  }
  return [cachePath stringByAppendingPathComponent:dirname];
}
```

现在修复我们的数据库打开代码，替换以下行：

```swift
leveldb::Status status = leveldb::DB::Open(options, 
    "/tmp/testdbios", &db);
```

使用这两行代码，这些代码使用我们的辅助方法来查找缓存目录：

```swift
NSString* tempPath = [self pathForCachedDir:
   @"testdbios"];
leveldb::Status status = leveldb::DB::Open(options, 
  [tempPath UTF8String], &db);
```

现在在设备上运行将正常工作并显示警报。在模拟器上运行仍然有效，但会将文件放在一个会变化的位置。如果你在最后一行设置断点，你可以在设备上的调试器中看到 `tempPath` 的值，它将显示为：

```swift
/var/mobile/Applications/37742F10-B3D6-4945-A4CA-5F7764D4E33A/Library/Caches/testdbios 
```

但在模拟器中，它是在你的桌面上的绝对路径，例如：

```swift
/Users/andydent/Library/Application Support/iPhone Simulator/6.1/Applications/1017B648-DA52-4909-A1B2-18343722686A/Library/Caches/testdbios
```

在这个路径中，模拟器类型会变化，下一个目录是 iOS 版本，然后最终是库目录，这与设备上的相同。

# 摘要

再次构建 LevelDB 静态库，并学习了如何选择一个不同的标准 C++ 库以匹配 Xcode 的默认设置。创建一个简单的 iOS 应用程序，我们能够将其与适用于模拟器和 iOS 设备的 LevelDB 库链接起来。然后我们看到了它在模拟器中的运行情况与设备上沙盒文件系统的对比。
