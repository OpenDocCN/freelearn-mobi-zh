

# 理解持久化存储

虽然，作为 iOS 开发者，我们主要关注 UI 相关的主题，如 UIKit 和 SwiftUI，但还有其他 iOS 开发的关键方面需要考虑，例如持久化存储。这个主题至关重要，因为它使我们能够在 App 关闭后存储和检索信息。

管理持久化存储的好处很多，其中一些细节如下：

+   **提升用户体验**：能够保存和稍后检索用户数据的 App 可以提供更好的用户体验。例如，如果用户已从我们的后端服务器下载了信息，我们可以在他们下次进入 App 时立即展示这些信息，而无需等待网络请求完成。

+   **提供离线访问**：离线访问是一个伟大的功能，允许用户在离线时也能使用我们的 App。例如，一个消息应用可能允许用户在无互联网连接的情况下查看他们之前的对话。

+   **保持本地状态**：使用持久化存储，我们可以在 App 关闭后保持本地状态。例如，存储访问令牌、用户配置文件详情或从用户上次停止的地方继续用户体验是我们可以添加到 App 中的关键功能。

持久化存储对于 iOS 开发者来说是一个关键组件，它涉及到用户体验、安全和效率。

本章涵盖了持久化存储中的以下重要主题：

+   掌握 *Core* *Data* 问题

+   使用 **UserDefaults** 处理持久化状态

+   在 *密钥链* 中存储敏感信息

+   与 *文件系统* 一起工作

许多年轻的开发者常常对名为 Core Data 的框架感到畏惧。让我们从探索这个主题开始。

# 掌握 Core Data 问题

大多数面试官不会询问关于通用框架的问题（除了 UIKit、SwiftUI 和 Foundation 之外），但 Core Data 被视为一个例外。**Core Data** 是 iOS 开发中的基本框架，因为它是一个设置数据层和管理持久化存储的优化且简单的解决方案。

Core Data 在这些年中不断发展，成为许多开发者的主要框架。它完美地集成到 iOS 开发生态系统中，与 Xcode 和其他 iOS 框架完美配合。

关于 Core Data 有几个概念需要了解：

+   **数据模型**：Core Data 围绕一个数据模型构建，该模型定义了 App 中存储的数据结构。数据模型通常使用 Xcode 的数据建模工具确定，包括实体（代表 App 中的对象）、属性（描述这些对象的属性）和关系（定义实体之间如何相互关联）。

+   **管理对象上下文**：管理对象上下文是 Core Data 框架的核心。它负责管理应用数据对象（称为“管理对象”）的生命周期，并为应用提供了查询、创建、更新和删除这些对象的方式。它还负责处理撤销/重做操作和管理对象关系。

+   **持久化存储协调器**：持久化存储协调器负责管理应用的数据持久化存储，数据存储在磁盘上。它协调管理对象上下文和持久化存储之间的通信，并确保对管理对象上下文所做的更改被正确地持久化到磁盘。

+   **查询请求**：我们使用查询请求来查询应用的数据模型并从管理对象上下文中检索特定对象。我们还可以使用谓词（以过滤结果）、排序描述符（以排序结果）和查询限制（以限制返回的结果数量）来自定义查询请求。

+   **关系**：Core Data 提供了一种强大的机制来定义数据模型中实体之间的关系。关系可以是一对一、一对多或多对多，可以是单向的或双向的。关系还可以配置删除规则，这些规则定义了当关系断开时对象应该如何被删除。

+   **迁移**：Core Data 包含在不同数据模型版本之间迁移数据的工具。这允许开发者随着时间的推移更改数据模型，同时保留现有数据。

我们还了解 Core Data 的另一个重要术语，那就是 **Core Data 栈**。Core Data 栈是一个层集，允许我们的应用与持久化存储进行交互。

这里是 Core Data 栈的三个层级：

+   **管理对象模型**：这是包含数据模型的底层。因为一切都是围绕数据结构构建的，所以我们从这个层开始。

+   **持久化存储协调器**：这是基于对象模型构建的持久化存储协调器。协调器使用数据模型来定义一个与数据方案相对应的持久化存储。存储可以基于 XML、SQLite、JSON 或后端服务。Core Data 允许我们基于我们想要的任何技术来构建持久化存储。关于持久化存储的另一个重要事项是，它需要与数据模型完全匹配。任何数据模型的变化都要求对存储进行修改并执行迁移。

+   **管理对象上下文**：这是管理应用数据对象生命周期（称为“管理对象”）的管理对象上下文。它为应用提供了查询、创建、更新和删除这些对象的方式。它还负责处理撤销/重做操作和管理对象关系。

