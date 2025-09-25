# 第五章. Unity 5 Pro 中的资产包

本章将包括 Unity 5 中资产包的概述。您将学习如何为 Android 设备实时下载新代码和数据。在本章结束时，读者将发现资产包在实践中的安全性技术。

本章将涵盖以下主题：

+   Unity 5 中资产包概述

+   为 Android 设备实时下载新代码和数据

+   实践中的安全性技术

# Unity 5 中资产包概述

资产包是**Unity Pro**特有的功能。资产包的两个主要思想是：

+   在您的应用程序中轻松下载内容

+   在您的应用程序中上传新内容

Unity 允许将您的资产导出为文件，这些文件被称为资产包。您的应用程序可以在需要时下载这些压缩文件。这种方法将通过流式传输以下内容来减少最终构建大小：预制体、动画、二进制文件、纹理、音频剪辑、网格和场景，其中将使用资产包。Unity 支持所有其他资产类型。对于二进制文件，您应将扩展名设置为`.bytes`，Unity 将识别这些文件为`TextAsset`。要使用资产包，您只需创建它们并将它们上传到您的服务器。在 Unity 编辑器中，您可以从场景中的资产构建资产包。在需要将您的资产包上传到服务器的情况下，您可以使用任何数据通信协议；例如，SSH、FTP、FTPS、SFTP 或任何其他协议，具体取决于您的选择。在实时情况下，您在脚本中编写的应用程序将下载必要的资产包，以便进一步处理这些导出文件中的打包资产。

我们将介绍您应该做什么来创建`AssetBundle`文件。为此任务，您应使用名为`BuildPipeline`的 Unity 编辑器类。

### 注意

如果您在脚本中使用任何 Unity 编辑器类，那么您应该始终记住将这些脚本保存在名为`Editor`的文件夹中，该文件夹位于项目中的任何子目录的`Assets`文件夹内。

现在，让我们创建一个简单的 C#脚本以创建`AssetBundle`。首先，我们应该导入两个 Unity 命名空间：

```java
  using UnityEngine;
  using UnityEditor;
```

在这些代码行之后，您应该声明您的公共类；例如，您可以创建一个非常简单的类声明：

```java
  public class BuilderAssetBundle {
    // the code you will see below
  }
```

在下一步中，我们将创建一个具有`MenuItem`的静态函数，以便将来从 Unity 编辑器菜单中选择：

```java
  [MenuItem("PacktPub/AssetBundles/Build Asset Bundle")]
  static void Build() {
    // the code you will see below
  }
```

然后，您需要用单个指令或一行代码或类似的内容填充您的静态函数，根据您的需求，您将看到已经完成的简单类来构建您的`AssetBundle`：

```java
  BuildPipeline.BuildAssetBundle(
    Selection.activeObject, 
   Selection.GetFiltered(typeof(Object),SelectionMode.DeepAssets),
    "Assets/Your/Path/To/YourAssetBundle.unity3d",
    BuildAssetBundleOptions.CollectDependencies | 
    BuildAssetBundleOptions.CompleteAssets
  );
```

此函数创建一个压缩的`Assets/Your/Path/To/YourAssetBundle.unity3d`文件，其中包含打包的资产列表（来自项目文件夹中的任何资产），如果`AssetBundle`创建成功则返回`true`，否则返回`false`。此函数的第一个变量`Selection.activeObject`指示要使用哪个对象来从`AssetBundle`中检索打包的资产，使用`AssetBundle.mainAsset`属性。如果你不使用它，可以将此值设置为`null`。第二个变量是`array Object[]`，指定需要打包的资产。第三个变量非常简单，它只是你想要保存`AssetBundle`文件的位置。通过具体调整`BuildAssetBundleOptions`标志，你可以指定自动包含所有依赖项或仅包含完整资产。此外，使用这些选项，你可以通过设置`UncompressedAssetBundle`标志来指定你不想在`AssetBundle`文件中压缩资产。如果你想，你可以在通过`WWW.LoadFromCacheOrDownload`调用下载导出的文件时检查你的资产包的`CRC`校验和。要创建资产包，你可以使用三个不同的函数：

