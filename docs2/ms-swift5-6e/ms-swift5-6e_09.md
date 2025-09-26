

# 协议和协议扩展

当我观看 2015 年 WWDC 关于协议扩展和**面向协议编程**（**POP**）的演示时，我必须承认我非常怀疑。我长期从事**面向对象编程**（**OOP**），因此我不确定这种新的编程范式是否能够解决苹果所声称的所有问题。既然我不是一个让怀疑阻碍尝试新事物的人，我就设置了一个新的项目，该项目与我现在正在工作的项目相似，但使用苹果对 POP 的建议编写代码。我还广泛使用了协议扩展。我可以真诚地说，与原始项目相比，新项目要干净得多。我相信，协议扩展将是那些定义性特征之一，将一种编程语言与其他编程语言区分开来。

在本章中，你将学习以下主题：

+   协议如何用作类型？

+   我们如何使用协议在 Swift 中实现多态？

+   我们如何使用协议扩展？

+   为什么我们想要使用协议扩展？

虽然协议扩展基本上是语法糖，但在我看来，它们是 Swift 编程语言中最重要的一些新增功能之一。通过协议扩展，我们能够为符合协议的任何类型提供方法和属性实现。为了真正理解协议和协议扩展的有用性，让我们更好地了解协议。

虽然 Swift 中的类、结构和枚举都可以遵守协议，但在这个章节中，我们将专注于类和结构。枚举用于表示有限数量的案例，虽然在我的经验中，枚举遵守协议的有效用例是存在的，但它们很少见。只需记住，在任何我们提到类或结构的地方，我们也可以使用枚举。

让我们从了解它们在 Swift 中是如何作为完整类型开始的，来探索协议。

# 协议作为类型

尽管在协议中没有实现任何功能，但它们在 Swift 编程语言中仍然被视为完整类型，可以像任何其他类型一样使用。这意味着我们可以将协议用作函数的参数类型或返回类型。

我们还可以将它们用作变量、常量和集合的类型。让我们看看一些例子。对于这些例子，我们将使用以下`PersonProtocol`协议：

```swift
protocol PersonProtocol {
    var firstName: String { get set } 
    var lastName: String { get set } 
    var birthDate: Date { get set } 
    var profession: String { get }
init(firstName: String,lastName: String, birthDate: Date)
} 
```

在这个第一个例子中，协议被用作函数的参数类型和返回类型： 

```swift
func updatePerson(person: PersonProtocol) -> PersonProtocol {
    // Code to update person goes here 
return person
} 
```

在这个例子中，`updatePerson()`函数接受一个`PersonProtocol`协议类型的参数，并返回一个`PersonProtocol`协议类型的值。接下来的例子展示了如何将协议用作常量、变量或属性的类型：

```swift
var myPerson: PersonProtocol 
```

在这个例子中，我们创建了一个名为 `myPerson` 的 `PersonProtocol` 协议类型的变量。协议也可以用作存储集合的项目类型，例如数组、字典或集合：

```swift
var people: [PersonProtocol] = [] 
```

在这个最后的例子中，我们创建了一个 `PersonProtocol` 协议类型的数组。尽管 `PersonProtocol` 协议没有实现任何功能，但我们仍然可以在需要指定类型时使用协议。然而，协议不能像类或结构体那样被实例化。这是因为协议中没有实现任何功能。例如，当尝试创建 `PersonProtocol` 协议的实例时，如下面的例子所示，我们会收到编译时错误：

```swift
var test = PersonProtocol(firstName: "Jon", lastName: "Hoffman", birthDate:bDateProgrammer) 
```

我们可以在需要协议类型的地方使用任何符合我们协议的类型实例。例如，如果我们定义了一个变量为 `PersonProtocol` 协议类型，那么我们可以用任何符合此协议的类或结构体来填充这个变量。为了这个例子，让我们假设有两个类型，分别命名为 `SwiftProgrammer` 和 `FootballPlayer`，它们符合 `PersonProtocol` 协议：

