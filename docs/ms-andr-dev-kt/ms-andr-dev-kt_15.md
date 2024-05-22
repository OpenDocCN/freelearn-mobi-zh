# 第十五章：迁移到 Kotlin

如果您有一个遗留项目或要迁移到 Kotlin 的现有 Java 模块，迁移应该很容易。做到这一点的人已经考虑到了这一点。正如您记得的，Kotlin 是可互操作的。因此，一些模块不需要完全迁移；相反，它们可以包含在 Kotlin 项目中。这取决于您的决定。因此，让我们准备好进行迁移！

在本章中，我们将涵盖以下主题：

+   准备迁移

+   转换类

+   重构和清理

# 准备迁移

正如我们所说，我们需要决定是否完全将我们的模块重写为 Kotlin，还是继续用 Kotlin 编写代码，但保留其在纯 Java 中的遗留。我们会怎么做？在本章中，我们将展示一点点。

在这一点上，我们的当前项目没有任何需要迁移的内容。因此，我们将创建一些代码。如果您没有具有包结构的 Java 源目录，请创建它。现在，添加以下包：

+   `activity`

+   `model`

这些包等同于我们已经在 Kotlin 源代码中拥有的包。在`activity`包中，添加以下类：

+   `MigrationActivity.java`代码如下：

```kt
       package com.journaler.activity; 

       import android.os.Bundle; 
       import android.support.annotation.Nullable; 
       import android.support.v7.app.AppCompatActivity; 

       import com.journaler.R; 

       public class MigrationActivity extends AppCompatActivity { 

        @Override 
        protected void onCreate(@Nullable Bundle savedInstanceState) { 
          super.onCreate(savedInstanceState); 
          setContentView(R.layout.activity_main); 
        } 

        @Override 
        protected void onResume() { 
          super.onResume(); 
        } 
       }
```

+   `MigrationActivity2.java`：确保其实现与`MigrationActivity.java`完全相同。我们只需要一些代码基础来展示和迁移。

在 Android `manifest`文件中注册两个活动，如下所示：

```kt
        <manifest xmlns:android=
        "http://schemas.android.com/apk/res/android" 
        package="com.journaler"> 
        ... 
        <application 
         ... 
        > 
        ... 
         <activity 
            android:name=".activity.MainActivity" 
            android:configChanges="orientation" 
            android:screenOrientation="portrait"> 
            <intent-filter> 
              <action android:name="android.intent.action.MAIN" /> 
              <category android:name=
              "android.intent.category.LAUNCHER" /> 
            </intent-filter> 
         </activity> 

         <activity 
            android:name=".activity.NoteActivity" 
            android:configChanges="orientation" 
            android:screenOrientation="portrait" /> 

         <activity 
            android:name=".activity.TodoActivity" 
            android:configChanges="orientation" 
            android:screenOrientation="portrait" /> 

         <activity 
            android:name=".activity.MigrationActivity" 
            android:configChanges="orientation" 
            android:screenOrientation="portrait" /> 

         <activity 
            android:name=".activity.MigrationActivity2" 
            android:configChanges="orientation" 
            android:screenOrientation="portrait" /> 
        </application> 

      </manifest> 
```

正如您所看到的，Java 代码与 Kotlin 代码一起使用没有任何问题。您的 Android 项目可以同时使用两者！现在，请考虑一下，您是否真的需要进行任何转换，还是您愿意保留现有的 Java 内容？让我们在`model`包中添加类：

+   `Dummy.java`代码如下：

```kt
        package com.journaler.model; 

        public class Dummy { 

          private String title; 
          private String content; 

          public Dummy(String title) { 
            this.title = title; 
          } 

          public Dummy(String title, String content) { 
            this.title = title; 
            this.content = content; 
          } 

          public String getTitle() { 
            return title; 
          } 

          public void setTitle(String title) { 
            this.title = title; 
          } 

          public String getContent() { 
            return content; 
          } 

         public void setContent(String content) { 
           this.content = content; 
         } 

       } 
```

