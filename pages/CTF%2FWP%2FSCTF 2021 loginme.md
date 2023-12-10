tags:: [[CTF/Golang]], [[CTF/Gin]]

- # 思路
	- 通过header绕过`c.ClientIP`检测，然后``{{.}}``SSTI
	- 模板引擎大概是[[CTF/SSTI/Golang_text_template]]