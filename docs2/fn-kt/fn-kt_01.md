# 第一章：Kotlin – 数据类型、对象和类

在本章中，我们将介绍 Kotlin 的类型系统、**面向对象编程**（OOP）与 Kotlin、修饰符、解构声明等。

Kotlin 主要是一种面向对象（OOP）语言，同时具有一些函数式特性。当我们使用面向对象的语言来解决问题时，我们试图以与问题相关的信息以抽象的方式模拟问题中的一部分对象。

如果我们为公司设计一个 HR 模块，我们将用状态或数据（姓名、出生日期、社会保险号等）和行为（支付薪水、调到另一个部门等）来模拟员工。因为一个人可能非常复杂，有些信息对于我们的问题或领域并不相关。例如，员工的自行车喜好风格对于我们的 HR 系统并不相关，但对于在线自行车店来说却非常相关。

一旦我们确定了对象（包括数据和行为）以及它们与我们领域其他对象的关系，我们就可以开始开发和编写代码，这些代码将成为我们软件解决方案的一部分。我们将使用语言结构（结构是一种说法，指的是允许的语法）来编写对象、类别、关系等等。

Kotlin 有许多我们可以用来编写程序的构造，在本章中，我们将介绍其中许多构造，例如：

+   类

+   继承

+   抽象类

+   接口

+   对象

+   泛型

+   类型别名

+   空类型

+   Kotlin 的类型系统

+   其他类型

# 类

**类**是 Kotlin 中的基础类型。在 Kotlin 中，类是一个模板，它为实例提供状态、行为和类型（稍后会有更多介绍）。

定义一个类只需要一个名称：

```kt
class VeryBasic
```

`VeryBasic`不是非常有用，但仍然是一个有效的 Kotlin 语法。

`VeryBasic`类没有任何状态或行为；尽管如此，你仍然可以声明`VeryBasic`类型的值，如下面的代码所示：

```kt
fun main(args: Array<String>) {
    val basic: VeryBasic = VeryBasic()
}
```

正如你所见，`basic`值具有`VeryBasic`类型。用不同的方式表达，`basic`是`VeryBasic`的一个实例。

在 Kotlin 中，类型可以被推断；因此，前面的例子等同于以下代码：

```kt
fun main(args: Array<String>) {
    val basic = VeryBasic()
}
```

由于`basic`是一个`VeryBasic`实例，它具有`VeryBasic`类型的副本状态和行为，即没有。真是太遗憾了。

# 属性

如前所述，类可以有状态。在 Kotlin 中，类状态由**属性**表示。让我们看看蓝莓纸杯蛋糕的例子：

```kt
class BlueberryCupcake {
  var flavour = "Blueberry"
}
```

`BlueberryCupcake`类有一个`has-a`属性`flavour`，其类型为`String`。

当然，我们可以有`BlueberryCupcake`类的实例：

```kt
fun main(args: Array<String>) {
    val myCupcake = BlueberryCupcake()
    println("My cupcake has ${myCupcake.flavour}")
}
```

现在，因为我们声明了`flavour`属性为变量，它的内部值可以在运行时被改变：

```kt
fun main(args: Array<String>) {
    val myCupcake = BlueberryCupcake()
    myCupcake.flavour = "Almond"
    println("My cupcake has ${myCupcake.flavour}")
}
```

在现实生活中这是不可能的。纸杯蛋糕不会改变它们的味道（除非它们变陈了）。如果我们将`flavour`属性更改为一个值，它就不能被修改：

```kt
class BlueberryCupcake {
    val flavour = "Blueberry"
}

fun main(args: Array<String>) {
    val myCupcake = BlueberryCupcake()
    myCupcake.flavour = "Almond" //Compilation error: Val cannot be reassigned
    println("My cupcake has ${myCupcake.flavour}")
}
```

让我们声明一个新的类来表示杏仁纸杯蛋糕：

```kt
class AlmondCupcake {
    val flavour = "Almond"
}

fun main(args: Array<String>) {
    val mySecondCupcake = AlmondCupcake()
    println("My second cupcake has ${mySecondCupcake.flavour} flavour")
}
```

这里有些奇怪。`BlueberryCupcake`和`AlmondCupcake`在结构上相同；只是内部值发生了变化。

在现实生活中，你不需要为不同的纸杯蛋糕风味准备不同的烤盘。同一个高质量的烤盘可以用于各种风味。同样，一个设计良好的`Cupcake`类可以用于不同的实例：

```kt
class Cupcake(flavour: String) { 
  val flavour = flavour
}
```

`Cupcake`类有一个带有参数`flavour`的构造函数，该参数被分配给`flavour`值。

因为这是一个非常常见的习语，Kotlin 有一些语法糖来更简洁地定义它：

```kt
class Cupcake(val flavour: String)
```

现在，我们可以定义具有不同风味的几个实例：

