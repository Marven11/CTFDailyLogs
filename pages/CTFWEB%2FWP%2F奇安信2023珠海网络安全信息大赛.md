# Reverse
	- ticket
		- [原题](https://lewiserii.github.io/wp/%E5%A5%87%E5%AE%89%E4%BF%A1CTF%E5%9F%BA%E7%A1%80%E5%9F%B9%E8%AE%AD%E9%A2%98%E7%9B%AEwp.html#%E4%B8%80%E5%BC%A0%E6%9D%A5%E8%87%AA%E5%A4%8F%E5%A4%A9%E7%9A%84%E8%BD%A6%E7%A5%A8)直接抄
		- 大恢复术
			- ```python
			  with open("./选手下载附件/exp.pyc", "rb") as f:
			      b = f.read()
			  
			  b = b[:7] + b"\x00\x00\x00\x00" + b[7:]
			  with open("./exp.pyc", "wb") as f:
			      f.write(b)
			  
			  ```
		- 然后直接用pycdc反编译，运行即可
	- wow
		- [原题](https://lewiserii.github.io/wp/%E5%A5%87%E5%AE%89%E4%BF%A1CTF%E5%9F%BA%E7%A1%80%E5%9F%B9%E8%AE%AD%E9%A2%98%E7%9B%AEwp.html#%E5%95%8A%EF%BC%9F)直接抄
		- ```python
		  v1 = [42, 43, 32, 23, 85, 5, 15, 8, 22, 7, 7, 68, 14, 5, 15, 17]
		  key = "anylab"
		  for x in range(16):
		      print(chr((v1[x] & 255) ^ (ord(key[x % 6]) & 255)), end="")
		  ```
	- static link
		- [原题](https://primelyw.github.io/2020/04/15/2018%E7%BD%91%E9%BC%8E%E6%9D%AF/)直接抄
		- ```ruby
		  #/bin/ruby
		  key=[ 0x66, 0x0A, 0x07, 0x0B, 0x1D, 0x08, 0x51, 0x38, 0x1F, 0x5C,  0x14, 0x38, 0x30, 0x0A, 0x1A, 0x28, 0x39, 0x59, 0x0C, 0x24,  0x24, 0x22, 0x01, 0x1F, 0x1E, 0x73, 0x1D, 0x3A, 0x08, 0x05, 0x15, 0x0A]
		  s=''
		  i=1 
		  until i==32 do (0..i-1).each do |k| key[k+i] ^= key[k]  end ; i <<= 1 end 
		  
		  key.each do |i| s<<i.chr end
		  
		  puts s
		  
		  ```
	- protect eyes
		- [原题](https://www.52pojie.cn/thread-605059-1-1.html)直接抄
		- `flag{sssn-trtk-tcea-akJr}`
	- jeb
		- flag中间的部分就是："Tenshine"字符串进行md5后的奇数号字符
- # Crypto
	- LCG
		- [原题](https://www.ctfiot.com/77157.html)直接抄
	- related
		- [原题](http://note.shenghuo2.top/01RSA%E5%9F%BA%E7%A1%80%E7%AF%87/P19.Franklin-Reiter%E6%94%BB%E5%87%BB%E8%84%9A%E6%9C%AC/#_3)直接抄
	- Simple Encryption
		- [原题](https://skysec.top/2018/09/24/2018%E5%AE%89%E6%81%92%E6%9D%AF-9%E6%9C%88%E6%9C%88%E8%B5%9BWriteup/#Crypto2)直接抄
		- ```python
		  output = "u6WHK2bnAsvTP/lPagu7c/K3la0mrveKrXryBPF/LKFE2HYgRNLGzr1J1yObUapw"
		  key = md5(sha1("flag"))
		  for result in range(0xB18E):
		      passwd = generate_passwd(key.decode("hex"),result)
		      r = decrypt(output.decode("base64"), passwd)
		      if 'flag' in r:
		          print r
		  ```
		- `flag{552d3a0e567542d99694c4d61d1a652e}`
	- super password
		- 所谓加密技巧就是将明文和对应的secret拼起来进行md5，flag就是secret
		- 提示secret中间时一个十位数的数字，猜`1234567890`
		- 验证：
			- ```python
			  print(enc_dec.md5_encode(r"{FLAG:1234567890}p5Zg6LtD"))
			  # f09ebdb2bb9f5eb4fbd12aad96e1e929
			  ```
	- script
		- md5爆破