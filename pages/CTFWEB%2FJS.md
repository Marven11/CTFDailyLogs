# 原型链污染
	- [[CTFWEB/JS/原型链污染]]
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
	- [[CTFWEB/JWT]]
- # 绕过
	- [[CTFWEB/JS/WAF绕过]]
- # 沙盒逃逸
	- [[CTFWEB/JS/沙盒逃逸]]
- # 注释
	- JS 也可以使用 html 的注释`<!-- -->`
- # 解混淆
	- https://dev-coco.github.io/Online-Tools/JavaScript-Deobfuscator.html
	- https://blog.csdn.net/weixin_43411585/article/details/123437953
	- https://blog.csdn.net/freeking101/article/details/121668637
- # unicode的解析问题
	- ```js
	  req.session.guestǃ=true // 感叹号是unicode里的，是属性名的一部分
	  ```
- # js.map读取
	- .js.map文件里存放着网站的API路由
	- https://xz.aliyun.com/t/15315?time__1311=GqjxnD2ieYqqlxGghDy0Q7i%3Dait2Q43x#toc-9