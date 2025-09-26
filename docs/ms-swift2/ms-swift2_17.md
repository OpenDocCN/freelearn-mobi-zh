# 第十七章. 在 Swift 中采用设计模式

尽管四人帮的《设计模式：可复用面向对象软件元素》一书首次出版于 1994 年 10 月，但直到最近 6 或 7 年，我才开始关注设计模式。像大多数经验丰富的开发者一样，当我最初开始阅读有关设计模式的内容时，我认识到了很多模式，因为我已经在没有意识到它们是什么的情况下使用了它们。我必须说，自从我开始阅读有关设计模式以来，过去 6 或 7 年，我没有在不使用四人帮的至少一种设计模式的情况下编写过任何严肃的应用程序。我要告诉你，我绝对不是设计模式的狂热者，实际上，如果我陷入关于设计模式的对话，通常只有一两样我可以不用查找就能说出名字，但有一件事我记得很清楚，那就是主要模式的概念以及它们旨在解决的问题。这样，当我遇到这些问题之一时，我可以查找适当的模式并应用它。

在本章中，你将了解以下主题：

+   引用类型和值类型之间的区别

+   什么是设计模式

+   构成设计模式创建、结构和行为类别的模式类型

+   如何在 Swift 中实现建造者、工厂方法和单例创建模式

+   如何在 Swift 中实现桥接、外观和代理结构模式

+   如何在 Swift 中实现策略和命令行为模式

# 值类型与引用类型

在第五章中，我们讨论了值类型和引用类型之间的区别。了解这两种类型的基本区别非常重要，尤其是在我们构建代码架构时。某些设计模式与引用类型配合得最好，而其他则与值类型配合得最好；因此，知道何时使用每种类型在设计模式中很重要。考虑到这一点，让我们回顾一下引用类型和值类型之间的区别。

类是一个引用类型。这意味着当我们传递一个类的实例在我们的代码中时，我们正在传递原始实例的引用。由于我们正在传递原始实例的引用，因此对这个实例所做的任何更改都会反映到原始实例上。

结构体、枚举和元组都是值类型。当我们传递一个值类型的实例时，我们正在传递该类型的副本。这意味着对这个副本所做的任何更改都不会反映到原始实例上。让我们通过查看一些代码来了解值类型和引用类型之间的区别。我们将首先创建一个名为`MyClass`的类和一个名为`MyStruct`的结构体。这两种类型都包含一个名为`number`的单个属性，其类型为 Int：

```swift
class MyClass {
    var number = 0
}

struct MyStruct {
    var number = 0
}
```

现在，让我们创建一个`MyClass`类的实例。我们还将创建一个`MyClass`类型的第二个常量，它是由第一个实例创建的。然后我们将改变其中一个实例的`number`属性，并查看两个实例中的值：

```swift
let myClass1 = MyClass()
let myClass2 = myClass1

myClass2.number = 5

print("myClass1 = \(myClass1.number)")
print("myClass2 = \(myClass2.number)")
```

如果我们运行这段代码，我们会看到以下输出：

```swift
myClass1 = 5
myClass2 = 5
```

如我们所见，当我们在一个实例中改变了`number`属性时，它在两个实例中都改变了值。这也意味着在内存中只有一个`MyClass`类的实例。

现在，让我们看看这个相同的例子，但这次，我们将使用`MyStruct`结构（值类型）而不是`MyClass`类（引用类型）：

```swift
var myStruct1 = MyStruct()
var myStruct2 = myStruct1

myStruct2.number = 5

print("myStruct1 = \(myStruct1.number)")
print("myStruct2 = \(myStruct2.number)")
```

如果我们运行这段代码，我们会看到以下输出：

```swift
myStruct1 = 0
myStruct2 = 5
```

注意，在这个例子中，当我们改变一个实例的`number`属性时，它并没有改变另一个实例的属性。由于`myStruct2`结构是通过`myStruct`结构的副本创建的，我们现在在内存中有两个`MyStruct`结构的实例。

还要注意，我们使用`let`关键字将`MyClass`类的实例定义为常量；然而，我们使用`var`关键字将`MyStruct`结构的实例定义为变量。

当一个常量引用一个引用类型的实例时，我们无法改变常量所引用的实例；然而，我们可以改变该实例的属性值，就像上一个例子中所示。当一个常量引用一个值类型的实例时，我们不仅无法改变常量所引用的实例，而且也无法改变任何属性值。Swift 的数组和字典是值类型，这就是为什么当它们被声明为常量时是不可变的。这意味着，在我们的上一个例子中，为了改变`number`属性的值，我们需要将`MyStruct`结构的实例作为变量而不是常量来创建。

### 注意

在本章的一些设计模式中，我们使用了结构体，而在其他一些中，我们使用了类。选择使用结构体或类来举例是基于作者的经验。对于大多数模式，选择使用结构体或类应该基于单个应用程序的需求。在每个部分中，我们将解释为什么选择结构体或类，以帮助您理解为什么做出这样的选择。

# 什么是设计模式

每个经验丰富的开发者都有一套非正式的策略，这些策略塑造了他/她设计和编写应用程序的方式。这些策略是由他们过去的经验和他们在以前的项目中必须克服的障碍所塑造的。尽管这些开发者可能发誓支持他们的策略，但这并不意味着他们的策略已经完全经过检验和证明。使用这些策略也引入了不同开发者之间不一致的实现。

