## 第二章：二维数组/矩阵

在物理学计算、图形设计和游戏设计中，尤其是基于网格的益智游戏中，常用的一个集合类型是二维数组/矩阵。二维数组简单来说就是数组的成员也是数组。这些数组可以以行列的形式以矩形方式表达。

例如，15 个拼图游戏中的 4x4（4 行，4 列）拼图板可以表示如下：

```swift
var tileBoard = [[1,2,3,4],
                 [5,6,7,8],
                 [9,10,11,12],
                 [13,14,15,""]]
```

在 15 个拼图游戏中，你的目标是使用一个空位（用空字符串`""`表示）移动拼图，使它们最终按照我们看到的 1-15 的顺序排列。游戏开始时，数字会以随机且可解的顺序排列，然后玩家必须交换数字和空白空间。

### 小贴士

为了更好地执行对 15 游戏（和其他游戏）中每个拼图的各种操作和/或存储有关每个拼图的信息，最好创建一个拼图对象，而不是使用这里看到的原始值。为了理解矩阵或二维数组是什么，只需注意数组是如何被双重括号`[[]]`包围的。我们将在后面的示例游戏中使用`SwiftSweeper`来更好地理解益智游戏是如何使用对象的二维数组来创建完整游戏的。

这里有一些声明空白二维数组的方法，使用严格类型：

```swift
var twoDTileArray : [[Tiles]] = []     //blank 2D array of type,Tiles
var anotherArray = Array<Array<Tile>>()  //same array, using Generics
```

变量`twoDTileArray`使用双括号`[[Tiles]]`来声明它为一个空白的二维数组/矩阵，用于虚构的类型，即`tiles`。变量`anotherArray`是一个相当奇怪的声明数组，它使用尖括号字符`<>`作为括号。它使用的是所谓的**泛型**。泛型是一个相当高级的话题，我们将在后面更多地讨论。它们允许在广泛的数据类型和类之间实现非常灵活的功能。目前，我们可以将其视为处理对象的一种通用的方法。

要填充数组的这两种版本的数据，我们就会使用 for 循环。关于循环和迭代的更多内容将在本章后面解释。

## 集合

这是我们如何在 Swift 中创建各种游戏物品集合的方法：

```swift
var keyItems = Set([Dungeon_Prize, Holy_Armor, Boss_Key,"A"])
```

这个`keyItems`集合包含各种对象和一个字符`A`。与数组不同，集合是无序的，包含唯一的项。因此，与`stageNames`不同，尝试获取`keyItems[1]`会返回错误，而`items[1]`可能不一定是`Holy_Armor`对象，因为在集合中对象的放置是内部随机的。集合相对于数组的优势在于，集合非常适合检查集合中的重复对象和特定内容搜索。集合使用散列来定位集合中的项目，因此检查集合内容中的项目可能比在数组中更快。在游戏开发中，游戏的关键物品，玩家可能只能获得一次且不应有重复，可以作为集合使用得很好。使用函数`keyItems.contains(Boss_Key)`在这种情况下返回`true`的布尔值。

### 注意

集合是在 Swift 1.2 和 Xcode 6.3 中添加的。它们的类由泛型类型 `Set<T>` 表示，其中 `T` 是集合的类类型。换句话说，集合 `Set([45, 66, 1233, 234])` 将是 `Set<Int>` 类型，而我们的示例将是一个 `Set<NSObject>` 实例，因为它包含各种数据类型的集合。

我们将在本章后面讨论更多关于泛型和类层次的内容。

## 字典

在 Swift 中，字典可以这样表示：

```swift
var playerInventory: [Int : String]  =  [1 : "Buster Sword",  43 : "Potion", 22: "StrengthBooster"]
```

字典使用 `键 : 值` 关联，因此 `playerInventory[22]` 根据键 `22` 返回值 `StrengthBooster`。键和值都可以初始化为几乎任何类类型***。除了给出的库存示例外，我们还可以有以下的代码：

```swift
var stageReward: [Int : GameItem] = [:] //blank initialization
//use of the Dictionary at the end of a current stage
stageReward = [currentStage.score : currentStage.rewardItem]

```

### 注意

***在 Swift 中，尽管字典的值相当灵活，但确实存在限制。键必须符合所谓的可哈希协议。基本数据类型，如 `Int` 和 `String`，已经具有这种功能。因此，如果你要创建自己的类/数据结构，这些类/数据结构将用于字典，例如将玩家动作与玩家输入映射，必须首先使用此协议。我们将在本章后面讨论更多关于协议的内容。***

字典类似于集合，因为它们是无序的，但它们还有一个额外的层次，即与内容相关联的键和值，而不是仅仅的哈希键。像集合一样，字典非常适合快速插入和检索特定数据。在 iOS 应用和 Web 应用中，字典用于解析和选择 JavaScript 对象表示法 (JSON) 数据。

在游戏开发领域，可以使用 JSON 或通过 Apple 的内部数据类 `NSUserDefaults` 来保存和加载游戏数据、设置游戏配置或访问游戏 API 的特定成员。

例如，以下是在 iOS 游戏中使用 Swift 保存玩家最高分的其中一种方法：

```swift
let newBestScore : Void = NSUserDefaults.standardUserDefaults().setInteger(bestScore, forKey: "bestScore")
```

这段代码直接来自一个名为 PikiPop 的已发布的 Swift 开发游戏，我们将不时使用它来展示实际游戏应用中使用的代码。

再次提醒，字典是无序的，但 Swift 有方法遍历或搜索整个字典。我们将在下一节以及后续的循环和控制流部分进行更深入的讨论。

# 可变/不可变集合

我们遗漏的一个重要讨论是如何从数组、集合和字典中减去、编辑或添加内容。然而，在我们这样做之前，你应该理解可变和不可变数据/集合的概念。

可变集合是简单的数据，可以对其进行更改、添加或减去，而不可变集合则不能进行更改、添加或减去。

为了在 Objective-C 中有效地处理可变和不可变集合，我们必须事先明确声明集合的可变性。例如，Objective-C 中类型为`NSArray`的数组始终是不可变的。我们可以调用`NSArray`上的方法来编辑集合，但幕后，这将创建全新的`NSArray`对象，因此在游戏生命周期中经常这样做会相当低效。Objective-C 通过类类型`NSMutableArray`解决了这个问题。

