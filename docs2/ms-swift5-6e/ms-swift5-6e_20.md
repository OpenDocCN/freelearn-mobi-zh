# 20

# 在 Swift 中采用设计模式

尽管四巨头《设计模式：可复用面向对象软件元素》的第一版于 1994 年 10 月发布，但我对设计模式的研究只有 14 年的时间。像大多数经验丰富的开发者一样，当我最初开始阅读有关设计模式的内容时，我认识到了很多模式，因为我已经在不知不觉中使用了它们。我必须说，自从我开始阅读有关设计模式的内容以来，我没有编写过一个不使用至少一种设计模式的严肃应用程序。我会告诉你，我绝对不是一个设计模式狂热者，而且如果我陷入关于设计模式的对话中，通常只有一两个我能不查资料就能叫出名字的。但有一件事我记得很清楚，那就是主要设计模式背后的概念以及它们旨在解决的问题。这样，当我遇到这些问题之一时，我就可以查找适当的模式并应用它。所以，记住，当你阅读这一章时，花时间去理解设计模式背后的主要概念，而不是试图记住模式本身。

在本章中，你将学习以下主题：

+   什么是设计模式？

+   哪些类型的设计模式构成了设计模式的创建、结构和行为类别？

+   如何在 Swift 中实现单例和建造者创建模式

+   如何在 Swift 中实现桥接、外观和代理结构模式

+   如何在 Swift 中实现命令和策略行为模式

# 什么是设计模式？

每个经验丰富的开发者都有一套非正式的策略，这些策略塑造了他们设计和编写应用程序的方式。这些策略是由他们的过去经验和他们在以前的项目中必须克服的障碍所塑造的。虽然这些开发者可能对自己的策略深信不疑，但这并不意味着他们的策略已经得到了充分的检验。使用这些策略也可能在不同项目和开发者之间引入不一致的实现。

虽然设计模式的概念可以追溯到 20 世纪 80 年代中期，但直到四巨头在 1994 年发布了《设计模式：可复用面向对象软件》一书，它们才变得流行。这本书的作者 Erich Gamma、Richard Helm、Ralph Johnson 和 John Vlissides（也被称为四巨头）讨论了面向对象编程的陷阱，并描述了 23 个经典的软件设计模式。这些模式被分为三个类别：*创建型*、*结构型*和*行为型*。

设计模式识别常见的软件开发问题，并提供解决这些问题的策略。这些策略在多年来的实践中已被证明是解决它们旨在解决的问题的有效方案。使用这些模式可以大大加快开发过程，因为它们为我们提供了已被证明可以解决常见软件开发问题的解决方案。

当我们使用设计模式时，我们获得的另一个优点是易于维护的代码，因为几个月或几年后当我们查看代码时，我们将能够识别出模式并理解代码的功能。如果我们适当地记录代码并记录我们正在实施的设计模式，这也有助于其他开发者理解代码的功能。

设计模式背后的两种主要哲学是代码重用和灵活性。作为一名软件架构师，将可重用性和灵活性构建到代码中是至关重要的。这使我们能够轻松地维护代码，并在未来使应用程序更容易扩展以满足未来的需求，因为我们都知道需求变化有多快。

虽然设计模式有很多优点，并且对开发者和架构师来说极为有益，但它们并不是解决世界饥饿问题的方案，就像一些开发者所宣称的那样。在你的开发生涯中，你可能会遇到一些认为设计模式是不可变法则的开发者或架构师。这些开发者通常试图在不需要的情况下强制使用设计模式。一个好的经验法则是，在尝试解决问题之前，确保你有一个需要解决的问题。

设计模式是避免和解决常见编程问题的起点。我们可以将每个设计模式视为一个食谱。就像一个好的食谱一样，我们可以对其进行调整以满足特定的口味。但我们通常不想偏离原始食谱太远，因为我们可能会把它搞砸。

也有时候我们没有一个想要制作的菜肴的食谱，就像有时候我们面对的问题没有现成的设计模式来解决。在这种情况下，我们可以利用我们对设计模式及其潜在哲学的知识来为问题提出一个有效的解决方案。

设计模式通常分为三个类别。具体如下：

+   **创建模式**：创建模式支持对象的创建

+   **结构模式**：结构模式关注类型和对象组合

+   **行为模式**：行为模式在类型之间进行通信

虽然“四人帮”定义了超过 20 种设计模式，但我们将在本章中只查看一些更受欢迎的模式的示例。让我们从查看创建模式开始。

设计模式最初是为面向对象编程定义的。在本章中，尽可能的，我们将关注以更协议导向的方式实现模式。因此，本章中的示例可能与其他设计模式书籍中的示例略有不同，但解决方案的底层哲学将是相同的。

# 创建型设计模式

创建型模式是处理对象创建的设计模式。这些模式以适合特定情况的方式创建对象。

创建型模式背后有两个基本思想。第一个是封装关于“哪些”具体类型应该被创建的知识，第二个是隐藏这些类型实例的创建方式。

有五种著名的模式属于创建型模式类别。它们如下：

+   **抽象工厂模式**：这种模式提供了一个创建相关对象的接口，而不指定具体的类型

+   **建造者模式**：这种模式将复杂对象的构建与其表示分离，因此可以使用相同的流程创建类似类型的对象

+   **工厂方法模式**：这种模式创建对象，而不暴露对象（或对象类型）创建的底层逻辑

+   **原型模式**：这种模式通过克隆现有对象来创建对象

+   **单例模式**：这种模式允许一个（且仅一个）类的实例在整个应用程序的生命周期中存在

在本章中，我们将展示如何在 Swift 中实现单例模式和建造者模式。让我们从一个最具争议性且可能被过度使用的模式——单例模式——开始。

## 单例模式

单例模式的使用在开发社区的一些角落中是一个相当有争议的话题。主要原因之一是单例模式可能是被过度使用和误用的模式之一。这个模式引起争议的另一个原因是它将全局状态引入应用程序中，这提供了在任何应用程序点更改对象的能力。单例模式还可能引入隐藏的依赖和紧耦合。我个人认为，如果单例模式被正确使用，使用它是没有问题的。然而，我们确实需要小心不要误用它。

