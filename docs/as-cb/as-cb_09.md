# 第九章。改善性能

性能很重要，因为它会影响您的应用在 Google Play 商店上的评价。我们想要一个五星级的应用！在高端设备上，您的应用可能会顺利运行，没有任何问题，但在用户的低端设备上，情况可能会有所不同。它可能运行缓慢或者内存不足，导致应用崩溃。

![改善性能](img/B04299_09_01.jpg)

在本章中，您将学习以下配方：

+   内存分析器和性能工具

+   糟糕的应用程序-性能改进

+   过度绘制问题

# 介绍

我们如何检测我们的应用是否会有性能问题？Android 应用程序中常见的问题是什么？我们如何解决这些问题？

在性能方面，可能会出现一些问题，如下所示：

+   **内存泄漏**：尽管 Android 配备了自己的内存管理系统，但可能会发生内存泄漏。

+   **内存不足异常**：您的应用程序可能会很容易耗尽内存，导致应用程序崩溃。例如，在低端设备上处理大图像时会出现这种情况。

+   **过度绘制**：过度绘制是指视图上的像素被绘制多次的现象。它可能导致用户界面无响应或延迟。

在接下来的示例中，我们将检查这里列出的问题。Android SDK 和 Android Studio 都配备了一些很好的工具来检查您的应用。

# 内存分析器和性能工具

您的应用程序可能会受到内存泄漏或分配过多内存的影响。

**垃圾收集器**（**GC**）负责清理我们不再需要使用的任何东西，这是一个很好的帮手，但不幸的是，它并不完美。它只能删除被识别为不可达的对象。未清理的对象会一直占用空间。过一段时间，如果创建了越来越多的对象，就可能会发生`OutOfMemoryError`，就像尝试加载一些大图像时会发生的情况，这是许多 Android 应用程序常见的崩溃场景。

内存泄漏有些难以发现。幸运的是，Android Studio 配备了内存监视器。它可以为您提供应用程序内存使用情况的概述，并提供一些关于内存泄漏的线索。

我们将使用这个**内存监视器**来找出是否不需要的 GC 事件模式导致了性能问题。除此之外，我们将使用**分配跟踪器**来确定代码中可能存在的问题所在。

## 准备工作

对于这个配方，如果您已经完成了前几章中的任何一个配方，那将是很好的。如果可能的话，它应该是从互联网获取数据（文本和图像）的配方，例如第二章中的应用程序，*具有基于云的后端的应用程序*。当然，任何其他应用程序都可以，因为我们将检查工具来检查我们的应用程序以改进它。

## 如何做...

让我们看看我们的应用程序的性能如何！

1.  启动 Android Studio 并打开您选择的应用程序。

1.  在设备上运行您的应用程序（或使用虚拟 Genymotion 设备）。

1.  **内存监视器**位于**内存**选项卡上，您可以在**Android**选项卡上找到它。

1.  如果没有显示，请使用*Cmd* + *6*（对于 Windows：*Alt* + *6*）快捷键使其出现。

1.  运行您的应用程序，查看内存监视器记录您的应用程序的内存使用情况。在下面的示例中，我运行了一个从 FourSquare API 加载了 200 个场馆（包含文本和图片）的应用程序。每次我按下按钮时，我会请求 200 个更多的场馆，导致图表中显示的峰值。请给我更多附近的咖啡店：![如何做...](img/B04299_09_02.jpg)

1.  应用的内存使用显示为深蓝色。未分配的内存显示为浅蓝色。当您的应用开始运行时，分配的内存会增长，直到没有更多的内存，或者当 GC 到达并完成其工作时，它会减少。

1.  这些都是常见的事件，最终，您可以通过单击左侧窗口上方的“内存”选项卡上的“启动 GC”图标（**Initiate GC**）来自己调用 GC。

1.  只有在短时间内分配了大量内存或 GC 事件更频繁时才会引起怀疑。您的应用程序可能存在内存泄漏。

1.  同样，您可以监视 CPU 使用情况。您可以在**Android**面板的**CPU**选项卡上找到它。如果您在这里注意到非常高的峰值，那么您的应用程序可能做得太多了。在下面的截图中，一切看起来都很好：![如何操作...](img/B04299_09_03.jpg)

