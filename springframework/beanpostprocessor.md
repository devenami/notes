## BeanPostProcessor

`BeanPostProcessor`是Spring提供的`bean`通过`反射`初始化对象后进行调用的接口。

## 作用

通过实现`BeanPostProcessor`接口，并将实现类交由`spring`管理，我们可以实现在`bean`创建完成时，手动的处理一些事情，如获取指定的方法、注解、属性等等。

## 使用

创建`JAVA`类并实现`BeanPostProcessor`接口。同时在类上标注`@Component`。

```java
@Component
public class BeanPostProcessor implements BeanPostProcessor {
    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        return bean;
    }

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        return bean;
    }
```

**postProcessBeforeInitialization**

对象创建完成，但并未初始化时进行调用。

**postProcessAfterInitialization**

对象创建完成，且已被成功的初始化，当前对象为直接可用的对象。



## 遇到的问题

### BeanPostProcessor中处理的方法，反射调用后切面失效

`BeanPostProcessor`处理的对象是未经过切面代理的。也就是说，在`BeanPostProcessor`中获取到的`bean`实例，并不能正确的相应Spring AOP功能。

**解决方案**：

可以使用`曲线救国`的方式，在`BeanPostProcessor`处理逻辑中，仅存储`beanName`和`methodName`，在使用的位置使用`ApplicationContext`通过`beanName`获取bean，从而解决问题。

#### 案例

**处理bean**

```java
@Component
public class AndroidBeanPostProcessor implements BeanPostProcessor {

    // 入口方法集合
    private static volatile List<AndroidMethodWrap> enterMethodList = Collections.synchronizedList(new ArrayList<>());

    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        Method[] methods = bean.getClass().getMethods();
        Arrays.stream(methods).forEach(method -> {
            AndroidEnter androidEnter = method.getAnnotation(AndroidEnter.class);
            if (androidEnter != null) {
                AndroidMethodWrap wrap = new AndroidMethodWrap();
                wrap.setBean(beanName);
                wrap.setMethod(method.getName());
                wrap.setMethodArgs(method.getParameterTypes());
                wrap.setSendHeader(androidEnter.sendHeader());
                int repeat = androidEnter.repeat();
                if (repeat < 1) {
                    throw new IllegalArgumentException("Repeat in AndroidEnter must be great than 0");
                }
                wrap.setRepeat(repeat);
                enterMethodList.add(wrap);
            }
        });
        return bean;
    }

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {return bean;}

    public static List<AndroidMethodWrap> getEnterMethodList() {
        return enterMethodList;}
}
```

**反射调用bean的方法**

```java
List<AndroidMethodWrap> enterMethodList = AndroidBeanPostProcessor.getEnterMethodList();
enterMethodList.forEach(wrap -> {
    try {
        String beanName = wrap.getBean();
        // 获取新的bean
        Object bean = AndroidSpringContextContainer.getBean(beanName);
        String methodName = wrap.getMethod();
        Class<?>[] methodArgs = wrap.getMethodArgs();
        Method method = bean.getClass().getMethod(methodName, methodArgs);
        Object[] objs = new Object[methodArgs.length];
        method.invoke(bean, objs);
    } catch (NoSuchMethodException | IllegalAccessException | InvocationTargetException e) {
        throw new RuntimeException(e);
    }
});
```

