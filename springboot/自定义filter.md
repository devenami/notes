### 什么是Filter

`Filter`属于`Servlet`规范内的组件，用于过滤请求。

其运行原理类似于AOP，可以在请求前后执行部分逻辑。



### 如何自定义Filter

#### 编写Filter类

```java
public class MyFilter implements javax.servlet.Filter {
    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
    }

    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain chain) throws IOException, ServletException {
		// 请求处理之前
        chain.doFilter(request, response);
        // 请求处理之后
    }

    @Override
    public void destroy() {
    }
}
```

#### 注册Filter

在普通的`JAVA WEB`项目中可以在`web.xml`中进行注册过滤器，但是在`SpringBoot`项目中不存在`web.xml`配置文件。

`Servlet3`提供了通过注解的方式注册Filter。也可以通过`SpringBoot`提供的方式注册Filter

**Servlet 3 注解方式注册Filter**

```java
@javax.servlet.annotation.WebFilter(urlPatterns = "/*", filterName = "myFilter")
public class MyFilter implements javax.servlet.Filter {}
```

**SpringBoot Bean 方式注册Filter**

通过在`Spring IOC`中注册`FilterRegistrationBean`就可以自动注册过滤器。

```java
@Bean
public FilterRegistrationBean<MyFilter> myFilter() {
    FilterRegistrationBean<MyFilter> bean = new FilterRegistrationBean<>();
    bean.setFilter(new MyFilter());
    bean.addUrlPatterns("/*");
    bean.setName("myFilter");
    bean.setOrder(1);
    return bean;
}
```