设计模式识别一个常见的软件开发问题，并提供了解决该问题的策略。多年来，这些设计模式背后的策略已被证明能够有效地解决它们旨在解决的问题。

虽然设计模式有很多优点，并且对开发人员和架构师来说非常有益，但它们并不是一些开发者所说的解决世界饥饿问题的方案。在你的开发生涯中，你可能会遇到一些认为设计模式是不可变法则的开发人员或架构师。这些开发者通常会试图强制使用设计模式，即使它们不是必要的。一个好的经验法则是，在尝试解决问题之前，确保你有一个问题要解决。

请记住，设计模式是避免和解决常见编程问题的起点。我们可以将每个设计模式视为一道菜品的食谱，就像一个好的食谱一样，我们可以对其进行调整以满足我们的特定口味，但我们通常不想偏离原始食谱太远，因为我们可能会把它搞砸。

也有时候我们没有某个菜品的食谱，就像有时候没有设计模式来解决我们面临的问题一样。在这些情况下，我们可以利用我们对设计模式和它们背后的哲学的了解，来提出一个有效的解决方案来解决问题。

设计模式可以分为三类：

+   **创建型模式**：这些模式支持对象的创建

+   **结构模式**：这些模式关注类和对象的组合

+   **行为模式**：这些模式关注类之间的通信

虽然“四人帮”定义了超过 20 种设计模式，但我们将在本章中仅给出一些最流行模式的示例。让我们从创建型模式开始。

# 创建型模式

创建型模式是处理对象创建的设计模式。这些模式以适合特定情况的方式创建对象。创建型模式背后有两个基本思想。第一个是封装应该创建哪些具体类的知识，第二个是隐藏这些类的实例是如何创建的。在创建型模式类别中，有五种知名的模式：

+   **抽象工厂模式**：提供一个接口来创建相关对象，而不指定具体的类

+   **建造者模式**：将复杂对象的构建与其表示分离，以便可以使用相同的流程创建类似类型的对象

+   **工厂方法模式**：创建对象而不暴露对象或其类型创建的底层逻辑

+   **原型模式**：通过克隆现有对象来创建对象

+   **单例模式**：这允许一个类在应用程序的生命周期内只有一个实例

在本章中，我们将向您展示如何在 Swift 中使用构建器、工厂方法和单例模式的示例。让我们从最具有争议性且可能被过度使用的模式之一——单例模式开始。

## 单例设计模式

单例模式在开发社区的一些角落里是一个相当有争议的话题。其中一个主要原因是单例模式可能是最被过度使用和误用的模式。这个模式引起争议的另一个原因是，单例模式将全局状态引入了应用程序中，这使得在应用程序的任何点上都可以更改对象，从而忽略了作用域。我个人认为，如果单例模式被正确使用，使用它是没有问题的；然而，我们确实需要小心不要误用它。

单例模式限制了类在应用程序生命周期内的实例化，使其只有一个实例。当我们需要恰好一个对象来协调应用程序内的操作时，这个模式非常有效。单例的一个良好用途是，如果我们的应用程序通过蓝牙与远程设备通信，并且我们还想在整个应用程序中保持这个连接。虽然有些人可能会说我们可以从一页传递连接类的实例到另一页，但这本质上就是单例。

在我看来，在这个例子中，单例模式要干净得多，因为有了单例模式，任何需要连接的页面都可以获取它，而不必强迫每个页面都维护实例。这也允许我们在不每次切换到另一页时重新连接的情况下维护连接。

### 注意

在 Swift 中实现单例模式有几种方法。这里介绍的方法使用了类常量，这是在 Swift 1.2 版本中引入的。

让我们看看如何使用 Swift 实现单例模式。以下代码示例展示了如何创建一个单例类：

```swift
class MySingleton {
  static let sharedInstance = MySingleton()
  var number = 0

  private init() {}

}
```

我们可以看到，在 `MySingleton` 类中，我们创建了一个名为 `sharedInstance` 的静态常量，它包含了一个 `MySingleton` 类的实例。静态常量可以在不实例化类的情况下被调用。由于我们声明了 `sharedInstance` 常量是静态的，因此在整个应用程序的生命周期中只有一个实例存在，从而创建了单例模式。

我们还创建了一个私有的初始化器，这将限制其他代码创建 `MySingleton` 类的另一个实例。

现在，让我们看看这个模式是如何工作的。`MySingleton` 模式还有一个名为 `number` 的属性，其类型为 Int。我们将监控这个属性在我们使用 `sharedInstance` 属性创建多个 `MySingleton` 类型的变量时的变化，如下面的代码所示：

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

在这个例子中，我们使用`sharedInstance`属性创建了三个`MySingleton`类型的变量。我们最初将第二个`MySingleton`变量（`singleB`）的`number`属性设置为数字`2`。当我们打印出`singleA`、`singleB`和`singleC`的`number`属性值时，我们看到这三个的`number`属性都等于`2`。然后我们改变第三个`MySingleton`变量（`singleC`）的`number`属性值为数字`3`。当我们再次打印出`number`属性值时，我们看到这次，所有三个的值现在都是`3`。因此，当我们改变任何实例的`number`属性值时，它将改变所有三个的值，因为每个变量都指向同一个实例。

