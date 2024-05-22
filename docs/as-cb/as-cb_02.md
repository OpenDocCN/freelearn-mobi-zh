# 第二章：具有基于云的后端的应用程序

本章将教您如何构建一个不需要自己的后端但使用基于云的解决方案的应用程序。

在本章中，您将学习以下食谱：

+   设置 Parse

+   从云端获取数据

+   将数据提交到云端

# 介绍

许多应用程序需要后端解决方案，允许用户与服务器或彼此通信，例如在社交应用程序中，哪个应用程序今天不是社交应用程序呢？您还可以考虑业务应用程序，例如用于物流目的的应用程序。

当然，我们可以编写自己的 API，在某个地方托管它，并编写一些 Android 代码与之通信，包括查询、缓存和应用程序需要支持的所有其他功能。不幸的是，开发所有这些可能是一个非常耗时的过程，而且由于这通常是最有价值的资产，必须有另一种方法来做到这一点。

好消息是，您不必自己做所有这些事情。互联网上有几种现成的移动后端解决方案，例如 QuickBlox、Firebase、Google 应用引擎和 Parse 等，只是其中最知名的几种。

这些解决方案各自擅长特定事情；尽管如此，一个解决方案可能比另一个更合适。例如，以 QuickBlox 为例，它提供了设置事物的最快方式，但需要付出代价。它也不像其他选项那样灵活。Firebase，最近被 Google 收购，是一个非常好的解决方案，特别是如果您需要实时支持；例如，用于聊天应用程序。Parse，被 Facebook 收购，没有实时选项，但更灵活，并且有一些有趣的第三方集成可供选择。

当选择特定解决方案时，当然还有其他考虑因素。提供这种解决方案的各方（Facebook 和 Google）可能可以访问您存储在云中的数据，包括您的用户群，这不一定是坏事，但可能会对您选择的策略产生影响。还要考虑诸如可扩展性和数据锁定等问题，这两者都是奢侈问题，但当您的应用程序变得更受欢迎时，仍然可能成为问题。

Parse 是我最喜欢的，因为它目前是大多数用途最灵活的解决方案。它没有数据锁定（所有数据都可以导出），但它是可扩展的（如果您选择付费计划而不是免费计划），它适用于所有相关的移动平台，甚至允许我们创建云模块（在云中运行的方法，可以定期安排，并且/或者可以被应用程序访问）。在所有可用的热门服务中，这个服务提供了将后端附加到移动应用程序的最简单方法。

### 注意

将来可能会发生变化，特别是对于 Android 开发人员，如果 Google 应用引擎（顺便说一句，也可以用于 iOS 应用程序）与 Android Studio 的集成得到进一步改进。您已经可以在**构建**菜单中找到**部署模块到应用引擎**选项。

# 设置 Parse

想象一个这样的场景：在一个中心点，订单正在被收集并将被准备运输。商品需要被送达，客户收到他们订购的商品后需要在应用程序中签名。每个司机都有一个移动设备和一个应用程序来在数字化过程中支持这个过程。

这是我们将提供接下来的三个食谱的过程，我们将使用 Parse 进行，因为它是我们将要创建的解决方案最合适的后端。

即将介绍的食谱描述了如何设置 Parse，如何从 Parse 中获取数据到您的 Android 应用程序，以及如何发送数据，例如应用程序中的签名，到 Parse。

## 准备就绪

要完成这个食谱，您需要运行 Android Studio 并具有互联网访问权限。就是这样。

## 如何做...

让我们首先创建一个连接到 Parse 后端的应用程序，这样我们就有了一个可以构建应用程序的基础。让我们将应用程序命名为`CloudOrder`。接下来的步骤如下：

1.  启动 Android Studio 并开始一个新的 Android Studio 项目。将应用程序命名为`CloudOrder`，并为**公司域**字段输入`packtpub.com`或适合您或您公司的任何其他名称。然后，单击**下一步**按钮。

