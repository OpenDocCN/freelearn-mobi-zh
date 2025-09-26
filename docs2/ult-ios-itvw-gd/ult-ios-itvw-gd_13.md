

# 库管理

到目前为止，我们已经讨论了 *我们编写的代码* - Swift、UIKit 和 SwiftUI。但现代开发者的工作并不仅仅是编写代码。了解如何集成代码可以是一个生产力倍增器，大大提高我们的效率，并使我们能够在更短的时间内完成更多的工作，而不仅仅是知道如何编码。

**CocoaPods** 和 *Swift 包管理器* 是我们今天管理第三方和本地依赖项的主要解决方案。对于任何 iOS 开发者来说，彻底理解这些工具是至关重要的。

本章从以下主题的角度涵盖了 CocoaPods 和 Swift 包管理器：

+   学习 *CocoaPods 的构建方式*，包括如 **Podfile** 和 **Podspec** 文件等不同组件

+   概述 *CocoaPods 的最佳实践* 和用例

+   涵盖 Swift 包管理器的 *创建过程*

+   学习 Swift 包管理器的 *常用命令*

+   学习如何在我们的项目中 *使用 Swift 包*

+   比较 Swift 包管理器与 CocoaPods 的 *不同优势*

+   使用 Swift 包管理器 *组织我们的项目*

这是一个“简单”的章节，然而，在当今的 iOS 开发世界中，它是一个至关重要的主题。让我们从 CocoaPods 作为我们的第一个依赖管理器开始。

# 精通 CocoaPods

**CocoaPods** 是 iOS 开发者中最受欢迎的依赖管理器之一，并且作为一个开源项目已经维护了多年。

CocoaPods 经常是许多库开发者的首选，并且拥有大量可以轻松集成到 iOS 项目中的框架。

除了拥有大量的框架之外，CocoaPods 还支持与本地框架的集成。它可以帮助我们将项目模块化到不同的库中，使其更加灵活和有序。

让我们看看 CocoaPods 的工作原理以及它是如何构建的。

## 学习如何构建 CocoaPods

当我们使用 CocoaPods 在 Xcode 项目中管理依赖项时，CocoaPods 会创建一个新的工作区，该工作区包括我们的项目以及我们在 `Podfile` 文件中指定的任何依赖项。当我们运行 `pod install` 命令时，CocoaPods 会自动创建此工作区。

Xcode 工作区

在 Xcode 中，工作区是一个用于存放一个或多个 Xcode 项目以及构建我们的应用程序所需的任何其他文件和资源的容器。工作区用于组织和管理我们应用程序的不同组件，使开发测试我们的代码更加容易。

在 CocoaPods 中使用工作区有几个好处。它通过将项目和其依赖项包含在同一个工作区中来简化依赖项管理，简化集成，并通过保持依赖项更新来遵循关注点分离原则，无论我们的主项目如何。

除了工作区之外，CocoaPods 还包含几个不同的组件，每个组件都在管理 iOS 的依赖项中扮演着角色。

这里是 CocoaPods 的一些关键组件。

### Podfile

`Podfile` 是一个文件，它指定了我们项目所需的依赖项。它使用简单的 Ruby 语法声明每个 pod 的名称和版本号，以及任何需要的选项或配置。`Podfile` 通常位于项目的根目录中。

这里是一个 `Podfile` 文件的例子：

```swift
platform :ios, 16.0'target 'MyApp' do
  use_frameworks!
  pod 'Alamofire', '~> 5.4'
  pod 'SwiftyJSON', '~> 4.0'
end
```

现在，让我们了解文件是如何构建的：

+   要针对的平台是 iOS 16.0

+   **Podfile** 的目标是名为 **MyApp** 的 Xcode 项目。

+   **use_frameworks!** 指令告诉 CocoaPods 将依赖项构建为 **动态框架**。

+   pod 指令指定了项目的两个依赖项，**Alamofire** 和 **SwiftyJSON**，以及它们的版本要求。

`~>` 操作符是一个 **乐观操作符**，用于指定 pod 的版本，并允许它更新到下一个主要版本。

例如，看看以下行：

```swift
pod 'Alamofire', '~> 5.4'
```

在这种情况下，CocoaPods 将安装和更新 `Alamofire` pod 到版本 6.0（不包括 6.0 本身）。这允许我们享受热修复和次要版本，而不会破坏向后兼容性。

