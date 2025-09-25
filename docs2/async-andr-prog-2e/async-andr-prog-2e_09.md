# 第九章。在本地层上的异步工作

在前面的章节中，我们一直在使用 Android SDK 提供的 Java 线程 API 和并发原语来构建我们的异步结构。Java 线程是我们应用程序中独立执行线路，它自动附加到 Android 虚拟机，并绑定到系统上的一个本地线程。在前面的章节示例中，我们在 JVM 上执行了 Java 编译的字节码，并使用 Java 同步和并发原语来解决正确性和活跃性问题。

在本章中，我们将利用 Java 本地接口（JNI）来执行用 C/C++ 编写的代码并将其编译成本地代码。本地代码直接在硬件上运行并使用本地 CPU **应用程序二进制接口**（**ABI**），由于编译器进行的优化或开发人员使用特定 ABI 技术引入的优化，通常比字节码运行得更快。因此，当我们对设备进行密集计算操作时，这可能是提高应用程序性能和降低功耗的方法。

考虑到这一点，我们将学习如何使用 JNI 接口在本地代码（C/C++）上执行并发任务，从本地层与 Java 代码交互，并从本地代码更新 UI。

之后，我们将学习如何创建本地线程并使用同步原语如 `mutex` 和 `condition` 来避免在设备并行执行多行代码且它们共享相同的内存段时可能出现的任何内存一致性问题时。

最后，我们将启动一组线程在本地层上运行后台工作，并将结果分派到 UI。

在本章中，我们将涵盖：

+   JNI 简介

+   Android NDK

+   从 Java 代码调用 C 函数

+   从 Java 代码调用 C++ 成员/静态函数

+   从本地代码访问 Java 对象

+   在 Java 线程上执行本地后台工作

+   在本地线程上执行异步工作

+   从本地代码与 Java 监视器交互

+   在本地层上处理 Java 异常

# JNI 简介

**JNI** 是一个接口，允许从 **Java 虚拟机**（**JVM**）执行用 C、C++ 或汇编编写的本地代码。该接口严格定义了任何 JNI 实现应该如何行动以管理和控制 Java 代码与机器代码之间的交互。此外，机器代码能够与 JVM 交互并创建对象、执行成员函数、修改成员变量和处理 Java 异常。

允许您在 Java 代码中执行机器代码的 JNI 通常用于：

+   加速应用程序的一些关键部分。由于代码直接在硬件上运行，它可以利用特定的指令集来提高执行效率：

    +   示例：使用 SIMD 指令来加速音频或视频浮点运算。

+   将现有的 C/C++库集成到您的 Android 应用程序中。您可以将任何针对 Android 平台编写的遗留代码或库移植到您的 Android 应用程序中并使用它：

    +   示例：将开源库如`opencv`、`libgdx`和`box2d`集成到您的应用程序运行时。

+   要使用 Java API 无法访问的平台相关特性：

    +   示例：如 poll 和信号量等低级 OS 特性或如 OpenGL、OpenSL ES 或 OpenMAX AL 等本地 API。

### 注意

注意，将 C/C++和 JNI 添加到您的项目中并非免费，通常会增加项目的复杂性，使得调试、构建和测试变得更加困难。因此，在决定在您的应用程序中使用它之前，您必须评估其成本效益。

## Android NDK（本地开发工具包）

