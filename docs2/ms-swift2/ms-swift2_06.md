# 第六章 使用协议和协议扩展

当我在观看 2015 年 WWDC 关于协议扩展和面向协议编程的演讲时，我必须承认我非常怀疑。我从事面向对象编程已经很长时间了，以至于我不确定这种新的编程范式是否能够解决苹果所声称的所有问题。既然我不是那种让怀疑阻碍尝试新事物的人，我就设置了一个新的项目，这个项目与我现在正在工作的项目相似，但我使用了苹果面向协议编程的建议来编写代码，并在代码中广泛使用了协议扩展。我可以诚实地说我被这个新项目与原始项目相比的整洁程度所震惊。我相信协议扩展将是那些定义性特征之一，将一种编程语言与其它编程语言区分开来。我也相信许多主要语言很快就会拥有类似的功能。

在本章中，你将学习：

+   协议作为类型的使用

+   如何在 Swift 中使用协议实现多态

+   如何使用协议扩展

+   为什么我们想要使用协议扩展

协议扩展是苹果新面向协议编程范式的骨架，并且可以说是 Swift 编程语言最重要的补充之一。有了协议扩展，我们能够为符合协议的任何类型提供方法和属性实现。为了真正理解协议和协议扩展有多有用，让我们更好地理解协议。

### 注意

虽然`class`、`struct`和`enum`都可以在 Swift 中符合协议，但在这个章节中，我们将专注于`class`和`struct`。当我们需要表示有限数量的情况时，我们会使用`enum`，虽然有一些有效的用例，其中枚举会符合协议，但在我个人的经验中，这种情况非常罕见。只需记住，在任何我们提到`class`或`struct`的地方，我们也可以使用`enum`。

让我们从了解它们在 Swift 中是如何作为完整的类型开始探索协议。

# 协议作为类型

尽管在协议中没有实现任何功能，但在 Swift 编程语言中，它们仍然被视为一个完整的类型，可以像任何其他类型一样使用。这意味着我们可以将协议用作函数中的参数类型或返回类型。我们还可以将它们用作变量、常量和集合的类型。让我们看看一些例子。对于这些例子，我们将使用`PersonProtocol`协议：

```swift
protocol PersonProtocol {
    var firstName: String {get set}
    var lastName: String {get set}
    var birthDate: NSDate {get set}
    var profession: String {get}

    init (firstName: String, lastName: String, birthDate: NSDate)
}
```

在这个第一个例子中，我们将看到我们如何将协议用作函数、方法或初始化器中的参数类型或返回类型：

```swift
  func updatePerson(person: PersonProtocol) -> PersonProtocol {
      // Code to update person goes here
      return person
   }
```

在这个例子中，`updatePerson()`函数接受一个`PersonProtocol`协议类型的参数，然后返回一个`PersonProtocol`协议类型的值。现在让我们看看我们如何可以将协议用作常量、变量或属性的类型：

```swift
var myPerson: PersonProtocol
```

在这个例子中，我们创建了一个名为`myPerson`的`PersonProtocol`协议类型的变量。我们还可以将协议用作集合（如数组、字典或集合）中的项目类型来存储：

```swift
var people: [PersonProtocol] = []
```

在这个最后的例子中，我们创建了一个`PersonProtocol`协议类型的数组。从这三个例子中我们可以看到，尽管`PersonProtocol`协议没有实现任何功能，我们仍然可以在需要指定类型时使用协议。然而，我们无法创建协议的实例。这是因为协议中没有实现任何功能。例如，如果我们尝试创建`PersonProtocol`协议的实例，我们会收到`error: 协议类型 'PersonProtocol' 不能实例化`错误，如下面的例子所示：

```swift
var test = PersonProtocol(firstName: "Jon", lastName: "Hoffman", birthDate: bDateProgrammer)
```

我们可以在需要协议类型的地方使用任何符合我们协议的类或结构体的实例。例如，如果我们定义一个变量为`PersonProtocol`协议类型，那么我们可以用任何符合`PersonProtocol`协议的类或结构体来填充这个变量。为了这个例子，让我们假设我们有两个类型名为`SwiftProgrammer`和`FootballPlayer`，它们符合`PersonProtocol`协议：