```kt
fun main(args: Array<String>) {
    val myBlueberryCupcake = Cupcake("Blueberry")
    val myAlmondCupcake = Cupcake("Almond")
    val myCheeseCupcake = Cupcake("Cheese")
    val myCaramelCupcake = Cupcake("Caramel")
}
```

# 方法

在 Kotlin 中，一个类的行为由方法定义。技术上，**方法**是一个成员函数，因此，我们在以下章节中学到的关于函数的知识也适用于方法：

```kt
class Cupcake(val flavour: String) {
  fun eat(): String {
    return "nom, nom, nom... delicious $flavour cupcake"
  }
}
```

`eat()`方法返回一个`String`值。现在，让我们调用`eat()`方法，如下面的代码所示：

```kt
fun main(args: Array<String>) {
    val myBlueberryCupcake = Cupcake("Blueberry")
    println(myBlueberryCupcake.eat())
}
```

以下表达式是前面代码的输出：

![](img/bb269540-7a2c-44fa-806b-ddf52f7597cf.png)

没有什么令人震惊的，但这是我们第一个方法。稍后，我们会做更多有趣的事情。

# 继承

随着我们继续在 Kotlin 中建模我们的领域，我们发现具体对象相当相似。如果我们回到我们的人力资源示例，员工和承包商相当相似；他们都有名字、出生日期等；他们也有一些差异。例如，承包商有日工资，而员工有月薪。很明显，他们是相似的——他们都是人；人是包含承包商和员工的超集。因此，他们都有自己独特的特征，使他们足够不同，可以分类到不同的子集中。

这就是继承的全部内容，有组和子组，它们之间有联系。在继承层次结构中，如果你向上移动层次结构，你会看到更多通用特性和行为，如果你向下移动，你会看到更具体的行为。玉米卷和微处理器都是对象，但它们有不同的用途和用途。

让我们引入一个新的`Biscuit`类：

```kt
class Biscuit(val flavour: String) { 
  fun eat(): String { 
    return "nom, nom, nom... delicious $flavour biscuit" 
  } 
}
```

再次，这个类看起来几乎与`Cupcake`完全相同。我们可以重构这些类以减少代码重复：

```kt
open class BakeryGood(val flavour: String) { 
  fun eat(): String { 
    return "nom, nom, nom... delicious $flavour bakery good" 
  } 
} 

class Cupcake(flavour: String): BakeryGood(flavour) 
class Biscuit(flavour: String): BakeryGood(flavour)
```

我们引入了一个新的`BakeryGood`类，它具有`Cupcake`和`Biscuit`类的共享行为和状态，并且我们让这两个类都扩展了`BakeryGood`。通过这样做，`Cupcake`（和`Biscuit`）现在与`BakeryGood`有了一个*是*关系；另一方面，`BakeryGood`是`Cupcake`类的超类或父类。

注意`BakeryGood`被标记为`open`。这意味着我们特别设计了这个类以便扩展。在 Kotlin 中，你不能扩展一个不是`open`的类。

将常见的行为和状态移动到父类的过程称为**泛化**。让我们看一下下面的代码：

```kt
fun main(args: Array<String>) {
    val myBlueberryCupcake: BakeryGood = Cupcake("Blueberry")
    println(myBlueberryCupcake.eat())
}
```

让我们尝试我们的新代码：

![图片](img/068015b9-405f-4a9d-b1a6-77986d9fbfd4.png)

唉，这不是我们预期的。我们需要进一步折射它：

```kt
open class BakeryGood(val flavour: String) { 
  fun eat(): String { 
    return "nom, nom, nom... delicious $flavour ${name()}" 
  } 

  open fun name(): String { 
    return "bakery good" 
  } 
} 

class Cupcake(flavour: String): BakeryGood(flavour) { 
  override fun name(): String { 
    return "cupcake" 
  } 
} 

class Biscuit(flavour: String): BakeryGood(flavour) { 
  override fun name(): String { 
    return "biscuit" 
  } 
}
```

它工作了！让我们看看输出结果：

![图片](img/b658a569-6f48-4c67-8ddb-d491338b4481.png)

我们声明了一个新的方法，`name()`；它应该被标记为 `open`，因为我们设计它可以在其子类中可选地修改。

在子类中修改方法定义称为 **重写**，这就是为什么两个子类中的 `name()` 方法都被标记为 `override`。

在层次结构中扩展类和重写行为的过程称为 **特殊化**。

经验法则

将一般状态和行为放在层次结构的顶部（泛化），将特定状态和行为放在子类中（特殊化）。

现在，我们可以有更多的面包店商品了！让我们看看下面的代码：

```kt
open class Roll(flavour: String): BakeryGood(flavour) { 
  override fun name(): String { 
    return "roll" 
  } 
} 

class CinnamonRoll: Roll("Cinnamon")
```

子类也可以被扩展。它们只需要被标记为 `open`：

