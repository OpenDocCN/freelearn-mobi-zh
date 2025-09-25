# 第七章：在 Kotlin 中处理文件操作

本章将涵盖以下内容：

+   使用 InputReader 从文件中读取

+   使用 InputReader 从文件中读取所有行

+   使用 InputReader 逐行读取

+   使用 BufferedReader 从文件中读取

+   使用 BufferedReader 从文件中读取所有行

+   使用 BufferedReader 逐行读取

+   通过网络读取字符串和 JSON

# 简介

Kotlin I/O 用于输入和输出处理。Kotlin 提供了`kotlin.io` API 来处理文件和流。`kotlin.io`中使用的某些函数是`java.io`类的扩展。总的来说，使用`kotlin.io`从文件和流中读取和写入非常简单。

# 使用 InputReader 从文件中读取

`Kotlin.io`提供了一个干净、简洁的 API 来读取和写入文件。实现这一目标的一种方法是通过使用`InputReader`。我们将在本食谱中看到如何做到这一点。

# 准备中

您需要安装一个首选的开发环境，该环境可以编译和运行 Kotlin。您也可以使用命令行来完成此目的，为此您需要安装 Kotlin 编译器和 JDK。您还可以使用 IntelliJ IDEA 作为开发环境。

# 如何做到这一点…

读取文件的方法有很多，但了解其背后的动机非常重要，这样我们才能为我们的目的选择正确的方法：

1.  首先，我们将尝试获取文件的`InputStream`，并使用读取器来读取内容：

```kt
import java.io.File
import java.io.InputStream

fun main(args: Array<String>) {
    val inputStream: InputStream = File("lorem.txt").inputStream()
    val inputString = inputStream.reader().use { it.readText() }
    println(inputString)
}
```

1.  在前面的代码块中，`lorem.txt`只是一个我们想要读取的文件。该文件位于我们的代码源文件相同的文件夹中。如果我们需要读取位于不同文件夹中的文件，它看起来类似于以下内容：

```kt
File("/path/to/file/lorem.txt")
```

1.  这段代码只是将文件中的所有文本打印到控制台上。

1.  读取文件内容的另一种方法是直接创建文件的读取器，就像我们在以下代码中所做的那样：

```kt
import java.io.File

fun main(args: Array<String>) {
  val inputString = File("lorem.txt").reader().use { it.readText() }
  println(inputString)
}
```

1.  上述两个代码块输出的结果将简单地是文件中的文本，正如它本身那样。在我们的例子中，如下所示：

```kt
Lorem ipsum dolor sit amet, consectetur adipiscing elit.
Nunc consequat eleifend mauris, eget congue ipsum consectetur id.
Proin hendrerit felis metus, vitae suscipit mi tempus facilisis.
Proin ut leo tellus. Donec nec lacus vel ante venenatis porttitor et sit amet purus.
Sed tincidunt turpis ac metus pharetra dapibus.
Integer sed auctor tellus. Morbi a metus luctus, viverra enim vel, imperdiet est.
Curabitur purus massa, hendrerit id ligula et, finibus elementum purus.
In ut consectetur lacus.
Suspendisse non mauris eget dolor faucibus pharetra quis sed turpis.
Vivamus eget lectus vel mi faucibus dignissim.
Class aptent taciti sociosqu ad litora torquent per conubia nostra, per inceptos himenaeos.
Ut vitae velit non nunc consectetur imperdiet.
Nunc feugiat diam tellus, in pellentesque nisl dapibus quis.
Proin luctus sapien ac ante tempor, eget mollis odio aliquet.
```

1.  现在，如果我们想逐行读取文件，因为我们要对每一行进行处理，那该怎么办呢？在这种情况下，我们使用`useLines()`方法来代替`use()`方法。

1.  查看以下示例，其中我们从文件中获取输入流，并使用`useLines()`方法逐行读取：

```kt
import java.io.File
import java.io.InputStream

fun main(args: Array<String>) {
    val listOfLines = mutableListOf<String>()
    val inputStream: InputStream = File("lorem.txt").inputStream()
    inputStream.reader().useLines { lines -> lines.forEach { listOfLines.add(it)} }
    listOfLines.forEach{println("$ " + it)}
}
```

1.  或者，如果我们希望直接在文件上使用读取器，我们这样做：

```kt
import java.io.File

fun main(args: Array<String>) {
    val listOfLines = mutableListOf<String>()

    File("lorem.txt").reader().useLines { lines -> lines.forEach { listOfLines.add(it)} }
    listOfLines.forEach{println("$ " + it)}
}
```

