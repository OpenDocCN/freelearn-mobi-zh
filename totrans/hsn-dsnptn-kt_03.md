# 理解结构型模式

本章涵盖了 Kotlin 中的结构型模式。一般来说，结构型模式处理对象之间的关系。

我们将讨论如何在不产生复杂的类层次结构的情况下扩展我们对象的功能，以及如何适应未来的变化或如何修复过去所做的某些决定，以及如何减少我们程序的内存占用。

在本章中，我们将涵盖以下主题：

+   装饰者

+   适配器

+   桥接

+   组合

+   外观

+   享元

+   代理

# 装饰者

在上一章中，我们讨论了**原型**设计模式，它允许创建具有略微（或不是那么略微）不同数据的类的实例。

如果我们想要创建一组具有略微不同行为的类呢？好吧，由于 Kotlin 中的函数是一等公民（更多内容将在后面讨论），你可以使用原型设计模式来实现这一点。毕竟，这正是 JavaScript 成功做到的事情。但本章的目标是讨论解决同一问题的另一种方法。毕竟，设计模式都是关于方法的。

通过实现这个设计模式，我们允许我们的代码用户指定他们想要添加的能力。

# 增强类

你的老板——抱歉，敏捷大师——昨天向你提出了一个紧急要求。从现在开始，你系统中的所有映射数据结构都必须变成`HappyMaps`。

那么，你不知道什么是`HappyMaps`吗？它们是目前最热门的东西。它们就像常规的`HashMap`一样，但是当你覆盖现有的值时，它们会打印出以下输出：

```kt
Yay! Very useful
```

所以，你在编辑器中输入以下代码：

```kt
class HappyMap<K, V>: HashMap<K, V>() {
    override fun put(key: K, value: V): V? {
        return super.put(key, value).apply {
            this?.let {
                println("Yay! $key")
            }
        }
    }
}
```

当我们在上一章讨论**构建者**设计模式时已经看到了`apply()`，而`this?.let { ... }`是一个更优雅的方式来表达`if (this != null) { ... }`。

我们可以使用以下代码来测试我们的解决方案：

```kt
fun main(args : Array<String>) {
    val happy = HappyMap<String, String>()
    happy["one"] = "one"
    happy["two"] = "two"
    happy["two"] = "three"
}
```

前面的代码按预期打印以下输出：

```kt
Yay! two
```

那就是唯一被重载的关键。

# 运算符重载

稍等一下，当我们扩展一个映射时，方括号是如何继续工作的？它们不是某种魔法吗？

嗯，实际上没有。这里没有魔法。正如你可能从本节标题猜到的，Kotlin 支持运算符重载。运算符重载意味着同一个运算符根据它接收到的参数类型的不同而表现出不同的行为。

如果你曾经使用过 Java，你已经熟悉运算符重载了。想想加号运算符是如何工作的。让我们看看这里给出的例子：

```kt
System.out.println(1 + 1); // 2
System.out.println("1" + "1") // 11
```

根据两个参数是字符串还是整数，`+`符号的行为不同。

但是，在 Java 的世界里，这只有语言本身才能做到。无论我们怎么尝试，以下代码都无法编译：

```kt
List<String> a = Arrays.asList("a");
List<String> b = Collections.singletonList("b"); // Same for one argument
List<String> c = a + b;
```

在 Java 9 中，还有一个`List.of()`，它服务于与`Arrays.asList()`类似的目的。

在 Kotlin 中，相同的代码打印出`[a, b]`：

```kt
val a = listOf("a")
val b = listOf("b")
println(a + b)
```

好吧，这很有道理，但也许它只是语言特性：

```kt
data class Json(val j: String)
val j1 = Json("""{"a": "b"}""")
val j2 = Json("""{"c": "d"}""")
println(j1 + j2) // Compilation error!
```

我告诉过您，这很神奇！您不能简单地将两个任意的类组合在一起。

但等等。如果我们为我们的`Json`类创建一个扩展函数`plus()`，如下所示：

```kt
operator fun Json.plus(j2: Json): Json {
   // Code comes here
}
```

除了第一个关键字`operator`之外，其他所有内容都应该对您很熟悉。我们通过添加一个新的函数来扩展`Json`对象，该函数获取另一个`Json`并返回`Json`。

我们实现函数体是这样的：

```kt
val jsonFields = this.j.split(":") + j2.j.split(":")
val s = (jsonFields).joinToString(":")
return Json ("""{$s}""")
```

这实际上并不是连接任何 JSON，而是在我们的例子中连接`Json`。我们从我们的`Json`中取值，从另一个`Json`中取值，然后将它们连接起来，并在它们周围加上一些花括号。

现在看看这一行：

```kt
println(j1 + j2)
```

上述代码会打印出以下输出：

```kt
{{"a": "b"}:{"c": "d"}}
```

实际上，它会打印出：`Json(j={{"a": "b"}}:{"c": "d"}})`。这是因为我们为了简洁起见，在示例中没有重写`toString()`方法。