```kt
open class Donut(flavour: String, val topping: String) : BakeryGood(flavour)
{
    override fun name(): String {
        return "donut with $topping topping"
    }
}

fun main(args: Array<String>) {
    val myDonut = Donut("Custard", "Powdered sugar")
    println(myDonut.eat())
}
```

我们还可以创建具有更多属性和方法的方法。

# 抽象类

到目前为止，一切顺利。我们的面包店看起来不错。然而，我们当前模型有一个问题。让我们看看下面的代码：

```kt
fun main(args: Array<String>) {
    val anyGood = BakeryGood("Generic flavour")
}
```

我们可以直接实例化 `BakeryGood` 类，这太通用。为了纠正这种情况，我们可以将 `BakeryGood` 标记为 `abstract`：

```kt
abstract class BakeryGood(val flavour: String) { 
  fun eat(): String { 
    return "nom, nom, nom... delicious $flavour ${name()}" 
  }

  open fun name(): String { 
    return "bakery good" 
  } 
}
```

**抽象类** 是一个专为扩展而设计的类。抽象类不能被实例化，这解决了我们的问题。

什么是 `abstract` 与 `open` 的不同之处？

这两个修饰符都允许我们扩展一个类，但 `open` 允许我们实例化，而 `abstract` 不允许。

现在我们不能实例化，`BakeryGood` 类中的 `name()` 方法就不再那么有用，而且除了 `CinnamonRoll` 之外的所有子类都重写了它（`CinnamonRoll` 依赖于 `Roll` 的实现）：

```kt
abstract class BakeryGood(val flavour: String) { 
  fun eat(): String { 
    return "nom, nom, nom... delicious $flavour ${name()}" 
  } 

  abstract fun name(): String 
}
```

被标记为 `abstract` 的方法没有主体，只有签名声明（方法签名是识别方法的一种方式）。在 Kotlin 中，签名由方法名、参数数量、参数类型和返回类型组成。

任何直接扩展 `BakeryGood` 的类都必须重写 `name()` 方法。重写抽象方法的术语是 **实现**，从现在起，我们将使用它。所以，`Cupcake` 类实现了 `name()` 方法（Kotlin 没有用于方法实现的关键字；方法实现和方法重写都使用关键字 `override`）。

让我们引入一个新的类，`Customer`；面包店总是需要顾客的：

```kt
class Customer(val name: String) {
  fun eats(food: BakeryGood) {
    println("$name is eating... ${food.eat()}")
  }
}

fun main(args: Array<String>) {
    val myDonut = Donut("Custard", "Powdered sugar")
    val mario = Customer("Mario")
    mario.eats(myDonut)
}
```

`eats(food: BakeryGood)` 方法接受一个 `BakeryGood` 参数，因此任何扩展 `BakeryGood` 参数的类的实例，无论有多少层等级。只需记住，我们可以直接实例化 `BakeryGood`。

如果我们想要一个简单的 `BakeryGood` 呢？例如，进行测试。

有一个替代方案，一个匿名子类：

```kt
fun main(args: Array<String>) {
    val mario = Customer("Mario")

    mario.eats(object : BakeryGood("TEST_1") {
        override fun name(): String {
            return "TEST_2"
        }
    })
}
```

这里引入了一个新关键字，`object`。稍后我们将更详细地介绍 `object`，但就目前而言，只需知道这是一个 **对象表达式**。对象表达式定义了一个匿名类的实例，该类扩展了一个类型。

在我们的例子中，对象表达式（技术上，是 **匿名类**）必须重写 `name()` 方法，并将值作为参数传递给 `BakeryGood` 构造函数，就像标准类一样。

记住，一个 `object` 表达式是一个实例，因此它可以用来声明值：

```kt
val food: BakeryGood = object : BakeryGood("TEST_1") { 
  override fun name(): String { 
    return "TEST_2" 
  } 
} 

mario.eats(food)
```

# 接口

公开和抽象类非常适合创建层次结构，但有时它们不够用。一些子集可能跨越看似无关的层次结构，例如，鸟类和大型灵长类动物都是两足动物，它们都是动物和脊椎动物，但它们并不直接相关。这就是为什么我们需要不同的结构，Kotlin 给我们提供了接口（其他语言以不同的方式处理这个问题）。

我们的面包店产品很棒，但我们需要先烹饪它们：

```kt
abstract class BakeryGood(val flavour: String) { 
  fun eat(): String { 
    return "nom, nom, nom... delicious $flavour ${name()}" 
  } 

  fun bake(): String { 
    return "is hot here, isn't??" 
  } 

  abstract fun name(): String 
}
```

我们的新的 `bake()` 方法将烹饪我们所有惊人的产品，但是等等，甜甜圈不是烘焙的，而是油炸的。

如果我们能把 `bake()` 方法移动到第二个抽象类 `Bakeable` 中会怎样？让我们在下面的代码中尝试一下：

