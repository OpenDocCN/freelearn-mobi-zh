# 第二章：奠定基础

本章主要讨论了构成 Kotlin 编程语言核心元素的基本构建块。每个构建块本身可能看起来微不足道，但是当它们组合在一起时，它们会创建非常强大的语言结构。我们将讨论引入了严格的空安全和智能转换的 Kotlin 类型系统。此外，我们还将看到 JVM 世界中一些新的操作符，以及与 Java 相比的许多改进。我们还将介绍处理应用程序流程和以统一方式处理相等性的新方法。

在本章中，我们将涵盖以下主题：

+   变量、值和常量

+   类型推断

+   严格的空安全

+   智能转换

+   Kotlin 数据类型

+   控制结构

+   异常处理

# 变量

在 Kotlin 中，我们有两种类型的变量：`var`或`val`。第一种`var`是可变引用（读写），可以在初始化后更新。`var`关键字用于定义 Kotlin 中的变量。它相当于普通（非 final）的 Java 变量。如果我们的变量需要在某个时候更改，我们应该使用`var`关键字进行声明。让我们看一个变量声明的例子：

```kt
    fun main(args: Array<String>) { 
        var fruit:String =  "orange" //1 
        fruit  = "banana" //2 
    } 
```

1.  创建水果变量，并用变量`orange`的值进行初始化

1.  重新初始化水果变量为`banana`的值

第二种类型的变量是只读引用。这种类型的变量在初始化后不能被重新分配。

`val`关键字可以包含自定义的 getter，因此在技术上它可以在每次访问时返回不同的对象。换句话说，我们无法保证对底层对象的引用是不可变的：

`val random: Int`

`get() = Random().nextInt()`

自定义 getter 将在第四章“类和对象”中更详细地讨论。

`val`关键字相当于带有`final`修饰符的 Java 变量。使用不可变变量很有用，因为它确保变量永远不会被错误地更新。*不可变性*的概念对于在不担心正确的数据同步的情况下处理多个线程也是有帮助的。要声明不可变变量，我们将使用`val`关键字：

```kt
    fun main(args: Array<String>) { 
        val fruit:String= "orange"//1 
        a = "banana" //2  Error 
    } 
```

1.  创建水果变量，并用字符串`orange`的值进行初始化

1.  编译器会抛出错误，因为水果变量已经被初始化

Kotlin 还允许我们在文件级别定义变量和函数。我们将在第三章“玩转函数”中进一步讨论。

请注意，变量引用的类型（`var`，`val`）与引用本身有关，而不是所引用对象的属性。这意味着当使用只读引用（`val`）时，我们将无法更改指向特定对象实例的引用（我们将无法重新分配变量值），但我们仍然可以修改所引用对象的属性。让我们使用数组来看看它的运行情况：

```kt
    val list = mutableListOf("a","b","c") //1 
    list = mutableListOf("d", "e") //2 Error 
    list.remove("a") //3 
```

1.  初始化可变列表

1.  编译器会抛出错误，因为值引用不能被改变（重新分配）

1.  编译器将允许修改列表的内容

关键字`val`不能保证底层对象是不可变的。

如果我们真的想确保对象不会被修改，我们必须使用不可变引用和不可变对象。幸运的是，Kotlin 的标准库包含了任何集合接口（`List`与`MutableList`，`Map`与`MutableMap`等）的不可变等价物，对于用于创建特定集合实例的辅助函数也是如此：

| **变量/值定义** | **引用可以改变** | **对象状态可以改变** |
| --- | --- | --- |
| `val = listOf(1,2,3)` | 否 | 否 |
| `val = mutableListOf(1,2,3)` | 否 | 是 |
| `var = listOf(1,2,3)` | 是 | 否 |
| - `var = mutableListOf(1,2,3)` | 是 | 是 |

# 类型推断

正如我们在之前的例子中看到的，与 Java 不同，Kotlin 类型是在变量名之后定义的：

```kt
    var title: String 
```

乍一看，这可能看起来对 Java 开发人员来说很奇怪，但这个构造是 Kotlin 一个非常重要的特性的基础，称为**类型推断**。类型推断意味着编译器可以从上下文（分配给变量的表达式的值）中推断类型。当变量声明和初始化一起进行（单行）时，我们可以省略类型声明。让我们看一下以下变量定义：

```kt
    var title: String = "Kotlin" 
```

`title`变量的类型是`String`，但我们真的需要隐式类型声明来确定变量类型吗？在表达式的右侧，我们有一个字符串`Kotlin`，我们将其赋给了左侧表达式中定义的变量`title`。

我们指定了变量类型为`String`，但这是显而易见的，因为这与分配表达式的类型（`Kotlin`）的类型相同。幸运的是，这个事实对于 Kotlin 编译器来说也是显而易见的，所以当声明变量时我们可以省略类型，因为编译器将尝试从当前上下文中确定变量的最佳类型：

```kt
    var title = "Kotlin" 
```

请记住，类型声明被省略，但变量的类型仍然隐式设置为`String`，因为 Kotlin 是一种强类型语言。这就是为什么前面两个声明是相同的，Kotlin 编译器仍然能够正确验证变量的所有未来用法。这里有一个例子：

```kt
    var title = "Kotlin" 
    title = 12 // 1, Error 
```

1.  推断类型为`String`，我们正在尝试分配`Int`

如果我们想要将`Int`（值`12`）赋给标题变量，那么我们需要指定标题类型为`String`和`Int`的公共类型。在类型层次结构中最接近的一个是`Any`：

```kt
    var title: Any = "Kotlin" 
    title = 12
```

Any 是 Java 对象类型的等价物。它是 Kotlin 类型层次结构的根。Kotlin 中的所有类都明确继承自类型`Any`，甚至是`String`或`Int`等原始类型

![](img/Image00021.jpg)Any 定义了三个方法：`equals`，`toString`和`hashCode`。Kotlin 标准库包含了这种类型的一些扩展。我们将在第七章 *扩展函数和属性*中讨论扩展。

正如我们所看到的，*类型推断* 不仅限于原始值。让我们直接从函数中推断类型：

```kt
    var total = sum(10, 20) 
```

在前面的例子中，推断的类型将与函数返回的类型相同。我们可能猜测它将是`Int`，但它也可能是`Double`，`Float`或其他类型。如果从上下文中不明显推断出类型，我们可以在变量名上放置插入符号，并运行 Android Studio 的表达式类型命令（对于 Windows，是*Shift* + *Ctrl* + *P*，对于 macOS，是箭头键+ *control* + *P*）。这将在工具提示中显示变量类型，如下所示：

![](img/Image00022.gif)

类型推断也适用于泛型类型：

```kt
    var persons = listOf(personInstance1, personInstance2) 

    // Inferred type: List<Person> () 
```

假设我们只传递`Person`类的实例，推断的类型将是`List<Person>`。`listOf`方法是 Kotlin 标准库中定义的一个辅助函数，允许我们创建集合。我们将在第七章 *扩展函数和属性*中讨论这个主题。让我们看一些更高级的例子，使用 Kotlin 标准库类型`Pair`，其中包含由两个值组成的一对：

```kt
    var pair = "Everest" to 8848 // Inferred type: Pair<String, Int> 
```

在前面的例子中，使用*中缀函数*创建了一个`pair`实例，这将在第四章 *类和对象*中讨论，但现在我们只需要知道这两个声明返回相同类型的`Pair`对象：

```kt
    var pair = "Everest" to 8848   

    // Create pair using to infix method 
    var pair2 = Pair("Everest", 8848) 

    // Create Pair using constructor 
```

类型推断也适用于更复杂的情况，例如从推断类型中推断类型。让我们使用 Kotlin 标准库的`mapOf`函数，并将`Pair`类的`to`方法中缀到定义`map`。对中的第一项将用于推断`map`键类型；第二项将用于推断值类型：

```kt
    var map = mapOf("Mount Everest" to 8848, "K2" to 4017) 
    // Inferred type: Map<String, Int> 
```

`Map<String, Int>`的泛型类型是从传递给`Pair`构造函数的参数的类型推断出来的。我们可能会想知道，如果用于创建`map`的推断类型的对不同会发生什么？第一对是`Pair<String, Int>`，第二对是`Pair<String, String>`：

```kt
    var map = mapOf("Mount Everest" to 8848, "K2" to "4017") 
    // Inferred type: Map<String, Any> 
```

在前面的情景中，Kotlin 编译器将尝试为所有对推断出一个公共类型。两对中的第一个参数都是`String`（`Mount Everest`，`K2`），因此在这里自然会推断出`String`。每对的第二个参数不同（第一对为`Int`，第二对为`String`），因此 Kotlin 需要找到最接近的公共类型。选择`Any`类型，因为这是上游类型层次结构中最接近的公共类型：

![](img/Image00023.jpg)

正如我们所看到的，类型推断在大多数情况下都做得很好，但是我们仍然可以选择显式定义数据类型，例如，我们想要不同的变量类型：

```kt
    var age: Int = 18
```

在处理整数时，`Int`类型总是默认选择，但是我们仍然可以显式定义不同的类型，例如，`Short`，以节省一些宝贵的 Android 内存：

