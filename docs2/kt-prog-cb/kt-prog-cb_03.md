# 第三章：类和对象

本章将涵盖以下食谱：

+   初始化构造函数的主体

+   将一种数据类型转换为另一种类型

+   如何检查一个对象的数据类型

+   如何在 Kotlin 中使用抽象类

+   如何在 Kotlin 中遍历类的属性

+   如何使用内联属性

+   如何使用嵌套类

+   在 Kotlin 中获取类

+   使用委托属性

+   使用枚举

# 简介

在本章中，你将了解与 Kotlin 面向对象编程相关的食谱。使用面向对象的方法，你可以通过创建对象将复杂问题分解为更小的问题。与 Java 相比，Kotlin 的 OOP 风格有一些不同——例如，在 Kotlin 中，所有类默认都是封闭的（final），如果你想使它们可扩展，你需要使用`open`关键字。不仅对于类——默认情况下，方法也是 final 的，你需要使用`open`关键字。使用 Kotlin，处理类和对象所需的代码更少。哦！顺便说一句，我告诉你我们甚至不需要使用`new`关键字来创建对象了吗？所以，在 Kotlin 中创建新对象就像这样：

```java
var person=Person()
```

上述代码将创建一个可变类型的`Person`对象，因为我们使用了`var`作为修饰符。可变对象意味着它可以改变其值。如果你想创建一个不可变对象，你可以使用`val`关键字。所以同样的例子将如下所示：

```java
val person=Person()
```

因此，让我们开始查看一些有助于你在 Kotlin 中进行面向对象编程的食谱。

# 初始化构造函数的主体

在 Java 世界中，我们通常在构造函数中初始化类的字段，如下面的代码所示：

```java
class Student{
 int roll_number;
 String name;
 Student(int roll_number,String name){
     this.roll_number =roll_number;
     this.name = name;
 }
}
```

因此，如果参数的名称与属性的名称相似（通常为了使代码更易读），我们需要使用`this`关键字。在这个食谱中，我们将看到如何在 Kotlin 中实现相同的功能（显然代码更少）。

# 准备工作

你需要一个 IDE 来编写和执行你的代码。我将使用 IntelliJ IDEA。我们将创建一个具有`name`和`roll_number`属性的`Student`类。

# 如何做到...

让我们看看初始化构造函数的步骤：

1.  Kotlin 提供了一种语法，可以用更少的代码初始化属性。以下是 Kotlin 中类初始化的示例：

```java
class Student(var roll_number:Int, var name:String)
```

1.  你甚至不需要定义类的主体，属性的初始化仅在主构造函数中发生（主构造函数是类头的一部分）。显然，你可以根据是否需要保持属性可变来选择`var`或`val`。现在，如果你尝试创建一个对象，你可以用以下方式做到：

```java
var student_A=Student(1,"Rashi Karanpuria")
```

1.  为了确认，让我们尝试打印其属性以查看我们是否能够初始化它：

```java
println("Roll number: ${student_A.roll_number} Name: ${student_A.name}")
```

这是输出：

```java
 Roll number: 1 Name: Rashi Karanpuria
```

1.  然而，如果你想的话，你还可以在构造函数中放置默认值：

```java
class Student constructor(var roll_number:Int, var name:String="Sheldon")
```

1.  然后，你可以创建如下对象：

```java
var student_sheldon= Student(25)   // Object with name Sheldon and age 25

var student_amy=Student(25, "Amy")     // Object with name Amy and age 25
```

1.  如果类有一个主构造函数，每个次级构造函数都需要委托给主构造函数，无论是直接还是通过另一个次级构造函数（s）间接地。

1.  我们使用此关键字来委托给同一类的另一个构造函数：

```java
class Person(val name: String) {
     constructor(name: String, lastName: String) : this(name) {
         // Do something maybe
     }
 }
```

