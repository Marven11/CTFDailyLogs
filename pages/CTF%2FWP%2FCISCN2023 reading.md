# 思路
	- 通过`/proc/self/maps`目录读取内存分块
	- 然后根据分块的起始地址和结束地址读`/proc/self/mem`找key
- # 脚本
	-