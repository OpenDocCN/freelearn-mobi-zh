# 第十章。面向对象编程

在本章中，我们将发现，在 Kotlin 中，类对几乎所有事情都是基础的，实际上，几乎所有事情都是一个类。

我们已经谈到了重用他人的代码，特别是 Android API，但在本章中，我们将真正掌握这是如何工作的，并学习**面向对象编程**（**OOP**）以及如何使用它。

在本章中，我们将涵盖以下主题：

+   介绍 OOP 和封装、多态和继承的三个关键主题

+   基本类，包括如何编写我们的第一个类，包括为数据/变量封装添加**属性**和函数以完成任务

+   探索**可见性修饰符**，进一步帮助和完善封装。

+   了解**构造函数**，使我们能够快速准备我们的类以转换为可用的对象/实例

+   编写一个基本的类小应用程序，以实践我们在本章学到的一切

如果你试图记住本章（或下一章），你将不得不在你的大脑中腾出很多空间，而且你可能会忘记一些非常重要的东西。

一个很好的目标是尽量理解它。这样，你的理解将变得更加全面。在需要时，你可以参考本章（和下一章）进行复习。

### 提示

如果你对本章或下一章的内容并不完全理解也没关系！继续阅读，并确保完成所有的应用程序。

# 介绍 OOP

在第一章中，*开始使用 Android 和 Kotlin*，我们提到 Kotlin 是一种面向对象的语言。面向对象的语言要求我们使用 OOP；这不是可选的额外部分，而是 Kotlin 的一部分。

让我们多了解一点。

## OOP 到底是什么？

OOP 是一种编程方式，它涉及将我们的需求分解成比整体更易管理的块。

每个块都是自包含的，并且可能被其他程序重用，同时与其他块一起工作。

这些块就是我们所说的对象。当我们计划/编写一个对象时，我们使用一个类。类可以被看作是对象的蓝图。

我们实现了一个类的对象。这被称为类的**实例**。想想一个房子的蓝图——你不能住在里面，但你可以建造一座房子；所以，你建造了它的一个实例。通常，当我们为我们的应用程序设计类时，我们写它们来代表现实世界的事物。

然而，OOP 不仅仅是这样。它也是一种做事情的方式——一种定义最佳实践的方法。

OOP 的三个核心原则是**封装**、**多态**和**继承**。这些听起来可能很复杂，但一步一步来说，都是相当简单的。

### 封装

**封装**意味着通过允许你选择的变量和函数来访问，使你的代码的内部工作免受使用它的代码的干扰。

这意味着你的代码可以随时更新、扩展或改进，而不会影响使用它的程序，只要暴露的部分仍然以相同的方式访问。

你可能还记得来自第一章的这行代码，*开始使用 Android 和 Kotlin*：

```kt
locationManager.getLastKnownLocation(LocationManager.GPS_PROVIDER)
```

通过适当的封装，如果卫星公司或 Android API 团队需要更新他们的代码工作方式，也不要紧。如果`getLastKnownLocation`函数签名保持不变，我们就不必担心内部发生了什么。我们在更新之前编写的代码在更新后仍将正常工作。

如果一辆汽车的制造商去掉了车轮，将其变成了电动悬浮汽车，如果它仍然有方向盘、油门和刹车踏板，驾驶它不应该是一个挑战。

当我们使用 Android API 的类时，我们是按照 Android 开发人员设计他们的类的方式来使用的。

在本章中，我们将深入探讨封装。

### 多态性

多态性使我们能够编写的代码不太依赖于我们试图操作的类型，使我们的代码更清晰、更高效。多态性意味着**多种形式**。如果我们编码的对象可以是多种类型的东西，那么我们就可以利用这一点。一些未来的例子将会让这一点更加清晰。类比会让你更加真实地理解。如果我们有汽车工厂，只需改变给机器人的指令和装配线上的零件，就可以制造货车和小型卡车，那么这个工厂就是多态的。

如果我们能够编写能够处理不同类型数据的代码而无需重新开始，这不是很有用吗？我们将在第十一章中看到一些例子，*Kotlin 中的继承*。

我们还将在第十二章中了解更多关于多态性的内容，*Kotlin 与 UI 和空值的连接*。

### 继承

正如它听起来的那样，**继承**意味着我们可以利用其他人的类的所有特性和好处（包括封装和多态性），同时进一步调整他们的代码以适应我们的情况。实际上，我们已经这样做了，每次使用`:`运算符时：

```kt
class MainActivity : AppCompatActivity() {
```

`AppCompatActivity`类本身继承自`Activity`。因此，每次创建新的 Android 项目时，我们都继承自`Activity`。我们可以做得更多，我们将看到这是如何有用的。

想象一下，世界上最强壮的男人和最聪明的女人在一起。他们的孩子很有可能会从基因遗传中获得重大好处。Kotlin 中的继承让我们可以用另一个人的代码和我们自己的代码做同样的事情。

我们将在下一章中看到继承的实际应用。

## 为什么要这样做？

当小心使用时，所有这些面向对象编程允许你添加新功能，而不太担心它们如何与现有功能交互。当你必须更改一个类时，它的自包含（封装）性质意味着对程序的其他部分的影响较小，甚至可能为零。这就是封装的部分。

你可以使用其他人的代码（如 Android API），而不知道甚至可能不关心它是如何工作的。想想一下 Android 生命周期、`Toast`、`Log`、所有的 UI 小部件、监听卫星等等。我们不知道，也不需要知道它们内部是如何工作的。更详细的例子是，`Button`类有将近 50 个函数 - 我们真的想要为一个按钮自己写这么多吗？最好使用别人的`Button`类。

面向对象编程使你能够轻松地为高度复杂的情况编写应用程序。

通过继承，你可以创建类的多个相似但不同的版本，而无需从头开始编写类，并且由于多态性，你仍然可以使用原始类型对象的函数来处理新对象。

这真的很有道理。而且 Kotlin 从一开始就考虑到了所有这些，所以我们被迫使用所有这些面向对象编程 - 然而，这是一件好事。让我们快速回顾一下类。

## 类回顾

类是一堆代码的容器，可以包含函数、变量、循环和我们已经学过的其他 Kotlin 语法。类是 Kotlin 包的一部分，大多数包通常会有多个类。通常情况下，尽管不总是如此，每个新类都将在其自己的`.kt`代码文件中定义，文件名与类名相同，就像我们迄今为止所有基于活动的类一样。

一旦我们编写了一个类，我们就可以使用它来创建任意数量的对象。记住，类是蓝图，我们根据蓝图制作对象。房子不是计划，就像对象不是类一样-它是从类制作的对象。对象是一个引用变量，就像一个字符串，稍后我们将发现引用变量的确切含义。现在，让我们看一些实际的代码。

# 基本类

类涉及两个主要步骤。首先，我们必须声明我们的类，然后我们可以通过实例化它将其变成一个实际可用的对象。记住，类只是一个蓝图，你必须使用蓝图来构建一个对象，然后才能对其进行任何操作。

## 声明类

类可以根据其目的的不同而具有不同的大小和复杂性。这是一个类声明的绝对最简单的例子。

记住，我们通常会在一个与类同名的文件中声明一个新的类。

### 注意

在本书的其余部分，我们将介绍一些例外情况。

让我们看看声明类的三个例子：

