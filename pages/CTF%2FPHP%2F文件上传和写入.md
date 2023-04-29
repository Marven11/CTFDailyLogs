alias:: CTF/PHP/文件上传, CTF/PHP/文件写入
tags:: CTF/PHP/WAF绕过

- ## 文件名
	- {{embed [[CTF/PHP/特殊后缀名]]}}
	- 题目将 php 和 html 替换成"" -> 使用 phtmlhp
		- 使用`.php.jpg`或者`.jpg.php`
		- 使用`.php.sb`
			- http 服务器遇到 sb 等不能解析的文件后缀名时，会寻找前面的后缀名
	- 使用`c.php/b.php/..`
	- 使用`c.php/b.php/.`
	- 使用`a.PhP`
- ## 文件内容
	- {{embed [[CTF/PHP/特殊标签]]}}
	- `exif_imagetype`
		- 加上图片文件头绕过检测
	- 使用[[CTF/PHP/htaccess]]或者[[CTF/PHP/user_ini]]
	- 使用[[CTF/PHP/一句话]]
	- 使用 utf-16be 绕过
	- ((64185f7a-3b8a-4f7e-bfdd-c9e2c9b95192))
- ## 条件竞争
	- 在检测到恶意后缀名时，有时服务器不是拒绝存入文件，而是在存入文件后删除。
	- 这时我们就可以在恶意文件存入后，删除前访问此文件。
	- 例： [[CTF/WP/2022rydCTF/revenge_upload]]