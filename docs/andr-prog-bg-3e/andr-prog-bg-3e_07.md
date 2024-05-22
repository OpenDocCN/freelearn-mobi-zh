# 第七章：*第七章*：Java 变量、运算符和表达式

在本章和下一章中，我们将学习和实践 Java 数据的核心基础知识以及如何操作这些数据。在本章中，我们将专注于创建和理解数据本身，在下一章中，我们将看到如何操作和响应它。

我们还将快速回顾一下我们在前几章学到的关于 Java 的知识，然后深入学习如何编写我们自己的 Java 代码。我们即将学习的原则不仅适用于 Java，还适用于其他编程语言。

通过本章结束时，你将能够舒适地编写 Java 代码，在 Android 中创建和使用数据。本章将带你了解以下主题：

+   理解 Java 语法和行话

+   使用变量存储和使用数据

+   使用变量

+   使用运算符更改变量中的值

+   尝试表达式

让我们学习一些 Java。

# 技术要求

你可以在 GitHub 上找到本章的代码文件，网址为[`github.com/PacktPublishing/Android-Programming-for-Beginners-Third-Edition/tree/main/chapter%2007`](https://github.com/PacktPublishing/Android-Programming-for-Beginners-Third-Edition/tree/main/chapter%2007)。

# Java 无处不在

我们即将学习的核心 Java 基础知识适用于我们从中继承的类（比如`AppCompatActivity`），以及我们自己编写的类（正如我们将在*第十章*，*面向对象编程*中开始做的那样）。

在我们编写自己的类之前学习基础知识更合乎逻辑，我们将使用扩展的`Activity`类`AppCompatActivity`，在一个迷你项目中添加一些 Java 代码。我们将再次使用`Log`和`Toast`类，以在`Activity`类的重写`onCreate`方法中看到我们编码的结果，以触发我们代码的执行。

然而，当我们转到*第十章*，*面向对象编程*并开始编写我们自己的类，以及更多地了解他人编写的类是如何工作的，我们在这里学到的一切也将适用于那个时候——事实上，你在本章和下一章学到的所有 Java 知识，如果你将它从`Activity`类中剥离出来，粘贴到另一个 Java 环境中，比如以下环境：

+   任何主要的桌面操作系统

+   许多现代电视

+   卫星导航

+   智能冰箱

Java 也可以在那里运行！

呼唤所有的 Java 大师

如果你已经做过一些 Java 编程并理解以下关键字（`if`、`else`、`while`、`do while`、`switch`和`for`），你可能可以直接跳到*第十章*，*面向对象编程*。或者，你可能想浏览一下这些信息作为复习。

让我们继续学习如何在 Java 中编码。

# 理解 Java 语法和行话

在整本书中，我们将使用简单的英语来讨论一些技术问题。你永远不会被要求阅读一个以非技术语言解释的 Java 或 Android 概念的技术解释。

到目前为止，在一些场合我已经要求你接受一个简化的解释，以便在更合适的时候提供更充分的解释，就像我在类和方法中所做的那样。

话虽如此，Java 和 Android 社区充满了讲技术术语的人，要加入并从这些社区中学习，你需要理解他们使用的术语。因此，本书的方法是使用完全通俗的语言学习一个概念或欣赏一个想法，但同时作为学习的一部分介绍术语/技术术语。

Java 语法是我们将 Java 语言元素组合在一起，以在 Dalvik 虚拟机（VM）中产生可工作的代码的方式。Java 语法是我们使用的单词和将这些单词组成类似句子的结构的组合，构成我们的代码。

这些 Java“单词”数量众多，但分成小块学习肯定比任何人类语言更容易。我们称这些单词为**关键字**。

我相信如果您能阅读，那么您就能学会 Java，因为学习 Java 要容易得多。那么，是什么区分了完成初级 Java 课程的人和专业程序员呢？

这正是区分语言学生和大师诗人的相同之处。精通 Java 并不在于我们知道如何使用多少个 Java 关键字，而在于我们如何使用它们。语言的精通来自于实践、进一步学习，并更加熟练地使用关键字。许多人认为编程与科学一样是一门艺术，这也有一定道理。

## 更多代码注释

随着您在编写 Java 程序方面变得更加高级，您用来创建程序的解决方案将变得更长、更复杂。此外，正如我们将在后面的章节中看到的，Java 被设计为通过让我们将代码分成单独的类，往往跨越多个文件来管理复杂性。

**代码注释**是 Java 程序中没有任何功能的部分。编译器会忽略这些注释。它们用于帮助程序员记录、解释和澄清他们的代码，使其对自己以后或其他可能需要使用或更改代码的程序员更易理解。

我们已经看到了单行注释。这里重复一下：

```kt
// this is a comment explaining what is going on
```

前面的注释以两个`//`斜杠字符开头。注释在行末结束。因此，该行上的任何内容仅供人类阅读，而下一行上的任何内容（除非是另一个注释）都需要是语法正确的 Java 代码，如下所示：

```kt
// I can write anything I like here
but this line will cause an error
```

我们可以使用多个单行注释，如下所示：

```kt
// Below is an important note
// I am an important note
// We can have as many single line comments like this as we like
```

单行注释也很有用，如果我们想暂时禁用一行代码。我们可以在代码前面加上`//`，这样它就不会包含在程序中。记住这行代码，告诉 Android 加载我们的布局？

```kt
// setContentView(R.layout.activity_main);
```

在前面的情况下，当运行时布局将不会被加载，应用程序将显示空白屏幕，因为编译器会忽略整行代码。

重要提示

我们在*第五章**,* *使用 CardView 和 ScrollView 创建美观的布局*中看到了这一点，当我们暂时注释掉了其中一个方法。

Java 中还有另一种类型的注释，称为**多行注释**。多行注释适用于跨越多行的较长注释，以及在代码文件顶部添加版权信息等内容。与单行注释一样，多行注释可以用于临时禁用代码，通常跨越多行。

在`/*`和`*/`之间的所有内容都会被编译器忽略。以下是一些示例：

```kt
/*
   You can tell I am good at this because my
   code has so many helpful comments in it.
*/
```

多行注释中的行数没有限制；使用哪种类型的注释将取决于情况。在本书中，我将始终在文本中明确解释每行代码，但您通常会在代码本身中发现大量的注释，这些注释会进一步解释、洞察或提供上下文。因此，彻底阅读所有代码也是一个好主意。

```kt
/*
   The winning lottery numbers for next Saturday are
   9,7,12,34,29,22
   But you still want to make Android apps?
*/
```

提示

所有优秀的 Java 程序员都会在他们的代码中大量使用注释！

# 使用变量存储和使用数据

我们可以想象一个`variableA`。这些名称就像是我们程序员对用户 Android 设备内存的窗口。

变量是内存中的值，通过使用它们的名称可以随时使用或更改。

计算机内存具有高度复杂的寻址系统，幸运的是我们不需要与之交互。Java 变量允许我们为程序需要处理的所有数据制定自己方便的名称。**Dalvik VM**（**DVM**）将处理与操作系统的交互的所有技术细节，操作系统将再与物理内存交互。

因此，我们可以将我们的 Android 设备的内存想象成一个巨大的仓库，只等着我们添加我们的变量。当我们为变量分配名称时，它们被存储在仓库中，准备在我们需要它们时使用。当我们使用变量的名称时，设备知道我们指的是什么。然后我们可以告诉它做这样的事情：

+   为`variableA`分配一个值

+   将`variableA`添加到`variableB`

+   测试`variableB`的值，并根据结果采取行动

+   ……等等，我们很快就会看到

在一个典型的应用程序中，我们可能会有一个名为`unreadMessages`的变量，用来保存用户未读消息的数量。当有新消息到达时，我们可以增加它，当用户阅读消息时，我们可以减少它，并在应用程序布局的某个地方向用户显示它，以便他们知道有多少未读消息。

以下是可能出现的一些情况：

+   用户收到三条新消息，因此将`3`添加到`unreadMessages`的值。

+   用户登录应用程序，因此使用`Toast`显示一条消息以及存储在`unreadMessages`中的值。

+   用户看到一堆消息来自他们不喜欢的人，并删除了六条消息。然后我们可以从`unreadMessages`中减去 6。

这些是变量名称的任意示例，如果您没有使用 Java 限制的任何字符或关键字，实际上可以随意命名变量。

然而，在实践中，最好采用**命名约定**，以使您的变量名保持一致。在本书中，我们将使用一种松散的变量命名约定，以小写字母开头。当变量名中有多个单词时，第二个单词将以大写字母开头。这称为**驼峰命名法**。

以下是一些示例：

+   `unreadMessages`

+   `contactName`

+   `isFriend`

在查看一些带有变量的真实 Java 代码之前，我们需要首先看一下我们可以创建和使用的变量的类型。

## 变量的类型

可以想象，即使是一个简单的应用程序也会有相当多的变量。在上一节中，我们介绍了`unreadMessages`变量作为一个假设的例子。如果一个应用程序有一个联系人列表，并需要记住每个联系人的名字，那么我们可能需要为每个联系人创建变量。

当应用程序需要知道联系人是否也是朋友还是普通联系人时怎么办？我们可能需要编写代码来测试朋友状态，然后将该联系人的消息添加到适当的文件夹中，以便用户知道这些消息是来自朋友还是其他人。

计算机程序中另一个常见的要求，包括 Android 应用程序，是真或假的错误。

为了涵盖您可能想要存储或操作的各种数据类型，Java 有**类型**。

### 基本类型

有许多类型的变量，我们甚至可以发明自己的类型。但是现在，我们将看一下最常用的内置 Java 类型，公平地说，它们几乎涵盖了我们可能会遇到的每种情况。提供一些示例是解释类型的最佳方式。

我们已经讨论了假设的`unreadMessages`变量。这个变量当然是一个数字，因此我们必须告诉 Java 编译器这一点，给它一个适当的类型。

另一方面，假设的`contactName`变量当然将保存组成联系人姓名的字符。

保存常规数字的类型称为`int`类型，比如`unreadMessages`，如果我们尝试将其他类型的数据存储在这样的变量中，我们肯定会遇到麻烦，正如我们从以下截图中可以看到的：

![](img/Figure_7.01_B16773.jpg)

图 7.1 – 存储联系人姓名

正如我们所看到的，Java 被设计成不可能让这些错误进入运行中的程序。

以下是 Java 中的主要变量类型：

+   `整数`：`整数`类型用于存储整数，即整数。这种类型使用 32 位（**位**）内存，因此可以存储略大于 20 亿的值，包括负值。

+   `长整型`：正如名称所暗示的，当需要更大的数字时，可以使用`长整型`数据类型。`长整型`类型使用 64 位内存，我们可以在其中存储 2 的 63 次方。如果您想看看它是什么样子，这就是它：9,223,372,036,854,775,807。也许令人惊讶的是，长整型变量有用之处，但关键是，如果较小的变量可以胜任，我们应该使用它，因为我们的程序将使用更少的内存。

重要提示

您可能想知道何时会使用这些大小的数字。明显的例子可能是进行复杂计算的数学或科学应用，但另一个用途可能是用于计时。当您计算某事花费的时间时，Java `Date`类使用自 1970 年 1 月 1 日以来的毫秒数。毫秒是一秒的千分之一，所以自 1970 年以来已经有相当多的毫秒了。

+   `浮点数`：这是用于浮点数的数据类型——也就是说，小数点后有精度的数字。由于数字的小数部分占用的内存空间与整数部分一样多，因此与非浮点数相比，`float`类型中数字的范围会减少。因此，除非我们的变量需要额外的精度，否则`float`不会是我们的数据类型选择。

+   `双精度`：当`float`类型的精度不够时，我们有`double`。

+   `布尔`：我们将在整本书中使用大量布尔值。`布尔`变量类型可以是`true`或`false`；没有其他值。布尔值回答以下问题：

*联系人是朋友吗？*

*有新消息吗？*

*布尔值的两个例子足够吗？*

+   `字符`：在`字符`类型中存储单个字母数字字符。它本身不会改变世界，但如果我们将它们放在一起，它可能会有用。

+   `short`：这种类型类似于`int`的节省空间的版本。它可以用来存储具有正负值的整数，并且可以对其进行数学运算。它与`int`的区别在于它只使用 16 位内存，这只是与`int`相比的内存量的一半。`short`的缺点是它只能存储与`int`相比一半范围的值，从-32768 到 32767。

+   `字节`：这种类型类似于`short`的更节省空间的版本。它可以用来存储具有正负值的整数，并且可以对其进行数学运算。它与`int`和`short`的区别在于它只使用 8 位内存，这只是与`byte`相比的内存量的一半，与`int`相比只有四分之一的内存。`byte`的缺点是它只能存储与`int`相比一半范围的值，从-32768 到 32767。总共节省 8 或 16 位的内存是不太可能有影响的；但是，如果您需要在程序中存储数百万个整数，那么`short`和`byte`是值得考虑的。

重要提示

我将这个关于数据类型的讨论保持在一个实用的水平上，这对本书的内容是有用的。如果您对数据类型的值是如何存储以及为什么限制是什么感兴趣，那么请查看*Oracle Java 教程*网站 http://docs.oracle.com/javase/tutorial/java/nutsandbolts/datatypes.html。请注意，您不需要比我们已经讨论过的更多信息来继续阅读本书。

正如我们刚刚学到的，我们可能想要存储的每种数据类型都需要特定数量的内存。因此，我们必须在开始使用变量之前让 Java 编译器知道变量的类型。

先前描述的变量称为**原始**类型。大多数原始类型在不同的编程语言中都被使用（以及关键字），因此，如果您对类型和关键字有很好的理解，那么跳到另一种语言将比第一次容易得多！这些类型使用预定义的内存量，因此，使用我们的仓库存储类比，适合预定义大小的存储盒。

正如“原始”标签所示，它们不像**引用**类型那样复杂。

### 引用类型

您可能已经注意到，我们没有涵盖我们之前用来介绍保存字母数字数据的变量类型`String`。

#### 字符串

字符串是特殊类型变量的一个例子，称为**引用**类型。它们简单地指向内存中存储变量的位置，但引用类型本身并不定义特定的内存量。其原因很简单：因为我们并不总是知道在程序运行之前需要存储多少数据。

我们可以将字符串和其他引用类型看作是不断扩展和收缩的存储盒。那么，这些`String`引用类型中的一个不会最终碰到另一个变量吗？

当我们将设备的内存视为一个充满标记存储盒的巨大仓库时，您可以将 DVM 视为一个超级高效的叉车司机，将不同类型的存储盒放在最合适的位置；如果有必要，DVM 将在几分之一秒内迅速移动物品，以避免碰撞。此外，如果需要，Dalvik，这个叉车司机，甚至会立即蒸发掉任何不需要的存储盒。

所有这些都发生在不断卸载各种类型的新存储盒并将它们放在最适合该类型变量的地方的同时。Dalvik 将引用变量保存在仓库的不同部分，我们将在*第十二章*，*堆栈、堆和垃圾收集器*中了解更多细节。

字符串可以用来存储任何键盘字符，就像`char`类型，但长度几乎可以是任意的。从联系人的姓名到整本书都可以存储在单个`String`类型中。我们将经常使用字符串，包括在本章中。

还有一些其他引用类型我们也会探讨。

#### 数组

数组是一种存储大量相同类型变量的方法，以便快速有效地访问。我们将在*第十五章*，*数组、映射和随机数*中研究数组。

将数组想象成我们仓库中的一排通道，所有特定类型的变量都按照精确的顺序排列。数组是引用类型，因此 Dalvik 将它们保存在与字符串相同的仓库部分。例如，我们可以使用数组来存储数十个联系人。

#### 类

另一种引用类型是`class`类型，我们已经讨论过但没有正确解释。我们将在*第十章*，*面向对象编程*中熟悉类。

现在，我们知道我们可能想要存储的每种数据类型都需要一定的内存。因此，在我们开始使用变量之前，我们必须让 Java 编译器知道变量的类型。我们用变量**声明**来做到这一点。

# 使用变量

这就够理论了。让我们看看我们如何使用我们的变量和类型。请记住，每种原始类型都需要特定数量的真实设备内存。这就是为什么编译器需要知道变量的类型的原因之一。

## 变量声明

我们必须首先使用名称`unreadMessages`声明`int`，我们将输入以下内容：

```kt
int unreadMessages;
```

就是这样——简单地声明类型（在本例中是`int`），然后留出一个空格，输入你想要用于此变量的名称。还要注意，行末的分号`;`将告诉编译器我们已经完成了这一行，接下来的内容（如果有的话）不是变量声明的一部分。

同样地，对于几乎所有其他变量类型，声明方式都是相同的。以下是一些示例。示例中的变量名是任意的。这就像在仓库中预留一个带标签的储物箱。

看一下以下代码片段：

```kt
long millisecondsElapsed;
float accountBalance;
boolean isFriend;
char contactFirstInitial;
String messageText;
```

请注意，我说的是*几乎所有其他变量类型*。其中一个例外是`class`类型的变量。我们已经看到了一些声明`class`类型变量的代码。您还记得*第三章*中的这个代码片段吗？在`MainActivity.java`文件中，*探索 Android Studio 和项目结构*？

```kt
FloatingActionButton fab…
```

这段编辑过的代码片段声明了一个名为`fab`的`FloatingActionButton`类型的变量。但我们有点跑题，将在*第十章* *面向对象编程*中回到类。

## 变量初始化

**初始化**是下一步。在这里，对于每种类型，我们将一个值初始化到变量中。这就像在仓库的储物箱中放入一个值。

```kt
unreadMessages = 10;
millisecondsElapsed = 1438165116841l;// 29th July 2016 11:19am
accountBalance = 129.52f;
isFriend = true;
contactFirstInitial = 'C';
messageText = "Hi reader, I just thought I would let you know that Charles Babbage was an early computing pioneer and he invented the difference engine. If you want to know more about him, you can click find look here: www.charlesbabbage.net";
```

请注意，`char`变量在初始化值周围使用`'`单引号，而`String`类型使用`"`双引号。

我们也可以将声明和初始化步骤合并。在这里，我们声明并初始化了与之前相同的变量，但是在一步中完成：

```kt
int unreadMessages = 10;
long millisecondsElapsed = 1438165116841l;//29th July 2016 11:19am
float accountBalance = 129.52f;
boolean isFriend = true;
char contactFirstInitial = 'C';
String messageText = " Hi reader, I just thought I would let you know that Charles Babbage was an early computing pioneer and he invented the difference engine. If you want to know more about him, you can click this link www.charlesbabbage.net";
```

无论我们是分开声明和初始化，还是一起进行，都取决于具体情况。重要的是我们必须在某个时候都要做这两件事。

```kt
int a;
// That's me declared and ready to go!
// The line below attempts to output a to the console
Log.i("info", "int a = " + a);
// Oh no I forgot to initialize a!!
```

这将导致以下情况：

**编译器错误：变量 a 可能尚未初始化**

这个规则有一个重要的例外。在某些情况下，变量可以有**默认值**。我们将在*第十章* *面向对象编程*中看到这一点；但是，声明和初始化变量是一个好习惯。

# 使用运算符更改变量中的值

当然，在几乎任何程序中，我们都需要对这些变量的值进行“操作”。我们使用**运算符**来操作（更改）变量。以下是一些最常见的 Java 运算符列表，它们允许我们操作变量。您不需要记住它们，因为我们将在第一次使用它们时逐行查看每行代码。我们已经在初始化变量时看到了第一个运算符，但是我们将再次看到它，这次会更加有趣。

## 赋值运算符

这是赋值运算符：`=` 

它使运算符左侧的变量与右侧的值相同——例如，就像这行代码中的那样：

```kt
unreadMessages = newMessages;
```

在执行了前一行代码之后，存储在`unreadMessages`中的值将与`newMessages`中的值相同。

## 加法运算符

这是加法运算符：`+`

它将两侧的值相加，通常与赋值运算符一起使用。例如，它可以将两个具有数值的变量相加，就像下一行代码中的那样：

```kt
 unreadMessages = newMessages + unreadMessages; 
```

一旦前面的代码执行了，`newMessages`和`unreadMessages`持有的值的组合值现在存储在`unreadMessages`中。作为同样事情的另一个例子，请看这行代码：

```kt
accountBalance = yesterdaysBalance + todaysDeposits; 
```

重要提示

请注意，同时在运算符的两侧同时使用同一个变量是完全可以接受的。

## 减法运算符

这是减法运算符：`-`

它将从左侧的值中减去右侧的值。通常与赋值运算符一起使用，就像这个代码示例中：

```kt
unreadMessages = unreadMessages - 1; 
```

或者，作为一个类似的例子，它在这行代码中使用：

```kt
accountBalance = accountBalance - withdrawals;
```

在上一行代码执行后，`accountBalance`将保留其原始值减去`withdrawals`中的值。

## 除法运算符

这是除法运算符：`/`

它将把左边的数字除以右边的数字。同样，它通常与赋值运算符一起使用。以下是一个示例代码行：

```kt
fairShare = numSweets / numChildren;
```

如果在上一行代码中`numSweets`持有九个糖果，`numChildren`持有三个糖果，那么`fairShare`现在将持有三个糖果的价值。

## 乘法运算符

这是乘法运算符：`*`

它将变量和数字相乘，与许多其他运算符一样，通常与赋值运算符一起使用。例如，看看下一行代码：

```kt
answer = 10 * 10; 
```

或者，看看这行代码：

```kt
biggerAnswer = 10 * 10 * 10;
```

在前两行代码执行后，`answer`保持值 100，`biggerAnswer`保持值 1000。

## 递增运算符

这是递增运算符：`++`

递增运算符是从某物中加一的快速方法。例如，看看下一行代码，它使用了加法运算符：

```kt
myVariable = myVariable + 1; 
```

上一行代码的结果与这里更紧凑的代码相同：

```kt
   myVariable ++; 
```

## 递减运算符

这是递减运算符：`--`

递减运算符（正如你可能猜到的）是从某物中减去一个的快速方法。例如，看看下一行代码，它使用了减法运算符：

```kt
myVariable = myVariable -1; 
```

上一行代码与`myVariable --;`相同。

重要提示

这些运算符的正式名称与之前解释的略有不同，例如，除法运算符是乘法运算符之一。但是之前给出的名称对于学习 Java 来说更有用，如果你在与 Java 社区的某人交谈时使用术语*除法运算符*，他们会完全明白你的意思。

Java 中甚至有更多的运算符。在下一章中，当我们学习在 Java 中做出决定时，我们将遇到其中一些。

重要提示

如果你对运算符感到好奇，在*Oracle Java 教程*网站上有一个完整的运算符列表，网址为[`docs.oracle.com/javase/tutorial/java/nutsandbolts/operators.html`](http://docs.oracle.com/javase/tutorial/java/nutsandbolts/operators.html)。本书中完成项目所需的所有运算符都将在本书中得到充分解释。链接是为我们中更好奇的人提供的。

# 尝试表达

让我们尝试使用一些声明、赋值和运算符。当我们将这些元素捆绑成一些有意义的语法时，我们称之为`Toast`并`Log`以检查我们的结果。

## 表达自己的演示应用程序

创建一个名为`Expressing Yourself`的新项目，使用下载包的`/Expressing Yourself`文件夹。

切换到`onCreate`方法，就在`}`闭合大括号之前，添加这段代码：

```kt
int numMessages;
```

在上一行代码的下方，我们将初始化一个值为`numMessages`。

接下来，添加这行代码：

```kt
numMessages = 10;
```

在上一行代码之后，`onCreate`的`}`闭合大括号之前，添加以下代码：

```kt
// Output the value of numMessages
Log.i("numMessages = ", "" + numMessages);
numMessages++;
numMessages = numMessages + 1;
Log.i("numMessages = ", "" + numMessages);
// Now a boolean (just true or false)
boolean isFriend = true;
Log.i("isFriend = ", "" + isFriend);
// A contact and an important message
String contact = "James Gosling";
String message = "Dear reader, I invented Java.";
// Now let's play with those String variables
Toast.makeText(this, "Message from " + contact, Toast.LENGTH_SHORT).show();
Toast.makeText(this, "Message is: " + message, Toast.LENGTH_SHORT).show();
```

重要提示

你需要导入`Toast`和`Log`类，就像我们之前做的那样。

运行应用程序，我们可以检查输出，然后再检查代码。在 logcat 窗口中，你会看到以下输出：

```kt
numMessages =: 10
numMessages =: 12
isFriend =: true
```

在屏幕上，你会看到两个弹出的`Toast`消息。第一个说**来自詹姆斯·高斯林的消息。**第二个说**消息是：亲爱的读者，我发明了 Java。**这在下面的截图中显示：

![图 7.2 - 第二个弹出的 Toast 消息](img/Figure_7.02_B16773.jpg)

图 7.2 - 第二个弹出的 Toast 消息

让我们逐行检查代码，确保每行都清晰明了，然后再继续。

首先，我们声明并初始化了一个名为`numMessages`的`int`类型变量。我们本可以在一行上完成，但我们是这样做的：

```kt
int numMessages;
numMessages = 10;
```

接下来，我们使用`Log`输出一条消息。这次，我们不是简单地在`""`双引号之间输入消息，而是使用`+`运算符将`numMessages`添加到输出中，正如我们在控制台中看到的，`numMessages`的实际值被输出，如下所示：

```kt
// Output the value of numMessages
Log.i("numMessages = ", "" + numMessages);
```

为了进一步证明我们的`numMessages`变量像它应该的那样多才多艺，我们使用了`++`运算符，这应该将它的值增加`1`，然后使用`+ 1`将`numMessages`加到自身上。然后我们输出了`numMessages`的新值，并确实发现它的值从 10 增加到 12，如下面的代码片段所示：

```kt
numMessages ++;
numMessages = numMessages + 1;
Log.i("numMessages = ", "" + numMessages);
```

接下来，我们创建了一个名为`isFriend`的`boolean`类型变量，并将其输出到控制台。我们从输出中看到`true`被显示。当我们在下一节中看到决策制定时，这种变量类型将充分证明其有用性。代码如下所示：

```kt
// Now a boolean (just true or false)
boolean isFriend = true;
Log.i("isFriend = ", "" + isFriend);
```

在此之后，我们声明并初始化了两个`String`类型的变量，如下所示：

```kt
// A contact and an important message
String contact = "James Gosling";
String message = "Dear reader, I invented Java.";
```

最后，我们使用`Toast`输出`String`变量。我们使用了`"Message from "`消息的硬编码部分，并使用`+ contact`添加了消息的变量部分。我们也使用了相同的技术来形成第二个`Toast`消息。

提示

当我们将两个字符串连接在一起以形成一个更长的`String`类型时，这被称为**连接**。

```kt
// Now let's play with those String variables
Toast.makeText(this, "Message from " + contact, Toast.LENGTH_SHORT).show();
Toast.makeText(this, "Message is:" + message, Toast.LENGTH_SHORT).show();
```

现在，我们可以声明变量，将它们初始化为一个值，稍微改变它们的值，并使用`Toast`或`Log`输出它们。

# 总结

最后，我们使用了一些严肃的 Java。我们学习了关于变量、声明和初始化。我们看到了如何使用运算符来改变变量的值。如果你不记得所有的东西，没关系，因为我们将在整本书中不断地使用这些技术和关键字。

在下一章中，让我们看看如何根据这些变量的值做出决定，以及这对我们有多有用。