```kt
// This code goes in a file named Soldier.kt
class Soldier

// This code would go in a file called Message.kt
class Message

// This code would go in a file called ParticleSystem.kt
class ParticleSystem
```

### 提示

请注意，我们将在本章结束时进行一个完整的工作项目练习。在下载包的`Chapter10/Chapter Example Classes`文件夹中，还有本章中所有理论示例的完整类。

在上面的代码中要注意的第一件事是，我已经将三个类声明合并在一起。在真实的代码中，每个声明都应该包含在自己的文件中，文件名与类名相同，扩展名为`.kt`。

要声明一个类，我们使用`class`关键字，后面跟着类的名称。因此，我们可以得出结论，在前面的代码中，我们声明了一个名为`Soldier`的类，一个名为`Message`的类，以及一个名为`ParticleSystem`的类。

我们已经知道，类可以并且经常模拟现实世界的事物。因此，可以安全地假设这三个假设的类将模拟一个士兵（也许来自游戏）、一条消息（也许来自电子邮件或短信应用程序）和一个粒子系统（也许来自科学模拟应用程序）。

### 注意

粒子系统是一个包含个体粒子的系统，这些粒子作为该系统的一部分。在计算中，它们用于模拟/可视化化学反应/爆炸和粒子行为，也许是烟雾等事物。在第二十一章中，*线程和启动实时绘图应用程序*，我们将构建一个使用粒子系统使用户的绘画看起来活灵活现的酷炫绘图应用程序。

然而，很明显，像我们刚刚看到的三个简单声明并不包含足够的代码来实现任何有用的功能。我们将在一会儿扩展类声明。首先，让我们看看如何使用我们声明的类。

## 实例化类

要从我们的类构建一个可用的对象，我们需要转到另一个代码文件。到目前为止，在整本书中，我们已经使用`AppCompatActivity`类中的`onCreate`函数来演示不同的概念。虽然你可以在 Android 的任何地方实例化一个类，但由于生命周期函数的存在，通常会使用`onCreate`来实例化我们的类的对象/实例。

看一下以下代码。我已经突出了要关注的新代码：

```kt
class MainActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

         // Instantiating one of each of our classes
             val soldier = Soldier()
 val message = Message()
 val particleSystem = ParticleSystem()    

   } // End of onCreate function

}// End of MainActivity class
```

在前面的代码中，我们实例化了三个先前声明的类的实例（创建了一个可用的对象）。让我们更仔细地研究一下语法。这是实例化`Soldier`类的代码行：

```kt
val soldier = Soldier()
```

首先，我们决定是否需要更改我们的实例。与常规变量一样，我们选择`val`或`var`。接下来，我们给我们的实例命名。在前面的代码中，对象/实例被称为`soldier`，但我们也可以称之为`soldierX`，`marine`，`john117`，甚至`squashedBanana`。名称是任意的，但与变量一样，给它们起一个有意义的名字是有意义的。此外，与变量一样，按照惯例，但不是必须的，以小写字母开头的名称和名称中的任何后续单词的首字母大写。

### 注意

在使用它们来声明类的实例时，`val`和`var`之间的区别更加微妙和重要。我们将首先学习有关类的细节，在第十二章中，*将我们的 Kotlin 连接到 UI 和可空性*，我们将重新讨论`val`和`var`，以了解我们的实例底层发生了什么。

代码的最后部分包含赋值运算符`=`，后面跟着类名`Soldier`，以及一对开放和关闭的括号`()`。

赋值运算符告诉 Kotlin 编译器将代码右侧的结果赋给左侧的变量。类型推断确定`soldier`是`Soldier`类型。

类名后面那个看起来奇怪但也许熟悉的`()`暗示着我们在调用一个函数。我们确实在调用一个特殊的函数，称为**构造函数**，它是由 Kotlin 编译器提供的。关于构造函数有很多要讨论的，所以我们将把这个话题推迟到本章稍后。

现在，我们只需要知道，下一行代码创建了一个名为`soldier`的`Soldier`类型的可用对象：

```kt
val soldier = Soldier()
```

记住，面向对象编程的目标之一是我们可以重用我们的代码。我们不仅限于只有一个`Soldier`类型的对象。我们可以有任意多个。看看下面的代码块：

```kt
val soldier1 = Soldier()
val soldier2 = Soldier()
val soldier3 = Soldier()
```

`soldier1`，`soldier2`和`soldier3`实例都是独立的、不同的实例。它们都是同一类型 - 但这是它们唯一的联系。你和你的邻居可能都是人类，但你们不是同一个人。如果我们对`soldier1`做了什么，或者改变了`soldier1`的某些东西，那么这些操作只会影响`soldier1`。`soldier2`和`soldier3`实例不受影响。事实上，我们可以实例化一整支`Soldier`对象的军队。

面向对象编程的力量正在慢慢显现，但在我们讨论的这个阶段，房间里的大象是，我们的类实际上并没有做任何事情。此外，我们的实例不持有任何值（数据），因此我们也无法对它们进行任何更改。

## 类有函数和变量（有点）

当我们在本章后面的*类变量是属性*部分时，我将很快解释略微神秘的**（有点）**标题。

我们在讨论 Kotlin 时学到的任何代码都可以作为类的一部分使用。这就是我们使我们的类有意义，使我们的实例真正有用的方法。让我们扩展类声明并添加一些变量和函数。

### 使用类的变量

首先，我们将向我们空的`Soldier`类添加一些变量，就像下面的代码一样：

```kt
class Soldier{

    // Variables
    val name = "Ryan"
    val rank = "Private"
    val missing = true
}
```

记住，所有前面的代码都将放在一个名为`Soldier.kt`的文件中。现在我们有了一个带有一些成员变量的类声明，我们可以像下面的代码中所示那样使用它们：

```kt
// First declare an instance of Soldier called soldier1
val soldier1 = Soldier()

// Now access and print each of the variables  
Log.i("Name =","${soldier1.name}")
Log.i("Rank =","${soldier1.rank}")
Log.i("Missing =","${soldier1.missing}")
```

如果将代码放在`onCreate`函数中，将在 logcat 窗口中产生以下输出：

```kt
Name =: Ryan
Rank =: Private
Missing =: true

```

在前面的代码中，我们以通常的方式实例化了`Soldier`类的一个实例。但现在，因为`Soldier`类有一些带有值的变量，我们可以使用**点语法**来访问这些值：

```kt
instanceName.variableName
```

或者，我们可以通过使用这个具体的例子来访问这些值：

```kt
soldier1.name
soldier1.rank
// Etc..
```

要清楚的是，我们使用实例名称，而不是类名称：

```kt
Soldier.name // ERROR!
```

### 提示

通常情况下，我们将在继续进行时涵盖一些例外和变化。

如果我们想要更改变量的值，我们可以使用完全相同的点语法。当然，如果你回想起第七章中讲到的，*Kotlin 变量、运算符和表达式*，可以更改的变量需要声明为`var`，而不是`val`。这是重新设计的`Soldier`类，以便我们可以稍微不同地使用它：

```kt
class Soldier{

    // Member variables
    var name = "Ryan"
    var rank = "Private"
    var missing = true
}
```

现在，我们可以使用点语法来操纵变量的值，就像它们是常规的`var`变量一样：