单例模式限制了一个类在应用程序生命周期内的实例化只能有一个实例。当我们需要恰好一个对象来协调应用程序内的操作时，这种模式非常有效。单例的一个良好用途示例是，如果应用程序通过蓝牙与远程设备通信，并且我们还想在整个应用程序中保持该连接。有些人可能会说，我们可以从一页传递到另一页连接类的实例，这本质上就是单例。在我看来，在这个例子中，单例模式是一个更干净的解决方案，因为使用单例模式，任何需要连接的页面都可以获取它，而不必强迫每个页面都维护实例。这也允许我们在转到另一页时无需重新连接来维护连接。

### 理解问题

单例模式旨在解决的问题是，在应用程序的生命周期中我们需要一个且仅有一个类型的实例。单例模式通常用于我们需要集中管理内部或外部资源，并且需要一个单一的全球访问点。单例模式的另一个流行用途是，当我们想要在整个应用程序中整合一组相关的活动，而这些活动在一个地方不维护状态时。

在*第九章*，*协议与协议扩展*中，我们在文本验证示例中使用了单例模式。在那个例子中，我们使用单例模式是因为我们想要创建一个单一实例的类型，然后可以被应用程序的所有组件使用，而无需我们为这些类型创建新的实例。这些文本验证类型没有可以改变的状态。它们只有执行文本验证的方法和定义如何验证文本的常量。虽然有些人可能不同意我的观点，但我认为这类类型是单例模式的绝佳候选者，因为没有理由创建这些类型的多个实例。

在那个例子中，我们使用结构体来实现它，但这并不是真正的单例，因为结构体是一个值类型。真正的单例是通过引用（类）类型实现的。

当使用单例模式时，最大的担忧之一是多线程应用程序中的竞态条件。问题发生在当一个线程更改单例的状态时，另一个线程正在访问它，从而产生意外的结果。例如，如果`TextValidation`类存储要验证的文本，然后我们调用一个方法进行验证，一个线程可能在原始线程进行验证之前更改存储的文本。在实现此模式之前，了解单例将在您的应用程序中如何使用是明智的。

### 理解解决方案

在 Swift 中实现单例模式有几种方法。在我们使用的方法中，当第一次访问类常量时创建了一个类的单个实例。然后我们将使用类常量来在整个应用程序的生命周期中访问这个实例。我们还将创建一个私有初始化器，这将防止外部代码创建类的额外实例。

注意，我们在描述中使用的是“类”这个词，而不是“类型”。这样做的原因是单例模式只能真正通过引用类型来实现。

### 实现单例模式

让我们看看我们是如何使用 Swift 实现单例模式的。下面的代码示例展示了如何创建一个单例类：

```swift
class MySingleton {
    static let sharedInstance = MySingleton() 
    var number = 0
    private init() {}
} 
```

我们可以看到，在 `MySingleton` 类中，我们创建了一个名为 `sharedInstance` 的静态常量，它包含了一个 `MySingleton` 类的实例。静态常量可以在不实例化类的情况下被调用。由于我们声明了 `sharedInstance` 常量是静态的，因此在整个应用程序的生命周期中只存在一个实例，从而创建了单例模式。

我们还创建了一个私有初始化器，它不能在类外部访问，这将限制其他代码创建 `MySingleton` 类的额外实例。

现在，让我们看看这个模式是如何工作的。`MySingleton` 模式还有一个名为 `number` 的属性，它是一个整数。我们将监控这个属性在我们使用 `sharedInstance` 属性创建多个 `MySingleton` 类型的变量时的变化，如下面的代码所示：

```swift
var singleA = MySingleton.sharedInstance 
var singleB = MySingleton.sharedInstance 
var singleC = MySingleton.sharedInstance
singleB.number = 2
print(singleA.number)
print(singleB.number)
print(singleC.number)
singleC.number = 3
print(singleA.number)
print(singleB.number)
print(singleC.number) 
```

在这个例子中，我们使用 `sharedInstance` 属性创建了三个 `MySingleton` 类型的变量。我们最初将第二个 `MySingleton` 变量（`singleB`）的 `number` 属性设置为数字 `2`。当我们打印出 `singleA`、`singleB` 和 `singleC` 实例的 `number` 属性值时，我们看到所有三个的 `number` 属性值都等于 `2`。

然后，我们将第三个 `MySingleton` 实例（`singleC`）的 `number` 属性的值更改为数字 `3`。当我们再次打印出 `number` 属性的值时，我们看到现在所有三个的值都是 `3`。因此，当我们更改任何实例中 `number` 属性的值时，所有三个的值都会改变，因为每个变量都指向同一个实例。

在这个例子中，我们使用引用（类）类型实现了单例模式，因为我们想确保在整个应用程序中只存在该类型的一个实例。如果我们用结构体或枚举这样的值类型实现这个模式，我们就有可能出现该类型的多个实例。

如果你还记得，每次我们传递一个值类型的实例时，我们实际上是在传递该实例的一个副本，这意味着如果我们用值类型实现单例模式，每次调用`sharedInstance`属性时，我们都会收到一个新的副本，这实际上会破坏单例模式。

当我们需要在整个应用程序中维护对象的状态时，单例模式非常有用；然而，请注意不要过度使用它。除非有特定的要求（*要求*是这里的关键词）在整个应用程序的生命周期中只有一个类的实例，否则不应使用单例模式。如果我们仅仅为了方便而使用单例模式，那么我们可能是在误用它。

请记住，虽然苹果公司通常建议我们优先使用值类型而不是引用类型，但仍然有许多例子，例如单例模式，我们需要使用引用类型。当我们不断告诉自己优先使用值类型而不是引用类型时，很容易忘记有时需要引用类型。不要忘记在使用此模式时使用引用类型。

现在，让我们看看构建者设计模式。

## 构建者模式

**构建者模式**帮助我们创建复杂对象，并强制执行这些对象的创建过程。使用此模式，我们通常将创建逻辑从复杂类型中分离出来，并将创建逻辑放入另一个类型中。这允许我们使用相同的构建过程来创建类型的不同表示形式。

### 理解问题

构建者模式旨在解决当类型实例需要大量可配置值时的问题。我们可以在创建类的实例时设置配置选项，但如果选项设置不正确或我们不知道所有选项的正确值，这可能会引起问题。另一个问题是，每次创建类型实例时可能需要的代码量来设置所有可配置选项。

### 理解解决方案

构建者模式通过引入一个称为构建者类型的中间代理来解决此问题。这个构建者类型包含创建原始复杂类型实例所需的大部分，如果不是全部，信息。

