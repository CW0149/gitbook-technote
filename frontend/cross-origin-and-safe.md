# 跨域与前端安全

## 跨域

### 同源策略

为保证Web的安全，浏览器对页面访问权限做了比较严格的限制，以防止恶意网站窃取其它网站的隐私数据如cookie，以及冒充用户身份直接向其它网站请求数据。浏览器默认非同源(同协议、域名、端口)的网站互相访问是不安全的，因此对非同源网站的访问制定了同源策略。

同源策略限制了页面的以下行为以保护用户数据：
- Cookie、LocalStorage 和 IndexDB 无法读取。
- DOM 无法获得-无法通过DOM获取页面数据。
- AJAX 请求不能发送-无法冒充用户请求其它网站后台。

但是，这种限制有时也会造成一些问题。因为，现在的网站功能很丰富，为快速实现功能，通常需要调用其它网站的api，向其它网站请求数据。有时我们可能维护不同域名的多个网站，而我们希望有的网站间能够共享数据以实现体验上的优化。上述情况下我们就需要打破这种限制。

同源策略默认网站间是不可相互信任的，并且所有非公开数据都是需要被保护的。理论来说，如果在服务器将资源设置成公开或将不同源网站标记为相互可信任就可以打破这种限制了。有哪些方法能够打破同源限制呢？

### 子域名数据共享

只要给同子域名不同的网站设置了相同的document.domain(当前域或父域，不能为根域)，它们就能相互访问Cookie和DOM。

浏览器会区分 document.domain 属性从没有被设定过值和被显示的设定为跟该文档的URL的domain一致的值。尽管这两种状况下，该属性会返回同样的值，两个文档只有在 document.domain 都被设定为同一个值，表明他们打算协作；或者都没有设定 document.domain 属性并且URL的域是一致的，这两种条件下，一个文档才可以去访问另一个文档。

### 独立域名网站通信

HTML5中引入了跨文档通信 API（Cross-document messaging），来解决不同源间通信问题。这个API为window对象新增了一个window.postMessage方法，允许跨窗口通信，不论这两个窗口是否同源。

```
// aaa.com
var popup = window.open('http://bbb.com', 'title');

// 第一个参数是具体的信息内容
// 第二个参数是接收消息的窗口的源（origin）。也可以设为*，表示不限制域名，向所有窗口发送。
popup.postMessage('Hello World!', 'http://bbb.com');

// 子窗口向父窗口发送消息
window.opener.postMessage('Nice to see you', 'http://aaa.com');

// 父窗口和子窗口都可以通过message事件，监听对方的消息。
// event.source：发送消息的窗口
// event.origin: 消息发向的网址，可以过滤不是发给本窗口的消息
// event.data: 消息内容
window.addEventListener('message', receiveMessage);
function receiveMessage(event) {
  if (event.origin !== 'http://aaa.com') return;
  if (event.data === 'Hello World') {
      event.source.postMessage('Hello', event.origin);
  } else {
    console.log(event.data);
  }
}
```

通过window.postMessage，读写其他窗口的 LocalStorage 也成为了可能。

### 跨域ajax

由于浏览器AJAX请求只能发给同源的网址，对服务器没有限制，我们可以通过架设服务器代理来避开对同源的限制。除此之外，还有其它三种方法：
- JSONP
- WebSocket
- CORS

#### JSONP

script、img、iframe、link、video、audio等带有src属性的标签可以从不同的域加载和执行资源。JSONP就利用了script标签的这个特点。网页通过添加一个script元素，向服务器请求JSON数据，服务器收到请求后，将数据放在一个指定名字的回调函数里传回来，从而实现对跨域资源的访问。

具体操作是，与服务端约定好一个函数名，当请求文件的时候，服务端返回一个JSON，这个JSON会被解析成 JavaScript，JavaScript 则调用了约定好的函数，并且将数据当做参数传入。文件返回格式如`"typeof x === 'function' && x(...)"`。

JSONP是服务器与客户端跨源通信的常用方法，操作简单方便、浏览器支持度高。很多公用API接口都支持JSONP。

#### CORS

CORS（Cross-Origin Resource Sharing），是主流的跨域解决方案，获得所有主流浏览器支持。相比JSONP只能发GET请求，CORS允许任何类型的请求。

CORS需要浏览器和服务器同时支持。整个CORS通信过程，都是浏览器自动完成，不需要用户参与。浏览器一旦发现AJAX请求跨源，就会自动添加一些附加的头信息，有时还会多出一次附加的请求。事实上，实现CORS通信的关键是服务器，只要服务器实现了CORS接口，就可以跨源通信。