```kt
// First declare an instance of Soldier called soldier1
val soldier1 = Soldier()

// Now access and print each of the variables  
Log.i("Name =","${soldier1.name}")
Log.i("Rank =","${soldier1.rank}")
Log.i("Missing =","${soldier1.missing}")

// Mission to rescue Private Ryan succeeds
soldier1.missing = false;

// Ryan behaved impeccably
soldier1.rank = "Private First Class"

// Now access and print each of the variables  
Log.i("Name =","${soldier1.name}")
Log.i("Rank =","${soldier1.rank}")
Log.i("Missing =","${soldier1.missing}")
```

前面的代码将在 logcat 窗口中产生以下输出：

```kt
Name =: Ryan
Rank =: Private
Missing =: true
Name =: Ryan
Rank =: Private First Class
Missing =: false

```

在前面的输出中，首先我们看到与之前相同的三行，然后我们看到另外三行，表明 Ryan 不再失踪，并且已经晋升为`列兵`。

### 使用类的函数和变量

现在我们可以给我们的类提供数据，是时候通过给它们一些可以做的事情来使它们更有用了。为了实现这一点，我们可以给我们的类提供函数。看一下`Soldier`类的这段扩展代码。我已经将变量恢复为`val`并突出显示了新代码：

```kt
class Soldier{

    // members
    val name = "Ryan"
    val rank = "Private"
    val missing = true

    // Class function
 fun getStatus() {
 var status = "$rank $name"
 if(missing){
 status = "$status is missing!"
 }else{
 status = "$status ready for duty."
 }

 // Print out the status
 Log.i("Status",status)
 }
}
```

`getStatus`函数中的代码声明了一个名为`status`的新`String`变量，并使用`rank`和`name`中包含的值对其进行初始化。然后，它使用`if`表达式检查`missing`中的值，并根据`missing`是`true`还是`false`附加`is missing`或`ready for duty`。

然后我们可以像下面的代码演示的那样使用这个新函数：

```kt
val soldier1 = Soldier()
soldier1.getStatus()
```

与之前一样，我们创建了`Soldier`类的一个实例，然后在该实例上使用点语法调用`getStatus`函数。前面的代码将在 logcat 窗口中产生以下输出：

```kt
Status: Private Ryan is missing!

```

如果我们将`missing`的值更改为`false`，将产生以下输出：

```kt
Status: Private Ryan ready for duty.

```

请注意，类中的函数可以采用我们在第九章中讨论过的任何形式，*Kotlin 函数*。

如果你认为所有这些类的东西都很棒，但同时似乎有点僵化和不灵活，那么你是正确的。如果所有`Soldier`实例都叫 Ryan 并且都失踪，那有什么意义呢？当然，我们已经看到我们可以使用`var`变量然后更改它们，但这可能仍然很尴尬和冗长。

我们需要更好地操纵和初始化每个实例中的数据的方法。如果我们回想一下本章开头时我们简要讨论了封装的主题，那么我们也会意识到我们不仅需要允许代码操纵我们的数据，还需要控制这种操纵何时以及如何进行。

为了获得这些知识，我们需要更多地了解类中的变量，然后更详细地了解封装和可见性，最后揭示当我们实例化类的实例时，在代码末尾看到的那些类似函数的括号`()`到底是什么意思。

### 类变量是属性

原来在 Kotlin 中，类变量不仅仅是我们已经了解的普通变量。它们是**属性**。到目前为止，我们已经学到的关于如何使用变量的一切仍然成立，但是属性比值更多。它有**getter**，**setter**，以及一个特殊的类变量称为**field**隐藏在幕后。

Getter 和 setter 可以被视为编译器自动生成的特殊函数。事实上，我们已经在不知情的情况下使用了它们。

当我们在类中声明的属性/变量上使用点语法时，Kotlin 使用 getter 来“获取”值。当我们使用点语法设置值时，Kotlin 使用 setter。

当我们使用刚刚看到的点语法时，并不直接访问字段/变量本身。这种抽象的原因是为了帮助封装。

如果你之前在其他面向对象的语言（也许是 Java 或 C++）中做过一些编程，这可能会让你感到困惑，但如果你使用过更现代的面向对象语言（也许是 C#），那么这对你来说不会是全新的。如果 Kotlin 是你的第一门语言，那么你可能比有过往经验的人更有优势，因为你不会受到以前学习的包袱。

而且，你可能会猜到，如果变量是`var`，那么会提供一个 getter 和一个 setter，但如果是`val`，那么只会提供一个 getter。因此，当`Soldier`类中的变量（我们从现在开始大多数时候称之为属性）是`var`时，我们可以获取和设置它们，但当它们是`val`时，我们只能获取它们。

Kotlin 给了我们灵活性来**重写**这些 getter 和 setter，以改变当我们获取和设置属性及其关联字段的值时发生的情况。

### 提示

当属性使用字段时，它被称为**后备字段**。正如我们将看到的，一些属性不需要后备字段，因为它们可以依赖于 getter 和 setter 中的逻辑来使它们有用。

在这一点上，使用字段的一些示例将使事情更清晰。

### 使用带有 getter、setter 和字段的属性的示例

我们可以使用 getter 和 setter 来控制可以分配给其后备字段的值范围。例如，考虑将下一行代码添加到`Soldier`类中：

```kt
var bullets = 100
get() {
   Log.i("Getter being used","Value = $field")
   return field
}
set(value) {
   field = if (value < 0) 0 else value
   Log.i("Setter being used","New value = $field")
}
```

前面的代码添加了一个名为`bullets`的新`var`属性，并将其初始化为 100。然后我们看到一些新代码。getter 和 setter 被重写了。去掉 getter 和 setter 中的代码，以便以最简单的形式看到其运行：

```kt
get() {
   //.. Executes when we try to retrieve the value
}
set(value) {
   //.. Executes when we try to set the value 
}
```

明确一点，在访问`Soldier`类的实例中的`bullet`值时，getter 和 setter 中的代码会执行。看看下面的代码中可能会发生的情况：

```kt
// In onCreate or some other function/class from our app
// Create a new instance of the Soldier class
val soldier = Soldier()
// Access the value of bullets
Log.i("bullets = ","${soldier.bullets}")// Getter will execute
// Reduce the number of bullets by one
soldier.bullets --
Log.i("bullets =","${soldier.bullets}")// Setter will execute
```

在前面的代码中，我们首先创建了`Soldier`类的一个实例，然后获取存储在`bullet`属性中的值并打印出来。这触发了 getter 代码的执行。

接下来，我们减少（减少一个）`bullet`属性存储的值。任何试图改变属性持有的值的操作都会触发 setter 中的代码。

如果我们执行前面的四行代码，将在 logcat 窗口中得到以下输出：

```kt
Getter being used: Value = 100
bullets =: 100
Getter being used: Value = 100
Setter being used: New value = 99
Getter being used: Value = 99
bullets =: 99

```

创建一个名为`soldier`的`Soldier`实例后，我们使用`Log.i`将值打印到 logcat 窗口。由于此代码访问了属性存储的值，getter 代码运行并打印出以下内容：

```kt
Getter being used: Value = 100

```

然后 getter 使用下一行代码将值返回给`Log.i`函数：

`return field`