当我们需要在整个应用程序中维护一个对象的状态时，单例模式非常有用，但请注意不要过度使用它。单例模式只有在有具体要求（关键字是需求）在整个应用程序的生命周期中只有一个实例时才应该使用。如果我们仅仅为了方便而使用单例模式，那么我们就是在误用它。

### 注意

对于单例模式，我们创建了`MySingleton`类型作为一个类（引用类型），因为我们希望确保在整个应用程序中只存在该类型的一个实例。如果我们把`MySingleton`类型作为一个结构（值类型），我们就会面临存在多个实例的风险，因为结构是值类型。

现在，让我们看看构建者设计模式。

## 构建者设计模式

构建者模式帮助我们创建复杂对象，并强制执行这些对象的创建过程。使用这个模式，我们通常将创建逻辑从复杂的类中分离出来，并将其放入另一个类中。这允许我们使用相同的构建过程来创建类的不同表示形式。

在本节中，我们将通过创建一个`Burger`类并使用各种不同的汉堡构建者来创建不同类型的汉堡来了解如何使用构建者模式。在我们了解如何使用构建者模式之前，让我们看看如何在不使用构建者模式的情况下创建一个`Burger`类以及我们可能会遇到的问题。

以下代码创建了一个名为`BurgerOld`的类，并且没有使用构建者模式：

```swift
class BurgerOld {
    var name: String
    var patties: Int
    var bacon: Bool
    var cheese: Bool
    var pickles: Bool
    var ketchup: Bool
    var mustard: Bool
    var lettuce: Bool
    var tomato: Bool

    init(name: String, patties: Int, bacon: Bool, cheese: Bool, pickles: Bool,ketchup: Bool,mustard: Bool,lettuce: Bool,tomato: Bool) {
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

在`BurgerOld`类中，有几个属性定义了汉堡上有什么以及汉堡的名称。由于我们需要知道哪些项目在汉堡上，哪些不在，当我们创建`BurgerOld`类的实例时，初始化器要求我们定义每个项目。这可能导致我们应用程序中的一些复杂初始化，更不用说如果我们有多个标准汉堡（培根芝士汉堡、芝士汉堡、汉堡等），我们需要确保我们正确地定义了每个汉堡。让我们看看如何创建`BurgerOld`类的实例：

```swift
// Create Hamburger
var burgerOld = BurgerOld(name: "Hamburger", patties: 1, bacon: false, cheese: false, pickles: false, ketchup: false, mustard: false, lettuce: false, tomato: false)