多亏了 Swift 类型推断的灵活性，我们已经知道如何使集合可变或不可变！在 Swift 中，关于数据可变性的常量和变量概念已经涵盖了。在创建集合时使用关键字`let`将使该集合不可变，而使用`var`将初始化它为可变集合。

```swift
//mutable Array
var unlockedLevels : [Int] =  [1, 2, 5, 8]

//immutable Dictionary
let playersForThisRound : [PlayerNumber:PlayerUserName] = [453:"userName3344xx5", 233:"princeTrunks", 6567: "noScopeMan98", 211: "egoDino"]
```

整数数组`unlockedLevels`可以简单地编辑，因为它是一个变量。不可变的字典`playersForThisRound`不能更改，因为它已经被声明为常量。关于额外的类类型没有额外的模糊性。

## 编辑/访问集合数据

只要集合类型是变量，使用`var`关键字，我们就可以对数据进行各种编辑。让我们回到我们的`unlockedLevels`数组。许多游戏在玩家进步时都有解锁级别的功能。假设玩家达到了解锁之前被锁定的第 3 关所需的高分（因为`3`不是数组的成员）。我们可以使用`append`函数将`3`添加到数组中：

```swift
unlockedLevels.append(3)
```

Swift 的另一个巧妙属性是，我们可以使用`+=`赋值运算符向数组添加数据：

```swift
unlockedLevels += [3]
```

然而，这样做只会将`3`添加到数组的末尾。所以，我们之前的数组`[1, 2, 5, 8]`现在变成了`[1, 2, 5, 8, 3]`。这可能不是期望的顺序，因此为了在第三个位置插入数字`3`，即`unlockedLevels[2]`，我们可以使用以下方法：

```swift
unlockedLevels.insert(3, atIndex: 2)
```

现在，我们的解锁级别数组已按`[1, 2, 3, 5, 8]`排序。

这是在假设我们知道数组中索引为`3`之前的元素已经排序。Swift 提供了各种排序功能，可以帮助保持数组排序。我们将把排序的细节留到本章后面的循环和控制流讨论中。

从数组中移除项目非常简单。让我们再次使用我们的`unlockedLevels`数组。想象一下，我们的游戏有一个玩家可以前往和返回的开放世界，而玩家刚刚解锁了一个触发事件并阻止访问第 1 关的秘密。现在必须从解锁的级别中移除第 1 关。我们可以这样做：

```swift
unlockedLevels.removeAtIndex(0) // array is now  [2, 3, 5, 8]
```

或者，想象一下，玩家已经失去了所有的生命，并得到了一个 **游戏结束** 的消息。对此的惩罚可能是锁定最远的关卡。尽管这可能是一个相当令人愤怒的方法，并且我们知道第 8 关是我们数组中最远的关卡，但我们可以使用数组类型的 `.removeLast()` 函数将其删除。

```swift
unlockedLevels.removeLast() // array is now [2,3,5]
```

### 注意

这假设我们知道集合的确切顺序。集合或字典可能更适合控制你游戏中的某些方面。

这里有一些编辑集合或字典的快速指南。

**集合**

```swift
inventory.insert("Power Ring")    //.insert() adds items to a set
inventory.remove("Magic Potion")  //.remove() removes a specific item
inventory.count                   //counts # of items in the Set
inventory.union(EnemyLoot)        //combines two Sets
inventory.removeAll()             //removes everything from the Set
inventory.isEmpty                 //returns true
```

**字典**

```swift
var inventory = [Float : String]() //creates a mutable dictionary

/*
one way to set an equipped weapon in a game; where 1.0 could represent the first "item slot" that would be placeholder for the player's "current weapon"  
*/
inventory.updateValue("Broadsword", forKey: 1.0)

//removes an item from a Dictionary based on the key value
inventory.removeValueForKey("StatusBooster")

inventory.count                   //counts items in the Dictionary
inventory.removeAll(keepCapacity: false) //deletes the Dictionary
inventory.isEmpty                 //returns false

//creates an array of the Dictionary's values
let inventoryNames = String

//creates an array of the Dictionary's keys
let inventoryKeys = String
```

### 迭代集合类型

我们在讨论集合类型时不能不提如何成批迭代它们。

下面是我们在 Swift 中迭代数组、集合或字典的一些方法：

```swift
//(a) outputs every item through the entire collection
  //works for Arrays, Sets and Dictionaries but output will vary
for item in inventory {
    print(item)
}

//(b) outputs sorted item list using Swift's sorted() function
  //works for Sets
for item in sorted(inventory) {
    print("\(item)")
}

//(c) outputs every item as well as its current index
  //works for Arrays, Sets and Dictionaries
for (index, value) in enumerate(inventory) {
    print("Item \(index + 1): \(value)")
}

//(d)
//Iterate through and through the keys of a Dictionary
for itemCode in inventory.keys {
    print("Item code: \(itemCode)")
}

//(e)
//Iterate through and through the values of a Dictionary
for itemName in inventory.values {
    print("Item name: \(itemName)")
}
```

如前所述，这是通过所谓的 for-loop 实现的；在这些例子中，我们展示了 Swift 如何使用 `in` 关键字利用 for-in 变体。在这些所有例子中，代码将重复，直到达到集合的末尾。在示例（c）中，我们还看到了 Swift 函数 `enumerate()` 的使用。这个函数为每个项目返回一个复合值，`(index,value)`。这个复合值被称为元组，Swift 对元组的利用为函数、循环以及代码块提供了广泛的功能。

我们将在后面更深入地探讨元组、循环和代码块。

# Objective-C 和 Swift 比较

这里快速回顾一下我们的 Swift 代码，并与 Objective-C 的等效代码进行比较。

## Objective-C

下面是 Objective-C 的示例代码：

```swift
const int MAX_ENEMIES = 10;  //constant
float playerPower = 1.3;     //variable

//Array of NSStrings
NSArray * stageNames = @[@"Downtown Tokyo", @"Heaven Valley", @" Nether"];

//Set of various NSObjects
NSSet *items = [NSSet setWithObjects: Weapons, Armor,
 HealingItems,"A", nil];

//Dictionary with an Int:String key:value
NSDictionary *inventory = [NSDictionary dictionaryWithObjectsAndKeys:
             [NSNumber numberWithInt:1], @"Buster Sword",
             [NSNumber numberWithInt:43], @"Potion",
             [NSNumber numberWithInt:22], @"Strength",
nil];
```

