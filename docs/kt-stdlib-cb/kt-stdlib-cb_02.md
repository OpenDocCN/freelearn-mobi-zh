# 第二章：表达式函数和可调整接口

在本章中，我们将涵盖以下菜谱：

+   声明具有默认参数的可调整函数

+   声明包含默认实现的接口

+   扩展类的功能

+   解构类型

+   返回多个数据

+   内联闭包类型的参数

+   函数的内联表示法

+   使用泛型重载参数的智能类型检查

+   重载运算符

# 简介

本章将重点探讨一些 Kotlin 特性，这些特性可以帮助编写健壮、灵活且干净的函数和接口。阅读以下菜谱后，您将了解语言特定的支持和方法，用于减少样板代码和运行时性能改进。您还将了解标准库函数在底层是如何实现的，以及如何有效地与它们一起工作。

# 声明具有默认参数的可调整函数

当创建新函数时，我们经常需要允许一些参数是可选的。这迫使我们使用方法重载来创建具有相同名称但与不同用例和场景相关的不同参数集的多个函数声明。通常，在底层，每个函数变体都会调用具有默认实现的基本函数。让我们考虑一个简单的函数示例，该函数计算以恒定加速度率移动的物体的位移：

```java
fun calculateDisplacement(initialSpeed: Float, 
                          acceleration: Float, 
                          duration: Long): Double =
    initialSpeed * duration + 0.5 * acceleration * duration * duration
```

我们可能还需要为对象初始速度始终等于零的情况提供一个位移计算。在这种情况下，我们最终会以以下方式对基本函数进行重载：

```java
fun calculateDisplacement(acceleration: Float, duration: Long): Double = calculateDisplacement(0f, acceleration, duration)
```

然而，Kotlin 允许您通过单个具有可选参数的函数来减少多个声明，并处理多种不同的用例。在本菜谱中，我们将设计一个具有可选 `initialSpeed: Float` 参数的可调整版本的 `calculateDisplacement()` 函数。

# 如何做到这一点...

1.  让我们为该函数声明基本实现：

```java
fun calculateDisplacement(initialSpeed: Float, 
                          acceleration: Float, 
                          duration: Long): Double =
    initialSpeed * duration + 0.5 * acceleration * duration * 
    duration
```

1.  让我们为 `initialSpeed` 参数声明一个默认值：*

```java
fun calculateDisplacement(initialSpeed: Float = 0f, 
                          acceleration: Float, 
```

```java
                          duration: Long): Double =
    initialSpeed * duration + 0.5 * acceleration * duration * 
    duration
```

# 它是如何工作的...

我们已经为 `initialSpeed` 参数声明了一个默认值，等于 `0`。一旦我们分配了默认值，`initialSpeed` 参数就变成了可选的。现在，我们可以在调用函数时省略它，如下面的示例所示：

```java
val displacement = calculateDisplacement(acceleration = 9.81f, duration = 1000)
```

注意，如果我们省略了一些参数并使用它们的默认值，我们必须明确指定其他参数的值及其名称。这允许编译器将值映射到特定的参数。当然，我们能够使用标准方式覆盖默认值：

```java
val displacement = calculateDisplacement(10f, 9.81f, 1000)
```

# 参见

+   Kotlin 使得声明包含默认函数实现的接口成为可能。您可以在 *声明包含默认实现的接口* 菜谱中了解更多关于此功能的信息。

# 声明包含默认实现的接口

Kotlin 通过提供为其函数声明默认实现和定义其属性默认值的能力，使接口成为一个强大的语言元素。这些功能将接口提升到了一个新的水平，允许你将其用于比简单合同声明更高级的应用程序。

在这个菜谱中，我们将定义一个可重用的接口，用于验证用户在抽象注册表单的输入字段中输入的电子邮件地址值。该接口将提供两个函数。第一个函数负责解析电子邮件地址并决定给定的值是否为有效的电子邮件地址，第二个函数负责从表单中输入的电子邮件文本中提取用户的登录名。

# 准备工作

声明具有默认函数实现的接口很简单。我们不需要声明函数头，还需要包括其体：

