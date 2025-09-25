# 使概览/仪表板屏幕更加完善

当你在第七章“创建概览屏幕”中构建概览/仪表板屏幕时，使用了`RecyclerView`，并通过 Room 和数据绑定从数据库检索记录列表并显示给用户，效果非常好。然而，还可以做得更好。`RecyclerView`是一个功能强大的数据展示引擎，我们实际上只是触及了它能力的一小部分。在本章中，我们将更深入地探讨`RecyclerView`周围的一些生态系统，并将一些重大改进集成到示例中。具体来说，我们将探讨以下内容：

+   以多种方式布局具有多个视图类型的`RecyclerView`

+   提高`RecyclerView`性能的方法

+   动画化`RecyclerView`中的更改

+   将复杂性从主线程中移除

# 多种视图类型

`RecyclerView`能够处理几乎任何数量的不同类型的屏幕小部件，并独立地回收它们。这是一种非常强大且有用的技术，不仅能够显示不同类型的数据，而且还能以大多数情况下对用户透明的方式调整`RecyclerView`的布局。然而，你需要考虑如何具体地分割布局。

通常情况下，你会在`RecyclerView`中使用不同视图类型的主要原因有两个：

+   使用分隔符将长列表项分割开

+   当你想要渲染不同类型的数据时

让我们从创建和添加分隔符开始；当数据绑定到每个小部件时，你可以调整每个小部件的边距，但这并不能帮助用户理解分隔符为什么存在。通常，你希望分隔符携带它所代表的具体细节，例如日期标签。在这些情况下，你需要小部件来渲染标签。

你可以创建一个特殊的布局变体，包括分隔符，通过将其嵌入`LinearLayout`中实现。例如，如果你想向旅行报销应用的概览中显示的报销项添加分隔符标签，你可以添加一个名为`card_claim_item_with_divider`的特殊布局，其外观可能如下所示：

```kt
<?xml version="1.0" encoding="utf-8"?>
<layout >
  <data>
    <variable
        name="presenter"
        type="com.packtpub.claim.ui.presenters.ItemPresenter" />

    <variable
        name="item"
        type="com.packtpub.claim.model.ClaimItem" />
  </data>

  <LinearLayout
      android:layout_width="match_parent"
      android:layout_height="wrap_content"
      android:orientation="vertical">

    <TextView
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginTop="@dimen/grid_spacer1"
        android:layout_marginBottom="@dimen/grid_spacer1"
        android:text="@{presenter.dividerLabel(item)}"
        android:textAppearance="@style/TextAppearance.AppCompat.Caption" />

    <include
        item="@{item}"
        layout="@layout/card_claim_item"
        presenter="@{presenter}" />
  </LinearLayout>
</layout>
```

这种方法实现起来非常简单，因为分隔符被制作成出现在其下方项的一部分。这反过来意味着你的`Adapter`实现只需要决定一个项是否需要分隔符，而不是像跟踪分隔符作为自己的对象类型那样。

然而，这种方法也有几个显著的缺点；每个分隔符现在都携带一个整个卡片，如果没有分隔符在屏幕上，`RecyclerView`仍然会在屏幕外维护一个分隔符的池。这意味着整个未使用的卡片都无法使用，并且占用了比应有的更多内存和数据。这种方法的其他问题是，你将小部件嵌套在另一个`LinearLayout`层中。`LinearLayout`以与`RecyclerView`附加的`LinearLayoutManager`完全相同的方式渲染包含的小部件。因此，这个布局引入了一个对布局系统实际上没有增加任何价值的控件，并且会负面影响应用程序的性能。

那么，替代方案是什么呢？实际上非常简单；将分隔符视为`RecyclerView`中的特殊项目。当你拆分`RecyclerView`时，每个视图类型都会被赋予一个整数标识符，这使得`RecyclerView`能够独立地在不同的回收池中跟踪它们，并确保每个分隔符只用在正确的位置。将分隔符作为特殊项目引入的最简单方法，就是在数据集中将其作为特殊项目引入。这可以通过在需要分隔符的`ClaimItem`对象`List`中添加 null 值来实现，但这与数据绑定层不太兼容，并且扩展性不好。