1.  要了解更多关于内存问题的信息，我们可以使用另一个工具。从“工具”菜单中，选择“Android”和“Android 设备监视器”选项。该工具带有堆视图、内存监视器和分配跟踪器，这些都是提供有关应用程序使用的内存的见解的工具。

1.  如果尚未选择，请单击顶部导航栏上出现的“Dalvik 调试监视器服务器”（**DDMS**）按钮。`DDMS`是一个提供线程和堆信息以及其他一些内容的调试工具。

1.  选择“堆”选项卡。在窗口的右侧，选择应用程序，应该会出现在设备名称的下方。如果找不到您的应用程序，可能需要重新运行您的应用程序。![如何操作...](img/B04299_09_04.jpg)

1.  内存请求将通过从内存池中分配部分来处理，这称为堆。在任何给定时间，堆的某些部分正在使用，而某些部分未使用，因此可供将来分配使用。

1.  **堆**选项卡可以帮助您诊断内存泄漏，显示系统为您的应用程序分配了多少内存。在这里，您可以确定意外或不必要地分配的对象类型。如果分配的内存不断增加，那么这是您的应用程序存在内存泄漏的强烈迹象。

### 注意

如果未启用堆更新，请查看“设备”选项卡上的按钮。单击“更新堆”按钮（截图左侧第二个按钮）。

1.  堆输出仅在 GC 事件之后显示。在堆选项卡上，找到“Cause GC”按钮并单击它以强制 GC 执行其工作。之后，“堆”选项卡将看起来有点像这样：![如何操作...](img/B04299_09_05.jpg)

1.  在上面的截图中显示了关于应用程序堆使用情况的大量信息。单击表中的任何项目以获取更多信息。这里显示的信息可以帮助您确定应用程序的哪些部分导致了太多的分配。也许，您需要减少分配的数量或更早释放内存。

1.  为了更好地了解您的应用程序的关键部分以及确切导致问题的堆栈跟踪，您可以单击“分配跟踪器”选项卡。

1.  在该选项卡上，单击“开始跟踪”按钮。

1.  以某种方式与您的应用程序进行交互，例如刷新列表，转到详细视图或您的应用程序所做的任何操作，并且您想要测量。

1.  单击“获取分配”按钮以更新分配列表。

1.  作为您为应用程序启动的操作的结果，您将在此处看到所有最近的分配。

1.  要查看堆栈跟踪，请单击任何分配。在下一个示例中，我们正在调查在表行中加载图像。跟踪显示了在哪个线程中分配了什么类型的对象以及在哪里。![如何操作...](img/B04299_09_06.jpg)

如果愿意，您可以玩一下，以更多了解 Android 设备监视器。既然您已经看到了一些测量结果的工具，让我们更仔细地看看如何处理它们以及如何避免内存问题。我们下一节再见！

## 还有更多...

**Android 设备监视器**和 Android Studio 附带的内存工具都有许多其他选项可供您探索。这些选项将帮助您提高应用程序的质量和性能。这将使您和您的应用程序用户感到满意！

## 另请参阅

+   第二章, *具有基于云的后端的应用程序*

+   第八章, *提高质量*

+   第十章, *测试您的应用程序*

# 这里是一个糟糕的应用程序 - 性能改进

在 Android 应用程序开发中，有哪些应该做和不应该做的事情，以避免性能问题，即使这些问题可能不会在您自己的设备上出现？测试 Android 应用程序很困难，因为有这么多的设备。谨慎写代码总比抱憾写代码要好。

有人说编写高效代码有两个基本规则：不要做不需要做的工作（因此来自第八章的 DRY 和 YAGNI 原则，*提高质量*），如果可以避免的话，不要分配内存。除此之外，还有一点很有趣，那就是有各种可用的库，它们不仅可以节省您的时间，而且还可以证明非常高效。当然，重新发明轮子也可能出现错误。

例如，考虑`RetroFit`库，它将使编写用于消耗网络服务的代码变得更加容易，或者考虑`Picasso`，这是一个图像加载库，它将通过一行代码从 URL 加载图像，而无需过多担心诸如线程、图像大小调整、转换或内存管理等问题。

总的来说，一些良好的做法如下：

+   优化位图内存使用。

+   在隐藏用户界面时释放内存。

+   不要在布局中使用太多嵌套视图。

+   不要创建不必要的对象、类或内部类。

+   在可能的情况下，使用原始类型而不是对象。

