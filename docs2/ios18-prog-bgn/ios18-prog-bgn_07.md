# 6

# 函数和闭包

到目前为止，你可以编写相对复杂的程序，这些程序可以做出决策并重复指令序列。你还可以使用集合类型为你的程序存储数据。随着你编写的程序在大小和复杂性上增长，理解它们所做的工作将变得更加困难。

为了使大型程序更容易理解，Swift 允许你创建**函数**，这让你可以通过调用单个名称来组合多个指令并执行它们。你还可以创建**闭包**，这让你可以组合多个指令而不命名，并将它们分配给常量或变量。

到本章结束时，你将了解函数、嵌套函数、作为返回类型的函数、作为参数的函数以及`guard`语句。你还将了解如何创建和使用闭包。

本章将涵盖以下主题：

+   探索函数

+   探索闭包

# 技术要求

本章的 Xcode 游乐场位于本书代码包的`Chapter06`文件夹中，可以在此处下载：

[`github.com/PacktPublishing/iOS-18-Programming-for-Beginners-Ninth-Edition`](https://github.com/PacktPublishing/iOS-18-Programming-for-Beginners-Ninth-Edition)

查看以下视频，看看代码的实际效果：

[`youtu.be/lgPDkddf_tc`](https://youtu.be/lgPDkddf_tc)

如果你希望从头开始，创建一个新的游乐场，并将其命名为`FunctionsAndClosures`。

你可以在进行过程中输入并运行本章中的所有代码。让我们先了解函数。

# 探索函数

函数对于封装执行特定任务的多个指令非常有用，如下所示：

+   计算餐厅餐费的 10%服务费

+   计算你希望购买的汽车的月供

这就是函数的样子：

```swift
func functionName(parameter1: ParameterType, ...) -> ReturnType {
  code
} 
```

每个函数都有一个描述性的名称。你可以定义一个或多个作为输入值的函数，这些值被称为**参数**。你还可以定义函数完成后的输出，这被称为其**返回类型**。参数和返回类型都是可选的。

你“调用”函数的名称来执行它。这就是函数调用的样子：

```swift
functionName(parameter1: argument1, …) 
```

你提供与函数参数类型匹配的输入值（称为**参数**）。

要了解更多关于函数的信息，请访问[`docs.swift.org/swift-book/documentation/the-swift-programming-language/functions/`](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/functions/)。

在下一节中，我们将看看如何创建一个函数来计算服务费。

## 创建函数

在其最简单的形式中，函数只是执行一些指令，没有任何参数或返回类型。你将通过编写一个计算餐费服务费的函数来了解这是如何工作的。服务费应该是餐费的 10%。

将以下代码添加到你的游乐场中，创建并调用此函数并运行它：

```swift
func serviceCharge() {
  let mealCost = 50
  let serviceCharge = mealCost / 10
  print("Service charge is \(serviceCharge)")
}
serviceCharge() 
```

你刚刚创建了一个名为`serviceCharge()`的非常简单的函数。它所做的只是计算价值 50 美元的餐费的 10%服务费，即`50 / 10`，返回`5`。然后你使用其名称调用此函数。你将在调试区域看到**服务费是 5**的显示。

这个函数不是很实用，因为每次调用这个函数时`mealCost`总是`50`。此外，结果只打印在调试区域，不能在程序的其他地方使用。让我们给这个函数添加一些参数和一个返回类型，使其更有用。

按照以下所示修改你的代码：

```swift
func serviceCharge(**mealCost****:** **Int****) ->** **Int**{
  **return** **mealCost** **/****10**
}
**let****serviceChargeAmount****=****serviceCharge****(****mealCost****:** **50****)**
**print(****serviceChargeAmount****)** 
```

这样就更好了。现在，你可以在调用`serviceCharge(mealCost:)`函数时设置餐费，并将结果分配给一个变量或常量。尽管这样看起来有点别扭。你应该尝试使 Swift 中的函数签名读起来像英语句子，因为这被认为是一种最佳实践。让我们看看如何在下一节中做到这一点，你将使用**自定义标签**使你的函数更符合英语风格，更容易理解。

## 使用自定义参数标签

注意，`serviceCharge(mealCost:)`函数不太符合英语风格。你可以给参数添加一个自定义标签，使函数更容易理解。

按照以下所示修改你的代码：

```swift
func serviceCharge(**forMealPrice** mealCost: Int) -> Int {
  mealCost / 10
}
let serviceChargeAmount = serviceCharge(**forMealPrice**: 50)
print(serviceChargeAmount) 
```

函数的工作方式与之前相同，但调用它时，你使用`serviceCharge(forMealPrice:)`。这听起来更像是英语，更容易弄清楚函数的作用。此外，注意如果你的函数体只包含一个语句，则`return`关键字是可选的。

在下一节中，你将学习如何在其他函数体内使用几个较小的函数，这些被称为**嵌套函数**。

## 使用嵌套函数

在另一个函数体内有一个函数，这些被称为嵌套函数。这允许你将多个相关函数放在一起，使封装函数更容易理解。

嵌套函数可以使用封装函数的变量。让我们通过编写一个计算贷款月供的函数来了解嵌套函数是如何工作的。

输入并运行以下代码：

```swift
func calculateMonthlyPayments(carPrice: Double, downPayment: Double, interestRate: Double, paymentTerm: Double) -> Double {
  func loanAmount() -> Double {
    carPrice - downPayment
  }
  func totalInterest() -> Double {
    interestRate * paymentTerm
  }
  func numberOfMonths() -> Double {
    paymentTerm * 12
  }
  return ((loanAmount() + (loanAmount() *
  totalInterest() / 100 )) / numberOfMonths())
}
calculateMonthlyPayments(carPrice: 50000, downPayment: 5000, interestRate: 3.5, paymentTerm: 7.0) 
```

这里，在`calculateMonthlyPayments(carPrice:downPayment:interestRate:paymentTerm:)`中有三个函数。让我们来看看它们：

+   第一个嵌套函数`loanAmount()`通过从`carPrice`中减去`downPayment`来计算总贷款金额。它返回`50000 - 5000` = `45000`。

+   第二个嵌套函数`totalInterest()`通过将`interestRate`乘以`paymentTerm`来计算支付期限产生的总利息金额。它返回`3.5 * 7` = `24.5`。

+   第三个嵌套函数`numberOfMonths()`通过将`paymentTerm`乘以`12`来计算支付期限的总月份数。它返回`7 * 12` = `84`。

注意，三个嵌套函数都使用了封装函数的变量。返回的值是 `( 45000 + ( 45000 * 24.5 / 100 ) ) / 84` = `666.96`，这是您为了购买这辆车每月必须支付的金额。

如您所见，Swift 中的函数与其他语言中的函数类似，但有一个酷炫的特性。在 Swift 中，函数是**一等类型**，因此它们可以用作参数和返回类型。让我们在下一节中看看这是如何实现的。

## 使用函数作为返回类型

一个函数可以作为其返回类型返回另一个函数。输入并运行以下代码以创建一个函数，该函数使用两种可能的方法之一生成π的值：

```swift
func approximateValueOfPi1() -> Double {
  3.14159
}
func approximateValueOfPi2() -> Double {
  22.0 / 7.0
}
func pi() -> (() -> Double) {
  approximateValueOfPi1
  // approximateValueOfPi2
}
pi()() 
```

`approximateValueOfPi1()` 和 `approximateValueOfPi2()` 都是没有任何参数并返回π近似值的函数。`pi()` 函数的返回类型是一个没有参数并返回 `Double` 类型的函数。这意味着它可以返回 `approximateValueOfPi1`（如所示）或 `approximateValueOfPi2`，因为这两个函数都符合预期的返回类型。

`pi()()` 调用了返回 `3.14159` 的函数 `approximateValueOfPi1`。**3.14159** 在结果区域中显示。

让我们看看在下一节中如何将一个函数用作另一个函数的参数。

## 使用函数作为参数

一个函数可以将另一个函数作为参数。输入并运行以下代码以创建一个函数，该函数用于判断一个满足特定条件的数字是否存在于数字列表中：

```swift
func isThereAMatch(listOfNumbers: [Int], condition: (Int) -> Bool) -> Bool {
  for number in listOfNumbers {
    if condition(number) {
      return true
    }
  }
  return false
}
func numberIsOdd(number: Int) -> Bool {
  (number % 2) > 0
}
func numberIsEven(number: Int) -> Bool {
  (number % 2) == 0
}
let numbersList = [1, 3, 5, 7]
isThereAMatch(listOfNumbers: numbersList, condition: numberIsOdd) 
```

`isThereAMatch(listOfNumbers:condition:)` 有两个参数：一个整数数组和函数。提供的作为参数的函数必须接受一个整数值并返回一个布尔值。

`numberIsOdd(number:)` 和 `numberIsEven(_:)` 都接受一个整数并返回一个布尔值，这意味着任一函数都可以作为第二个参数的参数。包含奇数的数组 `numbersList` 用作第一个参数的参数。当 `numberIsOdd` 作为第二个参数的参数时，`isThereAMatch(listOfNumbers:condition:)` 被调用时会返回 `true`。尝试使用 `numberisEven` 作为第二个参数的参数。

函数作为参数和返回类型可能难以理解，但在您学习旅程的这个阶段相对较少，所以如果您一开始不理解，请不要担心。随着您经验的积累，这会变得更容易理解。

在下一节中，您将了解如果使用的参数不合适，如何从函数中提前退出。

## 使用守卫语句提前退出函数

假设您需要一个函数用于在线购物终端。这个函数将在您购买东西时计算借记卡或信用卡的剩余余额。您想要购买的物品价格输入在一个文本字段中。

文本框中的值被转换为整数，这样你就可以计算剩余的卡余额。如果输入数据有问题，能够提前退出函数是非常有用的。

输入并运行以下代码：

```swift
func buySomething(itemValueEntered itemValueField: String, cardBalance: Int) -> Int {
  guard let itemValue = Int(itemValueField) else {
    print("Error in item value")
    return cardBalance
  }
  let remainingBalance = cardBalance - itemValue
  return remainingBalance
}
print(buySomething(itemValueEntered: "10", cardBalance: 50))
print(buySomething(itemValueEntered: "blue", cardBalance: 50)) 
```

你应该在调试区域看到这个结果：

```swift
40
Error in item value
50 
```

让我们看看这个函数是如何工作的。函数体内的第一行是一个`guard`语句。这个语句检查一个条件是否为`true`；如果不是，它将退出函数。在这里，它被用来检查用户是否在在线购买终端输入了有效的价格。

如果是这样，该值可以成功转换为整数，你可以计算剩余的卡余额。否则，`guard`语句中的`else`子句将被执行。错误信息将被打印到调试区域，并且返回未更改的卡余额。

对于`print(buySomething(itemValueEntered: "10", cardBalance: 50))`，项目价格成功从卡余额中扣除，并返回`40`。

对于`print(buySomething(itemValueEntered: "blue", cardBalance: 50))`，`guard`语句的条件失败，其`else`子句被执行，导致错误信息被打印到调试区域，并返回`50`。

现在，你已经知道了如何创建和使用函数。你还看到了如何使用自定义参数标签、嵌套函数、函数作为参数或返回类型，以及`guard`语句。

现在，让我们看看闭包。与函数一样，闭包允许你组合多个指令，但闭包没有名字，并且可以被分配给一个常量或变量。你将在下一节中看到它们是如何工作的。

# 探索闭包

闭包，就像函数一样，包含一系列指令，可以接受参数并返回值。然而，闭包没有名字。闭包中的指令序列被大括号(`{ }`)包围，`in`关键字将参数和返回类型与闭包体分开。

闭包可以被分配给一个常量或变量，所以当你需要在程序内部传递它们时，它们非常方便。例如，假设你有一个从互联网下载文件的应用程序，并且你需要在文件下载完成后对文件进行一些操作。你可以在闭包内部放置一个处理文件的指令列表，并在文件下载完成后让程序执行它。

要了解更多关于闭包的信息，请访问[`docs.swift.org/swift-book/documentation/the-swift-programming-language/closures/`](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/closures/)。

你现在将编写一个闭包，该闭包将对数字数组中的每个元素应用计算。将以下代码添加到你的游乐场中，然后点击**运行**按钮来运行它：

```swift
var numbersArray = [2, 4, 6, 7]
let myClosure = { (number: Int) -> Int in
   let result = number * number
   return result
}
let mappedNumbers = numbersArray.map(myClosure) 
```

这将一个计算数字的平方的闭包赋值给 `myClosure`。然后 `map()` 函数将此闭包应用于 `numbersArray` 中的每个元素。每个元素都乘以自身，并在结果区域出现 `[4, 16, 36, 49]`。

您可以以更简洁的方式编写闭包，您将在下一节中看到如何做到这一点。

## 简化闭包

新开发者遇到困难的一件事是经验丰富的 Swift 程序员编写闭包时使用的非常简洁的方法。考虑以下示例中的代码：

```swift
var testNumbers = [2, 4, 6, 7]
let mappedTestNumbers = testNumbers.map({ 
  (number: Int) -> Int in
  let result = number * number
  return result
})
print(mappedTestNumbers) 
```

在这里，您有 `testNumbers`，一个数字数组，您使用 `map(_:)` 函数依次将闭包映射到数组的每个元素。闭包中的代码将数字乘以自身，生成该数字的平方。结果 **[4, 16, 36, 49]** 然后打印到调试区域。您将看到，闭包代码可以写得更加简洁。

当闭包的类型已知时，您可以删除参数类型、返回类型或两者。单行闭包隐式返回其唯一语句的值，这意味着您可以删除 `return` 语句。因此，您可以按如下方式编写闭包：

```swift
let mappedTestNumbers = testNumbers.map({ number in
  number * number
}) 
```

当闭包是函数的唯一参数时，您可以省略包围闭包的括号，如下所示：

```swift
let mappedTestNumbers = testNumbers.map { number in
  number * number
} 
```

您可以使用表示它们在参数列表中相对位置的数字来引用参数，而不是使用名称，如下所示：

```swift
let mappedTestNumbers = testNumbers.map { $0 * $0 } 
```

因此，闭包现在确实非常简洁，但对于新开发者来说可能难以理解。请随意以您舒适的方式编写闭包。

您现在已经知道如何创建和使用闭包，以及如何更简洁地编写它们。太棒了！

# 概述

在本章中，您学习了如何将语句分组到函数中。您学习了如何使用自定义参数标签、函数内的函数、函数作为返回类型、函数作为参数以及 `guard` 语句。当您需要在程序的不同点完成相同任务时，这将很有用。

您还学习了如何创建闭包。当您需要在程序中传递代码块时，这将很有用。

在下一章中，我们将研究类、结构和枚举。类和结构允许创建可以存储状态和行为复杂对象，枚举可以用来限制可以分配给变量或常量的值，从而减少出错的可能性。

# 加入我们的 Discord！

与其他用户、专家和作者本人一起阅读这本书。提出问题，为其他读者提供解决方案，通过 Ask Me Anything 会话与作者聊天，等等。扫描二维码或访问链接加入社区。

[`packt.link/ios-Swift`](https://packt.link/ios-Swift%0D)

![](https://packt.link/ios-Swift%0D)
