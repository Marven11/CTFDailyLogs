# 常用脚本
	- 打通用waf
		- ```shell
		  find . -type f -path "*.php" | xargs sed -i 's/<?php/<?php\nfile_exists("\/tmp\/waf.php") \&\& include "\/tmp\/waf.php";\n/'
		  ```
	- mysql的备份还原
		- ```shell
		  #备份mysql数据库
		  mysqldump -u 用户名 -p 密码 数据库名 > back.sql　　　　
		  mysqldump --all-databases > bak.sql　　　　　　
		  #还原mysql数据库
		  mysql -u 用户名 -p 密码 数据库名 < bak.sql
		  
		  ```