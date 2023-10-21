# 相关函数
	- > 这些PHP函数可以用来读取文件
	- `file_get_contents`
	- `readgzfile`
	- `readfile`
	- `show_source`
	- `highlight_file`
	- `include`
	  id:: 62f4a85a-d984-422f-8e43-2f0abb915f91
	- `include_once`
	- `require`
	- `require_once`
	- `fopen`
	- `system`
		- 命令执行的函数在此不一一列举
		- `system('cat /flag')`
		- `system('cat /fla?')`
		- 如何应对： ((42eb4318-43ea-4079-963d-61943df49a8c))
- # 相关类
	- ```php
	  $a=new SplFileObject("/flag");
	  echo $a;
	  ```
- # 相关漏洞
	- `php -S`可能会造成源码泄漏
		- 例子： ((64f2a4b1-ae66-4b1e-9a95-744f30a923cb))