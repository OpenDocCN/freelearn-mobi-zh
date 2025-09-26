# 第一章：Swift 中函数式编程入门

在本章中，我们将介绍函数式编程范式，例如不可变性、无状态编程、纯函数、一等函数和高阶函数。本章将介绍 Swift 编程语言和 Swift 中的函数式编程范式。以下主题将涵盖，并附带示例：

+   为什么函数式编程很重要？

+   什么是函数式编程？

+   Swift 语言基础

+   不可变性

+   一等、高阶和纯函数

+   可选值和模式匹配

+   闭包

+   类型别名

# 为什么函数式编程很重要？

软件解决方案正变得越来越复杂，为了未来的维护和扩展，有必要对它们进行很好的结构化。软件工程师试图将软件模块化成更小的部分，并在不同的部分和层中抽象出复杂性。将代码分成更小的部分使得可以单独处理每个问题。这种方法提高了协作性，因为不同的工程师可以负责不同的部分。此外，他们可以在不关心其他部分的情况下专注于软件的特定部分。

将软件划分为更小的部分并不是大多数项目和编程语言中的最大挑战。例如，在**面向对象编程**（**OOP**）中，软件被划分为更小的部分，如包、类、接口和方法。工程师倾向于通过领域、逻辑和层来划分软件为这些构建块。类是创建实例和对象的配方。正如其名所示，面向对象编程中最重要的构建块是对象。工程师处理对象，它们的作用和责任应该是清晰和可理解的。

在面向对象编程（OOP）中，将构建块连接起来并不像将它们分开那样容易。不同对象之间的连接可能会导致它们之间产生强烈的耦合。耦合是面向对象编程中复杂性的最大来源。模块或类中的任何变化都可能迫使所有耦合的模块和类发生变化。此外，由于耦合的模块或类，特定的模块或类可能更难重用和测试。

软件工程师试图通过合理地构建软件结构和应用不同的原则和设计模式来放松耦合。例如，当正确应用时，**单一职责、开闭原则、里氏替换原则、接口隔离原则和依赖倒置原则**（**SOLID**）通常会使软件易于维护和扩展。

尽管有可能减少耦合并简化软件结构，但管理内存、引用实例和测试不同对象仍然很困难，因为在面向对象编程（OOP）中，对象是开放的，容易发生变化和突变。

在函数式编程中，纯函数是最重要的构建块。纯函数不依赖于自身之外的数据，并且它们不会改变自身之外存在的数据。纯函数易于测试，因为它们总是会提供相同的结果。

纯函数可以在不同的线程或核心上执行，而不需要任何处理多线程和多进程的机制。这是函数式编程相对于 OOP 的一个重要优势，因为在 OOP 中处理多核编程机制非常复杂。此外，为多核计算机编程正变得越来越重要，因为硬件工程师最终达到了光速的限制。计算机时钟在不久的将来不会更快，因此，为了每秒有更多的周期，硬件工程师正在将更多的处理器添加到芯片上。我们似乎没有尽头，不知道我们的计算机中会有多少处理器。一个程序要使用的处理器数量越多，处理多线程和多核机制的复杂性就越大。

函数式编程消除了对复杂的多核编程机制的需求，并且由于纯函数不依赖于任何实例或自身之外的数据，因此很容易在不改变其他部分的情况下更改它们。

# 什么是函数式编程？

我们知道函数式编程很重要，但它到底是什么？与函数式编程相关的炒作很多，关于它的定义也有很多，但简单来说，它是一种将计算建模为表达式的评估的编程风格。函数式编程是一种声明式编程风格，与被归类为命令式编程的面向对象编程（OOP）相对。

从理论上讲，函数式编程采用了范畴论的概念，范畴论是数学的一个分支。了解范畴论对于能够以函数式编程并不必要，但学习它将帮助我们掌握一些更高级的概念，例如*函子*、*应用函子*和*单子*。我们将在后面的章节中深入探讨范畴论及其与函数式编程的关系，所以现在我们不会谈论数学，而是从实用主义的角度来探讨函数式编程。

让我们从例子开始，了解函数式编程和 OOP 风格之间的差异。以下示例给出了两种不同的数组元素乘法方法：

```swift
let numbers = [9, 29, 19, 79]

// Imperative example
var tripledNumbers:[Int] = []
for number in numbers {
    tripledNumbers.append(number * 3)
}
print(tripledNumbers)

// Declarative example
let tripledIntNumbers = numbers.map({ number in 3 * number })
print(tripledIntNumbers)

```

在命令式示例中，我们给出一个命令遍历数组中的每个元素，将每个元素乘以`3`，并将其添加到新的数组中。在声明式示例中，我们声明了数字应该如何映射。我们将在接下来的章节中提供更多关于声明式编程的示例。

在函数式编程中，函数是基本的构建块。在面向对象编程（OOP）中，程序由类和语句组成，当执行时，这些语句会改变类的状态。

函数式编程避免使用可变状态。避免可变状态使得代码更容易测试、阅读和理解，尽管在某些情况下（如文件和数据库操作）避免可变状态并不容易。

函数式编程要求函数是一等公民。一等函数被当作任何其他值一样对待，可以被传递给其他函数或作为函数的结果返回。

函数可以形成高阶函数，它们接受其他函数作为参数。高阶函数用于重构代码和减少重复。高阶函数可以用来实现**领域特定语言**（**DSL**）。

函数是纯函数，它们不依赖于自身之外的数据，也不改变自身之外的数据。纯函数每次执行都会提供相同的结果。纯函数的这个特性被称为**引用透明性**，使得在代码上可以进行**等式推理**。

在函数式编程中，表达式可以懒加载。例如，在下面的代码示例中，只有数组中的第一个元素被评估：

```swift
let oneToFour = [1, 2, 3, 4]
let firstNumber = oneToFour.lazy.map({ $0 * 3}).first!
print(firstNumber) // The result is going to be 3

```

