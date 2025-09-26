# 第九章. 不可变性的重要性

在面向对象和函数式编程中，不可变对象是在初始化后其状态不能被改变或修改的对象。因此，可变对象在其生命周期结束时保持不变，此时它被解除初始化。相比之下，可变对象在初始化后可以被其他对象无数次地更改。

不可变对象提高了可读性和运行时效率，使用它们简化了我们的应用程序。

本章将通过讨论以下主题并附带代码示例来介绍不可变性的概念：

+   不可变性

+   不可变性的好处

+   可变性的案例

+   与方法比较的示例

    +   副作用和意外后果

    +   测试性

+   复制构造函数

+   镜头

# 不可变性

不可变对象是在初始化后其状态不能被修改的对象。不可变对象这一特性在多线程应用程序中至关重要，因为它允许一个线程在不担心其他线程更改的情况下操作由不可变对象表示的数据。

如果一个对象及其所有属性都是不可变的，则认为该对象是不可变的。在某些情况下，即使某些内部属性发生变化，但对象的状态从外部看起来是不可变的，这样的对象也被认为是不可变的。例如，使用缓存技术来缓存资源密集型计算结果的对象可以被视为不可变对象。

不可变对象具有以下特性：

+   它们易于构建、测试和使用

+   它们易于理解和推理

+   它们天生是线程安全的，没有同步问题

+   它们不需要复制构造函数

+   它们总是具有失败原子性，所以如果不可变对象抛出异常，它将不会陷入不希望/不确定的状态

+   它们提供更高的安全性

# 不可变变量

在命令式编程风格中，应用变量中保持不变的内容被称为常量，以区分那些在执行过程中可能被更改的变量。例如，可能包括视图的高度和宽度或π的几位小数值。

与 Objective-C 等编程语言不同，其中一些类型是可变的，而另一些不是，Swift 提供了一种创建同一类型不可变或可变版本的方法。在 Swift 中，我们使用`let`和`var`关键字来创建和存储值：

+   `var` 关键字用于创建可以稍后更改的变量，换句话说，用于创建可变变量

+   `let` 关键字用于创建不能稍后更改的常量，换句话说，不可变变量或常量

因此，在 Swift 中，我们不需要有像`NSMutableArray`与`NSArray`或`NSMutableDictionary`与`NSDictionary`这样的类型来区分可变性和不可变性。我们可以简单地使用`var`或`let`来定义`Dictionary`，使其可变或不可变。

此外，Swift 编译器总是建议并警告我们那些未更改且将来将被转换为常量的变量。

## 弱不可变与强不可变

有时，一个对象的一些属性可能是不可变的，而其他属性可能是可变的。这类对象被称为弱不可变对象。弱不可变性意味着即使对象的其他部分可能是可变的，我们也不能更改对象状态中的不可变部分。如果所有属性都是不可变的，那么该对象就是不可变的。如果对象创建后整个对象都不能被修改，则该对象被称为强不可变对象。

# 引用类型与值类型

我们在之前的章节中已经讨论了这个问题，但重要的是要强调，在大多数面向对象编程（OOP）语言中，实例可以被共享，对象可以通过它们的引用传递。Swift 类和闭包也是如此。在这些情况下，重要的是要理解，当对象通过引用共享时，对象的状态可能会被改变。换句话说，如果任何用户更改了可变对象的引用，所有其他使用该对象的用户都将受到影响。

# 不可变性的好处

我们已经知道不可变性有助于安全性和性能，但在实际应用开发中，不可变性可以为我们带来更多的好处，这将在以下章节中解释。

## 线程安全

在多线程应用程序中，不可变对象非常有用，因为多个线程可以操作不可变对象的数据，而不用担心其他线程对数据所做的更改。

由于不可变对象几乎不可更改，因此我们可以安全地假设在从不同线程访问对象时它们将保持不变。这个假设简化了大多数复杂且难以解决和维护的多线程问题。例如，我们根本不需要考虑同步/锁定机制。

假设我们有一个包含可变数组类型的可变对象，例如，一个具有四个属性的`Product`类：

```swift
struct Producer {
    let name: String
    let address: String
}

class Product {
    var name: String = ""
    var price: Double = 0.0
    var quantity: Int = 0
    var producer: Producer

    init(name: String,
        price: Double,
     quantity: Int,
     producer: Producer) {

        self.name = name
        self.price = price
        self.quantity = quantity
        self.producer = producer
    }
}

```