当我们创建属性时，Kotlin 创建了一个后备字段。在 getter 或 setter 中访问后备字段的方式是使用名称`field`。因此，前面的代码行的工作方式与在函数中的方式相同，并返回值，允许调用代码中的`Log.i`调用打印出值，我们将得到下一行输出：

```kt
bullets =: 100

```

下一行代码可能是最有趣的。这里再次提供以便参考：

```kt
soldier.bullets --
```

我们可能会猜想这只是触发了 setter 的执行，但是如果我们检查 logcat 中的下两行输出，我们会看到生成了以下两行输出：

```kt
Getter being used: Value = 100
Setter being used: New value = 99

```

减少（或增加）的操作需要使用 getter（知道要减少多少）然后使用 setter 来改变值。

请注意，setter 有一个名为`value`的参数，我们可以在 setter 的主体中引用它，就像普通的函数参数一样。

接下来，实例被用来输出`bullets`属性所持有的值，我们可以看到再次使用了 getter，并且输出是由类中的 getter 代码和实例（类外部）中的代码生成的。接下来再次显示最后两行输出：

```kt
Getter being used: Value = 99
bullets =: 99

```

现在我们可以看另一个使用 getter 和 setter 的例子。

正如前面提到的，有时属性根本不需要后备字段。有时，允许 getter 和 setter 中的逻辑处理通过属性访问的值就足够了。查看下面的代码，我们可以将其添加到`Soldier`类中来演示这一点：

```kt
var packWeight = 150
val gunWeight = 30
var totalWeight = packWeight + gunWeight
   get() = packWeight + gunWeight
```

在上面的代码中，我们创建了三个属性：一个名为`packWeight`的`var`属性，我们将使用即将创建的实例来更改它，一个名为`gunWeight`的`val`属性，我们永远不需要更改它，以及另一个名为`totalWeight`的`var`属性，它被初始化为`packWeight + gunWeight`。有趣的部分是，我们覆盖了`totalWeight`的 getter，以便它使用`packWeight + gunWeight`重新计算其值。接下来，让我们看看如何使用`Soldier`类的实例来使用这些新属性，然后我们将看到输出：

```kt
// Create a soldier
val strongSoldier = Soldier()

// Print out the totalWeight value
Log.i("totalWeight =","${strongSoldier.totalWeight}")

// Change the value of packWeight
strongSoldier.packWeight = 300

// Print out the totalWeight value again
Log.i("totalWeight =","${strongSoldier.totalWeight}")
```

在上面的代码中，我们创建了一个名为`strongSoldier`的`Soldier`实例。接下来，我们将`totalWeight`的值打印到 logcat。第三行代码将`packWeight`的值更改为`300`，然后最后一行代码打印出`totalWeight`的值，它将使用我们覆盖的 getter。以下是这四行代码的输出：

```kt
totalWeight =: 180
totalWeight =: 330

```

从输出中我们可以看到，`totalWeight`的值完全取决于`packWeight`和`gunWeight`中存储的值。输出的第一行是`packWeight`的起始值（`150`）加上`gunWeight`的值（`30`），第二行输出等于`packWeight`的新值加上`gunWeight`。

就像函数一样，这个非常灵活的属性系统会引发一些问题。

### 何时使用覆盖的 getter 和 setter

何时利用这些不同的技术需要通过实践和经验来决定；关于何时适合使用特定技术并没有硬性规定。在这个阶段，只需要理解在类的主体（函数之外）声明的变量实际上是属性，而属性是通过 getter 和 setter 访问的。这些 getter 和 setter 对于实例的用户来说并不是透明的，并且除非被类的程序员覆盖，否则编译器会默认提供它们。这就是封装的本质；类的程序员控制类的工作方式。属性提供对其相关值（称为后备字段）的间接访问，尽管有时这个后备字段是不需要的。

### 提示

简化讨论时将属性称为变量是可以的（我有时这样做）。特别是当 getter、setter 和字段与讨论无关时。

在下一节中，我们将看到更多可以使用 getter 和 setter 的方法，所以让我们继续讨论可见性修饰符。

# 可见性修饰符

可见性修饰符用于控制变量、函数甚至整个类的访问/可见性。正如我们将看到的，根据代码中尝试访问的位置，可以有不同级别的访问权限的变量、函数和类。这允许类的设计者实践良好的封装，并且只向类的用户提供他们选择的功能和数据。举一个有点牵强但有用的例子，用于与卫星通信并获取 GPS 数据的类的设计者不会允许访问`dropOutOfTheSky`函数。

这是 Kotlin 中的四个访问修饰符。

## 公共

将类、函数和属性声明为`public`意味着它们根本不被隐藏/封装。实际上，默认可见性是`public`，因此到目前为止我们所见过和使用的一切都是公共的。我们可以通过在所有类、函数和属性声明之前使用`public`关键字来明确表示这一点，但这并不是必要的。当某物被声明为`public`（或保持默认状态）时，不使用封装。这只是偶尔我们想要的。通常，公开类的函数将公开类的核心功能。

## 私有

我们将讨论的下一个访问修饰符是`private`。通过在声明之前加上`private`关键字，属性、函数和类可以被声明为`private`，如下一个假设的代码所示：

```kt
private class SatelliteController {
   private var gpsCoordinates = "51.331958,0.029057"

   private fun dropOutOfTheSky() {
   }
}
```

`SatelliteController`类被声明为`private`，这意味着它只能在同一文件中使用（可以实例化）。尝试在`onCreate`中实例化一个实例可能会导致以下错误：

![Private](img/B12806_10_01.jpg)

这引发了一个问题，即类是否可以被使用。将类声明为`private`比使用我们将要讨论的剩余修饰符要少得多，但这确实会发生，并且有各种技术使其成为一种可行的策略。然而，更有可能的是，`SatelliteController`类将以更加可访问的`public`可见性进行声明。

继续，我们有一个名为`gpsCoordinates`的`private`属性。假设我们将`SatelliteController`类更改为公共类，那么我们就可以实例化它并继续我们的讨论。即使`SatelliteController`被声明为`public`，或者保持默认状态为`public`，私有的`gpsCoordinates`属性仍然对类的实例不可见，如下一个截图所示：

![Private](img/B12806_10_02.jpg)

正如我们在前面的截图中所看到的，`gpsCoordinates`属性是不可访问的，因为它是`private`的，正如我们在本章前面讨论属性时所看到的，当属性保持默认状态时，它是可访问的。这些访问修饰符的目的是，类的设计者可以选择何时以及何物来公开。很可能 GPS 卫星希望分享 GPS 坐标。然而，很可能它不希望类的用户在计算坐标方面起任何作用。这表明我们希望类的用户能够读取数据，但不能写入/更改数据。这是一个有趣的情况，因为第一反应可能是将属性设置为`val`属性。这样用户就可以获取数据，但不能更改数据。但问题是 GPS 坐标显然是会变化的，因此它需要是一个`var`属性，只是不希望它是一个可以从类外部更改的`var`属性。

当我们将属性声明为`private`时，Kotlin 会自动将 getter 和 setter 也设为`private`。我们可以通过重写 getter 和/或 setter 来改变这种行为。为了解决我们需要一个在类外部不可改变但在类内部可改变和可读的`var`属性的问题，我们将保留默认的 setter，使其无法在外部改变，并重写 getter，以便在外部可读。看看下面对`SatelliteController`类的重写：

