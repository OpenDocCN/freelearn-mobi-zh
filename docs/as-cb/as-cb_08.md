# 第八章。提高质量

您刚刚完成了应用的编码。现在呢？尽快将其放到 Play 商店上！

不要等待，您还没有完成！您是否正确测试了您的应用？它是否适用于任何 Android 版本？在任何设备上？在任何情况下？

在本章中，我们将重点关注：

+   模式和支持注释

+   使用 Robolectrics 进行单元测试

+   代码分析

# 介绍

有一些常见的陷阱要避免，以及一些模式，您可能希望应用以提高应用程序的质量。您已经在之前的章节中看到了其中一些。此外，还有一些有趣的工具可以用来测试和分析您的代码。

在接下来的路线图中，您会注意到在将应用上线之前，您需要完成不同的阶段：

![介绍](img/B04299_08_01.jpg)

您的代码结构、健壮性、可维护性以及其与功能要求的符合程度是关键因素。

功能质量通过软件测试来衡量，因此我们需要将应用分发给我们的测试人员。我们将在第十章中讨论这一点，*测试您的应用程序*。

通过运行单元测试和手动代码检查（同行审查）或使用诸如 Android Lint 之类的工具来评估结构质量，您将在本章的最后一个配方中了解更多有关它。现在的问题是代码架构是否满足良好软件工程的要求？

总的来说，有一些有趣的原则将帮助您提高代码的质量。其中一些列在这里：

+   学习活动生命周期，并以正确的方式使用片段。

+   如果可以避免，就不要分配内存。

+   避免过于沉重的片段和活动。

+   考虑**模型视图控制器**（**MVC**）方法。应用正确的模式。

+   在一个地方解决一次问题。**不要重复自己**（**DRY**）。

+   不要做不需要做的工作（尚未）。也被称为：**你不会需要它**（**YAGNI**）。

下一个配方将让您了解模式是什么，以及为什么您会想要应用它们。

# 模式和支持注释

质量是一项严肃的业务，因此我们将把它与一些乐趣结合起来。在即将到来的配方中，我们将创建一个测验应用。我们将使用 Google Play 服务进行此操作，并且我们将研究可以应用于我们的应用的模式，特别是 MVC 和**模型视图控制器**（**MVP**）方法。

那么设计模式实际上是什么？设计模式是常见问题的解决方案。我们可以在任何地方重用这样的模式。没有必要重新发明轮子（除非您当然可以想到更好的轮子），也没有必要重复自己。

模式是我们可以信任的最佳实践。它们可以帮助我们加快开发过程，包括测试。

一些模式包括：

+   MVC

+   MVP

+   可观察的

+   工厂

+   单例

+   支持注释

+   Google Play 服务

## MVC

MVC 最适合较大的项目。这种模式的好处是关注点的分离。我们可以将 UI 代码与业务逻辑分开。控制器将负责显示哪个视图。它将从另一层获取数据，一个类似存储库的类，该类将从某处获取其数据，并通过模型（或模型列表）将数据传递给 UI。控制器不知道数据来自何处以及如何显示。这些是存储库类和 UI 的任务，分别。

## MVP

在大多数情况下，MVP 是与 Android 应用程序开发一起使用的更合适的模式，因为活动和片段的性质。使用 MVP 模式，一个 Presenter 包含视图的 UI 逻辑。视图的所有调用都直接委托给它。Presenter 将通过接口与视图通信，允许我们稍后使用模拟数据创建单元测试。

## 观察者模式

我们在第七章中已经看到了这种模式，*内容提供者和观察者*。观察者观察另一个对象的变化。

## 工厂模式

这种模式有助于创建对象。我们之前使用过的位图工厂（并且我们将在本教程中再次使用）是工厂模式的一个很好的例子。

## 单例

单例模式将防止我们拥有对象的多个实例。通常，它是一个（类）方法，返回一个实例。如果它不存在，它将被创建，否则它将返回先前创建的实例。应用程序类就是单例模式的一个例子。

## 支持注释

支持注释可以帮助我们向代码检查工具（如 lint）提供提示。它们可以帮助您通过添加元数据标签并运行代码检查来检测问题，例如空指针异常和资源类型冲突。支持库本身已经用这些注释进行了注释。是的，他们自己也在使用注释，这证明使用注释是正确的方法。

基本上有三种我们可以使用的注释类型：空值注释、资源类型注释和 IntDef \ StringDef 注释。例如，我们可以使用`@NonNull`注释来指示给定参数不能为空，或者我们可以使用`@Nullable`注释来指示返回值可以为空。

## Google Play 服务

Play Games SDK 提供跨平台的 Google Play 游戏服务，让您可以轻松地在平板电脑和移动设备游戏中集成流行的游戏功能，例如成就、排行榜、保存的游戏和实时多人游戏（在 Android 上）选项。

现在理论已经足够了！让我们创建我们的测验应用程序，并应用我们在这里讨论过的一些理论。

