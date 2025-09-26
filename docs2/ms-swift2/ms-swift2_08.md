# 第八章. 处理 XML 和 JSON 数据

多年来，我一直将**可扩展标记语言**（**XML**）作为首选的数据交换格式，用于系统之间的数据交换。它的简单性、可读性和易用性使其成为易选。在我看来，XML 的唯一真正缺点是 XML 文档的大小很大。移动设备，如 iOS 设备，在未连接到 Wi-Fi 网络时，依赖于通过移动网络交换数据。这些移动网络通常比标准的 Wi-Fi 或有线网络慢。大多数移动设备也有数据计划，限制了用户在一个月内可以使用的数据量。那时，我开始真正考虑使用**JavaScript 对象表示法**（**JSON**）在系统之间交换数据。现在，我几乎完全使用 JSON 来交换数据，尤其是在移动设备上。尽管对于移动开发来说，JSON 似乎正在成为首选格式，但 XML 仍然非常广泛地被使用，因为它通常比 JSON 更容易阅读和使用。对于开发者来说，了解这两种格式都是很有帮助的。

在本章中，我们将介绍：

+   解析 XML 文档

+   构建 XML 文档

+   解析 JSON 文档

+   构建 JSON 文档

# XML 和 JSON

并非很久以前，大多数基于消费者的应用程序都是自包含的，不需要与外部服务交换数据。然而，在智能手机和数据驱动应用程序的时代，现在开发不需要与外部服务交换数据的应用程序已经很少见了。这使得应用开发者了解如何以标准格式交换数据变得至关重要。

现在，API 设计者倾向于选择两种格式之一来交换数据——XML 或 JSON。多年来，已经推广了许多其他的数据交换格式，但 XML 和 JSON 无疑是当前的领导者。主要原因在于 XML 和 JSON 的开放性和互操作性是其他数据交换格式无法比拟的。很难找到一个不提供 XML 和/或 JSON 以交换数据的公共 Web API。

苹果公司提供了简单高效的 API 来处理 XML 和 JSON 数据。虽然有一些第三方库和框架在苹果 API 之上提供了一定的优势和劣势，但在本章中，我们将坚持使用苹果的 API。让我们看看我们将如何使用 Swift 解析 XML 和 JSON 文档，但首先让我们创建一些我们将用于本章中 XML 和 JSON 示例的常见文件。

# 常见文件

让我们先创建一个结构，该结构将用于定义我们 XML 和 JSON 文档中有效的标签。这些标签将是：

+   `books`: 这是包含所有其他元素的根元素

+   `book`: 此元素包含关于特定书籍的所有信息

+   `author`: 此元素包含作者的姓名

+   `publisher`: 此元素包含出版者的名称

+   `category`：此元素包含书籍的分类

+   `description`：此元素包含书籍的描述

+   `name`：这是 XML 示例中书籍元素的属性，也是 JSON 示例中的标准元素。此元素包含书籍的名称

`DocTags` 结构将定义包含这七个标签名称的七个静态属性。以下代码展示了如何定义这个结构：

```swift
struct DocTags {
  static let BOOKS_TAG = "books"
  static let BOOK_TAG = "book"

  static let AUTHOR_TAG = "author"
  static let PUBLISHER_TAG = "publisher"
  static let NAME_TAG = "name"
  static let CATEGORY_TAG = "category"
  static let DESCRIPTION_TAG = "description"
}
```

### 注意

而不是使用结构体，我们可以在枚举中定义我们的文档标签。我们使用哪一个实际上完全取决于个人喜好。

在 `DocTags` 结构体中定义的七个属性都是使用 `static` 和 `let` 关键字定义的。`static` 关键字将属性定义为 `static` 属性。一个 `static` 属性是不与结构体的任何给定实例相关联的，对 `static` 属性的任何更改都会反映在结构体的所有实例中。与 `instance` 属性相比，`static` 属性的优势在于我们不需要创建结构体的实例就可以使用它。

接下来，我们需要创建一个包含每本书信息的类。我们将把这个类命名为 `Book`：

```swift
class Book {
  var name = ""
  var author = ""
  var publisher = ""
  var category = ""
  var description = ""
}
```

### 注意

