# 介绍
	- PHP在很多时候（比如对比两个变量时）会自动转换变量的类型。
	- 比如：`114 == "114"`为true
- # 数组
	- 在本来为字符串的位置传入一个数组
	- `array(1 => T, 2 => F, 0 => C) == array(C, T, F)`但`array(1 => T, 2 => F, 0 => C) !== array(C, T, F)`
- # 两个不同的对象转换成字符串后相等
  id:: 62efc5c6-a0d5-4eab-906e-0db03366b1e9
	- `$a = Error("asdf"); $b = Error("asdf", 1) //(a和b必须写在同一行)`
	- 满足 `($a != $b) && (md5($a) === md5($b)) && (sha1($a)=== sha1($b))`
- # intval很大的“三位”数字
	- 1e9
	- `intval('1e9') > 100`是true
- # MD5绕过
	- [[CTF/MD5]]