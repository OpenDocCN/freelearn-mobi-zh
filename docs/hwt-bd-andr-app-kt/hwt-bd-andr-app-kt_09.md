# 第九章：使用 JUnit、Mockito 和 Espresso 进行单元测试和集成测试

概述

在本章中，你将学习关于在 Android 平台上进行测试以及如何创建单元测试、集成测试和 UI 测试。你将看到如何创建这些类型的测试，分析它们的运行方式，并使用 JUnit、Mockito、Robolectric 和 Espresso 等框架进行工作。你还将学习关于测试驱动开发，这是一种将测试置于实现之上的软件开发实践。通过本章的学习，你将能够将你的新测试技能结合起来应用到一个真实项目中。

# 介绍

在之前的章节中，你学习了如何加载背景数据并在用户界面中显示它，以及如何设置 API 调用来检索数据。但是你如何确保一切正常？如果你处于一个你过去没有太多互动的项目中需要修复一个错误的情况下怎么办？你如何知道你所应用的修复不会触发另一个错误？这些问题的答案是通过测试。

在本章中，我们将分析开发人员可以编写的测试类型，并查看可用的测试工具以简化测试体验。首先出现的问题是台式机或笔记本电脑使用不同的操作系统来开发移动应用。这意味着测试也必须在设备或模拟器上运行，这将减慢测试的速度。为了解决这个问题，我们有两种类型的测试：`test`文件夹将在你的机器上运行，而`androidTest`文件夹将在设备或模拟器上运行。

这两种测试都依赖于 Java 的**JUnit**库，它帮助开发人员设置他们的测试并将它们分组到不同的类别中。它还提供了不同的配置选项，以及其他库可以构建的扩展。我们还将研究测试金字塔，它帮助指导开发人员如何组织他们的测试。我们将从金字塔的底部开始，代表着**单元测试**，向上移动通过**集成测试**，最终达到顶部，代表着**端到端测试**（UI 测试）。你将有机会学习到帮助编写每种类型测试的工具：

+   **Mockito**和**mockito-kotlin**主要帮助进行单元测试，并且对于创建模拟或测试替身非常有用，我们可以操纵输入以便断言不同的场景。（模拟或测试替身是一个模仿另一个对象实现的对象。每当一个测试与模拟对象交互时，你可以指定这些交互的行为。）

+   **Robolectric**是一个开源库，它将 Android 框架引入你的机器，允许你在本地测试活动和片段，而不是在模拟器上。这可以用于单元测试和集成测试。

+   `EditText`组件等）和断言（验证视图显示特定文本，当前显示给用户，启用等）在应用的 UI 中的仪器测试。

在本章中，我们还将介绍**测试驱动开发**（**TDD**）。这是一个测试优先的软件开发过程。简单来说，就是先编写测试。我们将分析在为 Android 应用程序开发功能时采用这种方法。要记住的一件事是，为了正确测试应用程序，其类必须正确编写。一种方法是清晰地定义类之间的边界，并根据您希望它们完成的任务对它们进行拆分。一旦您做到了这一点，您还可以在编写类时依赖于**依赖反转**和**依赖注入**原则。当这些原则得到正确应用时，您应该能够将虚假对象注入到测试对象中，并操纵输入以适应您的测试场景。依赖注入还有助于编写插装测试，以帮助您用本地数据替换进行网络调用的模块，以使您的测试独立于网络等外部因素。插装测试是在设备或模拟器上运行的测试。 "插装"关键字来自插装框架，该框架组装这些测试，然后在设备上执行它们。

理想情况下，每个应用程序应该有三种类型的测试：

+   **单元测试**：这些是验证单个类和方法的本地测试。它们应该占大多数测试，并且它们应该快速、易于调试和易于维护。它们也被称为小型测试。

+   **集成测试**：这些是使用 Robolectric 的本地测试，或验证应用程序模块和组件之间交互的插装测试。这些比单元测试更慢，更复杂。复杂性的增加是由于组件之间的交互。这些也被称为中型测试。

+   **UI 测试（端到端测试）**：这些是验证完整用户旅程和场景的插装测试。这使它们更复杂，更难以维护；它们应该代表您总测试数量中的最少部分。这些也被称为大型测试。

在下图中，您可以观察到**测试金字塔**。Google 的建议是保持 70:20:10（单元测试：集成测试：UI 测试）的比例：

![图 9.1：测试金字塔](img/B15216_09_01.jpg)

图 9.1：测试金字塔

如前一节所述，单元测试是验证代码的一小部分的测试，大多数测试应该是覆盖各种场景（成功、错误、限制等）的单元测试。理想情况下，这些测试应该是本地的，但也有一些例外情况，您可以将它们制作成插装测试。这些情况很少，应该限制在您想要与设备的特定硬件交互时。

# JUnit

JUnit 是一个用于在 Java 和 Android 中编写单元测试的框架。它负责测试的执行方式，允许开发人员配置他们的测试。它提供了许多功能，例如以下内容：

+   `@Before`和`@After`注解。

+   **断言**：用于验证操作的结果与预期值是否一致。

+   **规则**：这些允许开发人员设置多个测试的常见输入。

+   **运行器**：使用这些，您可以指定如何执行测试。

+   **参数**：这些允许使用多个输入执行测试方法。

+   **排序**：这些指定测试应该以何种顺序执行。

+   **匹配器**：这些允许您定义模式，然后用于验证测试对象的结果，或者帮助您控制模拟的行为。

在 Android Studio 中，创建新项目时，`app`模块会在 Gradle 中带有 JUnit 库。这应该在`app/build.gradle`中可见：

```kt
testImplementation 'junit:junit:4.13.1'
```

让我们看看我们需要测试的以下类：

```kt
class MyClass {
    fun factorial(n: Int): Int {
        return IntArray(n) {
            it+1
        }.reduce { acc, i ->
            acc * i
        }
    }
}
```

这个方法应该返回数字 `n` 的阶乘。我们可以从一个检查值的简单测试开始。为了创建一个新的单元测试，您需要在项目的 `test` 目录中创建一个新的类。大多数开发人员遵循的典型约定是在 `test` 目录中的相同包下为您的类名称添加 `Test` 后缀，并将其放在相同的包下。例如，`com.mypackage.ClassA` 将在 `com.mypackage.ClassATest` 中进行测试：

```kt
import org.junit.Assert.assertEquals
import org.junit.Test
class MyClassTest {
    private val myClass = MyClass()
    @Test
    fun computesFactorial() {
        val n = 3
        val result = myClass.factorial(n)
        assertEquals(6, result)
    }
}
```

在这个测试中，您可以看到我们初始化了被测试的类，测试方法本身使用了 `@Test` 注解。测试方法本身将断言 `(3!)==6`。断言使用了 JUnit 库中的 `assertEquals` 方法。开发中的一个常见做法是将测试分为三个区域，也称为 AAA（Arrange-Act-Assert）：

+   Arrange - 初始化输入的地方

+   Act - 调用被测试方法的地方

+   Assert - 验证的地方

我们可以编写另一个测试来确保值是正确的，但这意味着我们最终会重复代码。现在我们可以尝试编写一个参数化测试。为了做到这一点，我们需要使用参数化测试运行器。前面的测试有其自己的内置运行器，由 JUnit 提供。参数化运行器将为我们提供的不同值重复运行测试，并且看起来像下面这样。（请注意，出于简洁起见，已删除了导入语句。）

```kt
@RunWith(Parameterized::class)
class MyClassTest(
    private val input: Int,
    private val expected: Int
) {
    companion object {
        @Parameterized.Parameters
        @JvmStatic
        fun getData(): Collection<Array<Int>> = listOf(
            arrayOf(0, 1),
            arrayOf(1, 1),
            arrayOf(2, 2),
            arrayOf(3, 6),
            arrayOf(4, 24),
            arrayOf(5, 120)
        )
    }
    private val myClass = MyClass()
    @Test
    fun computesFactorial() {
        val result = myClass.factorial(input)
        assertEquals(expected, result)
    }
}
```

这实际上将运行六个测试。`@Parameterized` 注解的使用告诉 JUnit 这是一个具有多个参数的测试，并且还允许我们为测试添加一个构造函数，该构造函数将表示我们的阶乘函数的输入值和输出。然后我们使用 `@Parameterized.Parameters` 注解定义了一组参数。这个测试的每个参数都是一个单独的列表，包含输入和期望的输出。当 JUnit 运行这个测试时，它将为每个参数运行一个新的实例，然后执行测试方法。这将产生五个成功和一个失败的结果，当我们测试 *0!* 时会失败，这意味着我们发现了一个错误。我们从未考虑 *n = 0* 的情况。现在，我们可以回到我们的代码来修复失败。我们可以通过用 `fold` 函数替换 `reduce` 函数来做到这一点，`fold` 函数允许我们指定初始值为 `1`：

```kt
fun factorial(n: Int): Int {
        return IntArray(n) {
            it + 1
        }.fold(1, { acc, i -> acc * i })
    }
```

现在运行测试，它们都会通过。但这并不意味着我们在这里就完成了。有很多事情可能会出错。如果 `n` 是一个负数会发生什么？由于我们在处理阶乘，可能会得到非常大的数字。在我们的示例中，我们使用整数，这意味着整数在 *12!* 之后会溢出。通常情况下，我们会在 `MyClassTest` 类中创建新的测试方法，但由于使用了参数化运行器，我们所有的新方法都会运行多次，这将花费我们的时间，因此我们将创建一个新的测试类来检查我们的错误：

```kt
class MyClassTest2 {
    private val myClass = MyClass()
    @Test(expected = MyClass.FactorialNotFoundException::class)
    fun computeNegatives() {
        myClass.factorial(-10)
    }
}
```

这将导致被测试的类发生以下变化。

```kt
class MyClass {
    @Throws(FactorialNotFoundException::class)
    fun factorial(n: Int): Int {
        if (n < 0) {
            throw FactorialNotFoundException
        }
        return IntArray(n) {
            it + 1
        }.fold(1, { acc, i -> acc * i })
    }
    object FactorialNotFoundException : Throwable()
}
```

让我们解决非常大的阶乘的问题。我们可以使用 `BigInteger` 类，它能够容纳大数字。我们可以更新测试如下（未显示导入语句）：

