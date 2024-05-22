# 第十五章。处理数据和生成随机数

我们取得了很好的进展。我们对 Android UI 选项和 Kotlin 的基础知识有了一个全面的了解。在前几章中，我们开始将这两个领域结合起来，并使用 Kotlin 代码操作 UI，包括一些新的小部件。然而，在构建自我备忘录应用程序时，我们在知识上遇到了一些空白。在本章中，我们将填补这些空白中的第一个，然后在下一章中，我们将使用这些新信息来继续应用程序。我们目前没有办法管理大量相关数据。除了声明、初始化和管理数十、数百甚至数千个属性或实例之外，我们如何让我们的应用程序用户拥有多个备忘录？我们还将快速了解一下随机数。

在本章中，我们将涵盖以下主题：

+   随机数

+   数组

+   一个简单的数组迷你应用

+   一个动态数组迷你应用

+   范围

+   ArrayLists

+   哈希映射

首先，让我们了解一下`Random`类。

# 一个随机的转移

有时，我们会在我们的应用程序中需要一个随机数，对于这些情况，Kotlin 为我们提供了`Random`类。这个类有很多可能的用途，比如如果我们的应用程序想要显示每日随机提示，或者一个需要在不同场景之间选择的游戏，或者一个随机提问的测验。

`Random`类是 Android API 的一部分，在我们的 Android 应用程序中完全兼容。

让我们看看如何创建随机数。`Random`类已经为我们做好了所有的工作。首先，我们需要创建一个`Random`对象，如下所示：

```kt
val randGenerator = Random()
```

然后，我们使用我们新对象的`nextInt`函数来生成一个在某个范围内的随机数。以下代码行使用我们的`randGenerator`对象生成随机数，并将结果存储在`ourRandomNumber`变量中：

```kt
var ourRandomNumber = randGenerator.nextInt(10)
```

我们输入的范围开始于零。因此，前一行将生成一个在 0 和 9 之间的随机数。如果我们想要一个在 1 和 10 之间的随机数，我们只需在同一行的代码末尾添加增量运算符：

```kt
ourRandomNumber ++
```

我们还可以使用`Random`对象使用`nextLong`、`nextFloat`和`nextDouble`获取其他类型的随机数。

# 使用数组处理大量数据

也许你会想知道当我们有很多变量需要跟踪时会发生什么。我们的自我备忘录应用程序有 100 条备忘录，或者游戏中的高分榜有前 100 名的分数？我们可以声明和初始化 100 个单独的变量，如下所示：

```kt
var note1 = Note()
var note2 = Note()
var note3 = Note()
// 96 more lines like the above
var note100 = Note()
```

或者，通过使用高分示例，我们可以使用以下代码：

```kt
var topScore1: Int
var topScore2: Int
// 96 more lines like the above
var topScore100: Int
```

立即，这段代码可能看起来笨拙，但是当有人获得新的最高分，或者我们想让我们的用户排序他们的备忘录显示顺序时，会怎样？使用高分榜场景，我们必须将每个变量中的分数向下移动一个位置。这是一个噩梦的开始，如下代码所示：

```kt
topScore100 = topScore99;
topScore99 = topScore98;
topScore98 = topScore97;
// 96 more lines like the above
topScore1 = score;
```

肯定有更好的方法。当我们有一整个数组的变量时，我们需要的是一个 Kotlin **数组**。数组是一个对象，最多可以容纳预定的固定最大数量的元素。每个元素都是一个具有一致类型的变量。

以下代码声明了一个可以容纳`Int`类型变量的数组；例如高分榜或一系列考试成绩：

```kt
var myIntArray: IntArray
```

我们也可以声明其他类型的数组，如下所示：

```kt
var myFloatArray: FloatArray
var myBooleanArray: BooleanArray
```

在使用这些数组之前，每个数组都需要有一个固定的最大分配存储空间。就像我们对其他对象所做的那样，我们必须在使用数组之前对其进行初始化，我们可以这样做：

