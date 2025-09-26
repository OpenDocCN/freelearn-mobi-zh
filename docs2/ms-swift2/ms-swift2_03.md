# 第三章。使用集合和 Cocoa 数据类型

一旦我超越了基本的 Hello World 初学者应用程序，我很快就开始意识到变量的不足，尤其是在我刚开始编写的 Mad Libs 风格的应用程序中。这些应用程序要求用户输入多个字符串，我为每个用户输入的字段创建了一个单独的变量。拥有所有这些单独的变量很快就变得非常繁琐。我记得和一个朋友谈论过这件事，他问我为什么不用数组。当时，我对数组不熟悉，所以我让他给我展示一下它们是什么。尽管他有一个 TI-99/4A，而我有一个 Commodore Vic-20，但数组的概念是相同的。即使今天，现代开发语言中的数组与我在 Commodore Vic-20 上使用的数组具有相同的基本概念。虽然确实可以不使用集合创建一个有用的应用程序，但正确使用集合确实可以使应用程序开发变得更加容易。

本章将涵盖以下主题：

+   Swift 中的数组是什么以及如何使用它

+   Swift 中的字典是什么以及我们如何使用它

+   Swift 中的集合是什么以及我们如何使用它

+   Swift 中的元组是什么以及我们如何使用它

+   如何在 Swift 中使用 Cocoa 数据类型

+   如何在 Swift 中使用 Foundation 数据类型

# Swift 集合类型

集合是一组或存储具有共同意义的数据。Swift 提供了三种原生集合类型来存储数据。这些集合类型是数组、集合和字典。数组按顺序存储数据，集合是无序的唯一数据集合，字典是无序的键/值对集合。在数组中，我们通过数组中的位置（索引）访问数据；在集合中，我们倾向于遍历集合；而字典通常使用唯一键来访问。

存储在 Swift 集合中的数据必须具有相同的类型。这意味着，例如，我们无法存储一个字符串值和一个整数数组。由于 Swift 不允许我们在集合中混合数据类型，当我们从集合中检索数据时，我们可以确定数据类型。这是另一个表面上看可能像是一个缺点，但实际上是一个设计特性，有助于消除常见的编程错误。在本章中，我们将看到如何通过使用 `AnyObject` 和 `Any` 别名来绕过这个特性。

# 可变性

对于熟悉 Objective-C 的人来说，你们会知道存在不同的类来表示可变和不可变集合。例如，要定义一个可变数组，我们使用 `NSMutableArray` 类，而要定义一个不可变数组，我们使用 `NSArray` 类。Swift 与之不同，因为它没有为可变和不可变集合提供单独的类。相反，我们通过使用 `let` 和 `var` 关键字来定义集合是常量（不可变）还是变量（可变）。这在 Swift 中应该看起来很熟悉，因为，在 Swift 中，我们使用 `let` 关键字定义常量，使用 `var` 关键字定义变量。

### 注意

除非有特定的需求需要更改集合中的对象，否则创建不可变集合是一种良好的实践。这允许编译器优化性能。

让我们开始我们的集合之旅，先看看最常见的集合类型——数组类型。

# 数组

数组是现代编程语言中非常常见的组件，几乎可以在所有现代编程语言中找到。在 Swift 中，数组是相同类型对象的有序列表。这与 Objective-C 中的 `NSArray` 类不同，后者可以包含不同类型的对象。

当创建数组时，我们必须通过显式类型声明或通过类型推断来声明要存储在其中的数据类型。通常，我们只在创建空数组时显式声明数组的数据类型。如果我们用数据初始化数组，我们应该让编译器使用类型推断来推断最合适的数组数据类型。

数组中的每个对象称为一个元素。这些元素按照一定的顺序存储，可以通过其在数组中的位置（索引）来访问。

## 创建和初始化数组

我们可以使用数组字面量来初始化一个数组。数组字面量是一组我们预先填充到数组中的值。以下示例展示了如何使用 `let` 关键字定义一个不可变的整数数组：

```swift
let arrayOne = [1,2,3]
```

正如我们提到的，如果我们需要创建一个可变数组，我们将使用 `var` 关键字来定义数组。以下示例展示了如何定义一个可变数组：

```swift
var arrayTwo = [4,5,6]
```

在前面的两个示例中，编译器通过查看数组字面量中存储的值的类型来推断数组中存储的值的类型。如果我们需要创建一个空数组，我们需要显式声明要存储在数组中的值的类型。以下示例展示了如何声明一个可以用来存储整数的空数组：

```swift
var arrayThree = [Int]()
```

在前面的示例中，我们使用整数值创建了数组，本章中的大多数数组示例也将使用整数值；然而，我们可以在 Swift 中使用任何类型创建数组。唯一的规则是，一旦定义了一个数组包含特定类型，数组中的所有元素都必须是那种类型。以下示例展示了如何创建各种数据类型的数组：

```swift
var arrayOne = [String]()
var arrayTwo = [Double]()
var arrayThree = [MyObject]()
```

Swift 确实提供了用于处理非特定类型的特殊类型别名。这些别名是`AnyObject`和`Any`。我们可以使用这些别名来定义元素类型不同的数组，如下所示：

```swift
var myArray: [AnyObject] = [1,"Two"]
```

我们应该只在明确需要这种行为时才使用`Any`和`AnyObject`别名。始终明确我们集合中包含的数据类型是更好的做法。

我们也可以初始化一个数组，使其具有特定的大小，并且将数组中的所有元素都设置为预定义的值。如果我们想创建一个数组并预先填充默认值，这会非常有用。以下示例定义了一个包含七个元素的数组，每个元素都包含数字`3`：

```swift
var arrayFour = Int
```

虽然最常见的数组是一维数组，但我们也可以创建多维数组。多维数组实际上不过是一个数组的数组。例如，二维数组是一个数组的数组，而三维数组是一个数组的数组的数组。以下示例展示了在 Swift 中创建二维数组的两种方式：

```swift
var multiArrayOne = [[1,2],[3,4],[5,6]]
var multiArrayTwo = [[Int]]()
```

## 访问数组元素

