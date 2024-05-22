# 第三章。Material Design

这一章将教你什么是 Material Design，为什么它是一个很大的改进，以及为什么你应该在你的应用中使用它。

在这一章中，你将学到：

+   回收视图和卡片视图

+   涟漪和高程

+   出色的过渡

# 介绍

随着 Material Design 的引入，Android 应用的外观终于成熟了。它们可以与 iOS 设计很好地竞争。Android Material 应用具有扁平设计，但也有一些有趣的区别，比如高程。例如考虑下面的图：

![Introduction](img/B04299_03_01.jpg)

把它想象成多张纸片。它是基于，嗯，材料的。每张纸片都有特定的高程。所以，环境实际上是一个有光和阴影等效果的 3D 世界。任何动作都应该具有真实世界的行为，就好像移动的元素是真实的物体一样。动画是 Material Design 的另一个重要元素。

首先看一下[`www.google.co.in/design/spec/material-design/introduction.html`](https://www.google.co.in/design/spec/material-design/introduction.html)来了解 Material Design 的全部内容。当然，对设计师来说有很多有趣的东西，而你可能只对所有这些美丽的东西的实现感兴趣；然而，这个链接为你提供了更多关于 Material Design 的背景信息。

长时间以来，大多数 Android 应用都受到糟糕的设计的困扰，或者在早期根本没有设计。或者，它们看起来与为 iPhone 制作的应用非常相似，包括所有 iOS 典型的元素。

看一下下一个应用的截图：

![Introduction](img/B04299_03_03.jpg)

使用 Material Design，这就是现在大多数谷歌应用的外观。

现在许多谷歌的 Android 应用都使用 Material Design。它们都遵循相同的交互和设计准则。界面是极简主义的，正如人们对谷歌所期望的那样。此外，界面变得更加统一，使得更容易理解和使用。

以前，响应性是你自己要注意的事情。Material Design 带来了涟漪和其他效果，做着同样的事情，即提供用户输入的反馈，但它更容易实现，更加优雅。

至于组件，Material Design 规定了例如特定情况下按钮应该是什么样子。想想用于操作的浮动按钮，或者用于对话框中的扁平按钮。它还用**RecyclerView**替换了**ListView**，这样可以更灵活地显示列表。**CardViews**是常见的元素，你可以经常在谷歌应用中看到它们的使用。各种动画提供了更自然的过渡，比如用于导航或滚动目的的动画。

Material Design 不仅适用于最新和最好的。虽然它随 Android Lollipop（5.0）和更高版本一起发布，但大多数 Material Design 功能可以通过`v7 支持`库在 Android 2.1 及更高版本中使用，这使我们能够应用 Material Design 并仍然支持几乎所有的 Android 设备。

总的来说，Material Design 为你的应用美化提供了很多。人们也想变得更美丽。健康应用因此而蓬勃发展。找出健康的饮食，建议多喝水，以及建议跑步或健身锻炼是这类应用的常见目标。为了展示 Material Design 的美丽，我们将创建一个可以帮助人们变得更健康的应用。

那么，`喝水并自拍`应用怎么样？人们需要更经常地喝水，如果他们这样做，他们就能看到效果。美丽的人们应该有一个美丽的应用。这是有道理的，不是吗？

# 回收视图和卡片视图

`RecyclerView`取代了传统的列表视图。它提供了更多的灵活性，可以以网格形式或水平或垂直项目的形式显示列表的元素。现在，我们可以选择在合适的地方显示卡片，而不是行。

在我们的应用中，每个卡片应该显示有关条目的一些文本和我们拍摄的图片的缩略图。这就是本教程的全部内容。

## 准备工作

要完成本教程，您需要运行 Android。还要确保您已安装了最新的 SDK。（您可以通过打开 SDK 管理器来检查是否安装了最新的 SDK）。为此，打开**工具**菜单，选择**Android**，然后选择**SDK 管理器**选项。

## 如何做...

让我们使用以下步骤来调查如何使用`recycler view`和卡片：

1.  启动 Android Studio 并开始一个新项目。将应用程序命名为`WaterApp`，并在**公司域**字段中输入`packtpub.com`。然后，点击**下一步**按钮。

1.  在下一个对话框中选择**空白活动**，然后点击**下一步**按钮。

1.  在下一个对话框中，点击**完成**按钮。

1.  在`app`文件夹中的`build.gradle`文件中，如下所示，在`dependencies`部分添加`recycler view`的依赖项：

```kt
dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
    compile 'com.android.support:appcompat-v7:22.1.1'
    compile 'com.android.support:recyclerview-v7:+'
}
```

1.  在`build.gradle`文件中将`minSdkVersion`更改为至少`21`。

### 注意

这不一定是最小所需的版本，但由于用于向后兼容目的的支持库不包含所有的 Material 设计功能，我选择在这里选择 API 级别 21，以确保安全。

1.  通过单击`build.gradle`文件编辑后出现的黄色条上的**立即同步**标签来同步您的项目，或者如果没有出现，请单击工具栏上的**同步项目与 Gradle 文件**按钮。

1.  打开`activity_main.xml`布局文件，删除`Hello World TextView`，并向布局中添加一个`RecyclerView`标签，如下所示：

```kt
<android.support.v7.widget.RecyclerView
    android:id="@+id/main_recycler_view"
    android:scrollbars="vertical"
    android:layout_width="match_parent"
    android:layout_height="match_parent"/>
```

1.  在您的`MainActivity`类中，在`setContentView`之后的`onCreate`方法中添加以下内容：

```kt
RecyclerView recyclerView = (RecyclerView) 
 findViewById(R.id.main_recycler_view);
```

1.  `RecyclerView`类还不是一个已知的类。使用*Alt* + *Enter*快捷键添加正确的导入语句，或者自己添加以下行：

```kt
import android.support.v7.widget.RecyclerView;
```

1.  我们将在这个教程中使用线性布局管理器。在第 9 步中添加的行后添加以下行：

```kt
LinearLayoutManager layoutManager = new LinearLayoutManager(this);
recyclerView.setLayoutManager(layoutManager);
```

1.  创建一个新的包并将其命名为`models`，在该包中创建一个新的`Drink`类，如下所示：

```kt
package com.packt.waterapp.models;import java.util.Date;
public class Drink {
    public Date dateAndTime;
    public String comments;
    public String imageUri;
}
```

这里，`Date`类指的是`java.util.Date`包（这是指定的，因为还有一个同名的与 SQL 相关的类）。

1.  让我们创建一个布局来显示这些项目。在项目树中的`layout`包上右键单击，创建一个新的资源文件。为此，从菜单中选择**新建**和**新建布局资源文件**。将其命名为`adapter_main.xml`，然后点击**确定**按钮。

1.  将布局切换到**文本**模式，将`LinearLayout`的方向从`垂直`改为`水平`，为其添加一些填充，并向其添加一个图像视图，如下面的代码片段所示。我们还将添加一个默认图像，以便我们有东西可以查看：

```kt
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout 
android:orientation="horizontal" android:layout_width="match_parent"
android:padding="8dp" android:layout_height="match_parent">
<ImageView android:id="@+id/main_image_view"
android:src="img/ic_menu_camera"
android:scaleType="center"
android:layout_width="90dp"
android:layout_height="90dp" />
</LinearLayout>
```

1.  在图像旁边，我们想要使用两个`TextView`小部件显示日期和时间以及评论，这两个小部件包裹在另一个`LinearLayout`小部件中。在`ImageView`标签之后添加这些：

```kt
<LinearLayoutandroid:orientation="vertical"
    android:layout_width="match_parent"
    android:layout_height="wrap_content">
    <TextView
        android:id="@+id/main_date_time_textview"
		android:layout_marginTop="8dp"
		android:textSize="12sp"
		android:textColor="@color/material_blue_grey_800"
		android:layout_width="match_parent"
		android:layout_height="wrap_content" />
    <TextView
        android:id="@+id/main_comment_textview"
		android:layout_marginTop="16dp"
		android:maxLines="3"
		android:textSize="16sp"
		android:textColor="@color/material_deep_teal_500"
		android:layout_width="match_parent"
		android:layout_height="wrap_content" />
	</LinearLayout>
```

1.  创建另一个包并将其命名为`adapters`。在该包中创建`MainAdapter`类，该类将使用`ViewHolder`类，帮助我们将数据显示在我们想要的位置。我们还包括所有需要被重写的方法，比如`onBindViewHolder`方法和`getItemCount`方法：

```kt
public class MainAdapter extends RecyclerView.Adapter<MainAdapter.ViewHolder> {
    private ArrayList<Drink> mDrinks;private Context mContext;public static class ViewHolder extends        RecyclerView.ViewHolder {
        public TextView mCommentTextView;
        public TextView mDateTimeTextView;
        public ImageView mImageView;
        public ViewHolder(View v) {
            super(v);
        }
    }
    public MainAdapter(Context context, 
      ArrayList<Drink> drinks) {
        mDrinks = drinks;
        mContext = context;
    }
    @Override
    public MainAdapter.ViewHolder  
     onCreateViewHolder(ViewGroup parent,  int viewType) {
        View v = LayoutInflater.from(
         parent.getContext()).inflate(
          R.layout.adapter_main, parent, false);
        ViewHolder viewHolder = new ViewHolder(v);
        viewHolder.mDateTimeTextView =  
         (TextView)v.findViewById(
          R.id.main_date_time_textview);
        viewHolder.mCommentTextView =  
         (TextView)v.findViewById(
          R.id.main_comment_textview);
        viewHolder.mImageView = 
         (ImageView)v.findViewById(
          R.id.main_image_view);
        return viewHolder;
    }
    @Override
    public int getItemCount() {
        return mDrinks.size();
    }
}
```

1.  我们还有更多的事情要做。添加`onBindViewHolder`方法，并添加实现以将数据实际绑定到正确的小部件上：

```kt
@Override
public void onBindViewHolder(ViewHolder holder,int position) {
    Drink currentDrink = mDrinks.get(position);
    holder.mCommentTextView.setText(
     currentDrink.comments);
    holder.mDateTimeTextView.setText(
     currentDrink.dateAndTime.toString());
    if (currentDrink.imageUri != null){
        holder.mImageView.setImageURI(
         Uri.parse(currentDrink.imageUri));
    }
}
```

1.  在`MainActivity`文件中，我们需要有一个适配器的实例和一些要显示的数据。添加一个私有适配器和一个包含`Drink`项目的私有数组列表：

```kt
private MainAdapter mAdapter;private ArrayList<Drink> mDrinks;
```

1.  在`onCreate`方法的末尾，告诉`recyclerView`使用哪个适配器，并告诉适配器使用哪个数据集：

```kt
mAdapter = new MainAdapter(this, mDrinks);
recyclerView.setAdapter(mAdapter);

```

1.  在`MainActivity`文件中，我们想添加一些虚拟数据，以便我们对事情将会是什么样子有一些想法。在我们创建`MainAdapter`类之前的`onCreate`方法中添加以下内容：

```kt
mDrinks = new ArrayList<Drink>();
Drink firstDrink = new Drink();
firstDrink.comments = "I like water with bubbles most of the time...";
firstDrink.dateAndTime = new Date();
mDrinks.add(firstDrink);Drink secondDrink = new Drink();
secondDrink.comments = "I also like water without bubbles. It depends on my mood I guess ;-)";
secondDrink.dateAndTime = new Date();
mDrinks.add(secondDrink);
```

使用*Alt* + *enter*快捷键导入所需的包。

运行您的应用程序以验证到目前为止一切都进行得很顺利。您的应用程序将显示两个包含我们在上一步中创建的示例数据的条目。

### **使用卡片视图**

应用程序看起来还可以，但我不想称其为美丽。让我们看看是否可以稍微改进一下。以下步骤将帮助我们使用卡片视图创建应用程序：

1.  在`app`文件夹中的`build.gradle`文件中添加一个`CardView`依赖项，就在对`recycler view`的依赖项之后：

```kt
compile 'com.android.support:cardview-v7:+'
```

然后再次同步您的项目。

### 注意

顺便说一句，如果这个应用是真的，那么避免不愉快的惊喜，通过指定确切的版本而不是在版本号中使用`+`号来解决任何依赖项可能存在的问题。目前，对于这个特定的依赖项，版本号是`21.0.0`，但在您阅读本文时，可能已经有了新版本。

1.  如果出现错误提示 Gradle 无法解析卡片视图依赖项，则单击**安装存储库并同步项目**链接，接受许可证，并单击**下一步**按钮。等待一段时间，直到下载完成并安装完成。完成后，单击**完成**按钮。再次同步您的项目。

1.  创建一个新的布局并命名为`adapter_main_card_view.xml`。在`LinearLayout`标签中添加一些填充，在`LinearLayout`标签内部添加一个`CardView`：

```kt
<?xml version="1.0" encoding="utf-8"?><LinearLayout 
    android:orientation="vertical"   
    android:layout_width="match_parent"
    android:padding="4dp"  
    android:layout_height="match_parent">
    <android.support.v7.widget.CardView
        xmlns:card_view=
        "http://schemas.android.com/apk/res-auto"android:id="@+id/card_view"
        android:layout_gravity="center"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"card_view:cardCornerRadius="4dp"></android.support.v7.widget.CardView>
</LinearLayout>
```

1.  从先前的布局`adapter_main.xml`文件中，复制`ImageView`和两个`TextView`小部件（但不包括包含这两个`TextView`小部件的`LinearLayout`），并将它们粘贴到您已添加到`adapter_main_card_view.xml`文件中的`CardView`中。

1.  因为`CardView`的行为就像`FrameLayout`，所以您需要为文本标签设置边距。为两个文本视图添加左边距。还修改`TextView`评论的顶部边距：

```kt
<TextView
    android:id="@+id/main_date_time_textview"
	android:layout_marginTop="8dp"
	android:layout_marginLeft="100dp"
	android:textSize="12sp"
	android:textColor="@color/material_blue_grey_800"
	android:layout_width="match_parent"
	android:layout_height="wrap_content" />
<TextView
    android:id="@+id/main_comment_textview"
	android:layout_marginTop="32dp"
	android:layout_marginLeft="100dp"
	android:maxLines="3"
	android:textSize="16sp"
	android:textColor="@color/material_deep_teal_500"
	android:layout_width="match_parent"
	android:layout_height="wrap_content" />
```

1.  现在，通过更改`onCreateViewHolder`方法中的布局 ID，告诉`MainAdapter`类使用这个布局：

```kt
View v = LayoutInflater.from(parent.getContext()). inflate(R.layout.adapter_main_card_view, parent, false);
```

再次运行应用程序，我们将看到这次它会是什么样子：

![使用卡片视图](img/B04299_03_04.jpg)

1.  在下一个教程中，我们将添加一个提升的浮动按钮，并创建一个新的活动，允许我们的应用程序的用户添加饮料、评论和自拍。

## 还有更多...

有很多关于材料设计的文档。浏览各种网站上提供的各种示例，比如[`www.materialup.com`](https://www.materialup.com)，[`materialdesignblog.com`](http://materialdesignblog.com)或[`material-design.tumblr.com`](http://material-design.tumblr.com)。

或者，下载一些在 Play 商店中可用的材料设计应用程序，例如 Inbox、Google+、Wunderlist、Evernote、LocalCast 和 SoundCast 应用程序。

# 涟漪和高程

尽管高程和涟漪并不一定会使人们更加美丽，但将这些和其他材料设计原则应用到我们的应用程序中肯定会有助于美化它。

在上一个教程中，我们创建了一个列表来显示所有已登录的饮料。在这个教程中，我们将添加一个提升的按钮来添加新条目。此外，我们将创建一个新的活动。

对于每个条目，用户可以描述一些关于他喝的东西的想法。当然，用户必须能够每次都自拍，以便以后他可以检查喝那么多水或绿茶（或者啤酒）是否确实对他的健康和外貌产生了积极的影响。

## 准备工作

对于这个指南，如果您已经完成了上一个指南，那将是很好的，因为这将建立在我们以前的成就之上。

## 如何做...

让我们添加一个浮动按钮，并创建一个新的活动来编辑新条目：

1.  在`res/drawable`文件夹中添加一个新的可绘制资源文件，命名为`button_round_teal_bg.xml`，然后点击**OK**按钮。

1.  使用 XML，我们将为按钮创建一个圆形椭圆形状。首先删除选择器标签（如果有）。将其包装在`ripple`标签中。`ripple`在按钮被按下时提供可见反馈；我选择了一种蓝绿色的材料设计变体作为颜色，但您当然可以选择任何您喜欢的颜色。作为灵感，您可以查看[`www.google.com/design/spec/style/color.html`](http://www.google.com/design/spec/style/color.html)。文件的内容如下例所示：

```kt
<ripple android:color="#009789">
    <item>
        <shape android:shape="oval">
            <solid android:color="?android:colorAccent"/>
        </shape>
    </item>
</ripple>
```

### 提示

如果遇到任何错误，请检查`build.gradle`文件中的`minSdkVersion`。有关更多信息，请参考第一条指南的第 5 步。

1.  在`activity_main.xml`布局文件中的循环视图后添加一个按钮：

```kt
<ImageButton
    android:id="@+id/main_button_add"
	android:elevation="1dp"
	android:layout_width="48dp"
    android:layout_height="48dp"
    android:layout_alignParentBottom="true"
    android:layout_alignParentRight="true"
    android:layout_margin="16dp"
    android:tint="@android:color/white"
    android:background="@drawable/button_round_teal_bg"
    android:src="img/ic_input_add"/>
```

### 注意

颜色应该在单独的颜色资源文件中定义。此外，高程和边距应该放在尺寸资源文件中。由于这超出了本指南的范围，我建议您稍后再做这些。

1.  接下来，我们希望有一些阴影，还希望在按钮被按下或释放时改变高程。在`res`文件夹中创建一个新的目录，命名为`anim`。在此文件夹中，创建一个新的动画资源文件。将文件命名为`button_elevation.xml`，然后点击**OK**按钮：

```kt
<selector >
    <item android:state_pressed="true">
        <objectAnimator
            android:propertyName="translationZ"android:duration="@android:integer/config_shortAnimTime"
            android:valueFrom="1dp"
            android:valueTo="4dp"android:valueType="floatType"/></item>
    <item>
        <objectAnimator
            android:propertyName="translationZ"android:duration="@android:integer/config_shortAnimTime"
            android:valueFrom="4dp"
            android:valueTo="1dp"
            android:valueType="floatType"/>
    </item>
</selector>
```

1.  通知图像按钮有关这个新的资源文件。在您的`activity_main.xml`布局中，为图像按钮添加以下行：

```kt
android:stateListAnimator="@anim/button_elevation"
```

1.  在 MainActivity 类的 onCreate 方法末尾，为我们刚刚创建的按钮添加一个`OnClickListener`，并调用`showEntry`方法，我们将在一两分钟内创建：

```kt
findViewById(R.id.main_button_add).setOnClickListener(new  
 View.OnClickListener() {
    @Override
    public void onClick(View v) {
        showEntry();}
});
```

1.  创建一个新的布局资源文件，命名为`activity_entry.xml`，并将`FrameLayout`用作根元素。然后点击**OK**按钮。

1.  为评论添加一个`EditText`小部件，一个拍照按钮和另一个保存条目的按钮。然后将这些元素包装在`CardView`小部件中。在`CardView`小部件之后添加一个`ImageView`小部件，就像这样：

```kt
<?xml version="1.0" encoding="utf-8"?> <FrameLayout xmlns:android= 
 "http://schemas.android.com/apk/res/android"
    android:padding="8dp" android:layout_width="match_parent"   
    android:layout_height="match_parent">
    <android.support.v7.widget.CardView 
        android:id="@+id/card_view"
        android:layout_width="match_parent"
        android:layout_height="200dp"
        card_view:cardCornerRadius="4dp">
    <EditText                                                                                                  android:id="@+id/entry_edit_text_comment"android:lines="6"android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginRight="60dp"/>
    <ImageButton 
	    android:id="@+id/entry_image_button_camera"
        android:src="img/ic_menu_camera"
        android:layout_gravity="right"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content" />
    <Button 
	    android:id="@+id/entry_button_add"
        android:layout_gravity="bottom"
        android:text="Add entry"
        android:layout_width="match_parent"
        android:layout_height="wrap_content" />
    </android.support.v7.widget.CardView>
    <ImageView
        android:id="@+id/entry_image_view_preview"
        android:scaleType="fitCenter"
        android:layout_marginTop="210dp"
        android:layout_width="match_parent"
        android:layout_height="match_parent" />
</FrameLayout>
```

1.  创建一个新的类，命名为`EntryActivity`，然后点击**OK**按钮。

1.  使您的类从`Activity`继承，重写`onCreate`方法，并将内容视图设置为您刚刚创建的布局：

```kt
public class EntryActivity extends Activity {
    @Override
	protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_entry);
    }
}
```

1.  不要忘记在`AndroidManifest.xml`文件中添加新的活动：

```kt
<activity android:name=".EntryActivity"/>
```

1.  在`MainActivity`类中，添加`showEntry`方法和显示新活动所需的实现。我们将在这里使用`startActivityForResult`方法，因为这将允许`EntryActivity`稍后返回数据：

```kt
private int REQUEST_NEW_ENTRY = 1;
private void showEntry(){
    Intent intent = new Intent(this, EntryActivity.class);
    startActivityForResult(intent, REQUEST_NEW_ENTRY);
}
```

现在，如果您运行应用程序并按下按钮，您将注意到视觉反馈。为了正确看到效果，您可能需要使用触控笔或放大按钮的大小。如果您释放按钮，您将看到条目布局。在布局中，如果您按住**添加条目**按钮（或相机按钮），您将注意到涟漪效果。我们不必为此做任何特殊处理。随着 Lollipop 的推出（以及之前的描述），这是按钮的默认行为。但是，这些按钮看起来有点无聊，就像您在浮动按钮中看到的那样，有很多自定义选项可用。让我们按照下一步操作：

1.  在`EntryActivity`类中，为相机按钮设置`OnClickListener`，并对`add`按钮执行相同的操作：

```kt
findViewById(R.id.entry_image_button_camera).setOnClickListener( 
 new View.OnClickListener() {
    @Override
    public void onClick(View v) {
        takePicture();
    }
});
findViewById(R.id.entry_button_add).setOnClickListener(new 
 View.OnClickListener() {
    @Override
    public void onClick(View v) {
    }
}
);
```

1.  添加一个私有成员，用于包含我们将要拍摄的照片的 URI：

```kt
private Uri mUri;
```

1.  创建一个`takePicture`方法并为其添加实现。我们将使用时间戳提前创建一个带有唯一图像名称的文件，并告诉图像捕获意图使用`Uri`来访问该文件：

```kt
private int REQUEST_IMAGE_CAPTURE = 1;
private void takePicture(){
    File  filePhoto = new  
    File(Environment.getExternalStorageDirectory(),String.valueOf(new Date().getTime())+"selfie.jpg");
    mUri = Uri.fromFile(filePhoto);
    Intent intent = new   
     Intent("android.media.action.IMAGE_CAPTURE");
    intent.putExtra(MediaStore.EXTRA_OUTPUT, mUri);
    startActivityForResult(intent, REQUEST_IMAGE_CAPTURE);
}
```

1.  重写`onActivityResult`方法，一旦拍照就会触发。如果一切顺利，我们需要创建刚刚拍摄的文件的位图，并显示其预览：

```kt
@Override
    protected void onActivityResult(int requestCode, int resultCode,Intent data) {
    super.onActivityResult(requestCode, resultCode, data);
    if (requestCode == REQUEST_IMAGE_CAPTURE &&
        resultCode == RESULT_OK){
        Bitmap bitmap = getBitmapFromUri();
        ImageView preview = (ImageView)  
          findViewById(R.id.entry_image_view_preview);
        preview.setImageBitmap(bitmap);}
}
```

1.  接下来，实现`getBitmapFromUri`方法：

```kt
public Bitmap getBitmapFromUri() {
    getContentResolver().notifyChange(mUri, null);
    ContentResolver resolver = getContentResolver();
    Bitmap bitmap;
    try {
        bitmap = android.provider.MediaStore.Images.Media.getBitmap(  
         resolver, mUri);
        return bitmap;
    } 
    catch (Exception e) {
        Toast.makeText(this, e.getMessage(),  
         Toast.LENGTH_SHORT).show();
       return null;}
}
```

1.  在`AndroidManifest.xml`文件中添加适当的权限和功能：

```kt
<uses-permission android:name="android.permission.CAMERA" />
<uses-permission  
  android:name="android.permission.READ_EXTERNAL_STORAGE"/>
<uses-feature android:name="android.hardware.camera" />
```

1.  现在让我们实现`submitEntry`方法。我们将返回评论和图片的`uri`，然后结束活动：

```kt
private void submitEntry(){
    EditText editComment =  (EditText)
      findViewById(R.id.entry_edit_text_comment);
    Intent intent = new Intent();
    intent.putExtra("comments", editComment.getText().toString());
    if (mUri != null) {
        intent.putExtra("uri", "file://" +   
          mUri.getPath().toString());}
    setResult(Activity.RESULT_OK, intent);
    finish();
}
```

1.  为`add`按钮的`onClick`事件添加实现。只需调用`submitEntry`方法：

```kt
findViewById(R.id.entry_button_add).setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View v) {
        submitEntry();
    }
});
```

1.  在`MainActivity`类中，我们将通过重写`onActivityResult`方法来处理返回的结果。将创建一个新的饮料并添加到饮料列表中。最后，我们将通过添加以下片段通知适配器需要显示更新：

```kt
@Override
    protected void onActivityResult(int requestCode, int resultCode, Intent data) {
    super.onActivityResult(requestCode, resultCode, data);
    if (requestCode == REQUEST_NEW_ENTRY && 
        resultCode == RESULT_OK) {
        Bundle bundle = data.getExtras();
        Drink newDrink = new Drink();
        newDrink.comments = bundle.getString("comments");
        newDrink.imageUri = bundle.getString("uri");
        newDrink.dateAndTime = new Date();
        mDrinks.add(newDrink);
        mAdapter.notifyDataSetChanged();
}
```

1.  在`MainAdapter`类中，我们需要做一些工作来显示每个图像的缩略图。将以下内容添加到`onBindViewHolder`方法的末尾：

```kt
if (currentDrink.imageUri != null){
    Bitmap bitmap =    
     getBitmapFromUri(Uri.parse(currentDrink.imageUri));
    holder.mImageView.setImageBitmap(bitmap);
}
```

1.  如果已知项目的`Uri`，我们需要为其显示缩略图。我们将在`MainAdapter`中以稍有不同的方式实现`getBitmapFromUri`。方法如下：

```kt
public Bitmap getBitmapFromUri(Uri uri) {
    mContext.getContentResolver().notifyChange(uri, null);
    ContentResolver cr = mContext.getContentResolver();
    try {
       Bitmap bitmap =   
android.provider.MediaStore.Images.Media.getBitmap(cr, uri);
       return bitmap;
    }
    catch (Exception e) {
        Toast.makeText(mContext, e.getMessage(),  
         Toast.LENGTH_SHORT).show();
        return null;
    }
}
```

现在，运行应用程序。您可以使用真实设备或 Genymotion。如果您使用 Genymotion，您需要启用相机，如第一章*欢迎使用 Android Studio*中所述。单击**添加**按钮，喝一杯水，输入一些评论，然后自拍。点击**添加条目**按钮，使其出现在列表中。

太棒了！您现在已经完成了。该应用程序远非完美，但我们已经做出了一些有趣的举措。美化需要时间。在下一个示例中，我们将通过添加过渡来实现一些令人惊叹的东西。

### 注意

在某些设备上，但不是所有设备，图片可能会被旋转。这是 Android 开发面临的挑战之一，我们将在第六章*捕获和分享*中涵盖这个主题。

## 还有更多…

除了在应用程序的生命周期内，条目列表尚未持久化。如果需要，可以通过将条目存储在 SQLite 数据库中或最终使用 Parse 来使条目持久化，这在第二章*具有基于云的后端的应用程序*中讨论。由于持久性不是本示例的目标，这里不会进一步讨论。在第七章*内容提供程序和观察者*中，将讨论 SQLite 和内容提供程序。

### 注意

自 API 级别 23 以来，有一个可用的 FloatingActionButton 小部件。它有两种大小：默认和迷你。

## 另请参阅

+   第二章*具有基于云的后端的应用程序*

+   第六章*捕获和分享*

+   第七章*内容提供程序和观察者*

# 出色的过渡

如果单击任何卡片，它将再次显示条目视图，其中包括评论和我们之前拍摄的图片的预览。

我们不仅希望从列表视图转到详细视图。Material design 还负责出色的自然过渡。这个示例将应用这一点。

## 准备就绪

要完成这个示例，您需要先运行之前的示例。这个示例将为其添加一些动画。

## 如何做…

以下步骤将帮助我们为应用程序添加动画：

1.  在`MainAdapter`类的`ViewHolder`中添加一个`mDrink`成员：

```kt
public Drink mDrink;
```

1.  在`onBindViewHolder`方法中的同一文件中，在`currentDrink`初始化后，通知`view holder`有关实际饮料的信息：

```kt
Drink currentDrink = mDrinks.get(position);
holder.mDrink = currentDrink;
```

1.  在`onCreateViewHolder`方法中，添加一个`OnClickListener`到末尾：

```kt
v.setTag(viewHolder);
v.setOnClickListener(new View.OnClickListener() {
    @Override
	    public void onClick(View view) {
        ViewHolder holder = (ViewHolder) view.getTag();
        if (view.getId() == holder.itemView.getId()) 
        {
        }
    }
});
```

1.  如果视图被点击，我们希望`EntryActivity`类显示所选的饮料条目。为了能够通知条目有关选择，我们需要将`Drink`模型设为`parcelable`类：

```kt
public class Drink implements Parcelable
```

1.  我们需要实现一些方法：

```kt
@Override
public int describeContents() {
    return 0;
}
@Override
public void writeToParcel(Parcel out, int flags) {
    out.writeLong(dateAndTime.getTime());
    out.writeString(comments);
    out.writeString(imageUri);
}
public static final Parcelable.Creator<Drink> CREATOR = new 
 Parcelable.Creator<Drink>() {
    public Drink createFromParcel(Parcel in) {
        return new Drink(in);
    }
    public Drink[] newArray(int size) {
        return new Drink[size];
    }
};
```

1.  为`Drink`类添加两个构造函数——一个默认的和一个带有 parcel 的，这样我们就可以重新创建对象并用适当的值填充它：

```kt
public Drink(){
}
public Drink(Parcel in) {
    dateAndTime = new Date(in.readLong());
    comments = in.readString();
    imageUri = in.readString();
}
```

1.  在`MainAdapter`类中，添加一个用于请求的私有变量。这种方法使您的代码更易读：

```kt
private int REQUEST_EDIT_ENTRY = 2;
```

### 提示

所谓的魔术数字很容易被误解，应尽量避免使用。这些和其他的示例仅用于演示目的，但在现实世界中，您应尽可能使用自解释的常量。在这里，`REQUEST_EDIT_ENTRY`比在代码中的某个地方只放置数字`2`更有意义。

1.  现在，在`MainAdapter`的`onCreateViewHolder`方法中我们之前创建的`onClick`方法中，我们可以启动一个新的条目活动并将所选的饮料作为参数传递。`onClick`方法的实现现在将如下所示：

```kt
v.setOnClickListener(new View.OnClickListener() {
    @Override
	    public void onClick(View view) {
        ViewHolder holder = (ViewHolder) view.getTag();
        if (view.getId() == holder.itemView.getId()) {
            Intent intent = new Intent(mContext,    
             EntryActivity.class);
            intent.putExtra("edit_drink", holder.mDrink);
    ((Activity)mContext).startActivityForResult(intent,  
              REQUEST_EDIT_ENTRY); }
    }
});
```

1.  在`EntryActivity`类的`onCreate`方法中，我们将检索并显示所选饮料的属性。将此实现添加到方法的末尾：

```kt
Intent intent = getIntent();
if (intent.hasExtra("edit_drink")) {
    Drink editableDrink = intent.getParcelableExtra("edit_drink");
    EditText editComment =    
     (EditText)findViewById(R.id.entry_edit_text_comment);
    editComment.setText(editableDrink.comments);
    if (editableDrink.imageUri != null) {
        mUri = Uri.parse(editableDrink.imageUri);
        Bitmap bitmap = getBitmapFromUri();
        ImageView preview = (ImageView) 
         findViewById(R.id.entry_image_view_preview);
        preview.setImageBitmap(bitmap);
    }
}
```

评论的 EditText 将填充评论，以便用户可以编辑它们。如果饮料条目附有图像，它将显示在预览图像视图中。现在，如果我们有一种简单而酷的方法将图像的缩略图动画到预览中：

1.  惊喜！有。在`res/values`文件夹中的`strings.xml`文件中添加一个新的字符串资源：

```kt
<string name="transition_preview">transition_preview 
  </string>
```

1.  在`MainAdapter`类的`onCreateViewHolder`方法中，在`onClick`实现中，并且在`startActivityForResult`方法之前，我们将使用`ActivityOptionsCompat`类来创建从缩略图（holder 的`mImageView`成员）到条目活动布局中预览图像的过渡：

```kt
ActivityOptionsCompat options =  
 ActivityOptionsCompat.makeSceneTransitionAnimation(
  ((Activity)mContext), holder.mImageView,    
   mContext.getString (R.string.transition_preview));
```

1.  通过用这个实现替换下一行上的`startActivityForResult`调用来提供这些选项：

```kt
ActivityCompat.startActivityForResult(((Activity) mContext),  
 intent, REQUEST_EDIT_ENTRY, options.toBundle());
```

1.  打开`adapter_main_card_view.xml`布局文件，并将此行添加到图像视图（具有`main_image_view`ID 的小部件）：

```kt
android:transitionName="@string/transition_preview"
```

1.  在`activity_entry.xml`布局中，也将此行添加到`ImageView`小部件（具有`entry_image_view_preview`ID 的小部件）。这样 Android 就知道缩略图到更大的预览图像的过渡应该去哪里）。

### 注意

使用字符串资源是一个好的实践。我们可以在这里使用这些资源，以确保我们在代码的任何地方都在谈论相同的过渡，但这也对本地化目的非常有用。

现在，如果您运行您的应用程序并点击`MainActivity`类中的任何卡片，您将看到缩略图被放大并适合于`EntryActivity`类的布局中预览图像的占位符。如果选择返回按钮，则显示反向过渡。在以前的版本中，我们不能只用几行代码来做到这一点！

### 主题

作为奖励，让我们按照以下步骤进行一些主题设置：

1.  访问[`www.materialpalette.com`](http://www.materialpalette.com)并选择两种颜色。主题设置出现了一个颜色集，我们可以将其用作主题，如下截图所示：![Theming](img/B04299_03_06.jpg)

1.  在`res/values`文件夹中创建一个`color.xml`文件，并添加建议的颜色名称和值。我在网站上选择了蓝色和靛蓝色，所以我的颜色资源文件看起来像这样：

```kt
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <color name="primary_dark">#1976d2</color><color name="primary">#2193f3</color>
    <color name="light_primary">#bbdefb</color>
    <color name="text">#ffffff</color>
    <color name="accent">#536dfe</color>
    <color name="primary_text">#212121</color>
    <color name="secondary_text">#727272</color>
    <color name="divider_color">#b6b6b6</color>
</resources>
```

1.  编辑`res/values`文件夹中的`styles.xml`文件，并使其看起来像这样：

```kt
<resources><style name="AppTheme" parent="Theme.AppCompat.Light">
      <item name="android:colorPrimary">@color/primary</item>
      <item name="android:colorPrimaryDark">@color/primary_dark 
      /item>
      <item name="android:colorAccent">@color/accent</item>
      <item name="android:textColor">@color/text</item>
      <item name="android:textColorPrimary">@color/primary_text 
      </item>
     <item name="android:textColorSecondary">
        @color/secondary_text
      </item>
  </style></resources>
```

上述代码的输出如下截图所示：

![Theming](img/B04299_03_07.jpg)

1.  修改您的布局文件并更改文本视图和其他元素，以便它可以反映颜色方案。运行应用程序。

## 它是如何工作的...

Android 的活动转换将处理一切。我们只需要告诉它什么，哪里以及如何。只需几行代码，API 就可以让您在活动之间创建有意义的转换，这将大大改善应用程序的用户体验。

每一步都让你的应用程序看起来越来越好！不幸的是，这就是材料设计介绍的结束。无论你想要在哪里进行改进，都可以随意尝试并享受乐趣！动画，用户体验和布局是高质量应用程序的重要元素。

对于可穿戴应用程序来说，这可能更加重要，正如我们将在下一章中看到的那样。但是，我们如何在如此小的屏幕上实现出色的用户体验呢？

## 还有更多...

我们只看到了材料设计的一些方面。还有很多东西等待我们去发现。

进一步改善应用程序的外观和用户体验，将实现添加到`MainActivity`类中以处理您添加的饮料条目的数据，并在需要时进行增强。或者，您可以查看现有的应用程序，看看如何将它们实现。
