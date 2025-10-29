# 构建信使 Android 应用程序 – 第 I 部分

在上一章中，我们开始构建信使应用程序，通过设计和实现客户端信使应用程序将与之通信的 REST 应用程序编程接口。在实现后端 API 的过程中，我们涵盖了众多内容，例如使用 Spring Boot、RESTful 应用程序编程接口及其工作原理、使用 PostgreSQL 创建数据库，以及将 Spring Boot Web 应用程序部署到 AWS 等，仅举几例。

在本章中，我们将通过实现 Android Messenger 应用程序并将其与我们创建的 RESTful API（第四章，*使用 Spring Boot 2.0 设计和实现 Messenger 后端*）集成，进一步深入我们的应用程序开发之旅。在开发 Messenger Android 应用程序的过程中，我们将学习大量新主题，例如：

+   构建 MVP Android 应用程序

+   通过 HTTP 进行服务器通信

+   使用 Retrofit 进行操作

+   响应式编程

+   在 Android 应用中使用基于令牌的认证

在本章中，您将亲身体验到 Kotlin 在 Android 应用程序开发领域的强大功能。让我们深入到信使应用程序的开发中。

# 开发信使应用程序

首先，我们需要为应用程序创建一个新的 Android Studio 项目。创建一个名为`Messenger`、包名为`com.example.messenger`的新 Android Studio 项目。您可以随意查看第一章，*基础*，以刷新您对 Android 项目创建的记忆。在项目设置过程中，当被要求创建一个新的启动器活动时，将活动命名为`LoginActivity`并使其成为一个空活动。

# 包含项目依赖项

在本章中，我们将使用多个外部应用程序依赖项。因此，现在将它们包含在项目中非常重要。打开您的模块级`build.gradle`文件，并添加以下依赖项到其中：

```
dependencies {
  implementation fileTree(dir: 'libs', include: ['*.jar'])
  implementation "org.jetbrains.kotlin:kotlin-stdlib-jre7
                  :$kotlin_version"
  implementation 'com.android.support:appcompat-v7:26.1.0'
  implementation 'com.android.support.constraint:constraint-layout:1.0.2'
  implementation 'com.android.support:recyclerview-v7:26.1.0'
  implementation 'com.android.support:design:26.1.0'

  implementation "android.arch.persistence.room:runtime:1.0.0-alpha9-1"
  implementation "android.arch.persistence.room:rxjava2:1.0.0-alpha9-1"
  implementation 'com.android.support:support-v4:26.1.0'
  implementation 'com.android.support:support-vector-drawable:26.1.0'
  annotationProcessor "android.arch.persistence.room:compiler
                       :1.0.0-alpha9-1"

  implementation "com.squareup.retrofit2:retrofit:2.3.0"
  implementation "com.squareup.retrofit2:adapter-rxjava2:2.3.0"
  implementation "com.squareup.retrofit2:converter-gson:2.3.0"
  implementation "io.reactivex.rxjava2:rxandroid:2.0.1"

  implementation 'com.github.stfalcon:chatkit:0.2.2'

  testImplementation 'junit:junit:4.12'
  androidTestImplementation 'com.android.support.test:runner:1.0.1'
  androidTestImplementation 'com.android.support.test.espresso
                             :espresso-core:3.0.1'
}
```

确保在`build.gradle`文件中不存在冲突的 Android 支持库版本。现在修改`build.gradle`项目文件，以包含`jcenter`和 Google 仓库以及 Android 构建工具依赖项：

顶级构建文件，其中您可以添加适用于所有子项目/模块的配置选项：

```
buildscript {
  ext.kotlin_version = '1.1.4-3'
  repositories {
    google()
    jcenter()
  }
  dependencies {
    classpath 'com.android.tools.build:gradle:3.0.0-alpha9'
    classpath "org.jetbrains.kotlin:kotlin-gradle-plugin:$kotlin_version"
  }
}

allprojects {
  repositories {
    google()
    jcenter()
  }
}

task clean(type: Delete) {
  delete rootProject.buildDir
}
```

不要担心现在添加的依赖项，所有内容将在本章中一一揭晓。

# 开发登录用户界面

一旦项目创建完成，在`com.example.messenger`应用程序源包中创建一个名为`ui`的新包。这个包将包含 Android 应用程序中所有与用户界面相关的类和逻辑。在`ui`包内创建一个`login`包。正如你可能已经猜到的，这个包将包含与用户登录过程相关的类和逻辑。将`LoginActivity`移动到`login`包中。移动了`LoginActivity`之后，我们的首要任务是创建一个适合登录活动的布局。

定位到`activity_login.xml`布局资源文件并更改以下内容：

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".ui.login.LoginActivity"
    android:orientation="vertical"
    android:paddingTop="32dp"
    android:paddingBottom="@dimen/default_margin"
    android:paddingStart="@dimen/default_padding"
    android:paddingEnd="@dimen/default_padding"
    android:gravity="center_horizontal">
    <EditText
        android:id="@+id/et_username"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:inputType="text"
        android:hint="@string/username"/>
    <EditText
        android:id="@+id/et_password"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginTop="@dimen/default_margin"
        android:inputType="textPassword"
        android:hint="@string/password"/>
    <Button
        android:id="@+id/btn_login"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginTop="@dimen/default_margin"
        android:text="@string/login"/>
    <Button
        android:id="@+id/btn_sign_up"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginTop="@dimen/default_margin"
        android:background="@android:color/transparent"
        android:text="@string/sign_up_solicitation"/>
    <ProgressBar
        android:id="@+id/progress_bar"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:visibility="gone"/>
</LinearLayout>
```

我们在这里使用了一些字符串和尺寸资源，但我们还没有在适当的`.xml`资源文件中创建它们。我们必须在此处添加这些资源。同时，我们还将包括在应用程序开发阶段后期可能需要的资源，以消除在程序文件和资源文件之间跳转的需要。打开项目的字符串资源文件（`strings.xml`）并确保以下资源被添加到其中：

```
<resources>
  <string name="app_name">Messenger</string>
  <string name="username">Username</string>
  <string name="password">Password</string>
  <string name="login">Login</string>
  <string name="sign_up_solicitation">
    Don\'t have an account? Sign up!
  </string>
  <string name="sign_up">Sign up</string>
  <string name="phone_number">Phone number</string>
  <string name="action_settings">settings</string>
  <string name="hint_enter_a_message">Type a message…</string>

  <!--  Account settings -->
  <string name="title_activity_settings">Settings</string>
  <string name="pref_header_account">Account</string>
  <string name="action_logout">logout</string>
</resources>

```

现在创建一个尺寸资源文件（`dimens.xml`）并将以下尺寸资源添加到其中：

```
<?xml version="1.0" encoding="utf-8"?>
<resources>
  <dimen name="default_margin">16dp</dimen>
  <dimen name="default_padding">16dp</dimen>
</resources>
```

现在已经添加了必要的项目资源，导航回`activity_login.xml`并切换设计预览屏幕以查看创建的布局：

![图片](img/36fd17e0-95ab-4357-933c-b2118da19fc8.jpg)

布局简单但实用，这对于我们正在构建的简单消息应用来说非常完美。

# 创建登录视图

现在我们必须处理`LoginActivity`。由于我们正在使用 MVP 模式构建此应用程序，`LoginActivity`实际上是一个视图。显然，`LoginActivity`与任何通用视图都大不相同。它是一个关注登录过程的视图。我们可以确定一组必要的操作，一个向用户展示登录界面的视图必须具备这些操作。这些行为包括：

+   当登录正在进行时，它必须向用户显示进度条

+   当需要时，它必须能够隐藏进度条

+   它必须能够在遇到时向用户显示适当的字段错误

+   它必须能够将用户导航到其主页

+   它必须能够将未注册用户导航到注册屏幕

确定了前面的行为后，我们必须确保`LoginActivity`——作为一个登录视图——展现出这些行为。一个完美的方法是利用接口。在`login`包中创建一个名为`LoginView`的接口，包含以下内容：

```
package com.example.messenger.ui.login

