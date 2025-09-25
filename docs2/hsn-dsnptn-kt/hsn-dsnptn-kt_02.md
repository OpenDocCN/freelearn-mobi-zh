# 第二章：与创建型模式一起工作

在本章中，我们将介绍如何在 Kotlin 中实现经典创建型模式。这些模式处理*如何*和*何时*创建你的对象。掌握这些模式将使你能够更好地管理对象，适应变化，并编写易于维护的代码。

在本章中，我们将涵盖以下主题：

+   单例模式

+   工厂方法

+   抽象工厂

+   构造器

+   原型模式

# 单例模式

这是邻里的最受欢迎的单个男孩。每个人都认识他，每个人都谈论他，任何人都可以轻易找到他。

即使那些在其他设计模式被提及时会皱眉的人也会知道它的名字。在某个时候，它甚至被宣布为反模式，但这仅仅是因为它的广泛流行。所以，对于那些第一次听说它的人来说，这个模式是关于什么的？

通常，如果你有一个对象，你可以创建尽可能多的其实例。比如说，你有一个`Cat`类：

```kt
class Cat
```

你可以生产尽可能多的其实例（具体来说，是猫），你想要多少就有多少：

```kt
val firstCat = Cat()
val secondCat = Cat()
val yetAnotherCat = Cat()
```

没有任何问题。

如果我们想要禁止这种行为呢？显然，我们第一次必须以某种方式创建一个对象。但从第二次开始，我们需要认识到这个对象已经被初始化过一次，并返回其实例。这就是单例背后的主要思想。

在 Java 和一些其他语言中，这个任务相当复杂。仅仅将构造函数设为私有并*记住*该对象已经被初始化至少一次是不够的。我们还需要防止竞态条件，即两个不同的线程试图同时初始化它。如果我们允许这样做，就会破坏单例的整个概念，因为两个线程将持有同一对象的两个实例的引用。

在 Java 中解决此问题需要执行以下操作之一：

+   接受单例将在应用程序启动时立即初始化，而不是在首次访问时初始化

+   编写一些智能代码来防止这种竞态条件，同时保持性能

+   使用已经解决这个问题的框架

Kotlin 只是为这个引入了一个保留关键字。看，一个如下所示的对象：

```kt
object MySingelton{}
```

你不需要那里的大括号。它们只是为了视觉一致性。

这将声明和初始化结合在一个关键字中。从现在起，`MySingleton`可以从你的代码的任何地方访问，并且将只有一个实例。

当然，这个对象并没有做任何有趣的事情。让我们让它计算调用次数：

```kt
object CounterSingleton {
    private val counter = AtomicInteger(0)
```

```kt

    fun increment() = counter.incrementAndGet()
}
```

我们现在不会测试它的线程安全性，这是一个将在第八章中讨论的主题，即*线程和协程*，它涉及到线程。现在，我们只测试它是如何调用我们的单例的：

```kt
for (i in 1..10) {
    println(CounterSingleton.increment())
}
```

这将按预期打印出 1 到 10 之间的数字。正如你所看到的，我们根本不需要`getInstance()`方法。

`object`关键字不仅用于创建 Singleton。我们稍后会深入讨论。

对象不能有构造函数。如果你想为你的 Singleton 添加一些初始化逻辑，比如第一次从数据库或网络上加载数据，你可以使用`init`块代替：

```kt
object CounterSingleton {

    init {
        println("I was accessed for the first time")
    }
    // More code goes here
}
```

还演示了 Kotlin 中的 Singleton 是延迟初始化的，而不是像一些人从其声明的简便性所怀疑的那样是立即初始化的。就像常规类一样，对象可以扩展其他类并实现接口。我们将在第十章中回到这一点，*惯用和反模式*。

# 工厂方法

工厂方法完全是关于创建对象。但为什么我们需要一个方法来创建对象？这不是构造函数的全部内容吗？

嗯，构造函数有其固有的局限性，我们即将讨论。

# 工厂

我们从《设计模式》这本书中 Gang of Four 提出的工厂方法开始。

这是我教给学生们的第一个模式之一。他们通常对设计模式的整体概念非常焦虑，因为它有一种神秘和复杂的氛围。所以，我通常会问他们以下问题。

假设你有一些类声明，例如：

```kt
class Cat {
    val name = "Cat"
}
```

你能写一个函数返回该类的新实例吗？大多数人都能成功：

```kt
fun catFactory() : Cat {
    return Cat()
}
```

检查一切是否正常工作：