## 准备工作

对于本教程，您需要拥有最新版本的 Android Studio 和已安装 Google Play 服务的真实设备，这对大多数设备来说都是成立的。或者，您可以在虚拟 Genymotion 设备上安装它们，但这将需要一些额外的准备工作。

此外，您需要拥有（或创建）一个 Google 开发者帐户。

## 如何做...

然后开始。启动 Android Studio 并执行以下步骤，因为我们将要构建一些伟大的东西：

1.  在 Android Studio 中创建一个新项目。命名为`GetItRight`，然后点击**下一步**按钮。

1.  选择**手机和平板电脑**选项，然后点击**下一步**按钮。

1.  在**为移动设备添加活动**视图中，选择**Google Play 服务**，然后点击**下一步**按钮。

1.  接受**活动名称**和**标题**字段，然后点击**完成**按钮。

1.  将您的网络浏览器指向 Google 开发者控制台，如果您还没有帐户，请登录或注册。您可以在以下网址找到它：[`console.developers.google.com`](https://console.developers.google.com)。

1.  在开发者控制台中，点击游戏选项卡（网页左侧的游戏图标）。

1.  如果被要求，接受服务条款。

1.  点击**设置 Google Play 服务**按钮。

1.  输入应用程序名称`Get It Right Sample`，选择一个类别：**问答**，然后点击**继续**按钮。

1.  在游戏详情视图中，输入描述，然后点击**保存**按钮。

1.  接下来，您需要生成一个 Oauth2 客户端 ID。要这样做，请点击**关联应用**链接。

1.  选择**Android**作为您的操作系统，输入`packt.com.getitright`作为**包名称**，保持其他设置不变，然后点击**保存并继续**按钮。

1.  在第 2 步中，点击**立即授权您的应用**按钮。在**品牌信息**弹出对话框中，点击**继续**按钮。

1.  **客户端 ID**对话框出现。输入`packt.com.getitright`作为包名称。要获取签名证书指纹，打开**终端应用程序**（对于 Windows：命令提示符）并输入：

```kt
keytool -exportcert -alias androiddebugkey -keystore ~/.android/debug.keystore  -list –v

```

1.  如果要求`keystore`密码，默认的调试 keystore 密码是`android`。

1.  复制并粘贴指纹（SHA1），然后点击**创建客户端**按钮。

1.  点击**返回列表**按钮，然后点击**继续下一步**按钮。

1.  在**Android 应用程序详细信息**视图中，您将看到**应用程序 ID**（如果向下滚动一点），我们稍后将需要它。复制其值。

### 排行榜

按照提供的步骤为应用程序添加排行榜：

1.  在网页的左侧，选择**排行榜**，然后点击**添加新排行榜**按钮。将新排行榜命名为`GetItRight Leaderboard`，然后点击**保存**按钮。注意排行榜**ID**。我们稍后会用到它：![排行榜](img/B04299_08_02.jpg)

1.  打开项目`app`目录内的`build.gradle`文件，并添加 Google Play 服务的依赖项：

```kt
compile 'com.google.android.gms:play-services:7.5.0'
```

1.  同步您的项目。如果无法解析 Google Play 服务，将生成一个错误，其中包含一个链接，上面写着**安装存储库并同步项目**。点击此链接进行操作。

1.  打开`AndroidManifest.xml`文件，并向应用程序标签添加元数据标记：

```kt
<meta-data 
 android:name="com.google.android.gms.games.APP_ID"android:value="@string/app_id" />
```

1.  此外，将`app_id`添加到`strings.xml`文件中：

```kt
<resources><string name="app_name">GetItRight</string><string name="app_id">your app id</string>
```

1.  在`GooglePlayServicesActivity`类的`onConnected`方法的第一行设置断点。对于`onConnectionFailed`方法的第一行也是如此。使用 Google Play 服务模板和提供的应用 ID，您应该已经能够连接到 Google Play 服务。运行应用程序（调试模式）以查看是否成功。

1.  创建一个新的 Android 资源目录，并选择**layout**作为资源类型；在该目录中创建一个新的布局资源文件，并命名为`activity_google_play_services.xml`。

1.  向`strings.xml`资源文件添加一些新的字符串：

```kt
<string name="incorrect_answer">That is incorrect</string><string name="correct_answer">That is the correct 
 answer!</string><string name="leader_board">LEADER BOARD</string>
```

1.  为`activity_google_play_service`资源文件创建布局：

```kt
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android=
   "http://schemas.android.com/apk/res/android"android:orientation="vertical"  
    android:layout_width="match_parent"android:padding="16dp"
    android:background="@android:color/holo_blue_dark"
    android:layout_height="match_parent">
    <ScrollView
      android:layout_width="match_parent"
      android:layout_height="wrap_content"><LinearLayout
         android:orientation="vertical"android:layout_width="match_parent"android:layout_height="wrap_content">
          <ImageView
            android:id="@+id/image"android:src= 
             "@android:drawable/ic_popup_sync"android:layout_width="match_parent"android:layout_height="300px" />
          <TextView
            android:id="@+id/text"android:textColor="@android:color/white"android:text="Question"android:textSize="24sp"android:layout_width="match_parent"android:layout_height="wrap_content" />
          <LinearLayout
            android:orientation="vertical"android:layout_width="match_parent"android:layout_height="wrap_content">
          <Button
            android:id="@+id/button_1"            
            android:layout_width="match_parent"android:layout_height="wrap_content"android:gravity="center_vertical|left" />
          <Button
            android:id="@+id/button_2"android:layout_width="match_parent"android:layout_height="wrap_content"                     
            android:gravity="center_vertical|left" />
          <Button
           android:id="@+id/button_3"android:layout_width="match_parent"android:layout_height="wrap_content"android:gravity="center_vertical|left" />
          <Button
           android:id="@+id/button_4"           
           android:layout_width="match_parent"android:layout_height="wrap_content"android:gravity="center_vertical|left" />
          <Button
           android:id="@+id/button_test"android:text="@string/leader_board"android:layout_width="match_parent"android:layout_height="wrap_content"android:gravity="center_vertical|left" />
          </LinearLayout>
        </LinearLayout>
    </ScrollView>
</LinearLayout>
```

1.  打开`GooglePlayServicesActivity`文件。在`onCreate`方法中，加载布局并为所有按钮设置点击监听器：

```kt
setContentView(R.layout.activity_google_play_services); 
findViewById(R.id.button_1).setOnClickListener(this);
findViewById(R.id.button_2).setOnClickListener(this);
findViewById(R.id.button_3).setOnClickListener(this);
findViewById(R.id.button_4).setOnClickListener(this); 
findViewById(R.id.button_test).setOnClickListener(this);
```

1.  为`GooglePlayServicesActivity`文件实现`onClickListener`方法。Android Studio 将建议一个实现，您可以接受此建议，也可以自己添加实现：

```kt
public class GooglePlayServicesActivity extends Activity implements GoogleApiClient.ConnectionCallbacks,GoogleApiClient.OnConnectionFailedListener, 
   View.OnClickListener { 
@Override
public void onClick(View v) {
}
```

1.  添加两个私有成员，一个用于我们的排行榜请求，另一个用于保存您的排行榜 ID：

```kt
private int REQUEST_LEADERBOARD = 1;
private String LEADERBOARD_ID = "<your leaderboard id>";
```

1.  为`onClick`方法创建实现。我们正在准备用户点击多项选择选项的情况。对于**排行榜**（测试）按钮，我们可以立即添加实现：

```kt
@Override
public void onClick(View v) {
    switch (v.getId()){
        case R.id.button_1:
        case R.id.button_2:
        case R.id.button_3:
        case R.id.button_4: 
            break;
        case R.id.button_test:
         startActivityForResult( 
          Games.Leaderboards.getLeaderboardIntent(  
           mGoogleApiClient, LEADERBOARD_ID),  
            REQUEST_LEADERBOARD);
         break;
    }
}
```

1.  创建一个新的包并命名为`models`。创建`Answer`、`Question`和`Quiz`类：

要添加`Answer`类，您需要以下代码：

```kt
public class Answer {
    private String mId;
    private String mText;
    public String getId() {
        return mId;
    }
    public String getText() {
       return mText;
    }
    public Answer (String id, String text) {
        mId = id;
        mText = text;
    }
}
```

要添加`Question`类，请使用以下代码：

```kt
public class Question {
    private String mText;
    private String mUri;
    private String mCorrectAnswer;
    private String mAnswer;
    private ArrayList<Answer> mPossibleAnswers;
    public String getText(){
        return mText;
    }
    public String getUri(){
        return mUri;}
    public String getCorrectAnswer(){
        return mCorrectAnswer;
    }
    public String getAnswer(){
        return mAnswer;
    }
    public Question (String text, String uri, String 
     correctAnswer){
        mText = text;
        mUri = uri;
        mCorrectAnswer = correctAnswer;
    }
    public Answer addAnswer(String id, String text){
        if (mPossibleAnswers==null){
            mPossibleAnswers = new ArrayList<Answer>();
        }
        Answer answer = new Answer(id,text);
        mPossibleAnswers.add(answer);
        return answer;
    }
    public ArrayList<Answer> getPossibleAnswers(){
        return mPossibleAnswers;
    }
}
```

要添加`Quiz`类，请使用以下代码：

```kt
public class Quiz {
    private ArrayList<Question> mQuestions;
    public ArrayList<Question> getQuestions(){
        return mQuestions;
    }
    public Question addQuestion(String text, String uri, String 
     correctAnswer){
        if (mQuestions==null){
            mQuestions = new ArrayList<Question>();
        }
        Question question = new Question( 
         text,uri,correctAnswer);
        mQuestions.add(question);
        return question;
    }
}
```

1.  创建一个新的包并命名为`repositories`。创建一个新的类并命名为`QuizRepository`。向测验添加一些问题。您可以使用以下示例中的问题，但如果愿意，也可以自己创建一些问题。在真实的应用程序中，问题和答案当然不会是硬编码的，而是从数据库或后端检索的（请注意，我们随时可以更改此行为，而无需修改除此类之外的任何内容）：

```kt
public class QuizRepository {
    public Quiz getQuiz(){
      Quiz quiz = new Quiz();
      Question q1 = quiz.addQuestion(
      "1\. What is the largest city in the world?",  
       "http://cdn.acidcow.com/pics/20100923/
        skylines_of_large_cities_05.jpg" , "tokyo");
        q1.addAnswer("delhi" , "Delhi, India");
        q1.addAnswer("tokyo" , "Tokyo, Japan");
        q1.addAnswer("saopaulo" , "Sao Paulo, Brazil");
        q1.addAnswer("nyc" , "New York, USA");
        Question q2 = quiz.addQuestion("2\. What is the largest animal in the world?","http://www.onekind.org/uploads/a-z/az_aardvark.jpg" , "blue_whale");
        q2.addAnswer("african_elephant" , "African Elephant");
       q2.addAnswer("brown_bear" , "Brown Bear");
        q2.addAnswer("giraffe" , "Giraffe");
        q2.addAnswer("blue_whale" , "Blue whale");
        Question q3 = quiz.addQuestion("3\. What is the highest mountain in the world?","http://images.summitpost.org/medium/ 815426.jpg", "mount_everest");
        q3.addAnswer("mont_blanc" , "Mont Blanc");
        q3.addAnswer("pico_bolivar" , "Pico Bolívar");
        q3.addAnswer("mount_everest" , "Mount Everest");
        q3.addAnswer("kilimanjaro" , "Mount Kilimanjaro");
        return quiz;
    }
}
```

1.  在`GamePlayServicesActivity`类中，添加这三个私有成员：

```kt
private Quiz mQuiz;
private int mScore;
private int mQuestionIndex=0;
```

1.  为`newGame`方法添加实现。我们将通过向存储库请求来获取`Quiz`对象。重置分数和问题索引后，我们调用`displayQuestion`方法，该方法通过实际显示问题、可能的答案和漂亮的图片来实现 UI 逻辑：

```kt
private void newGame(){
    mQuiz = new QuizRepository().getQuiz();
    mScore = 0;
    mQuestionIndex = 0;
    displayQuestion(mQuiz.getQuestions().get(mQuestionIndex));
private void displayQuestion(Question question){ 
    TextView questionText = (TextView)findViewById(R.id.text); 
    displayImage(question); 
    questionText.setText(question.getText());
    ArrayList<Answer> answers = question.getPossibleAnswers();
    setPossibleAnswer(findViewById(R.id.button_1), 
     answers.get(0));
    setPossibleAnswer(findViewById(R.id.button_2), 
     answers.get(1));
    setPossibleAnswer(findViewById(R.id.button_3), answers.get(2));
    setPossibleAnswer(findViewById(R.id.button_4), answers.get(3));
}
private void setPossibleAnswer(View v, Answer answer){
    if (v instanceof Button) {
        ((Button) v).setText(answer.getText());
        v.setTag(answer);
    }
}
private void displayImage(final Question question){ 
    new Thread(new Runnable() {
        public void run(){
            try {
              URL url = new URL(question.getUri());
              final Bitmap image = BitmapFactory.decodeStream(url.openConnection().getInputStream());
               runOnUiThread(new Runnable() {
                    @Override
                    public void run() {
                        ImageView imageView = (ImageView) 
                          findViewById(R.id.image);
                        imageView.setImageBitmap(image);
                    }
                });
            }
            catch (Exception ex){
                Log.d(getClass().toString(), ex.getMessage());
            }
        }
    }).start();
}
```

### 让游戏开始！

以下步骤可用于添加新游戏的方法：

1.  在`onCreate`方法的末尾，我们将调用`newGame`方法：

```kt
newGame();
```

1.  修改`onClick`方法，这样当用户点击任何按钮时我们可以做出响应。如果点击了任何一个多选按钮，我们将调用`checkAnswer`方法。我们选择的是正确答案吗？多么令人兴奋：

```kt
@Override
public void onClick(View v) {
    switch (v.getId()){
        case R.id.button_1:
        case R.id.button_2:
        case R.id.button_3:
        case R.id.button_4:
            checkAnswer(v);
            break;
        case R.id.button_test: startActivityForResult( 
         Games.Leaderboards.getLeaderboardIntent(
          mGoogleApiClient, LEADERBOARD_ID), REQUEST_LEADERBOARD);
         break;
}

   }
```

1.  添加`checkAnswer`方法。我们将比较给定答案与问题的正确答案，根据结果，我们将调用`onGoodAnswer`或`onWrongAnswer`方法。根据答案，你的进展将被决定：如果答案错误，游戏结束，我们将显示排行榜。

1.  如果没有更多的问题，我们将提交用户的分数并显示排行榜。排行榜本身将处理所有相关逻辑。提交的分数是否足够高，使你的名字出现在榜单的顶部？通过以下片段来检查：

```kt
private void checkAnswer(View v){ 
    if (v instanceof Button){
        Answer answer = (Answer)((Button)v).getTag();
        if (mQuiz.getQuestions().get(mQuestionIndex).  
         getCorrectAnswer().equalsIgnoreCase( 
          answer.getId())){
            onGoodAnswer();
        }
        else{
            onWrongAnswer();
        }
    }
}
private void onWrongAnswer(){
    Toast.makeText(this, getString( 
     R.string.incorrect_answer), Toast.LENGTH_SHORT).show();
    startActivityForResult(
     Games.Leaderboards.getLeaderboardIntent( 
     mGoogleApiClient, LEADERBOARD_ID), 
      REQUEST_LEADERBOARD);
}
private void onGoodAnswer(){
    mScore+= 1000;
    Games.Leaderboards.submitScore(mGoogleApiClient, 
     LEADERBOARD_ID, mScore);
    Toast.makeText(this, getString(R.string.correct_answer), 
     Toast.LENGTH_SHORT).show();
    mQuestionIndex++;
    if (mQuestionIndex < mQuiz.getQuestions().size()){
        displayQuestion(mQuiz.getQuestions().get( 
         mQuestionIndex));
    }
    else{
        startActivityForResult( 
         Games.Leaderboards.getLeaderboardIntent( 
          mGoogleApiClient, LEADERBOARD_ID), 
           REQUEST_LEADERBOARD);
	}
}
```

1.  为了做好单元测试和代码检查，让我们添加注释支持。在`app`文件夹中打开`build.gradle`文件并添加依赖项。在修改文件后，点击出现的**立即同步**链接：

```kt
compile 'com.android.support:support-annotations:22.2.0'
```

1.  如果出现“无法解析支持注释”的错误，则点击出现的**安装存储库并同步项目**链接。

1.  如果一切顺利，我们可以添加注释，例如在`CheckAnswer`方法的参数上：

```kt
private void checkAnswer(@NonNull View v){
```

1.  在`Question`类中，我们可以为`getPossibleAnswers`方法添加`@Nullable`注释，如果我们没有为问题提供任何多选选项的话，这可能是情况：

```kt
@Nullable
public ArrayList<Answer> getPossibleAnswers(){
    return mPossibleAnswers;
}
```

1.  稍后，如果我们进行一些分析，这将导致`GooglePlayServiceActivity`出现警告，我们将在*代码分析*中更仔细地查看这一点：

```kt
Method invocation 'answers.get(0)' may produce  'java.lang.NullPointerException' 
```

如果你喜欢，你可以玩这个游戏并添加一些注释。只是不要花太多时间。我们来玩游戏吧！

运行你的应用程序，并成为排行榜上的第一名。因为目前你是唯一的测试玩家，我猜这不会太难。

你刚刚创建了自己的测验应用程序，如果你愿意，可以添加一些其他具有挑战性的问题，如下面的屏幕截图所示：

![让游戏开始吧！](img/B04299_08_03.jpg)

我们已经调查了 Google Play 服务，并且我们一直在为我们的应用使用 MVC 方法。此外，我们还研究了如何使用注释，在进行一些代码分析后，这些注释可以帮助我们改进代码。

## 还有更多...

我们只是匆匆一瞥了一下模式以及如何应用它们。查看互联网或获取一些优秀的书籍，以了解更多关于模式的知识。另外，请参阅[`www.google.com/design/spec/patterns/app-structure.html`](https://www.google.com/design/spec/patterns/app-structure.html)。

确保你也阅读了有关支持注释的文档。使用它们有很多更多的可能性。在[`tools.android.com/tech-docs/support-annotations`](http://tools.android.com/tech-docs/support-annotations)中查看文档。

此外，我们只使用了 Google Play 服务的一小部分。我们只知道如何登录和如何使用排行榜。如果你愿意，你可以查看其他选项。有关此内容，请参阅[`developers.google.com/games/services/android/quickstart`](https://developers.google.com/games/services/android/quickstart)。

## 另请参阅

+   参见第七章，*内容提供者和观察者*。

# 使用 Robolectric 进行单元测试

单元测试是一种测试方法，其中测试代码的各个单元。例如，可以测试视图或存储库，以检查它是否满足要求。与大多数其他测试不同，这些测试通常由软件开发人员开发和运行。

理想情况下，一个测试用例完全独立于其他用例和其他单元。由于类通常依赖于其他替代品，例如需要使用模拟对象。在上一个教程中，`QuizRepository`类提供了硬编码的测验数据（存根或模拟数据），但正如建议的那样，意图是测验数据应该从后端获取。

我们将准备好我们在上一个教程中创建的应用程序进行单元测试，并且我们将自己创建一些测试。**Robolectric**将帮助我们完成这些。尽管自 Android Studio 1.2 版本发布以来，基于 JUnit 的单元测试设置变得更加容易，但它仍然不如 Robolectric 强大。

Robolectric 不需要额外的模拟框架，它也可以在模拟器之外运行，这使我们能够将单元测试与持续集成环境结合起来，就像我们将在第十章中所做的那样，*测试你的应用程序*。

## 准备工作

对于这个教程，最理想的情况是上一个教程已经成功完成。如果你选择跳过本章的这一部分，当然可以打开你自己的项目，并以更或多或少相同的方式设置单元测试。这取决于你。

## 如何做...

那么我们要做些什么来创建和运行一些单元测试呢？让我们找出来：

1.  打开我们在上一个教程中创建的项目。

1.  在`app`文件夹中打开`build.gradle`文件，并为 Robolectric 添加一个依赖项：

```kt
testCompile 'org.robolectric:robolectric:3.0'
```

1.  将`src`文件夹中的`androidTest`文件夹重命名为`test`。

1.  从**Run**菜单中选择**Edit configurations**选项。

1.  在 Run\Debug Configuration 窗口的左侧，选择**Defaults**和**JUnit**。在右侧将**Working directory**的内容更改为`$MODULE_DIR$`，然后点击**OK**按钮。

1.  将**ApplicationTest**类重命名为`QuizRepositoryTest`。

1.  向**QuizRepositoryTest**类添加一些测试。我们将使用 Robolectric 进行这项工作。正如你所注意到的，我们将在这里使用注解，就像我们在上一个教程中所做的那样：

```kt
@Config(constants = BuildConfig.class, sdk = 21)
@RunWith(RobolectricGradleTestRunner.class)
public class QuizRepositoryTest {
    private QuizRepository mRepository; 
    @Beforepublic void setup() throws Exception {
       mRepository = new QuizRepository();
        assertNotNull("QuizRepository is not 
        instantiated", mRepository);
    }
    @Test
    public void quizHasQuestions() throws Exception {
        Quiz quiz = mRepository.getQuiz();
        ArrayList<Question> questions = quiz.getQuestions();
        assertNotNull("quiz could not be created", quiz);

        assertNotNull("quiz contains no questions",       
         questions);
        assertTrue("quiz contains no questions", 
         questions.size()>0);
    }
    @Test
    public void quizHasSufficientQuestions() throws 
     Exception {
        Quiz quiz = mRepository.getQuiz();
        ArrayList<Question> questions = quiz.getQuestions();
        assertNotNull("quiz could not be created", quiz);
        assertNotNull("quiz contains no questions", 
         questions);
        assertTrue("quiz contains insufficient questions", questions.size()>=10);
    }
}
```

1.  创建另一个测试类，以便我们可以测试该活动。将新类命名为`GooglePlayServicesActivityTest`。在这个测试中，我们也可以进行一些布局测试：

```kt
@Config(constants = BuildConfig.class, sdk = 21)
@RunWith(RobolectricGradleTestRunner.class)
public class GooglePlayServicesActivityTest {
    private GooglePlayServicesActivity activity;
    @Before
    public void setup() throws Exception {
       activity = Robolectric.setupActivity( 
        GooglePlayServicesActivity.class);
        assertNotNull("GooglePlayServicesActivity is not instantiated", activity);
    }
    @Test
    public void testButtonExistsAndHasCorrectText() throwsException {
        Button testButton = (Button) activity.findViewById( 
         R.id.button_test); 
        assertNotNull("testButton could not be found",testButton); 
}
```

1.  打开`build variants`窗格，并选择`Unit tests`而不是`Instrumentation tests`。

现在`test`包中的所有内容都将被突出显示为绿色（你可能需要先进行重建）。如果你右键单击`packt.com.getitright`包名或者你创建的任何测试类，你将在上下文菜单中找到一个选项**Run tests in packt.com.getright**或**Run QuizRepositoryTest**。例如，选择运行`QuizRepositoryTest`。如果选择此选项，Gradle 会开始思考一会儿。一段时间后，结果会显示出来。

默认情况下只显示失败的测试。要查看成功的测试，点击左侧显示测试树上方的**Hide passed**按钮。

你会看到**quizHasQuestions**测试已经通过。然而，**quizHasSufficientQuestions**测试失败了。这是有道理的，因为我们的测试要求我们的测验至少有 10 个问题，而我们只添加了三个问题到测验中，如下图所示：

![如何做...](img/B04299_08_04.jpg)

在`QuizRepository`中为`Quiz`添加七个问题，以便做对。当然，你也可以作弊，通过修改测试来达到目的，但我们就说这是一个业务需求吧。

重新运行测试。每个单元测试都成功了。万岁！创建一些你能想到的其他单元测试。

单元测试是一个非常有趣的选择，因为我们也可以将其用于持续集成的目的。想象一下，每次您将源代码提交（和推送）到 GitHub 或 BitBucket 等中央存储库时，我们都运行单元测试的情景。如果编译和所有单元测试都成功，我们可以自动创建一个新的（临时的）发布，或者被通知编译或任何测试失败。

## 还有更多...

还有很多其他工具和方法可用于移动测试目的。

除了单元测试，我们还希望测试**用户界面**（**UI**），例如使用 Espresso。

### Espresso

Espresso 适用于编写简洁可靠的 Android UI 测试。测试通常包含点击、文本输入和检查。编写测试实际上非常简单。以下是使用 Espresso 的测试示例：

```kt
@Test
public void testLogin() {
   onView(withId(R.id.login)).perform(
    typeText("mike@test.com"));
   onView(withId(R.id.greet_button)).perform(click());
}
```

引用网站上的话：

> “*Espresso 测试清楚地陈述期望、交互和断言，而不受到样板内容、自定义基础设施或混乱的实现细节的干扰*”。

有关更多信息，请参阅[`code.google.com/p/android-test-kit/wiki/Espresso`](https://code.google.com/p/android-test-kit/wiki/Espresso)。

### 方法

在测试方面，有不同的方法可以考虑。其中一种方法是**测试驱动开发**（**TDD**）。如果功能和所有要求都已知，我们可以在开发应用程序之前定义我们的测试。当然，所有测试最初都会失败，但这实际上是件好事。它将概述需要做的事情，并集中精力做正确的事情。如果您开始开发得越来越多，测试将成功，剩下的工作量也会减少。

另一种更近期的方法是**行为驱动开发**（**BDD**）。这种测试方法是基于功能的，其中一个功能是从特定的角度表达的一系列故事。

BDD 工具可以作为单元测试的一种风格，例如`Rspec`，也可以作为更高级别的验收测试风格：`Cucumber`。

### Cucumber、Gherkin 和 Calabash

不，这不是突然出现在这里的蔬菜店广告。**Cucumber**是一种以 BDD 风格编写的自动化验收测试的工具。它允许执行以业务面向文本编写的功能文档。

以下是使用**Gherkin**的功能文件的示例。它有两个目的：文档和自动化测试：

```kt
Scenario: Login
  Given I am on the Login Screen
  Then I touch the "Email" input field
  Then I use the keyboard and type "test@packt.com"
  Then I touch the "Password" input field
  Then I use the keyboard and type "verysecretpassword"
  Then I touch "LOG IN"
  Then I should see "Hello world"
```

`Gherkin`是一种可读性强的领域特定语言，它可以让您描述软件的行为，而不详细说明该行为是如何实现的。因此，非开发团队成员也可以编写这些测试。

需要一些粘合代码来使事情发生。在 Cucumber 中，这个过程是在步骤定义中定义的。Cucumber 通常让您用 Ruby 语言编写这些步骤定义。

通过 Calabash 框架，您可以使用 Cucumber 为 Android 和 iOS 创建测试。它使您能够定义和执行自动化验收测试。Calabash 的另一个很棒的地方是，它允许您在云上运行自动化测试，例如使用 TestDroid 的服务。

### 首先要做的事情！

要了解有关 Cucumber 的更多信息，请访问[`cucumber.io`](https://cucumber.io)。

您可以在[`calaba.sh`](http://calaba.sh)找到 Calabash 框架。

还可以查看[www.testdroid.com](http://www.testdroid.com)了解有关使用 TestDroid 云测试环境在尽可能多的设备上进行测试的更多信息。

最后，要在时间、质量和金钱之间找到一个良好的平衡。测试应用程序的方法取决于您（或您的公司或您的客户）认为这些元素中的每个元素有多有价值。至少创建单元测试和 UI 测试。还要不要忘了性能测试，但这是下一章将讨论的一个话题！

## 另请参阅

+   参考第九章，*性能改进*

+   参考第十章，*测试您的应用程序的 Beta 版*

# 代码分析

代码分析工具，如 Android Lint，可以帮助你检测潜在的错误，以及如何优化你的应用程序的安全性、可用性和性能。

Android Lint 随 Android Studio 一起提供，但也有其他可用的工具，如：Check Style，**项目** **Mess Detector**（**PMD**）和 Find Bugs。在这个示例中，我们只会看一下 Android Lint。

## 准备工作

+   最理想的情况是，你已经完成了本章的前两个示例，所以我们现在将检查应用的结果。但是，你也可以在任何项目上使用`Android Lint`（或其他工具）来查看哪里可以改进。

### 注意

第一个示例的支持注解影响了显示的结果。是的，没错，我们引起了这些警告。

## 操作步骤...

我们不需要安装任何东西来获取 Android Lint 报告，因为它已经在 Android Studio 中了。只需按照下一步骤来使用它：

1.  打开你在之前示例中创建的项目。或者，打开你自己的项目。

1.  从**分析**菜单中选择**代码检查**。检查范围是整个项目。单击**确定**按钮继续。

1.  检查结果将以树形视图呈现。展开并选择项目以查看每个项目的内容，如下面的快照所示：![操作步骤...](img/B04299_08_05.jpg)

1.  这里看起来很严重，但实际上并不是那么糟糕。有一些问题根本不是致命错误，但修复它们可以极大地改进你的代码，这正是我们目前的目标。

1.  例如，查看**声明冗余** | **声明访问可以更弱** | **可以是私有**问题。导航到它。双击它跳转到问题出现的代码。右键单击它。上下文菜单立即提供了解决方案。选择**使字段私有**选项应用正确的解决方案。如果这样做，此项目将被标记为已完成（划掉）。

1.  现在看看**硬编码文本**。如果你双击与此问题相关的任何项目，你会看到问题所在。

1.  为了方便起见，我们放置了一个临时文本（如`Text View`中的`Question`）。如果这是真的，我们应该使用一个字符串资源。在这里，我们可以安全地删除这个文本。如果你重新运行代码检查，问题将消失：

```kt
<TextView
    android:id="@+id/text"android:textColor="@android:color/white"android:textSize="24sp"android:layout_width="match_parent"
    android:layout_height="wrap_content" />
```

1.  接下来，看看**常量条件和异常**下的**可能的错误**。对于`GooglePlayServicesActivity`文件，它说：

```kt
Method invocation 'answers.get(0)' may produce 'java.lang.NullPointerException'
```

1.  如果你双击这条消息，你会发现问题所在：

```kt
setPossibleAnswer(findViewById(R.id.button_1), answers.get(0));
```

1.  这一行可能会产生`Null Pointer Exception`。为什么？如果你通过选择并按下*Cmd* + *B*（对于 Windows：*Ctrl* + *B*）来查看`getPossibleAnswers`方法的声明，你就会找到原因：

```kt
@Nullable
public ArrayList<Answer> getPossibleAnswers(){return mPossibleAnswers;}
```

啊对了！我们在第一个示例中自己添加了这个注解，以提醒我们以后（或其他开发人员）返回的答案可能为空。有几种方法可以解决这个问题。

1.  我们可以在这里删除`@Nullable`注解，但那样做是不好的，因为答案实际上可能是空的。我们也可以选择忽略这个警告。

1.  最好的解决方案是在执行任何操作之前实际测试`getAnswers`方法的结果。就像这样：

```kt
ArrayList<Answer> answers = question.getPossibleAnswers();
if (answers == null){
    return;
}
```

1.  展开**声明冗余** | **方法可以是 void** | **问题**。它说：

```kt
Return value of the method is never used 

```

1.  双击问题跳转到代码。嗯，那个警告是正确的，但假设我确实想要返回答案，因为我相当确定（你能有多确定？）我以后会使用它。在这种情况下，你可以右键单击问题，选择**对成员进行抑制**选项。你将不会再被这个问题打扰，因为它会在你的代码中添加`SuppressWarnings`注释：

```kt
@SuppressWarnings("UnusedReturnValue")public Answer addAnswer(String id, String text){
```

1.  最后，看看**拼写警告**。展开**拼写**和底层的**拼写错误**和**应用**项目。就在那里。一个`拼写错误`！

```kt
Typo: In word 'getitright' 

```

我们现在没有**getitright**，是吗？由于这是我们应用程序的名称，也是包名称的一部分，我相当确定我们可以安全地忽略这个警告。这一次，我们右键单击类型，选择**保存到字典**选项：

1.  警告列表似乎是无穷无尽的，但所有这些项目有多严重呢？在 Android Studio 的左侧，你会找到一个带有**按严重性分组**工具提示的按钮。点击它。

1.  现在树视图包含一个错误节点（如果有的话），一个警告节点和一个拼写错误节点。如果你只专注于错误和警告，并了解每个项目是关于什么，那么你将改进你的代码，并且实际上会学到很多，因为每个问题都附带了问题的描述和如何修复的建议。

很好，你今天学到了一些很酷的东西！并且通过应用模式、运行单元测试以及修复`Android Lint`报告的问题来编写更好的代码。

我们现在知道我们的应用程序做了它应该做的事情，并且在一些重构之后它结构良好。

接下来要想的是，如果我们从互联网加载的图像是现在的 10 倍大小会发生什么？如果我们有 1000 个问题呢？不真实？也许。

我们的测验应用在低端设备上的表现如何？在下一章中，我们将寻找这些和其他问题的答案。

## 另请参阅

+   参考第九章, *性能*

+   参考第十章, *测试您的应用程序的 Beta 版*
