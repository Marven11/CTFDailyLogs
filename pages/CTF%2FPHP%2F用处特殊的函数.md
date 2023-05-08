- ## getallheaders
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
## preg_match()
	- `$` 会忽略末尾的 `%0a`
## base64_decode()
	- `base64_decode($s)` 会忽略不合法的字符，但 `base64_decode($s, true)` 不会
## basename()
id:: 64572c54-7814-4588-9fec-96dd86e3ab23
	- ```
	  basename(urldecode("config.php/%80")) === "config.php"
	  ```
## json_decode()
	- 会转义形如 `\u0061` 的部分
## create_function()
	- 注入，代码将会立即执行：
	  
	  ```php
	  create_function('','}echo 123;//')
	  ```
	  
	  生成的函数名为 `%00lambda_%d` ， 其中 `%d` 为一个递增的数字，在新建线程时为 1
## parse_url()
	- 在解析特殊地址时会发生错误
- ## finfo_file() 和 getimagesize()
	- 传入的文件为一个 png 图片的前 20 个字节时，这两个文件会给出不同的结果
	- ((6418600e-2e09-453d-b103-91bd7c6a7041))
- ## get_defined_vars()
	- RT
- ## mt_random()
	- 可以用 php_mt_seed 根据生成的随机数反推出种子
- ## ob_get_contents()
	- 会截获输出的内容
	  
	  可以通过 exit()在截获前停止
- ## intval()
	- 处理数组时会返回 0 或 1
## filter_var()
	- FILTER_VALIDATE_EMAIL绕过
		- RFC 3696规定，邮箱地址分为local part和domain part两部分。local part中包含特殊字符，需要如下处理：
			- 将特殊字符用`\`转义，如`Joe\'Blow@example.com`
			- 或将local part包裹在双引号中，如`"Joe'Blow"@example.com`
			- local part长度不超过64个字符
		- 虽然PHP没有完全按照RFC 3696进行检测，但支持上述第2种写法。所以，我们可以利用之绕过FILTER_VALIDATE_EMAIL的检测。
		- 也就是说`"@aaa'"@example.com`这个邮箱是合法的
		- [参考](https://www.leavesongs.com/PENETRATION/some-tricks-of-attacking-lnmp-web-application.html)