```swift
var myPerson: PersonProtocol
myPerson = SwiftProgrammer(firstName: "Jon", lastName: "Hoffman", birthDate: bDateProgrammer)
print("\(myPerson.firstName) \(myPerson.lastName)")
myPerson = FootballPlayer(firstName: "Dan", lastName: "Marino", birthDate:bDatePlayer)
print("\(myPerson.firstName) \(myPerson.lastName)") 
```

在这个例子中，我们首先创建了一个 `PersonProtocol` 协议类型的 `myPerson` 变量。然后，我们将变量设置为 `SwiftProgrammer` 类型的实例，并打印出姓名和姓氏。接下来，我们将 `myPerson` 变量设置为 `FootballPlayer` 类型的实例，并再次打印出姓名和姓氏。需要注意的是，Swift 不关心实例是类还是结构体。唯一重要的是类型必须符合 `PersonProtocol` 协议类型。

我们可以将 `PersonProtocol` 协议用作数组的类型，这意味着我们可以用符合协议的任何类型的实例来填充数组。再次强调，类型是类还是结构体并不重要，只要它符合 `PersonProtocol` 协议。

# 协议的多态性

在之前的例子中，我们看到的是一种多态形式。多态这个词来自希腊语词根 **poly**，意为许多，和 **morphe**，意为形式。在编程语言中，多态是单一接口对多种类型（许多形式）。在之前的例子中，单一接口是 `PersonProtocol` 协议，而多种类型是任何符合该协议的类型。

多态性赋予我们以统一的方式与多种类型交互的能力。为了说明这一点，我们可以扩展之前的例子，其中我们创建了一个 `PersonProtocol` 类型的数组，并遍历了这个数组。然后，我们可以使用在 `PersonProtocol` 协议中定义的属性和方法来访问数组中的每个项目，无论实际类型如何。让我们看看这个例子：

```swift
for person in people { 
    print("\(person.firstName)\(person.lastName):\(person.profession)")
} 
```

当我们定义变量、常量、集合类型等类型为协议类型时，我们可以使用符合该协议的任何类型的实例。这是一个非常重要的概念，也是使协议和协议扩展如此强大的许多事情之一。

当我们使用协议来访问实例时，如前一个示例所示，我们仅限于使用协议本身定义的属性和方法。如果我们想使用特定于单个类型的属性或方法，我们需要将实例转换为该类型。

# 使用协议进行类型转换

类型转换是一种检查实例类型和/或将实例视为指定类型的方法。在 Swift 中，我们使用`is`关键字来检查实例是否是特定类型，并使用`as`关键字将实例视为特定类型。

首先，让我们看看我们如何使用`is`关键字检查实例类型。以下示例显示了如何进行此操作：

```swift
for person in people {
    if let p = person as? SwiftProgrammer { 
        print("\(person.firstName) is a Swift Programmer")
    }
} 
```

在这个示例中，我们使用`if`条件语句来检查`people`数组中的每个元素是否是`SwiftProgrammer`类型的实例，如果是，我们将打印出该人是 Swift 程序员到控制台。虽然这是一个检查我们是否有特定类或结构体实例的好方法，但如果我们要检查多个类型，则效率不高。使用`switch`语句会更高效，如下一个示例所示：

```swift
for person in people {
    switch person {
        case is SwiftProgrammer:
            print("\(person.firstName) is a Swift Programmer") 
        case is FootballPlayer:
            print("\(person.firstName) is a Football Player") 
        default:
            print("\(person.firstName) is an unknown type")
    }
} 
```

在前一个示例中，我们展示了如何使用`switch`语句检查数组中每个元素的实例类型。为此检查，我们在每个`case`语句中使用`is`关键字尝试匹配实例类型。

在*第六章*，*控制流*中，我们看到了如何使用`where`语句来过滤条件语句。我们还可以使用`where`语句与`is`关键字一起过滤数组，如下面的示例所示：