对于普通跨域请求，只需服务端设置Access-Control-Allow-Origin。

对于带cookie请求(cookie为跨域请求接口所在域的cookie，而非当前页)，前后端都需要设置字段。因为为了方便，有的开发人员可能给所有回应设置Access-Control-Allow-Origin: \*，为了避免意外，需要后台服务器指明Access-Control-Allow-Origin的域名，以及知名Access-Control-Allow-Credentials为true。前台也需要设置`xhr.withCredentials = true;`。

```
// 页面
xhr.withCredentials = true;

// 服务器
res.writeHead(200, {
  'Access-Control-Allow-Credentials': 'true', // 后端允许发送Cookie
  'Access-Control-Allow-Origin': 'http://www.domain1.com', // 允许访问的域（协议+域名+端口）
  'Set-Cookie': 'l=a123456;Path=/;Domain=www.domain2.com;HttpOnly' // HttpOnly:脚本无法读取cookie
});
```

#### Websocket

WebSocket是一种通信协议，使用ws://（非加密）和wss://（加密）作为协议前缀。该协议不实行同源政策，只要服务器支持，就可以通过它进行跨源通信。

浏览器发出的WebSocket请求的头信息会带有Origin字段，表示该请求发自哪个域名。因此，服务器可以根据这个字段，判断是否许可本次通信，也就不需要同源策略。如果该域名在白名单内，服务器就会做出回应。

## 前端安全

同源策略通过限制可能的恶意网站对不同源网站的访问用以确保用户数据的安全性。网站安全则是讲“黑客”如何通过各种方式获取用户隐私数据或者恶搞网站。方式主要包括，伪造网站、直接在目标网站上窃取用户数据(植入代码)或进行流量劫持、破解用户密码等。

### [XSS跨站脚本攻击](https://www.cnblogs.com/unclekeith/p/7750681.html)

XSS(Cross Site Scripting)-跨站脚本攻击，是攻击者将客户端脚本代码注入到其他用户的浏览器中的攻击手段。由于注入到浏览器的代码来自站点，因此可以做类似将该用户cookie发送给攻击者的事。

#### 攻击方式

##### 反射型XSS(非持久型)

Web客户端使用Server端脚本生成页面为用户提供数据时，如果未经验证的用户数据被包含在页面中而未经HTML实体编码，客户端代码便能够注入到动态页面中。


##### 存储型XSS(持久型)

黑客将攻击脚本上传到Web服务器上，使得所有访问该页面的用户都面临信息泄漏的可能，其中也包括了Web服务器的管理员。

最典型的就是留言板XSS。用户提交了一条包含XSS代码的留言到数据库。当目标用户查询留言时，那些留言的内容会从服务器解析之后加载出来。浏览器发现有XSS代码，就当做正常的HTML和JS解析执行。XSS攻击就发生了。

##### DOM XSS

DOM-based XSS是一种基于DOM的XSS漏洞。简单理解，DOM XSS就是出现在JavaScript代码中的漏洞。与普通XSS不同的是，DOM XSS代码不需要服务器端的解析响应的直接参与，即恶意代码并不在返回页面源码中回显，这使我们无法通过特征匹配来检测DOM XSS，给自动化漏洞检测带来了挑战。

例如：前端经常读取URL中的参数来显示到页面中，那么攻击者完全可以伪造一个带有XSS攻击代码的URL到参数当中，直接导致页面执行XSS脚本，触发XSS攻击。

```
http://www.abc.com/test.html 代码如下：

<script>
eval(location.hash.substr(1));
</script>
触发方式为：http://www.abc.com/test.html#alert(1)
```

如上，eval语句有一个作用是将一段字符串转换为真正的JS语句，因此在JS中使用eval是很危险的事情，容易造成XSS攻击。应避免使用eval语句。

```
test.addEventListener('click', function () {
  var node = window.eval(txt.value)
  window.alert(node)
}, false)

txt中的代码如下
<img src='null' onerror='alert(123)' />
```

#### 危害

- 通过document.cookie盗取cookie
- 使用js或css破坏页面正常的结构与样式
- 流量劫持（通过访问某段具有window.location.href定位到其他页面）
- Dos攻击：利用合理的客户端请求来占用过多的服务器资源，从而使合法用户无法得到服务器响应。
- 利用iframe、frame、XMLHttpRequest或上述Flash等方式，以（被攻击）用户的身份执行一些管理动作，或执行一些一般的如发微博、加好友、发私信等操作。
- 利用可被攻击的域受到其他域信任的特点，以受信任来源的身份请求一些平时不允许的操作，如进行不当的投票活动。