```kt
val c = catFactory() 
println(c.name) // Indeed prints "Cat"
```

嗯，这真的很简单，对吧？

现在，基于我们提供的参数，这个方法能否创建两个对象之一？

假设我们现在有一个`Dog`：

```kt
class Dog {
    val name = "Dog"
}
```

在两个类型的对象之间进行实例化选择只需要传递一个参数：

```kt
fun animalFactory(animalType: String) : Cat {
    return Cat()
}
```

当然，我们现在不能总是返回一个`Cat`。因此，我们创建一个公共接口来返回：

```kt
interface Animal {
   val name : String
}
```

剩下的就是使用`when`表达式返回正确类的实例：

```kt
return when(animalType.toLowerCase()) {
    "cat" -> Cat()
    "dog" -> Dog()
    else -> throw RuntimeException("Unknown animal $animalType")
}
```

这就是工厂方法的核心：

+   获取一些值。

+   返回实现公共接口的其中一个对象。

当从配置创建对象时，这个模式非常有用。想象一下，我们有一个文本文件，其内容如下，来自兽医诊所：

```kt
dog, dog, cat, dog, cat, cat
```

现在，我们希望为每种动物创建一个空配置文件。假设我们已经读取了文件内容并将它们分割成一个列表，我们可以做以下操作：

```kt
val animalTypes = listOf("dog", "dog", "cat", "dog", "cat", "cat")

for (t in animalTypes) {
  val c = animalFactory(t) 
    println(c.name)
} 
```

`listOf`是一个来自 Kotlin 标准库的函数，它创建了一个不可变列表，包含提供的对象。

如果你的工厂方法不需要状态，我们可以将其留为一个函数。

但如果我们想为每个动物分配一个唯一的顺序标识符呢？看看下面的代码块：

```kt
interface Animal {
   val id : Int
   // Same as before
}

class Cat(override val id: Int) : Animal { 
    // Same as before
}

class Dog(override val id: Int) : Animal {
    // Same as before
}
```

注意我们可以在构造函数内部覆盖值。

我们的工厂现在成为一个合适的类：

```kt
class AnimalFactory { 
    var counter = 0

    fun createAnimal(animalType: String) : Animal {
        return when(animalType.trim().toLowerCase()) {
            "cat" -> Cat(++counter)
            "dog" -> Dog(++counter)
            else -> throw RuntimeException("Unknown animal $animalType")
        }
    } 
}
```

因此，我们还需要初始化它：

```kt
val factory = AnimalFactory()
for (t in animalTypes) {
    val c = factory.createAnimal(t) 
    println("${c.id} - ${c.name}")
} 
```

上述代码的输出如下：

```kt
1 - Dog 
2 - Dog 
3 - Cat 
4 - Dog 
5 - Cat 
6 - Cat
```

这是一个相当直接的例子。我们为我们的对象提供了一个公共接口（在这个例子中是`Animal`），然后根据一些参数，我们决定实例化哪个具体类。

如果我们决定支持不同的品种呢？看看下面的代码：

```kt
val animalTypes = listOf("dog" to "bulldog", 
                         "dog" to "beagle", 
                         "cat" to "persian", 
                         "dog" to "poodle", 
                         "cat" to "russian blue", 
                         "cat" to "siamese")
```

就像我们在第一章“Kotlin 入门”中看到的`downTo`函数一样，它看起来像是一个操作符，但实际上是一个创建一对对象的函数：在我们的例子中是(`cat`, `siamese`)。当我们深入讨论`infix`函数时，我们还会回到它。

我们可以将实际的对象实例化委托给其他工厂：

```kt
class AnimalFactory {
    var counter = 0
    private val dogFactory = DogFactory()
    private val catFactory = CatFactory()

    fun createAnimal(animalType: String, animalBreed: String) : Animal {
        return when(animalType.trim().toLowerCase()) {
            "cat" -> catFactory.createDog(animalBreed, ++counter)
            "dog" -> dogFactory.createDog(animalBreed, ++counter)
            else -> throw RuntimeException("Unknown animal $animalType")
        }
    }
}
```

工厂重复相同的模式：

```kt
class DogFactory {
    fun createDog(breed: String, id: Int) = when(breed.trim().toLowerCase()) {
        "beagle" -> Beagle(id)
        "bulldog" -> Bulldog(id)
        else -> throw RuntimeException("Unknown dog breed $breed")
    }
}
```

