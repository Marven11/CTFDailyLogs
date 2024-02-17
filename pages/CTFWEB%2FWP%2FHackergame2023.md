# 赛博井字棋
	- 手动构造请求覆盖对手的棋子。。。
- # JSON ⊂ YAML?
	- flag1: `{"a": 1e2}`
	- flag2: `{"a": 1, "a": 1}`
- # Git? Git!
	- ```shell
	  # cd .git/objects
	  ls */*|grep -v pack | awk -F/ '{print $1$2}'|xargs -n 1 -I {} git cat-file -p {} | grep flag
	  ```
- # HTTP 集邮册
	- `[200, 400, 404, 405, 505]`...搓不出来了
	- 200: dddd
	- 400: 改HTTP version
	- 404: dddd
	- 405: 改method
	- 505: 改version为`HTTP/2`
- # Docker for Everyone
	- `docker run -v /:/mnt alpine sh -c 'cat /mnt/dev/shm/flag'`
- 剩下的题目一股misc味。。。