`Podfile` 是我们所有依赖项的项目配置文件，必须仔细维护。

### Podfile.lock

`Podfile.lock` 是一个文件，它存储了我们项目中安装的依赖项的具体版本信息。它确保在每台机器上安装相同的依赖项版本，这有助于防止版本冲突和其他问题。

它看起来是这样的：

```swift
PODS:  - AFNetworking (2.6.3)
  - Firebase/Analytics (7.6.0)
  - Firebase/CoreOnly (7.6.0)
  - FirebaseAnalytics (7.6.0)
  - FirebaseCore (7.6.0)
  - FirebaseCoreDiagnostics (7.6.0)
  - FirebaseInstallations (7.6.0)
  - GoogleAppMeasurement (7.6.0)
DEPENDENCIES:
  - AFNetworking (~> 2.6.3)
  - Firebase/Analytics
  - GoogleAppMeasurement (~> 7.6.0)
```

当我们运行 `pod update` 或 `pod install` 时，CocoaPods 会自动生成 `Podfile.lock`，我们不应该手动更新它，因为它可能导致冲突和问题。

### Pods 目录

`Pods` 目录是一个包含 CocoaPods 为我们的项目安装的所有依赖项的目录。它包括每个 pod 的源代码、头文件和编译的二进制文件。

### Podspec

`Podspec` 是一个文件，它描述了一个单独的 pod，包括其名称、版本、源代码位置、依赖项和其他元数据。Podspecs 发布到 CocoaPods 仓库，并由 CocoaPods 用于下载和安装 pod。

让我们看看 podspec 文件的例子：

```swift
Pod::Spec.new do |s|  s.name         = "MyLibrary"
  s.version      = "1.0.0"
  s.summary      = "A library for iOS and macOS
      development."
  s.description  = "MyLibrary provides a set of tools and
      utilities for iOS and macOS development."
  s.homepage     = "https://github.com/
      myusername/MyLibrary"
  s.license      = "MIT"
  s.author       = { "My Name" => "myemail@example.com" }
  s.platform     = :ios, '14.0'
  s.source       = { :git => "https://github.com/
      myusername/MyLibrary.git", :tag => "#{s.version}" }
  s.source_files = "Sources/**/*.{h,m,swift}"
  s.swift_version = '5.4'
  s.dependency   "Alamofire", "~> 5.4"
  s.dependency   "SwiftyJSON", "~> 4.0"
end
```

podspec 文件是用 Ruby 编写的，Ruby 是一种动态的面向对象的编程语言。实际上，`Podfile` 文件也是用 Ruby 编写的，因为它被认为是编写框架配置文件的方便且流行的方式。

在这个例子中，podspec 文件描述了一个名为 `MyLibrary` 的库，版本为 `1.0.0`。我们还可以看到摘要、描述和其他一般细节，例如主页、许可证和作者。

在源文件和源代码中，我们可以看到 Git 仓库的位置（在这个例子中，在 GitHub 上）以及 pod 中包含的文件在 **通配符模式** 下的位置。

通配符模式

在 Podfile 中，可以使用通配符模式来定义 pod 应该安装的目录。

通配符模式（“*****”）代表任何字符以匹配任何文件或目录。例如，如果我们想指定所有以“M”开头的目录，我们可以使用以下命令：

**Pod "MyPod", :path => "****M*"**

通配符模式不是 Podfile 特有的模式——它在许多类 Unix 命令行工具和终端中都有使用。

最后，我们添加了 pod 所需的依赖项，这样 CocoaPods 就会知道相应地管理其依赖关系树。

重要的是要注意，我们可以轻松地使用一个 podspec 文件来创建一个本地库并将其集成到我们的项目中。以下是一个本地库的 podspec 文件示例：

```swift
Pod::Spec.new do |s|  s.name         = "MyFramework"
  s.version      = "1.0.0"
  s.summary      = "A framework that modularizes code from
      MyProject."
  s.homepage     = "https://github.com/
      myusername/MyFramework"
  s.license      = "MIT"
  s.author       = { "My Name" => "myemail@example.com" }
  s.platform     = :ios, '14.0'
  s.source       = { :path => "." }
  s.source_files = "MyFramework/**/*.{h,m,swift}"
  s.public_header_files = "MyFramework/**/*.h"
  s.frameworks   = "UIKit"
  s.dependency   "Alamofire", "~> 5.4"
  s.dependency   "SwiftyJSON", "~> 4.0"
  s.swift_version = '5.4'
  s.pod_target_xcconfig = {
    'SWIFT_INCLUDE_PATHS' => '$(SRCROOT)
        /MyProject/MyModule'
  }
end
```

