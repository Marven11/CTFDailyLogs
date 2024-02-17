# 缺陷
	- 在设置GET参数时会直接将参数拼接在 url 的后面
		- 如果参数前有空格就会直接将参数解析成地址
		- 举例：将`{"name": " file://proc/stat"}`传给 curl 模块解析
		- curl 就会将最终的地址解析为`http://url.com/?name= file://proc/stat`，从而解析为两条地址，分别获取
	- 当 CURLOPT_SAFE_UPLOAD 为 true 时，如果在请求前面加上@的话 phpcurl 组件是会把后面的当作绝对路径请求，来读取文件。
	- `CURLOPT_HTTPHEADER`CRLF
		- 将这个作为一个header传入：`"Aaa: Bbb\r\nadmin: true"`
		- `curl_setopt($ch, CURLOPT_HTTPHEADER, ["Aaa: Bbb\r\nadmin: true"]);`
		- 例子：[[CTFWEB/WP/攻防世界ez_curl]]
- # 示例代码
	- ```php
	  function get($url){
	      $ch=curl_init();
	      curl_setopt($ch, CURLOPT_URL, $url);
	      curl_setopt($ch, CURLOPT_HEADER, false);
	      curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
	      curl_setopt($ch, CURLOPT_FOLLOWLOCATION, true);
	      curl_setopt($ch, CURLOPT_SSL_VERIFYPEER, false); 
	      curl_setopt($ch, CURLOPT_SSL_VERIFYHOST, false);
	      $res = curl_exec($ch);
	      curl_close($ch);
	      return $res;
	  }
	  ```