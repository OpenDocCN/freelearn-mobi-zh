# 第八章。哦，太好了！看看在 Foundation 框架中的新内容

去年，苹果公司做了一场关于在代码中使用值类型和面向协议的设计模式的益处的精彩演讲。今年，随着 Swift 3 的发布，苹果团队将值类型添加到了 Foundation 框架中。我们许多喜爱且广泛使用的类现在都有了值类型的对应版本。在本章中，我们将介绍如何使用 Foundation 值类型，并让你思考为什么值类型从一开始就没有包含在 Swift 中！

新增于 Foundation 且由 Swift 3 使之成为可能的新测量和单位 API [`developer.apple.com/reference/foundation/nsmeasurement`](https://developer.apple.com/reference/foundation/nsmeasurement) 解决了 Objective-C 和 Swift 开发者长期以来的痛点。苹果公司为我们的问题创造了一个极其周到的解决方案。我们将通过几个示例来强化这些概念，以便你在未来更好地应对测量挑战。

# 可变性和 Foundation 值类型

Swift 语言的一个关键概念是赋予开发者控制其对象可变性的能力。我们使用 `let` 来使一个值成为常量，使用 `var` 来使一个值成为变量。然而，某些类型，当从 Objective-C 导入时，并不提供易于使用的可变性功能。Swift 3 通过添加一组新的 Foundation 值类型来包裹引用类型，从而为开发者提供可变选项，旨在改变这一现状。实际上，这并不是什么新鲜事，因为 Foundation 已经在 Objective-C 和 Swift 中使用了许多值类型。Foundation 已经存在的类型包括原始类型、枚举、选项集和 C 结构类型，这些在 Swift 和 Objective-C 的早期版本中已经是值类型。

为了使从引用类型到值类型的转换成为可能，Swift 使用了 *copy-on-write* 技术来处理新值类型，其底层数据包含的不仅仅是简单数据。在 *copy-on-write* 中，值类型代表一个指向共享数据的指针，并且只有在有请求通过其引用之一更改数据时，才会复制数据。以下表格列出了添加到 Foundation 框架中的新值类型。

表 1：新的值类型：

| **值类型** | **类 Type** |
| --- | --- |
| `AffineTransform` | `NSAffineTransform` |
| `CharacterSet` | `NSCharacterSet`, `NSMutableCharacterSet` |
| `Data` | `NSData` |
| `Date` | `NSDate` |
| `DateComponents` | `NSDateComponents` |
| `DateInterval` (新) |  |
| `Decimal` | `NSDecimal` |
| `IndexPath` | `NSIndexPath` |
| `IndexSet` | `NSIndexSet`, `NSMutableIndexSet` |
| `Measurement` (新) |  |
| `Notification` | `NSNotification` |
| `PersonNameComponents` | `NSPersonNameComponents` |
| `URL` | `NSURL` |
| `URLComponents` | `NSURLComponents` |
| `URLRequest` | `NSURLRequest`, `NSMutableURLRequest` |
| `URLQueryItem` | `NSURLQueryItem` |
| `UUID` | `NSUUID` |

这些新的值类型工作方式与它们的对应类型相同，额外的好处是你可以以更透明的方式控制可变性。一些新的值类型被实现为结构体类型，并将公开一个或多个可变方法。好消息是新的 Foundation 值类型对现有的 Objective-C 代码没有影响。然而，它将对 Swift 侧产生影响。任何现有的 Swift 代码迁移到 Swift 3，并使用引用类型，将转向调用新的值类型。虽然迁移器会做很多繁重的工作，但你仍有机会进一步优化你的代码。这是因为迁移器采用了一种尽可能少做更改的策略。例如，迁移器不会尝试更改任何 Swift 子类以用于引用类型。

# 值类型与引用类型

作为复习，让我们快速了解一下值类型和引用类型之间的区别。值类型持有值，在赋值或作为参数传递给方法时复制内容。

```swift
let avg1: Double = 98.1 
var avg2 = avg1 
avg2 += 1.2   // -> 99.3 

```

在前面的例子中，`avg2` 复制了 `avg1` 的值，并且可以自由更改其值而不影响 `avg1` 的值。另一方面，引用类型默认共享内容。更改其中一个变量将改变每个引用共享的底层值。

```swift

let dateComponents1 = NSDateComponents() 
dateComponents1.setValue(10, forComponent: .Day) 
dateComponents1.day // => 10 
var dateComponents2 = dateComponents1 
dateComponents2.setValue(2, forComponent: .Day) 
dateComponents1.day // => 2 
dateComponents2.day // => 2 

```

在这个例子中，我们使用引用类型 `NSDateComponents` 创建 `dateComponents1` 并将日历单位设置为 10。然后我们创建另一个变量并将第一个值赋给它。由于两个变量共享其底层数据，修改任何一个变量都会改变两个值。

# 嵌套枚举

Foundation 框架利用了 Swift 3 的新嵌套枚举功能。这个功能允许迁移器将相关的 Objective-C 枚举导入到 Swift 中，作为一个公共类型。例如，`NSDateFormatterStyle` 和 `NSDateFormatterBehavior` 枚举被导入到 Swift 的 `DateFormatter` 类中作为嵌套枚举。让我们看看如何在 Swift 3 中实现嵌套枚举，从 Objective-C 枚举开始。

在 Objective-C 中：

```swift
typedef NS_ENUM(NSUInteger, NSDateFormatterStyle), 
{ 
   NSDateFormatterNoStyle     = kCFDateFormatterNoStyle, 
   NSDateFormatterShortStyle  = kCFDateFormatterShortStyle, 
   NSDateFormatterMediumStyle = kCFDateFormatterMediumStyle, 
   NSDateFormatterLongStyle   = kCFDateFormatterLongStyle, 
   NSDateFormatterFullStyle   = kCFDateFormatterFullStyle 
}; 

typedef NS_ENUM(NSUInteger, NSDateFormatterBehavior){ 
   NSDateFormatterBehaviorDefault = 0, 
   NSDateFormatterBehavior10_0    = 1_000, 
   NSDateFormatterBehavior10_4    = 1_040, 
}; 

```

在 Swift 2.2-更好但尚未完成：

```swift
enum NSDateFormatterStyle : UInt { 
    case NoStyle 
    case ShortStyle 
    case MediumStyle 
    case LongStyle 
    case FullStyle 
} 

enum NSDateFormatterBehavior : UInt { 
    case BehaviorDefault 
    case Behavior10_0 
    case Behavior10_4 
} 

```

在 Swift 3 中：

```swift
DateFormatter { 

    public enum Style : UInt { 
        case none 
        case short 
        case medium 
        case long 
        case full 
    } 

    public enum Behavior : UInt { 
        case `default` 
        case behavior10_4 
    } 
} 

```

在 Swift 3 中使用嵌套枚举比 Swift 2.2 有所改进，因为我们不再需要担心基于 C 的枚举污染全局命名空间。

# 强类型字符串枚举

Foundation 框架有很多基于字符串的常量。例如，`UIKit` 使用 `NSNotifications` 发布 iOS 应用生命周期的通知。

```swift
NSString *const UIApplicationDidEnterBackgroundNotification        
NSString *const UIApplicationWillEnterForegroundNotification      NSString *const UIApplicationDidFinishLaunchingNotification; 

```

对于 Foundation 来说，Objective-C 现在有了使用强类型字符串枚举的能力。这个新特性允许 Foundation 团队更新 Objective-C 中的枚举。我们之前列出的通知现在可以使用 `NSNotificationName` 类型，并将先前的常量转换为：

在 Objective-C 中：

```swift
typedef NSString *NSNotificationName NS_EXTENSIBLE_STRING_ENUM; 
NSNotificationName const UIApplicationDidEnterBackgroundNotification        
NSNotificationName const UIApplicationWillEnterForegroundNotification      NSNotificationName const UIApplicationDidFinishLaunchingNotification; 

```

在 Swift 中，`NSNotificationName` 类型作为 `NSNotification.Name` 的扩展导入。

```swift
extension NSNotification.Name { 
public static let UIApplicationDidEnterBackground: NSNotification.Name 
public static let UIApplicationWillEnterForeground: NSNotification.Name 
public static let UIApplicationDidFinishLaunching: NSNotification.Name 
} 

```

# 类属性

由于历史原因，Foundation 框架中包含许多属性状态与类型紧密关联的案例。因此，你会在 Objective-C 类型上找到许多类属性。对于 Swift 3 的发布，我们得到了对 Objective-C 的一个调整，允许我们使用新的语法来创建类属性。这些 Objective-C 中的类属性也将作为类属性导入 Swift 中。

在 Objective-C（旧方法）：

```swift
@interface NSCharacterSet  
+ (NSCharacterSet *)controlCharacterSet; 
+ (NSCharacterSet *)whitespaceCharacterSet; 
+ (NSCharacterSet *)whitespaceAndNewlineCharacterSet; 
@end 

```

在 Objective-C（新方法）：

```swift
@interface NSCharacterSet  
@property(class, readonly, strong) controlCharacterSet; 
@property(class, readonly, strong) whitespaceCharacterSet; 
@property(class, readonly, strong) whitespaceAndNewlineCharacterSet; 
@end 

```

在 Swift 2.2 中：

```swift
class NSCharacterSet{ 
 class func controlCharacters() -> NSCharacterSet 
 class func whitespaces() -> NSCharacterSet 
 class func whitespacesAndNewlines() -> NSCharacterSet 
} 

```

在 Swift 3 中：

```swift
class NSCharacterSet{ 
 class var controlCharacters: CharacterSet { get } 
 class var whitespaces: CharacterSet { get } 
 class var whitespacesAndNewlines: CharacterSet { get } 
} 

```

# 值类型的类型安全访问

Swift 3 的新特性，你可以使用 Foundation 值类型来在编译时进行类型安全检查，以消除使用基于引用的 Foundation 类型时在运行时无法发现的许多错误。让我们通过一个示例来演示 Swift 2.2 中的运行时检查。

在 Swift 2.2 中：

```swift

if let filePath = NSBundle.mainBundle().pathForResource("testFile", ofType: "txt"){ 
    let fileURL = NSURL.fileURLWithPath(filePath) 
    let keys = [NSURLCreationDateKey, NSURLPathKey, NSURLFileSizeKey,NSURLTagNamesKey] 
    var values = try fileURL.resourceValuesForKeys(keys) 
    if let creationDate = values[NSURLCreationDateKey]{ 
        print("creationDate: \(creationDate)") 
    } 

    values[NSURLTagNamesKey] = ["test", "sample", "playground"] 
    values[NSURLCreationDateKey] = "now"   // à creates an error 
    print(values[NSURLTagNamesKey]) 
    print(values[NSURLCreationDateKey]) 
    try fileURL.setResourceValues(values)  
} 

```

在我们的示例中，我创建了一个指向现有文件`testFile.txt`的引用。我想了解一些文件属性，这些属性可以通过将字符串数组传递给我的文件引用的`resourceValuesForKeys`方法来获取。我可以使用索引符号读取值，甚至更新值。此外，我可以用`setResourceValues`方法将新值写回文件。我们遇到的问题很微妙，直到我们执行这段代码才会显示出来。`NSURLCreationDateKey`期望其值是一个有效的`NSDate`。

然而，我传递了一个字符串值。在运行时，当我们尝试使用新的资源值更新我们的文件时，我们的代码块崩溃并出现错误。我们想要的是一种在编译时检查约束的方法。Swift 3 现在为我们提供了这样做的方法，Foundation 框架也已更新以反映这一新特性。让我们更新我们的示例，以反映使用类型安全访问我们的文件资源。

在 Swift 3 中：

```swift
if let filePath = Bundle.main.path(forResource: "testFile",  
ofType: "txt") { 
    var fileURL = URL(fileURLWithPath: filePath) 
    let keys: Set<URLResourceKey> = [.creationDateKey, .pathKey, 
                                     .fileSizeKey, .tagNamesKey] 
    let values = try fileURL.resourceValues(forKeys: keys)  
    if let creationDate = values.creationDate{ 
        print("creationDate: \(creationDate)") 
    } 

    var newvalues = values 
    newvalues.creationDate = "now" 
    //error: cannot assign value of type 'String' to type 'Date?' 
    //    newvalues.creationDate = "now" 
    //                             ^~~~~ 
} 

```

在我们的更新示例中，我们的资源类型现在是类型为`URLResourceKey`的强类型值。当我们从文件请求资源时，我们返回一个包含强类型属性的 struct。这两个变化使我们能够进行类型安全访问，并帮助我们编译时捕捉问题。这是一个相当好的特性，Foundation 团队也同意，这从他们更新的所有 API 中可以看出，这些 API 为我们提供了更好的类型约束。

# 测量和单位

随着 Swift 3 的发布，Foundation 团队发布了一个新的 API，使开发者处理测量变得更加容易。通过允许开发者从常见的内置单位（或创建自定义单位）创建测量，在单位之间进行转换，以及为显示格式化它们。我们将涵盖 Measurement API 的主要区域，以便你准备好在你的项目中使用它们。

## 测量

一个测量由一个单位和值组成。单位是一个包含单个属性以持有符号的类。我们稍后会回到单位。现在，让我们专注于你可以用测量做什么。

```swift
public struct Measurement<UnitType : Unit> : ReferenceConvertible, Comparable, Equatable { 

    public typealias ReferenceType = NSMeasurement 

    /// The unit component of the `Measurement`. 
    public let unit: UnitType 

    /// The value component of the `Measurement`. 
    public var value: Double 

    /// Create a `Measurement` given a specified value and unit. 
    public init(value: Double, unit: UnitType) 
} 

let initialAngle = Measurement(value: 30, unit: UnitAngle.degrees) 
let maxAngle = Measurement(value: 90, unit: UnitAngle.degrees) 
var currentAngle = initialAngle * 2.5  // 75.0 ° 

if currentAngle > maxAngle{ 
    print("Angle is greater than max supported angle") 
} 

```

通过测量，我们可以定义一个值及其单位类型，这可以用于转换为其他类型。我们还获得了验证，确保我们不会混合不兼容的测量值。将长度单位添加到存储体积的单位上并不一定有意义。如果没有测量 API，你将不得不编写自己的方法来确保适当的约束保持有效。

## 单位和维度

*Unit* 是一个具有符号属性的类。`Dimension` 是 `Unit` 的子类，用于表示单位类别。`Dimension` 包含一个基本单位，并允许通过 `UnitConverter` 类型在单位之间进行转换。`Dimension` 类型还包含了许多常见单位的单例，这些单位在处理测量时可能会遇到。

```swift
class Unit : NSObject, NSCopying, NSSecureCoding { 
    open var symbol: String { get } 
    public init(symbol: String) 
} 

class Dimension : Unit, NSSecureCoding {     
    @NSCopying open var converter: UnitConverter { get } 
    public init(symbol: String, converter: UnitConverter) 
    class func baseUnit() -> Self 
} 

```

下表列出了 Foundation 中可用的 `Dimension` 子类。

表 2，`Dimension` 子类：

| `UnitAcceleration` | `UnitAngle` | `UnitArea` |
| --- | --- | --- |
| `UnitConcentrationMass` | `UnitDispersion` | `UnitDuration` |
| `UnitElectricCharge` | `UnitElectricCurrent` | `UnitElectricPotentialDifference` |
| `UnitElectricResistance` | `UnitEnergy` | `UnitFrequency` |
| `UnitFuelEfficiency` | `UnitLength` | `UnitIlluminance` |
| `UnitMass` | `UnitPower` | `UnitPressure` |
| `UnitSpeed` | `UnitTemperature` | `UnitVolume` |

当使用内置的 `Dimension` 子类时，你可以免费获得基本的转换。例如，我创建了一个 `boxHeight` 测量值，单位为 `UnitLength.feet`，以及一个 `*smallBoxHeight*`，单位为 `UnitLength.inches`。然后，我将这两个测量值相加并存储了值。`totalHeight` 变量使用 `UnitLength` 的基本单位米来存储组合值。

```swift
let boxHeight = Measurement(value: 3.2, unit: UnitLength.feet) 
let smallBoxHeight = Measurement(value: 20, unit: UnitLength.inches)  
let totalHeight = boxHeight + smallBoxHeight 
print(totalHeight) // 1.48336 m  

```

我们还可以使用我们的比较操作，如 `==`、`>`、`<`，因为我们的测量值在转换之前会转换为基本单位。

```swift
if boxHeight > smallBoxHeight{ 
    print("boxHeight is still larger") 
} 

```

## 自定义单位

将来会有需要创建一个不符合预构建 `Dimension` 子类之一的单元类型的时候。为了创建自己的单位，你需要在现有的 `Dimension` 上创建一个单元类型，提供符号和单位转换函数。

```swift
let burgerSymbol = "\u{1F354}" //  
let burgers = UnitLength(symbol: burgerSymbol , converter: UnitConverterLinear(coefficient: 2)) 

```

你可以向任何 `Dimension` 子类添加新的单位。我们前面的例子在 `UnitLength` 上创建了一个新的 `burgers` 单位。如果你需要一个更多基于时间的单位，你可以在 `UnitDuration` 上创建一个单位。

```swift
let gittieSecond = UnitDuration(symbol: "gs", converter: UnitConverterLinear(coefficient: 0.5)) 

```

通过提供转换函数，你确保了你的单位可以在其单位和关联的 `Dimension` 的基本单位之间进行转换。

## 自定义维度类型

现在我们知道了如何为 `Dimension` 子类创建新的单位，接下来要问的问题是，你是否可以创建自定义的 `Dimension` 类型。正如你可能已经猜到的，创建一个新的 `Dimension` 子类也是可能的，并且很简单。

```swift
public class ToddlerMovement: Dimension{ 
    static let tummyCrawl = ToddlerMovement(symbol: "crawls", converter: UnitConverterLinear(coefficient: 0.25)) 
    static let allFours = ToddlerMovement(symbol: "allfours", converter: UnitConverterLinear(coefficient: 2.0)) 
    static let shakySteps = ToddlerMovement(symbol: "shaky_steps", converter: UnitConverterLinear(coefficient: 1)) 
} 

let playPenDistance = Measurement(value: 4, unit: ToddleMovement.tummyCrawl) 
let kitchenFloorDistance = Measurement(value: 20, unit: ToddleMovement.shakySteps) 
let almostThereDistance = Measurement(value: 10, unit: ToddleMovement.allFours) 

print(playPinDistance) // 4.0 crawls 
print(kitchenFloorDistance) // 20.0 shaky_steps 
print(almostThereDistance) // 10.0 allfours 

```

## 转换函数

在之前的 `Measurement` 示例中，我们使用了 `UnitConverterLinear` 函数作为我们的转换函数。如前所述，每次向 `Dimension` 添加新单位时，都需要包含一个转换函数。转换方法用于在 `Dimension` 基本单位值之间进行转换。`UnitConverterLinear` 转换器允许你在关系是线性的情况下（例如，*y = mx + b*）在单位之间进行转换。此类有两个初始化器：

1.  当没有偏移量时，提供一个仅包含斜率（系数或我们线性方程中的 *m*）。

1.  另一个允许你提供斜率和偏移量（b）。

```swift
class UnitConverterLinear : UnitConverter { 
   var coefficient: Double { get } 
   var constant: Double { get } 
   init(coefficient: Double) 
   init(coefficient: Double, constant: Double) 
} 

```

线性转换：

```swift
baseUnit = value * coefficient + constant 
value = (baseUnit - constant) / coefficient 

```

`UnitConverterLinear` 转换器非常有用，它将允许你通过仅提供两个单位值之间的比率来覆盖许多转换。然而，这个转换器可能不适合两个单位之间的所有关系。因此，你还可以子类化 `UnitConverter` 并在需要时创建一个满足你用例的转换公式。

```swift
class UnitConverter : NSObject { 
   open func baseUnitValue(fromValue value: Double) -> Double 
   open func value(fromBaseUnitValue baseUnitValue: Double) -> Double 
} 

```

如果你选择创建自己的自定义单位转换器，你需要重写 `baseUnitValue(fromValue value: Double)` 和 `value(fromBaseUnitValue baseUnitValue: Double)` 方法。这两个方法用于内部为你执行转换。

## 测量值格式化

处理格式化问题很棘手，你只需看看 `NSDateFormatter` 就能明白在处理日期时事情可能会多么复杂。幸运的是，Foundation 团队为我们添加了一个新的测量格式化器来承担所有繁重的工作。`MeasurementFormatter` 将处理我们的 `Measurements` 和 `Unit` 类型格式化，同时考虑到用户的地区设置。

```swift
class MeasurementFormatter : Formatter, NSSecureCoding { 
   var unitOptions: MeasurementFormatter.UnitOptions 
   var unitStyle: Formatter.UnitStyle 
   @NSCopying var locale: Locale! 
   @NSCopying var numberFormatter: NumberFormatter! 
   func string(from measurement: Measurement<Unit>) -> String 
   func string(from unit: Unit) -> String 
} 

```

`MeasurementFormatter` 上的 `UnitOptions` 属性是一个特殊的属性，我们需要花点时间来讨论。`UnitOptions` 属性是 `MeasurementFormatter` 上的一个结构体，并指示我们的格式化器默认使用用户地区设置的首选单位。

```swift
extension MeasurementFormatter { 
   public struct UnitOptions : OptionSet { 

   public init(rawValue: UInt) 

   public static var providedUnit:  
                           MeasurementFormatter.UnitOptions { get } 
   public static var naturalScale:  
                           MeasurementFormatter.UnitOptions { get } 
   public static var temperatureWithoutUnit: 
                           MeasurementFormatter.UnitOptions { get } 
    } 
} 

```

在我们的示例中，我们创建了一个自定义单位（`*gittieSecond*`）和一个使用此单位的测量。然后我们创建了一个 `MeasurementFormatter` 并调用 `string` 方法，传递我们的测量，以获取使用 `Dimension` 基本单位格式的测量。尽管我们的自定义单位有一个 `gs` 符号，但格式化器会将我们的测量转换为并使用 `seconds` 显示。

```swift
let formatter = MeasurementFormatter() 
let gittieSecond = UnitDuration(symbol: "gs", converter: UnitConverterLinear(coefficient: 0.5)) 
let getterThere = Measurement(value: 5, unit: gittieSecond) // 5.0 gs 
let formattedGS = formmatter.string(from: getterThere) 
print(formattedGS) // 2.5 sec 

```

要将格式化器的输出从使用基本单位更改为使用我们的 `gittieSecond` 单位，我们需要修改格式化器上的 `UnitOptions` 属性。在我的地区设置中，`UnitDuration` 的首选单位是 `seconds`。通过更改 `UnitOptions` 使用提供的 `referenceUnit`，我们确保我们的格式化器将使用与传递给格式化器的 `getterThere` 测量相同的单位。

```swift
formatter.unitOptions = .providedUnit 
formatter.string(from: getterThere) // 5 gs 

```

# 摘要

在本章中，我们讨论了 Foundation 框架的变化。我们从 Foundation 中可变性和新值类型的话题开始讨论。你了解到在代码中，值类型和引用类型都有其合适的位置。接下来，我们介绍了嵌套枚举和强类型字符串枚举。我们探讨了类属性和值类型的类型安全访问优势。最后，我们花费了大量时间讨论了今年在 Foundation 中引入的新测量 API。在下一章中，我们将讨论测试和调试。Swift 3 在测试和调试方面引入了多项变化，这些变化应该有助于提高你编写的代码质量。
