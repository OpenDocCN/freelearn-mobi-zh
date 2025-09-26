# 第十章。使用可选类型

当我开始使用 Swift 时，我遇到的最大难题是可选类型。由于我的背景是 Objective-C、C、Java 和 Python，我能够将 Swift 的许多特性与我所知道的另一种语言中的工作方式联系起来，但可选类型却不同。在我使用的其他语言中，真的没有什么与可选类型相似的东西，因此我花了很多时间去完全理解它们。虽然我在第二章中简要介绍了可选类型，即“学习变量、常量、字符串和运算符”，这足以让我们开始学习，但我们还需要涵盖更多信息，才能真正理解可选类型是什么，如何正确使用它们，以及为什么它们在 Swift 语言中如此重要。

在本章中，我们将涵盖以下主题：

+   可选类型的简介

+   Swift 中可选类型的必要性

+   解包可选类型

+   可选绑定

+   可选链

# 介绍可选类型

当我们在 Swift 中声明变量时，它们默认是非可选的，这意味着它们必须包含一个有效、非 nil 的值。如果我们尝试将非可选变量设置为 nil，将会导致一个 `Type '{type}' does not conform to protocol 'NilLiteralConvertible'` 错误，其中 `{type}` 是变量的类型。

例如，以下代码在尝试将 `message` 变量设置为 `nil` 时将抛出错误，因为 `message` 是一个非可选类型：

```swift
var message: String = "My String"
message = nil
```

非常重要的是要理解 Swift 中的 nil 与 Objective-C 中的 nil 非常不同。在 Objective-C 中，nil 是指向不存在对象的指针；然而，在 Swift 中，nil 是值的缺失。这个概念对于完全理解 Swift 中的可选类型非常重要。

定义为可选的变量可以包含一个有效值，也可以表示值的缺失。我们通过将其分配一个特殊的 nil 值来表示值的缺失。任何类型的可选类型都可以设置为 nil，而在 Objective-C 中，只有对象可以设置为 nil。

要真正理解可选类型背后的概念，让我们看看定义可选类型的一行代码：

```swift
var myString: String?
```

末尾的问号表示 `myString` 变量是一个可选类型。当我们查看这段代码时，将这一行代码读作“`myString` 变量是一个可选的字符串类型”是不正确的。实际上，我们应该将这一行代码读作“`myString` 变量是一个可选类型，它可能包含一个字符串类型，也可能不包含任何值”。这两行代码之间的细微差别实际上在理解可选类型的工作方式上有着很大的不同。

可选类型是 Swift 中的一个特殊类型。当我们定义 `myString` 变量时，我们实际上将其定义为可选类型。为了理解这一点，让我们看看一些更多的代码：

```swift
var myString1: String?
var myString2: Optional<String>
```

这两个声明是等效的。两行都声明了一个可能包含字符串类型或可能没有值的可选类型。在 Swift 中，我们可以将值的缺失视为设置为 `nil`，但始终要记住，这与在 Objective-C 中设置 `nil` 是不同的。在这本书中，当我们提到 `nil` 时，我们是指 Swift 如何使用 `nil`，而不是 Objective-C 如何使用 `nil`。

可选类型是一个有两个可能值的枚举，`None` 和 `Some(T)`，其中 `T` 是相关类型的关联值。如果我们将可选设置为 `nil`，那么它将具有 `None` 的值，如果我们设置一个值，那么可选将具有 `Some` 的值，并带有相关类型的关联值。在 第二章，*学习变量、常量、字符串和运算符* 中，我们解释了 Swift 中的枚举可以有关联值。关联值允许我们在枚举成员值中存储额外的信息。

内部，可选类型定义如下：

```swift
enum Optional<T> {
  case None
  case Some(T)
}
```

在这里，`T` 是与可选类型关联的类型。`T` 符号用于定义泛型。我们可以在 第十一章，*使用泛型* 中了解更多关于泛型的信息。

# Swift 中可选类型的需求

现在，最迫切的问题是为什么 Swift 需要可选类型？为了理解为什么 Swift 有可选类型，我们应该检查可选类型旨在解决什么问题。

在大多数语言中，可以在不提供初始化值的情况下创建一个变量。例如，在 Objective-C 中，两行代码都是有效的：

```swift
int i;
MyObject *m;
```

现在，假设 `MyObject` 类有如下方法：

```swift
-(int)myMethodWithValue:(int)i {
    return i*2;
}
```

这个方法接受从 `i` 参数传递的值，将其乘以 2，并返回结果。让我们尝试使用以下代码调用这个方法：

```swift
MyObject *m;
NSLog(@"Value: %d",[m myMethodWithValue:5]);
```

