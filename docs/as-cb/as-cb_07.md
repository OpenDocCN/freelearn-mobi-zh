# 第七章：内容提供程序和观察者

在大多数应用程序中，我们需要持久化数据，并经常使用 SQLite 来实现这一目的。

非常常见的情况是列表和详细视图。通过使用内容提供程序，我们不仅提供了应用程序之间的通信方式，还在我们自己的应用程序中节省了大量工作。

在本章中，您将学习：

+   内容提供程序

+   使用内容提供程序消耗和更新数据

+   将投影更改为在您的应用程序中显示**关键绩效指标**（**KPIs**）

+   使用内容提供程序与其他应用程序进行通信

# 介绍

如果我们想要创建一个新的行，或者想要编辑数据库中的一行，应用程序将显示包含详细信息的片段或活动，用户可以在那里输入或修改一些文本和其他值。一旦记录被插入或更新，列表需要知道这些变化。告诉列表活动或片段有关这些变化并不难做到，但有一种更优雅的方法可以实现这一点。为此，以及其他我们将在以后了解的原因，我们将研究内容提供程序的内容。

Android 内容提供程序框架允许我们为应用程序创建更好的设计。其中一个特点是它允许我们注意到某些数据已经发生了变化。即使在不同的应用程序之间也可以工作。

# 内容提供程序

构建内容提供程序是一件非常聪明的事情。内容提供程序 API 具有一个有趣的功能，允许应用程序观察数据集的变化。

内容提供程序将一个进程中的数据与另一个进程中运行的代码连接起来，甚至可以在两个完全不同的应用程序之间进行连接。如果您曾经编写过从 Gallery 应用中选择图像的代码，您可能已经经历过这种行为。某些组件操作其他组件依赖的持久数据集。内容提供程序可以使用许多不同的方式来存储数据，可以存储在数据库中，文件中，甚至可以通过网络进行存储。

数据集由唯一的 URI 标识，因此可以要求在某个 URI 发生变化时进行通知。这就是观察者模式的应用之处。

观察者模式是一种常见的软件设计模式，其中一个对象（主题）具有一个或多个依赖对象（观察者，也称为监听器），它们将自动被通知任何状态更改。

## 还有更多...

### 设计模式

