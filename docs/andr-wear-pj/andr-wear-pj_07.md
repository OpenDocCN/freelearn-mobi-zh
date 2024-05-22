# 任意地点的导航方式 - 用户界面控制及更多功能

现在你已经学会了如何为 Android Wear 应用程序带来 Google Maps 的生命，并探索 SQLite 集成，我们需要用户界面控制和更多增强功能。在本章中，让我们专注于通过添加功能，如移动地图上的标记和更改地图类型，使地图应用程序更具功能性和直观性。你将在本章中学习以下主题：

+   标记控制

+   地图类型

+   地图缩放控制

+   在穿戴设备上的街景视图

+   最佳实践

标记不仅仅是地图上表示坐标的符号。通过将标记的默认符号替换为有意义的图像描绘，标记被用来传达这是什么类型的地点；例如，如果是加油站，标记符号可以类似于加油枪符号或医院。

# 更改标记颜色和自定义

使用`MarkerOptions`类，我们可以更改标记的颜色和图标。以下代码解释了更改标记的图标和颜色。

要更改标记的颜色，请参考以下代码：

```java
private void addMarker(Memory memory) {
    Marker marker = mMap.addMarker(new MarkerOptions()
          .draggable(true).icon(BitmapDescriptorFactory.defaultMarker
          (BitmapDescriptorFactory.HUE_CYAN)).alpha(0.7f)
            .position(new LatLng(memory.latitude, memory.longitude)));

    mMemories.put(marker.getId(), memory);
}

```

我们现在可以看到，标记的颜色已从红色变为带有透明度的青色。如果你希望移除透明度，可以移除传递给标记选项的`.alpha()`值：

![](img/00094.jpeg)

改变了标记的颜色

要将标记更改为`drawable`目录中的图标，请检查以下代码：

```java
private void addMarker(Memory memory) {
    Marker marker = mMap.addMarker(new MarkerOptions()
            .draggable(true).icon(BitmapDescriptorFactory.fromResource
            (R.drawable.ic_edit_location)).alpha(0.7f)
            .position(new LatLng(memory.latitude, memory.longitude)));

    mMemories.put(marker.getId(), memory);
}

```

这将替换默认的标记图标，使用我们从 drawable 目录中传递的自定义图像。我们需要确保图标大小不过大，它具有最佳的大小 72x72：

![](img/00095.jpeg)

使用自定义图像更改标记图像

之前的代码片段将帮助更改标记的颜色或图标，但对于更复杂的情况，我们可以动态构建标记视觉资产并将其添加到地图中。

使用简单的 Java 代码创建我们自己的自定义设计的标记怎么样？我们将创建一个在图像顶部绘制简单文本的标记。以下代码解释了如何使用`Bitmap`类和`Canvas`类在图像上绘制文本：

```java
 private void addMarker(Memory memory) {

        Bitmap.Config conf = Bitmap.Config.ARGB_8888;
        Bitmap bmp = Bitmap.createBitmap(80, 80, conf);
        Canvas canvas1 = new Canvas(bmp);

        // paint defines the text color, stroke width and size
        Paint color = new Paint();
        color.setTextSize(15);
        color.setColor(Color.BLACK);

        // modify canvas
        canvas1.drawBitmap(BitmapFactory.decodeResource(getResources(),
                R.drawable.ic_edit_location), 0,0, color);
        canvas1.drawText("Notes", 30, 35, color);

        // add marker to Map

        Marker marker = mMap.addMarker(new MarkerOptions()
                .draggable(true).icon(BitmapDescriptorFactory
                .fromBitmap(bmp)).alpha(0.7f)
                .position(new LatLng(memory.latitude, 
                memory.longitude)));

        mMemories.put(marker.getId(), memory);
    }

```

下面的截图显示了使用位图和画布绘制的带有注释的标记：

![](img/00096.jpeg)

动态添加来自 realmdb 信息的标记。

# 拖动标记并更新位置

在`MapActivity`中实现`GoogleMap.OnMarkerDragListener`接口，并实现`OnMarkerDragListener`接口中的所有回调方法：

```java
@Override
public void onMarkerDragStart(Marker marker) {

}

@Override
public void onMarkerDrag(Marker marker) {

}

@Override
public void onMarkerDragEnd(Marker marker) {

}

```

实现了接口中的这三个方法后，在第三个回调`onMarkerDragEnd`中，我们可以用更新后的位置详情更新内存。我们还可以在`onMapReady`回调中注册`draglistner`：

```java
@Override
public void onMapReady(GoogleMap googleMap) {

mMap.setOnMarkerDragListener(this);
...
}

```

然后，用以下代码更新`onMarkerDragEnd`方法：