那么，这个`operator`关键字是什么意思呢？

与其他一些语言不同，您不能覆盖 Kotlin 语言中存在的每个操作符，而只能选择一些。

尽管有限，但可以重写的所有操作符列表相当长，所以我们在这里不列出。您可以在官方文档中查阅：

[`kotlinlang.org/docs/reference/operator-overloading.html`](https://kotlinlang.org/docs/reference/operator-overloading.html)。

尝试将您的扩展方法重命名为：

+   `prus()`: 只是一个打字错误的名称

+   `minus()`: 与减号`-`相关联的现有函数

您会发现您的代码无法编译。

我们一开始使用的方括号被称为**索引访问操作符**，与`get(x)`和`set(x, y)`方法相关联。

# 嘿，我的映射在哪里？

第二天，您的产品经理联系您。显然，他们现在想要一个`SadMap`，每次从其中删除一个键时，它都会变得**悲伤**。按照之前的模式，您再次扩展了映射：

```kt
class SadMap<K, V>: HashMap<K, V>() {
    override fun remove(key: K): V? {
        println("Okay...")
        return super.remove(key)
    }
}
```

但然后首席架构师要求在某些情况下，一个映射可以同时感到高兴和悲伤。CTO 已经有一个关于`SuperSadMap`的伟大想法，它将打印出以下输出两次：

```kt
Okay...
```

那么，我们需要的是结合我们对象行为的能力。

# 伟大的组合者

这次我们将有所不同。我们不会逐个组合解决方案，而是先看完整的解决方案，然后分解它。这里的代码将帮助您理解原因：

```kt
class HappyMap<K, V>(private val map: MutableMap<K, V> =                                        mutableMapOf()) : 
      MutableMap<K, V> by map {

    override fun put(key: K, value: V): V? {
        return map.put(key, value).apply {
            this?.let { println("Yay! $key") }
        }
    }
}
```

这里的难点在于理解签名。在装饰器模式中，我们需要的是：

+   能够接收我们正在装饰的对象

+   保留对其的引用

+   当我们的装饰器被调用时，我们决定是否要改变我们持有的对象的行为，或者要委托调用

由于我们需要实际做很多事情，这个声明相当复杂。毕竟，它在一行中做了很多事情，这应该相当令人印象深刻。让我们逐行分解：

```kt
class HappyMap<K, V>(...
```

我们班级的名字是`HappyMap`，它有两个类型参数，`K`和`V`，分别代表**键**和**值**：

```kt
... (private val map: MutableMap<K, V> ...
```

在我们的构造函数中，我们接收带有类型`K`和`V`的`MutableMap`，与我们的相同：

```kt
... = mutableMapOf()) ...
```

如果没有传递映射，我们使用默认参数值初始化我们的属性，这个默认值是一个空的可变映射：

```kt
... : MutableMap<K, V> ...
```

我们的这个类扩展了`MutableMap`接口：

```kt
... by map
```

它也将所有未重写的方法**委托**给我们将要包装的对象，在我们的例子中是一个映射。

使用代理的`SadMap`代码被省略了，但你可以通过组合`HappyMap`的声明和之前`SadMap`的实现轻松地重新生成它。

现在我们来组合我们的`SadHappyMap`，以取悦首席架构师：

```kt
val sadHappy = SadMap(HappyMap<String, String>())
sadHappy["one"] = "one"
sadHappy["two"] = "two"

```

```kt
sadHappy["two"] = "three"
sadHappy["a"] = "b"
sadHappy.remove("a")
```

我们得到以下输出：

```kt
Yay! two // Because it delegates to HappyMap
Okay...  // Because it is a SadMap
```

同样地，我们现在可以创建`SuperSadMap`：

```kt
val superSad = SadMap(HappyMap<String, String>())
```

我们也可以取悦 CTO。

装饰器设计模式在`java.io.*`包中广泛使用，包括如 reader 和 writer 等类。

# 注意事项

装饰器设计模式很棒，因为它让我们可以**即时**组合对象。使用 Kotlin 的`by`关键字将使其简单易行。但仍然有一些限制，你需要注意。

首先，你无法看到装饰器的**内部**：

```kt
println(sadHappy is SadMap<*, *>) // True
```

那是顶层包装器，所以没问题：

```kt
println(sadHappy is MutableMap<*, *>) // True
```

那就是我们实现的接口，所以编译器知道它：

```kt
println(sadHappy is HappyMap<*, *>) // False
```

尽管`SadMap`包含`HappyMap`并且可能表现得像它，但它并不是一个`HappyMap`！在执行类型转换和类型检查时，请记住这一点。

第二点，与第一点相关，是装饰器通常不知道它直接包装的是哪个类，这很难进行优化。想象一下我们的 CTO 要求`SuperSadMap`打印`Okay... Okay...`然后就是它，在同一行。为了做到这一点，我们需要捕获整个输出，或者调查我们将要包装的所有类，这些任务相当复杂。

使用这个强大的设计模式时，请记住这些要点。它允许动态地向对象添加新责任（在我们的例子中，打印`Yay`是一种责任），而不是通过子类化对象。每个新的责任都是你添加的新包装层。

# 适配器

适配器，或者有时被称为包装器，其主要目标是转换一个接口到另一个接口。在物理世界中，最好的例子就是一个电源插头适配器，或者一个 USB 适配器。

想象一下自己在晚上很晚的时候，手机电量只剩下 7%。你的手机充电器忘在了办公室，在城市的另一头。你只有一款欧盟插头充电器和一根 USB 迷你线。但你的手机是 USB Type-C，因为你不得不升级。而且你人在纽约，所以所有的插座当然都是 US Type-A。你该怎么办？哦，很简单。你在半夜找一根 USB 迷你转 USB Type-C 适配器，并希望在这个过程中也不要忘记带上那个欧盟转 US 插头适配器。电量只剩下 5%，时间正在流逝。

因此，现在我们更好地理解了适配器在物理世界中的作用，让我们看看我们如何在代码中应用同样的方法。

让我们从接口开始：

```kt
interface UsbTypeC
interface UsbMini

interface EUPlug
interface USPlug
```

现在，我们可以声明一个手机和一个电源插座：

```kt
// Power outlet exposes USPlug interface
fun powerOutlet() : USPlug {
    return object : USPlug {}
}

```

```kt
fun cellPhone(chargeCable: UsbTypeC) {

}
```

当然，我们的充电器在各个方面都是错误的：

```kt
// Charger accepts EUPlug interface and exposes UsbMini interface
fun charger(plug: EUPlug) : UsbMini {
    return object : UsbMini {}
}
```

这里我们遇到了以下错误：

```kt
Type mismatch: required EUPlug, found USPlug: charger(powerOutlet())

Type mismatch: required UsbTypeC, found UsbMini: cellPhone(charger(powerOutlet()))
```

# 不同的适配器

因此，我们需要两种类型的适配器。

在 Java 中，你通常会为此创建一对类。在 Kotlin 中，我们可以用扩展函数来替换它们。

我们可以通过以下扩展函数采用美国插头来与欧盟插头一起工作：

```kt
fun USPlug.toEUPlug() : EUPlug {
    return object : EUPlug {
        // Do something to convert 
    }
}
```

我们可以用类似的方式创建一个迷你 USB 和 type-C USB 之间的 USB 适配器：

```kt
fun UsbMini.toUsbTypeC() : UsbTypeC {
    return object : UsbTypeC {
        // Do something to convert
    }
}
```

最后，通过将这些适配器组合在一起，我们重新上线：

```kt
cellPhone(
    charger(
        powerOutlet().toEUPlug()
    ).toUsbTypeC()
)
```

如你所见，我们不需要在对象内部组合一个对象来适配它们。幸运的是，我们也不需要继承接口和实现。有了 Kotlin，我们的代码保持简短而直接。

# 适配器在现实世界中的应用

你可能也遇到过这些适配器。大多数情况下，它们在*概念*和*实现*之间进行适配。例如，让我们以*集合*的概念与*流*的概念为例：

```kt
val l = listOf("a", "b", "c")

fun <T> streamProcessing(stream: Stream<T>) { 
    // Do something with stream
}
```

即使这样做在逻辑上可能合理，你也不能简单地将一个集合传递给接收流的功能：

```kt
streamProcessing(l) // Doesn't compile
```

幸运的是，集合为我们提供了`.stream()`方法：

```kt
streamProcessing(l.stream()) // Adapted successfully
```

# 使用适配器的注意事项

你有没有通过适配器将 110v 的美国电器插入 220v 的欧盟插座，然后完全烧毁过？

如果你不小心，这可能会发生在你的代码上。以下使用另一个适配器的示例可以编译：

```kt
fun <T> collectionProcessing(c: Collection<T>) {
    for (e in c) {
        println(e)
    }
}

val s = Stream.generate { 42 }
collectionProcessing(s.toList())
```

但是它永远不会完成，因为`Stream.generate()`产生了一个无限整数列表。所以，要小心，并且明智地应用这个模式。

# 桥接

与我们遇到的其他设计模式不同，桥接模式更少关注于以智能方式组合对象，而是更多地关于如何不滥用继承的指导原则。它的工作方式实际上非常简单。

让我们回到我们正在构建的策略游戏。我们为所有的步兵单位都有一个接口：

```kt
interface Infantry {
    fun move(x: Long, y: Long)

    fun attack(x: Long, y: Long)
}
```

我们有具体的实现：

```kt
open class Rifleman : Infantry {
    override fun attack(x: Long, y: Long) {
        // Shoot
    }

    override fun move(x: Long, y: Long) {
        // Move at its own pace
    }
}

open class Grenadier : Infantry {
    override fun attack(x: Long, y: Long) {
        // Throw grenades
    }

    override fun move(x: Long, y: Long) {
        // Moves slowly, grenades are heavy
    }
}
```

如果我们想要升级我们的单位的能力呢？

升级后的单位应该有双倍的伤害，但移动速度保持不变：

```kt
class UpgradedRifleman : Rifleman() {
    override fun attack(x: Long, y: Long) {
        // Shoot twice as much
    }
}

class UpgradedGrenadier : Grenadier() {
    override fun attack(x: Long, y: Long) {
        // Throw pack of grenades
    }
}
```

现在，我们的游戏设计师决定我们也需要这些单位的轻量版。也就是说，它们以与常规单位相同的方式攻击，但移动速度是它们的两倍：

```kt
class LightRifleman : Rifleman() {
    override fun move(x: Long, y: Long) {
        // Running with rifle
    }
}

class LightGrenadier : Grenadier() {
    override fun move(x: Long, y: Long) {
        // I've been to gym, pack of grenades is no problem
    }
}
```

由于设计模式都是关于适应变化的，所以我们的设计师来了，并要求所有步兵单位都能够呼喊，也就是说，大声清晰地宣布他们的单位名称：

```kt
interface Infantry {
    // As before, move() and attack() functions

    fun shout() // Here comes the change
}
```

我们现在该做什么呢？

我们去更改六个不同类的实现，幸运的是只有六个而不是十六个。

# 桥接变化

根据你的看法，**桥接**设计模式可能类似于我们已讨论的**适配器**，或者我们在下一章将要讨论的**策略**。

桥接设计模式背后的想法是简化当前深度为三级的类层次结构：

```kt
Infantry --> Rifleman  --> Upgraded Rifleman                                                                           --> Light Rifleman             
         --> Grenadier --> Upgraded Grenadier       
                       --> Light Grenadier
```

为什么我们有这样一个复杂的层次结构？

这是因为我们有三个正交属性：武器类型、武器强度和移动速度。

也就是说，如果我们把那些属性传递给一个实现我们一直使用的相同接口的类的构造函数：

```kt
class Soldier(private val weapon: Weapon,
              private val legs: Legs) : Infantry {
    override fun attack(x: Long, y: Long) {
        // Find target
        // Shoot
        weapon.causeDamage()
    }

    override fun move(x: Long, y: Long) {
        // Compute direction
        // Move at its own pace
        legs.move()
    }
}
```

`Soldier`接收的属性应该是接口，这样我们就可以稍后选择它们的实现：

```kt
interface Weapon {
    fun causeDamage(): PointsOfDamage
}

interface Legs {
    fun move(): Meters
}
```

但`Meters`和`PointsOfDamage`是什么？它们是我们声明在某个地方的类或接口吗？

让我们短暂地偏离一下。

# 类型别名

首先，我们将看看它们是如何声明的：

```kt
typealias PointsOfDamage = Long
typealias Meters = Int
```

我们在这里使用了一个新的关键字，`typealias`。从现在起，我们可以使用`Meters`而不是普通的`Int`来从我们的`move()`方法返回。它们不是新类型。Kotlin 编译器在编译过程中始终会将`PointsOfDamage`转换为`Long`。使用它们提供了两个优点：

+   更好的语义，正如我们的情况。我们可以确切地知道我们返回的值的含义。

+   Kotlin 的主要目标之一是简洁。类型别名允许我们隐藏复杂的泛型表达式。我们将在接下来的章节中展开讨论。

# 你现在在军队里了

回到我们的`Soldier`类。我们希望它尽可能适应，对吧？他知道他可以移动或使用他的武器来做出更大的贡献。但他究竟会如何做呢？

我们完全忘记了实现这些部分！让我们从我们的武器开始：

```kt
class Grenade : Weapon {
    override fun causeDamage() = GRENADE_DAMAGE
}

class GrenadePack : Weapon {
    override fun causeDamage() = GRENADE_DAMAGE * 3
}

class Rifle : Weapon {
    override fun causeDamage() = RIFLE_DAMAGE
}

```

```kt
class MachineGun : Weapon {
    override fun causeDamage() = RIFLE_DAMAGE * 2
}
```

现在，让我们看看我们如何移动：

```kt
class RegularLegs : Legs {
    override fun move() = REGULAR_SPEED
}

class AthleticLegs : Legs {
    override fun move() = REGULAR_SPEED * 2
}
```

# 常量

我们将所有参数定义为常量：

```kt
const val GRENADE_DAMAGE : PointsOfDamage = 5L
const val RIFLE_DAMAGE = 3L
const val REGULAR_SPEED : Meters = 1
```

这些值非常有效，因为它们在编译时是已知的。

与 Java 中的`static final`变量不同，它们不能放在类内部。你应该将它们放在你的包的顶层，或者嵌套在`object`内部。

注意，尽管 Kotlin 有类型推断，但我们仍然可以明确指定常量的类型，甚至可以使用类型别名。那么，在你的代码中使用`DEFAULT_TIMEOUT : Seconds = 60`而不是

`DEFAULT_TIMEOUT_SECONDS = 60`如何？

# 一种致命的武器

剩下的就是让我们看到，在新层次结构中，我们仍然可以做完全相同的事情：

```kt
val rifleman = Soldier(Rifle(), RegularLegs())
val grenadier = Soldier(Grenade(), RegularLegs())
val upgradedGrenadier = Soldier(GrenadePack(), RegularLegs())
val upgradedRifleman = Soldier(MachineGun(), RegularLegs())
val lightRifleman = Soldier(Rifle(), AthleticLegs())
val lightGrenadier = Soldier(Grenade(), AthleticLegs())
```

现在，我们的层次结构看起来是这样的：

```kt
Infantry --> Soldier

Weapon --> Rifle
       --> MachineGun
       --> Grenade
       --> GrenadePack 

Legs --> RegularLegs
     --> AthleticLegs
```

更简单地进行扩展和理解。与之前讨论的一些其他设计模式不同，我们没有使用任何我们不了解的特殊语言特性，只是使用了一些工程最佳实践。

# 组合

你可能在这个部分结束时会有一种这种感觉，这个模式有点尴尬。这是因为它有一个灵魂伴侣，它的伴随模式，**迭代器**，我们将在下一章讨论。当两者结合时，它们才能真正发光。所以，如果你感到困惑，在你熟悉了**迭代器**之后，再回到这个模式。

话虽如此，我们可以开始分析这个模式。有一个**组合**设计模式可能看起来有点奇怪。毕竟，所有**结构模式**不都是关于组合对象吗？

与桥接设计模式的情况类似，名称可能并不反映其真正的优势。

# 一起

回到我们的策略游戏中，我们有一个新的概念：一个排。一个排由零个或多个步兵单位组成。这将是复杂数据结构的一个很好的例子。

这里是我们拥有的接口和类：

```kt
interface InfantryUnit

class Rifleman : InfantryUnit

class Sniper : InfantryUnit
```

你会如何实现它？我们将在下一节中看到。

# 排

`Squad`，显然，必须是一组步兵单位。所以，这应该很简单：

```kt
class Squad(val infantryUnits: MutableList<InfantryUnit> =         mutableListOf())
```

我们甚至设置了一个默认参数值，这样其他程序员就不需要传递他自己的士兵列表，除非他真的需要。

为了确保它能够工作，我们将创建三个士兵并将他们放入其中：

```kt
val miller = Rifleman()
val caparzo = Rifleman()
val jackson = Sniper()

val squad = Squad()

squad.infantryUnits.add(miller)
squad.infantryUnits.add(caparzo)
squad.infantryUnits.add(jackson)

println(squad.infantryUnits.size) // Prints 3
```

但第二天，Dave，那位程序员，带着新的要求来找我们。他认为添加士兵需要太多的代码行，甚至使用`mutableListOf()`传递它们。

他希望像这样初始化排：

```kt
val squad = Squad(miller, caparzo, jackson)
```

看起来不错，但究竟我们该如何做到这一点呢？

# 可变参数和次要构造函数

到目前为止，我们一直在使用类的默认构造函数。这是在类名之后声明的那个。但在 Java 中，我们可以为类定义多个构造函数。为什么 Kotlin 限制我们只能有一个？

实际上，并不是这样。我们可以使用`constructor`关键字为类定义次要构造函数：

```kt
class Squad(val infantryUnits: MutableList<InfantryUnit> = mutableListOf()) {
    constructor(first: InfantryUnit) : this(mutableListOf()) {
        this.infantryUnits.add(first)
    }

    constructor(first: InfantryUnit, 
                second: InfantryUnit) : this(first) {
        this.infantryUnits.add(second)
    }

    constructor(first: InfantryUnit, 
                second: InfantryUnit, 
                third: InfantryUnit) : 
        this(first, second) {
        this.infantryUnits.add(third)
    }
}
```

注意我们如何将一个构造函数委托给另一个：

```kt
    constructor(first: InfantryUnit) : this(mutableListOf()) {
    }                                     ⇑
                                          ⇑
    constructor(first: InfantryUnit,      ⇑ // Delegating
                second: InfantryUnit) : this(first) {    
    }
```

但这显然不是正确的方法，因为我们无法预测 Dave 可能传递给我们多少个元素。如果你来自 Java，你可能已经考虑过可变参数函数，它可以接受任意数量的相同类型的参数。在 Java 中，你会将参数声明为`InfantryUnit... units`。

Kotlin 为我们提供了`vararg`关键字来达到同样的目的。结合这两种方法，我们得到了以下这段不错的代码：

```kt
class Squad(val infantryUnits: MutableList<InfantryUnit> =     mutableListOf()) {

    constructor(vararg units: InfantryUnit) : this(mutableListOf()) {
        for (u in units) {
            this.infantryUnits.add(u)
        }
    }
}
```

# 计算子弹数量

游戏设计师在傍晚时分抓住你，当然是在你准备回家的那一刻。他想为整个排添加弹药计数，这样每个排都能报告它还剩下多少弹药：

```kt
fun bulletsLeft(): Long {
    // Do your job
}
```

有什么问题吗？

好吧，你看，狙击手有单独的子弹作为弹药：

```kt
class Bullet
```

步兵将子弹存放在弹匣中：

```kt
class Magazine(capacity: Int) {
    private val bullets = List(capacity) { Bullet() }
}
```

幸运的是，你的排里还没有*机枪手*，因为他们携带弹药在弹带上...

所以，你有一个可能嵌套也可能不嵌套的复杂结构。你需要对这个结构作为一个整体执行某些操作。这正是组合设计模式真正发挥作用的地方。

你看，这个名字有点令人困惑。组合（Composite）与其说是关于组合对象，不如说是关于将不同类型的对象作为同一棵树的节点来处理。为此，它们都应该实现相同的接口。

这可能一开始并不明显。毕竟，一个*步兵*显然不是一个*排*。但与其将接口视为一种*is-a*关系，我们更应该将其视为一种*能力提供者*。例如，Android 经常采用这种模式。

我们的能力是计数子弹：

```kt
interface CanCountBullets {
    fun bulletsLeft(): Int
}
```

`Squad`和`InfantryUnit`都应该实现此接口：

```kt
interface InfantryUnit : CanCountBullets

class Squad(...) : CanCountBullets {
    ...
}

class Magazine(...): CanCountBullets {
    ...
}
```

现在，由于每个人都有同样的能力，无论嵌套有多深，我们都可以要求顶层对象查询其下的一切。

`Magazine`和`Sniper`简单地计数它们包含的子弹。以下示例展示了我们如何跟踪`Magazines`中的子弹数量：

```kt
class Magazine(...): CanCountBullets {
    ...
    override fun bulletsLeft() = bullets.size
}
```

以下示例展示了我们如何跟踪狙击手`Sniper`拥有的子弹数量：

```kt
class Sniper(initalBullets: Int = 50) : InfantryUnit {
    private val bullets = List(initalBullets) { Bullet () }
    override fun bulletsLeft() = bullets.size
}
```

对于`Rifleman`，我们可以检查他们的`Magazines`并查看它们包含多少子弹：

```kt
class Rifleman(initialMagazines: Int = 3) : InfantryUnit {
    private val magazines = List<Magazine>(initialMagazines) {
        Magazine(5)
    }

    override fun bulletsLeft(): Int {
        return magazines.sumBy { it.bulletsLeft() }
    }
}
```

最后，对于小队，我们计算小队包含的所有单位的计数总和：

```kt
override fun bulletsLeft(): Int {
    return infantryUnits.sumBy { it.bulletsLeft() }
}
```

明天，当你的产品经理突然发现他需要实现一个排（这是一个小队的集合），你将装备齐全，准备就绪。

# 外观

在不同的实现和方案中，**外观**可能类似于**适配器**或**抽象工厂**。

它的目标似乎很简单——简化与另一个类或类族交互：

+   当我们想到*简化*时，我们通常想到**适配器**设计模式

+   当我们想到*类族*时，我们通常想到**抽象工厂**

那就是所有混淆通常都来自的地方。为了更好地理解它，让我们回到我们用于抽象工厂设计模式的示例。

# 保持简单

假设我们想要实现`loadGame()`方法。这个方法将需要一个我们已创建的文件（我们稍后会讨论），或者至少需要以下内容：

1.  至少需要创建两个总部（否则游戏已经胜利）

1.  每个总部都必须生产它拥有的建筑

1.  每个建筑都必须生产它拥有的单位

1.  所有单位都必须神奇地传送到游戏保存时它们所在的位置

1.  如果给单位下达了任何命令（比如*摧毁所有敌方基地*），它们应该继续执行这些命令

我们将在下一章讨论我们实际上如何通过**命令**设计模式向我们的单位下达命令，敬请期待。

现在，通常情况下，除非是**Minecraft (TM**)，否则不会只有一个人在制作游戏。还有那个其他的人，Dave，他处理所有的命令逻辑。他对建造建筑不太感兴趣。但在他的角色中，他也需要经常加载保存的游戏。

作为所有属于你的基地的开发者，你可以给他一套你写的指令，说明游戏应该如何正确加载。他可能会或可能不会遵循这套指令。也许他会忘记移动单位或建造建筑。游戏可能会崩溃。你可以使用外观设计模式来简化他的工作。

Dave 现在面临的主要问题是什么？

要加载游戏，他需要与至少三个不同的界面进行交互：

+   总部

+   建筑物

+   单位

他希望只有一个接口，类似于：

```kt
interface GameWorld
```

那正好是他需要的所有方法：

```kt
fun loadGame(file: File) GameWorld
```

嘿，但那看起来像是一个静态工厂方法！

是的，有时候，设计模式是相互嵌入的。我们使用静态工厂方法来创建我们的类，但它的目标是作为其他更复杂类的门面。使用门面并不意味着我们不向客户端暴露门面背后隐藏的接口。Dave 在游戏加载成功后仍然可以使用每个小单元发出命令。

简单，对吧？

# 轻量级模式

轻量级模式是一个没有任何状态的对象。这个名字来源于*非常轻*。

如果你已经阅读了前两章中的任意一章，你可能已经想到了一种应该非常轻的对象类型：`data`类。但`data`类完全是关于状态的。那么，它与轻量级设计模式有什么关系呢？

为了更好地理解这个模式，我们需要回顾二十年前。

回到 1994 年，当原始的《设计模式》一书出版时，你的普通 PC 有 4 MB 的 RAM。其中一个主要目标就是节省那宝贵的 RAM，因为你可以将其装入其中的内容是有限的。

现在，一些手机有 4 GB 的 RAM。当我们讨论轻量级设计模式时，请记住这个事实。

# 保守的做法

想象一下我们正在构建一个 2D 横向卷轴街机平台游戏。也就是说，你有你的角色，你可以用箭头键或游戏手柄来控制它。你的角色可以向左、向右移动，并且可以跳跃。

由于我们是一家由一名开发者（同时也是图形设计师、产品经理和销售代表）、两只猫和一只名叫迈克尔的金丝雀组成的小型独立公司，我们在游戏中只使用了十六种颜色。我们的角色高 64 像素，宽 64 像素。我们称他为**Maronic**，这也是我们游戏的名字。

我们的角色有很多敌人，主要由食肉性的坦桑尼亚蜗牛组成：

```kt
class TansanianSnail
```

由于它是一个 2D 游戏，每个蜗牛只有两个移动方向——向左和向右：

```kt
enum class Direction {
   LEFT,
   RIGHT
}
```

每个蜗牛持有一对图像和一个方向：

```kt
class TansanianSnail() {
   val directionFacing = Direction.LEFT

   val sprites = listOf(java.io.File("snail-left.jpg"), 
                        java.io.File("snail-right.jpg"))
}
```

我们可以获取当前显示蜗牛面向哪个方向的精灵：

```kt
    fun TansanianSnail.getCurrentSprite() : java.io.File {
        return when (directionFacing) {
            Direction.LEFT -> sprites[0]
            Direction.RIGHT -> sprites[1]
        }
    }
```

我们可以在屏幕上绘制它：

```kt
      _____
  \|_\___  \
  /________/    <-- With a bit of imagination you'll see it's a snail
```

但当它移动时，它基本上只是向左或向右滑动。我们希望有三个动画精灵来重现蜗牛的动作：

```kt
---------------------------------------------------------------------
     _____ |    _____ |    _____ |    _____ |    _____ |    _____        
 _|_/___∂ \|\|_/___∂ \|\/_/___∂ \|\|_/___∂ \|\|_/___∂ \|\|_/___∂ \
 /____\___/| \___\___/|/____\___/|/____\___/|/____\___/|/____\___/
---------------------------------------------------------------------
left-3      left-2     left-1     right-1    right-2    right-3      
```

要在我们的代码中实现它：

```kt
    val sprites = List(8) { i ->
              java.io.File(when {
                        i == 0 -> "snail-left.jpg"
                        i == 1 -> "snail-right.jpg"
                        i in 2..4 -> "snail-move-left-${i-1}.jpg"
                        else -> "snail-move-right${(4-i)}.jpg"
                    })
            }
```

我们通过传递一个`block`函数作为构造函数初始化一个包含八个元素的列表。对于每个元素，我们决定获取哪个图像：

+   位置 0 和 1 用于静止图像

+   位置 2 到 4 用于向左移动

+   位置 5 到 7 用于向右移动

现在让我们来做一些数学计算。每个蜗牛是一个 64 x 64 的图像。假设每种颜色正好占用一个字节，单个图像在内存中占用 4 KB 的 RAM。由于一个蜗牛有八个图像，我们需要为每个蜗牛分配 32 KB 的 RAM，这样我们才能在 1 MB 的内存中容纳 32 只蜗牛。

由于我们希望在屏幕上拥有成千上万这种危险且极快的生物，并且能够在十年前的手机上运行我们的游戏，因此我们显然需要一个更好的解决方案。

# 节省内存

我们蜗牛的问题是什么？它们实际上相当胖，是重量级的蜗牛。

我们希望把它们放在节食中。每个蜗牛在其蜗牛身体内存储了八张图片。但实际上，这些图片对每个蜗牛来说都是相同的。

如果我们提取那些精灵：

```kt
val sprites = List(8) { i ->
    java.io.File(when {
        i == 0 -> "snail-left.jpg"
        i == 1 -> "snail-right.jpg"
        i in 2..4 -> "snail-move-left-${i-1}.jpg"
        else -> "snail-move-right${(4-i)}.jpg"
    })
}
```

然后我们每次都把这个列表传递给`getCurrentSprite()`函数：

```kt
fun TansanianSnail.getCurrentSprite(sprites: List<java.io.File>) :     java.io.File { ... }
```

这样，无论我们生成多少蜗牛，我们只会消耗 256 KB 的内存。我们可以生成数百万个，而不会影响我们程序的大小。

当然，我们应该担心我们传递的数据的不可变性。这意味着在任何时候，我们都不能将`null`分配给我们的`sprites`变量，如下所示：

```kt
sprites = null
```

这样会产生`NullPointerException`。而且，如果有人要`clear()`这个列表，那将会是灾难性的：

```kt
sprites.clear()
```

幸运的是，Kotlin 为我们处理了这个问题。因为我们使用了`val`，列表被精确地分配了一次。而且，因为我们使用了 List，它产生了一个不可变列表，这个列表不能被更改或清除。

所有的前述行甚至无法编译：

```kt
sprites.clear()
sprites[0] = File("garbage")
sprites[0] = null
```

即使现在内存很充足，你仍然可以争论这种模式的有用性。但，正如我们之前所说的，工具箱中的工具并不占多少空间，拥有另一个设计模式可能仍然是有用的。

# 代理

这是一个行为不当的设计模式。就像装饰者一样，它扩展了对象的功能。但是，与总是按指示行事的装饰者不同，拥有一个**代理**可能意味着当被要求时，对象会做完全不同的事情。

# 简单地进入 RMI 世界

在讨论代理时，许多资料，尤其是与 Java 相关的资料，会转向讨论另一个概念，RMI。

在 JVM 世界中，RMI 代表远程方法调用，这是一种远程过程调用（RPC）。这意味着你能够调用一些不在你本地机器上，而是在某个远程机器上的代码。

虽然这是一个非常聪明的解决方案，但它非常特定于 JVM，在微服务时代已经变得不那么流行了，因为在微服务时代，每一块代码可能都是用完全不同的编程语言编写的。

# 一个替代方案

当我们讨论创建型模式时，我们已经讨论了*昂贵*对象的想法。例如，访问网络资源的对象，或者创建需要花费很多时间的对象。

在**Funny Cat App**（由金丝雀 Michael 发明；还记得他从 Flyweight 模式中吗？）我们每天为用户提供有趣的猫图片。在我们的主页和移动应用程序上，每个用户都会看到很多有趣的猫缩略图。当他点击或触摸任何这些图片时，它就会扩展到全屏的辉煌。

但在网络中获取猫的图片非常昂贵，并且消耗大量内存，尤其是如果这些是那些在晚餐后倾向于再来一份甜点的猫的图片。所以，我们想要做的是拥有一个智能对象，某种可以自我管理的对象。

当第一个用户访问此图像时，它将通过网络获取。没有避免的方法。

但是当它第二次被访问时，无论是通过这个还是其他用户，我们都希望避免再次访问网络，而是返回之前缓存的那个结果。这就是我们所说的 *行为不当* 部分。与每次都通过网络访问的预期行为不同，我们有点懒，返回我们已经准备好的结果。这有点像走进一家便宜的小餐馆，点了一份汉堡，两分钟后就拿到了，但却是冷的。嗯，这是因为有人不喜欢洋葱，所以之前就把它退回厨房了。这是真的。

这听起来像很多逻辑。但是，正如你可能猜到的，特别是在遇到装饰器设计模式之后，Kotlin 可以通过减少你需要编写的样板代码来达到你的目标，从而创造奇迹：

```kt
data class CatImage(private val thumbnailUrl: String, 
                    private val url: String) {
    val image: java.io.File by lazy {
        // Actual fetch goes here
    }
}
```

如你所注意到的，我们使用 `by` 关键字将此字段的初始化委托给一个名为 `lazy` 的标准函数。

第一次调用 `image` 将会执行我们的代码块并将结果保存到 `image` 属性中。

有时，代理设计模式被分为三个子模式：

+   虚拟代理：懒加载结果

+   远程代理：向远程资源发出调用

+   保护或访问控制代理：拒绝未经授权的访问

根据你的观点，你可以将我们的示例视为虚拟代理或虚拟和远程代理的组合。

# 懒加载委托

你可能会想知道如果两个线程同时尝试初始化图像会发生什么。默认情况下，`lazy()` 函数是同步的。只有一个线程会获胜，其他线程将等待图像准备好。

如果你不在乎两个线程执行 `lazy` 块（例如，它并不那么昂贵），你可以使用 `by lazy(LazyThreadSafetyMode.PUBLICATION)`。

如果你绝对需要性能，并且你绝对确信两个线程永远不会同时执行相同的块，你可以使用 `LazyThreadSafetyMode.NONE`。

# 概述

在本章中，我们学习了结构型设计模式如何帮助我们创建更灵活的代码，这些代码可以轻松适应变化，有时甚至可以在运行时进行。我们已经介绍了 Kotlin 中的操作符重载及其局限性。你应该知道如何使用 `typealias` 创建类型名的快捷方式，以及如何使用 `const` 定义有效的常量。

我们已经介绍了在 Kotlin 中如何通过实现相同的接口并使用 `by` 关键字来实现委托给另一个类的工作方式。

此外，我们还介绍了可以使用 `vararg` 接收任意数量参数的函数，以及使用 `lazy` 进行懒初始化。

在下一章中，我们将讨论经典设计模式的第三组：行为模式。