```java
interface MyInterface {
    fun foo() {
        // default function body
    }
}
```

# 如何操作...

1.  声明一个新的接口，称为 `EmailValidator`：

```java
interface EmailValidator {}
```

1.  添加一个字符串属性，用于存储当前文本输入：

```java
interface EmailValidator {
    var input: String
}
```

1.  将 `isEmailValid()` 函数添加到接口中：

```java
interface EmailValidator {
    var input: String
    fun isEmailValid(): Boolean = input.contains("@")
}
```

1.  添加 `getUserLogin()` 函数：

```java
interface EmailValidator {
    var input: String

    fun isEmailValid(): Boolean = input.contains("@")

    fun getUserLogin(): String =
        if (isEmailValid()) {
            input.substringBefore("@")
        } else {
            ""
        }
}
```

# 它是如何工作的...

现在，让我们试一试，看看我们如何在实际操作中使用 `EmailValidator` 接口。假设我们有一个 `RegistrationForm` 类，它包含一个钩子方法，每次输入文本被修改时都会被调用：

```java
class RegistrationForm() {
    fun onInputTextUpdated(text: String) {
        // do some actions on text changed
    }
}
```

为了使用我们的 `EmailValidator` 接口，我们需要声明一个实现该接口的类。我们可以修改 `RegistrationForm` 类以实现 `EmailValidator` 接口：

```java
class RegistrationForm(override var input: String = ""): EmailValidator {
    fun onInputTextUpdated(newText: String) {
        this.input = newText

        if (!isEmailValid()) {
            print("Wait! You've entered wrong email...")
        } else {
            print("Email is correct, thanks: ${getUserLogin()}!")
        }
    }
}
```

每次调用 `onInputUpdated()` 函数时，我们都会更新在 `EmailValidator` 接口中声明的 `input: String` 属性。一旦更新完成，我们就使用 `EmailValidator` 接口的 `isEmailValid()` 和 `getUserLogin()` 函数值。将函数实现提取到接口中使得它们可以被重用，并轻松地集成到多个类中。唯一需要实际实现的部分是 `EmailValidator` 接口的 `input` 属性，它存储用户插入的文本的当前状态。以平滑的方式集成 `EmailValidator` 接口使其在可重用性和适应不同场景的应用程序中表现出色。

# 还有更多...

重要的是要记住，尽管我们可以在接口中定义默认函数实现，但我们无法为接口属性实例化默认值。与类属性不同，接口属性是抽象的。它们没有可以存储当前值（状态）的后备字段。如果我们在一个接口中声明一个属性，我们需要在实现该接口的类或对象中实现它。这是接口和抽象类之间的主要区别。抽象类可以有构造函数，可以存储属性及其实现。

与 Java 一样，我们不能扩展多个类；然而，我们可以实现多个接口。当我们有一个类实现了包含默认实现的多个接口时，我们可能会遇到由具有相同签名的函数引起的冲突：

```java
interface A {
    fun foo() {
        // some operations 
    }
}

interface B {
    fun foo() {
        // other operations
    }
}
```

在这种情况下，我们需要显式重写`foo()`函数以解决冲突：

```java
class MyClass: A, B {
    override fun foo() {
        print("I'm the first one here!")
    }
}
```

否则，我们会得到以下错误：

```java
Class 'MyClass' must override public open fun foo(): Unit because it inherits multiple interface methods of it.
```

# 参见

+   Kotlin 的一个类似特性是能够声明函数参数的默认值。您可以在*使用默认参数声明可调整函数*菜谱中了解更多信息。

# 扩展类的功能

在实现新功能或重构现有代码时，我们经常将代码的一部分提取到函数中，以便在不同的地方重用。如果提取的函数足够原子化，我们通常会将它导出到外部实用工具类中，这些类的首要目的是扩展现有类的功能。Kotlin 提供了一个有趣的替代方案。它提供了一个内置功能，允许我们使用*扩展函数*和*扩展属性*来扩展其他类的功能。

在这个菜谱中，我们将扩展`Array<T>`类的功能，并向其添加一个`swap(a:T, b: T)`扩展函数，该函数负责交换数组中两个给定元素的位置。

