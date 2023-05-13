- # cookie
	- 某些时候 cookie 可能是突破口
	- 信息泄漏
	- [[CTF/数据收集]]
- # header
	- Client-IP, X-Forwarded-For 和 X-Forward-For 的使用
		- Client-IP: 127.0.0.1
		- 类似的header
			- ```
			  X-Forwarded-For: 127.0.0.1
			  X-Forwarded: 127.0.0.1
			  Forwarded-For: 127.0.0.1
			  Forwarded: 127.0.0.1
			  X-Forwarded-Host: 127.0.0.1
			  X-remote-IP: 127.0.0.1
			  X-remote-addr: 127.0.0.1
			  True-Client-IP: 127.0.0.1
			  X-Client-IP: 127.0.0.1
			  Client-IP: 127.0.0.1
			  X-Real-IP: 127.0.0.1
			  Ali-CDN-Real-IP: 127.0.0.1
			  Cdn-Src-Ip: 127.0.0.1
			  Cdn-Real-Ip: 127.0.0.1
			  CF-Connecting-IP: 127.0.0.1
			  X-Cluster-Client-IP: 127.0.0.1
			  WL-Proxy-Client-IP: 127.0.0.1
			  Proxy-Client-IP: 127.0.0.1
			  Fastly-Client-Ip: 127.0.0.1
			  True-Client-Ip: 127.0.0.1
			  ```
	- Referer
		- Referer: http://www.google.com
- # 301跳转
	- 有时301跳转带有body而浏览器不会显示
- # robots.txt
	- 记得检查
- # 备份文件
	- ```python
	  suffixs = [
	    "tar",
	    "tar.gz",
	    "zip",
	    "rar",
	  ]
	  filenames = [
	    "web",
	    "website",
	    "backup",
	    "back",
	    "www",
	    "wwwroot",
	    "temp",
	  ]
	  ```
	- www.zip
	- index.php.swp
	- `.DS_Store`MacOS 的文件夹描述文件
- # Log
	- /var/log/nginx/access.log
- # CRLF 注入
	- 可以直接注入另一个请求或者另一个Header
	- ```php
	  'user_agent'=>"test\r\nCookie: PHPSESSID=0mrbj32r0ptfcnf7l6kmmgh1c4"
	  ```
- # 发送数组
	- `b[1]=T&b[0]=C&b[2]=F`
- # url 编码
	- 变量名也可以进行 url 编码
- # HTTP的绕过
  id:: 63076ac4-9cae-4a60-a951-c063f877f29b
	- ## 参数污染绕过
	  id:: 630453de-b4ad-43cf-91a5-0d25d8bd4768
		- ```
		  ?id=1&id=2&id=exp
		  asp.net+iis:id=1,2,exp
		  asp+iis:id=1,2,exp
		  php+apache:id=exp
		  ```
	- ## 解析绕过
		- id:: 63076ab9-2a45-48d9-9810-24530a8ed07e
		  ```
		  Iis5.0-6.0解析漏洞：
		  
		  “.asp --> /xx.asp/xx.jpg//.asp，.asa目录下的文件都解析成asp文件.asp --> xx.asp;.jpg//服务器默认不解析;号后面的内容”
		  
		  Iis7.5解析漏洞(php.ini开启fix_pathinfo)：
		  
		  “.php --> /xx.jpg //上传.jpg一句话，访问时后面加上/xx.php”
		  
		  apache解析漏洞：
		  
		  “.php --> /test.php.php123//从右往左，能别的后缀开始解析”
		  
		  nginx解析漏洞(php.ini开启fix_pathinfo)：
		  
		  “.php --> xxx.jpg%00.php//Nginx <8.03 空字节代码执行漏洞”
		  ```
	- ## Content-Type绕过
	- 有的waf识别到Content-Type类型multipart/form-data后，会将它认为是文件上传请求，从而不检测其他种类攻击只检测文件上传，导致被绕过。
	- “application/x-www-form-urlencodedè multipart/form-data”
	- ## HTTP请求方式绕过
	- waf在对危险字符进行检测的时候，分别为post请求和get请求设定了不同的匹配规则，请求被拦截，变换请求方式有几率能绕过检测。
	- 另外，云锁/安全狗安装后默认状态对post请求检测力度较小，可通过变换请求方式绕过。
-