+   如果您不需要对象的任何成员，最好使用静态方法而不是虚拟方法。静态调用会更快。

+   尽量避免使用内部的 getter 和 setter，因为在 Android 中直接访问字段要快得多。

+   如果整数可以胜任，就不要使用浮点数。

+   如果注册了监听器，那么一定要确保取消注册。在活动生命周期的相应对中注册和取消注册。例如，在`onCreate`方法中注册，然后在`onDestroy`方法中取消注册。或者，在`onResume`方法中注册，然后在`onPause`方法中取消注册。

+   如果某个操作花费的时间超过几秒钟，请向用户提供反馈。让用户知道您的应用程序并没有死机，而是在忙着处理！通过显示进度指示器来显示正在进行的操作。

+   始终进行测量。使用性能工具来了解您的应用程序的表现如何。

### 提示

**Android Studio 提示**

您在寻找什么吗？按两次*Shift*键，然后开始输入您要搜索的内容。或者，要显示所有最近的文件，请使用*Cmd* + *E*（对于 Windows：*Ctrl* + *E*）快捷键。

## 准备就绪

对于这个示例，您只需要安装并运行 Android Studio，最好还有一个具有互联网访问权限的真实设备。

## 如何做...

让我们创建一个真正糟糕的应用程序，这样我们就有东西可以修复。我们不会优化位图内存使用。我们会大量使用嵌套视图，做一些其他非常糟糕的事情，对于这个示例，我们将显示有史以来最糟糕的电影列表。这就是糟糕的应用程序：

1.  在 Android Studio 中创建一个新项目。

1.  将其命名为`BadApp`，然后单击**下一步**按钮。

1.  勾选**手机和平板电脑**选项，然后单击**下一步**按钮。

1.  选择**空白活动**，然后单击**下一步**按钮。

1.  接受名称并单击**完成**按钮。

1.  打开`activity_main.xml`布局，并用一个具有漂亮背景颜色的列表视图替换内容，该列表视图位于具有另一个漂亮背景颜色的相对布局中。我们这样做是因为我们想在下一个示例中演示概述问题：

```kt
<RelativeLayout xmlns:android=  
  "http://schemas.android.com/apk/res/android"

    android:layout_width="match_parent"
    android:layout_height="match_parent" 
    android:paddingLeft="@dimen/activity_horizontal_margin"
android:paddingRight="@dimen/activity_horizontal_margin"
    android:paddingTop="@dimen/activity_vertical_margin"
    android:background="@android:color/holo_orange_dark"
    android:paddingBottom="@dimen/activity_vertical_margin" 
    tools:context=".MainActivity">
   <ListView
       android:id="@+id/main_list"
       android:background="@android:color/holo_blue_bright"
       android:layout_width="match_parent"
       android:layout_height="match_parent"></ListView>
</RelativeLayout>
```

1.  创建一个新的布局文件，命名为`adapter.xml`。让我们有一些嵌套视图和许多背景颜色。都是为了糟糕的应用程序。

```kt
<?xml version="1.0" encoding="utf-8"?>
<FrameLayout xmlns:android= 
 "http://schemas.android.com/apk/res/android"
    android:orientation="vertical"    
    android:layout_width="match_parent"
    android:background="@android:color/holo_green_light"
    android:padding="8dp"android:layout_height="match_parent">
    <ImageView
        android:id="@+id/main_image"
        android:src="img/ic_media_play"android:layout_marginTop="8dp"android:layout_width="80dp"android:scaleType="fitCenter"android:layout_height="60dp" />
    <TableLayout
        android:layout_marginTop="8dp"android:layout_marginLeft="90dp"android:layout_width="match_parent"android:layout_height="wrap_content"><TableRow android:background=  
          "@android:color/holo_purple">
            <TextView android:layout_width="match_parent"
                android:id="@+id/main_text_title"
                android:layout_marginTop="8dp"
                android:textSize="24sp"
                android:layout_height="wrap_content"
                android:textColor="@android:color/white"/>
    </TableRow>
        <TableRow android:background=
           "@android:color/holo_blue_light">
             <TextView android:layout_width="match_parent"android:id="@+id/main_text_year"android:layout_height="wrap_content"android:textSize="20sp"android:layout_marginTop="8dp"android:textColor="@android:color/white"/></TableRow>
        <TableRow android:background= 
           "@android:color/holo_green_dark">
           <LinearLayout
               android:orientation="vertical"android:layout_height="wrap_content"android:layout_width="match_parent"android:layout_marginTop="16dp">
               <TextView android:layout_width="match_parent"android:id="@+id/main_text_genre"android:layout_height="wrap_content"android:textSize="16sp"android:layout_marginTop="8dp"android:background=   "@android:color/holo_green_dark"android:textColor="@android:color/white"/>
                <TextView android:layout_width="match_parent"android:id="@+id/main_text_director"android:layout_height="wrap_content"android:textSize="16sp"android:layout_marginTop="8dp"android:background=
                    "@android:color/holo_green_light"android:textColor="@android:color/white"/>
               <TextView android:layout_width="match_parent"android:id="@+id/main_text_actors"android:layout_height="wrap_content"android:textSize="16sp"android:layout_marginTop="8dp"android:background=  "@android:color/holo_green_dark"android:textColor="@android:color/white"/></LinearLayout>
        </TableRow>
    </TableLayout>
</FrameLayout>
```

