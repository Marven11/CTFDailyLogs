title:: CTFWEB/WP/CTFShow/web386

- # 施工中
	- /page.php 有限制的文件读取
	- /install/index.php
		- 提示存在文件`lock.dat`
		- ```php
		  error_reporting(0);
		  $dbhost ="127.0.0.1";
		  $dbuser = "root";
		  $dbpwd = "root";
		  $dbname = "ctfshow";
		  $charName = "utf-8"; 
		  $file = 'lock.dat';
		  if(file_exists($file)){
		  	die('lock.dat存在，你已经安装过了，请勿重复安装');
		  }
		  echo '请务必在安装成功后删除本文件';
		  echo '<br>需要重新安装请访问install/?install,管理员密码将重置为默认密码';
		  if(isset($_GET['install'])){
		  	$conn = new mysqli($dbhost,$dbuser,$dbpwd,$dbname);
		  	if(mysqli_connect_errno()){
		  	 	die(json_encode(array(mysqli_connect_error())));
		  	}
		  	$sql = "update admin_user set username='admin',password='admin888' where username='admin';";
		  	$result = $conn->query($sql);
		  	echo '<br>安装成功，请立即删除本文件';
		  
		  }
		  ```
	- 查看主页发现一个秘密路径`alsckdfy/layui/css/tree.css`
- # WP
	- 查看秘密路径
	- 发现`alsckdfy/check.php`，使用`page.php`读取发现flag
-