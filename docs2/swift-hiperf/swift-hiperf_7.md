# 第七章. 懒惰的重要性

提高应用程序性能的另一种方法是推迟代码的执行，直到需要结果。这听起来非常合理；我们运行的代码越少，所需的时间就越少。这种设计模式通常被称为**懒惰**。有很多事情可以是懒惰的，也有很多不同的方法可以推迟代码执行。在本章中，我们将涵盖以下主题：

+   懒惰的心态

+   懒加载

+   懒惰集合和评估

# 懒惰的心态

首先，理解懒惰模式的工作原理、它如何提高应用程序的性能以及何时使用它非常重要。同样重要的是不要滥用它，因为这会使代码更加复杂且难以阅读。此外，跟踪执行流程也会变得困难。滥用它还会降低应用程序的整体性能。

懒惰模式的一般思想是在有人请求该指令的结果之前，推迟对该指令的评估。

通常，代码是按指令顺序执行的，从文件的顶部或函数的开始处开始。如今，我们的应用程序更加复杂，由许多文件、窗口、库、组件和层组成，但它们仍然以相同的方式执行代码。由于我们的系统越来越大，使它们变得懒惰非常重要，这样我们就不必在启动应用程序时执行所有工作。让我们学习一些使代码变得懒惰的技术。

## 分离

非常重要的是将代码分离成具有**单一职责**模式的组件。组件应该只做一件事，并且要做好。让我们看看这个简单的例子，以了解这如何提高性能：

```swift
struct Person {
  let name: String
  let age: Int
}

func analyze(people: [Person]) {
  let names = people.map { $0.name }
  let last = names.maxElement()

  let alphabetOrder = names.sort { $0 > $1 }
  let lengthOrder = names.sort { $0.characters.count < $1.characters.count }
  let longestName = lengthOrder.last

  print(last, alphabetOrder, lengthOrder, longestName)

  let age = people.map { $0.age }
  let youngest = age.minElement()
  let oldest = age.maxElement()
  let average = age.reduce(0, combine: +) / age.count

  print(youngest, oldest, average)
}

let people = [Person(name: "Sam", age: 3),
  Person(name: "Lisa", age: 68),
  Person(name: "Jesse", age: 35)
]

people + EnglandPopulation()
analyze(people)
```

这段代码的问题在于`analyze`函数做了两件事。正因为如此，内存使用峰值是原来的两倍。当我们完成对名称的分析后，用于它的内存不会立即释放，而是保留到我们从函数返回。如果我们试图分析英格兰的全部人口，这将需要相当多的内存。通过将它们分成单独的函数，我们可以改善内存使用：

```swift
func analyze(people: [Person]) {

  let names = people.map { $0.name }
  analyzeNames(names)

  let age = people.map { $0.age }
  analyzeAge(age)
}

func analyzeNames(names: [String]) {
  ...
}

func analyzeAge(age: [Int]) {
  ...
}
```

你应该经常将代码分离技术应用于不同的组件，例如。创建简单的模型，并仅使用当前视图所需的数据。将代码拆分为更小的组件是有用的，但你不应将其拆分得过于细小。

## 按需工作

懒加载代码的主要思想是仅在需要时才执行工作。这样，你可以在没有人请求结果的情况下延迟代码执行或完全删除它。当你规划应用程序的架构时，试着考虑这些问题：这个资源或数据在应用程序中何时会被使用？它现在必须实例化吗？还是可以等待？这份数据会被使用多频繁？如果经常使用，那么你可能应该缓存它；如果不经常使用，那么你可能应该懒加载它。数据是否很重？你需要在之后清理它吗？提出所有这些问题有助于你使用*按需工作*的方法构建更好的应用程序架构。

## 近似结果

另一个想法是将懒加载和异步执行结合起来。如果你想执行一个异步任务，因为它需要很长时间，但你又想立即得到结果，你可以立即返回一个近似结果并继续执行。例如，核心数据异步获取实现了这个模式。它返回你将要获取的项目的大致数量。

# 懒加载

