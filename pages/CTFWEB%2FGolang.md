- [[Golang]]
- # 参考
	- https://bycsec.top/2021/02/07/golang%E7%9A%84%E4%B8%80%E4%BA%9B%E5%AE%89%E5%85%A8%E9%97%AE%E9%A2%98/
- # 整数反转
	- `strconv.Atoi`存在整数反转问题
		- ```golang
		  v , _ := strconv.Atoi("4294967377")
		  s  := int32(v)
		  fmt.Println(s)
		  // 81
		  ```
- 随机数的种子默认为1
- go.http CRLF
-