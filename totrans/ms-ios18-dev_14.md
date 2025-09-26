# <st c="0">14</st>

# <st c="3">使用Swift Testing提升应用质量</st>

<st c="48">为什么测试是编程书籍的一部分呢？</st> <st c="87">测试不是</st> **<st c="113">质量保证</st>** <st c="130">(</st>**<st c="132">QA</st>**<st c="134">)** <st c="137">团队职责的一部分吗？</st>

<st c="150">你很快会发现，测试是我们作为iOS开发者开发周期和文化的一部分。</st> <st c="255">许多开发者认为测试是一项重要的任务，但他们没有时间去做。</st> <st c="335">不幸的是，他们最终会因bug和</st> <st c="389">长久的重构而付出代价。</st>

<st c="404">在本章中，我们将做以下事情：</st>

+   <st c="447">理解测试的重要性</st>

+   <st c="484">学习Xcode的测试</st> <st c="511">历史</st>

+   <st c="519">探索Swift Testing</st> <st c="546">框架的基本知识</st>

+   <st c="562">了解如何使用套件、测试计划和</st> <st c="619">方案</st>来管理测试

+   <st c="630">学习可以帮助我们维护</st> <st c="668">测试</st>的技巧

<st c="677">在本章结束时，你将准备好利用Swift Testing来提升你的测试技能。</st>

<st c="775">在我们回答</st> *<st c="797">如何</st>* <st c="800">问题之前，让我们先从</st> <st c="828">为什么</st> <st c="832">开始。</st>

# <st c="836">技术要求</st>

<st c="859">你必须从Apple的</st> <st c="936">App Store</st>下载16.0或更高版本的Xcode，用于本章。</st>

<st c="946">你还需要运行最新版本的macOS（Ventura或更高版本）。</st> <st c="1019">在App Store中搜索</st> `<st c="1030">Xcode</st>` <st c="1035">，选择并下载最新版本。</st> <st c="1098">启动Xcode，并遵循系统可能提示的任何附加安装说明。</st> <st c="1202">一旦Xcode完全启动，你就可以开始了。</st>

