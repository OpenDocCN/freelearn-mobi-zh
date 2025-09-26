# <st c="0">1</st>

# <st c="2">iOS 18 新增功能</st>

<st c="22">苹果在 2024 年的 WWDC 上推出了 iOS 18，作为其年度开发者大会的一部分，与 macOS、tvOS、iPadOS、watchOS 和 visionOS 一起。</st>

<st c="159">利用我们应用在每个主要操作系统版本中的最新特性和功能，使我们获得竞争优势。</st> <st c="272">以下是苹果选择在 SDK 中改进特定领域的原因——市场研究或技术趋势是采用新技术的充分理由。</st>

<st c="439">然而，要理解 iOS 18 的改进之处，我们首先必须了解这个版本背后的背景——这是本章的一个目标。</st>

<st c="578">在本章中，我们将涵盖以下主题：</st> <st c="614">：</st>

+   <st c="631">理解 iOS</st> <st c="650">18 背景</st>

+   <st c="663">探索</st> <st c="674">Swift 测试</st>

+   <st c="687">了解新的 Swift</st> <st c="717">数据改进</st>

+   <st c="734">尝试新的</st> <st c="750">缩放过渡效果</st>

+   <st c="765">在我们的</st> <st c="799">iPad 应用中添加浮动标签栏</st>

+   <st c="808">在 SwiftUI 中对滚动视图有更多控制</st> <st c="847"></st>

+   <st c="857">更改文本渲染行为</st>

+   <st c="894">从另一个视图定位子视图</st>

+   <st c="934">进入</st> <st c="948">人工智能革命</st>

<st c="961">如果这听起来像是一个令人兴奋的章节，您并没有错。</st> <st c="1022">让我们先了解</st> <st c="1069">iOS 18 的背景。</st>

# <st c="1076">技术要求</st>

<st c="1099">对于本章，从 App Store 下载 Xcode 版本 16.0 或更高版本是必不可少的。</st>

<st c="1193">确保您正在使用最新的 macOS 版本（Ventura 或更高版本）。</st> <st c="1279">只需在 App Store 中搜索 Xcode，选择最新版本，然后继续下载。</st> <st c="1377">打开 Xcode 并完成出现的任何进一步设置说明。</st> <st c="1445">在 Xcode 完全运行后，您</st> <st c="1491">就可以开始了。</st>

<st c="1501">本章包含许多代码示例，可以在以下</st> <st c="1578">GitHub 仓库中找到：</st>