+   `BuildPipeline.BuildAssetBundle`：这将构建任何类型的资产。

+   `BuildPipeline.BuildStreamedSceneAssetBundle`：这将仅包含场景。

+   `BuildPipeline.BuildAssetBundleExplicitAssetNames`：这个函数与`BuildPipeline.BuildAssetBundle`类似，但有一些区别。在这个函数中，你可以指定所有包含对象的字符串标识符。

### 注意

你可以为 Web-Player 平台创建资产包并在独立平台上使用它们，反之亦然。或者，对于移动平台，你可以仅使用它们的构建文件。例如，你可以为 Android 设备创建资产包并仅用于 Android 平台，但不能用于 iOS 平台，反之亦然。对于 iOS 平台的构建文件，你只能在其内部使用。

为了使用你的类来创建资产包，你需要进行两个简单的步骤：

1.  你应该在项目中选择一个或多个资产，这些资产将被打包到你的`AssetBundle`中。

1.  从 Unity 菜单中，你应该导航到**PacktPub** | **AssetBundles** | **Build Asset Bundle**。

创建`AssetBundle`的完成脚本应类似于以下代码所示：

```java
using UnityEngine;
using UnityEditor;

public class BuilderAssetBundle {
  [MenuItem("PacktPub/AssetBundles/Build Asset Bundle")]
  static void Build() {
    BuildPipeline.BuildAssetBundle(
      Selection.activeObject,
      Selection.GetFiltered(
        typeof(Object),
        SelectionMode.DeepAssets
      ),
      "Assets/Your/Path/To/YourAssetBundle.unity3d",
      BuildAssetBundleOptions.CollectDependencies | 
      BuildAssetBundleOptions.CompleteAssets
    );
  }
}
```

我们已经实现了资产包的创建，现在让我们探索如何在脚本中使用它们。要使用你的资产包，你应该遵循以下两个简单的步骤：

1.  从任何本地存储（如硬盘）或远程存储（如任何 Web 服务器）下载你的`AssetBundle`文件，Unity 提供了`WWW`辅助类来处理此类问题。

1.  加载，或者说，从你的`AssetBundle`文件中解包你的资产，以便在游戏中进一步使用。

在以下代码中，我们打算了解使用`AssetBundle`的情况。首先，我们需要创建一个简单的 C#脚本，例如命名为`ImporterAssetBundle`。在代码编辑器中打开脚本，并按以下所示进行更改：

```java
using UnityEngine;
using System.Collections;

public class ImporterAssetBundle : MonoBehaviour {
  void Start() {
    StartCoroutine(Import());
  }

  public IEnumerator Import() {
    using (WWW wwwData = WWW.LoadFromCacheOrDownload(
        "http://your-domain.com/your/path/url/new2.unity3d",
        23 // your asset bundle version, as an example only
    )) {
      yield return wwwData;
      GameObject obj = www.assetBundle.mainAsset as GameObject;
      Instantiate(obj);
      www.assetBundle.Unload(false);
    }
  }
}
```

将此脚本作为组件附加到你的 GameObject 上。此脚本中的 URL 仅作为示例，因此你应该设置你自己的`AssetBundle`文件 URL。基于这个版本，Unity 的缓存系统将决定是否下载你的文件。如果此文件已经以相同的版本号缓存，那么 Unity 将加快你的应用程序的速度。在这个脚本中，我们在`Start`事件中调用了`Import`协程，但你可以在需要的地方调用此函数，并且可以按需多次调用。在`Import`协程中，我们首先使用`WWW.LoadFromCacheOrDownload`，其唯一目的是通过提供 URL 路径和版本号来加载所需的`AssetBundle`文件。在下载过程中，Unity 将不会执行`yield return www`命令之后的指令。只有当下载完成后，Unity 才会运行我们协程示例中的下一个命令。

+   使用`www`实例从下载的`AssetBundle`中检索`mainAsset`作为 GameObject 实例

+   在当前场景中实时创建检索到的`GameObject`实例的新实例

+   卸载用于此**AssetBundle**的所有内存（以提高性能）。