<st c="1252">本章包含许多代码示例，其中一些可以在以下GitHub</st> <st c="1346">仓库</st>中找到：[<st c="1358">https://github.com/PacktPublishing/Mastering-iOS-18-Development/tree/main/Chapter14</st>](https://github.com/PacktPublishing/Mastering-iOS-18-Development/tree/main/Chapter14)

# <st c="1441">理解测试的重要性</st>

<st c="1481">对于许多开发者来说，测试</st> <st c="1510">是他们编写代码时必须处理的一个不必要的开销。</st>

<st c="1576">这种思维方式在某种程度上是可以理解的。</st> <st c="1625">我们已经完成了代码的编写，构建了一个应用程序，并看到一切按预期运行。</st> <st c="1723">我们不需要移动到下一个任务，我们需要更改目标，添加一个测试函数，只是为了再次确认它运行良好。</st> <st c="1857">为什么要在上面浪费时间</st> <st c="1876">呢？</st>

<st c="1882">此外，在许多情况下，编写这些测试函数需要大量的工作。</st> <st c="1954">我们如何测试 SwiftUI 视图或网络调用呢？</st> <st c="2004">这甚至意味着什么？</st>

<st c="2027">这些总结说明了为什么测试不是一种常见的做法，或者至少</st> <st c="2098">不够。</st>

<st c="2109">这个问题的根源在于开发者如何处理测试和编写代码。</st> <st c="2199">测试不仅仅是检查我们的函数是否按预期运行；它还涉及到代码结构、关注点的分离、编写过程、工作文化和我们对待日常工作的方式。</st> <st c="2378">我们的日常工作的处理方式。</st>

<st c="2394">让我们看看</st> <st c="2413">以下函数：</st>

[PRE0]

<st c="2651">这个函数检查用户是否可以根据标准，如权限、列表类型和状态，将任务添加到特定的列表中。</st> <st c="2785">现在，假设我们需要确保这个函数能够正常工作。</st> <st c="2851">我们如何做到这一点呢？</st> <st c="2871">我们需要在不同的状态下运行我们的应用程序来查看</st> <st c="2924">结果吗？</st>

<st c="2936">我们都知道，确保我们的代码能够正确运行是我们开发过程的一部分。</st> <st c="3023">这是一个经典的例子，说明了编写测试用例和在应用程序的不同状态下运行可以简化我们的开发过程。</st> <st c="3148">当我们添加未来的任务时，我们理解测试为什么如此重要，比如</st> <st c="3214">重构和</st> <st c="3239">错误修复。</st>

<st c="3249">在我们深入研究 Swift 测试之前，让我们了解 Apple 平台上的测试历史。</st>

# <st c="3342">学习 Apple 平台上的测试历史</st>

<st c="3390">随着 Apple 开发工具</st> <st c="3417">多年的发展，测试工具</st> <st c="3459">也得到了</st> <st c="3465">发展。</st>

<st c="3480">第一个为 Apple 平台定制的测试框架</st> <st c="3518">是</st> **<st c="3543">SenTestingKit</st>**<st c="3556">，基于 OCUnit 开源框架。</st>

<st c="3600">SenTestingKit 于 2005 年推出，并集成到 Xcode 中，为编写和运行</st> <st c="3719">Objective-C 代码</st>提供了基本功能。

<st c="3736">2013 年，Apple 引入了</st> **<st c="3763">XCTest</st>**<st c="3769">，它采用了更现代的测试方法，具有更好的 Xcode 集成以及对 Objective-C</st> <st c="3876">和 Swift</st>的支持。

<st c="3886">让我们看看</st> *<st c="3922">理解测试的重要性</st>* <st c="3961">部分中的代码示例，并看看一个</st> <st c="3995">XCTest 测试</st>的例子：

[PRE1]

<st c="4345">在这个用户示例中，我们看到一个简单的</st> <st c="4383">测试函数，该函数测试用户是否可以将任务添加到受保护的列表中。</st>

<st c="4456">有几件事情</st> <st c="4480">值得注意：</st>

+   <st c="4493">测试函数是</st> `<st c="4527">CanUserAddTaskTests</st>` <st c="4546">类的一部分，继承自</st> `<st c="4573">XCTestCase</st>` <st c="4583">超类。</st>

+   <st c="4595">测试函数名称以</st> `<st c="4635">test</st>` <st c="4639">短语开头。</st> <st c="4648">该</st> `<st c="4652">test</st>` <st c="4656">短语表示 XCTest 框架，这是一个</st> <st c="4707">测试函数。</st>

+   <st c="4724">测试验证表达式是通过一个特定的函数（</st>`<st c="4788">XCTAssertFalse</st>`<st c="4803">）完成的，该函数检查特定的表达式是否</st> `<st c="4853">为假</st>`<st c="4858">。我们有一系列用于</st> <st c="4892">各种条件的函数。</st>

<st c="4911">虽然这些都是我们在 Xcode 中编写测试的一部分，但它们与现代 Swift/SwiftUI 方法不匹配——使用结构体、宏以及更简单和通用的 Swift 函数。</st> <st c="5106">这就是 Swift 测试出现的原因。</st>

<st c="5156">让我们一起来探索 Swift</st> <st c="5177">测试。</st>

# <st c="5194">探索 Swift 测试基础</st>

<st c="5229">我们将从将 Swift 测试框架添加到现有项目</st> <st c="5255">开始我们的旅程。</st>

<st c="5317">从 Xcode 的菜单中选择</st> **<st c="5325">文件</st>** <st c="5329">|</st> **<st c="5332">新建</st>** <st c="5335">|</st> **<st c="5338">目标</st>** <st c="5344">来完成此操作。</st> <st c="5379">然后，在模板选择器中，找到</st> **<st c="5417">单元测试包</st>** <st c="5436">并选择它（</st>*<st c="5452">图 14</st>**<st c="5462">.1</st>*<st c="5464">）：</st>

![图 14.1：新的目标模板选择器](img/B21795_14_1.jpg)

<st c="5636">图 14.1：新的目标模板选择器</st>

*<st c="5680">图 14</st>**<st c="5690">.1</st>* <st c="5692">显示了 Xcode 中的模板选择器窗口。</st> <st c="5737">在执行测试搜索时，**<st c="5775">单元测试包</st>** <st c="5794">很容易找到。</st> <st c="5814">请注意，我们还有一个</st> **<st c="5839">UI 测试包</st>** <st c="5856">模板。</st> <st c="5867">然而，在 Swift 测试中目前还不支持 UI 测试，所以现在我们将专注于</st> <st c="5947">单元测试。</st>

<st c="5960">我们如何进行 UI 测试？</st>

<st c="5991">UI 测试，也称为端到端测试，是应用测试中的一个不同</st> <st c="6014">主题。</st> <st c="6052">它也被我们称为“黑盒”测试，意味着测试函数不知道内部代码，只知道用户界面组件。</st> <st c="6075">进行 UI 测试的基本方法是使用 XCTest，苹果之前使用的测试框架。</st> <st c="6219">然而，有一些服务提供更简单或跨平台的远程运行 UI</st> <st c="6394">测试的方法。</st>

<st c="6409">一旦你选择了</st> **<st c="6430">单元测试包</st>** <st c="6449">模板，点击</st> **<st c="6464">下一步</st>**<st c="6468">。现在，我们需要填写一些关于我们新测试目标（</st>*<st c="6537">图14</st>**<st c="6547">.2</st>*<st c="6549">）的详细信息：</st>

![图14.2：为我们的新测试目标选择选项](img/B21795_14_2.jpg)

<st c="6806">图14.2：为我们的新测试目标选择选项</st>

<st c="6859">在</st> *<st c="6863">图14</st>**<st c="6872">.2</st>*<st c="6874">中，我们可以填写目标的名字、团队和包标识符。</st> <st c="6939">我们还可以在旧的XCTest和新Swift Testing框架之间进行选择。</st> <st c="7019">在这种情况下，我们将选择</st> **<st c="7046">Swift Testing</st>**<st c="7059">。</st>

<st c="7060">点击</st> **<st c="7065">保存</st>**<st c="7069">，恭喜你——你有一个新的</st> <st c="7101">测试目标</st> <st c="7108">了！</st>

<st c="7123">让我们编写我们的</st> <st c="7140">第一个测试！</st>

## <st c="7151">添加基本测试</st>

<st c="7171">我们的模板</st> <st c="7185">包含一个基本的、空的</st> <st c="7211">测试函数：</st>

[PRE2]

<st c="7395">尽管代码</st> <st c="7416">非常简洁，但我们可以看到与XCTest相比的一些变化：</st>

+   `<st c="7545">Testing</st>`<st c="7552">，并且我们应该将其导入到我们想要测试的每个文件中。</st> <st c="7602">要测试的每个文件中。</st>

+   `<st c="7698">XCTestCase</st>`<st c="7708">，我们在Swift Testing中使用结构体。</st> **<st c="7749">结构体</st>** <st c="7756">不仅更轻量级且易于使用，而且在尝试并行运行测试时也更有帮助。</st> <st c="7860">记住，结构体是值类型，这意味着每次我们传递一个结构体时，我们都会得到数据的一个副本。</st> <st c="7967">这有助于在测试时检查状态。</st>

+   `<st c="8031">@Test</st>``<st c="8081">@Test</st>` <st c="8086">宏，它帮助SwiftData框架管理</st> <st c="8137">其测试。</st>

+   `<st c="8159">#expect</st>``<st c="8197">XCTAssert</st>` <st c="8206">函数，我们使用</st> `<st c="8229">#expect</st>` <st c="8236">宏，这对于我们想要测试的任何表达式都很有帮助。</st> <st c="8288">要测试的表达式。</st>

<st c="8296">我们可以通过点击测试函数旁边的菱形按钮或按</st> *<st c="8393">⌘</st>**<st c="8394">U</st>*<st c="8395">来快速运行我们的测试。如果一切如预期进行，我们的测试应该通过。</st>

<st c="8451">现在，让我们用一些实际的测试来填充我们的代码。</st> <st c="8501">在我们的例子中，我们有一个处理计数器的视图模型。</st> <st c="8562">我们有</st> `<st c="8570">increment</st>` <st c="8579">和</st> `<st c="8584">decrement</st>` <st c="8593">函数以及一个</st> `<st c="8610">count</st>` <st c="8615">变量：</st>

[PRE3]

让我们使用 Swift Testing 测试`CounterViewModel`的功能。

我们需要做的第一件事是向 Swift Testing 提供对我们的应用目标的访问权限：

[PRE4]

我们将`@testable`属性添加到`import`命令中，以启用对内部实体的访问。

现在，让我们编写我们的第一个测试函数：

[PRE5]

在我们的测试函数中，我们初始化视图模型，调用其增加函数，并验证结果。如果`#expect`宏函数内的表达式为`false`，则测试失败。

这三个阶段——准备、执行和验证——是任何测试流程的一部分，无论我们使用 Swift Testing 还是任何其他测试框架。

现在，让我们将包含此测试的结构体（`CounterViewModelTests`）重命名，并运行我们的测试。

在 Xcode 中，我们可以通过标签页打开左侧面板（或者直接按**⌘6**），然后我们可以看到我们的测试列表（**图 14.3**）：

![**图 14.3**：Xcode 中列出的测试](img/B21795_14_3.jpg)

**图 14.3**：Xcode 中列出的测试

在**图 14.3**中，我们可以看到测试面板上我们的测试结构，这反映在我们创建结构体和测试函数的方式上。

在本章开头，我们通过检查一个简单的代码示例来讨论了 Swift Testing 和 Xcode 之间的区别。其中之一的变化是使用`@Test`宏。

除了指示一个测试函数，`@Test` 宏还有额外的功能帮助我们配置测试。

例如，让我们使用`@Test`宏为我们的测试函数提供一个名称。

## 为我们的测试函数提供名称

为测试函数提供有表达力和意义的名称至关重要，并且当我们在项目中拥有数百个测试时，这可能会很有价值。

在 XCTest 中，为了做到这一点，我们需要将测试函数重命名为类似以下的内容：

[PRE6]

<st c="11395">函数名称正确地描述了测试，但感觉笨拙且不自然，尤其是当我们有成百上千的</st> <st c="11517">测试函数时。</st>

<st c="11532">使用</st> `<st c="11542">@Test</st>` <st c="11547">Swift 宏，我们可以为每个测试提供一个可读的名称：</st>

[PRE7]

<st c="11699">将测试描述添加到</st> `<st c="11735">@Test</st>` <st c="11740">Swift 宏中使其更易于阅读，并且它还很好地与 Xcode 集成（</st>*<st c="11824">图 14</st>**<st c="11834">.4</st>*<st c="11836">）：</st>

![图 14.4：Xcode 中的测试面板，带有自定义名称](img/B21795_14_4.jpg)

<st c="11983">图 14.4：Xcode 中的测试面板，带有自定义名称</st>

*<st c="12038">图 14</st>**<st c="12048">.4</st>* <st c="12050">显示了之前相同的测试函数，现在有一个可读的并且</st> <st c="12115">有意义的名称。</st>

<st c="12131">The</st> `<st c="12136">@Test</st>` <st c="12141">Swift 宏不仅提供了为我们的函数命名。</st> <st c="12205">我们还可以用它来禁用和启用测试。</st> <st c="12253">让我们看看如何</st> <st c="12270">做到这一点。</st>

## <st c="12278">启用和禁用测试</st>

<st c="12307">有时，一个测试可能变得</st> <st c="12336">无关紧要，我们希望暂时将其从测试列表中删除。</st> <st c="12406">我们可以删除它或对其注释。</st> <st c="12441">然而，这些解决方案可能需要更舒适和实用的长期解决方案。</st> <st c="12529">因此，让我们使用</st> `<st c="12547">@Test</st>` <st c="12552">宏来使其更加优雅。</st>

<st c="12585">在 Swift 测试中，所有测试默认启用。</st> <st c="12638">要禁用特定测试，我们可以使用</st> `<st c="12681">disabled()</st>` <st c="12691">函数：</st>

[PRE8]

<st c="12808">我们可以看到</st> `<st c="12829">disabled()</st>` <st c="12839">函数现在是</st> `<st c="12867">@Test</st>` <st c="12872">参数之一。</st> <st c="12885">在这种情况下，测试函数将不会运行，我们还可以看到该函数现在在测试面板中已禁用（</st>*<st c="13000">图 14</st>**<st c="13010">.5</st>*<st c="13012">）：</st>

![图 14.5：测试面板中的禁用测试](img/B21795_14_5.jpg)

<st c="13159">图 14.5：测试面板中的禁用测试</st>

*<st c="13204">图 14</st>**<st c="13214">.5</st>* <st c="13216">显示了我们的测试函数变为灰色。</st> <st c="13253">在这种情况下，执行整个测试运行将跳过</st> <st c="13307">该测试。</st>

<st c="13317">然而，有些情况下我们需要我们的测试函数仅在特定条件下运行，例如当用户登录或处于特定的 A/B</st> <st c="13472">测试条件下。</st>

<st c="13487">在这种情况下，我们将在测试函数中实现条件作为保护语句，从而使测试函数成功。</st> <st c="13620">但这听起来不是一个好办法——当测试函数没有运行时，它成功。</st>

<st c="13719">幸运的是，基于特定标准的测试函数启用是 Swift 测试支持的功能。</st> <st c="13822">我们只需要在 `<st c="13879">@Test</st>` <st c="13884">宏头中添加 enabled 函数，包括一个</st> <st c="13909">布尔表达式：</st>

[PRE9]

extension Tag {

    @Tag static let critical: Self

}

[PRE10]

@Test("测试重置函数", <st c="16511">.tags(.critical))</st> func testResetFunction() {

[PRE11]

.tags(.critical, .calculations, .performance))

[PRE12]

struct UnitConverter {

    static func metersToYards(_ meters: Double) -> Double {

        return meters * 1.09361

    }

}

[PRE13]

struct UnitConverterTests {

    @Test func testConvertingMetersToYards_1meter() {

        #expect(UnitConverter.metersToYards(1.0) ==

        1.09361)

    }

    @Test func testConvertingMetersToYards_3_5meter() {

        #expect(UnitConverter.metersToYards(3.5) ==

        3.827635)

    }

}