我们需要设置 Core Data 栈以开始使用 Core Data。这可以通过使用 `NSPersistentContainer` 快速完成：

```swift
import CoreData// 1\. Create a persistent container
let container = NSPersistentContainer(name: "MyDataModel")
// 2\. Load the persistent store
container.loadPersistentStores { (storeDescription, error)
    in
    if let error = error {
        fatalError("Failed to load persistent store: \(error)")
    }
}
// 3\. Create a managed object context
let context = container.viewContext
```

在代码示例中，我们使用`MyDataModel`。这一行也创建了持久存储并返回一个容器对象。

之后，我们将持久存储加载到堆栈中，并创建一个管理对象上下文，这样我们就可以开始使用 Core Data 了。

一旦我们有了上下文，我们就可以执行实体创建、获取、更新和删除操作：

```swift
let newEmployee = Employee(context: context)newEmployee.name = "John Doe"
newEmployee.title = "Software Engineer"
newEmployee.startDate = Date()
do {
    try context.save()
} catch {
    fatalError("Failed to save context: \(error)")
}
```

代码如此简单，以至于它自身就能说明一切。我们根据之前创建的上下文创建一个新的`Employee`对象，设置其属性，并使用`context`的保存方法保存新对象。

`Employee`是`NSManagedObject`的子类，它允许我们直接访问实体的属性，并提供了修改其数据的一种简单方式。`context.save()`操作将更改提交到持久存储。

一般而言，Core Data 的主要用法很简单，而且随着时间的推移变得更加简单。虽然设置 Core Data 堆栈对于使用 Core Data 至关重要，但这并不是开发者需要掌握的唯一方面。由于与 Core Data 一起工作的挑战和复杂性，面试官通常会询问这些挑战，而不仅仅是基本设置。

这里有一些我们可能在面试前需要很好地学习的挑战：

+   **处理并发**：并发是一个复杂的话题，不仅在于 Core Data，还在于任何持久存储或本地数据。在 Core Data 中执行并发任务并避免数据丢失和异常，存在几种技术和模式。

+   **设计数据模型**：从技术上讲，设置具有实体和属性的数据模型是一项简单的工作，因为我们的大部分工作都在模型编辑器中完成，这是一个内置的 Xcode 编辑器。但真正的挑战是以一种服务于我们应用程序关键旅程和任务的方式设计数据模型。我们需要掌握主要术语，如**多对多**和**一对多**，并完全理解**删除规则**。

+   **理解数据迁移**：随着时间的推移，我们不得不修改我们的数据模型，添加、编辑和删除实体和属性。在持久存储中有数据时更改数据模型称为“数据迁移”，这也是作为 iOS 开发者，我们需要理解和知道如何执行的关键话题。在这个领域的错误可能会导致数据损坏甚至崩溃。

现在，让我们过渡到一些专门针对这些主题的 Core Data 相关的问题。

## “你如何设计一个支持并发同时确保线程安全的 Core Data 堆栈，以及你如何在多线程环境中使用 NSManagedObjectIDs 来促进这一点？”

*这个问题的意义是什么？*

并发是 Core Data 开发中的一个重要话题，妥善处理并发显示了我们对 Core Data 上下文工作原理的深入理解。

如果我们理解了 Core Data 上下文的工作原理，我们也应该回答这个问题的第二部分——`NSManagedObjectID`，它可以帮助我们在不同的上下文中识别对象。

*答案是什么？*

首先，让我们谈谈与 Core Data 并发相关的两个原则：

+   **将上下文视为沙盒**：当与上下文一起工作时，我们可以创建对象、更新和删除它们。我们只在这个上下文中执行所有这些操作，而不是在持久存储中。当我们调用**save()**方法时，Core Data 会将更改推送到父上下文，如果没有父上下文，则推送到持久存储。我们应该将上下文视为沙盒——我们可以在准备好时执行更改并提交它们。

+   **从创建上下文的同一线程访问上下文**：在 Core Data 中，上下文属于一个线程。一旦我们创建了一个新的后台线程，我们就无法访问在另一个线程中创建或检索的对象。每个线程都需要自己的上下文来允许它。

在我们理解了这两个原则之后，我们可以尝试定义设置我们的 Core Data 堆栈的不同模式。这些模式依赖于我们的应用程序需求和需求。

让我们回顾一下。

### 使用多个上下文

在多个上下文模式中，我们有几个私有上下文，并且每个上下文都直接与持久存储一起工作。多个上下文负责读取和写入，并提供了一种灵活的方式来处理我们应用程序中的不同并发操作，甚至以模块化的方式做到这一点。

这里有一个例子：

