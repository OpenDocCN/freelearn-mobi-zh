# 第五章：函数作为一等公民

在上一章中，我们看到了 Kotlin 特性与面向对象编程的关系。本章将介绍以前在标准 Android 开发中不存在的高级函数式编程特性。其中一些在 Java 8 中引入（通过 Retrolambda 插件在 Android 中引入），但 Kotlin 引入了更多的函数式编程特性。

这一章是关于高级函数和函数作为一等公民的。大多数概念对于过去使用过函数式语言的读者来说都是熟悉的。

在本章中，我们将涵盖以下主题：

+   函数类型

+   匿名函数

+   Lambda 表达式

+   lambda 表达式中单个参数的隐式名称

+   高阶函数

+   最后一个参数中的 lambda 约定

+   Java **Single Abstract Method**（**SAM**）lambda 接口

+   在参数上使用 Java 方法和 Java Single Abstract Method

+   函数类型中的命名参数

+   类型别名

+   内联函数

+   函数引用

# 函数类型

Kotlin 支持函数式编程，函数在 Kotlin 中是一等公民。在给定的编程语言中，一等公民是指支持其他实体通常可用的所有操作的实体。这些操作通常包括作为参数传递，从函数返回，并分配给变量。因此，“函数在 Kotlin 中是一等公民”这句话应该被理解为：“在 Kotlin 中可以将函数作为参数传递，从函数返回，并将其分配给变量”。虽然 Kotlin 是一种静态类型语言，但需要定义函数类型以允许这些操作。在 Kotlin 中，用于定义函数类型的表示法如下：

```kt
    (types of parameters)->return type 
```

以下是一些例子：

+   `(Int)->Int`：接受`Int`作为参数并返回`Int`的函数

+   `()->Int`：不接受任何参数并返回`Int`的函数

+   `(Int)->Unit`：接受`Int`并不返回任何东西（只有`Unit`，不需要返回）

以下是一些可以保存函数的属性的示例：

```kt
    lateinit var a: (Int) -> Int   
    lateinit var b: ()->Int 
    lateinit var c: (String)->Unit 
```

*函数类型*通常被定义为变量或参数的类型，可以将函数分配给它们，或者作为高阶函数的参数或结果类型。在 Kotlin 中，函数类型可以被视为接口。

我们将在本章后面看到，Kotlin 函数可以在参数中接受其他函数，甚至返回它们：

```kt
    fun addCache(function: (Int) -> Int): (Int) -> Int { 
        // code 
    } 

    val fibonacciNumber: (Int)->Int = // function implementation 
    val fibonacciNumberWithCache = addCache(fibonacciNumber) 
```

如果一个函数可以接受或返回一个函数，那么函数类型也需要能够定义接受函数作为参数或返回函数的函数。这可以通过简单地将函数类型表示法放置为参数或返回类型来实现。以下是一些例子：

+   `(String)->(Int)->Int`：接受`String`并返回一个接受`Int`类型并返回`Int`的函数。

+   `(()->Int)->String`：接受另一个函数作为参数，并返回`String`类型的函数。参数中的函数不接受任何参数，并返回`Int`。

每个带有函数类型的属性都可以像函数一样被调用：

```kt
    val i = a(10) 
    val j = b() 
    c("Some String") 
```

函数不仅可以存储在变量中，还可以用作泛型。例如，我们可以将函数保存在列表中：

```kt
    var todoList: List<() -> Unit> = // ... 
    for (task in todoList) task() 
```

前面的列表可以存储带有`() -> Unit`签名的函数。

# 底层的函数类型是什么？

在底层，函数类型只是泛型接口的一种语法糖。让我们看一些例子：

+   `()->Unit`签名是`Function0<Unit>`的接口。表达式是`Function0`，因为它没有参数，而`Unit`是返回类型。

+   `(Int)->Unit`签名是`Function1<Int, Unit>`的接口。表达式是`Function1`，因为它有一个参数。

+   `()->(Int, Int)->String`签名是`Function0<Function2<Int, Int, String>>`的接口。

所有这些接口只有一个方法，`invoke`，它是一个操作符。它允许对象像函数一样使用：

```kt
    val a: (Int) -> Unit = //... 
    a(10)        // 1 
    a.invoke(10) // 1 
```

1.  这两个语句具有相同的含义

函数接口不在标准库中。它们是合成的编译器生成的类型（它们在编译过程中生成）。因此，函数类型参数的数量没有人为限制，标准库的大小也不会增加。

# 匿名函数

定义函数为对象的一种方式是使用**匿名函数**。它们的工作方式与普通函数相同，但在`fun`关键字和参数声明之间没有名称，因此默认情况下它们被视为对象。以下是一些示例：

```kt
    val a: (Int) -> Int = fun(i: Int) = i * 2 // 1 
    val b: ()->Int = fun(): Int { return 4 } 
    val c: (String)->Unit = fun(s: String){ println(s) } 
```

1.  这是一个匿名的单表达式函数。请注意，与普通的单表达式函数一样，当它从表达式返回类型中推断出时，不需要指定返回类型。

考虑以下用法：

```kt
    // Usage 
    println(a(10))      // Prints: 20 
    println(b())        // Prints: 4 
    c("Kotlin rules")   // Prints: Kotlin rules 
```

在前面的示例中，函数类型是显式定义的，但是由于 Kotlin 有很好的类型推断系统，函数类型也可以从匿名默认函数定义的类型中推断出来：

```kt
    var a = fun(i: Int) = i * 2 
    var b = fun(): Int { return 4 } 
    var c = fun(s: String){ println(s) } 
```

它也可以反过来。当我们定义属性的类型时，我们不需要在匿名函数中显式设置参数类型，因为它们是从表达式返回类型中推断出来的：

```kt
    var a: (Int)->Int = fun(i) = i * 2 
    var c: (String)->Unit = fun(s){ println(s) }
```

如果我们查看函数类型的方法，那么我们将看到里面只有`invoke`方法。`invoke`方法是一个操作符函数，它可以像函数调用一样使用。这就是为什么可以通过在括号内使用`invoke`调用来实现相同的结果：

```kt
    println(a.invoke(4))        // Prints: 8 
    println(b.invoke())         // Prints: 4 
    c.invoke("Hello, World!")   // Prints: Hello, World! 
```

这种知识在某些情况下是有帮助的，比如当我们将函数保存在可空变量中时。例如，我们可以使用`invoke`方法通过安全调用：

```kt
    var a: ((Int) -> Int)? = null // 1 
    if (false) a = fun(i: Int) = i * 2 
    print(a?.invoke(4)) // Prints: null 
```

1.  变量`a`是可空的，我们使用安全调用来调用。

让我们看一个 Android 示例。我们经常希望定义一个单一的错误处理程序，其中包括多个日志记录方法，并将其作为参数传递给不同的对象。以下是我们可以使用匿名函数来实现它的方式：

```kt
    val TAG = "MainActivity" 
    val errorHandler = fun (error: Throwable) { 
        if(BuildConfig.DEBUG) { 
            Log.e(TAG, error.message, error) 
        } 
        toast(error.message) 
        // Other methods, like: Crashlytics.logException(error) 
    } 

    // Usage in project 
    val adController = AdController(errorHandler) 
    val presenter = MainPresenter(errorHandler) 

    // Usage 
    val error = Error("ExampleError") 
    errorHandler(error) // Logs: MainActivity: ExampleError 
```

匿名函数简单而有用。它们是定义可以用作对象并传递的函数的简单方式。但是有一种更简单的方法可以实现类似的行为，它被称为 lambda 表达式。

# Lambda 表达式

在 Kotlin 中定义匿名函数的最简单方式是使用一个称为 lambda 表达式的特性。它们类似于 Java 8 的 lambda 表达式，但最大的区别是 Kotlin 的 lambda 实际上是闭包，因此允许我们从创建上下文更改变量。这在 Java 8 的 lambda 中是不允许的。我们将在本节后面讨论这种差异。让我们从一些简单的示例开始。Kotlin 中的 lambda 表达式具有以下表示法：

```kt
    { arguments -> function body } 
```

返回值是最后一个表达式的结果。以下是一些简单的 lambda 表达式示例：

+   `{ 1 }`：一个 lambda 表达式，不接受参数并返回 1。它的类型是`()->Int`。

+   `{ s: String -> println(s) }`：一个 lambda 表达式，接受一个`String`类型的参数，并打印它。它返回`Unit`。它的类型是`(String)->Unit`。