你可以通过自己实现`Beagle`、`Bulldog`、`CatFactory`以及所有不同的猫品种来确保你理解这个例子。

最后一点要注意的是我们现在如何使用一对参数调用我们的`AnimalFactory`：

```kt
for ((type, breed) in animalTypes) {
    val c = factory.createAnimal(type, breed)
    println(c.name)
}
```

这被称为**解构声明**，在处理这样的数据对时特别有用。

# 静态工厂方法

静态工厂方法是由 Joshua Bloch 在他的书《Effective Java》中推广的。为了更好地理解它，让我们看看 Java 标准库本身的例子，即`valueOf()`方法：

```kt
Long l1 = new Long("1");
Long l2 = Long.valueOf("1");
```

构造函数和`valueOf()`方法都接收`String`作为输入，并产生`Long`作为输出。

那么，为什么静态工厂方法有时比构造函数更好？

# 静态工厂方法的优点

下面是静态工厂方法相对于构造函数的一些优点：

+   它为构造函数提供了一个更好的名称，它期望的，有时甚至是什么它产生的。

+   我们通常不期望构造函数抛出异常。另一方面，常规方法抛出的异常是完全有效的。

+   说到期望，我们期望构造函数是快速的。

但这些都是心理上的优势。这种方法还有一些技术上的优势。

# 缓存

静态工厂方法可能提供缓存，就像`Long`类那样。它不是为任何值总是返回一个新的实例，而是`valueOf()`方法会检查缓存中是否已经解析了这个值。如果是，它将返回缓存的实例。反复使用相同的值调用静态工厂方法可能产生的垃圾回收量比始终使用构造函数要少。

# 子类化

当调用构造函数时，我们总是实例化我们指定的类。另一方面，调用静态工厂方法可能产生类的实例，或者其子类之一。在讨论了在 Kotlin 中实现这种设计模式的实现之后，我们再回到这一点。

# Kotlin 中的静态工厂方法

我们已经在*单例*部分中讨论了`object`关键字。现在我们将看到它的另一个用途是`companion`对象。

在 Java 中，静态工厂方法是声明为`static`的。但在 Kotlin 中，没有这样的关键字。相反，不属于类实例的方法可以声明在`companion`对象内部：

```kt
class NumberMaster {
    companion object {
        fun valueOf(hopefullyNumber: String) : Long {
            return hopefullyNumber.toLong()
        }
    }
}
```

伴随对象可以有名称：例如，companion object Parser。但这只是为了清晰说明这个对象的目标。

调用`companion`对象不需要实例化一个类：

```kt
println(NumberMaster.valueOf("123")) // Prints 123
```

此外，直接在类的实例上调用它将不起作用，这与 Java 不同：

```kt
println(NumberMaster().valueOf("123")) // Won't compile
```

该类可能只有一个伴侣对象。

# 伴侣对象

在 Java 中，静态工厂方法声明如下：

```kt
private static class MyClass { 

 // Don't want anybody to use it but me 
  private MyClass() { 
  } 

 // This will replace the public constructor 
  public static MyClass create() { 
    return new MyClass(); 
  } 
} 
```

它们被这样称呼：

```kt
MyClass myClass = MyClass.create(); 
```

但在 Kotlin 中，没有这样的关键字叫做静态。相反，不属于类实例的方法可以在`companion`对象内部声明。

我们之前在*单例*部分讨论了`object`关键字，现在我们将通过以下示例来探讨这个重要关键字的其他用法：

```kt
   class NumberMaster { 
       companion object { 
           fun valueOf(hopefullyNumber: String) : Long { 
               return hopefullyNumber.toLong() 
           }  
       } 
   } 
```

正如你所见，在我们的类中，我们声明了一个以`companion`关键字为前缀的对象。

这个对象有一套自己的函数。这样的好处是什么？你可能想知道。

就像 Java 中的静态方法一样，调用`companion`对象不需要实例化一个类：

```kt
println(NumberMaster.valueOf("123")) // Prints 123 
```

此外，直接在类的实例上调用它将不起作用，这与 Java 不同：

```kt
println(NumberMaster().valueOf("123")) // Won't compile 
```

`companion`对象可能有一个名称解析器，例如。但这只是为了清晰说明这个对象的目标。

该类可能只有一个`companion`对象。

通过使用`companion`对象，我们可以实现与 Java 中完全相同的行为：

```kt
private class MyClass private constructor() { 

    companion object { 
        fun create(): MyClass { 
            return MyClass() 
        } 
    } 
} 
```