懒加载模式允许你在尝试使用对象之前延迟创建对象。这种模式可以在任何编程语言中实现。在 Objective-C 中，我们使用了属性的获取器并检查它是否已初始化。Swift 为语言添加了对懒加载的支持，这使得应用此模式变得更加容易。有许多东西可以被懒加载，我们将在本章中介绍它们。

## 全局常量和变量

在 Swift 中，全局变量和常量始终是懒加载的。这意味着每个全局变量仅在第一次访问它时才被初始化。作为一个测试，让我们创建一个新的 `Person.swift` 文件并将此代码添加到其中：

```swift
struct Person {
  let name: String
  let age: Int

  init(name: String, age: Int) {
    self.name = name
    self.age = age
    print("\(name) Created")
  }
}

let Jon = Person(name: "Jon", age: 20)
let Sam = Person(name: "Sam", age: 28)
```

此文件包含两个全局常量：`Jon` 和 `Sam`。我们还向 `Person` 结构体的 `init` 方法中添加了一个日志语句，这样我们就可以看到何时创建了这个人物。现在让我们尝试访问其中一个全局常量并查看控制台输出：

```swift
print("Start!")
print("Age: \(Jon.age)")
//print("Age: \(Sam.age)")

Console Output:

Start!
Jon Created
Age: 20
```

如你所见，只有 `Jon` 全局常量被创建，并且是在我们尝试访问其年龄属性时创建的。是的，Swift 中的全局常量和变量具有强大的功能，但你几乎永远不应该使用全局常量！有更好的方法来做这件事。

全局变量甚至更脆弱，因为任何人都可以在整个应用程序中更改它们。

## 类型属性

结构体和类都可以有一个 `type` 属性。`type` 属性属于类型本身，而不是该类型的实例。无论你创建多少个该类型的实例，`type` 属性的副本都只有一个。它们的行为类似于全局变量和常量，但它们具有该类型的命名空间作用域。此外，类型属性可以声明为私有，并从应用程序的其余部分隐藏。

我们可以通过将全局常量移动到 `Person` 结构定义中来非常容易地改进我们的代码：

```swift
struct Person {
  let name: String
  let age: Int

  static let Jon = Person(name: "Jon", age: 20)
  static let Sam = Person(name: "Sam", age: 25)

  ...
}
```

要访问`type`属性，我们需要在其前加上它声明的类型名称；要访问`Jon`类型常量，我们需要写`Person.Jon`。类与结构体在类型属性方面具有相同的语法和功能：

```swift
print("Age: \(Person.Jon.age)")
print("Age: \(Person.Sam.age)")
```

## 懒加载属性

还有懒加载实例属性的方法。让我们扩展我们的`Person`类型并为其添加一个`HealthData`结构体：

```swift
struct HealthData {
  init() {
    print("HealthData Created") 
  }
}

struct Person {
  let name: String
  let age: Int

  var healthData = HealthData()
}
```

现在，每次我们创建一个新的个人，它都会创建一个`healthData`实例。在我们的例子中，`HealthData`可能是一个连接到数据库、获取健康数据并执行大量工作的重对象。我们不需要在创建`Person`的同时创建`HealthData`结构体；相反，我们希望在需要时才创建`HealthData`结构体。

要使一个属性表现得是懒加载的，你只需要在其声明中添加`lazy`属性：

```swift
  lazy var healthData = HealthData()
```

`lazy`实例属性必须声明为变量而不是常量。

### 提示

非常重要不要滥用懒加载实例属性。访问懒加载属性会增加一点性能开销，用于检查它是否已初始化。

因此，分析系统哪些部分需要同时创建非常重要，因为它们会同时使用，而有些可能根本不会使用；因此，我们将它们设置为懒加载以延迟它们的创建。

当你访问懒加载属性时，它实际上会修改一个值，因此实例必须声明为变量。如果你使用具有懒加载存储属性的课程，它可以声明为常量。这是因为它是一个常量引用，更改应用于值：