```kt
abstract class Bakeable { 
  fun bake(): String { 
    return "is hot here, isn't??" 
  } 
} 

class Cupcake(flavour: String) : BakeryGood(flavour), Bakeable() { //Compilation error: Only one class may appear in a supertype list 
  override fun name(): String { 
    return "cupcake" 
  } 
}
```

错误！在 Kotlin 中，一个类不能同时扩展两个类。让我们看看下面的代码：

```kt
interface Bakeable { 
  fun bake(): String { 
    return "is hot here, isn't??" 
  } 
} 

class Cupcake(flavour: String) : BakeryGood(flavour), Bakeable { 
  override fun name(): String { 
    return "cupcake" 
  } 
}
```

然而，它可以扩展多个接口。一个 **接口** 是一个定义行为的类型；在 `Bakeable` 接口的情况下，那就是 `bake()` 方法。

因此，公开/抽象类和接口之间的区别是什么？

让我们从以下相似之处开始：

+   它们都是类型。在我们的例子中，`Cupcake` 与 `BakeryGood` 有一个 *is-a* 关系，并且与 `Bakeable` 也有一个 *is-a* 关系。

+   它们都通过方法定义行为。

+   虽然公开类可以直接实例化，但抽象类和接口不行。

现在，让我们看看以下的不同之处：

+   一个类可以扩展一个类（公开或抽象），但不能扩展多个接口。

+   公开/抽象类可以有构造函数。

+   公开/抽象类可以初始化自己的值。接口的值必须在扩展接口的类中初始化。

+   公开类必须声明可以重写的公开方法。抽象类可以同时有公开和抽象方法。

在接口中，所有方法都是公开的，没有实现的方法不需要抽象修饰符：

```kt
interface Fried { 
  fun fry(): String 
} 

open class Donut(flavour: String, val topping: String) : BakeryGood(flavour), Fried { 
  override fun fry(): String { 
    return "*swimming on oil*" 
  } 

  override fun name(): String { 
    return "donut with $topping topping" 
  } 
}
```

何时使用其中一个或另一个？：

+   当使用公开/抽象类时：

    +   这个类应该被扩展和实例化

+   当需要应用多重继承时使用抽象类：

    +   这个类不能被实例化

    +   需要一个构造函数

    +   存在初始化逻辑（使用 `init` 块）

让我们看看以下代码：

```kt
abstract class BakeryGood(val flavour: String) {
  init { 
    println("Preparing a new bakery good") 
  } 

  fun eat(): String { 
    return "nom, nom, nom... delicious $flavour ${name()}" 
  } 

  abstract fun name(): String 
}
```

+   当需要使用接口时：

    +   使用公开类：

    +   不需要初始化逻辑

我的建议是，你应该始终从接口开始。接口更直接、更简洁；它们还允许更模块化的设计。如果需要数据初始化/构造函数，则可以转向抽象/开放的。

与抽象类一样，对象表达式可以与接口一起使用：

```kt
val somethingFried = object : Fried { 
  override fun fry(): String { 
    return "TEST_3" 
  } 
}
```

# 对象

我们已经介绍了对象表达式，但关于对象还有更多内容。**对象**是自然的单例（这里的“自然”是指作为语言特性出现，而不是作为行为模式实现，如其他语言中的情况）。单例是一种只有一个实例的类型，并且 Kotlin 中的每个对象都是单例。这打开了许多有趣的模式（以及一些不良实践）。作为单例的对象对于协调系统中的操作很有用，但如果不小心使用来保持全局状态，也可能很危险。

对象表达式不需要扩展任何类型：

```kt
fun main(args: Array<String>) {
    val expression = object {
        val property = ""

        fun method(): Int {
            println("from an object expressions")
            return 42
        }
    }

    val i = "${expression.method()} ${expression.property}"
    println(i)
}
```

在这种情况下，`expression` 值是一个没有特定类型的对象。我们可以访问其属性和函数。

有一个限制——没有类型的对象表达式只能在本地使用，即在方法内部，或者私有地，在类内部：

```kt
class Outer {
    val internal = object {
        val property = ""
    }
}

fun main(args: Array<String>) {
    val outer = Outer()

    println(outer.internal.property) // Compilation error: Unresolved reference: property
}
```

在这种情况下，`property` 值无法访问。

# 对象声明

对象也可以有一个名称。这种对象被称为 **对象声明**：

```kt
object Oven {
  fun process(product: Bakeable) {
    println(product.bake())
  }
}

fun main(args: Array<String>) {
    val myAlmondCupcake = Cupcake("Almond")
    Oven.process(myAlmondCupcake)
}
```

对象是单例；你不需要实例化 `Oven` 来使用它。对象还可以扩展其他类型：

```kt
interface Oven {
  fun process(product: Bakeable)
}

object ElectricOven: Oven {
  override fun process(product: Bakeable) {
    println(product.bake())
  }
}

fun main(args: Array<String>) {
    val myAlmondCupcake = Cupcake("Almond")
    ElectricOven.process(myAlmondCupcake)
}
```

# 伴随对象