由于制作项目是一个迭代过程，你可能会多次修改你的资源，这可能会迫使你在每次更改后重新制作资源包，以便能够测试它们。尽管在 Unity 编辑器中可以加载资源包，但这并不是最佳解决方案。相反，在 Unity 编辑器中进行测试时，你应该使用辅助函数`Resources.LoadAssetAtPath`来避免需要使用和更新资源包。该函数允许你以从资源包中加载的方式加载你的资源，但我们将跳过构建过程，你的资源始终是可用的。让我们创建一个新的 C#脚本，它将改进我们的最后一个示例，并添加一些异常处理以及`Resources.LoadAssetAtPath`在 Unity 编辑器导入时的功能。创建一个新的脚本，例如命名为`MyAssetBundleImporter`。下一步是在文件的开始处声明我们将使用的所需命名空间：

```java
  using UnityEngine;
  using System.Collections;
After these lines we will declare our public class:
  public class MyAssetBundleImporter {
    // the code you will see below
  }
```

让我们在类中声明一个`public`属性，用于从你的`AssetBundle`中检索的对象：

```java
  public Object assetBundleObject;
```

此外，在这个类中，让我们声明我们的`public`和特定的`AssetBundle`结构，以及其公共属性，以便进一步使用：

```java
  public struct AssetBundleStruct {
    public string assetSourceName;
    public string assetSourcePath;
    public string assetBundleUrl;
    public int assetBundleVersion;
  }
```

接下来，我们将声明此脚本的核心，即 Unity 将作为协程执行的功能：

```java
  public IEnumerator Import<T>(AssetBundleStruct abs) where T : Object {}
```

协程的第一步是将我们的`assetBundleObject`初始化为`null`：

```java
  assetBundleObject = null;
```

在那行代码之后，让我们声明主要条件，以决定如何导入你的资源包：

```java
  #if UNITY_EDITOR
    // 1st part, the code you will see below
  #else
    // 2nd part, the code you will see below
  #endif
```

让我们为`if`/`else`/`endif`预处理器语句的第一部分编写代码：

```java
  assetBundleObject = Resources.LoadAssetAtPath(
    abs.assetSourcePath, typeof(T)
  );

  if (null == assetBundleObject){
    Debug.LogError("AssetBundle ERROR Path: " + abs.assetSourcePath);
    Debug.LogError("Asset Bundle could not be found !!!");
  }
  yield break;
```

现在，我们将为`if`/`else`/`endif`预处理器语句的第二部分编写代码：

```java
  WWW www;
  if (Caching.enabled) {
    while (false == Caching.ready) {
      yield return null;
    }
    www = WWW.LoadFromCacheOrDownload(
      abs.assetBundleUrl, abs.assetBundleVersion
    );
  } else {
    www = new WWW(abs.assetBundleUrl);
  }

  yield return www;

  if (null != www.error) {
    Debug.LogError(www.error);
    www.Dispose();
    yield break;
  }

  AssetBundle ab = www.assetBundle;
  www.Dispose();
"  www = null;
  if (string.Empty == abs.assetSourceName || null == abs.assetSourceName) {
    assetBundleObject = ab.mainAsset;
  } else {
    assetBundleObject = ab.Load(abs.assetSourceName, typeof(T));
  }

  ab.Unload(false);
```

让我们创建一个简单的脚本，继承自`MonoBehaviour`，这将展示你如何使用你的示例类`MyAssetBundleImporter`：

```java
using UnityEngine;
using System.Collections;

public class ExampleMyImporterUsage : MonoBehaviour {
  public MyAssetBundleImporter.AssetBundleStruct abs;

  private string _tmpStr;
  private Object _tmpObj;
  void Start() {
    abs = new MyAssetBundleImporter.AssetBundleStruct();
    abs.assetBundleUrl = "http://yourapp.com/your/bundle.unity3d";
    abs.assetBundleVersion = 0;
    abs.assetSourceName = "YourPrefabName";

    StartCoroutine(Import());
  }

  IEnumerator Import() {
    MyAssetBundleImporter mabi = new MyAssetBundleImporter();
    yield return StartCoroutine(mabi.Import<GameObject>(abs));
    if (null != mabi.assetBundleObject) {
      _tmpObj = Instantiate(mabi.assetBundleObject);
    }
  }

  void OnGUI() {
    if (null != _tmpObj) {
      _tmpStr = _tmpObj.name + " was successfully created.";
      GUILayout.Label(_tmpStr);
    } else {
      GUILayout.Label("ERROR: Cannot import your AssetBundle.");
    }
  }
}
```

