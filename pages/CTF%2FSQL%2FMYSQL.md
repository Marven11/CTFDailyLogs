# mysql的字符串存储机制
	- [参考](https://zhuanlan.zhihu.com/p/112806366) [例题]([[CTF/WP/BUUCTF/FBCTF2019 Products Manager]])
	- 对于`VARCHAR(n)`，如果插入的字符串不够长，mysql会在后方插入空格，如果插入的字符串过长，mysql会将其截断
- # CVE-2012-2122
	- 当连接 MariaDB/MySQL 时,输入的密码会与期望的正确密码比较,由于不正确的处
	  理,会导致即便是 memcmp()返回一个非零值,也会使 MySQL 认为两个密码是相同
	  的。 也就是说只要知道用户名,不断尝试就能够直接登入 SQL 数据库。按照公告
	  说法大约 256 次就能够蒙对一次
	  MariaDB versions from 5.1.62, 5.2.12, 5.3.6, 5.5.23 are not.
	  MySQL versions from 5.1.63, 5.5.24, 5.6.6 are not.