虽然 Apple 建议我们使用值类型（结构体和枚举）而不是引用类型（类），但在本章的示例中，我们更倾向于使用引用类型，这样我们就可以在构建过程中传递我们书籍类型的实际实例，而不是值。

如我们所见，`Book` 类包含五个属性。这些属性将包含关于每本书的信息。

当我们解析 XML 和 JSON 文档时，我们将能够检索文档的每个元素及其存储在元素中的值；因此，我们需要一种方法来获取这些信息并设置 `Book` 属性的值。考虑到这一点，让我们创建一个辅助函数，该函数将接受元素的名称及其关联的值作为参数。然后，我们将根据元素的名称设置适当的属性。让我们把这个函数命名为 `addValue` 并将其添加到我们的 `Book` 类中：

```swift
func addValue(tagName: String, withValue value: String) {
  switch tagName {
  case DocTags.NAME_TAG:
    self.name = value
  case DocTags.AUTHOR_TAG:
    self.author = value
  case DocTags.PUBLISHER_TAG:
    self.publisher = value
  case DocTags.CATEGORY_TAG:
    self.category = value
  case DocTags.DESCRIPTION_TAG:
    self.description = value
  default:
    break
  }
}
```

`addValue` 函数将使用 `switch` 语句将元素名称与 `DocTags` 结构体中定义的每个标签进行比较。如果找到匹配项，它将设置相应属性的值。如果没有找到匹配项，它将跳过该元素；通常，我们应该能够简单地忽略额外的标签。

### 注意

通常，当我们处理一个类时，如本例中的情况，解析器代码并不太糟糕。当你开始处理更复杂的 XML 和 JSON 文档时，可能需要多个类，拥有像 `addValue` 方法这样的辅助方法可以显著清理解析代码，并使其更容易阅读。这些辅助函数可以放在它们自己的类中，或者作为数据存储类的一部分（如前例所示），具体取决于对特定应用程序的最佳做法。通常，我更喜欢将辅助函数与数据存储类分开。

# XML 和 NSXMLParser 类

在 Swift 中解析 XML 文档时，我们将使用 Apple 的 `NSXMLParser` 类。虽然 `NSXMLParser` 有几种替代方案，每种方案都有其自身的优缺点，但我一直发现 `NSXMLParser` 简单易懂且易于使用。它还设计得与 Apple 的其他 API 保持一致，这意味着如果我们熟悉 Apple 的其他 API，`NSXMLParser` 将显得相当直观。

`NSXMLParser` 类是一个 **简单 XML API** (**SAX**) 解析器。SAX 解析器提供了一种按顺序解析 XML 文档的机制。与将整个文档读入内存然后进行解析的 **文档对象模型** (**DOM**) 解析器不同，SAX 解析器在解析过程中报告每个解析事件。这允许在解析过程中占用更小的内存空间。这也意味着我们需要编写代码来处理解析 XML 文档所需的每个解析事件。

`NSXMLParser` 类可以从 URL、`NSData` 对象或通过流解析 XML 文档。为了从各种来源解析 XML 文档，我们将使用适当的初始化器初始化 `NSXMLParser` 类：

+   `Init(contentsOfURL:)`：使用提供的 URL 引用的内容初始化 `NSXMLParser` 类

+   `Init(data:)`：使用 `NSData` 对象的内容初始化 `NSXMLParser` 类

+   `Init(stream:)`：使用提供的流的内容初始化 `NSXMLParser` 类

对于本章的 XML 示例，我们将使用 `init(data:)` 初始化器来解析 XML 文档的字符串表示形式。`NSData` 类旨在与二进制数据一起使用。初始化器可以很容易地替换为其他任何初始化器，以从 URL 或流中解析 XML 文档。XML 解析示例将被设计为解析以下 XML 文档：

```swift
<?xml version="1.0"?>
<books>
  <book name="iOS and OS X Network Development Cookbook">
    <author>Jon Hoffman</author>
    <publisher>PacktPub</publisher>
    <category>Programming</category>
    <description>Network development for iOS and OS X</description>
  </book>
  <book name="Mastering Swift">
    <author>Jon Hoffman</author>
    <publisher>PacktPub</publisher>
    <category>Programming</category>
    <description>Learning Swift</description>
  </book>
</books>
```

