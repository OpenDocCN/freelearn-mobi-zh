# 第四章：Swift 核心变更将让你欲罢不能

许多库都受到了影响以完成这项工作，包括 Swift 标准库、所有 Cocoa 和 Cocoa Touch 框架、Core Graphics 和 Grand Central Dispatch。随着 Swift 3 的发布，我们可以期待一些减少语言与 Objective-C 链接尴尬性的变化，展现出更多的 *Swifty-ness*。Swift 团队引入了新的 API 指南，目的是赋予语言其自身的特色。结果是整个语言范围内的大规模重命名和重构工作。Swift 3 在与 Objective-C 和 C API 的交互方面经历了一次巨大的改头换面。Swift 团队的目标是让你的开发体验感觉更像 Swift，而不是直接将 Objective-C 倒入你的代码中。Swift 是它自己的语言，应该有自己的感觉，就像任何其他编程语言一样。然而，Swift 的早期版本受到了与 Objective-C 和 C API 交互需求的强烈影响。

在本章中，我们将快速概述编写良好 Swift API 的哲学。之后，我们将用剩余的章节来探讨 Swift 3 中对 Objective-C 功能的引用和使用以及从 Objective-C 和 C 导入代码到 Swift 3 的语言改进。Swift 3 的每一次语言变化都由一个或多个 Swift Evolution 提案所涵盖。当我们介绍一个新特性时，我还会提供记录该变更背后理由的提案编号。虽然了解新特性的实际理由对于理解如何实现其代码不是至关重要的，但我认为了解背后所付出的努力和有时发生的辩论是有趣的。Swift Evolution 仓库包含了大量关于已接受和被拒绝提案的信息。如果你是一个细心的观察者，你还会看到一些被接受用于 Swift 3 发布但未能及时在发布中实现提案。

# 大规模重命名

