tags:: CTFWEB/Python

- 一个python的http server框架
- # 模板
	- 模板文件放在`views/`文件夹中，文件名为`.html`
- # Cookie
	- 保存app传入的对象，序列化并签名后放在用户的cookie中
	- 读取时会使用 [[CTFWEB/Python/Pickle]]反序列化
	- [题目参考](((644b83b7-1c5b-4abb-9329-42c4f0b7fa44)))