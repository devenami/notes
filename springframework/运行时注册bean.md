## 运行时移除bean

```java
    /**
     * 动态移除 bean
     *
     * @param context spring context
     * @param clazz   被移除的 bean 类型
     */
    public void removeBeanFromIoc(ApplicationContext context, Class<?> clazz) {
        // 转型 ApplicationContext
        ConfigurableApplicationContext applicationContext = (ConfigurableApplicationContext) context;
        // 获取 bean 工厂
        DefaultListableBeanFactory beanFactory = (DefaultListableBeanFactory) applicationContext.getBeanFactory();
        // 获取 bean 名称
        String[] beanNames = applicationContext.getBeanNamesForType(clazz);
        if (beanNames != null) {
            // 根据名称移除 bean
            Arrays.stream(beanNames).forEach(beanFactory::removeBeanDefinition);
        }
    }
```

## 运行时注册bean

```java
    /**
     * 动态注册bean到ioc
     *
     * @param context  spring context
     * @param beanName 新注册的bean名称
     * @param clazz    bean类型
     */
    public void registerBean2IOC(ApplicationContext context, String beanName, Class<?> clazz) {
        // 转型 ApplicationContext
        ConfigurableApplicationContext applicationContext = (ConfigurableApplicationContext) context;
        // 获取 bean 工厂
        DefaultListableBeanFactory beanFactory = (DefaultListableBeanFactory) applicationContext.getBeanFactory();

        // 定义bean
        BeanDefinitionBuilder beanDefinitionBuilder = BeanDefinitionBuilder.genericBeanDefinition(clazz);
        // 设置bean的属性值
        beanDefinitionBuilder.addPropertyValue("属性名", "属性值");
        beanDefinitionBuilder.addPropertyReference("属性名", "引用的bean名称");
        // 注册到 ioc
        beanFactory.registerBeanDefinition(beanName, beanDefinitionBuilder.getBeanDefinition());
    }
```

## 缺陷

通过这种方式注册的bean，只能通过`ApplicationContext.getBean(clazz)`获取的才是正确的bean，无法参与到`Autowired`这种注入。

如果想通过这种方式实现bean对象的替换是不可行的。

解决动态替换的方案有：

* 动态注册bean后，反射源对象，将新注册的bean属性拷贝到源对象(不建议使用)
* 使用 `java agent` 
* 使用 `spring -loaded` 

