# 异步流控制与事件

## 设计模式

## 事件模型、事件处理机制

思考：自定义事件的应用场景？

### 事件代理和委托

思考：事件代理和委托的优缺点？

### AJAX & fetch

思考：有 xhr 为什么还要 fetch？

扩展：了解 https://github.com/axios/axios 的实现（有时间可以看看源码）

### 异步流程控制

了解 callback hell 是如何产生的，以及 Promise + async/await 为什么能避免回调地狱

了解一下 node 的 error first

Promise + async/await

### callbackify & promisify

思考一下：如何实现 callback 和 promise 的互相转换

### RxJS


## 资料

* [观察者模式](https://addyosmani.com/resources/essentialjsdesignpatterns/book/#observerpatternjavascript)
* [观察者模式与发布/订阅模式区别](https://www.cnblogs.com/lovesong/p/5272752.html)


* [事件接口(MDN)](https://developer.mozilla.org/en-US/docs/Web/API/Event)
* [DOM事件体系](https://www.w3.org/TR/DOM-Level-3-Events/#dom-event-architecture)
* [事件模型](http://javascript.ruanyifeng.com/dom/event.html)
* [JavaScript 事件委托详解](https://zhuanlan.zhihu.com/p/26536815)
* [w3c事件模型除了冒泡还有捕获，捕获阶段存在的意义是什么？](https://www.zhihu.com/question/39474653)


* [Ajax(MDN)](https://developer.mozilla.org/zh-CN/docs/Web/Guide/AJAX)
* [XMLHttpRequest(MDN)](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest)
* [Fetch_API(MDN)](https://developer.mozilla.org/zh-CN/docs/Web/API/Fetch_API)


* [回调地狱](http://callbackhell.com/)
* [Promise(MDN)](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)
* [async_function(MDN)](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function)


* [util.callbackify(original)](https://nodejs.org/dist/latest-v9.x/docs/api/util.html#util_util_callbackify_original)
* [util.promisify(original)](https://nodejs.org/dist/latest-v9.x/docs/api/util.html#util_util_promisify_original)
* [promisify](https://github.com/nodejs/node/blob/e3d05a61215b2958cc1951759e08e816b35f5027/lib/internal/util.js#L255-L290)
* [callbackify](https://github.com/nodejs/node/blob/e3d05a61215b2958cc1951759e08e816b35f5027/lib/util.js#L370-L395)


* [Generator(MDN)](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Generator)
* [function\*(MDN)](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/function%2A)


* [RxJS](https://rxjs-dev.firebaseapp.com/)
* [构建流式应用—RxJS详解](https://github.com/joeyguo/blog/issues/11)