# 使用 NSXMLParserDelegate 协议

`NSXMLParserDelegate` 协议定义了几个可选方法，这些方法可以被 `NSXMLParser` 代理定义。当 `NSXMLParser` 解析 XML 文档时发生某些解析事件时，会调用这些方法。`NSXMLParserDelegate` 方法还可以定义几个可选方法，用于处理 **文档类型定义** (**DTD**) 标记。DTD 标记通过定义一系列有效元素和属性来定义 XML 文档的合法结构。

在本章中，我们将实现以下代理方法来处理 XML 示例：

+   `parserDidStartDocument(_:)`: 当解析器开始解析 XML 文档时，会调用此方法

+   `parserDidEndDocument(_:)`: 解析器成功解析整个 XML 文档后，会调用此方法

+   `parser(_: didStartElement: namespaceURI: qualifiedName: attributes:)`: 当解析器遇到元素的开始标签时，会调用此方法

+   `parser(_: didEndElement: namespaceURI: qualifiedName:)`: 当解析器遇到元素的结束标签时，会调用此方法

+   `parser(_:parseErrorOccurred:)`: 当解析器遇到一个关键错误且无法解析文档时，会调用此方法

+   `parser(_:foundCharacters:)`: 当解析器需要提供当前元素的所有或部分字符的数据的字符串表示时，会调用此方法

让我们看看我们将如何使用 `NSXMLParser` 和 `NSXMLParserDelegate` 来解析 XML 文档。

# 解析 XML 文档

要解析一个 XML 文档，我们首先创建一个符合 `NSXMLParseDelegate` 协议的类或结构体。在我们的例子中，我们将这个类命名为 `MyXMLParser`。我们的 `MyXMLParser` 类定义将看起来像这样：

```swift
class MyXMLParser: NSObject, NSXMLParserDelegate {

  }
```

在 `MyXMLParser` 类内部，我们将添加三个属性，这些属性将在解析文档时被解析器使用。这三个属性是：

+   `books`: 这个属性将是一个可选的数组，包含 XML 文档中定义的书籍列表

+   `book`: 这将是一个可选的 `Book` 类实例，代表 XML 文档中正在解析的当前书籍

+   `elementData`: 这将是一个字符串类的实例，包含正在解析的当前元素的值

这些属性将定义如下：

```swift
var books: [Book]?
var book: Book?
var elementData = ""
```

现在我们需要添加 `NSXMLParserDelegate` 方法。我们首先添加的是 `parseXmlString` 方法，它将用于启动 `NSXMLParser` 类：

```swift
func parseXmlString(xmlString: String) {
   let xmlData = xmlString.dataUsingEncoding(NSUTF8StringEncoding)
   let parser = NSXMLParser(data: xmlData!)
   parser.delegate = self
   parser.parse()
}
```

我们从将 `xmlString` 变量转换为 `NSData` 对象开始 `parseXmlString()` 方法，使用 `dataUsingEncoding()` 方法。`dataUsingEncoding()` 方法来自 `NSString` 类，但我们可以使用它与我们的 Swift 字符串类型，因为 Swift 自动将 Swift 字符串类型桥接到 `NSString` 类。

我们然后使用 `init(data:)` 初始化器来初始化 `NSXMLParser`。这个初始化器调用如下：

`NSXMLParser(data: xmlData!)`

然后，我们将 `NSXMLParser` 代理设置为当前 `MyXmlParser` 类的实例。我们可以这样做，因为 `MyXmlParser` 类符合 `NSXMLParserDelegate` 协议。这允许类的当前实例在文档解析时接收警报。

最后，调用 `parse()` 方法开始解析 XML 文档。

现在，让我们添加 `parserDidStartDocument()` 方法。当 `NSXMLParser` 开始解析 XML 文档时，会调用此方法：

```swift
func parserDidStartDocument(parser: NSXMLParser!) {
  println("Started XML parser")
}
```

在我们的例子中，在解析文档之前我们不需要进行任何设置；因此，`parserDidStartDocument()` 方法只是将 `Started XML Parser` 消息打印到控制台。

