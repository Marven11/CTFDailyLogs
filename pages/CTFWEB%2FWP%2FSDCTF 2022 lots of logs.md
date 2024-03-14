# 思路
	- 在`index.js`中发现log地址
	- 下载全部Log,使用脚本过滤发现入侵记录
	- 使用入侵记录中的密码登陆另一个后台得到flag
- # 过滤脚本
	- ```python
	  rules = [
	      "Welcome to LoggerOS 11.02.3 (GNU/Linux 4.7.8-23-generic)",
	      "john@logger:~# ./logger",
	      "",
	      re.compile(r"[A-Z][a-z]{2} \d{1,2}/\d{1,2} \d{1,2}:\d{1,2}:\d{1,2} START : \*{26} STARTING LOGGING \*{26}"),
	      re.compile(r"[A-Z][a-z]{2} \d{1,2}/\d{1,2} \d{1,2}:\d{1,2}:\d{1,2} LOG   : \d+ logs logged!"),
	      re.compile(r"[A-Z][a-z]{2} \d{1,2}/\d{1,2} \d{1,2}:\d{1,2}:\d{1,2} PROC  : Processing \d+ logs! "),
	      re.compile(r"[A-Z][a-z]{2} \d{1,2}/\d{1,2} \d{1,2}:\d{1,2}:\d{1,2} SELL  : Sold \d+ logs for \$\d+\.\d+ each! "),
	      re.compile(r"[A-Z][a-z]{2} \d{1,2}/\d{1,2} \d{1,2}:\d{1,2}:\d{1,2} PROF  : Profitted \$\d+.\d+! \(\$\d+.\d+ total\)"),
	      re.compile(r"[A-Z][a-z]{2} \d{1,2}/\d{1,2} \d{1,2}:\d{1,2}:\d{1,2} PROC  : Processing \d+ logs! \(\d+ logs still need processing\)"),
	      re.compile(r"[A-Z][a-z]{2} \d{1,2}/\d{1,2} \d{1,2}:\d{1,2}:\d{1,2} STOP  : \*{26} STOPPING LOGGING \*{26}"),
	      re.compile(r"[A-Z][a-z]{2} \d{1,2}/\d{1,2} \d{1,2}:\d{1,2}:\d{1,2} FIN   : Sold \d+ logs for a total of \$\d+ profit!"),
	  
	  ]
	  # matches = {i:[] for i in range(len(rules))}
	  found = False
	  
	  def match_rule(rule, line):
	      if isinstance(rule, str) and line == rule:
	          return True
	      elif isinstance(rule, re.Pattern) and rule.match(line):
	          return True
	      return False
	  
	  path = Path("logs/")
	  for file in os.listdir(path):
	      with open(path / file, "r", encoding="utf-8") as f:
	          for i, line in enumerate(iter(f.readline, "")):
	              line = line.removesuffix("\n")
	              found = not any(match_rule(rule, line) for rule in rules)
	              if found:
	                  print(file, i, line)
	                  break
	          if found:
	              break
	  ```