```java
@Override
public void onMarkerDragEnd(Marker marker) {

    Memory memory = mMemories.get(marker.getId());
    updateMemoryPosition(memory, marker.getPosition());
    mDataSource.updateMemory(memory);

}

```

之前的代码片段在标记被拖动时更新位置。

# InfoWindow 点击事件

当用户点击`InfoWindow`时，它允许用户删除标记。为了监听`Infowindow`的点击事件，我们需要实现`GoogleMap.OnInfoWindowClickListener`及其回调方法`onInfoWindowClick(..)`。

在`onMapready`回调中注册`infoWindoClicklistner`，如下所示：

```java
mMap.setOnInfoWindowClickListener(this);

```

在回调方法内部，当用户点击时，让我们设计一个警告对话框。它应该允许用户删除标记：

```java
@Override
public void onInfoWindowClick(final Marker marker) {
    final Memory memory = mMemories.get(marker.getId());
    String[] actions  = {"Delete"};
    AlertDialog.Builder builder = new AlertDialog.Builder(this);
    builder.setTitle(memory.city+", "+memory.country)
            .setItems(actions, new DialogInterface.OnClickListener() {
                @Override
                public void onClick
                (DialogInterface dialog, int which) {
                    if (which == 0){
                        marker.remove();
                        mDataSource.deleteMemory(memory);
                    }
                }
            });

    builder.create().show();
}

```

![](img/00097.jpeg)

# 用户界面控件

对于可穿戴设备，用户界面控件（如缩放和位置控件）是禁用的。我们可以使用`UISettings`类启用它们。`UISettings`类扩展了 Google Map 用户界面对象的设置。要获取此接口，请调用`getUiSettings()`。

下面的布尔方法返回组件的启用或禁用状态：

+   `public boolean isCompassEnabled ()`：获取指南针是否启用/禁用

+   `public boolean isMyLocationButtonEnabled ()`：获取我的位置按钮是否启用/禁用

+   `public boolean isZoomControlsEnabled ()`：获取缩放控件是否启用

+   `public boolean isZoomGesturesEnabled ()`：获取是否启用缩放手势

+   `public boolean isTiltGesturesEnabled ()`：获取是否启用倾斜手势

+   `public boolean isRotateGesturesEnabled ()`：获取是否启用旋转手势

+   `public boolean isScrollGesturesEnabled ()`：获取是否启用/禁用滚动手势

    这些方法将返回组件的状态。

    为了在应用程序中启用这些组件，`getUiSettings()`将提供适当的设置方法，如下所示：

+   `public void setCompassEnabled (boolean enabled)`：启用或禁用指南针

+   `public void setIndoorLevelPickerEnabled (boolean enabled)`：设置当启用室内模式时，室内楼层选择器是否启用

+   `public void setMyLocationButtonEnabled (boolean enabled)`：启用或禁用我的位置按钮

+   `public void setRotateGesturesEnabled (boolean enabled)`：设置是否启用旋转手势的偏好设置

+   `public void setZoomControlsEnabled (boolean enabled)`：启用或禁用缩放控件

让我们通过`WearMapdiary`应用程序来看看这个功能。我们将为应用程序启用缩放控件。在`OnMapready`方法中，为`mMap`对象添加以下代码行：

```java
mMap.getUiSettings().setZoomControlsEnabled(true);

```

![](img/00098.jpeg)

同样，我们可以设置其他用户界面控件。可穿戴设备上的所有这些控件都有一定的限制，例如，`setIndoorLevelPickerEnabled`在可穿戴设备上不起作用。

# 地图类型

地图类型决定了地图的整体表现形式。例如，地图集通常包含关注显示边界的政治地图，而道路地图显示一个城市或地区的所有道路。Google Maps Android API 提供了四种类型的地图，以及一个不显示地图的选项。让我们更详细地看看这些选项：

+   **普通：** 典型的道路地图。显示道路、人类建造的一些特征和重要的自然特征，如河流。道路和特征标签也可见。

+   **混合型（Hybrid）**：卫星照片数据加上道路地图。道路和特征标签也可见。

+   **卫星：** 卫星照片数据。道路和特征标签不可见。

+   **地形：** 地形数据。地图包括颜色、等高线、标签和透视阴影。一些道路和标签也可见。

+   **无（None）**：无瓦片。地图将被渲染为没有加载瓦片的空网格。

让我们来看看`WearMapdiary`应用程序中的这个功能。我们将更改应用程序的地图类型。在 `OnMapready` 方法中，在 `mMap` 对象内添加以下代码行，将地图类型更改为混合型（Hybrid）：

```java
mMap.setMapType(GoogleMap.MAP_TYPE_HYBRID);

```

