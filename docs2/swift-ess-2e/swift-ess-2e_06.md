# 第六章 解析网络数据

许多 iOS 应用程序需要与其他服务器或设备进行通信。本章介绍了 Swift 中的 HTTP 和非 HTTP 网络以及如何从 JSON 或 XML 中解析数据。它首先演示了如何有效地从 URL 加载数据，然后是如何流式传输较大的数据响应。最后，它总结了如何执行除 HTTP 之外的协议的同步和异步网络请求。

本章将介绍以下主题：

+   从 URL 加载数据

+   从后台线程更新用户界面

+   解析 JSON 和 XML 数据

+   基于流的连接

+   异步数据通信

# 从 URL 加载数据

从远程网络源加载数的最常见方式是使用形式为 [`raw.githubusercontent.com/alblue/com.packtpub.swift.essentials/master/CustomViews/CustomViews/SampleTable.json`](https://raw.githubusercontent.com/alblue/com.packtpub.swift.essentials/master/CustomViews/CustomViews/SampleTable.json) 的 HTTP（或 HTTPS）URL。

可以使用来自 `Foundation` 模块（该模块通过 `UIKit` 模块间接导入）的 `NSURL` 类来操作 URL。主要的 `NSURL` 初始化器接受一个带有完整 URL 的 `String` 初始化器，尽管存在其他初始化器用于创建相对 URL 或对文件路径的引用。

`NSURLSession` 类通常用于执行与 URLs 相关的操作，可以通过初始化器创建单个会话，或者可以使用标准的**共享会话**。在旧版本的 iOS 和 Mac OS X 中使用了 `NSURLConnection` 类。在某些教程中仍可以看到对该类的引用，或者如果需要支持 Mac OS X 10.8 或 iOS 6，则可能需要该类；否则，应首选 `NSURLSession` 类。

`NSURLSession` 类提供了一种创建任务的方法。这些包括：

+   **数据任务**：这可以用于以编程方式处理网络数据

+   **上传任务**：这可以用于将数据上传到远程服务器

+   **下载任务**：这可以用于将数据下载到本地存储或恢复先前的或部分下载

任务是从 `NSURLSession` 类的方法创建的，可以接受一个 URL 参数和一个可选的*完成处理程序*。完成处理程序类似于代理，但它可以针对每个任务进行自定义，通常表示为一个函数。

任务可以被*暂停*或*恢复*以停止和启动过程。默认情况下，任务处于暂停状态，因此它们必须最初恢复以开始处理。

当数据任务完成时，完成处理程序会回调三个参数：一个表示返回数据的 `NSData` 对象，一个表示远程 URL 服务器响应的 `NSURLResponse` 对象，以及一个可选的 `NSError` 对象，如果请求过程中发生任何错误。 

在此基础上，上一章中创建的 `SampleTable` 可以通过获取会话、启动数据任务然后恢复来从网络 URL 加载数据。当数据可用时，完成处理程序会被调用，这可以用来将内容添加到表格中。

修改 `SampleTable` 类的 `viewDidLoad` 方法，通过在方法末尾添加以下代码来加载 `SampleTable.json` 文件：

```swift
let url = NSURL(string: "https://raw.githubusercontent.com/
  alblue/com.packtpub.swift.essentials/master/
  CustomViews/CustomViews/SampleTable.json")!
let session = NSURLSession.sharedSession()
let encoding = NSUTF8StringEncoding
let task = session.dataTaskWithURL(url,completionHandler:
 {data,response,error -> Void in
  let contents = String(data:data!,encoding:encoding)!
  self.items += [(url.absoluteString,contents)]
 // table data won't reload – needs to be on ui thread
  self.tableView.reloadData()
})
task.resume()
```

这将创建一个 `NSURL` 和一个 `NSURLSession`，然后创建数据、任务并立即恢复。内容下载完成后，会调用完成处理程序，将数据作为 `NSData` 对象传递。使用 `String` 初始化器从 `NSData` 对象中解码 `UTF8` 文本，并将其显式转换为 `String` 类型，以便将其添加到 `items` 数组中。

### 小贴士

`NSURLSession` 类还提供了其他工厂方法，包括一个接受配置参数的方法，该参数包括选项，例如是否应该缓存响应、网络连接是否应该通过蜂窝网络进行，以及是否应该将任何 cookie 或其他头信息与任务一起发送。

最后，将项目添加到 `items` 中，并重新加载 `tableView` 以显示新数据。请注意，如果不在主 UI 线程上运行，则不会立即生效；表格需要旋转或移动以重新绘制显示。在 UI 线程上运行将在本章后面的 *网络和用户界面* 部分进行介绍。

## 处理错误

错误是生活中不可避免的事实，尤其是在网络连接间歇性的移动设备上。完成处理程序会调用第三个参数，它表示在操作过程中引发的任何错误。如果这是 `nil`，则表示操作成功；如果不为 `nil`，则可以使用 `error` 的 `localizedDescription` 属性来通知用户。

为了测试目的，如果检测到错误，请将 `localizedDescription` 添加到列表中的 `items`。按照以下方式修改 `viewDidLoad` 方法：

```swift
let task = session.dataTaskWithURL(url, completionHandler:
 {data,response,error -> Void in
  if error == nil {
    let contents = String(data:data!,encoding:encoding)!
    self.items += [(url.absoluteString,contents)]
  } else {
 self.items += [("Error",error!.localizedDescription)]
 }
 // table data won't reload – needs to be on UI thread
  self.tableView.reloadData()
})
```

可以通过在 URL 中使用不存在的域名或未知协议来模拟错误。

## 处理缺失内容

如果无法联系远程服务器，例如当域名不正确或服务器关闭时，会报告错误。如果服务器正在运行，则不会报告错误；但仍然有可能请求的文件找不到，或者服务器在处理请求时遇到错误。这些错误会通过 HTTP 状态码报告。

### 注意

如果找不到 HTTP URL，服务器会返回一个 `404` 状态码。客户端可以使用这个状态码来判断是否应该访问不同的文件或者是否应该查询另一个服务器。例如，浏览器通常会请求一个 `favicon.ico` 文件，并使用它来显示一个小图标；如果这个文件缺失，则显示一个通用的页面图标。一般来说，`4xx` 响应是客户端错误，而 `5xx` 响应是服务器错误。

`NSURLResponse`对象没有 HTTP 状态码的概念，因为它可以用于任何协议，包括`ftp`。然而，如果请求使用了 HTTP，那么响应很可能是 HTTP，因此它可以转换为一个`NSURLHttpResponse`，它有一个`statusCode`属性。这可以用来在文件找不到时提供更具体的反馈。按照以下方式修改代码：

```swift
if error == nil {
  let httpResponse = response as! NSHTTPURLResponse
 let statusCode = httpResponse.statusCode
 if (statusCode >= 400 && statusCode < 500) {
 self.items += [("Client error \(statusCode)",
 url.absoluteString)]
 } else if (statusCode >= 500) {
 self.items += [("Server error \(statusCode)",
 url.absoluteString)]
 } else {
    let contents = String(data:data!,encoding:encoding)!
    self.items += [(url.absoluteString,contents)]
  }
} else {...}
```

现在，如果服务器响应但表示客户端提出了一个错误请求或服务器遇到了问题，用户界面将相应地更新。

## 嵌套的 if 和 switch 语句

有时，错误处理逻辑可能会因为处理不同的案例而变得复杂，尤其是当需要测试不同值时。在前面的章节中，需要检查`NSError`和 HTTP 的`statusCode`。

使用带有`where`子句的`switch`语句是另一种方法。这些可以用来测试多个不同的条件，并显示正在测试的条件部分。尽管`switch`语句需要一个单一的表达式，但可以使用*元组*将多个值组合成一个单一表达式。

使用元组的一个优点是它允许根据类型进行情况匹配。在网络案例中，一些 URL 基于`http`或`https`，这意味着响应将是一个`NSHTTPURLResponse`类型。然而，如果 URL 是不同类型（例如`file`或`ftp`协议），那么它将是`NSURLResponse`的不同子类型。使用`as`无条件地转换到`NSHTTPURLResponse`在这些情况下将失败并导致崩溃。

测试可以重写为一个`switch`块，如下所示：

```swift
switch (data,response,error) {
  case (_,_,let e) where e != nil:
    self.items += [("Error",e.localizedDescription)]
  case (_,let r as NSHTTPURLResponse,_) 
   where r.statusCode >= 400 && r.statusCode < 500:
    self.items += [("Client error \(r.statusCode)",
     url.absoluteString)]
  // see note below
  case (_,let r as NSHTTPURLResponse,_) 
   where r.statusCode >= 500:
    self.items += [("Server error \(r.statusCode)",
      url.absoluteString)]
  default:
    let contents = String(data:data!,encoding:encoding)!
    self.items += [(url.absoluteString,contents)]
}
```

在这个例子中，`default`块用于执行成功条件，而之前的`case`语句用于匹配错误条件。

`case (_,_,let e) where e != nil`情况是一个*条件模式匹配*的例子。在 Swift 中，下划线被称为*通配符模式*（在其他语言中也称为**空槽**），它可以匹配任何值。第三个参数`let e`是一个*值绑定模式*，在这个情况下相当于`let e = error`。最后，`where`子句添加了测试以确保只有当`e`不是`nil`时，此情况才会发生。

### 小贴士

在`case`语句中，可以使用标识符`error`而不是`let e`，使用`case (_,_,_) where error != nil`将产生相同的效果。然而，在`switch`语句外部捕获值用于情况匹配是不良的做法，因为如果`error`变量被重命名，那么`case`语句可能会变得无效。通常，在`case`语句中使用`let`模式以确保匹配正确的表达式值。

第二和第三个情况同时执行一个`let`赋值和一个类型测试/转换。当`case (_,let r as NSHTTPURLResponse,_)`匹配时，不仅元组中该部分的值被赋给常量`r`，它还被转换为`NSHTTPURLResponse`类型。如果该值不是`NSHTTPURLResponse`类型，则自动跳过该情况语句。这相当于一个带有`is`表达式的`if`测试，后面跟着一个使用`as`的转换。

虽然两种模式相同，但`where`子句不同。第一个`where`子句寻找`r.statusCode`为 400 或更大但小于 500 的情况，而第二个匹配的是`r.statusCode`为 500 或更大的情况。

### 小贴士

无论使用嵌套的`if`语句还是`switch`语句，执行测试的代码可能非常相似。这通常取决于开发者的偏好，但更多的开发者可能更熟悉嵌套的`if`语句。在 Swift 中，`switch`语句比其他语言更强大，因此这种模式可能变得更加流行。

在 Swift 2 中，可以使用`guard`语句来确保如果发生某些错误条件，可以采取适当的行动。`guard`语句类似于`if`语句，但没有`true`块，并且`false`块必须始终离开函数。例如，代码可以被重写为：

```swift
guard error == nil else {
  self.items += [("Error",error!.localizedDescription)]
  return
}
let statusCode = (response as! NSHTTPURLResponse).statusCode

guard statusCode < 500 else {
  self.items += [("Server error \(statusCode)",
    url.absoluteString)]
  return
}

guard statusCode < 400 else {
  self.items += [("Client error \(statusCode)",
   url.absoluteString)]
  return
}

let contents = String(data:data!,encoding:encoding)!
self.items += [(url.absoluteString,contents)]
```

请注意，`guard`块必须退出调用函数；因此，如果需要额外的操作，要么将实现体的主体移动到不同的函数中，要么使用`switch`或`if`块。本章后面的示例假设使用`if`块以保持简单。

## 网络和用户界面

当前回调方法的一个突出问题是无法保证回调函数一定会在*主线程*中被调用。因此，用户界面操作可能无法正确执行或引发错误。正确的解决方案是使用主线程设置另一个调用。

在 Swift 中访问主线程的方式与在 Objective-C 中相同：使用**Grand Central Dispatch**（**GCD**）。可以使用`dispatch_get_main_queue`访问*主队列*，这是所有 UI 更新应使用的线程。后台任务通过`dispatch_async`提交到队列。要在主线程上调用`reloadData`，可以这样包装：

```swift
dispatch_async(dispatch_get_main_queue(), {
  self.tableView.reloadData()
})

```

这种调用方式对 Objective-C 和 Swift 都有效（尽管 Objective-C 使用^（尖括号）作为块前缀）。然而，Swift 为接受块的函数有特殊的语法；块可以被提升出函数的参数，并作为尾随参数留下。这被称为*尾随闭包*：

```swift
dispatch_async(dispatch_get_main_queue()) {
  self.tableView.reloadData()
}

```

虽然这只是微小的差异，但它使得`dispatch_async`看起来更像是一个关键字，例如`if`或`switch`，它接受一段代码块。这可以用于任何最终参数是函数的函数；在函数定义中不需要特殊语法。此外，相同的技巧也适用于在 Swift 外部定义的函数；在`dispatch_async`的情况下，该函数被定义为 C 语言函数，并且可以透明地以可移植的方式使用。

## 在主线程上运行函数

每当 UI 需要更新时，更新必须在主线程上运行。这可以通过使用之前的模式来完成，因为它们将始终在线程中执行。然而，每次需要时都记住这样做可能会很痛苦。

可以构建一个 Swift 函数，该函数接受另一个函数并在主线程上自动运行。`NSThread.isMainThread`可以用来确定当前线程是否是 UI 线程；因此，要在主线程上运行代码块，无论它是否在主线程上，可以使用以下方法：

```swift
func runOnUIThread(fn:()->()) {
  if NSThread.isMainThread() {
    fn()
  } else {
    dispatch_async(dispatch_get_main_queue(), fn)
  }
}
```

这允许使用以下方式将代码提交到后台线程：

```swift
self.runOnUIThread(self.tableView.reloadData)
```

### 小贴士

由于缺少括号，`reloadData`函数没有被调用，但它被作为函数指针传递。它在`runOnUIThread`函数内部被调度到正确的线程。

如果需要调用多个函数，可以创建一个内联块。由于这可以作为`runOnUIThread`方法的尾随闭包传递，因此括号是可选的：

```swift
self.runOnUIThread {
  self.tableView.backgroundColor = UIColor.redColor()
  self.tableView.reloadData()
  self.tableView.backgroundColor = UIColor.greenColor()
}
```

# 解析 JSON

在网络上发送结构化数据最流行的机制是将数据编码为**JSON**，代表**JavaScript 对象表示法**。这提供了一个分层树形数据结构，可以存储简单的数字、逻辑和基于字符串的类型，以及数组和字典表示。

Mac OS X 和 iOS 都内置了 JSON 文档的解析器，在`NSJSONSerialization`类中。这提供了一种解析数据对象并返回包含 JSON 对象键/值对的`NSDictionary`或表示 JSON 数组的`NSArray`的方法。其他字面量被解析，并作为`NSNumber`或`NSString`值表示。

JSON 解析器使用`JSONObjectWithData`从包含字符串的`NSData`对象创建一个对象。这通常是网络 API 返回的格式，并且可以使用`dataUsingEncoding`与内置编码类型之一（如`NSUTF8StringEncoding`）从现有字符串创建。

一个简单的数字 JSON 数组可以按以下方式解析：

```swift
let array = "[1,2,3]".dataUsingEncoding(NSUTF8StringEncoding)!
let parsed = try? NSJSONSerialization.JSONObjectWithData(
  array, options:.AllowFragments)
```

这个返回类型是一个可选的`AnyObject`。可选性表示数据内容可能不是有效的 JSON 数据。可以使用`as`关键字将其转换为适当类型；如果解析失败，则会抛出错误。

可以使用 `options` 来指示返回类型是否应该是可变的。可变数据允许调用者在解析函数返回后添加或删除项目；如果没有指定，返回值将是不可变的。`NSJSONReadingOptions` 选项包括 `MutableContainers`（包含数据结构是可变的）、`MutableLeaves`（子叶是可变的）和 `AllowFragments`（允许解析非对象、非数组值）。

`SampleTable.json` 文件（在 `viewDidLoad` 方法中引用）存储一个条目数组，每个条目的 `title` 和 `content` 字段包含文本数据：

```swift
[{"title":"Sample Title","content":"Sample Content"}]
```

要解析 JSON 文件并将条目添加到表中，将 `SampleTable` 中的 `default` 子句替换为以下内容：

```swift
default:
  let parsed = try? NSJSONSerialization.JSONObjectWithData(
    data!, options:.AllowFragments) as! NSArray
  for entry in parsed {
    self.items += 
      [(entry["title"] as! String,
        entry["content"] as! String)]
  }
```

运行应用程序将在表中显示 **样本标题** 和 **样本内容** 条目，这些条目已从书籍的 GitHub 仓库加载并解析。

## 处理错误

如果解析 JSON 数据时出现问题，则 `try? JSONObjectWithData` 函数的返回类型将为 `nil` 值。如果类型是隐式解包的，则访问元素将导致错误：

```swift
do {
 let parsed = try NSJSONSerialization.JSONObjectWithData(data!,
 options:.AllowFragments) {
  // do something with parsed
} catch let error as NSError {
  self.items += [("Error", 
   "Cannot parse JSON \(error.localizedDescription)")]
  // show message to user
}
```

`parsed` 值的类型将是 `AnyObject?`，尽管 `let` 块会隐式解包该值，这被称为 *可选绑定*。在上一节中，代码被直接转换为 `NSArray`，但如果返回的结果包含不同类型（例如，`NSDictionary` 或片段类型 `NSNumber` 或 `NSString`），则尝试转换为与运行时类型不兼容的类型将导致失败。

可以使用 `if [object] is [type]` 来测试对象的类型。然而，由于下一步通常是将它转换为不同的类，使用 `as`，一个简写形式 `as?` 可以在一步中完成测试和转换：

```swift
 if let array = parsed as? NSArray {
  for entry in array {
    // process elements
  }
} else {
  self.items += [("Error", "JSON is not an array")]
}
```

`switch` 语句可以同时检查多个值的类型。由于这些值是可选的 `AnyObject` 对象，因此在 Swift 中使用之前需要将它们转换为 `String`：

```swift
for entry in array {
  switch (entry["title"], entry["content"]) {
    case (let title as String, let content as String):
      self.items += [(title,content)]
    default:
      self.items += [("Error", "Missing unknown entry")]
  }
}
```

现在当应用程序运行时，任何错误都会被检测和处理，而不会导致应用程序崩溃。

# 解析 XML

虽然 JSON 更常用，但仍有许多基于 XML 的网络服务。幸运的是，iOS 自 5 以来就存在 XML 解析，在 `NSXMLParser` 类中，并且从 Swift 中访问很简单。例如，一些数据源（如博客文章）使用 XML 文档，如 Atom 或 RSS。

`NSXMLParser` 是一个面向流的解析器；也就是说，它会按顺序报告看到的各个元素。解析器调用 `delegate` 通知元素被看到和完成。当看到元素时，解析器还会包括任何存在的属性；对于文本节点，包括字符串内容。解析 XML 文件涉及解析器中的某些状态管理。本节中使用的示例将解析 Atom（新闻源）文件，其（简化后的）结构如下所示：

```swift
<feed >
  <title>AlBlue's Blog</title>
  <link href="http://alblue.bandlem.com/atom.xml" rel="self"/>
  <entry>
    <title type="html">QConLondon and Swift Essentials</title>
    <link href="http://alblue.bandlem.com/2015/01/qcon-swift-essentials.html"/>
    ... 
  </entry>
  ...
</feed> 
```

在这个情况下，目标是提取出所有来自源的数据元素`entry`，特别是`title`和`link`。这会带来一些挑战，这些挑战将在稍后变得明显。

## 创建解析代理

解析 XML 文件需要创建一个遵守`NSXMLParserDelegate`协议的类。为此，创建一个新的类`FeedParser`，它扩展`NSObject`并遵守`NSXMLParserDelegate`协议。

它应该有一个`init`方法，该方法接受一个`NSData`，并且有一个`items`属性，用于在解析后获取结果：

```swift
class FeedParser: NSObject, NSXMLParserDelegate {
  var items:[(String,String)] = []
  init(_ data:NSData) {
    // parse XML
  }
}
```

### 小贴士

`NSXMLParserDelegate`协议要求对象也遵守`NSObjectProtocol`。最简单的方法是子类化`NSObject`。首先提到的超类型是父类；第二个和后续的超类型必须是协议。

## 下载数据

XML 解析器可以解析下载时的数据流，或者它可以接受之前已下载的`NSData`对象。在成功下载后，可以使用`FeedParser`解析`NSData`实例并返回项目列表。

虽然可以给单个表达式分配与上次相似的临时值，但可以将语句写在一行中（尽管请注意，这里没有错误处理）。将以下内容添加到`SampleTable`类的`viewDidLoad`方法末尾：

```swift
session.dataTaskWithURL(
  NSURL(string:"https://alblue.bandlem.com/Tag/swift/atom.xml")!,
  completionHandler: {data,response,error -> Void in
    if let data = data {
      self.items += FeedParser(data).items
      self.runOnUIThread(self.tableView.reloadData)
    }
}).resume()
```

这将下载作者博客中 Swift 文章的 Atom XML 源，博客地址为[`alblue.bandlem.com`](https://alblue.bandlem.com)。目前，数据尚未解析，因此在此步骤中不会向表中添加任何内容。

### 小贴士

确保下载操作和解析都在主线程之外处理，因为这两个操作可能需要一些时间。一旦数据下载完成，就可以解析它，解析完成后，可以通知 UI 重新显示内容。

## 解析数据

为了处理下载的 XML 文件，需要解析数据。这涉及到编写一个解析代理来监听`title`和`link`元素。然而，`title`和`link`元素既存在于单个`entry`级别，也存在于博客的顶级。因此，在解析器中需要表示某种状态，以便检测解析器是否在`entry`元素内部，从而允许使用正确的值。

元素通过`parser:didStartElement:`方法和`parser:didEndElement:`方法报告。这可以用来确定解析器是否在`entry`元素内部，通过在`entry`元素开始时设置一个布尔值，并在元素结束时重置它。将以下内容添加到`FeedParser`类中：

```swift
var inEntry:Bool = false
func parser(parser: NSXMLParser,
 didStartElement elementName: String,
 namespaceURI: String?, qualifiedName: 
 String?, attributes: [String:String]) {
  switch elementName {
    case "entry":
      inEntry = true
    default: break
  }
}
```

`link`将元素的引用值存储在元素的`href`属性中。这会在调用开始元素时传递，因此存储是微不足道的。在此点，标题可能未知，因此`link`的值必须存储在可选字段中：

```swift
var link:String?
...
// in parser:didStartElement method
case "entry":
  inEntry = true
case "link":
 link = attributes["href"]
default break;
```

`title` 将其数据存储为文本节点，需要使用另一个布尔标志来指示解析器是否在 `title` 节点内部。文本节点通过 `parser:foundCharacters:` 代理方法报告。将以下内容添加到 `FeedParser`：

```swift
var title:String?
var inTitle: Bool = false
...
// in parser:didStartElement method
case "entry":
  inEntry = true
case "title":
 inTitle = true
case "link":
...
func parser(parser: NSXMLParser, foundCharacters string:String) {
 if inEntry && inTitle {
 title = string
 }
}

```

当看到 `entry` 元素的末尾时，将 `title` 和 `link` 存储为可选字段，可以将这些字段附加到 `items` 列表中，然后重置解析器的状态：

```swift
func parser(parser: NSXMLParser,
 didEndElement elementName: String,
 namespaceURI: String?, qualifiedName: String?) {
  switch elementName {
    case "entry":
      inEntry = false
      if title != nil && link != nil {
        items += [(title!,link!)]
      }
      title = nil
      link = nil
    case "title":
      inTitle = false
    default: break
  }
}
```

最后，在实现了回调方法之后，接下来的步骤是从之前传递的数据中创建一个 `NSXMLParser`，设置 `delegate`（以及可选的命名空间处理），然后调用解析器：

```swift
init(_ data:NSData) {
  let parser = NSXMLParser(data: data)
  parser.shouldProcessNamespaces = true
  super.init()
  parser.delegate = self
  parser.parse() }
```

### 小贴士

在调用 `super.init` 之后才能将 `self` 赋值给 `delegate`。

现在当应用程序运行时，将显示一组新闻源项目。

### 小贴士

如果在 iOS 9 目标上运行并从 http 网站下载，控制台可能会看到 **App Transport Security 已阻止明文 HTTP 资源加载** 的消息。修复此问题的解决方案是在 `Info.plist` 文件中添加一个异常，允许通过 HTTP 进行连接，无论是显式域名还是所有域名。在第一个 `<dict>` 元素之后添加以下内容到 `Info.plist`：

```swift
<key>NSAppTransportSecurity</key>
<dict>
  <key>NSAllowsArbitraryLoads</key>
  <true/>
</dict>
```

现在当应用程序运行时，错误应该不再出现。

# 直接网络连接

虽然大多数应用程序网络将涉及通过标准协议（如 HTTP(S)）下载内容，并使用标准表示，但有时需要特定的数据流协议。在这种情况下，*流*导向的过程将允许读取或写入单个字节，或者可以使用*数据报*或*数据包*导向的过程来发送单个数据包。

有网络库支持这两种情况；一个基于 Objective-C 的高级 `NSStream` 类提供了一个驱动基于流的响应的机制，尽管使用 `CoreFoundation` 或 `POSIX` 层可以实现低级别的数据包连接，但使用 `MultipeerConnectivity` 模块进行本地多人游戏通常是合适的。

### 注意

使用 `MultipeerConnectivity` 模块进行本地网络涉及创建一个 `MCSession`，然后通过 `sendData` 发送 `NSData` 对象到已连接的节点，并使用 `MCSessionDelegate` 从已连接的节点接收数据。这通常用于同步世界的状态，例如玩家的当前位置或健康。

## 打开基于流的连接

流是一个可靠、有序的字节序列，大多数互联网协议都使用流。可以使用 `NSStream` 类的 `getStreamsToHostWithName` 方法从网络主机和端口创建流。这允许同时获取 `NSInputStream` 和 `NSOutputStream`。

### 注意

由于这是一个现有的 Objective-C API，流通过 *输入输出参数* 返回。在 Swift 中，这表示参数通过 ampersand (`&`) 返回，并将变量声明为可选的。

然后，可以使用这些输入/输出流异步或同步地发送数据。异步机制涉及在应用程序的运行循环上调度数据处理，这在 *异步读写* 部分有所介绍。同步机制使用 `read` 和 `write` 来接收或发送数据缓冲区。

### 小贴士

一旦获取了流，它们需要被 *打开* 以接收或发送数据。忘记这一步将导致没有网络数据被发送。

为了简化获取流，可以将以下内容作为 `NSStream` 类的扩展来创建。扩展使方法看起来来自原始类，但实际上是在类外部实现的。在 `CustomViews` 项目中添加一个 `StreamExtensions.swift` 文件，内容如下：

```swift
extension NSStream {
  class func open(host:String,_ port:Int)
   -> (NSInputStream, NSOutputStream)? {
    var input:NSInputStream?
    var output:NSOutputStream?
    NSStream.getStreamsToHostWithName(
      host, port: port, 
      inputStream: &input,
      outputStream: &output)
    guard let i = input, o = output else {
      return nil
    }
    o.open()
    i.open()
    return (i,o)
  }
}
```

通过调用 `NSStream.open(host,port)` 可以获得对远程主机的连接，该方法返回一个打开的输入/输出流对。

## 同步读写

`NSInputStream` 的 `read` 方法允许同步地从流中读取字节，而 `NSOutputStream` 的 `write` 方法允许将字节写入流。它们接受不同的类型，但在 Swift 中最常见的方法是创建一个字节数组 `[UInt8]` 作为缓冲区，然后使用 `UnsafeMutablePointer`（在 C 中相当于 ampersand）读取到或从中。

`read` 和 `write` 方法都返回读取/写入的字节数。这可能为负（错误情况），为零，或者为正（已处理字节的情况）。这两个调用都接受一个缓冲区和最大长度，尽管不能保证会处理完整的最大长度。

### 小贴士

总是检查 `write` 或 `read` 的返回值，因为可能只有缓冲区的一部分被写入。对于同步连接，最佳实践是使用 `while` 循环包装调用或具有其他形式的 `retry`，以确保所有数据都被写入。

### 向 NSOutputStream 写入数据

为了更容易将 `NSData` 内容写入流，可以在 `NSOutputStream` 上创建一个扩展方法，该方法根据数据的大小执行完整写入：

```swift
extension NSOutputStream {
  func writeData(data:NSData) -> Int {
    let size = data.length
    var completed = 0
    while completed < size {
      let wrote = write(UnsafePointer(data.bytes) +
       completed, maxLength:size - completed)
      if wrote < 0 {
        return wrote
      } else {
        completed += wrote
      }
    }
    return completed
  }
}
```

此代码接受一个 `NSData` 并将其写入底层流，返回写入的字节数（如果有问题，则返回负值）。检查 `write` 方法的返回值，如果值为负，则直接返回给调用者。否则，使用写入的字节数增加 `completed` 计数器。

如果写入的字节数达到请求的数据大小，则返回该值。否则，循环递归，这次从上次离开的点开始。

### 注意

虽然在 Swift 中不常见，但可以通过获取 `data.bytes` 数组的 `UnsafePointer` 来执行指针算术，然后通过已写入的字节数增加它。剩余字节的长度通过 `size-completed` 计算。

### 从 NSInputStream 读取

可以使用类似的方法从 `NSInputStream` 读取完整缓冲区，通过创建一个返回已知大小字节数组的 `readBytes` 方法，以及将其转换为 `NSData` 以便于处理/解析的方法：

```swift
extension NSInputStream {
  func readBytes(size:Int) -> [UInt8]? {
    let buffer = Array<UInt8>(count:size,repeatedValue:0)
    var completed = 0
    while completed < size {
      let read = self.read(
       UnsafeMutablePointer(buffer) + completed,
       maxLength: size - completed)
      if read < 0 {
        return nil
      } else {
        completed += read
      }
    }
    return buffer
  }
  func readData(size:Int) -> NSData? {
    if let buffer = readBytes(size) {
      return NSData(
       bytes: UnsafeMutablePointer(buffer),
       length: buffer.count)
    } else {
      return nil
    }
  }
}
```

`readData` 方法返回一个 `NSData`，而 `readBytes` 方法返回一个 `UInt8` 值的数组。使用 `NSData` 方法在某些情况下很有用（尤其是，从返回的数据中创建一个 `String`），而在其他情况下，能够直接处理字节很有用（例如，解析二进制格式）。两者都存在使得可以根据需要使用任一方法。

### 提示

同步读取可能会永远阻塞；如果客户端应用程序请求恰好 10 个字节，但服务器只发送了 9 个字节，那么它将永久挂起，直到发送了第十个字节。最佳实践是使用异步读取，这样就不会以这种方式阻塞。

### 读取和写入十六进制和 UTF8 数据

能够将数据作为 UTF8 值或十六进制值处理在某些协议中可能很有用。尽管 `NSString` 和 `NSData` 都提供了转换到和从 UTF8 的方法，但其语法过于冗长，因为它基于现有的 Objective-C 方法。

为了便于转换，可以创建扩展方法以提供简单地将转换到和从 UTF8 表示的方法。除了类和实例函数之外，还可以使用扩展向现有对象添加动态属性。这可以通过在文件 `Extensions.swift` 中添加扩展来实现，如下所示：

```swift
extension NSData {
  var utf8string:String {
    return String(data:self,
     encoding:NSUTF8StringEncoding)!
  }
}
extension String {
  var utf8data:NSData {
    return self.dataUsingEncoding(
      NSUTF8StringEncoding, allowLossyConversion: false)!
  }
}
```

这允许使用更紧凑的表达式，例如 `data.utf8string` 和 `string.utf8data`。每次表达式被评估时，相关的 getter 函数将被调用。

### 提示

在本书编写时，Swift 中没有命名扩展的标准约定。如果有对单一类型数据的扩展——例如之前提到的流——那么文件可以命名为 `[Type]Extensions.swift`。或者，可以使用该名称来命名被调用的方法类型；例如，在这种情况下，可以使用 `UTF8Extensions.swift`。

可以将解析十六进制数据从字符串和整数添加到 `String` 和 `Int` 类型中，如下所示：

```swift
extension String {
  func fromHex() -> Int {
    var result = 0
    for c in self.characters {
      result *= 16
      switch c {
      case "0":result += 0      case "1":result += 1
      case "2":result += 2      case "3":result += 3
      case "4":result += 4      case "5":result += 5
      case "6":result += 6      case "7":result += 7
      case "8":result += 8      case "9":result += 9
      case "a","A":result += 10 case "b","B":result += 11
      case "c","C":result += 12 case "d","D":result += 13
      case "e","E":result += 14 case "f","F":result += 15
      default: break
      }
    }
    return result;
  }
}
extension Int {
  func toHex(digits:Int) -> String {
    return String(format:"%0\(digits)x",self)
  }
}
```

这允许使用 `int.toHex` 和 `string.fromHex` 创建十六进制值。

### 实现 Git 协议

可以编写一个客户端，使用 `git://` 协议查询远程 git 服务器，以确定远程标签/分支/引用的哈希值。

### 注意

`git://` 协议通过发送带有 ASCII 码前缀的四位十六进制数字的数据包行来工作，这些数字表示其余数据（包括这四个初始数字）的长度。发送 `git-upload-pack` 请求将返回远程仓库上的引用列表。

由于 `git://` 协议使用数据包行，因此创建一个 `PacketLineExtensions.swift` 文件，内容如下：

```swift
extension NSOutputStream {
  func writePacketLine(message:String = "") -> Int {
    let data = message.utf8data
    let length = data.length
    if length == 0 {
      return writeData("0000".utf8data)
    } else {
      let prefix = (length + 4).toHex(4).utf8data
      return self.writeData(prefix) + self.writeData(data)
    }
  }
}
```

当传递一个空的 `NSData` 对象时，将写入特殊的包行 `0000`，表示对话结束。当写入非空的 `NSData` 时，数据长度以十六进制值（包括用于长度的 4 个字节）表示，然后是数据本身。

### 注意

这将导致以下协议对话：

```swift
> 004egit-upload-pack /alblue/com.packtpub.swift.essentials.git\0host=github.com\0
< 00dfadaa46b98ce211ff819f0bb343395ad6a2ec6ef1 HEAD\0multi_ack thin-pack side-band side-band-64k ofs-delta shallow no-progress include-tag multi_ack_detailed symref=HEAD:refs/heads/master agent=git/2:2.1.1+github-611-gd89bd9f
< 003fadaa46b98ce211ff819f0bb343395ad6a2ec6ef1 refs/heads/master
> 0000
< 0000
```

读取数据包行类似：

```swift
extension NSInputStream {
  func readPacketLine() -> NSData? {
    if let data = readData(4) {
      let length = data.utf8string.fromHex()
      if length == 0 {
        return nil
      } else {
        return readData(length - 4)
      }
    } else {
      return nil
    }
  }
  func readPacketLineString() -> NSString? {
    if let data = self.readPacketLine() {
      return data.utf8string
    } else {
      return nil
    }
  }
}
```

在这种情况下，读取前 4 个字节以确定剩余长度。如果它是零，则返回 `nil` 值以指示流结束。如果它不是零，则读取数据（减去用于数据包行长度头的 4 个字节）。还提供了一个 `readPacketLineString` 以便于轻松创建作为 `NSString` 的数据包行。

### 远程列出 git 引用

要远程查询 git 仓库的引用，需要发送 `git-upload-pack` 命令，并附带要查询的仓库的引用，以及可选的主机。为了提供一种程序化查询的 API，创建一个 `RemoteGitRepository` 类，该类具有一个初始化器，用于存储主机、端口和仓库，以及一个 `lsRemote` 函数，该函数返回引用的值：

```swift
class RemoteGitRepository {
  let host:String
  let repo:String
  let port:Int
  init(host:String, repo:String, _ port:Int = 9418) {
    self.host = host
    self.repo = repo
    self.port = port
  }
  func lsRemote() -> [String:String] {
    var refs = [String:String]()
    // load the data
    return refs
  }
}
```

要从仓库加载数据，需要在默认端口（在这种情况下，`9418` 是 `git://` 协议的默认端口）上连接到远程主机。一旦流被打开，发送 `git-upload-pack [repository]\0host=[host]\0` 数据包行，然后可以读取形式为 `hash reference` 的行。将以下内容添加到 `lsRemote` 函数中：

```swift
// load the data
if let (input,output) = NSStream.open(host,port) {
  output.writePacketLine(
   "git-upload-pack \(repo)\0host=\(host)\0")
  while true {
    if let response = input.readPacketLineString() {
      let hash = String(response.substringToIndex(41))
      let ref = String(response.substringFromIndex(41))
      if ref.hasPrefix("HEAD") {
        continue
      } else {
        refs[ref] = hash
      }
    } else {
      break
    }
  }
  output.writePacketLine()
  input.close()
  output.close()
}
```

在一个具有适当的 `host` 和 `repo` 的 `RemoteGitRepository` 实例上调用 `lsRemote` 函数将返回一个通过引用的哈希列表。

### 将网络调用集成到 UI 中

由于网络可能会引入延迟，甚至可能导致完全失败，因此网络调用不应在 UI 线程上执行。以前，`SampleTable` 被用来引入一个 `runOnUIThread` 函数。可以使用类似的方法在后台线程上运行函数。将以下内容添加到 `SampleTable` 类中：

```swift
func runOnBackgroundThread(fn:()->()) {
  dispatch_async(
   dispatch_get_global_queue(
    DISPATCH_QUEUE_PRIORITY_DEFAULT, 0)
   ,fn)
}
```

这将允许 `viewDidLoad` 调用一个调用，以从仓库查询远程引用并将其添加到表中。与之前一样，必须从 UI 线程调用更新表的调用。将以下内容添加到 `viewDidLoad` 方法的末尾：

```swift
runOnBackgroundThread {
  let repo = RemoteGitRepository(host: "github.com", 
   repo: "/alblue/com.packtpub.swift.essentials.git")
  for (ref,hash) in repo.lsRemote() {
    self.items += [(ref,hash)]
  }
  self.runOnUIThread(self.tableView.reloadData)
}
```

现在当应用程序启动时，应将远程仓库中对应的分支和标签条目添加到表中。

## 异步读写

除了同步的读写操作，还可以执行*异步*读写操作。应用程序不需要在`while`循环中旋转，而是可以使用在应用程序的运行循环上安排的回调。

要接收回调，必须创建一个实现了`NSStreamDelegate`的类并将其分配给流的`delegate`字段。当事件发生时，`stream`方法会根据事件类型和关联的流被调用。

流通过`scheduleInRunLoop`（使用`NSRunLoop.mainRunLoop()`和`NSDefaultRunLoopMode`模式）进行注册。最后，流可以被打开。

### 小贴士

如果在设置代理或调度到运行循环之前打开流，则不会传递事件。

事件在`NSStreamEvent`类中定义，包括`HasSpaceAvailable`（用于输出流）和`HasBytesAvailable`（用于输入流）。通过响应回调，应用程序可以异步处理结果。

### 小贴士

当使用 Swift 时，`NSStreamDelegate`在输入流或输出流上被视为一个`weak`代理。当使用内联类提供输入解析时，这会引发问题；这样做会导致`EXC_BAD_ACCESS`，因为代理会被运行时自动回收。这可以通过在初始化器中存储对`self`的强循环引用并在流关闭时将其赋值为`nil`来避免。

### 从 NSInputStream 异步读取数据

这对于异步协议特别有用，例如 XMPP，它可能在任意时间发送额外的消息。这也允许电池供电的设备在远程服务器慢或挂起时不需要旋转 CPU。

要异步接收数据，代理必须实现`NSStreamDelegate`方法`stream(stream:handleEvent)`。当数据可用时，会发送`HasBytesAvailable`事件，然后可以相应地读取数据。

为了将之前的示例转换为异步形式，需要做一些更改。首先，在*打开流连接*部分创建的`open`扩展方法需要增加一个`connect`方法，但不立即执行`open`：

```swift
class func open(host:String,_ port:Int)
 -> (NSInputStream, NSOutputStream)? {
  if let (input,output) = connect(host,port) {
    input.open()
    output.open()
    return (input,output)
  } else {
    return nil
  }
}
class func connect(host:String,_ port:Int)
  -> (NSInputStream, NSOutputStream)? {
    var input:NSInputStream?
    var output:NSOutputStream?
    NSStream.getStreamsToHostWithName(
      host, port: port, 
      inputStream: &input,
      outputStream: &output)
    guard let i = input, o = output else {
      return nil
    }
    return (i,o)
  } 
}
```

### 小贴士

为了异步接收事件，必须在流打开之前设置代理并将流调度到运行循环中。

### 创建流代理

要创建一个流代理，创建一个名为`PacketLineParser.swift`的文件，并包含以下内容：

```swift
class PacketLineParser: NSObject, NSStreamDelegate {
  let output:NSOutputStream
  let callback:(NSString)->()
  var capture:PacketLineParser?
  init(_ output:NSOutputStream, _ callback:(NSString) -> ()) {
    self.output = output
    self.callback = callback
    super.init()
    capture = self
  }
  func stream(stream: NSStream, handleEvent: NSStreamEvent) {
    let input = stream as! NSInputStream
    if handleEvent == NSStreamEvent.HasBytesAvailable {
      if let line = input.readPacketLineString() {
        callback(line)
      } else {
        output.writePacketLine()
        input.close()
        output.close()
        capture = nil
      }
    }
  }
}
```

此解析器有一个回调，对于读取的每个数据包行都会被调用；当发送`HasBytesAvailable`事件时，行会被读取（使用与之前相同的同步机制）然后传递给回调。与同步方法不同，这里没有`while`循环——当数据可用时，它触发数据的解析。

### 小贴士

由于这将分配给一个输入流代理（它持有弱引用），因此需要使用 `capture = self` 来捕获自身的循环引用，以避免运行时从实例中移除。当流关闭时，`capture` 将被设置为 `nil`，从而释放实例。

`readPacketLine` 返回 `nil` 以指示错误或完成的流；在这种情况下，发送一个空的数据包行（以告知远程服务器不需要进一步交互），然后关闭两个流。

### 处理错误

在流内容成功和发生通信错误时，都需要清理流并将它们从运行循环中移除。除了 `HasBytesAvailable` 事件外，还有在遇到流的末尾或发生错误时发送的事件。

这些应该以连接自然结束时相同的方式进行处理；资源应该被整理，特别是流应该从运行循环处理中移除。最后，应该移除循环引用，以便可以移除 `delegate` 对象。

可以将现有的 `close` 代码移动到它自己的单独函数中，并且当流结束或发生错误时，可以执行相同的清理操作：

```swift
func stream(stream: NSStream, handleEvent: NSStreamEvent) {
  let input = stream as! NSInputStream
  if handleEvent == NSStreamEvent.HasBytesAvailable {
    if let line = input.readPacketLineString() {
      callback(line)
    } else {
      closeStreams(input,output)
    }
  }
  if handleEvent == NSStreamEvent.EndEncountered 
 || handleEvent == NSStreamEvent.ErrorOccurred {
 closeStreams(input,output)
 }
}
func closeStreams(input:NSInputStream,_ output:NSOutputStream) {
  if capture != nil {
    capture = nil
    output.removeFromRunLoop(NSRunLoop.mainRunLoop(),
     forMode: NSDefaultRunLoopMode)
    input.removeFromRunLoop(NSRunLoop.mainRunLoop(),
     forMode: NSDefaultRunLoopMode)
    input.delegate = nil
    output.delegate = nil
    if output.streamStatus != NSStreamStatus.Closed {
      output.writePacketLine()
      output.close()
    }
    if input.streamStatus != NSStreamStatus.Closed {
      input.close()
    }
  }
}
```

### 异步列出引用

要异步提供引用列表，代理需要设置一个合适的回调来解析返回的数据。而不是方法返回一个字典（这将需要同步阻塞），将传递一个回调，可以在找到引用时调用。

### 注意

请注意，有两个独立的回调：`PacketLineParser` 回调（它读取网络数据，并在每个数据包行的基础上返回 `NSString` 实例），以及引用解析回调（它将 `NSString` 转换为 `(String,String)` 元组）。

要开始这个过程，需要在同步发送 `git-upload-pack` 之后异步处理后续响应。这可以通过在 `RemoteGitRepository` 类中创建一个新的方法 `lsRemoteAsync` 来实现，该方法接受一个用于 `(String,String)` 元组的回调函数：

```swift
func lsRemoteAsync(fn:(String,String) -> ()) {
  if let (input,output) = NSStream.connect(host,port) {
    input.delegate = PacketLineParser(output) {
    (response:NSString) -> () in
      let hash = String(response.substringToIndex(41))
      let ref = String(response.substringFromIndex(41))
      if !ref.hasPrefix("HEAD") {
        fn(ref,hash)
      }
    }
    input.scheduleInRunLoop(NSRunLoop.mainRunLoop(), 
     forMode: NSDefaultRunLoopMode)
    input.open()
    output.open()
    output.writePacketLine(
     "git-upload-pack \(repo)\0host=\(host)\0")
  }
}
```

这将创建一个连接（但不会打开流），设置 `delegate`，并为输入流安排运行循环，最后打开两个流以进行交互。完成这些操作后，就像之前一样发送初始的 `git-upload-pack` 消息。此时，`lsRemoteAsync` 方法返回，随后在从服务器接收到输入数据时发生后续事件。

当通过 `PacketLineParser` 回调接收到一行时，它将被分割成一个引用和一个哈希值，然后把结果交给最初在参数中传入的回调。

### 注意

异步编程通常涉及许多回调。与看起来像 `A;B;C;` 的同步程序不同，异步程序通常看起来像 `A(callback:B(callback:C))`。当发生输入触发——网络请求、用户交互或定时器触发时，可以通过这些嵌套回调发生一系列操作。

由于在 `while` 循环中阻塞会浪费 CPU 能量直到条件满足，因此出于电池性能的原因，通常更喜欢异步管道。

### 在 UI 中显示异步引用

要在屏幕上显示异步数据，必须修改回调以允许单个元素更新 GUI。

在 `SampleTable` 中，不要调用 `repo.lsRemote`（它执行同步查找），而应使用 `repo.lsRemoteAsync`。这需要一个回调，它可以用来更新表格数据并导致视图重新加载内容：

```swift
// for (ref,hash) in repo.lsRemote() {
//   self.items += [(ref,hash)]
// }
repo.lsRemoteAsync() { (ref:String,hash:String) in
  self.items += [(ref,hash)]
  self.runOnUIThread(self.tableView.reloadData)
}
```

现在当应用程序运行时，引用将被异步更新，UI 不会被缓慢或挂起的服务器阻塞。

### 将数据异步写入 NSOutputStream

除非需要大文件上传，否则异步发送不如异步读取有用。如果有大量数据，那么在单个 `write` 调用中不太可能同步写入。最好异步执行任何额外的写入。

要异步写入数据，需要将 `completed` 计数存储为函数外的变量。`write` 方法可以用来替换之前的 `while` 循环，通过在流方法的每次迭代中写入数据的一个片段。尽管在这个例子中不需要代码，代码可能看起来像这样：

```swift
…
self.data = data
// initial write to kick off subsequent events
completed = output.write(UnsafePointer(data.bytes), 
 maxLength: data.length
…
var completed:Int
var data:NSData?
func stream(stream: NSStream, handleEvent: NSStreamEvent) {
  let output = stream as! NSOutputStream
  if handleEvent == NSStreamEvent.HasSpaceAvailable 
   && data != nil {
    let size = data!.length
    completed += output.write(
     UnsafePointer(data!.bytes) + completed,
     maxLength: size – completed)
    if completed == size {
      completed = 0
      data = nil
    }
  }
}
```

异步数据始终以同步写入数据的调用开始。如果并非所有数据都被写入（换句话说，`completed < size`），则后续回调将在 `NSStreamDelegate` 上发生。然后可以使用与同步情况类似的技术来继续 `data` 值，但不使用 `while` 循环。而不是迭代阻塞以写入整个数据值，流调用将被多次调用（实际上替换了 `while` 循环的每次迭代）。在最后一次运行时，当 `completed == size` 时，数据被释放，完成计数器被重置。

### 注意

流回调被调用足够多次以写入所有数据。如果没有数据被写入，则事件不再被调用。只有当传递了额外的值时，才会写入新数据。在从不同线程写入数据时必须小心，因为数据值被处理为一个实例变量，覆盖它可能会导致数据丢失。读者被邀请将单个元素数据扩展到未完成数据元素的数组中，以便它们可以适当地排队。

# 摘要

本章介绍了在基于 Swift 的应用程序中处理网络数据时常用的技术，特别关注如何使用异步技术访问数据来最大化便携设备的电池使用。

由于大多数网络请求很可能会通过 HTTP(S)提供基于 JSON 或 XML 的表示，本章的第一节涵盖了使用`NSURLSession`和异步的`dataTask`操作从远程服务器拉取数据。然后第二和第三部分介绍了如何根据所需格式从 JSON 或 XML 中解析这些数据。

最后一节展示了如何直接建立网络连接来处理除 HTTP 之外的协议；作为一个例子，展示了如何执行远程`git`命令以找出远程 git 仓库中可用的引用。这以两种形式呈现：首先作为一个同步 API（以展示如何与流一起工作的技术，并解释 git 协议），然后转换为异步 API，这可以用来最小化 CPU 周期，从而降低电池使用，以便将来执行其他此类转换。

下一章将展示如何将本书中涵盖的所有想法整合到一个 iOS 应用程序中，以显示 GitHub 仓库。
