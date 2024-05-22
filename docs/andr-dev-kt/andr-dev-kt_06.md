# 第六章：泛型是你的朋友

在上一章中，我们讨论了与 Kotlin 中的函数式编程和函数作为一等公民相关的概念。

在本章中，我们将讨论泛型类型和泛型函数的概念，称为泛型。我们将学习它们存在的原因以及如何使用它们-我们将定义泛型类、接口和函数。我们将讨论如何在运行时处理泛型，看一下子类型关系，并处理泛型可空性

在本章中，我们将讨论泛型类型和泛型函数的概念，称为泛型。我们将学习它们存在的原因以及如何使用它们，以及如何定义泛型类、接口和函数。我们将讨论如何在运行时处理泛型，看一下子类型关系，并处理泛型可空性。

在本章中，我们将涵盖以下主题：

+   泛型类

+   泛型接口

+   泛型函数

+   泛型约束

+   泛型可空性

+   变异

+   使用地点目标与声明地点目标

+   声明地点目标

+   类型擦除

+   具体化和擦除类型参数

+   星投影语法

+   变异

# 泛型

**泛型**是一种编程风格，其中类、函数、数据结构或算法以后可以指定确切的类型。通常，泛型提供类型安全性以及重用特定代码结构的能力，用于各种数据类型。

泛型在 Java 和 Kotlin 中都存在。它们的工作方式类似，但 Kotlin 在 Java 泛型类型系统上提供了一些改进，比如使用地点变异、星投影语法和具体化类型参数。我们将在本章讨论它们。

# 泛型的需求

程序员经常需要一种方法来指定集合只包含特定类型的元素，比如`Int`，`Student`或`Car`。如果没有泛型，我们将需要为每种数据类型创建单独的类（`IntList`，`StudentList`，`CarList`等）。这些类的内部实现非常相似，唯一的区别在于存储的数据类型。这意味着我们需要多次编写相同的代码（比如向集合添加或删除项目）并分别维护每个类。这是很多工作，所以在实现泛型之前，程序员通常操作通用列表。这迫使他们每次访问时都需要转换元素：

```kt
    // Java

    ArrayList list = new ArrayList();

    list.add(1);

    list.add(2);

    int first = (int) list.get(0);

    int second = (int) list.get(1);
```

转换会增加样板代码，并且在将元素添加到集合时没有类型验证。泛型是这个问题的解决方案，因为泛型类定义并使用占位符而不是真实类型。这个占位符称为**类型参数**。让我们定义我们的第一个泛型类：

```kt
    class SimpleList<T> // T is type parameter
```

类型参数意味着我们的类将使用特定类型，但这种类型将在类创建期间指定。这样，我们的`SimpleList`类可以为各种类型实例化。我们可以使用*类型参数*为泛型类参数化各种数据类型。这允许我们从单个类创建多个数据类型：

```kt
     // Usage

    var intList: SimpleList<Int>

    var studentList: SimpleList<Student>

    var carList:SimpleList<Car>
```

`SimpleList`类是使用*类型参数*（`Int`，`Student`和`Car`）进行参数化的，定义了可以存储在给定列表中的数据类型。

# 类型参数与类型参数

函数具有参数（在函数声明内部声明的变量）和参数（传递给函数的实际值）。泛型也适用类似的术语。*类型参数*是泛型中声明的类型的蓝图或占位符，*类型参数*是用于参数化泛型的实际类型。

我们可以在方法签名中使用*类型参数*。这样，我们可以确保我们将能够向我们的列表添加特定类型的项目并检索特定类型的项目：

```kt
    class SimpleList<T> { 

       fun add(item:T) { // 1 
           // code 
       }  
       fun get(intex: Int): T { // 2 
           // code 
       } 
    } 
```

1.  泛型类型参数`T`用作项目类型

1.  类型参数用作返回类型

可以添加到列表或从列表中检索的项目的类型取决于*类型参数*。让我们看一个例子：

```kt
    class Student(val name: String)

    val studentList = SimpleList<Student>()

    studentList.add(Student("Ted"))

    println(studentList.getItemAt(0).name)
```

我们只能从列表中添加和获取`Student`类型的项目。编译器将自动执行所有必要的类型检查。可以保证集合只包含特定类型的对象。将不兼容类型的对象传递给 add 方法将导致编译时错误：

```kt
    var studentList: SimpleList<Student>

    studentList.add(Student("Ted"))

    studentList.add(true) // error
```

我们无法添加布尔值，因为期望的类型是`Student`。

Kotlin 标准库在`kotlin.collections`包中定义了各种通用集合，如`List`，`Set`和`Map`。我们将在第七章中讨论它们，*扩展函数和属性*。

在 Kotlin 中，通常将通用与高阶函数（在第五章中讨论）和扩展函数（我们将在第七章中讨论）结合使用。这些连接的示例是函数：`map`，`filter`，`takeUntil`等。我们可以执行通用操作，其细节将有所不同。例如，我们可以使用`filter`函数在集合中查找匹配的元素，并指定如何检测匹配的元素：

