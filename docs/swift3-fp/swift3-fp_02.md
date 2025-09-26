# 第二章. 函数和闭包

在上一章中，我们概述了函数式编程和 Swift 编程语言，介绍了一些关于函数的关键概念。由于函数是函数式编程的基本构建块，本章将深入探讨它，并解释与函数式 Swift 中函数的定义和用法相关的所有方面，同时附上代码示例。

本章从函数的定义开始，继续讨论其他相关主题，如函数类型和元组，最后以更高级的主题结束，如首等函数、高阶函数、函数组合、闭包、柯里化和缓存。

本章将涵盖以下主题，并附上代码示例：

+   函数的一般语法

+   定义和使用函数参数

+   设置内部和外部参数

+   设置默认参数值

+   定义和使用可变参数函数

+   从函数返回值

+   定义和使用嵌套函数

+   函数类型

+   纯函数

+   首等函数

+   高阶函数

+   函数组合

+   自定义运算符的定义

+   定义和使用闭包

+   函数柯里化

+   递归

+   缓存

# 什么是函数？

**面向对象编程**（**OOP**）对大多数开发者来说看起来非常自然，因为它模拟了现实生活中的类或换句话说，蓝图及其实例，但它带来了许多复杂性和问题，例如实例和内存管理、复杂的多线程和并发编程。

在面向对象编程（OOP）成为主流之前，我们习惯于使用过程式语言进行开发。在 C 编程语言中，我们没有对象和类；我们会使用结构体和函数指针。因此，我们现在讨论的是主要依赖于函数的过程式编程，就像过程式语言依赖于过程一样。我们能够在 C 语言中不使用类就开发出非常强大的程序；事实上，大多数操作系统都是用 C 语言开发的。还有其他多用途的编程语言，例如谷歌的 Go 语言，它不是面向对象的，但由于其性能和简单性而变得越来越受欢迎。

那么，我们是否能够在 Swift 中不使用类就编写非常复杂的应用程序呢？我们可能会想知道为什么我们应该这样做。通常，我们不应该这样做，但尝试这样做将使我们了解函数式编程的能力。这就是为什么在讨论其他构建块，如`类`、`结构体`和`枚举`之前，我们将有一个关于函数的整个章节。

函数是一段执行特定任务的代码块，可以存储，可以持久化数据，并且可以被传递。我们将其定义在独立的 Swift 文件中作为全局函数，或者在其他构建块如`类`、`结构体`、`枚举`和`协议`中作为方法。

如果它们在类中定义，它们被称为方法，但在定义上，Swift 中函数和方法之间没有区别。

在其他构建块中定义它们使得方法可以使用父级的范围，或者能够改变它。它们可以访问其父级的范围，并且它们有自己的范围。在函数内部定义的任何变量都不可以在函数外部访问。它们内部定义的变量以及相应的分配的内存将在函数终止时消失。

函数在 Swift 中非常强大。我们可以仅使用函数来组合程序，因为函数可以接收和返回函数，捕获它们声明上下文中的变量，并且可以在内部持久化数据。为了理解函数式编程范式，我们需要详细了解函数的能力。我们需要思考是否可以避免类而只使用函数，因此我们将在本章的后续部分涵盖与函数相关的所有细节。

# 函数和方法的通用语法

我们可以如下定义函数或方法：

```swift
accessControl func functionName(parameter: ParameterType) throws
  -> ReturnType { }
```

如我们所知，当函数在对象中定义时，它们就变成了方法。

定义方法的第一步是告诉编译器它可以从哪里访问。这个概念在 Swift 中被称为访问控制，并且有三种访问控制级别。我们将如下解释方法级别的访问控制：

+   **公共访问**：如果实体在同一模块中，它可以访问被定义为`public`的方法。如果实体不在同一模块中，我们需要导入该模块才能调用该方法。当我们开发框架时，我们需要将我们的方法和对象标记为`public`，以便其他模块可以使用它们。

+   **内部访问**：任何被定义为`internal`的方法可以从模块中的其他实体访问，但不能从其他模块访问。

+   **私有访问**：任何被定义为`private`的方法只能从同一源文件中访问。

默认情况下，如果我们没有提供访问修饰符，变量或函数就变为内部访问。

使用这些访问修饰符，我们可以正确地组织我们的代码；例如，如果我们定义一个实体为内部访问，我们可以从其他模块中隐藏细节。我们甚至可以将方法的细节隐藏在其他文件中，如果我们将它们定义为私有。

在 Swift 2.0 之前，我们必须将所有内容定义为公共的，或者将所有源文件添加到测试目标中。Swift 2.0 引入了`@testable import`语法，使我们能够定义内部或私有方法，这些方法可以从测试模块中访问。

方法通常可以以三种形式存在：

+   **实例方法**：我们需要获取一个对象的实例（在这本书中，我们将`classes`、`structs`和`enums`称为对象）才能调用其中定义的方法，然后我们才能访问该对象的范围和数据。

+   **静态方法**：Swift 也称之为类型方法。它们不需要任何对象实例，并且不能访问实例数据。它们通过在对象类型名称后放置一个点来调用（例如，`Person.sayHi()`）。`static`方法不能被它们所在的对象的子类覆盖。

+   **类方法**：类方法类似于`static`方法，但它们可以被子类覆盖。

我们已经讨论了方法定义所需的关键字；现在我们将集中讨论函数和方法共有的语法。本书不涉及与方法相关的一些概念，因为我们将在 Swift 中专注于函数式编程。

继续讨论函数定义，现在出现的是强制使用的`func`关键字，它用于告诉编译器它将处理一个函数。

然后是函数名，这也是强制性的，建议使用驼峰式命名法，首字母小写。函数名应该说明函数的功能，并且在我们定义对象的方法时，建议使用动词形式。

基本上，我们的类将被命名为名词，方法将被命名为动词，它们是对类的命令。在纯函数式编程中，由于函数不驻留在其他对象中，它们可以按其功能命名。

参数跟在`func`名称之后。它们将在括号内定义，以向函数传递参数。即使我们没有参数，括号也是强制性的。我们将在本章的“定义和使用函数参数”部分中涵盖参数的所有方面。

接下来是`throws`关键字，它不是强制性的。标记了`throws`关键字的函数或方法可能会抛出错误，也可能不会。我们将在后续章节中介绍错误处理机制。目前，当我们看到函数或方法签名中的它们时，了解它们是什么就足够了。

