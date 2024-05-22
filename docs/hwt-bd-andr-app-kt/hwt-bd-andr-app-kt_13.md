# 第十三章：13. RxJava 和协程

概述

本章将介绍如何使用 RxJava 和协程进行后台操作和数据操作。它涵盖了如何使用 RxJava 从外部 API 检索数据以及如何使用协程进行操作。您还将学习如何使用 RxJava 操作符和 LiveData 转换来操作和显示数据。

在本章结束时，您将能够使用 RxJava 在后台管理网络调用，并使用 RxJava 操作符转换数据。您还将能够使用 Kotlin 协程在后台执行网络任务，并使用 LiveData 转换操作来操作数据。

# 介绍

您现在已经学会了 Android 应用程序开发的基础知识，并实现了诸如 RecyclerView、通知、从网络服务获取数据和服务等功能。您还掌握了最佳实践的测试和持久化数据的技能。在上一章中，您学习了依赖注入。现在，您将学习后台操作和数据操作。

一些 Android 应用程序可以自行运行。但是，大多数应用程序可能需要后端服务器来检索或处理数据。这些操作可能需要一段时间，具体取决于互联网连接、设备设置和服务器规格。如果长时间运行的操作在主 UI 线程中运行，应用程序将被阻塞，直到任务完成。应用程序可能会变得无响应，并可能提示用户关闭并停止使用它。

为了避免这种情况，可以将可能需要花费不确定时间的任务异步运行。异步任务意味着它可以与另一个任务并行运行或在后台运行。例如，当异步从数据源获取数据时，您的 UI 仍然可以显示或与用户交互。

您可以使用 RxJava 和协程等库进行异步操作。我们将在本章讨论它们。让我们开始使用 RxJava。

# RxJava

RxJava 是**Reactive Extensions**（**Rx**）的 Java 实现，这是一种用于响应式编程的库。在响应式编程中，您有可以被观察的数据流。当值发生变化时，您的观察者可以收到通知并做出相应的反应。例如，假设点击按钮是您的可观察对象，并且您有观察者在监听它。如果用户点击该按钮，您的观察者可以做出反应并执行特定操作。

RxJava 使异步数据处理和处理错误变得更简单。以常规方式编写它很棘手且容易出错。如果您的任务涉及一系列异步任务，那么编写和调试将会更加复杂。使用 RxJava，可以更轻松地完成，并且代码量更少，更易读和易于维护。RxJava 还具有广泛的操作符，可用于将数据转换为所需的类型或格式。

RxJava 有三个主要组件：可观察对象、观察者和操作符。要使用 RxJava，您需要创建发出数据的可观察对象，使用 RxJava 操作符转换数据，并使用观察者订阅可观察对象。观察者可以等待可观察对象产生数据，而不会阻塞主线程。

## 可观察对象、观察者和操作符

让我们详细了解 RxJava 的三个主要组件。

**可观察对象**

可观察对象是可以被监听的数据源。它可以向其监听者发出数据。

`Observable`类表示一个可观察对象。您可以使用`Observable.just`和`Observable.from`方法从列表、数组或对象创建可观察对象。例如，您可以使用以下方式创建可观察对象：

```kt
val observable = Observable.just("This observable emits this string")
val observableFromList = Observable.fromIterable(listOf(1, 2, 3, 4))
```

