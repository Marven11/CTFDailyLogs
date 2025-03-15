- ```php
  function filter($str) {
      $filter = "/ |\*|#|;|,|is|union|like|regexp|for|and|or|file|--|\||`|&|" . urldecode('%09') . "|" . urldecode("%0a") . "|" . urldecode("%0b") . "|" . urldecode('%0c') . "|" . urldecode('%0d') . "|" . urldecode('%a0') . "/i";
      if (preg_match($filter, $str)) {
          die("you can't input this illegal char!");
      }
      return $str;
  }
  ```
-