我们现在可以像以下代码所示那样实例化我们的对象：

```kt
// This won't compile 
//val instance = MyClass() 

// But this works as it should 
val instance = MyClass.create() 
```

Kotlin 证明了自己是一种非常实用的语言。它里面的每一个关键字都有接地气的含义。

# 抽象工厂

抽象工厂是一个被广泛误解的模式。它因非常复杂和奇特而臭名昭著，但实际上，它相当简单。如果你理解了工厂方法，你很快就会理解这个模式。这是因为抽象工厂是工厂的工厂。实际上就是这样。工厂是一个能够创建其他类的函数或类。抽象工厂是一个创建工厂的类。

你可能已经理解了这一点，但仍想知道这种模式的用途可能是什么。在现实世界中，抽象工厂的主要用途可能是框架，特别是 Spring 框架，它使用抽象工厂的概念通过注解和 XML 文件创建其组件。但鉴于创建我们自己的框架可能相当繁琐，让我们看看另一个这个模式非常有用的例子——战略游戏。

我们将其称为*CatsCraft 2: 狗的复仇*。

# 抽象工厂的实际应用

我们的战略游戏将包括建筑和单位。让我们从声明所有建筑共有的内容开始：

```kt
interface Building<in UnitType, out ProducedUnit> 
        where UnitType : Enum<*>, ProducedUnit : Unit {
    fun build(type: UnitType) : ProducedUnit
}
```

所有建筑都应该实现`build()`函数。在这里，我们第一次看到了 Kotlin 中的泛型，所以让我们稍微讨论一下。

# Kotlin 泛型简介

泛型是指定类型之间关系的一种方式。嗯，这并没有帮助解释多少，对吧？让我们再试一次。泛型是类型的抽象。不，仍然很糟糕。

我们将尝试一个示例，然后：

```kt
    val listOfStrings = mutableListOf("a", "b", "c") 
```

好吧，这很简单；我们已经多次覆盖了它。这段代码只是创建了一个字符串列表。但它实际上意味着什么？

让我们尝试以下代码行：

```kt
listOfStrings.add(1) 
```

这行代码无法编译。那是因为 `mutableListOf()` 函数使用了泛型：

```kt
public fun <T> mutableListOf(vararg elements: T): MutableList<T> 
```

泛型创建了一个期望。无论我们使用什么类型来创建我们的列表，从现在起我们只能放那种类型进去。这是一个伟大的语言特性，因为一方面，我们可以泛化我们的数据结构或算法。无论它们包含什么类型，它们都会以完全相同的方式工作。

另一方面，我们仍然有类型安全。`listOfStrings` 的 `first()` 函数保证返回一个 `String`（在这种情况下）而不是其他任何东西。

在泛型的方面，Kotlin 使用一种类似于 Java 但略有不同的方法。我们不会在本节中涵盖泛型的所有方面，但将只提供一些指导，以便更好地理解这个例子。随着我们的深入，我们将遇到更多泛型的用法。

让我们看看另一个例子。

我们将创建一个名为 `Box` 的类。我知道这很无聊：

```kt
class Box<T> { 
    private var inside: T? = null 

    fun put(t: T) { 
        inside = t 
    }    
    fun get(): T? = inside 
} 
```

然而，这个盒子的好处是，通过使用泛型，我可以把它几乎任何东西放进去，例如，一只猫：

```kt
class Cat 
```

当我创建一个盒子的实例时，我指定它可以容纳什么：

```kt
val box = Box<Cat>() 
```

在编译时，泛型将确保它只会持有正确类型的对象：

```kt
box.put(Cat()) // This will work 
val cat = box.get() // This will always return a Cat, because that's what our box holds 
box.put("Cat") // This won't work, String is not a Cat 
```

如你所知，Java 使用 `?` extends 和 super 关键字来指定只读和只写类型。

Kotlin 使用 `in`、`out` 和 `where` 的概念。

被标记为 `in` 的类型可以用作参数，但不能用作返回值。这也被称为协变。实际上，这意味着我们可以返回 `ProducedUnit` 或从它继承的类型，但不能返回在层次结构中高于 `ProducedUnit` 的类型。

被标记为 `out` 的类型只能用作返回值，不能用作参数。这被称为逆变。

此外，我们还可以使用 `where` 关键字对类型引入约束。在我们的情况下，我们要求第一个类型实现 `Type` 接口，而第二个类型实现 `Unit` 接口。

