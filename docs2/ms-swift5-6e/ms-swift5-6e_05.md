# 5

# 使用 Swift 集合

一旦我超越了基本的“Hello, World!”入门级应用程序，我很快就意识到变量的不足，尤其是在我开始编写的 Mad Libs 风格的应用程序中。这些应用程序要求用户输入多个字符串，这导致了为用户输入的每个输入字段创建单独的变量。

拥有所有这些单独的变量很快就变得繁琐。我记得和一个朋友谈论过这件事，他问我为什么不用数组。当时，我对数组不太熟悉，所以我让他给我展示一下它们是什么。尽管他有一个 TI-99/4A，而我有一个 Commodore Vic-20，但数组的概念是相同的。即使今天，现代开发语言中的数组也和我当年在 Commodore Vic-20 上使用的数组有相同的基本概念。当然，不使用集合，如数组，也可以创建有用的应用程序，但正确使用集合可以显著简化应用程序开发。

在本章中，我们将涵盖以下主题：

+   Swift 中数组是什么以及如何使用它

+   Swift 中字典是什么以及如何使用它

+   Swift 中集合是什么以及如何使用它

# Swift 集合类型

集合将多个项目组合成一个单元。Swift 提供了三种原生集合类型。这些集合类型是数组、字典和集合。*数组*按顺序存储数据，*字典*是无序的键值对集合，而*集合*是无序的唯一值集合。在数组中，我们通过数组内的位置或索引来访问数据，而在集合中我们通常遍历集合，字典则是通过唯一键来访问。

存储在 Swift 集合中的数据必须是同一类型。这意味着，例如，我们无法在整数数组中存储字符串值。由于 Swift 不允许我们在集合中混合数据类型，因此当我们从集合中检索元素时，我们可以确定数据类型。这是另一个表面上可能看起来像是一个缺点，但实际上有助于消除常见的编程错误。

让我们从查看集合的可变性开始。

## 可变性

对于熟悉 Objective-C 的人来说，你们会知道存在不同的类来表示可变和不可变集合。例如，要定义一个可变数组，我们使用 `NSMutableArray` 类，而要定义一个不可变数组，我们使用 `NSArray` 类。Swift 稍有不同，因为它不包含用于可变和不可变集合的单独类。相反，我们通过使用 `let` 和 `var` 关键字来定义集合是常量（不可变）还是变量（可变）。这应该看起来很熟悉，因为我们用 `let` 关键字定义常量，用 `var` 关键字定义变量。

除非有特定的需要更改集合中的对象，否则创建不可变集合是一个好的实践。这允许编译器优化性能。

让我们从查看最常见的集合类型：数组类型开始，来开始我们的集合之旅。

# 数组

数组几乎可以在所有现代编程语言中找到。在 Swift 中，数组是相同类型对象的有序列表。

当创建一个数组时，我们必须通过显式类型声明或通过类型推断来声明可以存储在其中的数据类型。通常，我们只有在创建一个空数组时才会显式声明数组的数据类型。如果我们用数据初始化一个数组，编译器会使用类型推断来推断数组的数据类型。

数组中的每个对象称为**元素**。这些元素按顺序存储，可以通过在数组中搜索其位置（索引）来访问。

## 创建和初始化数组

我们可以使用数组字面量来初始化一个数组。数组字面量是一组预填充数组的值。以下示例展示了如何使用`let`关键字定义一个不可变的整数数组：

```swift
let arrayOne = [1,2,3] 
```

如果我们需要创建一个可变数组，我们会使用`var`关键字来定义数组，就像我们定义标准变量一样。以下示例展示了如何定义一个可变数组：

```swift
var arrayTwo = [4,5,6] 
```

在前面的两个示例中，编译器通过查看数组字面量中存储的值的类型来推断数组中存储的值的类型。如果我们想创建一个空数组，我们需要显式声明要存储在数组中的值的类型。Swift 中有两种声明空数组的方法。以下示例展示了如何声明一个可以用来存储整数的空可变数组：

```swift
var arrayThree = [Int]()
var arrayThree: [Int] = [] 
```

在前面的示例中，我们创建了包含整数值的数组，本章中的大多数数组示例也将使用整数值；然而，我们可以在 Swift 中使用任何类型的数组。唯一的规则是，一旦定义了一个数组包含特定类型，数组中的所有元素都必须是那种类型。以下示例展示了我们如何创建各种数据类型的数组：

```swift
var arrayOne = [String]()
var arrayTwo = [Double]()
var arrayThree = [MyObject]() 
```

Swift 为处理非特定类型提供了特殊的类型别名。这些别名是`AnyObject`和`Any`。我们可以使用这些别名来定义元素类型不同的数组，如下所示：

```swift
var myArray: [Any] = [1,"Two"] 
```

`AnyObject`别名可以表示任何类类型的实例，而`Any`别名可以表示任何类型的实例，包括函数类型。我们应该只在有明确需要这种行为时使用`Any`和`AnyObject`别名。始终明确我们集合中包含的数据类型是更好的做法。

如果需要在单个集合中混合类型，我们可以考虑使用元组。

数组也可以初始化为特定大小，并将所有元素设置为预定义的值。如果我们想创建一个数组并预先填充默认值，这会非常有用。以下示例定义了一个包含`7`个元素的数组，每个元素包含数字`3`：

```swift
var arrayFour = Int 
```