除了修改我们的`source`和`source_files`属性外，我们还可以看到我们现在有一个`pod_target_xcconfig`属性来指定项目中模块的源代码路径。

`Podfile`在这种情况下将看起来像这样：

```swift
platform :ios, '14.0'target 'MyApp' do
  use_frameworks!
  pod 'MyFramework', :path => '../MyFramework'
end
```

在`Podfile`中，我们指导`MyFramework`到其本地路径，其中存在源文件。

### Pod 命令行工具

pod 命令行工具是处理 CocoaPods 的主要接口。它提供了一套命令，允许我们安装、更新和管理项目的依赖项。

这些是我们可以使用的一些常用命令：

+   **pod install**：这个命令安装**Podfile**中指定的依赖项并生成一个 Xcode 工作空间，其中包含我们的项目和已安装的依赖项。

+   **pod update**：这个命令将**Podfile**中指定的依赖项更新到最新版本并安装。我们可以选择更新特定的库或一系列库。

+   **pod lib create**：这个命令生成一个新的 CocoaPods 库模板。我们可以用它快速设置新 pod 的目录结构、文件和配置。

+   **pod search**：这个命令在 CocoaPods 仓库中搜索与给定查询匹配的库。我们可以通过名称、描述、作者或其他标准来搜索库。

就像许多开发工具一样，CocoaPods 基于命令行工具、配置文件和终端，因此在 CocoaPods 的情况下不建议有“终端恐惧症”。

我们讨论了五个不同的 CocoaPods 组件，用于管理和理解 CocoaPods。如果你以前从未使用过 CocoaPods 或者没有创建过你的 pod，创建一个新的项目并尝试使用它是一个好主意，然后阅读 CocoaPods 文档，它清晰且直接。我保证，一个小时后你会更容易理解。

现在，让我们跳入两个可能在面试中遇到的问题，关于 CocoaPods 的。

## “你在使用 CocoaPods 时遵循哪些最佳实践，为什么它们很重要？”

*为什么这个问题很重要？*

CocoaPods 或任何其他依赖关系管理器都是项目的一个基本组成部分。

在某种程度上，我们甚至可以说这是我们的项目中的**脆弱部分**，原因有几个：

+   **这不是我们自己编写的代码**：CocoaPods 将其他开发者编写的数千行代码集成到我们的项目中。我们对添加他人编写的代码对安全性、性能和稳定性产生的影响几乎没有控制权。然而，这些代码可能已经经过多次修复和测试周期。

+   **存在依赖项冲突的潜在风险**：一个结构不良的**Podfile**可能导致不同版本的依赖项之间发生冲突，并可能损害项目的稳定性。

+   **过时的框架可能导致安全漏洞**：开发者经常锁定框架到特定版本以保持项目稳定性高。这可能导致具有安全和稳定性问题的过时代码。

由于我刚才提到的这些原因，一个结构良好的`Podfile`极大地影响我们的项目质量。

*答案是什么？*

当我们使用 CocoaPods 时，可以遵循一些最佳实践：

+   **保持我们的框架更新**：我们需要确保我们获得最新的错误修复和功能，以保持项目稳定。我之前提到的`~>`运算符建议用于获取最新的热修复和次要版本。

+   **理解语义版本控制**：保持框架更新至关重要。然而，我们应该了解语义版本控制是如何工作的。我们希望确保向后兼容性并避免破坏性更改。

+   **保持依赖项的最小化**：我们应该只包含我们真正需要的框架，并移除那些 Apple SDK 可以替换的框架。保持我们的项目轻量级和简单至关重要，避免潜在的冲突和崩溃问题。

+   **保持 Podfile 的整洁和可读性**：我们应该将我们的**Podfile**视为代码库的一部分。以逻辑方式分组依赖项可以为我们提供灵活性和清晰度。在此处的一个最佳实践是在每个库旁边添加注释并解释我们为什么添加每个库。库可能在我们应用中存在多年，我们添加的注释可以帮助我们在未来。

