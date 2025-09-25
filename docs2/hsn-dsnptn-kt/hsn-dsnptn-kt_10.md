# 第十章：习语和反模式

本章讨论了 Kotlin 的最佳和最坏实践。你将了解惯用的 Kotlin 代码应该是什么样子，以及哪些模式应该避免。

完成这一章后，你应该能够编写更易读、更易于维护的 Kotlin 代码，并避免一些常见的陷阱。

在本章中，我们将涵盖以下主题：

+   Let

+   Apply

+   Also

+   Run

+   With

+   实例检查

+   尝试使用资源

+   内联函数

+   重新声明

+   常量

+   构造函数重载

+   处理空值

+   显式异步

+   验证

+   密封，而不是枚举

+   更多同伴

+   Scala 函数

# Let

通常，我们使用`let()`只在对象`not null`时执行某些操作：

```kt
val sometimesNull = if (Random().nextBoolean()) "not null" else null

sometimesNull?.let {
    println("It was $it this time")
}
```

这里一个常见的陷阱是`let()`本身也可以用于空值：

```kt
val alwaysNull = null

alwaysNull.let { // No null pointer there
    println("It was $it this time") // Always prints null
}
```

使用`let()`进行空值检查时，不要忘记问号`?`。

`let()`的返回值与其操作的类型无关：

```kt
val numberReturned = justAString.let {
    println(it)
    it.length
}
```

这段代码将打印`"string"`并返回其长度为`Int 6`。

# Apply

我们已经在之前的章节中讨论了`apply()`。它返回它操作的对象，并将上下文设置为`this`。这个函数最有用的用例是设置可变对象的字段。

想想你有多少次不得不创建一个空构造函数的类，然后依次调用很多 setter：

```kt
class JamesBond {
    lateinit var name: String
    lateinit var movie: String
    lateinit var alsoStarring: String
}

val agentJavaWay = JamesBond()
agentJavaWay.name = "Sean Connery"
agentJavaWay.movie = "Dr. No"
```

我们只能设置`name`和`movie`，但`alsoStarring`留空，如下所示：

```kt
val `007` = JamesBond().apply {
    this.name = "Sean Connery"
    this.movie = "Dr. No"
}

println(`007`.name)
```

由于上下文设置为`this`，我们可以将其简化为以下漂亮的语法：

```kt
val `007` = JamesBond().apply {
    name = "Sean Connery"
    movie = "Dr. No"
}
```

这个函数在你处理通常有很多 setter 的 Java 类时特别有用。

# Also

单表达式函数非常简洁和优雅：

```kt
fun multiply(a: Int, b: Int): Int = a * b
```

但通常，你有一个单语句函数，它还需要写入日志，例如。

你可以写成以下方式：

```kt
fun multiply(a: Int, b: Int): Int {
    val c = a * b
    println(c)
    return c
}
```

但这样它就不再是单语句函数了，对吧？

我们还引入了另一个变量。来拯救，`also()`：

```kt
fun multiply(a: Int, b: Int): Int = (a * b).also { println(it) }
```

这个函数将表达式的结果设置到`it`，并返回表达式的结果。

这也适用于你想要在一系列调用中产生副作用的情况：

```kt
val l = (1..100).toList()

l.filter{ it % 2 == 0 }
    .also { println(it) } // Prints, but doesn't change anything
    .map { it * it }

```

# Run

与线程无关，`run()`与`let()`类似，但它将上下文设置为`this`而不是使用`it`：

```kt
val justAString = "string"

val n = justAString.run { 
    this.length
}
```

通常，`this`可以省略：

```kt
val n = justAString.run { 
    length
}
```

当你计划在同一个对象上调用多个方法时，这非常有用，就像`apply()`一样。

与`apply()`不同，返回结果可能完全不同：

```kt
val year = JamesBond().run {
    name = "ROGER MOORE"
    movie = "THE MAN WITH THE GOLDEN GUN"
    1974 // <= Not JamesBond type
}
```

# With

与其他四个作用域函数不同，`with()`不是一个扩展函数。

这意味着你不能这样做：

```kt
"scope".with { ... }
```

相反，`with()`接收一个作为参数的对象，你想要作用域的对象：

```kt
with("scope") {
    println(this.length) // "this" set to the argument of with()
}
```