![](img/00099.jpeg)

地图是混合型，即卫星图像和标签。现在，若要将地图类型更改为地形，请在 `mMap` 中插入以下代码：

```java
object'mMap.setMapType(GoogleMap.MAP_TYPE_TERRAIN);

```

![](img/00100.jpeg)

地形类型地图的外观如前图所示。现在，若要将地图类型更改为 `NONE`，请在 `mMap` 对象中插入以下代码：

```java
mMap.setMapType(GoogleMap.MAP_TYPE_NONE);

```

![](img/00101.jpeg)

当您选择没有地图时，它看起来如前一个屏幕截图所示。现在，若要将地图类型更改为卫星，请在 `mMap` 对象中插入以下代码：

```java
mMap.setMapType(GoogleMap.MAP_TYPE_SATELLITE)

```

![](img/00102.jpeg)

# 穿戴应用中的 Streetview

Google Street View 提供了其覆盖区域内指定道路的全景 360 度视图。Streetview 是可视化用户目的地或任何地址的好方法。添加 Streetview 为应用程序增添了现实世界的视觉元素，并为用户提供有意义的上下文。用户可以与 Streetview 互动；用户将喜欢在 Streetview 中平移和扫描位置。

要创建街景地图，我们将创建一个新的片段或活动，并可以启动活动或附加片段。在这个例子中，让我们使用 `SupportStreetViewPanoramaFragment` 类创建一个新活动，并在 `onMapLongclick` 回调中启动活动：

```java
Lets create a new activity with the following layout and java code. 
//Java class
public class StreetView extends AppCompatActivity {

    private static final LatLng SYDNEY = new LatLng(-33.87365, 
    151.20689);

    @Override
    protected void onCreate(final Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_street_view);

        SupportStreetViewPanoramaFragment streetViewPanoramaFragment =
                (SupportStreetViewPanoramaFragment)
                        getSupportFragmentManager()
                        .findFragmentById(R.id.Streetviewpanorama);

        streetViewPanoramaFragment.getStreetViewPanoramaAsync(
                new OnStreetViewPanoramaReadyCallback() {
                    @Override
                    public void onStreetViewPanoramaReady
                    (StreetViewPanorama panorama) {
                        // Only set the panorama to SYDNEY on startup 
                        (when no panoramas have been
                        // loaded which is when the savedInstanceState 
                        is null).
                        if (savedInstanceState == null) {
                            panorama.setPosition(SYDNEY);
                        }
                    }
                });
    }
}

```

在新的布局资源中添加以下代码，并将文件命名为 `activity_street_view`：

```java

<?xml version="1.0" encoding="utf-8"?>

<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
 android:layout_width="match_parent"
 android:layout_height="match_parent">

<fragment
 android:id="@+id/Streetviewpanorama"
 android:layout_width="match_parent"
 android:layout_height="match_parent"
 class="com.google.android.gms.maps.SupportStreetViewPanoramaFragment" />
</FrameLayout>

```

现在，在 `MapActivity` 中启动此活动 `onMapLongclicklistner`。在启动活动之前，请确保您已将应用程序或活动主题更改为 `Theme.AppCompat.Light.NoActionBar`：

```java
android:theme="@style/Theme.AppCompat.Light.NoActionBar"
@Override
    public void onMapLongClick(LatLng latLng) {
        // Display the dismiss overlay with a button to exit this 
        activity.

        //        mDismissOverlay.show();

        Intent street = new Intent(MapsActivity.this, 
        StreetView.class);
        startActivity(street);

```

![](img/00103.jpeg)

现在，我们拥有了一个完整、可运行的基本 Streetview 穿戴应用，你可以平移和旋转 360 度。

+   **多段线（Polylines）**：多段线扩展到对象类。多段线是一系列点的列表，在这些连续点之间绘制线段。多段线具有以下属性：

    +   **点：** 线的顶点。在连续点之间绘制线段。多段线需要起点和终点来绘制线条。