从 Swift 5.1 开始，随着 SE-0245 的引入，我们有了创建未初始化数组的能力。使用此功能，我们不需要用默认值填充数组，而是可以根据需要提供一个**闭包来填充数组**。我们将在*第十四章*，*使用闭包*中展示这一功能。

虽然最常见的数组是一维数组，但也可以创建多维数组。实际上，多维数组不过是一个数组的数组。例如，二维数组是一个数组的数组，而三维数组则是一个数组的数组的数组。以下示例展示了在 Swift 中创建二维数组的两种方式：

```swift
var multiArrayOne = [[1,2],[3,4],[5,6]]
var multiArrayTwo = [[Int]]() 
```

现在我们已经看到了如何初始化数组，让我们看看我们如何访问数组的元素。

## 访问数组元素

下标语法用于从数组中检索值。对于数组，下标语法是指一个数字出现在两个方括号之间，该数字指定了我们想要检索的元素在数组中的位置（索引）。以下示例展示了如何使用下标语法从数组中检索元素：

```swift
let arrayOne = [1,2,3,4,5,6]
print(arrayOne[0]) //Displays '1'
print(arrayOne[3]) //Displays '4' 
```

在前面的代码中，我们创建了一个包含六个整数的数组。然后我们打印出索引`0`和`3`处的值。

需要注意的一个重要事实是，Swift 数组中的索引从数字`0`开始。这意味着数组中的第一个元素的索引为`0`。数组中的第二个元素的索引为`1`。

如果我们想要从多维数组中检索单个值，我们需要为数组的每个维度提供一个下标。如果我们不为每个维度提供下标，我们将检索数组而不是数组中的单个值。以下示例展示了我们如何定义一个二维数组并从两个维度中检索单个值：

```swift
let multiArray = [[1,2],[3,4],[5,6]]
let arr = multiArray[0]  //arr contains the array [1,2]
let value = multiArray[0][1]  //value contains 2 
```

在前面的代码中，我们首先定义了一个二维数组。当我们检索第一维索引`0`的值（`multiArray[0]`）时，我们检索到数组`[1,2]`。当我们检索第一维索引`0`和第二维索引`1`的值（`multiArray[0][1]`）时，我们检索到整数`2`。

我们可以使用`first`和`last`属性来检索数组的第一个和最后一个元素。由于数组可能为空，因此`first`和`last`属性返回一个可选值。以下示例展示了如何使用这些属性来检索一维和二维数组的第一个和最后一个元素：

```swift
let arrayOne = [1,2,3,4,5,6]
let first = arrayOne.first    //first contains
let last = arrayOne.last    //last contains 6
let multiArray = [[1,2],[3,4],[5,6]]
let arrFirst1 = multiArray[0].first    //arrFirst1 contains 1
let arrFirst2 = multiArray.first    //arrFirst2 contains[1,2]
let arrLast1 = multiArray[0].last    //arrLast1 contains 2
let arrLast2 = multiArray.last    //arrLast2 contains [5,6] 
```

现在让我们看看我们如何计算数组的元素数量。

## 计算数组元素的数量

有时，知道数组中元素的数量是至关重要的。Swift 中的数组类型包含一个只读的`count`属性。以下示例展示了如何使用此属性来检索单维和多维数组中的元素数量：

```swift
let arrayOne = [1,2,3]
let multiArrayOne = [[3,4],[5,6],[7,8]]
print(arrayOne.count)    //Displays 3
print(multiArrayOne.count)    //Displays 3 for the three array
print(multiArrayOne[0].count)    //Displays 2 for the two elements 
```

`count`属性返回的值是数组中的元素数量，而不是数组的最大有效索引。对于非空数组，最大有效索引是数组元素数量减 1。这是因为数组的第一个元素具有索引号`0`。例如，如果一个数组有两个元素，有效的索引是`0`和`1`，而`count`属性将返回`2`。以下代码说明了这一点：

```swift
let arrayOne = [0,1]
print(arrayOne[0])     //Displays 0
print(arrayOne[1])     //Displays 1
print(arrayOne.count)     //Displays 2 
```

如果我们尝试从数组中检索超出数组范围的元素，应用程序将抛出**数组索引越界**错误。因此，如果我们不确定数组的大小，验证索引是否不在数组范围之外是一个好的做法。以下示例说明了这个概念：

```swift
//This example will throw an array index out of range error 
let arrayOne = [1,2,3,4]
print(arrayOne[6])
//This example will not throw an array index out of range error 
let arrayTwo = [1,2,3,4]
if (arrayTwo.count > 6) { 
    print(arrayTwo[6])
} 
```

在前面的代码中，第一个块将抛出**数组索引越界**错误，因为我们试图从`arrayOne`数组中索引`6`处访问值；然而，数组中只有四个元素。第二个示例不会抛出错误，因为我们试图在尝试访问第六个索引之前检查`arrayTwo`数组是否包含超过六个元素。

## 数组是否为空？

要检查数组是否为空（即不包含任何元素），我们使用`isEmpty`属性。如果数组为空，则此属性将返回`true`；如果不为空，则返回`false`。以下示例展示了如何检查数组是否为空：

```swift
var arrayOne = [1,2]
var arrayTwo = [Int]()
arrayOne.isEmpty  //Returns false because the array is not empty
arrayTwo.isEmpty  //Returns true because the array is empty 
```

现在让我们看看我们如何可以打乱数组。

## 打乱数组

