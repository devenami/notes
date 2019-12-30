#### 创建存储过程语法

```sql
CREATE
	[DEFINER = user] 
	PROCEDURE sp_name ([proc_parameter[,...]])
	[characteristic ...] routine_body
```

#### 创建函数语法

```sql
CREATE
	[DEFINER = user]
	FUNCTION sp_name ([func_parameter[,...]]) 
	RETURNS type [characteristic ...] routine_body
```

#### 参数解释

| 参数           | 格式                                                         |
| -------------- | ------------------------------------------------------------ |
| proc_parameter | [ IN \| OUT \|  INOUT ]   参数名   参数类型                  |
| func_parameter | 参数名   参数类型                                            |
| type           | 所有mysql支持的数据类型                                      |
| characteristic | COMMENT 'string'<br />              \| LANGUAGE SQL<br />              \| [NOT] DETERMINISTIC <br />              \| { CONTAINS SQL <br />                         \|NO SQL <br />                         \|READS SQL DATA<br />                         \| MODIFIES SQL DATA}<br />              \| SQL SECURITY { DEFINER \| INVOKER } |
| routine_body   | 常规SQL代码                                                  |

每个存储过程或函数都应该有相应的数据库，如果设置想用的数据库则需要将名称修改为 db_name.sp_name.

调用存储过程和函数需要使用 `CALL`

参数列表必须存在**（）**，无论有没有参数

每一个参数默认为**IN**输入参数，如果要指定其他类型，可以使用**OUT**或**INOUT**修饰在参数前。

**注： **对于存储过程来说存在*IN*、*OUT*、*INOUT*， 但是对于函数来说，只存在*IN*

*IN*参数对于存储过程是来时输入参数，存储过程可以修改该参数，但是当存储过程执行完成返回后，修改的参数值对外界并不可见。也就是说：外界看到的输入参数的值，还是原来调用存储过程之前的值，并看不过存储过程修改后的值。

*OUT*是存储过程返回给调用者的参数。初始值为*NULL*，但是当存储过程执行完返回时，对于*OUT*参数的修改对外界是可见的。也就是说：外界看到的输出参数的值，在存储过程内部被修改后，外界该变量的值也跟着改变了。

*INOUT*参数的初始化是由调用者处理的。该值即使输入参数也是输出参数，且存储过程对于该参数的修改对外界是可见的。

**完整的存储过程例子**

```sql
mysql> delimiter //

mysql> CREATE PROCEDURE simpleproc (OUT param1 INT)
    -> BEGIN
    ->   SELECT COUNT(*) INTO param1 FROM t;
    -> END//
Query OK, 0 rows affected (0.00 sec)

mysql> delimiter ;

mysql> CALL simpleproc(@a);
Query OK, 0 rows affected (0.00 sec)

mysql> SELECT @a;
+------+
| @a   |
+------+
| 3    |
+------+
1 row in set (0.00 sec)
```

**delimiter**命令将MySQL的分隔符由**；**改为了**//**，这使得存储过程中使用的**；**可以传递给服务器，而不是由MySQL本身去解释。

**returns**子句只能在函数体中使用。它声明了函数体的返回类型，并且函数体中必须存在**return value**语句。

如果return 后面的value是其他的类型，则会被强制转换为 returns 指定的类型。

**完整的函数的列子**

```sql
mysql> CREATE FUNCTION hello (s CHAR(20))
mysql> RETURNS CHAR(50) DETERMINISTIC
    -> RETURN CONCAT('Hello, ',s,'!');
Query OK, 0 rows affected (0.00 sec)

mysql> SELECT hello('world');
+----------------+
| hello('world') |
+----------------+
| Hello, world!  |
+----------------+
1 row in set (0.00 sec)
```

在函数中没有必要使用**delimiter**将**；**改变为其他的分隔符，因为函数内部是不会包含分号的。