```kt
    val fruits = listOf("Babana", "Orange", "Apple", "Blueberry") 
    val bFruits = fruits.filter { it.startsWith("B") } //1 
    println(bFruits) // Prints: [Babana, Blueberry] 
```

1.  我们可以调用`startsWith`方法，因为集合只能包含`Strings`，所以 lambda 参数（`it`）具有相同的类型。

# 通用约束

默认情况下，我们可以使用任何类型的*类型参数*对通用类进行参数化。但是，我们可以限制可以用作*类型参数*的可能类型。为了限制*类型参数*的可能值，我们需要定义一个*类型参数边界*。最常见的*约束*类型是*上界*。默认情况下，所有类型参数都具有`Any?`作为隐式*上界*。这就是为什么以下两个声明是等价的：

```kt
    class SimpleList<T>

    class SimpleList<T: Any?>
```

前面的边界意味着我们可以使用任何类型作为`SimpleList`类的*类型参数*（包括可空类型）。这是可能的，因为所有可空和非可空类型都是`Any?`的子类型：

```kt
    class SimpleList<T>

    class Student

    //usage

    var intList = SimpleList<Int>()

    var studentList = SimpleList<Student>()

    var carList = SimpleList<Boolean>()
```

在某些情况下，我们希望限制可用作*类型参数*的数据类型。为了实现这一点，我们需要明确定义一个*类型参数*上界。假设我们只想能够将数值类型用作`SimpleList`类的*类型参数*：

```kt
    class SimpleList<T: Number>

    //usage

    var numberList = SimpleList<Number>()

    var intList = SimpleList<Int>()

    var doubleList = SimpleList<Double>()

    var stringList = SimpleList<String>() //error
```

`Number`类是一个抽象类，即 Kotlin 数值类型（`Byte`，`Short`，`Int`，`Long`，`Float`，和`Double`）的超类。我们可以使用`Number`类及其所有子类（`Int`，`Double`等）作为*类型参数*，但我们不能使用`String`类，因为它不是`Number`的子类。任何尝试添加不兼容类型的操作都将被 IDE 和编译器拒绝。类型参数还包括 Kotlin 类型系统的可空性。

# 可空性

当我们定义一个具有无界类型参数的类时，我们可以使用非可空和可空类型作为*类型参数*。偶尔，我们需要确保特定的通用类型不会被参数化为可空*类型参数*。为了阻止使用可空类型作为*类型参数*的能力，我们需要明确定义一个非可空**类型参数上界**：

```kt
    class Action (val name:String)

    class ActionGroup<T : Action> 

    // non-nullable type parameter upper bound

    var actionGroupA: ActionGroup<Action>

    var actionGroupB: ActionGroup<Action?> // Error
```

现在我们无法将可空的*类型参数*（`Action?`）传递给`ActionGroup`类。

让我们考虑另一个例子。假设我们想要检索`ActionGroup`中的最后一个`Action`。`last`方法的简单定义如下：

```kt
    class ActionGroup<T : Action>(private val list: List<T>) {

        fun last(): T = list.last()

    }
```

让我们分析当我们将空列表传递给构造函数时会发生什么：

```kt
    val actionGroup = ActionGroup<Action>(listOf())

    //...

    val action = actionGroup.last 

    //error: NoSuchElementException: List is empty

    println(action.name)
```

我们的应用程序崩溃了，因为当列表为空时，`last`方法会抛出错误。我们可能更喜欢在列表为空时返回空值而不是异常。Kotlin 标准库已经有了一个相应的方法，它将返回一个空值：

```kt
    class ActionGroup<T : Action>(private val list: List<T>) {

        fun lastOrNull(): T = list.lastOrNull() //error

    }
```

代码将无法编译，因为最后一个方法可能会返回 null，无论*类型参数*是否为空（列表中可能没有元素返回）。为了解决这个问题，我们需要通过在类型参数使用位置（`T?`）添加一个问号来强制可空返回类型：

```kt
    class ActionGroup<T : Action>(private val list: List<T>) { // 1

        fun lastOrNull(): T? = list.lastOrNull() // 2

    }
```

1.  类型参数*声明位置*（类型参数声明的代码位置）

1.  类型参数*使用位置*（类型参数使用的代码位置）

`T?`参数意味着`lastOrNull`方法始终是可空的，无论潜在的*类型参数*是否为空。请注意，我们将类型参数`T`的界限恢复为非空类型`Action`，因为我们希望存储非空类型并仅在某些情况下处理可空性（例如不存在的最后一个元素）。让我们使用我们更新的`ActionGroup`类：

```kt
    val actionGroup= ActionGroup<Action>(listOf())

    val actionGroup = actionGroup.lastOrNull() 

    // Inferred type is Action?

    println(actionGroup?.name) // Prints: null
```