```kt
class SatelliteController {
    var gpsCoordinates = "51.331958,0.029057"
    private set

    private fun dropOutOfTheSky() {
    }
}
```

在上面的代码中，`SatelliteController`类和`gpsCoordinates`属性都是`public`的。此外，`gpsCoordinates`是一个`var`属性，因此是可变的。然而，仔细看一下属性声明后的代码行，因为它将 setter 设置为`private`，这意味着类外的代码无法访问它进行更改；但因为它是一个`var`属性，类内的代码可以对其进行任何操作。

现在我们可以在`onCreate`函数中编写以下代码来使用该类：

```kt
// This still doesn't work which is what we want
// satelliteController.gpsCoordinates = "1.2345, 5.6789"

// But this will print the gpsCoordinates
Log.i("Coords=","$satelliteController.gpsCoordinates")
```

现在，由于代码将 setter 设置为私有，我们无法从实例更改值，但可以愉快地读取它，就像前面的代码演示的那样。请注意，setter 不能更改其可见性，但可以（正如我们在首次讨论属性时看到的）重写其功能。

继续讨论`dropOutOfSky`函数的功能，这是`private`且完全不可访问的。只有`SateliteController`类内部的代码才能调用该函数。如果我们希望类的用户能够访问函数，就像我们已经看到的那样，我们只需将其保留为默认可见性。`SatelliteController`类可能有类似下面代码的函数：

```kt
class SatelliteController {
    var gpsCoordinates = "51.331958,0.029057"
    private set

    private fun dropOutOfTheSky() {
    }

    fun updateCoordinates(){
        // Recalculate coordinates and update
        // the gpsCoordinates property
        gpsCoordinates = "21.123456, 2.654321"

        // user can now access the new coordinates
        // but still can't change them
    }
}
```

在前面的代码中，添加了一个公共的`updateCoordinates`函数。这允许类的实例使用以下代码：

```kt
satelliteController.updateCoordinates()
```

然后，前面的代码将触发`updateCoordinates`函数的执行，这将导致类内部更新属性，然后可以访问并提供新值。

这引出了一个问题：哪些数据应该是私有的？应该使用的可见性级别部分可以通过常识学习，部分通过经验学习，部分通过问自己这个问题：“谁真正需要访问这些数据以及在什么程度上？”我们将在本书的其余部分中练习这三件事。以下是一些更多的假设代码，显示了`SatelliteController`类的一些私有数据和更多私有函数：

```kt
class SatelliteController {
    var gpsCoordinates = "51.331958,0.029057"
    private set

    private var bigProblem = false

    private fun dropOutOfTheSky() {
    }

    private fun doDiagnostics() {
      // Maybe set bigProblem to true
      // etc
    }

    private fun recalibrateSensors(){
      // Maybe set bigProblem to true
      // etc
    }

    fun updateCoordinates(){
        // Recalculate coordinates and update
        // the gpsCoordinates property
        gpsCoordinates = "21.123456, 2.654321"

        // user can now access the new coordinates
        // but still can't change them
    }

    fun runMaintenance(){
        doDiagnostics()
        recalibrateSensors()

        if(bigProblem){
            dropOutOfTheSky()
        }

    }
}
```

在上述代码中，有一个名为`bigProblem`的新私有`Boolean`属性。它只能在内部访问。甚至不能在外部读取。有三个新函数，一个名为`runMaintenance`的公共属性，它运行两个私有函数`doDiagnostics`和`calibrateSensors`。这两个函数可以访问并更改`bigProblem`的值（如果需要）。在`runMaintenance`函数中，进行了一个检查，看看`bigProblem`是否为 true，如果是，则调用`dropOutOfTheSky`函数。

### 提示

显然，在真实卫星的代码中，除了掉出天空之外，可能首先会寻求其他解决方案。

让我们看看最后两个可见性修饰符。

## 受保护的

当使用`protected`可见性修饰符时，其影响比`public`和`private`更微妙。当函数或属性声明为`protected`时，它几乎是私有的 - 但并非完全如此。我们将在下一章中探讨的另一个关键面向对象编程主题是继承，它允许我们编写类，然后编写另一个继承该类功能的类。`protected`修饰符将允许函数和属性对这些类可见，但对所有其他代码隐藏。

我们将在整本书中进一步探讨这个问题。

## 内部

内部修饰符比其他修饰符更接近公共。它会将属性/函数暴露给同一包中的任何代码。如果考虑到一些应用程序只有一个包，那么这是相当宽松的可见性。我们不会经常使用它，我只是想让你了解一下，以便完整起见。

## 可见性修饰符总结

尽管我们已经讨论了好几页，但我们只是触及了可见性修饰符的表面。关键是它们存在，其目的是帮助封装并使您的代码不太容易出错，并且更具可重用性。结合属性、函数、getter 和 setter，Kotlin 非常灵活，我们可以用更多的例子来说明何时以及在何处使用每个可见性修饰符，以及何时、在何处以及如何以不同方式重写 getter 和 setter。使用这些技术构建工作程序更有用。这是我们将在整本书中做的事情，我经常会提到为什么我们使用特定的可见性修饰符或者为什么我们以特定的方式使用 getter/setter。我还鼓励您在本章末尾进行基本类演示应用。

# 构造函数

在本章中，我们一直在实例化对象（类的实例），并且我们已经深入讨论了各种语法。直到现在，有一小部分代码我们一直忽略。下面的代码我们以前看过几次，但我已经突出显示了一小部分，以便我们进一步讨论：

```kt
val soldier = Soldier()

```

代码末尾的括号初始化对象的代码看起来就像前一章中调用函数时的代码（没有任何参数）。事实上，情况确实如此。当我们声明一个类时，Kotlin 提供（在幕后）一个名为**构造函数**的特殊函数，用于准备实例。

到目前为止，在本章中，我们已经在一行中声明和初始化了所有的实例。通常，我们需要在初始化中使用一些更多的逻辑，而且我们经常需要允许初始化类的代码传递一些值（就像一个函数）。这就是构造函数的原因。

通常，这个默认构造函数就是我们需要的全部内容，我们可以忘记它，但有时我们需要做更多的工作来设置我们的实例，以便它准备好使用。Kotlin 允许我们声明自己的构造函数，并给我们三个主要选项：主要构造函数、次要构造函数和`init`块。

## 主要构造函数

主要构造函数是在类声明中声明的构造函数。看看下面的代码，它定义了一个允许类的用户传入两个值的构造函数。正如我们所期望的那样，这段代码将放在一个名为`Book.kt`的文件中。

```kt
class Book(val title: String, var copiesSold: Int) {
   // Here we put our code as normal
   // But title and copiesSold are properties that
   // are already declared and initialized
}
```

在上面的代码中，我们声明了一个名为`Book`的类，并提供了一个接受两个参数的构造函数。当初始化时，它需要传递一个不可变的`String`值和一个可变的`Int`值。提供这样的构造函数，然后使用它来实例化一个实例，声明和初始化了`title`和`copiesSold`属性。没有必要以通常的方式声明或初始化它们。

看看下面的代码，它展示了如何实例化这个类的一个实例：

```kt
// Instantiate a Book using the primary constructor
val book = Book("Animal Farm", 20000000)
```

在上面的代码中，使用主要构造函数实例化了一个名为`book`的对象，属性`title`和`copiesSold`分别初始化为`Animal Farm`和`20000000`（两千万）。

