# 第十二章：Lambda 和委托

本章将涵盖以下内容：

+   使用 lambda 的点击监听器

+   在 Kotlin 中使用懒委托

+   使用可观察的委托

+   使用可撤销的委托

+   编写自己的委托

+   使用 lateinit 修饰符

+   与 SharedPreferences 一起工作

+   在 Kotlin 中创建多个 let 的链

+   创建全局变量

# 简介

在本章中，我们将探讨 Kotlin 语言的函数式特性。Kotlin 通过 lambda 表达式内置了函数式编程。Java 直到现在都缺少这种现代语言特性，但 Java 8 已经包含了 lambda 表达式。然而，由于大多数 Android 设备不支持 Java 8，Android 开发者无法使用这个特性。在本章中，我们将介绍这些内容，并学习关于委托的知识。委托是 Kotlin 的一个强大语言特性。那么，让我们开始吧！

# 使用 lambda 的点击监听器

在 Android 中，onclick 监听器是那些曾经占用大量行数的东西之一，即使代码的重要部分只有一行。Kotlin 极大地简化了 Android 框架，其中最好的改进之一就是`onClickListener`。在本菜谱中，我们将看到如何通过 lambda 的帮助简化传统的长点击监听器。

# 准备工作

我将使用 Android Studio 3 来编写代码。您可以在 Android Studio 3+中创建一个新的 Kotlin 项目，并包含一个空白活动，因为我们不会使用其他菜谱中的任何代码。您还需要对 Android 开发有一个中级理解。

# 如何做…

让我们按照给定的步骤来了解如何使用 lambda 表达式来使用点击监听器：

1.  让我们从创建一个包含一些视图的活动开始，例如一个可以附加`onClickListener`的按钮。查看以下 XML 布局，这是一个可能的活动布局：

```kt
<?xml version="1.0" encoding="utf-8"?>
<android.support.design.widget.CoordinatorLayout

    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <android.support.design.widget.AppBarLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:theme="@style/AppTheme.AppBarOverlay">

        <android.support.v7.widget.Toolbar
            android:id="@+id/toolbar"
            android:layout_width="match_parent"
            android:layout_height="?attr/actionBarSize"
            android:background="?attr/colorPrimary"
            app:popupTheme="@style/AppTheme.PopupOverlay" />

    </android.support.design.widget.AppBarLayout>

    <LinearLayout

        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:background="@color/white"
        android:orientation="vertical"
        app:layout_behavior="@string/appbar_scrolling_view_behavior">

        <Button
            android:id="@+id/btn1"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_margin="8dp"
            android:text="@string/button1"/>

        <Button
            android:id="@+id/btn2"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_margin="8dp"
            android:text="@string/button2"/>

        <Button
            android:id="@+id/btn3"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_margin="8dp"
            android:text="@string/button3"/>

    </LinearLayout>

</android.support.design.widget.CoordinatorLayout>
```

1.  以下是我们布局的外观，其中包含三个需要附加`onClickListener`的按钮：

![图片](img/dccf397f-96e8-434c-baf3-37c3b3f64ad8.jpeg)

1.  现在，让我们看看当我们在 Java 中为三个按钮附加`onClickListener`时的代码：

```kt
public class HelloWorldActivity extends AppCompatActivity {
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_hello_world);
        final Button btn1 = (Button) findViewById(R.id.btn1);
        final Button btn2 = (Button) findViewById(R.id.btn2);
        final Button btn3 = (Button) findViewById(R.id.btn3);
        final Toolbar toolbar = (Toolbar) findViewById(R.id.toolbar);
        setSupportActionBar(toolbar);
        getSupportActionBar().setTitle("Let's click");
        btn1.setOnClickListener(new View.OnClickListener() {
 public void onClick(View v) {
 Toast.makeText(HelloWorldActivity.this, "Button 1 has been clicked by you! One", Toast.LENGTH_SHORT).show();
 }
 });
        btn2.setOnClickListener(new View.OnClickListener() {
 public void onClick(View v) {
 Toast.makeText(HelloWorldActivity.this, "Button 2 has been clicked by you! Two.", Toast.LENGTH_SHORT).show();
 }
 });
 btn3.setOnClickListener(new View.OnClickListener() {
 public void onClick(View v) {
 Toast.makeText(HelloWorldActivity.this, "Button 3 has been clicked by you! Three", Toast.LENGTH_SHORT).show();
 }
 });
    }
}
```

