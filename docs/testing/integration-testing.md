# 3.集成测试

## 3.1. 概述

能够在不需要部署到应用服务器或连接到其他企业基础设施的情况下执行一些集成测试是很重要的。这样做可以让你测试诸如:

- 正确连接Spring IoC容器上下文。
- 数据访问使用JDBC或一个ORM工具。这包括SQL语句的正确性、Hibernate查询、JPA实体映射等等。

Spring框架为`spring-test`模块中的集成测试提供了一流的支持。实际JAR文件的名称可能包括发布版本，也可能在org.springframework中。测试表单，取决于您从哪里得到它(请参阅依赖项管理部分以获得解释)。这个库包括`org.springframework.test`包，它包含用于与Spring容器集成测试的有价值的类。此测试不依赖于应用程序服务器或其他部署环境。这类测试的运行速度比单元测试慢，但比等效的Selenium测试或依赖于部署到应用服务器的远程测试快得多。

单元和集成测试支持以注解驱动的Spring TestContext框架的形式提供。TestContext框架与所使用的实际测试框架无关，它允许在各种环境中检测测试，包括JUnit、TestNG和其他环境。

## 3.2. 集成测试的目标

Spring的集成测试支持有以下主要目标:

- 管理测试之间的Spring IoC容器缓存。
- 提供测试fixture实例的依赖注入。
- 提供适合于集成测试的事务管理。
- 提供特定于spring的基类，以帮助开发人员编写集成测试。

接下来的几节将描述每个目标，并提供到实现和配置细节的链接。

## 3.4. 注解
### 3.4.1. Spring测试注解

Spring框架提供了以下一组特定于Spring的注释，您可以在单元测试和集成测试中结合TestContext框架使用这些注释。有关更多信息，请参见相应的javadoc，包括默认属性值、属性别名和其他详细信息。

Spring的测试注解包括以下内容:

- `@BootstrapWith`
- `@ContextConfiguration`
- `@WebAppConfiguration`
- `@ContextHierarchy`
- `@ActiveProfiles`
- `@TestPropertySource`
- `@DynamicPropertySource`
- `@DirtiesContext`
- `@TestExecutionListeners`
- `@Commit`
- `@Rollback`
- `@BeforeTransaction`
- `@AfterTransaction`
- `@Sql`
- `@SqlConfig`
- `@SqlMergeMode`
- `@SqlGroup`

#### `@BootstrapWith`
`@BootstrapWith`是一个类级注解，您可以使用它来配置Spring TestContext框架如何引导。具体来说，您可以使用`@BootstrapWith`来指定一个定制的`TestContextBootstrapper`。请参阅引导TestContext框架的章节了解更多细节。

#### `@ContextConfiguration`
`@ContextConfiguration`定义了用于确定如何为集成测试加载和配置`ApplicationContext`的类级元数据。具体来说，`@ContextConfiguration`声明用于加载上下文的应用程序上下文资源位置或组件类。

资源位置通常是位于类路径中的XML配置文件或`Groovy`脚本，而组件类通常是`@Configuration`类。但是，资源位置也可以引用文件系统中的文件和脚本，组件类可以是`@Component`类、`@Service`类，等等。请参阅组件类以了解更多细节。

下面的例子显示了引用XML文件的`@ContextConfiguration`注解:
```java
@ContextConfiguration("/test-config.xml")
class XmlApplicationContextTests {
    // class body...
}
```

下面的例子显示了引用一个类的`@ContextConfiguration`注解:
```java
@ContextConfiguration(classes = TestConfig.class)
class ConfigClassApplicationContextTests {
    // class body...
}
```

除了声明资源位置或组件类之外，还可以使用`@ContextConfiguration`来声明`ApplicationContextInitializer`类。下面的例子展示了这种情况:
```java
@ContextConfiguration(initializers = CustomContextIntializer.class)
class ContextInitializerTests {
    // class body...
}
```

### 3.4.2. 标准注解
### 3.4.3. JUnit 4注解
### 3.4.4. JUnit 5注解
### 3.4.5. 元注解

## 3.5. Spring TestContext框架
## 3.6. Spring MVC测试框架
## 3.7. WebTestClient

WebTestClient是一个围绕WebClient的壳，使用它来执行请求，并公开一个专用的、连贯的API来验证响应。WebTestClient通过使用模拟请求和响应绑定到WebFlux应用程序，或者它可以通过HTTP连接测试任何web服务器。

### 3.7.1. 设置

要创建WebTestClient，您必须从多个服务器设置选项中选择一个。实际上，您要么将WebFlux应用程序配置为绑定到它，要么使用URL连接到运行中的服务器。

#### 绑定Controller

下面的例子展示了如何创建一个服务器设置来一次测试一个@Controller:

```java
client = WebTestClient.bindToController(new TestController()).build();
```

前面的示例加载WebFlux Java配置并注册给定的控制器。通过使用模拟请求和响应对象，在没有HTTP服务器的情况下测试得到的WebFlux应用程序。在构建器上有更多的方法来定制默认的WebFlux Java配置。

### 3.7.2. Writing Tests

```java
client.get().uri("/persons/1")
            .accept(MediaType.APPLICATION_JSON)
            .exchange()
            .expectStatus().isOk()
            .expectHeader().contentType(MediaType.APPLICATION_JSON);

```

```java
client.get().uri("/persons")
        .exchange()
        .expectStatus().isOk()
        .expectBodyList(Person.class).hasSize(3).contains(person);
```
