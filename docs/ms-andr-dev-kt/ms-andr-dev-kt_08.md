# 第八章：Android 偏好设置

在上一章中，我们处理了存储在 SQLite 数据库中的复杂数据。这一次，我们将处理一种更简单的数据形式。我们将涵盖一个特定的用例，以演示 Android 共享偏好设置的使用。

假设我们想要记住我们的`ViewPager`类的最后一页位置，并在每次启动应用程序时打开它。我们将使用共享偏好设置来记住它，并在每次视图页面位置更改时持久化该信息，并在需要时检索它。

在这个相当简短的章节中，我们将涵盖以下主题：

+   Android 的偏好设置是什么，你如何使用它们？

+   定义自己的偏好设置管理器

# Android 的偏好设置是什么？

我们应用程序的偏好设置是由 Android 的共享偏好设置机制持久化和检索的。共享偏好设置本身代表 Android 及其 API 访问和修改的 XML 数据。Android 处理有关检索和保存偏好设置的所有工作。它还提供了这些偏好设置为私有的机制，隐藏在公共访问之外。Android SDK 具有一套用于偏好设置管理的优秀类。还有可用的抽象，因此您不仅限于默认的 XML，而可以创建自己的持久化层。

# 你如何使用它们？

要使用共享偏好设置，您必须从当前上下文获取`SharedPreferences`实例：

```kt
    val prefs = ctx.getSharedPreferences(key, mode) 
```

在这里，`key`表示将命名此共享偏好设置实例的`String`。系统中的 XML 文件也将具有该名称。这些是可以从`Context 类`获得的模式（操作模式）：

+   `MODE_PRIVATE`：这是默认模式，创建的文件只能被我们的调用应用程序访问

+   `MODE_WORLD_READABLE`：这已被弃用

+   `MODE_WORLD_WRITEABLE`：这已被弃用

然后，我们可以存储值或按以下方式检索它们：

```kt
    val value = prefs.getString("key", "default value")  
```

所有常见数据类型都有类似的`getter`方法。

# 编辑（存储）偏好设置

我们将通过提供偏好设置编辑的示例来开始本节：

```kt
    preferences.edit().putString("key", "balue").commit() 
```

`commit()`方法立即执行操作，而`apply()`方法在后台执行操作。

如果使用`commit()`方法，永远不要从应用程序的主线程获取或操作共享偏好设置。

确保所有写入和读取都在后台执行。您可以使用`AsyncTask`来实现这一目的，或者使用`apply()`而不是`commit()`。

# 删除偏好设置

删除偏好设置，有一个`remove`方法可用，如下所示：

```kt
    prefs.edit().remove("key").commit() 
```

不要通过用空数据覆盖它们来删除您的偏好设置。例如，用 null 覆盖整数或用空字符串覆盖字符串。

# 定义自己的偏好设置管理器

为了实现本章开头的任务，我们将创建一个适当的机制来获取共享偏好设置。

创建一个名为`preferences`的新包。我们将把所有与`preferences`相关的代码放在该包中。对于共享偏好设置管理，我们将需要以下三个类：

+   `PreferencesProviderAbstract`：这是提供对 SharedPreferences 的访问的基本抽象

+   `PreferencesProvider`：这是`PreferencesProviderAbstract`的实现

+   `PreferencesConfiguration`：这个类负责描述我们尝试实例化的偏好设置

使用这种方法的好处是在我们的应用程序中统一访问共享偏好设置的方法。

让我们定义每个类如下：

+   `PreferencesProviderAbstract`类代码如下：

```kt
         package com.journaler.perferences 

         import android.content.Context 
         import android.content.SharedPreferences 

         abstract class PreferencesProviderAbstract { 
           abstract fun obtain(configuration: PreferencesConfiguration,
           ctx: Context): SharedPreferences 
         } 
```

+   `PreferencesConfiguration`类代码如下：

```kt
         package com.journaler.perferences 
         data class PreferencesConfiguration
         (val key: String, val mode: Int) 
```

+   `PreferencesProvider`类代码如下：

```kt
        package com.journaler.perferences 

        import android.content.Context 
        import android.content.SharedPreferences 

        class PreferencesProvider : PreferencesProviderAbstract() { 
          override fun obtain(configuration: PreferencesConfiguration,
          ctx: Context): SharedPreferences { 
            return ctx.getSharedPreferences(configuration.key,
            configuration.mode) 
          } 
        } 
```

正如你所看到的，我们创建了一个简单的机制来获取共享偏好设置。我们将加以整合。打开`MainActivity`类，并根据以下代码进行扩展：

```kt
     class MainActivity : BaseActivity() { 
       ... 
       private val keyPagePosition = "keyPagePosition" 
       ... 

       override fun onCreate(savedInstanceState: Bundle?) { 
         super.onCreate(savedInstanceState) 

         val provider = PreferencesProvider() 
         val config = PreferencesConfiguration("journaler_prefs",
         Context.MODE_PRIVATE) 
         val preferences = provider.obtain(config, this) 

         pager.adapter = ViewPagerAdapter(supportFragmentManager) 
         pager.addOnPageChangeListener(object :
         ViewPager.OnPageChangeListener { 
            override fun onPageScrollStateChanged(state: Int) { 
                // Ignore 
         } 

         override fun onPageScrolled(position: Int, positionOffset:
         Float, positionOffsetPixels: Int) { 
                // Ignore 
         } 

         override fun onPageSelected(position: Int) { 
           Log.v(tag, "Page [ $position ]") 
           preferences.edit().putInt(keyPagePosition, position).apply() 
         } 
       }) 

       val pagerPosition = preferences.getInt(keyPagePosition, 0) 
       pager.setCurrentItem(pagerPosition, true) 
       ... 
      } 
      ... 
     } 
```

我们创建了`preferences`实例，用于持久化和读取视图页面位置。构建并运行您的应用程序；滑动到其中一个页面，然后关闭您的应用程序并再次运行。如果您查看 Logcat，您将看到类似以下内容的信息（通过`Page`进行过滤）：

```kt
     V/Main activity: Page [ 1 ] 
     V/Main activity: Page [ 2 ] 
     V/Main activity: Page [ 3 ] 
     After we restarted the application: 
     V/Main activity: Page [ 3 ] 
     V/Main activity: Page [ 2 ] 
     V/Main activity: Page [ 1 ] 
     V/Main activity: Page [ 0 ] 
```

我们在关闭后再次打开应用程序，并滑动回索引为`0`的页面。

# 总结

在本章中，您学习了如何使用 Android 共享偏好机制来持久化应用程序偏好设置。正如您所看到的，创建应用程序偏好设置并在应用程序中使用它们非常容易。在下一章中，我们将专注于 Android 中的并发性。我们将学习 Android 提供的机制，并举例说明如何使用它们。
