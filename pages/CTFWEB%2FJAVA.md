- [[CTFWEB/Log4j2]]
- # WAR包的常见结构
  id:: 641ae855-b000-4991-80f0-8172ecb29684
	- ```text
	  webapp.war 
	    |    index.jsp 
	    | 
	    |— images 
	    |— static
	    |— META-INF 
	    |— WEB-INF 
	            |   web.xml                   // WAR包的描述文件 
	            | 
	            |— classes 
	            |          action.class       // java类文件，编译后的字节码
	            | 
	            |— lib 
	                      other.jar           // 依赖的jar包 
	                      share.jar 
	  ```
	- `classes/`
		- 存储类的方式：`com.web.servlet.homeServlet`类存储在`classes/`下的`com/web/servlet/homeServlet.class`中\
	- `static/`
		- 存储静态文件，可以上传jsp webshell
	- `web.xml`
		- 描述网站路由的文件
- # 反编译class文件
	- 可以使用`jadx`或者自带的`javap`
	- `javap`: `javap -c HelloWorld`
	- `jadx`: `jadx a.class`
- # 伪协议
	- `file://`可以读取文件夹
- # OGNL
	- https://cloud.tencent.com/developer/article/1554322
	-