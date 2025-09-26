# 第十六章：使用 Swift 进行网络开发

我在大学期间选修了几门网络课程，至今仍记得在那些课程中搭建我的第一个 Novell NetWare 网络。看到并学习计算机如何在网络上进行通信，我感到无比着迷。然后，在 20 世纪 90 年代初，我购买了第一台调制解调器，并开始拨打我所在地区的公告板服务。这真的很令人兴奋，因为现在我可以连接到我所在城市的公告板服务。这使我能够从这些公告板服务下载和上传文件，我开始下载我能找到的所有关于计算机通信的内容。这导致了我早期在网络安全和管理领域的职业生涯。然后，当我的第一个女儿出生时，我决定我不想总是处于待命状态，所以我回到了最初让我接触计算机的事情，那就是编程。然而，我仍然非常享受网络和网络安全的领域。我在网络和编程方面的背景使我能够对两者都有独特的理解。今天构成网络的东西（互联网和 TCP/IP 网络）与我在大学时构成网络的东西完全不同，但好事是，我们的应用程序通过网络进行通信的方式已经变得更加标准化，这使得编写通过网络进行通信的应用程序变得更加容易。

在本章中，你将学习以下主题：

+   如何使用`NSURLSession` API 进行 HTTP GET 请求

+   如何使用`NSURLSession` API 进行 HTTP POST 请求

+   如何使用系统配置 API 检查我们的网络连接

+   如何使用`RSNetworking2`库轻松地将网络功能添加到你的应用程序中

# 网络开发是什么？

网络开发是编写代码，使我们的应用程序能够从远程服务或设备发送和接收数据。在本章的引言中，我提到了购买我的第一台调制解调器，并连接到我所在城市的公告板服务。这些公告板服务中的大多数使用单个调制解调器，这意味着在任何时候只有一个用户可以连接到它们。对于在互联网环境中长大的人来说，这些公告板看起来非常奇怪和过时；然而，在当时，它们是计算机共享信息的方式。当时，能够连接到镇上的计算机并上传/下载文件是非常令人惊叹的。然而，如今，我们与世界各地的服务和设备进行通信，却无需多想。

在我刚开始编写应用程序的时候，开发一个通过网络连接进行通信的应用程序是很罕见的，而且也很难找到有网络开发经验的开发者。在当今世界，几乎每个应用程序都有某种网络通信的要求。

在本章中，我们将向您展示如何连接到基于 **表示状态转移** （**REST**） 的网络服务。表示状态转移是万维网的软件架构风格。通常，这些服务通过 HTTP 使用与网页浏览器相同的 HTTP 动词（get、put、delete 和 post）进行通信。

