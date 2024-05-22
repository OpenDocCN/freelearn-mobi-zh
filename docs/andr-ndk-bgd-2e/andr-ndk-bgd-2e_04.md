# 第四章：从本地代码回调 Java

> *为了发挥其最大潜力，JNI 允许从 C/C++ 回调 Java 代码。"回调"是因为本地代码首先从 Java 被调用，然后反过来调用 Java。这种调用是通过反射 API 完成的，几乎可以做任何直接在 Java 中能做的事情。*
> 
> *在使用 JNI 时需要考虑的另一个重要问题是线程。本地代码可以在由 Dalvik VM 管理的 Java 线程上运行，也可以从使用标准 POSIX 原语创建的本机线程上运行。显然，除非将本地线程转换为管理的 Java 线程，否则本地线程不能调用 JNI 代码！使用 JNI 编程需要了解所有这些细微之处。本章将引导你了解主要的几个问题。*
> 
> *最后一个主题是特定于 Android 而不是 JNI 的：特定的 Android 位图 API 旨在为运行在这些小型（但强大）设备上的图形应用程序提供完全的处理能力。*
> 
> *Android NDK 还提供了一个新的 API，以本地方式访问一种重要的对象类型：位图。特定的 Bitmap API，它是 Android 独有的，为运行在这些小型（但强大）设备上的图形应用程序提供了完全的处理能力。*

我们在上一章中开始的 `Store` 项目将作为展示 JNI 回调和同步的画布。为了说明位图处理，我们将创建一个新项目，在本地代码中解码设备的摄像头馈送。

总结一下，在本章中，我们将学习如何：

+   从本地代码调用 Java

+   将本地线程附加到 Dalvik VM，并与 Java 线程处理同步

+   在本地代码中处理 Java 位图

在本章结束时，你应该能够使 Java 和 C/C++ 互相通信和同步。

# 从本地代码回调 Java

在上一章中，我们了解了如何使用 JNI 方法 `FindClass()` 获取 Java 类描述符。然而，我们还可以获得更多！实际上，如果你是一个常规的 Java 开发者，这应该会让你想起一些东西：Java 反射 API。JNI 与其类似，它可以修改 Java 对象字段，运行 Java 方法，以及访问静态成员，但这一切都来自本地代码！

在 `Store` 项目的最后一部分，让我们增强我们的商店应用程序，使其在成功插入条目时通知 Java。

### 注意

本书提供的最终项目名为 `Store_Part10`。

# 动手实践时间——确定 JNI 方法签名

让我们先定义一个 Java 接口，本地 C/C++ 代码将通过 JNI 调用这个接口：

1.  创建一个 `StoreListener.java`，其中包含一个定义几个回调的接口，一个用于整数，一个用于字符串，一个用于颜色，如下所示：

    ```java
    package com.packtpub.store;

    public interface StoreListener {
        void onSuccess(int pValue);

        void onSuccess(String pValue);

        void onSuccess(Color pValue);
    }
    ```

1.  打开 `Store.java` 并进行一些更改。

    +   声明一个成员委托 `StoreListener`，成功回调将被发送给它

    +   更改 `Store` 构造函数以注入委托监听器，这将是 `StoreActivity`

        ```java
        Public class Store implements StoreListener {
         private StoreListener mListener;
            public Store(StoreListener pListener) {
                mListener = pListener;
            }
            ...
        ```

        最后，实现`StoreListener`接口及其相应的方法，这些方法只是将调用转发给委托：

        ```java
            ...
         public void onSuccess(int pValue) {
         mListener.onSuccess(pValue);
         }

         public void onSuccess(String pValue) {
         mListener.onSuccess(pValue);
         }

         public void onSuccess(Color pValue) {
         mListener.onSuccess(pValue);
            }
        }
        ```

1.  打开`StoreActivity.java`并在`PlaceholderFragment`中实现`StoreListener`接口。

    同时，相应地更改`Store`构造：

    ```java
    public class StoreActivity extends Activity {
        ...
        public static class PlaceholderFragment extends Fragment
     implements StoreListener {
     private Store mStore = new Store(this);
            ...
    ```

    当接收到成功回调时，会弹出一个简单的提示消息：

    ```java
            ...
     public void onSuccess(int pValue) {
     displayMessage(String.format(
     "Integer '%1$d' successfuly saved!", pValue));
     }

     public void onSuccess(String pValue) {
     displayMessage(String.format(
     "String '%1$s' successfuly saved!", pValue));
     }

     public void onSuccess(Color pValue) {
     displayMessage(String.format(
     "Color '%1$s' successfuly saved!", pValue));
            }
        }
    }
    ```

1.  在`Store`项目的目录中打开终端，并运行`javap`命令以确定方法签名。

    ```java
    javap –s -classpath bin/classes com.packtpub.store.Store
    ```

    ![动手实践——确定 JNI 方法签名](img/9645_04_01.jpg)

## *刚才发生了什么？*

使用 JNI API 回调 Java 方法需要**描述符**，我们将在下一部分看到。为了确定一个 Java 方法描述符，我们需要一个**签名**。实际上，Java 中的方法可以**重载**，这意味着可以有相同名称但不同参数的两个方法。这就是为什么需要签名的原因。

我们可以使用`javap`来确定一个方法的签名，`javap`是一个 JDK 实用程序，用于反汇编`.class`文件。然后这个签名可以传递给 JNI 反射 API。正式地说，签名是以下这样声明的：

```java
(<Parameter 1 Type Code>[<Parameter 1 Class>];...)<Return Type Code>
```

例如，方法`boolean myFunction(android.view.View pView, int pIndex)`的签名将是`(Landroid/view/View;I)Z`。另一个例子，`(I)V`，意味着需要整数并返回 void。最后一个例子，`(Ljava/lang/String;)V`，意味着传递了一个 String 作为参数。

下表总结了 JNI 中可用的各种类型及其代码：

| Java 类型 | 本地类型 | 本地数组类型 | 类型代码 | 数组类型代码 |
| --- | --- | --- | --- | --- |
| `boolean` | `jboolean` | `jbooleanArray` | `Z` | `[Z` |
| `byte` | `jbyte` | `jbyteArray` | `B` | `[B` |
| `char` | `jchar` | `jcharArray` | `C` | `[C` |
| `double` | `jdouble` | `jdoubleArray` | `D` | `[D` |
| `float` | `jfloat` | `jfloatArray` | `F` | `[F` |
| `int` | `jint` | `jintArray` | `I` | `[I` |
| `long` | `jlong` | `jlongArray` | `J` | `[J` |
| `Short` | `jshort` | `jshortArray` | `S` | `[S` |
| `Object` | `jobject` | `jobjectArray` | `L` | `[L` |
| `String` | `jstring` | `N/A` | `L` | `[L` |
| `Class` | `jclass` | `N/A` | `L` | `[L` |
| `Throwable` | `jthrowable` | `N/A` | `L` | `[L` |
| `void` | `void` | `N/A` | `V` | `N/A` |

