- # 工具
	- http://www.zhaoyuanma.com/phpjm.html
- # 函数
	- ```php
	  function evall($s){
	      echo $s;
	      echo "\n---\n";
	      eval(str_replace("eval(", "evall(", $s));
	  }
	  ```