```swift
for person in people where person is SwiftProgrammer { 
    print("\(person.firstName) is a Swift Programmer")
} 
```

现在，让我们看看我们如何将类或结构的实例转换为特定类型。为此，我们将使用`as`关键字。由于类型转换可能会失败，如果实例不是指定类型，则`as`关键字有两种形式：`as?`和`as!`。使用`as?`形式，如果转换失败，它返回`nil`，而使用`as!`形式，如果转换失败，我们得到运行时错误。因此，除非我们绝对确定实例类型或在我们进行转换之前检查实例类型，否则建议使用`as?`形式。

虽然我们在本书中展示了使用`as!`进行类型转换的示例，以便您了解它的存在，但我们强烈建议您不要在项目中使用它，因为它可能导致运行时错误。

让我们看看我们如何使用`as?`关键字将类或结构的实例转换为指定类型：

```swift
for person in people {
    if let p = person as? SwiftProgrammer { 
        print("\(person.firstName) is a Swift Programmer")
    }
} 
```

由于`as?`关键字返回一个可选值，我们使用可选绑定来执行类型转换，如下面的示例所示。

现在我们已经涵盖了协议的基础知识，让我们深入探讨 Swift 中最激动人心的特性之一：协议扩展。

# 协议扩展

协议扩展使我们能够将协议扩展以提供符合类型的方法和属性实现。它们还允许我们为所有符合类型的提供通用实现，从而消除了在每个单独的类型中提供实现或创建类层次结构的需要。虽然协议扩展可能看起来并不太令人兴奋，但一旦你看到它们真正的强大之处，它们将改变你对代码思考和编写的看法。

让我们从如何在一个非常简单的例子中使用协议扩展开始。我们将首先定义一个名为`Dog`的协议，如下所示：

```swift
protocol Dog {
    var name: String { get set } 
    var color: String { get set }
} 
```

使用此协议，我们声明任何符合`Dog`协议的类型都必须具有名为`name`和`color`的`String`类型的两个属性。接下来，让我们定义符合此`Dog`协议的三个类型。我们将这些类型命名为`JackRussel`、`WhiteLab`和`Mutt`。以下代码展示了我们如何定义这些类型：

```swift
struct JackRussel: Dog{ 
    var name: String
    var color: String
}
class WhiteLab: Dog{
    var name: String
    var color: String
    init(name: String, color: String) {
        self.name = name
        self.color = color
    }
}
struct Mutt: Dog{ 
    var name: String 
    var color: String
} 
```

我们故意将`JackRussel`和`Mutt`类型创建为结构体，将`WhiteLab`类型创建为类，以展示两种类型设置的不同之处，并说明在协议和协议扩展方面它们是如何被同等对待的。

在这个例子中，我们可以看到最大的区别是结构体类型提供了一个默认的初始化器，但在类中我们必须提供初始化器来填充属性。

现在假设我们想要为符合协议的每个类型提供一个名为`speak`的方法。在协议扩展之前，我们会在协议中添加方法定义，如下面的代码所示：

```swift
protocol Dog{
    var name: String { get set } 
    var color: String { get set } 
    func speak() -> String
} 
```

一旦在协议中定义了方法，我们就需要在每个符合协议的类型中提供方法的实现。根据符合此协议的类型数量，这可能需要一些时间来实现，并且可能会影响大量的代码。以下代码示例展示了我们可能如何实现此方法：

```swift
struct JackRussel: Dog{ 
    var name: String
    var color: String
    func speak() -> String { 
        return "Woof"
    }
}
class WhiteLab: Dog{ 
    var name: String 
    var color: String
    init(name: String, color: String) { 
        self.name = nameself.color = color}
    func speak() -> String { 
        return "Woof"
    }
}
struct Mutt: Dog{ 
    var name: String 
    var color: String
    func speak() -> String { 
        return "Woof Woof"
    }
} 
```