interface LoginView {
  fun showProgress()
  fun hideProgress()
  fun setUsernameError()
  fun setPasswordError()
  fun navigateToSignUp()
  fun navigateToHome()
}
```

到目前为止，`LoginView`的情况还不错。不过，我们的接口存在一些问题。一个`LoginView`必须能够将其布局视图绑定到适当的对象表示。此外，如果发生身份验证错误，`LoginView`必须能够向用户提供反馈。你可能认为这两个行为都不应该与`LoginView`区分开来。你说得对。所有视图都应该能够将布局元素绑定到程序性对象。此外，注册视图也应该能够在身份验证过程中出现问题时向用户提供某种形式的反馈。

我们将创建两个不同的接口来强制执行这些行为。我们将第一个接口命名为`BaseView`。在`com.example.messenger.ui`中创建一个`base`包，并向该包中添加一个名为`BaseView`的接口，内容如下：

```
package com.example.messenger.ui.base

import android.content.Context

interface BaseView {
  fun bindViews()
  fun getContext(): Context
}
```

`BaseView`接口强制实现类声明`bindViews()`和`getContext()`函数，分别用于视图绑定和上下文检索。

现在在`com.example.messenger.ui`中创建一个`auth`包，并向该包中添加一个名为`AuthView`的接口，内容如下：

```
package com.example.messenger.ui.auth

interface AuthView {
  fun showAuthError()
}
```

干得好！现在回到`LoginView`接口，确保它扩展`BaseView`和`AuthView`，如下所示：

```
package com.example.messenger.ui.login

import com.example.messenger.ui.auth.AuthView
import com.example.messenger.ui.base.BaseView

interface LoginView : BaseView, AuthView {
  fun showProgress()
  fun hideProgress()
  fun setUsernameError()
  fun setPasswordError()
  fun navigateToSignUp()
  fun navigateToHome()
}
```

通过将`LoginView`接口声明为`BaseView`和`AuthView`的扩展，我们确保每个实现`LoginView`的类都必须声明`bindViews()`、`getContext()`和`showAuthError()`函数，以及`LoginView`中声明的那些函数。重要的是要注意，任何实现`LoginView`的类实际上都是`LoginView`、`BaseView`和`AuthView`类型的。一个类具有许多类型的特征被称为多态。

在设置了`LoginView`之后，我们可以继续工作在`LoginActivity`上。首先，我们将创建`LoginActivity`以实现`BaseView`和`AuthView`中声明的方 法，然后我们将添加特定于`LoginView`的方法。`LoginActivity`的代码如下：

```
package com.example.messenger.ui.login

import android.content.Context
import android.content.Intent
import android.support.v7.app.AppCompatActivity
import android.os.Bundle
import android.view.View
import android.widget.Button
import android.widget.EditText
import android.widget.ProgressBar
import android.widget.Toast
import com.example.messenger.R

class LoginActivity : AppCompatActivity(), LoginView, View.OnClickListener {

  private lateinit var etUsername: EditText
  private lateinit var etPassword: EditText
  private lateinit var btnLogin: Button
  private lateinit var btnSignUp: Button
  private lateinit var progressBar: ProgressBar

  override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    setContentView(R.layout.activity_login)

    bindViews()
  }
```

在调用时将布局视图对象引用绑定到视图元素：

```

  override fun bindViews() {
    etUsername = findViewById(R.id.et_username)
    etPassword = findViewById(R.id.et_password)
    btnLogin = findViewById(R.id.btn_login)
    btnSignUp = findViewById(R.id.btn_sign_up)
    progressBar = findViewById(R.id.progress_bar)
    btnLogin.setOnClickListener(this)
    btnSignUp.setOnClickListener(this)
  }

  /**
    * Shows an appropriate Authentication error message when invoked.
  */
  override fun showAuthError() {
    Toast.makeText(this, "Invalid username and password combination.", Toast.LENGTH_LONG).show()
  }

  override fun onClick(view: View) {

  }

  override fun getContext(): Context {
    return this
  }
}
```

到目前为止一切顺利。我们已经成功地在`LoginActivity`中实现了`BaseView`和`AuthView`方法。我们仍然必须处理`LoginView`特定的方法`showProgress()`、`hideProgress()`、`setUsernameError()`、`setPasswordError()`、`navigateToSignUp()`和`navigateToHome()`。这些方法的必要实现如下。继续并将它们添加到`LoginActivity`中。

```
override fun hideProgress() {
  progressBar.visibility = View.GONE
}

override fun showProgress() {
  progressBar.visibility = View.VISIBLE
}

override fun setUsernameError() {
  etUsername.error = "Username field cannot be empty"
}

override fun setPasswordError() {
  etPassword.error = "Password field cannot be empty"
}

override fun navigateToSignUp() {

}

override fun navigateToHome() {

}
```

添加之前定义的所有方法后，你使`LoginActivity`类实现了`LoginView`以及`View.OnClickListener`接口。因此，`LoginActivity`提供了这些接口中声明的函数的实现。注意当前`LoginActivity`实例是如何通过`this`传递给`btnLogin.setOnClickListener()`的。我们可以这样做，因为我们已经将`LoginActivity`声明为实现了`View.OnClickListener`接口。因此，`LoginActivity`是一个有效的`View.OnClickListener`实例（这是一个多态工作的完美例子）。

现在我们已经对登录视图做了一些合理的工作，我们必须创建一个适当的模型来处理登录逻辑。我们还需要创建这个模型将与之通信的必要服务和数据存储库。我们首先将处理所需的服务，然后开发必要的数据存储库，最后构建交互器。

# 创建 Messenger API 服务和数据存储库

在深入我们的应用程序开发过程之前，我们必须考虑一个关键问题：数据存储。我们必须问自己两个非常重要的问题：数据将存储在哪里，以及如何访问存储的数据？

关于数据存储的位置，数据将同时本地存储（在 Android 设备上）和远程存储（在 Messenger API 上）。第二个问题的答案同样简单直接。为了访问存储的数据，我们需要创建合适的模型、服务和存储库以方便数据检索。

# 使用 SharedPreferences 本地存储数据

首先最重要的是，我们需要注意本地存储。由于这是一个简单的应用程序，我们不需要在本地存储很多数据。我们只需要在设备上存储访问令牌和用户详情。我们将使用`SharedPreferences`来完成这项工作。

首先，在应用程序的`source`包内创建一个`data`包。我们之前已经确定我们将要处理本地和远程存储的数据。因此，在`data`包内创建两个额外的包。第一个命名为`local`，第二个命名为`remote`。类似于我们在第二章中创建的*Tetris*应用程序所采用的方法，即构建 Android 应用程序 – Tetris 和实现 Tetris 逻辑和功能，我们将使用`AppPreferences`类来本地持久化数据。在`local`包内创建一个`AppPreferences`类，并填充以下内容：

```
package com.example.messenger.data.local

import android.content.Context
import android.content.SharedPreferences
import com.example.messenger.data.vo.UserVO

class AppPreferences private constructor() {

  private lateinit var preferences: SharedPreferences

  companion object {
    private val PREFERENCE_FILE_NAME = "APP_PREFERENCES"

    fun create(context: Context): AppPreferences {
      val appPreferences = AppPreferences()
      appPreferences.preferences = context
      .getSharedPreferences(PREFERENCE_FILE_NAME, 0)
        return appPreferences
      }
    }

    val accessToken: String?
      get() = preferences.getString("ACCESS_TOKEN", null)

    fun storeAccessToken(accessToken: String) {
      preferences.edit().putString("ACCESS_TOKEN", accessToken).apply()
    }

    val userDetails: UserVO
    get(): UserVO {
```

返回包含适当用户详情的`UserVO`实例：

```
  return UserVO(
    preferences.getLong("ID", 0),
    preferences.getString("USERNAME", null),
    preferences.getString("PHONE_NUMBER", null),
    preferences.getString("STATUS", null),
    preferences.getString("CREATED_AT", null)
  )
}
```

将通过`UserVO`传递给应用程序的`SharedPreferences`文件的用户详情存储起来：

```
  fun storeUserDetails(user: UserVO) {
    val editor: SharedPreferences.Editor = preferences.edit()

    editor.putLong("ID", user.id).apply()
    editor.putString("USERNAME", user.username).apply()
    editor.putString("PHONE_NUMBER", user.phoneNumber).apply()
    editor.putString("STATUS", user.status).apply()
    editor.putString("CREATED_AT", user.createdAt).apply()
  }

