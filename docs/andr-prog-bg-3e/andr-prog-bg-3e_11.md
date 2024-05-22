# 第十一章：更多面向对象编程

本章是我们对面向对象编程的风潮之旅（理论和实践）的第二部分。我们已经简要讨论了封装、继承和多态性的概念，但在本章中，我们将看到它们在一些演示应用程序中更加实际的运用。虽然工作示例将展示这些概念以其最简单的形式，但这仍然是朝着通过我们的 Java 代码控制布局的重要一步。

在本章中，我们将探讨以下内容：

+   深入了解封装及其帮助我们的方式

+   深入了解继承及如何充分利用

+   更详细地解释多态性

+   静态类及我们已经在使用的方式

+   抽象类和接口

首先，我们将处理封装。

# 技术要求

您可以在 GitHub 上找到本章中的代码文件[`github.com/PacktPublishing/Android-Programming-for-Beginners-Third-Edition/tree/main/chapter%2011`](https://github.com/PacktPublishing/Android-Programming-for-Beginners-Third-Edition/tree/main/chapter%2011)。

# 还记得封装吗？

到目前为止，我们真正看到的是一种代码组织约定，我们编写类，充满了变量和方法。我们确实讨论了所有这些面向对象编程的更广泛目标，但现在我们将进一步探讨，并开始看到我们如何实际通过面向对象编程实现封装。

封装的定义

封装描述了对象隐藏其数据和方法不让外界访问的能力，只允许访问您选择的变量和方法。这意味着您的代码始终可以更新、扩展或改进，而不会影响使用它的程序——只要暴露的部分仍然以相同的方式可访问。它还允许使用您封装的代码的代码变得更简单、更易于维护，因为任务的大部分复杂性都封装在您的代码中。

但是你不是说我们不需要知道内部发生了什么吗？所以你可能会像这样质疑我们迄今为止所看到的东西：如果我们不断地设置实例变量，比如`rambo.health = 100;`，难道不可能最终出现问题，比如这样吗？

```kt
rambo.soldierType = "fluffy bunny";
```

封装保护了您的类的对象，使其无法以其不应有的方式使用。通过控制类代码的使用方式，它只能做您想要做的事情，并且具有您可以控制的值范围。

它不会被强制出现错误或崩溃。此外，您可以自由地更改代码的内部工作方式，而不会破坏程序的其余部分或使用旧版本代码的任何程序：

```kt
weightlifter.legstrength = 100;
weightlifter.armstrength = 1;
weightlifter.liftHeavyWeight();
// one typo and weightlifter rips own arms off
```

封装不仅对于编写其他人将使用的代码（例如我们使用的 Android API）至关重要，而且在编写自己将重复使用的代码时也是必不可少的，因为它将使我们免受自己的错误。此外，程序员团队将广泛使用封装，以便团队的不同成员可以在同一程序上工作，而不需要所有团队成员都知道其他团队成员的代码如何工作。我们可以为了获得同样的优势而封装我们的类，以下是如何做到的。

## 使用访问修饰符控制类的使用

类的设计者控制着任何使用其类的程序所能看到和操作的内容。我们可以像这样添加一个`class`关键字：

```kt
public class Soldier{
   //Implementation goes here
}
```

### 类访问修饰符

到目前为止，我们已经讨论了上下文中类的两个主要访问修饰符。让我们依次简要地看一下每一个：

+   `public`：这很简单。声明为 public 的类可以被所有其他类看到。

+   `default`：当未指定访问修饰符时，类具有默认访问权限。这将使其对同一包中的类公开，但对所有其他类不可访问。

现在我们可以开始封装这个东西了。但是，即使乍一看，所描述的访问修饰符也不是非常精细。我们似乎只能完全封锁包外的任何东西，或者完全自由。

实际上，这里的好处很容易被利用。想法是设计一个包含一组任务的类包。然后，包的所有复杂内部工作，那些不应该被任何人干扰的东西，应该是默认访问权限（只能被包内的类访问）。然后我们可以提供一些精心选择的公共类，供其他人（或程序的其他不同部分）使用。

重要说明

对于本书中应用程序的大小和复杂性来说，创建多个包是过度的。当然，我们将使用其他人的包和类（公共部分），所以了解这些内容是值得的。

### 类访问权限总结

一个设计良好的应用程序可能由一个或多个包组成，每个包只包含默认或默认和公共类。

除了类级别的隐私控制之外，Java 还为我们程序员提供了非常精细的控制，但要使用这些控制，我们必须更详细地研究变量。

## 使用访问修饰符控制变量的使用

为了加强类的可见性控制，我们有变量访问修饰符。这是一个声明了`private`访问修饰符的变量：

```kt
private int myInt;
```

还要注意，我们对变量访问修饰符的所有讨论也适用于对象变量。例如，这里声明、创建和分配了我们的`Soldier`类的一个实例。如你所见，这种情况下指定的访问权限是公共的：

```kt
public Soldier mySoldier = new Soldier();
```

在将修饰符应用于变量之前，必须首先考虑类的可见性。如果类*a*对类*b*不可见，比如因为类*a*具有默认访问权限，而类*b*在另一个包中，那么在类*a*的变量上使用任何访问修饰符都没有任何影响 - 类*b*无法看到其中任何一个。

因此，有必要在必要时向另一个类显示一个类，但只公开直接需要的变量 - 而不是所有的变量。

以下是不同变量访问修饰符的解释。

### 变量访问修饰符

变量访问修饰符比类访问修饰符更多，也更精细。访问修改的深度和复杂性不在于修饰符的范围，而在于我们可以如何巧妙地组合它们以实现封装的可贵目标。以下是变量访问修饰符：

+   `public`：你猜对了，任何包中的任何类或方法都可以看到这个变量。只有当你确定这就是你想要的时候才使用`public`。

+   `protected`：这是继`public`之后的下一个最不限制的。只要它们在同一个包中，`protected`变量可以被任何类和任何方法看到。

+   `default`：`default`听起来不像`protected`那么限制，但实际上更加限制。当没有指定访问权限时，变量具有`default`访问权限。`default`限制的事实或许意味着我们应该考虑隐藏我们的变量，而不是暴露它们。在这一点上，我们需要介绍一个新概念。你还记得我们曾简要讨论过继承以及如何可以快速地继承一个类的属性，然后使用`extends`关键字对其进行改进吗？只是为了记录，`default`访问权限的变量对子类是不可见的；也就是说，当我们像对 Activity 一样扩展一个类时，我们无法看到它的默认变量。我们将在本章后面更详细地讨论继承。

+   `private`：`private`变量只能在声明它们的类内部可见。这意味着，与默认访问权限一样，它们对子类（从所讨论的类继承的类）也是不可见的。

### 变量访问权限总结

一个设计良好的应用程序可能由一个或多个包组成，每个包只包含`default`或`default`和`public`类。在这些类中，变量将具有精心选择和不同的访问修饰符，以实现我们封装目标的目标。

在我们开始实际操作之前，让我们再谈一下所有这些访问修改的东西中的一个小技巧。

## 方法也可以有访问修饰符

我们已经在*第九章**，学习 Java 方法*中简要提到，方法可以有访问修饰符。这是有道理的，因为方法是我们的类可以*做*的事情。我们将想要控制我们的类的用户可以做什么和不能做什么。

这里的一般想法是，一些方法将只在内部执行操作，因此不需要类的用户，而一些方法将是类的用户使用类的基础。

### 方法访问修饰符

方法的访问修饰符与类变量的访问修饰符相同。这使得事情容易记住，但再次表明，成功的封装是一种设计问题，而不是遵循任何特定规则的问题。

作为一个例子，只要它在一个公共类中，这种方法可以被任何其他类使用：

```kt
public useMeEverybody(){
   //do something everyone needs to do here
}
```

而这个方法只能被它所属的类内部使用：

```kt
private secretInternalTask(){
   /*
         do something that helps the class function 
         internally
         Perhaps, if it is part of the same class,
         useMeEverybody could use this method...
         On behalf of the classes outside of this class.
         Neat!
   */
}
```

下一个没有指定访问权限的方法具有默认可见性。它只能被同一包中的其他类使用。如果我们扩展持有此“默认”访问方法的类，子类将无法访问此父类的方法：

```kt
fairlySecretTask(){
   // allow just the classes in the package
   // Not for external use
}
```

在我们继续之前的最后一个例子中，这是一个`protected`方法，只对包可见，但可以被我们扩展它的类使用-就像`onCreate`一样：

```kt
protected packageTask(){
   // Allow just the classes in the package
   // And you can use me if you extend me too
}
```

让我们快速回顾一下方法封装，但请记住，你不需要记住所有的东西。

### 方法访问总结

方法访问应该被选择为最好地执行我们已经讨论过的原则。它应该为你的类的用户提供他们所需要的访问权限，最好是没有更多。通过这样做，我们实现了我们的封装目标，比如保护代码的内部工作免受使用它的程序的干扰，出于我们已经讨论过的所有原因。

## 使用 getter 和 setter 访问私有变量

现在，如果将变量隐藏为私有是最佳实践，我们需要考虑如何允许访问它们，而不破坏我们的封装。如果`Hospital`类的对象想要访问`Soldier`类型的对象的`health`成员变量，以便增加它，`health`变量应该是私有的，因为我们不希望任何代码片段都可以更改它。

为了能够尽可能多地将成员变量设置为私有，同时仍然允许对其中一些进行有限访问，我们使用**getter**和**setter**。Getter 和 setter 只是获取和设置变量值的方法。

这不是我们必须学习的一些特殊的新的 Java 东西。这只是一个使用我们已经知道的东西的惯例。让我们看一下使用我们的`Soldier`类和`Hospital`类示例的 getter 和 setter。

在这个例子中，我们的两个类分别在自己的文件中创建，但在同一个包中。首先，这是我们假设的`Hospital`类：

```kt
class Hospital{
   private void healSoldier(Soldier soldierToHeal){
         int health = soldierToHeal.getHealth();
         health = health + 10;
         soldierToHeal.setHealth(health);
   }
}
```

我们的`Hospital`类的实现只有一个方法，`healSoldier`。它接收一个`Soldier`对象的引用作为参数。因此，这个方法将在传入的任何`Soldier`对象上工作：`vassily`，`wellington`，`rambo`，或其他人。

它还有一个本地的`health`变量，它用来临时保存并增加士兵的健康。在同一行中，它将`health`变量初始化为`Soldier`对象的当前健康状况。`Soldier`对象的健康状况是私有的，因此使用公共的`getHealth` getter 方法。

然后`health`增加了 10，`setHealth` setter 方法加载了新的恢复后的健康值，返回到`Soldier`对象。

关键在于，尽管`Hospital`对象可以改变`Soldier`对象的健康状况，但它只能在 getter 和 setter 方法的范围内这样做。getter 和 setter 方法可以被编写来控制和检查可能错误的，甚至有害的值。

接下来，看看我们刚刚使用的假设的`Soldier`类，它具有最简单的 getter 和 setter 方法的实现：

```kt
public class Soldier{

   private int health;
   public int getHealth(){
         return health;
   }
   public void setHealth(int newHealth){
         // Check for bad values of newHealth
         health = newHealth;
   }
}
```

我们有一个名为`health`的实例变量，它是私有的。私有意味着它只能被`Soldier`类的方法更改。然后我们有一个公共的`getHealth`方法，它返回私有的`health` int 变量中保存的值。由于这个方法是公共的，任何具有`Soldier`类型对象访问权限的代码都可以使用它。

接下来，实现了`setHealth`方法。同样，它是公共的，但这次它接受一个`int`作为参数，并将传入的任何内容分配给私有的`health`变量。在更像生活的例子中，我们会在这里编写更多的代码，以确保传入的值在我们期望的范围内。

现在我们声明、创建和赋值，创建每个新类的对象，并看看我们的 getter 和 setter 是如何工作的：

```kt
Soldier mySoldier = new Soldier();
// mySoldier.health = 100;//Doesn't work, private
// we can use the public setter setHealth() instead
mySoldier.setHealth(100); //That's better
Hospital militaryHospital = new Hospital();
// Oh no mySoldier has been wounded
mySoldier.setHealth(10);
/*        
   Take him to the hospital.
   But my health variable is private
   And Hospital won't be able to access it
   I'm doomed - tell Laura I love her
   No wait- what about my public getters and setters?
   We can use the public getters and setters 
   from another class
*/
militaryHospital.healSoldier(mySoldier);
// mySoldiers private variable health has been increased 
// by 10\. I'm feeling much better thanks!
```

我们看到我们可以直接在我们的`Soldier`类型的对象上调用我们的公共`setHealth`和`getHealth`方法。不仅如此，我们还可以调用`Hospital`对象的`healSoldier`方法，传入对`Soldier`对象的引用，后者也可以使用公共的 getter 和 setter 来操作私有的`health`变量。

我们看到私有的`health`变量是可以直接访问的，但完全受`Soldier`类的设计者控制。

如果你想尝试一下这个示例，*第十一章*文件夹中的代码包中有一个名为`GettersAndSetters`的工作应用程序的代码。我已经添加了几行代码来打印到控制台。

重要提示

Getter 和 setter 有时被称为它们更正确的名称**访问器**和**修改器**。我们将坚持使用 getter 和 setter。我只是想让你知道这个行话。

我们的示例和解释可能又引发了更多问题。这很好。

通过使用封装特性（如访问控制），就像签署了一个关于如何使用和访问类、它的方法和变量的重要协议。这份合同不仅仅是关于现在的协议，而且是对未来的暗示保证。当我们继续阅读本章时，我们会看到更多的方式来完善和加强这份合同。

提示

在需要的时候使用封装，或者当然，如果你的雇主要求你使用它的话。在一些小型学习项目中，如本书中的一些示例中，封装通常是多余的。当然，除非你学习的主题就是封装本身。

我们在学习这些 Java OOP 的东西时，假设你将来会想要编写更复杂的应用程序，无论是在 Android 上还是其他使用 OOP 的平台上。此外，我们将使用 Android API 中广泛使用它的类，并且这也将帮助我们理解那时发生了什么。通常情况下，在本书中，我们将在实现完整项目时使用封装，并经常忽略它，当展示单个想法或主题的小代码示例时。

## 使用构造函数设置我们的对象

有了这些私有变量及其 getter 和 setter，这是否意味着我们需要为每个私有变量都需要一个 getter 和 setter？那么对于一个有很多需要在开始时初始化的变量的类呢？想想以下情况：

```kt
mySoldier.name
mysoldier.type
mySoldier.weapon
mySoldier.regiment
...
```

其中一些变量可能需要 getter 和 setter，但如果我们只想在对象首次创建时设置一些东西，以使对象正确运行呢？

我们肯定不需要为每个变量都有两个方法（一个 getter 和一个 setter）吧？

幸运的是，这是不必要的。为了解决这个潜在的问题，有一个特殊的方法叫做**构造函数**。我们在*第十章**面向对象编程*中讨论实例化一个类的对象时，简要提到了构造函数的存在。让我们再看看构造函数。

在这里，我们创建了一个类型为`Soldier`的对象，并将其赋给一个名为`mySoldier`的对象：

```kt
Soldier mySoldier = new Soldier();
```

这里没有什么新的，但是看一下代码行的最后部分：

```kt
...Soldier();
```

这看起来可疑地像一个方法。

一直以来，我们一直在调用一个特殊的方法，称为构造函数，这个方法是由编译器在幕后自动创建的。

然而，现在到了重点，就像一个方法一样，我们可以*覆盖*构造函数，这意味着我们可以在使用新对象之前对其进行有用的设置。下面的代码展示了我们如何做到这一点：

```kt
public Soldier(){
   // Someone is creating a new Soldier object

   health = 200;
   // more setup here
}
```

构造函数在语法上与方法有很多相似之处。但是，它只能在使用`new`关键字的情况下调用，并且它是由编译器自动为我们创建的 - 除非我们像在先前的代码中那样创建自己的构造函数。

构造函数具有以下特点：

+   它们没有返回类型

+   它们与类具有完全相同的名称

+   它们可以有参数

+   它们可以被重载

在这一点上，还有一些 Java 语法是有用的，那就是 Java 的`this`关键字。

当我们想要明确指出我们正在引用哪些变量时，就会使用`this`关键字。再看看这个例子构造函数，再看一个假设的`Soldier`类的变体：

```kt
public class Soldier{
   String name;
   String type;
   int health;
   // This is the constructor
   // It is called when a new instance is created
   public Soldier(String name, String type, int health){
          // Someone is creating a new Soldier object

          this.name = name;
          this.type = type;
          this.health = health;
          // more setup here
   }
}
```

这次，构造函数为每个我们想要初始化的变量都有一个参数。通过使用`this`关键字，当我们指的是成员变量或参数时就很清楚。

关于变量和`this`还有更多的技巧和转折，当应用到一个实际项目时，它们会更有意义。在下一个应用程序中，我们将探索本章迄今为止学到的所有内容，还有一些新的想法。

首先，再多一点面向对象编程。

# 静态方法

我们已经对类有相当多的了解。例如，我们知道如何将它们转换为对象并使用它们的方法和变量。但是有些地方不太对。自从书的开头，我们一直在使用两个类，比其他类更频繁地使用`Log`和`Toast`来输出到 logcat 或用户的屏幕，但我们从未实例化过它们！这怎么可能呢？我们从未这样做过：

```kt
Log myLog  = new Log();
Toast myToast = new Toast();
```

我们直接使用了这些类，就像这样：

```kt
Log.i("info","our message here");
Toast.makeText(this, "our message",      
Toast.LENGTH_SHORT).show();
```

类的**静态**方法可以在没有首先实例化类的对象的情况下使用。我们可以将其视为属于类的静态方法，而所有其他方法都属于类的对象/实例。

而且你现在可能已经意识到，`Log`和`Toast`都包含静态方法。要清楚：`Log`和`Toast`*包含*静态方法；它们本身仍然是类。

类既可以有静态方法，也可以有常规方法，但是常规方法需要以常规方式使用，通过类的实例/对象。

再看一下`Log.i`的使用：

```kt
Log.i("info","our message here");
```

在这里，`i`是静态访问的方法，该方法接受两个参数，都是 String 类型。

接下来，我们看到了`Toast`类的静态方法`makeText`的使用：

```kt
Toast.makeText(this, "our message", 
Toast.LENGTH_SHORT).show();
```

`Toast`类的`makeText`方法接受三个参数。

第一个是`this`，它是对当前类的引用。我们在谈论构造函数时看到，为了明确地引用对象的当前实例的成员变量，我们可以使用`this.health`，`this.regiment`等等。

当我们像在上一行代码中那样使用`this`时，我们指的是类本身的实例；不是`Toast`类，而是上一行代码中的`this`是对方法所在的类的引用。在我们的例子中，我们已经从`MainActivity`类中使用了它。

在 Android 中，许多事情都需要引用`Activity`的实例才能完成它们的工作。在本书中，我们将经常传递`this`（对`Activity`的引用）来使 Android API 中的类/对象能够完成它们的工作。我们还将编写需要`this`作为一个或多个方法参数的类。因此，我们将看到如何处理传递进来的`this`。

`makeText`方法的第二个参数当然是一个`String`。

第三个参数是访问一个`final`变量`LENGTH_SHORT`，同样是通过类名而不是类的实例。如果我们像下一行代码那样声明一个变量：

```kt
public static final int LENGTH_SHORT = 1;
```

如果变量是在一个名为`MyClass`的类中声明的，我们可以像这样访问变量：`MyClass.LENGTH_SHORT`，并像任何其他变量一样使用它，但`final`关键字确保变量的值永远不会改变。这种类型的变量被称为**常量**。

`static`关键字对变量也有另一个影响，特别是当它不是一个常量（可以改变）时，我们将在我们的下一个应用程序中看到它的作用。

现在，如果你仔细看代码行的最后，显示一个`Toast`消息给用户，你会看到另一个新的东西，`.show()`。

这被称为`Toast`类，但只使用了一行代码。实际触发消息的是`show`方法。

当我们继续阅读本书时，我们将更仔细地研究链式调用，比如在*第十四章*中，*Android 对话框窗口*，当我们创建一个弹出对话框时。

提示

如果你想详细了解`Toast`类及其其他一些方法，你可以在这里查看：[`developer.android.com/reference/android/widget/Toast.html`](http://developer.android.com/reference/android/widget/Toast.html)。

静态方法通常在具有如此通用用途的类中提供，以至于创建该类的对象是没有意义的。另一个非常有用的具有静态方法的类是`Math`。这个类实际上是 Java API 的一部分，而不是 Android API 的一部分。

提示

想写一个计算器应用程序吗？使用`Math`类的静态方法比你想象的要容易。你可以在这里查看它们：[`docs.oracle.com/javase/7/docs/api/java/lang/Math.html`](http://docs.oracle.com/javase/7/docs/api/java/lang/Math.html)。

如果你尝试这个，你需要以同样的方式导入`Math`类，就像你导入我们使用过的所有其他类一样。接下来，我们可以尝试一个实际的迷你应用程序来理解封装和静态方法。

# 封装和静态方法迷你应用程序

我们已经看到了对变量和它们的作用域的访问是如何受控制的，我们最好看一个例子来了解它们的作用。这些不会是变量使用的实际实例，更多的是一个演示，以帮助理解类、方法和变量的访问修饰符，以及引用或原始和局部或实例变量的不同类型，以及静态和最终变量和`this`关键字的新概念。

完成的代码在下载包的*第十一章*文件夹中。它被称为`Access Scope This And Static`。

创建一个新的空活动项目，并将其命名为`Access Scope This And Static`。

通过右键单击项目资源管理器中的现有`MainActivity`类并单击`AlienShip`来创建一个新类。

现在，我们将声明我们的新类和一些成员变量。请注意，`numShips`是`private`和`static`。我们很快将看到这个变量在类的所有实例中是相同的。`shieldStrength`变量是`private`，`shipName`是公共的：

```kt
public class AlienShip {

   private static int numShips;
   private int shieldStrength;
   public String shipName;
```

接下来是构造函数。我们可以看到构造函数是公共的，没有返回类型，并且与类名相同-根据规则。在其中，我们递增了私有静态的`numShips`变量。请记住，每当我们创建一个新的`AlienShip`类型的对象时，这将发生。此外，构造函数使用私有的`setShieldStrength`方法为私有变量`shieldStrength`设置一个值：

```kt
public AlienShip(){
   numShips++;
   /*
         Can call private methods from here because I am 
         part of the class.
         If didn't have "this" then this call 
         might be less clear
         But this "this" isn't strictly necessary
         Because of "this" I am sure I am setting 
         the correct shieldStrength
   */
   this.setShieldStrength(100);
}
```

这是公共静态的 getter 方法，这样`AlienShip`外部的类就可以找出有多少个`AlienShip`对象了。我们还将看到我们如何使用静态方法：

```kt
    public static int getNumShips(){
        return numShips;
    }
```

这是我们的私有`setShieldStrength`方法。我们本可以直接从类内部设置`shieldStrength`，但下面的代码显示了我们如何通过使用`this`关键字区分`shieldStrength`局部变量/参数和`shieldStrength`成员变量：

```kt
private void setShieldStrength(int shieldStrength){

   // "this" distinguishes between the 
   // member variable shieldStrength
   // And the local variable/parameter of the same name
   this.shieldStrength = shieldStrength;

}
```

接下来的方法是 getter，这样其他类就可以读取但不能更改每个`AlienShip`对象的护盾强度：

```kt
public int getShieldStrength(){
    return this.shieldStrength;
}
```

现在我们有一个公共方法，每次击中`AlienShip`对象时都可以调用。它只是打印到控制台，然后检测该对象的`shieldStrength`是否为零。如果是，它调用`destroyShip`方法，我们将在下面看到：

```kt
public void hitDetected(){
    shieldStrength -=25;
    Log.i("Incomiming: ","Bam!!");
    if (shieldStrength == 0){
        destroyShip();
    }
}
```

最后，对于我们的`AlienShip`类，我们将编写`destroyShip`方法。我们打印一条消息，指示基于其`shipName`已被销毁的飞船，并递减`numShips`静态变量，以便我们可以跟踪类型`AlienShip`的对象数量：

```kt
   private void destroyShip(){
         numShips--;
         Log.i("Explosion: ", ""+this.shipName + " 
         destroyed");
    }
} // End of the class
```

现在我们切换到我们的`MainActivity`类，并编写一些使用我们新的`AlienShip`类的代码。所有代码都放在`setContentView`调用之后的`onCreate`方法中。首先，我们创建两个名为`girlShip`和`boyShip`的新`AlienShip`对象：

```kt
// every time we do this the constructor runs
AlienShip girlShip = new AlienShip();
AlienShip boyShip = new AlienShip();
```

在下一个代码中，看看我们如何获取`numShips`的值。我们使用`getNumShips`方法，就像我们所期望的那样。但是，仔细看语法。我们使用的是类名，而不是对象。我们还可以使用不是静态的方法访问静态变量。我们这样做是为了看到静态方法的运行方式：

```kt
// Look no objects but using the static method
Log.i("numShips: ", "" + AlienShip.getNumShips());
```

现在，我们为我们的公共`shipName`字符串变量分配名称：

```kt
// This works because shipName is public
girlShip.shipName = "Corrine Yu";
boyShip.shipName = "Andre LaMothe";
```

在接下来的代码中，我们尝试直接为私有变量分配一个值。这是行不通的。然后我们使用公共的 getter 方法`getShieldStrength`来打印出在构造函数中分配的`shieldStrength`：

```kt
// This won't work because shieldStrength is private
// girlship.shieldStrength = 999;
// But we have a public getter
Log.i("girl shields: ", "" + girlShip.getShieldStrength());
Log.i("boy shields: ", "" + boyShip.getShieldStrength());
// And we can't do this because it's private
// boyship.setShieldStrength(1000000);
```

最后，我们通过使用`hitDetected`方法来炸毁一些东西，并偶尔检查我们两个对象的`shieldStrength`：

```kt
// let's shoot some ships
girlShip.hitDetected();
Log.i("girl shields: ", "" + girlShip.getShieldStrength());

Log.i("boy shields: ", "" + boyShip.getShieldStrength());

boyShip.hitDetected();
boyShip.hitDetected();
boyShip.hitDetected();

Log.i("girl shields: ", "" + girlShip.getShieldStrength());

Log.i("boy shields: ", "" + boyShip.getShieldStrength());

boyShip.hitDetected();//Ahhh!

Log.i("girl shields: ", "" + girlShip.getShieldStrength());
Log.i("boy shields: ", "" + boyShip.getShieldStrength());
```

当我们认为我们已经摧毁了一艘飞船时，我们再次使用我们的静态`getNumShips`方法来查看我们的静态变量`numShips`是否被`destroyShip`方法改变：

```kt

Log.i("numShips: ", "" + AlienShip.getNumShips());
```

运行演示并查看控制台输出：

```kt
numShips: 2
girl shields: 100
boy shields: 100
Incomiming: Bam!!
girl shields:﹕ 75
boy shields:﹕ 100
Incomiming: Bam!!
Incomiming: Bam!!
girl shields:﹕ 75
boy shields:﹕ 25
Incomiming: Bam!!
Explosion: Andre LaMothe destroyed
girl shields: 75
boy shields: 0
numShips: 1
boy shields: 0
numShips: 1
```

在前面的示例中，我们看到我们可以通过使用`this`关键字区分相同名称的局部变量和成员变量。我们还可以使用`this`关键字编写代码，引用当前正在操作的对象。

我们看到静态变量-在这种情况下，`numShips`-在所有实例中是一致的；此外，通过在构造函数中递增它，并在我们的`destroyShip`方法中递减它，我们可以跟踪我们当前拥有的`AlienShip`对象的数量。

我们还看到，我们可以使用静态方法，使用类名和点运算符而不是实际对象。

提示

是的，我知道这就像生活在房子的蓝图中一样-但这也非常有用。

最后，我们证明了如何使用访问修饰符隐藏和公开某些方法和变量。

接下来，我们将看一下继承的主题。

# 面向对象编程和继承

我们已经看到，我们可以通过实例化/创建来自 Android 等 API 的类的对象来使用其他人的代码。但是这整个 OOP 的东西甚至比那更深入。

如果有一个类有很多有用的功能，但不完全符合我们的要求，我们可以从该类继承，然后进一步完善或添加其工作方式和功能。

你可能会惊讶地听到我们已经这样做了。实际上，我们已经为我们创建的每个应用程序都这样做了。当我们使用`extends`关键字时，我们正在继承。记住这一点：

```kt
public class MainActivity extends AppCompatActivity ...
```

在这里，我们继承了`AppCompatActivity`类以及它的所有功能-更具体地说，类设计者希望我们能够访问的所有功能。以下是我们可以对我们扩展的类做的一些事情。

我们甚至可以重写一个方法*并且*仍然部分依赖于我们继承的类中的重写方法。例如，我们每次扩展`AppCompatActivity`类时都重写了`onCreate`方法。但是当我们这样做时，我们也调用了类设计者提供的默认实现：

```kt
super.onCreate(... 
```

在*第六章*，*Android 生命周期*中，我们重写了几乎所有 Activity 类的生命周期方法。

我们主要讨论继承，以便我们了解周围发生的事情，并作为最终能够设计有用的类的第一步，我们或其他人可以扩展。

考虑到这一点，让我们看一些示例类，并看看我们如何扩展它们，只是为了看看语法并作为第一步，也为了能够说我们已经这样做了。

当我们看这一章的最后一个主要主题，多态性时，我们也将同时深入研究继承。这里有一些使用继承的代码。

这段代码将放在一个名为`Animal.java`的文件中：

```kt
public class Animal{
   // Some member variables
   public int age;
   public int weight;
   public String type;
   public int hungerLevel;
   public void eat(){
          hungerLevel--;
   }
   public void walk(){
          hungerLevel++;
   }
}
```

然后在一个名为`Elephant.java`的单独文件中，我们可以这样做：

```kt
public class Elephant extends Animal{

   public Elephant(int age, int weight){
         this.age = age;
         this.weight = weight;
         this.type = "Elephant";
         int hungerLevel = 0;
   }
}
```

我们可以看到在前面的代码中，我们实现了一个名为`Animal`的类，它有四个成员变量：`age`，`weight`，`type`和`hungerLevel`。它还有两个方法，`eat`和`walk`。

然后我们用`Elephant`扩展了`Animal`。`Elephant`现在可以做任何`Animal`可以做的事情，它也有所有的变量。

我们在`Elephant`构造函数中初始化了`Animal`的变量，`Elephant`在创建对象时将两个变量（`age`和`weight`）传递给构造函数，并且为所有`Elephant`对象分配了两个变量（`type`和`hungerLevel`）。

我们可以继续编写一堆其他扩展`Animal`的类，也许是`Lion`，`Tiger`和`ThreeToedSloth`。每个类都会有`age`，`weight`，`type`和`hungerLevel`，并且每个类都能`walk`和`eat`。

好像 OOP 已经不够有用了，我们现在可以模拟现实世界的对象。我们还看到，通过子类化/扩展/继承其他类，我们可以使 OOP 变得更加有用。我们可能想要学习的术语是被扩展的类是**超类**，继承超类的类是**子类**。我们也可以说父类和子类。

提示

像往常一样，我们可能会问关于继承的这个问题。为什么？原因是这样的：我们可以在父类中编写一次通用代码；我们可以更新该通用代码，所有继承自它的类也会更新。此外，子类只能使用公共/受保护的实例变量和方法。因此，如果设计得当，这也进一步增强了封装的目标。

让我们构建另一个小应用程序来玩一下继承。

# 继承示例应用程序

我们已经看过了如何创建类的层次结构来模拟适合我们应用程序的系统。因此，让我们尝试一些使用继承的简单代码。完成的代码在*Chapter 11*文件夹中。它被称为`Inheritance Example`。

使用`AlienShip`、另一个`Fighter`和最后一个`Bomber`创建一个名为`Inheritance Example`的新项目。

以下是`AlienShip`类的代码。它与我们之前的类演示`AlienShip`非常相似。不同之处在于构造函数现在接受一个`int`参数，它用于设置护盾强度。

构造函数还会将消息输出到 logcat 窗口，这样我们就可以看到它何时被使用。`AlienShip`类还有一个新方法`fireWeapon`，声明为`abstract`。

将一个类声明为抽象类可以保证任何子类`AlienShip`都必须实现其自己的`fireWeapon`版本。注意类的声明中有`abstract`关键字。我们必须这样做是因为它的一个方法也使用了`abstract`关键字。当我们讨论这个演示和下一节中讨论多态时，我们将解释`abstract`方法和`abstract`类。

将以下代码添加到`AlienShip`类中：

```kt
public abstract class AlienShip {
    private static int numShips;
    private int shieldStrength;
    public String shipName;
    public AlienShip(int shieldStrength){
        Log.i("Location: ", "AlienShip constructor");
        numShips++;
        setShieldStrength(shieldStrength);
    }
    public abstract void fireWeapon();
    // Ahh my body where is it?
    public static int getNumShips(){
        return numShips;
    }
    private void setShieldStrength(int shieldStrength){
        this.shieldStrength = shieldStrength;
    }
    public int getShieldStrength(){
        return this.shieldStrength;
    }
    public void hitDetected(){
        shieldStrength -=25;
        Log.i("Incomiming: ", "Bam!!");
        if (shieldStrength == 0){
            destroyShip();
        }
    }
    private void destroyShip(){
        numShips--;
        Log.i("Explosion: ", "" + this.shipName + " 
        destroyed");
    }
}
```

现在我们将实现`Bomber`类。注意调用`super(100)`。这将使用`shieldStrength`的值调用超类的构造函数。我们可以在这个构造函数中进一步初始化特定的`Bomber`，但现在，我们只是打印出位置，这样我们就可以看到`Bomber`构造函数何时被执行。因为我们必须，我们还必须实现一个抽象`fireWeapon`方法的`Bomber`特定版本。将以下代码添加到`Bomber`类中：

```kt
public class Bomber extends AlienShip {
    public Bomber(){
        super(100);
        // Weak shields for a bomber
        Log.i("Location: ", "Bomber constructor");
    }
    public void fireWeapon(){
        Log.i("Firing weapon: ", "bombs away");
    }
}
```

现在我们将实现`Fighter`类。注意调用`super(400)`。这将使用`shieldStrength`的值调用超类的构造函数。我们可以在这个构造函数中进一步初始化特定的`Fighter`，但现在，我们只是打印出位置，这样我们就可以看到`Fighter`构造函数何时被执行。我们还必须实现一个抽象`fireWeapon`方法的`Fighter`特定版本。将以下代码添加到`Fighter`类中：

```kt
public class Fighter extends AlienShip{
    public Fighter(){
        super(400);
        // Strong shields for a fighter
        Log.i("Location: ", "Fighter constructor");
    }
    public void fireWeapon(){
        Log.i("Firing weapon: ", "lasers firing");
    }
}
```

接下来，我们将编写`MainActivity`的`onCreate`方法。像往常一样，在调用`setContentView`方法之后输入此代码。这是使用我们的三个新类的代码。代码看起来相当普通 - 没有什么新东西。有趣的是输出：

```kt
Fighter aFighter = new Fighter();
Bomber aBomber = new Bomber();
// Can't do this AlienShip is abstract -
// Literally speaking as well as in code
// AlienShip alienShip = new AlienShip(500);
// But our objects of the subclasses can still do
// everything the AlienShip is meant to do
aBomber.shipName = "Newell Bomber";
aFighter.shipName = "Meier Fighter";
// And because of the overridden constructor
// That still calls the super constructor
// They have unique properties
Log.i("aFighter Shield:", ""+ aFighter.getShieldStrength());
Log.i("aBomber Shield:", ""+ aBomber.getShieldStrength());
// As well as certain things in certain ways
// That are unique to the subclass
aBomber.fireWeapon();
aFighter.fireWeapon();
// Take down those alien ships
// Focus on the bomber it has a weaker shield
aBomber.hitDetected();
aBomber.hitDetected();
aBomber.hitDetected();
aBomber.hitDetected();
```

运行应用程序，您将在 logcat 窗口中获得以下输出：

```kt
Location:﹕ AlienShip constructor
Location:﹕ Fighter constructor
Location:﹕ AlienShip constructor
Location:﹕ Bomber constructor
aFighter Shield:﹕ 400
aBomber Shield:﹕ 100
Firing weapon:﹕ bombs away
Firing weapon:﹕ lasers firing
Incomiming:﹕ Bam!!
Incomiming:﹕ Bam!!
Incomiming:﹕ Bam!!
Incomiming:﹕ Bam!!
Explosion:﹕ Newell Bomber destroyed
```

我们可以看到子类的构造函数如何调用超类的构造函数。我们还可以清楚地看到`fireWeapon`方法的各个实现正如预期地工作。

让我们更仔细地看一下最后一个重要的面向对象编程概念，多态。然后我们将能够在 Android API 中做一些更实际的事情。

# 多态

我们已经知道多态意味着*不同的形式*。但对我们来说意味着什么呢？

简而言之：

任何子类都可以作为使用超类的代码的一部分。

这意味着我们可以编写更简单、更容易理解和更容易更改的代码。

此外，我们可以为超类编写代码，并依赖于这样一个事实：无论它被子类化多少次，在一定的参数范围内，代码仍然可以正常工作。让我们讨论一个例子。

假设我们想要使用多态来帮助编写一个动物园管理应用程序。我们可能希望有一个`feed`之类的方法。我们可能希望将要喂养的动物的引用传递到`feed`方法中。这似乎需要为每种类型的`Animal`编写一个`feed`方法。

然而，我们可以编写具有多态返回类型和参数的多态方法：

```kt
Animal feed(Animal animalToFeed){
   // Feed any animal here
   return animalToFeed;
}
```

前面的方法将`Animal`作为参数，这意味着可以将从扩展`Animal`的类构建的任何对象传递给它。正如你在前面的代码中看到的，该方法也返回`Animal`，具有完全相同的好处。

多态返回类型有一个小问题，那就是我们需要意识到返回的是什么，并且在调用方法的代码中明确表示出来。

例如，我们可以像这样处理将`Elephant`传递到`feed`方法中：

```kt
someElephant = (Elephant) feed(someElephant);
```

注意前面代码中的`(Elephant)`。这明确表示我们想要从返回的`Animal`中得到`Elephant`。这被称为**转换**。我们将在本书的其余部分偶尔使用转换。

因此，你甚至可以*今天*编写代码，并在一周、一个月或一年后创建另一个子类，而相同的方法和数据结构仍然可以工作。

此外，我们可以对我们的子类强制执行一组规则，规定它们可以做什么，不能做什么，以及如何做。因此，一个阶段的巧妙设计可以影响其他阶段。

但我们真的会想要实例化一个实际的`Animal`吗？

## 抽象类

在继承示例应用程序中，我们使用了一个抽象类，但让我们深入一点。抽象类是一个不能被实例化的类；它不能被制作成一个对象。那么，它就是一个永远不会被使用的蓝图？但这就像付钱给建筑师设计你的房子，然后永远不建造它？你可能会对自己说，“我有点明白了抽象方法的概念，但抽象类就是愚蠢。”

如果我们或类的设计者想要强制我们在使用他们的类之前继承，他们可以将一个类声明为**abstract**。然后，我们就不能从中创建一个对象；因此，我们必须首先扩展它并从子类创建一个对象。

我们还可以声明一个方法为`abstract`，然后该方法必须在扩展具有抽象方法的类的任何类中被重写。

让我们看一个例子——这会有所帮助。我们通过像这样使用`abstract`关键字来声明一个类为`abstract`：

```kt
abstract class someClass{
   /*
         All methods and variables here.
         As usual!
         Just don't try and make 
         an object out of me!
   */
}
```

是的，但为什么？

有时我们想要一个可以用作多态类型的类，但我们需要保证它永远不能被用作对象。例如，“动物”本身并没有太多意义。

我们不谈论动物；我们谈论*动物的类型*。我们不会说，“哦，看那只可爱的毛茸茸的白色动物。”或者，“昨天我们去宠物店买了一只动物和一个动物床。”这太，嗯，*抽象*了。

因此，抽象类有点像一个模板，可以被任何`extends`它（继承自它）的类使用。

我们可能想要一个`Worker`类，并将其扩展为`Miner`、`Steelworker`、`OfficeWorker`，当然还有`Programmer`。但一个普通的`Worker`到底是做什么的呢？我们为什么要实例化一个？

答案是我们可能不想实例化一个；但我们可能想要将其用作多态类型，这样我们可以在方法之间传递多个工作子类，并且可以拥有可以容纳所有类型的`Worker`的数据结构。

我们称这种类型的类为抽象类，当一个类有一个抽象方法时，它必须被声明为抽象类。并且所有抽象方法必须被任何扩展它的类重写。

这意味着抽象类可以提供一些在其所有子类中都可用的共同功能。例如，`Worker`类可能有`height`、`weight`和`age`成员变量。

它可能还有`getPayCheck`方法，这不是抽象的，并且在所有子类中都是相同的，但`doWork`方法是抽象的，必须被重写，因为所有不同类型的工作者都会以不同的方式`doWork`。

这使我们顺利地进入了多态的另一个领域，这将在本书中为我们带来更多便利。

## 接口

接口就像一个类。哦！这里没有什么复杂的。但它就像一个始终是抽象的类，只有抽象方法。

我们可以将接口视为一个完全抽象的类，其中所有方法都是抽象的，也没有成员变量。

好吧，你大概可以理解抽象类，因为至少它可以在其方法中传递一些功能，这些功能不是抽象的，并且可以作为多态类型。但说真的，这个接口似乎有点毫无意义。

让我们看一个最简单的通用接口示例，然后我们可以进一步讨论它。

要定义一个接口，我们输入以下内容：

```kt
public interface someInterface{
   void someAbstractMethod();
   // omg I've got no body

   int anotherAbstractMethod();
   // Ahh! Me too
   // Interface methods are always abstract and public 
   // implicitly but we could make it explicit if we prefer
   public abstract void 
   explicitlyAbstractAndPublicMethod();
   // still no body though

}
```

接口的方法没有方法体，因为它们是抽象的，但它们仍然可以有返回类型和参数，或者没有。

要使用接口，我们在类声明后使用`implements`关键字：

```kt
public class someClass implements someInterface{
   // class stuff here
   /* 
         Better implement the methods of the interface 
         or we will have errors.
         And nothing will work
   */

   public void someAbstractMethod(){
         // code here if you like 
         // but just an empty implementation will do
   }
   public int anotherAbstractMethod(){
         // code here if you like 
         // but just an empty implementation will do
         // Must have a return type though 
         // as that is part of the contract
         return 1;   
   }
     Public void explicitlyAbstractAndPublicMethod(){
     }
}
```

这使我们能够使用多态性来处理来自完全不相关的继承层次结构的多个不同对象。如果一个类实现了一个接口，整个东西就可以被传递或用作它本身 - 因为它就是那个东西。它是多态的（多种形式）。

我们甚至可以让一个类同时实现多个不同的接口。只需在每个接口之间加上逗号，并在`implements`关键字后列出它们。只需确保实现所有必要的方法。

在本书中，我们将更频繁地使用 Android API 的接口，而不是编写我们自己的接口。在*第十三章**，匿名类 - 使 Android 小部件活跃*中，我们将在 Java Meet UI 应用程序中使用`OnClickListener`接口。

许多事情在被点击时可能想要知道。也许是`Button`或`TextView`小部件等等。因此，使用接口，我们不需要为每种我们想要点击的 UI 元素类型编写不同的方法。

# 经常问的问题

1.  这个类声明有什么问题？

```kt
   private class someClass{
         // class implementation goes here
   }
```

没有私有类。类可以是公共的或默认的。公共类是公共的；默认类在其自己的包内是私有的。

1.  封装是什么？

封装是我们如何以一种方式包含我们的变量、代码和方法，以仅暴露我们想要暴露给其他代码的部分和功能。

# 总结

在本章中，我们涵盖了比其他任何章节都更多的理论。如果你没有记住所有内容，或者有些代码看起来有点太深入了，那么你仍然完全成功了。

如果你只是理解 OOP 是通过封装、继承和多态编写可重用、可扩展和高效的代码，那么你就有成为 Java 大师的潜力。

简而言之，面向对象编程使我们能够在其他人不知道我们在编写代码时会做什么的情况下使用其他人的代码。

你所需要做的就是不断练习，因为我们将在整本书中不断地使用这些概念，所以你在这一点上甚至不需要已经掌握它们。

在下一章中，我们将重新讨论本章的一些概念，以及看一些 OOP 的新方面，以及它如何使我们的 Java 代码与我们的 XML 布局进行交互。

但首先，有一个重要的即将到来的新闻快讯！

重要提示

所有的 UI 元素 - `TextView`、`ConstraintLayout`、`CalenderView`和`Button` - 也是类。它们的属性是成员变量，它们有大量的方法，我们可以使用这些方法来做各种各样的事情。这可能会很有用。

在接下来的两章中，我们将更多地了解这一点，但首先，我们将看看 Android 如何处理垃圾。