```swift
var myPerson: PersonProtocol

myPerson = SwiftProgrammer(firstName: "Jon", lastName: "Hoffman", birthDate: bDateProgrammer)
print("\(myPerson.firstName) \(myPerson.lastName)")

myPerson = FootballPlayer(firstName: "Dan", lastName: "Marino", birthDate: bDatePlayer)
print("\(myPerson.firstName) \(myPerson.lastName)")
```

在这个例子中，我们首先创建了一个`PersonProtocol`协议类型的`myPerson`变量。然后我们将变量设置为`SwiftProgrammer`类型的实例并打印出名字和姓氏。接下来，我们将`myPerson`变量设置为`FootballPlayer`类型的实例并再次打印出名字和姓氏。需要注意的是，Swift 并不关心实例是类还是结构体。重要的是类型符合`PersonProtocol`协议类型。因此，如果我们的`SwiftProgrammer`类型是结构体，而`FootballPlayer`类型是类，我们之前的例子将完全有效。

正如我们之前看到的，我们可以将我们的`PersonProtocol`协议用作数组的类型。这意味着我们可以用符合`PersonProtocol`协议的任何类型的实例来填充数组。再次强调，只要类型符合`PersonProtocol`协议，类型是类还是结构体并不重要。以下是一个例子：

```swift
var programmer = SwiftProgrammer(firstName: "Jon", lastName: "Hoffman", birthDate: bDateProgrammer)

var player = FootballPlayer(firstName: "Dan", lastName: "Marino", birthDate: bDatePlayer)

var people: [PersonProtocol] = []
people.append(programmer)
people.append(player)
```

在这个例子中，我们创建了一个`SwiftProgrammer`类型的实例和一个`FootballPlayer`类型的实例。然后我们将这两个实例添加到`people`数组中。

# 使用协议的多态性

我们在之前的例子中看到的是一种多态形式。多态这个词来源于希腊语根词*Poly*，意为许多，*morphe*，意为形式。在编程语言中，多态是单一接口对多种类型（许多形式）。在之前的例子中，单一接口是`PersonProtocol`协议，多种类型是符合该协议的任何类型。

多态性使我们能够以统一的方式与多个类型交互。为了说明这一点，我们可以扩展我们之前的示例，其中我们创建了一个`PersonProtocol`类型的数组并遍历该数组。然后我们可以使用在`PersonProtocol`协议中定义的属性和方法来访问数组中的每个项目，而不管实际的类型是什么。让我们看看这个示例：

```swift
for person in people {
    print("\(person.firstName) \(person.lastName): \(person.profession)")
}
```

如果我们运行此示例，输出将类似于以下内容：

```swift
Jon Hoffman: Swift Programmer
Dan Marino: Football Player
```

在本章中，我们提到过几次，当我们定义变量、常量、集合类型等类型为协议类型时，我们就可以使用符合该协议的任何类型的实例。这是一个非常重要的概念，也是协议和协议扩展之所以强大的原因。

当我们使用协议来访问实例时，如前一个示例所示，我们仅限于使用协议中定义的属性和方法。如果我们想使用特定于单个类型的属性或方法，我们需要将该实例转换为该类型。

# 使用协议进行类型转换

类型转换是一种检查实例类型的方法，并且/或者将实例视为指定的类型。在 Swift 中，我们使用`is`关键字来检查实例是否是特定类型，并使用`as`关键字将实例视为特定类型。

首先，让我们看看如何使用`is`关键字来检查实例类型。以下示例展示了如何进行此操作：

```swift
for person in people {
  if person is SwiftProgrammer {
     print("\(person.firstName) is a Swift Programmer")
}
}
```

在这个示例中，我们使用`if`条件语句来检查`people`数组中的每个元素是否是`SwiftProgrammer`类型的实例，如果是，我们将打印出该人是 Swift 程序员到控制台。虽然这是一种检查我们是否具有特定类或结构体实例的好方法，但如果我们要检查多个类型，则效率不高。如果我们想要检查多个类型，使用`switch`语句会更高效，如下一个示例所示：

```swift
for person in people {
    switch (person) {
    case is SwiftProgrammer:
        print("\(person.firstName) is a Swift Programmer")
    case is FootballPlayer:
        print("\(person.firstName) is a Football Player")
    default:
        print("\(person.firstName) is an unknown type")
    }
}
```