```kt
@RunWith(Parameterized::class)
class MyClassTest(
    private val input: Int,
    private val expected: BigInteger
) {
    companion object {
        @Parameterized.Parameters
        @JvmStatic
        fun getData(): Collection<Array<Any>> = listOf(
            arrayOf(0, BigInteger.ONE),
            arrayOf(1, BigInteger.ONE),
            arrayOf(2, BigInteger.valueOf(2)),
            arrayOf(3, BigInteger.valueOf(6)),
            arrayOf(4, BigInteger.valueOf(24)),
            arrayOf(5, BigInteger.valueOf(120)),
            arrayOf(13, BigInteger("6227020800")),
            arrayOf(25, BigInteger("15511210043330985984000000"))
        )
    }
    private val myClass = MyClass()
    @Test
    fun computesFactorial() {
        val result = myClass.factorial(input)
        assertEquals(expected, result)
    }
}
```

现在被测试的类看起来像这样：

```kt
    @Throws(FactorialNotFoundException::class)
    fun factorial(n: Int): BigInteger {
        if (n < 0) {
            throw FactorialNotFoundException
        }
        return IntArray(n) {
            it + 1
        }.fold(BigInteger.ONE, { acc, i -> acc * i.toBigInteger() })
    }
```

在前面的示例中，我们使用 `IntArray` 实现了阶乘。这个实现更多地基于 Kotlin 能够将方法链接在一起的能力，但它有一个缺点：当不需要时它使用了数组的内存。我们只关心阶乘，而不是存储从 1 到 *n* 的所有数字。我们可以将实现更改为一个简单的 `for` 循环，并在重构过程中使用测试来指导我们。在您的应用程序中有测试的两个好处可以在这里观察到：

+   它们作为更新后的文档，说明了特性应该如何实现。

+   Android Studio 测试技巧

让我们更新代码，摆脱`IntArray`：

```kt
    @Throws(FactorialNotFoundException::class)
    fun factorial(n: Int): BigInteger {
        if (n < 0) {
            throw FactorialNotFoundException
        }
        var result = BigInteger.ONE
        for (i in 1..n){
            result = result.times(i.toBigInteger())
        }
        return result
    }
```

如果我们修改`factorial`函数，如前面的示例所示，并运行测试，我们应该看到它们都通过了。

在某些情况下，您的测试将使用测试或应用程序常见的资源（数据库、文件等）。理想情况下，这不应该发生在单元测试中，但总会有例外。让我们分析一下这种情况，看看 JUnit 如何帮助我们。我们将添加一个`companion`对象，它将存储结果，以模拟这种行为：

```kt
    companion object {
        var result: BigInteger = BigInteger.ONE
    }
    @Throws(FactorialNotFoundException::class)
    fun factorial(n: Int): BigInteger {
        if (n < 0) {
            throw FactorialNotFoundException
        }
        for (i in 1..n) {
            result = result.times(i.toBigInteger())
        }
        return result
    }
```

它们在重构代码时通过保持相同的断言并检测新的代码更改是否破坏了它来指导我们。

```kt
    @Before
    fun setUp(){
        MyClass.result = BigInteger.ONE
    }
    @After
    fun tearDown(){
        MyClass.result = BigInteger.ONE
    }
    @Test
    fun computesFactorial() {
        val result = myClass.factorial(input)
        assertEquals(expected, result)
    }
```

在测试中，我们添加了两个带有`@Before`和`@After`注解的方法。当引入这些方法时，JUnit 将按以下方式更改执行流程：所有带有`@Before`注解的方法将被执行，将执行带有`@Test`注解的方法，然后将执行所有带有`@After`注解的方法。这个过程将对类中的每个`@Test`方法重复执行。

如果您发现自己在`@Before`方法中重复相同的语句，可以考虑使用`@Rule`来消除重复。我们可以为前面的示例设置一个测试规则。测试规则应该在`test`或`androidTest`包中，因为它们的使用仅限于测试。它们往往用于多个测试中，因此可以将规则放在`rules`包中（未显示导入语句）：

```kt
class ResultRule : TestRule {
    override fun apply(
        base: Statement,
        description: Description?
    ): Statement? {
        return object : Statement() {
            @Throws(Throwable::class)
            override fun evaluate() {
                MyClass.result = BigInteger.ONE
                try {
                    base.evaluate()
                } finally {
                    MyClass.result = BigInteger.ONE
                }
            }
        }
    }
}
```

如果我们执行前面代码的测试，将开始看到一些测试失败。这是因为在第一个测试执行`factorial`函数后，结果将具有执行测试的值，当执行新测试时，阶乘的结果将乘以结果的先前值。通常，这是好的，因为测试告诉我们我们做错了什么，我们应该纠正这个问题，但是对于这个示例，我们将直接在测试中解决这个问题：

```kt
    @JvmField
    @Rule
    val resultRule = ResultRule()
    private val myClass = MyClass()
    @Test
    fun computesFactorial() {
        val result = myClass.factorial(input)
        assertEquals(expected, result)
    }
```

为了将规则添加到测试中，我们使用`@Rule`注解。由于测试是用 Kotlin 编写的，我们使用`@JvmField`来避免生成 getter 和 setter，因为`@Rule`需要一个公共字段而不是方法。

# ![图 9.2：Android Studio 中的测试输出

在前面的示例中，我们可以看到规则将实现`TestRule`，而`TestRule`又带有`apply()`方法。然后我们创建一个新的`Statement`对象，它将执行基本语句（测试本身）并在语句之前和之后重置结果的值。现在我们可以修改测试如下：Android Studio 提供了一套很好的快捷方式和可视化工具来帮助测试。如果要为类创建新的测试或转到类的现有测试，可以使用*Ctrl* + *Shift* + *T*（Windows）或*Command* + *Shift* + *T*（Mac）快捷键。要运行测试，有多种选项：右键单击文件或包，然后选择`Run Tests in...`选项，或者如果要独立运行测试，可以转到特定的测试方法并选择顶部的绿色图标，这将执行类中的所有测试；或者，对于单个测试，可以单击`@Test`注解方法旁边的绿色图标。这将触发测试执行，显示在`Run`选项卡中，如下截图所示。测试完成后，它们将变成红色或绿色，取决于它们的成功状态：Android Studio 提供了一套很好的快捷方式和可视化工具来帮助测试。如果要为类创建新的测试或转到类的现有测试，可以使用*Ctrl* + *Shift* + *T*（Windows）或*Command* + *Shift* + *T*（Mac）快捷键。要运行测试，有多种选项：右键单击文件或包，然后选择`Run Tests in...`选项，或者如果要独立运行测试，可以转到特定的测试方法并选择顶部的绿色图标，这将执行类中的所有测试；或者，对于单个测试，可以单击`@Test`注解方法旁边的绿色图标。这将触发测试执行，显示在`Run`选项卡中，如下截图所示。测试完成后，它们将变成红色或绿色，取决于它们的成功状态：图 9.2：Android Studio 中的测试输出在测试中可以找到的另一个重要功能是调试功能。这很重要，因为您可以调试测试和被测试的方法，所以如果在修复问题时遇到问题，您可以使用此功能查看测试使用的输入以及代码如何处理输入。您可以在测试旁边的绿色图标中找到的第三个功能是`Run With Coverage`选项。这有助于开发人员确定测试覆盖的代码行以及跳过的代码行。覆盖率越高，发现崩溃和错误的机会就越大：![图 9.3：Android Studio 中的测试覆盖率](img/B15216_09_03.jpg)

图 9.3：Android Studio 中的测试覆盖率

在上图中，您可以看到我们的类的覆盖范围，其中包括被测试的类的数量、被测试的方法的数量和被测试的行数。

另一种运行 Android 应用程序测试的方法是通过命令行。这通常在您的项目中有`Terminal`选项卡的情况下非常方便。`Terminal`选项卡通常位于 Android Studio 底部栏附近的`Logcat`选项卡附近。在每个 Android Studio 项目中，都有一个名为`gradlew`的文件。这是一个可执行文件，允许开发人员执行 Gradle 命令。为了运行本地单元测试，您可以使用以下命令：

+   `gradlew.bat test`（适用于 Windows）

+   `./gradlew test`（适用于 Mac 和 Linux）

执行该命令后，应用程序将被构建和测试。您可以在 Android Studio 右侧的`Gradle`选项卡中找到可以在`Terminal`中输入的各种命令。从`Terminal`或`Gradle`选项卡执行时，测试的输出可以在`app/build/reports`文件夹中找到：

![图 9.4：Android Studio 中的 Gradle 命令](img/B15216_09_04.jpg)

图 9.4：Android Studio 中的 Gradle 命令

# Mockito

在前面的示例中，我们看了如何设置单元测试以及如何使用断言来验证操作的结果。如果我们想要验证某个方法是否被调用呢？或者如果我们想要操纵测试输入以测试特定情景呢？在这些情况下，我们可以使用**Mockito**。这是一个帮助开发人员设置虚拟对象的库，可以将其注入到被测试的对象中，并允许它们验证方法调用、设置输入，甚至监视测试对象本身。

该库应该添加到您的`test` Gradle 设置中，如下所示：

```kt
testImplementation 'org.mockito:mockito-core:3.6.0'
```

现在，让我们看一下以下代码示例（请注意，为简洁起见，以下代码片段中的导入语句已被删除）：

```kt
class StringConcatenator(private val context: Context) {
    fun concatenate(@StringRes stringRes1: Int, 
      @StringRes stringRes2: Int): String {
      return context.getString(stringRes1).plus(context
          .getString(stringRes2))
    }
}
```

在这里，我们有`Context`对象，通常无法进行单元测试，因为它是 Android 框架的一部分。我们可以使用`mockito`创建一个测试替身，并将其注入到`StringConcatenator`对象中。然后，我们可以操纵对`getString()`的调用，以返回我们选择的任何输入。这个过程被称为模拟。

```kt
class StringConcatenatorTest {
    private val context = Mockito.mock(Context::class.java)
    private val stringConcatenator = StringConcatenator(context)
    @Test
    fun concatenate() {
        val stringRes1 = 1
        val stringRes2 = 2
        val string1 = "string1"
        val string2 = "string2"
        Mockito.`when`(context.getString(stringRes1)).thenReturn(string1)
        Mockito.`when`(context.getString(stringRes2)).thenReturn(string2)
        val result = stringConcatenator.concatenate(stringRes1,
            stringRes2)
        assertEquals(string1.plus(string2), result)
    }
}
```