让我们描述一下最后一个脚本中发生了什么。首先，我们声明了一个公共的`AssetBundleStruct`变量和两个私有变量。然后，我们在`Start`方法中初始化我们的`AssetBundleStruct`变量。接下来，我们在`Start`函数中调用协程`Import`。在`Import`协程中，我们创建了我们类`MyAssetBundleImporter`的一个实例，并使用我们初始化的结构`abs`调用它的`Import`协程。如果我们导入了对象并且它不等于`null`，那么我们在场景中实例化那个 GameObject。此外，我们展示了简单的`GUI`标签，指示我们是否成功导入了`AssetBundle`。为了使用这个（只有如果你的`AssetBundle`已经创建并上传）脚本，你只需将其作为组件附加到你的场景中的 GameObject 上，并设置正确的值。如果这两个简单的步骤已经完成，那么你就可以在 Unity 编辑器或构建中播放和测试你的游戏。

如果你需要从你的`AssetBundle`获取包含的所有对象的数组，你应该使用名为`AssetBundle.LoadAll`的函数。在需要获取字符串标识符列表的情况下，你应该保留一个特定的`TextAsset`作为映射，以保存你的资源名称。

在以下步骤中，我们将展示一个简单的例子，关于在构建资源包之前调整不同的纹理压缩。我们只需要记住两个简单的步骤：

1.  为了在构建资源包之前强制重新导入你的资源，你应该使用`AssetDatabase.ImportAsset`函数。

1.  之后，你应该使用`AssetPostprocessor.OnPreprocessTexture`来调整纹理压缩的正确值。

现在，让我们编写一个简单的代码示例，你可以在你的项目中使用，就像这本书中的其他示例一样：

```java
  using UnityEngine;
  using UnityEditor;  
```

和往常一样，我们声明所需的命名空间（记住这个脚本将使用`UnityEditor`并且应该位于`Editor`文件夹中，我们之前已经讨论过这个 Unity 要求）。下一步是定义一个简单的类，使用以下代码：

```java
  public class TextureFormatAssetBundles {
    // the code you will see below
  }
```

在简单的类声明之后，我们需要设置一个`public`和`static`变量，同时还要设置三个不同但基本的功能（你可以使用任何想要的纹理格式；在这个例子中，我们使用`DXT1`、`DXT5`和`ETC_RGB4`）：

```java
  public static TextureImporterFormat tif;

  [MenuItem("PacktPub/AssetBundles/Create Asset Bundle DXT1")]
  static void SetTextureFormatDXT1() {
    tif = TextureImporterFormat.DXT1;
    CreateAssetBundle();
  }

  [MenuItem("PacktPub/AssetBundles/Create Asset Bundle DXT5")]
  static void SetTextureFormatDXT5() {
    tif = TextureImporterFormat.DXT5;
    CreateAssetBundle();
  }

  [MenuItem("PacktPub/AssetBundles/Create Asset Bundle ETC_RGB4")]
  static void SetTextureFormatETC_RGB4() {
    tif = TextureImporterFormat.ETC_RGB4;
    CreateAssetBundle();
  }
```

现在，我们可以编写我们的主函数`CreateAssetBundle`，它将完成所有脏活累活：

```java
  static void CreateAssetBundle() {
    // the code you will see below
  }
```

在我们的第一步中，我们调用 `EditorUtility.SaveFilePanel` 方法以显示 Unity 对话框并从中获取选定的路径字符串。此外，我们还需要从这个函数返回一个空的位置变量：

```java
  string selectedPath = EditorUtility.SaveFilePanel(
    "Save", // TITLE
    string.Empty, // DIRECTORY PATH
    "Your AssetBundle Name", // DEFAULT FILE NAME
    "unity3d" // FILE EXTENSION
  );

  if (selectedPath.Length == 0) return;
```

