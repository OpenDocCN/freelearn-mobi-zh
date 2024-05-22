# 第十二章：将我们的 Kotlin 连接到 UI 和可空性

通过本章的结束，我们的 Kotlin 代码和 XML 布局之间的缺失链接将被完全揭示，让我们有能力像以前一样向布局添加各种小部件和 UI 功能，但这一次我们将能够通过我们的代码来控制它们。

在本章中，我们将控制一些简单的 UI 元素，比如`Button`和`TextView`，在下一章中，我们将进一步操作一系列 UI 元素。

为了让我们理解发生了什么，我们需要更多地了解应用程序中的内存，特别是其中的两个区域-**堆栈**和**堆**。

在本章中，我们将涵盖以下主题：

+   Android UI 元素也是类

+   垃圾回收

+   我们的 UI 在堆上

+   更多的多态性

+   可空性- val 和 var 重新审视

+   转换为不同类型

准备让您的 UI 活起来。

# 所有的 Android UI 元素也是类

当我们的应用程序运行并且从`onCreate`函数中调用`setContentView`函数时，布局会从 XML UI 中**膨胀**，并作为可用对象加载到内存中。它们存储在内存的一个部分，称为堆。

但是这个堆在哪里？我们在代码中肯定看不到 UI 实例。我们怎么才能得到它们呢？

每个 Android 设备内部的操作系统都会为我们的应用程序分配内存。此外，它还将不同类型的变量存储在不同的位置。

我们在函数中声明和初始化的变量存储在称为堆栈的内存区域。我们已经知道如何使用简单的表达式在堆栈上操作变量。所以，让我们再谈谈堆。

### 注意

重要事实：所有类的对象都是引用类型变量，只是指向存储在堆上的实际对象的引用-它们并不是实际的对象。

把堆想象成仓库的另一个区域。堆有大量的地板空间用于奇形怪状的物体，用于较小物体的货架，以及许多长排的小尺寸隔间等。这就是对象存储的地方。问题是我们无法直接访问堆。把它想象成仓库的受限区域。你实际上不能去那里，但你可以*引用*那里存储的东西。让我们看看引用变量到底是什么。

它是一个我们通过引用引用和使用的变量。引用可以宽松地但有用地定义为地址或位置。对象的引用（地址或位置）在堆栈上。

因此，当我们使用点运算符时，我们正在要求操作系统在特定位置执行任务，这个位置存储在引用中。

### 提示

引用变量就是这样-一个引用。它们是访问和操作对象（属性和函数）的一种方式，但它们并不是实际的对象本身。

为什么我们会想要这样的系统？给我我的对象就放在堆栈上！这就是为什么。

## 快速休息一下，扔掉垃圾

整个堆栈和堆的作用是什么。

正如我们所知，操作系统会为我们跟踪所有的对象，并将它们存储在我们仓库的一个专门区域，称为堆。在我们的应用程序运行时，操作系统会定期扫描堆栈，我们仓库的常规货架，并匹配堆上的对象的引用。它发现的任何没有匹配引用的对象，都会被销毁。或者，用正确的术语来说，它进行**垃圾回收**。

想象一辆非常有洞察力的垃圾车穿过我们的堆，扫描物体以匹配参考（在堆栈上）。没有参考意味着它被垃圾回收了。

如果一个对象没有相关的引用变量，我们无法对其进行任何操作，因为我们无法访问它/引用它。垃圾收集系统通过释放未使用的内存帮助我们的应用程序更有效地运行。

如果这个任务留给我们来完成，我们的应用程序将会更加复杂。

因此，函数内声明的变量是局部的，位于堆栈上，只能在声明它们的函数内部可见。一个属性（对象的属性）位于堆上，可以在任何有引用的地方引用它，如果访问修饰符（封装）允许的话。

## 关于堆栈和堆的七个有用的事实

让我们快速看看我们对堆栈和堆学到了什么：

+   你不会删除对象，而是操作系统在认为合适的时候发送垃圾收集器。通常情况下，当对象没有活动引用时，垃圾收集器会进行清理。

+   变量位于堆栈上，只能在声明它们的特定函数内部可见。

+   属性位于堆上（与其对象/实例一起），但是对象/实例的引用（其地址）是堆栈上的局部变量。

+   我们控制着堆栈中的内容。我们可以使用堆上的对象，但只能通过引用它们。

