# 第八章 Kotlin 决策和循环

我们刚刚学会了变量，并且现在了解如何使用表达式更改它们所持有的值，但是我们如何根据变量的值采取行动呢？

我们当然可以将新消息的数量添加到先前未读消息的数量中，但是例如，当用户已读完所有消息时，我们如何在应用程序内触发一个操作呢？

第一个问题是我们需要一种方法来测试变量的值，然后在值落在一系列值范围内或等于特定值时做出响应。

编程中常见的另一个问题是，我们需要根据变量的值来执行代码的某些部分一定次数（多次或有时根本不执行）。

为了解决第一个问题，我们将学习使用 `if`、`else` 和 `when` 在 Kotlin 中做决策。为了解决后者，我们将学习使用 `while`、`do` – `while`、`for`、`continue` 和 `break` 在 Kotlin 中做循环。

此外，我们将了解到，在 Kotlin 中，决策也是产生值的表达式。在本章中，我们将涵盖以下主题：

+   使用 `if`、`else`、`else` – `if` 和 `switch` 进行决策

+   `when` 演示应用程序

+   Kotlin `while` 循环和 `do` - `while` 循环

+   Kotlin `for` 循环

现在让我们更多地了解 Kotlin。

# 在 Kotlin 中做决策

我们的 Kotlin 代码将不断做出决策。例如，我们可能需要知道用户是否有新消息，或者他们是否有一定数量的朋友。我们需要能够测试我们的变量，看它们是否满足某些条件，然后根据它们是否满足条件来执行特定的代码部分。

在本节中，随着我们的代码变得更加深入，以一种更易读的方式呈现代码有助于使其更易读。让我们看一下代码缩进，以使我们关于做决策的讨论更加容易。

## 为了清晰起见缩进代码

您可能已经注意到我们项目中的 Kotlin 代码是缩进的。例如，在 `MainActivity` 类内的第一行代码被缩进了一个制表符。此外，每个函数内的第一行代码也被另一个制表符缩进；下面是一个带注释的图表，以便清楚地说明这一点：

![为了清晰起见缩进代码](img/B12806_08_01.jpg)

请注意，当缩进块结束时，通常是以一个闭合大括号（`}`）结束，它的缩进程度与开始块的代码行相同。

我们这样做是为了使代码更易读。然而，这并不是 Kotlin 语法的一部分，如果我们不去做这个，代码仍然会编译。

随着我们的代码变得更加复杂，缩进和注释有助于保持代码的含义和结构清晰。我现在提到这一点是因为当我们开始学习 Kotlin 中做决策的语法时，缩进变得特别有用，建议您以相同的方式缩进代码。大部分缩进是由 Android Studio 为我们完成的，但并非全部。

现在我们知道如何更清晰地呈现我们的代码，我们可以学习更多运算符，然后开始使用 Kotlin 做决策。

## 更多 Kotlin 运算符

我们已经学会了使用运算符进行添加（+）、减去（-）、乘以（*）、除以（/）、赋值（=）、递增（++）和递减（--）。现在我们将探索一些更有用的运算符，然后我们将直接学习如何使用它们。

### 提示

不要担心记住以下每个运算符。简要浏览它们和它们的解释，然后继续下一节。在那里，我们将使用一些运算符，并且当我们看到它们允许我们做什么的示例时，它们将变得更加清晰。它们在此处以列表的形式呈现，只是为了从一开始就清晰地展示运算符的种类和范围。在后面关于实现的讨论中，这个列表也会更方便以后参考。

我们使用运算符创建一个表达式，这个表达式要么为真，要么为假。我们用括号或方括号括起这个表达式，就像这样：`(表达式放在这里)`。

### 比较运算符

这是比较运算符。它测试相等性，要么为真，要么为假；它是`Boolean`类型的：

```kt
==
```

例如，表达式`(10 == 9)`是假的。10 显然不等于 9。然而，表达式`(2 + 2 == 4)`显然是真的。