下一步是声明一个包含我们选择对象的数组：

```java
  Object[] selectedAssets = Selection.GetFiltered(
    typeof(Object), SelectionMode.DeepAssets
  );
```

此外，我们必须在循环中处理我们数组中的每个纹理，以便使用 `AssetDatabase.GetAssetPath` 方法获取该资产源路径。对于纹理，我们必须使用 `AssetDatabase.ImportAsset` 函数强制进行纹理预处理：

```java
  for (int i=0; i < selectedAssets.Length; i++) {
    Object obj = selectedAssets[i];
    if ((obj is Texture) == false) continue;
    string texturePath = AssetDatabase.GetAssetPath(
      (UnityEngine.Object) obj
    );
    AssetDatabase.ImportAsset(texturePath);
  }
```

之后，我们必须构建我们的 `AssetBundle`：

```java
  BuildPipeline.BuildAssetBundle(
    Selection.activeObject,
    selectedAssets,
    selectedPath,
    BuildAssetBundleOptions.CollectDependencies |
    BuildAssetBundleOptions.CompleteAssets
  );
```

在最后阶段，我们可以用我们选择的资产数组初始化 `Selection.objects` 列表变量，以便查看所有这些：

```java
  Selection.objects = selectedAssets;
```

现在，我们应该创建一个非常简单的类（这个脚本应该放在我们之前提到的 `Editor` 文件夹中），它继承自 `AssetPostprocessor` 类，如下所示：

```java
using UnityEngine;
using UnityEditor;

public class TextureAssetsPreprocessor : AssetPostprocessor {
  void OnPreprocessTexture() {
    TextureImporter ti = assetImporter as TextureImporter;
    ti.textureFormat = TextureFormatAssetBundles.tif;
  }
}
```

# 为 Android 设备实时下载新的代码和数据

在从你的包中检索资产时，你可以使用三个不同的函数：

+   `AssetBundle.Load`: 这将按给定名称加载一个对象；它也会阻塞主线程。

+   `AssetBundle.LoadAsync`: 这将按给定名称加载一个对象；它不会阻塞主线程。对于大型资产，请使用此方法。

+   `AssetBundle.LoadAll`: 这将加载你的 `AssetBundle` 中的所有对象。

在卸载资产时，可以使用 `AssetBundle.Unload` 方法。让我们看看以下代码中异步方法的简单用法示例，没有任何异常处理和任何检查（就像 `skeleton`）：

```java
using UnityEngine;
using System.Collections;

public class GetAssetBundleAsync : MonoBehaviour {
  public string assetBundleUrl = "http://yourweb.com/yourBundle.unity3d";
  public int assetBundleVersion = 1;

  IEnumerator Start() {
    WWW www = WWW.LoadFromCacheOrDownload(
      assetBundleUrl, assetBundleVersion
    );

    yield return www;

    AssetBundle ab = www.assetBundle;

    AssetBundleRequest abr = ab.LoadAsync(
      "YourObjName", typeof(GameObject)
    );

    yield return abr;

    GameObject go = abr.asset as GameObject;

    ab.Unload(false);
    www.Dispose();
  }
}
```

## 管理已加载的资产包

如果之前没有卸载之前的包，则无法加载资产包：

```java
  AssetBundle ab = www.assetBundle;
```

### 小贴士

尽量始终保留导入的资产引用，以避免多次导入相同的资产。

Unity 将抛出异常，并且你的资产包变量（在我们的例子中是 `ab` 变量）将变为 `null`。

### 小贴士

尽可能早地卸载你的 `AssetBundle`。

你可以使用以下简单的脚本（如下所示）为你加载的包使用。所有这些代码应该对你来说都很清楚，所以让我们看看这个 C# 脚本：

