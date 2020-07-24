# 3.验证，数据绑定和类型转换

## 3.1.基于Spring Validator接口的验证

Spring提供了一个可以用来校验对象的`Validator`接口。

```java
public class Person {

    private String name;
    private int age;

    // the usual getters and setters...
}
```

下面的例子通过`org.springframework.validation.Validator`接口为`Person`类提供了验证：

* `supports(Class)`: 判断当前`Validator`是否支持目标类的验证
* `validate(Object, org.springframework.validation.Errors)`: 对目标对象进行验证，并将错误信息收集到`Errors`对象中


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


// TODO

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