请注意，即使我们使用非空*类型参数*对泛型进行参数化，`actionGroup`推断类型仍然是可空的。

使用位置的可空类型不会阻止我们在声明位置允许非空类型：

```kt
    open class Action

    class ActionGroup<T : Action?>(private val list: List<T>) {

        fun lastOrNull(): T? = list.lastOrNull()

    }

    // Usage

    val actionGroup = ActionGroup(listOf(Action(), null))

    println(actionGroup.lastOrNull()) // Prints: null
```

让我们总结一下上面的解决方案。我们为类型参数指定了一个非空界限，以阻止使用可空类型作为*类型参数*对`ActionGroup`类进行参数化。我们使用非空*类型参数*`Action`对`ActionGroup`类进行参数化。最后，我们在使用位置（`T?`）强制类型参数的可空性，因为如果列表中没有元素，最后一个属性可能会返回 null。

# 变化

子类型是面向对象编程范式中的一个流行概念。我们通过扩展类来定义两个类之间的继承：

```kt
    open class Animal(val name: String)

    class Dog(name: String): Animal(name)
```

类`Dog`扩展了类`Animal`，因此类型`Dog`是`Animal`的子类型。这意味着我们可以在需要`Animal`类型的表达式中使用`Dog`类型的表达式；例如，我们可以将其用作函数参数或将`Dog`类型的变量分配给`Animal`类型的变量：

```kt
    fun present(animal: Animal) {

        println( "This is ${ animal. name } " )

    }

    present(Dog( "Pluto" )) // Prints: This is Pluto
```

在我们继续之前，我们需要讨论类和类型之间的区别。类型是一个更一般的术语--它可以由类或接口定义，也可以内置到语言中（原始类型）。在 Kotlin 中，对于每个类（例如`Dog`），我们至少有两种可能的类型--非空（`Dog`）和可空（`Dog?`）。而且，对于每个泛型类（例如`class Box<T>`），我们可以定义多个数据类型（`Box*<Dog>*`，`Box<Dog?>`，`Box<Animal>`，`Box<Box<Dog>>`等）。

前面的例子仅适用于简单类型。变化指定了更复杂类型之间的子类型关系（例如`Box<Dog>`和`Box<Animal>`）与它们的组件之间的子类型关系（例如`Animal`和`Dog`）之间的关系。

在 Kotlin 中，泛型默认是*不变*的。这意味着泛型类型`Box<Dog>`和`Box<Animal>`之间没有子类型关系。`Dog`是`Animal`的子类型，但`Box<Dog>`既不是`Box<Animal>`的子类型也不是超类型：

```kt
    class Box<T>

    open class Animal

    class Dog : Animal()

    var animalBox = Box<Animal>()

    var dogBox = Box<Dog>()

    //one of the lines below line must be commented out,

    //otherwise Android Studio will show only one error

    animalBox = dogBox // 2, error

    dogBox = animalBox // 1, error
```

1.  错误类型不匹配。需要`Box<Animal>`，找到`Box<Dog>`。

1.  错误类型不匹配。需要`Box<Dog>`，找到`Box<Animal>`。

`Box<Dog>`类型既不是`Box<Animal>`的子类型也不是超类型，因此我们不能使用前面代码中显示的任何赋值。

我们可以定义`Box<Dog>`和`Box<Animal>`之间的子类型关系。在 Kotlin 中，泛型类型的子类型关系可以被保留（协变）、颠倒（逆变）或忽略（不变）。

当子类型关系是协变时，这意味着子类型关系被保留。泛型类型将与*类型参数*具有相同的关系。如果`Dog`是`Animal`的子类型，则`Box<Dog>`是`Box<Animal>`的子类型。

逆变是协变的确切相反，子类型关系被颠倒。泛型类型将与*类型参数*的关系相反。如果`Dog`是`Animal`的子类型，则`Box<Animal>`是`Box<Dog>`的子类型。以下图表显示了所有类型的变化：

！[](img/Image00043.jpg)

要定义协变或逆变行为，我们需要使用*变异修饰符*。

# 变异修饰符

Kotlin 中的泛型默认是不变的。这意味着我们需要将类型用作声明的变量或函数参数的类型：

```kt
    public class Box<T> { }

    fun sum(list: Box<Number>) { /* ... */ }

    // Usage

    sum(Box<Any>()) // Error

    sum(Box<Number>()) // Ok

    sum(Box<Int>()) // Error
```

我们不能使用泛型类型参数化为`Int`（`Number`的子类型）和`Any`（`Number`的超类型）。我们可以通过使用变异修饰符来放宽这个限制并改变默认的变异。在 Java 中，有一个问号（`?`）符号（通配符符号）用于表示未知类型。使用它，我们可以定义两种通配符边界--上界和下界。在 Kotlin 中，我们可以使用`in`和`out`修饰符来实现类似的行为。