在类/接口内部声明的对象可以被标记为伴随对象。观察以下代码中伴随对象的使用：

```kt
class Cupcake(flavour: String) : BakeryGood(flavour), Bakeable {
  override fun name(): String { 
    return "cupcake" 
  } 

  companion object { 
    fun almond(): Cupcake { 
      return Cupcake("almond") 
    } 

    fun cheese(): Cupcake { 
      return Cupcake("cheese") 
    } 
  } 
}
```

现在，可以直接使用类名来使用伴随对象中的方法，而不需要实例化它：

```kt
fun main(args: Array<String>) {
    val myBlueberryCupcake: BakeryGood = Cupcake("Blueberry")
    val myAlmondCupcake = Cupcake.almond()
    val myCheeseCupcake = Cupcake.cheese()
    val myCaramelCupcake = Cupcake("Caramel")
}
```

伴随对象的方法不能从实例中使用：

```kt
fun main(args: Array<String>) {
    val myAlmondCupcake = Cupcake.almond()
    val myCheeseCupcake = myAlmondCupcake.cheese() //Compilation error: Unresolved reference: cheese
}
```

伴随对象可以作为具有名称 `Companion` 的值在类外部使用：

```kt
fun main(args: Array<String>) {
    val factory: Cupcake.Companion = Cupcake.Companion
}
```

或者，`Companion` 对象也可以有一个名称：

```kt
class Cupcake(flavour: String) : BakeryGood(flavour), Bakeable {
    override fun name(): String {
        return "cupcake"
    }

    companion object Factory {
        fun almond(): Cupcake {
            return Cupcake("almond")
        }

        fun cheese(): Cupcake {
            return Cupcake("cheese")
        }
    }
}

fun main(args: Array<String>) {
    val factory: Cupcake.Factory = Cupcake.Factory
}
```

它们也可以不命名使用，如下面的代码所示：

```kt
fun main(args: Array<String>) {
    val factory: Cupcake.Factory = Cupcake
}
```

不要被这种语法弄混淆。没有括号的 `Cupcake` 值是伴随对象；`Cupcake()` 是一个实例。

# 泛型

这一部分只是泛型的一个简要介绍；稍后我们将详细讨论。

**泛型编程**是一种编程风格，它侧重于创建适用于一般问题的算法（以及相关数据结构）。

Kotlin 支持泛型编程的方式是使用类型参数。简而言之，我们用类型参数编写代码，然后在需要使用时，将这些类型作为参数传递。

以我们的 `Oven` 接口为例：

```kt
interface Oven {
  fun process(product: Bakeable)
}
```

烤箱是一种机器，因此我们可以更广泛地推广它：

```kt
interface Machine<T> {
  fun process(product: T)
}
```

`Machine<T>` 接口定义了一个类型参数 `T` 和一个 `process(T)` 方法。

现在，我们可以用 `Oven` 来扩展它：

```kt
interface Oven: Machine<Bakeable>
```

现在，`Oven` 使用 `Bakeable` 类型参数扩展了 `Machine`，因此 `process` 方法现在接受 `Bakeable` 作为参数。

# 类型别名

**类型别名**提供了一种定义已存在类型名称的方法。类型别名可以帮助使复杂类型更容易阅读，并且还可以提供其他提示。

在某种意义上，`Oven` 接口只是一个名称，代表一个 `Machine<Bakeable>`：

```kt
typealias Oven = Machine<Bakeable>
```

我们新的类型别名 `Oven` 与我们熟悉的 `Oven` 接口完全一样。它可以被扩展，并具有 `Oven` 类型的值。

类型别名也可以用来增强类型信息，提供与你的领域相关的有意义的名称：

```kt
typealias Flavour = String

abstract class BakeryGood(val flavour: Flavour) {
```

它也可以用于集合：

```kt
typealias OvenTray = List<Bakeable>
```

它也可以与对象一起使用：

```kt
typealias CupcakeFactory = Cupcake.Companion
```

# 可空类型

Kotlin 的一个主要特性是可空类型。**可空类型**允许我们显式地定义一个值是否可以包含或为空：

```kt
fun main(args: Array<String>) {
    val myBlueberryCupcake: Cupcake = null //Compilation error: Null can not be a value of a non-null type Cupcake
}
```

在 Kotlin 中这并不有效；`Cupcake` 类型不允许空值。要允许空值，`myBlueberryCupcake` 必须有不同的类型：

```kt
fun main(args: Array<String>) {
    val myBlueberryCupcake: Cupcake? = null
}
```

从本质上讲，`Cupcake` 是一个非空类型，而 `Cupcake?` 是一个可空类型。

在层次结构中，`Cupcake` 是 `Cupcake?` 的子类型。因此，在任何 `Cupcake?` 被定义的情况下，`Cupcake` 可以被使用，但反之则不行：

