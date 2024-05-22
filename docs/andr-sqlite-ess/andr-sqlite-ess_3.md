# 第三章：分享就是关怀

|   | *"数据真的驱动着我们所做的一切。"* |   |
| --- | --- | --- |
|   | --*– Jeff Weiner, LinkedIn* |

在上一章中，我们开始编写我们自己的联系人管理器。我们遇到了数据库中心应用程序的各种构建模块；我们涵盖了数据库处理程序和构建查询，以便从我们的数据库中获取有意义的数据。我们还探讨了如何在我们的 UI 和数据库之间建立连接，并以一种可消费的方式呈现给最终用户。

在这一章中，我们将学习如何通过内容提供程序访问其他应用程序的数据。我们还将学习如何构建自己的内容提供程序，以便与其他应用程序共享我们的数据。我们将研究 Android 的提供者，如**contactprovider**。最后，我们将构建一个测试应用程序来使用我们新构建的内容提供程序。

在本章中，我们将涵盖以下主题：

+   什么是内容提供程序？

+   创建内容提供程序

+   实现核心方法

+   使用内容提供程序

# 什么是内容提供程序？

内容提供程序是 Android 应用程序的第四个组件。它用于管理对结构化数据集的访问。内容提供程序封装数据，并提供抽象和定义数据安全性的机制。然而，内容提供程序主要用于被其他应用程序使用，这些应用程序使用提供程序的客户端对象访问提供程序。提供程序和提供程序客户端一起为数据提供了一致的标准接口，还处理了进程间通信和安全数据访问。

内容提供程序允许一个应用程序与其他应用程序共享数据。按设计，由应用程序创建的 Android SQLite 数据库对应用程序是私有的；从安全的角度来看，这是很好的，但当你想要在不同的应用程序之间共享数据时会很麻烦。这就是内容提供程序发挥作用的地方；通过构建自己的内容提供程序，您可以轻松地共享数据。重要的是要注意，尽管我们的讨论将集中在数据库上，但内容提供程序并不局限于此。它也可以用来提供通常存储在文件中的文件数据，如照片、音频或视频：

![什么是内容提供程序？](img/2951OS_03_01.jpg)

在上图中，注意应用程序 A 和 B 之间交换数据的交互方式。在这里，我们有一个**应用程序 A**，其活动需要访问**应用程序 B**的数据库。正如我们已经看到的，**应用程序 B**的数据库存储在内部存储器中，无法直接被**应用程序 A**访问。这就是**内容提供程序**出现的地方；它允许我们共享数据并修改对其他应用程序的访问。内容提供程序实现了查询、插入、更新和删除数据库中的数据的方法。**应用程序 A**现在请求内容提供程序代表它执行一些所需的操作。我们将探索这个问题的两面，但我们将首先使用**内容提供程序**从手机的联系人数据库中获取联系人，然后我们将构建我们自己的内容提供程序，供其他人从我们的数据库中获取数据。

## 使用现有内容提供程序

Android 列出了许多标准内容提供程序，我们可以使用。其中一些是`Browser`、`CalendarContract`、`CallLog`、`Contacts`、`ContactsContract`、`MediaStore`、`userDictionary`等。

在我们当前的联系人管理应用程序中，我们将添加一个新功能。在`AddNewContactActivity`类的 UI 中，我们将添加一个小按钮，以帮助系统的现有`ContentProvider`和`ContentResolver`提供程序从手机的联系人列表中获取联系人。我们将使用`ContactsContract`提供程序来实现这个目的。

### 什么是内容解析器？

应用程序上下文中的`ContentResolver`对象用于作为客户端与提供程序进行通信。`ContentResolver`对象与提供程序对象通信——提供程序对象是实现`ContentProvider`的类的实例。提供程序对象接收来自客户端的数据请求，执行请求的操作，并返回结果。

`ContentResolver`是我们应用程序中的一个单一的全局实例，它提供了对其他应用程序的内容提供程序的访问；我们不需要担心处理进程间通信。`ContentResolver`方法提供了持久存储的基本 CRUD（创建、检索、更新和删除）功能；它有调用提供程序对象中同名方法的方法，但不知道实现。随着我们在本章中的进展，我们将更详细地介绍`ContentResolver`。

![什么是内容解析器？](img/2951_03_02.jpg)

在前面的屏幕截图中，注意右侧的新图标，可以直接从手机联系人中添加联系人；我们修改了现有的 XML 以添加这个图标。相应的类`AddNewContactActivity`也将被修改：

```kt
public void pickContact() {
   try {
       Intent cIntent = new Intent(Intent.ACTION_PICK,
            ContactsContract.Contacts.CONTENT_URI);
      startActivityForResult(cIntent, PICK_CONTACT);
    } catch (Exception e) {
      e.printStackTrace();
      Log.i(TAG, "Exception while picking contact");
    }
   }
```

