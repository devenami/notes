## logback

Logback是由log4j创始人设计的另一个开源日志组件。它包含下面几个模块：

- logback-core：其它两个模块的基础模块
- logback-classic：它是log4j的一个改良版本，同时它完整实现了slf4j API使你可以很方便地更换成其它日志系统如log4j或JDK14 Logging
- logback-access：访问模块与Servlet容器集成提供通过Http来访问日志的功能

## 配置介绍

- Logger、appender及layout

　　Logger作为日志的记录器，把它关联到应用的对应的context上后，主要用于存放日志对象，也可以定义日志类型、级别。
　　Appender主要用于指定日志输出的目的地，目的地可以是控制台、文件、远程套接字服务器、 MySQL、PostreSQL、 Oracle和其他数据库、 JMS和远程UNIX Syslog守护进程等。 
　　Layout 负责把事件转换成字符串，格式化的日志信息的输出。

- logger context

　　各个logger 都被关联到一个 LoggerContext，LoggerContext负责制造logger，也负责以树结构排列各logger。其他所有logger也通过org.slf4j.LoggerFactory 类的静态方法getLogger取得。 getLogger方法以 logger名称为参数。用同一名字调用LoggerFactory.getLogger 方法所得到的永远都是同一个logger对象的引用。

- 有效级别及级别的继承

　　Logger 可以被分配级别。级别包括：TRACE、DEBUG、INFO、WARN 和 ERROR，定义于ch.qos.logback.classic.Level类。如果 logger没有被分配级别，那么它将从有被分配级别的最近的祖先那里继承级别。root logger 默认级别是 DEBUG。

- 打印方法与基本的选择规则

　　打印方法决定记录请求的级别。例如，如果 L 是一个 logger 实例，那么，语句 L.info("..")是一条级别为 INFO的记录语句。记录请求的级别在高于或等于其 logger 的有效级别时被称为被启用，否则，称为被禁用。记录请求级别为 p，其 logger的有效级别为 q，只有则当 p>=q时，该请求才会被执行。
该规则是 logback 的核心。级别排序为： TRACE < DEBUG < INFO < WARN < ERROR

## 默认配置

如果配置文件 logback-test.xml 和 logback.xml 都不存在，那么 logback 默认地会调用BasicConfigurator ，创建一个最小化配置。

最小化配置由一个关联到根 logger 的ConsoleAppender 组成。

输出用模式为%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n 的 PatternLayoutEncoder 进行格式化。

root logger 默认级别是 DEBUG。

- Logback的配置文件

　　Logback 配置文件的语法非常灵活。正因为灵活，所以无法用 DTD 或 XML schema 进行定义。尽管如此，可以这样描述配置文件的基本结构：以`<configuration>`开头，后面有零个或多个`<appender>`元素，有零个或多个`<logger>`元素，有最多一个`<root>`元素。

```xml
<configuration>
	<appender></appender>
    <appender></appender>
    
    <logger></logger>
    <logger></logger>
    
    <root></root>
</configuration>
```

- Logback默认配置的步骤

1. 尝试在 classpath下查找文件logback-test.xml；
2. 如果文件不存在，则查找文件logback.xml；
3. 如果两个文件都不存在，logback用BasicConfigurator自动对自己进行配置，这会导致记录输出到控制台。

## 配置文件解析

### 根节点`<Configuration>`

属性：

* scan: 当此属性设置为true时，配置文件如果发生改变，将会被重新加载，默认值为true。
* scanPeriod: 设置监测配置文件是否有修改的时间间隔，如果没有给出时间单位，默认单位是毫秒。当scan为true时，此属性生效。默认的时间间隔为1分钟。
* debug: 当此属性设置为true时，将打印出logback内部日志信息，实时查看logback运行状态。默认值为false。

### \<contextName>

用来设置上下文名称，每个logger都关联到logger上下文，默认上下文名称为default。

### \<property>

用来定义变量值，它有两个属性name和value，通过<property>定义的值会被插入到logger上下文中，可以使“${}”来使用变量。

属性：

* name: 变量的名称
* value: 的值时变量定义的值

### \<timestamp>

获取时间戳字符串，他有两个属性key和datePattern

属性：

