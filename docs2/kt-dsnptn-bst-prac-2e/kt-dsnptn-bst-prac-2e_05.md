# 第四章：*第四章*：熟悉行为模式

本章将基于 Kotlin 讨论行为模式。**行为模式**处理对象之间如何相互交互。

我们将学习一个对象如何根据情况改变其行为，对象如何在不了解彼此的情况下进行通信，以及如何轻松地遍历复杂结构。我们还将简要介绍 Kotlin 中的函数式编程概念，这将帮助我们轻松实现一些这些模式。

在本章中，我们将涵盖以下主题：

+   策略

+   迭代器

+   状态

+   命令

+   责任链

+   解释器

+   调解者

+   备忘录

+   访问者

+   模板方法

+   观察者

到本章结束时，您将能够以高度解耦和灵活的方式组织代码。

# 技术要求

除了前几章的要求外，您还需要一个**Gradle**启用的**Kotlin**项目，以便能够添加所需的依赖项。

您可以在此处找到本章的源代码：[`github.com/PacktPublishing/Kotlin-Design-Patterns-and-Best-Practices/tree/main/Chapter04`](https://github.com/PacktPublishing/Kotlin-Design-Patterns-and-Best-Practices/tree/main/Chapter04)。

# 策略

**策略**设计模式的目标是允许对象在运行时改变其行为。

让我们回顾一下我们在*第三章*中设计的平台游戏，在讨论**外观**设计模式时，*理解结构模式*。

加那利迈克尔，在我们这家小型独立游戏开发公司担任游戏设计师，想出了一个绝妙的主意。*如果我们给我们的英雄配备一套武器来保护我们免受那些可怕的肉食性蜗牛的伤害会怎么样呢？*

武器都会朝英雄面对的方向发射弹丸（你不想离那些危险的蜗牛太近）：

```kt
enum class Direction { 
```

```kt
    LEFT, RIGHT 
```

```kt
}
```

所有弹丸都应该有一对坐标（*记住，我们的游戏是 2D 的*）和一个方向：

```kt
data class Projectile(private var x: Int, 
```

```kt
                      private var y: Int, 
```

```kt
                      private var direction: Direction)
```

如果我们只发射一种类型的弹丸，那会很简单，因为我们已经在*第二章*中介绍了**工厂模式**，*使用创建型模式*。

我们可以在这里做类似的事情：

```kt
class OurHero { 
```

```kt
    private var direction = Direction.LEFT 
```

```kt
    private var x: Int = 42 
```

```kt
    private var y: Int = 173 
```

```kt
    fun shoot(): Projectile { 
```

```kt
        return Projectile(x, y, direction) 
```

```kt
    } 
```

```kt
}
```

但迈克尔希望我们的英雄至少拥有三种不同的武器：

+   **豌豆射手**：发射直线飞行的豌豆。我们的英雄一开始就有它。

+   **石榴**：击中敌人时会爆炸，就像手榴弹一样。

+   **香蕉**：当它到达屏幕末端时会像回旋镖一样返回。

*迈克尔，快点，给我们点空间！你不能就只坚持使用那些都一样的普通枪吗？*

## 水果武器库

首先，让我们讨论一下我们如何用 Java 的方式解决这个问题。

在 Java 中，我们会创建一个接口来抽象这些变化。在我们的情况下，变化的是英雄的武器：

```kt
interface Weapon { 
```

```kt
    fun shoot(x: Int, 
```

```kt
              y: Int, 
```

```kt
              direction: Direction): Projectile 
```

```kt
}
```

然后，所有其他武器都将实现这个接口。由于我们不处理诸如渲染或动画对象等方面，这里不会实现特定的行为：

```kt
// Flies straight 
```

```kt
class Peashooter : Weapon {
```

```kt
    override fun shoot(
```

```kt
        x: Int,
```

```kt
        y: Int,
```

```kt
        direction: Direction
```

```kt
    ) = Projectile(x, y, direction)
```

```kt
}
```

```kt
// Returns back after reaching end of the screen
```

```kt
class Banana : Weapon {
```

```kt
    override fun shoot(
```

```kt
        x: Int,
```

```kt
        y: Int,
```

```kt
        direction: Direction
```

```kt
    ) = Projectile(x, y, direction)
```

```kt
}
```

```kt
// Other similar implementations here 
```

我们游戏中的所有武器都将实现相同的接口，并覆盖其单个方法。

我们的英雄一开始会持有一把武器的引用，`Peashooter`：

```kt
private var currentWeapon: Weapon = Peashooter()
```

这个引用将实际射击过程委托给它：

```kt
fun shoot(): Projectile = currentWeapon.shoot(x, y,   direction)
```

剩下的就是能够装备另一件“武器”：

```kt
fun equip(weapon: Weapon) { 
```

```kt
    currentWeapon = weapon 
```

```kt
}
```

这就是**策略**设计模式的核心所在。它使得我们的算法——在这个例子中，是我们游戏中的武器——可以互换。

## 市民函数

使用 Kotlin，有更有效的方法来实现相同的功能，同时使用更少的类。这要归功于 Kotlin 中函数是第一类公民的事实。*但这意味着什么呢？*

首先，我们可以将函数赋值给我们的类的变量，就像任何其他标准值一样。将原始值赋给变量是有意义的：

```kt
val x = 7
```

你也可以像我们已经多次做的那样，将一个对象赋值给一个变量：

```kt
var myPet = Canary("Michael")
```

*那么，为什么你不能将一个函数赋值给变量呢？*

在 Kotlin 中，你可以轻松做到这一点。以下是一个例子：

```kt
val square = fun(x: Int): Long { 
```

```kt
    return (x * x).toLong() 
```

```kt
}
```

让我们看看这可能如何帮助我们简化我们的设计。

首先，我们将为所有武器定义一个命名空间。我们可以使用一个对象来做到这一点。这不是强制性的，但它有助于保持一切井然有序。然后，而不是类，我们的每件武器都将成为一个函数：

```kt
object Weapons {
```

```kt
    // Flies straight
```

```kt
    fun peashooter(x: Int, y: Int, direction: Direction): 
```

```kt
        Projectile {
```

```kt
        return Projectile(x, y, direction)
```

```kt
    }
```

```kt
    // Returns back after reaching end of the screen
```

```kt
    fun banana(x: Int, y: Int, direction: Direction): 
```

```kt
        Projectile {
```

```kt
        return Projectile(x, y, direction)
```

```kt
    }
```

```kt
    // Other similar implementations here   
```

```kt
}  
```

正如你所见，我们没有实现接口，而是有多个函数接收相同的参数并返回相同的对象。

最有趣的部分是我们的英雄。`OurHero` 类现在包含两个值，都是函数：

```kt
class OurHero { 
```

```kt
    var currentWeapon = Weapons::peashooter 
```

```kt
    val shoot = fun() { 
```

```kt
        currentWeapon(x, y, direction) 
```

```kt
    } 
```

```kt
}
```

可以互换的部分是 `currentWeapon`，而 `shoot` 现在是一个包装它的匿名函数。

为了测试我们的想法是否可行，我们可以先射击默认武器一次，然后切换到另一件武器并再次射击：

```kt
val hero = OurHero()
```

```kt
hero.shoot()
```

```kt
hero.currentWeapon = Weapons::banana
```

```kt
hero.shoot()
```

注意，这大大减少了我们需要编写的类的数量，同时保持了相同的功能。如果你的可互换算法没有状态，你可以用简单的函数来替换它。否则，引入一个接口，并让每个策略模式实现它。

这也是我们第一次使用函数引用操作符 `::`。这个操作符允许我们将函数当作变量来引用，而不是调用它。

**策略**是一个非常有价值的模式，当你的应用程序需要在运行时改变其行为时。一个例子是航班预订系统，允许超售；也就是说，将比座位更多的乘客放在航班上。你可能会决定你希望在航班前一天允许超售，然后禁止。你可以通过切换策略而不是在代码中添加复杂的检查来实现这一点。

现在，让我们看看另一个可以帮助我们处理复杂数据结构的模式。

# 迭代器

当我们在上一章讨论**组合**设计模式时，我们注意到这个设计模式感觉有点不完整。现在是时候让出生时分开的双胞胎重聚了。就像阿诺德·施瓦辛格和丹尼·德维托一样，他们非常不同，但彼此很好地补充。

如您从上一章所记得的，一个小队由士兵或其他小队组成。现在让我们创建一个：

```kt
val platoon = Squad(
```

```kt
    Trooper(),
```

```kt
    Squad(
```

```kt
        Trooper(),
```

```kt
    ),
```

```kt
    Trooper(),
```

```kt
    Squad(
```

```kt
        Trooper(),
```

```kt
        Trooper(),
```

```kt
    ),
```

```kt
    Trooper()
```

```kt
)
```

在这里，我们创建了一个由四个士兵组成的小队。

如果我们能够使用我们之前在*第一章*中学习的`for-each`循环打印出这个小队中的所有士兵，那将是有用的。

让我们试着写这段代码，看看会发生什么：

```kt
for (trooper in platoon) {
```

```kt
    println(trooper)
```

```kt
}
```

虽然这段代码无法编译，但 Kotlin 编译器为我们提供了一个有用的提示：

```kt
>For loop range must have an iterator method 
```

在我们遵循编译器的指导并实现方法之前，让我们简要讨论一下目前我们面临的问题。

我们的小队实现了组合设计模式，它不是一个扁平的数据结构。它可以包含包含其他对象的对象——小队可以包含士兵以及其他小队。然而，在这种情况下，我们想要抽象这种复杂性，并像处理一个士兵列表一样处理它。迭代器模式正是这样做的——它将我们的复杂数据结构简化为一个简单的元素序列。元素的顺序和要忽略的元素由迭代器决定。

要在`for-each`循环中使用我们的`Squad`对象，我们需要实现一个特殊函数，称为`iterator()`。由于这是一个特殊函数，我们需要使用`operator`关键字：

```kt
operator fun iterator() = ...
```

我们函数返回的是一个实现了`Iterator<T>`接口的匿名对象：

```kt
operator fun iterator() = object: Iterator<Trooper> {
```

```kt
    override fun hasNext(): Boolean {
```

```kt
        // Are there more objects to iterate over?
```

```kt
    }
```

```kt
    override fun next(): Trooper {
```

```kt
        // Return next Trooper
```

```kt
    }
```

```kt
}
```

再次强调，我们可以看到 Kotlin 中泛型的使用。`Iterator<Trooper>`意味着我们的`next()`方法返回的对象将始终是`Trooper`类型。

为了能够迭代所有元素，我们需要实现两个方法——一个用于获取下一个元素，另一个让循环知道何时停止。让我们通过执行以下步骤来实现：

1.  首先，我们需要为我们的迭代器提供一个状态。它将记住最后一个返回的元素：

    ```kt
    operator fun iterator() = object: Iterator<Trooper> { 
        private var i = 0 
        // More code here 
    }
    ```

1.  接下来，我们需要告诉它何时停止。在简单的情况下，这等于底层数据结构的大小：

    ```kt
    override fun hasNext(): Boolean { 
        return i < units.size 
    }
    ```

    这将会有点复杂，因为我们需要处理一些边缘情况。你可以在本书的 GitHub 仓库中找到完整的实现。

1.  最后，我们需要知道要返回哪个单位。在简单的情况下，我们只需返回当前元素，并将元素计数增加一个：

    ```kt
    override fun next() = units[i++]
    ```

    在我们的情况下，这要复杂一些，因为小队可以包含其他小队。再次，你可以在本书的 GitHub 仓库中找到完整的实现。

    有时候，将迭代器作为函数的参数也是有意义的：

    ```kt
    fun <T> printAnything(iter: Iterator<T>) { 
        while (iter.hasNext()) { 
            println(iter.next()) 
        } 
    }
    ```

    这个函数将遍历任何提供迭代器的对象。这也是 Kotlin 中泛型函数的一个例子。注意函数名称前的 `<T>`。

作为一名普通的开发者，你不会为了谋生而发明新的数据结构，你可能不会经常实现迭代器。然而，了解它们在幕后是如何工作的仍然很重要。

下一个部分将展示如何有效地设计有限状态机。

# 状态

你可以把状态设计模式看作是一个有偏见的策略模式，我们在本章开头讨论过。但是，虽然策略模式通常是由客户端从外部替换的，状态可能会根据它接收到的输入而内部改变。

看看客户使用策略模式编写的这个对话：

+   **客户端**：*这里有一件新的事情要做，从现在开始做吧。*

+   **策略**：*好的，没问题。*

+   **客户端**：*我喜欢你的地方是你从不会和我争论。*

与此相比：

+   **客户端**：*这里有一些我从你那里得到的新输入。*

+   **状态**：*哦，我不知道。也许我会开始做一些不同的事情。也许不会。*

客户端还应预期状态可能会拒绝其一些输入：

+   **客户端**：*这里有一些东西让你思考，状态。*

+   **状态**：*我不知道那是什么！你没看到我很忙吗？去找策略处理这件事吧！*

*那么，客户为什么还要容忍我们这种状态呢？* 好吧，状态擅长控制一切。

## 状态的五十种色彩

我们平台游戏中的肉食性蜗牛已经受够了这种虐待。所以，玩家向它们扔豌豆和香蕉，结果只是到达另一个可怜的城堡。*现在，它们要采取行动了！*

让我们看看状态设计模式如何帮助我们模拟一个角色的行为变化——在我们的案例中，是平台游戏中的敌人。默认情况下，蜗牛应该静止不动以节省蜗牛的能量。但是当英雄靠近时，它应该积极地冲向他们。

如果英雄设法伤害了它，它应该撤退舔舐伤口。然后，它将重复攻击直到其中一方死亡。

首先，我们将声明蜗牛生命中可能发生的事情：

```kt
interface WhatCanHappen { 
```

```kt
    fun seeHero() 
```

```kt
    fun getHit(pointsOfDamage: Int) 
```

```kt
    fun calmAgain() 
```

```kt
}
```

我们的蜗牛实现了这个接口，以便在它可能发生的事情上得到通知并相应地行动：

```kt
class Snail : WhatCanHappen { 
```

```kt
    private var healthPoints = 10 
```

```kt
    override fun seeHero() { 
```

```kt
    } 
```

```kt
    override fun getHit(pointsOfDamage: Int) { 
```

```kt
    } 
```

```kt
    override fun calmAgain() { 
```

```kt
    } 
```

```kt
}
```

现在，我们可以声明一个 `Mood` 类，我们将用 `sealed` 关键字标记它：

```kt
sealed class Mood { 
```

```kt
   // Some abstract methods here, like draw(), for example 
```

```kt
}
```

**密封类**是抽象的，不能被实例化。我们将在稍后看到使用它们的优点。但在那之前，让我们声明其他状态：

```kt
object Still : Mood()
```

```kt
object Aggressive : Mood()
```

```kt
object Retreating : Mood()
```

```kt
object Dead : Mood()
```

这些都是我们蜗牛的不同状态——抱歉，心情。

在状态设计模式中，`Snail` 是上下文。它持有状态。因此，我们为它声明一个成员：

```kt
class Snail : WhatCanHappen { 
```

```kt
    private var mood: Mood = Still 
```

```kt
    // As before 
```

```kt
}
```

现在，让我们定义当蜗牛看到我们的英雄时 `Snail` 应该做什么：

```kt
override fun seeHero() {
```

```kt
    mood = when(mood) {
```

```kt
        is Still -> Aggressive
```

```kt
    }
```

```kt
}
```

注意，这不能编译。这就是`sealed`类发挥作用的地方。就像`enum`一样，Kotlin 知道从它扩展的类数量是有限的。因此，它要求我们的`when`是详尽的，并指定其中的所有不同情况。

重要提示：

如果你使用 IntelliJ 作为你的 IDE，它甚至会建议你自动`添加剩余的分支`。

我们可以使用`else`来描述没有状态变化：

```kt
override fun seeHero() {
```

```kt
    mood = when(mood) {
```

```kt
        is Still -> Aggressive
```

```kt
        else -> mood
```

```kt
    }
```

```kt
}
```

当蜗牛被击中时，我们需要判断它是死是活。为此，我们可以使用不带参数的`when`：

```kt
override fun getHit(pointsOfDamage: Int) {
```

```kt
    healthPoints -= pointsOfDamage
```

```kt
    mood = when {
```

```kt
        (healthPoints <= 0) -> Dead
```

```kt
        mood is Aggressive -> Retreating
```

```kt
        else -> mood
```

```kt
    }
```

```kt
}
```

注意，我们在这里使用了`is`关键字，这与 Java 中的`instanceof`相同，但更简洁。

## 国家状况

之前的方法包含了我们上下文的大多数逻辑。你有时可能会看到不同的方法，这是有效的，因为你的上下文变得更大。

在这种方法中，`Snail`会变得很瘦：

```kt
class Snail { 
```

```kt
    internal var mood: Mood = Still(this) 
```

```kt
    private var healthPoints = 10 
```

```kt
    // That's all! 
```

```kt
}
```

注意，我们将`mood`标记为`internal`。这允许同一包中的其他类修改它。而不是让`Snail`实现`WhatCanHappen`，我们的`Mood`将实现它：

```kt
sealed class Mood : WhatCanHappen
```

现在，逻辑位于我们的状态对象中：

```kt
class Still(private val snail: Snail) : Mood() {
```

```kt
    override fun seeHero() {
```

```kt
        snail.mood = Aggressive
```

```kt
    }
```

```kt
    override fun getHit(pointsOfDamage: Int) {
```

```kt
        // Same logic from before
```

```kt
    }
```

```kt
    override fun calmAgain() {   
```

```kt
       // Return to Still state
```

```kt
    }
```

```kt
}
```

注意，我们的状态对象现在在构造函数中接收对其上下文的引用。

如果你的状态代码量相对较小，请使用第一种方法。如果变体差异很大，请使用第二种方法。一个现实世界的例子，其中广泛使用此模式，是 Kotlin 的**协程**机制。我们将在*第五章*，*介绍函数式编程*中详细讨论。

现在，让我们看看另一个封装动作的模式。

# 命令

这种设计模式允许你将动作封装在对象中，稍后执行。此外，如果我们可以在稍后执行一个动作，我们也可以执行多个，甚至可以精确地安排何时执行它们。

让我们回到我们的`Stormtrooper`管理系统，从*第三章*，*理解结构型模式*。这里是一个实现之前提到的`attack`和`move`函数的例子：

```kt
class Stormtrooper(...) { 
```

```kt
    fun attack(x: Long, y: Long) { 
```

```kt
        println("Attacking ($x, $y)") 
```

```kt
        // Actual code here 
```

```kt
    } 
```

```kt
    fun move(x: Long, y: Long) { 
```

```kt
        println("Moving to ($x, $y)") 
```

```kt
        // Actual code here 
```

```kt
    } 
```

```kt
}
```

我们甚至可以使用上一章中提到的**桥接**设计模式来提供实际的实现。

我们现在需要解决的问题是我们的小兵只能记住一个命令。仅此而已。如果他们从`(0, 0)`开始，这是屏幕的顶部，我们可以告诉他们`move(20, 0)`，即向右移动 20 步，然后`move(20, 20)`。在这种情况下，他们会直接移动到`(20, 20)`，并可能被摧毁，因为我们必须不惜一切代价避免那些叛军：

```kt
storm trooper -> good direction  -> (20, 0) 
```

```kt
          [rebel] [rebel]                     
```

```kt
        [rebel] [rebel] [rebel]               
```

```kt
           [rebel] [rebel] 
```

```kt
            (5, 20)                        (20, 20)
```

如果你从本书的开头开始跟随，或者至少在*第三章*，*理解结构型模式*中加入了进来，你可能已经有一个想法我们需要做什么，因为我们已经讨论了语言中*函数作为一等公民*的概念。

让我们为这个草拟一个草案。我们知道我们想要持有对象列表，但我们还不知道它们应该是哪种类型。所以，我们现在将使用`Any`：

```kt
class Trooper { 
```

```kt
    private val orders = mutableListOf<Any>()  
```

```kt
    fun addOrder(order: Any) {
```

```kt
        this.orders.add(order)
```

```kt
    } 
```

```kt
    // More code here 
```

```kt
}
```

然后，我们想要遍历列表并执行我们拥有的命令：

```kt
class Trooper { 
```

```kt
    ... 
```

```kt
    // This will be triggered from the outside once in a while
```

```kt
    fun executeOrders() {
```

```kt
        while (orders.isNotEmpty()) {
```

```kt
            val order = orders.removeFirst()
```

```kt
            order.execute() // Compile error for now 
```

```kt
        }
```

```kt
    } 
```

```kt
    ... 
```

```kt
}
```

注意，Kotlin 为我们提供了`isNotEmpty()`函数，用于集合，作为`!orders.isEmpty()`检查的替代，以及一个`removeFirst()`函数，它允许我们像使用队列一样使用我们的集合。

即使你不熟悉命令设计模式，你也可以猜到，如果我们想让我们的代码编译，我们可以定义一个只有一个方法`execute()`的接口：

```kt
interface Command { 
```

```kt
    fun execute() 
```

```kt
}
```

然后，我们可以在成员属性中同时持有列表：

```kt
private val commands = mutableListOf<Command>()
```

每种类型的命令，无论是移动命令还是攻击命令，都需要根据需要实现这个接口。这基本上就是 Java 实现这种模式的大多数情况下的建议。*但是有没有更好的方法呢？*

让我们再次看看`Command`。`execute()`方法不接受任何东西，也不返回任何东西，只是做些事情。这就像写以下代码：

```kt
fun command(): Unit { 
```

```kt
    // Some code here 
```

```kt
}
```

这与我们之前看到的不同。我们可以进一步简化这个：

```kt
() -> Unit
```

而不是有一个名为`Command`的接口，我们将使用`typealias`：

```kt
typealias Command = ()-> Unit
```

这使得我们的`Command`接口变得冗余，并允许我们移除它。

现在，这一行再次停止编译：

```kt
command.execute() // Unresolved reference: execute 
```

这是因为`execute()`只是我们发明的一个名字。在 Kotlin 中，函数使用`invoke()`：

```kt
command.invoke() // Compiles
```

我们还可以省略`invoke()`，这将给我们以下代码：

```kt
fun executeOrders() {
```

```kt
    while (orders.isNotEmpty()) {
```

```kt
        val order = orders.removeFirst()
```

```kt
        order() // Executed the next order
```

```kt
    }
```

```kt
}
```

这很好，但当前，我们的函数没有任何参数。*如果我们的函数接收参数会发生什么？*

一个选择是改变我们的`Command`签名，以便我们接收两个参数：

```kt
(x: Int, y: Int)-> Unit
```

*但是，如果某些命令没有接收任何参数，或者只接收一个，或者接收超过两个参数怎么办？* 我们还需要记住在每一步传递给`invoke()`的内容。

一个更好的方法是有一个**函数生成器**。这是一个返回另一个函数的函数。如果你曾经使用过 JavaScript 语言，那么你会知道使用闭包来限制作用域和记住东西是一种常见的做法。我们在这里也会这样做：

```kt
val moveGenerator = fun(trooper: Trooper, 
```

```kt
                        x: Int, 
```

```kt
                        y: Int): Command { 
```

```kt
    return fun() { 
```

```kt
        trooper.move(x, y) 
```

```kt
    } 
```

```kt
}
```

当使用适当的参数调用时，`moveGenerator`将返回一个新的函数。这个函数可以在我们觉得合适的时候被调用，并且它会记住三件事：

+   要调用哪个方法

+   要使用哪些参数

+   在哪个对象上使用它

现在，我们的`Trooper`可能有一个这样的方法：

```kt
fun appendMove(x: Int, y: Int) = apply { 
```

```kt
    commands.add(moveGenerator(this, x, y)) 
```

```kt
}
```

这为我们提供了一个很好的流畅语法：

```kt
val trooper = Trooper() 
```

```kt
trooper.appendMove(20, 0) 
```

```kt
    .appendMove(20, 20) 
```

```kt
    .appendMove(5, 20) 
```

```kt
    .execute()
```

**流畅语法**意味着我们可以轻松地在同一对象上链式调用方法，而无需多次重复其名称。

这段代码将打印以下输出：

```kt
> Moving to (20, 0) 
```

```kt
> Moving to (20, 20) 
```

```kt
> Moving to (5, 20)
```

现在，我们可以向我们的`Trooper`发出任意数量的命令，而无需了解它们是如何在内部执行的。

接收或返回另一个函数的函数被称为**高阶函数**。在这本书中，我们将多次探索这样的函数。

## 撤销命令

虽然不是直接相关，但命令设计模式的一个优点是能够撤销命令。*如果我们想支持这样的功能会怎样呢？*

撤销操作通常非常棘手，因为它涉及到以下之一：

+   返回到之前的状态（如果有多个客户端，这是不可能的，因为这需要大量的内存）

+   计算增量（实现起来很棘手）

+   定义相反的操作（不一定总是可能的）

在我们的情况下，与从(0,0)移动到(0, 20)的命令相反的命令将是*从你现在所在的位置移动到(0,0)*。这可以通过存储一对命令来实现：

```kt
private val commands =   mutableListOf<Pair<Command, Command>>()
```

我们需要修改我们的`appendMove`函数，使其每次也存储反向命令：

```kt
fun appendMove(x: Int, y: Int) = apply { 
```

```kt
    val oppositeMove = /* If it's the first command,       generate move to current location. Otherwise, get the       previous command */ 
```

```kt
    commands.add(moveGenerator(this, x, y) to oppositeMove) 
```

```kt
}
```

计算相反的移动相当复杂，因为我们没有保存我们的士兵当前的位置（这本来就是我们应该实现的）。我们还将不得不处理一些边缘情况。但这应该能给你一个这样的行为是如何实现的思路。

**命令**设计模式是功能已经嵌入到语言中的另一个例子。在这种情况下，它作为一个一等公民存在，这减少了你自己实现设计模式的需求。在现实世界中，当你想要排队多个动作或安排稍后执行的动作时，这个模式是实用的。

# 职责链

我是一个糟糕的软件架构师，我不太喜欢与人交谈。因此，当我坐在*象牙塔*（这是我经常去的咖啡馆的名字）时，我写了一个小的网络应用程序。如果一个开发者有问题，他们不应该直接找我，哦不！他们需要通过这个系统给我一个适当的请求，而我只有在我认为他们的请求有价值时才会回答他们。

**过滤器链**是网络服务器中的一个普遍概念。通常，当一个请求到达你这里时，预期以下情况是真实的：

+   它的参数已经被验证过了。

+   如果可能的话，用户已经被认证了。

+   用户角色和权限已知，并且用户有权执行操作。

所以，我最初写的代码看起来像这样：

```kt
data class Request(val email: String, val question: String)
```

```kt
fun handleRequest(r: Request) {
```

```kt
    // Validate 
```

```kt
    if (r.email.isEmpty() || r.question.isEmpty()) {
```

```kt
        return
```

```kt
    }
```

```kt
    // Authenticate 
```

```kt
    // Make sure that you know whos is this user 
```

```kt
    if (r.isKnownEmail()) {
```

```kt
        return
```

```kt
    }
```

```kt
    // Authorize 
```

```kt
    // Requests from juniors are automatically ignored by 
```

```kt
       architects 
```

```kt
    if (r.isFromJuniorDeveloper()) {
```

```kt
        return
```

```kt
    }
```

```kt
    println("I don't know. Did you check StackOverflow?")
```

```kt
}
```

它有点乱，但它是有效的。

然后，我注意到一些开发者决定他们可以一次性问我两个问题。我们必须给这个函数添加一些更多的逻辑。但是等等——毕竟，我是一个架构师。*那么，有没有更好的方法来委派这个任务呢？*

**职责链**设计模式的目标是将一个复杂的逻辑部分分解成一系列较小的步骤，其中每个步骤，或链中的链接，决定是否继续到下一个步骤或返回一个结果。

这次，我们不会学习新的 Kotlin 技巧，而是使用我们已知的那些。所以，例如，我们可以从实现一个像这样的接口开始：

```kt
interface Handler { 
```

```kt
    fun handle(request: Request): Response 
```

```kt
}
```

我们从未讨论过我回应一位开发者时的样子。那是因为我保持的责任链非常长且复杂，通常情况下，他们倾向于自行解决问题。坦白说，我从未需要回答过他们中的任何一个。但让我们假设回应看起来可能像这样：

```kt
data class Response(val answer: String)
```

我们可以用 *Java 方式* 来做这件事，并在每个处理器内部实现每块逻辑：

```kt
class BasicValidationHandler(private val next: Handler) :   Handler { 
```

```kt
    override fun handle(request: Request): Response { 
```

```kt
        if (request.email.isEmpty() ||
```

```kt
          request.question.isEmpty()) { 
```

```kt
            throw IllegalArgumentException() 
```

```kt
        } 
```

```kt
        return next.handle(request) 
```

```kt
    } 
```

```kt
}
```

如你所见，这里，我们正在实现一个接口，它有一个方法，我们用我们期望的行为来覆盖它。

其他过滤器看起来会非常类似于这个。我们可以以任何我们想要的顺序来组合它们：

```kt
val req = Request("developer@company.com",          "Who broke my build?") 
```

```kt
val chain = BasicValidationHandler(
```

```kt
    KnownEmailHandler(
```

```kt
        JuniorDeveloperFilterHandler(
```

```kt
            AnswerHandler()
```

```kt
        )
```

```kt
    )
```

```kt
) 
```

```kt
val res = chain.handle(req) 
```

但这次我不会问你关于更好做事方式的修辞问题。当然，有更好的方法。我们现在处于 Kotlin 世界。我们已经在上一节中看到了如何使用各种函数。所以，让我们为这个任务定义一个函数：

```kt
typealias Handler = (request: Request) -> Response
```

对于仅仅接收请求并返回响应的东西，我们没有单独的类和接口。以下是我们如何通过使用一个简单的函数作为值在我们的应用程序中实现身份验证的示例：

```kt
val authentication = fun(next: Handler) =
```

```kt
    fun(request: Request): Response {
```

```kt
        if (!request.isKnownEmail()) {
```

```kt
            throw IllegalArgumentException()
```

```kt
        }
```

```kt
        return next(request)
```

```kt
    }
```

在这里，`authentication` 是一个接收函数并返回函数的函数。这种模式允许我们轻松地组合这些函数：

```kt
val req = Request("developer@company.com",     "Why do we need Software Architects?") 
```

```kt
val chain = basicValidation(authentication
```

```kt
  (finalResponse())) 
```

```kt
val res = chain(req) 
```

```kt
println(res)
```

你选择使用哪种方法取决于你。例如，使用接口更加明确，如果你正在创建一个其他人可能想要扩展的库或框架，这可能更适合你。

使用函数更加简洁，如果你只是想以更可管理的方式拆分你的代码，这可能是一个更好的选择。

你可能已经在现实世界中多次看到这种方法。例如，许多网络服务器框架使用它来处理跨切面关注点，如身份验证、授权、日志记录，甚至路由请求。有时，这些被称为 **过滤器** 或 **中间件**，但最终都是相同的责任链设计模式。我们将在 *第十章*，*使用 Ktor 的并发微服务* 和 *第十一章*，*使用 Vert.x 的响应式微服务* 中更详细地讨论它，我们将看到一些最受欢迎的 Kotlin 框架是如何实现它的。

下一个设计模式将与其他所有设计模式都略有不同，并且也更为复杂。

# 解释器

这个设计模式可能看起来非常简单或非常复杂，这取决于你在计算机科学方面的背景知识。一些讨论经典软件设计模式的书籍甚至决定完全省略它，或者把它放在某个地方，仅供好奇的读者阅读。

这背后的原因是 **解释器** 设计模式处理特定语言的翻译。*但我们为什么需要这个？我们不是已经有编译器来做这个了吗？*

## 我们需要更深入

所有开发者都必须掌握多种语言或子语言。即使是普通开发者，我们也使用不止一种语言。想想那些构建你项目的工具，比如 Maven 或 Gradle。你可以将它们的配置文件和构建脚本视为具有特定语法的语言。如果你将元素顺序打乱，你的项目将无法正确构建。这是因为这样的项目有解释器来分析配置文件并对其采取行动。

其他示例包括查询语言，无论是 SQL 的某个变体还是 NoSQL 数据库的特定语言。如果你是 Android 开发者，你可能也会将 XML 布局视为此类语言。甚至 HTML 也可以被认为是定义用户界面的语言。当然，还有其他语言。

可能你已经使用过一些定义了自定义测试语言的测试框架，例如**Cucumber**（[github.com/cucumber](http://github.com/cucumber)）。

这些示例中的每一个都可以被称为**领域特定语言（DSL）**。DSL 是语言中的语言，为特定领域构建。我们将在下一节讨论它们是如何工作的。

## 您自己的语言

在本节中，我们将定义一个简单的*SQL 领域特定语言（DSL）*。我们不会定义其格式或语法；相反，我们将提供一个示例，说明它应该看起来像什么：

```kt
val sql = select("name, age") {
```

```kt
    from("users") {
```

```kt
        where("age > 25")
```

```kt
    } // Closes from
```

```kt
} // Closes select 
```

```kt
println(sql)
```

我们语言的目标是提高可读性并防止一些常见的 SQL 错误，例如拼写错误（例如使用*FORM*而不是`FROM`）。我们将在过程中介绍编译时验证和自动完成。

上一段代码将输出以下内容：

```kt
> SELECT name, age FROM users WHERE age > 25
```

我们将从最简单的一部分开始——实现`select`函数：

```kt
fun select(columns: String, from: SelectClause.()->Unit):  
```

```kt
    SelectClause { 
```

```kt
    return SelectClause(columns).apply(from) 
```

```kt
}
```

我们可以使用单表达式符号来编写这个，但在这里我们使用更冗长的版本以提高可读性。这是一个有两个参数的函数。第一个是一个`String`，很简单。第二个是一个接收无参数并返回无参数的函数。

最有趣的部分是我们指定了 lambda 的接收者：

```kt
SelectClause.()->Unit
```

这是一个非常聪明的技巧，所以请务必跟上。记得我们在*第一章*，*从 Kotlin 入门*中讨论的扩展函数，并在*第二章*，*使用创建型模式*中进行了扩展。上一段代码可以转换为以下代码：

```kt
(SelectClause)->Unit
```

在这里，你可以看到尽管这个 lambda 看起来接收不到任何东西，但它接收一个参数：`SelectClause`类型的对象。第二个技巧在于`apply()`函数的使用，我们之前已经见过。

让我们看看这一行：

```kt
SelectClause(columns).apply(from)
```

这可以转换为以下代码片段：

```kt
val selectClause = SelectClause(columns) 
```

```kt
from(selectClause) 
```

```kt
return selectClause
```

以下是将执行的前一段代码的步骤：

1.  初始化`SelectClause`，这是一个简单的对象，其构造函数接收一个参数。

1.  使用`from()`函数，并传入一个`SelectClause`实例作为其唯一参数。

1.  返回一个`SelectClause`实例。

这段代码只有在 `from()` 对 `SelectClause` 做一些有用的事情时才有意义。

让我们再次看看我们的 DSL 示例：

```kt
select("name, age", { 
```

```kt
    this@select.from("users", { 
```

```kt
        where("age > 25") 
```

```kt
    }) 
```

```kt
})
```

我们现在明确指出了接收者，这意味着 `from()` 函数将调用 `SelectClause` 对象上的 `from()` 方法。

你可以开始猜测这个方法的样子了。它接收 `String` 作为其第一个参数，并接收另一个 lambda 作为第二个参数：

```kt
class SelectClause(private val columns: String) {
```

```kt
    private lateinit var from: FromClause
```

```kt
    fun from(
```

```kt
        table: String,
```

```kt
        where: FromClause.() -> Unit
```

```kt
    ): FromClause {
```

```kt
        this.from = FromClause(table)
```

```kt
        return this.from.apply(where)
```

```kt
    }
```

```kt
    override fun toString() = "SELECT $columns $from"
```

```kt
}
```

这个例子可以缩短，但那样我们就需要在 `apply()` 中使用 `apply()`，这在这个阶段可能会让人感到困惑。

这是我们第一次看到 `lateinit` 关键字。记住，Kotlin 编译器对空安全非常严格。如果我们省略 `lateinit`，它将要求我们用默认值初始化变量。但由于我们将在稍后才知道这一点，我们是在请求编译器稍微放松一下。

重要提示：

注意，如果我们没有履行承诺并忘记初始化变量，当我们第一次访问它时，我们会得到 `UninitializedPropertyAccessException`。

这个关键字相当危险，所以请谨慎使用。

让我们回到我们之前的代码；我们只是做了以下操作：

1.  创建一个 `FromClause` 的实例。

1.  将 `FromClause` 存储为 `SelectClause` 的成员。

1.  将 `FromClause` 的实例传递给 `where` lambda。

1.  返回一个 `FromClause` 的实例。

希望你现在开始理解其精髓：

```kt
select("name, age", { 
```

```kt
    this@select.from("users", { 
```

```kt
        this@from.where("age > 25") 
```

```kt
    }) 
```

```kt
})
```

*这是什么意思？* 在理解了 `from()` 方法之后，这应该会简单得多。`FromClause` 必须有一个名为 `where()` 的方法，它接收一个 `String` 类型的参数：

```kt
class FromClause(private val table: String) {
```

```kt
    private lateinit var where: WhereClause
```

```kt
    fun where(conditions: String) = this.apply {
```

```kt
        where = WhereClause(conditions)
```

```kt
    }
```

```kt
    override fun toString() = "FROM $table $where"
```

```kt
}
```

注意，我们已经履行了承诺，这次缩短了方法。

我们使用接收到的字符串初始化了一个 `WhereClause` 的实例，并返回了它——就这么简单：

```kt
class WhereClause(private val conditions: String) {
```

```kt
    override fun toString() = "WHERE $conditions"
```

```kt
}
```

`WhereClause` 只打印 `WHERE` 和它接收到的条件：

```kt
class FromClause(private val table: String) { 
```

```kt
    // More code here... 
```

```kt
    override fun toString() = "FROM $table $where"
```

```kt
}
```

`FromClause` 打印 `FROM` 以及它接收到的表名和 `WhereClause` 打印的内容：

```kt
class SelectClause(private val columns: String) { 
```

```kt
    // More code here... 
```

```kt
    override fun toString() = "SELECT $columns $from" 
```

```kt
}
```

`SelectClause` 打印 `SELECT`、它得到的列以及 `FromClause` 打印的内容。

### 休息一下

Kotlin 提供了创建可读性和类型安全的 DSL 的美好功能。但解释器设计模式是工具箱中最难的一个。如果你一开始没有理解，请花些时间调试之前的代码。理解每一步中的 `this` 表达式，以及我们调用对象函数和对象方法的时候。

## 调用后缀

我们在这个部分的最后才提到了 Kotlin DSL 的最后一个概念，以免让你感到困惑。

让我们再次看看我们的 DSL：

```kt
val sql = select("name, age") { 
```

```kt
              from("users") { 
```

```kt
                  where("age > 25") 
```

```kt
              } // Closes from 
```

```kt
          } // Closes select
```

注意，尽管 `select` 函数接收两个参数——一个字符串和一个 lambda ——但 lambda 是写在圆括号外面的，而不是里面。

这被称为 **调用后缀**，在 Kotlin 中是一种常见的做法。如果我们的函数接收另一个函数作为其最后一个参数，我们可以将其传递出圆括号。

这使得语法更加清晰，特别是对于像这样的 DSL。

解释器设计模式和 Kotlin 生成类型安全的构建器的能力非常有吸引力。但正如他们所说，*权力越大，责任越大*。所以，考虑一下你的情况是否足够复杂，以至于需要构建一种语言中的语言，或者使用 Kotlin 的基本语法是否足够。

现在，让我们回到我们正在构建的游戏中，看看我们如何可以解耦对象通信。

# 中介

我们游戏开发团队有一些真正的问题——这些问题与代码没有直接关系。如您所知，我们的小型独立公司只有我一个人，一只名叫*迈克尔*的金丝雀，他担任产品经理，以及两名猫设计师，他们大部分时间都在睡觉，但偶尔会制作一些不错的原型。我们完全没有**质量保证**（**QA**）。这可能也是我们的游戏总是崩溃的原因之一。

最近，迈克尔向我介绍了一只名叫肯尼的鹦鹉，他恰好是质量保证（QA）人员：

```kt
interface QA { 
```

```kt
    fun doesMyCodeWork(): Boolean 
```

```kt
} 
```

```kt
interface Parrot { 
```

```kt
    fun isEating(): Boolean 
```

```kt
    fun isSleeping(): Boolean 
```

```kt
} 
```

```kt
object Kenny : QA, Parrot { 
```

```kt
    // Implements interface methods based on parrot     // schedule 
```

```kt
}
```

`Kenny`是一个简单的对象，实现了两个接口：`QA`，用于进行质量保证工作，以及`Parrot`，因为它是一只鹦鹉。

鹦鹉质量保证人员非常积极。他们随时准备测试我游戏的最新版本。但他们不喜欢在睡觉或吃饭时被打扰：

```kt
object Me
```

```kt
object MyCompany {
```

```kt
    val cto = Me
```

```kt
    val qa = Kenny
```

```kt
    fun taskCompleted() {
```

```kt
        if (!qa.isEating() && !qa.isSleeping()) {
```

```kt
            println(qa.doesMyCodeWork())
```

```kt
        }
```

```kt
    }
```

```kt
}
```

如果肯尼有任何问题，我给了他我的直拨电话号码：

```kt
object Kenny : ... { 
```

```kt
    val developer = Me 
```

```kt
}
```

肯尼是一只勤奋的鹦鹉。但我们有如此多的错误，我们不得不雇佣第二只鹦鹉质量保证人员，布拉德。如果肯尼有空，我会把工作交给他，因为他更熟悉我们的项目。但如果他忙，我会检查布拉德是否有空，然后把任务交给他：

```kt
class MyCompany { 
```

```kt
    ... 
```

```kt
    val qa2 = Brad 
```

```kt
    fun taskCompleted() { 
```

```kt
        ... 
```

```kt
        else if (!qa2.isEating() && !qa2.isSleeping()) { 
```

```kt
            println(qa2.doesMyCodeWork()) 
```

```kt
        } 
```

```kt
    } 
```

```kt
}
```

布拉德作为初级员工，通常会先向肯尼确认。而且肯尼也把我的电话号码给了他：

```kt
object Brad : QA, Parrot { 
```

```kt
    val senior = Kenny 
```

```kt
    val developer = Me 
```

```kt
    ... 
```

```kt
}
```

然后，布拉德向我介绍了乔治。乔治是一只猫头鹰，所以他的睡眠时间与肯尼和布拉德不同。这意味着他可以在晚上检查我的代码。

乔治会与肯尼和我一起检查一切：

```kt
object George : QA, Owl { 
```

```kt
    val developer = Me 
```

```kt
    val mate = Kenny
```

```kt
    ... 
```

```kt
}
```

问题在于乔治是一个狂热的足球迷。所以在给他打电话之前，我们需要检查他是否在看比赛：

```kt
class MyCompany { 
```

```kt
    ... 
```

```kt
    val qa3 = George 
```

```kt
    fun taskCompleted() { 
```

```kt
        ... 
```

```kt
        else if (!qa3.isWatchingFootball()) { 
```

```kt
            println(qa3.doesMyCodeWork()) 
```

```kt
        } 
```

```kt
    } 
```

```kt
}
```

肯尼出于习惯，也会向乔治确认，因为乔治是一只知识渊博的猫头鹰：

```kt
object Kenny : QA, Parrot { 
```

```kt
    val peer = George 
```

```kt
    ... 
```

```kt
}
```

然后，还有桑德拉。她是一种不同类型的鸟，因为她不是 QA 的一部分，而是一名文案：

```kt
interface Copywriter { 
```

```kt
    fun areAllTextsCorrect(): Boolean 
```

```kt
} 
```

```kt
interface Kiwi 
```

```kt
object Sandra : Copywriter, Kiwi { 
```

```kt
    override fun areAllTextsCorrect(): Boolean { 
```

```kt
        return ... 
```

```kt
    } 
```

```kt
}
```

我尽量不去打扰她，除非有重大发布：

```kt
class MyMind { 
```

```kt
    ... 
```

```kt
    val translator = Sandra 
```

```kt
    fun taskCompleted(isMajorRelease: Boolean) { 
```

```kt
        ... 
```

```kt
        if (isMajorRelease) { 
```

```kt
            println(translator.areAllTranslationsCorrect()) 
```

```kt
        } 
```

```kt
    } 
```

```kt
}
```

我在这里有几个问题：

+   第一，我试图记住所有那些名字时，我的大脑几乎要爆炸了。你们的可能也是。

+   第二点，我需要记住如何与每个人互动。我是那个在打电话给他们之前做所有检查的人。

+   第三，注意乔治如何试图与肯尼确认一切，肯尼也与乔治确认。幸运的是，到目前为止，当肯尼给他打电话时，乔治总是在看足球比赛。而肯尼在乔治需要确认某事时正在睡觉。否则，他们可能会在电话上永远卡住。

+   第四点，也是我最烦恼的事情，是肯尼计划很快就要离开去开设自己的初创公司，ParrotPi。想象一下我们现在得改多少代码！

我只想检查我的代码是否一切正常。其他人应该做所有的谈话！

## 中介

**中介**设计模式就是一个控制狂。它不喜欢一个对象直接与另一个对象交谈。有时当这种情况发生时，它会生气。不——每个人都应该只通过他来发言。*这个解释是什么？*它减少了对象之间的耦合。而不是了解一些其他对象，每个人都应该只了解他们，即调解人。

我决定迈克尔应该管理所有这些流程，并作为它们的调解人：

```kt
interface Manager { 
```

```kt
    fun isAllGood(majorRelease: Boolean): Boolean 
```

```kt
}
```

只有迈克尔会知道所有的其他鸟：

```kt
object Michael : Canary, ProductManager {
```

```kt
    private val kenny = Kenny(this)
```

```kt
    private val brad = Brad(this)
```

```kt
    override fun isAllGood(majorRelease: Boolean): Boolean {
```

```kt
        if (!kenny.isEating() && !kenny.isSleeping()) {
```

```kt
            println(kenny.doesMyCodeWork())
```

```kt
        } else if (!brad.isEating() && !brad.isSleeping()) {
```

```kt
            println(brad.doesMyCodeWork())
```

```kt
        }
```

```kt
        return true
```

```kt
    }
```

```kt
}
```

注意调解人如何封装不同对象之间的复杂交互，同时提供一个非常简单的接口。

我只会记住迈克尔，他会做剩下的：

```kt
class MyCompany(private val manager: Manager) { 
```

```kt
    fun taskCompleted(isMajorRelease: Boolean) { 
```

```kt
        println(manager.isAllGood(isMajorRelease)) 
```

```kt
    } 
```

```kt
}
```

我也会更改我的电话号码，并确保每个人都只得到迈克尔的：

```kt
class Brad(private val manager: Manager) : ... { 
```

```kt
   // No reference to Me here 
```

```kt
   ... 
```

```kt
}
```

现在，如果有人需要别人的意见，他们需要先通过迈克尔：

```kt
class Kenny(private val manager: Manager) : ... { 
```

```kt
   // No reference to George, or anyone else 
```

```kt
   ... 
```

```kt
}
```

正如你所见，我们通过这个模式无法学到关于 Kotlin 的新知识。

## 调解人类型

中介模式有两种*类型*。我们将它们称为*严格*和*宽松*。我们之前看到了严格版本。我们告诉调解人确切要做什么，并期望从它那里得到答复。

简单版本会要求我们通知调解人发生了什么，但不要期望立即得到答复。相反，如果他们需要反过来通知我们，他们应该给我们打电话。

## 调解人注意事项

迈克尔突然变得非常重要。每个人都只知道他，只有他才能管理他们的互动。他甚至可能成为一个*全能对象*，无所不知、无所不能，这是来自*第九章*，*惯用和反模式*的反模式。即使他那么重要，也要确保定义这个调解人应该做什么，以及——更重要的是——不应该做什么。

让我们继续我们的例子，并讨论另一个行为模式。

# 记忆

自从迈克尔成为经理以来，如果我有问题，就很难找到他。而且当我确实问他问题时，他只是扔下东西就跑去下一个会议。

昨天，我问他我们游戏中应该引入什么新武器。他告诉我应该是一个*椰子加农炮*，一目了然。但今天，当我向他展示这个功能时，他生气地对我尖叫！最后，他说他告诉我实现一个*菠萝发射器*。我很幸运他只是一个金丝雀。

如果我能记录他，那么当我们再次开会，因为他的注意力不集中而出现混乱时，我就可以简单地重放他说的所有内容。

让我们先总结一下我的问题——迈克尔的想法是他的，而且只有他的：

```kt
class Manager { 
```

```kt
    private var thoughts = mutableListOf<String>()
```

```kt
    ... 
```

```kt
}
```

问题在于，由于迈克尔是一个金丝雀，他只能在他脑海中保留`2`个想法：

```kt
class Manager {
```

```kt
    ...
```

```kt
    fun think(thought: String) {
```

```kt
        thoughts.add(thought)
```

```kt
        if (thoughts.size > 2) {
```

```kt
            thoughts.removeFirst()
```

```kt
        }
```

```kt
    }
```

```kt
}
```

如果迈克尔一次思考超过`2`件事情，他会忘记他最初思考的事情：

```kt
michael.think("Need to implement Coconut Cannon")
```

```kt
michael.think("Should get some coffee")
```

```kt
michael.think("Or maybe tea?") // Forgot about Coconut   Cannon
```

```kt
michael.think("No, actually, let's implement Pineapple   Launcher") // Forgot that he wanted coffee
```

即使在录音中，他说的内容也很难理解（因为他没有给出任何回应）。

即使我真的记录了他，迈克尔也可以声称那是他说的，而不是他真正想说的。

备忘录设计模式通过保存对象的内部状态来解决此问题，该状态不能从外部更改（这样迈克尔就不能否认他说过的话）并且只能由对象本身使用。

在 Kotlin 中，我们可以使用一个`inner`类来实现这一点：

```kt
class Manager { 
```

```kt
    ...
```

```kt
    inner class Memory(private val mindState: List<String>) {
```

```kt
        fun restore() {
```

```kt
            thoughts = mindState.toMutableList()
```

```kt
        }
```

```kt
    } 
```

```kt
}
```

在这里，我们可以看到一个新关键字`inner`，用于标记我们的类。如果我们省略这个关键字，类将被称为`Nested`，类似于 Java 中的静态嵌套类。内部类可以访问外部类的私有字段。因此，我们的`Memory`类可以轻松地改变`Manager`类的内部状态。

现在，我们可以通过创建当前状态的印记来记录迈克尔此刻所说的话：

```kt
fun saveThatThought(): Memory {
```

```kt
    return Memory(thoughts.toList())
```

```kt
}
```

在这一点上，我们可以通过一个对象来捕捉他的想法：

```kt
val michael = Manager()
```

```kt
michael.think("Need to implement Coconut Cannon")
```

```kt
michael.think("Should get some coffee")
```

```kt
val memento = michael.saveThatThought()
```

```kt
michael.think("Or maybe tea?")
```

```kt
michael.think("No, actually, let's implement Pineapple   Launcher")
```

现在，我们需要添加一种回到先前思考路径的方法：

```kt
class Manager {
```

```kt
    ... 
```

```kt
    fun `what was I thinking back then?`(memory: Memory) {
```

```kt
        memory.restore()
```

```kt
    }
```

```kt
}
```

在这里，我们可以看到，如果我们想在函数名中使用特殊字符，如空格，我们可以这样做，但前提是函数名被反引号包围。通常这不是最好的主意，但它有其用途，我们将在*第十章*中讨论，*使用 Ktor 的并发微服务*。

剩下的就是使用`memento`回到过去：

```kt
with(michael) {
```

```kt
    think("Or maybe tea?")
```

```kt
    think("No, actually, let's implement Pineapple         Launcher")
```

```kt
}
```

```kt
michael.`what was I thinking back then?`(memento)
```

最后一次调用会将迈克尔的思想引回到思考椰子炮上。

注意我们如何使用`with`标准函数来避免在每一行重复`michael.think()`。这个函数在你需要在同一代码块中经常引用同一对象且希望避免重复时很有帮助。

我不期望你在现实世界中经常看到备忘录设计模式的实现。但它在某些需要恢复到先前状态的应用类型中可能仍然有用。

在本章的开头，我们讨论了迭代器设计模式，它帮助我们处理复杂的数据结构。接下来，我们将探讨另一个具有类似目标的设计模式。

# 访问者

这种设计模式通常是组合设计模式的亲密朋友，我们在*第三章*中讨论过，*理解结构型模式*。它可以从复杂的树状结构中提取数据，或者为树的每个节点添加行为，就像装饰器设计模式对一个单一对象所做的那样。

我的计划，作为一个懒惰的软件架构师，实施得相当顺利。我的请求响应系统基于责任链模式运作得很好，以至于我没有太多时间喝咖啡。但我担心一些开发者开始怀疑我有点儿骗子。

为了让他们困惑，我计划每周发送包含所有最新流行词汇文章链接的电子邮件。当然，我并不打算亲自阅读它们——我只是想从一些流行的技术网站上收集它们。

## 编写爬虫

让我们看看以下数据结构，它与我们在讨论迭代器设计模式时的情况非常相似：

```kt
Page(Container(Image(), 
```

```kt
               Link(), 
```

```kt
               Image()), 
```

```kt
     Table(), 
```

```kt
     Link(), 
```

```kt
     Container(Table(), 
```

```kt
               Link()), 
```

```kt
     Container(Image(), 
```

```kt
               Container(Image(), 
```

```kt
                         Link())))
```

`Page` 是其他 HTML 元素的容器，但本身不是 `HtmlElement`。`Container` 包含其他容器、表格、链接和图像。`Image` 在 `src` 属性中持有其链接。`Link` 则有 `href` 属性。

我们想要做的是从对象中提取所有 URL。

我们将首先创建一个函数，该函数将接收我们对象树的根——在这种情况下是一个 `Page` 容器——并返回所有可用链接的列表：

```kt
fun collectLinks(page: Page): List<String> { 
```

```kt
    // No need for intermediate variable there 
```

```kt
    return LinksCrawler().run { 
```

```kt
        page.accept(this) 
```

```kt
        this.links 
```

```kt
    } 
```

```kt
}
```

使用 `run` 允许我们控制从块的主体返回的内容。在这种情况下，我们将返回我们收集到的 `links` 对象。在 `run` 块内部，这指的是它所操作的对象——在我们的例子中，是 `LinksCrawler`。

在 Java 中，实现访问者设计模式的建议是为每个将接受我们新功能类的类添加一个方法。我们将这样做，但不是针对所有类。相反，我们只为容器元素定义此方法：

```kt
private fun Container.accept(feature: LinksCrawler) { 
```

```kt
    feature.visit(this) 
```

```kt
} 
```

```kt
// Or using a shorter syntax: 
```

```kt
private fun Page.accept(feature: LinksCrawler) =   feature.visit(this)
```

我们的功能需要内部持有集合并公开它以供读取。在 Java 中，我们只为这个成员指定 getter；不需要 setter。在 Kotlin 中，我们可以指定值而不需要后置字段：

```kt
class LinksCrawler { 
```

```kt
    private var _links = mutableListOf<String>() 
```

```kt
    val links 
```

```kt
        get()= _links.toList() 
```

```kt
    ... 
```

```kt
}
```

我们希望我们的数据结构是不可变的。这就是我们调用 `toList()` 的原因。

重要提示：

如果我们使用迭代器设计模式，遍历分支的函数可以进一步简化。

对于容器，我们只需将它们的元素传递下去：

```kt
class LinksCrawler { 
```

```kt
    ... 
```

```kt
    fun visit(page: Page) { 
```

```kt
        visit(page.elements) 
```

```kt
    } 
```

```kt
    fun visit(container: Container) =         visit(container.elements) 
```

```kt
    ... 
```

```kt
}
```

将父类指定为 `sealed` 帮助编译器进一步优化。我们在本章讨论状态设计模式时讨论了密封类。以下是代码：

```kt
sealed class HtmlElement 
```

```kt
class Container(...) : HtmlElement(){ 
```

```kt
    ... 
```

```kt
} 
```

```kt
class Image(...) : HtmlElement() { 
```

```kt
    ... 
```

```kt
} 
```

```kt
class Link(...) : HtmlElement() { 
```

```kt
    ... 
```

```kt
} 
```

```kt
class Table : HtmlElement()
```

我们树状结构中最有趣的逻辑在叶子节点：

```kt
class LinksCrawler { 
```

```kt
    ... 
```

```kt
    private fun visit(elements: List<HtmlElement>) { 
```

```kt
        for (e in elements) { 
```

```kt
            when (e) { 
```

```kt
                is Container -> e.accept(this) 
```

```kt
                is Link -> _links.add(e.href) 
```

```kt
                is Image -> _links.add(e.src) 
```

```kt
                else -> {} 
```

```kt
            } 
```

```kt
        } 
```

```kt
    } 
```

```kt
}
```

注意，在某些情况下，我们可能不想做任何事情。这通过我们的 `else` 子句中的空块来指定，`else -> {}`。这是 Kotlin 中 **智能转换** 的另一个例子。

注意，在我们检查元素是 `Link` 之后，我们获得了对其 `href` 属性的类型安全访问。这是因为编译器为我们做了转换。对于 `Image` 元素也是如此。

尽管我们实现了目标，但这个模式的可用性可以讨论。正如你所见，它是我们更冗长的元素之一，并在接收额外行为和访问者模式本身之间引入了紧密耦合。 

# 模板方法

一些懒惰的人将他们的懒惰变成艺术。以我为例。以下是我的日常日程：

1.  上午 8:00 – 上午 9:00：到达办公室

1.  上午 9:00 – 上午 10:00：喝咖啡

1.  上午 10:00 – 中午 12:00：参加一些会议或审查代码

1.  中午 12:00 – 下午 1:00：外出吃午餐

1.  下午 1:00 – 下午 4:00：参加一些会议或审查代码

1.  下午 4:00：偷偷溜回家

我日程表中的某些部分永远不会改变，而有些则会。具体来说，我的日程表中有两个时间段，任何数量的会议都可能占据。

起初，我以为我可以使用那种设置和销毁逻辑来装饰我的变动日程，这些逻辑发生在前后。但后来有午餐，对于建筑师来说这是神圣的，而且发生在中间。

Java 在你该做什么方面非常明确。首先，你创建一个抽象类。然后，你将所有你想自己实现的方法标记为 `private`：

```kt
abstract class DayRoutine { 
```

```kt
    private fun arriveToWork() { 
```

```kt
        println("Hi boss! I appear in the office 
```

```kt
            sometimes!") 
```

```kt
    } 
```

```kt
    private fun drinkCoffee() { 
```

```kt
        println("Coffee is delicious today") 
```

```kt
    } 
```

```kt
    ... 
```

```kt
    private fun goToLunch() { 
```

```kt
        println("Hamburger and chips, please!") 
```

```kt
    } 
```

```kt
    ... 
```

```kt
    private fun goHome() { 
```

```kt
        // Very important no one notices me, so I must keep         // quiet! 
```

```kt
        println() 
```

```kt
    } 
```

```kt
    ... 
```

```kt
}
```

所有每天都会变化的方法都应该定义为 `abstract`：

```kt
abstract class DayRoutine { 
```

```kt
    ... 
```

```kt
    abstract fun doBeforeLunch() 
```

```kt
    ... 
```

```kt
    abstract fun doAfterLunch() 
```

```kt
    ... 
```

```kt
}
```

如果你想要能够替换一个函数，同时也想提供一个默认实现，你应该将其留为 `public`：

```kt
abstract class DayRoutine { 
```

```kt
    ... 
```

```kt
    open fun bossHook() { 
```

```kt
        // Hope he doesn't hook me there 
```

```kt
    } 
```

```kt
    ... 
```

```kt
}
```

记住，在 Kotlin 中 `public` 是默认的可访问性。

最后，你有一个执行你的算法的方法。它默认是 `final` 的：

```kt
abstract class DayRoutine { 
```

```kt
    ... 
```

```kt
    fun runSchedule() { 
```

```kt
        arriveToWork() 
```

```kt
        drinkCoffee() 
```

```kt
        doAfterLunch() 
```

```kt
        goToLunch() 
```

```kt
        doAfterLunch() 
```

```kt
        goHome() 
```

```kt
    } 
```

```kt
}
```

现在，如果我们想有一个周一的日程表，我们可以简单地实现缺失的部分：

```kt
class MondaySchedule : DayRoutine() { 
```

```kt
    override fun doBeforeLunch() { 
```

```kt
        println("Some pointless meeting") 
```

```kt
        println("Code review. What this does?") 
```

```kt
    } 
```

```kt
    override fun doAfterLunch() { 
```

```kt
        println("Meeting with Ralf") 
```

```kt
        println("Telling jokes to other architects") 
```

```kt
    } 
```

```kt
    override fun bossHook() { 
```

```kt
        println("Hey, can I have you for a sec in my             office?") 
```

```kt
    } 
```

```kt
}
```

*Kotlin 在这个基础上又添加了什么？* 它通常做的事情 – 简洁。正如我们之前看到的，这可以通过函数来实现。

我们有三个 *动态部分* – 两个强制性的活动（软件架构师必须在午餐前后做些事情）和一个可选的（老板可能在他在家之前阻止他）：

```kt
fun runSchedule(beforeLunch: () -> Unit, 
```

```kt
               afterLunch: () -> Unit, 
```

```kt
               bossHook: (() -> Unit)? = fun() { println() }) { 
```

```kt
    ... 
```

```kt
}
```

我们将有一个函数，它接受最多三个其他函数作为其参数。前两个是强制性的，而第三个可能根本不提供或用 `null` 分配，以明确表示我们不希望该函数发生：

```kt
fun runSchedule(...) { 
```

```kt
    ... 
```

```kt
    arriveToWork() 
```

```kt
    drinkCoffee() 
```

```kt
    beforeLunch() 
```

```kt
    goToLunch() 
```

```kt
    afterLunch() 
```

```kt
    bossHook?.let { it() } 
```

```kt
    goHome() 
```

```kt
}
```

在这个函数内部，我们将有我们的算法。`beforeLunch()` 和 `afterLunch()` 的调用应该是清晰的；毕竟，这些是我们作为参数传递给我们的函数。第三个，`bossHook` 可能是 `null`，所以我们只有在它不是 `null` 的情况下才执行它。我们可以使用以下结构来实现这一点：

```kt
?.let { it() }
```

*那么其他函数呢 – 我们总是想自己实现的函数呢？*

Kotlin 有一个关于 **局部函数** 的概念。这些是位于其他函数中的函数：

```kt
fun runSchedule(...) { 
```

```kt
    fun arriveToWork(){ 
```

```kt
        println("How are you all?") 
```

```kt
    } 
```

```kt
    val drinkCoffee = { println("Did someone left the milk         out?") } 
```

```kt
    fun goToLunch() = println("I would like something         italian") 
```

```kt
    val goHome = fun () { 
```

```kt
        println("Finally some rest") 
```

```kt
    } 
```

```kt
    arriveToWork() 
```

```kt
    drinkCoffee() 
```

```kt
    ... 
```

```kt
    goToLunch() 
```

```kt
    ... 
```

```kt
    goHome() 
```

```kt
}
```

这些都是声明局部函数的有效方式。无论你如何定义它们，它们的调用方式都是相同的。局部函数只能由它们被声明的父函数访问，并且是提取常见逻辑而不需要暴露的好方法。

有了这个，我们就剩下了代码结构。定义算法的结构，但让其他人决定在某些点做什么 – 这就是模板方法的核心。

我们几乎到了本章的结尾。还有最后一个设计模式要讨论，但它是最重要的之一。

# 观察者

这可能是本章的一个亮点，这个设计模式为我们提供了一个通往以下章节的桥梁，这些章节专门讨论函数式编程。

*那么，观察者模式究竟是什么？* 你有一个 *发布者*，也可以称为 *主题*，它可能有多个 *订阅者*，也称为 *观察者*。每当发布者发生有趣的事情时，所有订阅者都应该得到更新。

这可能看起来很像**中介者**设计模式，但有一个转折。订阅者应该能够在运行时注册或注销自己。

在经典实现中，所有订阅者/观察者都需要实现一个特定接口，以便发布者可以更新它们。但由于 Kotlin 有高阶函数，我们可以省略这部分。发布者仍然需要提供一种方法，让观察者能够订阅和取消订阅。

这可能听起来有点复杂，所以让我们看看以下示例。

## 动物合唱团示例

因此，一些动物决定拥有自己的合唱团。猫被选为合唱团的指挥（它根本不喜欢唱歌）。

问题在于这些动物逃离了 Java 世界，因此它们没有共同的接口。相反，每个都有不同的发声方式：

```kt
class Bat { 
```

```kt
    fun screech() { 
```

```kt
        println("Eeeeeee") 
```

```kt
    } 
```

```kt
} 
```

```kt
class Turkey { 
```

```kt
    fun gobble() { 
```

```kt
        println("Gob-gob") 
```

```kt
    } 
```

```kt
} 
```

```kt
class Dog { 
```

```kt
    fun bark() { 
```

```kt
        println("Woof") 
```

```kt
    } 
```

```kt
    fun howl() { 
```

```kt
        println("Auuuu") 
```

```kt
    } 
```

```kt
}
```

幸运的是，猫不仅因为嗓音不佳而被选为指挥，而且还因为足够聪明，能够一直跟随到这一章。所以，它知道在 Kotlin 世界中，它可以接受函数：

```kt
class Cat {
```

```kt
    fun joinChoir(whatToCall: ()->Unit) {
```

```kt
        ...
```

```kt
    }
```

```kt
    fun leaveChoir(whatNotToCall: ()->Unit) {
```

```kt
        ...
```

```kt
    }
```

```kt
}
```

之前，我们学习了如何传递一个新函数作为参数，以及一个字面量函数。*但是，我们如何传递成员函数的引用呢？*

我们可以像在策略设计模式中做的那样来做这件事；也就是说，通过使用成员引用操作符（`::`）：

```kt
val catTheConductor = Cat() 
```

```kt
val bat = Bat() 
```

```kt
val dog = Dog() 
```

```kt
val turkey = Turkey() 
```

```kt
catTheConductor.joinChoir(bat::screech) 
```

```kt
catTheConductor.joinChoir(dog::howl) 
```

```kt
catTheConductor.joinChoir(dog::bark) 
```

```kt
catTheConductor.joinChoir(turkey::gobble)
```

现在，猫需要以某种方式保存所有这些订阅者。幸运的是，我们可以将它们放在一个映射中。*那这个键是什么？* 这应该是函数本身：

```kt
class Cat { 
```

```kt
    private val participants = mutableMapOf<()->Unit, ()-        >Unit>() 
```

```kt
    fun joinChoir(whatToCall: ()->Unit) { 
```

```kt
        participants[whatToCall] = whatToCall 
```

```kt
    } 
```

```kt
    ... 
```

```kt
}
```

如果所有那些`()->Unit`实例让你感到头晕，请务必使用`typealias`来赋予它们更多的语义意义，例如 *订阅者*。

现在，蝙蝠决定离开合唱团。毕竟，没有人能听到它那美妙的歌声：

```kt
class Cat { 
```

```kt
    ... 
```

```kt
    fun leaveChoir(whatNotToCall: ()->Unit) { 
```

```kt
        participants.remove(whatNotToCall) 
```

```kt
    } 
```

```kt
    ... 
```

```kt
}
```

所有的`bat`需要做的只是再次传递其订阅者函数：

```kt
catTheConductor.leaveChoir(bat::screech)
```

这就是为什么我们最初使用映射的原因。现在，猫可以调用所有合唱团成员，告诉他们唱歌——好吧，发声：

```kt
typealias Times = Int 
```

```kt
class Cat { 
```

```kt
    ... 
```

```kt
    fun conduct(n: Times) { 
```

```kt
        for (p in participants.values) { 
```

```kt
            for (i in 1..n) { 
```

```kt
                p() 
```

```kt
            } 
```

```kt
        } 
```

```kt
    } 
```

```kt
}
```

因此，排练进行得很顺利。但猫在完成所有那些循环后非常累。它宁愿将任务委托给合唱团成员。这没问题：

```kt
class Cat { 
```

```kt
    private val participants = mutableMapOf<(Int)->Unit, 
```

```kt
         (Int)->Unit>() 
```

```kt
    fun joinChoir(whatToCall: (Int)->Unit) { 
```

```kt
        ... 
```

```kt
    } 
```

```kt
    fun leaveChoir(whatNotToCall: (Int)->Unit) { 
```

```kt
        ... 
```

```kt
    } 
```

```kt
    fun conduct(n: Times) { 
```

```kt
        for (p in participants.values) { 
```

```kt
            p(n) 
```

```kt
        } 
```

```kt
    } 
```

```kt
}
```

我们需要稍微修改订阅者以接收一个新参数。以下是对`Turkey`类的示例：

```kt
class Turkey { 
```

```kt
    fun gobble(repeat: Times) { 
```

```kt
        for (i in 1..repeat) { 
```

```kt
            println("Gob-gob") 
```

```kt
        } 
```

```kt
    } 
```

```kt
}
```

这有点问题。*如果猫要告诉每只动物发出什么样的声音：高音还是低音？* 我们将不得不再次更改所有订阅者以及猫。

在设计发布者时，传递具有许多属性的单个数据类，而不是数据类集合或其他类型。这样，如果添加了新属性，你就不必对订阅者进行太多重构：

```kt
enum class SoundPitch {HIGH, LOW} 
```

```kt
data class Message(val repeat: Times, val pitch:     SoundPitch) 
```

```kt
class Bat { 
```

```kt
    fun screech(message: Message) { 
```

```kt
        for (i in 1..message.repeat) { 
```

```kt
            println("${message.pitch} Eeeeeee") 
```

```kt
        } 
```

```kt
    } 
```

```kt
}
```

在这里，我们使用`enum`来描述不同的音调类型，并使用数据类来封装要使用的音调以及消息应该重复多少次。

确保你的消息是不可变的。*否则，你可能会遇到奇怪的行为！如果你有来自同一发布者的不同消息集怎么办？*我们可以使用智能转换来解决：

```kt
interface Message { 
```

```kt
    val repeat: Times 
```

```kt
    val pitch: SoundPitch  
```

```kt
} 
```

```kt
data class LowMessage(override val repeat: Times) : Message { 
```

```kt
    override val pitch = SoundPitch.LOW 
```

```kt
} 
```

```kt
data class HighMessage(override val repeat: Times) : 
```

```kt
  Message { 
```

```kt
    override val pitch = SoundPitch.HIGH 
```

```kt
} 
```

```kt
class Bat { 
```

```kt
    fun screech(message: Message) { 
```

```kt
        when (message) { 
```

```kt
            is HighMessage -> { 
```

```kt
                for (i in 1..message.repeat) { 
```

```kt
                    println("${message.pitch} Eeeeeee") 
```

```kt
                } 
```

```kt
            } 
```

```kt
            else -> println("Can't :(") 
```

```kt
        } 
```

```kt
    } 
```

```kt
}
```

观察者设计模式非常有用。其力量在于其灵活性。发布者不需要了解任何关于订阅者的信息，除了它调用的函数的签名。在现实世界中，它在反应式框架中得到了广泛的应用，我们将在*第六章*“线程和协程”和*第十一章*“使用 Vert.x 的反应式微服务”中讨论，以及在 Android 中，所有 UI 事件都实现为订阅。

# 摘要

这是一章很长的内容，但我们也学到了很多。我们完成了对所有经典设计模式的覆盖，包括 11 个行为模式。在 Kotlin 中，函数可以被传递到其他函数中，从函数中返回，并分配给变量。这就是高阶函数和函数作为一等公民的概念。如果你的类主要是关于行为，那么用函数替换它通常是有意义的。这个概念帮助我们实现了策略和命令设计模式。

我们了解到迭代器设计模式是语言中的另一个`operator`。密封类使`when`语句变得详尽，我们使用它们来实现状态设计模式。

我们还研究了解释器设计模式，并了解到带有接收者的 lambda 表达式可以使你的领域特定语言（DSL）的语法更清晰。另一个关键字`lateinit`告诉编译器在执行其空安全检查时可以稍微放松一些。*请谨慎使用！*

最后，在讨论观察者设计模式时，我们介绍了如何使用函数引用引用现有方法。

在下一章中，我们将从面向对象编程范式及其众所周知的设计模式转移到另一个范式——函数式编程。

# 问题

1.  中介者和观察者设计模式之间的区别是什么？

1.  什么是**领域特定语言**（**DSL**）？

1.  使用密封类或接口的好处是什么？