+   `{ a: Int, b: Int -> a + b }`：一个 lambda 表达式，接受两个`Int`参数并返回它们的和。它的类型是`(Int, Int)->Int`。

我们在上一章中定义的函数可以使用 lambda 表达式来定义：

```kt
    var a: (Int) -> Int = { i: Int -> i * 2 } 
    var b: ()->Int = { 4 } 
    var c: (String)->Unit = { s: String -> println(s) } 
```

虽然 lambda 表达式中的返回值是从最后一个语句中取得的，但是除非有带有标签的`return`语句，否则是不允许使用`return`的：

```kt
    var a: (Int) -> Int = { i: Int -> return i * 2 } 

    // Error: Return is not allowed there 
    var l: (Int) -> Int = l@ { i: Int -> return@l i * 2 } 
```

Lambda 表达式可以是多行的：

```kt
    val printAndReturn = { i: Int, j: Int -> 
        println("I calculate $i + $j") 
        i + j // 1 
    } 
```

1.  这是最后一个语句，因此这个表达式的结果将是一个返回值。

多个语句也可以在一行中定义，当它们用分号分隔时：

```kt
val printAndReturn = {i: Int, j: Int -> println("I calculate $i + $j"); 

                      i + j } 
```

lambda 表达式不仅需要操作由参数提供的值。Kotlin 中的 lambda 表达式可以使用创建它们的上下文中的所有属性和函数：

```kt
    val text = "Text" 
    var a: () -> Unit = { println(text) } 
    a() // Prints: Text 
    a() // Prints: Text 
```

这是 Kotlin 和 Java 8 lambda 使用之间最大的区别。Java 匿名对象和 Java 8 lambda 表达式都允许我们使用上下文中的字段，但 Java 不允许我们为这些变量分配不同的值（lambda 中使用的 Java 变量必须是 final）：

![](img/Image00036.jpg)

Kotlin 通过允许 lambda 表达式和匿名函数修改这些变量迈出了一步。包围局部变量并允许我们在函数体内更改它们的 lambda 表达式称为**闭包**。Kotlin 完全支持闭包定义。为了避免混淆 lambda 和闭包，在本书中，我们将始终称它们为 lambda。让我们看一个例子：

```kt
    var i = 1 
    val a: () -> Int = { ++i } 
    println (i)     // Prints: 1 
    println (a())   // Prints: 2 
    println (i)     // Prints: 2 
    println (a())   // Prints: 3 
    println (i)     // Prints: 3
```

lambda 表达式可以使用和修改局部上下文中的变量。这是一个计数器的例子，其中值保存在一个局部变量中：

```kt
    fun setUpCounter() { 
        var value: Int = 0 
        val showValue = { counterView.text = "$value" } 
        counterIncView.setOnClickListener { value++; showValue() } 

        // 1 
        counterDecView.setOnClickListener { value--; showValue() } 

        // 1 
    } 
```

1.  以下是如何在 Kotlin 中使用 lambda 表达式设置 View `onClickListener`。这将在*Java SAM support in Kotlin*部分中描述。

由于这个特性，使用 lambda 表达式变得更简单。请注意，在前面的例子中，没有指定`showValue`的类型。这是因为在 Kotlin lambda 中，当编译器可以从上下文中推断出类型时，参数的类型是可选的：

```kt
    val a: (Int) -> Int = { i -> i * 2 }  // 1 
    val c: (String)->Unit = { s -> println(s) } // 2 
```

1.  `i`的推断类型是`Int`，因为函数类型定义了一个`Int`参数。

1.  `s`的推断类型是`String`，因为函数类型定义了一个`String`参数。

正如我们在下面的例子中看到的，我们不需要指定参数的类型，因为它是从属性的类型中推断出来的。类型推断也可以以另一种方式工作--我们可以定义 lambda 表达式的参数类型以推断属性类型：

```kt
    val b = { 4 }                        // 1 
    val c = { s: String -> println(s) }  // 2 
    val a = { i: Int -> i * 2 }          // 3 
```

1.  推断的类型是`()->Int`，因为`4`是`Int`，并且没有参数类型。

1.  推断的类型是`(String)->Unit`，因为参数被定义为`String`，而`println`方法的返回类型是`Unit`。

1.  推断的类型是`(Int)->Int`，因为`i`被定义为`Int`，并且`Int`的 times 操作的返回类型也是`Int`。

这种推断简化了 lambda 表达式的定义。通常，在我们将 lambda 表达式定义为函数参数时，我们不需要每次指定参数类型。但还有另一个好处--虽然参数类型可以被推断，但可以使用更简单的表示法来表示单个参数的 lambda 表达式。让我们在下一节中讨论这个问题。

# 单个参数的隐式名称

当满足两个条件时，我们可以省略 lambda 参数定义并使用`it`关键字访问参数：

+   只有一个参数

+   参数类型可以从上下文中推断出来

举个例子，让我们再次定义属性`a`和`c`，但这次使用单个参数的隐式名称：

```kt
    val a: (Int) -> Int = { it * 2 }         // 1
    val c: (String)->Unit = { println(it) }  // 2 
```

1.  与`{ i -> i * 2 }`相同。

1.  与`{ s -> println(s) }`相同。

这种表示法在 Kotlin 中非常流行，主要是因为它更短，可以避免参数类型的指定。它还提高了 LINQ 风格中定义的处理的可读性。这种风格需要尚未介绍的组件，但只是为了展示这个想法，让我们看一个例子：

```kt
    strings.filter { it.length = 5 }.map { it.toUpperCase() } 
```

假设 strings 是`List<String>`，这个表达式会过滤长度等于`5`的字符串，并将它们转换为大写。

请注意，在 lambda 表达式的主体中，我们可以使用`String`类的方法。这是因为函数类型（例如`filter`的`(String)->Boolean`）是从方法定义中推断出来的，它从可迭代类型（`List<String>`）中推断出`String`。此外，返回的列表的类型（`List<String>`）取决于 lambda 返回的内容（`String`）。

LINQ 风格在函数式语言中很受欢迎，因为它使集合或字符串处理的语法变得非常简单和简洁。它将在第七章 *扩展函数和属性*中更详细地讨论。

# 高阶函数

高阶函数是一个至少接受一个函数作为参数或将函数作为其结果返回的函数。在 Kotlin 中，它得到了充分的支持，因为函数是一等公民。让我们在一个例子中看看它。假设我们需要两个函数：一个函数将从列表中添加所有`BigDecimal`数字，另一个函数将得到所有这些数字的乘积（列表中所有元素之间的乘法结果）：

```kt
    fun sum(numbers: List<BigDecimal>): BigDecimal { 
        var sum = BigDecimal.ZERO 
        for (num in numbers) { 
            sum += num 
        } 
        return sum 
    } 

    fun prod(numbers: List<BigDecimal>): BigDecimal { 
        var prod = BigDecimal.ONE 
        for (num in numbers) { 
            prod *= num 
        } 
        return prod 
    } 

    // Usage 
    val numbers = listOf( 
        BigDecimal.TEN,  
        BigDecimal.ONE,  
        BigDecimal.valueOf(2) 
    ) 
    print(numbers)          //[10, 1, 2] 
    println(prod(numbers))  // 20 
    println(sum(numbers))   // 13 
```

这些是可读的函数，但这些函数几乎相同。唯一的区别是名称、累加器（`BigDecimal.ZERO`或`BigDecimal.ONE`）和操作。如果我们遵循**DRY**（**不要重复自己**）规则，那么我们不应该在项目中留下两个相似代码的部分。虽然很容易定义一个函数，它将具有类似的行为，只是使用的对象不同，但很难定义一个函数，它将根据执行的操作不同（这里，函数根据用于累加的操作不同）。解决方案是使用函数类型，因为我们可以将操作作为参数传递。在这个例子中，可以这样提取公共方法：

```kt
    fun sum(numbers: List<BigDecimal>) = 
        fold(numbers, BigDecimal.ZERO) { acc, num -> acc + num } 

    fun prod(numbers: List<BigDecimal>) = 
       fold(numbers, BigDecimal.ONE) { acc, num -> acc * num } 

    private fun fold( 
        numbers: List<BigDecimal>, 
        start: BigDecimal, 
        accumulator: (BigDecimal, BigDecimal) -> BigDecimal 
    ): BigDecimal { 
        var acc = start 
        for (num in numbers) { 
            acc = accumulator(acc, num) 
        } 
        return acc 
    } 

    // Usage 

    fun BD(i: Long) = BigDecimal.valueOf(i) 
    val numbers = listOf(BD(1), BD(2), BD(3), BD(4)) 
    println(sum(numbers))   // Prints: 10 
    println(prod(numbers))  // Prints: 24 
```

