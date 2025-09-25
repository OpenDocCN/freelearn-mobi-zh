# 第四章：熟悉行为模式

本章讨论了 Kotlin 中的行为模式。行为模式处理对象之间的交互方式。

我们将看到对象如何根据情况以完全不同的方式表现，对象如何在不了解彼此的情况下进行通信，以及我们如何轻松地遍历复杂结构。

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

# 策略

记得我们在第三章中讨论**外观**设计模式时设计的平台游戏**Maronic**吗？

好吧，作为我们小型独立游戏开发公司中的游戏设计师，迈克尔想出了一个很好的主意。如果我们给我们的英雄提供一套武器来保护我们免受那些可怕的肉食性蜗牛的伤害，会怎么样呢？

所有武器都会朝着我们英雄面对的方向发射弹丸（你不想靠近那些危险的蜗牛），：

```kt
enum class Direction {
    LEFT, RIGHT
}
```

所有弹丸都应该有一对坐标（记住，我们的游戏是 2D 的？）和一个方向：

```kt
abstract class Projectile(private val x: Int,
                          private val y: Int,
                          private val direction: Direction)
```

如果我们只发射一种类型的弹丸，那会很简单，因为我们已经在第二章中介绍了**工厂**模式，*使用创建型模式*：

```kt
class OurHero {
    private var direction = Direction.LEFT
    private var x: Int = 42
    private var y: Int = 173

    fun shoot(): Projectile {
        return object : Projectile(x, y, direction) {
            // Draw and animate projectile here
        }
    }
}
```

但迈克尔希望我们的英雄至少有三种不同的武器：

+   **豌豆射手**：发射直飞的豌豆。我们的英雄一开始就有它。

+   **石榴**：击中敌人时会爆炸，就像手榴弹一样。

+   **香蕉**：当它到达屏幕末端时会像回旋镖一样返回。

来吧，迈克尔，给我们点空间！你不能就坚持使用所有都一样的普通枪吗？！

# 果实军火库

首先，让我们讨论一下如何用**Java 方式**解决这个问题。

在 Java 中，我们会创建一个接口，来抽象变化。在我们的情况下，变化的是我们的英雄的武器：

```kt
interface Weapon {
    fun shoot(x: Int,
              y: Int,
              direction: Direction): Projectile
}
```

然后所有其他武器都会实现这个接口：

```kt
class Peashooter : Weapon {
    override fun shoot(x: Int,
                       y: Int,
                       direction: Direction) = 
                        object : Projectile(x, y, direction) {
        // Fly straight
    }
}

class Pomegranate : Weapon {
    override fun shoot(x: Int,
                       y: Int,
                       direction: Direction)  = 
                        object : Projectile(x, y, direction) {
        // Explode when you hit first enemy
    }
}

class Banana : Weapon {
    override fun shoot(x: Int,
                       y: Int,
                       direction: Direction)  = 
                        object : Projectile(x, y, direction) {
        // Return when you hit screen border
    }
}
```

然后，我们的英雄会持有一个武器的引用（最初是`Peashooter`）：

```kt
private var currentWeapon : Weapon = Peashooter()
```

它会将实际的射击过程委托给它：

```kt
fun shoot(): Projectile = currentWeapon.shoot(x, y, direction)
```

剩下的就是装备另一件武器的能力：

```kt
fun equip(weapon: Weapon) {
    currentWeapon = weapon
}
```

这就是**策略**设计模式的主要内容。现在，我们的算法（Maronic 的武器，在这种情况下）是可互换的。

# 市民函数

在 Kotlin 中，有一个更有效的方法来实现相同的功能，使用更少的类。这要归功于 Kotlin 中函数是**一等公民**的事实。

这是什么意思？

首先，我们可以将函数分配给类的变量，就像任何其他正常值一样。

显然，你可以将一个原始值分配给你的变量：

```kt
val x = 7
```

你可以将其分配给一个对象：

```kt
var myPet = Canary("Michael")
```

那么，为什么你应该能够将一个函数分配给你的变量？如下所示：

```kt
val square = fun (x: Int): Long {
    return (x * x).toLong()
}
```

在 Kotlin 中，这是完全有效的。

# 转换立场

那么，高阶函数是如何帮助我们这里的呢？

首先，我们将为所有武器定义一个命名空间。这不是强制性的，但有助于保持一切井然有序：

```kt
object Weapons {
    // Functions we'll be there soon
}
```

然后，而不是类，我们每件武器都将成为一个函数：

```kt
val peashooter = fun(x: Int, y: Int, direction: Direction):             Projectile {
        // Fly straight
}

val banana = fun(x: Int, y: Int, direction: Direction): 
    Projectile {
        // Return when you hit screen border
}

val pomegranate = fun(x: Int, y: Int, direction: Direction):             Projectile {
        // Explode when you hit first enemy
}    
```

最有趣的部分是我们的英雄。它现在持有两个函数：

```kt
class OurHero {
    // As before
    var currentWeapon = Weapons.peashooter

    val shoot = fun() {
        currentWeapon(x, y, direction)
    }
}
```

可互换的部分是 `currentWeapon`，而 `shoot` 现在是一个包装它的匿名函数。

为了测试我们的想法是否可行，我们可以先射击默认武器一次，然后切换到“香蕉”并再次射击：

```kt
val h = OurHero()
h.shoot()
h.currentWeapon = Weapons.banana
h.shoot()
```

注意，这大大减少了我们需要编写的类的数量，同时保持了相同的功能。

# 迭代器

在上一章讨论 **组合** 设计模式时，我们注意到这个设计模式感觉有点不完整。现在是时候让出生时分开的双胞胎重聚了。就像阿诺德·施瓦辛格和丹尼·德维托一样，他们非常不同，但很好地互补。

# 一，二...许多

我们回到了我们的小队和班在 *CatsCraft 2: Dogs 的复仇* 策略游戏中。

如你从上一章所记得的，`Squad` 由 `InfantryUnits` 组成：

```kt
interface InfantryUnit

class Squad(val infantryUnits: MutableList<InfantryUnit> =         mutableListOf()) {   
}
```

每个小队现在也应该有一个指挥官了。

叫做“军士”的小队指挥官也是一个 `InfantryUnit`：

```kt
class Squad(...) {
    val commander = Sergeant()
}

class Sergeant: InfantryUnit
```

请忽略我们的军士没有名字并且是即时创建的事实。我们离发布这款游戏并击败竞争对手还有两天。名字现在不重要。

班是由小队组成的，它也有一个指挥官，称为 `中尉`：

```kt
class Platoon(val squads: MutableList<Squad> = mutableListOf()) {
    val commander = Lieutenant()
}

class Lieutenant: InfantryUnit
```

我们的首席执行官想要的是一个班，并且能够知道它由哪些单位组成。

因此，当我们在代码中有以下行时：

```kt
val rangers = Squad("Josh", "Ew    an", "Tom")
val deltaForce = Squad("Sam", "Eric", "William")
val blackHawk = Platoon(rangers, deltaForce)

for (u in blackHawk) {
    println(u)
}
```

我们将按照资历顺序打印：

```kt
Lieutenant, Sergeant, Josh, Ewan, Tom, ...
```

现在，这个任务可能对你来说微不足道，尤其是如果你来自 Java 世界。但在 1994 年，数据结构主要是原始类型的数组。是的，`Array<Int>`，我在看着你。

