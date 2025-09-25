# 自定义小部件和布局

在 Android 的日常开发中，你会发现核心平台和支持库为你提供了广泛的组件和布局，以构建你的应用程序。互联网上也有大量的开源和第三方小部件。*Android Arsenal*网站([`android-arsenal.com/`](https://android-arsenal.com/))是一个对 Android 可用的 API 进行了良好分类的列表，当你需要平台或支持库中不可用的功能时，它是一个极好的起点。即使有如此丰富的可用小部件和库，有时你也会发现自己想要一个尚未构建的小部件。

在任何平台上创建自己的小部件是一项庞大的任务。小部件需要能够使用图形原语（如线条、弧线、圆形和多边形）渲染自己，以尽可能看起来像原生应用。许多 Android 小部件（如`Button`）通过使用优秀的`Drawable`类和资源来避免这样做。这使得你只需通过更改小部件使用的可绘制资源（如你在第二章，*设计表单屏幕*）中使用的状态可绘制资源，就可以简单地自定义小部件的外观。

在本章中，我们将探讨如何构建自定义小部件和布局组件。我们将探讨在构建自己的`View`实现时应该遵循的最佳实践，以及如何使用 Android 图形 API 渲染 2D 图形。具体来说，我们将探讨以下内容：

+   创建一个完全自定义的`View`类

+   使用图形原语渲染 2D 图形

+   如何创建一个自定义的`ViewGroup`以产生自定义布局效果

+   使用`Drawable`对象渲染动画

+   创建能够自我动画的`View`类

# 创建自定义视图实现

有时候，现有的小部件无论你如何自定义都不够用。有时候，你需要显示平台不支持的东西。在这些情况下，你可能需要实现自己的自定义小部件。`View`类可以轻松扩展以产生许多不同的效果，但在你着手之前，有一些事情是值得了解的：

+   预期`View`的渲染将在`onDraw`方法中发生。

+   当渲染`View`的图形时，你将使用`Canvas`来发送绘图指令。

+   每个`View`负责计算其填充的偏移量，并且默认情况下，图形将被裁剪到这些维度。

+   你应该避免在`onDraw`方法中进行任何对象分配（包括数组，如果可能的话）。`onDraw`方法可能是任何应用程序中最时间敏感的方法调用，需要尽可能产生最少的垃圾。任何对象分配都应该在其他方法中完成，并在`onDraw`实现中仅使用。

在旅行报销示例中，如果用户能看到他们过去几天支出的简单概述图，那将非常棒。为此，我们需要编写一个能够为他们绘制此图的类。能够使用布局 XML 文件更改一些`View`属性（特别是线形图的尺寸和颜色）很有用。为此，你需要指定属性名称及其类型信息，以便布局资源编译器使用。按照以下说明编写简单的线形图`View`实现：

1.  在旅行报销应用中的`res/values`资源目录上右键单击，然后选择“新建”|“值资源文件”。

1.  将新文件命名为`attrs_spending_graph_view`。

1.  点击“确定”以创建新的资源文件。

1.  你将使用此文件声明一些新的 XML 属性，供资源编译器使用，这些属性可以在处理你的新图`View`类时用在布局 XML 文件中。这些 XML 属性提供了类型信息（以`format`属性的形式），这会影响资源编译器在布局 XML 中处理它们的方式：

```kt
<resources>
  <declare-styleable name="SpendingGraphView">
    <attr name="strokeColor" format="color" />
    <attr name="strokeWidth" format="dimension" />
  </declare-styleable>
</resources>
```

1.  现在，在组件包上右键单击，然后选择“新建”|“Java 类”。

1.  将新类命名为`SpendingGraphView`。

1.  将`Superclass`更改为`android.view.View`。

1.  点击“确定”以创建新的类。

1.  在`SpendingGraphView`中声明变量以保存可以在布局 XML 文件中指定的值。这些变量通常应反映 XML 文件中使用的名称，并且应使用合理的默认值初始化：

```kt
private int strokeColor = Color.GREEN;
private int strokeWidth = 2;
```

1.  接下来，声明一个数组以将数据点渲染到图中。在这个实现中，我们假设每个数据点是某个未指定日期的花费金额：

```kt
private double[] spendingPerDay;
```

1.  如前所述，`onDraw`实现应尽可能少做工作。在这个图形实现中，这意味着整个图形实际上是提前计算好的，并缓存到局部变量中，以便在`onDraw`方法中绘制。Android 图形 API 提供了一个`Path`类，用于定义任何抽象的连接线组，以及一个`Paint`类，用于定义颜色、笔触大小（笔）、填充样式等。你需要声明一个`Path`和一个`Paint`，以便计算和渲染：

```kt
private Path path = null;
private Paint paint = null;
```

在类的字段中存储小部件的渲染状态似乎与您所知道的状态存储和传递的位置相矛盾，但小部件是一种状态容器。其任务是向用户展示其状态并捕获事件以触发状态变化。将图形原语的建设从`onDraw`实现中排除，意味着图形管道不会被每帧从图形数据重新计算图形状态而减慢。

1.  现在，实现一个`View`类的标准构造函数。你希望所有这些构造函数都调用一个单独的`init()`方法来处理小部件的实际初始化，在这种情况下，还需要从布局 XML 中获取并读取属性：

```kt
public SpendingGraphView(final Context context) {
  super(context);
  init(null, 0);
}

public SpendingGraphView(
    final Context context,
    final AttributeSet attrs) {
  super(context, attrs);
  init(attrs, 0);
}

public SpendingGraphView(
    final Context context,
    final AttributeSet attrs,
    final int defStyle) {
  super(context, attrs, defStyle);
  init(attrs, defStyle);
}
```

1.  现在，实现 `init` 方法并使用 `Context` 将 `AttributeSet` 对象及其数据转换为 `TypedArray` 对象。这是从应用程序的当前 `Theme` 中合并所有样式信息的地方。当你完成一个 `TypedArray` 后，你需要回收它们，将它们交还给平台以供重用。这有助于 `obtainStyledAttributes` 方法的性能：

```kt
private void init(final AttributeSet attrs, final int defStyle) {
  final TypedArray a = getContext().obtainStyledAttributes(
          attrs, R.styleable.SpendingGraphView, defStyle, 0);

  strokeColor = a.getColor(
          R.styleable.SpendingGraphView_strokeColor,
          strokeColor);

  strokeWidth = a.getDimensionPixelSize(
          R.styleable.SpendingGraphView_strokeWidth,
          strokeWidth
  );

  a.recycle();
}
```

1.  为了正确绘制图表，你需要一个实用方法来帮助找到垂直轴的刻度。这涉及到找到图表将具有的最大值，不幸的是，Android 平台没有直接提供方法来做这件事，所以你需要自己实现它：

```kt
protected static double getMaximum(final double[] numbers) {
  double max = 0;

  for (final double n : numbers) {
    max = Math.max(max, n);
  }

  return max;
}
```

1.  下一步是实现图表数据的实际渲染方法。此方法将在主线程上调用，但不会作为渲染循环的一部分调用。相反，你将计算所有值并使用 `Path` 对象（矢量图形原语）绘制图表。然后，此方法将存储绘制的线条和要用于路径和绘制字段的 `Paint`，并发出信号，表示 `View` 是 *无效的*，需要尽快调用其 `onDraw` 方法：

```kt
protected void invalidateGraph() {
  if (spendingPerDay == null || spendingPerDay.length <= 1) {
    path = null;
    paint = null;
    invalidate();

    return;
  }

  final int paddingLeft = getPaddingLeft();
  final int paddingTop = getPaddingTop();
  final int paddingRight = getPaddingRight();
  final int paddingBottom = getPaddingBottom();

  final int contentWidth =
        getWidth() - paddingLeft - paddingRight;
  final int contentHeight =
        getHeight() - paddingTop - paddingBottom;
  final int graphHeight =
        contentHeight - strokeWidth * 2;

  final double graphMaximum = getMaximum(spendingPerDay);

  final double stepSize = (double) contentWidth / (double) (spendingPerDay.length - 1);
  final double scale = (double) graphHeight / graphMaximum;

  path = new Path();
  path.moveTo(paddingLeft, paddingTop);

  paint = new Paint();
  paint.setStrokeWidth(strokeWidth);
  paint.setColor(strokeColor);
  paint.setFlags(Paint.ANTI_ALIAS_FLAG);
  paint.setStyle(Paint.Style.STROKE);

  path.moveTo(
      paddingLeft,
      contentHeight - (float) (scale * spendingPerDay[0]));

  for (int i = 1; i < spendingPerDay.length; i++) {
    path.lineTo(
          (float) (i * stepSize) + paddingLeft,
          contentHeight - (float) (scale * spendingPerDay[i]));
  }

  invalidate();
}
```

实际上，将此代码封装在 `ActionCommand` 或 `AsyncTask` 中是完全可能的，这样这些计算就不会阻塞主线程。你需要在 `onForeground()` 中调用 `invalidate()` 方法，或者使用 `postInvalidate()` 方法代替（它将 `invalidate()` 信号发送到主线程）。如果图表预期展示的数据量非常大，将这种复杂性移动到后台线程是良好的实践。

1.  现在，你已经准备好覆盖 `onDraw` 方法，并在平台提供的 `Canvas` 上实际绘制图表。这个 `onDraw` 实现只是简单地验证图表是否已渲染，然后将字段绘制到屏幕上：

```kt
@Override
protected void onDraw(final Canvas canvas) {
  if (path == null || paint == null) {
    return;
  }

  canvas.drawPath(path, paint);
}
```

有用的事实是，你可以通过让它在一个离屏的 `Bitmap` 对象上绘制来自己构建一个 `Canvas`，这样你就可以通过调用它们的 `onDraw` 方法来捕获 *屏幕截图*。

1.  现在，你只需要几个获取器和设置器方法，以便让应用程序指定要渲染的数据，以及一个程序化的方式来更改和获取 XML 属性值。设置器方法还需要调用 `invalidateGraph()` 方法，以使数据被重新计算并渲染：

```kt
public void setSpendingPerDay(final double[] spendingPerDay) {
  this.spendingPerDay = spendingPerDay;
  invalidateGraph();
}

public int getStrokeColor() {
  return strokeColor;
}

public void setStrokeColor(final int strokeColor) {
  this.strokeColor = strokeColor;
  invalidateGraph();
}

public int getStrokeWidth() {
  return strokeWidth;
}

public void setStrokeWidth(final int strokeWidth) {
  this.strokeWidth = strokeWidth;
  invalidateGraph();
}
```

`SpendingGraphView` 需要通过 `setSpendingPerDay` 方法以编程方式传递其实际数据。幸运的是，这可以通过数据绑定系统轻松完成，该系统还会在数据更改时保持数据更新。

# 集成 SpendingGraphView

将 `SpendingGraphView` 集成到应用程序中就像在布局 XML 文件中声明它一样简单，并给它提供一些数据点以进行渲染：

```kt
<com.packtpub.claim.widget.SpendingGraphView
    android:id="@+id/spendingGraphView"
    android:layout_width="match_parent"
    android:layout_height="?attr/actionBarSize"
    app:strokeWidth="2dp"
    app:strokeColor="@color/colorAccent"
 app:spendingPerDay="@{dataSource.spendingPerDay}"/>
```

您还可以使用 `findViewById` 编程查找 `SpendingGraphView`，并从您的 Java 代码中调用 `setSpendingPerDay` 方法。将 `SpendingGraphView` 集成到旅行索赔示例中稍微复杂一些。该图表应位于概览屏幕上，因为它可以快速直观地显示用户过去几天的消费情况。如果用户开始滚动，图表需要从屏幕上滚动出去，以便为索赔项腾出更多屏幕空间。一个不错的方法是利用您编写的 `DisplayItem` 类来处理间隔，并在概览的开始处添加一个。让我们将新的 `SpendingGraphView` 集成到概览屏幕中：

1.  右键单击 `res/layout` 目录，然后选择“新建”|“布局资源文件”。

1.  将新资源文件命名为 `card_spending_graph`。

1.  将根元素更改为 `layout`。

1.  点击“确定”以创建新的资源文件。

1.  此布局资源将与 `DataBoundViewHolder` 一起使用，我们将间接通过 `item` 变量传递每日消费。还值得注意的是，您的 `SpendingGraphView` 上的自定义属性（`strokeColor` 和 `strokeWidth`）的 XML 命名空间是 `app` 命名空间。布局资源应该看起来像这样：

```kt
<?xml version="1.0" encoding="utf-8"?>
<layout>
    <data>
        <variable
            name="item"
            type="double[]" />
    </data>

    <android.support.v7.widget.CardView

        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        app:contentPadding="@dimen/grid_spacer1">

        <com.packtpub.claim.widget.SpendingGraphView
            android:id="@+id/spendingGraphView"
            android:layout_width="match_parent"
            android:layout_height="@dimen/spending_graph_height"
            app:spendingPerDay="@{item}"
            app:strokeColor="@color/colorAccent"
            app:strokeWidth="2dp" />
    </android.support.v7.widget.CardView>
</layout>
```

1.  使用 `layout_height` 属性上的代码助手，在您的 `dimens.xml` 值资源文件中创建一个名为 `spending_graph_height` 的新维度值（正如刚刚所强调的）：

```kt
    <dimen name="app_bar_height">180dp</dimen>
    <dimen name="spending_graph_height">80dp</dimen>
</resources>
```

1.  打开 `ClaimItemAdapter` 源文件。

1.  大多数更改将在 `CreateDisplayListCommand` 内部类中。您需要计算用户最近几天内的消费，为此，您需要知道每个索赔距离今天有多少天，以便可以将其金额添加到正确的日期。此方法简单地逐日倒退，直到达到给定的时间戳：

```kt
int countDays(final Date timestamp) {
  final Calendar calendar = Calendar.getInstance();
  calendar.setTime(timestamp);

  final Calendar counterCalendar = Calendar.getInstance();
  counterCalendar.clear(Calendar.HOUR_OF_DAY);
  counterCalendar.clear(Calendar.MINUTE);
  counterCalendar.clear(Calendar.SECOND);
  counterCalendar.clear(Calendar.MILLISECOND);

  int days = 0;
  while (calendar.before(counterCalendar)) {
    days++;
    counterCalendar.add(Calendar.DAY_OF_YEAR, -1);
  }

  return days;
}
```

时间 API，如 JODA 时间 ([`www.joda.org/joda-time/`](http://www.joda.org/joda-time/)) 和 Java 8 时间 API，提供了专门用于计算两个时间点之间差异（以各种不同的时间单位）的方法。然而，这些 API 的使用超出了本书的范围。

1.  接下来，您需要在 `CreateDisplayListCommand` 中添加另一个方法来创建表示用户过去几天消费的金额数组。为了使实现简单快捷，我们默认将其限制为十天。`getSpendingPerDay` 方法为这些天中的每一天创建一个双精度浮点数，并将每天的 `ClaimItem` 对象的金额添加到每个双精度浮点数中：

```kt
double[] getSpendingPerDay(final List<ClaimItem> claimItems) {
  final double[] daysSpending = new double[10];
  final int lastItem = daysSpending.length - 1;
  Arrays.fill(daysSpending, 0);

  for (final ClaimItem item : claimItems) {
    final int distance = countDays(item.getTimestamp());

    // the ClaimItems are in timestamp order
    if (distance > lastItem) {
      break;
    }

    daysSpending[lastItem - distance] += item.getAmount();
  }

  return daysSpending;
}
```

1.  在 `CreateDisplayListCommand` 中要做的最后一件事是，在列表中创建一个 `DisplayItem` 作为第一个条目：

```kt
public List<DisplayItem> onBackground(
    final List<ClaimItem> claimItems)
    throws Exception {

  final List<DisplayItem> output = new ArrayList<>();
  output.add(new DisplayItem(
 R.layout.card_spending_graph,
 getSpendingPerDay(claimItems)));

  for (int i = 0; i < claimItems.size(); i++) {
```

1.  你还需要向 `UpdateDisplayListCommand` 内部类添加一些新代码，因为它不知道如何比较 `DiffUtil` 的支出图。在 `DiffUtil.Callback` 中 `areItemsTheSame` 方法的实现中，你可以将 `card_spending_graph` 布局资源与分隔符完全相同对待，因为在列表中只有一个：

```kt
@Override
public boolean areItemsTheSame(
    final int oldItemPosition,
    final int newItemPosition) {
  // ...

  switch (newItem.layout) {
    case R.layout.card_claim_item:
        final ClaimItem oldClaimItem = (ClaimItem) oldItem.value;
        final ClaimItem newClaimItem = (ClaimItem) newItem.value;
        return oldClaimItem != null
             && newClaimItem != null
             && oldClaimItem.id == newClaimItem.id;
    case R.layout.widget_divider:
    case R.layout.card_spending_graph:
        return true;
  }

  return false;
}
```

1.  然而，你还需要 `DiffUtil` 来检测图数据可能已更改。在这种情况下，我们简单地假设数据已更改，并强制 `RecyclerView` 将新的数据点绑定到现有的 `SpendingGraphView`：

```kt
@Override
public boolean areContentsTheSame(
    final int oldItemPosition,
    final int newItemPosition) {
  final DisplayItem oldItem = oldDisplay.get(oldItemPosition);
  final DisplayItem newItem = newDisplay.get(newItemPosition);

  switch (newItem.layout) {
    case R.layout.card_claim_item:
        final ClaimItem oldClaimItem = (ClaimItem) oldItem.value;
        final ClaimItem newClaimItem = (ClaimItem) newItem.value;
        return oldClaimItem != null
             && newClaimItem != null
             && oldClaimItem.equals(newClaimItem);
    case R.layout.widget_divider:
        return true;
    case R.layout.card_spending_graph:
 return false;
  }

  return false;
}
```

1.  最后要确保的是用户不能通过滑动 `SpendingGraphView` 来从 `RecyclerView` 中删除它，因为这将对用户来说是一个巨大的惊喜。打开 `OverviewActivity` 源文件并定位到 `SwipeToDeleteCallback` 内部类。

1.  我们需要通知 `ItemTouchHelper` 列表中的第一个项目不能滑动或移动。我们通过重写默认的 `getMovementFlags` 方法来实现这一点。此方法通常只返回传递给构造函数的标志，但现在你希望这些标志仅对单个项目进行更改：

```kt
@Override
public int getMovementFlags(
    final RecyclerView recyclerView,
    final RecyclerView.ViewHolder viewHolder) {

  if (viewHolder.getAdapterPosition() == 0) {
    return 0;
  }

  return super.getMovementFlags(recyclerView, viewHolder);
}
```

# 创建布局实现

在大多数应用程序中，你会发现 `ConstraintLayout`、`CoordinatorLayout` 以及一些更原始的布局类（如 `LinearLayout` 和 `FrameLayout`）的组合已经足够满足你为用户界面设计的任何布局需求。然而，偶尔你也会发现自己需要自定义布局管理器来实现应用程序所需的特定效果。

布局类从 `ViewGroup` 类扩展，它们的工作是告诉它们的子小部件它们应该放置的位置以及它们应该有多大。它们通过两个阶段来完成这项工作：测量阶段和布局阶段。

所有 `View` 实现都应按照规范提供其实际大小的测量值。这些测量值随后被 `View` 小部件的父 `ViewGroup` 用于分配小部件在屏幕上所占用的空间。例如，一个 `View` 可能被指示最多消耗屏幕宽度。然后 `View` 必须确定它实际需要多少空间，并将该尺寸记录在其 **测量尺寸** 中。测量尺寸随后在布局过程中被父 `ViewGroup` 使用。

第二个阶段是布局阶段，由每个 `View` 小部件的父 `ViewGroup` 执行。此阶段将 `View` 定位在屏幕上，相对于其父 `ViewGroup` 的位置，并指定小部件将在屏幕上消耗的实际大小（通常基于测量阶段计算出的测量大小）。

当你实现自己的 `ViewGroup` 时，你需要确保在执行实际的布局操作之前，所有子 `View` 小部件都有机会测量自己。

让我们构建一个布局类，以使其子项呈圆形排列。为了保持实现简单，我们假设所有子小部件具有相同的大小（例如，如果它们都是图标）：

1.  在旅行报销示例应用中的 `widget` 包上右键单击，然后选择 New| Java Class。

1.  将新类命名为 `CircleLayout`。

1.  将超类更改为 `android.view.ViewGroup`。

1.  点击 OK 创建新类。

1.  声明标准的 `ViewGroup` 构造函数：

```kt
public CircleLayout(final Context context) {
  super(context);
}

public CircleLayout(
    final Context context,
    final AttributeSet attrs) {
  super(context, attrs);
}

public CircleLayout(
      final Context context,
      final AttributeSet attrs,
      final int defStyleAttr) {

  super(context, attrs, defStyleAttr);
}
```

1.  重写 `onMeasure` 方法以计算 `CircleLayout` 及其所有子 `View` 小部件的大小。测量规范以 `int` 值的形式传入，这些值使用 `MeaureSpec` 类中的 `static` 方法进行解释。测量规范有两种类型：*最多* 和 *正好*，每种类型都附加一个 *大小* 值。在这个特定的布局中，我们始终将 `CircleLayout` 测量为其规范中给出的大小。这意味着 `CircleLayout` 将始终消耗可用的最大空间。它还期望所有子项能够指定大小，而无需 `match_parent` 属性（因为这会导致每个子项占用所有可用空间）：

```kt
@Override
protected void onMeasure(
    final int widthMeasureSpec,
    final int heightMeasureSpec) {
  super.onMeasure(widthMeasureSpec, heightMeasureSpec);
  measureChildren(widthMeasureSpec, heightMeasureSpec);

  setMeasuredDimension(
        MeasureSpec.getSize(widthMeasureSpec),
        MeasureSpec.getSize(heightMeasureSpec));
}
```

1.  下一个要实现的方法是 `onLayout` 方法。这个方法负责在 `CircleLayout` 中对子 `View` 小部件进行实际排列，通过调用它们的 `layout` 方法。`layout` 方法不应该被重写，因为它与平台紧密相关，并执行多个其他重要操作（例如通知布局监听器）。相反，你应该重写 `onLayout`，但调用 `layout.CircleLayout` 假设所有子 `View` 小部件具有相同的大小（并在 `onLayout` 实现中强制执行这一点）。这个 `onLayout` 方法只是计算可用空间，然后将子 `View` 小部件定位在边缘周围的一个圆圈中：

```kt
protected void onLayout(
    final boolean changed,
    final int left,
    final int top,
    final int right,
    final int bottom) {

  final int childCount = getChildCount();

  if (childCount == 0) {
    return;
  }

  final int width = right - left;
  final int height = bottom - top;

  // if we have children, we assume they're all the same size
  final int childrenWidth = getChildAt(0).getMeasuredWidth();
  final int childrenHeight = getChildAt(0).getMeasuredHeight();

  final int boxSize = Math.min(
      width - childrenWidth,
      height - childrenHeight);

  for (int i = 0; i < childCount; i++) {
    final View child = getChildAt(i);
    final int childWidth = child.getMeasuredWidth();
    final int childHeight = child.getMeasuredHeight();

    final double x = Math.sin((Math.PI * 2.0)
          * ((double) i / (double) childCount));
    final double y = -Math.cos((Math.PI * 2.0)
          * ((double) i / (double) childCount));

    final int childLeft = (int) (x * (boxSize / 2))
          + (width / 2) - (childWidth / 2);
    final int childTop = (int) (y * (boxSize / 2))
          + (height / 2) - (childHeight / 2);
    final int childRight = childLeft + childWidth;
    final int childBottom = childTop + childHeight;

    child.layout(childLeft, childTop, childRight, childBottom);
  }
}
```

虽然 `onLayout` 方法的实现相当长，但它也很简单。大部分代码都关注于确定子 `View` 小部件的期望位置。布局代码需要尽可能快地执行，并且在 `onMeasure` 和 `onLayout` 方法中应避免分配任何对象（类似于 `onDraw` 的规则）。从性能角度来看，布局是构建屏幕的关键部分，因为没有完成布局，实际上无法进行渲染。每次布局结构发生变化时，布局都会重新运行。例如，如果你添加或删除任何子 `View` 小部件，或者更改 `ViewGroup` 的大小或位置。如果你使用 `CoordinatorLayout`，其中 `ViewGroup` 正在折叠（或者如果你将其大小作为属性动画的一部分进行更改），则 `ViewGroup` 的大小可能会在每一帧中更改。

# 创建动画视图

大多数小部件动画可以使用 Android 中的动画 API 来处理。标准的动画 API 旨在处理具有定义的开始和结束的动画，或者形成简单循环的动画。然而，有些动画不适合这种模式；一个很好的例子就是游戏。游戏有许多连续运行的动画，你甚至可以将整个游戏屏幕视为一个单一的、连续的动画。

有许多小部件需要连续动画化，而你标准的 Android 动画 API 将不起作用。在这些情况下，你需要一个可以连续动画并更新自己的`View`，只要它对用户可见。在这些情况下，需要稍微不同的设计，因为小部件将始终在变化。

为了说明如何编写一个具有连续动画的小部件，让我们编写一个`View`类，该类将动画化一些弹跳的`Drawable`对象。每个`Drawable`将被单独跟踪，当它到达一边时，它将“弹跳”并朝相反方向前进。这个类与旅行索赔示例代码无关，所以如果你喜欢，可以将其添加到新项目中。按照以下步骤编写`BouncingDrawablesView`：

1.  在你的默认包中，选择“新建| Java 类”。

1.  将类命名为`widget.BouncingDrawablesView`。

1.  将父类设置为`android.view.View`。

1.  点击“确定”以创建新的班级。

1.  场景中将有若干弹跳对象，你需要跟踪它们的位置和速度向量。为此，你希望将每个弹跳的`Drawable`封装在`Bouncer`对象中；我们将将其编写为内部类：

```kt
public static class Bouncer {
  final Drawable drawable;
  final Rect bounds;
  int speedX;
  int speedY;

  public Bouncer(
        final Drawable drawable,
        final int speedX,
        final int speedY) {

    this.drawable = drawable;
    this.bounds = drawable.copyBounds();
    this.speedX = speedX;
    this.speedY = speedY;
  }
```

1.  在`Bouncer`内部类中接下来要做的事情是创建一个单独的`step`方法，该方法将为下一个要渲染的动画帧设置`Bouncer`。此方法将接受一个参数，表示它正在渲染的*字段*的边界。如果下一个位置与字段的任何边缘发生碰撞，`Bouncer`将避免越过边缘，并在可能碰撞的轴上反转方向：

```kt
void step(final Rect boundary) {
  final int width = bounds.width();
  final int height = bounds.height();

  int nextLeft = bounds.left + speedX;
  int nextTop = bounds.top + speedY;

  if (nextLeft + width >= boundary.right) {
    speedX = -speedX;
    nextLeft = boundary.right - width;
  } else if (nextLeft < boundary.left) {
    speedX = -speedX;
    nextLeft = boundary.left;
  }

  if (nextTop + height >= boundary.bottom) {
    speedY = -speedY;
    nextTop = boundary.bottom - height;
  } else if (nextTop < boundary.top) {
    speedY = -speedY;
    nextTop = boundary.top;
  }

  bounds.set(
       nextLeft,
       nextTop,
       nextLeft + width,
       nextTop + height
  );
}
```

1.  `Bouncer`类还需要一个方便的绘制方法，该方法将在将`Drawable`渲染到给定的`Canvas`对象之前更新`Drawable`的边界。`Bouncer`跟踪自己的边界，因此所有`Bouncer`实例实际上可以共享同一个`Drawable`实例，只需在不同的位置在字段上绘制它即可：

```kt
  void draw(final Canvas canvas) {
    drawable.setBounds(bounds);
    drawable.draw(canvas);
  }
} // end of Bouncer inner class
```

1.  现在，在`BouncingDrawablesView`中，声明一个`Bouncer`对象的数组，这些对象将被`View`实现包含和动画化：

```kt
private Bouncer[] bouncers = null;
```

1.  `BouncingDrawableView`还需要一个状态字段来跟踪它是否应该进行动画：

```kt
private boolean running = false;
```

1.  接下来，声明标准的`View`实现构造函数：

```kt
public BouncingDrawablesView(
    final Context context) {
  super(context);
}

public BouncingDrawablesView(
    final Context context,
    final AttributeSet attrs) {
  super(context, attrs);
}

public BouncingDrawablesView(
    final Context context,
    final AttributeSet attrs,
    final int defStyleAttr) {
  super(context, attrs, defStyleAttr);
}
```

1.  通过简单地告诉每个`Bouncer`对象绘制自己来实现`onDraw`方法：

```kt
@Override
protected void onDraw(final Canvas canvas) {
  super.onDraw(canvas);

  if (bouncers == null) {
    return;
  }

  for (final Bouncer bouncer : bouncers) {
    bouncer.draw(canvas);
  }
}
```

1.  接下来，你需要实现实际逻辑来动画每一帧。通过创建一个 `onNextFrame` 方法来实现，这个方法首先检查动画是否应该继续运行（如果它没有运行，我们停止动画），然后告诉每个 `Bouncer` 在动画中移动一步。在你设置了下一个动画帧之后，你需要通过调用 `invalidate()` 方法告诉平台重新绘制 `BouncingDrawablesView`。一旦 `onNextFrame()` 方法完成，我们将它安排在 16 毫秒后再次调用（安排每秒大约 60 帧）：

```kt
private final Runnable postNextFrame = new Runnable() {
  @Override
  public void run() {
    onNextFrame();
  }
};

void onNextFrame() {
  if (bouncers == null || !running) {
    return;
  }

  final Rect boundary = new Rect(
       getPaddingLeft(),
       getPaddingTop(),
       getWidth() - getPaddingLeft() - getPaddingRight(),
       getHeight() - getPaddingTop() - getPaddingBottom()
  );

  for (final Bouncer bouncer : bouncers) {
    bouncer.step(boundary);
  }

  invalidate();
  getHandler().postDelayed(postNextFrame, 16);
}
```

1.  为了在 `BouncingDrawablesView` 变得可见时自动开始动画，并在不可见时停止它，你需要知道 `BouncingDrawablesView` 是何时附加到 `Window`（当它附加到屏幕组件时）。为此，你需要覆盖 `onAttachedToWindow` 并调用 `onNextFrame()`。然而，`onAttachedToWindow` 在布局执行之前被调用，所以你将 `onNextFrame()` 安排在当前事件队列的末尾运行：

```kt
@Override
protected void onAttachedToWindow() {
  super.onAttachedToWindow();
  running = true;

  post(postNextFrame);
}

@Override
protected void onDetachedFromWindow() {
  super.onDetachedFromWindow();
  running = false;
}
```

1.  最后，为 `Bouncer` 对象编写设置器和获取器：

```kt
public void setBouncers(final Bouncer[] bouncers) {
  this.bouncers = bouncers;
}

public Bouncer[] getBouncers() {
  return bouncers;
}
```

设置 `BouncingDrawablesView` 是一个非常简单的过程。一个 `Activity` 需要创建一个包含一些随机位置和速度的 `Bouncer` 对象数组，然后将它们传递给 `BouncingDrawablesView` 实例来处理。一旦 `BouncingDrawablesView` 在屏幕上可见，它将开始动画屏幕周围的 `Drawable` 对象。`BouncingDrawableView` 的简单配置示例可能看起来像这样：

```kt
final BouncingDrawablesView bouncingDrawablesView = (BouncingDrawablesView) findViewById(R.id.bouncing_view);
final BouncingDrawablesView.Bouncer[] bouncers = new BouncingDrawablesView.Bouncer[10];
final Random random = new Random();
final Resources res = getResources();

final Drawable icon = res.getDrawable(R.drawable.ic_other_black);
final int iconSize = res.getDimensionPixelSize(R.dimen.bouncing_icon_size);
for (int i = 0; i < bouncers.length; i++) {
  final Rect bounds = new Rect();
  bounds.top = random.nextInt(400);
  bounds.left = random.nextInt(600);
  bounds.right = bounds.left + iconSize;
  bounds.bottom = bounds.top + iconSize;
  icon.setBounds(bounds);

  bouncers[i] = new BouncingDrawablesView.Bouncer(
        icon,
        random.nextBoolean() ? 6 : -6,
        random.nextBoolean() ? 6 : -6
  );
}

bouncingDrawablesView.setBouncers(bouncers);
```

# 测试你的知识

1.  当为自定义小部件渲染专用图形时，你需要做以下哪一项？

    +   在离屏 `Bitmap` 中缓冲所有渲染

    +   设置自定义背景 `Drawable`

    +   覆盖 `onDraw` 方法

1.  你应该在 `onDraw` 中渲染时创建图形原语实例，如 `Drawable`、`Paint` 和 `Path` 的位置？

    +   在主线程上

    +   在 `onDraw` 方法中

    +   任何不会直接影响 `onDraw` 的地方

1.  布局过程涉及的两个阶段是什么？

    +   布局然后测量

    +   测量和布局

    +   测量然后渲染

1.  当绘制 `Drawable` 对象时，你需要做以下哪一项？

    +   传递一个有效的 `Canvas` 对象

    +   使用 `Canvas.paintDrawable`

    +   调用它的 `onDraw` 方法

1.  为了告诉平台一个小部件需要重新绘制自己（从主线程），你使用以下哪个？

    +   `View.redraw()`

    +   `View.invalidate()`

    +   `View.repaint()`

# 应用你的知识

本书所涵盖的大部分内容是理论（如何设计屏幕）和硬实践知识（编写代码以生成该屏幕）的结合。当你将良好的理论基础与你在工作的平台上的实践知识结合起来时，你就拥有了一个强大的组合。能够编写出色的应用程序并不仅仅是能够编写代码（编程中很少有关于仅仅能够编写代码的事情）。它还涉及到对用户界面的细节的关注，并且始终考虑你的用户。

当正确使用时，Android 是一个惊人的强大平台。在这本书中，你已经学会了使用数据绑定、Room 数据存储系统和`LiveData`等 API。Android 平台上的这些 API 组合不仅允许你快速开发优秀应用程序，而且还提供了代码库不同区域之间优秀的分离。它们也绝不会以任何方式减少你可以从底层平台和系统中利用的权力（如 SQLite）。

Android 社区规模庞大，除了核心平台之外，还有很多可以找到和使用的资源，可以使开发更加容易。以下是一些特别有用的资源、文档和 API 链接：

+   官方的 Android 平台参考：

    [`developer.android.com/reference/packages.html`](https://developer.android.com/reference/packages.html)

+   Firebase（处理托管、推送通知、数据库同步、身份验证以及更多）：

    [`firebase.google.com/`](https://firebase.google.com/)

+   Android Arsenal，一个非官方的第三方 API 和小部件列表：

    [`android-arsenal.com/`](https://android-arsenal.com/)

+   Joda-Time API，在 Java 8 之前在 Java 核心平台上的事实标准时间 API，虽然在 Android 上仍然有用：

    [`www.joda.org/joda-time/`](http://www.joda.org/joda-time/)

+   官方的 SQLite 网站：

    [`sqlite.org/`](https://sqlite.org/)

最后，这里有一些有趣的项目想法，你可以在完成这本书后尝试实现：

+   尝试扩展旅行报销示例，以允许进行多次旅行。

+   编写一个简单的支出跟踪器，允许用户输入并跟踪他们的支出。

+   一个打包/搬家组织应用程序，允许用户拍摄箱子的内容并记录它们的物品，以便他们在搬家时使用。

+   一个待办事项列表应用程序，允许用户创建他们需要完成的各项事务的列表，并在完成后勾选。为了让这更有趣，你可以添加提醒和截止日期（必须在特定日期和时间完成的项目）。

+   一个实时聊天应用程序，这要复杂一些；使用 Firebase 实时数据库来存储和同步聊天消息。

# 摘要

构建自己的自定义组件可能是一项大量工作，但也可以非常有益。对测量、布局和渲染周期拥有完全的控制权，为你提供了几乎可以构建任何你想象中的小部件的惊人力量。Android 还定义了一些优秀的默认值，让你可以专注于你的小部件应该如何看起来和工作，而不是陷入渲染管道的复杂性中。

`Drawable`类是 Android 中最强大的图形原语之一。由于它们实际上非常强大，所以很难称它们为原语。尽可能使用它们而不是`Bitmap`或`Path`，因为它们使得未来的改进更加简单，并且可以轻松地与资源系统集成。

使用`Handler`类来动画化小部件也是一个非常强大且底层的机制。在这些类型的动画中引入实时感通常是一个好主意，这样渲染时间稍长或稍短的画面就不会影响应用程序的整体感觉。这可以通过简单地使用每一帧的时间戳并根据该时间戳移动值来实现，而不是使用固定值。在这种情况下，`Bouncer`的速度将变为`像素/时间`的数量，而不是每帧固定数量的像素。

在构建自己的小部件或布局之前，你应该始终在网上四处看看，看看是否已经有一个现有的项目做了你想要的事情。了解小部件是如何实际构建和组合的，这是一项有用的知识，应该让你有信心不仅创建自己的，还可以帮助他人构建他们的。