`fold`函数遍历数字并使用每个元素更新`acc`。请注意，函数参数像任何其他类型一样定义，并且可以像任何其他函数一样使用。例如，我们可以有可变参数函数类型参数：

```kt
    fun longOperation(vararg observers: ()->Unit) {
        //... 
        for(o in observers) o()
    } 
```

在`longOperation`中，`for`用于迭代所有观察者并依次调用它们。这个函数允许提供多个函数作为参数。这里是一个例子：

```kt
    longOperation({ notifyMainView() }, { notifyFooterView() })
```

Kotlin 中的函数也可以返回函数。例如，我们可以定义一个函数，它将创建具有相同错误日志记录但不同标记的自定义错误处理程序：

```kt
    fun makeErrorHandler(tag: String) = fun (error: Throwable) { 
        if(BuildConfig.DEBUG) Log.e(tag, error.message, error) 
        toast(error.message) 
        // Other methods, like: Crashlytics.logException(error) 
    } 

    // Usage in project 
    val adController = AdController(makeErrorHandler("Ad in MainActivity")) 
    val presenter = MainPresenter(makeErrorHandler("MainPresenter")) 

    // Usage 
    val exampleHandler = makeErrorHandler("Example Handler") 
    exampleHandler(Error("Some Error")) // Logs: Example Handler: Some Error 
```

函数参数中使用函数的三种最常见情况是：

+   为函数提供操作

+   观察者（监听器）模式

+   线程操作后的回调

让我们详细看看它们。

# 为函数提供操作

正如我们在前一节中看到的，有时我们想从函数中提取公共功能，但它们在使用的操作上有所不同。在这种情况下，我们仍然可以提取这个功能，但是我们需要提供一个区分它们的操作参数。这样，任何常见的模式都可以被提取和重用。例如，我们通常只需要列表中与某些谓词匹配的元素，比如当我们只想显示活动元素时。传统上，这样实现：

```kt
    var visibleTasks = emptyList<Task>() 
    for (task in tasks) { 
        if (task.active) 
        visibleTasks += task 
    } 
```

虽然这是一个常见的操作，但我们可以提取仅根据谓词过滤一些元素的功能，以分离函数并更容易使用它：

```kt
    fun <T> filter(list: List<T>, predicate: (T)->Boolean) { 
        var visibleTasks = emptyList<T>() 
        for (elem in list) { 
            if (predicate(elem)) 
                visibleTasks += elem 
        } 
    } 

    var visibleTasks = filter(tasks, { it.active }) 
```

使用高阶函数的这种方式非常重要，并且在整本书中经常会被描述，但这并不是高阶函数经常被使用的唯一方式。

# 观察者（监听器）模式

当事件发生时，我们使用观察者（监听器）模式来执行操作。在 Android 开发中，观察者经常设置为视图元素。常见的例子是点击监听器、触摸监听器或文本监视器。在 Kotlin 中，我们可以在没有样板文件的情况下设置监听器。例如，设置按钮点击监听器如下所示：

```kt
    button.setOnClickListener({ someOperation() }) 
```

请注意，`setOnClickListener`是 Android 库中的一个 Java 方法。稍后，我们将详细看到为什么我们可以使用它与 lambda 表达式。监听器的创建非常简单。这是一个例子：

```kt
    var listeners: List<()->Unit> = emptyList() // 1 
    fun addListener(listener: ()->Unit) { 
        listeners += listener // 2 
    } 

    fun invokeListeners() { 
        for( listener in listeners) listener() // 3 
    } 
```

1.  在这里，我们创建一个空列表来保存所有监听器。

1.  我们可以简单地将监听器添加到监听器列表中。

1.  我们可以遍历监听器并依次调用它们。

很难想象有一个更简单的实现方式。还有另一个常见的用例，即在线程操作后常用的参数。

# 线程操作后的回调

如果我们需要执行一个长时间的操作，并且不想让用户等待，那么我们必须在另一个线程中启动它。为了能够在单独的线程中调用长时间操作后的回调，我们需要将其作为参数传递。这是一个示例函数：

```kt
fun longOperationAsync(longOperation: ()->Unit, callback: ()->Unit) { 
        Thread({ // 1 
            longOperation() // 2 
            callback() // 3 
        }).start() // 4 
    } 

    // Usage

    longOperationAsync

(

            longOperation = 

{ 

Thread.sleep(1000L

) }

,

            callback = 

{ 

print

("After second"

) } 

            // 5, Prints: 

After second

    )

    println

("Now"

) // 6, Prints: Now
```

1.  在这里，我们创建`Thread`。我们还传递了一个我们想要在构造函数参数上执行的 lambda 表达式。

1.  在这里，我们正在执行一个长时间的操作。

1.  在这里，我们启动了提供的回调操作。

1.  `start`是启动定义线程的方法。

1.  在一秒延迟后打印。

1.  立即打印。

实际上，有一些流行的替代方案可以使用回调，例如 RxJava。不过，经典的回调仍然常用，在 Kotlin 中可以无样板实现。

这些是使用高阶函数的最常见用例。所有这些都允许我们提取公共行为并减少样板文件。Kotlin 允许在高阶函数方面进行一些改进。

# 命名参数和 lambda 表达式的组合

在 Android 中使用默认命名参数和 lambda 表达式可以非常有用。让我们看一些其他实际的 Android 示例。假设我们有一个函数，它下载元素并将它们显示给用户。我们将添加一些参数：

+   `onStart`：这将在网络操作之前调用

+   `onFinish`：这将在网络操作之后调用

```kt
fun getAndFillList(onStart: () -> Unit = {}, 

    onFinish: () -> Unit = {}){ 
        // code 
    } 
```

然后，我们可以在`onStart`和`onFinish`中显示和隐藏加载旋转器：

```kt
    getAndFillList( 
        onStart = { view.loadingProgress = true } , 
        onFinish = { view.loadingProgress = false } 
    ) 
```

如果我们从`swipeRefresh`开始，那么当它完成时我们只需要隐藏它：

```kt
getAndFillList(onFinish = { view.swipeRefresh.isRefreshing = 

   false }) 
```

如果我们想进行一个安静的刷新，那么我们只需调用这个：

```kt
    getAndFillList() 
```

命名参数语法和 lambda 表达式是多用途函数的完美匹配。这连接了选择要实现的参数和应该实现的操作的能力。如果一个函数包含多个函数类型参数，那么在大多数情况下，应该使用命名参数语法。这是因为当使用多个 lambda 表达式作为参数时，它们很少是自解释的。

# 参数约定中的最后一个 lambda

在 Kotlin 中，高阶函数非常重要，因此使它们的使用尽可能舒适也很重要。这就是为什么 Kotlin 引入了一种特殊的约定，使高阶函数更简单和清晰。它的工作方式是：如果最后一个参数是一个函数，那么我们可以在括号外定义 lambda 表达式。让我们看看如果我们将其与`longOperationAsync`函数一起使用，该函数定义如下：

```kt
    fun longOperationAsync(a: Int, callback: ()->Unit) { 
        // ... 
    } 
```

函数类型在参数的最后位置。这就是为什么我们可以这样执行它：

```kt
    longOperationAsync(10) {
        hideProgress() 
    } 
```

由于最后一个 lambda 在参数约定中，我们可以将 lambda 定位在括号后面。看起来好像它在参数之外。

例如，让我们看看在 Kotlin 中如何在另一个线程中调用代码。在 Kotlin 中启动新线程的标准方式是使用 Kotlin 标准库中的`thread`函数。它的定义如下：

```kt
    public fun thread( 
        start: Boolean = true, 
        isDaemon: Boolean = false, 
        contextClassLoader: ClassLoader? = null, 
        name: String? = null, 
        priority: Int = -1, 
        block: () -> Unit): Thread { 
            // implementation 
        } 
```

正如我们所看到的，`block`参数，它接受应该异步调用的操作，位于最后位置。所有其他参数都有默认参数定义。这就是为什么我们可以这样使用`thread`函数：

```kt
    thread { /* code */ } 
```