# 准备工作

我们可以在项目的任何文件中声明扩展函数和扩展属性。然而，为了保持良好的组织结构，最好将它们放在专门的文件中。

扩展函数的语法与标准函数的语法非常相似。我们只需要添加有关新函数扩展的类型信息，如下所示：

```java
fun SomeClass.newFunctionName(args): ReturnType {
    // body
}
```

# 如何做到这一点...

1.  创建一个新文件，名为`Extensions.kt`，用于存储扩展函数的定义。

1.  在其中实现`swap()`函数：

```java
fun <T> Array<T>.swap(a: T, b: T) {
    val positionA = indexOf(a)
    val positionB = indexOf(b)

    if (positionA < 0 || positionB < 0) {
        throw IllegalArgumentException("Given param doesn't belong
        to the array")
    }

    val tmp = this[positionA]
    this[positionA] = this[positionB]
    this[positionB] = tmp
}
```

# 它是如何工作的...

因此，我们能够对`Array`类的任何实例调用`swap`函数。让我们考虑以下示例：

```java
val array: Array<String> = arrayOf("a", "b", "c", "d")
array.swap("c", "b")
print(array.joinToString())
```

这将在控制台输出以下内容：

```java
a, c, b, d
```

如您所见，我们可以在扩展函数中使用`this`关键字来访问类的当前实例。

# 还有更多...

除了扩展函数之外，Kotlin 还提供了扩展属性功能。例如，我们可以为`List<T>`类声明一个属性，该属性将保存有关最后一个元素索引值的详细信息：

```java
val <T> List<T>.lastIndex: Int  get() = size - 1
```

扩展函数在 Kotlin 标准库的类中是一个广泛使用的模式。它们与 Java、Kotlin、JavaScript 以及项目内部和外部依赖中定义的本地类无缝工作。

# 解构类型

通常，将复杂类型的一个对象转换为多个变量是非常实用的。这允许你为变量提供适当的命名，并简化代码。Kotlin 提供了一个简单内置的功能来实现这一点，称为*解构*：

```java
data class User(val login: String, val email: String, val birthday: LocalDate)

fun getUser() = User("Agata", "ag@t.pl", LocalDate.of(1990, 1, 18))

val (name, mail, birthday) = getUser()

print("$name was born on $birthday")
```

因此，这段代码会在控制台打印以下信息：

```java
Agata was born on 1990-01-18
```

非常棒！解构对于数据类是开箱即用的。Kotlin 标准库还为许多常见类型提供了这个功能。然而，当我们处理自定义的非数据类时，解构并不是明确可用的。特别是，当我们与其他语言（如 Java）编写的类库中的类一起工作时，我们需要手动定义解构机制。在这个菜谱中，我们将为以下定义的 Java 类实现解构：

```java
// Java code
public class LightBulb {
    private final int id;
    private boolean turnedOn = false;

    public LightBulb(int id) {
        this.id = id;
    }

    public void setTurnedOn(boolean turnedOn) {
        this.turnedOn = turnedOn;
    }

    public boolean getTurnedOn() {
        return turnedOn;
    }

    public int getId() {
        return id;
    }
}
```

# 准备工作

Kotlin 中的解构声明是位置相关的，与在其他语言中可用的基于属性名称的声明相反。这意味着 Kotlin 编译器根据属性的顺序决定哪个类属性与哪个变量相关联。为了允许自定义类的解构，我们需要添加名为`componentN`的函数的实现，其中*N*代表带有`operator`关键字的标记的组件编号，以便在解构声明中使用它们。

# 如何实现...

1.  声明一个扩展函数，返回`LightBulb`类的`id`属性：

```java
operator fun LightBulb.component1() = this.id

```

1.  添加另一个扩展`componentN`函数，用于返回`turnedOn`属性：

```java
operator fun LightBulb.component2() = this.turnedOn
```

# 它是如何工作的...

一旦我们声明了适当的`componentN`函数，我们就可以从`LightBulb`类型对象的解构中受益：

```java
val (id, turnedOn) = LightBulb(1)
print("Light bulb number $id is turned ${if (turnedOn) "on" else "off"}")
```