现在，让我们看看 `parser(_: didStartElement: namespaceURI: qualifiedName: attributes:)` 代理方法。在我们实现这个代理方法之前，我们需要弄清楚哪些元素在我们遇到它们的开始标签时需要我们执行任务。在我们的例子中，我们需要检查两个元素的开始标签——`books` 和 `book`。

`books` 元素是包含 XML 文档中所有项目的根元素。当我们遇到 `books` 元素的开始标签时，我们需要初始化 `books` 数组。这个 `books` 数组将包含在解析 XML 文档时生成的书籍实例列表。

当我们遇到 `book` 元素的开始标签时，我们需要创建 `Book` 类的新实例，因为开始标签意味着我们开始了一本新书。我们可能会认为在创建新实例之前，我们应该也将当前 `book` 属性的实例保存到 `books` 数组中，但我们将这样做是在遇到 `book` 结束标签而不是开始标签时。基于结束标签而不是开始标签保存信息时，实现总是看起来更干净。

下面是 `parser(_: didStartElement: namespaceURI: qualifiedName: attributes:)` 代理方法的代码：

```swift
func parser(parser: NSXMLParser, didStartElement elementName: String, namespaceURI: String?, qualifiedName qName: String?, attributes attributeDict: [String : String]) {
    if elementName == DocTags.BOOKS_TAG {
        books = []
    } else if elementName == DocTags.BOOK_TAG {
        book = Book()
        if let name = attributeDict[DocTags.NAME_TAG] {
            book!.addValue(DocTags.NAME_TAG, withValue: name as!)
        }

    }
}
```

我们从这个方法开始，查看 `elementName` 参数是否等于 `books` 标签，如果是，我们创建一个新数组，该数组将包含 XML 文档中定义的书籍。如果 `elementName` 参数不等于 `books` 标签，我们检查它是否等于 `Book` 标签。如果它等于 `book` 标签，我们将 `book` 属性设置为 `book` 类的新实例，清除任何之前保存的信息。然后我们检查元素是否有具有 `name` 键（`name`）的属性，如果有，我们将 `book` 实例的 `name` 属性设置为该属性的值。这个 `Book` 类的实例将包含关于书籍的信息。

我们将要实现的下一个代理方法是 `parser(_:foundCharacters:)` 代理方法。此方法接收正在解析的元素的值或部分值。

```swift
func parser(parser: NSXMLParser, foundCharacters string: String) {
  elementData += string
}
```

由于任何给定元素的值可能相当大，我们可能以片段的形式接收该值而不是一次性接收。这意味着 `parser(_:foundCharacters:)` 方法可能多次为同一元素调用。在我们的例子中，我们使用 `elementData` 属性来跟踪当前元素的值；因此，在 `parser(_:foundCharacters:)` 方法中，我们只需将字符串参数的值追加到 `elementData` 属性。当我们遇到元素的结束标签时，我们将清除 `elementData` 属性。

接下来，让我们看看当解析器遇到一个元素的结束标签时，如何使用`parser(_: didEndElement: namespaceURI: qualifiedName:)`代理方法。在我们实现这个方法之前，我们需要弄清楚在遇到这些元素的结束标签时，我们需要哪些元素来执行任务。在我们的例子中，我们需要检查是否遇到了`book`元素的结束标签。如果我们遇到了除了`book`元素结束标签之外的任何标签，我们将使用`book`实例的`addValue()`方法来确定如何处理这个值。

当我们遇到`book`元素的结束标签时，我们需要将当前`book`属性的实例添加到`books`数组中。如果这是一个其他元素的结束标签，我们将调用当前书实例的`addValue()`方法，记住`addValue()`方法将忽略任何它不认识的元素名称。

每当我们遇到一个元素的结束时，我们还需要清除`elementData`属性，以确保前一个元素的信息不会破坏下一个元素。

以下示例展示了我们将如何实现`parser(_: didEndElement: namespaceURI: qualifiedName:)`代理方法：

```swift
func parser(parser: NSXMLParser, didEndElement elementName: String, namespaceURI: String?, qualifiedName qName: String?) {
    if elementName == DocTags.BOOK_TAG {
        if let myBook = book {
            if var _ = books {
                books!.append(myBook)
            }
        }
        book = Book()
    } else if let myBook = book {
        myBook.addValue(elementName, withValue: elementData)
    }
    elementData = ""
}
```

