## getallheaders
	- 获取 header 数组
- ## include()
  id:: 62f4af50-43d4-4ada-bd3e-d102d3c6a74d
	- ```
	  include("asdfasdfasdf/../flag.php")
	  //即使asdfasdfasdf文件夹不存在也可以读取flag.php
	  ```
	- include 支持伪协议，也就是说可以填入 `data://text/plain,<?var_dump(123)?>` 之类的字符串
	- 可以在 ua 中添加代码，然后 include /var/log/nginx/access.log
	- inlcude 可以被某个伪协议弄崩溃
		- {{embed ((636665b7-767d-440c-8482-bb5e1dc0933c))}}
	- 还可以这样：
	- ```
	  <?php
	  include$_GET['a']
	  ?>
	  ```
- ## is_file()
	- 该函数检测的时候是以绝对路径来检测的
	- 如果其中一级目录不存在则返回false
- ## preg_match()
	- `$` 会忽略末尾的 `%0a`
	- [[CTFWEB/PHP/正则匹配回溯次数绕过]]
- ## base64_decode()
	- `base64_decode($s)` 会忽略不合法的字符，但 `base64_decode($s, true)` 不会
- ## basename()
  id:: 64572c54-7814-4588-9fec-96dd86e3ab23
	- ```
	  basename(urldecode("config.php/%80")) === "config.php"
	  ```
- ## json_decode()
	- 会转义形如 `\u0061` 的部分，可以用来绕过WAF
- ## create_function()
  id:: 64b5744d-30c9-4206-b43f-5e8499a08051
	- 注入，代码将会立即执行：
	  
	  ```php
	  create_function('','}echo 123;//')
	  ```
	  
	  生成的函数名为 `%00lambda_%d` ， 其中 `%d` 为一个递增的数字，在新建线程时为 1
- ## parse_url()
  id:: 645dbc96-bb3a-4052-8764-5d425ac10179
	- 在解析特殊地址时会发生错误
	- ((66adf55c-36a2-468b-81db-ee02adcdce2d))
- ## finfo_file() 和 getimagesize()
	- 传入的文件为一个 png 图片的前 20 个字节时，这两个文件会给出不同的结果
	- ((6418600e-2e09-453d-b103-91bd7c6a7041))
- ## get_defined_vars()
	- 可以拿到当前定义的所有变量，做无参数RCE时很常用
- ## mt_random()
	- 可以用 php_mt_seed 根据生成的随机数反推出种子
- ## ob_get_contents()
	- 会截获输出的内容
	  可以通过 exit()在截获前停止
- ## intval()
	- 处理数组时会返回 0 或 1
- ## filter_var()
	- FILTER_VALIDATE_EMAIL绕过
		- RFC 3696规定，邮箱地址分为local part和domain part两部分。local part中包含特殊字符，需要如下处理：
			- 将特殊字符用`\`转义，如`Joe\'Blow@example.com`
			- 或将local part包裹在双引号中，如`"Joe'Blow"@example.com`
			- local part长度不超过64个字符
		- 虽然PHP没有完全按照RFC 3696进行检测，但支持上述第2种写法。所以，我们可以利用之绕过FILTER_VALIDATE_EMAIL的检测。
		- 也就是说`"@aaa'"@example.com`这个邮箱是合法的
		- [参考](https://www.leavesongs.com/PENETRATION/some-tricks-of-attacking-lnmp-web-application.html)
- ## realpath()
	- 会将软链接解析成真实的路径，但是不会解析伪协议，所以会将`file:///flag.txt`解析成`file:`文件夹下的`flag.txt`
- ## unlink
	- unlink不会使用`php_stream_open_wrapper_ex`正规化路径，所以不会删除形如`a.txt/.`的路径
	- 避免文件被unlink删除，可以这样：
		- ```shell
		  ➜  ~ touch a.txt
		  ➜  ~ php 
		  <?php
		  unlink("a.txt/.");
		  ?>
		  PHP Warning:  unlink(a.txt/.): Not a directory in Standard input code on line 2
		  
		  ```
	- ```text
	      查看php源码，其实我们能发现，php读取、写入文件，都会调用php_stream_open_wrapper_ex来打开流，而判断文件存在、重命名、删除文件等操作则无需打开文件流。  
	  
	      我们跟一跟php_stream_open_wrapper_ex就会发现，其实最后会使用tsrm_realpath函数来将filename给标准化成一个绝对路径。而文件删除等操作则不会，这就是二者的区别。
	  
	      所以，如果我们传入的是文件名中包含一个不存在的路径，写入的时候因为会处理掉“../”等相对路径，所以不会出错；判断、删除的时候因为不会处理，所以就会出现“No such file or directory”的错误。
	  ```