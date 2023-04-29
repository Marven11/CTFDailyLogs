- 有时我们可以通过在请求路径或者header参数中插入换行来实现伪造header
- # Nginx
- $uri导致的CRLF注入漏洞
- ```
  location / {
    return 302 https://$host$uri;
  }
  ```
- 因为 `$uri` 是解码以后的请求路径，所以可能就会包含换行符，也就造成了一个CRLF注入漏洞
- [参考](https://www.leavesongs.com/PENETRATION/nginx-insecure-configuration.html)