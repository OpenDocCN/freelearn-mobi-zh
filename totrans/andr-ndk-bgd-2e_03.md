# 第三章 用 JNI 实现 Java 与 C/C++的接口

> \*Android 与 Java 密不可分。其内核和核心库是原生的，但 Android 应用框架几乎完全是用 Java 编写的，或者至少在 Java 的薄层中包装。不要期望直接在 C/C++中构建你的 Android GUI！大多数 API 只能从 Java 访问。最多，我们可以将其隐藏在封面下... 因此，如果无法将 Java 和 C/C++连接在一起，Android 上的原生 C/C++代码将毫无意义。\
> 
> \*这个角色是专门为 Java Native Interface API 准备的。JNI 是一个标准化规范，允许 Java 调用原生代码，原生代码也可以回调 Java。它是 Java 和原生代码之间的双向桥梁；将 C/C++的强大功能注入你的 Java 应用程序的唯一方式。\
> 
> \*得益于 JNI，人们可以像调用任何 Java 方法一样从 Java 调用 C/C++函数，将 Java 原始类型或对象作为参数传递，并将它们作为原生调用的结果接收。反之，原生代码可以通过类似反射的 API 访问、检查、修改 Java 对象或抛出异常。JNI 是一个需要小心使用的微妙框架，任何误用都可能导致灾难性的结局…\

在本章中，我们将实现一个基本的关键/值存储来处理各种数据类型。一个简单的 Java GUI 将允许定义一个由键（字符串）、类型（整数、字符串等）和与选定类型相关的值组成的*条目*。条目在固定大小的条目数组中检索、插入或更新（不支持删除），该数组将驻留在原生侧。

为了实现这个项目，我们将要：

+   初始化一个原生的 JNI 库

+   在原生代码中转换 Java 字符串

+   将 Java 原始数据传递给原生代码

+   在原生代码中处理 Java 对象引用

+   在原生代码中管理 Java 数组

+   在原生代码中引发和检查 Java 异常。

在本章结束时，你应该能够使用任何 Java 类型进行原生调用并使用异常。

JNI 是一个非常技术性的框架，需要小心使用，因为任何误用都可能导致灾难性的结局。本章并不试图详尽无遗地介绍它，而是专注于桥接 Java 和 C++之间差距的基本知识。

# 初始化一个原生的 JNI 库

在访问它们的原生方法之前，必须通过 Java 调用`System.loadLibrary()`来加载原生库。JNI 提供了一个钩子`JNI_OnLoad()`，以便插入你自己的初始化代码。让我们重写它以初始化我们的原生存储。

### 注意

本书提供了名为`Store_Part4`的项目作为结果。

# 动手实践——定义一个简单的 GUI

让我们为我们的`Store`创建一个 Java 图形用户界面，并将其绑定到我们将要创建的原生存储结构：

1.  重写`res/fragment_layout.xml`布局以定义如下图形界面。它定义了：

    +   一个**键** `TextView`标签和`EditText`以输入键

    +   一个**值** `TextView`标签和`EditText`以输入与键匹配的值

    +   一个**类型** `TextView` 标签和 `Spinner` 以定义值的类型

    +   一个**获取值**和一个**设置值**的 `Button` 以在存储中检索和更改值

        ```java
        <LinearLayout 

          a:layout_width="match_parent" a:layout_height="match_parent"
          a:orientation="vertical"
        tools:context="com.packtpub.store.StoreActivity$PlaceholderFragment">
          <TextView
            a:layout_width="match_parent" a:layout_height="wrap_content"
            a:text="Save or retrieve a value from the store:" />
          <TableLayout
            a:layout_width="match_parent" a:layout_height="wrap_content"
            a:stretchColumns="1" >
            <TableRow>
              <TextView a:id="@+id/uiKeyLabel" a:text="Key : " />
              <EditText a:id="@+id/uiKeyEdit" ><requestFocus /></EditText>
            </TableRow>
            <TableRow>
              <TextView a:id="@+id/uiValueLabel" a:text="Value : " />
              <EditText a:id="@+id/uiValueEdit" />
            </TableRow>
            <TableRow>
              <TextView a:id="@+id/uiTypeLabel" a:layout_height="match_parent"
                        a:gravity="center_vertical" a:text="Type : " />
              <Spinner a:id="@+id/uiTypeSpinner" />
            </TableRow>
          </TableLayout>
          <LinearLayout
            a:layout_width="wrap_content" a:layout_height="wrap_content"
            a:layout_gravity="right" >
            <Button a:id="@+id/uiGetValueButton" a:layout_width="wrap_content"
                    a:layout_height="wrap_content" a:text="Get Value" />
            <Button a:id="@+id/uiSetValueButton" a:layout_width="wrap_content"
                    a:layout_height="wrap_content" a:text="Set Value" />
          </LinearLayout>
        </LinearLayout>
        ```

    最终结果应如下所示：

    ![动手操作——定义一个简单的 GUI](img/9645_03_02.jpg)

1.  在 `StoreType.java` 中创建一个新的类，带有一个空的枚举：

    ```java
    package com.packtpub.store;

    public enum StoreType {
    }
    ```

1.  GUI 和本地存储需要绑定在一起。这是由 `StoreActivity` 类承担的角色。为此，当在 `onCreateView()` 中创建 `PlaceholderFragment` 时，初始化布局文件中先前定义的所有 GUI 组件：

    ```java
    public class StoreActivity extends Activity {
        ...
        public static class PlaceholderFragment extends Fragment {
            private Store mStore = new Store();
     private EditText mUIKeyEdit, mUIValueEdit;
     private Spinner mUITypeSpinner;
            private Button mUIGetButton, mUISetButton;
            private Pattern mKeyPattern;

            ...

            @Override
            public View onCreateView(LayoutInflater inflater,
                                     ViewGroup container,
                                     Bundle savedInstanceState)
            {
                View rootView = inflater.inflate(R.layout.fragment_store,
                                                 container, false);
                updateTitle();

     // Initializes text components.
     mKeyPattern = Pattern.compile("\\p{Alnum}+");
     mUIKeyEdit = (EditText) rootView.findViewById(
     R.id.uiKeyEdit);
     mUIValueEdit = (EditText) rootView.findViewById(
     R.id.uiValueEdit);

    ```

1.  `Spinner` 内容绑定到 `StoreType` 枚举。使用 `ArrayAdapter` 将 `Spinner` 和 `enum` 值绑定在一起。

    ```java
                ...
     ArrayAdapter<StoreType> adapter =
     new ArrayAdapter<StoreType>(getActivity(),
     android.R.layout.simple_spinner_item,
     StoreType.values());
     adapter.setDropDownViewResource(
     android.R.layout.simple_spinner_dropdown_item);
     mUITypeSpinner = (Spinner) rootView.findViewById(
     R.id.uiTypeSpinner);
     mUITypeSpinner.setAdapter(adapter);
                    ...
    ```

1.  **获取值**和**设置值**按钮触发私有方法 `onGetValue()` 和 `onSetValue()`，它们分别从存储中拉取数据和向存储推送数据。使用 `OnClickListener` 将按钮和方法绑定在一起：

    ```java
                ...
     mUIGetButton = (Button) rootView.findViewById(
     R.id.uiGetValueButton);
     mUIGetButton.setOnClickListener(new OnClickListener() {
     public void onClick(View pView) {
     onGetValue();
     }
     });
     mUISetButton = (Button) rootView.findViewById(
     R.id.uiSetValueButton);
     mUISetButton.setOnClickListener(new OnClickListener() {
     public void onClick(View pView) {
     onSetValue();
     }
     });
                return rootView;
            }
            ...
    ```

1.  在 `PlaceholderFragment` 中，定义 `onGetValue()` 方法，该方法将根据 GUI 中选择的 `StoreType` 从存储中检索条目。现在先让 switch 语句为空，因为它暂时不会处理任何类型的条目：

    ```java
            ...
            private void onGetValue() {
                // Retrieves key and type entered by the user.
                String key = mUIKeyEdit.getText().toString();
                StoreType type = (StoreType) mUITypeSpinner
                                                       .getSelectedItem();
                // Checks key is correct.
                if (!mKeyPattern.matcher(key).matches()) {
                    displayMessage("Incorrect key.");
                    return;
                }

                // Retrieves value from the store and displays it.
                // Each data type has its own access method.
                switch (type) {
                    // Will retrieve entries soon...
                }
            }
            ...
    ```

1.  然后，在 `PlaceholderFragment` 中，定义 `StoreActivity` 的 `onSetValue()` 方法，以在存储中插入或更新条目。如果值格式不正确，将显示一条消息：

    ```java
            ...
            private void onSetValue() {
                // Retrieves key and type entered by the user.
                String key = mUIKeyEdit.getText().toString();
                String value = mUIValueEdit.getText().toString();
                StoreType type = (StoreType) mUITypeSpinner
                                                       .getSelectedItem();
                // Checks key is correct.
                if (!mKeyPattern.matcher(key).matches()) {
                    displayMessage("Incorrect key.");
                    return;
                }

                // Parses user entered value and saves it in the store.
                // Each data type has its own access method.
                try {
                    switch (type) {
                        // Will put entries soon...
                    }
                } catch (Exception eException) {
                    displayMessage("Incorrect value.");
                }
                updateTitle();
            }
            ...
    ```

1.  最后，`PlaceholderFragment` 中的一个小助手方法 `displayMessage()` 将帮助在出现问题时警告用户。它显示一个简单的 Android Toast 消息：

    ```java
            ...
            private void displayMessage(String pMessage) {
                Toast.makeText(getActivity(), pMessage, Toast.LENGTH_LONG)
                     .show();
            }
        }
    }
    ```

## *刚才发生了什么？*

我们使用 Android 框架的几个视觉组件在 Java 中创建了一个基本的图形用户界面。如您所见，这里没有 NDK 的特定内容。故事的核心是本地代码可以与任何现有的 Java 代码集成。

显然，我们还需要做些工作，让我们的本地代码为 Java 应用程序执行一些有用的操作。现在让我们切换到本地端。

# 动手操作时间——初始化本地存储

我们需要创建并初始化我们将在本章下一部分使用的所有结构：

