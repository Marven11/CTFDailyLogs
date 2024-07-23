# ShareCard
	- fix
		- 在``parse_avatar``加上一行``assert self.avatar in os.listdir('avatars')``
		- 把``jwt.algorithms.get_default_algorithms()``换成``RS256``
		- 把``debug=True``删了
		-
- # ezjs
	- fix
		- ```shell
		  chmod a-w /app/views/*.ejs
		  chmod a-w /app/*.js
		  chmod a-w /app/views/
		  ```
-