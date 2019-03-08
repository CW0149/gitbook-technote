# 正则与模版引擎

## 正则

* [精通正则表达式笔记](../tech/regexp.md)

### 使用正则表达式方法

| 方法 |	描述 |
|:----------:|-------------|
| exec |	一个在字符串中执行查找匹配的RegExp方法，它返回一个数组（未匹配到则返回null） |
| test |	一个在字符串中测试是否匹配的RegExp方法，它返回true或false |
| match |	一个在字符串中执行查找匹配的String方法，它返回一个数组或者在未匹配到时返回null |
| search |	一个在字符串中测试匹配的String方法，它返回匹配到的位置索引，或者在失败时返回-1 |
| replace |	一个在字符串中执行查找匹配的String方法，并且使用替换字符串替换掉匹配到的子字符串 |
| split |	一个使用正则表达式或者一个固定字符串分隔一个字符串，并将分隔后的子字符串存储到数组中的String方法 |

### 元字符

| 常用元字符   |      含义      |
|:----------:|-------------|
| . | 匹配除换行符之外的任何单个字符，添加了s修饰符除外  |
| \\ | 转义字符。使用 new RegExp("pattern") 的时候不要忘记将 \ 进行转义，因为 \ 在字符串里面也是一个转义字符。 |
| ^ | 匹配输入的开始。如果多行标志被设置为true，那么也匹配换行符后紧跟的位置 |
| $ | 匹配输入的结束。如果多行标示被设置为true，那么也匹配换行符前的位置 |
| \w | 匹配一个单字字符（字母、数字或者下划线） |
| \W | 匹配一个非单字字符 |
| \d | 匹配一个数字字符 |
| \D | 匹配一个非数字字符 |
| \s | 匹配一个空白字符，包括空格、制表符、换页符和换行符。等价于[ \f\n\r\t\v\u00a0\u1680\u180e\u2000-\u200a\u2028\u2029\u202f\u205f\u3000\ufeff] |
| \S | 匹配一个非空白字符 |
| \n | 返回最后的第n个子捕获匹配的子字符串(捕获的数目以左括号计数) |
| \0 | 匹配 NULL (U+0000) 字符， 不要在这后面跟其它小数，因为 \0<digits> 是一个八进制转义序列 |
| \xhh | 与代码 hh 匹配字符（两个十六进制数字） |
| \uhhhh | 与代码 hhhh 匹配字符（四个十六进制数字） |
| \u{hhhh} | (仅当设置了u标志时) 使用Unicode值hhhh匹配字符 (十六进制数字) |


### 量词

| 量词 |	含义 |
|:----------:|-------------|
| + | 匹配前面一个表达式1次或者多次。等价于 {1,} |
| * | 匹配前一个表达式0次或多次。等价于 {0,} |
| ? | 匹配前面一个表达式0次或者1次。等价于 {0,1} |
| {n} | n是一个正整数，匹配了前面一个字符刚好发生了n次 |
| {n,} | n是一个正整数，匹配了前面一个字符发生次数 >= n 次 |
| {n, m} | n 和 m 都是整数。匹配前面的字符至少n次，最多m次。若 m 的值小于 n， 会报错 |

### 分支与字符组

分支用(|)表示，字符组用[]表示。

* 在字符组外面的元字符，在字符组中不一定是元字符，反之亦然。
	* -在字符组中的非开头结尾被当作元字符（表范围）处理。
	* ^在字符组外开头代表字符串起始位置。在字符组内部开头代表排除。其它位置则被当作普通字符处理。$在非尾位当作普通字符。
	* .?+..等在字符组中被当作普通字符。
	* \在字符组里面也代表转义字符，要表示\普通字符需要写成`\\`。\在字符串中也代表转义字符。
	* \b 在字符组中匹配退格符
* 排除型字符组([^...])表示匹配一个未列出的字符。
* 多选分支(|)没有字符组的排除功能([^...])
* 字符组只能也必须匹配一个字符。


### 分组和引用

```
/(\d{4})-(\d{2})-(\d{2})/
/(\d{4})-(\d{2})-\2/
/(?<year>\d{4})-(?<month>\d{2})-(?<day>\d{2})/
/(?<year>\d{4})-(?<month>\d{2})-\k<month>/
replace
RegExp.$[_1-9]
```

### 环视

* 肯定环视(?=)(?<=)
* 否定环视(?!)(?<\!)
* 顺序环视(?=)(?!)
* 逆序环视(?<=)(?<\!)
* 只匹配位置
* 有的正则引擎不支持逆序环视；支持逆序环视也可能不支持逆序引用/只能匹配固定长度文本/不容许多选分支。
	* 若逆序环视能匹配任意长度文本，引擎必须从字符串起始位置检查逆序环视表达式，如果逆序环视在长字符串尾端开始会造成很多浪费。


### 贪婪 & 惰性

贪婪模式——在匹配成功的前提下，尽可能多的去匹配：`/.*bbb/g.test(‘abbbaabbbaaabbb1234’)`，用于匹配优先量量词修饰的⼦子表达式。

惰性模式——在匹配成功的前提下，尽可能少的去匹配：`/.*?bbb/g.test(‘abbbaabbbaaabbb1234’)`，用于匹配忽略略优先量量词修饰⼦子表达式。

### 修饰符