在这种情况下，输出将是以下内容：

```kt
$ Lorem ipsum dolor sit amet, consectetur adipiscing elit.
$ Nunc consequat eleifend mauris, eget congue ipsum consectetur id.
$ Proin hendrerit felis metus, vitae suscipit mi tempus facilisis.
$ Proin ut leo tellus. Donec nec lacus vel ante venenatis porttitor et sit amet purus.
$ Sed tincidunt turpis ac metus pharetra dapibus.
$ Integer sed auctor tellus. Morbi a metus luctus, viverra enim vel, imperdiet est.
$ Curabitur purus massa, hendrerit id ligula et, finibus elementum purus.
$ In ut consectetur lacus.
$ Suspendisse non mauris eget dolor faucibus pharetra quis sed turpis.
$ Vivamus eget lectus vel mi faucibus dignissim.
$ Class aptent taciti sociosqu ad litora torquent per conubia nostra, per inceptos himenaeos.
$ Ut vitae velit non nunc consectetur imperdiet.
$ Nunc feugiat diam tellus, in pellentesque nisl dapibus quis.
$ Proin luctus sapien ac ante tempor, eget mollis odio aliquet.
```

# 它是如何工作的…

你注意到我们使用了 `use()` 和 `useLines()` 方法来读取文件吗？对 `Closeable.use()` 函数的调用将在 lambda 表达式执行结束时自动关闭输入。现在，我们当然可以使用 `Reader.readText()`，但这不会在执行后关闭流。除了 `use()` 之外，还有其他方法，如 `Reader.readText()` 等，可以用来读取流或文件的内容。使用任何方法的决策取决于我们是否希望在执行后自动关闭流，或者我们希望处理关闭资源，以及我们是否希望从流中读取或直接从文件中读取。

# 还有更多...

`BufferedReader` 一次从输入流中读取几个字符并将它们存储在缓冲区中。这就是为什么它被称为 `BufferedReader`。另一方面，`InputReader` 只读取输入流中的一个字符，其余字符仍然留在流中。在这种情况下没有缓冲区。这就是为什么 `BufferedReader` 快速，因为它维护一个缓冲区，从缓冲区中检索数据总是比从磁盘检索数据更快。

# 使用 InputReader 读取文件中的所有行

我们可以使用 `InputReader` 一次性读取文件中的所有行。在本教程中，我们将学习如何做到这一点。

# 准备工作

您需要安装一个首选的开发环境，该环境可以编译和运行 Kotlin。您也可以使用命令行来完成此目的，这需要安装 Kotlin 编译器和 JDK。您还可以使用 IntelliJ IDEA 作为开发环境。

# 如何操作...

让我们按照以下步骤来了解如何使用 `InputReader` 类读取文件：

1.  读取文件有两种方式，其中一种是将输入流附加到文件上。让我们看看如何操作，并使用 `InputReader` 来读取其内容：

```kt
import java.io.File
import java.io.InputStream

fun main(args: Array<String>) {
    val inputStream: InputStream = File("example2.txt").inputStream()
    val inputString = inputStream.reader().use { it.readText() }
    println(inputString)
}
```

1.  另一种方式是不获取流，直接读取文件的内容，如下例所示：

```kt
import java.io.File
fun main(args: Array<String>) {
    val inputString = File("example2.txt").reader().use { it.readText() }
    println(inputString)
}
```

在这种情况下，输出仅仅是文件的内容，没有变化：

```kt
A panoramic view of Lower Manhattan as seen at dusk from Jersey City, New Jersey, in November 2014\. Manhattan is the most densely populated borough of New York City. It is the city's economic and administrative center, and a major global cultural, financial, media, and entertainment center.
The second paragraph of this file is small.
```

我们使用 `use()` 方法是因为它在执行后关闭了流。

# 它是如何工作的...

将 `inputStream` 附加到文件上会返回一个字节数据流。我们可以使用流返回的读取器，或者我们可以直接在文件上使用读取器。`inputStream` 的 `read()` 方法读取流中的下一个字节。`readText()` 方法使用 UTF-8 或指定的字符集返回文件的整个内容作为字符串。

这个 `readText()` 方法不推荐用于大文件。它有一个内部限制，即文件大小为 2 GB。在处理大文件的情况下，我们从流中逐字节读取。