1.  打开`AndroidManifest.xml`文件，并添加对互联网访问的权限：

```kt
<uses-permission android:name="android.permission.INTERNET" />
```

1.  创建一个新类，命名为`BadMovie`：

```kt
public class BadMovie {
    public String title;
    public String genre;
    public String year;
    public String director;
    public String actors;
    public String imageUrl;
    public BadMovie(String title, String genre, String 
     year, String director, String actors, String 
      imageUrl){
        this.title = title;
        this.genre = genre;
        this.year =year;
        this.director = director;
        this.actors = actors;
        this.imageUrl = imageUrl;
    }
}
```

1.  创建一个适配器类，命名为`MainAdapter`。我们将使用`ViewHolder`类，并创建一个单独的线程从网络加载每个电影图像：

```kt
public class MainAdapter  extends ArrayAdapter<BadMovie> {
    private Context mContext;
    private int mAdapterResourceId;
    public List<BadMovie> Items = null;
    static class ViewHolder
        TextView title;
        TextView genre;
        ImageView image;
        TextView actors;
        TextView director;
        TextView year;
    }
    @Override
    public int getCount() {
        super.getCount();
        int count = Items != null ? Items.size() : 0;
        return count;
    }
    public MainAdapter(Context context, int adapterResourceId, 
     List<BadMovie> items) {
        super(context, adapterResourceId, items);
        this.Items = items;
        this.mContext = context;
        this.mAdapterResourceId = adapterResourceId;
    }
    @Override
	public View getView(int position, View convertView, 
     ViewGroup parent) {
        View v = null;
        v = convertView;
        if (v == null) {
            LayoutInflater vi = (LayoutInflater)    
            this.getContext().getSystemService(
             Context.LAYOUT_INFLATER_SERVICE);
            v = vi.inflate(mAdapterResourceId, null);
            ViewHolder holder = new ViewHolder();
            holder.title = (TextView) v.findViewById(
             R.id.main_text_title);
            holder.actors = (TextView) v.findViewById(
             R.id.main_text_actors);
            holder.image = (ImageView)       
             v.findViewById(R.id.main_image);
            holder.genre = (TextView)   
             v.findViewById(R.id.main_text_genre);
            holder.director = (TextView) 
             v.findViewById(R.id.main_text_director);
            holder.year = (TextView) 
             v.findViewById(R.id.main_text_year);
            v.setTag(holder);
        }

        final BadMovie item = Items.get(position); 
        if (item != null) {final ViewHolder holder = (ViewHolder) v.getTag();
           holder.director.setText(item.director);
           holder.actors.setText(item.actors);
           holder.genre.setText(item.genre);
           holder.year.setText(item.year);
           holder.title.setText(item.title);
           new Thread(new Runnable() {
            public void run(){
             try {
              final Bitmap bitmap = 
               BitmapFactory.decodeStream((
                InputStream) new  
               URL(item.imageUrl).getContent());
              ((Activity)getContext()).runOnUiThread(new  
              Runnable() {
                  @Override
                  public void run() {                    
                     holder.image.setImageBitmap(bitmap);
                   }
                });
             } 
             catch (Exception e) {
               e.printStackTrace();
             }
            }
          }).start();}
        return v;
    }
}
```