函数类型声明中的下一个实体是返回类型。如果一个函数不是 void 类型，返回类型将位于`->`符号之后。返回类型表明了函数将要返回的实体类型。

我们将在本章的“从函数返回值”部分详细讨论返回类型，因此现在我们可以继续讨论大多数编程语言中普遍存在的函数的最后一部分，那就是我们喜爱的`{ }`。我们将函数定义为功能块，而`{ }`定义了块的边界，以便函数体在这里声明并执行。我们将在`{ }`内编写功能。

## 函数定义的最佳实践

有一些经过验证的最佳实践是由出色的软件工程资源提供的，例如 *Clean Code: A Handbook of Agile Software Craftsmanship*，由 *Robert C. Martin* 编著，*Code Complete: A Practical Handbook of Software Construction, Second Edition*，由 *Steve McConnell* 编著，以及 Coding Horror ([`blog.codinghorror.com/code-smells/`](https://blog.codinghorror.com/code-smells/))，我们可以总结如下：

+   尽量不要让每个函数的代码行数超过 8-10 行，因为较短的函数或方法更容易阅读、理解和维护。

+   尽量保持参数数量最小，因为一个函数拥有的参数越多，它就越复杂。

+   函数至少应该有一个参数和一个返回值。

+   避免在函数名称中使用类型名称，因为这将是多余的。

+   力求函数只实现一个功能。

+   给函数或方法命名时，要确保它能正确描述其功能且易于理解。

+   一致地命名函数和方法。如果我们有一个连接函数，我们可以有一个断开连接的函数。

+   编写函数来解决当前问题，并在需要时进行泛化。尽量避免假设场景，因为可能**你不会用到它**（YAGNI）。

## 调用函数

如果函数定义在对象中，我们已经介绍了一种通用的语法来定义函数和方法。现在，我们来谈谈如何调用我们定义的函数和方法。要调用一个函数，我们将使用其名称并提供其所需的参数。在提供参数时存在一些复杂性，我们将在下一节中介绍。现在，我们将介绍最基本的参数类型，如下所示：

```swift
funcName(firstParam: "some String", secondParam: "some String")

```

这种类型的函数调用应该对 Objective-C 开发者来说很熟悉，因为第一个参数名称未命名，其余的都命名了。

要调用方法，我们需要使用 Swift 提供的点符号。以下示例适用于类实例方法和静态类方法：

```swift
let someClassInstance = SomeClass()
let paramName = "parameter name"
let secondParamName = "second Parameter"
someClassInstance.funcName(firstParam: paramName, secondParam:
  secondParamName)

```

# 定义和使用函数参数

在函数定义中，参数跟在函数名称后面，并且默认情况下是常量，因此如果我们不使用 `var` 标记它们，我们无法在函数体内部修改它们。在函数式编程中，我们避免可变性；因此，我们永远不会在函数中使用可变参数。

参数应放在括号内。如果我们没有任何参数，我们只需简单地放置一对括号，中间不包含任何字符：

```swift
func functionName() { } 

```

在函数式编程中，拥有至少一个参数的函数非常重要。我们将在接下来的章节中解释为什么它很重要。

我们可以有多个参数，用逗号分隔。在 Swift 中，参数是有名的，因此我们需要在冒号后提供参数名称和类型，如下例所示：

```swift
func functionName(firstParameter: ParameterType, secondParameter:
    ParameterType) {
    // function body
}

// To call:
functionName(firstParameter: paramName, secondParameter: secondParamName)

```

`ParameterType` 也可以是一个可选类型，因此如果我们的参数需要是可选的，函数将变为以下形式：

```swift
func functionName(parameter: ParameterType?, secondParameter: 
  ParameterType?) { }
```

Swift 允许我们提供外部参数名称，这些名称将在调用函数时使用。以下示例展示了语法：

```swift
Func functionName(externalParamName localParamName: ParameterType) 
// To call: 
functionName(externalParamName: parameter) 

```

在函数体中只能使用局部参数名称。

使用 `_` 语法可以省略参数名称；例如，如果我们不想在调用函数时提供任何参数名称，我们可以使用 `_` 作为 `externalParamName` 来表示第二个或后续参数。

如果我们想在函数调用中为第一个参数提供参数名称，我们基本上也可以将局部参数名称作为外部参数提供。在这本书中，我们将使用默认的函数参数定义。

参数可以有默认值，如下所示：

```swift
func functionName(parameter: Int = 3) {
    print("\(parameter) is provided.")
}

functionName(parameter: 5) // prints "5 is provided."
functionName() // prints "3 is provided."

```

参数可以被定义为 `inout` 以允许函数调用者获取在函数体中将要被改变的参数。由于我们可以使用元组作为函数的返回值，除非我们确实需要它们，否则不建议使用 `inout` 参数。

我们可以将函数参数定义为元组。例如，以下示例函数接受一个 `(Int, Int)` 类型的元组：

```swift
func functionWithTupleParam(tupleParam: (Int, Int)) {} 

```

在 Swift 中，变量在底层是以元组的形式表示的，因此函数的参数也可以是元组。例如，让我们有一个简单的 `convert` 函数，它接受一个 `Int` 类型的数组和乘数，并将其转换为不同的结构。现在我们不必担心这个函数的实现；我们将在 第六章 中介绍 map 函数：

```swift
let numbers = [3, 5, 9, 10]

func convert(numbers: [Int], multiplier: Int) -> [String] {
    let convertedValues = numbers.enumerated().map { (index, element) in
        return "\(index): \(element * multiplier)"
    }
    return convertedValues
}

```

如果我们将此函数用作 `let resultOfConversion = convert(numbers: numbers, multiplier: 3)`，结果将是 `["0: 9", "1: 15", "2: 27", "3: 30"]`。

我们可以用元组来调用我们的函数。让我们创建一个元组并将其传递给我们的函数：

```swift
let parameters = (numbers: numbers, multiplier: 3)
convert(parameters)

```

结果与之前的函数调用相同。然而，从 Swift 3.0 开始，函数调用中传递元组已被移除，因此不建议使用它们。

我们可以定义高阶函数，这些函数可以接收函数作为参数。在以下示例中，我们将 `funcParam` 定义为 `(Int, Int) -> Int` 类型的函数类型：

```swift
func functionWithFunctionParam(funcParam: (Int, Int)-> Int) 

```

在 Swift 中，参数可以是泛型类型。以下示例展示了一个具有两个泛型参数的函数。在这个语法中，我们可以在 `<>` 内放置任何类型（例如，`T` 或 `V`），这些类型应该用于参数定义：

```swift
func functionWithGenerics<T, V>(firstParam: T, secondParam) 

```

我们将在 第五章 中介绍泛型；到目前为止，了解语法就足够了。

# 定义和使用可变函数

Swift 允许我们定义具有可变参数的函数。可变参数可以接受零个或多个指定类型的值。可变参数类似于数组参数，但它们更易于阅读，并且只能作为多参数函数中的最后一个参数使用。

由于可变参数可以接受零值，我们需要检查它们是否为空。

以下示例展示了一个具有可变参数的`String`类型的函数：

```swift
func greet(names: String...) {
    for name in names {
        print("Greetings, \(name)")
    }
}

// To call this function
greet(names: "Steve", "Craig") // prints twice
greet(names: "Steve", "Craig", "Johny") // prints three times

```

# 从函数中返回值

如果我们需要我们的函数返回一个值、元组或另一个函数，我们可以在`->`之后提供`ReturnType`来指定它。例如，以下示例返回`String`：

```swift
func functionName() -> String { } 

```

任何在其定义中包含`ReturnType`的函数，其体中都应该有一个与匹配类型的`return`关键字。

在 Swift 中，返回类型可以是可选的，因此如果需要返回值是可选的，函数可以如下所示：

```swift
func functionName() -> String? { } 

```

元组可以用来提供多个返回值。例如，以下函数返回一个`(Int, String)`类型的元组：

```swift
func functionName() -> (code: Int, status: String) { } 

```

由于我们使用括号表示元组，因此应避免在单返回值函数中使用括号。

元组返回类型也可以是可选的，因此语法如下：

```swift
func functionName() -> (code: Int, status: String)? { } 

```

这种语法使得整个元组都是可选的；如果我们只想使`status`可选，我们可以将函数定义为如下：

```swift
func functionName() -> (code: Int, status: String?) { } 

```

在 Swift 中，函数可以返回函数。以下示例展示了一个返回类型为函数的函数，该函数接受两个`Int`值并返回一个`Int`值：

```swift
func funcName() -> (Int, Int)-> Int {} 

```

如果我们预期一个函数不返回任何值、元组或函数，我们只需不提供`ReturnType`：

```swift
func functionName() { } 

```

我们也可以使用`Void`关键字显式声明它：

```swift
func functionName() -> Void { } 

```

在函数式编程中，函数中具有返回类型非常重要。换句话说，避免具有`Void`作为返回类型的函数是一种良好的实践。具有`Void`返回类型的函数通常是一个会改变代码中另一个实体的函数；否则，我们为什么需要函数呢？好吧，我们可能想要将表达式记录到控制台/日志文件中，或将数据写入数据库或文件系统中的文件。在这些情况下，最好也有关于操作成功与否的返回或反馈。随着我们试图在函数式编程中避免可变性和有状态编程，我们可以假设我们的函数将以不同的形式返回。

这个要求与函数式编程的数学基础相一致。在数学中，一个简单的函数定义如下：

```swift
y = f(x) or f(x) -> y 

```

在这里，`f`是一个接受`x`并返回`y`的函数。因此，一个函数至少接收一个参数并返回至少一个值。在函数式编程中，遵循相同的范式可以使推理更容易，函数组合成为可能，代码更易于阅读。

# 纯函数

纯函数是没有任何副作用的功能；换句话说，它们不会改变或修改自身之外的数据或状态。此外，它们不访问任何数据或状态，除了提供的参数。纯函数就像数学函数，它们本质上是纯的。

纯函数返回的值仅由其参数值决定。纯函数易于测试，因为它们只依赖于它们的参数，并且不改变或访问任何自身之外的数据或状态。纯函数适合并发，因为它们不访问和改变全局数据或状态。

以下列表展示了纯函数和非纯函数的示例：

+   将字符串字面量打印到控制台不是纯函数，因为它修改了外部状态。

+   读取文件不是纯函数，因为它依赖于不同时间的外部状态。

+   字符串的长度是纯函数，因为它不依赖于状态。它只接受一个字符串作为输入，并返回长度作为输出。

+   获取当前日期不是纯函数，因为它在不同的日期调用时返回不同的值。

+   获取随机数不是纯函数，因为它每次调用时都返回不同的值。

使用纯函数可能听起来非常限制性，在现实场景中似乎无法利用，但本书后面将讨论其他可以提供相同功能的工具。

我们将在下一章更详细地了解纯函数的好处。

# 函数类型

函数参数类型及其返回类型定义了函数的类型。例如，以下代码示例的函数类型是`(Int, Double) -> String`：

```swift
func functionName(firstParam: Int, secondParam: Double) -> String 

```

我们将能够以使用其他类型的方式使用函数类型。以下代码示例展示了函数类型：

```swift
var simpleMathOperator: (Double, Double) -> Double 

```

在这里，`simpleMathOperator`是一个`(Double, Double) -> Double`类型的函数变量。

我们可以如下定义函数类型的`typealias`：

```swift
typealias SimpleOperator = (Double, Double) -> Double 

```

我们可以在`simpleMathOperator`定义中使用此`typealias`如下：

```swift
var simpleMathOperator: SimpleOperator 

```

我们可以定义具有相同类型的函数并将它们分配给我们的`simpleMathOperator`。以下代码片段中的函数类型是`(Double, Double) -> Double`，这实际上是`SimpleOperator`：

```swift
func addTwoNumbers(a: Double, b: Double) -> Double { return a + b } 

func subtractTwoNumbers(a: Double, b: Double) -> Double { return a - b } 

func divideTwoNumbers(a: Double, b: Double) -> Double { return a / b } 

func multiplyTwoNumbers(a: Double, b: Double) -> Double { return a * b } 

```

因此，我们可以将这些函数分配给`simpleMathOperator`，如下所示：

```swift
simpleMathOperator = multiplyTwoNumbers 

```

这意味着`simpleMathOperator`指向`multiplyTwoNumbers`函数：

```swift
let result = simpleMathOperator(3.0, 4.0) // result is 12

```

由于其他三个函数也有相同的函数类型，我们将能够将它们分配给相同的变量：

```swift
simpleMathOperator = addTwoNumbers 
let result = simpleMathOperator(3.5, 5.5) // result is 9 

```

我们可以将`SimpleOperator`用作其他函数的参数类型：

```swift
func calculateResult(mathOperator: SimpleOperator, a: Double, b: Double)
  -> Double {
    return mathOperator(a, b)
}

print("The result is \(calculateResult(mathOperator: simpleMathOperator,
  a: 3.5, b: 5.5))") // prints "The result is 9.0"
```

在这里，`calculateResult`函数有三个参数。`mathOperator`参数是函数类型。`a`和`b`参数是`Double`类型。当我们调用这个函数时，我们传递一个`simpleMathOperator`函数和两个`Double`值作为`a`和`b`。

重要的是要知道我们只传递了`simpleMathOperator`的引用，这不会执行它。在函数体中，我们使用这个函数，并用`a`和`b`调用它。

我们可以将`SimpleOperator`用作函数的返回类型：

```swift
func choosePlusMinus(isPlus: Bool) -> SimpleOperator {
    return isPlus ? addTwoNumbers : subtractTwoNumbers
}

let chosenOperator = choosePlusMinus(isPlus: true)
print("The result is \(chosenOperator(3.5, 5.5))") // prints "The result
  is 9.0"
```

在这里，`choosePlusMinus`函数有一个`Bool`参数；在其主体中，它检查此参数并返回具有相同类型`SimpleOperator`的`addTwoNumbers`或`subtractTwoNumbers`。

重要的是要理解，调用`choosePlusMinus(true)`不会执行返回的函数，实际上只是返回了`addTwoNumbers`的引用。我们将这个引用保存在`chosenOperator`中。`chosenOperator`变量变成了以下：

```swift
func addTwoNumbers(a: Double, b: Double) -> Double { return a + b } 

```

当我们调用`chosenOperator(3.5, 5.5)`时，我们将这两个数字传递给`addTwoNumbers`函数并执行它。

定义函数类型的能力使得函数在 Swift 中成为一等公民。函数类型用于一等和更高阶函数。这些能力使我们能够在 Swift 中应用函数式编程范式。

# 定义和使用嵌套函数

在 Swift 中，我们可以在其他函数内部定义函数。换句话说，我们可以在其他函数内部嵌套函数。嵌套函数只能在它们的封装函数内部访问，并且默认情况下对外界隐藏。封装函数可以返回嵌套函数，以便允许嵌套函数在其他作用域中使用。以下示例展示了一个包含两个嵌套函数的函数，并根据其`isPlus`参数的值返回其中一个：

```swift
func choosePlusMinus(isPlus: Bool) -> (Int, Int) -> Int {
    func plus(a: Int, b: Int) -> Int {
        return a + b
    }
    func minus(a: Int, b: Int) -> Int {
        return a - b
    }
    return isPlus ? plus : minus
}

```

# 一等函数

在本章的“函数类型”部分，我们看到了我们可以定义函数类型并存储和传递函数。在实践中，这意味着 Swift 将函数视为值。为了解释这一点，我们需要检查几个示例：

```swift
let name: String = "Your name"

```

在这个代码示例中，我们创建了一个`String`类型的常量`name`，并将其中的值（`"Your name"”）存储在其中。

当我们定义一个函数时，我们需要指定参数的类型：

```swift
func sayHello(name: String)

```

在这个例子中，我们的`name`参数是`String`类型。这个参数可以是任何其他值类型或引用类型。简单来说，它可以是`Int`、`Double`、`Dictionary`、`Array`、`Set`，或者它可以是`class`、`struct`或`enum`等对象类型。

现在，让我们调用这个函数：

```swift
sayHello(name: "Your name") // or
sayHello(name: name)

```

这里，我们为这个参数传递了一个值。换句话说，我们传递了之前提到的类型及其相应的值。

Swift 将函数视为上述其他类型，因此我们可以像其他类型一样将函数存储在变量中：

```swift
var sayHelloFunc = sayHello

```

在这个例子中，我们将`sayHello`函数保存在一个变量中，稍后可以使用，并将其作为值传递。

在纯面向对象编程（OOP）中，我们没有函数；相反，我们有方法。换句话说，函数只能存在于对象中，然后它们被称为方法。在 OOP 中，类是一等公民，而方法不是。方法不是唯一可访问的，也不能被存储或传递。在 OOP 中，方法访问它们定义在其中的对象的数据。

在函数式编程中，函数是一等公民。就像其他类型一样，它们可以被存储和传递。与 OOP 不同，那种方法只能访问其父对象的数据并更改它；在函数式编程中，它们可以被存储并传递给其他对象。

这个概念使我们能够以函数作为另一种类型来组合我们的应用程序。我们将更详细地讨论这个问题；现在，重要的是要理解为什么我们在 Swift 中将函数称为一等公民。

# 高阶函数

正如我们在本章的*定义和使用函数参数*和*函数类型*部分所看到的，在 Swift 中，函数可以接受函数作为参数。可以接受其他函数作为参数的函数称为高阶函数。这个概念与一等函数一起赋予了函数式编程和函数分解能力。

由于这个主题在函数式编程中至关重要，我们将通过另一个简单的例子来探讨。

假设我们需要开发两个函数，将两个`Int`值相加和相减，如下所示：

```swift
func subtractTwoValues(a: Int, b: Int) -> Int {
    return a - b
}

func addTwoValues(a: Int, b: Int) -> Int {
    return a + b
}

```

此外，我们还需要开发函数来计算两个`Int`值的平方和立方，如下所示：

```swift
func square(a: Int) -> Int {
    return a * a
}

func triple(a: Int) -> Int {
    return a * a * a // or return squareAValue(a) * a
}

```

假设我们需要另一个函数，该函数从两个平方值中减去：

```swift
func subtractTwoSquaredValues(a: Int, b: Int) -> Int {
    return (a * a) - (b * b)
}

```

假设我们需要添加两个平方值：

```swift
func addTwoSquaredValues(a: Int, b: Int) -> Int {
    return (a * a) + (b * b)
}

```

假设我们需要另一个函数，该函数将一个值乘以三并加到另一个乘以三的值上：

```swift
func multiplyTwoTripledValues(a: Int, b: Int) -> Int {
    return (a * a * a) * (b * b * b)
}

```

这样，我们不得不编写很多冗余和不灵活的函数。使用高阶函数，我们可以编写一个灵活的函数，如下所示：

```swift
typealias AddSubtractOperator = (Int, Int) -> Int
typealias SquareTripleOperator = (Int) -> Int
func calcualte(a: Int, b: Int, funcA: AddSubtractOperator, funcB:
  SquareTripleOperator) -> Int {
    return funcA(funcB(a), funcB(b))
}

```

这个高阶函数接受两个其他函数作为参数并使用它们。我们可以根据不同的场景调用它，如下所示：

```swift
print("The result of adding two squared values is: \(calcualte(a: 2, b: 2,
  funcA: addTwoValues, funcB: square))") // prints "The result of adding
  two squared value is: 8"

print("The result of subtracting two tripled value is: \(calcualte(a: 3,
  b: 2, funcA: subtractTwoValues, funcB: triple))") // prints "The result
  of adding two tripled value is: 19"