如前例所示，所有属性都被定义为可变的。现在让我们创建一个`Product`类型的数组：

```swift
let producer = Producer(name: "ABC",
                     address: "Toronto, Ontario, Canada")

var bananas = Product(name: "Banana",
                     price: 0.79,
                  quantity: 2,
                  producer: producer)

var oranges = Product(name: "Orange",
                     price: 2.99,
                  quantity: 1,
                  producer: producer)

var apples = Product(name: "Apple",
                    price: 3.99,
                 quantity: 3,
                 producer: producer)

var products = [bananas, oranges, apples]

```

假设我们需要`products`数组在不同的线程之间共享。不同的线程可能会以不同的方式更改数组。有些人可能会更改价格，而其他人可能会更改数量。有些人可能会向数组中添加或删除项目。

需要解决的首要问题是跟踪更改并知道谁更改了数组以及何时更改。这已经很复杂了，所以让我们简化问题，只向数组添加一个项目。此外，让我们假设我们只对最新的更改感兴趣。

为了能够跟踪最新的更改，让我们创建另一个对象：

```swift
class ProductTracker {
    private var products: [Product] = []
    private var lastModified: NSDate?

    func addNewProduct(item: Product) -> (date: NSDate,
                                  productCount: Int) {
        products.append(item)
        lastModified = NSDate()
        return (date: lastModified!, productCount: products.count)
    }

    func lastModifiedDate() -> NSDate? {
        return lastModified
    }

    func productList() -> [Product] {
        return products
    }
}

```

`ProductTracker`类有一个`products`数组和一个`lastModified`变量来跟踪最新的更改。此外，它有三个方法：一个用于向数组中添加新产品，另一个用于检索最后修改日期，最后一个用于检索产品列表。

假设我们想要使我们的`ProductTracker`类线程安全，并允许多个对象访问我们的`ProductTracker`对象。我们不能允许多个线程同时执行`addNewProduct`，同时其他线程列出产品。首先，我们需要一个锁定机制来锁定类在修改期间，其次，我们需要保护`lastModified`不被锁定修改，最后，需要一个解锁机制。

苹果提供了多种多线程机制，如`NSThread`、Grand Central Dispatch (GCD)和操作队列，以克服这些问题，但多线程仍然复杂。

## 引用透明性

引用透明性通常意味着我们可以始终用函数的返回值替换函数，而不会影响应用程序的行为。

引用透明性是代码可重用性的保证。它还否认了数据可变状态的存在。在可变状态的情况下，同一函数的两次调用可能产生两个不同的结果，这非常难以测试和维护。

## 低耦合

耦合是代码依赖性的度量。我们总是希望尽可能减少耦合，并使我们的代码组件尽可能独立。低耦合允许我们更改组件而不影响其他代码组件。低耦合的代码更容易阅读，因为每个组件都有其相对较小的责任区域，尽管我们只需要理解这部分代码，而不必花时间弄清楚整个系统是如何工作的。

不可变性有助于实现低耦合。不可变数据可以安全地通过不同的代码块，无需担心其被转换并影响代码的其他部分。纯函数转换数据并返回结果，而不影响输入数据。因此，如果函数包含错误，我们可以轻松找到它。此外，使用值类型和不可变数据结构意味着我们可以显著减少对象引用。

以下展示了数据转换的想法。我们有一个不可变数组`numbers`，我们需要计算它包含的所有偶数的总和：

```swift
let numbers: [Int] = [1, 2, 3, 4, 5]
let sumOfEvens = numbers.reduce(0){$0 + (($1 % 2 == 0) ? $1 : 0) }

```

`numbers`数组没有改变，可以传递给任何其他函数而不会产生任何副作用：

```swift
print(numbers) // [1, 2, 3, 4, 5] 
print(sumOfEvens) // 6

```

使用不可变数据的就地转换可以帮助我们减少耦合。

## 避免时间耦合

假设我们有一个依赖于另一个代码语句的代码语句，如下所示：