我们可以使用两种方法来实现构建者模式。第一种方法是有多个构建者类型，其中每个类型都包含以特定方式配置原始复杂对象所需的信息。在第二种方法中，我们使用单个构建者类型来设置所有可配置选项的默认值，然后根据需要更改这些值。

在本节中，我们将探讨使用构建者模式的两种方法，因为了解每种方法的工作方式非常重要。

### 实现构建者模式

在我们展示如何使用构建器模式之前，让我们看看如何在不使用构建器模式的情况下创建复杂结构以及我们遇到的问题。

以下代码创建了一个名为 `BurgerOld` 的结构，并且没有使用构建器模式：

```swift
struct BurgerOld {
    var name: String
    var patties: Int
    var bacon: Bool
    var cheese: Bool
    var pickles: Bool
    var ketchup: Bool
    var mustard: Bool
    var lettuce: Bool
    var tomato: Bool
    init(name: String, patties: Int, bacon: Bool, cheese: Bool, pickles:Bool, ketchup: Bool, mustard: Bool,lettuce: Bool, tomato: Bool) {
        self.name = name
        self.patties = patties
        self.bacon = bacon 
        self.cheese = cheese
        self.pickles = pickles
        self.ketchup = ketchup
        self.mustard = mustard
        self.lettuce = lettuce
        self.tomato = tomato
    }
} 
```

在 `BurgerOld` 结构中，我们有一些属性定义了汉堡上有哪些配料以及汉堡的名称。由于我们需要知道哪些项目在汉堡上，哪些不在，当我们创建 `BurgerOld` 结构的实例时，初始化器要求我们定义每个项目。这可能导致应用程序中一些复杂的初始化，更不用说如果我们有多个标准汉堡（培根芝士汉堡、芝士汉堡、汉堡等），我们需要确保每个都定义正确。让我们看看如何创建 `BurgerOld` 类的实例：

```swift
// Create Hamburger
var hamburger = BurgerOld(name: "Hamburger", patties: 1, bacon: false, cheese: false, pickles: true, ketchup: true, mustard: true, lettuce: false, tomato: false)
// Create Cheeseburger
var cheeseburger = BurgerOld(name: "Cheeseburger", patties: 1 , bacon: false, cheese: true, pickles: true, ketchup: true, mustard: true, lettuce: false, tomato: false) 
```

如我们所见，创建 `BurgerOld` 类型的实例需要大量的代码。现在，让我们看看一种更好的方法。在这个例子中，我们将展示如何使用多个构建器类型，其中每种类型将定义每种汉堡上有什么。我们将首先创建一个 `BurgerBuilder` 协议，其中将包含以下代码：

```swift
protocol BurgerBuilder { 
    var name: String { get }
    var patties: Int { get }
    var bacon: Bool { get }
    var cheese: Bool { get }
    var pickles: Bool { get }
    var ketchup: Bool { get }
    var mustard: Bool { get }
    var lettuce: Bool { get }
    var tomato: Bool { get }
} 
```

此协议简单地定义了任何实现此协议的类型所需的九个属性。现在，让我们创建两个实现此协议的结构：`HamburgerBuilder` 和 `CheeseBurgerBuilder` 结构：

```swift
struct HamburgerBuilder: BurgerBuilder { 
    let name = "Burger"
    let patties = 1 
    let bacon = false
    let cheese = false
    let pickles = true
    let ketchup = true 
    let mustard = true 
    let lettuce = false 
    let tomato = false
}
struct CheeseBurgerBuilder: BurgerBuilder { 
    let name = "CheeseBurger"
    let patties = 1 
    let bacon = false 
    let cheese = true 
    let pickles = true 
    let ketchup = true 
    let mustard = true
    let lettuce = false
    let tomato = false
} 
```

在 `HamburgerBuilder` 和 `CheeseBurgerBuilder` 结构中，我们所做的只是为每个所需的属性定义值。在更复杂类型中，我们可能需要初始化额外的资源。

现在，让我们看看 `Burger` 结构，它将使用 `BurgerBuilder` 协议的实例来创建自身的实例。以下代码展示了这种新的 `Burger` 类型：

```swift
struct Burger {
    var name: String
    var patties: Int
    var bacon: Bool 
    var cheese: Bool 
    var pickles: Bool 
    var ketchup: Bool 
    var mustard: Bool 
    var lettuce: Bool 
    var tomato: Bool
    init(builder: BurgerBuilder) { 
        self.name = builder.name
        self.patties = builder.patties
        self.bacon = builder.bacon
        self.cheese = builder.cheese
        self.pickles = builder.pickles
        self.ketchup = builder.ketchup
        self.mustard = builder.mustard
        self.lettuce = builder.lettuce
        self.tomato = builder.tomato
    }
    func showBurger() { 
        print("Name:\(name)") 
        print("Patties: \(patties)") 
        print("Bacon: \(bacon)") 
        print("Cheese: \(cheese)") 
        print("Pickles: \(pickles)") 
        print("Ketchup: \(ketchup)") 
        print("Mustard: \(mustard)") 
        print("Lettuce: \(lettuce)") 
        print("Tomato: \(tomato)")
    }
} 
```

与前面展示的 `BurgerOld` 结构相比，这个 `Burger` 结构的不同之处在于初始化器。在先前的 `BurgerOld` 结构中，初始化器接受九个参数——结构中定义的每个常量一个。在新结构中，初始化器接受一个参数，即符合 `BurgerBuilder` 协议的类型的一个实例。这个新的初始化器允许我们按照以下方式创建 `Burger` 类的实例：

```swift
// Create Hamburger
var myBurger = Burger(builder: HamburgerBuilder()) 
myBurger.showBurger()
// Create Cheeseburger
var myCheeseBurger = Burger(builder: CheeseBurgerBuilder())
// Let's hold the ketchup
myCheeseBurger.ketchup = false
myCheeseBurger.showBurger() 
```

如果我们将创建新 `Burger` 结构实例的方法与先前的 `BurgerOld` 结构进行比较，我们可以看到创建 `Burger` 结构的实例要容易得多。我们还知道我们正确地设置了每种汉堡的类型属性值，因为这些值是在构建器类中直接设置的。

如我们之前提到的，我们可以使用第二种方法来实现建造者模式。而不是有多个建造者类型，我们可以有一个单一的建造者类型，它将所有可配置选项设置为默认值；然后根据需要更改这些值。我在更新旧代码时经常使用这种方法，因为它很容易与现有代码集成。

