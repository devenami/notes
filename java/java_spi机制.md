### 什么是SPI

SPI的全名为：Service Provider Interface，这个东西是针对厂商或者插件的，在java.util.ServiceLoad的文档中有详细的介绍。

简单来说，我们的系统中抽象的各个模块，往往有很多不同的实现方案，比如日志模块的方案，jdbc模块的方案等。我们一般推荐模块之间基于接口编程，模块之间不对实现类进行硬编码。



### Java SPI机制

Java SPI提供了为某个接口寻找服务实现的的机制，有点类似于IOC的思想，就是将装配的控制权移交到程序之外，在模块化设计中这个机制尤为重要。



### 约定

当服务的提供者提供了服务接口的一种实现后，在**jar**包的**META-INF/services/**目录里同时创建一个服务接口命名的文件。该文件里就是实现服务接口的具体实现类。

而当外部程序装配这个模块时，就能通过该配置文件找到具体的实现类名，并装载实例化，完成模块的注入。

基于这样一个约定就能很好的找到服务接口的实现类，而不需要在代码里指定。jdk提供服务实现查找的一个工具类为**java.util.ServiceLoad**。



### 代码实现SPI

#### 定义抽象接口

首先需要定义一个抽象接口，用于将功能分离。具体实现由各个厂商负责。

```java
package com.demo.inter;

public interface Service {

    void say();
}
```

#### 定义实现接口的类

实现类用于实现抽象接口中的具体方法，为服务提供功能需要。

```java
package com.demo.a.impl;

import com.demo.inter.Service;

public class ServiceImpl implements Service {
    @Override
    public void say() {
        System.out.println("I am A");
    }
}
```

#### 定义配置文件

在基于MAVEN搭建的Java工程中，需要将 **META-INF/services/** 建立在 **resources** 目录下。

在 **resources** 目录下创建  **META-INF/services/** 目录，并在  **META-INF/services/** 目录下建立 **com.demo.inter.Service** 文件。

**com.demo.inter.Service** 文件中填写具体的实现类，如下：

```java
com.demo.a.impl.ServiceImpl
```

#### 定义测试类

```java
package com.demo;

import com.demo.inter.Service;
import java.util.Iterator;
import java.util.ServiceLoader;

public class Main {

    public static void main(String[] args) {
        // 使用ServiceLoad加载 META-INF/services/com.demo.inter.Service 文件下的内容
        ServiceLoader<Service> services = ServiceLoader.load(Service.class);
        // 获取所有的实现类
        Iterator<Service> iterator = services.iterator();
        if (iterator.hasNext()) {
            // META-INF/services/com.demo.inter.Service 文件中配置的实现类
            Service service = iterator.next();
            service.say();
        }
    }
}

```

运行代码后输出：

```java
I am A
```