```swift
let ola = Person(name: "Ola", age: 27)
let health = ola.healthData // Error! It's mutating a value

var bobby = Person(name: "Bobby ", age: 5)
let bobbyHealth = bobby.healthData // Works fine

let someClass = SomeClass()
someClass.healthData // Works fine because class is a reference type
```

关于懒加载属性的一个重要注意事项是，它们只初始化一次。如果你创建了一个懒加载的可选属性，稍后你想将其设置为`nil`并再次初始化，你需要手动完成。比如说`healthData`是一个非常重的实例，我们希望在不需要时清除它：

```swift
struct Person {
  ...
  lazy var healthData: HealthData? = HealthData()

  mutating func clearHealthData() {
    healthData = nil
  }
}

var ola = Person(name: "Ola", age: 27)
var health = ola.healthData //Get lazy loaded here
ola.clearHealthData() 
health = ola.healthData // nil, nothing happens here.
```

正如我们所说，懒加载的存储属性只初始化一次。所以，下次我们访问它，在我们清除它之后，就没有额外的初始化代码在运行。如果我们真的需要这种懒加载缓存，那么我们需要自己实现它。这并不难。我们需要一个存储属性和一个计算属性，如下所示：

```swift
struct Person {
  ...
  private var _healthData: HealthData?

  mutating func clearHealthData() {
    _healthData = nil
  }

  var healthData: HealthData {
    mutating get {
      _healthData = _healthData ?? HealthData()
      return _healthData!
    }
  }
}
```

我们在这里必须创建一个可变获取器，因为`var`属性不能声明为`mutating`。现在，如果我们再次运行之前的示例，我们会看到每次清理后都会创建`HealthData`：

```swift
var ola = Person(name: "Ola", age: 27)
var health = ola.healthData //Get lazy loaded here
ola.clearHealthData() 
health = ola.healthData // HealthData created again
```

## 计算属性

延迟属性初始化的另一种方式是使用计算属性。正如其名所示，计算属性每次访问时都会被计算。重要的是要记住，这样的属性每次都会被计算，因为这可能会对性能产生负面影响，或者如果你对其执行任何副作用。

计算属性的最好用途是当你想要提供一个只读属性，该属性使用内部数据进行计算时。一个很好的例子就是一个人的全名：

```swift
struct Person {
  let name: String
  let lastName: String
  let age: Int

  var fullName: String {
    print("calculating fullName")
    return "\(name) \(lastName)"
  }
}

var jack = Person(name: "Jack", lastName: "Samuel", age: 21)
print(jack.fullName)
print(jack.fullName)
```

在执行任何副作用或突变操作时，对计算属性进行操作时，我们必须非常注意。突变操作需要为结构显式指定，但对于类来说，这样做也是合法的。

# 懒加载集合和评估

另一个我们可以懒加载执行操作的地方是集合和序列。我们在其中存储了许多元素，有时执行如`filter`或`map`这样的操作会花费很多时间，可能也是不必要的。

在我们深入细节之前，让我们先看看一个小例子，看看为什么懒加载集合是如此有用。我们有一个集合。我们想要对其执行如映射这样的操作，并从结果中获取一个或几个元素。以下是使用数组实现它的方法：

```swift
let numbers = Array(1...1_000_000) 
let doubledNumbers = numbers.map { $0 * 2 }
doubledNumbers.last
```

当我们在`numbers`数组上调用`map`方法时，它会将操作应用于数组中的每个元素，并返回新的映射数组。因此，我们得到了一个新的`doubledNumbers`数组。我们的`map`闭包`{ $0 * 2 }`会根据数组中的元素数量被调用，在我们的例子中是 1,000,000 次。但我们需要的是数组中的最后一个元素。而不是映射每个元素，我们只想映射最后一个。在这种情况下，使用懒加载集合会更好：

```swift
let numbers = Array(1...1_000_000) 
let lazyNumbers = numbers.lazy
let doubledNumbers = lazyNumbers.map { $0 * 2 }
doubledNumbers.last
```

这里的唯一区别是我们添加了一个`lazy`方法调用来从数组创建`LazyCollection`：