MySQL允许在函数体中包含**DDL**语句，如*create*、*drop*等。同时也允许在存储过程中包含事务语句，如**commit**，但并不能在存储函数中包含事务语句。

在存储过程和存储函数中不允许使用**USE**语句，但可以在存储过程或存储函数调用之前，显示的使用**USE db_name**来声明使用的数据库。

**COMMENT**行为是MySQL扩展的，可以用来描述存储过程或存储函数，这些信息可以在**SHOW CREATE PROCEDURE**和**SHOW CREATE FUNCTION**语句执行后显示。

**LANGUAGE**行为存储语句书写的语言。服务端会忽略这种行为，且服务端仅支持标准的”SQL“。

如果一个例程总是为相同的输入参数生成相同的结果，那么它被认为是“**deterministic**确定性的”，否则被认为是“**not deterministic**不确定性的”（默认行为）。

#### 声明语句结束符

​	**delimiter $$** 将结束符替换为 **$$**

​	默认的语句结束符为**；**，在存储过程中更改结束符时，尽量在结束时修改回来。

#### 开始、结束符

​	**begin**...**end**

​	多个结束符之间可以嵌套，开始符和结束符前面可以加标签，如`[label:]begin...end[label]`

#### 变量的定义及赋值

​	变量声明使用如下格式

​		`declare 变量名[,可定义多个] 数据类型 [default 默认值];`

​	设置变量使用如下格式

​		`set 变量名 = 表达式[,变量名 = 表达式...];`

**用户变量**

​	用户变量名一般以**@**开头

​	用户变量的作用范围为整个用户区间，用户可以在**任何位置进行访问**，当然也不局限与多个存储过程访问

​	如 proc_1 设置一个 @my_name 的用户变量，proc_2 访问 @my_name 变量，若proc_1先执行，则proc_2就可以获取到proc_1中定义的变量值

​	禁止滥用用户变量，会导致程序难以理解及维护。

#### 查询数据库中存在的存储过程

* SELECT name FROM mysql.proc WHERE db = '数据库名';
* SELECT routine_name FROM information_schema.routines WHERE routine_schema = '数据库名';
* SHOW procedure status WHERE db = '数据库名';

**查看存储过程详情**

show procedure 数据库名.存储过程名

**修改存储过程**

alter procedure 数据库名.存储过程名

**删除存储过程**

drop procedure 数据库名.存储过程名

#### 判断语句

**if**

```sql
if 表达式 then
	代码块
else
	代码块
end if;
```

**case**

```sql
case 参数名
when 参数值 then
	代码块
when 参数值 then
	代码块
end case;
```

#### 循环语句

**while**

```sql
while 循环条件 do
	代码块
end while;
```

**repeat**

```sql
repeat
	代码块
util 循环条件
end repeat;
```

**loop**

loop循环没有循环条件，需要用户自己通过 **leave**关键字跳出循环

```sql
标签名:loop
	代码块
	if 表达式 then
		leave 标签名；
	end if;
end loop;
```

**标签**

​	标签可以用在 begin repeat while loop 语句前，格式为[标签名:]，标签只能在合法的语句前面使用。

​	可以使用 **leave 标签名** 跳出循环，使指定达到复合语句的最后一步。

​	也可以使用 **iterate 标签名** 引用复合语句标签来重复开始复合语句。

#### 游标

**步骤**

1. 创建游标：`declare 游标名 cursor for select 语句；`
2. 打开游标：`open 游标名;`
3. 提取游标数据：`fetch 游标名 [into 变量1，变量2...];`每次提取一行数据。
4. 关闭游标：`close 游标名;`

#### 内部函数

**found_row()** 与 **row_count()**

* 这两个函数都是mysql中用于统计上一条sql语句影响的行数
* **found_row()** 用于判断 **select** 语句得到的函数
* **row_count()** 用于判断 **update** 或 **delete** 影响的函数。但是 **update** 前后置一样，**row_count()** 则为 **0**

#### 最佳实践