所有这些值都与`javap`转储的值相对应。关于描述符和签名的更多信息，请查看 Oracle 文档 [`docs.oracle.com/javase/specs/jvms/se7/html/jvms-4.html#jvms-4.3`](http://docs.oracle.com/javase/specs/jvms/se7/html/jvms-4.html#jvms-4.3)。

既然我们已经有了正确的签名，现在可以从 C/C++中调用 Java 了。

# 动手实践——从本地代码回调 Java

让我们继续通过从本地代码调用我们定义的接口来构建`Store`：

1.  在`com_packtpub_store_Store.cpp`中，为每个回调声明类型为`jmethodID`的方法描述符，这将会被缓存：

    ```java
    ...
    static Store gStore;

    static jclass StringClass;
    static jclass ColorClass;

    static jmethodID MethodOnSuccessInt;
    static jmethodID MethodOnSuccessString;
    static jmethodID MethodOnSuccessColor;
    ...
    ```

1.  然后，在`JNI_OnLoad()`中缓存所有回调描述符。这可以通过两个主要步骤完成：

    使用 JNI 方法`FindClass()`获取类描述符。通过类的绝对包路径，可以找到类描述符，例如：`com./packtpub/store/Store`。

    使用`GetMethodID()`从类描述符中获取方法描述符。为了区分几个重载方法，必须指定之前用`javap`获取的签名：

    ```java
    ...
    JNIEXPORT jint JNI_OnLoad(JavaVM* pVM, void* reserved) {
        JNIEnv *env;
        if (pVM->GetEnv((void**) &env, JNI_VERSION_1_6) != JNI_OK) {
            abort();
        }
        ...
        // Caches methods.
     jclass StoreClass = env->FindClass("com/packtpub/store/Store");
     if (StoreClass == NULL) abort();

     MethodOnSuccessInt = env->GetMethodID(StoreClass, "onSuccess",
     "(I)V");
     if (MethodOnSuccessInt == NULL) abort();

     MethodOnSuccessString = env->GetMethodID(StoreClass, "onSuccess",
     "(Ljava/lang/String;)V");
     if (MethodOnSuccessString == NULL) abort();

     MethodOnSuccessColor = env->GetMethodID(StoreClass, "onSuccess",
     "(Lcom/packtpub/store/Color;)V");
     if (MethodOnSuccessColor == NULL) abort();
     env->DeleteLocalRef(StoreClass);

        // Store initialization.
        gStore.mLength = 0;
        return JNI_VERSION_1_6;
    }
    ...
    ```

1.  当在`setInteger()`中成功插入整数时，通知 Java 商店（即`pThis`）。要调用 Java 对象上的 Java 方法，只需使用`CallVoidMethod()`（这意味着被调用的 Java 方法返回 void）。为此，我们需要：

    +   对象实例

    +   方法签名

    +   如果适用，传递有效的参数（这里是一个整数值）

        ```java
        ...
        JNIEXPORT void JNICALL Java_com_packtpub_store_Store_setInteger
          (JNIEnv* pEnv, jobject pThis, jstring pKey, jint pInteger) {
            StoreEntry* entry = allocateEntry(pEnv, &gStore, pKey);
            if (entry != NULL) {
                entry->mType = StoreType_Integer;
                entry->mValue.mInteger = pInteger;

         pEnv->CallVoidMethod(pThis, MethodOnSuccessInt,
         (jint) entry->mValue.mInteger);
            }
        }
        ...
        ```

1.  对字符串重复该操作。在分配返回的 Java 字符串时不需要生成全局引用，因为它在 Java 回调中立即使用。我们也可以在使用后立即销毁这个字符串的局部引用，但 JNI 在从原生回调返回时会处理这个问题：

    ```java
    ...
    JNIEXPORT void JNICALL Java_com_packtpub_store_Store_setString
      (JNIEnv* pEnv, jobject pThis, jstring pKey, jstring pString) {
        // Turns the Java string into a temporary C string.
        StoreEntry* entry = allocateEntry(pEnv, &gStore, pKey);
        if (entry != NULL) {
            entry->mType = StoreType_String;
            ...

            pEnv->CallVoidMethod(pThis, MethodOnSuccessString,
     (jstring) pEnv->NewStringUTF(entry->mValue.mString));
        }
    }
    ...
    ```

1.  最后，对颜色重复该操作：

    ```java
    ...
    JNIEXPORT void JNICALL Java_com_packtpub_store_Store_setColor
      (JNIEnv* pEnv, jobject pThis, jstring pKey, jobject pColor) {
        // Save the Color reference in the store.
        StoreEntry* entry = allocateEntry(pEnv, &gStore, pKey);
        if (entry != NULL) {
            entry->mType = StoreType_Color;
            entry->mValue.mColor = pEnv->NewGlobalRef(pColor);

            pEnv->CallVoidMethod(pThis, MethodOnSuccessColor,
     (jstring) entry->mValue.mColor);
        }
    }
    ...
    ```

## *刚才发生了什么？*

启动应用程序并插入一个整数、字符串或颜色条目。系统会显示包含插入值的成功信息。原生代码通过 JNI 反射 API 调用了 Java 端。这个 API 不仅用于执行 Java 方法，也是处理传递给原生方法的`jobject`参数的唯一方式。然而，从 Java 调用 C/C++代码相对简单，而从 C/C++执行 Java 操作则要复杂一些！

尽管有些重复和冗长，调用任何 Java 方法都应该像这样简单：

+   从我们想要调用方法的类描述符中获取类描述符（这里的`Store` Java 对象）：

    ```java
    jclass StoreClass = env->FindClass("com/packtpub/store/Store");
    ```

+   获取我们想要调用的回调的方法描述符（如在 Java 中的`Method`类）。这些方法描述符是从拥有它的类描述符中获取的（如在 Java 中的`Class`）：

    ```java
    jmethodID MethodOnSuccessInt = env->GetMethodID(StoreClass,
                                                    "onSuccess", "(I)V");
    ```

+   可选地，缓存描述符以便它们可以在未来的原生调用中立即使用。同样，`JNI_OnLoad()`使得在执行任何原生调用之前缓存 JNI 描述符变得容易。以`Id`结尾的描述符，如`jmethodID`，可以自由缓存。它们不是可以泄漏的引用，或者相对于`jclass`描述符必须全局化。

    ### 提示

    缓存描述符绝对是好的实践，因为通过 JNI 反射获取字段或方法可能会产生一些开销。

+   在对象上使用必要的参数调用方法。相同的方法描述符可以用于相应类的任何对象实例：

    ```java
    env->CallVoidMethod(pThis, MethodOnSuccessInt, (jint) myInt); 
    ```

无论你需要在一个 Java 对象上调用什么方法，同样的过程总是适用。

## 关于 JNI 反射 API 的更多内容

了解反射 API 后，你基本上就掌握了 JNI 的大部分内容。以下是一些可能有用的方法：

+   `FindClass()`根据其绝对路径获取（局部）引用到`Class`描述符对象：

    ```java
    jclass FindClass(const char* name)
    ```

+   `GetObjectClass()` 的目的相同，不同之处在于 `FindClass()` 根据它们的绝对路径查找类定义，而另一个直接从对象实例（如 Java 中的 `getClass()`）查找类：

    ```java
    jclass GetObjectClass(jobject obj)
    ```

+   以下方法允许您获取方法和字段的 JNI 描述符，以及静态或实例成员。这些描述符是 ID，而不是对 Java 对象的引用。无需将它们转换为全局引用。这些方法需要方法或字段名称以及签名以区分重载。构造函数描述符的获取方式与方法的获取方式相同，不同之处在于其名称始终为 `<init>` 并且具有 void 返回值：

    ```java
    jmethodID GetMethodID(jclass clazz, const char* name,
                          const char* sig) 
    jmethodID GetStaticMethodID(jclass clazz, const char* name,
                                const char* sig)

    jfieldID GetStaticFieldID(jclass clazz, const char* name,
                              const char* sig)
    jfieldID GetFieldID(jclass clazz, const char* name, const char* sig)
    ```

+   有另一组方法可以通过对应的描述符来获取字段值。每种基本类型都有一对获取器和设置器方法，以及一个用于对象的方法：

    ```java
    jobject GetObjectField(jobject obj, jfieldID fieldID)
    <primitive> Get<Primitive>Field(jobject obj, jfieldID fieldID)

    void SetObjectField(jobject obj, jfieldID fieldID, jobject value)
    void Set<Primitive>Field(jobject obj, jfieldID fieldID,
                             <jprimitive> value)
    ```

+   对于根据它们的返回值分类的方法同样如此：

    ```java
    jobject CallObjectMethod(JNIEnv*, jobject, jmethodID, ...)

    <jprimitive> Call<Primitive>Method(JNIEnv*, jobject, jmethodID, ...);
    ```

+   这些方法存在带有 `A` 和 `V` 后缀的变体。行为相同，不同之处在于参数分别使用 `va_list`（即可变参数列表）或 `jvalue` 数组（`jvalue` 是所有 JNI 类型的联合体）指定：

    ```java
    jobject CallObjectMethodV(JNIEnv*, jobject, jmethodID, va_list);
    jobject CallObjectMethodA(JNIEnv*, jobject, jmethodID, jvalue*);
    ```

请查看 Android NDK `include` 目录中的 `jni.h` 文件，以了解 JNI 反射 API 的所有可能性。

## 调试 JNI

JNI 调用的目标通常是性能。因此，当调用其 API 方法时，JNI 并不执行高级检查。幸运的是，存在一种**扩展检查**模式，它执行高级检查并在 Android Logcat 中提供反馈。

要激活它，请从命令提示符运行以下命令：

```java
adb shell setprop debug.checkjni 1

```

设置此标志后，启动的应用程序可以使用扩展检查模式，直到将其设置为 `0`，或者直到设备重新启动。对于已获得根权限的设备，可以使用以下命令启动整个设备：

```java
adb shell stop
adb shell setprop dalvik.vm.checkjni true
adb shell start

```

如果一切正常，当你的应用程序启动时，Logcat 中会出现 **Late-enabling – Xcheck:jni** 的消息。然后，定期检查 Logcat 以查找其 JNI 警告或错误。

![调试 JNI](img/9645_04_04.jpg)

## 同步 Java 和本地线程

并行编程如今是一个主流课题。自从引入多核处理器以来，Android 也不例外。你可以在 Java 端完全进行线程处理（使用 Java Thread 和 Concurrency API），在本地端（使用 NDK 提供的 **POSIX PThread** API），以及更有趣的是，使用 JNI 在 Java 和本地端之间进行。

在这一部分，我们将创建一个后台线程，观察者，它始终关注数据存储中的内容。它遍历所有条目，然后休眠固定的时间。当观察者线程找到在代码中预定义的特定类型的键时，它会相应地采取行动。对于这一部分，我们只是将整数值裁剪到预定义的范围。

当然，线程需要同步。本地线程只在用户理解 UI 线程并且不修改它时访问和更新存储。本地线程在 C/C++中创建，但 UI 线程是 Java 线程。我们将使用 JNI 监视器来同步它们两者。

# 行动时间——使用 JNI 分配对象。

让我们定义一个后台观察者，它将使用在 Java 和 C/C++之间共享的对象作为锁：

1.  在`Store.java`中，添加两个新方法以启动和停止观察者线程。这两个方法分别返回和接受一个`long`类型的参数。这个值可以帮助我们在 Java 端保存本地指针：

    ```java
    public class Store implements StoreListener {
        ...
        public native long startWatcher();
     public native void stopWatcher(long pPointer);
    }
    ```

1.  创建一个新文件`StoreThreadSafe.java`。`StoreThreadSafe`类继承自`Store`类，旨在使用`synchronized` Java 代码块使`Store`实例线程安全。声明一个类型为`Object`的静态成员字段`LOCK`并定义一个默认构造函数：

    ```java
    package com.packtpub.store;

    import com.packtpub.exception.InvalidTypeException;
    import com.packtpub.exception.NotExistingKeyException;

    public class StoreThreadSafe extends Store {
        protected static Object LOCK;

        public StoreThreadSafe(StoreListener pListener) {
            super(pListener);
        }
        ...
    ```

1.  重写`Store`类的方法，如`getCount()`、`getInteger()`和`setInteger()`，使用与`LOCK`对象同步的 Java 代码块：

    ```java
        ...
        @Override
        public int getCount() {
            synchronized (LOCK) {
                return super.getCount();
            }
        }
        ...
        @Override
        public int getInteger(String pKey)
            throws NotExistingKeyException, InvalidTypeException
        {
            synchronized (LOCK) {
                return super.getInteger(pKey);
            }
        }

        @Override
        public void setInteger(String pKey, int pInt) {
            synchronized (LOCK) {
                super.setInteger(pKey, pInt);
            }
        }
        ...
    ```

1.  对所有其他方法，如`getString()`、`setString()`、`getColor()`、`setColor()`等，以及`stopWatcher()`方法执行相同的操作。不要重写`onSuccess`回调方法和`startWatcher()`方法：

    ```java
        ...
        @Override
        public void stopWatcher(long pPointer) {
            synchronized (LOCK) {
                super.stopWatcher(pPointer);
            }
        }
    }
    ```

    *不要*重写`onSuccess`回调方法和`startWatcher()`方法。

1.  打开`StoreActivity.java`，并用`StoreThreadSafe`的实例替换之前的`Store`实例。同时，创建一个类型为`long`的成员字段以保存指向观察者线程的本地指针。当片段恢复时，启动观察者线程并保存其指针。当片段暂停时，使用先前保存的指针停止观察者线程：

    ```java
    public class StoreActivity extends Activity {
        ...
        public static class PlaceholderFragment extends Fragment
        implements StoreListener {
            private StoreThreadSafe mStore = new StoreThreadSafe(this);
     private long mWatcher;
            private EditText mUIKeyEdit, mUIValueEdit;
            private Spinner mUITypeSpinner;
            private Button mUIGetButton, mUISetButton;
            private Pattern mKeyPattern;

            ...
     @Override
     public void onResume() {
     super.onResume();
     mWatcher = mStore.startWatcher();
     }
     @Override
     public void onPause() {
     super.onPause();
     mStore.stopWatcher(mWatcher);
            }
            ...
        }
    }
    ```

1.  编辑`jni/Store.h`并包含一个新的头文件`pthread.h`：

    ```java
    #ifndef _STORE_H_
    #define _STORE_H_

    #include <cstdint>
    #include <pthread.h>
    #include "jni.h"
    ```

1.  观察者在定时间隔更新后的`Store`实例上工作。它需要：

    +   它所监视的`Store`结构的实例。

    +   一个`JavaVM`，它是线程间唯一可以安全共享的对象，并且可以从中安全获取`JNIEnv`。

    +   用于同步的 Java 对象（对应于我们在 Java 端定义的`LOCK`对象）

    +   用于本地线程管理的`pthread`变量。

    +   停止观察者线程的指示器。

        ```java
        ...
        typedef struct { 
         Store* mStore; 
         JavaVM* mJavaVM; 
         jobject mLock; 
         pthread_t mThread; 
         int32_t mRunning; 
        } StoreWatcher;
        ...
        ```

1.  最后，定义三个方法以启动和停止观察者线程，运行它的主循环和处理一个条目：

    ```java
    ...
    StoreWatcher* startWatcher(JavaVM* pJavaVM, Store* pStore, 
     jobject pLock); 
    void stopWatcher(StoreWatcher* pWatcher); 
    void* runWatcher(void* pArgs); 
    void processEntry(StoreEntry* pEntry);
    #endif
    ```

1.  使用`javah`刷新 JNI 头文件`jni/com_packtpub_Store.h`。你应在其中看到两个新方法，`Java_com_packtpub_store_Store_startWatcher()`和`Java_com_packtpub_store_Store_stopWatcher()`。

    在`com_packtpub_store_Store.cpp`中，创建一个新的静态变量`gLock`，它将保存 Java 同步对象。

    ```java
    ...
    static Store gStore;
    static jobject gLock;
    ...
    ```

1.  使用 JNI 反射 API 在`JNI_OnLoad()`中创建`Object`类的一个实例：

    +   首先，使用`GetMethodID()`找到它的`Object`构造函数。在 JNI 中，构造函数名为`<init>`并且没有返回结果。

    +   然后，调用构造函数以创建一个实例并将其全局化。

    +   最后，当本地引用不再有用时，移除它们：

        ```java
        JNIEXPORT jint JNI_OnLoad(JavaVM* pVM, void* reserved) {
            JNIEnv *env;
            if (pVM->GetEnv((void**) &env, JNI_VERSION_1_6) != JNI_OK) {
                abort();
            }
            ...
         jclass ObjectClass = env->FindClass("java/lang/Object");
         if (ObjectClass == NULL) abort();
         jmethodID ObjectConstructor = env->GetMethodID(ObjectClass,
         "<init>", "()V");
         if (ObjectConstructor == NULL) abort();
         jobject lockTmp = env->NewObject(ObjectClass, ObjectConstructor);
         env->DeleteLocalRef(ObjectClass);
         gLock = env->NewGlobalRef(lockTmp);
         env->DeleteLocalRef(lockTmp);
            ...
        ```

1.  将创建的`Object`实例保存在`StoreThreadSafe.LOCK`字段中。这个对象将在应用程序的生命周期内用于同步：

    +   首先，使用 JNI 反射方法`FindClass()`和`GetStaticFieldId()`检索`StoreThreadSafe`类及其`LOCK`字段。

    +   然后，使用 JNI 方法`SetStaticObjectField()`将值保存到`LOCK`静态字段中，该方法需要字段签名（如方法）。

    +   最后，当`StoreThreadSafe`类不再有用时，移除对其的本地引用：

        ```java
            ...
         jclass StoreThreadSafeClass = env->FindClass(
         "com/packtpub/store/StoreThreadSafe");
         if (StoreThreadSafeClass == NULL) abort();
         jfieldID lockField = env->GetStaticFieldID(StoreThreadSafeClass,
         "LOCK", "Ljava/lang/Object;");
         if (lockField == NULL) abort();
         env->SetStaticObjectField(StoreThreadSafeClass, lockField, gLock);
         env->DeleteLocalRef(StoreThreadSafeClass);

            return JNI_VERSION_1_6;
        }
        ...
        ```

1.  实现`startWatcher()`，它调用之前定义的相应方法。它需要`JavaVM`，可以从`JNIEnv`对象使用`GetJavaVM()`获取。创建的`Store`的指针（即内存地址）作为一个`long`值返回给 Java 端，然后可以保存它以供以后使用：

    ```java
    ...
    JNIEXPORT jlong JNICALL Java_com_packtpub_store_Store_startWatcher
      (JNIEnv *pEnv, jobject pThis) {
        JavaVM* javaVM;
        // Caches the VM.
        if (pEnv->GetJavaVM(&javaVM) != JNI_OK) abort();

        // Launches the background thread.
        StoreWatcher* watcher = startWatcher(javaVM, &gStore, gLock);
        return (jlong) watcher;
    }
    ...
    ```

1.  通过实现`stopWatcher()`来结束，它将给定的`long`值转换回本地指针。将其传递给相应的方法：

    ```java
    ...
    JNIEXPORT void JNICALL Java_com_packtpub_store_Store_stopWatcher
      (JNIEnv *pEnv, jobject pThis, jlong pWatcher) {
        stopWatcher((StoreWatcher*) pWatcher);
    }
    ```

## *刚才发生了什么？*

我们使用 JNI 从本地代码分配一个 Java 对象，并将其保存在一个静态的 Java 字段中。这个例子展示了 JNI 反射 API 的强大功能；几乎在 Java 中可以做的任何事情，都可以通过 JNI 从本地代码完成。

为了分配 Java 对象，JNI 提供了以下方法：

+   使用`NewObject()`通过指定的构造方法实例化一个 Java 对象：

    ```java
    jobject NewObject(jclass clazz, jmethodID methodID, ...)
    ```

+   该方法存在带有`A`和`V`后缀的变体。行为相同，不同之处在于参数分别使用`va_list`或`jvalue`数组指定：

    ```java
    jobject NewObjectV(jclass clazz, jmethodID methodID, va_list args)
    jobject NewObjectA(jclass clazz, jmethodID methodID, jvalue* args)
    ```

+   `AllocObject()`分配一个新对象但不调用其构造函数。可能的用途是分配许多不需要初始化的对象，以获得一些性能提升。只有在你清楚自己在做什么时才使用它：

    ```java
    jobject AllocObject(jclass clazz)
    ```

在上一章中，我们为本地存储使用了静态变量，因为其生命周期与应用程序相关联。我们希望记住值，直到应用程序退出。如果用户离开活动，稍后再回来，只要进程仍然存活，值仍然可用。

对于观察者线程，我们使用了不同的策略，因为其生命周期与活动相关联。当活动获得焦点时，创建并启动线程。当活动失去焦点时，停止并销毁线程。由于这个线程可能需要时间来停止，因此在`Store`示例中快速多次切换屏幕时，可能会有几个实例同时运行。

因此，使用静态变量是不安全的，因为它们可能会被并发覆盖（导致内存泄漏），或者更糟糕的是，被释放（导致内存损坏）。当活动启动另一个活动时，也可能出现这类问题。在这种情况下，第一个活动的`onStop()`和`onDestroy()`在第二个活动的`onCreate()`和`onStart()`之后发生，如 Android 活动生命周期所定义。

相反，处理这种情况的一个更好的解决方案是允许 Java 端管理原生内存。在我们的示例中，一个指向在原生端分配的原生结构的指针被返回给 Java 端作为一个 `long` 值。任何进一步的 JNI 调用必须使用此指针作为参数执行。然后，当这块数据生命周期结束时，可以将此指针还给原生端。

### 提示

使用 `long` 值（在 64 位上表示）来保存原生指针是必要的，以便与从 Android Lollipop 开始的 64 位版本 Android（具有 64 位内存地址）保持兼容。

总结一下，谨慎使用原生静态变量。如果你的变量与应用程序生命周期相关联，静态变量是可以的。如果变量与活动生命周期相关联，你应在活动中分配它们的实例，并从那里管理它们以避免问题。

现在，我们在 Java 和原生端之间有了共享锁，让我们通过实现观察线程继续我们的示例。

# 行动时刻——运行并同步线程

让我们使用 POSIX PThread API 创建一个原生线程并将其附加到 VM：

1.  在 `Store.cpp` 中，包含 `unistd.h`，它提供了访问 `sleep()` 函数的权限：

    ```java
    #include "Store.h"
    #include <cstdlib>
    #include <cstring>
    #include <unistd.h>
    ...
    ```

    实现 `startWatcher()` 方法。该方法从 UI 线程中执行。为此，首先实例化并初始化一个 `StoreWatcher` 结构。

1.  然后，使用 `pthread` POSIX API 初始化并启动一个原生线程：

    ```java
    StoreWatcher* startWatcher(JavaVM* pJavaVM, Store* pStore,
            jobject pLock) {
        StoreWatcher* watcher = new StoreWatcher();
        watcher->mJavaVM = pJavaVM;
        watcher->mStore = pStore;
        watcher->mLock = pLock;
        watcher->mRunning = true;
    ...
    ```

    然后，使用 PThread POSIX API 初始化并启动一个原生线程：

    +   `pthread_attr_init()` 初始化必要的数据结构

    +   `pthread_create()` 启动线程

        ```java
        ...
            pthread_attr_t lAttributes;
            if (pthread_attr_init(&lAttributes)) abort();
            if (pthread_create(&watcher->mThread, &lAttributes,
                                    runWatcher, watcher)) abort();
            return watcher;
        }
        ...
        ```

1.  实现 `stopWatcher()` 方法，关闭运行指示器以请求观察线程停止：

    ```java
    ...
    void stopWatcher(StoreWatcher* pWatcher) { 
        pWatcher->mRunning = false; 
    } 
    ...
    ```

1.  在 `runWatcher()` 中实现线程的主循环。在这里，我们不再处于 UI 线程，而是处于观察线程。

    +   因此，首先使用 `AttachCurrentThreadAsDaemon()` 将线程作为守护进程附加到 Dalvik VM。此操作从给定的 `JavaVM` 返回 `JNIEnv`。这使我们能从这个新线程直接访问 Java 端。记住 `JNIEnv` 是线程特定的，不能直接在线程间共享。

    +   然后，使这个线程循环并在每次迭代中休眠几秒钟，使用 `sleep()`：

        ```java
        ...
        void* runWatcher(void* pArgs) {
            StoreWatcher* watcher = (StoreWatcher*) pArgs;
            Store* store = watcher->mStore;

            JavaVM* javaVM = watcher->mJavaVM;
            JavaVMAttachArgs javaVMAttachArgs;
            javaVMAttachArgs.version = JNI_VERSION_1_6;
            javaVMAttachArgs.name = "NativeThread";
            javaVMAttachArgs.group = NULL;

            JNIEnv* env;
            if (javaVM->AttachCurrentThreadAsDaemon(&env,
                    &javaVMAttachArgs) != JNI_OK) abort();
            // Runs the thread loop.
            while (true) {
                sleep(5); // In seconds.
                    ...
        ```

1.  在循环迭代中，使用 JNI 方法 `MonitorEnter()` 和 `MonitorExit()` 划定一个临界区（一次只能有一个线程进入）。这些方法需要一个对象来进行同步（就像 Java 中的 `synchronized` 块）。

    然后，你可以安全地：

    +   检查线程是否应该停止，并在那种情况下离开循环

    +   处理来自存储的每个条目

        ```java
                    ...
                // Critical section beginning, one thread at a time.
                // Entries cannot be added or modified.
                env->MonitorEnter(watcher->mLock);
                if (!watcher->mRunning) break;
                StoreEntry* entry = watcher->mStore->mEntries;
                StoreEntry* entryEnd = entry + watcher->mStore->mLength;
                while (entry < entryEnd) {
                    processEntry(entry);
                    ++entry;
                }
                // Critical section end.
                env->MonitorExit(watcher->mLock);
            }
            ...
        ```

    在退出之前，当线程即将结束和退出时，分离线程。始终分离已附加的线程非常重要，这样 Dalvik 或 ART VM 就不再管理它。

1.  最后，使用 `pthread_exit()` API 方法终止线程：

    ```java
        ...
        javaVM->DetachCurrentThread();
        delete watcher;
        pthread_exit(NULL);
    }
    ...
    ```

1.  最后，编写`processEntry()`方法，该方法所做的不过是检查整数条目的边界，并将其限制在任意范围`[-100000,100000]`内。你也可以处理其他任何你希望处理的条目：

    ```java
    ...
    void processEntry(StoreEntry* pEntry) {
        switch (pEntry->mType) {
        case StoreType_Integer:
            if (pEntry->mValue.mInteger > 100000) {
                pEntry->mValue.mInteger = 100000;
            } else if (pEntry->mValue.mInteger < -100000) {
                pEntry->mValue.mInteger = -100000;
            }
            break;
        }
    }
    ```

## *刚才发生了什么？*

使用 Eclipse Java 调试器（不是本地调试器）以调试模式编译并运行应用程序。当应用程序启动时，会创建一个本地后台线程并将其附加到 Dalvik VM。你可以在**调试**视图中看到它。然后，UI 线程和本地后台线程使用 JNI 监视器 API 同步，以正确处理并发问题。最后，当离开应用程序时，后台线程会被分离并销毁。因此，它从**调试**视图中消失：

![刚才发生了什么？](img/9645_04_02.jpg)

现在，在您的 Android 设备上的`Store`接口中，定义一个键并输入一个大于`100,000`的整数值。等待几秒钟，然后使用相同的键检索该值。它应该会被观察者线程限制在`100,000`以内。这个观察者会检查存储中的每个值，并在需要时进行更改。

观察者运行在一个本地线程上（即不是由 Java 虚拟机直接创建的）。NDK 允许使用 PThread POSIX API 创建本地线程。这个 API 是一个在 Unix 系统上广泛用于多线程的标准。它定义了一系列以`pthread_`为前缀的函数和数据结构，不仅可以创建线程，还可以创建**互斥锁**（互斥的缩写）或**条件变量**（让一个线程等待特定条件）。

PThread API 本身就是一个完整的主题，超出了本书的范围。你需要了解它才能掌握 Android 上的本地多线程。有关此主题的更多信息，请查看[`computing.llnl.gov/tutorials/pthreads/`](https://computing.llnl.gov/tutorials/pthreads/)和[`randu.org/tutorials/threads/`](http://randu.org/tutorials/threads/)。

## 使用 JNI 监视器同步 Java 和 C/C++

在 Java 端，我们使用带有任意锁对象的`synchronized`块来同步线程。Java 还允许方法（无论是否为本地方法）被声明为`synchronized`。在这种情况下，锁对象是隐式地定义为本地方法的对象。例如，我们可以如下定义一个本地方法：

```java
public class MyNativeClass {
 public native synchronized int doSomething();
    ...
}
```

在我们的情况下，这本来是无法工作的，因为本地端有一个单一的静态存储实例。我们需要一个单一的静态锁对象实例。

### 注意

请注意，这里使用的模式，即让`StoreThreadSafe`继承自`Store`类，覆盖其方法并使用静态变量，不应特别认为是最佳实践。由于`Store`和`lock`对象是静态的，本书为了简单起见使用了这种方式。

在本地端，使用 JNI 监视器进行同步，这相当于 Java 中的`synchronized`关键字：

+   `MonitorEnter()`表示临界区的开始。监视器与一个对象关联，该对象可以被视为一种标识符。一次只能有一个线程进入由这个对象定义的区间：

    ```java
    jint MonitorEnter(jobject obj)
    ```

+   `MonitorExit()`表示临界区的结束。必须调用它，以及`MonitorEnter()`，以确保监视器被释放，其他线程可以继续执行：

    ```java
    jint MonitorExit(jobject obj)
    ```

因为 Java 线程在内部是基于 POSIX 原始操作，所以也可以完全本地实现线程同步，使用 POSIX API。你可以在这个链接找到更多信息：[`computing.llnl.gov/tutorials/pthreads/`](https://computing.llnl.gov/tutorials/pthreads/)。

### 提示

Java 和 C/C++是具有相似但略有不同语义的不同语言。因此，始终注意不要期望 C/C++的行为像 Java。例如，volatile 在 Java 和 C/C++中的语义是不同的，因为它们遵循不同的内存模型。

## 附着和分离本地线程

默认情况下，Dalvik VM 不知道在同一进程中运行的本地线程。作为回报，本地线程也无法访问 VM...除非它附着到 VM。在 JNI 中，以下方法处理附着：

+   使用`AttachCurrentThread()`告诉虚拟机管理当前线程。一旦附着，当前线程的`JNIEnv`指针将在指定位置返回：

    ```java
    jint AttachCurrentThread(JNIEnv** p_env, void* thr_args)
    ```

+   使用`AttachCurrentThreadAsDaemon()`将线程作为守护线程附着。Java 规范定义了 JVM 在退出前不必等待守护线程结束，与普通线程相反。在 Android 上，这种区别没有实际意义，因为应用程序可以在任何时候被系统杀死：

    ```java
    jint AttachCurrentThreadAsDaemon(JNIEnv** p_env, void* thr_args)
    ```

+   `DetachCurrentThread()`表示线程不再需要被管理。像 Watcher 线程这样的已附着线程在退出前必须最终被分离。Dalvik 会检测未分离的线程，并通过终止并在日志中留下不干净的崩溃转储来做出反应！在分离时，持有的任何监视器都会被释放，任何等待的线程都会被通知：

    ```java
    jint DetachCurrentThread()
    ```

    ### 提示

    自从 Android 2.0 起，确保线程被系统分离的一种技术是使用`pthread_key_create()`将析构函数回调绑定到本地线程，并在其中调用`DetachCurrentThread()`。可以使用`pthread_setspecific()`将`JNIEnv`实例保存到线程本地存储中，以便将其作为参数传递给析构函数。

线程附着后，**ClassLoader** JNI 会使用 Java 类来对应调用堆栈上找到的第一个对象。对于纯本地线程，可能找不到`ClassLoader`。在这种情况下，JNI 使用系统`ClassLoader`，它可能无法找到你自己的应用程序类，也就是说，`FindClass()`失败。在这种情况下，可以在`JNI_OnLoad()`中全局缓存必要的 JNI 元素，或者与需要线程共享应用程序类加载器。

# 本地处理位图

Android NDK 提供了一个专门用于位图处理的 API，可以直接访问 Android 位图的表面。这个 API 是特定于 Android 的，与 JNI 规范无关。然而，位图是 Java 对象，在本地代码中需要作为对象处理。

为了更具体地了解位图如何从本地代码中修改，让我们尝试从本地代码解码一个摄像头馈送。在 Android 上记录的原始视频帧通常以特定的格式编码，即**YUV**，这与传统的 RGB 图像不兼容。在这种情况下，本地代码可以提供帮助，帮助我们解码这些图像。在以下示例中，我们将把每个颜色组件（即红、绿和蓝）提取到单独的位图中。

### 注意

本书提供的结果项目名为`LiveCamera`。

# 动手操作时间——解码摄像头的馈送

让我们在一个全新的项目中编写必要的 Java 代码以记录和显示图片：

1.  按照第二章 *开始一个本地 Android 项目*所示，创建一个新的混合 Java/C++项目：

    +   命名为`LiveCamera`

    +   主包是`com.packtpub.livecamera`

    +   主要活动是`LiveCameraActivity`

    +   主活动布局名为`activity_livecamera`

    +   使用**空白活动**模板

1.  创建后，将项目转换为已知的本地项目。在`AndroidManifest.xml`文件中，请求访问摄像头的权限。然后，将活动样式设置为`fullscreen`，并将其方向设置为`landscape`。横屏方向避免了在 Android 设备上遇到的多数摄像头方向问题：

    ```java
    <?xml version="1.0" encoding="utf-8"?>
    <manifest 
      package="com.packtpub.livecamera"
      android:versionCode="1" android:versionName="1.0" >
      <uses-sdk android:minSdkVersion="14" android:targetSdkVersion="19"/>
     <uses-permission android:name="android.permission.CAMERA" />
      <application
        android:allowBackup="false"
        android:icon="@drawable/ic_launcher"
        android:label="@string/app_name" >
        <activity
          android:name=".LiveCameraActivity"
          android:label="@string/app_name"
          android:screenOrientation="landscape"
     android:theme="@android:style/Theme.NoTitleBar.Fullscreen" >
          <intent-filter>
            <action android:name="android.intent.action.MAIN" />
            <category android:name="android.intent.category.LAUNCHER" />
          </intent-filter>
        </activity>
      </application>
    </manifest>
    ```

1.  按以下方式定义`activity_livecamera.xml`布局。它表示一个包含一个`TextureView`和三个`ImageView`元素的 2x2 网格：

    ```java
    <?xml version="1.0" encoding="utf-8"?>
    <LinearLayout 

      a:baselineAligned="true" a:orientation="horizontal"
      a:layout_width="fill_parent" a:layout_height="fill_parent" >
      <LinearLayout
        a:layout_width="fill_parent" a:layout_height="fill_parent"
        a:layout_weight="1" a:orientation="vertical" >
        <TextureView
          a:id="@+id/preview" a:layout_weight="1"
          a:layout_width="fill_parent" a:layout_height="fill_parent" />
        <ImageView
          a:id="@+id/imageViewR" a:layout_weight="1"
          a:layout_width="fill_parent" a:layout_height="fill_parent" />
      </LinearLayout>
      <LinearLayout
        a:layout_width="fill_parent" a:layout_height="fill_parent"
        a:layout_weight="1" a:orientation="vertical" >
        <ImageView
          a:id="@+id/imageViewG" a:layout_weight="1"
          a:layout_width="fill_parent" a:layout_height="fill_parent" />
        <ImageView
          a:id="@+id/imageViewB" a:layout_weight="1"
          a:layout_width="fill_parent" a:layout_height="fill_parent" />
      </LinearLayout>
    </LinearLayout>
    ```

1.  打开`LiveCameraActivity.java`文件，并按以下方式实现：

    +   首先，扩展`SurfaceTextureListener`，这将帮助我们初始化和关闭摄像头馈送

    +   然后，扩展`PreviewCallback`接口以监听新的摄像头帧

    不要忘记按以下方式加载本地静态库：

    ```java
    package com.packtpub.livecamera;
    ...
    public class LiveCameraActivity extends Activity implements
    TextureView.SurfaceTextureListener, Camera.PreviewCallback {
        static {
            System.loadLibrary("livecamera");
        }
        ...
    ```

1.  创建一些成员变量：

    +   `mCamera`是 Android 摄像头 API

    +   `mTextureView`显示原始摄像头馈送

    +   `mVideoSource`将摄像头帧捕获到字节缓冲区

    +   `mImageViewR`、`G`和`B`显示处理过的图像，每个颜色组件一个

    +   `mImageR`、`G`和 B`是`ImageView`的位图支持（即“后台缓冲区”）

        ```java
            ...
            private Camera mCamera;
            private TextureView mTextureView;
            private byte[] mVideoSource;
            private ImageView mImageViewR, mImageViewG, mImageViewB;
            private Bitmap mImageR, mImageG, mImageB;
            ...
        ```

    在`onCreate()`中，指定在前一步中定义的布局。

    然后，获取要显示图像的视图。

1.  最后，使用`setSurfaceTextureListener()`监听`TextureView`事件。你可以忽略在这个例子中不必要的回调：

    ```java
        ...
        @Override
        protected void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            setContentView(R.layout.activity_livecamera);
            mTextureView = (TextureView) findViewById(R.id.preview);
            mImageViewR = ((ImageView)findViewById(R.id.imageViewR));
            mImageViewG = ((ImageView)findViewById(R.id.imageViewG));
            mImageViewB = ((ImageView)findViewById(R.id.imageViewB));

            mTextureView.setSurfaceTextureListener(this);
        }
        @Override
        public void onSurfaceTextureSizeChanged(SurfaceTexture pSurface,
           int pWidth, int pHeight) {}

        @Override
        public void onSurfaceTextureUpdated(SurfaceTexture pSurface) {}
        ...
    ```

1.  在`LiveCameraActivity.java`中的`onSurfaceTextureAvailable()`回调在创建`TextureView`表面后被触发。在这里可以知道表面尺寸和像素格式。

    因此，打开 Android 相机并将`TextureView`设置为它的预览目标。使用`setPreviewCallbackWithBuffer()`监听新的相机帧：

    ```java
        ...
        @Override
        public void onSurfaceTextureAvailable(SurfaceTexture pSurface,
                                              int pWidth, int pHeight) {
            mCamera = Camera.open();
            try {
                mCamera.setPreviewTexture(pSurface);
                mCamera.setPreviewCallbackWithBuffer(this);
                // Sets landscape mode to avoid complications related to
                // screen orientation handling.
                mCamera.setDisplayOrientation(0);
                ...
    ```

1.  然后，调用`findBestResolution()`，我们将在下一节实现它以找到适合相机馈送的合适分辨率。相应地设置后者为`YCbCr_420_SP`格式（这应该是 Android 上的默认格式）。

    ```java
                ...
                Size size = findBestResolution(pWidth, pHeight);
                PixelFormat pixelFormat = new PixelFormat();
                PixelFormat.getPixelFormatInfo(mCamera.getParameters()
                                .getPreviewFormat(), pixelFormat);
                int sourceSize = size.width * size.height
                                * pixelFormat.bitsPerPixel / 8;
                // Set-up camera size and video format.
                // should be the default on Android anyway.
                Camera.Parameters parameters = mCamera.getParameters();
                parameters.setPreviewSize(size.width, size.height);
                parameters.setPreviewFormat(PixelFormat.YCbCr_420_SP);
                mCamera.setParameters(parameters);
                ...
    ```

1.  之后，设置视频缓冲区和显示相机帧的位图：

    ```java
                ...
                mVideoSource = new byte[sourceSize];
                mImageR = Bitmap.createBitmap(size.width, size.height,
                                              Bitmap.Config.ARGB_8888);
                mImageG = Bitmap.createBitmap(size.width, size.height,
                                              Bitmap.Config.ARGB_8888);
                mImageB = Bitmap.createBitmap(size.width, size.height,
                                              Bitmap.Config.ARGB_8888);
                mImageViewR.setImageBitmap(mImageR);
                mImageViewG.setImageBitmap(mImageG);
                mImageViewB.setImageBitmap(mImageB);
                ...
    ```

    最后，将视频帧缓冲区入队并开始相机预览：

    ```java
                ...
                mCamera.addCallbackBuffer(mVideoSource);
                mCamera.startPreview();
            } catch (IOException ioe) {
                mCamera.release();
                mCamera = null;
                throw new IllegalStateException();
            }
        }
        ...
    ```

1.  仍然在`LiveCameraActivity.java`中，实现`findBestResolution()`。Android 相机可以支持多种分辨率，这些分辨率高度依赖于设备。由于没有规定默认分辨率应该是什么，我们需要寻找一个合适的分辨率。在这里，我们选择适合显示表面的最大分辨率，或者如果没有找到，则选择默认分辨率。

    ```java
        ...
        private Size findBestResolution(int pWidth, int pHeight) {
            List<Size> sizes = mCamera.getParameters()
                            .getSupportedPreviewSizes();
            // Finds the biggest resolution which fits the screen.
            // Else, returns the first resolution found.
            Size selectedSize = mCamera.new Size(0, 0);
            for (Size size : sizes) {
                if ((size.width <= pWidth)
                 && (size.height <= pHeight)
                 && (size.width >= selectedSize.width)
                 && (size.height >= selectedSize.height)) {
                    selectedSize = size;
                }
            }
            // Previous code assume that there is a preview size smaller
            // than screen size. If not, hopefully the Android API
            // guarantees that at least one preview size is available.
            if ((selectedSize.width == 0) || (selectedSize.height == 0)) {
                selectedSize = sizes.get(0);
            }
            return selectedSize;
        }
    ...
    ```

1.  当`TextureView`表面在`onSurfaceTextureDestroyed()`中被销毁时，释放相机，因为这是一个共享资源。位图缓冲区也可以被回收和置空，以减轻垃圾收集器的工作。

    ```java
    ...
        @Override
        public boolean onSurfaceTextureDestroyed(SurfaceTexture pSurface)
        {
            // Releases camera which is a shared resource.
            if (mCamera != null) {
                mCamera.stopPreview();
                mCamera.release();
                // These variables can take a lot of memory. Get rid of
                // them as fast as we can.
                mCamera = null;
                mVideoSource = null;
                mImageR.recycle(); mImageR = null;
                mImageG.recycle(); mImageG = null;
                mImageB.recycle(); mImageB = null;
            }
            return true;
        }
    ...
    ```

1.  最后，在`onPreviewFrame()`中解码原始视频帧。每次有新帧准备好时，由`Camera`类触发此处理程序。

    原始视频字节传递给本地方法`decode()`，以及支持的位图，并选择每个颜色分量的过滤器。

    解码完成后，使表面无效以重新绘制它。

    最后，"重新入队"原始视频缓冲区以请求捕获新的视频帧。

    ```java
    ...
        @Override
        public void onPreviewFrame(byte[] pData, Camera pCamera) {
            // New data has been received from camera. Processes it and
            // requests surface to be redrawn right after.
            if (mCamera != null) {
                decode(mImageR, pData, 0xFFFF0000);
                decode(mImageG, pData, 0xFF00FF00);
                decode(mImageB, pData, 0xFF0000FF);
                mImageViewR.invalidate();
                mImageViewG.invalidate();
                mImageViewB.invalidate();

                mCamera.addCallbackBuffer(mVideoSource);
            }
        }

        public native void decode(Bitmap pTarget, byte[] pSource,
                                  int pFilter);
    }
    ```

## *刚才发生了什么？*

通过 Android Camera API，我们从设备的相机捕获了实时图像。在设置相机捕获格式和定义之后，我们创建了所有必要的捕获缓冲区和输出图像以在屏幕上显示。当应用程序需要新帧时，捕获内容被保存在由应用程序入队的缓冲区中。然后，这个缓冲区与位图一起被传递给本地方法，我们将在下一节中编写它。最后，输出图像显示在屏幕上。

视频馈送以 YUV NV21 格式编码。YUV 是一种最初在电子时代的早期发明的颜色格式，以使黑白视频接收器与彩色传输兼容，现在仍然被广泛使用。Android 规范保证默认帧格式为**YCbCr 420 SP**（或**NV21**）。

### 提示

尽管 YCbCr 420 SP 是 Android 上的默认视频格式，但模拟器只支持 YCbCr 422 SP。这个缺陷基本上是颜色交换，不应该造成太大麻烦。在真实设备上不会出现这个问题。

既然我们的实时图像已经被捕获，让我们在本地处理它。

# 动手实践时间——使用 Bitmap API 处理图片

让我们继续通过颜色通道在本地端解码和过滤图像：

1.  创建本地 C 源文件，`jni/CameraDecoder.c`（不是 C++文件，这样我们可以看到与用 C++编写的 JNI 代码的区别）。

    包含`android/bitmap.h`，它定义了 NDK 位图处理 API 和`stdlib.h`（不是`cstdlib`，因为此文件是用 C 编写的）：

    ```java
    #include <android/bitmap.h>
    #include <stdlib.h>
    ...
    ```

    编写一些实用宏以帮助解码视频。

    +   `toInt()`函数将 jbyte 转换为整数，使用掩码擦除所有无用的位。

    +   `max()`函数获取两个值中的最大值。

    +   `clamp()`函数将值限制在定义的区间内。

    +   `color()`从每个颜色分量构建一个 ARGB 颜色。

        ```java
        ...
        #define toInt(pValue) \
            (0xff & (int32_t) pValue)
        #define max(pValue1, pValue2) \
            (pValue1 < pValue2) ? pValue2 : pValue1
        #define clamp(pValue, pLowest, pHighest) \
            ((pValue < 0) ? pLowest : (pValue > pHighest) ? pHighest : pValue)
        #define color(pColorR, pColorG, pColorB) \
            (0xFF000000 | ((pColorB << 6)  & 0x00FF0000) \
                        | ((pColorG >> 2)  & 0x0000FF00) \
                        | ((pColorR >> 10) & 0x000000FF))
        ...
        ```

1.  实现`decode()`本地方法。

    首先，获取位图信息并检查其像素格式是否为 32 位 RGBA。然后，锁定它以允许绘图操作。

    之后，使用`GetPrimitiveArrayCritical()`获取作为 Java 字节数组传递的输入视频帧内容：

    ```java
    ...
    void JNICALL decode(JNIEnv * pEnv, jclass pClass, jobject pTarget,
            jbyteArray pSource, jint pFilter) {
        // Retrieves bitmap information and locks it for drawing.
        AndroidBitmapInfo bitmapInfo;
        uint32_t* bitmapContent;
        if (AndroidBitmap_getInfo(pEnv,pTarget, &bitmapInfo) < 0) abort();
        if (bitmapInfo.format != ANDROID_BITMAP_FORMAT_RGBA_8888) abort();
        if (AndroidBitmap_lockPixels(pEnv, pTarget,
                (void**)&bitmapContent) < 0) abort();

        // Accesses source array data.
        jbyte* source = (*pEnv)->GetPrimitiveArrayCritical(pEnv,
                pSource, 0);
        if (source == NULL) abort();
        ...
    ```

1.  将原始视频帧解码为输出位图。视频帧以 YUV 格式编码，这与 RGB 有很大不同。YUV 格式以三个分量编码颜色：

    +   一个亮度分量，即颜色的灰度表示。

    +   两个色度分量，它们编码颜色信息（也称为**Cb**和**Cr**，因为它们代表蓝色差和红色差）。

    +   有许多基于 YUV 颜色的帧格式。这里，我们按照 YCbCr 420 SP（或 NV21）格式转换帧。这种图像帧由一个 8 位 Y 亮度样本缓冲区组成，后面跟着一个交错的 8 位 V 和 U 色度样本缓冲区。VU 缓冲区是子采样的，这意味着与 Y 样本相比，U 和 V 样本较少（对于 4 个 Y 样本，有 1 个 U 样本和 1 个 V 样本）。以下算法处理每个像素，并使用适当的公式将每个 YUV 像素转换为 RGB（更多信息请参见`http://www.fourcecc.org/fccyvrgb.php`）：

        ```java
        ...
            int32_t frameSize = bitmapInfo.width * bitmapInfo.height;
            int32_t yIndex, uvIndex, x, y;
            int32_t colorY, colorU, colorV;
            int32_t colorR, colorG, colorB;
            int32_t y1192;

            // Processes each pixel and converts YUV to RGB color.
            // Algorithm originates from the Ketai open source project.
            // See http://ketai.googlecode.com/.
            for (y = 0, yIndex = 0; y < bitmapInfo.height; ++y) {
                colorU = 0; colorV = 0;
                // Y is divided by 2 because UVs are subsampled vertically.
                // This means that two consecutives iterations refer to the
                // same UV line (e.g when Y=0 and Y=1).
                uvIndex = frameSize + (y >> 1) * bitmapInfo.width;

                for (x = 0; x < bitmapInfo.width; ++x, ++yIndex) {
                    // Retrieves YUV components. UVs are subsampled
                    // horizontally too, hence %2 (1 UV for 2 Y).
                    colorY = max(toInt(source[yIndex]) - 16, 0);
                    if (!(x % 2)) {
                        colorV = toInt(source[uvIndex++]) - 128;
                        colorU = toInt(source[uvIndex++]) - 128;
                    }

                    // Computes R, G and B from Y, U and V.
                    y1192 = 1192 * colorY;
                    colorR = (y1192 + 1634 * colorV);
                    colorG = (y1192 - 833  * colorV - 400 * colorU);
                    colorB = (y1192 + 2066 * colorU);

                    colorR = clamp(colorR, 0, 262143);
                    colorG = clamp(colorG, 0, 262143);
                    colorB = clamp(colorB, 0, 262143);

                    // Combines R, G, B and A into the final pixel color.
                    bitmapContent[yIndex] = color(colorR,colorG,colorB);
                    bitmapContent[yIndex] &= pFilter;
                }
            }
            ...
        ```

    最后，释放之前获取的 Java 字节缓冲区并解锁背后的位图。

    ```java
        ...
        (*pEnv)-> ReleasePrimitiveArrayCritical(pEnv,pSource,source,0);
        if (AndroidBitmap_unlockPixels(pEnv, pTarget) < 0) abort();
    }
    ...
    ```

1.  JNI 允许在`JNI_OnLoad()`中手动注册本地方法，而不是依赖命名约定来查找本地方法。

    因此，定义一个表来描述要注册其名称、签名和地址的本地方法。这里，只需指定`decode()`。

    然后，在`JNI_OnLoad()`中，找到声明本地方法`decode()`的 Java（这里是`LiveCameraActivity`），并使用`RegisterNatives()`告诉 JNI 使用哪个方法：

    ```java
    ...
    static JNINativeMethod gMethodRegistry[] = {
      { "decode", "(Landroid/graphics/Bitmap;[BI)V", (void *) decode }
    };
    static int gMethodRegistrySize = sizeof(gMethodRegistry)
                                   / sizeof(gMethodRegistry[0]);

    JNIEXPORT jint JNI_OnLoad(JavaVM* pVM, void* reserved) {
        JNIEnv *env;
        if ((*pVM)->GetEnv(pVM, (void**) &env, JNI_VERSION_1_6) != JNI_OK)
        { abort(); }

        jclass LiveCameraActivity = (*env)->FindClass(env,
                "com/packtpub/livecamera/LiveCameraActivity");
        if (LiveCameraActivity == NULL) abort();
        (*env)->RegisterNatives(env, LiveCameraActivity,
                gMethodRegistry, 1);
        (*env)->DeleteLocalRef(env, LiveCameraActivity);

        return JNI_VERSION_1_6;
    }
    ```

1.  按照以下方式编写`Application.mk` makefile：

    ```java
    APP_PLATFORM := android-14
    APP_ABI := all
    ```

1.  按照以下方式编写`Android.mk` makefile（将其链接到定义 Android Bitmap API 的`jnigraphics`模块）：

    ```java
    LOCAL_PATH := $(call my-dir)

    include $(CLEAR_VARS)

    LOCAL_MODULE    := livecamera
    LOCAL_SRC_FILES := CameraDecoder.c
    LOCAL_LDLIBS    := -ljnigraphics

    include $(BUILD_SHARED_LIBRARY)
    ```

## *刚才发生了什么？*

编译并运行应用程序。未经任何转换，原始视频馈送显示在左上角。原始视频帧在本地代码中解码，并将每个颜色通道提取到三个 Java 位图中。这些位图显示在屏幕每个角的三个`ImageView`元素内。

![刚才发生了什么？](img/9645_04_03.jpg)

用于解码 YUV 帧的算法源自 Ketai 开源项目，这是一个针对 Android 的图像和传感器处理库。更多信息请访问[`ketai.googlecode.com/`](http://ketai.googlecode.com/)。请注意，YUV 到 RGB 是一项昂贵的操作，很可能会成为程序中的争议点（我们将在第十章，*使用 RenderScript 进行密集计算*中介绍的**RenderScript**可以在该任务中提供帮助）。

这里展示的代码远非最优（解码算法可以进行优化，使用多个缓冲区捕获的视频帧，可以减少内存访问，并且代码可以是多线程的），但它概述了如何使用 NDK 本地处理位图。

借助 Android NDK 位图 API，在`jnigraphics`模块中定义，本地代码可以直接访问位图表面。这个 API 可以看作是 JNI 的 Android 特定扩展，定义了以下方法：

+   `AndroidBitmap_getInfo()`用于获取位图信息。当出现问题时，返回值将为负数，否则为`0`：

    ```java
    int AndroidBitmap_getInfo(JNIEnv* env, jobject jbitmap,
                              AndroidBitmapInfo* info);
    ```

+   位图信息在`AndroidBitmapInfo`结构中获取，定义如下：

    ```java
    typedef struct {
        uint32_t    width;  // Width in pixels
        uint32_t    height; // Height in pixels
        uint32_t    stride; // Number of bytes between each line
        int32_t     format; // Pixel structure (see AndroidBitmapFormat)
        uint32_t    flags;  // Unused for now
    } AndroidBitmapInfo;
    ```

+   `AndroidBitmap_lockPixels()`在处理位图时提供对其的独占访问。当出现问题时，返回值为负数，否则为`0`：

    ```java
    int AndroidBitmap_lockPixels(JNIEnv* env, jobject jbitmap, void** addrPtr);
    ```

+   `AndroidBitmap_unlockPixels()`释放对位图的独占锁定。当出现问题时，返回值为负数，否则为`0`：

    ```java
    int AndroidBitmap_unlockPixels(JNIEnv* env, jobject jbitmap);
    ```

对任何位图的绘制操作都系统地分为三个主要步骤：

1.  首先，获取位图表面。

1.  然后，修改位图像素。在这里，视频像素被转换为 RGB 并写入位图表面。

1.  最后，释放位图表面。

在本地访问位图时，必须系统地锁定位图并在访问后解锁。绘制操作必须在锁定/解锁对之间强制执行。更多信息请查看`bitmap.h`头文件。

## 手动注册本地方法

在我们的商店示例中，本地方法原型已通过`Javah`使用特定的名称和参数约定自动生成。然后，Dalvik VM 可以在运行时通过“猜测”它们的名称来加载它们。然而，这种约定很容易被打破，并且在运行时没有灵活性。幸运的是，JNI 允许您手动注册将从 Java 中调用的本地方法。还有比`JNI_OnLoad()`更好的地方来做这件事吗？

注册是通过以下 JNI 方法完成的：

```java
jint RegisterNatives(jclass clazz, const JNINativeMethod* methods,
                     jint nMethods)
```

+   `jclass`是对托管本地方法的 Java 类的引用。我们将在本章和下一章中更详细地了解它。

+   `methods`是一个`JNINativeMethod`数组，该结构描述了要注册的本地方法。

+   `nMethods`表示`methods`数组内描述的方法数量。

`JNINativeMethod`结构定义如下：

```java
typedef struct {
    const char* name;
    const char* signature;
    void*       fnPtr;
} JNINativeMethod;
```

第一个和第二个元素是对应 Java 方法的`name`和`signature`，第三个参数`fnPtr`，是指向原生侧对应方法的指针。这样，你可以摆脱`javah`及其讨厌的命名约定，并在运行时选择要调用的方法。

## C 中的 JNI 与 C++中的 JNI 对比。

NDK 允许用 C（如我们的`LiveCamera`示例）或 C++（如我们的`Store`示例）编写应用程序。JNI 也是如此。

C 不是一种面向对象的语言，但 C++是。这就是为什么你不能像在 C++中那样在 C 中编写 JNI。在 C 中，`JNIEnv`实际上是一个包含函数指针的结构。当然，当你得到`JNIEnv`时，所有这些指针都已初始化，你可以像使用对象一样调用它们。然而，这个在面向对象语言中隐含的参数，在 C 中作为第一个参数给出（以下代码中的`env`）。此外，首次运行方法时需要取消引用`JNIEnv`：

```java
JNIEnv *env = ...;
(*env)->RegisterNative(env, ...); 
```

C++代码更自然、更简单。这个参数是隐式的，无需取消引用`JNIEnv`，因为方法不再声明为函数指针，而是作为真正的成员方法：

```java
JNIEnv *env = ...;
env->RegisterNative(env, ...); 
```

因此，尽管非常相似，但你不能以完全相同的方式在 C 中编写 JNI 代码，就像在 C++中编写一样。

# 总结

得益于 JNI，Java 和 C/C++可以紧密集成在一起。Android 现在完全双语化了！Java 可以使用任何类型的数据或对象调用 C/C++代码，原生代码也可以回调 Java。

我们还发现了如何使用 JNI 反射 API 从原生代码调用 Java 代码。实际上，几乎任何 Java 操作都可以通过它从原生代码执行。然而，为了最佳性能，类、方法或字段描述符必须被缓存。

我们还了解了如何将线程附加到虚拟机，并使用 JNI 监视器同步 Java 和原生线程。多线程代码可能是编程中最困难的主题之一。要谨慎处理！

最后，我们通过 JNI 原生处理了位图，并手动解码了视频流。然而，从默认的 YUV 格式（根据 Android 规范，每个设备都应该支持）到 RGB 的转换成本较高。

在处理 Android 上的原生代码时，几乎总是离不开 JNI。它是一个冗长且技术性很强的 API，更不用说它还繁琐，需要小心处理。要深入理解它的细微之处，可能需要一整本书的篇幅。而本章则为你提供了将你自己的 C/C++模块集成到 Java 应用程序中的基本知识。

在下一章，我们将看到如何创建一个完全原生的应用程序，它完全摆脱了 JNI。
