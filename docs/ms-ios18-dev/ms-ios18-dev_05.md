# 第六章：<st c="0">5</st>

# <st c="2">使用 WidgetKit 增强 iOS 应用程序</st>

<st c="43">随着 iPhone 这些年的发展，新增了功能以利用大屏幕、内存容量和</st> <st c="183">强大的处理器。</st>

<st c="202">这些功能之一是主屏幕小部件——扩展我们的应用程序并在</st> <st c="337">新位置提供信息和甚至交互的绝佳方式。</st>

<st c="348">在本章中，我们将涵盖以下主题：</st>

+   <st c="401">小部件的概念</st> <st c="411">是</st>

+   <st c="421">了解小部件是如何</st> <st c="440">工作的</st>

+   <st c="452">添加我们的第一个小部件并构建条目</st> <st c="495">时间线</st>

+   <st c="505">添加一个</st> <st c="512">用户可配置的小部件</st>

+   <st c="536">确保我们的小部件是最新的</st> <st c="563">的</st>

+   <st c="570">自定义小部件动画</st>

+   <st c="602">添加用户交互，例如按钮</st> <st c="641">和开关</st>

+   <st c="652">将控制小部件添加到控制中心和</st> <st c="700">锁屏</st>

<st c="711">因此，让我们从基础知识开始——小部件的概念是什么？</st> <st c="763">是</st>

# <st c="774">技术要求</st>

<st c="797">对于本章，从</st> <st c="881">App Store</st>下载 Xcode 版本 15.0 或更高版本是必要的。</st>

<st c="891">确保您正在运行最新版本的 macOS（Ventura 或更新版本）。</st> <st c="972">只需在 App Store 中搜索 Xcode，选择最新版本，然后继续下载。</st> <st c="1070">打开 Xcode 并完成出现的任何进一步设置说明。</st> <st c="1138">在 Xcode 完全运行后，您</st> <st c="1184">就可以开始了。</st>

<st c="1194">要获得额外的功能，例如在 widget 和应用程序之间共享数据，您必须在您的配置文件中设置 AppGroups 并定义您的 AppGroups。</st>

<st c="1349">从以下</st> <st c="1394">GitHub 链接</st>下载示例代码：

