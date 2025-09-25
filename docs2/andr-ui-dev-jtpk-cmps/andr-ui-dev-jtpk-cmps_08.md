# 第六章：整合各个部分

前几章探讨了 Jetpack Compose 的各个方面。例如，*第二章*，*理解声明式范式*，将传统的视图系统与组合函数进行了比较，并解释了声明式方法的好处。*第四章*，*布局 UI 元素*，使你对一些内置布局组合器，如`Box()`、`Row()`和`Column()`，有了扎实的理解。在*第五章*，*管理组合函数的状态*中，我们探讨了状态及其在 Compose 应用中的重要作用。

现在，是时候看看这些关键元素如何在现实世界的应用中协同工作了。在本章中，您将学习如何为主题化 Compose 应用。我们还将查看`Scaffold()`，这是一个集成 UI 元素，它收集了与活动最初相关的一些概念，如工具栏和菜单，我们将学习如何添加基于屏幕的导航。

在本章中，我们将涵盖以下主题：

+   为 Compose 应用添加样式

+   集成工具栏和菜单

+   添加导航

我们将首先为 Compose 应用设置一个自定义主题。您可以定义很多颜色、形状和文本样式，内置的 Material 组合器在绘制自身时将使用这些样式。我还会向您展示在添加依赖于应用主题的额外 Jetpack 组件时需要注意的事项，例如*Jetpack Core Splashscreen*。

在下一节，*集成工具栏和菜单*中，我们将向您介绍应用栏和选项菜单。您还将学习如何创建 snack bars。

在最后的主体部分，*添加导航*，我将向您展示如何将您的应用结构化为屏幕。我们将使用*Jetpack Navigation*的 Compose 版本在它们之间进行导航。

# 技术要求

本章包含一个示例应用，`ComposeUnitConverter`，如下截图所示：

![图 6.1 – ComposeUnitConverter 应用](img/B17505_06_1.jpg)

图 6.1 – ComposeUnitConverter 应用

请参阅*第一章*，*构建您的第一个 Compose 应用*中的*技术要求*部分，了解如何安装和设置 Android Studio，以及如何获取本书的配套仓库。