`thread`定义有很多其他参数，我们可以通过使用命名参数语法或依次提供它们来设置它们：

```kt
    thread (name = "SomeThread") { /*...*/ } 
    thread (false, false) { /*...*/ } 
```

参数约定中的最后一个 lambda 是语法糖，但它使使用高阶函数变得更容易。这是这种约定真正产生差异的两种最常见情况：

+   命名代码周围

+   使用 LINQ 风格处理数据结构

让我们仔细看看它们。

# 命名代码环绕

有时我们需要标记代码的某些部分以不同的方式执行。`thread`函数就是这种情况。我们需要一些代码以异步方式执行，因此我们用从`thread`函数开始的括号将其包围起来。

```kt
    thread {  
        operation1() 
        operation2() 
    } 
```

从外部看，它看起来好像是由名为`thread`的块包围的代码的一部分。让我们看另一个例子。假设我们想要记录某个代码块的执行时间。作为辅助，我们将定义`addLogs`函数，它将与执行时间一起打印日志。我们将以以下方式定义它：

```kt
    fun addLogs(tag: String, f: () -> Unit) { 
        println("$tag started") 
        val startTime = System.currentTimeMillis() 
        f() 
        val endTime = System.currentTimeMillis() 
        println("$tag finished. It took " + (endTime - startTime)) 
    } 
```

以下是该函数的用法：

```kt
    addLogs("Some operations") { 
        // Operations we are measuring 
    } 
```

以下是其执行示例：

```kt
    addLogs("Sleeper") { 
        Thread.sleep(1000) 
    } 
```

在执行前面的代码时，将呈现以下输出：

```kt
    Sleeper started 
    Sleeper finished. It took 1001 
```

打印的毫秒数可能会有所不同。

这种模式在 Kotlin 项目中非常有用，因为一些模式与代码块相关联。例如，在执行至少需要此版本才能工作的功能之前，通常会检查 API 的版本是否在 Android 5.x Lollipop 之后。为了检查它，我们使用了以下条件：

```kt
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP) {  
        // Operations 
    } 
```

但在 Kotlin 中，我们可以以以下方式提取函数：

```kt
    fun ifSupportsLolipop(f:()->Unit) { 
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP) 

        {  
            f() 
        } 
    } 
    //Usage    

    ifSupportsLollipop { 
        // Operation 
    } 
```

这不仅舒适，而且还减少了代码中的冗余。这通常被称为非常好的实践。还要注意，这种约定使我们能够定义与标准结构类似工作的控制结构。例如，我们可以定义一个简单的控制结构，只要主体中的语句不返回错误，就会一直运行。以下是定义和用法：

```kt
    fun repeatUntilError(code: ()->Unit): Throwable { 
        while (true) { 
            try { 
                code() 
            } catch (t: Throwable) { 
                return t 
            } 
        } 
    } 

    // Usage 
    val tooMuchAttemptsError = repeatUntilError { 
        attemptLogin() 
    } 
```

额外的优势是我们的自定义数据结构可以返回一个值。令人印象深刻的是，它不需要任何额外的语言支持，我们可以定义几乎任何我们想要的控制结构。

# 使用 LINQ 风格处理数据结构

我们已经提到 Kotlin 允许 LINQ 风格的处理。参数中的最后一个 lambda 约定是另一个有助于其可读性的组件。例如，看下面的代码：

```kt
    strings.filter { it.length == 5 }.map { it.toUpperCase() } 
```

它比不使用参数中的最后一个 lambda 约定的表示法更易读：

```kt
    strings.({ s -> s.length == 5 }).map({ s -> s.toUpperCase() }) 
```

再次强调，这种处理将在稍后的第七章中详细讨论，*扩展函数和属性*，但现在我们已经了解了两个改进其可读性的特性（参数中的最后一个 lambda 约定和单个参数的隐式名称）。

参数中的最后一个 lambda 约定是 Kotlin 引入的一个特性，旨在改进 lambda 表达式的使用。还有更多这样的改进，它们如何一起工作对于使高阶函数的使用简单、可读和高效非常重要。

# Kotlin 中的 Java SAM 支持

在 Kotlin 中使用高阶函数非常容易。问题在于，我们经常需要与 Java 进行交互，而 Java 本身不支持它。它通过使用只有一个方法的接口来实现替代。这种接口称为**单一抽象方法**（**SAM**）或功能接口。我们需要以这种方式设置函数的最佳示例是在使用`View`元素上的`setOnClickListener`时。在 Java（直到 8）中，没有比使用匿名内部类更简单的方法：

```kt
    //Java 
    button.setOnClickListener(new OnClickListener() { 
        @Override public void onClick(View v) { 
            // Operation 
        } 
    }); 
```

在前面的例子中，`OnClickListener`方法是 SAM，因为它只包含一个方法`onClick`。虽然 SAM 经常被用作函数定义的替代，但 Kotlin 也为它们生成了一个包含函数类型作为参数的构造函数。它被称为 SAM 构造函数。SAM 构造函数允许我们通过调用其名称并传递*函数文字*来创建 Java SAM 接口的实例。以下是一个例子：

```kt
    button.setOnClickListener(OnClickListener { 
        /* ... */ 
    }) 
```

*函数文字*是定义未命名函数的表达式。在 Kotlin 中，有两种*函数文字*：

1. 匿名函数

2. Lambda 表达式

Kotlin *function literal* 已经被描述过了：

`val a = fun() {} // Anonymous function`

`val b = {} // Lambda expression`

更好的是，对于每个接受 SAM 的 Java 方法，Kotlin 编译器都会生成一个接受函数作为参数的版本。这就是为什么我们可以这样设置 `OnClickListener`：

```kt
    button.setOnClickListener { 
        // Operations 
    } 
```

请记住，Kotlin 编译器只为 Java SAM 生成 SAM 构造函数和函数方法。它不会为具有单个方法的 Kotlin 接口生成 SAM 构造函数。这是因为 Kotlin 社区正在推动在 Kotlin 代码中使用函数类型而不是 SAM。当一个函数是用 Kotlin 编写的并包含 SAM 时，我们无法将其用作具有 SAM 参数的 Java 方法：

```kt
    interface OnClick { 
        fun call() 
    } 

    fun setOnClick(onClick: OnClick) { 
        //... 
    } 

    setOnClick {  } // 1\. Error 
```

1.  这不起作用，因为 `setOnClick` 函数是用 Kotlin 编写的。

在 Kotlin 中，不应该以这种方式使用接口。首选的方式是使用函数类型而不是 SAM：

```kt
    fun setOnClick(onClick: ()->Unit) { 
        //...   
    } 

    setOnClick {  } // Works 
```

Kotlin 编译器为在 Java 中定义的每个 SAM 接口生成一个 SAM 构造函数。这个接口只包括可以替代 SAM 的函数类型。看看下面的接口：

```kt
    // Java, inside View class 
    public interface OnClickListener { 
        void onClick(View v); 
    } 
```

我们可以在 Kotlin 中这样使用它：

```kt
    val onClick = View.OnClickListener { toast("Clicked") } 
```

或者我们可以将其作为函数参数提供：

```kt
    fun addOnClickListener(d: View.OnClickListener) {} 
    addOnClickListener( View.OnClickListener { v -> println(v) }) 
```

以下是 Java SAM lambda 接口和 Android 方法的更多示例：

```kt
    view.setOnLongClickListener { /* ... */; true } 
    view.onFocusChange { view, b -> /* ... */ } 

    val callback = Runnable { /* ... */ } 
    view.postDelayed(callback, 1000) 
    view.removeCallbacks(callback) 
```

以下是来自 RxJava 的一些示例：

```kt
    observable.doOnNext { /* ... */ } 
    observable.doOnEach { /* ... */ } 
```

现在，让我们看看 Kotlin 中如何实现对 SAM 定义的替代方案。

# 命名 Kotlin 函数类型

Kotlin 不支持在 Kotlin 中定义的类型的 SAM 转换，因为首选的方式是使用函数类型。但是 SAM 在经典函数类型上有一些优势：命名参数。当函数类型的定义很长或者作为参数传递多次时，最好将函数类型命名。当仅凭类型无法清楚地知道每个参数的含义时，最好使用命名参数。

在接下来的部分中，我们将看到可以为函数类型的参数和整个定义命名。可以通过类型别名和函数类型中的命名参数来实现。这样，可以在坚持使用函数类型的同时获得 SAM 的所有优势。