当解析器完成对文档的解析后，它将调用`parser(_:parseErrorOccurred:)`代理方法。在我们的例子中，我们将使用这个方法简单地打印出`books`数组中每本书的名称和作者到屏幕上，如下面的代码所示：

```swift
func parserDidEndDocument(parser: NSXMLParser) {
  if let myBooks = books {
    for myBook in myBooks {
      println("Found - \(myBook.name) \(myBook.author)")
    }
  }
}
```

如果在解析文档时遇到错误，将调用`parser:parseErrorOccurred:`代理方法来处理错误。在我们的例子中，我们将打印错误到控制台，但通常我们需要正确处理错误：

```swift
func parser(parser: NSXMLParser parseErrorOccurred parseError: NSError {
  print"Parse Error occurred (parseError)")
}
```

要解析一个 XML 文档，我们将使用`MyXmlParser`类，如下所示：

```swift
var xmlParser = MyXMLParser()
xmlParser.parseXmlString(xmlString)
```

现在我们已经看到了如何解析 XML 文档，让我们看看我们如何创建一个 XML 文档。

## XML 和 NSXMLDocument

`NSXMLDocument`类及其相关类使得从我们的自定义对象创建 XML 文档变得非常容易；然而，在撰写本书时，它们仅适用于基于 OS X 的项目。希望苹果公司中有人会意识到我们需要一种好的方法来为 iOS 项目构建 XML 文档，而无需使用第三方框架或手动创建文档。

要为基于 OS X 的应用程序使用 Swift 构建 XML 文档，我们需要使用三个基础类。这些类如下：

+   `NSNode`：这个类是`NSXMLDocument`和`NSXMLElement`类的超类。这将用于向`NSXMLElement`类的实例添加属性。

+   `NSXMLDocument`：这个类是 XML 文档的最高级对象。

+   `NSXMLElement`：XML 文档中的所有元素都是`NSXMLElement`类的实例。

让我们看看如何使用这三个类来构建一个 XML 文档。为此，我们将创建一个名为`buildXMLString(books:)`的函数，它接受一个`Book`对象数组作为其唯一参数：

```swift
func buildXMLString(books: [Book]?)] -> String {
  if let myBooks = books {
    let xmlRoot = NSXMLElement(name: DocTags.BOOKS_TAG)
    let xmlData = NSXMLDocument(rootElement: xmlRoot)
    for book in myBooks {
      let bookElement = NSXMLElement(name: DocTags.BOOK_TAG)
      xmlRoot.addChild(bookElement)
      let nameAttribute = NSXMLNode.attributeWithName(DocTags.NAME_TAG, stringValue:book.name) as NSXMLNode
      bookElement.addAttribute(nameAttribute)
      bookElement.addChild(NSXMLElement(name: DocTags.AUTHOR_TAG, stringValue: book.author))
      bookElement.addChild(NSXMLElement(name: DocTags.CATEGORY_TAG, stringValue: book.category))
      bookElement.addChild(NSXMLElement(name: DocTags.DESCRIPTION_TAG, stringValue: book.description))
      bookElement.addChild(NSXMLElement(name: DocTags.PUBLISHER_TAG, stringValue: book.publisher))
    }
    return xmlData.XMLString
  }
  else {
    return ""
  }
}
```

由于`books`参数被定义为可选的，我们在`buildXMLString()`函数的开始处使用`if let myBooks = books`行来验证它是否不为空。如果它是空的，我们返回一个空字符串；否则，我们开始构建 XML 文档。

我们正在使用的`NSXMLDocument`类的初始化器需要 XML 文档的根元素；因此，我们将首先使用`BOOKS_TAG`常量创建`xmlRoot`常量，然后使用它来创建`NSXMLDocument`类的一个实例。接下来，我们将遍历`books`数组中的每个`Book`类实例。

对于`Book`类的每个实例，我们创建一个新的`NSXMLElement`类实例，包含书名，这将包含关于书籍的信息。这个元素将是我们的 XML 文档中的`<Book></Book>`元素。关于书籍的所有信息要么是这个元素的属性，要么是这个元素的子元素。