### 提示

也就是说，除了在《1984》中，2 + 2 == 5（[`en.wikipedia.org/wiki/Nineteen_Eighty-Four`](https://en.wikipedia.org/wiki/Nineteen_Eighty-Four)）。

### 逻辑 NOT 运算符

这是逻辑 NOT 运算符：

```kt
!
```

它用于测试表达式的否定。如果表达式为假，那么 NOT 运算符会使表达式为真。

例如，表达式`(!(2+2 == 5))`为真，因为 2 + 2 *不是* 5。但是，`(!(2 + 2 = 4))`的进一步例子是假的。这是因为 2 + 2 *显然是* 4。

### 不等于运算符

这是不等于运算符，它是另一个比较运算符：

```kt
!=
```

不等于运算符测试是否不相等；例如，`(10 != 9)`表达式为真，因为 10 不等于 9。另一方面，`(10 != 10)`为假，因为 10 等于 10。

### 大于运算符

另一个比较运算符（还有一些其他的）是大于运算符：

```kt
>
```

这个运算符测试一个值是否大于另一个值。表达式`(10 > 9)`为真，但是表达式`(9 > 10)`为假。

### 小于运算符

你可能猜到了，这个运算符测试一个值是否小于另一个值；这是这个运算符的样子：

```kt
<
```

表达式`(10 < 9)`为假，因为 10 不小于 9，而表达式`(9 < 10)`为真。

### 大于或等于运算符

这个运算符测试一个值是否大于或等于另一个值，如果其中一个为真，结果就为真。这就是这个运算符的样子：

```kt
>=
```

例如，表达式`(10 >= 9)`为真，表达式`(10 >= 10)`也为真，但是表达式`(10 >= 11)`为假，因为 10 既不大于也不等于 11。

### 小于或等于运算符

像前一个运算符一样，这个运算符测试两个条件，但这次是**小于**或等于；看看下面的运算符：

```kt
<=
```

表达式`(10 <= 9)`为假，表达式`(10 <= 10)`为真，表达式`(10 <= 11)`也为真。

### 逻辑 AND 运算符

这个运算符称为逻辑 AND。它测试表达式的两个或多个独立部分，整个表达式的两个或所有部分都必须为真才能为真：

```kt
&&
```

逻辑 AND 通常与其他运算符一起使用，以构建更复杂的测试。表达式`((10 > 9) && (10 < 11))`为真，因为两个部分都为真。另一方面，表达式`((10 > 9) && (10 < 9))`为假，因为表达式的一个部分为真-`(10 > 9)`，而另一个部分为假-`(10 < 9)`。

### 逻辑 OR 运算符

这个运算符叫做逻辑 OR，它和逻辑 AND 一样，只是表达式的两个或多个部分中只有一个为真，整个表达式才为真：

```kt
||
```

再看一下我们用于逻辑 AND 的上一个例子，但是，用`||`替换`&&`。表达式`((10 > 9) || (10 < 9))`现在为真，因为表达式的一个或多个部分需要为真。

在本章和整本书的其余部分中，以更实际的情境看到这些运算符，将有助于澄清它们的不同用途。现在我们知道如何使用运算符、变量和值来形成表达式。接下来，我们可以看一种结构化和组合表达式的方法，以做出几个深刻的决定。

## 如何使用所有这些运算符来测试变量

所有这些运算符在没有正确使用它们来做出影响真实变量和代码的真实决定的方法时几乎是无用的。

现在我们已经拥有了所有需要的信息，我们可以看一个假设的情况，然后实际检查一些决策的代码。

### 使用 if 表达式

正如您所见，运算符本身的作用很小，但看到我们可以使用的广泛和多样的范围的一部分是很有用的。现在当我们开始使用最常见的运算符`==`时，我们可以开始看到它们为我们提供的强大而精细的控制。

让我们通过检查以下代码来使之前的示例不那么抽象。

```kt
val time = 9

val amOrPm = if(time < 12) {
  "am"
} else {
  "pm"
}

Log.i("It is ", amOrPm)
```

上述代码首先声明并初始化了一个名为`time`的`Int`类型，其值为`9`。代码的下一行非常有趣，因为它做了两件事。`if(time < 12)`表达式是一个测试；我们知道时间小于`12`，因为我们刚刚将其初始化为`9`。由于条件为真，`if`表达式返回`"am"`值，并且在`if`表达式之前的代码行的第一部分声明并初始化了一个名为`amOrPm`的新`String`类型，并赋予了该值。

如果我们将`time`变量初始化为不少于 12 的任何值（即 12 或更高），则从`else`块中返回的值将是`"pm"`。如果您将上述代码复制并粘贴到项目中，例如`onCreate`函数，logcat 中的输出将如下所示：

```kt
It is: am

```

`if`表达式被评估，如果条件为真，则执行第一组花括号中的代码（`{…}`）；如果条件为假，则执行`else {…}`块中的代码。

值得注意的是，`if`不一定要返回一个值，而是可以根据测试的结果执行一些代码；看一下以下示例代码：

```kt
val time = 13

if(time < 12) {
  // Execute some important morning task here
} else {
  // Do afternoon work here
}
```

在上述代码中，没有返回值；我们只关心正确的代码部分是否被执行。

### 提示

从技术上讲，仍然返回一个值（在这种情况下为 true 或 false），但我们选择不对其进行任何操作。

此外，我们的`if`表达式可以处理超过两个结果，我们稍后会看到。

我们还可以在 String 模板中使用`if`。我们在上一章中看到，我们可以通过在`$`符号后的花括号之间插入表达式来将表达式插入到`String`类型中。以下是上一章的代码提醒：

```kt
Log.i("info", "$name 
was born in $yearOfBirth and is $age years old. 
Next year he will be ${currentYear - yearOfBirth} years old)")
```

在上述代码中的突出部分将导致从`currentYear`中减去`yearOfBirth`的值被打印在消息的其余部分中。

以下代码示例显示了我们如何以相同的方式将整个`if`表达式插入到`String`模板中：

```kt
val weight = 30
val instruction = 
  "Put bag in ${if (weight >= 25) "hold" else "cabin" }"

Log.i("instruction is ", instruction)
```

上述代码使用`if`来测试`weight`变量是否初始化为大于或等于 25 的值。根据表达式是否为真，它将单词`hold`或单词`cabin`添加到`String`初始化中。

如果您执行上述代码，您将获得以下输出：

```kt
instruction is: Put this bag in the hold

```

如果您将`weight`的初始化更改为 25 以下的任何值并执行代码，您将获得以下输出：

```kt
instruction is: Put this bag in the cabin

```

让我们看一个更复杂的例子。

## 如果他们过桥，就射击他们！

在下一个示例中，我们将使用`if`，一些条件运算符和一个简短的故事来演示它们的用法。

船长快要死了，知道他剩下的下属经验不是很丰富，他决定写一个 Kotlin 程序（还能干什么？）在他死后传达他的最后命令。部队必须守住桥的一侧，等待增援，但有一些规则来决定他们的行动。

船长想要确保他的部队理解的第一个命令如下：

**如果他们过桥，就射击他们。**

那么，我们如何在 Kotlin 中模拟这种情况呢？我们需要一个`Boolean`变量-`isComingOverBridge`。下一部分代码假设`isComingOverBridge`变量已经被声明并初始化为`true`或`false`。

然后我们可以这样使用`if`：

```kt
if(isComingOverBridge){

   // Shoot them

}
```

如果`isComingOverBridge`布尔值为 true，则大括号内的代码将执行。如果`isComingOverBridge`为 false，则程序在`if`块之后继续执行，而不运行其中的代码。

### 否则，做这个代替

船长还想告诉他的部队，如果敌人不从桥上过来，他们应该待在原地等待。

为此，我们可以使用`else`。当我们希望在`if`表达式不为 true 时明确执行某些操作时，我们使用`else`。

例如，如果敌人不从桥上过来，我们可以编写以下代码告诉部队待在原地：

```kt
if(isComingOverBridge){

   // Shoot them

}else{

   // Hold position

}
```

然后船长意识到问题并不像他最初想的那么简单。如果敌人从桥上过来，但是部队太多怎么办？他的小队将被压制和屠杀。

因此，他提出了以下代码（这次，我们也将使用一些变量）：

```kt
var isComingOverBridge: Boolean
var enemyTroops: Int
var friendlyTroops: Int

// Code that initializes the above variables one way or another

// Now the if
if(isComingOverBridge && friendlyTroops > enemyTroops){

   // shoot them

}else if(isComingOveBridge && friendlyTroops < enemyTroops) {

   // blow the bridge

}else{

   // Hold position

}
```

上述代码有三条可能的执行路径。第一种情况是，如果敌人从桥上过来，友军数量更多：

```kt
if(isComingOverBridge && friendlyTroops > enemyTroops)
```

第二种情况是，如果敌军正在从桥上过来，但数量超过友军：

```kt
else if(isComingOveBridge && friendlyTroops < enemyTroops)
```

如果其他两条路径都不成立，第三种可能的结果是由最终的`else`语句捕获的，没有`if`条件。

### 提示

**读者挑战**

您能发现上述代码的一个缺陷吗？这可能会让一群经验不足的部队陷入完全混乱的状态？敌军和友军的数量恰好相等的可能性没有得到明确处理，因此将由最终的`else`语句处理，这是用于没有敌军的情况。任何自尊的船长都希望他的部队在这种情况下战斗，他可以改变第一个`if`语句以适应这种可能性，如下所示：

`if(isComingOverBridge && friendlyTroops >= enemyTroops)`

最后，船长最后关心的是，如果敌人拿着白旗投降并立即被屠杀，那么他的士兵将成为战争罪犯。这里需要的代码是显而易见的；使用`wavingWhiteFlag`布尔变量，他可以编写以下测试：

```kt
if (wavingWhiteFlag){

   // Take prisoners

}
```

然而，放置这段代码的位置不太清楚。最后，船长选择了以下嵌套解决方案，并将`wavingWhiteFlag`的测试更改为逻辑非，如下所示：

```kt
if (!wavingWhiteFlag){

   // not surrendering so check everything else

   if(isComingOverBridge && friendlyTroops >= enemyTroops){

          // shoot them
   }else if(isComingOverBridge && friendlyTroops < 
                enemyTroops) {

         // blow the bridge

   }

}else{

   // this is the else for our first if
   // Take prisoners

}

// Holding position
```

这表明我们可以嵌套`if`和`else`语句以创建深入和详细的决策。

我们可以继续使用`if`和`else`做出更多更复杂的决定，但是我们在这里看到的已经足够作为介绍了。

很可能值得指出的是，很多时候，解决问题有多种方法。*正确*的方法通常是以最清晰和最简单的方式解决问题。

现在我们将看一些其他在 Kotlin 中做决策的方法，然后我们可以将它们全部放在一个应用程序中。

## 使用`when`进行决策

我们已经看到了将 Kotlin 运算符与`if`和`else`语句结合使用的广泛且几乎无限的可能性。但是，有时，在 Kotlin 中做出决策可能有其他更好的方法。

当我们希望根据一系列可能的结果做出决策并执行不同的代码段时，我们可以使用`when`。以下代码声明并初始化`rating`变量，然后根据`rating`的值向 logcat 窗口输出不同的响应：

```kt
val rating:Int = 4
when (rating) {
  1 -> Log.i("Oh dear! Rating = ", "$rating stars")
  2 -> Log.i("Not good! Rating = ", "$rating stars")
  3 -> Log.i("Not bad! Rating = ", "$rating stars")
  4 -> Log.i("This is good! Rating = ", "$rating stars")
  5 -> Log.i("Amazing! Rating = ", "$rating stars")

  else -> {    
    Log.i("Error:", "$rating is not a valid rating")
  }
}
```

如果您将上述代码复制并粘贴到应用程序的`onCreate`函数中，它将产生以下输出：

```kt
This is good! Rating =: 4 stars

```

该代码首先将名为`rating`的`Int`变量初始化为`4`。然后，`when`块使用`rating`作为条件：

```kt
val rating:Int = 4
when (rating) {
```

接下来，处理了评分可能初始化为的五种不同可能性。对于每个值，从`1`到`5`，都会向 logcat 窗口输出不同的消息：

```kt
1 -> Log.i("Oh dear! Rating = ", "$rating stars")
2 -> Log.i("Not good! Rating = ", "$rating stars")
3 -> Log.i("Not bad! Rating = ", "$rating stars")
4 -> Log.i("This is good! Rating = ", "$rating stars")
5 -> Log.i("Amazing! Rating = ", "$rating stars")
```

最后，如果没有指定的选项为真，则会执行`else`块：

```kt
else -> {
  Log.i("Error:", "$rating is not a valid rating")
}
```

让我们通过构建一个小型演示应用程序来看一下`when`的稍微不同的用法。

## When Demo 应用

要开始，请创建一个名为`When Demo`的新 Android 项目。使用**空活动**项目模板，并将所有其他选项保持在通常的设置中。通过在编辑器上方单击**MainActivity.kt**标签，切换到`MainActivity.kt`文件，我们可以开始编码。

您可以在下载包的`Chapter08/When Demo`文件夹中获取此应用的代码。该文件还包括与我们先前讨论的表达式和`if`相关的代码。为什么不尝试玩一下代码，运行应用程序并研究输出呢？

在`onCreate`函数内添加以下代码。该应用程序演示了多个不同的值可以触发相同执行路径：

```kt
// Enter an ocean, river or breed of dog
val name:String = "Nile"
when (name) {
  "Atlantic","Pacific", "Arctic" -> 
    Log.i("Found:", "$name is an ocean")

  "Thames","Nile", "Mississippi" -> 
    Log.i("Found:", "$name is a river")

  "Labrador","Beagle", "Jack Russel" -> 
    Log.i("Found:", "$name is a dog")

  else -> {
    Log.i("Not found:", "$name is not in database")
  }
}
```

在前面的代码中，根据`name`变量初始化的值，有四条可能的执行路径。如果使用`Atlantic`、`Pacific`或`Arctic`的任何一个值，则执行以下代码行：

```kt
Log.i("Found:", "$name is an ocean")
```

如果使用`Thames`、`Nile`或`Mississippi`的任何一个值，则执行以下代码行：

```kt
Log.i("Found:", "$name is a river")
```

如果使用了`Labrador`、`Beagle`或`Jack Russel`的任何一个值，则执行以下代码行：

```kt
Log.i("Found:", "$name is a dog")
```

如果没有使用海洋、河流或狗来初始化`name`变量，则应用程序将分支到`else`块并执行以下代码行：

```kt
Log.i("Not found:", "$name is not in database")
```

如果使用`name`初始化为`Nile`（如前面的代码所做的那样）执行应用程序，则将在 logcat 窗口中看到以下输出：

```kt
Found:: Nile is a river

```

运行应用几次，每次将`name`的初始化更改为新的内容。注意，当您将`name`初始化为一个明确由语句处理的内容时，我们会得到预期的输出。否则，我们会得到`else`块处理的默认输出。

如果我们有很多代码要在`when`块中的选项中执行，我们可以将所有代码都放在一个函数中，然后调用该函数。我在以下假设的代码中突出显示了更改的行：

```kt
   "Atlantic","Pacific", "Arctic" -> 
         printFullDetailsOfOcean(name)

```

当然，我们将需要编写新的`printFullDetailsOfOcean`函数。然后，当`name`初始化为一个明确由语句处理的海洋之一时，将执行`printFullDetailsOfOcean`函数。然后执行将返回到`when`块之外的第一行代码。

### 提示

您可能想知道将`name`变量放在`printFullDetailsOfOcean(name)`函数调用的括号中的意义。发生的情况是，我们将存储在`name`变量中的数据传递给`printFullDetailsOfOcean`函数。这意味着`printFullDetailsOfOcean`函数可以使用该数据。这将在下一章中更详细地介绍。

当然，这段代码严重缺乏与 GUI 的交互。我们已经看到如何从按钮点击中调用函数，但即使这样也不足以使这段代码在真正的应用程序中有价值。我们将在第十二章中看到我们如何解决这个问题，*将我们的 Kotlin 连接到 UI 和可空性*。

我们还有另一个问题，那就是代码执行完毕后，就什么都不做了！我们需要它不断地询问用户的指令，不只是一次，而是一遍又一遍。我们将在下一步解决这个问题。

# 使用循环重复代码

在这里，我们将通过查看 Kotlin 中的几种**循环**类型，包括`while`循环、`do-while`循环和`for`循环，学习如何以受控且精确的方式重复执行代码的部分。我们还将了解在何种情况下使用这些不同类型的循环是最合适的。

询问循环与编程有什么关系是完全合理的，但它们确实如其名称所示。它们是重复执行代码的一种方式，或者循环执行相同的代码部分，尽管每次可能会有不同的结果。

这可能意味着重复执行相同的操作，直到循环的代码提示循环结束。它可以是由循环代码本身指定的预定次数的迭代。它可能是直到满足预定情况或**条件**为止。或者，它可能是这些事情的组合。除了`if`、`else`和`when`，循环也是 Kotlin**控制流语句**的一部分。

我们将学习 Kotlin 提供的所有主要类型的循环，使用其中一些来实现一个工作的迷你应用程序，以确保我们完全理解它们。让我们先看一下 Kotlin 中的第一种和最简单的循环类型，即`while`循环。

# while 循环

Kotlin 的`while`循环具有最简单的语法。回想一下`if`语句；我们可以在`if`语句的条件表达式中使用几乎任何组合的运算符和变量。如果表达式评估为真，则执行`if`块中的代码。对于`while`循环，我们也使用一个可以评估为真或假的表达式：

```kt
var x = 10

while(x > 0) {
  Log.i("x=", "$x")
  x--
}
```

看一下上述代码；这里发生的情况如下：

1.  在`while`循环之外，声明了一个名为`x`的`Int`类型，并将其初始化为 10。

1.  然后，`while`循环开始；它的条件是`x > 0`。因此，`while`循环将执行其主体中的代码。

1.  循环体中的代码将重复执行，直到条件评估为假。

因此，上述代码将执行 10 次。

在第一次循环中，`x`等于 10，在第二次循环中，它等于 9，然后是 8，依此类推。但一旦`x`等于 0，它当然不再大于 0。此时，执行将退出`while`循环，并继续执行`while`循环之后的第一行代码（如果有的话）。

与`if`语句一样，`while`循环可能甚至不会执行一次。看一下以下示例，`while`循环中的代码将不会执行：

```kt
var x = 10

while(x > 10){
   // more code here.
   // but it will never run 
  // unless x is greater than 10.
}
```

此外，条件表达式的复杂度或循环体中的代码量没有限制；以下是另一个例子：

```kt
var newMessages = 3
var unreadMessages = 0

while(newMessages > 0 || unreadMessages > 0){
   // Display next message
   // etc.
}

// continue here when newMessages and unreadMessages equal 0
```

上述`while`循环将继续执行，直到`newMessages`和`unreadMessages`都等于或小于零。由于条件使用逻辑或运算符(`||`)，其中一个条件为真将导致`while`循环继续执行。

值得注意的是，一旦进入循环体，即使表达式在中途评估为假，循环体也会始终完成。这是因为直到代码尝试开始另一次循环时才会再次测试：

```kt
var x = 1

while(x > 0){
   x--
   // x is now 0 so the condition is false
   // But this line still runs
   // and this one
   // and me!
}
```

上述循环体将执行一次。我们还可以设置一个永远运行的`while`循环！这被称为**无限循环**；以下是一个无限循环的例子：

```kt
var x = 0

while(true){
   x++ // I am going to get very big!
}
```

上述代码将永远不会结束；它将永远循环。我们将看到一些控制何时跳出`while`循环的解决方案。接下来，我们将看一下`while`循环的变体。

# do-while 循环

`do`-`while`循环的工作方式与普通的`while`循环相同，只是`do`块的存在保证了即使`while`表达式的条件不评估为真，代码也会至少执行一次：

```kt
var y = 10
do {
  y++
  Log.i("In the do block and y=","$y")
}
while(y < 10)
```

如果您将此代码复制并粘贴到`onCreate`函数中的一个应用程序中，然后执行它，输出可能不是您所期望的。以下是输出：

```kt
In the do block and y=: 11

```

这是一个不太常用但有时是解决问题的完美方案。即使`while`循环的条件为假，`do`块也会执行其代码，将`y`变量递增到 11，并打印一条消息到 logcat。`while`循环的条件是`y < 10`，因此`do`块中的代码不会再次执行。但是，如果`while`条件中的表达式为真，则`do`块中的代码将继续执行，就像是常规的`while`循环一样。

# 范围

为了继续讨论循环，有必要简要介绍范围的主题。范围与 Kotlin 的数组主题密切相关，我们将在第十五章*处理数据和生成随机数*中更全面地讨论。接下来是对范围的快速介绍，以便我们能够继续讨论`for`循环。

看一下使用范围的以下代码行：

```kt
val rangeOfNumbers = 1..4 
```

发生的情况是，我们使用类型推断来创建一个值的列表，其中包含值 1、2、3 和 4。

我们还可以显式声明和初始化一个列表，如下面的代码所示：

```kt
val rangeOfNumbers = listOf(1, 2, 3, 4, 5, 6, 7, 8, 9, 10)
```

上面的代码使用`listOf`关键字来显式创建一个包含 1 到 10 的数字的列表。

在我们学习关于数组的更深入的知识时，我们将更深入地探讨它们的工作原理，第十五章*处理数据和生成随机数*。然后，我们将看到范围、数组和列表比我们在这里涵盖的要多得多。通过查看`for`循环，这个快速介绍有助于我们完成对循环的讨论。

# For 循环

要使用`for`循环，我们需要一个范围或列表。然后，我们可以使用`for`循环来遍历该列表，并在每一步执行一些代码；看一下以下示例：

```kt
// We could do this...
// val list = listOf(1, 2, 3, 4, 5, 6, 7, 8, 9, 10)
// It is much quicker to do this...
val list = 1..10
for (i in list)
  Log.i("Looping through list","Current value is $i")
```

看一下如果将此内容复制并粘贴到应用程序中会产生的输出：

```kt
Looping through list: Current value is 1
Looping through list: Current value is 2
Looping through list: Current value is 3
Looping through list: Current value is 4
Looping through list: Current value is 5
Looping through list: Current value is 6
Looping through list: Current value is 7
Looping through list: Current value is 8
Looping through list: Current value is 9
Looping through list: Current value is 10

```

从输出中可以看出，`list`变量确实包含从 1 到 10 的所有值。在每次循环中，`i`变量保存当前值。您还可以看到，`for`循环允许我们遍历所有这些值，并根据这些值执行一些代码。

此外，当我们希望循环包含多行代码时，可以在`for`循环中使用开放和关闭的大括号：

```kt
for (i in list){
  Log.i("Looping through list","Current value is $i")
   // More code here
  // etc.
}
```

在 Kotlin 中，`for`循环非常灵活，可以处理的不仅仅是简单的`Int`值。在本章中，我们不会探讨所有选项，因为我们需要先了解更多关于类的知识。然而，在本书的其余部分，我们将在许多地方回到`for`循环。

# 使用`break`和`continue`控制循环

刚刚讨论了通过代码控制循环的所有方法，重要的是要知道，有时我们需要提前退出循环，而不是按照循环的条件指定的那样执行。

对于这种情况，Kotlin 有`break`关键字。以下是`break`在`while`循环中的作用：

```kt
var countDown = 10
while(countDown > 0){

  if(countDown == 5)break

  Log.i("countDown =","$countDown")
  countDown --
}
```

在上面的代码中，`while`循环的条件应该使代码在`countDown`变量大于零时重复执行。然而，在`while`循环内部，有一个`if`表达式，检查`countDown`是否等于 5。如果等于 5，则使用`break`语句。此外，在`while`循环内部，`countDown`的值被打印到 logcat 窗口，并递减（减少 1）。当执行此代码时，看一下以下输出：

```kt
countDown =: 10
countDown =: 9
countDown =: 8
countDown =: 7
countDown =: 6

```

从上面的输出可以看出，当`countDown`等于 5 时，`break`语句执行，执行提前退出`while`循环，而不会打印到 logcat 窗口。

有时，我们可能只想执行循环中的一部分代码，而不是完全停止循环。为此，Kotlin 有`continue`关键字。看看下面的带有`while`循环的代码，它演示了我们如何在应用程序中使用`continue`：

```kt
var countUp = 0
while(countUp < 10){
  countUp++

  if(countUp > 5)continue

  Log.i("Inside loop","countUp = $countUp")
}
Log.i("Outside loop","countUp = $countUp")
```

在前面的代码中，我们将一个名为`countUp`的变量初始化为零。然后我们设置了一个`while`循环，当`countUp`小于 10 时继续执行。在`while`循环内部，我们增加（加 1）`countUp`。下一行代码检查`countUp`是否大于 5，如果是，就执行`continue`语句。下一行代码将`countUp`的值打印到 logcat 窗口。只有当`countUp`为 5 或更低时，打印值的代码行才会执行，因为`continue`语句将应用程序的执行返回到循环的开始。看看下面的代码输出，以验证发生了什么：

```kt
Inside loop: countUp = 1
Inside loop: countUp = 2
Inside loop: countUp = 3
Inside loop: countUp = 4
Inside loop: countUp = 5
Outside loop: countUp = 10

```

您可以在前面的输出中看到，当`countUp`的值为 5 或更低时，它被打印出来。一旦它的值超过 5，`continue`语句将阻止执行打印的代码行。然而，循环外的最后一行代码打印了`countUp`的值，你可以看到它的值是 10，这表明循环中的第一行代码，即增加`countUp`的代码，一直执行到`while`循环条件完成。

`break`和`continue`关键字也可以用在`for`循环和`do`-`while`循环中。

# 示例代码

如果你想玩转循环代码，可以创建一个名为`Loops Demo`的新项目，并将本章中的任何代码复制到`onCreate`函数的末尾。我已经将我们在讨论循环时使用的代码放在了`Chapter08/Loops Demo`文件夹中。

# 总结

在本章中，我们使用`if`、`else`和`when`来做出表达式的决策并分支我们的代码。我们看到并练习了`while`、`for`和`do`-`while`来重复我们代码的部分。此外，我们使用`break`在条件允许之前跳出循环，并使用`continue`有条件地执行循环中的部分代码。

如果你不记得所有内容也没关系，因为我们将不断地在整本书中使用所有这些技术和关键字。我们还将探索一些更高级的使用这些技术的方法。

在下一章中，我们将更仔细地研究 Kotlin 函数，这是我们所有测试和循环代码的去处。
