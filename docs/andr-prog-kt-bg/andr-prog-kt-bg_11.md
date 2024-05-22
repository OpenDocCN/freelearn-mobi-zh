# 第十一章：Kotlin 中的继承

在本章中，我们将看到继承的实际应用。实际上，我们已经看到了，但现在我们将更仔细地研究它，讨论其好处，并编写我们可以继承的类。在整个章节中，我将向您展示几个继承的实际例子，并在本章结束时改进我们在上一章中的海战模拟，并展示我们如何通过使用继承来节省大量的输入和未来的调试。

在本章中，我们将涵盖以下主题：

+   **面向对象编程**（**OOP**）和继承

+   使用开放类进行继承

+   重写函数

+   关于多态性的更多内容

+   抽象类

+   继承示例应用程序

让我们开始，让我们再多谈一点理论。

# OOP 和继承

我们已经看到了如何通过实例化/创建对象从类中重用我们自己的代码和其他人的代码。但是整个 OOP 的概念甚至比这更深入。

如果有一个类中有大量有用的功能，但不完全符合我们的要求怎么办？想想我们编写`Carrier`类时的情况。它与`Destroyer`类非常接近，我们几乎可以复制粘贴它。我们可以从一个类**继承**，然后进一步完善或添加其工作方式和功能。

您可能会惊讶地听到我们已经这样做了。实际上，我们已经对我们创建的每个应用程序都这样做了。当我们使用`:`语法时，我们正在继承。您可能还记得`MainActivity`类中的这段代码：

```kt
class MainActivity : AppCompatActivity() {
```

在这里，我们从`AppCompatActivity`类继承了所有功能-或者更具体地说，类的设计者希望我们能够访问的所有功能。

我们甚至可以重写一个函数，并在一定程度上依赖于我们继承的类中的重写函数。例如，每次我们继承`AppCompatActivity`类时，我们都重写了`onCreate`函数。但是当我们这样做时，我们也调用了类设计者提供的默认实现：

```kt
super.onCreate(... 
```

### 提示

`super`关键字指的是被继承的超类。

而且，在第六章中，*Android 生命周期*，我们重写了`Activity`类的许多生命周期函数。请注意，您可以有多个级别的继承，尽管良好的设计通常建议不要有太多级别。例如，我已经提到`AppCompatActivity`继承自`Activity`，而我们又从`AppCompatActivity`继承。

有了这个想法，让我们看一些示例类，并看看我们如何扩展它们，只是为了看到语法，作为第一步，并且能够说我们已经做到了。

# 使用开放类进行继承

在这一点上，学习的一些有用术语是被继承的类被称为**超类**或**基类**。其他常见的称呼这种关系的方式是**父**类和**子**类。子类继承自父类。

默认情况下，类不能被继承。它被称为**final**类-不开放用于扩展或继承。但是，很容易将类更改为可继承的。我们只需要在类声明中添加`open`关键字。

## 基本继承示例

看看下面的代码，它使用`open`关键字与类声明，并使该类可以被继承：

```kt
open class Soldier() {

    fun shoot () {
        Log.i("Action","Bang bang bang")
    }
}
```

### 提示

本章中的所有示例都可以在`Chapter11/Chapter Examples`文件夹中找到。

现在我们可以继续创建`Soldier`类型的对象并调用`shoot`函数，就像下面的代码一样：

```kt
val soldier = Soldier()
soldier.shoot()
```

前面的代码仍然会将`Bang bang bang`输出到 logcat 窗口；我们不必继承它才能使用它。然而，如果我们想要精细化或专门化我们对`Soldier`类的使用，我们可以创建一个专门类型的`Soldier`并继承`shoot`函数。我们可以创建更多的类，也许`Special Forces`和`Paratrooper`，并使用`:`语法从`Soldier`继承。以下是`SpecialForces`类的代码：