注意

`` ` ``是 Kotlin 中的转义字符，不应与引号混淆。它允许开发人员为方法设置任何他们想要的名称，包括特殊字符或保留字。

在测试中，我们已经创建了一个`mock`上下文。当测试`concatenate`方法时，我们使用 Mockito 在调用特定输入的`getString()`方法时返回一个特定的字符串。这样我们就可以断言最终的结果。

Mockito 不仅限于仅模拟 Android Framework 类。我们可以创建一个`SpecificStringConcatenator`类，它将使用`StringConcatenator`从`strings.xml`中连接两个特定的字符串：

```kt
class SpecificStringConcatenator(private val stringConcatenator:   StringConcatenator) {
    fun concatenateSpecificStrings(): String {
        return stringConcatenator.concatenate(R.string.string_1,           R.string.string_2)
    }
}
```

我们可以这样为它编写测试：

```kt
class SpecificStringConcatenatorTest {
    private val stringConcatenator = Mockito
      .mock(StringConcatenator::class.java)
    private val specificStringConcatenator = 
      SpecificStringConcatenator(stringConcatenator)
    @Test
    fun concatenateSpecificStrings() {
        val expected = "expected"
        Mockito.'when'(stringConcatenator.concatenate(R.string.string_1, 
          R.string.string_2))
            .thenReturn(expected)
        val result = specificStringConcatenator
          .concatenateSpecificStrings()
        assertEquals(expected, result)
    }
}
```

在这里，我们模拟了先前的`StringConcatenator`并指示模拟返回一个特定的结果。如果我们运行测试，它将失败，因为 Mockito 仅限于模拟最终类。在这里，它遇到了与 Kotlin 冲突的问题，除非我们将类指定为*open*，否则所有类默认都是*final*。幸运的是，我们可以应用一个配置来解决这个问题，而不必使受测试的类为*open*：

1.  在`test`包中创建一个名为`resources`的文件夹。

1.  在`resources`文件夹中，创建一个名为`mockito-extensions`的文件夹。

1.  在`mockito-extensions`文件夹中，创建一个名为`org.mockito.plugins.MockMaker`的文件。

1.  在文件中，添加以下行：

```kt
    mock-maker-inline
    ```

在无法使用 JUnit 断言的回调或异步工作的情况下，可以使用`mockito`来验证对回调或 lambdas 的调用：

```kt
class SpecificStringConcatenator(private val stringConcatenator:   StringConcatenator) {
    fun concatenateSpecificStrings(): String {
        return stringConcatenator.concatenate(R.string.string_1,           R.string.string_2)
    }
    fun concatenateWithCallback(callback: Callback) {
        callback.onStringReady(concatenateSpecificStrings())
    }
    interface Callback {
        fun onStringReady(input: String)
    }
}
```

在上面的例子中，我们添加了`concatenateWithCallback`方法，它将使用`concatenateSpecificStrings`方法的结果来调用回调。对这个方法的测试将如下所示：

```kt
    @Test
    fun concatenateWithCallback() {
        val expected = "expected"
        Mockito.`when`(stringConcatenator.concatenate(R.string.string_1,           R.string.string_2))
            .thenReturn(expected)
        val callback =           Mockito.mock(SpecificStringConcatenator.Callback::class.java)
        specificStringConcatenator.concatenateWithCallback(callback)
        Mockito.verify(callback).onStringReady(expected)
    }
```

这里，我们创建了一个模拟的`Callback`对象，然后可以验证最终的预期结果。请注意，我们不得不重复设置`concatenateSpecificStrings`方法来测试`concatenateWithCallback`方法。您不应该模拟正在测试的对象；然而，您可以使用`spy`来更改它们的行为。我们可以对`stringConcatenator`对象进行监视，以改变`concatenateSpecificStrings`方法的结果：

```kt
    @Test
    fun concatenateWithCallback() {
        val expected = "expected"
        val spy = Mockito.spy(specificStringConcatenator)
        Mockito.`when`(spy.concatenateSpecificStrings())          .thenReturn(expected)
        val callback           = Mockito.mock(SpecificStringConcatenator.Callback::class.java)
        specificStringConcatenator.concatenateWithCallback(callback)
        Mockito.verify(callback).onStringReady(expected)
    }
```

Mockito 还依赖于依赖注入来初始化类变量，并且有一个自定义构建 JUnit 测试运行器。这可以简化我们变量的初始化，如下所示：

```kt
@RunWith(MockitoJUnitRunner::class)
class SpecificStringConcatenatorTest {
    @Mock
    lateinit var stringConcatenator: StringConcatenator
    @InjectMocks
    lateinit var specificStringConcatenator: SpecificStringConcatenator
}
```

在上面的例子中，`MockitoRunner`将使用`@Mock`注释的变量用模拟对象注入。接下来，它将创建一个新的非模拟实例字段，并用`@InjectionMocks`注释。当创建这个实例时，Mockito 将尝试注入符合该对象构造函数签名的模拟对象。

## mockito-kotlin

您可能已经注意到，在前面的示例中，来自 Mockito 的`when`方法已经转义。这是因为与 Kotlin 编程语言冲突。Mockito 主要是为 Java 构建的，当 Kotlin 创建时，它引入了这个关键字。类似这样的冲突可以使用`` ` ``字符。这个，连同其他一些小问题，导致在 Kotlin 中使用 Mockito 时有些不便。引入了一些库来包装 Mockito 并在使用时提供更好的体验。其中之一是`mockito-kotlin`。您可以使用以下命令将此库添加到您的模块中：

```kt
testImplementation "com.nhaarman.mockitokotlin2:mockito-kotlin:2.2.0"
```

这个库添加的一个明显的变化是用`whenever`替换`when`方法。另一个有用的变化是用泛型替换`mock`方法，而不是类对象。其余的语法与 Mockito 语法类似。

现在我们可以使用新的库更新之前的测试，从`StringConcatenatorTest`开始（为了简洁起见，导入语句已被删除）：

```kt
class StringConcatenatorTest {
    private val context = mock<Context>()
    private val stringConcatenator = StringConcatenator(context)
    @Test
    fun concatenate() {
        val stringRes1 = 1
        val stringRes2 = 2
        val string1 = "string1"
        val string2 = "string2"
        whenever(context.getString(stringRes1)).thenReturn(string1)
        whenever(context.getString(stringRes2)).thenReturn(string2)
        val result =           stringConcatenator.concatenate(stringRes1, stringRes2)
        assertEquals(string1.plus(string2), result)
    }
}
```

