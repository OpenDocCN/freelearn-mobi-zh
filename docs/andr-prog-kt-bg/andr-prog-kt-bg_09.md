# 第九章。Kotlin 函数

函数是我们应用程序的构建模块。我们编写执行特定任务的函数，然后在需要执行该特定任务时调用它们。由于我们应用程序中需要执行的任务将会非常多样化，我们的函数需要适应这一点并且非常灵活。Kotlin 函数非常灵活，比其他与 Android 相关的语言更灵活。因此，我们需要花费一个完整的章节来学习它们。函数与面向对象编程密切相关，一旦我们理解了函数的基础知识，我们就能够很好地掌握更广泛的面向对象编程学习。

这是本章的内容：

+   函数基础和回顾

+   函数返回类型和 return 关键字

+   单表达式函数

+   默认参数

+   更多与函数相关的主题

我们已经对函数有了一些了解，所以需要回顾一下。

# 函数基础和回顾

我们已经看到并使用了函数。一些是由 Android API 为我们提供的，比如`onCreate`和其他生命周期函数。

我们自己编写了其他函数；例如，`topClick`和`bottomClick`。但是，我们还没有适当地解释它们，函数还有更多我们还没有看到的内容。

### 注意

您经常会听到另一个与函数密切相关且几乎与函数同义的术语。如果您之前学过 Java 或其他面向对象的语言，情况就是如此。我指的是**方法**。从技术角度来看，方法和函数之间的区别很少重要，部分区别在于我们的代码中声明函数/方法的位置。如果您想在编程上正确，可以阅读这篇文章，其中深入探讨了这个问题，并提供了多种观点：