```swift
func sendRequest() {
    let sessionConfig = URLSessionConfiguration.default()
    let session = URLSession(configuration: sessionConfig,
                                  delegate: nil,
                             delegateQueue: nil)

    var url: NSURL?
    var request: URLRequest

    /* First request block starts: */
    url = URL(string: "https://httpbin.org/get")
    request = URLRequest(url: url! as URL)
    request.httpMethod = "GET"

    let task = session.dataTask(with: request,
                   completionHandler: {

        (data: Data?, response: URLResponse?, error: NSError?) -> Void in
        if (error == nil) {
            let statusCode = (response as! HTTPURLResponse).statusCode
            print("URL Session Task Succeeded: HTTP \(statusCode)")
        } else {
            print("URL Session Task Failed: %@",   error!.localizedDescription);
        }
        })
    task.resume()
    /* First request block ends */

    /* Second request block starts */
    url = URL(string: "http://requestb.in/1g4pzn21") // replace with
      a new requestb.in
    request = URLRequest(url: url! as URL)

    let secondTask = session.dataTask(with: request,
                         completionHandler: {

        (data: Data?, response: URLResponse?, error: NSError?) -> Void in
        if (error == nil) {
            let statusCode = (response as! HTTPURLResponse).statusCode
            print("URL Session Task Succeeded: HTTP \(statusCode)")
        } else {
            print("URL Session Task Failed: %@",
              error!.localizedDescription);
        }
    })
    secondTask.resume()
}

```

在前面的代码示例中，我们设置了两个不同的 HTTP 请求。假设我们不再需要第一个请求，我们删除以下代码块：

```swift
url = URL(string: "https://httpbin.org/get")
      request = URLRequest(url: url! as URL)
      request.httpMethod = "GET"
```

编译器不会抱怨，但因为我们删除了`request.httpMethod = "GET"`，我们的第二个请求将无法工作。这种情况被称为时间耦合。如果我们使用`let`的不可变定义，我们就可以避免时间耦合。

## 避免身份可变性

如果对象的内部状态相同，我们可能需要对象是相同的。当修改对象的状态时，我们并不期望它改变其身份。不可变对象完全避免了这个问题。

## 失败原子性

如果一个类抛出运行时异常，它可能会处于损坏状态。不可变性防止了这个问题。由于对象的状态仅在初始化期间修改，因此对象永远不会处于损坏状态。初始化要么失败，拒绝对象初始化，要么成功，创建一个有效的坚固对象，该对象永远不会改变其封装的状态。

## 并行化

不可变性使得并行化代码执行更容易，因为对象和实例之间没有冲突。

## 异常处理和错误管理

如果我们只使用不可变类型，即使有异常，我们应用程序的内部状态也将保持一致，因为我们的不可变对象不会保持不同的状态。

## 缓存

对不可变对象的引用可以被缓存，因为它们不会改变；因此，下次我们尝试访问它时，将快速检索到相同的不可变对象。一个示例技术是第二章中解释的**记忆化**，*函数和闭包*。

## 状态比较

应用程序的状态是在给定时间点所有对象的集合状态。状态随时间快速变化，并且应用程序需要改变状态以继续运行。

然而，不可变对象在时间上具有固定的状态。一旦创建，不可变对象的状态就不会改变，尽管整个应用程序的状态可能会改变。这使得跟踪发生的事情和简化状态比较变得容易。

## 编译器优化

编译器对在其生命周期内值不会改变的项的`let`语句进行优化更好。例如，Apple 写道：“在所有情况下，如果集合不需要改变，创建不可变集合都是良好的实践。这样做可以使 Swift 编译器优化您创建的集合的性能。”（在适当的情况下，请优先使用`let`而不是`var`。）

# 可变性案例

每当我们需要更改不可变对象时，我们都需要创建一个新的、修改后的副本。这可能对于小型和简单的对象来说可能并不昂贵和繁琐，但在我们拥有具有大量属性和操作的大型或复杂对象的情况下，这将是如此。

对于具有独特身份的对象，例如用户的个人资料，更改现有对象比创建一个新的、修改过的副本要简单得多，也更直观。我们可能希望维护用户个人资料的单一对象，并在必要时对其进行修改。这可能不是一个很好的例子，因为很难看到这种情况下性能的惩罚，但执行速度对于某些类型的应用程序（如游戏）来说可能是一个非常重要的区别因素。例如，用可变对象表示我们的游戏角色可能会使我们的游戏比需要每次更改时都创建一个新的、修改过的游戏角色副本的替代实现运行得更快。

此外，我们的现实世界感知不可避免地基于可变对象的概念。我们在现实生活中处理周围的所有对象。这些对象大多数时候是相同的，如果需要，我们会改变它们的一些特性。

