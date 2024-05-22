# 第十五章：*第十五章*：数组、映射和随机数

在本章中，我们将学习 Java 数组，它允许我们以有组织和高效的方式操纵大量数据。我们还将使用与数组有密切关系的 Java 类`ArrayList`，并研究它们之间的区别。

一旦我们熟悉处理大量数据，我们将看看 Android API 提供了什么帮助，让我们轻松地将我们新发现的数据处理技能与用户界面连接起来，而不费吹灰之力。

本章的主题包括以下内容：

+   `Random`类

+   使用数组处理数据

+   数组小应用程序

+   包括一个小型应用程序的动态数组

+   包括一个小型应用程序的多维数组

+   `ArrayList`类

+   增强型`for`循环

+   Java HashMap

首先，让我们了解一下`Random`类。

# 技术要求

您可以在 GitHub 上找到本章的代码文件，网址为[`github.com/PacktPublishing/Android-Programming-for-Beginners-Third-Edition/tree/main/chapter%2015`](https://github.com/PacktPublishing/Android-Programming-for-Beginners-Third-Edition/tree/main/chapter%2015)。

# 一个随机的转移

有时在我们的应用程序中，我们会需要一个随机数，而 Java 为我们提供了`Random`类来满足这些需求。这个类有许多可能的用途。例如，也许我们的应用程序想要显示每日随机提示，或者是一个需要在不同情景之间选择的游戏，或者是一个随机提问的测验。

`Random`类是 Java API 的一部分，完全兼容我们的 Android 应用程序。

让我们看看如何创建随机数，然后在本章中我们将把它应用到实际中。所有的工作都由`Random`类为我们完成。首先，我们需要创建一个`Random`类型的对象：

```kt
Random randGenerator = new Random();
```

然后我们使用我们新对象的`nextInt`方法在一定范围内生成一个随机数。下一行代码使用我们的`Random`对象生成随机数，并将结果存储在`ourRandomNumber`变量中：

```kt
int ourRandomNumber = randGenerator.nextInt(10);
```

我们输入的范围开始于零。因此，前一行将生成一个介于 0 和 9 之间的随机数。如果我们想要一个介于 1 和 10 之间的随机数，我们只需这样做：

```kt
ourRandomNumber ++;
```

我们还可以使用`Random`对象使用`nextLong`、`nextFloat`和`nextDouble`方法获取其他类型的随机数。

我们将在本章后面用一个快速地理测验应用程序实际使用`Random`类。

# 使用数组处理大量数据

也许您会想知道当我们有很多变量需要跟踪时会发生什么。我们的自我备忘录应用程序有 100 条备忘录，或者游戏中有前 100 名得分的高分表怎么办？我们可以像这样声明和初始化 100 个单独的变量：

```kt
Note note1;
Note note2;
Note note3;
// 96 more lines like the above
Note note100;
```

或者我们可以这样做：

```kt
int topScore1;
int topScore2;
int topScore3;
// 96 more lines like the above
int topScore100;
```

一开始，这可能看起来笨拙，但是当有人获得新的最高分或者我们想让用户对他们的笔记显示顺序进行排序时怎么办？使用高分情景，我们必须将每个变量中的分数下移一个位置。噩梦开始了：

```kt
topScore100 = topScore99;
topScore99 = topScore98;
topScore98 = topScore97;
// 96 more lines like the above
topScore1 = score;
```

一定有更好的方法。当我们有一整个数组的变量时，我们需要的是一个 Java 数组。数组是一个引用变量，最多可以容纳预先确定的固定数量的元素。每个元素都是一个具有一致类型的变量。

以下代码声明了一个可以容纳`int`类型变量的数组，可能是一个高分表或一系列考试成绩：

```kt
int [] intArray;
```

我们还可以声明其他类型的数组，包括`Note`等类，如下所示：

```kt
String [] classNames;
boolean [] bankOfSwitches;
float [] closingBalancesInMarch;
Note [] notes;
```

在使用之前，这些数组中的每一个都需要分配固定的最大存储空间。就像其他对象一样，我们必须在使用数组之前对其进行初始化：

```kt
intArray = new int [100];
```

上面的代码分配了最多 100 个`int`大小的存储空间。想象一下我们的内存仓库中有 100 个连续的存储空间。这些空间可能被标记为`intArray[0]`，`intArray[1]`，`intArray[2]`等等，每个空间都保存一个单独的`int`值。也许稍微令人惊讶的是，存储空间从零开始，而不是 1。因此，在一个 100 宽的数组中，存储空间将从 0 到 99。

我们可以像这样初始化一些存储空间中的值：

```kt
intArray[0] = 5;
intArray[1] = 6;
intArray[2] = 7;
```

但请注意，我们只能将预先声明的类型放入数组中，数组保存的类型永远不会改变：

```kt
intArray[3]= "John Carmack"; // Won't compile String not int
```

因此，当我们有一个`int`类型的数组时，每个这些`int`变量被称为什么？这些变量的名称是什么，我们如何访问其中存储的值？数组表示法语法替换了变量名称。我们可以对数组中的变量进行与常规变量相同的操作：

```kt
intArray [3] = 123;
```

上面的代码将值`123`分配给`intArray`的第四个位置。这里是另一个例子：

```kt
intArray[10] = intArray[9] - intArray[4];
```

上面的代码从`intArray`的第五个位置减去第十个位置的值，并将答案存储在第十一个位置。我们还可以将数组中的值分配给相同类型的常规变量，如下所示：

```kt
int myNamedInt = intArray [3];
```

但请注意，`myNamedInt`是一个单独且独立的原始变量，对它的任何更改都不会影响存储在`intArray`引用中的值。它在仓库中有自己的空间，并且与数组没有关联。更具体地说，数组在堆上，而`int`变量在栈上。

## 数组是对象

我们说过数组是引用变量。将数组变量视为给定类型的一组变量的地址。也许，使用仓库的类比，`someArray`是一个过道编号。因此，`someArray[0]`，`someArray[1]`等等是过道编号，后面是过道中的位置编号。

数组也是对象；也就是说，它们有我们可以使用的方法和属性。例如：

```kt
int lengthOfSomeArray = someArray.length;
```

在上面的代码中，我们将`someArray`的长度分配给名为`lengthOfSomeArray`的`int`变量。

我们甚至可以声明一个数组的数组。这是一个数组，其中每个元素中都隐藏着另一个数组，如下所示：

```kt
String[][] countriesAndCities;
```

在前面的数组中，我们可以在每个国家中保存一个城市列表。不过，现在先不要疯狂地使用数组。只需记住，数组最多可以容纳预定数量的任意类型的变量，并且可以使用以下语法访问这些值：

```kt
someArray[someLocation];
```

让我们在一个真实的应用程序中使用一些数组，试着理解如何在真实代码中使用它们以及我们可能用它们做什么。

# 简单数组示例迷你应用程序

让我们制作一个简单的工作数组示例。您可以在可下载的代码包中找到此示例的完整代码。它在*第十五章*`/Simple Array Example/MainActivity.java`。

使用空活动模板创建一个项目，并将其命名为`Simple Array Example`。

首先，我们声明我们的数组，分配五个空间，并为每个元素初始化值。然后我们将每个值输出到 logcat 控制台。将此代码添加到`onCreate`方法中，就在调用`setContentView`之后：

```kt
// Declaring an array
int[] ourArray;
// Allocate memory for a maximum size of 5 elements
ourArray = new int[5];
// Initialize ourArray with values
// The values are arbitrary, but they must be int
// The indexes are not arbitrary. 0 through 4 or crash!
ourArray[0] = 25;
ourArray[1] = 50;
ourArray[2] = 125;
ourArray[3] = 68;
ourArray[4] = 47;
//Output all the stored values
Log.i("info", "Here is ourArray:");
Log.i("info", "[0] = "+ourArray[0]);
Log.i("info", "[1] = "+ourArray[1]);
Log.i("info", "[2] = "+ourArray[2]);
Log.i("info", "[3] = "+ourArray[3]);
Log.i("info", "[4] = "+ourArray[4]);
```

接下来，我们将数组的每个元素相加，就像我们可以对常规的`int`类型变量一样。请注意，当我们将数组元素相加时，我们是在多行上这样做的。这没问题，因为我们省略了分号，直到最后一个操作，所以 Java 编译器将这些行视为一个语句。将我们刚刚讨论的代码添加到`MainActivity.java`：

```kt
/*
   We can do any calculation with an array element
   provided it is appropriate to the contained type
   Like this:
*/
int answer = ourArray[0] +
ourArray[1] +
ourArray[2] +
ourArray[3] +
ourArray[4];
Log.i("info", "Answer = "+ answer);
```

运行示例并在 logcat 窗口中查看输出。

请记住，模拟器显示不会发生任何事情，因为所有输出都将发送到我们在 Android Studio 中的 logcat 控制台窗口。这是输出：

```kt
info﹕ Here is ourArray:
info﹕ [0] = 25
info﹕ [1] = 50
info﹕ [2] = 125
info﹕ [3] = 68
info﹕ [4] = 47
info﹕ Answer = 315 
```

我们声明了一个名为`ourArray`的数组来保存`int`变量，然后为该类型的最多五个变量分配了空间。

接下来，我们为数组中的五个空间分配了一个值。请记住，第一个空间是`ourArray[0]`，最后一个空间是`ourArray[4]`。

接下来，我们简单地将每个数组位置的值打印到控制台上，从输出中我们可以看到它们保存了我们在上一步中初始化的值。然后我们将`ourArray`中的每个元素相加，并将它们的值初始化为`answer`变量。然后我们将`answer`打印到控制台上，我们可以看到确实所有的值都被相加了，就像它们是普通的`int`类型一样，它们确实是，只是以不同的方式存储和访问。

# 使用数组进行动态处理

正如我们在所有这些数组内容开始时讨论的，如果我们需要单独声明和初始化数组的每个元素，那么数组与常规变量相比并没有太大的好处。让我们看一个动态声明和初始化数组的例子。

## 动态数组示例

让我们做一个简单的动态数组示例。您可以在下载包中找到此示例的工作项目。它在*第十五章*`/Dynamic Array Example/MainActivity.java`中。

使用空活动模板创建一个项目，并将其命名为“动态数组示例”。

在`onCreate`方法中的`setContentView`方法调用之后，输入以下代码。在我们讨论和分析代码之前，看看您能否猜出输出会是什么：

```kt
// Declaring and allocating in one step
int[] ourArray = new int[1000];
// Let's initialize ourArray using a for loop
// Because more than a few variables is allot of typing!
for(int i = 0; i < 1000; i++){
   // Put the value of our value into ourArray
   // At the position decided by i.
   ourArray[i] = i*5;
   //Output what is going on
   Log.i("info", "i = " + i);
   Log.i("info", "ourArray[i] = " + ourArray[i]);
}
```

运行示例应用程序，记住屏幕上不会发生任何事情，因为所有输出都将发送到我们在 Android Studio 中的 logcat 控制台窗口。以下是输出：

```kt
info﹕ i = 0
info﹕ ourArray[i] = 0
info﹕ i = 1
info﹕ ourArray[i] = 5
info﹕ i = 2
info﹕ ourArray[i] = 10
... 994 iterations of the loop removed for brevity.
info﹕ ourArray[i] = 4985
info﹕ i = 998
info﹕ ourArray[i] = 4990
info﹕ i = 999
info﹕ ourArray[i] = 4995
```

首先，我们声明并分配了一个名为`ourArray`的数组，用于保存最多 1,000 个`int`值。请注意，这次我们在一行代码中完成了两个步骤：

```kt
int[] ourArray = new int[1000];
```

然后我们使用了一个设置为循环 1,000 次的`for`循环：

```kt
(int i = 0; i < 1000; i++){
```

我们用值为`i`乘以 5 来初始化数组中的空间，从 0 到 999，就像这样：

```kt
ourArray[i] = i*5;
```

然后，为了演示`i`的值以及数组中每个位置保存的值，我们输出了`i`的值，然后是数组中相应位置保存的值，就像这样：

```kt
Log.i("info", "i = " + i);
Log.i("info", "ourArray[i] = " + ourArray[i]);
```

所有这些都发生了 1,000 次，产生了我们所看到的输出。当然，我们还没有在真实世界的应用程序中使用这种技术，但我们很快将使用它来使我们的“备忘录”应用程序保存几乎无限数量的备忘录。

# 使用数组进入 n 维

我们非常简要地提到了数组甚至可以在每个位置上保存其他数组。但是，如果一个数组保存了很多保存了其他某种类型的数组的数组，我们如何访问包含数组中的值呢？无论如何，我们为什么需要这个呢？看看下一个示例，多维数组在哪里可以派上用场。

## 多维数组迷你应用程序

让我们做一个简单的多维数组示例。您可以在下载包中找到此示例的工作项目。它在*第十五章*`/Multidimensional Array Example/MainActivity.java`中。

使用空活动模板创建一个项目，并将其命名为“多维数组示例”。

在`onCreate`中的`setContentView`调用之后，添加以下代码，包括声明和初始化一个二维数组（已突出显示）：

```kt
// Random object for generating question numbers
Random randInt = new Random();
// a variable to hold the random value generated
int questionNumber;
// declare and allocate in separate stages for clarity
// but we don't have to
String[][] countriesAndCities;
// Now we have a 2 dimensional array
countriesAndCities = new String[5][2];
// 5 arrays with 2 elements each
// Perfect for 5 "What's the capital city" questions
// Now we load the questions and answers into our arrays
// You could do this with less questions to save typing
// But don't do more or you will get an exception
countriesAndCities [0][0] = "United Kingdom";
countriesAndCities [0][1] = "London";
countriesAndCities [1][0] = "USA";
countriesAndCities [1][1] = "Washington";
countriesAndCities [2][0] = "India";
countriesAndCities [2][1] = "New Delhi";
countriesAndCities [3][0] = "Brazil";
countriesAndCities [3][1] = "Brasilia";
countriesAndCities [4][0] = "Kenya";
countriesAndCities [4][1] = "Nairobi";
```

现在我们使用`for`循环和我们的`Random`对象输出数组的内容。请注意，尽管问题是随机的，但我们始终可以选择正确的答案。在上一个代码之后添加以下代码：

```kt
/*
     Now we know that the country is stored at element 0
     The matching capital at element 1
     Here are two variables that reflect this
*/
int country = 0;
int capital = 1;
// A quick for loop to ask 3 questions
for(int i = 0; i < 3; i++){
   // get a random question number between 0 and 4
   questionNumber = randInt.nextInt(5);
   // and ask the question and in this case just
   // give the answer for the sake of brevity
   Log.i("info", "The capital of "
   +countriesAndCities[questionNumber][country]);

   Log.i("info", "is "
   +countriesAndCities[questionNumber][capital]);
} // end of for loop
```

运行示例，记住屏幕上不会发生任何事情，因为所有输出都将发送到我们在 Android Studio 中的 logcat 控制台窗口。以下是输出：

```kt
info﹕ The capital of USA
info﹕ is Washington
info﹕ The capital of India
info﹕ is New Delhi
info﹕ The capital of United Kingdom
info﹕ is London
```

刚才发生了什么？让我们一块一块地过一遍，这样我们就知道到底发生了什么。

我们创建一个名为`randInt`的`Random`类型的新对象，准备在程序后面生成随机数：

```kt
Random randInt = new Random();

```

有一个简单的`int`变量来保存一个问题编号：

```kt
int questionNumber;
```

在这里我们声明了一个名为`countriesAndCities`的数组数组。外部数组保存数组：

```kt
String[][] countriesAndCities;

```

现在我们在数组中分配空间。第一个外部数组现在可以保存五个数组，每个内部数组可以保存两个字符串：

```kt
countriesAndCities = new String[5][2];
```

现在我们初始化我们的数组来保存国家和它们对应的首都。注意到每一对初始化的外部数组编号保持不变，表明每个国家/首都对在一个内部数组中，一个`String`数组。当然，每个内部数组都保存在外部数组的一个元素中（保存数组的数组）：

```kt
countriesAndCities [0][0] = "United Kingdom";
countriesAndCities [0][1] = "London";
countriesAndCities [1][0] = "USA";
countriesAndCities [1][1] = "Washington";
countriesAndCities [2][0] = "India";
countriesAndCities [2][1] = "New Delhi";
countriesAndCities [3][0] = "Brazil";
countriesAndCities [3][1] = "Brasilia";
countriesAndCities [4][0] = "Kenya";
countriesAndCities [4][1] = "Nairobi";
```

为了使即将到来的`for`循环更清晰，我们声明和初始化`int`变量来表示我们数组中的国家和首都。如果你回顾一下数组初始化，所有的国家都保存在内部数组的位置`0`，所有对应的首都都在位置`1`：

```kt
int country = 0;
int capital = 1;
```

现在我们设置一个`for`循环运行三次。请注意，这不仅仅是访问我们数组的前三个元素；它只是确定我们循环的次数。我们可以让它循环一次或一千次；示例仍然有效：

```kt
for(int i = 0; i < 3; i++){
```

接下来，我们确定要提问的问题 - 或者更具体地说，我们的外部数组的哪个元素。记住，`randInt.nextInt(5)`返回一个 0 到 4 之间的数字 - 这正是我们需要的，因为我们有一个包含五个元素的外部数组，从 0 到 4：

```kt
questionNumber = randInt.nextInt(5);
```

现在我们可以通过输出内部数组中保存的字符串来提问，而这些内部数组又是由前一行随机生成的外部数组保存的：

```kt
   Log.i("info", "The capital of "
   +countriesAndCities[questionNumber][country]);
   Log.i("info", "is "
   +countriesAndCities[questionNumber][capital]);
}//end of for loop
```

值得一提的是，在本书的其余部分我们将不再使用多维数组。所以，如果对这些数组内部的数组还有一点模糊，那没关系。你知道它们存在，知道它们能做什么，如果有必要的话可以重新学习。

## 数组越界异常

当我们尝试访问一个不存在的数组元素时，就会发生数组越界异常。有时编译器会为我们捕捉到它，以防止错误进入工作中的应用程序。例如，看看这段代码：

```kt
int[] ourArray = new int[1000];
int someValue = 1; // Arbitrary value
ourArray[1000] = someValue;
// Won't compile as compiler knows this won't work.
// Only locations 0 through 999 are valid
```

但如果我们做这样的事情呢：

```kt
int[] ourArray = new int[1000];
int someValue = 1;// Arbitrary value
int x = 999;
if(userDoesSomething){
   x++; // x now equals 1000
}
ourArray[x] = someValue;
// Array out of bounds exception if userDoesSomething 
// evaluates to true! This is because we end up referencing
// position 1000 when the array only has positions 0 
// through 999
// Compiler can't spot it. App will crash!
```

我们唯一能避免这个问题的方法是知道这个规则：数组从零开始，到它们的长度-1。我们还可以使用清晰、可读的代码，在这种代码中更容易评估我们所做的事情，并更容易发现问题。

# `ArrayList`

`ArrayList` 就像是一个增强版的普通 Java 数组。它克服了数组的一些缺点，比如需要预先确定大小。它添加了一些有用的方法来使数据易于管理，并且使用了一个更清晰的增强版`for`循环，比普通的`for`循环更容易使用。

让我们看一些使用`ArrayList`实例的代码：

```kt
// Declare a new ArrayList called myList to hold int variables
ArrayList<int> myList;

// Initialize the myList ready for use
myList = new ArrayList<int>();
```

在前面的代码中，我们声明并初始化了一个名为`myList`的新`ArrayList`。我们也可以像这段代码所示的那样一步完成：

```kt
ArrayList<int> myList = new ArrayList<int>();
```

到目前为止还没有特别有趣的东西，所以让我们看看我们实际上可以用`ArrayList`做些什么。这次我们使用一个`String ArrayList`实例：

```kt
// declare and initialize a new ArrayList
ArrayList<String> myList = new ArrayList<String>();
// Add a new String to myList in the next available location
myList.add("Donald Knuth");
// And another
myList.add("Rasmus Lerdorf");
// And another
myList.add("Richard Stallman");
// We can also choose 'where' to add an entry
myList.add(1, "James Gosling");
// Is there anything in our ArrayList?
if(myList.isEmpty()){
   // Nothing to see here
}else{
   // Do something with the data
}
// How many items in our ArrayList?
int numItems = myList.size();
// Now where did I put James Gosling?
int position = myList.indexOf("James Gosling");
```

在前面的代码中，我们看到我们可以在`ArrayList`对象上使用`ArrayList`类的一些非常有用的方法；这些方法列在下面：

+   我们可以添加一个项目（`myList.add`）。

+   在特定位置添加（`myList.add(x, value)`）。

+   检查`ArrayList`是否为空（`myList.isEmpty`）。

+   看看它有多少元素（`myList.size()`）。

+   获取给定项目的当前位置（`myList.indexOf`）。

注意

`ArrayList`类中甚至还有更多的方法，你可以在这里阅读：http://docs.oracle.com/javase/7/docs/api/java/util/ArrayList.html。到目前为止，我们已经看到的足以完成本书。 

有了所有这些功能，我们现在只需要一种方法来动态处理`ArrayList`实例。

## 增强型 for 循环

这是增强型`for`循环的条件：

```kt
for (String s : myList)
```

前面的例子会逐个遍历`myList`中的所有项目。在每一步中，`s`会保存当前的`String`值。

因此，这段代码将在控制台上打印出我们上一节`ArrayList`代码示例中的所有杰出程序员：

```kt
for (String s : myList){
   Log.i("Programmer: ","" + s);
}
```

我们也可以使用增强型`for`循环来处理常规数组：

```kt
int [] anArray = new int [];
// We can initialize arrays quickly like this
anArray {0, 1, 2, 3, 4, 5}
for (int s : anArray){
   Log.i("Contents = ","" + s);
}
```

还有一个即将到来的新闻快讯！

# 数组和 ArrayList 实例是多态的

我们已经知道我们可以将对象放入数组和`ArrayList`中。但是多态意味着它们可以处理多个不同类型的对象，只要它们有一个共同的父类型，都可以放在同一个数组或`ArrayList`中。

在*第十章**，面向对象编程*中，我们学到多态意味着*不同的形式*。但在数组和`ArrayList`的上下文中，这对我们意味着什么呢？

简化到最简单的形式：任何子类都可以作为使用超类的代码的一部分。

例如，如果我们有一个`Animal`实例的数组，我们可以将任何一个是`Animal`子类的对象放入`Animal`数组中 - 也许是`Cat`和`Dog`实例。

这意味着我们可以编写更简单、更易理解、更易更改的代码：

```kt
// This code assumes we have an Animal class
// And we have a Cat and Dog class that extends Animal
Animal myAnimal =  new Animal();
Dog myDog = new Dog();
Cat myCat = new Cat();
Animal [] myAnimals = new Animal[10];
myAnimals[0] = myAnimal; // As expected
myAnimals[1] = myDog; // This is OK too
myAnimals[2] = myCat; // And this is fine as well
```

此外，我们可以为超类编写代码，并依赖于这样一个事实，即使它被子类化多少次，在一定的参数范围内，代码仍然可以工作。让我们继续我们之前的例子：

```kt
// 6 months later we need elephants
// with its own unique aspects
// If it extends Animal we can still do this
Elephant myElephant = new Elephant();
myAnimals[3] = myElephant; // And this is fine as well
```

但是当我们从多态数组中移除一个对象时，我们必须记得将其转换为我们想要的类型：

```kt
Cat newCat = (Cat) myAnimals[2];
```

我们刚刚讨论的对`ArrayList`也适用。有了这些新的工具包，包括数组、`ArrayList`，以及它们的多态性，我们可以继续学习一些更多的 Android 类，这些类很快就会用到我们的 Note to Self 应用程序中。

# 更多的 Java 集合 - 了解 Java HashMap

Java 的`HashMap`很棒。它是 Java 集合框架的一部分，也是我们在下一章中将在 Note to Self 项目中使用的`ArrayList`类的一种近亲。它们基本上封装了一些有用的数据存储技术，否则对我们来说可能会相当技术性。

我觉得值得先看一下`HashMap`。假设我们想要存储角色扮演游戏中许多角色的数据，每个不同的角色都由`Character`类型的对象表示。

我们可以使用一些我们已经了解的 Java 工具，比如数组或`ArrayList`。Java 的`HashMap`也类似于这些东西，但是使用`HashMap`，我们可以为每个`Character`对象提供一个唯一的键/标识符，并使用该键/标识符访问任何这样的对象。

哈希术语来自于将我们选择的键/标识符转换为`HashMap`类内部使用的东西的过程。这个过程被称为哈希。

我们选择的键/标识符可以访问任何我们的`Character`实例。在`Character`类的情况下，一个好的键/标识符候选者可能是角色的名字。

每个键/标识符都有一个相应的对象；在这种情况下，它是`Character`类型的。这被称为键值对。

我们只需给`HashMap`一个键，它就会给我们相应的对象。不需要担心我们存储角色的索引是哪个，无论是 Geralt、Ciri 还是 Triss；只需将名字传递给`HashMap`，它就会为我们完成工作。

让我们看一些例子。你不需要输入任何代码 - 只需熟悉它的工作原理。

我们可以声明一个新的`HashMap`来保存键和`Character`实例，就像这样的代码：

```kt
Map<String, Character> characterMap;
```

前面的代码假设我们已经编写了一个名为`Character`的类。

我们可以像这样初始化`HashMap`：

```kt
characterMap = new HashMap();
```

我们可以像这样添加一个新的键和其关联的对象：

```kt
characterMap.put("Geralt", new Character());
```

我们也可以使用这个：

```kt
characterMap.put("Ciri", new Character());
```

我们也可以使用这个：

```kt
characterMap.put("Triss", new Character());
```

注意

所有示例代码都假设我们可以在其他地方赋予`Character`实例它们独特的属性，以反映它们的内部差异。

我们可以像这样从`HashMap`中检索条目：

```kt
Character ciri = characterMap.get("Ciri");
```

或者我们可以直接使用`Character`类的方法，就像这样：

```kt
characterMap.get("Geralt").drawSilverSword();
// Or maybe call some other hypothetical method
characterMap.get("Triss").openFastTravelPortal("Kaer Morhen");
```

先前的代码调用了假设的`Character`类上的`drawSilverSword`和`openFastTravelPortal`方法。

注意

`HashMap`类也有很多有用的方法，就像`ArrayList`一样。在这里查看`HashMap`的官方 Java 页面：[`docs.oracle.com/javase/tutorial/collections/interfaces/map.html`](https://docs.oracle.com/javase/tutorial/collections/interfaces/map.html)。

让我们谈谈“Note to Self”应用程序。

# “Note to Self”应用程序

尽管我们学到了很多，但我们还没有准备好为“Note to Self”应用程序应用解决方案。我们可以更新我们的代码，将大量的笔记存储在`ArrayList`实例中，但在这之前，我们还需要一种方法来在 UI 中显示`ArrayList`的内容。例如，将整个`ArrayList`的内容放入`TextView`小部件中看起来并不好。

解决方案是适配器和一个名为`RecyclerView`的特殊 UI 布局。我们将在下一章中介绍它们。

# 常见问题

1.  一个只能进行*真实*计算的计算机如何可能生成真正的随机数？

实际上，计算机无法创建真正随机的数字，但`Random`类使用一个种子来产生一个在严格的统计检验下会被认为是真正随机的数字。要了解更多关于种子和生成随机数的信息，请查看这篇文章：[`en.wikipedia.org/wiki/Random_number_generation`](https://en.wikipedia.org/wiki/Random_number_generation)。

# 总结

在本章中，我们看了如何使用简单的 Java 数组来存储大量数据，只要它们是相同类型的。我们还使用了`ArrayList`类，它类似于一个带有大量额外功能的数组。此外，我们发现数组和`ArrayList`实例都是多态的，这意味着一个数组（或`ArrayList`）可以容纳多个不同的对象，只要它们都是从同一个父类派生的。

此外，我们还了解了`HashMap`类，它也是一种数据存储解决方案，但允许以不同的方式访问。

在下一章中，我们将学习`Adapter`和`RecyclerView`类，将我们的理论付诸实践，并增强“Note to Self”应用程序。
