tags:: [[CTFWEB/SQL]]

- # 题目
	- 一个文件管理器，可以上传文件，重命名文件和删除文件
	- 根据泄漏源码可知
		- 上传文件时后台会将文件名和后缀名分开保存
		- 重命名文件时会从数据库取出原文件名和后缀名，根据数据库中的后缀名来重命名
		-
- # 思路
	- 注意到
		- 文件上传处有后缀名检测，重命名处没有后缀名检测
		- 有全局`addslashes`，但可以SQL二次注入
	- 我们可以
		- 先用 ((63e524a4-ca11-4726-9b9f-023603e16984)) 创建一个文件名为`ghost.jpg`，后缀名为空的记录
		- 然后将webshell命名为`ghost.jpg`上传
			- 此时数据库中拥有两个与`ghost.jpg`有关的记录：
				- `filename="ghost.jpg", extension=""`，即我们需要的“幽灵记录”
				- `filename="ghost", extension=".jpg"`
		- 最后在重命名处输入`ghost.jpg`和`ghost.php`将webshell改回php后缀
- # SQL二次注入
  id:: 63e524a4-ca11-4726-9b9f-023603e16984
	- `rename.php`会将原文件名从数据库中取出，然后作为`oldname`再插入数据库
	- 我们只需要在正常的文件名后方插入`',extension='`即可清空此文件在数据库中的后缀名
	-
-