```kt
class SpecialForces: Soldier(){
    fun SneakUpOnEnemy(){
        Log.i("Action","Sneaking up on enemy")
    }
}
```

注意使用冒号表示继承。它还添加了一个`sneakUpOnEnemy`函数。

接下来，考虑`Paratrooper`类的以下代码：

```kt
class Paratrooper: Soldier() {
    fun jumpOutOfPlane() {
        Log.i("Action", "Jump out of plane")
    }
}
```

前面的代码还使`Paratrooper`从`Soldier`继承，并添加了`jumpOutOfPlane`函数。

这是我们如何使用这两个新的子类的：

```kt
val specialForces = SpecialForces()
specialForces.shoot()
specialForces.SneakUpOnEnemy()

val paratrooper = Paratrooper()
paratrooper.shoot()
paratrooper.jumpOutOfPlane()
```

在前面的代码中，我们实例化了一个`SpecialForces`实例和一个`Paratrooper`实例。该代码演示了两个实例都可以访问基类中的`shoot`函数，并且两个类都可以访问自己的专门函数。代码的输出将如下所示：

```kt
Action: Bang bang bang
Action: Sneaking up on enemy
Action: Bang bang bang
Action: Jump out of plane

```

继承还有更多内容。让我们看看当我们需要进一步完善基类/超类的功能时会发生什么。

## 重写函数

重写函数是我们已经做过的事情，但我们需要进一步讨论。我们已经在我们编写的每个应用程序中重写了`onCreate`函数，并且在第六章中，*Android 生命周期*，我们重写了`AppCompatActivity`类的许多其他函数。

考虑一下我们可能想要添加一个`Sniper`类。起初这可能看起来很简单。只需编写一个类，继承自`Soldier`，并添加一个`getIntoPosition`函数，也许。如果我们想让`Sniper`类的射击方式与普通的`Soldier`不同怎么办？看看`Sniper`类的以下代码，它重写了`shoot`函数，并用`Sniper`类的专门版本替换了它：

```kt
class Sniper: Soldier(){
    override fun shoot(){
        Log.i("Action","Steady… Adjust for wind… Bang.")
    }

    fun getIntoPosition(){
        Log.i("Action","Preparing line of sight to target")
    }
}
```

你可能会认为工作已经完成，但这会导致一个小问题。在`Sniper`类中有一个错误，如下一个截图所示：

![重写函数](img/B12806_11_01.jpg)

错误是因为`shoot`函数没有被写成可以被重写。默认情况下，函数是 final 的，就像类一样。这意味着子类必须按原样使用它。解决方案是回到`Soldier`类并在`shoot`函数声明前面添加`open`关键字。以下是带有微妙但至关重要的添加的`Soldier`类的更新代码：

```kt
open class Soldier() {

    open fun shoot () {
        Log.i("Action","Bang bang bang")
    }
}
```

现在我们已经修复了错误，可以编写以下代码来实例化`Sniper`类并使用重写的`shoot`函数：

```kt
val sniper = Sniper()
sniper.shoot()
sniper.getIntoPosition()
```

这产生了以下输出：

```kt
Action: Steady… Adjust for wind… Bang.
Action: Preparing line of sight to target

```

我们可以看到已使用重写的函数。值得注意的是，即使子类重写了父类的函数，它仍然可以使用父类的函数。考虑一下，如果狙击手的狙击步枪用完了，需要切换到其他武器会发生什么。看看`Sniper`类的重新编写代码：

```kt
class Sniper: Soldier(){
    // He forget to bring enough ammo
    var sniperAmmo = 3

    override fun shoot(){
        when (sniperAmmo > 0) {
            true -> {
                Log.i("Action", "Steady… Adjust for wind… Bang.")
                sniperAmmo--;
            }
            false -> super.shoot()
        }
    }

    fun getIntoPosition(){
        Log.i("Action","Preparing line of sight to target")
    }
}
```