```

这个简单的例子展示了高阶函数在函数组合和程序模块化中的实用性。

# 函数组合

在上一节中，我们看到了一个可以接受两个不同函数并按预定义顺序执行它们的高阶函数的例子。这个函数在灵活性方面并不强，如果我们想以不同的方式组合这两个接受的函数，它就会崩溃。函数组合可以解决这个问题，并使其更加灵活。为了展示这个概念，我们首先将检查一个非函数组合的例子，然后我们将介绍函数组合。

假设在我们的应用中，我们需要与一个后端 RESTful API 进行交互，并接收一个包含按顺序排列的价格列表的`String`值。这个后端 RESTful API 是由第三方开发的，并且设计得并不完善。不幸的是，它返回的`String`中包含用逗号分隔的数字：

```swift
"10,20,40,30,80,60" 

```

在使用之前，我们需要格式化我们接收到的内容。我们将从`String`中提取元素并创建一个数组，然后我们将为每个项目添加`$`作为货币符号，以便在表格视图中使用。以下代码示例展示了解决这个问题的一种方法：

```swift
let content = "10,20,40,30,80,60"

func extractElements(_ content: String) -> [String] {
    return content.characters.split(separator: ",").map { String($0) }
}

let elements = extractElements(content)

func formatWithCurrency(content: [String]) -> [String] {
    return content.map {"\($0)$"}
}

