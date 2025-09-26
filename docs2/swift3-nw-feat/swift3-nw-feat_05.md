# 第五章. 函数和运算符变化 – 以新的方式完成任务

如果你想在 Swift 中编写有用的代码，或者在任何编程语言中，你至少需要创建函数和使用运算符。在本章中，我们将探讨函数声明和使用方面的变化，以及这些变化如何转化为更好的 Swift 代码。我们还将解释运算符的变化，并突出一些已被从语言中移除的运算符。

继续上一章的主题，我将提供 Swift 进化提案编号。让我们开始吧！

# 函数声明变化

Swift 提供了一套非常灵活的规则来定义函数。你可以创建没有参数、有参数，甚至有参数标签的函数。每个 Swift 函数都有一个类型、参数（或没有参数）以及返回类型。对于 Swift 3，语言经过调整以使事物更加一致和简单。

## 一致的参数命名 [SE-0046]

参数命名用于在函数定义中为每个参数命名。在 Swift 2.2 及更早版本中，函数参数可以同时定义本地和外部标签。本地参数标签是必需的，因为这个标签用于在函数体中引用参数。当提供外部参数标签时，它用于实际的函数调用中。你可以将外部标签视为在调用位置的*闪亮的描述性名称*，以提供对参数表示的深入了解。内部标签，正如其名称所暗示的，是函数在逻辑实现中使用的名称。由于没有人会看到这个本地参数，你可以将其缩短以节省一些按键。

这里事情变得有趣了。默认情况下，Swift 2.2 在调用函数时会丢弃你的外部名称。进一步增加混乱的是，当外部名称不存在时，Swift 使用你的本地名称作为任何剩余参数的外部名称。真正奇怪的是，Swift 只对函数这样做。当你创建一个类、结构体或枚举初始化器（一种设置初始值的特殊类型的函数）时，Swift 将为每个参数创建一个外部名称。

### 注意

