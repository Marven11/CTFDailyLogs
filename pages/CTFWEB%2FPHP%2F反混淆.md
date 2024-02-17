# 工具
	- http://www.zhaoyuanma.com/phpjm.html
- # 函数
	- 用这个函数，将原文件中的eval都改成evall，然后再执行就可以看到eval执行的内容
	- ```php
	  function evall($s){
	      echo $s;
	      echo "\n---\n";
	      eval(str_replace("eval(", "evall(", $s));
	  }
	  ```