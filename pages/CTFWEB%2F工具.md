## 常用工具
	- ```text
	  c-jwt-cracker
	  ctf-png-size-solver
	  dirsearch
	  fuzzDicts
	  GitHack
	  Gopherus
	  HashPump
	  Lazymux
	  lazymux.conf
	  log4j2-exploits
	  Packer-Fuzzer
	  PHP_INCLUDE_TO_SHELL_CHAR_DICT
	  pker
	  PNG-IDAT-Payload-Generator
	  Python-dsstore
	  RsaCtfTool
	  secret_shell
	  sqlmap-dev
	  svnExploit
	  test2.py
	  test.py
	  volatility3
	  ```
- ## Gopherus
  id:: 62f14125-43fc-4da5-b83f-4d56aa1d2157
	- Gopher协议可以在MySQL没有密码的时候控制MySQL
	- 只需要用户名，生成`gopher://`协议的SQL控制Payload
	- 参见：
		- [[CTFWEB/WP/CTFShow/web308]]
		- [[CTFWEB/PHP/伪协议]]
		- ((62f27746-a352-41d4-b76d-63fc1f7c9e4f))
- ## dirsearch
	- 查询网站隐藏路径
- ## John the Ripper
	- 解密/etc/shadow
- ## slowloris
	- DDoS 爆破工具
- ## arjun
	- 参数爆破工具
- ## hping
	- hping -S --flood -V [IP]
- ## zmap
	- 假设路由器的 MAC 地址是`48:73:97:31:10:01`
	- `sudo zmap -p 8080 192.168.0.0/16 --gateway-mac=48:73:97:31:10:01 -o results.csv`
- ## HashPump
  id:: 62f4b96a-c4f5-425c-997f-bafd13e8a5c0
	- 已知`$known_md5 = md5($secret + "asdfasdf")`
	  id:: 62f4b97a-0660-44b1-82d3-8321bf2fa4bc
	- 求`$known_md5 == md5($secret + "asdfxxxxxxxxxxxxxxxxxxxxxxxxxxxxasdf")`中的 xxxxxxxxxx...是什么
- ## amass
	- 域名爆破工具