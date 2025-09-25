# 使用 Spring Security 保护应用程序

安全性是企业、电子商务和银行项目中的首要任务之一。这些项目需要创建一个安全系统，因为它们交换数百万美元并存储组织的受保护资源。

Spring Security 是庞大的 Spring 框架系列的一个子任务。它已被升级以与 Spring MVC Web 应用程序框架一起使用，但同样也可以与 Java servlets 一起使用。这支持与一系列其他技术的认证集成，例如**轻量级目录访问协议**（**LDAP**）、**Java 认证和授权服务**（**JAAS**）和 OpenID。它被开发为一个针对基于 Java 的企业环境的完整安全解决方案。

在本章中，我们将了解 Spring Security 及其模块，并学习如何在基于 Spring 的项目中实现安全性。本章将涵盖以下主题：

+   Spring Security 架构

+   Spring Security 的优势

+   Spring Security 特性

+   Spring Security 模块

+   实施 Spring Security

+   使用 Spring Security 基本认证保护 REST

+   使用 Spring Security OAuth2 保护 REST

# 技术要求

您需要添加以下依赖项以启用和使用 Spring Security 的功能。以下是需要添加到 Spring 项目的`pom.xml`文件中的依赖项：

```kt
<dependency>
   <groupId>org.springframework.security</groupId>
   <artifactId>Spring_Security_SUB_Module_Name</artifactId>
   <version>CURRENT_RELEASE_VERSION</version>
</dependency>

<dependency>
   <groupId>org.springframework.security</groupId>
   <artifactId>spring-security-core</artifactId>
   <version>5.1.1.RELEASE</version>
</dependency>
```