1.  创建 `jni/Store.h` 文件，该文件定义了存储数据结构：

    +   `StoreType` 枚举将反映相应的 Java 枚举。现在先让它为空。

    +   `StoreValue` 联合体将包含可能的存储值中的任何一个。现在也先让它为空。

    +   `StoreEntry` 结构包含存储中的一条数据。它由一个键（由 `char*` 制作的原始 C 字符串）、一个类型（`StoreType`）和一个值（`StoreValue`）组成。

        ### 注意

        请注意，我们将在第九章，*将现有库移植到 Android*中了解如何设置和使用 C++ STL 字符串。

    +   `Store` 是一个主要结构，定义了一个固定大小的条目数组和长度（即已分配的条目数）：

        ```java
        #ifndef _STORE_H_
        #define _STORE_H_

        #include <cstdint>

        #define STORE_MAX_CAPACITY 16

        typedef enum {
        } StoreType;

        typedef union {
        } StoreValue;

        typedef struct {
            char* mKey;
            StoreType mType;
            StoreValue mValue;
        } StoreEntry;

        typedef struct {
            StoreEntry mEntries[STORE_MAX_CAPACITY];
            int32_t mLength;
        } Store;
        #endif
        ```

        ### 提示

        包含保护（即`#ifndef`, `#define`, 和 `#endif`），它们确保头文件在编译期间只被包含一次，可以用非标准（但广泛支持的）预处理器指令`#pragma once`来替换。

1.  在`jni/com_packtpub_Store.cpp`中，实现`JNI_OnLoad()`初始化钩子。在内部，将`Store`数据结构的唯一实例初始化为一个静态变量：

    ```java
    #include "com_packtpub_store_Store.h"
    #include "Store.h"

    static Store gStore;

    JNIEXPORT jint JNI_OnLoad(JavaVM* pVM, void* reserved) {
     // Store initialization.
     gStore.mLength = 0;
     return JNI_VERSION_1_6;
    }
    ...
    ```

1.  相应地更新本地`store getCount()`方法，以反映分配给商店的条目数量：

    ```java
    ...
    JNIEXPORT jint JNICALL Java_com_packtpub_store_Store_getCount
      (JNIEnv* pEnv, jobject pObject) {
     return gStore.mLength;
    }
    ```

## *刚才发生了什么？*

我们用简单的 GUI 和本地内存中的数据数组构建了商店项目的基石。包含的本地库可以通过以下调用加载：

+   `System.load()`，它接收库的全路径作为参数。

+   `System.loadLibrary()`，它只需要库名称，不需要路径、前缀（即`lib`）或扩展名。

本地代码初始化在`JNI_OnLoad()`钩子中发生，该钩子在本地代码的生命周期内只被调用一次。这是初始化和缓存全局变量的完美位置。JNI 元素（类、方法、字段等）也经常在`JNI_OnLoad()`中被缓存，以提高性能。我们将在本章和下一章中了解更多相关信息。

请注意，在 Android 中，由于无法保证在进程终止之前卸载库，因此在 JNI 规范中定义的挂起调用`JNI_OnUnload()`几乎是没用的。

`JNI_OnLoad()`签名被系统地定义如下：

```java
JNIEXPORT jint JNICALL JNI_OnLoad(JavaVM* vm, void* reserved);
```

使得`JNI_OnLoad()`如此有用的原因是它的`JavaVM`参数。通过它，你可以按照以下方式检索**JNIEnv 接口指针**：

```java
JNIEXPORT jint JNI_OnLoad(JavaVM* pVM, void* reserved) {
 JNIEnv *env;
 if (pVM->GetEnv((void**) &env, JNI_VERSION_1_6) != JNI_OK) {
 abort();
    }
    ...
    return JNI_VERSION_1_6;
}
```

### 提示

在 JNI 库中的`JNI_OnLoad()`定义是可选的。但是，如果省略它，你可能会在启动应用程序时在**Logcat**中看到警告**No JNI_OnLoad found in <mylib>.so**。这绝对没有后果，可以安全地忽略。

`JNIEnv`是所有 JNI 调用的主要入口点，这就是为什么它会被传递给所有本地方法的原因。它提供了一系列方法，以便从本地代码访问 Java 原始类型和数组。它还通过类似反射的 API，使本地代码能够完全访问 Java 对象。我们将在本章和下一章中更详细地了解其特性。

### 提示

`JNIEnv`接口指针是线程特定的。你绝对不能在线程之间共享它！只能在获取它的线程上使用它。只有 JavaVM 元素是线程安全的，可以在线程之间共享。

# 在本地代码中转换 Java 字符串

我们将处理的第一种条目是字符串。字符串在 Java 中作为（几乎）经典的对象表示，可以通过 JNI 在本地端操作并转换为本地字符串，即原始字符数组。尽管字符串由于其异构表示的复杂性而显得复杂，但它们是一等公民。

在这一部分，我们将把 Java 字符串发送到原生端，并将其转换为对应的原生字符串。我们还会将它们重新转换回 Java 字符串。

### 注意

本书提供了名为`Store_Part5`的项目，其中包含此结果。

# 行动时间——处理原生存储中的字符串

让我们处理存储中的字符串值：

1.  打开`StoreType.java`并在枚举中指定我们存储处理的新字符串类型：

    ```java
    public enum StoreType {
     String
    }
    Open Store.java and define the new functionalities our native key/value store provides (for now, only strings):
    public class Store {
        ...
        public native int getCount();

     public native String getString(String pKey);
     public native void setString(String pKey, String pString);
    }
    ```

1.  在`StoreActivity.java`中，在`onGetValue()`方法中从原生`Store`获取字符串条目。根据当前在 GUI 中选定的`StoreType`类型进行操作（尽管目前只有一个可能的类型）：

    ```java
    public class StoreActivity extends Activity {
        ...
        public static class PlaceholderFragment extends Fragment {
            ...
            private void onGetValue() {
                ...
                switch (type) {
     case String:
     mUIValueEdit.setText(mStore.getString(key));
     break;
                }
            }
            ...
    ```

1.  在`onSetValue()`方法中插入或更新存储中的字符串条目：

    ```java
            ...
            private void onSetValue() {
                ...
                try {
                    switch (type) {
     case String:
     mStore.setString(key, value);
     break;
                    }
                } catch (Exception eException) {
                    displayMessage("Incorrect value.");
                }
                updateTitle();
            }
            ...
        }
    }
    ```

1.  在`jni/Store.h`中，包含一个新的`header jni.h`以访问 JNI API。

    ```java
    #ifndef _STORE_H_
    #define _STORE_H_

    #include <cstdint>
    #include "jni.h"
    ...
    ```

1.  接下来，将字符串集成到原生的`StoreType`枚举和`StoreValue`联合体中：

    ```java
    ...
    typedef enum {
     StoreType_String
    } StoreType;

    typedef union {
     char*     mString;
    } StoreValue;
    ...
    ```

1.  通过声明用于检查、创建、查找和销毁条目的实用方法来结束。`JNIEnv`和`jstring`是在`jni.h`头文件中定义的 JNI 类型：

    ```java
    ...
    bool isEntryValid(JNIEnv* pEnv, StoreEntry* pEntry, StoreType pType);

    StoreEntry* allocateEntry(JNIEnv* pEnv, Store* pStore, jstring pKey);

    StoreEntry* findEntry(JNIEnv* pEnv, Store* pStore, jstring pKey);

    void releaseEntryValue(JNIEnv* pEnv, StoreEntry* pEntry);
    #endif
    ```

1.  创建一个新文件`jni/Store.cpp`以实现所有这些实用方法。首先，`isEntryValid()`仅检查条目是否已分配并具有预期的类型：

    ```java
    #include "Store.h"
    #include <cstdlib>
    #include <cstring>

    bool isEntryValid(JNIEnv* pEnv, StoreEntry* pEntry, StoreType pType) {
        return ((pEntry != NULL) && (pEntry->mType == pType));
    }
    ...
    ```

1.  `findEntry()`方法通过将传入的参数与存储中的每个键进行比较，直到找到匹配项。它不使用传统的原生字符串（即`char*`），而是接收一个`jstring`参数，这是在原生端对 Java `String`的直接表示。

1.  要从 Java `String`中恢复原生字符串，请使用 JNI API 中的`GetStringUTFChars()`获取一个临时字符缓冲区，其中包含转换后的 Java 字符串。然后可以使用标准的 C 语言例程操作其内容。`GetStringUTFChars()`必须与`ReleaseStringUTFChars()`的调用配对，以释放在`GetStringUTFChars()`中分配的临时缓冲区：

    ### 提示

    Java 字符串在内存中以 UTF-16 字符串的形式存储。当在原生代码中提取其内容时，返回的缓冲区以修改后的 UTF-8 编码。修改后的 UTF-8 与标准 C 字符串函数兼容，后者通常在由 8 位每个字符组成的字符串缓冲区上工作。

    ```java
    ...
    StoreEntry* findEntry(JNIEnv* pEnv, Store* pStore, jstring pKey) {
        StoreEntry* entry = pStore->mEntries;
        StoreEntry* entryEnd = entry + pStore->mLength;

        // Compare requested key with every entry key currently stored
        // until we find a matching one.
        const char* tmpKey = pEnv->GetStringUTFChars(pKey, NULL);
        while ((entry < entryEnd) && (strcmp(entry->mKey, tmpKey) != 0)) {
            ++entry;
        }
        pEnv->ReleaseStringUTFChars(pKey, tmpKey);

        return (entry == entryEnd) ? NULL : entry;
    }
    ...
    ```

    ### 提示

    JNI 不会原谅任何错误。例如，如果你在`GetStringUTFChars()`中将`NULL`作为第一个参数传递，虚拟机将立即终止。此外，Android JNI 并不完全遵守 JNI 规范。尽管 JNI 规范指出，如果无法分配内存，`GetStringUTFChars()`可能会返回`NULL`，但在这种情况下，Android VM 会直接终止。

1.  实现`allocateEntry()`，该方法要么创建一个新的条目（即增加存储长度并返回最后一个元素），要么如果键已存在则释放其先前值后返回现有条目。

    如果条目是新的，请将其键转换为可以在内存中保留的原生字符串。实际上，原始 JNI 对象在其方法调用的持续时间内存在，并且不能在其作用域之外保留：

    ```java
    ...
    StoreEntry* allocateEntry(JNIEnv* pEnv, Store* pStore, jstring pKey) {
        // If entry already exists in the store, releases its content
        // and keep its key.
        StoreEntry* entry = findEntry(pEnv, pStore, pKey);
        if (entry != NULL) {
            releaseEntryValue(pEnv, entry);
        }
        // If entry does not exist, create a new entry
        // right after the entries already stored.
        else {
            entry = pStore->mEntries + pStore->mLength;

            // Copies the new key into its final C string buffer.
            const char* tmpKey = pEnv->GetStringUTFChars(pKey, NULL);
            entry->mKey = new char[strlen(tmpKey) + 1];
            strcpy(entry->mKey, tmpKey);
            pEnv->ReleaseStringUTFChars(pKey, tmpKey);

            ++pStore->mLength;
        }
        return entry;
    }
    ...
    ```

