## 原因

数据库在开发过程中使用了N张表， 同时由于测试，开发等过程产生了非常多的错误数据，因此需要对数据库中的所有表进行清空操作。如果直接手工对每个表指定**DELETE**或**truncate**错误，效率将会异常低下。



## 清档步骤

###  一：获取清表脚本

执行如下脚本获取清表脚本, 需要将一下脚本中的数据库名替换为自己的库名

```sql
SELECT
	CONCAT(
		'truncate TABLE ',
		table_schema,
		'.',
		TABLE_NAME,
		';'
	)
FROM
	INFORMATION_SCHEMA. TABLES
WHERE
	table_schema IN ('my_db_name');
```

生成的脚本如下：

```sql
truncate TABLE my_db_name.my_table_1;
truncate TABLE my_db_name.my_table_2;
```

### 二：执行清表脚本

在MYSQL客户端执行以上获取的清表脚本即可。