```swift
public var lazy: LazyCollection<Self> { get }
```

当我们在懒加载集合上调用`map`方法时，它不会立即对每个元素进行映射，而是延迟执行并返回另一个`LazyCollection`——`LazyMapCollection`。下次当我们要求`doubledNumbers`获取最后一个数字时，它会对最后一个数字进行映射并返回给我们。因此，我们只调用了一次我们的`map`，这正是我们所需要的。

所有的`LazyCollections`和`LazySequences`都按照相同的原理工作；它们在需要时才执行操作，当你从其中拉取一个元素时。当你调用任何应该对序列执行操作的函数，如`map`或`filter`时，它不会立即执行。相反，`LazySequence`保存需要执行的操作并返回一个新的`LazySequence`。只有当你从其中拉取数据时，`LazySequence`才会执行那个操作，比如当你请求最后一个元素时。

## 序列和集合

为了更好地理解懒加载，了解序列和集合之间的区别是有用的。我们可以将懒加载操作应用于它们，但集合允许我们做更多的事情，我们将会看到原因。

### 序列

序列表示为可遍历的元素集合。我们对序列的主要操作是从第一个元素开始向前迭代其元素。为了做到这一点，序列使用了一个 `GeneratorType` 协议。`GeneratorType` 协议有一个 `next` 方法，它返回下一个元素，如果没有元素则返回 nil。这是 `GeneratorType` 上唯一可用的方法：

```swift
mutating func next() -> Self.Element?
```

### 注意

`SequenceType` 和 `GeneratorType` 协议非常简单。每个协议都只需要实现一个方法：

```swift
protocol SequenceType {
  func generate() -> Self.Generator
}
protocol GeneratorType {
  mutating func next() -> Self.Element?
}
```

这里是 SequenceType 实例可以执行的最简单和最基本操作，即对其元素的迭代：

```swift
let seq = AnySequence(1...10)
for i in seq {
  i
}
```

### 小贴士

如果你希望你的自定义类型能在 `for…in` 循环中使用，你需要为你的类型实现一个 `SequenceType` 协议。

我们迭代序列的另一种方式是手动从生成器获取下一个元素：

```swift
let gen = seq.generate()
while let num = gen.next() {
  num
}
```

Swift 标准库使用协议扩展来为 `SequenceType` 提供额外的功能。在 Swift 2.0 中，`SequenceType` 变得非常强大，拥有许多方法，如 `map` 和 `filter`、`sort`、`equal` 以及许多其他方法。

### 集合

另一方面，集合表示一组项目，其中每个项目都可以通过其索引访问。集合的最简单例子是一个数组。集合类型也实现了 `SequenceType` 协议，因此你可以在需要序列的任何地方使用集合。

除了 `SequenceType` 协议外，`CollectionType` 还需要实现三个额外的方法：

```swift
subscript (position: Self.Index) -> Self._Element { get }
var startIndex: Self.Index { get }
var endIndex: Self.Index { get }
```

`CollectionType` 通过下标方法为我们提供了对元素的随机访问。因为集合知道第一个和最后一个索引，它还可以计算大小，并从开始和结束两个方向迭代其元素。这给了我们更多的能力。

### 实现我们自己的序列和集合

作为了解序列和集合的总结，让我们实现我们自己的序列和集合类型。我们还可以添加一个 `print` 语句，这样我们就可以在控制台看到它实际执行工作的情况。这对于检查惰性集合的行为非常有用。让我们从序列开始：

```swift
struct InfiniteNums: SequenceType {

  func generate() -> AnyGenerator<Int> {
    var num = 0

    return anyGenerator {
      print("gen \(num)")
      return num++
    }
  }
}
```

这里是我们简单的无限数列。我们的生成器只是返回下一个整数。现在让我们创建一个自定义集合。它将比序列复杂一些：