更好的方法是使用数据集中的`wrapper`对象，告诉`Adapter`实现如何渲染每个项目。这个列表可以预先计算，并减少了布局和渲染的复杂性。这也允许为数据集中的每个项目做出非常复杂的选择，而不会影响用户对应用程序性能的感知。让我们构建一个`DisplayItem`类，它可以与`DataBoundViewHolder`类一起使用，允许在单个`Adapter`中使用任意数量的不同项目类型：

1.  在旅行索赔示例项目中，在`com.packtpub.claim.ui`包上右键单击，并选择“新建| Java 类”。

1.  将新类命名为`DisplayItem`并点击“确定”。

1.  声明一个整数字段来表示每个`DisplayItem`对象的布局资源。这些将由`Adapter`类用来确定加载和渲染哪个布局：

```kt
public class DisplayItem {
  public final int layout;
```

1.  这个类预期将作为混合列表的一部分使用，因此在这一级别使用泛型是不合适的。声明一个普通的`Object`字段来保存`DisplayItem`要绑定到其布局的数据（如果有）：

```kt
public final Object value;
```

1.  现在，你需要一个构造函数来分配这两个字段：

```kt
public DisplayItem(
    final int layout,
    final Object value) {

  this.layout = layout;
  this.value = value;
}
```

1.  为了方便`Adapter`类，`DisplayItem`将提供一个`bindItem`方法来帮助`DataBoundViewHolder`类：

```kt
public <I> void bindItem(final DataBoundViewHolder<?, I> holder) {
  @SuppressWarnings("unchecked") final I item = (I) value;
  holder.setItem(item);
}
```

这是一个非常简单的类来实现，但它在`Adapter`的实现方式上产生了非常大的差异。由于`Adapter`中的数据集不再是直接从数据库或网络读取的原始数据集，你可以自由地混合各种数据源，而无需在`onBindViewHolder`方法中做任何工作。`DisplayItem`有点像`ViewHolder`，但实际上并不持有用户界面小部件；相反，它只是指示需要使用哪种布局来显示携带的数据。

# 引入分隔符

为了在声明概览屏幕中引入分隔符，你需要对从 Room 数据库层传递的数据进行第二次遍历，并确定哪些项需要分隔符。这应该在后台工作线程中完成，以便较大的数据集不会影响用户体验。让我们开始工作，并在旅行声明应用中添加一些简单的分隔符，以显示在不同天制作的声明项之间；这将需要对`ClaimItemAdapter`类的工作方式做出一些重大更改。最明显的变化是，它现在将有一个`DisplayItem`对象的`List`，而不是直接包含`ClaimItem`对象的`List`。

按照以下步骤重构`ClaimItemAdapter`以使用`DisplayItem`对象在`RecyclerView`中混合声明项和分隔符：

1.  首先，你需要一条漂亮的线，可以用作分隔符。这将是一个可以使用`ImageView`小部件渲染的可绘制资源。在`res/drawable`目录上右键单击，然后选择“新建|可绘制资源文件”。

1.  将新文件命名为`horizontal_divider`，然后点击“确定”以创建新的资源。

1.  切换到文本编辑器。

1.  默认情况下，Android Studio 将创建一个`selector`可绘制资源，但你想要声明一个`shape`可绘制资源。用以下 XML 可绘制资源替换生成的模板代码：

```kt
<?xml version="1.0" encoding="utf-8"?>
<shape 
    android:shape="line">

    <stroke
        android:width="1dp"
        android:color="#e0e0e0" />
</shape>
```

1.  你还需要一个用于`RecyclerView`中分隔符的布局。在`res/layout`目录上右键单击，然后选择“新建|布局资源文件”。

1.  将新布局文件命名为`widget_divider`。

1.  将根元素更改为`layout`。

1.  点击“确定”以创建新的布局资源文件。

1.  新布局实际上不需要绑定任何变量，因此你可以将`data`部分留空。使用`ImageView`来渲染全宽度的新的`horizontal_divider`：

```kt
<layout >
    <data></data>

    <ImageView
 android:layout_width="match_parent"
 android:layout_height="@dimen/grid_spacer1"
 android:layout_marginTop="@dimen/grid_spacer1"
 android:src="img/horizontal_divider" />
</layout>
```

