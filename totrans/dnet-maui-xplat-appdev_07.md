# 6

# 基于依赖注入的软件设计

在介绍了.NET MAUI中的导航和Shell之后，我们已经为构建一个综合应用程序打下了基础。然而，我们目前正使用一个模拟数据服务，我们打算在本章中对其进行修改。在深入探讨之前，让我们首先回顾一下软件设计的最佳实践，从设计原则概述开始。稍后，我们将探讨如何在我们的应用程序中利用依赖注入。

软件设计原则和模式通常是软件设计最佳实践的骨架。这些原则提供了规则和指南，软件设计师在构建高效和清晰的设计结构时遵循这些规则和指南。它们在塑造软件设计过程中起着关键作用，因为它们规定了最有效的实践。设计模式是经验丰富的面向对象软件开发者采用的有效最佳实践。它们作为模板，旨在解决特定上下文中的重复性设计问题，提供可重用的解决方案，这些解决方案可以应用于软件设计中的常见问题。

**依赖注入（DI**）是一种软件设计模式和技巧，它确保一个类不依赖于其依赖项。它通过解耦对象的利用与其创建来实现这一点。这里的目的是创建一个更适应性强、模块化且易于调试和维护的系统。DI体现在**依赖倒置原则（DIP**）中，这是面向对象编程和设计中的五个SOLID原则之一。

在本章中，我们将探讨以下主题：

+   设计原则概述

+   实现依赖注入（DI）

+   替换模拟数据存储

DI是实现依赖倒置设计原则的方法，也称为DIP。DIP是SOLID设计原则之一，我们将学习如何将SOLID原则融入我们的设计过程。在本章的开头，我们将提供SOLID设计原则的概述，然后再深入讨论DI。

# 技术要求

要测试和调试本章的源代码，您需要在您的PC或Mac上安装Visual Studio 2022。有关详细信息，请参阅*第1章*，*开始使用.NET MAUI*中的*开发环境设置*部分。