就像函数一样，你可以塑造构造函数，拥有任意组合、类型和数量的参数。

主要构造函数的潜在缺点是属性从传入的参数中获取值，没有任何灵活性。如果我们需要在将它们分配给属性之前对传入的值进行一些计算怎么办？幸运的是，我们可以处理这个问题。

## 次要构造函数

次要构造函数是在类声明之外单独声明的构造函数，但仍然在类体内。关于次要构造函数需要注意的几件事是，你不能在参数中声明属性，而且你还必须从次要构造函数的代码中调用主要构造函数。次要构造函数的优势在于你可以编写一些逻辑（代码）来初始化你的属性。看看下面的代码，它展示了这一点。同时，我们还将介绍一个新的关键字：

```kt
// Perhaps the user of the class 
// doesn't know the time as it
// is yet to be confirmed
class Meeting(val day: String, val person: String) {
    var time: String = "To be decided"
    // The user of the class can
    // supply the day, time and person 
    // of a meeting
    constructor(day: String, person: String, time: String)
            :this(day, person ){

        // "this" refers to the current instance
        this.time = time
        // time (the property) now equals time
        // that was passed in as a parameter
    }
}
```

在上面的代码中，我们声明了一个名为`Meeting`的类。主要构造函数声明了两个属性，一个叫做`day`，一个叫做`person`。接下来，声明了一个名为`time`的属性，并初始化为值`To be decided`。

接下来是次要构造函数。注意参数前面有`constructor`关键字。你还会注意到，次要构造函数包含三个参数，与主要构造函数相同的两个参数，还有一个叫做`time`的参数。

请注意，`time`参数与先前声明和初始化的`time`属性不是同一个实体。次要构造函数只包含“一次性”参数，它们不会成为像主构造函数那样的持久属性。这使我们首先可以调用主构造函数传递`day`和`person`，其次（在次要构造函数的主体中）将通过`time`参数传递的值分配给`time`属性。

### 提示

您可以提供多个次要构造函数，只要签名都不同。通过匹配调用/实例化代码的参数，将调用适当的次要构造函数。

### 我们需要谈谈这个

我是说，我们需要谈谈`this`关键字。当我们在类内部使用`this`时，它会引用当前实例 - 因此它会作用于自身。

因此，`this(day, person)`代码调用初始化`day`和`person`属性的主构造函数。此外，`this.time = time`代码会将通过`time`参数传递的值分配给实际的`time`属性（`this.time`）。

### 注意

顺便提一句，如果不明显的话，`Meeting`类需要额外的函数才能使其有意义，比如`setTime`、`getMeetingDetails`，可能还有其他函数。

当用户不知道时间时（通过主构造函数）或者当他们知道时间时（通过次要构造函数）可以创建`Meeting`类的实例。

### 使用 Meeting 类

我们将通过调用我们的构造函数之一来实例化我们的实例，如下面的代码所示：

```kt
// Book two meetings
// First when we don't yet know the time
val meeting = Meeting("Thursday", "Bob")

// And secondly when we do know the time
val anotherMeeting = Meeting("Wednesday","Dave","3 PM")
```

在上面的代码中，我们初始化了`Meeting`类的两个实例，一个叫做`meeting`，另一个叫做`anotherMeeting`。在第一次实例化时，我们调用了主构造函数，因为我们不知道时间；而在第二次实例化时，我们调用了次要构造函数，因为我们知道时间。

如果需要，我们可以有多个次要构造函数，只要它们都调用主构造函数。

## 初始化块

Kotlin 被设计为一种简洁的语言，通常有更简洁的方法来初始化我们的属性。如果类不依赖于多个不同的签名，那么我们可以坚持使用更简洁的主构造函数，并在`init`块中提供任何必需的初始化逻辑：

```kt
init{
  // This code runs when the class is instantiated
  // and can be used to initialize properties
}
```

这可能是足够的理论了；让我们在一个工作应用程序中使用我们一直在谈论的一切。接下来，我们将编写一个使用类的小应用程序，包括主构造函数和`init`块。

# 基本类应用程序和使用 init 块

您可以在代码下载中获取此应用程序的完整代码。它位于`Chapter10/Basic Classes`文件夹中。但是，继续阅读以创建您自己的工作示例会更有用。

我们将使用本章学到的知识创建几个不同的类，以将理论付诸实践。我们还将看到我们的第一个示例，即类如何通过将类作为参数传递到另一个类的函数中相互交互。我们已经知道如何在理论上做到这一点，只是还没有在实践中看到它。

当类首次实例化时，我们还将看到另一种初始化数据的方法，即使用`init`块。

我们将创建一个小应用程序，用于模拟船只、码头和海战的想法。

### 注意

本章和下一章应用程序的输出将只是文本，显示在 logcat 窗口中。在第十二章中，*将我们的 Kotlin 连接到 UI 和可空性*，我们将把我们在前五章学到的关于 Android UI 的知识和在接下来的六章中学到的关于 Kotlin 的知识结合起来，让我们的应用程序活起来。

使用空活动模板创建一个名为`Basic Classes`的应用程序。现在我们将创建一个名为`Destroyer`的新类：

1.  在项目资源管理器窗口中右键单击`com.gamecodeschool.basicclasses`（或者您的包名）文件夹。

1.  选择**新建** **|** **Kotlin 文件/类**。

1.  在**名称：**字段中，键入`Destroyer`。

1.  在下拉框中选择**类**。

1.  单击**OK**按钮将新类添加到项目中。

1.  重复前面的五个步骤，创建另外两个类，一个叫做`Carrier`，另一个叫做`ShipYard`。

新的类已经为我们创建了一个类声明和大括号，准备好我们的代码。自动生成的代码还包括包声明，这将根据您在创建项目时的选择而有所不同。这是我目前代码的样子。

在`Destroyer.kt`中：

```kt
package com.gamecodeschool.basicclasses

class Destroyer {
}
```

在`Carrier.kt`中：

```kt
package com.gamecodeschool.basicclasses

class Carrier {
}
```

在`ShipYard.kt`中：

```kt
package com.gamecodeschool.basicclasses

class ShipYard {
}
```

让我们从编写`Destroyer`类的第一部分开始。接下来是构造函数、一些属性和一个`init`块。添加代码到项目中，学习它，然后我们将回顾我们所做的事情：

```kt
class Destroyer(name: String) {
    // What is the name of this ship
    var name: String = ""
        private set

    // What type of ship is it
    // Always a destroyer
    val type = "Destroyer"

    // How much the ship can take before sinking
    private var hullIntegrity = 200

    // How many shots left in the arsenal
    var ammo = 1
    // Cannot be directly set externally
        private set

    // No external access whatsoever
    private var shotPower = 60

    // Has the ship been sunk
    private var sunk = false

    // This code runs as the instance is being initialized
    init {
        // So we can use the name parameter
        this.name = "$type $name"
    }
```

首先要注意的是构造函数接收一个名为`name`的`String`值。它没有声明为`val`或`var`属性。因此，它不是一个属性，只是一个在实例初始化后将不复存在的常规参数。我们很快将看到如何利用它。

在前面的代码中，我们声明了一些属性。请注意，大多数都是可变的`var`，除了`type`，它是一个初始化为`Destroyer`的`String` `val`类型。还要注意，大多数都是`private`访问，除了两个。