在前一个示例中，我们展示了如何使用`switch`语句来检查数组中每个元素的实例类型。为此检查，我们在每个`case`语句中使用`is`关键字尝试匹配实例类型。

在第四章中，*控制流和函数*，我们看到了如何使用`where`语句来过滤条件语句。我们还可以使用`where`语句与`is`关键字一起过滤数组，如下面的示例所示：

```swift
for person in people where person is SwiftProgrammer {
    print("\(person.firstName) is a Swift Programmer")

}
```

现在让我们看看我们如何将一个类或结构体的实例转换为特定类型。为此，我们可以使用 `as` 关键字。由于转换可能会失败，如果实例不是指定的类型，`as` 关键字有两种形式：`as?` 和 `as!`。使用 `as?` 形式，如果转换失败，它返回 nil，而使用 `as!` 形式，如果转换失败，我们会得到一个运行时错误；因此，除非我们绝对确定实例类型，或者在我们进行转换之前对实例类型进行了检查，否则建议使用 `as?` 形式。

让我们看看我们如何使用 `as?` 关键字将一个类或结构体的实例转换为指定的类型：

```swift
for person in people {
    if let p = person as? SwiftProgrammer {
        print("\(person.firstName) is a Swift Programmer")
    }
}
```

由于 `as?` 关键字返回一个可选值，我们可以使用可选绑定来执行转换，如下面的例子所示。如果我们确定实例类型，我们可以使用 `as!` 关键字。以下示例展示了如何使用 `as!` 关键字在过滤数组结果时仅返回 `SwiftProgrammer` 类型的实例：

```swift
for person in people where person is SwiftProgrammer {
  let p = person as! SwiftProgrammer
}
```

现在我们已经涵盖了协议的基础知识，即多态工作和类型转换，让我们深入了解 Swift 协议扩展中最激动人心的一个新特性。

# 协议扩展

协议扩展允许我们扩展一个协议，为符合的类型提供方法和属性实现。它们还允许我们为所有符合的类型提供通用实现，从而消除了在每个单独的类型中提供实现或创建类层次结构的需要。虽然协议扩展可能看起来并不太吸引人，但一旦你看到它们真正的强大之处，它们将改变你思考和使用代码的方式。

让我们从使用一个非常简单的例子来看如何使用协议扩展开始。我们将从定义一个名为 `DogProtocol` 的协议开始，如下所示：

```swift
protocol DogProtocol {
    var name: String {get set}
    var color: String {get set}
}
```

使用这个协议，我们是在说任何符合 `DogProtocol` 协议的类型，都必须有两个 String 类型的属性，即 `name` 和 `color`。现在让我们定义符合此协议的三个类型。我们将这些类型命名为 `JackRussel`、`WhiteLab` 和 `Mutt`，如下所示：

```swift
struct JackRussel: DogProtocol {
    var name: String
    var color: String
}

class WhiteLab: DogProtocol {
    var name: String
    var color: String

    init(name: String, color: String) {
        self.name = name
        self.color = color
    }
}

struct Mutt: DogProtocol {
    var name: String
    var color: String
}
```

我们故意将 `JackRussel` 和 `Mutt` 类型创建为结构体，将 `WhiteLab` 类型创建为类，以展示两种类型设置方式的不同，并说明当涉及到协议和协议扩展时，它们是如何以相同的方式被处理的。在这个例子中，我们可以看到的最大区别是结构体类型提供了一个默认的初始化器，但在类中，我们必须提供初始化器来填充属性。

现在假设我们想要为符合 `DogProtocol` 协议的每个类型提供一个名为 `speak` 的方法。在协议扩展之前，我们会从向协议添加方法定义开始，如下面的代码所示：

```swift
protocol DogProtocol {
    var name: String {get set}
    var color: String {get set}
   func speak() -> String
}
```

一旦在协议中定义了该方法，我们随后就需要为符合该协议的每个类型提供一个该方法的实现。根据符合此协议的类型数量，实现这一过程可能需要一些时间。以下代码示例展示了我们可能如何实现此方法：