在 Java 中遍历数组并不难：

```kt
int[] array = new int[] {1, 2, 3};

for (int i = 0; i < array.length; i++) {
    System.out.println(i);
}
```

对于更复杂的东西我们该怎么办？

# 传递价值观

如果你使用像 IntelliJ 这样的 IDE，它将给出有关可能问题的提示：

```kt
for (u in blackHawk) { <== For-loop range must have an 'iterator()'                            method
    // Wanted to do something here
}
```

因此，我们的班需要有一个名为 `iterator()` 的函数。由于它是一个特殊函数，我们需要使用 `operator` 关键字。

```kt
operator fun iterator() = ...
```

我们函数返回的是一个实现了 `Iterator<T>` 接口的匿名对象：

```kt
... = object: Iterator<InfantryUnit> {
    override fun hasNext(): Boolean {
        // Are there more objects to iterate over?
    }

    override fun next(): InfantryUnit {
        // Return next InfantryUnit
    }
}
```

迭代器设计模式背后的思想是将对象存储数据的方式（在我们的例子中，它类似于一棵树）与我们遍历这些数据的方式分开。正如你可能知道的，树可以通过两种方式之一进行迭代：

+   深度优先（也称为 **深度优先搜索** (**DFS**))

+   广度优先（也称为 **广度优先搜索** (**BFS**))

但当我们需要获取所有元素时，我们真的在乎吗？

因此，我们将这两个关注点分开：存储一边，重复一边。

要遍历所有元素，我们需要实现两个方法，一个用于获取下一个元素，另一个用于让循环知道何时停止。

作为例子，我们将为`Squad`实现这个对象。对于排，逻辑将是类似的，但需要更多的数学。

首先，我们需要为我们的迭代器提供一个状态。它将记住最后返回的元素：

```kt
operator fun iterator() = object: Iterator<InfantryUnit> {
    var i = 0
    // More code here
}
```

接下来，我们需要告诉它何时停止。在我们的例子中，元素的总数是队伍的所有单位加上中士：

```kt
override fun hasNext(): Boolean {
    return i < infantryUnits.size + 1
}
```

最后，我们需要知道返回哪个单位。如果是第一次调用，我们将返回中士。接下来的调用将返回队伍成员之一：

```kt
override fun next() =
    when (i) {
        0 -> commander
        else -> infantryUnits[i - 1]
    }.also { i++ }
```

注意，我们想要返回下一个单位，同时也要增加我们的计数器。

为了这个，我们使用`also` `{}`块。

这只是这个模式的一种用法。

同一个对象也可能有多个迭代器。例如，我们可以有一个用于我们的队伍的第二个迭代器，它会按顺序遍历元素。

要使用它，我们需要通过名称调用它：

```kt
for (u in deltaForce.reverseIterator()) {
    println(u)
}
```

由于现在它只是一个返回迭代器的简单函数，我们不需要`operator`关键字：

```kt
fun reverseIterator() = object: Iterator<InfantryUnit> {
    // hasNext() is same as before
}
```

唯一的变化发生在`next()`方法中：

```kt
override fun next() =
        when (i) {
            infantryUnits.size -> commander
            else -> infantryUnits[infantryUnits.size - i - 1]
        }.also { i++ }
```

有时候，将迭代器作为函数的参数也是有意义的：

```kt
fun <T> printAll(iter: Iterator<T>) {
    while (iter.hasNext()) {
        println(iter.next())
    }
}
```

这个函数将遍历任何提供迭代器的对象：

```kt
printAll(deltaForce.iterator())
printAll(deltaForce.reverseIterator())
```

这将打印我们的队伍成员两次，一次是正常顺序，一次是逆序。

作为一名普通的开发者，他或她并不以发明新的数据结构为生，你现在可能经常实现迭代器。但了解它们在幕后是如何工作的仍然很重要。

# 状态

你可以将**状态**设计模式视为一个有偏见的策略，我们在本章的开头讨论了它。但是，虽然策略是由外部，即*客户*改变的，状态可能会根据它接收到的输入内部改变。

看看这个客户与策略之间的对话：

客户：这里有一些新的事情要做，从现在开始做。

策略：好的，没问题。

客户：我喜欢你的地方是，你从不和我争论。

与这个比较一下：

客户：这里有一些我从你那里得到的新输入。

状态：哦，我不知道。也许我会开始做一些不同的事情。也许不会。

客户应该也期望状态可能会拒绝一些输入：

客户：这里有一些东西让你思考，状态。

状态：我不知道那是什么！你没看到我很忙吗？去烦一下策略吧！

那么，为什么客户仍然容忍我们那个状态呢？嗯，状态真的很擅长控制一切。

# 五十种状态

肉食性蜗牛对这个马罗尼英雄已经受够了。他向它们扔豌豆和香蕉，结果却只是到达另一个令人遗憾的城堡。现在它们要行动了！

默认情况下，蜗牛应该静止不动以节省蜗牛能量。但是当英雄靠近时，它将积极地向他冲刺。

如果英雄设法伤害了它，它应该撤退去舔伤口。然后它将重复攻击，直到其中一个死去。

首先，我们声明蜗牛生命中可能发生的事情：

```kt
interface WhatCanHappen {
    fun seeHero()

    fun getHit(pointsOfDamage: Int)

    fun calmAgain()
}
```

我们的蜗牛实现了这个接口，因此它可以被通知任何可能发生的事情，并相应地采取行动：

```kt
class Snail : WhatCanHappen {
    private var healthPoints = 10

    override fun seeHero() {
    }

    override fun getHit(pointsOfDamage: Int) {
    }

    override fun timePassed() {
    }
}
```

现在，我们声明`Mood`类，我们用`sealed`关键字标记它：

```kt
sealed class Mood {
   // Some abstract methods here, like draw(), for example
}
```

密封类是抽象的，不能被实例化。我们将在稍后看到使用它们的优点。但在那之前，让我们声明其他状态：

```kt
class Still : Mood() 

class Aggressive : Mood()

class Retreating : Mood()

class Dead : Mood()
```

这些都是我们蜗牛的不同状态，对不起，是情绪。

在状态设计模式术语中，`Snail`是上下文。它持有状态。因此，我们为它声明一个成员：

```kt
class Snail : WhatCanHappen {
    private var mood: Mood = Still()
    // As before
}
```

现在让我们定义当`Snail`看到我们的英雄时应该做什么：

```kt
override fun seeHero() {
        mood = when(mood) {
            is Still -> Aggressive()
        }
    }
```

编译错误！嗯，这就是`sealed`类发挥作用的地方。就像`enum`一样，Kotlin 知道从它扩展的类数量是有限的。因此，它要求我们的`when`是详尽的，并在其中指定所有不同的案例。

如果你使用 IntelliJ 作为你的 IDE，它甚至会自动建议你“添加剩余的分支”。

让我们描述我们的状态：

```kt
override fun seeHero() {
    mood = when(mood) {
        is Still -> Aggressive()
        is Aggressive -> mood
        is Retreating -> mood
        is Dead -> mood
    }
}
```

当然，`else`仍然有效：

```kt
override fun timePassed() {
    mood = when(mood) {
        is Retreating -> Aggressive()
        else -> mood
    }
}
```

