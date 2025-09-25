# 使用 Kotlin 函数式编程特性塑造代码

在本章中，我们将涵盖以下菜谱：

+   高效使用 lambda 表达式

+   发现基本的范围函数——`let`、`also`和`apply`

+   使用`run`范围函数以干净的方式初始化对象

+   使用高阶函数

+   函数柯里化

+   函数组合

+   实现 Either Monad 设计模式

+   自动函数记忆化的方法

# 简介

尽管 Kotlin 被隐式地认可为一种面向对象的语言，但它仍然对其他编程风格和范式持开放态度。多亏了 Kotlin 的内置特性，我们能够轻松地将函数式编程模式应用到我们的代码中。能够从其他函数中返回函数或传递一个函数作为参数，使我们能够从延迟计算中受益。此外，我们能够在代码的不同层返回函数，而不是已经计算出的值。这导致了懒加载特性的出现。

与 Scala 或其他函数式编程语言相比，Kotlin 不需要我们使用专门的函数式风格设计模式。它也缺乏它们的一些现成实现。然而，作为回报，它在软件架构和实现细节方面为开发者提供了更多的灵活性。Kotlin 语言和标准库组件为基本的函数式编程概念提供了全面内置支持。更复杂的概念总是可以从头实现或从一些可用的外部库中重用。值得一试的项目有 Kotlin Arrow ([`arrow-kt.io`](http://arrow-kt.io)) 和 funKTionale ([`github.com/MarioAriasC/funKTionale`](https://github.com/MarioAriasC/funKTionale))。然而，请记住罗伯特·C·马丁的话——*编写既面向对象又函数式的程序是完全可能的。这不仅可能，而且是可取的。不存在“OO vs FP”，两者是正交的，并且很好地共存*<q>。</q> 应该理解，函数式编程只是可用的工具之一。它应该被明智地使用，并且仅在适用的情况下使用。

本章重点介绍 Kotlin 内部支持的函数式编程特性。它通过使用最先进的函数式编程概念来解决实际问题，为您提供了实践经验。到本章结束时，您应该熟悉 Kotlin 语言对函数式编程方法的支持以及可以帮助实现它的标准库组件。

# 高效使用 lambda 表达式

在这个菜谱中，我们将探讨 lambda 和闭包的概念。我们将编写一部分 Android 应用程序代码，负责处理按钮点击操作。

# 准备工作

为了实现这个菜谱的代码，您需要使用 Android Studio IDE 创建一个新的 Android 应用程序项目。

假设我们有一个以下类，它是一种应用视图层的控制器：

```kt
class RegistrationScreen : Activity() {
    private val submitButton: Button by lazy { findViewById(R.id.submit_button) }  

    override fun onCreate(savedInstanceState: Bundle?) {
```

```kt
        // hook function called once the screen is displayed
    }
}
```

它包含对 `submitButton: Button` 实例的引用。在 `onCreate()` 函数中，我们将实现处理按钮点击的逻辑。一旦按钮被点击，我们希望使其不可见。

为了在按钮点击时执行某些操作，我们需要在 `View` 子类上调用 `View.setOnClickListener(listener: OnClickListener)` 函数。`OnClickListener` 是一个如下定义的函数式接口：

```kt
public interface OnClickListener {
    void onClick(View view);
}
```

在底层，当用户点击视图时，Android 操作系统会调用 `onClick()` 函数。

在 Kotlin 中实现函数式接口有两种方式：

+   定义实现接口的对象：

```kt
val myInterfaceInstance = object: MyInterface {
    override fun foo() {
        // foo function body
    }
}
```

+   将接口作为函数处理并实现它，例如，以 lambda 的形式：

```kt
val myInterfaceAsFunction: () -> Unit = { 
    // foo function body
}
```

# 如何实现...

1.  调用 `setOnClickListener` 函数，并传递一个空的 `OnClickListener` 实例作为 lambda 表达式：

```kt
class RegistrationScreen : Activity() {
    private val submitButton: Button by lazy { findViewById(R.id.submit_button) }  

    override fun onCreate(savedInstanceState: Bundle?) {
        submitButton.setOnClickListener { view: View ->
            // do something on click
        }
    }
}
```

1.  在函数体内部修改 `submitButton` 实例的可见性：

```kt
class RegistrationScreen : Activity() {
    private val submitButton: Button by lazy { findViewById(R.id.submit_button) }  

    override fun onCreate(savedInstanceState: Bundle?) {
        submitButton.setOnClickListener {
            submitButton.visibility = View.INVISIBLE
        }
    }
}
```

# 它是如何工作的...

由于将 `OnClickListener` 作为函数处理，我们能够以简洁的 lambda 表达式形式实现它。lambda 的主体将在用户点击按钮时全局调用。在我们的例子中，一旦按钮被点击，它将被隐藏起来。

Lambda 表达式是语言中最基本的函数特性之一，在标准库组件中被广泛使用。它们可以看作是函数或函数式接口实现的缩写形式。Lambda 帮助正确组织代码并减少大量样板代码。lambda 表达式的语法可以看作是放置在 `{` `}` 符号之间的代码块。Lambda 表达式可以显式定义函数参数，例如：

```kt
val myFunction: (View) -> Unit = { view ->
   view.visibility = View.INVISIBLE
}
```

为了简洁起见，可以省略显式参数。然而，我们仍然可以使用 `it` 修饰符来访问它：

```kt
val myFunction: (View) -> Unit = { 
   it.visibility = View.INVISIBLE
}
```

# 还有更多...

当与 lambda 一起工作时，每次我们想要执行它们体内的代码时，我们都需要在它们上调用 `invoke()` 函数或其等效的 `()` 操作符：

```kt
val callback: () -> Unit = { println("The job is done!") }
callback.invoke()
callback()
```

上述代码将打印文本两次：

```kt
"The job is done!"
"The job is done!"
```

还有另一种将函数作为参数传递给其他函数的简洁方式。我们可以使用函数引用来完成：

```kt
fun hideView(view: View): Unit  {
    view.visibility = View.INVISIBLE
}

submitButton.setOnClickListener(::hideView)
```

函数引用方法在代码库中重用函数实现时特别有用。

# 发现基本的作用域函数 – let, also, apply

在这个菜谱中，我们将探索标准库中的三个有用的扩展函数—`let`、`also` 和 `apply`。它们与 lambda 表达式配合得很好，有助于编写干净和安全的代码。我们将练习它们的用法，同时将它们应用于实现一系列数据处理操作。

# 准备工作

假设我们可以使用以下函数获取日期：

```kt
fun getPlayers(): List<Player>? 
```

这里，`Player` 类是这样定义的：

```kt
data class Player(val name: String, val bestScore: Int)
```

我们希望对 `getPlayers()` 函数的结果执行以下操作序列：

1.  将列表中的原始玩家集打印到控制台

1.  按降序对 `Player` 对象集合进行排序

1.  将 `Player` 集合转换为由 `Player.name` 属性获取的字符串列表

1.  将集合限制为第一个元素并打印到控制台

为了完成这个任务，首先，我们需要熟悉 `let`、`also` 和 `apply` 函数的特性。它们作为泛型类型的扩展函数提供在标准库中。让我们探索 `let`、`also` 和 `apply` 函数的头部：

```kt
public inline fun <T, R> T.let(block: (T) -> R): R

public inline fun <T> T.also(block: (T) -> Unit): T

public inline fun <T> T.apply(block: T.() -> Unit): T 
```

它们看起来很相似，然而，在返回类型和参数方面有一些细微的差异。以下表格比较了这三个函数：

| **函数** | **返回类型** | **块参数中的参数** | **块参数定义** |
| --- | --- | --- | --- |
| `let` | `R` (从块体) | 显式 it | `(T) -> R` |
| `also` | `T` (this) | 显式 it | `(T) -> Unit` |
| `apply` | `T` (this) | 隐式 this | `T.() -> Unit` |

# 如何做到这一点...

1.  将 `let` 函数与安全运算符一起使用以确保空安全：

```kt
getPlayers()?.let {}
```

1.  在 `let` 函数的 lambda 参数块内部，使用 `also()` 函数将列表中的原始玩家集打印到控制台：

```kt
getPlayers()?.let {
 it.also {
        println("${it.size} players records fetched")
 println(it)
 }
}
```

1.  使用 `let()` 函数执行排序和映射转换：

```kt
getPlayers()?.let {
    it.also {
        println("${it.size} players records fetched")
        println(it)
    }.let {
        it.sortedByDescending { it.bestScore }
    }
```

1.  使用 `let()` 函数将玩家集合限制为具有最高分数的单个 `Player` 实例：

```kt
getPlayers()?.let {
    it.also {
        println("${it.size} players records fetched")
        println(it)
    }.let {
        it.sortedByDescending { it.bestScore }
    }.let {
        it.first()
 }
```

1.  将最佳玩家的名字打印到控制台：

```kt
getPlayers()?.let {
    it.also {
        println("${it.size} players records fetched")
        println(it)
    }.let {
        it.sortedByDescending { it.bestScore }
    }.let {
        it.first()
    }.apply {
        val name = this.name
        print("Best Player: $name")
    }
}
```

# 它是如何工作的...

让我们测试我们的实现。为了测试的目的，我们可以假设 `getPlayers()` 函数返回以下结果：

```kt

fun getPlayers(): List<Player>? = listOf(
        Player("Stefan Madej", 109),
        Player("Adam Ondra", 323),
        Player("Chris Charma", 239))
```

我们实现的代码将打印以下输出到控制台：

```kt
3 players records fetched
[Player(name=Stefan Madej, bestScore=109), Player(name=Adam Ondra, bestScore=323), Player(name=Chris Charma, bestScore=239)]
Best Player: Adam Ondra
```

注意，在 `apply()` 函数的情况下，在函数 lambda 块内部访问类属性和函数时，我们可以省略 `this` 关键字：

```kt
apply {
    print("Best Player: $name")
}
```

这只是为了在示例代码中提高清晰度而使用。

`let()` 函数的有用特性是它可以用来确保给定对象的空安全。在以下 `let` 范围内的示例中，即使某些后台线程试图修改可变 `results` 变量的原始值，`players` 参数也始终持有非空值：

```kt
var result: List<Player>? = getPlayers()
result?.let { players: List<Player> ->
    ...
}
```

# 参见

+   如果你想了解更多关于 lambda 表达式的内容，请查看 *有效使用 lambda 表达式* 菜谱

# 使用 `run` 范围函数以干净的方式初始化对象

在这个菜谱中，我们将探索标准库提供的另一个有用的扩展函数，称为 `run()`。我们将使用它来创建和设置 `java.util.Calendar` 类的实例。

# 准备工作

首先，让我们通过以下函数头探索标准库中定义的 `run()` 函数的特性：

```kt
public inline fun <T, R> T.run(block: T.() -> R): R
```

它被声明为一个泛型类型的扩展函数。`run`函数在`block`参数内部提供了隐式的`this`参数，并返回`block`执行的返回值。

# 如何实现...

1.  声明`Calendar.Builder`类的实例并将其`run()`函数应用于它：

```kt
val calendar = Calendar.Builder().run {
    build()
}
```

1.  将所需的属性添加到构建器中：

```kt
val calendar = Calendar.Builder().run {
    setCalendarType("iso8601")
 setDate(2018, 1, 18)
 setTimeZone(TimeZone.getTimeZone("GMT-8:00"))
    build()
}
```

1.  将日历中的日期打印到控制台：

```kt
val calendar = Calendar.Builder().run {
    setCalendarType("iso8601")
    setDate(2018, 1, 18)
    setTimeZone(TimeZone.getTimeZone("GMT-8:00"))
    build()
}
print(calendar.time)
```

# 它是如何工作的...

`run`函数应用于`Calendar.Builder`实例。在传递给`run`函数的 lambda 内部，我们可以通过`this`修饰符访问`Calendar.Builder`的属性和方法。换句话说，在`run`函数块内部，我们正在访问`Calendar.Builder`实例的作用域。在食谱代码中，我们省略了使用`this`关键字调用`Builder`方法。我们可以直接调用它们，因为`run`函数允许通过隐式的`this`修饰符在其作用域内访问`Builder`实例。

# 还有更多...

我们还可以将`run()`函数与安全的`?`运算符一起使用，以提供`run()`函数作用域内`this`关键字引用的对象的 null 安全性。您可以在以下配置 Android `WebView`类的示例中看到它的实际应用：

```kt
webview.settings?.run {
    this.javaScriptEnabled = true
    this.domStorageEnabled = false
}
```

在前面的代码片段中，我们确保在`run`函数的作用域内`settings`属性不为 null，并且我们可以通过`this`关键字访问它。

# 相关内容

+   Kotlin 标准库提供了另一个类似的扩展函数，称为`apply()`，它对于对象的初始化非常有用。主要区别在于它返回调用它的对象的原始实例。您可以在第五章的*“以美味的方式实现构建器”*食谱中探索它，*采用 Kotlin 概念的优雅设计模式*。

# 使用高阶函数

Kotlin 被设计为提供对函数的一等支持。例如，我们能够轻松地将函数作为参数传递给另一个函数。我们还可以创建一个可以返回另一个函数的函数。这种类型的函数称为*高阶函数*。这个强大的功能有助于轻松编写函数式风格的代码。返回函数而不是值的能力，加上将函数实例作为参数传递给另一个函数的能力，使得延迟计算和清晰塑造代码成为可能。在本食谱中，我们将实现一个辅助函数，该函数将测量传递给它作为参数的其他函数的执行时间。

# 如何实现...

实现`measureTime`函数：

```kt
fun measureTime(block: () -> Unit): Long {
    val start = System.currentTimeMillis()
    block()
    val end = System.currentTimeMillis()

    return end - start
}
```

# 它是如何工作的...

`measureTime()`函数接受一个名为`block`的函数类型参数。`block`函数参数在`measureTime()`函数内部使用`()`修饰符调用。最后，返回时间戳（在块执行前后）之间的差异。

让我们分析以下示例，展示`measureTime()`函数的实际应用。我们可以考虑以下函数负责计算给定整数的阶乘：

```kt
fun factorial(n: Int): Long {
    sleep(10)
    return if (n == 1) n.toLong() else n * factorial(n - 1)
}
```

为了测量`factorial()`函数的执行时间，我们可以使用`measureTime()`函数如下：

```kt
val duration = measureTime {
    factorial(13)
}
print("$duration ms")
```

结果，我们得到打印到控制台上的执行时间：

```kt
154 ms
```

注意，也可以将函数引用而不是 lambda 实例作为`measureTime()`函数的参数传递：

```kt
fun foo() = sleep(1000)
val duration = measureTime(::foo)
print("$duration ms")
```

# 函数柯里化

柯里化是函数式编程中的一种常见技术。它允许将给定的多参数函数转换为一串函数，每个函数只有一个参数。每个生成的函数处理原始（未柯里化）函数的一个参数，并返回另一个函数。

在这个菜谱中，我们将实现一个可以应用于任何接受三个参数的函数的自动柯里化机制。

# 准备工作

为了理解函数柯里化的概念，让我们考虑以下处理三个参数的函数的示例：

```kt
fun foo(a: A, b: B, c: C): D 
```

它的柯里化形式看起来是这样的：

```kt
fun carriedFoo(a: A): (B) -> (C) -> D 
```

换句话说，`foo`函数的柯里化形式将接受一个`A`类型的单个参数，并返回一个以下类型的函数：`(B) -> (C) -> D`。返回的函数负责处理原始函数的第二个参数，并返回另一个函数，该函数接受第三个参数并返回一个类型为`D`的值。

在下一节中，我们将实现`curried()`扩展函数，该函数用于以下声明的泛型函数类型：`<P1, P2, P3, R> ((P1, P2, P3))`。`curried()`函数将返回一个单参数函数链，并将适用于任何接受三个参数的函数。

# 如何做到这一点...

1.  声明`curried()`函数的标题：

```kt
fun <P1, P2, P3, R> ((P1, P2, P3) -> R).curried(): (P1) -> (P2) -> (P3) -> R 
```

1.  实现函数`curried()`的主体：

```kt
fun <P1, P2, P3, R> ((P1, P2, P3) -> R).curried(): (P1) -> (P2) -> (P3) -> R =
        { p1: P1 ->
            { p2: P2 ->
                { p3: P3 ->
                    this(p1, p2, p3)
 }
             }
        }
```

# 它是如何工作的...

让我们探索如何在实际中使用`curried()`函数。在以下示例中，我们将对以下函数实例调用`curried()`，该实例负责计算三个整数的和：

```kt
fun sum(a: Int, b: Int, c: Int): Int = a + b + c
```

为了获得`sum()`函数的柯里化形式，我们必须在它的引用上调用`curried()`函数：

```kt
::sum.curried()
```

然后，我们可以以以下方式调用柯里化的求和函数：

```kt
val result: Int = ::sum.curried()(1)(2)(3)
```

最后，`result`变量将被分配一个等于`6`的整数值。

为了调用`curried()`扩展函数，我们使用`::`修饰符访问`sum()`函数引用。然后，我们依次调用由柯里化函数返回的函数序列中的下一个函数。

之前的代码可以用更冗长的形式来写，带有显式的类型声明：

```kt
val sum3: (a: Int) -> (b: Int) -> (c: Int) -> Int = ::sum.curried()
val sum2: (b: Int) -> (c: Int) -> Int = sum3(1)
val sum1: (c: Int) -> Int = sum2(2)
val result: Int = sum1(3)
```

在幕后，柯里化机制的实现只是返回嵌套在彼此内部的函数。每次调用特定函数时，它都会返回另一个具有减少一个参数的函数。

# 还有更多...

有一个类似的模式称为*部分应用*。它比柯里化更灵活，因为它不限制每个函数处理的参数数量。例如，给定一个如下声明的`foo`函数：

```kt
fun foo(a: A, b: B, c: C): D 
```

我们可以将其转换为以下形式：

```kt
fun foo(a: A, c: C) -> (B) -> D
```

当我们无法在当前作用域中为函数提供所需的所有参数时，柯里化和部分应用都非常有用。我们可以只将可用的参数应用到函数上，并返回转换后的函数。

# 函数组合

在*函数柯里化*配方中，我们发现了一种巧妙的方法，可以从函数中提取新的函数。在这个配方中，我们将实现相反的转换。有一个选项可以合并多个现有函数的声明，并从中定义一个新的函数，这是一种常见的函数式编程模式，称为*函数组合*。Kotlin 没有提供内置的函数组合机制。然而，多亏了对函数类型操作扩展的内置支持，我们能够手动实现一个可重用的组合机制。

# 准备工作

为了熟悉函数组合，让我们研究以下示例。假设我们定义了以下函数：

```kt
fun length(word: String) = word.length
fun isEven(x:Int): Boolean = x.rem(2) == 0
```

第一个函数负责返回给定字符串的长度。第二个函数检查给定的整数是否为偶数。为了定义一个基于这两个函数的新函数，我们可以进行嵌套函数调用：

```kt
fun isCharCountEven(word: String): Boolean = isEven(length(word))
```

然而，如果我们能够对函数引用进行操作，那就更好了。为了使其更加简洁，我们希望能够使用以下语法声明`isCharCountEven()`函数，用于函数组合：

```kt
val isCharCountEven: (word: String) -> Boolean = ::length and ::isEven
```

# 如何实现...

1.  声明一个名为`and()`的单参数函数的`infix`扩展函数：

```kt
infix fun <P1, R, R2> ((P1) -> R).and(function: (R) -> R2): (P1) -> R2 = {

}
```

1.  内部调用基本函数和`and()`函数的参数：

```kt
infix fun <P1, R, R2> ((P1) -> R).and(function: (R) -> R2): (P1) -> R2 = {
    function(this(it))
}
```

# 它是如何工作的...

为了探索我们的函数组合实现，让我们使用`and()`函数，通过`length()`属性和`isEven()`函数组合`isCharCountEven()`函数：

```kt
fun length(word: String) = word.length
fun isEven(x:Int): Boolean = x.rem(2) == 0
val isCharCountEven: (word: String) -> Boolean = ::length and ::isEven
print(isCharCountEven("pneumonoultramicroscopicsilicovolcanoconiosis"))
```

上述代码将返回以下输出：

```kt
false
```

在底层，`and()`扩展函数只是依次调用给定的两个函数。然而，多亏了中缀表示法，我们可以在代码中执行组合，同时避免嵌套函数调用。此外，前一个示例中`::length and ::isEven`调用的结果返回一个新的函数实例，它可以像普通函数一样轻松重用。

# 实现 Either Monad 设计模式

Monad 的概念是函数式编程设计模式之一。我们可以将 Monad 理解为对数据类型的封装，它为该数据类型添加特定的功能或为封装对象的不同的状态提供自定义处理程序。最常用的之一是 Maybe monad。Maybe monad 旨在提供有关封装属性存在的信息。它可以在属性可用时返回封装类型的实例，在不可用时返回空值。Java 8 引入了`Optional<T>`类，它实现了 Maybe 概念。这是一种避免在 null 值上操作的好方法。

然而，除了有关于不可用状态的信息之外，我们通常还希望能够提供一些额外的信息。例如，如果服务器返回一个空响应，那么获取一个错误代码或消息而不是`null`或空响应字符串将是有用的。这是一个适用于另一种类型 Monad 的场景，通常称为`Either`，我们将在本食谱中实现它。

# 如何操作...

1.  将`Either`声明为一个`sealed`类：

```kt
sealed class Either<out E, out V>
```

1.  添加两个`Either`的子类，分别表示错误和值：

```kt
sealed class Either<out L, out R> {
    data class Left<out L>(val left: L) : Either<L, Nothing>()
    data class Right<out R>(val right: R) : Either<Nothing, R>()
}
```

1.  添加用于方便实例化`Either`的工厂函数：

```kt
sealed class Either<out L, out R> {
    data class Left<out L>(val left: L) : Either<L, Nothing>()
    data class Right<out R>(val right: R) : Either<Nothing, R>()

    companion object {
        fun <R> right(value: R): Either<Nothing, R> = 
         Either.Right(value)
        fun <L> left(value: L): Either<L, Nothing> = 
         Either.Left(value)
    }
}
```

# 它是如何工作的...

为了使用`Either`类并从中受益于`Either.right()`和`Either.left()`方法，我们可以实现一个`getEither()`函数，该函数将尝试执行作为参数传递给它的某些操作。如果操作成功，它将返回包含操作结果的`Either.Right`实例，否则，它将返回`Either.Left`，包含一个抛出的异常实例：

```kt
fun <V> getEither(action: () -> V): Either<Exception, V> =
        try { Either.right(action()) } catch (e: Exception) { Either.left(e) }
```

按照惯例，我们使用`Either.Right`类型来提供默认值，而使用`Either.Left`来处理任何可能的边缘情况。

# 还有更多...

`Either` Monad 可以提供的一个基本函数式编程特性，是能够对其值应用函数。我们可以简单地通过`fold()`函数扩展`Either`类，该函数可以接受两个函数作为参数。第一个函数应用于`Either.Left`类型，第二个函数应用于`Either.Right`：

```kt
sealed class Either<out L, out R> {
    data class Left<out L>(val left: L) : Either<L, Nothing>()
    data class Right<out R>(val right: R) : Either<Nothing, R>()

    fun <T> fold(leftOp: (L) -> T, rightOp: (R) -> T): T = when (this) {
        is Left -> leftOp(this.left)
        is Right -> rightOp(this.right)
    }

  //...
}
```

`fold()`函数将从`leftOp`或`rightOp`函数返回一个值，取决于使用的是哪一个。我们可以通过一个服务器请求解析示例来说明`fold()`函数的用法。

假设我们声明了以下类型：

```kt
data class Response(val json: JsonObject)
data class ErrorResponse(val code: Int, val message: String)
```

我们还有一个负责提供后端响应的函数：

```kt
fun someGetRequest(): Either<ErrorResponse, Response> = //..
```

我们可以使用`fold()`函数以正确的方式处理返回值：

```kt
someGetRequest().fold({
    showErrorInfo(it.message)
}, {
    parseAndDisplayResults(it.json)
})
```

我们还可以通过其他有用的函数扩展`Either`类，这些函数类似于标准库中用于数据处理操作的可用的函数——`map`、`filter`和`exists`。

# 自动函数记忆化的方法

缓存是一种用于通过缓存昂贵函数调用的结果并在需要时重新使用它们的准备好的值来优化程序执行速度的技术。尽管缓存在内存使用和计算时间之间造成明显的权衡，但通常提供所需的性能至关重要。通常，我们将此模式应用于计算昂贵的函数。它可以帮助优化多次以相同参数值调用自身的递归函数。缓存可以轻松地添加到函数实现中。然而，在这个菜谱中，我们将创建一个通用、可重用的缓存机制，可以应用于任何函数。

# 如何实现...

1.  声明一个负责缓存结果的`Memoizer`类：

```kt
class Memoizer<P, R> private constructor() {

    private val map = ConcurrentHashMap<P, R>()

    private fun doMemoize(function: (P) -> R):
        (P) -> R = { param: P ->
        map.computeIfAbsent(param) { param: P ->
                    function(param)
                }
            }

    companion object {
        fun <T, U> memoize(function: (T) -> U): (T) -> U =
                Memoizer<T, U>().doMemoize(function)
    }
}
```

1.  为`(P) -> R`函数类型提供一个`memoized()`扩展函数：

```kt
fun <P, R> ((P) -> R).memoized(): (P) -> R = Memoizer.memoize<P, R>(this)
```

# 它是如何工作的...

`memoize()`函数接受一个单参数函数的实例作为其参数。`Memoizer`类包含`ConcurrentHashmap<P, R>`实例，用于缓存函数的返回值。该映射存储传递给`memoize()`作为参数的函数作为键，并将它们的返回值作为其值。首先，`memoize()`函数查找传递给函数作为参数的特定参数的值。如果该值存在于映射中，则返回。否则，执行函数，并将结果既通过`memoize()`返回，又放入映射中。这是通过标准库提供的方便的`inline fun <K, V> ConcurrentMap<K, V>.computeIfAbsent(key: K, defaultValue: () -> V): V`扩展函数实现的。

此外，我们还提供了一个`Function1`类型的扩展函数`memoized()`，允许我们直接将`memoize()`函数应用于函数引用。

Kotlin 中的底层函数被编译成 Java 字节码中的`FunctionN`接口实例，其中`N`对应函数参数的数量。正因为这个事实，我们能够为函数声明一个扩展函数。例如，为了为接受两个参数的函数`（P, Q）-> R`添加一个扩展函数，我们需要定义一个扩展函数为`fun <P, Q, R> Function2<P, Q, R>.myExtension(): MyReturnType`。

现在，让我们看看我们如何从`memoized()`函数的实际应用中受益。让我们考虑一个递归计算整数的阶乘的函数：

```kt
fun factorial(n: Int): Long = if (n == 1) n.toLong() else n * factorial(n - 1)
```

我们可以将`memoized()`扩展函数应用于启用结果缓存：

```kt
val cachedFactorial = ::factorial.memoized()
println(" Execution time: " + measureNanoTime { cachedFactorial(12) } + " ns")
println(" Execution time: " + measureNanoTime { cachedFactorial(13) } + " ns")
```

以下代码在标准计算机上给出以下输出：

```kt
Execution time: 1547274 ns
Execution time: 24690 ns
```

如您所见，尽管第二次计算需要更多的`factorial()`函数递归调用次数，但它比第一次计算花费的时间少得多。

# 还有更多...

我们可以为接受超过一个参数的其他函数实现类似的自动记忆化实现。为了声明一个接受*N*个参数的扩展函数，我们需要为`FunctionN`类型实现一个扩展函数。