例如，我们在家里粉刷墙壁而不是更换整个墙壁。我们将墙壁视为具有修改过的属性（在这种情况下是颜色）的相同对象。当我们粉刷墙壁时，墙壁的身份保持不变，但其状态发生变化。

因此，无论何时我们在应用中用模型来表示现实世界的对象，使用可变对象来感知和实现领域模型都是不可避免的更容易。

# 一个例子

我们理解在某些情况下不可变性会使我们的生活变得更难。在前一节中，我们几乎没有触及这些问题的表面。我们将在接下来的章节中更详细地检查这些问题。

让我们以函数式编程（**FP**）风格重新开发我们的`Product`示例，并将其与面向对象（OOP）的对应版本进行比较。

让我们在本章中使`Product`示例不可变，并检查结果：

```swift
struct FunctionalProduct {
    let name: String
    let price: Double
    let quantity: Int
    let producer: Producer
}

```

现在我们有了`struct`而不是类，所有属性都是不可变的。此外，我们不需要`init`方法，因为`struct`会自动提供它。

我们还需要修改我们的`ProductTracker`类：

```swift
struct FunctionalProductTracker {
    let products: [FunctionalProduct]
    let lastModified: NSDate

    func addNewProduct(item: FunctionalProduct) -> (date: NSDate,
                                                products:
                                                  [FunctionalProduct]) {

        let newProducts = self.products + [item]
        return (date: NSDate(), products: newProducts)
    }
}

```

我们的`FunctionalProductTracker`简化了：它是具有不可变`products`数组的`struct`，我们的`addNewProduct`不会修改对象的状态，而是每次都提供一个包含`products`的新数组。实际上，我们可以从这个`struct`中移除`addNewProduct`方法，并在客户端对象中处理它。

## 副作用和意外后果

我们的可变示例设计可能会产生不可预测的副作用。如果有多个客户端持有`ProductTracker`实例的引用，产品可以通过以下两种方式从任何这些客户端的下面发生变化：

+   我们可以直接重新分配`products`的值。这可以通过将其设置为`private`以供客户端调用来修复，但对于类内的修改则无法修复。

+   我们可以从任何客户端调用`addNewProduct()`并修改`products`。

无论哪种方式，由于修改，都可能会出现副作用和意外的后果。

在我们的不可变示例中，由于我们的 `FunctionalProductTracker` 是一个值类型，所有属性都是不可变的，因此不可能产生那些意外的后果。`products` 不能直接更改（它是一个常量），而 `addNewProduct()` 返回一个新的实例，因此所有客户端都将处理他们期望处理的实例。

## 可测试性

我们可变示例的 `addNewProduct()` 方法没有返回值。虽然我们可以为它编写单元测试，但如何实现断言并不明显，因为该方法在我们的现有实例中产生了副作用，我们需要了解这些副作用。

我们不可变示例的 `addNewProduct()` 方法返回一个新的 `Product` 数组。我们只需检查 `products` 的值并断言。我们仍然有旧的和新的实例，所以我们有我们需要的所有东西来确保我们的代码按预期工作。