当蜗牛被击中时，我们需要决定它是死是活。为此，我们可以使用不带参数的`when`：

```kt
override fun getHit(pointsOfDamage: Int) {
    healthPoints -= pointsOfDamage

    mood = when {
        (healthPoints <= 0) -> Dead()
        mood is Aggressive -> Retreating()
        else -> mood
    }
}
```

注意，我们使用了`is`关键字，这与 Java 中的`instanceof`相同，但更简洁。

# 国家状况

之前的方法在大多数逻辑都在我们的*上下文*中。你有时可能会看到不同的方法，这在你的*上下文*变大时是有效的。

在这种方法中，`Snail`会变得非常瘦：

```kt
class Snail {
    internal var mood: Mood = Still(this)

    private var healthPoints = 10
    // That's all!
}
```

注意，我们将`mood`标记为`internal`。这允许同一包中的其他类修改它。

而不是让`Snail`实现`WhatCanHappen`，我们的`Mood`将：

```kt
sealed class Mood : WhatCanHappen
```

现在逻辑位于我们的状态对象中：

```kt
class Still(private val snail: Snail) : Mood() {
    override fun seeHero() = snail.mood.run {
            Aggressive(snail)
        }

    override fun getHit(pointsOfDamage: Int) = this
    override fun timePassed() = this
}
```

注意，我们的状态对象现在在构造函数中接收对其上下文的引用。

这是第一次遇到`run`扩展函数。它的等效函数将是：

```kt
override fun seeHero(): Mood {
    snail.mood = Aggressive(snail)
    return snail.mood
}
```

通过使用`run`，我们可以保留相同的逻辑，但省略函数体。

你需要决定使用哪种方法。在我们的例子中，这实际上会产生更多的代码，将不得不自己实现所有的方法。

# 命令

这种设计模式允许你将动作封装在对象中，稍后执行。

此外，如果我们可以在稍后执行一个动作，为什么不执行多个呢？为什么不精确地安排何时执行？

这正是我们现在在**CatsCraft 2: 狗的复仇**游戏中需要做的。*Dave，我们最近雇佣的一位新开发者，整个周末都在努力工作，没有人敢打扰他。他为我们的毛茸茸的士兵实现了以下抽象方法*：

```kt
class Soldier(...)... {
    fun attack(x: Long, y: Long) {
        println("Attacking ($x, $y)")
        // Actual code here
    }

    fun move(x: Long, y: Long) {
        println("Moving to ($x, $y)")
        // Actual code here
    }
}
```

他甚至可能使用了上一章中提到的**桥接**设计模式来做到这一点。

我们现在需要解决的问题是士兵只能记住一个*命令*。就是这样。如果他从`(0, 0)`，屏幕的顶部开始，我们首先告诉他`move(20, 0)`，即向右移动 20 步，然后到`move(20, 20)`，所以他将会直线移动到`(20, 20)`，并且可能会被完全摧毁，因为必须不惜一切代价避开狗敌人：

```kt
cat ⇒  good direction  ⇒    (20, 0)

          [dog] [dog]                   ⇓
        [dog] [dog] [dog]               ⇓
           [dog] [dog]
            (5, 20)                  (20, 20)
```

如果你从本书的开头开始阅读，或者至少从第三章，“理解结构型模式”开始加入，你可能已经有一个想法，我们需要做什么，因为我们已经讨论了语言中*函数作为一等公民*的概念。

但即使你决定只是弄清楚命令设计模式在 Kotlin 中应该如何工作，或者随机打开这本书到这一节，我们也会简要解释如何解决这个狗障碍。

让我们为它画一个骨架。我们知道我们想要保持一个对象的列表，但我们还不知道它们应该是哪种类型。所以现在我们将使用`Any`：

```kt
class Soldier {
    private val orders = mutableListOf<Any>() 

    fun anotherOrder(action: Any) {
        this.orders.add(command)
    }
    // More code here
}
```

然后，我们想要遍历列表并执行我们拥有的命令：

```kt
class Soldier {
    ...
    // This will be triggered from the outside once in a while
    fun execute() {
        while (!orders.isEmpty()) {
            val action = orders.removeAt(0)
            action.execute() // Compile error for now
        }
    }
    ...
}
```

所以，即使你不熟悉命令设计模式，你也可以猜到我们可以定义一个只有一个方法的接口，`execute()`：

```kt
interface Command {
    fun execute()
}
```

然后在成员属性中保持相同时间的一个列表：

```kt
private val commands = mutableListOf<Command>()
```

根据需要实现此接口。这基本上就是大多数情况下 Java 实现此模式所建议的。但难道没有更好的方法吗？

让我们再次看看命令。它的`execute()`方法不接收任何东西，也不返回任何东西，但做了一些事情。它与以下代码相同：

```kt
fun command(): Unit {
  // Some code here
}
```

完全没有区别。我们可以进一步简化这个：

```kt
() -> Unit
```

我们不再需要一个名为`Command`的接口，而将使用`typealias`：

```kt
typealias Command = ()->Unit
```

现在，这一行再次停止编译：

```kt
command.execute() // Unresolved reference: execute 
```

好吧，那是因为`execute()`只是我们发明的一个名字。在 Kotlin 中，函数使用`invoke()`：

```kt
command.invoke() // Compiles
```

这很好，但 Dave 编写的函数接收参数，而我们的函数没有任何参数。

一个选项是改变我们的`Command`签名以接收两个参数：

```kt
(x: Int, y: Int)->Unit
```

但如果某些命令不接收任何参数，或者只接收一个，或者接收两个以上呢？我们还需要记住在每一步传递给`invoke()`的内容。

一个更好的方法是拥有一个函数生成器。也就是说，一个返回另一个函数的函数。

如果你曾经使用过 JavaScript 语言，使用闭包来限制作用域和*记住*东西是一种常见的做法。我们也会这样做：

```kt
val moveGenerator = fun(s: Soldier,
                        x: Int,
                        y: Int): Command {
    return fun() {
        s.move(x, y)
    }
}
```

当用适当的参数调用时，`moveGenerator`将返回一个新的函数。这个函数可以在我们找到合适的时候调用，并且它将*记住*：

+   调用哪个方法

+   使用哪些参数

+   在哪个对象上

现在，我们的`Soldier`可能有一个像这样的方法：

```kt
fun appendMove(x: Int, y: Int) = apply {
        commands.add(moveGenerator(this, x, y))
}
```

它为我们提供了一个流畅的语法：

```kt
val s = Soldier()
s.appendMove(20, 0)
    .appendMove(20, 20)
    .appendMove(5, 20)
    .execute()
```

这段代码将打印以下内容：

```kt
Moving to (20, 0)
Moving to (20, 20)
Moving to (5, 20)
```

# 撤销命令

虽然不是直接相关，但命令设计模式的一个优点是能够撤销命令。如果我们想要支持这样的功能呢？

撤销通常非常棘手，因为它涉及到以下之一：

+   返回到之前的状态（如果有多个客户端则不可能，需要大量内存）

+   计算增量（实现起来很棘手）

+   定义相反的操作（不一定总是可能的）

在我们的情况下，命令*从(0,0)移动到(0,20)*的反面将是*从你现在所在的位置移动到(0,0)*。这可以通过存储一对命令来实现：

```kt
private val commands = mutableListOf<Pair<Command, Command>>()
```