通常情况下，我们可以省略`this`：

```kt
with("scope") {
    length
}
```

就像`run()`和`let()`一样，你可以从`with()`返回任何结果。

# 实例检查

来自 Java，你可能倾向于检查你的对象类型，使用`is`，然后使用`as`进行转换：

```kt
interface Superhero
class Batman : Superhero {
    fun callRobin() {
        println("To the Bat-pole, Robin!")
    }
}

class Superman : Superhero {
    fun fly() {
        println("Up, up and away!")
    }
}

fun doCoolStuff(s : Superhero) {
    if (s is Superman) {
        (s as Superman).fly()
    }
    else if (s is Batman) {
        (a as Batman).callRobin()
    }
}
```

但正如你可能知道的，Kotlin 有智能转换，所以在这种情况下，隐式转换不是必需的：

```kt
fun doCoolStuff(s : Superhero) {
    if (s is Superman) {
        s.fly()
    }
    else if (s is Batman) {
        s.callRobin()
    }
}
```

此外，在大多数情况下，使用`when()`进行智能转换会产生更干净的代码：

```kt
fun doCoolStuff(s : Superhero) {
    when(s) {
        is Superman -> s.fly()
        is Batman -> s.callRobin()
        else -> println("Unknown superhero")
    }
}
```

作为一条经验法则，你应该避免使用类型转换，并尽可能多地依赖智能转换：

```kt
// Superhero is clearly not a string
val superheroAsString = (s as String)
```

但如果你绝对必须，还有一个安全的类型转换操作符：

```kt
val superheroAsString = (s as? String)
```

# 使用 try-with-resources

Java7 引入了 `AutoCloseable` 的概念和 try-with-resources 语句。

这个声明允许我们提供一组资源，这些资源在代码使用完毕后会自动关闭。不再有忘记关闭文件的风险（或者至少风险更小）。

在 Java7 之前，那是一个完全的混乱：

```kt
BufferedReader br = null; // Nulls are bad, we know that
try {
    br = new BufferedReader(new FileReader("/some/peth"));
    System.out.println(br.readLine());
}
finally {
    if (br != null) { // Explicit check
        br.close(); // Boilerplate
    }
}
```

在 Java7 之后：

```kt
try (BufferedReader br = new BufferedReader(new FileReader("/some/peth"))) {
    System.out.println(br.readLine());
}
```

在 Kotlin 中，`this` 语句被 `use()` 函数替换：

```kt
val br = BufferedReader(FileReader(""))

br.use {
    println(it.readLine())
}
```

# 内联函数

你可以将内联函数视为编译器的复制/粘贴指令。每次编译器看到标记为内联的函数调用时，它都会用具体的函数体替换调用。

只有当它是一个高阶函数，并且其中一个参数是 lambda 表达式时，才使用内联函数：

```kt
inline fun doesntMakeSense(something: String) {
    println(something)
}
```

这是最常见的使用场景，你希望使用 `inline`：

```kt
inline fun makesSense(block: () -> String) {
    println("Before")
    println(block())
    println("After")
}
```

你可以像往常一样调用它，使用代码块体：

```kt
makesSense {
    "Inlining"
}
```

但如果你查看字节码，你会发现它实际上被转换成了生成的行，而不是函数调用：

```kt
println("Before")
println("Inlining")
println("After")
```

在实际代码中，你会看到以下内容：

```kt
String var1 = "Before"; <- Inline function call
System.out.println(var1);
var1 = "Inlining";
System.out.println(var1);
var1 = "After";
System.out.println(var1);

var1 = "Before"; // <- Usual code
System.out.println(var1);
var1 = "Inlining";
System.out.println(var1);
var1 = "After";
System.out.println(var1);
```

注意，这两个代码块之间没有任何区别。

由于内联函数是复制/粘贴的，所以如果你有超过几行代码，就不应该使用它。将其作为常规函数会更有效率。

# Reified

由于内联函数是复制的，我们可以消除 JVM 的一项主要限制——类型擦除。毕竟，在函数内部，我们知道我们得到的确切类型。

让我们看看以下示例。你希望创建一个泛型函数，该函数将接收一个数字，但只有在它与函数类型相同时才会打印它。

你可以尝试编写如下内容：

