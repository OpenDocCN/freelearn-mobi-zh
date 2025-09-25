# 使用 Kotlin 和 RxKotlin 进行函数式编程

函数式编程范式与面向对象编程（**OOP**）略有不同。它侧重于使用声明性和表达性的程序以及不可变数据，而不是语句。函数式编程的定义指出：“函数式编程是一种编程系统，它依赖于将程序结构化为不可变数据的数学函数的评估，并避免状态改变。”它是一种声明性编程范式，建议使用小型、可重用的声明性函数。

我们已经看到了函数式编程的定义；现在，你难道不想深入了解它的定义，看看它究竟意味着什么吗？所有语言都支持函数式编程吗？如果不是，那么哪些语言支持，Kotlin 又如何呢？响应式编程与函数式编程究竟有什么关系？最后，我们需要学习什么，才能进行函数式编程？

在本章中，我们将涵盖以下主题：

+   函数式编程入门

+   函数式编程与响应式编程的关系

+   Kotlin 的突破性特性——协程

# 介绍函数式编程

因此，函数式编程希望您将编程逻辑分布到可重用的声明性小函数中。将您的逻辑分布到小块代码中会使代码模块化且不复杂，因此您可以在任何给定点重构/更改任何模块/代码的一部分，而不会对其他模块产生影响。

函数式编程需要一些接口和语言的支持，因此我们不能说任何语言都是函数式的，除非它提供某种支持来实现函数式编程。然而，函数式编程并不是什么新事物；实际上，它是一个相当古老的概念，并且有几种语言支持它。我们称这些语言为函数式编程语言，以下是一些最受欢迎的函数式编程语言的列表：

+   Lisp

+   Clojure

+   Wolfram

+   Erlang

+   OCaml

+   Haskell

+   Scala

+   F#

Lisp 和 Haskell 是一些最古老的语言，并且至今仍在学术界和工业界中使用。当谈到 Kotlin 时，它从第一个稳定版本开始就提供了对函数式编程的出色支持，与 Java 相比，Java 在 Java 8 之前没有任何对函数式编程的支持。你可以在面向对象和函数式编程风格之间，甚至两种风格的混合中使用 Kotlin，这对我们来说真的是一个巨大的好处。Kotlin 对诸如高阶函数、函数类型和 lambda 等特性提供了第一级支持，如果你正在做或探索函数式编程，Kotlin 是一个不错的选择。

**函数式响应式编程**（FRP）的概念实际上是将响应式编程与函数式编程相结合的产物。编写函数式编程的主要目的是实现模块化编程；这种模块化编程对于实现响应式编程或更确切地说，实现响应式宣言的四个原则非常有帮助，有时甚至是必需的。

# 函数式编程基础

函数式编程包含一些新的概念，如 Lambda、纯函数、高阶函数、函数类型和内联函数，我们将学习这些内容。非常有趣，不是吗？

注意，尽管在许多程序员的词汇中，纯函数和 Lambda 是相同的，但实际上它们并不相同。在本章的后续部分，我们将了解更多关于它们的内容。

# Lambda 表达式

Lambda 或 Lambda 表达式通常意味着 *匿名函数*，即没有名称的函数。你也可以说 Lambda 表达式是一个函数，但并非每个函数都是 Lambda 表达式。并非每种编程语言都支持 Lambda 表达式，例如，Java 直到 Java 8 才有这个功能。Lambda 表达式的实现也因语言而异。Kotlin 对 Lambda 表达式有很好的支持，在 Kotlin 中实现它们既简单又自然。现在让我们看看 Kotlin 中 Lambda 表达式是如何工作的：

```kt
    fun main(args: Array<String>) { 
      val sum = { x: Int, y: Int -> x + y } // (1) 
      println("Sum ${sum(12,14)}")// (2) 
      val anonymousMult = {x: Int -> (Random().nextInt(15)+1) * x}
      // (3) 
      println("random output ${anonymousMult(2)}")// (4) 
    } 
```