1.  在`MainActivity`文件中，添加一个包含所有电影的私有成员：

```kt
private ArrayList<BadMovie> mBadMovies;
```

1.  在`onCreate`方法中添加实现，以添加几千部糟糕的电影，为它们创建一个适配器，并告诉列表视图相关信息：

```kt
mBadMovies = new ArrayList<BadMovie>();
for (int iRepeat=0;iRepeat<=20000;iRepeat++) {
    mBadMovies.add(new BadMovie("Popstar", "Comedy", "2000", "Paulo Segio de Almeida", "Xuxa Meneghel,Luighi Baricelli", "https://coversblog.files.wordpress.com/2009/03/xuxa-popstar.jpg"));
    mBadMovies.add(new BadMovie("Bimbos in Time", "Comedy","1993", "Todd Sheets", "Jenny Admire, Deric Bernier","http://i.ytimg.com/vi/bCHdQ1MB1D4/maxresdefault.jpg"));
    mBadMovies.add(new BadMovie("Chocolat", "Comedy", "2013", "Unknown", "Blue Cheng-Lung Lan, MasamiNagasawa", "http://i.ytimg.com/vi/EPlbiYD1MmM/maxresdefault.jpg"));
    mBadMovies.add(new BadMovie("La boda o la vida", "1974", "year", "Rafael Romero Marchent", "Manola Codeso, La Polaca", "http://monedasycolecciones.com/10655-thickbox_default/la-boda-o-la-vida.jpg"));
    mBadMovies.add(new BadMovie("Spudnuts", "Comedy", "2005", "Eric Hurt", "Brian Ashworth, Dave Brown, Mendy St. Ours", "http://lipponhomes.com/wp-content/uploads/2014/03/DSCN0461.jpg"));}

//source: www.imdb.com
MainAdapter adapter = new MainAdapter(this, R.layout.adapter, mBadMovies);
((ListView)findViewById(R.id.main_list)).setAdapter(adapter);
```

1.  现在运行您的应用程序。根据**互联网电影数据库**（**IMDB**）的用户，这些是有史以来最糟糕的喜剧电影。我们故意多次添加了这些电影，以创建一个巨大的列表，其中每一行都使用了从互联网加载缩略图的原始方法，如下图所示：![操作步骤...](img/B04299_09_07.jpg)

1.  根据您测试应用程序的设备，您可能需要滚动一段时间，或者错误可能会立即出现。

1.  这是迟早会出现在**LogCat**中的。在应用程序崩溃后，检查日志。使用*Cmd* + *6*快捷键（对于 Windows：*Alt* + *6*）显示**LogCat**。它会显示类似于这样的内容：

```kt
packt.com.thebad E/AndroidRuntime﹕ FATAL EXCEPTION: Thread-3529
java.lang.OutOfMemoryError: Failed to allocate a 7238412 byte allocation with 53228 free bytes and 51KB until OOM
```

1.  这就是发生的地方：

```kt
At packt.com.thebad.MainAdapter$1.run(MainAdapter.java:82)
```

1.  还要查看内存和 CPU 监视器。您的设备很难受。如果您滚动列表，就会出现这种情况。

以下屏幕截图提供了**内存**报告：

![操作步骤...](img/B04299_09_08.jpg)

以下屏幕截图提供了**CPU**报告：

![操作步骤...](img/B04299_09_09.jpg)

1.  如果您想多次加载全尺寸图像，就会得到这样的结果。由于我们无论如何都显示缩略图，因此没有必要这样做，而且您的设备无法处理。让我们来解决这个问题。

### 注意

我们还存在线程问题，因为错误的图像可能出现在行上。

1.  尽管最好的解决方案是让服务器返回缩略图而不是大图像，但我们并不总能控制这一点，特别是在处理第三方来源时。因此，解决内存问题的一种方法是在`MainAdapter`类中加载位图时为`BitmapFactory Options`设置`inSampleSize`属性，就像我们在前几章的示例中所做的那样。

1.  但是，在这里使用`Picasso`库将更加高效。`Picasso`是一个流行的图像库，将简化我们的流程。除其他功能外，它将在单独的线程中从互联网加载图像，并将其缩小到其容器的大小，这里是适配器布局中的图像视图。

1.  打开`app`文件夹中的`build.gradle`文件，并添加`Picasso`的依赖项：