要了解更多关于这个和其他**面向对象**（**OO**）设计模式，您可以查看[`www.oodesign.com/observer-pattern.html`](http://www.oodesign.com/observer-pattern.html)。

### RxJava

RxJava 是一个非常有趣的库，也可以在 Android 版本中使用。响应式编程与观察者模式有主要相似之处。响应式代码的基本构建块也是可观察对象和订阅者。

要了解更多关于 Rx 和 RxJava，您可以访问这些网站：

+   [`github.com/reactivex/rxandroid`](https://github.com/reactivex/rxandroid)

+   [`github.com/ReactiveX/RxJava/wiki/How-To-Use-RxJava`](https://github.com/ReactiveX/RxJava/wiki/How-To-Use-RxJava)

+   [`blog.danlew.net/2014/09/15/grokking-rxjava-part-1/`](http://blog.danlew.net/2014/09/15/grokking-rxjava-part-1/)

## 另请参阅

+   第八章 ，*提高质量*

# 使用内容提供程序消耗和更新数据 - 每日想法

为了演示如何创建和使用内容提供程序，我们将创建一个应用程序，用于存储您每天的想法和快乐程度。

是的，有一些应用程序正在这样做；但是，如果您想创建一个应用程序来记录体育笔记和分数，可以随意修改代码，因为它基本上涉及相同的功能。

在这个示例中，我们将使用内容提供程序存储新的想法并检索它们。对于应用程序的各个元素，我们将使用片段，因为它们将清楚地展示观察者模式的效果。

## 准备工作

对于这个配方，您只需要运行 Android Studio 并拥有一个物理或虚拟的 Android 设备。

## 如何做...

让我们看看如何使用内容提供程序设置项目。我们将使用导航抽屉模板：

1.  在 Android Studio 中创建一个名为`DailyThoughts`的新项目。点击**下一步**按钮。

1.  选择**手机和平板电脑**选项，然后点击**下一步**按钮。

1.  选择**导航抽屉活动**，然后点击**下一步**按钮。

1.  接受**自定义活动**页面上的所有值，然后点击**完成**按钮。

1.  打开`res/values`文件夹中的`strings.xml`文件。修改以`title_section`开头的条目的字符串。用我们应用程序所需的菜单项替换它们。还替换`action_sample`字符串：

```kt
<string name="title_section_daily_notes">Daily  
 thoughts</string><string name="title_section_note_list">Thoughts 
 list</string>
<string name="action_add">Add thought</string>
```

1.  打开`NavigationDrawerFragment`文件，在`onCreate`方法中，相应地修改适配器的字符串：

```kt
mDrawerListView.setAdapter(new ArrayAdapter<String>(
        getActionBar().getThemedContext(),
        android.R.layout.simple_list_item_activated_1,
        android.R.id.text1,
        new String[]{
                getString(R.string.title_section_daily_notes),
                getString(R.string.title_section_note_list)
        }));
```

1.  在同一个类中，在`onOptionsItemSelected`方法中，删除显示 toast 的第二个`if`语句。我们不需要它。

1.  从`res/menu`文件夹中打开`main.xml`。删除设置项，并修改第一项，使其使用`action_add`字符串。还重命名它的 ID 并为其添加一个漂亮的图标：

```kt
<menu xmlns:android= 
 "http://schemas.android.com/apk/res/android"  
   tools:context=".MainActivity">
<item android:id="@+id/action_add"  
 android:title="@string/action_add"android:icon="@android:drawable/ic_input_add"android:showAsAction="withText|ifRoom" />
</menu>
```

1.  在`MainActivity`文件中，在`onSectionAttached`部分，为不同的选项应用正确的字符串：

```kt
public void onSectionAttached(int number) {
    switch (number) {
        case 0:
            mTitle = getString(  
             R.string.title_section_daily_notes);
            break;
        case 1:
            mTitle = getString( 
             R.string.title_section_note_list);
             break;
    }
}
```

1.  创建一个名为`db`的新包。在这个包中，创建一个名为`DatabaseHelper`的新类，它继承`SQLiteOpenHelper`类。它将帮助我们为我们的应用程序创建一个新的数据库。它将只包含一个表：`thoughts`。每个`Thought table`将有一个 id，一个名称和一个幸福评分：

```kt
public class DatabaseHelper extends SQLiteOpenHelper {
    public static final String DATABASE_NAME = 
     "DAILY_THOUGHTS";
    public static final String THOUGHTS_TABLE_NAME =   
     "thoughts";
    static final int DATABASE_VERSION = 1;
    static final String CREATE_DB_TABLE =
      " CREATE TABLE " + THOUGHTS_TABLE_NAME +
      " (_id INTEGER PRIMARY KEY AUTOINCREMENT, " +" name TEXT NOT NULL, " +" happiness INT NOT NULL);";public DatabaseHelper(Context context){
        super(context, DATABASE_NAME, null, 
         DATABASE_VERSION);}
    @Override
    public void onCreate(SQLiteDatabase db)
    {
        db.execSQL(CREATE_DB_TABLE);
    }
    @Override 
	 public void onUpgrade(SQLiteDatabase db, int 
     oldVersion, int newVersion) {
        db.execSQL("DROP TABLE IF EXISTS " +  
         THOUGHTS_TABLE_NAME);
        onCreate(db);}
}
```

1.  创建另一个包并命名为`providers`。在这个包中，创建一个名为`ThoughtsProvider`的新类。这将是我们所有日常想法的内容提供程序。将其作为`ContentProvider`类的后代。

1.  从**代码**菜单中，选择**实现方法**选项。在出现的对话框中，所有可用的方法都被选中。接受这个建议，然后点击**确定**按钮。您的新类将扩展这些方法。

1.  在类的顶部，我们将创建一些静态变量：

```kt
static final String PROVIDER_NAME =  
 "com.packt.dailythoughts";
static final String URL = "content://" + PROVIDER_NAME +  
 "/thoughts";
public static final Uri CONTENT_URI = Uri.parse(URL);
public static final String THOUGHTS_ID = "_id";
public static final String THOUGHTS_NAME = "name";
public static final String THOUGHTS_HAPPINESS = 
 "happiness";
static final int THOUGHTS = 1;
static final int THOUGHT_ID = 2;
static final UriMatcher uriMatcher;
static{
    uriMatcher = new UriMatcher(UriMatcher.NO_MATCH);
    uriMatcher.addURI(PROVIDER_NAME, "thoughts", 
     THOUGHTS);
    uriMatcher.addURI(PROVIDER_NAME, "thoughts/#",   
     THOUGHT_ID);
}
```

1.  添加一个私有成员`db`，引用`SQLiteDatabase`类，并修改`onCreate`方法。我们创建一个新的数据库助手：

```kt
private SQLiteDatabase db;
@Override 
   public boolean onCreate() {
    Context context = getContext();
    DatabaseHelper dbHelper = new DatabaseHelper(context);
    db = dbHelper.getWritableDatabase();
    return (db == null)? false:true;
}
```

### 查询

接下来，实现`query`方法。查询返回一个游标对象。游标表示查询的结果，并指向查询结果中的一个，因此结果可以被高效地缓冲，因为它不需要将数据加载到内存中：

```kt
private static HashMap<String, String> 
 THOUGHTS_PROJECTION; 
@Override 
public Cursor query(Uri uri, String[] projection, 
 String selection, String[] selectionArgs, String 
  sortOrder) {
   SQLiteQueryBuilder builder = new SQLiteQueryBuilder();
   builder.setTables( 
    DatabaseHelper.THOUGHTS_TABLE_NAME);
   switch (uriMatcher.match(uri)) {
      case THOUGHTS:
        builder.setProjectionMap(
         THOUGHTS_PROJECTION);
         break;
      case THOUGHT_ID:
        builder.appendWhere( THOUGHTS_ID + "=" + uri.getPathSegments().get(1));
        break;
      default:
        throw new IllegalArgumentException(
         "Unknown URI: " + uri);
    }
    if (sortOrder == null || sortOrder == ""){
        sortOrder = THOUGHTS_NAME;
    }
    Cursor c = builder.query(db, projection,selection, selectionArgs,null, null, sortOrder);
    c.setNotificationUri(    
     getContext().getContentResolver(), uri);
    return c;
}
```

### 注意

`setNotificationUri`调用注册指令以监视内容 URI 的更改。

我们将使用以下步骤实现其他方法：

1.  实现`getType`方法。`dir`目录表示我们想要获取所有的想法记录。`item`术语表示我们正在寻找特定的想法：

```kt
@Override 
public String getType(Uri uri) {
    switch (uriMatcher.match(uri)){
      case THOUGHTS:
        return "vnd.android.cursor.dir/vnd.df.thoughts";
     case THOUGHT_ID:
       return "vnd.android.cursor.item/vnd.df.thoughts";
     default:
       throw new IllegalArgumentException(
        "Unsupported URI: " + uri);
    }
}
```

1.  实现`insert`方法。它将基于提供的值创建一个新记录，如果成功，我们将收到通知：

```kt
@Override
public Uri insert(Uri uri, ContentValues values) {
   long rowID = db.insert(  
    DatabaseHelper.THOUGHTS_TABLE_NAME , "", values);
   if (rowID > 0)
   {
      Uri _uri = ContentUris.withAppendedId(CONTENT_URI, 
       rowID);
      getContext().getContentResolver().notifyChange( _uri, 
       null);
      return _uri;
    }
    throw new SQLException("Failed to add record: " + uri);
}
```

1.  `delete`和`update`方法超出了本配方的范围，所以我们现在不会实现它们。挑战：在这里添加您自己的实现。

1.  打开`AndroidManifest.xml`文件，并在`application`标签内添加`provider`标签：

```kt
<providerandroid:name=".providers.ThoughtsProvider"android:authorities="com.packt.dailythoughts"android:readPermission=  
     "com.packt.dailythoughts.READ_DATABASE"android:exported="true" />
```

### 注意

出于安全原因，在大多数情况下，您应该将导出属性的值设置为`false`。我们将此属性的值设置为`true`的原因是，稍后我们将创建另一个应用程序，该应用程序将能够从此应用程序中读取内容。

1.  添加其他应用程序读取数据的权限。我们将在最后一个配方中使用它。将其添加到`application`标签之外：

```kt
<permission   
 android:name="com.packt.dailythoughts.READ_DATABASE"android:protectionLevel="normal"/>
```

1.  打开`strings.xml`文件并向其中添加新的字符串：

```kt
<string name="my_thoughts">My thoughts</string>
<string name="save">Save</string>
<string name="average_happiness">Average 
  happiness</string>
```

1.  创建两个新的布局文件：`fragment_thoughts.xml`用于我们的想法列表和`fragment_thoughts_detail`用于输入新的想法。

1.  为`fragment_thoughts.xml`定义布局。 一个`ListView`小部件很适合显示所有的想法：

```kt
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android= 
 "http://schemas.android.com/apk/res/android"
   android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical" >
    <ListView
        android:id="@+id/thoughts_list"android:layout_width="match_parent"android:layout_height="wrap_content" ></ListView>
</LinearLayout> 
```

1.  `fragment_thoughts_detail.xml`的布局将包含`EditText`和`RatingBar`小部件，以便我们可以输入我们的想法和我们当前的幸福程度：

```kt
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android=
  "http://schemas.android.com/apk/res/android"android:orientation="vertical"android:layout_gravity="center"android:layout_margin="32dp"android:padding="16dp"android:layout_width="match_parent"android:background="@android:color/holo_green_light"android:layout_height="wrap_content">
    <TextView
        android:layout_margin="8dp"android:textSize="16sp"android:text="@string/my_thoughts"
     android:layout_width="match_parent"android:layout_height="wrap_content" />
    <EditText
        android:id="@+id/thoughts_edit_thoughts"android:layout_margin="8dp"android:layout_width="match_parent"android:layout_height="wrap_content" />
    <RatingBar
        android:id="@+id/thoughs_rating_bar_happy"android:layout_width="wrap_content"android:layout_height="wrap_content"android:layout_gravity="center_horizontal"android:clickable="true"android:numStars="5"android:rating="0" />
    <Button
        android:id="@+id/thoughts_detail_button"android:text="@string/save"          
        android:layout_width="match_parent"android:layout_height="wrap_content" />
</LinearLayout>
```

1.  还要为想法列表中的行创建布局。将其命名为`adapter_thought.xml`。添加文本视图以显示 ID、标题或名称和评分：

```kt
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android=
  "http://schemas.android.com/apk/res/android"
    android:orientation="vertical"
    android:layout_gravity="center"
    android:layout_margin="32dp"
    android:padding="16dp"
    android:layout_width="match_parent"
    android:background=
     "@android:color/holo_green_light"
    android:layout_height="wrap_content">
    <TextView
        android:layout_margin="8dp"
        android:textSize="16sp"
        android:text="@string/my_thoughts"
     android:layout_width="match_parent"
  android:layout_height="wrap_content" />
    <EditText
        android:id="@+id/thoughts_edit_thoughts"
        android:layout_margin="8dp"
        android:layout_width="match_parent"
        android:layout_height="wrap_content" />
    <RatingBar
        android:id="@+id/thoughs_rating_bar_happy"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="center_horizontal"
        android:clickable="true"
        android:numStars="5"
        android:rating="0" />
    <Button
        android:id="@+id/thoughts_detail_button"
        android:text="@string/save"          
        android:layout_width="match_parent"
        android:layout_height="wrap_content" />
</LinearLayout>

```

1.  创建一个新的包，命名为：`fragments`，并向其中添加两个新的类：`ThoughtsDetailFragment`和`ThoughtsFragment`，它们都将是`Fragment`类的子类。

1.  在`ThoughtsFragment`类中，添加`LoaderCallBack`的实现：

```kt
public class ThoughtsFragment extends Fragment   
  implementsLoaderManager.LoaderCallbacks<Cursor>{
```

1.  从**代码**菜单中选择**实现方法**，接受建议的方法，并单击**确定**按钮。它将创建`onCreateLoader`，`onLoadFinished`和`onLoaderReset`的实现。

1.  添加两个私有成员，它们将保存列表视图和适配器：

```kt
private ListView mListView;private SimpleCursorAdapter mAdapter;
```

1.  重写`onCreateView`方法，在其中我们将填充布局并获取对列表视图的引用。从这里，我们还将调用`getData`方法：

```kt
@Override
public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
    final View view = inflater.inflate( 
     R.layout.fragment_thoughts, container, false);
    mListView = (ListView)view.findViewById( 
     R.id.thoughts_list);
    getData();
    return view;
}
```

### 加载程序管理器

以下步骤将帮助我们向应用程序添加加载程序管理器：

1.  实现`getData`方法。我们将使用`loaderManager`的`initLoader`方法。投影定义了我们想要检索的字段，目标是`adapter_thought_title`布局中的 ID 数组，这将节省我们使用`SimpleCursorAdapter`类的一些工作。

```kt
private void getData(){String[] projection = new String[] { 
     ThoughtsProvider.THOUGHTS_ID,   
     ThoughtsProvider.THOUGHTS_NAME, 
     ThoughtsProvider.THOUGHTS_HAPPINESS};
    int[] target = new int[] {    
     R.id.adapter_thought_id,  
     R.id.adapter_thought_title,  
     R.id.adapter_thought_rating };
    getLoaderManager().initLoader(0, null, this);
    mAdapter = new SimpleCursorAdapter(getActivity(),   
     R.layout.adapter_thought, null, projection,  
      target, 0);
    mListView.setAdapter(mAdapter); 
}
```

1.  在`initLoader`调用之后，需要创建一个新的加载程序。为此，我们将不得不实现`onLoadFinished`方法。我们将使用与适配器相同的投影，并使用我们在前面步骤中创建的`ThoughtsProvider`的`uri`内容创建`CursorLoader`类。我们将按 ID（降序）对结果进行排序：

```kt
@Override
public Loader<Cursor> onCreateLoader(int id, Bundle args) {
        String[] projection = new String[] { 
     ThoughtsProvider.THOUGHTS_ID,   
     ThoughtsProvider.THOUGHTS_NAME, 
     ThoughtsProvider.THOUGHTS_HAPPINESS};
    String sortBy = "_id DESC";CursorLoader cursorLoader = new 
    CursorLoader(getActivity(), 
    ThoughtsProvider.CONTENT_URI, projection, null, 
     null, sortBy);
    return cursorLoader;
}
```

1.  在`onLoadFinished`中，通知适配器加载了数据：

```kt
mAdapter.swapCursor(data);
```

1.  最后，让我们为`onLoaderReset`方法添加实现。在这种情况下，数据不再可用，因此我们可以删除引用。

```kt
mAdapter.swapCursor(null);
```

1.  让我们来看看`ThoughtsDetailFragment`方法。重写`onCreateView`方法，填充布局，并为布局中的保存按钮添加点击监听器：

```kt
@Override
public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
    final View view = inflater.inflate( 
     R.layout.fragment_thoughts_detail, container,  
      false); 
   view.findViewById( 
    R.id.thoughts_detail_button).setOnClickListener( 
     new View.OnClickListener() {
        @Override
        public void onClick(View v) {
            addThought();
        }
    });
    return view;
}
```

1.  添加`addThought`方法。我们将根据通过`EditText`和`RatingBar`字段输入创建新的内容值。我们将根据提供的 URI 使用内容解析器的`insert`方法。插入新记录后，我们将清除输入：

```kt
private void addThought(){
    EditText thoughtsEdit = 
     (EditText)getView().findViewById(    
      R.id.thoughts_edit_thoughts);
    RatingBar happinessRatingBar =            
     (RatingBar)getView().findViewById(
      R.id.thoughs_rating_bar_happy);
    ContentValues values = new ContentValues();
    values.put(ThoughtsProvider.THOUGHTS_NAME, 
     thoughtsEdit.getText().toString());
    values.put(ThoughtsProvider.THOUGHTS_HAPPINESS,    
     happinessRatingBar.getRating());
    getActivity().getContentResolver().insert( 
     ThoughtsProvider.CONTENT_URI, values);
    thoughtsEdit.setText("");
    happinessRatingBar.setRating(0);
}
```

1.  再次是将事物粘合在一起的时候了。打开`MainActivity`类，并添加两个私有成员，它们将引用我们创建的片段，如下所示：

```kt
private ThoughtsFragment mThoughtsFragment;
private ThoughtsDetailFragment mThoughtsDetailFragment;
```

1.  添加两个私有成员，如果需要，将它们初始化，并返回实例：

```kt
private ThoughtsFragment getThoughtsFragment(){
    if (mThoughtsFragment==null) {
        mThoughtsFragment = new ThoughtsFragment();
    }
    return mThoughtsFragment;
}
private ThoughtsDetailFragment 
getThoughtDetailFragment() {
   if (mThoughtsDetailFragment==null){
    mThoughtsDetailFragment = new ThoughtsDetailFragment();
    }
    return mThoughtsDetailFragment;
}
```

1.  删除`onNavigationDrawerItemSelected`的实现，并添加一个新的来显示想法列表。我们稍后将实现 KPI 选项：

```kt
@Override
  public void onNavigationDrawerItemSelected(int  
  position) {
   FragmentManager fragmentManager =    
    getFragmentManager();
   if (position==1) {
        fragmentManager.beginTransaction().   
         replace(R.id.container, 
          getThoughtsFragment()).commit();
    }
}
```

1.  在`onOptionsItemSelected`方法中，测试 id 是否为`action_add`，如果是，则显示详细片段。在获取 id 的行后立即添加实现：

```kt
if (id== R.id.action_add){FragmentManager fragmentManager = 
     getFragmentManager();
    fragmentManager.beginTransaction().add( 
     R.id.container, getThoughtDetailFragment()  
      ).commit();
}
```

### 注意

这里使用`add`而不是`replace`。我们希望详细片段出现在堆栈的顶部。

1.  保存详细信息后，片段必须再次被移除。再次打开`ThoughtsDetailFragment`。在`addThought`方法的末尾，添加以下内容以完成操作：

```kt
getActivity().getFragmentManager().beginTransaction().
 remove(this).commit();
```

1.  然而，最好让活动处理片段的显示，因为它们旨在成为活动的辅助程序。相反，我们将为`onSave`事件创建一个监听器。在类的顶部，添加一个`DetailFragmentListener`接口。还创建一个私有成员和一个 setter：

```kt
public interface DetailFragmentListener {
    void onSave();
}
private DetailFragmentListener 
 mDetailFragmentListener; 
public void setDetailFragmentListener(  
 DetailFragmentListener listener){
    mDetailFragmentListener = listener;
}
```

1.  在`addThought`成员的末尾添加这些行，以便让监听器知道已保存事物：

```kt
if (mDetailFragmentListener != null){
    mDetailFragmentListener.onSave();
}
```

1.  返回`MainActivity`类，并为其添加一个监听器实现。如果需要，您可以使用**代码**菜单中的**实现方法**选项：

```kt
public class MainActivity extends Activityimplements NavigationDrawerFragment. 
   NavigationDrawerCallbacks, 
    ThoughtsDetailFragment.DetailFragmentListener {
@Override 
 public void onSave() {      
  getFragmentManager().beginTransaction().remove(
   mThoughtsDetailFragment).commit();
}
```

1.  要告诉详细片段主活动正在监听，请滚动到`getThoughtDetailFragment`类并在创建新详细片段后立即调用`setListener`方法：

```kt
mThoughtsDetailFragment.setDetailFragmentListener(this);
```

现在运行应用程序，从导航抽屉中选择**Thoughts list**，然后单击加号添加新的想法。以下截图显示了添加想法的示例：

![加载程序管理器](img/B04299_07_01.jpg)

我们不需要告诉包含列表的片段有关我们在详细片段中创建的新想法。使用具有观察者的内容提供程序，列表将自动更新。

这样我们就可以完成更多，写更少容易出错的功能，从而写更少的代码，这正是我们想要的。它使我们能够提高代码的质量。

## 另请参阅

+   参见第五章, *大小很重要*

+   参见第八章, *提高质量*

# 更改投影以在应用程序中显示 KPI

我们可以使用不同的投影和相同的观察者模式来显示一些 KPI。实际上，这很容易，正如我们将在本示例中看到的那样。

## 准备工作

对于这个示例，您需要成功完成上一个示例。

## 如何做...

我们将继续在上一个示例中的应用程序上工作，并添加一个新视图来显示 KPI：

1.  打开您在上一个示例中工作的项目。

1.  添加一个新的布局，`fragment_thoughts_kpi.xml`：

```kt
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android=  
 "http://schemas.android.com/apk/res/android"
  android:orientation="vertical"   
  android:layout_width="match_parent"
  android:gravity="center_horizontal"   
  android:padding="16dp"
  android:layout_height="match_parent">
  <TextView
        android:id="@+id/thoughts_kpi_count"          
        android:textSize="32sp"
        android:layout_margin="16dp"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content" />
    <TextView
        android:id="@+id/thoughts_kpi_avg_happiness"
        android:text= "@string/average_happiness"
        android:textSize="32sp"
        android:layout_margin="16dp"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content" />
    <RatingBar
        android:id="@+id/thoughts_rating_bar_happy"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="center_horizontal"
        android:clickable="false"
        android:numStars="5"
        android:rating="0" />
</LinearLayout>

```

1.  添加一个新的片段并命名为`ThoughtsKpiFragment`。它是从`Fragment`类继承的。我们将在这里使用`LoaderManager`，所以它基本上看起来像这样：

```kt
public class ThoughtsKpiFragment extends Fragment    
 implements LoaderManager.LoaderCallbacks<Cursor> {
   @Override
    public Loader<Cursor> onCreateLoader(int id, Bundle args) {return null;
    }
    @Override
	public void onLoadFinished(Loader<Cursor> loader, Cursordata) {
    }
    @Override
    public void onLoaderReset(Loader<Cursor> loader) {
    }
}
```

1.  因为我们将使用两个加载程序来显示两个不同的 KPI，所以我们首先要添加两个常量值：

```kt
public static int LOADER_COUNT_THOUGHTS = 1;
public static int LOADER_AVG_RATING = 2;
```

1.  覆盖`onCreate`方法：

```kt
@Override
public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
    final View view = inflater.inflate( 
     R.layout.fragment_thoughts_kpi, container, false);
    getKpis();
    return view;
}
```

1.  创建`getKpis`方法（在这里我们为不同目的两次初始化加载程序）：

```kt
private void getKpis(){
    getLoaderManager().initLoader(LOADER_COUNT_THOUGHTS, null, 
     this);
    getLoaderManager().initLoader(LOADER_AVG_RATING, null, 
     this); 
}
```

1.  添加`onCreateLoader`方法的实现。这次投影取决于加载程序的 ID。投影就像您期望的那样，如果它是普通的 SQL。我们正在计算行数，并计算平均幸福指数：

```kt
@Override 
 public Loader<Cursor> onCreateLoader(int id, Bundle args) {
    if (id == LOADER_COUNT_THOUGHTS) {
      String[] projection = new String[] {"COUNT(*) AS kpi"};
      android.content.CursorLoader cursorLoader = new android.content.CursorLoader(getActivity(),  
        ThoughtsProvider.CONTENT_URI, projection, null, null, 
         null);
      return cursorLoader;
    }
    else {
      String[] projection = new String[]
         {"AVG(happiness) AS kpi"};
      android.content.CursorLoader cursorLoader = new 
      android.content.CursorLoader(getActivity(), 
       ThoughtsProvider.CONTENT_URI, projection, null, null, 
        null);
      return cursorLoader;}
}
```

1.  一旦数据到达，我们将到达`onLoadFinished`方法，并调用方法显示数据（如果有的话）：

```kt
@Override
public void onLoadFinished(Loader<Cursor> loader, Cursor data) {
    if (data == null || !data.moveToNext()) {
        return;
    }
    if (loader.getId() == LOADER_COUNT_THOUGHTS) {
        setCountedThoughts(data.getInt(0)); 
    }
    else{
        setAvgHappiness(data.getFloat(0));
    }
}
```

1.  添加`setCountedThoughts`和`setAvgHappiness`方法。如果片段仍附加到活动中，我们将更新文本视图或评分栏：

```kt
private void setCountedThoughts(final int counted){
    if (getActivity()==null){
        return;
    }
    getActivity().runOnUiThread(new Runnable() {
        @Override
        public void run() {
          TextView countText = (TextView)getView().findViewById(
             R.id.thoughts_kpi_count);
          countText.setText(String.valueOf(counted));
        }
    });
}
private void setAvgHappiness(final float avg){
    if (getActivity()==null){
        return;
    }
    getActivity().runOnUiThread(new Runnable() {
        @Override
		public void run() {
            RatingBar ratingBar =        
             (RatingBar)getView().findViewById(
              R.id.thoughts_rating_bar_happy);
            ratingBar.setRating(avg);}
    });
}
```

1.  在`MainActivity`文件中，添加一个 KPI 片段的私有成员：

```kt
private ThoughtsKpiFragment mThoughtsKpiFragment;
```

1.  创建一个`getKpiFragment`方法：

```kt
private ThoughtsKpiFragment getKpiFragment(){
    if (mThoughtsKpiFragment==null){
        mThoughtsKpiFragment = new ThoughtsKpiFragment();
    }
    return mThoughtsKpiFragment;
}
```

1.  找到`onNavigationDraweItemSelected`方法，并将其添加到`if`语句中：

```kt
… 
else if (position==0){ 
    fragmentManager.beginTransaction()
            .replace(R.id.container, getKpiFragment())
            .commit();
}
```

运行您的应用程序。现在我们的想法应用程序中有一些整洁的统计数据：

![如何做...](img/B04299_07_02.jpg)

在这个和上一个示例中，我们已经看到了一旦掌握了内容提供程序的概念，处理数据变得多么容易。

到目前为止，我们在同一个应用程序中完成了所有这些工作；然而，由于我们已经准备好导出内容提供程序，让我们找出如何在不同的应用程序中读取我们的想法。现在就让我们来做吧。

## 另请参阅

参见第五章, *大小很重要*

参见第八章, *提高质量*

# 使用内容提供程序与其他应用程序通信

如果您阅读谷歌关于内容提供程序的文档，您会注意到内容提供程序基本上是为了在请求时向其他应用程序提供数据。这些请求由`ContentResolver`类的方法处理。

我们将创建一个新的应用程序，它将从另一个应用程序中读取我们的日常想法。

## 准备工作

对于这个示例，您需要成功完成上一个示例。确保您也向应用程序添加了一些想法，否则将没有东西可读，正如显而易见的船长所告诉我们的那样。

## 如何做...

首先我们将创建一个新的应用程序。它将读取我们的想法。这是肯定的！

1.  在 Android Studio 中创建一个新项目，命名为`DailyAnalytics`，然后点击**确定**按钮。

1.  选择**手机和平板电脑**，然后点击**下一步**按钮。

1.  选择**空白活动**，然后点击**下一步**按钮。

1.  接受**自定义活动**视图中的所有值，然后点击**完成**按钮。

1.  打开`AndroidManifest.xml`文件，并添加与`DailyThought`应用程序通信所需的权限：

```kt
<uses-permission android:name=  
 "com.packt.dailythoughts.READ_DATABASE"/>
```

1.  打开`activity_main.xml`布局，并将`TextView`应用程序的`id`更改为`main_kpi_count`：

```kt
<TextView
    android:id="@+id/main_kpi_count"android:text="@string/hello_world"  
    android:layout_width="wrap_content"android:layout_height="wrap_content" />
```

1.  在`MainActivity`类中，添加`LoaderCallBack`实现：

```kt
public class MainActivity extends Activity  implementsLoaderManager.LoaderCallbacks<Cursor>
```

1.  在`onCreate`方法的末尾调用`initLoader`：

```kt
getLoaderManager().initLoader(0, null, this);
```

1.  为`onCreateLoader`方法添加一个实现。它的工作方式基本与应用程序的内容提供程序相同：

```kt
@Override
public Loader<Cursor> onCreateLoader(int id, Bundle args) {
    Uri uri = Uri.parse(  
     "content://com.packt.dailythoughts/thoughts");
    String[] projection = new String[] { "_id", "name", 
     "happiness"};
    String sortBy = "name";
    CursorLoader cursorLoader = new  
    android.content.CursorLoader(
     this,uri, projection, null, null, null);
    return cursorLoader;
}
```

1.  在`onLoadFinished`方法中，我们可以根据您在其他应用程序中输入的内容显示一些分析：

```kt
@Override
public void onLoadFinished(Loader<Cursor> loader, 
 Cursor data) {
   final StringBuilder builder = new StringBuilder();
    builder.append(
     "I know what you are thinking of... \n\n");
   while ( (data.moveToNext())){
       String onYourMind = data.getString(1);
       builder.append("You think of "+
         onYourMind+". ");
       if (data.getInt(2) <= 2){
           builder.append(
            "You are sad about this...");
        }
        if (data.getInt(2) >= 4) {
           builder.append("That makes you happy!");
        }
        builder.append("\n");
    }
    builder.append("\n Well, am I close? ;-)");
    runOnUiThread(new Runnable() {
        @Override
		public void run() {TextView countText = (TextView) 
           findViewById(R.id.main_kpi_count);
          countText.setText(String.valueOf(
           builder.toString()));}});}
```

运行应用程序，看到所有你的想法出现在这里，如下所示：

![如何做...](img/B04299_07_03.jpg)

可怕，不是吗？使用内容提供程序，很容易在不同的应用程序之间共享数据。这就是许多应用程序如联系人或画廊的工作方式。

## 还有更多...

我们已经学习了内容提供程序的工作原理，并且已经偷偷看了一眼观察者模式。使用这个和其他模式可以提高我们应用程序的质量。

现在事情将变得非常严肃。避免潜在错误，减少需要编写的代码量，并使其在任何 Android 设备上运行！我们将在下一章中找出如何做到这一点。

## 另请参阅

+   参考第八章, *提高质量*
