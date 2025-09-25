# 第二章：Gradle 的 Groovy 基础知识

在本章中，我们将学习 Groovy 编程语言的一些基本概念。本章简要介绍了 Groovy 数据类型、控制结构、面向对象概念、集合、闭包和构建器。这只是冰山一角。由于这不是一本 Groovy 书籍，我们无法涵盖所有主题。本章旨在为那些来自 Java 背景并基本了解**面向对象编程**（**OOP**）概念的初学者提供帮助。这将帮助他们开始使用 Groovy。本章还将作为继续 Gradle 脚本编写的一个工具箱。

# 概述

Groovy 是 Java 平台上的动态编程语言。你可能想知道为什么我们特别提到 Java 平台。通过 Java 平台，我们的意思是 Groovy 代码编译成字节码，字节码在 JVM 上执行，类似于任何其他 Java 类。除了面向对象（OOP）特性外，它还提供了脚本语言（如 Python 和 Smalltalk）的能力，使得它们可以通过类似 Java 的语法在 Groovy 中使用。

由于 Groovy 运行在 JVM 上，它可以很容易地与 Java 集成，并且很好地融入现有的基础设施中。例如，Groovy 代码的构建和部署与 Java 代码的构建和部署相同，你只需向库中添加另一个 JAR 文件就可以轻松地将 Groovy 和 Java 混合在一起。Groovy 不是唯一运行在 JVM 上的语言。其他一些语言包括 Scala、Clojure、JRuby、Jython 等等。在我看来，如果你有一些 Java 背景，与其它语言相比，Groovy 的学习要容易得多。它具有非常类似 Java 的语法，并且大多数 Java 语法都是有效的 Groovy 语法。它只是简化了编码。Groovy 从未打算取代 Java。它的目的是补充 Java，扩展它使其更容易使用，并且它还使用了现代语言特性，如闭包、构建器和元编程。

以下是一些 Groovy 的关键特性。

## 与 Java 的集成

许多人认为 Groovy 是一种脚本语言。是的，它提供了脚本支持，但说 Groovy 只是一种脚本语言是不正确的。除了脚本之外，它完美地融入了面向对象的世界。如前所述，Groovy 还提供了与 Java 的无缝集成。从 Groovy 调用 Java 就像编写 Groovy 代码一样简单。每个 Groovy 类型都是`java.lang.Object`的子类型。

## 最小化代码

Groovy 的一个优点是它减少了执行一些复杂任务（如解析 XML 文件和访问数据库）所需的代码量。使用 Groovy，你总是可以混合 Java 代码。如果你已经使用 Java 一段时间了，我认为你会欣赏使用 Groovy 的简单性，因为你可以通过编写更少的代码来实现更多的功能。

## 简化的 I/O 操作

在使用 Java 进行开发时，I/O 操作是开发者面临的主要痛点之一，在 Groovy 中变得简单得多。在 Groovy 中执行 I/O 操作更有趣。Groovy 提供了简单的属性来读取/写入文件。它已经向`java.io.File`类添加了如此多的实用方法。

## 与 Ant 的集成

与 Java 一样，Groovy 提供了与 Ant 的无缝集成。Groovy 有一个辅助类`AntBuilder`，它赋予了 Groovy 使用 Ant 功能的能力，使开发者的生活变得更加简单。无论是计算任何文件的校验和，还是根据任何过滤条件将目录内容从一个位置复制到另一个位置。借助 Ant 功能，Groovy 使开发者更加高效。在第八章“迁移”中，我们将更详细地讨论这个话题。

## 构建器类

除了 AntBuilder 之外，Groovy 还提供了`NodeBuilder`、`MarkupBuilder`和`SwingBuilder`的能力。借助这些构建器，开发者能够以更简单的方式完成任务，与没有构建器的生活相比。`MarkupBuilder`在处理 XML 操作时很有用。`SwingBuilder`提供了简化了 Swing 框架的 API，有助于构建用户友好的 GUI 应用程序。`NodeBuilder`在处理对象树结构时很有帮助。

## 闭包

闭包的引入是 Groovy 的一个大卖点。Groovy 中的闭包是一个匿名代码块，它可以接受参数，返回值，并引用和使用其周围作用域中声明的变量。闭包通常与函数式语言相关联。

Groovy 闭包就像是一个定义后并在稍后执行的代码块。它具有一些特殊属性，例如隐式变量和自由变量。我们将在本章后面的部分详细讨论闭包。

