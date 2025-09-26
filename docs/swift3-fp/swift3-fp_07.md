# 第七章. 处理可选值

在日常 Swift 应用程序开发中，我们需要处理可选值，因为我们需要调用的某些方法可能返回一些值或没有返回值。本章探讨了可选值的概念，并提供了处理它们的多种技术。

本章将通过代码示例涵盖以下主题：

+   可选类型

+   解包可选值

+   可选绑定

+   Guard

+   合并

+   可选链

+   可选映射

+   以函数式方式处理可选值

+   `fmap` 和 `apply` 用于多级函数映射

# 可选类型

在我们的日常应用程序开发中，我们会遇到期望收到值但实际没有收到值的情况。例如，假设我们有一个项目列表，我们需要在列表中搜索特定的值。我们正在寻找的特定值可能不在列表中。我们已经遇到了很多这样的场景。

其他例子可以是调用一个网络服务并接收一个没有我们所需字段的 JSON 有效负载，或者查询数据库但没有收到预期的值。

当值不存在时我们将收到什么，我们将如何处理它？

在像 C 这样的编程语言中，我们可以在不赋予其值的情况下创建一个变量。如果我们尝试在赋值之前使用该变量，我们会得到一个未定义的值。

在 Swift 中，我们可以定义一个没有值的变量，但如果我们不为其赋值，则不能使用它。换句话说，在使用之前我们需要初始化它。Swift 的这个特性确保我们不会收到未定义的值。

那么当我们需要定义一个变量但不知道其值将是什么时怎么办？

为了克服这些类型的场景，Swift 提供了 `Optional` 类型，它可以有 `Some` 或 `None` 值，并且可以在可能不存在值的情况下使用。

问号（`?`）用于定义一个可选变量。以下示例展示了可选定义的例子：

```swift
// Optional value either contains a value or contains nil
var optionalString: String? = "A String literal"
optionalString = nil

```

在这个例子中，我们定义了一个可选类型的变量。`String?` 类型是一个可选类型，它可能包含一个 `String` 值。

我们能够将 `nil` 赋值给 `optionalString`。如果我们尝试将 `nil` 赋值给 Swift 中的任何非可选类型，编译器将会对此提出警告，这与 Objective-C 等其他语言不同。

例如，在下面的例子中，编译器会抱怨 `nil cannot be assigned to type 'String'`：

```swift
var aString: String = "A String literal"
aString = nil

```

Swift 中对不存在值的编译时检查以防止运行时错误是 Swift 类型安全特性之一。类型安全使得在开发早期阶段更容易捕捉到问题。让我们通过一个 Objective-C 的例子来查看可选值的实际价值：

```swift
NSString *searchedItem = [self searchItem:@"an item"];
NSString *text = @"Found item: ";
NSString *message = [text stringByAppendingString:searchedItem];

```

假设我们有一个列表，我们通过调用 `searchItem` 来启动搜索。在这种情况下，我们的 `searchItem` 方法接受 `NSString` 并返回 `NSString`。这个调用的结果可能是 `nil`。如果我们使用返回的 `NSString` 并尝试将其追加到另一个 `NSString` 上，它将能够编译，但如果 `searchItem` 是 `nil`，则可能会使应用程序崩溃。

我们可以在使用之前检查 `searchedItem` 是否不是 `nil` 来解决这个问题。然而，可能存在一些情况，其他开发者忘记这样做或没有看到它的必要性。

当然，收到关于这类用法的编译时错误会更安全。由于 Swift 是类型安全的，我们将在运行时不会遇到任何这样的惊喜。

因此，我们已经了解了为什么我们需要可选类型，但它是什么，又是如何定义的呢？

在幕后，`Optional` 是一个包含两个情况的 `enum`——一个是 `None`，另一个是 `Some`，其关联的泛型值如下：

```swift
enum Optional<T> {
    case None
    case Some(T)
}

```

# 解包可选类型

到目前为止，我们知道可选类型会将自己包裹在内部。包裹意味着实际数据存储在外部结构中。

例如，我们这样打印 `optionalString`：

```swift
print(optionalString)

```

结果将是 `Optional("A String literal")`。

我们将如何解包可选类型并使用我们需要的值？在接下来的章节中，我们将介绍不同的解包可选类型的方法。

## 强制解包

解包可选类型最简单也是最危险的方法是强制解包。简而言之，`!` 可以用来强制从 `Optional` 中解包值。

以下示例强制解包 `optionalString`：

```swift
optionalString = "An optional String"
print(optionalString!)

```

如果可选类型没有值，强制解包可选类型可能会导致错误，因此不建议使用这种方法，因为它很难确保在不同情况下可选类型中是否有值。

