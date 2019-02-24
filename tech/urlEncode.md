# URL编码

今天，使用markdown编辑编辑文章，碰到如下链接中带有小括号的的带链接title(`[title](url)`)：<br>
`[为什么你的n(Node版本管理)命令不起作用](http://hardog.net/2016/02/28/Nodejs/为什么你的(Node版本管理)命令不起作用)`<br>
由于小括号对于markdown有特殊含义，编辑后的实际页面显示和链接跳转与预期不用。于是产生了一个问题：如何能在markdown中编辑链接中能带小括号的title。

引文markdown本身支持html a标签，方法之一即是将title写成a链接形式：<br>
`<a href="http://hardog.net/2016/02/28/Nodejs/为什么你的n(Node版本管理)命令不起作用">为什么你的n(Node版本管理)命令不起作用</a>`<br>
不过，这种写法有时会破坏上文本编辑时上下文一致的美感。
另一种方法则是将它写成：<br>`[为什么你的n(Node版本管理)命令不起作用](http://hardog.net/2016/02/28/Nodejs/为什么你的n%28Node版本管理%29命令不起作用)`<br>左小括号用%28代替，右小括号用%29代替。[点击这里查看ASCII字符对应的URL编码](http://www.w3school.com.cn/tags/html_ref_urlencode.html)。

为什么使用%28能够%29能够分别代替左右括号呢？带着这个疑问我搜索了一些URL编码相关文章。下面我的URL编码相关摘录。

## [关于URL编码]((http://www.ruanyifeng.com/blog/2010/02/url_encoding.html)

网络标准RFC 1738做了硬性规定：只有字母和数字[0-9a-zA-Z]、一些特殊符号`$-_.+!*'(),`、以及某些保留字，才可以不经过编码直接用于URL。

但是，RFC 1738没有规定具体的编码方法，而是交给应用程序（浏览器）自己决定。这导致"URL编码"成为了一个混乱的领域。

URL编码汉字包含四种不同的情况。
1. 网址路径中包含汉字使用utf-8编码，如`%E6%98%A5%E8%8A%82`。
2. 查询字符串包含汉字，用的是操作系统的默认编码。
3. Get方法生成的URL包含汉字，用的是网页的编码。
4. Ajax调用的URL包含汉字，IE总是采用GB2312编码（操作系统的默认编码），而Firefox总是采用utf-8编码。

有没有办法能够保证客户端只用一种编码方法向服务器发出请求？答案是有的，可以使用Javascript先对URL编码，然后再向服务器提交。Javascript语言用于编码的函数，一共有三个，最古老的一个就是escape()，其它两个是encodeURI()、encodeURIComponent。

escape()虽然这个函数现在已经不提倡使用了，但是由于历史原因，很多地方还在使用它。它不能直接用于URL编码，它的真正作用是返回一个字符的Unicode编码值。具体规则是，除了ASCII字母、数字、标点符号`@ * _ + - . /`以外，对其他所有字符进行编码。在\u0000到\u00ff之间的符号被转成%xx的形式，其余符号被转成%uxxxx的形式。对应的解码函数是unescape()。[点击这里查看unicode相关资料](http://www.ruanyifeng.com/blog/2007/10/ascii_unicode_and_utf-8.html)。

encodeURI()是Javascript中真正用来对URL编码的函数。
它着眼于对整个URL进行编码，因此除了常见的符号以外，对其他一些在网址中有特殊含义的符号`; / ? : @ & = + $ , #`，也不进行编码。编码后，它输出符号的utf-8形式，并且在每个字节前加上%。

最后一个Javascript编码函数是encodeURIComponent()。与encodeURI()的区别是，它用于对URL的组成部分进行个别编码，而不用于对整个URL进行编码。
因此，`; / ? : @ & = + $ , #`，这些在encodeURI()中不被编码的符号，在encodeURIComponent()中统统会被编码。至于具体的编码方法，两者是一样。

## [HTML5 URL 编码](http://wiki.jikexueyuan.com/project/html5/url-encoding.html)

URL 编码就是将 URLs 中不宜打印的字符或者具有特殊意义的字符转换为 Web 浏览器和服务器明白且普遍接受的表示法。 这些字符包括：

- ASCII 控制字符 - 不宜打印的字符通常用于输出控制。字符范围是十六进制的 00-1F（十进制的 0-31）和 7F（十进制的 127）。下面提供了完整的编码表。

- 非 ASCII 控制字符 - 这些字符超出了 128 个 ASCII 字符集的范围。这个范围是 ISO-拉丁字符集的一部分以及包含整个十六进制的 ISO-拉丁字符集 00-FF （十进制的 128-255）的“前半部分”。下面提供了完整的编码表。

- 保留字符 - 诸如美元符号，和号，加号，通用符号，正斜杠，冒号，分好，等号，问号以及 “at”这类符号。所有这些符号在 URL 内都有不同的意义，因此需要编码。下面提供了完整的编码表。

- 不安全字符 - 包括空格，问号，小于符号，大于符号，磅字符，百分比符号，大括号左边部分，大括号右边部分，管道符，反斜杠，插入符号，波浪线。左方括号，右方括号，沉音符。出于某些原因，这些字符出现在 URLs 中存在被误解的可能性。这些字符也应该始终被编码。下面提供了完整的编码表。

编码表示法需要三个字符替换期望的字符：一个百分号，两个在 ASCII 字符集中表示字符位置的十六进制数字。

## 总结

JavaScript中encodeURI()、encodeURIComponent可以将URL中不能直接使用的字符(不宜打印的字符、具有特殊意义的字符等)转换为 Web 浏览器和服务器明白且普遍接受的utf-8编码格式。一个字符转化后会变成格式为百分号加上两个十六进制数字的三个字符。

JavaScript对ASCII中打印字符如[0-9a-zA-Z()]等也定义了URL编码。于是像小括号这种URL支持但是编辑器可能冲突的字符就可以使用[URL编码表](http://www.w3school.com.cn/tags/html_ref_urlencode.html)中对应的URL编码替代了。


## 相关资料

* [关于URL编码](http://www.ruanyifeng.com/blog/2010/02/url_encoding.html)
* [一张图看懂encodeURI、encodeURIComponent、decodeURI、decodeURIComponent的区别](https://juejin.im/post/5835836361ff4b0061f38a5d)
* [HTML5 URL 编码](http://wiki.jikexueyuan.com/project/html5/url-encoding.html)
* [HTML URL 编码](http://www.w3school.com.cn/tags/html_ref_urlencode.html)
* [ASCII](https://baike.baidu.com/item/ASCII)
* [ASCII，Unicode 和 UTF-8](http://www.ruanyifeng.com/blog/2007/10/ascii_unicode_and_utf-8.html)
* [Unicode与JavaScript详解](http://www.ruanyifeng.com/blog/2014/12/unicode.html)