本章的源代码可在以下GitHub分支中找到：[https://github.com/PacktPublishing/.NET-MAUI-Cross-Platform-Application-Development-Second-edition/tree/main/2nd/chapter06](https://github.com/PacktPublishing/.NET-MAUI-Cross-Platform-Application-Development-Second-edition/tree/8067aa8a0ea9a275b33e64adaae7b09f7414851b)。

要查看本章的源代码，我们可以使用以下命令：

[PRE0]

要了解更多关于本书源代码的信息，请参阅*第2章*，*构建我们的第一个.NET MAUI应用程序*中的*管理本书的源代码*部分。

# 设计原则概述

设计原则是高级指导方针，提供了关于设计考虑的有价值建议。这些原则可以提供基本指导，帮助你做出更好的设计决策。一些通用设计原则不仅适用于软件设计，也适用于其他设计领域。

在我们探讨软件开发中常用的设计原则（SOLID）之前，让我们先回顾一些通用设计原则。

## 探索设计原则的类型

设计原则涵盖了一个广泛的主题领域。因此，而不是深入细节，我将从我在开发过程中实施设计原则的经验中提供见解，对本书中讨论的原则提供一个简洁的概述。我们将从高级原则如 **DRY**、**KISS** 和 **YAGNI** 开始，然后逐步过渡到在软件开发中更常用的那些。在面向对象编程（**OOP**）的领域，最广泛使用的设计原则是 SOLID 原则。

### 重复即冗余 (DRY)

正如人们常说的，不要重复造轮子；我们应该努力重用现有的组件，而不是重新开发已经创建的内容。

### 简单至上，愚蠢 (KISS)

我们应该选择简单直接的方法，而不是在设计过程中引入不必要的复杂性。

### 你不会需要它 (YAGNI)

我们应该在需要时实现功能。在软件开发中，有一种趋势是使设计具有前瞻性。这可能会创造出实际上并不需要的东西，并增加解决方案的复杂性。

### SOLID 设计原则

SOLID 设计原则在软件开发中被广泛采用，并作为众多设计模式的高级指导方针。SOLID 是一个首字母缩略词，它包含了以下五个原则：

+   **单一职责原则 (SRP)**: 一个类应该只有一个职责。遵循这个设计原则，开发者应该只有一个理由去修改一个类。通过在实现过程中考虑这个原则，生成的代码更容易理解，并且更有效地适应不断变化的需求。

+   **开闭原则 (OCP)**: 类应该对扩展开放，但对修改封闭。这个原则背后的核心概念是在引入新功能时防止对现有代码造成破坏。

+   **里氏替换原则 (LSP)**: 如果父类型的对象可以在某个上下文中使用，那么具有子类型的对象也应该能够以相同的方式运行，而不会引起任何错误或中断。

+   **接口隔离原则 (ISP)**: 设计不应该实现它不使用的接口，并且一个类不应该被迫依赖于它不打算实现的方法。我们应该设计简洁简单的接口，而不是庞大复杂的接口。

+   **依赖倒置原则（DIP）**：这个原则强调软件模块的解耦。高级模块不应该直接依赖于低级模块。两者都应该依赖于抽象。抽象不应该依赖于细节。细节应该依赖于抽象。

设计原则是帮助我们做出更好设计决策的指南。然而，最终的责任在于我们，在实施过程中确定最合适的行动方案，而不是仅仅依赖于这些原则。

由于我们将专注于本章中DI的使用，请参阅*进一步阅读*部分以获取有关设计原则（SOLID）和设计模式的更多信息。

## 使用设计原则

讨论了各种设计原则后，让我分享一些在实践实施中获得的见解和经验教训。

在我们应用程序的模型中，我使用了来自Dominik Reichl的**KeePassLib**。在将其移植到.NET Standard的过程中，我修改了继承层次结构，如图*图6.1*所示：

![图6.1：Item、PwEntry和PwGroup的类图](img/B21554_06_01.png)

图6.1：Item、PwEntry和PwGroup的类图

在将KeePassLib移植到.NET Standard的过程中，我开发了一个抽象父类`Item`，用于组（`PwGroup`）和条目（`PwEntry`）。这种修改似乎违反了SOLID原则中的OCP。这种方法的理由源于我从过去实施中学到的一课。

在早期版本中，在1.2.3之前，我没有按照描述的方式实现**KPCLib**。相反，我直接使用了`PwGroup`和`PwEntry`，这需要分别处理组和条目。这导致了`ItemsPage`和`ItemsViewModel`的复杂性增加。这种方法最显著的影响是无法明确区分模型和视图模型。因此，我不得不在视图模型中直接使用**KeePassLib**来管理许多细节。然而，在引入`Item`抽象父类之后，我成功地隐藏了服务（`IDataStore`和`IUserService`）和`PassXYZLib`中的大部分复杂实现。这导致了视图和视图模型中任何依赖KeePassLib的代码的消除。

这种变化的灵感来自KISS原则，而不仅仅是遵循OCP。在考虑其他SOLID原则，如LSP和SRP时，这种修改显著提高了整体架构。重要的是要认识到，在实际工作中，各种设计原则之间可能会出现冲突。最终，我们的责任是做出明智的决定，而不是教条地遵循设计原则。最有效的设计决策通常源于从先前的失败中获得的见解。

回到我们的主要焦点，我们现在将讨论通过采用 SOLID 设计原则之一——依赖反转来增强设计。作为 SOLID 设计原则的一部分，依赖反转强调软件模块的分离，并提供了一些如何实现这一点的指导。其背后的基本概念是在可能的情况下优先依赖抽象。在实践中，DI 是一种经常用来实现依赖反转概念的技巧。

# 实现依赖注入（DI）

DI 是一种可以在 .NET MAUI 中使用的技巧。尽管这不是一个新颖的概念，但它已经在像 ASP.NET Core 和 Java Spring Framework 这样的后端框架中被广泛使用。DI 通过将对象的用法与其创建解耦，从而简化了依赖反转（DIP），消除了对对象的直接依赖。在我们的应用中，一旦我们分离了 `IDataStore` 接口的实现，我们就可以开始使用模拟实现，并随后替换为实际实现。

在 .NET MAUI 中，我们将称之为 MS.DI 的 `Microsoft.Extensions.DependencyInjection` 服务作为内置功能可供我们使用。

在 .NET 领域，除了 MS.DI 之外，还有许多可用的 DI 容器。其中一些替代方案，如 Autofac DI 容器和 Simple Injector DI 容器，与 MS.DI 相比提供了更强大的功能和灵活性。在这个时候，人们可能会想知道为什么我们选择 MS.DI 而不是其他强大且灵活的 DI 容器。在这种情况下，重新审视 KISS（保持简单）和 YAGNI（不要过早优化）原则是至关重要的。我们不应该选择一个更强大的解决方案，假设我们可能会在未来使用某些功能。相反，最简单、最有效的方法是利用我们已有的东西，而不需要任何额外的努力。

使用 MS.DI，我们可以避免引入额外的依赖。无论我们的意图如何，它已经包含在 .NET MAUI 的配置中。通过简单地添加几行代码，我们就可以增强我们的设计。其他 DI 容器可能提供更复杂的功能，但我们需要在我们的代码中包含额外的依赖，并在使用它们之前进行必要的配置。如果你正在处理复杂系统设计，建议评估可用的 DI 容器，并选择最适合你系统的那个。在我们的场景中，**PassXYZ.Vault** 是一个相对简单的应用，我们不会直接从 Autofac 或 Simple Injector 提供的先进 DI 功能中受益。MS.DI 提供的功能对于我们的实现来说是足够的。

在我们的应用中，我们旨在解耦的模块是来自 KeePass 提供的第三方库的模型层。如图 *图 6.2* 所示，我们的系统由三个不同的程序集组成：**KPCLib**、**PassXYZLib** 和 **PassXYZ.Vault**。

![计算机服务器的图  描述自动生成](img/B21554_06_02.png)

图 6.2：包图

**KPCLib** 包包含两个命名空间，**KeePassLib** 和 **KPCLib**。**PassXYZLib** 作为扩展包，使用 .NET MAUI 特定的实现来增强 **KPCLib** 包的功能。我们的应用程序 **PassXYZ.Vault** 直接依赖于 **PassXYZLib** 包，间接依赖于 **KPCLib** 包。根据 DI 原则，我们旨在建立对抽象的依赖，而不是对具体实现的依赖。为了实现这一点，我们设计了两个接口，`IDataStore` 和 `IUserService`，使我们能够从实际实现中解耦。

需要访问 KPCLib 和 PassXYZLib 的实际实现被封装在实现这两个接口的类中：`IDataStore` 和 `IUserService`。访问 **KPCLib** 和 **PassXYZLib** 中功能所需的其余代码可以利用这两个接口。关键点是我们始终有灵活性，在必要时替换 `IDataStore` 和 `IUserService` 的实现。剩余的代码不会受到这些更改的影响。

为了将 MS.DI 作为 DI 服务使用，涉及两个主要步骤：注册和解析。

初始时，我们必须在程序启动期间注册我们的接口（如 `IDataStore`）及其相应的实现（如 `MockDataStore`）。之后，我们可以在整个程序中利用这些已注册的接口，而无需手动创建它们。然后，我们可以使用 DI 容器解决这些已注册的依赖项。

`ServiceCollection` 类作为注册的手段，而 `ServiceProvider` 类则促进解析，如图 6.3 所示：

![图 6.3：MS.DI 的使用](img/B21554_06_03.png)

图 6.3：MS.DI 的使用

*图 6.3* 展示了 `ServiceCollection` 和 `ServiceProvider` 的简化类图，位于顶部。`ServiceCollection` 作为 `IServiceCollection` 接口的默认实现，而 `ServiceProvider` 则作为 `IServiceProvider` 接口的默认实现。

这些接口，`IServiceCollection` 和 `IServiceProvider`，允许我们注册和解析依赖。在图 6.3 的底部，有一个序列图说明了如何使用这两个接口来实现这一点。为了更清晰地理解，我们将使用 `IDataStore` 服务作为示例来解释 `IServiceCollection` 和 `IServiceProvider` 的使用。

为了使用 DI 实现 `IDataStore` 服务，我们可以遵循后续代码块中概述的步骤：

[PRE1]

**（1**）首先，我们必须创建一个实现 `IServiceCollection` 接口的 `ServiceCollection` 类的实例。

**(2)** `IServiceCollection` 接口本身并不指定任何方法。相反，在 MS.DI 命名空间中定义了一系列扩展方法。在这些方法中，可以使用 `AddSingleton` 扩展方法来注册实现 `IDataStore` 接口的具体 `MockDataStore` 类。`AddSingleton` 方法将在下一节中解释。此方法使用泛型类型来指定接口及其实现。此外，`AddSingleton` 扩展方法还有几个重载变体可供使用。

**(3)** 为了访问对象，我们可以通过调用与 `IServiceCollection` 关联的 `BuildServiceProvider` 扩展方法来获取 `ServiceProvider` 的实例。`ServiceProvider` 类符合 `IServiceProvider` 接口。值得注意的是，`IServiceProvider` 接口位于 `System` 命名空间中，并专门定义了 `GetService` 方法。其他方法被指定为扩展方法，可以在 `Microsoft.Extensions.DependencyInjection` 命名空间中找到，如图 *6.3* 所示。

**(4)** 一旦我们有了 `ServiceProvider` 的实例，我们可以使用 `GetRequiredService` 扩展方法解析 `IDataStore` 接口。

要管理服务的范围，我们可以在该范围内解析它如下：

[PRE2]

我们将在下一节中讨论范围。

尽管MS.DI是一个轻量级的依赖注入服务，但它为.NET MAUI应用程序提供了足够的功能，如下所述：

+   实例的终身管理

+   构造函数、方法和属性注入

在接下来的章节中，我们将更深入地探讨这些功能。

## 终身管理

当使用依赖注入（DI）时，你应该考虑注册服务的实例在销毁或创建新实例之前应该被重用或保留多长时间。终身管理对于定义服务实例生成和共享的范围至关重要。

在生命周期管理方面，以下是一些需要考虑的方面：

+   **资源管理**：资源，例如由服务使用的数据库连接，不应无限期地保持开启状态。例如，如果单例服务保持数据库连接开启，它将保留该资源直到应用程序的生命周期结束。

+   **性能**：每次需要服务时都创建一个新的服务实例可能并不总是最有效的方法，尤其是如果构建服务是资源密集型的。

+   **隔离**：如果你的服务需要隔离（例如，如果它维护某些状态），根据满足这种隔离要求的范围配置其生命周期是至关重要的。

+   **线程问题**：单例服务需要是线程安全的，因为它们在不同的请求之间共享，通常在单独的线程上并行处理。

使用 MS.DI，我们可以通过配置 `ServiceCollection` 来管理这些实例的生命周期。

依赖注入中通常有三种生命周期类型，我们可以使用扩展方法来配置它们：

+   **Singleton**：当首次请求服务时创建服务的单个实例，并在整个应用程序的生命周期内重用。所有调用者都接收相同的实例，这意味着单例服务表现得像一个共享的全局资源。单例服务可以保留状态，并且对于提供资源（如日志记录、缓存或配置）的集中式管理非常有用。可以使用扩展方法`AddSingleton`在整个应用程序的生命周期内创建单个实例。

+   **Scoped**：作用域服务为每个作用域创建一个新的实例，通常在Web应用程序中为每个请求创建。每个作用域都有一个服务的实例，该实例在该特定作用域内的所有组件之间共享。作用域服务适用于维护特定于单个请求或用户交互的状态，例如用户信息、请求详情或请求上下文中的数据库连接。可以使用扩展方法`AddScoped`创建一个实例并在定义的作用域内重用该实例。

+   **Transient**：每次请求服务时，`Transient`服务都会创建一个新的实例，确保每个调用者都得到一个唯一的实例，而不共享状态或资源。`Transient`服务适用于没有内部状态或资源管理需求的服务的服务。它们通常轻量级，不需要在不同组件之间共享。可以使用扩展方法`AddTransient`为每次调用创建一个实例。

[PRE3]

[PRE4]

在上述代码中，在标记为**（1）**的行中，我们将`IUserService`注册为`Singleton`对象，`IDataStore`注册为`Scoped`对象，`ItemsViewModel`注册为`Transient`对象。

在注册之后，在标记为**（2）**的行中，我们实例化了一个`ServiceProvider`并将其存储在`rootContainer`变量中。在标记为**（3）**的行中，利用`rootContainer`，我们生成了两个作用域，分别命名为**scope1**和**scope2**。

这些创建的对象的生命周期管理可以在*图6.4*中检查：

![软件应用程序的图示 自动生成描述](img/B21554_06_04.png)

图6.4：MS.DI中的生命周期管理

变量`userService`被创建为一个`Singleton`对象，确保只有一个实例存在，并且其生命周期与应用程序相同。两个作用域——即**scope1**和**scope2**——具有由我们的设计决定的独立生命周期。`Scoped`对象——**dataStore1**和**dataStore2**——与它们所属的作用域具有相同的作用域。同时，`ItemViewModel`的实例是`Transient`对象。

在三种方法——`AddSingleton`、`AddScoped`和`AddTransient`——中，已经定义了多种重载变体，以满足与`ServiceCollection`配置相关的各种需求。

在我们的应用程序中，我们有`IDataStore`接口实现的两个版本：

+   `DataStore`：这个版本代表实际的实现。

+   `MockDataStore`：这个版本用于测试目的。

使用MS.DI，我们能够在调试构建中使用`MockDataStore`，并在发布构建中使用`DataStore`。此配置可以按照后续代码片段中所示执行：

[PRE5]

## .NET MAUI配置DI

MS.DI被纳入.NET发布版，使其在.NET 5或后续版本的所有类型的应用程序中可用。正如我们在上一节中讨论的，我们可以使用`ServiceCollection`和`ServiceProvider`来实现DI。然而，在.NET MAUI中利用MS.DI有一个更直接的方法。由于DI作为.NET通用主机配置的一部分集成，我们无需手动创建`ServiceCollection`的实例。这使我们能够直接使用预配置的DI服务，无需任何额外的工作。

要深入了解.NET MAUI中预配置的DI服务，让我们回顾一下*图6.5*中所示的.NET MAUI应用程序启动过程。此图包含一个类图和一个时序图，展示了参与过程的类。

请注意，*图6.5*中的数字代表对象的类型（**(1)** = MauiProgram，**(2)** = MauiApp，和**(3)** = MauiAppBuilder）：

![程序图描述自动生成](img/B21554_06_05.png)

图6.5：.NET MAUI DI配置

在*图6.5*的顶部，我们可以看到.NET MAUI应用程序的初始化涉及四个不同的类：

+   **平台入口点**：.NET MAUI应用程序的初始化发生在特定平台的代码中。对于.NET MAUI项目，这可以在**Platforms**文件夹中找到。为每个平台定义了不同的类，如*表6.1*所示。在*图6.5*中，我们使用`MauiApplication`的Android版本作为代表示例。

    | **平台** | **入口点类** | **实现接口** |
    | --- | --- | --- |
    | Android | `MauiApplication` | `IPlatformApplication` |
    | iOS/macOS | `MauiUIApplicationDelegate` |
    | Windows | `MauiWinUIApplication` |

    表6.1：不同平台中的入口点

    所有入口点类都实现了`IPlatformApplication`接口，如下面的代码片段所示：

    [PRE6]

    `IPlatformApplication`接口定义了一个名为`Services`的属性，其类型为`IServiceProvider`。一旦应用程序初始化，这个属性就可以直接用来解析DI对象。

    所有平台入口点类也实现了一个重写方法，`CreateMauiApp`，它调用在`MauiProgram`类中定义的静态方法。请参考以下代码，*表6.1*和*图6.5*。

+   `MauiProgram` **(1)**: 在随后的 `MauiProgram` 实现代码中，可以明显看出，每个 .NET MAUI 应用程序都必须定义一个静态的 `MauiProgram` 类，并包含一个 `CreateMauiApp` 方法。`CreateMauiApp` 方法由所有平台入口点的重写函数调用。这个重写函数最终返回一个 `MauiApp` 实例：

    [PRE7]

+   `MauiApp` **(2)**: 在 `CreateMauiApp` 中，通过调用函数 `MauiApp.CreateBuilder` 创建一个 `MauiAppBuilder` 实例。

+   `MauiAppBuilder` **(3)**: `MauiAppBuilder` 包含一个名为 `Services` 的属性，该属性是 `IServiceCollection` 接口类型。这个属性允许我们为 .NET MAUI 应用程序配置依赖注入。

根据对 .NET MAUI 应用程序启动过程的上述分析，很明显，`IServiceCollection` 和 `IServiceProvider` 都在这个过程中被初始化。因此，我们可以方便地使用它们，而无需额外的配置。

下面的代码展示了 `MauiProgram` 的实现。在这里，我们注册了接口——`IDataStore` 和 `IUserService`——以及包括 `LoginService`、视图模型和页面在内的多个类。需要注意的是，所有这些组件都是单例对象，除了 `ItemsViewModel` 和 `ItemsPage`：

[PRE8]

在为接口和类配置了依赖注入之后，我们可以在实现中使用它们。`IServiceProvider` 接口使我们能够有效地解析对象。在实现依赖注入时，有三种主要的注入依赖的方法：构造函数注入、方法注入和属性注入。在接下来的章节中，我们将探讨如何将这些方法应用到我们的编程中。

## 构造函数注入

使用构造函数注入，一个类的必要依赖作为构造函数的参数提供，允许我们通过构造函数本身来解析依赖。在 `ItemsPage` 的代码背后，`ItemsPage` 依赖于其视图模型 `ItemsViewModel`。我们可以将 `ItemsPage` 的构造函数设置如下：

[PRE9]

在 `ItemsPage` 的构造函数中，我们通过参数 `viewModel` 注入依赖。在这个例子中，MS.DI 根据在 `MauiProgram` 中定义的配置解析 `viewModel`。

构造函数注入是最常见且最推荐的形式的依赖注入，因为对象总是带有所需的依赖项被创建。最大的优点是它使依赖项明确，对象永远不会处于不完整的状态。

通常情况下，可能无法通过构造函数注入依赖，例如当一个类包含可选依赖项或需要动态更改依赖项时。在这些情况下，使用方法注入或属性注入将是推荐的方法。

## 方法注入

与在对象实例化时使用构造函数注入来提供所需依赖项不同，方法注入直接将依赖项传递给使用它们的那些方法。方法注入是依赖注入（DI）技术中的一种，其中依赖项通过方法参数提供给对象。

在我们的代码中，我们可以通过方法而不是构造函数来设置依赖项，如下面的代码所示：

[PRE10]

在此示例中，`IDataStore`是通过`SetDataStore`方法设置的，而不是通过构造函数注入。然而，它有一个缺点，即依赖对象可能会忘记设置依赖项。结果，对象可能处于不完整的状态。

在提供的代码示例中，使用方法注入使我们能够在同一代码中利用实际的和模拟的DataStore实现。与构造函数注入相比，方法注入还可以提供更细粒度的控制，以创建、传递和销毁依赖项。然而，它也可能使方法的调用更加复杂，因为调用者负责以参数的形式提供依赖项。

通常，为了简单性和更好的封装性，构造函数注入更受欢迎，但方法注入在某些特定用例中可能非常有价值。

## 属性注入

在属性注入中，依赖项是通过属性设置的。通常，方法和属性注入可以互相替换。它与方法注入类似——在许多情况下，我们可能无法使用构造函数注入，因此我们必须使用方法或属性注入。方法或属性注入的问题在于，依赖对象可能会忘记设置依赖项，因此对象可能处于不完整的状态。

通过在面向对象的语言中使用属性或注解，可以部分缓解这个问题，我们将在稍后探讨这种方法。

在.NET MAUI中，我们可以通过`IServiceProvider`解决依赖项。在一个.NET MAUI应用程序中，托管环境为我们生成一个`IServiceProvider`接口，如图6.5所示。

要获取`IServiceProvider`接口，我们可以使用平台特定入口点中定义的`IPlatformApplication`接口，如图6.1所示：

[PRE11]

列表6.1: `ServiceHelper.cs` ([https://epa.ms/ServiceHelper6-1](https://epa.ms/ServiceHelper6-1))

**(1)** 在`ServiceHelper`类中，我们定义了一个名为`Current`的静态变量，它维护对`IServiceProvider`的引用。我们可以通过`IPlatformApplication`接口的`Services`属性来获取`IServiceProvider`。

**(2)** 定义了一个名为`GetService`的静态方法，该方法反过来调用`IServiceProvider`接口的`GetService`方法。

**ServiceHelper**

对于**ServiceHelper**的实现，我参考了**MauiApp-DI** GitHub项目。感谢James Montemagno在GitHub上提供的示例代码！

([https://github.com/jamesmontemagno/MauiApp-DI](https://github.com/jamesmontemagno/MauiApp-DI))

通过使用`ServiceHelper`类，我们可以获取`IDataStore`的一个实例，如下所示：

[PRE12]

你可能会注意到，之前提供的属性注入代码与构造函数注入相比显得不那么优雅。我们必须手动设置依赖项。在最坏的情况下，对象可能处于不完整的状态。

到目前为止，我还没有发现一种更有效的方法来实现.NET MAUI中的这一功能。然而，在本书的后续章节中，当我们介绍Blazor混合应用时，我们将能够更有效地利用C#属性来处理属性注入。在Blazor中解析`IDataStore`接口时，可以采用更直接的方法，以下是一个示例：

[PRE13]

我们可以使用`Inject` C#属性来隐式解析依赖，而不需要显式调用`ServiceHelper`中的`GetService`方法。

通过使用依赖注入（DI），我们可以无缝地将`IDataStore`接口的模拟实现替换为实际实现。这种实现可以简化密码数据库的CRUD操作。在下一节中，我们将更详细地探讨这个新类。

# 替换模拟数据存储

如前几节所述，我们可以在`MauiProgram.cs`中注册数据存储服务的实现，如下所示：

[PRE14]

[PRE15]

在这里，`DataStore`是`IDataStore`服务的实际实现，我们将在本章的剩余部分中完全实现它。

密码数据库是KeePass 2.x格式的本地数据库。在这个数据库中，密码信息被组织成组和条目。`KeePassLib`命名空间包含一个`PwDatabase`类，该类旨在管理数据库操作。

要理解`PwDatabase`、`PwGroup`和`PwEntry`之间的关系，我们可以参考*图6.6*中的类图：

![图6.6：KeePass数据库的类图](img/B21554_06_06.png)

图6.6：KeePass数据库的类图

在**PwDatabase**中，定义了类型为**PwGroup**的**RootGroup**属性，它包含数据库中存储的所有组和条目。可以从**RootGroup**导航到特定的条目，KeePass数据库的数据结构。**PwEntry**定义了一组标准字段，如图6.7所示：

![手机屏幕截图  描述由系统自动生成](img/B21554_06_07.png)

图6.7：组、条目和字段

如果我们有一个只包含标准字段的条目列表，它将类似于一个表格。在 *图 6.7* 中，当前组包含五个条目（**GitHub**、**Google**、**Facebook**、**Instagram** 和 **Chase Bank**）以及一个子组（**Cloud**）。在左侧，`ItemsPage` 的截图显示了当前组中的项目。如果选择 **Google** 项目，它将出现在右侧截图的条目中。用户可以选择向条目添加额外字段，这使得 KeePass 数据库不同于关系数据库；它更类似于键值数据库。每个字段都是一个键值对，例如 URL 字段。

为了在我们的应用程序中利用 `PwDatabase`，我们定义了一个派生类，称为 `PxDatabase`。这个类引入了额外的属性和方法，例如 `CurrentGroup`、`DeleteGroup`、`DeleteEntry` 等。

要访问数据库，可以打开数据库文件并在其上执行 CRUD 操作。然而，在构建跨平台应用程序时，直接处理数据库文件可能对最终用户来说不方便。在 **PassXYZ.Vault** 中，使用了用户的概念而不是使用数据文件。在 **PassXYZLib** 中，定义了一个 `User` 类来封装底层文件操作。

为了访问数据库，我们在 `IDataStore` 和 `IUserService` 接口中定义了数据库初始化和 CRUD 操作。`DataStore` 和 `UserService` 具体类用于实现这两个接口。

## 初始化数据库

数据库初始化被包含在登录过程中，因此后续的登录方法定义在 `IUserService` 接口中：

[PRE16]

`UserService` 类是 `IUserService` 接口的一个实现。在 `UserService` 类中，定义了 `LoginAsync` 方法，如下所示：

[PRE17]

在 `LoginAsync` 方法中，调用 `IDataStore` 的 `ConnectAsync` 方法来执行实际任务。下面我们来查看 `ConnectAsync` 的实现：

[PRE18]

在 `ConnectAsync` 函数中，**（1**），使用一个独立任务来管理数据库的打开过程。调用 `PxDatabase` 的 `Open` 方法**（2**），并将 `User` 类的实例作为参数传递给 `Open` 方法。

在成功建立连接并初始化数据库后，我们需要实现数据库操作所需的方法。这可以包括数据检索、数据插入、数据更新和数据删除等任务。这些代表在数据库系统中交互、管理和维护数据的基本操作。

## 执行 CRUD 操作

[PRE19]

当用户导航到另一个组时，`SetCurrentGroup` 方法被调用，并带有一个参数来设置导航中的新位置：

[PRE20]

列表 6.2: `IDataStore.cs` ([https://epa.ms/IDataStore6-2](https://epa.ms/IDataStore6-2))

### 添加项目

CRUD 的初始操作涉及创建或添加项目。这个项目可以是添加到当前组中的条目或组。执行此添加操作的用户界面可以在 `ItemsPage` 中的工具栏项中找到，如下所示：

[PRE21]

我们可以看到在 *图 6.8* 中的 `ItemsPage` 的右上角显示了工具栏项图标：

![图 6.8：添加项目](img/B21554_06_08.png)

图 6.8：添加项目

当点击 **+** 按钮时，通过数据绑定调用 `ItemsViewModel` 中的 `AddItemCommand` 命令。

`AddItemCommand` 命令在视图中调用以下 `AddItem` 方法：

[PRE22]

列表 6.3：`ItemsViewModel.cs` ([https://epa.ms/ItemsViewModel6-3](https://epa.ms/ItemsViewModel6-3))

**(1)** 在 `AddItem` 函数中，显示了一个 `ActionSheet`，允许用户选择项目类型。项目类型可以是组或条目。

**(2)** 获取项目类型后，我们可以构建一个包含项目类型和查询参数名称的字典。然后，我们将这个字典对象存储在名为 `itemType` 的变量中。

**(3)** 这个 `itemType` 变量可以作为查询参数传递给 `NewItemPage`。在 *第5章*，*使用 .NET MAUI Shell 和 NavigationPage 进行导航* 中，我们学习了如何将字符串值作为查询参数传递给 Shell 导航中的页面。在这里，我们可以在将其包装在字典中之后，将对象作为查询参数传递给页面。

要添加新项目，用户界面在 `NewItemPage` 中定义，而逻辑在 `NewItemViewModel` 中管理。让我们查看 *列表 6.4* 中显示的 `NewItemViewModel` 实现：

[PRE23]

列表 6.4：`NewItemViewModel.cs` ([https://epa.ms/NewItemViewModel6-4](https://epa.ms/NewItemViewModel6-4))

`NewItemPage` 的设计非常直接，包含两个控件：**Entry** 和 **Editor**，分别用于编辑项目的名称和备注。**Entry** 控件用于输入或编辑单行文本，而 **Editor** 控件用于修改多行文本。在 `NewItemViewModel` 视图模型中，我们可以观察到添加新项目的流程，如下所示：

**(1)** 使用 `QueryPropertyAttribute` 定义了查询参数。**(2)** 声明为 `ItemSubType` 的 `Type` 属性用于获取查询参数。随后，获取的项目类型存储在 `_type` 后备变量中。在 `NewItemPage` 中，创建了两个工具栏项，并将它们的行为关联到视图模型中找到的 **Save** 和 **Cancel** 方法。

在用户界面中输入名称和备注并点击 **Save** 按钮**（3**），使用定义在 `IDataStore` 接口中的 `CreateNewItem` 工厂方法创建一个新的项目实例。**（4**）一旦新项目实例填充了用户输入，就可以通过调用 `AddItemAsync` 方法将其添加到数据库中。

我们现在已经实现了添加操作。在下一节中，让我们继续实现剩余的数据操作。

### 编辑或删除项目

在CRUD操作中，创建操作不需要现有项目。但是，要执行更新和删除操作，需要现有项目的实例。

在读取操作中，当项目是一个组时，我们通过向`ItemsPage`发送一个`ItemId`查询参数来实现它，并在`ItemsViewModel`视图模型的`ItemId`设置器中识别该组。相反，如果项目是一个条目，我们向`ItemDetailPage`发送一个`ItemId`查询参数，并在`ItemDetailViewModel`的`ItemId`设置器中定位该条目。

对于更新、编辑和删除操作，我们可以利用上下文操作。这些操作允许我们有效地在`ListView`中操作项目。需要注意的是，上下文操作在不同平台上具有独特的外观，如图*6.9*所示：

![图6.9：上下文操作](img/B21554_06_09.png)

图6.9：上下文操作

在iOS平台上，你可以通过向左滑动项目来执行对项目的操作。在Android系统中，你可以通过长按项目来访问上下文操作菜单，该菜单将出现在屏幕的右上角。在Windows上，你可能熟悉通过右击鼠标来显示上下文操作菜单。

在我们的应用程序中，我们在`ItemsPage`中引入了一个上下文操作菜单。我们按照以下示例在`ItemsPage`内的`ItemViewCell`中配置上下文操作：

[PRE24]

`ItemViewCell`是一个继承自`KeyValueView`的自定义视图，该视图在*第4章*，*探索MVVM和数据绑定*中引入。让我们查看*列表6.5*中的`ItemViewCell`代码：

[PRE25]

列表6.5：`ItemViewCell.cs` ([https://epa.ms/ItemViewCell6-5](https://epa.ms/ItemViewCell6-5))

在`ItemViewCell`中，我们为编辑和删除上下文操作定义了两个菜单项。我们将两个事件处理器（**（1**）`OnEditAction`和**（2**）`OnDeleteAction`）分配给它们相应的上下文操作。在事件处理器中，我们调用视图模型方法`Update`和`Delete`来执行所需的操作。

让我们查看`ItemsViewModel`中`Update`和`Delete`函数的源代码，如下所示：

[PRE26]

在`ItemsViewModel`中，要编辑或更新项目，**（1**），我们使用一个名为`FieldEditPage`的内容页面来执行编辑过程。在调用`FieldEditPage`的构造函数时，一个匿名函数作为参数传递。当用户在`FieldEditPage`中完成编辑时，将调用此函数。在此函数中，**（2**），调用`IDataStore`接口的`UpdateItemAsync`方法来更新项目。

删除操作相当直接。我们可以简单地从 `IDataStore` 接口调用 `DeleteItemAsync` 方法 **(3**) 来消除该条目。

一旦实现了 CRUD 操作，我们的应用程序将具备密码管理应用程序所需的必要功能。我们可以通过注册新用户来创建一个新的数据库。在创建新数据库后，我们可以登录以访问我们的数据。此外，在生成条目和组之后，我们有能力根据需要修改或删除它们。

# 摘要

在本章中，我们首先介绍了设计原则。随后，我们深入探讨了 SOLID 设计原则，并分享了从我们应用程序开发中获得的见解。其中最重要的 SOLID 原则是 **DIP**。**DI** 是一种将 DIP 应用于实际实施的技术。在我们的应用程序中，我们利用 .NET MAUI 内置的 DI 服务来解耦依赖关系，使我们能够将服务的实现与接口分离。

我们积累了关于 .NET MAUI 的丰富知识，并通过用实际实现替换 `MockDataStore` 成功完成了我们的应用程序实现。我们在新的 `IDataStore` 服务之上建立了 CRUD 操作，从而实现了一个功能齐全的密码管理应用程序。

尽管我们在应用程序中集成了基本功能，但用户通常期望密码管理应用程序中具有额外的期望功能，例如指纹扫描和一次性密码。其中一些功能是平台特定的，这需要了解平台集成。在下一章中，我们将深入探讨各种平台集成主题，以进一步提高我们的应用程序。

# 进一步阅读

+   *《面向 ASP.NET 开发者的 SOLID 原则与设计模式入门》，作者 Bipin Joshi*

+   **Autofac** 是一个用于 .NET Core、ASP.NET Core、.NET 4.5.1+ 以及更多环境的 **控制反转**（**IoC**）容器：[https://autofac.org/](https://autofac.org/)

+   **Simple Injector** 是一个支持 .NET 4.5 和 .NET Standard 的 DI 容器：[https://simpleinjector.org/](https://simpleinjector.org/%20)

# 在 Discord 上了解更多

加入我们社区的 Discord 空间，与作者和其他读者进行讨论：

[https://packt.link/cross-platform-app](https://packt.link/cross-platform-app)

![](img/QR_Code166522361691420406.png)