  fun clear() {
    val editor: SharedPreferences.Editor = preferences.edit()
    editor.clear()
    editor.apply()
  }
}
```

在`AppPreferences`中，我们定义了`storeAccessToken(String)`、`storeUserDetails(UserVO)`和`clear()`函数。`storeAccessToken(String)`将用于将远程服务器检索到的访问令牌存储到本地首选项文件中。`storeUserDetails(UserVO)`接受一个用户值对象（包含用户信息的数据对象）作为其唯一参数，并将值对象中包含的信息存储到首选项文件中。`clear()`方法，正如其名称所暗示的，清除首选项文件中存储的所有值。`AppPreferences`实例还具有`accessToken`和`userDetails`属性，每个属性都有专门的 getter 函数来检索其适当的值。除了在`AppPreferences`中定义的函数和属性之外，我们还创建了一个拥有单个`create(Context)`函数的伴生对象。`create()`方法，正如其名称所暗示的，创建并返回一个新的`AppPreferences`实例以供使用。我们将`AppPreferences`的主要构造函数设为私有，因为我们要求任何使用`AppPreferences`的类都使用`create()`来实例化`AppPreferences`。

# 创建值对象

与我们创建消息后端时所做的类似，我们需要创建值对象来模拟我们将要在整个应用程序中处理的数据的常见类型。在`data`包中创建一个`vo`包。我们正在创建的值对象对你来说已经很熟悉了。事实上，它们与我们开发 API 时创建的完全相同。我们将创建`ConversationListVO`、`ConversationVO`、`UserListVO`、`UserVO`和`MessageVO`。在`vo`包中创建 Kotlin 文件以保存这些值对象。在创建任何列表值对象数据模型之前，我们必须创建基本模型。这些模型是`UserVO`、`MessageVO`和`ConversationVO`。

创建一个`UserVO`数据类，如下所示：

```
package com.example.messenger.data.vo

data class UserVO(
  val id: Long,
  val username: String,
  val phoneNumber: String,
  val status: String,
  val createdAt: String
)
```

由于我们之前已经创建了值对象，所以之前的代码不需要太多解释。将`MessageVO`添加到你的`MessageVO.kt`文件中，如下所示：

```
package com.example.messenger.data.vo

data class MessageVO(
  val id: Long,
  val senderId: Long,
  val recipientId: Long,
  val conversationId: Long,
  val body: String,
  val createdAt: String
)
```

现在在`ConversationVO.kt`中创建一个`ConversationVo`数据类，如下所示：

```
package com.example.messenger.data.vo

data class ConversationVO(
  val conversationId: Long,
  val secondPartyUsername: String,
  val messages: ArrayList<MessageVO>
)
```

在创建了基本值对象之后，让我们创建`ConversationListVO`和`UserListVO`，好吗？`ConversationListVO`如下所示：

```
package com.example.messenger.data.vo

data class ConversationListVO(
  val conversations: List<ConversationVO>
)
```

`ConversationListVO`数据类有一个单一的`conversations`属性，其类型为`List`，只能包含`ConversationVO`类型的元素。`UserListVO`数据类与`ConversationListVO`类似，但有一个用户的属性，只能包含`UserVO`类型的元素，而不是`conversations`属性。以下是`UserListVO`数据类：

```
package com.example.messenger.data.vo