类型本身的名称，`UnitType` 和 `ProducedUnit`，可以是任何我们想要的，例如 `T` 和 `P`。但为了清晰起见，我们将使用更冗长的名称。

# 回到我们的基地

HQ 是一个可以生产其他建筑的特殊建筑。它记录了它至今为止所建造的所有建筑。同类型的建筑可以建造多次：

```kt
class HQ {
    val buildings = mutableListOf<Building<*, Unit>>()

    fun buildBarracks(): Barracks {
        val b = Barracks()
        buildings.add(b)
        return b
    }

    fun buildVehicleFactory(): VehicleFactory {
        val vf = VehicleFactory()
        buildings.add(vf)
        return vf
    }
}
```

你可能想知道关于泛型，星号 (`*`) 的含义是什么。它被称为星投影，意味着 *我对这个类型一无所知*。它与 Java 的原始类型类似，但它具有类型安全性。

所有其他建筑都会生产单位。单位可以是陆军或装甲车辆：

```kt
interface Unit 

interface Vehicle : Unit

interface Infantry : Unit
```

陆军可以是有步枪兵或火箭兵：

```kt
class Rifleman : Infantry

class RocketSoldier : Infantry

enum class InfantryUnits {
    RIFLEMEN,
    ROCKET_SOLDIER
}
```

在这里，我们第一次看到 `enum` 关键字。车辆可以是坦克或**装甲人员运输车**（**APCs**）：

```kt
class APC : Vehicle

class Tank : Vehicle

enum class VehicleUnits {
    APC,
    TANK
}
```

一个兵营是一个生产步兵的建筑：

```kt
class Barracks : Building<InfantryUnits, Infantry> {
    override fun build(type: InfantryUnits): Infantry {
        return when (type) {
            RIFLEMEN -> Rifleman()
            ROCKET_SOLDIER -> RocketSoldier()
        }
    }
}
```

我们不需要在 `when` 中使用 `else` 块。这是因为我们使用了 `enum`，Kotlin 确保了 `enum` 上的 `when` 是穷尽的。

一个车辆工厂是一个生产不同类型装甲车辆的建筑：

```kt
class VehicleFactory : Building<VehicleUnits, Vehicle> {
    override fun build(type: VehicleUnits) = when (type) {
        APC -> APC()
        TANK -> Tank()
    }
}
```

我们现在可以确保能够构建不同的单元：

```kt
val hq = HQ()
val barracks1 = hq.buildBarracks()
val barracks2 = hq.buildBarracks()
val vehicleFactory1 = hq.buildVehicleFactory()
```

接下来是单元的生成：

```kt
val units = listOf(
        barracks1.build(InfantryUnits.RIFLEMEN),
        barracks2.build(InfantryUnits.ROCKET_SOLDIER),
        barracks2.build(InfantryUnits.ROCKET_SOLDIER),
        vehicleFactory1.build(VehicleUnits.TANK),
        vehicleFactory1.build(VehicleUnits.APC),
        vehicleFactory1.build(VehicleUnits.APC)
)
```

我们已经看到了标准库中的 `listOf()` 函数。它将创建一个只读列表，其中包含我们建筑生产的不同单元。你可以遍历这个列表，确保这些确实是所需的单元。

# 进行改进

有些人可能会说，拥有 `VehicleFactory` 和 `Barracks` 类太繁琐了。毕竟，它们没有任何状态。相反，我们可以用对象来替换它们。

我们可以不用之前的 `buildBarracks()` 实现方式，而是有以下方式：

```kt
fun buildBarracks(): Building<InfantryUnits, Infantry> {
    val b = object : Building<InfantryUnits, Infantry> {
        override fun build(type: InfantryUnits): Infantry {
            return when (type) {
                InfantryUnits.RIFLEMEN -> Rifleman()
                InfantryUnits.ROCKET_SOLDIER -> RocketSoldier()
            }
        }
    }
    buildings.add(b)
    return b
}
```

我们已经看到了 `object` 关键字的不同用法：一次是在单例设计模式中，另一次是在工厂方法设计模式中。这里是我们使用 `object` 的第三种方式：用于动态创建匿名类。毕竟，`Barracks` 是一个建筑，给定 `InfantryUnitType`，它会产生 `Infantry`。

如果我们的逻辑简单，我们甚至可以进一步缩短声明：

```kt
fun buildVehicleFactory(): Building<VehicleUnits, Vehicle> {
    val vf = object : Building<VehicleUnits, Vehicle> {
        override fun build(type: VehicleUnits) = when (type) {
            VehicleUnits.APC -> APC()
            VehicleUnits.TANK -> Tank()
        }
    }
    buildings.add(vf)

    return vf
}
```

