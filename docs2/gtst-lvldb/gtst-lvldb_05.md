# 第五章：使用 Objective-C

正如我们所看到的，LevelDB 的本地 C++接口相当简单。然而，大多数 OS X 和 iOS 程序员希望能够使用他们熟悉的`NSData`和`NSString`数据类型与数据库一起使用。**blocks**在 Cocoa API 中的应用正在稳步增加，作为一种将小块逻辑应用于数据集合的方式。有些人也非常反感使用 C++，会避免任何缺乏 Objective-C 接口的东西。

# Objective-C 中 LevelDB 的开源封装

有三个重要的 Objective-C 封装 LevelDB，所有这些都是在 2011 年宣布时开始的。它们是搜索 Objective-C LevelDB 时的顶级结果和链接。

两个都附带了自己的项目来构建库或框架，但截至 2013 年 6 月，这些项目与 Xcode 4.6 或更高版本不兼容。这两个都包含了一套有价值的单元测试，但没有 iOS 的示例。它们的相对流行度很难判断。

而不是试图修复他们的构建，我们提供的`Sample05`包含了所有三个封装的源代码和一套共同的测试项目。它使用了我们之前创建的 LevelDB 库。这种复制封装类的方法更有可能适应 Xcode 的变化，并允许你通过进入源代码进行调试。

`Sample05`是一个非常重要的示例，可以全面探索代码，因为它包含了到目前为止我们所看到的所有数据更新和迭代概念的 Objective-C 示例代码，每个封装都从 C++重写。这让你可以并排看到风格上的差异。本章中的代码示例使用了所有三个封装，以进行对比。

