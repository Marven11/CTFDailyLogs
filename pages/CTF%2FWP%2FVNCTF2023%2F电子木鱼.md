tags:: [[CTF/整数溢出]]

- # 题目
	- `/upgrade`接口会根据`name`获取对应的cost(`int32`)，^^如果`name`是`Donate`就将其cost乘以`quantity`^^，最后让功德减去价格，得到新的功德值
		- 其中`Donate`对应的cost是10
	- `/`会在功德很多的时候返回flag
	- `/reset`会重置功德
- # 题解
	- 我们知道`i32(10)`乘以一个很大的数会造成上溢出，得到一个很小的数，我们可以通过这个方法得到大量功德
		- ```python
		  r = base_post(f"{url}/upgrade", data = {
		      "name": "Cost",
		      "quantity": 2 ** 28
		  }, acceptable_code = [200, 400])
		  print(r.text)
		  ```