在本例中，使用`lazy`关键字来获取集合的懒加载版本，因此只有数组中的第一个元素被乘以`3`，其余元素不会被映射。

# Swift 编程语言

Swift 是由 Apple 开发的开源混合语言，它结合了面向对象编程和协议导向编程，以及函数式编程范式。Swift 可以与 Objective-C 一起用于开发 macOS、iOS、tvOS 和 watchOS 应用程序。Swift 也可以在 Ubuntu Linux 上用于开发 Web 应用程序。本书解释了 Swift 3.0 Preview 1，并使用 Xcode 8.0 beta。GitHub 仓库中的源代码将频繁更新，以跟上 Swift 的变化。

## Swift 特性

Swift 借鉴了许多其他编程语言的概念，如 Scala、Haskell、C#、Rust 和 Objective-C，并具有以下特性。

### 现代语法

Swift 拥有现代语法，消除了像 Objective-C 这样的编程语言的冗余。例如，以下代码示例展示了具有属性和方法的 Objective-C 类。Objective-C 类在两个独立的文件中定义（接口和实现）。`VerboseClass.h`文件定义了一个接口，作为`NSObject`类的子类。它定义了一个属性`ourArray`和一个方法`aMethod`。

实现文件导入头文件类，并为`aMethod`提供实现：

```swift
// VerboseClass.h
@interface VerboseClass: NSObject
@property (nonatomic, strong) NSArray *ourArray;
- (void)aMethod:(NSArray *)anArray;
@end

// VerboseClass.m
#import "VerboseClass.h"

@implementation VerboseClass

- (void)aMethod:(NSArray *)anArray {
    self.ourArray = [[NSArray alloc] initWithArray:anArray];
}

@end

// TestVerboseClass.m
#import "VerboseClass.h"

@interface TestVerboseClass : NSObject

@end

@implementation TestVerboseClass

- (void)aMethod {
    VerboseClass *ourOBJCClass = [[VerboseClass alloc] init];
    [ourOBJCClass aMethod: @[@"One", @"Two", @"Three"]];
    NSLog(@"%@", ourOBJCClass.ourArray);
}

@end

```

在 Swift 中，可以通过以下方式实现类似的功能：

```swift
class ASwiftClass {
    var ourArray: [String] = []

    func aMethod(anArray: [String]) {
        self.ourArray = anArray
    }
}

let aSwiftClassInstance = ASwiftClass()
aSwiftClassInstance.aMethod(anArray: ["one", "Two", "Three"])
print(aSwiftClassInstance.ourArray)

```

从这个例子中可以看出，Swift 消除了许多不必要的语法，使代码非常简洁易读。

### 类型安全和类型推断

Swift 是一种类型安全的语言，与 Ruby 和 JavaScript 等语言不同。与 Objective-C 中的类型可变集合相反，Swift 提供了类型安全的集合。Swift 通过类型推断机制自动推导类型，这种机制存在于 C# 和 C++ 11 等语言中。例如，在以下示例中，`constString` 在编译时被推断为 `String`，因此不需要注释类型：

```swift
let constString = "This is a string constant" 

```

### 不可变性

Swift 使得定义不可变值（即常量）变得容易，从而增强了函数式编程，因为不可变性是函数式编程中的关键概念之一。一旦常量被初始化，它们就不能被更改。虽然在 Java 等语言中也可以实现不可变性，但并不像 Swift 那样容易。要在 Swift 中定义任何不可变类型，可以使用 `let` 关键字，无论它是自定义类型、集合类型还是 `Struct` 类型。

### 无状态编程

Swift 提供了非常强大的结构和枚举，它们通过值传递，并且可以是无状态的；因此，它们非常高效。无状态编程简化了并发和多线程。

### 一等函数

函数在 Swift 中是一等类型，就像 Ruby、JavaScript 和 Go 等语言一样，因此它们可以被存储、传递和返回。一等函数使 Swift 能够实现函数式编程风格。

### 高阶函数

高阶函数可以将其他函数作为其参数。Swift 提供了 `map`、`filter` 和 `reduce` 等高阶函数。此外，在 Swift 中，我们还可以开发自己的高阶函数和 DSL。

### 模式匹配

模式匹配是解构值并根据正确的值匹配来匹配不同的情况的能力。模式匹配能力存在于 Scala、Erlang 和 Haskell 等语言中。Swift 提供了带有 `where` 子句的强大 switch-cases 和 if-cases。

### 泛型

Swift 提供了泛型，这使得编写不特定于类型的代码成为可能，并且可以用于不同类型。

### 闭包

闭包是代码块，可以被传递。闭包捕获它们定义的上下文中的常量和变量。与 Objective-C 中的块相比，Swift 提供了更简单的语法。

### 下标

Swift 提供了下标，这些下标是访问集合、列表、序列或自定义类型成员的快捷方式。下标可以用来通过索引设置和获取值，而无需为设置和获取分别使用单独的方法。

### 可选链

Swift 提供了可选类型，它可以有某些或没有值。Swift 还提供了可选链，以安全有效地使用可选类型。可选链使我们能够查询和调用可能为 `nil` 的可选类型的属性、方法和下标。

### 扩展

Swift 提供了类似于 Objective-C 中的分类的扩展（Extensions）。扩展可以向现有的类、结构体、枚举或协议类型添加新功能，即使它是封闭源代码的。

### Objective-C 和 Swift 桥接头

桥接头（Bridging headers）使我们能够在项目中混合使用 Swift 和 Objective-C。这种功能使得我们能够在 Swift 项目中使用之前编写的 Objective-C 代码，反之亦然。

### 自动引用计数（Automatic Reference Counting，ARC）

Swift 通过 **自动引用计数**（**ARC**）处理内存管理，类似于 Objective-C，而不像 Java 和 C# 等使用垃圾回收的语言。ARC 用于初始化和解除初始化资源，从而在不再需要时释放类实例的内存分配。ARC 跟踪代码实例中的保留和释放，以有效地管理内存资源。

### REPL 和游乐场