1.  选择**手机和平板电脑**选项，可选择更改**最低 SDK**字段。在我的情况下，这将是 API 14（Android 4.x），这在撰写时是尽可能覆盖尽可能多的受众并从我们需要的 SDK 功能中受益的最佳选择。确保至少将目标定位到 API 级别 9，因为 Parse 不支持低于此级别的级别。单击**下一步**按钮继续。

1.  接下来，选择**空白活动**，然后单击**下一步**按钮。在下一页上，只需单击**完成**按钮。Android Studio 将为您设置新项目。

1.  现在，让我们转到[www.parse.com](http://www.parse.com)创建一个新帐户和一个应用程序。使用[www.Parse.com](http://www.Parse.com)注册。输入您的姓名、电子邮件地址和选择的密码，然后单击**注册**按钮。

1.  [www.Parse.com](http://www.Parse.com)上的下一页是**开始**页面。在显示有关您的应用程序名称的字段中输入`CloudOrder`或类似的内容。选择适合您情况的**公司类型**的值，并根据所选择的值完成任何其他字段。完成后，单击**开始使用 Parse**按钮。选择**数据**作为您要开始使用的产品。选择**移动**作为您的环境。接下来，选择一个平台。选择**Android**，在下一个视图中，选择**本机（Java）**选项。

1.  选择**现有项目**选项。我们正在创建一个新项目；但是，为了知道这里发生了什么，我们将自己做以下事情。

1.  现在，下载 SDK。在下载时，切换到 Android Studio，并将项目视图透视从**Android**更改为**项目**。然后，展开`app`文件夹。请注意，其中一个基础文件夹称为`libs`。

1.  将`Parse-x.x.x.jar`文件（其中`x.x.x`表示版本号）拖放到 Android Studio 中的`libs`文件夹中。如果出现**非项目文件访问**对话框，只需单击**确定**按钮。这样做后，`Parse-x.x.x.jar`将出现在`libs`文件夹下。

1.  在第一章中，*欢迎来到 Android Studio*，我们需要告诉 Gradle 关于这个 Parse 库。双击打开`apps`文件夹中的`build.gradle`文件。在依赖项部分，我们需要添加两行，所以它看起来就像下面的例子一样。在已经存在的两行之后，添加`bolts`和`parse`库的依赖项：

```kt
dependencies {
    compile 'com.android.support:appcompat-v7:22.0.0'
    compile 'com.parse.bolts:bolts-android:1.+'
    compile fileTree(dir: 'libs', include: 'Parse-*.jar')
}
```

### 注意

除了通过步骤 6 到 8 中描述的使用本地 JAR 文件，我们还可以使用类似于此的依赖项：

```kt
dependencies {
…
    compile 'com.parse:android:1.8.2'}
```

1.  在`AndroidManifest.xml`文件中，添加访问互联网所需的权限。`Manifest`文件将位于`/app/src/main`文件夹中。双击打开它。按照下面的例子添加访问互联网和访问网络状态的权限。还要为`包名`+`CloudOrderApplication`应用程序定义名称：

```kt
<?xml version="1.0" encoding="utf-8"?>
<manifest 
    package="com.packtpub.cloudorder" >
    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name= "android.permission.ACCESS_NETWORK_STATE" />
<application
    android:name="com.packtpub.cloudorder.CloudOrderApplication"
```

1.  选择并展开`src/main/java/com.packt.cloudorder`文件夹。右键单击此文件夹。在弹出的上下文菜单中，选择**新建**，在子菜单中选择**Java 类**。在显示的对话框中，将`CloudOrderApplication`输入到**名称**字段中。然后，单击**确定**按钮。

1.  使新类成为`Application`类的子类，并重写`onCreate`方法。在`onCreate`方法中，在`super.OnCreate()`之后，添加 Parse 的初始化，如 Parse 使用以下代码所示：

```kt
Parse.initialize(this, "your application Id", "your client Id");
```

1.  Android Studio 还不太满意。您会注意到 Android Studio IDE 中代码中的 Parse 部分被标记为红色。这是因为您的应用程序不知道这个类。每当您更改`gradle`文件时，您的项目都需要进行同步。要这样做，请单击带有工具提示“与 Gradle 文件同步项目”的按钮。您会在导航栏上找到这个按钮。或者，您也可以单击**立即同步**链接。![如何做...](img/B04299_02_01.jpg)

1.  同步之后，Android Studio 将了解 Parse 类，但您仍然需要为此添加一个导入子句。如果您将鼠标悬停在代码中读取`Parse`的部分上，您会注意到 Android Studio 建议这可能是指`com.parse.Parse`。按下*Alt* + *Enter*接受此建议，或者自己添加`import com.parse.Parse`行。最后，您的类将如下所示：

```kt
package com.packt.cloudorder; 
import android.app.Application;
import com.parse.Parse;
public class CloudOrderApplication extends Application{
    @Override
    public void onCreate(){
        super.onCreate();
        Parse.enableLocalDatastore(this);
        Parse.initialize(this, "your application Id", "your client Id");
    }
}
```

1.  我们几乎完成了配置基于 Parse 的应用程序。打开`MainActivity`文件，并将以下行添加到您的`onCreate`方法中：

```kt
ParseObject testObject = new ParseObject("CloudOrder");
testObject.put("customer", "Packt Publishing Ltd");
testObject.saveInBackground();
```

1.  不要忘记添加适当的导入语句。运行您的应用程序。如果一切设置成功，`CloudOrder`类的新对象将被发送到 Parse 并在 Parse 中创建。

1.  在 parse 网页上，点击导航栏顶部的**Core**按钮。查看网页左侧的**Data**部分。**CloudOrder**应该出现在那里，如果您点击它，您将看到包含您刚刚发送的属性（字段）的条目（行）。

这是[www.Parse.com](http://www.Parse.com)上的数据部分的样子：

![如何做...](img/B04299_02_02.jpg)

如果这个测试成功，删除你在`MainActivity`的`onCreate`方法中添加的三行代码，因为我们不再需要它们了。

干得好！你刚刚创建了你的第一个 Parse 应用！让我们继续看看如何扩展`CloudOrder`应用程序！

## 它是如何工作的...

Parse SDK 将负责检索或发送数据。使用`ParseObject`类、`Query`和其他 Parse 类，所有数据通信都将自动进行。

## 还有更多...

在[www.parse.com](http://www.parse.com)上，您将找到有关缓存策略、将数据保存到云端和其他有趣功能的其他信息。

# 从云端获取数据

我们的基于 Parse 的应用程序已经启动运行。现在，让我们看看如何从 Parse 获取订单并在列表中显示它们。

## 准备工作

要完成本教程，您需要先前的教程正在运行，互联网访问以及一些咖啡，尽管我必须承认最后一个不是绝对必要的。茶也可以。

## 如何做...

让我们看看如何通过使用以下步骤从 Parse 后端获取订单并使用列表视图显示它们来扩展我们的`CloudOrder`应用程序：

1.  在*设置 Parse*步骤的最后一步中，我们正在查看新创建的 Parse 实体和其中的数据。实体可以像我们所做的那样在应用程序中即时创建或扩展，但我们也可以在网页上定义列并在这里添加数据。点击**+Col**按钮，将新列添加到`CargoOrder`实体中。

1.  在模态中，显示**添加列**，从**选择类型**中选择**字符串**，并将新列命名为`address`。然后，点击**创建列**按钮。新列将被添加到已经可用的行中（您可能需要向右滚动以查看此内容）。

1.  添加另一列。从类型下拉框中选择**文件**，并将此字段命名为`signature`。最后，添加一个带有**数字**类型和`Status`名称的最后一列。现在，我们为每个**CargoOrder**行添加了三个新的自定义列。

1.  点击**地址**列并输入一个地址；例如，假设订单的送货地址应该是`1600 Amphitheatre Pkwy, Mountain View, CA 94043, United States`（这是谷歌总部的地址，但你当然可以在这里输入任何地址）。

1.  点击**+行**按钮创建一个新的**Cargo Order**行，并为**customer**和**address**字段输入其他值。重复几次以确保我们的应用程序中有一些数据可供使用。

1.  要从**CargoOrder**条目中检索行，我们首先需要创建一个表示订单的模型。在`MainActivity`和`CloudOrderApplication`类所在的位置创建一个新类。右键单击包名，选择**新建**和**Java 类**。命名你的新类为`CloudOrder`，然后点击**确定**按钮。将你的模型设置为`ParseObject`类的子类，并指示该类映射到哪个实体。你的类应该是这样的：

```kt
package com.packt.cloudorder; 
import com.parse.ParseClassName;
import com.parse.ParseObject;
@ParseClassName("CloudOrder")
public class CloudOrder extends ParseObject {...
```

1.  使用以下代码为我们在 Parse 中创建的列添加获取器和设置器：

```kt
public void setCustomer (String value) {
    put("customer", value);
}
public String getCustomer (){
    return getString("customer");
}
public void setAddress (String value) {
    put("address", value);
}
public String getAddress (){
    return getString("address");
}
```

1.  现在，告诉 Parse 关于这个新类。在`CloudOrderApplication`类中，在`Parse.Initialize`行之前添加这一行：

```kt
ParseObject.registerSubclass(CloudOrder.class); 
```

1.  为了在我们的应用程序中获取云订单，我们需要定义一个查询，指示我们究竟在寻找什么。在其最基本的形式中，查询看起来像以下代码片段。将其添加到`MainActivity`的`onCreate`方法中：

```kt
ParseQuery<ParseObject> query = ParseQuery.getQuery("CloudOrder");
```

1.  我们将使用`findInBackground`方法告诉 Parse 我们要异步执行这个查询。添加以下行来实现：

```kt
query.findInBackground(new FindCallback<ParseObject>() {
    public void done(List<ParseObject> items, ParseException e) {
        if (e==null){
            Log.i("TEST", String.format("%d objects found", items.size()));
        }
    }
});
```

1.  运行应用程序并检查**LogCat**（使用快捷键*Cmd* + *6*）。它会显示已找到的对象数量。这应该返回你在[www.parse.com](http://www.parse.com)为**CargoOrder**创建的行数。

1.  太好了！现在，如果我们有一个适配器可以让这些项目在列表视图中可用就好了。创建一个新类，命名为`CloudOrderAdapter`。将其设置为`CloudOrder`类型的数组适配器子类：

```kt
public class CloudOrderAdapter extends ArrayAdapter<CloudOrder> { …
```

1.  实现构造函数，创建一个视图持有者，并为所有需要被重写的方法添加实现。最终，你的适配器将是这样的：

```kt
public class CloudOrderAdapter extends ArrayAdapter<CloudOrder> {
    private Context mContext;
    private int mAdapterResourceId;
    public ArrayList<CloudOrder> mItems = null;
    static class ViewHolder{
        TextView customer;
        TextView address;
    }
    @Override	
    public int getCount(){
        super.getCount();
        int count = mItems !=null ? mItems.size() : 0;
        return count;
    }
    public CloudOrderAdapter (Context context, int adapterResourceId, ArrayList<CloudOrder>items) {
        super(context, adapterResourceId, items);
        this.mItems = items;
        this.mContext = context;
        this.mAdapterResourceId = adapterResourceId;
    }
    @Override
    public View getView(int position, View convertView, ViewGroup parent) {
        View v = null;
        v = convertView;
        if (v == null){
            LayoutInflater vi = (LayoutInflater)this.getContext().getSystemService(Context.LAYOUT_INFLATER_SERVICE);
            v = vi.inflate(mAdapterResourceId, null);
            ViewHolder holder = new ViewHolder();
            holder.customer = (TextView) v.findViewById(R.id.adapter_main_customer);
            holder.address = (TextView)v.findViewById(R.id.adapter_main_address);
            v.setTag(holder);
        }
        final CloudOrder item = mItems.get(position);
        if(item != null){
            final ViewHolder holder = (ViewHolder)v.getTag();
            holder.customer.setText(item.getCustomer());
            holder.address.setText(item.getAddress());
        }
        return v;
    }
}
```

1.  返回`MainActivity`类，并修改查询回调的代码，以便我们可以在那里用结果来填充我们新创建的适配器，如下所示：

```kt
ParseQuery<ParseObject> query = ParseQuery.getQuery("CloudOrder");
query.findInBackground(new FindCallback<ParseObject>(){
    public void done(List<ParseObject> items, ParseException e) {
        Object result = items;
        if (e == null){
            ArrayList<CloudOrder> orders = (ArrayList<CloudOrder>) result;
            Log.i("TEST", String.format("%d objects found", orders.size()));
            CloudOrderAdapter adapter = new CloudOrderAdapter(getApplicationContext(), R.layout.adapter_main, orders);
            ListView listView = (ListView)findViewById(R.id.main_list_orders);
            listView.setAdapter(adapter);;
        }
    }
});
```

1.  为了在我们的应用程序中显示订单，我们必须为其创建一个布局。展开`layout`文件夹，双击`activity_main.xml`文件以打开它。默认情况下，会显示布局的预览。通过点击 Android Studio 底部的**Text**选项卡，将布局显示为 XML。

1.  删除显示`Hello world`的**TextView**小部件，并添加一个列表视图：

```kt
<ListView
android:id="@+id/main_list_orders"
android:layout_width="wrap_content"
android:layout_height="match_parent"/>
```

1.  再次选择`layout`文件夹，右键单击它。从菜单中选择**新建**，然后从子菜单中选择**布局资源**。选择`adapter_main`作为文件名，然后点击**确定**按钮。将创建一个新的布局文件。将视角从设计更改为文本。

1.  在布局中添加两个文本视图，以便我们可以显示客户姓名和地址，并添加一些格式，如下所示：

```kt
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout 
    android:orientation="vertical" android:layout_width="match_parent"
    android:padding="8dp" android:layout_height="match_parent">
    <TextView
        android:text="(Customer)"
        android:textStyle="bold"
        android:textSize="20sp"
        android:textColor="@android:color/black"
        android:id="@+id/adapter_main_customer"
        android:layout_width="match_parent"
        android:layout_height="wrap_content" />
    <TextView
        android:text="(Address)"
        android:textSize="16sp"
        android:textColor="@android:color/darker_gray"
        android:id="@+id/adapter_main_address"
        android:layout_width="match_parent"
        android:layout_height="wrap_content" />
</LinearLayout>
```

1.  你已经完成了。运行你的应用程序。如果一切顺利，你会看到一个输出，就像下面的截图所示，这就是你从[www.parse.com](http://www.parse.com)获取订单后列表视图的样子：![操作步骤...](img/B04299_02_03.jpg)

1.  如果你遇到`class exception error`，再看一下第 8 步。你是否注册了`ParseOrder`子类？如果你遇到其他错误，请仔细重复每一步，检查是否有任何遗漏或不匹配的地方。

## 还有更多...

这个示例只是对 Parse 的简要介绍。在[www.parse.com](http://www.parse.com)上，您将找到更多关于如何从云中检索数据的信息，包括在查询中使用**where**和**order by**语句。它还为您提供了创建关系或复合查询所需的信息。

# 提交数据到云

现在我们已经完成了之前的示例，并且将使用我们的`CloudOrder`应用程序的司机知道去哪里获取特定订单，如果一旦货物交付，司机将能够选择该订单并要求客户在设备上签名。

在这个最后的示例中，我们将实现代码，让客户在设备上签名。签名将作为图像发送到 Parse，并且`CloudOrder`记录将被更新。

## 准备工作

要完成这个示例，您需要先运行之前的示例。

## 如何做…

1.  创建一个新的类，命名为`SignatureActivity`。

1.  创建一个新的布局，命名为`activity_signature.xml`。

1.  切换布局为**文本**。将**TextView**和**Button**小部件添加到布局中。确保布局看起来像这样：

```kt
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout 
    android:orientation="vertical" android:layout_width="match_parent"
    android:padding="8dp" android:layout_height="match_parent">
    <TextView
        android:id="@+id/signature_text"
        android:text=" Please sign here:"
        android:textSize="24sp"
        android:textColor="@android:color/black"
        android:layout_width="match_parent"
        android:layout_height="wrap_content" />
    <Button
        android:id="@+id/signature_button"
        android:text="Send signature"
        android:layout_width="match_parent"
        android:layout_height="wrap_content" />
</LinearLayout>
```

1.  为了让客户签名，我们需要创建一个自定义小部件。

1.  在`com.packt.cloudorder`包的下面，创建一个新包，命名为`widget`。

1.  在这个新包中，创建一个新类，命名为`SignatureView`。

1.  使`SignatureView`类从`View`类继承，并覆盖`onDraw`方法，以在屏幕上放置手指或触控笔时绘制路径。覆盖`onTouch`方法以创建路径。创建路径的代码段如下所示：

```kt
package com.packt.cloudorder.widget;
import android.content.Context;
import android.graphics.Bitmap;
import android.graphics.Canvas;
import android.graphics.Color;
import android.graphics.Paint;
import android.graphics.Path;
import android.graphics.drawable.Drawable;
import android.util.AttributeSet;
import android.view.MotionEvent;
import android.view.View;
public class SignatureView extends View {
    private Paint paint = new Paint();
    private Path path = new Path();
    public SignatureView(Context context, AttributeSet attrs) {
        super(context, attrs);
        paint.setAntiAlias(true);
        paint.setStrokeWidth(3f);
        paint.setColor(Color.BLACK);
        paint.setStyle(Paint.Style.STROKE);
        paint.setStrokeJoin(Paint.Join.ROUND);
    }
    @Override
    protected void onDraw(Canvas canvas) {
        canvas.drawPath(path, paint);
    }
    @Override
    public boolean onTouchEvent(MotionEvent event) {
        float eventX = event.getX();
        float eventY = event.getY();
        switch (event.getAction()) {
            case MotionEvent.ACTION_DOWN:
                path.moveTo(eventX, eventY);
                return true;
            case MotionEvent.ACTION_MOVE:
                path.lineTo(eventX, eventY);
                break;
            case MotionEvent.ACTION_UP: 
                break;
            default:
                return false;
        }
        invalidate();
        return true;
    } 
```

1.  将`getSignatureBitmap`方法添加到`SignatureView`类中，以便我们可以从`Signature view`小部件获取签名作为位图：

```kt
public Bitmap getSignatureBitmap() {
        Bitmap result = Bitmap.createBitmap(getWidth(), getHeight(), Bitmap.Config.ARGB_8888);
        Canvas canvas = new Canvas(result);
        Drawable bgDrawable =getBackground();
        if (bgDrawable!=null) {
            bgDrawable.draw(canvas);
        }else {
            canvas.drawColor(Color.WHITE);
            draw(canvas);
        }
        return result;
    }
} 
```

1.  返回`signature_activity`布局，并在文本视图和按钮之间添加签名视图：

```kt
<com.packt.cloudorder.widget.SignatureView
    android:id="@+id/signature_view"
    android:layout_width="match_parent"
	android:layout_height="200dp"
	android:layout_marginLeft="3dp"
	android:layout_marginTop="3dp"
	android:layout_marginRight="0dp"
	android:layout_marginBottom="18dp"/>
```

1.  构建项目。它应该消除任何渲染问题。

1.  实现`SignatureActivity`类。首先，将其设置为`Activity`的子类，并覆盖`onCreate`方法。将内容视图设置为我们刚刚创建的布局，并在布局中的按钮上添加一个`onClick`实现，如下所示：

```kt
public class SignatureActivity  extends Activity {
    @Override
	protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_signature);
        findViewById(R.id.signature_button).setOnClickListener(new View.OnClickListener(){
            @Override
			public void onClick(View v) {
            }
        });
    }
}
```

1.  在`MainActivity`声明之后，将活动添加到清单文件中，如下所示：

```kt
<activity android:name=".SignatureActivity"/>
```

1.  如果司机选择了任何订单，我们需要显示签名活动，然后需要知道选择了哪个订单。转到`MainActivity`类，并在`OnCreate`方法的末尾，在`Query.findInBackground`调用之后，添加`OnItemClickListener`到列表视图上：

```kt
((ListView)findViewById(R.id.main_list_orders)).setOnItemClickListener(new AdapterView.OnItemClickListener() {
    @Override
	public void onItemClick(AdapterView<?> parent, View view, int position, long id) {        }
});
```

1.  在`onItemClick`事件中，使用以下代码段找出选择了哪个订单：

```kt
ListView listView = (ListView)findViewById(R.id.main_list_orders);
CloudOrder order = (CloudOrder)listView.getAdapter().getItem(position);
gotoSignatureActivity(order);
```

1.  在`gotoSignatureActivity`方法中，我们希望使用意图启动`Signature`活动，并将从`MainActivity`传递到`SignatureActivity`的选择订单，使用如下所示的捆绑：

```kt
private void gotoSignatureActivity(CloudOrder order){
    Intent intent = new Intent(this, SignatureActivity.class);
    Bundle extras = new Bundle();
    extras.putString("orderId", order.getObjectId());
    intent.putExtras(extras);
    this.startActivity(intent);
}
```

1.  在`SignatureActivity`类中，将以下内容添加到按钮的`OnClick`实现中：

```kt
sendSignature();  
```

1.  对于`sendSignature`方法的实现，我们将创建一个新的`ParseFile`对象，并将来自签名视图的位图数据传递给它。我们将使用`saveInBackground`方法将文件发送到 Parse：

```kt
private void sendSignature() {
    final Activity activity = this; 
    SignatureView signatureView = (SignatureView)findViewById(R.id.signature_view); 
    ByteArrayOutputStream stream = new ByteArrayOutputStream();
    signatureView.getSignatureBitmap().compress(Bitmap.CompressFormat.PNG, 100, stream);
    byte[] data = stream.toByteArray();
    final ParseFile file = new ParseFile("signature.jpg", data); 
    file.saveInBackground(new SaveCallback() {
        @Override
		public void done(com.parse.ParseException e) {
        }
    });
}
```

1.  保存完成后，我们希望更新订单的信息，例如我们创建的文件和状态，例如`10`，这可能表示订单已完成或类似的状态。这里实际的值并不重要。

1.  如果在保存过程中没有发生错误，我们使用`ParseObject`类的`createWithoutData`方法，这样我们就可以传递正确的对象 ID 和我们想要更新的字段。我们也会保存这些更改，以便在 Parse 上更新记录。（为了简单起见，我们使用这种方法；尽管我们也可以使用`CloudOrder`对象来完成相同的事情）完成回调的实现如下：

```kt
if (e == null) {
 Bundle extras = getIntent().getExtras();
ParseObject order = ParseObject.createWithoutData("CloudOrder", extras.getString("orderId"));
                order.put("signature", file);
                order.put("status", 10);
order.saveInBackground(new SaveCallback() {
                    @Override
                    public void done(ParseException e) {
                        if (e==null){
                            Toast.makeText(activity, "Signature has been sent!", Toast.LENGTH_SHORT).show();
                        }
                    }
                });
```

1.  运行应用程序，选择一个订单，签名，然后单击**发送签名**按钮。如果一切顺利，将显示一个 toast，指示签名已发送。

这是顾客签名后签名的样子：

![操作步骤…](img/B04299_02_04.jpg)

1.  自己去[www.parse.com](http://www.parse.com)看看。刷新**Cloud order**的视图。注意，在应用程序中选择的订单中，`signature.jpg`文件出现在签名列中。双击它以查看其内容。在向其提交签名图像后，您的数据行可能如下所示：![操作步骤…](img/B04299_02_05.jpg)

实际上，您应该使用字符串资源而不是硬编码的值。通过重用字符串资源（或常量值），不仅可以用于类和字段名称，还可以用于其他文本，从而减少由拼写错误引起的错误数量。这将提高您的应用程序的质量。它也将使以后本地化应用程序变得更加容易。（在最后三章中，我们将更多地关注这些内容，但现在就开始使用这些好的实践。）以下步骤使用了字符串资源：

1.  查看`strings.xml`文件。它位于`res/values`文件夹中。想象一下，如果我们在步骤 19 中显示的 toast 中包含了文本。您的`strings.xml`文件可能如下所示：

```kt
<?xml version="1.0" encoding="utf-8"?>
<resources>
…<string name="app_name">Cloud order</string><string name="parse_class_cargo_order">CargoOrder</string>
    <string name="signature_send">Your signature has been sent.</string>
```

1.  在您的代码中，您可以使用`getString`方法引用字符串资源。例如，您可以用字符串引用替换步骤 19 中显示的 toast 的硬编码字符串，如下所示：

```kt
Toast.makeText(activity, getString(R.string.signature_send), Toast.LENGTH_SHORT).show();
```

1.  在您的布局文件中，您也可以引用这个字符串资源，例如，在一个文本视图中：

```kt
<TextView
    android:text="@string/signature_send"
	android:layout_width="wrap_content"
	android:layout_height="match_parent" />
```

我们将在以后深入介绍如何使用字符串、颜色、尺寸和其他类型的资源，但您可以通过用字符串资源引用替换本教程中的所有硬编码字符串，或在适用的情况下使用常量值来熟悉这些概念。

通过实现这个步骤，我们已经完成了我们的`CloudOrder`应用程序。随意进行进一步的定制，并在需要的地方进行增强。

## 工作原理...

自定义小部件在视图上绘制路径，然后将创建一个位图。使用`ParseFile`对象，位图数据将被发送到 Parse（然后将文件存储在 Amazon 并保留对文件的引用）。

如果成功，我们将更新适用于签名的**CloudOrder**行，指明**signature**列中的图像指向哪个文件。

## 还有更多...

请查看[www.parse.com](http://www.parse.com)上的文档。那里有一些有趣的功能可用，包括`saveEventually`方法和云代码选项。

如果没有可用的互联网连接，`saveEventually`方法将在本地存储更新，这对于移动应用程序是常见的情况。一旦恢复了互联网连接，这个方法将开始发送已排队等待发送到云端的数据。这个选项将为您节省大量麻烦和时间。

还要查看其他功能，比如云代码和各种可用的第三方集成，比如 Twilio，如果您想发送文本或语音消息（这对于入职流程中的确认目的可能很方便），以及 SendGrid，这是一个用于电子邮件传递的工具。

在本章的示例中，我们只需付出少许努力就实现了一些非常有趣的功能，这真的很棒！然而，该应用程序目前还不够吸引人。通过应用下一章将解释的材料设计概念，我们可以使应用程序看起来更加出色，并且更加直观易用。

## 另请参阅

+   有关更多信息，请参阅第三章 *材料设计*。