[`stackoverflow.com/questions/155609/whats-the-difference-between-a-method-and-a-function`](https://stackoverflow.com/questions/155609/whats-the-difference-between-a-method-and-a-function)

在本书中，我将所有方法/函数都称为函数。

## 基本函数声明

这是一个非常简单函数的例子：

```kt
fun printHello(){ 
  Log.i("Message=","Hello") 
}
```

我们可以这样调用`printHello`函数：

```kt
printHello()
```

结果将在 logcat 窗口中输出如下：

```kt
Message=: Hello

```

函数的第一行是**声明**，大括号内包含的所有代码是函数的**主体**。我们使用`fun`关键字，后面跟着函数的名称，然后是一个开括号和闭括号。名称是任意的，但最好使用描述函数功能的名称。

## 函数参数列表

声明可以采用多种形式，这给了我们很大的灵活性和权力。让我们看一些更多的例子：

```kt
fun printSum(a: Int, b: Int) { 
  Log.i("a + b = ","${a+b}") 
}
```

前面的`printSum`函数可以这样调用：

```kt
printSum(2, 3)
```

调用`printSum`函数的结果是，以下消息将输出到 logcat 窗口：

```kt
a + b =: 5
```

请注意，传递给函数的`2`和`3`值是任意的。我们可以传递任何我们喜欢的值，只要它们是`Int`类型的。

声明的这部分`(a: Int, b: Int)`称为**参数列表**，或者只是**参数**。这是函数期望并需要的类型列表，以便成功执行。参数列表可以采用多种形式，任何 Kotlin 类型都可以成为参数列表的一部分，包括根本没有参数（正如我们在第一个例子中看到的）。

当我们调用带有参数列表的函数时，我们必须在调用时提供匹配的参数。以下是我们可以调用前面的`printSum`函数示例的几种可能方式：

```kt
val number1 = 35
val number2 = 15
printSum(9, 1)// Prints a + b: = 10
printSum(10000, 1)// Prints a + b: = 10001
printSum(number1, number2)// Prints a + b: = 50
printSum(65, number1)// Prints a + b: = 100
```

如前面的例子所示，任何组合的值，其总和为两个`Int`值，都可以作为参数。我们甚至可以使用表达式作为参数，只要它们等于一个`Int`值。例如，这个调用也是可以的：

```kt
printSum(100 - 50, number1 + number2)// Prints a + b = 100
```

在上一个示例中，从 100 中减去 50，将结果（50）作为第一个参数传递，然后将`number1`加到`number2`，并将结果作为第二个参数传递。

这里是另外两个带有各种参数的函数，以及我们可能调用它们的示例：

```kt
// These functions would be declared(typed) 
// outside of other functions
// As we did for topClick and bottomClick
fun printName(first: String, second: String){
  Log.i("Joined Name =","$first $second")
}

fun printAreaCircle(radius: Float){
  Log.i("Area =","${3.14 * (radius *radius)}")
}
//…
// This code calls the functions
// Perhaps from onCreate or some other function
val firstName = "Gabe"
val secondName = "Newell"

// Call function using literal String
printName("Sid","Meier")

// Call using String variables
printName(firstName, secondName)

// If a circle has a radius of 3 
// What is the area
printAreaCircle(3f)
```

在讨论代码之前，让我们看一下我们从中得到的输出：

```kt
Joined Name =: Sid Meier
Joined Name =: Gabe Newell
Area =: 28.26

```

在上面的代码中，我们声明了两个函数。第一个叫做`printName`，它有两个`String`参数。声明与突出显示的参数名称再次显示如下。名称是任意的，但使用有意义的名称将使代码更容易理解：

```kt
fun printName(first: String, second: String){
  Log.i("Joined Name =","$first $second")
}
```

尝试使用除两个`String`值以外的任何内容调用该函数将导致错误。当我们调用这个函数时，`first`和`second`参数被初始化为变量，然后我们使用字符串模板将连接的名称打印到 logcat 窗口中。下面再次显示了实现这一点的代码行，其中突出显示了变量：

```kt
Log.i("Joined Name =","$first $second")
```

请注意代码中`$first`和`$second`之间的空格。请注意，这个空格也存在于我们之前看到的输出中。

第二个函数是`printAreaCircle`。它有一个名为`radius`的`Float`参数。这里是它的声明，以便参考：

```kt
fun printAreaCircle(radius: Float){
  Log.i("Area =","${3.14 * (radius * radius)}")
}
```

该函数使用初始化为函数调用时的`radius`变量，使用公式`3.14 * (radius * radius)`来计算圆的面积。

然后，代码继续调用第一个函数两次，第二个函数一次。以下是代码片段中再次显示的内容（为了便于理解，已删除了有用的注释）：

```kt
val firstName = "Gabe"
val secondName = "Newell"

printName("Sid","Meier")
printName(firstName, secondName)

printAreaCircle(3f)
```

请注意，我们可以使用文字值或变量调用函数，只要它们是与声明的参数匹配的正确类型。

要清楚地说明，函数声明位于任何其他函数之外，但位于类的开放和关闭大括号内。函数调用位于`onCreate`函数内。随着我们的应用程序变得更加复杂，我们将从代码的各个部分调用函数（甚至是其他代码文件）。`onCreate`函数只是一个方便的地方，用于讨论这些主题。

### 提示

如果您想更仔细地检查代码结构，包含此代码的文件位于`Chapter09/Functions Demo`文件夹中。创建一个新的 Empty Activity 项目，您可以复制并粘贴代码进行操作。

另一个观点，可能显而易见，但很值得一提的是，当我们为真实应用编写函数时，它们可以包含尽可能多的代码；它们不会像这些示例一样只是一行代码。我们在之前章节学到的任何代码都可以放入我们的函数中。

现在，让我们继续讨论另一个与函数相关的主题，它给我们更多的选择。

## 返回类型和返回关键字

我们经常需要从函数中获得一个结果。仅仅让函数知道结果是不够的。函数可以声明具有**返回类型**。看看下一个函数声明：

```kt
fun getSum(a: Int, b: Int): Int { 
  return a + b 
}
```

在上面的代码中，看一下参数列表的括号后面的突出部分。`：Int`代码表示函数可以并且必须向调用它的代码返回`Int`类型的值。函数体内的代码行使用`return`关键字来实现这一点。`return a + b`代码返回`a`和`b`的和。

我们可以像调用没有返回类型的函数一样调用`getSum`函数：

```kt
getSum(10, 10)
```

上面的代码行可以工作，但有点无意义，因为我们没有对返回的值做任何处理。下面的代码显示了更有可能的对`getSum`函数的调用：

```kt
val answer = getSum(10, 10)
```

在上述函数中，从函数返回的值用于初始化`answer`变量。由于返回类型是`Int`，Kotlin 推断`answer`也是`Int`类型。

我们还可以以其他方式使用`getSum`——下面显示了一个示例：

```kt
// Print out the returned value
Log.i("Returned value =","${getSum(10, 10)}")
```

前面的代码以另一种方式使用了`getSum`函数，通过使用字符串模板打印返回的值到 logcat 窗口。

任何类型都可以从函数中返回。以下是一些例子；首先是声明，然后是一些我们可能调用它们的方式：

```kt
// Return the area of the circle to the calling code
fun getAreaCircle(radius: Float): Float{
  return 3.14f * (radius * radius)
}

// Return the joined-up String to the calling code
fun getName(first: String, second: String): String{
  return "$first $second"
}

// Now we can call them from elsewhere in the code
Log.i("Returned area =","${getAreaCircle(3f)}")
Log.i("Returned name =","${getName("Alan","Turing")}")
```

以下是这两个函数调用将产生的输出：

```kt
Returned area =: 28.26
Returned name =: Alan Turing

```

我们可以看到圆的面积被检索并打印出来，名字被连接在一起被检索并打印出来。

### 注意

作为一个快速的健全检查，值得指出的是，我们实际上不需要编写函数来将数字相加或连接字符串。这只是一种展示函数各个方面的有用方式。

值得注意的是，即使函数没有返回类型，`return`关键字也有其用途。

例如，我们可以使用`return`关键字提前从函数返回。我们之前的所有函数示例（没有返回类型）在主体中的最后一行代码执行时自动返回到调用代码。下面是一个使用`return`关键字的例子：

```kt
fun printUpTo3(aNumber: Int){ // No return type!
  if(aNumber > 3){
    Log.i("aNumber is","TOO BIG! - Didn't you read my name")
    return // Going back to the calling code
  }

  Log.i("aNumber is","$aNumber")
}

// And now we call it with a few different values
printUpTo3(1)
printUpTo3(2)
printUpTo3(3)
printUpTo3(4)
```

看看我们运行前面的代码时的输出，然后我们将讨论它是如何工作的：

```kt
aNumber is: 1
aNumber is: 2
aNumber is: 3
aNumber is: TOO BIG! - Didn't you read my name

```

在函数体中，`if`表达式检查`aNumber`是否大于三，如果是，就打印一个不满的评论，并使用`return`关键字返回到调用代码，并避免将值打印到 logcat。从程序输出中，我们可以看到当`aNUmber`为一、二或三时，它被`printUpTo3`函数忠实地打印出来，但是一旦我们传入了四的值，我们得到了另一种结果。

## 函数体和单表达式函数

函数体可以是复杂的或简单的，我们需要它是什么样的。到目前为止，我展示的所有例子都是故意过于简单，这样我们就可以专注于函数本身而不是其中的代码。随着本书在更多真实世界的例子中的进展，我们将看到函数体中的代码变得更长更复杂。然而，函数体应该坚持执行一个特定的任务。如果你有一个函数在 Android Studio 中占据了整个屏幕，那很可能是它应该被拆分成多个函数的迹象。

当函数的主体非常简单，只包含一个表达式时，Kotlin 允许我们使用单表达式语法来缩短代码。例如，`getSum`函数可以改为以下代码：

```kt
fun getSum(a: Int, b: Int) = a + b
```

在前面的例子中，我们去掉了通常包裹在函数体中的花括号，并推断了返回类型，因为将`a`加到`b`只能得到一个`Int`变量，因为`a`和`b`本身就是`Int`变量。

# 使函数灵活

由于函数是我们代码的构建块，它们需要具有多样性，以满足我们可能需要做的任何事情。我们已经看到了如何创建非常多样的参数列表和返回类型，以及在代码中决定何时返回到调用代码。随着我们的进展，你会发现我们需要更多的选项。接下来是一些我们现在将介绍的更多 Kotlin 函数选项的快速概览，然后在本书的各个部分真正开始使用。

## 默认和命名参数

**默认参数**是指我们程序员为参数提供一个值（默认值），如果调用函数的代码没有提供该值，则将使用该值。**命名参数**是指调用函数的代码指定名称以及值。请注意，提供值是可选的。只因为为参数提供了默认值并不意味着调用代码不能通过提供值来覆盖它。看下面的例子：

```kt
fun orderProduct(giftWrap: Boolean = false,
                product: String,
                postalService: String = "Standard") {

   var details: String = ""

   if (giftWrap) {
       details += "Gift wrapped "
   }

   details += "$product "
   details += "by $postalService postage"

   Log.i("Product details",details)
}

// Here are some ways we can call this function
orderProduct(product = "Beer")
orderProduct(true, product = "Porsche")
orderProduct(true, product = "Barbie (Jet-Set Edition)", postalService = "Next Day")

orderProduct(product = "Flat-pack bookcase", 
   postalService = "Carrier Pigeon")
```

在前面的代码中，我们首先声明了一个名为`orderProduct`的函数。请注意，在参数列表中，我们声明了两个默认值，如下所示：

```kt
fun orderProduct(giftWrap: Boolean = false,
       product: String,
       postalService: String = "Standard") {
```

当我们调用函数时，可以在不指定`giftwrap`和/或`postalService`的情况下这样做。以下代码中的第一个函数调用清楚地表明了这一点：

```kt
orderProduct(product = "Beer")
```

请注意，当我们这样做时，需要指定参数的名称，它必须与参数列表中的名称以及类型匹配。在第二个函数调用中，我们为`giftwrap`和`product`指定了一个值：

```kt
orderProduct(true, product = "Porsche")
```

在第三个中，我们为所有三个参数指定了一个值，如下面的代码中再次看到的：

```kt
orderProduct(true, product = "Barbie (Jet-Set Edition)",
   postalService = "Next Day")
```

最后，在第四个中，我们指定了最后两个参数：

```kt
orderProduct(product = "Flat-pack bookcase", 
   postalService = "Carrier Pigeon")
```

函数本身的代码从声明一个名为`details`的`var`变量开始，它是一个`String`值。如果`giftwrap`的值为 true，则将`Gift Wrapped`附加到`Product details`。接下来，将`product`的值附加到`details`，最后将`postalService`的值与字面`String`值附加在两侧。

如果我们运行代码，这是在 logcat 窗口中的输出：

```kt
Product details: Beer by Standard postage
Product details: Gift wrapped Porsche by Standard postage
Product details: Gift wrapped Barbie (Jet-Set Edition) 
 by Next Day postage
Product details: Flat-pack bookcase by Carrier Pigeon postage

```

我们可以以多种方式调用函数，这非常有用。在其他编程语言中，当您希望以不同方式调用相同命名的函数时，必须提供多个版本的函数。虽然学习命名参数和默认参数可能会增加一些复杂性，但它肯定比不得不编写`orderProduct`函数的四个版本要好。这，连同类型推断，只是您经常会听到程序员赞扬 Kotlin 简洁性的两个原因之一。

使用命名参数和默认参数，我们可以选择提供尽可能多或尽可能少的数据，只要函数允许。简而言之，如果我们提供了所有没有默认值的参数的值，它将起作用。

### 提示

如果您想要使用这段代码进行操作，那么本章中的所有示例都在`Chapter09`文件夹中。创建一个空活动项目，然后将函数复制粘贴到`MainActivity`类中，将函数调用复制粘贴到`onCreate`函数中。

在我们进行这些操作时，会有一些注意事项，随着我们在整本书中进行更多的实际示例，我们将会看到它们。

## 更多关于函数的内容

函数还有更多内容，例如顶级函数、局部函数和可变参数函数，以及函数访问级别，但最好是在类和面向对象编程的主题旁边或之后讨论这些内容。

# 总结

在本章中，我们在学习函数方面取得了良好的进展。虽然函数自第一章以来一直潜伏在我们的代码中，但我们终于正式学习和理解了它们。我们了解了函数的不同部分：名称、参数和返回类型。我们已经看到函数实际上是在开放和关闭的大括号内部执行的，称为函数体。

我们还看到，我们可以使用`return`关键字随时从函数中返回，并且我们还可以将返回类型与`return`关键字结合使用，以使函数中的数据可用于首次调用函数的代码。

我们学会了如何使用默认和命名参数来提供同一函数的不同版本，而无需编写多个函数。这使我们的代码更加简洁和可管理。

我们还发现，在本章中，函数还有更多内容，但最好是在整本书中的各种项目中学习这些主题。

接下来，我们将转向最受关注的章节。我一直在参考和推迟到第十章，“面向对象编程”。最后，它来了，我们将看到类和对象与 Kotlin 结合的真正力量。在接下来的几章中，我们很快就会看到类和对象是释放 Android API 力量的关键。我们很快就能让我们的用户界面栩栩如生，并且将构建一些真正可用的应用程序，我们可以发布到 Play 商店。
