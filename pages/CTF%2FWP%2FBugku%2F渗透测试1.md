- # 1~6
	- 打页面的php在线执行功能即可
- # 7&8
	- 扫内网
		- ```
		  192.168.0.4:3306 open
		  192.168.0.4:80 open
		  192.168.0.2:80 open
		  192.168.0.1:80 open
		  192.168.0.1:22 open
		  192.168.0.3:8080 open
		  192.168.0.1:8080 open
		  192.168.0.2:9999 open
		  192.168.0.1:9999 open
		  ```
		- ```
		  [*] WebTitle: http://192.168.0.1:8080   code:302 len:0      title:None 跳转url: http://192.168.0.1:8080/login;jsessionid=A8649DFCD09C896D5F387AE8F916D35F
		  [*] WebTitle: http://192.168.0.3:8080   code:302 len:0      title:None 跳转url: http://192.168.0.3:8080/login;jsessionid=D5E2531BBF330B8D45CBB786A3A86E4E
		  [*] WebTitle: http://192.168.0.3:8080/login;jsessionid=D5E2531BBF330B8D45CBB786A3A86E4E code:200 len:2608   title:Login Page
		  [*] WebTitle: http://192.168.0.1:8080/login;jsessionid=A8649DFCD09C896D5F387AE8F916D35F code:200 len:2608   title:Login Page
		  [*] WebTitle: http://192.168.0.2        code:200 len:59431  title:W3School教程系统 | 打造专一的web在线教程系统
		  [*] WebTitle: http://192.168.0.1        code:200 len:59431  title:W3School教程系统 | 打造专一的web在线教程系统
		  [*] WebTitle: http://192.168.0.4        code:200 len:8351   title:博客首页
		  [+] http://192.168.0.1:8080/ poc-yaml-shiro-key [{mode cbc} {key kPH+bIxk5D2deZiIxcaaaA==}]
		  [+] http://192.168.0.3:8080/ poc-yaml-shiro-key [{key kPH+bIxk5D2deZiIxcaaaA==} {mode cbc}]
		  [+] http://192.168.0.4 poc-yaml-thinkphp5023-method-rce poc1
		  
		  ```
	- 打192.168.0.1:8080的shiro即可拿到第7个flag
	- 提权root拿到第8个flag
- # 9&10
	- 打192.168.0.4的thinkphp 5
	- 读网站目录和数据库的flag
	-