本章的所有代码文件都可以在 GitHub 上找到，地址为[`github.com/PacktPublishing/Android-UI-Development-with-Jetpack-Compose/tree/main/chapter_06`](https://github.com/PacktPublishing/Android-UI-Development-with-Jetpack-Compose/tree/main/chapter_06)。

# 为 Compose 应用添加样式

你的大部分 Compose UI 可能都会使用来自`androidx.compose.material`包的内置可组合函数。它们实现了被称为**Material Design**的设计语言及其继任者**Material You**（随着 Android 12 的推出而引入）。Material You 是 Android 的原生设计语言，尽管它也将在其他平台上可用。它扩展了笔、纸和卡片的概念，并大量使用基于网格的布局、响应式动画和过渡，以及填充和深度效果。Material You 提倡更大的按钮和圆角。可以从用户的壁纸生成自定义颜色主题。

## 定义颜色、形状和文本样式

虽然应用应该当然尊重系统和使用者对视觉外观的偏好，但你可能希望添加反映你的品牌或企业身份的颜色、形状或文本样式。那么，你如何修改内置的 Material 可组合函数的外观？

材料主题化的主要入口点是`MaterialTheme()`。这个可组合函数可以接收自定义颜色、形状和文本样式。如果没有设置值，则使用相应的默认值（`MaterialTheme.colors`、`MaterialTheme.typography`或`MaterialTheme.shapes`）。以下主题设置了自定义颜色，但将文本样式和形状保留为默认值：

```kt
@Composable
fun ComposeUnitConverterTheme(
  darkTheme: Boolean = isSystemInDarkTheme(),
  content: @Composable () -> Unit
) {
  val colors = if (darkTheme) {
    DarkColorPalette
  } else {
    LightColorPalette
  }
  MaterialTheme(
    colors = colors,
    content = content
  )
}
```

`isSystemInDarkTheme()`可组合函数检测设备当前是否正在使用深色主题。你的应用应该使用适合这种配置的颜色。我的例子有两个调色板，`DarkColorPalette`和`LightColorPalette`。以下是后者的定义方式：

```kt
private val LightColorPalette = lightColors(
  primary = AndroidGreen,
  primaryVariant = AndroidGreenDark,
  secondary = Orange,
  secondaryVariant = OrangeDark
)
```

`lightColors()`是`androidx.compose.material`包中的一个顶级函数。它为 Material 颜色规范提供了完整的颜色定义。你可以在[`material.io/design/color/the-color-system.html#color-theme-creation`](https://material.io/design/color/the-color-system.html#color-theme-creation)找到更多关于此的信息。`LightColorPalette`覆盖了默认的 primary、`primaryVariant`、secondary 和`secondaryVariant`值。所有其他（例如，`background`、`surface`和`onPrimary`）保持不变。

`primary`将在你的应用屏幕和组件中显示得最频繁。使用`secondary`，你可以突出和区分你的应用。例如，它用于单选按钮。开关的选中拇指颜色是`secondaryVariant`，而未选中拇指颜色则来自`surface`。

小贴士

材料可组合函数通常从名为`colors()`的可组合函数接收默认颜色，这些函数属于它们的伴随`…Defaults`对象。例如，如果没有传递颜色参数给`Switch()`，则`Switch()`会调用`SwitchDefaults.colors()`。通过查看这些`colors()`函数，你可以找出在你的主题中应该设置哪个颜色属性。

你可能想知道我是如何定义例如`AndroidGreen`的。实现这一点的最简单方法是这样的：

```kt
val AndroidGreen = Color(0xFF3DDC84)
```

如果你的应用不需要其他库或组件，这些库或组件依赖于传统的 Android 主题系统，这将非常有效。我们将在*使用基于资源的主题*部分转向这些场景。

除了颜色之外，`MaterialTheme()`还允许你提供替代形状。形状引导注意力并传达状态。基于大小，Material 可组合项被分组到形状类别中：

+   小型（按钮、snack bars、工具提示等）

+   中型（卡片、对话框、菜单等）

+   大型（纸张和抽屉等）

要传递给`MaterialTheme()`的替代形状集，你必须实例化`androidx.compose.material.Shapes`，并为你想要修改的类别（`small`、`medium`和`large`）提供`androidx.compose.foundation.shape.CornerBasedShape`抽象类的实现。`AbsoluteCutCornerShape`、`CutCornerShape`、`AbsoluteRoundedCornerShape`和`RoundedCornerShape`是`CornerBasedShape`的直接子类。

以下截图显示了一个带有切角的按钮。虽然这使按钮看起来不那么熟悉，但它给你的应用带来了一种独特的风格。然而，你应该确保你想要添加这个：

![Figure 6.2 – A button with cut corners![img/B17505_06_2.jpg](img/B17505_06_2.jpg)

图 6.2 – 带有切角的按钮

要实现这一点，只需在调用`MaterialTheme()`时添加以下行：

```kt
shapes = Shapes(small = CutCornerShape(8.dp)),
```

你可以在[`material.io/design/shape/applying-shape-to-ui.html#shape-scheme`](https://material.io/design/shape/applying-shape-to-ui.html#shape-scheme)找到有关将形状应用于 UI 的更多信息。

要更改 Material 可组合函数使用的文本样式，你需要将`androidx.compose.material.Typography`的实例传递给`MaterialTheme()`。`Typography`接收许多参数，例如`h1`、`subtitle1`、`body1`、`button`和`caption`。所有这些都是`androidx.compose.ui.text.TextStyle`的实例。如果你没有为参数传递值，则使用默认值。

以下代码块增加了按钮的文本大小：

```kt
typography = Typography(button = TextStyle(fontSize =
                                           24.sp)),
```

如果你将此行添加到`MaterialTheme()`的调用中，使用你的主题的所有按钮的文本将高 24 个缩放无关像素。但如何设置主题？为了确保你的完整 Compose UI 使用它，你应该尽早调用你的主题：

```kt
class ComposeUnitConverterActivity : ComponentActivity() {
  override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    val factory = …
    setContent {
      ComposeUnitConverter(factory)
    }
  }
}
```

在我的例子中，`ComposeUnitConverter()`是应用的可组合 UI 层次结构的根，因为它在`setContent {}`内部被调用：

```kt
@Composable
fun ComposeUnitConverter(factory: ViewModelFactory) {
  …
  ComposeUnitConverterTheme {
    Scaffold( ...
```

`ComposeUnitConverter()`立即委托给`ComposeUnitConverterTheme {}`，它接收剩余的 UI 作为其内容。`Scaffold()`是现实世界 Compose UI 的骨架。我们将在*集成工具栏和菜单*部分更详细地探讨这一点。

如果你需要以不同的方式样式化应用的部分，你可以通过覆盖父主题（*图 6.3*）来嵌套主题。让我们看看这是如何工作的：

```kt
@Composable
@Preview
fun MaterialThemeDemo() {
  MaterialTheme(
    typography = Typography(
      h1 = TextStyle(color = Color.Red)
    )
  ) {
    Row {
      Text(
        text"= "He"lo",
        style = MaterialTheme.typography.h1
      )
      Spacer(modifier = Modifier.width(2.dp))
      MaterialTheme(
        typography = Typography(
          h1 = TextStyle(color = Color.Blue)
        )
      ) {
        Text(
          text"= "Comp"se",
          style = MaterialTheme.typography.h1
        )
      }
    }
  }
}
```

在前面的代码片段中，基本主题配置了任何被样式化为`h1`的文本，使其显示为红色。第二个`Text()`使用嵌套主题将`h1`样式化为蓝色。因此，它覆盖了父主题：

![图 6.3 – 嵌套主题![图片 B17505_06_3.jpg](img/B17505_06_3.jpg)

图 6.3 – 嵌套主题

注意事项

您应用的所有部分都必须具有一致的外观。因此，您应谨慎使用嵌套主题。

在下一节中，我们将继续探讨样式和主题。我们将查看如何在清单文件中设置主题，以及库如何影响您定义 Compose 主题的方式。

## 使用基于资源的主题

自 API 级别 1 以来，Android 上就存在应用样式或主题。它基于资源文件。从概念上讲，样式和主题之间有一个区别。**样式**是一组属性，用于指定单个视图的外观（例如，字体颜色、字体大小或背景颜色）。因此，样式对可组合函数不重要。**主题**也是一组属性，但它应用于整个应用、活动或视图层次结构。Compose 应用中的许多元素由 Material 可组合元素提供；对于它们，基于资源的主题也不重要。然而，主题可以将样式应用于非视图元素，如状态栏和窗口背景。这可能对 Compose 应用相关。

样式和主题在`res/values`目录内的 XML 文件中声明，通常根据内容命名为`styles.xml`和`themes.xml`。主题通过清单文件中`<application />`或`<activity />`标签的`android:theme`属性应用于应用或活动。如果没有它们接收主题，`ComposeUnitConverter`将如下所示：

![图 6.4 – 组合单元转换器显示了一个额外的标题栏![图片 B17505_06_4.jpg](img/B17505_06_4.jpg)

图 6.4 – 组合单元转换器显示了一个额外的标题栏

为了避免不想要的额外标题栏，Compose 应用必须配置一个没有操作栏的主题，例如`Theme.AppCompat.DayNight.NoActionBar`，使用`android:theme="@style/..."`为`<application />`或`<activity />`。这样，`ComposeUnitConverter`看起来就像图 6.1。您注意到状态栏有深灰色背景吗？

当使用`Theme.AppCompat.DayNight`时，状态栏从`colorPrimaryDark`主题属性（或自 API 级别 21 以来的`android:statusBarColor`）接收其背景颜色。如果没有指定值，则使用默认值。因此，为了确保状态栏以适合剩余 UI 元素的颜色显示，您必须在`res/values`中添加一个名为`themes.xml`的文件：

```kt
<resources>
  <style name="Theme.ComposeUnitConverter"
         parent="Theme.AppCompat.DayNight.NoActionBar">
    <item
     name="colorPrimaryDark">@color/android_green_dark
    </item>
  </style>
</resources>
```

在清单文件中，`android:theme`的值必须更改为`@style/Theme.ComposeUnitConverter`。`@color/android_green_dark`代表颜色。除了这个表达式，你也可以直接传递值；例如，`#FF20B261`。然而，将它们存储在`res/values`目录下的名为`colors.xml`的文件中是最佳实践：

```kt
<resources>
  <color name="android_green_dark">#FF20B261</color>
  <color name="orange_dark">#FFCC8400</color>
</resources>
```

这样，你可以为深色主题分配不同的值。以下版本的`themes.xml`应该放在`res/values-night`中：

```kt
<resources>
  <style name="Theme.ComposeUnitConverter"
         parent="Theme.AppCompat.DayNight.NoActionBar">
    <item name="colorPrimaryDark">@color/orange_dark</item>
  </style>
</resources>
```

现在状态栏的背景颜色与剩余的 UI 元素相匹配。但是，我们需要在两个地方定义颜色：`colors.xml`和 Compose 主题。幸运的是，这很容易解决。通常，我们传递一个字面量，如下所示：

```kt
val AndroidGreenDark = Color(0xFF20B261)
```

而不是这样做，我们应该从资源中获取值。`colorResource()`可组合函数属于`androidx.compose.ui.res`包。它返回与 ID 标识的资源相关联的颜色。

以下调色板未指定一个`二级`颜色：

```kt
private val LightColorPalette = lightColors(
  primary = AndroidGreen,
  primaryVariant = AndroidGreenDark,
  secondaryVariant = OrangeDark
)
```

使用`colorResource()`添加颜色的工作方式如下：

```kt
@Composable
fun ComposeUnitConverterTheme(
  darkTheme: Boolean = isSystemInDarkTheme(),
  content: @Composable () -> Unit
) {
  val colors = if (darkTheme) {
    DarkColorPalette
  } else {
    LightColorPalette.copy(secondary = colorResource(
      id = R.color.orange_dark))
  }
  MaterialTheme(
    colors = colors,
    …
```

你在*定义颜色、形状和文本样式*部分看到了大部分内容。重要的区别在于我使用`copy()`创建了一个修改版的`LightColorPalette`（带有二级颜色），然后将其传递给`MaterialTheme()`。如果你将所有颜色存储在`colors.xml`中，你应该完全在你的主题可组合中创建你的调色板。

正如你所见，你可能需要为基于资源的主题提供一些值，这取决于你想要如何强烈地品牌你的应用。此外，某些非 Compose Jetpack 库也使用主题，例如*Jetpack Core Splashscreen*。这个组件使得 Android 12 的高级启动屏幕功能在旧平台上可用。启动屏幕的图像和颜色通过主题属性进行配置。库要求启动活动的主题具有`Theme.SplashScreen`作为其父主题。此外，主题必须提供`postSplashScreenTheme`属性，该属性指向启动屏幕关闭后要使用的主题。你可以在 Android 开发者文档中找到有关启动屏幕的更多信息：[`developer.android.com/guide/topics/ui/splash-screen`](https://developer.android.com/guide/topics/ui/splash-screen)。

小贴士

为了确保颜色的一致使用，如果多个组件依赖于基于资源的主题，`colors.xml`文件应该是你应用中的单一真相来源。

这就结束了我们对 Compose 主题的探讨。在下一节中，我们将转向一个重要的集成 UI 元素，称为`Scaffold()`，它充当内容的框架，提供顶部和底部栏、导航和操作的支持。

# 集成工具栏和菜单

早期安卓版本并不知道动作或应用栏的存在。它们是在 API 级别 11（蜂巢）中引入的。另一方面，选项菜单从一开始就存在，但需要通过按下专用硬件按钮来打开，并显示在屏幕底部。在安卓 3 中，它移动到了顶部，并变成了一个垂直列表。一些元素可以作为动作永久显示。从某种意义上说，选项菜单和动作栏合并了。最初，动作栏的所有方面都由宿主活动处理，但`AppCompat`支持库引入了另一种实现（`getSupportActionBar()`）。它至今仍作为 Jetpack 的一部分被广泛使用。

## 使用 Scaffold()来结构化你的屏幕

Jetpack Compose 包含几个与 Material Design 或 Material You 规范紧密相关的应用栏实现。它们可以通过`Scaffold()`添加到 Compose UI 中，这是一个充当应用框架或骨骼的可组合函数。以下代码片段是`ComposeUnitConverter` UI 的根。它设置主题，然后委托给`Scaffold()`：

```kt
@Composable
fun ComposeUnitConverter(factory: ViewModelFactory) {
  val navController = rememberNavController()
  val menuItems = listOf("Item #1", "Item #2")
  val scaffoldState = rememberScaffoldState()
  val snackbarCoroutineScope = rememberCoroutineScope()
  ComposeUnitConverterTheme {
    Scaffold(scaffoldState = scaffoldState,
      topBar = {
        ComposeUnitConverterTopBar(menuItems) { s ->
          snackbarCoroutineScope.launch {
            scaffoldState.snackbarHostState.showSnackbar(s)
          }
        }
      },
      bottomBar = {
        ComposeUnitConverterBottomBar(navController)
      }
    ) {
      ComposeUnitConverterNavHost(
        navController = navController,
        factory = factory
      )
    }
  }
}
```

`Scaffold()`实现了基本的 Material Design 视觉布局结构。你可以添加其他几个 Material 可组合函数，如`TopAppBar()`或`BottomNavigation()`。谷歌称这为`Scaffold()`可能需要记住不同的状态。你可以传递一个`ScaffoldState`，它可以通过`rememberScaffoldState()`创建。

我的示例使用`ScaffoldState`来显示一个 snackbar，这是一个出现在屏幕底部的简短临时消息。由于`showSnackbar()`是一个挂起函数，它必须从协程或另一个挂起函数中调用。因此，我们必须使用`rememberCoroutineScope()`创建和记住一个`CoroutineScope`，并调用其`launch {}`函数。

在下一节中，我将向你展示如何创建一个带有选项菜单的顶部应用栏。

## 创建顶部应用栏

屏幕顶部的应用栏是通过`TopAppBar()`实现的。你可以在这里提供一个导航图标、标题和动作列表：

```kt
@Composable
fun ComposeUnitConverterTopBar(menuItems: List<String>, 
                               onClick: (String) -> Unit) {
  var menuOpened by remember { mutableStateOf(false) }
  TopAppBar(title = {
    Text(text = stringResource(id = R.string.app_name))
  },
    actions = {
      Box {
        IconButton(onClick = {
          menuOpened = true
        }) {
          Icon(Icons.Default.MoreVert, "")
        }
        DropdownMenu(expanded = menuOpened,
          onDismissRequest = {
            menuOpened = false
          }) {
          menuItems.forEachIndexed { index, s ->
            if (index > 0) Divider()
            DropdownMenuItem(onClick = {
              menuOpened = false
              onClick(s)
            }) {
              Text(s)
            }
          }
        }
      }
    }
  )
}
```

`TopAppBar()`没有针对选项菜单的特定 API。相反，菜单被当作普通动作处理。动作通常是`IconButton()`可组合函数。它们以水平行的形式显示在应用栏的末尾。一个`IconButton()`接收一个`onClick`回调和一个可选的`enabled`参数，该参数控制用户是否可以与 UI 元素交互。

在我的示例中，回调仅将一个`Boolean`可变状态（`menuOpened`）设置为`false`。正如你很快就会看到的，这将关闭菜单。`content`（通常是一个图标）被绘制在按钮内部。`Icon()`可组合函数接收一个`ImageVector`实例和一个内容描述。你可以从资源中获取图标数据，但如果可能的话，应使用预定义的图形 – 在我的示例中，`Icons.Default.MoreVert`。接下来，让我们学习如何显示菜单。

Material Design 下拉菜单 (`DropdownMenu()`) 允许您紧凑地显示多个选择。它通常在您与另一个元素（如按钮）交互时出现。我的例子将 `DropdownMenu()` 放置在 `Box()` 中，并使用 `IconButton()` 确定屏幕上的位置。展开参数使菜单可见（打开）或不可见（关闭）。当用户请求关闭菜单，例如通过在菜单边界外轻触时，会调用 `onDismissRequest`。

内容应包含 `DropdownMenuItem()` 组合器。当点击相应的菜单项时，会调用 `onClick`。您的代码必须确保菜单被关闭。如果可能，您应该将执行的域逻辑作为参数传递，以便使您的代码可重用且无状态。在我的例子中，显示了一个 snack bar。

这就结束了我们对顶级应用栏的探讨。在下一节中，我将向您展示如何使用 `BottomNavigation()` 通过 Jetpack Navigation 的 Compose 版本在不同的屏幕间进行导航。

请注意

要在您的应用中使用 Jetpack Navigation 的 Compose 版本，您必须将 `androidx.navigation:navigation-compose` 的实现依赖项添加到您的模块级 `build.gradle` 文件中。

# 添加导航

`Scaffold()` 允许您使用其 `bottomBar` 参数在屏幕底部放置内容。例如，这可以是一个 `BottomAppBar()`。Material Design 底部应用栏提供访问底部导航抽屉以及最多四个操作，包括一个浮动操作按钮。`ComposeUnitConverter` 则添加了 `BottomNavigation()`。Material Design 底部导航栏允许在应用中的主要目的地之间进行切换。

## 定义屏幕

从概念上讲，主要目的地是 *屏幕*，在 Jetpack Compose 之前，这可能是显示在单独的活动中的。以下是 `ComposeUnitConverter` 中定义屏幕的方式：

```kt
sealed class ComposeUnitConverterScreen(
  val route: String,
  @StringRes val label: Int,
  @DrawableRes val icon: Int
) {
  companion object {
    val screens = listOf(
      Temperature,
      Distances
    )
    const val route_temperature = "temperature"
    const val route_distances = "distances"
  }
  private object Temperature : ComposeUnitConverterScreen(
    route_temperature,
    R.string.temperature,
    R.drawable.baseline_thermostat_24
  )
  private object Distances : ComposeUnitConverterScreen(
    route_distances,
    R.string.distances,
    R.drawable.baseline_square_foot_24
  )
}
```

`ComposeUnitConverter` 由两个屏幕组成——`Temperature` 和 `Distances`。`route` 唯一标识一个屏幕。`label` 和 `icon` 将显示给用户。让我们看看这是如何实现的：

```kt
@Composable
fun ComposeUnitConverterBottomBar(navController:
   NavHostController) {
  BottomNavigation {
    val navBackStackEntry by
          navController.currentBackStackEntryAsState()
    val currentDestination = navBackStackEntry?.destination
    ComposeUnitConverterScreen.screens.forEach { screen ->
      BottomNavigationItem(
        selected = currentDestination?.hierarchy?.any {
          it.route == screen.route } == true,
        onClick = {
          navController.navigate(screen.route) {
            launchSingleTop = true
          }
        },
        label = {
          Text(text = stringResource(id = screen.label))
        },
        icon = {
          Icon(
            painter = painterResource(id = screen.icon),
            contentDescription = stringResource(id =
              screen.label)
          )
        },
        alwaysShowLabel = false
      )
    }
  }
}
```

`BottomNavigation()` 的内容由 `BottomNavigationItem()` 项目组成。每个项目代表一个 *目的地*。我们可以通过简单的循环来添加它们：

```kt
ComposeUnitConverterScreen.screens.forEach { screen ->
```

如您所见，`ComposeUnitConverterScreen` 实例的 `label` 和 `icon` 属性在调用 `BottomNavigationItem()` 时被使用。`alwaysShowLabel` 控制当项目被选中时标签是否可见。如果相应的屏幕当前正在显示，则项目将被选中。当点击 `BottomNavigationItem()` 时，会调用其 `onClick` 回调。我的实现调用提供的 `NavHostController` 实例上的 `navigate()`，传递来自相应的 `ComposeUnitConverterScreen` 对象的 `route`。

到目前为止，我们已经定义了屏幕并将它们映射到 `BottomNavigationItem()` 项目上。当点击一个项目时，应用程序会导航到指定的路由。但是，路由是如何与可组合函数相关的呢？我将在下一节中向你展示。

## 使用 NavHostController 和 NavHost()

一个 `NavHostController` 的实例允许我们通过调用其 `navigate()` 函数来导航到不同的屏幕。我们可以在 `ComposeUnitConverter()` 内部通过调用 `rememberNavController()` 获取其引用，然后将其传递给 `ComposeUnitConverterBottomBar()`。路由与可组合函数之间的映射是通过 `NavHost()` 建立的。它属于 `androidx.navigation.compose` 包。以下是这个可组合函数的调用方式：

```kt
@Composable
fun ComposeUnitConverterNavHost(
  navController: NavHostController,
  factory: ViewModelProvider.Factory?
) {
  NavHost(
    navController = navController,
    startDestination =
        ComposeUnitConverterScreen.route_temperature
  ) {
    composable(ComposeUnitConverterScreen.route_temperature) {
      TemperatureConverter(
        viewModel = viewModel(factory = factory)
      )
    }
    composable(ComposeUnitConverterScreen.route_distances) {
      DistancesConverter(
        viewModel = viewModel(factory = factory)
      )
    }
  }
}
```

`NavHost()` 接收三个参数：

+   我们 `NavHostController` 的引用

+   起始目的地的路由

+   构建导航图的构建器

在 Jetpack Compose 之前，导航图通常通过一个 XML 文件定义。`NavGraphBuilder` 提供了对简单领域特定语言的访问。`composable()` 添加一个可组合函数作为目的地。除了路由之外，你还可以传递一个参数列表和一个深度链接列表。

小贴士

Jetpack Navigation 的详细描述超出了本书的范围。你可以在[`developer.android.com/guide/navigation`](https://developer.android.com/guide/navigation)找到更多信息。

# 摘要

本章展示了 Jetpack Compose 的关键元素如何在真实世界的应用程序中协同工作。你学习了如何为主题 Compose 应用程序以及如何保持 Compose 主题与基于资源的主题同步。

我还向你展示了 `Scaffold()` 如何充当应用程序框架或骨架。我们使用其槽 API 插入了一个带有菜单的顶部应用程序栏，以及一个底部栏，用于使用 Jetpack Navigation 的 Compose 版本在屏幕之间导航。

在下一章，“技巧、窍门和最佳实践”中，我们将讨论如何分离 UI 和业务逻辑。我们将重新访问 `ComposeUnitConverter`，这次将重点介绍其使用 ViewModel 的方式。