# 使用 InputReader 逐行读取

有时候我们需要逐行读取文件内容并进行处理。这可以通过使用 `InputReader` 逐行读取文件来实现。让我们看看如何操作。

# 准备工作

您需要安装一个首选的开发环境，该环境可以编译和运行 Kotlin。您也可以使用命令行来完成此目的，为此您需要安装 Kotlin 编译器和 JDK。您还可以使用 IntelliJ IDEA 作为开发环境。

# 如何做…

在以下步骤中，我们将学习如何使用`InputReader`类逐行读取文本：

1.  让我们从将`InputStream`附加到文件并逐行读取内容开始，如下所示：

```kt
import java.io.File
import java.io.InputStream

fun main(args: Array<String>) {
    val listOfLines = mutableListOf<String>()
    val inputStream: InputStream =  File("example2.txt").inputStream()
    inputStream.reader().useLines { lines -> lines.forEach { listOfLines.add(it)} }
    listOfLines.forEach{println("* " + it)}
}
```

1.  在这种情况下，每一行都附加了`*`。以下是输出结果：

```kt
* A panoramic view of Lower Manhattan as seen at dusk from Jersey City, New Jersey, in November 2014\. Manhattan is the most densely populated borough of New York City. It is the city's economic and administrative center, and a major global cultural, financial, media, and entertainment center.
* The second paragraph of this file is small.
```

1.  我们可以直接将读取器附加到文件上，并逐行读取。以下代码正是这样做的：

```kt
import java.io.File

fun main(args: Array<String>) {
    val listOfLines = mutableListOf<String>()

    File("example2.txt").reader().useLines { lines -> lines.forEach { listOfLines.add(it)} }
    listOfLines.forEach{println("* " + it)}
}
```

# 它是如何工作的…

`useLines()`方法为我们提供了一个遍历文件或流中所有行的可迭代对象，并对每一行（字符串）进行一些操作。我们将所有修改后的字符串添加到一个列表中，并打印出来。

# 使用`BufferedReader`读取文件

`BufferedReader`在读取时将一些字符存储在缓冲区中。这使得读取更快，因此更高效。在本例中，我们将了解如何使用`BufferedReader`读取文件内容。

# 准备中

您需要安装一个首选的开发环境，该环境可以编译和运行 Kotlin。您也可以使用命令行来完成此目的，为此您需要安装 Kotlin 编译器和 JDK。您还可以使用 IntelliJ IDEA 作为开发环境。

# 如何做…

按照以下步骤学习更多关于`BufferedReader`类的工作原理：

1.  我们可以直接将`BufferedReader`附加到文件上，并读取整个文件的内容，如下面的代码所示：

```kt
import java.io.File
import java.io.InputStream

fun main(args: Array<String>) {
    val inputString = File("lorem.txt").bufferedReader().use { it.readText() }
    println(inputString)
}
```

1.  我们也可以逐行处理所需的内容，以便能够单独处理每一行。在以下代码中，我们逐行读取并添加一个字符到字符串的开始处和字符后的字符串长度：

```kt
import java.io.File
import java.io.InputStream

fun main(args: Array<String>) {
    val listOfLines = mutableListOf<String>()
    File("lorem.txt").bufferedReader().useLines { 
        lines -> lines.forEach { 
            var x = "> (" + it.length + ") " + it;
            listOfLines.add(x)
        } 
    }
    listOfLines.forEach{println(it)}
}
```

1.  在前面的代码块中，我们是直接将读取器附加到文件上的。然而，在某些情况下，我们需要获取一个数据流。在这种情况下，我们可以从要读取的文件中获取一个输入流，然后将其附加到`BufferedReader`上。

1.  在以下代码中，我们尝试使用`BufferedReader`逐行从文件输入流中读取：

```kt
import java.io.File
import java.io.InputStream

fun main(args: Array<String>) {
    val listOfLines = mutableListOf<String>()
    val inputStream: InputStream = File("lorem.txt").inputStream()
    inputStream.bufferedReader().useLines { 
        lines -> lines.forEach { 
            var x = "> (" + it.length + ") " + it;
            listOfLines.add(x)
        } 
    }
    listOfLines.forEach{println(it)}
}
```

1.  当我们尝试一次性读取文件的全部内容时，以下是输出结果：