在 Java 中，上界通配符允许我们定义一个接受任何参数的函数，该参数是其子类型的某种类型。在下面的例子中，sum 函数将接受任何使用`Number`类或`Number`类的子类型（`Box<Integer>`，`Box<Double>`等）参数化的`List`：

```kt
    //Java

    public void sum(Box<? extends Number> list) { /* ... */ }

    // Usage

    sum(new Box<Any>()) // Error

    sum(new Box<Number>()) // Ok

    sum(new Box<Int>()) // Ok
```

现在我们可以将`Box<Number>`传递给我们的 sum 函数以及所有的子类型，例如`Box<Int>`。这种 Java 行为对应于 Kotlin 的 out 修饰符。它表示协变，限制类型为特定类型或该类型的子类型。这意味着我们可以安全地传递使用任何`Number`的直接或间接子类参数化的`Box`类的实例：

```kt
    class Box<T>

    fun sum(list: Box<out Number>) { /* ... */ }

    //usage

    sum(Box<Any>()) // Error

    sum(Box<Number>()) // Ok

    sum(Box<Int>()) // Ok
```

在 Java 中，下界通配符允许我们定义一个接受任何参数的函数，该参数是某种类型或其超类型。在下面的例子中，`sum`函数将接受任何使用`Number`类或`Number`类的超类型（`Box<Number>`和`Box<Object>`）参数化的`List`：

```kt
    //Java

    public void sum(Box<? super Number> list) { /* ... */ }

    //usage

    sum(new Box<Any>()) // Ok

    sum(new Box<Number>()) // Ok

    sum(new Box<Int>()) // Error
```

现在我们可以将`Box<Any>`传递给我们的 sum 函数以及所有的子类型，例如`Box<Any>`。这种 Java 行为对应于 Kotlin 的 in 修饰符。它表示逆变，限制类型为特定类型或该类型的超类型：

```kt
    class Box<T>

    fun sum(list: Box<in Number>) { /* ... */ }

    //usage

    sum(Box<Any>()) // Ok

    sum(Box<Number>()) // Ok

    sum(Box<Int>()) // Error
```

禁止同时使用`in`和`out`修饰符。我们可以以两种不同的方式定义变异修饰符。让我们在接下来的部分看看它们。

# 使用地点变异与声明地点变异

*使用地点*变异和*声明地点*变异基本上描述了代码中指定变异修饰符的位置。让我们考虑`View`和`Presenter`的例子：

```kt
    interface BaseView

    interface ProductView : BaseView

    class Presenter<T>

    // Usage

    var preseter = Presenter<BaseView>()

    var productPresenter = Presenter<ProductView>()

    preseter = productPresenter

    // Error: Type mismatch

    // Required: Presenter<BaseView>

    // Found: Presenter<ProductView>
```

类`Presenter`在其类型`parameterT`上是不变的。为了解决问题，我们可以明确定义子类型关系。我们可以以两种方式（使用地点和声明地点）进行定义。首先，让我们在使用地点定义变异：

```kt
    var preseter: Presenter<out BaseView> = Presenter<BaseView>() //1

    var productPresenter = Presenter<ProductView>()

    preseter = productPresenter
```

1.  类型参数使用地点定义的变异修饰符

现在`preseter`变量可以存储`Presenter<BaseView>`的子类型，包括`Presenter<ProductView>`。我们的解决方案有效，但实现可以改进。这种方法有两个问题。现在我们需要每次在想要使用泛型类型时指定`out`变异修饰符，例如，在不同类中的多个变量中使用它：

```kt
    //Variable declared inside class A and class B

    var preseter = Presenter<BaseView>()

    var preseter: Presenter<out BaseView> = Presenter<ProductView>()

    preseter = productPresenter  
```

类`A`和`B`都包含具有变异修饰符的`preseter`变量。我们失去了使用类型推断的能力，结果代码更冗长。为了改进我们的代码，我们可以在类型参数声明地点指定变异修饰符：

```kt
interface BaseView

interface ProductView: BaseView

class Presenter<out T> // 1   

//usage

//Variable declared inside class A and B

var preseter = Presenter<BaseView>()

var productPresenter = Presenter<ProductView>()

preseter = productPresenter
```

1.  在类型参数声明地点定义的变异修饰符

我们只需要在`Presenter`类内部定义一次变异修饰符。实际上，前面的两种实现是等价的，尽管*声明地点*变异更简洁，外部类的使用更容易

# 集合变异

在 Java 中，数组是协变的。默认情况下，我们可以传递一个`String[]`数组，即使期望的是`Object[]`数组：

```kt
    public class Computer {

        public Computer() {

            String[] stringArray = new String[]{"a", "b", "c"};

            printArray(stringArray); //Pass instance of String[]

        }

        void printArray(Object[] array) { 

            //Define parameter of type Object[]

            System.out.print(array);

        }

    }
```