让我们回到本章的开头。我们提到抽象工厂结合了多个相关的工厂。那么，在我们的案例中，所有工厂的共同点是什么？它们都是建筑，并且都生产单元。

有了这个原则，你就可以将其应用于许多不同的场景。如果你熟悉策略游戏，通常它们至少有两个不同的派系。每个派系可能都有不同的结构和单位。为了实现这一点，你可以根据需要重复这个模式。

假设我们现在有两个不同的派系，猫和狗，坦克和火箭步兵是这个派系的特权。狗有重型坦克和掷弹兵。我们需要在我们的系统中做出哪些改变？

首先，`HQ` 变成了一个接口：

```kt
interface HQ {
    fun buildBarracks(): Building<InfantryUnits, Infantry>
    fun buildVehicleFactory(): Building<VehicleUnits, Vehicle>
}
```

之前 `HQ` 的功能现在变成了 `CatHQ`：

```kt
class CatHQ : HQ { 
// Remember to add override to your methods
}
```

`DogHQ` 将需要重复相同的步骤，但使用不同的构建逻辑。

这种适应大变化的能力使得抽象工厂在某些用例中非常强大。

# 构建器

有时候，我们的对象非常简单，只有一个构造函数，无论是空的还是非空的。但有时，它们的创建非常复杂，基于很多参数。我们已经看到了一个提供 *更好的构造函数* 的模式——静态工厂方法设计模式。现在，我们将讨论构建器设计模式，它在某些方面相似，在某些方面不同。

# 编写电子邮件

作为软件架构师，我主要的沟通渠道之一是电子邮件。这可能对大多数软件开发角色都适用。

一封电子邮件包含以下内容：

+   地址（至少一个必填）

+   CC（零个或多个，可选）

+   标题（可选）

+   正文（可选）

+   附件（零个或多个，可选）

假设我真的很懒，并且想在我在街区骑自行车的时候安排发送电子邮件。

实际的调度逻辑将被推迟到第八章线程和协程和第九章设计用于并发，这两章讨论了调度和并发。现在，让我们看看我们的 `Mail` 类可能看起来像什么：

```kt
data class Mail(val to: String, 
           val cc: List<String>, 
           val bcc: List<String>,
           val title: String?,
           val message: String)
```

因此，我们在前面的章节中已经看到了 `data class` 的实际应用。我们还讨论了可空和非可空类型，例如 `String?` 与 `String`。

现在是讨论 Kotlin 中集合如何工作的好时机，因为这是我们第一次直接处理它们的类。

# Kotlin 中的集合类型

Kotlin 的主要目标之一是 Java 互操作性。所以 Kotlin 集合与 Java 兼容也就不足为奇了。当你指定你的函数接收 `List<T>` 时，实际上是你所熟悉的相同的 Java `List<T>`。

但 Kotlin 区分可变和不可变集合。`listOf()` 函数委托给 `Arrays.asList()`，并生成一个不可变列表，而 `mutableListOf()` 简单地调用 `ArrayList()`。

除了数据之外，Kotlin 集合还有许多有用的扩展方法，我们将在后面讨论。

# 创建电子邮件 - 第一次尝试

因此，在上午 10 点，我计划在我当地的咖啡馆喝咖啡。但我也想联系我的经理，因为我的工资条昨天没有到达。我尝试创建我的第一个电子邮件如下：

```kt
val mail = Mail("manager@company.com", // TO
    null,   // CC
    null,   // BCC
    "Ping", // Title
    null    // Message)
```

这可能在 Java 中有效，但在 Kotlin 中不会编译，因为我们不能将 `null` 传递给 `List<String>`。空安全在 Kotlin 中非常重要：

```kt
val mail = Mail("manager@company.com", // TO
    listOf(),  // CC
    listOf(),  // BCC
    "Ping",    // Title
    null       // Message) 
```

注意，由于我们的构造函数接收了很多参数，我不得不添加一些注释，这样我就不会迷路了。

Kotlin 编译器足够智能，可以推断我们传递的列表类型。由于我们的构造函数接收 `List<String>`，传递 `listOf()` 对于空列表就足够了。我们不需要像这样指定类型：`listOf<String>()`。在 Java 中，菱形运算符起到相同的作用。

哦，但我忘了附件。让我们改变我们的构造函数：

