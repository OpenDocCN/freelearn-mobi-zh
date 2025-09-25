# *第九章*：惯用和反模式

在前面的章节中，我们讨论了 Kotlin 编程语言的不同方面、函数式编程的好处以及并发设计模式。

本章讨论了 Kotlin 的最佳和最差实践。你将了解惯用的 Kotlin 代码应该是什么样子，以及哪些模式应该避免。本章包含了一个涵盖这些不同主题的最佳实践集合。

在本章中，我们将涵盖以下主题：

+   使用作用域函数

+   类型检查和转换

+   try-with-resources 语句的替代方案

+   内联函数

+   实现代数数据类型

+   重新泛型

+   高效使用常量

+   构造函数重载

+   处理 null 值

+   明确异步性

+   验证输入

+   优先使用密封类而不是枚举

完成本章后，你应该能够编写更易于阅读和维护的 Kotlin 代码，以及避免一些常见的陷阱。

# 技术要求

除了前几章的要求之外，你还需要一个**Gradle**启用的**Kotlin**项目，以便能够添加所需的依赖项。

你可以在这里找到本章的源代码：[`github.com/PacktPublishing/Kotlin-Design-Patterns-and-Best-Practices/tree/main/Chapter09`](https://github.com/PacktPublishing/Kotlin-Design-Patterns-and-Best-Practices/tree/main/Chapter09)。

# 使用作用域函数

Kotlin 有**作用域函数**的概念，这些函数在任何对象上都是可用的，可以替代编写重复代码的需要。除了其他好处之外，这些作用域函数帮助我们简化单表达式函数。它们被认为是高阶函数，因为每个作用域函数都接收一个 lambda 表达式作为参数。在本节中，我们将讨论所有必要的函数，并使用对象作为它们的*作用域*来执行它们的代码块。在本节中，我们将交替使用*作用域*和*上下文对象*来描述这些函数操作的对象。

## 令函数

我们可以使用`let()`函数在可空对象上调用一个函数，但前提是这个对象不是 null。

让我们以以下引言地图（我们在*第一章*，*Kotlin 入门*）为例：

```kt
val clintEastwoodQuotes = mapOf(
```

```kt
    "The Good, The Bad, The Ugly" to "Every gun makes its       own tune.",
```

```kt
    "A Fistful Of Dollars" to "My mistake: four coffins."
```

```kt
)
```

现在，让我们从一个可能不在集合中的电影中获取一个引言并打印它，但前提是它不是 null：

```kt
val quote = clintEastwoodQuotes["Unforgiven"]
```

```kt
if (quote != null) {
```

```kt
    println(quote)
```

```kt
} 
```

同样的代码可以使用`let`作用域函数重写：

```kt
clintEastwoodQuotes["Unforgiven"]?.let {
```

```kt
    println(it)
```

```kt
}
```

一个常见的错误是忘记在`let`之前使用安全导航操作符，因为`let()`本身也可以在 null 上工作：

```kt
clintEastwoodQuotes["Unforgiven"].let {
```

```kt
    println(it)
```

```kt
}
```

这段代码将在控制台打印`null`。确保在使用`let()`进行 null 检查时不要忘记问号（`?`）。

## Apply 函数

我们在前面章节中讨论了`apply()`。它返回它操作的对象，并将上下文设置为`this`。如果你需要初始化一个可变对象，可以使用`apply()`。

想想你有多少次不得不创建一个具有空构造函数的类，然后依次调用很多设置器。以下是一个例子，这可能是一个来自库的类。例如：

```kt
class JamesBond { 
```

```kt
    lateinit var name: String 
```

```kt
    lateinit var movie: String 
```

```kt
    lateinit var alsoStarring: String 
```

```kt
} 
```

当我们需要创建此类的新实例时，我们可以以过程式的方式这样做：

```kt
val agent = JamesBond() 
```

```kt
agent.name = "Sean Connery" 
```

```kt
agent.movie = "Dr. No"
```

或者，我们可以只设置 `name` 和 `movie`，并使用 `apply()` 函数将 `alsoStarring` 留空：

```kt
val `007` = JamesBond().apply { 
```

```kt
    this.name = "Sean Connery" 
```

```kt
    this.movie = "Dr. No" 
```

```kt
} 
```

```kt
println(`007`.name)
```

由于代码块的范围被设置为 `this`，我们可以进一步简化前面的代码：

```kt
val `007` = JamesBond().apply { 
```

```kt
    name = "Sean Connery" 
```

```kt
    movie = "Dr. No" 
```

```kt
}
```

使用 `apply()` 函数在处理通常有很多设置器和默认空构造函数的 Java 类时特别有用。

## `also` 函数

正如我们在本节介绍的引言中提到的，单表达式函数非常简洁。让我们看看以下简单的函数，它乘以两个数字：

```kt
fun multiply(a: Int, b: Int): Int = a * b
```

但通常，你有一个单语句函数，还需要执行其他操作，例如写入日志或产生其他副作用。为了实现这一点，我们可以将我们的函数重写如下：

```kt
fun multiply(a: Int, b: Int): Int { 
```

```kt
    val c = a * b 
```

```kt
    println(c) 
```

```kt
    return c 
```

```kt
}
```

我们不得不在这里使我们的函数更加冗长，并引入另一个变量。让我们看看我们如何使用 `also()` 函数来替代：

```kt
fun multiply(a: Int, b: Int): Int = 
```

```kt
    (a * b).also { println(it) }
```

此函数将表达式的结果分配给 `it` 并返回表达式的结果。`also()` 函数在你想在一系列调用中产生副作用时也非常有用：

```kt
val l = (1..100).toList() 
```

```kt
l.filter{ it % 2 == 0 } 
```

```kt
    // Prints, but doesn't mutate the collection
```

```kt
    .also { println(it) }     
```

```kt
    .map { it * it }
```

在这里，你可以看到我们可以继续使用 `map()` 函数来继续我们的调用链，即使我们使用了 `also()` 函数来打印列表中的每个元素。

## 运行函数

`run()` 函数与 `let()` 函数非常相似，但它将代码块的范围设置为 `this` 而不是使用 `it`。

让我们通过一个例子来更好地理解这一点：

```kt
val justAString = "string" 
```

```kt
val n = justAString.run {  
```

```kt
    this.length 
```

```kt
}
```

在这个例子中，`this` 被设置为引用 `justAString` 变量。

通常，`this` 可以省略，所以代码将如下所示：

```kt
val n = justAString.run {  
```

```kt
    length 
```

```kt
}
```

`run()` 函数主要用于初始化对象，就像我们之前讨论的 `apply()` 函数一样。然而，与 `apply()` 不同，它返回对象本身，而不是返回某些计算的结果：

```kt
val lowerCaseName = JamesBond().run {
```

```kt
    name = "ROGER MOORE"
```

```kt
    movie = "THE MAN WITH THE GOLDEN GUN"
```

```kt
    name.toLowerCase() // <= Not JamesBond type
```

```kt
}
```

```kt
println(lowerCaseName)
```

上述代码打印以下输出：

```kt
> roger moore
```

在这里，对象使用 `"ROGER MOORE"` 初始化。注意，在这里，我们对 `JamesBond` 对象进行了操作，但我们的返回值是一个 `String`。

## 使用函数

与其他四个作用域函数不同，`with()` 不是一个扩展函数。这意味着你不能这样做：

```kt
"scope".with { ... }
```

相反，`with()` 接收你想要作用域的对象作为参数：

```kt
with("scope") { 
```

```kt
    println(this.length) // "this" set to the argument of                          // with() 
```

```kt
}
```

并且，像往常一样，我们可以省略 `this`：

```kt
with("scope") { 
```

```kt
    length 
```

```kt
}
```

就像 `run()` 和 `let()` 一样，你可以从 `with()` 中返回任何结果。

在本节中，我们学习了各种作用域函数如何通过定义要在对象上执行的代码块来帮助减少样板代码的数量。在下一节中，我们将看到 Kotlin 还允许我们比其他语言编写更少的实例检查。

# 类型检查和转换

在编写代码时，你可能会倾向于检查你的对象类型，使用`is`，然后使用`as`进行转换。作为一个例子，让我们想象我们正在构建一个超级英雄系统。每个超级英雄都有自己的方法集：

```kt
interface Superhero 
```

```kt
class Batman : Superhero { 
```

```kt
    fun callRobin() { 
```

```kt
        println("To the Bat-pole, Robin!") 
```

```kt
    } 
```

```kt
} 
```

```kt
class Superman : Superhero { 
```

```kt
    fun fly() { 
```

```kt
        println("Up, up and away!") 
```

```kt
    } 
```

```kt
} 
```

此外，还有一个函数，超级英雄试图施展他们的超能力：

```kt
fun doCoolStuff(s: Superhero) { 
```

```kt
    if (s is Superman) { 
```

```kt
        (s as Superman).fly() 
```

```kt
    } 
```

```kt
    else if (s is Batman) { 
```

```kt
        (a as Batman).callRobin() 
```

```kt
    } 
```

```kt
}
```

但正如你所知，Kotlin 有智能转换，所以在这种情况下，隐式转换不是必需的。让我们使用智能转换重写这个函数，看看它们如何改进我们的代码。我们只需要从我们的代码中移除显式转换：

```kt
fun doCoolStuff(s: Superhero) { 
```

```kt
    if (s is Superman) { 
```

```kt
        s.fly() 
```

```kt
    } 
```

```kt
    else if (s is Batman) { 
```

```kt
        s.callRobin() 
```

```kt
    } 
```

```kt
}
```

此外，在大多数情况下，使用`when()`进行智能转换会产生更干净的代码：

```kt
fun doCoolStuff(s : Superhero) { 
```

```kt
    when(s) { 
```

```kt
        is Superman -> s.fly() 
```

```kt
        is Batman -> s.callRobin() 
```

```kt
        else -> println("Unknown superhero") 
```

```kt
    } 
```

```kt
}
```

作为一条经验法则，你应该避免使用转换，并尽可能多地依赖智能转换：

```kt
// Superhero is clearly not a string 
```

```kt
val superheroAsString = (s as String)
```

但如果你绝对必须，还有一个安全转换运算符：

```kt
val superheroAsString = (s as? String)
```

**安全转换运算符**如果对象无法转换，将返回 null，而不是抛出异常。

# try-with-resources 语句的替代方案

`AutoCloseable`和 try-with-resources 语句。

这个声明使我们能够提供一组资源，一旦代码使用完毕，这些资源将被自动关闭。因此，将不再有忘记关闭文件的风险（或者至少风险更小）。

在 Java 7 之前，这完全是混乱的，如下面的代码所示：

```kt
BufferedReader br = null; // Nulls are bad, we know that 
```

```kt
try { 
```

```kt
    br = new BufferedReader(new FileReader
```

```kt
      ("./src/main/kotlin/7_TryWithResource.kt "));
```

```kt
    System.out.println(br.readLine());
```

```kt
} 
```

```kt
finally { 
```

```kt
    if (br != null) { // Explicit check 
```

```kt
        br.close(); // Boilerplate 
```

```kt
    } 
```

```kt
}
```

在 Java 7 发布后，前面的代码可以写成以下形式：

```kt
try (BufferedReader br = new BufferedReader(new   FileReader("/some/path"))) { 
```

```kt
    System.out.println(br.readLine()); 
```

```kt
}
```

Kotlin 不支持这种语法。相反，try-with-resource 语句被`use()`函数所取代：

```kt
val br = BufferedReader(FileReader("./src/main   /kotlin/7_TryWithResource.kt"))
```

```kt
br.use {
```

```kt
    println(it.readLines())
```

```kt
}
```

对象必须实现`Closeable`接口，`use()`函数才可用。`Closeable`对象将在我们退出`use{}`块时关闭。

# 内联函数

你可以将`inline`函数视为编译器复制和粘贴你的代码的指令。每次编译器看到标记为`inline`关键字的函数调用时，它都会将调用替换为具体的函数体。

如果它是一个接受 lambda 作为其参数之一的高阶函数，那么使用`inline`函数是有意义的。这是你想要使用`inline`的最常见用例。

让我们看看这样的高阶函数，看看编译器将输出什么伪代码。

首先，这是函数定义：

```kt
inline fun logBeforeAfter(block: () -> String) { 
```

```kt
    println("Before") 
```

```kt
    println(block()) 
```

```kt
    println("After") 
```

```kt
}
```

在这里，我们向函数传递一个 lambda，或一个`block`。这个`block`简单地返回单词`"Inlining"`作为一个`String`：

```kt
logBeforeAfter { 
```

```kt
    "Inlining" 
```

```kt
}
```

如果你查看反编译的字节码的 Java 等效版本，你会看到根本就没有调用我们的`makesSense`函数。相反，你会看到以下内容：

```kt
String var1 = "Before"; <- Inline function call 
```

```kt
System.out.println(var1); 
```

```kt
var1 = "Inlining"; 
```

```kt
System.out.println(var1); 
```

```kt
var1 = "After"; 
```

```kt
System.out.println(var1); 
```

由于`inline`函数是代码的复制/粘贴，如果你有超过几行代码，就不应该使用它。将其作为常规函数会更有效率。但如果你有接受 lambda 作为参数的单表达式函数，使用`inline`关键字来优化性能是有意义的。最终，这是应用程序大小和性能之间的权衡。

# 实现代数数据类型

**代数数据类型**，或简称 **ADTs**，是函数式编程中的一个概念，它与我们在 *第三章* *理解结构模式* 中讨论的 **组合设计模式** 非常相似。

为了理解 ADTs 的工作原理以及它们的优点，让我们讨论如何在 Kotlin 中实现一个简单的二叉树。

首先，让我们为我们的树声明一个接口。由于树数据结构可以包含任何类型的数据，我们可以用类型（`T`）来参数化它：

```kt
sealed interface Tree<out T>
```

类型用 `out` 关键字标记，这意味着这个类型是 *协变的*。如果你不熟悉这个术语，我们将在实现接口时再讨论它。

协变的对立面是 *逆变的*。逆变类型应该使用 `in` 关键字标记。

我们还可以用 `sealed` 关键字标记这个接口。我们在 *第四章* *熟悉行为模式* 中讨论了将此关键字应用于常规类，而 `sealed` 接口是一个相对较新的特性，它是在 **Kotlin 1.5** 中引入的。

意义是相同的：只有接口的所有者才能实现它。这意味着在编译时就可以知道接口的所有实现。

接下来，让我们声明一个空树看起来像什么：

```kt
object Empty : Tree<Nothing> {
```

```kt
    override fun toString() = "Empty"
```

```kt
}
```

由于所有空树都是相同的，我们将其声明为一个对象。这是 `Nothing` 作为空树类型的另一个用途。这是 Kotlin 对象层次结构中的一个特殊类。

重要提示：

在 `Any` 和 `Nothing` 之间存在一些混淆，`Any` 代表任何类，类似于 Java 中的 `Object`，而 `Nothing` 代表没有类。我们将在本章后面看到为什么 `Any` 在这个情况下不起作用。

接下来，让我们定义一个非空树节点：

```kt
data class Node<T>(
```

```kt
    val value: T,
```

```kt
    val left: Tree<T> = Empty,
```

```kt
    val right: Tree<T> = Empty
```

```kt
) : Tree<T>
```

`Node` 也实现了 `Tree` 接口，但它是一个数据类，而不是一个对象，因为每个节点都是不同的。`Node` 的值类型是 `T`，这意味着它可以包含任何类型的值，但同一树中的所有节点都将包含相同类型的值。这是泛型真正的力量。

节点也有两个子节点，左和右，因为它是一个二叉树。默认情况下，它们都是空的。

由于类型是协变的，并且 `Empty` 是 `Nothing` 类型，我们可以指定节点子节点的默认值。`Nothing` 在类层次结构的底部，而 `Any` 在顶部。

当我们声明 `Tree` 的类型为 `out T` 时，我们的意思是 `Tree` 可以包含类型 `T` 的值或继承自该类型的任何值。

由于 `Nothing` 在类层次结构的底部，它 *继承* 自所有类型。

现在一切都已经设置好了，让我们学习如何创建我们刚刚定义的树的实例：

```kt
val tree = Node(
```

```kt
    1,
```

```kt
    Empty,
```

```kt
    Node(
```

```kt
        2,
```

```kt
        Node(3)
```

```kt
    )
```

```kt
)
```

```kt
println(tree)
```

在这里，我们创建了一个以 **1** 为根节点值和具有 **2** 值的右节点的树。右节点有一个值为 **3** 的左子节点。这就是我们的树看起来像什么：

![图 9.1 – 树形图](img/B17816_09_01.jpg)

图 9.1 – 树形图

上述代码输出以下内容：

```kt
> Node(value=1, left=Empty, right=Node(value=2, left=Node(value=3, left=Empty, right=Empty), right=Empty))
```

然而，以这种形式打印树并不很有趣。所以，让我们实现一个函数，如果树是数值型的，它会总结树的所有节点：

```kt
fun Tree<Int>.sum(): Long = when (this) {
```

```kt
    Empty -> 0
```

```kt
    is Node -> value + left.sum() + right.sum()
```

```kt
}
```

这也称为 ADT 上的**操作**。这是一个仅在包含整数的树上声明的扩展函数。

对于每个节点，我们检查它是否是`Empty`或`Node`。这就是`sealed`类和接口的美丽之处。由于编译器知道`Tree`接口恰好有两个实现，因此我们不需要在`when`表达式中使用`else`块。

如果它是一个`Empty`节点，我们使用`0`作为中性值。如果它不为空，那么我们将它的值与其左右子节点的值相加。

这个函数也是递归算法的另一个例子，我们在*第五章*，“介绍函数式编程”中讨论过。

现在，让我们讨论与 Kotlin 中的泛型相关的话题。

# 实化泛型

在本章前面，我们提到了`inline`函数。由于`inline`函数被复制，我们可以消除 JVM 的一个主要限制：**类型擦除**。毕竟，在函数内部，我们知道我们得到的确切类型。

让我们看看以下示例。我们希望创建一个泛型函数，该函数将接收一个`Number`（`Number`可以是`Int`或`Long`），但只有当它的类型与函数类型相同时才会打印它。

我们将从一种简单的实现开始，直接尝试在类型上执行实例检查：

```kt
fun <T> printIfSameType(a: Number) { 
```

```kt
    if (a is T) { // <== Error 
```

```kt
        println(a)    
```

```kt
    } 
```

```kt
}
```

然而，这段代码无法编译，我们会得到以下错误：

```kt
> Cannot check for instance of erased type: T
```

在这种情况下，我们通常在 Java 中传递类作为参数。我们可以在 Kotlin 中尝试类似的方法。如果你之前使用过 Android，你会立即认出这个模式，因为它在标准库中用得很多：

```kt
fun <T : Number> printIfSameType(clazz: KClass<T>, a: 
```

```kt
  Number) {
```

```kt
    if (clazz.isInstance(a)) {
```

```kt
        println("Yes")
```

```kt
    } else {
```

```kt
        println("No")
```

```kt
    }
```

```kt
}
```

我们可以通过运行以下行来检查代码是否正确工作：

```kt
printIfSameType(Int::class, 1) // Prints yes, as 1 is Int
```

```kt
printIfSameType(Int::class, 2L) // Prints no, as 2 is Long
```

```kt
printIfSameType(Long::class, 3L) // Prints yes, as 3 is Long
```

这段代码可以工作，但有一些缺点：

+   我们不能使用`is`运算符，而必须使用`isInstance()`函数。

+   我们必须传递正确的类；即`clazz: KClass<T>`。

这段代码可以通过使用`reified`类型来改进：

```kt
inline fun <reified T : Number> printIfSameReified(a:   Number) {
```

```kt
    if (a is T) {
```

```kt
        println("Yes")
```

```kt
    } else {
```

```kt
        println("No")
```

```kt
    }
```

```kt
}
```

这个函数与上一个函数的工作方式相同，但不需要输入一个类即可工作。使用`reified`类型的函数必须声明为`inline`。这是由于 JVM 上的类型擦除。

我们可以通过以下方式测试我们的代码是否仍然按预期工作：

```kt
printIfSameReified<Int>(1) // Prints yes, as 1 is Int
```

```kt
printIfSameReified<Int>(2L) // Prints no, as 2 is Long
```

```kt
printIfSameReified<Long>(3L) // Prints yes, as 3 is Long
```

注意，现在，我们在*尖括号*中指定函数操作的类型，例如`Int`或`Long`，而不是将其作为参数传递给函数。使用`reified`函数我们可以获得以下好处：

+   清晰的方法签名，无需传递一个类作为参数。

+   在函数内部使用`is`构造的能力。

+   它对类型推断友好，这意味着如果类型参数可以被编译器推断出来，它就可以完全省略。

当然，常规 `inline` 函数的相同规则也适用于这里。这段代码将被复制，所以它不应该太大。

现在，让我们考虑 `reified` 类型的另一个用例——**函数重载**。我们将尝试定义两个具有相同名称但操作类型不同的函数：

```kt
fun printList(list: List<Int>) { 
```

```kt
    println("This is a list of Ints") 
```

```kt
    println(list) 
```

```kt
} 
```

```kt
fun printList(list: List<Long>) { 
```

```kt
    println("This is a list of Longs") 
```

```kt
    println(list) 
```

```kt
}
```

这将无法编译，因为存在平台声明冲突。它们在 JVM 方面具有相同的签名：`printList(list: List)`。这是因为类型在编译过程中被擦除。

但使用 `reified`，我们可以轻松实现这一点：

```kt
inline fun <reified T : Any> printList(list: List<T>) {
```

```kt
    when {
```

```kt
        1 is T -> println("This is a list of Ints")
```

```kt
        1L is T -> println("This is a list of Longs")
```

```kt
        else -> println("This is a list of something else")
```

```kt
    }
```

```kt
    println(list)
```

```kt
}
```

由于整个函数是**内联**的，我们可以检查列表的实际类型并输出正确的结果。

# 高效使用常量

由于 Java 中的所有内容都是对象（除非是原始类型），我们习惯于将所有常量作为静态成员放入我们的对象中。

由于 Kotlin 有 `companion` 对象，我们通常尝试将它们放在那里：

```kt
class Spock {
```

```kt
    companion object {
```

```kt
        val SENSE_OF_HUMOR = "None"
```

```kt
    }
```

```kt
}
```

这将有效，但你应该记住，`companion object` 仍然是一个对象。

因此，这将大致转换为以下代码：

```kt
public class Spock {
```

```kt
    private static final String SENSE_OF_HUMOR = "None";
```

```kt
    public String getSENSE_OF_HUMOR() {
```

```kt
        return Spock.SENSE_OF_HUMOR;
```

```kt
    }
```

```kt
    ...
```

```kt
}
```

在这个例子中，Kotlin 编译器为我们的常量生成一个 getter，这增加了另一个间接层。

如果我们查看使用常量的代码，我们会看到以下内容：

```kt
String var0 = Spock.Companion.getSENSE_OF_HUMOR();
```

```kt
System.out.println(var0);
```

我们可以调用一个方法来获取常量值，这并不高效。

现在，让我们将此值标记为常量，看看编译器生成的代码如何变化：

```kt
class Spock { 
```

```kt
    companion object { 
```

```kt
        const val SENSE_OF_HUMOR = "None" 
```

```kt
    } 
```

```kt
}
```

这里是字节码的变化：

```kt
public class Spock { 
```

```kt
   public static final String SENSE_OF_HUMOR = "None"; 
```

```kt
   ...
```

```kt
}
```

下面是这个调用的示例：

```kt
String var1 = "None"; 
```

```kt
System.out.println(var1);
```

注意，代码中已经没有 `Spock` 类的引用了。编译器已经为我们**内联**了它的值。毕竟，它是常量，所以它永远不会改变，可以安全地**内联**。

如果你只需要一个常量，你也可以在类外部设置它：

```kt
const val SPOCK_SENSE_OF_HUMOR = "NONE"
```

如果你需要命名空间，你可以将其包裹在一个对象中：

```kt
object SensesOfHumor { 
```

```kt
    const val SPOCK = "None" 
```

```kt
}
```

现在我们已经学会了如何更有效地使用常量，让我们学习如何以惯用的方式处理构造函数。

# 构造函数重载

在 Java 中，我们习惯于使用重载构造函数。例如，让我们看看以下需要 `a` 参数并将 `b` 的值默认设置为 `1` 的 Java 类：

```kt
class User { 
```

```kt
    private final String name; 
```

```kt
    private final boolean resetPassword; 
```

```kt
    public User(String name) { 
```

```kt
        this(name, true); 
```

```kt
    } 
```

```kt
    public User(String name, boolean resetPassword) { 
```

```kt
        this.name = name; 
```

```kt
        this.resetPassword = resetPassword; 
```

```kt
    } 
```

```kt
}
```

我们可以通过使用 `constructor` 关键字定义多个构造函数来在 Kotlin 中模拟相同的行为：

```kt
class User(val name: String, val resetPassword: Boolean) {
```

```kt
    constructor(name: String) : this(name, true)
```

```kt
}
```

次构造函数，如类体中定义的，将调用主构造函数，为第二个参数提供默认值 `1`。

然而，通常最好有默认参数值和命名参数：

```kt
class User(val name: String, val resetPassword: Boolean =   true)
```

注意，所有次构造函数都必须使用 `this` 关键字委托给主构造函数。唯一的例外是当你有一个默认的主构造函数：

```kt
class User {
```

```kt
    val resetPassword: Boolean
```

```kt
    val name: String
```

```kt
    constructor(name: String, resetPassword: Boolean = 
```

```kt
      true) {
```

```kt
        this.name = name
```

```kt
        this.resetPassword = resetPassword
```

```kt
    }
```

```kt
}
```

接下来，让我们讨论如何在 Kotlin 代码中高效地处理空值。

# 处理空值

Kotlin 中的 `null`；例如：

```kt
// Will return "String" half of the time and null the other 
```

```kt
// half 
```

```kt
val stringOrNull: String? = if (Random.nextBoolean()) 
```

```kt
  "String" else null  
```

```kt
// Java-way check 
```

```kt
if (stringOrNull != null) { 
```

```kt
    println(stringOrNull.length) 
```

```kt
}
```

我们可以使用 `Elvis` 操作符（`?:`）重写此代码：

```kt
val alwaysLength = stringOrNull?.length ?: 0 
```

如果长度不是 `null`，这个操作符将返回其值。否则，它将返回我们提供的默认值，在这种情况下是 `0`。

如果你有一个嵌套对象，你可以链式调用这些检查。例如，让我们有一个包含 `Profile` 的 `Response` 对象，而 `Profile` 又包含名字和姓氏字段，这些字段可能是可空的：

```kt
data class Response(
```

```kt
    val profile: UserProfile?
```

```kt
)
```

```kt
data class UserProfile(
```

```kt
    val firstName: String?,
```

```kt
    val lastName: String?
```

```kt
)
```

这种链式调用将看起来像这样：

```kt
val response: Response? = Response(UserProfile(null, null))
```

```kt
println(response?.profile?.firstName?.length)
```

如果链中的任何字段为空，我们的代码不会崩溃。相反，它会打印 `null`。

最后，你可以使用 `let()` 块来进行空检查，正如我们在 *使用作用域函数* 部分简要提到的。使用 `let()` 函数的相同代码将看起来像这样：

```kt
println(response?.let { 
```

```kt
    it.profile?.let { 
```

```kt
        it.firstName?.length 
```

```kt
    } 
```

```kt
})
```

如果你想要在所有地方去掉 `it`，你可以使用另一个作用域函数，`run()`：

```kt
println(response?.run { 
```

```kt
    profile?.run { 
```

```kt
        firstName?.length 
```

```kt
    } 
```

```kt
})
```

尽量避免在生产代码中使用不安全的 `!!` 空操作符：

```kt
println(json!!.User!!.firstName!!.length)
```

这将导致 `KotlinNullPointerException`。然而，在测试期间，`!!` 操作符可能很有用，因为它可以帮助你更快地发现空安全的问题。

# 明确异步性

正如你在上一章中看到的，在 Kotlin 中创建异步函数非常简单。以下是一个示例：

```kt
fun CoroutineScope.getResult() = async { 
```

```kt
   delay(100) 
```

```kt
   "OK" 
```

```kt
}
```

然而，这种异步性可能对函数的用户来说是一个意外的行为，因为他们可能期望一个简单的值。

*你认为以下代码会打印什么？*

```kt
println("${getResult()}")
```

对于用户来说，前面的代码有些意外地打印了以下内容而不是 `"OK"`：

```kt
> Name: DeferredCoroutine{Active}@...
```

当然，如果你已经阅读了 *第六章*，*线程和协程*，你就会知道这里缺少的是 `await()` 函数：

```kt
println("${getResult().await()}")
```

但如果我们给我们的函数添加一个 `async` 后缀，那么它就会更加明显：

```kt
fun CoroutineScope.getResultAsync() = async { 
```

```kt
   delay(100) 
```

```kt
   "OK" 
```

```kt
}
```

Kotlin 的约定是在函数名末尾添加 `Async` 来区分异步函数和常规函数。如果你在使用 **IntelliJ IDEA**，它甚至会建议你重命名它。

现在，让我们来谈谈一些用于验证用户输入的内置函数。

# 验证输入

输入验证是一项必要但非常繁琐的任务。*你有多少次不得不编写如下代码？*

```kt
fun setCapacity(cap: Int) { 
```

```kt
    if (cap < 0) { 
```

```kt
        throw IllegalArgumentException() 
```

```kt
    } 
```

```kt
    ... 
```

```kt
}
```

相反，你可以使用 `require()` 函数来检查参数：

```kt
fun setCapacity(cap: Int) { 
```

```kt
    require(cap > 0) 
```

```kt
}
```

这使得代码更加流畅。你可以使用 `require()` 来检查空值：

```kt
fun printNameLength(p: Profile) { 
```

```kt
    require(p.firstName != null) 
```

```kt
}
```

但也有 `requireNotNull()` 可以做到这一点：

```kt
fun printNameLength(p: Profile) { 
```

```kt
    requireNotNull(p.firstName) 
```

```kt
}
```

使用 `check()` 来验证你对象的状态。这在提供用户可能没有正确设置的对象时非常有用：

```kt
class HttpClient { 
```

```kt
    var body: String? = null 
```

```kt
    var url: String = "" 
```

```kt
    fun postRequest() { 
```

```kt
        check(body != null) { 
```

```kt
            "Body must be set in POST requests" 
```

```kt
        } 
```

```kt
    } 
```

```kt
}
```

再次提醒，这里也有一个快捷方式：`checkNotNull()`。

`require()` 和 `check()` 函数之间的区别在于 `require()` 会抛出 `IllegalArgumentException`，这意味着提供给函数的输入是错误的。另一方面，`check()` 会抛出 `IllegalStateException`，这意味着对象的状态已被破坏。

考虑使用 `require()` 和 `check()` 函数来提高你代码的可读性。

最后，让我们讨论如何在 Kotlin 中有效地表示不同的状态。

# 优先考虑 `sealed` 类而不是枚举

来自 Java 的你可能会倾向于在 `enum` 上添加功能。

例如，假设你开发了一个允许用户订购披萨并跟踪其状态的程序。我们可以使用以下代码来完成这个任务：

```kt
// Java-like code that uses enum to represent State
```

```kt
enum class PizzaOrderStatus {
```

```kt
    ORDER_RECEIVED, PIZZA_BEING_MADE, OUT_FOR_DELIVERY,      COMPLETED;
```

```kt
    fun nextStatus(): PizzaOrderStatus {
```

```kt
        return when (this) {
```

```kt
            ORDER_RECEIVED -> PIZZA_BEING_MADE
```

```kt
            PIZZA_BEING_MADE -> OUT_FOR_DELIVERY
```

```kt
            OUT_FOR_DELIVERY -> COMPLETED
```

```kt
            COMPLETED -> COMPLETED
```

```kt
        }
```

```kt
    }
```

```kt
}
```

或者，你可以使用 `sealed` 类：

```kt
sealed class PizzaOrderStatus(protected val orderId: Int) {
```

```kt
    abstract fun nextStatus(): PizzaOrderStatus
```

```kt
}
```

```kt
class OrderReceived(orderId: Int) : 
```

```kt
  PizzaOrderStatus(orderId) {
```

```kt
    override fun nextStatus() = PizzaBeingMade(orderId)
```

```kt
}
```

```kt
class PizzaBeingMade(orderId: Int) : 
```

```kt
  PizzaOrderStatus(orderId) {
```

```kt
    override fun nextStatus() = OutForDelivery(orderId)
```

```kt
}
```

```kt
class OutForDelivery(orderId: Int) : 
```

```kt
  PizzaOrderStatus(orderId) {
```

```kt
    override fun nextStatus() = Completed(orderId)
```

```kt
}
```

```kt
class Completed(orderId: Int) : PizzaOrderStatus(orderId) {
```

```kt
    override fun nextStatus() = this
```

```kt
}
```

在这里，我们为每个对象状态创建了一个单独的类，这些类扩展了 `PizzaOrderStatus` `sealed` 类。

这种方法的优点是，我们现在可以更容易地存储状态及其 `status`。在我们的例子中，我们可以存储订单的 ID：

```kt
var status: PizzaOrderStatus = OrderReceived(123) 
```

```kt
while (status !is Completed) { 
```

```kt
    status = when (status) { 
```

```kt
        is OrderReceived -> status.nextStatus() 
```

```kt
        is PizzaBeingMade -> status.nextStatus() 
```

```kt
        is OutForDelivery -> status.nextStatus() 
```

```kt
        is Completed -> status 
```

```kt
    } 
```

```kt
}
```

通常情况下，如果你想要与状态关联数据，`sealed` 类是好的选择，你应该优先考虑它们而不是枚举。

# 摘要

在本章中，我们回顾了 Kotlin 的最佳实践以及语言的一些注意事项。现在，你应该能够编写更符合语言习惯、性能良好且易于维护的代码。

在必要时，你应该使用作用域函数，但确保不要过度使用它们，因为它们可能会使代码变得混乱，尤其是对于那些刚开始使用该语言的人来说。

一定要正确处理空值和类型转换，使用 `let()`、`Elvis` 操作符以及语言提供的智能转换功能。最后，泛型和 `sealed` 类和接口是描述不同类之间复杂关系和行为的有力工具。

在下一章中，我们将通过编写实际的微服务响应式设计模式来应用这些技能。

# 问题

1.  Kotlin 中 Java 的 try-with-resources 的替代方案是什么？

1.  Kotlin 中处理空值的不同选项有哪些？

1.  重新实例化的泛型可以解决哪些问题？