```swift
let persistentContainer = NSPersistentContainer(name: "MyApp")persistentContainer.loadPersistentStores(completionHandler: { (storeDescription, error) in
    if let error = error {
        fatalError("Failed to load persistent store: \(error)")
    }
})
let writeContext = NSManagedObjectContext(concurrencyType:
    .privateQueueConcurrencyType)
writeContext.persistentStoreCoordinator =
    persistentContainer.persistentStoreCoordinator
let readContext = NSManagedObjectContext(concurrencyType:
    .privateQueueConcurrencyType)
readContext.persistentStoreCoordinator =
    persistentContainer.persistentStoreCoordinator
```

在这个例子中，我们创建了两个上下文用于写入（`writeContext`）和读取（`readContext`）。重要的是要注意，我们对写入上下文所做的更改不会反映在读取上下文中，直到我们执行保存和重新检索。

多个上下文模式有一些显著的缺点。例如，这种模式可能会增加我们的数据库复杂性，并要求我们在上下文之间管理更改。

我们可能遇到的另一个缺点是与批处理更改和重做/撤销等特性相关的复杂性，这在某些应用程序用例中可能是关键的。

然而，我们有一个相当不错的替代方案：父-子上下文。让我们来谈谈它。

### 使用父-子上下文

如果多个上下文模式具有“扁平”结构，则父-子模式更具层次性。基本原理指出，我们有一个根上下文（也称为“父”），专门用于写入操作，而子上下文专门用于读取操作。父上下文是私有的，子上下文与主队列一起工作。我们在写入上下文中进行的每个更改都会反映在子上下文中。

父-子模式对于后台更新、离线编辑和撤销-重做用例非常出色。

这里是一个父-子模式的代码示例：

```swift
let parentContext = NSManagedObjectContext(concurrencyType:    .privateQueueConcurrencyType)
parentContext.persistentStoreCoordinator =
    persistentStoreCoordinator
let readContext1 = NSManagedObjectContext(concurrencyType:
    .mainQueueConcurrencyType)
readContext1.parent = parentContext
let readContext2 = NSManagedObjectContext(concurrencyType:
    .mainQueueConcurrencyType)
readContext2.parent = parentContext
```

我们可以看到父上下文是私有的，而子上下文是主要的。我们还可以看到我们如何直接将子上下文链接到其父上下文，而不是链接到持久存储。

“私有上下文”是什么意思？

私有上下文通常用于执行导入或导出数据等后台任务，允许这些任务异步执行而不阻塞主线程。

总体而言，父-子关系和多个上下文服务于不同的目的和用例。重要的是要说明它们提供了使用 Core Data 管理并发的基本原则。结合这些模式，甚至使用它们的原理来创建新的模式是完全可行的。

### 那么，使用`NSManagedObjectID`呢？

当与上下文一起工作时，我们不能将一个上下文中获取的托管对象用于另一个上下文。我们需要使用`NSManagedObjectID`重新获取它。这个关键点如果没有正确执行可能会导致异常。

这里有一个使用`NSManagedObjectID`在另一个上下文中重新获取相同对象的代码示例：

```swift
var object: MyEntity?let fetchRequest: NSFetchRequest<MyEntity> = MyEntity.fetchRequest()
fetchRequest.predicate = NSPredicate (format: "id == %@", someID)
do {
    object = try mainContext.fetch(fetchRequest).first
} catch {
    print("Error fetching object: \(error)")
}
guard let objectID = object?.objectID else {
    return
}
Let backgroundContext = NSManagedObjectContext
    (concurrencyType: .privateQueueConcurrencyType)
backgroundContext.persistentStoreCoordinator =
    persistentStoreCoordinator
backgroundContext.perform {
    let backgroundObject = backgroundContext.object(with:objectID)
}
```

每个托管对象都有一个名为`objectID`的属性，它在不同上下文中与同一对象相同。它的主要目的正是如此——确保在上下文之间移动时使用的是同一对象。`NSManagedObjectID`是执行 Core Data 并发操作时的一个基本概念。

## “对于一个包含成分、烹饪说明和用户评分的食谱应用，你的 Core Data 数据模型会是什么样子？”

*为什么这个问题很重要*？

我们是 iOS 开发者，而不是数据库管理员，但我们有重要的责任来设计一个符合我们应用业务需求的数据模型。糟糕的数据模型设计会直接影响应用的性能和稳定性。作为开发者，将应用的需求和工作流程转换为技术设计是我们的责任，这个问题评估了我们实现这一目标的熟练程度。

*答案是什么*？

为特定的应用使用创建数据模型需要我们遵循以下步骤：

1.  理解*应用的需求*和基本流程。

