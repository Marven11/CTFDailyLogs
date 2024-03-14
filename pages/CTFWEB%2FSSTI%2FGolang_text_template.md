# 介绍
	- go语言自带模板渲染引擎
	- https://pkg.go.dev/text/template#example-Template
	- 可以使用的函数和变量有限，渲染时需要传入一个struct提供渲染所需的数据（用户名等）
	-
- # payload
	- 打印当前struct中的所有内容：`{{.}}`