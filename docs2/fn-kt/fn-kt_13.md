# 第十三章：箭头类型

Arrow 包含了许多传统函数类型的实现，如 `Option`、`Either` 和 `Try`，以及许多其他类型类，如 `functor` 和 `monad`。

在本章中，我们将涵盖以下主题：

+   使用 `Option` 来管理空值

+   `Either` 和 `Try` 用于管理错误

+   组合和转换器

+   `State` 用于管理应用程序状态

# 选项

`Option<T>` 数据类型是值 `T` 的存在或不存在的一种表示。在 Arrow 中，`Option<T>` 是一个密封类，有两个子类型，`Some<T>`，一个表示值 `T` 存在的数据类，以及 `None`，一个表示值不存在的一个对象。作为一个密封类，`Option<T>` 不能有其他子类型；因此编译器可以全面检查，如果 `Some<T>` 和 `None` 都被覆盖。

我知道（或者假装知道）你此刻的想法——为什么我需要 `Option<T>` 来表示 `T` 的存在或不存在，如果 Kotlin 中已经有 `T` 表示存在，`T?` 表示不存在呢？

你是对的。但是 `Option` 提供的比可空类型更多的价值，让我们直接跳到例子：

```java
fun divide(num: Int, den: Int): Int? {
    return if (num % den != 0) {
        null
    } else {
        num / den
    }
}

fun division(a: Int, b: Int, den: Int): Pair<Int, Int>? {
    val aDiv = divide(a, den)
    return when (aDiv) {
        is Int -> {
            val bDiv = divide(b, den)
            when (bDiv) {
                is Int -> aDiv to bDiv
                else -> null
            }
        }
        else -> null
    }
}
```

`division` 函数接受三个参数——两个整数（`a`，`b`）和一个除数（`den`），如果两个数都能被 `den` 整除，则返回一个 `Pair<Int, Int>`，否则返回 `null`。

我们可以用 `Option` 表达相同的算法：

```java
import arrow.core.*
import arrow.syntax.option.toOption

fun optionDivide(num: Int, den: Int): Option<Int> = divide(num, den).toOption()

fun optionDivision(a: Int, b: Int, den: Int): Option<Pair<Int, Int>> {
   val aDiv = optionDivide(a, den)
   return when (aDiv) {
      is Some -> {
         val bDiv = optionDivide(b, den)
         when (bDiv) {
            is Some -> Some(aDiv.t to bDiv.t)
            else -> None
         }
      }
      else -> None
   }
}
```

函数 `optionDivide` 接受除法操作的空值结果，并使用 `toOption()` 扩展函数将其作为 `Option` 返回。

与 `division` 相比，`optionDivision` 没有太大变化，它是以不同类型表达的同一种算法。如果我们在这里停止，那么 `Option<T>` 在可空类型之上并没有提供额外的价值。幸运的是，情况并非如此；有更多使用 `Option` 的方法：

```java
fun flatMapDivision(a: Int, b: Int, den: Int): Option<Pair<Int, Int>> {
   return optionDivide(a, den).flatMap { aDiv: Int ->
      optionDivide(b, den).flatMap { bDiv: Int ->
         Some(aDiv to bDiv)
      }
   }
}
```

`Option` 提供了几个函数来处理其内部值，在这个例子中，`flatMap`（作为一个 monad）以及现在我们的代码看起来要短得多。

看看以下简短列表中的一些 `Option<T>` 函数：

| **函数** | **描述** |
| --- | --- |
| `exists(p :Predicate<T>): Boolean` | 如果存在值 `T`，则返回谓词 `p` 的结果，否则返回 null。 |
| `filter(p: Predicate<T>): Option<T>` | 如果值 `T` 存在并且满足谓词 `p`，则返回 `Some<T>`，否则返回 `None`。 |
|  `flatMap(f: (T) -> Option<T>): Option<T>` | 一个 `flatMap` 转换函数（类似于 monad）。 |
| `<R> fold(ifEmpty: () -> R, some: (T) -> R): R<R>` | 返回值被转换为 `R`，对于 `None` 调用 `ifEmpty`，对于 `Some<T>` 调用 `some`。 |
| `getOrElse(default:() -> T): T` | 如果存在值 `T`，则返回值 `T`，否则返回 `default` 结果。 |
| `<R> map(f: (T) -> R):Option<T>` | 一个转换函数（类似于 `functor`）。 |
| `orNull(): T?` | 将值 `T` 返回为可空的 `T?`。 |

最后的除法实现将使用列表推导式：

```java
import arrow.typeclasses.binding

fun comprehensionDivision(a: Int, b: Int, den: Int): Option<Pair<Int, Int>> {
   return Option.monad().binding {
      val aDiv: Int = optionDivide(a, den).bind()
      val bDiv: Int = optionDivide(b, den).bind()
      aDiv to bDiv
   }.ev()
}
```