# 函数类型中的命名参数

到目前为止，我们只看到了函数类型的定义，其中只指定了类型，而没有指定参数名称。参数名称已在 *function literals* 中指定：

```kt
    fun setOnItemClickListener(listener: (Int, View, View)->Unit) { 
        // code 
    } 
    setOnItemClickListener { position, view, parent -> /* ... */ } 
```

当参数不是自解释的，开发人员不知道参数的含义时就会出现问题。在 SAM 中有建议，而在前面示例中定义的函数类型中，它们并不真正有帮助：

![](img/Image00037.jpg)

解决方案是使用带有命名参数的函数类型。以下是它的样子：

```kt
    (position: Int, view: View, parent: View)->Unit 
```

这种表示法的好处是，IDE 建议这些名称作为 *function literal* 中参数的名称。由于这个原因，程序员可以避免任何混淆：

![](img/Image00038.jpg)

当同一函数类型多次使用时，定义每个函数类型的参数并不容易。在这种情况下，使用了不同的 Kotlin 功能 - 我们在下一节中描述的 *type alias* 。

# 类型别名

从 1.1 版本开始，Kotlin 具有称为 **type alias** 的功能，允许我们为现有类型提供替代名称。以下是一个类型别名定义的示例，我们已经制作了一个 `Users` 的列表：

```kt
    data class User(val name: String, val surname: String) 
    typealias Users = List<User> 
```

这样，我们可以为现有的数据类型添加更有意义的名称：

```kt
    typealias Weight = Double 
    typealias Length = Int 
```

类型别名必须在顶层声明。可以应用可见性修饰符来调整类型别名的范围，但它们默认是公共的。这意味着之前定义的类型别名可以无限制地使用：

```kt
    val users: Users = listOf( 
        User("Marcin", "Moskala"),  
        User("Igor", "Wojda") 
    ) 

    fun calculatePrice(length: Length) { 
        // ... 
    } 
    calculatePrice(10) 

    val weight: Weight = 52.0 
    val length: Length = 34 
```

请记住，别名用于提高代码的可读性，原始类型仍然可以互换使用：

```kt
    typealias Length = Int 
    var intLength: Int = 17 
    val length: Length = intLength 
    intLength = length 
```

`typealias`的另一个应用是缩短长泛型类型并为其提供更有意义的名称。当相同类型在代码中的多个位置使用时，这可以提高代码的可读性和一致性：

```kt
    typealias Dictionary<V> = Map<String, V> 
    typealias Array2D<T> = Array<Array<T>> 
```

类型别名通常用于命名函数类型：

```kt
    typealias Action<T> = (T) -> Unit 
    typealias CustomHandler = (Int, String, Any) -> Unit 
```

我们可以将它们与函数类型参数名称一起使用：

```kt
    typealias OnElementClicked = (position: Int, view: View, parent: View)->Unit 
```

然后我们得到参数建议：

![](img/Image00039.jpg)

让我们看一个例子，函数类型由类型别名命名的方法可以通过类实现。在这个例子中，函数类型的参数名称也建议作为方法参数名称：

```kt
    typealias OnElementClicked = (position: Int, view: View, parent: View)->Unit 

    class MainActivity: Activity(), OnElementClicked { 

        override fun invoke(position: Int, view: View, parent: View) { 
            // code 
        } 
    } 
```

这些是我们使用命名函数类型的主要原因：

+   名称通常比整个函数类型定义更短更容易

+   当我们传递函数时，在更改其定义后，如果我们使用类型别名，则无需在所有地方都进行更改

+   当我们使用类型别名时，更容易定义参数名称

这两个特性（函数类型中的命名参数和类型别名）的结合是 Kotlin 中不需要定义 SAM 的原因--所有 SAM 相对于函数类型的优势（名称和命名参数）都可以通过函数类型定义中的命名参数和类型别名来实现。这是 Kotlin 支持函数式编程的另一个例子。

# 未使用的变量下划线

在某些情况下，我们正在定义一个不使用所有参数的 lambda 表达式。当我们保留它们的名称时，可能会对阅读此 lambda 表达式并尝试理解其目的的程序员进行解构。让我们看一下过滤每个第二个元素的函数。第二个参数是元素值，在这个例子中没有使用：

```kt
    list.filterIndexed { index, value -> index % 2 == 0 } 
```

为了防止误解，有一些惯例，比如忽略参数名称：

```kt
    list.filterIndexed { index, ignored -> index % 2 == 0 } 
```

由于这些约定不清晰且有问题，Kotlin 引入了下划线表示法，用作未使用的参数的名称的替代：

```kt
    list.filterIndexed { index, _ -> index % 2 == 0 } 
```

建议使用此表示法，并在未使用 lambda 表达式参数时显示警告：

![](img/Image00040.jpg)

# 在 lambda 表达式中解构

在第四章 *类和对象* 中，我们已经看到了如何使用解构声明将对象解构为多个属性：

```kt
data class User(val name: String, val surname: String, val phone: String) 

val (name, surname, phone) = user 
```

自 1.1 版本以来，Kotlin 可以使用解构声明语法来进行 lambda 参数。要使用它们，您应该使用包含我们想要解构的所有参数的括号：

```kt
    val showUser: (User) -> Unit = { (name, surname, phone) -> 
        println("$name $surname have phone number: $phone")  
    } 

    val user = User("Marcin", "Moskala", "+48 123 456 789") 
    showUser(user) 

    // Marcin Moskala have phone number: +48 123 456 789 
```

Kotlin 的解构声明是基于位置的，与基于属性名称的解构声明相反，例如在 TypeScript 中可以找到。在基于位置的解构声明中，属性的顺序决定了分配给哪个变量。在基于属性名称的解构中，它由变量的名称决定：

`//TypeScript`

`const obj = { first: 'Jane', last: 'Doe' };`

`const { last, first } = obj;`

`console.log(first); // 打印：Jane`

`console.log(last); // 打印：Doe`

这两种解决方案各有优缺点。基于位置的解构声明对于重命名属性是安全的，但对于属性重新排序是不安全的。基于名称的解构声明对于属性重新排序是安全的，但对于属性重命名是脆弱的。

解构声明可以在单个 lambda 表达式中多次使用，并且可以与普通参数一起使用：

```kt
    val f1: (Pair<Int, String>)->Unit = { (first, second) -> 

        /* code */ } // 1 
    val f2: (Int, Pair<Int, String>)->Unit = { index, (f, s)-> 

        /* code */ } // 2 
    val f3: (Pair<Int, String>, User) ->Unit = { (f, s), (name, 

        surname, tel) ->/* code */ } // 3 
```

1.  `Pair`的解构

1.  `Pair`和其他元素的解构

1.  单个 lambda 表达式中的多个解构

请注意，我们可以将类解构为不到所有组件：

```kt
    val f: (User)->Unit = { (name, surname) -> /* code */ } 
```

解构声明中允许使用下划线表示法。它最常用于获取更多的组件：

```kt
    val f: (User)->Unit = { (name, _, phone) -> /* code */ } 
    val third: (List<Int>)->Int = { (_, _, third) -> third } 
```

可以指定解构参数的类型：

```kt
    val f = { (name, surname): User -> /* code */ } //1
```

1.  类型是从 lambda 表达式中推断出来的

还有解构声明定义的参数：

```kt
    val f = { (name: String, surname: String): User -> 

       /* code */}// 1 
    val f: (User)->Unit = { (name, surname) -> 

      /* code */ } // 2 
```

1.  类型是从 lambda 表达式中推断出来的。

1.  由于 lambda 表达式内部的类型信息不足，无法推断类型。

所有这些都使得 lambda 中的解构成为一个非常有用的特性。让我们看一些在 Android 中使用解构在 lambda 中的最常见用例。它用于处理`Map`的元素，因为它们是`Map.Entry`类型，可以被解构为`key`和`value`参数：

```kt
    val map = mapOf(1 to 2, 2 to "A") 
    val text = map.map { (key, value) -> "$key: $value" } 
    println(text) // Prints: [1: 2, 2: A] 
```

同样，成对的列表也可以被解构：

```kt
    val listOfPairs = listOf(1 to 2, 2 to "A") 
    val text = listOfPairs.map { (first, second) -> 

        "$first and $second" } 
    println(text) // Prints: [1 and 2, 2 and A] 
```

解构声明在我们想要简化数据对象处理时也会被使用：

