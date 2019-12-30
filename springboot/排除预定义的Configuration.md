## 排除预定义的Configuration

**SpringBoot** 提供了大量的预配置项，但并不是所有的预配置项都符合我们的要求，那么我们就需要将预配置项从**SpringBoot**的生命周期中移除，主要实现有两种方式。

### 通过注解排除自动配置项

`org.springframework.boot.autoconfigure.SpringBootApplication`注解提供了`exclude`和`excludeName`属性对配置项进行移除

`exclude`需要提供排除的Class，`excldeName`需要提供排除类的全类名。

这种方式的优点是使用简单。

### 通过自定义过滤器排除

**SpringBoot**允许我们自己编写`Starter`来对不同的框架实现预配置。在编写`Starter`的同时，我们不能帮用户选择项目的启动类，也就是我们无法使用`org.springframework.boot.autoconfigure.SpringBootApplication`注解，那么基于注解排除预配置项就无法实现。

如果我们自动以的`Starter`与`SpringBoot`提供的配置或者其他第三方的配置存在冲突的情况，那么我们就必须想办法将冲突项排除。

**SpringBoot**给我们提供了`org.springframework.boot.autoconfigure.AutoConfigurationImportFilter`接口来过滤所有的自动配置项，我们只需要实现该接口，并按照**SpringBoot**的要求，将该类配置到`META-INF/spring.factories`中即可。

**实现接口并对指定的配置项进行排除**

```java
import java.util.Collections;
import java.util.HashSet;
import java.util.Set;
import org.springframework.boot.autoconfigure.AutoConfigurationImportFilter;
import org.springframework.boot.autoconfigure.AutoConfigurationMetadata;

/**
 * spring boot 默认提供的数据源自动配置排除过滤器
 */
public class AutoConfigurationSkipFilter implements
    AutoConfigurationImportFilter {

  // 所有需要跳过的自动配置项
  private static final Set<String> SHOULD_SKIP = new HashSet<>(
      // 初始化要排除的自动配置项的全类名
  Collections.singletonList("org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration")
  );

  @Override
  public boolean[] match(String[] autoConfigurationClasses,
      AutoConfigurationMetadata autoConfigurationMetadata) {
    boolean[] matches = new boolean[autoConfigurationClasses.length];
    for (int i = 0; i < matches.length; i++) {
      // 对所有需要排除的配置项的match设置为false
      matches[i] = !SHOULD_SKIP.contains(autoConfigurationClasses[i]);
    }
    return matches;
  }
}

```

**META-INF/spring.factories**

```properties
org.springframework.boot.autoconfigure.AutoConfigurationImportFilter=\
  mypackage.AutoConfigurationSkipFilter
```

如果存在多个**ImportFilter**接口的实现，可以使用`,`分隔