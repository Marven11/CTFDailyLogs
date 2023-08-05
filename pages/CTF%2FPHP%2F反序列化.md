- [PHP 内置类的应用](https://www.cnblogs.com/iamstudy/articles/unserialize_in_php_inner_class.html#_label1_0)
  
  ```php
  O:1:"A":1:{
      s:4:"aaaa";
      O:1:"B":1:{
          s:4:"bbbb";
          s:5:"/flag";
      }
  }
  ```
- 使用大写 S 可以自动转义`\61`等符号
- protected 变量
	- `S:7:"\00*\00aaaa";`
- 可以试试通过改变字符串长度实现字符逃逸
- unserialize()会把多余的字符串当垃圾处理，在花括号内的就是正确的，花括号后面的就都被扔掉
- `O:1:"A":1:`也可以用`O:+1:"A":1:`和`O:1:"A":+1:`代替
- 对象内可以用指针：`R:2;`指向第二个元素
  id:: 6478aaed-0cfc-41e3-a12e-55de79af3104
- `__wakeup`绕过：修改类的键数量
  collapsed:: true
	- ```php
	  <?php
	  
	  class Test {
	      public $a;
	      function __wakeup() {
	          echo "wakeup";
	      }
	      function __destruct() {
	          echo "destruct";
	      }
	  }
	  
	  $t = new Test();
	  $t -> a = "114514";
	  $s = serialize($t);
	  $s = str_replace('"Test":1', '"Test":2', $s);
	  echo serialize($t);
	  $t2 = unserialize($s);
	  var_dump($t2);
	  ?>
	  
	  ```
		-
- 快速反序列化：参考[[CTF/WP/数博会 贵阳杯]]
- 自定义序列化对象在序列化后以C开头，[例题](((64b2aacc-598b-4be7-b7d4-48230820277e)))
- 可以使用[PHPGGC](https://github.com/ambionics/phpggc)自动构建ThinkPHP, Lavaral等的反序列化payload
-