1.  编写最后一个方法`releaseEntryValue()`，该方法在需要时释放为值分配的内存：

    ```java
    ...
    void releaseEntryValue(JNIEnv* pEnv, StoreEntry* pEntry) {
        switch (pEntry->mType) {
        case StoreType_String:
            delete pEntry->mValue.mString;
            break;
        }
    }
    ```

1.  使用上一章中看到的`javah`刷新 JNI 头文件`jni/com_packtpub_Store.h`。你应在其中看到两个新方法`Java_com_packtpub_store_Store_getString()`和`Java_com_packtpub_store_Store_setString()`。

1.  在`jni/com_packtpub_Store.cpp`中，插入`cstdlib`头文件：

    ```java
    #include "com_packtpub_store_Store.h"
    #include <cstdlib>
    #include "Store.h"
    ...
    ```

1.  借助之前生成的 JNI 头文件，实现原生方法`getString()`。此方法在存储区中查找传递的键并返回其对应的字符串值。如果出现任何问题，将返回默认的`NULL`值。

1.  Java 字符串并非真正的原始数据类型。我们之前已经看到，类型`jstring`和`char*`不能互换使用。要从原生字符串创建 Java `String`对象，请使用 JNI API 中的`NewStringUTF()`：

    ```java
    ...
    JNIEXPORT jstring JNICALL Java_com_packtpub_store_Store_getString
      (JNIEnv* pEnv, jobject pThis, jstring pKey) {
        StoreEntry* entry = findEntry(pEnv, &gStore, pKey);
        if (isEntryValid(pEnv, entry, StoreType_String)) {
            // Converts a C string into a Java String.
            return pEnv->NewStringUTF(entry->mValue.mString);
        } else {
            return NULL;
        }
    }
    ...
    ```

1.  然后，实现`setString()`方法，该方法分配一个条目（即，在存储区中创建一个新的条目，如果存在具有相同键的条目则重用），并将转换后的 Java 字符串值存储在其中。

1.  字符串值使用 JNI API 的`GetStringUTFLength()`和`GetStringUTFRegion()`方法直接从 Java 字符串翻译到我们自己的字符串缓冲区。这是之前使用的`GetStringUTFChars()`的替代方法。最后，我们一定不要忘记添加`null`字符，这是原始 C 字符串的标准：

    ```java
    ...
    JNIEXPORT void JNICALL Java_com_packtpub_store_Store_setString
      (JNIEnv* pEnv, jobject pThis, jstring pKey, jstring pString) {
        // Turns the Java string into a temporary C string.
        StoreEntry* entry = allocateEntry(pEnv, &gStore, pKey);
        if (entry != NULL) {
            entry->mType = StoreType_String;
            // Copy the temporary C string into its dynamically allocated
            // final location. Then releases the temporary string.
            jsize stringLength = pEnv->GetStringUTFLength(pString);
            entry->mValue.mString = new char[stringLength + 1];
            // Directly copies the Java String into our new C buffer.
            pEnv->GetStringUTFRegion(pString, 0, stringLength,
                                     entry->mValue.mString);
            // Append the null character for string termination.
            entry->mValue.mString[stringLength] = '\0';    }
    }
    ```

1.  最后，更新`Android.mk`文件以编译`Store.cpp`：

    ```java
    LOCAL_PATH := $(call my-dir)

    include $(CLEAR_VARS)

    LOCAL_MODULE    := com_packtpub_store_Store
    LOCAL_SRC_FILES := com_packtpub_store_Store.cpp Store.cpp

    include $(BUILD_SHARED_LIBRARY)
    ```

## *刚才发生了什么？*

运行应用程序。尝试使用不同的键和值保存几个条目。然后尝试从原生存储区获取它们。我们已经实现了在 Java 和 C/C++之间传递和检索字符串。这些值作为原生字符串保存在原生内存中。然后可以根据其键从存储区将条目作为 Java 字符串检索。

Java 和 C 字符串是完全不同的。Java 字符串需要一个具体的转换，以原生字符串的形式允许使用标准的 C 字符串例程处理它们的内容。实际上，`jstring`不是经典的`char*`数组的表示，而是对 Java `String`对象的引用，只能从 Java 代码中访问。

在这一部分中，我们发现了两种将 Java 字符串转换为原生字符串的方法：

+   通过预先分配一个内存缓冲区，将转换后的 Java 字符串复制到其中。

+   通过在由 JNI 管理的内存缓冲区中检索转换后的 Java 字符串。

选择哪种解决方案取决于客户端代码如何处理内存。

## 原生字符编码

JNI 提供了两种处理字符串的方法：

+   名称中包含 UTF 且处理修改后的 UTF-8 字符串的那些方法

+   名称中不包含 UTF 且处理 UTF-16 编码的那些方法

修改后的 UTF-8 和 UTF-16 字符串是两种不同的字符编码：

+   **修改后的 UTF-8**是 Java 特有的轻微变体的 UTF-8。这种编码可以表示标准 ASCII 字符（每个字符一个字节）或者可以扩展到 4 个字节来表示扩展字符（阿拉伯语、西里尔语、希腊语、希伯来语等）。标准 UTF-8 和修改后的 UTF-8 之间的区别在于对`null`字符的不同表示，后者根本不存在这个字符。这样，这些字符串可以用标准的 C 例程处理，而 C 例程使用`null`字符作为结束标志。

+   **UTF-16**是真正用于 Java 字符串的编码。每个字符用两个字节表示，因此 Java `char`的大小如此。因此，在本地代码中使用 UTF-16 而不是修改后的 UTF-8 更有效率，因为它们不需要转换。缺点是，经典的 C 字符串例程无法处理它们，因为它们不是以`null`结尾的。

