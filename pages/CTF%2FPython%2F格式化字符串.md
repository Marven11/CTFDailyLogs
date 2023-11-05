# 基础
	- python中格式化字符串的语法为
		- `"{}".format(114)`
		- `"{0}".format(114)`
		- `"{num}".format(num=114)`
	- python官方文档给出的语法
		- ```text
		  replacement_field ::=  "{" [field_name] ["!" conversion] [":" format_spec] "}"
		  field_name        ::=  arg_name ("." attribute_name | "[" element_index "]")*
		  arg_name          ::=  [identifier | digit+]
		  attribute_name    ::=  identifier
		  element_index     ::=  digit+ | index_string
		  index_string      ::=  <any source character except "]"> +
		  conversion        ::=  "r" | "s" | "a"
		  format_spec       ::=  <described in the next section>
		  ```
		- 可以看出其支持取属性和取item
- # 各类操作
	- 示例：
		- ```python
		  class A():
		      def __init__(self, o):
		          self.o = o
		      def __str__(self):
		          return str(self.o)
		  
		  s = "{0}".format(
		      A("111")
		  )
		  print(s)
		  ```
	- 取属性
		- `"{0.__dict__}"`
	- 取键
		- `"{0.__dict__[o]}"`
		- 这里`0.__dict__`是一个字典，取其的`o`键
	-