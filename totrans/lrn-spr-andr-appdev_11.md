# 评估

# 第一章

1.  Spring 基于 Java **标准版**（**SE**）构建，但可以运行 Java **企业版**（**EE**）。

1.  几乎所有支持 Spring 的 Java 支持的 IDE，例如 NetBeans 和 Visual Studio。

1.  Tomcat 是一个 Web 服务器。

1.  你也可以使用 Jetty 和 Undertow 进行 Spring 开发。

1.  不，你可以使用 IntelliJ IDEA、Visual Studio—Xamarin、PhoneGap、Corona 和 CppDroid，但强烈推荐使用 Android Studio，它是开发 Android 应用的官方 IDE。

# 第二章

1.  Kotlin 是一种静态类型编程语言，它编译成与 Java 相同的字节码。

1.  Kotlin 支持面向对象编程的所有特性。

1.  Kotlin 支持许多函数式编程的特性。

1.  要定义一个只读变量，我们必须使用`val`关键字和`var`关键字来表示可变性。

1.  要定义一个函数，我们必须使用`fun`关键字。函数可以被定义为第一类公民、类成员或局部变量。

# 第三章

1.  Spring 作为最广泛使用的 Java EE 框架而脱颖而出。Spring 框架的核心思想是 *依赖注入* 和 *面向切面编程*。Spring 框架也可以用于普通的 Java 应用程序，通过实现依赖注入，实现各个部分的解耦，并且我们可以执行跨切面任务。

1.  依赖注入配置设计使我们能够移除硬编码的依赖，使我们的应用程序几乎解耦，可扩展和可行。我们可以执行一个依赖注入示例，将依赖目标从编译时移动到运行时。使用依赖注入的一些优点包括关注点分离、减少样板代码、可配置部分和简单的单元测试。

1.  **面向切面编程**（**AOP**）是一种编程范式，它通过将软件应用程序的关注点隔离来补充 **面向对象编程**（**OOP**），以增强模块化。

1.  **控制反转**（**IoC**）是用于实现对象依赖之间解耦的工具。为了实现解耦和运行时对象的动态绑定，对象定义了由其他构建代理对象注入的依赖项。Spring IoC 容器是注入依赖到对象并为其准备的程序。

1.  任何由 Spring IoC 容器引入的普通 Java 类都称为 Spring Bean。我们使用 Spring `ApplicationContext` 来获取 Spring Bean 实例。

1.  与 MVC 配置设计类似，控制器是处理所有客户请求并将它们发送到安排的资源以处理的类。在 Spring MVC 中，`org.springframework.web.servlet.DispatcherServlet` 是前端控制器类，它根据 Spring Bean 配置引入上下文。

1.  `DispatcherServlet`是 Spring MVC 应用程序中的前端控制器，它堆叠 Spring Bean 配置文件，声明所有已配置的 Bean。如果启用了注解，它还会过滤组件并设计任何带有`@Component`、`@Controller`、`@Repository`或`@Service`注解的 Bean。

1.  `ContextLoaderListener`是启动和关闭 Spring 的根`WebApplicationContext`的监听器。其重要角色是将`ApplicationContext`的生命周期绑定到`ServletContext`的生命周期，并自动生成`ApplicationContext`。我们可以使用它来定义可以在多个 Spring 上下文中使用的共享 Bean。

1.  模板代码是重复出现的代码，用于类似的目的。

# 第四章

1.  REST 代表表示性状态转移；它是构建网络编程接口的一个新概念。

    RESTFUL 指的是通过应用 REST 构建思想组成的网络服务，被称为 RESTful 服务。它围绕系统资产以及资产状态应该如何通过 HTTP 协议传输到各种不同语言的客户端展开。在 RESTful 网络服务中，可以使用诸如`GET`、`POST`、`PUT`和`DELETE`等 HTTP 方法来执行 CRUD 操作。

1.  创建 Web API 的架构风格如下：

    +   HTTP 用于客户端-服务器通信

    +   XML/JSON 作为格式化语言

    +   简单 URI 作为服务的地址

    +   无状态通信

