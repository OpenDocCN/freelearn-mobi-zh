# <st c="0">6</st>

# <st c="2">SwiftUI 动画和 SF 符号</st>

<st c="35">上一章讨论了一个令人愉快的话题——小部件。</st> <st c="98">它们的美学水平既愉快又有效，这使得与它们一起工作变得既有趣又简单。</st> <st c="197">现在，我们将通过</st> <st c="246">SwiftUI 动画</st>将这种感觉进一步深化。

<st c="265">动画是 iOS 开发中的一个关键主题，因为它丰富了体验，并使我们的应用程序更直观、更易于使用。</st> <st c="400">如果您习惯了 UIKit 动画，您会注意到 SwiftUI 动画与 UIKit 的方法不同，它提供了一个声明式 API 来动画化</st> <st c="554">状态变化。</st>

<st c="568">随着这些新挑战的到来，也带来了确保我们的逻辑状态和 UI</st> <st c="658">始终一致的机会。</st>

<st c="673">在本章中，我们将</st> <st c="702">执行以下操作：</st>

+   <st c="716">讨论动画的重要性</st> <st c="740">动画</st>

+   <st c="753">理解 SwiftUI</st> <st c="777">动画概念</st>

+   <st c="794">使用视图修改器和</st> `<st c="851">withAnimation</st>` <st c="864">函数</st>执行基本动画

+   <st c="873">执行高级动画，如过渡和</st> <st c="926">关键帧动画</st>

+   <st c="945">动画 SF 符号</st>

<st c="964">解释为什么我们需要动画听起来很奇怪，有些人可能会对这个话题皱眉头。</st> <st c="1058">因此，我们的首要任务是在我们开始在屏幕上移动一个像素之前，将这个话题从桌面上移开。</st> <st c="1156">那么，让我们回答以下问题——为什么我们需要关心</st> <st c="1221">动画？</st>

# <st c="1238">技术要求</st>

<st c="1261">对于本章，从</st> <st c="1345">App Store</st>下载 Xcode 版本 16.0 或更高版本是至关重要的。

<st c="1355">确保您正在使用最新的 macOS 版本（Ventura 或更高版本）。</st> <st c="1441">只需在 App Store 中搜索 Xcode，选择最新版本，然后继续下载。</st> <st c="1539">打开 Xcode 并完成出现的任何进一步设置说明。</st> <st c="1607">Xcode 完全运行后，您</st> <st c="1653">就可以开始了。</st>

<st c="1663">从以下</st> <st c="1708">GitHub 链接</st>下载示例代码：