在`Sniper`类的新版本中，有一个名为`sniperAmmo`的新属性，并且初始化为`3`。重写的`shoot`函数现在使用`when`表达式来检查`sniperAmmo`是否大于零。如果大于零，则通常的文本将被打印到 logcat 窗口，并且`sniperAmmo`将被递减。这意味着表达式只会返回三次 true。`when`表达式还处理了当它为 false 时会发生什么，并调用`super.shoot()`。这行代码调用`Soldier`的`shoot`函数的版本-超类。

现在，我们可以尝试在`Sniper`实例上四次调用`shoot`函数，就像以下代码中的方式，并观察发生了什么：

```kt
val sniper = Sniper()
sniper.getIntoPosition()
sniper.shoot()
sniper.shoot()
sniper.shoot()
// Damn! where did I put that spare ammo
sniper.shoot()
```

这是我们从前面的代码中得到的输出：

```kt
Action: Preparing line of sight to target
Action: Steady… Adjust for wind… Bang.
Action: Steady… Adjust for wind… Bang.
Action: Steady… Adjust for wind… Bang.
Action: Bang bang bang

```

我们可以看到前三次调用`sniper.shoot()`都从`Sniper`类中重写的`shoot`函数输出，第四次仍然调用重写版本，但`when`表达式的`false`分支调用超类版本的`shoot`，我们从`Soldier`类中得到输出。

### 提示

到目前为止，基于继承的示例的工作项目可以在代码下载的`Chapter11`文件夹中找到。它被称为`Inheritance Examples`。

## 到目前为止的总结

好像面向对象编程还不够有用，我们现在可以模拟现实世界的对象。我们还看到，通过从其他类进行子类化/扩展/继承，我们可以使面向对象编程变得更加有用。

### 提示

通常情况下，我们可能会问自己这个关于继承的问题：为什么？原因大致如下：如果我们在父类中编写通用代码，那么我们可以更新该通用代码，所有继承它的类也将被更新。此外，我们可以通过可见性修饰符来辅助封装，因为子类只能使用公共/受保护的实例变量和函数，并且只能重写开放函数。因此，如果设计得当，这也进一步增强了封装的好处。

# 更多多态性

我们已经知道多态意味着许多形式，但对我们来说意味着什么呢？

简化到最简单的形式，意味着以下内容：

### 注意

任何子类都可以作为使用超类的代码的一部分。

这意味着我们可以编写更容易理解、更容易更改的代码。

此外，我们可以为超类编写代码，并依赖于无论它被子类化多少次，代码仍将在一定范围内工作。让我们讨论一个例子。

假设我们想使用多态来帮助编写一个动物园管理应用程序。我们可能会想要有一个函数，比如`feed`。我们还可以说我们有`Lion`，`Tiger`和`Camel`类，它们都继承自一个名为`Animal`的父类。我们可能还想将要喂食的动物的引用传递给`feed`函数。这似乎意味着我们需要为每种类型的`Animal`编写一个 feed 函数。

然而，我们可以使用多态参数编写多态函数：

```kt
fun feed(animalToFeed: Animal){
   // Feed any animal here
}
```

前面的函数有`Animal`作为参数，这意味着可以将从继承自`Animal`的类构建的任何对象传递给它。

因此，您甚至可以今天编写代码，然后在一周、一个月或一年后创建另一个子类，相同的函数和数据结构仍然可以工作。

此外，我们可以对我们的子类强制执行一组规则，规定它们可以做什么，不能做什么，以及如何做。因此，在一个阶段的巧妙设计可以影响其他阶段。

但我们真的会想要实例化一个实际的`Animal`吗？

## 抽象类和函数

抽象函数是使用`abstract`关键字声明的函数。到目前为止还没有问题。但是，抽象函数也根本没有主体。明确地说，抽象函数中没有任何代码。那么，我们为什么要这样做呢？答案是，当我们编写抽象函数时，我们强制任何从具有抽象函数的类继承的类来实现/重写该函数。以下是一个假设的抽象函数：

