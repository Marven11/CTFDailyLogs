- > 来看看别人的新生赛，做了三题，每题都出得挺恶心的。。。
- # 伪装者
	- HTTP基础+伪协议，秒了
	- `http://101.132.112.252:32243/img?url=file:///flag`
- # Ezweb
	- sql注入，但是加上单引号不会出现任何错误提示（甚至还会告诉你提交成功），只有继续投票后才能看到票数并没有增加
- # EZphp
	- 进题目，注释提示post `guess`，header提示`which rand()`, cookie提示seed的值
	- 爆破得到使用seed+`rand`取多个随机值，上传第7个，得到特殊header: `'Hint': 'Go to Unablet0guess.php'`
	- 然后打反序列化和[[CTFWEB/Shell/WAF绕过]]就完事了
	- 纯恶心人的题目，不知道是谁出的这题让人猜需要提交第几个随机数，毫无实战意义，简直就是为了出题而出题
	-