虽然这个方法有效，但效率并不高，因为每次我们更新协议时，都需要更新所有符合该协议的类型，因此会重复大量的代码，如本例所示。如果我们需要更改` speak()`方法的默认行为，我们就必须进入每个实现并更改该方法。这就是协议扩展发挥作用的地方。

使用协议扩展，我们可以将`speak()`方法定义从协议本身中提取出来，并在协议扩展中定义其默认行为。

如果我们在协议扩展中实现一个方法，我们不需要在协议中定义它。

以下代码展示了我们如何定义协议和协议扩展：

```swift
protocol Dog{
    var name: String { get set } 
    var color: String { get set }
}
extension Dog{
    func speak() -> String { 
      return "Woof Woof"
    }
  } 
```

我们首先定义了具有原始两个属性的`Dog`协议。然后我们创建了一个扩展该协议的协议扩展，其中包含`speak()`方法的默认实现。使用这段代码，我们不需要在所有符合`Dog`协议的类型中提供`speak()`方法的实现，因为它们会自动作为协议的一部分接收该实现。

让我们通过将符合`Dog`协议的三个类型恢复到它们的原始实现来查看这是如何工作的；然后它们应该会从协议扩展中接收`speak()`方法：

```swift
struct JackRussel: Dog{
    var name: String
    var color: String
}
class WhiteLab: Dog{
    var name: String
    var color: String
    init(name: String, color: String) {
        self.name = name
        self.color = color
    }
}
struct Mutt: Dog{ 
    var name: String 
    var color: String
} 
```

我们现在可以使用每种类型，如下面的代码所示：

```swift
let dash = JackRussel(name: "Dash", color: "Brown and White")
let lily = WhiteLab(name: "Lily", color: "White")
let maple = Mutt(name: "Buddy", color: "Brown")
let dSpeak = dash.speak() // returns "woof woof" 
let lSpeak = lily.speak() // returns "woof woof" 
let bSpeak = maple.speak() // returns "woof woof" 
```

如此例所示，通过将`speak()`方法添加到`Dog`协议扩展中，我们自动将该方法添加到所有符合该协议的类型中。协议扩展中的`speak()`方法可以被视为方法的默认实现，因为我们能够在类型实现中覆盖它。例如，我们可以在`Mutt`结构中覆盖`speak()`方法，如下面的代码所示：

```swift
struct Mutt: Dog{ 
    var name: String 
    var color: String
    func speak() -> String {
        return "I am hungry"
    }
} 
```

当我们为`Mutt`类型的实例调用`speak()`方法时，它将返回`I am hungry`字符串。