数组可以非常容易地使用`shuffle()`和`shuffled()`方法进行打乱。如果我们正在创建一个游戏，例如一副 52 张牌的牌局游戏，其中数组包含牌组中的所有牌，这将非常有用。要就地打乱数组，可以使用`shuffle()`方法；要将打乱的结果放入一个新数组中，同时保持原始数组不变，则应使用`shuffled()`方法。以下示例展示了这一点：

```swift
var arrayOne = [1,2,3,4,5,6]
arrayOne.shuffle()
let shuffledArray = arrayOne.shuffled() 
```

现在让我们看看我们如何将数据添加到数组中。

## 向数组中添加元素

静态数组有一定的用途，但能够动态地添加元素才是数组真正有用的地方。要将一个项目添加到数组的末尾，我们可以使用`append`方法。以下示例展示了如何将一个项目添加到数组的末尾：

```swift
var arrayOne = [1,2]
arrayOne.append(3)  //arrayOne will now contain 1, 2 and 3 
```

Swift 还允许我们使用加法赋值运算符（`+=`）将一个数组添加到另一个数组中。以下示例展示了如何使用加法赋值运算符将一个数组添加到另一个数组的末尾：

```swift
var arrayOne = [1,2]
arrayOne += [3,4]    //arrayOne will now contain 1, 2, 3 and 4 
```

你将元素追加到数组末尾的方式完全取决于你。我个人更喜欢赋值操作符，因为它对我来说更容易阅读，但在这本书中我们将使用两种方法。

## 将值插入到数组中

我们可以通过使用 `insert` 方法将值插入到数组中。`insert` 方法将所有项目向上移动一个位置，从指定的索引开始，为新元素腾出空间，然后将值插入到指定的索引。以下示例展示了如何使用此方法将新值插入到数组中：

```swift
var arrayOne = [1,2,3,4,5]
arrayOne.insert(10, at: 3)    //arrayOne now contains 1, 2, 3, 10, 4 and 5 
```

现在我们已经看到了如何插入一个值，让我们看看我们如何可以在数组中替换一个元素。

## 在数组中替换元素

我们使用下标语法在数组中替换元素。使用下标，我们选择要更新的数组元素，然后使用赋值操作符分配新值。以下示例展示了我们如何在数组中替换一个值：

```swift
var arrayOne = [1,2,3]
arrayOne[1] = 10    //arrayOne now contains 1,10,3 
```

你不能更新数组当前范围之外的值。尝试这样做将抛出与我们在尝试将值插入到数组范围之外时抛出的相同的**索引越界**异常。

现在让我们看看我们如何可以从数组中移除元素。

## 从数组中移除元素

我们可以使用三种方法来移除数组中的一个或所有元素。这些方法是 `removeLast()`、`remove(at:)` 和 `removeAll()`。以下示例展示了如何使用这三种方法从数组中移除元素：

```swift
var arrayOne = [1,2,3,4,5]
arrayOne.removeLast()    //arrayOne now contains 1, 2, 3 and 4
arrayOne.remove(at:2)    //arrayOne now contains 1, 2 and 4
arrayOne.removeAll()    //arrayOne is now empty 
```

`removeLast()` 和 `remove(at:)` 方法也会返回被移除元素的值。因此，如果我们想知道被移除项的值，我们可以重写 `remove(at:)` 和 `removeLast()` 行来捕获值，如下例所示：

```swift
var arrayOne = [1,2,3,4,5]
var removed1 = arrayOne.removeLast()  //removed1 contains the value 5
var removed = arrayOne.remove(at: 2)  //removed contains the value 3 
```

## 合并两个数组

要通过将两个数组相加来创建一个新数组，我们使用加法（`+`）操作符。以下示例展示了如何使用加法（`+`）操作符创建一个包含两个其他数组所有元素的新数组：

```swift
let arrayOne = [1,2] let arrayTwo = [3,4]
var combined = arrayOne + arrayTwo   //combine contains 1, 2, 3 and 4 
```

在前面的代码中，`arrayOne` 和 `arrayTwo` 保持不变，而 `combined` 数组包含来自 `arrayOne` 的元素，随后是来自 `arrayTwo` 的元素。

## 从数组中检索子数组

我们可以通过使用带范围操作符的下标语法从现有数组中检索子数组。以下示例展示了如何从现有数组中检索一系列元素：

```swift
let arrayOne = [1,2,3,4,5]
var subArray = arrayOne[2...4]    //subArray contains 3, 4 and 5 
```

操作符（三个点）被称为**双边范围操作符**。前面代码中的范围操作符表示我们想要从 `2` 到 `4`（包括元素 `2` 和 `4` 以及它们之间的所有元素）的所有元素。还有一个双边范围操作符，..<，被称为**半开范围操作符**。半开范围操作符的功能与前面的范围操作符相同；然而，它排除了最后一个元素。以下示例展示了如何使用 `..<` 操作符：

```swift
let arrayOne = [1,2,3,4,5]
var subArray = arrayOne[2..<4]     //subArray contains 3 and 4 
```

在前面的例子中，子数组包含两个元素：`3` 和 `4`。双向范围操作符在操作符两侧都有数字。在 Swift 中，我们不仅限于使用双向范围操作符；我们还可以使用单侧范围操作符。以下示例展示了我们如何使用单侧范围操作符：