+   堆由垃圾收集器保持清晰和最新。

+   当不再有有效引用指向对象时，对象将被垃圾收集。因此，当引用变量从堆栈中移除时，与之相关的对象就可以进行垃圾收集。当操作系统决定时机合适（通常非常迅速），它将释放 RAM 内存以避免耗尽。

+   如果我们设法引用一个不存在的对象，我们将会得到一个**NullPointerException**错误，应用程序将崩溃。Kotlin 的一个主要特性是它保护我们免受这种情况的发生。在 Kotlin 试图改进的 Java 中，**NullPointerException 错误**是应用程序崩溃的最常见原因。我们将在本章末尾附近的*Nullability –* `val` *and* `var` *revisited*部分学习更多关于 Kotlin 如何帮助我们避免**NullPointerException**错误的内容。

让我们继续看看这些信息对我们控制 UI 方面有什么帮助。

## 那么，这个堆究竟如何帮助我？

在 XML 布局中设置了`id`属性的任何 UI 元素都可以从堆中检索其引用并使用，就像我们在前两章中编写和声明自己的类一样。

如果我们使用基本活动模板创建一个项目（随意这样做，但你不需要这样做），将一个按钮拖到 UI 上，推断出约束，并在模拟器上运行应用程序。然后我们将得到下一个截图中所见的内容：

![那么，这个堆究竟如何帮助我？](img/B12806_12_01.jpg)

这就是我们应该从前五章中已经看到的内容可以期待的。如果我们将这行代码添加到`onCreate`函数中，那么将会发生一些有趣的事情：

```kt
button.text = "WOO HOO!"
```

再次运行应用程序并观察按钮的变化：

![那么，这个堆究竟如何帮助我？](img/B12806_12_02.jpg)

我们已经改变了按钮上的文本。

### 提示

此时，如果您之前使用 Java 编写 Android 应用程序，您可能想躺下几分钟，思考从现在开始生活将会变得多么容易。

这非常令人兴奋，因为它显示我们可以从我们的布局中获取一大堆东西的引用。然后我们可以开始使用这些对象由 Android API 提供的所有函数和属性。

代码中的`button`实例是指 XML 布局中`Button`小部件的`id`。我们代码中的`text`实例然后指的是`Button`类的`text`属性，我们代码中的`= "WOO HOO!"`文本使用了`text`属性的 setter 来改变它所持有的值。

### 提示

如果`Button`类（或其他 UI 元素）的`id`值不同，那么我们需要相应地调整我们的代码。

如果你认为在十一章之后，我们终于要开始在 Android 上做一些好玩的事情，那么你是对的！

让我们了解 OOP 的另一个方面，然后我们将能够构建迄今为止最功能强大的应用程序。

# Kotlin 接口

接口就像一个类。哦！这里没有什么复杂的。但是，它就像一个始终是抽象的类，只有抽象函数。

我们可以将接口看作是一个完全抽象的类，其所有函数和属性都是抽象的。当属性是抽象的时，它不持有值。它没有属性的后备字段。然而，当另一个类实现（使用）接口时，它必须重写属性，因此提供用于存储值的后备字段。

简而言之，接口是无状态的类。它们提供了一个没有任何数据的实现模板。

好吧，你大概能理解抽象类，因为至少它可以在其函数中传递一些功能，并在其属性中传递一些状态，这些状态不是抽象的，并且作为多态类型。

但是，说真的，这个界面似乎有点毫无意义。让我们看一个最简单的接口示例，然后我们可以进一步讨论。

定义接口，我们输入以下内容：

```kt
interface SomeInterface { 

   val someProperty: String 
   // Perhaps more properties

   fun someFunction() 
   // Perhaps more functions
   // With or without parameters
   // and return types
}
```

接口的函数没有主体，因为它们是抽象的，但它们仍然可以有返回类型和参数。

要使用接口，我们在类声明后使用相同的`:`语法：

```kt
class SomeClass() : SomeInterface{ 

   // Overriding any properties
   // is not optional
   // It is an obligation for a class
   // that uses the interface
   override val someProperty: String = "Hello" 

   override fun someFunction() { 
      // This implementation is not optional
      // It is an obligation for a class
      // that uses the interface
   } 
}
```

在前面的代码中，属性和函数已在实现接口的类中被重写。编译器强制接口的用户这样做，否则代码将无法编译。

