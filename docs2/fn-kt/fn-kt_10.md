# 第十章：函子、应用和函子

函子、应用和函子是关于函数式编程中最常搜索的词汇之一，如果你考虑到没有人知道它们的意思（实际上，有一些聪明的人知道他们在说什么），这就有意义了。特别是关于函子的混淆已经成为编程社区中的一个笑话/模因：

“一个函子是端内函子的范畴中的幺半群，有什么问题？”

这句话是虚构地归因于詹姆斯·艾里（James Iry）在他的经典博客文章《编程语言的简短、不完整且大部分错误的历史》中引用的菲利普·瓦德勒（Philip Wadler），([`james-iry.blogspot.co.uk/2009/05/brief-incomplete-and-mostly-wrong.html`](http://james-iry.blogspot.co.uk/2009/05/brief-incomplete-and-mostly-wrong.html))。

在本章中，我们将涵盖以下主题：

+   函子

+   选项、列表和作为函子的函数

+   摩纳哥（Monads）

+   应用（Applicatives）

# 函子（Functors）

如果我告诉你，你已经在 Kotlin 中使用了函子，你会感到惊讶吗？让我们看看以下代码：

```java
fun main(args: Array<String>) {
    listOf(1, 2, 3)
            .map { i -> i * 2 }
            .map(Int::toString)
            .forEach(::println)
}
```

`List<T>` 类有一个函数，`map(transform: (T) -> R): List<R>`。`map` 这个名字是从哪里来的？它来自范畴论。当我们从 `Int` 转换到 `String` 时，我们是从 `Int` 范畴映射到 `String` 范畴。在同样的意义上，在我们的例子中，我们是从 `List<Int>` 转换到 `List<Int>`（不是很令人兴奋），然后从 `List<Int>` 转换到 `List<String>`。我们没有改变外部类型，只是内部值。

那就是一个函子。一个 **函子** 是一个定义了如何转换或映射其内容的类型。你可以找到关于函子的不同定义，或多或少是学术性的；但原则上，它们都指向同一个方向。

让我们为函子类型定义一个泛型接口：

```java
interface Functor<C<_>> { //Invalid Kotlin code
    fun <A,B> map(ca: C<A>, transform: (A) -> B): C<B>
}
```

此外，它无法编译，因为 Kotlin 不支持高阶类型。

你可以在第十三章 Chapter 13 中找到关于 Kotlin 高阶类型（包括替代方案和 Kotlin 的未来）的更多信息，*箭头类型***。

在支持高阶类型的语言中，例如 **Scala** 和 **Haskell**，可以定义一个 `Functor` 类型，例如 Scala cats 函子：

```java
trait Functor[F[_]] extends Invariant[F] { self =>
  def mapA, B(f: A => B): F[B]

  //More code here
```

在 Kotlin 中，我们没有这些功能，但我们可以通过约定来模拟它们。如果一个类型有一个函数或扩展函数，那么 `map` 就是一个函子（这被称为**结构化类型**，通过其结构而不是其层次来定义类型）。

我们可以有一个简单的 `Option` 类型：

```java
sealed class Option<out T> {
    object None : Option<Nothing>() {
        override fun toString() = "None"
    }

    data class Some<out T>(val value: T) : Option<T>()

    companion object
}
```

然后，你可以为它定义一个 `map` 函数：

```java
fun <T, R> Option<T>.map(transform: (T) -> R): Option<R> = when (this) {
    Option.None -> Option.None
    is Option.Some -> Option.Some(transform(value))
}
```

然后按照以下方式使用它：

```java
fun main(args: Array<String>) {
    println(Option.Some("Kotlin")
            .map(String::toUpperCase)) //Some(value=KOTLIN)
}
```

现在，一个 `Option` 值对于 `Some` 和 `None` 将有不同的行为：

```java
fun main(args: Array<String>) {
    println(Option.Some("Kotlin").map(String::toUpperCase)) //Some(value=KOTLIN)
    println(Option.None.map(String::toUpperCase)) //None
}
```

扩展函数非常灵活，以至于我们可以为函数类型编写一个 `map` 函数，即 `(A) -> B`，因此将函数转换为函子：

```java
fun <A, B, C> ((A) -> B).map(transform: (B) -> C): (A) -> C = { t -> transform(this(t)) }
```

我们在这里改变的是通过应用参数函数 `transform: (B) -> C` 到函数 `(A) -> B` 的结果，将返回类型从 `B` 改为 `C`：

```java
fun main(args: Array<String>) {
    val add3AndMultiplyBy2: (Int) -> Int = { i: Int -> i + 3 }.map { j -> j * 2 }
    println(add3AndMultiplyBy2(0)) //6
    println(add3AndMultiplyBy2(1)) //8
    println(add3AndMultiplyBy2(2)) //10
}
```

如果你在其他函数式编程语言中有所经验，你会认出这种行为是前向函数组合（更多关于函数组合的内容请参阅 第十二章，*使用箭头入门）。

# Monads

**monad** 是一个定义了 `flatMap`（或在其他语言中的 `bind`）函数的 funtor 类型，该函数接收一个返回相同类型的 lambda。让我用一个例子来解释它。幸运的是，`List<T>` 定义了一个 `flatMap` 函数：

```java
fun main(args: Array<String>) {
    val result = listOf(1, 2, 3)
            .flatMap { i ->
                listOf(i * 2, i + 3)
            }
            .joinToString()

    println(result) //2, 4, 4, 5, 6, 6
}
```

在 `map` 函数中，我们只是转换 `List` 值的内容，但在 `flatMap` 中，我们可以返回一个具有更少或更多项的新 `List` 类型，这使得它比 `map` 更强大得多。

因此，一个通用的 monad 将看起来像这样（记住我们没有高阶类型）：

```java
interface Monad<C<_>>: Functor<C> { //Invalid Kotlin code
    fun <A, B> flatMap(ca:C<A>, fm:(A) -> C<B>): C<B>
}
```

现在，我们可以为我们的 `Option` 类型编写一个 `flatMap` 函数：

```java
fun <T, R> Option<T>.flatMap(fm: (T) -> Option<R>): Option<R> = when (this) {
    Option.None -> Option.None
    is Option.Some -> fm(value)
}
```

如果你仔细观察，你会发现 `flatMap` 和 `map` 非常相似；相似到我们可以用 `flatMap` 重写 `map`：

```java
fun <T, R> Option<T>.map(transform: (T) -> R): Option<R> = flatMap { t -> Option.Some(transform(t)) }
```

现在我们可以用 `flatMap` 函数的强大功能以一些酷炫的方式使用它，这些方式在普通的 `map` 中是不可能的：

```java
fun calculateDiscount(price: Option<Double>): Option<Double> {
    return price.flatMap { p ->
        if (p > 50.0) {
            Option.Some(5.0)
        } else {
            Option.None
        }
    }
}

fun main(args: Array<String>) {
    println(calculateDiscount(Option.Some(80.0))) //Some(value=5.0)
    println(calculateDiscount(Option.Some(30.0))) //None
    println(calculateDiscount(Option.None)) //None
}
```

我们的功能函数 `calculateDiscount` 接收并返回 `Option<Double>`。如果价格高于 `50.0`，我们返回一个包裹在 `Some` 中的 `5.0` 折扣，如果没有，则返回 `None`。

`flatMap` 的一个酷炫技巧是它可以嵌套：

```java
fun main(args: Array<String>) {
    val maybeFive = Option.Some(5)
    val maybeTwo = Option.Some(2)

    println(maybeFive.flatMap { f ->
        maybeTwo.flatMap { t ->
            Option.Some(f + t)
        }
    }) // Some(value=7)
}
```

在内部的 `flatMap` 函数中，我们可以访问这两个值并对它们进行操作。

我们可以通过组合 `flatMap` 和 `map` 来稍微缩短这个示例的写法：

```java
fun main(args: Array<String>) {
    val maybeFive = Option.Some(5)
    val maybeTwo = Option.Some(2)

    println(maybeFive.flatMap { f ->
        maybeTwo.map { t ->
            f + t
        }
    }) // Some(value=7)
}
```

因此，我们可以将我们的第一个 `flatMap` 示例重写为两个列表的组合——一个包含数字，另一个包含函数：

```java
fun main(args: Array<String>) {
    val numbers = listOf(1, 2, 3)
    val functions = listOf<(Int) -> Int>({ i -> i * 2 }, { i -> i + 3 })
    val result = numbers.flatMap { number ->
        functions.map { f -> f(number) }
    }.joinToString()

    println(result) //2, 4, 4, 5, 6, 6
}
```

这种嵌套多个 `flatMap` 或 `flatMap` 与 `map` 组合的技术非常强大，并且是另一个名为 monadic comprehensions（第十三章，*箭头类型*）的概念背后的主要思想，它允许我们组合 monadic 操作（更多关于 comprehensions 的内容请参阅）。

# Applicatives

我们之前的例子，在同一个类型的包装器内部调用 lambda，并在同一个包装器内部传递参数，是介绍 applicatives 的完美方式。

**applicative** 是一个定义了两个函数的类型，一个 `pure(t: T)` 函数，它返回包裹在 applicative 类型中的 `T` 值，以及一个 `ap` 函数（在其他语言中称为 `apply`），它接收一个包裹在 applicative 类型中的 lambda。

在上一节中，当我们解释 monads 时，我们让它们直接从 funtor 扩展，但事实上，monad 是从 applicative 扩展的，而 applicative 是从 funtor 扩展的。因此，我们的通用 applicative 的伪代码以及整个层次结构将看起来像这样：

```java
interface Functor<C<_>> { //Invalid Kotlin code
    fun <A,B> map(ca:C<A>, transform:(A) -> B): C<B>
}

interface Applicative<C<_>>: Functor<C> { //Invalid Kotlin code
    fun <A> pure(a:A): C<A>

    fun <A, B> ap(ca:C<A>, fab: C<(A) -> B>): C<B>
}

interface Monad<C<_>>: Applicative<C> { //Invalid Kotlin code
    fun <A, B> flatMap(ca:C<A>, fm:(A) -> C<B>): C<B>
}
```

简而言之，一个 applicative 是一个更强大的 funtor，而一个 monad 是一个更强大的 applicative。

现在，让我们为 `List<T>` 写一个 `ap` 扩展函数：

```java
fun <T, R> List<T>.ap(fab: List<(T) -> R>): List<R> = fab.flatMap { f -> this.map(f) }
```

我们可以回顾一下上一节中 *Monads* 部分的最后一个示例：

```java
fun main(args: Array<String>) {
    val numbers = listOf(1, 2, 3)
    val functions = listOf<(Int) -> Int>({ i -> i * 2 }, { i -> i + 3 })
    val result = numbers.flatMap { number ->
        functions.map { f -> f(number) }
    }.joinToString()

    println(result) //2, 4, 4, 5, 6, 6
}
```

让我们用 `ap` 函数来重写它：

```java
fun main(args: Array<String>) {
    val numbers = listOf(1, 2, 3)
    val functions = listOf<(Int) -> Int>({ i -> i * 2 }, { i -> i + 3 })
    val result = numbers
            .ap(functions)
            .joinToString()
    println(result) //2, 4, 6, 4, 5, 6
}
```

更容易阅读，但有一个警告——结果顺序不同。我们需要意识到并选择适合我们特定情况的最佳选项。

我们可以向我们的`Option`类添加`pure`和`ap`：

```java
fun <T> Option.Companion.pure(t: T): Option<T> = Option.Some(t)

```

`Option.pure`只是`Option.Some`构造函数的一个简单别名。

我们的`Option.ap`函数非常迷人：

```java
//Option
fun <T, R> Option<T>.ap(fab: Option<(T) -> R>): Option<R> = fab.flatMap { f -> map(f) }

//List
fun <T, R> List<T>.ap(fab: List<(T) -> R>): List<R> = fab.flatMap { f -> this.map(f) }
```

`Option.ap`和`List.ap`有相同的主体，使用`flatMap`和`map`的组合，这正是我们组合单子操作的方式。

使用单子，我们使用`flatMap`和`map`对两个`Option<Int>`进行求和：

```java
fun main(args: Array<String>) {
    val maybeFive = Option.Some(5)
    val maybeTwo = Option.Some(2)

    println(maybeFive.flatMap { f ->
        maybeTwo.map { t ->
            f + t
        }
    }) // Some(value=7)
}
```

现在，使用应用函数：

```java
fun main(args: Array<String>) {
    val maybeFive = Option.pure(5)
    val maybeTwo = Option.pure(2)

    println(maybeTwo.ap(maybeFive.map { f -> { t: Int -> f + t } })) // Some(value=7)
}
```

这不是很容易阅读。首先，我们使用一个 lambda `(Int) -> (Int) -> Int`（技术上是一个柯里化函数，关于柯里化函数的更多信息可以在第十二章，*开始使用 Arrow*）来映射`maybeFive`，它返回一个`Option<(Int) -> Int>`，可以作为`maybeTwo.ap`的参数。

我们可以用一个小技巧（我从 Haskell 那里借来的）使事情更容易阅读：

```java
infix fun <T, R> Option<(T) -> R>.`(*)`(o: Option<T>): Option<R> = flatMap { f: (T) -> R -> o.map(f) }
```

`infix`扩展函数`Option<(T) -> R>.`(*)` ``将允许我们从左到右读取`sum`操作；这有多酷？现在，让我们看看以下代码，它是使用应用函数对两个`Option<Int>`进行求和

```java
fun main(args: Array<String>) {
    val maybeFive = Option.pure(5)
    val maybeTwo = Option.pure(2)

    println(Option.pure { f: Int -> { t: Int -> f + t } } `(*)` maybeFive `(*)` maybeTwo) // Some(value=7)
}
```

我们用`pure`函数包装`(Int) -> (Int) -> Int` lambda，然后逐个应用`Option<Int>`。我们使用名称`(*)`作为对 Haskell 的`<*>`的致敬。

到目前为止，你可以看到应用函数让你可以做一些很酷的技巧，但单子更强大、更灵活。何时使用一个或另一个？这显然取决于你的特定问题，但我们的总体建议是使用尽可能少权力的抽象。你可以从函子`map`开始，然后是应用函数`ap`，最后是单子`flatMap`。所有的事情都可以用`flatMap`来完成（正如你所见，`Option`、`map`和`ap`都是使用`flatMap`实现的），但大多数时候`map`和`ap`可以更容易地推理。

回到函数，我们可以使一个函数表现得像一个应用函数。首先，我们应该添加一个纯函数：

```java
object Function1 {
    fun <A, B> pure(b: B) = { _: A -> b }
}
```

首先，我们创建一个对象`Function1`，因为函数类型`(A) -> B`没有伴随对象来添加新的扩展函数，就像我们为`Option`所做的那样：

```java
fun main(args: Array<String>) {
    val f: (String) -> Int = Function1.pure(0)
    println(f("Hello,"))    //0
    println(f("World"))     //0
    println(f("!"))         //0
}
```

`Function1.pure(t: T)`将一个`T`值包装在一个函数中并返回它，无论我们使用什么参数。如果你有其他函数式语言的经验，你会认出函数的`pure`作为一个`identity`函数（关于`identity`函数的更多信息可以在第十二章，*开始使用 Arrow*）。

让我们在一个函数`(A) -> B`中添加`flatMap`和`ap`：

```java
fun <A, B, C> ((A) -> B).map(transform: (B) -> C): (A) -> C = { t -> transform(this(t)) }

fun <A, B, C> ((A) -> B).flatMap(fm: (B) -> (A) -> C): (A) -> C = { t -> fm(this(t))(t) }

fun <A, B, C> ((A) -> B).ap(fab: (A) -> (B) -> C): (A) -> C = fab.flatMap { f -> map(f) }
```

我们已经涵盖了`map(transform: (B) -> C): (A) -> C`，并且我们知道它表现得像一个正向函数组合。如果你仔细观察`flatMap`和`ap`，你会看到参数有点反方向（并且`ap`被实现为所有其他类型的`ap`函数）。

但是，我们能用函数的 `ap` 做些什么呢？让我们看看下面的代码：

```java
fun main(args: Array<String>) {
    val add3AndMultiplyBy2: (Int) -> Int = { i: Int -> i + 3 }.ap { { j: Int -> j * 2 } }
    println(add3AndMultiplyBy2(0)) //6
    println(add3AndMultiplyBy2(1)) //8
    println(add3AndMultiplyBy2(2)) //10
}
```

嗯，我们可以组合函数，这并不令人兴奋，因为我们已经用 `map` 做过这样的事情。但是函数的 `ap` 有一个小技巧。我们可以访问原始参数：

```java
fun main(args: Array<String>) {
    val add3AndMultiplyBy2: (Int) -> Pair<Int, Int> = { i:Int -> i + 3 }.ap { original -> { j:Int -> original to (j * 2) } }
    println(add3AndMultiplyBy2(0)) //(0, 6)
    println(add3AndMultiplyBy2(1)) //(1, 8)
    println(add3AndMultiplyBy2(2)) //(2, 10)
}
```

在函数组合中访问原始参数在多个场景下很有用，例如调试和审计。

# 摘要

我们已经介绍了很多名字吓人但背后理念简单的酷概念。函子、应用和单子类型为我们打开了几种抽象和更强大的函数概念的大门，这些内容我们将在接下来的章节中介绍。我们学习了 Kotlin 的一些局限性以及我们如何在创建函数来模拟不同类型的函子、应用和单子时克服它们。我们还探讨了函子、应用和单子之间的层次关系。

在下一章中，我们将介绍如何有效地处理数据流。