```kt
abstract fun attack(): Int
```

没有主体，没有代码，甚至没有空花括号。任何想要从该类继承的类必须以与前面声明的签名完全相同的方式实现`attack`函数。

`abstract`类是一个不能被实例化的类-不能成为对象。那么，这就是一个永远不会被使用的蓝图？但这就像支付一个建筑师来设计你的家，然后永远不建造它！您可能会对自己说：“我有点明白抽象函数的概念，但抽象类只是愚蠢。”

如果一个类的设计者想要强制类的用户在使用他们的类之前继承，他们可以将一个类声明为`abstract`。然后，我们就不能从中创建对象；因此，我们必须先继承它，然后从子类创建对象。

让我们看一个例子。我们通过使用`abstract`关键字声明一个类为`abstract`类，像这样：

```kt
abstract class someClass{
   /*
         All functions and properties here.
         As usual!
         Just don't try and make 
         an object out of me!
   */
}
```

是的，但为什么呢？

有时我们想要一个可以用作多态类型的类，但我们需要保证它永远不能被用作对象。例如，`Animal`本身并没有太多意义。

我们不谈论动物，我们谈论*动物的类型*。我们不会说，“哦，看那只可爱的毛茸茸的白色动物”，或者，“昨天我们去宠物店买了一只动物和一个动物床。”这太抽象了。

因此，`abstract`类有点像一个模板，可以被任何继承自它的类使用。

我们可能想要一个`Worker`类，并扩展此类以创建`Miner`、`Steelworker`、`OfficeWorker`，当然还有`Programmer`。但是一个普通的`Worker`到底是做什么的呢？为什么我们会想要实例化一个呢？

答案是我们不想实例化一个，但我们可能想要将其用作多态类型，以便我们可以在函数之间传递多个工作子类，并且可以容纳所有类型的`Worker`的数据结构。

我们称这种类为抽象类，当一个类有一个抽象函数时，它必须被声明为抽象类。所有抽象函数必须被任何继承自它的类重写。

这意味着抽象类可以提供一些在其所有子类中都可用的常见功能。例如，`Worker`类可能具有`height`、`weight`和`age`属性。

它可能还有`getPayCheck`函数，这个函数不是抽象的，在所有子类中都是相同的，但是`doWork`函数是抽象的，必须被重写，因为所有不同类型的工作者都有非常不同的`doWork`。

# 使用继承示例应用程序的类

我们已经看过了我们可以创建类的层次结构来模拟适合我们应用程序的系统的方式。因此，让我们构建一个项目，以改进我们在上一章中进行的海战。

使用空活动模板创建一个名为`Basic Classes with Inheritance Example`的新项目。如你所料，完成的代码可以在`Chapter11`文件夹中找到。

这就是我们要做的：

+   将`Carrier`和`Destroyer`类的大部分功能放入`Ship`超类中。

+   为`Carrier`和`Destroyer`类都继承自`Ship`类，从而节省大量代码维护。

+   使用多态性来调整`Shipyard`类中的`serviceShip`函数，使其以`Ship`作为参数，从而可以为继承自`Ship`的任何实例提供服务，从而减少类中的函数数量。

+   我们还将看到，不仅代码量比以前少，而且封装性也比以前更好。

创建一个名为`Ship`的新类，并编写如下代码。然后我们将讨论它与上一个项目中的`Destroyer`和`Carrier`类的比较：

