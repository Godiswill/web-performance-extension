# web 性能分析 chrome 插件

## 关键性能指标

![critical-performance-index](https://raw.githubusercontent.com/Godiswill/web-performance-extension/master/demo/01.png)

- 关键性能指标。忽略页面卸载时间、重定向时间。
1. 首包：`responseStart - domainLookupStart`
2. 首次渲染/白屏：`responseEnd - fetchStart`
3. 首次可交互：`domInteractive - fetchStart`
4. DOM Ready：`domContentLoadedEventEnd - fetchStart`
5. 页面加载：`loadEventStart - fetchStart`

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

## 资源分析

- 首页资源域名数量、加载大小和域名分布情况。

![](https://raw.githubusercontent.com/Godiswill/web-performance-extension/master/demo/02.png)

- 请求资源域名分布，与各个域名平均耗时，跨域未配置响应头域名分布，缓存命中率。

![](https://raw.githubusercontent.com/Godiswill/web-performance-extension/master/demo/03.png)

- 文本压缩率，未开启压缩的域名。

![](https://raw.githubusercontent.com/Godiswill/web-performance-extension/master/demo/04.png)

- 初始类型占比情况，`mime` 类型占比分类。

![](https://raw.githubusercontent.com/Godiswill/web-performance-extension/master/demo/06.png)

- 抽样10条最耗时的资源和 ajax 请求。

![](https://raw.githubusercontent.com/Godiswill/web-performance-extension/master/demo/05.png)

## 开发打包
### 开发调试
```bash
yarn start
```
### 打包插件
```bash
yarn build
```
