- 参考
	- https://www.leavesongs.com/PENETRATION/javascript-prototype-pollution-attack.html
	- https://blog.csdn.net/qq_43085611/article/details/114745665
- # 简单示例
	- ![1648784344752](https://s2.loli.net/2022/04/01/fLGl2YoVkJMBbsz.png)
	- 错误示例
		- ```js
		  var a = {}, b = {}
		  // 这是给a增加一个键，原型链污染需要拿到对象的原型，修改其的键
		  a.__proto__ = {"aaa": "aaa"} 
		  ```
- # 绕过
	- `__proto__`
		- 使用`constructor.prototype`
- # merge
	- ```js
	  function merge(target, source) {
	      for (let key in source) {
	          if (key in source && key in target) {
	              merge(target[key], source[key])
	          } else {
	              target[key] = source[key]
	          }
	      }
	  }
	  ```
	- 如果使用这样的代码是无法实现污染的：
	- ```js
	  let o1 = {}
	  let o2 = {a: 1, "__proto__": {b: 2}}
	  merge(o1, o2)
	  console.log(o1.a, o1.b)
	  
	  o3 = {}
	  console.log(o3.b)
	  ```
	- 因为JS会将第二行中的`__proto__`识别成o2的原型，在merge进行遍历时就不会遍历到`__proto__`这个键
	- 但是这样是可以的：
	- ```js
	  let o1 = {}
	  let o2 = JSON.parse('{"a": 1, "__proto__": {"b": 2}}')
	  merge(o1, o2)
	  console.log(o1.a, o1.b)
	  
	  o3 = {}
	  console.log(o3.b)
	  ```
	- 因为JS会将第二行的`__proto__`识别成一个键
	-