```kt
fun <T> printIfSameType(a: Number) {
    if (a is T) { // <== Error
        println(a)   
    }
}
```

但这段代码会因为以下错误而无法编译：

```kt
Cannot check for instance of erased type: T
```

在这种情况下，我们通常在 Java 中这样做，即传递类作为参数：

```kt
fun <T: Number> printIfSameType(clazz: KClass<T>, a: Number) {
    if (clazz.isInstance(a) ) {
        println(a)
    }
}
```

我们可以通过运行以下两行来检查这段代码：

```kt
printIfSameType(Int::class, 1) // Print 1, as 1 is Int
printIfSameType(Int::class, 2L) // Prints nothing, as 2 is Long
```

这段代码有几个缺点：

+   我们不得不使用反射，为此，我们必须包含 `kotlin-reflect` 库：

```kt
compile group: 'org.jetbrains.kotlin', name: 'kotlin-reflect', version: '1.2.31'
```

+   我们不能使用 `is` 操作符，而必须使用 `isInstance()` 函数。

+   我们必须传递正确的类：

```kt
clazz: KClass<T>
```

相反，我们可以使用一个 `reified` 函数：

```kt
reified T> printIfSameTypeReified(a: Number) {
    if (a is T) {
        println(a)
    }
}
```

我们可以测试我们的代码是否仍然按预期工作：

```kt
printIfSameTypeReified<Int>(3) // Prints 3, as 3 is Int
printIfSameTypeReified<Int>(4L) // Prints nothing, as 4 is Long
printIfSameTypeReified<Long>(5) // Prints nothing, as 5 is Int
printIfSameTypeReified<Long>(6L) // Prints 6, as 6 is Long
```

这样我们就能获得该语言的所有好处：

+   没有必要使用另一个依赖项

+   清晰的方法签名

+   使用 `is` 构造的能力

当然，与常规内联函数相同的规则适用。这段代码将被复制，所以它不应该太大。

考虑另一个关于函数重载的例子：

```kt
fun printList(list: List<Int>) {
    println("This is a lit of Ints")
    println(list)
}

fun printList(list: List<Long>) {
    println("This is a lit of Longs")
    println(list)
}
```

这将无法编译，因为存在平台声明冲突。在 JVM 方面，两者具有相同的签名：`printList(list: List)`。

但使用 `reified`，我们可以实现这一点：

```kt
const val int = 1
const val long = 1L
inline fun <reified T : Any> printList(list: List<T>) {
    when {
        int is T -> println("This is a list of Ints")
        long is T -> println("This is a list of Longs")
        else -> println("This is a list of something else")
    }

    println(list)
}
```

# 常量

由于 Java 中的所有内容都是对象（除非你是原始类型），我们习惯于将所有常量作为静态成员放入我们的对象中。

由于 Kotlin 有伴随对象，我们通常尝试将它们放在那里：

```kt
class MyClass {
    companion object {
        val MY_CONST = "My Const"
    }
}
```

这将有效，但你应该记住，毕竟伴随对象是一个对象。

因此，这将被翻译成以下代码，或多或少：

```kt
public final class Spock {
   @NotNull
   private static final String MY_CONST = "My Const";
   public static final Spock.Companion Companion = new Spock.Companion(...);

   public static final class Companion {
      @NotNull
      public final String getMY_CONST() {
         return MyClass.MY_CONST;
      }

      private Companion() {
      }
   }
}
```

我们常量的调用看起来像这样：

```kt
String var1 = Spock.Companion.getSENSE_OF_HUMOR();
System.out.println(var1);
```

因此，我们在 `Spock` 类中有一个类，但我们想要的只是 `static final String`。

现在我们将这个值标记为常量：

```kt
class Spock {
    companion object {
        const val SENSE_OF_HUMOR = "None"
    }
}
```

这里是字节码的变化：

```kt
public final class Spock {
   @NotNull
   public static final String SENSE_OF_HUMOR = "NONE";
   public static final Spock.Companion Companion = new Spock.Companion(...);
   )
   public static final class Companion {
      private Companion() {
      }
        ...
   }
}
```

这里是调用：

```kt
String var1 = "NONE";
System.out.println(var1);
```