```swift
let arrayOne = [1,2,3,4,5]
var a = arrayOne[..<3]    //subArray contains 1, 2 and 3 
var b = arrayOne[...3]    //subArray contains 1, 2, 3 and 4
var c = arrayOne[2...]    //subArray contains 3, 4 and 5 
```

单侧范围操作符是在 Swift 语言的第 4 版中添加的。之前的范围操作符使我们能够从数组中访问连续的元素范围。

SE-0270 允许我们获取非连续的元素，这意味着元素可能不是相邻的。这个 Swift 标准库的更新引入了一个新的 `RangeSet` 类型，它是一个非连续索引的子范围。让我们看看以下代码是如何工作的：

```swift
var numbers = [1,2,3,4,5,6,7,8,9,10]
let evenNum = numbers.subranges(where: { $0.isMultiple(of: 2) })
print(numbers[evenNum].count)
//numbers[evenNum] contains 2,4,6,8,10 
```

在此代码中，我们定义了一个包含数字 `1` 到 `10` 的数组。然后我们使用 `subranges(where:)` 方法检索偶数元素。此方法接受一个闭包作为参数，这尚未讨论。现在我们只需要知道我们能够检索非连续的子数组，我们将在第十四章 *使用闭包* 中再次讨论这一点。

## 批量更改数组

我们可以使用范围操作符的索引语法来更改多个元素的值。以下示例展示了如何进行此操作：

```swift
arrayOne[1...2] = [12,13]  //arrayOne contains 1,12,13,4 and 5 
```

在前面的代码中，索引 `1` 和 `2` 处的元素将被更改为数字 `12` 和 `13`；因此，`arrayOne` 将包含 `1`、`12`、`13`、`4` 和 `5`。

在范围操作符中更改的元素数量不需要与传递的值的数量相匹配。Swift 通过首先移除由范围操作符定义的元素，然后插入新值来执行批量更改。以下示例演示了这一概念：

```swift
var arrayOne = [1,2,3,4,5]
arrayOne[1...3] = [12,13]  //arrayOne now contains 1, 12, 13 and 5 
```

在前面的代码中，`arrayOne` 数组开始时有五个元素。然后我们替换从 `1` 到 `3`（包括 `3`）的元素范围。这首先会导致从数组中移除 `1` 到 `3`（即三个元素）。在这三个元素被移除之后，然后向数组中添加两个新元素（`12` 和 `13`），从索引 `1` 开始。完成此操作后，`arrayOne` 将包含四个元素：`1`、`12`、`13` 和 `5`。使用相同的逻辑，我们也可以添加比移除更多的元素。以下示例说明了这一点：

```swift
var arrayOne = [1,2,3,4,5] 
arrayOne[1...3] = [12,13,14,15] 
//arrayOne now contains 1, 12, 13, 14, 15 and 5 (six elements) 
```

在前面的代码中，`arrayOne` 数组开始时有五个元素。然后我们说我们想要替换从 `1` 到 `3`（包括 `3`）的元素范围。与前面的例子一样，这会导致从数组中移除 `1` 到 `3`（即三个元素）。然后我们在索引 `1` 处向数组中添加四个元素（`12`、`13`、`14` 和 `15`）。完成此操作后，`arrayOne` 将包含六个元素：`1`、`12`、`13`、`14`、`15` 和 `5`。

## 数组算法

Swift 数组有几个方法，它们接受一个闭包作为参数。这些方法会根据闭包中的代码以某种方式转换数组。闭包是自包含的代码块，可以被传递，类似于 Objective-C 中的 blocks 和其他语言中的 lambdas。我们将在第十四章 *使用闭包* 中深入讨论闭包。现在，目标是熟悉 Swift 中算法的工作方式。

### 排序

`sort` 算法在原地排序数组。这意味着当使用 `sort()` 方法时，原始数组会被排序后的数组替换。闭包接受两个参数（分别由 `$0` 和 `$1` 表示），并且它应该返回一个布尔值，表示第一个元素是否应该放在第二个元素之前。以下代码展示了如何使用排序算法：

```swift
var arrayOne = [9,3,6,2,8,5]
arrayOne.sort(){ $0 < $1 }
//arrayOne contains 2,3,5,6,8 and 9 
```

之前的代码将按升序排序数组。我们知道这是因为规则会在第一个数字（`$0`）小于第二个数字（`$1`）时返回 `true`。因此，当排序算法开始时，它比较前两个数字（`9` 和 `3`），如果第一个数字（`9`）小于第二个数字（`3`），则返回 `true`。在我们的例子中，规则返回 `false`，所以数字被反转。算法以这种方式继续排序，直到所有数字都按正确顺序排序。

要按升序排序数组，我们实际上可以使用 `sort()` 方法本身，而不使用闭包，如下所示：

```swift
var arrayOne = [9,3,6,2,8,5]
arrayOne.sort() 
```

之前的示例按数值递增顺序排序了数组；如果我们想反转顺序，我们会在闭包中反转参数。以下代码展示了如何反转排序顺序：

```swift
var arrayOne = [9,3,6,2,8,5]
arrayOne.sort(){ $1 < $0 }
//arrayOne contains 9,8,6,5,3 and 2 
```

当我们运行此代码时，`arrayOne` 将包含元素 `9`、`8`、`6`、`5`、`3` 和 `2`。

之前的代码可以通过使用 `sort(by:)` 方法并传入大于或小于运算符来简化，如下所示：