对于这种实现，我们将创建一个单一的`BurgerBuilder`结构。这个结构将用于创建`BurgerOld`结构的实例，并且默认情况下将所有成分设置为它们的默认值。

`BurgerBuilder`结构还使我们能够在创建`BurgerOld`结构的实例之前更改将放在汉堡上的成分。我们创建`BurgerBuilder`结构的方式如下：

```swift
struct BurgerBuilder { 
    var name = "Burger"
    var patties = 1
    var bacon = false 
    var cheese = false 
    var pickles = true 
    var ketchup = true 
    var mustard = true 
    var lettuce = false 
    var tomato = false
mutating func setPatties(choice: Int) { 
        self.patties = choice
    }
mutating func setBacon(choice: Bool) { 
        self.bacon = choice
    }
mutating func setCheese(choice: Bool) { 
        self.cheese = choice
    }
mutating func setPickles(choice: Bool) { 
        self.pickles = choice
    }
mutating func setKetchup(choice: Bool) { 
        self.ketchup = choice
    }
mutating func setMustard(choice: Bool) { 
        self.mustard = choice
    }
mutating func setLettuce(choice: Bool) { 
        self.lettuce = choice
    }
mutating func setTomato(choice: Bool) { 
        self.tomato = choice
    }
    func buildBurgerOld(name: String) -> BurgerOld {
        return BurgerOld(name: name, patties: self.patties,bacon: self.bacon, cheese: self.cheese,pickles: self.pickles, ketchup: self.ketchup,mustard: self.mustard, lettuce: self.lettuce,tomato: self.tomato)
    }
} 
```

在`BurgerBuilder`结构中，我们定义了汉堡的九个属性（成分），然后为除了`name`属性之外的所有属性创建了一个 setter 方法。

我们还创建了一个名为`buildBurgerOld()`的方法，它将根据`BurgerBuilder`实例的属性值创建`BurgerOld`结构的实例。我们使用`BurgerBuilder`结构的方式如下：

```swift
var burgerBuilder = BurgerBuilder()
burgerBuilder.setCheese(choice: true)
burgerBuilder.setBacon(choice: true)
var jonBurger = burgerBuilder.buildBurgerOld(name: "Jon's Burger") 
```

在这个例子中，我们创建了一个`BurgerBuilder`结构的实例。然后我们使用`setCheese()`和`setBacon()`方法向汉堡中添加奶酪和培根。最后，我们调用`buildBurgerOld()`方法来创建`burgerOld`结构的实例。

如我们所见，两种实现建造者模式的方法都极大地简化了复杂类型的创建。这两种方法还确保了实例被正确地配置为默认值。如果你发现自己正在创建具有非常长和复杂初始化命令的类型实例，我建议你查看建造者模式，看看你是否可以使用它来简化初始化。

现在，让我们看看结构型设计模式。

# 结构型设计模式

**结构型设计模式**描述了类型如何组合成更大的结构。这些更大的结构通常更容易处理，并且可以隐藏许多单个类型的复杂性。结构型模式类别中的大多数模式都涉及对象之间的连接。

有七个广为人知的模式是结构型设计模式类型的一部分。具体如下：

+   **适配器（Adapter）**: 这允许具有不兼容接口的类型一起工作

+   **桥接（Bridge）**: 这用于将类型的抽象元素与实现分离，以便两者可以独立变化

+   **复合（Composite）**: 这允许我们将一组对象视为单个对象处理

+   **装饰器（Decorator）**: 这让我们能够在现有对象的方法中添加或覆盖行为

+   **外观（Facade）**: 这为更大的、更复杂的代码库提供了一个简化的接口

+   **享元（Flyweight）**: 这允许我们减少创建和使用大量相似对象所需的资源

+   **代理**：这是一个充当其他类或类的接口的类型

在本章中，我们将给出如何在 Swift 中使用桥接、外观和代理模式的示例。让我们首先看看桥接模式。

## 桥接模式

**桥接模式**解耦了抽象和实现，以便它们可以独立地变化。桥接模式也可以被视为两层抽象。

### 理解问题

桥接模式旨在解决几个问题，但我们在这里将要关注的问题通常随着时间的推移，随着新功能和新需求的到来而出现。在某个时刻，当这些需求到来时，我们需要改变功能之间的交互方式。最终，这将需要我们重构代码。

在面向对象编程中，这被称为爆炸性的类层次结构，但在协议导向编程中也可能发生。

### 理解解决方案

桥接模式通过将交互功能分离，将每个功能特有的功能与它们之间共享的功能分离来解决此问题。然后可以创建一个桥接类型，它将封装共享功能，将它们组合在一起。

### 实现桥接模式

为了演示我们如何使用桥接模式，我们将创建两个功能。第一个功能是一个消息功能，它将存储和准备我们希望发送的消息。第二个功能是发送者功能，它将通过特定的通道发送消息，例如电子邮件或短信。

让我们首先创建两个名为 `Message` 和 `Sender` 的协议。`Message` 协议将定义用于创建消息的类型的要求。`Sender` 协议将用于定义用于通过特定通道发送消息的类型的要求。

以下代码显示了如何定义这两个协议：

```swift
protocol Message {
    var messageString: String { get set } 
    init(messageString: String)
    func prepareMessage()
}
protocol Sender {
    func sendMessage(message: Message)
} 
```

`Message` 协议定义了一个名为 `messageString` 的单一属性，该属性的类型为 `String`。这个属性将包含消息的文本，并且不能为 nil。我们还定义了一个初始化器和一个名为 `prepareMessage()` 的方法。初始化器将用于设置 `messageString` 属性以及消息类型所需的其他任何内容。`prepareMessage()` 方法将用于在发送消息之前准备消息。此方法可以用于加密消息或添加格式。

`Sender` 协议定义了一个名为 `sendMessage()` 的方法。此方法将通过符合类型定义的通道发送消息。在此函数中，我们需要确保在发送消息之前调用消息类型的 `prepareMessage()` 方法。

现在让我们看看我们如何定义两个符合 `Message` 协议的类型：

```swift
class PlainTextMessage: Message { 
    var messageString: String
    required init(messageString: String) { 
        self.messageString = messageString
    }
    func prepareMessage() {
        //Nothing to do
    }
}
class DESEncryptedMessage: Message { 
    var messageString: String
    required init(messageString: String) { 
        self.messageString = messageString
    }
    func prepareMessage() {
        // Encrypt message here
        self.messageString = "DES: " + self.messageString
    }
} 
```