[PRE14]

@Test func testConvertingMetersToYards () {

    #expect(UnitConverter.metersToYards(1.0) == 1.09361)

    #expect(UnitConverter.metersToYards(3.5) == 3.827635)

    }

[PRE15]

@Test(参数：<st c="20621">arguments</st>: [(1.0, 1.09361), (3.5, 3.827635)])

    func testConvertingMetersToYards(data: <st c="20708">(Double,</st>

<st c="20716">Double)</st>) {

        #expect(UnitConverter.metersToYards<st c="20764">(data.0) ==</st>

<st c="20776">data.1)</st> }

[PRE16]

let convertingTests: [(Double, Double)] = [(1.0, 1.09361),

                                        (3.5, 3.827635)]

struct UnitConverterTests {

    @Test(arguments: convertingTests)

    func testConvertingMetersToYards(data: (Double,

    Double)) {

        #expect(UnitConverter.metersToYards(data.0) ==

        data.1)

    }

}

[PRE17]

<st c="25905">struct UnitConverterTests</st> {

    @Test func testConvertingMetersToYards_1meter() {

        #expect(UnitConverter.metersToYards(1.0) ==

        1.09361)

    }

}

[PRE18]

<st c="26458">@Suite("单位转换器测试")</st> struct UnitConverterTests {

    @Test func testConvertingMetersToYards_1meter() {

        #expect(UnitConverter.metersToYards(1.0) ==

        1.09361}

}