实际上，强制解包消除了类型安全的好处，并可能导致我们的应用程序在运行时崩溃。

## nil 检查

强制解包一个 `Optional` 可能会使我们的应用程序崩溃；为了消除崩溃问题，我们可以在解包之前检查变量是否不是 `nil`。

以下示例展示了一种简单的 `nil` 检查方法：

```swift
if optionalString != nil {
    print(optionalString!)
}

```

这种方法在编译和运行时都是安全的，但在编辑时可能会引起问题。例如，如果我们不小心将 `print` 行移出 `if` 块之外，编译器不会抱怨，但在运行时可能会使我们的应用程序崩溃。

## Optional 绑定

更好的方法可能是使用 Optional 绑定技术来找出 `Optional` 是否包含一个值。如果它包含一个值，我们将能够解包它并将其放入一个临时常量或变量中。

以下示例展示了可选绑定：

```swift
let nilName: String? = nil
if let familyName = nilName {
    greetingfamilyName = "Hello, Mr. \(familyName)"
} else {
    // Optional does not have a value
}

```

`if let familyName = nilName` 语句将 `Optional` 值赋给一个名为 `familyName` 的新变量。赋值语句的右侧必须是一个 `Optional`，否则编译器将报错。此外，这种方法确保我们使用的是未包装的临时版本，因此是安全的。

这种方法也称为 if-let 绑定，它对于解包 `Optionals` 并访问底层值非常有用，但如果遇到复杂的嵌套对象结构，例如 `JSON` 有效负载，语法会变得繁琐。

在这种情况下，我们需要有很多嵌套的 `if-let` 表达式：

```swift
let dict = ["One": 1, "Two": 2, "Three": 3]

if let firstValue = dict["One"] {
    if let secondValue = dict["Two"] {
        if let thirdValue = dict["Three"] {
            // Do something with three values
        }
    }
}

```

为了解决这个问题，我们可以使用多个可选绑定，如下所示：

```swift
if let
firstValue = dict["One"],
secondValue = dict["Two"],
thirdValue = dict["Three"] {
    // Do something with three values
}

```

这种语法使代码更易于阅读，但当我们需要绑定多级可选类型时，这仍然不是最佳方法。在接下来的章节中，我们将探讨不同的方法来进一步提高我们处理可选类型的可读性和可维护性。

# `guard`

`guard` 是 Swift 库中提供的另一种处理 `Optionals` 的方法。`guard` 方法与 `Optional if-let` 绑定不同，因为 `guard` 语句可以用于早期退出。我们可以使用 `guard` 语句要求在 `guard` 语句之后的代码执行之前，条件必须为 `true`。

以下示例展示了 `guard` 语句的使用：

```swift
func greet(person: [String: String]) {
    guard let name = person["name"] else {
        return
    }
    print("Hello Ms \(name)!")
}

greet(person: ["name": "Neco"]) // prints "Hello Ms Neco!"

```

在这个例子中，`greet` 函数需要一个表示人名的值；因此，它使用 `guard` 语句检查其是否存在。否则，它将返回并停止执行。

使用 `guard` 语句，我们可以首先检查失败场景，并在失败时返回。与 `if-let` 语句不同，`guard` 不提供新的作用域，因此在前面的例子中，我们能够在 `{ }` 之外使用 `name` 在我们的 `print` 语句中，这是不可能的。

与 `if-let` 语句类似，我们可以使用多个 `guard` 语句，如下所示：

```swift
func extractValue(dict: [String: Int]) {
    guard let
    firstValue = dict["One"],
    secondValue = dict["Two"],
    thirdValue = dict["Three"]
    else {
        return
    }
    // Do something with three values
} 

```

# 隐式解包的可选类型

我们可以通过在类型末尾附加一个感叹号 (`!`) 来定义隐式解包的可选类型。这些类型会自动解包。

以下示例展示了从字典中获取值两种方式。在第一个例子中，结果值将是一个可选类型。在第二个例子中，值将隐式解包：

```swift
let one = "One" 
let firstValue = dict["One"] 
let implictlyUnwrappedFirstValue: Int! = dict["One"]

```

与强制解包类似，隐式解包的可选类型可能会导致我们的应用程序在运行时崩溃，因此在使用它们时我们需要谨慎。

# 错误处理以避免可选类型

在我们的日常应用开发中，我们可能需要开发一些返回 `Optionals` 的函数。例如，假设我们需要读取一个文件并返回该文件的内容。使用可选类型，我们可以这样开发：

```swift
func checkForPath(path: String) -> String? {
    // check for the path
    return "path"
}

func readFile(path: String) -> String? {
    if let restult = checkForPath(path: path) {
        return restult
    } else {
        return nil
    }
}

```

