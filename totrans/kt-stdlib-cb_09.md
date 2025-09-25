# 杂项

在本章中，我们将涵盖以下食谱：

+   Kotlin 和 Java 互操作性问题和解决方案

+   Kotlin 和 JavaScript 互操作性问题和解决方案

+   生成类的重命名

+   将 Kotlin 代码反编译为 JVM 字节码

+   为导入添加自定义名称

+   使用类型别名包装复杂类型声明

+   表达式的`try…catch`声明

+   安全的类型转换

# 简介

本章将专注于展示各种问题的实用解决方案，这些问题是 Kotlin 开发者在日常工作中遇到的。在这里，你可以找到有关与 Java 和 JavaScript 互操作性的问题和解决方案的有用提示，以及一些帮助你更有效地编写代码的巧妙技巧。

# Kotlin 和 Java 互操作性

这个食谱将展示如何将 Java 和 Kotlin 类结合起来，并在同一应用程序组件中使用它们。我们将声明一个名为`ColoredText`的 Kotlin 数据类，它包含两种类型的`String`和`Color`属性。除了属性之外，它还将在伴随对象中暴露一个用于文本处理的实用函数。我们将学习如何使用这些属性，以及如何声明`ColoredText`类中的函数，使其在 Java 类中作为 JVM 静态方法可见。

# 如何做到...

1.  声明`ColoredText`类：

```kt
data class ColoredText
@JvmOverloads
constructor(
        var text: String = "",
        var color: Color = defaultColor) {

    companion object {
        @JvmField
        val defaultColor = Color.BLUE
    }
}
```

1.  在`companion`对象中实现一个静态 JVM 方法：

```kt
data class ColoredText
@JvmOverloads
constructor(
        var text: String = "",
        var color: Color = defaultColor) {

    companion object {
        @JvmField
        val defaultColor = Color.BLUE

 @JvmStatic
        fun processText(text: String): String =
 with(text) {
                    toLowerCase().trim().capitalize()
                }

    }

}
```

1.  添加一个成员函数，允许你将`text`属性打印到控制台：

```kt
data class ColoredText
@JvmOverloads
constructor(
        var text: String = "",
        var color: Color = defaultColor) {

 fun printToConsole() = println(text)

    companion object {
        @JvmField
        val defaultColor = Color.BLUE

        @JvmStatic
        fun processText(text: String): String =
                with(text) {
                    toLowerCase().trim().capitalize()
                }
    }
}
```

1.  实现一个使用 Kotlin 类函数和属性的 Java 类：

```kt
public class JavaApp {
    public static void main(String... args) {
        String rawText = 
              " one Of The Best Programming Skills You Can Have " +
              "Is Knowing When To Walk Away For Awhile. ";
        String text = ColoredText.processText(rawText);
        ColoredText myText =
              new ColoredText(text, ColoredText.defaultColor);
        myText.printToConsole();
    }
}
```

# 它是如何工作的...

结果，`JavaApp` Java 类中的主函数将打印以下智慧到控制台：

```kt
One of the best programming skills you can have is knowing when to walk away for awhile.
```

由于 Kotlin 和 Java 类都编译为包含在公共代码库中的相同 JVM 字节码，因此 Kotlin 和 Java 互操作性绝对无缝。然而，有一些特殊情况需要我们在将 Kotlin 声明以特定方式在 Java 侧可用时给予额外关注。

首先，在`ColoredText`类中，我们使用`@JvmOverloads`注解标记构造函数，这告诉编译器在声明任何默认属性值的情况下生成多个构造函数实例。多亏了这一点，我们可以在某些 Java 类中实例化`ColoredText`类，而无需传递`text`和/或`color`属性值。

接下来我们使用的注解是`@JvmField`，它告诉 Kotlin 编译器不要为这个属性生成 getter 和 setter 函数，并将其作为字段暴露。当通过 Kotlin 对象暴露一个常量值时，它提供了一个更简洁的语法来访问 Java 侧的值。