```kt
myIntArray = IntArray(100)
myFloatArray = FloatArray(100)
myBooleanArray = BooleanArray(100)
```

前面的代码分配了最多`100`个适当类型的存储空间。想象一下，我们的变量仓库中有 100 个连续的存储空间。这些空间可能被标记为`myIntArray[0]`，`myIntArray[1]`和`myIntArray[2]`，每个空间都包含一个`Int`值。这里稍微令人惊讶的是，存储空间从零开始，而不是 1。因此，在一个 100 宽的数组中，存储空间将从 0 到 99。

我们可以初始化一些存储空间如下：

```kt
myIntArray [0] = 5
myIntArray [1] = 6
myIntArray [2] = 7
```

但是，请注意，我们只能将预声明的类型放入数组中，并且数组保存的类型永远不会改变，如下面的代码所示：

```kt
myIntArray [3] = "John Carmack" 
// Won't compile String not Int
```

因此，当我们有一个`Int`类型的数组时，每个`Int`变量被称为什么，我们如何访问其中存储的值？数组表示法语法替换了变量的名称。此外，我们可以对数组中的变量进行与常规变量相同的操作；如下所示：

```kt
myIntArray [3] = 123
```

前面的代码将值 123 分配给数组中的第 4 个位置。

这是使用数组的另一个示例，就像使用普通变量一样：

```kt
myIntArray [10] = myIntArray [9] - myIntArray [4]
```

前面的代码从数组的第 5 个位置中减去数组的第 10 个位置中存储的值，并将答案赋给数组的第 11 个位置。

我们还可以将数组中的值赋给相同类型的常规变量，如下所示：

```kt
Val myNamedInt = myIntArray[3]
```

但是，请注意，`myNamedInt`是一个独立的变量，对它的任何更改都不会影响存储在`IntArray`引用中的值。它在仓库中有自己的空间，与数组没有其他联系。

在前面的示例中，我们没有检查任何字符串或对象。实际上，字符串是对象，当我们想要创建对象数组时，我们会稍微不同地处理它们；看一下下面的代码：

```kt
var someStrings = Array<String>(5) { "" }
// You can remove the String keyword because it can be inferred like 
// this
var someMoreStrings = Array(5) { "" }

someStrings[0]= "Hello "
someStrings[1]= "from "
someStrings[2]= "inside "
someStrings[3]= "the "
someStrings[4]= "array "
someStrings[5]= "Oh dear "
// ArrayIndexOutOfBoundsException
```

前面的代码声明了一个 String 对象数组，最多可以容纳五个对象。请记住，数组从 0 开始，因此有效的位置是从 0 到 4。如果尝试使用无效的位置，则会收到**ArrayIndexOutOfBoundsException**错误。如果编译器注意到错误，则代码将无法编译；但是，如果编译器无法发现错误，并且在应用程序执行时发生错误，则应用程序将崩溃。

我们可以避免这个问题的唯一方法是知道规则-数组从 0 开始，直到它们的长度减 1。因此，`someArray[9]`是数组中的第十个位置。我们还可以使用清晰易读的代码，这样更容易评估我们所做的事情并更容易发现问题。

您还可以在声明数组的同时初始化数组的内容，如下面的代码所示：

```kt
        var evenMoreStrings: Array<String> = 
                arrayOf("Houston", "we", "have", "an", "array")
```

前面的代码使用内置的 Kotlin 函数`arrayOf`来初始化数组。

在 Kotlin 中，您可以声明和初始化数组的方式非常灵活。我们还没有接近覆盖我们可以使用数组的所有方式，即使在书的最后，我们仍然不会覆盖所有内容。然而，让我们深入一点。

## 数组是对象

将数组变量视为给定类型的一组变量的地址。例如，使用仓库类比，`someArray`可以是过道编号。因此，`someArray[0]`和`someArray[1]`是过道编号，后跟过道中的位置编号。

因为数组也是对象，它们具有我们可以使用的函数和属性，如下面的示例所示：

