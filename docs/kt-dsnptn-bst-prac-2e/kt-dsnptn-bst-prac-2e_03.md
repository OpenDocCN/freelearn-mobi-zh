# 第二章：*第二章*：使用创建型模式

在本章中，我们将介绍如何使用**Kotlin**实现经典**创建型模式**。这些模式处理**如何**和**何时**创建你的对象。对于每个设计模式，我们将讨论它旨在实现的目标以及 Kotlin 如何满足这些需求。

我们在本章中将涵盖以下主题：

+   单例

+   工厂方法

+   抽象工厂

+   构建者

+   原型

精通这些设计模式将使你能够更好地管理你的对象，更好地适应变化，并编写易于维护的代码。

# 技术要求

对于本章，你需要安装以下内容：

+   **IntelliJ IDEA** **社区版**([`www.jetbrains.com/idea/download/`](https://www.jetbrains.com/idea/download/))

+   **OpenJDK** **11**（或更高版本）([`openjdk.java.net/install/`](https://openjdk.java.net/install/))

你可以在**GitHub**上找到本章的代码文件，地址为[`github.com/PacktPublishing/Kotlin-Design-Patterns-and-Best-Practices/tree/main/Chapter02`](https://github.com/PacktPublishing/Kotlin-Design-Patterns-and-Best-Practices/tree/main/Chapter02)。

# 单例

**单例**——镇上最受欢迎的单身汉。每个人都认识他，每个人都谈论他，每个人都知道在哪里找到他。

即使不喜欢使用设计模式的人也会知道单例模式。在某个时候，它甚至被宣称为**反模式**，但这仅仅是因为它的广泛流行。

*那么，对于那些第一次遇到它的人来说，这种设计模式到底是什么呢？*

通常，如果你有一个类，你可以创建尽可能多的实例。例如，假设我们都要求列出我们最喜欢的电影：

```java
val myFavoriteMovies = listOf("Black Hawk Down", "Blade   Runner")
```

```java
val yourFavoriteMovies = listOf(...)
```

注意，我们可以创建尽可能多的`List`实例，而且这没有问题。大多数类都可以有多个实例。

*接下来，如果我们俩都想列出《快速而愤怒》系列中的最佳电影呢？*

```java
val myFavoriteQuickAndAngryMovies = listOf()
```

```java
val yourFavoriteQuickAndAngryMovies = listOf()
```

注意，这两个列表完全相同，因为它们都是空的。而且它们会保持空的状态，因为它们是不可变的，而且《快速而愤怒》系列电影真的很糟糕。我希望你会同意这一点。

由于这两个类的实例完全相同，根据**equals 方法**，将它们多次保存在内存中并没有太多意义。如果所有对空列表的引用都指向同一个对象实例，那就太好了。实际上，如果你这么想的话，所有 null 都是相同的。

这就是单例设计模式背后的主要思想。

单例设计模式有几个要求：

+   我们系统中应该只有一个实例。

+   这个实例应该可以从我们系统的任何部分访问。

在`private`类中。然后，你还需要确保实例化最好是懒加载的、线程安全的，并且性能良好，以下是一些要求：

+   **延迟加载**：我们可能不想在程序启动时实例化单例对象，因为这可能是一个昂贵的操作。我们希望只在第一次需要时才实例化它。

+   **线程安全**：如果有两个线程试图同时实例化一个单例对象，它们都应该接收到相同的实例，而不是两个不同的实例。如果您不熟悉这个概念，我们将在 *第五章* *介绍函数式编程* 中介绍它。

+   **高效性**：如果有许多线程试图同时实例化一个单例对象，我们不应该长时间阻塞它们，因为这会阻止它们的执行。

在 Java 或 **C++** 中满足所有这些要求相当困难，或者至少非常冗长。

Kotlin 通过引入一个名为 `object` 的关键字使创建单例变得简单。您可能从 **Scala** 中认识这个关键字。通过使用这个关键字，我们将得到一个单例对象的实现，它满足我们所有的要求。

重要提示：

`object` 关键字不仅用于创建单例，我们将在本章后面深入讨论这一点。

我们声明对象就像一个普通类一样，但没有构造函数，因为单例对象不能由我们实例化：

```java
object NoMoviesList
```

从现在起，我们可以在代码的任何地方访问 `NoMoviesList`，并且它将只有一个实例：

```java
val myFavoriteQuickAndAngryMovies = NoMoviesList
```

```java
val yourFavoriteQuickAndAngryMovies = NoMoviesList
```

```java
println(myFavoriteQuickAndAngryMovies === 
```

```java
    yourFavoriteQuickAndAngryMovies) // true
```

注意到参照性等号检查两个变量是否指向内存中的同一个对象。*这真的是一个列表吗？*

让我们创建一个打印我们电影列表的函数：

```java
fun printMovies(movies: List<String>) {
```

```java
    for (m in movies) {
```

```java
        println(m)
```

```java
    }
```

```java
}
```

当我们传递一个初始电影列表时，代码可以正常编译：

```java
// Prints each movie on a newline    
```

```java
printMovies(myFavoriteMovies) 
```

但如果我们传递一个空的电影列表给它，代码将无法编译：

```java
printMovies(myFavoriteQuickAndAngryMovies) 
```

```java
// Type mismatch: inferred type is NoMoviesList but // List<String> was expected
```

原因在于我们的函数只接受 *字符串列表* 类型的参数，而没有任何东西告诉函数 `NoMoviesList` 是这种类型（尽管它的名字暗示了这一点）。

幸运的是，在 Kotlin 中，单例对象可以实现接口，并且有一个通用的 `List` 接口可用：

```java
object NoMoviesList : List<String>
```

现在，我们的编译器将提示我们实现所需的函数。我们将通过为 `object` 添加一个主体来实现这一点：

```java
object NoMoviesList : List<String> {
```

```java
    override val size = 0
```

```java
    override fun contains(element: String) = false 
```

```java
    ... /
```

```java
}
```

如果您愿意，我们可以将其他函数的实现留给您。这将是对您至今为止所学的 Kotlin 知识的良好练习。然而，您不必这样做。Kotlin 已经提供了一个创建任何类型空列表的函数：

```java
printMovies(emptyList())
```

如果您好奇，这个函数返回一个实现 `List` 的单例对象。您可以使用 IntelliJ IDEA 或在 GitHub 上查看其完整实现（[`github.com/JetBrains/kotlin/blob/master/libraries/stdlib/src/kotlin/collections/Collections.kt`](https://github.com/JetBrains/kotlin/blob/master/libraries/stdlib/src/kotlin/collections/Collections.kt)）。这是一个设计模式在现代软件中仍然积极应用的优秀例子。

Kotlin 的`object`与类有一个主要区别——它不能有构造函数。如果你需要为你的 Singleton 实现初始化，例如第一次从配置文件加载数据，你可以使用`init`块代替：

```java
object Logger {
```

```java
    init {
```

```java
        println("I was accessed for the first time")
```

```java
        // Initialization logic goes here
```

```java
    }
```

```java
    // More code goes here
```

```java
}
```

注意，如果 Singleton 从未被调用，它根本不会运行其初始化逻辑，从而节省资源。这被称为**延迟初始化**。

现在我们已经学会了如何限制对象的创建，让我们讨论如何在不直接使用构造函数的情况下创建对象。

# 工厂方法

**工厂方法**设计模式完全是关于创建对象。

*但为什么我们需要一个创建对象的方法？构造函数不是用来这个的吗？*

好吧，构造函数有其局限性。

例如，假设我们正在构建一个棋盘游戏。我们希望允许我们的玩家将游戏状态保存到文本文件中，然后从该位置恢复游戏。

由于棋盘的大小是预先确定的，我们只需要记录每个棋子的位置和类型。我们将使用代数符号来表示这一点——例如，位于 C3 的皇后棋子将被存储在我们的文件中为`qc3`，位于 A8 的兵棋子将被存储为`pa8`，以此类推。

假设我们已将此文件读入一个字符串列表（顺便说一句，这将是一个很好的 Singleton 设计模式的早期讨论的应用）。

给定符号列表，我们希望用它们填充我们的棋盘：

```java
// More pieces here
```

```java
val notations = listOf("pa8", "qc3", ...)
```

```java
val pieces = mutableListOf<ChessPiece>()
```

```java
for (n in notations) {
```

```java
    pieces.add(createPiece(n))
```

```java
}
```

```java
println(pieces)
```

在我们可以实现我们的`createPiece`函数之前，我们需要决定所有棋子共有的东西。我们将为此创建一个接口：

```java
interface ChessPiece {
```

```java
    val file: Char
```

```java
    val rank: Char
```

```java
}
```

注意，Kotlin 中的接口可以声明属性，这是一个非常强大的功能。

每个棋子都将是一个实现我们接口的`data class`：

```java
data class Pawn(
```

```java
    override val file: Char,
```

```java
    override val rank: Char
```

```java
) : ChessPiece
```

```java
data class Queen(
```

```java
    override val file: Char,
```

```java
    override val rank: Char
```

```java
) : ChessPiece
```

其他棋子的实现留给你作为练习来完成。

现在，剩下的就是实现我们的`createPiece`函数：

```java
fun createPiece(notation: String): ChessPiece {
```

```java
    val (type, file, rank) = notation.toCharArray()
```

```java
    return when (type) {
```

```java
        'q' -> Queen(file, rank)
```

```java
        'p' -> Pawn(file, rank)
```

```java
        // ...
```

```java
        else -> throw RuntimeException("Unknown piece: $type")
```

```java
    }
```

```java
}
```

在我们可以讨论这个函数实现了什么之前，让我们先介绍三个我们之前没有见过的新的语法元素。

首先，`toCharArray`函数将字符串分割成一个字符数组。由于我们假设所有的符号都是三个字符长，`0`位置的元素将代表棋子的*类型*，`1`位置的元素将代表其垂直列——也称为`file`，最后一个元素将代表其水平列——也称为`rank`。

接下来，我们可以看到三个值：`type`、`file`和`rank`，它们被括号包围。这被称为`data class`可以解构。

之前的代码示例类似于以下更冗长的代码：

```java
val type = notation.toCharArray()[0]
```

```java
val file = notation.toCharArray()[1]
```

```java
val rank = notation.toCharArray()[2]
```

现在，让我们专注于`when`表达式。根据表示类型的字母，它实例化`ChessPiece`接口的一个实现。记住，这正是工厂方法设计模式的核心。

为了确保你很好地掌握这个设计模式，请随意将其他棋子的类和逻辑实现作为练习。

最后，让我们看看函数的底部，在那里我们看到第一个`throw`表达式的使用。

这个表达式，正如其名所示，*抛出*一个异常，这将停止我们简单程序的正常执行。我们将在*第五章*，“介绍函数式编程”中讨论如何处理异常。

在现实世界中，工厂方法设计模式通常被需要将配置文件（无论是 XML、JSON 还是 YAML 格式）解析为运行时对象的库所使用。

## 静态工厂方法

有一个类似命名的模式（实现略有不同），它经常与工厂方法设计模式混淆，并在*四人帮*的书中描述——这是**静态工厂方法**设计模式。

静态工厂方法设计模式是由 Joshua Bloch 在他的书《*Effective Java*》中推广的。为了更好地理解这一点，让我们看看 Java 标准库中的几个例子：`valueOf()`方法。从字符串构建`Long`（即 64 位整数）至少有两种方式：

```java
Long l1 = new Long("1"); // constructor
```

```java
Long l2 = Long.valueOf("1"); // static factory method
```

构造函数和`valueOf()`方法都接收字符串作为输入，并产生`Long`作为输出。

*那么，为什么我们应该更喜欢静态工厂方法设计模式而不是简单的构造函数呢？*

与构造函数相比，使用静态工厂方法的一些优点如下：

+   它提供了显式命名不同对象构造函数的机会。当你的类有多个构造函数时，这特别有用。

+   我们通常不期望构造函数抛出异常。这并不意味着类的实例化不能失败。另一方面，常规方法的异常则更被接受。

+   说到期望，我们期望构造函数运行得快。但某些对象的构建本质上可能很慢。考虑使用静态工厂方法。

这些主要是风格上的优势；然而，这种方法也有技术上的优势。

### 缓存

静态工厂方法设计模式可能提供的`Long`实际上确实如此。而不是总是为任何值返回一个新实例，`valueOf()`会在缓存中检查此值是否已经被解析。如果是，它将返回缓存的实例。重复使用相同的值调用静态工厂方法可能产生的垃圾回收比始终使用构造函数要少。

### 子类化

当调用构造函数时，我们总是实例化我们指定的类。另一方面，调用静态工厂方法限制较少，可能产生类的实例或其子类之一。在讨论了在 Kotlin 中实现此设计模式之后，我们将讨论这一点。

### Kotlin 中的静态工厂方法

我们在本章的*单例*部分之前讨论了`object`关键字。现在，我们将看到它作为**伴随对象**的另一种用途。

在 Java 中，静态工厂方法被声明为`static`。但在 Kotlin 中，没有这样的关键字。相反，不属于类实例的方法可以声明在`companion object`内部：

```java
class Server(port: Long) {
```

```java
    init {
```

```java
        println("Server started on port $port")
```

```java
    }
```

```java
    companion object {
```

```java
        fun withPort(port: Long) = Server(port)
```

```java
  }
```

```java
}
```

重要提示：

伴随对象可以有名称——例如，`companion object`解析器。但这只是为了提供关于对象目标的清晰度。

如您所见，这次，我们声明了一个以`companion`关键字为前缀的对象。它位于类内部，而不是像我们在单例设计模式中看到的那样位于包级别。

这个对象有自己的方法，您可能会想知道这有什么好处。就像 Java 静态方法一样，当第一次访问包含的类时，会惰性实例化`companion` `object`：

```java
Server.withPort(8080) // Server started on port 8080
```

此外，在类的实例上调用它根本不起作用，与 Java 不同：

```java
Server(8080) // Won't compile, constructor is private
```

重要提示：

一个类可能只有一个`companion object`。

有时候，我们也希望静态工厂方法是实例化我们的对象的唯一方式。为了做到这一点，我们可以将对象的默认构造函数声明为`private`：

```java
class Server private constructor(port: Long) {
```

```java
    ...
```

```java
}
```

这意味着现在构建我们类实例的唯一方式是通过我们的静态工厂方法：

```java
val server = Server(8080))  // Doesn't compile
```

```java
val server = Server.withPort(8080) // Works!
```

现在我们来讨论另一个经常与工厂方法混淆的设计模式——抽象工厂。

# 抽象工厂

**抽象工厂**是一个被广泛误解的设计模式。它因非常复杂和奇特而臭名昭著。实际上，它相当简单。如果您理解了工厂方法设计模式，您会立刻理解这个。这是因为抽象工厂设计模式是工厂的工厂。仅此而已。*工厂*是一个能够创建其他类的函数或类。换句话说，抽象工厂是一个封装多个工厂方法的类。

您可能已经理解了这一点，但仍会想知道这种设计模式有什么用途。在现实世界中，抽象工厂设计模式通常用于从文件中获取配置的框架和库。**Spring 框架**就是这些中的一个例子。

为了更好地理解设计模式的工作原理，让我们假设我们有一个用 YAML 文件编写的服务器配置：

```java
server: 
```

```java
    port: 8080
```

```java
environment: production
```

我们的任务是从这个配置中构建对象。

在上一节中，我们讨论了如何使用工厂方法从同一系列中构建对象。但在这里，我们有两个相互关联但不是*兄弟姐妹*的对象系列。

首先，让我们将它们描述为接口：

```java
interface Property {
```

```java
    val name: String
```

```java
    val value: Any
```

```java
}
```

我们将返回一个接口而不是`data class`。您将在本节后面看到这如何帮助我们：

```java
interface ServerConfiguration {
```

```java
    val properties: List<Property>
```

```java
}
```

然后，我们可以提供基本实现供以后使用：

```java
data class PropertyImpl(
```

```java
    override val name: String,
```

```java
    override val value: Any
```

```java
) : Property
```

```java
data class ServerConfigurationImpl(
```

```java
    override val properties: List<Property>
```

```java
) : ServerConfiguration
```

服务器配置仅包含属性列表——而*属性*是由一个`name`对象和一个`value`对象组成的对。

这是我们第一次看到 `Any` 类型被使用。`Any` 类型是 Kotlin 对 Java 的 `object` 的版本，但有一个重要的区别：它不能为 null。

现在，让我们编写我们的第一个工厂方法，它将根据给定的字符串创建 `Property`：

```java
fun property(prop: String): Property {
```

```java
    val (name, value) = prop.split(":")
```

```java
    return when (name) {
```

```java
        "port" -> PropertyImpl(name, value.trim().toInt())
```

```java
        "environment" -> PropertyImpl(name, value.trim())
```

```java
        else -> throw RuntimeException("Unknown property:           $name")
```

```java
    }
```

```java
}
```

与许多其他语言一样，`trim()` 是一个在字符串上声明的函数，用于删除字符串中的任何空格。现在，让我们创建两个属性来表示我们的服务的端口 (`port`) 和环境 (`environment`)：

```java
val portProperty = property("port: 8080")
```

```java
val environment = property("environment: production") 
```

这段代码有一个小问题。为了理解它是什么，让我们尝试将 `port` 属性的值存储到另一个变量中：

```java
val port: Int = portProperty.value
```

```java
// Type mismatch: inferred type is Any but Int was expected
```

我们已经在工厂方法中确保 `port` 被解析为 `Int`。但现在，由于值的类型被声明为 `Any`，这个信息丢失了。它可以是一个 `String`、`Int` 或任何其他类型。我们需要一个新的工具来解决这个问题，所以让我们短暂地偏离一下，讨论 Kotlin 中的转换。

## 转换

在类型语言中，**转换**是一种尝试强制编译器使用我们指定的类型，而不是它推断出的类型。如果我们确定值的类型，我们可以在它上面使用一个 *不安全的* 转换：

```java
val port: Int = portProperty.value as Int
```

它被称为 *不安全的*，是因为如果值不是我们预期的类型，我们的程序将崩溃，而编译器无法警告我们。

或者，我们可以使用 *安全的* 转换：

```java
val port: Int? = portProperty.value as? Int
```

*安全的*转换不会使我们的程序崩溃，但如果对象的类型不是我们预期的，它将返回 null。注意，我们的 `port` 变量现在被声明为可空的 `Int` 类型，因此在编译时我们必须显式处理可能得不到我们想要的结果的情况。

## 继承

而不是求助于转换，让我们尝试另一种方法。我们不会使用一个具有 `Any` 类型值的单个实现，而是使用两个独立的实现：

```java
data class IntProperty(
```

```java
    override val name: String,
```

```java
    override val value: Int
```

```java
) : Property
```

```java
data class StringProperty(
```

```java
    override val name: String,
```

```java
    override val value: String
```

```java
) : Property
```

我们的工厂方法需要稍作修改才能返回这两个类中的一个：

```java
fun property(prop: String): Property {
```

```java
    val (name, value) = prop.split(":")
```

```java
    return when (name) {
```

```java
        "port" -> IntProperty(name, value.trim().toInt())
```

```java
        "environment" -> StringProperty(name, value.trim())
```

```java
        else -> throw RuntimeException("Unknown property:           $name")
```

```java
    }
```

```java
}
```

这看起来不错，但如果我们再次尝试编译我们的代码，它仍然不会工作：

```java
val portProperty = Parser.property("port: 8080")
```

```java
val port: Int = portProperty.value 
```

虽然我们现在有两个具体的类，但编译器不知道解析的属性是 `IntProperty` 还是 `StringProperty`。它只知道它是 `Property`，并且值的类型仍然是 `Any`：

```java
> Type mismatch: inferred type is Any but Int was expected
```

我们需要另一个技巧，这个技巧被称为 **智能转换**。

## 智能转换

我们可以使用 `is` 关键字来检查一个对象是否为给定的类型：

```java
println(portProperty is IntProperty) // true
```

然而，Kotlin 编译器非常智能。*如果我们在一个* `if` *表达式中执行类型检查，这意味着* `portProperty` *确实是* `IntProperty`*，对吧？* 因此，它可以安全地进行转换。

Kotlin 编译器会为我们做这件事：

```java
if (portProperty is IntProperty) {
```

```java
    val port: Int = portProperty.value // works!
```

```java
}
```

现在不再有编译错误，我们也不必处理可空值。

智能转换也适用于 null。在 Kotlin 的类型层次结构中，不可为 null 的 `Int` 类型是可空类型 `Int?` 的子类，这对于所有类型都是真的。之前，我们提到，如果 *安全的* 转换失败，它将返回 `null`：

```java
val port: Int? = portProperty.value as? Int
```

我们可以检查 `port` 是否为 null，如果不是，它将智能地转换为非可空类型：

```java
if (port != null) {
```

```java
    val port: Int = port
```

```java
}
```

*太好了！* *但是等等，这段代码中发生了什么？*

在上一章中，我们提到值不能被重新赋值。但在这里，我们定义了两次 `port` 值。*这是怎么可能的？* 这不是一个错误，而是 Kotlin 的另一个特性，称为 **变量遮蔽**。

## 变量遮蔽

首先，让我们考虑如果没有遮蔽，我们的代码会是什么样子。我们必须声明两个不同名称的变量：

```java
val portOrNull: Int? = portProperty.value as? Int
```

```java
if (portOrNull != null) {
```

```java
    val port: Int = portOrNull // works
```

```java
}
```

然而，这造成了浪费，原因有两个。首先，变量名变得相当冗长。其次，`portOrNull` 变量很可能在此之后就不会再被使用了，因为 null 本身并不是一个非常有用的值。相反，我们可以在不同的作用域中声明具有相同名称的值，这些作用域由花括号（`{}`）表示。

请注意，变量遮蔽可能会让你感到困惑，并且它本质上是有缺陷的。然而，重要的是要意识到它的存在，但建议尽可能明确地命名你的变量。

## 工厂方法集合

既然我们已经对类型转换和变量遮蔽有了了解，让我们回到之前的代码示例，并实现第二个工厂方法，该方法将创建一个 `server` 配置对象：

```java
fun server(propertyStrings: List<String>): 
```

```java
  ServerConfiguration {
```

```java
    val parsedProperties = mutableListOf<Property>()
```

```java
    for (p in propertyStrings) {
```

```java
        parsedProperties += property(p)
```

```java
    }
```

```java
    return ServerConfigurationImpl(parsedProperties)
```

```java
}
```

此方法将配置文件中的行转换为 `Property` 对象，使用的是我们之前已经实现的 `property()` 工厂方法。

我们可以测试我们的第二个工厂方法是否正常工作：

```java
println(server(listOf("port: 8080", "environment: 
```

```java
  production")))
```

```java
> ServerConfigurationImpl(properties=[IntProperty(name=port, value=8080), StringProperty(name=environment, value=production)])
```

由于这两个方法相关联，将它们放在同一个类下会更好。让我们称这个类为 `Parser`。尽管我们还没有解析任何实际的文件，并且已经同意我们可以逐行获取其内容，但到这本书的结尾，你可能会同意实现实际的读取逻辑相当简单。

我们还可以使用静态工厂方法和我们在上一节中学到的 `companion object` 语法。

结果实现将看起来像这样：

```java
class Parser {
```

```java
    companion object {
```

```java
        fun property(prop: String): Property {
```

```java
           ...
```

```java
        }
```

```java
        fun server(propertyStrings: List<String>): ...{
```

```java
           ...        
```

```java
        }
```

```java
    }
```

```java
}
```

这种模式允许我们创建 *家族* 的对象——在这种情况下，`ServerConfig` 是属性的一个 *父类*。

之前的代码只是实现抽象工厂的一种方式。你可能会发现一些实现依赖于实现接口：

```java
interface Parser {
```

```java
    fun property(prop: String): Property
```

```java
    fun server(propertyStrings: List<String>):       ServerConfiguration
```

```java
}
```

```java
class YamlParser : Parser {
```

```java
    // Implementation specific to YAML files
```

```java
}
```

```java
class JsonParser : Parser {
```

```java
    // Implementation specific to JSON files
```

```java
}
```

如果你的工厂方法变得很长，这种方法可能更好。

你可能还有一个问题，那就是在哪里可以看到实际代码中使用的抽象工厂。一个例子是 `java.util.Collections` 类。它有 `emptyMap`、`emptyList` 和 `emptySet` 等方法，每个方法都生成一个不同的类。然而，它们共同的特点是它们都是集合。

# 构建器

有时，我们的对象非常简单，只有一个构造函数，无论是空的还是非空的。但有时，它们的创建非常复杂，基于很多参数。我们已经看到了一个提供**更好的构造函数**的模式——静态工厂方法设计模式。现在，我们将讨论**Builder**设计模式，它将帮助我们创建复杂对象。

作为这样一个对象的例子，想象我们需要设计一个发送电子邮件的系统。我们不会实现发送它们的实际机制，我们只是设计一个代表它的类。

一封电子邮件可能具有以下属性：

+   地址（至少一个必填）

+   CC（可选）

+   标题（可选）

+   正文（可选）

+   重要标志（可选）

我们可以在我们的系统中将电子邮件描述为一个`data class`：

```java
data class Mail_V1(
```

```java
    val to: List<String>,
```

```java
    val cc: List<String>?,
```

```java
    val title: String?,
```

```java
    val message: String?,
```

```java
    val important: Boolean,
```

```java
)
```

重要提示：

看一下前面代码中最后一个参数的定义。这个逗号不是打字错误。它被称为**尾随逗号**，这些是在**Kotlin 1.4**中引入的。这样做是为了你可以轻松地改变参数的顺序。

接下来，让我们尝试创建一封致我们经理的电子邮件：

```java
val mail = Mail_V1(
```

```java
    listOf("manager@company.com"),    // To
```

```java
    null,                             // CC
```

```java
    "Ping ",                          // Title
```

```java
    null,                             // Message,
```

```java
    true))                            // Important
```

注意，我们已将*carbon copy*（这就是`CC`所代表的意思）定义为可空的，这样它就可以接收电子邮件列表或 null。另一个选项是将它定义为`List<String>`并强制我们的代码传递`listOf()`。

由于我们的构造函数接收了大量的参数，我们不得不添加一些注释以避免混淆。

*但是，如果我们现在需要更改这个类会怎样呢？*

首先，我们的代码将停止编译。其次，我们需要跟踪注释。简而言之，具有长参数列表的构造函数很快就会变得混乱。

这正是 Builder 设计模式试图解决的问题。它将参数的分配与对象的创建解耦，并允许逐步创建复杂对象。在本节中，我们将看到解决这个问题的多种方法。

让我们先创建一个新的类，`MailBuilder`，它将包装我们的`Mail`类：

```java
class MailBuilder {
```

```java
    private var to: List<String> = listOf()
```

```java
    private var cc: List<String> = listOf()
```

```java
    private var title: String = ""
```

```java
    private var message: String = ""
```

```java
    private var important: Boolean = false
```

```java
    class Mail internal constructor(
```

```java
        val to: List<String>,
```

```java
        val cc: List<String>?,
```

```java
        val title: String?,
```

```java
        val message: String?,
```

```java
        val important: Boolean
```

```java
    )
```

```java
    ... // More code will come here soon
```

```java
}
```

我们的构建器具有与我们的结果类完全相同的属性。但这些属性都是可变的。

注意，构造函数使用`internal`可见性修饰符标记。这意味着我们的`Mail`类将对我们模块内的任何代码都是可访问的。

为了最终创建我们的类，我们将引入`build()`函数：

```java
fun build(): Mail {
```

```java
    if (to.isEmpty()) {
```

```java
        throw RuntimeException("To property is empty")
```

```java
    }
```

```java
    return Mail(to, cc, title, message, important)
```

```java
}
```

对于每个属性，我们还需要另一个函数来设置它：

```java
fun message(message: String): MailBuilder {
```

```java
    this.message = message
```

```java
    return this
```

```java
}
```

```java
// More functions for each of the properties
```

现在，我们可以使用我们的构建器以以下方式创建一个电子邮件：

```java
val email = MailBuilder("hello@hello.com").title("What's   up?").build()
```

在设置新值后，我们通过使用`this`返回对对象的引用，这为我们提供了访问下一个 setter 的权限，以便我们可以执行链式操作（请参阅本章的*流畅设置器*部分以了解解释）。

这是一个有效的方法。但它有几个缺点：

+   我们结果类的属性必须在构建器内部重复。

+   对于每个属性，我们需要声明一个函数来设置其值。

Kotlin 提供了两种其他方法，你可能觉得它们更有用。

## 流畅设置器

使用 `data class` 构造函数的方法将仅包含必填字段。所有其他字段将变为 `private`，我们将为这些字段提供设置器：

```java
data class Mail_V2(
```

```java
    val to: List<String>,
```

```java
    private var _message: String? = null,
```

```java
    private var _cc: List<String>? = null,
```

```java
    private var _title: String? = null,
```

```java
    private var _important: Boolean? = null
```

```java
) {
```

```java
    fun message(message: String) = apply {
```

```java
        _message = message
```

```java
    }
```

```java
    // Pattern repeats for every other field
```

```java
    //...
```

```java
}
```

重要提示：

在 Kotlin 中，使用下划线为 `private` 变量是常见的约定。这允许我们避免重复 `this.message = message` 和像 `message = message` 这样的错误。

在这个代码示例中，我们使用了 `apply` 函数。这是可以调用在每个 Kotlin 对象上的作用域函数系列的一部分，我们将在 *第九章*，*惯用和反模式* 中详细讨论它们。`apply` 函数在执行代码块后返回对象的引用。因此，它是上一个示例中设置器函数的简短版本：

```java
fun message(message: String): MailBuilder {
```

```java
    this.message = message
```

```java
    return this
```

```java
}
```

这为我们提供了与上一个示例相同的 API：

```java
val mailV2 = Mail_V2(listOf("manager@company.com")).message("Ping")
```

然而，我们可能根本不需要设置器。相反，我们可以使用之前讨论过的 `apply()` 函数在对象本身上。这是 Kotlin 中每个对象都有的扩展函数之一。这种方法仅当所有可选字段都是 `*变量*` 而不是 `*值*` 时才有效。

然后，我们可以这样创建我们的电子邮件：

```java
val mail = Mail_V2("hello@mail.com").apply {
```

```java
    message = "Something" 
```

```java
    title = "Apply"
```

```java
}
```

这是一个很好的方法，它需要更少的代码来实现。然而，这种方法也有一些缺点：

+   我们不得不将所有可选参数变为可变的。应始终优先考虑不可变字段，因为它们是线程安全的，并且更容易推理。

+   我们的所有可选参数也都是可空的。Kotlin 是一个空安全语言，所以每次我们访问它们时，我们首先必须检查它们的值是否已设置。

+   这种语法非常冗长。对于每个字段，我们需要一次又一次地重复相同的模式。

现在，让我们讨论这个问题的最后一种方法。

## 默认参数

在 Kotlin 中，我们可以为构造函数和函数参数指定默认值：

```java
data class Mail_V3(
```

```java
    val to: List<String>,
```

```java
    val cc: List<String> = listOf(),
```

```java
    val title: String = "",
```

```java
    val message: String = "",
```

```java
    val important: Boolean = false
```

```java
)
```

默认参数使用类型后面的 `=` 运算符设置。这意味着尽管我们的构造函数仍然有所有参数，但我们不需要提供它们。

所以，如果你想创建一个没有正文的电子邮件，你可以这样做：

```java
val mail = Mail_V3(listOf("manager@company.com"), listOf(), "Ping")
```

然而，请注意，我们必须通过提供一个空列表来指定我们不想在 CC 字段中包含任何人，这有点不方便。

*如果我们只想发送一个标记为重要的电子邮件怎么办？*

不需要使用流畅设置器指定顺序非常方便。Kotlin 有 `*命名参数*` 来实现这一点：

```java
val mail = Mail_V3(title = "Hello", message = "There", to = listOf("my@dear.cat"))
```

将默认参数与命名参数结合使用使得在 Kotlin 中创建复杂对象变得相当容易。因此，在 Kotlin 中，你几乎根本不需要 Builder 设计模式。

在实际应用中，你经常会看到使用 Builder 设计模式来构建服务器的实例。服务器将接受可选的主机、可选的端口等，然后在所有参数都设置好之后，你会调用一个监听方法来启动它。

# 原型

**原型**设计模式全部关于定制和创建相似但略有不同的对象。为了更好地理解它，让我们来看一个例子。

想象我们有一个管理系统，用于管理用户及其权限。表示用户的 `data class` 可能看起来像这样：

```java
data class User(
```

```java
    val name: String,
```

```java
    val role: Role,
```

```java
    val permissions: Set<String>,
```

```java
) {
```

```java
    fun hasPermission(permission: String) = permission in       permissions
```

```java
}
```

每个用户都必须有一个角色，每个角色都有一组权限。

我们将把角色描述为一个 `enum` 类：

```java
enum class Role {
```

```java
    ADMIN,
```

```java
    SUPER_ADMIN,
```

```java
    REGULAR_USER
```

```java
}
```

`enum` 类是一种表示常量集合的方式。这比将角色表示为字符串更方便，例如，我们在编译时检查此类对象是否存在。

当我们创建一个新的 *用户* 时，我们将为他们分配与具有相同 *角色* 的另一个用户相似的权限：

```java
// In real application this would be a database of users
```

```java
val allUsers = mutableListOf<User>()
```

```java
fun createUser(name: String, role: Role) {
```

```java
    for (u in allUsers) {
```

```java
        if (u.role == role) {
```

```java
            allUsers += User(name, role, u.permissions)
```

```java
            return
```

```java
        }
```

```java
    }
```

```java
    // Handle case that no other user with such a role exists
```

```java
}
```

让我们假设现在我们需要向 `User` 类添加一个新字段，我们将它命名为 `tasks`：

```java
data class User(
```

```java
    val name: String,
```

```java
    val role: Role,
```

```java
    val permissions: Set<String>,
```

```java
    val tasks: List<String>,
```

```java
) {
```

```java
   ...
```

```java
}
```

我们的 `createUser` 函数将停止编译。我们将不得不通过将新添加字段的值复制到我们类的新的实例中来更改它：

```java
allUsers += User(name, role, u.permissions, u.tasks)
```

每次更改 `User` 类时，都必须重复这项工作。

然而，还有一个更大的问题：*如果引入了新的需求，使得* `permissions` *属性，例如，*变为 `private`*，会发生什么？*

```java
data class User(
```

```java
    val name: String,
```

```java
    val role: Role,
```

```java
    private val permissions: Set<String>,
```

```java
    val tasks: List<String>,
```

```java
) {
```

```java
   ...
```

```java
}
```

我们的代码将再次停止编译，我们又将不得不再次更改它。代码更改的这种持续需求是我们需要另一种方法来解决这个问题的一个明显迹象。

## 从原型开始

原型的整个想法是能够轻松地克隆一个对象。至少有两个原因你可能想要这样做：

+   当创建对象非常昂贵时，例如需要从数据库中获取它时，这很有帮助。

+   如果你需要创建相似但略有不同的对象，并且不想反复重复相似的部分，这会很有帮助。

    重要提示：

    使用原型设计模式还有更多高级的理由。例如，JavaScript 使用原型来实现类似于类的继承行为，而不需要类。

幸运的是，Kotlin 修复了 Java `clone()` 方法的某些缺陷。数据类有一个 `copy()` 方法，它接受一个现有的 `data class`，并创建它的一个新副本，在此过程中可以选择更改一些属性：

```java
// Name argument is underscored here simply not to confuse 
```

```java
// it with the property of the same name in the User object
```

```java
fun createUser(_name: String, role: Role) {
```

```java
    for (u in allUsers) {
```

```java
        if (u.role == role) {
```

```java
            allUsers += u.copy(name = _name)
```

```java
            return
```

```java
        }
```

```java
    }
```

```java
    // Handle case that no other user with such a role exists
```

```java
}
```

类似于我们之前看到的 Builder 设计模式，命名参数允许我们以任何顺序指定可以更改的属性。我们只需要指定我们想要更改的属性。所有其他数据都将为我们复制，即使是 `private` 属性。

`data class` 是一种非常常见的设计模式，它已经成为语言语法的一部分。这是一个极其有用的特性，我们将在本书中多次看到它的应用。

# 摘要

在本章中，我们学习了何时以及如何使用创建型设计模式。我们首先讨论了如何使用`object`关键字来构造单例类，然后讨论了如果需要静态工厂方法时如何使用`companion object`。我们还介绍了如何使用解构声明一次性分配多个变量。

然后，我们讨论了智能转换，以及它们如何在抽象工厂设计模式中应用以创建对象系列。接着，我们转向建造者设计模式，并了解到函数可以有默认参数值。然后我们学习了我们可以不仅通过位置，还可以通过名称来引用它们的参数。

最后，我们介绍了数据类的`copy()`函数，以及它在实现原型设计模式时如何帮助我们产生略有不同的相似对象。你现在应该理解了如何使用创建型设计模式来更好地管理你的对象。

在下一章中，我们将介绍设计模式的第二组：**结构型模式**。这些设计模式将帮助我们创建可扩展和维护的对象层次结构。

# 问题

1.  列出我们在本章中学到的`object`关键字的两个用途。

1.  `apply()`函数是用来做什么的？

1.  提供一个静态工厂方法的示例。