在前面的程序中，在注释（1）中，我们声明了一个 Lambda 表达式，该表达式将两个数字相加并返回结果 `sum`；在注释（2）中，我们调用该函数并打印它；在注释（3）中，我们声明了另一个 Lambda，它将随机数与 `15` 绑定并乘以传递给它的值 `x`，然后返回结果；在注释（4）中，我们再次打印它。这两个 Lambda 表达式实际上都是函数，但没有函数名；因此，它们也被称为匿名函数。如果你与 Java 比较，Java 有匿名类的功能，但直到 Java 8 才包含 lambda/匿名函数。

如果你好奇输出结果，请参考以下截图：

![](img/89439c0a-af28-49ca-873c-7d67b118c364.jpg)

# 纯函数

纯函数的定义是，如果函数的返回值完全依赖于其参数/输入，那么这个函数可以被称为纯函数。所以，如果我们声明一个函数为 `fun func1(x:Int):Int`，那么它的返回值将严格依赖于其参数 `x`；比如说，如果你两次调用 `func1` 并传入值 `3`，那么两次的返回值将是相同的。纯函数可以是 Lambda 或命名函数。在前面的例子中，第一个 Lambda 表达式是一个纯函数，但第二个不是，因为对于第二个来说，即使传入相同的值，其返回值也可能在不同时间不同。让我们看看以下例子来更好地理解它：

```kt
    fun square(n:Int):Int {//(1) 
      return n*n 
    } 

    fun main(args: Array<String>) { 
      println("named pure func square = ${square(3)}") 
      val qube = {n:Int -> n*n*n}//(2) 
      println("lambda pure func qube = ${qube(3)}") 
    } 
```

这里的两个函数，`(1)`和`(2)`，都是纯函数——一个是命名的，而另一个是 lambda。如果你将值`3`传递给任一函数`n`次，它们的返回值在每次都会相同。纯函数没有副作用。

**副作用**：如果一个函数或表达式修改了其作用域之外的状态，或者除了返回值之外还有可观察的与调用函数或外部世界的交互，那么它就被说成有副作用。

