# HTTP

## MIME

Multipurpose Internet Mail Extensions。表示文件或字节流的类型和格式。常见的有`text/*，image/*，audio/*，video/*，application/*，multipart/*`。

## URI & URL & URN & PURL

URI指统一资源标识符，用来标识互联网上的资源。\
URL指统一资源定位器，是URI常见的表现形式，指明访问机制和资源的位置。包含`scheme, username, password, host, port, path, parameter, query, fragment`。\
URN指统一资源命名，指明在特定命名空间中的名称。与资源存放的位置无关。示例：`urn:namespace identifier:namespace specific string`\
PURL也是URL，但是指向中间解析服务器，访问时会返回资源真实的URL。该服务器提供了命名和解析服务。作为URN的过渡。

## 历史

### 0.9

### 1.0

### 1.1

## 特点

灵活可扩展；请求响应；无状态；可靠传输；

## 缺点

无状态；明文传输；队头阻塞；

## 报文

### 请求报文

> 起始行\
> method(请求方法) URI(资源) version(http版本)

#### method

GET: 获取资源\
HEAD: 获取资源元信息\
POST: 提交数据进行处理\
PUT: 修改资源\
DELETE: 删除资源\
OPTIONS: 获取资源的操作限制，跨域相关\
CONNECT: 建立连接隧道，用于代理服务器\
TRACE: 跟踪客户端到服务器的传输路径\
GET和POST区别点：1、语义；2、参数存放位置；3、参数编码；4、是否幂等；5、是否进行缓存；6、TCP报文数量；\
URL编码：将非ASCII码字符和界定符转为十六进制字节值(%20指空格)

#### 请求报文头部字段

* Accept: 数据格式，对应响应的Content-Type
* Accept-Encoding: 压缩方式，对应响应的Content-Encoding
* Accept-Language: 语言，对应响应的Content-Language
* Accept-Charset: 字符集，对应响应的Content-Type

### 响应报文

> 起始行\
> version(http版本) status code(状态码) reason phrase(原因说明)

#### status code

1xx: 中间状态，需后续操作\
100: continue; 101: switch protocol;

2xx: 成功状态\
200: ok; 204: no content; 206: partial content(Content-Range头部字段);

3xx: 重定向状态\
301: moved premanently; 302: moved temporary(浏览器不做缓存优化); 304: not modified;

4xx: 请求报文有误\
400: bad request; 403: forbidden; 404: not found; 405: method not allow; 406: not acceptable; 408: request timeout; 409: conflict; 413: request entity too large; 414: request uri too long; 429: too many request; 431: request header fields too large;

5xx: 服务端发生错误\
500: internal server error; 501: not implemented; 502: bad gateway; 503: service unavailable;

#### 响应报文头部字段

* Content-Length: 定长响应体需设置。如果小于正确值则进行截断，如果大于正常值则发生错误
* Transfer-Encoding: 非定长响应体设置为chunked，基于长连接进行推送数据
* Accept-Ranges: 表示接受分块获取资源，可用值为`bytes，none`
* Content-Range: 客户端设置`Range: bytes=start-end[, start-end]`，服务器需进行校验并返回对应的数据，分单段数据和多段数据处理。格式为`Content-Range: bytes start-end/total`

## 连接管理

### 短连接

发送请求并接收完响应后，关闭TCP连接。3次握手耗时，且存在慢启动。

### 长连接

多个http请求响应复用同一个TCP连接。后一个http请求响应需等待前面的，存在队头阻塞问题。\
队头阻塞解决方案：

* 并发TCP连接
* 域名分片

> 多连接存在一定缺点。服务器维持多连接资源消耗压力比较大；移动端也一样。

### 多路复用

http2.0解决方案。

## Cookie

`Cookie`和`Set-Cookie`头部字段。

* 生命周期：expires，max-age
* 作用域：domain, path。path为`/`匹配域名下任意路径
* 安全：secure, httponly, samesite

缺点

* 空间有限，最多存储`4kb`
* 每次请求都带上，存在性能问题，可通过指定domain和path
* 安全问题，以纯文本形式传输

## 缓存

### 强缓存

无需发送http请求

* Expires: 过期时间，http/1.0。存在服务器和客户端时间不一致问题
* Cache-Control: max-age, s-maxage, public, private, no-cache, must-revalidate, proxy-revalidate
* max-stale, min-fresh, only-if-cached

### 协商缓存

* Last-Modified, If-Modified-Since
* ETag, If-None-Match\
精准度上，ETag更优；性能上，Last-Modified更优；服务器优先考虑ETag。

### 缓存存放位置

* Service Worker
* Memory & Disk。各自有优缺点。比较大的文件会直接写入磁盘。内存使用率高的时候，优先写入磁盘。
* Push Cache

## 代理

* 负载均衡：随机算法、轮询、一致性hash、LRU
* 安全性：数据过滤、限流、心跳机制监听服务器状态
* 缓存

相关头部字段

* Via: 记录中间代理服务器
* X-Forwarded-For: 记录请求方ip地址。存在的问题：代理服务器需解析请求头并修改，存在一定性能问题；HTTPS中，原始报文不允许修改。可使用`代理协议`解决，PROXY + TCP4/TCP6 + 请求方地址 + 接收方地址 + 请求端口 + 接收端口。
* X-Real-IP: 记录客户端ip地址
* X-Forwarded-Host: 记录客户端域名
* X-Forwarded-Proto: 记录客户端协议名称

## 跨域

浏览器遵循同源策略（协议、主机、端口相同）。非同源站点有如下限制：不可读取和修改对方的DOM；不能访问对方的Cookie、IndexDB、LocalStorage；限制 XMLHttpRequest 请求。

### CORS

简单请求\
请求方法为GET、HEAD、POST；头部字段包括Accept、Accept-Language、Content-Type（application/x-www-form-urlencoded、multipart/form-data、text/plain）、Content-Language。

浏览器自动加上Origin, 服务器将允许的域名通过Access-Control-Allow-Origin返回，浏览器再根据该字段决定是否拦截响应

非简单请求\
先会发送预检请求

```http
OPTIONS URI HTTP/1.1
Origin: 当前地址
Host: 当前域名
Access-Control-Request-Method: 跨域请求使用的方法
Access-Control-Request-Headers: 跨域请求设置的头部字段
```

* Access-Control-Allow-Origin: 允许的域名
* Access-Control-Allow-Methods: 允许的请求方法
* Access-Control-Allow-Credentials: 允许发送Cookie
* Access-Control-Allow-Headers: 允许设置的头部字段
* Access-Control-Expose-Headers: 跨域请求返回的额外头部字段
* Access-Control-Max-Age: 预检请求的有效期

### JSONP

利用`script`标签可以获取跨域资源实现。\
兼容性好，但请求方法单一。

### 反向代理

利用反向代理服务器设置域名为站点域名，并将请求代理到真正的服务器。

### 其他

postMessage, WebSocket