1.  定义*不同的实体及其属性*，包括数据类型、默认值等。

1.  为相关属性创建*索引*。

1.  理解不同实体之间的*关系*——一对一或一对多。

1.  设置*删除*规则。

这些步骤在接近数据模型设计时始终相关，问题中描述的用例也不例外。

第一步是确定应用的主要工作流程。重要的是要记住，数据层应该支持业务逻辑和 UI，而不是独立存在。向面试官提出额外的问题是完全合法的；这甚至是任务的一部分。

第二步，我们可以定义基本实体：

+   **食谱**：这包含单个食谱的基本细节。属性：标题，创建日期，描述，难度级别和类别。

+   **成分**：这可以在多个食谱之间共享。属性：名称。

+   **指令**：这是使用配方时所需的步骤之一。属性：描述，标题。

+   **用户评分**：这是对单个食谱的评论。属性：名称，评分（**int**），和描述。

虽然实体是数据模型的关键部分，但如果没有明确定义它们之间的关系，它们通常是没有效果的。

因此，让我们了解实体是如何链接的，以及主要挑战（因为有一些挑战！）。

我们理解`Recipe`与`Instruction`和`User Rating`之间存在一对一和一对一的关系。这很简单。我们该如何处理成分呢？简单的方法是定义一个与`Ingredient`的一对多关系，就像我们处理`Instruction`和`User Rating`一样。关于`Ingredient`的问题是我们可以在多个食谱之间共享它。例如，假设我们有一个帮助用户根据成分找到所有可用食谱的功能。

在这种情况下，这里需要一个多对多的关系。因此，这里有一些我们学到的知识：为了有效地构建数据模型，与面试官沟通并评估应用程序的各种功能是很重要的。我们还应该讨论数据的潜在用例，以确定最佳方法。

如果我们需要在食谱之间共享成分并创建多对多关系，我们可能需要开发一个不同的实体，那就是`IngredientUsage`。

`IngredientUsage`代表一个食谱中`Ingredient`的一次使用，并在`Recipe`和`Ingredient`之间建立链接。除了这个链接之外，它还提供了更多信息，例如数量。

为了进一步探讨这个问题，让我们来看一下*图 9**.1*：

![图 9﻿.1 – Recipe、IngredientUsage 和 Ingredient 之间的关系](img/Figure_9.1_B18653.jpg)

图 9.1 – Recipe、IngredientUsage 和 Ingredient 之间的关系

在*图 9**.1*中，我们解决的主要问题是 Core Data 无法在关系中放置关于关系的附加信息。我们可以在`Recipe`和`Ingredient`之间定义一对多关系，但不能告诉数量。数量不能成为`Ingredient`的一部分，因为我们希望与其他食谱共享成分，这些食谱可能有不同的数量。这就是`IngredientUsage`的作用。

另一个有用的提示是记住，在多对多关系的任何情况下都可能发生类似的问题。如果需要加载包含更多信息的关系，为关系创建一个专门的实体可以有效地解决这个问题。

现在我们已经定义了数据模型之间的关系，我们需要考虑删除规则。Core Data 有几个删除规则，选择正确的规则很重要，这样我们才能随着时间的推移维护我们的数据。

例如，如果我们删除`Recipe`，我们希望其`IngredientUsage`、`User Rating`和`Instruction`数据被删除。因此，我们将`Recipe`与其他实体之间的删除规则定义为**级联**。级联是一个删除规则，如果源对象被删除，则删除相关对象。

然而，`IngredientUsage`和`Ingredient`之间并不是这样。如果我们删除`IngredientUsage`，我们不希望删除成分，因为它正在与其他`IngredientUsage`对象共享。在这种情况下，我们设置了**Nullify**的删除规则。

关于反向关系呢？如果我们删除了一个 `User Rating` 对象，是否需要删除一个 `Recipe` 对象？可能不需要。如果我们删除一个 `User Rating` 对象，我们希望保留 `Recipe`。对于 `Instruction` 也是如此。在这两种情况下，我们都将删除规则设置为 *Nullify*。

为了回顾我们的方法，我首先概述了五个基本步骤，这些步骤构成了定义数据模型的过程。重要的是要注意，每一步都是建立在之前步骤的基础上的。为了继续回答，我们可以与面试官采取一种协作方法，通过逐一讨论每个步骤并大声说出我们的思考过程。这样做将确保得到一个很好的答案，即使解决方案可能不是完美或理想的。记住，面试官想看到我们的思考，而不是解决一个真实的问题。

## “你将如何测试 iOS 应用中的 Core Data？”

*为什么这个问题很重要？*

