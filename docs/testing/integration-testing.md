# 3.集成测试

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