* key: 标识此`<timestamp>` 的名字；
* datePattern: 设置将当前时间（解析配置文件的时间）转换为字符串的模式，遵循java.txt.SimpleDateFormat的格式。

### \<appender>

负责写日志的组件，它有两个必要属性name和class。

* name指定appender名称
* class指定appender的全限定名。

#### ConsoleAppender

把日志输出到控制台，有以下子节点：

* `<encoder>`：对日志进行格式化。
* `<target>`：字符串System.out(默认)或者System.err

#### FileAppender

把日志添加到文件，有以下子节点：

* `<file>`：被写入的文件名，可以是相对目录，也可以是绝对目录，如果上级目录不存在会自动创建，没有默认值。
* `<append>`：如果是 true，日志被追加到文件结尾，如果是 false，清空现存文件，默认是true
* `<encoder>`：对记录事件进行格式化。
* `<prudent>`：如果是 true，日志会被安全的写入文件，即使其他的FileAppender也在向此文件做写入操作，效率低，默认是 false。

#### RollingFileAppender

滚动记录文件，先将日志记录到指定文件，当符合某个条件时，将日志记录到其他文件。有以下子节点：

* `<file>`：被写入的文件名，可以是相对目录，也可以是绝对目录，如果上级目录不存在会自动创建，没有默认值。
* `<append>`：如果是 true，日志被追加到文件结尾，如果是 false，清空现存文件，默认是true
* `<rollingPolicy>`:当发生滚动时，决定RollingFileAppender的行为，涉及文件移动和重命名
* `<prudent>`：当为true时，不支持FixedWindowRollingPolicy。支持TimeBasedRollingPolicy，但是有两个限制，1不支持也不允许文件压缩，2不能设置file属性，必须留空。	

* `<triggeringPolicy >`告知 RollingFileAppender 合适激活滚动
* `<encoder>`：对记录事件进行格式化。负责两件事，一是把日志信息转换成字节数组，二是把字节数组写入到输出流。
  PatternLayoutEncoder 是唯一有用的且默认的encoder ，有一个`<pattern>`节点，用来设置日志的输入格式。使用“%”加“转换符”方式，如果要输出“%”，则必须用“\”对“\%”进行转义。

##### \<rollingPolicy>

属性class定义具体的滚动策略类。

1、`class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy"`

最常用的滚动策略，它根据时间来制定滚动策略，既负责滚动也负责出发滚动。有以下子节点：

* `<fileNamePattern>`：必要节点，包含文件名及“%d”转换符，“%d”可以包含一个`java.text.SimpleDateFormat`指定的时间格式，如：%d{yyyy-MM}。
  如果直接使用 %d，默认格式是 yyyy-MM-dd。RollingFileAppender的file子节点可有可无，通过设置file，可以为活动文件和归档文件指定不同位置，当前日志总是记录到file指定的文件（活动文件），活动文件的名字不会改变；
  如果没设置file，活动文件的名字会根据fileNamePattern 的值，每隔一段时间改变一次。“/”或者“\”会被当做目录分隔符。	
* `<maxHistory>`:可选节点，控制保留的归档文件的最大数量，超出数量就删除旧文件。假设设置每个月滚动，且`<maxHistory>`是6，则只保存最近6个月的文件，删除之前的旧文件。注意，删除旧文件是，那些为了归档而创建的目录也会被删除。

2、`class="ch.qos.logback.core.rolling.SizeBasedTriggeringPolicy"`

查看当前活动文件的大小，如果超过指定大小会告知RollingFileAppender 触发当前活动文件滚动。只有一个节点:

* `<maxFileSize>`:这是活动文件的大小，默认值是10MB。

##### \<triggeringPolicy >

1、`class="ch.qos.logback.core.rolling.FixedWindowRollingPolicy"`

根据固定窗口算法重命名文件的滚动策略。有以下子节点：

* `<minIndex>`:窗口索引最小值
* `<maxIndex>`:窗口索引最大值，当用户指定的窗口过大时，会自动将窗口设置为12。
* `<fileNamePattern>`:必须包含“%i”例如，假设最小值和最大值分别为1和2，命名模式为 mylog%i.log,会产生归档文件mylog1.log和mylog2.log。还可以指定文件压缩选项，例如，mylog%i.log.gz 或者log%i.log.zip