Core Data 是一个至关重要的框架，在应用程序的数据层中扮演着重要的角色。因此，在我们测试应用程序时，它将始终存在。但这究竟意味着什么？我们如何测试持久化存储？这个问题检验了我们对于使用 Core Data 可以执行的不同测试以及执行这些测试的有效和一致性的工具的理解。

*答案是什么？*

我们可以使用 Core Data 执行三种不同的测试类型，我们已经在本书的早期讨论了测试。如果您需要刷新，请回到*第六章*并确保您熟悉不同类型的测试。

我们可以执行的第一种测试类型是单元测试——在这种情况下，我们想要模拟一个 Core Data 栈，而不使用实际的持久化存储文件。为了做到这一点，我们可以创建一个 **内存中的**持久化存储。内存中的持久化存储轻量级，不使用实际的数据库文件，并在 RAM 中执行所有 I/O 操作。

设置内存存储很简单：

```swift
let managedObjectModel = NSManagedObjectModel.mergedModel    (from: [Bundle.main])!
let persistentStoreCoordinator =NSPersistentStoreCoordinator
    (managedObjectModel: managedObjectModel)
do {
    try persistentStoreCoordinator.addPersistentStore(ofType: 
   NSInMemoryStoreType, configurationName: nil, at: nil, options: nil)
} catch {
    fatalError("Failed to create in-memory persistent store
        coordinator: \(error)")
}
```

在前面的代码中，我们添加了一个类型为 `NSInMemoryStoreType` 的持久化存储，它只创建一个内存中的存储。

与 Core Data 相关的另一种相关测试类型是集成测试——在这种情况下，我们希望保持 Core Data 存储不变，并验证我们在业务逻辑或甚至 UI 层执行的操作是否被保存到 Core Data 文件中。在测试用例前后使用临时数据库文件或清理数据存储是一个好的做法。

与 Core Data 相关的第三种测试是性能测试。Core Data 包含一些可能执行起来很重的 I/O 操作，确保我们的应用程序中没有瓶颈是一个好主意。我们可以使用 `XCTestCase` 中的 `measure` 函数来检查 I/O 操作：

```swift
func testFetchPerformance() {    let context = persistentContainer.viewContext
    let request = NSFetchRequest<MyEntity>(entityName: "MyEntity")
    measure {
        do {
            for _ in 0..<200 {
                let result = try context.fetch(request)
            }
            let result = try context.fetch(request)
            XCTAssertEqual(result.count, 1000)
        } catch {
            XCTFail("Failed to fetch objects:
                \(error.localizedDescription)")
        }
    }
```

在前面的例子中，我们执行了 Core Data 检索并测量了其持续时间。为了模拟工作负载，我们执行了 `200` 次检索操作。测试断言当持续时间超过某个阈值时。

总结一下——我们可以在单元测试中测试 Core Data，消除任何 I/O 操作，集成测试以确保我们的系统按预期工作，以及性能测试来检查 Core Data 对我们应用性能的影响。

# 使用 UserDefaults 处理持久状态

`UserDefaults`是 iOS 开发者以键值格式存储持久信息的根本且简单的方法。我们可以轻松地使用 UserDefaults 存储和检索布尔值、整数、字符串、数组和字典值。

例如，这是我们在 UserDefaults 中存储布尔值的方法：

```swift
let defaults = UserDefaults.standarddefaults.set(true, forKey: "isUserLoggedIn")
```

这是读取它的方法：

```swift
let defaults = UserDefaults.standardlet isUserLoggedIn = defaults.bool(forKey: "isUserLoggedIn")
```

UserDefaults 的目标不是存储和检索大型数据集——对于这种情况，UserDefaults 是一个慢速且不安全的解决方案。如果我们想管理本地数据存储，我们应该使用 Core Data 或 SQLite 来完成这项任务。

如前所述，UserDefaults 是一个非常简单且直接的工具。然而，它仍然有一些我们可能在准备面试时需要了解的高级功能。让我们回顾一些：

## “解释 iOS 应用及其扩展如何使用 UserDefaults 共享数据。设置和使用 UserDefaults 在应用及其扩展之间共享数据涉及哪些步骤？”

*为什么这个问题很重要？*

许多应用在其生命周期中快乐地独立存在。但一旦我们添加了一个扩展或另一个应用，我们可能需要在它们之间共享一些数据。例如，我们可能需要共享密钥、令牌或配置文件信息。

幸运的是，Apple 为我们提供了一个安全且简单的方式来做到这一点。让我们看看答案。

*答案是什么？*

这些是我们需要在 iOS 应用和其扩展之间共享数据时执行的步骤：