如果您同时从一个类继承并实现一个或多个接口，那么超类就会简单地放入接口的列表中。为了清楚地表明不同的关系，惯例是将超类放在列表的第一位。然而，编译器并不要求这样做。

这使我们能够在完全不相关的继承层次结构中使用多个不同对象的多态性。如果一个类实现了一个接口，整个东西就可以被传递或用作它就像是那个东西一样，因为它就是那个东西。它是多态的（多种形式）。

我们甚至可以让一个类同时实现多个不同的接口。只需在每个接口之间添加逗号，并确保重写所有必要的函数。

在本书中，我们将更频繁地使用 Android API 的接口，而不是编写我们自己的接口。在下一节中，我们将使用`OnClickListener`接口。

许多东西可能想要在被点击时知道，比如`Button`小部件或`TextView`小部件。因此，使用接口，我们不需要为每种类型的 UI 元素单独编写不同的函数。

让我们一起看看接口在同时连接我们的 Kotlin 代码和 UI 时的作用。

# 使用按钮和 TextView 小部件从我们的布局中，借助接口的一点帮助

要跟随这个项目，创建一个新的 Android Studio 项目，将其命名为`Kotlin Meet UI`，并选择**Empty Activity**模板。您可以在`Chapter12/Kotlin Meet UI`文件夹中找到代码和 XML 布局代码。

首先，让我们通过以下步骤构建一个简单的 UI：

1.  在 Android Studio 的编辑窗口中，切换到`activity_main.xml`，确保你在**Design**选项卡上。

1.  删除自动生成的`TextView`，即那个写着“Hello world!”的。

1.  在布局的顶部中心添加一个**TextView**小部件。

1.  将其**text**属性设置为`0`，其`id`属性设置为`txtValue`，其`textSize`设置为`40sp`。请特别注意`id`值的大小写。它的`V`是大写的。

1.  现在，将六个按钮拖放到布局上，使其看起来有点像下面的图表。确切的布局并不重要：![使用按钮和 TextView 小部件从我们的布局中，借助接口的一点帮助](img/B12806_12_03.jpg)

1.  当布局达到您想要的效果时，单击**Infer Constraints**按钮以约束所有 UI 项。

1.  依次双击每个按钮（从左到右，然后从上到下），并设置`text`和`id`属性，如下表所示：

| `text`属性 | `id`属性 |
| --- | --- |
| `add` | `btnAdd` |
| `take` | `btnTake` |
| `grow` | `btnGrow` |
| `shrink` | `btnShrink` |
| `hide` | `btnHide` |
| `reset` | `btnReset` |

完成后，您的布局应如下屏幕截图所示：

![使用按钮和 TextView 小部件从我们的布局中，借助接口的一点帮助](img/B12806_12_04.jpg)

按钮上的精确位置和文本并不是非常重要，但是给`id`属性赋予的值必须相同。原因是我们将使用这些`id`值从我们的 Kotlin 代码中获取对此布局中的`Button`实例和`TextView`实例的引用。

切换到编辑器中的**MainActivity.kt**选项卡，并找到以下行：

```kt
class MainActivity : AppCompatActivity(){
```

现在将代码行修改为以下内容：

```kt
class MainActivity : AppCompatActivity,
   View.OnClickListener{
```

在输入时，将会弹出一个列表，询问您要选择要实现的接口。选择**OnClickListener (android.view.view)**，如下一屏幕截图所示：

![使用按钮和 TextView 小部件从我们的布局中，借助接口的一点帮助](img/B12806_12_05.jpg)

### 提示

您需要导入`View`类。确保在继续下一步之前执行此操作，否则将会得到混乱的结果：

```kt
import android.view.View
```

注意到`MainActivity`声明被红色下划线标出，显示出错误。现在，因为我们已经将`MainActivity`添加为接口`OnClickListener`，我们必须实现`OnClickListener`的抽象函数。该函数称为`onClick`。当我们添加该函数时，错误将消失。

我们可以通过在包含错误的代码上任意左键单击，然后使用键盘组合*Alt* +*Enter*来让 Android Studio 为我们添加。左键单击**Implement members**，如下一屏幕截图所示：

![使用按钮和 TextView 小部件从我们的布局中，借助接口的一点帮助](img/B12806_12_06.jpg)