另一个常用的注解是 `@JvmStatic`。它的目的是告诉编译器需要为这个函数生成一个额外的静态方法，以便在 Java 中作为外部类的直接静态方法使用。例如，在我们的案例中，我们可以通过以下方式在 Java 中访问 `processText()` 函数，省略 `Companion` 元素：

```kt
ColoredText.processText("sample text")
```

# Kotlin 和 JavaScript 互操作性

在下面的菜谱中，我们将配置并实现一个示例 Web 应用程序项目，以探索 Kotlin 如何编译成 JavaScript。我们将实现一个简单的 Web 应用程序，当应用程序启动时将打开一个警告对话框。以下示例将展示如何将 Kotlin 和 JavaScript 代码结合在一起，并使用 Gradle 构建脚本配置 JavaScript 编译。

# 准备工作

为了设置项目以将 Kotlin 文件编译成 JavaScript，我们需要在模块级别的 Gradle 构建脚本中添加以下属性。首先，我们需要应用 Kotlin2Js 插件。我们可以通过以下声明来完成：

```kt
apply plugin: "kotlin2js"
```

到目前为止，每次我们执行 Gradle 的 `build` 任务时，Kotlin2JS 编译器都会生成来自 Kotlin 文件的相应函数和类，并将它们写入 `build/classes/kotlin/` 目录下的 JS 文件，该文件以项目名称命名。然而，我们可以通过指定输出文件参数来修改此默认行为：

```kt
compileKotlin2Js.kotlinOptions.outputFile = "${projectDir}/web/js/app.js"

```

因此，Kotlin 文件编译的输出将位于 `web/js 目录` 下的 `app.js` 文件中。

然而，为了执行转换后的 Kotlin 代码，我们还需要将其与 Kotlin JS 标准库链接起来。我们可以修改 Gradle 构建脚本，将所需的库包含在 `web/js` 输出目录中：

```kt
build.doLast {
    configurations.compile.each { File file ->
        copy {
            includeEmptyDirs = false

            from zipTree(file.absolutePath)
            into "${projectDir}/web/js/lib"
            include { fileTreeElement ->
                def path = fileTreeElement.path
                path.endsWith(".js") && (path.startsWith("META-INF/resources/") || !path.startsWith("META-INF/"))
            }
        }
    }
}
```

