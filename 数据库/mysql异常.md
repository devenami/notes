（1）

```java
com.mysql.jdbc.exceptions.jdbc4.CommunicationsException: Communications link failure
```

见名之意：超时，无法连接
所以解决办法也很好办：

``` properties
#在my.ini中添加
wait_timeout=1814400
#(21*3600*24) 21天，修改等待超时时间。
```

（2）

```java
com.mysql.jdbc.exceptions.jdbc4.MySQLNonTransientConnectionException: Data source rejected establishment of connection, message from server: "Too many connections"
```

原因：因为你的mysql安装目录下的my.ini中设定的并发连接数太少或者系统繁忙导致连接数被占满。
解决方式： 
打开MYSQL安装目录打开MY.INI找到max_connections（在大约第93行）默认是100 一般设到500～1000比较合适，重启mysql,这样1040错误就解决啦。 

```properties
max_connections=1000
```

（3）一直听大家说mysql是不区分大小写的，确实，在windows平台确实不区分大小写，可是一旦将程序移植到linux下，就有问题啦。因为在linux下mysql对大小写是敏感的。
  解决方法：找到etc文件夹下的my.cnf配置文件，找到mysqld：

```properties
Lower_case_table_names=1
```

​    （让大小写不敏感）不同的数字表示不同的意思，大家如果想详细了解，可以自己上网查一下，在此不详述了。

（4）往表中插入数据时出现：

```java
com.mysql.jdbc.exceptions.jdbc4.MySQLSyntaxErrorException: You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near '£¬photo_url='photos/non.gif', phone='013901290001', id_type='?¨ª???¡è', id_num='' at line 1，
```

 解决方法: 出现问题的SQL : 

```sql
insert into tb_process_form_attendance(runId,key)
```

key 为 MySQL 的关键词，需转义，使用 ``

```sql
insert into tb_process_form_attendance(runId,`key`)
```

（5）今天在写sql时遇到一个说在MySQL4.1中子查询是不能使用LIMIT的，手册中也明确指明 This version of MySQL doesn’t yet support ‘LIMIT & IN/ALL/ANY/SOME subquery。
解决办法：这样的语句是不能正确执行的。 

```sql
select * from table where id in (select id from table limit 10);
```

但是，只要你再来一层就行。如：

```sql
select * from table where id in (select t.id from (select * from table limit 10)as t)
```

（6）

```java
not unique table/alias 
```

则 SQL 语句中出现了非唯一的表或别名。
解决方法：
1、请检查出现问题位置的 SQL 语句中是否使用了相同的表名，或是定义了相同的表别名。
2、检查 SELECT 语句中要查询的字段名是不是定义重复，或者没有定义。