```kt
    fun setOnUserClickedListener(listener: (User)->Unit) { 
        listView.setOnItemClickListener { _, _, position, _ -> 
            listener(users[position]) 
        } 
    } 

    setOnUserClickedListener { (name, surname) -> 
        toast("Clicked to $name $surname") 
    } 
```

这在用于异步处理元素的库中特别有用（例如 RxJava）。它们的函数被设计用于处理单个元素，如果我们想要处理多个元素，那么我们需要将它们打包在`Pair`、`Triple`或其他一些数据类中，并在每一步使用解构声明：

```kt
getQuestionAndAnswer() 
    .flatMap { (question, answer) -> 

      view.showCorrectAnswerAnimationObservable(question, answer)  
    }   
    .subscribe( { (question, answer) -> /* code */ } ) 
```

# 内联函数

高阶函数非常有用，可以真正提高代码的可重用性。然而，使用它们的最大担忧之一是效率。Lambda 表达式被编译为类（通常是匿名类），在 Java 中对象的创建是一个繁重的操作。我们仍然可以以有效的方式使用高阶函数，同时保持所有的好处，方法是使函数内联。

内联函数的概念相当古老，主要与 C++或 C 有关。当一个函数被标记为内联时，在代码编译期间，编译器将所有函数调用替换为函数的实际体。此外，作为参数提供的 lambda 表达式将被替换为它们的实际体。它们不会被视为函数，而是作为实际的代码。这使得字节码更长，但运行时执行更加高效。后来，我们会看到标准库中几乎所有的高阶函数都被标记为内联。让我们看一个例子。假设我们用`inline`修饰符标记了`printExecutionTime`函数：

```kt
    inline fun printExecutionTime(f: () -> Unit) { 
        val startTime = System.currentTimeMillis() 
        f() 
        val endTime = System.currentTimeMillis() 
        println("It took " + (endTime - startTime))  
    } 

    fun measureOperation() { 
        printExecutionTime { 
            longOperation() 
        } 
    } 
```

当我们编译和反编译`measureOperation`时，我们会发现函数调用被其实际体替换，参数函数调用被 lambda 表达式的体替换：

```kt
    fun measureOperation() { 
        val startTime = System.currentTimeMillis() // 1 
        longOperation() // 2 
        val endTime = System.currentTimeMillis() 
        println("It took " + (endTime - startTime)) 
    } 
```

1.  来自`printExecutionTime`的代码被添加到`measureOperation`函数体中。

1.  位于 lambda 内部的代码位于其调用处。如果函数多次使用它，那么代码将替换每个调用。

`printExecutionTime`的体仍然可以在代码中找到。为了使示例更易读，它被跳过了。它被保留在代码中，因为在编译后可能会被使用，例如，如果这段代码被添加到项目中作为库。而且，当被 Kotlin 使用时，这个函数仍然会作为内联函数工作。

虽然不需要为 lambda 表达式创建类，但内联函数可以加速带有函数参数的函数的执行。这种差异非常重要，建议对至少有一个函数参数的所有短函数使用内联修饰符。不幸的是，使用内联修饰符也有其不好的一面。首先，我们已经提到了--生成的字节码更长。这是因为函数调用被函数体替换，而在此体内部的 lambda 调用被*函数文字*的体替换。此外，内联函数不能是递归的，也不能使用比此 lambda 表达式更严格的可见性修饰符的函数或类。例如，公共内联函数不能使用私有函数。原因是这可能导致代码注入到不能使用它们的函数中。这将导致编译错误。为了防止这种情况发生，Kotlin 不允许在放置它们的 lambda 表达式中使用比 lambda 表达式更不严格的修饰符的元素。这里有一个例子：

```kt
    internal fun someFun() {}  
    inline fun inlineFun() { 
        someFun() // ERROR 
    } 
```

事实上，在 Kotlin 中，如果我们抑制此警告，可以在`inline`函数中使用更严格可见性的元素，但这是不好的做法，绝不能这样使用：

`// Tester1.kt`

`fun main(args: Array<String>) { a() }`

`// Tester2.kt`

`inline fun a() { b() }`

`private fun b() { print("B") }` 这是怎么可能的？对于内部修饰符来说更简单，因为内部修饰符在底层是公共的。对于私有函数，会创建一个额外的`access$b`函数，它具有`public`可见性，并且只调用`b`函数：

`public static final void access$b() { b(); }`

这种行为只是为了解释为什么在`inline`函数内有时可以使用较不严格的修饰符（这些情况可以在 Kotlin 1.1 的 Kotlin 标准库中找到）。在项目中，我们应该设计元素，以便不需要使用这样的抑制。

另一个问题不太直观。虽然没有创建 lambda，但我们无法将函数类型的参数传递给另一个函数。这是一个例子：

```kt
    fun boo(f: ()->Int) { 
        //...  
    } 

    inline fun foo(f: () -> Int) { 
        boo (f) // ERROR, 1 
    } 
```

当函数是`inline`时，它的函数参数不能传递给不是内联的函数。

这不起作用，因为没有创建`f`参数。它只是被定义为由*函数字面值*主体替换。这就是为什么它不能作为参数传递给另一个函数。

处理它的最简单方法是将`boo`函数也设置为内联。然后就可以了。在大多数情况下，我们不能使太多的函数内联。以下是一些原因：

+   `inline`函数应该用于较小的函数。如果我们正在创建使用其他`inline`函数的`inline`函数，那么在编译后可能会生成一个大的结构。这是一个问题，因为编译时间和生成的代码大小都会受到影响。

+   虽然`inline`函数不能使用比它们更严格的可见性修饰符的元素，但如果我们想在库中使用它们，这将是一个问题，因为尽可能多的函数应该是私有的，以保护 API。

处理这个问题的最简单方法是将我们想要传递给另一个函数的函数参数设置为`noinline`。

# `noinline`修饰符

`noinline`是函数类型参数的修饰符。它使特定参数被视为普通函数类型参数（其调用不会被*函数字面值*主体替换）。让我们看一个`noinline`的例子：

```kt
    fun boo(f: ()->Unit) { 
        //... 
    } 

    inline fun foo(before: ()->Unit, noinline f: () -> Unit) { // 1 
        before() // 2 
        boo (f) // 3 
    } 
```

1.  在参数`f`之前使用`noinline`注解修饰符。

1.  之前的函数将被用作参数的 lambda 表达式的主体所替换。

1.  `f`是`noinline`，所以可以将其传递给`boo`函数。

使用`noinline`修饰符的两个主要原因如下：

+   当我们需要将特定的 lambda 传递给其他函数时

+   当我们频繁调用 lambda 并且不希望代码膨胀太多时

请注意，当我们将所有函数参数都设置为`noinline`时，几乎不会因为将函数设置为内联而获得性能提升。虽然使用`inline`可能不会有益，但编译器会显示警告。这就是为什么在大多数情况下，只有在有多个函数参数时才使用`noinline`，并且我们只将其应用于其中一些参数。

# 非局部返回

具有函数参数的函数可能类似于本地结构（例如循环）。我们已经看到了`ifSupportsLolipop`函数和`repeatUntilError`函数。更常见的例子是`forEach`修饰符。它是`for`控制结构的替代品，并且依次调用每个元素的参数函数。它的实现方式如下（在 Kotlin 标准库中有一个`forEach`修饰符，但我们稍后会看到它，因为它包含了尚未介绍的元素）：

```kt
    fun forEach(list: List<Int>, body: (Int) -> Unit) { 
        for (i in list) body(i) 
    } 

    // Usage 
    val list = listOf(1, 2, 3, 4, 5) 
    forEach(list) { print(it) } // Prints: 12345 
```

一个大问题是，在这种方式定义的`forEach`函数内部，我们无法从外部函数返回。例如，我们可以使用`for`循环来实现`maxBounded`函数：

```kt
    fun maxBounded(list: List<Int>, upperBound: Int, lowerBound: Int): 

    Int { 
        var currentMax = lowerBound 
        for(i in list) { 
            when { 
                i > upperBound -> return upperBound 
                i > currentMax -> currentMax = i 
            } 
        } 
        return currentMax 
    } 
```

如果我们希望将`forEach`作为`for`循环的替代方案，那么应该允许类似的可能性。问题在于，相同的代码，但使用`forEach`而不是`for`循环，将无法编译：

![](img/Image00041.jpg)

原因与代码的编译方式有关。我们已经讨论过，lambda 表达式被编译为包含定义代码的匿名对象的类，而在那里我们无法从`maxBounded`函数中返回，因为我们处于不同的上下文中。