1.  现在，打开`ClaimItemAdapter`源文件。

1.  将`ClaimItem`对象的`List`改为`DisplayItem`对象的`List`：

```kt
private List<DisplayItem> items = Collections.emptyList();
```

1.  声明一个新的重写方法--`getItemViewType`--并使用`DisplayItem.layout`值来识别在`RecyclerView`中将使用的布局之间的差异。此方法将委托给`DisplayItem`对象，并使用布局资源 ID 作为标识符：

```kt
@Override
public int getItemViewType(final int position) {
    return items.get(position).layout;
}
```

直接使用布局资源 ID 来确定`RecyclerView`中的不同视图类型是一个常见的技巧。这避免了内部 ID 数字和布局资源之间的映射。

1.  现在，将`onCreateViewHolder`方法更改为使用`viewType`来决定加载哪个布局资源。`viewType`将由`RecyclerView`传入，并将与`getItemViewType`返回的值相同：

```kt
@Override
public DataBoundViewHolder<ItemPresenter, ClaimItem>
    onCreateViewHolder(
      final ViewGroup parent,
      final int viewType) {

  return new DataBoundViewHolder<>(
      DataBindingUtil.inflate(
          layoutInflater,
          viewType,
          parent,
          false
      ),
      itemPresenter
  );
}
```

1.  将`onBindViewHolder`方法更改为使用`DisplayItem.bindItem`方法，而不是直接调用`DataBoundViewHolder.setItem`：

```kt
@Override
public void onBindViewHolder(
    final DataBoundViewHolder<ItemPresenter, ClaimItem> holder,
    final int position) {

  items.get(position).bindItem(holder);
}
```

1.  在`ClaimItemAdapter`类底部，你需要一个新的`ActionCommand`内部类来完成计算分隔符位置的工作，并将所有`ClaimItem`对象包装在`DisplayItem`对象中：

```kt
private class CreateDisplayListCommand
        extends ActionCommand<List<ClaimItem>, List<DisplayItem>> {
```

1.  `CreateDisplayListCommand`需要一个实用方法来决定是否在两个项目之间插入分隔符。这个实用方法将简单地检查两个项目是否在同一天有时间戳：

```kt
boolean isDividerRequired(
    final ClaimItem item1, final ClaimItem item2) {
  final Calendar c1 = Calendar.getInstance();
  final Calendar c2 = Calendar.getInstance();

  c1.setTime(item1.getTimestamp());
  c2.setTime(item2.getTimestamp());

  return c1.get(Calendar.DAY_OF_YEAR)
             != c2.get(Calendar.DAY_OF_YEAR)
         || c1.get(Calendar.YEAR)
             != c2.get(Calendar.YEAR);
}
```

1.  然后，你需要实现`ActionCommand`的`onBackground`方法，并将`ClaimItem`对象列表处理成`DisplayItem`对象列表：

```kt
@Override
public List<DisplayItem> onBackground(
    final List<ClaimItem> claimItems)
    throws Exception {

  final List<DisplayItem> output = new ArrayList<>();

  for (int i = 0; i < claimItems.size(); i++) {
    final ClaimItem item = claimItems.get(i);
    output.add(new DisplayItem(R.layout.card_claim_item, item));

    if (i + 1 < claimItems.size() // not the last item
        && isDividerRequired(item, claimItems.get(i + 1))) {

      output.add(new DisplayItem(R.layout.widget_divider, null));
    }
  }

  return output;
}
```

1.  要完成`CreateDisplayListCommand`的实现，你需要实现`onForeground`方法。这将把新的`DisplayItem`对象列表分配给`ClaimItemAdapter`，并通知`RecyclerView`发生变化：

```kt
@Override
public void onForeground(final List<DisplayItem> value) {
  ClaimItemAdapter.this.items = value;
  notifyDataSetChanged();
}
```

1.  你需要为每次`LiveData`更新提供一个`CreateDisplayListCommand`实例供`ClaimItemAdapter`使用。在`ClaimItemAdapter`类顶部创建一个新字段，并实例化它：