我们的第一反应可能是认为这段代码会显示 `Value: 10`；然而，这是错误的。实际上，这段代码会显示 `Value: 0`，因为我们没有在使用之前初始化 `m` 对象。

当我们忘记初始化一个对象或为变量设置值时，我们可能会在运行时得到意外的结果，就像我们刚才演示的那样。有时，这些意外的结果可能非常难以追踪。

使用可选类型，Swift 能够在编译时检测到这类问题，并在运行时提醒我们。如果我们期望在使用变量或对象之前它始终包含一个值，我们将变量声明为非可选类型（这是默认声明）。然后，在编译时，如果我们尝试在使用之前初始化它，我们会收到一个错误。让我们看看这个例子。以下代码会在编译时显示错误，因为我们试图在使用之前初始化一个非可选变量：

```swift
var myString: String
print(myString)
```

如果一个变量被声明为可选类型，在尝试使用它之前验证它是否包含有效的值是良好的编程实践。我们只应该声明变量为可选类型，如果确实有理由让变量不包含值。这也是 Swift 默认将变量声明为非可选类型的原因。

现在我们（希望）已经很好地理解了可选类型是什么以及它们旨在解决哪些类型的问题，让我们看看如何使用它们。

## 定义可选类型

通常，要定义可选类型，我们在变量声明中的类型后放置一个问号。请注意，我们在变量声明中定义的类型实际上是可选枚举中的关联值。以下代码展示了我们通常如何声明可选类型：

```swift
var myOptional: String?
```

这段代码声明了一个可能包含字符串或无值（nil）的可选变量。当一个变量以这种方式声明时，默认情况下，它被设置为无值。

## 使用可选类型

我们可以在代码中用多种方式使用可选类型。使用可选类型的关键是在访问之前始终验证它们是否包含有效的值。我们使用解包这个术语来指代检索可选类型的值。

### 强制解包可选类型

要解包或检索可选类型的值，我们在变量名后放置一个感叹号（`!`）。以这种方式强制解包可能非常危险，并且只有在我们确定值不是 nil 的情况下才应使用。

当我们使用感叹号来解包可选类型时，我们是在告诉编译器我们知道可选类型不包含 nil 值，因此请继续并给我们值。让我们看看如何做到这一点：

```swift
var myString1: String?
myString1 = "test"
var test: String = myString1!
```

这段代码将按预期工作，其中`test`变量将包含`test`字符串；然而，如果设置`myString1`可选类型为`test`的行被移除，当我们运行应用程序时，我们会收到运行时错误。请注意，编译器不会提醒我们问题，因为我们正在使用感叹号来解包可选类型；因此，编译器假设我们知道我们在做什么，并会愉快地为我们编译代码。我们应该在解包`myString1`可选类型之前验证它是否包含有效的值。以下示例是这样做的一种方式：

```swift
var myString1: String?
myString1 = "test"
if myString1 != nil {
    var test:String = myString1!
}
```

现在，如果设置`myString1`可选类型为`test`的行被移除，我们不会收到运行时错误，因为我们只解包了`myString`可选类型，如果它包含一个有效的（非 nil）值。

如我们刚才所描述的，解包可选类型并不是最理想的方式，我们不推荐以这种方式解包可选类型。我们可以将验证和解包合并为一步，称为**可选绑定**。

### 可选绑定

可选绑定是解包可选推荐的方式。使用可选绑定，我们执行检查以查看可选是否包含有效值，如果是，则将其解包到临时变量或常量中。这一切都在一个步骤中完成。

可选绑定使用`if`或`while`条件语句执行。如果我们想将可选的值放入常量中，其格式如下：

```swift
if let constantName = optional {
   statements
}
```

如果我们需要将值放入变量而不是常量中，可以使用`var`关键字而不是`let`关键字，如下例所示：

```swift
if var variableName = optional {
   statements
}
```

以下示例展示了如何执行可选绑定：

```swift
var myString3: String?
myString3 = "Space"
if let tempVar = myString3 {
    print(tempVar)
} else {
    print("No value")
}
```

在示例中，我们将`myString3`变量定义为可选类型。如果`myString3`可选包含有效值，则我们将新变量`tempvar`设置为`myString3`可选的值，并将值打印到控制台。如果`myString3`可选不包含值，则打印`No`值到控制台。

从 Swift 1.2 开始，我们能够使用可选绑定在同一可选绑定行中解包多个可选。例如，如果我们有三个名为`optional1`、`optional2`和`optional3`的可选，我们可以使用以下代码一次性尝试解包所有三个：

```swift
If let tmp1 = optional1, tmp2 = optional2, tmp3 = optional3 {
}
```