```kt
abstract class Ship(
        val name: String,
        private var type: String,
        private val maxAttacks: Int,
        private val maxHullIntegrity: Int) {

    // The stats that all ships have
    private var sunk = false
    private var hullIntegrity: Int
    protected var attacksRemaining: Int

    init{
        hullIntegrity = this.maxHullIntegrity
        attacksRemaining = 1
    }

    // Anything can use this function
    fun takeDamage(damageTaken: Int) {
        if (!sunk) {
            hullIntegrity -= damageTaken
            Log.i("$name damage taken =","$damageTaken")
            Log.i("$name hull integrity =","$hullIntegrity")

            if (hullIntegrity <= 0) {
                Log.i(type, "$name has been sunk")
                sunk = true
            }
        } else {
            // Already sunk
            Log.i("Error", "Ship does not exist")
        }
    }

    fun serviceShip() {
        attacksRemaining = maxAttacks
        hullIntegrity = maxHullIntegrity
    }

    fun showStats(){
        Log.i("$type $name",
                "Attacks:$attacksRemaining - Hull:$hullIntegrity")
    }

    abstract fun attack(): Int

}
```

首先，你会注意到这个类被声明为`abstract`，所以我们知道我们必须从这个类继承，而不能直接使用它。向下扫描代码，你会看到一个名为`attack`的抽象函数。我们现在知道，当我们从`Ship`继承时，我们需要重写并提供一个名为`attack`的函数的代码。这正是我们需要的，因为你可能记得航空母舰发动攻击，驱逐舰发射炮弹。

向上扫描前面的代码，你会看到构造函数声明了四个属性。其中两个属性是新的，另外两个与之前的项目具有相同的用途，但我们如何调用构造函数才是有趣的，我们很快就会看到。

两个新属性是`maxAttacks`和`maxHullIntegrity`，这样`Shipyard`就可以将它们恢复到适合特定类型船只的水平。

在`init`块中，未在构造函数中初始化的属性被初始化。接下来是`takeDamage`函数，它具有与上一个项目中的`takeDamage`函数相同的功能，只是它只在`Ship`类中，而不是在`Carrier`和`Destroyer`类中。

最后，我们有一个`showStats`函数，用于将与日志窗口相关的统计值打印出来，这意味着这些属性也可以是私有的。

请注意，除了`name`和一个叫做`attacksRemaining`的受保护属性之外，所有属性都是私有的。请记住，`protected`意味着它只在继承自`Ship`类的实例内可见。

现在，按照下面所示的方式编写新的`Destroyer`类：

```kt
class Destroyer(name: String): Ship(
        name,
        "Destroyer",
        10,
        200) {

    // No external access whatsoever
    private var shotPower = 60

    override fun attack():Int {
        // Let the calling code no how much damage to do
        return if (attacksRemaining > 0) {
            attacksRemaining--
            shotPower
        }else{
            0
        }
    }
}
```

现在，按照下面所示的方式编写`Carrier`类，然后我们可以比较`Destroyer`和`Carrier`：

```kt
class Carrier (name: String): Ship(
        name,
        "Carrier",
        20,
        100){

    // No external access whatsoever
    private var attackPower = 120

    override fun attack(): Int {
        // Let the calling code no how much damage to do
        return if (attacksRemaining > 0) {
            attacksRemaining--
            attackPower
        }else{
            0
        }
    }
}
```

请注意，前面两个类只接收一个名为`name`的`String`值作为构造函数参数。您还会注意到`name`没有用`val`或`var`声明，因此它不是一个属性，只是一个不会持久存在的临时参数。每个类的第一件事是继承自`Ship`并调用`Ship`类的构造函数，同时传入适用于`Destroyer`或`Carrier`的值。

两个类都有与攻击相关的属性。`Destroyer`有`shotPower`，`Carrier`有`attackPower`。然后它们都实现/重写`attack`函数以适应它们将执行的攻击类型。但是，两种类型的攻击将以相同的方式通过相同的函数调用触发。

按照下面所示的方式编写新的`Shipyard`类：

```kt
class ShipYard {
    fun serviceShip(shipToBeServiced: Ship){
        shipToBeServiced.serviceShip()
        Log.i("Servicing","${shipToBeServiced.name}")
    }
}
```

在`Shipyard`类中，现在只有一个函数。它是一个多态函数，以`Ship`实例作为参数。然后调用超类的`serviceShip`函数，该函数将将弹药/攻击和`hullIntegrity`恢复到适合船只类型的水平。