1.  **设置 App Group**：App Group 功能允许我们在多个扩展或应用之间组合和同步数据。我们使用开发者门户创建一个新的 App Group，并为其应用和扩展启用。

1.  **配置 Xcode 项目设置**：一旦我们在开发者门户中设置了 App Group，我们就需要为我们的应用和扩展配置 Xcode 项目设置。我们需要指定 App Group 标识符并启用我们的应用和扩展的 App Group 权限。

1.  **使用 UserDefaults 共享数据**：一旦配置了 Xcode 项目设置，我们就可以使用 UserDefaults 在应用和扩展之间共享数据。为此，我们需要创建一个新的**UserDefaults**对象，将 App Group 标识符作为套件名称，然后使用**set(_:forKey:)**方法保存数据，并使用**object(forKey:**)方法检索数据。

下面是一个如何设置和从共享`UserDefaults`中读取数据的代码示例：

```swift
let defaults = UserDefaults(suiteName: "group.com.yourcompany.     yourapp")!
defaults.set(true, forKey: "myBoolValue")
let myBoolValue = defaults.bool(forKey: "myBoolValue")
```

在这个代码示例中，我们正在设置一个新的`UserDefaults`，并传递我们在开发者门户中定义的 App Group 标识符。

在本节中，其余的代码保持不变：使用键值机制进行读取和写入。

如我们所见——在应用和扩展之间共享数据是直接的，但这也是许多开发者不太熟悉的事情。在应用和扩展之间共享 Core Data 存储也是一个基本功能。

## “你能解释如何在 iOS 应用中用 UserDefaults 存储结构体或类吗？”

*为什么这个问题很重要*？

保存原始字典和数组是使用 `UserDefaults` 时最常见的情况。但有许多情况我们需要存储整个对象或类。例如，我们有时需要保存包含完整详细信息的用户对象，而 Core Data 就像是用大锤砸核桃。

*什么是* *答案*？

`UserDefaults` 存储对象或结构体有两种流行的方式：

+   第一个选项是使用 **NSKeyedArchiver**。我们可以在 **UserDefaults** 中保存的数据类型之一是 **Data**。**NSKeyedArchiver** 可以将结构体或对象转换为 **Data** 对象，可以直接保存到 **NSKeyedArchiver** 中。要解档对象，我们可以使用 **NSKeyedUnarchiver**。让我们看看一个代码示例：

    ```swift
    struct Person {    var name: String    var age: Int}let person = Person(name: "John Smith", age: 30)let data = try? NSKeyedArchiver.archivedData(withRoot    Object: person, requiringSecureCoding: false)UserDefaults.standard.set(data, forKey: "person")let storedData = UserDefaults.standard.data(forKey:"person")if let storedPerson = try? NSKeyedUnarchiver.unarchivedObject    (ofClass: Person.self, from:storedData!) {}
    ```

在这个代码示例中，我们使用 `NSKeyedArchiver` 将 `Person` 结构体转换为 `Data` 对象。数据可以像任何其他保存到 `UserDefaults` 的数据类型一样轻松设置和恢复。在获取数据后，我们可以使用 `NSKeyedUnarchiver` 再次将其转换为 `Person` 对象。

+   第二个选项是使用 **JSONEncoder**。与 **NSKeyedArchiver** 一样，我们可以将结构体转换为 **Data** 对象。一旦我们有了数据对象，我们就可以从 **UserDefaults** 中设置和恢复 **Person** 对象。

让我们看看我们如何使用 `JSONEncoder` 保存和恢复相同的 `Person` 结构体：

```swift
struct Person: Codable {    var name: String
    var age: Int
}
let person = Person(name: "John Smith", age: 30)
let encoder = JSONEncoder()
if let encoded = try? encoder.encode(person) {
    UserDefaults.standard.set(encoded, forKey: "person")
    if let storedData = UserDefaults.standard.data
        (forKey: "person") {
        let decoder = JSONDecoder()
        if let storedPerson = try? decoder.decode
            (Person.self, from: storedData) {
        }
    }
}
```

我们使用 `JSONEncoder` 将结构体转换为数据，使用 `JSONDecoder` 将数据恢复回 `Person`。注意，在这种情况下，`Person` 需要遵守 *conform to Codable*，这要求其属性也遵守 Codable。

哪个选项更好？没有明确的答案。通常，对于我们的结构体和类来说，遵守 **Codable** 是最佳实践，因为它提供了更多功能，如自动解析和编码。

另一方面，当处理大型或复杂的数据集时，`NSKeyedArchiver` 比 `JSONEncoder` 更高效。

这两个 API 对于大多数情况都足够好，这取决于我们的应用结构和需求。

# 在 Keychain 中存储敏感信息

