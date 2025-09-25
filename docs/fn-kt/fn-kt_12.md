# 第十二章：开始使用 Arrow

**Arrow** ([`arrow-kt.io/`](http://arrow-kt.io/)) 是一个 Kotlin 库，它提供了函数式构造、数据类型和其他抽象。Kotlin 语法强大且灵活，Arrow 利用这一点提供了一些标准中不包含的功能。

Arrow 是将两个最成功和最受欢迎的函数式库 `funKTionale` 和 `Kategory` 结合成一个的结果。在 2017 年晚些时候，两个开发者团队担心分裂会损害整个 Kotlin 社区，因此决定联合起来创建一个单一、统一的函数式库。

在本章中，我们将介绍如何使用现有函数构建新的和更丰富的函数。我们将涵盖的一些主题包括以下内容：

+   函数组合

+   部分应用

+   柯里化

+   缓存

+   管道

+   光学

# 函数组合

函数式编程作为一个概念的一个重要部分是，以我们使用任何其他类型的方式使用函数——作为值、参数、返回值等。我们可以用其他类型做的一件事是将它们作为构建其他类型的构建块；同样的概念也可以应用于函数。

函数组合是一种使用现有函数构建函数的技术；类似于 Unix 管道或通道管道，函数的结果值被用作下一个函数的参数。

在 Arrow 中，函数组合以一系列的 `infix` 扩展函数的形式出现：

| **函数** | **描述** |
| --- | --- |
| `compose` | 将右手函数调用的结果作为左手函数的参数。 |
| `forwardCompose` | 将左手函数调用的结果作为右手函数的参数。 |
| `andThen` | 是 `forwardCompose` 的别名。 |

让我们组合一些函数：

```java
import arrow.syntax.function.andThen
import arrow.syntax.function.compose
import arrow.syntax.function.forwardCompose
import java.util.*

val p: (String) -> String = { body -> "<p>$body</p>" }

val span: (String) -> String = { body -> "<span>$body</span>" }

val div: (String) -> String = { body -> "<div>$body</div>" }

val randomNames: () -> String = {
    if (Random().nextInt() % 2 == 0) {
        "foo"
    } else {
        "bar"
    }
}

fun main(args: Array<String>) {
    val divStrong: (String) -> String = div compose strong

    val spanP: (String) -> String = p forwardCompose span

    val randomStrong: () -> String = randomNames andThen strong

    println(divStrong("Hello composition world!"))
    println(spanP("Hello composition world!"))
    println(randomStrong())
}
```

要构建 `divStrong: (String) -> String` 函数，我们组合 `div:(String) -> String` 和 `strong:(String) -> String`。换句话说，`divStrong` 等同于以下代码片段：

```java
val divStrong: (String) -> String = { body -> "<div><strong>$body</div></strong>"}
```

对于 `spanP:(String) -> String`，我们按照以下方式组合 `span:(String) -> (String)` 和 `p:(String) -> String`：

```java
val spanP: (String) -> String = { body -> "<span><p>$body</p></span>"}
```

注意我们使用的是相同的类型 `(String) -> String`，但任何具有其他函数所需正确返回类型的函数都可以进行组合。

让我们用函数组合重写我们的 `Channel` 管道示例：

```java
data class Quote(val value: Double, val client: String, val item: String, val quantity: Int)

data class Bill(val value: Double, val client: String)

data class PickingOrder(val item: String, val quantity: Int)

fun calculatePrice(quote: Quote) = Bill(quote.value * quote.quantity, quote.client) to PickingOrder(quote.item, quote.quantity)

fun filterBills(billAndOrder: Pair<Bill, PickingOrder>): Pair<Bill, PickingOrder>? {
   val (bill, _) = billAndOrder
   return if (bill.value >= 100) {
      billAndOrder
   } else {
      null
   }
}

fun warehouse(order: PickingOrder) {
   println("Processing order = $order")
}

fun accounting(bill: Bill) {
   println("processing = $bill")
}

fun splitter(billAndOrder: Pair<Bill, PickingOrder>?) {
   if (billAndOrder != null) {
      warehouse(billAndOrder.second)
      accounting(billAndOrder.first)
   }
}

fun main(args: Array<String>) {
   val salesSystem:(Quote) -> Unit = ::calculatePrice andThen ::filterBills forwardCompose ::splitter
   salesSystem(Quote(20.0, "Foo", "Shoes", 1))
   salesSystem(Quote(20.0, "Bar", "Shoes", 200))
   salesSystem(Quote(2000.0, "Foo", "Motorbike", 1))
}

```

`salesSystem: (Quote) -> Unit` 函数的行为非常复杂，但它是由其他函数作为构建块构建的。

# 部分应用

使用函数组合，我们通过取两个函数来创建第三个函数；使用部分应用，我们通过向现有函数传递参数来创建一个新的函数。

Arrow 提供了两种部分应用风格——显式和隐式。

显式风格使用一系列称为 `partially1`、`partially2`，一直到 `partially22` 的扩展函数。隐式风格通过一系列扩展，重载了 `invoke` 操作符：

```java
package com.packtpub.functionalkotlin.chapter11

import arrow.syntax.function.invoke
import arrow.syntax.function.partially3

fun main(args: Array<String>) {
   val strong: (String, String, String) -> String = { body, id, style -> "<strong id=\"$id\" style=\"$style\">$body</strong>" }

   val redStrong: (String, String) -> String = strong.partially3("font: red") //Explicit

   val blueStrong: (String, String) -> String = strong(p3 = "font: blue") //Implicit

   println(redStrong("Red Sonja", "movie1"))
   println(blueStrong("Deep Blue Sea", "movie2"))
}
```

两种风格都可以按以下方式链式使用：

```java
fun partialSplitter(billAndOrder: Pair<Bill, PickingOrder>?, warehouse: (PickingOrder) -> Unit, accounting: (Bill) -> Unit) {
   if (billAndOrder != null) {
      warehouse(billAndOrder.second)
      accounting(billAndOrder.first)
   }
}

fun main(args: Array<String>) {
   val splitter: (billAndOrder: Pair<Bill, PickingOrder>?) -> Unit = ::partialSplitter.partially2 { order -> println("TESTING $order") }(p2 = ::accounting)

   val salesSystem: (quote: Quote) -> Unit = ::calculatePrice andThen ::filterBills forwardCompose splitter
   salesSystem(Quote(20.0, "Foo", "Shoes", 1))
   salesSystem(Quote(20.0, "Bar", "Shoes", 200))
   salesSystem(Quote(2000.0, "Foo", "Motorbike", 1))
}
```

我们原始的 `splitter` 函数不够灵活，因为它直接调用了仓库和会计函数。`partialSplitter` 函数通过将 `warehouse` 和 `accounting` 作为参数来解决这个问题；然而，一个 `(Pair<Bill, PickingOrder>?, (PickingOrder) -> Unit, (Bill) -> Unit)` 函数不能用于组合。然后，我们部分应用了两个函数——一个 lambda 和一个引用。

# 绑定

部分应用的一个特殊情况是 **绑定**。使用绑定时，你向 `(T) -> R` 函数传递一个 `T` 参数，但不执行它，实际上返回一个 `() -> R` 函数：

```java
fun main(args: Array<String>) {

   val footer:(String) -> String = {content -> "<footer&gt;$content</footer>"}
   val fixFooter: () -> String = footer.bind("Functional Kotlin - 2018") //alias for partially1
   println(fixFooter())
}
```

`bind` 函数只是 `partially1` 的别名，但给它一个单独的名字并使其语义更加正确是有意义的。

# 反转

**反转**接受任何函数并返回其参数顺序相反的函数（在其他语言中，此函数被称为 **flip**）。让我们看看以下代码：

```java
import arrow.syntax.function.partially3
import arrow.syntax.function.reverse

fun main(args: Array<String>) {
   val strong: (String, String, String) -> String = { body, id, style -> "<strong id=\"$id\" style=\"$style\">$body</strong>" }

   val redStrong: (String, String) -> String = strong.partially3("font: red") //Explicit

   println(redStrong("Red Sonja", "movie1"))

   println(redStrong.reverse()("movie2", "The Hunt for Red October"))

}
```

我们的 `redStrong` 函数使用起来有些笨拙，因为我们期望首先有 `id` 然后有 `body`，但可以通过 `reverse` 扩展函数轻松修复。`reverse` 函数可以应用于从参数 `1` 到 `22` 的函数。

# 管道

`pipe` 函数接受一个 `T` 值并使用它调用 `(T) -> R` 函数：

```java
import arrow.syntax.function.pipe

fun main(args: Array<String>) {
    val strong: (String) -> String = { body -> "<strong>$body</strong>" }

   "From a pipe".pipe(strong).pipe(::println)
}
```

**管道**类似于函数组合，但不同之处在于我们不是生成新函数，而是可以链式调用函数以产生新的值，从而减少嵌套调用。管道在其他语言中，如 **Elm** 和 **Ocaml** 中，被称为操作符 `|>`：

```java
fun main(args: Array<String>) {
   splitter(filterBills(calculatePrice(Quote(20.0, "Foo", "Shoes", 1)))) //Nested

   Quote(20.0, "Foo", "Shoes", 1) pipe ::calculatePrice pipe ::filterBills pipe ::splitter //Pipe
}
```

两行是等价的，但第一行必须从后往前理解，第二行应该从左到右阅读：

```java
import arrow.syntax.function.pipe
import arrow.syntax.function.pipe3
import arrow.syntax.function.reverse

fun main(args: Array<String>) {
   val strong: (String, String, String) -> String = { body, id, style -> "<strong id=\"$id\" style=\"$style\">$body</strong>" }

   val redStrong: (String, String) -> String = "color: red" pipe3 strong.reverse()

   redStrong("movie3", "Three colors: Red") pipe ::println
}
```

当 `pipe` 应用于多参数函数时，使用其变体 `pipe2` 到 `pipe22`，它表现得像 `partially1`。

# 柯里化

将柯里化应用于具有 *n* 个参数的函数，例如 `(A, B) -> R`，将其转换为 `n` 个函数调用的链，`(A) -> (B) -> R`：

```java
import arrow.syntax.function.curried
import arrow.syntax.function.pipe
import arrow.syntax.function.reverse
import arrow.syntax.function.uncurried

fun main(args: Array<String>) {

   val strong: (String, String, String) -> String = { body, id, style -> "<strong id=\"$id\" style=\"$style\">$body</strong>" }

   val curriedStrong: (style: String) -> (id: String) -> (body: String) -> String = strong.reverse().curried()

   val greenStrong: (id: String) -> (body: String) -> String = curriedStrong("color:green")

   val uncurriedGreenStrong: (id: String, body: String) -> String = greenStrong.uncurried()

   println(greenStrong("movie5")("Green Inferno"))

   println(uncurriedGreenStrong("movie6", "Green Hornet"))

   "Fried Green Tomatoes" pipe ("movie7" pipe greenStrong) pipe ::println
}
```

柯里化形式的函数可以通过 `uncurried()` 转换为正常的多参数形式。

# 柯里化和部分应用之间的区别

在柯里化和部分应用之间存在一些混淆。一些作者将它们视为同义词，但它们是不同的：

```java
import arrow.syntax.function.curried
import arrow.syntax.function.invoke

fun main(args: Array<String>) {
   val strong: (String, String, String) -> String = { body, id, style -> "<strong id=\"$id\" style=\"$style\">$body</strong>" }

   println(strong.curried()("Batman Begins")("trilogy1")("color:black")) // Curried

   println(strong("The Dark Knight")("trilogy2")("color:black")) // Fake curried, just partial application

   println(strong(p2 = "trilogy3")(p2 = "color:black")("The Dark Knight rises")) // partial application   
}
```

这些区别是显著的，并且可以帮助我们决定何时使用其中一个：

|  | **柯里化** | **部分应用** |
| --- | --- | --- |
| **返回值** | 当一个 *N* 元函数被柯里化时，它返回一个大小为 *N* 的函数链，(柯里化形式)。 | 当一个函数或 *N* 元函数被部分应用时，它返回一个 *N - 1* 元函数。 |
| **参数应用** | 柯里化后，只有链的第一个参数可以被应用。 | 任何参数都可以以任何顺序应用。 |
| **还原** | 可以将柯里化形式的函数还原为多参数函数。 | 由于部分应用不会改变函数形式，因此还原不适用。 |

部分应用可以更加灵活，但一些函数式风格倾向于偏好柯里化。重要的是要掌握的是，这两种风格都是不同的，并且都由 Arrow 支持。

# 逻辑补码

**逻辑补码**接受任何谓词（一个返回 `Boolean` 类型的函数）并将其否定。让我们看看以下代码：

```java
import arrow.core.Predicate
import arrow.syntax.function.complement

fun main(args: Array<String>) {
   val evenPredicate: Predicate<Int> = { i: Int -> i % 2 == 0 }
   val oddPredicate: (Int) -> Boolean = evenPredicate.complement()

   val numbers: IntRange = 1..10
   val evenNumbers: List<Int> = numbers.filter(evenPredicate)
   val oddNumbers: List<Int> = numbers.filter(oddPredicate)

   println(evenNumbers)
   println(oddNumbers)
}
```

注意，我们使用的是 `Predicate<T>` 类型，但它只是 `(T) -> Boolean` 的别名。从 `0` 到 `22` 参数的谓词都有补充扩展函数。

# 缓存

**缓存**是一种缓存纯函数结果的技巧。缓存函数的行为像一个普通函数，但它存储了与产生该结果提供的参数相关联的先前计算的结果。

缓存化的经典例子是斐波那契数列：

```java
import arrow.syntax.function.memoize
import kotlin.system.measureNanoTime

fun recursiveFib(n: Long): Long = if (n < 2) {
   n
} else {
   recursiveFib(n - 1) + recursiveFib(n - 2)
}

fun imperativeFib(n: Long): Long {
   return when (n) {
      0L -> 0
      1L -> 1
      else -> {
         var a = 0L
         var b = 1L
         var c = 0L
         for (i in 2..n) {
            c = a + b
            a = b
            b = c
         }
         c
      }
   }
}

fun main(args: Array<String>) {

   var lambdaFib: (Long) -> Long = { it } //Declared ahead to be used inside recursively

   lambdaFib = { n: Long ->
      if (n < 2) n else lambdaFib(n - 1) + lambdaFib(n - 2)
   }

   var memoizedFib: (Long) -> Long = { it }

   memoizedFib = { n: Long ->
      if (n < 2) n else memoizedFib(n - 1) + memoizedFib(n - 2)
   }.memoize()

   println(milliseconds("imperative fib") { imperativeFib(40) }) //0.006

   println(milliseconds("recursive fib") { recursiveFib(40) }) //1143.167
   println(milliseconds("lambda fib") { lambdaFib(40) }) //4324.890
   println(milliseconds("memoized fib") { memoizedFib(40) }) //1.588
}

inline fun milliseconds(description: String, body: () -> Unit): String {
   return "$description:${measureNanoTime(body) / 1_000_000.00} ms"
}
```

我们缓存版本的执行速度比递归函数版本快 700 多倍（后者几乎比 lambda 版本快四倍）。命令式版本是无敌的，因为它被编译器高度优化：

```java
fun main(args: Array<String>) = runBlocking {

   var lambdaFib: (Long) -> Long = { it } //Declared ahead to be used inside recursively

   lambdaFib = { n: Long ->
      if (n < 2) n else lambdaFib(n - 1) + lambdaFib(n - 2)
   }

   var memoizedFib: (Long) -> Long = { it }

   memoizedFib = { n: Long ->
      println("from memoized fib n = $n")
      if (n < 2) n else memoizedFib(n - 1) + memoizedFib(n - 2)
   }.memoize()
   val job = launch {
      repeat(10) { i ->
         launch(coroutineContext) { println(milliseconds("On coroutine $i - imperative fib") { imperativeFib(40) }) }
         launch(coroutineContext) { println(milliseconds("On coroutine $i - recursive fib") { recursiveFib(40) }) }
         launch(coroutineContext) { println(milliseconds("On coroutine $i - lambda fib") { lambdaFib(40) }) }
         launch(coroutineContext) { println(milliseconds("On coroutine $i - memoized fib") { memoizedFib(40) }) }
      }
   }

   job.join()

}
```

缓存函数内部使用线程安全的结构来存储其结果，因此可以在协程或任何其他并发代码上安全使用。

使用缓存函数存在潜在缺点。第一个缺点是读取内部缓存的进程比实际计算或内存消耗要高，因为目前，缓存函数不暴露任何行为来控制其内部存储。

# 部分函数

**部分函数**（不要与部分应用函数混淆）是一个对于其参数类型的每个可能值不一定有定义的函数。相比之下，**总函数**是对于其参数类型的每个可能值都有定义的函数。

让我们看看以下示例：

```java
fun main(args: Array<String>) {
   val upper: (String?) -> String = { s:String? -> s!!.toUpperCase()} //Partial function, it can't transform null

   listOf("one", "two", null, "four").map(upper).forEach(::println) //NPE
}
```

`upper` 函数是一个部分函数；尽管 `null` 是一个有效的 `String?` 值，但它不能处理 `null` 值。如果你尝试运行此代码，它将抛出 `NullPointerException`（**NPE**）。

Arrow 为类型 `(T) -> R` 的部分函数提供了一个显式类型 `PartialFunction<T, R>`：

```java
import arrow.core.PartialFunction

fun main(args: Array<String>) {
 val upper: (String?) -> String = { s: String? -> s!!.toUpperCase() } //Partial function, it can't transform null

 val partialUpper: PartialFunction<String?, String> = PartialFunction(definetAt = { s -> s != null }, f = upper)

 listOf("one", "two", null, "four").map(partialUpper).forEach(::println) //IAE: Value: (null) isn't supported by this function 
}
```

`PartialFunction<T, R>` 接收一个谓词 `(T) -> Boolean` 作为第一个参数，该参数必须返回 `true`，如果函数对该特定值有定义。`PartialFunction<T, R>` 函数扩展自 `(T) -> R`，因此它可以作为一个普通函数使用。

在这个例子中，代码仍然抛出异常，但现在类型为 `IllegalArgumentException`（**IAE**），并带有有用的消息。

为了避免抛出异常，我们必须将我们的部分函数转换为总函数：

```java
fun main(args: Array<String>) {

   val upper: (String?) -> String = { s: String? -> s!!.toUpperCase() } //Partial function, it can't transform null

   val partialUpper: PartialFunction<String?, String> = PartialFunction(definetAt = { s -> s != null }, f = upper)

   listOf("one", "two", null, "four").map{ s -> partialUpper.invokeOrElse(s, "NULL")}.forEach(::println)
}
```

一种选择是使用 `invokeOrElse` 函数，在值 `s` 对此函数未定义时返回默认值：

```java
fun main(args: Array<String>) {

   val upper: (String?) -> String = { s: String? -> s!!.toUpperCase() } //Partial function, it can't transform null

   val partialUpper: PartialFunction<String?, String> = PartialFunction(definetAt = { s -> s != null }, f = upper)

   val upperForNull: PartialFunction<String?, String> = PartialFunction({ s -> s == null }) { "NULL" }

   val totalUpper: PartialFunction<String?, String> = partialUpper orElse upperForNull

   listOf("one", "two", null, "four").map(totalUpper).forEach(::println)
}
```

第二种选择是使用 `orElse` 函数创建一个总函数，该函数由几个部分函数组成：

```java
fun main(args: Array<String>) {
   val fizz = PartialFunction({ n: Int -> n % 3 == 0 }) { "FIZZ" }
   val buzz = PartialFunction({ n: Int -> n % 5 == 0 }) { "BUZZ" }
   val fizzBuzz = PartialFunction({ n: Int -> fizz.isDefinedAt(n) && buzz.isDefinedAt(n) }) { "FIZZBUZZ" }
   val pass = PartialFunction({ true }) { n: Int -> n.toString() }

   (1..50).map(fizzBuzz orElse buzz orElse fizz orElse pass).forEach(::println)
}
```

通过`isDefinedAt(T)`函数，我们可以重用内部谓词，在这种情况下，用于构建`fizzBuzz`的条件。当在`orElse`链中使用时，声明顺序具有优先权，首先为某个值定义的部分函数将被执行，而链中的其他函数将被忽略。

# 身份和常量

身份和常量是简单的函数。`identity`函数返回提供的参数值；类似于加法和乘法恒等性质，将 0 加到任何数上仍然是同一个数。

`constant<T, R>(t: T)`函数返回一个新函数，该函数将始终返回`t`值：

```java
fun main(args: Array<String>) {

   val oneToFour = 1..4

   println("With identity: ${oneToFour.map(::identity).joinToString()}") //1, 2, 3, 4

   println("With constant: ${oneToFour.map(constant(1)).joinToString()}") //1, 1, 1, 1

}
```

我们可以使用`constant`重新编写我们的`fizzBuzz`值：

```java
fun main(args: Array<String>) {
   val fizz = PartialFunction({ n: Int -> n % 3 == 0 }, constant("FIZZ"))
   val buzz = PartialFunction({ n: Int -> n % 5 == 0 }, constant("BUZZ"))
   val fizzBuzz = PartialFunction({ n: Int -> fizz.isDefinedAt(n) && buzz.isDefinedAt(n) }, constant("FIZZBUZZ"))
   val pass = PartialFunction<Int, String>(constant(true)) { n -> n.toString() }

   (1..50).map(fizzBuzz orElse buzz orElse fizz orElse pass).forEach(::println)
}
```

身份和常量函数在函数式编程或数学算法的实现中非常有用，例如，常量是 SKI 组合演算中的 K。

# 透镜

**透镜**是优雅地更新不可变数据结构的抽象。透镜的一种形式是`Lens`（或透镜，具体取决于库实现）。`Lens`是一个功能引用，可以聚焦（因此得名）到结构中，并读取、写入或修改其目标：

```java
typealias GB = Int

data class Memory(val size: GB)
data class MotherBoard(val brand: String, val memory: Memory)
data class Laptop(val price: Double, val motherBoard: MotherBoard)

fun main(args: Array<String>) {
   val laptopX8 = Laptop(500.0, MotherBoard("X", Memory(8)))

   val laptopX16 = laptopX8.copy(
         price = 780.0,
         motherBoard = laptopX8.motherBoard.copy(
               memory = laptopX8.motherBoard.memory.copy(
                     size = laptopX8.motherBoard.memory.size * 2
               )
         )
   )

   println("laptopX16 = $laptopX16")
}
```

要从现有值创建一个新的`Laptop`值，我们需要使用多个嵌套的复制方法和引用。在这个例子中，这并不那么糟糕，但您可以想象在一个更复杂的数据结构中，事情可能会变得混乱。

让我们编写我们非常第一个`Lens`值：

```java
val laptopPrice: Lens<Laptop, Double> = Lens(
      get = { laptop -> laptop.price },
      set = { price -> { laptop -> laptop.copy(price = price) } }
)
```

`laptopPrice`值是一个`Lens<Laptop, Double>`，我们使用函数`Lens<S, T, A, B>`（实际上是`Lens.invoke`）初始化它。`Lens`接受两个函数作为参数，作为`get: (S) -> A`和`set: (B) -> (S) -> T`。

如您所见，`set`是一个柯里化函数，因此您可以像这样编写您的设置：

```java
import arrow.optics.Lens

val laptopPrice: Lens<Laptop, Double> = Lens(
      get = { laptop -> laptop.price },
      set = { price: Double, laptop: Laptop -> laptop.copy(price = price) }.curried()
)
```

根据您的偏好，这可以使阅读和编写变得更加容易。

现在您已经拥有了第一个透镜，它可以用来设置、读取和修改笔记本电脑的价格。这并不太令人印象深刻，但透镜的魔力在于它们的组合：

```java
import arrow.optics.modify

val laptopMotherBoard: Lens<Laptop, MotherBoard> = Lens(
      get = { laptop -> laptop.motherBoard },
      set = { mb -> { laptop -> laptop.copy(motherBoard = mb) } }
)

val motherBoardMemory: Lens<MotherBoard, Memory> = Lens(
      get = { mb -> mb.memory },
      set = { memory -> { mb -> mb.copy(memory = memory) } }
)

val memorySize: Lens<Memory, GB> = Lens(
      get = { memory -> memory.size },
      set = { size -> { memory -> memory.copy(size = size) } }
)

fun main(args: Array<String>) {
   val laptopX8 = Laptop(500.0, MotherBoard("X", Memory(8)))

   val laptopMemorySize: Lens<Laptop, GB> = laptopMotherBoard compose motherBoardMemory compose memorySize

   val laptopX16 = laptopMemorySize.modify(laptopPrice.set(laptopX8, 780.0)) { size ->
      size * 2
   }

   println("laptopX16 = $laptopX16")
}
```

我们通过将`Laptop`到`memorySize`的所有透镜组合创建了`laptopMemorySize`；然后，我们可以设置笔记本电脑的价格并修改其内存。

尽管透镜很酷，但看起来有很多样板代码。不用担心，Arrow 可以为您生成这些透镜。

# 配置 Arrows 代码生成

在一个 Gradle 项目中，添加一个名为`generated-kotlin-sources.gradle`的文件：

```java
apply plugin: 'idea'

idea {
    module {
        sourceDirs += files(
            'build/generated/source/kapt/main',
            'build/generated/source/kaptKotlin/main',
            'build/tmp/kapt/main/kotlinGenerated')
        generatedSourceDirs += files(
            'build/generated/source/kapt/main',
            'build/generated/source/kaptKotlin/main',
            'build/tmp/kapt/main/kotlinGenerated')
    }
}
```

然后，在`build.gradle`文件中，添加以下内容：

```java
apply plugin: 'kotlin-kapt'
apply from: rootProject.file('gradle/generated-kotlin-sources.gradle')
```

在`build.gradle`文件中添加一个新的依赖项：

```java
dependencies {
    ...
    kapt    'io.arrow-kt:arrow-annotations-processor:0.5.2'
    ...     
}
```

一旦配置完成，您就可以使用常规的构建命令生成 Arrow 代码，`./gradlew build`。

# 生成透镜

一旦配置了 Arrow 的代码生成，您可以将`@lenses`注解添加到您希望生成透镜的数据类中：

```java
import arrow.lenses
import arrow.optics.Lens
import arrow.optics.modify

typealias GB = Int

@lenses data class Memory(val size: GB)
@lenses data class MotherBoard(val brand: String, val memory: Memory)
@lenses data class Laptop(val price: Double, val motherBoard: MotherBoard)

fun main(args: Array<String>) {
   val laptopX8 = Laptop(500.0, MotherBoard("X", Memory(8)))

   val laptopMemorySize: Lens<Laptop, GB> = laptopMotherBoard() compose motherBoardMemory() compose memorySize()

   val laptopX16 = laptopMemorySize.modify(laptopPrice().set(laptopX8, 780.0)) { size ->
      size * 2
   }

   println("laptopX16 = $laptopX16")
}
```

Arrow 为我们的数据类生成与构造函数参数数量一样多的透镜，名称约定为`classProperty`，并在同一包中，因此不需要额外的导入。

# 摘要

在本章中，我们介绍了 Arrow 的许多特性，这些特性为我们提供了创建、生成和丰富现有函数的工具。我们使用现有的函数组合成新的函数；我们包括了部分应用和柯里化。我们还通过记忆化缓存了纯函数的结果，并使用透镜修改了数据结构。

Arrow 的特性为使用基本函数式原则创建丰富且可维护的应用程序打开了可能性。

在下一章中，我们将介绍更多的 Arrow 特性，包括`Option`、`Either`和`Try`等数据类型。