当然，还有很多其他特性需要学习。我们将在本章中讨论其中的一些。更多详细信息，请参阅[`www.groovy-lang.org/`](http://www.groovy-lang.org/)的 Groovy 文档。

# Groovy 脚本中的 Hello World

我们已经讨论了 Groovy 是什么以及它的一些重要特性。让我们创建一个 Hello World 程序，并用 Groovy 感受它的魔力。这里我们假设 Groovy 已安装在系统上，`GROOVY_HOME`指向安装目录，并且`<GROOVY_HOME>/bin`已添加到 PATH 环境变量中：

```java
file: GroovyTest.groovy
  println "Hello Groovy"
```

就这些了。是的，对于一个简单的 Groovy 程序，你不需要声明任何包，任何主类，或者任何分号，只需一个简单的`println`语句就能创建你的第一个 Groovy 程序。

要执行程序，请使用以下命令：

```java
$ groovy GroovyTest.groovy
 Hello Groovy

```

使用`groovy`命令来执行 Groovy 脚本。Groovy 脚本的美妙之处在于它可以执行任何文件，而不仅仅是扩展名为`.groovy`的文件。甚至你可以在`Test.text`文件中编写前面的`println`语句，并使用 groovy 命令来执行该文件。在 Groovy 中，文件扩展名并不重要，但为了使文件结构更易于阅读，建议为 Groovy 文件使用`.groovy`扩展名。

执行 Groovy 文件还有另一种方法。你可以编译 Groovy 文件，生成类似于 Java 的类文件，然后执行这些类文件。请执行以下步骤：

1.  要编译并生成类文件，请使用以下命令：

    ```java
    $ groovyc GroovyTest.groovy

    ```

1.  要在 Windows 上运行生成的类文件，你需要执行以下命令。如果在 Linux/Unix 环境中执行，请使用`$GROOVY_HOME`：

    ```java
    $ java -cp %GROOVY_HOME%/embeddable/groovy-all-2.3.1.jar;. GroovyTest

    ```

执行编译后的 Groovy 文件与执行 Java 文件相同。开发者需要在类路径中添加`groovy-all-<version>.jar`。你还需要确保你的编译类所在的目录在类路径中。在先前的示例中，我们将当前目录"`.`"添加到类路径中，以查找`GroovyTest.class`文件。

执行 Groovy 脚本的方式并不重要。在这两种情况下，Groovy 脚本都在 JVM 内部执行。这两种方法都将 Groovy 脚本编译成字节码。`groovy <filename>`命令直接将类存储在内存中，而使用`groovyc`命令编译脚本会创建一个类文件并将其存储在磁盘上，你可以稍后使用 Java 命令执行它。

# 数据类型

任何编程语言的第一件事就是了解数据类型；任何编程语言如何存储数据。与其它编程语言类似，Groovy 也提供了一套不同的数据类型，用于数值、字符串、字符等。与 Java 相比，Groovy 中没有原始数据类型。Groovy 将一切视为对象，这使得 Groovy 成为一门纯粹的面向对象语言。原始数据类型的问题在于，开发者无法对它们执行任何对象级别的操作，例如调用它们的方法。此外，你也不能将它们作为对象存储在映射和集合（需要对象的集合）中。以下表格显示了原始数据类型和包装类型及其默认值：

| 数据类型 | 包装类型 | 默认值 |
| --- | --- | --- |
| byte | Byte | 0 |
| 短名 | 简称 | 0 |
| int | Integer | 0 |
| long | Long | 0L |
| float | Float | 0.0f |
| double | Double | 0.0d |
| char | Character | \u0000 |
| boolean | Boolean | false |
| String | 不适用 | null |

Groovy 允许你甚至使用 int、byte、short 等声明变量，它内部将它们转换为相应的类，例如，int 转换为 Integer，char 转换为 Character，等等。

## 字符串

你可能想知道，为什么我们只在这里讨论字符串？这是因为与 Java 相比，Groovy 提供了不同的变体来表示字符串，如下面的代码所示：

```java
def s1='This is single quote string.'

def s2="This is double quote string."

def s3="""This is multi line String.
You can write multiple lines here."""

def s4 ="Example of Gstring, You can refer to variable also like ${s1}"

def s5='''This is multi line String.
You can write multiple lines here.'''

def s6 =/ This is 'slashy' String.
It can also contains multiple lines $s1
/
```

在这里，`s1`包含单引号中的字符串。这个字符串的大小是固定的，因为我们这样写它。

`s2`变量包含与 Java 字符串类似的被双引号包围的字符串。

变量`s3`包含被三个双引号包围的字符串，这允许你声明一个多行字符串。

在`s4`中，字符串包含一个嵌入的变量，它将被解析为其值。这正式称为 GString。你可以使用`${variable}`或`$variable`来声明占位符。

Groovy 还支持另一种格式，即字符串在`/`（斜杠）内声明。它也支持多行字符串。

### Groovy 中的动态类型

Groovy 提供了对静态类型和动态类型特性的支持。静态类型在编译时提供更多的检查，更多的内存优化，以及更好的对 Groovy 使用的 IDE 的支持。它还提供了关于变量或方法参数类型的额外信息。然而，Groovy 的力量在于动态类型。在许多场景中，你不确定变量将存储什么类型的值或函数将返回什么类型的值。在这种情况下，Groovy 提供了使用动态类型的使用灵活性。你可以使用`def`关键字定义变量或方法，如下面的代码所示：

```java
def var1
var1 ='a'     
println var1.class   // will print class java.lang.String
var1 = 1       
println var1.class   // will print class java.lang.Integer
def method1() {/*method body*/}
```

动态类型的另一个用途是在没有保证类型的对象上调用方法。这通常被称为**鸭子类型**。例如，考虑以下场景，在整数、列表和字符串等不同数据类型上调用简单的加法方法。根据不同的输入参数，每次方法返回不同的输出。

```java
def addition(a, b) { return a + b}
addition (1, 2)      // Output: 3
addition ([1,2], [4, 5])  // Output: [1, 2, 4, 5]
addition('Hi ', 3)    // Output: Hi 3
```

如你所见，当加法方法以整数作为参数被调用时，它执行了算术加法。对于列表类型的参数，加法方法通过合并两个列表创建一个新的列表。同样，对于字符串类型的参数，它执行简单的连接。在这个例子中，`+`运算符根据输入类型参数被解释为不同的方法调用。

### 提示

Java 和 Groovy 之间一个主要的不同点在于 Groovy 支持操作符重载。

到目前为止一切顺利。但如果在用户定义的对象，比如`Person`上调用加法方法会怎样呢？这将在下面的代码中展示：

```java
class Person{
  String name

  @Override
  public String toString() {
    return "Person [name=" + name +"]";
  }
}

p1 = new Person()
p2 = new Person()
addition(p1, p2)  // Output: groovy.lang.MissingMethodException
```

这是预期的，因为在`Person`类中我们没有定义加法方法。如果我们定义`Person`类中的加法方法，对加法（调用`p1 + p2`或`p1.plus(p2)`）方法的调用将会成功。

另一种解决方案是实现`methodMissing`方法。这是 Groovy 中的一个非常强大的概念。在 Gradle 源代码中，你会多次找到对这个方法的引用。

因此，我们不需要定义一个加法方法，我们可以定义一个`methodMissing`方法，如下所示：

```java
def methodMissing(String name, args) {
  if (name.startsWith("plus") ) {
// write your own implementation
    return "plus method intercepted"
  }
  else {
    println "Method name does not start with plus"
    throw new MissingMethodException(name, this.class, args)
  }
}
```

现在，如果我们调用`Person`对象的加法方法，我们会发现输出为`plus method intercepted`，如下面的代码所示：

```java
addition(p1, p2)  // Output: plus method intercepted
```

# 类、bean 和方法

本节介绍了类、方法和 bean。Groovy 类与使用`class`关键字声明的 Java 类类似。通常，类定义从包名开始，然后是导入包语句。与 Java 对应物的一个关键区别是，Groovy 默认导入六个包和两个类。所以，如果你创建任何类，这些包和类将自动可用：

```java
import java.lang.* // this is the only default import in Java
import java.util.*
import java.io.*
import java.net.*
import groovy.lang.*
import groovy.util.*
import java.math.BigInteger
import java.math.BigDecimal
```

Groovy 中的类和方法默认具有公共访问权限，而在 Java 中它被设置为`package-private`。我们将从一个示例 Groovy 类开始：

```java
class Order {
  int orderNo
  Customer orderedByCustomer
  String description

  static main(args) {
    Order order1 = new Order();
    order1.orderNo = 1;
    order1.orderedByCustomer = new Customer(name: "Customer1", email: "cust1@example.com")
    order1.setDescription("Ordered by Customer1")
    println order1.orderByCustomer.showMail()
  }
}

class Customer{
  String name
  String email
  String address

  String showMail(){
  email
 }
}
```

在这里，我们创建了两个类，`Order`和`Customer`，并包含了一些字段；在`main`方法中，我们创建了对象，最后在`Customer`对象上调用`showMail()`方法。注意对象是如何用值初始化的。`Order`对象是通过默认构造函数创建的，然后对象通过在字段上定义的`setter`方法进行初始化。

然而，对于`Customer`对象，它是通过具有命名参数的构造函数完成的。`Customer`对象通过构造函数中的属性值对进行初始化。然而，我们还没有在类定义中定义任何参数化构造函数。那么它是如何工作的呢？

我们在类中创建了没有访问修饰符的字段。如果字段使用默认访问权限创建，那么 Groovy 会自动创建一个具有公共`getter`和`setter`方法的字段。如果我们指定了任何访问修饰符（`public`、`private`或`protected`），则只会创建字段；不会创建`getter`或`setter`方法。在我们前面的例子中，`orderNo`、`orderByCustomer`和`description`字段没有使用访问修饰符声明。因此，我们能够调用`Order`对象的`setDescription`方法。其他字段通过字段名进行访问。在这种情况下，Groovy 会在字段上内部调用相应的`getter`或`setter`方法。这个特性在 Groovy 中被称为**属性**。因此，Groovy 中的每个类都有属性，并为这些属性自动创建`getter`和`setter`方法。这与 Java bean 方法类似，其中私有字段通过公共`getter`和`setter`方法创建，但代码行数更少，因为`getter`和`setter`方法是由 Groovy 隐式提供的。这就是为什么 Groovy 对象通常被称为**纯旧 Groovy 对象**（**POGO**）。

回到构造函数声明，当使用命名参数创建`Customer`对象时，实际上创建了一个默认构造函数，然后，对于构造函数中的每个属性，分别调用相应的`setter`方法来初始化属性。

Groovy 中的方法与 Java 类似，但类方法的可见性默认设置为 `public`。要调用类上的方法，我们需要创建该类的对象。在 Groovy 脚本中，如果您没有提供任何类定义，方法调用将通过按名称调用方法来完成。如果方法支持动态返回类型，则方法声明应从 `def` 关键字开始。

Groovy 还支持具有默认参数值的方法调用。在以下示例中，`sum` 方法被定义为具有三个参数 `x`、`y` 和 `z`，其中 `y` 和 `z` 的值分别为 `10` 和 `1`。`sum(1)` 和 `sum(1,2)` 方法应分别返回 `12` 和 `4`。

```java
def sum(x,y=10,z=1) {x+y+z}
// x = 1
sum(1)
// x = 1, y= 2
sum(1, 2)
```

Groovy 在方法中不需要显式返回语句。默认情况下，最后一个评估的表达式会被作为方法输出返回。在上面的例子中，我们没有提到 `return x+y+z`。它将默认返回。

# 控制结构

在本节中，我们将讨论基本控制结构，即 `if…else` 语句、`switch` 语句、`for` 循环和 `while` 循环。

## if-else 条件

Groovy 中的 `if…else` 条件与 Java 类似，只有一个例外，那就是 Groovy 如何评估逻辑 `if` 条件。在以下示例中，`if` 条件对布尔值和整数值都评估为真。在 Groovy 中，非零整数、非空值、非空字符串、初始化的集合和有效的匹配器都被评估为布尔值 true。这被称为 **Groovy 真值**。让我们看一下以下代码：

```java
def condition1 = true
int condition2 = 0
if(condition1){
  println("Condition 1 satisfied")
  if(condition2){
    println("Condition 2 satisfied")
  }else{
    println("Condition 2 failed")
  }
}else{
  println("Condition 1 failed")
}
```

Groovy 也支持三元运算符 `(x? y: z)`，类似于 Java，可以用来编写标准的 `if-else` 逻辑：

```java
(condition2> 0 )? println("Positive") : println("Negative")
```

Groovy 还提供了一个称为 Elvis 运算符的额外运算符。它可以用作在用户想要验证变量是否为 null 值的场景中三元运算符的简短版本。考虑以下示例：

```java
def inputName
String username = inputName?:"guest"
```

如果 inputName 不为空，则 username 将会是 inputName，否则将默认值 "guest" 分配给 username

## switch 语句

Groovy 支持 Class、Object、Range、Collection、Pattern 和 Closure 作为 `switch` 语句中的分类器。任何实现了 `isCase` 方法的对象都可以用作 `switch` 语句中的分类器。以下示例展示了为各种分类器定义的案例。只需尝试不同的输入值并观察 `switch` 语句的输出：

```java
def checkInput(def input){
switch(input){    
  case [3, 4, 5]   :   println("Array Matched"); break;
  case 10..15      :   println("Range Matched"); break;
  case Integer     :   println("Integer Matched"); break;
  case ~/\w+/      :   println("Pattern Matched"); break;
  case String      :   println("String Matched"); break;
  default          :   println("Nothing Matched"); break;
}
}
checkInput(3)  // will print Array Matched
checkInput(1)  // will print Integer Matched
checkInput(10)  // will print Range Matched
checkInput("abcd abcd") // will print String Matched
checkInput("abcd")  // will print Pattern Matched
```

## 循环

Groovy 支持两种循环类型：`for (initialize; condition; increment)` 和 `for-each`。`for-each` 风格表示为 `for(variable in Iterable) { body}`。由于循环在可迭代的对象集合上工作，它可以很容易地应用于数组、范围、集合等。让我们看一下以下代码：

```java
// Traditional for loop
for(int i = 0; i< 3; i++) {/* do something */ }
// Loop over a Range
for(i in 1..5) println(i)
// Array iteration
def arr = ["Apple", "Banana", "Mango"]
for(i in arr) println(i)
// for applied on Set
for(i in ([10,10,11,11,12,12] as Set)) println(i)
```

`while` 循环与 Java 的 `while` 循环类似，尽管 Groovy 不支持 `do-while` 风格的循环。让我们演示一下 `while` 循环：

```java
int count = 0
while(count < 5) {
  println count++
}
```

# 集合

我们假设你已经对 **Java 集合框架**（**JCF**）有基本的了解，所以我们不会讨论集合框架的基本原理。我们从 Groovy 提供的集合框架和不同集合对象提供的常用实用方法开始。

Groovy 支持不同的集体数据类型来存储对象组，例如范围、列表、集合和映射。如果你已经是 Java 程序员，你会发现与 Java 相比，在 Groovy 中操作集体数据类型是多么容易。除了集合、列表和映射之外，Groovy 还引入了范围，这在 Java 中是不存在的。

## Set

集合是一个无序的对象集合，没有重复元素。它可以被视为一个具有唯一性限制的无序列表，通常由列表构建而成。集合最多可以包含一个 `null` 元素。正如其名称所暗示的，这个接口模型化了数学集合抽象。

以下代码片段解释了如何创建 `Set`。可以使用 `add`、`addAll`、`remove` 或 `removeAll` 方法向 `Set` 中添加或删除元素。

你可能在数学课上已经学到了很多关于 `Set` 的知识，老师会教你不同的集合操作，例如并集和交集。Groovy 也提供了类似的功能。两个集合的并集包含两个集合中所有独特的元素和共同元素，且不重复。交集找出两个集合之间的共同元素。`Set1` 和 `Set2` 的补集将包含所有在 `Set2` 中不存在的 `Set1` 的元素。

让我们看看以下代码：

```java
// Creating a Set
def Set1 = [1,2,1,4,5,9] as Set
Set Set2 = new HashSet( ['a','b','c','d'] )

// Modifying a Set
Set2.add(1)
Set2.add(9)
Set2.addAll([4,5])        // Set2: [1, d, 4, b, 5, c, a, 9]

Set2.remove(1)        
Set2.removeAll([4,5])    // Set2: [d, b, c, a, 9]

// Union of Set
Set Union = Set1 + Set2     // Union: [1, 2, 4, 5, 9, d, b, c, a]

// Intersection of Set
Set intersection = Set1.intersect(Set2)    // Intersection: [9]

// Complement of Set
Set Complement = Union.minus(Set1)    // Complement: [d, b, c, a]
```

## List

与 `Set` 相比，`List` 是一个有序的对象集合，`List` 可以包含重复元素。可以使用 `List list = []` 创建一个 `List`，这会创建一个空的列表，它是 `java.util.ArrayList` 的一个实现。

以下代码片段展示了如何创建 `List`、从列表中读取值以及列出 `List` 的一些实用方法：

```java
// Creating a List
 def list1 = ['a', 'b', 'c', 'd']
 def list2 = [3, 2, 1, 4, 5] as List

// Reading a List
println list1[1]          // Output: b
println list2.get(4)      // Output: 5
println list1.get(5)      //Throws IndexOutOfBoundsException

// Some utility method on List
//Sort a List
println list2.sort()      // Output: [1, 2, 3, 4, 5]
// Reserve a list
println list1.reverse()      // Output: [d, c, b, a]
// Finding elements
println ("Max:" + list2.max() + ":Last:" + list1.last())   
// Output: Max:5:Last:d
```

一些 `List` 方法接受闭包。以下示例展示了如何使用 `find` 方法找到第一个偶数，以及使用 `findAll` 方法列出所有偶数：

```java
println list2.find({ it %2 == 0})    // Output: 2
println list2.findAll({it %2 == 0})  // Output: [2, 4]
```

不要被花括号内的 "it" 关键字弄混淆。我们将在 *闭包* 部分讨论这个问题。

## Map

Map 是一个键值对集合，其中键是唯一的。在 Groovy 中，键值对由冒号分隔。可以通过 `[:]` 创建一个空的 Map。默认情况下，Map 的类型是 `java.util.HashMap`。如果键是 `String` 类型，你可以在 Map 声明中避免使用单引号或双引号。例如，如果你想创建一个以 `name` 为键、以 `Groovy` 为值的 Map，你可以使用以下表示法：

```java
Map m1 = [name:"Groovy"]
```

在这里，`[name: "Groovy"]` 与 `["name":"Groovy"]` 是相同的。默认情况下，Map 的键是字符串。但如果你想要将某个变量作为键，那么请使用括号，如下面的代码所示：

```java
String s1 = "name"
Map m1 = [(s1):"Groovy"]
```

或者，你可以按以下方式创建一个 Map：

```java
def m2 = [id:1,title: "Mastering Groovy" ] as Map
```

你可以使用`key m2.get("id")`或`m2["id"]`从 Map 中获取对象。

### 小贴士

如果键是字符串，那么要获取值，你需要指定双引号内的键（`""`）。如果你没有指定双引号内的键，它将把它视为变量名并尝试解析它。

现在我们将讨论 Map 的一些实用方法（each、any 和 every），这些方法接受闭包：

```java
Map ageMap = [John:24, Meera:28,Kat:31,Lee:19,Harry:18]
```

要解析`Map`中的每个条目，你可以使用`each`。它接受条目或键值作为参数，如下表所示：

|

```java
ageMap.each {key, value ->
  println "Name is "+key
  println "Age is " + value
}
```

|

```java
ageMap.each {entry  ->
  println "Name is "+entry.key
  println "Age is " + entry.value
}
```

|

如果你想验证`Map`数据，你可以根据需要使用`.every`或`.any`。`.every`方法检查并确保所有记录都满足所述条件，而`.any`只是检查是否有任何记录满足条件。例如，如果你想检查是否有任何用户年龄超过 25 岁：

```java
ageMap.any {entry -> entry.value > 25 }
```

它返回一个布尔值作为输出；在这种情况下，为真，因为 Meera 是 28 岁。

如果你想检查所有用户是否都超过 18 岁：

```java
ageMap.every {entry -> entry.value > 18 }
```

它将返回 false，因为 Harry 是 18 岁。

你也可以使用与我们在*列表*部分中使用`List`相同的模式使用`find`和`findAll`方法为`Map`。

## 范围

除了 Java 集合类型外，Groovy 还支持一种新的集体数据类型`Range`。它定义为两个值（通常是起点和终点），由两个点分隔。

要创建一个`范围`，请使用以下代码：

```java
def range1 = 1..10
Range range2 = 'a'..'e'
```

要从`Range`读取值，请使用以下代码：

```java
range1.each { println it }
```

你也可以使用`.any`和`.every`运算符来验证特定要求的范围。它检查条件并返回一个布尔值。让我们看看以下代码：

```java
range1.any { it > 5 }
range1.every { it > 0 }
```

要修改范围间隔，请使用以下代码。如果你想将范围间隔从默认的 1 修改为任何其他数字，你可以通过 step 方法设置它。它返回一个列表：

```java
List l1 = range1.step(2)    //Output: [1, 3, 5, 7, 9]
```

要获取范围的起始元素和结束元素，请使用`From`和`To`元素，如下所示代码：

```java
range1.getFrom()      //Output: 1
range1.getTo()        //Output: 10
```

`isReverse()`方法用于检查范围趋势，以查看范围是否使用`to value`（较高值）到`from value`（较低值）构建：

```java
range1.isReverse()       // Output: false
```

# 闭包

闭包通常与函数式语言相关联。Groovy 提供了一种非常简单的方法来创建闭包对象。Groovy 闭包就像用花括号编写的代码块。许多人将闭包与 Java 中的匿名函数相关联。

Groovy 中的闭包可以接受参数并返回一个值。默认情况下，Groovy 闭包中的最后一个语句是 `return` 语句。这意味着如果你没有从闭包显式返回任何值，它将默认返回闭包最后一个语句的输出。通常，我们定义闭包如下 `{argument list-> closure body}`。在这里，参数列表是闭包接受的逗号分隔值。参数是可选的。如果没有指定参数，则在闭包体中将有一个隐式的无类型参数 `it` 可用。如果闭包调用时没有提供参数，则 `it` 将为 null。

在下面的例子中，对于闭包 `addTwo` 的第一次调用，分配给变量 `it` 的值是 `2`，但在第二次调用中，`it` 被分配为 null：

```java
def addTwo = {it+2 }
addTwo(2)          // Output: 4
addTwo()          // NullPointerException
```

或者，你甚至可以声明一个类型为闭包的变量。在 Groovy 中，闭包是 `groovy.lang.Closure` 类的子类：

```java
groovy.lang.Closure closure1 = { println it }
closure1("This will be printed") // Output: This will be printed
```

为了将闭包体与参数列表分开，我们使用 `->` 操作符。闭包体由零个或多个 Groovy 语句组成。像方法一样，它也可以在其作用域内引用和声明变量。

在下面的代码片段中，`addOne` 方法能够在其作用域内引用 `constantValue` 变量，尽管它是在闭包作用域外定义的。这样的变量被称为 `free` 变量。在闭包的花括号内定义的变量将被视为局部变量：

```java
int constantValue = 9
def addOne = { Integer a -> constantValue + a }

addOne(1)         // unnamed () invocation. Output: 10
addOne.call(1)    // call() invocation. Output: 10
addOne("One")     // MissingMethodException
```

在前面的例子中，闭包的参数类型是整数类型。使用闭包时，花括号内的语句不会在显式调用它们之前执行，无论是使用 `call()` 方法还是通过闭包的 `unnamed ()` 调用语法。在我们的例子中，闭包在第二行被声明，但当时并没有进行评估。只有在显式地对闭包调用 `call()` 方法时，它才会执行。这是闭包和代码块之间的重要区别。它们看起来可能相同，但实际上并不相同。闭包只有在调用 `call()` 方法时才会执行；不是在其定义时间。记住，闭包在 Groovy 中是一等对象，可以使用无类型变量或使用闭包变量来引用。在这两种情况下，它都源自 `groovy.lang.Closure` 类。这个类有重载的 `call()` 方法，可以没有参数或多个参数来调用闭包。

当 `addOne` 闭包以整数作为参数被调用时，它执行成功。然而，对于字符串类型作为参数，它抛出异常。同时观察，当我们将字符串作为参数传递给 `addOne` 闭包时，编译器并没有抱怨。这是因为所有参数都是在运行时检查的；编译器没有进行静态类型检查。

这个闭包上的 `doCall()` 方法是动态生成的，它只接受 Integer 类型的参数。因此，任何非 Integer 类型的调用都将抛出异常。`doCall()` 方法是隐式方法，不能被重写也不能被重新定义。当我们调用闭包的 call 方法或 `unnamed ()` 语法时，这个方法总是隐式调用。

我们将通过讨论代理的概念来结束对闭包的讨论。这个特性在 Gradle 中被广泛使用。例如，当我们定义构建脚本中的存储库闭包或依赖闭包时，这些闭包将在 `RepositoryHandler` 或 `DependencyHandler` 类中执行。这些类作为代理传递给闭包。您可以参考 Gradle API 获取更多详细信息。在这里，我们不妨简单化问题。我们将通过简单的示例来理解这个概念。

考虑以下示例，我们正在尝试打印一个 `myValue` 变量，该变量在类中未定义。显然，这个调用将抛出异常，因为这个变量在作用域内未定义：

```java
class PrintValue{
  def printClosure = {
    println myValue
  }
}
def pcl = new PrintValue().printClosure
pcl()   //Output: MissingPropertyException: No such property
```

可能存在一种情况，我们想要在另一个类上执行这个闭包。这个类可以作为代理传递给闭包：

```java
class PrintHandler{
  def myValue = "I'm Defined Here"
}

def pcl = new PrintValue().printClosure
pcl.delegate = new PrintHandler()
pcl()

OUTPUT: I'm Defined Here
```

在这个例子中，`PrintHandler` 类已定义了 `myValue` 变量。我们已经委托并针对 `PrintHandler` 类执行了闭包。

到目前为止，它按预期工作。现在，如果 `myValue` 在 `PrintValue` 类中被重新定义：

```java
class PrintValue{
  def myValue = "I'm owner"
  def printClosure = {
    println myValue
  }
}
```

在此场景中，在执行闭包时，我们将找到输出为 `I'm owner`。这是因为，当闭包尝试解析 `myValue` 变量时，它发现该变量定义在所有者的作用域内（定义闭包的 `PrintValue` 类），因此它没有将调用委托给 `PrintHandler` 类。正式来说，这被称为 `OWNER_FIRST` 策略，这是默认策略。策略的解析方式是这样的——首先检查闭包，然后是闭包的作用域，然后是闭包的所有者，最后是代理。Groovy 非常灵活，它为我们提供了更改策略的能力。例如，要将调用委托给 `PrintHandler` 类，我们应该指定策略为 `DELEGATE_FIRST`：

```java
def pcl = new PrintValue().printClosure
pcl.resolveStrategy = Closure.DELEGATE_FIRST
pcl.delegate = new PrintHandler()
pcl()
```

使用 `DELEGATE_FIRST` 策略时，闭包将首先尝试将属性或方法解析到代理，然后是所有者。其他重要的策略包括：

+   `OWNER_ONLY`：它尝试仅在所有者内部解析属性或方法，并不进行代理。

+   `DELEGATE_ONLY`：闭包将解析属性引用或方法到代理。它完全忽略所有者。

+   `TO_SELF`：它将解析属性引用或方法到自身，并经过常规的 `MetaClass` 查找过程。

这确实是一个非常简短的描述。我建议您参考 Groovy 文档以获取更多详细信息，链接如下：[`docs.groovy-lang.org/latest/html/api/groovy/lang/Closure.html`](http://docs.groovy-lang.org/latest/html/api/groovy/lang/Closure.html)。

# 构建器

在 Groovy 中，另一个重要特性是 Builder。Groovy Builders 允许您创建复杂的树状分层对象结构。例如，SwingUI 或 XML 文档可以非常容易地使用 Groovy 中的 DSL 或类似闭包的功能创建，得益于`BuilderSupport`类及其子类`MarkupBuilder`和`SwingBuilder`的支持。

让我们通过一个示例来尝试理解。我们在本章的早期创建了`Order`类。假设我们有一个订单列表，我们想要将详细信息存储在名为`orders.xml`的文件中。因此，我们列表中的每个`Order`对象都应该作为 XML 文件中的一个节点保存。这些`Order`节点再次应该包含子节点、孙子节点，依此类推。如果我们尝试在 Java 中实现类似 DOM 的解析器，创建这种树状结构可能会很复杂：

```java
<orders>
  <order>
    <no>1</no>
    <description>Ordered by customer 1</description>
    <customer>
      <name firstname='Customer1' />
      <email>cust1@example.com</email>
    </customer>
  </order>
  <order>
    <no>2</no>
    <description>Ordered by customer 2</description>
    <customer>
      <name firstname='Customer2' />
      <email>cust2@example.com</email>
    </customer>
  </order>
  ….
</orders>
```

但在 Groovy 中，这只是一些代码行和一些方法调用，结合闭包和命名参数。在下面的示例中，我们从`MarkupBuilder`类创建了一个`builder`对象来创建 XML 文档。然后我们定义`orders`为文档的根。然而，`builder`没有定义名为`orders`的方法。那么这是怎么工作的呢？

如前所述，`MarkupBuilder`类是`BuilderSupport`类的子类。`BuilderSupport`具有`createNode`、`invokeMethod`、`nodeCompleted`、`setCurrent`、`setParent`等方法。在运行时，通过在名为`orders`的构建器上调用`createNode`方法创建一个对象。以类似的方式，对于每个`order`对象，创建`no`、`description`和`customer`节点。最后，通过调用构建器对象的`setParent`方法将每个`order`节点附加到父`orders`节点：

```java
def builder = new groovy.xml.MarkupBuilder(new FileWriter("orders.xml"))

  builder.orders{
    for(i in orderlist){
      order{
        no(i.orderNo)
        description(i.description)
        customer{
          name(firstname : i.orderedBy.name)
          email(i.orderedBy.email)
        }
      }
    }
  }
```

# 摘要

在本章中，我们讨论了一些基本的基本概念。我们学习了类、方法、bean、集合框架和闭包的概念。我们还开发了一个标记构建器来生成 XML 文件。这确实是对 Groovy 的非常简短的介绍。然而，在我看来，这个介绍应该足够编写您项目的 Gradle 脚本。

从下一章开始，我们将开始探索 Gradle 的核心功能。在下一章中，我们将学习 Groovy 中的任务管理。我们将仔细研究 Gradle 支持的不同内置任务。我们还将了解任务依赖和任务配置。然后我们将为构建脚本创建一些自定义任务。