我们使用下标语法来从数组中检索值。数组的下标语法是，一个数字出现在两个方括号之间，这个数字指定了我们想要检索的元素在数组中的位置（索引）。以下示例展示了如何使用下标语法从数组中检索元素：

```swift
let arrayOne = [1,2,3,4,5,6]
print(arrayOne[0])  //Displays '1'
print(arrayOne[3])  //Displays '4'
```

在前面的代码中，我们首先创建了一个包含六个整数的数组。然后我们打印出索引`0`和`3`的值。

如果我们想要从多维数组中检索单个值，我们需要为每个维度提供一个下标。如果我们没有为每个维度提供下标，我们将返回一个数组，而不是数组中的单个值。以下示例展示了我们如何定义一个二维数组，并在两个维度中检索单个值：

```swift
var multiArray = [[1,2],[3,4],[5,6]]
var arr = multiArray[0] //arr contains the array [1,2]
var value = multiArray[0][1] //value contains 2
```

在前面的代码中，我们首先定义了一个二维数组。当我们检索第一维索引`0`的值（`multiArray[0]`）时，我们检索到的是数组`[1,2]`。当我们检索第一维索引`0`和第二维索引`1`的值（`multiArray[0][1]`）时，我们检索到的是整数`2`。

我们可以使用`first`和`last`属性来检索数组的第一个和最后一个元素。由于数组可能为空，`first`和`last`属性返回的是可选值。以下示例展示了如何使用`first`和`last`属性来检索单维和多维数组的第一和最后一个元素：

```swift
let arrayOne = [1,2,3,4,5,6]
var first = arrayOne.first  //first contains 1
var last = arrayOne.last  //last contains 6

let multiArray = [[1,2],[3,4],[5,6]]
var arrFirst1 = multiArray[0].first //arrFirst1 contains 1
var arrFirst2 = multiArray.first //arrFirst2 contains[1,2]
var arrLast1 = multiArray[0].last //arrLast1 contains 2
var arrLast2 = multiArray.last  //arrLast2 contains [5,6]
```

## 计算数组的元素

有时，知道数组中的元素数量是至关重要的。要获取元素数量，我们会使用只读的`count`属性。以下示例展示了如何使用此属性来检索单维和多维数组中的元素数量：

```swift
let arrayOne = [1,2,3]
let multiArrayOne = [[3,4],[5,6],[7,8]]
print(arrayOne.count)  //Displays 3
print(multiArrayOne.count)  //Displays 3 for the three arrays
print(multiArrayOne[0].count)  //Displays 2 for the two elements
```

`count`属性返回的是数组中的元素数量，而不是数组的最大有效索引。对于非空数组，最大有效索引是数组元素数量减一。这是因为数组的第一个元素索引号为零。例如，如果一个数组有两个元素，有效的索引是`0`和`1`，而`count`属性将返回`2`。以下代码说明了这一点：

```swift
let arrayOne = [0,1]
print(arrayOne[0])  //Displays 0
print(arrayOne[1])  //Displays 1
print(arrayOne.count) //Displays 2
```

如果我们尝试使用下标语法从数组中检索一个元素，而该索引超出了数组的范围，应用程序将抛出`Array index out of range`错误。因此，如果我们不确定数组的大小，验证索引是否不在数组范围之外是一种良好的做法。以下示例说明了这个概念：

```swift
//This example will throw an array index out of range error
var arrayTwo = [1,2,3,4]
print(arrayTwo[6])

//This example will not throw an array index out of range error
var arrayOne = [1,2,3,4]
if (arrayOne.count> 6) {
    print(arrayOne[6])
}
```

在前面的代码中，第一段代码会抛出一个`数组索引超出范围`错误异常，因为我们试图从`arrayTwo`数组中访问索引为`6`的值；然而，该数组中只有四个元素。第二个例子不会抛出`数组索引超出范围`错误异常，因为我们正在检查`arrayOne`数组是否包含超过六个元素，如果没有，我们就不尝试访问索引`6`的值。

## 数组是否为空？

要检查数组是否为空（不包含任何元素），我们使用`isEmpty`属性。如果数组为空，该属性将返回`true`；如果它包含元素，则返回`false`。以下示例展示了如何检查数组是否为空：

```swift
var arrayOne = [1,2]
var arrayTwo = [Int]()
arrayOne.isEmpty  //Returns false because the array is not empty
arrayTwo.isEmpty  //Returns true because the array is empty
```

## 追加到数组

静态数组有些有用，但能够动态地添加元素才是数组真正有用的地方。要向数组的末尾添加一个项目，我们可以使用`append`方法。以下示例展示了如何将项目追加到数组的末尾：

```swift
var arrayOne = [1,2]
arrayOne.append(3)  //arrayOne will now contain 1, 2 and 3
```

Swift 还允许我们使用加法赋值运算符（`+=`）将一个数组追加到另一个数组中。以下示例展示了如何使用加法赋值运算符将数组追加到另一个数组的末尾：

```swift
var arrayOne = [1,2]
arrayOne += [3,4]  //arrayOne will now contain 1, 2, 3 and 4
```

将元素追加到数组末尾的方式完全取决于你。我个人更喜欢赋值运算符，因为它对我来说更容易阅读，但在这本书中我们将使用两种方式。

## 向数组中插入值

我们可以通过使用`insert`方法将值插入到数组中。`insert`方法将移动从指定索引开始的所有项目，向上移动一个位置，为新元素腾出空间，然后将值插入到指定索引。以下示例展示了如何使用`insert`方法将新值插入到数组中：

```swift
var arrayOne = [1,2,3,4,5]
arrayOne.insert(10, atIndex: 3) //arrayOne now contains 1, 2, 3, 10, 4 and 5
```

### 注意

您不能插入超出数组当前范围的值。尝试这样做将抛出`索引超出范围异常`。例如，在先前的代码中，如果我们尝试在索引 10 处插入一个新的整数，我们将收到一个`索引超出范围异常错误`，因为`arrayOne`只包含五个元素。这个例外是我们能够直接在最后一个元素之后插入一个项；因此，我们可以将项插入到索引`6`。然而，建议我们使用`append`函数来追加项以避免错误。

## 替换数组中的元素

