# 第九章：*第九章*：学习 Java 方法

随着我们开始逐渐熟悉 Java 编程，在本章中，我们将更仔细地研究方法，因为尽管我们知道可以**调用**它们来执行它们的代码，但它们比我们迄今讨论的更多。

在本章中，我们将研究以下内容：

+   方法结构

+   方法重载与覆盖

+   一个方法演示迷你应用程序

+   方法如何影响我们的变量

+   方法递归

首先，让我们快速回顾一下方法。

# 技术要求

您可以在 GitHub 上找到本章中的代码文件[`github.com/PacktPublishing/Android-Programming-for-Beginners-Third-Edition/tree/main/chapter%2009`](https://github.com/PacktPublishing/Android-Programming-for-Beginners-Third-Edition/tree/main/chapter%2009)。

# 方法重温

这张图总结了我们对方法的理解：

![图 9.1 – 理解方法](img/Figure_9.1_B16773.jpg)

图 9.1 – 理解方法

正如我们在图中所看到的，关于方法还有一些疑问。我们将彻底揭开方法的面纱，看看它们是如何工作的，以及方法的其他部分到底在做什么，这将在本章后面讨论。在*第十章**，面向对象编程*和*第十一章**，更多面向对象编程*中，我们将在讨论面向对象编程时澄清方法的最后几个部分的神秘之处。

## 究竟什么是 Java 方法？

方法是一组变量、表达式和控制流语句，它们被捆绑在一个以**签名**为前缀的大括号内。我们已经使用了很多方法，但我们还没有仔细地看过它们。

让我们从方法结构开始。

# 方法结构

我们编写的方法的第一部分称为签名。这是一个假设的方法签名：

```kt
public boolean addContact(boolean isFriend, string name)
```

如果我们添加一对大括号`{}`，并加入一些方法执行的代码，那么我们就有了一个完整的方法——一个**定义**。这是另一个虚构的但在语法上是正确的方法：

```kt
private void setCoordinates(int x, int y){
   // code to set coordinates goes here
}
```

正如我们所见，我们可以从代码的其他部分使用我们的新方法，如下所示：

```kt
// I like it here
setCoordinates(4,6);// now I am going off to setCoordinates method
// Phew, I'm back again - code continues here
```

在我们调用`setCoordinates`方法的时候，我们程序的执行将分支到该方法内部包含的代码。该方法将逐步执行其中的所有语句，直到达到结束，然后将控制返回给调用它的代码，或者如果它遇到`return`语句，则更早地返回。然后，代码将从方法调用后的第一行继续运行。

这是另一个完整的方法示例，包括使方法返回到调用它的代码的代码：

```kt
int addAToB(int a, int b){
   int answer = a + b;
   return answer;
}
```

使用前面方法的调用可能如下所示：

```kt
int myAnswer = addAToB(2, 4); 
```

显然，我们不需要编写方法来将两个`int`变量相加，但这个例子帮助我们更深入地了解方法的工作原理。首先，我们传入值`2`和`4`。在方法签名中，值`2`被赋给`int a`，值`4`被赋给`int b`。

在方法体内，变量`a`和`b`被相加，并用于初始化新变量`int answer`。行`return answer`将存储在`answer`中的值返回给调用代码，导致`myAnswer`被初始化为值`6`。

请注意，前面示例中的每个方法签名都有所不同。这是因为 Java 方法签名非常灵活，允许我们精确地构建我们需要的方法。

方法签名确切地定义了方法如何被调用以及方法如何返回值，这值得进一步讨论。

让我们给签名的每个部分起一个名字，这样我们就可以把它分成几部分，并了解这些部分。

以下加粗的文本是一个方法签名，其部分已经标记好以便讨论。另外，看一下接下来的表格，以进一步澄清签名的哪一部分是哪一部分。这将使我们对方法的讨论更加简单明了：

**修饰符** | **返回类型** | **方法名称（参数）**

![](img/B16773_09_Table_1.jpg)

## 修饰符

在我们之前的例子中，我们只在一些例子中使用了修饰符，部分原因是方法不必使用修饰符。修饰符是一种指定代码可以使用（调用）你的方法的方式，通过使用`public`和`private`等修饰符。

变量也可以有修饰符，如下所示：

```kt
// Most code can see me
public int a;
// Code in other classes can't see me
private String secret = "Shhh! I am private";
```

修饰符（用于方法和变量）是一个重要的 Java 主题，但最好在我们讨论其他重要的 Java 主题时处理，这些主题我们已经在几次中绕过了 - 类。我们将在下一章中这样做。

## 返回类型

表中的下一个是返回类型。像修饰符一样，返回类型是可选的。所以，让我们仔细看一下。我们已经看到我们的方法可以做任何我们可以用 Java 编码的事情。但是如果我们需要方法所做的结果呢？到目前为止，我们所见过的返回类型的最简单的例子如下：

```kt
int addAToB(int a, int b){
   int answer = a + b;
   return answer;
}
```

在这里，签名中的返回类型被突出显示。返回类型是`int`。`addAToB`方法将一个值返回给调用它的代码，这个值将适合在一个`int`变量中。

返回类型可以是我们到目前为止所见过的任何 Java 类型。然而，方法不一定要返回一个值。当方法不返回值时，签名必须使用`void`关键字作为返回类型。当使用`void`关键字时，方法体不得尝试返回一个值，否则会导致编译错误。但是，它可以使用没有值的`return`关键字。

以下是一些返回类型和使用`return`关键字的有效组合：

```kt
void doSomething(){
   // our code
   // I'm done going back to calling code here
   // no return is necessary
}
```

另一个组合如下：

```kt
void doSomethingElse(){
   // our code
   // I can do this as long as I don't try and add a value
   return;
}
```

以下代码是另一个组合：

```kt
void doYetAnotherThing(){
   // some code
   if(someCondition){

         // if someCondition is true returning to calling 
         code 
         // before the end of the method body
         return;
   }

   // More code that might or might not get executed

   return;
   /* 
         As I'm at the bottom of the method body 
         and the return type is void, I'm 
         really not necessary but I suppose I make it 
         clear that the method is over.
   */
}
String joinTogether(String firstName, String lastName){
   return firstName + lastName;
}
```

我们可以依次调用前面的每个方法，如下所示：

```kt
// OK time to call some methods
doSomething();
doSomethingElse();
doYetAnotherThing();
String fullName = joinTogether("Alan ","Turing")
// fullName now = Alan Turing
// continue with code from here
```

前面的代码将依次执行每个方法中的所有代码。

## 方法的名称

表中的下一个是方法的名称。当我们设计自己的方法时，方法名是任意的。但是惯例是使用清楚解释方法将要做什么的动词。此外，使用名称的第一个单词的第一个字母小写，后面的单词的第一个字母大写的约定。这被称为驼峰命名法，就像我们在学习变量名时学到的那样。考虑下一个例子：

```kt
void XGHHY78802c(){
   // code here
}
```

前面的方法是完全合法的，并且可以工作；然而，让我们看看使用这些约定的三个更清晰的例子：

```kt
void doSomeVerySpecificTask(){
   // code here
}
void getMyFriendsList(){
   // code here
}
void startNewMessage(){
   // code here
} 
```

这样更清晰，因为名称明确表明了方法将要做什么。

让我们来看看方法中的参数。

## 参数

表中的最后一个主题是**参数**。我们知道方法可以将结果返回给调用代码。但是如果我们需要从调用代码中*向*方法*共享*一些数据值呢？

参数允许我们将值发送到被调用的方法中。当我们查看返回类型时，我们已经看到了一个带有参数的例子。我们将看同样的例子，但更仔细地看一下参数：

```kt
int addAToB(int a, int b){
   int answer = a + b;
   return answer;
}
```

在这里，参数被突出显示。参数包含在括号中，`(参数放在这里)`，紧跟在方法名之后。

请注意，在方法体的第一行中，我们使用`a + b`，就好像它们已经被声明和初始化为变量一样。那是因为它们是。方法签名的参数是它们的声明，调用方法的代码初始化它们，就像下一行代码中突出显示的那样。

```kt
int returnedAnswer = addAToB(10,5);
```

另外，正如我们在之前的例子中部分看到的，我们不仅仅在参数中使用`int`。我们可以使用任何 Java 类型，包括我们自己设计的类型。

此外，我们也可以混合和匹配类型。我们还可以使用尽可能多的参数来解决我们的问题。一个例子可能会有所帮助。

```kt
void addToAddressBook(char firstInitial, 
String lastName, 
String city, 
int age){

   /*
         all the parameters are now living, breathing,
         declared and initialized variables.

         The code to add details to address book goes here.
   */
}
```

前面的例子将有四个声明和初始化的变量，准备好使用。

现在我们将看看方法主体——放在方法内部的内容。

## 主体

在我们之前的例子中，我们一直在伪代码中使用注释来描述我们的方法主体，比如以下的注释：

```kt
// code here
```

还有以下的用法：

```kt
// some code
```

`addToAddressBook`方法也被使用了。

```kt
/*
             all the parameters are now living, breathing,
             declared and initialized variables.

             The code to add details to address book goes 
             here.
      */
```

但我们已经完全知道主体中要做的事情。到目前为止，我们学到的任何 Java 语法都可以在方法的主体中使用。事实上，如果我们回想一下，到目前为止我们在本书中编写的所有代码都*已经*在一个方法中。

我们接下来可以做的最好的事情是编写一些在主体中执行操作的方法。

# 使用方法演示应用程序

在这里，我们将快速构建两个应用程序，以进一步探索方法。首先，我们将使用`Real World Methods`应用程序探索基础知识，然后我们将一窥新主题，`Exploring Method Overloading`应用程序。

像往常一样，您可以以通常的方式打开已经输入的代码文件。下面的两个方法示例可以在*第九章*文件夹和`Real World Methods`和`Exploring Method Overloading`子文件夹中的下载包中找到。

## 真实世界的方法

首先，让我们创建一些简单的工作方法，包括返回类型参数和完全运作的主体。

要开始，创建一个名为`Real World Methods`的新 Android 项目，使用`MainActivity.java`文件，通过在编辑器上方的**MainActivity.java**标签上单击左键，我们可以开始编码。

首先，将这三个方法添加到`MainActivity`类中。将它们添加到`onCreate`方法的闭合大括号`}`后面。

```kt
String joinThese(String a, String b, String c){
   return a + b + c;
}
float getAreaCircle(float radius){
   return 3.141f * radius * radius;
}
void changeA(int a){
   a++;
}
```

我们添加的第一个方法叫做`joinThese`。它将返回一个`String`值，并需要传入三个`String`变量。在方法主体中，只有一行代码。`return a + b + c`代码将连接传入的三个字符串，并将连接后的字符串作为结果返回。

下一个方法名为`getAreaCircle`，接受一个`float`变量作为参数，然后也返回一个`float`变量。方法的主体简单地使用了圆的面积公式，结合传入的半径，然后将答案返回给调用代码。`3.141`末尾的奇怪的`f`是为了让编译器知道这个数字是`float`类型的。任何浮点数都被假定为`double`类型，除非它有尾随的`f`。

第三个和最后一个方法是所有方法中最简单的。请注意，它不返回任何东西；它有一个`void`返回类型。我们包括了这个方法，以明确一个我们想要记住关于方法的重要观点。但在我们讨论它之前，让我们看看它的实际操作。

现在，在`onCreate`方法中，在调用`setContentView`方法之后，添加这段代码，调用我们的三个新方法，然后在 logcat 窗口中输出一些文本：

```kt
String joinedString = joinThese("Methods ", "are ", "cool ");
Log.i("joinedString = ","" + joinedString);
float area  = getAreaCircle(5f);
Log.i("area = ","" + area);
int a = 0;
changeA(a);
Log.i("a = ","" + a);
```

运行应用程序，查看 logcat 窗口中的输出，这里为您提供方便：

```kt
joinedString =: Methods are cool
area =: 78.525
a =: 0
```

在 logcat 输出中，我们可以看到的第一件事是`joinedString`字符串的值。正如预期的那样，它是我们传入`joinThese`方法的三个单词的连接。

接下来，我们可以看到`getAreaCircle`确实计算并返回了基于传入半径的圆的面积。

`a`变量即使在传入`changeA`方法后仍保持值`0`的事实，值得单独讨论。

### 发现变量范围

输出的最后一行最有趣：`a=: 0`。在`onCreate`方法中，我们声明并初始化了`int a`为`0`，然后调用了`changeA`方法。在`changeA`的主体中，我们用代码`a++`增加了`a`。然而，在`onCreate`方法中，当我们使用`Log.i`方法将`a`的值打印到 logcat 窗口时，它仍然是**0**。

因此，当我们将`a`传递给`changeA`方法时，实际上传递的是存储在`a`中的*值*，而不是*实际变量*`a`。这在 Java 中被称为按值传递。

提示

当我们在一个方法中声明一个变量时，它只能在该方法中被*看到*。当我们在另一个方法中声明一个变量时，即使它具有完全相同的名称，它也*不是*同一个变量。变量只在声明它的方法内部具有**作用域**。

对于所有基本变量，将它们传递给方法的工作方式是这样的。对于引用变量，它的工作方式略有不同，我们将在下一章中看到。

重要说明

我已经和一些刚接触 Java 的人谈过这个作用域概念。对一些人来说，这似乎是显而易见的，甚至是自然的。然而，对于其他人来说，这是一个持续困惑的原因。如果你属于后一种情况，不要担心，因为我们将在本章稍后再谈一些关于这个问题的内容，而在未来的章节中，我们将更深入地探讨作用域，并确保它不再是一个问题。

让我们看一个关于方法的另一个实际例子，并同时学到一些新东西。

# 探索方法重载

随着我们开始意识到，方法作为一个主题是相当深入的。但希望通过一步一步地学习，我们会发现它们并不令人畏惧。我们也将在下一章回到方法。现在，让我们创建一个新项目来探索**方法重载**的主题。

创建一个新的`探索方法重载`，然后我们将继续编写三个方法，但稍微有些不同。

正如我们将很快看到的，我们可以创建多个具有相同名称的方法，只要参数不同。这个项目中的代码很简单。它的工作方式可能看起来有点奇怪，直到我们分析它之后。

在第一个方法中，我们将简单地称之为`printStuff`，并通过参数传递一个`int`变量进行打印。

将此方法插入在`onCreate`方法的`}`之后，但在`MainActivity`类的`}`之前。记得以通常的方式导入`Log`类：

```kt
void printStuff(int myInt){
   Log.i("info", "This is the int only version");
   Log.i("info", "myInt = "+ myInt);
}
```

在第二个方法中，我们还将称之为`printStuff`，但传入一个`String`变量进行打印。将此方法插入在`onCreate`方法的`}`之后，但在`MainActivity`类的`}`之前：

```kt
void printStuff(String myString){
   Log.i("info", "This is the String only version");
   Log.i("info", "myString = "+ myString);
}
```

在这第三个方法中，我们将再次称之为`printStuff`，但传入一个`String`变量和一个`int`值进行打印。将此方法插入在`onCreate`的`}`之后，但在`MainActivity`类的`}`之前：

```kt
void printStuff(int myInt, String myString){
   Log.i("info", "This is the combined int and String 
   version");
   Log.i("info", "myInt = "+ myInt);
   Log.i("info", "myString = "+ myString);
}
```

最后，在`onCreate`方法的`}`之前插入这段代码，以调用方法并将一些值打印到 logcat 窗口：

```kt
// Declare and initialize a String and an int
int anInt = 10;
String aString = "I am a string";

// Now call the different versions of printStuff
// The name stays the same, only the parameters vary
printStuff(anInt);
printStuff(aString);
printStuff(anInt, aString);
```

现在我们可以在模拟器或真实设备上运行应用程序。这是输出：

```kt
Info: This is the int only version
Info: myInt = 10
Info: This is the String only version
Info: myString = I am a string
Info: This is the combined int and String version
Info: myInt = 10
Info: myString = I am a string
```

正如你所看到的，Java 将具有相同名称的三个方法视为不同的方法。正如我们刚刚展示的那样，这是有用的。这被称为**方法重载**。

方法重载和覆盖的混淆

**重载**是指当我们有多个具有相同名称但不同参数的方法时。

**覆盖**是指用相同的名称和参数列表替换一个方法。

我们对重载和覆盖已经了解足够，可以完成这本书；但如果你勇敢而且思绪飘忽，是的，你可以覆盖一个重载的方法，但这是另一个时间的事情。

这就是它的工作原理。在我们编写代码的每个步骤中，我们创建了一个名为`printStuff`的方法。但是每个`printStuff`方法都有不同的参数，因此实际上是可以单独调用的不同方法：

```kt
void printStuff(int myInt){
   ...
}
void printStuff(String myString){
   ...
}
void printStuff(int myInt, String myString){
   ...
}
```

每个方法的主体都是微不足道的，只是打印出传入的参数，并确认调用的方法版本。

我们代码的下一个重要部分是当我们明确指出要调用的方法版本，使用与签名中参数匹配的特定参数。在最后一步，我们依次调用每个方法，使用匹配的参数，这样 Java 就知道需要调用的确切方法：

```kt
printStuff(anInt);
printStuff(aString);
printStuff(anInt, aString);
```

现在我们已经知道关于方法的所有需要知道的东西，让我们快速再看一下方法和变量之间的关系。然后，我们会更深入地了解这个作用域现象。

# 重新访问作用域和变量

你可能还记得在`真实世界方法`项目中，稍微令人不安的异常是，一个方法中的变量似乎与另一个方法中的变量不同，即使它们有相同的名称。如果你在一个方法中声明一个变量，无论是生命周期方法还是我们自己的方法，它只能在该方法内使用。

如果我们在`onCreate`中这样做是没有用的：

```kt
int a = 0;
```

然后，在`onPause`或其他方法中，我们尝试这样做：

```kt
a++;
```

我们会得到一个错误，因为`a`只在它声明的方法中可见。起初，这可能看起来像是一个问题，但令人惊讶的是，这实际上是 Java 的一个特别有用的特性。

我已经提到用来描述这一点的术语是**作用域**。当变量可用时，就说它在作用域内，当不可用时，就说它不在作用域内。作用域的主题最好与类一起讨论，我们将在*第十章**，面向对象编程*和*第十一章**，更多面向对象编程*中这样做，但是作为对未来的一瞥，你可能想知道一个类可以有自己的变量，当它有时，它们对整个类都有作用域；也就是说，所有的方法都可以“看到”并使用它们。我们称它们为**成员**变量或**字段**。

要声明一个成员变量，只需在类的开始之后使用通常的语法，在类中声明的任何方法之外。假设我们的应用程序像这样开始：

```kt
public class MainActivity extends AppCompatActivity {

   int mSomeVariable = 0;
   // Rest of code and methods follow as usual
   // ...
```

我们可以在这个类的任何方法中使用`mSomeVariable`。我们的新变量`mSomeVariable`只是为了提醒我们它是一个**成员**变量，所以在变量名中加上了`m`。这不是编译器要求的，但这是一个有用的约定。

在我们继续讲解类之前，让我们再看一个方法的主题。

# 方法递归

方法递归是指一个方法调用自身。这乍一看可能像是一个错误，但实际上是解决一些编程问题的有效技术。

这里有一些代码展示了一个递归方法的最基本形式：

```kt
void recursiveMethod() {
     recursiveMethod();
}
```

如果我们调用`recursiveMethod`方法，它的唯一代码行将调用自身，然后再调用自身，然后再调用自身，依此类推。这个过程将永远持续下去，直到应用程序崩溃，在 Logcat 中会出现以下错误：

```kt
java.lang.StackOverflowError: stack size 8192KB
```

当方法被调用时，它的指令被移动到处理器的一个区域，称为堆栈，当它返回时，它的指令被移除。如果方法从不返回，而是不断添加更多的指令副本，最终堆栈将耗尽内存（或溢出），我们会得到`StackOverflowError`。

我们可以尝试使用下一个截图来可视化前四个方法调用。此外，在下一个截图中，我划掉了对方法的调用，以显示如果我们能够在某个时刻阻止方法调用，最终所有的方法都将返回并从堆栈中清除：

![图 9.2 - 方法调用](img/Figure_9.2_B16773.jpg)

图 9.2 - 方法调用

为了使我们的递归方法有价值，我们需要增强两个方面。我们将很快看到第二个方面。首先，最明显的是，我们需要给它一个目的。我们可以让我们的递归方法求和（相加）从 0 到给定目标值（比如 10、100 或更多）范围内的数字的值。让我们通过给它这个新目的并相应地重命名它来修改前面的方法。我们还将添加一个具有类范围（在方法之外）的变量`answer`：

```kt
int answer = 0;
void computeSum(int target) {
answer += target;
     computeSum(target-1);
}
```

现在我们有一个名为`computeSum`的方法，它以一个`int`作为参数。如果我们想要计算 0 到 10 之间所有数字的总和，我们可以这样调用该方法：

```kt
computeSum(10);
```

以下是每个函数调用时`answer`变量的值：

第一次调用`computeSum`：`answer` = 10

第二次调用`computeSum`：`answer` = 19

…

第十次调用`computeSum`：`answer` = 55

表面上看成功 - 直到你意识到该方法在`target`变量达到 0 之后仍然继续调用自身。事实上，我们仍然面临着第一个递归方法的相同问题，经过数万次方法调用后，应用程序将再次崩溃并出现`StackOverflowError`。

我们需要一种方法来阻止方法在`target`等于 0 时继续调用自身。我们解决这个问题的方法是检查`target`的值是否为 0，如果是，我们就停止调用该方法。看看下面显示的额外突出显示的代码：

```kt
void computeSum(int target) {
     answer += target;
     if(target > 0) {
          Log.d("target = ", "" + target);
          computeSum(target - 1);
     }
     Log.d("answer = ", "" + answer);
```

我们使用`if`语句来检查目标变量是否大于 0。当方法被最后一次调用时，我们还有额外的`Log.d`代码来输出`answer`的值。在阅读输出后的解释之前，看看你能否弄清楚发生了什么。

调用`computeSum(10)`的输出如下：

```kt
target =: 10
target =: 9
target =: 8
target =: 7
target =: 6
target =: 5
target =: 4
target =: 3
target =: 2
target =: 1
answer =: 55
```

`if(target > 0)`告诉代码首先检查`target`变量是否大于 0。如果是，然后才调用方法并传入`target - 1`的值。如果不是，那么它就停止整个过程。

重要提示

我们不会在本书中使用方法递归，但这是一个有趣的概念需要理解。

我们对方法了解得足够多，可以完成书中的所有项目。让我们通过一些问题和答案进行一个快速回顾。

# 问题

1.  这个方法定义有什么问题？

```kt
doSomething(){
   // Do something here
}
```

没有声明返回类型。你不必从方法中返回一个值，但在这种情况下它的返回类型必须是`void`。方法应该如下所示：

```kt
void doSomething(){
   // Do something here
}
```

1.  这个方法定义有什么问题？

```kt
float getBalance(){
   String customerName = "Linus Torvalds";
   float balance = 429.66f;
   return userName;
}
```

该方法返回一个字符串(`userName`)，但签名规定它必须返回一个`float`类型。以`getBalance`这样的方法名，这段代码可能是原本想要的：

```kt
float getBalance(){
   String customerName = "Linus Torvalds";
   float balance = 429.66f;
   return balance;
}
```

1.  我们什么时候调用`onCreate`方法？（提示：这是一个诡计问题！）

我们不需要。Android 决定何时调用`onCreate`方法，以及构成 Activity 生命周期的所有其他方法。我们只覆盖对我们有用的方法。但是，我们会调用`super.onCreate`，以便我们的重写版本和原始版本都被执行。

重要提示

为了完全披露，从我们的代码中技术上可以调用生命周期方法，但在本书的上下文中我们永远不需要这样做。最好将这些事情留给 Android。

# 总结

在前五章中，我们对各种小部件和其他 UI 元素变得相当熟练。我们还构建了广泛的 UI 布局选择。在本章和前三章中，我们已经相当深入地探索了 Java 和 Android 活动生命周期，尤其是考虑到我们完成得多么快。

我们在一定程度上创建了 Java 代码和 UI 之间的交互。我们通过设置`onClick`属性调用了我们的方法，并使用`setContentView`方法加载了我们的 UI 布局。然而，我们并没有真正建立 UI 和 Java 代码之间的适当连接。

我们现在真正需要做的是将这些东西结合起来，这样我们就可以开始使用 Android UI 来显示和操作我们的数据。为了实现这一点，我们需要更多地了解类的知识。

自从《第一章》《开始 Android 和 Java》以来，类一直潜伏在我们的代码中，我们甚至有点使用它们。然而，直到现在，除了不断地参考《第十章》《面向对象编程》之外，我们还没有适当地解决它们。在下一章《第十章》《面向对象编程》中，我们将快速掌握类的知识，然后我们终于可以开始构建应用程序，使 UI 设计和我们的 Java 代码完美地协同工作。

# 进一步阅读

我们已经学到了足够的 Java 知识来继续阅读本书。然而，看到更多 Java 实例并超越最低必要的知识总是有益的。如果你想要一个学习 Java 更深入的好资源，那么官方的 Oracle 网站是一个不错的选择。请注意，您不需要学习这个网站来继续阅读本书。另外，请注意，Oracle 网站上的教程并不是在 Android 环境中设置的。该网站是一个有用的资源，可以收藏并浏览：

+   官方 Java 教程：[`docs.oracle.com/javase/tutorial/`](https://docs.oracle.com/javase/tutorial/)

+   官方 Android 开发者网站：[`developer.android.com/training/basics/firstapp`](https://developer.android.com/training/basics/firstapp)