现在，左键单击**OK**以确认我们希望 Android Studio 添加`onClick`方法/函数。错误已经消失，我们可以继续添加代码。我们还有一个`onClick`函数，很快我们将看到我们将如何使用它。

### 注意

术语上的一个快速说明。**方法**是在类中实现的函数。Kotlin 允许程序员独立于类实现函数，因此所有方法都是函数，但并非所有函数都是方法。我选择在本书中始终将所有方法称为函数。有人认为方法可能是一个更精确的术语，但在本书的上下文中，两者都是正确的。如果您愿意，可以将类中的函数称为方法。

现在，在类声明内部但在任何函数之外/之前添加以下属性：

```kt
class MainActivity : AppCompatActivity(), View.OnClickListener {

 // An Int property to hold a value
 private var value = 0

```

我们声明了一个名为`value`的`Int`属性，并将其初始化为`0`。请注意，它是一个`var`属性，因为我们需要更改它。

接下来，在`onCreate`函数内，添加以下六行代码：

```kt
// Listen for all the button clicks
btnAdd.setOnClickListener(this)
btnTake.setOnClickListener(this)
txtValue.setOnClickListener(this)
btnGrow.setOnClickListener(this)
btnShrink.setOnClickListener(this)
btnReset.setOnClickListener(this)
btnHide.setOnClickListener(this)
```

### 提示

使用*Alt* +*Enter*键组合从`activity_main.xml`布局文件中导入所有`Button`和`TextView`实例。或者，手动添加以下导入语句：

```kt
import kotlinx.android.synthetic.main.activity_main.* 
```

上述代码设置了我们的应用程序以侦听布局中按钮的点击。每行代码都执行相同的操作，但是在不同的按钮上。例如，`btnAdd`指的是我们布局中`id`属性值为`btnAdd`的按钮，`btnTake`指的是我们布局中`id`属性值为`btnTake`的按钮。

然后每个按钮实例调用自身的`setOnClickListener`函数。传入的参数是`this`。从第十章中记住，*面向对象编程*，`this`指的是代码所在的当前类。因此，在前面的代码中，`this`指的是`MainActivity`。

`setOnClickListener`函数设置我们的应用程序调用`OnClickListener`接口的`onClick`函数。现在，每当我们的按钮之一被点击，`onClick`函数将被调用。所有这些都是因为`MainActivity`实现了`OnClickListener`接口。

如果你想验证这一点，暂时从类声明的末尾删除`View.OnClickListener`代码，我们的代码将突然充满一片红色的错误。这是因为`this`不再是`OnCLickListener`类型，因此无法传递给各个按钮的`setOnClickListener`函数，`onClick`函数也会显示错误，因为编译器不知道我们试图覆盖什么。接口是使所有这些功能结合在一起的关键。

### 提示

如果之前删除了`View.OnClickListener`，请在类声明的末尾替换它。

现在，滚动到 Android Studio 在我们实现`OnClickListener`接口后添加的`onClick`函数。添加`Float size`变量声明和一个空的`when`块，使其看起来像下面的代码。要添加的新代码已经突出显示。在下一个代码中还有一件事需要注意和实现。当`onClick`函数由 Android Studio 自动生成时，在`v: View?`参数后添加了一个问号。删除问号，如下面的代码所示：

```kt
override fun onClick(v: View) {
 // A local variable to use later
 val size: Float

 when (v.id) {

 }
}
```

记住，`when`将检查匹配表达式的值。`when`条件是`v.id`。`v`变量被传递给`onClick`函数，`v.id`标识了被点击的按钮的`id`属性。它将匹配布局中我们按钮的`id`。

### 注意

如果你对我们删除的那个奇怪的问号感到好奇，它将在下一节中解释：*可空性——val 和 var 重新讨论*。

接下来我们需要处理每个按钮的操作。将下面的代码块添加到`when`表达式的大括号内，然后我们将讨论它。首先尝试自己解决代码，你会惊讶地发现我们已经理解了多少。