// Create Cheeseburger
var burgerOld = BurgerOld(name: "Cheeseburger", patties: 1, bacon: false, cheese: false, pickles: false, ketchup: false, mustard: false, lettuce: false, tomato: false)
```

现在，让我们看看更好的做法。我们将首先创建一个`BurgerBuilder`协议，其中将包含以下代码：

```swift
protocol BurgerBuilder {
    var name: String {get}
    var patties: Int {get}
    var bacon: Bool {get}
    var cheese: Bool {get}
    var pickles: Bool {get}
    var ketchup: Bool {get}
    var mustard: Bool {get}
    var lettuce: Bool {get}
    var tomato: Bool {get}
}
```

此协议简单地定义了任何实现此协议的类所需的九个属性。现在，让我们创建两个实现此协议的类——`HamburgerBuilder`和`CheeseBurgerBuilder`类：

```swift
class HamBurgerBuilder: BurgerBuilder {
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

class CheeseBurgerBuilder: BurgerBuilder {
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

在`HamburgerBuilder`和`CheeseBurgerBuilder`类中，我们所做的只是为每个必需的属性定义值。在更复杂的类中，我们可能需要初始化由该实例需要的其他对象。

现在，让我们看看我们的`Burger`类，它将使用`BugerBuilder`协议的实现来创建自身的实例。让我们看看以下代码：

```swift
class Burger {
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
    print("Name:    \(name)")
    print("Patties: \(patties)")
    print("Bacon:   \(bacon)")
    print("Cheese:  \(cheese)")
    print("Pickles: \(pickles)")
    print("Ketchup: \(ketchup)")
    print("Mustard: \(mustard)")
    print("Lettuce: \(lettuce)")
    print("Tomato:  \(tomato)")
}
}
```

与本节前面展示的`BurgerOld`类相比，这个`Burger`类的主要区别在于初始化器。在先前的`BurgerOld`类中，初始化器接受九个参数——每个类中定义的一个常量。在新的`Burger`类中，初始化器接受一个参数，即符合`BurgerBuilder`协议的类的实例。这个新的初始化器允许我们像这样创建`Burger`类的实例：

```swift
// Create Hamburger
var myBurger = Burger(builder: HamBurgerBuilder())
myBurger.showBurger()

// Create Cheeseburger with tomatoes
var myCheeseBurgerBuilder = CheeseBurgerBuilder()
var myCheeseBurger = Burger(builder: myCheeseBurgerBuilder)
myCheeseBurger.tomato = false
myCheeseBurger.showBurger()
```

如果我们将创建新`Burger`类实例的方式与早期的`BurgerOld`类进行比较，我们可以看到创建`Burger`类的实例要容易得多。我们还知道我们为每种汉堡设置了正确的值，因为这些值是在构建类中直接设置的。

如我们所见，建造者模式帮助我们简化了复杂对象的创建。它还确保我们的对象被完全创建。

### 注意

在这个例子中，对于我们的构建类型，我们选择使用类（引用类型）。实际上，使用引用类型或值类型并没有太大的优势；因此，选择了引用类型，因为它没有理由复制我们的构建类型。

在我们最后一个创建型模式的例子中，我们将看看工厂方法模式。

## 工厂方法模式

工厂方法模式使用工厂方法来创建对象的实例，而不指定将要创建的确切类。这允许我们在运行时选择要创建的确切类。

让我们通过创建一个允许我们从多个型号中选择计算机的计算机商店类来查看如何使用工厂方法模式。我们将首先创建一个名为`Computer`的协议。每个代表不同计算机模型的类都将实现`Computer`协议。以下是`Computer`协议的代码：

```swift
protocol Computer {
  func getType() -> String
}
```

`Computer`协议中只有一个方法，它返回一个表示计算机型号的字符串类型。现在，让我们创建三个具体类来实现`Computer`协议：

```swift
class MacbookPro: Computer {
  func getType() -> String {
    return "Macbook Pro"
  }
}
class IMac: Computer {
  func getType() -> String {
    return "iMac"
  }
}

class MacMini: Computer {
  func getType() -> String {
    return "MacMini"
  }
}
```

实现`Computer`协议的三个类中的每一个都在`getType()`方法中返回一个独特的字符串类型。这将标识创建了哪个类。现在，让我们看看我们的`ComputerStore`类，它将根据我们寻找的计算机类型创建这三个类中的一个实例。让我们看看以下代码：

```swift
class ComputerStore {
  enum ComputerType {
    case Laptop
    case Desktop
    case Headless
  }

  func getModel(type: ComputerType) -> Computer {
    switch(type) {
    case ComputerType.Laptop:
      return MacbookPro()
    case ComputerType.Desktop:
      return IMac()
    case ComputerType.Headless:
      return MacMini()  
    }
  }
}
```

在`ComputerStore`类中，我们首先创建一个名为`ComputerType`的枚举，它定义了我们销售的计算机类型。这些类型是`Laptop`、`Desktop`和`Headless`。

`ComputerStore`类有一个方法，那就是`getModel()`方法。该方法接受一个参数，该参数是`ComputerType`类型，并返回一个符合`Computer`协议的类型的实例，具体取决于传入的`ComputerType`枚举。在这个方法中，我们创建了一个`switch`语句，该语句将创建并返回一个符合`Computer`协议的类的实例。

现在，让我们看看如何使用`ComputerStore`类：

```swift
var laptop = store.getModel(.Laptop)
print(laptop.getType())
```

在这个例子中，我们首先创建`ComputerStore`类的一个实例。然后，我们调用`getModel()`方法，通过传递一个`ComputerType`值来检索符合`Computer`协议的类的实例。调用`getModel()`方法的代码不需要知道后端代码如何选择要创建哪种类型的类；它只知道它应该得到一个符合`Computer`协议或`nil`的有效实例。

我发现自己经常使用这个模式。每当有多个类型符合相同的协议时，我们可能需要考虑使用工厂方法模式来集中创建这些对象；否则，我们可能会发现我们在应用程序的多个部分中重复了对象创建代码。

### 注意

与构建者模式类似，我们选择使用类来表示不同的计算机类型，主要是因为创建多个计算机类型的副本没有意义。

关于设计模式，尤其是创建模式的关键思想之一是，我们将有关如何以及创建什么的逻辑从我们的通用代码中提取出来，并将其放入特定的类或函数中。然后，当我们未来需要修改我们的代码时，逻辑嵌入在单个位置，可以轻松更改，而不是在代码的多个位置中都有逻辑。

现在，让我们来看看结构化设计模式。

# 结构型设计模式

结构型设计模式描述了如何将类组合成更大的结构。这些更大的结构通常更容易处理，并隐藏了大量单个类的复杂性。结构型模式类别中的大多数模式都涉及对象之间的连接。

有七个著名的模式属于结构型设计模式类型：

+   **适配器（Adapter）**：这允许具有不兼容接口的类一起工作

+   **桥接（Bridge）**：这用于将类的抽象元素与实现分离，以便两者可以独立变化

+   **组合（Composite）**：这允许我们将一组对象视为单个对象

+   **装饰器（Decorator）**：这让我们可以在现有对象的方法中添加或覆盖行为

+   **外观（Façade）**：这为更大的更复杂的代码库提供了一个简化的接口

+   **享元（Flyweight）**：这允许我们减少创建和使用大量相似对象所需的资源

+   **代理（Proxy）**：这是一个充当其他类或类的接口的类

在本章中，我们将给出如何在 Swift 中使用桥接、外观和代理模式的示例。让我们首先看看桥接模式。

## 桥接模式

桥接模式将抽象与实现解耦，以便它们可以独立变化。桥接模式也可以被视为一种双层抽象。

在本节中，我们将通过创建一个简单的通用遥控器类来展示如何使用桥接模式，该类可以控制多个电视对象。我们将首先创建遥控器和电视的协议，如下面的代码所示：

```swift
protocol TV {
  var currentChannel: Int {get set}

  func turnOn()
  func turnOff()
}

protocol RemoteControl {
  var tv: TV {get set}
  init(tv: TV)
}
```

`TV` 协议定义了一个属性和两个函数。`currentChannel` 属性用于跟踪电视当前所在的频道。函数 `turnOn()` 和 `turnOff()` 用于打开或关闭电视。

`RemoteControl` 协议定义了一个属性和一个初始化器。`tv` 属性持有我们想要控制的电视实例。初始化器将使用符合 `TV` 协议的类型启动遥控器。

现在，我们将扩展 `TV` 和 `RemoteControl` 协议，为符合协议的类型添加常用功能。请注意，这里添加的功能可以在符合协议的类型中重写：

```swift
extension TV {
    mutating func changeChannel(channel: Int) {
        self.currentChannel = channel
    }
}

extension RemoteControl {
    func turnOn() {
        tv.turnOn()
    }
    func turnOff() {
        tv.turnOff()
    }
    mutating func setChannel(channel: Int) {
        tv.changeChannel(channel)
    }
    mutating func nextChannel() {
        tv.changeChannel(tv.currentChannel + 1)
    }
    mutating func prevChannel() {
        tv.changeChannel(tv.currentChannel - 1)
    }
}
```

在 `TV` 扩展中，我们添加了一个方法来更改电视的频道。在 `RemoteControl` 扩展中，我们添加了五个方法，这些方法可以打开/关闭电视或更改电视的频道。

现在，让我们看看如何创建符合 `TV` 协议的结构。为此，我们将定义两个协议的具体实现，如下所示：

```swift
struct VizioTV: TV {

    var currentChannel = 1

    func turnOn() {
        print("Vizio On")
    }
    func turnOff() {
        print("Vizio Off")
    }
}

struct SonyTV: TV {

    var currentChannel = 1

    func turnOn() {
        print("Sony On")
    }
    func turnOff() {
        print("Sony Off")
    }
}
```

使用这段代码，我们定义了`SonyTV`和`VizioTV`对`TV`协议的实现。在这些结构体中，我们实现了`TV`协议的所有要求。我们将使用这些实现来告诉通用遥控器控制哪个电视。现在，让我们看看如何实现`RemoteControl`协议，如下所示：

```swift
class MyUniversalRemote: RemoteControl {
    var tv: TV

    required init(tv: TV) {
        self.tv = tv
    }
}
```

在`MyUniversalRemote`类中，我们实现了`Remote`协议所需的初始化器。

要使用此模式，我们首先创建一个我们想要控制的`TV`类型的实例。然后，我们会使用该实例来启动我们的遥控器类型，如下面的代码所示：

```swift
var myTv = VizioTV()
var remote = MyUniversalRemote(tv: myTv)
remote.turnOn()
remote.nextChannel()
print("Channel on: \(myTv.currentChannel)")
remote.nextChannel()
print("Channel on: \(myTv.currentChannel)")
remote.turnOff()
```

桥接模式可以看作是两层抽象，其中抽象和实现不应该在编译时绑定。这允许我们在运行时定义要使用哪些对象。这也允许我们通过创建实现`TV`协议的新类来简单地给我们的`myUniversalRemote`类添加更多的电视。

### 注意

对于这个模式，我们使用结构体实现了符合`TV`协议的类型。选择结构体是因为创建一个`TV`类型的实例非常容易，然后可以使用它来创建多个遥控器类型的实例，如下面的示例所示：

```swift
var myTv = VizioTV()
var remoteForTV1 = MyUniversalRemote(tv: myTv)
var remoteForTV2 = MyUniversalRemote (tv: myTv)
```

在这个例子中，如果`VizioTV`类型是用类实现的，那么两个`MyUniversalRemote`实例将引用同一个电视而不是不同的电视。因此，尽管我们有两个电视，每个电视都有单独的遥控器，但这两个遥控器实际上都在操作同一个电视。

有时候我们想要这种行为，对于那些时候，我们应该使用类；然而，根据我的经验，这通常不是我们想要的行为。

现在，让我们看看结构类别中的下一个模式——外观模式。

## 外观模式

外观模式提供了一个简化的大规模和更复杂代码库的接口。这允许我们通过隐藏一些复杂性来使我们的库更容易使用和理解。它还允许我们将多个 API 组合成一个单一、更易于使用的 API，这正是我们将在示例中看到的。

在这个例子中，我们将创建一个简化的旅行 API，将酒店、航班和租车 API 组合成一个单一、易于使用的接口。我们将从定义酒店、航班和租车类开始，如下所示：

```swift
struct HotelBooking {
    static func getHotelNameForDates(to: NSDate, from: NSDate) -> [String]? {
        let hotels = [String]()
        //logic to get hotels
        return hotels
    }
}

struct FlightBooking {
    static func getFlightNameForDates(to: NSDate, from: NSDate) -> [String]? {
        let flights = [String]()
        //logic to get flights
        return flights
    }
}

struct RentalCarBooking {
    static func getRentalCarNameForDates(to: NSDate, from: NSDate) -> [String]? {
        let cars = [String]()
        //logic to get flights
        return cars
    }
}
```

在这些 API 中的每一个，我们定义一个单一的静态方法，该方法将返回一个列表，其中包含（酒店、航班或租车）在请求日期可用的项目。实际上，我们在这里没有实现任何逻辑，因为我们需要定义一个数据源，而我更喜欢保持示例简单，以便集中精力了解模式的工作原理。

现在，让我们看看我们的`TravelFacade`类，它将这三个 API 组合成一个单一、更易于使用的 API，如下面的代码所示：

```swift
class TravelFacade {
var hotels: [String]?
var flights: [String]?
var cars: [String]?    

    init(to: NSDate, from: NSDate) {
        hotels = HotelBooking.getHotelNameForDates(to, from: from)
        flights = FlightBooking.getFlightNameForDates(to, from: from)
        cars = RentalCarBooking.getRentalCarNameForDates(to, from: from)
    }
}
```

在 `TravelFacade` 类内部，我们创建了一个单例初始化器，它接受两个 `NSDate` 对象作为参数。然后我们使用这两个 `NSDate` 对象来检索在由日期定义的时间段内可用的酒店、航班和租赁汽车。

当我们有一个复杂的 API 结构想要简化时，外观模式非常有用。它同样非常有用，当我们有一系列多个相关的 API，就像我们在示例中看到的那样，将它们合并到一个 API 中。

### 注意

对于这个模式，我们在实现三种预订类型时选择了使用结构体；然而，使用哪种类型（类或结构体）实际上取决于应用程序的个体设计。在本章的 *桥接模式* 部分，我们能够说在大多数情况下结构体会被优先选择；然而，在这个模式中，我们真的不能说哪种类型在大多数情况下会被优先选择。

现在，让我们看看我们的最后一个结构化模式，即代理设计模式。

## 代理设计模式

在代理设计模式中，有一个对象充当其他对象的接口。这个包装类，即代理，可以给对象添加功能，使对象可以通过网络访问，或者限制对对象的访问。

在本节中，我们将通过创建一个可以添加多个楼层平面图（每个楼层平面图代表房子的不同楼层）的房屋类来演示代理模式。让我们首先创建一个 `FloorPlanProtocol` 协议：

```swift
protocol FloorPlanProtocol {
  var bedRooms: Int {get set}
  var utilityRooms: Int {get set}
  var bathRooms: Int {get set}
  var kitchen: Int {get set}
  var livingRooms: Int {get set}
}
```

在 `FloorPlanProtocol` 中，我们定义了五个属性，这些属性将代表每个楼层平面图中包含的房间数量。现在，让我们创建一个名为 `FloorPlan` 的 `FloorPlanProtocol` 协议的实现，如下所示：

```swift
struct FloorPlan: FloorPlanProtocol {
  var bedRooms = 0
  var utilityRooms = 0
  var bathRooms = 0
  var kitchen = 0
  var livingRooms = 0
}
```

`FloorPlan` 类实现了从 `FloorPlanProtocol` 所需的所有五个属性，并为它们分配了默认值。接下来，我们将创建一个名为 `House` 的类，它将代表一栋房子：

```swift
class House {
  private var stories = [FloorPlanProtocol]()

  func addStory(floorPlan: FloorPlanProtocol) {
    stories.append(floorPlan)
  }
}
```

在我们的 `House` 类中，我们有一个 `FloorPlanProtocols` 对象的数组，其中每个楼层平面图将代表房子的一个楼层。我们还有一个名为 `addStory()` 的函数，它接受一个符合 `FloorPlanProtocol` 协议的对象实例。这个函数将楼层平面图添加到 `FloorPlanProtocols` 协议的数组中。

如果我们考虑这个类的逻辑，可能会遇到一个问题。问题是我们可以添加任意多的楼层平面图，这可能会导致房子有 60 或 70 层高。如果我们是在建造摩天大楼，那将很棒，但我们只是想建造基本的单户家庭住宅。如果我们想限制楼层平面图的数量而不改变 `House` 类（我们无法改变它，或者我们只是不想改变它），我们可以实现代理模式。以下示例展示了如何实现 `HouseProxy` 类，其中我们限制了可以添加到房子中的楼层平面图数量，如下所示；

```swift
class HouseProxy {
 var house = House()

  func addStory(floorPlan: FloorPlanProtocol) -> Bool {
    if house.stories.count < 3 {
      house.addStory(floorPlan)
      return true
    }
    else {
      return false
    }
  }
}
```

我们从创建`HouseProxy`类的一个实例开始，然后创建一个名为`addStory()`的方法，允许我们向房子添加一个新的楼层平面图。在`addStory()`方法中，我们检查房子的楼层数是否少于三，如果是这样，我们就将楼层平面图添加到房子中并返回`true`。如果楼层数等于或大于三，则我们不将楼层平面图添加到房子中并返回`false`。让我们看看我们如何使用这个代理：

```swift
var ourHouse = HouseProxy()

var basement = FloorPlan(bedRooms: 0, utilityRooms: 1, bathRooms: 1, kitchen: 0, livingRooms: 1)
var firstStory = FloorPlan(bedRooms: 1, utilityRooms: 0, bathRooms: 2, kitchen: 1, livingRooms: 1)
var secondStory = FloorPlan(bedRooms: 2, utilityRooms: 0, bathRooms: 1, kitchen: 0, livingRooms: 1)
var additionalStory = FloorPlan(bedRooms: 1, utilityRooms: 0, bathRooms: 1, kitchen: 1, livingRooms: 1)

print(ourHouse.addStory(basement))
print(ourHouse.addStory(firstStory))
print(ourHouse.addStory(secondStory))
print(ourHouse.addStory(additionalStory))
```

在我们的示例代码中，我们首先创建了一个名为`ourHouse`的`HouseProxy`类实例。然后，我们创建了四个`FloorPlan`类实例，每个实例都有不同数量的房间。最后，我们尝试将每个楼层平面图添加到`ourHouse`实例中。如果我们运行代码，我们会看到前三个`FloorPlan`类实例成功添加到了房子中，但最后一个没有添加，因为我们只能添加三层楼。

当我们想要向一个类添加一些额外的功能或错误检查，但又不想改变实际的类本身时，代理模式非常有用。

### 注意

对于代理模式，我们选择使用一个类来实现这个模式，因为通常我们不会想要复制我们代理的类型。相反，我们通常希望保持对实例所做的更改。这在某种程度上是桥接模式的反义词，在我的经验中，结构在大多数情况下会被优先考虑。

现在，让我们看看行为设计模式。

# 行为设计模式

行为设计模式解释了对象之间如何交互。这些模式描述了不同的对象如何发送消息给彼此以使事情发生。

有九种众所周知的设计模式属于结构设计模式类型：

+   **责任链**：这个用于处理各种请求，每个请求都可能被委派给不同的处理者。

+   **命令**：这创建可以封装动作或参数的对象，以便稍后或由不同的组件调用。

+   **迭代器**：这允许我们按顺序访问对象中的元素，而不暴露底层结构。

+   **中介者**：这个用于减少相互通信的类之间的耦合。

+   **备忘录**：这个用于捕获对象的当前状态，并以可以稍后恢复的方式存储它。

+   **观察者**：这允许一个对象发布其状态的变化。其他对象可以订阅，以便在发生任何变化时得到通知。

+   **状态**：这个用于在对象的内部状态改变时改变对象的行为。

+   **策略**：这允许在运行时从一组算法中选择一个。

+   **访问者**：这是一种将算法与对象结构分离的方法。

在本节中，我们将给出如何在 Swift 中使用策略和命令模式的示例。让我们首先看看命令模式。

## 命令设计模式

命令设计模式允许我们定义可以在以后执行的操作。此模式通常封装了在以后时间调用或触发操作所需的所有信息。

在本节中，我们将通过创建一个 `Light` 类来演示如何使用命令模式。在这个例子中，我们将定义两个命令——`lightOnCommand` 和 `lightOffCommand`。然后我们将使用 `turnOnLight()` 和 `turnOffLight()` 方法来调用这些命令。

我们将首先创建一个名为 `Command` 的协议，所有我们的命令都需要遵守。以下是 `Command` 协议：

```swift
protocol Command {
  func execute()
}
```

此协议包含一个名为 `execute` 的方法，它将用于执行命令。现在，让我们看看 `Light` 类将用于打开和关闭灯光的 `LightOneCommand` 和 `LightOffCommand` 类。它们如下所示：

```swift
struct RockerSwitchLightOnCommand: Command {
  func execute() {
    print("Rocker Switch:  Turning Light On")
  }
}

struct RockerSwitchLightOffCommand: Command {
  func execute() {
    print("Rocker Switch:  Turning Light Off")
  }
}
struct PullSwitchLightOnCommand: Command {
  func execute() {
    print("Pull Switch:  Turning Light On")
  }
}

struct PullSwitchLightOffCommand: Command {
  func execute() {
    print("Pull Switch:  Turning Light Off")
  }
}
```

`RockerSwitchLightOffCommand`、`RockerSwitchLightOnCommand`、`PullSwitchLightOnCommand` 和 `PullSwitchLightOffCommand` 命令通过实现 `execute()` 方法符合 `Command` 协议，因此我们可以在我们的 `Light` 类中使用它们。现在，让我们看看如何实现 `Light` 类：

```swift
class Light {
  var lightOnCommand: Command
  var lightOffCommand: Command

  init(lightOnCommand: Command, lightOffCommand: Command) {
    self.lightOnCommand = lightOnCommand
    self.lightOffCommand = lightOffCommand
  }

  func turnOnLight() {
    self.lightOnCommand.execute()
  }

  func turnOffLight() {
    self.lightOffCommand.execute()
  }
}
```

在 `Light` 类中，我们首先创建两个名为 `lightOnCommand` 和 `lightOffCommand` 的变量，它们持有符合 `Command` 协议的类的实例。然后我们创建一个初始化器，它允许我们在初始化类时设置这两个命令。最后，我们创建 `turnOnLight()` 和 `turnOffLight()` 方法，我们将使用这些方法来打开和关闭灯光。在这些方法中，我们调用适当的命令来打开或关闭灯光。

我们将像这样使用 `Light` 类：

```swift
var on = PullSwitchLightOnCommand()
var off = PullSwitchLightOffCommand()
var light = Light(lightOnCommand: on, lightOffCommand: off)

light.turnOnLight()
light.turnOffLight()

light.lightOnCommand = RockerSwitchLightOnCommand()
light.turnOnLight()
```

在这个例子中，我们首先创建了一个名为 `on` 的 `PullSwitchLightOnCommand` 类的实例和一个名为 `off` 的 `PullSwitchLightOffCommand` 类的实例。然后我们使用我们刚刚创建的两个命令创建了一个 `Light` 类的实例，并调用 `Light` 实例的 `turnOnLight()` 和 `turnOffLight()` 方法来打开和关闭我们的灯光。在最后两行中，我们将 `lightOnCommand` 方法从最初设置为 `PullSwitchLightOnCommand` 类的实例更改为 `RockerSwitchLightOnCommand` 类的实例。当我们将灯光打开时，`light` 实例将使用 `RockerSwitchLightOnCommand` 类。这允许我们在运行时更改 `Light` 类的功能。

使用命令模式有许多好处。其中一个主要的好处是我们能够在运行时设置命令的实现，这也允许我们根据需要在整个应用程序的生命周期中用符合`Command`协议的不同实现替换命令。命令模式的另一个优点是将命令实现的细节封装在命令类本身中，而不是在容器类中。

### 注意

对于命令模式，我们使用结构体来实现我们的命令类型，因为创建一个命令类型的实例并将其用于创建`Light`类的多个实例非常容易。在这种情况下，如果`Light`类中的任何内容在命令实例中发生变化，那么它将反映在所有使用该命令实例的`Light`类的实例中。通常，我们不想这种行为；然而，如果您的应用程序需要这种行为，那么您应该使用类而不是结构体。

现在，让我们看看我们的最后一个设计模式，即策略模式。

## 策略模式

策略模式在某种程度上与命令模式相似，因为它们都允许我们将实现细节从我们的调用类中解耦，并允许我们在运行时切换实现。最大的区别是，策略模式旨在封装算法。通过替换算法，我们期望对象执行相同的功能，但以不同的方式。在命令模式中，当我们替换命令时，我们期望对象以不同的方式工作。

在本节中，我们将通过展示如何在运行时替换压缩策略来演示策略模式。让我们从这个示例开始，创建一个`CompressionStrategy`协议，我们的每个压缩类都将遵循这个协议。让我们看一下以下代码：

```swift
protocol CompressionStrategy {
  func compressFiles(filePaths: [String])
}
```

此协议定义了一个名为`compressFiles()`的方法，它接受一个参数，即包含要压缩的文件路径的字符串数组。现在，我们将创建两个符合`CompressionStrategy`协议的结构。这些类是`ZipCompressionStrategy`和`RarCompressionStrategy`类，如下所示：

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

这两个结构体通过具有名为`compressFiles()`的方法来实现`CompressionStrategy`协议，该方法接受一个字符串数组。在这些方法中，我们只是简单地打印出我们正在使用的压缩名称。通常，我们会在这些方法中实现压缩逻辑。

现在，让我们看看将被调用来压缩文件的`CompressContent`类：

```swift
class CompressContent {
  var strategy: CompressionStrategy

  init(strategy: CompressionStrategy) {
    self.strategy = strategy
  }

  func compressFiles(filePaths: [String]) {
    self.strategy.compressFiles(filePaths)
  }
}
```

在这个课程中，我们首先定义了一个名为 `strategy` 的变量，它将包含一个符合 `CompressStrategy` 协议的类的实例。然后我们创建了一个初始化器，用于在类初始化时设置压缩类型。最后，我们创建了一个名为 `compressFiles()` 的方法，它接受一个包含我们希望压缩的文件路径的字符串数组。在这个方法中，我们使用在 `strategy` 变量中设置的压缩策略来压缩文件。

我们将像这样使用 `CompressContent` 类：

```swift
var filePaths = ["file1.txt", "file2.txt"]
var zip = ZipCompressionStrategy()
var rar = RarCompressionStrategy()

var compress = CompressContent(strategy: zip)
compress.compressFiles(filePaths)

compress.strategy = rar
compress.compressFiles(filePaths)
```

我们首先创建一个包含我们希望压缩的文件的字符串数组。我们还创建了 `ZipCompressionStrategy` 和 `RarCompressionStrategy` 类的实例。然后我们创建了一个 `CompressContent` 类的实例，将压缩策略设置为 `ZipCompressionStrategy` 实例，并调用 `compressFiles()` 方法，该方法将在控制台打印出 `Using zip compression` 消息。然后我们将压缩策略设置为 `RarCompressionStrategy` 实例，并再次调用 `compressFiles()` 方法，该方法将在控制台打印出 `Using rar compression` 消息。

策略模式非常适合在运行时设置要使用的算法，这也允许我们根据应用程序的需要交换不同的实现。策略模式的另一个优点是，我们将算法的细节封装在策略类本身中，而不是在主实现类中。

### 注意

就像命令模式一样，我们使用结构体来实现策略模式，因为它非常容易创建一个策略类型的实例，然后使用它来创建 `CompressContent` 类的多个实例。在这种情况下，如果策略实例中发生了任何变化，它将反映在所有的 `CompressContent` 类型中。通常，我们并不希望这种行为；然而，如果您的应用程序需要这种行为，那么您应该使用类而不是结构体。

这标志着我们对 Swift 中设计模式的探索结束。

# 摘要

设计模式是解决我们在现实世界的应用程序设计中反复看到的软件设计问题的解决方案。这些模式旨在帮助我们创建可重用和灵活的代码。设计模式还可以使我们的代码更容易被其他开发人员以及我们自己阅读和理解，当我们几个月/几年后回顾我们的代码时。

如果我们仔细查看本章中的示例，我们会注意到设计模式的一个关键组成部分是协议。几乎所有的设计模式（单例设计模式除外）都使用协议来帮助我们创建非常灵活和可重用的代码。

如果这是你第一次真正关注设计模式，你可能注意到了一些与你在过去自己的代码中可能使用过的策略的相似之处。当经验丰富的开发者第一次接触设计模式时，这种情况是预料之中的。我还会鼓励你更多地阅读有关设计模式的内容，因为它们肯定会帮助你创建更灵活和可重用的代码。