正如你所看到的，`` ` ``字符消失了，我们对`Context`对象的模拟初始化也简化了。我们可以对`SpecificConcatenatorTest`类应用同样的东西（为了简洁起见，已经删除了导入语句）:

```kt
@RunWith(MockitoJUnitRunner::class)
class SpecificStringConcatenatorTest {
    @Mock
    lateinit var stringConcatenator: StringConcatenator
    @InjectMocks
    lateinit var specificStringConcatenator: SpecificStringConcatenator
    @Test
    fun concatenateSpecificStrings() {
        val expected = "expected"
        whenever(stringConcatenator.concatenate(R.string.string_1,           R.string.string_2))
            .thenReturn(expected)
        val result =           specificStringConcatenator.concatenateSpecificStrings()
        assertEquals(expected, result)
    }
    @Test
    fun concatenateWithCallback() {
        val expected = "expected"
        val spy = spy(specificStringConcatenator)
        whenever(spy.concatenateSpecificStrings()).thenReturn(expected)
        val callback = mock<SpecificStringConcatenator.Callback>()
        specificStringConcatenator.concatenateWithCallback(callback)
        verify(callback).onStringReady(expected)
    }
}
```

## 练习 9.01: 测试数字的总和

使用 JUnit、Mockito 和 `mockito-kotlin` 为下面的类编写一组测试，这些测试应该覆盖以下场景：

+   断言`0`、`1`、`5`、`20`和`Int.MAX_VALUE`的值。

+   断言负数的结果。

+   修复代码，并用公式 *n*(n+1)/2* 替换数字的求和部分。

    注

    在整个练习过程中，未显示导入语句。要查看完整的代码文件，请参考[`packt.live/35TW8JI`](http://packt.live/35TW8JI)：

要测试的代码如下。

```kt
class NumberAdder {
    @Throws(InvalidNumberException::class)
    fun sum(n: Int, callback: (BigInteger) -> Unit) {
        if (n < 0) {
            throw InvalidNumberException
        }
        var result = BigInteger.ZERO
        for (i in 1..n){
          result = result.plus(i.toBigInteger())
        }
        callback(result)

    }
    object InvalidNumberException : Throwable()
}
```

执行以下步骤完成这个练习：

1.  让我们确保必要的库被添加到`app/build.gradle` 文件中：

```kt
     testImplementation 'junit:junit:4.13.1'
     testImplementation 'org.mockito:mockito-core:3.6.0'
     testImplementation 'com.nhaarman.mockitokotlin2:mockito-kotlin:2.2.0'
    ```

1.  创建一个名为`NumberAdder`的类，然后将上述代码复制到其中。

1.  将光标移动到新创建的类内部，然后使用 *Command* + *Shift* + *T* 或 *Ctrl* + *Shift* + *T* 创建一个名为`NumberAdderParameterTest`的测试类。

1.  在这个类内创建一个参数化测试，它将断言对`0`、`1`、`5`、`20`和`Int.MAX_VALUE`值的结果：

```kt
    @RunWith(Parameterized::class)
    class NumberAdderParameterTest(
        private val input: Int,
        private val expected: BigInteger
    ) {
        companion object {
            @Parameterized.Parameters
            @JvmStatic
            fun getData(): List<Array<out Any>> = listOf(
                arrayOf(0, BigInteger.ZERO),
                arrayOf(1, BigInteger.ONE),
                arrayOf(5, 15.toBigInteger()),
                arrayOf(20, 210.toBigInteger()),
                arrayOf(Int.MAX_VALUE, BigInteger("2305843008139952128"))
            )
        }
        private val numberAdder = NumberAdder()
        @Test
        fun sum() {
            val callback = mock<(BigInteger) -> Unit>()
            numberAdder.sum(input, callback)
            verify(callback).invoke(expected)
        }
    }
    ```

1.  创建一个专门处理负数抛出异常的测试类，名为`NumberAdderErrorHandlingTest`：

```kt
    @RunWith(MockitoJUnitRunner::class)
    class NumberAdderErrorHandlingTest {
        @InjectMocks
        lateinit var numberAdder: NumberAdder
        @Test(expected = NumberAdder.InvalidNumberException::class)
        fun sum() {
            val input = -1
            val callback = mock<(BigInteger) -> Unit>()
            numberAdder.sum(input, callback)
        }
    }
    ```

1.  由于 *1 + 2 + ...n = n * (n + 1) / 2*，我们可以在代码中使用这个公式，这将使方法的执行更快：

```kt
    class NumberAdder {
        @Throws(InvalidNumberException::class)
        fun sum(n: Int, callback: (BigInteger) -> Unit) {
            if (n < 0) {
                throw InvalidNumberException
            }
             callback(n.toBigInteger().times((n.toBigInteger() +            1.toBigInteger())).divide(2.toBigInteger()))
        }
        object InvalidNumberException : Throwable()
    }
    ```

通过右键单击测试所在的包并选择`Run all in [package_name]`来运行测试。将出现类似以下的输出，表示测试已通过：

![图 9.5: 练习 9.01 的输出](img/B15216_09_05.jpg)

图 9.5: 练习 9.01 的输出

通过完成这个练习，我们已经迈出了单元测试的第一步，成功为一个操作创建了多个测试用例，初步了解了 Mockito，并通过测试指导我们如何重构代码，而不引入新问题。

# 集成测试

假设您的项目已通过单元测试覆盖了大部分逻辑。现在您需要将这些经过测试的类添加到活动或片段中，并要求它们更新您的 UI。您如何确定这些类能够良好地协同工作？这个问题的答案就在于集成测试。这种测试的理念是确保应用程序内的不同组件能够良好地集成在一起。一些示例包括如下内容：

+   确保与您的存储组件良好地解析数据并进行良好的交互。  

+   存储组件能够正确地存储和检索数据。  

+   UI 组件加载和显示适当的数据。  

+   应用程序中不同屏幕之间的过渡。  

为了帮助集成测试，有时会以“给定 - 当 - 然后”格式编写要求。这些通常代表用户故事的验收标准。看下面的例子：  

```kt
Given I am not logged in
And I open the application
When I enter my credentials
And click Login
Then I see the Main screen
```

我们可以使用这些步骤来解决如何为正在开发的功能编写集成测试。  

在 Android 平台上，可以通过两个库实现集成测试：  

+   **Robolectric**：此库使开发人员能够对 Android 组件进行单元测试；也就是在没有实际设备或模拟器的情况下执行集成测试。  

+   **Espresso**：这个库对于在 Android 设备或模拟器上进行仪器测试非常有用。

我们将在下一节详细研究这些库。  

## Robolectric

Robolectric 最初是一个开源库，旨在让用户能够在本地测试中单元测试 Android 框架的类，而不是仪器测试的一部分。最近，它得到了 Google 的认可，并已与 AndroidX Jetpack 组件集成。该库的主要好处之一是简化了活动和碎片的测试。这在集成测试时也很有用，因为我们可以使用此功能确保我们的组件互相良好集成。一些 Robolectric 的特点如下：  

+   实例化和测试活动和碎片生命周期的可能性

+   测试视图膨胀的可能性  

+   为不同的 Android API、方向、屏幕大小、布局方向等提供配置的可能性  

+   改变`Application`类的可能性，从而有助于更改模块以允许插入数据模拟  

为了添加 Robolectric 以及 AndroidX 集成，我们需要以下库：  

```kt
    testImplementation 'org.robolectric:robolectric:4.3'
    testImplementation 'androidx.test.ext:junit:1.1.1'
```

第二个库将带来一组测试 Android 组件所需的`utility`方法和类。  

假设我们必须交付一个功能，其中我们显示文本`Result x`，其中`x`是用户将在`EditText`元素中插入的数字的阶乘函数。为了实现这一点，我们有两个类，一个计算阶乘，另一个是如果数字是正数，则将单词`Result`与阶乘连接起来，如果数字是负数，则返回文本`Error`。阶乘类将如下所示（在此示例中，为简洁起见，省略了导入语句）：  

```kt
class FactorialGenerator {
    @Throws(FactorialNotFoundException::class)
    fun factorial(n: Int): BigInteger {
        if (n < 0) {
            throw FactorialNotFoundException
        }
        var result = BigInteger.ONE
        for (i in 1..n) {
            result = result.times(i.toBigInteger())
        }
        return result
    }
    object FactorialNotFoundException : Throwable()
}
```

`TextFormatter`类将如下所示：  

```kt
class TextFormatter(
    private val factorialGenerator: FactorialGenerator,
    private val context: Context
) {
    fun getFactorialResult(n: Int): String {
        return try {
            context.getString(R.string.result,               factorialGenerator.factorial(n).toString())
        } catch (e: FactorialGenerator.FactorialNotFoundException) {
            context.getString(R.string.error)
        }
    }
}
```

我们可以在我们的活动中组合这两个组件，类似于这样：  

```kt
class MainActivity : AppCompatActivity() {
    private lateinit var textFormatter: TextFormatter
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        ttextFormatter = TextFormatter(FactorialGenerator(), 
          applicationContext)
        findViewById<Button>(R.id.button).setOnClickListener {
            findViewById<TextView>(R.id.text_view).text               = textFormatter.getFactorialResult(findViewById<EditText>                (R.id.edit_text).text.toString().toInt())
        }
    }
}
```

在这种情况下，我们可以观察到三个组件相互作用。我们可以使用 Robolectric 来测试我们的 activity。通过测试创建组件的 activity，我们还可以测试这三个组件之间的交互。我们可以编写如下的测试：

```kt
@RunWith(AndroidJUnit4::class)
class MainActivityTest {
    private val context = getApplicationContext<Application>()
    @Test
    fun `show factorial result in text view`() {
        val scenario = launch<MainActivity>(MainActivity::class.java)
        scenario.moveToState(Lifecycle.State.RESUMED)
        scenario.onActivity { activity ->
            activity.edit_text.setText(5.toString())
            activity.button.performClick()
            assertEquals(context.getString(R.string.result,               "120"),activity.text_view.text)
        }
    }
}
```

在前面的例子中，我们可以看到 AndroidX 对 activity 测试的支持。`AndroidJUnit4` 测试运行器将设置 Robolectric 并创建必要的配置，而 `launch` 方法将返回一个 `scenario` 对象，我们可以通过这个对象来实现测试所需的条件。

如果我们想为测试添加配置，可以在类和每个测试方法上使用 `@Config` 注释：

```kt
@Config(
    sdk = [Build.VERSION_CODES.P],
    minSdk = Build.VERSION_CODES.KITKAT,
    maxSdk = Build.VERSION_CODES.Q,
    application = Application::class,
    assetDir = "/assetDir/"
)
@RunWith(AndroidJUnit4::class)
class MainActivityTest 
```

我们也可以在 `test/resources` 文件夹中的 `robolectric.properties` 文件中指定全局配置，就像这样：

```kt
sdk=28
minSdk = 14
maxSdk = 29
```

最近添加到 Robolectric 的另一个重要功能是对 Espresso 库的支持。这允许开发人员使用 Espresso 的语法与视图进行交互并对视图进行断言。可以与 Robolectric 结合使用的另一个库是 `FragmentScenario`，它允许测试片段的可能性。可以通过以下方式在 Gradle 中添加这些库：

```kt
    testImplementation 'androidx.fragment:fragment-testing:1.1.0'
    testImplementation 'androidx.test.espresso:espresso-core:3.2.0'