```swift
var arrayTwo = [9,3,6,2,8,5]
arrayTwo.sort(by: <) 
```

在之前的代码中，通过使用小于运算符，数组被按升序排序。如果我们使用大于运算符，数组将按降序排序。

### 排序

虽然排序算法在原地排序数组（即，它替换了原始数组），但 `sorted` 算法不会改变原始数组；它相反地创建了一个包含原始数组中排序元素的新数组。以下示例展示了如何使用排序算法：

```swift
var arrayOne = [9,3,6,2,8,5]
let sorted = arrayOne.sorted(){ $0 < $1 }
//sorted contains 2,3,5,6,8 and 9
//arrayOne contains 9,3,6,2,8 and 5 
```

在运行此代码后，`arrayOne` 将包含原始未排序的数组（`9`、`3`、`6`、`2`、`8` 和 `5`），而 `sorted` 数组将包含新的排序后的数组（`2`、`3`、`5`、`6`、`8` 和 `9`）。

### 过滤

**filter** 算法将通过过滤原始数组来返回一个新的数组。这是最强大的数组算法之一，最终可能会成为你使用最多的算法之一。如果你需要根据一组规则检索数组的子集，我建议使用此算法而不是尝试编写自己的方法来过滤数组。

闭包接受一个参数，并且它应该返回一个布尔值 `true`，如果元素应该包含在新的数组中，如下面的代码所示：

```swift
var arrayOne = [1,2,3,4,5,6,7,8,9]
let filtered = arrayOne.filter{$0 > 3 && $0 < 7}
//filtered contains 4,5 and 6 
```

在前面的代码中，我们传递给算法的规则是，如果数字大于 `3` 且小于 `7`，则返回 `true`；因此，任何大于 `3` 且小于 `7` 的数字都被包含在新的 `filtered` 数组中。

下一个示例展示了我们如何检索包含其名称中字母 `o` 的城市子集：

```swift
var city = ["Boston", "London", "Chicago", "Atlanta"] 
let filteredCity = city.filter{$0.range(of:"o") != nil}
//filtered contains "Boston", "London" and "Chicago" 
```

在前面的代码中，我们使用 `range(of:)` 方法来返回字符串是否包含字母 `o`。如果方法返回 `true`，则字符串被包含在 `filtered` 数组中。

### Map

虽然过滤器算法用于选择数组中的某些元素，但 **map** 用于对数组中的所有元素应用逻辑。以下示例展示了如何使用 map 算法将每个数字除以 10：

```swift
var arrayOne = [10, 20, 30, 40]
let applied = arrayOne.map{ $0 / 10}
//applied contains 1,2,3 and 4 
```

在前面的代码中，新数组包含数字 `1`、`2`、`3` 和 `4`，这是通过将原始数组中的每个元素除以 `10` 得到的结果。

由 map 算法创建的新数组不需要包含与原始数组相同的元素类型；然而，新数组中的所有元素必须属于同一类型。在以下示例中，原始数组包含整数值，但由 map 算法创建的新数组包含字符串元素：

```swift
var arrayTwo = [1, 2, 3, 4]
let applied = arrayTwo.map{ "num:\($0)"}
//applied contains "num:1", "num:2", "num:3" and "num:4" 
```

在前面的代码中，我们创建了一个字符串数组，该数组将原始数组中的数字追加到 `num:` 字符串。

### 计数

我们可以将过滤器算法与 `count` 方法结合起来，以计算匹配规则的数组中项目数量。例如，如果我们有一个包含测试成绩的数组，我们可以使用 **count** 算法来计算有多少成绩大于或等于 `90`，如下所示：

```swift
let arrayOne = [95, 90, 75, 80,60]
let count = arrayOne.filter{ $0 >= 90 }.count 
```

正如我们使用过滤器算法一样，我们可以使用数组类型的方法，例如字符串类型的 `range(of:)` 方法。例如，而不是像在过滤器算法中那样返回包含其名称中字母 `o` 的城市子集，我们可以这样计数城市：

```swift
var city = ["Boston", "London", "Chicago", "Atlanta"] 
let count1 = city.filter{$0.range(of:"o") != nil}.count 
```

在前面的计数中，`count1` 常量包含 `3`。

### Diff

在 Swift 5.1 和 SE-0240 中，引入了 **Diff** 算法。这个 Swift 语言的补充使得支持有序集合（如数组）的 `diff` 和修补成为可能。要真正看到这个变化的威力，我们需要了解 `switch` 语句的工作原理，这在 *第六章*，*控制流* 中介绍；因此，我们将简要展示如何使用 `applying` 方法实现 Diff 算法。我们将在 *第六章*，*控制流* 中介绍 `switch` 语句时更详细地研究这个算法。

让我们看看以下代码：

```swift
var scores1 = [100, 81, 95, 98, 99, 65, 87]
var scores2 = [100, 98, 95, 91, 83, 88, 72]
let diff2 = scores2.difference(from: scores1)
var newArray = scores1.applying(diff2) ?? [] 
```

在前面的代码中，我们首先创建了两个数组。然后我们使用了 `difference(from:)` 方法，它返回两个数组之间的差异。这个新数组现在将包含以下值：`100`，`98`，`95`，`91`，`83`，`88` 和 `72`。返回值是一个枚举集合，告诉我们如何从一个集合生成一个包含另一个集合相同元素的集合。现在这可能不太容易理解，但当我们深入研究时，它将变得清晰。