#### 防范

**代码可能存在的问题：**

- 没有过滤危险的DOM节点。如具有执行脚本能力的script, 具有显示广告和色情图片的img, 具有改变样式的link, style, 具有内嵌页面的iframe, frame等元素节点。
- 没有过滤危险的属性节点。如事件, style, src, href等
- 没有对cookie设置httpOnly。

**防范措施：**

- 编码：不能对用户输入的内容都保持原样，对用户输入的数据进行字符实体编码。
- 解码：原样显示内容的时候必须解码，不然显示不到内容了。
- 过滤：把输入的一些不合法的东西都过滤掉，从而保证安全性。如移除用户上传的DOM属性，如onerror，移除用户上传的Style节点，iframe, script节点等。

### [CSRF跨站伪造请求](http://renxm.com/javascript/2018/04/11/web-security.html)

CSRF(Cross-site request forgery)，也被称为one-click attack 或者session riding。是一种挟制用户在当前已登录的Web应用程序上执行非本意的操作的攻击方法。攻击者并不能通过CSRF攻击来直接获取用户的账户控制权，也不能直接窃取用户的任何信息。他们能做到的，是欺骗用户浏览器，让其以用户的名义运行操作。

跟跨网站脚本（XSS）相比，XSS 利用的是用户对指定网站的信任，CSRF 利用的是网站对用户网页浏览器的信任。

#### 攻击方式

跨站请求攻击是攻击者通过一些技术手段欺骗用户的浏览器去访问一个自己曾经认证过的网站并运行一些操作。这利用了web中用户身份验证的一个漏洞：简单的身份验证只能保证请求发自某个用户的浏览器，却不能保证请求本身是用户自愿发出的。

**下面是攻击的实例解释：**

John是一个恶意用户，他知道某个网站允许已登陆用户使用包含了账户名和数额的POST请求来转帐给指定的账户。John 构造了包含他的银行卡信息和某个数额做为隐藏表单项的表单页面，然后通过Email发送给了其它的站点用户（还有一个伪装成到 “快速致富”网站的链接的提交按钮）.

如果某个用户点击了提交按钮，一个POST请求就会发送给服务器，该请求中包含了交易信息以及浏览器中与该站点关联的所有客户端cookie(将相关联的站点cookie信息附加发送是正常的浏览器行为)。服务器会检查这些cookie，以判断对应的用户是否已登陆且有权限进行上述交易。

最终的结果就是任何已登陆到站点的用户在点击了提交按钮后都会进行这个交易。John发财啦！