当`forEach`函数被标记为内联时，我们会遇到一种情况。正如我们已经提到的，这个函数的主体在编译期间会替换其调用，参数中的所有函数都会被其主体替换。因此，在那里使用`return`修饰符是没有问题的。然后，如果我们将`forEach`设置为内联，我们可以在 lambda 表达式中使用 return：

```kt
    inline fun forEach(list: List<Int>, body: (Int)->Unit) { 
        for(i in list) body(i) 
    } 

    fun maxBounded(list: List<Int>, upperBound: Int, 

        lowerBound: Int): Int { 
        var currentMax = lowerBound 
        forEach(list) { i -> 
            when { 
                i > upperBound -> return upperBound 
                i > currentMax -> currentMax = i 
            } 
        } 
        return currentMax 
    } 
```

这就是`maxBounded`函数在 Kotlin 中的编译方式，当它被反编译为 Java 时，代码看起来是这样的（经过一些清理和简化）：

```kt
    public static final int maxBounded(@NotNull List list, 

    int upperBound, int lowerBound) { 
        int currentMax = lowerBound; 
        Iterator iter = list.iterator(); 

        while(iter.hasNext()) { 
            int i = ((Number)iter.next()).intValue(); 
            if(i > upperBound) { 
                return upperBound; // 1 
            } 

            if(i > currentMax) { 
                currentMax = i; 
            } 
        } 

        return currentMax; 
    } 
```

在上面的代码中，`return`很重要--它在 lambda 表达式中被定义，并且从`maxBounded`函数中返回。

在`inline`函数的 lambda 表达式中使用的`return`修饰符称为非局部返回。

# 在 lambda 表达式中标记返回

让我们看一个需要从 lambda 表达式返回而不是从函数返回的情况。我们可以使用标签来实现这一点。以下是使用标签从 lambda 表达式返回的示例：

```kt
    inline fun <T> forEach(list: List<T>, body: (T) -> Unit) { // 1 
        for (i in list) body(i) 
    } 

    fun printMessageButNotError(messages: List<String>) { 
        forEach(messages) messageProcessor@ { // 2 
            if (it == "ERROR") return@messageProcessor // 3 
            print(it) 
        } 
    } 

    // Usage 
    val list = listOf("A", "ERROR", "B", "ERROR", "C") 
    processMessageButNotError(list) // Prints: ABC 
```

1.  这是`forEach`函数的通用实现，可以处理任何类型的列表。

1.  我们为`forEach`参数中的 lambda 表达式定义标签。

1.  我们从由标签指定的 lambda 表达式中返回。

另一个 Kotlin 特性是，作为函数参数定义的 lambda 表达式具有一个默认标签，其名称与它们所定义的函数相同。这个标签称为**隐式标签**。当我们想要从`forEach`函数中定义的 lambda 表达式返回时，我们可以通过使用`return@forEach`来实现。让我们看一个例子：

```kt
    inline fun <T> forEach(list: List<T>, body: (T) -> Unit) { // 1 
        for (i in list) body(i) 
    } 

    fun processMessageButNotError(messages: List<String>) { 
        forEach(messages) { 
            if (it == "ERROR") return@forEach // 1 
            process(it) 
        } 
    } 

    // Usage 
    val list = listOf("A", "ERROR", "B", "ERROR", "C") 
    processMessageButNotError(list) // Prints: ABC 
```

1.  隐式标签名称取自函数名称。

请注意，虽然`forEach`函数是内联的，我们也可以使用非局部返回来从`processMessageButNotError`函数中返回：

```kt
    inline fun <T> forEach(list: List<T>, body: (T) -> Unit) { 
        for (i in list) body(i) 
    }  

    fun processMessageButNotError(messages: List<String>) { 
        forEach(messages) { 
            if (it == "ERROR") return 
            process(it) 
        } 
    } 

    // Usage 
    val list = listOf("A", "ERROR", "B", "ERROR", "C") 
    processMessageButNotError(list) // Prints: A 
```

让我们来看一个更复杂的使用非局部返回标签的例子。假设我们有两个`forEach`循环，一个嵌套在另一个内部。当我们使用隐式标签时，它将从更深层的循环中返回。在我们的例子中，我们可以用它来跳过特定消息的处理：

```kt
    inline fun <T> forEach(list: List<T>, body: (T) -> Unit) { // 1 
        for (i in list) body(i)  
    } 

    fun processMessageButNotError(conversations: List<List<String>>) { 
        forEach(conversations) { messages -> 
            forEach(messages) { 
                if (it == "ERROR") return@forEach // 1\. 
                process(it) 
            } 
        } 
    } 

    // Usage 
    val conversations = listOf( 
        listOf("A", "ERROR", "B"),  
        listOf("ERROR", "C"),  
        listOf("D") 
    ) 
    processMessageButNotError(conversations) // ABCD 
```

1.  这将从`forEach`函数中定义的 lambda 中返回，该函数还将消息作为参数。

我们不能使用隐式标签从同一上下文中的另一个 lambda 表达式中返回，因为它被更深层次的隐式标签所遮蔽。

在这些情况下，我们需要使用非局部隐式标签返回。这只允许在内联函数参数中使用。在我们的例子中，当`forEach`是内联的时，我们可以通过这种方式从*函数字面值*返回：

```kt
    inline fun <T> forEach(list: List<T>, body: (T) -> Unit) { // 1 
        for (i in list) body(i) 
    } 

    fun processMessageButNotError(conversations: List<List<String>>) { 
        forEach(conversations) conv@ { messages -> 
            forEach(messages) { 
                if (it == "ERROR") return@conv // 1\. 
                print(it) 
            } 
        } 
    } 

    // Usage 
    val conversations = listOf( 
        listOf("A", "ERROR", "B"), 
        listOf("ERROR", "C"), 
        listOf("D") 
    ) 
    processMessageButNotError(conversations) // AD 
```

1.  这将从在 conversations 上调用的`forEach`中定义的 lambda 中返回。

我们也可以只使用非局部返回（没有任何标签的返回）来完成处理：

```kt
    inline fun <T> forEach(list: List<T>, body: (T) -> Unit) { // 1 
        for (i in list) body(i) 
    } 

    fun processMessageButNotError(conversations: List<List<String>>) { 
        forEach(conversations) { messages -> 
            forEach(messages) { 
                if (it == "ERROR") return // 1\. 
                process(it) 
           } 
        } 
    } 
```

1.  这将从`processMessageButNotError`函数中返回并完成处理。

# Crossinline 修饰符

有时，我们需要在内联函数的函数类型参数中，不直接在函数体中使用，而是在另一个执行上下文中使用，比如本地对象或嵌套函数。但是内联函数的标准函数类型参数不允许以这种方式使用，因为它们允许非局部返回，如果这个函数可以在另一个执行上下文中使用，就不应该允许这种情况。为了通知编译器不允许非局部返回，这个参数必须被注释为`crossinline`。然后它将像我们在`inline`函数中期望的替换一样起作用，即使它在另一个 lambda 表达式中使用时：

```kt
    fun boo(f: () -> Unit) { 
        //... 
    } 

    inline fun foo(crossinline f: () -> Unit) { 
        boo { println("A"); f() } 
    } 

    fun main(args: Array<String>) { 
        foo { println("B") } 
    } 
```

这将被编译如下：

```kt
    fun main(args: Array<String>) { 
        boo { println("A"); println("B") } 
    } 
```

虽然没有使用函数创建属性，但不可能将`crossinline`参数传递给另一个函数作为参数：

![](img/Image00042.jpg)

让我们看一个实际的例子。在 Android 中，我们不需要`Context`来在应用程序的主线程上执行操作，因为我们可以使用`Looper`类的`getMainLooper`静态函数获取主循环。因此，我们可以编写一个顶级函数，允许简单地将线程切换到主线程。为了优化它，我们首先检查当前线程是否不是主线程。当它是时，操作就被调用。当它不是时，我们创建一个在主线程上操作的处理程序，并进行一个后续操作以从那里调用它。为了加快此函数的执行，我们将使`runOnUiThread`函数内联，但然后为了允许从另一个线程调用操作，我们需要使它`crossinline`。这是这个描述的函数的实现：

