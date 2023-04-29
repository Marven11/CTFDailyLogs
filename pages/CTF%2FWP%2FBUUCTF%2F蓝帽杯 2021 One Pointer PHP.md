- # 源代码
	- ```php
	  <?php
	  include "user.php";
	  if($user=unserialize($_COOKIE["data"])){
	  	$count[++$user->count]=1;
	  	if($count[]=1){ // 赋值失败会返回false
	  		$user->count+=1;
	  		setcookie("data",serialize($user));
	  	}else{
	  		eval($_GET["backdoor"]);
	  	}
	  }else{
	  	$user=new User;
	  	$user->count=1;
	  	setcookie("data",serialize($user));
	  }
	  ?>
	  ```
- # 思路
	- RCE
		- PHP的整数最大为`9223372036854775807`
		- 如果我们让`$user -> count`为整数最大值减一，那if中的语句就会因为没有更大的值二返回false
	- 提权
		- `find / -perm -u=s -type f 2>/dev/null`发现php有suid文件，可以通过绕过php设置的限制读取flag
		- 绕过php的限制： ((63ea6ed9-e1e8-447f-bd58-356b4589061c))