书名是`book`元素的属性；因此，我们需要创建一个包含属性名称和值的`NSXMLNode`类实例。我们使用`NXMLNode.attributeWithName()`函数来完成此操作。然后我们使用`addAttribute()`函数将该属性添加到`book`元素中。

接下来，我们使用`addChild()`函数将书籍的其余信息（`author`、`category`、`description`和`publisher`）作为子节点添加到`book`元素中。

最后，我们使用`XMLString`属性将`NSXMLDocument`类转换为字符串，并将该字符串返回给调用该函数的代码。

# XML 和手动构建 XML 文档

由于我们无法在 iOS 项目中使用`NSXMLNode`、`NSXMLDocument`和`NSXMLElement`类，我们通常需要手动构建 XML 字符串或使用第三方库。这种方法容易出错，并且需要我们非常了解 XML 文档是如何构建的，但如果我们小心，我们可以通过这种方式创建简单的 XML 文档。

让我们看看如何手动创建一个 XML 文档。为此，我们将创建一个名为`builXMLString()`的函数，它接受一个`Book`对象数组作为其唯一参数。我们还将创建一个名为`getElementString()`的辅助类，该类将创建一个 XML 元素的字符串表示形式。`getElementString()`函数将接受两个元素：元素名称和值。让我们看看以下代码：

```swift
func buildXMLString(books: [Book]?) -> String {
  var xmlString = ""
  if let myBooks = books {
    xmlString = "<\(DocTags.BOOKS_TAG)>"
    for book in myBooks {
      xmlString += "<\(DocTags.BOOK_TAG) \(DocTags.NAME_TAG)=\"\(book.name)\">"
      xmlString += getElementString(DocTags.AUTHOR_TAG, elementValue: book.author)
      xmlString += getElementString(DocTags.CATEGORY_TAG, elementValue: book.category)
      xmlString += getElementString(DocTags.DESCRIPTION_TAG, elementValue: book.description)
      xmlString += getElementString(DocTags.PUBLISHER_TAG, elementValue: book.publisher)
      xmlString += "<\\\(DocTags.BOOK_TAG)>"
    }
    xmlString += "<\\\(DocTags.BOOKS_TAG)>"
  }
  return xmlString
}
func getElementString(elementName: String, elementValue: String) ->String {
  return "<\(elementName)>\"\(elementValue)\"<\\\(elementName)>"
}
```

由于`books`参数被定义为可选的，我们在`buildXMLString()`函数的开始处使用`if let myBooks = books`行来验证它是否不为空。如果它是空的，函数将返回一个空字符串；否则，我们开始构建 XML 文档。

在这个类中，我们只是创建表示 XML 标签的字符串，并将它们附加到`xmlString`变量。在函数结束时，`xmlString`变量将包含 XML 文档。

`getElementString()`函数创建一个包含元素开始标签、元素值和元素结束标签的字符串。此函数用于添加本例中的大多数 XML 元素。

如我们所见，如果没有对 XML 文档语法的深入了解，几乎不可能使用这种方法构建复杂的文档。我们还需要非常小心，不要忘记在元素末尾的结束标签。

# JSON 和 NSJSONSerialization

为了序列化和反序列化 JSON 文档，我们将使用`NSJSONSerialization`类。正如我们将看到的，使用`NSJSONSerialization`类与 JSON 文档相比，使用`NSXMLParser`类与 XML 文档相比要容易得多；然而，当我们尝试访问信息时，它可能会更容易出错。只需记住，在访问之前始终检查定义为可选的值是否为 null。

与`NSXMLParser`类不同，`NSJSONSerialization`类将解析整个 JSON 文档内存，然后返回一个 JSON 对象；因此，需要编写的代码要少得多，但它的内存消耗也更大。

`NSJSONSerialization`类可以从`NSData`对象或通过流解析 JSON 文档。为了从各种来源解析 JSON 文档，我们使用`NSJSONSerialization`类和适当的静态方法：

+   `JSONObjectWithData(_: options: error:)`: 此初始化器将解析存储为`NSData`对象的 JSON 文档

+   `JSONObjectWithStream(_: options: error:)`: 此初始化器将从流中解析 JSON 文档

