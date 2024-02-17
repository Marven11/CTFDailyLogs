tags:: RCE

- # 执行PHP语句
	- `eval()`
		- 比如说`eval('echo 123;');`
		- 可以执行多条命令
	- `assert()`
		- 在低版本中是一个函数，高版本中和echo和eval一样是一个语法结构
	- `${}`/`${phpinfo()};`
	- `call_user_func`
		- 调用其他函数
		- 示例： `call_user_func('system', 'ls');`
	- `create_function`
		- 利用create_function立即执行命令而不是创建函数： ((64b5744d-30c9-4206-b43f-5e8499a08051))
- # 命令执行的方式
  id:: 64b5744d-4fd5-418b-b3a5-b89064d6ffdd
	- 反引号
		- 其中可以使用`$`包含变量
	- 执行shell指令
		- `exec()`
		- `shell_exec()`
		- `system()`
		- `passthru()`
		- `proc_open()`
- # 无参数
	- [[CTFWEB/PHP/无参数RCE]]
- # 绕过
	- [[CTFWEB/PHP/RCE的WAF绕过]]
-