```java
using UnityEngine;
using System;
using System.Collections;
using System.Collections.Generic;

static public class YourAssetBundleDispatcher {
  static Dictionary<string, YourBundleReference> dictionaryBundles;

  static YourAssetBundleDispatcher() {
         dictionaryBundles = new Dictionary<string, YourBundleReference>();
  }

     private class YourBundleReference {
         public AssetBundle ab = null;

         public int assetBundleVersion;
         public string assetBundleUrl;

         public YourBundleReference(string url, int version) {
               assetBundleUrl = url;
               assetBundleVersion = version;
         }
     };

  public static AssetBundle 
        RetrieveAssetBundle(string abUrl, int abVersion) {
         string bundleKey = abUrl + abVersion.ToString();

    YourBundleReference ybr;

    if (dictionaryBundles.TryGetValue(bundleKey, out ybr))
               return ybr.ab;
         else
               return null;
     }

     public static IEnumerator ImportAssetBundle(string abUrl, int abVersion){
          string bundleKey = abUrl + abVersion.ToString();

    if (dictionaryBundles.ContainsKey(bundleKey)) {
               yield return null;
         } else {
                using(WWW www = WWW.LoadFromCacheOrDownload(abUrl, abVersion)){
                   yield return www;

                   if (www.error != null)
                         throw new Exception("WWW ERROR:" + www.error);
        YourBundleReference ybr = new YourBundleReference(
          abUrl, abVersion
        );
                   ybr.ab = www.assetBundle;
                   dictionaryBundles.Add(bundleKey, ybr);
                }
         }
  }

  public static void Dispose(string abUrl, int abVersion, bool flag) {
    string bundleKey = abUrl + abVersion.ToString();

    YourBundleReference ybr;

         if (dictionaryBundles.TryGetValue(bundleKey, out ybr)){
               ybr.ab.Unload(flag);
               ybr.ab = null;
               dictionaryBundles.Remove(bundleKey);
         }
     }
}
```

你可以使用如下所示的调度器类：

```java
using UnityEngine;
using System.Collections;

class DispatcherUsage : MonoBehaviour {
  public string assetBundleUrl;
     public int assetBundleVersion;

     AssetBundle ab;

    void Start() {
    Debug.Log("Importing your Asset Bundle");
            ab = YourAssetBundleDispatcher.RetrieveAssetBundle(
      assetBundleUrl, assetBundleVersion
    );
             if(null != ab) StartCoroutine(ImportAssetBundle());
  }

     IEnumerator ImportAssetBundle() {
         yield return StartCoroutine(
      YourAssetBundleDispatcher.ImportAssetBundle(
        assetBundleUrl, assetBundleVersion
      )
    );

         ab = YourAssetBundleDispatcher.RetrieveAssetBundle(
      assetBundleUrl, assetBundleVersion
    );
  }

  void OnDisable() {
         YourAssetBundleDispatcher.Dispose(
      assetBundleUrl, assetBundleVersion, false
    );
  }
}
```

### 注意

有可能通过调用 Unity 的 `GameObject.Instantiate` 函数来克隆之前实例化的对象，以避免不必要的导入包。

## 资产包和二进制数据

Unity 将具有 `.bytes` 扩展名的二进制文件视为 `TextAsset`，它可以包含在你的 `AssetBundle` 中。以下是一个显示的 C# 脚本示例：

```java
using UnityEngine;
using System.Collections;

public class BinaryDataExample : MonoBehaviour {
  string assetBundleUrl = 
    "http://yourweb.com/path/to/yourAssetBundle_1.unity3d";

  IEnumerator Start() {
    WWW www = WWW.LoadFromCacheOrDownload(assetBundleUrl, 1);
    yield return www;

    AssetBundle ab = www.assetBundle;

    TextAsset textAsset = ab.Load(
      "YourBinaryFileName", typeof(TextAsset)
    ) as TextAsset;

    byte[] yourBinaryData = textAsset.bytes;
  }
}
```

## 资产包和脚本

你可以用脚本作为 `TextAsset` 文件构建你的资产包，这些文件只有在预编译成程序集时才能执行。以下代码展示了这个示例：

