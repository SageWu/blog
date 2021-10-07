# Puppeteer

通过`DevTools`协议，提供高级API来控制Chrome或Chromium。\
安装puppeteer默认会下载最新版本的浏览器，可通过环境变量或者npm配置修改其行为。安装puppeteer-core则默认不下载任何浏览器，提供驱动的核心。

结构\
Puppeteer => Browser => BrowserContexts => Pages => Workers, Frames => ExecutionContext

## 调试

* headless修改为false
* 配置slowMo，放慢操作
* 设置devtools，在代码中插入debegger语句
* 监听console事件
* 开启日志打印，设置DEBUG环境变量
* 使用ndb运行