这种行为在 Java 的早期版本中很重要，因为它允许我们使用不同类型的数组作为参数：

```kt
    // Java

    static void print(Object[] array) {

        for (int i = 0; i <= array.length - 1; i++)

        System.out.print(array[i] + " ");

        System.out.println();

    }

    // Usage

    String[] fruits = new String[] {"Pineapple","Apple", "Orange", 

                                    "Banana"};

    print(fruits); // Prints: Pineapple Apple Orange Banana

    Arrays.sort(fruits);

    print(fruits); // Prints: Apple Banana Orange Pineapple
```

但这种行为也可能导致潜在的运行时错误：

```kt
    public class Computer {

        public Computer() {

            Number[] numberArray = new Number[]{1, 2, 3};

            updateArray(numberArray);

        }

        void updateArray(Object[] array) {

            array[0] = "abc"; 

            // Error, java.lang.ArrayStoreException: java.lang.String

        }

    }
```

函数`updateArray`接受类型为`Object[]`的参数，我们正在传递`String[]`。我们正在使用`String`参数调用 add 方法。我们可以这样做，因为数组项的类型是`Object`，所以我们可以使用`String`，这是一个新值。最后，我们想要将`String`添加到可能只包含`String`类型项的通用数组中。由于默认的协变行为，编译器无法检测到这个问题，这将导致`ArrayStoreException`异常。

相应的代码在 Kotlin 中不会编译，因为 Kotlin 编译器将此行为视为潜在的危险。这就是为什么 Kotlin 中的数组是不变的原因。因此，在需要`Array<Any>`时传递除`Array<Number>`之外的类型将导致编译时错误：

```kt
    public class Array<T> { /*...*/ }
```

因此，在需要`Array<Any>`时传递除`Array<Number>`之外的类型将导致编译时错误：

```kt
    public class Array<T> { /*...*/ }

    class Computer {

        init {

            val numberArray = arrayOf<Number>(1, 2, 3)

            updateArray(numberArray)

        }

        internal fun updateArray(array: Array<Any>) {

            array[0] = "abc" 

            //error, java.lang.ArrayStoreException: java.lang.String

        }

    }
```

请注意，只有当我们可以修改对象时才可能发生潜在的运行时异常。变异也适用于 Kotlin 集合接口。在 Kotlin 标准库中，我们有两个以不同方式定义的列表接口。Kotlin `List`接口被定义为协变，因为它是不可变的（它不包含任何允许我们更改内部状态的方法），而 Kotlin `MutableList`接口是不变的。以下是它们类型参数的定义：

```kt
    interface List<out E> : Collection<E> { /*...*/ }

    public interface MutableList<E> : List<E>, MutableCollection<E> {     

        /*...*/ 

    }
```

让我们看看这些定义在实际中的后果。它使可变列表免受协变的风险：

```kt
    fun addElement(mutableList: MutableList<Any>) {

        mutableList.add("Cat")

    }

    // Usage

    val mutableIntList = mutableListOf(1, 2, 3, 4)

    val mutableAnyList = mutableListOf<Any>(1, 'A')

    addElement(mutableIntList) // Error: Type mismatch

    addElement(mutableAnyList)
```

该列表是安全的，因为它没有用于更改其内部状态的方法，并且其协变行为允许更广泛地使用函数：

```kt
    fun printElements(list: List<Any>) {

        for(e in list) print(e)

    }

    // Usage

    val intList = listOf(1, 2, 3, 4)

    val anyList = listOf<Any>(1, 'A')

    printElements(intList) // Prints: 1234

    printElements(anyList) // Prints: 1A
```

我们可以将`List<Any>`或其任何子类型传递给`printElements`函数，因为`List`接口是协变的。我们只能将`MutableList<Any>`传递给`addElement`函数，因为`MutableList`接口是不变的。

使用`in`和`out`修饰符，我们可以操纵变异行为。我们还应该意识到变异有一些限制。让我们讨论一下。

# 变异生产者/消费者限制

通过应用变异修饰符，我们可以为类/接口的某个类型参数（声明位置变异）或*类型参数*（使用位置变异）获得协变/逆变行为。然而，我们需要注意一个限制。为了使其安全，Kotlin 编译器限制了类型参数可以使用的位置。

对于不变（类型参数上默认没有变异修饰符），我们可以在`in`（函数参数的类型）和`out`（函数返回类型）位置上使用类型参数：

```kt
    interface Stack<T> {

        fun push(t:T) // Generic type at in position

        fun pop():T // Generic type at out position

        fun swap(t:T):T // Generic type at in and out positions

        val last: T // Generic type at out position

        var special: T // Generic type at out position

    }
```

通过变异修饰符，我们只限于一个位置。这意味着我们只能将类型参数用作方法参数的类型（`in`）或方法返回值（`out`）。我们的类可以是生产者或消费者，但永远不会同时。我们可以说该类*接收参数*或*提供参数*。