let formattedElements = formatWithCurrency(content: elements)

```

在这个代码示例中，我们单独处理每个函数。我们可以将第一个函数的结果作为第二个函数的输入参数。两种方法都很冗长，并且不具有函数式风格。此外，我们使用了`map`函数，这是一个高阶函数，但我们的方法仍然不是函数式的。

让我们以函数式的方式解决这个问题。

第一步将是识别每个函数的函数类型：

+   `extractElements`: `String -> [String]`

+   `formatWithCurrency`: `[String] -> [String]`

如果我们将这些函数连接起来，我们将得到以下结果：

```swift
extractElements: String -> [String] | formatWithCurrency: [String] 
  -> [String]
```

我们可以将这些函数与函数组合结合，组合后的函数将是`String -> [String]`类型。以下示例显示了组合过程：

```swift
let composedFunction = { data in
    formatWithCurrency(content: extractElements(data))
}

composedFunction(content)

```

在这个例子中，我们定义了`composedFunction`，它由两个其他函数组成。我们能够以这种方式组合函数，因为每个函数至少有一个参数和一个返回值。这种组合类似于函数的数学组合。假设我们有一个函数`f(x)`返回`y`，以及一个`g(y)`函数返回`z`。我们可以将`g`函数组合为`g(f(x)) -> z`。这种组合使我们的`g`函数接受`x`作为参数并返回`z`作为结果。这正是我们在`composedFunction`中所做的。

## 自定义操作符

虽然`composedFunction`比非函数版本更简洁，但它看起来并不美观。而且，它也不容易阅读，因为我们需要从内向外阅读它。让我们使这个函数更简单、更易读。一个解决方案是定义一个自定义操作符，用它来代替我们的组合函数。在接下来的章节中，我们将探讨可以用来定义自定义操作符的标准操作符。我们还将探索自定义操作符定义技术。学习这个概念很重要，因为我们将在这本书的其余部分使用它。

### 允许的操作符

Swift 标准库提供了一些可以用来定义自定义操作符的操作符。自定义操作符可以以一个 ASCII 字符——`/`, `=`, `-`, `+`, `!`, `*`, `%`, `<`, `>`, `&`, `|`, `^`, `?`, 或 `~` 或一个 Unicode 字符开头。在第一个字符之后，允许组合 Unicode 字符。

我们还可以定义以点开头的自定义操作符。如果一个操作符不以点开头，它不能在其他地方包含点。尽管我们可以定义包含问号`?`的自定义操作符，但它们不能仅由一个问号字符组成。此外，尽管操作符可以包含感叹号`!`，后缀操作符不能以问号或感叹号开头。

### 自定义操作符定义

我们可以使用以下语法定义自定义操作符：

```swift
operatorType operator operatorName { } 