我们使用下标语法来替换数组中的元素。使用下标，我们选择要更新的数组元素，然后使用赋值运算符分配新值。以下示例展示了我们如何在数组中替换一个值：

```swift
var arrayOne = [1,2,3]
arrayOne[1] = 10  //arrayOne now contains 1,10,3
```

### 注意

您不能更新超出数组当前范围的值。尝试这样做将抛出我们在尝试在数组范围之外插入值时抛出的相同的`索引超出范围`异常。

## 从数组中删除元素

我们可以使用三种方法来从数组中删除一个或所有元素。这些方法是`removeLast()`、`removeAtIndex()`和`removeAll()`。以下示例展示了如何使用这三种方法从数组中删除元素：

```swift
var arrayOne = [1,2,3,4,5]
arrayOne.removeLast()  //arrayOne now contains 1, 2, 3 and 4
arrayOne.removeAtIndex(2)  //arrayOne now contains 1, 2 and 4
arrayOne.removeAll()  //arrayOne is now empty
```

`removeLast()`和`removeAtIndex()`方法也会返回它正在删除的元素的值。因此，如果我们想了解被删除项的值，我们可以重写`removeAtIndex`和`removeLast`行来捕获该值，如下面的示例所示：

```swift
var arrayOne = [1,2,3,4,5]
var removed1 = arrayOne.removeLast()  //removed1 contains the value 5
var removed = arrayOne.removeAtIndex(2)  //removed contains the value 3
```

## 添加两个数组

要通过将两个数组相加来创建一个新数组，我们使用加法（`+`）运算符。以下示例展示了如何使用加法（`+`）运算符创建一个包含两个其他数组所有元素的新数组：

```swift
let arrayOne = [1,2]
let arrayTwo = [3,4]
var combine = arrayOne + arrayTwo //combine contains 1, 2, 3 and 4
```

在先前的代码中，`arrayOne`和`arrayTwo`保持不变，而组合数组包含来自`arrayOne`的元素，然后是来自`arrayTwo`的元素。

## 反转数组

我们可以使用`reverse()`方法从原始数组中创建一个新数组，其元素顺序相反。`reverse`方法不会改变原始数组。以下示例展示了如何使用`reverse()`方法：

```swift
var arrayOne = [1,2,3]
var reverse = arrayOne.reverse() //reverse contains 3,2 and 1
```

在先前的代码中，`arrayOne`的元素保持不变，而`reverse`数组将包含来自`arrayOne`的所有元素，但顺序相反。

## 从数组中检索子数组

我们可以通过使用带有范围的下标语法从现有数组中检索`子数组`。以下示例展示了如何从现有数组中检索一系列元素：

```swift
let arrayOne = [1,2,3,4,5]
var subArray = arrayOne[2…4] //subArray contains 3, 4 and 5
```

`…` 运算符（三个点）被称为范围运算符。在前面代码中，范围运算符表示我想要所有元素，从 `2` 到 `4`（包括元素 `2` 和 `4` 以及它们之间的所有元素）。还有一个范围运算符，即 `..<`，它与 `…` 范围运算符相同，但它排除了最后一个元素。以下示例展示了如何使用 `.<` 运算符。

```swift
let arrayOne = [1,2,3,4,5]
var subArray = arrayOne[2..<4] //subArray contains 3 and 4
```

在前面的示例中，`subArray` 将包含两个元素 `3` 和 `4`。

## 对数组进行批量更改

我们可以使用下标语法和范围运算符来更改多个元素的值。以下示例展示了如何使用下标语法来更改一系列元素的值。

```swift
var arrayOne = [1,2,3,4,5]
arrayOne[1…2] = [12,13]//arrayOne contains 1,12,13,4 and 5
```

在前面的代码中，索引 `1` 和 `2` 处的元素将被更改为数字 `12` 和 `13`。之后，当代码运行时，`arrayOne` 将包含 `1`、`12`、`13`、`4` 和 `5`。

在范围运算符中更改的元素数量不需要与传递的值的数量相匹配。Swift 会进行批量更改——它首先移除由范围运算符定义的元素，然后插入新的值。以下示例演示了这一概念：

```swift
var arrayOne = [1,2,3,4,5]
arrayOne[1…3] = [12,13]
//arrayOne now contains 1, 12, 13 and 5 (four elements)
```

在前面的代码中，`arrayOne` 有五个元素。然后我们说我们想要替换从元素 `1` 到 `3`（包括 `1` 和 `3`）的范围。这导致数组中从元素 `1` 到 `3`（共三个元素）被移除。然后我们在索引 `1` 处向数组中添加两个元素（`12` 和 `13`）。完成这些操作后，`arrayOne` 将包含以下四个元素：`1`、`12`、`13` 和 `5`。让我们看看如果我们尝试添加比移除更多的元素会发生什么：

```swift
var arrayOne = [1,2,3,4,5]
arrayOne[1...3] = [12,13,14,15]
//arrayOne now contains 1, 12, 13, 14, 15 and 5 (six elements)
```

在前面的代码中，`arrayOne` 有五个元素。然后我们说我们想要替换从元素 `1` 到 `3`（包括 `1` 和 `3`）的范围。这导致数组中从元素 `1` 到 `3`（共三个元素）被移除。然后我们在索引 `1` 处向数组中添加四个元素（`12`、`13`、`14` 和 `15`）。完成这些操作后，`arrayOne` 将包含以下六个元素：`1`、`12`、`13`、`14`、`15` 和 `5`。

## 数组算法

Swift 数组有几种方法，这些方法接受一个闭包作为参数。这些方法会转换数组，而闭包会影响数组如何被转换。闭包是包含代码的独立块，可以传递，类似于 Objective-C 中的 blocks 和其他语言中的 lambdas。我们将在第十二章使用闭包中深入讨论闭包。现在，我们只想熟悉 Swift 中算法的工作方式。

### sortInPlace

`sortInPlace`算法在原地排序数组。这意味着当使用`sortInPlace()`方法时，原始数组将被排序后的数组替换。闭包接受两个参数（分别由`$0`和`$1`表示），并且它应该返回一个布尔值，表示第一个元素是否应该放在第二个元素之前。以下代码展示了如何使用排序算法：