组合是一种计算技术，可以按顺序计算任何包含 `flatMap` 函数并能提供 monad 实例的类型（例如 `Option`、`List` 等）。

在 Arrow 中，使用协程进行组合。是的，协程在异步执行域之外也很有用。

如果我们将之前示例中的延续进行概述，它将看起来像这样（这是一个有助于理解协程的有用心理模型）

```java
fun comprehensionDivision(a: Int, b: Int, den: Int): Option<Pair<Int, Int>> {
   return Option.monad().binding {
      val aDiv: Int = optionDivide(a, den).bind()
      // start continuation 1
         val bDiv: Int = optionDivide(b, den).bind()
          //start continuation 2
            aDiv to bDiv
         //end continuation 2
 // end continuation 1
   }.ev()
}
```

`Option.monad().binding` 是一个协程构建器，而 `bind()` 函数是一个挂起函数。如果你正确地回忆起我们的协程章节，延续是任何挂起点之后（即挂起函数被调用时）的代码表示。在我们的示例中，我们有两个挂起点和两个延续，当我们返回（在最后一行块中）时，我们处于第二个延续中，我们可以访问两个值，`aDiv` 和 `bDiv`。

将此算法作为延续来阅读与我们的 `flatMapDivision` 函数非常相似。在幕后，`Option.monad().binding` 使用带有延续的 `Option.flatMap` 来创建组合；一旦编译，`comprehensionDivision` 和 `flatMapDivision` 在很大程度上是等价的。

`ev()` 方法将在下一节中解释。

# Arrow 的类型层次结构

Kotlin 的类型系统存在一个限制——它不支持**高阶类型**（**HKT**）。不深入类型理论的话，HKT 是一种声明其他泛型值作为类型参数的类型：

```java
class MyClass<T>() //Valid Kotlin code

class MyHigherKindedClass<K<T>>() //Not valid kotlin code
```

对于 Kotlin 的函数式编程来说，缺少 HKT 并不是一件好事，因为许多高级函数式构造和模式都使用了它们。