```swift
struct JackRussel: DogProtocol {
    var name: String
    var color: String
     func speak() -> String {
        return "Woof Woof"
    }
}

class WhiteLab: DogProtocol {
    var name: String
    var color: String

    init(name: String, color: String) {
        self.name = name
        self.color = color
    }
    func speak() -> String {
        return "Woof Woof"
    }

}

struct Mutt: DogProtocol {
    var name: String
    var color: String
    func speak() -> String {
        return "Woof Woof"
    }

}
```

虽然此方法可行，但效率不高，因为每次我们更新协议时，都需要更新所有符合该协议的类型，并且我们可能会在代码中重复很多内容，如本例所示。另一个担忧是，如果我们需要更改` speak()`方法的默认行为，我们必须进入每个实现并更改` speak()`方法。这就是协议扩展发挥作用的地方。

使用协议扩展，我们可以将` speak()`方法定义从协议本身中提取出来，并在协议扩展中定义它，带有默认行为。以下代码展示了我们将如何定义协议和协议扩展：

```swift
protocol DogProtocol {
    var name: String {get set}
    var color: String {get set}
}

extension DogProtocol {
    func speak() -> String {
        return "Woof Woof"
    }
}
```

我们首先使用原始的两个属性定义`DogProtocol`。然后创建一个扩展`DogProtocol`的协议扩展，其中包含` speak()`方法的默认实现。使用此代码，我们无需在每个符合`DogProtocol`的类型的实现中提供` speak()`方法的实现，因为它们会自动作为协议的一部分接收实现。让我们通过将符合`DogProtocol`的三个类型恢复到它们的原始实现来查看这是如何工作的，它们应该会从协议扩展中接收` speak()`方法：

```swift
struct JackRussel: DogProtocol {
    var name: String
    var color: String
}
class WhiteLab: DogProtocol {
    var name: String
    var color: String

    init(name: String, color: String) {
        self.name = name
        self.color = color
    }
}

struct Mutt: DogProtocol {
    var name: String
    var color: String
}
```

我们现在可以使用以下代码所示的方式使用每个类型：

```swift
let dash = JackRussel(name: "Dash", color: "Brown and White")
let lily = WhiteLab(name: "Lily", color: "White")
let buddy = Mutt(name: "Buddy", color: "Brown")
let dSpeak = dash.speak()  // returns "woof woof"
let lSpeak = lily.speak()  // returns "woof woof"
let bSpeak = buddy.speak() // returns "woof woof"
```

如此例所示，通过将` speak()`方法添加到` DogProtocol`协议扩展中，我们自动将该方法添加到所有符合` DogProtocol`的类型中。` DogProtocol`协议扩展中的` speak()`方法可以被视为` speak()`方法的默认实现，因为我们能够在类型实现中覆盖它。例如，我们可以在以下代码中覆盖` Mutt`结构体中的` speak()`方法：

```swift
struct Mutt: DogProtocol {
    var name: String
    var color: String
    func speak() -> String {
        return "I am hungry"
    }
}
```

当我们为`Mutt`类型的实例调用` speak()`方法时，它将返回字符串`"I am hungry"`。

现在我们已经看到了如何使用协议和协议扩展，让我们看看一个更实际的例子。在多个平台（iOS、Android 和 Windows）的众多应用程序中，我需要验证用户输入。这种验证可以用正则表达式非常容易地完成；然而，我们不希望各种正则表达式散布在我们的代码中。通过创建包含验证代码的不同类或结构体来解决这个问题非常容易；然而，我们必须组织这些类，使它们易于使用和维护。在 Swift 的协议扩展之前，我会使用协议来定义验证要求，然后为每个需要的验证创建一个符合协议的结构体。让我们看看这个预协议扩展方法。

### 注意

正则表达式是一系列字符，用于定义特定的模式。这个模式可以用来搜索字符串，以查看字符串是否匹配该模式或包含该模式的匹配项。大多数主要的编程语言都包含正则表达式解析器，如果你不熟悉正则表达式，了解它们可能是有益的。

以下代码显示了定义任何我们想要用于文本验证的类型的要求的`TextValidationProtocol`协议：

```swift
 protocol TextValidationProtocol {

    var regExMatchingString: String {get}
    var regExFindMatchString: String {get}
    var validationMessage: String {get}

    func validateString(str: String) -> Bool
    func getMatchingString(str: String) -> String?
}
```

在此协议中，我们定义了三个属性和两个方法，任何符合`TextValidationProtocol`的类型都必须实现。这三个属性是：