虽然我们在这本书中没有涵盖单元测试，但我们强烈建议您探索基于 QuickCheck 的库，如 Quick ([`github.com/Quick/Quick`](https://github.com/Quick/Quick)) 和 SwiftCheck ([`github.com/typelift/SwiftCheck`](https://github.com/typelift/SwiftCheck))，因为它们使用函数式编程技术来简化我们应用程序的单元测试过程。

# 复制构造函数和透镜

在检查我们的不可变示例实现后，我们无法断言它涵盖了命令式方法的所有功能。例如，它没有提供更改 `product` 的 `producer` 的方法。毕竟，我们无法更改它。

每当我们需要更改 `product` 的任何属性时，我们都需要经过以下过程：

```swift
let mexicanBananas = FunctionalProduct(name: bananas.name,
                                      price: bananas.price,
                                   quantity: bananas.quantity,
                                   producer: Producer(name: "XYZ",
                                                   address: "New Mexico,
                                                     Mexico"))
```

这个解决方案很冗长，看起来也不美观。让我们看看我们如何改进这个过程。

## 复制构造函数

第一种解决方案是提供一个新的 `init` 方法来复制当前实例。这种方法被称为复制构造函数。让我们添加我们的新 `init` 方法并利用它：

```swift
init(products: [FunctionalProduct],
    lastModified: NSDate) {

    self.products = products
    self.lastModified = lastModified
    }

    init(productTracker: FunctionalProductTracker,
               products: [FunctionalProduct]? = nil,
           lastModified: NSDate? = nil) {

    self.products = products ?? productTracker.products
    self.lastModified = lastModified ?? productTracker.lastModified
 }

```

我们还添加了默认的 `init`，因为通过向我们的 `struct` 添加新的 `init` 方法，我们失去了自动 `init` 生成的好处。我们还需要更改我们的 `addNewProduct` 来适应这些更改：

```swift
  func addNewProduct(item: FunctionalProduct) -> FunctionalProductTracker {

    return FunctionalProductTracker(productTracker: self,
                                          products: self.products + [item])
}

```

无论何时我们需要部分修改我们的对象，我们都可以使用这种技术轻松地做到这一点。

## 透镜

在上一节中，我们介绍了复制构造函数。在这里，我们将检查一个名为透镜的功能结构。简单来说，透镜是针对整个对象及其部分实现的 *函数式获取器和设置器*：

+   获取器：我们可以 *透过* 透镜查看不可变对象以获取其部分

+   设置器：我们可以使用透镜来改变不可变对象的一部分

让我们实现一个 `Lens`：

```swift
struct Lens<Whole, Part> {
    let get: Whole -> Part
    let set: (Part, Whole) -> Whole
}

```

让我们用它来改变我们的 `FunctionalProduct` 对象以获取和设置 `producer` 属性：

```swift
let prodProducerLens: Lens<FunctionalProduct, Producer> =
  Lens(get: { $0.producer},
       set: { FunctionalProduct(name: $1.name,
                               price: $1.price,
                            quantity: $1.quantity,
                            producer: $0)})
```

让我们更改 `mexicanBananas` 的生产者：

```swift
let mexicanBananas2 = prodProducerLens.set(Producer(name: "QAZ",
                                                 address: "Yucatan,
                                                   Mexico"),
                                           mexicanBananas)
```

通过我们的透镜，我们可以像前面代码所示那样更改它。

让我们检查另一个示例。假设我们有一个如下所示的 `Producer` 对象：

```swift
let chineeseProducer = Producer(name: "KGJ",
                             address: "Beijing, China")

```

我们想要更改 `address`：

```swift
let producerAddressLens: Lens<Producer, String> =
  Lens(get: { $0.address },
       set: { Producer(name: $1.name,
                    address: $0)})

let chineeseProducer2 = producerAddressLens.set("Shanghai, China",
  chineeseProducer)
```

假设我们有一个 `mexicanBananas2`，需要有一个中国香蕉生产者，那么我们可以使用：

```swift
let chineseBananaProducer = prodProducerLens.set(
  producerAddressLens.set("Shanghai, China", chineseProducer), 
  mexicanBananas2)
```

这种语法看起来并不简单，而且似乎我们并没有获得太多。在下一节中，我们将简化它。

### 镜头组成

镜头组成将有助于简化我们的镜头；让我们看看它是如何做到的：

```swift
infix operator >>> { associativity right precedence 100 }

func >>><A,B,C>(l: Lens<A,B>, r: Lens<B,C>) -> Lens<A,C> {
    return Lens(get: { r.get(l.get($0)) },
                set: { (c, a) in
                    l.set(r.set(c,l.get(a)), a)
    })
}

```

让我们来测试一下：

```swift
let prodProducerAddress = prodProducerLens >>> producerAddressLens
let mexicanBananaProducerAddress = prodProducerAddress.get(mexicanBananas2)
let newProducer = prodProducerAddress.set("Acupulco, Mexico",
  mexicanBananas2)
print(newProducer)

```

结果将是：

```swift
FunctionalProduct(name: "Banana",
                  price: 0.79,
                  quantity: 2,
                  producer: Producer(name: "QAZ", address: "Acupulco, Mexico"))
```

通过使用镜头和组成，我们能够获取和设置产品的生产地址。

# 摘要

在本章中，我们首先探讨了不可变性的概念。我们通过例子探讨了其重要性和好处。然后我们分析了可变性的情况，并通过一个例子来比较可变性和不可变性对我们代码的影响。

最后，我们探讨了以函数式方式获取和设置不可变对象的方法，例如复制构造函数和镜头。

在接下来的章节中，我们将介绍面向对象编程（OOP）、**协议导向编程**（**POP**）和**函数式响应式编程**（**FRP**）。然后，我们将探讨混合面向对象和函数式编程范式，换句话说，对象函数式编程。
