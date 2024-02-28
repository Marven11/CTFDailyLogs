tags:: CTFWEB/PHP/反混淆

- # 思路
	- 反混淆PHP获得webshell的密码，使用蚁剑的disabled_function绕过插件获得新的webshell`.antproxy.php`，用相同的密码连上并在根目录读取flag
- # php反混淆
	- http://www.zhaoyuanma.com/phpjm.html
- # 使用蚁剑
	- 使用插件中的LD_PRELOAD功能，保持默认设置攻击即可
-