```java
using UnityEngine;
using System.Collections;

public class AssetBundleScript : MonoBehaviour {
  string assetBundleUrl = 
    "http://yourweb.com/your/asset/bundle_test.unity3d";

  IEnumerator Start () {
    WWW www = WWW.LoadFromCacheOrDownload (assetBundleUrl, 1);
        yield return www;

        AssetBundle ab = www.assetBundle;

        TextAsset textAsset = ab.Load(
      "yourBinaryAssetName", typeof(TextAsset)
    ) as TextAsset;

        var assmbl = System.Reflection.Assembly.Load(textAsset.bytes);

    GameObject gameObj = new GameObject();
        gameObj.AddComponent(
      assmbl.GetType(
        "Your_ClassName_Inherited_From_MonoBehaviour"
      )
    );
  }
  }
```

## 资产包依赖

许多你的资源将依赖于其他资源，例如材质、纹理、着色器等。你可以构建包含所有这些资源的包，但这种方法可以减小 `AssetBundle` 文件的大小。此外，如果所有这些依赖项都用于你的其他包，这种方法将不会有效。这将浪费太多内存。相反，你可以创建一个包含所有这些共享依赖项的单独资源包，该资源包将被其他包使用。在使用这些依赖项的情况下，首先，你应该调用名为 `BuildPipeline.PushAssetDependencies` 的函数，然后你的共享包可以构建用于其他包。因此，在每次新关卡之前，你应该始终调用此函数，告诉 Unity 将你的包放入其堆栈中以便其他包进一步使用。在资源包创建结束时，你应该始终通过名为 `BuildPipeline.PopAssetDependencies` 的命令清空这个包堆栈。在你的应用程序中，你应该始终导入所有你的共享包，然后才能导入具有这些依赖项的其他包。让我们看看如何保存具有共享着色器的单独 `AssetBundle`（如图所示）在这个动作中：

```java
using UnityEngine;

public class YourAssetBundleShaders : MonoBehaviour {
        public Shader[] assetBundleShaders;
}
```

创建一个空的 GameObject 并将此脚本附加到它上，并在你的项目文件中的任何位置将其保存（在填充着色器数组之后）作为一个预制体。接下来的步骤是创建一个 C# 脚本来生成资源包，如图所示，其中 `YourAssetBundle_2` 需要 `YourAssetBundle_1`，而 `YourAssetBundle_3` 依赖于第一个和第二个包。这只是一个示例，你应该根据你的需求进行更改：

```java
using UnityEngine;
using UnityEditor;

public class AssetBundleGenerator {
  [MenuItem("PacktPub/AssetBundles/Generate all accessible Asset Bundles")]
       static void Generate() {
         BuildAssetBundleOptions options = 
                 BuildAssetBundleOptions.CollectDependencies | 
                    BuildAssetBundleOptions.CompleteAssets | 
                    BuildAssetBundleOptions.DeterministicAssetBundle;

    BuildPipeline.PushAssetDependencies();

             BuildPipeline.BuildAssetBundle(
      AssetDatabase.LoadMainAssetAtPath(
        "Assets/YourAssetName_1.prefab"
      ), 
      null, 
      "Your/Path/To/YourAssetBundle_1.unity3d", 
      options
    );

    BuildPipeline.PushAssetDependencies();
    BuildPipeline.BuildAssetBundle(
      AssetDatabase.LoadMainAssetAtPath(
        "Assets/YourAssetName_2.prefab"
      ), 
      null, 
      "YourPath/To/YourAssetBundle_2.unity3d", 
      options
    );

    BuildPipeline.BuildAssetBundle(
      AssetDatabase.LoadMainAssetAtPath(
        "Assets/YourAssetName_3.prefab"
      ), 
      null, 
      "YourPath/To/YourAssetBundle_3.unity3d", 
      options
    );           

    BuildPipeline.PopAssetDependencies();
    BuildPipeline.PopAssetDependencies();
  }

  [MenuItem("PacktPub/AssetBundles/Rebuild Asset Bundle")]
       static void Rebuild() {
    BuildAssetBundleOptions options = 
      BuildAssetBundleOptions.CollectDependencies | 
      BuildAssetBundleOptions.CompleteAssets | 
      BuildAssetBundleOptions.DeterministicAssetBundle;

    BuildPipeline.PushAssetDependencies();
    BuildPipeline.BuildAssetBundle(
      AssetDatabase.LoadMainAssetAtPath(
        "Assets/YourAssetBundleName_1.prefab"
      ), 
      null, 
      "YourPath/To/YourAssetBundle_1.unity3d", 
      options
    );

    BuildPipeline.PopAssetDependencies();
  }
  }
```