+   `regExMatchingString`：这是一个正则表达式字符串，用于验证输入字符串是否只包含有效字符。

+   `regExFindMatchString`：这是一个正则表达式字符串，用于从输入字符串中检索只包含有效字符的新字符串。这个正则表达式通常在我们需要实时验证输入时使用，因为用户输入信息时，它会找到输入字符串的最长匹配前缀。

+   `validationMessage`：这是如果输入字符串包含非有效字符时显示的错误信息。

此协议的两种方法如下：

+   `validateString`：如果输入字符串只包含有效字符，此方法将返回`true`。在这个方法中，将使用`regExMatchingString`属性来执行匹配。

+   `getMatchingString`：此方法将返回一个只包含有效字符的新字符串。这个方法通常在我们需要实时验证用户输入时使用，因为它会找到输入字符串的最长匹配前缀。我们将使用`regExFindMatchString`属性来检索新字符串。

现在我们来看看如何创建一个符合此协议的结构体。以下结构体会被用来验证输入字符串是否只包含字母字符：

```swift
struct AlphaValidation1: TextValidationProtocol {
    static let sharedInstance = AlphaValidation1()
    private init(){}

    let regExFindMatchString = "^[a-zA-Z]{0,10}"
    let validationMessage = "Can only contain Alpha characters"

    var regExMatchingString: String { get {
        return regExFindMatchString + "$"
        }
    }

    func validateString(str: String) -> Bool {
        if let _ = str.rangeOfString(regExMatchingString, options: .RegularExpressionSearch) {
            return true
        } else {
            return false
        }
    }
    func getMatchingString(str: String) -> String? {
        if let newMatch = str.rangeOfString(regExFindMatchString, options: .RegularExpressionSearch) {
            return str.substringWithRange(newMatch)
        } else {
            return nil
        }
    }
}
```

在此实现中，`regExFindMatchString`和`validationMessage`属性是存储属性，而`regExMatchingString`属性是一个计算属性。我们还在结构体内部实现了`validateString()`和`getMatchingString()`方法。

通常情况下，我们会有几个不同的类型符合`TextValidationProtocol`，其中每个类型都会验证不同类型的输入。正如我们从`AlphaValidation1`结构体中可以看到的，每个验证类型都涉及一些代码。大量的代码也会在每个类型中重复。两个方法（`validateString()`和`getMatchingString()`）和`regExMatchingString`属性的代码会在每个验证类中重复。这并不理想，但如果我们想避免创建一个包含重复代码的超级类类层次结构（我个人更喜欢使用值类型而不是类），我们就别无选择。现在让我们看看我们如何使用协议扩展来实现这一点。

使用协议扩展时，我们需要稍微改变一下对代码的思考方式。最大的不同是，我们既不需要也不希望定义协议中的所有内容。使用标准协议或当我们使用类层次结构时，所有你想要通过通用超类或协议访问的方法和属性都必须在超类或协议中定义。使用协议扩展时，如果我们打算在协议扩展中定义它，我们更倾向于不在协议中定义属性或方法。因此，当我们用协议扩展重写我们的文本验证类型时，`TextValidationProtocol`将大大简化，看起来类似于这样：

```swift
protocol TextValidationProtocol {
    var regExFindMatchString: String {get}
    var validationMessage: String {get}
}
```

在原始的`TextValidationProtocol`中，我们定义了三个属性和两个方法。正如我们在这个新协议中可以看到的，我们只定义了两个属性。现在我们已经定义了`TextValidationProtocol`，让我们为它创建一个协议扩展：

```swift
extension TextValidationProtocol {

    var regExMatchingString: String { get {
        return regExFindMatchString + "$"
        }
    }

    func validateString(str: String) -> Bool {
        if let _ = str.rangeOfString(regExMatchingString, options: .RegularExpressionSearch) {
            return true
        } else {
            return false
        }
    }
    func getMatchingString(str: String) -> String? {
        if let newMatch = str.rangeOfString(regExFindMatchString, options: .RegularExpressionSearch) {
            return str.substringWithRange(newMatch)
        } else {
            return nil
        }
    }
}
```