这两个方法的文档说明它们返回一个`AnyObject`类型的可选值。通常，这些方法的返回结果是在`NSDictionary`或`NSArray`类的实例中，具体取决于 JSON 文档。如果您不确定正在创建的对象类型，可以插入以下代码，其中`jsonResponse`变量是从两个静态方法返回的结果：

```swift
switch jsonResponse {
case is NSDictionary:
  // Code to parse a NSDictionary
case is NSArray:
  // Code to parse an NSArray
default:
  // Code to handle unknown type
}
```

在前面的代码中，我们使用`is`运算符来检查响应是否为`NSDictionary`或`NSArray`类型。

与`NSXMLParser`类不同，`NSJSONSerialization`类还可以用于从集合对象创建 JSON 文档。为此，我们将使用`dataWithJSONObject(_: options: error:)`初始化器，它将从集合对象序列化 JSON 文档。虽然可以使用除集合对象以外的其他对象来创建 JSON 文档，但一个合适的 JSON 文档通常是以字典或数组的格式。

在本章的 JSON 示例中，我们将向您展示如何解析以下 JSON 文档。该文档包含与 XML 示例中相同的信息，但现在它存储为一个 JSON 文档，如下面的代码所示：

```swift
{
  "books": [
    {
      "name": "iOS and OS X Network Development Cookbook",
      "author": "Jon Hoffman",
      "publisher": "PacktPub",
      "category": "Programming",
      "description": "Network development for iOS and OS X"
    },
    {
      "name": "Mastering Swift",
      "author": "Jon Hoffman",
      "publisher": "PacktPub",
      "category": "Programming",
      "description": "Learning Swift"
    }
  ]
}
```

让我们看看如何解析存储为字符串的 JSON 文档。

## 解析 JSON 文档

在本节中，我们将使用 `NSJSONSerialization` 类来解析之前显示的 JSON 文档。此函数中的 `jsonString` 变量代表之前显示的 JSON 文档。此函数将根据 JSON 文档中的信息创建一个 `Book` 对象数组。函数结束时，我们将打印出有关书籍的信息，以显示它们已正确解析自文档，以及返回书籍数组，如下面的代码所示：

```swift
func parseJson() throws {
    var myBooks: [Book] = []
    let jsonData = jsonString.dataUsingEncoding(NSUTF8StringEncoding)
    if let data = jsonData {

        let jsonDoc : AnyObject = try NSJSONSerialization.JSONObjectWithData(data, options: NSJSONReadingOptions.AllowFragments)

        if let books = jsonDoc.objectForKey(DocTags.BOOKS_TAG) as? NSArray {           
            for var i=0; i < books.count; i++ {
                if let dict = books.objectAtIndex(i) as? NSDictionary {
                    let book = Book()
                    addValueToBook(book, elementName: DocTags.AUTHOR_TAG, elementValue: (dict.objectForKey(DocTags.AUTHOR_TAG) as? String))
                    addValueToBook(book, elementName: DocTags.CATEGORY_TAG, elementValue: (dict.objectForKey(DocTags.CATEGORY_TAG) as? String))
                    addValueToBook(book, elementName: DocTags.DESCRIPTION_TAG, elementValue: (dict.objectForKey(DocTags.DESCRIPTION_TAG) as? String))
                    addValueToBook(book, elementName: DocTags.NAME_TAG, elementValue: (dict.objectForKey(DocTags.NAME_TAG) as? String))
                    addValueToBook(book, elementName: DocTags.PUBLISHER_TAG, elementValue: (dict.objectForKey(DocTags.PUBLISHER_TAG) as? String))
                    myBooks.append(book)
                }
            }

            for book in myBooks {
                print("Found - \(book.name) \(book.author)")
            }
        }
    }
}
```

`parseJson()` 函数首先将包含 JSON 文档的 `jsonString` 变量转换为 `NSData` 对象，以便我们可以使用 `NSJSONSerialization` 对象来解析它。由于转换结果是一个可选值，我们需要验证它是否不为空。我们使用以下代码来完成此操作：

```swift
if let data = jsonData
```

如果转换成功，我们就可以使用 `NSJSONSerialization` 类的 `JSONObjectWithData()` 方法从我们刚刚创建的 `NSData` 对象中创建 JSON 对象。