`type`属性是公共的，因此可以通过类的实例完全访问。`name`属性也是公共的，但具有`private`的 setter。这将允许实例获取值，但保护后备字段（值）不被实例更改。

`hullIntegrity`、`ammo`、`shotPower`和`sunk`属性都是`private`的，无法通过实例直接访问。请务必记住这些属性的值和类型。

前面代码的最后一部分是一个`init`块，在这个块中，`name`属性通过将类型和名称属性连接起来并在中间加上一个空格来进行初始化。

接下来，添加接下来的`takeDamage`函数：

```kt
fun takeDamage(damageTaken: Int) {
   if (!sunk) {
        hullIntegrity -= damageTaken
        Log.i("$name damage taken =","$damageTaken")
        Log.i("$name hull integrity =","$hullIntegrity")

        if (hullIntegrity <= 0) {
               Log.d("Destroyer", "$name has been sunk")
               sunk = true
        }
  } else {
         // Already sunk
         Log.d("Error", "Ship does not exist")
  }
}
```

在`takeDamage`函数中，`if`表达式检查`sunk`布尔值是否为 false。如果船只还没有沉没，那么`hullIntegrity`将减去传入的`damageTaken`值。因此，尽管`private`，实例仍然会间接影响`hullIntegrity`。关键是它只能以程序员决定的方式来做到这一点；在这种情况下，是我们。正如我们将看到的，所有私有属性最终都将以某种方式被操作。

此外，如果船还没有沉没，两个`Log.i`调用将损坏信息和剩余船体完整性信息输出到 logcat 窗口。最后，在未沉没的情况下`(!sunk)`，嵌套的`if`表达式检查`hullIntegrity`是否小于零。如果是，则打印一条消息表示船已经沉没，并将`sunk`布尔值设置为 true。

当调用`damageTaken`函数并且`sunk`变量为 true 时，`else`块将执行，并打印一条消息，表示船只不存在，因为它已经沉没了。

接下来，添加`shootShell`函数，它将与`takeDamage`函数一起工作。更确切地说，一个船只实例的`takeDamage`函数将与其他船只实例的`shootShell`函数一起工作，我们很快就会看到：

```kt
fun shootShell():Int {
  // Let the calling code no how much damage to do
  return if (ammo > 0) {
         ammo--
         shotPower
  }else{
        0
  }
}
```

在`shootShell`函数中，如果船只有弹药，`ammo`属性将减少一个，并将`shotPower`的值返回给调用代码。如果船只没有弹药（`ammo`不大于零），则将值`0`返回给调用代码。

最后，对于`Destroyer`类添加`serviceShip`函数，将`ammo`设置为`10`，`hullIntegrity`设置为`100`，以便船只完全准备好再次承受伤害（通过`takeDamage`）并造成伤害（通过`shootShell`）：

```kt
fun serviceShip() {
    ammo = 10
    hullIntegrity = 100
}
```

接下来，我们可以快速编写`Carrier`类，因为它非常相似。只需注意一下分配给`type`和`hullIntegrity`的值的细微差异。还要注意，我们使用`attacksRemaining`和`attackPower`，而不是`ammo`和`shotPower`。此外，`shootShell`已被替换为`launchAerialAttack`，这似乎更适合一艘航空母舰。将以下代码添加到`Carrier`类中：

```kt
class Carrier (name: String){
    // What is the name of this ship
    var name: String = ""
        private set

    // What type of ship is it
    // Always a destroyer
    val type = "Carrier"

    // How much the ship can take before sinking
    private var hullIntegrity = 100

    // How many shots left in the arsenal
    var attacksRemaining = 1
    // Cannot be directly set externally
        private set

    private var attackPower = 120

    // Has the ship been sunk
    private var sunk = false

    // This code runs as the instance is being initialized
    init {
        // So we can use the name parameter
        this.name = "$type $name"
    }

    fun takeDamage(damageTaken: Int) {
        if (!sunk) {
            hullIntegrity -= damageTaken
            Log.d("$name damage taken =","$damageTaken")
            Log.d("$name hull integrity =","$hullIntegrity")

            if (hullIntegrity <= 0) {
                Log.d("Carrier", "$name has been sunk")
                sunk = true
            }
        } else {
            // Already sunk
            Log.d("Error", "Ship does not exist")
        }
    }

    fun launchAerialAttack() :Int {
        // Let the calling code no how much damage to do
        return if (attacksRemaining > 0) {
            attacksRemaining--
            attackPower
        }else{
            0
        }
    }

    fun serviceShip() {
        attacksRemaining = 20
        hullIntegrity = 200
    }
}
```

在我们开始使用新的类之前的最后一段代码是`ShipYard`类。它有两个简单的函数：

```kt
class ShipYard {

    fun serviceDestroyer(destroyer: Destroyer){
        destroyer.serviceShip()
    }

    fun serviceCarrier(carrier: Carrier){
        carrier.serviceShip()
    }
}
```

第一个函数`serviceDestroyer`以`Destroyer`实例作为参数，并在该函数内部简单地调用实例的`serviceShip`函数。第二个函数`serviceCarrier`具有相同的效果，但以`Carrier`实例作为参数。虽然这两个函数很简短，但它们的后续使用很快就会揭示一些与类及其实例相关的有趣细微差别。

现在我们将创建一些实例，并通过模拟一场虚构的海战来让我们的类发挥作用。将以下代码添加到`MainActivity`类的`onCreate`函数中：

```kt
val friendlyDestroyer = Destroyer("Invincible")
val friendlyCarrier = Carrier("Indomitable")

val enemyDestroyer = Destroyer("Grey Death")
val enemyCarrier = Carrier("Big Grey Death")

val friendlyShipyard = ShipYard()

// Uh oh!
friendlyDestroyer.takeDamage(enemyDestroyer.shootShell())
friendlyDestroyer.takeDamage(enemyCarrier.launchAerialAttack())

// Fight back
enemyCarrier.takeDamage(friendlyCarrier.launchAerialAttack())
enemyCarrier.takeDamage(friendlyDestroyer.shootShell())

// Take stock of the supplies situation
Log.d("${friendlyDestroyer.name} ammo = ",
         "${friendlyDestroyer.ammo}")

Log.d("${friendlyCarrier.name} attacks = ",
         "${friendlyCarrier.attacksRemaining}")

// Dock at the shipyard
friendlyShipyard.serviceCarrier(friendlyCarrier)
friendlyShipyard.serviceDestroyer(friendlyDestroyer)

// Take stock of the supplies situation again
Log.d("${friendlyDestroyer.name} ammo = ",
         "${friendlyDestroyer.ammo}")

Log.d("${friendlyCarrier.name} attacks = ",
         "${friendlyCarrier.attacksRemaining}")

// Finish off the enemy
enemyDestroyer.takeDamage(friendlyDestroyer.shootShell())
enemyDestroyer.takeDamage(friendlyCarrier.launchAerialAttack())
enemyDestroyer.takeDamage(friendlyDestroyer.shootShell())
```

让我们回顾一下那段代码。代码首先实例化了两艘友方船只（`friendlyDestroyer`和`friendlyCarrier`）和两艘敌方船只（`enemyDestroyer`和`enemyCarrier`）。此外，还实例化了一个名为`friendlyShipyard`的`Shipyard`实例，为随之而来的不可避免的大屠杀做好准备：

