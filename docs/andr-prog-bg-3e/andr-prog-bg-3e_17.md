# 第十七章：*第十七章*：数据持久性和共享

在本章中，我们将看一下将数据保存到 Android 设备的永久存储的几种不同方法。此外，我们将首次向我们的应用程序添加第二个`Activity`。当在我们的应用程序中实现一个单独的“屏幕”，比如设置屏幕时，将其放在一个新的`Activity`中通常是有意义的。我们可以费力地隐藏原始 UI，然后显示新的 UI，但这很快会导致令人困惑和容易出错的代码。因此，我们将看看如何添加一个`Activity`类并在它们之间导航用户。

总之，在本章中，我们将做以下事情：

+   学习如何使用 Android 意图在`Activity`类之间切换并传递数据

+   为“Note to Self”项目在一个新的`Activity`类中创建一个简单（非常简单）的设置屏幕

+   使用`SharedPreferences`类持久化设置屏幕数据

+   学习**JavaScript 对象表示**（**JSON**）进行序列化

+   探索 Java 的`try`-`catch`-`finally`语法

+   在我们的“Note to Self”应用程序中实现保存数据

# 技术要求

您可以在 GitHub 上找到本章中的代码文件，网址为[`github.com/PacktPublishing/Android-Programming-for-Beginners-Third-Edition/tree/main/chapter%2017`](https://github.com/PacktPublishing/Android-Programming-for-Beginners-Third-Edition/tree/main/chapter%2017)。

# Android 意图

`Intent`类的命名非常恰当。它是一个演示我们应用程序中`Activity`类意图的类。它使意图清晰，并且也方便了它。

到目前为止，我们所有的应用程序都只有一个`Activity`，但许多 Android 应用程序包含多个`Activity`。

在它可能最常见的用法中，`Intent`类允许我们在`Activity`实例之间切换。但是，`Activity`实例是由具有成员变量的类制作的。那么，当我们在它们之间切换时，变量的值 - 数据 - 会发生什么？意图通过允许我们在`Activity`实例之间传递数据来解决了这个问题。

意图不仅仅是关联我们应用程序的`Activity`实例。它们还使我们能够与其他应用程序进行交互。例如，我们可以在我们的应用程序中提供一个链接，让用户发送电子邮件，打电话，与社交媒体互动，或在浏览器中打开网页，并让电子邮件应用程序、电话应用程序、社交媒体应用程序或网络浏览器完成所有工作。

这本书中没有足够的页面来深入研究与其他应用程序的交互，我们主要将专注于在活动之间切换和传递数据。

## 切换活动

假设我们的应用程序有两个基于`Activity`的类，因为我们很快就会有。我们可以假设通常情况下，我们有一个名为`MainActivity`的`Activity`类，这是应用程序的起点，还有一个名为`SettingsActivity`的第二个`Activity`。这是我们如何从`MainActivity`切换到`SettingsActivity`的方法：

```kt
// Declare and initialize a new Intent object called myIntent
Intent myIntent = new Intent(this, SettingsActivity.class);
// Switch to the SettingsActivity
startActivity(myIntent);
```

仔细看看我们如何初始化`Intent`对象。`Intent`有一个构造函数，它接受两个参数。第一个是对当前`Activity`的引用，`this`。第二个参数是我们想要打开的`Activity`类的名称，`SettingsActivity.class`。`SettingsActivity`末尾的`.class`部分使其成为`AndroidManifest.xml`文件中声明的`Activity`类的完整名称，我们将在不久的将来尝试意图时查看它。

唯一的问题是`SettingsActivity`不共享`MainActivity`的任何数据。从某种意义上说，这是一件好事，因为如果你需要从`MainActivity`获取所有数据，那么切换活动可能不是进行应用程序设计的最佳方式。然而，封装得如此彻底以至于这两个`Activity`实例完全不知道对方是不合理的。

## 在活动之间传递数据

如果我们为用户有一个登录屏幕，并且我们想要将用户的凭据传递给我们应用程序的每个`Activity`，我们可以使用意图来实现。

我们可以像这样向`Intent`类添加数据：

```kt
// Create a String called username 
// and set its value to bob
String username = "Bob";
// Create a new Intent as we have already seen
Intent myIntent = new Intent(this, SettingsActivity.class);
// Add the username String to the Intent
// using the putExtra method of the Intent class
myIntent.putExtra("USER_NAME", username);
// Start the new Activity as we did before
startActivity(myIntent);
```

在`SettingsActivity`中，我们可以这样检索字符串：

```kt
// Here we need an Intent also
// But the default constructor will do
// as we are not switching Activity
Intent myIntent = new Intent();
// Initialize username with the passed in String 
String username = intent.getExtra().getStringKey("USER_NAME");
```

在前两个代码块中，我们以与之前相同的方式切换了`Activity`。但在调用`startActivity`方法之前，我们使用`putExtra`方法将字符串加载到意图中。

我们使用`identifier`实例添加数据，该数据可以在检索`Activity`中用于标识和检索数据。

标识符名称由您决定，但应使用有用/易记的值。

然后，在接收`Activity`中，我们只需使用默认构造函数创建一个意图：

```kt
Intent myIntent = new Intent();
```

然后，我们可以使用`getExtras`方法和键值对中的适当标识符检索数据。

一旦我们想要开始发送多个值，就值得考虑不同的策略。

`Intent`类可以在发送比这更复杂的数据时帮助我们，但`Intent`类有其限制。例如，我们将无法发送`Note`对象。

# 向“自我备忘录”添加设置页面

现在我们已经掌握了关于 Android `Intent`类的所有知识，我们可以向我们的“自我备忘录”应用程序添加另一个屏幕（`Activity`）。我们将添加一个设置屏幕。

我们将首先为我们的设置屏幕创建一个新的`Activity`，并查看这对`AndroidManifest.xml`文件的影响；然后我们将为我们的设置屏幕创建一个非常简单的布局，并添加 Java 代码以从`MainActivity`切换到新的布局。但是，在学习如何将设置保存到磁盘之前，我们将推迟使用 Java 连接我们的设置屏幕。我们将在本章后面进行此操作，然后返回设置屏幕以使其数据持久化。

首先，让我们创建一个新的`Activity`类。我们将其称为`SettingsActivity`。

## 创建 SettingsActivity

这将是一个屏幕，用户可以在`RecyclerView`小部件中的每个笔记之间打开或关闭装饰性分隔符。这不会是一个全面的设置屏幕，但这将是一个有用的练习，并且我们将学习如何在活动之间切换以及将数据保存到磁盘。按照以下步骤开始：

1.  在项目资源管理器中，右键单击包含所有`.java`文件并与您的包名称相同的文件夹。从弹出的上下文菜单中，选择**新建** | **Activity** | **空白 Activity**。

1.  在`SettingsActivity`中。

1.  将所有其他选项保留为默认值，然后单击**完成**。

Android Studio 已为我们创建了一个基于`Activity`的类及其关联的`.java`文件。让我们快速查看一下幕后为我们完成的一些工作，因为了解正在发生的事情是有用的。

从项目资源管理器中的`manifests`文件夹中打开`AndroidManifest.xml`文件。注意文件中间大约有一半的以下代码行：

```kt
<activity android:name=".SettingsActivity"></activity>
```

这就是`Activity`类是如何`Activity`类未注册，然后尝试运行它将使应用程序崩溃。我们可以通过在新的`.java`文件中创建一个扩展`Activity`（或`AppCompatActivity`）的类来创建`Activity`类。但是，我们随后必须自己添加前面的代码。此外，通过使用新的`Activity`向导，我们自动生成了一个布局 XML 文件（`activity_settings.xml`）。

## 设计设置屏幕布局

我们将快速为我们的设置屏幕构建一个 UI，以下步骤和图示应该使这变得简单：

1.  打开`activity_settings.xml`文件，切换到**设计**选项卡，然后我们将快速布置我们的设置屏幕。在执行其余步骤时，请使用下一个图示作为指南：![图 17.1 – 设计设置屏幕](img/Figure_17.1_B16773.jpg)

图 17.1 – 设计设置屏幕

1.  将**Switch**小部件拖放到布局的中上部。我通过拖动边缘来拉伸我的小部件，使其变大。

1.  添加一个`id`属性`switch1`（如果默认情况下还没有），这样我们就可以使用`SettingsActivity.java`中的 Java 代码与其交互。

1.  使用约束处理程序来固定开关的位置，或者单击**推断约束**按钮以自动修复它。

我们现在为设置屏幕有了一个简单的新布局，并且`id`属性已经就位，准备在本章后面的 Java 代码中连接它。

## 使用户能够切换到设置屏幕

我们已经知道如何切换到`SettingsActivity`。此外，由于我们不会向其传递任何数据，也不会从中传递任何数据，因此我们可以只用两行 Java 代码就可以使其工作。

您可能已经注意到我们应用程序的操作栏中有一个菜单图标。这是我们创建项目时使用的基本活动模板的默认部分。如下图所示：

![图 17.2 – 菜单图标](img/Figure_17.2_B16773.jpg)

图 17.2 – 菜单图标

如果您点击它，您会发现默认情况下已经有一个菜单选项**设置**。当您点击菜单图标时，您将看到以下内容：

![图 17.3 – 设置选项](img/Figure_17.3_B16773.jpg)

图 17.3 – 设置选项

我们只需要将切换到`SettingsActivity`类的代码放在`MainActivity`类的`onOptionsItemSelected`方法中。Android Studio 甚至默认为我们提供了一个`if`块，以便我们将来可能想要添加设置屏幕。多么体贴。

切换到编辑器窗口中的`MainActivity.java`，并在`onOptionsItemSelected`方法中找到以下代码块：

```kt
//noinspection SimplifiableIfStatement
if (id == R.id.action_settings) {
   return true;
}
```

将此代码添加到先前显示的`if`块中，就在`return true`语句之前：

```kt
Intent intent = new Intent(this, SettingsActivity.class);
startActivity(intent);
```

您需要使用您喜欢的技术导入`Intent`类，以添加以下代码行：

```kt
import android.content.Intent;
```

现在，您可以运行应用程序并通过点击**设置**菜单选项来访问新的设置屏幕。此屏幕截图显示了模拟器上运行的设置屏幕：

![图 17.4 – 设置屏幕](img/Figure_17.4_B16773.jpg)

图 17.4 – 设置屏幕

要从`SettingsActivity`返回到`MainActivity`，您可以点击设备上的返回按钮。

# 使用 SharedPreferences 持久化数据

在 Android 中，有几种方法可以使数据持久化。持久化意味着如果用户退出应用程序，然后再次打开应用程序，他们的数据仍然可用。使用哪种方法取决于应用程序和数据类型。

在本书中，我们将介绍三种使数据持久化的方法。对于保存用户设置，我们只需要一个简单的方法。毕竟，我们只需要知道他们是否希望在`RecyclerView`小部件的每个笔记之间有装饰性分隔符。

让我们看看如何使我们的应用程序将变量保存和重新加载到设备的内部存储器中。我们需要使用`SharedPreferences`类。`SharedPreferences`是一个提供对可以由应用程序的所有`Activity`类访问和编辑的数据的访问权限的类。让我们看看如何使用它：

```kt
// A SharedPreferences for reading data
SharedPreferences prefs;
// A SharedPreferences.Editor for writing data
SharedPreferences.Editor editor; 
```

与所有对象一样，我们需要在使用之前初始化它们。我们可以通过使用`getSharedPreferences`方法并传递一个字符串来初始化`prefs`对象，该字符串将用于引用使用该对象读取和写入的所有数据。通常，我们可以使用应用程序的名称作为此字符串的值。在下一个代码中，`MODE_PRIVATE`表示任何类（仅在此应用程序中）都可以访问它：

```kt
prefs = getSharedPreferences("My App", MODE_PRIVATE);
```

然后，我们使用新初始化的`prefs`对象通过调用`edit`方法来初始化我们的`editor`对象：

```kt
editor = prefs.edit();
```

假设我们想要保存我们在名为`username`的字符串中拥有的用户名称。然后，我们可以像这样将数据写入设备的内部存储器中：

```kt
editor.putString("username", username);
editor.commit();
```

`putString`方法中使用的第一个参数是一个标签，可以用来引用数据；第二个是保存我们想要保存的数据的实际变量。上述代码的第二行启动了保存过程。因此，我们可以这样将多个变量写入磁盘：

```kt
editor.putString("username", username);
editor.putInt("age", age);
editor.putBoolean("newsletter-subscriber", subscribed);
// Save all the above data
editor.commit();
```

上述代码演示了您可以保存其他变量类型，并且它当然假定`username`，`age`和`subscribed`变量先前已经被声明并用适当的值初始化。

一旦`editor.commit()`执行完毕，数据就被存储了。我们甚至可以退出应用程序，甚至关闭设备，数据仍然会持久保存。

# 使用 SharedPreferences 重新加载数据

让我们看看下次运行应用程序时如何重新加载数据。这段代码将重新加载上一个代码保存的三个值。我们甚至可以声明我们的变量并用存储的值进行初始化：

```kt
String username  = 
   prefs.getString("username", "new user");
int age  = prefs.getInt("age", -1);
boolean subscribed = 
   prefs.getBoolean("newsletter-subscriber", false)
```

在上述代码中，我们使用适合数据类型的方法从磁盘加载数据，并使用了保存数据的相同标签。不太清楚的是每个方法调用的第二个参数。

`getString`，`getInt`和`getBoolean`方法需要一个默认值作为第二个参数。如果没有存储带有该标签的数据，它将返回默认值。

然后我们可以在我们的代码中检查这些默认值，并尝试获取真实的值。例如，看这里：

```kt
if (age == -1){
   // Ask the user for his age
}
```

我们现在知道足够的知识来保存我们用户的设置在 Note to Self 应用程序中。

# 使“Note to Self”设置持久化

我们已经学会了如何将数据保存到设备的内存中。当我们实现保存用户设置时，我们还将再次看到我们如何处理`Switch`输入，以及我们刚刚看到的代码将去哪里使我们的应用程序按我们想要的方式工作。

## 编写 SettingsActivity 类

大部分操作将在`SettingsActivity.java`文件中进行。因此，点击适当的选项卡，我们将逐步添加代码。

首先，我们需要一些成员变量，这些变量将为我们提供工作的`SharedPreferences`和`Editor`实例。我们还希望有一个成员变量来表示用户的设置选项：他们是否想要装饰性分隔线。

在`SettingsActivity`类的类声明之后添加以下成员变量：

```kt
private SharedPreferences mPrefs;
private SharedPreferences.Editor mEditor;
private boolean mShowDividers;
```

导入`SharedPreferences`类：

```kt
import android.content.SharedPreferences;
```

现在在`onCreate`方法中，添加代码来初始化`mPrefs`和`mEditor`：

```kt
mPrefs = getSharedPreferences("Note to self", MODE_PRIVATE);
mEditor = mPrefs.edit();
```

接下来，在`onCreate`方法中，让我们获取对我们的`Switch`小部件的引用，并加载代表我们用户先前选择是否显示分隔线的保存数据。

我们以与*第十三章**相同的方式获取对开关的引用，匿名类-让 Android 小部件活跃起来*。请注意默认值为`true`-显示分隔线。我们还将根据需要将开关设置为打开或关闭：

```kt
mShowDividers  = mPrefs.getBoolean("dividers", true);
Switch switch1 = findViewById(R.id.switch1);
// Set the switch on or off as appropriate
switch1.setChecked(mShowDividers);
```

您需要导入`Switch`类：

```kt
import android.widget.Switch;
```

接下来，我们创建一个匿名类来监听和处理对我们的`Switch`小部件的更改。

当`isChecked`变量为`true`时，我们使用`prefs`对象将`dividers`标签和`mShowDividers`变量设置为`true`；当未选中时，我们将它们都设置为`false`。

将以下代码添加到我们刚刚讨论的`onCreate`方法中：

```kt
switch1.setOnCheckedChangeListener(
      new CompoundButton.OnCheckedChangeListener() {
             public void onCheckedChanged(
                         CompoundButton buttonView, 
                               boolean isChecked) {
                   if(isChecked){
                         mEditor.putBoolean(
                                     "dividers", true);

                         mShowDividers = true;
                   }else{
                         mEditor.putBoolean(
                                     "dividers", false);

                         mShowDividers = false;
                   }
             }
      }
);
```

您需要导入`CompoundButton`类：

```kt
import android.widget.CompoundButton;
```

您可能已经注意到，在任何代码中，我们都没有调用`mEditor.commit`方法来保存用户的设置。我们本可以在检测到开关变化后放置它，但将它放在保证被调用但只调用一次的地方更简单。

我们将利用我们对`Activity`生命周期的了解，并重写`onPause`方法。当用户离开`SettingsActivity`屏幕，无论是返回到`MainActivity`还是退出应用程序，操作系统都会调用`onPause`方法，并保存设置。在`SettingsActivity`类的结束大括号之前添加以下代码来重写`onPause`方法并保存用户的设置：

```kt
@Override
protected void onPause() {
   super.onPause();
   // Save the settings here
   mEditor.commit();
}
```

最后，我们可以向`MainActivity`类添加一些代码，以在应用程序启动时或用户从设置屏幕切换回主屏幕时加载设置。

## 编写 MainActivity 类

在我们声明`NoteAdapter`实例后添加一些成员变量的突出代码：

```kt
private List<Note> noteList = new ArrayList<>();
private RecyclerView recyclerView;
private NoteAdapter mAdapter;
private boolean mShowDividers;
private SharedPreferences mPrefs;
```

导入`SharedPreferences`类：

```kt
import android.content.SharedPreferences;
```

现在我们有一个`boolean`成员来决定是否显示分隔符，以及一个`SharedPreferences`实例来从磁盘读取设置。

现在我们将重写`onResume`方法，初始化我们的`mPrefs`变量，并将设置加载到`mShowDividers`变量中。

在`MainActivity`类中添加重写的`onResume`方法，如下所示：

```kt
@Override
protected void onResume(){
   super.onResume();
   mPrefs = getSharedPreferences(
         "Note to self", MODE_PRIVATE);

   mShowDividers  = mPrefs.getBoolean(
         "dividers", true);
}
```

用户现在能够选择他们的设置。应用程序将根据需要保存和重新加载它们，但我们需要让`MainActivity`类响应用户的选择。

在`onCreate`方法中找到这段代码并删除它：

```kt
// Add a neat dividing line between items in the list
recyclerView.addItemDecoration(
   new DividerItemDecoration(
   this, LinearLayoutManager.VERTICAL));
```

上一个代码是在列表中的每个笔记之间设置分隔符。将这段新代码添加到`onResume`方法中，这是相同的代码行，被`if`语句包围，只有当`mShowDividers`为`true`时才选择性地使用分隔符。在`onResume`方法中在上一个代码后添加以下代码：

```kt
if(mShowDividers) {
   // Add a neat dividing line between list items
   recyclerView.addItemDecoration(
         new DividerItemDecoration(
         this, LinearLayoutManager.VERTICAL));
}else{
   // check there are some dividers
// or the app will crash
if(recyclerView.getItemDecorationCount() > 0) {
         recyclerView.removeItemDecorationAt(0);
}
}
```

运行应用程序并注意到分隔符已经消失；转到设置屏幕，打开分隔符，然后返回到主屏幕（使用返回按钮）- 看哪：现在有分隔符了。下一个图显示了带有和不带有分隔符的列表，以便说明我们添加的代码起作用，并且设置在这两个 Activity 类之间持续存在：

![图 17.5 - 带有和不带有分隔符的列表](img/Figure_17.5_B16773.jpg)

图 17.5 - 带有和不带有分隔符的列表

确保尝试退出应用程序并重新启动以验证设置是否已保存到磁盘。甚至可以关闭并重新打开模拟器，设置将持续存在。

现在我们有一个整洁的设置屏幕，我们可以永久保存用户选择的装饰偏好。当然，关于持久性的一个重要缺失环节是用户的基本数据，他们的笔记，仍然没有持久保存。

# 更高级的持久性

让我们考虑一下我们需要做什么。我们想要将一堆笔记保存到内部存储中。更具体地说，我们想要存储一系列字符串和相关的布尔值。这些字符串和布尔值代表用户的笔记标题、笔记文本，以及它是待办事项、重要事项还是想法。

考虑到我们已经了解的`SharedPreferences`类，乍一看这似乎并不特别具有挑战性 - 直到我们更深入地了解我们的需求。如果用户喜欢我们的应用程序并最终拥有 100 条笔记怎么办？我们需要 100 个键值对的标识符。这并非不可能，但开始变得尴尬。

现在考虑一下，我们想要增强应用程序，并让用户能够为它们添加日期。Android 有一个`Date`类非常适合这个用途。然后，我们可以相对简单地为我们的应用程序添加一些新功能，比如提醒。但是当涉及到保存数据时，事情突然变得复杂起来。

我们如何使用`SharedPreferences`来存储日期？它并不是为此而设计的。我们可以在保存时将其转换为字符串，然后在加载时再次转换回来，但这远非简单。

随着我们的应用程序功能的增加和用户获取更多的笔记，整个持久性问题变得一团糟。我们需要的是一种保存和加载对象的方法，实际的 Java 对象。如果我们可以简单地保存和加载对象，包括它们的内部数据（字符串，布尔值，日期或其他任何东西），我们的应用程序可以拥有我们可以想到的任何类型的数据，以满足我们的用户。

将数据对象转换为位和字节以存储在磁盘上的过程称为**序列化**；反向过程称为**反序列化**。单独的序列化是一个庞大而复杂的主题。幸运的是，正如我们所期望的那样，有一个类来处理大部分复杂性。

## 什么是 JSON？

**JSON**代表**JavaScript 对象表示**，它在 Android 和 Java 语言之外的领域被广泛使用。它可能最常用于在 Web 应用程序和服务器之间发送数据。

幸运的是，Android 上有可用的 JSON 类，几乎完全隐藏了序列化过程的复杂性。通过学习一些更多的 Java 概念，我们可以快速开始使用这些类，并开始将整个 Java 对象写入设备存储，而不必担心构成对象的原始类型是什么。

与我们迄今为止见过的其他类相比，JSON 类执行的操作具有比正常情况下更高的失败概率，这是超出它们的控制范围的。要找出原因以及可以采取什么措施，让我们看看 Java 异常。

## Java 异常 - try，catch 和 finally

所有这些关于 JSON 的讨论要求我们学习一个新的 Java 概念：**异常**。当我们编写一个执行可能失败的操作的类，特别是由于我们控制之外的原因，建议在我们的代码中明确这一点，以便任何使用我们的类的人都能为可能性做好准备。

保存和加载数据是一个这样的场景，其中失败是可能的，超出了我们的控制范围。想想当 SD 卡已被移除或已损坏时尝试加载数据的情况。另一个代码可能失败的情况是当我们编写依赖于网络连接的代码时 - 如果用户在数据传输过程中断网了会怎么样？

Java 异常是解决方案，JSON 类使用它们，所以现在是学习它们的好时机。

当我们编写一个使用有可能失败的代码的类时，我们可以通过使用`try`，`catch`和`finally`来为我们的类的用户做好准备。

我们可以在我们的类中编写方法，在签名的末尾使用`throws` Java 关键字 - 可能是这样的：

```kt
public void somePrecariousMethod() throws someException{
   // Risky code goes here
}
```

现在，任何使用`somePrecariousMethod`方法的代码都需要`try`和`catch`块 - 也许像这样：

```kt
try{
   ...
   somePrecariousMethod();
   ...
}catch(someException e){
   Log.e("Exception:" + e, "Uh ohh")
   // Take action if possible
}
```

可选地，如果我们想在`try`和`catch`块之后执行任何进一步的操作，我们还可以添加一个`finally`块：

```kt
finally{
   // More action here
}
```

在我们的“自言自语”应用中，我们将采取最少的操作来处理异常，并简单地将错误输出到 logcat，但您可以做一些事情，比如通知用户，重试操作，或者实施一些聪明的备用计划。

# 备份用户数据在“自言自语”中

因此，通过我们对异常的新认识，让我们修改我们的“自言自语”代码，然后我们可以介绍`JSONObject`和`JSONException`类。

首先，让我们对我们的`Note`类进行一些小修改。添加一些更多的成员，这些成员将作为我们`Note`类的每个方面的键值对的键：

```kt
private static final String JSON_TITLE = "title";
private static final String JSON_DESCRIPTION = "description";
private static final String JSON_IDEA = "idea";
private static final String JSON_TODO = "todo";
private static final String JSON_IMPORTANT = "important";
```

现在添加一个构造函数和一个接收`JSONObject`并抛出`JSONException`的空默认构造函数。构造函数的主体通过调用`JSONObject`的`getString`或`getBoolean`方法，传入键作为参数，来初始化定义单个`Note`对象属性的每个成员。我们还提供了一个空的默认构造函数，现在我们提供了我们的专门的构造函数，这是必需的：

```kt
// Constructor
// Only used when new is called with a JSONObject
public Note(JSONObject jo) throws JSONException {

   mTitle =  jo.getString(JSON_TITLE);
   mDescription = jo.getString(JSON_DESCRIPTION);
   mIdea = jo.getBoolean(JSON_IDEA);
   mTodo = jo.getBoolean(JSON_TODO);
   mImportant = jo.getBoolean(JSON_IMPORTANT);
}
// Now we must provide an empty default constructor 
// for when we create a Note as we provide a
// specialized constructor.
public Note (){
}
```

您需要导入`JSONException`和`JSONObject`类：

```kt
import org.json.JSONException;
import org.json.JSONObject;
```

接下来我们将看到的代码将给定`Note`对象的成员变量加载到`JSONObject`中。这是`Note`对象的成员被打包为一个单独的`JSONObject`，以便进行实际的序列化。

我们只需要使用适当的键和匹配的成员变量调用`put`方法。该方法返回`JSONObject`（我们将在一分钟内看到它在哪里），它还会抛出一个`JSONObject`异常。添加我们刚刚讨论过的代码：

```kt
public JSONObject convertToJSON() throws JSONException{

   JSONObject jo = new JSONObject();
   jo.put(JSON_TITLE, mTitle);
   jo.put(JSON_DESCRIPTION, mDescription);
   jo.put(JSON_IDEA, mIdea);
   jo.put(JSON_TODO, mTodo);
   jo.put(JSON_IMPORTANT, mImportant);
   return jo;
}
```

现在让我们创建一个`JSONSerializer`类，它将执行实际的序列化和反序列化。创建一个新类并将其命名为`JSONSerializer`。

让我们将其分成几个部分，并在编写每个部分的代码时讨论我们正在做什么。

首先，声明和一些成员变量：一个字符串来保存数据将被保存的文件名，以及一个`Context`对象，在 Android 中写入数据到文件是必要的。在您刚刚创建的类中添加突出显示的代码：

```kt
public class JSONSerializer {
    private String mFilename;
    private Context mContext;

   // All the rest of the code for the class goes here

}// End of class
```

注意

您需要导入`Context`类：

`import android.content.Context;`

先前的代码显示，类的结束大括号和随后为该类编写的所有代码应该放在其中。这是非常简单的构造函数，我们在其中初始化了作为参数传入的两个成员变量。添加`JSONSerializer`的构造函数：

```kt
public JSONSerializer(String fn, Context con){
   mFilename = fn;
   mContext = con;
}
```

现在我们可以开始编写类的真正要点。接下来是`save`方法。它首先创建一个`JSONArray`对象，这是一个专门用于处理 JSON 对象的`ArrayList`类。

接下来，代码使用增强的`for`循环来遍历`notes`数组列表中的所有`Note`对象，并使用`Note`类中我们之前添加的`convertToJSON`方法将它们转换为 JSON 对象。然后，我们将这些转换后的`JSONObject`实例加载到`jArray`中。

接下来，代码使用`Writer`实例和`OutputStream`实例组合将数据写入实际文件。请注意，`OutputStream`实例需要初始化`mContext`对象。添加我们刚刚讨论过的代码：

```kt
public void save(List<Note> notes)
   throws IOException, JSONException{
   // Make an array in JSON format
   JSONArray jArray = new JSONArray();
   // And load it with the notes
   for (Note n : notes)
         jArray.put(n.convertToJSON());
   // Now write it to the private disk space of our app
   Writer writer = null;
   try {
         OutputStream out = 
         mContext.openFileOutput(mFilename,
                mContext.MODE_PRIVATE);
         writer = new OutputStreamWriter(out);
         writer.write(jArray.toString());
   } finally {
         if (writer != null) {
                writer.close();
         }
   }
}
```

您需要为这些新类添加以下`import`语句：

```kt
import org.json.JSONArray;
import org.json.JSONException;
import java.io.IOException;
import java.io.OutputStream;
import java.io.OutputStreamWriter;
import java.io.Writer;
import java.util.List;
```

现在进行反序列化 - 加载数据。这一次，正如我们所预料的，该方法不接收任何参数，而是返回一个`ArrayList`实例。使用`mContext.openFileInput`创建一个`InputStream`实例，打开包含所有数据的文件。

我们使用`while`循环将所有数据附加到一个字符串中，并使用我们的新`Note`构造函数，该构造函数将 JSON 数据提取到常规原始变量中，以将每个`JSONObject`解包到一个`Note`对象中，并将其添加到返回给调用代码的`ArrayList`中：

```kt
public ArrayList<Note> load() throws IOException, JSONException{
   ArrayList<Note> noteList = new ArrayList<Note>();
   BufferedReader reader = null;
   try {
         InputStream in = 
         mContext.openFileInput(mFilename);
         reader = new BufferedReader(new 
         InputStreamReader(in));
         StringBuilder jsonString = new StringBuilder();
         String line = null;
         while ((line = reader.readLine()) != null) {
                jsonString.append(line);
         }
         JSONArray jArray = (JSONArray) new
               JSONTokener(jsonString.toString()
               ).nextValue();
         for (int i = 0; i < jArray.length(); i++) {
                noteList.add(new 
                Note(jArray.getJSONObject(i)));
         }
   } catch (FileNotFoundException e) {
         // we will ignore this one, since it happens
         // when we start fresh. You could add a log here.
   } finally {// This will always run
         if (reader != null)
                reader.close();
   }
   return noteList;
}
```

您需要添加以下导入：

```kt
import org.json.JSONTokener;
import java.io.BufferedReader;
import java.io.FileNotFoundException;
import java.io.InputStream;
import java.io.InputStreamReader;
import java.util.ArrayList;
```

现在我们只需要让我们的新类在`MainActivity`类中工作。在`MainActivity`声明之后添加一个新成员，如下所示。此外，删除`noteList`的初始化，只留下声明，因为我们现在将在`onCreate`方法中使用一些新代码来初始化它。我已经注释掉了您需要删除的行：

```kt
public class MainActivity extends AppCompatActivity {
   private JSONSerializer mSerializer;
   //private List<Note> noteList = new ArrayList<>();
   private List<Note> noteList;
```

现在，在`onCreate`方法中，我们通过使用文件名和`getApplicationContext()`调用`JSONSerializer`构造函数来初始化`mSerializer`，它返回应用程序的`Context`实例并且是必需的。然后我们可以使用`JSONSerializer load`方法来加载任何保存的数据。在处理浮动操作按钮的代码之后添加这段新的突出显示的代码。这段新代码必须出现在我们处理`RecyclerView`实例之前：

```kt
…
FloatingActionButton fab = 
         (FloatingActionButton) findViewById(R.id.fab);

fab.setOnClickListener(new View.OnClickListener() {
         @Override
         public void onClick(View view) {
                DialogNewNote dialog = new DialogNewNote();
                dialog.show(getSupportFragmentManager(), "");
         }
});
   mSerializer = new JSONSerializer("NoteToSelf.json", 
   getApplicationContext());
   try {
         noteList = mSerializer.load();
   } catch (Exception e) {
         noteList = new ArrayList<Note>();
         Log.e("Error loading notes: ", "", e);
   }
   recyclerView =
         findViewById(R.id.recyclerView);
mAdapter = new NoteAdapter(this, noteList);
             …
```

注意

此时您需要导入`Log`类：

`import android.util.Log;`

在先前的代码中，我展示了大量的上下文，因为它的定位对于它的工作是至关重要的。如果您在运行时遇到任何问题，请务必将其与*第十七章*`/java`文件夹中的下载包中的代码进行比较。

现在我们可以向`MainActivity`类添加一个新的方法，这样我们就可以调用它来保存所有用户的数据。这个新方法所做的就是调用`JSONSerializer`类的`save`方法，传入所需的`Note`对象列表：

```kt
public void saveNotes(){
   try{
         mSerializer.save(noteList);
   }catch(Exception e){
         Log.e("Error Saving Notes","", e);
   }
}
```

现在，就像我们保存用户设置时所做的那样，我们将重写`onPause`方法来保存用户的笔记数据。确保在`MainActivity`类中添加这段代码：

```kt
@Override
protected void onPause(){
   super.onPause();
   saveNotes();
}
```

就是这样。我们现在可以运行应用程序，并添加任意多的笔记。`ArrayList`会将它们全部存储在我们运行的应用程序中，我们的`RecyclerAdapter`将管理在`RecyclerView`中显示它们，现在 JSON 也会负责将它们加载到磁盘上，并加载它们回来。

# 经常问的问题

1.  我并没有完全理解本章的所有内容 - 我不适合成为程序员吗？

本章介绍了许多新的类、概念和方法。如果你的头有点疼，这是可以预料的。如果一些细节不清楚，不要让它阻碍你。继续进行接下来的几章（它们要简单得多），然后回顾这一章并检查已完成的代码文件。

1.  那么，序列化的详细工作原理是什么？

序列化确实是一个广阔的主题。你可能一辈子都写应用程序而从未真正需要理解它。这可能是计算机科学学位的主题。如果你想了解更多，请看一下这篇文章：[`en.wikipedia.org/wiki/Serialization`](https://en.wikipedia.org/wiki/Serialization)。

# 总结

在我们通过 Android API 的旅程中，现在值得回顾一下我们所知道的。我们可以布置自己的 UI 设计，并从各种各样的小部件中选择，让用户与 UI 进行交互。我们可以创建多个屏幕以及弹出对话框，并且可以捕获全面的用户数据。此外，我们现在可以使这些数据持久化。

当然，还有很多关于 Android API 的知识需要学习，甚至超出了这本书所教授的范围，但重点是我们现在已经知道足够的知识来规划和实现一个可工作的应用程序。你现在就可以开始自己的应用程序了。

如果你有立即开始自己的项目的冲动，那么我的建议是继续前进并去做。不要等到你认为自己是一个“专家”或更加准备好。阅读这本书，更重要的是，实现应用程序将使你成为一个更好的 Android 程序员，但没有什么比设计和实现自己的应用程序更能让你更快地学会！完全可以通过阅读这本书并同时进行自己的项目工作。

在下一章中，我们将通过使应用程序支持多语言来为这个应用程序添加最后的修饰。这非常快速和简单。