您可以在 GitHub 上找到本章的所有示例：[`github.com/PacktPublishing/Learn-Spring-for-Android-Application-Development/tree/master/Chapter05`](https://github.com/PacktPublishing/Learn-Spring-for-Android-Application-Development/tree/master/Chapter05)。

# Spring Security 架构

Spring Security 是基于 J2EE 的企业生产的安全服务解决方案。它通过使用其特定的依赖注入原则，帮助以更快、更简单的方式开发安全应用程序。为了开发基于 J2EE 的企业应用程序，Spring Security 是一个强大且灵活的认证和授权框架。认证是检查过程或客户端身份的过程。另一方面，授权意味着检查客户端在应用程序中执行活动的权限。

# 认证

认证是根据用户的用户名和密码识别用户或客户端的过程。它帮助用户根据其身份获取受保护系统对象的访问权限。对于认证过程，Spring Security 为我们提供了`AuthenticationManager`接口。此接口只有一个功能，名为`validate()`。

以下代码片段是`AuthenticationManager`接口的一个示例：

```kt
interface AuthenticationManager {
 @Throws(AuthenticationException::class)
 fun authenticate(authentication: Authentication): Authentication
} 
```

在此`AuthenticationManager`接口的`authenticate()`中完成了三个任务：

+   如果其能力可以检查输入代表一个有效的主体，则`authenticate()`返回`Authentication`。前面提到的代码通常返回`authenticated=true`。

+   如果能力发现输入不符合有效规则，它将抛出`AuthenticationException`。

+   如果能力无法选择任何内容，它将返回`null`。

`AuthenticationException`是一个运行时异常。应用程序以传统方式处理这个异常。

`ProviderManager`常用于实现`AuthenticationManager`，代表一系列`AuthenticationProvider`对象。如果没有可访问的父级，它将抛出`AuthenticationException`。

`AuthenticationProvider`类似于`AuthenticationManager`，但有一个额外的功能。这个额外的功能使客户端能够在支持给定的`Authentication`类型时进行查询。

这里是`AuthenticationProvider`接口的一些代码：

```kt
interface AuthenticationProvider {
 @Throws(AuthenticationException::class)
    fun authenticate(authentication:Authentication):Authentication
    fun supports(authentication: Class<*>): Boolean
}
```

这个接口有两个功能——`authenticate()`返回用户的认证详情，而`supports()`返回一个`Boolean`，如果认证和给定的用户名-密码对匹配，则返回`true`，否则返回`false`。

这里是使用`ProviderManager`的`AuthenticationManager`层次结构的图示：

![图片](img/b5fd0389-0cd9-48a8-a03f-6954119392f0.png)

根据这个图示，在一个应用中，`ProviderManager`可能有一组其他的`ProviderManager`实例，但第一个将作为父级。每个`ProviderManager`可能拥有多个`AuthenticationManager`。例如，如果所有网络资源都在相同的路径下，每个组都将拥有自己的专用`AuthenticationManager`。然而，将只有一个共同的父级，它将作为全局资源，并由这些专用`AuthenticationManager`实例共享。现在，让我们看看如何修改认证管理器。

# 修改认证管理器

Spring Security 提供了一些配置助手来设置应用中的认证管理器功能。这将有助于快速获取功能。`AuthenticationManagerBuilder`有助于修改认证管理器。

这里是一个如何在`ApplicationSecurity.kt`类中实现`AuthenticationManagerBuilder`的示例：

```kt
class ApplicationSecurity: WebSecurityConfigurerAdapter() {
    @Autowired
 fun initialize(builder: AuthenticationManagerBuilder, dataSource: DataSource){
builder.jdbcAuthentication().dataSource(dataSource).withUser("Sunnat629").password("packtPub").roles("USER")
 }
}
```

这里，我们为这个应用中的`USER`角色提供了一个用户名，`sunnat629`，和一个密码，`packtPub`。

Spring Boot 附带了一个默认的全局`AuthenticationManager`，它足够安全。你可以通过提供自己的`AuthenticationManager` bean 来替换它。

# 授权

授权是接受或拒绝访问网络资源的过程。它将授予访问使用资源中的数据。在`Authentication`过程之后，`Authorization`过程开始。`Authorization`用于处理访问控制。`AccessDecisionManager`是这个过程中的核心实体之一。

# 网络安全

Spring 安全性的 servlet 通道提供 Web 安全性。使用`@WebSecurityConfigurer`注解启用 Web 安全性，并在 Web 安全性类中覆盖`WebSecurityConfigurerAdapter`。

# 方法安全

这是一个由 Spring Security 提供的安全方法模块。我们可以在特定功能中提供一个角色，以便基于角色的用户可以访问该功能。

以下注解用于启用此功能：

```kt
 @EnableGlobalMethodSecurity(securedEnabled = true)
```

下面是一个如何在`SpringSecurityApplication.kt`类中启用方法安全的示例，这是我们的演示项目的主体应用程序类：

```kt
@SpringBootApplication
@EnableGlobalMethodSecurity(securedEnabled = true)
class SpringSecurityApplication{

    fun main(args: Array<String>) {
        runApplication<SpringSecurityApplication>(*args)
    }
}

```

现在，您可以创建方法资源，如下面的代码所示：

```kt
@Secured class CustomService{
    @Secured
    fun secure(): String{
 return "The is Secured..."
    }
}
```

在这里，我们使用`@Secured`注解创建了一个名为`CustomService`的安全类，然后创建了一个将返回 Spring 的安全函数。`@Secured`注解用于指定函数上的角色列表。

# Spring Security 的优势

Spring 安全框架提供了以下优势：

+   Spring Security 是一个开源的安全框架

+   它支持认证和授权

+   它保护常见任务

+   它可以与 Spring MVC 和 Servlet API 集成

+   它支持 Java 和 Kotlin 配置支持

+   开发和单元测试应用程序很容易

+   Spring 依赖注入和 AOP 可以轻松使用

+   它开发松散耦合的应用程序

# Spring Security 特性

Spring Security 实现了许多功能。

在这里，我们解释了一些常见和主要的功能：

+   **LDAP**: LDAP 是一个开放的应用协议。它通过互联网维护和访问分布式目录数据服务。

+   **OAuth 2.0 登录**: 此组件使得客户端能够通过利用他们在 Google、Facebook、Twitter 或 GitHub 上现有的账户来登录应用程序。

+   **基本访问认证**: 当客户端通过网络请求时，此方法提供用户名和密码。

+   **摘要访问认证**: 这要求程序在通过系统发送个人信息之前确认客户端的身份。

+   **Web 表单认证**: 在此认证系统中，Web 表单从 Web 浏览器收集和验证用户凭据。

+   **授权**: Spring Security 提供此功能，在客户端获取资源之前对其进行批准。

+   **HTTP 授权**: 这指的是对 Web 请求 URL 的 HTTP 授权。它使用 Apache Ant 路径或正则表达式。

+   **响应式支持**: 这提供了响应式编程和 Web 运行时支持。

+   **现代化的密码编码**: 从 Spring Security 5.0 引入了一个新的密码编码器，名为`DelegatingPasswordEncoder`。

+   **单点登录**: 此功能允许客户端使用单个账户访问多个应用程序。

+   **JAAS**: JAAS 是一个 Java 中实现的插件式认证模块。

+   **记住我**：Spring Security 利用 HTTP cookies，记住客户端的登录 ID 和密码，以便在客户端注销之前避免再次登录。

+   **软件本地化**：您可以用任何人类语言创建应用程序的用户界面。

# Spring Security 模块

在 Spring Security 3.0 中，Spring Security 模块已被隔离成几个子模块。然而，在当前版本中，有 12 个子模块。为了支持这些模块，代码被细分为独立的容器。这些容器目前是分离的，每个子模块都有不同的有用领域和第三方依赖。

这里是子模块 jar 列表：

+   `spring-security-core.jar`

+   `spring-security-remoting.jar`

+   `spring-security-web.jar`

+   `spring-security-config.jar`

+   `spring-security-ldap.jar`

+   `spring-security-oauth2-core.jar`

+   `spring-security-oauth2-client.jar`

+   `spring-security-oauth2-jose.jar`

+   `spring-security-acl.jar`

+   `spring-security-cas.jar`

+   `spring-security-openid.jar`

+   `spring-security-test.jar`

Spring Security Core 子模块是其他 Security 子模块（如`web`、`config`和`oauth2`）的基础模块。

# 实现 Spring Security

如果您想在项目中使用 Spring Security，您需要在 Maven 和 Gradle 中实现您想要使用的 Spring Security 依赖项。

让我们看看如何在 Maven 和 Gradle 中实现 Spring Security 依赖项。

# Maven

要实现安全依赖项，您需要在`pom.xml`中实现`spring-security-core`：

```kt
<dependency>
 <groupId>org.springframework.security</groupId>
 <artifactId>Spring_Security_SUB_Module_Name</artifactId>
 <version>CURRENT_RELEASE_VERSION</version>
</dependency>

<!--here is an example of a security core sub-modules-->
<dependency>
 <groupId>org.springframework.security</groupId>
 <artifactId>spring-security-core</artifactId>
 <version>5.1.1.RELEASE</version>
</dependency>

```

# Gradle

要实现依赖项，您需要在`build.gradle`中放入以下代码：

```kt
dependencies {
    implementation 'org.springframework.security:[Spring_Security_SUB_Module_Name]:CURRENT_RELEASE_VERSION'
}

// here is an example of a security core sub-modules
dependencies {
 implementation 'org.springframework.security:[spring-security-core]:5.1.1.RELEASE'
}

```

# 使用基本认证保护 REST

在这个主题中，我们将通过一个简单的项目学习基本认证。在这里，我们将创建一个示例，您将构建一个安全的 REST API。我们将创建一个项目并实现基本认证。这将帮助我们避免基本配置和完整的 Kotlin 配置时间。对于这个项目，您必须输入用户名和密码才能访问内容。这个项目没有 UI，因此您需要使用 HTTP 客户端来测试项目。在这里，我们使用 Insomnia ([`insomnia.rest/download/`](https://insomnia.rest/download/))。您可以从这里测试您的项目并访问内容。

在开始我们的项目之前，我们将了解基本认证及其用途。

# 什么是基本认证？

**基本认证**是最简单的认证方案，它是 HTTP 协议内建的。要使用它，客户端需要发送包含认证头部的 HTTP 请求，该头部包含单词*Basic*后跟一个空格。然后，给定的用户名和密码字符串将被视为`username/password`并编码为 Base64。例如，如果用户名和密码是`Sunnat629`和`pa$$worD`，这些将被转换为 Base64 编码，将变为`U3VubmF0NjI5L3BhcyQkd29yRA==`作为授权。最后，客户端将发送`Authorization: Basic U3VubmF0NjI5L3BhcyQkd29yRA==`到服务器。

Base64 可以轻松解码。这既不是加密也不是散列。如果你想使用基本认证，我们强烈建议你与其他安全工具一起使用，例如 HTTPS/SSL。

# 创建项目

我们将创建一个小项目，在这个项目中我们将实现基本认证安全来保护数据。用户需要通过我们的安全系统才能访问数据。让我们按照以下步骤创建项目：

1.  要创建项目，请访问[`start.spring.io/`](https://start.spring.io/)并修改给定的字段以满足你的需求。你可以在以下屏幕截图中查看我们的项目信息：

![图片](img/4047a3e3-1931-49b8-82b8-4674ba208e7a.png)

在这里，我们使用`Maven Project`，选择语言为`Kotlin`，Spring Boot 版本为`2.1.1 (SNAPSHOT)`。

我们已添加了`Security`、`Web`和`DevTools`依赖项。你可以在`pom.xml`中查看列表。

1.  当你选择“生成项目”时，你会以 ZIP 文件的形式获得项目。解压并使用你的 IDE 打开此项目。

1.  下载和更新 Maven 依赖项需要一点时间。以下是你的项目内容的截图：

![图片](img/0901227f-9f7f-40af-930f-25108808826a.png)

如果你需要添加新的依赖项或更新版本，请修改`pom.xml`。如果你想创建`kotlin`文件，你需要在`src->main->kotlin->{Package_NAME}`文件夹下创建文件。

# 配置 pom.xml

在这个`pom.xml`中，你将获得有关项目的所有信息。在这里，你可以插入新的依赖项，更新版本等。以下是示例`pom.xml`（完整代码在 GitHub 上，[`github.com/PacktPublishing/Learn-Spring-for-Android-Application-Development/tree/master/Chapter05`](https://github.com/PacktPublishing/Learn-Spring-for-Android-Application-Development/tree/master/Chapter05)）：

```kt
<groupId>com.packtpub.sunnat629</groupId> <artifactId>ssbasicauth</artifactId>
<version>0.0.1-SNAPSHOT</version>
<packaging>jar</packaging>

<name>Spring Security Basic Authentication</name>
<description>A sample project of Spring Security Basic Authentication</description>

----
----

<properties>
   <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
   <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
 <java.version>1.8</java.version>
   <kotlin.version>1.3.0</kotlin.version>
</properties>

<dependencies>
   <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-security</artifactId>
   </dependency>
   <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
   </dependency>
   <dependency>
      <groupId>com.fasterxml.jackson.module</groupId>
      <artifactId>jackson-module-kotlin</artifactId>
   </dependency>
 <dependency>
      <groupId>org.jetbrains.kotlin</groupId>
      <artifactId>kotlin-stdlib-jdk8</artifactId> </dependency>
   <dependency>
      <groupId>org.jetbrains.kotlin</groupId>
      <artifactId>kotlin-reflect</artifactId>
   </dependency>

   <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-test</artifactId>
      <scope>test</scope>
   </dependency>
   <dependency>
      <groupId>org.springframework.security</groupId>
      <artifactId>spring-security-test</artifactId>
      <scope>test</scope>
   </dependency>
</dependencies>

---
---
---
```

# 配置 Spring Bean

要配置 Spring Bean，我们将创建一个名为`SSBasicAuthApplication.kt`的应用程序文件，并使用 Java 配置，它配置 Spring Security 而不需要编写任何 XML 代码。

这是应用程序文件（`SSBasicAuthApplication.kt`）的简单代码：

```kt
@ComponentScan(basePackages = ["com.packtpub.sunnat629.ssbasicauth"])
@SpringBootApplication
class SSBasicAuthApplication: SpringBootServletInitializer()

fun main(args: Array<String>) {
 runApplication<SSBasicAuthApplication>(*args)
}
```

在这里，我们扩展了`SpringBootServletInitializer`。这将从传统的`WAR`存档中运行`SpringApplication`。此类负责将应用程序上下文中的`Servlet`、`Filter`和`ServletContextInitializer`豆绑定到服务器。

`@SpringBootApplication`是一个便利注解，相当于为`SSBasicAuthApplication`类声明`@Configuration`和`@EnableAutoConfiguration`。

在`@ComponentScan`注解中提及包名或包名集合，以指定基本包。这与`@Configuration`注解一起使用，以告诉 Spring 包扫描注解组件。

# Spring Security 配置

要为我们的项目添加 Spring Security 配置，请在应用程序包中使用以下代码创建一个名为**`SSConfig.kt`**的文件：

```kt
@Configuration @EnableWebSecurity
class SSConfig: WebSecurityConfigurerAdapter() {

    @Autowired
    private val authEntryPoint: AuthenticationEntryPoint? = null

    @Throws(Exception::class)
    override fun configure(http: HttpSecurity) {
        http.csrf().disable().authorizeRequests()
                .anyRequest().authenticated()
                .and().httpBasic()
                .authenticationEntryPoint(authEntryPoint)
    }

    @Autowired
    @Throws(Exception::class)
    fun configureGlobal(auth: AuthenticationManagerBuilder) {
        auth.inMemoryAuthentication()
                .withUser("sunnat629")
                .password(PasswordEncoderFactories.createDelegatingPasswordEncoder()
                        .encode("password"))
                .roles("USER")
    }
}
```

我们使用`@Configuration`注解了此类，这有助于 Spring 基于注解的配置。`@EnableWebSecurity`将启用 Spring Security 的 Web 安全支持。

我们扩展了`WebSecurityConfigurerAdapter`，这将使我们能够覆盖和自定义 Spring 功能。我们使用 HTTP 基本认证，并且所有请求都将使用此方法进行认证。

如果认证失败，我们需要处理这种情况。为此，创建一个名为`AuthenticationEntryPoint.kt`的认证入口点类并将其`autowire`。它将帮助在失败的情况下再次尝试此过程。

我们使用用户名`sunnat629`、密码`password`和`USER`角色。

# 配置认证入口点

配置认证入口点以处理失败的认证。当凭证未被授权时，此类主要负责发送响应。

下面是名为**`AuthenticationEntryPoint.kt`**的认证入口点类的代码：

```kt
@Component
class AuthenticationEntryPoint : BasicAuthenticationEntryPoint() {

    @Throws(IOException::class, ServletException::class)
    override fun commence(request: HttpServletRequest,
                          response: HttpServletResponse,
                          authEx: AuthenticationException) {
        response.addHeader("WWW-Authenticate", "Basic realm=$realmName")
        response.status = HttpServletResponse.SC_UNAUTHORIZED
        val writer = response.writer
        writer.println("HTTP Status 401 - " + authEx.message)
    }

    @Throws(Exception::class)
    override fun afterPropertiesSet() {
        realmName = "packtpub ssbasicauth"
        super.afterPropertiesSet()
    }
}
```

在这里，我们扩展了`BasicAuthenticationEntryPoint()`。这将向客户端返回`401 Unauthorized`响应的完整描述。

`401 Unauthorized Error`是一个 HTTP 响应状态码。这表示客户端发送的请求无法被认证。

# 配置 Spring WebApplicationInitializer

Spring `WebApplicationInitializer`使用 Servlet 3.0+实现来程序化配置`ServletContext`。

下面是`WebApplicationInitializer`类的示例代码，称为**`MyApplicationInitializer.kt`**：

```kt
class MyApplicationInitializer: WebApplicationInitializer {

    @Throws(ServletException::class)
    override fun onStartup(container: ServletContext) {

        val ctx = AnnotationConfigWebApplicationContext()
        ctx.servletContext = container

        val servlet = container.addServlet("dispatcher", DispatcherServlet(ctx))
        servlet.setLoadOnStartup(1)
        servlet.addMapping("/")
    }
}
```

本课程将帮助您使用`start`映射项目 URL 路径`"\"`。由于我们使用基于代码的注解代替 XML 配置，因此我们使用`AnnotationConfigWebApplicationContext`。

然后，我们创建并注册了分发器 servlet。

# 创建用户模型

通过访问简单的 REST API，我们创建了一个用户模型类。当客户端输入正确的用户名和密码时，这将返回一些用户详情的简单 JSON 输出。

下面是`Users.kt`的代码：

```kt
class Users(val id: String,
            val name: String,
            val email: String,
            val contactNumber: String)
```

在这个用户模型中，我们有一个`id`，一个`name`，一个`email`和一个`contactNumber`。我们将创建一个受我们安全系统保护的 JSON 类型 REST API。

# 创建控制器

控制器类将映射项目的 URL 路径。在这里，我们将使用`GET`或`POST` `HTTP`请求函数来创建 REST API。以下是项目控制器的一个示例代码，命名为**`UserController.kt`**：

```kt
@RestController
class UserController {

    @GetMapping(path = ["/users"])
    fun userList(): ResponseEntity<List<Users>>{
        return ResponseEntity(getUsers(), HttpStatus.OK)
    }

    private fun getUsers(): List<Users> {
        val user = Users("1","Sunnat", "sunnat123@gmail.com", "0123456789")
        val user1 = Users("2","Chaity", "chaity123@gmail.com", "1234567890")
        val user2 = Users("3","Jisan", "jisan123@gmail.com", "9876543210")
        val user3 = Users("4","Mirza", "mirza123@gmail.com", "5412309876")
        val user4 = Users("5","Hasib", "hasib123@gmail.com", "5678901234")

        return Arrays.asList<Users>(user, user1, user2, user3, user4)
    }
}
```

在这里，我们使用用户模型创建了一个包含五人的用户列表。在控制器中，`@RequestMapping`注解应用于类级别和/或方法级别。这会将特定的请求路径映射到控制器。使用`@GetMapping(path = ["/users"])`注解，如果 HTTP 状态是`OK`，客户端将发送`GET`请求以获取用户的列表。

# 使用 HTTP 客户端

要查看输出，请打开你的第三方 HTTP 客户端工具。在这里，我们使用 Insomnia。

运行项目后，打开 Insomnia。

请按照以下步骤测试项目：

1.  创建一个带有名称的新请求。

1.  在 GET 输入框中，输入`http://localhost:8080/user` URL。在这里，`localhost:8080`是根 URL，因为我们使用`@RequestMapping(path = ["/user"], method = [RequestMethod.GET])`在控制器类中，项目将在`http://localhost:8080/user`路径下运行。

1.  如果你点击发送按钮，你会看到一个`HTTP Status 401 - Bad credentials`错误，如下面的截图所示：

![](img/d117a43c-a6bc-4e69-ab94-3b5758114780.png)

虽然你使用的是基本认证，但你必须输入**用户名**和**密码**才能完成此请求。你需要点击 Auth（第二个标签）并选择`Basic`认证；你可以在那里输入**用户名**和**密码**。如果你输入随机的用户名和密码，你也会得到相同的错误。

在输入正确的**用户名**和**密码**后，你将得到以 JSON 格式输出的用户列表，如下面的截图所示：

![](img/7f32d0f3-0f92-4f2a-82af-f4b5e82cc79e.png)

你也可以在浏览器中测试。在那里，你会被要求输入**用户名**和**密码**。

你也可以使用浏览器查看 REST API：

![](img/4baeb0f8-550c-40e2-ba42-9dd827b76de6.png)

在输入用户名和密码后，我们可以看到用户列表：

![](img/944f295e-d503-4c56-a601-0776e8922b25.png)

你已经使用 Spring Security 基本认证创建了一个非常简单的项目。我们希望从现在开始，你可以借助 Spring Security 编写自己的基于认证的项目。

# 创建 Android 应用程序

是时候创建一个简单的 Android 应用程序作为客户端，从我们的基本认证服务器检索 REST API 了。首先，我们需要在 Android Studio 中创建一个应用程序并填写你的项目名称和公司域名。别忘了勾选`Include Kotlin support`。以下是创建应用程序项目窗口的截图：

![](img/aabf30ec-8bc8-41b7-ae73-8591beadad9f.png)

从手机和平板选项中选择最低 API 版本。对于这个项目，不需要添加其他选项。点击下一步后，您可以在 **Add an Activity to Mobile** 窗口中选择 Empty Activity。在重命名 **Activity Name** 和 **layout** 后，点击完成。构建 gradle 后，您就可以开始创建 Android 应用了。

现在，让我们看看如何在 Gradle 中实现项目的依赖项。

# Gradle 信息

在 Gradle 文件中，提及 Kotlin 依赖项和应用程序 Gradle 版本。以下是我的 Android Studio 的 Gradle 文件详情：

```kt
buildscript {
 ext.kotlin_version = '1.3.10'    repositories {
        google()
        jcenter()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:3.2.1'
        classpath "org.jetbrains.kotlin:kotlin-gradle-plugin:$kotlin_version"

        // NOTE: Do not place your application dependencies here; they belong
        // in the individual module build.gradle files
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

在这里，我们的 Gradle 版本是 `3.2.1`，Kotlin 版本是 `1.3.10`

# Gradle 依赖项

在这个 Gradle 文件中，我们将实现 Retrofit 的依赖项，这将帮助我们从前一个项目中获取 JSON 类型的 REST API。以下是所有依赖项：

```kt
implementation 'com.android.support:appcompat-v7:27.1.1'
implementation 'com.android.support.constraint:constraint-layout:1.1.3'

implementation 'com.google.code.gson:gson:2.8.5'

implementation 'com.squareup.retrofit2:retrofit:2.4.0'
implementation 'com.squareup.retrofit2:converter-gson:2.4.0'
implementation 'com.squareup.retrofit2:retrofit-converters:2.5.0'
implementation 'com.squareup.retrofit2:retrofit-adapters:2.5.0'
implementation 'com.squareup.okhttp3:logging-interceptor:3.12.0'
```

# 创建用户模型

我们将获取基于基本身份验证的 Spring 项目的 REST API，该项目是使用基本身份验证创建的。尽管 REST API 有四个实体（`id`、`name`、`email` 和 `contactNumber`），但我们将基于这个 REST API 创建一个模型。

这是 REST API 的输出，我们可以看到五个用户的详细信息：

![](img/26a0fb10-ac11-46cd-b71f-a3f08f20d12e.png)

根据 API，我们将为客户端创建一个用户模型。这是名为 **`UserModel`** 的模型类：**

```kt
class UserModel (val id: String
                 val name: String,
                 val contactNumber: String,
                 val id: String,
                 val email: String)
```

现在，我们需要创建一个接口，它将包含 HTTP 请求函数。在这个项目中，我们只会使用一个 `GET` 函数来检索所有用户的详细信息。在这里，我们使用 `GET` Retrofit 注解来编码有关参数和请求函数的详细信息。

这是 `UserService` 接口的代码：

```kt
interface UserService {
 @GET("/user")
    fun getUserList(): Call<List<UserModel>>
}
```

我们将搜索 `/user` 端点，这将提供一个用户模型列表。

# 实现用户服务

Retrofit 客户端调用 Gerrit API 并通过将调用结果打印到控制台来处理结果。

创建一个类，我们将构建 Retrofit 客户端，这将调用 API 并处理结果。这将负责使用 `Retrofit.builder` 类控制所有任务，并使用给定 URL 的基础进行配置。

这是 `UserServiceImpl.kt` 的代码：

```kt
class UserServiceImpl{
   fun getGithubServiceImpl(username:String, password:String): UserService {
        val retrofit = Retrofit.Builder()
                .client(getOkhttpClient(username, password))
                .baseUrl(YOUR_SERVER_DOMAIN)
                .addConverterFactory(GsonConverterFactory.create())
                .build()
        return retrofit.create(UserService::class.java)
    }

    private fun getOkhttpClient(username:String, password:String): OkHttpClient{
        return OkHttpClient.Builder()
                .addInterceptor(BasicAuthInterceptor(username, password))
                .build()
    }
}
```

根据这段代码，我们使用 `username` 和 `password` 设置了 `.client()`。然后我们实现了 `YOUR_SERVER_DOMAIN`（假设 Rest API 服务器的 URL 为 **`http://localhost:8080`**），`baseUrl()`，并且我们使用了 `OkHttpClient` 作为客户端。

# 使用 OkHttp 拦截器进行身份验证

虽然我们使用的是基于基本身份验证的安全机制，但我们需要一个 `username` 和 `password` 来授权访问这个 REST API。在这里，我们使用 `OkHttp` 拦截器进行身份验证。这将帮助您发送请求并获得访问资源的认证权限。

在这里，我们在 `OkHttpClient.Builder()` 中调用了 `BasicAuthInterceptor` 类：

```kt
 private fun getOkhttpClient(username:String, password:String): OkHttpClient{
        return OkHttpClient.Builder()
                .addInterceptor(BasicAuthInterceptor(username, password))
                .build()
    }
```

这是 `BasicAuthInterceptor.kt` 的类：

```kt
class BasicAuthInterceptor(user: String, password: String) : Interceptor {

    private val credentials: String = Credentials.basic(user, password)

    @Throws(IOException::class)
    override fun intercept(chain: Interceptor.Chain): Response {
        val request = chain.request()
        val authenticatedRequest = request.newBuilder()
                .header("Authorization", credentials).build()
        return chain.proceed(authenticatedRequest)
    }
}
```

在这个类中，只添加了凭证作为用户详情。在这里，客户端将使用`username`和`password`凭证发出请求。在每次请求期间，这个拦截器在执行之前起作用并修改请求头。因此，你不需要在 API 函数中添加`@HEADER("Authorization")`。

# 调用回调

在这里，我们从`MainActivity`调用`CallBack<>`。这个回调响应来自服务器或离线请求。这意味着在稍后的时间点返回长时间运行函数的结果。

检查`MainActivity.kt`代码以使用`CallBack`函数并处理结果：

```kt
class MainActivity : AppCompatActivity() {

    var username: String = "sunnat629"
    var password: String = "password"

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        val githubService: UserService = UserServiceImpl().getGithubServiceImpl(username,password)

        val call: Call<List<UserModel>> = githubService.getUserList()
        call.enqueue(object: Callback<List<UserModel>> {
            override fun onFailure(call: Call<List<UserModel>>, t: Throwable) {
                Log.wtf("PACKTPUB", t.message)
            }

            override fun onResponse(call: Call<List<UserModel>>, response: Response<List<UserModel>>) {
                val adapter = UserListAdapter(this@MainActivity, response.body())
                displayList.adapter = adapter
            }
        })
    }
}
```

让我们如下讨论前面的代码：

1.  首先，我们需要初始化`UserServiceImpl().getGithubServiceImpl(username,password)`，这样我们就可以从`UserService`调用`getUserList()`。

1.  然后我们将调用`enqueue(retrofit2.Callback<T>)`，这将异步执行，发送请求并获取响应。

1.  `enqueue()`有两个功能：`onResponse()`和`onFailure()`。如果有任何与服务器相关的错误，它将调用`onFailure()`，如果它收到响应和资源，它将调用`onResponse()`。我们还可以使用`onResponse()`函数的资源。

在这里，我们将获取`UserModel`列表的响应。我们可以在应用程序 UI 中显示这个列表。

# 创建 UI

在创建的`main_activity`布局中，我们将显示用户详情的列表，其中显示用户的姓名、电子邮件 ID 和联系电话——我们将使用`ListView`。

这是`MainActivity`类的`mainActivity`布局的代码：

```kt
<?xml version="1.0" encoding="utf-8"?>
<android.support.constraint.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <TextView
        android:id="@+id/textView"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="@string/user_title"
        app:layout_constraintEnd_toEndOf="parent"
        android:textStyle="bold"
        android:padding="5dp"
        android:gravity="center_horizontal"
        android:textAppearance="?android:textAppearanceLarge"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

    <ListView
        android:id="@+id/displayList"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginStart="8dp"
        android:layout_marginLeft="8dp"
        android:layout_marginTop="8dp"
        android:layout_marginEnd="8dp"
        android:layout_marginRight="8dp"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@+id/textView" />

</android.support.constraint.ConstraintLayout>
```

在这个布局中，我们有一个`TextView`和一个`ListView`。

我们将在`MainActivity`的`onResponse()`函数中使用这个`ListView`。

我们将获取列表并创建一个自定义适配器来显示用户列表，如下所示：

```kt
val adapter = UserListAdapter(this@MainActivity, 
response.body()//this is a arraylist 
)
```

在这里，我们有一个自定义适配器，我们将发送上下文和用户的`Array`列表。

# 创建自定义列表适配器

为了显示 REST API 的输出，我们需要创建一个自定义列表适配器，因此我们需要设计一个自定义列表适配器的 XML 文件。以下是列表中每一行的 XML 代码：

```kt
<?xml version="1.0" encoding="utf-8"?>
<android.support.constraint.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:padding="10dp">

    <TextView
        android:id="@+id/name"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:gravity="center_horizontal"
        android:padding="5dp"
        android:textAppearance="?android:textAppearanceMedium"
        android:textStyle="bold"
        app:layout_constraintBottom_toTopOf="@+id/contactNumber"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent"
        tools:text="@tools:sample/full_names" />

    <TextView
        android:id="@+id/contactNumber"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:gravity="center_horizontal"
        android:padding="5dp"
        android:textAppearance="?android:textAppearanceSmall"
        app:layout_constraintBottom_toTopOf="@+id/email"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@+id/name"
        tools:text="@tools:sample/cities" />

    <TextView
        android:id="@+id/email"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:gravity="center_horizontal"
        android:padding="5dp"
        android:textAppearance="?android:textAppearanceSmall"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@+id/contactNumber"
        tools:text="@tools:sample/cities" />

</android.support.constraint.ConstraintLayout>
```

在这里，我们有一个包含`name`、`contactNumber`和`email`的`TextView`。

然后，我们将创建适配器，命名为`UserListAdapter.kt`，如下所示：

```kt
class UserListAdapter(context: Context,
                      private val userList: List<UserModel>?) : BaseAdapter() {
    private val inflater: LayoutInflater = context.getSystemService(Context.LAYOUT_INFLATER_SERVICE)
            as LayoutInflater
    override fun getView(position: Int, convertView: View?, parent: ViewGroup?): View {
        val rowView = inflater.inflate(R.layout.user_list_item, parent, false)
        val name = rowView.findViewById(R.id.name) as TextView
        val email = rowView.findViewById(R.id.email) as TextView
        val contactNumber = rowView.findViewById(R.id.contactNumber) as TextView
        val userDetails = getItem(position) as UserModel
        name.text = userDetails.name
        email.text = userDetails.email
        contactNumber.text = userDetails.contactNumber
        return rowView
    }
    override fun getItem(position: Int): Any {
        return userList!![position]
    }
    override fun getItemId(position: Int): Long {
        return position.toLong()
    }
    override fun getCount(): Int {
        return userList!!.size
    }
}
```

这个类扩展了`BaseAdapter()`，这将添加几个继承的功能。

然后你需要添加`LayoutInflater`，它将 XML 布局转换为相应的`ViewGroups`和`Widgets`：

+   `getView()`为列表的一行创建一个视图。在这里，你需要定义所有基于 UI 的信息。

+   `getItem()`返回从服务器获取的列表位置。

+   `getItemId()`为列表中的每一行定义一个唯一的 ID。

+   `getCount()`返回列表的大小。

现在，在`getView()`中，你将添加布局元素，如下所示：

```kt
 val name = rowView.findViewById(R.id.name) as TextView
        val email = rowView.findViewById(R.id.email) as TextView
        val contactNumber = rowView.findViewById(R.id.contactNumber) as TextView
```

你永远不应该在主线程上执行长时间运行的任务。这将导致应用程序无响应（ANR）。

# 移动应用程序

一旦我们完成了代码，就是时候查看输出了。运行你的基本认证 Spring 项目，然后运行你的应用程序。以下是应用程序的输出，我们可以看到用户详情：

![图片](img/bb1d3a80-1f6b-4d39-9de8-b3a746152d48.png)

在下面的屏幕截图中，左侧是服务器 API，其中包含用户详情，右侧是 Android 应用程序的客户端输出：

![图片](img/2deb9410-34b7-440c-9604-1e9e61a024d0.png)

我们创建了一个客户端应用程序，它将获取基于基本认证的 Spring-Security REST API 的数据。

# 使用 Spring Security OAuth2 保护 REST

在最后一节，我们学习了如何创建一个基本的授权项目。这为项目提供了坚实的基础安全，但它不具备复杂或企业级项目所需的安全维度。由于这种安全可能被破解或黑客攻击，我们需要一个更稳固的安全框架来处理这类黑客攻击。OAuth 是最好的安全框架之一——它被 Google、Facebook、Twitter 和许多其他流行的平台广泛使用。现在我们将学习 OAuth2 及其应用。

# 什么是 OAuth2？

**OAuth** 是一种安全的授权协议，**OAuth2** 是 OAuth 协议的第二版。这个协议被称为**框架**。OAuth2 允许第三方应用程序提供对 HTTP 服务的有限访问，例如 Google、GitHub 或 Twitter。这种访问要么是为了所有者的利益，要么是为了使第三方应用程序能够访问用户账户。这就在网页和桌面或移动设备之间创建了一个授权流。它有一些重要的角色，用于控制用户的访问限制。

# OAuth2 角色

OAuth2 有四个角色：

+   **资源所有者**: 通常情况下，这就是你。

+   **资源服务器**: 服务器托管受保护的数据。例如，Google、Github 或 Twitter 托管你的个人和职业信息。

+   **客户端**: 一个请求资源服务器访问数据的程序。客户端可以是网站、桌面应用程序，甚至是移动应用程序。

+   **授权服务器**: 这个服务器将向客户端颁发访问令牌。这个令牌将是访问信息的密钥，它主要用于请求资源服务器以供客户端使用。

这是 OAuth 协议的一般工作流程图（每个协议的流程并不固定；它基于授权的类型）：

![图片](img/3a2aa391-2ab8-4090-bee9-6ed59fd69490.png)

下面是工作流程的步骤：

1.  为了访问服务资源，**应用程序**向**用户**发送**授权请求**。

1.  如果**用户**授权请求，**应用程序**将收到**授权许可**。

1.  **应用程序**将授权许可发送给**授权服务器**以获取**访问令牌**。

1.  如果**授权许可**有效且**应用程序**已认证，**授权服务器**将创建一个**访问令牌**。

1.  **应用程序** 从 **授权服务器** 获取 **访问令牌**。

1.  应用程序向 **资源服务器** 发送请求，以从服务器获取资源以及进行身份验证。

1.  使用令牌，**资源服务器** 向 **应用程序** 提供请求的资源。

# OAuth2 授权类型

有四种类型的 OAuth2 授权：

+   **授权代码**：在服务器端应用程序中使用，允许客户端获取一个长期访问令牌。然而，如果客户端请求服务器获取新令牌，此令牌将被无效化。

+   **隐式**：大部分情况下，这用于移动或 Web 应用程序。

+   **资源所有者密码凭证**：在这个授权中，凭证首先发送给客户。然后它们被发送到授权服务器。

+   **客户端凭证**：当客户端本身是资源所有者时使用。不需要从客户端端获取授权。

因此，这是 OAuth 协议的简要总结。现在让我们使用 Spring Security OAuth2 模块创建一个项目。

# 创建项目

我们将创建一个简单的基于 Spring Security OAuth2 的项目。为此，请访问 [`start.spring.io/`](https://start.spring.io/) 并根据您的需求修改给定的字段。

在这里，我们使用 Maven 项目，并将语言选择为 Kotlin。Spring Boot 版本为 2.1.1 (SNAPSHOT)。

选择生成项目后，您将获得一个 ZIP 文件的项目。解压并使用您的 IDE 打开此项目。

# Maven 依赖项

我们的主要依赖项是 `Web`、`Security`、`Cloud Security`、`Cloud OAuth2`、`JPA`、`H2`、`Lombok` 和 `Thymeleaf`。

下面是 `pom.xml` 中提到的 Maven 依赖项：

```kt
----
----
  <dependencies>
---
---
<!--spring security-->
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-config</artifactId>
    <version>5.2.0.BUILD-SNAPSHOT</version>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>

<!--spring cloud security-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-oauth2</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-security</artifactId>
</dependency>

----
----

<!--database-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <scope>runtime</scope>
</dependency>
----
----
```

# 配置资源服务器

资源服务器将拥有所有受保护的资源，这些资源由 OAuth2 令牌保护。现在是时候借助代码来了解这个资源服务器了。创建一个名为 **`ResourceServerConfig.kt`** 的资源服务器。

下面是我们的 **`ResourceServerConfig.kt`** 代码：

```kt
@Configuration
@EnableResourceServer
class ResourceServerConfig: ResourceServerConfigurerAdapter(){

    @Throws(Exception::class)
    override fun configure(http: HttpSecurity?) {
        http!!
                .authorizeRequests()
                .antMatchers("/open_for_all").permitAll() // anyone can enter
                .antMatchers("/private").authenticated() // only authorized user can enter
    }
}
```

要启用 OAuth 2.0 资源服务器机制的功能，您需要添加一个名为 `@EnableResourceServer` 的注解，尽管它是一个配置类，但您需要添加 `@Configuration` 注解。

此类扩展 `ResourceServerConfigurerAdapter`，然后扩展 `ResourceServerConfigurer`，这将使其能够覆盖和配置 `ResourceServerConfigurer`。

我们覆盖 `configure(http: HttpSecurity?)`，其中我们提到哪些 URL 路径受保护，哪些不受保护。

`authorizeRequests()` 允许根据 `HttpServletRequest` 的使用来限制访问。

`antMatchers()` 指的是映射中 Ant 风格路径模式实现的实现。

我们使用 `.antMatchers("/").permitAll()`，这允许所有用户访问此 URL 路径 `"/"`。此外，我们使用 `.antMatchers("/private").authenticated()`，这意味着用户需要令牌才能访问此 `/private` 路径。

# 配置授权服务器

授权服务器是一个配置类。在这个类中，我们将创建一个授权类型环境。授权类型帮助客户端从最终用户那里获取访问令牌。这个服务器的配置旨在实现客户端详情服务和令牌服务。它还负责全局启用或禁用机制中的某些组件。现在，创建一个名为 `AuthorizationServerConfig.kt` 的授权服务器类。

这是 `AuthorizationServerConfig.kt` 的代码：

```kt
@Configuration @EnableAuthorizationServer
class AuthorizationServerConfig: AuthorizationServerConfigurerAdapter() {

   @Autowired
   lateinit var authenticationManager: AuthenticationManager

    @Autowired
    lateinit var passwordEncoder: BCryptPasswordEncoder

    @Throws(Exception::class)
    override fun configure(security: AuthorizationServerSecurityConfigurer?) {
        security!!.checkTokenAccess("isAuthenticated()")
    }

    @Throws(Exception::class)
    override fun configure(clients: ClientDetailsServiceConfigurer?) {
       clients!!
               .inMemory()
               .withClient("client")
               .secret(passwordEncoder.encode("secret"))
               .authorizedGrantTypes("password")
               .authorities("ROLE_CLIENT", "ROLE_TRUSTED_CLIENT")
               .scopes("read", "write", "trust")
               .resourceIds("oauth2-resource")
               .accessTokenValiditySeconds(5000) // token validity time duration 5 minuets

    }

    @Throws(Exception::class)
    override fun configure(endpoints: AuthorizationServerEndpointsConfigurer?) {
        endpoints!!.authenticationManager(authenticationManager)
    }
}
```

`@EnableAuthorizationServer` 注解启用了 OAuth 2.0 授权服务器机制的功能。您需要添加 `@Configuration` 注解以使其成为配置类。

这个类扩展了 `AuthorizationServerConfigurerAdapter`，它又扩展了 `ResourceServerConfigurer`。这将使得能够覆盖和配置 `AuthorizationServerConfigurer`。有三个类型的 `configure()` 函数：

+   `ClientDetailsServiceConfigurer`: 这定义了客户端的详情服务。

+   `AuthorizationServerSecurityConfigurer`: 这定义了令牌端点的安全约束。

+   `AuthorizationServerEndpointsConfigurer`: 这定义了授权和令牌端点以及令牌服务。

根据我们的代码，在 `configure(security: AuthorizationServerSecurityConfigurer?)` 中，我们定义了是否检查已认证的令牌端点。

在 `configure(clients: ClientDetailsServiceConfigurer?)` 中，我们定义了 `ClientDetails` 服务。在这个项目中，我们没有使用数据库，因此我们使用 `ClientDetails` 服务的内存实现。以下是客户端的重要属性：

+   `withClient()`: 这是必需的，这是定义客户端 ID `"client"` 的地方。

+   `secret()`: 这是受信任客户端必需的，这是定义密钥 `"secret"` 的地方，但我们必须对密码进行编码。在这里，我们注入 `BCryptPasswordEncoder` 来编码密码或密钥。

+   `authorizedGrantTypes()`: 我们使用了 `"password"` 授权类型，这是客户端被授权使用的。

+   `scope()`: 范围用于限制客户端对资源的访问。如果范围未定义或为空，则表示客户端不受范围限制。在这里，我们使用 `"read"`、`"write"` 和 `"trust"`。

+   `authorities()`: 这用于授予客户端。

+   `resourceId()`: 这是一个可选 ID，用于资源。

+   `accessTokenValiditySeconds()`: 这指的是令牌的有效时间长度。

在 `configure(endpoints: AuthorizationServerEndpointsConfigurer?)` 中，我们已配置了 `AuthorizationEndpoint`，它支持授权类型。我们注入 `AuthenticationManager` 并通过 `AuthorizationServerEndpointsConfigurer` 进行配置。

# 创建安全配置

这是一个用于 Spring Security 的 Java 配置类，它允许用户在不使用 XML 的情况下轻松配置 Spring Security。创建一个名为 `SecurityConfiguration.kt` 的安全配置文件。以下是类的代码：

```kt
@Configuration
@EnableWebSecurity
class SecurityConfiguration: WebSecurityConfigurerAdapter() {

    @Throws(Exception::class)
    override fun configure(auth: AuthenticationManagerBuilder?) {
        auth!!
                .inMemoryAuthentication()
                .passwordEncoder(passwordEncoder())
             // user1 as USER
                .withUser("sunnat")
                .password(passwordEncoder().encode("password"))
                .roles("USER")
                .and()

                // user2 as ADMIN
               .withUser("admin")        
               .password(passwordEncoder().encode("password"))
                .roles("ADMIN")
    }

    @Throws(Exception::class)
    override fun configure(http: HttpSecurity?) {
        http!!
                .antMatcher("/**").authorizeRequests()
                .anyRequest().authenticated()
                .and()
                .formLogin()
                .and()
                .httpBasic()
    }

    @Bean(name = [BeanIds.AUTHENTICATION_MANAGER])
    @Throws(Exception::class)
    override fun authenticationManagerBean(): AuthenticationManager {
        return super.authenticationManagerBean()
    }

    @Bean
    fun passwordEncoder(): BCryptPasswordEncoder {
        return BCryptPasswordEncoder(16)
    }
}
```

这是一个配置类，因此你需要添加 `@Configuration` 注解。

此类扩展了 `WebSecurityConfigurerAdapter`，而 `@EnableWebSecurity` 注解提供了基于 Web 的安全机制。

根据此代码，我们在必需的功能中使用了两个 `@Bean` 注解。我们注入 `AuthenticationManager` 并通过 `AuthorizationServerEndpointsConfigurer` 进行配置。使用 `BCryptPasswordEncoder` 实例来编码密码。

在 `configure(http: HttpSecurity?)` 中，请注意以下内容：

+   `antMatcher("/**").authorizeRequests()` 表示此 `HttpSecurity` 只适用于以 `/**` 开头的 URL。

+   `anyRequest().authenticated()` 的使用保证了任何对我们的应用程序的请求都需要客户端进行确认。

+   `formLogin()` 允许用户通过基于表单的登录进行身份验证。

+   `httpBasic()` 表示用户通过 HTTP Basic 身份验证进行验证。

在 `configure(auth: AuthenticationManagerBuilder?)` 中，请注意以下内容：

+   `inMemoryAuthentication()` 包括将内存确认添加到 `AuthenticationManagerBuilder`，并将 `InMemoryUserDetailsManagerConfigurer` 恢复以允许自定义内存验证。

+   `passwordEncoder(passwordEncoder())` 表示密码将是一个编码密码。

+   `withUser("user")` 和 `withUser("admin")` 是用户的名称。

+   `password(passwordEncoder().encode("password"))` 是编码后的密码。

+   `roles("USER")` 和 `roles("ADMIN")` 是用户的角色。

# 创建控制器类

创建一个名为 `UserController.kt` 的控制器类，如下所示：

```kt
@RestController
@RequestMapping("/")
class UserController{

//    This is for all means there is no security issue for this URL path
    @GetMapping(value = ["/open_for_all", ""])
    fun home(): String{
        return "This area can be accessed by all."
    }

    //    Yu have to use token to get this URL path
    @GetMapping("/private")
    fun securedArea(): String{
        return "You used an access token to enter this area."
    }
}
```

在这里，我们将此类标注为 `@RestController`，它处理所有 Web 请求。`@RequestMapping("/")` 表示默认 URL 路径是 `"/"`。

`@GetMapping` 实现的功能是 `home()`，任何人都可以访问，以及 `securedArea()`，只有拥有 *访问令牌* 的人才能访问。我们在 `ResourceServerConfig` 类中配置了这些。

# 创建应用程序类

最后，创建一个名为 `SpringSecurityOAuth2Application.kt` 的应用程序类，这将把你的应用程序转换为 SpringBoot 应用程序：

```kt
@SpringBootApplication
class SpringSecurityOAuth2Application

fun main(args: Array<String>) {
    runApplication<SpringSecurityOAuth2Application>(*args)
}
```

# 应用程序属性

此步骤是可选的，尤其是在这个项目中。在这里，我们只是更改了这个项目的端口号。要更改它，请修改 `resources` 文件夹下的 `application.properties`：

```kt
#this project server port
server.port=8081
```

在这里，我们将端口号更改为 `8081`。

# 检查输出

如果你正在阅读本节，这意味着你已经正确配置了一切。完成项目后，你将拥有以下文件：

完成设置后，运行项目。如果没有错误，你可以找到运行窗口。以下截图显示没有错误，应用程序已准备好使用：

![](img/6e2f5ec2-e29c-4641-8b42-0288911faa18.png)

# 检查未受保护的 URL

现在，打开 *Insomnia* 应用程序。

从顶部邮箱创建一个 `GET` 请求，并使用 `http://localhost:8081/open_for_all` 的 URL。

你的结果将类似于以下截图：

![图片](img/956bc690-72ea-4a2e-b902-eec4e709ee8a.png)

在`ResourceServerConfig`类中，我们配置了`"/open_for_all"`可以被每个人访问。

# 获取访问令牌

从顶部邮箱创建一个`POST`请求，并写下`http://localhost:8081/oauth/token` URL。这是获取令牌的默认`POST` *URL*。

在多部分窗口中添加三个参数—`username=sunnat`、`password=password`和`grant_type=password`—：

![图片](img/b79568f5-19bf-4d32-b9c1-7f582311a7f4.png)

你可以在`SecurityConfiguration`类中找到`username`和`password`的信息，而`grant_type`可以在`AuthorizationServerConfig`中找到。在这里，我们使用**密码**授权类型。

前往**基本**窗口并输入**用户名**和**密码**。你可以在`AuthorizationServerConfig`类中找到这些信息，其中用户名在`withClient()`中提及，密码在`secret()`中。

我们添加了一张图片，展示了我们记录了**用户名**和**密码**的 Insomnia 工具。**现在点击发送按钮**。如果没有错误，你将获得以下`access_token`：

![图片](img/2fb2b231-90a0-4d9e-af22-e2249b6e8482.png)

你可以看到将用于访问受保护资源的`access_token`。"`expires_in`"表示在`4469`秒后令牌将过期。"`scope": "read write trust"`表示你可以读取、写入和修改资源。

# 访问受保护的 URL

我们找到了`access_token`，现在我们将使用它。为此，创建另一个`GET`请求并插入`http://localhost:8081/private`。

作为参数，使用具有给定令牌键值的`access_token`，然后点击发送：

![图片](img/a4b3e161-225e-4b04-bef5-76bdb512aa5f.png)

完成这些操作后，你可以访问受保护的`/private` URL，该 URL 在`ResourceServerConfig`类中进行了配置。

我们现在已准备好在我们的项目中使用 OAuth2 Spring Security。

# 常见错误和错误

在这个项目中，你可能会遇到一些常见错误。

例如，在构建和运行项目时可能会遇到一些错误。为了解决这个问题，请检查所有依赖项的版本是否为最新。此外，请确保每个依赖项都已存在。如果你使用数据库，请确保在`application.properties`中你有正确的数据库和模式名称。

在`POST`请求中，有时你会找到以下错误信息：

![图片](img/d1d5e9c8-90b9-4bae-b247-5bf90ca2d651.png)

之前的截图表明你输入了错误的`grant_type`。请检查参数以及你提及`grant_type`的`AuthorizationServerConfig`类：

![图片](img/e36835a3-80c0-47dc-a8dc-c52f7499284b.png)

请检查`SecurityConfiguration`类，并将系统的`username-password`与提供的`username`和`password`参数匹配。以下截图表示你在`Basic Auth`选项卡中输入了错误的客户端或密钥值：

![图片](img/6ed25197-c442-4fb8-ae9b-3cdc26f076d0.png)

上述截图表示你在基本认证选项卡中输入了错误的 `client` 或 `secret` 值。请将 `AuthorizationServerConfig` 中的 `client` 和 `secret` 值与基本认证选项卡中的值进行匹配：

![](img/ac63e400-14b3-44c0-92ab-5334b51000df.png)

上述截图表示您的令牌密钥已过期。您需要刷新一个新的访问令牌来解决此错误。

你可能会遇到其他错误。要查看解决方案，你总是可以搜索 StackOverflow ([`stackoverflow.com/`](https://stackoverflow.com/))。

# 摘要

在本章中，你学会了如何自信地使用 Spring Security。首先，我们介绍了 Spring Security 是什么以及其架构。我们还了解了使用 Spring Security 的优势，通过其特性和模块进行了探讨。现在，我们能够在任何项目中实现 Spring Security。我们学习了基本认证是什么，并通过一个示例展示了如何在项目中实现基本认证以及如何保护服务器中的资源。我们还学习了如何创建一个安全的 REST API。然后我们学习了如何创建一个 Android 客户端应用程序来从 REST API 中获取和使用受保护的资源。我们还学习了如何实现用户名和密码以获取基于基本认证的安全服务器的访问权限。此外，我们还熟悉了如何在客户端应用程序中的 listview 中使用自定义适配器。在最后一节中，我们探索了一个更安全的协议：OAuth2。我们学习了该协议的角色和工作流程。通过一个简单的项目，我们学习了如何配置 OAuth2 授权和资源服务器。最后，我们看到了如何使用第三方 HTTP 客户端检索 REST API。

在下一章中，我们将学习数据库，它非常重要，因为它是存储和处理您数据的主要地方。

# 问题

1.  Spring Security 的目标是？

1.  Spring Security 的基本类有哪些？

1.  需要哪个过滤器类用于 Spring Security？

1.  Spring Security 是否支持密码散列？

1.  OAuth 2.0 授权类型有哪些？

# 进一步阅读

这里有一份您可以参考的信息列表：

+   *Spring Security - 第三版* ([`www.packtpub.com/application-development/spring-security-third-edition`](https://www.packtpub.com/application-development/spring-security-third-edition)) by Mick Knutson, Robert Winch, Peter Mularien

+   *实战 Spring Security 5 for Reactive Applications* ([`www.packtpub.com/application-development/hands-spring-security-5-reactive-applications`](https://www.packtpub.com/application-development/hands-spring-security-5-reactive-applications)) by Tomcy John

+   *OAuth 2.0 烹饪书* ([`www.packtpub.com/virtualization-and-cloud/oauth-20-cookbook`](https://www.packtpub.com/virtualization-and-cloud/oauth-20-cookbook)) by Adolfo Eloy Nascimento