你也可以添加命令对：

```kt
fun appendMove(x: Int, y: Int) = apply {
    val oppositeMove = /* If it's the first command, generate move to current location. Otherwise, get the previous command */
    commands.add(moveGenerator(this, x, y) to oppositeMove)
}
```

实际上，计算反走法相当复杂，因为我们没有保存我们士兵当前的位置（这本来是戴夫应该实现的事情），我们还得处理一些边缘情况。

# 责任链

我是一个糟糕的软件架构师，我不喜欢和人说话。因此，当我坐在 The Ivory Tower（这是我经常去的咖啡馆的名字）里时，我写了一个小的网络应用程序。如果一个开发者有问题，他不应该直接找我，哦不。他需要通过这个系统给我发送一个适当的请求，而我只有在我认为这个请求有价值时才会回答他。

过滤链是网络服务器中一个非常常见的概念。通常，当一个请求到达你那里时，预期的是：

+   它的参数已经过验证

+   如果可能的话，用户已经认证过了

+   用户角色和权限已知，并且用户被授权执行操作

所以，我最初写的代码看起来像这样：

```kt
fun handleRequest(r: Request) {
    // Validate
    if (r.email.isEmpty() || r.question.isEmpty()) {
        return
    }

    // Authenticate
    // Make sure that you know whos is this user
    if (r.email.isKnownEmail()) {
        return
    }

    // Authorize
    // Requests from juniors are automatically ignored by architects
    if (r.email.isJuniorDeveloper()) {
        return
    }

    println("I don't know. Did you check StackOverflow?")
}
```

稍显混乱，但它是有效的。

然后，我注意到一些开发者决定他们可以一次性给我提出两个问题。我得给这个函数添加更多的逻辑。但是等等，我毕竟是个架构师。难道没有更好的方法来*委派*这个任务吗？

这次，我们不会学习新的 Kotlin 技巧，而是使用我们已经学过的。我们可以从一个这样的接口开始实现：

```kt
interface Handler {
    fun handle(request: Request): Response
}
```

我们从未讨论过我对开发者的回应是什么样子。那是因为我保持我的责任链非常长且复杂，通常，他们倾向于自己解决问题。坦白说，我从未需要回答过他们中的任何一个。但至少我们知道他们的请求看起来是什么样子：

```kt
data class Request(val email: String, val question: String)

data class Response(val answer: String)
```

然后，我们可以用 Java 的方式来做，并在每个处理器内部实现每块逻辑：

```kt
class BasicValidationHandler(private val next: Handler) : Handler {
    override fun handle(request: Request): Response {
        if (request.email.isEmpty() || request.question.isEmpty()) {
            throw IllegalArgumentException()
        }

        return next.handle(request)
    }
}
```

其他过滤器看起来会非常类似这个。我们可以按任何顺序组合它们：

```kt
val req = Request("developer@company.com", 
        "Who broke my build?")

val chain = AuthenticationHandler(
                BasicValidationHandler(
                    FinalResponseHandler()))

val res = chain.handle(req)

println(res) 
```

但这次我不会问你关于更好方法的修辞问题。当然有更好的方法，我们现在在 Kotlin 世界。我们已经在上一节中看到了函数的使用。所以，我们将为那个任务定义一个函数：

```kt
typealias Handler = (request: Request) -> Response
```

为什么会有一个单独的类和接口来接收请求并返回响应，简而言之：

```kt
val authentication = fun(next: Handler) =
    fun(request: Request): Response {
        if (!request.email.isKnownEmail()) {
            throw IllegalArgumentException()
        }
        return next(request)
    }
```

在这里，`authentication`是一个函数，它实际上接收一个函数并返回一个函数。

再次，我们可以组合这些函数：

```kt
val req = Request("developer@company.com", 
    "Why do we need Software Architects?")

val chain = basicValidation(authentication(finalResponse()))

val res = chain(req)

println(res)
```

选择哪种方法取决于你。使用接口更明确，如果你正在创建一个其他人可能想要扩展的库或框架，这将更适合你。

使用函数更简洁，如果你只是想以更可管理的方式分割你的代码，这可能是一个更好的选择。

# 解释器

这种设计模式可能看起来非常简单或非常复杂，这完全取决于你在计算机科学方面的背景。一些讨论经典软件设计模式的书籍甚至决定完全省略它，或者把它放在最后，仅供好奇的读者阅读。

这背后的原因是解释器设计模式处理翻译某些语言。但为什么我们需要这样做呢？我们不是已经有编译器来做了吗？

# 我们需要更深入地了解

在本节中，我们讨论了所有开发者都必须掌握多种语言或子语言。即使是普通开发者，我们也使用不止一种语言。想想那些构建你项目的工具，比如 Maven 或 Gradle。你可以将它们的配置文件、**构建脚本**视为具有特定语法的语言。如果你将元素顺序搞错，你的项目将无法正确构建。这是因为这样的项目有解释器来分析配置文件并对它们进行操作。

其他例子可以是 **查询语言**，无论是 SQL 的变体之一还是特定于 NoSQL 数据库的语言之一。

如果你是一名 Android 开发者，你可能也会将 XML 布局视为这类语言。甚至 HTML 也可以被视为定义 **用户界面** 的语言。当然，还有其他语言。