```swift
var arrayOne = [9,3,6,2,8,5]
arrayOne.sortInPlace(){ $0 < $1 }
//arrayOne contains 2,3,5,6,8 and 9
```

上述代码将数组按升序排序。我们可以通过我们的规则来判断，如果第一个数字（`$0`）小于第二个数字（`$1`），则返回`true`。因此，当排序算法开始时，它比较前两个数字（`9`和`3`），如果第一个数字（`9`）小于第二个数字（`3`），则返回`true`。在我们的例子中，规则返回`false`，所以数字被反转。算法以这种方式继续排序，直到所有数字都排序完成。

上述示例按数值升序排序了数组；如果我们想反转顺序，我们会在闭包中反转参数。以下代码展示了如何反转排序顺序：

```swift
var arrayOne = [9,3,6,2,8,5]
arrayOne.sortInPlace(){ $1 < $0 }
//arrayOne contains 9,8,6,5,3 and 2
```

当我们运行此代码时，`arrayOne`将包含元素`9`、`8`、`6`、`5`、`3`和`2`。

### `sort`

虽然`sortInPlace`算法在原地排序数组（替换原始数组），但`sort`算法不会改变原始数组，它而是创建一个新的数组，包含原始数组中的排序元素。以下示例展示了如何使用排序算法：

```swift
var arrayOne = [9,3,6,2,8,5]
let sorted = arrayOne.sort(){ $0 < $1 }
//sorted contains 2,3,5,6,8 and 9
//arrayOne contains 9,3,6,2,8 and 5
```

当我们运行此代码时，`arrayOne`将包含原始未排序数组（`9`、`3`、`6`、`2`、`8`和`5`），而排序后的数组将包含新的排序数组（`2`、`3`、`5`、`6`、`8`和`9`）。

### `filter`

`filter`算法将通过过滤原始数组来返回一个新的数组。这是最强大的数组算法之一，最终可能会成为我们使用最多的算法之一。如果我们需要根据一组规则从数组中检索子集，我建议使用此算法，而不是尝试编写自己的方法来过滤数组。闭包接受一个参数，并且如果元素应该包含在新数组中，它应该返回布尔值`true`，如下所示：

```swift
var arrayOne = [1,2,3,4,5,6,7,8,9]
let filtered = arrayFiltered.filter{$0 > 3 && $0 < 7}
//filtered contains 4,5 and 6
```

在前面的代码中，我们传递给算法的规则返回`true`，如果数字大于`3`或小于`7`；因此，任何大于`3`或小于`7`的数字都包含在新的过滤数组中。

让我们来看另一个例子；这个例子展示了如何从一个城市数组中检索出名字中包含字母 o 的子集：

```swift
var city = ["Boston", "London", "Chicago", "Atlanta"]
let filtered = city.filter{$0.rangeOfString("o") != nil}
//filtered contains "Boston", "London" and "Chicago"
```

在前面的代码中，我们使用`rangeOfString()`方法来返回`true`，如果字符串包含字母 o。如果方法返回`true`，则字符串包含在过滤数组中。

### `map`

map 算法返回一个新数组，该数组包含将闭包中的规则应用于数组中每个元素的结果。以下示例展示了如何使用 map 算法将每个数字除以`10`：

```swift
var arrayOne = [10, 20, 30, 40]
let applied = arrayOne.map{ $0 / 10}
//applied contains 1,2,3 and 4
```

在前面的代码中，新数组包含数字`1`、`2`、`3`和`4`，这是通过将原始数组中的每个元素除以`10`得到的结果。

通过 map 算法创建的新数组不需要包含与原始数组相同的元素类型；然而，新数组中的所有元素必须属于同一类型。在以下示例中，原始数组包含整数值，但通过 map 算法创建的新数组包含字符串元素：

```swift
var arrayOne = [1, 2, 3, 4]
let applied = arrayOne.map{ "num:\($0)"}
//applied contains "num:1", "num:2", "num:3" and "num:4"
```

在前面的代码中，我们创建了一个字符串数组，它将原始数组中的数字追加到`num:`字符串中。

### forEach

我们可以使用`forEach`来遍历一个序列。以下示例展示了我们如何这样做：

```swift
var arrayOne = [10, 20, 30, 40]
arrayOne.forEach{ print($0) }
```

此示例将在控制台打印以下结果：

```swift
10
20
30
40
```

虽然使用`forEach`方法非常简单，但它确实有一些限制。推荐遍历数组的方式是使用`for-in`循环，我们将在下一节中看到。

## 遍历数组

我们可以使用`for-in`循环按顺序遍历数组的所有元素。我们将在第四章中更详细地讨论`for-in`循环，*控制流和函数*。`for-in`循环将为数组中的每个元素执行一个或多个语句。以下示例展示了我们如何遍历数组的元素：

```swift
var arr = ["one", "two", "three"]
for item in arr {
    print(item)
}
```

在前面的示例中，`for-in`循环遍历`arr`数组，并为数组中的每个元素执行`print(item)`行。如果我们运行此代码，它将在控制台显示以下结果：

```swift
one
two
three
```

有时候，我们希望像在前面示例中那样遍历数组，但同时也想了解元素的索引和值。为此，我们可以使用`enumerate`方法，该方法为数组中的每个元素返回一个元组（稍后在本章的*元组*部分将详细介绍），其中包含元素的`index`和`value`。以下示例展示了如何使用`enumerate`函数：

```swift
var arr = ["one", "two", "three"]
for (index,value) in arr.arr.enumerate() {
    print"\(index) \(value)")
}
```

上述代码将在控制台显示以下结果：

```swift
0 one
1 two
2 three
```

现在我们已经介绍了 Swift 中的数组，让我们来看看什么是字典。

# 字典

虽然字典不如数组常用，但它们具有额外的功能，这使得它们非常强大。字典是一个容器，存储多个键值对，其中所有键都是同一类型，所有值也都是同一类型。键用作值的唯一标识符。由于我们是通过键来查找值，而不是通过值的索引，因此字典不能保证键值对存储的顺序。

字典适用于存储映射到唯一标识符的项目，其中唯一标识符应用于检索项目。例如，国家和它们的缩写是字典中可以存储的项目的好例子。在以下图表中，我们展示了国家和它们的缩写作为键值对：