`UserDefaults` 是一个出色的存储机制，但它不适合存储密码或令牌等数据。**Keychain** 是 Apple 为存储敏感数据提供的解决方案，它提供了更高的安全性，并且是 iOS 开发者保护其数据的重要工具。

在密钥链中存储数据比使用其他解决方案要复杂得多。密钥链提供了一个基于 C 函数的特定 API，以防止恶意黑客对该 API 的调用进行逆向工程。密钥链在保存时还需要更多信息，以便更有效地保存和索引。

让我们看看如何将一个简单的令牌存储在密钥链中，同时用类包装以方便使用：

```swift
import UIKitimport Security
class KeychainManager {
    private let serviceName = "MyAppTokenService"
    func saveToken(token: String) -> Bool {
        guard let tokenData = token.data(using: .utf8) else {
            return false
        }
        let keychainItem = [
            kSecClass: kSecClassGenericPassword,
            kSecAttrService: serviceName,
            kSecAttrAccount: "MyAppToken",
            kSecValueData: tokenData
        ] as CFDictionary
        let status = SecItemAdd(keychainItem, nil)
        return status == errSecSuccess
    }
}
```

从那个代码示例中，我们首先注意到我们导入了 `Security` 框架，该框架也负责身份验证、安全传输和数据保护。

我们还可以看到，代码比使用 `UserDefaults` 要复杂得多。我们需要创建一个具有多个属性的密钥链项目，而不是存储一个简单的值：

+   **kSecClass**: 此密钥指定了项目的安全级别。我们可以从几个常量中选择，例如 **kSecClassGenericPassword**、**kSecClassInternetPassword** 和 **kSecClassIdentity**。选择正确的类别确保我们的数据有适当的安全级别。

+   **kSecAttrService**: **kSecAttrService** 定义了我们项目的服务名称。我们可以将多个项目分组在一起，以增加安全性，以防应用程序的一部分被破坏，以共享部分密钥链项目，以及更好地组织。

+   **kSecAttrAccount**: 这用于向密钥链项目添加标识，例如用户名或电子邮件。

+   **kSecValueData**: 这是我们要保存的实际数据。它可以是 **Data** 或 **String**。

这四个密钥并不是我们创建密钥链项目时唯一可以使用的密钥，但它们是最常见的。一旦我们有了 `CFDictionary`，我们就可以使用 `SecItemAdd` 将密钥链项目推入密钥链。

这是如何从密钥链中读取令牌的示例：

```swift
    func readToken() -> String? {        let query: [String: Any] = [
            kSecClass as String: kSecClassGenericPassword,
            kSecAttrService as String: service,
            kSecAttrAccount as String: account,
            kSecReturnData as String: true,
            kSecMatchLimit as String: kSecMatchLimitOne
        ]
        var result: AnyObject?
        let status = SecItemCopyMatching(query as
            CFDictionary, &result)
        if status == errSecSuccess, let tokenData =
            result as? Data, let token = String(data:
                tokenData, encoding: .utf8) {
            return token
        } else {
            return nil
        }
    }
```

要读取令牌，我们再次创建 `CFDictionary` 并使用 `SecItemCopyMatching` 查询密钥链并检索令牌。之后，我们检查结果——如果状态是 `success` 并且我们有 `tokenData`，我们可以通过将其转换为字符串来提取令牌并返回它。

如我们所见，密钥链管理并不像其他存储工具那样简单，密钥链包装器是一个很好的解决方案，可以帮助我们简化这个过程。

## “什么是密钥链访问组，以及如何使用它来在 iOS 应用程序的不同组件之间安全地共享密钥链项目，例如应用程序及其扩展？”

*为什么这个问题很重要？*

在上一节中，我们讨论了 `UserDefaults` 以及如何在我们的应用程序和扩展（或其他应用程序）之间共享信息。现在我们继续这个问题，并询问如何在不同组件之间共享敏感信息。这样，我们的应用程序扩展可以更强大且更独立。

*答案是什么？*

**密钥链访问组** 是一个唯一标识符，指定了哪些密钥链项目可以被特定的应用程序或扩展访问。

访问组在应用程序的权限文件中定义，并在应用程序的不同组件之间提供敏感数据的 secure sharing，例如应用程序及其扩展。

通过为应用程序的多个组件指定相同的访问组，这些组件可以安全地共享密钥链项，而不会损害其完整性。这个特性对于 iOS 应用程序开发者来说非常重要，因为它使他们能够为应用程序的多个组件提供安全可靠的数据存储机制。

这就是如何设置密钥链访问组并使用它的方法。

首先，我们需要在应用程序的权限文件中定义一个新的访问组：