```kt
private final CreateDisplayListCommand createDisplayListCommand
 = new CreateDisplayListCommand();
private final LayoutInflater layoutInflater;
private final ItemPresenter itemPresenter;
private List<DisplayItem> items = Collections.emptyList();
```

1.  现在，你可以将构造函数更改为使用`CreateDisplayListCommand`而不是直接引用 Room 数据库返回的`ClaimItem`对象列表：

```kt
public ClaimItemAdapter(
        final Context context,
        final LifecycleOwner owner,
        final LiveData<List<ClaimItem>> liveItems) {

  this.layoutInflater = LayoutInflater.from(context);
  this.itemPresenter = new ItemPresenter(context);

  liveItems.observe(owner, new Observer<List<ClaimItem>>() {
    @Override
    public void onChanged(final List<ClaimItem> claimItems) {
 createDisplayListCommand.exec(claimItems);
 }
  });
}
```

如果你现在运行旅行报销应用程序，你会在不同日期捕获的任何报销项目之间看到一个漂亮而轻薄的分隔符。尝试使用日期选择器来改变日期并强制用户界面产生不同的卡片分组。你还会发现，因为所有数据仍然来自数据库，你可以添加和删除数据，用户界面将保持最新。

# 通过 Delta 事件更新

到目前为止，当数据库中的数据发生变化时，`ClaimItemAdapter`只是告诉`RecyclerView`数据已经改变。这不是最有效率的资源使用方式，因为`RecyclerView`实际上并不知道模型中的哪些内容发生了变化，它被迫重新布局整个场景，仿佛整个模型都发生了变化（尽管它会重用已经池化的小部件）。

`RecyclerView`实际上有一个二级机制，允许你告诉它哪些内容发生了变化，而不仅仅是说数据已经改变。这是通过一系列通知来实现的，这些通知会指示单个项目、范围被添加、删除或移动。问题是，为了使用这些方法，你需要知道实际上发生了什么变化。

大多数开发者的第一反应可能是使用更多的事件和信号从 DAO 或代理层来表示变化，然后在`Adapter`中捕获这些事件并将它们转发到`RecyclerView`。这可以工作，实际上，如果通过事件总线而不是`Adapter`直接附加到 DAO 层来实现，效果可能相当好。问题是，这也迫使你必须生成这些事件，翻译它们，并且当可能的并发更改列表变得更加复杂时，它们可能会变得难以控制。

另一种方法是让 Room 处理事件。当通过`LiveData`提供新数据时，你可以比较当前显示给用户的 dataset 与新的 dataset，并计算发生了什么变化。这与版本控制软件（如 Git 或 Mercurial）的工作方式相同；它们比较你所做的与开始时的内容，以创建 delta 或 diff，即更改的差异。这可能很复杂且工作量很大，但 Android 支持库为你提供了支持；它提供了一个名为`DiffUtil`的类，不仅可以用于计算几乎任何两个 dataset 之间的差异，还可以生成正确的 events 集，以传递给`RecyclerView`。让我们在`ClaimItemAdapter`中使用`DiffUtil`，只应用更改到`RecyclerView`：

1.  在搜索差异之前，能够确定两个`ClaimItem`对象是否指向相同的数据库记录，但内容不同，这是非常重要的。为此，你需要一个完整的`equals`方法，这可以通过 Android Studio 生成。在 Android Studio 中打开`ClaimItem`源文件。

1.  在编辑器中单击类名，然后从主菜单栏中选择代码|生成。

1.  从弹出菜单中选择 equals()和 hashCode()。

1.  使用 IDE 提供的所有默认设置，点击“下一步”和“完成”，直到向导完成。

1.  打开`ClaimItemAdapter`源文件。

1.  在`ClaimItemAdapter`类中`CreateDisplayListCommand`内部类下面，声明一个新的`ActionCommand`内部类来处理更新现有的`DisplayItem`对象列表，并触发所需的变化通知：

```kt
private class UpdateDisplayListCommand
        extends ActionCommand<
            Pair<List<DisplayItem>, List<ClaimItem>>,
            Pair<List<DisplayItem>, DiffUtil.DiffResult>
        > {
```