| 键 | 值 |
| --- | --- |
| US | 美国 |
| IN | 印度 |
| UK | 英国 |

## 创建和初始化字典

我们可以使用字典字面量来初始化字典，类似于我们使用数组字面量初始化数组。以下示例展示了如何使用前面图表中的键值对创建字典：

```swift
let countries = ["US":"UnitedStates","IN":"India","UK":"UnitedKingdom"]
```

前面的代码创建了一个包含前面图表中每个键值对的不可变字典。就像数组一样，要创建可变字典，我们将使用 `var` 关键字而不是 `let`。以下示例展示了如何创建包含国家的可变字典：

```swift
var countries = ["US":"UnitedStates","IN":"India","UK":"United Kingdom"]
```

在前面的两个例子中，我们创建了一个键和值都是字符串的字典。编译器推断键和值是字符串，因为这是我们放入的类型。如果我们想创建一个空字典，我们需要告诉编译器键和值的类型。以下示例创建具有不同键值类型的各种字典：

```swift
var dic1 = [String:String]()
var dic2 = [Int:String]()
var dic3 = [String:MyObject]()
```

### 备注

如果我们想在字典中使用自定义对象作为键，我们需要让我们的自定义对象符合 Swift 标准库中的 Hashable 协议。我们将在第五章*类和结构*中讨论协议和类，但就目前而言，只需了解可以使用自定义对象作为字典的键。

## 访问字典值

我们使用下标语法来检索特定键的值。如果字典不包含我们正在寻找的键，字典将返回 `nil`；因此，从这个查找返回的变量是一个可选变量。以下示例展示了如何使用下标语法从字典中检索值：

```swift
let countries = ["US":"United States", "IN":"India","UK":"United Kingdom"]
var name = countries["US"]
```

在前面的代码中，变量名将包含字符串，`United States`。

## 在字典中计数键或值

我们使用字典的 `count` 属性来获取字典中键值对的数量。以下示例展示了如何使用 `count` 属性来检索字典中键值对的数量：

```swift
let countries = ["US":"United States", "IN":"India","UK":"United Kingdom"];
var cnt = countries.count  //cnt contains 3
```

在前面的代码中，`cnt` 变量将包含数字 `3`，因为 `countries` 字典中有三个键值对。

## 字典是否为空？

要测试字典是否包含任何键值对，我们可以使用 `isEmpty` 属性。如果字典包含一个或多个键值对，则 `isEmpty` 属性将返回 `false`；如果它是空的，则返回 `true`。以下示例显示了如何使用 `isEmpty` 属性来确定我们的字典是否包含任何键值对：

```swift
let countries = ["US":"United States", "IN":"India","UK":"United Kingdom"]
var empty = countries.isEmpty
```

在前面的代码中，`isEmpty` 属性为 `false`，因为国家字典中有三个键值对。

## 更新键的值

要更新字典中键的值，我们可以使用下标语法或 `updateValue(value:, forKey:)` 方法。`updateValue(value:, forKey:)` 方法有一个下标语法没有的附加功能——它在更改值之前返回与键关联的原始值。以下示例显示了如何使用下标语法和 `updateValue(value:, forKey:)` 方法来更新键的值：

```swift
var countries = ["US":"United States", "IN":"India","UK":"United Kingdom"]

countries["UK"] = "Great Britain"
//The value of UK is now set to "Great Britain"

var orig = countries.updateValue("Britain", forKey: "UK")
//The value of UK is now set to "Britain" and orig now contains "Great Britain"
```

在前面的代码中，我们使用下标语法将与键 `UK` 关联的值从 `United Kingdom` 更改为 `Great Britain`。在替换之前，我们没有保存 `United Kingdom` 的原始值，因此我们无法看到原始值是什么。然后我们使用 `updateValue(value:, forKey:)` 方法将与键 `UK` 关联的值从 `Great Britain` 更改为 `Britain`。使用 `updateValue(value:, forKey:)` 方法，在字典中更改值之前，将 `Great Britain` 的原始值分配给 `orig` 变量。

## 添加键值对

要向字典中添加一个新的键值对，我们可以使用下标语法或与更新键值值相同的 `updateValue(value:, forKey:)` 方法。如果我们使用 `updateValue(value:, forKey:)` 方法，并且键当前不在字典中，则 `updateValue(value:, forKey:)` 方法将添加一个新的键值对并返回 nil。以下示例显示了如何使用下标语法以及 `updateValue(value:, forKey:)` 方法向字典中添加一个新的键值对：

```swift
var countries = ["US":"United States", "IN":"India","UK":"United Kingdom"]

countries["FR"] = "France" //The value of "FR" is set to "France"

var orig = countries.updateValue("Germany", forKey: "DE")
//The value of "DE" is set to "Germany" and orig is nil
```

在前面的代码中，国家字典开始时有三个键值对，然后我们使用下标语法向字典中添加第四个键值对 (`FR/France`)。然后我们使用 `updateValue(value:, forKey:)` 方法向字典中添加第五个键值对 (`DE/Germany`)。`orig` 变量被设置为 `nil`，因为国家字典中没有与 `DE` 键关联的值。

## 删除键值对

有时候我们需要从字典中移除值。我们可以使用下标语法，使用 `removeValueForKey()` 方法或 `removeAll()` 方法来完成此操作。`removeValueForKey()` 方法返回在移除之前键的值。`removeAll()` 方法从字典中移除所有元素。以下示例展示了如何使用下标语法、`removeValueForKey()` 方法和 `removeAll()` 方法从字典中移除键值对：

```swift
var countries = ["US":"United States", "IN":"India","UK":"United Kingdom"];

countries["IN"] = nil //The "IN" key/value pair is removed

var orig = countries.removeValueForKey("UK")
//The "UK" key value pair is removed and orig contains "United Kingdom"

countries.removeAll() //Removes all key/value pairs from the countries dictionary
```

