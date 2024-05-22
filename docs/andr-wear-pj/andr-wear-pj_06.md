# 第六章：任意地点的出行方式 - WearMap 和 Google API 客户端

地图是区域或区域部分的视觉表示。

我们人类会去不同的城市旅行，这些城市可能是国内的也可能是国际的。那么，追踪你所访问的地方怎么样？我们出于不同的原因使用地图，但在大多数情况下，我们使用地图来规划特定的活动，比如户外游览、骑自行车和其他类似活动。地图帮助人类智能找到从起点到目的地的最快路线。在这个项目中，我们将构建一个与 Google Maps 服务配合工作的 Wear 应用程序。

记录一下，Google 地图最初是在 2004 年 10 月作为一个 C++ 桌面程序开始的。Google 地图在 2005 年 2 月正式发布。Google 地图提供了一个 API，允许将地图嵌入第三方应用程序中；Google 地图提供了许多地方的空中和卫星视图。与其他地图服务相比，Google 地图是最佳的，地图进行了优化且其准确率非常高。

在这个项目中，让我们构建一个独立的 Wear 地图应用程序。当用户点击地图时，我们将允许用户写下关于该地点的故事并将其保存到 SQLite 数据库中作为一个标记。当用户点击标记时，我们应该向用户展示已保存的内容。在本章中，我们将了解以下重要概念：

+   在开发者 API 控制台中创建项目

+   使用 SHA1 指纹获取 Maps API 密钥

+   SQLite 集成

+   Google 地图

+   Google API 客户端及更多功能

+   地理编码器

# 让我们开始创建 WearMap

现在我们知道如何创建一个独立的应用程序。如果你是直接跟随这个项目，而没有参考第 第二章，*让我们帮助你捕捉心中所想 - WearRecyclerView 和更多* 和第 第三章，*让我们帮助你捕捉心中所想 - 保存数据和定制 UI* 中介绍的 Wear-note 应用程序，请务必跟进 Wear-note 应用程序以了解更多关于独立应用程序的信息。

我们将这个项目称为 **WearMapDiary**，因为我们存储的是地点及其详细信息。项目的包地址由开发者决定；在这个项目中，包地址是 `com.packt.wearmapdiary`，API 级别为 25 Nougat。在活动模板中，选择 Google Maps Wear 活动，如下所示：

![](img/00070.jpeg)

从活动选择器中选择 Google Maps Wear 活动模板

创建项目后，我们将看到项目的必要配置，其中包括已经添加的地图片段；它将设置 `DismissOverlays` 组件：

![](img/00071.jpeg)

将为与 Wear 地图活动一起工作而配置的示例代码生成。

我们需要在 `res/values` 目录下的 `google_maps_api.xml` 文件中为项目添加 Maps API 密钥：

![](img/00072.jpeg)

# Google API 控制台