```kt
data class Mail(val to: String, 
           val cc: List<String>, 
           val bcc: List<String>,
           val title: String?,
           val message: String?,
           val attachments: List<java.io.File>)
```

但然后我们的实例化又停止编译了：

```kt
val mail = Mail("manager@company.com", // TO
    listOf(), listOf(),
    "Ping",
    null) // Compilation error, No value passed for for parameter 'attachments'
```

这显然变得一团糟。

# 创建电子邮件 - 第二次尝试

让我们尝试一种流畅的设置方法。在我们的构造函数中，我们只有必填字段，其他所有字段都将成为设置器，因此创建一个新电子邮件看起来可能像这样：

```kt
Mail("manager@company.com").title("Ping").cc(listOf<String>())
```

这在很多方面都好多了：

+   字段的顺序现在可以是任意的，与构造函数不同。

+   哪个字段被设置现在更清晰，不再需要注释。

+   可选字段根本不需要设置。例如，CC 字段被设置，而 BCC 字段被省略。

让我们看看实现这种方法的一种方式。还有其他方便的方法来做这件事，我们将在第十章惯用和反模式中讨论：

```kt
data class Mail(// Stays the same
                private var _message: String = "",
                // ...) {
    fun message(message: String) : Mail {
        _message = message
        return this
    }
```

```kt
    // Pattern repeats for every other variable
}
```

在 Kotlin 中，使用下划线作为私有变量的命名约定是一种常见做法。这允许我们避免重复 `this.message = message` 和像 `message = message` 这样的错误。

这很好，非常类似于我们可能在 Java 中实现的效果。尽管我们现在不得不使我们的消息可变。但 Kotlin 提供了两种其他方法，你可能觉得它们更有用。

# 创建电子邮件——Kotlin 方式

和其他一些现代语言一样，Kotlin 提供了为函数参数设置 *默认值* 的能力：

```kt
data class Mail(val to: String, 
    val title: String = "",
    val message: String = "",
    val cc: List<String> = listOf(), 
    val bcc: List<String> = listOf(), 
    val attachments: List<java.io.File> = listOf())
```

因此，如果你想要发送一个没有 CC 的电子邮件，现在可以这样操作：

```kt
val mail = Mail("one@recepient.org", "Hi", "How are you")
```

但是，如果你想要发送带有 BCC 的电子邮件怎么办？而且，使用流畅设置器不需要指定顺序，这非常方便。Kotlin 提供了 *命名参数* 来实现这一点：

```kt
val mail = Mail(title= "Hello", message="There", to="my@dear.cat")
```

将默认参数与命名参数结合使用，使得在 Kotlin 中创建复杂对象比以前容易得多。还有另一种实现类似行为的方法：`apply()` 函数。这是 Kotlin 中每个对象都有的扩展函数之一。不过，为了使用这种方法，我们需要将所有可选字段改为变量而不是值：

然后，我们可以这样创建我们的电子邮件：

```kt
val mail = Mail("hello@mail.com").apply {
    message = "Something"
    title = "Apply"
}
```

`apply()` 函数是 **作用域函数** 家族中唯一的函数。我们将在后面的章节中讨论作用域函数的工作原理及其用途。现在，虽然我的老板认为我正在努力发送所有这些电子邮件，但我可以回到我的咖啡那里。现在它已经凉了！

# 创建电子邮件——Kotlin 方式——第二次尝试

让我们尝试一种流畅的设置方法，而不是这样。在我们的构造函数中，我们只有必填字段，其余的都将变成设置器。因此，要创建一个新的电子邮件，我们不再需要做以下操作：

```kt
   val mail = Mail("manager@company.com") 
   mail.title("Ping") 
   mail.cc(listOf<String>()) 
```

相反，我们将这样做：

```kt
Mail("manager@company.com").title("Ping").cc(listOf<String>())
```

流畅设置器允许我们将一个设置调用链接到另一个。

这在很多方面都更加方便：

+   字段的顺序现在可以是任意的，与构造函数中使用的顺序不同。

+   现在更清楚哪个字段正在被设置；不再需要注释。

+   可选字段根本不需要设置。例如，CC 字段被设置，而 BCC 字段被省略。

让我们看看实现这种方法的一种方式。还有其他方便的方法来做这件事，我们将在第十章第二百三十六部分“惯用句和反模式”中讨论：

```kt
   data class Mail(// Stays the same 
                   private var _message: String = "", 
                   // ...) { 
       fun message(message: String) : Mail { 
           _message = message 
return this } 
       // Pattern repeats for every other variable 
   } 
```

