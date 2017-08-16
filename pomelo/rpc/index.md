#RPC 方案配置

实际项目中，使用过zmq，也使用过pomelo-rpc，

## 方案1 zmq

```
//配置zmq
var zmq = require('pomelo-rpc-zeromq-renew');//0.0.9版本

app.set('proxyConfig', {
	rpcClient: zmq.client
});

app.set('remoteConfig', {
	rpcServer: zmq.server
});

```

##方案2 pomelo-rpc

```
var Client = require('pomelo-rpc').client;
var Server = require('pomelo-rpc').server;

app.set('proxyConfig', {
	rpcClient: Client
});

app.set('remoteConfig', {
	rpcServer: Server
});

```

##两种方案对比

[pomelo rpc zeromq 性能分析报告](https://github.com/NetEase/pomelo/wiki/pomelo-rpc-zeromq%E6%80%A7%E8%83%BD%E6%B5%8B%E8%AF%95%E6%8A%A5%E5%91%8A)
[pomelo rpc性能测试报告](https://github.com/NetEase/pomelo/wiki/pomelo-rpc%E6%80%A7%E8%83%BD%E6%B5%8B%E8%AF%95%E6%8A%A5%E5%91%8A)

### pomelo rpc增加回调超时机制

针对之前网友提出的[**rpc回调函数积压**](http://nodejs.netease.com/topic/52b2d8470a516e1851b256e4)问题, 在新版本的pomelo-rpc中通过增加了回调函数的超时机制解决，rpc客户端在记录应用层的回调函数的同时添加对应的定时器，如果rpc服务端收到对应的消息则将定时器清除，如果定时器超时则将对应的回调函数清除。定时器的超时时间开发者可以进行设置，默认是10s， 具体使用如下：

```

app.set('proxyConfig', {
	timeout: 1000 * 20
});

```