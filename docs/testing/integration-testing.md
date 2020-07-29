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

### 3.2.1. 上下文管理和缓存

Spring TestContext框架提供了一致的Spring `ApplicationContext`实例和`WebApplicationContext`实例的加载，以及这些上下文的缓存。支持缓存已加载的上下文非常重要，因为启动时间会成为一个问题——不是因为Spring本身的开销，而是因为Spring容器实例化的对象需要时间来实例化。例如，一个拥有50到100个Hibernate映射文件的项目可能会花费10到20秒来加载映射文件，并且在每个测试fixture中运行每个测试之前会导致整个测试运行速度变慢，从而降低开发人员的工作效率。

测试类通常声明XML或Groovy配置元数据的资源位置数组(通常在类路径中)或用于配置应用程序的组件类数组。这些位置或类与web.xml或用于生产部署的其他配置文件中指定的位置或类相同或类似。

默认情况下，一旦加载，配置的`ApplicationContext`将在每个测试中重用。因此，每个测试套件只需要花费一次设置成本，随后的测试执行就会快得多。在这个上下文中，术语“测试套件”意味着在同一个JVM中运行的所有测试——例如，针对给定的项目或模块，从Ant、Maven或Gradle构建中运行的所有测试。在测试破坏了应用程序上下文并需要重新加载(例如，通过修改bean定义或应用程序对象的状态)的不太可能的情况下，可以配置TestContext框架来重新加载配置并在执行下一个测试之前重新构建应用程序上下文。

请参阅使用TestContext框架进行上下文管理和上下文缓存。

### 3.2.2. 测试装置的依赖注入

当TestContext框架加载您的应用程序上下文时，它可以选择使用依赖注入配置测试类的实例。这为从应用程序上下文中使用预先配置的bean来设置测试fixture提供了一种方便的机制。这里一个强大的好处是，您可以跨各种测试场景重用应用程序上下文(例如，用于配置spring管理的对象图、事务代理、`DataSource`实例等)，从而避免为单个测试用例重复复杂的测试fixture设置。

例如，考虑一个场景，其中我们有一个类(`HibernateTitleRepository`)，它为`Title`域实体实现了数据访问逻辑。我们想要编写集成测试来测试以下领域:
- Spring配置:基本上，与`HibernateTitleRepository` bean配置相关的所有内容都正确并呈现出来了吗?
- Hibernate映射文件配置:所有的映射都正确吗?延迟加载设置是否正确?
- HibernateTitleRepository的逻辑:这个类的配置实例是否按预期执行?

请参阅使用TestContext框架注入测试fixture的依赖项。

### 3.2.3. 事务管理

在访问真实数据库的测试中，一个常见的问题是它们对持久性存储的状态的影响。即使在使用开发数据库时，状态的更改也可能影响将来的测试。而且，许多操作——比如插入或修改持久数据——不能在事务之外执行(或验证)。

TestContext框架解决了这个问题。默认情况下，框架为每个测试创建并回滚一个事务。您可以编写假设存在事务的代码。如果您在测试中调用以事务方式代理的对象，它们将根据其配置的事务语义正确地运行。此外，如果测试方法在为测试管理的事务中运行时删除了所选表的内容，则事务默认情况下回滚，并且数据库返回到执行测试之前的状态。通过使用在测试的应用程序上下文中定义的`PlatformTransactionManager` bean，可以为测试提供事务支持。

如果您想要提交一个事务(不常见，但是当您想要一个特定的测试填充或修改数据库时，偶尔会有用)，您可以通过使用`@Commit`注释告诉TestContext框架使事务提交而不是回滚。

### 3.2.4. 集成测试的支持类

Spring TestContext框架提供了一些`abstract`类型的支持类，这使得集成测试更容易。这些基本测试类提供了定义良好的测试框架挂钩，以及方便的实例变量和方法，让您访问:

- `ApplicationContext`，用于执行显式bean查找或测试整个上下文的状态。
- `JdbcTemplate`，用于执行SQL语句来查询数据库。您可以使用此类查询在执行与数据库相关的应用程序代码之前和之后确认数据库状态，Spring确保此类查询在与应用程序代码相同的事务范围内运行。当与ORM工具一起使用时，一定要避免误报。

此外，您可能想创建自己的自定义的、应用程序范围的超类，使用特定于项目的实例变量和方法。

请参阅TestContext框架的支持类。

### 3.3. JDBC测试支持
`org.springframework.test.jdbc`包包含`JdbcTestUtils`，它是与JDBC相关的实用程序函数的集合，旨在简化标准数据库测试场景。具体来说，`JdbcTestUtils`提供了以下静态实用程序方法。

- `countRowsInTable(..)`: 计算给定表中的行数。
- `countRowsInTableWhere(..)`: 使用所提供的`WHERE`子句计算给定表中的行数。
- `deleteFromTables(..)`: 删除给定表的所有数据。
- `deleteFromTablesWhere(..)`: 使用所提供的`WHERE`子句从给定表中删除行。
- `dropTables(..)`: 删除指定的表。

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