```kt
R.id.btnAdd -> {
   value++
   txtValue.text = "$value"
}

R.id.btnTake -> {
   value--
   txtValue.text = "$value"
}

R.id.btnReset -> {
   value = 0
   txtValue.text = "$value"
}

R.id.btnGrow -> {
   size = txtValue.textScaleX
   txtValue.textScaleX = size + 1
}

R.id.btnShrink -> {
   size = txtValue.textScaleX
   txtValue.textScaleX = size - 1
}

R.id.btnHide -> 
   if (txtValue.visibility 
            == View.VISIBLE) {
   // Currently visible so hide it
   txtValue.visibility = View.INVISIBLE

   // Change text on the button
   btnHide.text = "SHOW"

} else {
   // Currently hidden so show it
   txtValue.visibility = View.VISIBLE

   // Change text on the button
   btnHide.text = "HIDE"
}
```

以下是代码的第一行：

```kt
override fun onClick(v: View) {
```

`View`是`Button`、`TextView`等的父类。因此，也许正如我们所期望的那样，使用`v.id`将返回被点击的 UI 小部件的`id`属性，并触发首次调用`onClick`。

接下来，我们需要为我们想要响应的每个`Button` id 值提供一个`when`语句（和一个适当的操作）。以下是代码的一部分，以供您参考：

```kt
when (v.id) {

}
```

再看一下代码的下一部分：

```kt
R.id.btnAdd -> {
   value++
   txtValue.text = "$value"
}

R.id.btnTake -> {
   value--
   txtValue.text = "$value"
}

R.id.btnReset -> {
   value = 0
   txtValue.text = "$value"
}
```

前面的代码是前三个`when`分支。它们处理`R.id.btnAdd`、`R.id.btnTake`和`R.id.btnReset`。

`R.id.btnAdd`分支中的代码简单地增加了`value`变量，然后做了一些新的事情。

它设置了`txtValue`对象的`text`属性。这样做的效果是使这个`TextView`显示存储在`value`中的任何值。

**TAKE**按钮（`R.id.btnTake`）做的事情完全相同，只是从`value`中减去 1，而不是加 1。

`when`语句的第三个分支处理**RESET**按钮，将`value`设置为零，并再次更新`txtValue`的`text`属性。

在执行任何`when`分支的末尾，整个`when`块都会退出，`onClick`函数返回，生活恢复正常——直到用户的下一次点击。

让我们继续检查`when`块的下两个分支。以下是为了方便您再次查看：

```kt
R.id.btnGrow -> {
   size = txtValue.textScaleX
   txtValue.textScaleX = size + 1
}

R.id.btnShrink -> {
   size = txtValue.textScaleX
   txtValue.textScaleX = size - 1
}
```

接下来的两个分支处理我们 UI 中的**SHRINK**和**GROW**按钮。我们可以从 id 的`R.id.btnGrow`值和`R.id.btnShrink`值确认这一点。新的更有趣的是`TextView`类的 getter 和 setter 在按钮上使用。

`textScaleX`属性的 getter 返回所使用对象中文本的水平比例。我们可以看到它所使用的对象是我们的`TextView txtValue`实例。代码`size =`在代码行的开头将返回的值分配给我们的`Float`变量`size`。

每个`when`分支中的下一行代码使用`textScaleX`属性的 setter 来改变文本的水平比例。当按下**GROW**按钮时，比例设置为`size + 1`，当按下**SHRINK**按钮时，比例设置为`size - 1`。

总体效果是允许这两个按钮通过每次点击来放大和缩小`txtValue`中的文本，比例为`1`。

让我们看一下`when`代码的最后一个分支。以下是为了方便您再次查看：

```kt
R.id.btnHide -> 
   if (txtValue.visibility == View.VISIBLE) {
      // Currently visible so hide it
      txtValue.visibility = View.INVISIBLE

      // Change text on the button
      btnHide.text = "SHOW"

   } else {
      // Currently hidden so show it
      txtValue.visibility = View.VISIBLE

      // Change text on the button
      btnHide.text = "HIDE"
   }
```

前面的代码需要一点解释，所以让我们一步一步来。首先，在`when`分支内嵌套了一个`if`-`else`表达式。以下是`if`部分：

```kt
if (txtValue.visibility == View.VISIBLE)
```

要评估的条件是`txtValue.visibility == View.VISIBLE`。在`==`运算符之前的部分使用`visibility`属性的 getter 返回描述`TextView`当前是否可见的值。返回值将是`View`类中定义的三个可能的常量值之一。它们是`View.VISIBLE`，`View.INVISIBLE`和`View.GONE`。

如果`TextView`在 UI 上对用户可见，则 getter 返回`View.VISIBLE`，条件被评估为`true`，并且执行`if`块。