```kt
dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
    compile 'com.squareup.picasso:picasso:2.3.3'
}
```

1.  保存文件并单击出现的**立即同步**链接。

1.  打开`MainAdapter`类，并用一行代码替换加载图像的线程（以及其中的任何内容）。使用*Alt* + *Enter*快捷键添加`Picasso`导入：

```kt
Picasso.with(getContext()).load(item.imageUrl).resize(80,
  60).into(holder.image);
```

1.  就是这样。`Picasso`将负责下载和调整图像的大小。

1.  现在再次运行应用程序，并随意滚动列表。内存和线程问题都已解决。列表视图可以平滑滚动。

1.  如果查看**Android**面板的**内存**和**CPU**选项卡，您将了解到这样做的区别。

以下屏幕截图提供了**内存**报告：

![操作步骤...](img/B04299_09_10.jpg)

以下屏幕截图提供了**CPU**报告：

![操作步骤...](img/B04299_09_11.jpg)

我们刚刚修复了我们的应用程序，现在能够显示一大堆糟糕的电影。在下一个教程中，我们将检查应用程序是否存在过度绘制问题。在旧的或性能较差的设备上，可能会出现这些问题。

## 还有更多...

`Picasso`还有一些其他有趣的功能，比如创建圆形图像，旋转图像，或者自动显示错误或占位图像。

`Picasso`的替代方案是`Universal Image Loader`库。

`RetroFit`是一个强烈推荐的用于 API 通信的库。它是 Android 和 Java 的 REST 客户端，可以节省大量时间和头疼。

### 注意

**Android Studio 提示**

想要重构你的代码吗？使用快捷键*Ctrl* + *T*（对于 Windows：*Ctrl* + *Alt* + *Shift* + *T*）来查看你有哪些选项。例如，你可以重命名一个类或方法，或者从一个方法中提取代码。

# 过度绘制问题

你的应用程序的界面需要快速渲染，例如，滚动列表时的交互应该运行顺畅。特别是旧的或低端设备经常很难做到这些。无响应或缓慢的用户界面可能是结果，这通常是由所谓的过度绘制引起的。

过度绘制是指视图上的像素被绘制多次的现象。一个带有另一个背景颜色的视图的彩色背景就是过度绘制的一个例子（像素被绘制两次），但这并不是真正的问题。然而，过度绘制过多会影响应用程序的性能。

## 准备就绪

你需要有一个真实的设备，并且需要完成前一个教程中的`The Bad`应用程序，以演示过度绘制问题，但如果愿意，你也可以检查任何其他应用程序。

## 如何做...

你的设备包含一些有趣的开发者选项。其中之一是**调试 GPU 过度绘制**选项，可以通过以下步骤获得：

1.  在你的设备上，打开**设置**应用程序。

1.  选择**开发者选项**。

### 注意

如果你的设备上没有**开发者选项**项目，你需要先进入**关于设备**，然后点击**版本号**七次。完成后，返回。现在列表中会出现一个名为**开发者选项**的新选项。

1.  找到**调试 GPU 过度绘制**选项并点击它：![如何做...](img/B04299_09_12.jpg)

1.  在弹出的对话框中，选择**显示过度绘制**区域。

1.  现在，你的设备看起来有点像没有相应眼镜的 3D 电影，但实际上显示的是：颜色表示过度绘制的数量，没有颜色表示没有过度绘制（像素只被绘制一次），蓝色表示过度绘制 1 次，绿色表示过度绘制 2 次，浅红色表示过度绘制 3 次，深红色表示过度绘制 4 次甚至更多。

### 提示

最多过度绘制 2 次是可以接受的，所以让我们集中在红色部分。

1.  运行你想要检查的应用程序。在这个教程中，我选择了前一个教程中的`The Bad`应用程序进行检查，如下所示：![如何做...](img/B04299_09_13.jpg)

1.  是的，情况非常糟糕。每个视图都有自己的背景颜色，导致过度绘制。

1.  Android 足够智能，可以减少一些过度绘制的情况，但对于复杂的应用程序，你需要自己解决。当你查看前一个教程中的活动和适配器的布局时，这并不难。

1.  首先，打开`activity_main.xml`布局文件。删除列表视图中的`background`属性，因为它根本没有被使用。同时，也从`RelativeLayout`文件中删除背景属性，因为我不喜欢橙色，至少不适合应用程序。