**LevelDB-ObjC**，来自[`github.com/hoisie/LevelDB-ObjC`](https://github.com/hoisie/LevelDB-ObjC)，是最早且最简单的封装，提供了以下功能：

+   一种使用`NSString`调用基本的`Put`、`Get`和`Delete`操作的方法

+   使用`NSKeyedArchiver`将对象和字典或对象数组存储起来，以将它们编码到值中

+   通过一个可以停止迭代的块遍历所有键

**APLevelDB**，来自[`github.com/preble/APLevelDB`](https://github.com/preble/APLevelDB)，部分基于 LevelDB-ObjC，试图成为一个更干净的 Objective-C 封装，并且：

+   添加了独立的迭代器和通过下标访问的类似数组的直接访问：`db[@"key"] = @"value"`

+   添加了`WriteBatch`支持的单独协议

+   清晰地将不同的概念分离到不同的协议中

+   而不是埋藏编码实现，获取和设置`NSData`，这样你可以进行自己的编码并通过`NSData*`传递

**NuLevelDB**，来自[`github.com/nulayer/NULevelDB`](https://github.com/nulayer/NULevelDB)，是一个非常雄心勃勃的层，包含许多类，具有以下特点：

+   `NSData`用于键和值，使得组合复杂的键以及像其他`NSString`键和值一样容易

+   分离优化的 64 位整数键和许多用于范围和任意整数键集合的枚举（迭代）方法

+   一致地使用 `NSError**` 参数来获取错误信息

+   在文件 `NULDBDB+BulkAccess.m` 中，轻松复制整个数据库或特定的键列表，到/从 `NSDictionary` 和 `NSArray`；

+   它自己的 `NULDBSerializable` 协议，可以将对象分解成一系列单独的键/值对。

以下列出的一些问题可能对每个人来说都不是问题。请记住，这些包装器是开源的，易于扩展，如果你缺少某个功能。以下是最严重的不同包装器问题，或者是最有可能让你放弃使用特定包装器的问题。我发现 APLevelDB 通过一些小的修改更容易扩展。问题如下：

+   LevelDB-ObjC 和 NuLevelDB 不符合 ARC 规范。

+   LevelDB-ObjC 公开了 `leveldb/db.h` 头文件，因此你必须将使用它的任何文件转换为 `.mm` Objective-C++ 源文件。

+   LevelDB-ObjC 和 APLevelDB 的迭代方法没有任何指定范围的方式，因此你失去了 LevelDB 的大部分功能。它们的块可以结束迭代，但你不能从一个给定的键或部分键开始。

+   NuLevelDB 具有非常丰富的迭代支持，但忽略了任何数据库的自定义比较器，并仅使用 LevelDB 的 `BytewiseComparator` 类方法。

+   它们都没有公开 LevelDB 的完整读写选项。NuLevelDB 在写 `sync` 和读 `fill_cache` 属性方面表现最佳。

### 提示

**ARC**（**自动引用计数**）在 OS X Lion 和 iOS 5 中被引入，并且现在在大多数新的 Objective-C 代码中都被使用。然而，许多较旧的源代码尚未转换为 ARC。要在 ARC 项目中使用非 ARC 源文件，请使用 **构建阶段** 选项卡，**编译源代码** 部分，并在需要时为每个文件添加标志 `–fno-objc-arc`。你会知道这是必要的，因为出现了 **ARC 限制** 编译器错误。

# 使用 Objective-C 进行简单的数据访问

我们在 第三章 中看到的 LevelDB-ObjC 包装器中的核心数据创建、查找和删除操作如下所示。

创建一个数据库对象并打开一个文件：

```swift
LevelDB* db = [[LevelDB alloc]
  initWithPath:pathToSampleDB(sampleDBname) ];
```

检查以确保键不存在，然后添加几条记录，检查 `Packt` 键是否工作，但小写 `packt` 仍然失败：

```swift
assert( [db getString:@"Packt"] == nil );
[db putObject:@"Getting Started" forKey:@"Packt"];
[db putObject:@"with Leveldb" forKey:@"Packt2"];
assert( [[db getString:@"Packt"]
  isEqualToString:@"Getting Started"] );
assert( [db getString:@"packt"] == nil );
```

再次写入现有的键，更改其值，并检查以确保它已更改：

```swift
[db putObject:@"Is Started" forKey:@"Packt"];
assert( [[db getString:@"Packt"] isEqualToString:@"Is Started"] );
```

使用不同的方式从任意字符创建一个 `NSString` 对象，并使用该键证明我们可以通过该键读取它：

```swift
const char enBuf[] = {"APrefix\0Packt in a string"};
NSString* keyWithNull = [NSString stringWithCharacters:(const
  unichar*)enBuf length:25];
[db putObject:@"Part Key with embedded null" forKey:keyWithNull];
assert( [[db getString:keyWithNull]
  isEqualToString:@"Part Key with embedded null"] );
```

删除一条记录并检查以确保它不能再被读取：

```swift
[db deleteObject:@"Packt"];
assert( [db getString:@"Packt"] == nil );
```

我们可以打印出刚刚添加的所有键，如 `Sample04`，但将使用一个块在第一个三个记录后停止。块的一个典型用途是从集合中处理数据，无论是键还是键值。大多数使用块的迭代器也允许块通过返回 `NO` 来停止迭代：

```swift
  __block int count = 0; // updateable counter outside the block
  [db iterateKeys:^(NSString* key) {
    printStr( key );
    return (++count < 3) ? YES : NO;  // block returns a BOOL
  }];
```

# 扩展 APLevelDB 以公开 C++ API

如果我们想在 APLevelDB 或其他包装器中添加自己的扩展，必须公开内部的 `leveldb::DB*`。仅仅获取这个指针就足够了，这样我们就可以使用我们在前面章节中看到的所有 C++ 逻辑。这需要对 `APLevelDB.h` 进行一些小的修改。

首先，为 `getDB` 声明一个返回类型，这样就可以安全地将其包含在纯 Objective-C 中，我们不需要强迫人们使用 `.mm` 文件迁移到 Objective-C++：

```swift
#ifdef __cplusplus // forward declaration
  namespace leveldb { class DB; }
  typedef leveldb::DB* leveldbDBPtr;
#else
  typedef void* leveldbDBPtr;
#endif
```

然后，在 `@interface APLevelDB : NSObject` 中添加一个公共获取器：

```swift
- (leveldbDBPtr) getDB;
```

这在 `ApLevelDB.mm` 中简单地实现：

```swift
- (leveldbDBPtr) getDB   {  return _db; }
```

现在，我们可以添加一个 **类分类** 来扩展 `APLevelDB` 并添加其他方法。这里只展示了一个例子。我们声明了一个接受一个前缀字符串并应用一个块到匹配该前缀的键的方法，传递一个 `BOOL` 参数，以便块可以停止枚举：

```swift
@interface APLevelDB (APLevelDB_ADSearches)
- (void)enumerateKeysWithPrefix:(NSString*)prefix
  block:(void (^)(NSString *key, BOOL *stop))block;
@end

@implementation APLevelDB (APLevelDB_ADSearches)
- (void) enumerateKeysWithPrefix:(NSString*)prefix
  Keys:(void (^)(NSString* key, BOOL* stop))block
{
  BOOL stop = NO;
  leveldb::Slice prefixSlice = SliceFromString(prefix);
  std::unique_ptr<leveldb::Iterator> iter(
    [self getDB]->NewIterator(leveldb::ReadOptions()) );
  for (iter->Seek(prefixSlice);
    iter->Valid() && iter->key().starts_with(prefixSlice);
    iter->Next()) {
  NSString *k = StringFromSlice( iter->key() );
  block(k, &stop);
  if (stop)
    break;
  } 
}
```

这是一个如何能够获取数据库指针来允许编写一些 Objective-C++ 代码以向 APLevelDB 添加方法的简单示例。可以采用类似的技术来扩展其他框架。这也展示了 Objective-C++ 如何允许我们混合语言。`enumerateKeysWithPrefix:block` 方法是 APLevelDB 的一个扩展，具有纯 Objective-C 接口。它接受一个块，可以是纯 Objective-C。C++ 的部分是调用 `NewIterator` 并通过记录循环调用块。然而，你也可以直接传递 `leveldb::DB*` 到纯 C++。

# 导入文本数据以加载数据库

以下函数展示了导入一个制表符分隔的文本文件并使用 APLevelDB 数据库生成记录的完整循环。`Sample500.txt` 文件包含在 OS X 和 iOS 项目中。我们在 `main06ios.m` 或 `main06osx.m` 中使用了一个小的辅助函数，以找到相对于应用程序的数据文件。这会找到一个打包在 Cocoa iOS 或 OS X 应用程序包中的文件，这是包含演示的常见方法：

```swift
NSString* pathToBundledData(NSString* filename)
{
  return [[NSBundle mainBundle]
    pathForResource:filename ofType:@""];
}
```

命令行 OS X 工具，如 `Sample05`，不会自动打包文件。为了将样本复制到相对于应用程序的位置，它被添加到 **构建阶段** 选项卡的 **复制文件** 设置中，设置相对路径 `./SampleData`，正如在辅助函数中使用的那样：

```swift
NSString* pathToBundledData(NSString* filename)
{
  return [@"./SampleData/" stringByAppendingString:filename];
}
```

这里是整个 `testAPLevelDBImportStrings` 函数。打开文本文件，将其读取到一个单独的字符串中，然后将其拆分为行数组：

```swift
NSString* tabSepStr = [NSString
  stringWithContentsOfFile:pathToBundledData(@"Sample500.txt")
  encoding:NSUTF8StringEncoding error:&openError];
NSArray* tabbedLines = [tabSepStr
  componentsSeparatedByCharactersInSet:
    [NSCharacterSet newlineCharacterSet]];
```

使用另一个辅助函数 `pathToSampleDB` 创建一个数据库：

```swift
NSError* openError;
APLevelDB* db = [APLevelDB
  levelDBWithPath:pathToSampleDB(@"testImportAPLeveldb05")
  error:&openError];
```

在一个`WriteBatch`中，为了速度和安全，遍历行数组，每行用制表符分隔，每行写两条记录。第一个键是姓氏和名字的组合，包含一个 JSON 编码的数组。第二个记录由电话号码字段键入，其值是第一个键，所以它就像一个二级索引。以下代码展示了这一点：

```swift
id tabSep = [NSCharacterSet   
  characterSetWithCharactersInString:@"\t"];
id<APLevelDBWriteBatch> wb = [db beginWriteBatch];  
for (NSString* line in tabbedLines) {
  NSArray* fields = [line
    componentsSeparatedByCharactersInSet:tabSep];
  NSString* nameKey = [NSString stringWithFormat:@"%@|%@",
    fields[1], fields[0]];
  NSError* encErr;
  NSData* enc = [NSJSONSerialization dataWithJSONObject:fields
    options:0 error:&encErr];
  [wb setData:enc forKey:nameKey];  // main record
  if ([fields[5] length] > 0)
    [wb setString:nameKey forKey:fields[5];
}
[db commitWriteBatch:wb];
```

这使用 Apple 的标准`NSJSONSerialization`来打包记录，它对数组进行简单的引号处理以存储为字符串，例如：

```swift
["Prince","Kauk","Mckesson Drug Co","AZ","85027","623-581-7435","prince@kauk.com"]
```

要解包记录，我们使用`dataForKey`获取数据，然后使用`JSONObjectWithData`方法重新创建一个数组：

```swift
NSData* jsonRec = [db2 dataForKey:@"Bayer|Reva"];
NSError* decErr;
NSArray* ja = [NSJSONSerialization
  JSONObjectWithData:jsonRec options:0 error:&decErr];
```

这种方法可以序列化字典和简单类型（如`NSString`）的数组，包括嵌套的数组和字典。如果你的数据主要是文本，它既紧凑又高效。使用`NSCoder`的其他变体让你可以存储和重新创建自己的对象，但你必须编写编码和解码方法的覆盖。

# 摘要

快速查看 LevelDB 的 Objective-C 包装器让我们了解了它们是如何用于我们在 C++中掌握的相同任务的。你也学会了加载与你的应用程序捆绑的文本数据的一般技术。文本数据加载器展示了一种新的方法来编码键的数据值并构建电话号码的二级索引。现在，我们将继续在这个基础上添加 Cocoa 用户界面，以便在比控制台输出窗口更令人兴奋的地方查看数据。
