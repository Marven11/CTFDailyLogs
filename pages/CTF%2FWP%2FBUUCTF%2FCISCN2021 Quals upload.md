- # 整体结构
	- 看到`index.php`接收上传文件到`image/`文件夹，`example.php`会解压并处理图片，结果保存到到`example/`文件夹
	- `index.php`判断文件是不是图片，过滤文件名，且判断文件的`width`和`height`是否为1
	- `example.php`会将图片裁减后以png格式保存
- # 思路
	- 制作抗裁减的png格式图片马，将其包含在zip文件中上传，调用`example.php`解压图片马并使用
- # 制作抗裁减的png格式图片马
  id:: 64185f7a-3b8a-4f7e-bfdd-c9e2c9b95192
	- 可以使用[这个项目](https://github.com/huntergregal/PNG-IDAT-Payload-Generator/)生成图片马，此项目会payload放入png的IDAT块中
- # 绕过文件名检测
	- 遍历可以发现`İ`在小写后为`i`，可以借此绕过文件名对`i`的检测
- # 绕过getimagesize检测
  id:: 6418600e-2e09-453d-b103-91bd7c6a7041
	- ```python
	  with open("image.zip", "rb") as f:
	      b = f.read()
	  
	  b += b"""
	  #define width 1
	  #define height 1"""
	  ```
-