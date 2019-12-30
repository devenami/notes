## 修改预定义的JSON转换器

 springboot2.x版本修改了以前设置转换器的方式

### 原方式：

通过实现`org.springframework.web.servlet.config.annotation.WebMvcConfigurer`接口，实现`configureMessageConverters(List<HttpMessageConverter<?>> converters)`方法后，将自定义的转换器添加到`converters`集合的最后面即可。

### 现方式：

现在需要提供一个`org.springframework.boot.autoconfigure.http.HttpMessageConverters`类的bean到spring容器中,用于将预定义的转换器集合替换掉才行。

对比以前的方式，现在的方式更加灵活，spring也不需要从源转换器集合中寻找，直接使用用户提供的集合。

实现方式

```java
  @Bean
  public HttpMessageConverters httpMessageConverters() {
    // 自定义json格式化
    MappingJackson2HttpMessageConverter converter = new MappingJackson2HttpMessageConverter(
        JsonUtil.getMapper());
    return new HttpMessageConverters(converter);
  }
```

