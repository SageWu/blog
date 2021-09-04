# Performance

## 性能监控

### 性能指标

* 白屏时间：访问页面到页面有内容出现。
* 首屏时间：访问页面到首屏内容渲染完毕。
* 可交互：document的`readyState`变为`interactive`，可触发交互事件。
* 资源加载：`load`事件触发，所有资源加载完成。

### window.performance

memory：javascript对内存的占用。
timing：时间相关的信息。

## 白屏问题

由于框架是js驱动的，在js加载和解析完成前，页面暂无内容显示白屏。

### SSR

在服务端对页面进行渲染，再将渲染后的html返回给浏览器。\
不仅解决了白屏问题，同时解决了SEO的问题。\
缺点是维护成本较高；对服务端的压力比较大；给缓存造成一定的困难。

### 预渲染

使用`puppeteer`即无头浏览器，对页面进行预渲染，并对渲染结果进行快照处理，保存为html文件。在`webpack`中可使用`prerender-spa-plugin`。\
缺点是无法显示动态数据；预渲染展示的内容没有js交互。

### 骨架屏

将`puppeteer`生成的内容快照利用算法进行处理，生成骨架页面。在`webpack`中可使用`page-skeleton-webpack-plugin`。