```

在这里，`operatorType`可以是以下之一：

+   前缀

+   中缀

+   后缀

自定义中缀操作符也可以指定优先级和结合性：

```swift
infix operator operatorName { associativity left/right/none 
  precedence}
```

关联性的可能值是 `left`、`right` 和 `none`。左结合性运算符如果与相同优先级的其他左结合性运算符相邻，则与左侧结合。同样，右结合性运算符如果与相同优先级的其他右结合性运算符相邻，则与右侧结合。非结合性运算符不能与具有相同优先级的其他运算符相邻。

如果未指定，关联性值默认为 `none`。如果未指定，优先级值默认为 `100`。

使用前面的语法定义的任何自定义运算符在 Swift 中都不会有现有的意义；因此，应该定义并实现一个名为 `operatorName` 的函数。在下一节中，我们将检查自定义运算符定义及其相应函数定义的示例。

## 带有自定义运算符的组合函数

让我们定义一个新的自定义运算符来代替我们的组合函数：

```swift
infix operator |> { associativity left }
func |> <T, V>(f: T -> V, g: V -> V ) -> T -> V {
    return { x in g(f(x)) }
}

let composedWithCustomOperator = extractElements |> formatWithCurrency
composedWithCustomOperator("10,20,40,30,80,60")

```

在这个例子中，我们定义了一个新的运算符 `|>`，它接受两个泛型函数并将它们组合起来，返回一个函数，该函数的第一个函数的输入作为参数，第二个函数的返回值作为返回类型。

由于这个新运算符将要组合两个函数并且是二元的，我们将其定义为中缀运算符。然后我们需要使用运算符关键字。下一步将是选择我们新自定义运算符的表示法。由于我们将函数组合到左侧，我们需要将其指定为 **左结合性**。

为了能够使用此运算符，我们需要定义相应的函数。我们的函数接受两个函数，如下所示：

+   `f`: 这个函数接受一个泛型类型 `T` 并返回一个泛型类型 `V`

+   `g`: 这个函数接受一个泛型类型 `V` 并返回一个泛型类型 `V`

在我们的例子中，我们有以下函数：

+   `extractElements`: `String -> [String]`

+   `formatWithCurrency`: `[String] -> [String]`

因此，`T` 变为 `String`，而 `V` 变为 `[String]`。

我们的 `|>` 函数返回一个函数，该函数接受一个泛型类型 `T` 并返回一个泛型类型 `V`。我们需要从组合函数接收 `String -> [String]`，因此，`T` 再次变为 `String`，而 `V` 变为 `[String]`。

使用我们的 `|>` 自定义运算符可以使我们的代码更易读且更简洁。

# 闭包

闭包是没有 `func` 关键字的函数。闭包是包含特定功能的自包含代码块，可以被存储、传递并在代码中像函数一样使用。闭包捕获它们定义的上下文中的常量和变量。尽管闭包在 Objective-C 中等同于代码块，但与 C 和 Objective-C 的代码块语法相比，Swift 中的闭包语法更简单。我们在前面的部分中已经介绍过的嵌套函数是闭包的特殊情况。闭包是引用类型，可以作为变量、常量和类型别名存储。它们可以被传递给函数并从函数返回。

## 闭包语法

闭包的一般语法如下：

```swift
{ (parameters) -> ReturnType in
    // body of closure
}