[<st c="1406">https://github.com/PacktPublishing/Mastering-iOS-18-Development/tree/main/Chapter%205</st>](https://github.com/PacktPublishing/Mastering-iOS-18-Development/tree/main/Chapter%205)

# <st c="1492">小部件的概念</st>

<st c="1512">添加小部件不是 iOS 或，实际上，在</st> <st c="1577">苹果生态系统中</st>的新概念。

<st c="1593">小部件早在 2005 年的*<st c="1634">Tiger</st>* <st c="1639">版本中就存在于 macOS 中，作为仪表板功能的一部分。</st> <st c="1692">苹果公司采纳了这个想法，并在 iOS 8 的*<st c="1728">Today Widgets</st>* <st c="1741">中引入了通知中心，而在 iOS 14 中，苹果公司引入了主屏幕小部件，类似于 Android 操作系统中已经存在的小部件。</st> <st c="1911">在 iOS 18 中，苹果公司增加了第三方应用向控制中心和主屏幕添加小部件的能力。</st>

<st c="2032">小部件的理念不是作为一个完整的应用程序——小部件不应该是我们应用的迷你版本或其屏幕之一，而应该是我们当前应用功能的扩展。</st> <st c="2216">小部件的存在是为了增强用户便利性和生产力，总的来说，是提升整体体验。</st>

<st c="2235">小部件旨在提升用户便利性和生产力，通常来说，是提升整体体验。</st>

<st c="2335">在 iOS 中，小部件有三个关键角色：</st>

+   **<st c="2381">一目了然的信息**** <st c="2405">——小部件为我们应用的用户提供最新和重要的信息。</st> <st c="2488">这可能包括配送状态、股票价值、事件日历或任何其他在日常生活中有用的信息。</st>

+   **<st c="2610">通往我们应用的捷径**** <st c="2632">——点击小部件可以打开我们的应用，在许多情况下，还可以打开我们应用的特定屏幕。</st> <st c="2719">在 watchOS 中，使用小部件打开我们的应用尤为重要，因为与 iOS 不同，watchOS 的启动界面不是用户的默认视图。</st> <st c="2851">对于许多应用开发者来说，这是一种很好的推广他们应用的方式，并在主屏幕上*<st c="2920">争夺</st> <st c="2928">用户的注意力。</st>

+   **<st c="2973">执行基本操作**** <st c="2998">——从 iOS 17 开始，苹果公司增加了交互式小部件，允许用户在不打开应用的情况下执行基本操作，例如完成任务、打开车库门或接受支付请求。</st> <st c="3202">在 iOS 18 中，这一功能更进一步，我们甚至可以将小部件添加到控制中心，或者使用 iPhone 15 设备上的动作按钮打开它们。</st>

<st c="3366">在回顾不同的苹果平台时，我们可以看到，**一目了然地展示信息**的想法非常普遍——在 iOS、iPadOS、macOS 和 watchOS 中，我们有主屏幕和锁定屏幕的小部件、复杂性和实时活动。</st>

<st c="3594">例如，Yahoo!</st> <st c="3619">天气应用显示了用户当前位置的天气，而苹果的提醒应用显示了用户的未完成提醒。</st>

<st c="3745">对于苹果来说，将不同平台之间的线条拉直成一个单一框架是顺理成章的——</st> *<st c="3855">WidgetK</st><st c="3862">it</st>*<st c="3865">。</st>

# <st c="3866">理解小部件的工作原理</st>

<st c="3897">如本章开头所述，小部件不是迷你应用程序。</st> <st c="3980">相反，小部件是简单的视图，显示相关信息，并根据声明的**时间线**或**应用程序事件**进行更新。</st>

<st c="4108">小部件</st> <st c="4121">在不同于应用程序的进程中运行。</st> <st c="4158">它们接收运行时来执行任何代码，因此它们作为静态视图工作，向我们的用户展示预先准备好的信息。</st> <st c="4275">但是，由于我们的用户数据正在不断更新，我们可以创建一个条目数组，每个条目都包含信息和日期。</st> <st c="4400">*<st c="4404">WidgetCenter</st>* <st c="4416">负责为每个条目创建不同的视图，存储它，并根据条目的日期替换</st> <st c="4521">小部件 UI</st>。</st> <st c="4564">这个条目数组被称为</st> <st c="4596">一个</st> **<st c="4598">时间线</st>**<st c="4606">。</st>

<st c="4607">一个很好的例子是</st> *<st c="4632">下一个事件</st>* <st c="4642">小部件。</st> <st c="4651">*<st c="4655">下一个事件</st>* <st c="4665">小部件显示我们日历中的下一个事件，由于我们可以访问用户的日历，我们可以根据日历事件列表构建时间线并刷新小部件数据。</st> <st c="4848">我们所需提供的只是包括每个</st> <st c="4925">时间线条目</st>的不同数据的**时间线**。</st>

<st c="4940">使用时间线来更新小部件的内容，使小部件成为一种极其有效的方式向用户展示信息，无论是在电池使用还是在</st> <st c="5095">处理时间</st>上。

<st c="5111">然而，时间线也给我们与小部件一起工作的方式带来了一些挑战，因为与*<st c="5216">下一个事件</st>* <st c="5226">小部件</st>不同，并不是每个时间线都可以预先构建。</st>

<st c="5276">但在我们深入到解决问题的解决方案并尝试添加我们的</st> <st c="5360">第一个小部件</st>之前，让我们先等等。

# <st c="5373">添加小部件</st>

<st c="5389">小部件</st> <st c="5406">在应用程序外部运行和存在，因此它们被视为我们应用程序的</st> *<st c="5473">扩展</st> <st c="5482">之一</st>。</st>

<st c="5494">要添加一个新的小部件——我们需要通过选择**<st c="5534">小部件扩展</st>** <st c="5550">目标</st>来创建一个新的**<st c="5571">文件</st>** <st c="5575">-></st> **<st c="5579">新建</st>** <st c="5582">-></st> **<st c="5586">目标…</st>**<st c="5593">。</st>

<st c="5594">然后，在</st> **<st c="5608">为您的新的目标选择一个模板</st>** <st c="5645">窗口中，我们搜索小部件并添加小部件扩展（见</st> *<st c="5711">图 5</st>**<st c="5719">.1</st>*<st c="5721">）：</st>

![图 5.1：为您的新的目标窗口选择一个模板](img/B21795_05_1.jpg)

<st c="5870">图 5.1：为您的新的目标选择一个模板窗口</st>

<st c="5930">在点击</st> **<st c="5949">下一步</st>**<st c="5953">后，我们应该为我们的小部件提供一个名称，就像我们添加的任何目标一样。</st> <st c="6025">此外，取消选中</st> *<st c="6050">包含配置应用程序</st> * *<st c="6076">意图</st> * <st c="6082">复选框。</st>

<st c="6092">一旦添加了小部件，我们就可以看到一个新的目标，其名称是我们提供的。</st> <st c="6170">Xcode 会为我们创建一些文件，作为小部件模板的一部分（假设目标名称</st> <st c="6265">是</st> `<st c="6268">MyWidget</st>`<st c="6276">）：</st>

+   `<st c="6279">MyWidgetBundle</st>` <st c="6294">– 小部件包是我们扩展所包含的不同小部件的容器。</st> <st c="6377">目前，我们只有一个小部件，但可以添加更多。</st>

+   `<st c="6444">MyWidget</st>` <st c="6453">– 包含小部件代码本身，包括其 UI</st> <st c="6506">和配置。</st>

+   `<st c="6524">Assets</st>` <st c="6531">– 一个专门为小部件扩展设计的资产目录。</st>

+   `<st c="6589">Info.plist</st>` <st c="6600">– 就像任何目标一样，小部件扩展包含一个</st> `<st c="6657">plist</st>` <st c="6662">文件，其中包含有关小部件扩展的通用信息。</st>

<st c="6720">现在，是时候</st> <st c="6739">阐明小部件是什么了——我们为小部件有不同的尺寸并不意味着它们是不同的小部件，因为相同的小部件可以有多个尺寸。</st> <st c="6902">不同的小部件通常是指不同的产品、不同的 UI 和不同的用例。</st> <st c="6995">在我们的案例中，小部件包描述了不同的小部件，而不是不同的小部件尺寸。</st>

<st c="7093">现在我们已经将小部件添加到我们的项目中，我们可以运行我们的应用程序并将新小部件添加到我们的启动板上（</st>*<st c="7204">图 5</st>**<st c="7213">.2</st>*<st c="7215">）：</st>

![图 5.2：在启动板中的我们的新模板小部件](img/B21795_05_2.jpg)

<st c="7220">图 5.2：在启动板中的我们的新模板小部件</st>

<st c="7274">我们可以在</st> *<st c="7289">图 5</st>**<st c="7297">.2</st>* <st c="7299">中看到，新的小部件由当前时间和一些表情符号组成。</st> <st c="7365">这是玩它并尝试添加额外的</st> <st c="7430">小部件尺寸的好时机。</st>

## <st c="7443">配置我们的小部件</st>

<st c="7466">我们设置小部件的外观和行为的方式是通过确定其配置。</st> <st c="7553">我们有</st> <st c="7561">几个配置可以工作，并且它们都符合一个名为</st> <st c="7633">WidgetConfiguration</st>`<st c="7658">。</st>

<st c="7659">我们可用的配置之一是</st> `<st c="7706">StaticConfiguration</st>`<st c="7725">。</st> `<st c="7727">StaticConfiguration</st>` <st c="7746">允许我们创建一个没有任何</st> <st c="7788">用户可配置选项的小部件。</st>

<st c="7814">让我们看看当我们在 Xcode 中添加一个新小部件时，它提供的</st> `<st c="7840">StaticConfiguration</st>` <st c="7859">。</st> <st c="7894">new widget:</st>

```swift
 struct MyWidget: Widget {
    let kind: String = "MyWidget"
    var body: some WidgetConfiguration { <st c="7999">StaticConfiguration(kind: kind, provider:</st>
 <st c="8040">Provider()) { entry in</st>
 <st c="8063">MyWidgetEntryView(entry: entry)</st>
 <st c="8095">.containerBackground(.fill.tertiary, for:</st>
 <st c="8136">.widget)</st>
 <st c="8144">}</st>
 <st c="8146">.configurationDisplayName("My Widget")</st> .description("This is an example widget.")
    }
}
```

<st c="8230">我们可以看到</st> `<st c="8247">StaticConfiguration</st>` <st c="8266">具有与所有配置类型共享的几个属性。</st> <st c="8327">让我们深入看看它们，如下所示：</st>

+   `<st c="8357">kind</st>` <st c="8362">– 这是小部件配置的唯一标识符。</st> <st c="8417">它帮助我们使用</st> *<st c="8488">WidgetCenter</st>*<st c="8500">向特定小部件配置发送请求。</st>

+   `<st c="8501">configurationDisplayName</st>` <st c="8526">– 这是小部件显示名称，当用户想要选择要添加的正确小部件时，它将显示给用户。</st> <st c="8627">to add.</st>

+   `<st c="8634">description</st>` <st c="8646">– 这是小部件的描述，它显示在用户旁边，紧邻其</st> <st c="8721">显示名称。</st>

除了这三个参数之外，我们还有其他一些重要的参数。<st c="8734">supportedFamilies</st> <st c="8825">决定了小部件支持的尺寸。</st> <st c="8878">以下是一个如何限制小部件只以中等尺寸显示的示例：</st> <st c="8941">medium size:</st>

```swift
 .supportedFamilies([.systemMedium])
```

<st c="8988">另一个属性是</st> `<st c="9009">backgroundTask</st>`<st c="9023">，它允许我们的小部件在系统给它时间时执行后台操作。</st>

<st c="9113">注意</st> `<st c="9126">WidgetConfiguration</st>` <st c="9145">只是一个协议 – 当创建小部件时，我们需要在组件体中返回一个符合该协议的结构，并且</st> `<st c="9278">StaticConfiguration</st>` <st c="9297">只是实现这一点的其中一种方式。</st>

<st c="9325">目前，我们有三种配置可供我们使用：</st> <st c="9378">for us:</st>

+   `<st c="9385">StaticConfiguration</st>` <st c="9405">– 如前所述，此配置允许我们创建一个不可配置的小部件</st>

+   `<st c="9499">AppIntentConfiguration</st>` <st c="9522">– 这使用户能够自定义他们的小部件，例如，为天气小部件选择一个城市，或为</st> <st c="9653">提醒应用</st>

+   `<ActivityConfiguration>` – 这项配置显示了实时活动小部件的实时数据

一个小部件只能包含一个配置。如果我们需要多个配置，那么这是一个很好的迹象，表明我们需要创建具有不同配置的多个小部件，并在它们之间共享一些代码。

所有这些小部件配置听起来都很吸引人！让我们从探索`StaticConfiguration`开始。

## 工作与静态配置

一个静态小部件是一个没有用户可配置选项的小部件。例如，一个显示特定城市当前时间的小部件不能是静态的，因为用户需要指定一个城市或位置来配置小部件。

然而，一个静态小部件的好例子是一个显示整个月视图并标记当前日期的日历小部件，或者是一个显示最近播放的歌曲的音乐应用小部件。

尽管日历和音乐应用小部件显示的信息不是由用户更新的，但它们需要不时地更新自己。

如果我们回顾静态配置示例（在*《配置我们的小部件》*部分），我们可以看到一个名为`provider`的参数，它包含一个名为`entry`的视图构建器闭包参数。

使用`provider`和`entry`，我们可以以高效的方式在时间上为我们的小部件提供数据。

小部件的一个关键方面是提供时间上的数据，我们使用时间线提供者来实现这一点。现在，让我们了解时间线提供者是什么意思。

## 理解小部件的时间线提供者

有一个原因，为什么苹果公司花了近 14 年时间才在 iOS 主屏幕上支持小部件。主要原因是性能，包括电力和内存性能。虽然今天的设备功能强大，但在 Springboard 上拥有大量活动小部件会消耗大量的电力。因此，我们需要找到更有效的方法来高效地加载我们的小部件。

<st c="11570">我们在</st> *<st c="11602">理解小部件如何工作</st>* <st c="11632">部分提到了效率，所以让我们深入细节。</st> <st c="11676">与应用不同，小部件即使在可见时也不是活跃的。</st> <st c="11740">我们可以在特定时间“唤醒”这些小部件以重新加载它们的视图。</st> <st c="11809">为了设置特定时间段，我们需要创建一个时间线 – 一个包含时间点和相关数据的条目数组。</st> <st c="11921">创建一个更长的时线可以最大化我们小部件的更新频率。</st>

<st c="11935">例如，如果我们想重新加载显示下一个事件的日历小部件，我们可以创建一个包含每个事件的条目的时间线。</st> <st c="12095">每个条目包含事件时间和随后发生的事件名称。</st>

<st c="12173">相反，如果我们想创建一个显示全天信息的日历小部件，我们可能需要为每一天创建一个包含条目的时间线。</st> <st c="12312">在这种情况下，每个条目包含一天开始的时间和当天的活动列表。</st>

<st c="12412">创建一个更长的时线可以最大化我们小部件的更新频率。</st>

<st c="12493">现在，让我们转向代码并创建我们的第一个时间线。</st> <st c="12549">以下是一个显示下一个事件的时线提供者的示例：</st>

```swift
 struct EventEntry: TimelineEntry {
    let date: Date
    let nextEvent: String
}
struct Provider: TimelineProvider {
    func placeholder(in context: Context) -> EventEntry {
        EventEntry(date: Date(), nextEvent: "Loading")
    }
    func getSnapshot(in context: Context, completion:
      @escaping (EventEntry) -> Void) {
        let entry = EventEntry(date: Date(), nextEvent: "Go
          to the book store")
        completion(entry)
    }
    func getTimeline(in context: Context, completion:
      @escaping (Timeline<EventEntry>) -> Void) {
        let entries: [EventEntry] = getListOfEnties()
        let timeline = Timeline(entries: entries, policy:
          .atEnd)
        completion(timeline)
    }
    func getListOfEnties()->[EventEntry] {
      …
    }
}
```

<st c="13275">前面的代码包含两个结构体 –</st> `<st c="13321">EventEntry</st>` <st c="13331">和</st> `<st c="13336">Provider</st>`<st c="13344">.</st>

`<st c="13345">EventEntry</st>` <st c="13356">是一个符合</st> `<st c="13386">TimeLineEntry</st>` <st c="13399">协议的结构体。</st> <st c="13410">`<st c="13414">TimeLineEntry</st>` <st c="13427">`协议表示小部件时间线中的单个条目。</st> <st c="13487">该协议包含一个必需的变量</st> <st c="13529">名为 date:</st>

```swift
 var date: Date { get }
```

<st c="13563">`<st c="13568">date</st>` <st c="13572">变量包含我们期望小部件重新加载的时间点。</st> <st c="13653">除了`<st c="13664">date</st>`<st c="13668">`之外，我们还添加了一个表示条目下一个事件标题的变量，名为`<st c="13741">nextEvent</st>`<st c="13756">`。</st>

<st c="13757">我们的第二个结构体是`<st c="13769">Provider</st>`<st c="13787">`。`<st c="13793">Provider</st>` <st c="13801">结构体符合`<st c="13821">TimeLineProvider</st>`<st c="13837">`。`<st c="13855">Provider</st>` <st c="13863">结构体的目的是生成一个时间线，以便`<st c="13904">WidgetCenter</st>` <st c="13916">在需要时重新加载小部件。</st> <st c="13952">让我们看看`<st c="13970">Provider</st>` <st c="13978">是如何做到这一点的。</st>

### <st c="13989">生成时间线</st>

<st c="14011">我之前提到时间线是一系列时间线条目，但实际情况要复杂一些。</st> <st c="14118">查看时间线提供者实现，我们可以看到几个函数，这些函数可以帮助我们在任何给定时间提供静态 UI。</st>

<st c="14269">Provider 结构是一个协议实现</st>

<st c="14318">没有必要直接调用</st> `<st c="14347">Provider</st>` <st c="14355">函数。</st> <st c="14376">我们将时间线提供者传递给组件配置，配置在需要时使用</st> `<st c="14466">Provider</st>` <st c="14474">函数。</st>

<st c="14497">第一个和主要的功能是</st> `<st c="14532">getTimeLine</st>`<st c="14543">。让我们看看</st> `<st c="14585">getTimeline</st>` <st c="14596">函数的实现：</st>

```swift
 func getTimeline(in context: Context, completion: @escaping
    (Timeline<EventEntry>) -> Void) {
        let entries: [EventEntry] = getListOfEntries()
        let timeline = Timeline(entries: entries, policy:
          .atEnd)
        completion(timeline)
    }
```

<st c="14833">`<st c="14838">getTimeline()</st>` <st c="14851">函数</st> <st c="14861">创建一个条目数组，将其包装在一个</st> `<st c="14906">Timeline</st>` <st c="14914">结构中，并使用</st> `<st c="14948">completion</st>` <st c="14958">闭包返回它。</st> <st c="14968">这里有两个有趣的地方可以观察 –</st> `<st c="15023">Context</st>` <st c="15030">参数和</st> **<st c="15049">Timeline</st>** **<st c="15058">重载策略</st>**<st c="15071">：</st>

+   `<st c="15073">上下文</st>` <st c="15080">–</st> `<st c="15087">Context</st>` <st c="15094">参数包含有关小部件环境的信息，例如小部件系列（它是一个小部件吗？</st> <st c="15161">也许是一个中等大小的？），或者实际的小部件大小。</st> <st c="15250">如果小部件 UI 在大尺寸时显示更多信息，我们可能希望将更多数据加载到我们的时间线条目中。</st> <st c="15368">但这里最重要的信息可能是</st> `<st c="15424">isPreview</st>` <st c="15433">属性，它指示小部件是否出现在小部件库中。</st> <st c="15510">一般来说，在组件库中展示我们的组件的真实用户数据是最佳实践，但这并不仅限于安全或网络问题。</st> <st c="15677">因此，我们可以通过检查</st> `<st c="15752">isPreview</st>` <st c="15761">属性来为组件库提供模拟数据。</st>

+   `<st c="15771">策略</st>` `<st c="15778">– 我们提供给小部件的时间线有一个最终条目数。</st>` `<st c="15850">那么，当它们完成，时间线达到其尽头时会发生什么？</st>` `<st c="15921">这正是</st>` `<st c="15940">策略</st>` `<st c="15952">参数在描述时间线重新加载行为时的作用。</st>` `<st c="16017">有几个选项 –</st>` `<st c="16045">atEnd</st>` `<st c="16050">(</st>` `<st c="16052">WidgetKit</st>` `<st c="16062">请求一个新的时间线)，</st>` `<st c="16088">never</st>` `<st c="16093">(</st>` `<st c="16095">WidgetKit</st>` `<st c="16105">不请求一个新的时间线)，以及</st>` `<st c="16142">after(date:Date)</st>` `<st c="16158">(</st>` `<st c="16160">WidgetKit</st>` `<st c="16170">在特定日期生成一个新的时间线)。</st>` `<st c="16216">策略有助于</st>` `<st c="16237">WidgetCenter</st>` `<st c="16249">更好地优化时间线重新加载机制。</st>

`<st c="16302">在我们继续之前，关于时间线重新加载优化的几点说明。</st>` `<st c="16374">我们希望尽可能长地构建我们的时间线并不意味着我们的小部件需要不断重新加载。</st>` `<st c="16495">*<st c="16499">WidgetCenter</st>` `<st c="16511">为每个主屏幕上的小部件都有一个“预算”，指定一天中它执行刷新的时间。</st>` `<st c="16623">优化我们的时间线结构和“节省”系统预算符合我们的利益。</st>` `<st c="16724">精心规划时间线条目和重新加载策略可以帮助我们实现相关的事件驱动</st>` `<st c="16825">刷新间隔。</st>`

`<st c="16843">回到</st>` `<st c="16862">TimelineProvider</st>` `<st c="16878">协议，我们可以看到另外两个函数 –</st>` `<st c="16927">placeholder</st>` `<st c="16938">和</st>` `<st c="16943">getSnapshot</st>` `<st c="16954">。让我们</st>` `<st c="16962">实现它们。</st>`

`<st c="16977">第一个函数是</st>` `<st c="17000">getTimeline</st>` `<st c="17011">，它返回一个</st>` `<st c="17029">Timeline</st>` `<st c="17037">结构，其中包含特定时间段的实际数据条目列表。</st>` `<st c="17116">但这是否足以让我们的小部件完全功能正常？</st>`

`<st c="17171">答案是：不——在提供实际数据可能不足以满足的两种情况下。</st>`

`<st c="17265">*<st c="17270">placeholder</st>` `<st c="17281">函数解决了第一个用例。</st>` `<st c="17319">当用户将小部件添加到他们的主屏幕时，</st>` `<st c="17369">WidgetKit</st>` `<st c="17379">需要在小部件从我们的应用程序获取</st>` `<st c="17445">或更新真实数据之前立即显示某些内容。</st>` `<st c="17480">*<st c="17484">placeholder</st>` `<st c="17495">函数返回临时数据，仅用于向</st>` `<st c="17554">用户</st>` `<st c="17555">显示：</st>`

```swift
 func placeholder(in context: Context) -> EventEntry {
        EventEntry(date: Date(), nextEvent: "English
          class")
    }
```

<st c="17672">在我们的示例中，我们可以看到一个</st> `<st c="17702">占位符</st>` <st c="17713">函数，它返回</st> `<st c="17740">English</st>` `<st c="17748">类</st>` <st c="17753">文本。</st>

<st c="17759">返回临时数据而不是加载指示器很重要，例如，这是因为我们希望用户体验保持一致和流畅。</st> <st c="17922">同时，创造性地提出优雅的信息会更好。</st> <st c="17993">例如，如果我们的小部件有一个计时器或时间，显示 00:00 是一个好主意，以向用户表明计时器应该</st> <st c="18119">出现在这里。</st>

<st c="18131">第二个函数是</st> `<st c="18155">getSnapshot</st>`<st c="18166">。`<st c="18172">getSnapShot</st>` <st c="18183">函数甚至比</st> `<st c="18221">placeholder</st>`<st c="18232">` 更重要。当用户浏览小部件画廊时，系统会展示不同的小部件。</st> <st c="18319">这些小部件在没有系统生成的时间线的情况下被展示。</st>

<st c="18392">`getSnapshot</st>` <st c="18397">函数返回一个基于</st> `<st c="18428">TimelineEntry</st>`<st c="18441">的具有要在小部件画廊中呈现的数据的结构。</st>

<st c="18499">以下是一个</st> `<st c="18523">getSnapshot</st>` <st c="18534">函数的示例：</st>

```swift
 func getSnapshot(in context: Context, completion:
      @escaping (EventEntry) -> Void) {
        let entry = EventEntry(date: Date(), nextEvent: "Go
          to the book store")
        completion(entry)
    }
```

<st c="18720">在这段代码中，</st> `<st c="18739">getSnapshot</st>` <st c="18750">函数返回一个包含当前日期的示例事件。</st> <st c="18808">这个快照向用户展示了我们的</st> `<st c="18866">小部件</st>` <st c="18866">的目的。</st>

<st c="18880">请注意，在</st> `<st c="18899">占位符</st>` <st c="18910">和</st> `<st c="18915">getSnapshot</st>`<st c="18926">` 中，我们都有与</st> `<st c="18945">Context</st>` <st c="18952">参数</st> <st c="18963">相同的参数，就像我们在</st> `<st c="18988">getTimeline</st>` <st c="18999">函数中拥有的那样。</st> <st c="19010">我们需要</st> `<st c="19022">Context</st>` <st c="19029">的原因与之前相同——为了了解围绕</st> `<st c="19104">我们的小部件</st>` <st c="19104">的环境。</st>

<st c="19115">现在我们了解了如何生成时间线提供者，让我们来讨论</st> `<st c="19190">TimelineEntry</st>`<st c="19207">。</st>

### <st c="19208">构建我们的</st> `<st c="19245">TimelineEntry</st>` <st c="19208">结构</st>

我们现在可以看到，<st c="19245">`TimelineProvider`</st> 协议很简单，因为只需要实现三个函数。</st> <st c="19273">其中我们需要设计的一个是</st> `<st c="19410">TimelineEntry</st>`<st c="19423">`，它之所以重要，是因为它包含了我们需要的信息，不仅用于确定何时呈现信息，还包括呈现什么。</st> <st c="19433">信息。</st>

<st c="19575">时间线条目</st> `<st c="19593">的结构</st>` <st c="19606">需要符合我们的小部件目标并与它的 UI 对齐。</st> <st c="19664">因为我们根据时间线预先生成了所有条目，我们应该提前进行所有计算并生成一个可以轻松更新小部件</st> <st c="19836">内容的结构。</st>

<st c="19851">实际上，</st> `<st c="19861">时间线条目</st>` <st c="19874">可能由</st> <st c="19890">四个组件组成：</st>

+   `<st c="19906">日期</st>` <st c="19911">– 我们希望小部件重新加载特定条目信息的日期。</st> <st c="19923">注意，在大多数情况下，</st> `<st c="20020">日期</st>` <st c="20024">属性不是屏幕上展示的信息的一部分。</st> <st c="20090">例如，在日历小部件中，我们可能将日期属性作为</st> `<st c="20173">时间线条目</st>` <st c="20186">协议的一部分，并为实际</st> <st c="20216">事件时间使用类似</st> `<st c="20225">eventDate</st>` <st c="20241">的属性。</st>

+   `<st c="20401">标题</st>`<st c="20406">,</st> `<st c="20408">正文内容</st>`<st c="20416">, 和</st> `<st c="20422">时间字符串</st>`<st c="20432">, 可以简化我们的代码甚至</st> <st c="20465">提高性能。</st>

+   `<st c="20819">时间线条目</st>` <st c="20832">是用户与之交互时我们所拥有的全部。</st>

+   `<st c="20902">相关性</st>` <st c="20911">属性是我们作为</st> `<st c="20973">时间线条目</st>` <st c="20986">协议的一部分拥有的可选属性。</st> <st c="20997">在</st> `<st c="21004">相关性</st>` <st c="21013">属性中，我们可以确定条目对用户的关联优先级。</st> <st c="21090">例如，一个向用户展示下一个任务的待办事项应用可能希望将高分数设置给包含关键任务的条目。</st> <st c="21215">或者，一个在小部件中展示最新新闻的运动应用可能希望将高分数设置给包含用户喜欢的球队新闻的条目。</st> <st c="21362">条目的相关性值帮助</st> *<st c="21397">WidgetKit</st>* <st c="21407">决定如何在系统中何时展示小部件。</st> <st c="21464">例如，</st> *<st c="21477">WidgetKit</st>* <st c="21487">可能决定旋转堆叠小部件并展示一个高相关性信息的小部件。</st> <st c="21576">让我们看看如何为</st> `<st c="21608">相关性</st>` <st c="21617">设置</st> `<st c="21622">一个</st> `<st c="21624">时间线条目</st>`<st c="21637">的例子：</st>

    ```swift
     struct EventEntry: TimelineEntry {
        let date: Date
        let nextEvent: String
        var relevance: TimelineEntryRelevance? }
    let entry = EventEntry(date: date, nextEvent: "Go to
      the book store", <st c="21823">relevance:</st>
    <st c="21897">relevance</st> property to our <st c="21923">EventEntry</st> struct and set a score of <st c="21960">1.0</st>. It is worth noting that any efforts to manipulate the system and set high scores for all entries won’t succeed – Apple has built an algorithm that filters out widgets that have unrealistic values. As with many iOS frameworks, this is a situation where we need to follow the platform’s intended usage guidelines.
    ```

<st c="22276">现在我们已经创建了一个时间线，让我们转向主要话题，即构建我们的</st> <st c="22366">小部件 UI。</st>

# <st c="22376">构建我们的小部件 UI</st>

<st c="22399">创建条目时间线对于我们的小部件向用户提供准确和相关信息至关重要。</st> <st c="22503">但要做到这一点，我们还需要渲染小部件 UI。</st> <st c="22516">我们进行渲染的地方是小部件的结构中，正如我们在本章开头在</st> *<st c="22677">配置我们的</st> * *<st c="22693">小部件</st> * *<st c="22699">部分</st> 中所看到的。</st>

<st c="22708">让我们再次看看</st> <st c="22723">配置：</st>

```swift
 StaticConfiguration(kind: kind, provider: Provider()) {
    entry in <st c="22809">MyWidgetEntryView(entry: entry)</st>
 <st c="22840">.containerBackground(.fill.tertiary, for:</st>
 <st c="22881">.widget)</st> }
```

<st c="22891">正如我们所见，</st> `<st c="22910">StaticConfiguration</st>` <st c="22929">有一个返回 SwiftUI 视图的视图构建器，这可能是我们在</st> *<st c="23040">WidgetKit</st>* <st c="23050">中需要理解的第一件事——小部件仅使用 SwiftUI 构建。</st> <st c="23089">如果你还没有任何 SwiftUI 的经验，</st> *<st c="23143">WidgetKit</st>* <st c="23153">是一个开始学习的好机会。</st>

<st c="23185">可能引起你注意的可能是</st> `<st c="23241">containerBackground</st>` <st c="23260">视图修饰符。</st> <st c="23276">如果你还记得，我们讨论过小部件现在可以在苹果生态系统的不同地方显示——</st> *<st c="23385">iOS</st>* <st c="23388">(主屏幕和锁屏)，</st> *<st c="23425">padOS</st>*<st c="23430">，</st> *<st c="23432">macOS</st>*<st c="23437">，以及</st> *<st c="23443">watchOS</st>*<st c="23450">。但将我们的小部件放在不同的平台上的主要问题可能是小部件的背景。</st> <st c="23534">小部件的背景。</st>

<st c="23554">添加</st> `<st c="23566">containerBackground</st>` <st c="23585">视图修饰符确保小部件的背景会根据其容器进行调整，并且无论出现在哪里都始终看起来很好。</st> <st c="23708">它出现在哪里都一样。</st>

<st c="23719">如果我们再次查看</st> <st c="23734">我们的代码示例，我们可以看到</st> `<st c="23774">MyWidgetEntryView</st>` <st c="23791">接收一个参数，即当前的</st> 时间线条目。</st> <st c="23853">让我们看看我们能从中学习到什么。</st>

## <st c="23889">处理时间线条目</st>

<st c="23919">将时间线条目连接到小部件视图是理解小部件工作原理的核心。</st> <st c="24002">*<st c="24019">WidgetCenter</st>* <st c="24031">的主要作用是生成时间线，并在正确的时间为我们的小部件提供正确的时间线条目。</st>

<st c="24129">小部件</st> <st c="24141">配置视图构建器有一个参数，即特定的时间线条目，因此我们可以返回一个包含相关数据的</st> 小部件视图。</st>

<st c="24264">这是一个使用特定</st> <st c="24321">时间线条目的</st> 小部件视图的示例：</st>

```swift
 struct MyWidgetEntryView: View { <st c="24370">let entry:</st> EventEntry
    var body: some View {
        VStack(alignment: .leading) {
            Text("Next Event:")
                .font(.headline)
            Text(<st c="24486">entry.nextEventTitle</st>)
                .font(.title)
                .foregroundColor(.blue)
            Text("Time: \(<st c="24562">entry.nextEventTime</st>)")
                .font(.subheadline)
            Spacer()
        }
        .padding()
    }
}
```

<st c="24632">此代码</st> <st c="24643">示例显示了一个简单的视图，它在使用</st> <st c="24728">时间线条目时显示下一个事件标题和时间。</st>

<st c="24743">从时间线条目与组件视图协同工作的方式中，我们可以学到两件事：</st> <st c="24825">组件视图：</st>

+   **<st c="24837">条目应包含所有组件的数据</st>** <st c="24884">—— 我们在讨论时间线提供者时提到了这一点，但现在我们可以看到原因了。</st> <st c="24971">组件需要尽可能静态和简单。</st> <st c="25024">我们不希望在视图显示时执行任何数据获取操作。</st>

+   **<st c="25106">没有状态</st>** <st c="25124">—— 与常规 SwiftUI 视图不同，我们的组件视图没有状态。</st> <st c="25195">有些情况下，我们可能希望根据不同情况显示不同的视图。</st> <st c="25289">例如，在我们的下一个组件视图示例中，如果我们希望用户尚未批准其日历权限，我们可能想显示一条消息，提示用户*<st c="25309">连接到您的日历</st>**。</st> <st c="25360">如果用户尚未批准其日历权限，我们可能想显示一条消息，提示用户*<st c="25360">连接到您的日历</st>**。</st> <st c="25384">为了做到这一点，我们需要生成不同的时间线条目，并在静态配置闭包中显示不同的视图。</st> <st c="25447">无论如何，我们应该提前进行这些检查。</st>

<st c="25626">尽管组件自然是静态的，但它们的 UI 不必保持静态和粗体。</st> <st c="25720">在`<st c="25723">WidgetKit</st>`<st c="25732">`中，我们可以通过动画变化来使我们的组件生动起来。</st>

## <st c="25802">添加动画</st>

<st c="25820">我们已经</st> <st c="25832">了解了 iOS 开发中动画的工作原理——视图动画通过在两个或多个状态之间转换来实现。</st> <st c="25944">例如，如果一个特定的视图的透明度为`<st c="25994">1.0</st>` <st c="25997">，我们将其更改为`<st c="26018">0.5</st>`<st c="26021">，如果喜欢，UIKit 和 SwiftUI 可以对该变化进行动画处理。</st>

<st c="26076">组件是用 SwiftUI 编写的，在 SwiftUI 中，我们可以对状态变化进行动画处理。</st> <st c="26155">然而，组件并不使用状态。</st> <st c="26196">相反，我们通过时间线提供者（也许我们可以这样说，在某种程度上，时间线条目就是我们的组件状态）来更改组件内容。</st>

<st c="26338">从 iOS 16 开始，每当`<st c="26374">WidgetCenter</st>` <st c="26386">重新加载组件并使用新条目更改其内容时，它会自动执行此`<st c="26464">转换</st>`。</st>

<st c="26489">即使我们没有在组件中设置状态，我们能否自定义这个动画？</st> <st c="26497">当然可以，</st> <st c="26564">使用`<st c="26588">contentTransition</st>`<st c="26605">。</st>

<st c="26606">如前所述，在大多数情况下，SwiftUI 根据状态变化执行动画。</st> <st c="26689">例如，看看以下代码：</st> <st c="26714">以下代码：</st>

```swift
 @State private var isRed = false
    var body: some View {
        VStack {
            Color(isRed ? .red : .blue)
                .frame(width: 100, height: 100)
                .cornerRadius(10)
            Button("Change Color") { <st c="26898">withAnimation {</st>
 <st c="26913">self.isRed.toggle()</st>
 <st c="26933">}</st> }
        }
    }
```

<st c="26941">在这个代码示例中，我们有一个视图和一个按钮。</st> <st c="26993">点击按钮会改变视图的颜色，它是通过使用</st> `<st c="27066">withAnimation</st>` <st c="27079">函数来实现的。</st> <st c="27090">显然，这在一个组件中是不可行的，因为我们需要一个状态来</st> <st c="27154">完成这个操作。</st>

<st c="27162">相反，我们需要做的是定义内容在动画时如何变化。</st> <st c="27180">为了做到这一点，我们可以</st> <st c="27264">使用</st> `<st c="27268">contentTransition</st>`<st c="27285">:</st>

```swift
 Color(isRed ? .red : .blue)
                .frame(width: 100, height: 100)
                .cornerRadius(10)
 <st c="27367">.contentTransition(.opacity)</st> Button("Change Color") {
                withAnimation() {
                    self.isRed.toggle()
                }
            }
```

`<st c="27462">contentTransition</st>` <st c="27480">是一种我们可以添加到视图中的视图修改器，用于定义它们的过渡方法。</st> <st c="27555">想象一下，所有在组件中的内容更改都是通过</st> `<st c="27613">withAnimation</st>` <st c="27626">来完成的，而我们唯一需要做的就是改变</st> <st c="27674">过渡方法。</st>

<st c="27692">例如，以下</st> <st c="27726">代码片段：</st>

```swift
 Text(text)<st c="27818">withAnimation()</st> function, it will change its content with a nice numeric transition (you can try it yourself). If you are not familiar with the <st c="27962">withAnimation</st> function, *<st c="27986">Chapter 6</st>* provides a brief discussion on it.
			<st c="28030">In widgets, all we need to do is to add these to views with content that is based on our timeline entry, and SwiftUI will take care of the</st> <st c="28170">animation itself.</st>
			<st c="28187">Look at our widget again, now</st> <st c="28218">with</st> `<st c="28223">contentTransition</st>`<st c="28240">:</st>

```

struct MyWidgetEntryView : View {

    var entry: Provider.Entry

    var body: some View {

        VStack {

            Text("时间:")

            Text(entry.nextEventTime, style: .time)

            Text("下一个事件")

            Text(entry.nextEvent) <st c="28429">.contentTransition(.numericText())</st> }

    }

}

```swift

			<st c="28469">Even though</st> <st c="28482">there is no state or</st> `<st c="28503">withAnimation</st>` <st c="28516">function, the</st> `<st c="28531">nextEvent</st>` <st c="28540">title will animate its transition.</st> <st c="28576">The</st> `<st c="28580">contentTransiton</st>` <st c="28596">view modifier has additional options, such as opacity and symbol effects.</st> <st c="28671">Despite the fact that it is not designed explicitly for widgets, it’s the best way to make our widgets</st> <st c="28774">more alive.</st>
			<st c="28785">Customize our widget</st>
			<st c="28806">Up until now, we have discussed widgets based on a</st> `<st c="28858">staticConfiguration</st>`<st c="28877">. The</st> `<st c="28883">staticConfiguration</st>` <st c="28902">set is great for most widgets.</st> <st c="28934">However, there are cases where we</st> <st c="28968">want to provide our users the ability to customize</st> <st c="29019">and configure t</st><st c="29034">heir widgets with</st> <st c="29053">additional entities.</st>
			<st c="29073">Going back to our calendar widget, we want to allow the user to filter the next event information based on a</st> <st c="29183">specific calendar.</st>
			<st c="29201">To do that, we’ll start by creating a new file and add a struct called</st> `<st c="29273">CalendarWidgetIntent</st>` <st c="29293">that conforms</st> <st c="29308">to</st> `<st c="29311">WidgetConfigurationIntent</st>`<st c="29336">.</st>
			<st c="29337">Adding intent</st>
			<st c="29351">A</st> `<st c="29354">WidgetConfigurationIntent</st>` <st c="29379">is an App Intent we can use to configure widgets, and our</st> `<st c="29438">CalendarWidgetIntent</st>` <st c="29458">contains all the configuration information</st> <st c="29502">we need.</st>
			<st c="29510">Here is</st> <st c="29519">a basic</st> `<st c="29527">CalendarWidgetIntent</st>` <st c="29547">implementation:</st>

```

struct CalendarWidgetIntent: WidgetConfigurationIntent {

    static var title: LocalizedStringResource = "Select

    Calendar"

    @Parameter(title: "Calendar") var calendar:

    CalendarEntity

}

```swift

			<st c="29743">In the preceding code, we can see</st> <st c="29778">two properties:</st>

				*   `<st c="29793">title</st>` <st c="29799">– The title of the intent.</st> <st c="29827">It is important to note that we don’t see the title in the widget configuration string but rather in Siri Shortcuts.</st> <st c="29944">But we must add it since it is part of the</st> `<st c="29987">AppIntent</st>` <st c="29996">protocol (the</st> `<st c="30011">WidgetConfigurationIntent</st>` <st c="30036">inheritance from</st> `<st c="30054">AppIntent</st>` <st c="30063">protocol).</st>
				*   `<st c="30074">calendar</st>`<st c="30083">– This is the widget parameter that allows the user to configure the calendar the event belongs to.</st> <st c="30184">We can see that the</st> `<st c="30204">calendar</st>` <st c="30212">variable is prefixed by the</st> `<st c="30241">@Parameter</st>` <st c="30251">macro, which manages this property for the</st> <st c="30295">user’s configuration.</st>

			<st c="30316">Now, let’s add the</st> <st c="30336">App Intent.</st>
			<st c="30347">Adding AppEntity</st>
			<st c="30364">As you</st> <st c="30372">have noticed, the calendar variable is based on a type</st> <st c="30427">called</st> `<st c="30434">CalendarEntity</st>`<st c="30448">.</st>
			<st c="30449">If we want to support our own entity type, it needs to conform to</st> `<st c="30516">AppEntity</st>`<st c="30525">. Let’s see the</st> `<st c="30541">CalendarEntity</st>` <st c="30555">type implementation:</st>

```

struct CalendarEntity: AppEntity {

    let id: String

    let name: String

    static var typeDisplayRepresentation:

    TypeDisplayRepresentation = "Calendar"

    static var defaultQuery = CalendarQuery()

    var displayRepresentation: DisplayRepresentation {

        DisplayRepresentation(title: name)

    }

}

```swift

			<st c="30852">The</st> `<st c="30857">CalendarEntity</st>` <st c="30871">struct represents the data model for the</st> `<st c="30913">intent</st>` <st c="30919">parameter.</st> <st c="30931">First, we need to add the parameters we need in order to support the item when displaying</st> <st c="31021">the widget, such as</st> `<st c="31041">id</st>` <st c="31043">and</st> `<st c="31048">name</st>`<st c="31052">. Next, we’ll add some representation variables, such as</st> `<st c="31109">typeDisplayRepresentation</st>` <st c="31134">and</st> `<st c="31139">displayRepresentation</st>`<st c="31160">.</st>
			<st c="31161">Finally, we’ll add a static variable that handles the actual data fetching, and that’s the</st> `<st c="31253">defaultQuery</st>` <st c="31265">property.</st> <st c="31276">Remember that the user needs to select the desired calendar based on a list of calendars.</st> <st c="31366">To do that, we need to provide</st> *<st c="31397">WidgetKit</st>* <st c="31407">with a way to query our data to support the selection</st> <st c="31461">UI flow.</st>
			<st c="31469">So, what does the query look like?</st> <st c="31505">Let’s</st> <st c="31511">find out.</st>
			<st c="31520">Building the EntityQuery</st>
			<st c="31545">Sometimes, having a</st> <st c="31566">list of options for the user relies on a data store, and sometimes on</st> <st c="31636">static information.</st>
			<st c="31655">Regardless of the model type, if we want to provide options to the user, we need to have a simple and effective interface to work with, and that’s what the</st> `<st c="31812">EntityQuery</st>` <st c="31823">protocol</st> <st c="31833">is for.</st>
			<st c="31840">In our current</st> `<st c="31856">AppIntent</st>` <st c="31865">example, we let the user choose one of its calendars, so we need to build a struct named</st> `<st c="31955">CalendarQuery</st>` <st c="31968">that conforms</st> <st c="31983">to</st> `<st c="31986">EntityQuery</st>`<st c="31997">.</st>
			<st c="31998">Let’s look</st> <st c="32010">at a simple</st> `<st c="32022">CalendarQuery</st>` <st c="32035">example:</st>

```

struct CalendarQuery: EntityQuery {

    func entities(for identifiers: [CalendarEntity.ID])

    async throws -> [CalendarEntity] {

        allCalendars.filter { identifiers.contains($0.id) }

    }

    func suggestedEntities() async throws ->

    [CalendarEntity] {

        allCalendars

    }

    func defaultResult() async -> CalendarEntity? {

        nil

    }

}

```swift

			<st c="32352">Assume that</st> `<st c="32365">allCalendars</st>` <st c="32377">is an array containing all the</st> <st c="32409">user calendars.</st>
			<st c="32424">In this case,</st> `<st c="32439">CalendarQuery</st>` <st c="32452">implements three methods.</st> <st c="32479">Let’s quickly go</st> <st c="32496">over them:</st>

				*   `<st c="32506">entities(for identifiers:)</st>` <st c="32533">– This function returns calendar entities based on a list of IDs.</st> *<st c="32600">WidgetKit</st>* <st c="32610">uses it to show the</st> <st c="32630">selected calendar</st>
				*   `<st c="32647">suggestedEntities()</st>` <st c="32667">– This returns the list of entities in the</st> <st c="32711">pop-up menu</st>
				*   `<st c="32722">defaultResult()</st>` <st c="32738">– This returns the value when nothing</st> <st c="32777">is selected</st>

			<st c="32788">Now, let’s</st> <st c="32800">see how it looks (</st>*<st c="32818">Figure 5</st>**<st c="32827">.3</st>*<st c="32829">):</st>
			![Figure 5.3: The widget configuration menu](img/B21795_05_3.jpg)

			<st c="32885">Figure 5.3: The widget configuration menu</st>
			<st c="32926">In</st> *<st c="32930">Figure 5</st>**<st c="32938">.3</st>*<st c="32940">, we can see the</st> `<st c="33153">Bool</st>` <st c="33157">or</st> `<st c="33161">String</st>`<st c="33167">, and</st> *<st c="33173">WidgetKit</st>* <st c="33183">will create their corresponding</st> <st c="33215">input control.</st>
			<st c="33229">Let’s flip to the other side now and go to the widget UI to use the</st> `<st c="33298">AppEntity</st>` <st c="33307">the</st> <st c="33312">user selected.</st>
			<st c="33326">Using the AppEntity in our Widget</st>
			<st c="33360">Going back</st> <st c="33372">to our widget code, let’s examine the widget configuration</st> <st c="33431">code again:</st>

```

AppIntentConfiguration(kind: kind, <st c="33478">intent:</st>

<st c="33485">CalendarWidgetIntent.self,</st> provider:

    ConfigurableProvider(), content: { entry in

            ConfigurableWidgetView(entry: entry)

        })

```swift

			<st c="33606">The</st> `<st c="33611">AppIntentConfiguration</st>` <st c="33633">struct has an important property, which is the intent type it uses, and in this case, it is</st> `<st c="33726">CalendarWidgetIntent</st>`<st c="33746">. If we go back to the</st> *<st c="33769">Customize our widget</st>* <st c="33789">section, we can see that</st> `<st c="33815">CalendarWidgetIntent</st>` <st c="33835">contains all the information we need to present our widget according to the</st> <st c="33912">user configuration.</st>
			<st c="33931">Indeed, the timeline provider is now conforming to a different protocol,</st> `<st c="34005">AppIntentTimelineProvider</st>`<st c="34030">, which supports the intent configuration now.</st> <st c="34077">Let’s see how it creates</st> <st c="34102">a timeline:</st>

```

struct ConfigurableProvider: AppIntentTimelineProvider {

    func timeline(for configuration: CalendarWidgetIntent,

    in context: Context) async ->

        Timeline<ConfiguredNextEventEntry>

```swift

			<st c="34290">We can see that the timeline function inside</st> `<st c="34336">ConfigurableProvider</st>` <st c="34356">now receives the configuration parameter.</st> <st c="34399">From this point, all we need to do is use the information we have inside the configuration and create the relevant</st> <st c="34514">timeline entries.</st>
			<st c="34531">By now, we know how to set up a new widget, animate it, create its timeline, and even let the user configure it.</st> <st c="34645">Next, we’ll learn how to ensure our widgets stay up</st> <st c="34697">to date.</st>
			<st c="34705">Keeping our widgets up to date</st>
			<st c="34736">We have learned that we need to look ahead and create a timeline with different entries and dates to</st> <st c="34838">keep our widget up to date.</st> <st c="34866">But how does our widget work under</st> <st c="34901">the hood?</st>
			<st c="34910">Widgets don’t get any running time – once we generate the timeline entries,</st> *<st c="34987">WidgetCenter</st>* <st c="34999">generates their different views, keeps them persistently, and just switches them according to the</st> <st c="35098">provided timeline.</st>
			<st c="35116">So, there’s no way to update our widget without reloading the timeline, and when we created our timeline, we had to define its</st> <st c="35244">reload policy:</st>

```

let timeline = Timeline(entries: entries, <st c="35357">WidgetCenter</st> to reload the timeline immediately, due to data changes or any other alterations.

            <st c="35451">让我们看看它是如何发生的。</st>

            <st c="35477">使用 WidgetCenter 重新加载小部件</st>

            <st c="35515">在整个章节中，我经常提到</st> *<st c="35557">WidgetCenter</st>* <st c="35569">，但我还没有解释它的含义。</st>

            *<st c="35619">WidgetCenter</st>* 是一个包含当前使用配置的不同小部件信息的对象，它还提供了一个选项来 <st c="35758">重新加载它们。</st>

            <st c="35770">要使用 <st c="35778">WidgetCenter</st><st c="35790">，我们需要调用 <st c="35812">shared</st> <st c="35818">属性来访问其 <st c="35842">单例引用：</st>

```swift
 WidgetCenter.shared
```

            <st c="35882">WidgetCenter</st> 与我们迄今为止处理的其他代码之间的区别在于，我们是从应用中调用 <st c="35998">WidgetCenter</st> 而不是调用 <st c="36036">widget 扩展。</st>

            <st c="36053">让我们看看如何调用 <st c="36084">WidgetCenter</st> 来获取活动小部件的列表：</st>

```swift
 func getConfigurations() { <st c="36157">WidgetCenter.shared.getCurrentConfigurations</st> { result
      in
        if let widgets = try? result.get() {
            // handle our widgets
        }
    }
}
```

            <st c="36278">The</st> `<st c="36283">getCurrentConfigurations</st>` <st c="36307">函数使用闭包来返回一个活动小部件的数组。</st> <st c="36370">它们中的每一个都是 <st c="36394">WidgetInfo</st> <st c="36404">类型 – 一个包含有关特定 <st c="36450">配置小部件</st> <st c="36467">信息的结构。</st>

            <st c="36485">The</st> `<st c="36490">WidgetInfo</st>` <st c="36500">结构有三个属性 – kind, family，和 configuration：</st>

                +   `<st c="36566">kind</st>` <st c="36571">– 这是我们创建小部件配置时设置的字符串（再次查看 *<st c="36660">配置我们的</st> * *<st c="36676">小部件</st> * *<st c="36682">部分）。</st>

                +   `<st c="36692">family</st>` <st c="36699">– 小部件的家庭大小 – 小型、中型或大型。</st>

                +   `<st c="36758">configuration</st>` <st c="36772">– 包含用户配置信息的意图。</st> <st c="36832">The</st> `<st c="36836">configuration</st>` <st c="36849">属性是可选的。</st>

            <st c="36871">如果需要，我们可以使用这些信息来重新加载特定类型的小部件的时间线。</st> <st c="36964">例如，如果我们想重新加载具有 <st c="37023">MyWidget</st> <st c="37031">类型的小部件，我们需要调用</st> <st c="37049">以下内容：</st>

```swift
 WidgetCenter.shared.reloadTimelines(ofKind: "MyWidget")
```

            <st c="37119">注意，该函数说的是 <st c="37150">Timelines</st> <st c="37159">而不是 <st c="37168">Timeline</st>，因为可能存在多个相同类型的小部件。</st>

            <st c="37237">如果我们想重新加载我们应用中的所有小部件，我们可以调用 <st c="37296">reloadAllTimelines()</st> <st c="37316">函数：</st>

            `<st c="37326">WidgetCenter.shared.reloadAllTimelines()</st>`

            <st c="37367">在我们的小部件时间线中重新加载有几个很好的用例，例如当我们收到推送通知，或者当用户数据或设置发生变化时。</st> <st c="37526">如果你还记得，当我们讨论小部件时间线时，在</st> *<st c="37588">生成时间线</st>* <st c="37609">部分，我们提到了小部件每天可以重新加载的次数有一定的预算。</st> <st c="37729">但好消息是，如果我们的应用在前台或使用某些其他技术，例如在</st> <st c="37931">后台播放音频</st>，调用 `<st c="37767">reloadTimelines</st>` <st c="37782">或</st> `<st c="37786">reloadAllTimelines</st>` <st c="37804">函数不计入这个预算。</st>

            <st c="37946">在大多数</st> <st c="37955">情况下，</st> `<st c="37962">reloadTimelines</st>` <st c="37977">当更新后的数据已经在设备上或在我们的应用中时，工作得很好。</st> <st c="38051">但当我们本地的持久存储没有更新时，我们应该怎么办呢？</st>

            <st c="38120">我们执行一个网络请求,</st> <st c="38151">当然！</st>

            <st c="38161">前往网络获取更新</st>

            <st c="38191">在移动应用中，执行网络请求以更新本地数据是一种典型操作。</st> <st c="38281">但在小部件中它是如何工作的呢？</st> <st c="38289">does it work</st> <st c="38302">in widgets?</st>

            <st c="38313">让我们再次看看</st> `<st c="38332">getTimeline</st>` <st c="38343">函数：</st>

```swift
 func getTimeline(in context: Context, <st c="38465">getTimeline</st> function is an asynchronous function. It means that when we build our timeline, we can perform async operations such as open URL sessions and fetching data.
			<st c="38633">Let’s see an example of requesting the next</st> <st c="38678">calendar events:</st>

```

func getTimeline(in context: Context, completion: @escaping

    (Timeline<SimpleEntry>) -> Void) {

        var entries: [SimpleEntry] = [] <st c="38822">calendarService.fetchNextEvents { result in</st> switch result {

            case .success(let events):

                for event in events {

                    let entry = SimpleEntry(date:

                    event.alertTime, nextEvent:

                        event.title, nextEventTime:

                        event.date)

                    entries.append(entry)

                }

            case .failure(let error):

                print("Error fetching next events:

                \(error.localizedDescription)")

            }

        let timeline = Timeline(entries: entries, policy:

        .atEnd)

        completion(timeline)

}

    }

```swift

			<st c="39230">The</st> `<st c="39235">getTimeline</st>` <st c="39246">function implementation is similar to the previous</st> `<st c="39298">getTimeline</st>` <st c="39309">implementation we saw in the</st> *<st c="39339">Generating a timeline</st>* <st c="39360">section, and this time, we are fetching the</st> <st c="39405">events using the</st> `<st c="39422">calendarService</st>` <st c="39437">instance.</st> <st c="39448">The</st> `<st c="39452">calendarService</st>` <st c="39467">goes to our server and returns an array of events.</st> <st c="39519">Afterward, we loop the events, generate timeline entries, and return a timeline using the</st> `<st c="39609">completion</st>` <st c="39619">block.</st>
			<st c="39626">Up until now, we have seen how to create a widget, animate it, and ensure it is updated as much as we can.</st> <st c="39734">But if we want to make our widget shine, we need to add some</st> <st c="39795">user-interactive capabilities.</st>
			<st c="39825">Interacting with our widget</st>
			<st c="39853">Besides</st> <st c="39862">providing us with a glance at our app information, widgets are a great way to open our app in a specific location or manipulate data without even opening</st> <st c="40016">the app.</st>
			<st c="40024">As mobile developers, we sometimes wonder why implementing user interaction with a widget is such a big deal.</st> <st c="40135">After all, our users interact with our app daily, so why is it such a problem?</st> <st c="40214">But when we remember that widgets don’t really run, we can understand</st> <st c="40284">the challenge.</st>
			<st c="40298">The most basic way we have to allow interaction with our widgets is by using</st> <st c="40376">deep links.</st>
			<st c="40387">Opening a specific screen using links</st>
			<st c="40425">If you are not familiar with the concept of deep links, now is the time to straighten things out.</st> <st c="40524">A deep</st> <st c="40531">link is a link that opens our app on a specific screen.</st> <st c="40587">Today’s deep links format is similar to website URLs.</st> <st c="40641">For example, a deep link that opens our app in a specific calendar event screen can look something</st> <st c="40740">like this:</st>

```

http://www.myGreatCalendarAp.com/event/<eventID>/

```swift

			<st c="40800">To do that, the app needs to do</st> <st c="40833">three things:</st>

				*   *<st c="40846">Register to that specific domain</st>* <st c="40879">by placing a special JSON file on the</st> <st c="40918">relevant server</st>
				*   <st c="40933">Add the domain</st> *<st c="40949">entitlement</st>* <st c="40960">to</st> <st c="40964">our app</st>
				*   <st c="40971">Respond to</st> *<st c="40983">launching the app from a deep link</st>*<st c="41017">, parse the URL, and direct the user to the corresponding location within</st> <st c="41091">the app</st>

			<st c="41098">To learn more about deep links, I recommend reading about it in the Apple</st> <st c="41173">Developer website:</st>
			[<st c="41191">https://developer.apple.com/documentation/xcode/allowing-apps-and-websites-to-link-to-your-content</st>](https://developer.apple.com/documentation/xcode/allowing-apps-and-websites-to-link-to-your-content)
			<st c="41290">Going back to our topic, let’s see an example of adding a deep link to our</st> `<st c="41366">Next</st>` `<st c="41371">Event</st>` <st c="41376">widget:</st>

```

struct MyWidgetEntryView : View {

    var entry: Provider.Entry

    var body: some View {

        VStack {

            Text("时间:")

            Text(entry.nextEventTime, style: .time)

            Text("Next Event")

            Text(entry.nextEvent) <st c="41571">.widgetURL(URL(string:</st>

<st c="41593">"</st>https://www.myGreatCalendarApp.com/event/\(entry.eventID)/"<st c="41654">))</st> }

    }

}

```swift

			<st c="41663">In the</st> <st c="41671">preceding code example, we can see that we added a view modifier called</st> `<st c="41743">widgetURL</st>` <st c="41752">to the</st> `<st c="41760">Next Event</st>` `<st c="41770">Text</st>` <st c="41775">component.</st>
			<st c="41786">The</st> `<st c="41791">Next Event</st>` `<st c="41801">Text</st>` <st c="41806">component is indeed the view that accepts the user’s touch and opens the app in the specific deep link.</st> <st c="41911">But when the widget is small (</st>`<st c="41941">.systemSmall</st>`<st c="41953">), we can add only one deep link that is acceptable in the</st> <st c="42013">whole widget.</st>
			<st c="42026">In widgets with medium and large sizes, we can add multiple links to</st> <st c="42096">multiple components.</st>
			<st c="42116">It is worth noting that in terms of security, deep links work even when the device is locked, but require</st> *<st c="42223">FaceID</st>* <st c="42229">or passcode when tapping</st> <st c="42255">on them.</st>
			<st c="42263">In iOS 17, deep links are not the only option we have to allow users to interact with our widget, as it is possible to add buttons and toggles</st> <st c="42407">as well.</st>
			<st c="42415">Adding interactive capabilities</st>
			<st c="42447">Deep links</st> <st c="42459">in widgets are great, but they have one big problem – tapping on the widget always opens the app.</st> <st c="42557">But, sometimes, we want to update data or confirm something without entering the app.</st> <st c="42643">For example, maybe we want to accept a calendar invitation, approve a payment, or mark a task</st> <st c="42737">as completed.</st>
			<st c="42750">Because</st> <st c="42759">widgets don’t actually run, it’s a challenge to respond to a user interaction.</st> <st c="42838">Fortunately, there is a solution that we have already encountered in configurable widgets, and that’s</st> <st c="42940">App Intents.</st>
			<st c="42952">Using App Intents to add interactive widgets</st>
			<st c="42997">App Intents</st> <st c="43010">made especially for this kind of use case – to allow runtime for</st> <st c="43075">specific actions.</st>
			<st c="43092">So, how do App Intents help us?</st> <st c="43125">Let’s look at</st> *<st c="43139">Figure 5</st>**<st c="43147">.4</st>*<st c="43149">:</st>
			![Figure 5.4: App Intent event flow](img/B21795_05_4.jpg)

			<st c="43315">Figure 5.4: App Intent event flow</st>
			<st c="43348">In</st> *<st c="43352">Figure 5</st>**<st c="43360">.4</st>*<st c="43362">, we can see that the first stage is tapping on a button inside the widget.</st> <st c="43438">Starting from iOS 17, the</st> *<st c="43464">WidgetKit</st>* <st c="43474">framework has its own type of button, which can be linked to a specific</st> <st c="43546">App Intent:</st>

```

Button("Turn \(entry.isAlarm ? "Off" : "On") Alarm" , role:

nil, <st c="43802">entry.isAlaram</st> value. 但更有趣的是，我们还有一个额外的参数叫做 <st c="43904">Intent</st>，其中我们传递一个名为 <st c="43941">MyWidgetIntent</st> 的结构体，以及 <st c="43971">eventID</st>。

            <st c="43979">让我们谈谈应用意图，但这次是在</st> <st c="44046">用户交互</st>的上下文中。</st>

            <st c="44063">使用意图执行数据更改</st>

            我们已经说过，小部件不管理任何状态。因此，真正的 widget 状态是本地存储和 timeline provider 之间的一种组合。

            `<MyWidgetIntent>`接收一个`eventID`并负责联系`EventKit`并更新实际事件的警报信息。

            让我们看看`App Intent`：

```swift
 struct MyWidgetIntent: AppIntent {
    init() {
    }
    var eventID: String = ""
    init(eventID: String) {
        self.eventID = eventID
    }
    static var title: LocalizedStringResource = "Changing '
      event alarm settings." func perform() async throws -> some IntentResult {
        // working with EventKit and updating the event alarm data. return .result()
    }
}
```

            除了我们在*自定义我们的小部件*部分讨论的`LocalizedStringResource`静态属性之外，我们还有一个主要函数叫做`perform()`。`perform()`函数在用户点击与该 App Intent 关联的按钮时执行。请注意，`perform()`函数也是一个异步函数，它允许我们执行更重的任务，例如写入数据库或执行 URL 请求。

            一旦`perform()`函数完成执行，App Intent 就会触发*WidgetCenter*。

            更新小部件 UI

            现在本地存储已更新，是时候让*WidgetCenter*重新加载 Timeline Provider 了。我们应该已经熟悉这个过程了——Timeline Provider 获取相关的本地数据，并根据我们刚刚执行的变化构建时间线。最后，小部件 UI 正在被更新。

            如果我们想在不同的应用组件之间共享代码执行，使用 App Intent 也是很好的。例如，我们可以在我们的 widget 和 Siri Shortcut 之间共享逻辑代码。

            我们应该记住，即使小部件可能有它自己的运行时，将我们的代码分离以获得更好的灵活性和模块化仍然是一个好的实践。

            App Intent 的另一个很好的用途是控制小部件，这是 iOS 18 的另一个伟大补充。现在让我们来了解一下。

            添加控制小部件

            *<st c="46114">WidgetKit</st>* <st c="46125">提供了在启动器中展示我们的应用程序的方法。</st> <st c="46139">然而，它并不止于此。</st> <st c="46211">从 iOS 18 开始，我们可以在控制中心和锁屏上展示小部件，甚至可以将应用程序意图附加到 iPhone 15 Pro 的操作按钮上。</st>

            <st c="46375">将小部件添加到控制中心或锁屏上</st> <st c="46433">很容易。</st>

            <st c="46441">类似于我们通过遵循小部件协议创建小部件的方式，我们需要遵循</st> `<st c="46540">ControlWidget</st>` <st c="46553">协议来创建控制小部件。</st> <st c="46591">例如，想象我们有一个帮助我们控制智能家居配件的应用程序，我们想要创建一个可以打开和关闭我们家的主门的小部件。</st> <st c="46704">让我们从创建一个简单的控制小部件开始，命名为</st> `<st c="46803">MaindoorControl</st>`<st c="46818">：</st>

```swift
 struct MaindoorControl: ControlWidget {
    var body: some ControlWidgetConfiguration {
        StaticControlConfiguration(
            kind: "com.avitsadok.MaindoorControl"
        ) {
// rest of the widget goes here
        }
    }
}
```

            <st c="47012">在这个代码示例中，</st> `<st c="47039">MaindoorControl</st>` <st c="47054">小部件包含了从</st> `<st c="47075">body</st>` <st c="47079">变量到</st> `<st c="47106">ControlWidgetConfiguration</st>`<st c="47132">的时间点。</st> 这与我们如何在</st> *<st c="47204">配置我们的</st>* *<st c="47220">小部件</st>* <st c="47226">部分创建一个主屏幕小部件非常相似。</st>

            <st c="47235">在这种情况下，我们返回一个</st> `<st c="47279">StaticControlConfiguration</st>` <st c="47305">类型的实例，这意味着我们不向用户提供配置它的能力。</st> <st c="47376">然而，类似于主屏幕小部件，我们也可以通过返回</st> `<st c="47484">AppIntentControlConfiguration</st>` <st c="47513">（查看</st> *<st c="47527">自定义我们的</st>* *<st c="47541">小部件</st>* <st c="47547">部分）来添加一个用户可配置的控制小部件。</st>

            <st c="47557">我们可以添加两个控制小部件控件 – 一个切换和一个按钮。</st> <st c="47622">在我们的家庭主门状态控制的情况下，我们需要添加一个切换。</st> <st c="47702">让我们修改我们的代码并添加一个</st> `<st c="47734">ControlWidgetToggle</st>` <st c="47753">实例：</st>

```swift
 struct MaindoorControl: ControlWidget {
    var body: some ControlWidgetConfiguration {
        StaticControlConfiguration(
            kind: "com.avitsadok.MaindoorControl"
        ) { <st c="47918">ControlWidgetToggle(</st>
 <st c="47938">"Main door control",</st>
 <st c="47959">isOn: HouseManager.shared.isOpen,</st>
 <st c="47993">action: MaindoorIntent()</st>
 <st c="48018">) { isOn in</st>
 <st c="48030">Label(isOn ?</st> <st c="48044">"Opened" : "Closed",</st>
 <st c="48064">systemImage: isOn ?</st>
 <st c="48084">"door.left.hand.open" :</st>
 <st c="48108">"door.left.hand.closed")</st>
 <st c="48133">}</st> }
    }
}
```

            <st c="48141">在这个</st> <st c="48150">代码示例中，我们添加了</st> `<st c="48175">ControlWidgetToggle</st>`<st c="48194">，包含以下参数：</st>

                +   **<st c="48232">一个标题</st>** <st c="48240">– 在小部件画廊中显示的小部件标题。</st>

                +   `<st c="48310">isOn</st>` <st c="48315">– 在这里，我们将小部件连接到我们应用程序中的实际状态。</st> <st c="48369">。</st>

                +   `<st c="48377">action</st>` <st c="48384">– 当用户点击我们的控制小部件时运行的 App Intent。</st> <st c="48451">我们将在本节的后面部分介绍这一点。</st>

                +   `<st c="48545">标签</st>` <st c="48550">显示控制状态的标题和</st> <st c="48590">一个图像。</st>

            <st c="48599">小部件实例很简单。</st> <st c="48640">让我们看看它在我们的控制中心（</st>*<st c="48686">图 5</st>**<st c="48695">.5</st>*<st c="48697">）中的样子：</st>

            ![图 5.5：控制中心中的我们的控制小部件](img/B21795_05_5.jpg)

            <st c="48702">图 5.5：控制中心中的我们的控制小部件</st>

            *<st c="48754">图 5</st>**<st c="48763">.5</st>* <st c="48765">显示了控制中心的主门控制小部件。</st> <st c="48819">然而，我们的控制小部件还有另一个方面</st> <st c="48849">——将控制小部件连接到打开和关闭主门的操作。</st> <st c="48955">让我们看看我们在</st> `<st c="48973">MaindoorIntent</st>` <st c="48987">参数中看到的</st> `<st c="49009">action</st>` <st c="49015">结构：</st>

```swift
 struct MaindoorIntent: SetValueIntent {
    static let title: LocalizedStringResource = "Maindoor
      opening"
    @Parameter(title: "is open")
    var value: Bool
    func perform() throws -> some IntentResult {
      HouseManager.shared.isOpen = value
        return .result()
    }
}
```

            <st c="49275">在这个代码示例中，我们看到了</st> `<st c="49309">MaindoorIntent</st>` <st c="49323">实现。</st> <st c="49340">`<st c="49344">MaindoorIntent</st>` <st c="49358">结构符合`<st c="49385">SetValueIntent</st>` <st c="49399">协议，该协议包含一个我们可以设置的值。</st> <st c="49445">在这个例子中，值来自`<st c="49484">Bool</st>` <st c="49488">类型，我们可以用它来执行</st> <st c="49527">所需的操作。</st>

            <st c="49545">将控制小部件添加到我们的应用程序中涉及我们在添加主屏幕小部件和允许我们在小部件和其他</st> <st c="49725">应用程序组件之间共享代码的应用程序意图时看到的类似做法。</st>

            <st c="49740">摘要</st>

            <st c="49748">小部件是我们在 iOS 开发中可以与之交互的有趣且有趣的 UI 元素。</st> <st c="49828">它们提供流畅的 UI、出色的动画和可一览无余的用户体验。</st> <st c="49902">我们已经看到，每个 iOS 版本都添加了有趣的新小部件功能，使小部件比以往任何时候都更强大</st> <st c="50013">。</st>

            <st c="50023">在本章中，我们了解了小部件的概念、如何添加小部件、创建时间线以及添加用户可配置选项。</st> <st c="50158">此外，我们还学习了如何创建自定义动画，甚至添加用户交互。</st> *<st c="50238">WidgetKit</st>* <st c="50248">已经成为一个令人着迷的框架。</st> <st c="50297">在下一章中，我们将继续探讨如何改进用户体验，这次我们将使用</st> <st c="50396">SwiftUI 动画。</st>

```swift

```

```swift

```