```swift
struct Collection10: CollectionType {
  let data = Array(1...10)

  var startIndex: Int {
    return data.startIndex
  }

  var endIndex: Int {
    return data.endIndex
  }

  subscript (position: Int) -> Int {
    print("Pos \(position)")
    return data[position]
  }

  func generate() -> AnyGenerator<Int> {
    var index = 0

    return anyGenerator {
      print("Col index: \(index)")
      let next: Int? = index < self.endIndex ? self.data[index++] : nil
      return next
    }
  }
}
```

对于这个例子，我们创建了一个包含 10 个整数元素的不可变集合。它的实现仍然相当简单。因为我们的集合是封闭的，并且它有 `endIndex` 属性，生成器需要检查边界，当序列中没有更多元素时，它返回 `nil`。

你可以尝试使用这些新类型，并将它们与 Swift 标准库中的函数一起使用，这些函数接受序列或集合类型作为参数。现在让我们继续到最有趣的部分；让我们使用这些类型与一个惰性函数一起使用。

### 使用惰性

将集合或序列转换为懒版本非常简单。序列和集合都有一个 `lazy` 属性，它返回该集合或序列的懒加载版本：

```swift
public var lazy: LazyCollection<Self> { get }
public var lazy: LazySequence<Self> { get }
[1, 2, 3].lazy
AnySequence(1...10).lazy
```

### 使用懒序列

现在让我们用我们的无限数字序列和 `LazySequence` 的映射和过滤方法来玩玩：

```swift
let infNums = InfiniteNums()
let lazyNumbers = infNums.lazy

let oddNumbers = lazyNumbers.filter { $0 % 2 != 0 }
let doubled = lazyNumbers.map { $0 * 2 }
let mixed = lazyNumbers.filter { $0 % 4 != 0 }.map { $0  * 2 }

var gen = oddNumbers.generate()
var gen2 =  mixed.generate()

for _ in 0...10 {
  gen.next()
  gen2.next()
}
```

这里的代码相当简单。我们创建一个懒序列，并对其应用 `map` 和 `filter` 转换。然后，我们从其中提取前 10 个结果。因为集合是懒加载的，所以它只在开始从循环体中提取元素时开始执行映射和过滤，通过调用 `next` 方法。

因为 `LazySequence` 是一个 `SequenceType` 协议，所以你可以用它与任何接受 `SequenceType` 协议的函数一起使用，或者调用在该 `SequenceType` 中可用的任何方法。这种操作不会是懒加载的，并且会返回一个结果。如果我们尝试用我们的无限数字使用它们，可能会导致无限循环。这是因为我们的序列是无限的：

```swift
lazyNumbers.contains(3) // returns true, stops when found
// lazyNumbers.minElement() // Infinite loop
```

### 使用懒集合

与懒序列一起工作非常类似于与懒集合一起工作。`LazyCollection` 实现了一个 `CollectionType`，这意味着我们可以使用 `CollectionType` 中的所有方法与 `LazyCollection` 一起使用。

首先，让我们创建一个懒集合和一些额外的辅助函数来映射和过滤我们将要使用的集合：

```swift
let isOdd = { $0 % 2 != 0 }
let doubleElements = { $0 * 2 }

let col = Collection10()
let lazyCol = col.lazy
```

懒集合的 `map`、`reverse` 和 `filter` 方法有一些小的区别。它们返回不同的懒集合类型：`LazyMapCollection`、`LazyFilterCollection` 和 `ReverseRandomAccessCollection`：

```swift
lazyCol.map(doubleElements) //LazyMapCollection<Self.Elements, U>
lazyCol.reverse() //LazyCollection<ReverseRandomAccessCollection<Self.Elements>>
lazyCol.filter(isOdd)  //LazyFilterCollection<Self.Elements>
```

`LazyMapCollection` 和 `LazyFilterCollection` 类型都实现了 `LazyCollectionType`，并且它们有类似的方法。正因为如此，一些懒集合可能有一组不同的可用方法。

这些懒集合的一个有趣的行为差异是某些方法工作方式的不同。作为一个例子，让我们看看 `count` 和 `isEmpty` 方法。首先，让我们尝试用 `LazyMapCollection` 使用它们：