1.  我们也可能遇到需要在类中初始化其他事情的情况，而不仅仅是类的属性。这种情况可能是打开数据库连接，例如。在 Java 中，这是在构造函数中完成的，但在 Kotlin 中，我们有`init`块。初始化代码可以放入`init`块中：

```java
class Student(var roll_number:Int,var name: String) {
 init {
         logger.info("Student initialized")
     }
 }
```

1.  有时，我们也会通过依赖注入来初始化类的属性。如果你使用过 Dagger2，你一定熟悉对象被直接注入到类的构造函数中。为此，我们在构造函数关键字之前添加`@Inject`注解。每当构造函数有一个注解或可见性修饰符时，我们都需要有`constructor`关键字。下面是一个`constructor`关键字的示例：

```java
class Student @Inject constructor(compositeDisposable: CompositeDisposable) { ... }
```

1.  在这里，我们正在将`CompositeDisposable`类型的对象注入到构造函数中，由于我们使用注解（`@Inject`）来这样做，我们需要应用构造函数关键字。

1.  当你扩展一个类时，你需要初始化超类。在 Kotlin 中，这也非常简单。如果你的类有一个主构造函数，基类型必须在那里使用主构造函数的参数进行初始化。以下是一个相同的示例：

```java
class Student constructor(var roll_number:Int, var name:String):Person(name)
```

1.  然而，有时一个类可能没有主构造函数。在这种情况下，每个次级构造函数必须使用`super`关键字初始化基类型，或者可以委托给另一个执行此操作的构造函数。此外，不同的次级构造函数可以调用基类型的不同构造函数：

```java
class Student: Person {
 constructor(name: String) : super(name)
constructor(name: String, roll_number: Inte) :super(name)
 }
```

# 将一种数据类型转换为另一种类型

在 Java 中，我们通常通过在变量前添加所需类型来进行类型转换，如下所示：

```java
String a = Integer.toString(10)
```

此外，在 Java 中，数值可以直接转换为更大的数值类型，但在 Kotlin 中，这个特性因为类型安全而不存在——那么在 Kotlin 中如何将一个类型的对象转换为另一个类型的对象呢？我们将在本食谱中看到。

# 准备工作