+   **实现适配器模式**：虽然适配器模式并非 CocoaPods 独有，但它是一个将库无缝集成到我们项目中的优秀模式。通常，库的接口与我们的现有代码库并不自然地匹配。通过引入一个充当我们的代码库和库之间接口的类，我们可以有效地连接这两个组件。此外，适配器还可以帮助我们解耦代码库和库，减少依赖。

我们必须记住一个重要的事情——库是我们代码的一部分。我们应该像管理我们的项目代码库一样关注第三方框架，因为它们极大地影响我们项目的性能。

## “pod update 和 pod install 有什么区别？”

*这个* *问题为什么重要？*

在上一个问题中，我们讨论了管理 CocoaPods 的最佳实践。我们讨论的一个关键方面是管理 pods，以防止它们引起任何问题或破坏我们的代码。

`pod update` 和 `pod install` 命令都帮助我们决定更新 pods 的策略，实现一种谨慎和负责任的方式来保持第三方库的更新。

*答案是什么？*

`pod install` 和 `pod update` 是在 iOS 项目的 CocoaPods 依赖管理器中使用的命令。它们之间的主要区别在于它们处理依赖解析的方式。

`pod install` 安装 `Podfile.lock` 中指定的依赖项，并确保 pod 不会收到意外的更新。

另一方面，`pod update` 将当前 pods 更新到最新的次要版本，并最终将 `Podfile.lock` 更新到新版本。

有一个需要注意的事项是，如果没有 `Podfile.lock`，*这两个命令* *表现相似*。

在日常工作中，这两个命令之间的区别至关重要。`Podfile.lock` 帮助我们控制框架的更新，并保持 pods 在特定版本，只要我们使用 `pod install`，而 `pod update` 可以绕过 `Podfile.lock` 中写的内容。

我建议在讨论最佳实践时将 `Podfile.lock` 包含在项目的代码库中。这在与团队协作时尤其重要，因为 `Podfile.lock` 确保所有团队成员使用相同的框架版本。

总结本节，到现在为止，我们应该了解 CocoaPods 的工作原理以及如何使用它来链接第三方库并维护一个稳定和健壮的项目。

作为管理 iOS 项目依赖项的基本工具，CocoaPods 应该被给予极高的重视。这适用于任何依赖管理器，包括我们将要审查的 Swift 包管理器。

# 了解 Swift 包管理器

在过去几年中，CocoaPods 和 Carthage 在管理 iOS 项目的依赖项方面发挥了重要作用。

虽然 CocoaPods 和 Carthage 是出色的工具，但每个平台都需要有自己的内部依赖管理器，苹果确实开发了一个名为 **Swift 包管理器**（**SPM**）的本地依赖管理器。

那么，SPM 是什么？

SPM 是一个直接集成到 Xcode 中的依赖管理器，允许开发者轻松创建、管理和共享 Swift 包，这些是包含在代码中的自包含单元，可以在不同的项目中使用。一个包可以包含一个或多个目标，每个目标都是一个模块，可以被其他包或项目导入和使用。

让我们从零开始创建一个 Swift 包。

## 创建一个 Swift 包

创建一个新的 Swift 包很简单。有两种方法可以做到这一点——使用 *终端* 和 *Xcode*：

+   **使用终端**：打开 **终端** 应用，进入项目文件夹（或任何其他文件夹），并输入以下命令：

    ```swift
    swift package init --type library
    ```

此命令将创建一个包含相关子文件夹和文件的新的文件夹，以设置一个基本且空的 Swift 包。

+   **使用 Xcode 创建 Swift 包**: 如果您不想使用终端来创建 Swift 包，另一个选项是使用 Xcode。

让我们回顾创建新的 Swift 包所需的步骤：

1.  打开 Xcode 并从菜单栏选择 **文件** | **新建** | **包**。

1.  在 **创建新的 Swift 包** 对话框中，输入包详细信息，例如包名、组织机构和类型。您可以选择库或可执行包，并指定包的平台和产品。

1.  点击 **创建** 以创建新的 Swift 包。Xcode 将生成一个包含 **Sources** 目录和 **Package.swift** 清单文件的基本项目结构。