```swift
let lazyMap = lazyCol.map(doubleElements)
let count = lazyMap.count

lazyMap.isEmpty
lazyMap.reverse().isEmpty
```

调用 `count` 和 `isEmpty` 不会强制懒集合执行任何映射操作。因为映射不会改变源集合的元素数量，它可以使用底层集合数据——`startIndex` 和 `endIndex`——来计算这些方法的结果。

然而，在 `LazyFilterCollection` 上调用 `count` 和 `isEmpty` 确实需要懒集合执行一个过滤操作。对于 `isEmpty` 方法，它一旦找到元素就会停止；但对于 `count` 方法，它需要将过滤操作应用于每个元素。在这种情况下，调用 `count` 方法的行为与在常规集合上调用它相同：

```swift
lazyCol.filter(isOdd).isEmpty
lazyCol.filter(isOdd).count
```

值得注意的是，`map` 和 `reverse` 方法之间还有一个区别，那就是它们使用的下标方法和索引：

```swift
//Query elements
lazyCol.map(doubleElements)[3]

let revCol = lazyCol.reverse()
let ind = revCol.startIndex.advancedBy(2)
revCol[ind]

let revMapCol = lazyCol.reverse().map(doubleElements)
let index = revMapCol.startIndex.advancedBy(2)
revMapCol[index]
```

`LazyMapCollection`允许我们使用用于集合的原始索引；在我们的例子中，这是一个`Int`类型。`reverse`方法返回一个具有`ReverseRandomAccessIndex`类型索引的`ReverseRandomAccessCollection`。要使用其上的索引方法，我们需要使用`startIndex`或`endIndex`以及`advance`方法来移动索引到所需的位置。

这是对懒集合和序列的一般概述。然而，懒类型还有一个非常重要的特性需要我们介绍。

懒集合或序列在开始从其中拉取元素时执行操作，例如，元素的映射。这里的区别是，常规数组对每个元素只应用一次映射并立即返回结果。

懒集合每次从其中拉取元素时都会应用映射。如果您打算经常使用映射的结果，那么使用常规数组或添加某种缓存或记忆功能可能更好。让我们通过一个例子来看看为什么这可能是危险的：

```swift
let col = Array(0...10)
let lazyCol = col.lazy

var x = 10
let mapped = lazyCol.map { $0 + x++ }

for i in mapped {
  print(" \(i)", terminator:"") //10 12 14 16 18 20 22 24 26 28 30
}

print("")
for i in mapped {
  print(" \(i)", terminator:"") //21 23 25 27 29 31 33 35 37 39 41
}
```

这个例子展示了使用懒集合和在映射函数中的状态时存在的两个重要问题。当我们使用映射的懒集合时，我们期望结果在所有循环迭代中都是相同的，因为我们没有做任何改变。然而，实际上，它将是不同的，因为`map`函数被调用了两次，我们在映射函数中使用的状态已经改变。

我们可以通过从映射函数中移除状态来部分解决这个问题。现在，懒映射集合会产生相同的结果，但这并不能改变映射函数将在两个循环中执行的事实。如果我们把一个昂贵的操作放在映射函数内部，它会使我们的应用速度减半：

```swift
// No state
let mapped = lazyCol.map { $0 + $0 + 10 }
```

在结束懒集合的话题时，我想说，懒集合非常强大，但您应该像使用任何其他有助于提高性能的工具一样谨慎地使用它。

# 摘要

在本章中，我们介绍了两个可以提高应用程序性能的工具：懒加载和懒执行技术。首先，我们解释了为什么让代码表现出懒性很重要，何时应该这样做，以及如何实现。接下来，我们展示了 Swift 语言内置的懒加载功能以及如何使用它与全局变量、类型属性和懒存储属性一起使用。

本章的其余部分，我们介绍了如何使集合表现出懒性。我们使用了许多与懒集合相关的函数，并展示了集合与序列之间的区别。

在下一章和最后一章中，*发现所有底层的 Swift 力量*，我们将探讨一些更高级的工具，这些工具将帮助您分析 Swift 的力量，并对您在本书中学到的内容进行快速回顾。