在`if`块内，我们使用`visibility`属性的 setter 将其对用户不可见，使用`View.INVISIBLE`值。

除此之外，我们使用`text`属性的 setter 将`btnHide`对象上的文本更改为**SHOW**。

在`if`块执行后，`txtValue`将不可见，并且我们的 UI 上有一个按钮显示**SHOW**。当用户在这种状态下点击它时，`if`语句将为 false，`else`块将执行。在`else`块中，我们将情况反转。我们将`txtValue`对象的`visibility`属性设置回`View.VISIBLE`，并将`btnHide`上的`text`属性设置回**HIDE**。

如果有任何不清楚的地方，只需输入代码，运行应用程序，然后在看到它实际运行后再回顾一下最后的代码和解释。

我们已经准备好 UI 和代码，现在是时候运行应用程序并尝试所有按钮了。请注意，**ADD**和**TAKE**按钮会分别将`value`的值增加或减少一，并在`TextView`中显示结果。在下一张图片中，我点击了**ADD**按钮三次：

![使用按钮和 TextView 小部件从我们的布局中获得帮助](img/B12806_12_07.jpg)

请注意，**SHRINK**和**GROW**按钮增加了文本的宽度，**RESET**将`value`变量设置为零，并在`TextView`上显示它。在下面的截图中，我点击了**GROW**按钮八次：

![使用按钮和 TextView 小部件从我们的布局中获得帮助](img/B12806_12_08.jpg)

最后，**HIDE**按钮不仅隐藏`TextView`，还将其自身文本更改为**SHOW**，如果再次点击，则确实会重新显示`TextView`。

### 提示

我不会打扰你，向你展示一个隐藏的东西的图片。一定要在模拟器中尝试该应用，并跟着书本一起学习。如果你想知道`View.INVISIBLE`和`View.GONE`之间的区别，`INVISIBLE`只是隐藏了对象，但当使用`GONE`时，布局的行为就好像对象从未存在过一样，因此可能会影响剩余 UI 的布局。将代码行从`INVISIBLE`更改为`GONE`，并运行应用程序以观察差异。

请注意，在这个应用程序中不需要`Log`或`Toast`，因为我们最终是使用我们的 Kotlin 代码来操作 UI。

# 可空性 - val 和 var 重温

当我们用`val`声明一个类的实例时，并不意味着我们不能改变属性中保存的值。决定我们是否可以重新分配属性中保存的值的是属性本身是`val`还是`var`。

当我们用`val`声明一个类的实例时，这只意味着我们不能重新分配另一个实例给它。当我们想要重新分配一个实例时，我们必须用`var`声明它。以下是一些例子：

```kt
val someInstance = SomeClass()
someInstance.someMutableProperty = 1// This was declared as var
someInstance.someMutableProperty = 2// So we can change it

someInstance.someImutableProperty = 1
// This was declared with val. ERROR!
```

在前面的假设代码中，声明了一个名为`someInstance`的实例，它是`SomeClass`类型。它被声明为`val`。接下来的三行代码表明，如果它的属性被声明为`var`，我们可以更改这些属性，但是，正如我们已经学到的，当属性被声明为`val`时，我们不能更改它。那么，用`val`或`var`声明一个实例到底意味着什么？看看下面的假设代码：

```kt
// Continued from previous code
// Three more instances of the same class
val someInstance2 = SomeClass() // Immutable
val someInstance3 = SomeClass()// Immutable
var someInstance4 = SomeClass() // Mutable

// Let's change these instances around— or try to
someInstance = someInstance2 
// Error cannot reassign, someInstance is immutable

someInstance2 = someInstance3 // Error someInstance2 is immutable
someInstance3 = someInstance4 // Error someInstance3 is immutable

// However,
someInstance4 = someInstance 
// No problem! someInstance4 and someInstance are now the
// same object— refer to the same object on the heap

// Sometime in the future…
someInstance4 = someInstance3 // No problem
// Sometime in the future…
someInstance4 = someInstance2 // No problem
// Sometime in the future…
// I need a new SomeClass instance

someInstance4 = SomeClass() // No problem
// someInstance4 now uniquely refers 
// to a new object on the heap
```

前面的代码清楚地表明，当一个实例是`val`时，它不能被重新分配到堆上的另一个对象，但当它是`var`时可以。实例是`val`还是`var`并不影响其属性是`val`还是`var`。