```kt
val friendlyDestroyer = Destroyer("Invincible")
val friendlyCarrier = Carrier("Indomitable")

val enemyDestroyer = Destroyer("Grey Death")
val enemyCarrier = Carrier("Big Grey Death")

val friendlyShipyard = ShipYard()
```

接下来，`friendlyDestroyer`对象受到两次伤害。一次来自`enemyDestroyer`，一次来自`enemyCarrier`。这是通过`friendlyDestroyer`的`takeDamage`函数传入两个敌人的`shootShell`和`launchAerialAttack`函数的返回值来实现的：

```kt
// Uh oh!
friendlyDestroyer.takeDamage(enemyDestroyer.shootShell())
friendlyDestroyer.takeDamage(enemyCarrier.launchAerialAttack())
```

接下来，友方部队通过对`enemyCarrier`对象进行两次攻击进行反击，一次来自`friendlyCarrier`对象通过`launchAerialAttack`，一次来自`friendlyDestroyer`对象通过`shootShell`：

```kt
// Fight back
enemyCarrier.takeDamage(friendlyCarrier.launchAerialAttack())
enemyCarrier.takeDamage(friendlyDestroyer.shootShell())
```

然后将两艘友方船只的状态输出到 logcat 窗口：

```kt
// Take stock of the supplies situation
Log.d("${friendlyDestroyer.name} ammo = ",
         "${friendlyDestroyer.ammo}")

Log.d("${friendlyCarrier.name} attacks = ",
         "${friendlyCarrier.attacksRemaining}")
```

现在，适当的`Shipyard`实例的函数依次在适当的实例上调用。没有`enemyShipyard`对象，因此它们将无法进行修复和重新武装：

```kt
// Dock at the shipyard
friendlyShipyard.serviceCarrier(friendlyCarrier)
friendlyShipyard.serviceDestroyer(friendlyDestroyer)
```

接下来，再次打印统计数据，以便我们可以看到访问船坞后的差异：

```kt
// Take stock of the supplies situation again
Log.d("${friendlyDestroyer.name} ammo = ",
         "${friendlyDestroyer.ammo}")

Log.d("${friendlyCarrier.name} attacks = ",
         "${friendlyCarrier.attacksRemaining}")
```

然后，或许是不可避免的，友方部队击败了敌人：

```kt
// Finish off the enemy
enemyDestroyer.takeDamage(friendlyDestroyer.shootShell())
enemyDestroyer.takeDamage(friendlyCarrier.launchAerialAttack())
enemyDestroyer.takeDamage(friendlyDestroyer.shootShell())
```

运行应用程序，然后我们可以在 logcat 窗口中检查以下输出：

```kt
Destroyer Invincible damage taken =: 60
Destroyer Invincible hull integrity =: 140
Destroyer Invincible damage taken =: 120
Destroyer Invincible hull integrity =: 20
Carrier Big Grey Death damage taken =: 120
Carrier Big Grey Death hull integrity =: -20
Carrier: Carrier Big Grey Death has been sunk
Error: Ship does not exist
Destroyer Invincible ammo =: 0
Carrier Indomitable attacks =: 0
Destroyer Invincible ammo =: 10
Carrier Indomitable attacks =: 20
Destroyer Grey Death damage taken =: 60
Destroyer Grey Death hull integrity =: 140
Destroyer Grey Death damage taken =: 120
Destroyer Grey Death hull integrity =: 20
Destroyer Grey Death damage taken =: 60
Destroyer Grey Death hull integrity =: -40
Destroyer: Destroyer Grey Death has been sunk
```

这里是输出，这次分成几部分，以便我们清楚地看到哪些代码产生了哪些输出行。

友好的驱逐舰遭到袭击，使其船体接近破裂点：

```kt
Destroyer Invincible damage taken =: 60
Destroyer Invincible hull integrity =: 140
Destroyer Invincible damage taken =: 120
Destroyer Invincible hull integrity =: 20
```

敌方航空母舰遭到攻击并被击沉：

```kt
Carrier Big Grey Death damage taken =: 120
Carrier Big Grey Death hull integrity =: -20
Carrier: Carrier Big Grey Death has been sunk
```

敌方航空母舰再次遭到攻击，但因为它被击沉，`takeDamage`函数中的`else`块被执行：

```kt
Error: Ship does not exist
```

当前的弹药/可用攻击统计数据被打印出来，友方部队的情况看起来很糟糕：

```kt
Destroyer Invincible ammo =: 0
Carrier Indomitable attacks =: 0
```

快速访问船坞，情况会好得多：

```kt
Destroyer Invincible ammo =: 10
Carrier Indomitable attacks =: 20
```

友方部队全副武装并修复，完成了剩余驱逐舰的摧毁：

```kt
Destroyer Grey Death damage taken =: 60
Destroyer Grey Death hull integrity =: 140
Destroyer Grey Death damage taken =: 120
Destroyer Grey Death hull integrity =: 20
Destroyer Grey Death damage taken =: 60
Destroyer Grey Death hull integrity =: -40
Destroyer: Destroyer Grey Death has been sunk
```

如果有任何代码或输出似乎不匹配，请务必再次查看。

# 引用介绍

此时你可能会有一个困扰的想法。再次查看`Shipyard`类中的两个函数：

```kt
fun serviceDestroyer(destroyer: Destroyer){
        destroyer.serviceShip()
}

fun serviceCarrier(carrier: Carrier){
        carrier.serviceShip()
}
```

当我们调用那些函数并将`friendlyDestroyer`和`friendlyCarrier`传递给它们相应的`service…`函数时，我们从输出的前后看到，实例内的值已经改变了。通常，如果我们想保留函数的结果，我们需要使用返回值。发生的是，与具有常规类型参数的函数不同，当我们传递一个类的实例时，我们实际上是传递了**引用**到实例本身 - 不仅仅是其中的值的副本，而是实际的实例。

此外，所有与船相关的不同实例都是用`val`声明的，那么我们如何改变任何属性呢？对这个谜团的简短回答是，我们并没有改变引用本身，只是其中的属性，但显然需要进行更充分的讨论。

我们将开始探讨引用，然后深入探讨其他相关主题，比如第十二章中的 Android 设备内存，*将我们的 Kotlin 连接到 UI 和可空性*。目前，知道当我们将数据传递给函数时，如果它是一个类类型，我们传递的是一个等效的引用（虽然实际上并非如此）到真实的实例本身。

# 总结

我们终于写了我们的第一个类。我们已经看到我们可以在与类同名的文件中实现一个类。类本身在我们实例化一个对象/类的实例之前并不做任何事情。一旦我们有了一个类的实例，我们就可以使用它的特殊变量，称为属性，以及它的非私有函数。正如我们在基本类应用程序中证明的那样，每个类的实例都有自己独特的属性，就像当你买一辆工厂生产的汽车时，你会得到自己独特的方向盘、卫星导航和加速条纹。我们还遇到了引用的概念，这意味着当我们将一个类的实例传递给一个函数时，接收函数就可以访问实际的实例。

所有这些信息都会引发更多的问题。面向对象编程就是这样。因此，让我们在下一章中通过更仔细地研究继承来巩固所有这些类的内容。