在本章中，我们使用协议后缀命名我们的协议。这样做是为了使这一点非常明确，这是一个协议。这通常不是我们命名类型的方式。以下示例给出了我们如何正确命名协议的更好示例。您可以在 Swift API 设计指南中阅读有关 Swift 命名约定的更多信息：[`swift.org/documentation/api-design-guidelines/#general-conventions`](https://swift.org/documentation/api-design-guidelines/#general-conventions)。

现在我们已经看到了如何使用协议和协议扩展，让我们看看一个更贴近现实世界的例子。

## 现实世界的例子

在多个平台（iOS、Android 和 Windows）的众多应用程序中，我需要在用户输入时验证用户输入。这种验证可以使用正则表达式非常容易地完成；然而，我们不希望在代码中散布各种正则表达式。通过创建包含验证代码的不同类或结构，可以很容易地解决这个问题。然而，我们必须组织这些类型，以便它们易于使用和维护。在 Swift 的协议扩展之前，我会使用一个协议来定义验证要求，然后为每个需要的验证创建符合该协议的结构。让我们看看这种方法在协议扩展之前是如何工作的。

正则表达式是一系列字符，它定义了一个特定的模式。然后可以使用该模式来搜索字符串，以查看字符串是否与模式匹配或包含模式匹配。大多数主要编程语言都包含正则表达式解析器，如果您不熟悉正则表达式，了解它们可能是有益的。

以下代码展示了 `TextValidating` 协议，它定义了我们想要用于文本验证的任何类型的必要要求：

```swift
protocol TextValidating {
    var regExMatchingString: String { get } 
    var regExFindMatchString: String { get } 
    var validationMessage: String { get }
    func validateString(str: String) -> Bool
    func getMatchingString(str: String) -> String?
} 
```

Swift API 设计指南（[`swift.org/documentation/api-design-guidelines/`](https://swift.org/documentation/api-design-guidelines/)）指出，描述“是什么”的协议应命名为名词，而描述“能做什么”的协议应使用后缀 -able、-ible 或 -ing。考虑到这一点，我们命名文本验证协议为 `TextValidating`。

在此协议中，我们定义了三个属性和两个方法，任何符合协议的类型都必须实现。这三个属性如下：

+   `regExMatchingString`：这是一个正则表达式字符串，用于验证输入字符串是否只包含有效字符。

+   `regExFindMatchString`：这是一个正则表达式字符串，用于从输入字符串中检索只包含有效字符的新字符串。此正则表达式通常在我们需要实时验证用户输入信息时使用，因为它将找到输入字符串的最长匹配前缀。

+   `validationMessage`：这是如果输入字符串包含非有效字符时显示的错误信息。

此协议的两个方法如下：

+   `validateString`：如果输入字符串只包含有效字符，则此方法将返回 `true`。在此方法中，将使用 `regExMatchingString` 属性来执行匹配。

+   `getMatchingString`：此方法将返回一个只包含有效字符的新字符串。此方法通常在我们需要实时验证用户输入信息时使用，因为它将找到输入字符串的最长匹配前缀。我们将在此方法中使用 `regExFindMatchString` 属性来检索新字符串。

现在我们来看看如何创建一个符合此协议的结构。以下结构将用于验证输入字符串是否只包含字母字符：

```swift
struct AlphaValidation1: TextValidating {
    static let sharedInstance = AlphaValidation1()
    private init(){}
    let regExFindMatchString = "^[a-zA-Z]{0,10}"
    let validationMessage = "Can only contain Alpha characters" 
    var regExMatchingString: String {
        get {
            return regExFindMatchString + "$"
        }
    }
    func validateString(str: String) -> Bool {
        if let _ = str.range(of: regExMatchingString, options:                   .regularExpression) {
        return true
    } else {
        return false
        }
    }
    func getMatchingString(str: String) -> String? {
        if let newMatch = str.range(of: regExFindMatchString,
                          options:.regularExpression) {
        return String(str[newMatch])
    } else {
        return nil
        }
    }
} 
```

在此实现中，`regExFindMatchString` 和 `validationMessage` 属性是存储属性，而 `regExMatchingString` 属性是一个计算属性。

我们还在结构内部实现了 `validateString()` 和 `getMatchingString()` 方法。

通常，我们会有几个不同的类型符合协议，其中每个类型都会验证不同类型的输入。正如我们可以从 `AlphaValidation1` 结构中看到的那样，每个验证类型都涉及一些代码。大量的代码在每个类型中也会重复。方法和 `regExMatchingString` 属性的代码可能会在每个验证类中重复。这并不理想，但如果我们想避免创建一个包含重复代码的超类类层次结构（建议我们优先使用值类型而不是引用类型），在协议扩展之前，我们别无选择。现在让我们看看我们如何使用协议扩展来实现这一点。

使用协议扩展时，我们需要稍微改变一下对代码的思考方式。最大的不同是，我们既不需要也不希望定义协议中的所有内容。在标准协议中，所有你希望通过协议接口访问的方法和属性都必须在协议内部定义。

使用协议扩展时，如果我们打算在协议扩展中定义它，我们更倾向于不在协议中定义属性或方法。因此，当我们用协议扩展重写我们的文本验证类型时，`TextValidating` 将大大简化，如下所示：

```swift
protocol TextValidating {
    var regExFindMatchString: String { get } 
    var validationMessage: String { get }
} 
```

在原始的 `TextValidating` 协议中，我们定义了三个属性和两个方法。正如我们可以从新的协议中看到的那样，我们现在只定义了两个属性。现在我们已经定义了 `TextValidating` 协议，让我们为它创建协议扩展：

```swift
extension TextValidating {
    var regExMatchingString: String { 
        get {
            return regExFindMatchString + "$"
        }
    }
    func validateString(str: String) -> Bool {
        if let _ = str.range(of:regExMatchingString,                   options:.regularExpression){
        return true
    } else {
        return false
        }
    }
    func getMatchingString(str: String) -> String? {
        if let newMatch = str.range(of:regExFindMatchString,                          options:.regularExpression) {
        return str.substring(with: newMatch)
    } else {
        return nil
        }
    }
} 
```

在 `TextValidating` 协议扩展中，我们定义了两个方法和一个属性，这些在原始的 `TextValidating` 协议中已经定义，但在新的协议中没有定义。现在我们已经创建了协议和协议扩展，我们能够定义我们新的文本验证类型。在下面的代码中，我们定义了三个结构，我们将使用这些结构来验证用户输入的文本：

```swift
struct AlphaValidation: TextValidating {
    static let sharedInstance = AlphaValidation()
    private init(){}
    let regExFindMatchString = "^[a-zA-Z]{0,10}"
    let validationMessage = "Can only contain Alpha characters"
}
struct AlphaNumericValidation: TextValidating {
    static let sharedInstance = AlphaNumericValidation() 
    private init(){}
    let regExFindMatchString = "^[a-zA-Z0-9]{0,15}"
    let validationMessage = "Can only contain Alpha Numeric characters"
}
struct DisplayNameValidation: TextValidating {
    static let sharedInstance = DisplayNameValidation() 
    private init(){}
    let regExFindMatchString = "^[\\s?[a-zA-Z0-9\\-_\\s]]{0,15}"
    let validationMessage = "Can only contain Alphanumeric Characters"
} 
```

在每个文本验证结构中，我们创建一个静态常量和私有初始化器，这样我们就可以将结构作为单例使用。有关单例模式的更多信息，请参阅 *《Swift 中的设计模式采用》* 第二十章的 *“单例设计模式”* 部分。

在定义了单例模式之后，在每个类型中我们只需设置 `regExFindMatchString` 和 `validationMessage` 属性的值。现在我们几乎没有任何重复的代码。唯一重复的代码是单例模式的代码，我们不会想在协议扩展中放入这些代码，因为我们不希望将单例模式强加给所有符合的类型。

现在我们可以使用文本验证类型，如下面的代码所示：

```swift
var testString = "abc123"
var alpha = AlphaValidation.sharedInstance alpha.getMatchingString(str:testString) 
alpha.validateString(str: testString) 
AlphaValidation type. Then getMatchingString() is used to retrieve the longest matching prefix of the test string, which will be abc. Then, the validateString() method is used to validate the test string, but since the test string contains numbers, the method will return false.
```

现在的问题是，我们真的需要使用协议吗？

# 我需要使用协议吗？

当你已经知道面向对象编程（OOP）时，还需要使用协议和协议扩展吗？简短的答案是：不需要；然而，强烈推荐。在*第十章*，*协议导向设计*中，我们探讨了协议导向设计为何如此强大，以展示为什么你应该在协议导向编程（POP）中优先选择协议而不是面向对象编程（OOP）。通过理解协议和协议导向设计，你会更好地理解 Swift 标准库。

# 使用合成实现采用协议

Swift 可以在特定情况下自动为`Equatable`、`Hashable`和`Comparable`协议提供协议遵循。这意味着我们不需要编写实现这些协议的样板代码，而是可以使用合成实现。这仅适用于结构体或枚举（不是类）只包含遵循`Equatable`、`Hashable`和`Comparable`协议的存储属性（对于结构体）或关联值（对于枚举）。

Swift 标准库提供了`Equatable`、`Hashable`和`Comparable`协议。任何遵循`Equatable`协议的类型都可以使用等号运算符（==）来比较该类型的两个实例。任何使用`Comparable`协议的类型都可以使用比较运算符来比较该类型的两个实例。最后，任何遵循`Hashable`协议的类型都可以被哈希到`Hasher`实例中，以生成一个整数哈希值。

让我们来看一个例子。我们将从一个简单的结构体开始，该结构体将存储名称：

```swift
struct Name {
    var firstName = ""
    var lastName = ""
} 
```

现在，我们可以创建三个`Name`结构体的实例，如下面的代码所示：

```swift
let name1 = Name(firstName: "Jon", lastName: "Hoffman")
let name2 = Name(firstName: "John", lastName: "Hoffman")
let name3 = Name(firstName: "Jon", lastName: "Hoffman") 
```

如果我们尝试比较`Name`结构体的实例，如下面的代码所示，我们可能会收到编译时错误，因为`Name`结构体没有遵循`Equatable`协议：

```swift
name1 == name2
name1 == name3 
```

为了能够比较`Name`结构体的实例，所需的所有操作就是声明该结构体遵循`Equatable`协议，编译时将自动添加执行比较的样板代码。下面的代码展示了如何实现这一点：

```swift
struct Name: Equatable {
    var firstName = ""
    var lastName = ""
} 
```

注意，我们唯一改变的是将`Equatable`协议添加到结构体定义中。现在，我们能够成功比较`Name`结构体的实例。

# Swift 标准库

Swift 标准库为编写 Swift 应用程序定义了一个基础功能层。在这本书中我们迄今为止使用的一切都来自 Swift 标准库。该库定义了基本数据类型，如`String`、`Int`和`Double`类型。它还定义了集合、可选类型、全局函数以及这些类型遵循的所有协议。

要查看构成标准库的所有内容的最佳网站之一是[`swiftdoc.org`](http://swiftdoc.org)。该网站列出了构成标准库的所有类型、协议、运算符和全局变量，并包含了所有内容的文档。

让我们通过查看文档来了解标准库中协议的使用情况。当你第一次访问主页时，你会看到一个可搜索的列表，列出了构成标准库的所有内容。还有一个你可以从中选择的 Swift 类型完整列表。让我们通过点击 **Array** 链接来查看 Swift 的 `Array` 类型。这将带你到 `Array` 类型的文档页面。

这些文档页面非常有用，包含大量关于构成标准库的各种类型的信息，包括如何使用它们的示例。对于我们讨论的目的，我们感兴趣的章节是标记为 **继承** 的部分：

![](img/B16683_09_01.png)

图 9.1：继承文档

从继承部分，我们可以看到 Array 符合 7 个协议。如果你点击一些协议，例如 `MutableCollection` 协议，你会注意到它们符合其他协议。这一点现在可能不太容易理解，但在下一章，即第十章“面向协议设计”中，我们将探讨如何使用面向协议的方法来设计我们的应用程序和框架，然后我们将更好地理解 Swift 标准库是如何编写的。

# 摘要

在本章中，我们了解到 Swift 将协议视为完整的类型。我们还看到了如何在 Swift 中使用协议实现多态。我们通过深入探讨协议扩展以及如何在 Swift 中使用它们来结束本章。协议和协议扩展是苹果新 POP（面向协议）范式的骨架。这种新的编程模型有可能改变我们编写和思考代码的方式。虽然我们本章没有专门介绍 POP，但掌握本章的主题为我们提供了学习这种新编程模型所需的关于协议和协议扩展的坚实基础。在下一章中，我们将探讨在设计我们的应用程序时如何使用协议和协议扩展。
