## JDBC连接数据库 mysql serverTimezone

驱动包用的是mysql-connector-java-8.0.11.jar
新版的驱动类改成了com.mysql.cj.jdbc.Driver
新版驱动连接url也有所改动
I、指定时区

//北京时间东八区

```
serverTimezone=GMT+8 
```

这个时区要设置好，不然会出现时差，
如果你设置serverTimezone=UTC，连接不报错，
但是我们在用java代码插入到数据库时间的时候却出现了问题。
比如在java代码里面插入的时间为：2018-06-24 17:29:56
但是在数据库里面显示的时间却为：2018-06-24 09:29:56
有了8个小时的时差
UTC代表的是全球标准时间 ，但是我们使用的时间是北京时区也就是东八区，领先UTC八个小时。

```
//北京时间东八区
serverTimezone=GMT%2B8 
//或者使用上海时间
serverTimezone=Asia/Shanghai
```

### 引用

[JDBC连接数据库 mysql serverTimezone useSSL 时差](https://blog.csdn.net/love20yh/article/details/80799610)