```

使用 `scenario` 设置测试片段与使用 `scenario` 设置测试活动类似：

```kt
val scenario = launchFragmentInContainer<MainFragment>()
scenario.moveToState(Lifecycle.State.CREATED)
```

## Espresso

Espresso 是一个旨在以简洁方式执行交互和断言的库。最初设计为用于仪表化测试，现在已迁移到与 Robolectric 一起使用。执行操作的典型用法如下：

```kt
onView(Matcher<View>).perform(ViewAction)
```

用于验证，我们可以使用以下内容：

```kt
onView(Matcher<View>).check(ViewAssertion)
```

如果在 `ViewMatchers` 类中找不到自定义的 `ViewMatchers`，我们可以自定义。其中最常见的是 `withId` 和 `withText`。这两个允许我们基于它们的 `R.id.myId` 标识符或文本标识符识别视图。理想情况下，第一个应该用于识别特定的视图。Espresso 的另一个有趣之处是依赖于 `Hamcrest` 库进行匹配。这是一个旨在改进测试的 Java 库。如果必要，这允许组合多个匹配器。比如说，您的 UI 上具有相同的 ID，可以使用以下表达式缩小搜索范围以找到特定视图：

```kt
onView(allOf(withId(R.id.edit_text), withParent(withId(R.id.root))))
```

`allOf` 表达式将评估所有其他操作符，并且只有在内部所有操作符都通过时才会通过。前述的表达式将转换为“找到具有 `id=edit_text` 的父视图为 `id=R.id.root` 的视图。” 其他 `Hamcrest` 操作符可能包括 `anyOf`, `both`, `either`, `is`, `isA`, `hasItem`, `equalTo`, `any`, `instanceOf`, `not`, `null` 和 `notNull`。

`ViewActions` 与 `ViewMatchers` 有类似的方法。我们可以在 `ViewActions` 类中找到常见的方法。常见的包括 `typeText`、`click`、`scrollTo`、`clearText`、`swipeLeft`、`swipeRight`、`swipeUp`、`swipeDown`、`closeSoftKeyboard`、`pressBack`、`pressKey`、`doubleClick` 和 `longClick`。如果您有自定义视图并且需要特定操作，则可以通过实现 `ViewAction` 接口来实现自己的 `ViewAction` 元素。

与前面的示例类似，`ViewAssertions` 有自己的类。通常，可以使用 `matches` 方法，然后使用 `ViewMatchers` 和 `Hamcrest` 匹配器来验证结果：

```kt
onView(withId(R.id.text_view)).check(matches(withText("My text")))) 
```

前面的示例将验证具有 `text_view` ID 的视图中是否包含文本 `My text`：

```kt
onView(withId(R.id.button)).perform(click())
```

这将点击具有 ID 按钮的视图。

现在我们可以重写 Robolectric 测试并添加 Espresso，这将给我们带来以下内容（未显示导入语句）：

```kt
@RunWith(AndroidJUnit4::class)
class MainActivityTest {
    @Test
    fun `show factorial result in text view`() {
        val scenario = launch<MainActivity>(MainActivity::class.java)
        scenario.moveToState(Lifecycle.State.RESUMED)
        scenario.onActivity { activity ->
            onView(withId(R.id.edit_text)).perform(typeText("5"))
            onView(withId(R.id.button)).perform(click())
            onView(withId(R.id.text_view))
              .check(matches(withText(activity                 .getString(R.string.result, "120"))))
        }
    }
}
```

在前面的代码示例中，我们可以观察到如何使用 Espresso 输入数字 `5` 到 `EditText` 中，然后点击按钮，然后使用 `onView()` 方法获取到 `TextView` 中显示的文本，并使用 `perform()` 执行操作或使用 `check()` 进行断言。

注意

对于以下练习，您需要一个启用了 USB 调试的模拟器或实际设备。您可以通过在 Android Studio 中选择 `工具` | `AVD 管理器` 来创建一个，然后选择 `创建虚拟设备` 选项，并选择虚拟机类型，点击 `下一步`，然后选择 x86 映像。对于此练习来说，任何大于 Lollipop 的映像都应该可以。接下来，您可以给您的映像命名并单击 `完成`。

## 练习 9.02：双重积分

开发一个应用程序，观察以下要求：

```kt
Given I open the application
And I insert the number n
When I press the Calculate button
Then I should see the text "The sum of numbers from 1 to n is [result]"
Given I open the application
And I insert the number -n
When I press the Calculate button
Then I should see the text "Error: Invalid number"
```

您应该使用 Robolectric 和 Espresso 实现单元测试和集成测试，并将集成测试迁移到成为仪器化测试。

注意

在整个练习的过程中，我们没有显示导入语句。要查看完整的代码文件，请参考[`packt.live/2M1MtcY`](http://packt.live/2M1MtcY)：

实施以下步骤来完成此练习：

1.  让我们首先将必要的测试库添加到 `app/build.gradle` 中：

```kt
        testImplementation 'junit:junit:4.13.1'
        testImplementation 'org.mockito:mockito-core:3.6.0'
        testImplementation 'com.nhaarman.mockitokotlin2
          :mockito-kotlin:2.2.0'
        testImplementation 'org.robolectric:robolectric:4.4'
        testImplementation 'androidx.test.ext:junit:1.1.2'
        testImplementation 'androidx.test.espresso:espresso-core:3.3.0'
        androidTestImplementation 'androidx.test.ext:junit:1.1.2'
        androidTestImplementation 'androidx.test
          .espresso:espresso-core:3.3.0'
        androidTestImplementation 'androidx.test:rules:1.3.0'
    ```

1.  对于 Robolectric，我们需要添加额外的配置，首先在 `android` 闭包中的 `app/build.gradle` 中添加以下行：

```kt
    testOptions.unitTests.includeAndroidResources = true
    ```

1.  在 `test` 包中创建一个名为 `resources` 的目录。

1.  添加 `robolectric.properties` 文件，并在该文件中添加以下配置：

```kt
    sdk=28
    ```

1.  在 `test` 包中创建一个名为 `resources` 的文件夹。

1.  在 `resources` 中，创建一个名为 `mockito-extensions` 的文件夹。

1.  在 `mockito-extensions` 文件夹中，创建一个名为 `org.mockito.plugins.MockMaker` 的文件，并在文件中添加以下行：

```kt
    mock-maker-inline
    ```

1.  创建 `NumberAdder` 类。这与 *练习 9.01* 中的类似：

```kt
    import java.math.BigInteger
    class NumberAdder {
        @Throws(InvalidNumberException::class)
        fun sum(n: Int, callback: (BigInteger) -> Unit) {
            if (n < 0) {
                throw InvalidNumberException
            }
            callback(n.toBigInteger().times((n.toLong()           + 1).toBigInteger()).divide(2.toBigInteger()))
        }
        object InvalidNumberException : Throwable()
    }
    ```

1.  在 `test` 文件夹中为 `NumberAdder` 创建测试。首先，创建 `NumberAdderParameterTest`：

```kt
    @RunWith(Parameterized::class)
    class NumberAdderParameterTest(
        private val input: Int,
        private val expected: BigInteger
    ) {
        companion object {
            @Parameterized.Parameters
            @JvmStatic
            fun getData(): List<Array<out Any>> = listOf(
                arrayOf(0, BigInteger.ZERO),
                arrayOf(1, BigInteger.ONE),
                arrayOf(5, 15.toBigInteger()),
                arrayOf(20, 210.toBigInteger()),
                arrayOf(Int.MAX_VALUE, BigInteger("2305843008139952128"))
            )
        }
        private val numberAdder = NumberAdder()
        @Test
        fun sum() {
            val callback = mock<(BigInteger) -> Unit>()
            numberAdder.sum(input, callback)
            verify(callback).invoke(expected)
        }
    }
    ```

1.  然后，创建`NumberAdderErrorHandlingTest`测试：

```kt
    @RunWith(MockitoJUnitRunner::class)
    class NumberAdderErrorHandlingTest {
        @InjectMocks
        lateinit var numberAdder: NumberAdder
        @Test(expected = NumberAdder.InvalidNumberException::class)
        fun sum() {
            val input = -1
            val callback = mock<(BigInteger) -> Unit>()
            numberAdder.sum(input, callback)
        }
    }
    ```

1.  创建一个将总和格式化并与必要的字符串连接的类：

```kt
    class TextFormatter(
        private val numberAdder: NumberAdder,
        private val context: Context
    ) {
        fun getSumResult(n: Int, callback: (String) -> Unit) {
            try {
                numberAdder.sum(n) {
                    callback(
                        context.getString(
                            R.string.the_sum_of_numbers_from_1_to_is,
                            n,
                            it.toString()
                        )
                    )
                }
            } catch (e: NumberAdder.InvalidNumberException) {
                callback(context.getString
                  (R.string.error_invalid_number))
            }
        }
    }
    ```

1.  为这个类的成功和错误情况进行单元测试。从成功场景开始：

```kt
    @RunWith(MockitoJUnitRunner::class)
    class TextFormatterTest {
        @InjectMocks
        lateinit var textFormatter: TextFormatter
        @Mock
        lateinit var numberAdder: NumberAdder
        @Mock
        lateinit var context: Context
        @Test
        fun getSumResult_success() {
            val n = 10
            val sumResult = BigInteger.TEN
            val expected = "expected"
            whenever(numberAdder.sum(eq(n), any())).thenAnswer {
                (it.arguments[1] as (BigInteger)->Unit)
                    .invoke(sumResult)
            }
            whenever(context.getString
              (R.string.the_sum_of_numbers_from_1_to_is, n, 
                sumResult.toString())).thenReturn(expected)
            val callback = mock<(String)->Unit>()
            textFormatter.getSumResult(n, callback)
            verify(callback).invoke(expected)
        }
    ```

然后，为错误场景创建测试：

```kt
        @Test
        fun getSumResult_error() {
            val n = 10
            val expected = "expected"
            whenever(numberAdder.sum(eq(n),           any())).thenThrow(NumberAdder.InvalidNumberException)
            whenever(context.getString(R.string.error_invalid_number))          .thenReturn(expected)
            val callback = mock<(String)->Unit>()
            textFormatter.getSumResult(n, callback)
            verify(callback).invoke(expected)
        }
    }
    ```

1.  为`activity_main.xml`创建布局：

```kt
    <?xml version="1.0" encoding="utf-8"?>
    <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
        android:id="@+id/root"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:orientation="vertical">
        <EditText
            android:id="@+id/edit_text"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:inputType="number" />
        <Button
            android:id="@+id/button"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_gravity="center_horizontal"
            android:text="@string/calculate" />
        <TextView
            android:id="@+id/text_view"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_gravity="center_horizontal" />
    </LinearLayout>
    ```

1.  创建包含所有其他组件的`MainActivity`类：

```kt
    class MainActivity : AppCompatActivity() {
        private lateinit var textFormatter: TextFormatter

        override fun onCreate(savedInstanceState: Bundle?) {
            super.onCreate(savedInstanceState)
            setContentView(R.layout.activity_main)
            textFormatter = TextFormatter(NumberAdder(), applicationContext)
            findViewById<Button>(R.id.button).setOnClickListener {
                textFormatter.getSumResult(findViewById<EditText>              (R.id.edit_text).text.toString().toIntOrNull() ?: 0) {
                    findViewById<TextView>(R.id.text_view).text = it
                }
            }
        }
    }
    ```

1.  创建`MainActivity`的测试，并将其放在`test`目录中。它将包含两个测试方法，一个用于成功，一个用于错误：

```kt
    @RunWith(AndroidJUnit4::class)
    class MainActivityTest {
        @Test
        fun `show sum result in text view`() {
            val scenario = launch<MainActivity>(MainActivity::class.java)
            scenario.moveToState(Lifecycle.State.RESUMED)
            scenario.onActivity { activity ->
                onView(withId(R.id.edit_text)).perform(replaceText("5"))
                onView(withId(R.id.button)).perform(click())
                onView(withId(R.id.text_view)).check(matches(withText
                 (activity.getString
                   (R.string.the_sum_of_numbers_from_1_to_is, 5, "15"))))
            }
        }
        @Test
        fun `show error in text view`() {
            val scenario = launch<MainActivity>(MainActivity::class.java)
            scenario.moveToState(Lifecycle.State.RESUMED)
            scenario.onActivity { activity ->
                onView(withId(R.id.edit_text))
                    .perform(replaceText("-5"))
                onView(withId(R.id.button)).perform(click())
                onView(withId(R.id.text_view)).check(
                    matches(withText(activity.getString(
                    R.string.error_invalid_number))))
            }
        }
    }
    ```

如果你通过右键单击包含测试的包并选择“在[package_name]中全部运行”来运行测试，那么会出现类似以下的输出：

![图 9.6：执行 Exercise 9.02 test 文件夹中的测试的结果](img/B15216_09_06.jpg)

图 9.6：执行 Exercise 9.02 test 文件夹中的测试的结果

如果你执行前面的测试，你应该会看到类似*图 9.6*的输出。Robolectric 测试的执行方式与常规单元测试相同；但是，执行时间会增加。

1.  现在让我们将前面的测试迁移到一个有仪器的集成测试中。为了做到这一点，我们将把前面的测试从`test`包复制到`androidTest`包，并删除我们的测试中与场景相关的代码。复制文件之后，我们将使用`ActivityTestRule`，它将在每次测试执行之前启动我们的活动。我们还需要重命名类以避免重复，并且重命名测试方法，因为这种语法不支持仪器化测试：

```kt
    @RunWith(AndroidJUnit4::class)
    class MainActivityUiTest {
        @JvmField
        @Rule
        var activityRule: ActivityTestRule<MainActivity> = 
          ActivityTestRule(MainActivity::class.java)
        @Test
        fun showSumResultInTextView() {
            activityRule.activity.let { activity ->
                onView(withId(R.id.edit_text)).perform(replaceText("5"))
                onView(withId(R.id.button)).perform(click())
                onView(withId(R.id.text_view)).check(matches
                 (withText(activity.getString
                  (R.string.the_sum_of_numbers_from_1_to_is, 5, "15"))))
            }
        }
        @Test
        fun showErrorInTextView() {
            activityRule.activity.let { activity ->
                onView(withId(R.id.edit_text)).perform(replaceText("-5"))
                onView(withId(R.id.button)).perform(click())
                onView(withId(R.id.text_view)).check(matches               (withText(activity.getString                 (R.string.error_invalid_number))))
            }
        }
    }
    ```

    如果你通过右键单击包含测试的包并选择“在[package_name]中全部运行”来运行测试，那么会出现类似以下的输出：

![图 9.7：执行 Exercise 9.02 androidTest 文件夹中的测试的结果](img/B15216_09_07.jpg)

图 9.7：执行 Exercise 9.02 androidTest 文件夹中的测试的结果

在*图 9.7*中，我们可以看到 Android Studio 显示的结果输出。如果你在测试执行时仔细观察仿真器，你会发现对于每个测试，你的活动都会被打开，输入会被设置在字段中，并且按钮会被点击。我们的集成测试（在工作站和仿真器上）都尝试匹配需求的接受标准。集成测试验证相同的行为，唯一的区别是一个在本地检查，另一个在 Android 设备或仿真器上检查。这里的主要好处是 Espresso 能够弥合它们之间的鸿沟，使得集成测试更容易设置和执行。

# UI 测试

UI 测试是开发人员可以模拟用户行程并验证应用程序不同模块之间的交互的测试，也被称为端到端测试。对于小型应用程序，您可以拥有一个测试套件，但对于较大的应用程序，您应该分割您的测试套件以覆盖特定的用户行程（登录，创建账户，设置流程等）。因为它们在设备上执行，所以您需要在`androidTest`包中编写它们，这意味着它们将使用**Instrumentation**框架来运行。Instrumentation 的工作方式如下：

+   应用程序已构建并安装在设备上。

+   将在设备上安装一个测试应用程序，用于监视您的应用程序。

+   测试应用程序将执行对您的应用程序的测试并记录结果。

其中一个缺点是测试将共享持久化数据，因此如果一个测试在设备上存储数据，那么第二个测试可以访问该数据，这意味着存在失败的风险。另一个缺点是，如果测试遭遇崩溃，这将停止整个测试，因为被测试的应用程序已经停止。在 Jetpack 更新中引入了`app/build.gradle`文件来解决这些问题：

```kt
android {
    ...
    defaultConfig {
        ...
        testInstrumentationRunner           "androidx.test.runner.AndroidJUnitRunner"
        testInstrumentationRunnerArguments clearPackageData: 'true'
    }
    testOptions {
        execution 'ANDROIDX_TEST_ORCHESTRATOR'
    }
}
dependencies {
    ...
    androidTestUtil 'androidx.test:orchestrator:1.3.0'
}
```

您可以使用 Gradle 的`connectedCheck`命令在连接的设备上执行协调器测试，无论是从`Terminal`还是从 Gradle 命令列表中。

在配置中，您将注意到以下行：`testInstrumentationRunner`。这使我们能够为测试创建自定义配置，这给了我们机会将模拟数据注入到模块中：

```kt
testInstrumentationRunner "com.android.CustomTestRunner"
```

`CustomTestRunner`如下（以下代码段未显示导入语句）：

```kt
class CustomTestRunner: AndroidJUnitRunner() {
    @Throws(Exception::class)
    override fun newApplication(
        cl: ClassLoader?,
        className: String?,
        context: Context?
    ): Application? {
        return super.newApplication(cl,           MyApplication::class.java.name, context)
    }
}
```

测试类本身可以通过应用 JUnit4 语法，并借助`androidx.test.ext.junit.runners.AndroidJUnit4`测试运行程序来编写：

```kt
@RunWith(AndroidJUnit4::class)
class MainActivityUiTest {
}
```

来自 AndroidX 测试支持的另一个重要功能是活动规则。当使用默认构造函数时，此规则将在每次测试之前启动活动，并准备好进行交互和断言：

```kt
    @JvmField
    @Rule
    var activityRule: ActivityTestRule<MainActivity>       = ActivityTestRule(MainActivity::class.java)
