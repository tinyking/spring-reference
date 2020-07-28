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

### 2.1.4. Spring Web Reactive