让我们看看这种限制与在声明位置指定的变异修饰符的关系。以下是两个类型参数`R`和`T`的所有正确和不正确的用法：

```kt
    class ConsumerProducer<in T, out R> {

        fun consumeItemT(t: T): Unit { } // 1

        fun consumeItemR(r: R): Unit { } // 2, error

        fun produceItemT(): T { // 3, error

            // Return instance of type T

        }

        fun produceItemR(): R { // 4

            //Return instance of type R

        }

    }
```

1.  在 in 位置的 OK 类型参数`T`

1.  错误，类型参数`R`在 in 位置

1.  错误，类型参数`T`在 out 位置

1.  在 out 位置的 OK，类型参数`R`

正如我们所看到的，如果配置被禁止，编译器将报告错误。请注意，我们可以为两个类型参数`R`和`T`添加不同的修饰符。

位置限制仅适用于类外可访问（可见）的方法。这意味着不仅仅是所有`public`方法（`public`是默认修饰符），如之前所使用的，还包括标记为`protected`或`internal`的方法。当我们将方法可见性更改为`private`时，我们可以在任何位置使用我们的类型参数（`R`和`T`），就像不变的类型参数一样：

```kt
    class ConsumerProducer<in T, out R> {

        private fun consumeItemT(t: T): Unit { }

        private fun consumeItemR(r: R): Unit { }

        private fun produceItemT(): T {

            // Return instance of type T

        }

        private fun produceItemR(): R {

            //Return instance of type R

        }

    }
```

让我们看一下下表，它展示了类型参数作为类型使用时的所有允许位置：

| **可见性修饰符** | **不变性** | **协变性（out）** | **逆变性（in）** |
| --- | --- | --- | --- |
| `public`，`protected`，`internal` | in/out | out | in |
| `private` | in/out | in/out | in/out |

# 不变的构造函数

在前一节描述的`in`和`out`位置规则中，构造函数参数始终是不变的一个重要例外：

```kt
    class Producer<out T>(t: T)

    // Usage

    val stringProducer = Producer("A")

    val anyProducer: Producer<Any> = stringProducer
```

构造函数是公共的，类型参数`T`声明为`out`，但我们仍然可以在 in 位置使用它作为构造函数参数类型。原因是构造函数方法在实例创建后不能被调用，因此始终可以安全地调用它。

正如我们在第四章中讨论的那样，*类和对象*，我们还可以使用`val`或`var`修饰符直接在类构造函数中定义属性。当指定协变时，我们只能在构造函数中定义具有协变类型的只读属性（`val`）。这是安全的，因为只会生成 getter，因此在类实例化后，此属性的值不会改变：

```kt
    class Producer<out T>(val t: T) // Ok, safe
```

使用`var`时，编译器会生成 getter 和 setter，因此属性值可能在某个时刻发生变化。这就是为什么我们不能在构造函数中声明一个协变类型的读写（`var`）属性的原因：

```kt
    class Producer<out T>(var t: T) // Error, not safe
```

我们已经说过，变异限制只适用于外部客户端，因此我们仍然可以通过添加私有可见性修饰符来定义一个协变读写属性：

```kt
    class Producer<out T>(private var t:T)
```

另一个来自 Java 的流行的泛型类型限制与类型擦除有关。

# 类型擦除

类型擦除是引入到 JVM 中的，以使 JVM 字节码向后兼容于引入泛型之前的版本。在 Android 平台上，Kotlin 和 Java 都被编译成 JVM 字节码，因此它们都容易受到*类型擦除*的影响。

类型擦除是从泛型类型中移除*类型参数*的过程，因此泛型类型在运行时失去了一些类型信息（*类型参数*）：

```kt
    package test

    class Box<T>

    val intBox = Box<Int>()

    val stringBox = Box<String>()

    println(intBox.javaClass) // prints: test.Box

    println(stringBox.javaClass) // prints: test.Box
```

编译器可以区分两种类型并保证类型安全。然而，在编译过程中，编译器将参数化类型`Box<Int>`和`Box<String>`转换为`Box`（原始类型）。生成的 Java 字节码不包含任何与*类型参数*相关的信息，因此我们无法在运行时区分泛型类型。

类型擦除导致了一些问题。在 JVM 中，我们不能声明具有相同 JVM 签名的同一方法的两个重载：

```kt
    /*

    java.lang.ClassFormatError: Duplicate method name&signature...

    */

    fun sum(ints: List<Int>) {

        println("Ints")

    }

    fun sum(strings: List<String>) {

        println("Ints")

    }
```

当*类型参数*被移除时，这两个方法将具有完全相同的声明：

```kt
    /*

    java.lang.ClassFormatError: Duplicate method name&signature...

    */

    fun sum(ints: List) {

        println("Ints")

    }

    fun sum(strings: List) {

        println("Ints")

    }
```