在本章中，我们将使用苹果提供的基于 REST 的服务，允许开发者搜索 iTunes Store。在本章中，我们将使用苹果的服务进行几个示例。苹果对这个服务进行了很好的文档记录。文档可以在[`www.apple.com/itunes/affiliates/resources/documentation/itunes-store-web-service-search-api.html`](https://www.apple.com/itunes/affiliates/resources/documentation/itunes-store-web-service-search-api.html)找到。

如果您想对网络开发进行更详细的讨论，我建议您阅读我的第一本书，书名为 *iOS 和 OS X 网络编程食谱*，由 *Packt Publishing* 出版。在本章中，我们将专注于如何连接到基于标准的 REST 服务。

在我们查看如何连接到 REST 服务之前，让我们看看我们将要使用的苹果网络 API 中的类。这些类是苹果强大的 URL 加载系统的一部分。

# URL 会话类的概述

苹果的 URL 加载系统是一个框架，提供了一系列用于与 URL 交互的类。使用这些类一起，我们可以与使用标准互联网协议的服务进行通信。在本章中，我们将使用以下类来连接到并从 REST 服务检索信息：

+   `NSURLSession`: 这是主要的会话对象。它被编写为旧 `NSURLConnection` API 的替代品。

+   `NSURLSessionConfiguration`: 这用于配置 `NSURLSession` 对象的行为。

+   `NSURLSessionTask`: 这是一个处理从 URL 检索的数据的基类。苹果为 `NSURLSessionTask` 类提供了三个具体的子类。

+   `NSURL`: 这是一个表示要连接到的 URL 的对象。

+   `NSMutableURLRequest`: 这个类包含我们正在发出的请求的信息，并由 `NSURLSessionTask` 服务用于发出请求。

+   `NSHTTPURLResponse`: 这个类包含对我们请求的响应。

现在，让我们更深入地看看这些类中的每一个，以便我们对每个类的基本功能有一个基本的了解。

## NSURLSession

在 iOS 7 和 OS X 10.9 之前，当开发者想要从 URL 检索内容时，他们使用 `NSURLConnection` API。从 iOS 7 和 OS X 10.9 开始，`NSURLSession` 成为首选 API。`NSURLSession` API 可以被视为对较旧的 `NSURLConnection` API 的改进。

`NSURLSession` 对象提供了一个与各种协议（如 HTTP 和 HTTPS）交互的 API。会话对象，即 `NSURLSession` 的实例，管理这种交互。这些会话对象高度可配置，这使我们能够控制我们的请求方式以及我们如何处理返回的数据。

与大多数网络 API 一样，`NSURLSession` 是异步的。这意味着我们必须提供一种方式将服务的响应返回给需要它的代码。从会话返回结果的最流行方式是将完成处理程序块（闭包）传递给会话。然后，当服务成功响应或我们收到错误时，将调用此完成处理程序。本章中的所有示例都使用完成处理程序来处理从服务返回的数据。

## NSURLSessionConfiguration

`NSURLSessionConfiguration` 类定义了在使用 `NSURLSession` 对象连接到 URL 时使用的行为和策略。当使用 `NSURLSession` 对象时，我们通常首先创建一个 `NSURLSessionConfiguration` 实例，因为创建 `NSURLSession` 类的实例时需要这个类的实例。

`NSURLSessionConfiguration` 类定义了三种会话类型：

+   **默认会话配置**: 这种配置的行为类似于 `NSURLConnection` API

+   **临时会话配置**: 这种配置的行为类似于默认会话配置，不同之处在于它不会将任何内容缓存在磁盘上

+   **后台会话配置**: 这种会话允许在应用程序在后台运行时执行上传和下载操作

重要的一点是，在使用它来创建 `NSURLSession` 类的实例之前，我们应该确保适当地配置 `NSURLSessionConfiguration` 对象。当会话对象被创建时，它会创建我们提供的配置对象的副本。一旦创建会话对象，对配置对象所做的任何更改都会被会话忽略。如果我们需要更改配置，我们必须创建 `NSURLSession` 类的另一个实例。

## NSURLSessionTask

`NSURLSession` 服务使用 `NSURLSessionTask` 类的实例来调用我们正在连接的服务。`NSURLSessionTask` 类是一个基类，苹果提供了三个具体的子类供我们使用：

+   `NSURLSessionDataTask`: 这会将响应，以内存中的形式，直接作为一个或多个 `NSData` 对象返回给应用程序。这是我们通常最常使用的任务。

+   `NSURLSessionDownloadTask`: 这会将响应直接写入一个临时文件。

+   `NSURLSessionUploadTask`: 这用于制作需要请求体（如 POST 或 PUT 请求）的请求。

重要的一点是，任务不会向服务发送请求，直到我们调用 `resume()` 方法。

## 使用 NSURL 类

`NSURL`对象代表我们将要连接到的 URL。`NSURL`类不仅限于表示远程服务器的 URL，它也可以用来表示磁盘上的本地文件。在本章中，我们将专门使用`NSURL`类来表示我们连接到的远程服务的 URL。

## NSMutableURLRequest

`NSMutableURLRequest`类是`NSURLRequest`类的可变子类，它表示一个 URL 加载请求。我们使用`NSMutableRequest`类来封装我们的 URL 和请求属性。

重要的是要理解，`NSMutableURLRequest`类用于封装我们请求所需的信息，但它并不实际发出请求。为了发出请求，我们使用`NSURLSession`和`NSURLSessionTask`类的实例。

## NSURLHTTPResponse

`NSURLHTTPResponse`类是`NSURLResponse`类的子类，它封装了与 URL 请求响应相关的元数据。`NSURLHTTPResponse`类提供了访问与 HTTP 响应相关特定信息的方法。具体来说，这个类允许我们访问 HTTP 头字段和响应状态码。

我们在本节中简要介绍了多个类，它们如何实际结合在一起可能并不清楚；然而，一旦你看到本章稍后的一些示例，就会变得非常清晰。在我们进入示例之前，让我们快速了解一下我们将要连接到的服务类型。

# REST 网络服务

REST 已成为设备之间无状态通信最重要的技术之一。由于基于 REST 的服务轻量级和无状态的特性，随着更多设备连接到互联网，其重要性可能会继续增长。

REST 是一种用于设计网络应用程序的架构风格。REST 背后的理念是，我们不是使用复杂的机制，如 SOAP 或 CORBA 在设备之间进行通信，而是使用简单的 HTTP 请求进行通信。虽然从理论上讲，REST 不依赖于互联网协议，但它几乎总是使用它们来实现。因此，当我们访问 REST 服务时，我们几乎总是以与我们的网络浏览器与这些服务器交互相同的方式与 Web 服务器进行交互。

REST 网络服务使用 HTTP POST、GET、PUT 或 DELETE 方法。如果我们考虑一个标准的 CRUD（创建/读取/更新/删除）应用程序，我们会使用 POST 请求来创建或更新数据，使用 GET 请求来读取数据，使用 DELETE 请求来删除数据。

当我们在浏览器的地址栏中输入一个 URL 并按*Enter*键时，我们通常是在向服务器发送一个 GET 请求，并要求它发送与该 URL 相关的网页。当我们填写一个网页表单并点击提交按钮时，我们通常是在向服务器发送一个 POST 请求。然后我们在 POST 请求的主体中包含网页表单的参数。

现在，让我们看看如何使用苹果的网络 API 发起 HTTP GET 请求。

# 发起 HTTP GET 请求

在本例中，我们将向苹果的 iTunes 搜索 API 发起一个 GET 请求，以获取与搜索词 Jimmy Buffett 相关的项目列表。由于我们从服务中检索数据，根据 REST 标准，我们应该使用 GET 请求来检索数据。

虽然 REST 标准是使用 GET 请求从服务中检索数据，但没有任何阻止 Web 服务的开发者使用 GET 请求来创建或更新数据对象。不推荐以这种方式使用 GET 请求，但请记住，有些服务并不遵循 REST 标准。

以下代码向苹果的 iTunes 搜索 API 发起请求，并将结果打印到控制台：

```swift
public typealias dataFromURLCompletionClosure = (NSURLResponse!, NSData!) -> Void

public func sendGetRequest(handler: public func getConnect(
        handler: dataFromURLCompletionClosure) {

        let sessionConfiguration =
          NSURLSessionConfiguration.defaultSessionConfiguration();

        let urlString =
            "https://itunes.apple.com/search?term=jimmy+buffett"

        if let encodeString =
     urlString.stringByAddingPercentEncodingWithAllowedCharacters(
                NSCharacterSet.URLQueryAllowedCharacterSet()),
            url = NSURL(string: encodeString) {

            let request = NSMutableURLRequest(URL:url)
            request.HTTPMethod = "GET"
            let urlSession = NSURLSession(
                configuration:sessionConfiguration, delegate: nil,
 delegateQueue: nil)

            let sessionTask = 
urlSession.dataTaskWithRequest(request) {
                (data, response, error) in

                handler(response, data)
            }
            sessionTask.resume()
        }
    }
```

我们首先创建一个名为`DataFromURLCompletionClosure`的类型别名。`DataFromURLCompletionClosure`类型将用于本章的 GET 和 POST 示例。如果您不熟悉使用`typealias`对象来定义闭包类型，请参阅第十二章，*使用闭包*，以获取更多信息。

然后我们创建一个名为`sendGetRequest()`的函数，该函数将用于向苹果的 iTunes API 发起 GET 请求。这个函数接受一个名为 handler 的参数，它是一个符合`DataFromURLCompletionClosure`类型的闭包。handler 闭包将用于返回请求的结果。

在我们的`sendGetRequest()`方法中，我们首先使用`defaultSessionConfiguration()`方法创建一个`NSURLSessionConfiguration`类的实例，这会创建一个默认会话配置实例。如果我们需要，我们可以在创建后修改会话配置属性，但在这个例子中，我们想要的是默认配置。

在创建我们的会话配置之后，我们创建 URL 字符串。这是我们要连接到的服务的 URL。在 GET 请求中，我们将参数放在 URL 本身中。在这个特定的例子中，`https://itunes.apple.com/search`是 Web 服务的 URL。然后我们用问号（`?`）跟随 Web 服务 URL，这表示 URL 字符串的其余部分由 Web 服务的参数组成。

参数的形式是键/值对，这意味着每个参数都有一个键和一个值。在 URL 中，参数的键和值由等号（`=`）分隔。在我们的例子中，键是`term`，值是`jimmy+buffett`。接下来，我们通过`stringByAddingPercentEncodingWithAllowedCharacters()`方法运行我们刚刚创建的 URL 字符串，以确保我们的 URL 字符串已正确编码。我们使用`URLQueryAllowedCharacterSet`字符集与此方法一起使用，以确保我们有一个有效的 URL 字符串。

接下来，我们使用我们刚刚构建的 URL 字符串来创建一个名为 `url` 的 `NSURL` 实例。由于我们正在执行 GET 请求，这个 `NSURL` 实例将代表网络服务的位置和我们发送给它的参数。

我们使用我们刚刚创建的 `NSURL` 实例创建一个 `NSMutableURLRequest` 类的实例。我们使用 `NSMutableURLRequest` 类而不是 `NSURLRequest` 类，这样我们就可以设置我们请求所需的属性。在这个例子中，我们设置了 `HTTPMethod` 属性；然而，我们也可以设置其他属性，例如超时间隔或向我们的 HTTP 标头添加项。

现在，我们使用在 `sendGetRequest()` 函数开头创建的 `sessionConfiguration` 变量（`NSURLSessionConfiguration` 类的实例）来创建 `NSURLSession` 类的实例。`NSURLSession` 类提供了我们将用于连接到苹果的 iTunes 搜索 API 的 API。在这个例子中，我们使用 `NSURLSession` 实例的 `dataTaskWithRequest()` 方法来返回名为 `sessionTask` 的 `NSURLSessionDataTask` 实例。

`sessionTask` 实例是向 iTunes 搜索 API 发送请求的部分。当我们从服务收到响应时，我们使用处理程序回调来返回 `NSURLResponse` 对象和 `NSData` 对象。`NSURLResponse` 包含有关响应的信息，而 `NSData` 实例包含响应的主体。

最后，我们调用 `NSURLSessionDataTask` 实例的 `resume()` 方法来向网络服务发送请求。记住，正如我们之前提到的，一个 `NSURLSessionTask` 实例将不会向服务发送请求，直到我们调用 `resume()` 方法。

现在，让我们看看我们如何调用 `sendGetRequest()` 函数。首先，我们需要做的是创建一个闭包，该闭包将被传递给 `sendGetRequest()` 函数并在收到网络服务的响应时被调用。在这个例子中，我们将简单地打印响应到控制台。由于响应是 JSON 格式，我们可以使用在 第八章 中描述的 `NSJSONSerialization` 类来解析响应；然而，由于本章是关于网络编程，我们只需将响应打印到控制台。以下是代码：

```swift
var printResultsClosure: HttpConnect.DataFromURLCompletionClosure = {

  if let data = $1 {
    let sString = NSString(data: data, encoding: NSUTF8StringEncoding)
    print(sString)
  }
}
```

我们定义这个闭包，命名为 `printResultsClosure`，为 `DataFromURLCompletionClosure` 类型的实例。在闭包内部，我们解包第一个参数并将值设置为名为 `data` 的常量。如果第一个参数不是 nil，我们将数据常量转换为 `NSString` 类的实例，然后将其打印到控制台。

现在，让我们用以下代码调用 `sendGetRequest()` 方法：

```swift
let aConnect = HttpConnect()
aConnect.sendGetRequest(printResultsClosure)
```

此代码创建了一个`HttpConnect`类的实例，然后调用`sendGetRequest()`方法，将`printResultsClosure`闭包作为唯一参数传递。如果我们连接到互联网时运行此代码，我们将收到一个包含与 iTunes 上吉米·巴菲特相关项目的 JSON 响应。

现在我们已经看到了如何发起一个简单的 HTTP GET 请求，让我们看看如何向一个网络服务发起 HTTP POST 请求。

# 发起 HTTP POST 请求

由于苹果的 iTunes，API 使用 GET 请求来检索数据。在本节中，我们将使用免费的[`httpbin.org`](http://httpbin.org)服务向您展示如何发起 POST 请求。[`httpbin.org`](http://httpbin.org)提供的 POST 服务可以在[`httpbin.org/post`](http://httpbin.org/post)找到。此服务将回显它接收到的参数，以便我们可以验证我们的请求是否正确发起。

当我们发起 POST 请求时，我们通常有一些想要发送或提交给服务器的数据。这些数据以键/值对的形式存在。这些对之间由一个和号（`&`）符号分隔，每个键与其值之间由一个等号（`=`）分隔。作为一个例子，假设我们想要提交以下数据到我们的服务：

```swift
firstname: Jon
lastname: Hoffman
age: 47 years
```

POST 请求的正文将采用以下格式：

```swift
firstname=Jon&lastname=Hoffman&age=47
```

一旦我们有了合适格式的数据，我们就会使用`dataUsingEncoding()`方法，就像我们在对 GET 请求进行正确编码 POST 数据时所做的那样。

由于发送到服务器的数据是键/值格式，因此在发送到服务之前，最合适的方式是使用`Dictionary`对象来存储这些数据。考虑到这一点，我们需要创建一个方法，该方法将接受一个`Dictionary`对象并返回一个字符串对象，该对象可以用于 POST 请求。以下代码将实现这一点：

```swift
func dictionaryToQueryString(dict: [String : String]) -> String {
  var parts = [String]()
  for (key, value) in dict {
    let part: String = key + "=" + value
    parts.append(part);
  }
  return parts.joinWithSeparator("&")
}
```

此函数遍历`Dictionary`对象中的每个键/值对，并创建一个包含键和值（由等号`=`分隔）的`String`对象。然后我们使用`joinWithSeperator()`函数将数组中的每个项目连接起来，每个项目由指定的字符串分隔。在我们的例子中，我们想要用和号符号（`&`）分隔每个字符串。然后我们将这个新创建的字符串返回给调用它的代码。

现在，让我们创建我们的`sendPostRequest()`函数，该函数将发送 POST 请求到[`httpbin.org`](http://httpbin.org)的 POST 服务。我们将看到这个`sendPostRequest()`函数和我们在本章的*制作 HTTP GET 请求*部分中展示的`sendGetRequest()`函数之间有很多相似之处。让我们看一下以下代码：

```swift
public func sendPostRequest(handler: dataFromURLCompletionClosure) {

        let sessionConfiguration =
        NSURLSessionConfiguration.defaultSessionConfiguration()

        let urlString = "http://httpbin.org/post"
        if let encodeString =
            urlString.stringByAddingPercentEncodingWithAllowedCharacters(
                NSCharacterSet.URLQueryAllowedCharacterSet()),
            url = NSURL(string: encodeString) {

            let request = NSMutableURLRequest(URL:url)
            request.HTTPMethod = "POST"
            let params = dictionaryToQueryString(["One":"1 and 1", "Two":"2 and 2"])
            request.HTTPBody = params.dataUsingEncoding(
                NSUTF8StringEncoding, allowLossyConversion: true)

            let urlSession = NSURLSession(
                configuration:sessionConfiguration, delegate: nil, delegateQueue: nil)

            let sessionTask = urlSession.dataTaskWithRequest(request) {
                (data, response, error) in

                handler(response, data)
            }
            sessionTask.resume()
        }
    }
```

现在，让我们逐步分析这段代码。注意，我们正在使用与 `sendGetRequest()` 函数相同的类型别名，名为 `DataFromURLCompletionClosure`。如果你不熟悉使用 `typealias` 对象来定义闭包类型，请参阅 第十二章，*使用闭包*，以获取更多信息。

`sendPostRequest()` 函数接受一个名为 `handler` 的参数，它是一个符合 `DataFromURLCompletionClosure` 类型的闭包。当服务对我们的请求做出响应后，handler 闭包将被用来处理来自 [`httpbin.org`](http://httpbin.org) 服务的数据。

在我们的 `sendPostRequest()` 方法中，我们首先使用 `defaultSessionConfiguration()` 方法创建一个 `NSURLSessionConfiguration` 类的实例，该方法创建一个默认会话配置实例。我们可以在创建后会话配置属性，但在这个例子中，我们想要的是默认配置。

在我们创建了会话配置之后，我们创建我们的 URL 字符串。这是我们要连接到的服务的 URL。在这个例子中，URL 是 [`httpbin.org/post`](http://httpbin.org/post)。接下来，我们将我们刚刚创建的 URL 字符串通过 `stringByAddingPercentEncodingWithAllowedCharacters()` 方法运行，以确保我们的 URL 字符串被正确编码。我们使用 `URLQueryAllowedCharacterSet` 字符集与这个方法一起使用，以确保我们有一个有效的 URL 字符串。

接下来，我们使用我们刚刚构建的 URL 字符串来创建一个名为 `url` 的 `NSURL` 类实例。由于这是一个 POST 请求，这个 `NSURL` 实例将代表我们要连接到的网络服务的位置。

我们现在使用我们刚刚创建的 `NSURL` 实例来创建一个 `NSMutableURLRequest` 类的实例。我们使用 `NSMutableURLRequest` 类而不是 `NSURLRequest` 类，这样我们就可以设置我们请求所需的属性。在这个例子中，我们设置了 `HTTPMethod` 属性；然而，我们也可以设置其他属性，例如超时间隔或向我们的 HTTP 头中添加项。

现在，我们使用我们在本节开头展示的 `dictionaryToQueryString()` 函数来构建我们要发送到服务器的数据。我们使用 `dataUsingEncoding()` 函数确保我们的数据在发送到服务器之前被正确编码，最后，数据被添加到 `NSMutableURLRequest` 实例的 `HTTPBody` 属性中。

我们使用在函数开头创建的 `sessionConfiguration` 变量（`NSURLSessionConfiguration` 类的实例）来创建 `NSURLSession` 类的实例。`NSURLSession` 类提供了我们将用于连接到 [`httpbin.org`](http://httpbin.org) POST 服务的 API。在这个例子中，我们使用 `NSURLSession` 实例的 `dataTaskWithRequest()` 方法来返回一个名为 `sessionTask` 的 `NSURLSessionDataTask` 类的实例。

`sessionTask` 实例是用来向 [`httpbin.org`](http://httpbin.org) POST 服务发起请求的。当我们从服务收到响应时，我们使用处理程序回调来返回 `NSURLResponse` 对象和 `NSData` 对象。`NSURLResponse` 包含有关响应的信息，而 `NSData` 实例包含响应的主体。

最后，我们调用 `NSURLSessionDataTask` 实例的 `resume()` 方法来向网络服务发起请求。记住，正如我们之前提到的，`NSURLSessionTask` 类不会向服务发送请求，直到我们调用 `resume()` 方法。

我们可以像调用 `sendGetRequest()` 方法一样，以完全相同的方式调用 `sendPostRequest()` 方法。

在开发通过互联网与其他设备和服务通信的应用程序时，验证我们是否有一个网络连接是一个好的做法。在开发移动应用程序时，验证我们不是在使用移动连接（3G、4G 等等）传输大量数据也是一个好的做法。

让我们看看如何验证我们是否有一个网络连接以及我们有什么类型的连接。

# 检查网络连接

随着我们创建通过互联网与其他设备和服务通信的应用程序，我们最终将想要验证在发起网络调用之前我们已经建立了网络连接。当我们编写移动应用程序时，还需要考虑用户所拥有的网络连接类型。作为移动应用程序开发者，我们需要记住，我们的用户可能有一个限制每月可以发送/接收的数据量的移动数据计划。如果他们超过了这个限制，他们可能需要支付额外的费用。如果我们的应用程序发送大量数据，那么在用户处于蜂窝网络的情况下，在发送这些数据之前提醒用户可能是合适的。

下一个示例将展示我们如何验证我们是否有一个网络连接，它还告诉我们我们有什么类型的连接。我们将首先导入系统配置 API，并定义一个包含不同连接类型的枚举。我们将以这种方式导入系统配置 API：

```swift
import SystemConfiguration
```

我们创建一个 `ConnectionType` 枚举。这个枚举将被用作 `networkConnectionType()` 的返回类型：

```swift
public enum ConnectionType {
    case NONETWORK
    case MOBILE3GNETWORK
    case WIFINETWORK
}
```

现在，让我们看看检查网络连接类型的代码：

```swift
public func networkConnectionType(hostname: NSString) -> ConnectionType {

    let reachabilityRef = SCNetworkReachabilityCreateWithName(nil,hostname.UTF8String)

    var flags: SCNetworkReachabilityFlags = SCNetworkReachabilityFlags()
        SCNetworkReachabilityGetFlags(reachabilityRef!, &flags)

    let reachable: Bool = (flags.rawValue & SCNetworkReachabilityFlags.Reachable.rawValue) != 0
    let needsConnection: Bool = (flags.rawValue & SCNetworkReachabilityFlags.ConnectionRequired.rawValue) != 0
    if reachable && !needsConnection {
        // what type of connection is available
        let isCellularConnection = (flags.rawValue & SCNetworkReachabilityFlags.IsWWAN.rawValue) != 0
        if isCellularConnection {
            // cellular connection available
            return ConnectionType.MOBILE3GNETWORK 
        } else {
            // wifi connection available
            return ConnectionType.WIFINETWORK 
        }
    }
    return ConnectionType.NONETWORK // no connection at all
}
```

`networkConnectionType()`函数首先创建一个`SCNetworkReachability`引用。为了创建`SCNetworkRechabilityRef`引用，我们使用`SCNetworkReachabilityCreateWithName()`函数，该函数创建了一个指向提供的宿主的可达性引用。

在我们获取到`SCNetworkReachabilityRef`引用之后，我们需要从引用中检索`SCNetworkReachabilityFlags`枚举。这是通过`SCNetworkReachabilityGetFlags()`函数完成的。

一旦我们有了网络可达性标志，我们就可以开始测试我们的连接。我们使用位与（`&`）运算符来查看主机是否可达，以及在我们能够连接到主机之前是否需要建立连接（`needsConnection`）。如果可达标志为假（我们目前无法连接到主机），或者如果`needsConnection`为真（我们需要在连接之前建立连接），我们返回`NONETWORK`，这意味着主机当前不可达。

如果我们能够连接到主机，然后我们会检查是否有蜂窝连接，通过再次检查网络可达性标志。如果有蜂窝连接，我们返回`MOBILE3GNETWORK`，否则，我们假设我们有一个 Wi-Fi 连接，并返回`WIFINETWORK`。

### 注意

如果你正在编写连接到互联网上其他设备或服务的应用程序，我建议将此功能放入标准库中使用，因为你会经常需要检查网络连接性，以及你所拥有的连接类型。

现在我们已经看到了如何使用苹果的网络 API 连接到远程服务，我想展示一个你可以用在你的应用程序中的网络库。这个网络库使得连接到互联网上各种类型的服务变得非常简单和容易。这是一个我创建和维护的库，但我肯定会欢迎任何愿意为代码库做出贡献的人。这个库叫做**RSNetworking**。

# Swift 2 的 RSNetworking2

你可以在 GitHub 上找到`RSNetworking2`，链接为[`github.com/hoffmanjon/RSNetworking2`](https://github.com/hoffmanjon/RSNetworking2)

`RSNetworking2`库是一个完全用 Swift 编程语言编写的网络库。`RSNetworking2`是使用苹果强大的 URL 加载系统（[`developer.apple.com/library/mac/documentation/Cocoa/Conceptual/URLLoadingSystem/URLLoadingSystem.html`](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/URLLoadingSystem/URLLoadingSystem.html)）构建的，该系统具有我们在此章中较早使用的`NSURLSession`类。`RSNetworking2`的主要设计目标是使开发者能够轻松快速地将强大的异步网络请求添加到他们用 Swift 编写的应用程序中。

我们可以使用`RSNetworking2`的以下三种方式：

+   `RSURLRequest`：此 API 提供了一个非常简单且易于使用的接口，用于向服务发送单个 GET 请求。

+   `RSTransaction`和`RSTransactionRequest`：这些 API 提供了一种非常强大且灵活的方式来对服务发起 GET 和 POST 请求。此 API 还使得对服务发起多个请求变得非常容易。

+   `Extensions`：`RSNetworking2`为`UIImageView`和`UIButton`类提供了扩展，以便从 URL 动态加载图像，并在加载后将其插入到`UIImageView`或`UIButton`类中。

让我们更详细地查看这些 API，并提供一些使用示例。

## RSURLRequest

使用`RSURLRequest` API，我们可以向服务发起 GET 请求，我们只需要提供 URL 和我们希望发送给服务的参数。`RSURLRequest` API 公开了四个函数。这些函数如下：

+   `dataFromURL(url: NSURL, completionHandler handler: RSNetworking.dataFromURLCompletionClosure)`: 这个方法从 URL 中检索一个`NSData`对象。这是主要函数，并被其他三个函数用来在转换成所需格式之前检索`NSData`对象。

+   `stringFromURL(url: NSURL, completionHandler handler: RSNetworking.stringFromURLCompletionClosure)`: 这个方法从 URL 中检索一个`NSString`对象。此函数使用`dataFromURL()`函数来检索一个`NSData`对象，并将其转换为`NSString`对象。

+   `dictionaryFromJsonURL(url: NSURL, completionHandler handler: RSNetworking.dictionaryFromURLCompletionClosure)`: 这个方法从 URL 中检索一个`NSDictionary`对象。此函数使用`dataFromURL()`函数来检索一个`NSData`对象，并将其转换为`NSDictionary`对象。从 URL 返回的数据应该是 JSON 格式，以便此函数能正常工作。

+   `imageFromURL(url: NSURL, completionHandler handler: RSNetworking.imageFromURLCompletionClosure)`: 这个方法从 URL 中检索一个`UIImage`对象。此函数使用`dataFromURL()`函数来检索一个`NSData`对象，并将其转换为`UIImage`对象。

现在，让我们看看如何使用`RSURLRequest` API 的示例。在这个例子中，我们将向苹果的 iTunes 搜索 API 发起请求，就像我们在本章的*制作 HTTP GET 请求*部分所做的那样：

```swift
func rsURLRequestExample() {
  var client = RSURLRequest()

  if let testURL =     NSURL(string:"https://itunes.apple.com/search?term=jimmy+buffett&m edia=music") {

      client.dictionaryFromJsonURL(testURL, completionHandler: resultsHandler)   
  }
}
```

让我们逐步分析这段代码。我们首先创建一个`RSURLRequest`类的实例和一个`NSURL`类的实例。`NSURL`实例代表我们希望连接到的服务的 URL，由于我们正在发起 GET 请求，它还包含我们发送给服务的参数。如果我们回想一下之前的*制作 HTTP GET 请求*部分，当我们发起 HTTP GET 请求时，我们发送给服务的参数包含在 URL 本身中。

苹果的 iTunes 搜索 API 以 JSON 格式返回搜索结果。我们可以在 API 文档中看到这一点，也可以通过将搜索结果打印到控制台来验证；因此，我们将使用 `RSURLRequest` 类的 `dictionaryFromJsonURL()` 方法来向服务发出请求。如果我们想获取数据，我们也可以使用 `dataFromURL()` 或 `stringFromURL()` 方法，但此方法专门编写来处理从基于 REST 的网络服务返回的 JSON 数据。

`dictionaryFromJsonURL()` 方法将 `NSURLSession` 请求返回的数据转换为 `NSDictionary` 对象。我们在这里使用 `NSDictionary` 对象而不是 Swift 的 `Dictionary` 对象，因为网络服务可能返回多种类型（字符串、数组、数字等），并且如果我们回想起来，Swift 的 `Dictionary` 对象只能有一个键类型和一个值类型。

当我们调用 `dictionaryFromJsonURL()` 方法时，我们传递我们想要连接的 URL 以及一个完成处理程序，该处理程序将在服务信息返回并转换为 `NSDicationary` 对象后调用。

现在，让我们看看我们的完成处理程序：

```swift
var resultsHandler:RSURLRequestRSURLRequestRSURLRequestRSURLRequest.dictionaryFromURLCompletionClosure = {
  var response = $0
  var responseDictionary = $1
  var error = $2
  if error == nil {
    let res = "results"
    if let results = responseDictionary[res] as? NSArray {
      print(results[0])

    }
    else {
      print("Problem with data")
    }
  }
  else {
    //If there was an error, log it
    print("Error : \(error)")
  }
}
```

我们的完成处理程序是 `RSURLRequest.dictionaryFromURLCompletionClosure` 类型。这种类型与 `RSTransactionRequest.dictionaryFromRSTransactionCompletionClosure` 类型的定义方式相同，这使我们能够为 `RSURLRequests` 和 `RSTransactionRequest` 请求使用相同的闭包。

我们通过检索传递的三个参数并将它们分配给 `response`、`responseDictionary` 和 `error` 变量来开始完成处理程序。然后我们检查 `error` 变量以查看它是否为 `nil`。如果是 `nil`，则表示我们收到了有效的响应，可以检索 `NSDictionary` 对象的值。

在这个例子中，我们从服务返回的 `NSDictionary` 对象中检索与 `results` 键关联的 `NSArray` 值。这个 `NSArray` 值将包含与我们的搜索词关联的 iTunes 商店中的项目列表。一旦我们有了 `NSArray` 值，我们就将数组的第一个元素打印到控制台。

`RSURLRequest` API 非常适合向服务发出单个 GET 请求。现在，让我们看看 `RSTransaction` 和 `RSTransactionRequest` API，这些 API 可以用于 POST 和 GET 请求，并且在我们需要向同一服务发出多个请求时应使用。

## RSTransaction 和 RSTransactionRequest

`RSTransaction` 和 `RSTransactionRequest` 类允许我们配置一个事务（`RSTransaction`），然后使用该事务向服务发出请求（`RSTransactionRequest`）。使这个 API 如此强大的一个因素是，我们通过简单地更新事务并重新提交它来执行后续请求是多么容易。让我们看看这两个类公开的 API。

### RSTransaction

`RSTransaction` 类定义了我们希望进行的交易。它公开了四个属性和一个初始化器。

属性如下：

+   `TransactionType`: 这定义了 HTTP 请求方法。目前定义了三种类型——GET、POST 和 UNKNOWN。只有 GET 和 POST 实际上发送请求。

+   `baseURL`: 这是用于请求的基本 URL。这通常看起来像 [`itunes.apple.com`](https://itunes.apple.com)。如果我们使用非标准端口，我们会在服务器 URL 后面跟一个冒号和端口号，例如 `http://mytestserver:8080`。

+   `path`: 这是将被添加到基本 URL 的路径。这可能是 `search` 这样的内容。它也可以包括一个更长的路径字符串，例如 `path/to/my/service`。

+   `parameters`: 这是一个包含要发送到服务的参数的 `Dictionary` 对象。

初始化器如下：

+   `init(transactionType: RSTransactionType, baseURL: String, path: String, parameters: [String: String])`: 这将使用所有必需的属性初始化 `RSTransaction` 类。

### RSTransactionRequest

`RSTransactionRequest` 类构建并发送由以下四个函数定义的请求：

+   `dataFromRSTransaction(transaction: RSTransaction, completionHandler handler: RSNetworking.dataFromRSTransactionCompletionCompletionClosure)`: 这个函数从由 `RSTransaction` 类定义的服务中检索一个 `NSData` 对象。这是主函数，并被其他三个函数用来在将其转换为请求的格式之前检索 `NSData` 对象。

+   `stringFromRSTransaction(transaction: RSTransaction, completionHandler handler: RSNetworking.stringFromRSTransactionCompletionCompletionClosure)`: 这个函数从由 `RSTransaction` 类定义的服务中检索一个 `NSString` 对象。这个函数使用 `dataFromRSTransaction()` 函数来检索 `NSData` 对象，并将其转换为 `NSString` 对象。

+   `dictionaryFromRSTransaction(transaction: RSTransaction, completionHandler handler: RSNetworking.dictionaryFromRSTransactionCompletionCompletionClosure)`: 这个函数从由 `RSTransaction` 类定义的服务中检索一个 `NSDictionary` 对象。这个函数使用 `dataFromRSTransaction()` 函数来检索 `NSData` 对象，并将其转换为 `NSDictionary` 对象。从 URL 返回的数据应该是 JSON 格式，这样这个函数才能正常工作。

+   `imageFromRSTransaction(transaction: RSTransaction, completionHandler handler: RSNetworking.imageFromRSTransactionCompletionCompletionClosure)`: 这个函数从由 `RSTransaction` 类定义的服务中检索一个 `UIImage` 对象。这个函数使用 `dataFromRSTransaction()` 函数来检索 `NSData` 对象，并将其转换为 `UIImage` 对象。

现在，让我们看看如何使用 `RSTransaction` 和 `RSTransactionRequest` 类向苹果的 iTunes 搜索 API 发出 GET 请求的示例。在这个例子中，我们将使用本章 *RSURLRequest* 部分中定义的相同的 `resultsHandler` 闭包。让我们看一下以下代码：

```swift
func rsTransactionExample() {

  let rsRequest = RSTransactionRequest()

   //First request
  let rsTransGet = RSTransaction(transactionType: RSTransactionType.GET, baseURL: "https://itunes.apple.com", path: "search", parameters: ["term":"jimmy+buffett", "media":"music"])
  rsRequest.dictionaryFromRSTransaction(rsTransGet, completionHandler: resultsHandler)

  //Second request  
  rsTransGet.parameters = ["term":"jim", "media":"music"]
  rsRequest.dictionaryFromRSTransaction(rsTransGet, completionHandler: resultsHandler)
}
```

在这个例子中，我们首先创建一个名为 `rsRequest` 的 `RSTransactionRequest` 类实例。这个 `RSTransactionRequest` 实例将被用来向我们的 `RSTransaction` 实例中定义的服务发送请求。

在创建 `RSTransactionRequest` 实例后，我们使用 `RSTransaction` 初始化器创建一个名为 `rsTransGet` 的 `RSTransction` 类实例。在这个初始化器中，我们按以下方式定义以下属性：

+   `transactionType`: `transactionType` 被设置为 `RSTransactionType`.GET (这也可以是 `RSTransactionType`.POST 或 `RSTransactionType`. UNKNOWN)

+   `baseURL`: `baseURL` 被设置为 [`itunes.apple.com`](https://itunes.apple.com)

+   `path`: `path` 被设置为搜索

+   `parameters`: 参数被设置为 `["term":"jimmy+buffett","media":"music"]`

最后，我们使用 `RSTransactionRequest` 实例的 `dictionaryFromRSTransaction()` 方法。此方法接受两个参数；第一个是定义要发送的事务的 `RSTransaction` 实例，第二个是数据从服务返回后将被调用的完成处理程序。

正如我们之前提到的，使 `RSTransaction` 和 `RSTransactionRequest` 类易于使用的一个原因是，向同一服务进行后续请求非常简单。在我们的例子中，在发出初始请求后，我们更改参数并向同一服务发出第二个请求。需要注意的一点是，由于这些是异步请求，如果我们连续发出两个这样的请求，我们无法保证哪个请求会先返回。

现在，让我们看看 `RSNetworking2` 库的最后部分——扩展。

### 扩展

在 Swift 中，扩展向现有类添加新功能。`RSNetworking2` 为 `UIImageView` 和 `UIbutton` 类提供了扩展。这些扩展允许我们从 URL 加载图像，然后将其添加到 `UIImageView` 或 `UIButton` 中，一旦图像下载完成。我们还可以放置一个占位符图像，在最终图像下载之前将在 `UIImageView` 或 `UIButton` 中显示。一旦图像下载完成，占位符图像将被下载的图像替换。

`UIImageView` 和 `UIButton` 扩展都公开了四个新方法：

+   `setImageForURL(url: NSString, placeHolder: UIImage)`: 此方法将 `UIImageView` 或 `UIButton` 扩展的图像设置为占位符图像，然后从提供的 URL 异步下载图像。一旦图像下载完成，它将用下载的图像替换占位符图像。

+   `setImageForURL(url: NSString)`: 这个方法异步从 URL 下载图像。一旦图像下载完成，它将`UIImageView`或`UIButton`扩展的图像设置为下载的图像。

+   `setImageForRSTransaction(transaction:RSTransaction, placeHolder: UIImage)`: 这个方法将`UIImageView`或`UIButton`中的图像设置为占位符图像，然后从提供的`RSTransaction`对象异步下载图像。一旦图像下载完成，它将用下载的图像替换占位符图像。

+   `setImageForRSTransaction(transaction:RSTransaction)`: 这个方法异步从提供的`RSTransaction`对象下载图像。一旦图像下载完成，它将`UIImageView`或`UIButton`扩展的图像设置为下载的图像。

`UIButton`和`UIImageView`扩展的使用方式完全相同。为了了解如何使用这些扩展，让我们看看如何使用`UIImageView`扩展来查看从互联网下载的图像：

```swift
let url = "http://is4.mzstatic.com/image/pf/us/r30/Features/2a/b7/da/dj.kkir mfzh.100x100-75.jpg"
 if let iView: UIImageView = imageView, image = UIImage(named: "loading") {
   iView.setImageForURL(url, placeHolder: image)
}
```

在这个例子中，我们首先定义了我们图像的 URL。然后我们验证`imageView`变量是否包含`UIImageView`类的实例。请注意，我们通常不会在`if-let`语句中定义常量类型（`UIImageView`类型），但我在这个例子中定义了类型，以展示`imageView`常量应该是`UIImageView`类的实例。接下来，我们创建了一个名为`loading`的`UIImage`类的实例。这个图像将被用作占位符图像，在我们从 URL 下载最终图像的过程中显示。

现在我们已经拥有了图像的 URL 和占位符图像，我们使用`setImageForURL()`扩展方法。这个方法接受两个参数——下载图像的 URL 和占位符图像。一旦我们调用这个方法，`RSNetworking2`将`UIImageView`类的图像设置为提供的占位符图像，并从提供的 URL 下载图像。一旦图像下载完成，`RSNetworking2`将用下载的图像替换占位符图像。

在本章中，我们简要介绍了`RSNetworking2`的一些示例。更多示例可以在`RSNetworking2`的 GitHub 网站上找到：[`github.com/hoffmanjon/RSNetworking2`](https://github.com/hoffmanjon/RSNetworking2)。

# 摘要

在当今世界，一个开发者必须具备良好的网络开发知识。在本章中，我们看到了如何使用 Apple 的`NSURLSession` API，结合其他类，连接到基于 HTTP REST 的 Web 服务。`NSURLSession` API 被编写为旧版`NSURLConnection` API 的替代品，并且现在是进行网络请求时推荐使用的 API。

我们还看到了如何使用苹果的系统配置 API 来确定我们有什么类型的网络连接。如果我们正在为移动设备（iPhone、iPod 或 iPad）开发应用程序，了解我们是否有网络连接以及连接类型是至关重要的。

我们在本章中讨论了`RSNetworking2`，这是一个开源网络库，完全用 Swift 编写，由我维护。`RSNetworking2`使我们能够非常快速和容易地将网络功能添加到我们的应用程序中。它还向`UIImageView`和`UIButton`类添加了一个扩展，用于从互联网动态加载图像，并在下载完成后显示它们。我鼓励任何希望参与 RSNetworking 开发的人。
