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

# ![图 9.2：Android Studio 中的测试输出在前面的示例中，我们可以看到规则将实现`TestRule`，而`TestRule`又带有`apply()`方法。然后我们创建一个新的`Statement`对象，它将执行基本语句（测试本身）并在语句之前和之后重置结果的值。现在我们可以修改测试如下：Android Studio 提供了一套很好的快捷方式和可视化工具来帮助测试。如果要为类创建新的测试或转到类的现有测试，可以使用*Ctrl* + *Shift* + *T*（Windows）或*Command* + *Shift* + *T*（Mac）快捷键。要运行测试，有多种选项：右键单击文件或包，然后选择`Run Tests in...`选项，或者如果要独立运行测试，可以转到特定的测试方法并选择顶部的绿色图标，这将执行类中的所有测试；或者，对于单个测试，可以单击`@Test`注解方法旁边的绿色图标。这将触发测试执行，显示在`Run`选项卡中，如下截图所示。测试完成后，它们将变成红色或绿色，取决于它们的成功状态：Android Studio 提供了一套很好的快捷方式和可视化工具来帮助测试。如果要为类创建新的测试或转到类的现有测试，可以使用*Ctrl* + *Shift* + *T*（Windows）或*Command* + *Shift* + *T*（Mac）快捷键。要运行测试，有多种选项：右键单击文件或包，然后选择`Run Tests in...`选项，或者如果要独立运行测试，可以转到特定的测试方法并选择顶部的绿色图标，这将执行类中的所有测试；或者，对于单个测试，可以单击`@Test`注解方法旁边的绿色图标。这将触发测试执行，显示在`Run`选项卡中，如下截图所示。测试完成后，它们将变成红色或绿色，取决于它们的成功状态：图 9.2：Android Studio 中的测试输出在测试中可以找到的另一个重要功能是调试功能。这很重要，因为您可以调试测试和被测试的方法，所以如果在修复问题时遇到问题，您可以使用此功能查看测试使用的输入以及代码如何处理输入。您可以在测试旁边的绿色图标中找到的第三个功能是`Run With Coverage`选项。这有助于开发人员确定测试覆盖的代码行以及跳过的代码行。覆盖率越高，发现崩溃和错误的机会就越大：![图 9.3：Android Studio 中的测试覆盖率](img/B15216_09_03.jpg)

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

```kt is an escape character present in Kotlin and should not be confused with a quote mark. It allows the developer to give methods any name that they want, including special characters or reserved words.

In the test, we have created a `mock` context. When the `concatenate` method was tested, we used Mockito to return a specific string when the `getString()` method was called with a particular input. This allowed us to then assert the final result.

Mockito is not limited to mocking Android Framework classes only. We can create a `SpecificStringConcatenator` class that will use `StringConcatenator` to concatenate two specific strings from `strings.xml`:

```

class SpecificStringConcatenator(private val stringConcatenator:   StringConcatenator) {

fun concatenateSpecificStrings(): String {

return stringConcatenator.concatenate(R.string.string_1,           R.string.string_2)

}

}

```kt

We can write the test for it as follows:

```

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

```kt

Here, we are mocking the previous `StringConcatenator` and instructing the mock to return a specific result. If we run the test, it will fail because Mockito is limited to mocking final classes. Here, it encounters a conflict with Kotlin that makes all classes *final* unless we specify them as *open*. Luckily, there is a configuration we can apply that solves this dilemma without making the classes under test *open*:

1.  Create a folder named `resources` in the `test` package.
2.  In `resources`, create a folder named `mockito-extensions`.
3.  In the `mockito-extensions` folder, create a file named `org.mockito.plugins.MockMaker`.
4.  Inside the file, add the following line:

```

mock-maker-inline

```kt

In situations where you have callbacks or asynchronous work and cannot use the JUnit assertions, you can use `mockito` to verify the invocation on the callback or lambdas:

```

class SpecificStringConcatenator(private val stringConcatenator:   StringConcatenator) {

fun concatenateSpecificStrings(): String {

return stringConcatenator.concatenate(R.string.string_1,           R.string.string_2)

}

fun concatenateWithCallback(callback: Callback) {

callback.onStringReady(concatenateSpecificStrings())

}

接口回调 {

当字符串准备好时，输入字符串

}

}

```kt

In the preceding example, we have added the `concatenateWithCallback` method, which will invoke the callback with the result of the `concatenateSpecificStrings` method. The test for this method would look something like this:

```

@Test

fun concatenateWithCallback() {

val expected = "expected"

Mockito.`when`(stringConcatenator.concatenate(R.string.string_1,           R.string.string_2))

.thenReturn(expected)

val callback =           Mockito.mock(SpecificStringConcatenator.Callback::class.java)

specificStringConcatenator.concatenateWithCallback(callback)

Mockito.verify(callback).onStringReady(expected)

}

```kt

Here, we are creating a mock `Callback` object, which we can then verify at the end with the expected result. Notice that we had to duplicate the setup of the `concatenateSpecificStrings` method in order to test the `concatenateWithCallback` method. You should never mock the objects you are testing; however, you can use `spy` to change their behavior. We can spy the `stringConcatenator` object in order to change the outcome of the `concatenateSpecificStrings` method:

```

@Test

fun concatenateWithCallback() {

val expected = "expected"

val spy = Mockito.spy(specificStringConcatenator)

Mockito.`when`(spy.concatenateSpecificStrings())          .thenReturn(expected)

val callback           = Mockito.mock(SpecificStringConcatenator.Callback::class.java)

specificStringConcatenator.concatenateWithCallback(callback)

Mockito.verify(callback).onStringReady(expected)

}

```kt

Mockito also relies on dependency injection to initialize class variables and has a custom build JUnit test runner. This can simplify the initialization of our variables, as follows:

```

@RunWith(MockitoJUnitRunner::class)

class SpecificStringConcatenatorTest {

@Mock

lateinit var stringConcatenator: StringConcatenator

@InjectMocks

lateinit var specificStringConcatenator: SpecificStringConcatenator

}

