### 什么是Conditional

`condition` 是`spring`提供的一个特性，用于在初始化对象时对初始化条件进行检测，满足条件的对象才予以创建。

`SpringBoot`提供了`ConditionalOnBean`、`ConditionalOnClass`、`ConditionalOnProperty`等。分别用于不同的场景下对`bean`对象的创建行为进行限制。



### 如何自定义Conditional

#### 1、创建自定义的注解

```java
@Target({ ElementType.TYPE, ElementType.METHOD })
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Conditional(MyCondition.class)
public @interface MyConditionAnnotation {
    String value() default "";
}
```

通过创建自定义的注解，并且使用`Conditional`注解在我们自定义的注解上面。`Conditional`需要传递一个实现了`org.springframework.context.annotation.Condition`的类。`spring`会通过调用该实现类的`matches`方法来判断是否要创建对象。

#### 2、创建`Condition`的实现类。

在`SpringBoot`中，我们可以通过继承`org.springframework.boot.autoconfigure.condition.SpringBootCondition`类，并重写`getMatchOutcome`方法进行实现。

`getMatchOutcome`需要返回一个`ConditionOutcome`对象，我们可以通过`new ConditionOutcome(bool,desc)`来创建该对象，构造器中需要接受两个参数，第一个为当前匹配是否成功，第二个为描述。当我们检测到当前环境满足Bean的初始化条件，就需要将第一个参数设为`true`。

**创建匹配类**

```java
public class MyCondition extends SpringBootCondition {
    @Override
    public ConditionOutcome getMatchOutcome(ConditionContext context, AnnotatedTypeMetadata metadata) {
        // 处理逻辑
        return new ConditionOutcome(true, "");
    }
}
```

#### 3、注意事项

1、在匹配类中可能无法自动注入bean

由于`Conditional`用于标注当前环境是否满足`bean`的创建条件，所以通过`context.getBeanFactory().getBean("name")`可能无法获取到希望的对象，因为这里查询的对象可能还没有被初始化。

2、不能通过自动注入配置的类来获取配置参数值。

就如上面解释的。在匹配类被调用时，对象可能没有被初始化，所以当我们直接访问配置类时，可能获取不到正确的配置。

解决方式是使用`PropertyResolver`进行获取指定属性。

```java
PropertyResolver resolver = context.getEnvironment();
String property = resolver.getProperty("属性名");
```



### 完整案例

#### 自定义注解

```java
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Conditional(OnEnvironmentConditional.class)
public @interface ConditionalOnEnvironment {

    /**
     * 当前条件支持的环境列表
     */
    Environment[] value() default Environment.PRODUCT;

}
```

#### 可能会用到的枚举

```java
public enum Environment {
    /**
     * 开发环境
     */
    DEVELOP,
    /**
     * 测试环境
     */
    TEST,
    /**
     * 生产环境
     */
    PRODUCT
}
```

#### 编写匹配条件类

```java
public class OnEnvironmentConditional extends SpringBootCondition {

    @Override
    public ConditionOutcome getMatchOutcome(ConditionContext context, AnnotatedTypeMetadata metadata) {
        PropertyResolver resolver = context.getEnvironment();
        String property = resolver.getProperty("business.environment", Environment.PRODUCT.name());
        Environment environment = Environment.valueOf(property.toUpperCase());
        if (metadata.isAnnotated(ConditionalOnEnvironment.class.getName())) {
            Map<String, Object> annotationAttributes = metadata.getAnnotationAttributes(ConditionalOnEnvironment.class.getName());
            Environment[] envs = (Environment[]) annotationAttributes.get("value");
            List<Environment> envList = Arrays.asList(envs);
            if (envList.contains(environment)) {
                return new ConditionOutcome(true, "环境检查通过");
            }
            return new ConditionOutcome(false, "环境检查未通过");
        }
        return new ConditionOutcome(true, "no ConditionalOnEnvironment marked");
    }
}
```

#### 使用方式

```java
@Bean
@ConditionalOnEnvironment({Environment.DEVELOP, Environment.TEST})
public FilterRegistrationBean<AllowOriginFilter> allowOriginFilter() {
    FilterRegistrationBean<AllowOriginFilter> bean = new FilterRegistrationBean<>();
    bean.setFilter(new AllowOriginFilter());
    bean.addUrlPatterns("/*");
    bean.setName("AllowOriginFilter");
    bean.setOrder(1);
    return bean;
}
```

#### 配置文件

```properties
business.environment = develop
```