在前面的代码中，`countries` 字典最初有三个键值对。然后我们将与键 `IN` 关联的值设置为 `nil`，这将从字典中移除键值对。我们使用 `removeValueForKey()` 方法来移除与 `UK` 键关联的键。在移除与 `UK` 键关联的值之前，`removeValueForKey()` 方法将值保存在 `orig` 变量中。最后，我们使用 `removeAll()` 方法来移除 countries 字典中剩余的所有键值对。

现在让我们看看集合类型。

# 集合

集合类型是一个类似于数组类型的泛型集合。虽然数组类型是一个有序集合，可能包含重复的元素，但集合类型是一个无序集合，其中每个元素必须是唯一的。

与字典中的键类似，存储在数组中的类型必须符合 Hashable 协议。这意味着该类型必须提供一种方法来计算自己的哈希值。Swift 的所有基本类型，如 String、Double、Int 和 Bool，都符合 Hashable 协议，并且默认情况下可以用于集合。

让我们看看如何使用集合类型。

## 初始化集合

我们有几种初始化集合的方法。就像数组和字典类型一样，Swift 需要知道将要存储的数据类型。这意味着我们必须告诉 Swift 要在集合中存储的数据类型，或者用一些数据初始化它，以便它可以推断数据类型。

就像数组和字典类型一样，我们使用 var 和 let 关键字来声明集合是否可变：

```swift
//Initializes an empty Set of the String type
var mySet = Set<String>() 

//Initializes a mutable set of the String type with initial values
var mySet = Set(["one", "two", "three"])

//Creates aimmutable set of the String type.
let mySet = Set(["one", "two", "three"])
```

## 将元素插入到集合中

我们使用 `insert` 方法将一个元素插入到集合中。如果我们尝试插入一个已经存在于集合中的元素，该元素将被忽略，并且不会抛出错误。以下是一些如何将元素插入到集合中的示例：

```swift
var mySet = Set<String>() 
mySet.insert("One")
mySet.insert("Two")
mySet.insert("Three")
```

## 集合中的元素数量

我们可以使用 `count` 属性来确定 Swift 集合中元素的数量。以下是如何使用 `count` 方法的示例：

```swift
var mySet = Set<String>() 
mySet.insert("One")
mySet.insert("Two")
mySet.insert("Three")
print("\(mySet.count) items")
```

当执行此代码时，它将在控制台打印出消息 `"Three items"`，因为集合包含三个元素。

## 检查集合中是否包含一个元素

我们可以通过使用 `contains()` 方法非常容易地检查一个集合是否包含一个元素，如下所示：

```swift
var mySet = Set<String>() 
mySet.insert("One")
mySet.insert("Two")
mySet.insert("Three")
var contain = mySet.contains("Two")
```

在前面的示例中，`contain` 变量被设置为 True，因为集合确实包含字符串 `"Two"`。

## 遍历集合

我们可以使用 `for` 语句遍历 Set 中的项目。以下示例展示了我们如何遍历集合中的项目：

```swift
for item inmySet {
    print(item)
}
```

上述示例将打印出集合中的每个项目到控制台。

## 从集合中移除项目

我们可以移除单个项目或集合中的所有项目。要移除单个项目，我们会使用 `remove()` 方法，要移除所有项目，则使用 `removeAll()` 方法。以下示例展示了如何从集合中移除项目：

```swift
//The remove method will return and remove an item from a set
var item = mySet.remove("Two")

//The removeAll method will remove all items from a set
mySet.removeAll()
```

## 集合操作

苹果提供了四种方法，我们可以使用这些方法从两个其他集合中构建一个集合。这些操作可以在集合中就地执行，在其中一个集合上执行，或者用于创建一个新集合。这些操作包括：

+   `union` 和 `unionInPlace`：这些创建一个包含两个集合所有唯一值的集合

+   `subtract` 和 `subtractInPlace`：这些创建一个包含第一个集合中不在第二个集合中的值的集合

+   `intersect` 和 `intersectInPlace`：这些创建一个包含两个集合共有值的集合

+   `exclusiveOr` 和 `exclusiveOrInPlace`：这些创建一个新集合，其中包含在任一集合中但不在两个集合中的值

让我们看看一些示例，并查看从每个操作中得到的每个结果。对于所有集合操作示例，我们将使用以下两个集合：

```swift
var mySet1 = Set(["One", "Two", "Three", "abc"])
var mySet2 = Set(["abc","def","ghi", "One"])
```

现在让我们看看我们的示例。我们将首先查看使用 union 方法。这个方法将取两个集合中的唯一值来创建另一个集合：

```swift
var newSetUnion = mySet1.union(mySet2)
```

`newSetUnion` 变量将包含以下值：`"One", "Two", "Three", "abc", "def", "ghi"`。现在让我们看看 subtract 方法。这个方法将创建一个集合，其中包含第一个集合中不在第二个集合中的值：

```swift
var newSetSubtract = mySet1.subtract(mySet2)
```

在这个例子中，`newSetSubtract` 变量将包含 `"Two"` 和 `"Three"` 这两个值，因为它们是唯一不在第二个集合中的两个值。

现在让我们看看 intersect 方法。intersect 方法将创建一个新集合，其中包含两个集合之间的公共值：

```swift
var newSetIntersect = mySet1.intersect(mySet2)
```

在这个例子中，`newSetIntersect` 变量将包含 `"One"` 和 `"abc"` 这两个值，因为它们是两个集合之间的公共值。

现在让我们看看 `exclusiveOr` 方法。这个方法将创建一个新集合，其中包含在任一集合中但不在两个集合中的值：

```swift
//newSetExclusiveOr = {"Two", "Three", "def", "ghi"}
var newSetExclusiveOr = mySet1.exclusiveOr(mySet2)
```

在这个例子中，`newSetExclusiveOr` 变量将包含 `"Two", "Three", "def"` 和 `"ghi"`。

这四种操作（`union`、`subtract`、`intersect` 和 `exclusiveor` 方法）增加了数组中不存在的一些附加功能。与数组相比，查找速度更快，当集合的顺序不重要且集合中的对象必须是唯一的时候，Set 可以是一个非常有用的替代品。

# 元组

元组将多个值组合成一个单一的复合值。与数组和字典不同，元组中的值不必是同一类型。以下示例展示了如何定义一个元组：

