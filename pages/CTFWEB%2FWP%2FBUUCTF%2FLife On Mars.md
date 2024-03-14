tags:: CTFWEB/SQL

- # 题解
- ![a](https://s1.328888.xyz/2022/08/08/0FHYI.png)
- 分析网页，猜测后台的sql可能为:
- `select name, description from TABLE_NAME`
- `select name, description from table where location=LOCATION`
- 审计网页代码可以发现网页有query接口，可以传入search参数进行查询
- 使用引号包裹失败，猜测SQL为上方第一种
- 剩下的为普通SQL题目解法，省略
- # 分析
- 在审计网页时，不仅要善于找出其中的漏洞，还要试图猜测包括SQL在内的后台代码逻辑