Arrow 团队正在致力于**Kotlin 进化和增强过程**（**KEEP**）——这是一个社区流程，用于添加新的语言特性，在 Kotlin 中称为类型类扩展（[`github.com/Kotlin/KEEP/pull/87`](https://github.com/Kotlin/KEEP/pull/87)）以支持 HKT 和其他特性。目前，尚不清楚这个 KEEP（代码为 *KEEP-87*）何时会被纳入 Kotlin，但到目前为止，这是最受评论的提案，并吸引了大量关注。由于它仍在进行中，细节尚不明确，但其中透露出一丝希望。

Arrow 解决这个问题的方法是通过对称为基于证据的 HKTs 的技术进行模拟。

让我们看看一个 `Option<T>` 的声明：

```java
package arrow.core

import arrow.higherkind
import java.util.*

/**
 * Represents optional values. Instances of `Option`
 * are either an instance of $some or the object $none.
 */
@higherkind
sealed class Option<out A> : OptionKind<A> {
  //more code goes here

```

`Option<A>` 使用了 `@higherkind` 注解，这与我们上一章中提到的 `@lenses` 类似；这个注解用于生成支持基于证据的 HKTs 的代码。`Option<A>` 从 `OptionKind<A>` 扩展而来：

```java
package arrow.core

class OptionHK private constructor()
typealias OptionKind<A> = arrow.HK<OptionHK, A>

@Suppress("UNCHECKED_CAST", "NOTHING_TO_INLINE")
inline fun <A> OptionKind<A>.ev(): Option<A> =
  this as Option<A>
```

`OptionKind<A>` 是 `HK<OptionHK, A>` 的类型别名，所有这些代码都是使用 `@higherkind` 注解处理器生成的。`OptionHK` 是一个不可实例化的类，用作 `HK` 和 `OptionKind` 的唯一标签名称，而 `OptionKind` 是 HKT 的一种中间表示形式。`Option.monad().binding` 返回 `OptionKind<T>`，这就是为什么我们需要在最后调用 `ev()` 来返回一个合适的 `Option<T>`：

```java
package arrow

interface HK<out F, out A>

typealias HK2<F, A, B> = HK<HK<F, A>, B>

typealias HK3<F, A, B, C> = HK<HK2<F, A, B>, C>

typealias HK4<F, A, B, C, D> = HK<HK3<F, A, B, C>, D>

typealias HK5<F, A, B, C, D, E> = HK<HK4<F, A, B, C, D>, E>

```

`HK` 接口（**higher-kinded** 的简称）用于表示一元到 `HK5` 的 HKT，其中 `HK5` 表示五元。在 `HK<F, A>` 上，`F` 表示类型，`A` 表示泛型参数，因此 `Option<Int>` 是 `OptionKind<Int>` 值，它是 `HK<OptionHK, Int>`。

让我们看看 `Functor<F>`：

```java
package arrow.typeclasses

import arrow.*

@typeclass
interface Functor<F> : TC {

    fun <A, B> map(fa: HK<F, A>, f: (A) -> B): HK<F, B>

}

```

`Functor<F>` 扩展 `TC`，一个标记接口，并且，正如你可以猜到的，它有一个 `map` 函数。`map` 函数接收 `HK<F, A>` 作为第一个参数，并接收一个 lambda `(A) -> B` 来将 `A` 的值转换为 `B`，并将其转换为 `HK<F, B>`。

让我们创建我们的基本数据类型 `Mappable`，它可以提供 `Functor` 类型类的实例：

```java
import arrow.higherkind

@higherkind
class Mappable<T>(val t: T) : MappableKind<T> {
   fun <R> map(f: (T) -> R): Mappable<R> = Mappable(f(t))

   override fun toString(): String = "Mappable(t=$t)"

   companion object
}
```

我们类 `Mappable<T>` 用 `@higherkind` 注解，并扩展 `MappableKind<T>`，必须有一个伴随对象，无论它是空的还是非空的。

现在，我们需要创建我们的 `Functor<F>` 实现：

```java
import arrow.instance
import arrow.typeclasses.Functor

@instance(Mappable::class)
interface MappableFunctorInstance : Functor<MappableHK> {
   override fun <A, B> map(fa: MappableKind<A>, f: (A) -> B): Mappable<B> {
      return fa.ev().map(f)
   }
}
```

我们的 `MappableFunctorInstance` 接口扩展 `Functor<MappableHK>` 并用 `@instance(Mappable::class)` 注解。在 `map` 函数内部，我们使用第一个参数 `MappableKind<A>` 并使用其 `map` 函数。

`@instance` 注解将生成一个扩展接口的对象，`MappableFunctorInstance`。它将创建一个 `Mappable.Companion.functor()` 扩展函数，使用 `Mappable.functor()`（这是我们如何使用 `Option.monad()` 的方式）来获取实现 `MappableFunctorInstance` 的对象。

另一个替代方案是让 Arrow 衍生的实例自动提供，前提是你的数据类型具有正确的函数：

```java
import arrow.deriving 

@higherkind
@deriving(Functor::class)
class DerivedMappable<T>(val t: T) : DerivedMappableKind<T> {
   fun <R> map(f: (T) -> R): DerivedMappable<R> = DerivedMappable(f(t))

   override fun toString(): String = "DerivedMappable(t=$t)"

   companion object
}
```

`@deriving` 注解将生成 `DerivedMappableFunctorInstance`，这通常是你手动编写的。

现在，我们可以创建一个泛型函数来使用我们的 `Mappable` 函子：

```java
import arrow.typeclasses.functor

inline fun <reified F> buildBicycle(mapper: HK<F, Int>,
                           noinline f: (Int) -> Bicycle,
                           FR: Functor<F> = functor()): HK<F, Bicycle> = FR.map(mapper, f)
```

`buildBicycle` 函数将接受任何 `HK<F, Int>` 作为参数，并使用由 `arrow.typeclasses.functor` 函数返回的 `Functor` 实现应用函数 `f`，并返回 `HK<F, Bicycle>`。

函数 `arrow.typeclass.functor` 在运行时解析，符合 `Functor<MappableHK>` 要求的实例：

```java
fun main(args: Array<String>) {

   val mappable: Mappable<Bicycle> = buildBicycle(Mappable(3), ::Bicycle).ev()
   println("mappable = $mappable") //Mappable(t=Bicycle(gears=3))

   val option: Option<Bicycle> = buildBicycle(Some(2), ::Bicycle).ev()
   println("option = $option") //Some(Bicycle(gears=2))

   val none: Option<Bicycle> = buildBicycle(None, ::Bicycle).ev()
   println("none = $none") //None

}
```

我们可以使用 `buildBicycle` 与 `Mappeable<Int>` 或任何其他 HKT 类，例如 `Option<T>`。

使用箭头方法处理 HKT 的问题之一是它必须在运行时解析其实例。这是因为 Kotlin 没有对隐式或编译时解决类型类实例的支持，这使得 Arrow 只能选择这个替代方案，直到 *KEEP-87* 被批准并包含在语言中：

```java
@higherkind
class NotAFunctor<T>(val t: T) : NotAFunctorKind<T> {
   fun <R> map(f: (T) -> R): NotAFunctor<R> = NotAFunctor(f(t))

   override fun toString(): String = "NotAFunctor(t=$t)"
}
```

因此，你可以有一个具有 `map` 函数的 HKT，但没有 `Functor` 实例无法使用，但这并不是编译错误：

```java
fun main(args: Array<String>) {

   val not: NotAFunctor<Bicycle> = buildBicycle(NotAFunctor(4), ::Bicycle).ev()
   println("not = $not")

}
```

使用 `NotAFunctor<T>` 函数调用 `buildBicycle` 会编译，但在运行时将抛出 `ClassNotFoundException` 异常。

现在我们已经了解了 Arrow 的层次结构如何工作，我们可以介绍其他类。

# Either

`Either<L, R>` 表示两种可能值之一 `L` 或 `R`，但不是同时表示。`Either` 是一个密封类（类似于 `Option`），有两个子类型 `Left<L>` 和 `Right<R>`。通常 `Either` 用于表示可能失败的结果，使用左侧表示错误，右侧表示成功的结果。因为表示可能失败的操作是常见场景，Arrow 的 `Either` 是右偏的，换句话说，除非有其他说明，否则所有操作都在右侧运行。

让我们将我们的除法示例从 `Option` 转换为 `Either`：

```java
import arrow.core.Either
import arrow.core.Either.Right
import arrow.core.Either.Left

fun eitherDivide(num: Int, den: Int): Either<String, Int> {
   val option = optionDivide(num, den)
   return when (option) {
      is Some -> Right(option.t)
      None -> Left("$num isn't divisible by $den")
   }
}
```

现在而不是返回一个 `None` 值，我们正在向用户返回有价值的信息：

```java
import arrow.core.Tuple2

fun eitherDivision(a: Int, b: Int, den: Int): Either<String, Tuple2<Int, Int>> {
   val aDiv = eitherDivide(a, den)
   return when (aDiv) {
      is Right -> {
         val bDiv = eitherDivide(b, den)
         when (bDiv) {
            is Right -> Right(aDiv.getOrElse { 0 } toT bDiv.getOrElse { 0 })
            is Left -> bDiv as Either<String, Nothing>
         }
      }
      is Left -> aDiv as Either<String, Nothing>
   }
}
```

在 `eitherDivision` 中，我们使用 Arrow 的 `Tuple<A, B>` 而不是 Kotlin 的 `Pair<A, B>`。元组比 Pair/Triple 提供更多功能，从现在开始我们将使用它。要创建 `Tuple2`，可以使用扩展 `infix` 函数，`toT`。

接下来，是一个 `Either<L, R>` 函数的简短列表：

| **函数** | **描述** |
| --- | --- |
| `bimap(fa:(L) -> T, fb:(R) -> X): Either<T, X>` | 使用 `fa` 对 `Left` 进行转换，使用 `fb` 对 `Right` 进行转换，以返回 `Either<T, X>`。 |
| `contains(elem:R): Boolean` | 如果 `Right` 值与 `elem` 参数相同，则返回 `true`，对于 `Left` 返回 `false`。 |
| `exists(p:Predicate<R>):Boolean` | 如果 `Right`，则返回谓词 `p` 的结果，对于 `Left` 总是返回 `false`。 |
| `flatMap(f: (R) -> Either<L, T>): Either<L, T>` | 与 `Monad` 中的 `flatMap` 函数类似，使用 `Right` 的值。 |
| `fold(fa: (L) -> T, fb: (R) -> T): T` | 执行 `fa` 对 `Left` 和 `fb` 对 `Right` 返回 `T` 值。 |
| `getOrElse(default:(L) -> R): R` | 返回 `Right` 值，或 `default` 函数的结果。 |
| `isLeft(): Boolean` | 如果是 `Left` 的实例，则返回 `true`，对于 `Right` 返回 `false`。 |
| `isRight(): Boolean` | 如果是 `Right` 的实例，则返回 `true`，对于 `Left` 返回 `false`。 |
| `map(f: (R) -> T): Either<L, T>` | 与 `Functor` 中的 `map` 函数类似，如果 `Right`，则使用函数 `f` 将其转换为 `Right<T>`，如果 `Left`，则返回相同值而不进行转换。 |
| `mapLeft(f: (L) -> T): Either<T, R>` | 与 `Functor` 中的 `map` 函数类似，如果 `Left`，则使用函数 `f` 将其转换为 `Left<T>`，如果 `Right`，则返回相同值而不进行转换。 |
| `swap(): Either<R, L>` | 返回类型和值互换的 `Either`。 |
| `toOption(): Option<R>` | 对于 `Right` 返回 `Some<T>`，对于 `Left` 返回 `None`。 |

`flatMap` 版本看起来符合预期：

```java
fun flatMapEitherDivision(a: Int, b: Int, den: Int): Either<String, Tuple2<Int, Int>> {
   return eitherDivide(a, den).flatMap { aDiv ->
      eitherDivide(b, den).flatMap { bDiv ->
         Right(aDiv toT bDiv)
      }
   }
}
```

`Either` 具有单子实现，因此我们可以调用绑定函数：

```java
fun comprehensionEitherDivision(a: Int, b: Int, den: Int): Either<String, Tuple2<Int, Int>> {
   return Either.monad<String>().binding {
      val aDiv = eitherDivide(a, den).bind()
      val bDiv = eitherDivide(b, den).bind()

      aDiv toT bDiv
   }.ev()
```

注意 `Either.monad<L>()`；对于 `Either<L, R>`，它必须定义 `L` 类型：

```java
fun main(args: Array<String>) {
   eitherDivision(3, 2, 4).fold(::println, ::println) //3 isn't divisible by 4
}

```

在下一节中，我们将学习单子变换器。

# 单子变换器

`Either`和`Option`使用简单，但如果我们将两者结合会发生什么呢？

```java
object UserService {

   fun findAge(user: String): Either<String, Option<Int>> {
       //Magic  
   }
}
```

`UserService.findAge`返回`Either<String, Option<Int>>`；`Left<String>`表示访问数据库或其他基础设施时出错，`Right<None>`表示数据库中没有找到值，`Right<Some<Int>>`表示找到了值：

```java
import arrow.core.*
import arrow.syntax.function.pipe

fun main(args: Array<String>) {
 val anakinAge: Either<String, Option<Int>> = UserService.findAge("Anakin")

 anakinAge.fold(::identity, { op ->
         op.fold({ "Not found" }, Int::toString)
     }) pipe ::println 
}
```

要打印年龄，我们需要两个嵌套的折叠，没有什么太复杂的。问题出现在我们需要执行访问多个值的操作时：

```java
import arrow.core.*
import arrow.syntax.function.pipe
import kotlin.math.absoluteValue

fun main(args: Array<String>) {
   val anakinAge: Either<String, Option<Int>> = UserService.findAge("Anakin")
   val padmeAge: Either<String, Option<Int>> = UserService.findAge("Padme")

   val difference: Either<String, Option<Either<String, Option<Int>>>> = anakinAge.map { aOp ->
      aOp.map { a ->
         padmeAge.map { pOp ->
            pOp.map { p ->
               (a - p).absoluteValue
            }
         }
      }
   }

   difference.fold(::identity, { op1 ->
      op1.fold({ "Not Found" }, { either ->
         either.fold(::identity, { op2 -> 
            op2.fold({ "Not Found" }, Int::toString) })
      })
   }) pipe ::println
}
```

摩纳哥不组合，这使得这些操作很快就会变得复杂。但是，我们总是可以依赖列表解析，不是吗？现在，让我们看看以下代码：

```java
import arrow.core.*
import arrow.syntax.function.pipe
import arrow.typeclasses.binding
import kotlin.math.absoluteValue

fun main(args: Array<String>) {
   val anakinAge: Either<String, Option<Int>> = UserService.findAge("Anakin")
   val padmeAge: Either<String, Option<Int>> = UserService.findAge("Padme")

   val difference: Either<String, Option<Option<Int>>> = Either.monad<String>().binding {
      val aOp: Option<Int> = anakinAge.bind()
      val pOp: Option<Int> = padmeAge.bind()
      aOp.map { a ->
         pOp.map { p ->
            (a - p).absoluteValue
         }
      }
   }.ev()

   difference.fold(::identity, { op1 ->
      op1.fold({ "Not found" }, { op2 ->
         op2.fold({ "Not found" }, Int::toString) }) }) pipe ::println
}
```

这样更好，返回类型不那么长，`fold`也更易于管理。让我们看看以下代码片段中的嵌套列表解析：

```java
fun main(args: Array<String>) {
   val anakinAge: Either<String, Option<Int>> = UserService.findAge("Anakin")
   val padmeAge:  Either<String, Option<Int>> = UserService.findAge("Padme")

   val difference: Either<String, Option<Int>> = Either.monad<String>().binding {
      val aOp: Option<Int> = anakinAge.bind()
      val pOp: Option<Int> = padmeAge.bind()
      Option.monad().binding {
         val a: Int = aOp.bind()
         val p: Int = pOp.bind()
         (a - p).absoluteValue
      }.ev()
   }.ev()

   difference.fold(::identity, { op ->
      op.fold({ "Not found" }, Int::toString)
   }) pipe ::println
}
```

现在，我们有两个值和结果具有相同的类型。但我们还有一个选择，摩纳哥转换器。

**摩纳哥转换器**是两个摩纳哥的组合，可以作为一个整体执行。在我们的例子中，我们将使用`OptionT`（**Option Transformer**的缩写），因为`Option`是嵌套在`Either`内部的摩纳哥类型：

```java
import arrow.core.*
import arrow.data.OptionT
import arrow.data.monad
import arrow.data.value
import arrow.syntax.function.pipe
import arrow.typeclasses.binding
import kotlin.math.absoluteValue

fun main(args: Array<String>) {
   val anakinAge: Either<String, Option<Int>> = UserService.findAge("Anakin")
   val padmeAge: Either<String, Option<Int>> = UserService.findAge("Padme")

   val difference: Either<String, Option<Int>> = OptionT.monad<EitherKindPartial<String>>().binding {
      val a: Int = OptionT(anakinAge).bind()
      val p: Int = OptionT(padmeAge).bind()
      (a - p).absoluteValue
   }.value().ev()

   difference.fold(::identity, { op ->
      op.fold({ "Not found" }, Int::toString)
   }) pipe ::println
}
```

我们使用`OptionT.monad<EitherKindPartial<String>>().binding`。`EitherKindPartial<String>`摩纳哥意味着包装类型是`Either<String, Option<T>>`。

在`binding`块内部，我们使用`OptionT`对类型为`Either<String, Option<T>>`（技术上是对类型为`HK<HK<EitherHK, String>, Option<T>>`的值）的值调用`bind(): T`，在我们的例子中`T`是`Int`。

之前我们只使用了`ev()`方法，但现在我们需要使用`value()`方法来提取`OptionT`内部值。

在下一节中，我们将学习关于`Try`类型的内容。

# Try

**Try**表示可能成功或失败的计算。`Try<A>`是一个密封类，有两个可能的子类—`Failure<A>`，表示失败，`Success<T>`表示成功操作。

让我们用`Try`来写我们的除法示例：

```java
import arrow.data.Try

fun tryDivide(num: Int, den: Int): Try<Int> = Try { divide(num, den)!! }
```

创建`Try`实例的最简单方法是使用`Try.invoke`操作符。如果块内部抛出异常，它将返回`Failure`；如果一切顺利，例如返回`Success<Int>`，则`!!`操作符将抛出`NPE`，如果除法返回 null：

```java
fun tryDivision(a: Int, b: Int, den: Int): Try<Tuple2<Int, Int>> {
   val aDiv = tryDivide(a, den)
   return when (aDiv) {
      is Success -> {
         val bDiv = tryDivide(b, den)
         when (bDiv) {
            is Success -> {
               Try { aDiv.value toT bDiv.value }
            }
            is Failure -> Failure(bDiv.exception)
         }
      }
      is Failure -> Failure(aDiv.exception)
   }
}
```

让我们看看`Try<T>`函数的简短列表：

| **函数** | **描述** |
| --- | --- |
| `exists(p: Predicate<T>): Boolean` | 如果`Success<T>`返回`p`结果，在`Failure`时总是返回`false`。 |
| `filter(p: Predicate<T>): Try<T>` | 如果操作成功并且通过谓词`p`，则返回`Success<T>`，否则返回`Failure`。 |
| `<R> flatMap(f: (T) -> Try<R>): Try<R>` | `flatMap`函数，类似于摩纳哥。 |
| `<R> fold(fa: (Throwable) -> R, fb:(T) -> R): R` | 返回值转换为`R`，如果为`Failure`则调用`fa`。 |
| `getOrDefault(default: () -> T): T` | 返回值`T`，如果为`Failure`则调用默认值。 |
| `getOrElse(default: (Throwable) -> T): T` | 返回值`T`，如果为`Failure`则调用默认值。 |
| `isFailure(): Boolean` | 如果是 `Failure` 则返回 `true`，否则返回 `false`。 |
| `isSuccess(): Boolean` | 如果是 `Success` 则返回 `true`，否则返回 `false`。 |
| `<R> map(f: (T) -> R): Try<R>` | 如同在函子中转换函数。 |
| `onFailure(f: (Throwable) -> Unit): Try<T>` | 在 `Failure` 上执行操作。 |
| `onSuccess(f: (T) -> Unit): Try<T>` | 在 `Success` 上执行操作。 |
| `orElse(f: () -> Try<T>): Try<T>` | 在 `Success` 上返回自身或在 `Failure` 上返回 `f` 的结果。 |
| `recover(f: (Throwable) -> T): Try<T>` | 转换 `map` 函数用于 `Failure`。 |
| `recoverWith(f: (Throwable) -> Try<T>): Try<T>` | 转换 `flatMap` 函数用于 `Failure`。 |
| `toEither() : Either<Throwable, T>` | 转换为 `Either`——`Failure` 转换为 `Left<Throwable>`，`Success<T>` 转换为 `Right<T>`。 |
| `toOption(): Option<T>` | 转换为 `Option`——`Failure` 转换为 `None`，`Success<T>` 转换为 `Some<T>`。 |

`flatMap` 的实现与 `Either` 和 `Option` 非常相似，展示了拥有一个共同的命名和行为约定的价值：

```java
fun flatMapTryDivision(a: Int, b: Int, den: Int): Try<Tuple2<Int, Int>> {
   return tryDivide(a, den).flatMap { aDiv ->
      tryDivide(b, den).flatMap { bDiv ->
         Try { aDiv toT bDiv }
      }
   }
}
```

Monadic 理解也适用于 `Try`：

```java
fun comprehensionTryDivision(a: Int, b: Int, den: Int): Try<Tuple2<Int, Int>> {
   return Try.monad().binding {
      val aDiv = tryDivide(a, den).bind()
      val bDiv = tryDivide(b, den).bind()
      aDiv toT bDiv
   }.ev()
}
```

另一种使用 `MonadError` 实例的 monadic 理解：

```java
fun monadErrorTryDivision(a: Int, b: Int, den: Int): Try<Tuple2<Int, Int>> {
   return Try.monadError().bindingCatch {
      val aDiv = divide(a, den)!!
      val bDiv = divide(b, den)!!
      aDiv toT bDiv
   }.ev()
}
```

使用 `monadError.bindingCatch`，任何抛出异常的操作都会提升到 `Failure`，最后返回值会被包裹在 `Try<T>` 中。`MonadError` 也适用于 `Option` 和 `Either`。

# State

**State** 是一种提供函数式处理应用程序状态的结构的结构。`State<S, A>` 是 `S -> Tuple2<S, A>` 的抽象。**S** 代表状态类型，`Tuple2<S, A>` 是结果，其中 `S` 是新更新的状态，`A` 是函数返回值。

我们可以从一个简单的例子开始，一个返回两个东西的函数，一个价格和计算它的步骤。为了计算价格，我们需要加上 20% 的 `VAT`，如果 `price` 值超过某个阈值，则应用折扣：

```java
import arrow.core.Tuple2
import arrow.core.toT
import arrow.data.State

typealias PriceLog = MutableList<Tuple2<String, Double>>

fun addVat(): State<PriceLog, Unit> = State { log: PriceLog ->
    val (_, price) = log.last()
    val vat = price * 0.2
    log.add("Add VAT: $vat" toT price + vat)
    log toT Unit
}
```

我们有一个类型别名 `PriceLog` 用于 `MutableList<Tuple2<String, Double>>`。`PriceLog` 将是我们的 `State` 表示；每个步骤都用 `Tuple2<String, Double>` 表示。

我们的第一个函数 `addVat(): State<PriceLog, Unit>` 表示第一步。我们使用 `State` 构建器编写这个函数，它接收 `PriceLog`，在应用任何步骤之前的初始状态，并必须返回一个 `Tuple2<PriceLog, Unit>`，我们使用 `Unit` 因为在这个点上我们不需要价格：

```java
fun applyDiscount(threshold: Double, discount: Double): State<PriceLog, Unit> = State { log ->
    val (_, price) = log.last()
    if (price > threshold) {
        log.add("Applying -$discount" toT price - discount)
    } else {
        log.add("No discount applied" toT price)
    }
    log toT Unit
}
```

`applyDiscount` 函数是我们的第二步。我们在这里引入的唯一新元素是两个参数，一个用于 `threshold`，另一个用于 `discount`：

```java
fun finalPrice(): State<PriceLog, Double> = State { log ->
    val (_, price) = log.last()
    log.add("Final Price" toT price)
    log toT price
}
```

最后一步由函数 `finalPrice()` 表示，现在我们返回 `Double` 而不是 `Unit`：

```java
import arrow.data.ev
import arrow.instances.monad
import arrow.typeclasses.binding

fun calculatePrice(threshold: Double, discount: Double) = State().monad<PriceLog>().binding {
    addVat().bind() //Unit
    applyDiscount(threshold, discount).bind() //Unit
    val price: Double = finalPrice().bind()
    price
}.ev()
```

为了表示步骤序列，我们使用 monadic 理解并按顺序使用 `State` 函数。从一个函数到下一个函数，`PriceLog` 状态隐式流动（只是某些协程连续性的魔法）。最后，我们产生最终价格。添加新步骤或切换现有步骤就像添加或移动行一样简单：

```java
import arrow.data.run
import arrow.data.runA

fun main(args: Array<String>) {
    val (history: PriceLog, price: Double) = calculatePrice(100.0, 2.0).run(mutableListOf("Init" toT 15.0))
    println("Price: $price")
    println("::History::")
    history
            .map { (text, value) -> "$text\t|\t$value" }
            .forEach(::println)

    val bigPrice: Double = calculatePrice(100.0, 2.0).runA(mutableListOf("Init" toT 1000.0))
    println("bigPrice = $bigPrice")
}
```

要使用 `calculatePrice` 函数，你必须提供阈值和折扣值，然后使用初始状态调用扩展函数 `run`。如果你只对价格感兴趣，可以使用 `runA`，或者只对历史感兴趣，使用 `runS`。

避免使用 `State` 产生的问题。不要混淆扩展函数 `arrow.data.run` 与扩展函数 `kotlin.run`（默认导入）。

# 使用 `State` 的核心递归

`State` 在核心递归中很有用；我们可以用 `State` 重新编写我们的旧例子：

```java
fun <T, S> unfold(s: S, f: (S) -> Pair<T, S>?): Sequence<T> {
   val result = f(s)
   return if (result != null) {
      sequenceOf(result.first) + unfold(result.second, f)
   } else {
      sequenceOf()
   }
}
```

我们原始的 `unfold` 函数使用一个函数，`f: (S) -> Pair<T,S>?`，这与 `State<S, T>` 非常相似：

```java
fun <T, S> unfold(s: S, state: State<S, Option<T>>): Sequence<T> {
    val (actualState: S, value: Option<T>) = state.run(s)
    return value.fold(
            { sequenceOf() },
            { t ->
                sequenceOf(t) + unfold(actualState, state)
            })
}
```

我们不再使用 lambda `(S) -> Pair<T, S>?`，而是使用 `State<S, Option<T>>`，并使用 `Option` 的 fold 函数，对于 `None` 使用空 `Sequence`，对于 `Some<T>` 使用递归调用：

```java
fun factorial(size: Int): Sequence<Long> {
   return sequenceOf(1L) + unfold(1L to 1) { (acc, n) ->
      if (size > n) {
         val x = n * acc
         (x) to (x to n + 1)
      } else
         null
   }
}
```

我们旧的阶乘函数使用 `unfold` 与 `Pair<Long, Int>` 和 lambda—`(Pair<Long, Int>) -> Pair<Long, Pair<Long, Int>>?`：

```java
import arrow.syntax.option.some

fun factorial(size: Int): Sequence<Long> {
    return sequenceOf(1L) + unfold(1L toT 1, State { (acc, n) ->
        if (size > n) {
            val x = n * acc
            (x toT n + 1) toT x.some()
        } else {
            (0L toT 0) toT None
        }
    })
}
```

重新编写的阶乘使用 `State<Tuple<Long, Int>, Option<Long>>`，但内部逻辑几乎相同，尽管我们新的阶乘没有使用 null，这是一个显著的改进：

```java
fun fib(size: Int): Sequence<Long> {
   return sequenceOf(1L) + unfold(Triple(0L, 1L, 1)) { (cur, next, n) ->
      if (size > n) {
         val x = cur + next
         (x) to Triple(next, x, n + 1)
      }
      else
         null
   }
}

```

同样，`fib` 使用 unfold 与 `Triple<Long, Long, Int>` 和 lambda `(Triple<Long, Long, Int>) -> Pair<Long, Triple<Long, Long, Int>>?`：

```java
import arrow.syntax.tuples.plus

fun fib(size: Int): Sequence<Long> {
    return sequenceOf(1L) + unfold((0L toT 1L) + 1, State { (cur, next, n) ->
        if (size > n) {
            val x = cur + next
            ((next toT x) + (n + 1)) toT x.some()
        } else {
            ((0L toT 0L) + 0) toT None
        }
    })
}
```

重新编写的 `fib` 使用 `State<Tuple3<Long, Long, Int>, Option<Long>>`。请注意，扩展操作符函数 `plus`，与 `Tuple2<A, B>` 和 `C` 一起使用将返回 `Tuple3<A, B, C>`：

```java
fun main(args: Array<String>) {
    factorial(10).forEach(::println)
    fib(10).forEach(::println)
}
```

现在，我们可以使用我们的核心递归函数来生成序列。`State` 的其他许多用途我们在这里无法涵盖，例如来自 *企业集成模式* 的 *消息历史*（[`www.enterpriseintegrationpatterns.com/patterns/messaging/MessageHistory.html`](http://www.enterpriseintegrationpatterns.com/patterns/messaging/MessageHistory.html)）或具有多个步骤的表单导航，如飞机检查或长注册表单。

# 摘要

Arrow 提供了许多数据类型和类型类，可以显著简化复杂任务，并提供了一套标准的惯用和表达式。在本章中，我们学习了如何使用 `Option` 抽象 null 值，以及如何使用 `Either` 和 `Try` 表达计算。我们创建了一个数据类型类，还学习了单调组合和转换。最后但同样重要的是，我们使用了 `State` 来表示应用程序状态。

通过本章，我们到达了这次旅程的终点，但请放心，这并不是你学习函数式编程的终点。正如我们在第一章所学，函数式编程全部关于使用函数作为构建块来创建复杂程序。同样，通过在这里学到的所有概念，你现在可以理解和掌握新的、令人兴奋的、更强大的想法。

现在你开始了一段新的学习之旅。