我们添加了一个新的方法`pickContact()`来准备一个意图以选择联系人。`Intent.ACTION_PICK`允许我们从数据源中选择一个项目；此外，我们只需要知道提供程序的**统一资源标识符**（**URI**），在我们的情况下是`ContactsContract.Contacts.CONTENT_URI`。这个功能也由消息、画廊和联系人提供。如果您查看第二章的代码，*连接点*，您会发现我们已经使用了相同的代码从画廊中选择图像。联系人屏幕将弹出，允许我们浏览或搜索我们需要迁移到我们的新应用程序的联系人。注意`onActivityResult`，也就是说，我们的下一站我们将修改这个方法来处理我们对联系人的相应请求。让我们看看我们需要添加的代码，以从 Android 的联系人提供程序中选择联系人：

```kt
{
.
.
.

else if (requestCode == PICK_CONTACT) {
      if (resultCode == Activity.RESULT_OK)

       {
          Uri contactData = data.getData();
          Cursor c = getContentResolver().query(contactData, null, null, null, null);
         if (c.moveToFirst()) {
             String id = c
                   .getString(c
                         .getColumnIndexOrThrow(ContactsContract.Contacts._ID));

             String hasPhone = c
                   .getString(c
                         .getColumnIndex(ContactsContract.Contacts.HAS_PHONE_NUMBER));

            if (hasPhone.equalsIgnoreCase("1")) {
                Cursor phones = getContentResolver()
                      .query(ContactsContract.CommonDataKinds.Phone.CONTENT_URI,
                           null,
                           ContactsContract.CommonDataKinds.Phone.CONTACT_ID
                                  + " = " + id, null, null);
               phones.moveToFirst();
               contactPhone.setText(phones.getString(phones
                      .getColumnIndex("data1")));

               contactName
                      .setText(phones.getString(phones
                            .getColumnIndex(ContactsContract.Contacts.DISPLAY_NAME)));

 }
…..
```

### 提示

