tags:: [[CTFWEB/Golang]], [[CTFWEB/Gin]]

- # 思路
	- 通过header绕过`c.ClientIP`检测，然后``{{.}}``SSTI
	- 模板引擎大概是[[CTFWEB/SSTI/Golang_text_template]]