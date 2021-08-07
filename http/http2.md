# HTTP/2

## 二进制分帧层

对报文进行分帧，每一帧都通过二进制字节流的方式传输。

## 多路复用

每一帧都有对应的流id，代表所属的流。这样可以达到在单个TCP连接上传输多个流，每个流对于请求和响应。

## 优先级

流可以设置依赖以及权重，据此构建依赖树，得到流的优先级。

## Server Push

服务端会把资源关联的其他资源先推送到客户端，客户端可以进行缓存，加速后续资源请求速度。

## 头部压缩

使用HPACK算法，分别有静态字典和动态字典。

## Reset Frame

应用层关闭流连接。