- g → [global](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/RegExp/global)。"g" 标志意味着正则表达式应该测试字符串中所有可能的匹配。
- i → [ignoreCase](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/RegExp/ignorecase)。"i" 标志意味着在字符串进行匹配时，应该忽略大小写。
- m → [multiline](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/RegExp/multiline)。"m" 标志意味着一个多行输入字符串被看作多行。例如，使用 "m"，"^" 和 "$" 将会从只匹配正则字符串的开头或结尾，变为匹配字符串中任一行的开头或结尾。
- y → [sticky](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/RegExp/sticky)。仅从正则表达式的 lastIndex 属性表示的索引处搜索
- u → [unicode](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/RegExp/unicode)。"u" 标志开启了多种 Unicode 相关的特性。使用 "u" 标志，任何 Unicode 代码点的转义都会被解释。
- s → [dotAll](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/RegExp/dotall)。"s" 标志表示.元字符应该能匹配字符串中的换行符。

## 模版引擎

### 它是什么

模板文件和数据可以通过模板引擎生成最终文件。例如，使用模版 [ejs](https://ejs.bootcss.com/) ，你编写的 ejs 格式文件通过 ejs 模版引擎可以被转换成 html 代码，我们可以在模版中使用如下语法：

```
<% if (user) { %>
  <h2><%= user.name %></h2>
<% } %>
```

一个前端迷你模版引擎：[JavaScript Micro-Templating](https://johnresig.com/blog/javascript-micro-templating/)。

### 引擎分析

#### JavaScript Micro-Templating 存在的问题

1. 性能：模板引擎渲染的时候依赖 Function 构造器实现，Function 与 eval、setTimeout、setInterval 一样，提供了使用文本访问 javascript 解析引擎的方法，但这样执行 javascript 的性能非常低下。
2. 调试：由于是动态执行字符串，若遇到错误调试器无法捕获错误源，导致模板 BUG 调试变得异常痛苦。在没有进行容错的引擎中，局部模板若因为数据异常甚至可以导致整个应用崩溃，随着模板的数目增加，维护成本将剧增。

#### 提升模版引擎效率方法

1. 预编译。
2. 更快的字符串相加方式。很多人误以为数组 push 方法拼接字符串会比 += 快，要知道这仅仅是 IE6-8 的浏览器下。实测表明现代浏览器使用 += 会比数组 push 方法快，而在 v8 引擎中，使用 += 方式比数组拼接快 4.7 倍。所以 可以根据 javascript 引擎特性采用不同的字符串拼接方式。

### DOM Template & JSX

#### DOM Template

- 比较好静态分析
- 可以把模版直接插⼊入DOM，不会有太奇葩的事情发⽣
- 不需要写词法分析，一般可以直接⽤用现成的HTML parser

#### JSX

 JSX 是 JavaScript 的一个语法扩展，在React中被推荐使用。JSX看起来像模版语言，但是它拥有Javascript的能力。

- 打破了了内容和逻辑分离这个 web 传统的分离方式
- 尽量减少了了概念的引⼊
- 通过 component 化来拆解应⽤

### Web Component

**Web Components 由四项主要技术组成，它们可以一起使用来创建封装功能的定制元素：**

- Custom elements：一组JavaScript API。允许你定义custom elements及其行为，然后在用户界面中按需使用。
- Shadow DOM：一组JavaScript API。用于将封装的 Shadow DOM 树附加到元素并控制其关联的功能。通过这种方式，可以保持元素的功能私有，这样就可以被脚本化和样式化，而不用担心与文档的其他部分发生冲突。
- HTML templates（HTML模板）：&lt;template&gt; 和 &lt;slot&gt; 元素可以用来编写不在呈现页面中显示的标记模板，并作为自定义元素结构的基础被多次重用。
- HTML Imports（HTML导入）：一旦定义了自定义组件，最简单的重用它的方法就是使其定义细节保存在一个单独的文件中，然后使用导入机制将其导入到想要实际使用它的页面中。HTML 导入就是这样一种机制，尽管存在争议 — Mozilla 根本不同意这种方法，并打算在将来实现更合适的。

**实现web component的基本方法通常如下：**

1. 使用ECMAScript 2015类语法创建一个类，来指定web组件的功能。
2. 使用CustomElementRegistry.define()方法注册新自定义元素 ，并向其传递要定义的元素名称、指定元素功能的类，以及其所继承自的元素(可选)。
3. 如果需要的话，使用Element.attachShadow()方法将一个shadow DOM附加到自定义元素上。使用通常的DOM方法向shadow DOM中添加子元素、事件监听器等等。
4. 如果需要的话，使用<template> 和 <slot>方法定义一个HTML模板。再次使用常规DOM方法克隆模板并将其附加到shadow DOM中。
5. 在页面任何你喜欢的位置使用自定义元素，就像使用常规HTML元素那样。

## 资料

* [Regular_Expressions(MDN)](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Guide/Regular_Expressions)
* [RegExp Api(MDN)](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/RegExp)
* [es2018新特性](https://medium.com/front-end-weekly/javascript-whats-new-in-ecmascript-2018-es2018-17ede97f36d5)


* [前端模版](https://github.com/FE-star/2017.8/blob/master/%E4%BB%8E%E5%89%8D%E7%AB%AF%E5%B0%8F%E5%B7%A5%E5%88%B0%E4%B8%AD%E7%BA%A7%E5%B7%A5%E7%A8%8B%E5%B8%88%E7%9A%84%E5%BF%85%E5%A4%87%E6%8A%80%E8%83%BD%E2%80%94%E2%80%94%E5%89%8D%E7%AB%AF%E6%A8%A1%E7%89%88.pdf)
* [高性能JavaScript模板引擎原理解析](https://cdc.tencent.com/2012/06/15/%E9%AB%98%E6%80%A7%E8%83%BDjavascript%E6%A8%A1%E6%9D%BF%E5%BC%95%E6%93%8E%E5%8E%9F%E7%90%86%E8%A7%A3%E6%9E%90/)
* [doT 源码](https://github.com/olado/doT/blob/master/doT.js)
* [Web Components(MDN)](https://developer.mozilla.org/zh-CN/docs/Web/Web_Components)