```kt
Lorem ipsum dolor sit amet, consectetur adipiscing elit.
Nunc consequat eleifend mauris, eget congue ipsum consectetur id.
Proin hendrerit felis metus, vitae suscipit mi tempus facilisis.
Proin ut leo tellus. Donec nec lacus vel ante venenatis porttitor et sit amet purus.
Sed tincidunt turpis ac metus pharetra dapibus.
Integer sed auctor tellus. Morbi a metus luctus, viverra enim vel, imperdiet est.
Curabitur purus massa, hendrerit id ligula et, finibus elementum purus.
In ut consectetur lacus.
Suspendisse non mauris eget dolor faucibus pharetra quis sed turpis.
Vivamus eget lectus vel mi faucibus dignissim.
Class aptent taciti sociosqu ad litora torquent per conubia nostra, per inceptos himenaeos.
Ut vitae velit non nunc consectetur imperdiet.
Nunc feugiat diam tellus, in pellentesque nisl dapibus quis.
Proin luctus sapien ac ante tempor, eget mollis odio aliquet.
```

1.  输出类似于文件，忽略`charset`。如果需要，我们也可以指定所需的`charset`，如下面的代码所示：

```kt
bufferedReader(charset).use { it.readText() }
```

1.  当我们使用上述任一示例逐行读取时，我们得到以下输出：

```kt
> (56) Lorem ipsum dolor sit amet, consectetur adipiscing elit.
> (65) Nunc consequat eleifend mauris, eget congue ipsum consectetur id.
> (64) Proin hendrerit felis metus, vitae suscipit mi tempus facilisis.
> (84) Proin ut leo tellus. Donec nec lacus vel ante venenatis porttitor et sit amet purus.
> (47) Sed tincidunt turpis ac metus pharetra dapibus.
> (81) Integer sed auctor tellus. Morbi a metus luctus, viverra enim vel, imperdiet est.
> (71) Curabitur purus massa, hendrerit id ligula et, finibus elementum purus.
> (24) In ut consectetur lacus.
> (68) Suspendisse non mauris eget dolor faucibus pharetra quis sed turpis.
> (46) Vivamus eget lectus vel mi faucibus dignissim.
> (91) Class aptent taciti sociosqu ad litora torquent per conubia nostra, per inceptos himenaeos.
> (46) Ut vitae velit non nunc consectetur imperdiet.
> (60) Nunc feugiat diam tellus, in pellentesque nisl dapibus quis.
> (61) Proin luctus sapien ac ante tempor, eget mollis odio aliquet.
```

# 它是如何工作的…

使用`InputStream`可以帮助我们获取要读取的文件的流。我们也可以直接从文件中读取。在两种情况下，`BufferedReader`都会预先保存一些数据到其缓冲区中，以便更快地操作，这比使用`InputReader`时提高了整体读取操作的效率。

我们使用 `use()` 和/或 `useLines()` 方法代替 `Reader.readText()` 等方法，这样它会在执行结束时自动关闭输入流，这是一种更干净、更负责任地处理文件 I/O 的方法。然而，如果需要，当想要自己处理流的打开和关闭时，可以使用 `Reader.readText()` 等方法。

# 使用 BufferedReader 逐行读取文件中的所有行

`BufferedReader` 可以用来读取文件或输入流的内容。它预先保存了一些读取的内容，因此读取操作更快。在本菜谱中，我们将学习如何使用 `BufferedReader` 一次性读取文件的所有内容。

# 准备工作

您需要安装一个首选的开发环境，该环境可以编译和运行 Kotlin。您也可以使用命令行来完成此目的，这需要安装 Kotlin 编译器和 JDK。您还可以使用 IntelliJ IDEA 作为开发环境。

# 如何操作...

在以下步骤中，我们将学习如何使用 `BufferedReader` 读取文件的所有行：

1.  让我们从获取文件的 `InputStream` 开始，并使用 `BufferedReader` 来一次性读取文件内容：

```kt
import java.io.File
import java.io.InputStream
fun main(args: Array<String>) {
    val inputStream: InputStream = File("lorem.txt").inputStream()
    val inputString = inputStream.bufferedReader().use {     it.readText() }
    println(inputString)
}
```

1.  在这种情况下，输出将与文件完全相同，当然，这取决于字符集。这里有一个使用另一个字符集的例子：

```kt
import java.io.File
import java.io.InputStream
fun main(args: Array<String>) {
    val inputStream: InputStream = File("lorem.txt").inputStream()
    val inputString =   inputStream.bufferedReader(Charsets.ISO_8859_1).use {  it.readText() }
    println(inputString)
}
```