1.  `SOAPUI`工具用于`SOAP WS`和 Firefox *海报*插件用于`RESTFUL`服务。

1.  基于 REST 架构的 Web 服务被称为 RESTful Web 服务。这些服务使用 HTTP 方法来执行 REST 架构的理念。RESTful Web 服务通常定义一个 URI（统一资源标识符），一个服务，提供资产描述，例如 JSON 和一组 HTTP 方法。

1.  URI 代表统一资源标识符。REST 架构中的每个资源都通过其 URI 来识别。URI 的目的是在提供 Web 服务的服务器上找到资源。

1.  它是一个 HTTP 成功代码：`OK`。

1.  它是一个 HTTP 客户端错误代码：`Not Found`。

# 第五章

1.  Spring 安全主要针对两个领域，这些领域是认证和授权。

1.  `SecurityContext`和`SecurityContextHolder`类是 Spring 安全的一部分。

1.  `DelegatingFilterProxy`类对于 Spring 安全是必需的，这个类来自`package org.springframework.web.filter`。

1.  是的。Spring 安全支持密码散列。

1.  授权代码、隐式、密码、客户端凭证、设备代码、刷新令牌。

# 第六章

1.  H2 是一个非常轻量级的开源 Java 数据库，它可以嵌入到 Java 应用程序中。它也可以在客户端-服务器模型上运行。

1.  *资源*意味着在*REST*架构中数据将如何表示。它允许客户端使用 HTTP 方法（例如`GET`，`POST`，`PUT`，`DELETE`等）读取、写入、修改和创建资源。

1.  CRUD 代表创建、读取、更新和删除。

1.  DAO 是数据持久化的抽象。*Repository*是对象集合的抽象。

1.  SQLite 使用动态类型。内容可以存储为**`INTEGER`**，`REAL`，`TEXT`，`BLOB`或`NULL`。

1.  SQLite 数据库的替代方案有 OrmLite，Couchbase Lite 和 Snappy DB。

1.  标准的 SQLite 命令有 `SELECT`, `CREATE`, `INSERT`, `UPDATE`, `DROP`, 和 `DELETE`。

1.  SQLite 有一些缺点。如下所示：

    +   它用于处理低到中等的 HTTP 请求流量。

    +   在大多数情况下，SQLite 的大小限制为 2 GB。

# 第七章

1.  调用栈是线程或协程分配的内存的一部分，包含在线程或协程上下文中调用的函数的堆栈和局部变量。

1.  线程池是一种使用一组等待从队列中获取工作的线程的模式的抽象。

1.  回调是一种用于传递异步操作结果的模式。

1.  协程是轻量级线程，因为创建协程不需要像创建线程那样多的资源。

# 第八章

1.  响应式编程是一种处理异步事件的途径。

1.  Mono 是一个可以发出零个或一个事件的发布者。

1.  Observable 是 RxJava 中的一个类，它发出一系列值。

1.  调度器是线程池的抽象。

# 第九章

1.  EER 代表**E**nhanced **E**ntity-**R**elationship。它是一个高级模型，是对 ER 模型修改和传递版本的模型。它有助于更精确地创建数据库模式。

1.  CRUD 代表创建、读取、更新和删除。

1.  Postgresql，MySQL，MongoDB 等等。

1.  Postman。Insomnia 易于使用且功能强大。

1.  最小 API 为 21；目标和最大值将是最新 API（当前最新 API 是 28）。

1.  MVC, MVP 和 MVVM。

1.  Android Studio, Genymotion（免费/痛苦），Remix OS 和 Nox Player。

1.  ANR 是**A**pplication **N**ot **R**esponding 的缩写。当应用程序因后台的一些错误而卡在 UI 上时，就会发生这种情况。

1.  Sketch 是最好的，Adobe XD 也是设计原型的一个强大应用。

# 第十章

1.  基于文本的命令行，以及基于 AWT 和 Swing 的图形测试机制。

1.  Google。

1.  在基于 Java 的应用程序中。

1.  单元测试、集成测试和 UI 测试。

1.  70%小（单元测试），20%中等（集成测试），和 10%大（UI 测试）。

1.  处理它的最佳方式是使用模拟器。
