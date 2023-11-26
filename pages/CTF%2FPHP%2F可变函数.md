# 根据字符串查找函数执行
	- ```php
	  # 像这样
	  $a="var_dump";
	  $a(123);
	  # 这样也可以用：
	  ("var_dump")("123");
	  ```
- # 调用类的函数
	- ```php
	  <?php
	  class C{
	      public function find() {
	          echo "what";
	      }
	  }
	  $c = new C;
	  ([$c, "find"])(); // 调用对象的成员函数
	  ("C::find")(); // 调用类的函数，此时没有$this变量
	  ?>
	  ```