这里，`checkForPath` 是一个不完整的函数，用于检查文件是否存在。

当我们调用 `readFile` 函数时，我们需要检查结果的可选类型：

```swift
if let result = readFile(path: "path/to") {
    // Do something with result
}

```

在这种情况下，我们不是使用可选值，而是可以使用错误处理来重定向控制流以消除错误并提供恢复：

```swift
enum Result: ErrorProtocol {
    case failure
    case success
}

func readFile(path: String) throws -> String {
    if let restult = checkForPath(path: path) {
        return restult
    } else {
        throw Result.failure
    }
}

```

当我们调用这个函数时，我们需要将其包裹在一个`do`块中并`catch`到`exception`：

```swift
do {
    let result = try readFile(path: "path/to")
} catch {
    print(error)
}

```

## try!

如果我们知道方法调用不可能失败，或者如果它失败则我们的代码将损坏，我们应该崩溃应用程序，我们可以使用`try!`。

当我们使用`try!`关键字时，我们不需要在代码块周围有`do`和`catch`，因为我们承诺它永远不会失败！这是一个很大的承诺，我们应该避免。

如果我们必须绕过错误处理，例如检查数据库文件是否存在，我们可以这样做：

```swift
do {
    let result = try readFile(path: "path/to")
} catch {
    print(error)
}

```

## try?

我们可以使用`try?`将错误转换为`Optional`值来处理错误。

如果在评估`try?`表达式时抛出错误，则表达式的值将是`nil`。例如，在以下示例中，如果我们无法读取文件，则结果将是`nil`：

```swift
let result = try? readFile(path: "path/to")

```

# 空值合并

Swift 提供了`??`运算符用于空值合并。它解包`Optionals`并为`nil`情况提供回退或默认值。例如，`a ?? b`如果`a`有值则解包可选`a`，如果`a`是`nil`则返回默认值`b`。

在这个例子中，如果`Optional` `a`不是`nil`，则空值合并运算符后面的表达式将不会被评估。空值合并适用于我们可以提供回退或默认值的情况。

# 可选链

可选链是一个查询和调用可能当前为`nil`的可选属性、方法和下标的进程。Swift 中的可选链类似于 Objective-C 中的`nil`消息，但以一种适用于任何类型并且可以检查成功或失败的方式工作。

以下示例展示了两个不同的类。其中一个类`Person`有一个类型为`Optional`（`residence`）的属性，它包装了另一个类类型`Residence`：

```swift
class Residence {
    var numberOfRooms = 1
}

class Person {
    var residence: Residence?
}

```

我们将创建一个`Person`类的实例，名为`sangeeth`：

```swift
let residence = Residence()
residence.numberOfRooms = 5
let sangeeth = Person()
sangeeth.residence = residence

```

要检查`numberOfRooms`，我们需要使用`Person`类的居住属性，它是一个可选值。可选链使我们能够如下遍历可选值：

```swift
if let roomCount = sangeeth.residence?.numberOfRooms {
    // Use the roomCount
    print(roomCount)
}

```

`roomCount`变量将如预期的那样是五个。

这可以用来通过可选链调用方法和下标。

我们可以通过将问号替换为感叹号来强制解包任何链项：

```swift
let roomCount = sangeeth.residence!.numberOfRooms

```

再次提醒，当我们使用强制解包时，我们需要谨慎。

# 函数式处理可选值

到目前为止，我们已经介绍了许多不同的方法和工具来处理可选值。让我们看看我们是否可以使用函数式编程范式来简化这个过程。

## 可选映射

对数组进行映射将为数组中的每个元素生成一个元素。我们能否映射可选以生成非可选值？如果我们有`Some`，则映射它；否则，返回`None`。让我们看看这个例子：

```swift
func mapOptionals<T, V>(transform: (T) -> V, input: T?) -> V? {
    switch input {
        case .some(let value): return transform(value)
        case .none: return .none
    }
}

```

我们的`input`变量是一个泛型`optional`，我们有一个转换函数，它接受`input`并将其转换为泛型类型。最终结果将是一个泛型`optional`类型。在函数体中，我们使用模式匹配来返回相应的值。让我们测试这个函数：

```swift
class User {
    var name: String?
}

```

我们创建了一个名为`User`的虚拟类，其中包含一个`Optional`变量。我们如下使用该变量：

```swift
func extractUserName(name: String) -> String {
    return "\(name)"
}

var nonOptionalUserName: String {
    let user = User()
    user.name = "John Doe"
    let someUserName = mapOptionals(transform: extractUserName,
      input: user.name)
    return someUserName ?? ""
}

```