1.  你注意到我们为了仅仅附加显示 toast 的点击监听器而必须编写的代码量吗？所有这些代码只是为了三个 onclick 监听器；现在，让我们看看在 Kotlin 中编写相同代码的差异：

```kt
class HelloWorldActivity2 : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        setSupportActionBar(toolbar)
        supportActionBar?.title = "Let's click"
        btn1.setOnClickListener(object : View.OnClickListener {
 override fun onClick(v: View?) {
 toast("Button 1 has been clicked by you! One")
 }
 })
 btn2.setOnClickListener(object : View.OnClickListener {
 override fun onClick(v: View?) {
 toast("Button 2 has been clicked by you! Two")
 }
 })
 btn3.setOnClickListener(object : View.OnClickListener {
 override fun onClick(v: View?) {
 toast("Button 3 has been clicked by you! Three")
 }
 })
    }
}
```

1.  代码确实更少、更整洁，这要归功于 Kotlin 的**合成属性**和**Anko**的 toast 辅助函数。现在，让我们尝试使用**lambda**并看看它带来的差异：

```kt
class HelloWorldActivity2 : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        setSupportActionBar(toolbar)
        supportActionBar?.title = "Let's click"
        btn1.setOnClickListener({ toast("Button 1 has been clicked by you! One") })
 btn2.setOnClickListener({ toast("Button 2 has been clicked by you! Two") })
 btn3.setOnClickListener({ toast("Button 3 has been clicked by you! Three") })
    }
}
```

哇！向 lambda 的强大力量致敬。注意代码量减少了多少，看起来更整洁、更易读。这是一次大量的样板代码减少，这为我们节省了时间和精力。

# 它是如何工作的…

Lambda 函数是未声明但作为表达式传递的函数。在 Kotlin 中，如果一个函数接收一个接口，我们可以用 lambda 来替换它。例如，`setOnClickListener`函数接收`View.OnClickListener`，因此我们可以使用 lambda：

```kt
fun setOnClickListener(listener: (View) -> Unit)
someView.setOnClickListener({ view -> doSomething() })
```

此外，如果没有参数要传递给 lambda 函数，我们可以省略箭头，如果最后一个传递的参数是函数，我们可以将其移出括号：

```kt
someView.setOnClickListener() { doSomething() }
```

然后，如果传递给 lambda 函数的实际上是唯一参数，您可以完全省略括号：

```kt
button.setOnClickListener { doSomething() }
```

# 还有更多...

我们可以使用 Kotlin 的库 **Anko** 进一步减少代码。Anko 提供了一个接受 lambda 表达式的方法，该表达式在 `onClick` 事件发生时执行：

```kt
class HelloWorldActivity2 : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        setSupportActionBar(toolbar)
        supportActionBar?.title = "Let's click"
        btn1.onClick { toast("Button 1 has been clicked by you! One") }
 btn2.onClick { toast("Button 2 has been clicked by you! Two") }
 btn3.onClick { toast("Button 3 has been clicked by you! Three") }
    }
}
```

# 在 Kotlin 中使用懒代理

懒加载结构主要用于属性的懒初始化，这在初始化的对象是重对象（需要时间初始化）时尤其有用。在启动时实例化重对象可能会导致移动用户体验中的性能下降。懒初始化可以解决我们的问题。在这个菜谱中，我们将学习如何使用 Kotlin 的懒代理，让我们开始吧！

# 准备工作

我们将使用 Android Studio 3.0 进行编码，请确保您已下载最新版本。

# 如何做...

在以下步骤中，我们将学习如何在 Kotlin 中使用懒代理：

1.  首先，让我们看看如何通过懒初始化创建属性。语法如下：

```kt
val / var <property name>: <Type> by <delegate>
```

1.  在创建懒代理时，我们使用 `by lazy`，如下所示：

```kt
class MainActivity : AppCompatActivity() {
    private val textView : TextView by lazy {
        findViewById<TextView>(R.id.textView) as TextView
    }
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        textView.text="ABC"
    }
}
```

懒代理在第一次访问时初始化对象并存储值，然后对于后续访问返回该值。

# 它是如何工作的...

属性的委托看起来像这样：

```kt
class Delegate {
    operator fun getValue(
            thisRef: Any?,
            property: KProperty<*>
    ): String {
        // return value
    }
 operator fun setValue(
            thisRef: Any?,
            property: KProperty<*>, value: String
    ) {
        // assign
    }
}
```