这两种选项都很简单直观，只需几秒钟即可完成。现在，让我们了解我们创建了什么。

## 检查包清单和文件夹

Swift 包由三个组件组成：

+   **源** **目录**: 此目录包含我们包的 Swift 源文件。默认情况下，它包括一个与包同名的子目录和一个与目录同名的单个源文件。

+   **测试** **目录**: 此目录包含我们代码的测试文件。默认情况下，它包括一个单个子目录，一个与目录同名的单个测试文件。

+   **Package.swift**: 此文件是包清单文件。它包含有关包的信息，例如其名称、版本和依赖项，并由 SPM 用于构建和管理包。

在 *精通 CocoaPods* 部分，我们讨论了 CocoaPods 并提到了 `podspec` 文件。在 SPM 中，`package.swift` 文件与 CocoaPods 中的 `podspec` 文件具有类似的作用。

这里是一个标准 `package.swift` 文件内容的简短示例：

```swift
// Package.swiftimport PackageDescription
let package = Package(
    name: "MyPackage",
    platforms: [
        .macOS(.v10_12), .iOS(.v10), .watchOS(.v3), .tvOS(.v10)
    ],
    products: [
        .library(name: "MyPackage", targets: ["MyPackage"])
    ],
    dependencies: [
        .package(url: "https://github.com/
            Alamofire/Alamofire.git", from: "5.0.0")
    ],
    targets: [
        .target(name: "MyPackage", dependencies: ["Alamofire"]),
        .testTarget(name: "MyPackageTests", dependencies:
            ["MyPackage"])
    ]
)
```

我们可以看到 `package.swift` 文件是用，嗯…… *Swift* 编写的。因此，对于 iOS 开发者来说阅读它应该是简单的。让我们了解它说了什么。

此 `Package.swift` 文件指定了名为 `"MyPackage"` 的 Swift 包的详细信息。该包针对多个平台 - macOS、iOS、watchOS 和 tvOS。它提供了一个与包同名的单个库产品。

该包依赖于 `Alamofire` 包，指定为具有最小版本 `5.0.0` 的依赖项。该包还包含两个目标，一个用于包本身，另一个用于其测试。包目标依赖于 `Alamofire` 包，而测试目标依赖于 `MyPackage` 目标。

当 SPM 安装我们的包时，它也会建立我们在 `package.swift` 文件中定义的依赖项，就像它与 CocoaPods 一样工作。

另一个我们应该了解的有趣的事情是为什么它被称为“包”而不是“库”。原因是 Swift 包可以包含 *多个库*（在“产品”下）和不同的目标，理解这个层次结构对于创建灵活的包至关重要。

我们如何构建和测试包？让我们看看。

## Swift 包常见命令

虽然可以在 Xcode 中使用 SPM，但了解主要的终端命令仍然至关重要。这是因为用户界面倾向于更频繁地变化，而命令行工具则随着时间的推移保持更一致。

然而，一个更重要的原因是，终端命令为我们提供了将它们集成到脚本和 CI 机器中的能力，使它们变得强大而有效。

这里是命令列表：

+   **swift package init**：我们之前在讨论 Swift 包创建时看到了这个命令。它将在当前目录中初始化一个新的空 Swift 包。

+   **swift package update**：这会将包的依赖项更新到最新兼容版本。它与 CocoaPods 中的 **pod update** 类似。

+   **swift build**：这将构建包及其依赖项，生成二进制或库产品。

+   **swift test**：如果需要，这将运行包的单元测试，并构建包。

+   **swift package clean**：这将删除包的构建工件，包括构建目录。

在面试的背景下，将这个命令列表视为一个“功能列表”。这个列表应该显示我们如何操作和维护一个 Swift 包。

## 使用 Swift 包

使用 Swift 包库就像我们添加到项目中的任何其他库一样。我们关于访问级别的所有知识也适用于这种情况。

例如，一个应用只能访问*公共*和*开放*的功能、类和属性，而*内部*级别则保留给库。

此外，为了使用库，我们需要将其导入到我们的代码中。以下是一个示例：

```swift
import MyPackagelet myObject = MyClass()
myObject.myMethod()
```

在这个例子中，我们首先使用 `import` 语句将 `MyPackage` 模块导入到我们的代码中。然后，我们创建 `MyClass` 类的一个实例并调用其 `myMethod` 函数。在这种情况下，`MyClass` 是 `MyPackage` 的一部分。这个例子适用于处理依赖项时的应用和包。

