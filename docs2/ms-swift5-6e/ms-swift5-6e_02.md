

# Swift 文档和安装 Swift

我在职业生涯中大部分时间都在担任 Linux 系统管理员和网络安全管理员。这些职位要求我从源代码编译和安装软件包。与下载预构建的二进制文件相比，从源代码构建软件包有很多优点。在我看来，最大的优点是你可以获取最新版本，而无需等待其他人构建它。这使我能够及时对我的系统进行最新的安全更新。使用 Swift，我们也能够下载最新的代码并自行编译，而无需等待其他人构建它。

在本章中，你将学习：

+   关于 swift.org 网站及其提供的内容

+   如何找到 Swift 的最新文档

+   安装 Swift 的方法

+   如何从源代码构建 Swift，包括其完整的工具链和包管理器

在上一章中，我们提到苹果公司已将 Swift 作为开源项目发布，并设立了 [swift.org](http://swift.org) 网站专门服务于 Swift 社区。这意味着我们可以下载 Swift 语言的源代码，检查它，并自行构建 Swift。在我们真正深入 Swift 语言并使用它进行开发之前，让我们看看如何从源代码构建 Swift 以及苹果为我们提供的资源。我们将首先查看 [swift.org](http://swift.org) 网站。

# Swift.org

2015 年 12 月 3 日，苹果公司正式发布了 Swift 语言，包括支持库、调试器和包管理器，并将其以 Apache 2.0 许可证的形式开源给社区。当时，创建了 [swift.org](http://swift.org) 网站，作为社区进入项目的门户。这个网站拥有丰富的信息，应该是你了解 Swift 社区和语言本身发生情况的首选网站。博客文章会更新 Swift 的新版本发布、新的 Swift 开源库、标准库的变更以及其他 Swift 新闻。

你还可以下载适用于多种 Linux 版本的预构建二进制文件。在本书编写时，我们可以下载适用于 Ubuntu 16.04、Ubuntu 18.04、Ubuntu 20.04、CentOS 8 和 Amazon Linux 2 的预构建二进制文件。入门页面提供了之前提到的 Linux 版本的依赖项列表以及如何安装二进制文件的说明。

网站还包括官方的 Swift 文档，其中包括语言指南、Swift 简介以及 API 设计指南。理解 API 设计指南对于确保你的代码符合推荐的编码标准至关重要。在 *第十八章*，*Swift 格式化和风格指南* 中，我们提供了 Swift 编码标准的建议，这些建议与苹果公司的建议相辅相成。

你还可以找到有关如何为 Swift 社区做出贡献的信息，Swift 源代码的下载位置，甚至还有一个关于推荐 Google Summer of Code 项目的部分。如果你真的想深入研究 Swift 开发，无论是服务器端、Mac 还是 iOS 开发，我建议定期访问 [swift.org](http://swift.org) 网站，以了解 Swift 社区正在发生的事情。

苹果和 Swift 社区也提供了一些你可以用作参考的文档资源。

# Swift 文档

苹果和 Swift 社区作为一个整体，已经发布了许多资源来帮助开发者用 Swift 编程。可以在 [`developer.apple.com/documentation/`](https://developer.apple.com/documentation/) 找到的苹果官方文档，包括 Swift 的 API 文档以及苹果的所有框架。苹果的框架中只有一小部分是开源的，并且可以在所有平台上工作；然而，如果你想要开始使用苹果的某个框架，这绝对是一个开始的地方。然而，找到特定 Swift API 的文档可能有些困难。

要快速找到 Swift API 的文档，我最喜欢的网站是 [`swiftdoc.org`](https://swiftdoc.org)。这个网站导航起来非常方便，并为所有类型、协议、运算符和组成 Swift 语言的全局函数自动生成了文档。我注意到这个网站并不总是保持最新；然而，它对于任何 Swift 开发者来说都是一个极好的参考资料。生成这个网站的代码也是开源的，可以在 GitHub 上找到：[`github.com/SwiftDocOrg/swift-doc`](https://github.com/SwiftDocOrg/swift-doc)。GitHub 页面提供了如何生成你自己的离线文档的说明。

我接下来要提到的最后一个网站是我最近在我最喜欢的 Swift 网站上发现的。它是位于 [`www.hackingwithswift.com/example-code/`](https://www.hackingwithswift.com/example-code/) 的 Hacking with Swift 网站上的 Swift 知识库。一旦你学会了 Swift，需要知道如何进行 JSON 解析、提取 PDF 或任何其他特定功能，你在这里很可能找到你需要的东西。

现在我们知道了在哪里查找 Swift 的文档，让我们看看我们可以用哪些不同的方式来安装 Swift。

# 从 swift.org 安装 Swift

如果你正在为 Apple 平台开发和编程，我强烈建议你使用 Xcode 中的 Swift 版本。苹果不会批准使用与 Xcode 版本不同的 Swift 版本编译的应用程序。这听起来可能有些极端，但它确保了应用程序是用稳定的 Swift 版本编译的，并且这个版本已经经过全面审查，可以与你的 Xcode 版本一起工作。

如果你使用的是在 [swift.org](http://swift.org) 网站上有预构建二进制文件的 Linux 发行版，建议你使用这些二进制文件。它们是让 Swift 运行起来最快、最简单的方法。你还可以在 [swift.org](http://swift.org) 网站的“入门”部分找到完整的安装说明和依赖项列表。

如果你使用的 Linux 发行版没有提供预构建的二进制文件，如果你想尝试 Swift 的最新版本，或者你只是想看看从源代码构建 Swift 是什么样的体验，你也可以这样做。

# 从源代码构建 Swift 和 Swift 工具链

有许多网站展示了如何从源代码构建 Swift，但遗憾的是，这些网站中的大多数只提供了构建 Swift 语言本身的说明，而没有工具链。我发现这并不太有用，除非你只编写非常简单的应用程序。在我看来，在没有整个工具链和包管理器的情况下为 Linux 构建 Swift 更多的是一种“我能做到吗”的练习，而不是构建可以长期使用的软件。

虽然不建议在生产系统中使用 Swift 的最新构建版本，但它确实使我们能够使用语言的最新功能，并验证引入到我们的应用程序中的更改与 Swift 语言的未来版本兼容。

在本章中，我们将探讨如何从源代码构建 Swift、其整个工具链以及 Swift 包管理器。由于每个 Linux 发行版和 macOS 都有所不同，我需要选择一个平台来编写这些说明；因此，我使用了 Ubuntu 18.04 和本书编写时的当前 Swift 5.3 开发版本。对于其他发行版或版本，你可能需要更改已安装的依赖项或它们的安装方式。构建 Swift 本身的命令将在所有平台上都是相同的。

这些命令中的一些可能相当长，难以手动输入。所有的命令都包含在本书可下载的代码中的文本文件里，你可以从中剪切和粘贴。

现在，让我们开始构建 Swift，通过安装依赖项来启动这个过程。

## 安装依赖项

我们首先需要确保我们拥有所有必需的依赖项。以下命令包括了我在不同 Linux 发行版上需要安装的依赖项。

你可能会发现你的发行版已经预装了一些这些依赖项，但为了确保你拥有所有内容，请运行以下命令：

```swift
sudo apt-get install git cmake ninja-build clang python uuid-dev libicu-dev icu-devtools libedit-dev libxml2-dev libsqlite3-dev swig libpython-dev python-six libncurses5-dev pkg-config libcurl4-openssl-dev systemtap-sdt-dev tzdata rsync 
```

需要注意的一点是依赖项可能会发生变化，如果你尝试从源代码编译并且收到一个错误信息，表明某些东西缺失，你可以使用 `apt-get install` 命令或系统上的包管理器来添加它。现在我们已经安装了所有依赖项，我们需要下载 Swift 源代码。

## Swift 源代码

要下载 Swift 源代码，我们需要创建一个新的目录，将代码下载到该目录，然后运行 `git` 命令来检索源代码。以下命令将 Swift 源代码下载到名为 `swift-source` 的目录：

```swift
mkdir swift-source
cd swift-source
git clone https://github.com/apple/swift.git
./swift/utils/update-checkout --clone 
```

现在我们已经获取了源代码并克隆了我们需要的部分，接下来让我们构建 Swift。

## 构建 Swift

在您开始构建 Swift 之前，您需要了解这取决于您的系统或虚拟机设置，可能需要数小时才能构建完成。如果您正在使用虚拟机，例如 VirtualBox，我强烈建议您为虚拟机分配多个核心；这将显著缩短构建时间。以下命令将构建 Swift、其工具链和包管理器：

```swift
./swift/utils/build-script --preset=buildbot_swiftpm_linux_platform,tools=RA,stdlib=RA 
```

一旦构建完成所有内容，我们需要将其安装到某个位置，并将二进制文件放入我们的路径中。

## 安装 Swift

现在我们已经构建了 Swift 和其工具链，我们已准备好安装它。我喜欢在 `/opt` 目录下安装 Swift，其他人更喜欢在 `/usr/local/share` 目录下安装。您将其放在哪个目录下完全取决于您。如果您想将其安装在其他位置，只需将路径中的 `/opt` 替换为您希望安装到的目录即可。

让我们从更改到 `/opt` 目录并创建一个名为 `swift` 的新目录开始安装过程。我们需要更改此目录的权限，以便我们可以读取、写入和执行文件。以下命令将执行此操作：

```swift
cd /opt
sudo mkdir swift
sudo chmod 777 swift 
```

命令 `chmod 777 swift` 为此计算机的所有用户添加了读取、写入和执行权限。我喜欢使用这种模式，因为这样任何系统用户都可以使用 Swift；然而，这也可以被认为是一个安全问题，因为它也意味着任何人都可以修改文件。使用此模式时请自行承担风险，对于生产系统，我强烈建议您查看直接需要此权限的用户，并对其进行更严格的限制。

接下来，我们需要将构建的 Swift 二进制文件移动到 `swift` 目录。为此，我们将更改到 `swift` 目录，为我们的构建创建一个新的目录，更改到该目录，并复制文件。以下命令将执行此操作：

```swift
cd swift
mkdir swift-5.3-dev
cd swift-5.3-dev
cp -R ~/swift-source/build/buildbot_incremental/toolchain-linux-x86_64/* ./ 
```

现在我们想要创建一个指向此目录的符号链接，名为 `swift-current`。这样做的原因是它允许我们向我们的 `PATH` 环境变量中添加一个条目，这样操作系统就可以在不需要我们输入完整路径的情况下找到 Swift 可执行文件。如果我们使用 `swift-current` 路径而不是 `swift.5.3-dev` 路径来设置此条目，那么在安装新的 Swift 版本时，我们可以简单地更改 `swift-current` 符号链接指向的位置，从而使一切正常工作。我们可以使用以下命令从 `/opt/swift` 目录中执行此操作：

```swift
sudo ln -s /opt/swift/swift-5.3-dev swift-current 
```

现在我们需要在我们的 `PATH` 变量中创建条目。为此，我们将 `/opt/swift/swift-current/usr/bin/` 目录添加到位于你主目录中的 `.profile` 文件中的 `PATH` 变量中。然后更新环境。以下命令会这样做：

```swift
cd ~
echo 'PATH=$PATH:/opt/swift/swift-current/usr/bin/' >> .profile
source ~/.profile 
```

我们现在应该已经安装了 Swift 并准备好使用。我们最后想要做的是测试我们的安装。

## 测试安装

我们最后需要做的是验证 Swift 是否已成功安装。为此，我们可以运行以下命令：

```swift
swift --version 
```

输出应该看起来像这样，但带有你安装的 Swift 版本：

```swift
Apple Swift version 5.3-dev (LLVM a60975d8a4, Swift afe134eb2e)
Target: x86_64-unknown-linux-gnu 
```

如果输出看起来像这样，那么恭喜你，Swift 已经成功安装到你的系统中。如果存在问题，并且你的系统无法找到 `swift` 命令，那么问题可能出在路径上。你首先想要做的是打印出你的 `PATH` 变量来验证 `/opt/swift/swift-current/usr/bin/` 是否在你的路径中。以下命令会这样做：

```swift
echo $PATH 
```

如果 `/opt/swift/swift-current/usr/bin/` 不在你的路径中，你可以尝试手动运行 `PATH=$PATH:/opt/swift/swift-current/usr/bin/` 命令，而不是将其添加到你的 `.profile` 文件中。

最后，让我们验证包管理器是否也正确安装。为此，我们想在你的主目录下创建一个 `swift-code` 目录，并创建一个新的包，如下面的命令所示：

```swift
cd ~
mkdir swift-code
cd swift-code/
mkdir test
cd test
swift package init --type=executable 
```

输出应该看起来像这样：

```swift
Creating executable package: test
Creating Package.swift
Creating README.md
Creating .gitignore
Creating Sources/
Creating Sources/test/main.swift
Creating Tests/
Creating Tests/LinuxMain.swift
Creating Tests/testTests/
Creating Tests/testTests/testTests.swift
Creating Tests/testTests/XCTestManifests.swift 
```

我们现在已经准备好使用 Swift 和包管理器。

# 使用 Swift 包管理器

你可以用包管理器做很多事情，这使得它在 Linux 平台上创建复杂的应用程序成为必需品。它帮助向项目中添加依赖项，并使我们能够将代码拆分成多个文件并创建库项目。你同样可以在 Mac 平台上使用包管理器，但我发现使用 Xcode 更容易。

对于这本书中的示例，我们不需要添加依赖项或使用多个文件。让我们看看我们如何在包管理器中简单地构建和运行一个可执行项目，这样你就可以使用它来运行这本书中的示例。记住，你同样可以在 Apple 平台上使用包管理器。当包管理器在 `Sources/test/` 目录中创建 `main.swift` 文件时，它向其中添加了以下代码：

```swift
print("Hello, world!") 
```

这段代码给我们提供了一个基本的 `Hello World` 应用程序。当你阅读这本书时，你可以用这本书中的示例替换这段代码。为了了解我们如何使用包管理器构建和运行应用程序，让我们现在保持代码不变，并运行以下命令：

```swift
swift build
swift run 
```

你应该看到以下输出：

```swift
Hello, world! 
```

`swift build` 命令编译了我们的应用程序，而 `swift run` 命令运行了由前一个命令构建的可执行文件。本书中的大部分代码不需要包管理器来运行，可能直接使用编译器会更简单。但请记住，对于比简单示例更大的项目，你将希望使用包管理器或 Xcode。

接下来，让我们看看如何使用 Swift 编译器。

# 使用 Swift 编译器

Swift 编译器是构建 Swift 代码的基本实用工具，它被包管理器、Xcode 和任何其他将 Swift 代码构建为可执行文件的工具使用。我们也可以自己调用它。要了解如何自己调用它，创建一个名为 `hello.swift` 的文件，并添加如下所示的 `print("Hello, world!")` 代码：

```swift
echo `print("Hello, world!")` >> hello.swift 
```

现在，我们可以使用以下命令编译此代码，该命令调用 Swift 编译器：

```swift
swiftc hello.swift 
```

最后，我们可以像运行其他任何可执行文件一样执行新创建的应用程序：

```swift
./hello 
```

我们将看到我们的 `Hello, world!` 消息。

# 概述

在本章中，我们探讨了苹果公司和 Swift 社区提供的一些不同文档。当你学习 Swift 时，这些文档可能是必不可少的，而且在你掌握了这门语言之后，它们也可以作为参考。我们还探讨了如何构建和安装 Swift 及其完整的工具链。虽然不建议在生产系统中使用 Swift 的最新构建版本，但我通常会在虚拟机或我的桌面设置中保留一个最近的构建版本。这使我能够使用语言的最新功能，并且可以运行我的代码以检查我是否引入了与语言未来版本不兼容的更改。

在下一章中，我们将开始深入了解这门语言本身，我们将看到如何在 Swift 中使用变量和常量。我们还将探讨各种数据类型以及如何在 Swift 中使用运算符。
