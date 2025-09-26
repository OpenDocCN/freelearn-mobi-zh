

# 第七章：类、结构和枚举

在上一章中，你学习了如何使用函数和闭包来分组指令序列。

是时候考虑如何在代码中表示复杂对象了。例如，考虑一辆车。你可以使用一个`String`常量来存储车名，以及一个`Double`变量来存储车价，但它们之间并没有关联。你已经看到你可以通过分组指令来创建函数和闭包。在本章中，你将学习如何使用**类**和**结构**将常量和变量组合成一个单一实体，以及如何操作它们。你还将学习如何使用**枚举**来分组一组相关值。

到本章结束时，你将学会如何创建和初始化一个类，从现有类创建子类，创建和初始化一个结构，区分类和结构，以及创建枚举。

本章将涵盖以下主题：

+   理解类

+   理解结构

+   理解枚举

# 技术要求

本章的 Xcode 游乐场位于本书代码包的`Chapter07`文件夹中，可以在此处下载：

[`github.com/PacktPublishing/iOS-18-Programming-for-Beginners-Eighth-Edition`](https://github.com/PacktPublishing/iOS-18-Programming-for-Beginners-Eighth-Edition%0D)

查看以下视频，看看代码的实际效果：

[`youtu.be/Yl9UuzSR_oE`](https://youtu.be/Yl9UuzSR_oE%0D)

如果你希望从头开始，创建一个新的游乐场，并将其命名为`ClassesStructuresAndEnumerations`。你可以一边阅读一边输入并运行本章中的所有代码。让我们从学习什么是类以及如何声明和定义它开始。

# 理解类

类对于表示复杂对象非常有用，例如：

+   公司的个别员工信息

+   在电子商务网站上出售的物品

+   你家中用于保险目的的物品

下面是一个类声明和定义的例子：

```swift
class ClassName {
  property1
  property2
  property3
  method1() {
    code
  }
  method2() {
    code
  }
} 
```

每个类都有一个描述性的名称，它包含用于表示对象的变量或常量。与类关联的变量或常量称为**属性**。

类还可以包含执行特定任务的函数。与类关联的函数称为**方法**。

一旦你声明并定义了一个类，你就可以创建该类的**实例**。想象一下，你正在为动物园创建一个应用程序。如果你有一个`Animal`类，你可以使用该类的实例来表示动物园中的不同动物。这些实例的每个属性都将有不同的值。

要了解更多关于类的信息，请访问[`docs.swift.org/swift-book/documentation/the-swift-programming-language/classesandstructures`](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/classesandstructures)。

让我们看看如何使用类。你将学习如何声明和定义类，根据类声明创建实例，并操作这些实例。你将从下一节开始创建一个表示动物的类。

## 创建类声明

让我们声明并定义一个可以存储关于动物详细信息的类。将以下代码添加到你的游乐场中：

```swift
class Animal {
  var name: String = ""
  var sound: String = ""
  var numberOfLegs: Int = 0
  var breathesOxygen: Bool = true
  func makeSound() {
    print(sound)
  }
} 
```

你刚刚声明了一个非常简单的名为 `Animal` 的类。惯例规定，类名应以大写字母开头。这个类有属性来存储动物的名字、它发出的声音、它有多少条腿以及它是否呼吸氧气。这个类还有一个名为 `makeSound()` 的方法，它会将产生的噪音打印到调试区域。

现在你有了 `Animal` 类，让我们在下一节中使用它来创建一个表示动物的实例。

## 创建类的实例

一旦你声明并定义了一个类，你就可以创建该类的实例。现在，你将创建一个表示猫的 `Animal` 类的实例。按照以下步骤操作：

1.  要创建 `Animal` 类的实例，列出所有属性并调用它的 `makeSound()` 方法；在你的类声明之后输入以下代码并运行：

    ```swift
    let cat = Animal()
    print(cat.name)
    print(cat.sound)
    print(cat.numberOfLegs)
    print(cat.breathesOxygen)
    cat.makeSound() 
    ```

你可以通过在实例名称后键入一个点，然后跟随着你想要的属性或方法来访问实例属性和方法。你将在调试区域看到实例属性和方法的值。由于这些值是在创建类时分配的默认值，因此 `name` 和 `sound` 包含空字符串，`numberOfLegs` 包含 `0`，`breathesOxygen` 包含 `true`，而 `makeSound()` 方法打印一个空字符串。

1.  让我们给这个实例的属性分配一些值。按照以下所示修改你的代码：

    ```swift
    let cat = Animal()
    **cat****.****name****=****"Cat"**
    **cat****.****sound****=****"Mew"**
    **cat****.****numberOfLegs****=****4**
    **cat****.****breathesOxygen****=****true**
    print(cat.name) 
    ```

现在，当你运行程序时，以下内容将在调试区域显示：

```swift
Cat
Mew
4
true
Mew 
```

所有实例属性值和 `makeSound()` 方法的输出都将打印到调试区域。

注意，在这里，你首先创建实例，然后分配值给该实例。也可以在创建实例时分配值，这可以通过在类声明中实现一个 **初始化器** 来完成。

1.  初始化器负责确保在创建类时所有实例属性都有有效的值。让我们为 `Animal` 类添加一个初始化器。按照以下所示修改你的类定义：

    ```swift
    class Animal {
      var name: String
      var sound: String
      var numberOfLegs: Int
      var breathesOxygen: Bool
    **init****(****name****:** **String****,** **sound****:** **String****,** **numberOfLegs****:**
    **Int****,** **breathesOxygen****:** **Bool****) {**
    **self****.****name****=** **name**
    **self****.****sound****=** **sound**
    **self****.****numberOfLegs****=** **numberOfLegs**
    **self****.****breathesOxygen****=** **breathesOxygen**
     **}**
      func makeSound() {
        print(sound)
      }
    } 
    ```

如你所见，初始化器使用了 `init` 关键字，并有一个参数列表，这些参数将用于设置属性值。请注意，`self` 关键字区分了属性名和参数。例如，`self.name` 指的是属性，而 `name` 指的是参数。

在初始化过程结束时，类中的每个属性都应该有一个有效的值。

1.  在这个阶段，你会在代码中看到一些错误，因为函数调用没有参数。你需要更新你的函数调用以解决这个问题。按照以下所示修改你的代码并运行：

    ```swift
     func makeSound() {
        print(sound)
      }
    }
    let cat = Animal(**name****:** **"Cat"****,** **sound****:** **"Mew"****,** **numberOfLegs****:** **4****,** **breathesOxygen****:** **true**)
    print(cat.name) 
    ```

结果与*步骤 2*中的相同，但你在一个指令中创建了实例并设置了其属性。太棒了！

现在有不同类型的动物，如哺乳动物、鸟类、爬行动物和鱼类。你可以为每种类型创建一个类，但你也可以基于现有类创建一个**子类**。让我们在下一节中看看如何做到这一点。

## 创建子类

一个类的子类继承了一个现有类的所有方法和属性。如果你愿意，你还可以向其中添加额外的属性和方法。例如，对于一家 IT 公司，你可以有`CustomerSupportAgent`作为`Employee`类的子类。这个类将具有`Employee`类的所有属性，以及客户支持角色所需的额外属性。

现在，你将创建`Mammal`，它是`Animal`类的子类。按照以下步骤进行：

1.  要声明`Mammal`类，在`Animal`类声明之后键入以下代码：

    ```swift
    class Mammal: Animal {
      let hasFurOrHair: Bool = true
    } 
    ```

在类名后面键入`: Animal`会使`Mammal`类成为`Animal`类的子类。它具有在`Animal`类中声明的所有属性和方法，以及一个额外的属性：`hasFurOrHair`。由于`Animal`类是`Mammal`类的父类，你可以将其称为`Mammal`类的超类。

1.  按照以下所示修改创建你的类实例的代码，然后运行它：

    ```swift
    let cat = **Mammal**(name: "Cat", sound: "Mew", numberOfLegs: 4, breathesOxygen: true) 
    ```

`cat`现在是一个`Mammal`类的实例，而不是`Animal`类的实例。正如你所看到的，调试区域显示的结果与之前相同，没有错误。不过，`hasFurOrHair`的值尚未显示。让我们修复这个问题。

1.  在你的游乐场中的所有其他代码之后键入以下代码以显示`hasFurOrHair`属性的值并运行它：

    ```swift
    print(cat.hasFurOrHair) 
    ```

由于`Animal`类的初始化器没有参数来分配`hasFurOrHair`的值，将使用默认值，调试区域将显示`true`。

你已经看到子类可以具有额外的属性。子类还可以具有额外的属性和方法，子类中的方法实现可以与超类实现不同。让我们在下一节中看看如何做到这一点。

## 覆盖超类方法

到目前为止，你一直在使用多个`print()`语句来显示类实例的值。你将实现一个`description()`方法来在调试区域显示所有实例属性，因此不再需要多个`print()`语句。按照以下步骤进行：

1.  按照以下所示修改你的`Animal`类声明以实现`description()`方法：

    ```swift
    class Animal {
      var name: String
      var sound: String
      var numberOfLegs: Int
      var breathesOxygen: Bool = true
      init(name: String, sound: String, numberOfLegs:
      Int, breathesOxygen: Bool) {
        self.name = name
        self.sound = sound
        self.numberOfLegs = numberOfLegs
        self.breathesOxygen = breathesOxygen
      }
      func makeSound() {
        print(sound)
      }
    **func****description****() ->** **String** **{**
    **"****name:** **\(****name****)****sound:** **\(****sound****)**
     **numberOfLegs:** **\(****numberOfLegs****)**
     **breathesOxygen:** **\(****breathesOxygen****)****"**
     **}**
    } 
    ```

1.  按照以下所示修改你的代码，用`description()`方法代替多个`print()`语句，然后运行程序：

    ```swift
    let cat = Mammal(name: "Cat", sound: "Mew",
    numberOfLegs: 4, breathesOxygen: true)
    **print(****cat****.****description****())**
    cat.makeSound() 
    ```

你将在调试区域看到以下内容：

```swift
name: Cat sound: Mew numberOfLegs: 4 breathesOxygen: true
Mew 
```

如您所见，尽管`description()`方法在`Mammal`类中没有实现，但它却在`Animal`类中实现了。这意味着它将被`Mammal`类继承，并且实例属性将被打印到调试区域。请注意，`hasFurOrHair`属性的值缺失，并且您不能将其放入`description()`方法中，因为`Animal`类中没有`hasFurOrHair`属性。

1.  您可以更改`Mammal`类中`description()`方法的实现，以显示`hasFurOrHair`属性的值。将以下代码添加到您的`Mammal`类定义中并运行它：

    ```swift
    class Mammal: Animal {
      let hasFurOrHair: Bool = true
    **override****func****description****() ->** **String** **{**
    **super****.****description****()** **+****" hasFurOrHair:**
    **\(****hasFurOrHair****)"**
     **}**
    } 
    ```

`override`关键字在这里用于指定实现的`description()`方法将替换超类实现。`super`关键字用于调用超类的`description()`实现。然后，`hasFurOrHair`的值被添加到`super.description()`返回的字符串中。

您将在调试区域看到以下内容：

```swift
name: Cat sound: Mew numberOfLegs: 4 breathesOxygen: true hasFurOrHair: true
Mew 
```

`hasFurOrHair`属性的值在调试区域中显示，表明您正在使用`Mammal`子类的`description()`方法实现。

您已经创建了类和子类声明，并为两者创建了实例。您还为两者添加了初始化器和方法。太酷了！让我们在下一节中看看如何声明和使用结构体。

# 理解结构体

与类一样，结构体也组合了用于表示对象和执行特定任务的属性和方法。还记得您创建的`Animal`类吗？您也可以使用结构体来完成相同的事情。不过，类和结构体之间还是有区别的，您将在本章后面了解更多。

下面是结构体声明和定义的示例：

```swift
struct StructName {
  property1
  property2
  property3
  method1() {
    code
  }
  method2(){
    code
  }
} 
```

如您所见，结构体与类非常相似。它也有一个描述性的名称，可以包含属性和方法。您也可以创建结构体的实例。

想要了解更多关于结构体的信息，请访问[`docs.swift.org/swift-book/documentation/the-swift-programming-language/classesandstructures`](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/classesandstructures)。

让我们看看如何与结构体一起工作。您将学习如何声明和定义结构体，基于结构体创建实例，并对其进行操作。您将在下一节中创建一个结构体来表示爬行动物。

## 创建结构体声明

继续动物主题，让我们声明并定义一个可以存储关于爬行动物详细信息的结构体。在您的游乐场中所有其他代码之后添加以下代码：

```swift
struct Reptile {
  var name: String
  var sound: String
  var numberOfLegs: Int
  var breathesOxygen: Bool
  let hasFurOrHair: Bool = false
  func makeSound() {
    print(sound)
  }
  func description() -> String {
    "Structure: Reptile name: \(name)
    sound: \(sound)
    numberOfLegs: \(numberOfLegs)
    breathesOxygen: \(breathesOxygen)
    hasFurOrHair: \(hasFurOrHair)"
  }
} 
```

如你所见，这几乎与你之前所做的`Animal`类声明相同。结构体的名称也应该以大写字母开头，并且这个结构体有属性来存储动物的名字、它发出的声音、它有多少条腿、它是否呼吸氧气以及它是否有毛皮或毛发。这个结构体还有一个`makeSound()`方法，它会将发出的声音打印到调试区域。

现在你有了`Reptile`结构体的声明，让我们在下一节中使用它来创建一个表示蛇的实例。

## 创建结构体的实例

与类一样，你可以从结构体声明中创建实例。现在，你将创建一个表示蛇的`Reptile`结构体的实例，打印出该实例的属性值，并调用`makeSound()`方法。在你的游乐场中的所有其他代码之后输入以下内容并运行它：

```swift
var snake = Reptile(name: "Snake", sound: "Hiss",
numberOfLegs: 0, breathesOxygen: true)
print(snake.description())
snake.makeSound() 
```

注意，你不需要实现一个初始化器；结构体自动为其所有属性获得一个名为**成员初始化器**的初始化器。真方便！以下内容将在调试区域显示：

```swift
Structure: Reptile name: Snake sound: Hiss numberOfLegs: 0 breathesOxygen: true hasFurOrHair: false
Hiss 
```

尽管结构体声明与类声明非常相似，但类和结构体之间有两个区别：

+   结构体不能从另一个结构体继承

+   类是**引用类型**，而结构体是**值类型**

让我们看看下一节中值类型和引用类型之间的区别。

## 比较值类型和引用类型

类是引用类型。这意味着当你将类实例赋值给变量时，你是在变量中存储原始实例的内存位置，而不是实例本身。

结构体是值类型。这意味着当你将结构体实例赋值给变量时，该实例被复制，你对原始实例所做的任何更改都不会影响副本。

现在，你将创建一个类的实例和一个结构体的实例，并观察它们之间的差异。按照以下步骤操作：

1.  你将首先创建一个包含结构体实例的变量，并将其赋值给第二个变量，然后更改第二个变量中属性的值。输入以下代码并运行它：

    ```swift
    struct SampleValueType {
      var sampleProperty = 10
    }
    var a = SampleValueType()
    var b = a
    b.sampleProperty = 20
    print(a.sampleProperty)
    print(b.sampleProperty) 
    ```

在这个例子中，你声明了一个包含一个属性`sampleProperty`的结构体`SampleValueType`。然后，你创建了该结构体的一个实例并将其赋值给变量`a`。之后，你将`a`赋值给一个新的变量`b`。然后，你将`b`的`sampleProperty`值更改为`20`。

当你打印出`a`的`sampleProperty`值时，调试区域将显示`10`，这表明对`b`的`sampleProperty`值的任何更改都不会影响`a`的`sampleProperty`值。这是因为当你将`a`赋值给`b`时，`a`的一个副本被赋值给`b`，因此它们是独立的实例，不会相互影响。

1.  接下来，你将创建一个包含类实例的变量，并将其分配给第二个变量，然后更改第二个变量的属性值。输入以下代码并运行它：

    ```swift
    class SampleReferenceType {
      var sampleProperty = 10
    }
    var c = SampleReferenceType()
    var d = c
    c.sampleProperty = 20
    print(c.sampleProperty)
    print(d.sampleProperty) 
    ```

在这个例子中，你声明了一个包含一个属性 `sampleProperty` 的类 `SampleReferenceType`。然后，你创建了该类的实例并将其分配给一个变量 `c`。之后，你将 `c` 分配给一个新的变量 `d`。接下来，你将 `d` 的 `sampleProperty` 值更改为 `20`。

当你打印出 `c` 的 `sampleProperty` 值时，在调试区域中会打印出 `20`，这表明对 `c` 或 `d` 的任何更改都会影响相同的 `SampleReferenceType` 实例。

现在，问题是，你应该使用类还是结构体？让我们在下一节中探讨这个问题。

## 在类和结构体之间做出选择

你已经看到你可以使用类或结构体来表示复杂对象。那么，你应该使用哪一个呢？

除非你需要类才能实现某些功能，例如子类，否则建议使用结构体。这有助于防止由于类是引用类型而可能发生的某些微妙错误。

太棒了！现在你已经了解了类和结构体，让我们看看枚举，它允许你将相关值分组，在下一节中。

# 理解枚举

枚举允许你将相关值分组，例如以下内容：

+   指南针方向

+   交通信号灯颜色

+   彩虹的颜色

为了理解为什么枚举对于这个目的来说非常理想，让我们考虑以下示例。

想象你正在编写交通信号灯的代码。你可以使用一个整型变量来表示不同的交通信号灯颜色，其中 `0` 代表红色，`1` 代表黄色，`2` 代表绿色，如下所示：

```swift
var trafficLightColor = 2 
```

虽然这是一种表示交通信号灯的方法，但当将 `3` 分配给 `trafficLightColor` 时会发生什么？这是一个问题，因为 `3` 并不代表有效的交通信号灯颜色。因此，如果我们能将 `trafficLightColor` 的可能值限制为它可以显示的颜色，那就更好了。

下面是一个枚举声明和定义的例子：

```swift
enum EnumName {
  case value1
  case value2
  case value3
} 
```

每个枚举都有一个描述性名称，其主体包含该枚举的关联值。

要了解更多关于枚举的信息，请访问 [`docs.swift.org/swift-book/documentation/the-swift-programming-language/enumerations`](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/enumerations)。

让我们看看如何使用枚举。你将学习如何创建和操作它们。你将从下一节创建一个表示交通信号灯颜色的枚举开始。

## 创建一个枚举

让我们创建一个枚举来表示交通信号灯。按照以下步骤操作：

1.  将以下代码添加到您的游乐场中并运行它：

    ```swift
    enum TrafficLightColor {
      case red
      case yellow
      case green
    }
    var trafficLightColor = TrafficLightColor.red 
    ```

这创建了一个名为 `TrafficLightColor` 的枚举，它将红色、黄色和绿色值分组在一起。`trafficLightColor` 变量的值限制为 `red`、`yellow` 和 `green`；设置任何其他值将生成错误。

1.  就像类和结构体一样，枚举也可以包含方法。让我们给 `TrafficLightColor` 添加一个方法。按照以下所示修改你的代码，使 `TrafficLightColor` 返回一个表示交通灯颜色的字符串，并运行它：

    ```swift
    enum TrafficLightColor {
      case red
      case yellow
      case green
    **func****description****() ->** **String** **{**
    **switch****self** **{**
    **case****.red****:**
    **"red"**
    **case****.yellow****:**
    **"yellow"**
     **case .green:**
     **"green"**
     **}**
     **}**
    }
    var trafficLightColor = TrafficLightColor.red
    **print(****trafficLightColor****.****description****())** 
    ```

`description()` 方法返回一个字符串，取决于 `trafficLightColor` 的值。由于 `trafficLightColor` 的值为 `TrafficLightColor.red`，**红色**将出现在调试区域。

你已经学会了如何创建和使用枚举来存储分组值，以及如何向它们添加方法。做得好！

# 摘要

在本章中，你学习了如何使用类声明复杂对象，如何创建类的实例，如何创建子类，以及如何重写类方法。你还学习了如何声明结构体，创建结构体的实例，以及理解引用类型和值类型之间的区别。最后，你学习了如何使用枚举来表示一组特定的值。

现在，你知道了如何使用类和结构体来表示复杂对象，以及如何在你的程序中使用枚举将相关值分组在一起。

在下一章中，你将学习如何使用协议在类和结构体中指定常见特性，如何使用扩展扩展内置类的功能，以及如何在程序中处理错误。

# 加入我们的 Discord 社区！

与其他用户、专家和作者本人一起阅读这本书。提问，为其他读者提供解决方案，通过 Ask Me Anything 会话与作者聊天，等等。扫描二维码或访问链接加入社区。

[`packt.link/ios-Swift`](https://packt.link/ios-Swift%0D)

![](https://packt.link/ios-Swift%0D)