最终结果将是一个非可选的 String。我们的`mapOptionals`函数与 Haskell 中的`fmap`函数类似，它定义为`<^>`运算符。

让我们将这个函数转换为运算符：

```swift
infix operator <^> { associativity left }

func <^><T, V>(transform: (T) -> V, input: T?) -> V? {
    switch input {
        case .some(let value): return transform(value)
        case .none: return .none
    }
}

```

在这里，我们只定义了一个中缀运算符并定义了相应的函数。让我们尝试这个函数，看看它是否提供相同的结果：

```swift
var nonOptionalUserName: String {
    let user = User()
    user.name = "John Doe"
    let someUserName = extractUserName <^> user.name
    return someUserName ?? ""
}

```

结果与我们的上一个例子相同，但代码更易读，所以我们可能更喜欢使用它。

## 多个可选值映射

我们之前的例子展示了使用函数式编程技术对单个`optional`值进行映射。如果我们需要一起映射多个可选值怎么办？在*Optional 绑定*部分，我们介绍了处理多个`Optional`值绑定的一种非函数式方法，在本节中，我们将探讨多个可选值映射。由于可选值是*applicative functors*的实例，我们将开发一个`apply`函数来在可选值上使用：

```swift
func apply<T, V>(transform: ((T) -> V)?, input: T?) -> V? {
    switch transform {
        case .some(let fx): return fx <^> input
        case .none: return .none
    }
}

```

`apply`函数与`fmap`函数非常相似。`transform`函数是可选的，我们使用模式匹配来返回`none`或`some`。

在 Haskell 中，`apply`函数表示为`<*>`运算符。这个运算符已经被 Swift 函数式编程社区采用，因此我们将其用作`apply`函数：

```swift
infix operator <*> { associativity left }

func <*><T, V>(transform: ((T) -> V)?, input: T?) -> V? {
    switch transform {
        case .some(let fx): return fx <^> input
        case .none: return .none
    }
}

```

我们可以如下测试我们的`apply`函数：

```swift
func extractFullUserName(firstName: String)(lastName: String) -> String {
    return "\(firstName) \(lastName)"
}

```

`extractFullUserName`函数是一个柯里化函数，应该显式地转换为返回闭包，因为 Apple 在 Swift 2.2 中弃用了函数柯里化，并在 Swift 3.0 中将其移除。

让我们将其转换为 Swift 3.0 版本：

```swift
func extractFullUserName(firstName: String) -> (String) -> String { 
    return { (lastName: String) -> String in
        return "\(firstName) \(lastName)"
    }
}

```

现在我们可以使用这个函数来提取完整的用户名：

```swift
class User { 
    var firstName: String? 
    var lastName: String? 
} 
var fullName: String { 
    let user = User() 
    user.firstName = "John" 
    user.lastName = "Doe"
    let fullUserName = extractFullUserName <^> user.firstName <*> 
      user.lastName
    return fullUserName ?? "" 
}

```

通过组合`fmap`和`apply`函数，我们能够`map`两个可选值。

事实上，`Optional`类型是一个`monad`，因此它实现了`map`和`flatMap`方法，我们不需要自己开发它。

以下示例展示了在`optional`类型上调用`map`方法：

```swift
let optionalString: String? = "A String literal" 
let result = optionalString.map { "\($0) is mapped" }

```

结果将是一个包含以下值的`Optional String`：

`A String literal is mapped`。

此外，我们可以使用`flatMap`来过滤`nil`值，并将可选值数组转换为未包装值数组。

在以下示例中，在`optional Array`上调用`flatMap`将消除我们的`Array`中的第三个元素（索引：2）：

```swift
let optionalArray: [String?] = ["First", "Second", nil, "Fourth"] 
let nonOptionalArray = optionalArray.flatMap { $0 } 
print(nonOptionalArray) 

```

结果将是`["First", "Second", "Fourth"]`。

# 摘要

在本章中，我们熟悉了处理可选值的不同技术。我们讨论了处理可选值的内置技术，例如可选绑定、守卫（guard）、合并（coalescing）和可选链（optional chaining）。然后我们探讨了函数式编程技术来处理可选值。我们创建了`fmap`和`apply`函数以及相关的运算符来解决多个可选绑定问题。尽管一些开发者可能更喜欢使用内置的多个可选绑定，但实际探索函数式编程技术可以更好地理解我们将能够应用于其他问题的概念。

在接下来的章节中，我们将探讨一些函数式数据结构的例子，例如半群（Semigroup）、幺半群（Monoid）、二叉搜索树（Binary Search Tree）、链表（Linked List）、栈（Stack）和惰性列表（Lazy List）。
