tags:: CTFWEB/PHP/文件上传和写入, CTFWEB/PHP/WAF绕过

- # WP
- ```php
  <?php
    $file = $_GET['file'];
    $content = $_POST['content'];
    $file = str_replace("php", "???", $file);
    $file = str_replace("data", "???", $file);
    $file = str_replace(":", "???", $file);
    $file = str_replace(".", "???", $file);
    file_put_contents(urldecode($file), "<?php die('大佬别秀了');?>".$content); 
  ?>
  ```
- 可以利用urldecode编码关键词，从而实现绕过
  
  然后可以使用伪协议`php://filter/write=string.rot13/resource=test.php`破坏die函数的执行。