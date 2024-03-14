# 示例
	- ```python
	  s3cret="this is flag"
	  
	  codes='''
	  def waff():
	      def f():
	          yield g.gi_frame.f_back
	  
	      g = f()  #生成器
	      frame = next(g) #获取到生成器的栈帧对象
	      b = frame.f_back.f_back.f_globals['s3cret'] #返回并获取前一级栈帧的globals
	      return b
	  b=waff()
	  '''
	  locals={}
	  code = compile(codes, "test", "exec")
	  exec(code,locals)
	  print(locals["b"])
	  ```
	- 代码逻辑
		- 创建一个生成器并让生成器返回其栈帧的上一帧`frame`
		- 使用frame找到前两帧（也就是全局）并从中找到`s3cret`变量
		-
- # 参考
	- https://xz.aliyun.com/t/13635