这段代码会在控制台打印以下输出：

```java
Light bulb number 1 is turned off
```

如您所见，`component1()`函数被分配给解构声明的第一个变量——`id`。同样，第二个`turnedOn`变量被分配给`component2()`函数的结果。

# 还有更多...

由于解构对象赋值中的属性是位置相关的，有时我们被迫声明比我们想要使用的变量更多的变量。如果我们不需要使用某个值，我们可以使用下划线，以避免编译器提示未使用的变量，并稍微简化代码：

```java
val (_, turnedOn) = LightBulb(1)
print("Light bulb is turned ${if (turnedOn) "on" else "off"}")
```

解构也适用于函数返回值：

```java
val (login, domain) = "agata@magdalena.com".split("@")
print("login: $login, domain: $domain")
```

上述代码将返回以下输出：

```java
login: agata, domain: magdalena.com
```

我们还可以在 lambda 表达式中使用解构声明：

```java
listOf(LightBulb(0), LightBulb(1))
        .filter { (_, isOn) -> isOn }
        .map { (id, _) -> id }
```

解构声明的一个有用应用是迭代。例如，我们可以使用此功能遍历映射条目：

```java
val lightBulbsWithNames = 
        mapOf(LightBulb(0) to "Bedroom", LightBulb(1) to "Kitchen")

for ((lightbulb, name) in lightBulbsWithNames) {
    lightbulb.turnedOn = true
}
```

# 返回多个数据

虽然 Kotlin 没有提供多返回功能，但由于数据类和解构声明，编写返回多个不同类型值的函数相当方便。在这个菜谱中，我们将实现一个返回两个数字除法结果的函数。结果将包含商和余数值。

# 如何操作...

1.  让我们从声明一个用于返回类型的数据类开始：

```java
data class DivisionResult(val quotient: Int, val remainder: Int)
```

1.  让我们实现`divide()`函数：

```java
fun divide(dividend: Int, divisor: Int): DivisionResult {
    val quotient = dividend.div(divisor)
    val remainder = dividend.rem(divisor)
    return DivisionResult(quotient, remainder)
}
```

# 它是如何工作的...

我们可以看到`divide()`函数的作用：

```java
val dividend = 10
val divisor = 3
val (quotient, remainder) = divide(dividend, divisor)

print("$dividend / $divisor = $quotient r $remainder")
```

以下代码将打印以下输出：

```java
10 / 3 = 3 r 1
```

由于我们返回的是数据类实例，即`DivisionResult`类，我们可以利用解构特性并将结果分配给一组单独的变量。

# 还有更多...

Kotlin 标准库提供了现成的`Pair`和`Triple`类。我们可以使用它们来返回任意类型的两个和三个值。这消除了为返回类型创建专用数据类的需求。另一方面，使用数据类使我们能够使用更有意义的名称，这增加了代码的清晰度。

以下示例演示了使用`Pair`类同时返回两个对象：

```java
fun getBestScore(): Pair<String, Int> = Pair("Max", 1000)
val (name, score) = getBestScore()
print("User $name has the best score of $score points")
```

# 相关内容

+   如果你想要更熟悉解构声明，可以查看**解构类型**菜谱。

# 内联闭包类型的参数

使用高阶函数可能会导致运行时性能下降。将函数作为 lambda 参数传递以及它们在函数体内的虚拟调用会导致运行时开销。然而，在许多情况下，我们可以通过内联 lambda 表达式参数来消除这种开销。

在这个菜谱中，我们将实现`lock()`函数，该函数将自动化与 Java `java.util.concurrent.locks.Lock`接口的工作。该函数将接受两个参数——`Lock`接口的一个实例以及在获取锁后应调用的函数。最后，我们的`lock()`函数应释放锁。我们还希望允许内联函数参数。

# 准备工作

要声明内联函数，我们只需在函数头部之前添加`inline`修饰符。

# 如何操作...

让我们声明一个带有两个参数的`lock()`函数——`Lock`接口的一个实例以及获取锁后要调用的函数：

```java
inline fun performHavingLock(lock: Lock, task: () -> Unit) {
    lock.lock()
    try {
       task()
    }
    finally {
        lock.unlock()
    }
}
```