读取操作调用 `getValue` 方法，而写入操作调用 `setValue`。

懒属性有三个评估模式：

+   `LazyThreadSafetyMode.SYNCHRONIZED`：初始化仅在单个线程上发生。其余线程看到缓存的值。这也是初始化的默认模式。

+   `LazyThreadSafetyMode.PUBLICATION`：当不需要初始化代理的同步时使用。它可以从多个线程同时调用，并且可以在每个线程上执行初始化。然而，如果初始化由一个线程执行，它将返回而不执行初始化。

+   `LazyThreadSafetyMode.NONE`：不使用锁来同步初始化，因此开销较小。

# 使用可观察的代理

之前，我们看到了如何处理委托属性。在这个菜谱中，我们将学习如何处理可观察的代理。这个代理帮助我们观察属性中的任何变化。那么，让我们开始吧。

# 准备工作

我们将使用 IntelliJ IDEA 编写代码。您可以使用任何能够执行 Kotlin 代码的 IDE。

# 如何做...

可观察的代理接受一个默认值和一个包含旧值和新值的结构。让我们看看下一个示例：

```kt
fun main(args: Array<String>) {
    var a:String by Delegates.observable("",{_,oldValue,newValue ->
        println("old value: $oldValue, new value: $newValue ")
    })
    a="a"
    a="b"
>}
//Output:old value: , new value: a 
 old value: a, new value: b
```

在前面的例子中，我们将初始值提供为空字符串。每次我们尝试更新`a`属性的值时，都会执行该结构。我们已经更改了`a`的值两次，因此我们看到了两条打印语句。

# 还有更多…

