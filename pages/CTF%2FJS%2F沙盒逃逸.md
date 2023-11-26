# shadowRealm
	- [参考](https://test-cuycc6s9lprw.feishu.cn/docx/T7budbiSWoTNd4xQGVicHL1Vnpf)
	- 代码执行过程
		- 首先是[`ShadowRealm.prototype.evaluate`函数](https://tc39.es/proposal-shadowrealm/#sec-shadowrealm.prototype.evaluate)被调用
		- 然后将代码传给`PerformShadowRealmEval`[函数](https://tc39.es/proposal-shadowrealm/#sec-performshadowrealmeval)，其中创建了一个新的context并使用`Evaluate`在这个context上执行代码，代码会运行在一个标准的ecma262环境中
		- 最后发现此环境支持[import](https://tc39.es/ecma262/#sec-import-calls)
	- ```js
	  import('child_process').then(m=>m.execSync('/readflag > /app/asserts/flag'));
	  1;
	  ```
- # VM2
	- ((64b57457-20a6-4068-818f-95cce18c7fdb))