+   +   **宽度：** 线段宽度以屏幕像素为单位。宽度是常数，与相机缩放无关。

    +   **颜色：** 线段颜色采用 ARGB 格式；与 Color 使用的格式相同。

    +   **起始/结束端帽：** 定义了在折线开始或结束时使用的形状。支持的端帽类型：**ButtCap, SquareCap, RoundCap**（适用于实线描边模式）和 **CustomCap**（适用于任何描边模式）。默认情况下，起始和结束都是 **ButtCap**。

    +   **连接类型：** 连接类型定义了在折线除起始和结束顶点外的所有顶点处连接相邻线段时要使用的形状。

    +   **描边模式：** 沿着线条重复的实线或模式项序列。选择包括以下内容：

        +   间隙

        +   虚线

        +   点

    +   **Z-Index：** 此图块覆盖与其他覆盖层相比的绘制顺序。

    +   **可见性：** 指示线段的可见性，或者告诉线段是否被绘制。

    +   **测地线状态：** 指示折线的段是否应被绘制为测地线，而不是在墨卡托投影上的直线。测地线是地球上两点之间的最短路径。构建测地线曲线时假定地球是一个球体。

    +   **可点击性：** 当你希望用户点击折线时触发一个事件。它与通过 `setOnPolylineClickListener(GoogleMap.OnPolylineClickListener)` 注册的 `GoogleMap.OnPolylineClickListener` 一起工作。

    +   **标签：** 与折线关联的对象。例如，该对象可以包含关于折线表示内容的数据。这比存储一个分离的 `Map<Polyline, Object>` 要简单。另一个例子，你可以关联一个与数据集中的 ID 对应的字符串 ID。

在 `onMapready` 回调中添加以下代码，并将其附加到地图实例上：

```java
Polyline line = mMap.addPolyline(new PolylineOptions()
        .add(new LatLng(-34, 151), new LatLng(-37, 74.0))
        .width(5)
        .color(Color.WHITE));

```

![](img/00104.jpeg)

# 最佳实践

Android Wear 对于快速和一目了然的信息非常有用。在最新的 Google Play 服务中，穿戴设备上最被请求的功能是地图，Google Maps 的更新也来到了 Android Wear，这意味着你可以理想地开发地图应用程序，就像我们为移动应用程序开发一样，开发穿戴地图应用程序的过程没有变化。这意味着只需几行代码和配置就能获得最佳的类开发体验。

让我们讨论一下 Android Wear 地图应用程序的一些常见用例以及如何实现最佳的地图应用程序体验：

+   最常见的用例之一是简单地显示地图；由于穿戴设备显示较小，我们可能需要全屏显示整个地图。

    你的应用程序可能需要显示一个标记来指代地标。需要允许用户在地图上平移并找到地图上的地点。

+   安卓穿戴保留了从左向右滑动来关闭当前应用程序的手势。如果你不需要你的地图平移，这将继续有效。但是，如果你需要你的地图应用程序在地图上移动和平移，我们需要重写这个特定的关闭手势以减少混淆，并让用户退出应用程序。为此，我们可以实现`dismissoverlay`视图，并将其附加到长按事件上。该视图将处理关闭动作。

+   另一个常见的用例是在地图上选择位置，这样你可以与朋友分享位置。为了实现这一点，我们可以将标记放置在屏幕中央，并让用户在地图上平移并选择最近的平移`latlong`值，这表示在地图片段组件内选择的位置。然后，使用地图`oncamerachange`监听器来检测用户是否在地图上进行了平移。我们可以通过`cameraposition.target.letlong`值访问新位置。

+   释放我们不使用的组件是一个好习惯；例如，当我们初始化它时，释放 Google API 客户端。我们应在活动生命周期回调中释放它。

有关实现最佳穿戴地图应用程序的更多信息，请点击此链接：[`developers.google.com/maps/documentation/android-api/wear.`](https://developers.google.com/maps/documentation/android-api/wear)

# 总结

在本章中，你学习了如何添加 UI 控件，如缩放、地图类型等。

使用 Google Maps Android API，你了解了用户如何与以下关键项的穿戴地图应用程序互动：

**添加 UI 控件：** UI 控件帮助用户以更个性化的方式控制地图。

**拖动标记并更新位置标签：** 当用户想要修改地图上的标记放置时，拖动同一标记是一个很好的方法。

**自定义标记：** 我们知道标记标识地图上的位置。自定义标记可以帮助用户了解位置类型。自定义标记传达更多关于位置的信息；例如，位置处的燃油图标表示该位置是加油站。

**不同的地图类型：** 不同的地图类型帮助用户以个性化的方式体验地图。

**信息窗口点击事件：** 信息窗口是一种特殊的覆盖层，用于在地图上的给定位置显示内容（通常是文本或图像）的弹出气球。`InfoWindow`点击事件有助于执行某些操作。对于 WearMapDiary 应用程序的范围，我们正在附加`dialogfragment`以更新片段区域中的文本。

****多段线：**** 多段线指定一系列坐标作为 LatLng 对象的数组。这表示地图上的一个图形路径。

**街景视图：** Google 街景提供了从其覆盖区域内的指定道路上的全景 360 度视图。

现在，除了`wearmapdiary`之外，我们还可以利用所有这些与地图相关的想法，打造出最能帮助用户的应用程序。