```

您还可以使用规则来避免启动活动，并自定义用于启动测试的意图：

```kt
    @JvmField
    @Rule
    var activityRule: ActivityTestRule<MainActivity> =       ActivityTestRule(MainActivity::class.java, false ,false)
    @Test
    fun myTestMethod() {
        val myIntent = Intent()
        activityRule.launchActivity(myIntent)
    }
```

`@Test`方法本身在专用测试线程中运行，这就是为什么类似 Espresso 这样的库很有用。Espresso 将自动将与 UI 线程上的视图的每次交互移动。Espresso 可以与 Robolectric 测试一样用于 UI 测试：

```kt
    @Test
    fun myTest() {
        onView(withId(R.id.edit_text)).perform(replaceText("5"))
        onView(withId(R.id.button)).perform(click())
        onView(withId(R.id.text_view))          .check(matches(withText("my test")))
    }
```

通常，在 UI 测试中，您将发现可能重复的交互和断言。为了避免在您的代码中重复多个场景，您可以应用一种称为`Robot`类的模式，其中交互和断言可以分组到特定方法中。您的测试代码将使用这些机器人并进行断言。典型的机器人看起来会像这样：

```kt
class MyScreenRobot {
    fun setText(): MyScreenRobot {
        onView(ViewMatchers.withId(R.id.edit_text))          .perform(ViewActions.replaceText("5"))
        return this
    }
    fun pressButton(): MyScreenRobot {
        onView(ViewMatchers.withId(R.id.button))          .perform(ViewActions.click())
        return this
    }
    fun assertText(): MyScreenRobot {
        onView(ViewMatchers.withId(R.id.text_view))          .check(ViewAssertions.matches(ViewMatchers           .withText("my test")))
        return this
    }
}
```

测试结果将如下所示：

```kt
    @Test
    fun myTest() {
       MyScreenRobot()
           .setText()
           .pressButton()
           .assertText()
    }
```

因为应用程序可能是多线程的，有时需要一段时间从各种来源（互联网、文件、本地存储等）加载数据，UI 测试将必须知道何时 UI 可用以进行交互。一个实现这一点的方法是通过使用空闲资源。这些是可以在测试之前向 Espresso 注册并注入到您的应用程序组件中的对象，在这些组件中进行多线程工作。应用程序将在工作进行中将它们标记为非空闲，并且当工作完成时为空闲。正是在这一点上，Espresso 将开始执行测试。其中最常用的之一是`CountingIdlingResource`。这个特定的实现使用一个计数器，在您希望 Espresso 等待您的代码完成执行时应该增加它，并在您希望让 Espresso 验证您的代码时减少它。当计数器达到`0`时，Espresso 将恢复测试。具有空闲资源的组件示例看起来像这样：

```kt
class MyHeavyliftingComponent(private val   countingIdlingResource:CountingIdlingResource) {
    fun doHeavyWork() {
        countingIdlingResource.increment()
        // do work
        countingIdlingResource.decrement()
    }
}
```

`Application`类可以用来注入空闲资源，就像这样：

```kt
class MyApplication : Application(){
    val countingIdlingResource = CountingIdlingResource("My heavy work")
    val myHeavyliftingComponent =       MyHeavyliftingComponent(countingIdlingResource)
}
```

然后，在测试中，我们可以访问`Application`类并将资源注册到 Espresso：

```kt
@RunWith(AndroidJUnit4::class)
class MyTest {
    @Before
    fun setUp() {
        val myApplication = getApplicationContext<MyApplication>()
        IdlingRegistry.getInstance()          .register(myApplication.countingIdlingResource)
    }
}
```

Espresso 配备了一组扩展，可用于断言不同的 Android 组件。其中一种扩展是意图测试。在想要单独测试活动时（更适用于集成测试）这将会很有用。为了使用它，您需要将该库添加到 Gradle 中：

```kt
androidTestImplementation 'androidx.test.espresso:espresso-intents:3.3.0'
```

添加完库后，您需要使用`IntentsTestRule`来设置必要的意图监控。该规则是`ActivityTestRule`的子类：

```kt
    @JvmField
    @Rule
    var intentsRule: IntentsTestRule<MainActivity>       = IntentsTestRule(MainActivity::class.java)
```

为了断言意图的值，您需要触发适当的操作，然后使用`intended`方法：

```kt
        onView(withId(R.id.button)).perform(click())
        intended(allOf(
            hasComponent(hasShortClassName(".MainActivity")),
            hasExtra(MainActivity.MY_EXTRA, "myExtraValue")))