1.  现在，让我们快速查看一个不获取此文件 `inputStream` 的代码示例：

```kt
import java.io.File
import java.io.InputStream
fun main(args: Array<String>) {
    val inputString = File("lorem.txt").bufferedReader().use { it.readText() }
    println(inputString)
}
```

1.  尽管您可能已经猜到了输出，但这里还是提供了输出：

```kt
Lorem ipsum dolor sit amet, consectetur adipiscing elit.
Nunc consequat eleifend mauris, eget congue ipsum consectetur id.
Proin hendrerit felis metus, vitae suscipit mi tempus facilisis.
Proin ut leo tellus. Donec nec lacus vel ante venenatis porttitor et sit amet purus.
Sed tincidunt turpis ac metus pharetra dapibus.
Integer sed auctor tellus. Morbi a metus luctus, viverra enim vel, imperdiet est.
Curabitur purus massa, hendrerit id ligula et, finibus elementum purus.
In ut consectetur lacus.
Suspendisse non mauris eget dolor faucibus pharetra quis sed turpis.
Vivamus eget lectus vel mi faucibus dignissim.
Class aptent taciti sociosqu ad litora torquent per conubia nostra, per inceptos himenaeos.
Ut vitae velit non nunc consectetur imperdiet.
Nunc feugiat diam tellus, in pellentesque nisl dapibus quis.
Proin luctus sapien ac ante tempor, eget mollis odio aliquet.
```

# 它是如何工作的...

`BufferedReader` 在读取时会将一些字符存储在缓冲区中，因此读取操作更快。我们可以直接将 `BufferedReader` 连接到文件或流，并从中读取。

`use()` 方法确保在执行完成后关闭文件或流。

# 使用 bufferedReader 逐行读取

在这个菜谱中，我们将了解如何使用 `bufferedReader` 逐行读取文件内容。

# 准备工作

您需要安装一个首选的开发环境，该环境可以编译和运行 Kotlin。您也可以使用命令行来完成此目的，这需要安装 Kotlin 编译器和 JDK。您还可以使用 IntelliJ IDEA 作为开发环境。

# 如何操作...

在给定的步骤中，我们将学习如何使用 `BufferedReader` 逐行读取文件：

1.  让我们从获取文件的 `InputStream` 开始，并使用 `BufferedReader` 来逐行读取文件内容：

```kt
import java.io.File
import java.io.InputStream
fun main(args: Array<String>) {
    val listOfLines = mutableListOf<String>()
    val inputStream: InputStream = File("lorem.txt").inputStream()
    inputStream.bufferedReader().useLines {
        lines -> lines.forEach {
            var x = "# (" + it.length + ") " + it.substring(0,8);
            listOfLines.add(x)
        }
    }
    listOfLines.forEach{println(it)}
}
```

在这种情况下，输出如下：

```kt
# (56) Lorem ip
# (65) Nunc con
# (64) Proin he
# (84) Proin ut
# (47) Sed tinc
# (81) Integer
# (71) Curabitu
# (24) In ut co
# (68) Suspendi
# (46) Vivamus
# (91) Class ap
# (46) Ut vitae
# (60) Nunc feu
# (61) Proin lu
```

1.  如果我们使用字符集，输出将取决于我们使用的字符集。这是一个带有字符集的代码示例：

```kt
import java.io.File
import java.io.InputStream
fun main(args: Array<String>) {
    val listOfLines = mutableListOf<String>()
    val inputStream: InputStream = File("lorem.txt").inputStream()
    inputStream.bufferedReader(Charsets.US_ASCII).useLines {
        lines -> lines.forEach {
            var x = "# (" + it.length + ") " + it.substring(0,8);
            listOfLines.add(x)
        }
    }
    listOfLines.forEach{println(it)}
}
```

1.  现在，让我们通过一个直接从文件读取的代码示例来了解：

```kt
import java.io.File
import java.io.InputStream
fun main(args: Array<String>) {
    val listOfLines = mutableListOf<String>()
    File("lorem.txt").bufferedReader().useLines {
        lines -> lines.forEach {
            var x = "# (" + it.length + ") " + it.substring(0,8);
            listOfLines.add(x)
        }
    }
    listOfLines.forEach{println(it)}
}
```

# 它是如何工作的...

