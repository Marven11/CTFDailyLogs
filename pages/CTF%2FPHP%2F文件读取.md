- `file_get_contents`
- `readgzfile`
- `readfile`
- `show_source`
- `highlight_file`
- `include`
  id:: 62f4a85a-d984-422f-8e43-2f0abb915f91
- `include_once`
- `require`
- `require_once`
- `fopen`
- `system`
	- `system('cat /flag')`
	- `system('cat /fla?')`
	- ((42eb4318-43ea-4079-963d-61943df49a8c))
- ```php
  $a=new SplFileObject("/flag");
  echo $a;
  ```
- `php -S`可能会造成源码泄漏
	- 例子： ((64f2a4b1-ae66-4b1e-9a95-744f30a923cb))