```kt
    var age: Short = 18
```

另一方面，如果我们需要存储更大的值，我们可以将`age`变量的类型定义为`Long`。我们可以像以前一样使用显式类型声明，也可以使用*字面常量*：

```kt
    var age: Long = 18 // Explicitly define variable type
    var age = 18L      

    // Use literal constant to specify value type
```

这两个声明是相等的，它们都将创建类型为`Long`的变量。

目前，我们知道代码中有更多情况可以省略类型声明，以使代码语法更简洁。然而，有些情况下，由于上下文中缺乏信息，Kotlin 编译器将无法推断类型。例如，简单的声明而没有赋值将使类型推断变得不可能：

```kt
    val title // Error 
```

在前面的例子中，变量将在稍后初始化，因此无法确定其类型。这就是为什么类型必须明确指定的原因。一般规则是，如果编译器知道表达式的类型，则可以推断类型。否则，必须明确指定。Android Studio 中的 Kotlin 插件做得很好，因为它确切地知道类型无法推断的地方，然后会突出显示错误。这使我们能够在编写代码时立即通过 IDE 显示正确的错误消息，而无需完成应用程序。

# 严格的空安全

根据**敏捷软件评估**（* [`p3.snf.ch/Project-144126`](http://p3.snf.ch/Project-144126) *）研究，缺少空检查是 Java 系统中最常见的错误模式。Java 中错误的最大来源是`NullPointerExceptions`。它如此之大，以至于 2009 年在一次会议上，Tony Hoare 爵士为发明空引用而道歉，称其为*十亿美元的错误*（[`en.wikipedia.org/wiki/Tony_Hoare`](https://en.wikipedia.org/wiki/Tony_Hoare)）。

为了避免`NullPointerException`，我们需要编写防御性代码，在使用对象之前检查它是否为 null。许多现代编程语言，包括 Kotlin，都采取了将运行时错误转换为编译时错误的步骤，以提高编程语言的安全性。在 Kotlin 中实现这一点的一种方法是向语言类型系统添加*空安全机制*。这是可能的，因为 Kotlin 类型系统区分可以保存 null（可空引用）和不可以保存 null（非空引用）的引用。Kotlin 的这一单一特性使我们能够在开发的早期阶段检测到许多与`NullPointerException`相关的错误。编译器和 IDE 将阻止许多`NullPointerException`。在许多情况下，编译将失败，而不是应用程序在运行时失败。

严格的空安全是 Kotlin 类型系统的一部分。默认情况下，常规类型不能为 null（不能存储 null 引用），除非明确允许。要存储 null 引用，我们必须将变量标记为可空（允许存储 null 引用），方法是在变量类型声明中添加问号后缀。这是一个例子：

```kt
    val age: Int = null //1, Error 
    val name: String? = null //2 
```

1.  编译器会抛出错误，因为这种类型不允许为 null。

1.  编译器将允许空赋值，因为类型使用问号后缀标记为可空。

我们不允许在可能为 null 的对象上调用方法，除非在调用之前执行了空值检查：

```kt
    val name: String? = null 
    // ... 
    name.toUpperCase() // error, this reference may be null 
```

我们将在下一节学习如何处理这个问题。Kotlin 中的每种非空类型都有其对应的可空类型：`Int` 对应 `Int?`，`String` 对应 `String?` 等等。相同的规则也适用于 Android 框架中的所有类（`View` 对应 `View?`），第三方库（`OkHttpClient` 对应 `OkHttpClient?`），以及开发人员定义的所有自定义类（`MyCustomClass` 对应 `MyCustomClass?`）。这意味着每个非泛型类都可以用来定义两种类型，可空和非空。非空类型也是其可空对应类型的子类型。例如，`Vehicle` 除了是 `Vehicle?` 的子类型外，也是 `Any` 的子类型：

![](img/Image00024.jpg)

`Nothing` 类型是一个空类型（无法实例化的类型），它不能有实例。我们将在 第三章 *Playing with Functions* 中详细讨论它。这种类型层次结构是为什么我们可以将非空对象（`Vehicle`）赋值给可空类型的变量（`Vehicle?`），但不能将可空对象（`Vehicle?`）赋值给非空变量（`Vehicle`）的原因：

```kt
    var nullableVehicle: Vehicle?  
    var vehicle: Vehicle 

    nullableVehicle = vehicle // 1 
    vehicle = nullableVehicle // 2, Error 

```

1.  可以进行赋值

1.  错误，因为 `nullableVehicle` 可能为 null

我们将在接下来的章节中讨论处理可空类型的方法。现在让我们回到类型定义。在定义 *泛型类型* 时，有多种定义可空性的可能性，因此让我们通过比较包含类型为 `Int` 的泛型 `ArrayList` 的不同声明来检查各种集合类型。下面是一个表格，展示了关键的区别：

| **类型声明** | **列表本身可以为 null** | **元素可以为 null** |
| --- | --- | --- |
| `ArrayList<Int>` | 否 | 否 |
| `ArrayList<Int>?` | 是 | 否 |
| `ArrayList<Int?>` | 否 | 是 |
| `ArrayList<Int?>?` | 是 | 是 |

理解不同的空类型声明方式很重要，因为 Kotlin 编译器强制执行它以避免 `NullPointerException`。这意味着编译器在访问任何可能为 null 的引用之前强制执行空值检查。现在让我们来看看 `Activity` 类的 `onCreate` 方法中常见的 Android/Java 错误：

```kt
    //Java 
    @Override 
    public void onCreate(Bundle savedInstanceState) { 
        super.onCreate(savedInstanceState); 
        savedInstanceState.getBoolean("locked"); 
    } 
```

在 Java 中，这段代码将编译正常，访问 null 对象将导致应用在运行时崩溃并抛出 `NullPointerException`。现在让我们来看看相同方法的 Kotlin 版本：

```kt
    override fun onCreate(savedInstanceState: Bundle?) { //1 
         super.onCreate(savedInstanceState) 
         savedInstanceState.getBoolean("key") //2 Error 
    } 
```

1.  `savedInstanceState` 定义为可空的 `Bundle?`

1.  编译器会抛出错误

`savedInstanceState` 类型是一个平台类型，Kotlin 可以将其解释为可空或非空。我们将在接下来的章节中讨论平台类型，但现在我们将定义 `savedInstanceState` 为可空类型。我们这样做是因为我们知道在创建 Activity 时会传递 `null`。只有在使用保存的实例状态重新创建 Activity 时才会传递 `Bundle` 的实例：

我们将在 第三章 *Playing with Functions* 中讨论函数，但现在我们已经可以看到，在 Kotlin 中声明函数的语法与 Java 非常相似。

在 Kotlin 中修复上述错误的最明显的方法是以与 Java 相同的方式检查空值：

```kt
    override fun onCreate(savedInstanceState: Bundle?) { 
        super.onCreate(savedInstanceState)
    } 

    override fun onCreate(savedInstanceState: Bundle?) { 
        super.onCreate(savedInstanceState) 

        val locked: Boolean 
        if(savedInstanceState != null) 
            locked = savedInstanceState.getBoolean("locked")  
        else 
            locked = false 
    } 
```

前面的结构呈现了一些样板代码，因为在 Java 开发中进行空检查是一种非常常见的操作（特别是在 Android 框架中，其中大多数元素都是可空的）。幸运的是，Kotlin 允许使用一些更简单的解决方案来处理可空变量。第一个是*安全调用*运算符。

# 安全调用

安全调用运算符只是一个问号后跟一个点。重要的是要理解，安全转换运算符将始终返回一个值。如果运算符的左侧为 null，则它将返回 null，否则将返回右侧表达式的结果：

```kt
    override fun onCreate(savedInstanceState: Bundle?) { 
         super.onCreate(savedInstanceState) 
         val locked: Boolean? = savedInstanceState?.getBoolean("locked") 
    } 
```

如果`savedInstanceState`是`null`，那么将返回`null`，否则将返回评估`savedInstanceState?.getBoolean("locked")`表达式的结果。请记住，可空引用调用可能始终返回可空值，因此整个表达式的结果是可空`Boolean*?*`。如果我们想确保获得非空布尔值，我们可以结合*安全调用*运算符和*Elvis*运算符，在下一节中讨论。

可以将*安全调用*运算符的多个调用链接在一起，以避免嵌套的`if`表达式或像这样的复杂条件：

```kt
    //Java idiomatic - multiple checks
    val quiz: Quiz = Quiz()
    //...
    val correct: Boolean?

    if(quiz.currentQuestion != null) {
        if(quiz.currentQuestion.answer != null ) {
            //do something
        }
    }

    //Kotlin idiomatic - multiple calls of save call operator 
    val quiz: Quiz = Quiz() 

    //... 

    val correct = quiz.currentQuestion?.answer?.correct  

    // Inferred type Boolean? 
```

前面的链的工作方式如下--只有在`answer`值不为 null 时才会访问`correct`，只有在`currentQuestion`值不为 null 时才会访问`answer`。结果，该表达式将返回`correct`属性返回的值，或者如果*安全调用*链中的任何对象为 null，则返回 null。

# Elvis 运算符

Elvis 运算符由问号后跟冒号（`?:`）表示，并具有以下语法：

```kt
    first operand ?: second operand 
```

Elvis 运算符的工作方式如下：如果`第一个操作数`不为 null，则将返回该操作数，否则将返回`第二个操作数`。Elvis 运算符允许我们编写非常简洁的代码。

我们可以将 Elvis 运算符应用于我们的示例，以检索变量`locked`，这将始终是非空的：

```kt
    override fun onCreate(savedInstanceState: Bundle?) { 
        super.onCreate(savedInstanceState) 

        val locked: Boolean = savedInstanceState?.

                    getBoolean("locked") ?: false 
    } 
```

在前面的示例中，如果`savedInstanceState`不为`null`，Elvis 运算符将返回`savedInstanceState?.getBoolean("locked")`表达式的值，否则将返回 false。这样我们可以确保`locked`变量。借助 Elvis 运算符，我们可以定义默认值。还要注意，只有在左侧为 null 时才会评估右侧表达式。然后提供将在表达式为可空时使用的默认值。回到上一节的测验示例，我们可以轻松修改代码以始终返回非空值：

```kt
    val correct = quiz.currentQuestion?.answer?.correct ?: false 
```

结果是，该表达式将返回`correct`属性返回的值，或者如果*安全调用*链中的任何对象为 null，则返回`false`。这意味着该值将始终被返回，因此推断出非空布尔类型。

该运算符的名称来自著名的美国歌手兼词曲作家埃尔维斯·普雷斯利，因为他的发型类似于问号。

![](img/Image00025.jpg)

# 非空断言

处理空值的另一个工具是*非空断言*运算符。它由双感叹号（`!!`）表示。该运算符将可空变量显式转换为非空变量。以下是一个使用示例：

```kt
    var y: String? = "foo" 
    var size: Int = y!!.length 
```

通常，我们无法将可空属性`length`的值分配给非空变量`size`。但是，作为开发人员，我们可以向编译器保证，这个可空变量在这里将有一个值。如果我们是对的，我们的应用程序将正常工作，但如果我们错了，并且变量具有空值，应用程序将抛出`NullPointerException`。让我们检查一下我们的活动方法`onCreate()`：

```kt
override fun onCreate(savedInstanceState: Bundle?) { 
    super.onCreate(savedInstanceState) 
    val locked: Boolean = savedInstanceState!!.getBoolean("locked")  
} 
```

前面的代码将编译，但这段代码会正确工作吗？正如我们之前所说的，当恢复活动实例时，`savedInstanceState`将被传递给`onCreate`方法，因此这段代码将在没有异常的情况下工作。然而，在创建活动实例时，`savedInstanceState`将为空（没有以前的实例可以恢复），因此将在运行时抛出`NullPointerException`。这种行为类似于 Java，但主要区别在于在 Java 中，访问潜在可空对象而不进行空值检查是默认行为，而在 Kotlin 中，我们必须强制进行；否则，我们将得到编译错误。

这个运算符只有少数正确的可能应用场景，所以当你使用它或在代码中看到它时，要考虑它可能存在的危险或警告。建议很少使用非空断言，并且在大多数情况下应该用安全调用或智能转换来替代。

对抗非空断言文章提供了一些有用的例子，其中非空断言运算符被其他安全的 Kotlin 构造替换，网址为[`bit.ly/2xg5JXt`](http://bit.ly/2xg5JXt)。

实际上，在这种情况下，使用非空断言运算符是没有意义的，因为我们可以使用`let`以更安全的方式解决我们的问题。

# 让

处理可空变量的另一个工具是`let`。这实际上不是运算符，也不是语言的特殊构造。它是在 Kotlin 标准库中定义的一个函数。让我们看看`let`与*安全调用*运算符结合的语法：

```kt
    override fun onCreate(savedInstanceState: Bundle?) { 
        super.onCreate(savedInstanceState) 

        savedInstanceState?.let{ 
            println(it.getBoolean("isLocked")) // 1 
        } 
    } 
```

1.  `let`中的`savedInstanceState`可以使用变量访问

命名它。

如前所述，安全调用运算符的右侧表达式只有在左侧不为空时才会被评估。在这种情况下，右侧是一个接受另一个函数（lambda）作为参数的`let`函数。如果`savedInstanceState`不为空，则将执行`let`后面的代码块。我们将在第七章中学习更多关于它以及如何定义这样的函数，*扩展函数和属性*。

# 空值和 Java

我们知道 Kotlin 要求明确定义可以保存空值的引用。另一方面，Java 对空值的处理更为宽松，因此我们可能会想知道 Kotlin 如何处理来自 Java 的类型（基本上是整个 Android SDK 和用 Java 编写的库）。在可能的情况下，Kotlin 编译器将从代码中确定类型的空值，并使用空值注解将类型表示为实际的可空或非可空类型。

Kotlin 编译器支持多种空值注解，包括：

+   Android (`com.android.annotations`和`android.support.annotations`)

+   JetBrains (`@Nullable`和`@NotNull`来自`org.jetbrains.annotations`包)

+   JSR-305 (`Javax.annotation` )

我们可以在 Kotlin 编译器源代码中找到完整的列表([`github.com/JetBrains/kotlin/blob/master/core/descriptor.loader.Java/src/org/jetbrains/kotlin/load/Java/JvmAnnotationNames.kt`](https://github.com/JetBrains/kotlin/blob/master/core/descriptor.loader.Java/src/org/jetbrains/kotlin/load/Java/JvmAnnotationNames.kt))

我们之前在 Activity 的`onCreate`方法中看到过这一点，其中`savedInstanceState`的类型被明确设置为可空类型`Bundle?`：

```kt
    override fun onCreate(savedInstanceState: Bundle?) { 
        ...

    } 
```

然而，有许多情况下无法确定变量的空值。所有来自 Java 的变量都可以为空，除非标记为非可空。我们可以将它们全部视为可空，并在每次访问之前进行检查，但这是不切实际的。作为这个问题的解决方案，Kotlin 引入了*平台类型*的概念。这些是来自 Java 类型的类型，具有放松的空值检查，这意味着每个平台类型可能是空或非空。

虽然我们不能自己声明平台类型，但这种特殊的语法存在是因为编译器和 Android Studio 有时需要显示它们。我们可以在异常消息或方法参数列表中发现平台类型。平台类型语法只是在变量类型声明中的一个感叹号后缀：

```kt
View! // View defined as platform type
```

我们可以将每种平台类型视为可空的，但类型的可空性通常取决于上下文，因此有时我们可以将它们视为不可空的变量。这个伪代码展示了平台类型的可能含义：

```kt
    T! = T or T? 
```

作为开发者，我们有责任决定如何处理这种类型，是可空的还是非可空的。让我们考虑一下`findViewById`方法的用法：

```kt
    val textView = findViewById(R.id.textView)  
```

`findViewById`方法实际上会返回什么？`textView`变量的推断类型是什么？可空类型（`TestView`）还是非可空类型（`TextView?`）？默认情况下，Kotlin 编译器对`findViewById`方法返回的值的可空性一无所知。这就是为什么`TextView`的推断类型具有平台类型`View!`。

这就是我们所说的开发者责任。作为开发者，我们必须决定，因为只有我们知道布局是否在所有配置（纵向、横向等）中定义了`textView`，还是只在其中一些配置中定义了。如果我们在当前布局中定义了适当的视图，`findViewById`方法将返回对该视图的引用，否则将返回 null：

```kt
    val textView = findViewById(R.id.textView) as TextView // 1 
    val textView = findViewById(R.id.textView) as TextView? // 2 
```

1.  假设`textView`在每个配置的每个布局中都存在，因此`textView`可以定义为非可空的。

1.  假设`textView`在所有布局配置中都不存在（例如，只在横向布局中存在），则`textView`必须定义为可空的，否则当加载没有`textView`的布局时，应用程序将抛出`NullPointerException`。

# 转换

转换概念得到了许多编程语言的支持。基本上，转换是将一个特定类型的对象转换为另一种类型的方法。在 Java 中，我们需要在访问其成员之前显式地转换对象，或者将其转换并将其存储在转换后的类型的变量中。Kotlin 简化了转换的概念，并通过引入*智能转换*将其提升到了下一个级别。

在 Kotlin 中，我们可以执行几种类型的转换：

+   显式地将对象转换为不同类型（*安全转换*操作符）

+   将对象隐式地转换为不同类型或可空类型转换为非可空类型（*智能转换*机制）

# 安全/不安全转换操作符

在 Java 或 Kotlin 等强类型语言中，我们需要使用转换操作符将一个类型的值显式地转换为另一个类型。典型的转换操作是将一个特定类型的对象转换为其超类型（向上转换）、子类型（向下转换）或接口的另一个对象类型。让我们从 Java 中可以执行的转换的小提示开始：

```kt
    Fragment fragment = new ProductFragment(); 
    ProductFragment productFragment = (ProductFragment) fragment; 
```

在上面的例子中，有一个`ProductFragment`的实例被分配给存储`Fragment`数据类型的变量。为了能够将这些数据存储到只能存储`ProductFragment`数据类型的`productFragment`变量中，我们需要执行显式转换。与 Java 不同，Kotlin 有一个特殊的`as`关键字，表示*不安全转换*操作符来处理转换：

```kt
val fragment: Fragment = ProductFragment() 
val productFragment: ProductFragment =  fragment as ProductFragment 
```

`ProductFragment`变量是`Fragment`的子类型，所以上面的例子将正常工作。问题在于将不兼容的类型转换为另一种类型会抛出异常`ClassCastException`。这就是为什么`as`操作符被称为不安全的转换操作符：

```kt
    val fragment : String = "ProductFragment" 
    val productFragment : ProductFragment =  fragment as 

        ProductFragment  

    \\ Exception: ClassCastException  
```

为了解决这个问题，我们可以使用*安全转换*操作符`as?`。有时它被称为**可空转换**操作符。该操作符尝试将一个值转换为指定的类型，并在无法转换值时返回 null。这是一个例子：

```kt
    val fragment: String = "ProductFragment" 
    val productFragment: ProductFragment? =  fragment as? 

        ProductFragment   
```

请注意，使用安全转换操作符要求我们将`name`变量定义为可空的（`ProductFragment?`而不是`ProductFragment`）。作为替代，我们可以使用不安全转换操作符和可空类型`ProductFragment?`，这样我们就可以确切地看到我们要转换为的类型：

```kt
    val fragment: String = "ProductFragment" 
    val productFragment: ProductFragment? =  fragment as 

        ProductFragment? 
```

如果我们想要一个非空的`productFragment`变量，那么我们必须使用 Elvis 操作符分配一个默认值：

```kt
    val fragment: String = "ProductFragment" 
    val productFragment: ProductFragment? = fragment as? 

        ProductFragment ?: ProductFragment() 
```

现在，`fragment as? ProductFragment`表达式将在没有任何错误的情况下进行评估。如果此表达式返回一个非空值（可以执行转换），则该值将分配给`productFragment`变量，否则将为`productFragment`变量分配一个默认值（`ProductFragment`的新实例）。以下是这两个操作符之间的比较：

+   不安全转换（`as`）：在转换不可能时抛出`ClassCastException`

+   安全转换（`as?`）：在转换不可能时返回 null

现在，当我们了解了安全转换和不安全转换操作符之间的区别，我们可以安全地从片段管理器中检索片段：

```kt
var productFragment: ProductFragment? = supportFragmentManager 
.findFragmentById(R.id.fragment_product) as? ProductFragment 
```

安全转换和不安全转换操作符用于转换复杂对象。在使用原始类型时，我们可以简单地使用 Kotlin 标准库的转换方法之一。Kotlin 标准库中的大多数对象都有用于简化常见类型转换的标准方法。约定是这种类型的函数具有前缀 to，并且我们要转换为的类的名称。在此示例中，`Int`类型使用`toString`方法转换为`String`类型：

```kt
val name: String 
    val age: Int = 12 
    name = age.toString(); // Converts Int to String
```

我们将在原始数据类型部分讨论原始类型及其转换。

# 智能转换

智能转换将一个类型的变量转换为另一个类型，但与安全转换相反，它是隐式完成的（我们不需要使用`as`或`as?`转换操作符）。智能转换仅在 Kotlin 编译器绝对确定变量在检查后不会被更改时才有效。这使它们非常适用于多线程应用程序。通常，智能转换适用于所有不可变引用（`val`）和本地可变引用（`var`）。我们有两种智能转换：

+   类型智能转换将一个类型的对象转换为另一个类型的对象

+   将可空引用转换为非空引用的空值智能转换

# 类型智能转换

让我们表示上一节中的`Animal`和`Fish`类：

![](img/Image00026.jpg)

假设我们想调用`isHungry`方法，并且我们想检查`animal`是否是`Fish`的实例。在 Java 中，我们必须这样做：

```kt
    \\Java 
    if (animal instanceof Fish){ 
        Fish fish = (Fish) animal; 
        fish.isHungry(); 

        //or 
        ((Fish) animal).isHungry(); 
    } 
```

这段代码的问题在于它的冗余性。我们必须检查`animal`实例是否为`Fish`，然后在此检查后明确将`animal`转换为`Fish`。如果编译器能够为我们处理这些冗余的转换，那不是很好吗？事实证明，当涉及到转换时，Kotlin 编译器确实很聪明，因此它将使用智能转换机制为我们处理所有这些冗余的转换。这是智能转换的一个例子：

```kt
    if(animal is Fish) { 
        animal.isHungry() 
    } 
```

Android Studio 中的智能转换

如果智能转换不可能，Android Studio 将显示适当的错误，因此我们将确切知道是否可以使用它。当我们访问需要转换的成员时，Android Studio 会用绿色背景标记变量。![](img/Image00027.jpg)

在 Kotlin 中，我们不必显式将`animal`实例转换为`Fish`，因为在类型检查后，Kotlin 编译器将能够隐式处理转换。现在，在`if`块内，变量`animal`被转换为`Fish`。结果与前面的 Java 示例完全相同（Kotlin 中调用操作符的实例与 Java 中相同）。因此，我们可以安全地调用`isHungry`方法，而无需进行任何显式转换。请注意，在这种情况下，此智能转换的范围受到`if`块的限制：

```kt
    if(animal is Fish) { 
        animal.isHungry() //1 
    } 

    animal.isHungry() //2, Error 
```

1.  在这种情况下，animal 实例是 Fish，所以我们可以调用`isHungry`方法。

1.  在这种情况下，animal 实例仍然是 Animal，所以我们无法调用`isHungry`方法。

然而，还有其他情况，智能转换范围大于单个块，就像以下示例中一样：

```kt
    val fish:Fish? = // ... 
    if (animal !is Fish) //1 
        return 

    animal.isHungry() //1 
```

1.  从这一点开始，animal 将被隐式转换为非空 Fish

在前面的示例中，如果`animal`不是`Fish`，整个方法将从函数返回，因此编译器知道`animal`在代码块的其余部分必须是`Fish`。Kotlin 和 Java 条件表达式是惰性求值的。

这意味着在表达式`condition1() && condition2()`中，只有当`condition1`返回`true`时才会调用方法`condition2`。这就是为什么我们可以在条件表达式的右侧使用*智能转换*类型：

```kt
    if (animal is Fish && animal.isHungry()) { 
        println("Fish is hungry") 
    } 
```

请注意，如果`animal`不是`Fish`，则根本不会评估条件表达式的第二部分。当它被评估时，Kotlin 知道`animal`是`Fish`（智能转换）。

# 非空智能转换

*智能转换*还处理其他情况，包括空值检查。假设我们有一个标记为可空的`view`变量，因为我们不知道`findViewById`是否会返回视图或 null：

```kt
val view: View? = findViewById(R.layout.activity_shop) 
```

我们可以使用安全调用运算符访问`view`的方法和属性，但在某些情况下，我们可能希望对同一对象执行更多操作。在这些情况下，智能转换可能是更好的解决方案：

```kt
    val view: View? 

    if ( view != null ){ 
        view.isShown() 

        // view is casted to non-nullable inside if code block 
    } 

    view.isShown() // error, outside if the block view is nullable 
```

在执行此类空检查时，编译器会自动将可空视图（`View*?*`）转换为非空（`View`）。这就是为什么我们可以在`if`块内调用`isShown`方法，而不使用安全调用运算符。在`if`块之外，视图仍然是可空的。

每个*智能转换*只适用于只读变量，因为读写变量可能在执行检查和访问变量之间发生变化。

*智能转换*也适用于函数的`return`语句。如果我们在函数内部执行空值检查并使用 return 语句，那么变量也将被转换为非空：

```kt
    fun setView(view: View?){

        if (view == null)

        return

        //view is casted to non-nullable

        view.isShown()

    }
```

在这种情况下，Kotlin 绝对确定变量值不会为空，因为否则函数将调用`return`。函数将在第三章*Playing with Functions*中更详细地讨论。我们可以使用 Elvis 运算符使前面的语法更简单，并在一行中执行空值检查：

```kt
    fun verifyView(view: View?){ 
        view ?: return 

        //view is casted to non-nullable 
        view.isShown() 
        //.. 
    } 
```

而不仅仅是从函数返回，我们可能希望更明确地说明现有问题并抛出异常。然后我们可以与错误抛出一起使用 Elvis 运算符：

```kt
    fun setView(view: View?){ 
        view ?: throw RuntimeException("View is empty") 

        //view is casted to non-nullable 
        view.isShown() 
    } 
```

正如我们所看到的，*智能转换*是一种非常强大的机制，可以减少空值检查的次数。这就是为什么它被 Kotlin 大量利用的原因。记住一般规则——*智能转换*只有在 Kotlin 绝对确定变量即使在另一个线程中也不会在转换后改变时才有效。

# 原始数据类型

在 Kotlin 中，一切都是对象（引用类型，而不是原始类型）。我们找不到像 Java 中可以使用的原始类型。这减少了代码复杂性。我们可以在任何变量上调用方法和属性。例如，这是我们如何将`Int`变量转换为`Char`：

```kt
    var code: Int = 75 
    code.toChar() 
```

通常（只要可能），底层类型如`Int`、`Long`或`Char`都经过优化（存储为原始类型），但我们仍然可以像在任何其他对象上一样调用它们的方法。

默认情况下，Java 平台将数字存储为 JVM 原始类型，但当需要可空数字引用（例如`Int?`）或涉及泛型时，Java 使用*装箱表示*。**装箱**意味着将原始类型包装成相应的装箱原始类型。这意味着实例的行为就像一个对象。Java 原始类型的装箱表示的示例是*int 与 Integer*或*long 与 Long*。由于 Kotlin 编译为 JVM 字节码，这里也是如此：

```kt
    var weight: Int = 12 // 1 
    var weight: Int? = null // 2 
```

1.  值以原始类型存储

1.  值以装箱整数（复合类型）存储

这意味着每次我们创建一个数字（`Byte`，`Short`，`Int`，`Long`，`Double`，`Float`），或者使用`Char`，`Boolean`，它将被存储为原始类型，除非我们将其声明为可空类型（`Byte?`，`Char?`，`Array?`等）；否则，它将被存储为装箱表示：

```kt
    var a: Int = 1 // 1 
    var b: Int? = null // 2 
    b = 12 // 3 
```

1.  `a`是非空的，因此它以原始类型存储

1.  `b`为空，因此它被存储为装箱表示

1.  `b`仍然以装箱表示存储，尽管它有一个值

无法使用原始类型对泛型类型进行参数化，因此将执行装箱。重要的是要记住，使用装箱表示（复合类型）而不是主要表示可能会导致性能损失，因为它将始终创建与原始类型表示相比的内存开销。对于包含大量元素的列表和数组，这可能是显而易见的，因此对于应用程序性能来说，使用主要表示可能至关重要。另一方面，当涉及单个变量甚至多个变量声明时，我们不必担心表示的类型，即使在内存有限的 Android 世界中也是如此。

现在让我们讨论最重要的 Kotlin 原始数据类型：数字、字符、布尔值和数组。

# 数字

用于数字的基本 Kotlin 数据类型等效于 Java 数值原语：

![](img/Image00028.jpg)

然而，Kotlin 处理数字的方式与 Java 有些不同。第一个区别是数字没有隐式转换--较小的类型不会隐式转换为较大的类型：

```kt
    var weight : Int = 12 
    var truckWeight: Long = weight // Error1 
```

这意味着我们不能将`Int`类型的值分配给`Long`变量而不进行显式转换。正如我们所说，在 Kotlin 中，一切都是对象，所以我们可以调用方法并显式将`Int`类型转换为`Long`来解决问题：

```kt
    var weight:I nt = 12 
    var truckWeight: Long = weight.toLong() 
```

起初，这可能看起来像样板代码，但实际上这将使我们避免许多与数字转换相关的错误，并节省大量调试时间。这实际上是 Kotlin 语法比 Java 更多的代码量的一个罕见例子。Kotlin 标准库支持以下数字转换方法：

+   `toByte()`：Byte

+   `toShort()`：Short

+   `toInt()`：Int

+   `toLong()`：Long

+   `toFloat()`：Float

+   `toDouble()`：Double

+   `toChar()`：Char

然而，我们可以显式指定数字文字以更改推断的变量类型：

```kt
    val a: Int = 1 
    val b = a + 1 // Inferred type is Int 
    val b = a + 1L // Inferred type is Long
```

Kotlin 和 Java 数字之间的第二个区别是，某些情况下数字文字略有不同。对于整数值，有以下类型的文字常量：

```kt
    27 // Decimals by default 
    27L // Longs are tagged by a upper case L suffix 
    0x1B // Hexadecimals are tagged by 0x prefix 
    0b11011 // Binaries are tagged by 0b prefix 
```

不支持八进制文字。Kotlin 还支持浮点数的传统表示法：

```kt
    27.5 // Inferred type is Double 
    27.5F // Inferred type is Float. Float are tagged by f or F 
```

# Char

Kotlin 中的字符存储在类型`Char`中。在许多方面，字符类似于字符串，因此我们将集中讨论相似之处和不同之处。要定义`Char`，我们必须使用单引号，与`String`相反，我们在其中使用双引号：

```kt
    val char = 'a' \\ 1 
    val string = "a" \\ 2 
```

1.  定义类型为`Char`的变量

1.  定义类型为`String`的变量

在字符和字符串中，可以使用反斜杠来转义特殊字符。支持以下转义序列：

+   `\t`：制表符

+   `\b`：退格

+   `\n`：换行

+   `\r`：换行

+   `\'`：引号

+   `\"`：双引号

+   `\\`：斜杠

+   `\$`：美元符号

+   `\u`：Unicode 转义序列

让我们定义包含阴阳 Unicode 字符（U+262F）的*Char*：

```kt
    var yinYang = '\u262F'
```

# 数组

在 Kotlin 中，数组由`Array`类表示。要在 Kotlin 中创建数组，我们可以使用许多 Kotlin 标准库函数。最简单的是`arrayOf()`：

```kt
    val array = arrayOf(1,2,3)   // inferred type Array<Int> 
```

默认情况下，此函数将创建一个装箱`Int`的数组。如果我们想要一个包含*Short*或*Long*的数组，那么我们必须明确指定数组类型：

```kt
    val array2: Array<Short> = arrayOf(1,2,3) 
    val array3: Array<Long> = arrayOf(1,2,3) 
```

如前所述，使用装箱表示可能会降低应用程序的性能。这就是为什么 Kotlin 有一些专门表示原始类型数组的类，以减少装箱内存开销：`ShortArray`，`IntArray`，`LongArray`等。这些类与`Array`类没有继承关系，尽管它们具有相同的一组方法和属性。要创建这些类的实例，我们必须使用相应的工厂函数：

```kt
    val array =  shortArrayOf(1, 2, 3) 
    val array =  intArrayOf(1, 2, 3) 
    val array =  longArrayOf(1, 2, 3) 
```

重要的是要注意并牢记这个微妙的差别，因为这些方法看起来相似，但创建了不同的类型表示：

```kt
    val array = arrayOf(1,2,3) // 1 
    val array = longArrayOf(1, 2, 3) // 2 
```

1.  泛型数组的装箱长元素（推断类型：`Array<Long>`）

1.  包含原始长元素的数组（推断类型：`LongArray`）

知道数组的确切大小通常会提高性能，因此 Kotlin 还有另一个库函数`arrayOfNulls`，它创建一个给定大小的数组，其中填充了空元素：

```kt
    val array = arrayOfNulls(3) // Prints: [null, null, null] 
    println(array) // Prints: [null, null, null] 
```

我们还可以使用将数组大小作为第一个参数并且可以返回每个数组元素的初始值的 lambda 作为第二个参数的工厂函数来填充预定义大小的数组：

```kt
    val array = Array (5) { it * 2 } 
    println(array) // Prints: [0, 2, 4, 8, 10] 
```

我们将在第五章，*函数作为一等公民*中更详细地讨论 lambda（匿名函数）。在 Kotlin 中访问数组元素的方式与 Java 相同：

```kt
    val array = arrayOf(1,2,3) 
    println(array[1]) //Prints: 2 
```

元素的索引方式也与 Java 相同，意味着第一个元素的索引为 0，第二个元素的索引为 1，依此类推。并非所有的东西都一样，也存在一些差异。主要的一个是，Kotlin 中的数组与 Java 不同，Kotlin 中的数组是不变的。我们将在第六章，*泛型是你的朋友*中讨论*变异*。

# 布尔类型

布尔是一个逻辑类型，有两个可能的值：`true`和`false`。我们还可以使用可空布尔类型：

```kt
    val isGrowing: Boolean = true 
    val isGrowing: Boolean? = null 
```

布尔类型还支持通常在大多数现代编程语言中可用的标准内置操作：

+   `||`：逻辑或。当两个谓词中任何一个返回`true`时返回`true`。

+   `&&`：逻辑与。当两个谓词都返回`true`时返回`true`。

+   `!`：否定运算符。对于`false`返回`true`，对于`true`返回`false`。

请记住，我们只能对任何类型的条件使用非空布尔值。

与 Java 一样，在`||`和`&&`中，谓词是惰性评估的，只有在需要时才会评估（*惰性连接*）。

# 复合数据类型

让我们讨论内置到 Kotlin 中的更复杂的类型。一些数据类型与 Java 相比有重大改进，而另一些则是全新的。

# 字符串

Kotlin 中的字符串的行为方式与 Java 类似，但它们有一些很好的改进。

要开始访问指定索引处的字符，我们可以使用*索引*运算符，并以访问数组元素的方式访问字符：

```kt
    val str = "abcd" 
    println (str[1]) // Prints: b 
```

我们还可以访问 Kotlin 标准库中定义的各种扩展，这些扩展使得处理字符串更加容易：

```kt
    val str = "abcd" 
    println(str.reversed()) // Prints: dcba 
    println(str.takeLast(2)) // Prints: cd 
    println("john@test.com".substringBefore("@")) // Prints: john 
    println("john@test.com".startsWith("@")) // Prints: false 
```

这正是 Java 中的`String`类，因此这些方法不是`String`类的一部分。它们被定义为扩展。我们将在第七章，*扩展函数和属性*中学习更多关于扩展的知识。

查看`String`类文档以获取方法的完整列表（[`kotlinlang.org/api/latest/jvm/stdlib/kotlin/-string/`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin/-string/)）。

# 字符串模板

构建字符串是一个简单的过程，但在 Java 中通常需要长的连接表达式。让我们直接跳到一个例子。这是在 Java 中实现的由多个元素构建的字符串：

```kt
\\Java 
String name = "Eva"; 
int age = 27; 
String message = "My name is" + name + "and I am" + age + "years old";
```

在 Kotlin 中，我们可以通过使用*字符串模板*大大简化字符串创建的过程。我们可以简单地使用一个美元符号将变量放在字符串中，而不是使用连接。在插值期间，字符串占位符将被实际值替换。这里有一个例子：

```kt
    val name = "Eva" 
    val age = 27 
    val message = "My name is $name and I am $age years old" 
    println(message) 

    //Prints: My name is Eva  and I am 27 years old 
```

这与连接一样高效，因为在底层编译代码创建了一个`StringBuilder`并将所有部分附加在一起。字符串模板不仅限于单个变量。它们还可以在`${`和`}`字符之间包含整个表达式。它可以是一个函数调用，将返回值或属性访问，如下面的片段所示：

```kt
    val name = "Eva" 
    val message = "My name has ${name.length} characters" 
    println(message) //Prints: My name has 3 characters 
```

这种语法允许我们创建更清晰的代码，而无需每次需要从变量或表达式中获取值来构造字符串时都打破字符串。

# 范围

范围是定义一系列值的一种方式。它由序列中的第一个和最后一个值表示。我们可以使用范围来存储重量、温度、时间和年龄。范围使用双点符号（在底层，范围使用`rangeTo`运算符）来定义：

```kt
    val intRange = 1..4 // 1 
    val charRange= 'b'..'g' // 2 
```

1.  推断类型是`IntRange`（相当于`i >= 1 && i <= 4`）

1.  推断类型是`CharRange`（相当于从`'b'`到`'g'`的字母）

请注意，我们使用单引号来定义字符范围。

`Int`，`Long`和`Char`类型的范围可用于在`for... each`循环中迭代下一个值：

```kt
    for (i in 1..5) print(i) // Prints: 1234 
    for (i in 'b'..'g') print(i) // Prints: bcdefg 
```

范围可以用来检查一个值是否大于起始值并且小于结束值：

```kt
    val weight = 52 
    val healthy = 50..75 

    if (weight in healthy) 
        println("$weight is in $healthy range") 

        //Prints: 52 is in 50..75 range 
```

它也可以用于其他类型的范围，比如`CharRange`：

```kt
    val c = 'k'      // Inferred type is Char
    val alphabet = 'a'..'z'  

    if(c in alphabet) 
        println("$c is character") //Prints: k is a character 
```

在 Kotlin 中，范围是闭合的（包括结束）。这意味着范围结束值包括在范围内：

```kt
    for (i in 1..1) print(i) // Prints: 123
```

注意，Kotlin 中的范围默认是递增的（默认情况下步长为 1）：

```kt
    for (i in 5..1) print(i) // Prints nothing 
```

要以相反的顺序迭代，我们必须使用`downTo`函数，将步长设置为`-1`。就像这个例子：

```kt
    for (i in 5 downTo 1) print(i) // Prints: 54321 
```

我们也可以设置不同的步骤：

```kt
    for (i in 3..6 step 2) print(i) // Prints: 35 
```

请注意，在`3..6`范围内，最后一个元素没有被打印出来。这是因为*步进索引*在每个循环迭代中移动两步。因此，在第一次迭代中，它的值为`3`，在第二次迭代中，它的值为`5`，最后，在第三次迭代中，值将为`7`，因此它被忽略，因为它在范围之外。

由`step`函数定义的步骤必须是正的。如果我们想定义一个负步骤，那么我们应该使用`downTo`函数和`step`函数一起使用：

```kt
    for (i in 9 downTo 1 step 3) print(i) // Prints: 963 
```

# 集合

编程的一个非常重要的方面是使用集合。Kotlin 提供了多种类型的集合，并与 Java 相比有许多改进。我们将在第七章中讨论这个主题，*扩展函数和属性*。

# 语句与表达式

Kotlin 比 Java 更广泛地使用表达式，因此了解*语句*和*表达式*之间的区别很重要。程序基本上是一系列语句和表达式。表达式产生一个值，可以作为另一个表达式、变量赋值或函数参数的一部分使用。表达式是一个或多个*操作数*（被操作的数据）和零个或多个*操作符*（表示特定操作的标记）的序列，可以评估为单个值。

![](img/Image00029.jpg)

让我们回顾一些来自 Kotlin 的表达式的例子：

| **表达式（产生一个值）** | **分配的值** | **类型的表达式** |
| --- | --- | --- |
| `a = true` | `true` | 布尔值 |
| `a = "foo" + "bar"` | `"foobar"` | 字符串 |
| `a = min(2, 3)` | `2` | 整数 |
| `a = computePosition().getX()` | `getX`方法返回的值 | 整数 |

另一方面，语句执行一个动作，不能赋值给变量，因为它们根本没有值。语句可以包含用于定义类（`class`），接口（`interface`），变量（`val`，`var`），函数（`fun`），循环逻辑（`break`，`continue`）等的语言关键字。当表达式返回的值被忽略时（不将值赋给变量，不从函数中返回它，不将其用作其他表达式的一部分等），表达式也可以被视为语句。

Kotlin 是一种面向表达式的语言。这意味着在 Kotlin 中，许多在 Java 中是语句的构造在 Kotlin 中被视为表达式。第一个主要的区别是 Java 和 Kotlin 对待*控制结构*的方式不同。在 Java 中，它们被视为语句，而在 Kotlin 中，所有控制结构都被视为表达式，除了循环。这意味着在 Kotlin 中，我们可以使用控制结构编写非常简洁的语法。我们将在接下来的章节中看到例子。

# 控制流

Kotlin 有许多来自 Java 的控制流元素，但它们提供了更灵活的使用方式，在某些情况下，它们的使用更简化了。Kotlin 引入了一个称为`when`的新控制流构造，作为 Java `switch... case`的替代品。

# if 语句

在本质上，Kotlin 的`if`子句的工作方式与 Java 相同：

```kt
    val x = 5 

    if(x > 10){ 
        println("greater") 
    } else { 
        println("smaller") 
    } 
```

带有块体的版本如果块包含单个语句或表达式也是正确的：

```kt
    val x = 5 

    if(x > 10) 
        println("greater") 
    else 
        println("smaller") 
```

然而，Java 将`if`视为*语句*，而 Kotlin 将`if`视为*表达式*。这是主要区别，这个事实使我们能够使用更简洁的语法。例如，我们可以直接将`if`表达式的结果作为函数参数传递：

```kt
    println(if(x > 10) "greater" else "smaller") 
```

我们可以将我们的代码压缩成一行，因为`if`表达式的结果（String 类型）被评估，然后传递给`println`方法。当条件`x > 10`为`true`时，这个表达式将返回第一个分支（greater），否则这个表达式将返回第二个分支（smaller）。让我们看另一个例子：

```kt

    val hour = 10 
    val greeting: String 
    if (hour < 18) { 
        greeting = "Good day" 
    } else { 
        greeting = "Good evening" 
    } 
```

在前面的例子中，我们将`if`用作语句。但是我们知道，在 Kotlin 中，`if`是一个表达式，表达式的结果可以赋值给一个变量。这样我们可以直接将`if`表达式的结果赋值给一个 greeting 变量：

```kt
    val greeting = if (hour < 18) "Good day" else "Good evening" 
```

但有时需要在`if`语句的分支中放置一些其他代码。我们仍然可以将`if`用作表达式。然后匹配`if`分支的最后一行将作为结果返回：

```kt
    val hour = 10 

    val greeting = if (hour < 18) { 
        //some code 
        "Good day" 
    } else { 
        //some code 
        "Good evening" 
    } 

    println(greeting) // Prints: "Good day" 
```

如果我们将`if`视为表达式而不是语句，则表达式需要有一个`else`分支。Kotlin 版本甚至比 Java 更好。由于`greeting`变量被定义为非空，编译器将验证整个`if`表达式，并检查所有情况是否都有分支条件。由于`if`是一个表达式*，*我们可以在*字符串模板*中使用它：

```kt
val age = 18 
val message = "You are ${ if (age < 18) "young" else "of age" } person" 
println(message) // Prints: You are of age person 
```

将`if`视为*表达式*给我们带来了以前在 Java 世界中无法实现的广泛可能性。

# when 表达式

Kotlin 中的`when`表达式是一种多路分支语句。`when`表达式被设计为 Java `switch... case`语句的更强大的替代品。`when`语句通常提供了比大量的`if... else if`语句更好的替代方案，但它提供了更简洁的语法。让我们看一个例子：

```kt
    when (x) { 
        1 -> print("x == 1") 
        2 -> print("x == 2") 
        else -> println("x is neither 1 nor 2") 
    } 
```

`when`表达式将其参数与所有分支一一匹配，直到满足某个分支的条件。这种行为类似于 Java 的`switch... case`，但我们不必在每个分支后写冗余的`break`语句。

与 `if` 子句类似，我们可以将 `when` 用作*语句*，忽略返回值，也可以将其用作*表达式*，并将其值赋给变量。如果 `when` 用作*表达式*，则满足分支的最后一行的值成为整体表达式的值。如果它用作语句，则值将被简单地忽略。通常情况下，如果没有一个前面的分支满足条件，`else` 分支将被评估：

```kt
    val vehicle = "Bike" 

    val message= when (vehicle) { 
        "Car" -> { 
            // Some code 
            "Four wheels" 
        } 
        "Bike" -> { 
            // Some code 
            "Two wheels" 
        } 
        else -> { 
            //some code 
            "Unknown number of wheels" 
        } 
    } 

    println(message) //Prints: Two wheels 
```

每当一个分支有多于一条指令时，我们必须将其放在由两个大括号 `{...}` 定义的代码块内。如果 `when` 被视为表达式（`when` 的评估结果被赋值给变量），每个块的最后一行被视为返回值。我们已经看到了 `if` 表达式的相同行为，所以现在我们可能已经意识到这是 Kotlin 许多构造的共同行为，包括将在本书中进一步讨论的 lambda。

如果 `when` 用作表达式，`else` 分支是强制的，除非编译器可以证明分支条件覆盖了所有可能的情况。我们还可以使用逗号将多个匹配参数放在单个分支中处理：

```kt
    val vehicle = "Car" 

    when (vehicle) { 
        "Car", "Bike" -> print("Vehicle")
        else -> print("Unidentified funny object") 
    } 
```

`when` 的另一个好特性是检查变量类型的能力。我们可以轻松验证值是否是特定类型的 `is` 或 `!is`。智能转换再次变得方便，因为我们可以在分支块中访问匹配类型的方法和属性，而无需进行任何额外的检查：

```kt
    val name = when (person) { 
        is String -> person.toUpperCase()
        is User -> person.name 

        //Code is smart casted to String, so we can 

        //call String class methods 

        //... 

    } 
```

类似地，我们可以检查范围或集合是否包含特定值。这次我们将使用 `is` 和 `!is` 关键字：

```kt
    val riskAssessment = 47 

    val risk = when (riskAssessment) { 
        in 1..20 -> "negligible risk" 
        !in 21..40 -> "minor risk" 
        !in 41..60 -> "major risk" 
        else -> "undefined risk" 
    } 

    println(risk) // Prints: major risk 
```

实际上，我们可以在 `when` 分支的右侧放置任何类型的表达式。它可以是方法调用或任何其他表达式。考虑以下示例，其中第二个 `when` 表达式用于 `else` 语句：

```kt
    val riskAssessment = 80 
    val handleStrategy = "Warn" 

    val risk = when (riskAssessment) { 
        in 1..20 -> print("negligible risk") 
        !in 21..40 -> print("minor risk") 
        !in 41..60 -> print("major risk") 
        else -> when (handleStrategy){ 
            "Warn" -> "Risk assessment warning"  
            "Ignore" -> "Risk ignored" 
            else -> "Unknown risk!" 
        }  
    } 

    println(risk) // Prints: Risk assessment warning 
```

正如我们所看到的，`when` 是一个非常强大的构造，比 Java 的 `switch` 更具控制力，但它更强大的原因在于它不仅仅局限于检查相等的值。在某种程度上，它甚至可以用作 `if... else if` 链的替代。如果没有向 `when` 表达式提供参数，分支条件会表现为布尔表达式，当条件为 `true` 时执行分支：

```kt
private fun getPasswordErrorId(password: String) = when { 
    password.isEmpty() -> R.string.error_field_required 
    passwordInvalid(password) -> R.string.error_invalid_password 
    else -> null 
} 
```

所有呈现的示例都需要一个 `else` 分支。每当所有可能的情况都被覆盖时，我们可以省略 `else` 分支（穷尽 `when`）。让我们看一个布尔值的最简单的例子：

```kt
    val large:Boolean = true 

    when(large){ 
        true -> println("Big") 
        false -> println("Big") 
    } 
```

编译器可以验证所有可能的值都已处理，因此不需要指定 `else` 分支。相同的逻辑适用于将在第四章中讨论的枚举和密封类，即*类和对象*。

Kotlin 编译器执行检查，因此我们可以确定不会漏掉任何情况。这减少了在 `switch` 语句内部开发人员忘记处理所有情况的常见 Java bug 的可能性（尽管多态通常是更好的解决方案）*。*

# 循环

循环是一种控制结构，重复相同的指令集，直到满足终止条件。在 Kotlin 中，循环可以迭代任何提供迭代器的内容。迭代器是一个具有两个方法的接口：`hasNext` 和 `next`。它知道如何迭代集合、范围、字符串或任何可以表示为元素序列的实体。

要迭代某些内容，我们必须提供一个 `iterator()` 方法。由于 `String` 没有这个方法，因此在 Kotlin 中它被定义为一个扩展函数。扩展将在第七章中介绍，即*扩展函数和属性*。

Kotlin 提供了三种类型的循环：`for`、`while` 和 `do... while`。它们都与其他编程语言中的循环相同，因此我们将简要讨论它们。

# for 循环

在 Kotlin 中不存在经典的 Java `for` 循环，需要显式定义迭代器。以下是 Java 中这种类型循环的示例：

```kt
    //Java 
    String str = "Foo Bar"; 
    for(int i=0; i<str.length(); i++) 
    System.out.println(str.charAt(i)); 
```

要从头到尾迭代项目集合，我们可以简单地使用`for`循环：

```kt
    var array = arrayOf(1, 2, 3) 

    for (item in array) { 
        print(item) 
    } 
```

它也可以在没有块体的情况下定义：

```kt
    for (item in array) 
        print(item) 
```

如果`collection`是一个泛型集合，那么*item*将被智能转换为与泛型集合类型相对应的类型。换句话说，如果集合包含`Int`类型的元素，那么 item 将被智能转换为`Int`：

```kt
    var array = arrayOf(1, 2, 3) 

    for (item in array) 
        print(item) // item is Int 
```

我们还可以通过索引遍历集合：

```kt
    for (i in array.indices) 
        print(array[i]) 
```

`array.indices`参数返回带有所有索引的`IntRange`。它相当于(`1.. array.length - 1` *)*。还有一个替代的`withIndex`库方法，它返回一个包含索引和值的`IndexedValue`属性的列表。可以这样解构这些元素：

```kt
    for ((index, value) in array.withIndex()) { 
        println("Element at $index is $value") 
    } 
```

`(index, value)`结构被称为破坏性声明，我们将在第四章，*类和对象*中讨论它。

# while 循环

`while`循环重复一个块，当其条件表达式返回`true`时：

```kt
    while (condition) { 
        //code 
    } 
```

还有一个`do... while`循环，它会重复块，只要*条件表达式*返回`true`：

```kt
    do { 
        //code 
    } while (condition)  
```

与 Java 相反，Kotlin 可以使用在`do... while`循环内声明的变量作为条件。

```kt
    do { 
        var found = false 
        //.. 
    } while (found) 
```

`while`和`do... while`循环之间的主要区别在于条件表达式的评估时间。`while`循环在代码执行之前检查条件，如果条件不为真，则代码不会被执行。另一方面，`do... while`循环首先执行循环体，然后评估条件表达式，因此循环体至少会执行一次。如果这个表达式为`true`，循环将重复。否则，循环终止。

# 其他迭代

还有其他使用内置标准库函数进行集合迭代的方法，比如`forEach`。我们将在第七章，*扩展函数和属性*中介绍它们。

# Break and continue

Kotlin 中的所有循环都支持经典的`break`和`continue`语句。`continue`语句继续执行该循环的下一次迭代，而`break`停止最内部封闭循环的执行：

```kt
    val range = 1..6 

    for(i in range) { 
        print("$i ") 
    } 

    // prints: 1 2 3 4 5 6 
```

现在让我们添加一个`条件`，并在这个条件为`true`时中断迭代：

```kt
    val range = 1..6 

    for(i in range) { 
        print("$i ") 

        if (i == 3) 
            break 
    } 

    // prints: 1 2 3 
```

`break`和`continue`语句在处理嵌套循环时特别有用。它们可以简化我们的控制流程，并显著减少执行的工作量，以节省宝贵的 Android 资源。让我们执行一个嵌套迭代并中断外部循环：

```kt
    val intRange = 1..6 
    val charRange = 'A'..'B' 

    for(value in intRange) { 
        if(value == 3) 
            break 

        println("Outer loop: $value ") 

        for (char in charRange) { 
            println("\tInner loop: $char ") 
        } 
    } 

    // prints 
    Outer loop: 1  
        Inner loop: A  
        Inner loop: B  
    Outer loop: 2  
        Inner loop: A  
        Inner loop: B  
```

我们使用`break`语句在第三次迭代开始时终止外部循环，因此嵌套循环也被终止。请注意在控制台上添加缩进的`\t`转义序列的使用。我们还可以利用`continue`语句跳过循环的当前迭代：

```kt
    val intRange = 1..5 

    for(value in intRange) { 
        if(value == 3) 
            continue 

        println("Outer loop: $value ") 

        for (char in charRange) { 
            println("\tInner loop: $char ") 
        } 
    } 

    // prints 
    Outer loop: 1  
        Inner loop: A  
        Inner loop: B  
    Outer loop: 2  
        Inner loop: A  
        Inner loop: B  
    Outer loop: 4  
        Inner loop: A  
        Inner loop: B  
    Outer loop: 5  
        Inner loop: A  
        Inner loop: B  
```

当当前值等于`3`时，我们跳过外部循环的迭代。

`continue`和`break`语句都在封闭循环上执行相应的操作。然而，有时我们希望从一个循环中终止或跳过另一个循环的迭代；例如，从内部循环中终止外部循环的迭代：

```kt
    for(value in intRange) { 
        for (char in charRange) { 
            // How can we break outer loop here? 
        } 
    } 
```

幸运的是，`continue`语句和`break`语句都有两种形式--有标签和无标签。我们已经看到了无标签，现在我们需要有标签来解决我们的问题。下面是一个使用有标签的 break 的例子：

```kt
    val charRange = 'A'..'B' 
    val intRange = 1..6 

    outer@ for(value in intRange) { 
        println("Outer loop: $value ") 

        for (char in charRange) { 
            if(char == 'B') 
                break@outer 

            println("\tInner loop: $char ") 
        } 
    } 

    // prints 
    Outer loop: 1  
        Inner loop: A  
```

`@outer`是标签名称。按照惯例，标签名称始终以`*@*`开头，后跟标签名称。标签放置在循环之前。给循环加标签允许我们使用有资格的`break`（`break@outer`），这是一种停止引用该标签的循环的执行的方法。前面的有资格的`break`（带标签的 break）跳转到标记有该标签的循环之后的执行点。

放置`return`语句将中断所有循环并从匿名或命名函数中返回：

```kt
    fun doSth() { 
        val charRange = 'A'..'B' 
        val intRange = 1..6 

        for(value in intRange) { 
            println("Outer loop: $value ") 

            for (char in charRange) { 
                println("\tInner loop: $char ") 

                return 
            } 
        }   
    } 

    //usage 
    println("Before method call") 
    doSth() 
    println("After method call") 

    // prints

    Outer loop: 1 

        Inner loop: A  
```

方法调用之后：

```kt
    Outer loop: 1  
        Inner loop: A  
```

# 异常

大多数 Java 编程指南，包括《Effective Java》这本书，都提倡有效性检查的概念。这意味着我们应该始终验证参数或对象的状态，并在有效性检查失败时抛出异常。Java 异常系统有两种异常：已检查异常和未检查异常。

未检查异常意味着开发人员不必使用`try... catch`块来捕获异常。默认情况下，异常会一直传递到调用堆栈的最上层，因此我们可以决定在哪里捕获它们。如果我们忘记捕获它们，它们将一直传递到调用堆栈的最上层，并以适当的消息停止线程执行（因此它们会提醒我们）：

![](img/Image00030.jpg)

Java 有一个非常强大的异常系统，在许多情况下，它迫使开发人员明确标记可能引发异常的每个函数，并通过将它们包围在`try... catch`块中明确捕获每个异常（已检查异常）。这对于非常小的项目效果很好，但在真正的大型应用程序中，这往往会导致冗长的代码：

```kt
    // Java 
    try { 
        doSomething() 
    } catch (IOException e) { 
        // Must be safe 
    } 
```

不是将异常传递到调用堆栈中，而是通过提供一个空的 catch 块来忽略它，因此它不会被正确处理，而是会消失。这种代码可能掩盖关键异常，并给人一种错误的安全感，导致意外问题和难以找到的错误。

在讨论 Kotlin 中如何处理异常之前，让我们比较一下两种类型的异常：

| **代码** | **已检查异常** | **未检查异常** |
| --- | --- | --- |
| 函数声明 | 我们必须指定函数可能抛出的异常。 | 函数声明不包含所有抛出异常的信息。 |
| 异常处理 | 抛出异常的函数必须被`try... catch`块包围。 | 我们可以捕获异常并在需要时执行某些操作，但我们不是被迫这样做。异常在调用堆栈中上升。 |

Kotlin 和 Java 异常系统之间最大的区别在于，在 Kotlin 中所有异常都是未检查的。这意味着即使这是一个可能引发已捕获异常的 Java 方法，我们也永远不必用`try... catch`块包围一个方法。我们仍然可以这样做，但我们不是被迫这样做：

```kt
    fun foo() { 
        throw IOException() 
    } 

    fun bar() { 
        foo () //no need to surround method with try-catch block 
    } 
```

这种方法消除了代码冗余，并提高了安全性，因为我们不需要引入空的*catch*块。

# `try... catch`块

Kotlin 的`try... catch`块相当于 Java 的`try... catch`块。让我们看一个快速的例子：

```kt
    fun sendFormData(user: User?, data: Data?) { // 1 
        user ?: throw NullPointerException("User cannot be null") 

        // 2 
        data ?: throw NullPointerException("Data cannot be null")         

        //do something 
    } 

    fun onSendDataClicked() { 
        try { // 3 
            sendFormData(user, data) 
        } catch (e: AssertionError) { // 4 
            // handle error 
        }  finally { // 5 
            // optional finally block 
        } 
    } 
```

1.  异常没有像 Java 那样在函数签名上指定。

1.  我们检查数据的有效性，并抛出`NullPointerException`（请注意，在创建对象实例时不需要使用 new 关键字）。

1.  `try... catch`块是 Java 中的类似结构。

1.  仅处理特定的异常（`AssertionError`异常）。

1.  `finally`块总是被执行。

可能有零个或多个`catch`块，`finally`块可以省略。但是，至少应该有一个`catch`或`finally`块存在。

在 Kotlin 中，异常处理`try`是一个表达式，因此它可以返回一个值，我们可以将其值分配给一个变量。实际分配的值是执行块的最后一个表达式。让我们检查设备上是否安装了特定的 Android 应用程序：

```kt
val result = try { // 1 
    context.packageManager.getPackageInfo("com.text.app", 0)  //2 
    true 
} catch (ex: PackageManager.NameNotFoundException) { // 3 
    false 
} 
```

1.  `try... catch`块返回的值是由单个表达式函数返回的值。

1.  如果应用程序已安装，`getPackageInfo`方法将返回一个值（此值被忽略），并且将执行包含`true`表达式的下一行。这是`try`块执行的最后一个操作，因此它的值将被分配给一个变量（`true`）。

如果应用程序未安装，`getPackageInfo`将抛出`PackageManager.NameNotFoundException`，并且将执行`catch`块的最后一行，其中包含一个`false`表达式，因此它的值将被分配给一个变量。

# 编译时常量

由于`val`变量是只读的，在大多数情况下我们可以将其视为常量。我们需要意识到它的初始化可能会延迟，这意味着有些情况下`val`变量可能在编译时未被初始化，例如，将方法调用的结果赋给一个值：

```kt
   val fruit:String  = getName() 
```

这个值将在运行时被赋值。然而，有些情况下我们需要在编译时知道这个值。当我们想要向注解传递参数时，需要确切的值。注解是由注解处理器处理的，它在应用程序启动之前就运行了：

![](img/Image00031.jpg)

为了确保值在编译时是已知的（因此可以被注解处理器处理），我们需要用`const`修饰符标记它。让我们定义一个自定义注解`MyLogger`，并用一个参数定义最大日志条目，并用它注解一个`Test`类：

```kt
    const val MAX_LOG_ENTRIES = 100 

        @MyLogger(MAX_LOG_ENTRIES ) 

        // value available at compile time

        class Test {}
```

关于使用`const`有一些限制我们必须意识到。第一个限制是它必须用原始类型或`String`类型的值进行初始化。第二个限制是它必须在顶层声明或作为对象的成员声明。我们将在第四章，*类和对象*中讨论对象。第三个限制是它们不能有自定义的 getter。

# 代表

Kotlin 为委托提供了一流的支持。与 Java 相比，这是非常有用的改进。事实上，在 Android 开发中有许多委托的应用，因此我们决定在这个主题上多花一个章节（第八章，*代表*）。

# 总结

在本章中，我们讨论了变量、值和常量之间的区别，并讨论了基本的 Kotlin 数据类型，包括范围。我们还研究了一个强制严格的空安全的 Kotlin 类型系统，以及使用各种操作符和智能转换处理可空引用的方法。我们知道，通过利用类型推断和 Kotlin 中被视为表达式的各种控制结构，我们可以编写更简洁的代码。最后，我们讨论了异常处理的方法。

在下一章中，我们将学习关于函数，并介绍不同的定义方式。我们将涵盖单表达式函数、默认参数和命名参数语法等概念，并讨论各种修饰符。
