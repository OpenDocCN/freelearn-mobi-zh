# Kotlin 入门

在本章中，我们将介绍基本的 Kotlin 语法，并讨论哪些设计模式是好的，以及为什么应该在 Kotlin 中使用它们。

本章的目标不是涵盖整个语言词汇表，而是让你熟悉一些基本概念和惯用语。接下来的章节将逐渐向你展示更多与我们将讨论的设计模式相关的语言特性。

本章将涵盖以下主题：

+   基本语言语法和特性

+   设计模式简介

# 基本语言语法和特性

无论你是从 Java、C#、Scala 还是任何其他静态类型编程语言开始，你都会发现 Kotlin 语法非常熟悉。这不是巧合，而是为了让那些在其他语言中有经验的人尽可能顺利地过渡到这门新语言。除了这种熟悉感之外，Kotlin 还带来了大量的特性，如更好的类型安全性。随着我们前进，你会发现它们都在尝试解决现实世界的问题。这种实用主义方法在语言中非常一致。例如，Kotlin 最强的优势之一是完整的 Java 互操作性。你可以在 Java 和 Kotlin 类旁边使用，并且可以自由地使用任何可用于 Kotlin 项目的 Java 库。

总结来说，语言的目标如下：

+   实用主义

+   具有清晰的语法

+   具有类型安全性

+   互操作性

第一章将讨论如何实现这些目标。

# 多范式

编程语言中的主要范式包括过程范式、面向对象范式和函数式范式。

Kotlin 是一个实用的语言，它允许使用任何这些范式。它有类和继承，来自面向对象的方法。它有来自函数式编程的高阶函数。但如果你不想的话，不必将所有内容都封装在类中。你可以将整个代码结构为一系列过程和结构。你将看到所有这些方法是如何结合在一起的，因为不同的示例将使用不同的范式来解决讨论中的问题。

# 代码结构

当你开始用 Kotlin 编程时，你需要做的第一件事是创建一个新文件。Kotlin 的扩展名通常是 `.kt`。

与 Java 不同，文件名和类名之间没有强烈的关系。你可以将尽可能多的公共类放入文件中，只要这些类彼此相关，并且文件不要太长，难以阅读。

# 无分号

在 Java 中，每一行代码都必须以分号结束：

```kt
System.out.println("Hello"); //<- This is a semicolon System.out.println("World"); //<- I still see you, semicolon 
```

但 Kotlin 是一种实用主义语言。因此，它会在编译期间推断出应该放置分号的位置：

```kt
println("Hello") //<- No semicolon here
println("World") //<- Not here
```

大多数情况下，你不需要在代码中添加分号。它们被认为是可选的。

# 命名约定

作为一个惯例，如果你的文件中只有一个类，那么将文件名与类名相同。

