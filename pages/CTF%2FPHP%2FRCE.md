# 命令执行的方式
id:: 64b5744d-4fd5-418b-b3a5-b89064d6ffdd
	- 反引号
		- 其中可以使用`$`包含变量
	- `eval()`/执行**php 语句**
	- `exec()`
	- `shell_exec()`
	- `system()`
	- `passthru()`
	- `proc_open()`
	- `assert()`
	- `${}`/`${phpinfo()};`
	- `call_user_func`
- # 无参数
	- [[CTF/PHP/无参数注入]]
- # 绕过
	- [[CTF/PHP/RCE的WAF绕过]]
-