```swift
<key>com.example.myapp.shared-keychain</key><array>
    <string>$(AppIdentifierPrefix)com.example.myapp</string>
</array>
```

我们还必须在扩展的权限文件中定义相同的访问组。

现在，我们可以在我们的代码中使用我们创建的访问组来保存和检索密钥链值：

```swift
func savePasswordToKeychain(password: String) -> Bool {    guard let data = password.data(using: .utf8) else {
        return false
    }
    let query: [String: Any] = [
        kSecClass as String: kSecClassGenericPassword,
        kSecAttrAccessGroup as String:
            "com.example.myapp.shared-keychain",
        kSecAttrAccount as String: "myPassword",
        kSecValueData as String: data
    ]
    SecItemDelete(query as CFDictionary)
    let status = SecItemAdd(query as CFDictionary, nil)
    return status == errSecSuccess
}
```

注意我们添加了`kSecAttrAccessGroup`键到我们的密钥链项中，并使用我们的新密钥链组。

当与密钥链一起工作时，我们可以看到我们的大部分样板代码是不同的密钥链值管理，而设置访问组则简单且直接。

一般而言——与 iOS 密钥链一起工作并不像我们拥有的其他工具那样简单——它需要更多的代码，使用 C 函数，并提供额外的键和信息。但 iOS 开发要求我们处理敏感信息，甚至在我们之间共享它。因此，我们必须了解密钥链是如何工作的，以及如何使用其 API 来处理它。

# 与文件系统一起工作

有一个常见的假设是 iOS“没有文件系统”。尽管有“文件”应用，但对于大多数标准用户来说，文件系统几乎是隐藏的。

对于 iOS 开发者来说，情况并非如此。

iOS 开发者使用 iOS 文件系统来存储文档、图像、缓存文件，甚至数据库文件。

文件系统允许我们存储大量信息，处理资源，甚至与其他应用程序组件共享数据。大多数面试问题都集中在组织我们的文件和响应不同的用例上。理解沙盒是如何构建的对于我们作为开发者来说至关重要。

因此，让我们回顾一个关于我们的沙盒结构的问题。

## “你能解释一下 iOS 应用程序中以下文件夹的用途吗：Documents、Library、Cache 和 Temp？你将如何决定在你的应用程序中存储不同类型的文件使用哪个文件夹？”

*为什么这个问题很重要？*

从技术角度来看，iOS 中的文件操作在读取和写入方面通常很简单。然而，关键在于掌握在适当文件夹中正确组织文件的方法论概念。每个文件夹都有其独特的作用和特性，iOS 系统会明确管理每个文件夹。

*答案是什么？*

如前所述，每个文件夹都有其独特的作用和特性，所以让我们在这里了解一下：

+   **Documents**: 此文件夹旨在存储用户可以创建或编辑的数据，例如文档、图片和视频。此文件夹由 iCloud 备份，并且用户可以通过 iTunes 文件共享看到。我们应该为此文件夹使用用户期望在应用关闭后仍然可用的数据。

+   **Library**: 此文件夹旨在存储由用户未创建的应用特定数据，例如下载内容、缓存文件和首选项。此文件夹由 iCloud 备份，但用户通过 iTunes 文件共享无法看到。我们应该为此文件夹使用对应用功能重要但必要时可以重新创建的数据。

+   **Cache**: 此文件夹设计用于存储可以重新生成或再次下载的临时文件。此文件夹不由 iCloud 备份，当设备存储空间不足时，系统可以清空此文件夹。我们应该为此文件夹使用对应用功能非关键且必要时可以丢弃的数据。

+   **Temp**: iOS 还为存储临时文件提供了一个临时目录，称为**Temp**文件夹。此文件夹仅用于存放所需的临时文件，当应用关闭时可以删除。

内容类型引导我们决定将文件存储在哪个文件夹中。例如，我们希望使用`Documents`文件夹来存储用户生成的文件。如果我们需要临时文件来生成信息或计算，可以使用`Temp`文件夹。`Library`文件夹适合存储本地数据持久存储文件。

将文件存储在错误的文件夹可能会导致意外的行为，例如数据丢失、性能问题，以及无端增加用户备份大小。

# 摘要

在本章中，我们讨论了持久内存的一些关键主题。我们讨论了 Core Data 并发和数据模型设计、`UserDefaults`的高级主题、如何使用 Keychain 处理敏感信息，以及我们应用沙盒中的不同文件夹。

到现在为止，我们应该为面试中的那个主题做好准备！

下一章将涵盖一个可能阻碍 iOS 开发者（如果他们不熟悉）的应用可扩展性的关键主题：CocoaPods 和 Swift 包管理器。