你需要安装一个首选的开发环境，该环境可以编译和运行 Kotlin。你也可以使用命令行来完成这个任务，这需要安装 Kotlin 编译器和 JDK。我正在使用[`try.kotlinlang.org/`](https://try.kotlinlang.org/)上的在线 IDE 来编译和运行我的 Kotlin 代码，用于本食谱。

# 如何做到这一点...

让我们按照以下步骤了解如何将一种数据类型转换为另一种类型：

1.  让我们尝试一个非常基础的例子——尝试将`Int`转换为`Long`和`Float`：

```java
fun main(args: Array<String>) {
    var a = 1
    var b: Float = a.toFloat()
    var c = a.toLong()
    println("$a is Int while $b is Float and $c is Long")
}
```

1.  类似地，`Long`可以转换为`Float`和`Int`，如下所示：

```java
fun main(args: Array<String>) {
    var a = 1000000000000000000L
    var b: Float = a.toFloat()
    var c = a.toInt()
    println("$a is Long while $b is Float and $c is Integer")
}
```

上述代码的输出如下所示：

```java
1000000000000000000 is Long while 9.9999998E17 is Float and -1486618624 is Integer
```

1.  让我们尝试一个更有趣的转换，使用`Byte`、`Int`和`Strings`：

```java
fun main(args: Array<String>) {
    var a = 15623
    var b: Byte = a.toByte()
    var c = a.toString()
    println("$a is Int while $b is Byte and $c is String")
}
```

下面是一个可以使用于 Kotlin 中类型转换的方法列表：

+   `toByte()`: Byte

+   `toShort()`: Short

+   `toInt()`: 整数

+   `toLong()`: 长整数

+   `toFloat()`: 浮点数

+   `toDouble()`: 双精度浮点数

+   `toChar()`: 字符

+   `toString()`: 字符串

# 它是如何工作的...

基本上，Kotlin 是一种类型安全的语言，并确保类型不能在语言中直接转换。此外，`String` 和 `String` 不一样？正如预期的那样，没有方法可以将变量转换为布尔类型。从较大的类型转换为较小的类型是可能的，但可能会截断结果值。

# 如何检查对象的类型

经常需要检查对象在运行时是否为特定类型。在 Java 中，我们使用 `instanceof` 关键字；在 Kotlin 中，它是 `is` 关键字。

# 准备工作

您需要安装一个首选的开发环境，用于编译和运行 Kotlin。您也可以使用命令行，为此您需要安装 Kotlin 编译器和 JDK。我正在使用 [`try.kotlinlang.org/`](https://try.kotlinlang.org/) 上的在线 IDE 来编译和运行我的 Kotlin 代码，以完成这个菜谱。

# 如何做...

让我们看看如何按以下步骤检查对象的类型：

1.  让我们尝试一个非常基础的例子，尝试使用 `is` 与字符串和整数。在这个例子中，我们将检查一个字符串和一个整数的类型：

```java
fun main(args: Array<String>) {
    var a : Any = 1
    var b : Any = "1"
    if (a is String) {
        println("a = $a is String")
    }
    else {
        println("a = $a is not String")
    }
    if (b is String) {
        println("b = $b is String")
    }
    else {
        println("b = $b is not String")
    }
}

```

1.  同样，我们可以使用 `!is` 来检查对象是否不是 `String` 类型，如下所示：

```java
fun main(args: Array<String>) {
    var b : Any = 1
    if (b !is String) {
        println("$b is not String")
    }
    else {
        println("$b is String")
    }
}
```

如果你记得 Kotlin 中 `when` 的用法，我们就不需要使用 `is` 关键字，因为 Kotlin 有智能转换的功能，如果比较的对象不是同一类型，则会抛出错误。

# 它是如何工作的...

基本上，`is` 操作符用于在 Kotlin 中检查对象的类型，而 `!is` 是 `is` 操作符的否定。

Kotlin 编译器跟踪不可变值，并在需要时安全地进行转换。这就是智能转换的工作原理；`is` 是一个安全转换操作符，而不安全的转换操作符是 `as` 操作符。

# 还有更多...

让我们尝试一个使用 `as` 操作符的例子，它是 Kotlin 中的类型转换操作符。这是一个不安全的转换操作符。以下代码示例会抛出 `ClassCastException`，因为我们不能将整数转换为字符串：

```java
fun main(args: Array<String>) {
   var a : Any = 1
   var b = a as String
}
```

另一方面，以下代码由于变量 `a` 是 `Any` 类型，可以成功运行，因此可以被转换为 `String`：

```java
fun main(args: Array<String>) {
    var a : Any = "1"
    var b = a as String
    println(b.length)
}
```

# 如何在 Kotlin 中使用抽象类

抽象类是不能实例化的类，这意味着我们不能创建抽象类的对象。使用抽象类的主要灵感是我们可以从它们继承。当一个类从抽象类继承时，它实现了父类的所有抽象方法。

# 准备工作

您需要安装一个首选的开发环境，用于编译和运行 Kotlin。您也可以使用命令行，为此您需要安装 Kotlin 编译器和 JDK。我正在使用 [`try.kotlinlang.org/`](https://try.kotlinlang.org/) 上的在线 IDE 来编译和运行我的 Kotlin 代码，以完成这个菜谱。

# 如何做...

现在，让我们看看如何按以下步骤使用 `abstract` 类：

1.  `abstract` 关键字用于声明一个 `abstract` 类。让我们创建一个抽象类并尝试从它继承：

```java
abstract class Mammal {
    abstract fun move(direction: String)
}
```

1.  要使一个类成为 `Mammal` 类的子类，我们使用 `:` 操作符，如下例所示。请注意在超类方法实现之前使用的 `override` 关键字：

```java
class Dog : Mammal() {
    override fun move(direction: String) {
        println(direction)
    }
}
```

1.  如果我们不希望子类实现某个方法，我们不应将其声明为 `abstract` 或 `open`，如下例所示：

```java
fun main(args: Array<String>) {
    var x = Dog()
    x.move("North")
    println(x.show(123))
}
class Dog : Mammal() {
    override fun move(direction: String) {
        println(direction)
    }
}
abstract class Mammal {
    fun show(y: Int): String {
        return y.toString()
    }
    abstract fun move(direction: String)
}
```

1.  如果我们在每个类中声明 `init` 块，如下所示，我们将得到一个输出，其中超类的 `init` 块首先被调用：

```java
fun main(args: Array<String>) {
    var x = Dog()
    x.move("North")
    println(x.show(123))
}
class Dog : Mammal() {
    init {
        println ("Hey from Dog")
    }
    override fun move(direction: String) {
        println(direction)
    }
}
abstract class Mammal {
    init {
        println ("Hey from Mammal")
    }
    fun show(y: Int): String {
        return y.toString()
    }
    abstract fun move(direction: String)
}
```

最终程序的输出如下：

```java
Hey from Mammal
Hey from Dog
North
123
```

# 它是如何工作的...

`Dog` 类是 `Mammal` 类的子类，并继承其所有方法。声明为 `abstract` 的方法应由 `Dog` 类实现。`show()` 方法在 `Mammal` 类中，但可以通过 `Dog` 对象调用，因为创建的对象是 `Mammal` 类型。

超类的 `init` 块在子类之前被调用。

# 如何在 Kotlin 中遍历类的属性

Kotlin 中的反射允许我们在运行时对程序的结构进行自省。这也使我们能够自省类修饰符、方法和属性。

在这个菜谱中，我们将看到如何遍历 Kotlin 类的属性。那么，让我们开始吧！

# 准备工作

我们将使用 IntelliJ IDEA IDE 进行编码。我们将创建一个 `Student` 类，它将具有 `roll_number` 和 `name` 属性。然后我们将看到如何遍历其属性。

如果您不使用 IntelliJ IDE 或 Android Studio，您可能需要在类路径中包含反射库。前往 [`kotlinlang.org/docs/reference/reflection.html`](https://kotlinlang.org/docs/reference/reflection.html) 了解更多关于反射的信息。

# 如何做到这一点...

在以下步骤中，我们将看到如何遍历一个类的属性：

1.  这里是我们的 `Student` 类，具有 `roll_number` 和 `full_name` 属性：

```java
class Student constructor(var roll_number:Int, var full_name:String)
```

1.  现在，我们将使用 `for` 语句，因为我们想遍历一个类可以拥有的多个属性：

```java
fun main(args: Array<String>) {
    var student=Student(2013001,"Aanand Shekhar Roy")
    for (property in Student::class.memberProperties) {
        println("${property.name} = ${property.get(student)}")
    }
}
```

这是输出结果：

```java
full_name = Aanand Shekhar Roy
roll_number = 2013001
```

# 它是如何工作的...

实现相当简单。我们能够实现对类属性的检查，因为我们使用了反射，而 `memberProperties` 只是 `KClass` 的许多函数之一。

需要注意的一点是，`memberProperties` 返回此类及其所有超类中声明的所有非扩展属性。假设我们有一个 `Person` 类，如下所示：

```java
open class Person{
     val isHuman:Boolean=true
}
```

此外，我们通过 `Person` 类扩展我们的 `Student` 类，然后使用 `memberProperties` 方法之前相同的代码将产生如下输出：

```java
full_name = Aanand Shekhar Roy
roll_number = 2013001
isHuman = true
```

因此，如果您只想遍历 `Student` 类中声明的字段，您将需要使用 `declaredMemberProperties` 方法。以下是一个使用 `declaredMemberProperties` 的示例：

```java
for (property in Student::class.declaredMemberProperties) {
    println("${property.name} = ${property.get(student)}")
}
```

这是输出结果：

```java
full_name = Aanand Shekhar Roy
roll_number = 2013001
```

前面的例子是关于 Kotlin 的`KClass`。假设你想遍历`Java Class<T>`的属性——你可以使用 Kotlin 扩展属性来获取 Kotlin 的`KClass<T>`，然后你可以继续操作，例如`something.javaClass.kotlin.memberProperties`。

# 还有更多...

查看 Kotlin 反射库提供的方法列表（[`kotlinlang.org/api/latest/jvm/stdlib/kotlin.reflect/-k-class/index.html`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.reflect/-k-class/index.html)），借助它你可以在运行时执行许多内省操作。

# 如何处理内联属性

Kotlin 的一个优点是高阶函数，它允许我们将函数作为其他函数的参数使用。然而，它们是对象，因此它们会带来内存开销（因为每个实例都会在堆中分配空间，我们还需要方法来调用函数）。我们可以通过使用内联函数来改善这种情况。内联注解意味着特定的函数以及函数参数将在调用位置展开；这有助于减少调用开销。

类似地，内联关键字可以与没有后置字段的属性和属性访问器一起使用。让我们看看在这个菜谱中是如何做到这一点的。

# 准备工作

你需要安装一个首选的开发环境，用于编译和运行 Kotlin。你也可以使用命令行，这需要安装 Kotlin 编译器和 JDK。我正在使用[`try.kotlinlang.org/`](https://try.kotlinlang.org/)在线 IDE 来编译和运行我的 Kotlin 代码，以完成这个菜谱。你也可以使用 IntelliJ IDEA 作为开发环境。

# 如何操作...

让我们看看如何在这些步骤中处理内联属性：

1.  让我们尝试一个例子，在 Kotlin 中内联一个属性的访问器：

```java
var x.valueIsMaxedOut: Boolean
inline get() = x.value == CONST_MAX
```

1.  在这个例子中，我们只是使用了`inline`关键字与`get`访问器。我们也可以通过使整个属性内联来声明`get`和`set`访问器为内联，如这个代码片段所示：

```java
inline var x.valueIsMaxedOut: Boolean
get() = x.value == CONST_MAX
set(value) {
    // set field here
    println(“Value set!”)
}
```

在前面的代码片段中，两个访问器都被内联了。

1.  然而，需要注意的是，如果属性有一个后置字段或访问器不引用后置字段，内联就不会与属性或访问器一起工作。这里的代码是一个我们无法使用`inline`的场景的例子：

```java
var x.valueIsMaxedOut: Boolean = true
get() = x.value == CONST_MAX
set(value) {
    // set field here
    println(“Value set!”)
}
```

另一点需要记住的是，尽管内联属性通过仅在调用位置展开来减少调用开销，但它们也会增加整体字节码，因此内联不应与大型函数或访问器一起使用。

# 它是如何工作的...

因此，基本上，当我们希望减少内存开销时，我们会使用内联。就像内联函数一样，我们也可以将属性声明为内联或属性的访问器作为内联。然而，需要注意的是，内联会增加字节码的量，因此建议不要内联具有大量代码逻辑的函数或访问器。

# 如何处理嵌套类

在这个菜谱中，我们将展示如何在 Kotlin 中使用嵌套类。嵌套类是其封装类的成员。

# 准备工作

您需要安装一个首选的开发环境，该环境可以编译和运行 Kotlin。您也可以使用命令行，为此您需要安装 Kotlin 编译器和 JDK。我正在使用在线 IDE [`try.kotlinlang.org/`](https://try.kotlinlang.org/) 来编译和运行我的 Kotlin 代码，以完成这个菜谱。

# 如何操作...

现在我们将分步骤展示如何使用嵌套类：

1.  让我们尝试一个 Kotlin 嵌套类的例子：

```java
fun main(args: Array<String>) {
    var a1 = outCl()
    a1.printAB()
    outCl.inCl().printB()
}
class outCl {
var a = 6
    fun printAB () {
    var b_ = inCl().b
    println ("a = $a and b = $b_ from inside outCl")
}

class inCl {
    var b = "9"
        fun printB() {
            println ("b = $b from inside inCl")
        }
    }
}

```

这是输出：

```java
a = 6 and b = 9 from inside outCl
b = 9 from inside inCl
```

1.  现在，让我们尝试一个 `inner` 类的例子。要将嵌套类声明为 `inner`，我们使用 `inner` 关键字。`inner` 类可以访问外部类的成员，因为它们携带指向外部类的引用：

```java
fun main(args: Array<String>) {
    var a = outCl()
    a.printAB()
    a.inCl().printAB()
}
class outCl {
    var a = 6
    fun printAB () {
        var b_ = inCl().b
        println ("a = $a and b = $b_ from inside outCl")
    }
    inner class inCl {
        var b = "9"
        fun printAB() {
            println ("a = $a and b = $b from inside inCl")
        }
    }
}
```

上述代码的输出如下：

```java
a = 6 and b = 9 from inside outCl
a = 6 and b = 9 from inside inCl
```

# 它是如何工作的...

嵌套类可以通过在另一个类内部声明嵌套类来创建。在这种情况下，要访问嵌套类，您创建一个静态引用，类似于 `outerClass.innerClass()`，您还可以使用此方法创建内部类的对象。

另一方面，`inner` 类是通过在嵌套类中添加 `inner` 关键字创建的。在这种情况下，我们像访问外部类的成员一样访问内部类，即使用外部类的对象，如下所示：

```java
var outerClassObject = outerClass()
outerClassObject.innerClass().memberVar
```

嵌套类无法访问外部类的成员，因为它没有外部类对象的任何引用。另一方面，内部类可以访问外部类的所有成员，因为它有一个指向外部类对象的引用。

# 还有更多...

我们也可以在 Kotlin 中使用 `object` 关键字创建匿名内部类，如下所示：

```java
val customTextTemplateListener = object:ValueEventListener{
    override fun onCancelled(p0: DatabaseError?) {
    }
    override fun onDataChange(dataSnapshot: DataSnapshot?) {
    }
}
```

# 在 Kotlin 中获取类

在这个菜谱中，我们将探讨在 Kotlin 中获取类引用的方法。主要，我们将使用反射。反射是一个库，它提供了在运行时而不是编译时检查代码的能力。在 Java 中，我们可以通过 `getClass()` 获取变量的类，例如 `something.getClass()`。让我们看看如何在 Kotlin 中解析变量的类。

# 如何操作...

1.  Java 中解析变量名称的等效方法是使用 `.getClass()` 方法，例如，`something.getClass()`。在 Kotlin 中，我们可以通过 `something.javaClass` 实现相同的功能。

1.  要获取反射类的引用，我们以前在 Java 中使用 `something.class`，其 Kotlin 等价物是 `something::class`。这返回一个 `KClass`。这个 `KClass` 的特殊之处在于它提供了与 Java 反射类提供的功能相当的自省能力。

    注意，KClass 与 Java 的 `Class` 对象不同。如果您想从 Kotlin 的 `KClass` 获取 Java 的 `Class` 对象，请使用 `.java` 扩展属性：

```java
val somethingKClass: KClass<Something> = Something::class
val a: Class<Something> = somethingKClass.java
val b: Class<Something> = Something::class.java
```

1.  后者示例将被编译器优化，以避免分配中间的 KClass 实例。

    如果你使用 Kotlin 1.0，你可以通过调用 `.kotlin` 扩展属性将获得的 Java 类转换为 KClass 实例，例如，`something.javaClass.kotlin`。

# 还有更多...

正如刚才所描述的，`KClass` 为你提供了反射能力。以下是一些 `KClass` 的方法：

+   `isAbstract`：如果此类是抽象的则为真

+   `isCompanion`：如果此类是伴随对象则为真

+   `isData`：如果此类是数据类则为真

+   `isFinal`：如果此类是最终类则为真

+   `isInner`：如果此类是内部类则为真

+   `isOpen`：如果此类是公开的则为真

查阅此链接以获取 KClass 提供的完整函数列表（[`kotlinlang.org/api/latest/jvm/stdlib/kotlin.reflect/-k-class/`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.reflect/-k-class/))。

# 使用代理属性进行操作

Kotlin 1.1 带来了许多更新；其中之一是代理属性。有三种类型的代理属性：

+   `lazy`：懒加载属性是首先评估的，并在之后返回相同的实例，就像缓存一样

+   `observable`：每当发生更改时都会通知监听器

+   `map`：属性存储在映射中，而不是每个字段中

在这个菜谱中，我们将看到如何使用这些代理。那么，让我们开始吧。

# 准备中

我们将处理 Android 代码，因此我们需要 Android Studio 3。

# 如何做到这一点...

让我们看看代理属性的一个简单例子：

1.  首先，我们将处理懒加载代理属性。简单来说，这个代理可以延迟对象的创建，直到我们第一次访问它。当你处理重量级对象时，这非常重要；它们需要很长时间才能创建——例如，创建数据库实例或可能是 dagger 组件。不仅如此，结果会被记住，并且对于此类代理属性的后续 `getValue()` 调用，将返回相同的值。让我们看一个例子：

```java
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
     val button by lazy { findViewById<Button>(R.id.submit_button) }                                              setContentView(R.layout.activity_main)
     button.text="Submit"
}
```

1.  上述是活动的标准 `onCreate` 方法。如果你仔细观察，我们在 `setContentView(..)` 方法之前设置了 `button` 变量。当你运行它时，它运行得很好。如果你没有使用懒加载，它将给出一个 `NullPointerException`，类似于这样：

```java
Caused by: java.lang.NullPointerException: Attempt to invoke virtual method 'void android.widget.Button.setText(java.lang.CharSequence)' on a null object reference
```

1.  按钮变量为空，因为我们是在 `setContentView` 之前调用它的。然而，这对于懒创建的 `button` 对象来说并不是问题，因为尽管我们在 `setContentView` 之前声明了它，但 `button` 对象是在第一次访问后创建的，即当我们尝试在它上设置属性时。

1.  因此，使用懒加载构造，你不需要考虑初始化代码放置的位置，对象的初始化将延迟到第一次使用时。

另一个需要注意的关键点是，默认情况下，懒加载属性的评估将是同步的，这意味着值在一个线程中计算，其余的线程将看到相同的值。有三种初始化类型：

+   `LazyThreadSafetyMode.SYNCHRONIZED`：这是默认模式，确保只有一个线程可以初始化实例。

+   `LazyThreadSafetyMode.PUBLICATION`：在此模式下，多个线程可以执行初始化。

+   `LazyThreadSafetyMode.NONE`：当我们确定初始化只会在一个线程上发生时，使用此模式。例如，在 Android 的案例中，我们可以确定视图将由 UI 线程初始化。由于这并不保证线程安全，它具有更少的开销。

另一个有用的委托是可观察的委托。这个委托帮助我们观察属性的任何更改。例如，让我们看看 `observable` 委托的一个非常基本的实现：

```java
fun main(args: Array<String>) {
    val paris=Travel()
     paris.placeName="Paris"
     paris.placeName="Italy"
}
class Travel {
     var placeName:String by Delegates.observable("<>"){
         property, oldValue, newValue ->
         println("oldValue = $oldValue, newValue = $newValue")
    }
}
```

这是输出：

```java
oldValue = <>, newValue = Paris
oldValue = Paris, newValue = Italy
```

如我们所见，`observable` 委托接受两件事：一个默认值（我们指定为 `<>`）和一个处理程序，该处理程序在属性修改时被调用。

让我们现在处理 `vetoable` 委托。它与 `observable` 委托非常相似，但使用它可以“否决”修改。让我们看看一个例子：

```java
fun main(args: Array<String>) {
    val paris=Travel()
    paris.placeName="Paris"
    paris.placeName="Italy"
    println(paris.placeName)
}
class Travel {
    var placeName:String by Delegates.vetoable("<>"){
        property, oldValue, newValue ->
            if(!newValue.equals("Paris")){
                return@vetoable false
            }
            true
    }
}
```

这是输出：

```java
Paris
```

如前例所示，如果 `newValue` 不等于 `"Paris"`，我们将返回 `false`，并且修改将被中止。如果您想进行修改，您需要从构造函数中返回 `true`。

有时，您会根据动态值创建一个对象，例如，在解析 JSON 的情况下。对于这些应用程序，我们可以使用 `map` 实例本身作为委托属性的委托。让我们在这里看看一个例子：

```java
fun main(args: Array<String>) {
    val paris=Travel(mapOf(
        "placeName" to "Paris"
    ))
    println(paris.placeName)
}
class Travel(val map:Map<String,Any?>) {
    val placeName: String by map
}
```

这是输出：

```java
Paris
```

要使其适用于 `var` 属性，您需要使用 `MutableMap`，因此前面的示例可能看起来像这样：

```java
fun main(args: Array<String>) {
    val paris=Travel(mutableMapOf(
        "placeName" to "Paris"
    ))
    println(paris.placeName)
}
class Travel(val map:MutableMap<String,Any?>) {
    var placeName: String by map
}
```

当然，输出将是相同的。

# 更多...

可观察的委托属性在适配器中可以广泛使用。适配器用于在某种列表中填充数据。通常，当数据更新时，我们只需更新适配器中的成员变量列表，然后调用 `notifyDatasetChanged()`。借助可观察的和 `DiffUtils`，我们只需更新实际更改的内容，而不是更改一切。这导致性能更加高效。

# 与枚举一起工作

枚举用于变量只能取一小组可能值的情况。一个例子是类型常量（方向："North"、"South"、"East" 和 "West"）。借助枚举，您可以避免传递无效常量的错误，并且还可以记录哪些值是可接受的。

在这个菜谱中，我们将了解如何在 Kotlin 中使用枚举。

# 准备工作

我们将使用 IntelliJ IDEA 来编写和运行代码。首先，我们将创建一个简单的类型安全的枚举 `Direction`，其成员包括 NORTH、SOUTH、EAST 和 WEST（代表四个方向）。

# 如何做到这一点...

让我们看看 `enum` 类的一个例子：

1.  在这个例子中，我们将创建一个方向枚举。我们将假设只有四个方向：

```java
enum class Direction {
    NORTH,SOUTH,EAST,WEST
}
fun main(args: Array<String>) {
    var north_direction=Direction.NORTH
    if(north_direction==Direction.NORTH){
        println("Going North")
    }else{
        println("No idea where you're going!")
    }
}
```

1.  正如你所见，变量（`north_direction`）只需在`enum`类中预定义的常量中取值。

1.  我们还可以使用默认值来初始化枚举：

```java
enum class Direction(var value:Int) {
    NORTH(1),SOUTH(2),EAST(3),WEST(4)
}
fun main(args: Array<String>) {
    var north_direction=1
    if(north_direction==Direction.NORTH.value){
        println("Going North")
    }else{
        println("No idea where you're going!")
    }
}

//Output: Going North
```

# 还有更多...

强烈建议你不在你的 Android 项目中使用枚举。根据谷歌工程师的说法，添加一个枚举将使最终 DEX 文件的大小增加大约 13 倍。它还会产生运行时开销的问题，你的应用将需要更多的空间。

Android 文档中是这样说的：

"枚举通常需要的内存是静态常量的两倍以上。你应该严格避免在 Android 中使用枚举。"

然而，如果你想享受枚举的便利性，你可以使用 Android 的注解库，它有`TypeDef`注解——但是遗憾的是，在本书编写时，Kotlin 不支持这一点，所以我们希望它能在 Kotlin 的未来版本中得到添加。
