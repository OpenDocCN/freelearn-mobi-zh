# 第十一章：使用 Kotlin 的响应式微服务

在本章中，我们将通过使用 Kotlin 编程语言构建一个微服务来运用我们迄今为止学到的技能。我们还想让这个微服务是响应式的，并且尽可能接近现实生活。为此，我们将使用 Vert.x 框架，其优势将在下一节中列出。

你可能已经厌倦了创建待办事项或购物清单。

因此，相反，这个微服务将是一个 *猫收容所*。这个微服务应该能够做到以下事情：

+   提供一个我们可以 ping 的端点来检查服务是否正在运行

+   列出目前收容所中的猫咪

+   提供一种方法来添加新的猫咪

你需要开始以下内容：

+   JDK 1.8 或更高版本

+   IntelliJ IDEA

+   Gradle 4.2 或更高版本

+   PostgreSQL 9.4 或更高版本

本章将假设你已经安装了 `PostgreSQL` 并且你对它有基本的工作知识。如果没有，请参阅官方文档：[`www.postgresql.org/docs/9.4/static/tutorial-install.html`](https://www.postgresql.org/docs/9.4/static/tutorial-install.html)。

在本章中，我们将涵盖以下主题：

+   开始使用 Vert.x

+   处理请求

+   测试

+   与数据库一起工作

+   EventBus

# 开始使用 Vert.x

我们将为我们的微服务使用的框架称为 **Vert.x**。它是一个与 **reactive extensions** 共享许多共同点的响应式框架，我们在 第七章 *保持响应式* 中讨论了它。它是异步和非阻塞的。

让我们通过一个具体的例子来理解这意味着什么。

我们将从一个新的 Kotlin Gradle 项目开始。从你的 IntelliJ IDEA 中，打开 File | New | Project，在 New Project 向导中选择 Gradle | Kotlin，然后点击 Finish。给你的项目一个 `GroupId`（我选择了 `me.soshin`）和一个 `ArtifactId`（在我的例子中是 `catsShelter`）。

Gradle 是一个构建工具，类似于 Maven 和 Ant。它有一个很好的语法，并以优化的方式编译你的项目。你可以在这里了解更多：[`gradle.org/`](https://gradle.org/)。

在下一屏上，选择 Use auto-import 和 Create directories for empty content roots，然后点击 Finish。

接下来，将以下依赖项添加到你的 `build.gradle` 文件中。

```java
dependencies {
    def $vertx_version = '3.5.1'
    ...
    compile group: 'io.vertx', name: 'vertx-core', version: $vertx_version
    compile group: 'io.vertx', name: 'vertx-web', version: $vertx_version
    compile group: 'io.vertx', name: 'vertx-lang-kotlin', version: $vertx_version
    compile group: 'io.vertx', name: 'vertx-lang-kotlin-coroutines', version: $vertx_version
}
```

以下是对每个依赖项的解释：

+   `vertx-core` 是核心库

+   `vertx-web` 是必需的，因为我们希望我们的服务是基于 REST 的

+   `vertx-lang-kotlin` 提供了使用 Vert.x 编写 Kotlin 代码的惯用方法

+   最后，`vertx-lang-kotlin-coroutines` 与我们在 第九章 详细讨论的协程集成，*专为并发设计*

注意，我们定义了一个变量来指定我们应该使用 Vert.x 的哪个版本。截至目前，最新稳定版本是 3.5.1，但到你阅读这本书的时候，它将是 3.5.2 或甚至 3.6.0。

作为一般规则，所有 Vert.x 库应该使用相同的版本，这时变量就变得很有用了。

在 `src/main/kotlin` 文件夹中创建一个名为 `Main.kt` 的文件，内容如下：

```java
fun main(vararg args: String) {
   val vertx = Vertx.vertx()

   vertx.createHttpServer().requestHandler{ req ->
            req.response().end("OK")
        }.listen(8080)
}
```

这就是你需要启动一个当你在浏览器中打开 [`localhost:8080`](http://localhost:8080) 时会响应 *OK* 的网络服务器的所有内容。

现在，让我们了解这里实际上发生了什么。我们使用第三章的 **工厂方法** 从 Understanding Structural Patterns 创建一个 Vert.x 实例。 

**Handler** 只是一个简单的监听器，或者是一个订阅者。如果你不记得它是如何工作的，请查看第四章的 Getting Familiar with Behavioral Patterns，了解 **Observable** 设计模式。在我们的情况下，它将为每个新的请求被调用。这就是 Vert.x 的异步特性在起作用。

注意，`requestHandler()` 是一个接收块的函数。像任何其他惯用的 Kotlin 代码一样，你不需要括号。

如果你使用的是 IntelliJ IDEA 等集成开发环境，你可以直接运行它。另一种选择是将以下行添加到你的 `build.gradle` 文件中：

```java
apply plugin: 'application'
mainClassName = "com.gett.MainKt"
```

然后，你可以简单地使用以下命令启动它：

```java
./gradlew run
```

另一个选择是使用 `VertxGradlePlugin` ([`github.com/jponge/vertx-gradle-plugin`](https://github.com/jponge/vertx-gradle-plugin))，它将做同样的事情。

# 路由

注意，无论我们指定哪个 URL，我们总是得到相同的结果。

当然，这不是我们想要达到的目标。让我们先添加最基本的服务端点，它只会告诉我们服务正在运行。

为了做到这一点，我们将使用 `Router`：

```java
val vertx = Vertx.vertx() // Was here before
val router = Router.router(vertx)
...
```

`Router` 允许你为不同的 HTTP 方法和 URL 指定处理器。

但是，默认情况下，它不支持协程。让我们通过创建一个扩展函数来解决这个问题：

```java
fun Route.asyncHandler(fn : suspend (RoutingContext) -> Unit) {
    handler { ctx ->
        launch(ctx.vertx().dispatcher()) {
            try {
                fn(ctx)
            } catch(e: Exception) {
                ctx.fail(e)
            }
        }
    }
}
```

如果你熟悉现代 JavaScript，这类似于 `async() => {}`。

现在，我们可以使用这个新的扩展方法：

```java
router.get("/alive").asyncHandler {
    // Some response comes here
    // We now can use any suspending function in this context
}
```

我们看到了如何在第一个示例中返回一个平面文本响应。所以，让我们返回 JSON 代替。大多数实际应用程序使用 JSON 进行通信。

将以下行添加到你的处理器中：

```java
...
val json = json {
    obj (
       "alive" to true
    )
}
it.respond(json.toString())
...
```

我们声明的另一个扩展函数是 `respond()`。它看起来如下所示：

```java
fun RoutingContext.respond(responseBody: String = "", status: Int = 200) {
    this.response()
            .setStatusCode(status)
            .end(responseBody)
}
```

现在将你的路由器连接到服务器。

你可以通过用以下行替换之前的服务器实例化来实现这一点：

```java
vertx.createHttpServer().
   requestHandler(router::accept).listen(8080)
```

现在，所有路由都将由 `Router` 处理。

你可以在浏览器中打开 `http://localhost:8080/alive` 并确保你得到 `{"alive": true}` 的响应。

恭喜！你已经成功创建了第一个返回 JSON 的路由。从现在起，无论何时你不确定你的应用程序是否正在运行，你都可以简单地使用这个 URL 来检查它。当你使用负载均衡器时，这一点尤为重要，因为负载均衡器需要知道在任何时候有多少应用程序可用。

# 处理请求

我们接下来的任务是向我们的虚拟收容所添加第一只猫。

这应该是一个 `POST` 请求，其中请求体的内容可能看起来像这样：`{"name": "Binky", "age": 4}`。

如果你熟悉像 **curl** 或 **Postman** 这样的工具来发出 `POST` 请求，那很好。如果不熟悉，我们将在下一节编写一个测试来检查这个场景。

我们首先需要做的是在我们初始化我们的路由器之后添加以下行：

```java
router.route().handler(BodyHandler.create())
```

这将告诉 Vert.x 将请求体解析为 JSON，适用于任何请求。另一种方法是使用 `router.route("/*")`。

现在，让我们确定我们的 URL 应该是什么样子。良好的实践是将我们的 API URL 进行版本控制，所以我们希望它如下所示：

```java
api/v1/cats
```

因此，我们可以假设以下：

+   `GET api/v1/cats` 将返回我们庇护所中所有的猫。

+   `POST api/v1/cats` 将添加一只新的猫。

+   `GET api/v1/cats/34` 如果存在，将返回 `ID=34` 的猫，否则返回 404。

理解了这一点后，我们可以继续如下操作：

```java
router.post("/api/v1/cats").asyncHandler { ctx ->
    // Some code of adding a cat comes here
}
router.get("/api/v1/cats").asyncHandler { ctx ->
    // Code for getting all the cats
}

```

最后一个端点需要接收一个路径参数。我们使用分号符号来表示：

```java
router.get("/api/v1/cats/:id").asyncHandler { ctx ->
    // Fetches specific cat
}
```

# Verticles

现在遇到了一个问题。我们的代码位于 `Main.kt` 文件中，它越来越大。我们可以通过使用 verticles 来开始分割它。

你可以把 verticle 看作是一个轻量级 actor。让我们看看以下代码的例子：

```java
class ServerVerticle: CoroutineVerticle() {

    override suspend fun start() {
        val router = router()
        vertx.createHttpServer().requestHandler(router::accept).listen(8080)
    }

    private fun router(): Router {
        val router = Router.router(vertx)
        // Our router code comes here now
        ...
        return router
    }
}
```

现在我们需要启动这个 verticle。有几种不同的方法可以做到这一点，但最简单的方法是将这个类的实例传递给 `deployVerticle()` 方法：

```java
vertx.deployVerticle(ServerVerticle())
```

现在我们的代码被分成两个文件，`ServerVerticle.kt` 和 `Main.kt`。

注意，但是 `/api/v1/cats/` 每次都会重复。有没有一种方法可以消除这种冗余？实际上，有。它被称为 **子路由**。

# 子路由

我们将保持 `/alive` 端点不变，但我们将所有其他端点提取到一个单独的函数中：

```java
private fun apiRouter(): Router {
    val router = Router.router(vertx)

    router.post("/cats").asyncHandler { ctx ->
        ctx.respond(status=501)
    }
    router.get("/cats").asyncHandler { ctx ->
        ...
    }
    router.get("/cats/:id").asyncHandler { ctx ->
        ...
    }
    return router
}
```

有一种更流畅的方式来定义它，但我们保留了原来的方式，因为它更易读。

就像我们向 Vert.x 服务器实例提供主路由器一样，我们现在将子路由器按如下方式提供给主路由器：

```java
router.mountSubRouter("/api/v1", apiRouter())
```

保持我们的代码干净和良好分离非常重要。

# 测试

在我们继续将猫添加到数据库之前，让我们首先编写一些测试来确保到目前为止一切正常。

为了做到这一点，我们将使用 **TestNG** 测试框架。你也可以使用 **JUnit** 或 **VertxUnit** 来达到同样的目的。

首先，将以下行添加到你的 `build.gradle` 的 **dependencies** 部分：

```java
testCompile group: 'org.testng', name: 'testng', version: '6.11'
```

现在，我们将创建我们的第一个测试。它应该位于 `/src/test/kotlin/<your_package>`。

所有集成测试的基本结构看起来像这样：

```java
class ServerVerticleTest {
    // Usually one instance of VertX is more than enough
    val vertx = Vertx.vertx()

    @BeforeClass
    fun setUp() {
        // You want to start your server once
        startServer()
    }

    @AfterClass
    fun tearDown() {
        // And you want to stop your server once
        vertx.close()
    }

    @Test
    fun testAlive() {
        // Here you assert something
    }

    // More tests come here
    ...
}
```

一个好技巧是使用 Kotlin 反引号符号来命名你的测试。

你可以像这样命名你的测试：

```java
@Test
fun testAlive() {
    ...
}
```

但更好的命名测试的方式是这样的：

```java
@Test
fun `Tests that alive works`() {
    ...
}
```

现在我们想要向我们的 `/alive` 端点发出实际的 HTTP 请求，例如，并检查响应代码。为此，我们将使用 Vert.x 网络客户端。

将其添加到你的 `build.gradle` 依赖项部分：

```java
compile group: 'io.vertx', name: 'vertx-web-client', version: $vertx_version
```

如果你打算只在测试中使用它，你可以指定 `testCompile` 而不是 `compile`。但 `WebClient` 非常有用，你最终可能还是会将其用在代码中。

# 辅助方法

在我们的测试中，我们将创建两个辅助函数，分别称为 `get()` 和 `post()`，它们将向我们的测试服务器发出 `GET` 和 `POST` 请求。

我们将从 `get()` 开始：

```java
private fun get(path: String): HttpResponse<Buffer> {
    val d1 = CompletableDeferred<HttpResponse<Buffer>>()

    val client = WebClient.create(vertx)
    client.get(8080, "localhost", path).send {
        d1.complete(it.result())
    }

    return runBlocking {
        d1.await()
    }
}
```

第二种方法 `post()` 将非常相似，但它还将有一个请求体参数：

```java

private fun post(path: String, body: String = ""): HttpResponse<Buffer> {
    val d1 = CompletableDeferred<HttpResponse<Buffer>>()

    val client = WebClient.create(vertx)
    client.post(8080, "localhost", path).sendBuffer(Buffer.buffer(body), { res ->
        d1.complete(res.result())
    })

    return runBlocking {
        d1.await()
    }
}
```

这两个函数都使用了 Kotlin 提供的默认参数值协程。

你应该编写自己的辅助函数或根据你的需求修改它们。

我们还需要另一个辅助函数 `startServer()`，我们已经在 `@BeforeClass` 中提到过它。它应该看起来像这样：

```java
private fun startServer() {
    val d1 = CompletableDeferred<String>()
    vertx.deployVerticle(ServerVerticle(), {
        d1.complete("OK")
    })
    runBlocking {
        println("Server started")
        d1.await()
    }
}
```

我们需要两个新的扩展函数来方便我们。这些函数将把服务器响应转换为 JSON：

```java
private fun <T> HttpResponse<T>.asJson(): JsonNode {
    return this.bodyAsBuffer().asJson()
}

private fun Buffer.asJson(): JsonNode {
    return ObjectMapper().readTree(this.toString())
}
```

现在我们已经准备好编写我们的第一个测试：

```java
@Test
fun `Tests that alive works`() {
    val response = get("/alive")
    assertEquals(response.statusCode(), 200)

    val body = response.asJson()
    assertEquals(body["alive"].booleanValue(), true)
}
```

运行 `./gradlew test` 以检查这个测试是否通过。

接下来，我们将编写另一个测试；这次是为猫的创建端点。

起初，它将失败：

```java
@Test
fun `Makes sure cat can be created`() {
   val response = post("/api/v1/cats",
                """
                {
                    "name": "Binky",
                    "age": 5
                }
                """)

   assertEquals(response.statusCode(), 201)
   val body = response.asJson()

   assertNotNull(body["id"])
   assertEquals(body["name"].textValue(), "Binky")
   assertEquals(body["age"].intValue(), 5)
}
```

注意，我们的服务器返回状态码 `501 Not Implemented`，并且没有返回 `cat` ID。

我们将在下一节讨论数据库持久性时修复这个问题。

# 与数据库一起工作

如果没有将我们的对象（即猫）保存到某种持久存储的能力，我们将无法取得更大的进展。

为了做到这一点，我们首先需要连接到数据库。

将以下两行添加到你的 `build.gradle` 依赖部分：

```java
compile group: 'org.postgresql', name: 'postgresql', version: '42.2.2'
compile group: 'io.vertx', name: 'vertx-jdbc-client', version: $vertx_version
```

第一行代码获取 `PostgreSQL` 驱动。第二行添加了 Vert.x JDBC 客户端，这使得 Vert.x 在拥有驱动程序的情况下可以连接到任何支持 JDBC 的数据库。

# 管理配置

现在我们想要将数据库配置存储在某个地方。对于本地开发，可能将配置硬编码是可行的。

当我们连接到数据库时，我们至少需要指定以下参数：

+   用户名

+   密码

+   主机

+   数据库名

我们应该在哪里存储它们？

当然，一个选项当然是将这些值硬编码。这对于本地环境来说是可以的，但当我们部署这个服务到其他地方时怎么办呢？

**你会去，我不能来！XDSpringBoot** 做的，或者我们可以尝试从环境变量中读取它们。无论如何，我们需要一个封装这个逻辑的对象，如下面的代码所示：

```java
object Config {
    object Db {
        val username = System.getenv("DATABASE_USERNAME") ?: "postgres"
        val password = System.getenv("DATABASE_PASSWORD") ?: ""
        val database = System.getenv("DATABASE_NAME") ?: "cats_db"
        val host = System.getenv("DATABASE_HOST") ?: ""

        override fun toString(): String {
            return mapOf("username" to username,
                    "password" to password,
                    "database" to database,
                    "host" to host).toString()
        }
    }

    override fun toString(): String {
        return mapOf(
                "Db" to Db
        ).toString()
    }
}
```

这当然只是你可以采取的一种方法。

现在，我们将使用此配置代码创建 `JDBCClient`：

```java
fun CoroutineVerticle.getDbClient(): JDBCClient {
    val postgreSQLClientConfig = JsonObject(
            "url" to "jdbc:postgresql://${Config.Db.host}:5432/${Config.Db.database}",
            "username" to Config.Db.username,
            "password" to Config.Db.password)
    return JDBCClient.createShared(vertx, postgreSQLClientConfig)
}
```

在这里，我们选择了一个扩展函数，它将在所有 `CoroutineVerticles` 上工作。

为了简化与 `JDBCClient` 一起工作，我们将向其中添加一个名为 `query()` 的方法：

```java
fun JDBCClient.query(q: String, vararg params: Any): Deferred<JsonObject> {
    val deferred = CompletableDeferred<JsonObject>()
    this.getConnection { conn ->
        conn.handle({
            result().queryWithParams(q, params.toJsonArray(), { res ->
                res.handle({
                    deferred.complete(res.result().toJson())
                }, {
                    deferred.completeExceptionally(res.cause())
                })
            })
        }, {
            deferred.completeExceptionally(conn.cause())
        })
    }

    return deferred
}
```

我们还会添加 `toJsonArray()` 方法，因为这是我们 `JDBCClient` 使用的：

```java
private fun <T> Array<T>.toJsonArray(): JsonArray {
    val json = JsonArray()

    for (e in this) {
        json.add(e)
    }

    return json
}
```

注意这里 Kotlin 泛型是如何被用来简化转换同时保持类型安全的。

我们还会添加一个 `handle()` 函数，它将为我们提供一个简单的 API 来处理异步错误：

```java
inline fun <T> AsyncResult<T>.handle(success: AsyncResult<T>.() -> Unit, failure: () -> Unit) {
    if (this.succeeded()) {
        success()
    }
    else {
        this.cause().printStackTrace()
        failure()
    }
}
```

为了确保一切正常工作，我们将在我们的`/alive`路由上添加一个检查：

```java
val router = Router.router(vertx)
val dbClient = getDbClient()
...
router.get("/alive").asyncHandler {
    val dbAlive = dbClient.query("select true as alive")
    val json = json {
        obj (
                "alive" to true,
                // This is JSON, but we can access it as an array
                "db" to dbAlive.await()["rows"]
        )
    }
    it.respond(json)
}
```

需要添加的行用粗体标出。

在添加这些行并打开[`localhost:8080/alive`](http://localhost:8080/alive)之后，你应该得到以下 JSON 代码：

```java
{"alive":true, "db":[{"alive":true}]}
```

# 管理数据库

当然，我们的测试没有通过。这是因为我们还没有创建我们的数据库。确保你在命令行中运行以下行：

```java
$ createdb cats_db
```

在我们确认数据库正在运行之后，让我们实现我们的第一个真实端点。

我们将保持我们的 SQL 与实际代码的清晰分离。将以下内容添加到你的`ServerVerticle`中：

```java
private val insert = """insert into cats (name, age)
            |values (?, ?::integer) RETURNING *""".trimMargin()
```

我们在这里使用多行字符串，通过`|`和`trimMargin()`来重新对齐它们。

现在用以下代码调用这个查询：

```java
...
val db = getDbClient()
router.post("/cats").asyncHandler { ctx ->
    db.queryWithParams(insert, ctx.bodyAsJson.toCat(), {
       it.handle({
          // We'll always have one result here, since it's our row
          ctx.respond(it.result().rows[0].toString(), 201)
       }, {
          ctx.respond(status=500)
       })
   })
}
```

注意，我们没有在任何地方打印错误信息。这是因为我们定义了`handle()`函数来处理这个任务。

我们还定义了自己的函数来解析请求体，将`JsonObject`转换为`JsonArray`，这是`JDBCClient`所期望的：

```java
private fun JsonObject.toCat() = JsonArray().apply {
   add(this@toCat.getString("name"))
   add(this@toCat.getInteger("age"))
}
```

注意，这里有两个不同的`this`版本。一个指的是`apply()`函数的内部作用域。另一个指的是`toCat()`函数的外部作用域。要引用外部作用域，我们使用`@scopeName`注解。

正如你所见，扩展函数是清理代码的极其强大的工具。

当你再次运行我们的测试时，你会注意到它仍然失败，但现在有一个不同的错误代码。这是因为我们还没有创建我们的表。让我们现在就创建它。有几种方法可以做到这一点，但最方便的方法是简单地运行以下命令：

```java
psql -c "create table cats (id bigserial primary key, name varchar(20), age integer)" cats_db
```

再次运行你的测试以确保它通过。

# EventBus

这是第二次我们遇到了相同的问题：我们的类越来越大，我们通常尽可能避免这种情况。

如果我们再次将创建猫的逻辑拆分到一个单独的文件中呢？让我们称它为`CatVerticle.kt`。

但是，我们需要一种方法让`ServerVerticle`与`CatVerticle`通信。在像**SpringBoot**这样的框架中，你会使用**依赖注入**来达到这个目的。但是对于响应式框架呢？

# 消费者

为了解决通信问题，Vert.x 使用**EventBus**。它是我们在第四章中讨论的**Observable**设计模式的实现，*熟悉行为模式*。任何 verticle 都可以通过事件总线发送消息，在这些两种模式之间进行选择：

+   `send()`将消息发送给单个订阅者

+   `publish()`将消息发送给所有订阅者

无论使用哪种方法发送消息，你都可以使用 EventBus 上的`consumer()`方法来订阅它：

```java
const val CATS = "cats:get"

class CatVerticle : CoroutineVerticle() {

    override suspend fun start() {
        val db = getDbClient()
        vertx.eventBus().consumer<JsonObject>(CATS) { req ->
            ...
        }
    }
}
```

类型指定了我们期望接收消息的对象。在这种情况下，它是`JsonObject`。常量`CATS`是我们订阅的键。它可以是任何字符串。通过使用命名空间，我们确保未来不会发生冲突。如果我们要在我们的收容所中添加狗，我们将使用另一个具有另一个命名空间的常量。例如：

```java
const val DOGS  = "dogs:get" // Just an example, don't copy it
```

现在我们添加以下两个查询，它们只是多行字符串常量：

```java
private const val QUERY_ALL = """select * from cats"""
class CatVerticle : CoroutineVerticle() {

    private val QUERY_WITH_ID = """select * from cats
                     where id = ?::integer""".trimIndent()
...
}
```

为什么我们在类内部放置一个，在类外部放置另一个？

`QUERY_ALL`是一个简短的查询，它适合一行。我们可以允许自己将其作为一个常量。另一方面，`QUERY_WITH_ID`是一个较长的查询，需要一些缩进。由于我们只在运行时移除缩进，所以我们不能将其作为一个常量。因此，我们使用成员值。在现实世界的项目中，你的大部分查询可能都需要是私有值。但了解两种方法之间的区别很重要。

我们用以下代码填充我们的消费者：

```java
...
try {
    val body = req.body()
    val id: Int? = body["id"]
    val result = if (id != null) {
        db.query(QUERY_WITH_ID, id)
    } else {
        db.query(QUERY_ALL)
    }
    launch {
        req.reply(result.await())
    }
}
catch (e: Exception) {
    req.fail(0, e.message)
}
...
```

如果请求中包含猫的 ID，我们就获取这只特定的猫。否则，我们获取所有可用的猫。

我们使用`launch()`是因为我们想要`await()`结果，并且我们没有返回值。

# 生产者

剩下的就是从`ServerVerticle`调用猫。为此，我们将在我们的`CoroutineVerticle`中添加另一个方法：

```java
fun <T> CoroutineVerticle.send(address: String,
                               message: T,
                               callback: (AsyncResult<Message<T>>) -> Unit) {
    this.vertx.eventBus().send(address, message, callback)
}
```

然后我们可以这样处理我们的请求：

```java
...
router.get("/cats").asyncHandler { ctx ->
    send(CATS, ctx.queryParams().toJson()) {
        it.handle({
            val responseBody = it.result().body()
            ctx.respond(responseBody.get<JsonArray>("rows").toString())
        }, {
            ctx.respond(status=500)
        })
    }
}
...
```

注意，我们正在重用之前定义的同一个常量，称为`CATS`。

这样，我们可以轻松地检查谁可以发送这个事件，谁消费它。如果成功，我们将返回一个 JSON。否则，我们将返回一个 HTTP 错误代码。

我们添加的另一个方法是`toJson()`在`MultiMap`上。`MultiMap`是一个包含我们的查询参数的对象：

```java
private fun MultiMap.toJson(): JsonObject {
    val json = JsonObject()

    for (k in this.names()) {
        json.put(k, this[k])
    }

    return json
}
```

为了确保一切按预期工作，让我们为我们的新端点创建两个额外的测试。

只别忘了在你的`Main.kt`文件和测试中的`startServer()`函数中添加以下行：

```java
...
vertx.deployVerticle(CatVerticle())
...
```

# 更多测试

现在添加以下基本测试：

```java
@Test
fun `Make sure that all cats are returned`() {
    val response = get("/api/v1/cats")
    assertEquals(response.statusCode(), 200)

    val body = response.asJson()

    assertTrue(body.size() > 0)
}
```

为了确保你理解所有这些是如何协同工作的，这里有一些你可能希望完成的额外任务：

1.  将添加新猫的逻辑移动到`CatVerticle`。

1.  实现获取单个猫的功能。注意代码与获取所有猫的代码非常相似？重构它以使用 Kotlin 的一个酷特性——局部函数，我们之前已经讨论过了。

1.  按照同样的原则实现删除和更新猫的功能。

# 摘要

本章汇总了我们关于 Kotlin 设计模式和习惯用法所学的所有内容，以生成一个可扩展的微服务。而且，多亏了 Vert.x，它也是反应式的，这使得它具有极高的可扩展性。它还进行了测试，正如任何现实世界的应用程序应该的那样。

在我们的应用程序中，类是根据领域而不是层来划分的，这与通常的 MVC 架构相反。Vert.x 中的最小工作单元被称为 verticle，verticles 通过 EventBus 进行通信。

我们的 API 遵循了所有 REST 的最佳实践：使用 HTTP 动词和有意义的路径来访问资源，以及消费和生成 JSON。

你可以将同样的原则应用到你要编写的任何其他实际应用中，我们确实希望你会选择 Vert.x 和 Kotlin 来实现这一点。