这个类接收并处理包含两个参数的`Pair`对象。作为输入，我们将传递旧的`DisplayItem`对象`List`，以及它需要处理的新的`ClaimItem`对象`List`。作为输出，它将生成新的`DisplayItem`对象`List`，以及一个`DiffResult`，可以用来触发更新事件。

1.  在`UpdateDisplayListCommand`中，你首先需要的是`onBackground`方法。这个方法将使用通过`Pair`传入的`DisplayItem`对象`List`作为“旧”的项`List`，并通过直接调用`CreateDisplayListCommand`来生成一个“新”的`DisplayItem`对象`List`：

```kt
@Override
public Pair<List<DisplayItem>, DiffUtil.DiffResult> onBackground(
        final Pair<List<DisplayItem>, List<ClaimItem>> args)
        throws Exception {

    final List<DisplayItem> oldDisplay = args.first;
    final List<DisplayItem> newDisplay =
            createDisplayListCommand.onBackground(args.second);
```

1.  现在你有了当前显示给用户的`List`和需要显示的`List`，是时候计算它们之间的差异了。为了保持完全通用，`DiffUtil`定义了一个回调接口，用于查询两个列表的详细信息。在`UpdateDisplayListCommand`类中，我们将简单地使用匿名内部类：

```kt
final DiffUtil.DiffResult result =
      DiffUtil.calculateDiff(new DiffUtil.Callback() {
  @Override
  public int getOldListSize() {
    return oldDisplay.size();
  }

  @Override
  public int getNewListSize() {
    return newDisplay.size();
  }
```

1.  `Callback`实现还需要一个方法来比较两个不同位置的项目，以查看它们是否看起来是相同的项。首先，我们需要检查它们的布局是否看起来相同。如果布局不相同，我们可以确信它们不是同一个对象。如果布局相同，那么我们可以查看布局整数作为`DisplayItem`对象中数据类型的指示器。如果是`ClaimItem`，我们使用对象的数据库 ID 来查看它们是否代表数据库中的相同记录：

```kt
@Override
public boolean areItemsTheSame(
    final int oldItemPosition,
    final int newItemPosition) {
  final DisplayItem oldItem = oldDisplay.get(oldItemPosition);
  final DisplayItem newItem = newDisplay.get(newItemPosition);

  if (oldItem.layout != newItem.layout) {
    return false;
  }

  switch (newItem.layout) {
    case R.layout.card_claim_item:
      final ClaimItem oldClaimItem = (ClaimItem) oldItem.value;
      final ClaimItem newClaimItem = (ClaimItem) newItem.value;
      return oldClaimItem != null
          && newClaimItem != null
          && oldClaimItem.id == newClaimItem.id;
    case R.layout.widget_divider:
      return true;
  }

  return false;
}
```

1.  `Callback`还需要另一个方法来测试两个对象的实际内容是否已更改。此方法仅在`areItemsTheSame`方法返回 true 时由`DiffUtil`调用，这允许你通过假设两边代表相同的记录来在实现中采取一些捷径：

```kt
@Override
public boolean areContentsTheSame(
    final int oldItemPosition,
    final int newItemPosition) {
  final DisplayItem oldItem = oldDisplay.get(oldItemPosition);
  final DisplayItem newItem = newDisplay.get(newItemPosition);

  switch (newItem.layout){
    case R.layout.card_claim_item:
      final ClaimItem oldClaimItem = (ClaimItem) oldItem.value;
      final ClaimItem newClaimItem = (ClaimItem) newItem.value;
      return oldClaimItem != null
          && newClaimItem != null
          && oldClaimItem.equals(newClaimItem);
    case R.layout.widget_divider:
      return true;
  }

  return false;
}
```

1.  这就完成了`Callback`的实现。现在，你需要通过返回一个包含新的`DisplayItem`对象列表和`DiffResult`的`Pair`来关闭`onBackground`方法：

```kt
  }); // end of the DiffUtil.Callback implementation

  return Pair.create(newDisplay, result);
} // end of the onBackground implementation
```