```

`intended`方法的工作方式类似于`onView`方法。它需要一个可以与`Hamcrest`匹配器组合的匹配器。与 Intent 相关的匹配器可以在`IntentMatchers`类中找到。该类包含了断言`Intent`类的不同方法：extras、data、components、bundles 等等。

另一个重要的扩展库来帮助`RecyclerView`。Espresso 的`onData`方法只能测试`AdapterViews`，如`ListView`，而不能断言`RecyclerView`。为了使用该扩展，您需要向项目中添加以下库：

```kt
androidTestImplementation   'com.android.support.test.espresso:espresso-contrib:3.0.2'
```

该库提供了一个`RecyclerViewActions`类，其中包含一组方法，允许您对`RecyclerView`内的项目执行操作：

```kt
onView(withId(R.id.recycler_view))  .perform(RecyclerViewActions.actionOnItemAtPosition(0, click()))
```

前面的语句将点击位置为`0`的项目：

```kt
onView(withId(R.id.recycler_view)).perform(RecyclerViewActions   .scrollToPosition<RecyclerView.ViewHolder>(10))
```

这将滚动到列表中的第十个项目：

```kt
onView(withText("myText")).check(matches(isDisplayed()))
```

前面的代码将检查是否显示了带有`myText`文本的视图，这也适用于`RecyclerView`项。

## 练习 9.03：随机等待时间

编写一个应用程序，它将有两个屏幕。第一个屏幕将有一个按钮。当用户按下按钮时，它将等待 1 到 5 秒之间的随机时间，然后启动显示文本`x 秒后打开`的第二屏幕，其中`x`是经过的秒数。编写一个 UI 测试，以覆盖此场景，并调整以下特性以用于测试：

+   当运行测试时，`random`函数将返回值`1`。

+   `CountingIdlingResource`将用于指示计时器何时停止。

注意

在本练习中，未显示导入语句。要查看完整的代码文件，请参考[`packt.live/38V7krh`](http://packt.live/38V7krh)。

进行以下步骤来完成这个练习：

1.  将以下库添加到`app/build.gradle`：

```kt
        implementation 'androidx.test.espresso:espresso-core:3.3.0'
        testImplementation 'junit:junit:4.13.1'
        androidTestImplementation 'androidx.test.ext:junit:1.1.2'
        androidTestImplementation 'androidx.test:rules:1.3.0'
    ```

1.  然后，从`Randomizer`类开始：

```kt
    class Randomizer(private val random: Random) {
        fun getTimeToWait(): Int {
            return random.nextInt(5) + 1
        }
    }
    ```

1.  接下来，创建一个`Synchronizer`类，它将使用`Randomizer`和`Timer`等待随机时间间隔。它还将使用`CountingIdlingResource`来标记任务的开始和结束：

```kt
    class Synchronizer(
        private val randomizer: Randomizer,
        private val timer: Timer,
        private val countingIdlingResource: CountingIdlingResource
    ) {
        fun executeAfterDelay(callback: (Int) -> Unit) {
            val timeToWait = randomizer.getTimeToWait()
            countingIdlingResource.increment()
            timer.schedule(CallbackTask(callback, timeToWait),           timeToWait * 1000L)
        }
        inner class CallbackTask(
            private val callback: (Int) -> Unit,
            private val time: Int
        ) : TimerTask() {
            override fun run() {
                callback(time)
                countingIdlingResource.decrement()
            }
        }
    }
    ```

1.  现在创建一个`Application`类，负责创建前述所有类的实例：

```kt
    class MyApplication : Application() {
        val countingIdlingResource =       CountingIdlingResource("Timer resource")
        val randomizer = Randomizer(Random())
        val synchronizer = Synchronizer(randomizer, Timer(),       countingIdlingResource)
    }
    ```

1.  将`MyApplication`类添加到`AndroidManifest`中`application`标签中，带有`android:name`属性。

1.  创建一个`activity_1`布局文件，其中包含一个父布局和一个按钮：

```kt
    <?xml version="1.0" encoding="utf-8"?>
    <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:orientation="vertical">

        <Button
            android:id="@+id/activity_1_button"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_gravity="center"
            android:text="@string/press_me" />
    </LinearLayout>
    ```

1.  创建一个包含父布局和`TextView`的`activity_2`布局文件：

```kt
    <?xml version="1.0" encoding="utf-8"?>
    <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:orientation="vertical">
        <TextView
            android:id="@+id/activity_2_text_view"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_gravity="center" />
    </LinearLayout>
    ```

1.  创建`Activity1`类，它将实现按钮点击的逻辑：

```kt
    class Activity1 : AppCompatActivity() {
        override fun onCreate(savedInstanceState: Bundle?) {
            super.onCreate(savedInstanceState)
            setContentView(R.layout.activity_1)
            findViewById<Button>(R.id.activity_1_button)
              .setOnClickListener {
                (application as MyApplication).synchronizer
                  .executeAfterDelay {
                    startActivity(Activity2.newIntent(this, it))
                }
            }
        }
    }
    ```

1.  创建`Activity2`类，它将通过意图显示接收到的数据：

```kt
    class Activity2 : AppCompatActivity() {
        companion object {
            private const val EXTRA_SECONDS = "extra_seconds"
            fun newIntent(context: Context, seconds: Int) =
                Intent(context, Activity2::class.java).putExtra(
                    EXTRA_SECONDS, seconds
                )
        }
        override fun onCreate(savedInstanceState: Bundle?) {
            super.onCreate(savedInstanceState)
            setContentView(R.layout.activity_2)
            findViewById<TextView>(R.id.activity_2_text_view).text =
                getString(R.string.opened_after_x_seconds,               intent.getIntExtra(EXTRA_SECONDS, 0))
        }
    }
    ```

1.  在`androidTest`目录中创建一个`FlowTest`类，它将注册`MyApplication`对象的`IdlingResource`并断言点击的结果：

```kt
    @RunWith(AndroidJUnit4::class)
    @LargeTest
    class FlowTest {
        @JvmField
        @Rule
        var activityRule: ActivityTestRule<Activity1> =       ActivityTestRule(Activity1::class.java)
        private val myApplication = getApplicationContext<MyApplication>()
        @Before
        fun setUp() {
            IdlingRegistry.getInstance().register(myApplication           .countingIdlingResource)
        }
        @Test
        fun verifyFlow() {
            onView(withId(R.id.activity_1_button)).perform(click())
            onView(withId(R.id.activity_2_text_view))          .check(matches(withText(myApplication             .getString(R.string.opened_after_x_seconds, 1))))
        }
    }
    ```

1.  多次运行测试并检查测试结果。请注意，测试有 20%的成功机会，但它将等待按钮被点击。这意味着闲置资源正在工作。还要观察这里存在随机因素。

1.  测试不喜欢随机性，所以我们需要通过使`Randomizer`类为开放的，并在`androidTest`目录中创建一个子类来消除它。我们可以对`MyApplication`类做同样的事情，并提供一个称为`TestRandomizer`的不同随机器：

```kt
    class TestRandomizer(random: Random) : Randomizer(random) {
        override fun getTimeToWait(): Int {
            return 1
        }
    }
    ```

1.  现在，以一种我们可以从子类中重写随机器的方式修改`MyApplication`类：

```kt
    open class MyApplication : Application() {
        val countingIdlingResource =       CountingIdlingResource("Timer resource")
        lateinit var synchronizer: Synchronizer
        override fun onCreate() {
            super.onCreate()
            synchronizer = Synchronizer(createRandomizer(), Timer(),           countingIdlingResource)
        }
        open fun createRandomizer() = Randomizer(Random())
    }
    ```

1.  在`androidTest`目录中，创建`TestMyApplication`，它将扩展`MyApplication`并重写`createRandomizer`方法：

```kt
    class TestMyApplication : MyApplication() {
        override fun createRandomizer(): Randomizer {
            return TestRandomizer(Random())
        }
    }
    ```

1.  最后，创建一个仪表测试运行器，其将在测试内使用这个新的`Application`类：

```kt
    class MyApplicationTestRunner : AndroidJUnitRunner() {
        @Throws(Exception::class)
        override fun newApplication(
            cl: ClassLoader?,
            className: String?,
            context: Context?
        ): Application? {
            return super.newApplication(cl,           TestMyApplication::class.java.name, context)
        }
    }
    ```

1.  将新的测试运行器添加到 Gradle 配置中：

```kt
    android {
        ...
        defaultConfig {        
            ...
            testInstrumentationRunner            "com.android.testable.myapplication            .MyApplicationTestRunner"
        }
    }
    ```

现在运行测试，一切应该与*图 9.8*类似地通过：

![图 9.8：练习 9.03 的输出](img/B15216_09_08.jpg)

图 9.8：练习 9.03 的输出

这种类型的练习展示了如何避免测试中的随机性，并提供具体和可重复的输入，使我们的测试更加可靠。类似的方法也适用于依赖注入框架，可以在测试套件中替换整个模块，以确保测试的可靠性。最常替换的一种是 API 通信。这种方法解决的另一个问题是等待时间的减少。如果这种情况在你的测试中重复出现，那么它们的执行时间会因此增加。

# 测试驱动开发

假设你的任务是构建一个显示带有加、减、乘、除选项的计算器的活动。你还必须为你的实现编写测试。通常情况下，你会构建你的 UI 和你的活动以及一个单独的`Calculator`类。然后，你会为`Calculator`类编写单元测试，然后为你的活动类编写单元测试。

在`Calculator`测试下。在这里，你还需要在`Calculator`类中创建必要的方法，以避免编译时错误。

如果你在这个阶段运行你的测试，它们会失败。这将迫使你实现你的代码，直到测试通过。一旦你的`Calculator`测试通过，你就可以把你的计算器连接到你的 UI，直到你的 UI 测试通过。虽然这看起来像是一个违反直觉的方法，一旦掌握了这个过程，它能解决两个问题：

+   因为你会确保你的代码是可测试的，所以写代码的时间会减少，并且你只需要写必要数量的代码来使测试通过。

+   由于开发者能够分析不同的结果，会减少引入的 bug。

请看下图，显示了 TDD 循环：

![图 9.9：TDD 循环](img/B15216_09_09.jpg)

图 9.9：TDD 循环

在前述图中，我们可以看到 TDD 过程中的开发循环。你应该从测试失败的点开始。实现变更以使测试通过。当你更新或添加新特性时，你可以重复这个过程。

回到我们的阶乘例子，我们开始时有一个没有涵盖所有场景的阶乘函数，不得不在添加新测试时不断更新函数。TDD 就是以这个想法为基础的。你从一个空函数开始。你开始定义你的测试场景：成功的条件是什么？最小值是多少？最大值是多少？有没有例外情况？它们是什么？这些问题可以帮助开发者定义他们的测试案例。然后，这些案例可以被书写。接下来我们看看这如何通过下一个练习来实际做到。

## 练习 9.04：使用 TDD 计算数字之和

编写一个函数，该函数将以整数*n*作为输入，并返回 1 到*n*的数字之和。这个函数应该采用 TDD 方法编写，并且应满足以下标准：

+   对于*n<=0*，该函数将返回值`-1`。

+   该函数应能返回`Int.MAX_VALUE`的正确值。

+   该函数应该快速，即使对于`Int.MAX_VALUE`也是如此。

注

在整个练习过程中，不会显示导入语句。要查看完整的代码文件，请参考[`packt.live/3a0jJd9`](http://packt.live/3a0jJd9)：

执行以下步骤完成此练习：

1.  确保以下库已添加到`app/build.gradle`中：

```kt
    testImplementation 'junit:junit:4.13.1'
    ```

1.  创建一个具有`sum`方法的`Adder`类，该方法将返回`0`，以满足编译器：

```kt
    class Adder {
        fun sum(n: Int): Int = 0
    }
    ```

1.  在测试目录中创建一个`AdderTest`类并定义我们的测试用例。我们将有以下测试用例：*n=1*、*n=2*、*n=0*、*n=-1*、*n=10*、*n=20* 和 *n=Int.MAX_VALUE*。我们可以将成功场景分为一个方法，不成功的场景分为另一个方法：

```kt
    class AdderTest {
        private val adder = Adder()
        @Test
        fun sumSuccess() {
            assertEquals(1, adder.sum(1))
            assertEquals(3, adder.sum(2))
            assertEquals(55, adder.sum(10))
            assertEquals(210, adder.sum(20))
            assertEquals(2305843008139952128L, adder.sum(Int.MAX_VALUE))
        }
        @Test
        fun sumError(){
            assertEquals(-1, adder.sum(0))
            assertEquals(-1, adder.sum(-1))
        }
    }
    ```

1.  如果我们对`AdderTest`类运行测试，我们会看到类似以下图表的输出，意味着所有测试都失败了：![图 9.10：练习 9.04 的初始测试状态](img/B15216_09_10.jpg)

图 9.10：练习 9.04 的初始测试状态

1.  让我们先通过在循环中实现从 1 到*n*的求和来处理成功场景：

```kt
    class Adder {
        fun sum(n: Int): Long {
            var result = 0L
            for (i in 1..n) {
                result += i
            }
            return result
        }
    }
    ```

1.  如果我们现在运行测试，你会发现其中一个会通过，另一个会失败，类似于以下图表：![图 9.11：解决练习 9.04 成功场景后的测试状态](img/B15216_09_11.jpg)

图 9.11：解决练习 9.04 成功场景后的测试状态

1.  如果我们查看执行成功测试所需的时间，似乎有点长。当一个项目中存在成千上万的单元测试时，这些时间就会相加起来。我们现在可以通过应用*n(n+1)/2*的公式来优化我们的代码以解决这个问题：

```kt
    class Adder {
        fun sum(n: Int): Long {
            return (n * (n.toLong() + 1)) / 2
        }
    }
    ```

现在运行测试将显著减少速度到几毫秒。

1.  现在，让我们专注于解决我们的失败场景。我们可以通过为*n*小于或等于`0`时添加一个条件来做到这一点：

```kt
    class Adder {
        fun sum(n: Int): Long {
            return if (n > 0) (n * (n.toLong() + 1)) / 2 else -1
        }
    }
    ```

1.  如果现在运行测试，我们应该看到它们全部通过，类似于以下图：![图 9.12：练习 9.04 的通过测试

]（img/B15216_09_12.jpg）

图 9.12：练习 9.04 的通过测试

在这个练习中，我们已经将 TDD 的概念应用到了一个非常小的示例中，以演示如何使用该技术。我们已经观察到，从骨架代码开始，我们可以创建一套测试来验证我们的条件，通过不断运行测试，我们改进了代码，直到所有测试都通过的地步。您可能已经注意到，这个概念并不直观。一些开发人员很难定义骨架代码应该有多大，才能开始创建测试用例，而其他人则出于习惯，专注于先编写代码，然后再开发测试。无论哪种情况，开发人员都需要通过该技术进行大量练习，直到熟练掌握为止。

## Activity 9.01：使用 TDD 开发

使用 TDD 方法，开发一个包含三个活动并按以下方式工作的应用程序：

+   在活动 1 中，您将显示一个数字`EditText`元素和一个按钮。单击按钮后，将`EditText`中的数字传递给活动 2。

+   Activity 2 将异步生成项目列表。项目的数量将由从活动 1 传递的数字表示。您可以使用`Timer`类，延迟 1 秒。列表中的每个项目将显示文本`Item x`。 `x`是列表中的位置。单击项目时，应将单击的项目传递给活动 3。

+   活动 3 将显示文本`You clicked y`。 `y`是用户单击的项目的文本。

应用程序将具有以下测试：

+   使用 Mockito 和`mockito-kotlin`进行单元测试，注释为`@SmallTest`

+   使用 Robolectric 和 Espresso 进行集成测试，并用`@MediumTest`进行注释

+   使用 Espresso 进行 UI 测试，并用`@LargeTest`进行注释，使用 Robot 模式

从命令行运行测试命令。

为了完成此活动，您需要采取以下步骤：

1.  您需要 Android Studio 4.1.1 或更高版本，以及 Kotlin 1.4.21 或更高版本的 Parcelize Kotlin 插件

1.  为每个活动和其 UI 创建三个活动。

1.  在`androidTest`文件夹中，为每个活动创建三个机器人：

+   Robot 1 将包含与`EditText`和按钮的交互。

+   Robot 2 将断言屏幕上的项目数量和与列表中项目的交互。

+   Robot 3 将断言`TextView`中显示的文本。

1.  创建一个仪器测试类，其中将使用前述机器人进行一个测试方法。

1.  创建一个`Application`类，其中将保存将进行单元测试的所有类的实例。

1.  创建三个表示集成测试的类，每个类对应一个活动。每个集成测试类将包含一个测试方法，用于交互和数据加载。每个集成测试将断言在活动之间传递的意图。

1.  创建一个类，用于提供 UI 所需的文本。它将引用一个`Context`对象，并包含两个方法，用于为 UI 提供文本，将返回一个空字符串。

1.  创建前述类的测试，在其中测试两种方法。

1.  实现类以使前述测试通过。

1.  创建一个类，负责在`Activity2`中加载列表，并提供一个加载的空方法。该类将引用计时器和空闲资源。在这里，您还应该创建一个数据类，用于表示`RecyclerView`的模型。

1.  为前述类创建一个单元测试。

1.  创建前述类的实现并运行单元测试，直到它们通过。

1.  在`Application`类中，实例化已进行单元测试的类，并开始在您的活动中使用它们。直到您的集成测试通过为止。

1.  提供`IntegrationTestApplication`，它将返回负责加载的类的新实现。这是为了避免使您的活动 2 的集成测试等待加载完成。

1.  提供`UiTestApplication`，它将再次减少模型的加载时间，并将空闲资源连接到 Espresso。实现剩下的工作以使 UI 测试通过。

注意

此活动的解决方案可在以下网址找到：http://packt.live/3sKj1cp

# 总结

在本章中，我们看了不同类型的测试和可用于实施这些测试的框架。我们还看了测试环境以及如何为每个环境构建结构，以及如何将代码结构化为可以单独进行单元测试的多个组件。我们分析了测试代码的不同方式，我们应该如何进行测试，以及通过查看不同的测试结果，我们如何改进我们的代码。通过 TDD，我们了解到通过从测试开始，我们可以更快地编写代码，并确保它更少出错。活动是所有这些概念汇集在一起构建一个简单的 Android 应用程序的地方，我们可以观察到，通过添加测试，开发时间增加了，但这在长期内通过消除在修改代码时出现的可能错误而得到了回报。

我们学习过的框架是一些最常见的框架，但还有其他一些建立在这些框架之上并被开发人员在他们的项目中使用的框架，包括 mockk（一种为 Kotlin 设计的模拟库，充分利用了语言的许多特性），Barista（建立在 Espresso 之上，简化了 UI 测试的语法），屏幕截图测试（对 UI 测试进行截图并进行比较以验证是否引入了错误），UIAutomator 和 monkeyrunner（执行 UI 测试而无需访问应用程序代码，但是建立在其之上），Spoon（允许在多个模拟器上并行执行 UI 测试以减少测试时间），以及 Firebase 测试实验室（允许在云中执行测试）。

将这里介绍的所有概念视为适用于软件工程世界中的两个过程的构建块：自动化和持续集成。自动化将开发人员手中的冗余和重复工作交给机器。与其让一组质量保证人员测试您的应用程序以确保满足要求，不如通过各种测试和测试用例指示机器来测试应用程序，只需一个人审查测试结果。持续集成建立在自动化的概念之上，以便在您提交代码进行其他开发人员审查时立即验证代码。具有持续集成的项目将按以下方式设置：开发人员将工作提交到 GitHub 等源代码存储库进行审查。

然后，云中的一台机器将开始执行整个项目的测试，确保没有任何问题，开发人员可以继续进行新的任务。如果测试通过，那么其他开发人员可以审查代码，当正确时，可以合并并在云中创建新的构建并分发给团队的其他成员和测试人员。在初始开发人员可以安全地进行其他工作的同时进行所有这些操作。如果在过程中出现任何失败，那么他们可以暂停新任务并解决工作中的任何问题。然后可以将持续集成过程扩展为持续交付，在准备提交到 Google Play 时可以设置类似的自动化，几乎完全由机器处理，开发人员只需进行少量参与。在接下来的章节中，您将了解如何在构建使用设备存储功能并连接到云以请求数据的更复杂的应用程序时组织代码。每个组件都可以进行单独的单元测试，并且可以应用集成测试来断言多个组件的成功集成。