1.  从`main_text_genre`、`main_text_director`和`main_text_actors`文本视图中删除`background`属性。同时，从它们的父视图中删除`background`属性，即出现在`TableLayout`中的最后一个`TableRow`。

1.  如果重新运行应用程序，应用程序不仅会布局得更好一些，而且你还会注意到过度绘制的迹象减少了。

1.  让我们检查一下是否可以进一步改进。将根部的`FrameLayout`更改为`RelativeLayout`。摆脱`TableLayout`并相对定位文本视图：

```kt
<?xml version="1.0" encoding="utf-8"?><RelativeLayout xmlns:android=
  "http://schemas.android.com/apk/res/android"
    android:orientation="vertical"    
    android:layout_width="match_parent"
    android:background="@android:color/holo_green_light"
    android:padding="8dp"
    android:layout_height="match_parent">
    <ImageView
        android:id="@+id/main_image"android:src="img/ic_media_play"android:layout_marginTop="8dp"android:layout_width="80dp"android:scaleType="fitCenter"android:layout_height="60dp" />
    <TextView android:layout_width="match_parent"android:id="@+id/main_text_title"android:layout_marginTop="8dp"android:layout_toRightOf="@+id/main_image"android:background="@android:color/holo_purple"android:textSize="24sp"android:layout_height="wrap_content"android:textColor="@android:color/white"android:text="Line 1"/>
    <TextView android:layout_width="match_parent"android:id="@+id/main_text_year"android:layout_height="wrap_content"android:layout_toRightOf="@+id/main_image"android:layout_below="@+id/main_text_title"android:background=
         "@android:color/holo_blue_light"android:textSize="20sp"android:layout_marginTop="8dp"android:textColor="@android:color/white"android:text="Line 2"/>
    <TextView android:layout_width="match_parent"android:id="@+id/main_text_genre"android:layout_height="wrap_content"android:layout_toRightOf="@+id/main_image"android:layout_below="@+id/main_text_year"android:textSize="16sp"android:layout_marginTop="8dp"android:textColor="@android:color/white"android:text="Sub  1"/>
    <TextView android:layout_width="match_parent"android:id="@+id/main_text_director"android:layout_height="wrap_content"android:layout_toRightOf="@+id/main_image"android:layout_below="@+id/main_text_genre"android:textSize="16sp"android:layout_marginTop="8dp"android:textColor="@android:color/white"android:text="Sub 2"/>
    <TextView android:layout_width="match_parent"android:id="@+id/main_text_actors"android:layout_height="wrap_content"android:layout_toRightOf="@+id/main_image"android:layout_below="@+id/main_text_director"android:textSize="16sp"android:layout_marginTop="8dp"android:textColor="@android:color/white"android:text="Sub 3"/>
</RelativeLayout>
```

1.  再次运行您的应用程序。它变得越来越好了，不是吗？

1.  要进一步改进您的应用程序，请删除所有`text`属性。它们只是用来检查我们是否在使用`layout_toRightOf`和`layout_below`属性时做对了。

在这个示例中，我们通过优化布局进一步改进了我们的糟糕应用程序。而且，它不再难看。实际上，它变得相当不错。

### 使用哪种布局类型？

使用`RelativeLayout`比`LinearLayout`更有效，但不幸的是，如果，例如，您想要移动或删除另一个视图引用的文本视图，则对开发人员不太友好。

`FrameLayout`要简单得多，但它没有这个问题，而且似乎表现和`RelativeLayout`一样好。

另一方面，它并不打算包含许多子部件。请注意，最终重要的是最小数量的嵌套布局视图，因此您应该选择适合您的需求并且性能最佳的容器。

太棒了！我们的应用程序在所有设备上都运行流畅。我们不再期望出现任何奇怪的错误。

现在让我们将其发送给我们的 Beta 用户，看看他们对此的看法。一旦我们完成最后一章，我们将讨论*临时分发*，我们就会知道了。

## 还有更多...

还有更多有趣的工具，也许您想检查以改进应用程序的质量和性能。

我们之前提到过`Espresso`。`Robotium`是另一个用于 UI 测试的 Android 测试自动化框架。您可以在[`robotium.com`](http://robotium.com)找到它。

## 另请参阅

+   第八章, *提高质量*

+   第十章, *测试您的应用程序的 Beta 版*