为了帮助构建需要 Java 层和本地层之间动态协作的 Android 应用程序，Android 开发者网站上提供了一个名为 Android NDK 的开发工具包([`developer.android.com/ndk/index.html`](http://developer.android.com/ndk/index.html))。

Android NDK 是一个 Android 工具集，允许您将用 C/C++编写的代码编译成 Android 支持的多个 ABIs，它还能够将用 C 或 C++编写的现有库编译到 Android 平台。

在我们更详细地继续之前，您应该在您的开发平台上安装 NDK 包，按照[`developer.android.com/ndk/downloads`](http://developer.android.com/ndk/downloads)上定义的说明进行操作。

在撰写本文时，NDK 的最新版本是 10e，因此我们将以此版本为基础编写代码和示例，本章的其余部分也将以此为基础。

您将在应用程序中编写的 Java 源代码，由 Android SDK 编译，生成 Android 字节码，该字节码将在任何 Android 设备上的 Android JVM 上被解释。

使用用 C 或 C++编写的源代码，NDK 编译器将其转换为具有不同指令集、硬件特性、调用约定和存储约定的 CPU 代码。每种 CPU 架构都有自己的 ABI，它定义了机器代码应该如何排列以与 CPU 硬件交互。

NDK 工具集包含工具，可以抽象这些硬件特性，并为以下 ABIs 生成机器代码：armeabi、armeabi-v7a、arm64-v8a、x86、x86_64、mips 和 mips64。

大多数情况下，您希望支持尽可能多的设备，因此默认情况下，NDK 将为所有支持的 CPU 架构和指令集生成代码。

# 从 Java 代码调用 C 函数

如前所述，强大的 JNI 接口能够管理双向交互，从 Java 到 C 和从 C 到 Java。

声明方法的 Java 类使用关键字 `native` 声明该方法的行为是在本地代码中实现的。与常规 Java 方法一样，JNI `native` 方法能够接收 Java 对象或原始数据类型作为参数，并返回原始数据类型和对象。

让我们看看一个 `native` 方法定义在 Java 类中会是什么样子：

```java
public class MyNativeActivity extends Activity {

  @Override
  protected void onCreate(Bundle savedInstanceState) {
    ...
    cTv.setText(isPrime(2) ? "true" : "false");
  }
  …
  private native boolean isPrime(int number );
}
```

前面的活动将调用本地代码来检查一个数字是否为质数，并在 UI 上打印结果。

注意，我们的函数接收一个原始数据作为参数，并返回一个原始 `boolean` 类型的结果，并且没有函数体，就像一个抽象函数一样。

当成员函数声明为 native 时，我们通知编译器该函数将在 C/C++ 中实现，在一个在运行时动态加载的本地库中。

现在成员函数已经在 Java 侧声明为 native，让我们使用 `javah` 在本地代码中声明和实现我们的本地方法。

`javah` 能够帮助开发者生成使用 JNI 接口命名的约定来生成本地方法原型，SDK 成为一个方便的工具，可以为您所有的类生成本地方法的头文件。

要使用它，请前往您的项目目录，创建一个 `jni` 目录，并运行下一个 `javah` 以生成您本地函数的头文件。在 Android Studio IDE 中，打开一个终端窗口并转到 `src/main` 目录：

```java
javah  -d jni -classpath <sdk_direcory>/android.jar:../../build/intermediates/classes/debug/  com.packpublishing.asynchronousandroid.chapter9.MyNativeActivity
```

如果一切按预期进行，文件 `com_..._chapter9_MyNativeActivity.h` 将会生成我们的 `native` 方法声明：

```java
JNIEXPORT jboolean JNICALL Java_com_packpublishing_asynchronousandroid_chapter9_MyNativeActivity_isPrime
  (JNIEnv *, jobject, jint);
```

`native` 方法将接收指向 JVM 环境的 `JNIEnv*` 指针，一个指向实际调用该方法的 Java 对象实例的 `jobject` 引用，以及一个整型参数。

在以下 JNI 规范中声明的先前方法，应在您的代码中声明和实现，并在运行时通过共享库加载。

现在我们已经声明了方法，让我们在 `jni` 目录下创建一个 `source` 文件，名为 `c_functions.c`，其中包含 `isPrime` 函数的 `native` 方法实现：

```java
#include "com_packpublishing_asynchronousandroid_chapter9_MyNativeActivity.h"

#ifdef __cplusplus
extern "C" {
#endif

jboolean  Java_com_packpublishing_asynchronousandroid_chapter9_MyNativeActivity_isPrime( JNIEnv *env, jobject obj, jint number) {
    int c;
    for (c = 2; c < number ; c++) {
        if (number % c == 0)
            return JNI_FALSE;
    }
    return JNI_TRUE;
}
#ifdef __cplusplus
}
#endif
```

当调用 `MyNativeActivity.isPrime` 时，JNI 接口将处理过程透明地转发到本地代码函数，传递一个本地整型原始数据 (`jint`)。Android JNI 实现自动将 Java 类型 `int` 的值转换为本地类型 (`jint`)，执行本地函数，并在最后返回由 JNI 接口自动转换为 Java 原始数据 `boolean` 的 `jboolean`。

以下表格显示了 Java 类型如何映射到本地类型：

| Java 类型 | 本地类型 | 描述 |
| --- | --- | --- |
| `boolean` | `Jboolean` | 无符号 8 位 |
| `byte` | `Jbyte` | 有符号 8 位 |
| `char` | `jchar` | 无符号 16 位 |
| `short` | `jshort` | 有符号 16 位 |
| `int` | `jint` | 有符号 32 位 |
| `long long` | `jlong` | 有符号 64 位 |
| `float` | `jfloat` | 32 位浮点数 |
| `double` | `jdouble` | 64 位浮点数 |
| `Object` | `jobject` | 任何 Java 对象 |
| `Class` | `class` | 类对象 |
| `String` | `jstring` | 字符串对象 |
| `void` | `void` |   |

虽然我们在源文件中声明并实现了本地函数，但 JVM 不会找到该方法，直到我们加载包含我们的本地函数的共享库。首先，我们将在项目的根目录 `local.properties` 文件中定义 `ndk` 文件夹：

```java
ndk.dir=<Path to downloaded NDK package>/android-ndk-r10e
```

接下来，在我们的 `build.gradle` 模块配置中，我们将在 `ndk` 配置部分下定义共享库的名称：

```java
apply plugin: 'com.android.library'

android {
    defaultConfig {
        minSdkVersion 9
     ...
        ndk { moduleName "mylib" } 
    } 
}
```

最后，Android Studio 能够编译 `c_functions.c` 并为构建目录中支持的构建目标生成名为 `mylib` 的共享库：

```java
├── lib
│   ├── arm64-v8a
│   │   └── libmylib.so
│   ├── armeabi
│   │   └── libmylib.so
│   ├── armeabi-v7a
│   │   └── libmylib.so
│   ├── mips
│   │   └── libmylib.so
│   ├── mips64
│   │   └── libmylib.so
│   ├── x86
│   │   └── libmylib.so
│   └── x86_64
│       └── libmylib.so
```

这些库将被打包在一个通用的 `apk` 文件中，以便我们的 Android 应用程序加载。

剩下的工作就是在我们的代码中使用它之前，在我们的运行时加载库。要在 JVM 运行时加载共享库，`java.lang.System` 类提供了一个 `static` 方法来加载共享库及其依赖项，因此在我们使用它之前，我们将在我们的 `Activity` 类中添加一个静态部分，以便在类加载器加载我们的类时立即加载库：

```java
public class MyNativeActivity extends Activity {
   …
   static {
      System.loadLibrary("mylib");
   }
}
```

当需要库时，`System` 类会自动检测设备正在运行的 ABI 并加载所需的平台相关库。因此，如果您正在 x86 设备上运行，将加载 `x86/libmylib.so`。

# 从本地代码调用 C++ 函数

到目前为止，我们已经调用了一个在 `c_functions.c` 源文件中实现的 C 函数，因此，在下一节中，我们将向您展示如何调用 C++ 成员类。

首先，我们将向 `MyNativeActivty` 添加一个名为 `isPrimeCPlusPlus` 的本地方法，该方法返回 `String` 类型的结果。让我们看看本地函数声明将如何看起来：

```java
public class MyNativeActivity extends Activity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
      ...
      TextView cPlusTv = (TextView)  
          findViewById(R.id.helloFromCPlusPlus);
      cPlusTv.setText(isPrimeCPlusPlus(4));
    }

    public native String isPrimeCPlusPlus(int number);
}
```

对新的 `MyNativeActivity` 类定义运行 `javah` 工具将生成一个具有以下签名的新的函数声明：

```java
JNIEXPORT jstring JNICALL Java_com_packpublishing_asynchronousandroid_chapter9_MyNativeActivity_isPrimeCPlusPlus(JNIEnv *, jobject, jint);
```

接下来，我们将在名为 `mylib.cpp` 的 C++ 源文件中实现素数函数，作为一个 `class static` 函数：

```java
#include "com_packpublishing_asynchronousandroid_chapter9_MyNativeActivity.h"

class Math {
  public:
  static int isPrime(int number) { ...// Elided for brevity}
};

#ifdef __cplusplus
extern "C" {
#endif

jstring  Java_com_packpublishing_asynchronousandroid_chapter9_MyNativeActivity_isPrimeCPlusPlus
(JNIEnv * env, jobject obj, jint number) {

  return (env)->NewStringUTF(
     Math::isPrime(number) ? "Is Prime" : "Is not Prime");
}

#ifdef __cplusplus
}
#endif
```

如果您在 Android Studio 中构建项目，`mylib.cpp` 源文件将被检测到，并将新的函数和类添加到 `libmylib.so` 共享库文件中。

一旦我们运行应用程序，C++ 系统默认库将加载一个最小的系统 C++ 运行时。默认的 C++ 运行时不提供 C++ 标准库、异常支持和 **运行时类型信息**（**RTTI**）。因此，如果您想使用 C++ 标准库的字符串类、容器、流和通用算法，您必须在加载库之前显式加载所需的 C++ 运行时。有关 Android 上可用的 C++ 运行时的完整和最新比较，请访问 Android 开发者网站上的 C++ Runtimes。

如果我们想使用不同于系统运行时的 C++ 运行时，我们必须在模块的 `build.gradle` 文件中显式设置运行时：

```java
  ndk {
    moduleName "mylib"
    stl "c++_shared"
  }
```

此外，在我们加载我们的库或任何依赖于它的库之前，我们必须先加载非默认的 C++ 运行时库：

```java
public class MyNativeActivity extends Activity {
  static {
    System.loadLibrary("c++_shared");
    System.loadLibrary("mylib");
  }
}
```

由于 `c++_shared` 提供了完整的 STL 库实现，从现在起我们将使用此运行时作为代码示例的基础 C++ 运行时。

太好了！到目前为止，我们已经学习了如何使用 JNI 接口与原生方法交互，所以我们的下一步是学习如何从原生代码访问 Java 对象。

# 从原生代码访问 Java 对象

当我们调用原生函数时，C 或 C++ 函数接收一个指向 JNI 方法表的 `JNIEnv` 指针，这些 JNI 方法用于与 JVM 运行时交互。`JNIEnv` 指针为我们提供了一组原始数据，可用于查找 Java 类定义、设置或获取 Java 对象字段值、调用静态或成员 Java 对象函数、创建 Java 对象、与 Java 监视器交互或处理异常。

我们的下一个示例将在原生函数中计算 UI 小部件 `EditText` 上的单词数量，并使用原生代码中的计数结果更新 `TextView` 文本。因此，我们将学习如何使用 `JNIEnv` 访问成员 Java 对象字段，以及如何使用 `JNIEnv` 接口调用 Java 对象方法（`TextView.setText`）。

让我们首先定义我们的原生函数，并在 `EditField` 内容更改时调用它：

```java
public class MyNativeActivity extends Activity {

  protected EditText inputTextEt = null;
  protected TextView charCountTv = null;

  @Override
  protected void onCreate(Bundle savedInstanceState) {

     // Reference stored as member fields for native access	
    inputTextEt = (EditText) findViewById(R.id.inputText);
    charCountTv = (TextView) findViewById(R.id.charCount);

    // Called every time the code changes
    inputTextEt.addTextChangedListener(new TextWatcher() {
      @Override
      public void onTextChanged(CharSequence s, int start, 
                                int before, int count) {
        updateWordCounter(s.toString());
      }
      ...
    });
  }
  // Native function that calculates the number of words 
  // in a string
  private native void updateWordCounter(String s);
}
```

注意，已向 `Activity` 添加了新函数，请确保您运行 `javah` 以生成新的原生函数声明。

接下来，我们将定义一个 JNI 原生函数，用于计算字符串中的单词数量：

```java
class Util {
  public:
  static int countWords(const std::string &strString) { 
    ... 
  };
}

Void Java_com_packpublishing_asynchronousandroid_chapter9_MyNativeActivity_updateWordCounter(JNIEnv *env, jobject obj, jstring text) {

    std::string content(env->GetStringUTFChars(text, 0));

    size_t word_cnt= Util::countWords(content);

    // Update the TexView with word_cnt integer

}
```

我们留下了原生实现，因为我们将会逐步实现它。作为第一步，我们将从 `charCountTv` 对象字段获取用于在 UI 文本输入中显示单词数量的 `TextView` 对象实例。

要访问 Java 对象字段或方法，始终需要一个 `jmethodID` 或 `jfieldID`：

```java
jmethodID   GetMethodID(JNIEnv* env, jclass clazz, 
                        const char*name, 
                        const char* methodSignature);

jfieldID    GetFieldID(JNIEnv*, jclass clazz, 
                       const char*name, 
                       const char* fieldTypeCode);
```

为了构建 `methodSignature` 或 `fieldTypeCode`（TC），我们必须使用以下表格将 Java 类型映射到类型代码：

| Java 类型 | 类型代码 (TC) |
| --- | --- |
| `boolean` | `Z` |
| `Byte` | `B` |
| `Char` | `C` |
| `double` | `D` |
| `float` | `F` |
| `Int` | `I` |
| `Long` | `J` |
| `short` | `S` |
| `Object` | `L<package>;` |
| `Void` | `V` |

在转换数组时，始终在类型代码前加上 `[` 字符。

要创建一个 `jfieldID`，我们需要一个单独的类型代码。然而，要构建方法签名，我们使用以下格式：

```java
 (<Argument 1 TC ><Argument N TC>) <Return TC>
```

让我们看看如何使用以下说明中的指令在原生代码中获取 `charCountTv TextView` 对象：

```java
// 1\. Obtain a reference to the MyNativeActivity class definiton
jclass activityClass = env->GetObjectClass(obj);

// 2\. Get the fieldId for the charCountTv TextView 
   jfieldID charCountFId = env->GetFieldID(activityClass, "charCountTv", "Landroid/widget/TextView;");

// 3\. Retrieve the object using the object and the jfieldID
jobject tvObj = env->GetObjectField(obj,charCountFId); 
```

一旦我们有了 `TextView` 引用，我们就可以调用 `setText(CharSequence)` 实例方法来发布找到的单词数量。要调用 Java 方法，我们将使用 JNI 函数 `CallVoidMethod`，该函数由方法签名创建的 `jmethodId`：

```java
void CallVoidMethod(JNIEnv *env, jclass clazz,
                    jmethodID methodID, ...);
```

让我们看看更新`TextView charCountTv`以显示单词数的原生代码将如何看起来：

```java
// 1\. Get the TextView class definition 
jclass textViewClass = env->GetObjectClass(tvObj);

// 2\. Get the methodId for the TextView.setText function
jmethodID setTextMId = env->GetMethodID(
       textViewClass, "setText", "(Ljava/lang/CharSequence;)V");

// 3\. Invoke the SetText instance function
   env->CallVoidMethod(
     tvObj,setTextMId,env->NewStringUTF(wordCountStr);
```

要调用`static`方法和具有不同结果类型的方法，JNI 接口为我们提供了一组具有以下签名的函数：

```java
// To invoke a Class static method that returns a Java Type
<NativeType> CallStatic<Type>Method(
   JNIEnv *env, jobject obj, jmethodID methodID, ...);

// To invoke a method that returns a Java <Type>  
<NativeType> Call<Type>Method(
   JNIEnv *env, jobject obj, jmethodID methodID, ...);

// To invoke a method that returns a Java <Type[]>  
<NativeArrayType> Call<Type>MethodA(
  JNIEnv *env, jobject obj, jmethodID methodID, ...);
```

现在我们已经了解了如何使用 JNI 接口调用原生函数的基本知识，我们就可以开始使用 JNI 在原生代码上执行异步工作了。

# 在 Java 线程上执行原生后台工作

在前面的章节中，我们使用了 JNI 接口在主线程上执行原生函数。由于它们在主线程上运行，函数能够更新 UI，访问`Activity`实例字段，或者直接更新任何 UI 小部件。

然而，正如我们之前讨论的那样，对于长时间计算或密集型任务，我们必须在后台线程上执行它们。

在前面的章节中，我们学习了如何使用`AsyncTask`、`Loader`、`Handler`和远程服务在后台线程上执行工作，这些线程不会降低 UI 响应性或干扰 UI 渲染。

在这些任何 Android 特定构造中，后台线程已经附加到 JVM 上。因此，后台线程已经拥有访问一个准备好使用的 JNI 环境的权限。

在我们的下一个示例中，我们将使用`Loader`构造函数并构建`AsyncTaskLoader`，在后台加载图像，在原生代码中将图像转换为灰度，并在 UI 屏幕上发布结果。

首先，在我们深入探讨原生函数的细节之前，我们将详细说明`Loader` Java 类定义将如何看起来：

```java
public class GrayImageLoader extends AsyncTaskLoader<Result<Bitmap>> {

  final String fileName;
  Bitmap grayImage;

  public ToGrayImageLoader(Context ctx, String fileName) {
    super(ctx);
    this.fileName =  fileName;
  }

  @Override
  public Result<Bitmap> loadInBackground() {
    Result<Bitmap> result = new Result<Bitmap>();
    try {
      BitmapFactory.Options options = new BitmapFactory.Options();
      options.inPreferredConfig = Bitmap.Config.ARGB_8888;

      // Build a RGBA 8888 Bitmap to represent the image 
      Bitmap b = BitmapFactory.decodeFile(this.fileName, options);

      // Convert the Image to Gray scale on Native code
      Bitmap originalImage = BitmapFactory.decodeStream(
        getContext().getAssets().open(fileName));

 	  // Fill the result with the Gray Image
      result.obj = convertImageToGray(originalImage);
    } catch (Exception e) {
      result.error = e;
    }
    return result;
  }

  private native Bitmap convertImageToGray(Bitmap original);
  …
}
```

注意，我们的`Loader`将加载一个被通用`Result`类包裹的`Bitmap`图像，就像我们在前面的章节中所做的那样。当 Java 或原生代码发生任何异常时，`Result.error`将被填充，使得`Loader`消费者能够检测到错误并相应地做出反应。

我们的加载器将接收一个参数，即从资源加载的图像文件名，并将图像解码为`ARGB_8888`格式的`Bitmap`对象，并返回一个灰度图像

当原生函数在后台线程上无错误执行时，`Result<Bitmap>`对象将被传递到 UI 线程上的`Loader`消费者，以便更新到设备屏幕。

对我们的新`AsynTaskLoader`类执行`javah`应该生成具有以下函数签名的`com_packpublishing_asynchronousandroid_chapter9_ToGrayImageLoader.h`头文件：

```java
JNIEXPORT jobject JNICALL Java_com_packpublishing_asynchronousandroid_chapter9_ToGrayImageLoader_convertImageToGray(JNIEnv * env, jobject loader, jobject bitmap);
```

要在原生层处理`Bitmap`对象，需要`jnigraphics`共享库。因此，让我们更新我们的`gradle`构建配置，将我们的库与`jnigraphics`共享库链接：

```java
ndk {
    moduleName "mylib"
    stl "stlport_shared"
    ldLibs "jnigraphics", "log"
}
```

将我们的库`mylib`与`jnigraphics`链接将强制动态加载器在每次通过`System.loadLibrary`加载我们的库时加载`jnigraphics`库。除此之外，`gradle`系统将为**应用程序包文件**（**APK**）中所需的几个 ABIs 打包`jnigraphics`共享库。

现在我们已经在头文件中定义了方法，是时候实现将原始`Bitmap`转换为灰度`Bitmap`的原生函数了。

首先，让我们创建包含`jni`方法定义和所需的`jnigraphics`头文件的源文件`image.cpp`：

```java
#include "com_packpublishing_asynchronousandroid_chapter9_ToGrayImageLoader.h"
#include <android/bitmap.h>
```

接下来，我们将实现将原始像素转换为灰度像素的功能：

```java
jobject Java_com_packpublishing_asynchronousandroid_chapter9_ToGrayImageLoader_convertImageToGray(JNIEnv * env, jobject obj, jobject bitmap) {

  AndroidBitmapInfo info; // Image Information
  void * pixels; // Pixel Matrix
  int ret; // Jni Graphics operation result code

   // Reads the Image width, height, format,...
  if ( (ret = AndroidBitmap_getInfo(env, bitmap, & info)) < 0) {
    jclass clazz = env->FindClass("java/lang/RuntimeException");
    env->ThrowNew(clazz, "Failed to get bitmap info");
    return 0;
  }
  // Loads the bitmap pixel matrix on pixels pointer
  if ((ret = AndroidBitmap_lockPixels(env,bitmap,(void **)& pixels)) < 0) {
    // Exception Generation Elided for brevity
  }
  // Convert each pixel to gray
  ...
  AndroidBitmap_unlockPixels(env, bitmap);
  return bitmap;
}
```

在`jnigraphics`库的帮助下，我们可以使用`AndroidBitmap_getInfo`读取图像信息，如果一切顺利，图像信息将被存储在局部变量`info`中以供进一步使用。

然而，如果`AndroidBitmap_getInfo`失败，我们将在 JVM 中抛出异常并立即从原生函数返回，因为我们调用了`return`。在正常情况下，如果我们使用`ThrowNew`在 JVM 中抛出异常，原生函数不会停止并将控制权传递给异常处理器。因此，如果在原生代码调用期间抛出异常，当函数返回时，JNI 接口将检测到它并将执行权传递给异常处理器。

在我们的例子中，我们使用从`Findclass` JNI 函数获得的`jclass`生成`RuntimeException`。

当我们完成位图处理时，我们通过`AndroidBitmap_unlockPixels`解锁像素，并将`Bitmap` `jobject`返回给最初从后台线程调用原生方法的`loadInBackground`函数。

如您所知，处理过的位图将由`AsyncTaskLoader`在 UI 线程中交付，并可用于更新屏幕上显示图像的`ImageView`或其他类型的 UI 小部件。让我们看看`LoaderCallback.onLoadFinished`回调可能的样子：

```java
@Override
public void onLoadFinished(Loader<Result<Bitmap>> loader, Result<Bitmap> data) {
  if ( data.obj != null ) {
    ImageView iv = (ImageView)findViewById(R.id.grayImage);
    iv.setImageBitmap(data.obj);
  } else {
    Log.e("<TAG>", data.error.getMessage(), data.error);
  }
}
```

在这个简单的例子中，我们能够在`AsyncTaskLoader`的帮助下以机器码的形式执行异步工作，尽管类似的程序也可以通过`AsyncTask`子类、普通线程或甚至`HandlerThread`来完成。这类 Android 构造使用由 Android JVM 管理的 Java 后台线程，因此，不需要显式地将这些线程附加到 JVM，因为它们是 JVM 系统的一部分，并且有自己的`JNIEnv`。

在下一章中，我们将学习如何创建纯原生线程，并使用它们以一致和可靠的方式为我们的 Android 应用程序执行后台工作。

# 在原生线程上执行异步工作

Android NDK 包含 POSIX 线程 C API，它提供了一个创建和销毁本地线程、本地互斥同步原语（如命名互斥锁）和条件变量的 API，类似于 Java 监视器，允许线程等待资源状态的变化。除了这个全局 API 之外，开发者还可以访问在 `clang` 和 `gnu_stl` C++ 运行时上可用的更高层次的 C++11 线程 API。

由于这两个框架都提供了相同类型的并发功能，我们将使用 C++11 线程框架，因为它简单且与 Java Thread API 相似。

首先，让我们更新我们的 `ndk build.gradle` 以使用支持我们将在后续代码示例中使用的线程 API 的 clang C++ 运行时：

```java
ndk {
  moduleName "mylib"
  stl "c++_shared"
  cppFlags.add("-frtti")
  cppFlags.add("-exceptions)
}
```

## 将本地线程附加到 JVM 和从 JVM 中分离

为了与我们的 JVM 交互并为我们执行并发后台工作，本地线程应该附加到当前虚拟机并构建自己的 `JNIEnv`。

线程 `JNIEnv` 与特定的本地线程绑定，不能与其他线程共享，因为它管理自己的引用和本地线程环境。

为了更实际地向您展示，在接下来的几段中，我们将构建一个代码示例，该示例创建附加到 JVM 的本地线程，在后台执行工作并与 UI 线程交互，使用众所周知的 Android 处理器结构发布 `keep-alive` 消息。

要将任何线程附加到 JVM，我们需要访问全局虚拟机结构，`JavaVM`：

```java
jint AttachCurrentThread(JavaVM *vm, void **p_env, void *thr_args) 
```

一个获取 JVM 结构的好方法是从 `JNI_OnLoad` 函数中检索，这是一个当我们的库被 JavaVM 加载时自动调用的函数。当 JNI 接口回调被调用时，我们将保存 JavaVM 引用以供将来使用：

```java
// Java VM Global Pointer
static JavaVM* gVm = NULL;
jint JNI_OnLoad (JavaVM* vm, void* reserved) {
    gVm = vm;
    return JNI_VERSION_1_4;
}
```

随着 JVM 全局指针准备就绪并可供使用，我们能够将任何本地线程附加到应用程序 JVM 并开始与 `JNIEnv` 交互。

作为起点，我们将创建一个高级 C++ 类，该类会自动附加到 JVM，并在实例被销毁时从 JVM 中分离。这个类将作为我们线程示例的基类，提供一个用于原生线程创建的通用抽象接口。

让我们看看 `JavaThread` 类定义将如何看起来：

```java
#include <thread> // including the C++11 Thread Header
#include <jni.h>  // JNI Header
class JavaThread: public std::thread {
public:	 
  JavaThread();
  Void join(); // Wait for the thread to finish
  void entryPoint();
  void start(); 
  void stop();
protected:
   // Method that subclass should implement to define
   // the unit of work 
   virtual void run() = 0;    	 
   virtual void onDetach() {};JNIEnv* threadEnv = NULL; // Thread specific JNI Environment
   std::thread thread_; // C++11 Thread 
   // is Thread attached to JVM
bool isStarted = false;
   std::condition_variable startCond;
   std::mutex startMutex;
   volatile bool shouldStop = false;
   std::condition_variable stopCond;
   std::mutex stopMutex;
};
```

在这个类头文件中，我们从原始的 C++ 线程类中继承 `JavaThread` 并定义抽象方法 `run`。任何工作线程都可以继承 `JavaThread`，提供自己的 `run` 方法实现。此外，特定于 JNI 接口环境的受保护线程存储在 `threadEnv` 中，供线程子类将来使用。

此外，我们将向您介绍从线程头文件中可用的 C++同步原语。`std:mutex`是一个互斥原语，它一次只允许一个线程进入受保护的临界区。如果一个线程正在执行临界代码，另一个尝试进入临界区的线程将阻塞执行，直到执行临界代码的线程释放锁。以下是一个简单的示例：

```java
 // Thread waits for his turn
 mutex.lock();
 ...// Only one thread enters on this section
 mutex.unlock();
```

在 Java 中，可以通过在 Java 块或同步块中使用`synchronized`关键字实现相同的行为。

类似于 Java 监视器的条件并发原语，可以用来阻塞一个线程或一组线程的执行，直到另一个线程修改共享信息并发送信号通知等待的线程。

现在我们知道了这些 C++并发原语用于什么，让我们实现自动附加和从 JVM 中分离的`JavaThread`。

首先，我们将通过传递`entryPoint`函数作为线程的运行时函数，在原生的`start()`方法中启动背景原生线程：

```java
void JavaThread::start() {
    thread_ = std::thread(&JavaThread::entryPoint, this);
    std::unique_lock<std::mutex> lck(startMutex);
    // wait until the Thread is attached to JVM
    while (!isStarted) startCond.wait(lck);
}
```

当`std::thread`在系统中创建线程时，它将调用我们的对象中的`entrypoint`函数来初始化 JNI 环境。

同时，我们将阻塞调用线程，直到新线程附加到 JVM 并发送信号到`startCond`条件变量。接下来，当`std::thread`在操作系统中初始化新线程时，它将执行控制权更改为构造函数中指定的成员函数，即`JavaThread::entryPoint`。在这个函数中，我们将原生线程附加到 JVM，并将执行调度到子类`run()`方法。

让我们看看我们如何实现`entryPoint`函数：

```java
void JavaThread::entryPoint() {

  // Attach current thread to Java virtual machine
  // and obrain JNIEnv interface pointer
  { 
    // Acquires the start Mutex to access the conditional variable
    std::unique_lock<std::mutex> lck(startMutex);
    // Ataches the current thread to the JVM 
    // and caches the JNIEnv 
    if ( gVm->AttachCurrentThread(&threadEnv, NULL) != 0) {
      ..// Handle the error 
    }
    isStarted = true;  // Changes the shared variable
    startCond.notify_all(); // Notify the thread constructor
  }
  onAttach();
  try {
    // Run the subclass method
    run();
  } catch (...) {
    // Detach current thread when an exception happens
    onDetach();
    gVm->DetachCurrentThread();
    throw;
  }
  // Detach current thread from Java virtual machine
  onDetach(); 
  gVm->DetachCurrentThread();
}
```

注意，即使在`run`执行期间抛出运行时异常，线程也会从 JVM 中分离出来。当线程从 JVM 中分离出来时，所有线程监视器都会释放，并且所有等待此线程的 Java 线程都会收到通知。

对于停止机制，我们将使用一个`boolean`变量和一个`condition`变量来通知`shouldStop`条件已更改。稍后，我们的`JavaThread`子类将利用此机制来停止`run()`执行。

让我们看看`stop`方法将如何看起来：

```java
void JavaThread::stop() {
    // Acquire the stop mutex 
    std::unique_lock<std::mutex> lck(stopMutex);    
    // Change the should stop condition 
this->shouldStop =true;    
    // Notify any thread waiting for this signal that shouldstop
    // condition has changed.
    stopCond.notify_all();
}
```

在完全定义了原生线程基类之后，我们现在可以创建我们的派生类，该类在`run`方法中实现所需的行为。

如前所述，我们将使用`Handler`构造来提交从后台线程到 UI 线程的消息。由于它们在同一个进程中运行并共享相同的内存，我们可以安全地将`Handler`对象的引用传递给原生后台线程。

首先，在我们开始实现我们的`JavaThread`子类之前，我们将编写`NativeThreadsActivity`并实现一个`Handler`匿名子类来接收原生线程的消息：

```java
public class NativeThreadsActivity extends Activity {  
  public static final int HEALTHCHECK = 0; // Handler Message Code
  ...
  // Process the Message sent by the native threads
  Handler myHandler = new Handler() {
    public void handleMessage(Message msg) {
      switch (msg.what) {
      case HEALTHCHECK:
        TextView tv = (TextView)findViewById(R.id.console);
        tv.setText((String) msg.obj + tv.getText());
        break;
      }
    }
  };
  // Start the Native Threads when the start button is clicked
  public native void startNativeThreads(Handler handler);
  // Stop The Native Threads when the stop button is clicked
  public native void stopNativeThreads();
}
Once our handler receives a message with the code HEALTHCHECK it will prepend the String received on the msg.obg to a TextView on the Activity UI screen. 
```

此类还将负责在每次点击**启动**和**停止**按钮时启动和停止本地线程。

代码示例中省略了启动和停止按钮的设置。然而，**启动**按钮将调用 `startNativeThreads` 的本地函数，并将 `myHandler` 作为 `Handler` 参数传递，而**停止**按钮将调用 `stopNativeThreads` 来停止本地线程的执行。此外，我们还可以在 `Activity.onStop` 中调用 `stopNativeThreads` 来在活动被销毁时停止线程。

现在我们需要实现一个将在后台运行的 `JavaThread`，并通过 `handler` 对象向 UI 线程提交一个 `healthcheck` 消息。由于 `handler` 来自不同的 `jniEnv`，首先需要从原始 `handler` 创建一个 JNI 全局引用。让我们先从实现构造函数开始，该构造函数创建一个全局的 `Handler` 对象引用并将引用存储在一个 `member` 变量中：

```java
class HealthCheckThread: public JavaThread {

     jobject handlerObj; // Cache the Global Reference  
public:
    HealthCheckThread(JNIEnv *env_,jobject handlerObj_):
      JavaThread(), 
      // Use the main threadJNIEnv to create a global ref
      handlerObj(env_->NewGlobalRef(handlerObj_)) {} 
     }
```

在构造函数中，我们从主线程接收了对象引用，调用了我们的 `JavaThread` 默认构造函数，并创建了一个全局引用来存储原始引用。

## JNI 引用解释

理解 JNI 引用如何被 JVM 管理非常重要，因为如果我们不正确使用它们，我们可能会使应用程序崩溃或引入内存泄漏。内存泄漏会影响应用程序性能，增加电池消耗，并在长期内导致应用程序因 `java.lang.OutOfMemoryError` 异常而崩溃。如您所知，JVM 垃圾收集器 (GC) 管理应用程序内存使用，在对象不再使用时清理对象。当一个对象在内存中没有引用时，该对象被认为是垃圾收集的候选对象，因此，当 GC 发现一个未引用的对象时，它将释放该对象从内存中。

JNI 支持三种类型的引用：

+   **局部引用** – 附属于线程、`JNIEnv` 的引用，其生命周期有效于本地方法的持续时间。该引用传递给本地方法，并在方法返回时销毁。用户也可以在本地方法中创建和删除局部引用，以防止任何对象被垃圾回收。请注意，局部引用在其创建的 `JNIEnv` 中有效。以下 JNI 函数可用于管理局部引用：

    ```java
    Jobject NewLocalRef(jobject);
    void DeleteLocalRef (jobject);
    ```

    +   JVM 提供了一个函数来在当前 JNI 帧中分配空间以存储局部引用。默认情况下，它具有 16 个引用的容量：

        ```java
        jint EnsureLocalCapacity(jint);
        ```

+   **全局引用** – 用于保持全局对象无限期存活期的引用。这类引用可以在线程`JNIEnv`对象之间共享。在不再需要时，显式地从 JVM 删除引用是至关重要的。当你不释放系统中的引用时，你正在应用程序中创建内存泄漏。请注意，当你从系统中释放引用时，该引用就不再有效，因此如果你尝试使用它，JNI 接口将抛出异常。以下 JNI 函数用于管理局部引用：

    ```java
    jobject NewGlobalRef (jobject);
    void DeleteGlobalRef(jobject);
    ```

+   **弱全局引用** – 与全局引用类似，但它不会阻止对象在它是对象唯一活动引用时被垃圾回收。JNI 中的弱全局引用是 Java 弱引用的简化版本：

    ```java
    jweak NewWeakGlobalRef(JNIEnv *env, jobject obj);
    void DeleteWeakGlobalRef(JNIEnv *env, jweak obj);
    ```

## 从本地线程与 UI 交互

由于我们希望缓存一个`Handler`对象的引用，该对象在`startNativeThreads`执行期间仍然存活，因此在将其保存到成员变量之前创建一个全局引用是有意义的。该引用将在我们的后台线程中用于向 UI 提交消息。

由于我们在`HealthCheckThread`类中创建了一个全局引用，为了在 JVM 中释放引用并避免任何内存损失，我们将在线程停止期间调用的`HealthCheckThread.onDetach()`函数中删除全局引用：

```java
class HealthCheckThread: public JavaThread {
    ...
    virtual void onDetach(){
        jniEnv()->DeleteGlobalRef(handlerObj);
    }
}
```

接下来，我们将更新`HealthCheckThread`并实现一个`run`方法，该方法将提交健康检查消息到 UI 线程关联的`Handler`对象：

```java
virtual void run(){

  while (!shouldStop) {
    std::unique_lock<std::mutex> lck(stopMutex);
    // Do Work
    // ...
    sendHealthMessage();
    // Wait until a stop signal is sent
    stopCond.wait_for(lck, std::chrono::seconds(1));
  }
    }
```

`run`函数将持续执行，直到`shouldStop`为`true`。此外，在每次循环之间，线程将发送一条消息并阻塞一秒钟，除非发送停止信号通知线程停止。在这种情况下，使用本地条件变量唤醒线程从一秒钟的睡眠中醒来，当设置停止条件时。

关于`HealthCheckThread`类剩下的工作就是实现`sendHealthMessage`：

```java
void sendHealthMessage() {

  // Get the Handler class from the JVM
  jclass handlerClass = jniEnv()->FindClass("android/os/Handler");

  // Get the Handler.obtainMessage methodId 
  jmethodID obtainMId = jniEnv()->GetMethodID(handlerClass,  
 "obtainMessage","(ILjava/lang/Object;)Landroid/os/Message;");
  ...
  // Build up the alive message
  std::ostringstream oss;
  oss << "Thread[" << std::this_thread::get_id() 
      << "] is alive at " << ctime( & tt) << std::endl;;

  // Obtain a message object 
  jobject messagObj = jniEnv()->CallObjectMethod(handlerObj, 
                      obtainMId,
                      HEALTHCHECK_MESSAGE,
                      jniEnv()->NewStringUTF(oss.str().c_str()));

  // Get the Handler.senMessage methodId
  jmethodID sendMsgMId = jniEnv()->GetMethodID(handlerClass, 
                         "sendMessage","(Landroid/os/Message;)Z");

  // Enqueues a new message on the main thread looper
  jniEnv()->CallBooleanMethod(handlerObj,sendMsgMId, messagObj);
  // Deletes the local references
  jniEnv()->DeleteLocalRef(handlerClass);
  jniEnv()->DeleteLocalRef(messagObj);
}
```

在开始使用处理器对象之前，我们使用`obtainMessage`实例方法从`Handler`全局消息队列中检索一个`Message`。为了构建传递给`Message`对象的字符串消息，我们使用`ostringstream`、线程 ID 和当前`datetime`格式化消息。

然后，我们将构建的`Message`推送到处理器对象，以便在我们的`Activity`中传递。最后，我们从本地`JNIEnv`删除创建的局部引用。

## 启动本地线程

为了完成我们的示例，我们将编写`startNativeThreads`和`stopNativeThreads`本地方法。每次我们点击开始或停止按钮时，这些方法将创建和销毁本地线程。为了简洁起见，省略了 UI 代码。让我们首先看看`startNativeThreads`：

```java
static const int num_threads = 10;
static JavaThread* threads[num_threads];

void  Java_com_packpublishing_asynchronousandroid_chapter9_NativeThreadsActivity_startNativeThreads
        (JNIEnv *jEnv, jobject activity, jobject handler){

    LOGI("Starting  %d Native Threads",num_threads);
    // Launch a group of threads
    for (int i = 0; i < num_threads; ++i) {
        threads[i] = new HealthCheckThread(jEnv,handler);
        threads[i]->start();
     }
}
```

在 `startNativeThreads` 中，我们创建了 `num_thread` 个线程，并将主线程的 `JNIEnv` 和处理程序引用传递给 `HealthCheckThread` 构造函数。从构造函数返回的 `HealthCheckThread` 指针被缓存在一个静态数组中，以供将来使用。

## 停止本地线程

由于我们使用 C++ 操作符 `new` 在动态内存中分配 `HealthCheckThread` 对象，在 `stopNativeThreads` 中，除了停止线程执行外，还需要释放动态内存，以避免在本地代码中出现内存泄漏。因此，剩下的就是实现 `stopNativeThreads`：

```java
void Java_com_packpublishing_asynchronousandroid_chapter9_NativeThreadsActivity_stopNativeThreads(JNIEnv *env, jobject activity){

    LOGI("Stopping %d Native Threads", num_threads);

    for (int i = 0; i < num_threads; ++i) {
        // Notify the thread to stop
     threads[i]->stop();        
        // This blocks the execution of the current thread until
        // HealthCheckThread native thread finishes
        threads[i]->join();        
  // De-allocates memory previously allocated 
        delete threads[i];
    }
}
```

`stopNativeThreads` 函数将使用 `JavaThread::stop` 成员函数停止创建的线程。如前所述，`stop` 成员函数将使用条件原语来通知运行循环它应该完成执行。在我们通知后台线程后，我们等待它完成，并销毁存储在数组指针中的对象。

太好了！在本节中，我们能够启动本地线程，将它们附加到 JVM，并使用 Handler 对象与主线程交互。在这个过程中，我们学习了 C++ 的 `condition` 和 `mutex` 并发原语，以同步对共享资源的访问。尽管我们一直在使用 C++11 并发原语来创建和同步线程，但我们也可以使用 POSIX `pthread` 库提供的并发原语来编写我们的示例。

POSIX 库 `libpthread` 也提供了管理本地线程、互斥并发原语（互斥锁）和条件变量的方法。

# 在本地层处理 Java 异常

当在 Java 中，如果在方法执行期间抛出异常，JVM 会停止正常的方法执行，并尝试在运行时找到一个异常处理程序来控制执行，但当你从 JNI 代码执行 Java 方法时，情况并不相同。

JNI 要求开发者在 Java 方法调用发生异常后显式实现异常处理。

此外，当异常处理挂起时，只有少数 JNI 函数是安全的可以调用：`DeleteGlobalRef`、`DeleteLocalRef`、`DeleteWeakGlobalRef`、`ExceptionCheck`、`ExceptionClear`、`ExceptionDescribe`、`ExceptionOccurred`、`MonitorExit`、`PopLocalFrame`、`PushLocalFrame`、`Release<PrimitiveType>ArrayElements`、`ReleasePrimitiveArrayCritical`、`ReleaseStringChars`、`ReleaseStringCritical` 和 `ReleaseStringUTFChars`。

在本地函数中处理 Java 异常有三种方法。

第一种方法是使用 `ExceptionClear` 清除挂起的异常，并继续执行本地代码。这种方法很少是安全的，你需要审查所有错误流程，以验证你是否正确处理了异常。

第二种方式是，一旦检测到挂起的异常，释放 JNI 分配的资源，停止本地代码执行，并将控制权返回给 Java 代码。在这种情况下，JNI 将尝试在调用本地方法的 Java 帧中找到一个 Java 异常处理器。

第三种方式是释放挂起的异常，生成一个新的具有不同类类型的异常，并使用新的异常从本地方法返回，该异常将在 Java 代码中待处理。

在我们的下一个示例中，我们将遵循第三种方法，因为我们将使用 JNI 中大多数可用的函数来处理异常。首先，我们将向您展示如何使用 JNI 异常处理函数清除挂起的 Java 异常。除此之外，我们将停止本地方法执行，释放所有本地资源，并抛出一个不同的异常，以便在 Java 中由 `RuntimeException` 处理器处理。

首先，我们将编写一个 `Activity`，调用本地方法将在 JVM 中产生一个挂起的异常：

```java
public class ExceptionActivity extends Activity {

 ...
 OnClickListener onClickListener = new OnClickListener() {
    @Override
    public void onClick(View v) {
      try {
        // Allocate a ByteBuffer with a size of 8 bytes
        ByteBuffer byteBuffer = ByteBuffer.allocate(8);

        // Call a native function that will try to access 
        // an out of bounds buffer position
        genException(byteBuffer);
	    // Catches a Runtime Exception
      } catch (RuntimeException e) {
        // Prints the Exception Stack Trace to the TextView
        TextView console = (TextView)findViewById(R.id.console);
        StringWriter sw = new StringWriter();
        e.printStackTrace(new PrintWriter(sw));
        console.setText(sw.toString());
      }
    }
  };
  ...
  // Native Function that will generate and Exception
  private native void genException(ByteBuffer buffer);
}
```

每次点击 `genException` 按钮时，调用一个会抛出运行时异常（`java.lang.IndexOutOfBoundsException`）的本地方法。

`onClick (View v)` 方法只能处理 `java.lang.RuntimeException`，因此我们必须在本地函数中处理 `IndexOutOfBoundsException` 并将其转换为 `RuntimeException`：

```java
void Java_com_packpublishing_asynchronousandroid_chapter9_ExceptionActivity_genException(JNIEnv * jniEnv, jobject activityObj, jobject byteBuffer){

  // Get the ByteBuffer class 
  jclass byteBufC= jniEnv->GetObjectClass(byteBuffer);

  jmethodID getMid = jniEnv>GetMethodID(byteBufC,"get","(I)B");

// Trying to access a buffer position out of the buffer capacity
  jbyte byte = jniEnv->CallByteMethod(byteBuffer,getMid,32);

  if (jniEnv->ExceptionCheck()) {
    // Prints the exception  on the standard Error
    jniEnv->ExceptionDescribe();    
    // Clears the exception on the JVM
    jniEnv->ExceptionClear();
    jclass excC = jniEnv>FindClass("java/lang/RuntimeException");
    jniEnv->ThrowNew(excC,"Failed to get byte from buffer");
    // Release the Allocated Resources 
    jniEnv->DeleteLocalRef(excC);
    jniEnv->DeleteLocalRef(byteBufC);        
    // Return with Pending RuntimeException
    return;
  }
  ...
}
```

有两个函数用于检测 JNI 本地函数中的异常：

`ExceptionOccurred` 函数如果存在尚未处理的挂起异常，则返回一个 `jthrowable` 对象引用，如果没有异常准备就绪，则返回 null。

当 JVM 中存在未处理的未处理异常时，`ExceptionCheck` 函数返回 `jboolean`，该函数将返回 `JNI_TRUE` 作为结果。

假设我们不想使用 `ExceptionOccurred` 返回的 `jthrowable`，我们将使用 `ExceptionCheck` 来检测异常并进入异常处理代码分支。

然后，使用 `ExceptionDescribe` 函数，我们将打印当前挂起的 `throwable` 栈跟踪到错误输出，而使用 `ExceptionClear`，我们将从 JVM 中清除挂起的 `IndexOutOfBoundsException`。

由于我们只能在 `Activity` 函数中处理 `RuntimeException`，我们将向 JVM 附带一个 `RuntimeException`，以便在本地代码返回时立即处理。

总结来说，由于我们打算停止本地函数执行，我们必须在从本地函数返回之前释放任何资源或 JNI 引用。

是的，借助这些 JNI 异常，您应该能够检测和处理由 Java 方法调用引起的任何已解决的未处理异常。如前所述，在尝试安全调用其他 JNI 方法之前，手动处理任何挂起的异常至关重要，这些方法调用 `member` 或 `static` 函数，`get` 或 `set` 对象上的字段，甚至创建新对象。

# 从本地代码与 Java 监视器交互

到目前为止，我们一直在 Java 线程中使用`synchronized`语句或`synchronized`方法同步对共享资源的访问：

```java
synchronized (obj) { ... // synchronized block }
synchronized void incrementCount() { ... // synchronized methods }
```

当我们执行本地方法并希望访问多个 Java 代码和本地代码之间共享的资源或变量时，JNI 为我们提供了`MonitorEnter`和`MonitorExit`方法来控制对由 Java `synchronized`块管理的互斥区的访问：

```java
jint MonitorEnter(JNIEnv *env, jobject obj);
jint MonitorExit(JNIEnv *env, jobject obj);
```

`MonitorEnter`是负责获取 Java 监视器范围的访问的函数，当另一个本地线程或 Java 线程是监视器的所有者时可能会阻塞。当任何线程获得对块的访问时，JVM 将确保除了当前线程外，没有其他线程进入临界区。

`MonitorExit`是负责释放之前使用`MonitorEnter`获得的监视器的函数，给其他线程进入互斥区的机会。

### 注意

为了防止死锁条件，任何`MonitorEnter`调用都必须跟随一个`MonitorExit`调用。

在我们的下一个代码示例中，我们将演示这种技术来同步 Java 代码和本地代码使用的共享对象的访问。

我们将创建一个本地线程，该线程将不断从我们在`Activity`中管理的共享队列列表中轮询命令请求。

`StatsActivity`将有一个按钮用于将命令推送到共享队列列表，并将由本地线程发送的请求响应显示在`TextView`中。而 UI 将在主线程中将命令推送到请求队列列表，本地代码将尝试在后台线程中从本地代码中拉取命令，两者都需要对共享队列列表进行同步访问。

我们的命令将要求本地层发送有关程序运行所需主内存数量的信息。一旦收到响应，它将在 UI 的`TextView`中打印出来。

首先，让我们从 UI 的角度定义推送新命令的代码：

```java
public class StatsActivity extends Activity {
     // Memory RSS(Resident Set Size) SIZE Retrieve size Request
  public static final int MEM_RSS_REQUEST = 0;

  // Shared Resource between Java and Native
  Queue<Integer> requests = new LinkedList<Integer>();
  Object queueLock = new Object();

  OnClickListener onRSSReqListener = new OnClickListener() {
    @Override
    public void onClick(View v) {
      synchronized (queueLock) {
        requests.add(MEM_RSS_REQUEST);
      }
    }
  };

  @Override
  protected void onCreate(Bundle savedInstanceState) {
    ...
    RSSButton.setOnClickListener(onRSSReqListener);
  }
}
```

注意，在`onRSSReqListener`将新请求命令推送到队列列表之前，它获取了对由`queueLock`对象控制的同步部分的访问。

由于`queueLock`对象将作为访问共享资源的保护对象，我们必须将其传递给本地代码。

由于我们已编写了命令请求消费者，现在我们将关注命令请求消费者，名为`CPUStatThread`的本地 C++ `JavaThread`子类，它将处理请求并发送命令响应。

如前所述，`CPUStatThread`将实现`run`方法，并通过`Activity`的`Handler`发送响应，因此我们首先在新的源代码文件`stats.cpp`中实现`run`方法以从`Activity`检索请求：

```java
#include "thread.h" // Header where JavaThread is defined

static const int RSS_REQUEST= 0;

class CPUStatThread: public JavaThread {

  // Reference to the Activity and received on the constructor
  jobject activityObj; 
  ...
  virtual void run() {
    while ( !shouldStop ) {
      std::unique_lock<std::mutex> lck(stopMutex);
      processMessage();
      // Wait until a stop signal is sent
         stopCond.wait_for(lck,std::chrono::milliseconds(200));
    }
  }
  void processMessage(){

    jclass activityClass = jniEnv()->GetObjectClass(activityObj);

    // Retrieve the QueueLock(lockObj) and the Handler 
    // Fields(handlerObj) objects from Activity and 
    // getNextRequest methodId
	 ... 
    // Acquire the queue monitor	 
    jniEnv()->MonitorEnter(lockObj);

    // Retrieve the next command request to be processed
    int requestCode = jniEnv()->CallIntMethod(activityObj, 
                                              getNextRequestMid);
    switch (requestCode){
      case RSS_REQUEST:
        LOGI("Received a RSS Request");
        sendRSSMessage(handlerObj);
        break;
    }
    // Release the queue monitor	
      jniEnv()->MonitorExit(lockObj);
	// Release local References to avoid leaks
   ...
  }
};
```

我们的运行方法将从`StatsActivity`中检索`queueLock`字段，并在获取到由`queueLock`控制的同步块访问权限后，使用`StatsActivity`的`getNextRequest`方法从队列中拉取一个新的请求：

```java
public class StatsActivity extends Activity {

  int getNextRequest(){
    return requests.size()> 0 ? requests.remove():-1;
  }
}
```

`getNextRequest`将在没有可处理的内容时返回-1，因此我们的线程将根据我们在运行方法中定义的，休眠 100 毫秒。

当接收到`RSS_REQUEST`时，我们的原生后台将在`sendRSSMessage`方法中处理它，并通过系统进程消耗的内存发送一个响应。

## 包装原生数据对象

到目前为止，为了从原生代码向 Java 代码发送任何类型的结构化数据，我们一直在构建和分发常规 Java 对象。然而，为了减少从原生类型转换为 Java 类型以及相反所需的开销，发送原生包装结构或对象指针到 Java 运行时可能是有意义的，而不是在原生代码中创建纯 Java 对象。

最可靠的技术是将原生地址存储在包装对象的 64 位和 32 位指针兼容的长成员变量中：

```java
public class MyObject {
  // Transports a pointer to original 
  // native object or structure
  long nativePtr; 
}
```

如你所知，JVM 垃圾回收器会持续维护堆内存并清理未引用的对象，以便为应用程序下一次所需的分配释放更多内存。然而，这并不适用于使用`new`运算符或`malloc`函数在动态内存中分配的原生对象。

当我们在原生堆中创建一个对象时，我们总是必须显式地使用`delete`运算符或`free`函数来释放它，因此为了强制所有包装原生对象的内存清理，我们将定义一个接口，该接口定义了释放底层原生对象所需的功能：

```java
public interface Disposable {
  // Releases the native objects wrapped on the object
  void dispose();
}
```

### 注意

尽管我们可以在对象被垃圾回收时使用`finalize`方法来释放任何原生资源，但无法保证垃圾回收器会在未来的任何特定时间调用`finalize`方法。

为了在我们的示例中演示这种技术，我们将使用`Handler`发送回一个包裹在 Java 对象中的原生`CPUStat``struct`。

让我们先定义在发送时携带与进程内存消耗相关的信息的原生`CPUStat`：

```java
struct CPUStat{
    double vm; // Virtual Memory Size
    double rss; // Process Resident Memory    
    // constructor 
    CPUStat( double &vm_,
             double &rss_):vm(vm_),rss(rss_){}
};

And its Java counterpart:

public class JCPUStat implements Disposable {    
    // Reference to the native struct stored on a long
    long nativePtr;

    public JCPUStat(long nativePtr){
        this.nativePtr = nativePtr;
    }
    native long getRSSMemory();

    @Override
    public native void dispose();
}
```

注意，我们的`JCPUStat`实现了之前解释的可丢弃对象，因此剩下的只是编写 JCPUStat 类的原生方法：

```java
   // Generic function to convert a nativePtr member to a T pointer 
template <typename T>
T * getNativePtr(JNIEnv * env, jobject obj) {
  jclass c = env->GetObjectClass(obj);
  jfieldID nativePtrFID =  env->GetFieldID(c, "nativePtr", "J");
  jlong nativePtr = env->GetLongField(obj, nativePtrFID);
  return reinterpret_cast<T * >(nativePtr);
}

void unsetNativePtr(JNIEnv * env, jobject obj) {
  jclass c = env->GetObjectClass(obj);
  jfieldID nativePtrFID =  env->GetFieldID(c, "nativePtr", "J");
  env->SetLongField(obj, nativePtrFID, 0);
}

void Java_com_packpublishing_asynchronousandroid_chapter9_JCPUStat_dispose(JNIEnv *env, jobject obj){

   // Retrieves the pointer to the original structure
   CPUStat *stat = getNativePtr<CPUStat>(env,obj);
   if ( stat != 0 ) {
     delete stat;  // Releases the memory allocated to stat
     unsetNativePtr(ev,obj); 
   }
   }
```

为了简化原生引用的处理，我们创建了两个通用函数来操作包装对象中的`nativePtr`字段。第一个函数`getNativePtr`将从对象中获取指针字段，并借助`reinterpret_cast`将存储在`CPUStat`指针中的原始长值转换。

在我们获取到原始指针后，我们可以调用`delete`运算符来释放系统中的内存，并将 nativePtr 设置为 0。将指针设置为 0 可以防止在错误地调用`dispose`方法两次时发生双重释放。

接下来，在定义了包装类之后，我们将处理原始请求，并构建一个`JCPUStat`响应对象，使用活动的`Handler`将其发送回`Activity`：

```java
// Function to retrieve the Memory Usage 
void CPUStatThread::processMemUsage(
  double& vm_usage, double& resident_set){...}

void CPUStatThread::sendRSSMessage(jobject & handlerObj) {

  double vm, rss;
  // Read the mempory usage
  processMemUsage(vm, rss);
  jclass jCpuStatClass = jniEnv()->FindClass(
    "com.packpublishing.asynchronousandroid.chapter9.JCPUStat");

  // Find the JCPUStat Constructor
  jmethodID  jCpuConstructorMid = jniEnv()->GetMethodID(
    jCpuStatClass, "<init>", "(J)V");
  // Create a native CPUStat object 
  CPUStat * cpuStat = new CPUStat(vm, rss);

  // Wrap the native object on a JCPUStat object  
  jlong nativePtr = reinterpret_cast<jlong>(cpuStat);
  jobject jCpuStat = jniEnv()->NewObject(
    jCpuStatClass, jCpuConstructorMid, nativePtr);

  // Get the Handler Reference and send Message
  ...    
  // Build up the Response Message with the 
  jobject messagObj = jniEnv()->CallObjectMethod(
     handlerObj, obtainMId, RSS_RESPONSE, jCpuStat);

  // Push the message to the main Thread Handler                   
  jniEnv()->CallBooleanMethod(handlerObj, sendMsgMId, messagObj);
  // Clean up the local references
  ...
  }
```

我们的`sendRSSMessage`函数将使用系统功能计算进程消耗的内存，并构建一个包装本地 C++结构的`JCPUStat`对象。之后，使用`sendRSSMessage`函数传入的活动处理程序成员对象将`JCPUStat`调度到主线程。最后，我们清理了在局部作用域中创建的所有局部引用。

完整的源代码可在 Packt Publishing 网站上找到。查看完整的源代码，以了解我们是如何确定当前进程消耗的内存的。

为了完成示例，我们将更新`StatsActivity`以在`Handler`上处理 RSS 命令响应：

```java
public class StatsActivity extends Activity {
  public static final int MEM_RSS_REQUEST = 0;
  public static final int MEM_RSS_RESPONSE = 1;
  public Handler myHandler = new Handler() {
    public void handleMessage(Message msg) {
      switch (msg.what){
        case MEM_RSS_RESPONSE:
          TextView tv = (TextView) findViewById(R.id.console);
          JCPUStat stat = (JCPUStat) msg.obj;
          tv.setText("Memory Consumed is "+stat.getRSSMemory());
          // Releases the native object and frees the memory
          stat.dispose();
          break;
      }
    }
  };
  ...
  public native void startCPUStatThread();
  public native void stopCPUStatThread()
}
```

一旦我们从`Message`对象中获取到`JCPUStat`，我们就使用其本地方法`getRSSMemory`读取 RSS 内存，然后在控制台 UI 小部件上打印结果。

正如我们之前解释的，`JCPUStat.dispose`方法在 Java 运行时被显式调用，以销毁由后台线程发送给我们的本地对象。JVM GC 不会清理本地对象，因此我们必须调用`dispose`以释放附加到`Disposable`对象的本地资源。

`getRSSMemory`方法类似于`dispose`方法，将利用`nativePtr`字段从本地对象检索存储的 RSS 值。让我们看看它的样子：

```java
jlong Java_com_packpublishing_asynchronousandroid_chapter9_JCPUStat_getRSSMemory(JNIEnv *env, jobject obj) {
    CPUStat *stat = getNativePtr<CPUStat>(env,obj);
    return (jlong)stat->rss;
}
```

为了简洁，省略了`startCPUStatThread`和`stopCPUStatThread`，因为它们与之前示例中启动本地线程所使用的代码非常相似——请参阅可下载的示例以获取完整代码。

太好了！我们学习了如何将本地对象包装在 Java 对象中，我们定义了一个接口，在本地对象不再需要时从 Java 对象中清除本地内存，我们还学习了如何通过调用对象构造函数从本地 Java 对象创建对象。

# 摘要

在本章中，我们向您介绍了 JNI，这是一个在 Java 上可用的标准 API，用于与用汇编、C 或 C++编写的本地代码交互，它对安装了 Android NDK 套件的任何 Android 开发者都可用。

在第一部分，我们解释了如何在 Android Studio 上设置带有 JNI 代码的项目，以及如何从您的应用程序中的任何 Java 类调用 C 函数和 C++成员函数。

之后，我们使用 JNI 接口在本地函数上执行一个`Loader`异步后台工作。本地函数能够在由`AsyncTaskLoader`创建的 Java 后台线程中将彩色图像转换为灰度图像。

接下来，我们将发现如何将使用 C++标准库创建的纯本地线程附加和分离到 JVM。附加的线程作为一个正常的 Java 线程工作，并管理自己的 JNI 环境、资源和引用。

同时，我们还发现了 JNI 全局引用和局部引用之间的差异，以及如何从本地代码作用域访问 Java 对象字段。

我们还学习了一种将原生对象包装在 Java 对象中的技术，并定义了一个具体接口来处理附加到 Java 对象上的 JNI 资源。

在本章结束时，我们学习了如何检测和处理在 JVM 上由 Java 函数抛出的挂起异常。

本章中解释的所有技术，您应该能够将任何用 C/C++ 编写的代码集成到您的异步后台执行中。除此之外，您还可以利用原生代码来优化应用程序中的关键功能或集成一些原生实用库。

在下一章中，我们将学习如何使用 Google GCM 从您的服务器高效地推送和拉取实时消息，以及如何使用 Google Play Services 框架安排工作。
