# 思路
	- 这一题除了普通的读flag之外还有一个骚操作
	- 可以使用`sed${IFS}-i${IFS}s/!preg/preg/g${IFS}index.php`直接修改index.php，去除preg_match前面的叹号
	-