我们已经学到，当讨论属性时，如果我们不需要改变一个值，最好的做法是声明为`val`。对于对象/实例也是如此。如果我们不需要重新分配一个实例，我们应该将其声明为`val`。

## 空对象

当我们将对象或属性声明为`var`时，我们有选择不立即初始化它，有时这正是我们需要的。当我们不初始化一个对象时，它被称为**空引用**，因为它不指向任何东西。我们经常需要声明一个对象，但直到我们的应用程序运行时才初始化它，但这可能会引起问题。看看更多的假设代码：

```kt
var someInstance5: SomeClass
someInstance5.someMutableProperty = 3
```

在前面的代码中，我们声明了一个名为`someInstance5`的`SomeClass`的新实例，但我们没有初始化它。现在，看看这个截图，看看当我们在初始化之前尝试使用这个实例时会发生什么：

![空对象](img/B12806_12_09.jpg)

编译器不允许我们这样做。当我们需要在程序执行期间初始化一个实例时，我们必须明确地将其初始化为`null`，以便编译器知道这是有意的。此外，当我们将实例初始化为`null`时，我们必须使用**可空运算符**。看看下一个修复刚才问题的假设代码：

```kt
var someInstance5: SomeClass? = null
```

在前面的代码中，可空运算符用在`SomeClass?`类型的末尾，并且实例被初始化为`null`。当我们使用可空运算符时，我们可以将实例视为不同的类型 - *SomeClass 可空*，而不仅仅是*SomeClass*。

然后，我们可以在代码中需要的时候初始化实例。我们将在第十四章中看到一些真实的例子，*Android 对话框窗口*，以及本书的其余部分，但现在，这是我们可能有条件地初始化这个空对象的一种假设方式：

```kt
var someBoolean = true
// Program execution or user input might change 
// the value of someBoolean 

if(someBoolean) {
   someInstance5 = someInstance
}else{
   someInstance5 = someInstance2
}
```

然后，我们可以像平常一样使用`someInstance5`。

### 安全调用运算符

有时我们需要更灵活性。假设我们需要`someInstance5`中一个属性的值，但无法保证它已经初始化？在这种情况下，我们可以使用**安全调用**`?`运算符：

```kt
val someInt = someInstance5?.someImmutableProperty
```

在前面的代码中，如果`someInstance5`已经初始化，则将使用`someImmutable`属性中存储的值来初始化`someInt`。如果尚未初始化，则`someInt`将被初始化为 null。因此，请注意，`someInt`被推断为可空类型`Int`，而不是普通的`Int`。

### 非空断言

会出现一些情况，我们无法在编译时保证实例已初始化，并且无法让编译器相信它会被初始化。在这种情况下，我们必须使用**非空断言**`!!`运算符来断言对象不为空。考虑以下代码：

```kt
val someBoolean = true
if(someBoolean) {
   someInstance5 = someInstance
}

someInstance5!!.someMutableProperty = 3
```

在前面的代码中，`someInstance5`可能尚未初始化，我们使用了非空断言运算符，否则代码将无法编译。

还要注意，如果我们编写了一些错误的逻辑，并且在使用时实例仍然为空，那么应用程序将崩溃。实际上，应尽量少地使用`!!`运算符，而应优先使用安全调用运算符。

## 回顾空值性

空值性还有更多内容，我们还没有涵盖到。讨论不同运算符的不同用法可能需要写很多页，而且还有更多的运算符。关键是，Kotlin 旨在帮助我们尽可能避免由于空对象而导致的崩溃。然而，看到可空类型、安全调用运算符和非空断言运算符的实际应用要比理论更有教育意义。在本书的其余部分中，我们将经常遇到这三种情况，希望它们的上下文会比它们的理论更有教育意义。

# 总结

在本章中，我们最终在代码和 UI 之间有了一些真正的交互。原来，每当我们向 UI 添加一个小部件时，我们都在添加一个我们可以在代码中引用的类的 Kotlin 实例。所有这些对象都存储在一个称为堆的内存区域中，与我们自己的类的任何实例一起。

现在我们已经可以学习并使用一些更有趣的小部件。我们将在下一章第十三章中看到很多这样的小部件，*给 Android 小部件赋予生命*，并且在本书的其余部分中我们还将继续介绍新的小部件。
