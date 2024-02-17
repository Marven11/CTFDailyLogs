# reload
id:: 64259f13-0d14-4e84-a803-0b3b864163f8
	- 在比赛环境中，经常会阉割掉一些内置函数，我们可以尝试使用reload来重载。
	- 在Python2中，reload是内置函数，而在Python3中，reload则为imp或importlib模块下的函数，使用方法：
	- ```python
	  from imp import reload
	  reload(os).popen("whoami").read()
	  ```
	  ```python
	  from importlib import reload
	  reload(os).popen("whoami").read()
	  ```
	-