Google API 控制台是一个网络门户，允许开发者为他们的项目开发管理 Google 服务，可以通过[`console.developers.google.com`](https://console.developers.google.com)访问。

1.  使用你的 Google 账户访问开发者控制台。创建一个项目 `packt-wear` 或对开发者来说方便的其他名称：

![](img/00073.jpeg)

1.  成功创建项目后，前往 API 管理器 | 库部分，并启用 Google Maps Android API：

![](img/00074.jpeg)

1.  点击“启用”按钮，为 Android 启用地图：

![](img/00075.jpeg)

1.  在控制台启用 API 后，我们需要使用开发机器的 SHA1 指纹和项目的包地址创建 API 密钥，如下所示：

![](img/00076.jpeg)

1.  要获取你的设备的 SHA1 指纹，请打开 Android Studio。在 Android Studio 的右侧，你会看到 Gradle 项目菜单。然后，按照以下步骤操作：

    1.  点击“Gradle”（在右侧面板上，你会看到 Gradle 栏）

    1.  点击“刷新”（在 Gradle 栏中点击“刷新”；你将看到项目的 Gradle 脚本列表）

    1.  点击你的项目（从**列表**（根）中的项目名称）

    1.  点击“任务”

    1.  点击“Android”

1.  1.  双击 signingReport（你将在运行栏中获得 SHA1 和 MD5）：

![](img/00077.jpeg)

1.  1.  复制你的 SHA1 指纹，粘贴到 Google API 控制台，并保存：

![](img/00078.jpeg)

1.  1.  现在，从控制台复制 API 密钥，并将其粘贴到项目的 `google_maps_api.xml` 文件中，如下所示：

![](img/00079.jpeg)

+   1.  现在，将你的 Gradle 范围切换到应用，并在 Wear 模拟器或你的 Wear 设备中编译项目：

![](img/00080.jpeg)

如果你的模拟器中没有更新 Google Play 服务，Wear 会显示一个错误屏幕以更新 Play 服务：

![](img/00081.jpeg)

如果你有一个实际的 Wear 设备，当最新的 Google Play 服务更新可用时，Wear 操作系统将负责下载更新。对于模拟器，我们需要将其连接到实际设备以添加账户。首先，通过 **adb** 连接 Android 手机并启动 Wear 模拟器。

从 Play 商店安装 Android Wear 伴侣应用[`play.google.com/store/apps/details?id=com.google.android.wearable.app&hl=en`](https://play.google.com/store/apps/details?id=com.google.android.wearable.app&hl=en)。

![](img/00082.jpeg)

在 Android Wear 应用程序中，选择模拟器，然后在 Android Studio 终端中输入以下命令：

```java
adb -d forward tcp:5601 tcp:5601

```

当模拟器连接到你的真实手机后，你可以添加已经同步到手机的账户，或者添加一个新的账户。

下图展示了 Wear 账户的同步屏幕：

![](img/00083.jpeg)

+   成功添加账户后，开始更新你的 Google Play 服务：

![](img/00084.jpeg)

+   现在，完成所有这些配置后，在 Android Studio 中编译程序，并在 Wear 设备上查看地图：

![](img/00085.jpeg)

# Google API 客户端

`GoogleApiClient`扩展了**Object**类。Google API 客户端为所有 Google Play 服务提供了一个共同的入口点，并在用户设备与每个 Google 服务之间管理网络连接。Google 建议使用`GoogleApiClient`以编程方式获取用户的位置。

在每个线程上创建`GoogleApiClient`。`GoogleApiClient`服务连接在内部被缓存。`GoogleApiClient`实例不是线程安全的，因此创建多个实例很快。`GoogleApiClient`与各种静态方法一起使用。其中一些方法要求`GoogleApiClient`已连接；有些会在`GoogleApiClient`连接之前排队调用。

下面是一个与 Google `LocationServices` 连接创建`GoogleApiClient`实例的代码示例：

```java
GoogleApiClient mGoogleApiClient = new GoogleApiClient.Builder(this)
.addConnectionCallbacks(this)
.addOnConnectionFailedListener(this)
.addApi(LocationServices.API)
.build();

```

# 配置项目以实现功能

我们知道为了更好的代码管理和将来维护代码，创建包的重要性。让我们为项目创建一个包含四个不同名称的包，分别是 adapter、model、util 和 view。

我们在 model 包内编写我们的普通旧 Java 对象。我们将在 util 包中配置所有与数据库相关的类，以及在 view 包中配置自定义视图，如对话框片段、`TextView`等。对于自定义`infoWindow`，我们必须在`adapter`包内创建一个`infoWindowAdapter`。

使用`GoogleApiClient`获取位置信息非常重要。现在我们已经配置了 Wear 地图活动，并使用我们添加的 API 密钥绘制了地图，是时候利用`GoogleApiClient`获取位置详情了。

# 利用`GoogleApiClient`获取用户的位置信息

现在，在`MapActivity`类中，我们需要实现以下接口：

+   `GoogleApiClient.ConnectionCallback`

+   `GoogleApiClient.OnConnectionFailedListener`

然后，我们需要从这两个接口重写三个方法，它们分别是：

+   `public void onConnected(..){}`

+   `public void onConnectionSuspended(..){}`

+   `public void onConnectionFailed(..){}`

在`onConnected`方法中，我们可以使用`GoogleApiClient`实例实例化位置服务。首先，让我们将`GoogleApiClient`添加到项目中。在`MapActivity`的全局范围内创建一个`GoogleApiClient`实例：

```java
private GoogleApiClient mGoogleApiClient;

```

添加一个名为`addGoogleAPIClient(){ }`的 void 方法，用于获取位置服务 API：

```java
private void addGoogleAPIClient(){
    mGoogleApiClient = new GoogleApiClient.Builder(this)
            .addConnectionCallbacks(this)
            .addOnConnectionFailedListener(this)
            .addApi(LocationServices.API)
            .build();
}

```

为了让 Google Play 服务处理与位置相关的任务，请在 gradle wear 模块中添加以下依赖项：

`compile 'com.google.android.gms:play-services-location:11.0.2'`

现在，在`onConnected`方法中，附加`mGoogleApiClient`：

```java
@Override
public void onConnected(@Nullable Bundle bundle) {

    Location location =      LocationServices.FusedLocationApi
    .getLastLocation(mGoogleApiClient);
    double latitude = location.getLatitude();
    double longitude = location.getLongitude();

}

```

`Locationservice`在请求位置之前需要权限检查。让我们在 manifest 和 Activity 中添加权限。

在 Manifest 中添加以下权限：

```java
<uses-permission android:name="android.permission.INTERNET"/>
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE"/>
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
<!-- The following two permissions are not required to use
     Google Maps Android API v2, but are recommended. -->
<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION"/>
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION"/>

```

在`MapActivity.java`类中编写一个检查权限的方法，如下所示：

```java
private boolean checkPermission(){
    int result = ContextCompat.checkSelfPermission(MapsActivity.this, 
    Manifest.permission.ACCESS_FINE_LOCATION);
    if (result == PackageManager.PERMISSION_GRANTED){

        return true;

    } else {

        return false;

    }
}

```

按如下方式重写`onRequestPermissionsResult(..){}`方法：

```java
@Override
public void onRequestPermissionsResult(int requestCode, String permissions[], int[] grantResults) {
    switch (requestCode) {
        case PERMISSION_REQUEST_CODE:
            if (grantResults.length > 0 && grantResults[0] == 
            PackageManager.PERMISSION_GRANTED) {

            } else {

            }
            break;
    }
}

```

现在，我们有了权限检查方法；在`onConnected`方法中处理它：

```java
@Override
public void onConnected(@Nullable Bundle bundle) {
    if (checkPermission()) {
        Location location = LocationServices.FusedLocationApi
        .getLastLocation(mGoogleApiClient);
        double latitude = location.getLatitude();
        double longitude = location.getLongitude();
    }else{

    }

}

```

让我们编写一个方法来检查 Wear 设备上是否内置了 GPS。通过使用`packagemanager`类，我们可以检索 Wear 设备上可用的硬件。让我们写一个名为`hasGps()`的方法：

```java
private boolean hasGps() {
    return getPackageManager().hasSystemFeature(
      PackageManager.FEATURE_LOCATION_GPS);
}

```

如果你想要用户知道他们的设备是否有 GPS 设备，或者在开发过程中只是想要记录下来，你可以在`onCreate()`方法中使用这个方法：

```java
if (!hasGps()) {
    Log.d(TAG, "This hardware doesn't have GPS.");
    // Fall back to functionality that does not use location or
    // warn the user that location function is not available.
}

```

如果你的可穿戴应用使用内置 GPS 记录数据，你可能想要通过实现`onLocationChanged()`方法，使用`LocationListner`接口将位置数据与手持设备同步。

要使你的应用能够感知位置，请使用`GoogleAPIclient`。

想要了解更多关于权限的信息，请点击这个链接：[`developer.android.com/training/articles/wear-permissions.html`](https://developer.android.com/training/articles/wear-permissions.html)。

现在，让我们处理`onMapclick`方法，以处理在地图上添加标记的过程。为此，在你的活动中实现`GoogleMap.OnMapClickListener`并实现其回调方法，这将为你提供带有经纬度的`onmapclick`。将点击上下文添加到你的`onMapReady`回调中，如下所示：

```java
mMap.setOnMapClickListener(this);

```

在`onMapClick`方法中，我们可以使用`latLng`添加以下标记：

```java
@Override
public void onMapClick(LatLng latLng) {
    Log.d(TAG, "Latlng is "+latLng);
}

```

在`onMapclick`方法中添加标记使用`MarkerOptions()`。对于谷歌设计的高级标记，我们将使用地图的`addmarker`方法，并添加带有位置、标题和摘要（标题下方的简短描述）的新`MarkerOptions`：

```java
@Override
public void onMapClick(LatLng latLng) {
    Log.d(TAG, "Latlng is "+latLng);
    mMap.addMarker(new MarkerOptions()
            .position(latLng)
            .title("Packt wear 2.0")
            .snippet("Map is cool in wear device"));
}

```

添加带有`infowindow`的标记后：

![](img/00086.jpeg)

现在，我们已经有了地图，并且正在向地图添加标记，但我们需要处理地理编码以获取坐标的地址名称。

# 使用 GeoCoder 的地理空间数据

使用`GeoCoder`类通过坐标获取地址。地理编码通常是将街道地址或位置的其它描述转换为（纬度，经度）坐标的过程。逆地理编码是将（纬度，经度）坐标转换为（部分）地址的过程。

在`OnMapClick`方法中，进行以下更改：

```java
@Override
public void onMapClick(LatLng latLng) {
    Log.d(TAG, "Latlng is "+latLng);

   //Fetching the best address match
    Geocoder geocoder = new Geocoder(this);
 List<Address> matches = null;
 try {
 matches = geocoder.getFromLocation(latLng.latitude, 
        latLng.longitude, 1);
 } catch (IOException e) {
 e.printStackTrace();
 }
 Address bestAddress = (matches.isEmpty()) ? null : matches.get(0);
 int maxLine = bestAddress.getMaxAddressLineIndex(); 
    mMap.addMarker(new MarkerOptions()
            .position(latLng)
            .title(bestAddress.getAddressLine(maxLine - 1))
            .snippet(bestAddress.getAddressLine(maxLine)));
}

```

上述代码片段将标记添加到地图上，并在信息窗口中显示位置名称：

![](img/00087.jpeg)

在点击地图时弹出的视图在 Android 中称为`infowindow`。它类似于网页开发中的 ToolTip 组件。在这个项目中，我们需要在用户点击地图的任何地方保存数据；我们需要借助`infowindow`显示自定义地图标记。我们需要编写一个适配器，实现`GoogleMap.InfoWindowAdapter`与自定义布局，如下所示：

# 信息窗口适配器

以下实现解释了如何为地图标记编写我们自定义的`infowindow`适配器：

```java
//XML latout for customising  infowindow 
<LinearLayout 
              android:orientation="horizontal"
              android:layout_width="match_parent"
              android:layout_height="match_parent">
    <TextView
        android:id="@+id/title"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"/>
    <TextView
        android:id="@+id/snippet"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"/>
</LinearLayout>

```

`windowadapter`类实现了`GoogleMap.InfoWindowAdapter`，包含两个回调方法`getInfoWindow(..){}`和`getInfoContents(..){}`。我们可以通过`getInfoContent`方法来填充自定义布局：

```java
public class WearInfoWindowAdapter implements GoogleMap.InfoWindowAdapter {
    private LayoutInflater mLayoutInflater;
    private View mView;
    MarkerAdapter(LayoutInflater layoutInflater){
        mLayoutInflater = layoutInflater;
    }
    @Override
    public View getInfoWindow(Marker marker) {
        return null;
    }
    @Override
    public View getInfoContents(Marker marker) {
        if (mView == null){
            mView = mLayoutInflater.inflate(R.layout.marker, null);
        }
        TextView titleView = (TextView)mView.findViewById(R.id.title);
        titleView.setText(marker.getTitle());
        TextView snippetView = 
        (TextView)mView.findViewById(R.id.snippet);
        snippetView.setText(marker.getSnippet());
        return mView;
    }
}

```

为了更好的代码访问和维护，将前面的适配器类添加到 adapter 包中。`InfoWindowAdapter`没有使用任何数据来填充视图；我们使用与标记关联的任何数据来填充视图。如果我们想在标题和摘要之外添加任何内容，适配器本身无法做到这一点。我们需要创建一个机制以编程方式实现这一点。

在 model 包中创建`Memory`类。`Memory`类是用户选择添加标记的地方：

```java
public class Memory {
    double latitude;
    double longitude;
    String city; // City name
    String country; // Country name
    String notes; // saving notes on the location 
}

```

现在，我们已经有了 memory，自定义`infowindow`适配器已准备好与`onMapclick`实现一起工作。对于每个标记，我们将添加一个 memory 类关联。为了临时保存所有 memory，让我们使用`HashMap`：

```java
private HashMap<String, Memory> mMemories = new HashMap<>();

```

让我们将标记添加到`HashMap`中，以便访问`Marker`属性，例如`Marker` ID 等。适配器的完整代码如下：

```java
public class WearInfoWindowAdapter implements GoogleMap.InfoWindowAdapter {

    public LayoutInflater mLayoutInflater;
    public View mView;
    public HashMap<String, Memory> mMemories;

    WearInfoWindowAdapter(LayoutInflater layoutInflater, 
    HashMap<String,Memory> memories){
        mLayoutInflater = layoutInflater;
        mMemories = memories;
    }

    @Override
    public View getInfoWindow(Marker marker) {
        return null;
    }

    @Override
    public View getInfoContents(Marker marker) {
        if (mView == null) {
            mView = mLayoutInflater.inflate(R.layout.marker, null);
        }
        Memory memory = mMemories.get(marker.getId());

        TextView titleView = (TextView)mView.findViewById(R.id.title);
        titleView.setText(memory.city);
        TextView snippetView = 
        (TextView)mView.findViewById(R.id.snippet);
        snippetView.setText(memory.country);
        TextView notesView = (TextView)mView.findViewById(R.id.notes);
        notesView.setText(memory.notes);

        return mView;
    }
}

```

在`OnMapClick`方法中，添加以下更改：

```java
@Override
public void onMapClick(LatLng latLng) {
    Log.d(TAG, "Latlng is "+latLng);

    Geocoder geocoder = new Geocoder(this);
    List<Address> matches = null;
    try {
        matches = geocoder.getFromLocation(latLng.latitude, 
        latLng.longitude, 1);
    } catch (IOException e) {
        e.printStackTrace();
    }

    Address bestAddress = (matches.isEmpty()) ? null : matches.get(0);
    int maxLine = bestAddress.getMaxAddressLineIndex();

    Memory memory = new Memory();
    memory.city = bestAddress.getAddressLine(maxLine - 1);
    memory.country = bestAddress.getAddressLine(maxLine);
    memory.latitude = latLng.latitude;
    memory.longitude = latLng.longitude;
    memory.notes = "Packt and wear 2.0 notes...";

    Marker marker = mMap.addMarker(new MarkerOptions()
            .position(latLng));

    mMemories.put(marker.getId(), memory);
}

```

使用以下代码在`onMapready`方法中将新的`Marker`附加到地图上：

```java
mMap.setInfoWindowAdapter(new WearInfoWindowAdapter(getLayoutInflater(), mMemories));

```

现在，编译程序。你应该能够看到如下更新的`infoWindow`：

![](img/00088.jpeg)

# 用于记录位置信息的自定义`DialogFragment`

`DialogFragment`是一个在活动中浮动的对话框窗口。在 Wear 设备上它不会浮动，但它提供了 Wear 优化的设计。查看以下实现代码。

在继续之前，将 Memory 类实现为可序列化接口：

```java
public class Memory implements Serializable {
    public double latitude;
    public double longitude;
    public String city;
    public String country;
    public String notes;
}

```

在 layout 目录中添加以下布局文件，并将布局文件命名为`memory_dialog_fragment.xml`。创建文件后，在布局文件内添加以下代码：

```java
<?xml version="1.0" encoding="utf-8"?>
<android.support.wearable.view.BoxInsetLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:id="@+id/container"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MapsActivity"
    tools:deviceIds="wear">

    <LinearLayout
        xmlns:android="http://schemas.android.com/apk/res/android"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="vertical"
        android:padding="5dp"
        app:layout_box="all"
        android:layout_gravity="center"
        android:gravity="center">

        <TextView
            android:id="@+id/city"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content" />

        <TextView
            android:id="@+id/country"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content" />

        <EditText
            android:id="@+id/notes"
            android:layout_width="match_parent"
            android:layout_height="wrap_content" />

    </LinearLayout>
</android.support.wearable.view.BoxInsetLayout>

```

创建布局文件后，让我们处理创建自定义对话框的 Java 代码。创建一个名为`MemoryDialogFragment`的类，并继承自`DialogFragment`。

创建一个接口来处理`DialogFragment`的`SaveClicked`和`cancelClicked`按钮：

```java
public interface Listener{
    public void OnSaveClicked(Memory memory);
    public void OnCancelClicked(Memory memory);
}

```

现在，将以下实例添加到`MemoryDialogFragment`的全局范围内。：

```java
private static final String TAG = "MemoryDialogFragment";
private static final String MEMORY_KEY = "MEMORY";
private Memory mMemory;
private Listener mListener;
private View mView;

```

现在，让我们处理正确地在正确字段中填充布局的数据：

```java
@Override
public Dialog onCreateDialog(Bundle savedInstanceState) {

    mView = getActivity().getLayoutInflater()
    .inflate(R.layout.memory_dialog_fragment, null);
    TextView cityView = (TextView) mView.findViewById(R.id.city);
    cityView.setText(mMemory.city);
    TextView countryView = (TextView) mView.findViewById(R.id.country);
    countryView.setText(mMemory.country);

    AlertDialog.Builder builder = new 
    AlertDialog.Builder(getActivity());
    builder.setView(mView)
            .setTitle(getString(R.string.dialog_title))
            .setPositiveButton(getString(R.string.DialogSaveButton), 
            new DialogInterface.OnClickListener() {
                @Override
                public void onClick(DialogInterface dialog, int which) 
                {
                    EditText notesView = (EditText) 
                    mView.findViewById(R.id.notes);
                    mMemory.notes = notesView.getText().toString();
                    mListener.OnSaveClicked(mMemory);
                }
            })
            .setNegativeButton(getString(R.string.DialogCancelButton), 
            new DialogInterface.OnClickListener() {
                @Override
                public void onClick(DialogInterface dialog, int which) 
                {
                    mListener.OnCancelClicked(mMemory);
                }
            });

    return builder.create();
}

```

我们将在`oncreate`方法中从`Memory`获取序列化的数据：

```java
@Override
public void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    Bundle args = getArguments();
    if (args != null){
        mMemory = (Memory)args.getSerializable(MEMORY_KEY);
    }
}

```

`MemoryDialogFragment`的完整代码如下：

```java
public class MemoryDialogFragment extends DialogFragment  {

    private static final String TAG = "MemoryDialogFragment";
    private static final String MEMORY_KEY = "MEMORY";

    private Memory mMemory;
    private Listener mListener;
    private View mView;

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        Bundle args = getArguments();
        if (args != null){
            mMemory = (Memory)args.getSerializable(MEMORY_KEY);
        }
    }

    @Override
    public Dialog onCreateDialog(Bundle savedInstanceState) {

        mView = getActivity().getLayoutInflater()
        .inflate(R.layout.memory_dialog_fragment, null);
        TextView cityView = (TextView) mView.findViewById(R.id.city);
        cityView.setText(mMemory.city);
        TextView countryView = (TextView) 
        mView.findViewById(R.id.country);
        countryView.setText(mMemory.country);

        AlertDialog.Builder builder = new 
        AlertDialog.Builder(getActivity());
        builder.setView(mView)
                .setTitle(getString(R.string.dialog_title))
                .setPositiveButton(getString
                (R.string.DialogSaveButton), 
                new DialogInterface.OnClickListener() {
                    @Override
                    public void onClick
                    (DialogInterface dialog, int which) {
                        EditText notesView = (EditText) 
                        mView.findViewById(R.id.notes);
                        mMemory.notes = notesView.getText().toString();
                        mListener.OnSaveClicked(mMemory);
                    }
                })
                .setNegativeButton(getString
                (R.string.DialogCancelButton), 
                new DialogInterface.OnClickListener() {
                    @Override
                    public void onClick
                    (DialogInterface dialog, int which) {
                        mListener.OnCancelClicked(mMemory);
                    }
                });

        return builder.create();
    }

    public static MemoryDialogFragment newInstance(Memory memory){
        MemoryDialogFragment fragment = new MemoryDialogFragment();
        Bundle args = new Bundle();
        args.putSerializable(MEMORY_KEY, memory);
        fragment.setArguments(args);

        return fragment;
    }

    @Override
    public void onAttach(Activity activity) {
        super.onAttach(activity);
        try{
            mListener = (Listener)getActivity();
        }catch (ClassCastException e){
            throw new IllegalStateException("Activity does not 
            implement contract");
        }

    }

    @Override
    public void onDetach() {
        super.onDetach();
        mListener = null;
    }

    public interface Listener{
        public void OnSaveClicked(Memory memory);
        public void OnCancelClicked(Memory memory);
    }
}

```

在`OnMapClick`方法中，进行以下更改：

```java
@Override
public void onMapClick(LatLng latLng) {
    Log.d(TAG, "Latlng is "+latLng);

    Memory memory = new Memory();
 updateMemoryPosition(memory, latLng);
    MemoryDialogFragment.newInstance(memory)
    .show(getFragmentManager(),MEMORY_DIALOG_TAG);
}

```

现在，编译程序。在`mapclick`时，你将看到以下屏幕。用户可以在 edittext 字段中输入关于地图位置的自己的想法：

![](img/00089.jpeg)

现在我们已经添加了输入对话框，让我们来处理将数据保存到 SQLite 的操作。

# 配置 SQLite 并保存标记

对于任何优秀的软件来说，持久化所有必要的数据都是基本用例。Android SDK 内置了 SQLite 存储解决方案。它占用的空间非常小，速度也非常快。如果程序员熟悉 SQL 查询和操作，那么使用 SQLite 将会轻松愉快。

# 模式和合约

本质上，对于数据库，我们需要创建一个数据模式，这是对数据库组织方式的正式声明。该模式反映在 SQLite 查询语句中。合约类是一个包含常量的容器，这些常量定义了 URI、表和列的名称。合约类允许在同一个包中的所有其他类中使用相同的常量。

对于`WearMapDiary`的范围，我们将在`DBHelper`类中创建所有实例。现在，让我们创建`DBhelper`类，它打开并连接应用程序到 SQLite，并处理查询：

```java
public class DbHelper extends SQLiteOpenHelper {

    private static final String DATABASE_NAME = "traveltracker.db";
    private static final int DATABASE_VERSION = 3;
    public static final String MEMORIES_TABLE = "memories";
    public static final String COLUMN_LATITUDE = "latitude";
    public static final String COLUMN_LONGITUDE = "longitude";
    public static final String COLUMN_CITY = "city";
    public static final String COLUMN_COUNTRY = "country";
    public static final String COLUMN_NOTES = "notes";
    public static final String COLUMN_ID = "_id";

    private static DbHelper singleton = null;

    public static DbHelper getInstance(Context context){
        if (singleton == null){
            singleton = new DbHelper(context.getApplicationContext());
        }
        return singleton;
    }

    private DbHelper(Context context) {
        super(context, DATABASE_NAME, null, DATABASE_VERSION);
    }

    @Override
    public void onCreate(SQLiteDatabase db) {
        db.execSQL("CREATE TABLE "+MEMORIES_TABLE+" ("
                +COLUMN_ID+" INTEGER PRIMARY KEY AUTOINCREMENT, "
                +COLUMN_LATITUDE +" DOUBLE, "
                +COLUMN_LONGITUDE +" DOUBLE, "
                +COLUMN_CITY +" TEXT, "
                +COLUMN_COUNTRY +" TEXT, "
                +COLUMN_NOTES +" TEXT"
                +")");
    }

    @Override
    public void onUpgrade(SQLiteDatabase db, int oldVersion, int 
    newVersion) {
        db.execSQL("DROP TABLE IF EXISTS "+MEMORIES_TABLE);
        onCreate(db);
    }
}

```

我们需要创建一个`Datasource`来管理所有查询，以及读写 SQLite 中的数据。在这里，这个类中，我们将创建多个方法来创建数据、读取数据、更新数据和删除数据：

```java
public class MemoriesDataSource {
    private DbHelper mDbHelper;
    private String[] allColumns = {
            DbHelper.COLUMN_ID, DbHelper.COLUMN_CITY,
            DbHelper.COLUMN_COUNTRY, DbHelper.COLUMN_LATITUDE,
            DbHelper.COLUMN_LONGITUDE, DbHelper.COLUMN_NOTES
    };

    public MemoriesDataSource(Context context){
        mDbHelper = DbHelper.getInstance(context);
    }

    public void createMemory(Memory memory){
        ContentValues values = new ContentValues();
        values.put(DbHelper.COLUMN_NOTES, memory.notes);
        values.put(DbHelper.COLUMN_CITY, memory.city);
        values.put(DbHelper.COLUMN_COUNTRY, memory.country);
        values.put(DbHelper.COLUMN_LATITUDE, memory.latitude);
        values.put(DbHelper.COLUMN_LONGITUDE, memory.longitude);
        memory.id = mDbHelper.getWritableDatabase()
        .insert(DbHelper.MEMORIES_TABLE, null, values);
    }

    public List<Memory> getAllMemories(){

        Cursor cursor = allMemoriesCursor();
        return cursorToMemories(cursor);
    }

    public Cursor allMemoriesCursor(){
        return mDbHelper.getReadableDatabase()
        .query(DbHelper.MEMORIES_TABLE, 
        allColumns,null, null, null, null, null);
    }

    public List<Memory> cursorToMemories(Cursor cursor){
        List<Memory> memories =  new ArrayList<>();
        cursor.moveToFirst();
        while (!cursor.isAfterLast()){
            Memory memory = cursorToMemory(cursor);
            memories.add(memory);
            cursor.moveToNext();
        }
        return memories;
    }

    public void updateMemory(Memory memory){
        ContentValues values = new ContentValues();
        values.put(DbHelper.COLUMN_NOTES, memory.notes);
        values.put(DbHelper.COLUMN_CITY, memory.city);
        values.put(DbHelper.COLUMN_COUNTRY, memory.country);
        values.put(DbHelper.COLUMN_LATITUDE, memory.latitude);
        values.put(DbHelper.COLUMN_LONGITUDE, memory.longitude);

        String [] whereArgs = {String.valueOf(memory.id)};

        mDbHelper.getWritableDatabase().update(
                mDbHelper.MEMORIES_TABLE,
                values,
                mDbHelper.COLUMN_ID+"=?",
                whereArgs
        );
    }

    public void deleteMemory(Memory memory){
        String [] whereArgs = {String.valueOf(memory.id)};

        mDbHelper.getWritableDatabase().delete(
                mDbHelper.MEMORIES_TABLE,
                mDbHelper.COLUMN_ID+"=?",
                whereArgs
        );
    }

    private Memory cursorToMemory(Cursor cursor){
        Memory memory = new Memory();
        memory.id = cursor.getLong(0);
        memory.city = cursor.getString(1);
        memory.country = cursor.getString(2);
        memory.latitude = cursor.getDouble(3);
        memory.longitude = cursor.getDouble(4);
        memory.notes = cursor.getString(5);
        return memory;
    }
}

```

为了在后台使用`cursorLoader`执行所有这些查询，我们将编写另一个类，我们将这个类称为`DBCurserLoader`：

```java
public abstract class DbCursorLoader extends AsyncTaskLoader<Cursor> {

    private Cursor mCursor;

    public DbCursorLoader(Context context){
        super(context);
    }

    protected abstract Cursor loadCursor();

    @Override
    public Cursor loadInBackground() {
        Cursor cursor = loadCursor();
        if (cursor != null){
            cursor.getCount();
        }

        return cursor;
    }

    @Override
    public void deliverResult(Cursor data) {
        Cursor oldCursor = mCursor;
        mCursor = data;

        if (isStarted()){
            super.deliverResult(data);
        }

        if (oldCursor != null && oldCursor != data){
            onReleaseResources(oldCursor);
        }
    }

    @Override
    protected void onStartLoading() {
        if (mCursor != null){
            deliverResult(mCursor);
        }
        if (takeContentChanged() || mCursor == null){
            forceLoad();
        }
    }

    @Override
    protected void onStopLoading() {
        cancelLoad();
    }

    @Override
    public void onCanceled(Cursor data) {
        super.onCanceled(data);

        if (data != null) {
            onReleaseResources(data);
        }
    }

    @Override
    protected void onReset() {
        super.onReset();

        onStopLoading();

        if (mCursor != null){
            onReleaseResources(mCursor);
        }
        mCursor = null;
    }

    private void onReleaseResources(Cursor cursor){
        if (!cursor.isClosed()){
            cursor.close();
        }
    }
}

```

创建另一个类，用于从`memoryDatasource`加载所有记忆，并扩展到`DBCursorLoader`：

```java
public class MemoriesLoader extends DbCursorLoader {

    private MemoriesDataSource mDataSource;

    public MemoriesLoader(Context context, MemoriesDataSource 
    memoriesDataSource){
        super(context);
        mDataSource = memoriesDataSource;
    }

    @Override
    protected Cursor loadCursor() {
        return mDataSource.allMemoriesCursor();
    }
}

```

现在，我们的 SQLite 配置工作正常。让我们在`MapActivity`中处理保存数据到 SQLite 的`onMapclick`。

# 在 SQLite 中保存数据

要将 SQLite 连接到活动，并在 SQLite 中保存数据，请实现活动`LoaderManager.LoaderCallbacks<Cursor>`并在`onCreate`方法中实例化数据源：

```java
mDataSource = new MemoriesDataSource(this);
getLoaderManager().initLoader(0,null,this);

```

实现`LoaderManager.LoaderCallbacks<Cursor>`接口的回调方法：

```java
@Override
public Loader<Cursor> onCreateLoader(int id, Bundle args) {
    return null;
}

@Override
public void onLoadFinished(Loader<Cursor> loader, Cursor data) {

}

@Override
public void onLoaderReset(Loader<Cursor> loader) {

}

```

现在，将`addingMarker`代码重构为一个方法，如下所示：

```java
private void addMarker(Memory memory) {
    Marker marker = mMap.addMarker(new MarkerOptions()
            .draggable(true)
            .position(new LatLng(memory.latitude, memory.longitude)));

    mMemories.put(marker.getId(), memory);
}

```

我们仍然需要处理拖动标记以用于将来的实现。让我们将可拖动属性设置为 true。现在，在`OnMapClick`方法中，调用以下代码：

```java
@Override
public void onMapClick(LatLng latLng) {
    Log.d(TAG, "Latlng is "+latLng);

    Memory memory = new Memory();
    updateMemoryPosition(memory, latLng);
    MemoryDialogFragment.newInstance(memory)
    .show(getFragmentManager(),MEMORY_DIALOG_TAG);
}

```

让我们重构`UpdateMemoryPosition`方法，它从`latlng`获取地址并将其添加到`Memory`：

```java
private void updateMemoryPosition(Memory memory, LatLng latLng) {
    Geocoder geocoder = new Geocoder(this);
    List<Address> matches = null;
    try {
        matches = geocoder.getFromLocation(latLng.latitude, 
        latLng.longitude, 1);
    } catch (IOException e) {
        e.printStackTrace();
    }

    Address bestMatch = (matches.isEmpty()) ? null : matches.get(0);
    int maxLine = bestMatch.getMaxAddressLineIndex();
    memory.city = bestMatch.getAddressLine(maxLine - 1);
    memory.country = bestMatch.getAddressLine(maxLine);
    memory.latitude = latLng.latitude;
    memory.longitude = latLng.longitude;
}

```

现在，我们正在 SQLite 中保存数据。当我们关闭并重新打开地图时，我们没有读取并将标记数据添加到地图中：

![](img/00090.jpeg)

现在，让我们读取 SQLite 数据并将其添加到地图中。

`LoaderManager`类的`onCreateLoader`回调方法通过`Datasource`实例将数据添加到`MemoryLoader`，如下所示：

```java
@Override
public Loader<Cursor> onCreateLoader(int id, Bundle args) {
    Log.d(TAG,"onCreateLoader");
    return new MemoriesLoader(this, mDataSource);
}

```

在`onLoadFinished`方法中，从游标中获取数据并将其添加到地图中：

```java
@Override
public void onLoadFinished(Loader<Cursor> loader, Cursor cursor) {
    Log.d(TAG,"onLoadFinished");
    onFetchedMemories(mDataSource.cursorToMemories(cursor));
}

```

从获取的数据中向地图添加标记：

```java
private void onFetchedMemories(List<Memory> memories) {
    for(Memory memory: memories){
        addMarker(memory);
    }
}

```

现在，我们有一个名为`WearMapDiary`的功能性 Wear 应用，它查找地址并在地图上关于位置保存快速笔记。它将标记添加到 SQLite 数据库中，并在我们在 Wear 设备上打开应用程序时将标记附加到地图上：

![](img/00091.jpeg)

从应用中检索笔记和位置信息：

![图片](img/00092.jpeg)

在此对话框中，用户可以输入他或她想要保存在当前位置的数据：

![图片](img/00093.jpeg)

到目前为止，我们已经探讨了如何将地图集成到穿戴设备上，并清楚地了解了如何获取地图 API 密钥。我们使用`GoogleApiclient`来获取位置服务。我们正在检查 GPS 硬件的可用性：

以下步骤简要介绍了如何编写自定义标记。

+   通过实现`GoogleMap.InfoWindowAdapter`探索了自定义`InfoWindow`适配器。

+   使用`boxinsetlayout`为 Wear 兼容性创建了自定义的`dialogFragment`。

+   `Geocoder`类获取`GeoSpatial`数据

+   SQLite 及其与地图数据的集成

现在，是时候了解关于穿戴设备上的地图的更多信息了。

# 独立地图应用与移动同步地图应用之间的区别

面向 Wear 2.0 平台的手表应用可以通过板载 Wi-Fi 传输器连接到 Wi-Fi。我们可以缓存地图等更多内容，但它仍然缺乏移动地图应用程序的舒适性。通常，对于独立的 Wear 应用，目标 API 级别是 25，并带有安全操作运行时权限。在本章中，我们添加了处理运行时权限的代码。

# 将应用标识为独立应用

Wear 2.0 要求手表应用的 Android `Manifest`文件中有一个新的元数据元素，作为`<application>`元素的子元素。新元数据元素的名称是`com.google.android.wearable.standalone`，值必须是 true 或 false：

```java
 <meta-data
    android:name="com.google.android.wearable.standalone"
    android:value="true" />

```

由于独立应用是独立或半独立的，因此 iPhone 用户和缺少 Play 商店的 Android 手机（如 BlackBerry android 分叉操作系统和诺基亚定制 Android 手机）可以安装它们。

如果手表应用依赖于手机应用，请将前一个元数据元素的值设置为 false。

即使值是 false，手表应用也可以在相应的手机应用安装之前安装。因此，如果手表应用检测到配套手机缺少必要的手机应用，手表应用应提示用户安装手机应用。

# 在手表应用和手机应用之间共享数据

手表应用和手机应用之间可以共享数据，或者共享特定于应用的数据。你可以使用标准的 Android 存储 API 在本地存储数据。例如，你可以使用[SharedPreferences APIs](https://developer.android.com/reference/android/content/SharedPreferences.html)，SQLite，或者内部存储（就像在手机上一样）。消息传递 API 的手表应用可以与对应的手机应用通信。

# 从另一台设备上检测你的应用

在**CapabilityAPI**中，你的 Wear 应用可以检测到与 Wear 应用对应的手机应用。Wear 设备可以自发地以及静态地向配对的设备广播它们的事件。要检查配对 Wear 设备宣传的功能，更多信息请查看此链接：[`developer.android.com/training/wearables/data-layer/messages.html#AdvertiseCapabilities`](https://developer.android.com/training/wearables/data-layer/messages.html#AdvertiseCapabilities)。

请注意，并非所有手机都支持 Play 商店（如 iPhone 等）。本节描述了这些情况的最佳实践：你的独立手表应用需要你的手机应用，而你的手机应用也需要你的独立手表应用。

# 指定功能名称以检测你的应用

对于每种设备类型（手表或手机）对应的应用，请在`res/values/wear.xml`文件中为功能名称指定一个唯一的字符串。例如，在你的移动模块中，`wear.xml`文件可能包含以下代码，在 Wear 和移动模块中：

```java
<resources>
    <string-array name="android_wear_capabilities">
        <item>verify_remote_example_phone_app</item>
    </string-array>
</resources>

```

# 检测并引导用户安装相应的手机应用

Wear 2.0 引入了独立应用程序。Wear 应用足够强大，可以在没有移动应用支持的情况下运行。在必须要有移动应用的紧急情况下，Wear 应用可以指导用户安装移动支持应用和相应的 Wear 应用，通过以下步骤：

+   使用`CapabilityApi`检查你的手机应用是否已安装在配对的手机上。更多信息，请查看谷歌提供的这个示例：[`github.com/googlesamples/android-WearVerifyRemoteApp.`](https://github.com/googlesamples/android-WearVerifyRemoteApp)

+   如果你的手机应用没有安装在手机上，使用`PlayStoreAvailability.getPlayStoreAvailabilityOnPhone()`来检查它是什么类型的手机。

+   如果返回`PlayStoreAvailability.PLAY_STORE_ON_PHONE_AVAILABLE`为`true`，表示手机中已安装 Play 商店。

+   在 Wear 设备上调用`RemoteIntent.startRemoteActivity()`，使用市场 URI（[market://details?id=com.example.android.wearable.wear.finddevices](http://use%20a%20market%20URI%20such%20as:%20market://details?id=com.example.android.wearable.wear.finddevices)）在手机上打开 Play 商店。

+   如果返回`PlayStoreAvailability.PLAY_STORE_ON_PHONE_UNAVAILABLE`，这意味着该手机很可能是 iOS 手机（没有 Play 商店）。通过在 Wear 设备上调用`RemoteIntent.startRemoteActivity()`并使用此 URI 打开 iPhone 上的 App Store：[`itunes.apple.com/us/app/yourappname`](https://itunes.apple.com/us/app/yourappname)。也请参阅从手表打开 URL。在 iPhone 上，从 Android Wear，你无法编程确定你的手机应用是否已安装。作为最佳实践，为用户提供一种机制（例如，一个按钮），以手动触发打开 App Store。

若要更详细地了解独立应用，请查看以下链接：[独立应用介绍](https://developer.android.com/wear/preview/features/standalone-apps.html)

# 在 Wear 设备上保持应用活跃

当我们为不同的使用场景编写应用时，需要做出一些调整。我们知道，在不使用应用时，应该让应用在 Wear 设备上进入休眠状态，以获得更好的电池性能；但是，当我们为地图构建应用时，有必要让地图对用户可见且处于活跃状态。

Android 为此提供了一个简单的配置：一个激活环境模式的几行代码方法：

```java
//oncreate Method
setAmbientEnabled();

```

这将在地图上启动环境模式。当用户不再积极使用应用时，API 切换到非交互式和低色彩渲染的地图：

```java
@Override
public void onEnterAmbient(Bundle ambientDetails) {
    super.onEnterAmbient(ambientDetails);
    mMapFragment.onEnterAmbient(ambientDetails);
}

```

下面的代码在 WearMap 上退出了环境模式。当用户开始积极使用应用时，API 切换到地图的正常渲染：

```java
@Override
public void onEnterAmbient(Bundle ambientDetails) {
    super.onEnterAmbient(ambientDetails);
    mMapFragment.onEnterAmbient(ambientDetails);
}

```

# 为你的应用配置 WAKE_LOCK

当一些 Wear 应用始终可见时，它们非常有用。让应用始终可见会影响电池寿命，因此在你添加此功能到应用时，应仔细考虑这一影响。

在清单文件中添加 `WAKE_LOCK` 权限：

```java
<uses-permission android:name="android.permission.WAKE_LOCK" />

WAKE_LOCK mechanism:
```

```java
 // Schedule a new alarm
    if (isAmbient()) {
        // Calculate the next trigger time
        long delayMs = AMBIENT_INTERVAL_MS - (timeMs % 
        AMBIENT_INTERVAL_MS);
        long triggerTimeMs = timeMs + delayMs;

        mAmbientStateAlarmManager.setExact(
AlarmManager.RTC_WAKEUP,
            triggerTimeMs,
            mAmbientStatePendingIntent);

    } else {
        // Calculate the next trigger time for interactive mode
    }

```

用户可以使用语音输入，而不是使用输入法框架读取输入，这需要在你的 Wear 设备上保持网络活跃：

```java
private static final int SPEECH_REQUEST_CODE = 0;

// Create an intent that can start the Speech Recognizer activity
private void displaySpeechRecognizer() {
    Intent intent = new 
    Intent(RecognizerIntent.ACTION_RECOGNIZE_SPEECH);
    intent.putExtra(RecognizerIntent.EXTRA_LANGUAGE_MODEL,
            RecognizerIntent.LANGUAGE_MODEL_FREE_FORM);
// Start the activity, the intent will be populated with the speech text
    startActivityForResult(intent, SPEECH_REQUEST_CODE);
}

// This callback is invoked when the Speech Recognizer returns.
// This is where you process the intent and extract the speech text from the intent.
@Override
protected void onActivityResult(int requestCode, int resultCode,
        Intent data) {
    if (requestCode == SPEECH_REQUEST_CODE && resultCode == RESULT_OK) 
    {
        List<String> results = data.getStringArrayListExtra(
                RecognizerIntent.EXTRA_RESULTS);
        String spokenText = results.get(0);
        // Do something with spokenText
    }
    super.onActivityResult(requestCode, resultCode, data);
}

```

# 了解完全交互模式和精简模式

Google Maps 安卓 API 可以作为精简模式地图提供静态图片。

为 Android 地图添加精简模式与配置正常地图类似，因为它将使用相同的类和接口。我们可以通过以下两种方式设置 Google 地图为精简模式：

+   作为 `MapView` 或 `MapFrgament` 的 XML 属性

+   使用 `GoogleMapOptions` 对象

```java
<fragment xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:map="http://schemas.android.com/apk/res-auto"
    android:name="com.google.android.gms.maps.MapFragment"
    android:id="@+id/map"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    map:cameraZoom="13"
    map:mapType="normal"
    map:liteMode="true"/>

```

或者，按照以下方式使用 `GoogleMapOptions` 对象：

```java
GoogleMapOptions options = new GoogleMapOptions().liteMode(true);

```

交互模式允许应用使用所有的生命周期方法，包括 `onCreate()`、`onDestroy()`、`onResume()` 和 `onPause()`，以及所有 Google API 功能，使应用完全交互。相应的代价是会有网络依赖。

有关交互模式和精简模式的更多信息，请查看以下链接：[交互与精简模式](https://developers.google.com/maps/documentation/android-api/lite)

# 概述

我们已经来到了章节的末尾，期待着对 WearMapDiary 应用进行改进。现在，我们了解了如何创建一个`MapsActivity`，设置地图和 Google API 密钥，配置 Wear 模拟器中的 Google Play 服务，运行时权限检查，检查 GPS 硬件，以及使用`geocoder`类获取位置名称。我们已经理解了地图的交互模式和 Lite 模式的概念。在下一章中，让我们进一步了解 Wear 和地图用户界面控件以及其他 Google 地图技术，例如街景，更改地图类型等等。