```kt
fun eat(cupcake: Cupcake?){
//  something happens here    
}

fun main(args: Array<String>) {
    val myAlmondCupcake = Cupcake.almond()

    eat(myAlmondCupcake)

    eat(null)
}
```

Kotlin 编译器在可空类型和非空类型实例之间做出区分。

让我们以这些值为例：

```kt
fun main(args: Array<String>) {
    val cupcake: Cupcake = Cupcake.almond()
    val nullabeCupcake: Cupcake? = Cupcake.almond()
}
```

接下来，我们将对可空类型和非空类型都调用 `eat()` 方法：

```kt
fun main(args: Array<String>) {
    val cupcake: Cupcake = Cupcake.almond()
    val nullableCupcake: Cupcake? = Cupcake.almond()

    cupcake.eat() // Happy days
    nullableCupcake.eat() //Only safe (?.) or non-null asserted (!!.) calls are allowed on a nullable receiver of type Cupcake?
}
```

在 `cupcake` 上调用 `eat()` 方法就像吃馅饼一样简单；在 `nullableCupcake` 上调用 `eat()` 会产生编译错误。

为什么？对于 Kotlin 来说，从一个可空值中调用方法是有风险的，可能会抛出一个潜在的 `NullPointerException`（以下简称 **NPE**）。因此，为了安全起见，Kotlin 将其标记为编译错误。

如果我们真的想从一个可空值中调用一个方法或访问一个属性，会发生什么？

好吧，Kotlin 提供了处理可空值的选择，所有这些都是显式的。在某种程度上，Kotlin 在告诉你，“*展示给我，你知道你在做什么*”。

让我们回顾一些选项（在接下来的章节中我们还将介绍更多选项）。

# 检查空值

在 `if` 块中将空值检查作为一个条件：

```kt
fun main(args: Array<String>) {
    val nullableCupcake: Cupcake? = Cupcake.almond()

    if (nullableCupcake != null) {
      nullableCupcake.eat()
    }
}
```

Kotlin 将执行智能转换。在 `if` 块内部，`nullableCupcake` 是 `Cupcake`，而不是 `Cupcake?`；因此，可以访问任何方法或属性。

# 检查非空类型

这与上一个类似，但它直接检查类型：

```kt
if (nullableCupcake is Cupcake) {
  nullableCupcake.eat()
}
```

它也可以与 `when` 一起使用：

```kt
when (nullableCupcake) {
  is Cupcake -> nullableCupcake.eat()
}
```

两种选项，检查空值和非空类型，都有些冗长。让我们看看其他选项。

# 安全调用

**安全调用**允许在值非空时访问可空值的属性和方法（在底层，在字节码级别，安全调用被转换为 `if(x != null)`）：

```kt
nullableCupcake?.eat()
```

但是，如果你在表达式中使用它呢？

```kt
val result: String? = nullableCupcake?.eat()
```

如果我们的值是空，它将返回空，所以 `result` 必须有 `String?` 类型。

这样就打开了在链上使用安全调用的机会，如下所示：

```kt
val length: Int? = nullableCupcake?.eat()?.length
```

# Elvis (?:) 操作符

Elvis 操作符（`?:`）在表达式中使用空值时返回一个替代值：

```kt
val result2: String = nullableCupcake?.eat() ?: ""
```

如果`nullabluCupcake?.eat()`是`null`，则`?:`运算符将返回替代值`""`。

显然，Elvis 运算符可以与一系列安全调用一起使用：

```kt
val length2: Int = nullableCupcake?.eat()?.length ?: 0
```

# `!!!`运算符

而不是`null`值，`!!`运算符将抛出一个 NPE：

```kt
val result: String = nullableCupcake!!.eat()
```

如果你能够处理 NPE，则`!!`运算符提供了一个相当方便的功能，即免费智能转换：

```kt
val result: String = nullableCupcake!!.eat()

val length: Int = nullableCupcake.eat().length
```

如果`nullableCupcake!!.eat()`没有抛出 NPE，Kotlin 将从下一行开始将其类型从`Cupcake?`更改为`Cupcake`。

# Kotlin 的类型系统

**类型系统**是一组规则，用于确定语言构造的类型。

一个（好的）类型系统将帮助你：

+   确保程序组成部分以一致的方式连接

+   理解你的程序（通过减少你的认知负荷）

+   表达业务规则

+   自动低级优化

我们已经覆盖了足够的内容来理解 Kotlin 的类型系统。

# `Any`类型

Kotlin 中的所有类型都扩展自`Any`类型（等等，实际上这不是真的，但为了解释的目的，请耐心等待）。

我们创建的每个类和接口都隐式扩展了`Any`。因此，如果我们编写一个接受`Any`作为参数的方法，它将接收任何值：

```kt
fun main(args: Array<String>) {

    val myAlmondCupcake = Cupcake.almond()

    val anyMachine = object : Machine<Any> {
      override fun process(product: Any) {
        println(product.toString())
      }
    }

    anyMachine.process(3)

    anyMachine.process("")

    anyMachine.process(myAlmondCupcake)    
}
```