您可以在示例项目中检查 Kotlin2JS 插件的配置：[`github.com/PacktPublishing/Kotlin-Standard-Library-Cookbook/tree/master/Kotlin-Samples-JS`](https://github.com/PacktPublishing/Kotlin-Standard-Library-Cookbook/tree/master/Kotlin-Samples-JS).

# 如何做到这一点...

1.  创建一个新的 Kotlin 文件，`AlertDialogApp.kt`，其中包含 `main()` 函数：

```kt
fun main(args : Array<String>) {}
```

1.  声明对 JS `alert()` 函数的引用：

```kt
fun main(args : Array<String>) {}

external fun alert(message: Any?): Unit
```

1.  实现一个 `showAlert()` 函数并在 `main()` 函数中调用它：

```kt
fun main(args : Array<String>) {
    showAlert()
}

fun showAlert() {
 val number: dynamic = js("Math.floor(Math.random() * 1000)")
 val message = "There were $number viruses found on your computer! \uD83D\uDE31"
    println("showing alert")
 alert(message)
}

external fun alert(message: Any?): Unit
```

# 它是如何工作的...

如您所见，在运行 Gradle 构建任务后，`AlertDialogApp.kt` 文件将被转换为 `app.js` JavaScript 代码，位于 `web/js` 目录下，以及链接在 `web/js/lib` 目录下的 `kotlin.js` 文件。

我们可以通过在网页浏览器中运行它来测试生成的 JS 代码。为了做到这一点，我们将在项目的主目录下创建一个名为 `test_app.html` 的示例 HTML 文件，该文件将链接 `kotlin.js` 标准库文件并运行包含从 `AlertDialogApp.kt` 文件生成的 `main()` 函数实现的 `app.js` 文件：

```kt
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Test</title>
</head>
<body>
    <script src="img/kotlin.js"></script>
    <script src="img/app.js"></script>
</body>
</html>
```

结果，当我们在一个网络浏览器中打开 `test_app.html` 文件时，我们将遇到以下弹出对话框：

![图片](img/5840543c-882b-48dd-ab51-02c17c8ac10d.png)

在 `AlertDialogApp.kt` 文件中的 `showAlert()` 函数内部，我们使用 JavaScript 的 `Math.floor()` 和 `Math.random()` 函数生成一个 0-1,000 之间的随机整数。我们使用 Kotlin 标准库中可用的 `js()` 函数在 Kotlin 代码中内联 JS 代码。正如你所看到的，`js()` 返回的结果被声明为 `dynamic` 类型。

`dynamic` 修饰符用于声明动态类型，这是每个 JavaScript 对象的特征。在 Kotlin 代码中，它可以作为强类型声明的替代品使用。当我们处理可能返回未指定类型结果的第三方 JS 库时，使用它是有意义的。

接下来，我们使用通过内联的 JS 代码生成的随机整数来组合将要显示的消息。最后，我们调用 JS 的 `alert()` 函数，并将组合的消息传递给它。这次，我们使用了 `external` 修饰符，它告诉 Kotlin 编译器相应的声明是用纯 JavaScript 编写的，它不应该为它生成实现。

# 还有更多...

你可能会想知道是否有实际的方法可以在基于 Gradle 的项目中处理 `npm` JS 依赖项。有一个解决方案允许你轻松地将你的 Kotlin 项目与 `npm` 依赖项集成，称为 Kotlin Frontend Gradle 插件。你可以在官方项目指南中了解更多信息：[`github.com/Kotlin/kotlin-frontend-plugin`](https://github.com/Kotlin/kotlin-frontend-plugin)。

# 重命名生成的函数

在这个菜谱中，我们将学习如何在 Kotlin 函数被编译成生成的 JVM 字节码时修改其名称。我们需要这个功能是因为在生成 JVM 字节码时会发生类型擦除。然而，多亏了 `@JvmName` 注解，我们可以声明多个不同的函数，但它们具有相同的名称，并在 Kotlin 代码中使用它们的原始名称，同时保持它们的 JVM 字节码名称不同，以满足编译器的要求。

# 如何做到这一点...

1.  声明两个具有相同名称的函数：

```kt
fun List<String>.join(): String {
    return joinToString()
}

fun List<Int>.join(): String =
    map { it.toString() }
            .joinToString()    
```

1.  使用适当的注解标记函数：

```kt
@JvmName("joinStringList")
fun List<String>.join(): String {
    return joinToString()
}

@JvmName("joinIntList")
fun List<Int>.join(): String =
    map { it.toString() }
            .joinToString()
```

# 它是如何工作的...

由于提供了替代函数名，我们能够将它们编译成 JVM 字节码。然而，你可以轻松地测试我们可以在 Kotlin 代码中使用它们的原始名称。这是因为 Kotlin 编译器能够根据它们的返回类型和泛型类型参数值正确地识别它们。

你可以通过在整数列表和字符串列表上运行 `join()` 函数来测试这一点：

```kt
fun main(vararg args: String) {
    println(listOf(1, 2, 3).join())
    println(listOf("a", "b", "c").join())
}
```

结果，前面的代码将打印以下文本到控制台：

```kt
1, 2, 3
a, b, c
```

请记住，当你想要从 Java 调用这些函数时，你需要使用它们的替代名称：`joinStringList()` 和 `joinIntList()`。

# 还有更多...

此外，还有一个相应的 `@JsName` 注解，它允许你更改 JavaScript 函数和类的名称。如果你使用 Kotlin2JS 插件将 Kotlin 文件编译到 JavaScript，你可以使用它。如果你想熟悉 Kotlin2JS 插件的基础知识，你可以检查 *Kotlin 和 JavaScript 互操作性* 食谱。

# 参见

+   如果你想要了解如何探索从 Kotlin 文件生成的最终 JVM 字节码，请阅读 *反编译 Kotlin 代码到 Java 和 JVM 字节码* 食谱

# 反编译 Kotlin 代码到 Java 和 JVM 字节码

在本食谱中，我们将学习如何轻松地将 Kotlin 文件反编译，以查看其对应的 JVM 字节码是如何实现的，以及字节码对应的 Java 实现看起来会是什么样子。这可以帮助你发现 Kotlin 各种概念在底层是如何实现的。这也有助于代码调试和优化。

# 准备工作

让我们创建一个新的 Kotlin 文件，命名为 `Recipe4.kt`，其中包含以下示例实现，以便查看其字节码转换：

```kt
data class A(val a: String = "a") {
    companion object {
        @JvmStatic
        fun foo(): String = "Wooo!"
    }    
}
```

# 如何操作...

1.  在 IntelliJ 中打开 `Recipe4.kt` 文件。

1.  从工具/ Kotlin 菜单中选择“显示 Kotlin 字节码”选项。框将显示 JVM 字节码实现。

1.  点击 Kotlin 字节码视图中的“反编译”按钮。相应的 Java 实现将从字节码中反编译出来，并出现在新窗口中。

# 它是如何工作的...

分析为 `data class A` 生成的 Java 实现的任务留作读者练习。你可以通过从 Kotlin 类定义中移除 `data` 修饰符来实验，并观察字节码和反编译的 Java 实现中的变化。

# 为导入添加自定义名称

在本食谱中，我们将探讨如何为 `import` 声明添加自定义名称。我们将导入 `java.lang.StringBuilder` 类，为其添加一个自定义名称，并在示例代码中使用它来演示其用法。

# 如何操作...

1.  使用自定义别名导入 `StringBuilder` 类：

```kt
import java.lang.StringBuilder as builder
```

1.  在示例代码中使用自定义的 `StringBuilder` 名称：

```kt
import java.lang.StringBuilder as builder

fun main(vararg args: String) {
 val text = builder()
 .append("Code is like humor. ")
 .append("When you have to explain it, ")
 .append("it’s bad.")
 .toString()
 println(text)
}
```

# 它是如何工作的...

如你所见，我们能够使用替代名称而不是 `StringBuilder` 类。这是一个小功能，但有时可以用来使你的代码更容易阅读。我们的示例代码将打印以下文本到控制台：

```kt
Code is like humor. When you have to explain it, it’s bad.
```

# 使用类型别名包装复杂类型声明

有时我们需要处理长或冗长的类型声明。幸运的是，在 Kotlin 中，我们能够为任何现有类型分配一个替代名称，并使用较短的替代名称。这也可以帮助你编写更易于理解的优雅代码。本食谱将演示如何使用类型别名。

# 准备工作

假设我们有两个预定义的类：

```kt
data class Song(val title: String)
data class Artist(val name: String)
```

我们将定义一个`Song`类型值的映射类型别名和一个泛型键类型–`Map<T, List<Song>>`。接下来，我们将使用它来定义一个函数，该函数将返回给定`Map<Artist, List<Song>>`对象的最受欢迎的`Artist`实例。

# 如何实现...

1.  声明一个`Map<T, List<Song>>`类型的泛型类型别名：

```kt
typealias GrouppedSongs<T> = Map<T, List<Song>>
```

1.  使用类型别名实现`getMostPopularArtist()`函数：

```kt
fun getMostPopularArtist(songs: GrouppedSongs<Artist>) =
    songs.toList().sortedByDescending {it.second.size }.first().first
```

# 它是如何工作的...

使用类型别名，我们能够为类型提供一个自定义名称，并且可以在`getMostPopularArtist(songs: GrouppedSongs<Artist>)`中使用它，而不是使用`Map<Artist, List<Song>>`类型，这导致了一个更有意义的声明。我们可以通过用示例数据调用`getMostPopularArtist()`来测试我们的实现：

```kt
val songs: GrouppedSongs<Artist> =
        mapOf(
                Artist("Bob Dylan") to
                        listOf(Song("Blowing In The Wind"),
                               Song("To Fall in Love With You")),

                Artist("Louis Armstrong") to
                        listOf(Song("What A Beautiful World"))
        )

println("${getMostPopularArtist(songs)} is the most popular")
```

因此，我们将得到以下文本打印到控制台：

```kt
Artist(name=Bob Dylan) is most popular
```

# 表达式的 try…catch 声明

Kotlin 被宣传为一种极其表达性的语言。然而，这是语言的一个特点，在开始时可能并不明显，尤其是如果你习惯了像 Java 或 JavaScript 这样的其他语言。为了更清楚地展示语言风格，在这个菜谱中，我们将通过将其视为表达式来发现如何在 Kotlin 中以`try…catch`声明的方式工作。

# 准备工作

让我们考虑以下 Java 代码：

```kt
int value;
try {
    result = parseInt(input);
} catch (NumberFormatException e) {
} finally {
    result = 0;
}
```

它声明了一个`int`类型的`result`变量。接下来，它尝试使用`Integer.parseInt()`函数将字符串值解析为整数，如果成功，则将结果赋值给`value`变量。如果`parseInt()`无法解析字符串，则将默认值`0`赋值给`value`变量。

# 如何实现...

1.  在`try…catch`声明中调用`parseInt()`函数：

```kt
try {
    parseInt("fdsaa")
} catch (e: NumberFormatException) {
    0
}
```

1.  将`try…catch`声明的结果赋值给`value`变量：

```kt
val result = try {
    parseInt("fdsaa")
} catch (e: NumberFormatException) {
    0
}
```

# 它是如何工作的...

就这样。Kotlin 中的`try…catch`声明可以被赋值给变量。原因是它实际上是一个表达式！在我们的例子中，`try…catch`返回`parseInt()`函数的结果，当函数抛出异常时，它返回`0`。

# 更多...

类似地，我们可以将其他语言声明视为表达式。将控制流语句（如`if`和`when`）返回的值赋给变量是一种常见的做法。例如，我们可以以下这种方式使用`when`作为表达式：

```kt
val result = when(input) {
    is Int -> input
    is String -> parseInt(input)
    else -> 0
}
```

# 安全的类型转换

无论何时进行类型转换，我们都应该牢记这是一个潜在的异常来源。这就是为什么我们应该始终使用`is`修饰符进行类型检查，或者在`try…catch`块中进行转换的原因。然而，在 Kotlin 中，我们还有一个安全的转换选项，它不会抛出`ClassCastException`，而是返回`null`。在这个菜谱中，我们将测试安全转换的实际应用。

# 如何实现...

1.  让我们先定义一个返回随机`Double`值的`Number`类型的函数：

```kt
fun getRandomNumber(): Number = Random().nextDouble() * 10
```

1.  尝试使用安全转换操作符将函数的结果转换为不同的类型，并将转换后的值打印到控制台：

```kt
println(getRandomNumber() as? Int)
println(getRandomNumber() as? Double)
println(getRandomNumber() as? String)
```

# 它是如何工作的...

上述代码不会失败也不会抛出任何异常。它只会返回 `null` 值。我们的转换测试代码将打印以下输出到控制台：

```kt
null
8.802117014997226
null
```

使用 `as?` 转换修饰符是传统方式的整洁替代品。如果你正在处理可空类型，即在与不提供空安全的外部库一起工作时，可以使用它。然而，如果你能从空安全中受益，最好使用标准的转换操作。