这些类型都包含符合 `Message` 协议所需的功能。这些类型之间唯一的真正区别在于 `prepareMessage()` 方法。在 `PlainTextMessage` 类中，`prepareMessage()` 方法是空的，因为我们不需要在发送消息之前对消息进行任何操作。`DESEncryptionMessage` 类的 `prepareMessage()` 方法通常会包含加密消息的逻辑，但在这个示例中，我们将在消息的开头添加一个 `DES` 标签，这样我们就可以知道这个方法被调用了。

现在，让我们创建两个将符合 `Sender` 协议的类型。这些类型通常会处理通过特定通道发送消息；然而，在这个示例中，我们只是将消息打印到控制台：

```swift
class EmailSender: Sender {
    func sendMessage(message: Message) { 
        print("Sending through E-Mail:")
        print("\(message.messageString)")
    }
}
class SMSSender: Sender {
    func sendMessage(message: Message) { 
        print("Sending through SMS:")
        print("\(message.messageString)")
    }
} 
```

`EmailSender` 和 `SMSSender` 类型都通过实现 `sendMessage()` 函数符合 `Sender` 协议。

现在我们可以使用这两个功能，如下面的代码所示：

```swift
var myMessage = PlainTextMessage(messageString: "Plain Text Message") 
myMessage.prepareMessage()
var sender = SMSSender() 
sender.sendMessage(message: myMessage) 
```

这将很好地工作，我们可以在需要创建和发送消息的任何地方添加类似的代码。现在，假设在不久的将来，我们收到一个需求，需要在发送消息之前验证消息，以确保它符合我们通过该通道发送消息的要求。

要做到这一点，我们首先需要将 `Sender` 协议修改为添加 `verify` 功能。

新的 `Sender` 协议如下所示：

```swift
protocol Sender {
    var message: Message? { get set } 
    func sendMessage()
    func verifyMessage()
} 
```

我们向 `Sender` 协议添加了一个名为 `verifyMessage()` 的方法，并添加了一个名为 `Message` 的属性。我们还更改了 `sendMessage()` 方法的定义。原始的 `Sender` 协议被设计为简单地发送消息，但现在我们需要在调用 `sendMessage()` 函数之前验证消息；因此，我们不能像上一个定义中那样简单地传递消息。

现在，我们需要更改符合 `Sender` 协议的类型，使它们符合这个新协议。下面的代码展示了我们将如何进行这些更改：

```swift
class EmailSender: Sender { 
    var message: Message? 
    func sendMessage() {
        print("Sending through E-Mail:")
        print("\(message!.messageString)")
    }
    func verifyMessage() { 
        print("Verifying E-Mail message")
    }
}
class SMSSender: Sender { 
    var message: Message? 
    func sendMessage() {
        print("Sending through SMS:") 
        print("\(message!.messageString)")
    }
    func verifyMessage() { 
        print("Verifying SMS message")
    }
} 
```

在对符合 `Sender` 协议的类型所做的更改之后，我们需要更改代码使用这些类型的方式。以下示例展示了我们现在如何使用它们：

```swift
var myMessage = PlainTextMessage(messageString: "Plain Text Message")
myMessage.prepareMessage()
var sender = SMSSender()
sender.message = myMessage
sender.verifyMessage()
sender.sendMessage() 
```

这些更改并不困难；然而，如果没有桥接模式，我们就需要重构整个代码库，并在发送消息的每个地方进行更改。桥接模式告诉我们，当我们有两个紧密交互的层次结构时，我们应该将这些交互逻辑放入一个桥接类型中，这样就可以将逻辑封装在一个地方。这样，当我们收到新的需求或增强时，我们可以在一个地方进行更改，从而限制必须进行的重构。我们可以为消息和发送者层次结构创建一个桥接类型，如下面的示例所示：

```swift
struct MessagingBridge {
    static func sendMessage(message: Message, sender: Sender) { 
        var sender = sender
        message.prepareMessage()
        sender.message = message
        sender.verifyMessage()
        sender.sendMessage()
    }
} 
```

消息和发送者层次结构如何交互的逻辑现在被封装到`MessagingBridge`结构中。现在，当逻辑需要更改时，我们只需要更改这个结构，而无需重构整个代码库。

桥接模式是一个非常值得记住和使用的模式。曾经（现在仍然）有几次我后悔没有在我的代码中使用桥接模式，因为众所周知，需求经常变化，能够在代码库的一个地方而不是整个代码库中做出更改可以节省我们未来大量的时间。

现在，让我们看看结构类别中的下一个模式：外观模式。

## 外观模式

**外观模式**提供了一个简化的接口来访问更大、更复杂的代码库。这使我们能够通过隐藏一些复杂性来使库更容易使用和理解。它还允许我们将多个 API 组合成一个单一、易于使用的 API，这正是我们将在示例中看到的。

### 理解问题

外观模式通常用于我们有一个复杂系统，该系统具有大量独立且旨在协同工作的 API。有时在初始应用程序设计中很难确定我们应该在哪里使用外观模式。原因是我们通常试图简化初始 API 设计；然而，随着时间的推移，随着需求的变化和新功能的添加，API 变得越来越复杂，然后就会明显地知道应该在何处使用外观模式。

### 理解解决方案

外观模式的主要思想是在一个简单的接口后面隐藏 API 的复杂性。这为我们提供了几个优点，最明显的是它简化了我们与 API 的交互方式。它还促进了松散耦合，这使得 API 可以根据需求的变化而变化，而无需重构使用它们的所有代码。

### 实现外观模式

为了演示外观模式，我们将创建三个 API：`HotelBooking`、`FlightBooking`和`RentalCarBooking`。这些 API 将用于搜索和预订酒店、航班和租赁汽车。虽然我们可以在代码中非常容易地单独调用每个 API，但我们将创建一个`TravelFacade`结构，这将允许我们通过单次调用访问 API 的功能。

我们将首先定义三个 API。每个 API 都需要一个数据存储类来存储有关酒店、航班或租赁汽车的信息。我们将从实现酒店 API 开始：

```swift
struct Hotel {
    //Information about hotel room
}
struct HotelBooking {
    static func getHotelNameForDates(to: Date, from: Date) -> [Hotel]? { 
        let hotels = [Hotel]()
        //logic to get hotels
        return hotels
    }
    static func bookHotel(hotel: Hotel) {
        // logic to reserve hotel room
    }
} 
```