```

闭包定义以`{`开始，然后我们定义闭包类型，最后使用`in`关键字将闭包定义与其实现分开。

在`in`关键字之后，我们编写闭包体，并通过关闭`}`来完成我们的闭包。

闭包可以用来定义变量。以下闭包定义了一个接受`Int`并返回`Int`的类型闭包变量：

```swift
let closureName: (Int) -> (Int) = {/* */ }

```

闭包可以存储为可选变量。以下闭包定义了一个接受`Int`并返回`Optional Int`的类型闭包变量：

```swift
var closureName: (Int) -> (Int)?

```

闭包可以被定义为`类型别名`。以下示例展示了具有两个`Int`参数并返回`Int`的闭包的`typealias`：

```swift
typealias closureType = (Int, Int) -> (Int)

```

同样的`typealias`也可以用于函数类型定义，因为在 Swift 中函数被命名为命名的闭包。

闭包可以用作函数调用的参数。例如，以下示例展示了一个使用闭包作为参数的函数调用，该闭包接收`Int`并返回`Int`：

```swift
func aFunc(closure: (Int) -> Int) -> Int {
    // Statements, for example:
    return closure(5)
}

let result = aFunc(closure: { number in
    // Statements, for example:
    return number * 3
})

print(result)

```

闭包可以用作函数参数。以下示例展示了一个接收闭包的数组排序方法：

```swift
var anArray = [1, 2, 5, 3, 6, 4]

anArray.sort(isOrderedBefore: { (param1: Int, param2: Int) -> Bool in
    return param1 < param2
})

```

此语法可以通过隐式类型简化，因为 Swift 编译器能够从上下文中推断参数的类型：

```swift
anArray.sort(isOrderedBefore: { (param1, param2) -> Bool in
    return param1 < param2
})

```

使用 Swift 的类型推断，可以通过隐式返回类型进一步简化语法：

```swift
anArray.sort(isOrderedBefore: { (param1, param2) in
    return param1 < param2
})

```

当我们需要将闭包作为函数的最后一个参数传递时，Swift 允许我们省略开闭括号，换句话说，如果我们的闭包是尾随闭包：

```swift
anArray.sort { (param1, param2) in
    return param1 < param2
}

```

此外，Swift 还提供了一个简写参数符号，可以用作替代使用参数：

```swift
anArray.sort {
    return $0 < $1
}

```

我们可以通过消除`return`关键字进一步简化此语法，因为我们只有一行表达式如下：

```swift
anArray.sort {  0 < $1 }

```

使用 Swift 的类型推断，我们能够极大地简化闭包的语法。

## 捕获值

闭包可以捕获它们创建的周围上下文中的变量和常量。闭包可以在其体内部引用这些变量并修改它们，即使定义变量的原始作用域不再存在。

当闭包作为函数的参数传递但函数返回后调用时，称闭包逃逸了函数。闭包逃逸的一种方式是将其存储在函数外部定义的变量中。

以下是一个逃逸闭包的示例，换句话说，是完成处理程序：

```swift
func sendRequest(responseType: String.Type, completion:
  (responseData:String, error:NSError?) -> Void) {
    // execute some time consuming operation, if successful {
        completion(responseData: "Response", error: nil)
    //}
}

sendRequest(String.self) {
    (response: String?, error: NSError?) in
    if let result = response {
        print(result)
    } else if let serverError = error {
        // Error
    }
}

```

我们有一个名为`sendRequest`的函数，它有两个参数——`responseType`为`String.Type`类型，以及`completion`，它是一个闭包类型，接受一个`String`和一个可选的`NSError`参数，并且不返回任何值。

假设我们在函数体中执行一些异步耗时操作，例如从文件中读取、从数据库中读取或调用网络服务。

要调用此函数，我们需要提供`String.self`和一个闭包作为参数。我们的闭包中有两个变量——一个名为`response`的`Optional String`类型的变量和一个名为`error`的`NSError`可选类型的变量。由于我们的函数没有返回类型，它不会向其调用者返回任何值。这就是函数逃逸的概念。

我们传递的闭包在函数外部逃逸，因为它将在耗时的异步操作成功完成后被调用，并且随后发生调用：

```swift
completion(responseData: "Response", error: nil)

