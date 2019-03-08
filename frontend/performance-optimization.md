# 性能优化

## [为什么要做性能优化](https://developers.google.cn/web/fundamentals/performance/why-performance-matters/)

* 性能关乎用户的去留
* 性能关乎转化率的提升
* 性能关乎用户体验
* 性能关乎用户

1. https://github.com/FE-star/exercise19
2. 尝试 PWA，至少写个 demo 玩玩 https://codelabs.developers.google.com/codelabs/your-first-pwapp/

## 性能checklist

## 常用性能测试工具

## HTTP 缓存

可以从 rfc7234 这个标准开始，了解相关的规定，rfc2616 已经废弃了

## 客户端缓存

- localStorage + sessionStorage
- App Cache (manifest)
- Service Worker

## 执行优化

- 测试代码的性能

## JIT(Just In Time)


## Lazy

- throttle & debounce
- setTimeout & requestAnimationFrame
- lazyload & preload
- 大列表优化

## 其他

- MTU + inline + base64
- 小图 vs 大图
- progressive vs baseline
- webp + webm
- HTTP/2 ?
- QUIC ?


## 资料

### 文章

* [Why performance matters](https://developers.google.cn/web/fundamentals/performance/why-performance-matters/)
* [前端性能checkList](https://github.com/JohnsenZhou/Front-End-Performance-Checklist)
* [chrome developer tools](https://developers.google.com/web/tools/chrome-devtools/)
* [Web Performance Best Practices and Rules](http://yslow.org/)
* [前端性能优化最佳实践](https://csspod.com/frontend-performance-best-practices/)
* [How to Make a Favicon Small and Cacheable](https://www.keycdn.com/blog/make-a-favicon)


* [rfc2616 is dead](https://www.mnot.net/blog/2014/06/07/rfc2616_is_dead)
* [rfc7234](https://tools.ietf.org/html/rfc7234)
* [Cache control(MDN)](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Cache-Control)
http://www.cnblogs.com/vajoy/p/5341664.html


* [A crash course in just-in-time (JIT) compilers](https://hacks.mozilla.org/2017/02/a-crash-course-in-just-in-time-jit-compilers/)
* [知乎专栏](https://zhuanlan.zhihu.com/qianduandaha)
* [Maximum transmission unit](https://en.wikipedia.org/wiki/Maximum_transmission_unit)
* [V8 性能优化杀手](https://juejin.im/post/5959edfc5188250d83241399)


### 工具

* [PageSpeed](https://developers.google.com/speed/pagespeed/insights/)
* [lighthouse](https://chrome.google.com/webstore/detail/lighthouse/blipmdconlkpinefehnmjammfjpmpbjk)


* [jsben](http://jsben.ch/)
* [benchmarkjs](https://benchmarkjs.com/)
* [jsperf](https://github.com/jsperf/jsperf.com)