[<st c="1720">https://github.com/PacktPublishing/Mastering-iOS-18-Development/tree/main/Chapter%206</st>](https://github.com/PacktPublishing/Mastering-iOS-18-Development/tree/main/Chapter%206)

# <st c="1806">动画的重要性</st>

<st c="1835">有些人可能认为执行动画主要是为了娱乐，实际上并不会真正影响一个</st> <st c="1931">应用程序的可用性。</st> <st c="1948">但事实是，动画在增强用户参与度和界面设计方面起着至关重要的作用，尤其是在移动应用程序中。</st> <st c="2087">以下是使用动画的一些好处：</st> <st c="2114">使用动画：</st>

+   首先，动画为用户的操作提供**视觉反馈**——当用户点击按钮时，按钮会变大，这有助于他们知道他们触摸了正确的位置。

+   动画还可以提供**指导和导航**——页面之间的转换表明我们是“向前”还是“向后”移动我们的流程。

+   动画还有助于**错误处理**——我们可以动画化错误消息和一般问题，并减少用户的挫败感。

+   最重要的是，在许多情况下，动画是应用品牌和独特性的**一部分**，并提供那种特殊的触感，加强用户与**应用**之间的联系。

现在我们已经了解了动画的重要性，让我们看看 SwiftUI 的声明式方法如何与该概念相一致。

# 理解 SwiftUI 动画的概念

对于一个来自 UIKit 并在 SwiftUI 中迈出第一步的开发者来说，在声明式框架中编写动画的概念可能会感觉有点尴尬。

[PRE0]

在这个示例中，我们修改了 `<st c="3436">sampleView</st>` 在 `<st c="3456">UIView</st>` 动画闭包中的 alpha 级别。

虽然这看起来很简单，但它带来一个显著的缺点——需要将动画动作同步到屏幕状态。

然而，在 *SwiftUI* 中，屏幕状态始终与 UI 保持同步，这对动画也是如此。

在 SwiftUI 中实现动画有几种方法；有些确实很简单，而另一些则允许我们提供高级和复杂的动画。

<st c="4333">让我们热身，并从一些</st> <st c="4368">基本动画</st> 开始。

# <st c="4385">执行基本</st><st c="4411">动画</st>

<st c="4414">理解 SwiftUI 动画工作原理的基本方法是将一个状态值与特定的</st> <st c="4527">动画流程</st> 相关联。

<st c="4542">在 SwiftUI 中执行基本动画有三种方式：</st> <st c="4595">。</st>

+   `<st c="4617">animation</st>` **<st c="4626">修饰符</st>** <st c="4635">– 将动画添加到一个</st> <st c="4663">特定的视图</st> 上

+   `<st c="4687">withAnimation</st>` **<st c="4700">全局函数</st>** <st c="4716">– 通过改变</st> <st c="4752">几个状态</st> 来执行动画

+   `<st c="4773">animation()</st>` **<st c="4784">方法</st>** <st c="4791">– 将动画附加到一个</st> <st c="4822">绑定值</st> 上

<st c="4835">开发者通常会感到困惑，并认为这里有一些重复 – 执行相同功能的不同方法。</st> <st c="4959">但事实是，这三种方法都服务于不同的目的和需求。</st> <st c="5027">取决于我们根据特定的代码结构和流程来决定合适的方法。</st> <st c="5110">有时，你可能想要对特定的视图执行特定的动画；偶尔，它可能是多个视图共享的体验。</st> <st c="5254">理解不同的用例可以帮助我们决定如何正确地执行动画。</st>

<st c="5349">让我们首先给一个特定的视图添加一个动画。</st> <st c="5390">。</st>

## <st c="5404">使用动画视图修饰符</st>

<st c="5438">动画</st> <st c="5453">视图修饰符的目标是在某个值发生变化时向特定的视图添加动画。</st> <st c="5482">以下是一个使用动画</st> <st c="5582">视图修饰符的示例：</st>

[PRE1]

<st c="5878">前面的代码通过向其宽度和高度添加</st> `<st c="5932">50</st>` <st c="5934">点来改变圆的尺寸，并且它是通过使用动画视图修饰符来做到这一点的。</st> <st c="6022">请注意，动画视图修饰符有一个值参数 – 动画修饰符监视其变化的值。</st> <st c="6139">在这种情况下，我们使用</st> `<st c="6164">width</st>` <st c="6169">状态变量。</st>

<st c="6185">动画</st> <st c="6200">视图修饰符非常适合在特定值发生变化时改变特定的视图。</st> <st c="6236">然而，在某些情况下，这种方法可能会令人困惑。</st> <st c="6283">在这种情况下，我们在代码的特定位置定义动画，但在另一个位置执行更改。</st> <st c="6346">此外，如果我们想要执行多个动画，使用动画视图修饰符可能会很麻烦。</st>

<st c="6565">如果我们想执行多个更改，我们可以使用</st> `<st c="6621">withAnimation:</st>` <st c="6635">函数。</st> <st c="6646">让我们看看如何</st> <st c="6663">利用它。</st>

## <st c="6674">使用 withAnimation 函数</st>

<st c="6707">在其基本形式中，</st> `<st c="6731">withAnimation:</st>` <st c="6745">函数接受一个闭包作为参数，并动画化</st> <st c="6799">在该闭包内所做的任何更改。</st> <st c="6828">通常，这是通过触发事件来完成的。</st> <st c="6837">让我们看看一个简单的</st> <st c="6886">代码示例：</st>

[PRE2]

<st c="7482">此代码示例在按钮被点击时同时动画化两个圆的位置。</st> <st c="7579">我们可以看到，与动画视图修改器不同，通过使用</st> `<st c="7645">withAnimation:</st>` <st c="7659">函数，我们可以更清晰、更简单地绑定动画的变化。</st> <st c="7719">和简单。</st>

<st c="7730">另一个</st> <st c="7739">优势是</st> `<st c="7754">withAnimation:</st>` <st c="7768">具有的是</st> <st c="7776">执行一个</st> **<st c="7801">完成代码</st>** <st c="7816">一旦动画结束。</st>

<st c="7840">让我们看一下以下</st> <st c="7876">代码示例：</st>

[PRE3]

<st c="8259">此代码创建了一个蓝色圆圈和一个写着</st> **<st c="8311">开始</st>**<st c="8316">的按钮。一旦用户点击按钮，圆圈</st> <st c="8360">动画化其位置，并在最后，按钮标题变为</st> **<st c="8427">重置</st>**<st c="8432">。然后，点击按钮将圆圈带回，并在</st> <st c="8490">反向动画结束时，按钮标题返回</st> <st c="8549">到</st> **<st c="8552">开始</st>**<st c="8557">。</st>

<st c="8558">动画中的完成块对于同步流程阶段至关重要。</st> <st c="8626">例如，折叠侧边栏并在结束时导航到新屏幕是一个出色的完成</st> <st c="8744">块使用的例子。</st>

<st c="8756">现在，是时候给我们的动画增添更多活力了。</st>

## <st c="8813">用弹簧动画给我们的动画增添一些活力</st>

<st c="8873">如果你已经尝试过你之前看到的代码示例，你可能已经注意到动画运行得相当流畅，但有点，嗯，无聊。</st> <st c="8971">这是因为动画是线性运行的，并不</st> <st c="9032">那么有趣。</st>

<st c="9105">尝试将以下参数添加到</st> <st c="9148">前面的示例中：</st>

[PRE4]

<st c="9292">在这个例子中，我们在</st> `<st c="9319">.bouncy(extraBounce: 0.3)</st>` <st c="9344">中添加了</st> `<st c="9352">withAnimation</st>` <st c="9365">函数。</st> <st c="9376">运行代码显示与之前相同的动画，但现在，当圆圈到达末端时它会弹跳。</st> <st c="9482">这是一个微小但重要的添加——弹跳效果为我们的动画增添了现实感，并可以提高</st> <st c="9597">用户参与度。

<st c="9613">我们可以添加到动画中的有趣视觉过渡有几个。</st> <st c="9624">例如，我们可以使用</st> `<st c="9750">.</st>``<st c="9751">smooth</st>` <st c="9758">函数来使弹跳更平滑：</st>

[PRE5]

<st c="9809">我们还可以通过使用小的</st> <st c="9893">弹跳量来使弹跳更迅速，通过使动画更快：</st>

[PRE6]

<st c="9930">建议查看 Apple 的文档以发现更多我们可以轻松应用到我们</st> <st c="10050">动画中的视觉过渡：</st> [<st c="10062">https://developer.apple.com/documentation/swiftui/animation</st>](https://developer.apple.com/documentation/swiftui/animation)<st c="10121">。</st>

<st c="10122">到目前为止，我们只执行了非常基本的动画。</st> <st c="10172">但现代应用需要现代体验。</st> <st c="10216">让我们继续探索一些创建</st> <st c="10258">高级动画的更多方法。

# <st c="10278">执行高级动画</st>

<st c="10309">我们提到过渡对于指导和导航非常出色，这部分概念的一部分是</st> <st c="10407">在我们的画布上提供关于进入和离开视图的清晰度。</st> <st c="10475">从底部滑动视图可以提供抽屉打开和关闭的感觉，缩放视图可以直观地表示一个</st> <st c="10624">正在进行的过程的进度。

<st c="10640">到目前为止，我们已经讨论了如何将视图从一个状态动画化到另一个状态。</st> <st c="10715">现在，我们将探索过渡——一种在视图出现</st> <st c="10790">或消失时动画化视图的方法。

## <st c="10803">执行过渡</st>

<st c="10826">实现</st> <st c="10840">视图过渡很简单——我们有一些很好的内置过渡可供选择，如果还不够，我们还可以创建一个</st> <st c="10970">自定义过渡。

<st c="10988">让我们从一些基本的、内置的过渡开始。</st> <st c="11018">过渡。</st>

### <st c="11039">实现内置过渡</st>

<st c="11073">要添加</st> <st c="11081">过渡效果，我们应该使用</st> `<st c="11113">transition</st>` <st c="11123">修饰符与我们要动画化的特定视图，通过使用</st> `<st c="11200">withAnimation</st>` <st c="11213">函数来触发它，该函数我们在</st> *<st c="11247">使用 withAnimation 函数</st>* *<st c="11271">部分</st>中了解过。

<st c="11288">这是一个简单的</st> <st c="11324">过渡中的幻灯片</st>示例：

[PRE7]

<st c="11586">代码示例由</st> `<st c="11616">VStack</st>` <st c="11622">和一个按钮以及文本组成。</st> <st c="11647">我们还有一个状态来决定文本是可见还是隐藏。</st>

<st c="11718">点击按钮使用</st> `<st c="11768">withAnimation</st>` <st c="11781">函数显示文本。</st> <st c="11792">但是文本</st> <st c="11805">也有一个描述其如何出现的过渡视图修饰符，在这种情况下，使用一个</st> <st c="11909">滑动进入过渡。</st>

<st c="11931">过渡视图修饰符描述了视图的显示方式和消失方式。</st> <st c="12015">。</st>

<st c="12028">The</st> `<st c="12033">slide</st>` <st c="12038">transition inserts the view by moving it from the leading edge and removing it toward the trailing edge.</st> <st c="12144">请注意，滑动过渡的方向不能更改，它们由 SwiftUI 框架设置。</st> <st c="12248">但是，我们还可以使用几个其他过渡来实现我们</st> <st c="12318">期望的行为：</st>

+   `<st c="12335">move</st>`<st c="12340">: 将视图移动到/从特定边缘：</st>

    [PRE8]

+   `<st c="12442">scale</st>`<st c="12448">: 以特定数量和从特定锚点缩放视图：</st>

    [PRE9]

+   `<st c="12591">opacity</st>`<st c="12599">: 对视图执行“淡入/淡出”效果：</st> <st c="12637">。</st>

    [PRE10]

<st c="12696">这些类型的过渡在 Apple 网站和 SDK 中有很好的文档记录，我们也可以通过章节的</st> <st c="12819">GitHub 仓库</st> 来尝试它们。</st>

<st c="12837">需要注意的是，我们可以使用这些过渡来显示和隐藏动画。</st> <st c="12924">然而，在某些情况下，我们可能更喜欢显示和隐藏时使用不同的动画。</st> <st c="13014">隐藏和显示使用不同的动画称为</st> **<st c="13079">非对称过渡</st>**<st c="13100">。让我们</st> <st c="13108">看看一个代码示例</st> <st c="13127">来演示这一点：</st>

[PRE11]

Text("缩放和滑动") <st c="13545">.transition(.scale.combined(with:</st>

<st c="13578">.slide))</st>

[PRE12]

.transition(.scale.combined(with: .slide.combined(with:

.opacity)))

[PRE13]

struct ViewRotationModifier: ViewModifier {

    let angle: Angle

    let opacity: CGFloat

    let scale: CGFloat

    func body(content: Content) -> some View {

        content

            .rotationEffect(angle)

            .scaleEffect(scale)

            .opacity(opacity)

    }

}

[PRE14]

let rotate = AnyTransition.modifier(

    active: <st c="15350">ViewRotationModifier</st>(angle: .degrees(360),

    opacity: 0.0, scale: 0.0),

    identity: <st c="15431">ViewRotationModifier</st>(angle: .degrees(0),

    opacity: 1.0, scale: 1.0)

)

[PRE15]

struct CustomizedTransitionView: View {

    @State private var showRectangle: Bool = false

    var body: some View {

        VStack {

            Spacer()

            if showRectangle {

                Rectangle()

                    .frame(width: 100, height: 100)

                    .foregroundColor(.blue) <st c="15907">.transition(rotate)</st> }

            Spacer()

            Button("插入矩形") {

                withAnimation {

                    showRectangle.toggle()

                }

            }

        }

    }

}

[PRE16]

struct AnimationProperties {

    var scale = 1.0

    var opacity = 1.0

}

struct KeyFrameAnimations: View {

    var body: some View {

        Circle()

            .foregroundColor(.red)

            .frame(width:100, height:100)

            .<st c="18911">关键帧动画器</st>(initialValue:

            AnimationProperties(), repeating: true) {

                content, value in

                content

                    .opacity(value.opacity)

                    .scaleEffect(value.scale)

            } <st c="19064">关键帧</st>: { _ in <st c="19083">KeyframeTrack</st>(\.scale) { <st c="19109">CubicKeyframe</st>(0.7, duration: 0.8) <st c="19144">CubicKeyframe</st>(1.0,

                                duration: 0.8)

                } <st c="19181">关键帧轨道</st>(\.opacity) { <st c="19209">CubicKeyframe</st>(0.3, duration: 0.8) <st c="19244">CubicKeyframe</st>(1.0, duration: 0.8)

                }

            }

    }

}

[PRE17]

CubicKeyframe(0.5, duration: 0.2, <st c="21871">起始速度</st>: 0.5, <st c="21892">结束速度</st>: 0.8)

CubicKeyframe(0.7, duration: 0.5)

[PRE18]

<st c="23463">let duration: TimeInterval = 1.8</st> var body: some View {

        Circle()

            .foregroundColor(.red)

            .frame(width:100, height:100)

            .keyframeAnimator(initialValue:

            AnimationProperties(), repeating: true) {

            content, value in

                content

                    .opacity(value.opacity)

                    .scaleEffect(value.scale)

            } keyframes: { _ in

                KeyframeTrack(\.scale) {

                    CubicKeyframe(0.7, <st c="23795">持续时间: 0.5 *</st>

<st c="23810">持续时间</st>)

                    CubicKeyframe(1.0,

                    duration: <st c="23851">0.5 * duration</st>)

                }

                KeyframeTrack(\.opacity) {

                    CubicKeyframe(0.3, <st c="23916">持续时间: 0.5 *</st>

<st c="23931">持续时间</st>)

                    CubicKeyframe(1.0, <st c="23962">持续时间: 0.5 *</st>

<st c="23977">持续时间</st>)

                }

            }

[PRE19]

var body: some View {

        HStack { <st c="25519">Image(systemName:</st>

<st c="25536">"alarm.waves.left.and.right.fill")</st> Text("闹钟")

        }.font(.system(size: 30))

    }

[PRE20]

struct SFSymbolsAnimationView: View {

    @State private var animate = false

    var body: some View {

        HStack {

            Image(systemName:

            "alarm.waves.left.and.right.fill") <st c="27851">.symbolEffect(.bounce, options: .repeating,</st>

<st c="27894">值: animate)</st> Text("10:30")

        }.font(.system(size: 40))

            .onTapGesture {

            animate = true

        }

    }

}

[PRE21]

Image(systemName: "person.2.wave.2")

[PRE22]

Image(systemName: "person.2.wave.2")

                .foregroundStyle(.棕色, .蓝色)

[PRE23]

Image(systemName: "person.3.sequence.fill")

                .foregroundStyle(.red, .blue, .<st c="29887">棕色</st>)

[PRE24]

Image(systemName: "arrowshape.turn.up.forward") <st c="31224">.environment(\.layoutDirection, .rightToLeft)</st>

[PRE25]

Image(systemName: "character.book.closed") <st c="32137">.environment(\.locale, .init(identifier: "he"))</st>

[PRE26]