可空值呢？让我们看看它：

```kt
fun main(args: Array<String>) {

    val anyMachine = object : Machine<Any> {
      override fun process(product: Any) {
        println(product.toString())
      }
    }

    val nullableCupcake: Cupcake? = Cupcake.almond()

    anyMachine.process(nullableCupcake) //Error:(32, 24) Kotlin: Type mismatch: inferred type is Cupcake? but Any was expected
}
```

`Any`与任何其他类型相同，并且也有一个可空对应物，`Any?`。`Any`扩展自`Any?`。因此，最终，`Any?`是 Kotlin 类型系统层次结构中的顶级类。

# 最小公共类型

由于类型推断和表达式评估，有时在 Kotlin 中存在一些表达式，其中不清楚返回哪种类型。大多数语言通过返回可能类型选项之间的最小公共类型来解决这个问题。Kotlin 采取了不同的路线。

让我们看看一个模糊表达式的例子：

```kt
fun main(args: Array<String>) {
    val nullableCupcake: Cupcake? = Cupcake.almond()

    val length = nullableCupcake?.eat()?.length ?: ""
}
```

`length`的类型是什么？`Int`还是`String`？不，`length`值的类型是`Any`。这很合理。`Int`和`String`之间的最小公共类型是`Any`。到目前为止，一切顺利。现在让我们看看以下代码：

```kt
val length = nullableCupcake?.eat()?.length ?: 0.0
```

按照这个逻辑，在这种情况下，`length`应该有`Number`类型（`Int`和`Double`之间的公共类型），对吧？

错误的，`length`仍然是`Any`。Kotlin 在这些情况下不会搜索最小公共类型。如果你想得到一个特定的类型，它必须被显式声明：

```kt
val length: Number = nullableCupcake?.eat()?.length ?: 0.0
```

# `Unit`类型

Kotlin 没有`void`返回值的方法（就像 Java 或 C 那样）。相反，一个方法（或更准确地说，一个表达式）可以有一个`Unit`类型。

`Unit`类型表示表达式被调用是为了其副作用，而不是其返回值。`Unit`表达式的经典例子是`println()`，这是一个仅为了其副作用而被调用的方法。

`Unit`，就像任何其他 Kotlin 类型一样，从`Any`扩展而来，并且可以是可空的。`Unit?`看起来很奇怪且不必要，但这是为了与类型系统保持一致性。拥有一致的类型系统有多个优点，包括更好的编译时间和工具支持：

```kt
anyMachine.process(Unit)
```

# `Nothing` 类型

`Nothing` 是位于整个 Kotlin 层次结构底部的类型。`Nothing` 继承了所有 Kotlin 类型，包括 `Nothing?`。

但是，为什么我们需要 `Nothing` 和 `Nothing?` 类型？

`Nothing` 代表一个无法执行的表达式（基本上是抛出异常）：

```kt
val result: String = nullableCupcake?.eat() ?: throw RuntimeException() // equivalent to nullableCupcake!!.eat()
```

在 Elvis 操作符的一侧，我们有 `String`。在另一侧，我们有 `Nothing`。因为 `String` 和 `Nothing` 之间的公共类型是 `String`（而不是 `Any`），所以 `result` 的值是 `String`。

`Nothing` 对于编译器也有特殊的意义。一旦在表达式中返回 `Nothing` 类型，之后的行就会被标记为不可达。

`Nothing?` 是空值的类型：

```kt
val x: Nothing? = null

val nullsList: List<Nothing?> = listOf(null)
```

# 其他类型

类、接口和对象是面向对象类型系统的良好起点，但 Kotlin 提供了更多构造，例如数据类、注解和枚举（还有一个额外的类型，称为密封类，我们将在后面介绍）。

# 数据类

在 Kotlin 中创建主要目的是持有数据的类是一种常见模式（在其他语言中也是一种常见模式，例如 JSON 或 Protobuff）。

Kotlin 有一种特定的类用于此目的：

```kt
data class Item(val product: BakeryGood,
  val unitPrice: Double,
  val quantity: Int)
```

要声明数据类，有一些限制：

+   主要构造函数应至少有一个参数

+   主要构造函数的参数必须是 `val` 或 `var`

+   数据类不能是抽象的、开放的、密封的或内部的

在这些限制下，数据类提供了很多好处。

# 规范方法

**规范方法**是在 `Any` 中声明的。因此，Kotlin 中的所有实例都有这些方法。

对于数据类，Kotlin 会为所有规范方法创建正确的实现。

方法如下：

+   `equals(other: Any?): Boolean`: 此方法比较值等价性，而不是引用。

+   `hashCode(): Int`: 哈希码是实例的数值表示。当在同一个实例上多次调用 `hashCode()` 时，它应该始终返回相同的值。当两个实例在 `equals` 中返回 true 时，它们必须具有相同的 `hashCode()`。