我们还可以通过更改生成函数的 JVM 名称来解决此问题。我们可以使用`JvmName`注解在将代码编译为 JVM 字节码时更改其中一个方法的名称：

```kt
    @JvmName("intSum") fun sum(ints: List<Int>) {

        println("Ints")

    }

    fun sum(strings: List<String>) {

        println("Ints")

    }
```

从 Kotlin 中使用此函数的用法没有改变，但由于我们更改了第一个函数的 JVM 名称，因此我们需要使用新名称从 Java 中使用它：

```kt
    // Java

    TestKt.intSum(listOfInts);
```

有时我们希望在运行时保留*类型参数*，这就是`reified` *类型参数*非常方便的地方。

# 具体化类型参数

有些情况下，在运行时访问类型参数会很有用，但由于类型擦除，这是不允许的：

```kt
    fun <T> typeCheck(s: Any) {

        if(s is T){ 

        // Error: cannot check for instance of erased type: T

            println("The same types")

        } else {

            println("Different types")

        }

    }
```

为了能够克服 JVM 的限制，Kotlin 允许我们使用特殊的修饰符来在运行时保留*类型参数*。我们需要使用 reified 修饰符标记类型参数：

```kt
    interface View

    class ProfileView: View

    class HomeView: View

    inline fun <reified T> typeCheck(s: Any) { // 1

        if(s is T){

            println("The same types")

        } else {

        println("Different types")

        }

    }

    // Usage

    typeCheck<ProfileView>(ProfileView()) // Prints: The same types

    typeCheck<HomeView>(ProfileView()) // Prints: Different types

    typeCheck<View>(ProfileView()) // Prints: The same types
```

1.  类型参数标记为精炼，函数标记为`inline`。

现在我们可以安全地在运行时访问*类型参数*类型。具体化类型参数仅适用于内联函数，因为在编译过程中（内联），Kotlin 编译器会替换具体化*类型参数*的实际类。这样，*类型参数*就不会被类型擦除移除。

我们还可以对具体化类型使用反射来检索有关类型的更多信息：

```kt
    inline fun <reified T> isOpen(): Boolean {

        return T::class.isOpen

    }
```

在 JVM 字节码级别，具体类型或原始类型的包装类型表示具体类型参数的出现。这就是为什么具体类型参数不受类型擦除影响的原因。

使用具体类型参数允许我们以全新的方式编写方法。在 Java 中启动新的`Activity`，我们需要这样的代码：

```kt
    //Java

    startActivity(Intent(this, ProductActivity::class.java))
```

在 Kotlin 中，我们可以定义`startActivity`方法，这将使我们以更简单的方式导航到`Activity`：

```kt
    inline fun <reified T : Activity> startActivity(context: Context) {

        context.startActivity(Intent(context, T::class.java))

    }

    // Usage

    startActivity<UserDetailsActivity>(context)
```

我们定义了`startActivity`方法，并通过使用*类型参数*传递了关于我们要启动的`Activity`（`ProductActivity`）的信息。我们还定义了一个显式的具体类型参数边界，以确保我们只能使用`Activity`（及其子类）作为*类型参数*。

# startActivity 方法

为了正确使用`startActivity`方法，我们需要一种方法来将参数传递给正在启动的`Activity`（`Bundle`）。可以更新前面的实现以支持这样的参数：

```kt
    startActivity<ProductActivity>("id" to 123, "extended" to true)
```

