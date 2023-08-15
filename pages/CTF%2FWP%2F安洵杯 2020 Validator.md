tags:: CTF/JS/原型链污染

- [参考](https://xz.aliyun.com/t/8581#toc-2)
- # 思路
	- 使用express-validator依赖的lodash <4.17.17 的这个[漏洞](https://security.snyk.io/vuln/SNYK-JS-LODASH-608086)污染`system_open`绕过检测
- # lodash <4.17.17 原型链污染
	- lodash的set和get函数实习细节
		- 作用：将第二个参数解析为键的路径读取/写入到第一个参数（一个对象）之中
		- 如果第二个参数存在于第一个参数中则不将其转换成路径
	- express-validator中select-fields的实现细节
		- 如果key中存在`.`则在两旁加上`["`和`"]`转义
		  id:: 64ce6313-a5e1-47d8-96ca-7187f2a562c4
	- express-validator调用lodash的set时的细节
		- 其使用lodash向req中写入数据，使用到四个变量：
			- req
			- location: body等req中的键
			- path: 具体向req[location]写入数据的路径，如`"a.b.c"`
			- newValue: 数据
		- 如果`path == ""`则直接向`req[location]`中写入数据，否则以path为路径向`req[location]`写入数据
	- payload：
		- ```python
		  r = base_post(
		      url,
		      json={
		          "a": {"__proto__": {"system_open": "yes"}},
		          'a"].__proto__["system_open': 222,
		      },
		      acceptable_code=None,
		  )
		  print(r.text)
		  ```
		- 工作原理
			- 首先payload的键``a"].__proto__["system_open``被[express-validator](((64ce6313-a5e1-47d8-96ca-7187f2a562c4)))变换为`["a"].__proto__["system_open"]`
			- 然后lodash根据`["a"].__proto__["system_open"]`去寻找值，得到`instance.value`为`yes`
			- 再然后express-validator再使用lodash获取路径`["a"].__proto__["system_open"]`的值，因为包含``__proto__``所以只能找到undefined
			- 最后express-validator将`instance.value`通过runner转换为newValue, 值仍然是`yes`，并调用lodash将yes放进路径`["a"].__proto__["system_open"]`中
	-