# 它是如何工作的...

`performHavingLock()`函数允许我们为其`task`参数提供的函数提供同步。

```java
performHavingLock(ReentrantLock()) {
 print("Wait for it!")
}
```

因此，`performHavingLock()`函数将打印以下输出到控制台：

```java
Wait for it!
```

在底层，`inline`修饰符会影响函数本身以及传递给它的 lambda 表达式。它们都将内联到生成的字节码中：

```java
Lock lock = (Lock)(new ReentrantLock());
lock.lock();

try {
   String var2 = "Wait for it!";
   System.out.print(var2);
} finally {
   lock.unlock();
}
```

如果我们没有使用`inline`修饰符，编译器将创建一个`Function0`类型的单独实例，以便将 lambda 参数传递给`performHavingLock()`函数。内联 lambda 可能会导致生成的代码增长。然而，如果我们以合理的方式（即避免内联大型函数）进行内联，这将有助于性能。

# 还有更多...

如果你只想将函数传递的一些 lambda 表达式内联，你可以使用`noinline`修饰符标记一些函数参数：

```java
inline fun foo(inlined: () -> Unit, noinline notInlined: () -> Unit) {  
    // ... 
}
```

Kotlin 还允许声明内联类属性。`inline`修饰符可以与没有后置字段的属性的 getter 和 setter 方法一起使用。例如：

```java
val foo: Foo  
    inline get() = Foo()  

var bar: Bar  
    get() = ...  
    inline set(v) { ... }
```

我们还可以注释整个属性：

```java
inline var bar: Bar  
    get() = ...  
    set(v) { ... }
```

因此，内联的 getter 和 setter 将与常规的内联函数以相同的方式进行表示。

# 函数的中缀表示法

为了使我们的代码更接近自然语言，Kotlin 为包含单个参数的函数提供了中缀表示法。这样，我们可以不使用括号来调用函数。在本菜谱中，我们将学习如何为`String`类型设计一个名为`concat()`的中缀扩展函数，该函数负责连接两个字符串值。

# 准备中

为了启用函数的中缀表示法，我们只需在函数标题之前添加`infix`关键字。

# 如何做到...

声明`concat()`扩展函数并实现其主体：

```java
infix fun String.concat(next: String): String = this + next
```

# 它是如何工作的...

让我们通过运行以下代码来测试`concat()`函数：

```java
print("This" concat "is" concat "weird")
```

太棒了！我们刚刚将以下文本打印到控制台：

```java
Thisisweird
```

# 还有更多...

Kotlin 标准库广泛使用中缀表示法。你可以利用中缀函数以整洁的方式塑造你的代码。一个值得注意的中缀函数是`Map.Entry<K, V>`类提供的`to()`扩展函数，它允许你以极简的方式声明映射条目：

```java
val namesWithBirthdays: Map<String, LocalDate> =
        mapOf("Agata" to LocalDate.of(1990, 1, 18))
```

`to()`扩展函数是为泛型类型`A`和泛型参数类型`B`声明的，它返回一个`Pair<A, B>`类的实例。

标准库中提供了许多支持中缀表示法的其他函数。如果你检查你每天使用的函数的实现，可能会发现它们也有中缀形式。

# 参见

+   你可以在*重载运算符*菜谱中了解另一个有助于使代码更易于阅读的酷特性。

# 使用泛型重载参数的智能类型检查

在实现支持泛型类型参数的函数时，我们经常需要提供有关对象类型在运行时的额外信息。在 JVM 平台上，类型在`Class<T>`类实例中有它们的表示。例如，当我们使用`Gson`库将 JSON 格式的数据解析到 Kotlin 类实例时，我们可能会遇到这样的需求：

```java
data class ApiResponse(val gifsWithPandas: List<ByteArray>)
data class Error(val message: String)

fun parseJsonResponse(json: String): ApiResponse {
    Gson().fromJson(json, ApiResponse::class.java)
}
```

