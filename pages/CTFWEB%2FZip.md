# 内嵌软链接
	- ```
	  ln -s /var/www/html/index.php index
	  zip --symlinks index.zip index
	  ```
	- 实现用软链接写入目标上的任意文件
	  id:: 65d9fefc-9834-4cd9-bb87-ae06590683df
		- 需要两个压缩包
		- 压缩包1：创建到目标文件夹的软链接
		- 压缩包2：将文件解压到创建的软链接中
-