### \<logger>

用来设置某一个包或具体的某一个类的日志打印级别、以及指定`<appender>`。

`<loger>`仅有一个name属性，一个可选的level和一个可选的addtivity属性。

可以包含零个或多个`<appender-ref>`元素，标识这个appender将会添加到这个logger

* name: 用来指定受此loger约束的某一个包或者具体的某一个类。
* level: 用来设置打印级别，大小写无关

###  \<root>

它是根logger,是所有`<logger>`的上级。只有一个level属性，因为name已经被命名为"root",且已经是最上级了。

可以包含零个或多个`<appender-ref>`元素

* level: 用来设置打印级别，大小写无关



## 使用案例

### pom.xml

```xml
　<properties>
　　　　<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
　　　　<logback.version>1.1.7</logback.version>
　　　　<slf4j.version>1.7.21</slf4j.version>
　　</properties>

　　<dependencies>
　　　　<dependency>
　　　　　　<groupId>org.slf4j</groupId>
　　　　　　<artifactId>slf4j-api</artifactId>
　　　　　　<version>${slf4j.version}</version>
　　　　　　<scope>compile</scope>
　　　　</dependency>
　　　　<dependency>
　　　　　　<groupId>ch.qos.logback</groupId>
　　　　　　<artifactId>logback-core</artifactId>
　　　　　　<version>${logback.version}</version>
　　　　</dependency>
　　　　<dependency>
　　　　　　<groupId>ch.qos.logback</groupId>
　　　　　　<artifactId>logback-classic</artifactId>
　　　　　　<version>${logback.version}</version>
　　　　　　</dependency>
　　</dependencies>
```

### logback.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration debug="false">
    <!--定义日志文件的存储地址 勿在 LogBack 的配置中使用相对路径-->
    <property name="LOG_HOME" value="/home" />
    <!-- 控制台输出 -->
    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
            <!--格式化输出：%d表示日期，%thread表示线程名，%-5level：级别从左显示5个字符宽度%msg：日志消息，%n是换行符-->
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n</pattern>
        </encoder>
    </appender>
    <!-- 按照每天生成日志文件 -->
    <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <!--日志文件输出的文件名-->
            <FileNamePattern>${LOG_HOME}/TestWeb.log.%d{yyyy-MM-dd}.log</FileNamePattern>
            <!--日志文件保留天数-->
            <MaxHistory>30</MaxHistory>
        </rollingPolicy>
        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
            <!--格式化输出：%d表示日期，%thread表示线程名，%-5level：级别从左显示5个字符宽度%msg：日志消息，%n是换行符-->
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n</pattern>
        </encoder>
        <!--日志文件最大的大小-->
        <triggeringPolicy class="ch.qos.logback.core.rolling.SizeBasedTriggeringPolicy">
            <MaxFileSize>10MB</MaxFileSize>
        </triggeringPolicy>
    </appender>
    
    <!-- 指定固定包的日志级别 -->
    <logger name="org.hibernate.type.descriptor.sql.BasicBinder" level="TRACE" />
    <logger name="org.hibernate.type.descriptor.sql.BasicExtractor" level="DEBUG" />

    <!-- 日志输出级别 -->
    <root level="INFO">
        <appender-ref ref="STDOUT" />
    </root>
</configuration>
```

### Test.java

```java
public static void main(String[] args) throws JoranException {
    // 加载配置文件, 如果使用的默认位置可以省略
    LoggerContext context = (LoggerContext) LoggerFactory.getILoggerFactory();
    JoranConfigurator configurator = new JoranConfigurator();
    configurator.setContext(context);
    context.reset();
    configurator.doConfigure(Thread.currentThread().
			getContextClassLoader().getResource("android/conf/logback.xml"));
    StatusPrinter.printInCaseOfErrorsOrWarnings(context);

    // 创建日志对象、输出日志
    Logger logger = LoggerFactory.getLogger("myTestName");
    logger.error("输出日志");
}
```

### 日志输出

```
2019-06-06 15:12:30.005 [main] ERROR myTestName - 输出日志
```