```kt
val howBig = someArray.size
```

在前面的示例中，我们将`someArray`的长度（即大小）分配给名为`howBig`的`Int`变量。

我们甚至可以声明一个数组的数组。这是一个数组，其中每个位置都隐藏着另一个数组；如下所示：

```kt
val cities = arrayOf("London", "New York", "Yaren")
val countries = arrayOf("UK", "USA", "Nauru")

val countriesAndCities = arrayOf(countries, cities)

Log.d("The capital of " +
   countriesAndCities[0][0],
   " is " +
   countriesAndCities[1][0])
```

前面的`Log`代码将在 logcat 窗口中输出以下文本：

```kt
The capital of UK:  is London

```

让我们在一个真实的应用程序中使用一些数组，试着理解如何在真实代码中使用它们以及它们可能被用来做什么。

# 一个简单的迷你应用程序数组示例

让我们做一个简单的工作数组示例。您可以在可下载的代码包中找到此项目的完整代码。它可以在`Chapter15/Simple Array Example/MainActivity.kt`文件中找到。

创建一个**Empty Activity**项目模板的项目，并将其命名为`Simple Array Example`。

首先，我们声明我们的数组，分配了五个空间，并为每个元素初始化了值。然后，我们将每个值输出到**logcat**窗口。

这与我们之前看到的例子略有不同，因为我们在声明数组的同时声明了大小。

在`setContentView`调用后的`onCreate`函数中添加以下代码：

```kt
// Declaring an array
// Allocate memory for a maximum size of 5 elements
val ourArray = IntArray(5)

// Initialize ourArray with values
// The values are arbitrary, but they must be Int
// The indexes are not arbitrary. Use 0 through 4 or crash!

ourArray[0] = 25
ourArray[1] = 50
ourArray[2] = 125
ourArray[3] = 68
ourArray[4] = 47

//Output all the stored values
Log.i("info", "Here is ourArray:")
Log.i("info", "[0] = " + ourArray[0])
Log.i("info", "[1] = " + ourArray[1])
Log.i("info", "[2] = " + ourArray[2])
Log.i("info", "[3] = " + ourArray[3])
Log.i("info", "[4] = " + ourArray[4])
```

接下来，我们将数组的每个元素相加，就像我们对普通的`Int`类型变量一样。请注意，当我们将数组元素相加时，我们为了清晰起见在多行上这样做。将我们刚刚讨论的代码添加到`MainActivity.kt`中，如下所示：

```kt
/*
   We can do any calculation with an array element
   provided it is appropriate to the contained type
   Like this:
*/
val answer = ourArray[0] +
      ourArray[1] +
      ourArray[2] +
      ourArray[3] +
      ourArray[4]

Log.i("info", "Answer = $answer")
```

运行示例，并注意 logcat 窗口中的输出。请记住，在模拟器显示上不会发生任何事情，因为所有输出都将发送到 Android Studio 中的 logcat 窗口；以下是输出：

```kt
info﹕ Here is ourArray:
info﹕ [0] = 25
info﹕ [1] = 50
info﹕ [2] = 125
info﹕ [3] = 68
info﹕ [4] = 47
info﹕ Answer = 315 

```

我们声明一个名为`ourArray`的数组来保存`Int`值，然后为该类型的最多五个值分配空间。

接下来，我们为`ourArray`的五个空间分配一个值。记住第一个空间是`ourArray[0]`，最后一个空间是`ourArray[4]`。

接下来，我们简单地将每个数组位置的值打印到 logcat 窗口中，从输出中我们可以看到它们保存了我们在上一步中初始化的值。然后，我们将`ourArray`中的每个元素相加，并将它们的值初始化为`answer`变量。然后，我们将`answer`打印到 logcat 窗口中，我们可以看到确实，所有的值都被相加在一起，就像它们是普通的`Int`类型一样（它们确实是），只是以不同的方式存储。

# 使用数组进行动态操作