通常，由于 JVM 类型擦除，我们无法在运行时访问泛型类型参数。然而，Kotlin 允许你克服这一限制，因为它在运行时保留了类型参数。在这个示例中，我们将调整 Gson 的`fromJson(json: String, Class<T>)`函数，以消除额外的类型参数。

# 准备工作

确保你的项目中包含了 Gson 依赖项（[`github.com/google/gson`](https://github.com/google/gson)）。如果你使用 Gradle，可以在构建脚本中使用以下声明来获取它：

```java
dependencies {
    compile 'com.google.code.gson:gson:2.8.2'
}
```

为了在运行时使类型参数可访问，我们需要用`reified`修饰符标记它，并将函数标记为`inline`。

# 如何实现...

1.  创建一个新文件，我们将在这里放置扩展函数的实现（例如，`GsonExtensions.kt`）

1.  在文件内部，声明一个`Gson`类的扩展函数：

```java
inline fun <reified T> Gson.fromJson(json: String): T { 
    return fromJson(json, T::class.java)
}
```

# 工作原理...

我们已经为`Gson`类型实现了一个扩展函数。由于添加了`reified`修饰符，我们可以在运行时访问泛型类型参数并将其传递给原始的`fromGson()`函数。

结果，我们能够在代码中使用更优雅版本的`fromGson()`函数：

```java
data class ApiResponse(val gifsWithPandas: List<ByteArray>)

val response = Gson().fromJson<ApiResponse>(json)
```

我们还可以利用 Kotlin 智能转换并从函数调用中省略显式的类型声明：

```java
val response: ApiResponse = Gson().fromJson(json)
```

# 重载运算符

Kotlin 语言提供了一套具有自己符号（例如，`+`、`-`、`*`或`/`）和优先级的运算符。在编译时，Kotlin 编译器将它们转换为关联函数调用或更复杂的语句。我们还可以重载运算符并为其指定的类型声明自定义底层实现。此实现将应用于使用运算符的指定类型的实例。

在这个示例中，我们将定义一个名为`Position`的类，表示三维空间中点的当前坐标。然后，我们将为我们的类实现自定义的`plus`和`minus`运算符，以便提供一种简单的方式来对其实例应用几何变换。结果，我们希望能够使用`+`和`-`运算符符号更新由`Position`类表示的点的坐标。

# 准备工作

为了为特定类型重载运算符，我们需要提供一个具有与运算符对应的固定名称的成员函数或扩展函数。此外，重载运算符的函数需要用`operator`关键字标记。

在以下表中，您可以找到语言中可用的操作符分组及其对应的表达式，它们被转换成：

**一元前缀**

| **操作符** | **表达式** |
| --- | --- |
| `+a` | `a.unaryPlus()` |
| `-a` | `a.unaryMinus()` |
| `!a` | `a.not()` |

**增量与减量**

| **操作符** | **表达式** |
| --- | --- |
| `a++` | `a.inc()` |
| `a--` | `a.dec()` |

**算术**

| **操作符** | **表达式** |
| --- | --- |
| `a + b` | `a.plus(b)` |
| `a - b` | `a.minus(b)` |
| `a * b` | `a.times(b)` |
| `a / b` | `a.div(b)` |
| `a % b` | `a.rem(b)` |
| `a..b` | `a.rangeTo(b)` |

**内联操作符**

| **操作符** | **表达式** |
| --- | --- |
| `a in b` | `b.contains(a)` |
| `a !in b` | `!b.contains(a)` |

**索引访问**

| **操作符** | **表达式** |
| --- | --- |
| `a[i]` | `a.get(i)` |
| `a[i, j]` | `a.get(i, j)` |
| `a[i_1, ..., i_n]` | `a.get(i_1, ..., i_n)` |
| `a[i] = b` | `a.set(i, b)` |
| `a[i, j] = b` | `a.set(i, j, b)` |
| `a[i_1, ..., i_n] = b` | `a.set(i_1, ..., i_n, b)` |

**调用操作符**

| **操作符** | **表达式** |
| --- | --- |
| `a()` | `a.invoke()` |
| `a(i)` | `a.invoke(i)` |
| `a(i, j)` | `a.invoke(i, j)` |
| `a(i_1, ..., i_n)` | `a.invoke(i_1, ..., i_n)` |

**增强赋值**

| **操作符** | **表达式** |
| --- | --- |
| `a += b` | `a.plusAssign(b)` |
| `a -= b` | `a.minusAssign(b)` |
| `a *= b` | `a.timesAssign(b)` |
| `a /= b` | `a.divAssign(b)` |
| `a %= b` | `a.remAssign(b)` |

**等式和比较**

| **操作符** | **表达式** |
| --- | --- |
| `a == b` | `a?.equals(b) ?: (b === null)` |
| `a != b` | `!(a?.equals(b) ?: (b === null))` |
| `a > b` | `a.comareTo(b) > 0` |
| `a < b` | `a.compareTo(b) < 0` |
| `a >= b` | `a.compareTo(b) >= 0` |
| `a <= b` | `a.compareTo(b) <= 0` |

# 如何做到这一点...

1.  使用`x`、`y`、`z`属性声明与笛卡尔坐标系中当前位置相关的`Position`数据类：

```java
data class Position(val x: Float, val y: Float, val z: Float)
```

1.  为`Position`类添加一个`plus`操作符实现：

```java
data class Position(val x: Float, val y: Float, val z: Float) {
    operator fun plus(other: Position) = 
      Position(x + other.x, y + other.y, z + other.z)
}
```

1.  覆盖`minus`操作符：

```java
data class Position(val x: Float, val y: Float, val z: Float) {
    operator fun plus(other: Position) = 
      Position(x + other.x, y + other.y, z + other.z)

    operator fun minus(other: Position) = 
      Position(x - other.x, y - other.y, z - other.z)
}
```

# 它是如何工作的...

现在我们可以使用`Position`类以及`plus`和`minus`操作符。让我们尝试使用减号操作符：

```java
val position1 = Position(132.5f, 4f, 3.43f)
val position2 = position1 - Position(1.5f, 400f, 11.56f)
print(position2)
```

就这样。前面的代码将打印以下结果到控制台：

```java
Position(x=131.0, y=-396.0, z=-8.13)
```

# 还有更多...

一些操作符有它们对应的复合*赋值*操作符定义。一旦我们覆盖了`plus`和`minus`操作符，我们就可以自动使用`plusAssign (+=)`和`minusAssign (-=)`操作符。例如，我们可以使用`plusAssign`操作符来更新`Position`实例状态如下：

```java
var position = Position(132.5f, 4f, 3.5f)
position += Position(1f, 1f, 1f)
print(position)
```

因此，我们将得到以下状态的`position`变量：

```java
Position(x=133.5, y=5.0, z=4.5)
```

重要的是要注意，*赋值*操作符返回`Unit`*.*这使得它在更新实例时，在内存分配效率方面比原始基本操作符（例如，`plus`或`minus`）是一个更好的选择。相比之下，基本操作符每次都会返回新的实例。

值得注意的是，Kotlin 还提供了对 Java 类的运算符重载。要重载运算符，我们只需向具有运算符名称和`public`可见性修饰符的类中添加一个适当的方法。以下是具有重载`plus`运算符的`Position`类的 Java 版本：

```java
public class Position { 
        private final float x; 
        private final float y; 
        private final float z;

        public Position(float x, float y, float z) { 
            this.x = x; 
            this.y = y; 
            this.z = z;
        } 

        public int getX() { 
            return x; 
        } 

        public int getY() { 
            return y; 
        } 

        public float getZ() {
            return z;
        }

```

```java
       public Position plus(Position pos) { 
 return new Position(pos.getX() + x, pos.getY() + y,
            pos.getZ() + z); 
 } 
}
```

在 Kotlin 代码中，可以这样使用它：

```java
val position = Position(2.f, 9.f, 55.5f) += (2.f, 2.f, 2.f)
```

Kotlin 标准库还包含不同运算符的预定义实现。你应每天使用的运算符之一是`MutableCollection`类型的`plus`运算符。这允许以下方式向集合中添加新元素：

```java
val list = mutableListOf("A", "B", "C")
list += "D"
print(list)
```

因此，前面的代码将在控制台打印以下输出：

```java
[A, B, C, D]
```
