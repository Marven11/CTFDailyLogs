# upload()
	- [RoarCTF 2019]Simple Upload think
	  
	  PHP 里的 upload()函数在不传参的情况下是批量上传的
- # 读源码
	- 重点关注app/controller文件夹
		- 里面的每个类中的每个函数都可以通过`/类名/函数名`这个URI访问
	-