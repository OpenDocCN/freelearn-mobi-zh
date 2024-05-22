# 第十八章：*第十八章*：本地化

本章内容简洁明了，我们将学习的内容可以使您的应用程序对数百万潜在用户更加可访问。我们将看到如何添加额外的语言。我们还将看到通过字符串资源以正确的方式添加文本在添加多种语言时对我们的好处。

在本章中，我们将涵盖以下内容：

+   通过添加西班牙语和德语语言使“Note to Self”应用程序多语言化

+   更全面地学习如何使用字符串资源

让我们开始吧。

# 技术要求

您可以在 GitHub 上找到本章中的代码文件[`github.com/PacktPublishing/Android-Programming-for-Beginners-Third-Edition/tree/main/chapter%2018`](https://github.com/PacktPublishing/Android-Programming-for-Beginners-Third-Edition/tree/main/chapter%2018)。

# 使“Note to Self”应用程序对西班牙语和德语使用者可访问

首先，我们需要向我们的项目添加一些文件夹 - 每种新语言一个文件夹。这些文本被归类为`res`文件夹。按照以下步骤向项目添加西班牙语和德语支持。

重要提示

虽然此项目的源文件位于*第十八章*文件夹中，但它们仅供参考。您需要按照下面描述的流程来实现多语言功能。

## 添加西班牙语支持

按照以下步骤添加西班牙语支持：

1.  右键单击`values-es`。

1.  左键单击**确定**。

1.  现在我们需要添加一个文件，我们可以把所有的西班牙语翻译放在里面。在**目录名称**字段中，右键单击`values-es`中的`strings.xml`。

1.  左键单击**确定**。

在这个阶段，我们有一个新的文件夹，用于西班牙语翻译，里面有一个`strings.xml`文件，用于字符串资源。让我们也为德语做同样的事情。

## 添加德语语言支持

按照以下步骤添加德语语言支持：

1.  右键单击`values-de`。

1.  左键单击**确定**。

1.  现在我们需要添加一个文件，我们可以把所有的德语翻译放在里面。在**目录名称**字段中，右键单击`values-de`中的`strings.xml`。

1.  左键单击**确定**。

这就是`strings.xml`文件夹的样子。您可能想知道`strings.xml`文件夹是从哪里来的，因为它与我们似乎在之前的步骤中创建的结构不符。Android Studio 正在帮助我们（显然）组织我们的文件和文件夹，以符合 Android 操作系统的要求。但是，您可以看到西班牙语和德语文件，分别由它们的特定于国家的扩展**es**和**de**指示：

![图 18.1 - strings.xml 文件夹](img/Figure_18.1_B16773.jpg)

图 18.1 - strings.xml 文件夹

现在我们可以向文件添加翻译。

## 添加字符串资源

正如我们所知，`strings.xml`文件包含应用程序将显示的单词 - 诸如“重要”，“待办事项”，“想法”等。通过为我们想要支持的每种语言都有一个`strings.xml`文件，我们可以让 Android 根据用户的语言设置选择适当的文本。

在您进行以下操作时，请注意，尽管我们将要翻译的单词的翻译放在值中，但`name`属性保持不变。如果您仔细想想，这是合乎逻辑的，因为我们在布局文件中引用的是`name`属性。

让我们提供翻译，看看我们取得了什么成就，然后回来讨论我们将如何处理 Java 代码中的文本。实现此代码的最简单方法是从原始的`strings.xml`文件中复制并粘贴代码，然后编辑每个`name`属性的值：

1.  双击打开`strings.xml`文件。确保选择带有**(es)**后缀的文件。编辑文件使其看起来像这样：

```kt
<?xml version="1.0" encoding="utf-8"?>
<resources>
<string name="app_name">Nota a sí mismo</string>
<string name="action_settings">Configuración</string>
<string name="action_add">add</string>
<string name="title_hint">Título</string>
<string name="description_hint">Descripción</string>
<string name="idea_text">Idea</string>
<string name="important_text">Importante</string>
<string name="todo_text">Que hacer</string>
<string name="cancel_button">Cancelar</string>
<string name="ok_button">Vale</string>
<string name="settings_title">Configuración</string>
</resources>
```

1.  双击打开`strings.xml`文件。确保选择与`strings.xml`文件相邻的文件，然后缺少的资源将从默认文件中获取。

我们所做的是提供了两种翻译。Android 知道哪种翻译是为哪种语言，因为它们所放置的文件夹。此外，我们使用了一个`name`属性来引用翻译。回顾一下以前的代码，您会看到相同的标识符用于两种翻译以及原始的`strings.xml`文件。

您甚至可以将本地化到不同版本的语言，例如美国或英国英语。完整的代码列表可以在这里找到：[`stackoverflow.com/questions/7973023/what-is-the-list-of-supported-languages-locales-on-android`](http://stackoverflow.com/questions/7973023/what-is-the-list-of-supported-languages-locales-on-android)。您甚至可以本地化资源，如图像和声音。在这里了解更多信息：[`developer.android.com/guide/topics/resources/localization.html`](http://developer.android.com/guide/topics/resources/localization.html)。

这些翻译是从谷歌翻译复制并粘贴而来的，因此很可能有些翻译与正确的相去甚远。像这样廉价地进行翻译可能是将具有基本字符串资源集的应用程序放入使用不同语言的用户设备的有效方式。一旦您开始需要任何深度的翻译，也许在叙事驱动的游戏或社交媒体应用的情况下，您肯定会受益于由人类专业人员进行的翻译。

这个练习的目的是展示 Android 的工作原理，而不是如何翻译。

注意

对于可能看到这里提供的翻译的局限性的任何西班牙语或德语使用者，我表示诚挚的歉意。

现在我们已经有了翻译，我们可以看到它们在一定程度上起作用。

# 在德语或西班牙语中运行 Note to Self

运行应用程序以查看它是否正常工作。现在我们可以更改本地化设置以在西班牙语中查看它。不同的设备在如何执行此操作上略有不同，但 Pixel 3 模拟器的选项如下：

1.  选择**设置** | **系统** | **语言和输入** | **添加语言**。接下来，选择**Español**，然后您将能够在列表中在西班牙语和英语之间切换。

1.  左键单击并拖动**Español (Estados Unidos)**，使其位于列表顶部。

恭喜，您的模拟器现在默认为西班牙语。完成本章后，您可以将首选语言拖回到列表顶部。

现在您可以以通常的方式运行应用程序。以下是应用程序以西班牙语运行的一些屏幕截图。我用 Photoshop 将一些屏幕截图并排放置，以展示 Note to Self 应用程序的不同屏幕：

![图 18.2 - 应用程序以西班牙语运行](img/Figure_18.2_B16773.jpg)

图 18.2 - 应用程序以西班牙语运行

在屏幕截图中，您可以清楚地看到我们的应用程序主要是用西班牙语翻译的。显然，用户输入的文本将是他们所说的任何语言；这不是我们应用程序的缺陷。然而，仔细看屏幕截图，注意我指出了一些文本仍然是英文的地方。我们的每个对话框窗口中仍然有一些未翻译的文本。

这是因为文本直接包含在我们的 Java 代码中。正如我们所见，使用多种语言的字符串资源然后在我们的布局中引用它们是很容易的，但是我们如何从我们的 Java 代码中引用字符串资源呢？

## 使翻译在 Java 代码中起作用

首先要做的是在三个`strings.xml`文件中创建资源。以下是需要添加到三个不同文件中的两个资源。

在`strings.xml`（没有任何国家后缀），在`<resources></resources>`标签中添加这两个资源：

```kt
<string name="add_new_note">Add a new note</string>
<string name="your_note">Your note</string>
```

在`strings.xml`中使用`<resources></resources>`标签：

```kt
<string name="add_new_note">Agregar una nueva nota</string>
<string name="your_note">Su nota</string>
```

在`strings.xml`中使用`<resources></resources>`标签：

```kt
<string name="add_new_note">Eine neue Note hinzufügen</string>
<string name="your_note">Ihre Notiz</string>
```

接下来，我们需要编辑一些 Java 代码，以引用资源而不是硬编码的字符串。

打开`DialogNewNote.java`文件并找到这行代码：

```kt
builder.setView(dialogView).setMessage("Add a new note");
```

编辑如下所示，使用我们刚刚添加的字符串资源而不是硬编码的文本：

```kt
builder.setView(dialogView).setMessage(getResources().
getString(R.string.add_new_note));
```

新代码使用了链式`getResources.getString`方法来替换先前硬编码的`"Add a new note"`文本。仔细看，你会发现发送给`getString`方法的参数是`R.string.add_new_note`字符串标识符。

`R.string`代码指的是`res`文件夹中的字符串资源，`add_new_note`是我们的标识符。Android 将能够根据应用所在设备的语言环境决定使用哪个版本（默认、西班牙语或德语）。

我们还有一个硬编码的字符串需要更改。

打开`DialogShowNote.java`文件并找到这行代码：

```kt
builder.setView(dialogView).setMessage("Your Note");
```

编辑如下所示，使用我们刚刚添加的字符串资源而不是硬编码的文本：

```kt
builder.setView(dialogView).setMessage(getResources().
getString(R.string.your_note));
```

新代码再次使用了链式`getResources.getString`方法来替换先前硬编码的`"Your note"`文本。同样，发送给`getString`的参数是字符串标识符，这次是`R.string.your_note`。

Android 现在可以根据应用所在设备的语言环境决定使用哪个版本（默认、西班牙语或德语）。下一张截图显示了**新建笔记**界面现在以适当的语言显示开头文本：

![图 18.3 – 新建笔记界面](img/Figure_18.3_B16773.jpg)

图 18.3 – 新建笔记界面

您可以添加任意多个字符串资源。作为*第三章*中的提醒，*探索 Android Studio 和项目结构*，请注意，使用字符串资源是向所有项目添加任何文本的推荐方式。本书中的教程（除了《自言自语》）将倾向于硬编码它们，以便制作更紧凑的教程。

# 总结

我们已经看到了如何满足世界上说不同语言的地区。我们现在可以让我们的应用全球化，同时添加更灵活的字符串资源，而不是硬编码所有文本。

在下一章中，我们将看到如何使用动画和插值器为我们的应用添加酷炫的动画效果。