+   `Dummy2.java`代码如下：

```kt
        package com.journaler.model; 

        import android.os.Parcel; 
        import android.os.Parcelable; 

        public class Dummy2 implements Parcelable { 

          private int count; 
          private float result; 

          public Dummy2(int count) { 
            this.count = count; 
            this.result = count * 100; 
         } 

         public Dummy2(Parcel in) { 
           count = in.readInt(); 
           result = in.readFloat(); 
         } 

         public static final Creator<Dummy2>
         CREATOR = new Creator<Dummy2>() { 
           @Override 
           public Dummy2 createFromParcel(Parcel in) { 
             return new Dummy2(in); 
           } 

           @Override 
           public Dummy2[] newArray(int size) { 
             return new Dummy2[size]; 
           } 
         }; 

         @Override 
         public void writeToParcel(Parcel parcel, int i) { 
           parcel.writeInt(count); 
           parcel.writeFloat(result); 
         } 

         @Override 
         public int describeContents() { 
           return 0; 
         } 

         public int getCount() { 
           return count; 
         } 

         public float getResult() { 
           return result; 
         } 
       }
```

让我们再次检查项目的 Kotlin 部分是否看到了这些类。在您的 Kotlin 源目录的根目录中创建一个新的`.kt`文件。让我们称之为`kotlin_calls_java.kt`：

```kt
    package com.journaler 

    import android.content.Context 
    import android.content.Intent 
    import com.journaler.activity.MigrationActivity 
    import com.journaler.model.Dummy2 

    fun kotlinCallsJava(ctx: Context) { 

      /** 
      * We access Java class and instantiate it. 
      */ 
      val dummy = Dummy2(10) 

      /** 
      * We use Android related Java code with no problems as well. 
      */ 
       val intent = Intent(ctx, MigrationActivity::class.java) 
       intent.putExtra("dummy", dummy) 
       ctx.startActivity(intent) 

    } 
```

正如您所看到的，Kotlin 在使用 Java 代码时没有任何问题。因此，如果您仍然希望进行迁移，您可以这样做。没问题。我们将在接下来的章节中这样做。

# 危险信号

将庞大和复杂的 Java 类转换为 Kotlin 仍然是一个可选项。无论如何，提供适当的单元测试或仪器测试，以便在转换后重新测试这些类的功能。如果您的任何测试失败，请仔细检查失败的原因。

您想要迁移的类可以通过以下两种方式进行迁移：

+   自动转换

+   手动重写

在处理庞大和复杂的类时，这两种方法都可能会带来一些缺点。完全自动转换有时会给您带来不太美观的代码。因此，在完成后，您应该重新检查和重新格式化一些内容。第二个选项可能会花费您很多时间。

结论-您始终可以使用原始的 Java 代码。从您将 Kotlin 作为主要语言开始，您可以用 Kotlin 编写所有新的东西。

# 更新依赖关系

如果您将 Android 项目的 100%纯 Java 代码切换到 Kotlin，您必须从头开始。这意味着您的第一个迁移工作将是更新您的依赖关系。您必须更改`build.gradle`配置，以便识别 Kotlin 并使源代码路径可用。我们已经在第一章中解释了如何在*开始 Android*中设置 Gradle 部分; 因此，如果您的项目中没有与 Kotlin 相关的配置，您必须提供它。

让我们回顾一下我们的 Gradle 配置：

+   `build.gradle`根项目代表了主`build.gradle`文件，如下所示：

```kt
        buildscript { 
          repositories { 
            jcenter() 
            mavenCentral() 
          } 
          dependencies { 
            classpath 'com.android.tools.build:gradle:2.3.3' 
            classpath 'org.jetbrains.kotlin:kotlin-gradle-
            plugin:1.1.51' 
          } 
       } 

      repositories { 
       jcenter() 
       mavenCentral() 
      }
```

+   主应用程序`build.gradle`解决了应用程序的所有依赖关系，如下所示：