一般而言，使用 SPM 非常简单直接。苹果在保持其能够在终端命令中执行一切功能的同时，出色地将它集成到 Xcode 中。

这里的目标是简要解释 SPM，然后我们再继续下一个面试问题。

## “与 CocoaPods 相比，使用 SPM 的优缺点是什么？”

**为什么这个问题很重要**？

这两个工具都非常适合管理 iOS 中的依赖项，但就像任何其他工具一样，它们都有其优点和缺点。

理解这些工具的实际差异可能甚至比使用它们更重要，因为后者是直接和技术的。选择正确的工具对我们的应用维护和稳定性有重大影响。

**答案是什么**？

就像任何利弊一样，答案取决于项目的需求和需求。然而，CocoaPods 和 SPM 之间有一些已知的不同之处。

这些是 *SPM 的优点*：

+   Swift 项目的内置工具，因此无需第三方依赖

+   简单且 *易于使用* 的语法

+   *与 Swift 和 Xcode 集成*，包括生成 Xcode 项目的支持

+   *自动依赖解析* 和缓存

+   *更快的* 构建时间，适用于更小的项目

这些是 *SPM 的缺点*：

+   对二进制依赖的支持有限

+   不支持 Objective-C 或混合语言项目

+   构建设置的定制选项有限

这个列表表明，SPM 的优缺点与之前在 *掌握 CocoaPods* 部分中提到的相反。

一方面，CocoaPods 比 SPM 更常用且更灵活。另一方面，它更复杂、更慢，在 Apple 的开发生态系统中表现得像一个外来者。

在开始面试之前，你应该尝试每个解决方案，以了解它们的感觉和可能性。

## “有哪些最佳实践可以用来组织和结构化 Swift 包以优化构建时间和最小化依赖冲突？”

*为什么这个问题很重要？*

这个问题超出了技术范畴。正如我们之前看到的，处理 Swift 包的技术部分很简单，即使是初级开发者也能轻松应对。然而，有效地组织和高效地管理项目以适应包，才是真正的挑战。

关于代码模块化和组织的大量文章和研究证明了这个问题的重要性。其中大部分甚至没有提到 SPM。

即使遵循这里提到的最佳实践的很小一部分，也能显著影响我们的项目。

*答案是什么？*

有一些最佳实践可以用来组织我们的代码以优化构建时间和解耦我们的依赖：

+   **保持包小**：包含许多依赖的大型包可能会减慢构建时间并增加冲突的风险。为了最小化这些问题，保持包小和模块化是一个好的实践。这使得管理依赖变得更容易，并减少了冲突的风险。

+   **最小化依赖**：为了减少冲突的风险并提高构建时间，尽可能最小化依赖是一个好的实践。这可以通过仅使用包的必需依赖项并避免不必要的或重复的依赖项来实现。

+   **使用语义版本控制**：我们可以使用语义版本控制来管理包及其依赖的版本号。通过使用语义版本控制，我们可以向其他开发者和包的用户传达更改和兼容性要求。

+   **使用增量构建**：SPM 支持增量构建，这意味着只有当进行更改时，才会重新构建包的必要部分。这有助于提高构建时间并减少不必要的重新编译。

我们可以说，这里描述的最佳实践可以用来模块化任何项目，甚至任何类。扁平的层次结构、最小化依赖和小的库都是我们在项目中管理库的好建议。

# 摘要

依赖管理器就像双刃剑。无论是 SPM 还是 CocoaPods，它们都可以在模块化和分离方面为我们节省时间，成为我们项目的优秀补充。然而，如果处理不当，它们也可能对我们的应用程序的稳定性和架构的简洁性产生破坏性的影响。这就是为什么作为 iOS 开发者，我们必须掌握这个话题。

在本章中，我们学习了 CocoaPods 和 SPM 的基础知识，包括最佳实践及其优缺点。到目前为止，当我们被问到关于 iOS 最常见的第三方依赖管理器时，我们应该已经涵盖了所有内容。

从某种意义上说，我们的下一章与本章所讨论的内容紧密相连。我们将逐渐从标准的 Swift 和 UIKit 主题转向设计和架构的世界。

我们的下一章将会非常精彩！