在`TextValidationProtocol`协议扩展中，我们定义了原始`TextValidationProtocol`中定义的两个方法和第三个属性，但在新协议中没有定义。现在我们已经创建了我们的协议和协议扩展，我们能够定义我们的文本验证类型。在下面的代码中，我们定义了三个结构体，我们将使用这些结构体在用户输入文本时验证文本：

```swift
struct AlphaValidation: TextValidationProtocol {
    static let sharedInstance = AlphaValidation()
    private init(){}

    let regExFindMatchString = "^[a-zA-Z]{0,10}"
    let validationMessage = "Can only contain Alpha characters"
}

struct AlphaNumericValidation: TextValidationProtocol {
    static let sharedInstance = AlphaNumericValidation()
    private init(){}

    let regExFindMatchString = "^[a-zA-Z0-9]{0,15}"
    let validationMessage = "Can only contain Alpha Numeric characters"
}

struct DisplayNameValidation: TextValidationProtocol {
    static let sharedInstance = DisplayNameValidation()
    private init(){}

    let regExFindMatchString = "^[\\s?[a-zA-Z0-9\\-_\\s]]{0,15}"
    let validationMessage = "Display Name can contain only contain Alphanumeric Characters"
}
```

在每个文本验证结构体中，我们创建一个静态常量和私有初始化器，以便我们可以将结构体用作单例。有关单例模式的更多信息，请参阅第十七章中的*单例设计模式*部分，*在 Swift 中使用设计模式*。

在我们定义了单例模式之后，在每种类型中我们只需设置 `regExFindMatchString` 和 `validationMessage` 属性的值。现在，我们实际上并没有复制代码，因为即使我们可以，我们也不希望在协议扩展中定义单例代码，因为我们不希望将该模式强加给所有符合的类型。

要使用文本验证类，我们希望创建一个字典对象，将 `UITextField` 对象映射到要使用的验证类，如下所示：

```swift
var validators = [UITextField: TextValidationProtocol]()
```

然后，我们可以像下面这样填充 `validators` 字典：

```swift
validators[alphaTextField] = AlphaValidation.sharedInstance

validators[alphaNumericTextField] = AlphaNumericValidation.sharedInstance

validators[displayNameTextField] = DisplayNameValidation.sharedInstance
```

现在，我们可以将文本字段的 `EditingChanged` 事件设置为名为 `keyPressed()` 的单个方法。要为每个字段设置编辑更改事件，我们将在视图控制器的 `viewDidLoad()` 方法中添加以下代码：

```swift
alphaTextField.addTarget(self,   action:Selector("keyPressed:"), forControlEvents: UIControlEvents.EditingChanged)
alphaNumericTextField.addTarget(self, action: Selector("keyPressed:"), forControlEvents: UIControlEvents.EditingChanged)
displayNameTextField.addTarget(self, action:           Selector("keyPressed:"), forControlEvents: UIControlEvents.EditingChanged)
```

现在让我们创建 `keyPressed()` 方法，当用户在字段中输入字符时，每个文本字段都会调用此方法：

```swift
@IBAction func keyPressed(textField: UITextField) {
        if let validator = validators[textField] where !validator.validateString(textField.text!) {
            textField.text = validator.getMatchingString(textField.text!)
            messageLabel?.text = validator.validationMessage
        }
    }
```

在这个方法中，我们使用 `if let validator = validators[textField]` 语句来检索特定文本字段的验证器，然后我们使用 `where !validator.validateString(textField.text!)` 语句来验证用户输入的字符串。如果字符串验证失败，我们使用 `getMatchingString()` 方法通过从输入字符串的第一个无效字符开始删除所有字符，并显示来自文本验证类的错误消息来更新文本字段中的文本。如果字符串通过验证，文本字段中的文本保持不变。

在本书的下载代码中，你可以找到一个示例项目，演示如何使用文本验证类型。

# 摘要

在本章中，我们看到了 Swift 如何将协议视为完整的类型。我们还看到了如何在 Swift 中使用协议实现多态。我们通过深入了解协议扩展来结束本章，并看到了我们如何在 Swift 中使用它们。

协议和协议扩展是苹果新协议导向编程范式的骨架。这种新的编程模型有可能改变我们编写和思考代码的方式。虽然我们在这章没有专门介绍协议导向编程，但理解本章的内容为我们提供了学习这种新编程模型所需的关于协议和协议扩展的坚实基础。