正如我们在本节开头讨论的，如果我们需要单独声明和初始化数组的每个元素，那么使用数组并没有比使用普通变量带来很大的好处。让我们看一个动态声明和初始化数组的例子。

## 动态数组示例

您可以在下载包中找到此示例的工作项目。它可以在`Chapter15/Dynamic Array Example/MainActivity.kt`文件中找到。

创建一个**Empty Activity**模板的项目，并将其命名为`Dynamic Array Example`。

在`onCreate`函数中的`setContentView`调用后，输入以下代码。在我们讨论和分析代码之前，看看你能否猜出输出结果是什么：

```kt
// Declaring and allocating in one step
val ourArray = IntArray(1000)

// Let's initialize ourArray using a for loop
// Because more than a few variables is allot of typing!

for (i in 0..999) {

   // Put the value into ourArray
   // At the position decided by i.
   ourArray[i] = i * 5

   //Output what is going on
   Log.i("info", "i = $i")
   Log.i("info", "ourArray[i] = ${ ourArray[i]}")
}
```

运行示例应用程序。请记住，屏幕上不会发生任何事情，因为所有输出都将发送到我们在 Android Studio 中的 logcat 窗口；以下是输出：

```kt
info﹕ i = 0
info﹕ ourArray[i] = 0
info﹕ i = 1
info﹕ ourArray[i] = 5
info﹕ i = 2
info﹕ ourArray[i] = 10

```

为了简洁起见，循环的 994 次迭代已被删除：

```kt
info﹕ ourArray[i] = 4985
info﹕ i = 998
info﹕ ourArray[i] = 4990
info﹕ i = 999
info﹕ ourArray[i] = 4995

```

首先，我们声明并分配了一个名为`ourArray`的数组，以保存最多 1,000 个`Int`值。请注意，这次我们在一行代码中执行了两个步骤：

```kt
val ourArray = IntArray(1000)
```

然后，我们使用了一个设置为循环 1,000 次的`for`循环：

```kt
for (i in 0..999) {
```

我们初始化数组中的空间，从 0 到 999，其值为`i`乘以 5，如下所示：

```kt
   ourArray[i] = i * 5
```

然后，为了演示`i`的值以及数组中每个位置的值，我们输出`i`的值，然后是数组对应位置的值，如下所示：

```kt
   //Output what is going on
   Log.i("info", "i = $i")
   Log.i("info", "ourArray[i] = ${ ourArray[i]}")
```

所有这些都发生了 1,000 次，产生了我们所看到的输出。当然，我们还没有在真实的应用程序中使用这种技术，但我们很快将使用它来使我们的自我备忘录应用程序保存几乎无限数量的备忘录。

# ArrayLists

`ArrayList`对象就像普通数组，但功能更强大。它克服了数组的一些缺点，比如必须预先确定其大小。它添加了几个有用的函数来使其数据易于管理，并被 Android API 中的许多类使用。这最后一点意味着如果我们想要使用 API 的某些部分，我们需要使用`ArrayList`。在第十六章中，*适配器和回收器*，我们将真正地让`ArrayList`发挥作用。首先是理论。

让我们看一些使用`ArrayList`的代码：

```kt
// Declare a new ArrayList called myList 
// to hold Int variables
val myList: ArrayList<Int>

// Initialize myList ready for use
myList = ArrayList()
```

在前面的代码中，我们声明并初始化了一个名为`myList`的新`ArrayList`对象。我们也可以在一步中完成这个操作，就像下面的代码所示：

```kt
val myList: ArrayList<Int> = ArrayList()
```

到目前为止，这并不特别有趣，所以让我们看看我们实际上可以用`ArrayList`做些什么。这次我们使用一个`String ArrayList`对象：

```kt
// declare and initialize a new ArrayList
val myList = ArrayList<String>()

// Add a new String to myList in 
// the next available location
myList.add("Donald Knuth")
// And another
myList.add("Rasmus Lerdorf")
// We can also choose 'where' to add an entry
myList.add(1,"Richard Stallman")

// Is there anything in our ArrayList?
if (myList.isEmpty()) {
   // Nothing to see here
} else {
   // Do something with the data
}

// How many items in our ArrayList?
val numItems = myList.size

// Now where did I put Richard?
val position = myList.indexOf("Richard Stallman")
```

