tags:: CTFWEB/Python

- # 污染static实现任意文件读
	- ```python
	  from sanic import Sanic
	  from sanic.response import text
	  from pathlib import Path
	  
	  app = Sanic(__name__)
	  app.static("/static/", "./php_docker/")
	  
	  @app.get("/")
	  async def hello_world(request):
	      app.router.name_index["__mp_main__.static"].handler.keywords["file_or_directory"] = "/home/cube/Code/toolkit/burst_kit"
	      app.router.name_index["__mp_main__.static"].handler.keywords["directory_handler"].directory = Path("/home/cube/Code/toolkit/burst_kit")
	      app.router.name_index["__mp_main__.static"].handler.keywords["directory_handler"].directory_view = True
	      return text("Hello, world.")
	  
	  
	  if __name__ == "__main__":
	  
	      app.run(host="0.0.0.0", port=8000)
	  
	  ```