Xcode 提供了 **读取-评估-打印循环**（**REPL**）命令行环境，用于在无需编写程序的情况下实验 Swift 编程语言。此外，Swift 还提供了游乐场（Playgrounds），使我们能够快速测试 Swift 代码片段，并通过可视化界面实时查看结果。本书中将广泛使用游乐场。此外，大多数章节中的代码示例都可以在 Swift Playgrounds App 中进行实验（[`developer.apple.com/swift/playgrounds/`](https://developer.apple.com/swift/playgrounds/)）。

## 语言基础

本节将简要介绍 Swift 编程语言的基础知识。本章后续小节中的主题将在后面的章节中详细解释。

### 类型安全和类型推断

Swift 是一种类型安全的语言。这意味着一旦我们定义了常量、变量或表达式，就不能更改其类型。此外，Swift 的类型安全特性使我们能够在编译时找到类型不匹配。

Swift 提供了类型推断。Swift 会自动推断变量、常量或表达式的类型，因此我们不需要在定义时指定类型。让我们检查以下表达式：

```swift
let pi = 3.14159 
var primeNumber = 691 
let name = "my name" 

```

在这些表达式中，Swift 推断 `pi` 为 `Double` 类型，`primeNumber` 为 `Int` 类型，以及 `name` 为 `String` 类型。如果我们需要特殊类型如 `Int64`，我们需要对类型进行注解。

### 类型注解

在 Swift 中，我们可以注解类型，换句话说，明确指定变量或表达式的类型。让我们看看以下示例：

```swift
let pi: Double = 3.14159 
let piAndPhi: (Double, Double) = (3.14159, 1.618) 
func ourFunction(a: Int) { /* ... */ } 

```

在这个示例中，我们定义了一个注解为 `Double` 的常量（`pi`），一个名为 `piAndPhi` 的元组注解为 `(Double, Double)`，以及 `ourFunction` 函数的参数注解为 `Int`。

### 小贴士

**下载示例代码**本书的代码包也托管在 GitHub 上，网址为 [`github.com/PacktPublishing/Swift-3-Functional-Programming`](https://github.com/PacktPublishing/Swift-3-Functional-Programming)。我们还有其他丰富的图书和视频的代码包，可在 [`github.com/PacktPublishing/`](https://github.com/PacktPublishing/) 找到。查看它们吧！

### 类型别名

类型别名定义了现有类型的替代名称。我们使用 `typealias` 关键字定义类型别名。类型别名在需要通过更合适的上下文名称引用现有类型时很有用，例如在处理来自外部源的具体大小数据时。例如，在以下示例中，我们为无符号 32 位整数提供了一个别名，该别名可以在我们稍后的代码中使用：

```swift
typealias UnsignedInteger = UInt32 

```

`typealias` 定义可以用作简化闭包和函数定义。

### 不可变性

Swift 使得定义可变和不可变变量成为可能。`let` 关键字用于不可变声明，而 `var` 关键字用于可变声明。任何使用 `let` 关键字声明的变量将不会开放进行更改。在以下示例中，我们使用 `var` 关键字定义 `aMutableString` 以便我们稍后能够修改它；相比之下，我们无法修改使用 `let` 关键字定义的 `aConstString`：

```swift
var aMutableString = "This is a variable String" 
let aConstString = "This is a constant String" 

```

在函数式编程中，建议尽可能使用 `let` 将属性定义为常量或不可变。不可变变量更容易跟踪且错误更少。在某些情况下，例如 *CoreData* 编程，*SDK* 要求可变属性；然而，在这些情况下，建议使用可变变量。

### 元组

Swift 提供元组，以便可以将多个值组合成一个单一的复合值。让我们考虑以下示例：

```swift
// Tuples 
let http400Error = (400, "Bad Request")
// http400Error is of type (Int, String), and equals (400, "Bad Request")

// Decompose a Tuple's content 
let (requestStatusCode, requestStatusMessage) = http400Error 

```

元组可以用作函数的返回类型，以实现多返回值函数。

### 可选类型

Swift 提供可选类型，以便在可能不存在值的情况下使用。可选类型将具有一些或没有值。`?` 符号用于将变量定义为可选。让我们考虑以下示例：

```swift
// Optional value either contains a value or contains nil
var optionalString: String? = "A String literal"
optionalString = nil

```

`!` 符号可以用来强制展开可选类型中的值。例如，以下示例强制展开 `optionalString` 变量：

```swift
optionalString = "An optional String"
print(optionalString!)

```

强制展开可选类型可能会导致错误，如果可选类型没有值，因此不建议使用这种方法，因为它很难确保在不同情况下可选类型中是否有值。更好的方法是使用可选绑定技术来找出可选类型是否包含值。让我们考虑以下示例：

```swift
let nilName:String? = nil
if let familyName = nilName {
    let greetingfamilyName = "Hello, Mr. \(familyName)"
} else {
    // Optional does not have a value
}

```

### 基本运算符

Swift 提供以下基本操作：

+   与许多不同的编程语言一样，`=` 运算符用于赋值。

+   加法运算符 `+` 用于加法，减法运算符 `-` 用于减法，乘法运算符 `*` 用于乘法，除法运算符 `/` 用于除法，取余运算符 `%` 用于求余。这些运算符是函数，可以被传递给其他函数。

+   一元减法运算符 `-i`，一元加法运算符 `+i`。

+   复合赋值运算符 `+=`、`-=` 和 `*=`。

+   等于运算符 `a == b`，不等式运算符 `a != b`，以及大于比较 `a > b`、小于比较 `a < b` 和小于等于比较 `a <= b`。

+   三元条件运算符，`question ? answer1: answer2`。

+   隐式展开 `a ?? b` 如果可选 `a` 有值，则展开可选 `a` 并返回默认值 `b`；如果 `a` 是 `nil`。

+   范围运算符：

    +   闭区间 (`a...b`) 包含值 `a` 和 `b`

    +   半开区间 (`a..<b`) 包含 `a` 但不包含 `b`

+   逻辑运算符：

    +   `!a` 运算符不是 NOT a

    +   `a && b` 运算符是逻辑与

    +   `a || b` 运算符是逻辑或

### 字符串和字符

在 Swift 中，`String` 是字符的有序集合。`String` 是一个结构体，而不是类。结构体是 Swift 中的值类型；因此，任何 `String` 都是值类型，通过值传递，而不是通过引用传递。

#### 不可变性

字符串可以使用 `let` 定义为不可变。使用 `var` 定义的字符串将是可变的。

#### 字符串字面量

`String` 字面量可用于创建 `String` 实例。在以下编码示例中，我们使用 `String` 字面量定义并初始化 `aVegetable`：

```swift
let aVegetable = "Arugula" 

```

#### 空字符串

空字符串可以按以下方式初始化：

```swift
var anEmptyString = ""
var anotherEmptyString = String()

```

这两个字符串都是空的，彼此等价。要确定一个 `String` 是否为空，可以使用 `isEmpty` 属性如下：

```swift
if anEmptyString.isEmpty {
    print("String is empty")
}

```

#### 连接字符串和字符

字符串和字符可以按以下方式连接：

```swift
let string1 = "Hello"
let string2 = " Mr"
var welcome = string1 + string2

var instruction = "Follow us please"
instruction += string2

let exclamationMark: Character = "!"
welcome.append(exclamationMark)

```

#### 字符串插值

字符串插值是一种通过在字符串字面量中包含它们的值来构造新的 `String` 值的方法，这些值可以是常量、变量、字面量和表达式。以下是一个示例：

```swift
let multiplier = 3
let message = "\(multiplier) times 7.5 is \(Double (multiplier) * 7.5)"
// message is "3 times 7.5 is 22.5"

```

#### 字符串比较

字符串可以使用 `==` 进行相等比较，使用 `!=` 进行不等比较。

`hasPrefix` 和 `hasSuffix` 方法可用于前缀和后缀相等性检查。

### 集合

Swift 提供了诸如数组、字典和集合等类型化的集合。在 Swift 中，与 Objective-C 不同，集合中的所有元素都将具有相同的类型，并且我们无法在定义后更改集合的类型。

我们可以使用 `let` 定义不可变集合，使用 `var` 定义可变集合，如下例所示：

```swift
// Arrays and Dictionaries
var cheeses = ["Brie", "Tete de Moine", "Cambozola", "Camembert"]
cheeses[2] = "Roquefort"
var cheeseWinePairs = [
    "Brie":"Chardonnay",
    "Camembert":"Champagne",
    "Gruyere":"Sauvignon Blanc"
]

cheeseWinePairs ["Cheddar"] = "Cabarnet Sauvignon"
// To create an empty array or dictionary
let emptyArray = [String]()
let emptyDictionary = Dictionary<String, Float>()
cheeses = []
cheeseWinePairs = [:]

```

`for-in` 循环可用于遍历集合中的项。

### 控制流程

Swift 提供了不同的控制流程，以下小节将进行解释。

#### for 循环

Swift 提供了 `for` 和 `for-in` 循环。我们可以使用 `for-in` 循环遍历集合中的项、数字序列（如范围）或字符串表达式中的字符。以下示例展示了如何使用 `for-in` 循环遍历 `Int` 数组中的所有项：

```swift
let scores = [65, 75, 92, 87, 68]
var teamScore = 0

for score in scores {
    if score > 70 {
        teamScore = teamScore + 3
    } else {
        teamScore = teamScore + 1
    }
}

```

以下是如何遍历字典：

```swift
for (cheese, wine) in cheeseWinePairs {
    print("\(cheese): \(wine)")
}

```

我们可以使用带有条件和增量/减量器的`for`循环。以下示例展示了`for`循环的示例：

```swift
var count = 0
for var i = 0; i < 3; ++i {
    count += i
}

```

由于从 Swift 3.0 开始移除了具有增量器/减量器的 C 风格`for`循环，建议使用以下方式的`for-in`循环来代替，如下所示：

```swift
var count = 0
for i in 0...3 {
    count += i
}

```

#### while loops

Swift 提供了`while`和`repeat-while`循环。一个`while`或`repeat-while`循环会执行一系列表达式，直到条件变为假。让我们考虑以下示例：

```swift
var n = 2
while n < 100 {
    n = n * 2
}

var m = 2
repeat {
    m = m * 2
} while m < 100

```

`while`循环在每个迭代的开始时评估其条件。`repeat-while`循环在每个迭代的结束时评估其条件。

#### stride

`stride`函数使我们能够以非一为步长遍历范围。有两个`stride`函数：`stride to`函数，它遍历排他性范围，以及`stride through`函数，它遍历包含性范围。让我们考虑以下示例：

```swift
let fourToTwo = Array(stride(from: 4, to: 1, by: -1)) // [4, 3, 2]
let fourToOne = Array(stride(from:4, through: 1, by: -1)) // [4, 3, 2, 1]

```

#### if

Swift 提供了`if`来定义条件语句。只有当条件语句为`true`时，才会执行一组语句。例如，在以下示例中，由于`anEmptyString`为空，将执行`print`语句：

```swift
var anEmptyString = ""
if anEmptyString.isEmpty {
    print("An empty String")
} else {
    // String is not empty.
}

```

#### switch

Swift 提供了`switch`语句来比较一个值与不同的匹配模式。一旦匹配到模式，将执行相关的语句。与大多数其他基于 C 的编程语言不同，Swift 不需要在每个`case`中都有`break`语句，并支持任何值类型。`switch`语句可用于范围匹配，并且`switch`语句中的`where`子句可以用来检查额外的条件。以下示例展示了带有附加条件检查的简单`switch`语句：

```swift
let aNumber = "Four or Five"
switch aNumber {
    case "One":
        let one = "One"
    case "Two", "Three":
        let twoOrThree = "Two or Three"
    case let x where x.hasSuffix("Five"):
        let fourOrFive = "it is \(x)"
    default:
        let anyOtherNumber = "Any other number"
}

```

#### guard

A `guard` statement can be used for early exits. We can use a `guard` statement to require that a condition must be `true` in order for the code after the `guard` statement to be executed. The following example presents the `guard` statement usage:

```swift
func greet(person: [String: String]) {
    guard let name = person["name"] else {
        return
    }
    print("Hello Ms \(name)!")
}

```

在这个例子中，`greet`函数需要一个表示人的`name`的值。因此，它使用`guard`语句检查其是否存在，否则它将返回并继续执行。

### 函数

函数是执行特定任务的独立代码块。

在 Swift 中，函数是一等公民，这意味着它们可以被存储、传递和返回。函数可以被柯里化，并定义为接受其他函数作为其参数的高阶函数。Swift 中的函数可以使用元组具有多个输入参数和多个返回值。让我们看看以下示例：

```swift
func greet(name: String, day: String) -> String {
    return "Hello \(name), today is \(day)"
}

greet(name: "Ted", day:"Saturday")

```

函数可以有可变参数。让我们考虑以下示例：

```swift
// Variable number of arguments in functions - Variadic Parameters
func sumOf(numbers:Int...) -> (Int, Int) {
    var sum = 0
    var counter = 0
    for number in numbers {
        sum += number
        counter += 1
    }
    return (sum, counter)
}

sumOf()
sumOf(numbers: 7, 9, 45)

```

在 Swift 3.0 之前，函数可以有可变和不可变参数。让我们考虑以下示例：

```swift
func alignRight(var string: String, count: Int, pad: Character) -> String {
    let amountToPad = count - string.characters.count
    if amountToPad < 1 {
        return string
    }
    let padString = String(pad)
    for _ in 1...amountToPad {
        string = padString + string
    }
    return string
} 

```

可变参数在 Swift 函数式编程中不受欢迎，并且从 Swift 3.0 开始被移除。

函数可以有`inout`参数。让我们考虑以下示例：

```swift
func swapTwoInts( a: inout Int, b: inout Int) {
    let temporaryA = a
    a = b
    b = temporaryA
}

```

在函数式 Swift 中，`inout` 参数并不受欢迎，因为它们会改变状态并使函数变得不纯。

在 Swift 中，我们可以定义嵌套函数。以下示例展示了一个名为 `add` 的函数嵌套在另一个函数内部。嵌套函数可以访问其父函数的作用域内的数据。在这个例子中，`add` 函数可以访问 `y` 变量：

```swift
func returnTwenty() -> Int {
    var y = 10
    func add() {
        y += 10
    }
    add()
    return y
}

returnTwenty()

```

在 Swift 中，函数可以返回其他函数。在以下示例中，`makeIncrementer` 函数返回一个函数，该函数接收一个 `Int` 值并返回一个 `Int` 值 (`Int -> Int`)：

```swift
// Return another function as its value
func makeIncrementer() -> (Int -> Int) {
    func addOne(number: Int) -> Int {
        return 1 + number
    }
    return addOne
}

var increment = makeIncrementer()
increment(7)

```

### 闭包

闭包是包含特定功能的自包含代码块，可以被存储、传递并在代码中使用。闭包在 C 和 Objective-C 中相当于块。闭包可以捕获并存储它们定义的上下文中任何常量和变量的引用。嵌套函数是闭包的特殊情况。闭包是引用类型，可以作为变量、常量和类型别名存储。它们可以被传递给函数并从函数返回。

以下示例展示了来自网站 http://goshdarnclosuresyntax.com 的 Swift 中不同闭包声明的示例：

```swift
// As a variable:
var closureName: (parameterTypes) -> (returnType)

//As an optional variable:
var closureName: ((parameterTypes) -> (returnType))?

//As a type alias:
typealias closureType = (parameterTypes) -> (returnType)

```

### 映射、过滤和缩减

Swift 提供了 `map`、`filter` 和 `reduce` 函数，它们是高阶函数。

#### 映射

`map` 函数解决了使用函数转换数组元素的问题。让我们考虑以下示例：

```swift
// Return an `Array` containing the results of calling `transform(x)` on
  each element `x` of `self`
// func map<U>(transform: (T) -> U) -> [U]
let numbers = [10, 30, 91, 50, 100, 39, 74]
var formattedNumbers: [String] = []

for number in numbers {
    let formattedNumber = "\(number)$"
    formattedNumbers.append(formattedNumber)
}

let mappedNumbers = numbers.map { "\($0)$" }

```

#### 过滤

`filter` 函数接受一个函数，该函数给定数组中的一个元素，返回 `Bool` 值，指示该元素是否应该包含在结果数组中。让我们考虑以下示例：

```swift
// Return an Array containing the elements x of self for which
  includeElement(x)` is `true`
// func filter(includeElement: (T) -> Bool) -> [T]
let someEvenNumbers = numbers.filter { $0 % 2 == 0 }

```

#### 缩减

`reduce` 函数将数组缩减为一个单一值。它接受两个参数：一个起始值和一个函数，该函数接受一个运行总和以及数组中的一个元素作为参数，并返回一个新的运行总和。让我们考虑以下示例：

```swift
// Return the result of repeatedly calling `combine` with an accumulated
  value initialized to `initial` and each element of `self`, in turn,
  that is return `combine(combine(...combine(combine(initial, self[0]),
  self[1]),...self[count-2]), self[count-1])`.
// func reduce<U>(initial: U, combine: (U, T) -> U) -> U
let total = numbers.reduce(0) { $0 + $1 }

```

### 枚举

在 Swift 中，枚举定义了一个相关值的公共类型，并使我们能够以类型安全的方式处理这些值。为每个枚举成员提供的值可以是 `String`、`Character`、`Int` 或任何浮点类型。枚举可以存储任何给定类型的关联值，如果需要，枚举的每个成员的值类型可以不同。枚举成员可以预先填充默认值（称为原始值），它们都是同一类型。让我们考虑以下示例：

```swift
enum MLSTeam {
    case montreal
    case toronto
    case newYork
    case columbus
    case losAngeles
    case seattle
}

let theTeam = MLSTeam.montreal

```

枚举值可以用 `switch` 语句进行匹配，这在以下示例中可以看到：

```swift
switch theTeam {
    case .montreal:
        print("Montreal Impact")
    case .toronto:
        print("Toronto FC")
    case .newYork:
        print("Newyork Redbulls")
    case .columbus:
        print("Columbus Crew")
    case .losAngeles:
        print("LA Galaxy")
    case .seattle:
        print("Seattle Sounders")
}

```

Swift 中的枚举实际上是通过对其他类型进行组合创建的代数数据类型。让我们考虑以下示例：

```swift
enum NHLTeam { case canadiens, senators, rangers, penguins, blackHawks,
  capitals}

enum Team {
    case hockey(NHLTeam)
    case soccer(MLSTeam)
}

struct HockeyAndSoccerTeams {
    var hockey: NHLTeam
    var soccer: MLSTeam
}

```

`MLSTeam`和`NHLTeam`枚举各有六个潜在值。如果我们将它们结合起来，我们将有两个新的类型。一个`Team`枚举可以是`NHLTeam`或`MLSTeam`，因此它有 12 个潜在值——即`NHLTeam`和`MLSTeam`潜在值的总和。因此，枚举`Team`是一个和类型。

要有一个`HockeyAndSoccerTeams`结构体，我们需要为`NHLTeam`和`MLSTeam`选择一个值，这样它就有 36 个潜在值——即`NHLTeam`和`MLSTeam`值的乘积。因此，`HockeyAndSoccerTeams`是一个乘积类型。

在 Swift 中，枚举的选项可以有多个值。如果它恰好是唯一选项，那么这个枚举就变成了乘积类型。以下示例展示了枚举作为乘积类型：

```swift
enum HockeyAndSoccerTeams {
    case Value(hockey: NHLTeam, soccer: MLSTeam)
}

```

由于我们可以在 Swift 中创建和或乘积类型，我们可以说 Swift 对代数数据类型有第一级支持。

### 泛型

泛型代码使我们能够编写灵活且可重用的函数和类型，它们可以与任何类型一起工作，前提是我们定义了要求。例如，以下使用`inout`参数来交换两个值的函数只能与`Int`值一起使用：

```swift
func swapTwoIntegers( a: inout Int, b: inout Int) {
    let tempA = a
    a = b
    b = tempA
}

```

为了使此函数与任何类型一起工作，可以使用泛型，如下例所示：

```swift
func swapTwoValues<T>( a: inout T, b: inout T) {
    let tempA = a
    a = b
    b = tempA
}

```

### 类与结构体

类和结构体是通用、灵活的构造，成为程序代码的构建块。它们具有以下特性：

+   可以定义属性来存储值

+   可以定义方法来提供功能

+   可以定义下标以使用下标语法访问它们的值

+   可以定义初始化器来设置其功能，超出默认实现

+   它们可以遵守协议以提供某些类型的标准功能

#### 类与结构体

本节比较类和结构体：

+   继承使一个类能够继承另一个类的特性

+   类型转换使我们能够在运行时检查和解释类实例的类型

+   析构器使类的实例能够释放它所分配的任何资源

+   引用计数允许对类实例有多个引用

+   结构体是值类型，因此在代码中传递时总是被复制

+   结构体不使用引用计数

+   类是引用类型

#### 选择类与结构体

当以下条件之一适用时，我们考虑创建结构体：

+   结构体的主要目的是封装几个相对简单的数据值

+   我们可以合理地预期，当我们分配或传递结构体的实例时，封装的值将被复制而不是被引用

+   结构体存储的任何属性本身也是值类型，这也会期望被复制而不是引用

+   结构体不需要从另一个现有类型继承属性或行为

结构体的良好候选者示例包括以下内容：

+   几何形状的大小

+   3D 坐标系统中的一个点

#### 身份运算符

由于类是引用类型，多个常量和变量可能在实际中指向同一个类的单个实例。为了确定两个常量或变量是否精确地指向同一个类的实例，Swift 提供了以下身份运算符：

+   等同于 (`===`)

+   不等同于 (`!==`)

### 属性

属性将值与特定的类、结构或枚举相关联。Swift 允许我们直接设置结构属性的字段属性，而无需将整个对象属性设置为新的值。所有结构都有自动生成的成员初始化器，可以用来初始化新结构实例的成员属性。这并不适用于类实例。

#### 属性观察者

属性观察者用于响应属性值的更改。每当属性的值被设置时，即使新值与属性的当前值相同，属性观察者都会被调用。我们可以在属性上定义以下观察者之一或两个：

+   `willSet` 观察者在值存储之前被调用

+   `didSet` 观察者在新值存储后立即被调用

在初始化器中设置属性之前，`willSet` 和 `didSet` 观察者不会被调用。

### 方法

方法是与特定类型相关联的函数。实例方法是调用特定类型实例的函数。类型方法是调用类型本身的函数。

以下示例展示了一个包含名为 `someTypeMethod()` 的类型方法的类：

```swift
class AClass {
    class func someTypeMethod() {
        // type method body
    }
}
```

我们可以将此方法称为如下：

```swift
AClass.someTypeMethod()

```

### 下标

下标是访问集合、列表、序列或任何实现下标的自定义类型的成员元素的快捷方式。让我们考虑以下示例：

```swift
struct TimesTable {
    let multiplier: Int
    subscript(index: Int) -> Int {
        return multiplier * index
    }
}

let fiveTimesTable = TimesTable(multiplier: 5)
print("six times five is \(fiveTimesTable[6])")
// prints "six times five is 30"

```

### 继承

一个类可以从另一个类继承方法、属性和其他特性：

```swift
class SomeSubClass: SomeSuperClass

```

Swift 类不继承自通用基类。我们定义的未指定超类的类自动成为我们构建的基础类。要覆盖继承的特性，我们使用 `override` 关键字作为覆盖定义的前缀。被覆盖的方法、属性或下标可以通过调用 `super` 来调用超类版本。要防止覆盖，可以使用 `final` 关键字。

### 初始化

准备一个类、结构体或枚举的实例以供使用的过程称为初始化。类和结构体必须在创建该类或结构体的实例时将所有存储属性设置为适当的初始值。存储属性不能处于中间状态。我们可以在初始化过程中修改常量属性的值，只要在初始化完成时将其设置为确定的值。Swift 为任何结构体或基类提供默认初始化器，如果它为所有属性提供默认值并且没有提供至少一个初始化器。让我们考虑以下示例：

```swift
class ShoppingItem {
    var name: String?
    var quantity = 1
    var purchased = false
}

var item = ShoppingItem()

```

如果我们没有定义任何自定义初始化器，`struct` 类型会自动接收一个成员初始化器，即使结构体的存储属性没有默认值。

Swift 为类类型定义了两种初始化器：

+   **指定初始化器**：能够完全初始化对象的方法

+   **便利初始化器**：依赖于其他方法来完成初始化的方法

### 解构

解构器在类实例被释放之前立即调用。Swift 在实例不再需要时自动释放实例，以释放资源（ARC）。

### 自动引用计数

引用计数仅适用于类的实例。结构体和枚举是值类型，不是引用类型，它们不是通过引用存储和传递的。

弱引用可以用来解决强引用循环，可以定义为如下：

```swift
weak var aWeakProperty 

```

一个非拥有引用不会对其引用的实例保持强引用。然而，与弱引用不同的是，非拥有引用始终定义为非可选类型。可以使用闭包捕获列表来解决闭包强引用循环。

当闭包和它捕获的实例将始终相互引用并在同一时间被释放时，可以在闭包中定义一个非拥有引用。

当捕获的引用可能在未来的某个时刻变为`nil`时，可以定义一个作为弱引用的捕获。弱引用始终为可选类型。让我们考虑以下示例：

```swift
class AClassWithLazyClosure {
    lazy var aClosure: (Int, String) -> String = {
        [unowned self] (index: Int, stringToProcess: String) -> String in
        // closure body goes here
        return ""
    }
}

```

### 可选值和可选链

可选值是 Swift 类型，可以具有一些或没有值。可选链是一个查询和调用可能当前为`nil`的可选值的属性、方法和子脚本的进程。Swift 中的可选链类似于 Objective-C 中的`nil`消息，但以一种适用于任何类型并且可以检查成功或失败的方式。让我们考虑以下示例：

```swift
// Optional chaining
class Residence {
    var numberOfRooms = 1
}

class Person {
    var residence: Residence?
}

let jeanMarc = Person()
// This can be used for calling methods and subscripts through optional
  chaining too
if let roomCount = jeanMarc.residence?.numberOfRooms {
    // Use the roomCount
}

```

在这个示例中，我们能够通过可选链访问`numberOfRooms`，这是可选类型（`Residence`）的一个属性。

### 错误处理

Swift 在运行时提供支持抛出、捕获、传播和处理可恢复错误的机制。

值类型应该符合 `ErrorType` 协议才能表示为错误。以下示例展示了某些 4xx 和 5xx HTTP 错误作为 `enum`：

```swift
enum HttpError: ErrorType {
    case badRequest
    case unauthorized
    case forbidden
    case requestTimeOut
    case unsupportedMediaType
    case internalServerError
    case notImplemented
    case badGateway
    case serviceUnavailable
}

```

我们将能够使用 `throw` 关键字抛出错误，并使用 `throws` 关键字标记可以抛出错误的函数。

我们可以使用 `do-catch` 语句通过运行代码块来处理错误。以下示例展示了在 `do-catch` 语句中处理 JSON 解析错误：

```swift
protocol HttpProtocol{
    func didRecieveResults(results:NSDictionary)
}

struct WebServiceManager {
    var delegate:HttpProtocol?
    let data: NSData
    func test() {
        do {
            let jsonResult: NSDictionary = try
              NSJSONSerialization.JSONObjectWithData(self.data,
              options: NSJSONReadingOptions.MutableContainers) as!
              NSDictionary
            self.delegate?.didRecieveResults(jsonResult)
        } catch let error as NSError {
            print("json error" + error.localizedDescription)
        }
    }
}

```

我们可以使用 `defer` 语句在代码执行离开当前代码块之前执行一系列语句，无论执行如何离开当前代码块。

### 类型转换

类型转换是一种检查实例类型的方式，并且/或者将实例作为它类层次结构中其他地方的不同超类或子类来处理。有两种类型的运算符用于检查和转换类型：

+   类型检查运算符（`is`）：这检查一个实例是否为确定的子类类型。

+   类型转换运算符（`as` 和 `as?`）：一个确定类类型的常量或变量可能在底层引用一个子类的实例。如果情况如此，我们可以尝试使用 `as` 将其向下转换为子类类型。

### Any 和 AnyObject

Swift 提供了两个特殊的类型别名来处理非特定类型：

+   `AnyObject` 可以表示任何类类型的实例

+   `Any` 可以表示任何类型的实例，包括结构体、枚举和函数类型

`Any` 和 `AnyObject` 类型别名必须仅在我们明确需要它们提供的行为和能力时使用。在我们的代码中明确指定我们期望与之一起工作的类型比使用 `Any` 和 `AnyObject` 类型更好，因为它们可以表示任何类型，并提供了动态性而不是安全性。让我们考虑以下示例：

```swift
class Movie {
    var director: String
    var name: String
    init(name: String, director: String) {
        self.director = director
        self.name = name
    }
}

let objects: [AnyObject] = [
    Movie(name: "The Shawshank Redemption", director: "Frank Darabont"),
    Movie(name: "The Godfather", director: "Francis Ford Coppola")
]

for object in objects {
    let movie = object as! Movie
    print("Movie: '\(movie.name)', dir. \(movie.director)")
}

// Shorter syntax
for movie in objects as! [Movie] {
    print("Movie: '\(movie.name)', dir. \(movie.director)")
}

```

### 嵌套类型

枚举通常被创建来支持特定类或结构的功能。同样，在复杂类型的上下文中声明仅用于该上下文的实用类和结构体可能也很方便。

Swift 允许我们声明嵌套类型，即在它们支持的类型定义中嵌套支持枚举、类和结构体。以下示例，借鉴自 *《Swift 编程语言（Swift 3.0）》*，由 *苹果公司* 提供，展示了嵌套类型：

```swift
struct BlackjackCard {
    // nested Suit enumeration
    enum Suit: Character {
        case spades = "♠",
        hearts = "♡",
        diamonds = "♢",
        clubs = "♣"
    }

    // nested Rank enumeration
    enum Rank: Int {
        case two = 2, three, four, five, six, seven, eight, nine, ten
        case jack, queen, king, ace

        // nested struct
        struct Values {
            let first: Int, second: Int?
        }

        var values: Values {
            switch self {
            case .ace:
                return Values(first: 1, second: 11)
            case .jack, .queen, .king:
                return Values(first: 10, second: nil)
            default:
                return Values(first: self.rawValue, second: nil)
            }
        }
    }

    let rank: Rank, suit: Suit

    var description: String {
        var output = "suit is \(suit.rawValue),"
        output += " value is \(rank.values.first)"
        if let second = rank.values.second {
            output += " or \(second)"
        }
        return output
    }
}

```

### 扩展

扩展可以向现有的类、结构体或枚举类型添加新功能。这包括扩展我们无法访问原始源代码的类型的能力。

Swift 中的扩展使我们能够执行以下操作：

+   定义实例方法和类型方法

+   提供新的初始化器

+   定义和使用新的嵌套类型

+   定义下标

+   添加计算属性和计算静态属性

+   使现有类型符合新的协议

扩展使我们能够向类型添加新功能，但我们无法覆盖现有功能。

在以下示例中，我们通过使 `AType` 符合两个协议来扩展它：

```swift
extension AType: AProtocol, BProtocol {
}

```

以下示例展示了通过添加计算属性对 `Double` 的扩展：

```swift
// Computed Properties
extension Double {
    var mm: Double { return self / 1_000.0 }
    var ft: Double { return self / 3.2884 }
}

let threeInch = 76.2.mm
let fiveFeet = 5.ft

```

### 协议

一个 `protocol` 定义了适合特定任务或功能的方法、属性和其他要求签名或类型。协议实际上并没有实现任何功能。它只描述了实现将看起来像什么。一个提供实际要求实现的类、结构体或枚举可以采用协议。协议使用与正常方法相同的语法，但不允许为方法参数指定默认值。`is` 操作符可以用来检查一个实例是否遵从协议。只有当我们的协议为类标记了 `@objc` 时，我们才能检查协议遵从性。`as` 操作符可以用来转换到特定的协议。

#### 协议作为类型

我们定义的任何协议都将成为我们代码中使用的完整类型。我们可以如下使用协议：

+   函数、方法或初始化器中的参数类型或返回类型

+   常量、变量或属性的类型

+   数组、字典或其他容器中项的类型

让我们看看以下示例：

```swift
protocol ExampleProtocol {
    var simpleDescription: String { get }
    mutating func adjust()
}

// Classes, enumerations and structs can all adopt protocols.
class SimpleClass: ExampleProtocol {
    var simpleDescription: String = "A very simple class example"
    var anotherProperty: Int = 79799

    func adjust() {
        simpleDescription += " Now 100% adjusted..."
    }
}

var aSimpleClass = SimpleClass()
aSimpleClass.adjust()
let aDescription = aSimpleClass.simpleDescription

struct SimpleStructure: ExampleProtocol {
    var simpleDescription: String = "A simple struct"
    // Mutating to mark a method that modifies the structure - For classes
      we do not need to use mutating keyword
    mutating func adjust() {
        simpleDescription += " (adjusted)"
    }
}

var aSimpleStruct = SimpleStructure()
aSimpleStruct.adjust()
let aSimpleStructDescription = aSimpleStruct.simpleDescription

```

#### 协议扩展

协议扩展允许我们在协议上定义行为，而不是在每个类型的单独遵从或全局函数中定义。通过在协议上创建扩展，所有遵从的类型自动获得此方法实现，而无需任何额外修改。当我们定义协议扩展时，我们可以指定遵从类型必须满足的约束，以便在扩展的方法和属性可用时。例如，我们可以扩展我们的 `ExampleProtocol` 以提供默认功能，如下所示：

```swift
extension ExampleProtocol {
    var simpleDescription: String {
        get {
            return "The description is: \(self)"
        }
        set {
            self.simpleDescription = newValue
        }
    }

    mutating func adjust() {
        self.simpleDescription = "adjusted simple description"
    }
}

```

### 访问控制

访问控制限制了来自其他源文件和模块的代码对我们代码的访问：

+   **公共**: 这使得实体可以在定义它们的模块中的任何源文件中使用，也可以在导入定义模块的另一个模块的源文件中使用

+   **内部**: 这使得实体可以在定义它们的模块中的任何源文件中使用，但不能在模块外部的任何源文件中使用

+   **私有**: 这限制了实体只能在其定义的源文件中使用

# 概述

本章首先解释了函数式编程的重要性，然后介绍了函数式编程中的关键范式。此外，它还通过代码示例介绍了 Swift 编程语言的基础。到目前为止，我们应该对函数式编程概念和 Swift 编程语言有一个广泛的了解。本章的所有主题将在接下来的章节中详细讲解。

我们将通过函数来深入探讨这些主题，因为它们是函数式编程中最基本的构建块。因此，接下来的章节将解释函数，并给出纯函数、一等函数、高阶函数和嵌套函数的示例。此外，它还将解释一些稍微高级的话题，例如记忆化、函数柯里化和组合。