在 Kotlin 中，使用下划线作为私有变量的命名约定是一种常见做法。这允许我们避免重复短语 `this.message = message` 和错误，例如 `message = message`。

这很好，并且非常类似于我们可能在 Java 中实现的效果，尽管我们现在不得不使我们的消息可变。

当然，我们也可以实现一个完整的构建器设计模式：

```kt
class MailBuilder(val to: String) { 
    private var mail: Mail = Mail(to) 
    fun title(title: String): MailBuilder { 
        mail.title = title 
        return this 
    }
    // Repeated for other properties 
    fun build(): Mail { 
        return mail 
    } 
} 
```

你可以用以下方式创建你的电子邮件：

```kt
val email = MailBuilder("hello@hello.com").title("What's up?").build()
```

但 Kotlin 提供了两种其他方法，你可能觉得它们更有用。

# 原型

这个设计模式完全是关于定制和创建相似但略有不同的对象。为了更好地理解它，我们将从一个例子开始。

# 构建你的电脑

想象一下，你有一个卖电脑的商店。

普通电脑由以下组成：

+   主板

+   CPU

+   图形卡

+   内存

大多数顾客实际上并不关心你在这台电脑中放入了什么组件。他们真正关心的是这台电脑是否能够在 60fps（每秒帧数）下运行*《壮丽盗车 7》*。

因此，你决定这样构建它：

```kt
data class PC(val motherboard: String = "Terasus XZ27",
             val cpu: String = "Until Atom K500",
             val ram: String = "8GB Microcend BBR5",
             val graphicCard: String = "nKCF 8100TZ")
```

所以当一位新顾客进来，想要尝试在邻里中大家都在谈论的这个游戏时，你只需这样做：

```kt
val pc = PC()
```

他们已经朝着家的方向出发，准备分享他们从 MPC7 获得的新经验。实际上，你的生意做得如此好，以至于你有一台电脑就放在那里，随时准备迎接下一个顾客的到来。

但然后又来了一位顾客。这位顾客很懂技术。所以，坦白说，他们认为对于他们玩的游戏来说，*nKCF 8100TZ 图形卡*根本不够。他们也了解到现在有*BBR6 内存*可用，他们想要*16GB*。当然，他们希望立即得到。但他们愿意用现金支付。

那就是你想对仓库里坐着的这台电脑稍作修改的时候，而不是组装一台新的。

# 从原型开始

原型的整个想法是能够轻松地克隆一个对象。你可能有很多原因想要这样做：

+   创建你的对象非常昂贵。你需要从数据库中检索它。

+   你创建的对象彼此相似但不同，你不想一遍又一遍地重复相似的部分。

使用这个模式还有更多高级的理由。例如，JavaScript 语言使用原型来实现类似于类的继承行为，而不使用类。

幸运的是，Kotlin 修复了*损坏的*Java `clone()` 方法。对于数据类，有`copy()`方法，它接受一个现有的数据类，并创建它的一个新副本，在此过程中可以选择更改一些属性：

```kt
val pcFromWarehouse = PC() // Our boring PC

val pwnerPC = pcFromWarehouse.copy(graphicCard = "nKCF 8999ZTXX",
        ram = "16GB BBR6") // Amazing PC

println(pwnerPC) // Make sure that PC created correctly
```

默认情况下，`clone()`方法创建一个浅拷贝，这可能对经验不足的开发者来说是不预期的。在 Java 中正确实现`clone()`方法非常困难。你可以在[`dzone.com/articles/shallow-and-deep-java-cloning`](https://dzone.com/articles/shallow-and-deep-java-cloning)上阅读关于各种陷阱的文章。

与我们在 Builder 设计模式中看到的情况类似，命名参数允许我们以任意顺序指定可以更改的属性。

剩下的唯一事情就是数现金，再买一些那些*nKCF 图形卡*。以防万一。

# 摘要

在本章中，我们学习了何时以及如何使用创建型家族中的设计模式。我们了解了`object`关键字的不同用法：作为单例、作为静态工厂方法的容器，以及作为接口匿名实现的实现。然后，我们通过使用`in`、`out`和`where`关键字，在 Kotlin 中看到了解构声明和泛型的工作原理。我们还学习了默认参数值和命名参数，随后是数据类的`copy()`函数。

在下一章中，我们将介绍设计模式的第二组，即结构型模式。这些模式有助于扩展我们对象的功能。