```kt

In the preceding example, `MockitoRunner` will inject the variables with the `@Mock` annotation with mocks. Next, it will create a new non-mocked instance of the field with the `@InjectionMocks` annotation. When this instance is created, Mockito will try to inject the mock objects that will match the signature of the constructor of that object.

## mockito-kotlin

You may have noticed, in the preceding example, that the `when` method from Mockito has escaped. This is because of a conflict with the Kotlin programming language. Mockito is built mainly for Java, and when Kotlin was created, it introduced this keyword. Conflicts like this are escaped using the ```字符。这个，连同其他一些小问题，导致在 Kotlin 中使用 Mockito 时有些不便。引入了一些库来包装 Mockito 并在使用时提供更好的体验。其中之一是`mockito-kotlin`。您可以使用以下命令将此库添加到您的模块中：

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

正如你所看到的，```kt character has disappeared, and our mock initialization for the `Context` object has been simplified. We can apply the same thing for the `SpecificConcatenatorTest` class (import statements have been removed for brevity):

```

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

```kt

## Exercise 9.01: Testing the Sum of Numbers

Using JUnit, Mockito, and `mockito-kotlin`, write a set of tests for the following class that should cover the following scenarios:

*   Assert the values for `0`, `1`, `5`, `20`, and `Int.MAX_VALUE`.
*   Assert the outcome for a negative number.
*   Fix the code and replace the sum of numbers with the formula *n*(n+1)/2*.

    Note

    Throughout this exercise, import statements are not shown. To see full code files, refer to [`packt.live/35TW8JI`](http://packt.live/35TW8JI)):

The code to test is as follows.

```

class NumberAdder {

@Throws(InvalidNumberException::class)

fun sum(n: Int, callback: (BigInteger) -> Unit) {

如果(n < 0) {

抛出 InvalidNumberException

}

var result = BigInteger.ZERO

for (i in 1..n){

result = result.plus(i.toBigInteger())

}

callback(result)

}

object InvalidNumberException : Throwable()

}

```kt

Perform the following steps to complete this exercise:

1.  Let's make sure the necessary libraries are added to the `app/build.gradle` file:

```

测试实现'junit:junit:4.13.1'

testImplementation 'org.mockito:mockito-core:3.6.0'

testImplementation 'com.nhaarman.mockitokotlin2:mockito-kotlin:2.2.0'

```kt

2.  Create a class named `NumberAdder` and copy the preceding code inside it.
3.  Move the cursor inside the newly created class and, with *Command* + *Shift* + *T* or *Ctrl* + *Shift* + *T*, create a test class called `NumberAdderParameterTest`.
4.  Create a parametrized test inside this class that will assert the outcomes for the `0`, `1`, `5`, `20`, and `Int.MAX_VALUE` values:

```

@RunWith(Parameterized::class)