还有更多函数可用于创建可观察对象，例如 `Observable.create`、`Observable.defer`、`Observable.empty`、`Observable.generate`、`Observable.never`、`Observable.range`、`Observable.interval` 和 `Observable.timer`。您还可以创建一个返回 `observable` 的函数。了解有关创建可观察对象的更多信息，请访问 [`github.com/ReactiveX/RxJava/wiki/Creating-Observables`](https://github.com/ReactiveX/RxJava/wiki/Creating-Observables)。

可观察对象可以是热的或冷的。冷可观察对象只有在有订阅者监听时才会发出数据。例如数据库查询或网络请求。另一方面，热可观察对象即使没有观察者也会发出数据。例如，Android 中的 UI 事件，如鼠标和键盘事件。

一旦创建了可观察对象，观察者就可以开始监听可观察对象将发送的数据。

**操作符**

操作符允许您在将数据传递给观察者之前修改和组合从可观察对象获取的数据。使用操作符会返回另一个可观察对象，因此您可以链接操作符调用。例如，假设您有一个可观察对象，它会发出从 1 到 10 的数字。您可以对其进行过滤，只获取偶数，并将列表转换为另一个包含每个项目平方的列表。要在 RxJava 中执行此操作，您可以使用以下代码：

```kt
Observable.range(1, 10)
.filter { it % 2 == 0 }
.map { it * it }
```

上述代码的输出将是一个数据流，其中包含值 4、16、36、64 和 100。

**观察者**

观察者订阅可观察对象，并在观察者发出数据时收到通知。它们可以监听可观察对象发出的下一个值或错误。`Observer` 类是观察者的接口。在创建观察者时，它有四种方法可以重写：

+   `onComplete`：当可观察对象完成发送数据时

+   `onNext`：当可观察对象发送新数据时

+   `onSubscribe`：当观察者订阅可观察对象时

+   `onError`：当可观察对象遇到错误时

要订阅可观察对象，可以调用 `Observable.subscribe()`，传入 `Observer` 接口的新实例。例如，如果要订阅从 `2` 到 `10` 的偶数可观察对象，可以执行以下操作：

```kt
Observable.fromIterable(listOf(2, 4, 6, 8, 10))
    .subscribe(object : Observer<Int> {
        override fun onComplete() {
            println("completed")
        }
        override fun onSubscribe(d: Disposable) {
            println("subscribed")
        }
        override fun onNext(t: Int) {
            println("next integer is $t")
        }
        override fun onError(e: Throwable) {
            println("error encountered")
        }
    })
```

使用此代码，观察者将打印下一个整数。它还会在订阅时打印文本，当可观察对象完成时，以及当遇到错误时。

`Observable.subscribe()` 具有不同的重载函数，其中您可以传递 `onNext`、`onError`、`onComplete` 和 `onSubscribe` 参数。这些函数返回一个 `disposable` 对象。在关闭活动时，可以调用其 `dispose` 函数以防止内存泄漏。例如，您可以使用一个变量来存储 `disposable` 对象：

```kt
val disposable = observable
            ...
            .subscribe(...)
```

然后，在您创建可观察对象的活动的 `onDestroy` 函数中，您可以调用 `disposable.dispose()` 来阻止观察者监听可观察对象：

```kt
override fun onDestroy() {
    super.onDestroy()
    disposable.dispose()
}
```

除了可观察对象、观察者和操作符之外，您还需要了解 RxJava 调度程序，这将在下一节中介绍。

## 调度程序

默认情况下，RxJava 是同步的。这意味着所有进程都在同一个线程中完成。有一些任务需要一段时间，例如数据库和网络操作，需要异步执行或在另一个线程中并行运行。为此，您需要使用调度程序。

调度程序允许您控制操作将在其中运行的线程。您可以使用两个函数：`observeOn` 和 `subscribeOn`。您可以使用 `subscribeOn` 函数设置可观察对象将在哪个线程上运行。`observeOn` 函数允许您设置下一个操作将在哪里执行。

例如，如果您有 `getData` 函数，该函数从网络获取数据并返回一个可观察对象，您可以订阅 `Schedulers.io` 并使用 `AndroidSchedulers.mainThread()` 观察 Android 主 UI 线程：

```kt
val observable = getData()
   .subscribeOn(Schedulers.io())
   .observeOn(AndroidSchedulers.mainThread())
   ...
```

`AndroidSchedulers`是 RxAndroid 的一部分，它是 RxJava 在 Android 上的扩展。您需要 RxAndroid 来在 Android 应用程序开发中使用 RxJava。

在下一节中，您将学习如何将 RxJava 和 RxAndroid 添加到您的项目中。

## 将 RxJava 添加到您的项目

您可以通过将以下代码添加到`app/build.gradle`文件的依赖项中，将 RxJava 添加到您的项目中：

```kt
implementation 'io.reactivex.rxjava3:rxandroid:3.0.0'
implementation 'io.reactivex.rxjava3:rxjava:3.0.7'
```

这将向您的 Android 项目中添加 RxJava 和 RxAndroid 库。RxAndroid 库已经包含了 RxJava，但最好还是添加 RxJava 依赖项，因为 RxAndroid 捆绑的版本可能不是最新版本。

## 在 Android 项目中使用 RxJava

RxJava 有几个好处，其中之一是处理长时间运行的操作，比如在非 UI 线程中进行网络请求。网络调用的结果可以转换为可观察对象。然后，您可以创建一个观察者来订阅可观察对象并呈现数据。在向用户显示数据之前，您可以使用 RxJava 操作符转换数据。

如果您使用 Retrofit，可以通过添加调用适配器工厂将响应转换为 RxJava 可观察对象。首先，您需要在`app/build.gradle`文件的依赖项中添加`adapter-rxjava3`，如下所示：

```kt
implementation 'com.squareup.retrofit2:adapter-rxjava3:2.9.0'
```

有了这个，您可以在您的`Retrofit`实例中使用`RxJava3CallAdapterFactory`作为调用适配器。您可以使用以下代码来实现：

```kt
val retrofit = Retrofit.Builder()
    ...
    .addCallAdapterFactory(RxJava3CallAdapterFactory.create())
    ...
```

现在，您的 Retrofit 方法可以返回您可以在代码中监听的`Observable`对象。例如，在调用电影端点的`getMovies` Retrofit 方法中，您可以使用以下内容：

```kt
@GET("movie")
fun getMovies() : Observable<Movie>
```

让我们尝试一下您迄今为止学到的知识，通过将 RxJava 添加到 Android 项目中。

## 练习 13.01：在 Android 项目中使用 RxJava

本章中，您将使用一个应用程序来显示使用 The Movie Database API 的热门电影。转到[`developers.themoviedb.org/`](https://developers.themoviedb.org/)并注册 API 密钥。在这个练习中，您将使用 RxJava 从电影/流行的端点获取所有热门电影的列表，而不考虑年份：

1.  在 Android Studio 中创建一个新项目。将项目命名为`Popular Movies`，并使用包名`com.example.popularmovies`。

1.  设置您想要保存项目的位置，然后单击`完成`按钮。

1.  打开`AndroidManifest.xml`并添加`INTERNET`权限：

```kt
<uses-permission android:name="android.permission.INTERNET" />
```

这将允许您使用设备的互联网连接进行网络调用。

1.  打开`app/build.gradle`文件，并在插件块的末尾添加 kotlin-parcelize 插件：

```kt
plugins {
    ...
    id 'kotlin-parcelize'
}
```

这将允许您为模型类使用 Parcelable。

1.  在`android`块中添加以下内容：

```kt
compileOptions {
    sourceCompatibility JavaVersion.VERSION_1_8
    targetCompatibility JavaVersion.VERSION_1_8
}
kotlinOptions {
    jvmTarget = '1.8'
}
```

这将允许您在项目中使用 Java 8。

1.  在`app/build.gradle`文件中添加以下依赖项：

```kt
implementation 'androidx.recyclerview:recyclerview:1.1.0'
implementation 'com.squareup.retrofit2:retrofit:2.9.0'
implementation 'com.squareup.retrofit2:adapter-rxjava3:2.9.0'
implementation 'io.reactivex.rxjava3:rxandroid:3.0.0'
implementation 'io.reactivex.rxjava3:rxjava:3.0.7'
implementation 'com.squareup.retrofit2:converter-moshi:2.9.0'
implementation 'com.github.bumptech.glide:glide:4.11.0'
```

这些行将向您的项目中添加 RecyclerView、Glide、Retrofit、RxJava、RxAndroid 和 Moshi 库。

1.  在`res/values`目录中创建一个`dimens.xml`文件，并添加一个`layout_margin`维度值：

```kt
<resources>
    <dimen name="layout_margin">16dp</dimen>
</resources>
```

这将用于视图的垂直和水平边距。

1.  创建一个名为`view_movie_item.xml`的新布局文件，并添加以下内容：

```kt
view_movie_item.xml
9    <ImageView
10        android:id="@+id/movie_poster"
11        android:layout_width="match_parent"
12        android:layout_height="240dp"
13        android:contentDescription="Movie Poster"
14        app:layout_constraintBottom_toBottomOf="parent"
15        app:layout_constraintEnd_toEndOf="parent"
16        app:layout_constraintStart_toStartOf="parent"
17        app:layout_constraintTop_toTopOf="parent"
18        tools:src="img/scenic" />
19
20    <TextView
21        android:id="@+id/movie_title"
22        android:layout_width="match_parent"
23        android:layout_height="wrap_content"
24        android:layout_marginStart="@dimen/layout_margin"
25        android:layout_marginEnd="@dimen/layout_margin"
26        android:ellipsize="end"
27        android:gravity="center"
28        android:lines="1"
29        android:textSize="20sp"
30        app:layout_constraintEnd_toEndOf="@id/movie_poster"
31        app:layout_constraintStart_toStartOf="@id/movie_poster"
32        app:layout_constraintTop_toBottomOf="@id/movie_poster"
33        tools:text="Movie" />
The complete code for this step can be found at http://packt.live/3sD8zmN.
```

这个布局文件包含电影海报和标题文本，将用于列表中的每部电影。

1.  打开`activity_main.xml`。用 RecyclerView 替换 Hello World TextView：

```kt
<androidx.recyclerview.widget.RecyclerView
    android:id="@+id/movie_list"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
app:layoutManager=  "androidx.recyclerview.widget.GridLayoutManager"
    app:layout_constraintBottom_toBottomOf="parent"
    app:layout_constraintTop_toTopOf="parent"
    app:spanCount="2"
    tools:listitem="@layout/view_movie_item" />
```

这个 RecyclerView 将显示电影列表。它将使用`GridLayoutManager`，有两列。

1.  为您的模型类创建一个新包`com.example.popularmovies.model`。创建一个名为`Movie`的新模型类，如下所示：

```kt
@Parcelize
data class Movie(
    val adult: Boolean = false,
    val backdrop_path: String = "",
    val id: Int = 0,
    val original_language: String = "",
    val original_title: String = "",
    val overview: String = "",
    val popularity: Float = 0f,
    val poster_path: String = "",
    val release_date: String = "",
    val title: String = "",
    val video: Boolean = false,
    val vote_average: Float = 0f,
    val vote_count: Int = 0
) : Parcelable
```

这将是代表 API 中的`Movie`对象的模型类。

1.  创建一个名为`DetailsActivity`的新活动，使用`activity_details.xml`作为布局文件。

1.  打开`AndroidManifest.xml`文件，并将`MainActivity`作为`DetailsActivity`的`parentActivityName`属性的值添加进去：

```kt
<activity android:name=".DetailsActivity"
            android:parentActivityName=".MainActivity" />
```

这将在详细信息活动中添加一个向上图标，以返回到主屏幕。

1.  打开`activity_details.xml`。添加所需的视图。（以下代码由于空间限制而被截断。请参考下面链接的文件以获取您需要添加的完整代码。）

```kt
activity_details.xml
9    <ImageView
10        android:id="@+id/movie_poster"
11        android:layout_width="160dp"
12        android:layout_height="160dp"
13        android:layout_margin="@dimen/layout_margin"
14        android:contentDescription="Poster"
15        app:layout_constraintStart_toStartOf="parent"
16        app:layout_constraintTop_toTopOf="parent"
17        tools:src="img/avatars" />
18
19    <TextView
20        android:id="@+id/title_text"
21        style="@style/TextAppearance.AppCompat.Title"
22        android:layout_width="0dp"
23        android:layout_height="wrap_content"
24        android:layout_margin="@dimen/layout_margin"
25        android:ellipsize="end"
26        android:maxLines="4"
27        app:layout_constraintEnd_toEndOf="parent"
28        app:layout_constraintStart_toEndOf="@+id/movie_poster"
29        app:layout_constraintTop_toTopOf="parent"
30        tools:text="Title" />
The complete code for this step can be found at http://packt.live/38WyRbQ.
```

这将在详情屏幕上添加海报、标题、发布日期和概述。

1.  打开`DetailsActivity`并添加以下内容：

```kt
class DetailsActivity : AppCompatActivity() {
    companion object {
        const val EXTRA_MOVIE = "movie"
        const val IMAGE_URL = "https://image.tmdb.org/t/p/w185/"
    }
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_details)
        val titleText: TextView = findViewById(R.id.title_text)
        val releaseText: TextView = findViewById(R.id.release_text)
        val overviewText: TextView = findViewById(R.id.overview_text)
        val poster: ImageView = findViewById(R.id.movie_poster)
        val movie = intent.getParcelableExtra<Movie>(EXTRA_MOVIE)
        movie?.run {
            titleText.text = title
            releaseText.text = release_date.take(4)
            overviewText.text = "Overview: $overview"
            Glide.with(this@DetailsActivity)
                .load("$IMAGE_URL$poster_path")
                .placeholder(R.mipmap.ic_launcher)
                .fitCenter()
                .into(poster)
        }
    }
}
```

这将显示所选电影的海报、标题、发布日期和概述。

1.  为电影列表创建一个适配器类。将类命名为`MovieAdapter`。添加以下内容：

```kt
class MovieAdapter(private val clickListener: MovieClickListener) :   RecyclerView.Adapter<MovieAdapter.MovieViewHolder>() {
    private val movies = mutableListOf<Movie>()
override fun onCreateViewHolder(parent: ViewGroup,   viewType: Int): MovieViewHolder {
        val view = LayoutInflater.from(parent.context)          .inflate(R.layout.view_movie_item, parent, false)
        return MovieViewHolder(view)
    }
    override fun getItemCount() = movies.size
    override fun onBindViewHolder(holder: MovieViewHolder,       position: Int) {
        val movie = movies[position]
        holder.bind(movie)
        holder.itemView.setOnClickListener {           clickListener.onMovieClick(movie) }
    }
    fun addMovies(movieList: List<Movie>) {
        movies.addAll(movieList)
        notifyItemRangeInserted(0, movieList.size)
    }
}
```

这个类将是您的 RecyclerView 的适配器。

1.  在`onBindViewHolder`函数之后为您的类添加`ViewHolder`：

```kt
class MovieAdapter...
    ...
    class MovieViewHolder(itemView: View) :       RecyclerView.ViewHolder(itemView) {
        private val imageUrl = "https://image.tmdb.org/t/p/w185/"
        private val titleText: TextView by lazy {
            itemView.findViewById(R.id.movie_title)
        }
        private val poster: ImageView by lazy {
            itemView.findViewById(R.id.movie_poster)
        }
        fun bind(movie: Movie) {
            titleText.text = movie.title
            Glide.with(itemView.context)
                .load("$imageUrl${movie.poster_path}")
                .placeholder(R.mipmap.ic_launcher)
                .fitCenter()
                .into(itemView.poster)
        }
    }
}
```

这将是`MovieAdapter`用于 RecyclerView 的`ViewHolder`。

1.  在`MovieViewHolder`声明之后，添加`MovieClickListener`：

```kt
class MovieAdapter...
    ...
    interface MovieClickListener {
        fun onMovieClick(movie: Movie)
    }
}
```

这个接口将在点击电影查看详情时使用。

1.  在`com.example.popularmovies.model`包中创建另一个名为`PopularMoviesResponse`的类：

```kt
data class PopularMoviesResponse (
    val page: Int,
    val results: List<Movie>
)
```

这将是您从热门电影 API 端点获取的响应的模型类。

1.  创建一个新的包`com.example.popularmovies.api`，并添加一个带有以下内容的`MovieService`接口：

```kt
interface MovieService {
@GET("movie/popular")
fun getPopularMovies(@Query("api_key") apiKey: String):   Observable<PopularMoviesResponse>
}
```

这将定义您将使用的端点来检索热门电影。

1.  创建一个`MovieRepository`类，并为`movieService`添加一个构造函数：

```kt
class MovieRepository(private val movieService: MovieService) { ... } 
```

1.  添加`apiKey`（值为 The Movie Database API 的 API 密钥）和一个`fetchMovies`函数来从端点检索列表：

```kt
private val apiKey = "your_api_key_here"
fun fetchMovies() = movieService.getPopularMovies(apiKey)
```

1.  创建一个名为`MovieApplication`的应用程序类，并为`movieRepository`添加一个属性：

```kt
class MovieApplication : Application() {
    lateinit var movieRepository: MovieRepository
}
```

这将是应用程序的应用程序类。

1.  覆盖`MovieApplication`类的`onCreate`函数并初始化`movieRepository`：

```kt
override fun onCreate() { 
  super.onCreate()
  val retrofit = Retrofit.Builder()
    .baseUrl("https://api.themoviedb.org/3/")
    .addConverterFactory(MoshiConverterFactory.create())
    .addCallAdapterFactory(RxJava3CallAdapterFactory.create())
    .build()
  val movieService = retrofit.create(MovieService::class.java)
  movieRepository = MovieRepository(movieService) 
}
```

1.  将`MovieApplication`设置为`AndroidManifest.xml`文件中应用程序的`android:name`属性的值：

```kt
<application
    ...
    android:name=".MovieApplication"
    ... />
```

1.  创建一个`MovieViewModel`类，并为`movieRepository`添加一个构造函数：

```kt
class MovieViewModel(private val movieRepository: MovieRepository) :   ViewModel() { ... }
```

1.  为`popularMovies`，`error`和`disposable`添加属性：

```kt
private val popularMoviesLiveData = MutableLiveData<List<Movie>>()
private val errorLiveData = MutableLiveData<String>()
val popularMovies: LiveData<List<Movie>>
    get() = popularMoviesLiveData
val error: LiveData<String>
    get() = errorLiveData
private var disposable = CompositeDisposable()
```

1.  定义`fetchPopularMovies`函数。在函数内部，从`movieRepository`获取热门电影：

```kt
    fun fetchPopularMovies() {
        disposable.add(movieRepository.fetchMovies()
            .subscribeOn(Schedulers.io())
            .map { it.results }
            .observeOn(AndroidSchedulers.mainThread())
            .subscribe({
                popularMoviesLiveData.postValue(it)
            }, { error ->
                errorLiveData.postValue("An error occurred:                   ${error.message}")
            })
        )
    }
```

当订阅时，这将在`Schedulers.io`线程中异步获取热门电影，并返回一个可观察对象，并在主线程上使用操作符。

1.  覆盖`MovieViewModel`的`onCleared`函数并处理`disposable`：

```kt
    override fun onCleared() {
        super.onCleared()
        disposable.dispose()
    }
```

当 ViewModel 被清除时，例如当活动被关闭时，这将处理`disposable`。

1.  打开`MainActivity`并定义一个电影适配器的字段：

```kt
private val movieAdapter by lazy {
    MovieAdapter(object : MovieAdapter.MovieClickListener {
        override fun onMovieClick(movie: Movie) {
            openMovieDetails(movie)
        }
    })
}
```

这将有一个监听器，当点击电影时将打开详情屏幕。

1.  在`onCreate`函数中，为`movie_list`的`RecyclerView`设置适配器：

```kt
val recyclerView: RecyclerView = findViewById(R.id.movie_list)
recyclerView.adapter = movieAdapter 
```

1.  在`MainActivity`上创建一个`getMovies`函数。在内部，初始化`movieRepository`和`movieViewModel`：

```kt
    private fun getMovies() {
        val movieRepository = (application as           MovieApplication).movieRepository
        val movieViewModel = ViewModelProvider(this, object :           ViewModelProvider.Factory {
            override fun <T : ViewModel?>               create(modelClass: Class<T>): T {
                return MovieViewModel(movieRepository) as T
            }
        }).get(MovieViewModel::class.java)
    }
```

1.  在`getMovies`函数的末尾，向`movieViewModel`的`popularMovies`和`error` LiveData 添加一个观察者：

```kt
private fun getMovies() {
        ...
        movieViewModel.fetchPopularMovies()
        movieViewModel.popularMovies
            .observe(this, { popularMovies ->
                movieAdapter.addMovies(popularMovies)
            })
            movieViewModel.error.observe(this, { error ->
                Toast.makeText(this, error, Toast.LENGTH_LONG).show()
            })
    }
```

1.  在`MainActivity`类的`onCreate`函数的末尾，调用`getMovies()`函数：

```kt
getMovies()
```

1.  在点击列表中的电影时添加`openMovieDetails`函数以打开详情屏幕：

```kt
private fun openMovieDetails(movie: Movie) { 
    val intent =       Intent(this, DetailsActivity::class.java).apply { 
        putExtra(DetailsActivity.EXTRA_MOVIE, movie)
    }
    startActivity(intent)
}
```

1.  运行您的应用程序。您将看到该应用程序将显示一个热门电影标题列表：![图 13.1：热门电影应用程序的外观](img/B15216_13_01.jpg)

图 13.1：热门电影应用程序的外观

1.  点击电影，您将看到它的发布日期和概述等详情：

![图 13.2：电影详情屏幕](img/B15216_13_02.jpg)

图 13.2：电影详情屏幕

您已经学会了如何使用 RxJava 从外部 API 检索响应。在下一节中，您将使用 RxJava 操作符将获取的数据转换为需要显示的数据。

## 使用 RxJava 操作符修改数据

当您有一个发出数据的 observable 时，您可以使用操作符在将其传递给观察者之前修改数据。您可以使用单个操作符或一系列操作符来获取所需的数据。您可以使用不同类型的操作符，例如转换操作符和过滤操作符。

转换操作符可以将 observable 中的项目修改为您喜欢的数据。 `flatMap（）`操作符将项目转换为 observable。在*练习 13.01* *在 Android 项目中使用 RxJava*中，您将 observable `PopularMoviesResponse`转换为 observable `Movies`如下：

```kt
.flatMap { Observable.fromIterable(it.results) }
```

另一个可以转换数据的操作符是`map`。`map（x）`操作符将函数`x`应用于每个项目，并返回具有更新值的另一个 observable。例如，如果您有一个数字列表的 observable，可以使用以下内容将其转换为另一个 observable 列表，其中每个数字都乘以 2：

```kt
.map { it * 2 }
```

过滤操作符允许您仅选择其中的一些项目。使用`filter（）`，您可以根据一组条件选择项目。例如，您可以使用以下内容过滤奇数：

```kt
.filter { it % 2 != 0 }
```

`first（）`和`last（）`操作符允许您获取第一个和最后一个项目，而使用`take（n）`或`takeLast（n）`，您可以获取*n*个第一个或最后一个项目。还有其他过滤操作符，如`debounce（）`，`distinct（）`，`elementAt（）`，`ignoreElements（）`，`sample（）`，`skip（）`和`skipLast（）`。

还有许多其他 RxJava 操作符可供您使用。让我们尝试在 Android 项目中使用 RxJava 操作符。

## 练习 13.02：使用 RxJava 操作符

在上一个练习中，您使用了 RxJava 从 The Movie Database API 获取了热门电影列表。现在，在将它们显示在 RecyclerView 之前，您将添加操作符来按标题排序电影并仅获取上个月发布的电影：

1.  打开*练习 13.01* *在 Android 项目中使用 RxJava*中的`Popular Movies`项目。

1.  打开`MovieViewModel`并导航到`fetchPopularMovies`函数。

1.  您将修改应用程序以仅显示今年的热门电影。将`.map { it.results }`替换为以下内容：

```kt
.flatMap { Observable.fromIterable(it.results) }
.toList()
```

这将把`MovieResponse`的 Observable 转换为`Movies`的 observable。

1.  在`toList（）`调用之前，添加以下内容：

```kt
.filter {
    val cal = Calendar.getInstance()
    cal.add(Calendar.MONTH, -1)
    it.release_date.startsWith(
        "${cal.get(Calendar.YEAR)}-${cal.get(Calendar.MONTH) + 1}"
    )
}
```

这将仅选择上个月发布的电影。

1.  运行应用程序。您将看到其他电影不再显示。只有今年发布的电影才会出现在列表上：![图 13.3：年度热门电影应用程序](img/B15216_13_03.jpg)

图 13.3：年度热门电影应用程序

1.  您还会注意到显示的电影没有按字母顺序排列。在`toList（）`调用之前使用`sorted`操作符对电影进行排序：

```kt
.sorted { movie, movie2 -> movie.title.compareTo(movie2.title) }
```

这将根据它们的标题对电影进行排序。

1.  运行应用程序。您将看到电影列表现在按标题按字母顺序排序：![图 13.4：按标题排序的年度热门电影应用程序](img/B15216_13_04.jpg)

图 13.4：按标题排序的年度热门电影应用程序

1.  在`toList（）`调用之前，使用`map`操作符将电影列表映射到另一个标题为大写的列表中：

```kt
.map { it.copy(title = it.title.toUpperCase(Locale.getDefault())) }
```

1.  运行应用程序。您将看到电影标题现在是大写字母：![图 13.5：电影标题为大写的应用程序](img/B15216_13_05.jpg)

图 13.5：电影标题为大写的应用程序

1.  在`toList（）`调用之前，使用`take`操作符仅从列表中获取前四部电影：

```kt
.take(4)
```

1.  运行应用程序。您将看到 RecyclerView 只会显示四部电影：![图 13.6：只有四部电影的应用程序](img/B15216_13_06.jpg)

图 13.6：只有四部电影的应用程序

1.  尝试其他 RxJava 操作符并运行应用程序查看结果。

您已经学会了如何使用 RxJava 操作符在显示它们之前操作来自外部 API 的检索响应。

在下一节中，您将学习如何使用协程而不是 RxJava 从外部 API 获取数据。

# 协程

Kotlin 1.3 中添加了协程，用于管理后台任务，例如进行网络调用和访问文件或数据库。Kotlin 协程是 Google 在 Android 上异步编程的官方推荐。他们的 Jetpack 库，如 LifeCycle、WorkManager 和 Room，现在包括对协程的支持。

使用协程，您可以以顺序方式编写代码。长时间运行的任务可以转换为挂起函数，当调用时可以暂停线程而不阻塞它。当挂起函数完成时，当前线程将恢复执行。这将使您的代码更易于阅读和调试。

将函数标记为挂起函数，可以在其中添加`suspend`关键字；例如，如果您有一个调用`getMovies`函数的函数，该函数从您的端点获取`movies`然后显示它：

```kt
val movies = getMovies()
displayMovies(movies) 
```

您可以通过添加`suspend`关键字将`getMovies()`函数设置为挂起函数：

```kt
suspend fun getMovies(): List<Movies> { ... }
```

在这里，调用函数将调用`getMovies`并暂停。在`getMovies`返回电影列表后，它将恢复其任务并显示电影。

挂起函数只能在挂起函数中调用，或者从协程中调用。协程具有上下文，其中包括协程调度程序。调度程序指定协程将使用的线程。您可以使用三个调度程序：

+   `Dispatchers.Main`：用于在 Android 的主线程上运行

+   `Dispatchers.IO`：用于网络、文件或数据库操作

+   `Dispatchers.Default`：用于 CPU 密集型工作

要更改协程的上下文，可以使用`withContext`函数，用于您希望在不同线程中使用的代码。例如，在您的挂起函数`getMovies`中，该函数从您的端点获取电影，您可以使用`Dispatchers.IO`：

```kt
suspend fun getMovies(): List<Movies>  {
    withContext(Dispatchers.IO) { ... }
}
```

在下一节中，我们将介绍如何创建协程。

## 创建协程

您可以使用`async`和`launch`关键字创建一个协程。`launch`关键字创建一个协程并不返回任何东西。另一方面，`async`关键字返回一个值，您可以稍后使用`await`函数获取。

`async`和`launch`必须从`CoroutineScope`创建，它定义了协程的生命周期。例如，主线程的协程范围是`MainScope`。然后，您可以使用以下内容创建协程：

```kt
MainScope().async { ... }
MainScope().launch { ... }
```

您还可以创建自己的`CoroutineScope`，而不是使用`MainScope`，通过使用`CoroutineScope`创建一个协程的上下文。例如，要为网络调用创建`CoroutineScope`，可以定义如下内容：

```kt
val scope = CoroutineScope(Dispatchers.IO)
```

当不再需要函数时，例如关闭活动时，可以取消协程。您可以通过从`CoroutineScope`调用`cancel`函数来实现：

```kt
scope.cancel()
```

ViewModel 还具有用于创建协程的默认`CoroutineScope`：`viewModelScope`。Jetpack 的 LifeCycle 还具有`lifecycleScope`，您可以使用它。当 ViewModel 被销毁时，`viewModelScope`被取消；当生命周期被销毁时，`lifecycleScope`也被取消。因此，您不再需要取消它们。

在下一节中，您将学习如何将协程添加到您的项目中。

## 将协程添加到您的项目中

您可以通过将以下代码添加到您的`app/build.gradle`文件的依赖项中，将协程添加到您的项目中：

```kt
implementation "org.jetbrains.kotlinx:kotlinx-coroutines-core:1.3.9"
implementation "org.jetbrains.kotlinx:kotlinx-coroutines-android:1.3.9"
```

`kotlinx-coroutines-core`是协程的主要库，而`kotlinx-coroutines-android`为主要的 Android 线程添加了支持。

在 Android 中进行网络调用或从本地数据库获取数据时，可以添加协程。

如果您使用的是 Retrofit 2.6.0 或更高版本，可以使用`suspend`将端点函数标记为挂起函数：

```kt
@GET("movie/latest")
suspend fun getMovies() : List<Movies>
```

然后，您可以创建一个协程，调用挂起函数`getMovies`并显示列表：

```kt
CoroutineScope(Dispatchers.IO).launch {
    val movies = movieService.getMovies()
    withContext(Dispatchers.Main) {
        displayMovies(movies)
    }
}
```

您还可以使用 LiveData 来响应您的协程。LiveData 是一个 Jetpack 类，可以保存可观察的数据。通过添加以下依赖项，您可以将 LiveData 添加到 Android 项目中：

```kt
implementation 'androidx.lifecycle:lifecycle-livedata-ktx:2.2.0'
```

让我们尝试在 Android 项目中使用协程。

## 练习 13.03：在 Android 应用程序中使用协程

在这个练习中，您将使用协程从 The Movie Database API 获取热门电影列表。您可以使用上一个练习中的`Popular Movies`项目，或者复制一个：

1.  在 Android Studio 中打开`Popular Movies`项目。

1.  打开`app/build.gradle`文件，并删除以下依赖项：

```kt
implementation 'com.squareup.retrofit2:adapter-rxjava3:2.9.0'
implementation 'io.reactivex.rxjava3:rxandroid:3.0.0'
implementation 'io.reactivex.rxjava3:rxjava:3.0.7'
```

由于您将使用协程而不是 RxJava，因此将不再需要这些依赖项。

1.  在`app/build.gradle`文件中，添加 Kotlin 协程的依赖项：

```kt
implementation 'org.jetbrains.kotlinx:kotlinx-coroutines-core:1.3.9'
implementation 'org.jetbrains.kotlinx:kotlinx-coroutines-android:1.3.9'
```

这将允许您在项目中使用协程。

1.  还要添加 ViewModel 和 LiveData 扩展库的依赖项：

```kt
implementation 'androidx.lifecycle:lifecycle-livedata-ktx:2.2.0'
implementation 'androidx.lifecycle:lifecycle-viewmodel-ktx:2.2.0'
```

1.  打开`MovieService`接口，并用以下代码替换它：

```kt
interface MovieService {
    @GET("movie/popular")
    suspend fun getPopularMovies(@Query("api_key") apiKey: String):       PopularMoviesResponse
}
```

这将把`getPopularMovies`标记为挂起函数。

1.  打开`MovieRepository`并为电影列表添加 movies 和 error LiveData：

```kt
    private val movieLiveData = MutableLiveData<List<Movie>>()
    private val errorLiveData = MutableLiveData<String>()
    val movies: LiveData<List<Movie>>
        get() = movieLiveData
    val error: LiveData<String>
        get() = errorLiveData
```

1.  将`fetchMovies`函数替换为一个挂起函数，以从端点检索列表：

```kt
    suspend fun fetchMovies() {
        try {
            val popularMovies = movieService.getPopularMovies(apiKey)
            movieLiveData.postValue(popularMovies.results)
        } catch (exception: Exception) {
            errorLiveData.postValue("An error occurred:               ${exception.message}")
        }
    }
```

1.  使用以下代码更新`MovieViewModel`的内容：

```kt
    init {
        fetchPopularMovies()
    }
    val popularMovies: LiveData<List<Movie>>
    get() = movieRepository.movies
    fun getError(): LiveData<String> = movieRepository.error
    private fun fetchPopularMovies() {
        viewModelScope.launch(Dispatchers.IO)  {
            movieRepository.fetchMovies()
        }
    }
```

`fetchPopularMovies`函数有一个协程，使用`viewModelScope`，它将从`movieRepository`获取电影。

1.  打开`MovieApplication`文件。在`onCreate`函数中，删除包含`addCallAdapterFactory`的行。它应该是这样的：

```kt
    override fun onCreate() {
        super.onCreate()
        val retrofit = Retrofit.Builder()
            .baseUrl("https://api.themoviedb.org/3/")
            .addConverterFactory(MoshiConverterFactory.create())
            .build()
        ...
    }
```

1.  打开`MainActivity`类。删除`getMovies`函数。

1.  在`onCreate`函数中，删除对`getMovies`的调用。然后，在`onCreate`函数的末尾，创建`movieViewModel`：

```kt
val movieRepository =   (application as MovieApplication).movieRepository
val movieViewModel =   ViewModelProvider(this, object: ViewModelProvider.Factory {
    override fun <T : ViewModel?>       create(modelClass: Class<T>): T {
        return MovieViewModel(movieRepository) as T
    }
}).get(MovieViewModel::class.java)
```

1.  之后，向`movieViewModel`的`getPopularMovies`和`error` LiveData 添加观察者：

```kt
        movieViewModel.popularMovies.observe(this, { popularMovies ->
            movieAdapter.addMovies(popularMovies
                .filter {
                    it.release_date.startsWith(
                        Calendar.getInstance().get(Calendar.YEAR)                          .toString()
                    )
                }
                .sortedBy { it.title }
            )
        })
        movieViewModel.getError().observe(this, { error ->
            Toast.makeText(this, error, Toast.LENGTH_LONG).show()
})
```

这将使用 Kotlin 的`filter`函数对电影列表进行过滤，只包括今年发布的电影。然后使用 Kotlin 的`sortedBy`函数按标题排序。

1.  运行应用程序。您将看到应用程序将显示今年发布的热门电影标题列表，按标题排序：

![图 13.7：应用程序显示今年发布的热门电影，按标题排序](img/B15216_13_07.jpg)

图 13.7：应用程序显示今年发布的热门电影，按标题排序

您已经使用协程和 LiveData 从远程数据源检索和显示了一组热门电影列表，而不会阻塞主线程。

在将 LiveData 传递到 UI 进行显示之前，您还可以首先转换数据。您将在下一节中了解到这一点。

# 转换 LiveData

有时，您从 ViewModel 传递到 UI 层的 LiveData 在显示之前需要进行处理。例如，您只能选择部分数据或者首先对其进行一些处理。在上一个练习中，您对数据进行了过滤，只选择了当前年份的热门电影。

要修改 LiveData，您可以使用`Transformations`类。它有两个函数，`Transformations.map`和`Transformations.switchMap`，您可以使用。

`Transformations.map`将 LiveData 的值修改为另一个值。这可用于过滤、排序或格式化数据等任务。例如，您可以将`movieLiveData`从电影标题转换为字符串 LiveData：

```kt
private val movieLiveData: LiveData<Movie>
val movieTitleLiveData : LiveData<String> = 
   Transformations.map(movieLiveData) { it.title }
```

当`movieLiveData`的值发生变化时，`movieTitleLiveData`也会根据电影的标题发生变化。

使用`Transformations.switchMap`，您可以将 LiveData 的值转换为另一个 LiveData。当您想要使用原始 LiveData 进行涉及数据库或网络操作的特定任务时使用。例如，如果您有一个表示电影`id`对象的 LiveData，您可以通过应用函数`getMovieDetails`将其转换为电影 LiveData，该函数从`id`对象（例如从另一个网络或数据库调用）返回电影详细信息的 LiveData：

```kt
private val idLiveData: LiveData<Int> = MutableLiveData()
val movieLiveData : LiveData<Movie> = 
    Transformations.switchMap(idLiveData) { getMovieDetails(it) }
fun getMovieDetails(id: Int) : LiveData<Movie> = { ... }
```

让我们在使用协程获取的电影列表上使用 LiveData 转换。

## 练习 13.04：LiveData 转换

在这个练习中，您将在传递给`MainActivity`文件中的观察者之前转换电影的 LiveData 列表：

1.  在 Android Studio 中，打开您在上一个练习中使用的“热门电影”项目。

1.  打开`MainActivity`文件。在`onCreate`函数中的`movieViewModel.popularMovies`观察者中，删除过滤器和`sortedBy`函数调用。代码应如下所示：

```kt
movieViewModel.getPopularMovies().observe(this,   Observer { popularMovies ->
    movieAdapter.addMovies(popularMovies)
})
```

现在将显示列表中的所有电影，而不按标题排序。

1.  运行应用程序。您应该看到所有电影（甚至是去年的电影），而不是按标题排序：![图 13.8：未排序的热门电影应用程序](img/B15216_13_08.jpg)

图 13.8：未排序的热门电影应用程序

1.  打开`MovieViewModel`类，并使用 LiveData 转换来更新`popularMovies`以过滤和排序电影：

```kt
        val popularMovies: LiveData<List<Movie>>
        get() = movieRepository.movies.map { list ->
        list.filter {
            val cal = Calendar.getInstance()
            cal.add(Calendar.MONTH, -1)
            it.release_date.startsWith(
                "${cal.get(Calendar.YEAR)}-${cal.get(Calendar.MONTH)                   + 1}"
            )
        }.sortedBy { it.title }
    }
```

这将选择上个月发布的电影，并在传递给`MainActivity`中的 UI 观察者之前按标题排序。

1.  运行应用程序。您会看到应用程序显示了按标题排序的今年热门电影列表：

![图 13.9：按标题排序的今年发布的电影应用程序](img/B15216_13_05.jpg)

图 13.9：按标题排序的今年发布的电影应用程序

您已经使用了 LiveData 转换来修改电影列表，只选择今年发布的电影。它们在传递给 UI 层的观察者之前也按标题排序。

# 协程通道和流

如果您的协程正在获取数据流或您有多个数据源并且逐个处理数据，您可以使用通道或流。

通道允许在不同的协程之间传递数据。它们是热数据流。它将在被调用时运行并发出值，即使没有监听器。而流是冷异步流。只有在收集值时才会发出值。

要了解有关通道和流的更多信息，您可以访问[`kotlinlang.org`](https://kotlinlang.org)。

# RxJava 与协程

RxJava 和协程都可以用于在 Android 中执行后台任务，例如网络调用或数据库操作。

那么应该使用哪一个？虽然您可以在应用程序中同时使用两者，例如，对于一个任务使用 RxJava，对于另一个任务使用协程，您还可以与`LiveDataReactiveStreams`或`kotlinx-coroutines-rx3`一起使用它们。然而，这将增加您使用的依赖项数量和您的应用程序的大小。

那么，RxJava 还是协程？以下表格显示了两者之间的区别：

![图 13.10：协程和 RxJava 之间的区别](img/B15216_13_10.jpg)

图 13.10：协程和 RxJava 之间的区别

让我们继续下一个活动。

## 活动 13.01：创建电视指南应用程序

很多人看电视。然而，大多数时候，他们不确定当前有哪些电视节目正在播放。假设你想开发一个应用程序，可以使用 Kotlin 协程和 LiveData 从 The Movie Database API 的`tv/on_the_air`端点显示这些节目的列表。

该应用程序将有两个屏幕：主屏幕和详情屏幕。在主屏幕上，您将显示正在播出的电视节目列表。电视节目将按名称排序。点击一个电视节目将打开详情屏幕，显示有关所选电视节目的更多信息。

完成步骤：

1.  在 Android Studio 中创建一个名为`TV Guide`的新项目，并设置其包名称。

1.  在`AndroidManifest.xml`文件中添加`INTERNET`权限。

1.  在`app/build.gradle`文件中，添加 Java 8 兼容性和 RecyclerView、Glide、Retrofit、RxJava、RxAndroid、Moshi、ViewModel 和 LiveData 库的依赖项。

1.  添加`layout_margin`维度值。

1.  创建一个`view_tv_show_item.xml`布局文件，其中包含用于海报的`ImageView`和用于电视节目名称的`TextView`。

1.  在`activity_main.xml`文件中，删除 Hello World TextView，并添加一个用于电视节目列表的 RecyclerView。

1.  创建一个名为`TVShow`的模型类。

1.  创建一个名为`DetailsActivity`的新活动，使用`activity_details.xml`作为布局文件。

1.  打开`AndroidManifest.xml`文件，在`DetailsActivity`声明中添加`parentActivityName`属性。

1.  在`activity_details.xml`中，添加用于电视节目详情的视图。

1.  在`DetailsActivity`中，添加用于显示所选电视节目详情的代码。

1.  为电视节目列表创建一个`TVShowAdapter`适配器类。

1.  创建另一个名为`TVResponse`的类，用于从 API 端点获取正在播出的电视节目的响应。

1.  创建一个`TelevisionService`类，用于添加 Retrofit 方法。

1.  创建一个名为`TVShowRepository`的类，其中包含`tvService`的构造函数，以及`apiKey`和`tvShows`的属性。

1.  创建一个挂起函数，从端点检索电视节目列表。

1.  创建一个`TVShowViewModel`类，其中包含`TVShowRepository`的构造函数。添加一个`getTVShows`函数，返回电视节目列表的 LiveData，以及`fetchTVShows`函数，从存储库中获取列表。

1.  创建一个名为`TVApplication`的应用程序类，其中包含`TVShowRepository`的属性。

1.  将`TVApplication`设置为`AndroidManifest.xml`文件中应用程序的值。

1.  打开`MainActivity`并添加代码，以在`ViewModel`更新其值时更新 RecyclerView。添加一个函数，点击列表中的电视节目将打开详情屏幕。

1.  运行你的应用程序。该应用程序将显示一个电视节目列表。点击一个电视节目将打开详情活动，显示节目详情。主屏幕和详情屏幕将类似于以下图示：

![图 13.11：电视指南应用的主屏幕和详情屏幕](img/B15216_13_11.jpg)

图 13.11：电视指南应用的主屏幕和详情屏幕

注意

此活动的解决方案可在以下网址找到：http://packt.live/3sKj1cp

# 总结

本章重点介绍了使用 RxJava 和协程进行后台操作。后台操作用于长时间运行的任务，例如从本地数据库或远程服务器访问数据。

您从 RxJava 的基础知识开始：可观察对象、观察者和操作符。可观察对象是提供数据的数据源。观察者监听可观察对象；当可观察对象发出数据时，观察者可以做出相应反应。操作符允许您修改可观察对象的数据，使其能够传递给观察者所需的数据。

接下来，您学习了如何使用调度程序使 RxJava 调用异步。调度程序允许您设置执行所需操作的线程。`subscribeOn`函数用于设置可观察对象将在哪个线程上运行，`observeOn`函数允许您设置下一个操作将在哪里执行。然后，您使用 RxJava 从外部 API 获取数据，并使用 RxJava 操作符对数据进行过滤、排序和修改。

接下来，你将学习使用 Kotlin 协程，这是 Google 推荐的异步编程解决方案。你可以使用`suspend`关键字将后台任务转换为挂起函数。协程可以使用`async`或`launch`关键字启动。

你已经学会了如何创建挂起函数以及如何启动协程。你还使用调度程序来改变协程运行的线程。最后，你使用协程来进行网络调用，并使用 LiveData 转换函数`map`和`switchMap`修改检索到的数据。

在下一章中，你将学习关于架构模式。你将学习诸如**MVVM**（**Model-View-ViewModel**）之类的模式，以及如何改进应用程序的架构。
