- ```php
  $a=new DirectoryIterator("glob:///*");
  foreach($a as $f){
  	echo $f.' ';
  }
  ```
-