最后一行使用 `applying()` 方法将更改应用到 `scores1` 数组上，并返回一个与 `scores2` 数组具有相同元素的数组；因此，在调用此方法之后，`newArray` 数组包含与 `scores2` 数组相同的元素。

### forEach

我们可以使用 `forEach` 算法遍历一个序列。以下示例展示了我们如何进行操作：

```swift
var arrayOne = [10, 20, 30, 40]
arrayOne.forEach{ print($0) } 
```

这个示例将在控制台打印以下结果：

```swift
10
20
30
40 
```

虽然使用 `forEach` 算法非常简单，但它确实有一些限制。推荐遍历数组的方式是使用 `for-in` 循环，我们将在下一节中看到。

## 遍历数组

我们可以使用 `for-in` 循环按顺序遍历数组的所有元素。`for-in` 循环将为数组的每个元素执行一个或多个语句。我们将在 *第六章*，*控制流* 中更详细地讨论 `for-in` 循环。以下示例展示了我们如何遍历数组的元素：

```swift
var arrayOne = ["one", "two", "three"] 
for item in arrayOne {
    print(item)
} 
```

在前面的例子中，`for-in` 循环遍历数组，并为数组中的每个元素执行 `print(item)` 行。如果我们运行此代码，它将在控制台显示以下结果：

```swift
one
two
three 
```

有时候我们希望遍历数组，就像前面的例子中那样，但我们还希望知道元素的索引以及其值。为此，我们可以使用数组的 `enumerated` 方法，它为数组中的每个元素返回一个元组，包含元素的索引和值。以下示例展示了如何使用此函数：

```swift
var arrayOne = ["one", "two", "three"]
for (index,value) in arrayOne.enumerated() { 
    print("\(index) \(value)")
} 
```

前面的代码将在控制台显示以下结果：

```swift
one
two
three 
```

现在我们已经在 Swift 中介绍了数组，让我们继续介绍字典。

# 字典

虽然字典不像数组那样常用，但它们具有额外的功能，这使得它们非常强大。字典是一个容器，存储多个键值对，其中所有键都是同一类型，所有值也都是同一类型。键用作值的唯一标识符。由于我们是通过键而不是值的索引来查找值，因此字典不保证键值对存储的顺序。

字典非常适合存储与唯一标识符相对应的项目，其中唯一标识符应用于检索项目。国家和它们的缩写是字典中可以存储的项目的一个很好的例子。在下面的表中，我们展示了国家和它们的缩写作为键值对：

| 键 | 值 |
| --- | --- |
| US | 美国 |
| IN | 印度 |
| UK | 英国 |

表 5.1：国家和它们的缩写

## 创建和初始化字典

我们可以使用字典字面量来初始化字典，这与我们使用数组字面量初始化数组的方式类似。以下示例展示了如何使用前面的图表中的键值对创建字典：

```swift
let countries = ["US":"UnitedStates","IN":"India","UK":"UnitedKingdom"] 
```

前面的代码创建了一个不可变的字典，其中包含我们在之前图表中看到的每个键值对。就像数组一样，要创建一个可变的字典，我们需要在`let`的位置使用`var`关键字。以下示例展示了如何创建一个包含国家的可变字典：

```swift
var countries = ["US":"UnitedStates","IN":"India","UK":"UnitedKingdom"] 
```

在前面的两个示例中，我们创建了一个字典，其中键和值都是字符串。编译器推断出键和值都是字符串，因为这是初始化字典时使用的键和值的类型。如果我们想创建一个空字典，我们需要告诉编译器键和值的类型。以下示例创建具有不同键值类型的各种字典：

```swift
var dic1 = [String:String]()
var dic2 = [Int:String]()
var dic3 = [String:MyObject]()
var dic4: [String:String] = [:]
var dic5: [Int:String] = [:] 
```

如果我们想在字典中使用自定义对象作为键，我们需要使自定义对象符合 Swift 标准库中的 **Hashable** 协议。我们将在本书的后面部分广泛讨论协议，但就现在而言，只需了解可以使用自定义对象作为字典中的键。

现在让我们看看我们如何访问字典的值。

## 访问字典值

我们使用下标语法来检索特定键的值。如果字典不包含我们正在寻找的键，字典将返回`nil`；因此，从这个查找返回的变量是一个可选变量。以下示例展示了如何使用下标语法从字典中检索值：

```swift
let countries = ["US":"United States", "IN":"India","UK":"UnitedKingdom"] 
var name = countries["US"] 
```

在前面的代码中，`name`变量包含`美国`字符串。

## 计算字典中的键或值数量

我们使用字典的`count`属性来获取字典中键值对的数量。以下示例展示了如何使用此属性：

```swift
let countries = ["US":"United States", "IN":"India","UK":"United Kingdom"] 
var cnt = countries.count    //cnt contains 3 
```

在前面的代码中，`cnt`变量将包含数字`3`，因为字典中有三个键值对。

## 字典是否为空？

要测试一个字典是否包含任何键值对，我们可以使用`isEmpty`属性。如果字典包含一个或多个键值对，则此属性将返回`false`；如果它是空的，则返回`true`。以下示例展示了如何使用此属性来确定我们的字典是否包含任何键值对：

```swift
let countries = ["US":"United States", "IN":"India","UK":"United Kingdom"]
var empty = countries.isEmpty 
```

在前面的代码中，`isEmpty`属性返回`false`，因为字典中有三个键值对。

