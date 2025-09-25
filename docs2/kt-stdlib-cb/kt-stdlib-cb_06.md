# 第六章：友好的 I/O 操作

在本章中，我们将介绍以下食谱：

+   读取文件内容

+   使用`use`函数确保流关闭

+   逐行读取文件内容

+   将内容写入文件

+   追加文件

+   简单的文件复制

+   遍历目录中的文件

# 简介

本章重点介绍 Kotlin 在 JVM 上使用`File`、`InputStream`和`OutputStream`类型的方法。我们将探索由标准库在`kotlin.io`包下提供的扩展函数组，这些函数专注于增强对 I/O 操作的支持。请注意，目前，在 Kotlin 版本 1.2 中，以下食谱仅适用于针对 JVM 平台的目标代码。

# 读取文件内容

在本食谱中，我们将检索文件内容作为文本并将其打印到控制台。我们将使用标准库的`File.readText()`扩展函数，它返回一个表示给定`File`实例文本内容的`String`。

# 准备工作

确保你的项目中包含一个非空样本文件以读取其内容。你可以克隆书中 GitHub 仓库提供的样本项目：[`github.com/PacktPublishing/Kotlin-Standard-Library-Cookbook`](https://github.com/PacktPublishing/Kotlin-Standard-Library-Cookbook)。在本食谱中，我们将使用位于样本项目`src/main/resources`目录中的`file1.txt`文件。

# 如何做到这一点...

1.  导入`File.separator`常量并将其分配一个别名：

```kt
import java.io.File.separator as SEPARATOR
```

1.  声明一个存储将要读取的文件路径的`String`：

```kt
val filePahtName = "src${SEPARATOR}main${SEPARATOR}resources${SEPARATOR}file1.txt" 
```

1.  使用指定的路径实例化一个`File`：

```kt
val filePahtName = "src${SEPARATOR}main${SEPARATOR}resources${SEPARATOR}file1.txt"
val file = File(filePahtName)
```

1.  从文件中读取文本并将其打印到控制台：

```kt
val filePahtName = "src${SEPARATOR}main${SEPARATOR}resources${SEPARATOR}file1.txt"
val file = File(filePahtName)
val fileText: String = file.readText()
println(fileText)
```

# 它是如何工作的...

`readText()`扩展函数返回表示给定文件文本的`String`值。这是一个方便读取文件内容的方法，因为它封装了从`FileInputStream`类读取字节的底层逻辑。在底层，在读取文件字节之前，该函数会检查文件是否有适当的大小以存储在内存中。

请记住，如果文件太大，将抛出`OutOfMemoryError`。每当文件太大而无法一次性处理时，你应该使用`BufferedReader`来访问其内容。你可以通过调用`File.bufferedReader()`扩展函数轻松地获得`BufferedReader`实例。

`readText()`函数还可以接受`charset: Charset`参数，默认设置为`Charsets.UTF_8`值。如果你想使用另一个`charset`，你可以通过传递适当的参数作为`charset`来指定它。你可以在`kotlin.text.Charsets`对象中找到可用的 charset 类型。你还可以在官方文档中找到它们：[`kotlinlang.org/api/latest/jvm/stdlib/kotlin.text/-charsets`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.text/-charsets)。

您可能已经注意到我们正在使用`File.separator`常量而不是硬编码的`"/"`字符。多亏了这一点，我们可以确保在不同的平台上使用正确的目录分隔符。为了简洁起见，您可以将`File.separator`导入为别名，例如`import java.io.File.separator as separator`。

# 参见

+   您还可以查看*逐行读取文件内容*配方，该配方解释了如何有效地逐行读取文件文本内容。

# 使用 use 函数确保流关闭

每当我们通过`FileInputStream`或`FileOutputStream`访问`File`的内容时，我们应该记住在完成文件操作后关闭它们。未关闭的流可能导致内存泄漏和性能显著下降。在这个配方中，我们将探讨如何使用标准库中`kotlin.io`包提供的`use()`扩展函数来自动关闭流。

# 准备工作

确保您的项目中包含一个非空样本文件以读取其内容。您可以通过克隆书中 GitHub 存储库提供的样本项目：[`github.com/PacktPublishing/Kotlin-Standard-Library-Cookbook`](https://github.com/PacktPublishing/Kotlin-Standard-Library-Cookbook)。在这个配方中，我们将使用位于样本项目`src/main/resources`目录中的`file1.txt`文件。

# 如何做到这一点...

1.  导入`File.separator`常量并将其分配别名：

```kt
import java.io.File.separator as SEPARATOR
```

1.  声明一个存储将要读取的文件路径的`String`：

```kt
val filePahtName = "src${SEPARATOR}main${SEPARATOR}resources${SEPARATOR}file1.txt" 
```

1.  为`file1.txt`文件实例化`FileInputStream`：

```kt
val filePahtName = "src${SEPARATOR}main${SEPARATOR}resources${SEPARATOR}file1.txt"
val stream = File(filePahtName).inputStream()
```

1.  在`use()`函数内部从流中读取字节：

```kt
val fileName = "src${SEPARATOR}main${SEPARATOR}resources${SEPARATOR}file1.txt"
val stream = File(fileName).inputStream()
stream.use {
    it.readBytes().also { println(String(it)) }
}
```

# 它是如何工作的...

首先，我们使用`File.inputStream()`扩展函数创建`FileInputStream`实例。接下来，我们在我们的流实例上调用`use()`扩展函数，将包含我们想要在流上执行的操作的 lambda 块作为参数。

在底层，在调用 lambda 表达式之后，`use()`函数会在流变量上调用`close()`函数。我们可以检查，当我们再次尝试使用`stream`变量访问文件时，将会抛出一个`java.io.IOException: Stream Closed`异常。

`use()`函数扩展了实现`Closeable`接口的任何类型。它接受一个 lambda 块作为参数，将可关闭资源实例作为参数传递给 lambda。`use`函数返回 lambda 块返回的值。在底层，使用了一个`try…catch`块，其中在`finally`块中调用了资源的`close()`函数。

# 逐行读取文件内容

在这个配方中，我们将检索文件内容作为一系列连续的文本行。我们将使用标准库扩展函数`File.readLines()`来返回一个表示给定`File`实例后续行的`String`类型的`List`。

# 准备工作

确保你的项目中包含一个非空样本文件以读取其内容。你可以在 GitHub 仓库中找到本书提供的样本项目：[`github.com/PacktPublishing/Kotlin-Standard-Library-Cookbook`](https://github.com/PacktPublishing/Kotlin-Standard-Library-Cookbook)。在这个菜谱中，我们将使用样本项目中位于`src/main/resources`目录下的`file1.txt`文件。

# 如何做...

1.  导入`File.separator`常量并为其指定别名：

```kt
import java.io.File.separator as SEPARATOR
```

1.  声明一个`String`，用于存储我们将要读取的文件的路径：

```kt
val filePahtName = "src${SEPARATOR}main${SEPARATOR}resources${SEPARATOR}file1.txt" 
```

1.  使用指定的路径实例化一个`File`：

```kt
val filePahtName = "src${SEPARATOR}main${SEPARATOR}resources${SEPARATOR}file1.txt"
val file = File(filePahtName)
```

1.  从文件中读取文本并将其打印到控制台：

```kt
val filePathName = "src${SEPARATOR}main${SEPARATOR}resources${SEPARATOR}file1.txt"
val file = File(fileName)
val fileLines = file.readLines()
fileLines.forEach { println(it) }
```

# 工作原理...

`readLines()`扩展函数返回一个`List<String>`实例，表示给定文件的文本行。这是一种方便读取文件内容的方法，因为它封装了从`FileInputStream`类读取字节的底层逻辑。

请记住，如果文件太大，将抛出`OutOfMemoryError`。每当文件太大而无法一次性处理时，你应该使用`BufferedReader`来访问其内容。你可以通过调用`File.bufferedReader()`扩展函数轻松地获得`BufferedReader`实例。

`readLines()`函数还可以接受`charset: Charset`参数，默认设置为`Charsets.UTF_8`值。如果你想使用其他字符集，你可以通过传递适当的参数作为`charset`来指定它。你可以在`kotlin.text.Charsets`对象中找到可用的字符集类型。你还可以在官方文档中找到它们：[`kotlinlang.org/api/latest/jvm/stdlib/kotlin.text/-charsets`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.text/-charsets)。

你可能已经注意到我们使用的是`File.separator`常量而不是硬编码的`"/"`字符。多亏了这一点，我们可以确保在不同的平台上使用正确的目录分隔符。为了简洁起见，你可以使用别名导入`File.separator`，例如`import java.io.File.separator as separator`。

# 参考内容

+   你还可以查看*读取文件内容*菜谱，它解释了如何一次性检索文件的文本内容作为`String`值

# 将内容写入文件

在这个菜谱中，我们将学习如何轻松创建一个新的`File`并向其写入文本。我们将使用标准库提供的`File.writeText()`扩展函数。然后，我们将通过将其打印到控制台来验证文件是否已成功创建以及是否包含正确的内容。

# 如何做...

1.  导入`File.separator`常量并为其指定别名：

```kt
import java.io.File.separator as SEPARATOR
```

1.  指定我们将要创建的新文件的路径：

```kt
val fileName = "src${SEPARATOR}main${SEPARATOR}resources${SEPARATOR}temp_file"
```

1.  使用指定的文件路径实例化文件：

```kt
val fileName = "src${SEPARATOR}main${SEPARATOR}resources${SEPARATOR}temp_file"
val file = File(fileName)
```

1.  在`apply`块中使用`writeText()`函数将文本写入文件：

```kt
val fileName = "src${SEPARATOR}main${SEPARATOR}resources${SEPARATOR}temp_file"
val file = File(fileName)
file.apply {
    val text =
 "\"No one in the brief history of computing " +
 "has ever written a piece of perfect software. " +
 "It's unlikely that you'll be the first.\" - Andy Hunt" 
```

```kt
 writeText(text) 
}
```

1.  将`temp_file`的内容打印到控制台：

```kt
val fileName = "src${SEPARATOR}main${SEPARATOR}resources${SEPARATOR}temp_file"
val file = File(fileName)
file.apply {
    val text =
        "\"No one in the brief history of computing " +
          "has ever written a piece of perfect software. " +
          "It's unlikely that you'll be the first.\" - Andy Hunt"
    writeText(text)
}
file.readText().apply { println(this) }
```

# 它是如何工作的...

执行前面的代码后，将在`src/main/resources`目录下创建一个新的`temp_file`文件。请注意，如果`temp_file`已存在，它将被覆盖。接下来，使用`writeText()`函数，其内容将被打印到控制台：

```kt
"No one in the brief history of computing has ever written a piece of perfect software. It's unlikely that you'll be the first." - Andy Hunt
```

`writeText()`函数封装了`java.io.FileOutputStream` API，提供了一种优雅的方式将内容写入文件。在底层，它访问`use()`函数中的`FileOutputStream`，因此您可以确信它会在写入操作期间自动关闭任何打开的流。

如果您要写入文件中的文本太大，无法一次性处理，您可以使用`BufferedWriter` API 来允许您写入和追加文件。您可以使用`File.bufferedWriter()`扩展函数轻松地获取`BufferedWriter`的实例。

您还可以向`writeText()`函数传递额外的`charset: Charset`参数，默认值为`Charsets.UTF_8`。如果您想使用其他字符集，可以通过传递适当的参数作为`charset`来指定它。您可以在`kotlin.text.Charsets`对象中找到可用的字符集类型。您也可以在官方文档中找到它们，链接为[`kotlinlang.org/api/latest/jvm/stdlib/kotlin.text/-charsets`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.text/-charsets)。

# 相关内容

+   查看关于*追加文件*的食谱，了解如何通过追加内容以灵活的方式修改文件内容

# 追加文件

在这个食谱中，我们将学习如何通过使用标准库提供的`File.appendText()`扩展函数轻松地创建一个新的`File`并写入文本。我们将通过多次追加其内容来验证文件是否成功创建以及是否包含正确的内容，并将其打印到控制台。

# 如何做到这一点...

1.  导入`File.separator`常量并将其分配一个别名：

```kt
import java.io.File.separator as SEPARATOR
```

1.  指定将要创建的新文件的路径：

```kt
val fileName = "src${SEPARATOR}main${SEPARATOR}resources${SEPARATOR}temp_file"
```

1.  使用指定的文件路径实例化文件：

```kt
val fileName = "src${SEPARATOR}main${SEPARATOR}resources${SEPARATOR}temp_file"
val file = File(fileName)
```

1.  如果文件已存在，则删除该文件：

```kt
val fileName = "src${SEPARATOR}main${SEPARATOR}resources${SEPARATOR}temp_file"
val file = File(fileName)
if (file.exists()) file.delete()
```

1.  使用下一个字符串值追加文件：

```kt
val fileName = "src${SEPARATOR}main${SEPARATOR}resources${SEPARATOR}temp_file"
val file = File(fileName)
if (file.exists()) file.delete()

file.apply {
    appendText("\"A language that doesn't affect the way you think ")
 appendText("about programming ")
 appendText("is worth knowing.\"")
 appendText("\n")
 appendBytes("Alan Perlis".toByteArray())
}
```

1.  将文件内容打印到控制台：

```kt
val fileName = "src${SEPARATOR}main${SEPARATOR}resources${SEPARATOR}temp_file"
val file = File(fileName)
if (file.exists()) file.delete()

file.apply {
    appendText("\"A language that doesn’t affect the way you think ")
    appendText("about programming ")
    appendText("is worth knowing.\"")
    appendText("\n")
    appendBytes("Alan Perlis".toByteArray())
}

file.readText().let { println(it) }
```

# 它是如何工作的...

执行前面的代码后，将在`src/main/resources`目录下创建一个新的`temp_file`文件，并且其内容将被打印到控制台：

```kt
"A language that doesn’t affect the way you think about programming is worth knowing."
Alan Perlis
```

`appendText()`和`appendBytes()`函数封装了`java.io.FileOutputStream` API，提供了一种优雅的方式将内容追加到文件中。在底层，它们在`use()`函数中访问`FileOutputStream`，因此您可以确信它会在写入操作期间自动关闭任何打开的流。

如果你想写入文件中的文本太大，一次无法处理，你可以使用`BufferedWriter` API，它允许你写入和追加文件。你可以通过使用`File.bufferedWriter()`扩展函数轻松地获得`BufferedWriter`实例。

你还可以将额外的`charset: Charset`参数传递给`appendText()`函数，默认值等于`Charsets.UTF_8`。如果你想使用其他字符集，你可以通过传递适当的参数作为`charset`参数来指定它。你可以在`kotlin.text.Charsets`对象中找到可用的字符集类型。你还可以在官方文档中找到它们，网址为[`kotlinlang.org/api/latest/jvm/stdlib/kotlin.text/-charsets`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.text/-charsets)。

# 简单的文件复制

在这个菜谱中，我们将探索一种将文件内容复制到新文件中的巧妙方法。我们将从指定的路径获取一个样本`File`实例，并将其内容复制到新文件中。最后，我们将打印两个文件的内容到控制台以验证操作。

# 准备工作

确保你的项目中包含一个非空样本文件以读取其内容。你可以在 GitHub 仓库中克隆本书提供的样本项目：[`github.com/PacktPublishing/Kotlin-Standard-Library-Cookbook`](https://github.com/PacktPublishing/Kotlin-Standard-Library-Cookbook)。在这个菜谱中，我们将使用位于样本项目`src/main/resources`目录中的`file2.txt`文件。

# 如何操作...

1.  导入`File.separator`常量并为其指定别名：

```kt
import java.io.File.separator as SEPARATOR
```

1.  为指定的`file2.txt`路径实例化一个`File`对象：

```kt
val sourceFileName = "src${SEPARATOR}main${SEPARATOR}resources${SEPARATOR}file2.txt"
val sourceFile = File(sourceFileName)
```

1.  创建一个名为`file2_copy.txt`的新`File`：

```kt
val sourceFileName = "src${SEPARATOR}main${SEPARATOR}resources${SEPARATOR}file2.txt"
val sourceFile = File(sourceFileName)

val targetFileName =         "src${SEPARATOR}main${SEPARATOR}resources${SEPARATOR}file2_copy.txt"
val targetFile = File(targetFileName)
```

1.  如果`file2_copy.txt`存在，则删除它：

```kt
val sourceFileName = "src${SEPARATOR}main${SEPARATOR}resources${SEPARATOR}file2.txt"
val sourceFile = File(sourceFileName)

val targetFileName =     "src${SEPARATOR}main${SEPARATOR}resources${SEPARATOR}file2_copy.txt"
val targetFile = File(targetFileName)

if (targetFile.exists()) targetFile.delete()
```

1.  将`file2.txt`的内容复制到`file2_copy.txt`文件中：

```kt
val sourceFileName = "src${SEPARATOR}main${SEPARATOR}resources${SEPARATOR}file2.txt"
val sourceFile = File(sourceFileName)

val targetFileName =     "src${SEPARATOR}main${SEPARATOR}resources${SEPARATOR}file2_copy.txt"
val targetFile = File(targetFileName)

if (targetFile.exists()) targetFile.delete()

sourceFile.copyTo(targetFile)
```

1.  将两个文件打印到控制台以进行验证：

```kt
val sourceFileName = "src${SEPARATOR}main${SEPARATOR}resources${SEPARATOR}file2.txt"
val sourceFile = File(sourceFileName)

val targetFileName =     "src${SEPARATOR}main${SEPARATOR}resources${SEPARATOR}file2_copy.txt"
val targetFile = File(targetFileName)

if (targetFile.exists()) targetFile.delete()

sourceFile.copyTo(targetFile)

File(sourceFileName).readText().apply { println(this) }
File(targetFileName).readText().apply { println(this) }
```

# 它是如何工作的...

你可以运行样本代码以验证，在调用`copyTo()`扩展函数后，两个文件包含相同的内容。在我们的例子中，我们得到以下输出：

```kt
"Testing can show the presence of errors, but not their absence." - E. W. Dijkstra
"Testing can show the presence of errors, but not their absence." - E. W. Dijkstra
```

在幕后，`copyTo()`函数读取源文件中的`InputStream`到缓冲区，并将其写入目标文件的`OutputStream`。内部，流在`use()`函数块中被访问，操作完成后会自动关闭它们。

除了目标`File`实例外，`copyTo()`函数还接受两个可选参数——`overwrite: Boolean`，默认设置为`false`，以及`bufferSize: Int`，默认值。请注意，无论目标文件路径上的哪些目录缺失，它们都将被创建。此外，如果目标文件已存在，`copyTo()`函数将失败，除非覆盖参数设置为`true`。

+   当`overwrite`参数设置为`true`且`target`指向一个目录时，只有当它为空时才会替换。

+   如果在一个指向目录的 `File` 实例上调用 `copyTo()`，它将不带内容进行复制。在目标路径下只会创建一个空目录。

+   `copyTo()` 函数不会保留复制的文件属性，也就是说，不会保留创建/修改日期和权限。

# 遍历目录中的文件

在这个菜谱中，我们将探讨如何遍历给定目录中的文件。我们将从指向目录的给定 `File` 获取 `FileTreeWalk` 类实例。我们将遍历给定目录内的所有文件，包括任何嵌套的子目录。我们还将过滤文件，排除那些没有 `.txt` 扩展名的文件，并将它们的路径和内容打印到控制台。

# 准备工作

确保你的项目中包含了具有 `.txt` 扩展名的样本文件。您可以从 GitHub 仓库中克隆本书提供的样本项目：[`github.com/PacktPublishing/Kotlin-Standard-Library-Cookbook`](https://github.com/PacktPublishing/Kotlin-Standard-Library-Cookbook)。在这个菜谱中，我们将使用样本项目的 `src/main/resources` 目录及其内容。

# 如何操作...

1.  导入 `File.separator` 常量并将其分配一个别名：

```kt
import java.io.File.separator as SEPARATOR
```

1.  从指向 `src/main/resources` 目录的 `File` 获取 `FileTreeWalk` 实例：

```kt
val directoryPath = "src${SEPARATOR}main${SEPARATOR}resources"
val fileTreeWalk: FileTreeWalk = File(directoryPath).walk()
```

1.  遍历所有非空 `.txt` 文件并打印：

```kt
val directoryPath = "src${SEPARATOR}main${SEPARATOR}resources"

val fileTreeWalk: FileTreeWalk = File(directoryPath).walk()
fileTreeWalk
 .filter { it.isFile }
        .filter { it.extension == "txt" }
        .filter { it.readBytes().isNotEmpty() }
        .forEach {
            it.apply {
                println(path)
 println(readText())
 println()
 }
        }
```

# 工作原理...

我们首先实例化一个引用 `src/main/resources` 目录的 `File` 类型实例，并在其上调用 `walk()` 函数。`walk()` 返回 `FileTreeWalk` 实例，这是一个在文件系统之上的高级抽象层，允许我们遍历原始 `File` 对象的文件和子目录。

`FileTreeWalk` 扩展了 `Sequence<File>` 接口，并提供了 `Iterator<File>` 实现，这允许我们遍历文件并对它们应用任何转换操作，就像我们在处理集合时做的那样。

接下来，我们应用一些过滤操作——移除引用目录的 `File` 对象，移除不包含 `.txt` 扩展名的文件，以及从序列中移除任何空文件。最后，我们使用 `forEach()` 函数一起打印连续文件的路径及其内容。

如您所观察到的，`FileTreeWalk` 序列提供的默认顺序是从上到下。我们可以通过调用 `walk()` 函数并设置 `direction` 参数为 `FileWalkDirection.BOTTOM_UP` 来定义一个反向序列。

`walk()` 函数也有两种现成的专用变体可用——`File.walkTopDown()` 和 `File.walkBottomUp()`。前者返回一个 `FileTreeWalk` 实例，其方向属性设置为 `FileWalkDirection.TOP_DOWN`，而后者将 `direction` 设置为 `FileWalkDirection.BOTTOM_UP`。