```kt
        apply plugin: "com.android.application" 
        apply plugin: "kotlin-android" 
        apply plugin: "kotlin-android-extensions" 

        repositories { 
          maven { url "https://maven.google.com" } 
        } 

        android { 
         ... 
         sourceSets { 
          main.java.srcDirs += [ 
                'src/main/kotlin', 
                'src/common/kotlin', 
                'src/debug/kotlin', 
                'src/release/kotlin', 
                'src/staging/kotlin', 
                'src/preproduction/kotlin', 
                'src/debug/java', 
                'src/release/java', 
                'src/staging/java', 
                'src/preproduction/java', 
                'src/testDebug/java', 
                'src/testDebug/kotlin', 
                'src/androidTestDebug/java', 
                'src/androidTestDebug/kotlin' 
           ] 
          } 
          ... 
          } 
         ... 
        } 

        repositories { 
          jcenter() 
          mavenCentral() 
        } 

        dependencies { 
          compile "org.jetbrains.kotlin:kotlin-reflect:1.1.51" 
          compile "org.jetbrains.kotlin:kotlin-stdlib:1.1.51" 
           ... 
          compile "com.github.salomonbrys.kotson:kotson:2.3.0" 
            ... 

          compile "junit:junit:4.12" 
          testCompile "junit:junit:4.12" 

          testCompile "org.jetbrains.kotlin:kotlin-reflect:1.1.51" 
          testCompile "org.jetbrains.kotlin:kotlin-stdlib:1.1.51" 

          compile "org.jetbrains.kotlin:kotlin-test:1.1.51" 
          testCompile "org.jetbrains.kotlin:kotlin-test:1.1.51" 

          compile "org.jetbrains.kotlin:kotlin-test-junit:1.1.51" 
          testCompile "org.jetbrains.kotlin:kotlin-test-junit:1.1.51" 
          ... 
        }
```

这些都是您应该满足的与 Kotlin 相关的依赖关系。其中之一是 Kotson，为`Gson`库提供 Kotlin 绑定。

# 转换类

最后，我们将迁移我们的类。我们有两种自动选项可用。我们将两种都使用。找到`MigrationActivity.java`并打开它。选择代码 | 将 Java 文件转换为`Kotlin`文件。转换需要几秒钟。现在，将文件从`Java`包拖放到`Kotlin`源包中。观察以下源代码：

```kt
    package com.journaler.activity 

    import android.os.Bundle 
    import android.support.v7.app.AppCompatActivity 

    import com.journaler.R 

    class MigrationActivity : AppCompatActivity() { 

      override fun onCreate(savedInstanceState: Bundle?) { 
        super.onCreate(savedInstanceState) 
        setContentView(R.layout.activity_main) 
      } 

      override fun onResume() { 
        super.onResume() 
      } 

    } 
```

正如我们所提到的，完全自动转换并不能得到完美的代码。在下一节中，我们将进行重构和清理。完成相同操作的第二种方法是将 Java 代码复制粘贴到`Kotlin`文件中。从`MigrationActivity2`中复制所有源代码。创建一个同名的新 Kotlin 类并粘贴代码。如果被询问，确认你希望执行自动转换。代码出现后，删除该类的 Java 版本。观察到源代码与迁移后的`MigrationActivity`类相同。

对`Dummy`和`Dummy2`类重复这两种方法。你得到的类将看起来像这样：

+   `Dummy`，第一个`Dummy`类示例：

```kt
       package com.journaler.model 

       class Dummy { 

         var title: String? = null 
         var content: String? = null 

         constructor(title: String) { 
           this.title = title 
         } 

         constructor(title: String, content: String) { 
           this.title = title 
           this.content = content 
        } 

      } 
```

+   `Dummy2`，第二个`Dummy`类示例：