## 更新键的值

要在字典中更新键的值，我们可以使用下标语法或`updateValue(_:, forKey:)`方法。`updateValue(_:, forKey:)`方法有一个下标语法没有的附加功能：它在更改值之前返回与键关联的原始值。以下示例展示了如何使用下标语法和`updateValue(_:, forKey:)`方法来更新键的值：

```swift
var countries = ["US":"United States", "IN":"India","UK":"United Kingdom"]
countries["UK"] = "Great Britain"
//The value of UK is now set to "Great Britain"
var orig = countries.updateValue("Britain", forKey: "UK")
//The value of UK is now set to "Britain"
//The orig variable equals "Great Britain" 
```

在前面的代码中，我们使用下标语法将`UK`键关联的值从`United Kingdom`更改为`Great Britain`。在替换之前，我们没有保存`United Kingdom`的原始值。然后我们使用`updateValue(_:, forKey:)`方法将`UK`键关联的值从`Great Britain`更改为`Britain`。使用`updateValue(_:, forKey:)`方法，在字典中更改值之前，将`Great Britain`的原始值赋给`orig`变量。

## 添加键值对

要向字典中添加新的键值对，我们可以使用下标语法或与更新键值相同的方法`updateValue(_:, forKey:)`。如果我们使用`updateValue(_:, forKey:)`方法，并且键当前不在字典中，则此方法将添加一个新的键值对并返回`nil`。以下示例展示了如何使用下标语法和`updateValue(_:, forKey:)`方法将新的键值对添加到字典中：

```swift
var countries = ["US":"United States", "IN":"India","UK":"United Kingdom"] 
countries["FR"] = "France" //The value of "FR" is set to"France"
var orig = countries.updateValue("Germany", forKey: "DE")
//The value of "DE" is set to "Germany" and orig is nil 
```

在前面的代码中，`countries`字典最初有三个键值对，然后我们使用下标语法向字典中添加第四个键值对（`FR/France`）。我们使用`updateValue(_:,forKey:)`方法向字典中添加第五个键值对（`DE/Germany`）。`orig`变量被设置为`nil`，因为`countries`字典没有与`DE`键关联的值。

## 删除键值对

有时候我们需要从字典中删除值。有三种方法可以实现这一点：下标语法、`removeValue(forKey:)`方法或`removeAll()`方法。`removeValue(forKey:)`方法在删除之前返回键的值。`removeAll()`方法从字典中删除所有元素。以下示例展示了如何使用所有三种方法从字典中删除键值对：

```swift
var countries = ["US":"UnitedStates","IN":"India","UK":"United Kingdom"] 
countries["IN"] = nil //The "IN" key/value pair is removed
var orig = countries.removeValue(forKey:"UK")
//The "UK" key value pair is removed and orig contains "United Kingdom"
countries.removeAll()
//Removes all key/value pairs from the countries dictionary 
```

在前面的代码中，`countries` 字典最初包含三个键值对。然后我们将与 `IN` 键关联的值设置为 `nil`，从而从字典中删除键值对。我们使用 `removeValue(forKey:)` 方法删除与 `UK` 键关联的键。在删除与 `UK` 键关联的值之前，`removeValue(forKey:)` 方法将值保存在 `orig` 变量中。最后，我们使用 `removeAll()` 方法从 `countries` 字典中删除所有剩余的键值对。

现在我们来看一下集合类型。

# 集合

集合类型是一个类似于数组类型的泛型集合。虽然数组类型是有序集合，可能包含重复项，但集合类型是无序集合，其中每个元素必须是唯一的。

就像字典中的键一样，存储在数组中的类型必须符合 `Hashable` 协议。这意味着该类型必须提供一种方法来计算自己的哈希值。Swift 的所有基本类型，如 `String`、`Double`、`Int` 和 `Bool`，都符合此协议，并且默认情况下可以用于集合。

让我们看看如何使用集合类型。

## 初始化集合

初始化集合有几种方法。就像数组和字典类型一样，Swift 需要知道将要存储的数据类型。这意味着我们必须告诉 Swift 要在集合中存储的数据类型，或者用一些数据初始化它，以便它可以推断数据类型。

就像数组和字典类型一样，我们使用 `var` 和 `let` 关键字来声明集合是否可变：

```swift
//Initializes an empty set of the String type 
var mySet = Set<String>()
//Initializes a mutable set of the String type with initial values 
var mySet = Set(["one", "two", "three"])
//Creates an immutable set of the String type. 
let mySet = Set(["one", "two", "three"]) 
```

## 将元素插入到集合中

我们使用 `insert` 方法将元素插入到集合中。如果我们尝试插入一个已经存在于集合中的元素，该元素将被忽略。以下是将元素插入到集合中的示例：

```swift
var mySet = Set<String>() 
mySet.insert("One") 
mySet.insert("Two")
mySet.insert("Three") 
```

`insert()` 方法返回一个元组，我们可以使用它来验证值是否成功添加到集合中。以下示例显示了如何检查返回值以查看是否成功添加：

```swift
var mySet = Set<String>() 
mySet.insert("One")
mySet.insert("Two")
var results = mySet.insert("One")
if results.inserted {
    print("Success")
} else { 
    print("Failed")
  } 
```

在这个示例中，由于我们尝试将 `One` 值添加到已经包含该值的集合中，所以会打印出 `Failed` 到控制台。