[PRE19]

@Suite("单位转换器测试", .<st c="27352">tags(.critical)</st>)

struct UnitConverterTests {

}

[PRE20]

@Suite("单位转换器测试", .<st c="27703">disabled()</st>)

struct UnitConverterTests {

}

[PRE21]

<st c="27970">@Suite("单位转换器测试")</st> struct UnitConverterTests { <st c="28030">@Suite("从米到码")</st> struct FromMetersToYardsTests {

    // 我们的测试函数

    }

}

[PRE22]

class NumberFilter {

    var numbers: [Int] = []

    var filteredNumbers: [Int] = [] <st c="41914">func filterNumbers(predicate: (Int) -> Bool) {</st> self.filteredNumbers =

        self.numbers.filter(predicate)

    }

}

[PRE23]

func filterNumbers(_ numbers: [Int], predicate: (Int) -> Bool) -> [Int] {

    return numbers.filter(predicate)

}

[PRE24]

func processAndSaveData(_ input: String) -> Bool {

// 数据处理

let processedData = // <执行一些数据操作

    code>

// 数据保存

return databaseService.saveData(processedData)

}

[PRE25]

func processAndSaveData(_ input: String) -> Bool {

    // 数据处理

    let processedData = processData(input)

    // 数据保存

    return databaseService.saveData(processedData)

} <st c="44306">private func processData(_ input: String) -> String {</st>

<st c="44359">return input.reversed()</st> }

[PRE26]

class UserViewModel {

    private let networkService<st c="44933">: NetworkServiceProtocol</st> var user: User? init(networkService<st c="44994">: NetworkServiceProtocol)</st> {

        self.networkService = networkService

    }

    func fetchUserDetails(for userId: String, completion:

    @escaping () -> Void) {

        networkService.fetchUserDetails(for: userId) {

        [weak self] user in

            self?.user = user

            completion()

        }

    }

}

[PRE27]

class MockNetworkService: NetworkServiceProtocol {

    var userToReturn: User? func fetchUserDetails(for userId: String, completion:

    @escaping (User?) -> Void) {

        completion(userToReturn)

    }

}

[PRE28]