注意，根本就没有调用这个常量，因为编译器已经为我们内联了它的值。毕竟，它是常量。

如果你只需要一个常量，你也可以在任何类外部设置它：

```kt
const val SPOCK_SENSE_OF_HUMOR = "NONE"
```

如果你需要命名空间，你可以将其包裹在一个对象中：

```kt
object SensesOfHumor {
    const val SPOCK = "NONE"
}
```

# 构造函数重载

在 Java 中，我们习惯于有重载的构造函数：

```kt
class MyClass {
    private final String a;
    private final Integer b;
    public MyClass(String a) {
        this(a, 1);
    }

    public MyClass(String a, Integer b) {
        this.a = a;
        this.b = b;
    }
}
```

我们可以在 Kotlin 中模拟相同的行为：

```kt
class MyClass(val a: String, val b: Int, val c: Long) {

    constructor(a: String, b: Int) : this(a, b, 0) 

    constructor(a: String) : this(a, 1)

    constructor() : this("Default")
}
```

但通常更好的做法是使用默认参数值和命名参数：

```kt

class BetterClass(val a: String = "Default",
                  val b: Int = 1,
                  val c: Long = 0)
```

# 处理空值

空值是不可避免的，尤其是在你使用 Java 库或从数据库获取数据时。

但你可以用 Java 的方式检查空值：

```kt
// Will return "String" half of the time, and null the other half
val stringOrNull: String? = if (Random().nextBoolean()) "String" else null 

// Java-way check
if (stringOrNull != null) {
    println(stringOrNull.length)
}
```

或者用更短的形式，使用 `Elvis` 操作符。如果长度不为空，这个操作符将返回其值。否则，它将返回我们提供的默认值，在这个例子中是零：

```kt
val alwaysLength = stringOrNull?.length ?: 0

println(alwaysLength) // Will print 6 or 0, but never null
```

如果你有一个嵌套对象，你可以链式调用这些检查：

```kt
data class Json(
        val User: Profile?
)

data class Profile(val firstName: String?,
                   val lastName: String?)

val json: Json? = Json(Profile(null, null))

println(json?.User?.firstName?.length)
```

最后，你可以使用 `let()` 块来进行这些检查：

```kt
println(json?.let {
    it.User?.let {
        it.firstName?.length
    }
})
```

如果你想要消除所有地方的 `it()`，你可以使用 run：

```kt
println(json?.run {
    User?.run {
        firstName?.length
    }
})
```

尽可能避免不安全的 `!!` 空值操作符：

```kt
println(json!!.User!!.firstName!!.length)
```

这将导致 `KotlinNullPointerException`。

# 显式异步

正如你在上一章中看到的，在 Kotlin 中引入并发非常容易：

```kt
fun getName() = async {
   delay(100)
   "Ruslan"
}
```

但这种并发可能对函数的用户来说是不预期的行为，因为他们可能期望一个简单的值：

```kt
println("Name: ${getName()}")
```

它会打印：

```kt
Name: DeferredCoroutine{Active}@...
```

当然，这里缺少的是 `await()`：

```kt
println("Name: ${getName().await()}")
```

但如果我们相应地命名我们的函数，这会显得更加明显：

```kt
fun getNameAsync() = async {
   delay(100)
   "Ruslan"
}
```

通常，你应该建立某种约定来区分异步函数和常规函数。

# 验证

你有多少次不得不编写如下代码：

```kt
fun setCapacity(cap: Int) {
    if (cap < 0) {
        throw IllegalArgumentException()
    }
    ...
}
```

相反，你可以使用 `require()` 来检查参数：

```kt
fun setCapacity(cap: Int) {
    require(cap > 0)
}
```

这使得代码更加流畅。

你可以使用 `require()` 来检查嵌套的空值：

```kt
fun printNameLength(p: Profile) {
    require(p.firstName != null)
}
```

但也有 `requireNotNull()` 来处理这种情况：

```kt
fun printNameLength(p: Profile) {
    requireNotNull(p.firstName)
}
```

使用 `check()` 来验证你对象的状态。这在你提供用户可能没有正确设置的对象时很有用：

```kt
private class HttpClient {
    var body: String? = null
    var url: String = ""

    fun postRequest() {
        check(body != null) {
            "Body must be set in POST requests"
        }
    }

    fun getRequest() {
        // This one is fine without body
    }
}
```