酒店 API 由`Hotel`和`HotelBooking`结构组成。`Hotel`结构将用于存储酒店房间的信息，而`HotelBooking`结构将用于搜索酒店房间并为旅行预订房间。航班和租赁汽车 API 与酒店 API 非常相似。以下代码显示了这两个 API：

```swift
struct Flight {
    //Information about flights
}
struct FlightBooking {
    static func getFlightNameForDates(to: Date, from: Date) ->[Flight]? {
        let flights = [Flight]()
        //logic to get flights return flights
    }
    static func bookFlight(flight: Flight) {
        // logic to reserve flight
    }
}
struct RentalCar {
    //Information about rental cars
}
struct RentalCarBooking {
    static func getRentalCarNameForDates(to: Date, from: Date)-> [RentalCar]?
    {
        let cars = [RentalCar]()
        //logic to get flights return cars
    }
    static func bookRentalCar(rentalCar: RentalCar) {
        // logic to reserve rental car
    }
} 
```

在这些 API 中，我们有一个用于存储信息的结构和一个用于提供搜索/预订功能的结构。在初始设计中，在应用程序中调用这些单个 API 会非常容易；然而，正如我们所知，需求往往会变化，这会导致 API 随着时间的推移而变化。

通过在这里使用外观模式，我们能够隐藏 API 的实现方式；因此，如果我们需要在未来更改 API 的工作方式，我们只需要更新外观类型，而不是重构所有代码。这使得代码在未来更容易维护和更新。现在让我们看看我们将如何通过创建一个`TravelFacade`结构来实现外观模式：

```swift
struct TravelFacade { 
    var hotels: [Hotel]?
    var flights: [Flight]? 
    var cars: [RentalCar]?
    init(to: Date, from: Date) {
        hotels = HotelBooking.getHotelNameForDates(to: to, from:from) 
        flights = FlightBooking.getFlightNameForDates(to: to, from:from) 
        cars = RentalCarBooking.getRentalCarNameForDates(to: to, from:from)
    }
    func bookTrip(hotel: Hotel, flight: Flight, rentalCar: RentalCar) {
        HotelBooking.bookHotel(hotel: hotel) 
        FlightBooking.bookFlight(flight: flight)
        RentalCarBooking.bookRentalCar(rentalCar: rentalCar)
    }
} 
```

`TravelFacade`类包含搜索三个 API 并预订酒店、航班和租赁车的功能。我们现在可以使用`TravelFacade`类来搜索酒店、航班和租赁车，而无需直接访问单个 API。

正如我们在本章开头提到的，在初始设计中何时使用外观模式并不总是显而易见的。

一个好的规则是：如果我们有几个 API 协同工作以执行任务，我们应该考虑使用外观模式。

现在，让我们看看最后一个结构化模式，即代理设计模式。

## 代理模式

在**代理设计模式**中，有一个类型充当另一个类型或 API 的接口。这个包装类，即代理，可以添加功能到对象中，使对象可以通过网络访问，或者限制对对象的访问。

### 理解问题

我们可以使用代理模式解决几个问题，但我发现我主要使用这个模式来解决两个问题之一。

我使用这个模式要解决的问题的第一个问题是，当我想在单个 API 和我的代码之间创建一层抽象时。API 可以是本地或远程 API，但我通常使用这个模式在代码和远程服务之间添加一个抽象层。这将允许在不需要重构大量应用程序代码的情况下更改远程 API。

我使用代理模式要解决的第二个问题是我需要更改 API，但没有代码或应用程序的其他地方已经对 API 有依赖。

### 理解解决方案

为了解决这些问题，代理模式告诉我们应该创建一个类型，该类型将充当与其他类型或 API 交互的接口。在示例中，我们将展示如何使用代理模式向现有类型添加功能。

### 实现代理模式

在本节中，我们将通过创建一个可以添加多个楼层平面图的房屋类来演示代理模式，其中每个楼层平面图代表房屋的不同楼层。让我们首先创建一个`FloorPlan`协议：

```swift
protocol FloorPlan {
    var bedRooms: Int { get set }
    var utilityRooms: Int { get set } 
    var bathRooms: Int { get set } 
    var kitchen: Int { get set }
    var livingRooms: Int { get set }
} 
```

在`FloorPlan`协议中，我们定义了五个属性，将代表每个楼层平面包含的房间数量。现在，让我们创建一个名为`HouseFloorPlan`的`FloorPlan`协议的实现，如下所示：

```swift
struct HouseFloorPlan: FloorPlan { 
    var bedRooms = 0
    var utilityRooms = 0 
    var bathRooms = 0 
    var kitchen = 0
    var livingRooms = 0
} 
```

`HouseFloorPlan`结构实现了`FloorPlan`协议所需的全部五个属性，并为它们分配了默认值。接下来，我们将创建`House`类型，它将代表一个房屋：

```swift
struct House {
    var stories = [FloorPlan]()
    mutating func addStory(floorPlan: FloorPlan) { 
        stories.append(floorPlan)
    }
} 
```

在`House`结构中，我们有一个符合`FloorPlan`协议的实例数组，其中每个楼层平面将代表房屋的一层。我们还有一个名为`addStory()`的函数，它接受一个符合`FloorPlan`协议的类型实例。此函数将楼层平面添加到`FloorPlan`协议的数组中。

如果我们考虑这个类的逻辑，可能会遇到一个问题：我们可以添加任意多的楼层平面，这可能会导致房屋高达 60 或 70 层。如果我们正在建造摩天大楼，那将很棒，但我们只想建造基本的单户住宅。如果我们想限制楼层平面的数量，而不改变`House`类（我们可能无法改变它，或者我们只是不想改变它），我们可以实现代理模式。以下示例展示了如何实现`HouseProxy`类，其中我们限制了可以向房屋添加的楼层平面数量：

```swift
struct HouseProxy { 
    var house = House()
    mutating func addStory(floorPlan: FloorPlan) -> Bool { 
        if house.stories.count < 3 {
            house.addStory(floorPlan: floorPlan)
            return true
        } else {
            return false
        }
    }
} 
```