可观察的委托在`RecyclerView`的情况下特别有用，因为我们可以使用`DiffUtils`来更新仅更改的项目，而不是用新的列表替换整个列表。有关更多信息，请参阅[第四章](https://www.safaribooksonline.com/library/view/kotlin-programming-cookbook/9781788472142/e79a66d5-bec1-4560-b363-2489175de44b.xhtml)中的配方，*在 Kotlin 中创建 RecyclerView 适配器*。

# 使用可撤销委托

**可撤销委托**与可观察委托非常相似，唯一的区别是它拒绝更改。在可观察委托中，每当可观察属性更改时，我们都可以获取新值和旧值。让我们看看 Kotlin 文档中提供的定义：

"返回一个读写属性委托，当属性更改时调用指定的回调函数，允许回调拒绝修改。"

# 准备工作

我将使用 IntelliJ IDEA 进行编码。您可以使用任何能够执行 Kotlin 代码的 IDE。

# 如何做…

让我们现在看看给定的步骤来理解`vetoable`修饰符：

1.  让我们快速看一下`vetoable`委托属性的实现：

```kt
fun main(args: Array<String>) {
    var student:Student by Delegates.vetoable(Student(10),{property, oldValue, newValue ->
        if(newValue.age>25){
            println("Age can't be greater than 25")
            return@vetoable false
        }
        true
    })
    student=Student(26)
}
class Student(var age:Int)
```

```kt
//Output: Age can't be greater than 25
```

1.  如您所见，由于年龄不能大于`25`，修改被`vetoable`委托“拒绝”。只有当年龄小于`25`时，才会分配新对象。

# 它是如何工作的…

让我们看看可撤销委托属性的声明：

```kt
inline fun <T> vetoable(
initialValue: T, 
crossinline onChange: (property: KProperty<*>, oldValue: T, newValue: T) -> Boolean
): ReadWriteProperty<Any?, T> (source)
```

`vetoable()`接受一个初始值，这可以是一个空列表，还有一个`onChange`回调，它在更改属性之前被调用。如果更改成功，回调返回 true；如果被拒绝，则返回 false。

# 还有更多…

如果你在`RecyclerView`适配器中使用可撤销委托，它特别有用。通常，你会直接将数据分配给列表，并可能调用`notifyDatasetChanged`，但这非常低效，因为它会导致重新加载所有数据。我们可以使用可撤销委托通过匹配旧值和新值来检查内容是否相同，并在内容相同的情况下拒绝修改。此外，我们可以使用`DiffUtils`仅更新更改的数据。`DiffUtils`是在 Android 支持库 26.01 及以后的版本中引入的，使`RecyclerView`更加高效。

# 编写你自己的委托

委托属性是 Kotlin 语言中最好的特性之一。我们已经看到了可观察的和可撤销的委托。在这个菜谱中，我们将学习如何创建我们自己的自定义委托。作为一个演示示例，我们将创建一个只能初始化一次的委托属性；如果再次初始化，它应该抛出异常。那么让我们深入探讨一下，看看我们如何实现它。

# 准备工作

我们将使用 IntelliJ IDEA 进行编码。你可以使用任何能够执行 Kotlin 代码的 IDE。

# 如何做到这一点…

现在，让我们深入探讨如何创建我们自己的委托：

1.  让我们创建一个名为 `SingleInitializationProperty` 的自定义委托。这个自定义委托属性如果变量没有被初始化，将抛出异常，并且它只能初始化一次。如果再次初始化，它将抛出异常。让我们看看我们的自定义委托类：

```kt
class SingleInitializableProperty<T>() : ReadWriteProperty<Any?, T>{
    private var value: T? = null
    override fun getValue(thisRef: Any?, property: KProperty<*>): T {
        if(value==null){
            throw IllegalStateException("Variable not initialized")
        }else {
            return value!!
        }
    }
    override fun setValue(thisRef: Any?, property: KProperty<*>, value: T) {
        if(this.value==null){
            this.value=value
        }else{
            throw IllegalStateException("Cannot be initialized twice")
        }
    }
}
```

1.  现在，我们已经创建了一个自定义委托，让我们尝试以下方式使用它而不进行初始化：

```kt
fun main(args: Array<String>) {
    var a:String by SingleInitializableProperty()
    println(a)
}
```

这就是输出结果：

```kt
Output: Exception in thread "main" java.lang.IllegalStateException: Variable not initialized
```

1.  让我们看看另一个例子；这次，我们将首先初始化它，然后访问它，然后再次尝试初始化它：

```kt
fun main(args: Array<String>) {
    var a:String by SingleInitializableProperty()
    a="first"
    println(a)
    a="second"
}
```

这里是输出结果：

```kt
Output:first
Exception in thread "main" java.lang.IllegalStateException: Cannot be initialized twice
```

# 它是如何工作的…

如你所见，我们在委托属性中实现了 `ReadWriteProperty` 接口，这意味着我们的变量将是 `var` 类型。如果你想使其不可变，你可以实现 `ReadOnlyProperty` 接口。

`getValue` 函数接收一个类的引用和属性元数据。`setValue` 函数反过来接收一个设置的值。在不可变属性（`val`）的情况下，将只有一个 `getValue` 函数。

# 使用 `lateinit` 修饰符

**延迟初始化（Lateinit**）是一个重要的初始化属性，因为如果你不想在构造函数中初始化你的变量，可以使用 `lazy` 和 `lateinit` 来实现。在这个菜谱中，我们将看到如何使用 `lateinit` 修饰符以及它与 `lazy` 修饰符的不同之处。

# 准备工作

我将使用 IntelliJ IDEA 进行编码；你可以使用任何能够执行 Kotlin 代码的 IDE。

# 如何做到这一点…

让我们按照给定的步骤来理解 `lateinit` 修饰符是如何工作的：

1.  在 Java 中，我们可以在声明变量后稍后初始化它，但 Kotlin 要求你在声明时立即初始化它（除非你使用特殊修饰符）。所以你可以这样做：

```kt
var student:Student?=null
```

或者，你可以这样做：

```kt
val student=Student()
```

这两种方法都有它们的缺点。第一种方法要求你在使用时检查可空性，而第二种初始化方法将使其不可变。

1.  为了克服限制，我们可以使用 `lateinit` 修饰符，通过它可以先声明后在我们想要的地方（但在第一次访问之前）初始化。这在使用依赖注入时尤其需要。让我们看看 Kotlin 文档中使用的 `lateinit` 修饰符来声明变量的例子：

```kt
public class MyTest {  
    lateinit var subject: TestSubject
    @SetUp fun setup() {  
        subject = TestSubject()  
    }   
    @Test fun test() {  
        subject.method() // dereference directly  
    } 
}
```

1.  如果你尝试在初始化之前访问变量，你会得到 `UninitializedPropertyAccessException`。如果你使用依赖注入，以下是使用 `lateinit` 的方法：

```kt
@Inject
 lateinit var mPresenter:EducationMvpPresenter
```

# 还有更多…

初始化属性的另一种方式是使用 `lazy` 修饰符；`lazy()` 基本上是一个接受 lambda 表达式并返回一个 lazy 实例的函数，该实例作为实现懒加载属性的代理。让我们看看下一个示例：

```kt
public class Student{ 
    val name: String by lazy { 
        “Aanand Shekhar Roy” 
    }
}
```

通过 `lazy` 初始化，我们将初始化推迟到我们第一次使用它的时候。属性仅在第一次访问时初始化，并且对于后续的访问返回相同的值。这就是为什么必须标记变量为不可变的原因。这真的可以帮助我们初始化耗时较多的对象，这些对象需要花费很多时间。懒加载初始化可以提高我们的启动时间。唯一的缺点是，由于它是一个 `val` 属性，你将无法稍后修改它。

# 使用 SharedPreferences

**SharedPreferences** 是 Android 设备上持久化数据存储的一种方式，通常用于以键值对的形式保存数据，例如应用程序的设置。Kotlin 通过其独特的语言结构使与共享偏好设置一起工作变得更加容易。在这个菜谱中，我们将看到 Kotlin 如何帮助我们轻松地处理 SharedPreferences。所以让我们开始吧。

# 准备工作

我们将在这个菜谱中使用 Android Studio 3.0。如果你有更早版本的 Android Studio，要么将其更新到 3.0，要么在其中配置 Kotlin。

# 如何操作...

为了能够定义和使用 SharedPreferences，我们需要遵循特定的步骤。我们将逐一介绍每个步骤，并一起实现：

1.  首先，我们将创建一个 `Prefs` 类，它将作为读取/写入我们应用程序 SharedPreferences 的单一入口。这将使处理所有 SharedPreferences 变得更容易，因为它们都将在一个地方。正如我们所知，共享偏好设置需要上下文存在，所以我们将上下文传递给主构造函数。我们还将创建一个单一的 SharedPreferences 对象，我们将在整个类中使用它：

```kt
class Prefs (mContext:Context){
    val sharedPrefences=mContext.getSharedPreferences("com.ankoexamples.app",Context.MODE_PRIVATE)
    val PREF_USERNAME="pref_username"
}
```

1.  例如，我们定义了一个 `PREF_USERNAME` SharedPreferences；在这里，我们将存储用户的用户名。现在有趣的部分开始了；记住 Kotlin 有一个属性，我们可以显式地定义如何获取和设置该属性。我们在这里也会使用同样的方法。让我们看看给出的代码：

```kt
var username:String
    get() = sharedPrefences.getString(PREF_USERNAME,null)
   set(value)=sharedPrefences.edit().putString(PREF_USERNAME,value).apply()
```

如你所见，在设置器中，我们正在编辑共享偏好设置，在获取器中，我们正在提取共享偏好设置的值。

1.  现在我们已经准备好了 `Prefs` 类，我们可以在我们的活动、片段等中使用它。最好的方法是在 `Application` 类中定义它，并从许多活动或片段中访问它，因为这样我们就不需要创建多个 `Prefs` 类的对象。所以让我们创建一个 `Application` 类和一个 `Prefs` 类的单例实例：

```kt
class App:Application() {
    companion object {
        var prefs: Prefs? = null
    }

    override fun onCreate() {
        prefs = Prefs(this)
        super.onCreate()
    }
}
```

我们将 `prefs` 变量添加到伴生对象中，以便能够静态地使用它。现在，由于我们将它放置在 `Application` 类中，我们只处理 `prefs` 对象的单个实例。

1.  我们还可以使用 `lazy` 构造来确保我们只在第一次访问时创建对象。这样做也有助于我们避免空检查。下面是我们的 `App` 类将如何看起来：

```kt
val prefs: Prefs by lazy {
    App.prefs!!
}
class App:Application() {
    companion object {
        var prefs: Prefs? = null
    }
    override fun onCreate() {
        prefs = Prefs(this)
        super.onCreate()
    }
}
```

1.  现在我们来看一个例子，向我们的 SharedPreferences 添加一个值：

```kt
class MainActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        prefs.username="Aanand"
    }
}
```

1.  现在使用共享偏好设置看起来非常简单，就像我们正在给变量赋值一样。访问它们也非常容易：

```kt
Log.d(prefs.username) // Aanand
```

# 还有更多...

如你所见，我们使用了 `apply()` 方法来保存偏好设置，这立即提交了内存中 SharedPreferences 的更改，同时也启动了对磁盘的异步提交；另一方面，`commit()` 方法则是同步写入持久存储。

# 在 Kotlin 中创建多个 `let` 的链

`let` 是 Kotlin 的 `Standard.kt` 库提供的一个非常有用的函数。它基本上是一个作用域函数，允许你在其作用域内声明变量。让我们看看下面的代码：

```kt
someVariable.let{
    // someVariable is present as "*it"* }
```

然而，最好的是它可以用来避免空检查。之前，你可能使用以下方法：

```kt
if(someVariable!=null){
    // do something 
}
```

虽然前面的代码是好的，但它并不非常适合用于修改属性。另一种选择是使用 `?.let` (`someVariable.?let{}`)，这确保了当变量不为 null 时代码块会运行。然而，如果我们有多个非空链，我们应该如何处理这些情况呢？让我们看看在这个菜谱中如何处理这些情况。

# 准备工作

我们将使用 IntelliJ IDEA 来编写代码。你可以使用任何能够执行 Kotlin 代码的 IDE。

# 如何做...

按照提到的步骤了解如何创建多个 `let` 的链：

1.  当你需要进行多次空检查时，你可以显然使用嵌套的 `if-else`，检查空条件，如下面的代码所示：

```kt
if(variableA!=null){
    if(variableB!=null){
        if(variableC!=null){
            // do something.
        }
    }
}
```

1.  由于我们知道 `let` 函数保证只有在对象不为 null 时才会运行代码块，我们需要创建一个函数来执行 `let` 的功能，但适用于三个变量场景。让我们看看我们的函数：

```kt
fun <T1: Any, T2: Any,T3:Any, R: Any> multiLet(p1: T1?, p2: T2?,p3:T3?, block: (T1, T2,T3)->R?): R? {
    return if (p1 != null && p2 != null &&p3!=null) block(p1, p2,p3) else null
}
```

1.  现在，我们可以像下面这样使用它：

```kt
fun main(args: Array<String>) {
    var variableA="a"
    var variableB="c"
    var variableC="b"
    multiLet(variableA,variableB,variableC){
        _,_,_->
        println("Everything not null")
    }
}

//Output: Everything not null
```

1.  以类似的方式，它也可以用于两个变量场景。你可能想知道如何在多对象场景中实现，比如在列表的情况下。让我们创建一个 `whenAllNotNull` 函数，它只会在列表的所有元素都不为 null 时运行代码块：

```kt
var nonNullList=listOf("a","b","c")
nonNullList.whenAllNotNull {
    println("all not null")
}
fun <T: Any, R: Any> Collection<T?>.whenAllNotNull(block: (List<T>)->R) {
    if (this.all { it != null }) {
        block(this.filterNotNull())
    }
}

Output: all not null
```

# 创建全局变量

在 Java 中，我们只需在类声明的开头定义变量并在之后初始化它，就可以创建一个全局变量。通过仅仅声明它，我们就可以将其用作全局变量。

在这个菜谱中，我们将学习如何在 Kotlin 中创建和使用全局变量。

# 准备工作

我将使用 IntelliJ 进行编码。你可以使用任何可以编写和执行 Kotlin 代码的 IDE。

# 如何做...

现在，让我们看看如何在 Kotlin 中创建全局变量。有两种方法可以实现，让我们逐一来看：

1.  一种实现方式是在类声明下声明它。我们可以使用 `var` 声明，如下所示：

```kt
fun main(args: Array<String>) {
    var student:Student?=null
}
```

然而，这种方法会在每次使用时都进行空值检查：

```kt
println(student?.age)
```

1.  为了防止这种情况，你可以使用 `val` 声明并初始化它，但这将导致变量不可变，这可能不是期望的行为。

1.  另一种声明全局变量的方式是使用 `lateinit` 修饰符。以下是前面代码的修改方式：

```kt
fun main(args: Array<String>) {
    lateinit var student:Student
    student=Student()
    println(student.age)
}
```

1.  `lateinit` 修饰符用于首先声明变量，无需将其定义为 null 或不可变。然而，在使用它之前我们需要对其进行初始化；否则，它将抛出 `UninitializedPropertyAccessException` 异常。

`lateinit` 修饰符不适用于原始类型。

1.  当你尝试使用依赖注入初始化变量时，`lateinit` 也可以很有用。这样，你可以在类体内部引用属性时避免进行空值检查。
