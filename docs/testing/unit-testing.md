# 2. 单元测试

依赖注入应该使您的代码比传统Java EE开发更少地依赖容器。组成应用程序的pojo应该可以在JUnit或TestNG测试中进行测试，使用`new`操作符实例化对象，而不需要Spring或任何其他容器。您可以使用模拟对象(结合其他有价值的测试技术)单独测试代码。如果遵循Spring的体系结构建议，那么代码基的干净分层和组件化将简化单元测试。例如，您可以通过stub或mock DAO或存储库接口来测试服务层对象，而不需要在运行单元测试时访问持久数据。

真正的单元测试通常运行得非常快，因为不需要设置运行时基础设施。将真正的单元测试作为开发方法的一部分可以提高生产力。您可能不需要测试章节的这一节来帮助您为基于ioc的应用程序编写有效的单元测试。然而，对于某些单元测试场景，Spring框架提供了模拟对象和测试支持类，这些将在本章中进行描述。

## 2.1. 模拟对象
Spring包括许多专门用于模拟的包:

- Environment
- JNDI
- Servlet API
- Spring Web Reactive

### 2.1.1. Environment

在`org.springframework.mock.env`中包含了`Environment`和`PropertySource`的模拟实现。`MockEnvironment`和`MockPropertySource`对于为依赖于特定环境属性的代码开发容器外测试非常有用。

### 2.1.2. JNDI
`org.springframework.mock.jndi`包包含JNDI SPI的部分实现，您可以使用它为测试套件或独立应用程序设置简单的JNDI环境。例如，如果JDBC数据源实例在测试代码中绑定到与在Java EE容器中相同的JNDI名称，那么您可以在测试场景中重用应用程序代码和配置，而无需修改。


### 2.1.3. Servlet API

`org.springframework.mock.web`包包含一组全面的Servlet API模拟对象，这些对象对于测试web上下文、控制器和过滤器非常有用。这些模拟对象是针对在Spring的Web MVC框架中使用的，通常比动态模拟对象(如EasyMock)或其他Servlet API模拟对象(如MockObjects)使用更方便。

### 2.1.4. Spring Web Reactive

`org.springframework.mock.http.server.reactive`包含用于WebFlux应用程序的`ServerHttpRequest`和`ServerHttpResponse`的模拟实现。`org.springframework.mock.web.server`包含一个模拟`ServerWebExchange`，它依赖于那些模拟请求和响应对象。

`MockServerHttpRequest`和`MockServerHttpResponse`都是从服务器特定实现的同一个抽象基类中扩展出来的，并与它们共享行为。例如，模拟请求在创建之后是不可变的，但是您可以使用`ServerHttpRequest`中的`mutate()`方法来创建修改后的实例。

为了让模拟响应正确地实现写契约并返回写完成句柄(即`Mono<Void>`)，默认情况下它使用带有`cache().then()`的`Flux`，它缓冲数据并使其可用于测试中的断言。应用程序可以设置自定义编写函数(例如，测试一个无限流)。

WebTestClient构建在模拟请求和响应的基础上，为测试没有HTTP服务器的WebFlux应用程序提供支持。客户机还可以用于运行服务器的端到端测试。

## 2.2. 单元测试支持类

### 2.2.1. 一般的测试工具

`org.springframework.test.util`包包含了一些用于单元和集成测试的通用实用工具。

`ReflectionTestUtils`是一组基于反射的实用程序方法。您可以在测试场景中使用这些方法，当您需要更改一个常量的值，设置一个非公共字段，调用一个非公共setter方法，或调用一个非公共配置或生命周期回调方法时，为以下用例测试应用程序代码:

- ORM框架(如JPA和Hibernate)允许`private`或`protected`字段访问，而不是域实体中的属性的`public` setter方法。
- Spring对注解(如@`Autowired`、`@Inject`和`@Resource`)的支持，这些注解为`private`或`protected`字段、setter方法和配置方法提供依赖注入。
- 对生命周期回调方法使用`@PostConstruct`和`@PreDestroy`等注释

`AopTestUtils`是一个与aop相关的实用程序方法的集合。可以使用这些方法获取对隐藏在一个或多个Spring代理之后的底层目标对象的引用。例如，如果您已经通过使用库(如EasyMock或Mockito)将bean配置为动态mock，并且该mock包装在Spring代理中，那么您可能需要直接访问底层的mock，以在其上配置期望并执行验证。关于Spring的核心AOP实用程序，请参阅`AopUtils`和`AopProxyUtils`。

### 2.2.2. Spring MVC测试工具

`org.springframework.test.web`包包含`ModelAndViewAssert`，您可以将其与JUnit、TestNG或其他用于处理Spring MVC `ModelAndView`对象的单元测试的测试框架结合使用。