## Swift

下面是 Swift 中等效的代码：

```swift
let MAX_ENEMIES = 10          //constant
var playerPower = 1.3         //variable

//Array of Strings
let stageNames : [String] = ["Downtown Tokyo","Heaven Valley","Nether"]    

//Set of various NSObjects
var items = Set([Weapons, Armor, HealingItems,"A"])

//Dictionary with an Int:String key:value
var playerInventory: [Int : String]  =  [1 : "Buster Sword",  43 : "Potion", 22: "StrengthBooster"]
```

在前面的代码中，我们使用了一些变量、常量、数组、集合和字典的例子。首先，我们看到它们的 Objective-C 语法，然后是使用 Swift 语法等效声明的声明。从这个例子中，我们可以看到 Swift 与 Objective-C 相比是多么紧凑。

# 字符和字符串

在本章的某些时间里，我们一直在提到字符串。字符串也是一种数据类型集合，但它是专门处理字符的集合，属于字符串类。Swift 是 Unicode 兼容的，因此我们可以有如下字符串：

```swift
let gameOverText =  "Game Over!"
```

我们可以有如下带有表情符号字符的字符串：

```swift
let cardSuits =  "♠ ♥ ♣ ♦"
```

在前面的代码中，我们创建了一个所谓的字符串字面量。字符串字面量是我们明确地在两个引号 "" 中定义字符串。

我们可以为游戏中的后续使用创建空字符串变量，例如：

```swift
var emptyString = ""               // empty string literal
var anotherEmptyString = String()  // using type initializer
```

这两种方法都是创建空字符串 "" 的有效方式。

## 字符串插值

我们还可以从其他数据类型的混合中创建字符串，这被称为**字符串插值**。字符串插值在游戏开发、调试以及一般字符串使用中相当常见。

最显著的例子是显示玩家的得分和生命值。这就是我们其中一个示例游戏 PikiPop 使用字符串插值来显示当前玩家状态的方式：

```swift
//displays the player's current lives
var livesLabel = "x \(currentScene.player!.lives)"

//displays the player's current score
var scoreText = "Score: \(score)"
```

注意`\(variable_name)`的格式化。我们实际上在之前的代码片段中已经见过这种格式化。在各种`print()`输出中，我们使用它来显示我们想要获取信息的变量、集合等。在 Swift 中，要在字符串中输出数据类型的值，可以使用这种格式化。

对于那些来自 Objective-C 的人来说，这与以下内容相同：

```swift
NSString *livesLabel = @"Lives: ";
int lives = 3;
NSString *livesText = [NSString stringWithFormat:@" %@ (%d days ago)", livesLabel, lives];
```

注意 Swift 如何使字符串插值比其 Objective-C 前辈更干净、更容易阅读。

### 修改字符串

有多种方法可以更改字符串，例如像我们对集合对象所做的那样向字符串中添加字符。以下是一些基本示例：

```swift
var gameText = "The player enters the stage"
gameText += " and quickly lost due to not leveling up"
/* gameText now says
"The player enters the stage and lost due to not leveling up" */
```

由于字符串本质上是由字符组成的数组，就像数组一样，我们可以使用`+=`赋值运算符向之前的字符串中添加内容。

此外，类似于数组，我们可以使用`append()`函数向字符串的末尾添加一个字符。

```swift
 let exclamationMark: Character = "!"
gameText.append(exclamationMark)
//gameText now says "The player enters the stage and lost due to not leveling up!"
```

下面是如何在 Swift 中遍历字符串中的字符：

```swift
for character in "Start!" {
    print(character)
}
//outputs:
//S
//t
//a
//r
//t
//!
```

注意我们再次使用了 for-in 循环，并且甚至有使用字符串字面量作为迭代内容的灵活性。

### 字符串索引

数组和字符串之间的另一个相似之处是，可以通过索引找到字符串的各个字符。然而，与数组不同，由于字符可以是不同大小的数据，可以分解为 21 位的数字，称为 Unicode 标量，因此它们不能在 Swift 中使用`Int`类型的索引值来定位。

相反，我们可以使用字符串的`.startIndex`和`.endIndex`属性，并通过`.successor()`和`.predecessor()`函数分别向前或向后移动一个位置，以检索所需的字符或字符。

下面是一些使用我们之前的`gameText`字符串的这些属性和函数的示例：

```swift
gameText[gameText.startIndex]              // = T
gameText[gameText.endIndex]                // = !
gameText[gameText.startIndex.successor()]  // = h
gameText[gameText.endIndex.predecessor()]  // = p
```

### 注意