如果三个可选中的任何一个解包失败，整个可选绑定语句将失败。

使用可选绑定将值赋给同名变量是完全可接受的。以下代码说明了这一点：

```swift
if let myOptional = myOptional {
  print(myOptional)
} else {
  print("myOptional was nil")
}
```

需要注意的一点是，`temp`变量仅在条件块内作用域，不能在条件块外使用。为了说明临时变量的作用域，让我们看一下以下代码：

```swift
var myOptional: String?
myOptional = "test"
if var tmp = myOptional {
    print("Inside:  \(tmp)")
}
// This next line will cause a compile time error
print("Outside: \(tmp)")
```

此代码无法编译，因为`tmp`变量仅在条件块内有效，而我们试图在条件块外使用它。

使用可选绑定比手动验证可选是否有值然后强制解包可选要干净和简单得多。

### 从函数、方法和子脚本返回可选

我们可以将函数或方法的返回类型设置为可选类型。这允许我们从函数或方法返回 nil（无值）。要设置返回类型为可选类型，我们将在函数或方法声明中的类型名称后插入一个问号。

以下示例展示了我们如何从函数或方法返回可选：

```swift
func getName(index: Int) -> String? {
    let names = ["Jon", "Kim", "Kailey", "Kara"]
    if index >= names.count || index < 0 {
        return nil
    } else {
        return names[index]
    }
}
```

在示例中，我们将返回类型定义为可选，可以是字符串值或无值。在函数内部，如果索引在数组范围内，我们将返回名称；如果索引超出数组范围，则返回`nil`。

以下代码展示了如何调用此函数，其中返回值是可选的：

```swift
var name = getName(2)
var name2 = getName(5)
```

在之前的代码中，`name`变量将包含`Kailey`，而`name2`变量将包含`nil`（无值）。注意，我们不需要将变量定义为可选的（使用问号），因为 Swift 知道它是一个可选类型，因为这是函数定义的返回类型。

我们还可以定义一个返回可选类型的下标。我们定义下标的方式与定义函数的方式相同。以下是一个返回可选的下标示例模板：

```swift
subscript(index: Int) -> String? {
   //some statements
}
```

根据这个定义，我们能够从我们的下标返回一个`nil`（无值）。

### 在函数或方法中使用可选作为参数

我们还可以将可选作为函数或方法的参数。这允许我们在需要时将`nil`（无值）传递给函数或方法。以下示例展示了如何为函数定义可选参数：

```swift
func optionalParam(myString: String?) {
  if let temp = myString {
    print("Contains value \(temp)")
  }
  else {
    print("Does not contain value")
  }
}
```

要将参数定义为可选类型，我们在参数定义中使用问号。在这个示例中，我们使用可选绑定来检查可选是否包含值。如果包含值，我们在控制台打印`Contains value`；否则，我们打印`Does not contain value`。

### 元组中的可选类型

我们可以将整个元组定义为可选的，或者将元组内的任何元素定义为可选的。当从函数或方法返回元组时，使用可选与元组结合特别有用。这允许我们返回元组的一部分（或全部）作为`nil`。以下示例展示了如何将元组定义为可选的，以及如何将元组的单个元素定义为可选类型：

```swift
var tuple1: (one: String, two: Int)?
var tuple2: (one: String, two: Int?)
```

第一行将整个元组定义为可选类型。第二行将元组内的第二个值定义为可选，而第一个值是非可选的。

# 可选链

可选绑定允许我们一次解包一个可选，但如果可选类型嵌套在其他可选类型中会怎样？这将迫使我们必须在其他可选绑定语句中嵌入可选绑定语句。有一种更好的方法来处理这个问题，那就是使用可选链。在我们查看可选链之前，让我们看看这是如何与可选绑定一起工作的：

```swift
class Collar {
    var color: String
    init(color: String) {
        self.color = color
    }
}

class Pet {
    var name: String
    var collar: Collar?
    init(name: String) {
        self.name = name
    }
}

class Person {
    var name: String
    var pet: Pet?
    init(name: String) {
        self.name = name
    }
}
```

在这个示例中，我们首先定义一个`Collar`类，它有一个属性定义。这个属性名为`color`，其类型为字符串。我们可以看到`color`属性不是可选的；因此，我们可以安全地假设它将始终有一个有效的值。

接下来，我们定义一个`Pet`类，它有两个属性定义。这些属性分别命名为`name`和`collar`。`name`属性是字符串类型，而`collar`属性是可选的，可能包含一个`Collar`类型对象。

最后，我们定义一个`Person`类，它也有两个属性。这些属性分别命名为`name`和`pet`。`name`属性是字符串类型，而`pet`属性是可选的，可能包含一个`Pet`类型对象。