如果你的文件包含多个类，那么文件名应该描述这些类的共同用途。根据 Kotlin 编码规范，使用驼峰式命名文件：[`kotlinlang.org/docs/reference/coding-conventions.html#naming-rules`](https://kotlinlang.org/docs/reference/coding-conventions.html#naming-rules)。

实际上，你不需要为简单的代码片段编写文件。你还可以在线与语言互动：尝试[`kotlinlang.org/`](http://kotlinlang.org/)或安装 Kotlin 后运行`kotlinc`并使用 REPL 和交互式 shell。

# 包

如果你的所有类和函数都在同一个文件夹或同一个命名空间下，这并不方便。这就是为什么 Kotlin，类似于许多其他语言，使用包的概念。

与 Java 一样，Kotlin 使用包：

```kt
package me.soshin.controllers
```

如果你混合使用 Java 和 Kotlin，Kotlin 文件应遵循 Java 包规则。

在纯 Kotlin 项目中，可以从文件夹结构中省略常见的包前缀。例如，如果你的所有项目都在`me.soshin`包下，请将控制器放在`/controllers`文件夹中，而不是像 Java 那样放在`/me/soshin/controllers`文件夹中。

# 类型

我们将从 Kotlin 类型系统开始，并将其与 Java 提供的进行比较。

Java 示例是为了熟悉，并不是为了证明 Kotlin 在任何一个方面都优于 Java。

# 类型推断

让我们在 Java 中定义一个简单的字符串：

```kt
String s = "Hello World";
```

我们定义了`s`是`String`类型。但为什么？在这个时候不是显然的吗？

Kotlin 为我们提供了类型推断：

```kt
val s = "Hello World"
```

现在，编译器将决定应该使用哪种类型的变量。与解释型语言（如 JavaScript、Groovy 或 Ruby）不同，变量的类型只定义一次。这不会起作用：

```kt
var s = "I'm a string"
s = 1 // s is a String
```

你可能会想知道为什么我们使用了一个`var`和一个`val`来定义变量。我们很快就会解释。

# `val`与`var`

在 Java 中，变量可以被声明为`final`。`final`变量只能赋值一次：

```kt
final String s = "Hi";
s = "Bye"; // Doesn't work
```

Kotlin 强烈建议尽可能使用不可变数据。Kotlin 中的`final`变量只是`val`：

```kt
val s = "Hi"
s = "Bye" // Doesn't work
```

如果你确实有一个想要重新分配变量的情况，请使用`var`：

```kt
var s = "Hi"
s = "Bye" // Works now
```

# 比较

我们在 Java 的早期学习中就被告知，使用`==`比较对象不会产生预期的结果，因为它测试的是引用相等性，我们需要使用`equals()`来做到这一点。

JVM 在基本情况下进行字符串池化以防止这种情况，因此为了示例，我们将使用`new String()`来避免这种情况：

```kt
String s1 = "ABC";
String s2 = new String(s1);

System.out.println(s1 == s2); // false
```

Kotlin 将`==`转换为`equals()`：

```kt
val s1 = "ABC"
val s2 = String(s1.toCharArray())

println(s1 == s2) // true
```

如果你确实想检查引用相等性，请使用`===`：

```kt
println(s1 === s2) // false
```

# 空安全

在 Java 世界中，最臭名昭著的异常可能是`NullPointerException`。

这种异常背后的原因是 Java 中的每个对象都可以是`null`。这里的代码展示了原因：

```kt
String s = "Hello";
...
s = null;
System.out.println(s.length); // Causes NullPointerException
```

在这种情况下，将`s`标记为`final`将防止异常发生。

但这个呢：

```kt
public class Printer {    
    public static void printLength(final String s) {
       System.out.println(s.length);
    }
}
```

从代码的任何地方都可以传递`null`：

```kt
Printer.printLength(null); // Again, NullPointerException
```

自 Java 8 以来，有一个`optional`构造：

```kt
if (optional.isPresent()) {
    System.out.println(optional.get());
}
```

在更函数式的方式中：

```kt
optional.ifPresent(System.out::println);
```

但是……这并没有解决我们的问题。我们仍然可以用`null`代替正确的`Optional.empty()`并使程序崩溃。

Kotlin 在编译时就会检查它：

```kt
val s : String = null // Won't compile
```

让我们回到我们的`printLength()`函数：

```kt
fun printLength(s: String) {
    println(s.length)
}
```

使用 null 调用这个函数将无法编译：

```kt
printLength(null) // Null can not be a value of a non-null type String
```

如果您希望您的类型能够接收 null 值，您需要使用问号将其标记为可空：

```kt
val notSoSafe : String? = null
```

# 声明函数

在 Java 中，一切都是对象。如果您有一个不依赖于任何状态的方法，它仍然必须被类包裹。您可能熟悉 Java 中许多只包含静态方法的`Util`类，它们的唯一目的是满足语言要求并将这些方法捆绑在一起。

在 Kotlin 中，函数可以声明在类外部，而不是以下代码：

```kt
public class MyFirstClass {
```

```kt

    public static void main(String[] args) {
        System.out.println("Hello world");
    }
}
```

只需这样就可以了：

```kt
fun main(args: Array<String>) {
    println("Hello, world!")
}
```

在任何类外部声明的函数已经是静态的。

本书中的许多示例都假设我们提供的代码被包裹在主函数中。如果您没有看到函数的签名，它可能应该是这样的：

`fun main(args: Array<String>)`。

声明函数的关键字是`fun`。参数类型跟在参数名称之后，而不是之前。如果函数不返回任何内容，可以完全省略返回类型。

如果您确实想声明返回类型？同样，它将紧跟在函数声明之后：

```kt
fun getGreeting(): String {
    return "Hello, world!"
}

fun main(args: Array<String>) {
    println(getGreeting())
}
```

关于函数声明还有很多其他主题，比如默认和命名参数、默认参数和可变数量的参数。我们将在接下来的章节中介绍它们，并附上相关示例。

# 控制流

可以说，控制流是编写程序的基础。我们将从两个条件表达式开始：`if`和`when`。

# 使用 if 表达式

之前提到 Kotlin 喜欢变量只分配一次。它也不太喜欢 null。您可能想知道这在现实世界中是如何工作的。在 Java 中，这样的结构相当常见：

```kt
public String getUnixSocketPolling(boolean isBsd) {
    String value = null;
    if (isBsd) {
        value = "kqueue";
    }
    else {
        value = "epoll";
    }

    return value;
}
```

当然，这是一个过于简化的情况，但仍然，您有一个变量，在某个时刻绝对必须是`null`，对吧？

在 Java 中，`if`只是一个语句，不返回任何内容。相反，在 Kotlin 中，`if`是一个表达式，意味着它返回一个值：

```kt
fun getUnixSocketPolling(isBsd : Boolean) : String {
    val value = if (isBsd) {
        "kqueue"
    } else {
        "epoll"
    }
    return value
}
```

如果您熟悉 Java，您可以轻松阅读这段代码。这个函数接收一个布尔值（不能为 null），并返回一个字符串（永远不会为 null）。但由于它是一个表达式，它可以返回一个结果。并且结果只分配给我们的变量一次。

我们甚至可以进一步简化它：

1.  返回类型可以推断

1.  作为最后一行的返回可以省略

1.  一个简单的`if`表达式可以写在一行中

因此，我们的最终结果在 Kotlin 中将看起来像这样：

```kt
fun getUnixSocketPolling(isBsd : Boolean) = if (isBsd) "kqueue" else "epoll"
```

Kotlin 中的单行函数非常酷且实用。但您应该确保除了您之外的其他人也能理解它们的功能。请谨慎使用。

# 使用`when`表达式

如果（不是字面意义上的玩笑）我们想在`if`语句中有更多的条件呢？

在 Java 中，我们使用`switch`语句。在 Kotlin 中，有一个更强大的`when`表达式，因为它可以嵌入一些其他的 Kotlin 特性。

让我们创建一个基于将产生建议一个不错的生日礼物的原因的金额的方法：

```kt
fun suggestGift(amount : Int) : String {
    return when (amount) {
        in (0..10) -> "a book"
        in (10..100) -> "a guitar"
        else -> if (amount < 0) "no gift" else "anything!"
    }
}
```

如你所见，`when`也支持一系列值。默认情况由`else`块处理。在下面的示例中，我们将更详细地阐述使用这个表达式的更强大方式。

一般规则是，如果有超过两个条件，使用`when`。对于简单的检查，使用`if`。

# 字符串插值

如果我们想要实际打印这些结果呢？

首先，正如你可能已经注意到的，在上面的一个示例中，Kotlin 提供了一个巧妙的`println()`标准函数，它封装了 Java 中更庞大的`System.out.println()`。

但是，更重要的是，就像许多其他现代语言一样，Kotlin 支持使用`${}`语法进行字符串插值。在之前的示例之后继续：

```kt
println("I would suggest: ${suggestGift(10)} ")
```

上述代码将打印以下内容：

```kt
I would suggest: a book
```

如果你正在插值一个变量，而不是一个函数，可以省略大括号：

```kt
val gift = suggestGift(100)
println("I would suggest: $gift ")
```

这将打印以下输出：

```kt
I would suggest: a guitar
```

# 类和继承

虽然 Kotlin 是多范式的，但它与基于类的 Java 编程语言有着强烈的亲和力。考虑到 Java 和 JVM 的互操作性，Kotlin 也有类和经典继承的概念也就不足为奇了。

# 类

为了声明一个`class`，我们使用类关键字，就像在 Java 中一样：

```kt
class Player {
}
```

Kotlin 中没有`new`关键字。类的实例化看起来就像这样：

```kt
// Kotlin figured out you want to create a new player
val p = Player() 
```

如果类没有主体，就像这个简单的例子一样，我们可以省略大括号：

```kt
class Player // Totally fine
```

# 继承

正如 Java 中的情况一样，抽象类由`abstract`关键字标记，接口由`interface`关键字标记：

```kt
abstract class AbstractDungeonMaster {
    abstract val gameName: String

    fun startGame() {
        println("Game $gameName has started!")
    }
}

interface Dragon
```

就像 Java 8 一样，Kotlin 中的接口可以有函数的默认实现，只要它们不依赖于任何状态：

```kt
interface Greeter {
 fun sayHello() {
 println("Hello")
 }
}
```

Kotlin 中没有`inherits`和`implements`关键字。相反，抽象类的名称以及该类实现的所有接口的名称都放在冒号后面：

```kt
class DungeonMaster: Greeter, AbstractDungeonMaster() {
    override val gameName: String
        get() = "Dungeon of the Beholder"
}
```

我们仍然可以通过其名称后面的括号来区分抽象类，并且仍然只能有一个`abstract`类，因为 Kotlin 中没有多重继承。

我们的`DungeonMaster`可以访问`Greeter`和`AbstractDungeonMaster`中的两个函数：

```kt
val p = DungeonMaster()
p.sayHello()  // From Greeter interface
p.startGame() // From AbstractDungeonMaster abstract class
```

调用前面的代码，它将打印以下输出：

```kt
Hello
Game Dungeon of the Beholder has started!
```

# 构造函数

我们的`DungeonMaster`现在看起来有点尴尬，因为它只能宣布开始一个游戏。让我们给我们的`abstract`类添加一个非空构造函数来解决这个问题：

```kt
abstract class AbstractDungeonMaster(private val gameName : String) {
    fun startGame() {
        println("Game $gameName has started!")
    }
}
```

现在，我们的`DungeonMaster`必须接收游戏名称并将其传递给`abstract`类：

```kt
open class DungeonMaster(gameName: String):
        Greeter, AbstractDungeonMaster(gameName)
```

如果我们想要通过一个`EvilDungeonMaster`来扩展`DungeonMaster`呢？

在 Java 中，除非被标记为`final`，否则所有类都可以被扩展。在 Kotlin 中，除非被标记为`open`，否则没有类可以被扩展。这同样适用于抽象类中的函数。这就是为什么我们最初将`DungeonMaster`声明为`open`的原因。

我们将对`AbstractDungeonMaster`进行一些修改，以赋予邪恶统治者更多的权力：

```kt
open fun startGame() {
    // Everything else stays the same
}
```

现在，我们在我们的`EvilDungeonMaster`实现中添加以下内容：

```kt
class EvilDungeonMaster(private val awfulGame: String) : DungeonMaster(awfulGame) {
    override fun sayHello() {
        println("Prepare to die! Muwahaha!!!")
    }

    override fun startGame() {
        println("$awfulGame will be your last!")
    }
}
```

而在 Java 中，`@Override`是一个可选的注解，在 Kotlin 中则是一个强制的关键字。

你不能隐藏超类型方法，并且没有显式使用`override`的代码将无法编译。

# 属性

在 Java 中，我们习惯于 getter 和 setter 的概念。一个典型的类可能看起来像这样：

```kt
public class Person {
    private String name;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    // More methods come here
}
```

如果我们要获取一个人的名字，我们调用`getName()`。如果我们想更改它，我们调用`setName()`。这很简单。

如果我们只想在对象实例化期间设置一次名字，我们可以指定非默认构造函数并删除 setter，如下所示：

```kt
public class ImmutablePerson {
    private String name;

    public ImmutablePerson(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }
}
```

所有这些都可以追溯到 Java 的起点，大约在 1995 年左右。

但如果你例如与 C#一起工作过，你可能熟悉属性的概念。为了理解它们，让我们回到第一个示例并稍作修改：

```kt
public class PublicPerson {
    public String name;
}
```

读取一个人的名字并不短多少：`p.name`。

此外，更改名字的方式更加直观：`p.name = "Alex";`。

但这样做，我们失去了对我们对象的大量控制。我们无法使`PublicPerson`不可变。如果我们想让每个人都能读取人的名字，他们也可以在任何时候更改它。如果我们后来决定所有名字都必须是大写字母，我们可以通过 setter 来实现这一点。但不是通过公共字段。

属性为所有这些问题提供了一个解决方案：

```kt
class Person() {
    var name : String = ""
}
```

这可能看起来与 Java 示例相同，带有所有其问题。但实际上，在幕后，它被编译为一个 getter 和 setter 对，就像第一个示例一样。

由于 Kotlin 中的属性被转换为 getter 和 setter，我们也可以控制它们的行为：

```kt
class Person {
    var name : String = ""
    set(value) {
        field = value.toUpperCase()
    }
}
```

注意，我们不需要检查`value`是否为 null。`String`类型简单地不能接收 null 值。

来自 Java，使用以下赋值可能看起来很直观：`this.name = value.toUpperCase()`。但在 Kotlin 中，这将创建一个循环依赖。相反，我们使用了一个`field`标识符，它是自动提供的。

# 数据类

记得 Kotlin 的全部都是关于生产力吗？Java 开发者最常见的任务之一是创建另一个 **普通的 Java 对象** (**POJO**)。如果你不熟悉 POJO，它基本上是一个只有 getter、setter 以及`equals`或`hashCode`方法实现的对象。

这个任务如此常见，以至于 Kotlin 将其内置到语言中：

```kt
data class User (val username : String, val password : String)
```

这将生成一个具有两个 getter 而没有 setter（注意`val`部分）的类，它还将以正确的方式实现`equals`、`hashCode`和`clone`函数。

数据类的引入是语言中减少样板代码的最大改进之一。

# 更多的控制流 - 循环

现在让我们讨论另一个常见的控制结构——循环。循环对于大多数开发者来说是一个非常自然的结构。没有循环，重复相同的代码块将非常困难（尽管我们将在后面的章节中讨论如何在没有循环的情况下做到这一点）。

# For 循环

Java 中的 `for` 循环，它将字符串的每个字符打印在新的一行上，可能看起来像这样：

```kt
final String word = "Word";
for (int i = 0; i < word.length; i++) {

}
```

Kotlin 中的相同循环如下：

```kt
val word = "Word";
for (i in 0..(word.length-1)) {
    println(word[i])
}
```

注意，虽然 Java 中的常规 `for` 循环是排他的（它根据定义排除了最后一个索引，除非有其他指定），但 Kotlin 中对范围的 `for` 循环是包含的。这就是为什么我们必须从长度中减去一来防止溢出（字符串索引超出范围）：`(word.length-1)`。

如果你想避免这种情况，可以使用 `until` 函数：

```kt
val word = "Word";
for (i in 0 until word.length) {
    println(word[i])
}
```

与其他一些语言不同，反转范围索引不起作用：

```kt
val word = "Word";
for (i in (word.length-1)..0) {
    println(word[i])
} // Doesn't print anything
```

如果你想要以相反的顺序打印单词，例如，使用 `downTo` 函数：

```kt
val word = "Word";
for (i in (word.length-1) downTo 0) {
    println(word[i])
}
```

它将打印以下输出：

```kt
d
r
o
W
```

可能会让人困惑的是，尽管 `until` 和 `downTo` 看起来更像操作符，但它们被称为函数。这是 Kotlin 另一个有趣的功能，称为 **中缀调用**，稍后我们将讨论。

# For-each 循环

当然，如果你对 Java 有点熟悉，你可能会争辩说，前面的代码可以通过使用 `for-each` 构造来改进：

```kt
final String word = "Word";

for (Character c : word.toCharArray()) {
    System.out.println(c);
}
```

在 Kotlin 中，这将是相同的：

```kt
val word = "Word"

for (c in word) {
    println(c)
}
```

# While 循环

`while` 循环的功能没有变化，所以我们非常简短地介绍它们：

```kt
var x = 0
while (x < 10) { 
   x++ 
   println(x)
}
```

这将打印从 1 到 10 的数字。请注意，我们被迫将 `x` 定义为 `var`。在接下来的章节中，我们将讨论更多更习惯性的方法来做这件事。

较少使用的 `do while` 循环也存在于该语言中：

```kt
var x = 5
   do { 
      println(x)
      x--
} while (x > 0)
```

# 扩展函数

你可能已经注意到，从前面的例子中，Kotlin 中的 `String` 有一些方法，其 Java 对应版本缺少，例如 `reversed()`。如果它与 Java 中的 `String` 类型相同，并且我们知道，Java 中的 `String` 不能被任何其他类扩展，因为它被声明为 `final`，那么这是如何实现的呢？

如果你查看源代码，你会发现以下内容：

```kt
public inline fun String.reversed(): String {
    return (this as CharSequence).reversed().toString()
}
```

这个特性被称为扩展函数，它也存在于一些其他语言中，例如 C# 或 Groovy。

要在不继承的情况下扩展一个类，我们可以在示例中函数名 `reversed` 前面加上我们想要扩展的类名。

请注意，扩展函数不能覆盖成员函数。`inline` 关键字将在后面的章节中讨论。

# 设计模式简介

现在我们对 Kotlin 的基本语法有点熟悉了，我们可以继续讨论设计模式是什么。

# 设计模式是什么？

围绕设计模式存在不同的误解。一般来说，它们如下：

+   缺失的语言特性

+   在动态语言中不是必需的

+   仅适用于面向对象的语言

+   仅适用于企业

但实际上，设计模式只是解决常见问题的一种经过验证的方法。作为一个概念，它们并不局限于特定的编程语言（例如 Java），也不局限于语言家族（例如 C 家族），甚至不局限于编程本身。你可能甚至听说过软件架构中的设计模式，它们讨论了不同的系统如何高效地相互通信。有面向服务的架构模式，你可能知道它为**面向服务架构**（**SOA**），以及从 SOA 演变而来的微服务设计模式，这些模式在过去几年中逐渐出现。未来肯定会给我们带来更多的设计模式**家族**。

即使在物理世界中，在软件开发之外，我们也被设计模式和某些问题的常见解决方案所包围。让我们看看一个例子。

# 生活中的设计模式

你最近乘坐过电梯吗？电梯墙上有没有镜子？为什么会有这样的设计？

当你上次乘坐没有镜子也没有玻璃墙的电梯时，你感觉如何？

我们在电梯中普遍安装镜子的主要原因是为了解决一个常见问题。乘坐电梯是无聊的。我们可以放一张图片。但如果你每天至少乘坐两次相同的电梯，图片也会很快变得无聊。虽然便宜，但改善不大。

我们可以安装一个电视屏幕，就像有些人做的那样。但这会使电梯更贵。而且它还需要大量的维护。我们需要在屏幕上放置一些内容，以避免重复。所以要么有一个人的责任是偶尔更新内容，要么有一个第三方公司为我们做这件事。我们还得处理屏幕硬件及其背后的软件可能出现的不同问题。当然，看到“蓝屏死机”是很有趣的，但只是轻微的乐趣。

一些建筑师甚至选择将电梯井安装在建筑外部，并使部分墙壁透明。这可能提供一些令人兴奋的视野。但这个解决方案也需要维护（脏窗户不会提供最好的视野），以及大量的建筑规划。

因此，我们安装了镜子。即使你独自乘坐，你也能看到吸引人的面孔。一些研究表明，我们发现自己比实际更吸引人。也许你有机会在重要会议之前最后一次审视你的外表。镜子从视觉上扩展了视觉空间，使整个旅程不那么令人窒息，或者不那么尴尬，如果这是新的一天，电梯真的很拥挤。

# 设计过程

让我们尝试理解我们刚才做了什么。

我们没有在电梯里发明镜子。我们见过它们成千上万次。但我们正式化了这个问题（乘坐电梯很无聊）并讨论了替代方案（电视屏幕、玻璃墙）以及常用解决方案的好处（解决问题，易于实现）。这就是设计模式的所有内容。

设计过程的基本步骤是：

1.  准确定义当前的问题。

1.  考虑不同的替代方案，基于其优缺点。

1.  选择解决问题的方案，同时最好地符合你的具体约束。

# 为什么在 Kotlin 中使用设计模式？

Kotlin 出现是为了解决当今的现实世界问题。在接下来的章节中，我们将讨论由四人帮在 94 年首次提出的 *设计模式*，以及从函数式编程范式出现的设计模式。

你会发现，一些设计模式非常常见或有用，以至于它们已经作为保留关键字或标准函数内置到语言中。其中一些需要结合一组语言特性。而一些则不再那么有用，因为世界已经前进，它们正被一些其他模式所取代。

但无论如何，熟悉设计模式和最佳实践扩展了你的“开发者工具箱”，并在你和你同事之间创造了共享的词汇。

# 摘要

因此，在本章中，我们介绍了 Kotlin 编程语言的主要目标。

我们讨论了定义的变量，例如 `val`、`var`、空安全性和类型推断。我们观察了程序流程是如何通过 `if`、`when`、`for` 和 `while` 等命令来控制的，我们还查看了解决类和接口定义的不同关键字：`class`、`interface`、`data` 和 `abstract` 类。我们学习了如何构建新的类，以及我们如何从接口继承和实现类。最后，我们学习了设计模式的好处，以及为什么在 Kotlin 中需要它们。

在下一章中，我们将开始讨论三种设计模式家族中的第一个：创建型模式。