知道在我们的 JSON 文档中，`books` 的根标签包含一个书籍数组，我们使用以下行尝试从我们刚刚创建的 JSON 对象中检索数组：

```swift
if let books = jsonDoc.objectForKey(DocTags.BOOKS_TAG) as? NSArray
```

这行代码检查对象是否不为空，并且也是 `NSArray` 类的实例。如果 JSON 对象应该包含一个 `NSDictionary` 对象，我们只需将 `as? NSArray` 替换为 `as? NSDictionary`。

如果我们能够成功从 JSON 对象中检索到 `NSArray` 类，那么我们将遍历 `NSArray` 类中的每个项目。在我们的例子中，`NSArray` 类的每个项目都是一个 `NSDictionary` 类的实例；然而，始终验证这一点是个好主意。为了验证每个项目是 `NSDictionary` 类的实例，我们使用以下代码：

```swift
if let dict = books[i] as? NSDictionary 
```

一旦我们有了 `NSDictionary` 对象，我们就使用 `addValueToBook()` 函数（我们将在下一分钟看到）来填充 `Book` 类的属性。

最后，我们通过打印出从 JSON 文档中提取的每本书的名称和作者来结束函数。让我们看看我们用来填充 `Book` 类属性的 `addValueToBook()` 函数：

```swift
func addValueToBook(book: Book, elementName: String, elementValue: String?) {
    if let value = elementValue {
      book.addValue(elementName, withValue: value)
    }
}
```

如果我们尝试从一个键不存在的 `NSDictionary` 对象中提取值，`NSDictionary` 对象将返回一个空对象。在这种情况下，我们需要在将其分配给不接受空值的属性之前验证该值是否不为空。`addValueToBook()` 函数在将值添加到 `Book` 类的实例之前验证这些值是否不为空。

### 注意

在使用 `NSJSONSerialization` 类时，最好进行过多的检查而不是不足。只需记住，如果我们尝试将非可选变量设置为 nil 或使用 nil 的对象，我们的应用程序将会崩溃。JSON 文档不是类型安全的；因此，建议检查返回值的类型，以确保它们是预期的类型。

现在，让我们看看如何使用 `NSJSONSerialization` 类创建 JSON 文档。

## 创建 JSON 文档

使用 `NSJSONSerialization` 类创建 JSON 文档非常简单，但同样，我们需要进行几个检查以确保没有出错。以下代码将从任何可以转换为 JSON 数据的对象（如字典和/或数组）创建有效的 JSON 文档：

```swift
func buildJSON(value: AnyObject) throws -> String {
    if NSJSONSerialization.isValidJSONObject(value) {
        let data = try NSJSONSerialization.dataWithJSONObject(value, options: [])
        if let string = NSString(data: data, encoding: NSUTF8StringEncoding) {
           return string as String
        }
    }
    return ""
}
```

在 `buildJSON()` 函数中，我们首先验证 `value` 参数是否为可以转换为 JSON 对象的类型。我们使用 `NSJSONSerialization` 类的 `isValidJSONObject()` 函数来完成此操作。如果 `value` 参数可以转换，则该函数将返回布尔值 `true`；否则，它将返回布尔值 `false`。

如果 `value` 参数可以转换为 JSON 对象，那么我们使用 `NSJSONSerialization` 类的 `dataWithJSONObject()` 函数将 `value` 参数转换为 JSON 数据。如果转换过程中出现问题，`dataWithJSONObject()` 函数将抛出一个错误，该错误将被抛回调用 `buildJSON()` 函数的代码。

最后，我们将 JSON 数据转换为 `String` 对象，并将其返回给调用该函数的代码。如果发生任何错误，我们将返回一个空字符串。

# 摘要

在本章中，我们了解到使用 `NSJSONSerialization` 类解析/构建 JSON 对象所需的代码比解析/构建 XML 对象少得多。然而，我们确实对使用 `NSXMLParser` 类如何解析文档有更多的控制。在使用 `NSJSONSerialization` 类和 `NSXMLParser` 类时，需要记住的关键点是，在尝试使用之前，我们需要检查可选变量是否不包含 nil 值。

虽然看起来大多数新的服务都在使用 JSON 格式而不是 XML，但了解这两种格式都是有益的，因为仍然有大量服务使用 XML。