在前面的代码中，我们看到我们可以在`ArrayList`对象上使用`ArrayList`类的一些有用的函数；这些函数如下：

+   我们可以添加一个条目（`myList.add`）

+   我们可以在特定位置添加一个条目（`myList.add(x, value)`）

+   我们可以检查`ArrayList`实例是否为空（`myList.isEmpty()`）

+   我们可以看到`ArrayList`实例的大小（`myList.size`）

+   我们可以获取给定条目的当前位置（`myList.indexOf...`）

### 注意

`ArrayList`类中甚至有更多的函数，但是到目前为止我们已经看到的足以完成这本书了。

有了所有这些功能，我们现在只需要一种方法来动态处理`ArrayList`实例。这就是增强`for`循环的条件的样子：

```kt
for (String s : myList)
```

前面的例子将逐个遍历`myList`中的所有项目。在每一步中，`s`将保存当前的`String`条目。

因此，这段代码将把我们上一节`ArrayList`代码示例中的所有杰出程序员打印到 logcat 窗口中，如下所示：

```kt
for (s in myList) {
   Log.i("Programmer: ", "$s")
}
```

它的工作原理是`for`循环遍历`ArrayList`中的每个`String`，并将当前的`String`条目分配给`s`。然后，依次对每个`s`使用`Log…`函数调用。前面的循环将在 logcat 窗口中创建以下输出：

```kt
Programmer:: Donald Knuth
Programmer:: Richard Stallman
Programmer:: Rasmus Lerdorf

```

`for`循环已经输出了所有的名字。Richard Stallman 之所以在 Donald Knuth 和 Rasmus Lerdof 之间是因为我们在特定位置（1）插入了他，这是`ArrayList`中的第二个位置。`insert`函数调用不会删除任何现有的条目，而是改变它们的位置。

有一个新的新闻快讯！

# 数组和 ArrayLists 是多态的

我们已经知道我们可以将对象放入数组和`ArrayList`对象中。然而，多态意味着它们可以处理多个不同类型的对象，只要它们有一个共同的父类型 - 都在同一个数组或`ArrayList`中。

在第十章，面向对象编程中，我们学到多态意味着多种形式。但在数组和`ArrayList`的上下文中，对我们意味着什么呢？

在其最简单的形式中，它意味着任何子类都可以作为使用超类的代码的一部分。

例如，如果我们有一个`Animals`数组，我们可以把任何`Animal`子类对象放在`Animals`数组中，比如`Cat`和`Dog`。

这意味着我们可以编写更简单、更易于理解和更易于更改的代码：

```kt
// This code assumes we have an Animal class
// And we have a Cat and Dog class that 
// inherits from Animal
val myAnimal = Animal()
val myDog = Dog()
val myCat = Cat()
val myAnimals = arrayOfNulls<Animal>(10)
myAnimals[0] = myAnimal // As expected
myAnimals[1] = myDog // This is OK too
myAnimals[2] = myCat // And this is fine as well
```

此外，我们可以为超类编写代码，并依赖于这样一个事实，即无论它被子类化多少次，在一定的参数范围内，代码仍然可以工作。让我们继续我们之前的例子如下：

```kt
// 6 months later we need elephants
// with its own unique aspects
// If it extends Animal we can still do this
val myElephant = Elephant()
myAnimals[3] = myElephant // And this is fine as well
```

我们刚刚讨论的一切对于`ArrayLists`也是真实的。

# 哈希映射

Kotlin 的`HashMap`很有趣；它们是`ArrayList`的一种表亲。它们封装了一些有用的数据存储技术，否则对我们来说可能会相当技术性。在回到自己的笔记应用之前，值得看一看`HashMap`。