```kt
    inline fun runOnUiThread(crossinline action: () -> Unit) { 
        val mainLooper = Looper.getMainLooper() 
        if (Looper.myLooper() == mainLooper) { 
            action() 
        } else { 
            Handler(mainLooper).post { action() } // 1 
        } 
    } 
```

1.  我们可以通过`crossinline`修饰符在 lambda 表达式中运行`action`。

`crossinline`注解很有用，因为它允许在 lambda 表达式或本地函数的上下文中使用函数类型，同时保持使函数`inline`的优势（在这种情况下不需要 lambda 创建）。

# 内联属性

自 Kotlin 1.1 以来，`inline`修饰符可以用于没有后备字段的属性。它可以应用于单独的访问器，这将导致它们的主体替换使用，或者它可以用于整个属性，这将产生与使两个访问器都是内联的相同结果。让我们创建一个内联属性，用于检查和更改元素的可见性。这是一个实现，其中两个访问器都是内联的：

```kt
var viewIsVisible: Boolean 
inline get() = findViewById(R.id.view).visibility == View.VISIBLE 
inline set(value) { 
  findViewById(R.id.view).visibility = if (value) View.VISIBLE 

  else View.GONE 
} 
```

如果我们将整个属性标注为内联，也可以实现相同的结果：

```kt
inline var viewIsVisible: Boolean 
get() = findViewById(R.id.view).visibility == View.VISIBLE 
  set(value) { 
    indViewById(R.id.view).visibility = if (value) View.VISIBLE 

      else View.GONE 
    } 

// Usage 
if (!viewIsVisible) 
viewIsVisible = true 
```

前面的代码将被编译如下：

```kt
if (!(findViewById(R.id.view).getVisibility() == View.VISIBLE)) 

{ 
  findViewById(R.id.view).setVisibility(true?View.VISIBLE:View.GONE); 
} 
```

这样，我们省略了 setter 和 getter 函数调用，并且应该期望在编译代码大小增加的代价下获得性能改进。尽管如此，对于大多数属性来说，使用`inline`修饰符应该是有利的。

# 函数引用

有时，我们想要作为参数传递的函数已经定义为一个单独的函数。然后我们可以只定义带有其调用的 lambda：

```kt
    fun isOdd(i: Int) = i % 2 == 1 

    list.filter { isOdd(it) } 
```

但 Kotlin 也允许我们将函数作为值传递。为了能够将顶级函数用作值，我们需要使用函数引用，它用作双冒号和函数名（`::functionName`）。这是一个例子，说明它如何用于为`filter`提供谓词：

```kt
    list.filter(::isOdd) 
```

这是一个例子：

```kt
    fun greet(){ 
        print("Hello! ") 
    } 

    fun salute(){ 
        print("Have a nice day ") 
    } 

    val todoList: List<() -> Unit> = listOf(::greet, ::salute) 

    for (task in todoList) { 
        task()  
    } 

    // Prints: Hello! Have a nice day 
```

函数引用是反射的一个例子，这就是为什么这个操作返回的对象也包含有关所引用函数的信息：

```kt
    fun isOdd(i: Int) = i % 2 == 1 

    val annotations = ::isOdd.annotations 
    val parameters = ::isOdd.parameters   
    println(annotations.size) // Prints: 0 
    println(parameters.size) // Prints: 1 
```

但这个对象也实现了函数类型，可以这样使用：

```kt
    val predicate: (Int)->Boolean = ::isOdd 
```

还可以引用方法。要这样做，我们需要写类型名称，两个冒号和方法名（`Type::functionName`）。这是一个例子：

```kt
    val isStringEmpty: (String)->Boolean = String::isEmpty 

    // Usage 
    val nonEmpty = listOf("A", "", "B", "") 
    .filter(String::isNotEmpty) 
    print(nonEmpty) // Prints: ["A", "B"] 
```

与前面的例子类似，当我们引用非静态方法时，需要提供类的实例作为参数。`isEmpty`函数是一个不带参数的`String`方法。对`isEmpty`的引用有一个`String`参数，该参数将被用作调用该函数的对象。对象的引用总是位于第一个参数。这里是另一个例子，其中方法已经定义了属性`food`：

```kt
    class User { 

        fun wantToEat(food: Food): Boolean { 
            // ... 
        } 
    } 

    val func: (User, Food) -> Boolean = User::wantToEat 
```

当我们引用 Java 静态方法时，情况就不同了，因为它不需要定义它所在的类的实例。这类似于*对象*或*伴生对象*的方法，其中对象是预先知道的，不需要提供。在这些情况下，有一个与被引用函数相同参数和相同返回类型的函数被创建：

```kt
    object MathHelpers { 
        fun isEven(i: Int) = i % 2 == 0 
    } 

    class Math { 
        companion object { 
            fun isOdd(i: Int) = i % 2 == 1 
        } 
    } 

    // Usage 
    val evenPredicate: (Int)->Boolean = MathHelpers::isEven 
    val oddPredicate: (Int)->Boolean = Math.Companion::isOdd 

    val numbers = 1..10 
    val even = numbers.filter(evenPredicate) 
    val odd = numbers.filter(oddPredicate) 
    println(even) // Prints: [2, 4, 6, 8, 10] 
    println(odd) // Prints: [1, 3, 5, 7, 9] 
```

在函数引用的使用中，有一些常见的用例，我们希望使用函数引用来提供对我们引用的类的方法。常见的例子是当我们想要将一些操作提取为同一类的方法时，或者当我们想要引用来自我们引用的类的引用成员函数的函数时。一个简单的例子是当我们定义网络操作之后应该做什么。它是在一个 Presenter（比如`MainPresenter`）中定义的，但它引用了所有的 View 操作，这些操作由`view`属性定义（例如，类型为`MainView`）：

```kt
    getUsers().smartSubscribe ( 
        onStart = { view.showProgress() }, // 1 
        onNext = { user -> onUsersLoaded(user) }, // 2 
        onError = { view.displayError(it) }, // 1 
        onFinish = { view.hideProgress() } // 1 
    ) 
```

1.  `showProgress`，`displayError`和`hideProgress`在`MainView`中定义。

1.  `onUsersLoaded`是在`MainPresenter`中定义的方法。

为了帮助这种情况，Kotlin 在 1.1 版本中引入了一个叫做**bound references**的功能，它提供了绑定到特定对象的引用。由于这个，这个对象不需要通过参数提供。使用这种表示法，我们可以用以下方式替换之前的定义：

```kt
    getUsers().smartSubscribe ( 
        onStart = view::showProgress, 
        onNext = this::onUsersLoaded, 
        onError = view::displayError, 
        onFinish = view::hideProgress 
    ) 
```

我们可能想要引用的另一个函数是构造函数。一个例子是当我们需要从**数据传输对象**（**DTO**）映射到模型中的类时：

```kt
    fun toUsers(usersDto: List<UserDto>) = usersDto.map { User(it) } 
```

在这里，`User`需要有一个构造函数，定义了它如何从`UserDto`构造。

DTO 是在进程之间传递数据的对象。它被使用是因为在系统之间的通信中使用的类（在 API 中）与系统内部使用的实际类（模型）不同。

在 Kotlin 中，构造函数的使用和处理类似于函数。我们也可以用双冒号和类名引用它们：

```kt
    val mapper: (UserDto)->User = ::User 
```

这样，我们可以用构造函数引用替换 lambda 的构造函数调用：

```kt
    fun toUsers(usersDto: List<UserDto>) = usersDto.map(::User) 
```

使用函数引用而不是 lambda 表达式给我们提供了更短和更易读的表示法。当我们传递多个函数作为参数，或者函数很长需要被提取时，这种表示法尤其有用。在其他情况下，有一个有用的 bounded reference，它提供了一个绑定到特定对象的引用。

# 总结

在本章中，我们讨论了将函数作为一等公民。我们已经看到了函数类型是如何被使用的。我们已经看到了如何定义*函数字面量*（匿名函数和 lambda 表达式），以及任何函数都可以作为对象使用，这要归功于函数引用。我们还讨论了高阶函数和不同的 Kotlin 特性来支持它们：单个参数的隐式名称、参数中的最后一个 lambda、Java SAM 支持、使用下划线表示未使用的变量，以及 lambda 表达式中的解构声明。这些特性为高阶函数提供了很好的支持，使函数不仅仅是一等公民。

在下一章中，我们将看到 Kotlin 中泛型是如何工作的。这将使我们能够定义更强大的类和函数。我们还将看到当与高阶函数连接时它们可以如何被使用。