# 实践中的安全性技术

接下来，C# 脚本示例将涵盖如何保护你的资源包的内容（如图所示）：

```java
using UnityEngine;
using System.Collections;

public class AssetBundleSecurityFirst : MonoBehaviour {
  string assetBundleUrl = 
    "http://yourweb.com/path/to/yourAssetBundle.unity3d";

  IEnumerator Start() {
    WWW www = WWW.LoadFromCacheOrDownload(assetBundleUrl, 1);
    yield return www;

    TextAsset textAsset = www.assetBundle.Load(
      "YourEncryptedAssetName", typeof(TextAsset)
    ) as TextAsset;

    /*byte[] yourDecryptedBytes = AnyDecryptionFunction(
      textAsset.bytes // your encrypted bytes
    );*/
  }
}
```

另一种安全的方式是加密整个资源包，而不是仅仅加密 `TextAsset` 数据，如图中所示的前一个代码。或者，在此方法中，你不能使用 `WWW.LoadFromCacheOrDownload` 方法。你始终需要从 `WWW` 流中导入你的包，如图中所示的下述代码：

```java
using UnityEngine;
using System.Collections;

public class AssetBundleSecuritySecond : MonoBehaviour {
  string assetBundleUrl = 
    "http://yourweb.com/path/to/yourAssetBundle.unity3d";

  IEnumerator Start () {
    WWW www = new WWW(assetBundleUrl);
    yield return www;

    /*byte[] yourEncryptedBytes = www.bytes;
    byte[] yourDecryptedBytes = 
      AnyDecryptionFunction(yourEncryptedBytes);

    AssetBundleCreateRequest assetBundleCreateRequest = 
      AssetBundle.CreateFromMemory(yourDecryptedBytes);

    yield return assetBundleCreateRequest;

    AssetBundle ab = assetBundleCreateRequest.assetBundle;*/

    // Here you can use your AssetBundle. 
    // The AssetBundle was not cached.
  }
}
```

最后且最好的保护方法是将你的加密 `AssetBundle` 作为 `TextAsset` 放置在另一个（未加密的）`AssetBundle` 内部。因此，我们可以像这里所示一样使用 Unity 的缓存系统来处理我们的资源包：

```java
using UnityEngine;
using System.Collections;

public class AssetBundleSecurityThird : MonoBehaviour {
  string assetBundleUrl = 
    "http://yourweb.com/path/to/yourAssetBundle.unity3d";

  IEnumerator Start() {
    WWW www = WWW.LoadFromCacheOrDownload(assetBundleUrl, 1);
    yield return www;

    TextAsset textAsset = www.assetBundle.Load(
      "YourEncryptedAsset", typeof(TextAsset)
    ) as TextAsset;

    /*byte[] yourEncryptedBytes = textAsset.bytes;
    byte[] yourDecryptedBytes = 
      AnyDecryptionFunction(yourEncryptedBytes);

    AssetBundleCreateRequest assetBundleCreateRequest = 
      AssetBundle.CreateFromMemory(yourDecryptedBytes);
    yield return assetBundleCreateRequest;

    AssetBundle ab = assetBundleCreateRequest.assetBundle;*/
    // Here you can use your AssetBundle. The AssetBundle was cached.
  }
}
```

# 摘要

在本章中，我们发现了资源包。这里有很多代码示例。我们学习了如何构建和导入你的资源包，以及如何通过不同的方法加密资源包中的数据。此外，在本章中，我们还探讨了如何创建和使用你的 `AssetBundle` 依赖项，以及如何使用二进制数据与资源包和可执行脚本。

在下一章中，我们将介绍不同的优化技术。你将在实践中学习如何使用遮挡剔除和细节级别优化技术。你将看到如何优化原生 C#和 Unity 脚本。最后，你将了解如何将原生 C#和 JavaScript 代码转换为 Unity 脚本。