+   `toString(): String`: 实例的字符串表示。当实例被连接到字符串时，将调用此方法。

# `copy()` 方法

有时，我们希望重用现有实例的值。`copy()` 方法允许我们创建数据类的新实例，并覆盖我们想要的参数：

```kt
val myItem = Item(myAlmondCupcake, 0.40, 5)

val mySecondItem = myItem.copy(product = myCaramelCupcake) //named parameter
```

在这种情况下，`mySecondItem` 从 `myItem` 复制 `unitPrice` 和 `quantity`，并替换 `product` 属性。

# 析构方法

按照惯例，任何具有一系列名为 `component1()`、`component2()` 等方法的类实例都可以用于析构声明。

Kotlin 将为任何数据类生成这些方法：

```kt
val (prod: BakeryGood, price: Double, qty: Int) = mySecondItem
```

`prod` 值使用 `component1()` 的返回值初始化，`price` 使用 `component2()` 的返回值，依此类推。尽管前面的示例使用了显式类型，但这些类型不是必需的：

```kt
val (prod, price, qty) = mySecondItem
```

在某些情况下，并不需要所有值。所有未使用的值都可以用（`_`）替换：

```kt
val (prod, _, qty) = mySecondItem
```

# 注解

注解是一种将元信息附加到您的代码上的方式（例如文档、配置等）。

让我们看看以下示例代码：

```kt
annotation class Tasty
```

一个`注解`本身可以被注解来修改其行为：

```kt
@Target(AnnotationTarget.CLASS)
@Retention(AnnotationRetention.RUNTIME)
annotation class Tasty
```

在这种情况下，`Tasty`注解可以应用于类、接口和对象，并且可以在运行时查询。

要获取完整的选项列表，请查看 Kotlin 文档。

注解可以有参数，但有一个限制，它们不能为空：

```kt
@Target(AnnotationTarget.CLASS)
@Retention(AnnotationRetention.RUNTIME)
annotation class Tasty(val tasty:Boolean = true)

@Tasty(false)
object ElectricOven : Oven {
  override fun process(product: Bakeable) {
    println(product.bake())
  }
}

@Tasty
class CinnamonRoll : Roll("Cinnamon")

@Tasty
interface Fried {
  fun fry(): String
}
```

要在运行时查询注解值，我们必须使用反射 API（`kotlin-reflect.jar`必须位于您的类路径中）：

```kt
fun main(args: Array<String>) {
    val annotations: List<Annotation> = ElectricOven::class.annotations

    for (annotation in annotations) {
        when (annotation) {
            is Tasty -> println("Is it tasty? ${annotation.tasty}")
            else -> println(annotation)
        }
    }
}
```

# 枚举

Kotlin 中的枚举是一种定义一组常量值的方式。枚举非常有用，但不仅限于配置值：

```kt
enum class Flour {
  WHEAT, CORN, CASSAVA
}
```

每个元素都是一个扩展了`Flour`类的对象。

就像任何对象一样，它们可以扩展接口：

```kt
interface Exotic {
  fun isExotic(): Boolean
}

enum class Flour : Exotic {
  WHEAT {
    override fun isExotic(): Boolean {
      return false 
    }
  },

  CORN {
    override fun isExotic(): Boolean {
      return false
    }
  },

  CASSAVA {
    override fun isExotic(): Boolean {
      return true
    }
  }
}
```

枚举也可以有抽象方法：

```kt
enum class Flour: Exotic {
  WHEAT {
    override fun isGlutenFree(): Boolean {
      return false
    }

    override fun isExotic(): Boolean {
      return false
    }
  },

  CORN {
    override fun isGlutenFree(): Boolean {
      return true
    }

    override fun isExotic(): Boolean {
      return false
    }
  },

  CASSAVA {
    override fun isGlutenFree(): Boolean {
      return true
    }

    override fun isExotic(): Boolean {
      return true
    }
  };

  abstract fun isGlutenFree(): Boolean
}
```

任何方法定义都必须在分隔最后一个元素的分号（`;`）之后声明。

当枚举与`when`表达式一起使用时，Kotlin 编译器会检查所有情况是否都已覆盖（单独或使用`else`）：

```kt
fun flourDescription(flour: Flour): String {
  return when(flour) { // error
    Flour.CASSAVA -> "A very exotic flavour"
  }
}
```

在这个例子中，我们只检查了`CASSAVA`，而没有检查其他元素；因此，它失败了：

```kt
fun flourDescription(flour: Flour): String {
  return when(flour) {
    Flour.CASSAVA -> "A very exotic flavour"
    else -> "Boring"
  }
}
```

# 摘要

在本章中，我们介绍了面向对象编程的基础以及 Kotlin 如何支持它。我们学习了如何使用类、接口、对象、数据类、注解和枚举。我们还探讨了 Kotlin 类型系统，并看到了它是如何帮助我们编写更好、更安全的代码的。

在下一章中，我们将从函数式编程的介绍开始。