```swift
var team = ("Boston", "Red Sox", 97, 65, 59.9)
```

在前面的例子中，我们创建了一个未命名的元组，其中包含两个字符串、两个整数和一个双精度浮点数。我们可以将这个元组中的值分解到一组变量中，如下面的例子所示：

```swift
var team = ("Boston", "Red Sox", 97, 65, 59.9)
var (city, name, wins, loses, percent) = team
```

在前面的代码中，`city` 变量将包含 `Boston`，`name` 变量将包含 `Red Sox`，`wins` 变量将包含 `97`，`loses` 变量将包含 `65`，最后，`percent` 变量将包含 `0.599`。

我们也可以通过指定值的地址来从元组中检索值。以下示例展示了如何通过位置检索值：

```swift
var team = ("Boston", "Red Sox", 97, 65, 59.9)
var city = team.0
var name = team.1
var wins = team.2
var loses = team.3
var percent = team.4
```

为了避免这一分解步骤，我们可以创建一个命名元组。命名元组将一个名称（键）与元组的每个元素关联起来。以下示例展示了如何创建命名元组：

```swift
var team = (city:"Boston", name:"Red Sox", wins:97, loses:65, percent:59.9)
```

要访问命名元组中的值，我们使用点语法。在前面代码中，我们将像这样访问元组的 `city` 元素：`team.city`。在前面代码中，`team.city` 元素将包含 `Boston`，`team.name` 元素将包含 `Red Sox`，`team.wins` 元素将包含 `97`，`team.loses` 元素将包含 `65`，最后，`team.percent` 元素将包含 `59.9`。

元组非常有用，可以用于各种目的。我发现它们非常适合替换那些仅用于存储数据且不包含任何方法的类和结构体。我们将在第五章类和结构体中了解更多关于类的内容。

# 使用 Cocoa 数据类型

到目前为止，在本章中，我们已经探讨了几个原生 Swift 数据类型，如字符串、数组和字典类型。虽然使用这些类型是首选的，但作为 Objective-C 兼容性的一部分，Apple 已经提供了方便且有效的方法，让我们在 Swift 应用程序中处理 Cocoa 数据类型。

一些 Cocoa 和 Swift 数据类型可以互换使用，而另一些则可以在 Cocoa 和 Swift 数据类型之间自动转换。那些可以互换使用或转换的数据类型被称为桥接数据类型。

Swift 还提供了对 Foundation 数据类型的覆盖，这使得我们能够以更接近原生 Swift 类型的方式处理 Foundation 数据类型。如果我们需要使用这些 Foundation 数据类型，我们需要在 Swift 文件的顶部添加以下导入语句：

```swift
import Foundation
```

让我们看看如何处理一些常见的 Cocoa 数据类型。

## NSNumber

Swift 会自动将某些原生数值类型，如 Int、UInt、Float、Bool 和 Double 转换为 `NSNumber` 对象。这允许我们将这些原生数值类型传递给期望 `NSNumber` 对象的参数。这种自动转换是单向的，因为 `NSNumber` 对象可以包含各种数值类型；因此，Swift 编译器将不知道要将 `NSNumber` 转换为哪种数值类型。以下示例展示了如何将原生 Swift Int 和 Double 转换为 `NSNumber`，以及如何将其转换回 Swift Int 和 Double。让我们看一下以下代码：

```swift
var inum = 7    //Creates an Int
var dnum = 10.6 //Creates a Double
var insnum: NSNumber = inum  //Bridges the Int to a NSNumber
var dnsnum: NSNumber = dnum  //Bridges the Double to a NSNumber
var newint = Int(insnum)       //Creates an Int from a NSNumber
var newdouble = Double(dnsnum) //Creates a Double from a NSNumber
```

在前面的代码中，Swift 自动将 `inum` 和 `dnum` 转换为 `NSNumber` 对象，而不需要进行类型转换；然而，当我们尝试将 `NSNumber` 对象转换回 Swift 的 `Int` 或 `Double` 类型时，我们需要将 `NSNumber` 对象类型转换为告诉 Swift 我们正在转换成哪种类型的数字。

## `NSString`

Swift 会自动将原生 `String` 类型桥接到 `NSString` 类型；然而，它不会自动将 `NSString` 对象桥接到原生 String 类型。这允许我们将原生字符串类型传递给期望 `NSString` 对象的参数。因此，当我们混合使用 Objective-C API 与我们的 Swift 项目集成时，它会在需要时自动将 `String` 类型转换为 `NSString` 对象。

这种自动桥接允许我们在 Swift 字符串上调用 `NSString` 方法。Swift 自动将字符串转换为 `NSString` 对象并调用该方法。以下示例展示了如何使用来自 `NSString` 类型的 `cStringUsingEncoding()` 方法将我们的字符串值转换为 C 字符串：

```swift
var str = "Hello World from Swift"
str.cStringUsingEncoding(NSUTF8StringEncoding)
```

要将 `NSString` 对象转换为字符串类型，我们将使用 `as` 关键字。由于 `NSString` 对象始终可以转换为字符串类型，我们不需要使用此类型转换运算符的可选版本 (`as?`)。以下示例展示了如何将 `NSString` 对象转换为字符串类型：

```swift
func testFunc(test: String) {
    print(test)
}
var nsstr: NSString = "abc"
testFunc(nsstr as String)
```

在下一个示例中，我们将 `NSString` 对象转换为原生 Swift 字符串类型，然后调用 Swift 字符串类型的 `toInt()` 方法，将字符串转换为整数，如下所示：

```swift
var nsstr: NSString = "1234"
var num = Int(nsstr as String)//num contains the number 1234
```

在前面的代码中，`num` 变量将包含数字 `1234`，而不是字符串 `1234`。

## `NSArray`

Swift 会自动在 `NSArray` 类和 Swift 原生数组类型之间进行桥接。由于 `NSArray` 对象的元素不需要是同一类型，当我们从 `NSArray` 对象桥接到 Swift 数组时，Swift 数组的元素被设置为 `[AnyObject]` 类型。`[AnyObject]` 类型是一个 Objective-C 或 Swift 类的实例，或者可以桥接到其中一个。