## 确定集合中的元素数量

我们可以使用 `count` 属性来确定集合中的元素数量。以下是如何使用此方法的示例：

```swift
var mySet = Set<String>() 
mySet.insert("One") 
mySet.insert("Two")
mySet.insert("Three")
print("\(mySet.count) items") 
```

当执行此代码时，它将在控制台打印出消息 `3 items`，因为集合包含三个元素。

## 检查集合是否包含一个元素

我们可以使用 `contains()` 方法来验证集合是否包含一个元素，如下所示：

```swift
var mySet = Set<String>()
mySet.insert("One") 
mySet.insert("Two")
mySet.insert("Three")
var contain = mySet.contains("Two") 
```

在前面的示例中，`contain` 变量被设置为 `true`，因为集合包含一个值为 `Two` 的字符串。

## 遍历集合

我们可以使用 `for-in` 语句遍历集合中的元素，就像我们处理数组一样。以下示例显示了如何遍历集合中的元素：

```swift
for item in mySet { 
    print(item)
} 
```

前面的示例将打印出集合中的每个元素到控制台。

## 移除集合中的项目

我们可以移除集合中的一个项目或所有项目。要移除单个项目，我们会使用`remove()`方法，要移除所有项目，我们会使用`removeAll()`方法。以下示例展示了如何从集合中移除项目：

```swift
//The remove method will return and remove an item from a set 
var item = mySet.remove("Two")
//The removeAll method will remove all items from a set 
mySet.removeAll() 
```

## 集合操作

苹果提供了四种方法，我们可以使用这些方法从两个其他集合中构建一个集合。这些操作可以在一个集合上就地执行，或者用于创建一个新的集合。这些操作如下：

+   `union`和`formUnion`：这些方法创建一个集合，包含两个集合的所有唯一值，可以理解为移除了重复项。

+   `subtracting`和`subtract`：这些方法创建一个集合，包含第一个集合中不在第二个集合中的值。

+   `intersection`和`formIntersection`：这些方法创建一个集合，包含两个集合共有的值。

+   `symmetricDifference`和`formSymmetricDifference`：这些方法创建一个新集合，包含在任一集合中但不在两个集合中的值。

让我们看看一些例子，看看可以从这些操作中获得哪些结果。对于所有集合操作的例子，我们将使用以下两个集合：

```swift
var mySet1 = Set(["One", "Two", "Three", "abc"])
var mySet2 = Set(["abc","def","ghi", "One"]) 
```

第一个例子使用了`union`方法。该方法从两个集合中提取唯一值以创建另一个集合：

```swift
var newSetUnion = mySet1.union(mySet2) 
```

`newSetUnion`变量将包含以下值：`One`、`Two`、`Three`、`abc`、`def`和`ghi`。我们可以使用`formUnion`方法就地执行`union`函数，而不创建一个新的集合：

```swift
mySet1.formUnion(mySet2) 
```

在这个例子中，`mySet1set`集合将包含`mySet1`和`mySet2`集合的所有唯一值。

现在让我们看看`subtract`和`subtracting`方法。这些方法将创建一个集合，包含第一个集合中不在第二个集合中的值：

```swift
var newSetSubtract = mySet1.subtracting(mySet2) 
```

在这个例子中，`newSetSubtract`变量将包含`Two`和`Three`值，因为这两个值是唯一不在第二个集合中的值。

我们使用`subtract`方法就地执行减法函数，而不创建一个新的集合：

```swift
mySet1.subtract(mySet2) 
```

在这个例子中，`mySet1`集合将包含`Two`和`Three`值，因为这两个值是唯一不在`mySet2`集合中的值。

现在让我们看看`intersection`方法，它通过创建一个新集合，从两个集合中提取共同的值：

```swift
var newSetIntersect = mySet1.intersection(mySet2) 
```

在这个例子中，`newSetIntersect`变量将包含`One`和`abc`值，因为它们是两个集合共有的值。

我们可以使用`formIntersection()`方法就地执行交集函数，而不创建一个新的集合：

```swift
mySet1.formIntersection(mySet2) 
```

在这个例子中，`mySet1`集合将包含`One`和`abc`值，因为它们是两个集合共有的值。

最后，让我们看看`symmetricDifference()`方法。这些方法将创建一个新集合，包含在任一集合中但不在两个集合中的值：

```swift
var newSetExclusiveOr = mySet1.symmetricDifference(mySet2) 
```

在这个例子中，`newSetExclusiveOr`变量将包含`Two`、`Three`、`def`和`ghi`值。

在原地执行此方法，我们使用 `fromSymmetricDifference()` 方法：

```swift
mySet1.formSymmetricDifference(mySet2) 
```

这四种操作（并集、减集、交集和对称差集）增加了数组所不具备的功能。与数组相比，集合类型具有更快的查找速度，当集合的顺序不重要且集合中的实例必须是唯一的时候，集合类型可以是一个非常有用的替代方案。

# 摘要

在本章中，我们介绍了 Swift 集合。对 Swift 的原生集合类型有良好的理解对于架构和开发 Swift 应用程序至关重要，因为除了最基本的应用程序之外，所有应用程序都使用它们。

Swift 的三种集合类型是数组、集合和字典。数组以有序集合的形式存储数据。集合以无序集合的形式存储唯一值。字典以无序集合的形式存储键值对。

在下一章中，我们将探讨如何使用 Swift 的控制流语句。