class NumberAdderParameterTest(

private val input: Int,

private val expected: BigInteger

) {

伴随对象 {

@Parameterized.Parameters

@JvmStatic

获取数据(): List<Array<out Any>> = listOf(

arrayOf(0, BigInteger.ZERO),

arrayOf(1, BigInteger.ONE),

创建一个包含 5 和 15.toBigInteger()的数组，

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

```kt

5.  Create a separate test class that handles the exception thrown when there are negative numbers, named `NumberAdderErrorHandlingTest`:

```

@RunWith(MockitoJUnitRunner::class)

class NumberAdderErrorHandlingTest {

@InjectMocks

lateinit var numberAdder: NumberAdder

@Test(expected = NumberAdder.InvalidNumberException::class)

fun sum() {

val input = -1

val callback = mock<(BigInteger) -> Unit>()

numberAdder.sum（input，callback）

}

}

```kt

6.  Since *1 + 2 + ...n = n * (n + 1) / 2*, we can use the formula in the code and this would make the execution of the method run faster:

```

类 NumberAdder {

@Throws（InvalidNumberException :: class）

fun sum（n：Int，callback：（BigInteger） - > Unit）{

如果（n <0）{

抛出 InvalidNumberException

}

callback（n.toBigInteger（）。times（（n.toBigInteger（）+ 1.toBigInteger（）））。divide（2.toBigInteger（）））

}

object InvalidNumberException：Throwable（）

}

```kt

Run the tests by right-clicking the package in which the tests are located and selecting `Run all in [package_name]`. An output similar to the following will appear, signifying that the tests have passed:

![Figure 9.5: Output of Exercise 9.01](img/B15216_09_05.jpg)

Figure 9.5: Output of Exercise 9.01

By completing this exercise, we have taken the first steps into unit testing, managed to create multiple test cases for a single operation, taken the first steps into understanding Mockito, and used tests to guide us on how to refactor code without introducing any new issues.

# Integration Tests

Let's assume your project is covered by unit tests where a lot of your logic is held. You now have to add these tested classes to an activity or a fragment and require them to update your UI. How can you be certain that these classes will work well with each other? The answer to that question is through integration testing. The idea behind this type of testing is to ensure that different components within your application integrate well with each other. Some examples include the following:

*   Ensuring that your API-related components parse the data well and interact well with your storage components.
*   The storage components are capable of storing and retrieving the data correctly.
*   The UI components load and display the appropriate data.
*   The transition between different screens in your application.

To aid with integration testing, the requirements are sometimes written in the format `Given - When - Then`. These usually represent acceptance criteria for a user story. Take the following example:

```

假设我没有登录

然后我打开应用程序

当我输入我的凭据时

然后点击登录

然后我看到主屏幕

```kt

We can use these steps to approach how we can write the integration tests for the feature we are developing.

On the Android platform, integration testing can be achieved with two libraries:

*   **Robolectric**: This library gives developers the ability to test Android components as unit tests; that is, executing integration tests without an actual device or emulator.
*   **Espresso**: This library is helpful in instrumentation tests on an Android device or emulator.

We'll have a look at these libraries in detail in the next sections.

## Robolectric

Robolectric started as an open source library that was meant to give users the ability to unit test classes from the Android framework as part of their local tests instead of the instrumented tests. Recently, it has been endorsed by Google and has been integrated with AndroidX Jetpack components. One of the main benefits of this library is the simplicity of testing activities and fragments. This is a benefit when it comes to integration tests because we can use this feature to make sure that our components integrate well with each other. Some of Robolectric's features are as follows:

*   The possibility to instantiate and test the activity and fragment life cycle
*   The possibility to test view inflation
*   The possibility to provide configurations for different Android APIs, orientations, screen size, layout direction, and so on
*   The possibility to change the `Application` class, which then helps to change the modules to permit data mocks to be inserted

In order to add Robolectric, along with the AndroidX integration, we will need the following libraries:

```

testImplementation'org.robolectric：robolectric：4.3'

testImplementation'androidx.test.ext：junit：1.1.1'

```kt

The second library will bring a set of `utility` methods and classes required for testing Android components.

Let's assume we have to deliver a feature in which we display the text `Result x`, where `x` is the factorial function for a number that the user will insert in the `EditText` element. In order to achieve this, we have two classes, one that computes the factorial and another that concatenates the word `Result` with the factorial if the number is positive, or it will return the text `Error` if the number is negative. The factorial class will look something like this (throughout this example, import statements have been removed for brevity):

```

class FactorialGenerator {

@Throws（FactorialNotFoundException :: class）

fun factorial（n：Int）：BigInteger {

如果（n <0）{

抛出 FactorialNotFoundException

}

var result = BigInteger.ONE

对于（i 在 1..n）{

result = result.times（i.toBigInteger（））

}

返回结果

}

object FactorialNotFoundException：Throwable（）

}

```kt

The `TextFormatter` class will look like this:

```

class TextFormatter（

私有 val factorialGenerator：FactorialGenerator，

私有 val 上下文：上下文

）{

fun getFactorialResult（n：Int）：String {

返回尝试{

context.getString（R.string.result，factorialGenerator.factorial（n）.toString（））

} catch（e：FactorialGenerator.FactorialNotFoundException）{

context.getString（R.string.error）

}

}

}

```kt

We can combine these two components in our activity and have something similar to this:

```

类 MainActivity：AppCompatActivity（）{

private lateinit var textFormatter：TextFormatter

override fun onCreate（savedInstanceState：Bundle？）{

super.onCreate（savedInstanceState）

setContentView（R.layout.activity_main）

ttextFormatter = TextFormatter（FactorialGenerator（），

applicationContext）

findViewById <Button>（R.id.button）。setOnClickListener（{

findViewById <TextView>（R.id.text_view）。text = textFormatter.getFactorialResult（findViewById <EditText>（R.id.edit_text）。text.toString（）。toInt（））

}

}

}

```kt

We can observe three components interacting with each other in this case. We can use Robolectric to test our activity. By testing the activity that creates the components, we can also test the interaction between all three of the components. We can write a test that looks like this:

```

@RunWith（AndroidJUnit4 :: class）

类 MainActivityTest {

私有 val 上下文= getApplicationContext <Application>()

@Test

fun`在文本视图中显示阶乘结果`（）{

val scenario = launch <MainActivity>（MainActivity :: class.java）

scenario.moveToState（Lifecycle.State.RESUMED）

scenario.onActivity {activity->

activity.edit_text.setText（5.toString（））

activity.button.performClick（）

assertEquals（context.getString（R.string.result，“120”），activity.text_view.text）

}

onView（withId（R.id.button））。perform（click（））

}

```kt

In the preceding example, we can see the AndroidX support for the activity test. The `AndroidJUnit4` test runner will set up Robolectric and create the necessary configurations, while the `launch` method will return a `scenario` object, which we can then play with in order to achieve the necessary conditions for the test.

If we want to add configurations for the test, we can use the `@Config` annotation both on the class and on each of the test methods:

```

@Config（

sdk = [Build.VERSION_CODES.P]，

minSdk = Build.VERSION_CODES.KITKAT，

maxSdk = Build.VERSION_CODES.Q，

应用程序= Application :: class，

assetDir =“/ assetDir /”

）

@RunWith（AndroidJUnit4 :: class）

类 MainActivityTest

```kt

We can also specify global configurations in the `test/resources` folder in the `robolectric.properties` file, like so:

```

sdk=28

minSdk = 14

maxSdk = 29

```kt

Another important feature that has recently been added to Robolectric is support for the Espresso library. This allows developers to use the syntax from Espresso in order to interact with views and make assertions on the views. Another library that can be used in combination with Robolectric is `FragmentScenario`, which allows the possibility to test fragments. These libraries can be added in Gradle using the following:

```

testImplementation'androidx.fragment：fragment-testing：1.1.0'

testImplementation'androidx.test.espresso：espresso-core：3.2.0'

```kt

Testing fragments is similar to activities using the `scenario` setup:

```

val scenario = launchFragmentInContainer <MainFragment>（）

scenario.moveToState（Lifecycle.State.CREATED）

```kt

## Espresso

Espresso is a library designed to perform interactions and assertions in a concise way. It was initially designed to be used in instrumented tests and now it has migrated to be used with Robolectric as well. The typical usage for performing an action is as follows:

```

onView（Matcher <View>）。perform（ViewAction）

```kt

For verification, we can use the following:

```

onView（Matcher <View>）。check（ViewAssertion）

```kt

We can provide custom `ViewMatchers` if none can be found in the `ViewMatchers` class. Some of the most common ones are `withId` and `withText`. These two allow us to identify views based on their `R.id.myId` identifier or the text identifier. Ideally, the first one should be used to identify a particular view. Another interesting aspect of Espresso is the reliance on the `Hamcrest` library for matchers. This is a Java library that aims to improve testing. This allows multiple matchers to be combined if necessary. Let's say that the same ID is present in different views on your UI. You can narrow your search for a specific view using the following expression:

```

onView（allOf（withId（R.id.edit_text），withParent（withId（R.id.root））））

```kt

The `allOf` expression will evaluate all of the other operators and will pass only if all of the operators inside will pass. The preceding expressions will translate to "Find the view with `id=edit_text` that has the parent with `id=R.id.root`." Other `Hamcrest` operators may include `anyOf`, `both`, `either`, `is`, `isA`, `hasItem`, `equalTo`, `any`, `instanceOf`, `not`, `null`, and `notNull`.

`ViewActions` have a similar approach to `ViewMatchers`. We can find common ones in the `ViewActions` class. Common ones include `typeText`, `click`, `scrollTo`, `clearText`, `swipeLeft`, `swipeRight`, `swipeUp`, `swipeDown`, `closeSoftKeyboard`, `pressBack`, `pressKey`, `doubleClick`, and `longClick`. If you have custom views and certain actions are required, then you can implement your own `ViewAction` element by implementing the `ViewAction` interface.

Similar to the preceding examples, `ViewAssertions` have their own class. Typically, the `matches` method is used where you can then use `ViewMatchers` and `Hamcrest` matchers to validate the result:

```

onView（withId（R.id.text_view））。check（matches（withText（“我的文本”）））

```kt

The preceding example will verify that the view with the `text_view` ID will contain the text `My text`:

```

testImplementation'com.nhaarman.mockitokotlin2

```kt

This will click the view with the ID button.

We can now rewrite the Robolectric test and add Espresso, which will give us this (import statement not shown):

```

@RunWith（AndroidJUnit4 :: class）

类 MainActivityTest {

@Test

fun`在文本视图中显示阶乘结果`（）{

val scenario = launch <MainActivity>（MainActivity :: class.java）

scenario.moveToState（Lifecycle.State.RESUMED）

scenario.onActivity {activity->

onView（withId（R.id.edit_text））。perform（typeText（“5”））

onView（withId（R.id.button））。perform（click（））

onView（withId（R.id.text_view））

.check（matches（withText（activity.getString（R.string.result，“120”）））

}

}

}

```kt

In the preceding code sample, we can observe how, using Espresso, we input the number `5` in `EditText`, then click on the button, and then assert the text displayed in `TextView` with the help of the `onView()` method to obtain a reference to the view, and then execute actions using `perform()` or make assertions using `check()`.

Note

For the following exercise, you will need an emulator or a physical device with USB debugging enabled. You can do so by selecting `Tools` | `AVD Manager` in Android Studio. Then, you can create one with the `Create Virtual Device` option by selecting the type of emulator, clicking `Next`, and then selecting an x86 image. Any image larger than Lollipop should be alright for this exercise. Next, you can give your image a name and click `Finish`.

## Exercise 9.02: Double Integration

Develop an application that observes the following requirements:

```

假设我打开应用程序

然后我插入数字 n

当我按下“计算”按钮时

然后我应该看到文本“从 1 到 n 的数字的总和是[result]”

假设我打开应用程序

然后我插入数字-n

当我按下“计算”按钮时

然后我应该看到文本“错误：无效数字”

```kt

You should implement both unit tests and integration tests using Robolectric and Espresso, and migrate the integration tests to become instrumentation tests.

Note

Throughout this exercise, import statements are not shown. To see full code files, refer to [`packt.live/2M1MtcY`](http://packt.live/2M1MtcY)):

Implement the following steps to complete this exercise:

1.  Let's start by adding the necessary test libraries to `app/build.gradle`:

```

testImplementation'junit：junit：4.13.1'

testImplementation'org.mockito：mockito-core：3.6.0'

}

：mockito-kotlin：2.2.0'

testImplementation'org.robolectric：robolectric：4.4'

testImplementation 'androidx.test.ext：junit：1.1.2'

testImplementation 'androidx.test.espresso：espresso-core：3.3.0'

androidTestImplementation 'androidx.test.ext：junit：1.1.2'

androidTestImplementation 'androidx.test

.espresso：espresso-core：3.3.0'

androidTestImplementation 'androidx.test：rules：1.3.0'

```kt

2.  For Robolectric, we will need to add extra configurations, the first of which is to add the following line to `app/build.gradle` in the `android` closure:

```

testOptions.unitTests.includeAndroidResources = true

```kt

3.  Create a `resources` directory in the `test` package.
4.  Add the `robolectric.properties` file and add the following configuration to that file:

```

sdk = 28

```kt

5.  Create a folder named `resources` in the test package.
6.  In `resources`, create a folder named `mockito-extensions`.
7.  In the `mockito-extensions` folder, create a file named `org.mockito.plugins.MockMaker`, and inside the file, add the following line:

```

模拟制造商内联

```kt

8.  Create the `NumberAdder` class. This is similar to the one in *Exercise 9.01*:

```

导入 java.math.BigInteger

类 NumberAdder {

@Throws（InvalidNumberException :: class）

fun sum（n：Int，callback：（BigInteger） - > Unit）{

if（n <0）{

抛出 InvalidNumberException

}

回调（n.toBigInteger（）。times（（n.toLong（）+ 1）。toBigInteger（））。divide（2.toBigInteger（）））

}

object InvalidNumberException：Throwable（）

}

```kt

9.  Create the tests for `NumberAdder` in the `test` folder. First, create `NumberAdderParameterTest`:

```

@RunWith（Parameterized :: class）

类 NumberAdderParameterTest（

私有 val input：Int，

私有 val expected：BigInteger

）{

伴随对象{

@Parameterized.Parameters

@JvmStatic

fun getData（）：List <Array <out Any>> = listOf（

arrayOf（0，BigInteger.ZERO），

arrayOf（1，BigInteger.ONE），

arrayOf（5，15.toBigInteger（）），

arrayOf（20，210.toBigInteger（）），

arrayOf（Int.MAX_VALUE，BigInteger（“2305843008139952128”））

）

}

private val numberAdder = NumberAdder（）

@Test

fun sum（）{

val callback = mock <（BigInteger） - > Unit >（）

numberAdder.sum（input，callback）

verify(callback).invoke(expected)

}

}

```kt

10.  Then, create the `NumberAdderErrorHandlingTest` test:

```

@RunWith（MockitoJUnitRunner :: class）

类 NumberAdderErrorHandlingTest {

@InjectMocks

lateinit var numberAdder：NumberAdder

@Test（expected = NumberAdder.InvalidNumberException :: class）

fun sum（）{

val input = -1

val callback = mock <（BigInteger） - > Unit >（）

numberAdder.sum（input，callback）

}

}

```kt

11.  Create a class that will format the sum and concatenate it with the necessary strings:

```

类 TextFormatter（

私有 val numberAdder：NumberAdder，

私有 val context：Context

）{

fun getSumResult（n：Int，callback：（String） - > Unit）{

尝试{

numberAdder.sum（n）{

回调（

context.getString（

R.string.the_sum_of_numbers_from_1_to_is，

n，

it.toString（）

）

）

}

} catch（e：NumberAdder.InvalidNumberException）{

callback（context.getString

（R.string.error_invalid_number））

}

}

}

```kt

12.  Unit test this class for both the success and error scenarios. Start with the success scenario:

```

@RunWith（MockitoJUnitRunner :: class）

类 TextFormatterTest {

@InjectMocks

lateinit var textFormatter：TextFormatter

@Mock

lateinit var numberAdder：NumberAdder

@Mock

lateinit var context：Context

@Test

fun getSumResult_success（）{

val n = 10

val sumResult = BigInteger.TEN

val expected = "expected"

每当（numberAdder.sum（eq（n），any（）））。thenAnswer {

（it.arguments [1] as（BigInteger） - > Unit）

.invoke（sumResult）

}

每当（context.getString

（R.string.the_sum_of_numbers_from_1_to_is，n，

sumResult.toString（））。thenReturn（expected）

val callback = mock <（String） - > Unit >（）

textFormatter.getSumResult（n，callback）

verify（callback）.invoke（expected）

}

```kt

Then, create the test for the error scenario:

```

@Test

fun getSumResult_error（）{

val n = 10

val expected = "expected"

每当（numberAdder.sum（eq（n），any（）））。thenThrow（NumberAdder.InvalidNumberException）

每当（context.getString（R.string.error_invalid_number））。thenReturn（expected）

val callback = mock <（String） - > Unit >（）

textFormatter.getSumResult（n，callback）

verify（callback）.invoke（expected）

}

}

```kt

13.  Create the layout for `activity_main.xml`:

```

<?xml version =“1.0” encoding =“utf-8”?>

<LinearLayout xmlns：android =“http://schemas.android.com/apk/res/android”

android：id =“@+id/root”

android：layout_width =“match_parent”

android：layout_height =“match_parent”

android：orientation =“vertical”>

<EditText

android：id =“@+id/edit_text”

android：layout_width =“match_parent”

android：layout_height =“wrap_content”

android：inputType =“number”/>

<Button

android：id =“@+id/button”

android：layout_width =“wrap_content”

android：layout_height =“wrap_content”

android：layout_gravity =“center_horizontal”

android：text =“@string/calculate”/>

<TextView

android：id =“@+id/text_view”

android：layout_width =“wrap_content”

android：layout_height =“wrap_content”

android：layout_gravity =“center_horizontal”/>

</LinearLayout>

```kt

14.  Create the `MainActivity` class, which will contain all the other components:

```

类 MainActivity：AppCompatActivity（）{

private lateinit var textFormatter：TextFormatter

重写 fun onCreate(savedInstanceState：Bundle？）{

super.onCreate（savedInstanceState）

setContentView（R.layout.activity_main）

textFormatter = TextFormatter(NumberAdder(), applicationContext)

findViewById<Button>(R.id.button).setOnClickListener {

textFormatter.getSumResult(findViewById<EditText>              (R.id.edit_text).text.toString().toIntOrNull() ?: 0) {

findViewById<TextView>(R.id.text_view).text = it

}

}

}

}

```kt

15.  Create a test for `MainActivity` and place it in the `test` directory. It will contain two test methods, one for success and one for error:

```

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

```kt

If you run the tests by right-clicking the package in which the tests are located and select `Run all in [package_name]`, then an output similar to the following will appear:

![Figure 9.6: Result of executing the tests in the test folder for Exercise 9.02    ](img/B15216_09_06.jpg)

Figure 9.6: Result of executing the tests in the test folder for Exercise 9.02

If you execute the preceding tests, you should see an output similar to *Figure 9.6*. The Robolectric test is executed in the same way as a regular unit test; however, there is an increase in the execution time.

16.  Let's now migrate the preceding test to an instrumented integration test. In order to do this, we will copy the preceding test from the `test` package into the `androidTest` package and remove the code related to scenarios from our tests. After copying the file, we will use `ActivityTestRule`, which will launch our activity before every test is executed. We will also need to rename the class to avoid duplicates and rename the test methods because the syntax is not supported for instrumented tests:

```

@RunWith(AndroidJUnit4::class)

类 MainActivityUiTest {

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

```kt

    If you run the tests by right-clicking the package in which the tests are located and select `Run all in [package_name]`, then an output similar to the following will appear:

![Figure 9.7: Result of executing the tests in the androidTest folder for Exercise 9.02](img/B15216_09_07.jpg)

Figure 9.7: Result of executing the tests in the androidTest folder for Exercise 9.02

In *Figure 9.7*, we can see what Android Studio displays as an output for the result. If you pay attention to the emulator while the tests are executing, you can see that for each test, your activity will be opened, the input will be set in the field, and the button will be clicked. Both of our integration tests (on the workstation and the emulator) try to match the accepted criteria of the requirement. The integration tests verify the same behavior, the only difference being that one checks it locally and the other checks it on an Android device or emulator. The main benefit here is the fact that Espresso was able to bridge the gap between them, making integration tests easier to set up and execute.

# UI Tests

UI tests are instrumented tests where developers can simulate user journeys and verify the interactions between different modules of the application. They are also referred to as end-to-end tests. For small applications, you can have one test suite, but for larger applications, you should split your test suites to cover particular user journeys (logging in, creating an account, setting up flows, and so on). Because they are executed on the device, you will need to write them in the `androidTest` package, which means they will run with the **Instrumentation** framework. Instrumentation works as follows:

*   The app is built and installed on the device.
*   A testing app will also be installed on the device that will monitor your app.
*   The testing app will execute the tests on your app and record the results.

One of the drawbacks of this is the fact that the tests will share persisted data, so if a test stores data on the device, then the second test can have access to that data, which means that there is a risk of failure. Another drawback is that if a test comes across a crash, this will stop the entire testing because the application under test is stopped. These issues were solved in the Jetpack updates with the introduction of the `app/build.gradle` file:

```

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

```kt

You can execute the orchestrator test on a connected device using Gradle's `connectedCheck` command, either from `Terminal` or from the list of Gradle commands.

In the configuration, you will notice the following line: `testInstrumentationRunner`. This allows us to create a custom configuration for the test, which gives us the opportunity to inject mock data into the modules:

```

testInstrumentationRunner "com.android.CustomTestRunner"

```kt

`CustomTestRunner` looks like this (import statements not shown in following code snippets):

```

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

```kt

The test classes themselves can be written by applying the JUnit4 syntax with the help of the `androidx.test.ext.junit.runners.AndroidJUnit4` test runner:

```

@RunWith(AndroidJUnit4::class)

class MainActivityUiTest {

}

```kt

Another important feature that comes from the AndroidX testing support is the activity rule. When this rule is used with the default constructor, the activity will be launched before each test and will be ready for interactions and assertions:

```

@JvmField

@Rule

var activityRule: ActivityTestRule<MainActivity>       = ActivityTestRule(MainActivity::class.java)

```kt

You can also use the rule to avoid starting the activity and customize the intent used to start it in your test:

```

@ JvmField

@Rule

var activityRule: ActivityTestRule<MainActivity> =       ActivityTestRule(MainActivity::class.java, false ,false)

@Test

fun myTestMethod() {

val myIntent = Intent()

activityRule.launchActivity(myIntent)

}

```kt

The `@Test` methods themselves run in a dedicated test thread, which is why a library such as Espresso is helpful. Espresso will automatically move every interaction with a view on the UI thread. Espresso can be used for UI tests in a similar way as it is used with Robolectric tests:

```

@Test

fun myTest() {

onView(withId(R.id.edit_text)).perform(replaceText("5"))

onView(withId(R.id.button)).perform(click())

onView(withId(R.id.text_view))          .check(matches(withText("my test")))

}

```kt

Typically, in UI tests, you will find interactions and assertions that may get repetitive. In order to avoid duplicating multiple scenarios in your code, you can apply a pattern called `Robot` class in which the interactions and assertions can be grouped into specific methods. Your test code will use the robots and assert them. A typical robot will look something like this:

```

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

```kt

The test will look like this:

```

@Test

fun myTest() {

MyScreenRobot()

.setText()

.pressButton()

.assertText()

}

```kt

Because apps can be multithreaded and sometimes it takes a while to load data from various sources (internet, files, local storage, and so on), the UI tests will have to know when the UI is available for interactions. One way to implement this is through the usage of idling resources. These are objects that can be registered to Espresso before the test and injected into your application's components where multithreaded work is done. The apps will mark them as non-idle when the work is in progress and idle when the work is done. It is at this point where Espresso will then start executing the test. One of the most commonly used ones is `CountingIdlingResource`. This specific implementation uses a counter that should be incremented when you want Espresso to wait for your code to complete its execution and decremented when you want to let Espresso verify your code. When the counter reaches `0`, Espresso will resume testing. An example of a component with an idling resource looks something like this:

```

class MyHeavyliftingComponent(private val   countingIdlingResource:CountingIdlingResource) {

fun doHeavyWork() {

countingIdlingResource.increment()

// do work

countingIdlingResource.decrement()

}

}

```kt

The `Application` class can be used to inject the idling resource, like this:

```

class MyApplication : Application(){

val countingIdlingResource =计数 IdlingResource（“我的繁重工作”）

val myHeavyliftingComponent = MyHeavyliftingComponent（countingIdlingResource）}

}

```kt

Then, in the test, we can access the `Application` class and register the resource to Espresso:

```

使用 AndroidJUnit4 :: class 运行

class MyTest {

@Before

fun setUp（）{

val myApplication = getApplicationContext <MyApplication>（）

IdlingRegistry.getInstance（）.register（myApplication.countingIdlingResource）

}

}

```kt

Espresso comes with a set of extensions that can be used to assert different Android components. One extension is intents testing. This is useful when you want to test an activity in isolation (more appropriate for integration tests). In order to use this, you need to add the library to Gradle:

```

androidTestImplementation'androidx.test.espresso：espresso-intents：3.3.0'

```kt

After you add the library, you need to use `IntentsTestRule` in order to set up the necessary intent monitoring. This rule is a subclass of `ActivityTestRule`:

```

@JvmField

@Rule

var intentsRule：IntentsTestRule <MainActivity> = IntentsTestRule（MainActivity :: class.java）

```kt

In order to assert the values of the intent, you need to trigger the appropriate action and then use the `intended` method:

```

onView（withId（R.id.button））。执行（click（））

intended（allOf（

hasComponent（hasShortClassName（“.MainActivity”）），

hasExtra（MainActivity.MY_EXTRA，“myExtraValue”））

```kt

The `intended` method works in a similar way to the `onView` method. It requires a matcher that can be combined with a `Hamcrest` matcher. The intent-related matchers can be found in the `IntentMatchers` class. This class contains methods to assert different methods of the `Intent` class: extras, data, components, bundles, and so on.

Another important extension library comes to the aid of `RecyclerView`. The `onData` method from Espresso is only capable of testing `AdapterViews` such as `ListView` and isn't capable of asserting `RecyclerView`. In order to use the extension, you need to add the following library to your project:

```

androidTestImplementation'com.android.support.test.espresso：espresso-contrib：3.0.2'

```kt

This library provides a `RecyclerViewActions` class, which contains a set of methods that allow you to perform actions on items inside `RecyclerView`:

```

onView（withId（R.id.recycler_view））。执行（RecyclerViewActions.actionOnItemAtPosition（0，click（）））

```kt

The preceding statement will click the item at position `0`:

```

onView（withId（R.id.recycler_view））。执行（RecyclerViewActions.scrollToPosition <RecyclerView.ViewHolder>（10））

```kt

This will scroll to the tenth item in the list:

```

onView（withText（“myText”））。check（matches（isDisplayed（）））

```kt

The preceding code will check whether a view with the `myText` text is displayed, which will also apply to `RecyclerView` items.

## Exercise 9.03: Random Waiting Times

Write an application that will have two screens. The first screen will have a button. When the user presses the button, it will wait a random time between 1 and 5 seconds and then launch the second screen, which will display the text `Opened after x seconds`, where `x` is the number of seconds that passed. Write a UI test that will cover this scenario with the following features adjusted for the test:

*   The `random` function will return the value `1` when the test is run.
*   `CountingIdlingResource` will be used to indicate when the timer has stopped.

Note

Throughout this exercise, import statements are not shown. To see full code files, refer to [`packt.live/38V7krh`](http://packt.live/38V7krh)):

Take the following steps to complete this exercise:

1.  Add the following libraries to `app/build.gradle`:

```

implementation'androidx.test.espresso：espresso-core：3.3.0'

testImplementation'junit：junit：4.13.1'

androidTestImplementation'androidx.test.ext：junit：1.1.2'

androidTestImplementation'androidx.test：rules：1.3.0'

```kt

2.  Then, start with a `Randomizer` class:

```

class Randomizer（private val random：Random）{

fun getTimeToWait（）：Int {

return random.nextInt（5）+ 1

}

}

```kt

3.  Next, create a `Synchronizer` class, which will use `Randomizer` and `Timer` to wait for the random time interval. It will also use `CountingIdlingResource` to mark the start of the task and the end of the task:

```

class Synchronizer（

private val randomizer：Randomizer，

private val timer：计时器，

private val countingIdlingResource：CountingIdlingResource

）{

fun executeAfterDelay（callback：（Int） - > Unit）{

val timeToWait = randomizer.getTimeToWait（）

countingIdlingResource.increment（）

timer.schedule（CallbackTask（callback，timeToWait），timeToWait * 1000L）

}

内部类 CallbackTask（

private val 回调：（Int） - > Unit，

private val time：Int

）：TimerTask（）{

override fun run（）{

回调（时间）

countingIdlingResource.decrement（）

}

}

}

```kt

4.  Now create an `Application` class that will be responsible for creating all the instances of the preceding classes:

```

class MyApplication：Application（）{

val countingIdlingResource =计数 IdlingResource（“计时器资源”）

val randomizer = Randomizer（Random（））

val 同步器=同步器（随机器，计时器（），计数 IdlingResource）

}

```kt

5.  Add the `MyApplication` class to `AndroidManifest` in the `application` tag with the `android:name` attribute.
6.  Create an `activity_1` layout file, which will contain a parent layout and a button:

```

<?xml version =“1.0” encoding =“utf-8”?>

<LinearLayout xmlns：android="http://schemas.android.com/apk/res/android"

android：layout_width =“match_parent”

android：layout_height =“match_parent”

android：orientation =“vertical”>

<Button

android：id =“@+id/activity_1_button”

android：layout_width =“wrap_content”

android：layout_height =“wrap_content”

android：layout_gravity =“center”

android：text =“@string / press_me”/>

</LinearLayout>

```kt

7.  Create an `activity_2` layout file, which will contain a parent layout and `TextView`:

```

<?xml version =“1.0” encoding =“utf-8”?>

<LinearLayout xmlns：android="http://schemas.android.com/apk/res/android"

android：layout_width =“match_parent”

android：layout_height =“match_parent”

android：orientation =“vertical”>

<TextView

android：id =“@+id/activity_2_text_view”

android：layout_width =“wrap_content”

android：layout_height =“wrap_content”

android：layout_gravity =“center”/>

</LinearLayout>

```kt

8.  Create the `Activity1` class, which will implement the logic for the button click:

```

class Activity1：AppCompatActivity（）{

override fun onCreate（savedInstanceState：Bundle？）{

super.onCreate（savedInstanceState）

setContentView（R.layout.activity_1）

findViewById <Button>（R.id.activity_1_button）

.setOnClickListener {

（应用程序作为 MyApplication）.synchronizer

.executeAfterDelay {

startActivity（Activity2.newIntent（this，it））

}

}

}

}

```kt

9.  Create the `Activity2` class, which will display the received data through the intent:

```

class Activity2：AppCompatActivity（）{

伴随对象{

private const val EXTRA_SECONDS =“extra_seconds”

fun newIntent（context：Context，seconds：Int）=

Intent（context，Activity2 :: class.java）。putExtra（

EXTRA_SECONDS，秒

）

}

override fun onCreate（savedInstanceState：Bundle？）{

super.onCreate（savedInstanceState）

setContentView（R.layout.activity_2）

findViewById <TextView>（R.id.activity_2_text_view）。text =

getString（R.string.opened_after_x_seconds，intent.getIntExtra（EXTRA_SECONDS，0））

}

}

```kt

10.  Create a `FlowTest` class in the `androidTest` directory, which will register `IdlingResource` from the `MyApplication` object and will assert the outcome of the click:

```

@RunWith（AndroidJUnit4 :: class）

@LargeTest

class FlowTest {

@JvmField

@Rule

var activityRule：ActivityTestRule <Activity1> = ActivityTestRule（Activity1 :: class.java）

private val myApplication = getApplicationContext<MyApplication>()

@Before

fun setUp（）{

IdlingRegistry.getInstance（）。register（myApplication.countingIdlingResource）

}

@Test

fun verifyFlow() {

onView（withId（R.id.activity_1_button））。perform（click（））

onView（withId（R.id.activity_2_text_view））.check（matches（withText（myApplication.getString（R.string.opened_after_x_seconds，1））））

}

}

```kt

11.  Run the test multiple times and check the test results. Notice that the test will have a 20% chance of success, but it will wait until the button is clicked. This means that the idling resource is working. Another thing to observe is that there is an element of randomness here.
12.  Tests don't like randomness, so we need to eliminate it by making the `Randomizer` class open and create a sub-class in the `androidTest` directory. We can do the same for the `MyApplication` class and provide a different randomizer called `TestRandomizer`:

```

类 TestRandomizer（随机：随机）：随机器（随机）

覆盖 fun getTimeToWait（）：Int {

return 1

}

}

```kt

13.  Now, modify the `MyApplication` class in a way in which we can override the randomizer from a subclass:

```

open class MyApplication：Application（）{

val countingIdlingResource = CountingIdlingResource（“Timer resource”）

lateinit var synchronizer：Synchronizer

覆盖 fun onCreate（）{

super.onCreate（）

synchronizer = Synchronizer（createRandomizer（），Timer（），countingIdlingResource）

}

open fun createRandomizer（）= Randomizer（Random（））

}

```kt

14.  In the `androidTest` directory, create `TestMyApplication`, which will extend `MyApplication` and override the `createRandomizer` method:

```

类 TestMyApplication：MyApplication（）{

覆盖 fun createRandomizer（）：Randomizer {

返回 TestRandomizer（Random（））

}

}

```kt

15.  Finally, create an instrumentation test runner that will use this new `Application` class inside the test:

```

class MyApplicationTestRunner：AndroidJUnitRunner（）{

@Throws（Exception :: class）

覆盖 fun newApplication（

cl：ClassLoader？

className：String？

上下文：上下文？

）：Application？

return super.newApplication（cl，TestMyApplication :: class.java.name，context）

}

}

```kt

16.  Add the new test runner to the Gradle configuration:

```

android {

...

defaultConfig {

...

testInstrumentationRunner "com.android.testable.myapplication.MyApplicationTestRunner"

}

}

```kt

By running the test now, everything should pass similar to *Figure 9.8*:

![Figure 9.8: Output of Exercise 9.03](img/B15216_09_08.jpg)

Figure 9.8: Output of Exercise 9.03

This type of exercise shows how to avoid randomness in a test and provide concrete and repeatable inputs to make our tests reliable. Similar approaches are taken with dependency injection frameworks where entire modules can be replaced in the test suite in order to ensure the test's reliability. One of the most common things to be replaced is API communication. Another issue this approach solves is the decrease in waiting time. If this type of scenario were to have been repeated across your tests, then the execution time of them would have increased as a result of this.

# Test-Driven Development

Let's assume that you are tasked with building an activity that displays a calculator with the add, subtract, multiply, and divide options. You must also write tests for your implementation. Typically, you would build your UI and your activity and a separate `Calculator` class. Then, you would write the unit tests for your `Calculator` class and then your activity class.

Under the `Calculator` test. Here, you would also need to create the necessary methods in the `Calculator` class to avoid compile-time errors.

If you run your tests in this phase, they would fail. This would force you to implement your code until the tests pass. Once your `Calculator` tests pass, you can connect your calculator to your UI until your UI tests pass. While this seems like a counter-intuitive approach, it solves two issues once the process is mastered:

*   Less time will be spent writing code because you will ensure that your code is testable, and you need to write only the amount of code necessary for the test to pass.
*   Fewer bugs will be introduced because developers will be able to analyze different outcomes.

Have a look at the following diagram, which shows the TDD cycle:

![Figure 9.9: TDD cycle](img/B15216_09_09.jpg)

Figure 9.9: TDD cycle

In the preceding figure, we can see the development cycle in a TDD process. You should start from a point where your tests are failing. Implement changes in order for the tests to pass. When you update or add new features, you can repeat the process.

Going back to our factorial examples, we started with a factorial function that didn't cover all our scenarios and had to keep updating the function every time a new test was added. TDD is built with that idea in mind. You start with an empty function. You start defining your testing scenarios: What are the conditions for success? What's the minimum? What's the maximum? Are there any exceptions from the main rule? What are they? These questions can help developers define their test cases. Then, these cases can be written. Let's now see how this can be done practically through the next exercise.

## Exercise 9.04: Using TDD to Calculate the Sum of Numbers

Write a function that will take as input the integer *n* and will return the sum of numbers from 1 to *n*. The function should be written with a TDD approach, and the following criteria should be satisfied:

*   For *n<=0*, the function will return the value `-1`.
*   The function should be able to return the correct value for `Int.MAX_VALUE`.
*   The function should be quick, even for `Int.MAX_VALUE`.

Note

Throughout this exercise, import statements are not shown. To see full code files, refer to [`packt.live/3a0jJd9`](http://packt.live/3a0jJd9)):

Perform the following steps to complete this exercise:

1.  Make sure that the following library is added to `app/build.gradle`:

```

testImplementation 'junit:junit:4.13.1'

```kt

2.  Create an `Adder` class with the `sum` method, which will return `0`, to satisfy the compiler:

```

类 Adder {

fun sum（n：Int）：Int = 0

}

```kt

3.  Create an `AdderTest` class in the test directory and define our test cases. We will have the following test cases: *n=1*, *n=2*, *n=0*, *n=-1*, *n=10*, *n=20*, and *n=Int.MAX_VALUE*. We can split the successful scenarios into one method and the unsuccessful ones into a separate method:

```

类 AdderTest {

private val adder = Adder（）

@Test

fun sumSuccess（）{

assertEquals（1，adder.sum（1））

assertEquals（3，adder.sum（2））

assertEquals（55，adder.sum（10））

assertEquals（210，adder.sum（20））

assertEquals（2305843008139952128L，adder.sum（Int.MAX_VALUE））

}

@Test

fun sumError（）{

assertEquals（-1，adder.sum（0））

assertEquals（-1，adder.sum（-1））

}

}

```kt

4.  If we run the tests for the `AdderTest` class, we will see an output similar to the following figure, meaning that all our tests failed:![Figure 9.10: Initial test status for Exercise 9.04    ](img/B15216_09_10.jpg)

Figure 9.10: Initial test status for Exercise 9.04

5.  Let's first address the success scenarios by implementing the sum in a loop from 1 to *n*:

```

类添加器{

fun sum（n：Int）：Long {

var result = 0L

for（i in 1..n）{

result += i

}

return result

}

}

```kt

6.  If we run the tests now, you will see that one will pass and the other will fail, similar to the following figure:![Figure 9.11: Test status after resolving the success scenario for Exercise 9.04    ](img/B15216_09_11.jpg)

Figure 9.11: Test status after resolving the success scenario for Exercise 9.04

7.  If we take a look at the time it took to execute the successful test, it seems a bit long. This can add up when thousands of unit tests are present in one project. We can now optimize our code to deal with the issue by applying the *n(n+1)/2* formula:

```

类 Adder {

fun sum（n：Int）：Long {

返回（n *（n.toLong（）+ 1））/ 2

}

}

```kt

Running the tests now will drastically reduce the speed to a few milliseconds.

8.  Now, let's focus on solving our failure scenarios. We can do this by adding a condition for when *n* is smaller than or equal to `0`:

```

类 Adder {

fun sum（n：Int）：Long {

如果（n> 0）（n *（n.toLong（）+ 1））/ 2 else -1```

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