### 提示

`Shipyard`类是肤浅的这一说法是正确的。我们本可以直接调用`serviceShip`而不将实例传递给另一个类。但是，这清楚地表明我们可以将两个不同的类视为相同类型，因为它们都继承自相同的类型。多态的概念甚至比这更深入，我们将在下一章中讨论接口时看到。毕竟，多态意味着许多事物，而不仅仅是两件事物。

最后，在`MainActivity`类的`onCreate`函数中添加代码，让我们的辛勤工作付诸实践：

```kt
val friendlyDestroyer = Destroyer("Invincible")
val friendlyCarrier = Carrier("Indomitable")

val enemyDestroyer = Destroyer("Grey Death")
val enemyCarrier = Carrier("Big Grey Death")

val friendlyShipyard = ShipYard()

// A small battle
friendlyDestroyer.takeDamage(enemyDestroyer.attack())
friendlyDestroyer.takeDamage(enemyCarrier.attack())
enemyCarrier.takeDamage(friendlyCarrier.attack())
enemyCarrier.takeDamage(friendlyDestroyer.attack())

// Take stock of the supplies situation
friendlyDestroyer.showStats()
friendlyCarrier.showStats()

// Dock at the shipyard
friendlyShipyard.serviceShip(friendlyCarrier)
friendlyShipyard.serviceShip(friendlyDestroyer)

// Take stock of the supplies situation
friendlyDestroyer.showStats()
friendlyCarrier.showStats()

// Finish off the enemy
enemyDestroyer.takeDamage(friendlyDestroyer.attack())
enemyDestroyer.takeDamage(friendlyCarrier.attack())
enemyDestroyer.takeDamage(friendlyDestroyer.attack())
```

这段代码完全遵循与以下相同的模式：

1.  攻击友方船只

1.  反击并击沉敌方航母

1.  打印统计数据

1.  造船厂进行修理和重新武装

1.  再次打印统计数据

1.  完成最后一个敌人

现在我们可以观察输出：

```kt
Invincible damage taken =: 60
Invincible hull integrity =: 140
Invincible damage taken =: 120
Invincible hull integrity =: 20
Big Grey Death damage taken =: 120
Big Grey Death hull integrity =: -20
Carrier: Big Grey Death has been sunk
Error: Ship does not exist
Destroyer Invincible: Attacks:0 - Hull:20
Carrier Indomitable: Attacks:0 - Hull:100
Servicing: Indomitable
Servicing: Invincible
Destroyer Invincible: Attacks:10 - Hull:200
Carrier Indomitable: Attacks:20 - Hull:100
Grey Death damage taken =: 60
Grey Death hull integrity =: 140
Grey Death damage taken =: 120
Grey Death hull integrity =: 20
Grey Death damage taken =: 60
Grey Death hull integrity =: -40
Destroyer: Grey Death has been sunk

```

在前面的输出中，我们可以看到几乎相同的输出。但是，我们用更少的代码和更多的封装实现了它，而且，如果在六个月后我们需要一个使用鱼雷进行攻击的`Submarine`类，那么我们可以在不更改任何现有代码的情况下添加它。

# 总结

如果您没有记住所有内容，或者有些代码看起来有点太深入了，那么您仍然成功了。

如果你只是理解 OOP 是通过封装、继承和多态编写可重用、可扩展和高效的代码，那么你就有成为 Kotlin 大师的潜力。

简而言之，OOP 使我们能够使用其他人的代码，即使那些其他人在编写代码时并不知道我们当时会做什么。

你所需要做的就是不断练习，因为我们将在整本书中一遍又一遍地使用这些概念，所以你不需要在这一点上甚至已经掌握它们。

在下一章中，我们将重新审视本章的一些概念，以及探讨面向对象编程的一些新方面，以及它如何使我们的 Kotlin 代码与 XML 布局进行交互。