`BufferedReader` 为我们提供了许多方法，我们可以使用这些方法逐行读取文件或输入流的内容。使用 `useLines()`，我们可以获取一个行序列，然后我们可以使用 `forEach` 来迭代它。用户可能会终止迭代循环，因此调用者需要关闭 `BufferedReader`，这正是 `useLines()` 所做的。我们只能迭代返回的序列一次。

`useLines()` 的语法如下：

```kt
inline fun <T> File.useLines(
    charset: Charset = Charsets.UTF_8, 
    block: (Sequence<String>) -> T
): T
```

# 还有更多…

我们还可以使用其他方法，如 `readLine()` 来实现这个目的。以下是一个示例代码：

```kt
import java.io.File
import java.io.InputStream
fun main(args: Array<String>) {
    val listOfLines = mutableListOf<String>()
    val reader = File("lorem.txt").bufferedReader()
    while(true) {
        var line = reader.readLine()
        if(line == null) break
        listOfLines.add("> "+line)
    }
    listOfLines.forEach{println(it)}
}
```

使用 `useLines()` 方法的优点是它在执行后关闭流。此外，前面示例中的代码是完成同样任务的一种更符合 Kotlin 风格和更简洁的方式。

Kotlin 提供的另一个返回行序列的方法是 `lineSequence()`，但它执行后不会关闭 `BufferedReader`，这就是为什么使用 `useLines()` 是一个好的选择。

最后，这取决于代码将要被使用的场景。

# 通过网络读取字符串和 JSON

网络是应用程序的一个基本组件。我们使用的许多应用程序都连接到互联网，并涉及在互联网上读取/写入数据。在这个菜谱中，我们将学习如何在 Kotlin 中执行网络请求。虽然你也可以使用像 Retrofit、Volley 这样的第三方库，但了解在 Kotlin 中是如何实现的仍然很有价值。所以，让我们开始吧！

# 准备工作

我们将使用 Android 代码，所以我将使用 Android Studio。还需要包含 anko-commons 库，因为我们将使用其方法来编写代码。

# 如何做…

让我们按照以下步骤来了解如何在 Kotlin 中进行网络请求：

1.  在 Kotlin 中使用简单的语法进行网络请求非常直接。以下是你在 Kotlin 中如何读取网络数据的方法：

```kt
val response = URL("<api_request>").readText()
```

1.  就这样！记住，当进行网络请求时，这相当于 Java 代码：

```kt
// 1\. Declare a URL Connection
URL url = new URL("http://www.google.com");
HttpURLConnection conn = (HttpURLConnection) url.openConnection();
// 2\. Open InputStream to connection
conn.connect();
InputStream in = conn.getInputStream();
// 3\. Download and decode the string response using builder
StringBuilder stringBuilder = new StringBuilder();
BufferedReader reader = new BufferedReader(new InputStreamReader(in));
String line;
while ((line = reader.readLine()) != null) {
    stringBuilder.append(line);
}

```

1.  然而，当然，如果你在主线程上尝试它，你会得到 `NetworkOnMainThreadException` 异常。为了避免这种情况，我们需要在后台进行网络调用。一种方法是通过使用 `Async` 任务。在 Java 中实现 `Async` 任务是一个痛苦的过程，但我们可以使用 Anko（一个 Kotlin 的库）轻松地做到这一点。这是你在 Kotlin 中使用 Anko 创建后台任务的方法：

```kt
doAsync{
    val response=URL("<network_url>").readText()
    uiThread{
       // Here you would do UI operation
       toast(" ... ")
    }
}
```

1.  在 Java 的 `Async` 任务实现中，即使活动正在被销毁，`Async` 任务也可能被触发。这导致了防御性编程，你必须检查 UI 是否仍然存在来进行 UI 操作。然而，Anko 的后台任务实现会处理这个问题，并且如果活动正在死亡，它不会触发任务。

# 它是如何工作的…

`doAsync`返回一个 Java `Future`。简单来说，`Future`是一个代理或包装器，它围绕着一个尚未存在的对象。当异步操作完成时，你可以从中提取它。如果你想要避免与`Future`一起工作，`doAsync`有一个不同的结构，它接受一个`ExecutorService`：

```kt
val executor = Executors.newScheduledThreadPool(5)

doAsync(executorService = executor){
    val result = URL("https://httpbin.org/get").readText()
    uiThread {
        toast(result)
    }
}
```

正如我们讨论的那样，如果活动正在关闭，则不会执行`uiThread`块。原因是它不持有上下文实例，而只持有弱引用。因此，即使该块未完成，上下文也不会泄漏。
