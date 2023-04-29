- ## Linux杂项
	- /etc/shadow
		- 可以使用 John the Ripper 解密
		- 每行密码的格式：
		  
		  ```none
		  用户名：加密密码：最后一次修改时间：最小修改时间间隔：密码有效期：密码需要变更前的警告天数：密码过期后的宽限时间：账号失效时间：保留字段
		  ```
		- 其中加密密码的格式：
		  
		  ```none
		  $加密方式的ID$盐$散列加密后的密码
		  ```
		  
		  | 加密方式的 ID | 对应的加密方式                                                   |
		  | :------------ | :--------------------------------------------------------------- |
		  | 1             | MD5                                                              |
		  | 2a            | Blowfish(not in mainline glibc;added in some Linux distribution) |
		  | 5             | SHA256                                                           |
		  | 6             | SHA512                                                           |
	- find
		- `find / -name flag*`
	- 自动寻找flag
		- `find / -type f 2>/dev/null | xargs grep 'flag{' 2>/dev/null`
	- /etc/hosts 和 /proc/net/arp
		- 可能保存着当前节点所在网络的信息
	- history
		- add space to avoid adding command to history
### proc
	- /proc/self/cwd
		- 当前程序所在的目录
	- /proc/self/fd/3
		- 在文件被删除后仍然可以通过这个地址读取文件
	- /proc/self/root
		- 根目录
	- /proc/net/arp
		- ARP表