让我们从 Swift API 设计指南的提案开始。大规模重命名提案共同代表了一个非常大的任务，包括 SE-0005、SE-0006、SE-0086 和 SE-0088。实现 API 指南代表了 Swift 3 语言的最大变化。我无法在这本简短的书中涵盖所有导致大规模重命名提案的 API 变化。幸运的是，你不需要理解库中每行更改的内容就能在 Swift 3 中高效工作。你有两个极有价值的资源，只需付出很少的努力就能带来回报。第一个资源是 Swift 迁移工具，它可以将现有的 Swift 项目转换为最新的语法。当你使用 Swift 迁移器时，你可以将你的 Swift 2.2 项目转换为 Swift 3，并免费获得大多数更改。第二个极其宝贵的资源是 Swift API 指南，它旨在帮助你编写更“Swift”的代码。Swift API 指南基于以下原则，如 [`swift.org/documentation/api-design-guidelines/`](https://swift.org/documentation/api-design-guidelines/) 上引用：

+   **使用点的清晰度**：这是你的最重要的目标。例如方法和属性等实体只声明一次但被反复使用。设计 API 以使这些使用清晰简洁。在评估设计时，阅读声明通常是不够的，总是要检查使用案例以确保它在上下文中看起来清晰。

+   **清晰度比简洁性更重要**：虽然 Swift 代码可以很紧凑，但使用尽可能少的字符来实现最小化代码并不是目标。在 Swift 代码中，简洁性（如果存在）是强大类型系统和自然减少模板代码的特性的副作用。

+   **编写文档注释**：这是针对每个声明。通过编写文档获得的见解可能会对你的设计产生深远的影响，所以不要推迟。

有关采用这些指南的更多详细信息，请参阅 WWDC 2016 关于 Swift API 指南的讲座 [`developer.apple.com/videos/play/wwdc2016/403/`](https://developer.apple.com/videos/play/wwdc2016/403/)。

## 在 Swift 3 中引用 Objective-C 代码

使用 Swift 3，我们得到了一系列使使用 Objective-C 和 C API 更愉快的变化。我们将介绍这些重要变化，这些变化将使在 Swift 3 中编写代码更加高效。

## 引用属性获取器和设置器的 Objective-C 选择器 - SE-0064

在 Objective-C 中，我们可以使用一个称为 **选择器** 的类型来引用方法的名称。Swift 2.2 引入了 `#selector` 表达式，以消除提供字符串字面量作为选择器名称时的易错性。在 Swift 3 中，语言通过允许你引用获取器和设置器方法来构建在 `#selector` 表达式之上。这个特性允许我们引用对象的获取器和设置器属性。让我们通过一个例子来看看我们如何访问 `ClassRoom` 类上某个属性的设置器：

```swift
class ClassRoom: NSObject{ 
    var roomNum: String 

    init(roomNum: String){ 
        self.roomNum = roomNum 
    } 
} 

let classRoom = ClassRoom(roomNum: "1-D1") 
classRoom.perform(#selector(setter: ClassRoom.roomNum), with: "2-D3")
```

我们现在可以使用 `#selector(setter: ClassRoom.roomNum)` 或 `#selector(getter: ClassRoom.roomNum)` 来访问 `roomNum` 的设置器。一旦我们有了引用，我们就可以使用 Objective-C 中可用的任何 NSObject 执行方法。

### 注意

你可以在 [`github.com/apple/swift-evolution/blob/master/proposals/0064-property-selectors.md`](https://github.com/apple/swift-evolution/blob/master/proposals/0064-property-selectors.md) 阅读提案

### 引用 Objective-C key paths [SE-0062]

与选择器类似，在 Swift 2.2 中使用 Objective-C *keypaths* 需要我们使用字符串字面量。Swift 3 引入了 `#keyPath` 表达式，通过用可以在编译时检查的对象替换易错的字符串字面量来提高准确性。下面的例子展示了 Swift 2.2 中是如何处理 keypaths 的，以及 Swift 3 中引用 keypaths 的改进。

对于 Objective-C：

```swift
class Employee: NSObject{ 
    var firstName: String 
    var lastName: String 
    var boss: Employee? 

    init(firstName: String, lastName: String, boss: Employee? = nil){ 
        self.firstName = firstName 
        self.lastName = lastName 
        self.boss = boss 
    } 
} 

let bossMan = Employee(firstName: "Jack", lastName: "Spark") 
let rocko = Employee(firstName: "Rocky", lastName: "Jones", boss: bossMan) 

```

对于 Swift 2.2：

```swift
rocko.value(forKeyPath: "Employee.boss.firstName") 

```

对于 Swift 3：

```swift
#keyPath(Employee.boss) // => boss 
rocko.value(forKeyPath: #keyPath(Employee.boss.firstName)) // => Jack 

```

在我们上面的 Swift 3 示例中，我们使用了编译时检查的 `#keyPath` 表达式，这允许我们安全地访问值。

### 注意

你可以在 [`github.com/apple/swift-evolution/blob/master/proposals/0062-objc-keypaths.md`](https://github.com/apple/swift-evolution/blob/master/proposals/0062-objc-keypaths.md) 阅读提案

# 从 Objective-C 和 C API 导入到 Swift 3

如果你维护 Objective-C 或 C 库，并希望为 Swift 提供更干净的语法，本节是为你准备的！我们现在有能力提供更多控制，以确定如何从 Objective-C 和 C 导入常量、全局函数和泛型到 Swift。

## 将 Objective-C 常量作为 Swift 类型导入 [SE-0033]

在头文件中定义的全局常量，在 Swift 中以相同的全局作用域导入。在许多情况下，将相关的常量分组在一起会更有帮助。在 Swift 3 中，你现在可以使用 `NS_STRING_ENUM` 或 `NS_EXTENSIBLE_STRING_ENUM` 注释你的类型声明，以便将这些声明导入为公共类型的成员。

### 将其作为结构体导入

如果你希望你的常量作为结构体的成员导入，那么请将 `NS_EXTENSIBLE_STRING_ENUM` 添加到你的常量类型声明的末尾。为了更一致地导入到 Swift，请使用与类型相同的名称作为前缀。在我们的例子中，我们创建了一个 `MPPlatformIdentifier` 类型，我们将其用作前缀来命名我们的常量：`MPPlatformIdentifierIOS`、`MPPlatformIdentifierMacOS`。

对于 Objective-C：

```swift
typedef NSString * MPPlatformIdentifier NS_EXTENSIBLE_STRING_ENUM; 
MPPlatformIdentifier const MPPlatformIdentifierIOS; 
MPPlatformIdentifier const MPPlatformIdentifierMacOS; 

```

这将导入到 Swift 中作为：

```swift
struct MPPlatformIdentifier : RawRepresentable { 
    typealias RawValue = String 

    init(rawValue: RawValue) 
    var rawValue: RawValue { get } 

    static var IOS: MPPlatformIdentifier { get } 
    static var macOS: MPPlatformIdentifier { get } 
} 

```

### 以枚举方式导入

您的另一个选择是将常量导入为枚举类型。当您在类型末尾添加 `NS_STRING_ENUM` 时，您的常量将作为枚举导入。您使用此新类型定义的每个常量都将添加到 Swift 中的枚举中。

对于 Objective-C：

```swift
typedef NSString * MPVersionEnum NS_STRING_ENUM; 
MPVersionEnum const MPVersionEnumV1; 
MPVersionEnum const MPVersionEnumV2; 
MPVersionEnum const MPVersionEnumV3; 

```

以以下方式导入到 Swift 中：

```swift
enum MPVersionEnum: String{ 
     case V1 
     case V2 
     case V3 
} 

```

### 注意

您可以在以下位置阅读提案：[`github.com/apple/swift-evolution/blob/master/proposals/0033-import-objc-constants.md`](https://github.com/apple/swift-evolution/blob/master/proposals/0033-import-objc-constants.md)

## 导入 Objective-C 轻量级泛型 [SE-0057]

在 Swift 2 中，您可以导入并交互 Objective-C 轻量级泛型。虽然您可以将任何形式的轻量级泛型导入到 Swift 2 中，但只有 Foundation 类型（`NSArray`、`NSSet` 和 `NSDictionary`）在导入后保留了它们的类型信息。

### 注意

您可以通过阅读有关此主题的 Swift 文档来了解更多关于轻量级泛型的信息：[`developer.apple.com/library/content/documentation/Swift/Conceptual/BuildingCocoaApps/InteractingWithObjective-CAPIs.html`](https://developer.apple.com/library/content/documentation/Swift/Conceptual/BuildingCocoaApps/InteractingWithObjective-CAPIs.html)

在 Swift 3 中，您可以导入自己的 Objective-C 泛型而不会丢失类型信息。在我们的示例中，我们使用 Foundation 集合类型和自定义泛型类定义了几个属性类型。请注意，在 Swift 2 和 Swift 3 中，泛型都正确地导入了 Foundation 集合类型的类型信息。

对于 Objective-C：

```swift
@property NSArray<MyClass *> * myClasses; 
@property NSDictionary<NSString *, MyClass *> * myClassDictionary; 
@property NSSet<MyClass *> *mySet; 
@property MyCollection<MyClass *> *myCollection; 

@interface MyCollection<__covariant ObjectType> : NSObject 
 -(void) addItem:(ObjectType)item; 
@end
```

以以下方式导入到 Swift 2 中：

```swift
var myClasses: [MyClass] 
var myClassDictionary:  [String : MyClass] 
var mySet: Set<MyClass> 
var myCollection: MyCollectionMyCollection Classfunc addItem(item: AnyObject!)
```

很遗憾，当我们导入到 Swift 2 时，我们自定义泛型属性 `myCollection` 的类型信息丢失了。Swift 正确地确定 `myCollection` 属性的类型为 `MyCollection`，但参数化数据丢失了。由于我们在导入时丢失了类型信息，访问 `myCollection` 类的 `addItem()` 方法将导致使用 `AnyObject` 作为参数类型的签名。这不是我们期望的类型 `MyClass`，与 Objective-C 签名匹配。

在 Swift 3 中，这个导入问题得到了解决。我们的自定义 Objective-C 泛型按预期导入。检查我们的 Swift 3 导入，我们可以看到所有的泛型都保留了它们的类型信息，并且我们的类方法将完全匹配自定义泛型类的 Objective-C 对应者。

以以下方式导入到 Swift 3 中：

```swift
var myClasses: [MyClass] 
var myClassDictionary:  [String : MyClass] 
var mySet: Set<MyClass> 
var myCollection: MyCollection<MyClass>  
MyCollection Class 
func addItem(_ item: MyClass)
```

### 注意

您可以在以下位置阅读提案：[`github.com/apple/swift-evolution/blob/master/proposals/0057-importing-objc-generics.md`](https://github.com/apple/swift-evolution/blob/master/proposals/0057-importing-objc-generics.md)

## 以成员方式导入 [SE-0044]

许多 C API 提供了允许你创建、访问和修改 C 结构的函数。以原样导入这些库将把这些函数添加到你的 Swift 全局命名空间中。虽然这在许多情况下是可行的，但可能更倾向于将这些导入分组在 Swift 的常见类型下。Swift 3 允许我们使用 `CF_SWIFT_NAME` 宏来控制初始化器、属性和方法在 Swift 中的显示方式。让我们看看我们如何完成这些任务。

### 定义一个初始化器

我们可以通过在头文件中函数定义后附加 `CF_SWIFT_NAME` 宏来修改 C 函数以创建 Swift 初始化器。为了告诉 Swift 我们想要将函数添加为特定类型的初始化器，我们需要提供类型（`MyPlatform`）后跟一个点和我们想要的 *init* 格式（即 C 函数定义中的参数）。Swift 将我们的新初始化器作为 `MyPlatform` 的扩展导入，代表我们的常见类型。

### 小贴士

Swift 添加任何扩展都需要存在一个常见类型。Swift 不会为你创建这个类型，如果不存在类型，Swift 将会静默失败。

对于 C：

```swift
MyPlatform* MyPlatformWithIdentifier(MPPlatformIdentifier identifier) CF_SWIFT_NAME(MyPlatform.init(identifier:)); 

```

这将导入到 Swift 中为：

```swift
extension MyPlatform { init(identifier: MPPlatformIdentifier) }
```

### 创建获取器和设置器

除了初始化器之外，我们还可以创建计算属性。我们可以定义将被导入到 Swift 中的获取器和设置器。对于属性的获取器，我们需要在作为获取器的 C 函数末尾添加 `CF_SWIFT_NAME` 宏，并提供一个常见类型和属性名称。我们的设置器将与获取器类似，我们将在作为设置属性服务的 C 函数中添加宏。下面是语法在实际中的应用。在 Swift 3 中，获取器和设置器将通过扩展添加到常见类型 `MyPlatform`。

对于 C：

```swift
//getter 
MPPlatformIdentifier MyPlatformGetIdentifier(MyPlatform *platform) CF_SWIFT_NAME(getter:MyPlatform.platformId(self:)); 

//setter 
Void MyPlatformSetIdentifier(MyPlatform *platform, MPPlatformIdentifier identifier) CF_SWIFT_NAME(setter:MyPlatform.platformId(self:newValue:)); 

```

这将导入到 Swift 中为：

```swift
extension MyPlatform {  
var platformId: MPPlatformIdentifier { get set } 
} 

```

### 添加方法

在我们将内容导入到 Swift 3 的过程中，我们可以将方法分组在常见类型下。这是通过在我们的头文件中 C 函数定义后附加 `CF_SWIFT_NAME` 宏来实现的。你需要为宏提供一个常见类型和一个与 C 函数使用相同数量变量的方法签名。Swift 将处理确定参数类型。你只需要提供你想要在 Swift 中用作签名的名称。

对于 C：

```swift

MPPlatformIdentifier MyPlatformRetreiveId(MyPlatform *platform, MPPlatformIdentifier identifier) CF_SWIFT_NAME(MyPlatform.retreiveId(self:identifier:)); 

```

这将导入到 Swift 中为：

```swift
extension MyPlatform { 
   func retreiveId(identifier: MPPlatformIdentifier) -> MPPlatformIdentifier 
}
```

### 创建静态变量

最后，我们可以在 Swift 3 中将全局变量作为与常见类型关联的静态变量导入。我们只需要为我们的 `CF_SWIFT_NAME` 宏提供一个常见类型和变量名称。我们的静态变量将通过扩展添加到 Swift 中的常见类型。

对于 C：

```swift
extern const MPPlatformIdentifier *MyPlatformTVOS CF_SWIFT_NAME(MyPlatform.tvOS); 

```

这将导入到 Swift 中为：

```swift
extension MyPlatform { static var tvOS: MPPlatformIdentifier } 

```

### 注意

你可以在以下位置阅读提案：[`github.com/apple/swift-evolution/blob/master/proposals/0044-import-as-member.md`](https://github.com/apple/swift-evolution/blob/master/proposals/0044-import-as-member.md)

# 摘要

我们刚刚完成了 Swift API 指南原则的讲解。此外，我还提供了如何查找记录每个变更动机的 Swift Evolution 提案的资源。我们还涵盖了在 Swift 中使用 Objective-C API 时处理`#selector`和`#keyPath`表达式的全新功能。最后，我们探讨了如何与 C API 一起工作并控制它们如何导入到 Swift 3 中。在下一章中，我们将介绍更多的语言变更。请保持关注，因为我们还有很多新内容要介绍！