data class UserListVO(
  val users: List<UserVO>
)
```

# 获取远程数据

我们已经确定，对于 Messenger Android 应用程序的功能至关重要的重要数据将存储在远程的 Messenger 后端。我们必须有一种有效的方法，让我们的 Android 应用程序能够访问后端持有的数据。为此，Messenger 应用程序需要能够通过 HTTP 与 API 进行通信。

# 与远程服务器通信

在 Android 中，有多种方式可以实现与远程服务器的通信。Android 社区中常用的网络库有 **Retrofit**、**OkHttp** 和 **Volley**。每个库都有其优点和缺点。我们将在这个项目中使用 Retrofit，但为了知识普及，我们将看看如何使用 OkHttp 与远程服务器通信。

# 使用 OkHttp 与服务器通信

OkHttp 是一个高效且易于使用的 HTTP 客户端。它支持同步和异步网络调用。在 Android 上使用 OkHttp 很简单。只需将其依赖项添加到项目模块级别的 `build.gradle` 文件中：

```
implementation 'com.squareup.okhttp3:okhttp:3.9.0'
```

# 使用 OkHttp 向服务器发送请求

如前所述，OkHttp 的 API 是为了易于使用而构建的。因此，通过 OkHttp 发送请求既快又无烦恼。以下是一个 `post(String, String)` 方法，它接受一个 URL 和 JSON 请求体作为参数，并将带有 JSON 体的 POST 请求发送到指定的 URL：

```
fun post(url: String, json: String): String {
  val mediaType: MediaType = MediaType.parse("application/json;
                                              charset=utf-8")
  val client:OkHttpClient = OkHttpClient()
  val body: RequestBody = RequestBody.create(mediaType, json)

  val request: Request = Request.Builder()
                                .url(url)
                                .post(body)
                                .build()

  val response: Response = client.newCall(request).execute()
  return response.body().string()
}
```

使用前面的函数很简单。像调用任何其他函数一样，用适当的值调用它：

```
val fullName: String = "John Wayne"
val response = post("http://example.com", "{ \"full_name\": $fullName")

println(response)
```

简单，对吧？很高兴你同意。使用 OkHttp 与远程服务器通信很有趣，但使用 Retrofit 来做这件事更有趣。我们几乎准备好使用 Retrofit 了，但在我们探索 Retrofit 之前，正确地模拟我们将在 HTTP 请求中发送的数据是个好主意。

# 模拟请求数据

我们将使用数据类来模拟我们希望发送到 API 的 HTTP 请求数据。在 `remote` 包中创建一个 `request` 包。有四个明显的请求包含数据负载，我们将向 API 发送这些请求。这些请求分别是登录请求、消息请求、状态更新请求和包含用户数据的请求。这四个请求将分别由 `LoginRequestObject`、`MessageRequestObject`、`StatusUpdateRequestObject` 和 `UserRequest` 对象来模拟。

以下代码片段显示了 `LoginRequestObject` 数据类。请将其添加到 `request` 包中，并为后续的其他请求对象做同样的事情：

```
package com.example.messenger.data.remote.request

data class LoginRequestObject(
  val username: String,
  val password: String
)
```

`LoginRequestObject` 数据类具有 `username` 和 `password` 属性，因为这些是需要提供给 API 登录端点的凭证。`MessageRequestObject` 数据类如下：

```
package com.example.messenger.data.remote.request

data class MessageRequestObject(val recipientId: Long, val message: String)
```

`MessageRequestObject` 也有两个属性。这些是 `recipientId`——接收消息的用户 ID，以及 `message`——发送的消息正文：

```
package com.example.messenger.data.remote.request

data class StatusUpdateRequestObject(val status: String)
```

`StatusUpdateRequestObject` 数据类有一个单一的 `status` 属性。正如其名所示，这是用户想要更新其当前状态消息的状态：

```
package com.example.messenger.data.remote.request

data class UserRequestObject(
  val username: String,
  val password: String,
  val phoneNumber: String = ""
)
```

`UserRequestObject` 与 `LoginRequestObject` 类似，但额外包含一个 `phoneNumber` 属性。此请求对象有多种用途，例如用于包含发送到 API 的用户注册数据。

创建了必要的请求对象后，我们可以继续创建实际的 `MessengerApiService`。

# 创建 Messenger API 服务

现在是我们创建一个执行与我们在第四章设计并实现 Messenger 后端中创建的 Messenger API 通信的至关重要的工作的服务的时候了。我们将使用 Retrofit 和 Retrofit 的 RxJava 适配器来创建此服务。Retrofit 是由 Square Inc. 为 Android 和 Java 构建的类型安全的 HTTP 客户端，RxJava 是用 Java 编写并针对 Java 的开源的 ReactiveX 实现。

在本章开头，我们使用以下行将 Retrofit 添加到我们的 Android 项目中：

```
implementation "com.squareup.retrofit2:retrofit:2.3.0"
```

我们还在模块级别的 `build.gradle` 脚本中添加了 Retrofit 的 RxJava 适配器依赖，如下所示：

```
implementation "com.squareup.retrofit2:adapter-rxjava2:2.3.0"
```

使用 Retrofit 创建服务的第一步是定义一个接口，该接口描述了你的 HTTP API。在你的应用程序 `source` 包中创建一个 `service` 包，并添加 `MessengerApiService` 接口，如下所示：

```
package com.example.messenger.service

import com.example.messenger.data.remote.request.LoginRequestObject
import com.example.messenger.data.remote.request.MessageRequestObject
import com.example.messenger.data.remote.request.StatusUpdateRequestObject
import com.example.messenger.data.remote.request.UserRequestObject
import com.example.messenger.data.vo.*
import io.reactivex.Observable
import okhttp3.ResponseBody
import retrofit2.Retrofit
import retrofit2.adapter.rxjava2.RxJava2CallAdapterFactory
import retrofit2.converter.gson.GsonConverterFactory
import retrofit2.http.*

interface MessengerApiService {

  @POST("login")
  @Headers("Content-Type: application/json")
  fun login(@Body user: LoginRequestObject):
           Observable<retrofit2.Response<ResponseBody>>

  @POST("users/registrations")
  fun createUser(@Body user: UserRequestObject): Observable<UserVO>

  @GET("users")
  fun listUsers(@Header("Authorization") authorization: String):
               Observable<UserListVO>

  @PUT("users")
  fun updateUserStatus(
    @Body request: StatusUpdateRequestObject,
    @Header("Authorization") authorization: String): Observable<UserVO>

  @GET("users/{userId}")
  fun showUser(
    @Path("userId") userId: Long,
    @Header("Authorization") authorization: String): Observable<UserVO>

  @GET("users/details")
  fun echoDetails(@Header("Authorization") authorization: String): Observable<UserVO>

  @POST("messages")
  fun createMessage(
    @Body messageRequestObject: MessageRequestObject,
    @Header("Authorization") authorization: String): Observable<MessageVO>

  @GET("conversations")
  fun listConversations(@Header("Authorization") authorization: String):
                       Observable<ConversationListVO>

  @GET("conversations/{conversationId}")
  fun showConversation(
    @Path("conversationId") conversationId: Long,
    @Header("Authorization") authorization: String):Observable<ConversationVO>
}
```

如前述代码片段所示，Retrofit 依赖于使用注解来正确描述要发送的 HTTP 请求。以下是一个示例片段：

```
@POST("login")
@Headers("Content-Type: application/json")
fun login(@Body user: LoginRequestObject): Observable<retrofit2.Response<ResponseBody>>
```

`@POST` 注解告诉 Retrofit，此函数描述了一个映射到 `/login` 路径的 HTTP POST 请求。`@Headers` 注解用于指定 HTTP 请求的头部。在前述代码片段中描述的 HTTP 请求中，`Content-Type` 头部已被设置为 `application/json`。因此，此请求发送的内容是 JSON。

`@Body` 注解指定传递给 `login()` 的 `user` 参数包含要发送到 API 的 JSON 请求体中的数据。`user` 是 `LoginRequestObject` 类型（我们之前创建了此请求对象）。最后，该函数被声明为返回一个包含 `retrofit2.Response` 对象的 `Observable` 对象。

除了 `@POST`、`@Headers` 和 `@Body` 注解之外，我们还使用了 `@GET`、`@PUT`、`@Path` 和 `@Header`。`@GET` 和 `@PUT` 分别用于指定 `GET` 和 `PUT` 请求。`@Path` 注解用于声明一个值作为发送的 HTTP 请求的路径参数。以下是一个示例 `showUser()` 函数：

```
@GET("users/{userId}")
fun showUser(
  @Path("userId") userId: Long,
  @Header("Authorization") authorization: String): Observable<UserVO>
```

`showUser` 是一个描述带有 `users/{userId}` 路径的 GET 请求的函数。`{userId}` 实际上不是 HTTP 请求路径的一部分。Retrofit 会将传递给 `showUser()` 的 `userId` 参数的值替换 `{userId}`。注意 `userId` 被注解为 `@Path("userId")`。这使 Retrofit 知道 `userId` 持有一个值，应该放置在 HTTP 请求 URL 路径中的 `{userId}` 位置。

`@Header` 与 `@Headers` 类似，但不同之处在于它用于在 HTTP 请求中指定单个头键值对。使用 `@Header("Authorization")` 注解授权将 HTTP 请求的 `Authorization` 头设置为授权中持有的值。

现在我们已经创建了一个适当的 `MessengerApiService` 接口来模拟我们的应用程序将要与之通信的 HTTP API，我们需要能够检索这个服务的实例。我们可以通过创建一个负责创建 `MessengerApiService` 实例的 `Factory` 伴生对象来轻松做到这一点：

```
package com.example.messenger.service

import com.example.messenger.data.remote.request.LoginRequestObject
import com.example.messenger.data.remote.request.MessageRequestObject
import com.example.messenger.data.remote.request.StatusUpdateRequestObject
import com.example.messenger.data.remote.request.UserRequestObject
import com.example.messenger.data.vo.*
import io.reactivex.Observable
import okhttp3.ResponseBody
import retrofit2.Retrofit
import retrofit2.adapter.rxjava2.RxJava2CallAdapterFactory
import retrofit2.converter.gson.GsonConverterFactory
import retrofit2.http.*

interface MessengerApiService {
  …
  …

  companion object Factory {
    private var service: MessengerApiService? = null
```

当被调用时，它返回一个 `MessengerApiService` 的实例。如果没有之前创建过实例，将创建一个新的 `MessengerApiService` 实例。

```
   fun getInstance(): MessengerApiService {
      if (service == null) {

        val retrofit = Retrofit.Builder()
                 .addCallAdapterFactory(RxJava2CallAdapterFactory.create())                   
                 .addConverterFactory(GsonConverterFactory.create())
                 .baseUrl("{AWS_URL}") 
                 // replace AWS_URL with URL of AWS EC2 
                 // instance deployed in the previous chapter
                 .build()

        service = retrofit.create(MessengerApiService::class.java)
      }

      return service as MessengerApiService
    }
  }
}
```

`Factory` 拥有一个单一的 `getInstance()` 函数，当被调用时构建并返回一个 `MessengerApiService` 的实例。使用 `Retrofit.Builder` 的一个实例来创建接口。我们将使用的 `CallAdapterFactory` 设置为 `RxJava2CallAdapterFactory`，并将使用的 `ConverterFactory` 设置为 `GsonConverterFactory`（这处理 JSON 序列化和反序列化）。不要忘记将 `"{AWS_URL}"` 替换为在 第四章，“使用 Spring Boot 2.0 设计和实现 Messenger 后端”中部署的 Messenger API AWS EC2 实例的 URL。

成功创建 `Retrofit.Builder()` 实例后，我们使用它来创建 `MessengerApiService` 的实例，如下所示：

```
service = retrofit.create(MessengerApiService::class.java)
```

最后，服务通过 `getInstance()` 返回以供使用。尽管我们已经创建了一个适合与 Messenger API 通信的服务，但如果没有在 `AndroidManifest` 中指定必要的权限，则无法用于与网络通信。打开项目的 `AndroidManifest` 并在 `<manifest></manifest>` 标签内添加以下两行代码：

```
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
```

现在我们已经准备好了消息服务，是时候创建适当的仓库来利用这个服务了。

# 实现数据仓库

您已经熟悉了仓库，因此无需对它们进行介绍。我们即将创建的仓库与 第四章，“使用 Spring Boot 2.0 设计和实现 Messenger 后端”中为 Messenger API 创建的仓库类似。唯一的区别是我们即将实现的仓库的数据源是一个远程服务器，而不是位于主机上的数据库。

在 `remote` 包内创建一个 `repository` 包。首先，我们将实现一个用户存储库来检索与应用程序用户相关的数据。将以下内容的 `UserRepository` 接口添加到存储库中：

```
package com.example.messenger.data.remote.repository

import com.example.messenger.data.vo.UserListVO
import com.example.messenger.data.vo.UserVO
import io.reactivex.Observable

interface UserRepository {

  fun findById(id: Long): Observable<UserVO>
  fun all(): Observable<UserListVO>
  fun echoDetails(): Observable<UserVO>
}
```

由于这是一个接口，我们需要创建一个实现 `UserRepository` 中指定函数的类。我们将把这个类命名为 `UserRepositoryImpl`。在存储库包中创建一个新的 `UserRepositoryImpl`，如下所示：

```
package com.example.messenger.data.remote.repository

import android.content.Context
import com.example.messenger.service.MessengerApiService
import com.example.messenger.data.local.AppPreferences
import com.example.messenger.data.vo.UserListVO
import com.example.messenger.data.vo.UserVO
import io.reactivex.Observable

class UserRepositoryImpl(ctx: Context) : UserRepository {

  private val preferences: AppPreferences = AppPreferences.create(ctx)
  private val service: MessengerApiService = MessengerApiService.getInstance()

  override fun findById(id: Long): Observable<UserVO> {
    return service.showUser(id, preferences.accessToken as String)
  }

  override fun all(): Observable<UserListVO> {
    return service.listUsers(preferences.accessToken as String)
  }

  override fun echoDetails(): Observable<UserVO> {
    return service.echoDetails(preferences.accessToken as String)
  }
}
```

前面的 `UserRepositoryImpl` 类有两个实例变量：`preferences` 和 `service`。`preferences` 变量是我们之前创建的 `AppPreferences` 类的实例，而 `service` 是通过在 `MessengerApiService` 接口中的 `Factory` 伴随对象中定义的 `getInstance()` 函数检索到的 `MessengerApiService` 实例。

`UserRepositoryImpl` 为在 `UserRepository` 中定义的 `findById()`、`all()` 和 `echoDetails()` 函数提供实现。这三个函数的实现使用了 `service` 来通过 HTTP 适当的请求从服务器检索所需的数据。`findById()` 调用 `service` 中的 `showUser()` 函数，向 Messenger API 发送请求以检索指定用户 ID 的用户详情。`showUser()` 函数需要当前登录用户的授权令牌作为其第二个参数。我们通过将 `preferences.accessToken` 作为函数的第二个参数传递给 `AppPreferences` 实例来提供所需的令牌。

`all()` 函数使用 `MessengerApiService#listUsers()` 来检索在信使服务上注册的所有用户。`echoDetails()` 函数使用 `MessengerApiService#echoDetails()` 函数来获取当前登录用户的详细信息。

让我们创建一个会话存储库，以方便访问与对话相关的数据。将以下内容的 `ConversationRepository` 接口添加到 `com.example.messenger.data.remote.repository`：

```
package com.example.messenger.data.remote.repository

import com.example.messenger.data.vo.ConversationListVO
import com.example.messenger.data.vo.ConversationVO
import io.reactivex.Observable

interface ConversationRepository {
  fun findConversationById(id: Long): Observable<ConversationVO>

  fun all(): Observable<ConversationListVO>
}
```

现在在包中创建一个相应的 `ConversationRepositoryImpl` 类，如下所示：

```
package com.example.messenger.data.remote.repository

import android.content.Context
import com.example.messenger.service.MessengerApiService
import com.example.messenger.data.local.AppPreferences
import com.example.messenger.data.vo.ConversationListVO
import com.example.messenger.data.vo.ConversationVO
import io.reactivex.Observable

class ConversationRepositoryImpl(ctx: Context) : ConversationRepository {

  private val preferences: AppPreferences = AppPreferences.create(ctx)
  private val service: MessengerApiService = MessengerApiService
                                             .getInstance()

```

它从 Messenger API 中检索与请求的会话 ID 相关的会话信息：

```
  override fun findConversationById(id: Long): Observable<ConversationVO> {
    return service.showConversation(id, preferences.accessToken as String)
  }
```

当被调用时，它从 API 中检索当前用户的全部活跃会话：

```
  override fun all(): Observable<ConversationListVO> {
    return service.listConversations(preferences.accessToken as String)
  }
}
```

`findConversationById(Long)` 通过函数传递的相应 ID 检索对应的会话线程。`all()` 函数简单地检索当前用户的全部活跃会话。

# 创建登录交互器

是时候创建我们的登录交互器了，它将作为登录演示者与之交互的模型。在 `login` 包中创建一个 `LoginInteractor` 接口，包含以下代码：

```
package com.example.messenger.ui.login

import com.example.messenger.data.local.AppPreferences
import com.example.messenger.ui.auth.AuthInteractor

interface LoginInteractor : AuthInteractor {

  interface OnDetailsRetrievalFinishedListener {
    fun onDetailsRetrievalSuccess()
    fun onDetailsRetrievalError()
  }

  fun login(username: String, password: String, 
      listener: AuthInteractor.onAuthFinishedListener)

  fun retrieveDetails(preferences: AppPreferences,
      listener: OnDetailsRetrievalFinishedListener)
}
```

正如你可能已经注意到的，`LoginInteractor` 扩展了 `AuthInteractor`。这与 `LoginView` 扩展 `AuthView` 的方式类似。`AuthInteractor` 接口声明了任何处理身份验证相关逻辑的交互器必须实现的行为和特征。现在让我们实现 `AuthInteractor` 接口。

继续在 `com.exampla.messenger.auth` 包中添加一个 `AuthInteractor` 接口：

```
package com.example.messenger.ui.auth

import com.example.messenger.data.local.AppPreferences
import com.example.messenger.data.remote.vo.UserVO

interface AuthInteractor {

  var userDetails: UserVO
  var accessToken: String
  var submittedUsername: String
  var submittedPassword: String

  interface onAuthFinishedListener {
    fun onAuthSuccess()
    fun onAuthError()
    fun onUsernameError()
    fun onPasswordError()
  }

  fun persistAccessToken(preferences: AppPreferences)

  fun persistUserDetails(preferences: AppPreferences)

}
```

每个 `AuthInteractor` 交互器都必须有以下字段：`userDetails`、`accessToken`、`submittedUsername` 和 `submittedPassword`。此外，实现 `AuthInteractor` 的交互器必须具有 `persistAccessToken(AppPreferences)` 和 `persistUserDetails(AppPreferences)` 方法。正如方法名所暗示的，它们将访问令牌和用户详细信息持久化到应用程序的 `SharedPreferences` 文件中。正如你可能已经猜到的，我们需要为 `LoginInteractor` 创建一个实现类。我们将把这个类称为 `LoginInteractorImpl`。

下面的 `LoginInteractorImpl` 类实现了 `login()` 方法。将其添加到 `ui` 包内的 `login` 包中：

```
package com.example.messenger.ui.login

import com.example.messenger.data.local.AppPreferences
import com.example.messenger.data.remote.request.LoginRequestObject
import com.example.messenger.data.vo.UserVO
import com.example.messenger.service.MessengerApiService
import com.example.messenger.ui.auth.AuthInteractor
import io.reactivex.android.schedulers.AndroidSchedulers
import io.reactivex.schedulers.Schedulers

class LoginInteractorImpl : LoginInteractor {

  override lateinit var userDetails: UserVO
  override lateinit var accessToken: String
  override lateinit var submittedUsername: String
  override lateinit var submittedPassword: String

  private val service: MessengerApiService = MessengerApiService
                                             .getInstance()

  override fun login(username: String, password: String, 
                     listener: AuthInteractor.onAuthFinishedListener) {
    when {

```

如果在登录表单中提交了空 `username`，则 `username` 无效。当发生这种情况时，会调用监听器的 `onUsernameError()` 函数：

```
username.isBlank() -> listener.onUsernameError() 
```

当提交空 `password` 时，调用监听器的 `onPasswordError()` 函数：

```
 password.isBlank() -> listener.onPasswordError()
else -> {     
```

初始化模型的 `submittedUsername` 和 `submittedPassword` 字段，并创建适当的 `LoginRequestObject`：

```
submittedUsername = username
submittedPassword = password
val requestObject = LoginRequestObject(username, password)   
```

使用 `MessengerApiService` 向 Messenger API 发送登录请求。

```
service.login(requestObject)
       .subscribeOn(Schedulers.io())   
         // subscribing Observable to Scheduler thread
       .observeOn(AndroidSchedulers.mainThread())  
          // setting observation to be done on the main thread
       .subscribe({ res ->
         if (res.code() != 403) {
           accessToken = res.headers()["Authorization"] as String
           listener.onAuthSuccess()
         } else {

```

当服务器返回 HTTP 403（禁止）状态码时，会发生分支。这表示登录失败，用户无权访问服务器。

无权访问服务器。

```
            listener.onAuthError()
          }
        }, { error ->
          listener.onAuthError()
          error.printStackTrace()
        })
      }
    }
  }
}
```

`login()` 函数通过首先验证提供的 `username` 和 `password` 参数是否为空来工作。当遇到空用户名时，会调用 `onAuthFinishedListener` 的 `onUsernameError()` 函数，当遇到空密码时，会调用 `onPasswordError()` 函数。如果提供的用户名和密码都不为空，它将利用 `MessengerApiService` 向信使 API 发送登录请求。如果登录请求成功，它将 `accessToken` 属性设置为从 API 响应的 `Authorization` 标头中检索到的访问令牌，然后调用监听器的 `onAuthSuccess()` 函数。在登录请求失败的情况下，会调用 `onAuthError()` 监听器函数。

在理解了登录过程之后，将下面的 `retrieveDetails()`、`persistAccessToken()` 和 `persistUserDetails()` 方法添加到 `LoginInteractorImpl` 中：

```
  override fun retrieveDetails(preferences: AppPreferences,
              listener: LoginInteractor.OnDetailsRetrievalFinishedListener) {

```

它在初始登录时检索用户详细信息：

```
  service.echoDetails(preferences.accessToken as String)
         .subscribeOn(Schedulers.io())
         .observeOn(AndroidSchedulers.mainThread())
         .subscribe({ res ->
           userDetails = res
           listener.onDetailsRetrievalSuccess()},
         { error ->
           listener.onDetailsRetrievalError()
           error.printStackTrace()})
}

override fun persistAccessToken(preferences: AppPreferences) {
  preferences.storeAccessToken(accessToken)
}

override fun persistUserDetails(preferences: AppPreferences) {
  preferences.storeUserDetails(userDetails)
}
```

一定要仔细阅读前面代码片段中的注释。它们详细解释了 `LoginInteractor` 的工作原理。现在是时候开始处理 `LoginPresenter` 了。

# 创建登录演示者

正如我们在第三章中看到的，**演示者**是视图和模型之间的中间人。为了正确促进干净的视图-模型交互，有必要为视图创建合适的演示者。创建一个演示者相当简单。我们首先需要创建一个接口，以正确声明演示者将展示的行为。在`login`包中创建一个`LoginPresenter`接口，代码如下：

```
package com.example.messenger.ui.login

interface LoginPresenter {
  fun executeLogin(username: String, password: String)
}
```

如前代码片段所示，我们希望一个作为`LoginView`的`LoginPresenter`的类的`executeLogin(String, String)`函数。这个函数将由视图调用，然后将与处理应用程序登录逻辑的模型进行交互。我们需要创建一个实现`LoginPresenter`的`LoginPresenterImpl`类：

```
package com.example.messenger.ui.login

import com.example.messenger.data.local.AppPreferences
import com.example.messenger.ui.auth.AuthInteractor

class LoginPresenterImpl(private val view: LoginView) : 
      LoginPresenter, AuthInteractor.onAuthFinishedListener, 
      LoginInteractor.OnDetailsRetrievalFinishedListener {

  private val interactor: LoginInteractor = LoginInteractorImpl()
  private val preferences: AppPreferences = AppPreferences.create(view.getContext())

  override fun onPasswordError() {
    view.hideProgress()
    view.setPasswordError()
  }

  override fun onUsernameError() {
    view.hideProgress()
    view.setUsernameError()
  }

  override fun onAuthSuccess() {
    interactor.persistAccessToken(preferences)
    interactor.retrieveDetails(preferences, this)
  }

  override fun onAuthError() {
    view.showAuthError()
    view.hideProgress()
  }

  override fun onDetailsRetrievalSuccess() {
    interactor.persistUserDetails(preferences)
    view.hideProgress()
    view.navigateToHome()
  }

  override fun onDetailsRetrievalError() {
    interactor.retrieveDetails(preferences, this)
  }

  override fun executeLogin(username: String, password: String) {
    view.showProgress()
    interactor.login(username, password, this)
  }
}
```

`LoginPresenterImpl`类实现了`LoginPresenter`、`AuthInteractor.onAuthFinishedListener`和`LoginInteractor.OnDetailsRetrievalFinishedListener`，因此实现了接口所需的所有行为。`LoginPresenterImpl`覆盖了七个函数：`onPasswordError()`、`onUsernameError()`、`onAuthSuccess()`、`onAuthError()`、`onDetailsRetrievalSuccess()`、`onDetailsRetrievalError()`和`executeLogin(String, String)`。`LoginPresenter`和`LoginInteractor`之间的交互可以在`onAuthSuccess()`和`executeLogin(String, String)`函数中看到。当用户提交他们的登录详情时，`LoginView`调用`LoginPresenter`中的`executeLogin(String, String)`函数。反过来，`LoginPresenter`使用`LoginInteractor`通过调用`login(String, String)`函数来处理实际的登录过程。

如果用户登录成功，`LoginInteractor`将调用`LoginPresenter`的`onAuthSuccess()`回调函数。这随后导致存储服务器返回的访问令牌和检索已登录用户的账户详情。当服务器拒绝登录请求时，调用`onAuthError()`并显示一个有用的错误信息给用户。

当交互者成功检索到用户的账户详情时，`LoginPresenter`的`onDetailsRetrievalSuccess()`回调被调用。这导致存储账户详情。在登录过程中向用户显示的进度条随后通过`view.hideProgress()`隐藏，之后用户通过`view.navigateToHome()`导航到主屏幕。如果用户详情的检索失败，`LoginInteractor`将调用`onDetailsRetrievalError()`。然后演示者通过再次调用`interactor.retrieveDetails(preferences, this)`请求再次尝试检索用户的账户详情。

# 完成 LoginView

如果你还记得，我们之前没有完成 `LoginView` 的实现。例如 `navigateToSignUp()`、`navigateToHome()` 和 `onClick(view: View)` 等函数都留有空的实现。此外，`LoginView` 与 `LoginPresenter` 没有任何交互。现在让我们来修复这个问题，好吗？

首先，为了将用户导航到注册屏幕和主页，我们需要这些屏幕的视图存在。我们现在不会关注实现它们的布局（这将在接下来的章节中完成）。我们只需要它们存在。在 `com.example.messenger.ui` 下创建 `signup` 和 `main` 包。在 `signup` 包中创建一个名为 `SignUpActivity` 的新空活动，在 `main` 包中创建一个名为 `MainActivity` 的新空活动。

现在打开 `LoginActivity.kt`。我们需要修改之前提到的函数以执行它们各自的任务。此外，我们需要为 `LoginPresenter` 实例和 `AppPreferences` 实例添加私有属性。以下代码片段展示了这些更改：

首先，将以下属性添加到 `LoginActivity` 类的顶部。

```
  private lateinit var progressBar: ProgressBar
  private lateinit var presenter: LoginPresenter 
  private lateinit var preferences: AppPreferences 
```

现在修改 `navigateToSignUp()`、`navigateToHome()` 和 `onClick(view: View)`，如下面的代码片段所示：

```
override fun navigateToSignUp() {
  startActivity(Intent(this, SignUpActivity::class.java))
}

override fun navigateToHome() {
  finish()
  startActivity(Intent(this, MainActivity::class.java))
}

override fun onClick(view: View) {
  if (view.id == R.id.btn_login) {
    presenter.executeLogin(etUsername.text.toString(),
                           etPassword.text.toString())
  } else if (view.id == R.id.btn_sign_up) {
    navigateToSignUp()
  }
}
```

`navigateToSignUp()` 使用显式意图在调用时启动 `SignUpActivity`。`navigateToHome()` 与 `navigateToSignUp()` 操作类似——它启动 `MainActivity`。`navigateToHome()` 和 `navigateToSignUp()` 之间的一个主要区别是，在启动 `MainActivity` 之前，`navigateToHome()` 会通过调用 `finish()` 销毁当前的 `LoginActivity` 实例。

`onClick()` 方法在点击登录按钮的情况下使用 `LoginPresenter` 开始登录过程。否则，如果点击注册按钮，将使用 `navigateToSignUp()` 启动 `SignUpActivity`。

到目前为止，做得很好！我们已经创建了与登录相关应用逻辑所需的所有视图、表示和模型。我们需要记住，在我们可以登录用户之前，我们首先需要在平台上注册用户。因此，我们必须实现我们的注册逻辑。我们将在下一节中这样做。

# 开发注册 UI

让我们开发注册用户界面。首先，我们必须实现必要的视图，从 `SignUpActivity` 的布局开始。在我们的 `SignUpActivity` 布局中，我们不需要很多元素。我们需要三个输入字段来获取要注册的用户的用户名、密码和电话号码。此外，我们还需要一个提交注册表单的按钮以及一个进度条来显示注册正在进行。

以下是我们 `activity_sign_up.xml` 布局的代码：

```
<?xml version="1.0" encoding="utf-8"?>
<android.support.constraint.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".ui.signup.SignUpActivity"
    android:paddingTop="@dimen/default_padding"
    android:paddingBottom="@dimen/default_padding"
    android:paddingStart="@dimen/default_padding"
    android:paddingEnd="@dimen/default_padding"
    android:orientation="vertical"
    android:gravity="center_horizontal">
    <EditText
        android:id="@+id/et_username"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="@string/username"
        android:inputType="text"/>
    <EditText
        android:id="@+id/et_phone"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginTop="@dimen/default_margin"
        android:hint="@string/phone_number"
        android:inputType="phone"/>
    <EditText
        android:id="@+id/et_password"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginTop="@dimen/default_margin"
        android:hint="@string/password"
        android:inputType="textPassword"/>
    <Button
        android:id="@+id/btn_sign_up"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginTop="@dimen/default_margin"
        android:text="@string/sign_up"/>
    <ProgressBar
        android:id="@+id/progress_bar"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginTop="@dimen/default_margin"
        android:visibility="gone"/>
</android.support.constraint.ConstraintLayout>
```

之前编写的 XML 布局的视觉翻译如下：

![图片](img/3479aa1d-d799-44cf-a594-53284db40d6e.jpg)

正如你所见，我们设计的布局包含了我们之前提到的所有必要元素。

# 创建注册交互器

现在，我们将实现一个注册交互器，作为我们尚未实现的注册展示器的通信模型。在 `signup` 包内创建一个 `SignUpInteractor` 接口，如下所示：

```
package com.example.messenger.ui.signup

import com.example.messenger.ui.auth.AuthInteractor

interface SignUpInteractor : AuthInteractor {

  interface OnSignUpFinishedListener {
    fun onSuccess()
    fun onUsernameError()
    fun onPasswordError()
    fun onPhoneNumberError()
    fun onError()
  }

  fun signUp(username: String, phoneNumber: String, password: String, 
             listener: OnSignUpFinishedListener)

  fun getAuthorization(listener: AuthInteractor.onAuthFinishedListener)
}
```

你可能已经注意到 `SignUpInteractor` 继承自 `AuthInteractor`。类似于 `LoginInteractor`，`SignUpInteractor` 需要使用 `userDetails`、`accessToken`、`submittedUsername` 和 `submittedPassword` 属性。此外，`SignUpInteractor` 需要能够通过 `persistAccessToken(AppPreferences)` 和 `persistUserDetails(AppPreferences)` 函数持久化用户的访问令牌和用户详情，这些函数在 `AuthInteractor` 接口中声明。

我们在 `SignUpInteractor` 中创建了一个 `OnSignUpFinishedListener` 接口，声明了必须由 `OnSignUpFinishedListener` 实现的回调。当我们将它实现时，这个监听器将是我们的 `SignUpPresenter`。

在创建 `SignUpInteractorImpl` 时，我们首先应该从其属性声明和 `login()` 方法的实现开始。创建如下所示的 `SignUpInteractorImpl`。确保将其添加到与 `SignUpInteractor` 相同的包中：

```
package com.example.messenger.ui.signup

import android.text.TextUtils
import android.util.Log
import com.example.messenger.data.local.AppPreferences
import com.example.messenger.data.remote.request.LoginRequestObject
import com.example.messenger.data.remote.request.UserRequestObject
import com.example.messenger.data.vo.UserVO
import com.example.messenger.service.MessengerApiService
import com.example.messenger.ui.auth.AuthInteractor
import io.reactivex.android.schedulers.AndroidSchedulers
import io.reactivex.schedulers.Schedulers

class SignUpInteractorImpl : SignUpInteractor {

  override lateinit var userDetails: UserVO
  override lateinit var accessToken: String
  override lateinit var submittedUsername: String
  override lateinit var submittedPassword: String

  private val service: MessengerApiService = MessengerApiService
                                             .getInstance()

  override fun signUp(username: String, 
                      phoneNumber: String, password: String,
                      listener: SignUpInteractor.OnSignUpFinishedListener){
    submittedUsername = username
    submittedPassword = password
    val userRequestObject = UserRequestObject(username, password,
                                              phoneNumber)

    when {
      TextUtils.isEmpty(username) -> listener.onUsernameError()
      TextUtils.isEmpty(phoneNumber) -> listener.onPhoneNumberError()
      TextUtils.isEmpty(password) -> listener.onPasswordError()
      else -> {
```

使用 `MessengerApiService` 将新用户注册到 Messenger 平台。

```

        service.createUser(userRequestObject)
               .subscribeOn(Schedulers.io())
               .observeOn(AndroidSchedulers.mainThread())
               .subscribe({ res ->
          userDetails = res
          listener.onSuccess()
        }, { error ->
          listener.onError()
          error.printStackTrace()
        })
      }
    }
  }
}
```

现在将 `getAuthorization()`、`persistAccessToken()` 和 `persistUserDetails()` 方法添加到 `SignUpInteractorImpl` 中：

```
  override fun getAuthorization(listener: 
                  AuthInteractor.onAuthFinishedListener) {
    val userRequestObject = LoginRequestObject(submittedUsername,
                                               submittedPassword)

```

让我们使用 `MessengerApiService` 将注册用户登录到平台：

```
    service.login(userRequestObject)
           .subscribeOn(Schedulers.io())
           .observeOn(AndroidSchedulers.mainThread())
           .subscribe( { res ->
      accessToken = res.headers()["Authorization"] as String

```

现在，用户已成功登录。因此，我们调用监听器的 `onAuthSuccess()` 回调：

```

    listener.onAuthSuccess()

  }, { error ->
    listener.onAuthError()
    error.printStackTrace()
  })
}

override fun persistAccessToken(preferences: AppPreferences) {
  preferences.storeAccessToken(accessToken)
}

override fun persistUserDetails(preferences: AppPreferences) {
  preferences.storeUserDetails(userDetails)
}
```

`SignUpInteractorImpl` 类是 `SignUpInteractor` 接口的直接实现。第 19 行到 22 行包含了 `userDetails`、`accessToken`、`submittedUsername` 和 `submittedPassword` 的属性声明，这些属性必须由 `AuthInteractor` 拥有。`signUp(String, String, String, SignUpInteractor.OnSignUpFinishedListener)` 包含了应用程序的注册逻辑。如果用户提交的所有值都有效，则用户将通过我们使用 Retrofit 创建的 `MessengerApiService` 的 `createUser(UserRequestObject)` 函数在平台上注册。

调用 `getAuthorization(AuthInteractor.onAuthFinishedListener)` 以授权新注册的 messenger 平台用户。请确保仔细阅读 `SignUpInteractorImpl` 中的注释以获取更多信息。

接下来，我们的任务是创建 `SignUpPresenter`。

# 创建注册展示器

当我们创建 `LoginPresenter` 时，我们需要创建一个 `SignUpPresenter` 接口以及一个 `SignUpPresenterImpl` 类。我们创建的 `SignUpPresenter` 并不复杂。对于这个应用程序，我们需要我们的注册展示器拥有 `AppPreferences` 类型的属性以及一个执行注册过程的函数。以下是我们创建的 `SignUpPresenter` 接口：

```
package com.example.messenger.ui.signup

import com.example.messenger.data.local.AppPreferences

interface SignUpPresenter {
  var preferences: AppPreferences

  fun executeSignUp(username: String, phoneNumber: String, password: String)
}
```

现在，以下是我们的 `SignUpPresenter` 实现的代码：

```
package com.example.messenger.ui.signup

import com.example.messenger.data.local.AppPreferences
import com.example.messenger.ui.auth.AuthInteractor

class SignUpPresenterImpl(private val view: SignUpView): SignUpPresenter,
                          SignUpInteractor.OnSignUpFinishedListener,
                          AuthInteractor.onAuthFinishedListener {

  private val interactor: SignUpInteractor = SignUpInteractorImpl()
  override var preferences: AppPreferences = AppPreferences
                                             .create(view.getContext())
```

当用户成功注册时，以下 `onSuccess()` 回调会被调用：

```
 override fun onSuccess() {
    interactor.getAuthorization(this)
  }
```

用户注册过程中发生错误时调用的回调：

```

  override fun onError() {
    view.hideProgress()
    view.showSignUpError()
  }

  override fun onUsernameError() {
    view.hideProgress()
    view.setUsernameError()
  }

  override fun onPasswordError() {
    view.hideProgress()
    view.setPasswordError()
  }

  override fun onPhoneNumberError() {
    view.hideProgress()
    view.setPhoneNumberError()
  }

  override fun executeSignUp(username: String, phoneNumber: String, 
                             password: String) {
    view.showProgress()
    interactor.signUp(username, phoneNumber, password, this)
  }

  override fun onAuthSuccess() {
    interactor.persistAccessToken(preferences)
    interactor.persistUserDetails(preferences)
    view.hideProgress()
    view.navigateToHome()
  }

  override fun onAuthError() {
    view.hideProgress()
    view.showAuthError()
  }
}
```

之前的 `SignUpPresenterImpl` 类实现了 `SignUpPresenter`、`SignUpInteractor.OnSignUpFinishedListener` 和 `AuthInteractor.onAuthFinishedListener` 接口，因此提供了许多必需函数的实现。这些函数包括 `onSuccess()`、`onError()`、`onUsernameError()`、`onPasswordError()`、`onPhoneNumberError()`、`executeSignUp(String, String, String)`、`onAuthSuccess()` 和 `onAuthError()`。`SignUpPresenterImpl` 以 `SignUpView` 类型作为其主构造函数的单个参数。

`executeSignUp(String, String, String)` 是一个由 `SignUpView` 调用来开始用户注册过程的函数。当用户的注册请求成功时，会调用 `onSuccess()`。该函数立即调用交互器的 `getAuthorization()` 函数来获取新注册用户的访问令牌。在注册请求失败的情况下，会调用 `onError()` 回调。这会隐藏向用户显示的进度条，并显示适当的错误信息。

当提交的用户名、密码或电话号码发生错误时，会调用 `onUsernameError()`、`onPasswordError()` 和 `onPhoneNumberError()` 方法作为回调。`onAuthSuccess()` 是在授权过程成功时调用的回调。另一方面，当授权失败时，会调用 `onAuthError()`。

# 创建注册视图

是时候开始工作于 `SignUpView` 了。首先我们需要创建一个 `SignUpView` 接口，之后我们将让 `SignUpActivity` 实现这个接口。请注意，在我们的应用中，一个 `SignUpView` 是 `BaseView` 和 `AuthView` 的扩展。以下就是 `SignUpView` 接口：

```
package com.example.messenger.ui.signup

import com.example.messenger.ui.auth.AuthView
import com.example.messenger.ui.base.BaseView

interface SignUpView : BaseView, AuthView {

  fun showProgress()
  fun showSignUpError()
  fun hideProgress()
  fun setUsernameError()
  fun setPhoneNumberError()
  fun setPasswordError()
  fun navigateToHome()
}
```

现在我们将修改项目中的 `SignUpActivity` 类以实现 `SignUpView` 并使用 `SignUpPresenter`。将以下代码片段中的更改添加到 `SignUpActivity`：

```
package com.example.messenger.ui.signup

import android.content.Context
import android.content.Intent
import android.support.v7.app.AppCompatActivity
import android.os.Bundle
import android.view.View
import android.widget.Button
import android.widget.EditText
import android.widget.ProgressBar
import android.widget.Toast
import com.example.messenger.R
import com.example.messenger.data.local.AppPreferences
import com.example.messenger.ui.main.MainActivity

class SignUpActivity : AppCompatActivity(), SignUpView, View.OnClickListener {

  private lateinit var etUsername: EditText
  private lateinit var etPhoneNumber: EditText
  private lateinit var etPassword: EditText
  private lateinit var btnSignUp: Button
  private lateinit var progressBar: ProgressBar
  private lateinit var presenter: SignUpPresenter

  override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    setContentView(R.layout.activity_sign_up)
    presenter = SignUpPresenterImpl(this)
    presenter.preferences = AppPreferences.create(this)
    bindViews()
  }

  override fun bindViews() {
    etUsername = findViewById(R.id.et_username)
    etPhoneNumber = findViewById(R.id.et_phone)
    etPassword = findViewById(R.id.et_password)
    btnSignUp = findViewById(R.id.btn_sign_up)
    progressBar = findViewById(R.id.progress_bar)
    btnSignUp.setOnClickListener(this)
  }

  override fun showProgress() {
    progressBar.visibility = View.VISIBLE
  }

  override fun hideProgress() {
    progressBar.visibility = View.GONE
  }

  override fun navigateToHome() {
    finish()
    startActivity(Intent(this, MainActivity::class.java))
  }

  override fun onClick(view: View) {
    if (view.id == R.id.btn_sign_up) {
      presenter.executeSignUp(etUsername.text.toString(),
                              etPhoneNumber.text.toString(),
                              etPassword.text.toString())
    }
  }
}
```

现在将下面的 `setUsernameError()`, `setPasswordError()`, `showAuthError()`, `showSignUpError()` 和 `getContext()` 函数添加到 `SignUpActivity` 中：

```
override fun setUsernameError() {
  etUsername.error = "Username field cannot be empty"
}

override fun setPhoneNumberError() {
  etPhoneNumber.error = "Phone number field cannot be empty"
}

override fun setPasswordError() {
  etPassword.error = "Password field cannot be empty"
}

override fun showAuthError() {
  Toast.makeText(this, "An authorization error occurred.
                        Please try again later.",
                 Toast.LENGTH_LONG).show()
}

override fun showSignUpError() {
  Toast.makeText(this, "An unexpected error occurred.
                        Please try again later.",
                 Toast.LENGTH_LONG).show()
}

override fun getContext(): Context {
  return this
}
```

到目前为止，你已经做得很好了！在这个阶段，我们完成了 Messenger 应用开发的一半。你的努力值得掌声。但我们还有一些工作要做——特别是主 UI。我们将在下一章完成 Messenger 应用的剩余部分。

# 摘要

在本章中，我们开始了 Messenger Android 应用的开发。在这个过程中，我们涵盖了众多主题。我们深入研究了 Model-View-Presenter 模式，并详细探讨了如何利用这种现代开发方法创建应用。

在本章的进一步内容中，我们学习了响应式编程，并广泛使用了 RxJava 和 RxAndroid。我们学习了如何使用 OkHttp 和 Retrofit 与远程服务器进行通信，之后我们更进一步，实现了一个完全功能性的 Retrofit 服务，用于与我们开发的第四章中提到的信使 API 进行通信，*使用 Spring Boot 2.0 设计和实现信使后端*。

在下一章中，我们将完成我们对信使应用程序的工作。