对于接下来的示例，让我们使用以下代码来初始化类：

```swift
var jon = Person(name: "Jon")
var buddy = Pet(name: "Buddy")
jon.pet = buddy
var collar = Collar(color: "red")
buddy.collar = collar
```

现在，假设我们想获取某人的宠物的颜色；然而，这个人可能没有宠物（`pet` 属性为 `nil`）或者宠物可能没有项圈（`collar` 属性为 `nil`）。我们可以使用可选绑定来钻入每一层，如下例所示：

```swift
if let tmpPet = jon.pet, tmpCollar = tmpPet.collar {
   print("The color of the collar is \(tmpCollar.color)")
  }
else {
    print("Cannot retrieve color")
}
```

虽然这个例子完全有效，并且会打印出消息`The color of the collar is red`，但代码相当混乱且难以理解，因为我们嵌套了可选绑定语句。

可选链允许我们在一行代码中钻入多个可选类型层级的属性、方法和子脚本。这些层级可以连在一起，如果任何一层返回 `nil`，整个链将优雅地失败并返回 `nil`。如果没有值返回 `nil`，则返回链的最后一个值。由于可选链的结果可能是一个 `nil` 值，结果总是以可选类型返回，即使我们检索的最终值不是可选类型。

要指定可选链，我们将在链中的每个可选值后面放置一个问号 (`?`)。以下示例展示了如何使用可选链使前一个示例更简洁、更容易阅读：

```swift
if let color = jon.pet?.collar?.color {
    print("The color of the collar is \(color)")
} else {
    print("Cannot retrieve color")
}
```

在这个例子中，我们在 `pet` 和 `collar` 属性后面加上问号，表示它们是可选类型，如果任一值是 `nil`，整个链将返回 `nil`。这段代码也会打印出消息`The color of the collar is red`；然而，它比使用可选绑定的前一个例子更容易阅读。

## 空值合并运算符

空值合并运算符与我们在本书第二章学习变量、常量、字符串和运算符中讨论的三元运算符类似。三元运算符根据比较运算符或布尔值的评估给变量赋值。空值合并运算符解包可选值，如果它包含一个值，则返回该值，如果可选值为 `nil`，则返回默认值。

让我们看看空值合并运算符的原型：

```swift
optionalA ?? defaultValue
```

在这个例子中，我们演示了当可选值包含 `nil` 和包含值时如何使用空值合并运算符：

```swift
var defaultName = "Jon"

var optionalA: String?
var optionalB: String?

optionalB = "Buddy"

var nameA = optionalA ?? defaultName
var nameB = optionalB ?? defaultName
```

在这个例子中，我们首先将 `defaultName` 变量初始化为 `Jon`。然后我们定义了两个名为 `optionalA` 和 `optionalB` 的可选值。`optionalA` 变量将被设置为 `nil`，而 `optionalB` 变量将被设置为 `Buddy`。

nil 合并运算符在最后两行中使用。由于 `optionalA` 变量包含一个 `nil`，`nameA` 变量将被设置为 `defaultName` 变量的值，即 `Jon`。`nameB` 变量将被设置为 `optionalB` 变量的值，因为它包含一个值。

# 摘要

虽然 Swift 语言中使用的可选类型的概念一开始可能有些陌生，但随着你使用它们的频率增加，它们会变得更加有意义。可选类型的一个最大优点是我们得到了额外的编译时检查，这会提醒我们在使用之前是否忘记初始化非可选变量。

本章要掌握的一个要点是可选的概念。为了加强这个概念，让我们回顾一下本章的一些段落。

非常重要的是要理解 Swift 中的 nil 与 Objective-C 中的 nil 非常不同。在 Objective-C 中，nil 是一个指向不存在对象的指针；然而，在 Swift 中 nil 是一个值的缺失。这个概念对于完全理解 Swift 中的可选类型非常重要。

被定义为可选的变量可以包含一个有效值，或者它可能没有值。我们通过将 Swift 的特殊 nil 值赋给变量来设置变量为无值状态。任何类型的可选变量都可以设置为 nil，而在 Objective-C 中，只有对象可以被设置为 nil。

可选类型是一个有两个可能值的枚举，`None` 和 `Some (T)`，其中 `T` 是相关类型的关联值。如果我们将可选设置为 `nil`，它将具有 `None` 的值，如果我们设置一个值，可选将具有 `Some` 的值，并带有相关类型的关联值。在 第二章，*学习变量、常量、字符串和运算符* 中，我们解释了 Swift 中的枚举可以有关联值。关联值允许我们在枚举成员值中存储额外的信息。