来源-Wikipedia [`en.wikipedia.org/wiki/Side_effect_(computer_science)`](https://en.wikipedia.org/wiki/Side_effect_(computer_science))。

需要注意的是，正如我们之前所说的，纯函数与 lambda 表达式无关，它们的定义完全不同。

以下是输出：

```kt
named pure func square = 9
lambda pure func qube = 27 
```

# 高阶函数

那些接受另一个函数作为参数或返回函数作为结果的函数被称为**高阶函数**。考虑以下示例以更好地理解它：

```kt
    fun highOrderFunc(a:Int, validityCheckFunc:(a:Int)->Boolean) {//(1) 
      if(validityCheckFunc(a)) {//(2) 
        println("a $a is Valid") 
      } else { 
        println("a $a is Invalid") 
      } 
    } 

    fun main(args: Array<String>) { 
      highOrderFun(12,{ a:Int -> a.isEven()})//(3) 
      highOrderFunc(19,{ a:Int -> a.isEven()}) 
    } 
```

在这个程序中，我们声明了一个`highOrderFunc`函数，它将接受一个`Int`和一个`validityCheckFunc(Int)`函数。我们在`highOrderFunc`函数内部调用`validityCheckFunc`函数，以检查值是否有效。然而，我们在调用`main`函数内部的`highOrderFunc`函数时定义了`validityCheckFunc`函数。

注意，在这个程序中，`isEven`函数是一个扩展函数，它是在你与书一起获得的`project`文件内部定义的。

这里是输出：

```kt
a 12 is Valid 
a 19 is Invalid 
```

# 内联函数

虽然函数是编写模块化代码的绝佳方式，但它有时可能会因为函数栈维护和开销而增加程序执行时间并减少内存优化。内联函数是避免函数式编程中这些障碍的绝佳方法。例如，请参见以下代码片段：

```kt
    fun doSomeStuff(a:Int = 0) = a+(a*a) 

    fun main(args: Array<String>) { 
      for (i in 1..10) { 
        println("$i Output ${doSomeStuff(i)}") 
      } 
    } 
```

让我们回顾一下内联函数的定义；它说，*内联函数是提高程序性能和内存优化的增强功能*。函数可以被指示给编译器，使其成为内联函数，这样编译器就可以在函数被调用的任何地方替换这些函数定义。编译器在编译时替换内联函数的定义，而不是在运行时引用函数定义；因此，不需要为函数调用、栈维护等额外内存，同时还能获得函数的好处。

上述程序声明了一个函数，用于添加两个数字并返回结果，我们将在循环中调用这个函数。而不是为这个目的声明一个函数，我们可以在将要调用函数的地方直接编写加法代码，但声明一个函数让我们可以在不影响剩余代码的情况下随时修改加法逻辑，例如，如果我们想修改加法为乘法或其他操作。如果我们将函数声明为内联的，那么该函数内部的代码将替换所有的函数调用，从而在保持自由度的同时提高性能。以下代码片段作为例子：

```kt
    inline fun doSomeStuff(a:Int = 0) = a+(a*a) 

    fun main(args: Array<String>) { 
      for (i in 1..10) { 
        println("$i Output ${doSomeStuff(i)}") 
      } 
    } 
```

这是程序的输出结果：

![](img/c9bc3123-9667-4acf-a4ff-d8ec701d34d0.jpg)

Kotlin 通过内联函数提供的一个额外功能是，如果你将一个高阶函数声明为`inline`，那么`inline`关键字将影响该函数本身以及传递给它的 lambda。让我们用`inline`修改高阶函数的代码：

```kt
    inline fun highOrderFuncInline(a:Int, validityCheckFunc:(a:Int)- 
    >Boolean) { 
      if(validityCheckFunc(a)) { 
         println("a $a is Valid") 
      } else { 
        println("a $a is Invalid") 
      } 
     } 

    fun main(args: Array<String>) { 
      highOrderFuncInline(12,{ a:Int -> a.isEven()}) 
      highOrderFuncInline(19,{ a:Int -> a.isEven()}) 
    } 
```

编译器将用其 lambda 替换所有对`validityCheckFunc`的调用，就像它会对具有定义的`highOrderFuncInline`做的那样。正如你所见，代码的修改并不多，只是在函数声明前添加了`inline`关键字，就能提高性能。

# 将函数式编程应用于`ReactiveCalculator`类

因此，现在，在尝试理解上一章中的`ReactiveCalculator`类之后，我们将尝试优化代码。让我们首先看看`ReactiveCalculator`类的`init`块：

```kt
    init{ 
      nums = Pair(a,b) 

      subjectAdd.map({ it.first+it.second }).subscribe({println
      ("Add = $it")} )//1 
      subjectSub.map({ it.first-it.second }).subscribe({println
      ("Substract = $it")} ) 
      subjectMult.map({ it.first*it.second }).subscribe
      ({println("Multiply = $it")} ) 
      subjectDiv.map({ it.first/(it.second*1.0) }).subscribe
      ({println("Divide = $it")} ) 

      subjectCalc.subscribe({ 
        with(it) { 
          calculateAddition() 
          calculateSubstraction() 
          calculateMultiplication() 
          calculateDivision() 
        } 
      }) 

      subjectCalc.onNext(this) 
     } 
```

因此，现在，有了函数式编程的知识，我们可以轻松地说，`map`和`subscribe`方法是接受函数作为参数的高阶函数。然而，你真的认为需要很多`subject`和`subscriber`吗？类上的`subscriber`不就足够完成这项工作了吗？让我们尝试修改和优化以下代码片段：

```kt
    class ReactiveCalculator(a:Int, b:Int) { 
      val subjectCalc: io.reactivex.subjects.Subject
      <ReactiveCalculator>   = 
      io.reactivex.subjects.PublishSubject.create() 

      var nums:Pair<Int,Int> = Pair(0,0) 

      init{ 
        nums = Pair(a,b) 

        subjectCalc.subscribe({ 
          with(it) { 
            calculateAddition() 
            calculateSubstraction() 
            calculateMultiplication() 
            calculateDivision() 
          } 
        }) 

        subjectCalc.onNext(this) 
       } 

       inline fun calculateAddition():Int { 
         val result = nums.first + nums.second 
         println("Add = $result") 
         return result 
       } 

       inline fun calculateSubstraction():Int { 
         val result = nums.first - nums.second 
         println("Substract = $result") 
         return result 
       } 

      inline fun calculateMultiplication():Int { 
        val result = nums.first * nums.second 
        println("Multiply = $result") 
        return result 
      } 

      inline fun calculateDivision():Double { 
        val result = (nums.first*1.0) / (nums.second*1.0) 
        println("Multiply = $result") 
        return result 
      } 

      inline fun modifyNumbers (a:Int = nums.first, b: 
      Int = nums.second) { 
        nums = Pair(a,b) 
        subjectCalc.onNext(this) 
      } 

      fun handleInput(inputLine:String?) { 
        if(!inputLine.equals("exit")) { 
            val pattern: java.util.regex.Pattern =
            java.util.regex.Pattern.compile
            ("([a|b])(?:\\s)?=(?:\\s)?(\\d*)"); 

            var a: Int? = null 
            var b: Int? = null 

            val matcher: java.util.regex.Matcher = 
            pattern.matcher(inputLine) 

            if (matcher.matches() && matcher.group(1) != null &&  
            matcher.group(2) != null) { 
                if(matcher.group(1).toLowerCase().equals("a")){ 
                    a = matcher.group(2).toInt() 
                } else if(matcher.group(1).toLowerCase().equals("b")){ 
                    b = matcher.group(2).toInt() 
                } 
            } 

            when { 
                a != null && b != null -> modifyNumbers(a, b) 
                a != null -> modifyNumbers(a = a) 
                b != null -> modifyNumbers(b = b) 
                else -> println("Invalid Input") 

            } 
         } 
      } 

    } 
```

因此，我们移除了所有其他的`subscriber`，只用一个来完成工作。以下是输出结果：

```kt
Initial Output with a = 15, b = 10
Add = 25
Substract = 5
Multiply = 150
Multiply = 1.5
Enter a = <number> or b = <number> in separate lines
exit to exit the program
a = 6
Add = 16
Substract = -4
Multiply = 60
Multiply = 0.6
b=4
Add = 10
Substract = 2
Multiply = 24
Multiply = 1.5
exit

```

我们订阅了类本身；因此，每当其变量发生变化时，我们会收到通知，并在`subscribe`方法中直接执行所有任务。此外，由于我们已经将函数声明为内联的，它们还将有助于性能优化。

# 协程

路径突破，可能是 Kotlin 中最令人兴奋的功能是协程。它们是编写异步、非阻塞代码的新方法，类似于线程，但更简单、更高效、更轻量级。协程是在 Kotlin 1.1 中添加的，并且仍然是实验性的，所以在生产环境中使用之前请三思。

在本书的后续章节中，你将了解 RxKotlin 中的调度器，它封装了线程的复杂性，但你只能在 RxKotlin 链中使用它，而协程可以在任何地方和任何时候使用。这确实是 Kotlin 的一个突破性特性。它们在线程上提供了很好的抽象，使得上下文变化和并发变得更容易。

请记住，RxKotlin 目前还没有使用协程；原因很简单——协程和 RxKotlin 中的调度器几乎具有相同的内部架构；虽然协程是新的，但调度器在 RxJava、RxJs、RxSwift 以及更多中已经存在很长时间了。

协程是开发者在不使用/无法使用 RxKotlin Schedulers 时实现并发的最佳选择。

因此，让我们先将其添加到我们的项目中。如果你正在使用 Gradle，请按照以下步骤操作（`apply plugin` 可以是 `'kotlin'` 或 `'kotlin-android'`，具体取决于你是否用于 JVM 或 Android）：

```kt
    apply plugin: 'kotlin' 

    kotlin { 
      experimental { 
        coroutines 'enable' 
      } 
    } 
```

然后，我们必须添加以下依赖项：

```kt
    repositories { 
      ... 
      jcenter() 
    } 
    dependencies { 
      ... 
      compile "org.jetbrains.kotlinx:kotlinx-coroutines-core:0.16" 
    } 
```

如果你正在使用 Maven，那么请在 `pom.xml` 文件中添加以下代码块：

```kt
    <plugin> 
      <groupId>org.jetbrains.kotlin</groupId> 
      <artifactId>kotlin-maven-plugin</artifactId> 
      ... 
      <configuration> 
        <args> 
            <arg>-Xcoroutines=enable</arg> 
        </args> 
      </configuration> 
    </plugin> 
    <repositories> 
      ... 
      <repository> 
        <id>central</id> 
        <url>http://jcenter.bintray.com</url> 
      </repository> 
    </repositories> 
    <dependencies> 
      ... 
      <dependency> 
        <groupId>org.jetbrains.kotlin</groupId> 
        <artifactId>kotlinx-coroutines-core</artifactId> 
        <version>0.16</version> 
      </dependency> 
    </dependencies> 
```

Apache Maven 是一个软件项目管理与理解工具。基于**项目对象模型**（**POM**）的概念，Maven 可以从中央信息点管理项目的构建、报告和文档。请参考以下网址获取更多信息–[`maven.apache.org/`](https://maven.apache.org/)。

那么，协程究竟是什么？在开发应用程序时，我们经常遇到需要执行长时间运行或耗时操作的情况，例如网络调用、数据库操作或一些复杂的计算。Java 中处理这种情况的唯一选项是使用线程，而这本身就是一个非常复杂的过程。每当面对这些情况时，我们都感到需要一种简单而强大的 API 来处理这些情况。来自 .NET 领域的开发者，尤其是那些之前使用过 C# 的开发者，对 `async`/`await` 操作符很熟悉；这某种程度上与 Kotlin 协程最为接近。

# 开始使用协程

因此，让我们考虑以下示例：

```kt
    suspend fun longRunningTsk():Long {//(1) 
      val time = measureTimeMillis {//(2) 
        println("Please wait") 
        delay(2,TimeUnit.SECONDS)//(3) 
        println("Delay Over") 
      } 
      return time 
    } 

    fun main(args: Array<String>) { 
      runBlocking {//(4) 
        val exeTime = longRunningTsk()//(5) 
        println("Execution Time is $exeTime") 
      } 
    } 
```

我们将检查代码，但首先让我们看看输出结果：

```kt
Please wait 
Delay Over 
Execution Time is 2018 
```

因此，现在让我们来理解代码。在注释（1）中，当我们声明函数时，我们使用`suspend`关键字标记函数，这个关键字用来标记一个函数为挂起状态，即当执行函数时，程序应该等待其结果；因此，不允许在主线程中挂起函数（这为你提供了主线程和挂起函数之间清晰的障碍）。在注释（2）中，我们使用`measureTimeMillis`启动了一个代码块，并将其值赋给了`val`类型的`time`变量。`measureInMillis`的功能相当简单——它执行传递给它的代码块，同时测量其执行时间，并返回相同的值。我们将在注释（3）中使用`delay`函数故意延迟程序执行 2 秒钟。在注释（4）中的`main`函数中的`runBlocking`代码块使程序等待直到注释（5）中调用的`longRunningTsk`函数完成。所以，这是一个相当简单的例子；然而，我们在这里使主线程等待。有时，你可能不希望这样；相反，你可能想要进行异步操作。所以，让我们也尝试实现这一点：

```kt
    fun main(args: Array<String>) { 
      val time = async(CommonPool) { longRunningTsk() }//(1) 
      println("Print after async ") 
      runBlocking { println("printing time ${time.await()}") }//(2) 
    } 
```

在这里，我们保持`longRunningTsk`不变，只是修改了`main`函数。在注释（1）中，我们将`time`变量赋值给`async`代码块内部的`longRunningTsk`。`async`代码块非常有趣；它在传递给它的协程上下文中异步执行其代码块内的代码。

基本上有三种协程上下文类型。`Unconfined`表示它将在主线程上运行，`CommonPool`在公共线程池上运行，或者你也可以创建一个新的协程上下文。

在注释（2）中，我们运行了一个阻塞代码，这将使`main`函数等待直到`time`变量的值可用；`await`函数帮助我们完成这个任务——它告诉`runBlocking`代码块等待直到`async`代码块完成执行，以便`time`变量的值可用。

# 构建序列

如我之前提到的，Kotlin 协程不仅仅是 Java 中的线程和 C#中的`async`/`await`。这是一个在学习后你会感到愤怒的功能，因为它在你学习编码时并不存在。为了锦上添花，这个功能是应用级别的，它甚至包含在`kotlin-stdlib`中，所以你可以在那里直接使用，无需做任何事情，甚至无需显式使用协程。

在学习我所说的内容之前，让我们做一些老式的代码，比如说斐波那契数列？以下代码块作为例子：

```kt
    fun main(args: Array<String>) { 
      var a = 0 
      var b = 1 
      print("$a, ") 
      print("$b, ") 

      for(i in 2..9) { 
        val c = a+b 
        print("$c, ") 
        a=b 
        b=c 
      } 
    } 
```

所以，这是用 Kotlin 编写的老式斐波那契数列程序。当你计划让用户输入要打印的数字数量时，这段代码会变得更有问题。如果我告诉你 Kotlin 有一个`buildSequence`函数可以为你完成这个任务，而且做得非常自然，方式也更简单？所以，现在让我们修改代码：

```kt
    fun main(args: Array<String>) { 
      val fibonacciSeries = buildSequence {//(1) 
        var a = 0 
        var b = 1 
        yield(a)//(2) 
        yield(b) 

        while (true) { 
            val c = a+b 
            yield(c)//(3) 
            a=b 
            b=c 
        } 
     } 

     println(fibonacciSeries.take(10) join "," )//(4) 

    } 
```

以下是对两个程序输出的描述：

```kt
0, 1, 1, 2, 3, 5, 8, 13, 21, 34 
```

现在，让我们理解这个程序。在注释（1）中，我们声明`val fibonacciSeries`由`buildSequence`块填充。每当我们在序列/数列中计算出一个要输出的值时，我们将输出那个值（在注释`2`和`3`中）。在注释`4`中，我们调用`fibonacciSeries`来计算到第`10`个变量，并用逗号（`,`）连接序列的元素。

因此，你学习了协程；现在，让我们将其实现到我们的程序中。

# 带有协程的`ReactiveCalculator`类

到目前为止，在`ReactiveCalculator`程序中，我们都在同一个线程上执行所有操作；你不认为我们应该异步地做这些事情吗？所以，让我们这样做：

```kt
    class ReactiveCalculator(a:Int, b:Int) { 
      val subjectCalc:
      io.reactivex.subjects.Subject<ReactiveCalculator> =
      io.reactivex.subjects.PublishSubject.create() 

      var nums:Pair<Int,Int> = Pair(0,0) 

      init{ 
        nums = Pair(a,b) 

        subjectCalc.subscribe({ 
            with(it) { 
                calculateAddition() 
                calculateSubstraction() 
                calculateMultiplication() 
                calculateDivision() 
            } 
        }) 

        subjectCalc.onNext(this) 
      } 

      inline fun calculateAddition():Int { 
        val result = nums.first + nums.second 
        println("Add = $result") 
        return result 
      } 

      inline fun calculateSubstraction():Int { 
        val result = nums.first - nums.second 
        println("Substract = $result") 
        return result 
      } 

      inline fun calculateMultiplication():Int { 
        val result = nums.first * nums.second 
        println("Multiply = $result") 
        return result 
     } 

     inline fun calculateDivision():Double { 
        val result = (nums.first*1.0) / (nums.second*1.0) 
        println("Division = $result") 
        return result 
     } 

     inline fun modifyNumbers (a:Int = nums.first, b: 
     Int = nums.second) { 
        nums = Pair(a,b) 
        subjectCalc.onNext(this) 

     } 

     suspend fun handleInput(inputLine:String?) {//1 
        if(!inputLine.equals("exit")) { 
            val pattern: java.util.regex.Pattern =  
            java.util.regex.Pattern.compile
            ("([a|b])(?:\\s)?=(?:\\s)?(\\d*)"); 

            var a: Int? = null 
            var b: Int? = null 

            val matcher: java.util.regex.Matcher =
            pattern.matcher(inputLine) 

            if (matcher.matches() && matcher.group(1) != null &&
            matcher.group(2) != null) { 
                if(matcher.group(1).toLowerCase().equals("a")){ 
                    a = matcher.group(2).toInt() 
                } else if(matcher.group(1).toLowerCase().equals("b")){ 
                    b = matcher.group(2).toInt() 
                } 
            } 

            when { 
                a != null && b != null -> modifyNumbers(a, b) 
                a != null -> modifyNumbers(a = a) 
                b != null -> modifyNumbers(b = b) 
                else -> println("Invalid Input") 

            } 
         } 
       } 

    } 

    fun main(args: Array<String>) { 
    println("Initial Out put with a = 15, b = 10") 
    var calculator: ReactiveCalculator = ReactiveCalculator(15, 10) 

    println("Enter a = <number> or b = <number> in separate lines\nexit
    to exit the program") 
    var line:String? 
    do { 
        line = readLine(); 
        async(CommonPool) {//2 
            calculator.handleInput(line) 
        } 
    } while (line!= null && !line.toLowerCase().contains("exit")) 
   } 
```

在注释（1）中，我们将`handleInput`函数声明为挂起，这告诉 JVM 这个函数应该需要更长的时间，调用此函数的上下文执行应该等待它完成。正如我之前已经提到的，挂起函数不能在主上下文中调用；所以，在注释（2）中，我们创建了一个`async`块来调用这个函数。

# 函数式编程 – 单子

函数式编程没有单子是不完整的。如果你对函数式编程感兴趣，那么你非常了解它；否则，你可能是第一次听说。那么，什么是单子呢？让我们来了解一下。单子的概念相当抽象；定义说“单子是一个结构，通过封装一个值并添加一些额外的功能来创建一个新的类型”。所以，让我们先使用单子；看看下面的程序：

```kt
    fun main(args: Array<String>) { 
      val maybeValue: Maybe<Int> = Maybe.just(14)//1 
      maybeValue.subscribeBy(//2 
        onComplete = {println("Completed Empty")}, 
        onError = {println("Error $it")}, 
        onSuccess = { println("Completed with value $it")} 
      ) 
      val maybeEmpty:Maybe<Int> = Maybe.empty()//3 
      maybeEmpty.subscribeBy( 
        onComplete = {println("Completed Empty")}, 
        onError = {println("Error $it")}, 
        onSuccess = { println("Completed with value $it")} 
      ) 
    } 
```

在这里，`Maybe`是一个封装了`Int`值并添加了一些额外功能的单子。`Maybe`单子表示它可能包含或不包含值，并且可以带或不带值或错误完成。所以，如果有错误，那么它显然会调用`onError`；如果没有错误，并且它有一个值，它将使用该值调用`onSuccess`；如果没有值也没有错误，它将调用`onComplete`。需要注意的是，这里的三个方法`onError`、`onComplete`和`onSuccess`都是终止方法，这意味着这三个中的任何一个都可能被`Maybe`单子调用，而其他的方法将永远不会被调用。

让我们逐步分析程序，以便更好地理解单子。在注释（1）中，我们将声明一个`Maybe`单子并将其赋值为`14`。在注释（2）中，我们将订阅这个单子。在注释（3）中，我们再次声明一个`Maybe`单子，这次赋予一个空值。订阅需要三个 lambda 表达式作为参数——当单子包含一个值时，`onSuccess`会被调用；当它不包含任何值时，`onComplete`会被调用；如果发生任何错误，则`onError`会被调用。现在让我们看看输出结果：

```kt
Completed with value 14 
Completed Empty 
```

因此，正如我们所看到的，对于`maybeValue`，`onSuccess`被调用，但对于`maybeEmpty`，`onComplete`方法被调用。

# 单一单子

`Maybe` 只不过是另一种类型的单子，还有很多；我们将在后面的章节中介绍其中一些最重要的，并将它们与响应式编程结合起来。

# 摘要

在本章中，我们学习了函数式编程。如果你对函数式编程的概念掌握得足够好，响应式编程的谜题将自动为你解决。我们还学习了函数式响应式编程的含义。

通过学习函数式编程，我们也对前一章中的约束有了清晰的认识。

我们还获得了对协程的介绍，这是 Kotlin 语言的一个突破性新特性。

我们已经用协程和几个新的函数式编程概念修改了我们的 `ReactiveCalculator` 类，并对其进行了优化。
