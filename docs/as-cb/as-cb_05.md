# 第五章。大小确实重要

本章是关于构建可以在各种设备上运行的应用程序：手机、平板、平板手机和电视。我们将连接到 YouTube 获取一些数据和视频来显示。

大小和上下文确实很重要。当然，我们可以将所有内容都放大，但这并不能真正使应用程序变得更好。平板提供的空间比手机更多，而在用户交互方面，电视与智能手机有所不同。我们如何使布局在每台设备上都能按比例缩放并看起来流畅？我们如何为每种类型的设备找到正确的方法？

在本章中，您将学习以下内容：

+   大小和上下文

+   手机、平板和平板手机布局

+   媒体播放

+   电视和媒体中心

# 大小和上下文

手机、平板、平板手机和电视等设备都配备了各种大小和形状的屏幕。我们如何从更大的屏幕中受益，或者如何为较小的屏幕提供智能导航，并在一个应用程序中保持相同的功能和代码？这就是这个第一个配方要解决的问题。

各种设备怎么样？随着可穿戴设备的推出，我们发现这些类型设备的用户行为有很大不同。同样的情况也适用于电视。一如既往，让我们先做第一步。让我们检查一个可以在手机和平板上运行的应用程序。

# 手机、平板和平板手机

手机上一个众所周知的模式是列表或回收视图，当您点击任何行时，它会显示一些详细信息。在小屏幕上，应用程序会将您导航到不同的视图。这种模式之所以存在，是因为手机屏幕上的空间不足。如果您在具有足够空间的设备上运行相同的应用程序，我们可以在屏幕的左侧显示列表，右侧显示详细信息。

多个布局是我们需要的，再加上多个片段。如果我们这样做，我们可以减少需要编写的代码量。我们不想重复自己，对吧？

片段是 Android 开发中功能强大但也经常被误解的组件。片段是（小）功能块，大多数时间都有自己的布局。使用片段容器，片段可以存在于多个位置和多个与活动相关的布局中。这就是我们如何重用功能和布局的方法。

不过，片段应该谨慎使用。如果没有合适的策略，使用片段的应用程序可能会给您带来很多麻烦。片段中的代码经常会引用活动。虽然这些代码可能仍在运行，但片段可能在中间被从活动中分离出来（例如，因为用户按下了返回按钮）。这可能导致您的应用程序崩溃。

## 准备工作

要完成此配方，您需要安装并运行 Android Studio，并且需要一部手机、平板和/或平板手机设备（一如既往，建议使用实体设备；但是您也可以使用 Genymotion 创建虚拟设备）。

由于我们将使用 YouTube Android API，您需要在设备上安装最新的 YouTube Android 应用程序。检查您的设备上是否有该应用程序，如果没有或者有更新的话，可以使用 Google Play 应用程序进行安装或更新。

