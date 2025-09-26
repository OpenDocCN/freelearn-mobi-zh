# 第六章：*第六章*：函数和闭包

到目前为止，你可以编写合理复杂的程序，这些程序可以做出决策并重复指令序列。你还可以使用集合类型存储程序数据。随着你编写的程序在大小和复杂性上增长，理解它们所做的工作将变得更加困难。

为了使大型程序更容易理解，Swift 允许你创建**函数**，这让你可以将多个指令组合在一起，并通过调用单个名称来执行它们。你还可以创建**闭包**，这让你可以将多个指令组合在一起，而不需要名称，并将其分配给常量或变量。

到本章结束时，你将了解函数、嵌套函数、作为返回类型的函数、作为参数的函数以及`guard`语句。你还将了解如何创建和使用闭包。

以下将涵盖以下主题：

+   理解函数

+   理解闭包

# 技术要求

本章的 Xcode 游乐场位于本书代码包的`Chapter06`文件夹中，可以在此处下载：

[`github.com/PacktPublishing/iOS-15-Programming-for-Beginners-Sixth-Edition`](https://github.com/PacktPublishing/iOS-15-Programming-for-Beginners-Sixth-Edition)

观看以下视频，看看代码是如何运行的：

[`bit.ly/3o2MYTs`](https://bit.ly/3o2MYTs)

如果你希望从头开始，创建一个新的游乐场，并将其命名为`FunctionsAndClosures`。

你可以在进行过程中输入并运行本章中的所有代码。让我们先学习函数。

# 理解函数

函数对于封装执行特定任务的一组指令非常有用，例如：

+   计算餐厅餐费的 10%服务费。

+   计算你希望购买的汽车的月供。

下面是一个函数的样子：

```swift
func functionName(parameter1: ParameterType, ...) -> ReturnType {
   code
}
```

每个函数都有一个描述性的名称。你可以定义一个或多个作为输入的函数值，这些值被称为**参数**。你还可以定义函数完成后的输出类型，这被称为其**返回类型**。参数和返回类型都是可选的。

你“调用”函数的名称来执行它。这就是函数调用的样子：

```swift
functionName(parameter1: argument1, …)
```

你提供与函数参数类型匹配的输入值（称为**参数**）。

重要信息

要了解更多关于函数的信息，请访问[`docs.swift.org/swift-book/LanguageGuide/Functions.html`](https://docs.swift.org/swift-book/LanguageGuide/Functions.html)。

在下一节中，我们将看看如何创建一个计算服务费的函数。

## 创建一个函数

在其最简单的形式中，函数只是执行一些指令，没有参数或返回类型。你将通过编写一个计算餐费服务费的函数来了解这是如何工作的。服务费应该是餐费的 10%。

将以下代码添加到您的游乐场中，以创建并调用此函数，然后点击 **播放/停止** 按钮运行它：

```swift
func serviceCharge() {
   let mealCost = 50
   let serviceCharge = mealCost / 10
   print("Service charge is \(serviceCharge)")
}
serviceCharge()
```

您刚刚创建了一个名为 `serviceCharge()` 的非常简单的函数。它所做的只是计算价值为 $50 的餐费的 10% 服务费，即 `50 / 10`，返回 `5`。然后您使用其名称调用此函数。您将在调试区域看到 `Service charge is 5`。

这个函数不太有用，因为每次调用此函数时 `mealCost` 总是 `50`，结果 `5` 只在调试区域打印，并且不能在您的程序的其他地方使用。让我们添加一些参数和一个返回类型来使这个函数更有用。

按照以下所示修改您的代码：

```swift
func serviceCharge(mealCost: Int) -> Int {
   return mealCost / 10
}
let serviceChargeAmount = serviceCharge(mealCost: 50)
print(serviceChargeAmount)
```

这要好得多。现在，您可以在调用 `serviceCharge(mealCost:)` 函数时设置餐费，并将结果分配给变量或常量。尽管如此，它看起来有点别扭。您应该尝试使 Swift 中的函数签名读起来像英语句子，因为这被认为是最佳实践。让我们在下一节中看看如何做到这一点，您将使用**自定义标签**使您的函数更具英语风格且更容易理解。

## 使用自定义参数标签

注意，`serviceCharge(mealCost:)` 函数不太像英语。您可以为参数添加自定义标签，使函数更容易理解。

按照以下所示修改您的代码：

```swift
func serviceCharge(forMealPrice mealCost: Int) -> Int {
   return mealCost / 10
}
let serviceChargeAmount = serviceCharge(forMealPrice: 50)
print(serviceChargeAmount)
```

函数的工作方式与之前完全相同，但为了调用它，您使用 `serviceCharge(forMealPrice:)`。这听起来更像英语，并使您更容易了解函数的功能。

在下一节中，您将学习如何在其他函数体内使用几个较小的函数，这些函数被称为**嵌套函数**。

## 使用嵌套函数

在另一个函数体内有一个函数是可能的，这些函数被称为嵌套函数。嵌套函数可以使用封装函数的变量。让我们通过编写一个计算贷款月供的函数来了解嵌套函数是如何工作的。

输入并运行以下代码：

```swift
func calculateMonthlyPayments(carPrice: Double, downPayment: Double, interestRate: Double, paymentTerm: Double) -> Double {
   func loanAmount() -> Double {
      return carPrice - downPayment
   }
   func totalInterest() -> Double {
      return interestRate * paymentTerm
   }
   func numberOfMonths() -> Double {
      return paymentTerm * 12
   }
   return ((loanAmount() + ( loanAmount() * 
   totalInterest() / 100 )) / numberOfMonths())
}
calculateMonthlyPayments(carPrice: 50000, downPayment: 5000, interestRate: 3.5, paymentTerm: 7.0)
```

在这里，`calculateMonthlyPayments (carPrice:downPayment:interestRate:paymentTerm:)` 中有三个函数。让我们来看看它们：

+   第一个嵌套函数 `loanAmount()` 通过从 `carPrice` 中减去 `downPayment` 来计算总贷款金额。它返回 `50000 - 5000` = `45000`。

+   第二个嵌套函数 `totalInterest()` 通过将 `interestRate` 乘以 `paymentTerm` 来计算支付期限中产生的总利息金额。它返回 `3.5 * 7` = `24.5`。

+   第三个嵌套函数 `numberOfMonths()` 通过将 `paymentTerm` 乘以 `12` 来计算支付期限中的总月份数。它返回 `7 * 12` = `84`。

注意，这三个嵌套函数都使用了封装函数的变量。返回的值是 `( 45000 + ( 45000 * 24.5 / 100 ) ) / 84` = `666.96`，这是您为了购买这辆车在 7 年内每月需要支付的金额。

如你所见，Swift 中的函数与其他语言中的函数类似，但有一个酷炫的功能。函数在 Swift 中是 **一等类型**，因此它们可以用作参数和返回类型。让我们在下一节中看看这是如何实现的。

## 使用函数作为返回类型

一个函数可以作为其返回类型返回另一个函数。输入并运行以下代码以创建一个生成 Pi 值的函数：

```swift
func makePi() -> (() -> Double) {
   func generatePi() -> Double {
      return 22.0 / 7.0
   }
   return generatePi
}
let pi = makePi()
print(pi())
```

`makePi()` 函数的返回类型是一个没有参数且返回类型为 `Double` 的函数。`generatePi()` 是一个没有参数且返回类型为 `Double` 的函数，它将是返回的函数。因此，`pi` 将被分配给 `generatePi()`，并在调用时返回 `22.0/7.0`。`3.142857142857143` 将在调试区域打印。

让我们看看一个函数如何作为另一个函数的参数在下一节中使用。

## 使用函数作为参数

一个函数可以将另一个函数作为参数。输入并运行以下代码以创建一个函数，该函数用于确定一个满足特定条件的数字是否存在于数字列表中：

```swift
func isThereAMatch(listOfNumbers: [Int], condition: (Int) -> Bool) -> Bool {
   for item in listOfNumbers {
      if condition(item) {
         return true
      }
   }
   return false
}
func oddNumber(number: Int) -> Bool {
   return (number % 2) > 0
}
var numbersList = [2, 4, 6, 7]
isThereAMatch(listOfNumbers: numbersList, condition: oddNumber)
```

`isThereAMatch(listOfNumbers:condition:)` 有两个参数；一个整数数组和函数。提供的函数作为参数必须接受一个整数值并返回一个布尔值。`oddNumber(number:)` 接受一个整数并返回 `true` 如果该数字是奇数，这意味着它可以作为第二个参数的参数。包含奇数的 `numbersList` 数组用作第一个参数的参数。由于 `numbersList` 包含奇数，当调用 `isThereAMatch(listOfNumbers:condition:)` 时，它将返回 `true`。

在下一节中，你将看到如果使用的参数不适合，如何在函数上执行早期退出。

## 使用 `guard` 语句提前退出函数

如果输入数据有问题，能够提前退出函数是有用的。比如说，你需要一个函数用于在线购买终端。这个函数将在你购买东西时计算借记卡或信用卡的剩余余额。你想要购买的商品价格输入在一个文本字段中。文本字段中的值被转换为整数，以便你可以计算剩余的卡余额。

输入并运行以下代码：

```swift
func buySomething(itemValueEntered itemValueField: String, cardBalance: Int) -> Int {
   guard let itemValue = Int(itemValueField) else {
      print("error in item value")
      return cardBalance
   }
   let remainingBalance = cardBalance - itemValue 
   return remainingBalance
}
print(buySomething(itemValueEntered: "10", cardBalance: 50))
print(buySomething(itemValueEntered: "blue", cardBalance: 50))
```

你应该在调试区域看到以下结果：

```swift
40
error in item value
50
```

让我们看看这个函数是如何工作的。函数体内的第一行是一个 `guard` 语句。这个语句检查一个条件是否为 `true`；如果不是，它将退出函数。在这里，它被用来检查用户是否在在线购买终端中输入了有效的价格。如果是，该值可以成功转换为整数，并且你可以计算剩余的卡余额。否则，`guard` 语句中的 `else` 子句将被执行。错误信息将在调试区域打印，并且将返回未更改的卡余额。

对于 `print(buySomething(itemValueEntered: "10", cardBalance: 50))`，商品价格成功从卡余额中扣除，并返回 `40`。

对于 `print(buySomething(itemValueEntered: "blue", cardBalance: 50))`，`guard` 语句的条件失败，其 `else` 子句被执行，导致错误信息被打印到调试区域，并返回 `50`。

现在你已经知道了如何创建和使用函数。你还看到了如何使用自定义参数标签、嵌套函数、函数作为参数或返回类型以及 `guard` 语句。

现在，让我们看看闭包。与函数一样，闭包允许你将多个指令组合在一起，但闭包没有名字，并且可以被分配给一个常量或变量。你将在下一节中看到它们是如何工作的。

# 理解闭包

闭包，就像函数一样，包含一系列指令，可以接受参数并返回值。然而，闭包没有名字。闭包中的指令序列被大括号（`{ }`）包围，`in` 关键字将参数和返回类型与闭包体分开。

闭包可以被分配给一个常量或变量，所以当你需要在程序内部传递它们时，它们非常有用。例如，假设你有一个应用程序从互联网上下载文件，并且你需要在文件下载完成后对文件进行一些操作。你可以在闭包中放置一个处理文件的指令列表，并在文件下载完成后让程序执行它。你将在 *第十六章* *使用 MapKit 入门* 中看到闭包是如何使用的。

重要信息

要了解更多关于闭包的信息，请访问 [`docs.swift.org/swift-book/LanguageGuide/Closures.html`](https://docs.swift.org/swift-book/LanguageGuide/Closures.html)。

现在，你将编写一个闭包，它将对数字数组中的每个元素执行计算。将以下代码添加到你的游乐场中，然后点击**播放/停止**按钮来运行它：

```swift
var numbersArray = [2, 4, 6, 7]
let myClosure = { (number: Int) -> Int in 
   let result = number * number
   return result
}
let mappedNumbers = numbersArray.map(myClosure)
```

这将分配一个计算数字的平方的闭包给 `myClosure`。然后 `map()` 函数将这个闭包应用到 `numbersArray` 的每个元素上。每个元素都乘以自身，然后 `[4, 16, 36, 49]` 出现在结果区域。

可以以更简洁的方式编写闭包，你将在下一节中看到如何做到这一点。

## 简化闭包

新的开发者可能会遇到的问题是经验丰富的 Swift 程序员编写闭包的非常简洁的方式。考虑以下示例中的代码：

```swift
var testNumbers = [2, 4, 6, 7]
let mappedTestNumbers = testNumbers.map({ (number: Int) 
   -> Int in 
   let result = number * number
   return result
})
print(mappedTestNumbers)
```

在这里，你有 `testNumbers`，一个数字数组，你使用 `map(_:)` 函数依次将闭包映射到数组的每个元素上。闭包中的代码将数字乘以自身，生成该数字的平方。然后，结果 `[4, 16, 36, 49]` 被打印到调试区域。正如你将看到的，闭包代码可以写得更加简洁。

当闭包的类型已知时，你可以移除参数类型、返回类型或两者。单语句闭包隐式返回其唯一语句的值，这意味着你也可以移除`return`语句。因此，你可以将闭包写成如下形式：

```swift
let mappedTestNumbers = testNumbers.map({ number in 
  number * number 
})
```

当闭包是函数的唯一参数时，你可以省略包围闭包的括号，如下所示：

```swift
let mappedTestNumbers = testNumbers.map { number in 
  number * number 
}
```

你可以通过一个表示它们在参数列表中相对位置的数字来引用参数，而不是通过名称，如下所示：

```swift
let mappedTestNumbers = testNumbers.map { $0 * $0 }
```

因此，闭包现在确实非常简洁，但对于新开发者来说可能有些难以理解。请随意以你舒适的方式编写闭包。

现在，你已经知道了如何创建和使用闭包，以及如何更简洁地编写它们。太棒了！

# 摘要

在本章中，你学习了如何将语句组合成函数。你了解了如何使用自定义参数标签、函数内的函数、函数作为返回类型以及函数作为参数。这在以后你需要在不同程序点完成相同任务时将非常有用。

你还学习了如何创建闭包。当你需要在程序中传递代码块时，这将非常有用。

在下一章中，你将学习类、结构和枚举。类和结构允许创建可以存储状态和行为的复杂对象，而枚举可以用来限制可以分配给变量或常量的值，从而减少出错的可能性。