我们从创建`House`类的一个实例开始`HouseProxy`类。然后我们创建一个名为`addStory()`的方法，它允许我们向房屋添加新的楼层平面。在`addStory()`方法中，我们检查房屋的故事数量是否少于三个；如果是这样，我们将楼层平面添加到房屋中并返回`true`。如果故事数量等于或大于三个，则我们不向房屋添加楼层平面并返回`false`。让我们看看我们如何使用这个代理：

```swift
var ourHouse = HouseProxy()
var basement = HouseFloorPlan(bedRooms: 0, utilityRooms: 1, bathRooms:1,kitchen: 0, livingRooms: 1)
var firstStory = HouseFloorPlan (bedRooms: 1, utilityRooms: 0,bathRooms: 2,kitchen: 1, livingRooms: 1)
var secondStory = HouseFloorPlan (bedRooms: 2, utilityRooms: 0,bathRooms: 1,kitchen: 0, livingRooms: 1)
var additionalStory = HouseFloorPlan (bedRooms: 1, utilityRooms: 0,bathRooms:1, kitchen: 1, livingRooms: 1)
ourHouse.addStory(floorPlan: basement)
ourHouse.addStory(floorPlan: firstStory)
ourHouse.addStory(floorPlan: secondStory)
ourHouse.addStory(floorPlan: additionalStory) 
```

在示例代码中，我们首先创建了一个名为`ourHouse`的`HouseProxy`类实例。然后我们创建了四个`HouseFloorPlan`类型的实例，每个实例的房间数量都不同。最后，我们尝试将每个楼层平面添加到`ourHouse`实例中。如果我们运行此代码，我们将看到前三个`floorplans`类的实例成功添加到房屋中，但最后一个没有添加，因为我们只能添加三层。

代理模式在我们想要为一个类型添加一些额外的功能或错误检查时非常有用，但我们又不想改变该类型的实际实现。我们还可以用它来在远程或本地 API 之间添加一层抽象。

现在，让我们看看行为设计模式。

# 行为设计模式

行为设计模式解释了类型之间如何交互。这些模式描述了不同类型的实例如何相互发送消息以使事情发生。

有九种著名的模式属于行为设计模式类型。它们如下所示：

+   **责任链**：用于处理各种请求，每个请求可能被委派给不同的处理者。

+   **命令**：创建可以封装操作或参数的对象，以便稍后或由不同的组件调用。

+   **迭代器**：允许我们按顺序访问对象中的元素，而不暴露底层结构。

+   **中介者**：用于减少相互通信的类型之间的耦合。

+   **备忘录**：用于捕获对象的当前状态，并以可以稍后恢复的方式存储它。

+   **观察者**：允许一个对象发布对其状态的更改。其他对象可以订阅，以便在发生任何更改时得到通知。

+   **状态**：当对象的内部状态发生变化时，用于改变对象的行为。

+   **策略**：这允许在运行时从算法家族中选择一个。

+   **访问者**：这是一种将算法与对象结构分离的方法。

在本节中，我们将给出如何在 Swift 中使用策略和命令模式的示例。让我们先看看命令模式。

## 命令模式

命令设计模式让我们能够定义可以在以后执行的操作。这个模式通常封装了在以后时间调用或触发操作所需的所有信息。

### 理解问题

在应用程序中，有时我们需要将命令的执行与其调用者分离。通常，这是当我们有一个需要执行多个操作之一的类型时，但需要运行时选择使用哪个操作。

### 理解解决方案

命令模式告诉我们应该将操作逻辑封装到一个符合命令协议的类型中。然后我们可以为调用者提供命令类型的实例。调用者将使用协议提供的接口来调用必要的操作。

### 实现命令模式

在本节中，我们将通过创建一个`Light`类型来演示如何使用命令模式。在这个类型中，我们将定义`lightOnCommand`和`lightOffCommand`命令，并使用`turnOnLight()`和`turnOffLight()`方法来调用这些命令。我们首先创建一个名为`Command`的协议，所有命令类型都将遵守。以下是`Command`协议：

```swift
protocol Command { 
    func execute()
} 
```

此协议包含一个名为`execute()`的方法，它将被用于执行命令。现在，让我们看看`Light`类型将使用哪些命令类型来打开和关闭灯光。它们如下所示：

```swift
struct RockerSwitchLightOnCommand: Command { 
    func execute() {
        print("Rocker Switch:Turning Light On")
    }
}
struct RockerSwitchLightOffCommand: Command { 
    func execute() {
        print("Rocker Switch:Turning Light Off")
    }
}
struct PullSwitchLightOnCommand: Command { 
    func execute() {
        print("Pull Switch:Turning Light On")
    }
}
struct PullSwitchLightOffCommand: Command { 
    func execute() {
        print("Pull Switch:Turning Light Off")
    }
} 
```

`RockerSwitchLightOffCommand`、`RockerSwitchLightOnCommand`、`PullSwitchLightOnCommand` 和 `PullSwitchLightOffCommand` 命令都通过实现 `execute()` 方法符合 `Command` 协议；因此，我们将能够在 `Light` 类型中使用它们。现在，让我们看看如何实现 `Light` 类型：

```swift
struct Light {
    var lightOnCommand: Command
    var lightOffCommand: Command
    func turnOnLight() { 
        self.lightOnCommand.execute()
    }
    func turnOffLight() { 
        self.lightOffCommand.execute()
    }
} 
```

在 `Light` 类型中，我们首先创建两个变量，分别命名为 `lightOnCommand` 和 `lightOffCommand`，它们将包含符合 `Command` 协议的类型实例。然后，我们创建 `turnOnLight()` 和 `turnOffLight()` 方法，我们将使用这些方法来打开和关闭灯光。在这些方法中，我们调用适当的命令来打开或关闭灯光。

我们将按照以下方式使用 `Light` 类型：

```swift
var on = PullSwitchLightOnCommand()
var off = PullSwitchLightOffCommand()
var light = Light(lightOnCommand: on, lightOffCommand: off)
light.turnOnLight()
light.turnOffLight()
light.lightOnCommand = RockerSwitchLightOnCommand() 
light.turnOnLight() 
```

在这个例子中，我们首先创建了一个名为 `on` 的 `PullSwitchLightOnCommand` 类型的实例和一个名为 `off` 的 `PullSwitchLightOffCommand` 类型的实例。然后，我们使用这两个刚刚创建的命令创建了一个 `Light` 类型的实例，并调用 `Light` 实例的 `turnOnLight()` 和 `turnOffLight()` 方法来打开和关闭灯光。在最后两行中，我们将 `lightOnCommand` 方法，原本设置为 `PullSwitchLightOnCommand` 类型的实例，更改为 `RockerSwitchLightOnCommand` 类型的实例。现在，每次我们打开灯光时，`Light` 实例将使用 `RockerSwitchLightOnCommand` 类型的实例。这允许我们在运行时更改 `Light` 类型的功能。