```

在这个调用中，我们传递`responseData`和错误，并调用完成闭包。然后调用函数的闭包体使用传递的变量执行。这个概念是一个非常强大的概念，它简化了所有的异步操作。与委托和通知等机制相比，它非常易于阅读和跟踪。

# 函数柯里化

函数柯里化将具有多个参数的单个函数转换为一系列具有一个参数的函数。让我们看看一个例子。假设我们有一个将`firstName`和`lastName`组合起来返回全名的函数：

```swift
func extractFullUserName(firstName: String, lastName: String) -> String {
    return "\(firstName) \(lastName)"
}

```

此函数可以转换为以下柯里化函数：

```swift
func curriedExtractFullUserName(firstName: String)(lastName:
  String) -> String {
    return "\(firstName) \(lastName)"
}

```

如此，我们可以看到我们将逗号替换为`) (`括号。

因此现在我们可以使用此函数如下：

```swift
let fnIncludingFirstName = curriedExtractFullUserName("John")
let extractedFullName = fnIncludingFirstName(lastName: "Doe")

```

在这里，`fnIncludingFirstName`将包含`firstName`，这样当我们使用它时，我们可以提供`lastName`并提取全名。我们将在接下来的章节中使用这种技术。

从 Swift 2.2 开始，Apple 已经弃用了函数柯里化，并将其从 Swift 3.0 中移除。建议将函数柯里化转换为显式返回闭包：

```swift
// Before:
func curried(x: Int)(y: String) -> Float {
    return Float(x) + Float(y)!
}

// Swift 3.0 syntax:
func curried(x: Int) -> (String) -> Float {
    return {(y: String) -> Float in
        return Float(x) + Float(y)!
    }
}

```

让我们将我们的柯里化函数显式地转换为返回闭包版本：

```swift
func explicityRetunClosure(firstName: String) -> (String) -> String {
    return { (lastName: String) -> String in
        return "\(firstName) \(lastName)"
    }
}

```

我们可以使用此函数如下，结果将是相同的：

```swift
let fnIncludingFirstName = explicityRetunClosure(firstName: "John")
let extractedFullName = fnIncludingFirstName("Doe")
```

# 递归

递归是函数在其内部调用自己的过程。调用自己的函数是递归函数。

递归最适合用于可以将大问题分解为重复子问题的问题。由于递归函数会调用自己来解决这些子问题，最终函数将遇到一个它可以不调用自己就能处理的子问题。这被称为基本情况，这是防止函数不断调用自己而停止所需的。

在基本情况中，函数不会调用自己。然而，当一个函数必须调用自己以处理其子问题时，这被称为递归情况。因此，在使用递归算法时，有两种类型的情况：基本情况递归情况。重要的是要记住，在递归和尝试解决问题时，我们应该问自己：*我的基本情况是什么，我的递归情况是什么？*

要应用这个简单的过程，让我们从一个递归的例子开始：阶乘函数。在数学中，一个数字后面的感叹号（*n!*）表示该数字的阶乘。一个数字 `n` 的阶乘是介于 `1` 和 `n` 之间所有整数的乘积。所以，如果 `n` 等于 `3`，那么 `n` 的阶乘将是 *3 * 2 * 1*，等于 `6`。我们也可以说 `3` 的阶乘等于 `3` 乘以 `2` 的阶乘，即 *3 * 2!* 或 *3 * 2 * 1*。所以，任何数字 `n` 的阶乘也可以定义为以下形式：

```swift
n! = n * (n - 1)!

```

我们还需要了解以下内容：

```swift
0! = 1! = 1

```

注意我们如何定义一个数字的阶乘为该数字乘以比该数字小 `1` 的整数的阶乘 *(n * (n - 1)!)*。所以，我们实际上是将问题分解为一个子问题，为了找到某个数字的阶乘，我们不断寻找比该数字小的整数的阶乘并将其相乘。所以，`3` 的阶乘等于 `3` 乘以 `2` 的阶乘，而 `2` 的阶乘等于 `2` 乘以 `1` 的阶乘。所以，如果我们有一个函数来找到给定数字的阶乘，那么我们的递归情况代码将类似于以下这样：

```swift
func factorial(n: Int) -> Int {
    return n * factorial(n: n - 1)
}

```

在这里，我们想要找到 `n` 个数的阶乘。

在这个例子中，我们将问题分解为一个子问题。仍然有一个问题需要我们解决。我们需要检查基本情况，以便能够停止函数无限调用自身。

因此，我们可以修改我们的阶乘示例如下：

```swift
func factorial(n: Int) -> Int {
    return n == 0 || n == 1 ? 1 : n * factorial(n: n - 1)
}

print(factorial(n: 3))

```

如此例所示，我们检查 `n`；如果它是 `0` 或 `1`，则返回 `1` 并停止递归。

另一个简单递归函数的例子如下：

```swift
func powerOfTwo(n: Int) -> Int {
    return n == 0 ? 1 : 2 * powerOfTwo(n: n - 1)
}

let fnResult = powerOfTwo(n: 3)

```

这个示例的非递归版本如下：

```swift
func power2(n: Int) -> Int {
    var y = 1
    for _ in 0...n - 1 {
        y *= 2
    }
    return y
}

let result = power2(n: 4)

```

从这个例子中我们可以看出，递归版本更具有表达性和更简洁。

以下示例展示了一个重复给定字符串所需时间的函数：

```swift
func repateString(str: String, n: Int) -> String {
    return n == 0 ? "" : str + repateString(str: str , n: n - 1)
}

print(repateString(str: "Hello", n: 4))

```

以下代码片段展示了不使用递归实现相同功能的方式，换句话说，以命令式编程风格：

```swift
func repeatString(str: String, n: Int) -> String {
    var ourString = ""
    for _ in 1...n {
        ourString += str
    }
    return ourString
}

print(repeatString(str: "Hello", n: 4))

```

非递归、命令式版本的代码稍微长一些，我们需要使用 `for` 循环和变量才能达到相同的结果。一些函数式编程语言，如 Haskell，没有 `for` 循环机制，我们必须使用递归；在 Swift 中，我们有 `for` 循环，但正如我们在这里看到的，尽可能使用递归函数会更好。

## 尾递归

尾递归是递归的一种特殊情况，在这种情况中，调用函数在对自己进行递归调用后不再执行任何操作。换句话说，如果一个函数的最终表达式是一个递归调用，那么这个函数就被称为尾递归函数。我们之前介绍过的递归示例都不是尾递归函数。

为了理解尾递归，我们将使用尾递归技术开发我们之前开发的 `factorial` 函数。然后我们将讨论它们之间的区别：

```swift
func factorial(n: Int, currentFactorial: Int = 1) -> Int {
    return n == 0 ? currentFactorial : factorial(n: n - 1,
      currentFactorial: currentFactorial * n)
}

print(factorial(n: 3))

```

注意，我们为 `currentFactorial` 提供了一个默认参数 `1`，但这只适用于函数的第一次调用。当阶乘函数递归调用时，默认参数会被递归调用传递的任何值覆盖。我们需要有那个第二个参数，因为它将保存我们打算传递给函数的当前阶乘值。

让我们尝试理解它是如何工作的，以及它与另一个阶乘函数的不同之处：

```swift
factorial(n: 3, currentFactorial: 1)
return factorial(n: 2, currentFactorial: 1 * 3) // n = 3
return factorial(n: 1, currentFactorial: 3 * 2) // n = 2
return 6 // n = 1

```

在这个函数中，每次调用阶乘函数时，都会将一个新的 `currentFactorial` 值传递给函数。函数基本上通过每次对自己的调用来更新 `currentFactorial`。由于它接受 `currentFactorial` 作为参数，我们能够保存当前的阶乘值。

所有对阶乘的递归调用，如 `factorial(2, 1 * 3)`，实际上并不需要返回才能得到最终值。我们可以看到，在任何一个递归调用实际返回之前，我们实际上已经到达了 `6` 这个值。

因此，如果一个函数的递归调用的最终结果——在这个例子中是 `6`——也是函数本身的最终结果，那么这个函数就是尾递归的。非尾递归函数在其最后一个函数调用中并没有达到最终状态，因为所有导致最后一个函数调用的递归调用都必须返回，才能得到最终结果。

# 记忆化

记忆化是将函数的输入结果存储起来的过程，以便提高我们程序的性能。我们可以记忆化纯函数，因为纯函数不依赖于外部数据，也不改变自身之外的内容。纯函数对于给定的输入每次都提供相同的结果。因此，我们可以保存或缓存结果——换句话说，记忆化结果——并使用它们在将来而不必经过计算过程。

为了理解这个概念，让我们看看以下示例，我们将手动记忆化 `power2` 函数：

```swift
var memo = Dictionary<Int, Int>()

func memoizedPower2(n: Int) -> Int {
    if let memoizedResult = memo[n] {
        return memoizedResult
    }
    var y = 1
    for _ in 0...n-1 {
        y *= 2
    }
    memo[n] = y
    return y
}
print(memoizedPower2(n: 2))
print(memoizedPower2(n: 3))
print(memoizedPower2(n: 4))
print(memo) // result: [2: 4, 3: 8, 4: 16]

```

如示例所示，我们定义了一个 `[Int, Int]` 类型的字典。我们将函数的输入结果保存到这个字典中。

这种方法可以正常工作，但我们需要手动修改和维护函数外部的一个集合，以便能够记忆化函数的结果。此外，它还为每个需要记忆化的函数添加了大量的模板代码。

在 2014 年的**全球开发者大会**（**WWDC**）上展示的**高级 Swift**会议（[`developer.apple.com/videos/play/wwdc2014-404/`](https://developer.apple.com/videos/play/wwdc2014-404/））提供了一个非常方便的记忆化函数，可以与任何纯函数一起使用。

观看视频是非常推荐的。让我们看看我们能否使用该会话中的 `memoize` 函数来自动化这个功能并重用它：

```swift
func memoize<T: Hashable, U>(fn: ((T) -> U, T) -> U) -> (T) -> U {
    var memo = Dictionary<T, U>()
    var result: ((T) -> U)!
    result = { x in
        if let q = memo[x] { return q }
        let r = fn(result, x)
        memo[x] = r
        return r
    }
    return result
}

```

这个函数看起来很复杂，但不用担心，我们将详细讲解它。

首先，它是一个泛型函数。不要担心泛型——我们将在第五章泛型和关联类型协议中详细讲解泛型，并且使用 `Hashable` 是因为我们需要将 `T` 作为键存储在字典中。

如果我们查看函数的签名，我们会看到 `memoize` 函数接受一个有两个参数和返回类型的函数。因此，`fn` 的签名，它是一个函数，如下所示：

```swift
((T) -> U, T) -> U

```

`fn` 的第一个参数是 `(T) -> U` 类型的函数，第二个参数是 `T` 类型，最后 `fn` 返回 `U` 类型。

好的，`memoize` 函数接收了前面代码片段中描述的 `fn`。

最后，`memoize` 函数返回一个 `(T) -> U` 类型的函数。现在让我们看看 `memoize` 函数的主体。首先，我们需要一个字典来缓存结果。其次，我们需要定义结果类型，它是一个闭包。在闭包体中，我们检查是否已经在我们的字典中有了这个键。如果有，我们返回它，否则调用函数并将结果保存在我们的缓存字典中。

现在我们可以使用这个函数来缓存不同函数调用的结果，并提高我们程序的性能。

以下示例展示了阶乘函数的缓存版本：

```swift
let factorial = memoize { factorial, x in
    x == 0 ? 1 : x * factorial(x - 1)
}

print(factorial(5))

```

`memoize` 函数期望一个闭包作为输入，因此我们可以使用尾随闭包语法。在先前的例子中，我们将阶乘函数和 `x` 参数作为输入传递给闭包，`in` 关键字之后的行是闭包的主体。在先前的例子中，我们使用 `memoize` 对递归函数进行了缓存，并且它工作得很好。让我们看看另一个例子：

```swift
let powerOf2 = memoize { pow2, x in
    x == 0 ? 1 : 2 * pow2(x - 1)
}

print(powerOf2(5))

```

在这个例子中，我们使用 `memoize` 函数来获取 `powerOf2` 函数的缓存版本。

只需编写一次 `memoize` 函数，我们就能将其用于任何纯函数来缓存数据并提高我们程序的性能。

# 摘要

本章从详细解释函数定义和用法开始，通过给出参数和返回类型的示例。然后，它继续介绍与函数式编程相关的概念，如纯函数、一等函数、高阶函数和嵌套函数。最后，它涵盖了函数组合、闭包、柯里化和缓存。到这一点，我们应该熟悉不同类型的函数和闭包及其用法。

在下一章中，我们将介绍类型，并探讨值类型与引用类型的概念。同时，我们将详细探讨值类型的特性，包括类型相等性、身份和转换。
