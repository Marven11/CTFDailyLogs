# 属性
	- ```
	  __dict__ 保存类实例或对象实例的属性变量键值对字典
	  __class__  返回类型所属的对象
	  __mro__    返回一个包含对象所继承的基类元组，方法在解析时按照元组的顺序解析。
	  __bases__   返回该对象所继承的基类
	  // __base__和__mro__都是用来寻找基类的
	  
	  __subclasses__   每个新类都保留了子类的引用，这个方法返回一个类中仍然可用的的引用的列表
	  __init__  类的初始化方法
	  __globals__  对包含函数全局变量的字典的引用
	  ```
- # 对象
	- ```
	  config
	  self
	  url_for, g, request, namespace, lipsum, range, session, dict, get_flashed_messages, cycler, joiner, config等 #
	  ```
- # 全局对象
	- [官方文档](https://jinja.palletsprojects.com/en/3.1.x/templates/#builtin-globals)