1.  在`UpdateDisplayListCommand`类的`onForeground`方法中，你需要将新的`DisplayItem`对象列表分配给`ClaimItemAdapter`，就像之前一样。然而，你不需要通知`RecyclerView`整个模型已更改，而是可以使用`DiffResult`来传递你发现的一系列差异事件：

```kt
@Override
public void onForeground(
    final Pair<List<DisplayItem>,
    DiffUtil.DiffResult> value) {
  ClaimItemAdapter.this.items = value.first;
  value.second.dispatchUpdatesTo(ClaimItemAdapter.this);
}
```

1.  在`ClaimItemAdapter`的顶部，你现在需要一个包含新`UpdateDisplayListCommand`类实例的字段：

```kt
public class ClaimItemAdapter extends
        RecyclerView.Adapter<DataBoundViewHolder<ItemPresenter, ClaimItem>> { private final UpdateDisplayListCommand updateCommand
 = new UpdateDisplayListCommand();
  private final CreateDisplayListCommand createDisplayListCommand
        = new CreateDisplayListCommand();
  private final LayoutInflater layoutInflater;
  private final ItemPresenter itemPresenter;
```

1.  现在，在`ClaimItemAdapter`类构造函数中，`LiveData`观察者再次发生变化。如果数据来自第一次通知，计算两个列表之间的差异没有意义，但如果是在之后的任何调用中，你现在可以通过`UpdateDisplayListCommand`来运行它：

```kt
public ClaimItemAdapter(
        final Context context,
        final LifecycleOwner owner,
        final LiveData<List<ClaimItem>> liveItems) {

  this.layoutInflater = LayoutInflater.from(context);
  this.itemPresenter = new ItemPresenter(context);

  liveItems.observe(owner, new Observer<List<ClaimItem>>() {
    @Override
    public void onChanged(final List<ClaimItem> claimItems) {
      if (!items.isEmpty()) {
 updateCommand.exec(Pair.create(items, claimItems));
 } else {
 createDisplayListCommand.exec(claimItems);
 }
    }
  });
}
```

这些更改可能看起来只是为了改变`Adapter`实现产生的事件而做的大量工作，但通过一些工作，它们可以相对通用，以便在不同的列表实现中重用。这些更改还带来了非常好的用户体验：动画。因为你现在告诉`RecyclerView`确切的变化，它将自动为你动画化这些变化。

由于分隔符，`DiffUtil` 在这个情况下也是一个出色的工具。尽管模型每次只更改一个 `ClaimItem`，但 `DiffUtil` 也负责添加和删除受这些更改影响的任何分隔符。如果你从数据库层触发这些事件中的每一个，你需要手动处理这些额外的更改，而尽管 `DiffUtil` 可能不是最有效的工具，但它保持了数据的绝对一致性。

# 测试你的知识

1.  在单个 `RecyclerView` 实例中，你可以使用多少种不同的视图类型？

    +   一个或两个

    +   任何数量

    +   256

1.  当使用 `DiffUtil` 时，以下哪一项适用于你正在比较的数据？

    +   它必须是一个数据库实体

    +   它必须是可比较的

    +   它通过回调函数暴露

1.  当向 `RecyclerView` 添加分隔符时，你应该做以下哪一项？

    +   将它们作为分隔符上方的项目的一部分

    +   在 `onBindViewHolder` 方法中将它们添加到显示中

    +   将它们作为数据集中的独立项目

# 摘要

在本章中，我们主要关注 `RecyclerView` 以及如何在你的应用程序中使其工作得更好，特别是对于概览/仪表板屏幕。添加分隔符和动画等更改不会改变应用程序的功能，但它们确实会改变用户体验。在这种情况下，它们使用户更容易理解屏幕，并更容易理解当事情发生变化时发生了什么。

这些类型的更改可以被视为“润色”应用程序。你可以不使用它们来构建应用程序以确保一切正常工作，然后之后添加它们。慢慢地构建一个可以快速润色任何应用程序的通用结构列表是个好主意。一个很好的例子是使用 `DiffUtil` 的通用 `ActionCommand` 来应用更改到 `Adapter`。

在下一章中，我们将花更多的时间来润色应用程序。我们将探讨动画、颜色和样式，并探索如何在应用程序中定义和使用它们以应用一致的主题。