最后，您需要一个开发者账户。如果您还没有一个，您需要首先从[`developer.android.com/distribute/googleplay/start.html`](http://developer.android.com/distribute/googleplay/start.html)创建一个。

除了购买本书之外，为自己购买一个开发者账户是一个非常好的投资，我强烈建议您这样做。无论如何，您都需要一个才能将您的应用程序提交到 Google Play 商店中！

## 如何做...

让我们看看如何创建我们自己的可穿戴应用程序并在设备上运行：

1.  开始一个新的 Android Studio 项目。将您的应用程序命名为`YouTubeMediaApp`，并在**公司域**字段中输入`packt.com`。然后点击**下一步**按钮。

1.  在接下来的对话框中，只选中**手机和平板电脑**选项，然后单击**下一步**按钮。

1.  在下一个对话框中，选择**空白活动**，然后单击**下一步**按钮。

1.  在**自定义活动**对话框中，单击**完成**按钮。

1.  Android Studio 将为您创建新项目。在 Android Studio 左侧的**项目**视图中，找到`app`文件夹中的`build.gradle`并打开它。

1.  在`app`文件夹中的`build.gradle`文件中添加一个依赖项到`dependencies`部分，以使用 YouTube 服务 API。我们将使用此 API 在 YouTube 上搜索视频：

```kt
compile 'com.google.apis:google-api-services-youtube:v3-rev120-1.19.0'
```

1.  同步项目（单击**立即同步**链接或使用工具栏中的**同步项目与 Gradle 文件**按钮）。

1.  打开`activity_main.xml`布局。创建一个框架布局，它将作为我们稍后要在此处显示的片段的容器。出于演示目的，我们将为其选择一个漂亮的背景颜色。让我们选择橙色：

```kt
<?xml version="1.0" encoding="utf-8"?>
<FrameLayout xmlns:android=
  "http://schemas.android.com/apk/res/android"
   android:layout_width="match_parent"
   android:layout_height="match_parent"
   android:background="@android:color/holo_orange_light"
   android:id="@+id/main_container_for_list_fragment">
</FrameLayout>
```

1.  添加一个新布局并命名为`fragment_list.xml`。在容器内创建一个列表视图。此列表将包含我们在 YouTube 上找到的视频的标题和其他信息：

```kt
<?xml version="1.0" encoding="utf-8"?>
<FrameLayout 
    android:orientation="vertical"     
    android:layout_width="match_parent"
    android:layout_height="match_parent">
<ListView
    android:id="@+id/main_video_list_view"
	android:visibility="visible"
	android:padding="6dp"
	android:layout_marginTop="0dp"
	android:layout_width="match_parent"
	android:layout_height="match_parent">
	</ListView>
</FrameLayout>
```

1.  添加一个新的 Java 类，命名为`ListFragment`，然后单击**确定**按钮继续。

1.  将新类设置为`Fragment`的子类，并重写`onCreate`方法。为列表视图创建一个私有成员，并按照以下代码在布局中添加对列表视图的引用：

```kt
public class ListFragment extends Fragment {
  private ListView mListView;
  @Override
  public View onCreateView(LayoutInflater inflater,    
   ViewGroup container, Bundle savedInstanceState) 
    final View view= inflater.inflate(  
      R.layout.fragment_list, container, false);
    mListView = (ListView)view.findViewById(
     R.id.main_video_list_view);
    return view;
  }
}
```

### 注意

除了`ListActivity`之外，还有一个`ListFragment`类，您可以从中继承。出于演示目的，我们将在这里从`Fragment`类继承并自行处理一些事情。

1.  在添加正确的导入语句（使用*Alt* + *Enter*快捷键或其他方式）时，您将能够选择要导入的包。您可以在`android.app.Fragment`和`android.support.v4.app.Fragment`包之间进行选择。后者仅用于向后兼容。由于我们将为我们的应用程序使用最新的 SDK，请在被询问时选择此导入语句：

```kt
import android.app.Fragment;
```

1.  为 YouTube 添加另一个私有成员和一个 YouTube 列表，并创建一个名为`loadVideos`的方法。首先，我们将初始化 YouTube 成员：

```kt
private YouTube mYoutube;
private YouTube.Search.List mYouTubeList;
private void loadVideos(String queryString){
 mYoutube = new YouTube.Builder(new NetHttpTransport(),
  new JacksonFactory(), new HttpRequestInitializer() {
   @Override
   public void initialize(HttpRequest hr) throws  
    IOException {}
 }).setApplicationName( 
  getString(R.string.app_name)).build();
}
```

1.  接下来，我们将告诉 YouTube 我们要寻找什么以及我们希望 API 返回什么信息。我们需要在`loadVideos`方法的末尾添加 try catch 结构，因为我们事先不知道是否能连接到 YouTube。将以下内容添加到`loadVideos`方法的末尾：

```kt
try{
 mYouTubeList = mYoutube.search().list("id,snippet");      
 mYouTubeList.setType("video");
 mYouTubeList.setFields( 
  "items(id/videoId,snippet/title,snippet/   
      description,snippet/thumbnails/default/url)");
}
catch (IOException e) {
  Log.d(this.getClass().toString(), "Could not 
    initialize: " + e);
}
```

1.  要使用 YouTube API，您必须首先注册您的应用程序。要这样做，请将浏览器导航到[`console.developers.google.com/project`](https://console.developers.google.com/project)。

1.  单击**创建**项目按钮。输入`YouTubeApp`作为项目名称，然后单击**创建**按钮。

1.  项目创建后，仪表板将显示在网页上。在左侧，展开**API 和身份验证**，然后单击**API**。

1.  在页面的右侧，单击 YouTube 数据 API。单击**启用 API**按钮。

1.  再次在左侧，单击 API 之后的**凭据**。在公共 API 访问下，单击**创建新密钥**按钮。

1.  在**创建新密钥**弹出对话框中，单击**Android 密钥**按钮。

1.  由于此应用仅用于演示目的，我们不需要查找所请求的**SHA1**值。只需单击**创建**按钮。

1.  现在，将为您创建一个 API 密钥。复制 API 密钥的值。

1.  在`AndroidManifest.xml`文件中，添加一个访问互联网的权限：

```kt
android:name="android.permission.INTERNET"/>
```

### 将其粘合在一起！

1.  现在回到`ListFragment`类，告诉 API 关于您的密钥，该密钥就在 YouTube 对象的`search`调用旁边：

```kt
mYouTubeList.setKey("Your API key goes here");
```

1.  创建一个新的`VideoItem`类，并添加成员以保存每个视频的请求信息。请注意，我们在这里使用 getter 和 setter：

```kt
private String title;
private String description;
private String thumbnailURL;
private String id;
public String getId() {
 return id;
}
public void setId(String id) {
 this.id = id;
}
public String getTitle() {
 return title;
}
public void setTitle(String title) {
 this.title = title;
}
public String getDescription() {
 return description;
}
public void setDescription(String description) {
 this.description = description;
}
public String getThumbnailURL() {
 return thumbnailURL;
}
public void setThumbnailURL(String thumbnail) {
 this.thumbnailURL = thumbnail;
}
```

1.  创建一个新布局并命名为`adapter_video.xml`。然后，添加文本视图以显示视频信息：

```kt
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout android:layout_width="match_parent"
  android:layout_height="wrap_content"
  android:orientation="vertical"
   xmlns:android= 
    "http://schemas.android.com/apk/res/android"
  android:padding="6dp">
<TextView
  android:id="@+id/adapter_video_id"android:textSize="14sp"android:textStyle="bold"android:layout_width="match_parent"android:layout_height="wrap_content" />
<TextView
  android:id="@+id/adapter_video_title"android:textSize="20sp"android:layout_marginTop="2dp"android:layout_width="match_parent"android:layout_height="wrap_content" /></LinearLayout>
```

1.  创建一个新的`VideoAdapter`类，并使其成为`ArrayAdapter`的子类，用于保存`VideoItem`类型的条目。一个视图持有者将帮助我们用列出的`VideoItem`对象的属性填充文本视图：

```kt
public class VideoAdapter extends ArrayAdapter<VideoItem> {
 private Context mContext;
 private int mAdapterResourceId;
 public ArrayList<VideoItem>mVideos = null;
 static class ViewHolder{
        TextView videoId;
        TextView videoTitle;
    }
@Override
 public int getCount(){
 super.getCount();
 int count = mVideos !=null ? mVideos.size() : 0;
 return count;
}
public VideoAdapter (Context context, int  
 adapterResourceId, ArrayList<VideoItem> items)
{
 super(context, adapterResourceId, items);
 this.mVideos = items;
 this.mContext = context;
 this.mAdapterResourceId = adapterResourceId; 
}
@Override
public View getView(int position, View convertView, ViewGroup parent)
{
 View v = convertView;
if (v == null){LayoutInflater vi =   
     (LayoutInflater)this.getContext().getSystemService(
      Context.LAYOUT_INFLATER_SERVICE);
    v = vi.inflate(mAdapterResourceId, null);
    ViewHolder holder = new ViewHolder();
    holder.videoId = (TextView)  
     v.findViewById(R.id.adapter_video_id);
    holder.videoTitle = (TextView) 
     v.findViewById(R.id.adapter_video_title);     
    v.setTag(holder);
 }
 final VideoItem item = mVideos.get(position);
 if(item != null){
  final ViewHolder holder = (ViewHolder)v.getTag();
  holder.videoId.setText(item.getId());
  holder.videoTitle.setText( item.getTitle());
 }
 return v;
}
```

1.  现在回到`ListFragment`类。在其中再添加两个私有成员，一个用于我们找到的视频列表，一个用于我们刚刚创建的适配器：

```kt
private List<VideoItem>mVideos;
private VideoAdapter mAdapter;
```

1.  在`ListFragment`类中添加一个`search`方法：

```kt
public List<VideoItem> search(String keywords){
 mYouTubeList.setQ(keywords);
try{
   SearchListResponse response = mYouTubeList.execute();
   List<SearchResult> results = response.getItems();
   List<VideoItem>  items = new ArrayList<VideoItem>();
    for(SearchResult result:results){

    VideoItem item = new VideoItem();
    item.setTitle(result.getSnippet().getTitle());
    item.setDescription(result.getSnippet().
     getDescription());

    item.setThumbnailURL(result.getSnippet().
     getThumbnails().getDefault().getUrl());
    item.setId(result.getId().getVideoId());
    items.add(item);
  }
  return items;
 }
catch(IOException e){
  Log.d("TEST", "Could not search: " + e);
 }
}
```

1.  在`loadVideos`方法的末尾，添加调用`search`方法和初始化适配器的实现：

```kt
mVideos =search(queryString§);
mAdapter = new VideoAdapter(getActivity(), R.layout.adapter_video, (ArrayList<VideoItem>) mVideos);
```

1.  告诉列表视图关于适配器，并调用适配器的`notifyDataSetChanged`方法，通知有新条目可供显示。为此，我们将使用一个在 UI 线程上运行的`Runnable`实例：

```kt
getActivity().runOnUiThread(new Runnable() {
public void run() {
   mListView.setAdapter(mAdapter);
   mAdapter.notifyDataSetChanged();
 }
});
```

1.  现在我们将异步加载视频信息，因为我们希望应用在从互联网获取数据时能够响应。创建一个新线程，并在`run`方法内调用`loadVideos`。假设我们想要查看*Android 开发*视频：

```kt
@Override
 public void onActivityCreated(Bundle bundle){
 super.onActivityCreated(bundle);
 new Thread(new Runnable() {
   public void run(){
      loadVideos("Android development");
   }
}).start();
}
```

1.  创建一个新的布局并命名为`fragment_details.xml`。在此片段中，我们将显示用户从列表中选择的视频的缩略图和描述。既然我们已经在这里，我们也可以添加一个播放按钮。我们将在下一个步骤中需要它：

```kt
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout  xmlns:android=  
 "http://schemas.android.com/apk/res/android"
android:orientation="vertical" android:layout_width="match_parent"android:layout_height="match_parent">
<Button
android:id="@+id/detail_button_play"android:text="@string/play"android:layout_width="match_parent"android:layout_height="wrap_content" />
<ImageView
android:id="@+id/detail_image"android:layout_width="match_parent"android:layout_height="wrap_content"android:src="img/gallery_thumb"/>
<TextView
android:layout_marginTop="16dp"android:id="@+id/detail_text"android:minHeight="200dp"
android:layout_width="match_parent"
android:layout_height="wrap_content" />
</LinearLayout>
```

1.  创建`DetailsFragment`类：

```kt
public class DetailsFragment  extends Fragment {
  @Override
  public View onCreateView(LayoutInflater inflater,
   ViewGroup container, Bundle savedInstanceState) {
    final View view= inflater.inflate(
      R.layout.fragment_details, container, false);
     return view;
  }
}
```

1.  在`DetailsFragment`类中添加`showDetails`私有方法。在此方法中，我们将设置描述的文本，并创建一个新的 runnable 实例来加载视频的缩略图。还添加`setVideo`方法并重写`onResume`方法：

```kt
private void showDetails(){
if (getView()!=null &&mVideo != null)
 {
   TextView tv = (TextView) 
    getView().findViewById(R.id.detail_text);
   final ImageView iv = (ImageView)    
    getView().findViewById(R.id.detail_image);
   tv.setText(mVideo.getDescription());
  new Thread(new Runnable() {
   public void run() {
      loadThumbnail(mVideo, iv);
    }
   }).start();
  }
}
public void setVideo(VideoItem video)
{
  mVideo = video;
  showDetails();
}
@Override
  public void onResume(){
  super.onResume();
  showDetails();
}
```

1.  现在，在`DetailsFragment`类中添加`loadThumbnail`方法和从给定 URL 加载缩略图图像的实现：

```kt
private void loadThumbnail(VideoItem video,final  
 ImageView iv){
try 
 {
    URL url = new URL(video.getThumbnailURL());
   final Bitmap bmp = BitmapFactory.decodeStream(   
    url.openConnection().getInputStream());

   getActivity().runOnUiThread(new Runnable() {
    public void run() {
      iv.setImageBitmap(bmp);
     }
    });
 }
 catch (Exception ex){
    Log.d(this.getClass().toString(), ex.getMessage());
 }
}
```

1.  如果用户在`ListFragment`类的列表视图中选择了一个项目，我们需要告诉`DetailFragment`显示相应的详情。在`ListFragment`类的`onCreateView`方法中，添加`onItemClick`处理程序：

```kt
mListView.setOnItemClickListener(new 
 AdapterView.OnItemClickListener() 
{
  @Override
  public void onItemClick(AdapterView<?> adapterView,    
    View view, int i, long l) 
    {
        VideoItem video = mVideos.get(i);
        onVideoClicked(video);
    }
});
return view;
```

1.  在`MainActivity`类中，添加两个静态成员，它们将代表`ListFragment`和`DetailsFragment`类的标签：

```kt
public static String TAG_LIST_FRAGMENT = "LIST";
public static String TAG_DETAILS_FRAGMENT = "DETAILS";

```

在`ListFragment`类中创建`onVideoClicked`方法。如果`DetailsFragment`存在（有一个带有`DETAILS`标签的片段），它将调用`DetailsFragment`的`showDetails`方法：

```kt
private void onVideoClicked(VideoItem video) {  
  DetailFragment detailsFragment = (DetailFragment)   
   getFragmentManager().findFragmentByTag(   
    MainActivity.TAG_DETAILS_FRAGMENT);
if (detailsFragment != null) { 
  detailsFragment.setVideo(video);}
}
```

1.  我们快要完成了。在`activity_main.xml`布局中，我们为片段创建了一个容器。现在我们将添加一些代码，以在该容器中显示`ListFragment`的内容。在`MainActivity`类中，为两个片段添加两个私有成员：

```kt
private DetailFragment mDetailsFragment;
private ListFragment mListFragment;
```

1.  创建`ListFragment`并将其添加到容器中：

```kt
mListFragment = new ListFragment();
FragmentTransaction ft =  
 getFragmentManager().beginTransaction();
ft.add(R.id.main_container_for_list_fragment, 
 mListFragment, TAG_LIST_FRAGMENT);
ft.commit();
```

1.  让我们为主活动创建另一个布局，但这次是为大屏幕，比如平板电脑。在`res`文件夹中，通过右键单击`res`项目，添加一个新的 Android 资源目录。选择**layout**作为**资源类型**，将目录命名为`layout-large`，然后单击 To 按钮。

1.  在新的`layout-large`目录中，添加一个新的布局并命名为`activity_main`。平板设备足够大，可以容纳我们的两个片段，因此对于此布局，我们将创建两个容器：一个用于列表，一个用于详情：

```kt
<?xml version="1.0" encoding="utf-8"?><FrameLayout xmlns:android=  
 "http://schemas.android.com/apk/res/android"
android:layout_width="match_parent"android:layout_height="match_parent"android:id="@+id/main_container">
<FrameLayout
android:layout_width="300dp"
android:layout_height="match_parent"
android:background="@android:color/holo_orange_light"
android:id="@+id/main_container_for_list_fragment">
</FrameLayout>
<FrameLayout
android:id="@+id/main_container_for_detail_fragment"android:background="@android:color/holo_blue_light"
android:layout_marginLeft="300dp"
android:layout_width="match_parent"
android:layout_height="match_parent">
</FrameLayout>
</FrameLayout>
```

1.  修改`MainActivity`的`onCreate`实现。如果容器可用，我们也将加载详情片段。将`commit`调用移到最后：

```kt
mListFragment = new ListFragment();
FragmentTransaction ft =  
 getFragmentManager().beginTransaction();
ft.add(R.id.main_container_for_list_fragment,  mListFragment, TAG_LIST_FRAGMENT);
if (findViewById(  
 R.id.main_container_for_detail_fragment)!= null){
  mDetailsFragment = new DetailFragment();ft.add(R.id.main_container_for_detail_fragment,  
  mDetailsFragment, TAG_DETAILS_FRAGMENT);
}
ft.commit();
```

1.  还有一件事，如果你允许我解释。嗯，实际上有几件事。如果应用正在手机上运行，我们需要从列表片段视图导航到详情片段视图。修改`MainActivity`文件中的`onVideoClicked`方法，以便在那里创建详情片段：

```kt
private void onVideoClicked(VideoItem video) {
  DetailFragment detailsFragment = (DetailFragment)    
   getFragmentManager().findFragmentByTag(  
    MainActivity.TAG_DETAILS_FRAGMENT);
 if (detailsFragment != null) {
   detailsFragment.setVideo(video);
 }
 else
 {
   FragmentTransaction ft =  getFragmentManager().beginTransaction();
   detailsFragment = new DetailFragment();
   ft.add(R.id.main_container_for_list_fragment,  
    detailsFragment, MainActivity.TAG_DETAILS_FRAGMENT);
   ft.addToBackStack(MainActivity.TAG_DETAILS_FRAGMENT); 
   ft.commit();
   detailsFragment.setVideo(video);
 }
}
```

1.  我们在上一步中添加的`addToBackStack`调用通知片段管理器所有片段都在堆栈上，因此我们可以提供导航方式。我们需要告诉我们的活动在按下返回按钮时如何行为：我们想离开活动还是我们想从堆栈中弹出一个片段？我们将覆盖`MainActivity`的`onBackPressed`方法，就像这样：

```kt
@Override 
public void onBackPressed() {
if (getFragmentManager().getBackStackEntryCount()>0){
        getFragmentManager().popBackStack();
    }
else {
this.finish();
    }
}
```

我们完成了！我们有一些工作要做，但现在我们有一个可以在具有导航的手机上运行并且如果有足够的空间将显示两个片段的应用程序，就像平板电脑一样。

为了查看差异，请在智能手机和平板电脑上运行应用程序。在手机上，它将类似于以下屏幕截图。在平板电脑上（如果您没有可用的平板电脑，可以使用 Genymotion），列表和详细信息都显示在单个视图中：

![粘合在一起！](img/B04299_05_02.jpg)

## 还有更多…

下一个教程将展示如何实现允许我们观看刚刚找到的视频的功能。毕竟，播放视频是我们想要的！

# 媒体播放

在上一个教程中，我们从 YouTube 检索了搜索结果，并在列表和详细片段中显示了它们。找到的条目代表视频，因此如果我们能够在应用程序中播放它们，那将是很好的。让我们找到一种方法来做到这一点。

由于我们知道视频 ID，因此很容易为其组合 URL 并在 web 视图中加载它们；但是，Google 为此提供了更简单的解决方案，并为此提供了 YouTube Android Player API。它有一些限制，但足够有趣。

## 准备工作

要完成本教程，您需要完成上一个教程，因为本教程从上一个教程结束的地方开始。虽然我建议您在物理手机和平板电脑上测试应用程序，但您当然也可以使用 Genymotion。

如果您使用虚拟设备，那么谷歌应用程序（以及 API 和播放器所依赖的 YouTube 应用程序）将丢失，并且该应用程序将因此失败。您需要首先在虚拟设备上下载并安装它们。

## 如何做…

让我们看看如何通过以下步骤扩展应用程序，以便为我们播放视频：

1.  从[`developers.google.com/youtube/android/player/downloads`](https://developers.google.com/youtube/android/player/downloads)下载 YouTube Player API。

1.  在下载的文件中，在`libs`文件夹中找到`YouTubeAndroidPlayerApi.jar`文件并复制它。

1.  从上一个教程中打开项目。

1.  在`app`模块中的`libs`文件夹中找到`libs`文件夹，并粘贴`YouTubeAndroidPlayerApi.jar`文件。

1.  `build.gradle`文件中的依赖项可能已经准备好包括`lib`文件中的任何文件；但是如果没有，添加依赖项：

```kt
compile fileTree(dir: 'libs', include: ['YouTubeAndroidPlayerApi.jar'])
```

1.  单击立即同步链接，或者如果它没有出现，请单击工具栏上的**使用 Gradle 文件同步项目**按钮。

1.  在`MainActivity`类中，添加一个用于将要创建的播放器片段的静态标签。还添加`YouTubePlayerFragment`的私有成员和一个公共成员来存储 YouTube 播放器，如果初始化成功的话：

```kt
public static String TAG_PLAYER_FRAGMENT = "PLAYER";
private YouTubePlayerFragment mPlayerFragment;
public YouTubePlayer mYouTubePlayer = null;
```

1.  在`layout-large`目录中打开`activity_main.xml`，将详细片段的高度更改为`300dp`，并将`YouTubePlayerFragment`添加到其中。预览可能会抱怨，因为它不知道应该如何渲染，但只要包被识别，这并不是真正的问题，如果您已成功完成步骤 5 和 6，那么包将被识别：

```kt
<?xml version="1.0" encoding="utf-8"?>
<FrameLayout
android:layout_width="match_parent"android:layout_height="match_parent"android:id="@+id/main_container">
<FrameLayout
android:layout_width="300dp"android:layout_height="match_parent"android:background="@android:color/holo_orange_light"android:id="@+id/main_container_for_list_fragment"></FrameLayout>
<FrameLayout
android:id="@+id/main_container_for_detail_fragment"android:background="@android:color/holo_blue_light"android:layout_marginLeft="300dp"android:layout_width="match_parent"android:layout_height="300dp"></FrameLayout>
<fragment
android:name="com.google.android.youtube.player.YouTubePlayerFragment"
android:id="@+id/main_youtube_player_fragment"android:layout_marginTop="300dp"android:layout_marginLeft="300dp"android:layout_width="match_parent"android:layout_height="match_parent"android:layout_weight="3"/>
</FrameLayout>
```

1.  在`onCreateView`中，在`ft.commit`之前，找到播放器片段的容器并初始化`YouTuberPlayer`：

```kt
mPlayerFragment = (YouTubePlayerFragment)  
 getFragmentManager().findFragmentById(
  R.id.main_youtube_player_fragment);if (mPlayerFragment != null) {
  ft.add(mPlayerFragment, TAG_PLAYER_FRAGMENT);  mPlayerFragment.initialize("Your API key", new 
   YouTubePlayer.OnInitializedListener() 
  {
   @Override 
     public void onInitializationSuccess( YouTubePlayer.Provider   
     provider, YouTubePlayer youTubePlayer, boolean isRestored) 
   {
     mYouTubePlayer = youTubePlayer;}
   @Override 
    public void onInitializationFailure(YouTubePlayer.Provider    
    provider, YouTubeInitializationResult 
     youTubeInitializationResult) {
      Log.d(this.getClass().toString(),   
       youTubeInitializationResult.toString()); 
 });
}
```

1.  在`DetailFragment`中，在`onCreateView`方法中为播放按钮添加一个点击处理程序，就在返回视图对象之前：

```kt
view.findViewById(R.id.detail_button_play).setOnClickListener(
 new View.OnClickListener() {
  @Override
  public void onClick(View v) {
    playVideo();}
});
```

1.  在`DetailFragment`中创建`playVideo`方法。如果播放器片段存在（在大屏幕设备上），并且已经初始化，它将播放视频；如果不存在（在小屏幕设备上），我们将创建一个播放器片段，初始化它，并将其添加到堆栈中：

```kt
private void playVideo(){
if (getActivity() != null && 
 ((MainActivity)getActivity()).mYouTubePlayer != null){
    ((MainActivity)getActivity()  
     ).mYouTubePlayer.cueVideo(mVideo.getId());
 }
 else {
    FragmentTransaction ft =  
     getFragmentManager().beginTransaction();
    YouTubePlayerFragment playerFragment = new 
    YouTubePlayerFragment();
   ft.add(R.id.main_container_for_list_fragment,   
    playerFragment, MainActivity.TAG_DETAILS_FRAGMENT);
   ft.addToBackStack(MainActivity.TAG_PLAYER_FRAGMENT);
   ft.commit();
   playerFragment.initialize("Your API key", new 
    YouTubePlayer.OnInitializedListener() {
      @Override
     public void onInitializationSuccess(YouTubePlayer.Provider 
       provider, YouTubePlayer youTubePlayer, boolean 
       isRestored) {
         if (!isRestored) {
             youTubePlayer.cueVideo(mVideo.getId());
          }
      }
      @Override
	   public void onInitializationFailure(YouTubePlayer.Provider 
       provider, YouTubeInitializationResult 
        youTubeInitializationResult) {
        Log.d(this.getClass().toString(),   
         youTubeInitializationResult.toString()); 
      }
   });
 }
}
```

通过这样，我们已经添加了一个简单但完全功能的实现来播放所选视频。

## 还有更多...

有许多选项可用于播放视频，例如全屏或原位播放，带按钮或不带按钮等。使用 Chrome Cast，媒体也可以发送到您的电视上，或者正如我们将在最后的食谱中看到的那样，我们可以为 Android TV 创建一个应用程序。

# 电视和媒体中心

无聊！电视上又没有什么好看的！至少没有什么看起来足够有趣的东西。运行在 Android 上的智能电视为开发者创造了一个全新有趣的世界。最终，我们得到了我们应得的屏幕尺寸！

然而，它也拥有不同类型的受众。用户与他们的手机和平板电脑的互动程度非常高。当涉及观看电视时，焦点更多地放在消费上。

好吧，电视上有什么？泡杯茶，开始观看节目。偶尔，用户可能对一些互动感兴趣（这种现象大多出现在第二屏应用程序中，因为并非每个人都拥有智能电视），但大多数时候，电视观众只是想靠在椅子上放松。

## 准备工作

这个食谱需要 Android Studio 正常运行和安装最新的 SDK。在这个食谱中，我们将为您提供一个关于电视应用程序的简要介绍。只需几个步骤，我们就可以创建一个媒体中心应用程序。不用担心，您不需要拥有 Android 电视。我们将创建一个虚拟的电视。

## 如何做...

让我们看看开发 Android TV 应用程序需要做什么：

1.  在 Android Studio 中创建一个新项目。将其命名为`PersonalTeeVee`，然后点击“下一步”按钮。

1.  选择电视选项，然后点击“下一步”按钮。

1.  选择 Android TV Activity，然后点击下一步。

1.  在“Activity Name”字段中输入`TeeVeeActivity`，在“Title”字段中输入`Personal Tee Vee`，然后点击“完成”按钮。

1.  Android Studio 为您创建了一个手机和一个电视模块。将配置更改为电视。您将看到如下图所示的内容：![如何做...](img/B04299_05_04.jpg)

1.  查看电视模块中的`AndroidManifest.xml`文件。注意“lean back”功能要求（告诉我们这是一个全屏体验的电视应用程序，没有任何重型互动，基本上是关于消费内容，比如观看视频）。还要注意我们不需要触摸屏。电视屏幕离得太远了，无法触摸。此外，没有人喜欢电视屏幕上的污渍：

```kt
<uses-feature
android:name="android.hardware.touchscreen"
android:required="false" />
<uses-feature
android:name="android.software.leanback"
android:required="true" />
```

1.  要测试电视应用程序，我们需要有一个虚拟电视设备。从“工具”|“Android”菜单中打开“AVD 管理器”选项。

1.  点击“创建虚拟设备”按钮。

1.  从类别列表中选择电视，并选择一个电视设备（1080p 或更高）。然后点击“下一步”按钮。

1.  选择一个系统镜像。例如，我选择了**API 级别 22 x86**。点击“下一步”。

1.  修改 AVD 的名称为您认为最合适的名称，然后点击“完成”按钮。将为您创建一个新的虚拟电视设备。

1.  点击播放按钮启动您的电视设备。如果它说**Google Play 服务已停止**，您现在可以忽略这条消息（尽管如果您想播放视频，您将需要它）。

1.  一旦设备启动，从 Android Studio 运行您的电视应用程序。默认情况下，它看起来像这样：![如何做...](img/B04299_05_05.jpg)

哇，这已经是一个完全功能的媒体中心应用程序了！

这只是一个简短的介绍，介绍了如何构建 Android TV 应用程序。玩玩它，调整一下。

## 还有更多...

虽然这个食谱中的应用程序专门用于电视，但我认为您没有理由不能将其制作成任何类型的设备的应用程序：手机、平板电脑和电视。如果您愿意，您可以将本章中的所有食谱合并为一个单一的应用程序。这是一个不错的挑战，不是吗？

除了 YouTube 之外，还有一些有趣的与媒体相关的 API 可以调查。例如，在[www.programmableweb.com](http://www.programmableweb.com)上，你可以找到一些有趣的 API。以下是其中一些：

| API | 导航 |
| --- | --- |
| YouTube | http://www.programmableweb.com/api/youtube-live-streaming |
| Vimeo | http://www.programmableweb.com/api/vimeo |
| Hey! Spread | http://www.programmableweb.com/api/heyspread |
| Pirateplay | http://www.programmableweb.com/api/pirateplay |
| Tinysong | http://www.programmableweb.com/api/tinysong |
| TwitVid | http://www.programmableweb.com/api/twitvid |

现在我们知道从哪里获取媒体项目，如何播放它们，以及如何自动创建媒体中心应用程序了。

接下来：让我们通过捕捉一些图像来创建一些媒体。下一章见！

## 另请参阅

+   第六章，*捕捉和分享*
