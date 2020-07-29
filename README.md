# web 性能分析 chrome 插件

## 关键性能指标

![critical-performance-index](https://raw.githubusercontent.com/Godiswill/web-performance-extension/master/demo/01.png)

- 关键性能指标。忽略页面卸载时间、重定向时间。
1. 首包：`responseStart - domainLookupStart`
2. 首次渲染/白屏：`responseEnd - fetchStart`
3. 首次可交互：`domInteractive - fetchStart`
4. DOM Ready：`domContentLoadedEventEnd - fetchStart`
5. 页面加载：`loadEventStart - fetchStart`
6. FP、FCP

随着 web 应用的复杂度越来越高，越来越丰富，考虑用户的交互体验不仅仅是
优化减少 `DOMContentLoaded` 和 `onLoad` 的触发时间了，而需要根据业务形态
来进行一个综合性的考量，确保在用户交互整个过程都尽量最优。

首包，可以大致看出当前的网络情况，一般 `DNS` 消耗时间比较大，可以做些 `dns-prefetch`。
条件好的，由于网络原因，可以对移动端做些传统的 DNS 机制切换到 httpDNS，确保一个网络的稳定，也可以精准的获取用户画像。

首次渲染，传统性能优化考虑的重点。模拟时间点，HTML 完全接收后，比如浏览器进程的一些调度切换也需要消耗一定的时间，
一般作为一个大致的渲染开始点，真实的会有些延后。刚开始浏览器的背景色是白色的，如果设置了 `body` 背景色为灰色，
浏览器优先渲染 `body` 那么视觉上会有一个颜色的切换，俗称白屏花费时间与灰屏渲染起点。
现代浏览器也提供了精准的计算时间 `FP`、`FCP`，通过 `performance.getEntriesByType('paint')[0].startTime` 获取。

首次可交互/TTI，通常有时弱网环境，DOM 也许未完全加载完毕，但是可交互的 DOM 尽早渲染完毕，可以不妨碍用户交互，也是可以接受的。
甚至比那些页面加载完毕，但有JS长任务阻塞在执行，妨碍了用户交互的场景更好。

关注你的主角元素 FMP(first meaningful paint)，你页面最主要的元素加载时间点需要业务方和开发人员来确定计算，并没有标准。

- 各阶段指标瀑布流
1. unLoad：`unloadEventEnd - unloadEventStart`
2. Redirect：`redirectEnd - redirectStart`
3. AppCache：`domainLookupStart - fetchStart`
4. DNS：`domainLookupEnd - domainLookupStart`
5. TCP：`connectEnd - connectStart`
6. TTFB：`responseStart - requestStart`
7. 数据传输：`responseEnd - responseStart`
8. DOM：`domContentLoadedEventStart - responseEnd`
9. DCL：`domContentLoadedEventEnd - domContentLoadedEventStart`
10. 资源加载：`loadEventStart - domContentLoadedEventEnd`
11. onLoad：`loadEventEnd - loadEventStart`

unLoad，可以拿到上一个卸载花费的时间，比如监听了 `beforeunload`，`unLoad` 事件。
一般不怎么建议无故监听 `beforeunload` 事件，会对前进后退的缓存获取和快速加载跳转页面产生影响。

Redirect 重定向，考虑到网络往返与浏览器要起新的进程去承接新页面，会影响站点的加载速度，尽量避免重定向。

DNS，上面优化方式有说道。其他例如脚本加 `async`、`defer` 属性避免阻塞 DOM 构建。
处理包的大小啊，按需加载、tree-shaking、丑化压缩、减少长任务时间分片、60FPS 动画等。


## 资源分析

- 首页资源域名数量、加载大小和域名分布情况。

![](https://raw.githubusercontent.com/Godiswill/web-performance-extension/master/demo/02.png)

根据请求数量，想想是不是请求过大能否减少或合并一些请求，尝试一些懒加载技术，避免请求占道抢占的关键链路资源的请求。
可以看下哪些域名是第三方，看看资源能否集成到自己的服务器上。不可控的资源可能对自己的站点服务产生极大影响，造成损失。

- 请求资源域名分布，与各个域名平均耗时，跨域未配置响应头域名分布，缓存命中率。

![](https://raw.githubusercontent.com/Godiswill/web-performance-extension/master/demo/03.png)

各域名加载资源数量与占比一目了然。各域名总耗时与并发耗时，可以评估一下资源服务器的负载能力，选取的 CDN 是否最优。

对于未设置跨域时间头是无法获取资源关键时间信息，为了便于统计和避免对统计产生干扰。
对自己可控的服务可以考虑是否需要配置。这里会排查跨域资源域名，并列出方便整改。

对于新应用或改动较大的应用，资源缓存无法利用，造成性能抖动较大，在大致的缓存率的情况下比较平均性能才比较有参考意义。
也可以观察老资源设置缓存的一个情况。

- 文本压缩率，未开启压缩的域名。

![](https://raw.githubusercontent.com/Godiswill/web-performance-extension/master/demo/04.png)

这里会统计一些比如 `HTML`、`CSS`、`JS`、`JSON`、`svg+XML` 等文本类型的一个压缩配置情况。
统计出那些未正常配置例如 `gzip` 的资源服务器。

- 初始类型占比情况，`mime` 类型占比分类。

![](https://raw.githubusercontent.com/Godiswill/web-performance-extension/master/demo/06.png)

便于分析网站的一个形态，然后着手对站点重量资源进行性能优化分析。

- 抽样10条最耗时的资源和 ajax 请求。

![](https://raw.githubusercontent.com/Godiswill/web-performance-extension/master/demo/05.png)

会抽检一些最耗时的请求，偶尔可以查看到一些不经常出现的采集脚本耗费过多时间，
是否需要考虑利用网页的生命周期来对上报时间点和方式做些优化。

## 使用
[插件地址](https://chrome.google.com/webstore/detail/web-%E6%80%A7%E8%83%BD%E5%88%86%E6%9E%90/mhpaedeeneljbnmhmofceiollddmfjcd)

点击插件或右键生成或关闭性能分析。

## 建议收集

有好的建议或需求欢迎大家随便提，持续维护。

## 开发打包
### 开发调试
```bash
yarn start
```
### 打包插件
```bash
yarn build
```