以下示例展示了我们如何在 Swift 中创建一个包含字符串和 Int 类型的 `NSArray` 对象，然后从该 `NSArray` 对象创建一个 Swift 数组：

```swift
var nsarr: NSArray = ["HI","There", 1,2]
var arr = nsarr as? [AnyObject]
```

在前面的代码中，`nsarr: NSArray` 包含四个元素——`HI`、`There`、`1` 和 `2`。在 `nsarr: NSArray` 中，有两个元素是字符串类型，两个是 Int 类型。当我们将 nsarr: `NSArray` 转换为 Swift 数组时，`arr` 数组变成了 `AnyObject` 类型的数组。

如果 `NSArray` 包含特定的对象类型，一旦它被桥接到 Swift 数组，我们就可以将 Swift 数组 `[AnyObject]` 降级为特定对象类型的数组。这个降级的唯一问题是如果数组中的任何元素不是指定的对象类型，降级就会失败，并且新的数组将被设置为 nil。以下示例展示了如何降级一个数组：

```swift
var nsarr: NSArray = ["HI","There"]
var arr = nsarr as [AnyObject]
var newarr = arr as? [String]
```

在前面的例子中，`newarr` 将是一个包含两个元素的字符串数组。现在，让我们将原始的 `NSArray` 改为包含两个整数以及两个字符串。新的代码现在看起来将类似于以下这样：

```swift
var nsarr: NSArray = ["HI","There", 1, 2]
var arr = nsarr as [AnyObject]
var newarr = arr as? [String]
```

由于原始 `NSArray` 定义了一个包含字符串和整数的数组，当我们尝试将 Swift 数组从 `[AnyObject]` 数组降级为字符串数组时，降级失败，并且 `newarr` 变量被设置为 nil。

当我们使用 `as?` 关键字将 `NSArray` 对象转换为数组类型时，建议我们使用可选绑定，因为可能会接收到 nil 值。以下示例说明了如何使用可选绑定进行此转换：

```swift
var nsarr: NSArray = ["HI","There", 1,2]
if let arr = nsarr as? [String] {
    // arr is a native Swift array type.
}
```

现在，让我们看看 `NSDictionary` 对象以及我们如何在 Swift 代码中使用它。

## `NSDictionary`

我们使用 `as` 和 `as?` 关键字在 `NSDictionary` 对象和 Swift 字典类型之间进行转换。以下示例展示了如何进行此转换：

```swift
var nsdic: NSDictionary = ["one":"HI", "two":"There"]
if let dic = nsdic as? [String: String] {
    var newDic = dicasNSDictionary
}
```

在前面的例子中，我们创建了一个包含两个键值对的 `NSDictionary` 对象。这个 `NSDictionary` 对象中的所有键和值都是字符串类型。在第二行，`nsdic2as? [String: String]` 将 `NSDictionary` 对象转换为键和值都是字符串类型的字典类型。然后我们使用 `as` 关键字将字典类型转换回 `NSDictionaryobject`。

在前面的例子中，当我们从 `NSDictionary` 对象转换为字典类型时，我们使用了可选绑定，因为如果 `NSDictionary` 对象中的任何值不是字符串类型，转换就会失败。以下示例说明了这一点：

```swift
var nsdic2: NSDictionary = ["one":"HI", "two":2]
if let dic2 = nsdic2 as? [String:String] {
    // Would not reach this because
    // conversion failed
}
```

在这个例子中，转换失败是因为其中一个值是整型而不是字符串类型。当我们从 `NSDictionary` 对象转换为 Swift 字典时，我们使用 `as?` 关键字，因为转换可能会失败，但当我们从 Swift 字典转换为 `NSDictionary` 对象时，我们使用 `as` 关键字，因为转换总是成功的。

现在让我们看看如何使用 Swift 中的 Foundation 数据类型。

# Foundation 数据类型

当使用 Foundation 数据类型时，Swift 提供了一个覆盖层，使得与之交互的感觉就像它们是原生的 Swift 类型。我们使用这个覆盖层来与 Foundation 类型交互，例如 `CGSize` 和 `CGRect` 用于 iOS 应用程序（`NSSize` 和 `NSRect` 用于 OS X 应用程序）。在开发 iOS 或 OS X 应用程序时，我们将定期与 Foundation 数据类型交互，因此看到这个覆盖层在行动中是很好的。

让我们看看如何初始化一些 Foundation 数据类型。以下示例定义了 `NSRange`、`CGRect` 和 `NSSize`：

```swift
var range = NSRange(location: 3, length: 5)
var rect = CGRect(x: 10, y: 10, width: 20, height: 20)
var size = NSSize(width: 20, height: 40)
```

这个覆盖层还允许我们以类似原生 Swift 类型的感觉访问属性和函数。以下示例展示了如何访问属性和函数：

```swift
var rect = CGRect(x: 10, y: 10, width: 20, height: 20)
rect.origin.x = 20 
//Changes the X value from 10 to 20

var rectMaxY = rect.maxY
//rectMaxY contains 30 (value of y + value of height)

var validRect = rect.isEmpty
//validRect contains false because rect is valid.
```

在前面的代码中，我们初始化了一个 `CGRect` 类型。然后我们将 `x` 属性从 `10` 改为 `20`，检索 `maxY` 属性的值，并检查 `isEmpty` 属性以确定我们是否有一个有效的 `CGRect` 类型。

我们只是刚刚触及了 Swift 和 Objective-C 之间互操作性的表面。我们将在本书后面的 第十三章 中深入讨论这种互操作性。

# 摘要

在本章中，我们介绍了 Swift 集合、元组、Foundation 和 Cocoa 数据类型。对 Swift 的原生集合类型有良好的理解对于用 Swift 架构和开发应用程序至关重要，因为除了非常基础的应用程序之外，所有应用程序都使用集合在内存中存储数据。

在撰写这本书的时候，Swift 仅用于在苹果（iOS 或 OS X）环境中开发应用程序，因此理解 Swift 如何与 Cocoa 和 Foundation 类型交互是至关重要的。虽然我们在本章中简要地介绍了这个主题，但我们将在本书后面的 第十三章，*使用混合匹配* 中更深入地探讨这种交互。