```kt
        package com.journaler.model 

        import android.os.Parcel 
        import android.os.Parcelable 

        class Dummy2 : Parcelable { 

          var count: Int = 0 
          private set 
          var result: Float = 0.toFloat() 
          private set 

          constructor(count: Int) { 
            this.count = count 
            this.result = (count * 100).toFloat() 
          } 

          constructor(`in`: Parcel) { 
            count = `in`.readInt() 
            result = `in`.readFloat() 
          } 

          override fun writeToParcel(parcel: Parcel, i: Int) { 
            parcel.writeInt(count) 
            parcel.writeFloat(result) 
          } 

          override fun describeContents(): Int { 
            return 0 
          } 

         companion object { 

           val CREATOR: Parcelable.Creator<Dummy2>
           = object : Parcelable.Creator<Dummy2> { 
              override fun createFromParcel(`in`: Parcel): Dummy2 { 
                return Dummy2(`in`) 
            } 

           override fun newArray(size: Int): Array<Dummy2> { 
              return arrayOfNulls(size) 
            } 
          } 
        } 

    } 
```

`Dummy2`类在转换时出现了问题。在这种情况下，你必须自己修复它。修复源代码。问题发生在以下行：

```kt
    override fun newArray(size: Int): Array<Dummy2> { ... 
```

通过将类型从`Array<Dummy2> int Array<Dummy2?>`进行切换来修复它，如下所示：

```kt
    override fun newArsray(size: Int): Array<Dummy2?> { ... 
```

简单！

这正是你在进行迁移时可能会面临的挑战！显而易见的是，在`Dummy`和`Dummy2`类中，我们通过切换到 Kotlin 显著减少了代码库。由于不再有 Java 实现，我们可以进行重构和清理。

# 重构和清理

为了在转换后获得最佳可能的代码，我们必须进行重构和清理。我们将调整我们的代码库以符合 Kotlin 的标准和习惯用法。为此，你必须全面阅读它。只有在这样做之后，我们才能认为我们的迁移完成了！

打开你的类并阅读代码。有很多改进的空间！在你做一些工作之后，你应该得到类似这样的结果：

`MigrationActivity`的代码如下：

```kt
    ... 
    override fun onResume() = super.onResume() 
    ... 
```

正如你所看到的，对于`MigrationActivity`（和`MigrationActivity2`）来说，并没有太多的工作。这两个类都非常小。对于`Dummy`和`Dummy2`这样的类，预计需要更大的努力：

+   `Dummy`类的代码如下：

```kt
        package com.journaler.model 

        class Dummy( 
          var title: String, 
          var content: String 
          ) { 

            constructor(title: String) : this(title, "") { 
            this.title = title 
           } 

       } 
```

+   `Dummy2`类的代码如下：

```kt
        package com.journaler.model 

        import android.os.Parcel 
        import android.os.Parcelable 

        class Dummy2( 
          private var count: Int 
        ) : Parcelable { 

          companion object { 
            val CREATOR: Parcelable.Creator<Dummy2> 
            = object : Parcelable.Creator<Dummy2> { 
              override fun createFromParcel(`in`: Parcel): 
              Dummy2 = Dummy2(`in`) 
              override fun newArray(size: Int): Array<Dummy2?> =
              arrayOfNulls(size) 
            }    
          } 

         private var result: Float = (count * 100).toFloat() 

         constructor(`in`: Parcel) : this(`in`.readInt()) 

         override fun writeToParcel(parcel: Parcel, i: Int) { 
           parcel.writeInt(count) 
         } 

         override fun describeContents() = 0 

        } 
```

这两个类版本在重构后与它们最初的 Kotlin 版本相比，现在得到了极大的改进。试着将当前版本与我们最初的 Java 代码进行比较。你觉得呢？

# 总结

在本章中，我们发现了迁移到 Kotlin 编程语言的秘密。我们演示了技术并提供了如何进行迁移以及何时进行迁移的建议。幸运的是，对我们来说，这似乎并不难！下一章将是我们的最后一章，所以，正如你已经知道的，是时候将我们的应用发布到世界上了！