为了为您的应用程序增添一些特色，可以从 Android 开发者网站[`goo.gl/4Msuct`](http://goo.gl/4Msuct)下载整套模板、源代码、操作栏图标包、颜色样本和 Roboto 字体系列。设计一个功能性应用程序是不完整的，如果没有遵循 Android 指南的一致 UI。

我们首先检查请求代码是否与我们的匹配。然后，我们交叉检查`resultcode`。我们通过在`Context`对象上调用`getcontentresolver`来获取`ContentResolver`对象；这是`android.content.Context`类的一个方法。由于我们在一个继承自`Context`的活动中，我们不需要显式地调用它。服务也是一样。现在我们将验证我们选择的联系人是否有电话号码。在验证必要的细节之后，我们提取我们需要的数据，比如联系人姓名和电话号码，并将它们设置在相关字段中。

# 创建内容提供程序

内容提供程序以两种方式提供数据访问：一种是以数据库的形式进行结构化数据，就像我们目前正在处理的例子一样，或者以文件数据的形式，也就是说，以图片、音频、视频等形式存储在应用程序的私有空间中。在我们开始深入研究如何创建内容提供程序之前，我们还应该回顾一下我们是否需要一个。如果我们想要向其他应用程序提供数据，允许用户从我们的应用程序复制数据到另一个应用程序，或者在我们的应用程序中使用搜索框架，那么答案就是肯定的。

就像其他 Android 组件（`Activity`、`Service`或`BroadcastReceiver`）一样，内容提供程序是通过扩展`ContentProvider`类来创建的。由于`ContentProvider`是一个抽象类，我们必须实现这六个抽象方法。这些方法如下：

| 方法 | 用法 |
| --- | --- |
| `void onCreate()` | 初始化提供程序 |
| `String getType(Uri)` | 返回内容提供程序中数据的 MIME 类型 |
| `int delete(Uri uri, String selection, String[] selectionArgs)` | 从内容提供程序中删除数据 |
| `Uri insert(Uri uri, ContentValues values)` | 将新数据插入内容提供程序 |
| `Cursor query(Uri uri, String[] projection, String selection, String[] selectionArgs, String sortOrder)` | 返回数据给调用者 |
| `int update(Uri uri, ContentValues values, String selection, String[] selectionArgs)` | 更新内容提供程序中的现有数据 |

随着我们在本章中的进展和应用程序的构建，这些方法将在以后更详细地讨论。

## 理解内容 URI

`ContentProvider`的每个数据访问方法都有一个内容 URI 作为参数，允许它确定要访问的表、行或文件。它通常遵循以下结构：

```kt
content://authority/Path/Id
```

让我们分析`content://` URI 组件的分解。内容提供程序的方案始终是`content`。冒号和双斜杠(`://`)充当与权限部分的分隔符。然后，我们有`authority`部分。根据规则，每个内容提供程序的权限都必须是唯一的。Android 文档推荐使用的命名约定是内容提供程序子类的完全限定类名。通常，它是一个包名称加上我们发布的每个内容提供程序的限定符。

剩下的部分是可选的，也称为**path**，用于区分内容提供程序可以提供的不同类型的数据。一个很好的例子是`MediaStore`提供程序，它需要区分音频、视频和图像文件。

另一个可选部分是`id`，它指向特定记录；根据`id`是否存在，URI 分别成为基于 ID 或基于目录。另一种理解方式是，基于 ID 的 URI 使我们能够在行级别单独与数据交互，而基于目录的 URI 使我们能够与数据库的多行交互。

例如，考虑`content://com.personalcontactmanager.provider/contacts`；随着我们继续本章的进展，我们很快就会遇到这个。

### 注意

顺便说一句，应用程序的包名称应始终是唯一的；这是因为 Play 商店上的所有应用程序都是通过其包名称进行识别的。Play 商店上的应用程序的所有更新都需要具有相同的包名称，并且必须使用最初使用的相同密钥库进行签名。例如，以下是 Gmail 应用程序的 Play 商店链接；请注意，在 URL 的末尾，我们将找到应用程序的包名称：

[play.google.com/store/apps/details?id=com.google.android.gm](http://play.google.com/store/apps/details?id=com.google.android.gm)

## 声明我们的合同类

声明合同是构建内容提供程序的非常重要的部分。这个类，正如其名称所示，将充当我们的内容提供程序和将要访问我们的内容提供程序的应用程序之间的合同。它是一个`public final`类，其中包含 URI、列名和其他元数据的常量定义。它也可以包含 Javadoc，但最大的优势是使用它的开发人员不需要担心表的名称、列和常量的名称，从而减少了容易出错的代码。

合同类为我们提供了必要的抽象；我们可以根据需要更改底层操作，也可以更改影响其他依赖应用程序的相应数据操作。需要注意的一点是，我们在未来更改合同时需要小心；如果我们不小心，可能会破坏使用我们合同类的其他应用程序。

我们的合同类看起来像下面这样：

```kt
public final class PersonalContactContract {

   /**
    * The authority of the PersonalContactProvider
    */
   public static final String AUTHORITY = "com.personalcontactmanager.provider";

   public static final String BASE_PATH = "contacts";

   /**
    * The Uri for the top-level PersonalContactProvider
    * authority
    */
   public static final Uri CONTENT_URI = Uri.parse("content://" + AUTHORITY 
         + "/" + BASE_PATH);

   /**
    * The mime type of a directory of items.
    */
   public static final String CONTENT_TYPE =                  
ContentResolver.CURSOR_DIR_BASE_TYPE + 
                  "/vnd.com.personalcontactmanager.provider.table";
   /**
    * The mime type of a single item.
    */
   public static final String CONTENT_ITEM_TYPE = 
ContentResolver.CURSOR_ITEM_BASE_TYPE + 
                 "/vnd.com.personalcontactmanager.provider.table_item";

   /**
    * A projection of all columns 
    * in the items table.
    */
   public static final String[] PROJECTION_ALL = { "_id", 
      "contact_name", "contact_number", 
      "contact_email", "photo_id" };

   /**
    * The default sort order for 
    * queries containing NAME fields.
    */
   //public static final String SORT_ORDER_DEFAULT = NAME + " ASC";

   public static final class Columns {
      public static String TABLE_ROW_ID = "_id";
      public static String TABLE_ROW_NAME  = "contact_name";
      public static String TABLE_ROW_PHONENUM = "contact_number";
      public static String TABLE_ROW_EMAIL = "contact_email";
      public static String TABLE_ROW_PHOTOID = "photo_id";
   }
}
```

`AUTHORITY`是在 Android 系统中注册的许多其他提供程序中标识提供程序的符号名称。`BASE_PATH`是表的路径。`CONTENT_URI`是提供程序封装的表的 URI。`CONTENT_TYPE`是包含零个或多个项目的游标的内容 URI 的 Android 平台的基本 MIME 类型。`CONTENT_ITEM_TYPE`是包含单个项目的游标的内容 URI 的 Android 平台的基本 MIME 类型。`PROJECTION_ALL`和`Columns`包含表的列 ID。

没有这些信息，其他开发人员将无法访问您的提供程序，即使它是开放访问的。

### 注意

提供程序内部可能有许多表，每个表都应该有一个唯一的路径；路径不是真正的物理路径，而是一个标识符。

## 创建 UriMatcher 定义

`UriMatcher`是一个实用类，它帮助匹配内容提供程序中的 URI。`addURI()`方法接受提供程序应该识别的内容 URI 模式。我们添加一个要匹配的 URI，以及在匹配此 URI 时返回的代码：

```kt
addURI(String authority, String path, int code)
```

我们将`authority`、`path`模式和整数值传递给`UriMatcher`的`addURI()`方法；当我们尝试匹配模式时，它返回我们定义的常量作为`int`值。

我们的`UriMatcher`看起来像下面这样：

```kt
private static final int CONTACTS_TABLE = 1;
private static final int CONTACTS_TABLE_ITEM = 2;

private static final UriMatcher mmURIMatcher = new UriMatcher(UriMatcher.NO_MATCH);
   static {
      mmURIMatcher.addURI(PersonalContactContract.AUTHORITY, 
            PersonalContactContract.BASE_PATH, CONTACTS_TABLE);
      mmURIMatcher.addURI(PersonalContactContract.AUTHORITY, 
            PersonalContactContract.BASE_PATH+  "/#",  
                       CONTACTS_TABLE_ITEM);
   }
```

请注意，它还支持使用通配符；我们在前面的代码片段中使用了井号（`#`），我们也可以使用通配符，比如`*`。在我们的情况下，使用井号，`"content://com.personalcontactmanager.provider/contacts/2"`这个表达式匹配，但使用`* "content://com.personalcontactmanager.provider/contacts`就不匹配了。

# 实现核心方法

为了构建我们的内容提供程序，下一步将是准备我们的核心数据库访问和数据修改方法，也就是 CRUD 方法。这是我们希望根据接收到的插入、查询或删除调用与数据交互的核心逻辑所在。我们还将实现 Android 架构的生命周期方法，比如`onCreate()`。

## 通过 onCreate()方法初始化提供程序

我们在`onCreate()`中创建我们的数据库管理器类的对象。`oncreate()`中应该有最少的操作，因为它在主 UI 线程上运行，可能会导致某些用户的延迟。最好避免在`oncreate()`中进行长时间运行的任务，因为这会增加提供程序的启动时间。甚至建议将数据库创建和数据加载推迟到我们的提供程序实际收到对数据的请求时，也就是将持续时间长的操作移到 CRUD 方法中：

```kt
@Override
Public Boolean onCreate() {
   dbm = new DatabaseManager(getContext());
   return false;
}   
```

## 通过 query()方法查询记录

`query()`方法将返回结果集上的游标。将 URI 传递给我们的`UriMatcher`，以查看它是否与我们之前定义的任何模式匹配。在我们的 switch case 语句中，如果是与表项相关的情况，我们检查`selection`语句是否为空；如果是，我们将选择语句构建到`lastpathsegment`，否则我们将选择附加到`lastpathsegment`语句。我们使用`DatabaseManager`对象在数据库上运行查询，并得到一个游标作为结果。`query()`方法预期会抛出`IllegalArgumentException`来通知未知的 URI；在查询过程中遇到内部错误时，抛出`nullPointerException`也是一个良好的做法：

```kt
@Override
public Cursor query(Uri uri, String[] projection, String selection,
      String[] selectionArgs, String sortOrder) {

   int uriType = mmURIMatcher.match(uri);
   switch(uriType) {

   case CONTACTS_TABLE:
      break;
   case CONTACTS_TABLE_ITEM:
      if (TextUtils.isEmpty(selection)) {
         selection = PersonalContactContract.Columns.TABLE_ROW_ID 
                  + "=" + uri.getLastPathSegment();
      } else {
         selection = PersonalContactContract.Columns.TABLE_ROW_ID 
                  + "=" + uri.getLastPathSegment() + 
               " and " + selection;
      }
      break;
   default:
      throw new IllegalArgumentException("Unknown URI: " + uri);
   }

   Cursor cr = dbm.getRowAsCursor(projection, selection, 
               selectionArgs, sortOrder);

   return cr;
}
```

### 注意

请记住，Android 系统必须能够跨进程边界通信异常。Android 可以为以下异常执行此操作，这些异常在处理查询错误时可能有用：

+   `IllegalArgumentException`：如果您的提供程序收到无效的内容 URI，您可以选择抛出此异常

+   `NullPointerException`：当对象为空且我们尝试访问其字段或方法时抛出

## 通过 insert()方法添加记录

正如其名称所示，`insert()`方法用于在我们的数据库中插入一个值。它返回插入行的 URI，并且在检查 URI 时，我们需要记住插入可以发生在表级别，因此方法中的操作在与表匹配的 URI 上进行处理。匹配后，我们使用标准的`DatabaseManager`对象将新值插入到数据库中。新行的内容 URI 是通过将新行的`_ID`值附加到表的内容 URI 构造的：

```kt
@Override
public Uri insert(Uri uri, ContentValues values) {

   int uriType = mmURIMatcher.match(uri);
   long id;

   switch(uriType) {
   case CONTACTS_TABLE:
      id = dbm.addRow(values);
      break;
   default:
      throw new IllegalArgumentException("Unknown URI: " + uri);
   }

   Uri ur = ContentUris.withAppendedId(uri, id);
   return ur;
}
```

## 通过 update()方法更新记录

`update()`方法更新适当表中的现有行，使用`ContentValues`参数中的值。首先，我们确定 URI，无论是基于目录还是基于 ID，然后我们构建选择语句，就像我们在`query()`方法中所做的那样。现在，我们将执行我们在第二章中构建此应用程序时定义的标准`DatabaseManager`的`updateRow()`方法，该方法返回受影响的行数。

`update()`方法返回更新的行数。根据选择条件，可以更新一行或多行：

```kt
@Override
public int update(Uri uri, ContentValues values, String selection,
      String[] selectionArgs) {
   int uriType = mmURIMatcher.match(uri);

   switch(uriType) {
   case CONTACTS_TABLE:
      break;
   case CONTACTS_TABLE_ITEM:
      if (TextUtils.isEmpty(selection)) {
         selection = PersonalContactContract.Columns.TABLE_ROW_ID
 + "=" + uri.getLastPathSegment();
      } else {
         selection = PersonalContactContract.Columns.TABLE_ROW_ID 
+ "=" + uri.getLastPathSegment() 
+ " and " + selection;
      }
      break;
   default:
      throw new IllegalArgumentException("Unknown URI: " + uri);
   }

   int count = dbm.updateRow(values, selection, selectionArgs);

   return count;
}
```

## 通过 delete()方法删除记录

`delete()`方法与`update()`方法非常相似，使用它的过程类似；在这里，调用是用来删除一行而不是更新它。`delete()`方法返回删除的行数。根据选择条件，可以删除一行或多行：

```kt
@Override
public int delete(Uri uri, String selection, String[] selectionArgs) {

   int uriType = mmURIMatcher.match(uri);

   switch(uriType) {
   case CONTACTS_TABLE:
      break;
   case CONTACTS_TABLE_ITEM:
      if (TextUtils.isEmpty(selection)) {
         selection = PersonalContactContract.Columns.TABLE_ROW_ID
 + "=" + uri.getLastPathSegment();
      } else {
         selection = PersonalContactContract.Columns.TABLE_ROW_ID 
 + "=" + uri.getLastPathSegment() 
 + " and " + selection;
      }
      break;
   default:
      throw new IllegalArgumentException("Unknown URI: " + uri);
   }

   int count = dbm.deleteRow(selection, selectionArgs);

   return count;
}
```

## 通过 getType()方法获取数据的返回类型

这个简单方法的签名接受一个 URI 并返回一个字符串值；每个内容提供者必须为其支持的 URI 返回内容类型。一个非常有趣的事实是，应用程序访问这些信息时不需要任何权限；如果我们的内容提供者需要权限，或者没有被导出，所有的应用程序仍然可以调用这个方法，而不管它们对检索 MIME 类型的访问权限如何。

所有这些 MIME 类型都应在合同类中声明：

```kt
@Override
public String getType(Uri uri) {

   int uriType = mmURIMatcher.match(uri);
   switch(uriType) {
   case CONTACTS_TABLE:
      return PersonalContactContract.CONTENT_TYPE;
   case CONTACTS_TABLE_ITEM:
      return PersonalContactContract.CONTENT_ITEM_TYPE;
   default:
      throw new IllegalArgumentException("Unknown URI: " + uri);   
   }

}
```

## 将提供者添加到清单中

另一个重要的步骤是将我们的内容提供者添加到清单中，就像我们对其他 Android 组件所做的那样。我们可以在这里注册多个提供者。这里的重要部分，除了`android:authorities`之外，还有`android:exported`；它定义了内容提供者是否可供其他应用程序使用。如果为`true`，则提供者可供其他应用程序使用；如果为`false`，则提供者不可供其他应用程序使用。如果应用程序具有与提供者相同的用户 ID（UID），它们将可以访问它：

```kt
<provider
   android:name="com.personalcontactmanager.provider.PersonalContactProvider"
   android:authorities="com.personalcontactmanager.provider"
   android:exported="true"
   android:grantUriPermissions="true" >
   </provider>
```

另一个重要的概念是**权限**。我们可以通过添加读取和写入权限来增加额外的安全性，其他应用程序必须在其清单 XML 文件中添加这些权限，并自动通知用户他们将要使用特定应用程序的内容提供者来读取、写入或两者兼而有之。我们可以通过以下方式添加权限：

```kt
android:readPermission="com.personalcontactmanager.provider.READ"
```

# 使用内容提供者

我们构建内容提供者的主要原因是允许其他应用程序访问我们数据库中的复杂数据存储并执行 CRUD 操作。现在我们将构建另一个应用程序来测试我们新构建的内容提供者。测试应用程序非常简单，只包括一个活动类和一个布局文件。它有标准按钮来执行操作。没有花哨的东西，只是用来测试我们刚刚实现的功能的工具。现在我们将深入研究`TestMainActivity`类并查看其实现：

```kt
public class TestMainActivity extends Activity {

public final String AUTHORITY = "com.personalcontactmanager.provider";
public final String BASE_PATH = "contacts";
private TextViewqueryT, insertT;

public class Columns {
   public final static String TABLE_ROW_ID = "_id";
   public final static String TABLE_ROW_NAME = "contact_name";
   public final static String TABLE_ROW_PHONENUM =

"contact_number";
   public final static String TABLE_ROW_EMAIL = "contact_email";
   public final static String TABLE_ROW_PHOTOID = "photo_id";
   }
```

要访问内容提供程序，我们需要诸如`AUTHORITY`和`BASE_PATH`的详细信息，以及数据库表的列名称；我们需要访问公共类`Columns`。为此目的。我们有更多的表，我们将看到更多这些类。通常，所有这些必要的信息将从内容提供程序的已发布合同类中获取。一些内容提供程序还需要在清单中实现读取或写入权限：

```kt
<uses-permissionandroid:name="AUTHORITY.permission.WRITE_TASKS"/>
```

在某些情况下，我们需要访问的内容提供程序可能会要求我们在清单中添加权限。当用户安装应用程序时，他们将在其权限列表中看到一个添加的权限：

```kt
@Override
protected void onCreate(Bundle savedInstanceState) {
   super.onCreate(savedInstanceState);
   setContentView(R.layout.activity_test_main);
   queryT = (TextView) findViewById(R.id.textQuery);
   insertT = (TextView) findViewById(R.id.textInsert);
   }
```

### 注意

要尝试其他应用程序的内容提供程序，请参阅[`goo.gl/NEX2hN`](http://goo.gl/NEX2hN)。

它列出了如何使用 Any.do 的内容提供程序-一个非常著名的任务应用程序。

我们将在活动的`onCreate()`中设置我们的布局并初始化我们需要的视图。要查询，我们首先需要准备与表匹配的 URI 对象。

现在内容解析器开始发挥作用；它充当我们准备的内容 URI 的解析器。在这种情况下，我们的`getContentResolver.query()`方法将获取所有列和行。现在，我们将游标移动到第一个位置，以便读取结果。出于测试目的，它被读取为一个字符串：

```kt
public void query(View v) {
  Uri contentUri = Uri.parse("content://" + AUTHORITY 
               + "/" + BASE_PATH);

  Cursor cr = getContentResolver().query(contentUri, null, 
            null, null, null);     

  if (cr != null) {
      if (cr.getCount() > 0) {
         cr.moveToFirst();
         String name = cr.getString(cr.getColumnIndexOrThrow( 
Columns.TABLE_ROW_NAME));
         queryT.setText(name);
      }
  }

  ....
  ....
}
```

现在，我们构建一个 URI 来读取特定行，而不是完整的表。我们已经提到，为了使 URI 基于 ID，我们需要将 ID 部分添加到我们现有的`contenturi`中。现在，我们构建我们的投影字符串数组，以作为我们`query()`方法中的参数传递：

```kt
public void query(View v) {

 ...
 ...

  Uri rowUri = contentUri = ContentUris.withAppendedId
            (contentUri, getFirstRowId());

  String[] projection = new String[] {
      Columns.TABLE_ROW_NAME, Columns.TABLE_ROW_PHONENUM,
      Columns.TABLE_ROW_EMAIL, Columns.TABLE_ROW_PHOTOID };

  cr = getContentResolver().query(contentUri, projection,
      null, null, null);

  if (cr != null) {
      if (cr.getCount() > 0) {
         cr.moveToFirst();
         String name = cr.getString(cr.getColumnIndexOrThrow(
                  Columns.TABLE_ROW_NAME));

         queryT.setText(name);

      }
  }

}   
```

`getFirstRowId()`方法获取表中第一行的 ID。这是因为第一行的 ID 并不总是`1`。当行被删除时，它会发生变化。如果具有行 ID`1`的表中的第一项被删除，那么具有行 ID`1`的第二项将成为第一项：

```kt
private int getFirstRowId() {

  int id = 1;
  Uri contentUri = Uri.parse("content://" + AUTHORITY + "/"
               + "contacts");
  Cursor cr = getContentResolver().query(contentUri, null,
            null, null, null);
  if (cr != null) {
      if (cr.getCount() > 0) {
         cr.moveToFirst();
         id = cr.getInt(cr.getColumnIndexOrThrow(
            Columns.TABLE_ROW_ID));
      }
  }
return id;

}
```

让我们更仔细地看一下`query()`方法：

```kt
public final Cursor query (Uri uri, String[] projection, String selection, String[] selectionArgs, String sortOrder)
```

在 API 级别 1 中，`query()`方法根据我们提供的参数返回结果集上的游标。以下是前面代码的参数：

+   `uri`：这是我们的情况下的`contentURI`，使用`content://`方案来检索内容。它可以基于 ID 或基于目录。

+   `projection`：这是要返回的列的列表，我们已经使用列名准备好了。传递`null`将返回所有列。

+   `selection`：格式化为 SQL `WHERE`子句，不包括`WHERE`本身，这充当一个过滤器，声明要返回哪些行。

+   `selectionArgs`：我们可以在`selection`中包含`?`参数标记。Android SQL 查询构建器将使用从`selectionArgs`绑定为字符串的值替换`?`参数标记，按照它们在`selection`中出现的顺序。

+   `sortOrder`：这告诉我们如何对行进行排序，格式化为 SQL `ORDER BY`子句。`null`值将使用默认排序顺序。

### 注意

根据官方文档，我们应该遵循一些指导方针以获得最佳性能：

+   提供明确的投影，以防止从存储中读取不会使用的数据。

+   在选择参数中使用问号参数标记，例如`phone=?`，而不是显式值，以便仅由这些值不同的查询将被识别为相同以进行缓存。

我们之前使用的相同过程用于检查`null`值和空游标，并最终从游标中提取所需的值。

现在，让我们看一下我们测试应用程序的`insert`方法。

我们首先构建我们的内容值对象和相关的键值对，例如，在相关的`Columns.TABLE_ROW_PHONENUM`字段中放入电话号码。请注意，因为诸如列名之类的细节以类的形式与我们共享，所以我们不需要担心实际的列名等细节。我们只需要通过`Columns`类的方式访问它。这确保我们只需要更新相关的值。如果将来内容提供程序发生某些更改并更改表名，其余功能和实现仍然保持不变。我们像之前在查询内容提供程序数据的情况下一样，构建我们所需的列名的投影字符串数组。

我们还构建我们的内容 URI；请注意，它与表匹配，而不是单独的行。`insert()`方法也返回一个 URI，不像`query()`方法返回结果集上的游标：

```kt
public void insert(View v) {

  String name = getRandomName();
  String number = getRandomNumber();

  ContentValues values = new ContentValues();
  values.put(Columns.TABLE_ROW_NAME, name);
  values.put(Columns.TABLE_ROW_PHONENUM, number);
  values.put(Columns.TABLE_ROW_EMAIL, name + "@gmail.com");
  values.put(Columns.TABLE_ROW_PHOTOID, "abc");

  String[] projection = new String[] {
      Columns.TABLE_ROW_NAME, Columns.TABLE_ROW_PHONENUM,
      Columns.TABLE_ROW_EMAIL, Columns.TABLE_ROW_PHOTOID };

  Uri contentUri = Uri.parse("content://" + AUTHORITY + "/"
            + BASE_PATH);

  Uri insertedRowUri = getContentResolver().insert(
            contentUri, values);

  //checking the added row
  Cursor cr = getContentResolver().query(insertedRowUri,
         projection, null, null, null);

  if (cr != null) {
      if (cr.getCount() > 0) {
           cr.moveToFirst();
           name = cr.getString(cr.getColumnIndexOrThrow(
               Columns.TABLE_ROW_NAME));
           insertT.setText(name);
      }
  }

}
```

`getRandomName()`和`getRandomNumber()`方法生成要插入表中的随机名称和数字：

```kt
private String getRandomName() {

      Random rand = new Random();
      String name = "" + (char) (122-rand.nextInt(26))
         + (char) (122-rand.nextInt(26))
         + (char) (122-rand.nextInt(26))
         + (char) (122-rand.nextInt(26))
         + (char) (122-rand.nextInt(26))
         + (char) (122-rand.nextInt(26))
         + (char) (122-rand.nextInt(26))
         + (char) (122-rand.nextInt(26)) ;

      return name;
}

public String getRandomNumber() {
  Random rand = new Random();
  String number = rand.nextInt(98989)*rand.nextInt(59595)+"";

  return number;
}
```

让我们更仔细地看看`insert()`方法：

```kt
public final Uri insert (Uri url, ContentValues values)
```

以下是上一行代码的参数：

+   `url`：要插入数据的表的 URL

+   `values`：以`ContentValues`对象的形式为新插入的行的值，键是字段的列名

请注意，在插入后，我们再次运行了`query()`方法，使用了`insert()`方法返回的 URI。我们运行这个查询是为了看到我们打算插入的值是否已经插入；这个查询将根据附加了 ID 的行的投影返回列。

到目前为止，我们已经涵盖了`query()`和`insert()`方法；现在，我们将涵盖`update()`方法。

我们在`insert()`方法中通过准备`ContentValues`对象来进行了进展。类似地，我们将准备一个对象，我们将在`ContentResolver`的`update()`方法中使用来更新现有行。在这种情况下，我们将构建我们的 URI 直到 ID，因为这个操作是基于 ID 的。更新由`rowUri`对象指向的行，它将返回更新的行数，这将与 URI 相同；在这种情况下，它是指向单个行的`rowUri`。另一种方法可能是使用指向表的`contentUri`和`selection`/`selectionArgs`的组合。在这种情况下，根据`selection`子句，更新的行可能多于一个：

```kt
public void update(View v) {

  String name = getRandomName();
  String number = getRandomNumber();

  ContentValues values = new ContentValues();
  values.put(Columns.TABLE_ROW_NAME, name);
  values.put(Columns.TABLE_ROW_PHONENUM, number);
  values.put(Columns.TABLE_ROW_EMAIL, name + "@gmail.com");
  values.put(Columns.TABLE_ROW_PHOTOID, " ");

  Uri contentUri = Uri.parse("content://" + AUTHORITY
                    + "/" + BASE_PATH);
  Uri rowUri = ContentUris.withAppendedId(
                    contentUri, getFirstRowId());
  int count = getContentResolver().update(rowUri, values, null, null);

}
```

让我们更仔细地看看`update()`方法：

```kt
public final int update (Uri uri, ContentValues values, String where, String[] selectionArgs)
```

以下是上一行代码的参数：

+   `uri`：这是我们希望修改的内容 URI

+   `values`：这类似于我们之前在其他方法中使用的值；传递`null`值将删除现有字段值

+   `where`：作为过滤器对行进行更新之前的 SQL `WHERE`子句

我们可以再次运行`query()`方法来查看更改是否反映出来；这个活动留给你作为练习。

最后一个方法是`delete()`，我们需要它来完成我们的 CRUD 方法。`delete()`方法的开始方式与其他方法类似；首先，准备我们的内容 URI 在目录级别，然后在 ID 级别构建它，也就是在单个行级别。之后，我们将其传递给`ContentResolver`的`delete()`方法。与`query()`和`insert()`方法返回整数值不同，`delete()`方法删除由我们基于 ID 的内容 URI 对象`rowUri`指向的行，并返回删除的行数。在我们的情况下，这将是`1`，因为我们的 URI 只指向一行。另一种方法可能是使用指向表的`contentUri`和`selection`/`selectionArgs`的组合。在这种情况下，根据`selection`子句，删除的行可能多于 1：

```kt
public void delete(View v) {

      Uri contentUri = Uri.parse("content://" + AUTHORITY
                              + "/" + BASE_PATH);
      Uri rowUri = contentUri = ContentUris.withAppendedId(
                              contentUri, getFirstRowId());
      int count = getContentResolver().delete(rowUri, null,
               null);
}
```

UI 和输出如下：

![使用内容提供程序](img/2951_03_03.jpg)

### 注意

如果你想更深入地了解 Android 内容提供程序是如何在各个表之间管理各种写入和读取调用的（提示：它使用`CountDownLatch`），你可以查看 Coursera 上 Douglas C. Schmidt 博士的视频以获取更多信息。视频可以在[`class.coursera.org/posa-002/lecture/49`](https://class.coursera.org/posa-002/lecture/49)找到。

# 总结

在本章中，我们介绍了内容提供程序的基础知识。我们学习了如何访问系统提供的内容提供程序，甚至我们自己的内容提供程序版本。我们从创建一个基本的联系人管理器，逐渐发展成为 Android 生态系统中的一个完整的成员，通过实现`ContentProvider`来在其他应用程序之间共享数据。

在接下来的章节中，我们将介绍`Loaders`、`CursorAdapters`、巧妙的技巧和提示，以及一些开源库，以使我们在使用 SQLite 数据库时更轻松。
