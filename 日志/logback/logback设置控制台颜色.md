## 原因

如果我们使用IDEA运行一个**Spring Boot**的项目，控制台打印的日志是彩色的，但当我们自定义**日志配置文件**后彩色却消失了。 

若我们不是使用的**Spring Boot**来搭建项目，那我们的控制台打印的日志也是不带颜色。

但是我们想要实现为**Spring Boot**那中形式该怎么办呢？

## 为日志增加颜色

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration scan="true" scanPeriod="60 seconds" debug="false">

    <!--输出到控制台-->
    <appender name="console" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <!--<pattern>%d %p (%file:%line\)- %m%n</pattern>-->
            <!--格式化输出：%d:表示日期    %thread:表示线程名     %-5level:级别从左显示5个字符宽度  %msg:日志消息    %n:是换行符-->
            <pattern>%black(console-) %red(%d{yyyy-MM-dd HH:mm:ss}) %green([%thread]) %highlight(%-5level) %boldMagenta(%logger{36}:%line): - %cyan(%msg%n)</pattern>
            <charset>UTF-8</charset>
        </encoder>
    </appender>

    <root level="info">
        <appender-ref ref="console" />
    </root>

</configuration>
```

**注意事项：**

```xml
<pattern>%black(console-) %red(%d{yyyy-MM-dd HH:mm:ss}) %green([%thread]) %highlight(%-5level) %boldMagenta(%logger{36}:%line): - %cyan(%msg%n)</pattern>
```

1.第一点，颜色%black %red等等 ，需要用括号将你要显示本颜色的子模块括起来

2.第二点，%red颜色等，前面要与上一个模块使用空格隔开

3.第三点，%-5：右边补空格至5字符，%+5：左边补空格至5字符



## pattern 的特殊标记

| 标记            | 解释                          |
| --------------- | ----------------------------- |
| %color(tag)     | 为tag设置color                |
| %highlight(tag) | tag高亮显示                   |
| %d{pattern}     | 日期及格式                    |
| %thread         | 线程                          |
| %level          | 日志等级                      |
| %logger{num}    | 打印日志所在类，长度限制在num |
| %line           | 行号                          |
| %msg            | 日志消息                      |



## 引用

文档引用：

[spring boot logback日志颜色渲染](https://www.cnblogs.com/sxdcgaq8080/p/7885340.html)

[logback 官网文档](https://logback.qos.ch/manual/layouts.html#coloring)