![csrf攻击图解](https://upload-images.jianshu.io/upload_images/3636937-b8ba0dd4bc821bdd.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp)

#### 防范

- 使用合理的请求方法，如敏感操作请使用POST方法，而不是GET；
- 使用token。前端在每一个请求后面加一个token，token根据skey等用户唯一值生成，然后后端用同样的算法进行校验，通过后即可。目前公司绝大部分的业务都使用到了token。
	- 如果要求在访问敏感数据请求时，要求用户浏览器提供不保存在cookie中，并且攻击者无法伪造的数据作为校验，那么攻击者就无法再运行CSRF攻击。这种数据通常是窗体中的一个数据项。服务器将其生成并附加在窗体中，其内容是一个伪随机数。当客户端通过窗体提交请求时，这个伪随机数也一并提交上去以供校验。正常的访问时，客户端浏览器能够正确得到并传回这个伪随机数，而通过CSRF传来的欺骗性攻击中，攻击者无从事先得知这个伪随机数的值，服务端就会因为校验token的值为空或者错误，拒绝这个可疑请求。
- 校验Referer字段
	- HTTP头中有一个Referer字段，这个字段用以标明请求来源于哪个地址。在处理敏感数据请求时，通常来说，Referer字段应和请求的地址位于同一域名下。
	- 这种办法简单易行，工作量低，仅需要在关键访问处增加一步校验。但因其完全依赖浏览器发送正确的Referer字段。虽然http协议对此字段的内容有明确的规定，但并无法保证来访的浏览器的具体实现，亦无法保证浏览器没有安全漏洞影响到此字段。并且也存在攻击者攻击某些浏览器，篡改其Referer字段的可能。
- 验证码

### SSRF服务器伪造请求

SSRF(Server-Side Request Forgery) 是一种由攻击者构造形成由服务端发起请求的一个安全漏洞。一般情况下，SSRF攻击的目标是从外网无法访问的内部系统。（正是因为它是由服务端发起的，所以它能够请求到与它相连而与外网隔离的内部系统）

SSRF 形成的原因大都是由于服务端提供了从其他服务器应用获取数据的功能且没有对目标地址做过滤与限制。比如从指定URL地址获取网页文本内容，加载指定地址的图片，下载等等。

#### 攻击方式



#### 危害

- 对外网、服务器所在内网、本地进行端口扫描，获取一些服务的banner信息;
- 攻击运行在内网或本地的应用程序（比如溢出）;
- 对内网web应用进行指纹识别，通过访问默认文件实现;
- 攻击内外网的web应用，主要是使用get参数就可以实现的攻击（比如struts2，sqli等）;
- 利用file协议读取本地文件等。

#### 防范

- 过滤返回信息，验证远程服务器对请求的响应是比较容易的方法。如果web应用是去获取某一种类型的文件,那么在把返回结果展示给用户之前先验证返回的信息是否符合标准。
- 统一设置错误信息，避免用户可以根据错误信息来判断远端服务器的端口状态。
- 限制请求的端口仅为http常用的端口，比如，80,443,8080,8090。
- 将内网ip加入黑名单。
- 禁用不需要的协议。仅仅允许http和https请求。可以防止类似于file:///,gopher://,ftp:// 等。

### hijack劫持

#### 攻击方式

**页面劫持**

iframe 嵌套某页面，骗取用户输入信息。

**HTTP劫持**

大多数情况是运营商HTTP劫持，当我们使用HTTP请求请求一个网站页面的时候，网络运营商会在正常的数据流中插入精心设计的网络数据报文，让客户端（通常是浏览器）展示“错误”的数据，通常是一些弹窗，宣传性广告或者直接显示某网站的内容，大家应该都有遇到过。

**JSON劫持**

JSON是一种轻量级的数据交换格式，而劫持就是对数据进行窃取（或者应该称为打劫、拦截比较合适）。恶意攻击者通过某些特定的手段，将本应该返回给用户的JSON数据进行拦截，转而将数据发送回给恶意攻击者。一般来说进行劫持的JSON数据都是包含敏感信息或者有价值的数据。

**DNS劫持**

DNS 劫持就是通过劫持了 DNS 服务器，通过某些手段取得某域名的解析记录控制权，进而修改此域名的解析结果，导致对该域名的访问由原IP地址转入到修改后的指定IP，其结果就是对特定的网址不能访问或访问的是假网址，从而实现窃取资料或者破坏原有正常服务的目的。

#### 防范

- 页面劫持：Window.parent 判断

## 资料

* [关于同源策略的背景知识](https://learnsolid.cn/docs/#/acl/same-origin)
* [浏览器同源政策及其规避方法 - 阮一峰的网络日志](http://www.ruanyifeng.com/blog/2016/04/same-origin-policy.html)
* [浏览器的同源策略 - Web 安全 | MDN](https://developer.mozilla.org/zh-CN/docs/Web/Security/Same-origin_policy)
* [前端常见跨域解决方案（全） - inroam - 博客园](https://www.cnblogs.com/roam/p/7520433.html)


* [前端安全之XSS攻击](https://www.cnblogs.com/unclekeith/p/7750681.html)
* [新手指南：DVWA-1.9全级别教程之XSS](https://www.freebuf.com/articles/web/123779.html)
* [XSS实战：我是如何拿下你的百度账号](https://zhuanlan.zhihu.com/p/24249045)
* [跨站请求伪造(wikipedia)](https://zh.wikipedia.org/wiki/%E8%B7%A8%E7%AB%99%E8%AF%B7%E6%B1%82%E4%BC%AA%E9%80%A0)
* [新手指南：DVWA-1.9全级别教程之CSRF](https://www.freebuf.com/articles/web/118352.html)
* [JavaScript防http劫持与XSS](https://www.cnblogs.com/coco1s/p/5777260.html)
* [JSON劫持漏洞](https://shiyousan.com/post/635445288414621221)
* [Json劫持漏洞简介](https://blog.spoock.com/2016/04/12/json-hijacking/)


* [hack实践(hackthissite)](https://www.hackthissite.org/)
* [乌云漏洞列表](http://www.anquan.us/search?keywords=&&content_search_by=by_bugs&&search_by_html=False&&page=1)
* [web攻击工具Burp Suite](https://portswigger.net/burp/)