有许多方法可以操作、混合、删除和检索字符串和字符的各种方面。有关更多信息，请务必查看官方 Swift 文档中关于字符和字符串的说明，链接为[`developer.apple.com/library/ios/documentation/Swift/Conceptual/Swift_Programming_Language/StringsAndCharacters.html`](https://developer.apple.com/library/ios/documentation/Swift/Conceptual/Swift_Programming_Language/StringsAndCharacters.html)。

# Swift 中的注释

在我们迄今为止的代码片段中，可能会注意到带有双斜杠`//`或带有斜杠和星号的`/* */`的注释。这是我们如何在 Swift 代码中进行注释或标记的方法。任何在 C++、Java、Objective-C、JavaScript 或其他语言中编码过的人都会看到 Swift 实际上工作方式相同。

单行注释以双斜杠`//`开始，而多行注释或注释块以`/*`开始，以`*/`结束。

以下是一个示例：

```swift
//This is a single line comment
/*
This is a comment block
that won't end until it reaches the closing asterisk/forward slash characters
 */
```

注释用于帮助导航代码、理解代码可能执行的操作，以及注释掉我们可能不想执行但希望保留以备后用的代码行（即，`print()`日志调用或备选的起始属性值）。

### 注意

从 Xcode 6 Beta 4 开始，我们还可以利用以下注释：

`// MARK`:, `// TODO:`, 和 `// FIXME`。`//MARK`与 Objective-C 的`#pragma` mark 等效，允许程序员为你的代码中的*部分*添加标签，这些标签可以在 Xcode 顶部的面包屑下拉列表中访问。`// TODO:`和`// FIXME`使我们能够将代码的一部分划分出来，我们希望将来可能添加功能或调试。即使结构良好的游戏类也可以让人难以梳理。这些额外标记工具的添加使得规划和使用我们游戏的代码变得更加容易。

# Boolean

所有编程、游戏或其他领域的一个基本组成部分是使用**布尔**值。布尔值通常返回`true`或`false`、`yes`或`no`、`0`或`1`的值。在 Swift 中，这是`Bool`类对象的工作。在我们过去的集合数据类型示例中使用`.isEmpty()`函数返回一个布尔值`true`或`false`，这取决于该集合是否为空。

在游戏开发中，我们可以使用布尔值的一种方式是有一个全局变量（一个可以在游戏/应用程序范围内访问的变量），用来检查游戏是否结束。

```swift
var isGameOver = false
```

这个变量来自 PikiPop 游戏，游戏开始时使用一个名为`isGameOver`的布尔类型变量，初始值为`false`。如果游戏事件导致此值变为`true`，则这会触发与游戏结束状态相关的事件。

### 注意

与 Objective-C 中的布尔值不同，Swift 只使用`true`或`false`值来表示布尔变量。*Swift 严格的类型安全*不允许使用`YES`和`NO`或`0`和`1`，正如我们在 Objective-C 和其他编程语言中看到的那样。

然而，阅读和控制我们游戏中的此类信息，即所谓的游戏状态，最好使用不止一个布尔值。这是因为你的游戏以及游戏中的角色可能有各种状态，例如*游戏结束*、*暂停*、*生成*、*空闲*、*运行*、*下落*等等。一个称为*状态机*的特殊对象最适合管理此类信息。当讨论**GameplayKit**框架时，我们将更详细地介绍状态机。

# 整数、无符号整数、浮点数和双精度浮点数

除了布尔值之外，我们到目前为止简要提到的另一种基本数据类型是各种数值对象，例如整数（Ints）、无符号整数（UInts）、浮点数/小数（floats）和双精度浮点数/小数（doubles）。

## 整数和无符号整数

整数表示负数和正数，而无符号整数表示正数。与 C 和其他编程语言一样，Swift 允许我们创建从 8 位、16 位、32 位和 64 位的各种整数和无符号整数。例如，Int32 类型是 32 位整数，而 UInt8 类型是 8 位无符号整数。Ints 和 UInts 的位数大小表示分配给存储值的多少空间。以我们的 UInt8 例子来说，由这种无符号 Int 类型构成的数字只能存储 0-255（或二进制系统中的 11111111）的值。这也被称为 1 字节（8 位）。如果我们需要存储大于 255 或负数，那么可能 Int16 类型就足够了，因为它可以存储介于 –32767 和 32767 之间的数字。通常，我们不必太担心我们的整数变量和常量分配的大小。因此，在大多数情况下，只需使用 `Int` 类名即可。

### 注意

`Int` 的大小将取决于我们正在工作的系统类型。如果我们在一个 32 位系统上编译我们的代码，整数将是 Int32，而在 64 位系统上相同的整数将是 Int64。

Swift 允许我们通过 `.min` 或 `.max` 类变量（即 `Int16.max = 32767` 和 `UInt.min = 0`）查看 `Int` 变量的最小和最大值。

## 浮点数和双精度浮点数

浮点数是 32 位浮点数/分数，例如 π（3.14）或黄金比例 φ（1.61803）。

在游戏设计中，我们经常使用浮点数和范围，无论是确定 2D 精灵的 *x* 和 *y* 坐标点，使用线性插值平滑 3D 空间中游戏的摄像机移动，还是在对象或 2D/3D 向量上应用各种物理力。每种情况下所需的精度将决定是否需要浮点数或 64 位浮点值，即双精度浮点数。双精度浮点数可以精确到 15 位小数，而浮点数可以精确到 6 位小数。

### 小贴士

实际上，在可以使用浮点数或双精度浮点数的情况下，使用双精度浮点数是最佳实践。

# Swift 中的对象

面向对象编程（OOP）的核心方面当然是对象的概念。C++ 在编程中开启了这种范式，而 Java、C#、苹果的 Objective-C 以及其他语言都基本上是从这个基础上构建的。

Swift 是一种面向对象（OOP）的语言，它具有与 Objective-C 相同的动态对象模型，但以更简洁、类型安全和紧凑的方式呈现。

你可以将对象理解为它的字面意思，一个抽象的*事物*或*容器*。一个对象可以像字符串这样简单，也可以像最新视频游戏中的玩家对象那样复杂。从技术角度讲，程序中的对象是对分配的内存块中一组各种数据的*引用*，但只需理解对象可以是变量或类、结构体或代码块的实例的引用即可。

一个对象可以与各种数据字段/方面相关联，例如属性、函数、父对象、子对象和协议。例如，在 C 语言中，一个整型变量通常只表示为原始数据，但 Swift 中的整型实际上是一个对象。因此，我们可以在代码中对 `Int` 对象访问额外信息和执行函数。我们之前在 `Int.max` 变量中看到了这一点，它返回 `Int` 类可以表示的最高数值。同样，这取决于你正在工作的机器，这可能是 `Int32.max` 或 `Int64.max` 的相同值。

```swift
var highestIntNumber : Int = Int.max
```

访问对象的函数和属性使用点符号，正如我们在前面的例子中所看到的。`Int.max` 和 `Int.min` 实际上是称为 **类变量** 的特殊属性，它们代表所有 `Int` 类型对象的实例。

让我们看看 Swift 如何通过一个虚构的 `Player` 类型对象来获取对象的属性和函数。

```swift
let currentPlayer = Player(name:"Fumi")       //(a)
let playerName = currentPlayer.getName()      //(b)
var playerHealth = currentPlayer.health       //(c)
currentPlayer.attackEnemy()                   //(d)
```

我们将回到行 `(a)` 的后半部分，但只需理解它创建了一个名为 `currentPlayer` 的 `Player` 类型对象的实例。行 `(c)` 创建了一个名为 `playerHealth` 的变量，它通过 `currentPlayer` 的 `health` 属性设置；这里使用的是 *点符号*。行 `(b)` 和 `(d)` 使用点符号调用 `getName()` 和 `attackEnemy()` 函数。在这个例子中，`getName()` 函数是一个返回字符串并将其分配给常量 `playerName` 的函数。行 `(c)` 创建了一个名为 `playerHealth` 的变量，它通过引用 `currentPlayer` 的健康属性创建，也使用点符号。行 `(d)` 是直接调用 `Player` 类的 `attackEnemy()` 函数，你可以想象它现在只是执行 `currentPlayer` 的攻击。这个函数不返回任何值，因此被称为 `void` 类型函数。

至于行 `(a)`，有人可能会注意到它没有使用点符号。这就是 Swift 实现所谓的类初始化器的方式；由类名后面的括号 `()` 和参数 `name`（发送一个字符串，`Fumi`）到对象的类初始化器指定。

当我们继续到函数和类时，我们将更深入地探讨 *对象* 的使用。

## 类型安全和类型推断

在 Swift 中，对象及其函数是类型安全的，正如我们将要看到的。这意味着如果我们对一个字符串对象执行一个预期为整数的函数，编译器将在早期阶段警告我们。在游戏设计的范畴内，如果我们让玩家执行只有敌人应该做的动作，Swift 将通过其固有的类型安全特性知道这一点。

Swift 的类型推断是我们之前提到过的。与其他语言不同，你每次初始化对象时都必须声明其类型，Swift 会推断你想要什么类型。例如，我们有以下内容：

```swift
var playerHealth = 100
//Swift automatically infers that playerHealth is an Int object
```

# 可选参数

如我们之前所述，Swift 是一种类型安全的语言。苹果公司创建 Swift 的初衷也是为了将尽可能多的潜在错误和 bug 保留在开发阶段的编译状态，而不是在运行时。尽管 Xcode 拥有一些出色的调试工具，如断点、日志和 LLDB 调试器，但运行时错误，尤其是在游戏中，可能很难发现，这可能会导致开发过程停滞。为了在编译期间保持类型安全并尽可能无 bug，Swift 处理了**可选**的概念。

简而言之，可选是可能为空或开始为空的对象。当然，nil 是一个没有任何引用的对象。

在 Objective-C 中，我们可以为游戏声明以下字符串变量：

```swift
NSString *playerStatus = @"Poisoned";
playerStatus = nil;
```

在 Swift 中，我们会以同样的方式编写，但我们会很快发现 Xcode 会因为这样做而给出编译错误：

```swift
var playerStatus = "Poisoned"
playerStatus = nil      //error!
```

对于任何刚开始接触 Swift 的人来说，如果我们做了一件像这样简单的事情，我们也会得到一个错误：

```swift
var playerStatus : String   //error
```

在我们的游戏中创建空/未声明的对象是有意义的，并且这是我们经常想在类开始时想要做的。我们希望有这种灵活性，以便根据游戏事件稍后分配值。Swift 似乎使这样一个基本概念变得不可能实现！不用担心；在大多数情况下，Xcode 会通知你，在这些`nil`对象末尾添加一个问号`?`。这就是如何声明一个可选对象。

因此，如果我们想在 Swift 中规划游戏属性和对象，我们可以这样做：

```swift
var playerStatus : String?  //optional String
var stageBoss : Boss?       //optional Boss object
```

## 解包可选

让我们想象一下，我们想在游戏中显示导致玩家失败的原因。

```swift
var causedGameOver:String? = whatKilledPlayer(enemy.recentAttack)
let text = "Player Lost Because: "
let gameOverMessage = text + causedGameOver  //error

```

因为字符串`causedGameOver`是可选的，Xcode 会因为我们没有解包可选而给出编译错误。要在可选中解包值，我们在可选末尾添加一个感叹号`!`。

这是我们的`Game Over`消息代码，现在使用解包的可选进行了修复：

```swift
var causedGameOver:String? = whatKilledPlayer(enemy.recentAttack)
let text = "Player Lost Because: "
let gameOverMessage = text + causedGameOver!  //code now compiles!

```

我们还可以在声明时强制解包可选，以便在运行时而不是在编译时处理任何潜在的错误。这种情况经常发生在`@IBOutlets`和`@IBActions`（与各种基于菜单/视图工具的故事板和其他工具链接的对象和函数）。

```swift
@IBOutlet var titleLabel: UILabel!      //label from a Storyboard
var someUnwrappedOptional : GameObject! //our own unwrapped optional
```

### 注意

尽管如此，如果可能的话，建议尽可能多地使用基本的包装可选`?`，以便让编译器找到任何潜在的错误。通过使用所谓的可选绑定和链式调用，我们可以在可选对象上进行一些早期的逻辑检查，这在之前的语言中可能需要涉及各种`if`语句/控制流语句来简单地检查空对象。

保持代码干净、安全且易于阅读是 Swift 的目标，也是为什么 Swift 有时会不遗余力地强制执行许多这些规则，特别是关于可选的。

## 可选绑定和链式调用

可选绑定是检查可选是否有值。这是使用非常方便的 if-let 或 if-var 语句完成的。让我们回顾一下我们之前的代码：

```swift
var causedGameOver:String? = whatKilledPlayer(enemy.recentAttack)
let text = "Player Lost Because: "
if let gotCauseOfDeath = causedGameOver {
    let gameOverMessage = text + gotCauseOfDeath
}
```

代码块 `if let gotCauseOfDeath = causedGameOver{…}` 做了两件事情。首先，使用关键字 `if let`，它自动创建一个名为 `gotCauseOfDeath` 的常量，并将其绑定到可选的 `causedGameOver`。这同时检查 `causedGameOver` 是否为 `nil` 或有值。如果不是 `nil`，那么 `if` 语句的代码块将执行；在这种情况下，创建一个名为 `gameOverMessage` 的常量，该常量将 `text` 常量与 `gotCauseOfDeath` 结合起来。

我们可以使用 if-var 进一步简化这一点：

```swift
let text = "Player Lost Because: "
if var causedGameOver = whatKilledPlayer(enemy.recentAttack) {
    let message = text + causedGameOver
}
```

if-var 语句使用我们之前使用的可选 `causedGameOver` 创建一个临时变量，并根据 `whatKilledPlayer(enemy.recentAttack)` 的结果进行布尔逻辑检查。如果返回一个非 `nil` 值，则该语句为真。注意，在这种情况下，我们不必使用可选的包装（`?`）或强制解包（`!`）。

可选链是在查询对象属性时使用点操作符，同时进行 nil/值检查，就像我们使用可选绑定时做的那样。例如，假设我们有一个游戏，其中某些敌对类型可以通过名为 `currentEnemy` 的敌对实例立即使玩家失去生命。在这个例子中，`currentEnemy.type` 将是一个返回击中玩家的敌对类型的字符串。可选链使用自定义点修饰符 `?.` 在访问可能为 `nil` 的属性时进行检查。以下代码可以帮助我们更好地理解它是如何工作的：

```swift
if let enemyType = currentEnemy?.type {
    if enemyType == "OneHitKill"
    {
      player.loseLife()  //run the player's lost l
    }
}
```

很可能我们不会创建一个没有指定类型的敌对者，但为了理解可选链，观察一下这个检查 `currentEnemy.type` 可能返回的 `nil` 对象是如何进行的。就像点操作符的功能一样，你可以深入到属性和属性的属性中，同样的，你也可以使用重复的 `?.per` 属性进行深入。在这个代码中，我们使用 `==` 进行布尔比较，以查看 `enemyType` 是否是字符串 `OneHitKill`。

如果 `if` 语句的语法有点神秘，不要担心；接下来，我们将讨论 Swift 如何使用 `if` 语句、循环以及其他我们可以控制各种对象数据和它们功能的方法。

## Swift 中的控制流

任何程序中的控制流只是代码中指令和逻辑的顺序。Swift，就像任何其他编程语言一样，使用各种语句和代码块来循环、更改和/或迭代你的对象和数据。这包括 `if` 语句、for 循环、do-while 循环和 Switch 语句等代码块。这些包含在函数中，构成了更大的结构，如类。

### 如果语句

在我们继续讨论 Swift 如何处理面向对象编程的主要主题之一，即函数和类之前，让我们快速回顾一下 if-else 语句。`if` 语句检查一个布尔语句是 `true` 还是 `false`。以下是一个示例：

```swift
if player.health <= 0{
   gameOver()
}
```

这检查玩家的健康是否小于或等于 `0`，由 `<=` 运算符表示。注意，Swift 可以接受没有括号，但如果我们愿意或者如果语句变得更复杂，比如这个例子，我们可以使用它：

```swift
if (player.health <=0) && (player.lives <=0){ //&& = "and"
   gameOver()
}
```

在这里，我们不仅检查玩家是否失去了所有健康，还检查他们的生命是否全部消失，使用 `&&` 运算符。在 Swift 中，就像在其他语言中一样，我们使用括号将单个布尔检查分开，并且像其他语言一样，我们使用两个竖线键 (`||`) 进行逻辑或检查。

这里是 Swift 中编写 `if` 语句的一些更多方法，包括添加了关键字 else-if 和 else，以及 Swift 如何检查非某条语句：

```swift
//(a)
if !didPlayerWin { stageLost() }

//(b)
if didPlayerWin
{            
   stageWon()
}
else
{
  stageLost()
}

//(c)
if (enemy == Enemy.angelType){enemy.aura = angelEffects}
else if(enemy == Enemy.demonType){enemy.aura = demonEffects}
else{ enemy.aura = normalEffects }

//(d)
if let onlinePlayerID = onlineConnection()?.packetID?.playerID
{
  print("Connected PlayerID: /(onlinePlayerID)"
}

//(e)
if let attack = player.attackType, power = player.power where power != 0 {
    hitEnemy(attack, power)
}

//(f)
let playerPower = basePower + (isPoweredUp ? 250 : 50)
```

让我们看看我们在代码中放入了什么：

+   `(a)`: 这通过 `!statement` 使用感叹号 `!` 检查语句的非 / 反转。

+   `(b)`: 这检查玩家是否获胜。否则，使用关键字 `else` 调用 `stageLost()` 函数。

+   `(c)`: 这检查一个敌人是否是天使，并相应地设置其光环效果。如果不是，则使用 else-if 检查它是否是恶魔，如果不是，则使用 `else` 语句捕获所有其他实例。我们可以有一系列连续的 else-if 语句，但如果开始堆叠太多，那么使用 for 循环和 Switch 语句将是一个更好的方法。

+   `(d)`: 使用可选链，我们根据 `if` 创建一个 `onlineID` 常量；我们能够使用 if-let 获取非 nil 的 `playerID` 属性。

+   `(e)`: 这使用 if-let，在 Swift 1.2 中可选绑定成为了一个特性。与在后端网络开发中如何执行 SQL 查询类似，我们可以创建非常紧凑、强大的早期逻辑检查。在示例 `(e)` 中，敌人根据攻击类型和玩家的力量接收攻击。

+   `(f)`: 这是一个将常量的创建与关键字 `let` 结合起来，并执行 `if` 语句的简短版本。在 Swift 中，我们使用问号 `?` 和冒号 `:` 来缩短 `if` 语句。简短 `if` 语句的格式为：`bool ? trueResult : falseResult`。如果 `isPoweredUp` 为 `true`，则 `playerPower` 等于 `basepower + 250`；如果为 `false`，则它是 `basepower + 50`。

### 对于循环

在处理集合之前，我们提到了 for-in 循环。这里再次是 Swift 中的 for-in 循环，它将遍历一个集合对象：

```swift
for itemName in inventory.values {
    print("Item name: \(itemName)")
}
```

对于一些习惯于使用旧方式使用 for 循环的程序员，不用担心，Swift 允许我们以 C 风格编写 for 循环，这是我们大多数人可能习惯的：

```swift
for var index = 0; index < 3; ++index {
    print("index is \(index)")  
}
```

这里是使用 for 循环而不使用索引变量的另一种方法，用下划线字符 `_` 标记，但当然使用 `Range<Int>` 对象类型来确定 for 循环迭代的次数：

```swift
let limit = 10
var someNumber = 1
for _ in 1...limit {
    someNumber *= 2
}
```

注意`1`和`limit`之间的`…`。这意味着这个 for-in 循环将从 1 迭代到 10。如果我们想让它从`0`迭代到`limit-1`（类似于在数组索引的范围内迭代），我们可以用`0..<limit`代替，其中`limit`等于数组的`.count`属性。

### do-while 循环

在编程中，另一个非常常见的迭代循环是 do-while 循环。很多时候我们可以只利用这个逻辑的 while 部分，所以让我们来看看我们可能如何和为什么使用 while 循环：

```swift
let score = player.score
var scoreCountNum = 0
while scoreCountNum < score {
    HUD.scoreText = String(scoreCountNum)
    scoreCountNum = scoreCountNum * 2
}
```

在游戏开发中，while 循环的一个用途（尽管在游戏应用中执行方式不同，但这可以适应每帧迭代一次）是显示玩家分数从 0 增加到玩家达到的分数——这是许多游戏在关卡结束时常见的审美。这个 while 循环将迭代，直到达到玩家的分数，并在 HUD 对象上显示直到那个分数的中间值。

do-while 循环实际上与 while 循环相同，只是额外的一个限制是至少迭代一次代码块。结束阶段的分数计数示例也可以说明为什么我们需要这样的循环。例如，让我们想象玩家在关卡结束时表现真的很糟糕，没有得到任何分数。在给出的 while 循环中，零分不会让我们进入 while 循环中的代码块，因为它不满足`scoreCountNum < score`的逻辑检查。在 while 循环中，我们还有显示分数文本的代码。虽然这可能对玩家来说很尴尬，但我们仍然想要计数到分数，更重要的是，仍然显示分数。以下是使用 do-while 循环完成的相同代码：

```swift
let score = player.score
var scoreCountNum = 0
do {
    HUD.scoreText = String(scoreCountNum)
    scoreCountNum = scoreCountNum * 2
} while scoreCountNum < score
```

```swift
GameCenter achievement (in this case, a 6x combo) based on the number of times the combo was achieved by the player. Don't worry too much about the GameCenter code (used with the GCHelper singleton object); that's something we will go over in future chapters when we make games in SpriteKit and SceneKit.
```

```swift
switch (comboX6_counter) {

            case 2:
                GCHelper.sharedInstance.reportAchievementIdentifier("Piki_ComboX6", percent: 25)
                break

            case 5:
                GCHelper.sharedInstance.reportAchievementIdentifier("Piki_ComboX6", percent: 50)
                break

            case 10:
                GCHelper.sharedInstance.reportAchievementIdentifier("Piki_ComboX6", percent: 100)

            default:
                break

        }
```

这里的 switch 语句使用了一个变量来计数玩家击中了 6X 连击的次数，即`comboX6_counter`，并根据`comboX6_counter`的值执行不同的任务。例如，当玩家完成了两次 6X 连击时，Piki_ComboX6 成就将完成 25%。当计数器达到 10 时，玩家将获得成就（当达到 100%时）。关键字`break`的作用是告诉循环在那个点退出；否则，下一个 case 块将继续迭代。有时，这可能符合游戏逻辑的需求，但请注意，Swift，像许多其他语言一样，如果没有`break`，将继续通过 switch 语句。关键字`default`是一个通用的块，当 switch 语句检查的项的值不是各种情况时被调用。它可以被认为是`else{}`块的等价物，而所有的情况都类似于`else if(){}`。然而，不同之处在于 Swift 要求处理 switch 的所有情况。因此，尽管我们可以用一个没有`else`的`if`来满足，但我们必须为 switch 语句提供一个默认情况。再次强调，这是为了在编码过程的早期保持 Swift 代码的安全和整洁。

## 函数和类

到目前为止，我们还没有讨论 Swift 或任何面向对象编程语言可能最重要的方面——语言如何处理对象上的函数以及如何组织这些对象、对象属性和函数，并执行各种面向对象设计概念，如多态和继承（使用类、Structs、枚举、协议和其他数据结构）。关于 Swift 如何利用这些概念，有更多内容可以讨论，但超出了本章的范围。但在本书的整个过程中，特别是在我们深入研究如何使用苹果的游戏中心 SpriteKit 和 SceneKit 框架时，我们将更详细地探讨这些主题。

### Functions

在 Objective-C 中，函数的写法如下：

```swift
-(int) getPlayerHealth() {
    return player.health;
}
```

这是一个简单的函数，它返回玩家的生命值作为一个整数——在 Objective-C 中的 `Int` 等价物。

函数/方法的结构在 Objective-C 中如下所示：

```swift
- (return_type) method_name:( argumentType1 )argumentName1
joiningArgument2:( argumentType2 )argumentName2 ...
joiningArgumentN:( argumentTypeN )argumentNameN
{
  function body
}
```

下面是相同的函数在 Swift 中的样子：

```swift
func getPlayerHealth() -> Int {
    return player.health
}
//How we'd use the function
var currentHealth : Int = 0
currentHealth = getPlayerHealth()
```

这就是 Swift 中函数的结构：

```swift
func function_name(argumentName1 : argumentType1, argumentName2 : argumentType2, argumentNameN : argumentTypeN) -> return_type
{
  function body
}
```

注意我们如何使用关键字 `func` 来创建一个函数，以及参数/参数名称的顺序是先类型后名称，由冒号（`:`）分隔，并放在括号内。

下面是一个典型的 Swift 中 void 函数的例子。void 类型的函数是一个不返回值的函数。

```swift
//with a Player type as a parameter
func displayPlayerName (player:Player){
     print(player.name)
}

//without any parameters; using a class property
func displayPlayerName(){
     print(currentPlayer.name)
}
```

在 void 函数中，不需要写 `->returnType`，即使没有参数，我们也要在函数名称的末尾放入 `()` 括号。

# Tuples

Swift 的一个相当强大的特性是函数返回类型（以及常量/变量）可以包含值的组合到一个单一值中。这些组合被称为 **元组**。下面是一个未命名的元组示例：

```swift
let http503Error = (503, "Service Unavailable")
```

下面是一个从苹果的 Swift 文档中直接使用的作为函数返回类型的元组示例。注意它使用了我们迄今为止学到的大部分内容：

```swift
func minMax(array: [Int]) -> (min: Int, max: Int) {
    var currentMin = array[0]
    var currentMax = array[0]
    for value in array[1..<array.count] {
        if value < currentMin {
            currentMin = value
        } else if value > currentMax {
            currentMax = value
        }
    }
    return (currentMin, currentMax)
}
Excerpt From: Apple Inc. "IOS Developer Library". https://developer.apple.com/library/ios/documentation/Swift/Conceptual/Swift_Programming_Language/Functions.html#//apple_ref/doc/uid/TP40014097-CH10-ID164
```

## Classes

在面向对象编程（OOP）中，类构成了对象的基本框架，包括其功能以及与其他类、对象和各种数据结构（如协议、Structs、扩展、泛型和枚举）的交互。在接下来的章节中，当我们开始构建我们的游戏时，我们将更深入地探讨所有这些概念，但就目前而言，让我们先了解类的基本知识以及它们在 Swift 中与 Objective-C 和其他语言的不同之处。

下面是 Swift 中类的基本结构：

```swift
//(a)
Global-project wide properties/variables
//(b)
class className : parentClassName, protocolName…protocolnName
{
//(c)
  class scope properties
//(d)
initializers (init(), convenience, required, etc)

//(e)
  func function_name1(argumentName1 : argumentType1, argumentName2 :    argumentType2, argumentNameN : argumentTypeN) -> return_type
 {
   function-scope variables and body
 }
                         .
                         .
                         .
  func function_nameN(argumentName1 : argumentType1, argumentName2 :    argumentType2, argumentNameN : argumentTypeN) -> return_type
 {
   function-scope variables and body
 }
//(f)
deinit()

} // end of the class
//(g)
global-project wide properties/variables (alternative position)
```

Swift 的类结构在某种程度上类似于我们在 C# 和 Java 中看到的，而不是 Objective-C 的两个文件（`.h`/`header`，`.m`/`.mm`/实现）的设置：

+   `(a)`: 我们可以在类声明之外拥有属性（如变量、常量、Structs 和枚举），这将使它们具有全局作用域，即在整个项目/游戏/应用程序中可访问。

+   `(b)`: 这是我们命名 `.swift` 文件时所表示的实际类。再次强调，这与 Objective-C 为单个类提供的 `classname.h - classname.m/.mm` 双文件设置不同。一个类可以是另一个类的子类。在 Swift 中，我们不必声明父类或基类。我们创建的类可以是它们自己的基类。我们可以通过从 NSObject 继承来创建类似于 Objective-C 类的类。这样做的好处是获得 Objective-C 运行时元数据和功能，但我们会因为额外的“负担”而牺牲性能。我们可以在 `parentClass` 的同一位置或 `parentClass` 冒号 `:` 之后声明这个类将遵守哪些协议。我们将在本书的后面部分更详细地讨论协议，但可以简单地将它们视为确保你的类使用与协议规定相同的函数。

+   `(c)`: 这些是我们将放置变量、常量、结构体、枚举和对象的地方，这些对象对于在类的范围内使用是相关的。

+   `(d)`: 初始化器是我们用来在部分 `(c)` 中设置属性的特殊函数，当其他类和数据结构通过 `className(initializer parameters)` 使用该类的实例时。在我们构建游戏时，我们将在下一章中更详细地讨论初始化器。它们不必放在类的顶部，但将它们放在那里是一个好的实践。

+   `(e)`: 这些是声明和开发你的类函数的地方。我们可以有被称为类函数的函数。这些函数用关键字 `class func` 标识。简而言之，类函数是类整体的一部分，而不是类的实例。将它们放在下一个更常见的函数类型（即公共函数）之上是一个好的实践，这些公共函数可以通过点操作符（即 `className.function(parameters)`）被其他类和属性访问。与 C# 和 Java 一样，我们可以使用 `private func` 关键字创建私有函数，这些函数只能被类的自身函数和属性访问。

+   `(f)`: `deinit()` 函数是一个特殊的可选函数，用于处理我们如何通过内存管理和消除所谓的内存泄漏来清理由我们的类分配的数据。Apple 的 **ARC** （**自动引用计数**）处理了大部分工作，但有时我们不得不在各个属性前使用诸如 weak 和 unowned 这样的关键字，以确保它们在使用后不会继续存在。

    这是一个相当复杂的话题，但值得深入研究以避免游戏中的内存泄漏。自动引用计数（ARC）确实负责处理大部分工作，但你的游戏中可能存在一些可能长时间存在的对象。强烈建议阅读苹果公司关于此主题的官方文档，因为 iOS 中的内存管理始终处于不断发展之中。你可以在 [`developer.apple.com/library/ios/documentation/Swift/Conceptual/Swift_Programming_Language/AutomaticReferenceCounting.html`](https://developer.apple.com/library/ios/documentation/Swift/Conceptual/Swift_Programming_Language/AutomaticReferenceCounting.html) 查看关于 ARC 和 Swift 中内存管理的完整文档。

+   `(g)`: 如果我们愿意，我们也可以在 `.swift` 文件的底部，在类声明之后，拥有全局属性。苹果自己的游戏示例，冒险 ([`developer.apple.com/library/ios/documentation/Swift/Conceptual/Swift_Programming_Language/Properties.html`](https://developer.apple.com/library/ios/documentation/Swift/Conceptual/Swift_Programming_Language/Properties.html))，将全局属性放在了这个位置。

# 摘要

关于 Swift 编程语言的内容远不止这里所能涵盖的。在本书的整个过程中，我们将根据即将到来的游戏编程需求，穿插一些关于 Swift 的额外信息和细微差别。

如果你希望更精通 Swift 编程语言，苹果实际上提供了一个名为 **游乐场** 的出色工具。

游乐场是在 2014 年 6 月的 *WWDC14* 上与 Swift 编程语言一起引入的，它允许我们在不创建项目、构建和运行的情况下测试各种代码输出和语法，在很多情况下我们只需要调整几个变量和函数循环迭代。

在官方 Swift 开发者页面 ([`developer.apple.com/swift/resources/`](https://developer.apple.com/swift/resources/)) 上有许多资源可以查看。

下面是两个非常推荐的游乐场：

+   **导游游乐场** ([`developer.apple.com/library/ios/documentation/Swift/Conceptual/Swift_Programming_Language/GuidedTour.playground.zip`](https://developer.apple.com/library/ios/documentation/Swift/Conceptual/Swift_Programming_Language/GuidedTour.playground.zip)): 这个游乐场涵盖了本章中提到的许多主题以及更多内容，从 **Hello World** 到 **泛型**。

+   **气球游乐场** ([`developer.apple.com/swift/blog/downloads/Balloons.zip`](https://developer.apple.com/swift/blog/downloads/Balloons.zip)): 气球游乐场是 *WWDC14* 的关键演示游乐场，展示了游乐场提供的许多功能，尤其是制作和测试游戏。

有时，学习编程语言的最佳方式是测试实时代码，这正是游乐场允许我们做的事情。

除了在我们的游戏中测试代码片段之外，iOS 9 还允许我们规划和构建我们的游戏，这是下一章的主题。