也许你曾经使用过定义了自定义测试语言的测试框架之一，例如 Cucumber：[github.com/cucumber](https://github.com/cucumber)。

这些示例中的每一个都可以被称为 **领域特定语言**（**DSL**）。一种语言中的语言。我们将在下一节中讨论它是如何工作的。

# 你自己的语言

在本节中，我们将定义一个简单的 DSL-for-SQL 语言。我们不会定义其格式或语法，而只是提供一个示例，说明它应该看起来像什么：

```kt
val sql = select("name, age", {
              from("users", {
                  where("age > 25")
              }) // Closes from
          }) // Closes select

println(sql) // "SELECT name, age FROM users WHERE age > 25"
```

我们语言的目标是提高可读性并防止一些常见的 SQL 错误，例如拼写错误（如 FORM 而不是 FROM）。我们将在过程中获得编译时验证和自动完成。

我们将从最简单的一部分——`select`——开始：

```kt
fun select(columns: String, from: SelectClause.()->Unit): 
    SelectClause {
    return SelectClause(columns).apply(from)
}
```

我们可以用单表达式表示法来写这个，但我们使用更冗长的版本以便于示例的清晰性。

这是一个有两个参数的函数。第一个是一个 `String`，很简单。第二个是一个接收什么都不返回的函数。

最有趣的部分是我们为我们的 lambda 指定了接收者：

```kt
SelectClause.()->Unit
```

这是一个非常聪明的技巧，所以请确保跟得上：

```kt
SelectClause.()->Unit == (SelectClause)->Unit
```

虽然看起来这个 lambda 没有接收任何东西，但实际上它接收了一个参数，一个类型为 `SelectClause` 的对象。

第二个技巧在于使用我们之前见过的 `apply()` 函数。

看看这个：

```kt
SelectClause(columns).apply(from)
```

这翻译成这样：

```kt
val selectClause = SelectClause(columns)
from(selectClause)
return selectClause
```

以下是将执行的前置代码的步骤：

1.  初始化 `SelectClause`，这是一个简单的对象，它在构造函数中接收一个参数。

1.  使用 `from()` 函数，并传入一个 `SelectClause` 实例作为唯一参数。

1.  返回一个 `SelectClause` 实例。

那段代码只有在 `from()` 对 `SelectClause` 做一些有用的事情时才有意义。

让我们再次看看我们的 DSL 示例：

```kt
select("name, age", {
 this@select.from("users", {
        where("age > 25")
    })
})
```

我们现在已经明确指定了接收者，这意味着 `from()` 函数将调用 `SelectClause` 对象上的 `from()` 方法。

你可以开始猜测这个方法的样子。很明显，它接收一个 `String` 作为其第一个参数，并接收另一个 lambda 作为其第二个参数：

```kt
class SelectClause(private val columns: String) {
    private lateinit var from : FromClause
    fun from(table: String, where: FromClause.()->Unit): FromClause {
        this.from = FromClause(table)
        return this.from.apply(where)
    }
}
```

这又可以进一步缩短，但这样我们可能需要在 `apply()` 中使用 `apply()`，这可能会让人感到困惑。

这是第一次遇到 `lateinit` 关键字。这个关键字相当危险，所以请谨慎使用。记住，Kotlin 编译器对空安全非常严格。如果我们省略 `lateinit`，它将要求我们使用默认值初始化变量。但由于我们将在稍后知道它，我们要求编译器稍微放松一下。注意，如果我们没有履行我们的承诺并忘记初始化它，当我们第一次访问它时，我们会得到 `UninitializedPropertyAccessException`。

回到我们的代码；我们做的只是：

1.  创建一个 `FromClause` 的实例

1.  将其存储为 `SelectClause` 的成员

1.  将 `FromClause` 的实例传递给 `where` lambda

1.  返回一个 `FromClause` 的实例

希望你现在开始理解其精髓：

```kt
select("name, age", {
    this@select.from("users", {
 this@from.where("age > 25")
    })
})
```

这意味着什么？在理解了 `from()` 方法之后，这应该会简单得多。`FromClause` 必须有一个名为 `where()` 的方法，它接收一个 `String` 类型的参数：

```kt
class FromClause(private val table: String) {
    private lateinit var where: WhereClause

    fun where(conditions: String) = this.apply {
        where = WhereClause(conditions)
    }
}
```

注意，我们这次履行了我们的承诺，缩短了方法。

我们使用接收到的字符串初始化了一个 `WhereClause` 的实例，并返回了它。就这么简单：

```kt
class WhereClause(private val conditions: String) {
    override fun toString(): String {
        return "WHERE $conditions"
    }
}
```

`WhereClause` 只打印出单词 `WHERE` 和它接收到的条件：

```kt
class FromClause(private val table: String) {
    // More code here...
    override fun toString(): String {
        return "FROM $table ${this.where}"
    }
}
```

`FromClause` 打印出单词 `FROM` 以及它接收到的表名，以及 `WhereClause` 打印出的所有内容：

```kt
class SelectClause(private val columns: String) {
    // More code here...
    override fun toString(): String {
        return "SELECT $columns ${this.from}"
    }
}
```

`SelectClause` 打印出单词 `SELECT`，它获取的列，以及 `FromClause` 打印出的任何内容。

# 休息一下

Kotlin 提供了创建可读性和类型安全的 DSL 的美丽功能。但解释器设计模式是工具箱中最难的一个。如果你一开始没有理解，请花些时间调试这段代码。理解每一步中的 `this` 的含义，以及我们调用函数和调用对象方法的时候。

# 调用后缀

为了不让你感到困惑，我们在本节结束前省略了一个 Kotlin DSL 的最后概念。

看看这个 DSL：

```kt
val sql = select("name, age", {
              from("users", {
                  where("age > 25")
              }) // Closes from
          }) // Closes select
```

它可以被重写为：

```kt
val sql = select("name, age") {
              from("users") {
                  where("age > 25")
              } // Closes from
          } // Closes select
```

这在 Kotlin 中是一种常见做法。如果我们的函数接收另一个函数作为其最后一个参数，我们可以将其传递出括号。

这使得 DSL 更加清晰，但一开始可能会让人感到困惑。

# 中介

没有其他办法。**中介者**设计模式就是一个控制狂。它不喜欢一个对象直接与另一个对象交流。当这种情况发生时，它有时会变得很生气。不，每个人都应该只通过他来交流。他的解释是什么？它减少了对象之间的耦合。不是每个人都应该知道一些其他对象，而是每个人都应该只认识他，即中介者。

# 森林中的麻烦

抛开建筑笑话不谈，我们，*Maronic* 开发团队，确实有一些实际问题。这些问题与代码没有直接关系。你可能还记得，我们的小型独立公司只有我一个人，一只名叫迈克尔的金丝雀，它担任产品经理，还有两位猫设计师，他们大部分时间都在睡觉，但偶尔也会制作出一些不错的原型。我们没有质量保证团队（即质量保证人员）。也许这就是我们的游戏总是频繁崩溃的原因之一。

最近，迈克尔介绍我认识了一只名叫 `Kenny` 的鹦鹉，他恰好是质量保证人员：

```kt
interface QA {
    fun doesMyCodeWork(): Boolean
}

interface Parrot {
    fun isEating(): Boolean
    fun isSleeping(): Boolean
}

object Kenny : QA, Parrot {
    // Implements interface methods based on parrot schedule
}
```

为了简化，本节将使用对象。

鹦鹉质量保证人员非常有动力。他们随时准备测试我游戏的最新版本。但他们真的很不喜欢在睡觉或吃饭时被打扰：

```kt
class MyMind {
    val qa = Kenny

    fun taskCompleted() {
        if (!qa.isEating() && !qa.isSleeping()) {
            println(qa.doesMyCodeWork())
        }
    }
}
```

如果 `Kenny` 有任何问题，我会给他我的直接电话号码：

```kt
object Kenny : ... {
    val developer = Me
}
```

`Kenny` 是一只勤奋的鹦鹉。但我们有如此多的虫子，以至于我们不得不雇佣第二只鹦鹉质量保证人员，Brad。如果 `Kenny` 有空，我会把工作交给他，因为他更熟悉我们的项目。但如果他忙，我会检查 Brad 是否有空，然后把这项任务交给他：

```kt
class MyMind {
    ...
    val qa2 = Brad

    fun taskCompleted() {
        ...
        else if (!qa2.isEating() && !qa2.isSleeping()) {
            println(qa2.doesMyCodeWork())
        }
    }
}
```

`Brad` 作为初级成员，通常会先向 `Kenny` 汇报。而且 `Kenny` 也把我的电话号码给了他：

```kt
object Brad : QA, Parrot {
    val senior = Kenny
    val developer = Me
    ...
}
```

然后 Brad 介绍我认识 `George`。`George` 是一只猫头鹰，所以他的睡眠时间与 `Kenny` 和 Brad 不同。这意味着他可以在夜间检查我的代码。问题是，`George` 是一名狂热的足球迷。所以在给他打电话之前，我们需要检查他是否正在看比赛：

```kt
class MyMind {
    ...
    val qa3 = George

    fun taskCompleted() {
        ...
        else if (!qa3.isWatchingFootball()) {
            println(qa3.doesMyCodeWork())
        }
    }
}
```

`Kenny` 习惯性地也会向 `George` 汇报，因为 `George` 是一只非常博学的猫头鹰：

```kt
object Kenny : QA, Parrot {
    val peer = George
    ...
}
```

`George` 会检查 `Kenny`，因为 `Kenny` 似乎也对足球感兴趣：

```kt

object George : QA, Owl {
    val mate = Kenny
    ...
}
```

`George` 喜欢在夜间用他的问题打扰我：

```kt
object George : QA, Owl {
    val developer = Me
    ...
}
```

然后是 `Sandra`。她是一种不同类型的鸟，因为她不是质量保证人员，而是一名文案：

```kt
interface Copywriter {
    fun areAllTextsCorrect(): Boolean
}

interface Kiwi

object Sandra : Copywriter, Kiwi {
    override fun areAllTextsCorrect(): Boolean {
        return ...
    }
}
```

我尽量不去打扰她，除非是重大发布：

```kt
class MyMind {
    ...
    val translator = Sandra

    fun taskCompleted(isMajorRelease: Boolean) {
        ...
        if (isMajorRelease) {
            println(translator.areAllTranslationsCorrect())
        }
    }
}
```

好吧，现在我有一些问题：

+   首先，我试图记住所有这些名字时，我的大脑几乎要爆炸了。你的可能也是。

+   其次，我还需要记住如何与每个人互动。在给他们打电话之前，所有的检查都是由我来做的。

+   第三，注意乔治如何试图与肯尼确认每一件事，肯尼也总是与乔治确认？幸运的是，到目前为止，乔治总是在肯尼打电话时看足球比赛。而肯尼在乔治需要确认某事时却在睡觉。否则，他们可能会在电话上陷入永恒的僵局...

+   第四，最让我烦恼的是，`Kenny` 打算不久后离开去开创自己的创业公司，ParrotPi。想象一下我们现在得改多少代码！

我只想检查我的代码是否一切正常。其他人应该做所有这些谈话！

# 中间人

因此，我决定迈克尔应该管理所有这些流程：

```kt
interface Manager {
    fun isAllGood(majorRelease: Boolean): Boolean
}
```

只有他知道所有其他的*鸟儿*：

```kt
object Michael: Canary, Manager {
    private val kenny = Kenny(this)
    // And all the others
    ...

    override fun isAllGood(majorRelease: Boolean): Boolean {
        if (!kenny.isEating() && !kenny.isSleeping()) {
            println(kenny.doesMyCodeWork())
        }
        // And all the other logic I had in MyMind
        ...
    }
}
```

我只会记住他，他会做剩下的：

```kt
class MyPeacefulMind(private val manager: Manager) {
    fun taskCompleted(isMajorRelease: Boolean) {
        println(manager.isAllGood(isMajorRelease))
    }
}
```

我也会更改我的电话号码，并确保每个人都只得到迈克尔的：

```kt

class Brad(private val manager: Manager) : ... {
   // No reference to Me here
   ...
}
```

现在，如果有人需要别人的意见，他们需要先通过迈克尔。

```kt
class Kenny(private val manager: Manager) : ... {
   // No reference to George, or anyone else
   ...
}
```

# 风味

中介有两个*风味*。我们将它们称为*严格*和*宽松*。我们之前看到的是严格版本。我们告诉中介确切要做什么，并期望它给出回应。

*宽松*版本将期望我们通知中介发生了什么，但不需要期望立即的回应。相反，如果他需要反过来通知我们，他应该调用我们。

# 注意事项

迈克尔突然变得非常重要。每个人都只知道他，而且只有他可以管理他们的互动。他甚至可能成为一个*神对象*，无所不知且全能，这是来自第九章的反模式，*设计用于并发*。即使他如此重要，也一定要定义这个中介应该做什么，更重要的是，它不应该做什么。

# 备忘录

自从迈克尔成为经理以来，如果我有问题很难找到他。而且当我确实问他问题时，他只是扔下东西就跑向下一个会议。

昨天，我问他我们将在我们的*Maronic*游戏中引入的下一件武器应该是什么。他告诉我应该是一个椰子大炮，一目了然。但今天，当我向他展示这个功能时，他愤怒地对我尖叫！他说他告诉我实现一个菠萝发射器。我很幸运他只是一个金丝雀...

但如果我能记录他，当我们在另一次会议中遇到麻烦，因为他没有全神贯注时，我就可以重新播放他说的所有内容。

# 记忆

首先总结我的问题——迈克尔的想法是他的，而且只有他的：

```kt
class Manager {
    private var lastThought = "Should get some coffee"
    private var repeatThat = 3
    private var thenHesitate = "Or maybe tea?"
    private var secretThought = "No, coffee it is"
    ...
}
```

此外，它们相当复杂且分散。我无法访问它们，只能访问它们的副产品：

```kt
class Manager {
    ...
    fun whatAreYouThinking() {
        for (i in 1..repeatThat) {
            println(lastThought)
        }
        println(thenHesitate)
    }
    ...
}
```

即使记录他说的内容也很困难（因为他不返回任何东西）。

即使我真的记录了他，迈克尔也可以声称那是他说的话，而不是他的意思：

你为什么给我带茶？我想要咖啡！

解决方案可能看起来很明显。让我们使用一个内部类，`thought`，来捕捉这个最后的想法：

```kt
class Manager {
    ...
    class Thought {
        fun captureThought(): CapturedThought {
            return CapturedThought(lastThought, 
                                   repeatThat,                              
                                   thenHesitate, 
                                   secretThought)
        }
    }

    data class CapturedThought(val thought: String, 
                               val repeat: Int, 
                               val hesitate: String,
                               val secret: String)
}
```

唯一的问题是这个代码无法编译。这是因为我们遗漏了一个新的关键字，`inner`，用来标记我们的类。如果我们省略这个关键字，类将被称为`Nested`，类似于 Java 中的静态嵌套类。

现在我们可以*记录*迈克尔此刻说的话：

```kt
val michael = Manager()

val captured = michael.Thought().captureThought()
```

让我们假设迈克尔在某个时刻改变了主意。我们将添加另一个函数来处理这种情况：

```kt
class Manager {
    ...
    fun anotherThought() {
        lastThought = "Tea would be better"
        repeatThat = 2
        thenHesitate = "But coffee is also nice"
        secretThought = "Big latte would be great"
    }
}
michael.anotherThought()
```

我们可以反复思考我们捕捉到的想法：

```kt
michael.whatAreYouThinking()
```

这将打印：

```kt
Tea would be better
Tea would be better
But coffee is also nice
```

让我们检查我们捕捉到了什么：

```kt
println(captured)
```

这将打印：

```kt
CapturedThought(thought=Should get some coffee, repeat=3, hesitate=Or maybe tea?, secret=No, coffee it is)
```

如果迈克尔允许，我们甚至可以倒带他的想法：

```kt
class Manager {
    ...
    inner class Thought {
        ...
        fun rewindThought(val previousThought: CapturedThought) {
            with(previousThought) {
                lastThought = thought
                repeatThat = repeat
                thenHesitate = hesitate
                secretThought = secret
            }
        }
    }
    ...
}
```

注意这里我们如何使用 `with` 标准函数来避免在每一行重复 `previousThought`。

# 访问者

这种设计模式通常是我们在 第三章 讨论的 **Composite** 设计模式的亲密朋友，*理解结构型模式*。它可以从复杂的树形结构中提取数据，或者向树的每个节点添加行为，就像 **Decorator** 设计模式一样。

因此，作为一个懒惰的软件架构师，我的计划实施得相当顺利。我的邮件发送系统来自 **Builder**，我的请求回答系统来自 **Chain of Responsibility**，都运行得相当好。但一些开发者开始怀疑我有点儿骗子。

为了混淆他们，我计划每周发送包含所有最新流行词汇文章链接的电子邮件。当然，我并不打算亲自阅读它们，只是从一些流行的技术网站上收集它们。

# 编写爬虫

让我们看看以下结构，这与我们讨论 **Iterator** 设计模式时非常相似：

```kt
Page(Container(Image(),
               Link(),
               Image()),
     Table(),
     Link(),
     Container(Table(),
               Link()),
     Container(Image(),
               Container(Image(),
                         Link())))
```

`Page` 是其他 `HtmlElements` 的容器，但本身不是 `HtmlElement`。`Container` 持有其他容器、表格、链接和图像。`Image` 在 `src` 属性中持有其链接。`Link` 有 `href` 属性。

我们首先创建一个函数，该函数将接收我们的对象树根，在这个例子中是一个 `Page`，并返回所有可用的链接列表：

```kt
fun collectLinks(page: Page): List<String> {
    // No need for intermediate variable there
    return LinksCrawler().run {
        page.accept(this)
        this.links
    }
}
```

使用 `run` 允许我们控制从块体返回的内容。在这种情况下，我们将返回我们收集到的 `links`。

在 Java 中，实现 **Visitor** 设计模式的建议是为每个接受我们新功能的类添加一个方法。我们将这样做，但不是为所有类。相反，我们只为容器元素定义此方法：

```kt
private fun Container.accept(feature: LinksCrawler) {
    feature.visit(this)
}

// Same as above but shorter
private fun Page.accept(feature: LinksCrawler) = feature.visit(this)
```

我们的功能需要内部持有集合，并且仅暴露读取目的。在 Java 中，我们会为该成员指定只读的 getter 而没有 setter。在 Kotlin 中，我们可以指定值而不需要后端字段：

```kt
class LinksCrawler {
    private var _links = mutableListOf<String>()

    val links
        get()= _links.toList()
    ...
}
```

我们希望我们的数据结构是不可变的。这就是我们调用 `toList()` 的原因。

如果我们使用 **Iterator** 设计模式，迭代分支的函数可以进一步简化。

对于容器，我们只需将它们的元素传递下去：

```kt
class LinksCrawler {
    ...
    fun visit(page: Page) {
        visit(page.elements)
    }

    fun visit(container: Container) = visit(container.elements)
    ...
}
```

将父类指定为 `sealed` 帮助编译器进一步优化：

```kt
sealed class HtmlElement

class Container(...) : HtmlElement(){
    ...
}

class Image(...) : HtmlElement() {
    ...
}

class Link(...) : HtmlElement() {
    ...
}

class Table : HtmlElement()
```

最有趣的逻辑在叶子节点：

```kt
class LinksCrawler {
    ...
    private fun visit(elements: List<HtmlElement>) {
        for (e in elements) {
            when (e) {
                is Container -> e.accept(this)
                is Link -> _links.add(e.href)
                is Image -> _links.add(e.src)
                else -> {}
            }
        }
    }
}
```

注意，在某些情况下，我们不想做任何事情。这通过 else 中的空块来指定：`else -> {}`。

这是我们第一次在 Kotlin 中看到 **smart casts**。

注意，在我们检查元素是 `Link` 之后，我们获得了对其 `href` 属性的类型安全访问。这是因为编译器为我们做了类型转换。同样，对于 `Image` 元素也是如此。

尽管我们实现了我们的目标，但这个模式的可用性可以争论。正如你所看到的，它是一个更冗长的元素之一，并且引入了接收额外行为和访问者本身的紧密耦合。

# 模板方法

一些懒惰的人将他们的懒惰变成了艺术。以我为例。以下是我的日常日程：

1.  8:00–9:00: 到达办公室

1.  9:00–10:00: 喝咖啡

1.  10:00–12:00: 参加一些会议或审查代码

1.  12:00–13:00: 外出吃午餐

1.  13:00–16:00: 参加一些会议或审查代码

1.  16:00: 悄悄溜回家

如你所见，日程表的一些部分永远不会改变，而有些则会改变。起初，我以为我可以用那个 `setup` 和 `teardown` 逻辑来“装饰”我变化的日程，这些逻辑发生在“之前”和“之后”。但午餐是神圣的，对于建筑师来说，它发生在“之间”。

Java 在你该做什么方面非常明确。首先，你创建一个抽象类。所有你想自己实现的方法你标记为私有：

```kt
abstract class DayRoutine {
    private fun arriveToWork() {
        println("Hi boss! I appear in the office sometimes!")
    }

    private fun drinkCoffee() {
        println("Coffee is delicious today")
    }

    ...

    private fun goToLunch() {
        println("Hamburger and chips, please!")
    }

    ...

    private fun goHome() {
        // Very important no one notices me
        println()
    }

    ...
}
```

对于所有每天都会变化的方法，你定义一个抽象类：

```kt
abstract class DayRoutine {
    ...
    abstract fun doBeforeLunch()
    ...
    abstract fun doAfterLunch()
    ...
}
```

如果你允许方法的变化，但想提供一个默认实现，你可以将其设置为公开：

```kt
abstract class DayRoutine {
    ...
    open fun bossHook() {
        // Hope he doesn't hook me there
    }
    ...
}
```

最后，你有一个执行你算法的方法。默认情况下它是最终的：

```kt
abstract class DayRoutine {
    ...
    fun runSchedule() {
        arriveToWork()
        drinkCoffee()
        doAfterLunch()
        goToLunch()
        doAfterLunch()
        goHome()
    }
}
```

如果我们现在想有一个周一的日程表，我们只需实现缺失的部分：

```kt
class MondaySchedule : DayRoutine() {
    override fun doBeforeLunch() {
        println("Some pointless meeting")
        println("Code review. What this does?")
    }

    override fun doAfterLunch() {
        println("Meeting with Ralf")
        println("Telling jokes to other architects")
    }

    override fun bossHook() {
        println("Hey, can I have you for a sec in my office?")
    }
}
```

Kotlin 在这个基础上添加了什么？它通常做的事情——简洁。正如我们之前所看到的，这可以通过函数来实现。

我们有三个“移动部件”——两个强制性活动（软件架构师必须在午餐前后做些事情）和一个可选的（老板可能在他在家之前或之后偷偷溜走之前阻止他）：

```kt
fun runSchedule(beforeLunch: ()->Unit,
                afterLunch: ()->Unit,
                bossHook: (()->Unit)? = fun() { println() }) {
    ...
}
```

我们将有一个函数，它接受最多三个其他函数作为其参数。前两个是强制性的，第三个可能根本不提供，或者用 `null` 分配，以明确表示我们不希望该函数发生：

```kt
fun runSchedule(...) {
    ...
    arriveToWork()
    drinkCoffee()
    beforeLunch()
    goToLunch()
    afterLunch()
    bossHook?.let { it() }
    goHome()
}
```

在这个函数内部，我们将有我们的算法。对 `beforeLunch()` 和 `afterLunch()` 的调用应该是清晰的；毕竟，这些是我们作为参数传递给我们的函数。第三个，bossHook，可能为 null，所以我们只有在它不为 null 时才执行它：`?.let { it() }`。

但其他函数怎么办，那些我们希望总是自己实现的函数呢？Kotlin 有一个局部函数的概念。这些函数位于其他函数中：

```kt
fun runSchedule(...) {
    fun arriveToWork(){
        println("How are you all?")
    }

    val drinkCoffee = { println("Did someone left the milk out?") }

    fun goToLunch() = println("I would like something italian")

    val goHome = fun () {
        println("Finally some rest")
    }

    arriveToWork()
    drinkCoffee()
    ...
    goToLunch()
    ...
    goHome()
}
```

这些都是声明局部函数的有效方式。无论你如何定义它们，它们的调用方式都是相同的。

我们得到了相同的结果，正如你所看到的。定义算法结构，但让其他人决定在某些点的操作：这就是模板方法的核心。

# 观察者

这可能是本章的一个亮点，这个设计模式将为我们提供一个通往以下章节的桥梁，这些章节专门介绍函数式编程。

那么，观察者模式是什么？你有一个*发布者*，它也可能被称为*主题*，它可能有多个*订阅者*，它们也可能被称为*观察者*。每当发布者发生有趣的事情时，它应该更新所有订阅者。

这可能看起来很像中介者设计模式，但有一个转折。订阅者应该能够在运行时注册或注销自己。

在经典实现中，所有订阅者/观察者都需要实现一个特定的接口，以便发布者能够更新它们。但是，由于 Kotlin 有高阶函数，我们可以省略这部分。发布者仍然需要提供观察者订阅和取消订阅的方式。

# 动物合唱团

所以，动物们决定有自己的合唱团。猫被选为合唱团的指挥（它根本不喜欢唱歌）。

问题在于动物们逃离了 Java 世界，没有共同的接口。相反，每个动物都有自己的方法来发出声音：

```kt
class Bat {
    fun screech() {
        println("Eeeeeee")
    }
}

class Turkey {
    fun gobble() {
        println("Gob-gob")
    }
}

class Dog {
    fun bark() {
        println("Woof")
    }

    fun howl() {
        println("Auuuu")
    }
}
```

幸运的是，猫不仅因为嗓音不佳而被选为指挥，而且因为它足够聪明，能够一直读到这一章。所以它知道在 Kotlin 世界中，它可以接受函数：

```kt
class Cat {
    ...
    fun joinChoir(whatToCall: ()->Unit) {
        ...
    }

    fun leaveChoir(whatNotToCall: ()->Unit) {
        ...
    }
    ...
}
```

之前，我们看到了如何传递一个新函数作为参数，以及如何传递一个字面函数。但是，我们如何传递成员函数的引用？

这就是成员引用操作符的作用——`::`：

```kt
val catTheConductor = Cat()

val bat = Bat()
val dog = Dog()
val turkey = Turkey()

catTheConductor.joinChoir(bat::screech)
catTheConductor.joinChoir(dog::howl)
catTheConductor.joinChoir(dog::bark)
catTheConductor.joinChoir(turkey::gobble)
```

现在，猫需要以某种方式保存所有这些订阅者。幸运的是，我们可以把它们放在一个映射中。那这个键会是什么？它可以是函数本身：

```kt
class Cat {
    private val participants = mutableMapOf<()->Unit, ()->Unit>()

    fun joinChoir(whatToCall: ()->Unit) {
        participants.put(whatToCall, whatToCall)
    }
    ...
}
```

如果所有这些`()->Unit`实例让你感到头晕，请务必使用`typealias`来赋予它们更多的语义意义，例如订阅者。

蝙蝠决定离开合唱团。毕竟，没有人能听到它那美妙的歌声：

```kt
class Cat {
    ...
    fun leaveChoir(whatNotToCall: ()->Unit) {
        participants.remove(whatNotToCall)
    }
    ...
}
```

所有`Bat`需要做的就是再次传递其订阅者函数：

```kt
catTheConductor.leaveChoir(bat::screech)
```

这就是为什么我们最初使用映射的原因。现在`Cat`可以调用所有合唱团成员并告诉他们唱歌。好吧，发出声音：

```kt
typealias Times = Int

class Cat {
    ...
    fun conduct(n: Times) {
        for (p in participants.values) {
            for (i in 1..n) {
                p()
            }
        }
    }
}
```

排练进行得很顺利。但`Cat`在做了所有这些循环之后感到非常累。它宁愿将工作委托给合唱团成员。这根本不是问题：

```kt
class Cat {
    private val participants = mutableMapOf<(Int)->Unit, (Int)->Unit>()

    fun joinChoir(whatToCall: (Int)->Unit) {
        ...
    }

    fun leaveChoir(whatNotToCall: (Int)->Unit) {
        ...
    }

    fun conduct(n: Times) {
        for (p in participants.values) {
            p(n)
        }
    }
}
```

我们的所有订阅者看起来都像火鸡：

```kt
class Turkey {
    fun gobble(repeat: Times) {
        for (i in 1..repeat) {
            println("Gob-gob")
        }
    }
}
```

实际上，这有点问题。如果`Cat`要告诉每个动物发出什么声音：高音还是低音？我们不得不再次更改所有订阅者，以及`Cat`本身。

在设计你的发布者时，传递具有许多属性的单个数据类，而不是数据类的集合或其他类型。这样，如果添加了新的属性，你就不必重构你的订阅者：

```kt
enum class SoundPitch {HIGH, LOW}
data class Message(val repeat: Times, val pitch: SoundPitch)

class Bat {
    fun screech(message: Message) {
        for (i in 1..message.repeat) {
            println("${message.pitch} Eeeeeee")
        }
    }
}
```

确保你的消息是不可变的。否则，你可能会遇到奇怪的行为！

如果你有来自同一发布者的不同消息集合，你会怎么办？

使用智能转换：

```kt
interface Message {
    val repeat: Times
    val pitch: SoundPitch 
}

data class LowMessage(override val repeat: Times) : Message {
    override val pitch = SoundPitch.LOW
}

data class HighMessage(override val repeat: Times) : Message {
    override val pitch = SoundPitch.HIGH
}

class Bat {
    fun screech(message: Message) {
        when (message) {
            is HighMessage -> {
                for (i in 1..message.repeat) {
                    println("${message.pitch} Eeeeeee")
                }
            }
            else -> println("Can't :(")
        }
    }
}
```

# 摘要

这是一章很长的内容。但我们同时也学到了很多。我们已经完成了对所有经典设计模式的覆盖，包括十一个行为模式。在 Kotlin 中，函数可以被传递给其他函数，从函数中返回，以及分配给变量。这就是“函数作为一等公民”的概念所在。如果你的类主要是关于行为，那么用函数来替换它通常是有意义的。迭代器是语言中的另一个 `operator`。密封类有助于使 `when` 语句变得详尽。`run` 扩展函数允许控制从它返回的内容。带有接收器的 lambda 在你的 DSLs 中提供了更清晰的语法。另一个关键字 `lateinit` 告诉编译器在它的空安全检查上稍微放松一些。请谨慎使用！最后，我们介绍了如何使用 `::` 引用现有方法。

在下一章中，我们将从具有众所周知设计模式的面向对象编程范式转向另一种范式——函数式编程。