Swift 枚举可以使用原始值类型定义。当你使用原始值创建 Swift 枚举时，你将得到一个初始化的枚举或 nil。你可以在 [`developer.apple.com/library/prerelease/content/documentation/Swift/Conceptual/Swift_Programming_Language/Enumerations.html#//apple_ref/doc/uid/TP40014097-CH12-ID145`](https://developer.apple.com/library/prerelease/content/documentation/Swift/Conceptual/Swift_Programming_Language/Enumerations.html#//apple_ref/doc/uid/TP40014097-CH12-ID145) 上了解更多关于 Swift 枚举的信息。

删除第一个参数名称的主要原因似乎是为了与 Objective-C 保持历史正确性。Objective-C 开发者被指示将第一个参数名称纳入函数名称中。Swift 也采用了这种行为，这可能是通过 migrator 将 Objective-C 库最初翻译成 Swift 的副作用。

Swift 是一种不断发展的语言，并且随着每个新版本的发布而变得越来越好。为了与新的 API 命名指南保持一致，Swift 3 现在默认使用局部名称作为所有参数的外部名称，包括第一个参数。

在 Swift 2.2 中：

```swift
func gettingSwiftyUsingPeopleNamed(names: [String],  
                                   descriptionNames: [String],    
                                   shouldCapitalize: Bool){ 
    // our function only has local names so those are the ones used 
} 

gettingSwiftyUsingPeopleNamed(["Joe", "Mark", "Roy", "Jessica"], 
           descriptionNames: ["Awesome", "Silly", "Tall", "Short"], 
           shouldCapitalize: true) 

```

在 Swift 3 中：

```swift
func gettingSwiftyUsing(names: [String],  
                         descriptions:[String],  
                         shouldCapitalize: Bool){ 
    // our function only has local names so those are the ones used 
} 

gettingSwiftyUsing(names: ["Joe", "Mark", "Roy", "Jessica"],  
descriptions: ["Awesome", "Silly", "Tall", "Short"],  
shouldCapitalize: true) 

```

### 注意

最后的想法——如果您在函数调用中不喜欢使用参数标签，您可以使用下划线作为外部名称来抑制它们。这对于任何参数位置都适用。

```swift
func boxit(_ width: Double, _ height: Double){ 
    // argument label will be omitted in function call 
} 

boxit(23, 14)
```

### 注意

您可以在以下链接中阅读提案 [`github.com/apple/swift-evolution/blob/master/proposals/0046-first-label.md`](https://github.com/apple/swift-evolution/blob/master/proposals/0046-first-label.md)

### 删除声明中的柯里化函数语法 [SE0002]

在 Swift 2 中，您有创建柯里化函数的能力；这些函数相当令人困惑，在 Swift 中似乎也没有多少价值。许多开发者在使用它们时产生的困惑主要集中在柯里化参数是否是主参数列表的一部分，或者柯里化参数是否表示新函数参数列表的开始。让我们考虑以下使用柯里化参数的示例：

```swift
func curried(cosx: Int)(siny: Int) -> Float { 
    return (Float(cosx) * Float(siny)) / Float(cosx) 
} 

let result = curried(2)(siny:3) 

```

判断 `(siny: Int)-> Float` 是否是参数的一部分非常令人困惑。Swift 团队最终决定我们根本不需要这种语法。因此，Swift 3 删除了这种语法，并建议您将函数重写为返回闭包：

```swift
func curriedV2(cosx: Int)->(Int)->Float{ 
    return { siny in 
        (Float(cosx) * Float(siny)) / Float(cosx) 
    } 
} 

let intermediateFunctionReturn = curriedV2(2) 
let result2 = intermediateFunctionReturn(3) 

```

在我们修订的示例中，我们定义了一个返回闭包的函数。然后，我们将这个初始调用的结果赋值给名为 `intermediateFunctionReturn` 的变量。最后，我们调用 `intermediateFunctionReturn` 闭包，传递我们的 Int 参数，以获取最终结果。

### 注意

您可以在以下链接中阅读提案 [`github.com/apple/swift-evolution/blob/master/proposals/0002-remove-currying.md`](https://github.com/apple/swift-evolution/blob/master/proposals/0002-remove-currying.md)

## 默认情况下对未使用结果的警告 [SE-0047]

属性是应用于声明或类型的特殊构造。您使用 `@` 符号后跟一个名称以及可选的任何属性参数（括号内）来指定属性。

```swift
@<attribute name> 
@<attribute name>(attribute arguments) 

```

在 Swift 2 中，`@warn_unused_result` 属性应用于函数或方法，以让编译器知道如果未使用结果就调用属性或方法，则应向用户发出警告。`@warn_unused_result` 属性还允许你提供消息或 `mutable_variant` 属性参数。当你想让开发者知道完成相同任务的 mutating 方法的名称时，使用 `mutable_variant` 选项。使用该属性的目的在于为使用你的方法的开发者提供指导，即返回值很重要，应该使用。

例如，Swift 2 为 Foundation 集合类提供了 `sort()` 方法和一个 `sortInPlace()` 方法（mutating）。如果你对一个数组调用 `sort()`（非 mutating）但没有使用结果，编译器会警告你可能真的需要 `sortInPlace()` 方法，该方法会修改数组且不返回任何内容。下面你可以看到 `@warn_unused_result` 的一个示例用法。

```swift
@warn_unused_result(mutable_variant="sortInPlace") 
    public func sort() -> [Self.Generator.Element] 

```

开发者对 `@warn_unused_result` 的主要问题是，它只有在将属性应用于所有相关位置时才有所帮助（即这是你主动采取的策略）。如果你忘记添加属性，则不会发出警告。在 Swift 3 中，逻辑相反，现在默认情况下你会收到关于未使用结果的警告。如果你想明确地让编译器知道返回值可以安全地忽略，你可以使用 `@discardableResult` 属性。下面是一个演示其用法的示例。

```swift
@discardableResult 
func complexFunctionNonEssentialResult()->Int{ 
    // do complex logic 
    // return trivial status code 
    return 123 
} 

```

### 注意

在 Swift 3 中，mutating 方法 `sortInPlace()` 变成了 `sort()`，而非 mutating 版本则从 `sort()` 变成了 `sorted()`。

### 注意

你可以在 [`github.com/apple/swift-evolution/blob/master/proposals/0047-nonvoid-warn.md`](https://github.com/apple/swift-evolution/blob/master/proposals/0047-nonvoid-warn.md) 上阅读更多关于该提案的信息。

### 从函数参数列表中移除 var [SE-0003]

在 Swift 2 中，在函数参数列表中使用 var 是有效的。由于参数类型默认是不可变的，开发者尝试在参数列表中使用 var 来向函数实现传递一个可变参数。虽然你可以在 Swift 中这样做，但实际上这证明是一个相当无用的策略。让我来解释一下。是的，你可以使用 var 关键字将一个可变变体传递到函数体中。然而，你对变量所做的任何更改都不会传播回原始类型。因此，你使用的是一个作用域限于函数体的可变副本。让我们通过一个示例来查看这在实践中是如何工作的：

```swift
func booyah(howHigh: Int){ 
    howHigh += 100 // -> illegal assignment 
} 

func booyahTake2(var yaFeelMe: Int){ 
    yaFeelMe += 100   // -> legal but doesn't write back to caller 
} 

```

你同样可以轻松地通过在函数体内部将传入的参数赋值给一个局部副本来实现相同的目标。下面是等效的函数定义：

```swift
func booyahTake3(yaFeelMe: Int){ 
    var yaFeelMe = yaFeelMe 
    yaFeelMe += 100 
} 

```

最后，许多开发者会将 `var` 参数与标记为 `inout` 的参数混淆。两种版本都会提供一个可变的局部副本，但只有 `inout` 变体才会将更改传播回函数调用者。鉴于整体上的混淆，Swift 3 中已经移除了 `var` 作为参数修饰符。在以下示例中，`howManyTimes` 变量通过将其值传播回函数调用者而更新，因为它被标记为 `inout` 参数：

```swift
func trifecta(inout howManyTimes: Int){ 
    howManyTimes += 2000  // updates the actual passed in variable 
} 

```

### 注意

你可以在这里阅读提案：[`github.com/apple/swift-evolution/blob/master/proposals/0003-remove-var-parameters.md`](https://github.com/apple/swift-evolution/blob/master/proposals/0003-remove-var-parameters.md)

### 移除 `++` 和 `--` 运算符 [SE-0004]

增量（`++`）和减少（`--`）运算符被添加到 Swift 中，因为它们在 C 语言中存在。此外，许多转向 Swift 的开发者习惯于在其他语言中看到它们。让我们来检查这些运算符是如何工作的，然后我们可以讨论一些需要注意的问题。

```swift
var row = 0 
let currentRow = ++row   // pre - adds 1 to row than assigns new value 
let nextRow = row++       // post - assigns than adds 1 from row  
let previousRow = --row  // pre - minus 1 from row than assigns to value 
let backOneRow = row--   // post - assigns than subtracts 1 from row 

```

不利之处/需要注意的问题：

+   很容易弄错增量/减量运算符的前置和后置部分，这会导致你得到错误的结果

+   语法是 `+=` 或 `-=` 的简写，这并没有节省你多少按键，与 `+= 1` 或 `-= 1` 相比

+   这些运算符主要在 C 风格的 `for…in` 循环中使用，其中返回值被忽略。由于我们已经有 `for…in` 循环、范围、映射以及 `enumerate/iterate` 函数，我们不需要这些运算符

+   你只能使用这些运算符与一组有限的类型（例如整数和浮点标量）一起使用

+   如果你正在学习你的第一门编程语言，这些运算符会增加你需要学习的内容量，而没有提供通过 Swift 提供的其他功能无法获得的有意义的价值

实际上，Swift 团队认为保留这些运算符的缺点超过了优点，因此决定在 Swift 3 中移除它们。

### 注意

你可以在这里阅读提案：[`github.com/apple/swift-evolution/blob/master/proposals/0004-remove-pre-post-inc-decrement.md`](https://github.com/apple/swift-evolution/blob/master/proposals/0004-remove-pre-post-inc-decrement.md)

### 移除 C 风格的 `for` 循环 [SE-0007]

与我们刚才讨论的增量/减量运算符类似，C 风格的 `for…in` 循环似乎也是为了其 C 语言血统而被添加到 Swift 中的。Swift 提供了几个比 C 风格循环更好的 *Swifty* 迭代和循环约定。事实上，C 风格的循环并不常用。一旦开发者开始精通 Swift 概念，他们通常会选择不使用 C 风格的循环。在考虑集合的迭代时，`for…in` 循环比 `for…in` 语句更难实现。最后，如果 C 风格的 `for` 循环在 Swift 中原本就不存在，没有人会想念它们或请求将它们包含到语言中。在 Swift 3 中，C 风格的 `for` 循环正式从语言中移除。

### 注意

你可以在 [`github.com/apple/swift-evolution/blob/master/proposals/0007-remove-c-style-for-loops.md`](https://github.com/apple/swift-evolution/blob/master/proposals/0007-remove-c-style-for-loops.md) 阅读更多关于该提案的信息。

### 从函数中移除隐式元组展开 [SE-0029]

在 Swift 的早期版本中，函数调用允许开发者以元组的形式传递参数列表，这通常被称为 *元组展开*。一个 *元组展开* 可以在某个地方定义，然后作为一个对象传递给函数，从而无需将单个参数传递给函数。让我们通过一个例子来更清楚地说明这个概念。

```swift
func fooTastic(members: [String], instruments:[String]){ 
    // fantastic work happening here... 
} 

let foo = (["Jackson", "Carey", "Wonderland"], instruments:["drums", "bass", "keyboard"]) 
fooTastic(foo) 

```

在我们的示例中，我们创建了 `fooTastic()` 函数来接受两个 String 数组参数。然后我们创建了一个封装我们想要传递给函数的参数的元组。最后，我们调用 `fooTastic()` 并传递我们的 foo 元组。这可以工作，但这里有一些需要考虑的缺点：

+   我们的 `foo` 元组必须与函数中参数表示的方式相匹配：这意味着我们必须去掉成员参数标签。我们必须在我们的元组中包含乐器标签，否则当我们将元组作为参数传递时，编译器会报错。

+   只传递一个元组给函数会使我们的方法看起来有重载，这对负责维护此代码的人来说很困惑。

+   元组展开的当前实现不一致且存在错误。

考虑到所有这些因素，我们根本不需要在语言中添加这种额外的复杂性，因此这个特性已被从 Swift 3 中移除。

### 注意

你可以在 [`github.com/apple/swift-evolution/blob/master/proposals/0029-remove-implicit-tuple-splat.md`](https://github.com/apple/swift-evolution/blob/master/proposals/0029-remove-implicit-tuple-splat.md) 阅读更多关于该提案的信息。

### 调整 `inout` 声明以进行类型装饰 [SE-0031]

这是对 Swift 3 的一个微小改动。`inout` 关键字已经被移动到冒号（:）的右边，并在函数定义中紧邻类型。关于 `inout` 变量在代码中的行为没有任何改变。你仍然在函数体内赋予带有 `inout` 装饰的参数修改值的能力。这个改动是为了让装饰名称更靠近它实际修改的类型。由于我们修改的是类型而不是标签，所以将关键字放在类型旁边更有意义。

在 Swift 2 中：

```swift
func trifecta(inout howManyTimes: Int){ 
    howManyTimes += 2000 
} 

```

在 Swift 3 中：

```swift
func trifecta(howManyTimes: inout Int){ 
    howManyTimes += 2000 
} 

```

### 注意

你可以在 [`github.com/apple/swift-evolution/blob/master/proposals/0031-adjusting-inout-declarations.md`](https://github.com/apple/swift-evolution/blob/master/proposals/0031-adjusting-inout-declarations.md) 阅读该提案。

### 将属性参数中的等号替换为冒号 [SE-0040]

正如我们在 *[SE-0047]* 中讨论的那样，属性是应用于声明或类型的特殊构造。你使用 `@` 符号后跟一个名称来指定属性，并可选地使用括号中的任何属性参数：

```swift
@<attribute name> 
@<attribute name>(attribute arguments) 

```

与常规函数和参数标签不同，你使用 `=` 而不是冒号来分隔参数名和它的值。这与 Swift 中其他标准操作模式不一致。因此，在 Swift 3 中，属性参数将与其他 Swift 参数一样接受相同的处理，使用冒号而不是等于号。

在 Swift 2 中：

```swift
@warn_unused_result(mutable_variant="sortInPlace") 
    public func sort() -> [Self.Generator.Element] 

```

在 Swift 3：

```swift
@available(*, deprecated, renamed: "NSUnderlyingErrorKey") 
    public static let underlyingErrorKey: ErrorUserInfoKey 

```

### 注意

你可以在[`github.com/apple/swift-evolution/blob/master/proposals/0040-attributecolons.md`](https://github.com/apple/swift-evolution/blob/master/proposals/0040-attributecolons.md)阅读这个提议。

### 标准化函数类型参数语法以要求使用括号 [SE-0066]

在 Swift 中定义函数时，使用括号来包围它们的参数列表。其目的是使函数声明开始和结束的位置清晰。然而，Swift 2 允许你在某些条件下调用函数而不使用括号。让我们通过一个例子来使这个概念更清晰。在下面的例子中，我们使用括号和它们的等效方式（省略括号）来定义函数：

```swift
let a: (Int) -> Int 
let b: (Int) -> (Int)-> Int 

```

这也可以写成不带括号的形式：

```swift
let a1: Int -> Int 
var b1: Int -> Int -> Int 

```

当然，第二种形式由于省略了括号而略短。然而，这种权衡引入了与语言中其他函数类型定义方式不一致的代码。坦白说，省略括号并没有带来任何真正的价值或表达性组件。以这种方式构建函数类型只是语法糖，没有实质内容。在 Swift 3 中，你将无法在定义函数类型时使用这种快捷形式。

### 注意

你可以在[`github.com/apple/swift-evolution/blob/master/proposals/0066-standardize-function-type-syntax.md`](https://github.com/apple/swift-evolution/blob/master/proposals/0066-standardize-function-type-syntax.md)了解更多关于这个提议的信息。

### 强制默认参数的顺序 [SE-0060]

在 Swift 中调用函数时，顺序很重要，但对于包含默认参数的函数来说，有一个例外。在这种情况下，你可以只使用部分参数名称来调用这类函数。让我们看看一些如何调用包含默认参数的函数的例子。

在 Swift 2 中：

```swift
func shifty(arg1: String = "", arg2: String = "", arg3: Int = 1){} 

```

调用 `shifty()` 函数的第一种方式是使用所有默认参数，这意味着在调用位置不传递任何内容。这在 Swift 2 中是有效的，并且通常是预期的行为。下面是一个仅使用具有所有默认参数值的函数的例子：

```swift
shifty() 

```

另一种调用 `shifty()` 的方式是省略我们不关心的参数，只传递我们关心的参数。我们可以只传递一个参数，比如 arg2 或 arg3，而我们的函数仍然可以正常工作。在下面的例子中，我们展示了在省略一些参数的情况下调用我们的 shifty 函数：

```swift
shifty(arg2: "") 
shifty(arg3: 3) 

```

最后，我们可以使用多个参数来调用函数。以下是一个使用多个参数调用 shifty 函数的示例用法：

```swift
shifty(arg2: "", arg3: 3) 

shifty("", arg3: 3) 

shifty(arg2: "", arg3: 4) 

```

允许这种行为实际上会让开发者感到有些困惑，并且与语言其他部分强制执行的严格顺序相矛盾。Swift 3 移除了这种行为，并强制您在使用默认参数时保持参数顺序。让我们看看我们的 `shifty()` 函数在 Swift 3 中的工作方式。

在 Swift 3 中：

```swift
func shifty(arg1: String = "arg1", arg2: String = "arg2", arg3: Int = 0){} 

```

使用所有默认参数调用函数的方式没有变化：

```swift
shifty() 

```

当我们用一个参数调用函数时，我们必须使用参数标签。另一个区别是我们不能随意选择调用参数的顺序。我们可以省略默认参数，但不能随意调用它们。让我们看看在 Swift 3 中我们可以如何调用我们的 `shifty()` 函数。

```swift
shifty(arg1: "") // valid 
shifty(arg2: "") // valid 
   shifty(arg3: 3)   //valid 
   shifty(arg3: 3, arg1: "test")   //invalid! 

```

如果您仔细思考一下，我敢打赌您能看出 Swift 团队做出这一改变的原因。我们牺牲了更短的语法以增强可读性。

### 注意

您可以在[`github.com/apple/swift-evolution/blob/master/proposals/0060-defaulted-parameter-order.md`](https://github.com/apple/swift-evolution/blob/master/proposals/0060-defaulted-parameter-order.md)阅读该提案

### 将可选要求仅限于 Objective-C [SE-0070]

Objective-C 协议有一个功能，允许开发者标记一些行为作为可选的。虽然这对于 Objective-C 来说是合理的，但它不会作为 Swift 功能有意义。在 Swift 中，协议作者可以使用协议扩展和协议继承提供默认实现。同样，使用协议继承，作者可以将可选方法添加到单独的协议中，开发者可以选择采用该协议以实现可选行为，而不是将其作为所有协议用户的必需要求。主要收获是，在 Swift 中处理可选协议要求有更好的选择，因此，在协议上添加 Objective-C 可选功能对 Swift 来说不是必要的。

既然我们知道为什么 Swift 团队选择不将此作为 Swift 功能包含在内，让我们讨论如何在 Swift 3 中处理 Objective-C 可选参数。基本上，我们使用 `@objc` 属性来装饰我们想要区分的作为 Objective-C 仅需要求的协议部分。我们还为每个函数签名添加了可选关键字。在大多数情况下，您不需要在 Swift 中进行任何更改。您也不必修改您的 Objective-C 代码。迁移器会为您处理所有工作。

在 Objective-C 中：

```swift
@protocol UICollectionViewDataSource <NSObject> 
@required 

- (NSInteger)collectionView:(UICollectionView *)collectionView numberOfItemsInSection:(NSInteger)section; 

- (UICollectionViewCell *)collectionView:(UICollectionView *)collectionView cellForItemAtIndexPath:(NSIndexPath *)indexPath; 

@optional 

- (NSInteger)numberOfSectionsInCollectionView:(UICollectionView *)collectionView; 

- (UICollectionReusableView *)collectionView:(UICollectionView *)collectionView viewForSupplementaryElementOfKind:(NSString *)kind atIndexPath:(NSIndexPath *)indexPath; 

- (BOOL)collectionView:(UICollectionView *)collectionView canMoveItemAtIndexPath:(NSIndexPath *)indexPath; 

- (void)collectionView:(UICollectionView *)collectionView moveItemAtIndexPath:(NSIndexPath *)sourceIndexPath toIndexPath:(NSIndexPath*)destinationIndexPath; 

@end 

Swift 
public protocol UICollectionViewDataSource : NSObjectProtocol { 

public func collectionView(_ collectionView: UICollectionView, numberOfItemsInSection section: Int) -> Int 

public func collectionView(_ collectionView: UICollectionView, cellForItemAt indexPath: IndexPath) -> UICollectionViewCell 

optional public func numberOfSections(in collectionView: UICollectionView) -> Int 

optional public func collectionView(_ collectionView: UICollectionView, viewForSupplementaryElementOfKind kind: String, at indexPath: IndexPath) -> UICollectionReusableView 

optional public func collectionView(_ collectionView: UICollectionView, canMoveItemAt indexPath: IndexPath) -> Bool 

optional public func collectionView(_ collectionView: UICollectionView, moveItemAt sourceIndexPath: IndexPath, to destinationIndexPath: IndexPath) 
}
```

我们上面的代码片段显示了 Objective-C 协议（带有可选方法）以及通过迁移器转换后的 Swift 版本。

### 注意

您可以在[`github.com/apple/swift-evolution/blob/master/proposals/0070-optional-requirements.md`](https://github.com/apple/swift-evolution/blob/master/proposals/0070-optional-requirements.md)阅读该提案

# 摘要

在本章中，我们介绍了函数的创建和调用方法。我们提到了一些不太**Swift**的特性，这些特性在 Swift 3 中被移除了。我们还探讨了属性和属性参数，重点关注语法变化，以及语言中的新增和删除功能。在下一章中，我们将深入探讨闭包和集合。
