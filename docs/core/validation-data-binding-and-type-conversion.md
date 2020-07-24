# 3.验证，数据绑定和类型转换

将验证视为业务逻辑有利有弊，Spring提供的验证(和数据绑定)设计不排除任何一种。特别地，验证不应该绑定到web层，而应该易于本地化，并且应该能够插入任何可用的验证器。考虑到这些问题，Spring提供了一个验证器契约，它既基本又在应用程序的每一层都非常有用。

数据绑定对于将用户输入动态绑定到应用程序的域模型(或用于处理用户输入的任何对象)非常有用。Spring提供了`DataBinder`来实现这一点。`Validator`和`DataBinder`组成了验证包，它主要用于但不限于web层。

BeanWrapper是Spring框架中的一个基本概念，在很多地方都有使用。但是，您可能不需要直接使用BeanWrapper。然而，由于这是参考文档，我们觉得应该进行一些解释。我们将在本章中解释BeanWrapper，因为如果您打算使用它，那么您很可能在尝试将数据绑定到对象时使用它。

Spring的DataBinder和底层BeanWrapper都使用PropertyEditorSupport实现来解析和格式化属性值。PropertyEditor和PropertyEditorSupport类型是JavaBeans规范的一部分，也会在本章中解释。Spring 3引入了一个核心。提供一般类型转换工具的转换包，以及用于格式化UI字段值的高级“格式”包。您可以使用这些包作为PropertyEditorSupport实现的更简单的替代方案。本章也将讨论它们。

Spring通过设置基础设施和Spring自己的Validator契约的适配器来支持Java Bean验证。应用程序可以一次全局启用Bean验证(如Java Bean验证中所描述的)，并专门为所有验证需求使用它。在web层中，应用程序可以进一步为每个DataBinder注册控件本地Spring验证器实例，如配置DataBinder中所述，这对于插入定制验证逻辑非常有用。


## 3.1.基于Spring Validator接口的验证

Spring提供了一个`Validator`接口，您可以使用它来验证对象。`Validator`接口的工作方式是使用一个`Errors`对象，这样，在验证时，验证器可以向`Errors`对象报告验证失败。

考虑下面一个小数据对象的例子:

```java
public class Person {

    private String name;
    private int age;

    // the usual getters and setters...
}
```

下一个示例通过实现org.springframework.validation的以下两个方法为Person类提供验证行为。验证器接口:

* `supports(Class)`: 判断当前`Validator`是否支持目标类的验证
* `validate(Object, org.springframework.validation.Errors)`: 对目标对象进行验证，并将错误信息收集到`Errors`对象中

实现验证器相当简单，特别是当您知道Spring框架还提供的`ValidationUtils`助手类时。下面的示例为`Person`实例实现验证器:

```java
public class PersonValidator implements Validator {

    /**
     * This Validator validates only Person instances
     */
    public boolean supports(Class clazz) {
        return Person.class.equals(clazz);
    }

    public void validate(Object obj, Errors e) {
        ValidationUtils.rejectIfEmpty(e, "name", "name.empty");
        Person p = (Person) obj;
        if (p.getAge() < 0) {
            e.rejectValue("age", "negativevalue");
        } else if (p.getAge() > 110) {
            e.rejectValue("age", "too.darn.old");
        }
    }
}
```

`ValidationUtils`类上的静态`rejectIfEmpty(..)`方法用于在`name`属性为空或空字符串时拒绝它。查看`ValidationUtils` javadoc，了解除了前面显示的示例之外，它还提供了哪些功能。

虽然可以实现单个验证器类来验证富对象中的每个嵌套对象，但最好将每个嵌套对象类的验证逻辑封装在其自己的验证器实现中。“富”对象的一个简单示例是由两个字符串属性(第一个和第二个名称)和一个复杂的`Address`对象组成的`Customer`。`Address`对象可以独立于`Customer`对象使用，因此实现了一个不同的`AddressValidator`。如果您希望您的`CustomerValidator`重用包含在`AddressValidator`类中的逻辑，而不求助于复制和粘贴，您可以依赖注入或实例化您的`CustomerValidator`中的`AddressValidator`，如下面的示例所示:

```java
public class CustomerValidator implements Validator {

    private final Validator addressValidator;

    public CustomerValidator(Validator addressValidator) {
        if (addressValidator == null) {
            throw new IllegalArgumentException("The supplied [Validator] is " +
                "required and must not be null.");
        }
        if (!addressValidator.supports(Address.class)) {
            throw new IllegalArgumentException("The supplied [Validator] must " +
                "support the validation of [Address] instances.");
        }
        this.addressValidator = addressValidator;
    }

    /**
     * 对于Customer及其子类实例，该校验都支持
     */
    public boolean supports(Class clazz) {
        return Customer.class.isAssignableFrom(clazz);
    }

    public void validate(Object target, Errors errors) {
        ValidationUtils.rejectIfEmptyOrWhitespace(errors, "firstName", "field.required");
        ValidationUtils.rejectIfEmptyOrWhitespace(errors, "surname", "field.required");
        Customer customer = (Customer) target;
        try {
            errors.pushNestedPath("address");
            ValidationUtils.invokeValidator(this.addressValidator, customer.getAddress(), errors);
        } finally {
            errors.popNestedPath();
        }
    }
}
```

验证错误被报告给传递给验证器的错误对象。对于Spring Web MVC，您可以使用`< Spring:bind/>`标记来检查错误消息，但是您也可以自己检查错误对象。关于它提供的方法的更多信息可以在javadoc中找到。

## 3.2.将编码解析为错误信息

我们讨论了数据绑定与验证。本节将介绍与验证错误对应的输出错误信息。在上一节所示的示例中，我们拒绝了`name`和`age`字段。如果我们想通过使用`MessageSource`输出错误消息，我们可以使用拒绝字段时提供的错误代码(在本例中是`name`和`age`)来实现。


## 3.3.Bean操作和`BeanWapper`

### 3.3.1.设置和获取基本的和嵌套的属性

**Table 11.属性例子**

| 表达式                 | 说明 |
| ---------------------- | ---- |
| `name`                 |      |
| `account.name`         |      |
| `account[2]`           |      |
| `account[COMPANYNAME]` |      |