使用命令模式有几个优点。其中一个主要优点是我们能够在运行时设置要调用的命令，这也让我们能够在整个应用程序的生命周期中根据需要替换符合 `Command` 协议的不同实现。命令模式的另一个优点是将命令实现的细节封装在命令类型本身中，而不是在容器类型中。

现在，让我们来看最后一个设计模式，即策略模式。

## 策略模式

策略模式与命令模式非常相似，因为它们都允许我们将实现细节与调用类型解耦，并且允许我们在运行时切换实现。主要区别在于策略模式旨在封装算法。通过替换算法，我们期望对象执行相同的功能，但以不同的方式。在命令模式中，当我们替换命令时，我们期望对象改变功能。

### 理解问题

在应用程序中，有时我们需要更改执行操作所使用的后端算法。通常，这是当我们有一个具有几种不同算法的类型，这些算法可以用来执行相同任务时，但需要根据运行时选择使用哪种算法。

### 理解解决方案

策略模式告诉我们应该将算法封装在一个符合策略协议的类型中。然后我们可以提供策略类型的实例供调用者使用。调用者将使用协议提供的接口来调用算法。

### 实现策略模式

在本节中，我们将通过展示如何在运行时替换压缩算法来演示策略模式。让我们从这个例子开始，创建一个 `CompressionStrategy` 协议，每个压缩类型都将符合此协议。让我们看看以下代码：

```swift
protocol CompressionStrategy {
    func compressFiles(filePaths: [String])
} 
```

此协议定义了一个名为 `compressFiles()` 的方法，该方法接受一个参数，即包含我们想要压缩的文件路径的字符串数组。我们现在将创建两个符合此协议的结构。这些是 `ZipCompressionStrategy` 和 `RarCompressionStrategy` 结构，如下所示：

```swift
struct ZipCompressionStrategy: CompressionStrategy { 
    func compressFiles(filePaths: [String]) {
        print("Using Zip Compression")
    }
}
struct RarCompressionStrategy: CompressionStrategy { 
    func compressFiles(filePaths: [String]) {
        print("Using RAR Compression")
    }
} 
```

这两种结构都通过使用名为 `compressFiles()` 的方法实现了 `CompressionStrategy` 协议，该方法接受一个字符串数组。在这些方法中，我们只是简单地打印出我们正在使用的压缩名称。通常，我们会将这些方法中的压缩逻辑实现出来。

现在，让我们看看 `CompressContent` 类，它将被用来压缩文件：

```swift
struct CompressContent {
    var strategy: CompressionStrategy
    func compressFiles(filePaths: [String]) { 
        self.strategy.compressFiles(filePaths: filePaths)
    }
} 
```

在这个类中，我们首先定义了一个名为 `strategy` 的变量，它将包含一个符合 `CompressionStrategy` 协议的类型实例。然后我们创建了一个名为 `compressFiles()` 的方法，该方法接受一个字符串数组作为参数，该数组包含我们希望压缩的文件路径列表。在这个方法中，我们使用 `strategy` 变量中设置的压缩策略来压缩文件。

我们将按照以下方式使用 `CompressContent` 类：

```swift
var filePaths = ["file1.txt", "file2.txt"]
var zip = ZipCompressionStrategy()
var rar = RarCompressionStrategy()
var compress = CompressContent(strategy: zip)
compress.compressFiles(filePaths: filePaths)
compress.strategy = rar
compress.compressFiles(filePaths: filePaths) 
```

我们首先创建一个包含我们希望压缩的文件的字符串数组。我们还创建了 `ZipCompressionStrategy` 和 `RarCompressionStrategy` 类型的实例。然后我们创建了一个 `CompressContent` 类的实例，将压缩策略设置为 `ZipCompressionStrategy` 实例，并调用 `compressFiles()` 方法，该方法将在控制台打印出 `Using zip compression` 消息。然后我们将压缩策略设置为 `RarCompressionStrategy` 实例，并再次调用 `compressFiles()` 方法，这将打印出 `Using rar compression` 消息到控制台。

策略模式非常适合在运行时设置要使用的算法，这也允许我们根据应用程序的需要用不同的实现来替换算法。策略模式的另一个优点是，我们将算法的细节封装在策略类型本身中，而不是在主实现类型中。

这篇关于 Swift 中设计模式的游览就此结束。

# 摘要

设计模式是针对我们在现实世界的应用程序设计中反复遇到的设计问题的解决方案。这些模式旨在帮助我们创建可重用和灵活的代码。设计模式还可以使代码对其他开发者以及我们在几个月或几年后回顾代码时更容易阅读和理解。

如果我们仔细观察本章的示例，我们会注意到设计模式的一个关键组成部分是协议。几乎所有的设计模式（单例设计模式除外）都使用协议来帮助我们创建非常灵活和可重用的代码。

如果你这是第一次真正地研究设计模式，你可能注意到了一些你在自己的代码中过去使用过的策略。当经验丰富的开发者第一次接触设计模式时，这是预料之中的。我也鼓励你阅读更多关于设计模式的内容，因为它们肯定会帮助你创建更灵活和可重用的代码。

Swift 是一种快速发展的语言，因此保持对其最新版本的跟进非常重要。由于 Swift 是一个开源项目，因此有大量的资源可以帮助你。我强烈建议你在你最喜欢的浏览器中收藏 [`swiftdoc.org`](http://swiftdoc.org)。它为 Swift 语言自动生成了文档，是一个极好的资源。

另一个需要收藏的网站是 [`swift.org`](https://swift.org)。这是主要的开源 Swift 网站。在这个网站上，你可以找到 Swift 源代码、博客文章、入门页面以及有关如何安装 Swift 的信息。

我还建议你在 swift.org 网站上注册一些邮件列表。这些列表位于社区部分。`Swift-users` 邮件列表是一个询问问题的绝佳地方，也是苹果公司监控的列表。如果你想跟上 Swift 的变化，那么我建议订阅 `swift-evolution-announce` 列表。

我希望你喜欢阅读这本书，就像我喜欢写这本书一样。