再次强调，对于 `null` 有一个快捷方式：`checkNotNull()`。

# 密封，而不是枚举

来自 Java，你可能想给你的 `enum` 赋予功能：

```kt
// Java code
enum PizzaOrderStatus {
    ORDER_RECEIVED, 
    PIZZA_BEING_MADE, 
    OUT_FOR_DELIVERY, 
    COMPLETED;

    public PizzaOrderStatus nextStatus() {
        switch (this) {
            case ORDER_RECEIVED: return PIZZA_BEING_MADE;
            case PIZZA_BEING_MADE: return OUT_FOR_DELIVERY;
            case OUT_FOR_DELIVERY: return COMPLETED;
            case COMPLETED:return COMPLETED;
        }
    }
}
```

相反，你可以使用 `sealed` 类：

```kt
sealed class PizzaOrderStatus(protected val orderId: Int) {
    abstract fun nextStatus() : PizzaOrderStatus
    class OrderReceived(orderId: Int) : PizzaOrderStatus(orderId) {
        override fun nextStatus(): PizzaOrderStatus {
            return PizzaBeingMade(orderId)
        }
    }

    class PizzaBeingMade(orderId: Int) : PizzaOrderStatus(orderId) {
        override fun nextStatus(): PizzaOrderStatus {
            return OutForDelivery(orderId)
        }
    }

    class OutForDelivery(orderId: Int) : PizzaOrderStatus(orderId) {
        override fun nextStatus(): PizzaOrderStatus {
            return Completed(orderId)
        }
    }

    class Completed(orderId: Int) : PizzaOrderStatus(orderId) {
        override fun nextStatus(): PizzaOrderStatus {
            return this
        }
    }
}
```

这种方法的优点是，我们现在可以传递数据以及状态：

```kt
var status: PizzaOrderStatus = OrderReceived(123)

while (status !is Completed) {
    status = when (status) {
        is OrderReceived -> status.nextStatus()
        is PizzaBeingMade -> status.nextStatus()
        is OutForDelivery -> status.nextStatus()
        is Completed -> status
    }
}
```

通常，如果你想要将数据与状态关联起来，密封类是很好的选择。

# 更多伴随对象

你只能有一个伴随对象在你的类中：

```kt
class A {
   companion {
   }
   companion {
   }
}
```

但你可以在你的类中拥有任意多的对象：

```kt
class A {
   object B {
   }
   object C {
   }
}
```

这有时用于产生命名空间。命名空间很重要，因为它为你提供了更好的命名约定。想想看，当你创建了像 `SimpleJsonParser` 这样的类，它继承自 `JsonParser`，而 `JsonParser` 又继承自 `Parser` 时的情况。你可以将这个结构转换为 `Json.Parser`，例如，这要简洁得多，也更实用，因为 Kotlin 代码应该是这样的。

# Scala 函数

从 Scala 转向 Kotlin 的开发者有时可能会这样定义他们的函数：

```kt
fun hello() = {
    "hello"
}
```

调用此函数不会打印你期望的内容：

```kt
println("Say ${hello()}")
```

它会打印以下内容：

```kt
 Say () -> kotlin.String
```

我们缺少的是第二组括号：

```kt
println("Say ${hello()()}")
```

它会打印以下内容：

```kt
Say hello
```

这是因为单表达式定义可以翻译成：

```kt
fun hello(): () -> String {
    return {
        "hello"
    }
}
```

它可以进一步翻译成：

```kt
fun helloExpandedMore(): () -> String {
    return fun():String {
        return "hello"
    }
}
```

现在，你至少可以看到那个函数的来源了。

# 摘要

在本章中，我们回顾了 Kotlin 中的最佳实践以及语言的一些注意事项。现在你应该能够编写更符合语言习惯、性能良好且易于维护的代码。

你应该利用作用域函数，但确保不要过度使用它们，因为它们可能会使代码变得混乱，尤其是对于那些刚开始学习这门语言的人来说。

确保正确处理空值和类型转换，使用 `let()`、`Elvis` 操作符以及语言提供的智能转换。

在下一章和最后一章中，我们将通过编写一个真实生活中的微服务来应用这些技能，使用我们所学的一切。