字符编码是一个复杂的主题，你可以访问[`www.oracle.com/technetwork/articles/javase/supplementary-142654.html`](http://www.oracle.com/technetwork/articles/javase/supplementary-142654.html)和[`developer.android.com/training/articles/perf-jni.html#UTF_8_and_UTF_16_strings`](http://developer.android.com/training/articles/perf-jni.html#UTF_8_and_UTF_16_strings)的 Android 文档获取更多信息。

## JNI 字符串 API

JNI 提供了几种方法来处理本地端的 Java 字符串：

+   `GetStringUTFLength()`计算修改后的 UTF-8 字符串的长度（以字节为单位，因为 UTF-8 字符串的字符大小不同），而`GetStringLength()`计算 UTF-16 字符串的字符数（不是字节，因为 UTF-16 字符的大小是固定的）：

    ```java
    jsize GetStringUTFLength(jstring string)
    jsize GetStringLength(jstring string)
    ```

+   `GetStringUTFChars()`和`GetStringChars()`通过 JNI 分配一个新的内存缓冲区，用于存储 Java 到本地（分别是修改后的 UTF-8 和 UTF-16）字符串转换的结果。当你想转换整个字符串而不想处理内存分配时，请使用它。最后一个参数`isCopy`，如果不为`null`，表示字符串是否被 JNI 内部复制，或者返回的缓冲区是否指向实际的 Java 字符串内存。在 Android 中，对于`GetStringUTFChars()`返回的`isCopy`值通常是`JNI_TRUE`，对于`GetStringChars()`则是`JNI_FALSE`（后者确实不需要编码转换）：

    ```java
    const char* GetStringUTFChars(jstring string, jboolean* isCopy)
    const jchar* GetStringChars(jstring string, jboolean* isCopy)
    ```

    ### 提示

    尽管 JNI 规范指出`GetStringUTFChars()`可能返回 NULL（这意味着操作可能因为例如无法分配内存而失败），但实际上，这种检查是没有用的，因为 Dalvik 或 ART VM 在这种情况下通常会终止。所以，尽量避免进入这种情况！如果你的代码旨在移植到其他 Java 虚拟机上，你仍然应该保留 NULL 检查。

+   `ReleaseStringUTFChars()`和`ReleaseStringChars()`方法用于释放`GetStringUTFChars()`和`GetStringChars()`分配的内存缓冲区，当客户端处理完毕后。这些方法必须始终成对调用：

    ```java
    void ReleaseStringUTFChars(jstring string, const char* utf)
    void ReleaseStringChars(jstring string, const jchar* chars)
    ```

+   `GetStringUTFRegion()`和`GetStringRegion()`获取 Java 字符串的全部或部分区域。它作用于由客户端代码提供和管理的字符串缓冲区。当您想要管理内存分配（例如，重用现有的内存缓冲区）或需要访问字符串的小部分时使用它：

    ```java
    void GetStringRegion(jstring str, jsize start, jsize len, jchar* buf)
    void GetStringUTFRegion(jstring str, jsize start, jsize len, char* buf)
    ```

+   `GetStringCritical()`和`ReleaseStringCritical()`与`GetStringChars()`和`ReleaseStringChars()`类似，但仅适用于 UTF-16 字符串。根据 JNI 规范，`GetStringCritical()`更有可能返回一个直接指针，而不进行任何复制。作为交换，调用者不得执行阻塞操作或 JNI 调用，并且不应长时间持有字符串（就像线程中的临界区）。实际上，Android 似乎不管你是否使用关键功能都表现相似（但这可能会改变）：

    ```java
    const jchar* GetStringCritical(jstring string, jboolean* isCopy)
    void ReleaseStringCritical(jstring string, const jchar* carray)
    ```

这是您需要了解的通过 JNI 处理 Java 字符串的基本知识。

# 将 Java 基本类型传递给本地代码

我们可以使用 JNI 处理的最简单的元素是 Java 基本类型。实际上，Java 端和本地端几乎使用相同的数据表示，这种数据不需要任何特定的内存管理。

在这一部分，我们将了解如何将整数传递到本地端，并将它们发送回 Java 端。

### 注意

本书提供的项目名为`Store_Part6`。

# 动手实践时间——在本地存储中处理基本类型。

1.  在`StoreType.java`中，将新管理的整数类型添加到枚举中：

    ```java
    public enum StoreType {
        Integer,
        String
    }
    ```

1.  打开`Store.java`文件，定义我们的本地存储提供的新整数功能：

    ```java
    public class Store {
        ...
        public native int getCount();

     public native int getInteger(String pKey);
     public native void setInteger(String pKey, int pInt);

        public native String getString(String pKey);
        public native void setString(String pKey, String pString);
    }
    ```

1.  在`StoreActivity`类中，更新`onGetValue()`方法，以便在 GUI 中选择整数条目时从存储中检索它们：

    ```java
    public class StoreActivity extends Activity {
        ...
        public static class PlaceholderFragment extends Fragment {
            ...
            private void onGetValue() {
                ...
                switch (type) {
     case Integer:
     mUIValueEdit.setText(Integer.toString(mStore
     .getInteger(key)));
     break;
                case String:
                    mUIValueEdit.setText(mStore.getString(key));
                    break;
                }
            }
            ...
    ```

1.  同时，在`onSetValue()`方法中插入或更新存储中的整数条目。在将条目数据传递到本地端之前，需要对其进行解析：

    ```java
            ...
            private void onSetValue() {
                ...
                try {
                    switch (type) {
     case Integer:
     mStore.setInteger(key, Integer.parseInt(value));
     break;
                    case String:
                        mStore.setString(key, value);
                        break;
                    }
                } catch (Exception eException) {
                    displayMessage("Incorrect value.");
                }
                updateTitle();
            }
            ...
        }
    }
    ```

1.  在`jni/Store.h`文件中，向本地`StoreType`枚举和`StoreValue`联合体中添加整数类型：

    ```java
    ...
    typedef enum {
     StoreType_Integer,
        StoreType_String
    } StoreType;
    typedef union {
     int32_t   mInteger;
        char*     mString;
    } StoreValue;
    ...
    ```

1.  使用`javah`刷新 JNI 头文件`jni/com_packtpub_Store.h`。应该出现两个新方法`Java_com_packtpub_store_Store_getInteger()`和`Java_com_packtpub_store_Store_setInteger()`。

1.  在`jni/com_packtpub_Store.cpp`文件中，借助生成的 JNI 头文件实现`getInteger()`方法。该方法仅返回条目的整数值，除了从`int32_t`隐式转换为`jint`外，不进行任何特定的转换。如果在检索过程中出现任何问题，将返回默认值：

    ```java
    ...
    JNIEXPORT jint JNICALL Java_com_packtpub_store_Store_getInteger
      (JNIEnv* pEnv, jobject pThis, jstring pKey) {
        StoreEntry* entry = findEntry(pEnv, &gStore, pKey);
        if (isEntryValid(pEnv, entry, StoreType_Integer)) {
            return entry->mValue.mInteger;
        } else {
            return 0;
        }
    }
    ...
    ```

1.  第二个方法`setInteger()`将给定的整数值存储在分配的条目中。注意，传递的 JNI 整数同样可以反向转换为 C/C++整数：

    ```java
    ...
    JNIEXPORT void JNICALL Java_com_packtpub_store_Store_setInteger
      (JNIEnv* pEnv, jobject pThis, jstring pKey, jint pInteger) {
        StoreEntry* entry = allocateEntry(pEnv, &gStore, pKey);
        if (entry != NULL) {
            entry->mType = StoreType_Integer;
            entry->mValue.mInteger = pInteger;
        }
    }
    ```

## *刚才发生了什么？*

运行应用程序。尝试使用不同的键、类型和值保存几个条目。然后尝试从本地存储中获取它们。这次我们已经实现了从 Java 到 C/C++ 传递和检索整数原始数据。

在本地调用期间，整数原始数据有多种形式；首先，Java 代码中的 `int`，然后是从/到 Java 代码传输期间的 `jint`，最后是本地代码中的 `int` 或 `int32_t`。显然，如果我们愿意，可以保留本地代码中的 JNI 表示形式 `jint`，因为所有这些类型实际上是等价的。换句话说，`jint` 只是一个别名。

### 提示

`int32_t` 类型是由 C99 标准库通过 `typedef` 引入的，旨在提高可移植性。与标准 `int` 类型的区别在于，它的字节大小对所有编译器和平台都是固定的。更多的数字类型在 `stdint.h`（在 C 中）或 `cstdint`（在 C++ 中）中定义。

所有原始类型在 JNI 中都有其适当的别名：

| Java 类型 | JNI 类型 | C 类型 | Stdint C 类型 |
| --- | --- | --- | --- |
| `boolean` | `Jboolean` | `unsigned char` | `uint8_t` |
| `byte` | `Jbyte` | `signed char` | `int8_t` |
| `char` | `Jchar` | `unsigned short` | `uint16_t` |
| `double` | `Jdouble` | `double` | `N/A` |
| `float` | `jfloat` | `float` | `N/A` |
| `int` | `jint` | `Int` | `int32_t` |
| `long` | `jlong` | `long long` | `int64_t` |
| `short` | `jshort` | `Short` | `int16_t` |

你可以完全像在这一部分中使用整数一样使用它们。关于 JNI 中原始类型更多信息可以在 [`docs.oracle.com/javase/6/docs/technotes/guides/jni/spec/types.html`](http://docs.oracle.com/javase/6/docs/technotes/guides/jni/spec/types.html) 找到

## 动手英雄——传递和返回其他原始类型

当前存储只处理整数和字符串。基于此模型，尝试为其他原始类型实现存储方法：`boolean`、`byte`、`char`、`double`、`float`、`long` 和 `short`。

### 注意

最终项目与此书一同提供，名称为 `Store_Part6_Full`。

# 从本地代码引用 Java 对象

如前一部分所述，我们知道在 JNI 中字符串由 `jstring` 表示，实际上它是一个 Java 对象，这意味着可以通过 JNI 交换任何 Java 对象！然而，由于本地代码不能直接理解或访问 Java，所有 Java 对象都有相同的表示形式，即 `jobject`。

在这一部分，我们将重点介绍如何在本地端保存对象以及如何将其发送回 Java。作为一个例子，我们将使用自定义对象 `Color`，尽管任何其他类型的对象也可以。

### 注意

最终项目与此书一同提供，名称为 `Store_Part7`。

# 动手时间——在本地存储中保存对象引用

1.  创建一个新的 Java 类 `com.packtpub.store.Color`，封装一个表示颜色的整数值。这个整数是通过 `android.graphics.Color` 类从包含 HTML 代码的 `String`（例如，`#FF0000`）解析得到的。

    ```java
    package com.packtpub.store;
    import android.text.TextUtils;
    public class Color {
        private int mColor;
        public Color(String pColor) {
            if (TextUtils.isEmpty(pColor)) {
                throw new IllegalArgumentException();
            }
            mColor = android.graphics.Color.parseColor(pColor);
        }
        @Override
        public String toString() {
            return String.format("#%06X", mColor);
        }
    }
    ```

1.  在 `StoreType.java` 中，将新的 Color 数据类型添加到枚举中：

    ```java
    public enum StoreType {
        Integer,
        String,
     Color
    }
    ```

1.  在 `Store` 类中，添加两个新的本地方法以获取和保存 `Color` 对象：

    ```java
    public class Store {
        ...
     public native Color getColor(String pKey);
     public native void setColor(String pKey, Color pColor);
    }
    ```

1.  打开 `StoreActivity.java` 文件，并更新方法 `onGetValue()` 和 `onSetValue()` 以解析和显示 `Color` 实例：

    ```java
    public class StoreActivity extends Activity {
        ...
        public static class PlaceholderFragment extends Fragment {
            ...
            private void onGetValue() {
                ...
                switch (type) {
                ...
     case Color:
     mUIValueEdit.setText(mStore.getColor(key)
                                    .toString());
     break;
                }
            }
            private void onSetValue() {
                ...
                try {
                    switch (type) {
                    ...
     case Color:
     mStore.setColor(key, new Color(value));
     break;
                    }
                } catch (Exception eException) {
                    displayMessage("Incorrect value.");
                }
                updateTitle();
            }
            ...
        }
    }
    ```

1.  在 `jni/Store.h` 中，将新的颜色类型添加到 `StoreType` 枚举中，并在 `StoreValue` 联合体中添加一个新成员。但是你应该使用什么类型呢？`Color` 是只在 Java 中已知的对象。在 JNI 中，所有 Java 对象都有相同的类型；`jobject`，一个（间接）对象引用：

    ```java
    ...
    typedef enum {
        ...
        StoreType_String,
     StoreType_Color
    } StoreType;
    typedef union {
        ...
        char*     mString;
     jobject   mColor;
    } StoreValue;
    ...
    ```

1.  使用 `javah` 重新生成 JNI 头文件 `jni/com_packtpub_Store.h`。你应在其中看到两个新的方法 `Java_com_packtpub_store_Store_getColor()` 和 `Java_com_packtpub_store_Store_setColor()`。

1.  打开 `jni/com_packtpub_Store.cpp` 并实现两个新生成的 `getColor()` 和 `setColor()` 方法。第一个方法只是简单地返回存储条目中保留的 Java Color 对象，如下代码所示：

    ```java
    ...
    JNIEXPORT jobject JNICALL Java_com_packtpub_store_Store_getColor
      (JNIEnv* pEnv, jobject pThis, jstring pKey) {
        StoreEntry* entry = findEntry(pEnv, &gStore, pKey);
        if (isEntryValid(pEnv, entry, StoreType_Color)) {
            return entry->mValue.mColor;
        } else {
            return NULL;
        }
    }
    ...
    ```

    第二个方法 `setColor()` 中引入了真正的细微差别。实际上，乍一看，简单地将 `jobject` 值保存在存储条目中似乎就足够了。然而，这种假设是错误的。在参数中传递或在 JNI 方法内创建的对象是局部引用。局部引用不能在本地方法范围之外（如对于字符串）的本地代码中保存。

1.  为了允许在本地方法返回后在本地代码中保留 Java 对象引用，它们必须被转换为全局引用，以通知 Dalvik VM 它们不能被垃圾收集。为此，JNI API 提供了 `NewGlobalRef()` 方法：

    ```java
    ...
    JNIEXPORT void JNICALL Java_com_packtpub_store_Store_setColor
      (JNIEnv* pEnv, jobject pThis, jstring pKey, jobject pColor) {
        // Save the Color reference in the store.
        StoreEntry* entry = allocateEntry(pEnv, &gStore, pKey);
        if (entry != NULL) {
            entry->mType = StoreType_Color;
            // The Java Color is going to be stored on the native side.
            // Need to keep a global reference to avoid a potential
            // garbage collection after method returns.
            entry->mValue.mColor = pEnv->NewGlobalRef(pColor);
        }
    }
    ```

1.  在 `Store.cpp` 中，修改 `releaseEntryValue()` 方法，当条目被新条目替换时删除全局引用。这是通过 `DeleteGlobalRef()` 方法完成的，它是 `NewGlobalRef()` 的对应方法：

    ```java
    ...
    void releaseEntryValue(JNIEnv* pEnv, StoreEntry* pEntry) {
        switch (pEntry->mType) {
        case StoreType_String:
            delete pEntry->mValue.mString;
            break;
     case StoreType_Color:
     // Unreferences the object for garbage collection.
     pEnv->DeleteGlobalRef(pEntry->mValue.mColor);
     break;
        }
    }
    ```

## *刚才发生了什么？*

运行应用程序。输入并保存一个颜色值，如 **#FF0000** 或 **red**，这是 Android 颜色解析器允许的预定义值。从存储中获取条目。我们设法在本地端引用了一个 Java 对象！Java 对象不是也不能转换为 C++ 对象。它们本质上是不同的。因此，要在本地端保留 Java 对象，我们必须使用 JNI API 保留对它们的引用。

来自 Java 的所有对象都由 `jobject` 表示，甚至 `jstring`（实际上内部是 `jobject` 的 `typedef`）。`jobject` 只是一个没有智能垃圾收集机制的“指针”（毕竟，我们至少部分想要摆脱 Java）。它不直接给你 Java 对象内存的引用，而是间接引用。实际上，与 C++ 对象相反，Java 对象在内存中没有固定的位置。它们在其生命周期内可能会被移动。无论如何，在内存中处理 Java 对象表示都是一个坏主意。

## 局部引用

本地调用的作用域限制在方法内，这意味着一旦本地方法结束，虚拟机将再次接管。JNI 规范利用这一事实，将对象引用限制在方法边界内。这意味着 `jobject` 只能在它被赋予的方法内安全使用。一旦本地方法返回，Dalvik VM 无法知道本地代码是否还持有对象引用，并且可以在任何时间决定收集它们。

这种类型的引用称为**本地**引用。当本地方法返回时，它们会自动释放（指的是引用，而不是对象，尽管垃圾收集器可能会这样做），以允许在后面的 Java 代码中进行适当的垃圾收集。例如，以下代码段应该是严格禁止的。在 JNI 方法外部保留这样的引用最终会导致未定义的行为（内存损坏、崩溃等）：

```java
static jobject gMyReference;
JNIEXPORT void JNICALL Java_MyClass_myMethod(JNIEnv* pEnv,
                                     jobject pThis, jobject pRef) {
    gMyReference = pRef;
    ...
}

// Later on...
env->CallVoidMethod(gMyReference, ...);
```

### 提示

对象作为本地引用传递给本地方法。由 JNI 函数返回的每个 `jobject`（除了 `NewGlobalRef()`）都是一个本地引用。请记住，默认情况下一切都是本地引用。

JNI 提供了几种用于管理本地引用的方法：

1.  `NewLocalRef()` 可以显式地创建一个本地引用（例如，从一个全局引用），尽管这在实践中很少需要：

    ```java
    jobject NewLocalRef(jobject ref)
    ```

1.  `DeleteLocalRef()` 方法可以在不再需要时用来删除一个本地引用：

    ```java
    void DeleteLocalRef(jobject localRef)
    ```

### 提示

本地引用不能在方法作用域之外使用，也不能在即使是单个本地调用期间在各个线程间共享！

你不需要显式删除本地引用。然而，根据 JNI 规范，JVM 只需要同时存储 16 个本地引用，并且可能会拒绝创建更多（这是特定于实现的）。因此，尽早释放未使用的本地引用是良好的实践，特别是在处理数组时。

幸运的是，JNI 提供了一些其他方法来帮助处理本地引用。

1.  `EnsureLocalCapacity()` 告诉 VM 它需要更多的本地引用。当此方法无法保证请求的容量时，它返回 `-1` 并抛出 Java `OutOfMemoryError`：

    ```java
    jint EnsureLocalCapacity(jint capacity)
    ```

1.  `PushLocalFrame()` 和 `PopLocalFrame()` 提供了第二种分配更多本地引用的方法。这可以理解为批量分配本地槽和删除本地引用的方式。当此方法无法保证请求的容量时，它也会返回 `-1` 并抛出 Java `OutOfMemoryError`：

    ```java
    jint PushLocalFrame(jint capacity)
    jobject PopLocalFrame(jobject result)
    ```

    ### 提示

    直到 Android 4.0 冰激凌三明治版本，本地引用实际上是直接指针，这意味着它们可以保持在其自然作用域之外并且仍然有效。现在不再是这样，这种有缺陷的代码应该避免。

## 全局引用

要能在方法作用域之外使用对象引用或长时间保存它，引用必须被设置为**全局**。全局引用还允许在各个线程间共享对象，而本地引用则不能。

JNI 提供了两个为此目的的方法：

1.  使用`NewGlobalRef()`创建全局引用，防止回收指向的对象，并允许其在线程间共享。同一个对象的两个引用可能是不同的：

    ```java
    jobject NewGlobalRef(jobject obj)
    ```

1.  使用`DeleteGlobalRef()`删除不再需要全局引用。如果没有它，Dalvik VM 会认为对象仍然被引用，永远不会回收它们：

    ```java
    void DeleteGlobalRef(jobject globalRef)
    ```

1.  使用`IsSameObject()`比较两个对象引用，而不是使用`==`，后者不是比较引用的正确方式：

    ```java
    jboolean IsSameObject(jobject ref1, jobject ref2)
    ```

### 提示

切记要配对使用`New<Reference Type>Ref()`和`Delete<Reference Type>Ref()`。否则会导致内存泄漏。

## 弱引用

弱引用是 JNI 中可用的最后一种引用类型。它们与全局引用相似，可以在 JNI 调用之间保持并在线程间共享。然而，与全局引用不同，它们不会阻止垃圾回收。因此，这种引用必须谨慎使用，因为它可能随时变得无效，除非每次在使用之前从它们创建全局或局部引用（并在使用后立即释放！）。

### 提示

当适当使用时，弱引用有助于防止内存泄漏。如果你已经进行了一些 Android 开发，你可能已经知道最常见的泄漏之一：从后台线程（通常是`AsyncTask`）保持对 Activity 的“硬”引用，以便在处理完成后通知 Activity。的确，在发送通知之前，Activity 可能会被销毁（例如，因为用户旋转了屏幕）。当使用弱引用时，Activity 仍然可以被垃圾回收，从而释放内存。

`NewWeakGlobalRef()`和`DeleteWeakGlobalRef()`是创建和删除弱引用所需仅有的方法：

```java
jweak NewWeakGlobalRef(JNIEnv *env, jobject obj);
void DeleteWeakGlobalRef(JNIEnv *env, jweak obj);
```

这些方法返回一个`jweak`引用，如果需要，可以将其强制转换为输入对象（例如，如果你创建了一个到`jclass`的引用，那么返回的`jweak`可以强制转换为`jclass`或`jobject`）。

然而，你不应直接使用它，而应将其传递给`NewGlobalRef()`或`NewLocalRef()`，并像平常一样使用它们的结果。要确保从弱引用发出的局部或全局引用有效，只需检查`NewGlobalRef()`或`NewLocalRef()`返回的引用是否为`NULL`。完成对象操作后，你可以删除全局或局部引用。每次重新使用该弱对象时，请重新开始这个过程。例如：

```java
jobject myObject = ...;
// Keep a reference to that object until it is garbage collected.
jweak weakRef = pEnv->NewWeakGlobalRef(myObject);
...

// Later on, get a real reference, hoping it is still available.
jobject localRef = pEnv->NewLocalRef(weakRef);
if (!localRef) {
// Do some stuff...
pEnv->DeleteLocalRef(localRef);
} else {
   // Object has been garbage collected, reference is unusable...
}

...
// Later on, when weak reference is no more needed.
pEnv->DeleteWeakGlobalRef(weakRef);
```

要检查弱引用本身是否指向一个对象，请使用`IsSameObject()`将`jweak`与`NULL`进行比较（不要使用`==`）：

```java
jboolean IsSameObject(jobject ref1, jobject ref2)
```

在创建全局或局部引用之前，不要试图检查弱引用的状态，因为指向的对象可能会被并发地回收。

### 提示

在 Android 2.2 Froyo 之前，弱引用根本不存在。直到 Android 4.0 Ice Cream Sandwich，除了`NewGlobalRef()`或`NewLocalRef()`之外，它们不能在 JNI 调用中使用。尽管这不再是强制性的，但在其他 JNI 调用中直接使用弱引用应被视为一种不良实践。

若要了解更多关于此主题的信息，请查看 JNI 规范，链接为：[`docs.oracle.com/javase/6/docs/technotes/guides/jni/spec/jniTOC.html`](http://docs.oracle.com/javase/6/docs/technotes/guides/jni/spec/jniTOC.html)。

# 管理 Java 数组

还有一种我们尚未讨论的数据类型：**数组**。数组在 Java 和 JNI 中都有其特定的位置。它们具有自己的类型和 API，尽管 Java 数组在本质上也是对象。

在这一部分，我们将通过允许用户在输入项中同时输入一组值来改进我们的存储。这组值将作为 Java 数组传递给本地存储，然后以传统的 C 数组形式存储。

### 注意事项

最终的项目作为本书的一部分提供，名为`Store_Part8`。

# 动手实践——在本地存储中处理 Java 数组

为了帮助我们处理数组操作，让我们下载一个辅助库，**Google Guava**（在撰写本书时为 18.0 版本），可在[`code.google.com/p/guava-libraries/`](http://code.google.com/p/guava-libraries/)获取。Guava 提供了许多用于处理原语和数组，以及执行“伪函数式”编程的有用方法。

将`guava jar`复制到项目`libs`目录中。打开**属性**项目，并转到**Java 构建路径** | **库**。通过点击**添加 JARs...**按钮并验证，引用 Guava jar。

1.  编辑`StoreType.java`枚举，并添加三个新值：`IntegerArray`、`StringArray`和`ColorArray`：

    ```java
    public enum StoreType {
        ...
        Color,
        IntegerArray,
        StringArray,
        ColorArray
    }
    ```

1.  打开`Store.java`文件，并添加新的方法以获取和保存`int`、`String`和`Color`数组：

    ```java
    public class Store {
        ...
     public native int[] getIntegerArray(String pKey);
     public native void setIntegerArray(String pKey, int[] pIntArray);
     public native String[] getStringArray(String pKey);
     public native void setStringArray(String pKey,
     String[] pStringArray);
     public native Color[] getColorArray(String pKey);
     public native void setColorArray(String pKey,Color[] pColorArray);
    }
    ```

1.  编辑`StoreActivity.java`，将本地方法连接到 GUI。

    修改`onGetValue()`方法，使其根据其类型从存储中检索数组，使用分号分隔符（得益于 Guava 连接器）连接其值，并最终显示它们：

    ```java
    public class StoreActivity extends Activity {
        ...
        public static class PlaceholderFragment extends Fragment {
            ...
            private void onGetValue() {
                ...
                switch (type) {
                ...
     case IntegerArray:
     mUIValueEdit.setText(Ints.join(";", mStore
     .getIntegerArray(key)));
     break;
     case StringArray:
     mUIValueEdit.setText(Joiner.on(";").join(
     mStore.getStringArray(key)));
     break;
     case ColorArray:
     mUIValueEdit.setText(Joiner.on(";").join(mStore
     .getColorArray(key)));
     break;            case IntegerArray:
                }
            }
            ...
    ```

1.  改进`onSetValue()`方法，在将值列表传输到`Store`之前将其转换成数组（得益于 Guava 的转换特性）：

    ```java
            ...
            private void onSetValue() {
                ...
                try {
                    switch (type) {
                    ...
                    case IntegerArray:
     mStore.setIntegerArray(key, Ints.toArray(
     stringToList(new Function<String, Integer>() {
     public Integer apply(String pSubValue) {
     return Integer.parseInt(pSubValue);
     }
     }, value)));
     break;
     case StringArray:
     String[] stringArray = value.split(";");
     mStore.setStringArray(key, stringArray);
     break;
     case ColorArray:
     List<Color> idList = stringToList(
     new Function<String, Color>() {
     public Color apply(String pSubValue) {
     return new Color(pSubValue);
     }
     }, value);
     mStore.setColorArray(key, idList.toArray(
     new Color[idList.size()]));
     break;
                    }
                } catch (Exception eException) {
                    displayMessage("Incorrect value.");
                }
                updateTitle();
            }
            ...
    ```

1.  编写一个辅助方法`stringToList()`，帮助您将字符串转换为目标类型的列表：

    ```java
            ...
            private <TType> List<TType> stringToList(
                            Function<String, TType> pConversion,
                            String pValue) {
                String[] splitArray = pValue.split(";");
                List<String> splitList = Arrays.asList(splitArray);
                return Lists.transform(splitList, pConversion);
            }
        }
    }
    ```

1.  在`jni/Store.h`中，将新的数组类型添加到`StoreType`枚举中。同时，在`StoreValue`联合体中声明新字段`mIntegerArray`、`mStringArray`和`mColorArray`。存储数组以原始 C 数组（即一个指针）的形式表示：

    ```java
    ...
    typedef enum {
        ...
        StoreType_Color,
     StoreType_IntegerArray,
     StoreType_StringArray,
     StoreType_ColorArray
    } StoreType;

    typedef union {
        ...
        jobject   mColor;
     int32_t*  mIntegerArray;
     char**    mStringArray;
     jobject*  mColorArray;
    } StoreValue;
    ...
    ```

1.  我们还需要记住这些数组的长度。在`StoreEntry`中的新字段`mLength`中输入此信息：

    ```java
    ...
    typedef struct {
        char* mKey;
        StoreType mType;
        StoreValue mValue;
     int32_t mLength;
    } StoreEntry;
    ...
    ```

1.  在`jni/Store.cpp`中，为新的数组类型在`releaseEntryValue()`中插入案例。实际上，当相应的条目被释放时，必须释放分配的数组。由于颜色是 Java 对象，删除每个数组项中保存的全局引用，否则永远不会进行垃圾回收（导致内存泄漏）：

    ```java
    void releaseEntryValue(JNIEnv* pEnv, StoreEntry* pEntry) {
        switch (pEntry->mType) {
        ...
     case StoreType_IntegerArray:
     delete[] pEntry->mValue.mIntegerArray;
     break;
     case StoreType_StringArray:
     // Destroys every C string pointed by the String array
     // before releasing it.
     for (int32_t i = 0; i < pEntry->mLength; ++i) {
     delete pEntry->mValue.mStringArray[i];
     }
     delete[] pEntry->mValue.mStringArray;
     break;
     case StoreType_ColorArray:
     // Unreferences every Id before releasing the Id array.
     for (int32_t i = 0; i < pEntry->mLength; ++i) {
     pEnv->DeleteGlobalRef(pEntry->mValue.mColorArray[i]);
     }
     delete[] pEntry->mValue.mColorArray;
     break;
        }
    }
    ...
    ```

1.  使用`Javah`重新生成 JNI 头文件`jni/com_packtpub_Store.h`。在`jni/com_packtpub_Store.cpp`中实现所有这些新方法。为此，首先添加`csdtint`包含。

    ```java
    #include "com_packtpub_store_Store.h"
    #include <cstdint>
    #include <cstdlib>
    #include "Store.h"
    ...
    ```

1.  然后，缓存`String`和`Color`的 JNI 类，以便在后续步骤中能够创建这些类型的对象数组。类可以通过`JNIEnv`自身的反射访问，并且可以从传递给`JNI_OnLoad()`的`JavaVM`中获取。

    我们需要检查找到的类是否为 null，以防它们无法加载。如果发生这种情况，虚拟机会引发异常，以便我们可以立即返回：

    ```java
    ...
    static jclass StringClass;
    static jclass ColorClass;

    JNIEXPORT jint JNI_OnLoad(JavaVM* pVM, void* reserved) {
     JNIEnv *env;
     if (pVM->GetEnv((void**) &env, JNI_VERSION_1_6) != JNI_OK) {
     abort();
     }
     // If returned class is null, an exception is raised by the VM.
     jclass StringClassTmp = env->FindClass("java/lang/String");
     if (StringClassTmp == NULL) abort();
     StringClass = (jclass) env->NewGlobalRef(StringClassTmp);
     env->DeleteLocalRef(StringClassTmp);
     jclass ColorClassTmp = env->FindClass("com/packtpub/store/Color");
     if (ColorClassTmp == NULL) abort();
     ColorClass = (jclass) env->NewGlobalRef(ColorClassTmp);
     env->DeleteLocalRef(ColorClassTmp);
        // Store initialization.
        gStore.mLength = 0;
        return JNI_VERSION_1_6;
    }
    ...
    ```

1.  编写`getIntegerArray()`的实现。JNI 整数数组用`jintArray`类型表示。如果`int`等同于`jint`，那么`int*`数组绝对不等同于`jintArray`。第一个是指向内存缓冲区的指针，而第二个是对对象的引用。

    因此，为了在这里返回`jintArray`，使用 JNI API 方法`NewIntArray()`实例化一个新的 Java 整数数组。然后，使用`SetIntArrayRegion()`将本地`int`缓冲区内容复制到`jintArray`中：

    ```java
    ...
    JNIEXPORT jintArray JNICALL
    Java_com_packtpub_store_Store_getIntegerArray
      (JNIEnv* pEnv, jobject pThis, jstring pKey) {
        StoreEntry* entry = findEntry(pEnv, &gStore, pKey);
        if (isEntryValid(pEnv, entry, StoreType_IntegerArray)) {
            jintArray javaArray = pEnv->NewIntArray(entry->mLength);
            pEnv->SetIntArrayRegion(javaArray, 0, entry->mLength,
                                    entry->mValue.mIntegerArray);
            return javaArray;
        } else {
            return NULL;
        }
    }
    ...
    ```

1.  为了在本地代码中保存 Java 数组，存在逆操作`GetIntArrayRegion()`。分配合适内存缓冲的唯一方式是使用`GetArrayLength()`测量数组大小：

    ```java
    ...
    JNIEXPORT void JNICALL Java_com_packtpub_store_Store_setIntegerArray
      (JNIEnv* pEnv, jobject pThis, jstring pKey,
       jintArray pIntegerArray) {
        StoreEntry* entry = allocateEntry(pEnv, &gStore, pKey);
        if (entry != NULL) {
            jsize length = pEnv->GetArrayLength(pIntegerArray);
            int32_t* array = new int32_t[length];
            pEnv->GetIntArrayRegion(pIntegerArray, 0, length, array);

            entry->mType = StoreType_IntegerArray;
            entry->mLength = length;
            entry->mValue.mIntegerArray = array;
        }
    }
    ...
    ```

Java 对象数组与 Java 基本数组不同。它们是用类类型（这里，缓存的`String jclass`）实例化的，因为 Java 数组是单类型的。对象数组本身用`jobjectArray`类型表示，可以通过 JNI API 方法`NewObjectArray()`创建。

与基本数组不同，不可能同时处理所有元素。相反，使用`SetObjectArrayElement()`逐个设置对象。这里，本地数组被填充了在本地存储的`String`对象，这些对象保持全局引用。因此，除了对新分配字符串的引用外，这里无需删除或创建任何引用。

```java
...
JNIEXPORT jobjectArray JNICALL
Java_com_packtpub_store_Store_getStringArray
  (JNIEnv* pEnv, jobject pThis, jstring pKey) {
    StoreEntry* entry = findEntry(pEnv, &gStore, pKey);
    if (isEntryValid(pEnv, entry, StoreType_StringArray)) {
        // An array of String in Java is in fact an array of object.
        jobjectArray javaArray = pEnv->NewObjectArray(entry->mLength,
                StringClass, NULL);
        // Creates a new Java String object for each C string stored.
        // Reference to the String can be removed right after it is
        // added to the Java array, as the latter holds a reference
        // to the String object.
        for (int32_t i = 0; i < entry->mLength; ++i) {
            jstring string = pEnv->NewStringUTF(
                    entry->mValue.mStringArray[i]);
            // Puts the new string in the array
            pEnv->SetObjectArrayElement(javaArray, i, string);
            // Do it here to avoid holding many useless local refs.
            pEnv->DeleteLocalRef(string);
        }
        return javaArray;
    } else {
        return NULL;
    }
}
...
```

在`setStringArray()`方法中，通过`GetObjectArrayElement()`逐个获取数组元素。返回的引用是局部的，应当将其变为全局引用，以便在本地安全地存储它们。

```java
...
JNIEXPORT void JNICALL Java_com_packtpub_store_Store_setStringArray
  (JNIEnv* pEnv, jobject pThis, jstring pKey,
   jobjectArray pStringArray) {
    // Creates a new entry with the new String array.
    StoreEntry* entry = allocateEntry(pEnv, &gStore, pKey);
    if (entry != NULL) {
        // Allocates an array of C string.
        jsize length = pEnv->GetArrayLength(pStringArray);
        char** array = new char*[length];
        // Fills the C array with a copy of each input Java string.
        for (int32_t i = 0; i < length; ++i) {
            // Gets the current Java String from the input Java array.
            // Object arrays can be accessed element by element only.
            jstring string = (jstring)
                         pEnv->GetObjectArrayElement(pStringArray, i);
            jsize stringLength = pEnv->GetStringUTFLength(string);
            array[i] = new char[stringLength + 1];
            // Directly copies the Java String into our new C buffer.
            pEnv->GetStringUTFRegion(string,0,stringLength, array[i]);
            // Append the null character for string termination.
            array[i][stringLength] = '\0';
            // No need to keep a reference to the Java string anymore.
            pEnv->DeleteLocalRef(string);
        }
        entry->mType = StoreType_StringArray;
        entry->mLength = length;
        entry->mValue.mStringArray = array;
    }
}
```

以`getColorArray()`开始，对颜色执行相同的操作。由于字符串和颜色在 Java 端都是对象，所以可以使用`NewObjectArray()`以相同的方式创建返回的数组。

使用 JNI 方法`SetObjectArrayElement()`将每个保存的`Color`引用放置在数组内。由于颜色在本地作为全局 Java 引用存储，无需创建或删除局部引用：

```java
...
JNIEXPORT jobjectArray JNICALL
Java_com_packtpub_store_Store_getColorArray
  (JNIEnv* pEnv, jobject pThis, jstring pKey) {
    StoreEntry* entry = findEntry(pEnv, &gStore, pKey);
    if (isEntryValid(pEnv, entry, StoreType_ColorArray)) {
        // Creates a new array with objects of type Id.
        jobjectArray javaArray = pEnv->NewObjectArray(entry->mLength,
                ColorClass, NULL);
        // Fills the array with the Color objects stored on the native
        // side, which keeps a global reference to them. So no need
        // to delete or create any reference here.
        for (int32_t i = 0; i < entry->mLength; ++i) {
            pEnv->SetObjectArrayElement(javaArray, i,
                                        entry->mValue.mColorArray[i]);
        }
        return javaArray;
    } else {
        return NULL;
    }
}
...
```

在`setColorArray()`中，颜色元素也是通过`GetObjectArrayElement()`逐个检索的。同样，返回的引用是局部的，应该使其全局化以在本地安全存储：

```java
...
JNIEXPORT void JNICALL Java_com_packtpub_store_Store_setColorArray
  (JNIEnv* pEnv, jobject pThis, jstring pKey,
   jobjectArray pColorArray) {
    // Saves the Color array in the store.
    StoreEntry* entry = allocateEntry(pEnv, &gStore, pKey);
    if (entry != NULL) {
        // Allocates a C array of Color objects.
        jsize length = pEnv->GetArrayLength(pColorArray);
        jobject* array = new jobject[length];
        // Fills the C array with a copy of each input Java Color.
        for (int32_t i = 0; i < length; ++i) {
            // Gets the current Color object from the input Java array.
            // Object arrays can be accessed element by element only.
            jobject localColor = pEnv->GetObjectArrayElement(
                    pColorArray, i);
            // The Java Color is going to be stored on the native side
            // Need to keep a global reference to avoid a potential
            // garbage collection after method returns.
            array[i] = pEnv->NewGlobalRef(localColor);
            // We have a global reference to the Color, so we can now
            // get rid of the local one.
            pEnv->DeleteLocalRef(localColor);
        }
        entry->mType = StoreType_ColorArray;
        entry->mLength = length;
        entry->mValue.mColorArray = array;
    }
}
```

## *刚才发生了什么？*

我们从 Java 传输数组到本地侧，反之亦然。Java 数组是只能通过专用的 JNI API 操作的 Java 对象。它们不能被转换为原生的 C/C++数组，也不能以同样的方式使用。

我们还了解了如何利用`JNI_OnLoad()`回调来缓存 JNI 类描述符。类描述符，类型为`jclass`（在幕后也是`jobject`），相当于 Java 中的`Class<?>`。它们允许我们定义我们想要的数组类型，有点像 Java 中的反射 API。我们将在下一章回到这个主题。

## 原始数组

可用的原始数组类型有`jbooleanArray`，`jbyteArray`，`jcharArray`，`jdoubleArray`，`jfloatArray`，`jlongArray`和`jshortArray`。这些类型表示对真实 Java 数组的引用。

这些数组可以使用 JNI 提供的多种方法进行操作：

1.  使用`New<Primitive>Array()`创建新的 Java 数组：

    ```java
    jintArray NewIntArray(jsize length)
    ```

1.  `GetArrayLength()`检索数组的长度：

    ```java
    jsize GetArrayLength(jarray array)
    ```

1.  `Get<Primitive>ArrayElements()`将整个数组检索到由 JNI 分配的内存缓冲区中。最后一个参数`isCopy`，如果不为空，表示 JNI 是否内部复制了数组，或者返回的缓冲区指针指向实际的 Java 字符串内存：

    ```java
    jint* GetIntArrayElements(jintArray array, jboolean* isCopy)
    ```

1.  `Release<Primitive>ArrayElements()`释放由`Get<Primitive>ArrayElements()`分配的内存缓冲区。总是成对使用。最后一个参数模式与`isCopy`参数相关，表示以下内容：

    +   如果设置为 0，那么 JNI 应该将修改后的数组复制回初始的 Java 数组，并告诉 JNI 释放其临时内存缓冲区。这是最常见的标志。

    +   如果设置`JNI_COMMIT`，那么 JNI 应该将修改后的数组复制回初始数组，但不释放内存。这样，客户端代码在将结果传回 Java 的同时，仍可以在内存缓冲区中继续处理。

    +   如果设置`JNI_ABORT`，那么 JNI 必须丢弃内存缓冲区中进行的任何更改，并保持 Java 数组不变。如果临时本地内存缓冲区不是副本，这将无法正确工作。

        ```java
        void ReleaseIntArrayElements(jintArray array, jint* elems, jint mode)
        ```

1.  `Get<Primitive>ArrayRegion()`将数组的全部或部分内容检索到由客户端代码分配的内存缓冲区中。例如，对于整数：

    ```java
    void GetIntArrayRegion(jintArray array, jsize start, jsize len,
                           jint* buf)
    ```

1.  `Set<Primitive>ArrayRegion()`从由客户端代码管理的本地缓冲区初始化 Java 数组的全部或部分内容。例如，对于整数：

    ```java
    void SetIntArrayRegion(jintArray array, jsize start, jsize len,
                           const jint* buf)
    ```

1.  `Get<Primitive>ArrayCritical()`和`Release<Primitive>ArrayCritical()`与`Get<Primitive>ArrayElements()`和`Release<Primitive>ArrayElements()`相似，但仅供直接访问目标数组（而不是副本）使用。作为交换，调用者不得执行阻塞或 JNI 调用，并且不应长时间持有数组（如线程的关键部分）。同样，所有基本类型都提供这两个方法：

    ```java
    void* GetPrimitiveArrayCritical(jarray array, jboolean* isCopy)
    void ReleasePrimitiveArrayCritical(jarray array, void* carray, jint mode)
    ```

## 尝试英雄——处理其他数组类型

利用新获得的知识，你可以为其他数组类型实现存储方法：`jbooleanArray`、`jbyteArray`、`jcharArray`、`jdoubleArray`、`jfloatArray`、`jlongArray`和`jshortArray`。

例如，你可以使用`GetBooleanArrayElements()`和`ReleaseBooleanArrayElements()`而不是`GetBooleanArrayRegion()`，为`jbooleanArray`类型编写`setBooleanArray()`方法。结果应该如下所示，两种方法与`memcpy()`配对调用：

```java
...
JNIEXPORT void JNICALL Java_com_packtpub_store_Store_setBooleanArray
  (JNIEnv* pEnv, jobject pThis, jstring pKey,
   jbooleanArray pBooleanArray) {
    // Finds/creates an entry in the store and fills its content.
    StoreEntry* entry = allocateEntry(pEnv, &gStore, pKey);
    if (entry != NULL) {
        entry->mType = StoreType_BooleanArray;
        jsize length = pEnv->GetArrayLength(pBooleanArray);
        uint8_t* array = new uint8_t[length];
        // Retrieves array content.
 jboolean* arrayTmp = pEnv->GetBooleanArrayElements(
 pBooleanArray, NULL);
        memcpy(array, arrayTmp, length * sizeof(uint8_t));
        pEnv->ReleaseBooleanArrayElements(pBooleanArray, arrayTmp, 0);
        entry->mType = StoreType_BooleanArray;
        entry->mValue.mBooleanArray = array;
        entry->mLength = length;
    }
}
...
```

### 注意

最终的项目以`Store_Part8_Full`的名字随本书提供。

## 对象数组

在 JNI 中，对象数组被称为`jobjectArray`，代表对 Java 对象数组的引用。对象数组是特殊的，因为与基本数组不同，每个数组元素都是对对象的引用。因此，每次在数组中插入对象时，都会自动注册一个新的全局引用。这样，本地调用结束时，引用就不会被垃圾回收。注意，对象数组不能像基本类型那样转换为“本地”数组。

对象数组可以使用 JNI 提供的几种方法进行操作：

1.  `NewObjectArray()`创建一个新的对象数组实例：

    ```java
    jobjectArray NewObjectArray(jsize length, jclass elementClass, jobject initialElement);
    ```

1.  `GetArrayLength()`检索数组的长度（与基本类型相同的方法）：

    ```java
    jsize GetArrayLength(jarray array)
    ```

1.  `GetObjectArrayElement()`从 Java 数组中检索单个对象引用。返回的引用是局部的：

    ```java
    jobject GetObjectArrayElement(jobjectArray array, jsize index)
    ```

1.  `SetObjectArrayElement()`将单个对象引用放入 Java 数组中。隐式创建全局引用：

    ```java
    void SetObjectArrayElement(jobjectArray array, jsize index, jobject value)
    ```

有关 JNI 功能的更详尽列表，请参见[`docs.oracle.com/javase/6/docs/technotes/guides/jni/spec/functions.html`](http://docs.oracle.com/javase/6/docs/technotes/guides/jni/spec/functions.html)。

# 引发和检查 Java 异常

在 Store 项目中处理错误并不令人满意。如果找不到请求的键，或者检索到的值类型与请求的类型不匹配，将返回默认值。不要尝试使用 Color 条目。我们确实需要一种方法来指示发生了错误！还有什么比异常更好的错误指示方法呢？

JNI 提供了必要的 API，在 JVM 级别抛出异常。这些异常是你在 Java 中可以捕获的异常。它们与你在其他程序中可以找到的常规 C++异常（我们将在第九章，*将现有库移植到 Android*中进一步了解）在语法和流程上没有任何共同之处。

在这一部分，我们将了解如何从本地代码抛出 JNI 异常到 Java 端。

### 注意

本书提供的项目成果名为`Store_Part9`。

# 动手实践时间——在本地存储中抛出和捕获异常

1.  按照以下方式创建类型为`Exception`的 Java 异常`com.packtpub.exception.InvalidTypeException`：

    ```java
    package com.packtpub.exception;

    public class InvalidTypeException extends Exception {
        public InvalidTypeException(String pDetailMessage) {
            super(pDetailMessage);
        }
    }
    ```

    对另外两个异常重复该操作：类型为`Exception`的`NotExistingKeyException`和类型为`RuntimeException`的`StoreFullException`。

1.  打开`Store.java`文件，并在`Store`类中的`getInteger()`方法上声明抛出的异常（`StoreFullException`是`RuntimeException`，不需要声明）：

    ```java
    public class Store {
        ...
        public native int getInteger(String pKey)
     throws NotExistingKeyException, InvalidTypeException;
        public native void setInteger(String pKey, int pInt);
        ...
    ```

    对所有其他 getter 方法的原型（字符串、颜色等）重复该操作。

1.  这些异常需要被捕获。在`onGetValue()`中捕获`NotExistingKeyException`和`InvalidTypeException`：

    ```java
    public class StoreActivity extends Activity {
        ...
        public static class PlaceholderFragment extends Fragment {
            ...
            private void onGetValue() {
                ...
                try {
                    switch (type) {
                    ...
                }
     // Process any exception raised while retrieving data.
     catch (NotExistingKeyException eNotExistingKeyException) {
     displayMessage(eNotExistingKeyException.getMessage());
     } catch (InvalidTypeException eInvalidTypeException) {
     displayMessage(eInvalidTypeException.getMessage());
                }
            }
    ```

1.  在`onSetValue()`方法中捕获`StoreFullException`，以防因为存储空间耗尽导致条目无法插入：

    ```java
            private void onSetValue() {
                ...
                try {
                    ...
                } catch (NumberFormatException eNumberFormatException) {
                    displayMessage("Incorrect value.");
     } catch (StoreFullException eStoreFullException) {
     displayMessage(eStoreFullException.getMessage());
                } catch (Exception eException) {
                    displayMessage("Incorrect value.");
                }
                updateTitle();
            }
            ...
        }
    }
    ```

1.  打开之前部分创建的`jni/Store.h`文件，并定义三个新的辅助方法来抛出异常：

    ```java
    ...
    void throwInvalidTypeException(JNIEnv* pEnv);

    void throwNotExistingKeyException(JNIEnv* pEnv);

    void throwStoreFullException(JNIEnv* pEnv);
    #endif
    ```

1.  编辑`jni/Store.cpp`文件，当从存储中获取不适当的条目时抛出`NotExistingKeyException`和`InvalidTypeException`。在用`isEntryValid()`检查条目时抛出它们是一个好地方：

    ```java
    ...
    bool isEntryValid(JNIEnv* pEnv, StoreEntry* pEntry, StoreType pType) {
        if (pEntry == NULL) {
            throwNotExistingKeyException(pEnv);
        } else if (pEntry->mType != pType) {
            throwInvalidTypeException(pEnv);
        }
        return !pEnv->ExceptionCheck();
    }
    ...
    ```

1.  `StoreFullException`显然是在插入新条目时抛出的。修改同一文件中的`allocateEntry()`，以检查条目插入：

    ```java
    ...
    StoreEntry* allocateEntry(JNIEnv* pEnv, Store* pStore, jstring pKey) {
        // If entry already exists in the store, releases its content
        // and keep its key.
        StoreEntry* entry = findEntry(pEnv, pStore, pKey);
        if (entry != NULL) {
            releaseEntryValue(pEnv, entry);
        }
        // If entry does not exist, create a new entry
        // right after the entries already stored.
        else {
            // Checks store can accept a new entry.
     if (pStore->mLength >= STORE_MAX_CAPACITY) {
     throwStoreFullException(pEnv);
     return NULL;
            }
            entry = pStore->mEntries + pStore->mLength;
            // Copies the new key into its final C string buffer.
            ...
        }
        return entry;
    }
    ...
    ```

实现`throwNotExistingException()`。为了抛出一个 Java 异常，首先需要找到对应的类（就像使用 Java 反射 API 一样）。由于我们可以假设这些异常不会被频繁抛出，我们可以不缓存类引用。然后，使用`ThrowNew()`抛出异常。一旦我们不再需要异常类引用，可以使用`DeleteLocalRef`()来释放它。

```java
...
void throwNotExistingKeyException(JNIEnv* pEnv) {
    jclass clazz = pEnv->FindClass(
                    "com/packtpub/exception/NotExistingKeyException");
    if (clazz != NULL) {
        pEnv->ThrowNew(clazz, "Key does not exist.");
    }
    pEnv->DeleteLocalRef(clazz);
}
```

对另外两个异常重复该操作。代码是相同的（即使是抛出一个运行时异常），只有类名会改变。

## *刚才发生了什么？*

启动应用程序，尝试获取一个不存在的键的条目。重复该操作，但这次是存储中存在的条目，但其类型与 GUI 中选择的类型不同。在这两种情况下，都会出现错误信息。尝试在存储中保存超过 16 个引用，你将再次得到错误。在每种情况下，都在本地端抛出了异常，并在 Java 端捕获。

在本地代码中引发异常并不是一个复杂的任务，但也不是微不足道的。异常使用类型为`jclass`的类描述符实例化。JNI 需要这个类描述符来实例化适当的异常类型。JNI 异常与 JNI 方法原型中未声明，因为它们与 C++异常无关（C 中无法声明的异常）。这就解释了为什么我们没有重新生成 JNI 头文件以适应`Store.java`文件中的更改。

## 在异常状态下执行代码

一旦引发异常，你在使用 JNI 调用时需要非常小心。实际上，在此之后的任何后续调用都会失败，直到发生以下任一事件：

1.  方法返回，并传播一个异常。

1.  异常被清除。清除异常意味着该异常已被处理，因此不会传播到 Java。例如：

    ```java
    // Raise an exception
    jclass clazz = pEnv->FindClass("java/lang/RuntimeException");
    if (clazz != NULL) {
      pEnv->ThrowNew(clazz, "Oups an exception.");
    }
    pEnv->DeleteLocalRef(clazz);

    ...

    // Detect and catch the exception by clearing it.
    jthrowable exception = pEnv->ExceptionOccurred();
    if (exception) {
      // Do something...
      pEnv->ExceptionDescribe();
      pEnv->ExceptionClear();
      pEnv->DeleteLocalRef(exception);
    }
    ```

在引发异常后，仍然可以安全调用少数几个 JNI 方法：

| `DeleteGlobalRef` | `PopLocalFrame` |
| --- | --- |
| `DeleteLocalRef` | `PushLocalFrame` |
| `DeleteWeakGlobalRef` | `Release<Primitive>ArrayElements` |
| `ExceptionCheck` | `ReleasePrimitiveArrayCritical` |
| `ExceptionClear` | `ReleaseStringChars` |
| `ExceptionDescribe` | `ReleaseStringCritical` |
| `ExceptionOccurred` | `ReleaseStringUTFChars` |
| `MonitorExit` |   |

不要尝试调用其他 JNI 方法。本地代码应尽快清理其资源并将控制权交还给 Java（或者自行处理异常）。实际上，JNI 异常与 C++异常没有任何共同之处。它们的执行流程完全不同。当从本地代码引发 Java 异常时，后者可以继续其处理。但是，一旦本地调用返回并将控制权交还给 Java VM，后者就会像往常一样传播异常。换句话说，从本地代码引发的 JNI 异常只影响 Java 代码（以及之前未列出的其他 JNI 调用）。

## 异常处理 API

JNI 提供了几种用于管理异常的方法，其中包括：

1.  使用`ThrowNew()`来引发异常本身，分配一个新的实例：

    ```java
    jint ThrowNew(jclass clazz, const char* message)
    ```

1.  使用`Throw()`来引发已经分配的异常（例如，重新抛出）：

    ```java
    jint Throw(jthrowable obj)
    ```

1.  使用`ExceptionCheck()`来检查是否有待处理的异常，无论是由谁引发的（本地代码还是 Java 回调）。返回一个简单的`jboolean`，这使得它适合进行简单的检查：

    ```java
    jboolean ExceptionCheck()
    ```

1.  使用`ExceptionOccurred()`获取引发异常的`jthrowable`引用：

    ```java
    jthrowable ExceptionOccurred()
    ```

1.  `ExceptionDescribe()`相当于 Java 中的`printStackTrace()`：

    ```java
    void ExceptionDescribe()
    ```

1.  使用`ExceptionClear()`可以在本地端将异常标记为已捕获：

    ```java
    void ExceptionClear()
    ```

学会如何使用这些方法来编写健壮的代码至关重要，特别是在从本地代码回调 Java 时。我们将在下一章中更深入地学习这个主题。

# 总结

在本章中，我们了解了如何让 Java 与 C/C++进行通信。现在 Android 几乎可以说双语了！Java 可以使用任何类型的数据或对象调用 C/C++代码。

我们首先使用 `JNI_OnLoad` 钩子初始化了一个原生的 JNI 库。然后，在原生代码内部转换 Java 字符串，并了解了修改后的 UTF-8 与 UTF-16 字符编码之间的区别。我们还传递了 Java 基本类型到原生代码。这些基本类型每个都有它们可以转换为的 C/C++ 等效类型。

我们还在原生代码中使用全局引用处理了 Java 对象引用，并学习了全局引用与局部引用之间的区别。前者必须谨慎删除以确保适当的垃圾回收，而后者的作用域为原生方法，并且由于默认数量有限，也必须小心管理。

我们还讨论了如何在原生代码中管理 Java 数组，以便我们可以像操作原生数组一样访问它们的内容。在原生代码中操作数组时，虚拟机可能会也可能不会复制数组。这个性能开销必须考虑在内。

最后，我们在原生代码中抛出并检查了 Java 异常。我们了解到它们的标准 C++ 异常流程是不同的。当异常发生时，只有少数几个清理的 JNI 方法是安全的调用。JNI 异常是 JVM 级别的异常，这意味着它们的流程与标准 C++ 异常完全不同。

然而，还有更多内容等待我们去探索。任何 Java 对象、方法或字段都可以被原生代码调用或检索。让我们在下一章中看看如何从 C/C++ 代码中调用 Java。