在前面的示例中，使用键和值填充参数（由内联*to*函数定义）。但是，此函数的实现超出了本书的范围。但是，我们可以使用现有的实现。**Anko**库（[`github.com/Kotlin/anko`](https://github.com/Kotlin/anko)）已经实现了具有所有必需功能的`startActivity`方法。我们只需要导入`Appcompat-v7-commons`依赖项。

```kt
    compile "org.jetbrains.anko:anko-appcompat-v7-commons:$anko_version"
```

Anko 为`Context`和`Fragment`类定义了扩展，因此我们可以在任何`Activity`或`Fragment`中使用此方法，就像在类中定义的任何其他方法一样，而无需在类中定义该方法。我们将在第七章中讨论扩展，*扩展函数和属性*。

请注意，具体类型参数有一个主要限制：我们无法从具体类型参数创建类的实例（不使用反射）。其背后的原因是构造函数总是只与具体实例相关联（它永远不会被继承），因此没有构造函数可以安全地用于所有可能的类型参数。

# 星投影

由于类型擦除，运行时只能获得不完整的类型信息。例如，泛型类型的类型参数是不可用的：

```kt
    val list = listOf(1,2,3)

    println(list.javaClass) // Prints: class java.util.Arrays$ArrayList
```

这导致了一些问题。我们无法执行任何检查来验证`List`包含哪些类型的元素：

```kt
    /*

    Compile time error: cannot check instance of erased type: 

    List<String>

    */

    if(collection is List<Int>) {

        //...

    }
```

问题出在运行时执行了一个检查，其中关于*类型参数*的信息是不可用的。然而，与 Java 相反，Kotlin 不允许我们声明原始类型（未使用*类型参数*的泛型类型）：

```kt
    SimpleList<> // Java: ok

    SimpleList<> // Kotlin: error
```

Kotlin 允许我们使用*星投影*语法，这基本上是一种表明*类型参数*信息缺失或不重要的方式：

```kt
    if(collection is List<*>) {

        //...

    }
```

通过使用星投影语法，我们说`Box`存储特定类型的参数：

```kt
    class Box<T>

    val anyBox = Box<Any>()

    val intBox = Box<Int>()

    val stringBox = Box<String>()

    var unknownBox: Box<*>

    unknownBox = anyBox // Ok

    unknownBox = intBox // Ok

    unknownBox = stringBox // Ok
```

请注意`Box<*>`和`Box<Any>`之间存在差异。如果我们想定义包含任何项的列表，我们将使用`Box<Any>`。但是，如果我们想定义包含特定类型项的列表，但是这种类型是未知的（可能是`Any`，`Int`，`String`等等。但是我们没有关于这种类型的信息），而`Box<Any>`表示列表包含`Any`类型的项。我们将使用`Box<*>`：

```kt
    val anyBox: Box<Any> = Box<Int> // Error: Type mismatch
```

如果泛型类型定义了多个类型参数，我们需要为每个缺失的*类型参数*使用星（`*`）：

```kt
    class Container<T, T2>

    val container: Container<*, *>
```

星投影在我们想对类型执行操作，但是*类型参数*信息不重要时也很有帮助：

```kt
    fun printSize(list: MutableList<*>) {

        println(list.size)

    }

    //usage

    val stringList = mutableListOf("5", "a", "2", "d")

    val intList = mutableListOf(3, 7)

    printSize(stringList) // prints: 4

    printSize(intList) // prints: 2
```

在前面的示例中，不需要关于*类型参数*的信息来确定集合大小。使用星投影语法减少了对变异修饰符的需求，只要我们不使用依赖于*类型参数*的任何方法。

# 类型参数命名约定

官方的 Java 类型参数命名约定（[`docs.oracle.com/javase/tutorial/java/generics/types.html`](https://docs.oracle.com/javase/tutorial/java/generics/types.html)）为参数命名定义了以下准则：

*按照惯例，类型参数名称为单个大写字母。这与您已经了解的变量命名约定形成鲜明对比，这是有充分理由的。如果没有这个约定，很难区分类型变量和普通类或接口名称。最常用的类型参数名称是：*

+   *E: 元素（Java 集合框架广泛使用）*

+   *K: 键*

+   *N: 数字*

+   *T: 类型*

+   *V: 值*

+   *S,U,V 等：第 2、第 3、第 4 个类型*

Kotlin 标准库中的许多类遵循这种约定。对于常见类（`List`，`Mat`，`Set`等）或定义简单类型参数的类（`Box<T>`类）来说，这种方式很好用。然而，对于自定义类和多个类型参数，我们很快意识到单个字母包含的信息量不足，有时很难快速判断类型参数代表的数据类型。对于这个问题有一些解决方案。

我们可以确保泛型得到适当的文档记录，是的，这肯定会有所帮助，但我们仍然无法仅通过查看代码来确定类型参数的含义。文档很重要，但我们应该将文档视为辅助信息源，并努力实现最高可能的代码可读性。

多年来，程序员已经开始转向更有意义的命名惯例。**Google Java Style Guide**（[`google.github.io/styleguide/javaguide.html#s5.2.8-type-variable-names`](https://google.github.io/styleguide/javaguide.html#s5.2.8-type-variable-names)）简要描述了官方 Java 类型参数命名约定和自定义命名约定的混合。他们提倡两种不同的风格。第一种是使用单个大写字母，可选地后跟单个数字（与 Java 描述的`S`，`U`，`V`名称相反）：

```kt
    class Box<T, T2>
```

第二种风格更具描述性，因为它为类型参数添加了有意义的前缀：

```kt
    class Box<RequestT>
```

不幸的是，类型参数名称没有单一的标准。最常见的解决方案是使用单个大写字母。这些都是简化的例子，但请记住，类通常在多个地方使用泛型，因此适当的命名将提高代码的可读性。

# 总结

在本章中，我们了解了泛型存在的原因，并讨论了定义泛型类和接口以及声明泛型类型的各种方式。我们知道如何通过使用使用点和声明点变异修饰符来处理子类型关系。我们学会了如何处理类型擦除，以及如何使用具体化的类型参数在运行时保留泛型类型。

在下一章中，我们将讨论 Kotlin 最令人兴奋的功能之一-扩展。这个功能允许我们为现有类添加新的行为。我们将学习如何为任何给定的类实现新的方法和属性，包括 Android 框架和第三方库中的最终类。