[<st c="1596">https://github.com/PacktPublishing/Mastering-iOS-18-Development/tree/main/Chapter%201</st>](https://github.com/PacktPublishing/Mastering-iOS-18-Development/tree/main/Chapter%201)

# <st c="1682">理解 iOS 18 背景</st>

<st c="1714">发布一个主要的 iOS 版本总是一件大事，即使它是第 18</st><st c="1786">个</st> <st c="1789">版本。</st> <st c="1799">让我们在 iOS 18 之前尝试分析 iOS SDK：</st>

+   **<st c="1846">SwiftUI</st>** <st c="1854">正在变得更加成熟和强大。</st> <st c="1878">然而，一些功能，如复杂的动画或过渡、手势处理、导航和绘图，仍然使用 SwiftUI 实现起来具有挑战性</st> <st c="2034">。</st>

+   **<st c="2048">Core Data</st>** <st c="2058">是大多数 iOS 开发者作为存储</st> <st c="2105">持久数据的解决方案的首选框架。</st>

+   <st c="2150">虽然</st> **<st c="2157">XCTest</st>** <st c="2163">被认为是一个强大且方便的测试框架，但它缺乏在其他平台上常见的功能，例如参数化测试和更好的</st> <st c="2328">测试组织。</st>

+   **<st c="2349">WidgetKit</st>**<st c="2359">的流行证明了在当今世界，能够一目了然地展示信息的能力至关重要。</st>

<st c="2458">没有人会否认这个列表的重要性。</st> <st c="2505">然而，一个苹果直到 WWDC 2024 才开始关注的至关重要的话题是</st> <st c="2579">人工智能。</st>

<st c="2603">OpenAI 的 ChatGPT 的兴起，随后是数千种机器学习和 AI 工具，使苹果陷入了一个尴尬的境地。</st> <st c="2722">这不是苹果第一次落后于一些暂时性的趋势，但这一次情况不同。</st> <st c="2825">AI 对人类潜在的影响表明，这不仅仅是一个常规趋势或技术演变；实际上，这是一场将改变</st> <st c="2978">世界的革命。</st>

<st c="2988">问题是，苹果在它的一系列平台和技术方面处于什么位置？</st> <st c="3065">它对 AI 革命有什么答案吗？</st>

<st c="3109">在深入探讨那个问题之前，让我们首先回顾一下 iOS 18 中引入的新特性和框架，并探讨最新版本如何应对我们在 iOS 开发中面临的一些关键挑战。</st> <st c="3314">不过，不用担心——我们将在最后一节和整本书中涵盖 AI 革命。</st> <st c="3412">现在，让我们讨论一个新的框架——</st> <st c="3449">Swift 测试。</st>

# <st c="3463">介绍 Swift 测试</st>

<st c="3489">Swift 测试框架</st> <st c="3522">采用了一种新颖且令人耳目一新的测试方法。</st> <st c="3570">Swift 测试框架包含现代特性，如宏，这些宏与结构体而不是类一起工作，并且可以标记测试和</st> <st c="3690">测试套件。</st>

<st c="3702">Swift 测试框架旨在取代 2013 年作为 Xcode 5 部分引入的 XCTest。</st> <st c="3797">XCTest 属于 Objective-C 为主流语言的旧时代。</st> <st c="3871">然而，Swift 取代了 Objective-C，苹果意识到 iOS 开发者需要一个现代化的</st> <st c="3954">测试框架。</st>

<st c="3972">这是一个简单的</st> <st c="3989">测试函数：</st>

[PRE0]

<st c="4309">我们可以看到在 Swift Testing 中编写测试函数是多么简单。</st> <st c="4381">注意前面的 Swift 宏，它除了提供测试描述外，还配置并标记了函数为关键。</st>

<st c="4512">如果你的应用没有测试函数，Swift Testing</st> <st c="4568">是一个很好的开始方式（要了解更多关于 Swift Testing 的信息，请参阅</st> [*<st c="4634">第 14 章</st>*](B21795_14.xhtml#_idTextAnchor430)<st c="4644">）。</st>

<st c="4647">现在，让我们讨论另一个处理我们</st> <st c="4706">持久存储</st>的新框架。</st>

# <st c="4723">介绍 Swift 数据改进</st>

<st c="4759">Swift Data 是在 WWDC 2023 作为 iOS 17 的一部分引入的，其目标是取代旧但流行的 Core</st> <st c="4785">Data 框架。</st>

<st c="4886">Swift Data 提供了一个基于 Swift 的现代 API，可以帮助减少与持久存储工作时的摩擦。</st> <st c="5004">我们在 Apple 开发工具中看到的一个趋势是，从 GUI 转向基于代码的工具。</st> <st c="5101">一个很好的例子是 SwiftUI – 尽管可以拖放组件来构建用户界面，但主要的实现方式是代码。</st> <st c="5250">App Intents 和 Swift Package Managers 也是如此。</st> <st c="5308">数据层也遵循同样的概念 – 在 Swift Data 中，我们没有数据模型编辑器，所以我们使用</st> <st c="5440">仅代码来构建数据模型。</st>

<st c="5450">例如，这是为</st> `<st c="5504">Book</st>` <st c="5508">实体创建数据模型的方法：</st>

[PRE1]

<st c="5599">乍一看，这似乎是一个普通的</st> `<st c="5641">Book</st>` <st c="5645">类 – 确实如此！</st> <st c="5665">这次，我们添加了</st> `<st c="5689">@Model</st>` <st c="5695">宏，它完成了所有的魔法。</st>

<st c="5729">当 Swift Data 介绍时，它已经拥有许多功能，例如关系和删除规则。</st> <st c="5834">尽管如此，许多开发者认为这个框架还不够成熟，无法取代</st> <st c="5920">Core Data。</st>

<st c="5930">在 iOS 18 中，Apple 为 Swift Data 添加了一些功能，如果它还没有，这将使其更接近它</st> <st c="6049">应该达到的地方。</st>

## <st c="6059">唯一值</st>

<st c="6072">iOS 18 的第一个也许是最重要的</st> <st c="6107">新特性</st> <st c="6119">是能够根据</st> **<st c="6160">模型属性</st>** <st c="6172">构建一个</st> **<st c="6160">唯一值</st>** <st c="6172">：</st>

[PRE2]

final class CSVStoreConfiguration: DataStoreConfiguration {

    typealias Store = CSVDataStore

    var name: String

    var schema: Schema? var fileURL: URL

    init(name: String, schema: Schema? = nil, fileURL: URL)

{

        self.name = name

        self.schema = schema

        self.fileURL = fileURL

    }

    static func == (lhs: CSVStoreConfiguration, rhs:

    CSVStoreConfiguration) -> Bool {

        return lhs.name == rhs.name

    }

    func hash(into hasher: inout Hasher) {

        hasher.combine(name)

    }

}

[PRE3]

最终类 <st c="8350">CSV数据存储</st>: 数据存储 {

    typealias 配置 = <st c="8403">CSV存储配置</st> typealias 快照 = 默认快照

    var 配置: <st c="8481">CSV存储配置</st> var 名称: String

    var 架构: 架构

    var 标识符: String

    必须初始化(_ 配置: CSV存储配置,

    迁移计划: (任何 SchemaMigrationPlan 类型)?)

    抛出 {

        self.configuration = configuration

        self.name = configuration.name

        self.schema = configuration.schema! self.identifier =

        配置文件URL的最后路径组件

    }

}

[PRE4]

func 获取<T>(_ 请求: 数据存储获取请求<T>)

抛出 -> 数据存储获取结果<T，默认快照>

    where T : 持久模型 {

        let 判断 = 请求描述符判断

        返回 数据存储获取结果(描述符:

        请求描述符，获取快照: [],

        相关快照: [:])

        . // 执行获取操作

    }

    func 保存(_ 请求:

    数据存储保存更改请求<默认快照>)

    抛出 -> 数据存储保存更改结果<默认快照>

    {

        var remappedIdentifiers = [持久标识符:

        持久标识符]()

        对于请求插入的快照 {

            // 插入新项目

        }

        对于请求更新的快照 {

            // 更新现有项目

        }

        对于请求删除的快照 {

            // 删除项目

        }

        返回

        数据存储保存更改结果<默认快照>(for:

        self.identifier,

        remappedIdentifiers: remappedIdentifiers)

    }

[PRE5]

<st c="11285">@Namespace() var 命名空间</st> var body: some View {

        NavigationStack {

            ScrollView {

                LazyVGrid(columns: [

                网格项(.自适应(minimum: 150)) ]) {

                    对于专辑专辑 {专辑 in

                        NavigationLink {

                            Image(album.imageName)

                                .可调整大小() <st c="11512">.导航过渡(.缩放(sourceID:专辑ID，在：</st>

<st c="11563">命名空间)</st> } 标签: {

                            Image(album.imageName)

                                .可调整大小()

                                .scaledToFit()

                                .frame(minWidth: 0,

                                最大宽度: .infinity)

                                .frame(height: 150)

                                .cornerRadius(8.0)

                        } <st c="11720">.matchedTransitionSource(id:</st>

<st c="11748">专辑ID，在：命名空间中</st> }

                }

            }

        }

        .padding()

    }

[PRE6]

结构体 ContentView: View {

    var body: some View {

        TabView {

            Tab("主页",系统图像:"house.fill") {  }

            Tab("个人资料",系统图像:

            "person.crop.circle") { }

            Tab("设置",系统图像:"gear") { }

        }

        .tint(.red) <st c="14402">.tabViewStyle(.sidebarAdaptable)</st> }

}

[PRE7]

ScrollViewReader {代理 in

            ScrollView {

                VStack(alignment: .leading, spacing: 16) {

                    ForEach(albums) {专辑 in

                        提取视图(album:专辑)

                            .id(album.id)

                    }

                }

            }

            .overlay(alignment: .bottom) {

                如果显示滚动到顶部 {

                    Button("滚动到顶部") {

                            代理滚动到(albums[0].id,

                            锚点: .top)

                    }

                    .buttonStyle(.borderedProminent)

                }

            } <st c="17458">.onScrollGeometryChange(for: Bool.self) {</st>

<st c="17499">几何形状在</st>

<st c="17511">几何内容偏移量.y <</st>

<st c="17538">几何内容内边距底部 + 300</st>

<st c="17574">} 动作：{旧值，新值 in</st>

<st c="17608">withAnimation {</st>

<st c="17624">显示滚动到顶部 = !newValue</st>

<st c="17652">}</st>

<st c="17654">}</st> }

[PRE8]

对于专辑 {专辑 in

    ExtractedView(album: album)

        .id(album.id)

        .<st c="19395">onScrollVisibilityChange(threshold: 0.9) {</st>

<st c="19438">visible in</st>

<st c="19449">if visible {</st>

<st c="19462">print("\(album.title) appears")</st>

<st c="19494">}</st>

<st c="19496">}</st> }

[PRE9]

<st c="21400">struct CustomTextRenderer: TextRenderer {</st>

<st c="21442">func draw(layout: Text.Layout, in ctx: inout</st>

<st c="21487">GraphicsContext) {</st>

<st c="21506">for (index, line) in layout.enumerated() {</st>

<st c="21549">ctx.opacity = Double(index + 1) * 0.1</st>

<st c="21587">ctx.rotate(by: Angle(degrees: Double(index) *</st>

<st c="21633">1))</st>

<st c="21637">ctx.draw(line)</st>

<st c="21652">}</st>

<st c="21654">}</st>

<st c="21656">}</st> struct ContentView: View {

    var body: some View {

        Text("SwiftUI中的文本新增了众多新特性")

            .font(.system(size: 60)) <st c="21784">.textRenderer(CustomTextRenderer())</st> }

}

[PRE10]

struct NewsView: View {

    var body: some View {

        Text("可再生能源领域取得重大突破：新型太阳能电池技术承诺提高30%的效率")

        Text("全球市场对突然加息作出反应：股市全面下跌")

        Text("历史性的和平协议达成：领导人签署协议结束长达数十年的冲突")

        Text("创新人工智能工具革新医疗保健：医生拥抱机器学习进行诊断")

        Text("自然灾害袭击：沿海城市发生大规模地震，救援行动正在进行")

    }

}

[PRE11]

struct ContentView: View {

    var body: some View {

        ScrollView {

            VStack {

                Text("最新头条")

                    .font(.title)

<st c="24170">Group(subviews: NewsView()) { collection in</st> if let firstHeadline = collection.first

                    {

                        firstHeadline

                            .font(.title2)

                        Spacer()

                    }

                    ForEach(collection.dropFirst()) {

                    newsItem in

                        newsItem

                            .font(.headline)

                        Spacer()

                    }

                }

            }

            .padding()

        }

    }

}

[PRE12]

Group(subviews: NewsView()) { collection in

[PRE13]
