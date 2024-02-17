tags:: CTF/Nginx, CTF/SSRF

- 读nginx记录，发现一个隐藏的网页，访问得到flag
- # 反序列化
	- 和 [[CTFWEB/WP/CTFShow/web308]]相同
	- if判断
		- 无
	- checkVersion
		- dao
			- fun: checkUpdate: ^^伪协议任意文件读取^^
			  id:: fa89aeb7-b109-475a-9533-ea8b1e14d0cf
	- addDpt
		- service
			- insert_dbt
				- dao: 普通SQL，无法注入
	- getAllDbt
		- service
			- get_dpt_all
				- dao: 普通SQL，无法注入
	- clearCache
		- service: 调用dao: clearCache
		- dao: shell_exec，但是只能有数字和换行
- 利用，和web308相同
- # 访问隐藏网页
- 读取`file:///etc/nginx/nginx.conf`，看到一个隐藏的网页
- 利用 ((fa89aeb7-b109-475a-9533-ea8b1e14d0cf)) 访问`http://127.0.0.1:4476`得到flag
-