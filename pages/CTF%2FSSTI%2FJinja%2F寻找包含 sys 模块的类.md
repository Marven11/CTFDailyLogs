- ```python
  import flask # 先import要搜索的模块
  import time
  import sys
  num = 0
  l = [].__class__.__mro__[1].__subclasses__()
  
  for k, item in enumerate(l):
    try:
         if "sys" in item.__init__.__globals__:
             print(k,item)
    except:
        pass
  ```