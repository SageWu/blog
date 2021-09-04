# Websocket

## 握手过程

通过http请求，双方协商升级协议。后续按Websocket协议发送数据。

1、客户端发起请求

```http
GET url http/1.1
Connection: Upgrade
Upgrade: websocket
Sec-Websocket-Version: 13
Sec-Websocket-Key: ...
```

其中`Sec-Websocket-Key`为随机字节序列的base64字符串。

2、服务端响应

```http
http/1.1 101 switching protocols
Connection: Upgrade
Upgrade: websocket
Sec-Websocket-Accept: ...
```

其中`Sec-Websocket-Accept`为`Sec-Websocket-Key`拼接特定字符串，再通过`SHA1`计算摘要，再转为base64。

## 心跳

发送方发起`ping`控制帧，接收方响应`pong`控制帧。