假设我们想要存储角色扮演游戏中许多角色的数据，每个不同的角色由`Character`类型的对象表示。

我们可以使用一些我们已经了解的 Kotlin 工具，比如数组或`ArrayList`。然而，使用`HashMap`，我们可以为每个`Character`对象提供一个唯一的键或标识符，并使用相同的键或标识符访问任何这样的对象。

### 注意

"哈希"一词来自于将我们选择的键或标识符转换为`HashMap`类内部使用的东西的过程。这个过程被称为**哈希**。

我们选择的键或标识符可以访问任何`Character`实例。在`Character`类的情况下，一个好的键或标识符候选者是角色的名字。

每个键或标识符都有一个相应的对象；在这种情况下，是`Character`实例。这被称为**键值对**。

我们只需给`HashMap`一个键，它就会给我们相应的对象。我们不需要担心我们存储了角色的哪个索引，比如 Geralt、Ciri 或 Triss；只需将名字传递给`HashMap`，它就会为我们完成工作。

让我们看一些例子。你不需要输入任何代码；只需熟悉它的工作原理。

我们可以声明一个新的`HashMap`实例来保存键和`Character`实例，如下所示：

```kt
val characterMap: Map<String, Character>
```

前面的代码假设我们已经编写了一个名为`Character`的类。然后我们可以初始化`HashMap`实例如下：

```kt
characterMap = HashMap()
```

然后，我们可以添加一个新的键及其关联的对象，如下所示：

```kt
characterMap.put("Geralt", Character())
characterMap.put("Ciri", Character())
characterMap.put("Triss", Character())
```

### 提示

所有示例代码都假设我们可以以某种方式给`Character`实例赋予它们的唯一属性，以反映它们在其他地方的内部差异。

然后，我们可以按如下方式从`HashMap`实例中检索条目：

```kt
val ciri = characterMap.get("Ciri")
```

或者，我们可以直接使用`Character`类的函数：

```kt
characterMap.get("Geralt").drawSilverSword()

// Or maybe call some other hypothetical function
characterMap.get("Triss").openFastTravelPortal("Kaer Morhen")
```

前面的代码调用了假设的`drawSilverSword`和`openFastTravelPortal`函数，这些函数是存储在`HashMap`实例中的`Character`类实例的假设函数。

有了这些新的工具包，如数组、`ArrayList`、`HashMap`，以及它们的多态性，我们可以继续学习一些更多的 Android 类，很快我们将用它们来增强我们的备忘录应用。

# 备忘录应用

尽管我们已经学到了很多，但我们还没有准备好将解决方案应用到备忘录应用中。我们可以更新我们的代码，将大量的`Note`实例存储在`ArrayList`中，但在这之前，我们还需要一种方法来在 UI 中显示`ArrayList`的内容。把整个东西放在`TextView`实例中看起来不好。

答：解决方案是**适配器**和一个名为`RecyclerView`的特殊 UI 布局。我们将在下一章中介绍它们。

# 常见问题

问：一个只能进行真实计算的计算机如何可能生成真正的随机数？

问：实际上，计算机无法创建真正随机的数字，但`Random`类使用一个**种子**，产生一个在严格的统计检验下被认为是真正随机的数字。要了解更多关于种子和生成随机数的信息，请查看以下文章：[`en.wikipedia.org/wiki/Random_number_generation`](https://en.wikipedia.org/wiki/Random_number_generation)。

# 总结

在本章中，我们看了如何使用简单的 Kotlin 数组来存储大量数据，只要它们是相同类型的数据。我们还使用了`ArrayList`，它类似于一个带有许多额外功能的数组。此外，我们发现数组和`ArrayList`都是多态的，这意味着一个数组（或`ArrayList`）可以容纳多个不同的对象，只要它们都是从同一个父类派生的。

我们还了解了`HashMap`类，它也是一种数据存储解决方案，但允许以不同的方式访问。

在下一章中，我们将学习关于`Adapter`和`RecyclerView`，将理论付诸实践，并增强我们的备忘录应用。
